# Agent 与 Tool Calling：原理、LangChain 基础与 MCP 接入

## 候选产品筛选接口

   * **候选产品（Candidate Products）**
       * > **定义**：从全量产品库中按本轮需求预先筛出的一小批产品，作为模型做最终推荐的候选池。
       * 推荐链路分四段：用户说出需求 → 后端按需求从产品库捞一批 → 候选产品连同需求交给模型 → 模型在池内筛选后推荐。
       * 拍板的是模型，但模型不在全量库里挑；接口的职责只有一句——查出一部分产品给模型用。
       * 全量产品可达数百、上千甚至上万条，整体发给模型不合适，所以必须先做一次初步筛选。
       * 查询候选产品，本质就是一次筛选。

   * **分类（category）**
       * 保险产品分四大类：重疾险、医疗险、寿险、意外险。
       * `category` 可传一个也可传多个；传两个就两类都推荐。
       * 查询按分类逐轮进行，每轮只查一个分类，最后汇总。

   * **保费预算（premium）**
       * 表示用户能接受的保费上限，落到查询条件是 `min_premium <= premium`。
       * 不传该参数就不加价格条件，不做价格筛选。
       * 产品表里除分类外还有最低保费字段，正好用来做这条过滤。

   * **每类限量（limit_per_category）**
       * 控制每个险种最多返回几条，接口约定值为 5，对应每轮查询的 `LIMIT 5`。
       * > **易错点**：限的是每个险种各 5 条，不是所有分类合计 5 条；查寿险与医疗险两类，返回的是 5 + 5 条再汇总。

   * **按分类分轮查询**
       * 一条 `IN` 语句查不完：`LIMIT` 限的是结果集总条数，`category IN ('life','medical') LIMIT 5` 完全可能 5 条全是寿险，医疗险一条没有。
       * 正确做法是一轮查一类，把 `category` 逐轮替换，最后手工合并各轮结果。
       * 每轮查询都带 `status = 'active'`，只取上架中的产品。
       * 排序按 `min_premium` 升序、同价再按 `id` 升序；`nulls_last()` 把价格为空的行排到末尾，不让它顶到最前。

   * **结果累积（extend）**
       * 进函数先定义空列表，每轮查到的结果追加进去，循环结束后自然汇总成一个篮子。
       * `list.extend()` 把另一个列表的元素逐个并入自身，原地变长；`+` 拼接产生新列表，原有对象不变。
       * 汇总必须用 `extend` 的原地语义，用 `+` 会丢掉累积结果。

   * **金额类型（Decimal）**
       * 金额与费率一律用 `Decimal`，它是十进制表示，没有精度损失。
       * 普通浮点数在计算机里存不住精确的十进制小数，不适合承载钱。

   * **字典去重（dict.fromkeys）**
       * 遍历分类前先用 `dict.fromkeys(category)` 去重，避免同一分类传两遍被查两遍。
       * 前端正常不会传重复值，这一层属于顺手加的保护，直接遍历原列表也能跑。

   * **接口分层**
       * 返回值为 `list[ProductResponse]`，无分页，直接一个列表。
       * 仓储层接收数据库会话、分类列表、预算、每类限量四个参数并返回产品列表。
       * 服务层只做透传：调仓储、传参、返回，不加额外处理。
       * 路由层调服务层并注入数据库会话。

---

## Agent 的本质

   * **智能体（Agent）**
       * > **定义**：能够感知环境、进行推理、自主决策并采取行动，以实现特定目标的智能系统。
       * 三个动作缺一不可：感知环境、推理、自主决策并采取行动。
       * 与聊天机器人的差距集中在后两个动作：聊天机器人只能生成文本，没有自主性。
       * **结论**：Agent 就是模型结合工具，给它安上手和脚，从而具备自主规划与任务执行能力；工具相当于模型的眼睛、手和脚。

   * **聊天机器人（Chatbot）**
       * 交互模式单一：你问一句，它答一句，核心能力只有文本生成。
       * 只能依据训练时见到的语料作答；训练时是一个值，现在早已变化，它感知不到，只能照训练时的样子编。
       * 比喻为"缸中之脑"：只有一个大脑，没有手、脚、眼鼻，不能感知外界也不能与外界交互，唯一能做的就是思考。
       * 对照对象是智能管家贾维斯：能看东西、能联网搜索、能发邮件、能订机票。

   * **规划、执行与感知闭环**
       * 同一句"五天北京、预算 8000、喜欢人文历史"，聊天机器人只能按旧训练数据编一套建议；Agent 先做任务规划。
       * 规划由模型完成：模型是大脑，先想清楚要查机票、查天气、查酒店、排行程、算预算余额。
       * 模型本身没有查机票、查天气的能力，必须提前准备工具并把工具清单告知它。
       * 拿到工具结果后要感知反馈：机票加酒店是否超预算；超了就带更严的价格限制重新查。
       * 判断与修正同样由模型完成，因为思考过程发生在大脑里。
       * 闭环反复转，直到查到的数据与规划结果符合用户要求为止。

```text
   用户目标：五天北京、预算 8000、喜欢人文历史
                         │
                         ▼
                ┌──────────────────┐
                │ 模型（大脑）做规划│
                └────────┬─────────┘
                         │ 这一步该干什么
                         ▼
                ┌──────────────────┐
                │     调用工具     │
                │  查机票 / 查天气 │
                └────────┬─────────┘
                         │ 拿到工具结果
                         ▼
                ┌──────────────────┐
                │     感知反馈     │
                │    预算够不够    │
                └────────┬─────────┘
                    否   │    是
              ┌──────────┘    └──────────▶ 输出旅行方案
              ▼
        修正条件，回到规划
```

---

## 工具与 Tool Calling 机制

   * **工具（Tool）**
       * > **定义**：工具的本质就是函数，只不过这个函数不是给人用的，是给模型用的。
       * 函数名对应工具名，参数列表对应工具接收的参数，文档注释对应工具说明书，`return` 的值对应工具执行结果。
       * 四个成分一一对应，写出这样一个函数就算定义好了一个工具。
       * 示例工具内部可以是假实现（永远返回晴天），不影响"定义"本身成立。
       * 模型看不到函数体，它只能看到递过去的说明书；说明书里没写的事，在模型眼里等于不存在。

   * **工具清单（tools）**
       * 模型跑在云端服务器，工具写在本地代码里，两者不在同一处。
       * 让模型知道有哪些工具的办法只有一个：把工具信息随请求一起发过去。
       * `tools` 是 chat completion 请求体里的一个参数，值是一个列表。
       * 列表每项形如 `{"type": "function", "function": {...}}`，外层 `type` 固定为 `function`。
       * `function` 里三件套：`name` 工具名、`description` 工具作用、`parameters` 参数说明。
       * `parameters` 用 JSON Schema 描述：`type` 为 `object`，`properties` 逐项列参数名与类型，`required` 列必填项。

```json
[
  {
    "type": "function",
    "function": {
      "name": "get_weather",
      "description": "Get the weather in a given location.",
      "parameters": {
        "type": "object",
        "properties": {
          "location": {"type": "string", "description": "The city name or coordinates."}
        },
        "required": ["location"]
      }
    }
  }
]
```

   * **工具调用（Tool Calling）**
       * > **定义**：模型只报"要调哪个工具、传什么参数"，真正执行发生在本地，执行结果再回传给模型的一整套机制。
       * 模型不需要真的调用远端函数，它只需要把调用意图传达出来，并拿到执行结果。
       * 核心只有两点：让模型知道有哪些工具；帮模型调用工具。
       * 工具清单、参数信息这些内容，本来就写在函数的文档注释里，取出来随请求发过去即可。

   * **调用意图（tool_calls）**
       * 模型返回的结果里，`content` 可能只是一句"我要检查一下天气"，真正的关键是 `tool_calls`。
       * 每个调用带一个模型生成的唯一 `id`，调用多个时靠它区分。
       * `function.name` 是要调的工具名，`function.arguments` 是参数，且是一段 JSON 字符串，不是字典，必须 `json.loads` 解析后才能用。
       * > **易错点**：带 `tool_calls` 的回复不是最终答案，只是中间步骤，不能直接转给用户。

   * **一次完整往返**
       * 程序把 `messages`（用户问题）与 `tools`（本地工具信息）组装成一次请求发给模型。
       * 模型推理后返回 `tool_calls`，指明工具名与参数。
       * 本地按名字找到函数、传参执行，拿到结果。
       * 把结果作为新消息，连同原问题与带 `tool_calls` 的消息再次发给模型——所谓"回传"就是重新发起一次请求。
       * 模型结合工具结果生成最终答案，程序再把答案返回用户。
       * 工具有多个时，模型按各工具的 `description` 与用户提问自行判断该调哪个。

---

## 手写 Tool Calling 与 ReAct 循环

   * **环境变量（Environment Variable）**
       * `.env` 是配置文件，不叫环境变量；环境变量是操作系统里的那一套。
       * `os.getenv()` 直接读 `.env` 里的键取不到值。
       * `load_dotenv()` 把 `.env` 里的键值对全部读出来塞进系统环境变量，之后就能用 `os.getenv` 取值。
       * 这一步与 pydantic 的 Settings 不冲突：Settings 只把值读进对象，并没有放进环境变量；而 LangChain、OpenAI SDK 取 key 走的是 `os.environ`。
       * > **易错点**：有 Settings 不等于环境变量里有 key；要让 key 进环境变量，仍然得 `load_dotenv()`。

   * **工具描述对象**
       * 两个函数就是两个工具：一个查天气、一个算平方根，内部实现都是假的。
       * 每个工具写成一段对象堆进列表，外层 `type` 写死 `function`。
       * `parameters` 里 `type` 写 `object`，`properties` 逐个列参数名与类型，`required` 说明哪些必填。
       * 手写这段极易少括号、错层级，正是框架要替掉的活。

   * **客户端与首次请求**
       * 客户端用 `OpenAI(api_key=..., base_url=...)` 构造，DeepSeek 也走这套兼容接口。
       * 请求方法是 `client.chat.completions.create()`，四个参数：模型名、消息、工具、厂商侧额外参数。
       * 厂商侧参数用 `extra_body` 传，关闭思考写作 `{"thinking": {"type": "disabled"}}`。
       * 首次请求返回的不是答案，而是"要不要调工具、调哪个"的判断结果。

   * **消息角色（Role）**
       * 消息角色共四种：`system`、`user`、`assistant`、`tool`。
       * `user` 装最初的问题；`assistant` 装回答或带 `tool_calls` 的调用意图；`tool` 装工具执行结果。
       * `tool` 是工具调用独有的角色，必须带 `tool_call_id` 与某次调用对上号，模型才知道这条结果是给哪个调用的。

   * **本地执行（execute_tool）**
       * 三件事：解析工具名与参数、按名字从工具映射表里取出函数并调用、把结果追加进消息列表。
       * 带 `tool_calls` 的那条 assistant 消息也要 `model_dump()` 后进历史，后面靠 `id` 对账。
       * 工具结果写成 `{"role": "tool", "tool_call_id": ..., "content": str(result)}`。
       * 第二次请求时消息列表里三样齐备：用户原问题、assistant 的调用意图、tool 的执行结果。

   * **ReAct（Reasoning and Acting）**
       * > **定义**："思考 + 执行"不断循环的模式；Re 是 reasoning（思考推理），Act 是 action（执行）。
       * 循环四段：用户提问 → 模型思考要不要调工具、调哪个 → 我们在本地执行 → 模型观察结果是否足以回答。
       * 不够就再调一次、再拿结果；够了才生成最终回答。
       * 分工固定：用户提问，模型负责思考，我们负责 action，执行结果再回给模型观察。
       * **结论**：Agent 不神秘，本质是模型加工具，再加这一圈 ReAct 循环。
       * 手写方案已被淘汰：每多一个工具就多一大段 JSON Schema、多一个分发分支，代码随工具数成倍增长。

---

## LangChain 的定位与最小 Agent

   * **LangChain**
       * 2022 年 10 月由 Harrison Chase 发布，时间与 ChatGPT 3.5 首次公开同期。
       * 最早只用于调模型，后来演变为智能体开发框架。
       * 官网把能力写成四段：build（构建）、test and evaluate（测试与评估）、deploy（部署）、monitor（监控与调试）。
       * 覆盖 Agent 开发的完整生命周期，每一段都有对应工具。

   * **build 三大框架**
       * LangChain：开发单个 Agent 的基础工具，负责把模型和工具绑起来。
       * LangGraph：工作流编排组件，把多个 Agent 组合成更复杂的流程。
       * Deep Agents：建在前两者之上的高级封装，开箱即用，但灵活性不如前两者。
       * LangSmith 负责测试、观察、评估与部署，收费，不在学习范围。
       * 企业里用得最多的是 LangChain 与 LangGraph。

   * **工具装饰器（@tool）**
       * 加在普通函数上，标记它不是普通函数而是工具。
       * 从 `langchain.tools` 或 `langchain` 导入均可。
       * 装饰器自动取三样信息：函数名作 `name`，文档注释作 `description`，参数列表作 `parameters`。
       * 文档注释不是写给同事的客套话，它是模型判断工具用途的唯一依据；写得含糊，模型就选错工具。
       * 加工具就是再写一个带 `@tool` 的函数。

   * **创建智能体（create_agent）**
       * 从 `langchain.agents` 导入，参数为模型与工具。
       * `model` 可以直接填模型名字符串，不必先初始化。
       * `tools` 类型是 Sequence，用列表包裹，只有一个工具也要包。
       * 模型与工具绑进 Agent 后，调用时只需发消息，不必再传工具。

   * **调用智能体（invoke）**
       * 入参必须是字典，字典必须有 `messages` 键，值是消息列表，每条消息是带 `role` 与 `content` 的字典。
       * 返回也是字典，`messages` 里有四条：`HumanMessage`、带 `tool_calls` 的 `AIMessage`、`ToolMessage`、最终 `AIMessage`。
       * 只取最后回答：`response["messages"][-1].content`。
       * 四步流程：加载环境变量 → 定义工具 → 定义 Agent → 调用 Agent。

   * **模型命名与选型**
       * 模型在 LangChain 里写个名字就够，不需要额外的模型对象。
       * 选型上 flash 版本升级后强于 pro，不必一见 pro 就往上冲。

---

## 模型初始化的三条路线

### 提供商推断

   * **模型提供商（Model Provider）**
       * > **定义**：模型的出处平台，如 OpenAI、DeepSeek、Anthropic、Google、阿里云。
       * 出处不同最直接的后果是接口地址不同，所以 `init_chat_model` 才有 `model_provider` 参数。
       * 只写模型名也能跑通，是因为 LangChain 按名字前缀推断提供商：`deepseek` 开头认定为 DeepSeek，`gpt` 开头认定为 OpenAI。
       * > **易错点**：推断依据是名字，不是实际连接的地址；名字一样、平台不一样时就会推错。典型是阿里云百炼上挂的 DeepSeek 模型，按名字会被认成 DeepSeek 官方平台。

   * **环境变量约定**
       * 每个提供商有固定的默认接口地址，推断出提供商后查表即可，所以 `base_url` 不用填。
       * 每个提供商有约定的环境变量名：OpenAI 读 `OPENAI_API_KEY`，DeepSeek 读 `DEEPSEEK_API_KEY`，所以 `api_key` 也不用填。
       * 约定只认标准名字，写成 `DSPIK` 这类自造名就找不到，接着报错。
       * 整套推断可以完全不理，手动传 `api_key` 与 `base_url` 一样能跑，只是多数时候没必要。

### 默认初始化

   * **init_chat_model**
       * 从 `langchain.chat_models` 导入，不是 `langchain_classic`，写错包函数就找不到。
       * `model` 是模型名称，必填；可以作位置参数，也可写 `model=...`，两种等价。
       * `model_provider` 不传则按名字前缀推断。
       * `api_key`、`base_url` 不传则走提供商约定。
       * 模型自身参数原样透传：`temperature` 采样温度、`max_tokens` 输出上限、`extra_body` 平台私有参数。

   * **默认路线的边界**
       * > **定义**：默认初始化是"只给名字，其余交给约定"；自定义初始化是"名字照给，但提供商、密钥、地址都得自己递上去"。
       * 默认路线只对 LangChain 支持的模型有效，名单基本是 OpenAI、DeepSeek、Anthropic、Google 几家，国内模型被支持的很少，DeepSeek 是其中一个。
       * 名单外的模型（如小米）推不出提供商——不是可能推错，是根本推不出来。
       * > **提示**：默认初始化不支持，不等于 LangChain 用不了这个模型；走自定义路线照样能调。

   * **五步骨架**
       * 加载环境变量 → 定义工具 → 初始化模型 → 创建 Agent（绑定模型与工具）→ 调用 Agent。
       * 比不单独初始化模型的版本多出的正是第三步。
       * 这五步后面会反复出现，写到不用想。

   * **思考开关**
       * 开启写作 `extra_body={"thinking": {"type": "enabled"}}`，关闭写作 `{"type": "disabled"}`。
       * 开启时模型先走一遍思路，返回里多一段 `reasoning content`，耗时约翻倍。
       * 关闭后耗时约减半，`reasoning content` 消失；一行配置换一倍时间差。
       * > **提示**：这种嵌套写法是 DeepSeek 平台私有约定，不同模型这一层长得完全不一样，去官方文档抄现成的即可，不必背。

### 自定义初始化

   * **OpenAI 兼容伪装**
       * 不在 LangChain 名单里的模型（如通义千问）三样都得自己递：提供商、`api_key`、`base_url`。
       * `model_provider` 不能填"阿里"这类 LangChain 不认识的值，填了等于没填。
       * 正解是填 `"openai"`：OpenAI 接口规范事实上已成行业通用标准，千问、DeepSeek 都兼容它。
       * 伪装有代价：写了 `model_provider="openai"`，LangChain 就默认地址指向 OpenAI、密钥去读 `OPENAI_API_KEY`，所以 `api_key` 与 `base_url` 必须手填。

   * **密钥入配置**
       * 每家账号的 key 都不一样，要在平台上自行开通申请，拿到属于自己账号的那一份。
       * key 与地址一律放 `.env`，不写死在代码里。
       * 变量名可以随便起，因为 LangChain 不认识这套、不会自动去读；阿里系习惯用 `DASHSCOPE_` 前缀。
       * `os.getenv("NAME")` 与 `os.environ.get("NAME")` 完全等价。
       * > **易错点**：不要把 key 直接粘成字符串字面量；key 一进代码就会跟着代码进版本库，`os.getenv` 读环境，代码里只有变量名。

   * **验证方式**
       * 模型建好后，Agent 代码一个字都不用改，照样 `create_agent(model, [tool])` 与 `invoke`。
       * 问一句"你是谁"，答"我是通义千问"就说明请求真的打到千问，而不是打到 OpenAI 拿到 401。

### 模型类初始化

   * **ChatOllama**
       * 适用于既不兼容 OpenAI、又不在 LangChain 名单里的平台，代表是本地部署平台 Ollama。
       * 从 `langchain_ollama` 导入 `ChatOllama`，不再用 `init_chat_model`。
       * 参数直接写模型名与温度：`ChatOllama(model="qwen3.5:0.8b", temperature=1.1)`。

   * **本地部署模型的调用特征**
       * 不需要 `api_key` 与 `base_url`：服务跑在本机，默认 `localhost`，也不验证密钥。
       * 本地推理没有网络往返，实测 800 毫秒左右出结果。
       * 问"你是谁"答"我是千问 3.5"，说明本地那份 0.8B 模型确实在服务。
       * > **易错点**：`ChatOllama` 的模型名参数叫 `model`，不叫 `name`；写成 `name=` 会报 `Field required`，报错看似缺必填项，实际是键名对不上。

### 三条路线对照

   * **路线选择条件**
       * 只看两个条件：LangChain 认不认识这个模型、对方兼不兼容 OpenAI 规范。
       * 两个条件依次对应三条路：认得的走默认；认不得但兼容的伪装成 OpenAI；认不得又不兼容的用专用模型类。

   * **各路线的代价**
       * 默认初始化最省事，代价是只能用 LangChain 名单内的模型。
       * 自定义初始化能接名单外模型，代价是密钥与地址都得自己维护。
       * 模型类初始化能接不兼容 OpenAI 的平台，代价是绑定到该平台的专用包。

| 模型情况 | 提供商 | api_key | base_url | 入口 |
| :--- | :--- | :--- | :--- | :--- |
| LangChain 支持的（DeepSeek、OpenAI 等） | 不写，按名字推断 | 不写，去环境变量读 | 不写，查表得到 | `init_chat_model(名字)` |
| 兼容 OpenAI 但不被支持（千问） | 写 `"openai"` 伪装 | 手填 | 手填 | `init_chat_model(...)` |
| 不兼容 OpenAI（Ollama） | 不需要 | 本机不需要 | 本机 localhost | `ChatOllama(...)` |

---

## 模型调用与 Agent 调用的分野

   * **直接调模型**
       * 不建 Agent 也能调模型，流程缩为三步：加载环境变量 → 初始化模型 → 调用模型。
       * 只需要一句回答、不需要规划与工具的场景，建 Agent 纯属多余，白搭一层工具绑定的开销。
       * 入参直接给字符串：`model.invoke("你好，你是谁")`。
       * 返回只有一条 `AIMessage`，文本在 `.content` 上，没有历史消息列表。

   * **两套 invoke 的进出**
       * 方法名相同、形状相似，但进出的东西完全是两套。
       * 模型调用相当于把 Agent 返回的一堆消息里，最后那条 AI 消息单独摘出来给你，中间步骤不返回。
       * > **易错点**：写过 Agent 之后再调模型，手会自己往 `invoke` 里塞 `{"messages": [...]}`，也会顺手写 `response["messages"]` 取值；前者不是它要的形状，后者它根本不是字典。

| 对比项 | Agent 调用 | 模型调用 |
| :--- | :--- | :--- |
| 方法 | `agent.invoke(...)` | `model.invoke(...)` |
| 入参 | `dict`，键 `messages` 指向消息列表 | 字符串，或消息列表 |
| 返回 | `dict`，键 `messages` 指向消息列表 | 一条 `AIMessage` |
| 取文本 | 从 `messages` 里取最后一条 | 直接读 `.content` |

   * **多消息入参**
       * `model.invoke` 的入参允许两种形态：`str`，或一个列表。
       * 列表元素不能直接放字符串，否则模型分不清哪句是用户说的、哪句是它自己前面说的。
       * 元素要换成带 `role` 与 `content` 的字典，常用 `system` 定人设、`user` 放提问，顺序即对话顺序。
       * > **提示**：模型自己不记上一轮；要维持人设，每一轮都得把 `system` 那条重新放回列表最前面。

---

## 消息对象与结构化输出

   * **消息基类（BaseMessage）**
       * 所有消息必备两样：`role`（角色）与 `content`（内容）。
       * `BaseMessage` 是一切消息的顶级父类，角色与内容这些最基本的信息都在里面。
       * 按角色派生出子类，把角色定死，构造时不必再写 `role`。

   * **四种子类**
       * `SystemMessage` 对应 `system`，用来定人设、定规则。
       * `HumanMessage` 对应 `user`，是我们说给模型的话。
       * `AIMessage` 对应 `assistant`，是模型返回的话，也用于存进记忆。
       * `ToolMessage` 对应 `tool`，是工具执行结果回填给模型。
       * 构造时第一个位置参数就是 `content`，例如 `SystemMessage("你是一个专业的 AI 助手")`。

   * **拼字典的代价**
       * 手写字典要加引号、加冒号，键名一个字母都不能错，错一个整段跑不通。
       * 写两三条还能忍，写到三五条就开始烦，消息越长代码越没法看。
       * 既然格式固定，就交给类封装：传对象，不传字典。

   * **结构化输出（Structured Output）**
       * > **定义**：让模型输出的不再是普通字符串，而是按预先定义的结构生成的固定格式数据。
       * 分两步：先把结构定义下来，再把结构绑到模型上。
       * 结构用 Pydantic 的 `BaseModel` 子类定义，每个字段用 `Field` 包一层并写 `description`；`description` 是写给模型看的说明书。
       * 绑定用 `model.with_structured_output(schema=结构类)`。
       * 这类需求不需要 Agent，直接调模型即可，它要的只是一次干净的回答。

   * **绑定返回新模型**
       * `with_structured_output` 不是给原模型"设置"一下就完，它返回的是一个带结构化输出能力的新模型。
       * 之后必须用这个新模型去 `invoke`，不能还拿原来的 `model` 调。
       * 绑完之后调用反而轻：一条消息就够，返回的是结构类对象，字段直接点出。
       * > **注意**：继续用原模型调用，结构化约束不会生效。

   * **结构化的收益**
       * 不加约束时输出是一大段字符串，要自己切、自己解析，字段还可能缺一两个。
       * 加了约束后字段一个不少、严格按类来，可以直接当对象用。
       * 换领域只需换结构类：人物设定就定义姓名、小传、性格，定义什么模型填什么。

---

## 阻塞调用、流式调用与系统提示词

   * **Agent 的阻塞调用**
       * `create_agent(model=model, tools=[])` 工具给空列表也能跑，架子先搭通，工具后面再往里加。
       * 入参必须是字典：`{"messages": [HumanMessage(...)]}`，`messages` 的值是列表，列表里装消息对象。
       * 返回也是字典，`response["messages"][-1].content` 取最后一条 AI 回答。
       * > **定义**：阻塞调用指调用发起后线程被占住，模型把结果全部生成完才返回，等待期间什么也干不了。
       * > **提示**：与流式对比时用"阻塞"这个词才准确，"同步"是口语混用。

   * **流式调用（stream_events）**
       * V3 模式的流式方法叫 `stream_events`，`version="v3"` 必须给，否则按 V1 处理。
       * 返回 `GraphRunStream` 对象，即一条流，不是结果；结果要从流里自己取。
       * 流里事件不止一种：`values` 是快照、`subgraphs` 是子 Agent、`extensions` 是自定义事件、`output` 是最终结果，只关心 `messages`。
       * 取值三层：`stream.messages` 拿消息事件 → 消息的 `.text` 是 `ChatModelStream` → 遍历它拿 `chunk`。
       * `end=""` 去掉 `print` 默认换行，否则一个片段一行、竖着往下掉；`flush=True` 立即输出，防缓冲区攒批导致的跳动甚至乱序。
       * 首 token 约几百毫秒出现，同内容走阻塞调用约五秒；运行时会提示 V3 流式协议处于实验阶段，不影响使用。
       * > **提示**：token 不一定是一个完整单词，实测 `deepseek` 会被拆成两段吐出，所以流式拿到的是片段，不是词。

```python
stream = agent.stream_events(
    {"messages": [HumanMessage("你好，你是谁")]},
    version="v3",
)

for message in stream.messages:
    for chunk in message.text:
        print(chunk, end="", flush=True)
```

   * **系统提示词（System Prompt）**
       * 放法一：在 `messages` 列表最前面塞一条 `SystemMessage`，本次调用生效。
       * 放法二：创建 Agent 时用 `system_prompt="..."` 一次绑定，之后 `messages` 里只放对话内容，每轮自动生效。
       * 系统提示词是固定内容，与本次问什么无关，每次调用都塞一遍既烦又容易漏。
       * **结论**：固定的人设与规则绑在 Agent 上，用 `system_prompt` 传一次，对话多少轮都算数。

---

## 工具描述的三种方式

   * **描述三要素**
       * 每次调模型都必须把工具信息一并告诉它，三样：工具的名字、作用描述、需要的参数。
       * 官方请求里的 `tools` 就是对这三样的承载：`name`、`description`、`parameters`。
       * `@tool` 取信息的途径不止文档注释一种，一共三种。

### 装饰器属性

   * **@tool 显式传参**
       * 名字直接传字符串：`@tool("square_root", ...)`，不用写成 `name=...`。
       * 作用描述写在 `description="..."`，参数约束可另用 `args_schema`。
       * 参数少、一句话能说清时最省事。

   * **工具命名**
       * 名字本身就是模型判断用途的线索：叫 `tool1` 模型不知道是干什么的，叫 `square_root` 一看就懂。
       * 删掉文档注释、名字又起得随便，工具就无法被正确调用，甚至会直接报错说函数里没有 doc string。

### 文档注释

   * **docstring 结构**
       * `@tool` 里什么属性都不加，函数后面直接跟一段文档注释。
       * 第一段写工具作用，`Args:` 段一个参数占一行，逐个说明含义。
       * 注释可以换行、可以写细，参数再多也放得下。

   * **参数多时的优势**
       * 三个参数（地点、温度单位、是否带预报）时，装饰器属性已经不好塞，文档注释仍很自然。
       * 写注释是顺手的事：既给函数加了说明，又完成了对模型的描述。
       * 实测模型会自行判断该传哪些参数，例如从"接下来几天"推出 `include_forecast=true`。

### BaseModel 约束

   * **Literal 钉死取值**
       * 参数多且有取值约束时用 BaseModel 作为 schema：`@tool(args_schema=WeatherInput)`。
       * 取值固定的参数用 `Literal["celsius", "fahrenheit"]` 声明，只允许在这几个值里选。
       * 不钉死的后果：代码用 `if unit == "celsius"` 判断，模型可能传中文"摄氏度"，判断不成立就会走进错误分支返回错值。
       * Pydantic 还会在代码侧校验，真传了别的值直接报错，而不是让 `if` 悄悄走错。

   * **args_schema 挂载**
       * 先定义继承 `BaseModel` 的类，每个属性跟一个 `Field(description=...)`。
       * 挂到装饰器后，文档注释只写一句简单介绍即可，参数约束交给类。
       * 缺点是繁琐：要先定义类再传，属于了解即可的写法。

### 描述方式的选择

   * **推荐方案**
       * 推荐文档注释：跟平常写函数一样加 `@tool`、加参数、配一段注释，其他两种看懂即可。
       * 理论上 BaseModel 能做的，文档注释也能做——把可选值写进文字说明一样有效，区别只是 BaseModel 多了"传错就报错"。

   * **参数名禁区**
       * 自定义工具的参数名有两个禁区：`config` 与 `runtime`，这两个名字 LangChain 内部要用，是框架预留的注入位。
       * > **注意**：参数一旦叫 `config` 或 `runtime`，框架会把它当成自己的注入位置，传进来的是运行时对象而不是模型给的值，工具行为就不受你控制。

---

## MCP 协议与角色分工

   * **模型上下文协议（Model Context Protocol，MCP）**
       * > **定义**：一套让 AI 应用便捷连接外部系统的开放协议，作用是暴露工具与获取工具，由 Anthropic 提出。
       * 动机是工具开发不完、全世界在重复开发：查天气、查时间、查地图、查酒店、查航班，各家 Agent 都有重复需求。
       * 有了统一协议，通用能力可由专业一方定义一次：航司提供查航班服务，气象部门提供天气服务，谁需要谁来取。
       * 它同时是"对外暴露工具"和"获取工具"的一套标准，全世界的工具遵循同一协议，调用方式也就统一了。
       * 类比 USB：外设厂家都按 USB 做接口，电脑只留 USB 口，任何外设一插就能用。
       * **结论**：MCP 只是协议与标准，规定怎么交互、怎么通信；它不是工具本身，也不是某个具体服务。

### 三个角色

   * **MCP 服务器（MCP Server）**
       * 提供 MCP 服务的应用，可以是远程服务，也可以是本地服务。
       * 工具挂在 server 后面，由它对外暴露。

   * **MCP 客户端（MCP Client）**
       * 用来连 MCP Server 的组件，连上就能读到 server 里的工具信息。
       * 想拿到某个 server 的工具，就用 client 去连它。

   * **MCP 主机（MCP Host）**
       * 可以理解成一个 AI 应用；AI 应用要用工具，内部必须持有一个 client。
       * 一个 host 可以挂多个 client，每个 client 各连一个 server。
       * 链路固定：host 持有 client → client 连 server → server 提供工具 → 工具交给 host 里的 Agent 调用。
       * 三者一句话概括：一个提供工具，一个获取工具，一个使用工具；对应 USB 那套就是外部设备、连接线、电脑。

```text
MCP Host（我们的 AI 应用）
├── client-1 ──▶ MCP Server（天气）──▶ 工具：查当前天气、查预报
├── client-2 ──▶ MCP Server（航班）──▶ 工具：查航班、订航班
├── client-3 ──▶ MCP Server（地图）──▶ 工具：地点搜索、路径规划
└── client-4 ──▶ MCP Server（文件）──▶ 工具：读文件、写文件
```

### 两种通信方式

   * **标准输入输出（stdio）**
       * 走本地进程的标准输入输出，是本地通信方式。
       * client 先从 server 端把工具"下载"到本地，作为一个进程跑起来，此后调工具不需要网络。
       * 一句话概括：先下载，再使用。

   * **流式 HTTP（Streamable HTTP）**
       * 底层就是 HTTP 请求与响应，是远程调用方式。
       * 工具留在远端、不落地，每调一次工具就发一次 HTTP 请求。
       * 本质仍是"发一个请求、收一个响应"，拿工具也是发请求。

   * **方式的选择**
       * 用哪种不由我们决定，取决于 server 提供哪种；对方只给 stdio 就只能走 stdio。
       * 两者的差别对写业务代码的人只剩配置里 `transport` 的取值不同。

---

## MCP 的接入方式

   * **服务来源**
       * 企业常见 MCP 服务已有现成提供：高德地图（酒店、地图、路径规划）、文件系统、时间、航班。
       * 到魔搭社区（ModelScope）的 MCP 分类里找，数量按领域分布：浏览器相关五六百、搜索一千多、交流协作五百多、开发工具三千多。
       * 挑选看访问量与调用量：访问量只有 1500、调用量为 0 的不选；调用量四百多万的国产服务算常用。
       * 时间 MCP 是典型刚需：模型训练数据有截止时间，问"现在几点"它天然不知道。

### stdio 配置

   * **mcpServers 结构**
       * 服务页给出的是一段 JSON：外层 `mcpServers`（带 s，可配多个服务），里面是"服务名 → 配置"。
       * 配置里两个键：`command` 是指令、`args` 是参数；`npx -y time-mcp` 就是启动命令。
       * `npx` 是 Node.js 的包下载管理器，负责把包拉到本地再跑起来——stdio 的"下载到本地"这一步由它完成。
       * 时间 MCP 提供六个工具：当前时间、相对时间、时间戳、某月天数、时间转换、一年中的第几周。

   * **MultiServerMCPClient 映射**
       * 先装 MCP 适配器 `langchain-mcp-adapters`，它相当于 MCP 客户端。
       * 客户端接收一个字典，内容就是 `mcpServers` 花括号里的部分，外层壳去掉。
       * 服务名、`command`、`args` 照抄；LangChain 侧要多补一个 `transport: "stdio"` 指定交互方式。
       * `await client.get_tools()` 直接返回工具对象，不必再自己定义。
       * 对比更早的手写工具：以前手写函数，现在准备客户端去拉，拉回来的就是工具。

```python
from langchain_mcp_adapters.client import MultiServerMCPClient

client = MultiServerMCPClient({
    "time-mcp": {
        "command": "npx",
        "args": ["-y", "time-mcp"],
        "transport": "stdio",
    }
})

tools = await client.get_tools()
```

### 异步工具

   * **get_tools 是异步函数**
       * 直接写 `tools = client.get_tools()` 会报 `TypeError: 'coroutine' object is not iterable`。
       * 看到 `coroutine` 字样，第一反应就是"这是协程，要 `await`"。
       * notebook 里加 `await` 后仍可能报错，这是单文件事件循环撞车，需在文件最上面处理已有循环并重启内核；另有一条过期警告，不影响运行。

   * **同步 invoke 调不了异步工具**
       * 把 MCP 工具交给 Agent 后直接问时间，报 `StructuredTool does not support sync invocation`。
       * MCP 是异步的，拉到的工具也是异步的；`await` 只能等待异步函数，不能把同步函数变成异步函数。
       * 正解是换方法名：`invoke` 是同步调用，`ainvoke` 是异步调用，且前面必须加 `await`。
       * 骨架仍是五步，只有第③步换成客户端拉取、第⑤步换成 `await agent.ainvoke(...)`。

```python
result = await agent.ainvoke(输入)
```

### HTTP 配置

   * **url 与 transport**
       * HTTP 模式配置里给的是服务地址而不是启动命令，键为 `url`。
       * 魔搭页面上的字段叫 `type`，到 LangChain 里要改名 `transport`，值填 `http`。
       * 在已有字典里加一个逗号、再加一对"服务名 + 配置"即可接入新服务。
       * `transport` 只有两种取值：`http` 走地址，`stdio` 走命令。

```python
mcp_config = {
    # ... 前面已经配好的时间 MCP ...
    "12306": {
        "url": "<服务地址>",
        "transport": "http",   # 魔搭上叫 type，到 LangChain 改成 transport
    },
}
```

   * **多服务串联**
       * 一句"帮我查下周四从北京到杭州的高铁"会先调时间 MCP 的日期工具，算出下周四的日期。
       * 再拿这个日期去调 12306 的工具，返回出发站、到达站、时间与商务座、一等座、二等座价格。
       * 工具来自哪个 MCP、哪个 server，Agent 自己按描述选择，业务代码不需要写分发。

### 消息美化

   * **pretty_print**
       * 返回的 `response` 是字典，`response["messages"]` 是本次交互的全部消息。
       * 每条 message 自带 `pretty_print()`，按角色分段打印，比直接 `print` 一坨字典清楚得多。
       * 遍历打印：`for message in response["messages"]: message.pretty_print()`。

   * **交互链的阅读**
       * 打印出来能看到完整链路：用户提问 → AI 说先查时间 → 调日期工具 → 拿到当前日期 → 推算下周四 → 调高铁工具 → 筛选后返回车次与价格。
       * 这条链就是 ReAct 循环的实际形态，工具调用与观察交替出现。

---

## 短期记忆与快照管理

   * **短期记忆（Short-term Memory）**
       * > **定义**：当前任务或当前会话的上下文，生命周期就是这一次任务或这一个会话。
       * 会话就是会话列表里的一项，里面可以有很多条消息；一个会话里的全部历史消息就是短期记忆。
       * 平常讲的"会话记忆"，就是短期记忆。
       * 它还包括任务的中间状态、执行结果等，但目前先只做会话历史。

   * **长期记忆（Long-term Memory）**
       * > **定义**：跨任务、跨会话的经验或知识，不是"永久保存"的意思。
       * 典型表现是记住用户身份：不管新建多少次会话，都知道你是做什么的，逐步汇总成人物画像，变成专属 Agent。
       * 国内模型如 DeepSeek 目前还没有实现这种能力。
       * **结论**：短期与长期的分界是生命周期，不是"临时存还是永久存"。

   * **记忆的实现位置**
       * 记忆不是模型本身的能力，一直靠外部功能实现。
       * 短期记忆靠拼接历史消息，长期记忆靠某种编码方式在对话中持续收集用户信息。
       * `Agent = model + harness`，harness 里就包含长期记忆与短期记忆。
       * 当前阶段只考虑短期记忆，够用。

### 状态与快照

   * **智能体状态（Agent State）**
       * LangChain 里会话的短期记忆都被称为 agent 的 state（状态），记忆就存在这个 state 里。
       * 默认存的就是消息，即一个 `messages` 列表：用户发的、AI 返回的，全在里面。
       * 以前要手动存库、手动查回拼接；LangChain 把这一套做成自动的。

   * **快照（Snapshot）**
       * 每与 AI 交互一次，消息进 state，state 变更，就为当前状态生成一个快照。
       * 第一次一问一答生成快照①，第二轮再生成快照②，依此类推；所有快照连起来就是完整会话历史。
       * 只为查历史，最后一个快照就够，它理论上已包含全部历史消息。
       * 保存每一次是为了时光回溯：可以从任意一轮重新发送，之前的对话从那里重来。

```text
第 1 轮：  state 变更 ──► 快照①
第 2 轮：  state 变更 ──► 快照②
第 3 轮：  state 变更 ──► 快照③
                            │
                            └──► 三张快照连起来 = 完整的会话历史
```

### Checkpointer 与 thread_id

   * **检查点存储器（Checkpointer）**
       * > **定义**：用来管理所有快照的对象；要取某个快照，找它。
       * 快照要持久化，否则一停机就没了，所以统一交给 Checkpointer 存取。
       * 数据库版相当于永久存储，不需要自己写 SQL，库已自动实现增删改查。

| 实现 | 快照存在哪 | 性质 |
| :--- | :--- | :--- |
| `InMemorySaver` | 进程内存 | 停机就没了 |
| `SqliteSaver` | SQLite 数据库 | 落盘保存 |
| `PostgresSaver` | PostgreSQL 数据库 | 落盘保存 |

   * **会话标识（thread_id）**
       * thread 就是会话，`thread_id` 是会话的唯一标识。
       * LangChain 以 `thread_id` 为 key 存该会话的所有快照，不同会话各走各的，记忆不会串。
       * 每发起一个新会话就要换一个新 id；同一个会话始终共享同一个 id。
       * 业务里的会话表 `chat_thread` 的 id 可以直接拿来当 `thread_id`，新建会话时把 id 交给 LangChain 即可。

```text
thread_id = "A"  ──►  快照① → 快照② → 快照③     （会话 A 的历史）
thread_id = "B"  ──►  快照① → 快照②              （会话 B 的历史）
                          │
                          ▼
                  Checkpointer 统一管理
```

### 接入三步

   * **导入 InMemorySaver**
       * 从 `langgraph.checkpoint.memory` 导入，不是从 LangChain 导。
       * 原因是 LangChain 底层基于 LangGraph，记忆这件事本身由 LangGraph 实现。

   * **绑定 checkpointer**
       * `create_agent` 以前只接收模型与工具，现在多传第三个参数 `checkpointer=InMemorySaver()`。
       * 加完这一行 Agent 就自带记忆，不需要写任何数据库增删改查。

   * **调用时传 config**
       * 每次请求都要带 `config = {"configurable": {"thread_id": "1"}}`。
       * `configurable` 属性名与 `thread_id` 键名都不能变，可变的只有 id 的值；值自己起，但要保证每个会话唯一。
       * `invoke` 与 `stream_events` 都能传，流式写法只是多一个 `version="v3"`。
       * 实测：先问"12 个苹果分给 4 个人"，再问"如果是 3 个人呢"，它直接接着上一轮的 12 个苹果算，说明记忆生效。
       * > **易错点**：`stream` 里拿到的 `chunk` 是消息片段，不是完整 message，不能对它调 `pretty_print()`，正常 `print` 即可。
