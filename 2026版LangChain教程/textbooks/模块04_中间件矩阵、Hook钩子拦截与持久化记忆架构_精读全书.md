# 2026版LangChain教程，langchain快速入门， Agent智能体rag项目实战·中间件矩阵、Hook钩子拦截与持久化记忆架构（第 4 册 / 共 5 册）

> **所属课程**：2026版LangChain教程，langchain快速入门， Agent智能体rag项目实战  
> **本册内容**：中间件矩阵、Hook钩子拦截与持久化记忆架构  
> **覆盖范围**：P64-P66 ~ P92-P97（共 34 讲 / 6 章 / 315 分钟音频）  
> **分册说明**：全书按内容分 5 册；本册为第 4 册  
> **整编说明**：正文逐字保留各块模块长文，仅补导读、目录与章间过渡；原长文同步保留于 `articles/` 供定向查阅。

---

## 导读与全景目录

1. 中间件机制全景与人机协同、自动摘要（P64-P66）
2. 隐私脱敏、任务清单与模型调用限制中间件（P67-P69）
3. 容灾重试、上下文编辑与 Node-style 钩子基础（P70-P80）
4. 钩子函数进阶：类式中间件、拦截跳转与包裹执行机制（P81-P85）
5. 记忆机制原理、短期内存持久化与云端数据库基础设施（P86-P91）
6. PostgreSQL持久化存储、消息治理与长期记忆架构（P92-P97）

---

## 中间件机制全景与人机协同、自动摘要
> 对应块：BLK15 | 覆盖分集：P64-P66

在完成智能体（Agent）核心构建机制的学习后，我们正式进入为智能体深度赋能的进阶架构。我们在构建智能体调用 `create_agent` 时，接触过几个核心要素：模型（Model）充当决策中枢的大脑，工具（Tools）作为执行具体任务的手脚赋予其外部交互能力，系统提示词（System Prompt）确立其身份角色与认知约束。而在这一组体系中，还有一个承前启后的关键参数——中间件（Middleware）。

中间件在智能体体系中充当着神经中枢的角色。它是 LangChain 1.x 架构中最重要、最具工程价值的王牌特性。它能够在智能体生命周期的关键流转节点进行拦截、控制与动态增强。理解并掌握中间件机制，是将实验性智能体推向高可用、高安全工业级生产环境的必经之路。

---

### 智能体中枢架构与中间件演进

在没有中间件的传统架构中，智能体的运行流程是线性的闭环：用户提出请求，系统拼接提示词送入大模型，大模型判断是否需要调用工具，若需要则触发工具并接收返回结果，在模型与工具之间可能产生多轮交替往复的循环，最终由模型组织自然语言输出给用户。

```text
+-------------------------------------------------------------------+
|                        传统无中间件架构流程                       |
|                                                                   |
|  [ 用户输入 ]                                                     |
|       |                                                           |
|       v                                                           |
|  [ 拼接提示词 ] --> [ 大模型 Model ] <==== 循环调用 ====> [ 工具 Tools ]
|                           |                                       |
|                           v                                       |
|                      [ 用户响应 ]                                 |
+-------------------------------------------------------------------+
```

上图展示了传统无中间件架构的运行拓扑。核心交互紧密耦合在模型与工具之间，流程单一直白。但在复杂的工业级业务场景下，这种裸奔的调用方式会面临巨大的维护瓶颈。

#### 钩子函数体系与生命周期拦截

中间件本质上是智能体执行流程中一系列钩子函数（Hook Functions）的封装集合。所谓钩子函数，是框架在核心执行流程的特定生命周期节点暴露出的标准化扩展接口。当程序流转至特定节点时，挂载在对应位置的钩子函数便会自动触发。开发者无需侵入框架底层，即可实现对运行细节的高度定制与精细控制。

在引入中间件架构后，LangChain 在智能体外层及核心调用周围布置了多层拦截网：

```text
+---------------------------------------------------------------------------+
|                          中间件多层拦截架构拓扑                           |
|                                                                           |
|   before_agent                                               after_agent  |
|        |                                                          ^       |
|        v                                                          |       |
|   +-------------------------------------------------------------------+   |
|   |                          Agent 执行边界                           |   |
|   |                                                                   |   |
|   |   before_model                                      after_model   |   |
|   |        |                                                 ^        |   |
|   |        v                                                 |        |   |
|   |   +-------------------------+       +-------------------------+   |   |
|   |   |     wrap_model_call     |       |     wrap_tool_call      |   |   |
|   |   |  +-------------------+  |       |  +-------------------+  |   |   |
|   |   |  |   大模型 Model    |  | <===> |  |    工具集 Tools   |  |   |   |
|   |   |  +-------------------+  |       |  +-------------------+  |   |   |
|   |   +-------------------------+       +-------------------------+   |   |
|   +-------------------------------------------------------------------+   |
+---------------------------------------------------------------------------+
```

上图展示了中间件在智能体生命周期中所构筑的完整拦截矩阵。这一体系划分出两个层级：
- 节点触发型钩子：位于流程特定节点前后，包括进入智能体前触发的 `before_agent`、退出智能体时触发的 `after_agent`，以及请求模型前执行的 `before_model` 和模型返回后触发的 `after_model`；
- 环绕包装型钩子：直接包裹执行体，包括环绕模型调用的 `wrap_model_call` 与环绕工具执行的 `wrap_tool_call`。这种包装机制允许在调用真正发生前拦截并修改输入，在调用完成后校验或替换返回值。

> **定义**：中间件是面向智能体生命周期的标准化管控模式。它将一系列生命周期钩子函数有机组合，在不改变智能体主体业务逻辑的前提下，实现非侵入式的状态监测、流程干预与能力注入。

#### 横切关注点分离与工程解耦价值

为什么必须引入中间件？我们可以考察实际生产环境中高度常见的七大典型场景：

- 动态模型路由：根据用户输入任务的复杂度，动态决定调用轻量模型还是全尺寸模型。例如简单常识问答路由至 1.5B 或 7B 的轻量小模型以降低开销，复杂逻辑推理无缝升级至 32B 乃至满血大模型；
- 工具权限隔离：根据用户身份权限细粒度限制可调用工具的集合，防止普通用户触发敏感操作或模型因无关工具过多产生幻觉；
- 容错与高可用保障：当外部工具服务网络波动报错时，自动进行重试策略处理，或直接向大模型返回结构化的兜底（Fallback）预案，避免全流程异常崩溃；
- 动态提示词注入：在每次进入模型前，根据当前执行上下文、环境参数或用户画像，动态向系统提示词中追加特定的运行指令；
- 执行链审计追踪：结构化记录从用户提问、多轮工具调用到最终生成的完整决策轨迹与性能日志，满足监管与排障需求；
- 敏感数据风控合规：对输入输出中的手机号、电子邮箱、银行卡号等敏感数据进行识别，执行星号遮蔽阻断或哈希化脱敏替换；
- 高危操作人机协同：在智能体即将调用发邮件、扣费、修改数据库或调用人事财务接口等关键危险工具前，挂起执行，等待人工审批放行或修改参数后方可继续。

如果将上述逻辑直接揉进智能体的主执行流程中，系统会迅速面临四大灾难：
- 主流程代码极度臃肿：业务核心代码被大量的鉴权、日志、风控、重试包裹，代码可读性与内聚性瞬间瓦解；
- 横切逻辑无法复用：这些诉求本质上属于系统级的横切需求（Cross-Cutting Concerns）。正如后端开发中 Spring 框架倡导的面向切面编程（AOP）思想，当工程中存在数十个智能体时，硬编码会导致海量重复冗余；
- 拦截控制粒度粗糙：缺乏统一规范的挂载点，修改拦截时序需要反复在主干流程中穿插手写 `if-else`，极易破坏状态机流转；
- 长期维护成本失控：一旦安全规范或重试策略发生变更，必须逐一修改所有智能体源码，引发严重的回归测试风险。

> **结论**：中间件的核心价值在于解耦。它将与核心业务无关、但与执行过程强相关的横切逻辑从智能体主干流程中彻底剥离，使智能体聚焦于“查天气、改数据、解问题”等纯粹业务，而由中间件构筑起标准化、可插拔的运行防护网。

---

### 内置中间件全景与功能分类矩阵

在 LangChain 架构设计中，中间件划分为自定义中间件与内置中间件两大体系。内置中间件中，部分是特定模型供应商独有的定制化实现，而更通用、更具工业普适性的，是与底层模型提供商解耦的通用中间件。

LangChain 官方共提供了 16 个与供应商解耦的内置中间件。我们无需死记硬背每个名称，但必须从功能维度建立结构化的分类认知。这一分类矩阵本身就是智能体生产化治理的完整全景图。

```text
+-----------------------------------------------------------------------------------+
|                         LangChain 内置中间件核心六大分类矩阵                      |
+--------------------------+--------------------------------------------------------+
| 类别                     | 核心目标与解决的工程痛点                               |
+--------------------------+--------------------------------------------------------+
| 成本与资源控制类         | 控配额、降成本、防死循环，解决智能体“太贵太能跑”的失控 |
| 稳定性与容错保障类       | 失败重试、备用降级，保障全链路高可用与鲁棒性           |
| 安全与合规风控类         | 敏感信息脱敏、高危操作人工干预，解决越权与合规风险     |
| 决策增强与智能编排       | 任务拆解分步、工具子集精简、子智能体调度               |
| 执行能力扩展类           | 赋予系统级环境操作能力，如 Shell、文件系统读写         |
| 开发调试与测试辅助类     | 模拟工具行为、调试超长对话压缩与上下文裁剪             |
+--------------------------+--------------------------------------------------------+
```

#### 成本控制与运行保障

大模型技术在企业内部的落地通常经历三个阶段：初期团队不了解大模型；随后进入探索期，企业为各业务线分发大量 Token 额度自由试用；最终进入第三阶段，企业赫然发现每个月的 Token 账单呈现指数级爆炸，消耗成本甚至显著超出了节省的人力成本。许多大型科技公司（如微软）都曾专门出台策略收紧内部研发环境的 Token 消耗上限。智能体具有多轮自迭代循环的特性，极易发生递归死循环或上下文冗余膨胀，因此成本管控是第一生命线。

- 成本与资源控制：
  - 限制模型调用总次数（Model Call Limit），防止提示词漏洞引发的模型自言自语死循环；
  - 限制工具调用总次数（Tool Call Limit），杜绝因工具参数错误导致的反复无效轮询；
  - 历史会话自动摘要（Summarization），在长对话窗口逼近饱和时，自动提炼压缩早前对话；
  - 上下文修剪与工具调用痕迹清理，只保留决策结论，清除冗长生涩的中间状态与日志。
- 稳定性与容错保障：
  - 针对大模型服务偶尔出现的超时、限流（Rate Limit）或服务中断，中间件支持配置备用降级模型（Fallback Model），当主模型崩溃时无缝切换；
  - 模型及工具调用失败的自动重试策略，配置指数退避算法，化解偶发网络抖动。

#### 安全合规与能力拓展

- 安全合规与风控：
  - 人在环（Human-in-the-loop）：智能体面对真实外部世界时具有破坏性潜能。在涉及发送正式邮件、删除数据库表项、调用财务打款或修改人事考勤时，中间件强制中断执行，由人工审查通过后方可下发；
  - 个人敏感信息（PII）拦截脱敏：自动扫描输入与输出文本，将邮箱、电话等敏感字段替换为马赛克星号或哈希指纹，防止合规违规与数据泄漏。
- 决策增强与智能编排：
  - 引入待办事项清单（To-do List）规划机制，将极其宽泛复杂的宏观目标自动拆解为有序步骤；
  - 针对挂载数十个工具导致的参数注意力涣散问题，利用轻量级子模型先筛选出最相关的 2~3 个工具，再递交主模型决策；
  - 动态派发子智能体，实现分而治之的多智能体层级调度。
- 执行能力与测试辅助：
  - 扩展宿主环境操作能力，包括调用系统 Shell 指令、搜索检索本地磁盘文件等；
  - 研发调试阶段提供工具模拟（Mocking）机制，在断网或无实际外部接口时加速自动化集成测试。

---

### SummarizationMiddleware 上下文动态压缩机制

在长生命周期的对话与任务中，多轮历史消息会无限制线性堆叠。当输入序列接近大模型上下文窗口边界时，不仅单次推理成本极为高昂，模型对早前指令的遵循能力也会发生严重退化（Attention Lost in the Middle）。

`SummarizationMiddleware` 是应对该问题的标准方案。其核心原理是：当历史消息达到设定的触发阈值时，自动调用指定的大模型对历史累积的会话进行浓缩提炼，生成紧凑的摘要文本，将其包装为一条 `HumanMessage` 强行置换到消息列表的最前端，同时保留最近数条未经压缩的原始交互，实现上下文的滑动窗口式平滑过渡。

```text
+-------------------------------------------------------------------------------+
|                       SummarizationMiddleware 上下文压缩流                    |
|                                                                               |
|  压缩前消息队列：                                                             |
|  [Msg 1] -> [Msg 2] -> [Msg 3] -> [Msg 4] -> [Msg 5] -> [Msg 6] (触发阈值)   |
|     \_________________________________/           \___________/               |
|                      |                                  |                     |
|           大模型提炼压缩为摘要文本                      保持原始明文          |
|                      |                                  |                     |
|                      v                                  v                     |
|  压缩后消息队列：                                                             |
|  [HumanMessage: "历史摘要内容"] -------------> [Msg 5] -> [Msg 6]            |
+-------------------------------------------------------------------------------+
```

上图清晰地展示了中间件对消息队列的重构机制。历史消息中较早的片段被归纳进单一的摘要消息中，最新消息被完整保留，从而以极小的 Token 开销维系长对话的连贯性。

#### 核心参数与触发策略

初始化 `SummarizationMiddleware` 时涉及的关键参数如下：

- `model`：用于执行文本摘要任务的模型。可以传入具体模型实例，亦可传入模型名称字符串（底层通过 `init_chat_model` 自动实例化）。在工程实践中，推荐在此处配置较主模型更便宜、响应更快的轻量模型专门负责摘要；
- `trigger`：触发条件列表。支持同时传入多个条件元组，任一条件满足即触发压缩动作。支持的三种策略：
  - `("tokens", 100)`：历史消息占用的 Token 总量达到指定阈值时触发；
  - `("messages", 6)`：历史消息的总条目数达到设定数值时触发；
  - `("fraction", 0.001)`：基于模型上下文容量的动态百分比触发（取值在 0~1 之间的小数）；
- `keep`：压缩触发后，尾部保留的未压缩原始消息规格。通常选择指定条目数，如 `("messages", 2)`，确保最近一轮的核心上下文不失真；
- `summary_prompt`：自定义提炼摘要所用的提示词模板。必须显式包含 `{messages}` 占位符，以便框架将待浓缩的消息流注入其中；
- `token_counter`：统计 Token 数量的计算函数，通常保持框架默认实现即可。

#### 基础摘要压缩实战与模型元数据配置

在编写实战代码时，我们需要关注模型元数据与比例触发机制的协同配合。

```python
from langchain.agents import create_agent
from langchain.agents.middleware import SummarizationMiddleware
from langchain_core.messages import HumanMessage, AIMessage
from langchain_openai import ChatOpenAI

# 1. 声明专用于执行摘要任务的模型实例
# 显式配置 profile 元数据以支持 fraction 计算
summary_model = ChatOpenAI(
    model="gpt-4o-mini",
    profile={"max_input_tokens": 128_000}
)

# 2. 组装摘要中间件
summarizer = SummarizationMiddleware(
    model=summary_model,
    trigger=[
        ("tokens", 100),
        ("messages", 6),
        ("fraction", 0.001)
    ],
    keep=("messages", 2)
)

# 3. 构建智能体，主执行模型可指定其他型号
agent = create_agent(
    model="deepseek:v4-flash",
    middleware=[summarizer]
)

# 4. 模拟包含较多历史交互的消息列表
history_messages = [
    HumanMessage(content="你好，我想制定一个全年的体能训练计划。"),
    AIMessage(content="没问题，请问你目前的体能水平和每周锻炼频率是怎样的？"),
    HumanMessage(content="我目前能跑 5 公里，每周能保证训练 3 次左右。"),
    AIMessage(content="收到，建议前三个月以基础有氧耐力和核心稳定性为主。"),
    HumanMessage(content="好的，那饮食方面需要配合什么补剂吗？"),
    AIMessage(content="初期保证蛋白质摄入即可，暂不需要高阶补剂。"),
    HumanMessage(content="明白，我们下周一正式开始第一阶段！"),
    AIMessage(content="你高兴得太早了，周一的训练量可不轻。"),
    HumanMessage(content="呵呵你什么意思，我底子可扎实了。")
]

# 5. 触发执行并打印观察重构后的消息结构
response = agent.invoke({"messages": history_messages})

for msg in response["messages"]:
    msg.pretty_print()
```

> **易错点**：当在 `trigger` 中启用了 `("fraction", 0.001)` 比例触发模式时，框架内部计算触发阈值的数学逻辑为：`触发阈值 = max_input_tokens * fraction`。若所使用的模型类实例底层未内置声明 `max_input_tokens` 字段，调用会直接崩溃抛出 `chat model profile ... max_input_tokens not specified` 错误。此时必须在模型构建时显式指定 `profile={"max_input_tokens": 128_000}`，为乘法计算提供基准数值。

在上述代码执行完毕后，分析终端输出的消息序列，可以看到极其清晰的结构演变：
- 开头第一条消息被替换为全新的 `HumanMessage`，内容为提取出的历史会话综合摘要；
- 紧随其后的是根据 `keep=("messages", 2)` 规则保留的倒数第二轮对话：“你高兴得太早了”与“呵呵你什么意思”；
- 列表末端则是智能体根据精炼上下文最新生成的最终回复 `AIMessage`。中间冗长的前期对话全部被平滑压缩。

#### 中文摘要提示词定制与格式控制

默认情况下，`SummarizationMiddleware` 内置生成的摘要前缀包含英文模板提示词，在纯中文应用场景下略显割裂。我们可以通过传入 `summary_prompt` 实现完全受控的个性化提炼规范：

```python
# 定义纯中文的紧凑摘要引导模板
custom_summary_prompt = "请对以下历史对话内容进行精炼摘要，提取用户的核心背景信息与核心共识。历史消息列表如下：\n{messages}"

summarizer_custom = SummarizationMiddleware(
    model=summary_model,
    trigger=[
        ("messages", 6)
    ],
    keep=("messages", 2),
    summary_prompt=custom_summary_prompt
)

agent_custom = create_agent(
    model="deepseek:v4-flash",
    middleware=[summarizer_custom]
)

response_custom = agent_custom.invoke({"messages": history_messages})

for msg in response_custom["messages"]:
    msg.pretty_print()
```

> **注意**：编写自定义 `summary_prompt` 时，必须保证模板字符串中严格包含 `{messages}` 占位符。中间件在触发提炼时，会将待裁剪的消息列表序列化后自动填入该插槽中；遗漏此占位符会导致框架解析失败。

引入自定义中文提示词后，首条 `HumanMessage` 的内容将完全遵循中文格式规范，语言风格统一且摘要精准度显著提升。

---

### HumanInTheLoopMiddleware 人机协同拦截机制

在多智能体与自主代理的实际运行中，模型生成的调用规划不一定完全可靠，特别是面对外部环境写操作（如写数据库、发邮件、执行金融交易）时，完全放任智能体自主运行蕴含着巨大的工程风险。

`HumanInTheLoopMiddleware`（人在环中间件，简称 HITL）是应对高风险操作的权威解决方案。“环”即模型调用工具的交替反馈循环（Loop）。该中间件的职能是在智能体即将真正触发工具调用的临界点强行挂起，将控制权交还给外部调用者，等待人工确认、拒绝甚至直接篡改调用参数后，再决定是否放行流程继续流转。

```text
+---------------------------------------------------------------------------------------+
|                       Human-in-the-loop (人在环) 两阶段流转时序                       |
|                                                                                       |
|  [阶段一：触发与挂起]                                                                 |
|  用户输入 ----> Agent 推理 ----> 命中敏感工具拦截 ----> 流程挂起中断 (Interrupt)      |
|                                                              |                        |
|                                                              v                        |
|                                                    抛出 action_requests 供人工审阅    |
|                                                                                       |
|  [阶段二：决策与接续]                                                                 |
|  人工审阅 ----> 组装决策 (Approve / Reject / Edit)                                    |
|                      |                                                                |
|                      v                                                                |
|  Command(resume=...) + 相同 thread_id ----> 恢复执行 ----> 工具运行 ----> 最终响应    |
+---------------------------------------------------------------------------------------+
```

上图展示了人在环机制的两阶段流转闭环。整个机制依赖中断与恢复状态机协同，人工决策不仅可以被动审批，还能主动重塑工具入参。

#### 决策行为模型与中断策略配置

面对挂起的工具执行请求，人工干预具备三种标准的决策行为（Decisions）：
- `approve`：完全批准放行，工具按照模型所生成的原始参数直接执行；
- `reject`：驳回拒绝，该工具将不会被执行，流程会接收到拦截拒绝的反馈；
- `edit`：编辑篡改。人工介入时如果发现模型规划的参数有误或存在偏差，无需驳回重来，而是直接在决策载荷中提供修改后的工具参数（如将原本查询“北京市”强制重写为“上海市”），放行后工具将严格采用人工修正后的参数运行。

配置人在环拦截规则时，主要涉及两个核心参数：
- `description_prefix`：全局中断描述前缀。当被挂起的工具没有定制独立的描述信息时，默认采用该前缀进行状态标识；
- `interrupt_on`：中断策略字典，以具体工具名称为 Key，精准配置该工具的拦截策略。每个工具可以赋值为三种形态之一：
  - 设为 `True`：该工具触发时必定中断挂起，且默认开放 `approve`、`reject`、`edit` 全部三种人工决策能力；
  - 设为 `False`：该工具不中断，直接放行执行；
  - 设为配置字典（符合 `InterruptOnConfig` 规范）：进行细粒度控制。字典支持配置 `allowed_decisions` 列表（例如限定只允许 `["approve", "reject"]`，彻底禁止人工 `edit` 参数）以及该工具独享的 `description` 描述文本。

> **规则**：描述文本具有就近覆盖原则。若工具在 `interrupt_on` 的配置字典中显式定义了 `description`，则以该工具私有的描述为准；若未单独声明，则统一继承外层的 `description_prefix`。

#### 阶段一：工具拦截挂起与短期会话绑定

人在环本质上是一个异步的两阶段调用：第一阶段触发模型并触发拦截退出，第二阶段人工介入后必须精准恢复到先前的现场接续执行。

如果两次调用之间彼此孤立，智能体就会丧失上下文现场。因此，智能体必须引入检查点检查器（Checkpointer，如内存级别的 `InMemorySaver`），并在调用时通过 `config={"configurable": {"thread_id": "xxx"}}` 绑定会话线程。同一个 `thread_id` 确保了两阶段处于同一状态快照生命周期内。

```python
from langchain.agents import create_agent
from langchain.agents.middleware import HumanInTheLoopMiddleware
from langchain_core.tools import tool
from langchain_openai import ChatOpenAI
from langgraph.checkpoint.memory import InMemorySaver
from rich import print as rprint

# 1. 定义四个功能各异的测试工具
@tool
def get_weather(city: str, is_forecast: bool = False) -> str:
    """查询指定城市的天气状况"""
    forecast_str = "未来趋势良好" if is_forecast else "当前无预报"
    return f"{city}天气：晴朗，气温 25℃，{forecast_str}"

@tool
def get_news(topic: str) -> str:
    """获取指定主题的最新新闻"""
    return f"关于 {topic} 的最新头条：技术革命正在深入推进。"

@tool
def read_email_tool(folder: str) -> str:
    """读取指定邮箱文件夹中的未读邮件"""
    return f"邮箱 {folder} 中共有 2 封关于项目进度的未读邮件。"

@tool
def send_email_tool(recipient: str, subject: str, content: str) -> str:
    """发送正式邮件给指定收件人"""
    return f"邮件已成功发送给 {recipient}，主题：{subject}。"

# 2. 声明短期记忆状态保存器
checkpointer = InMemorySaver()

# 3. 编排差异化的人机协同拦截规则
hitl_middleware = HumanInTheLoopMiddleware(
    description_prefix="系统拦截：工具调用已挂起",
    interrupt_on={
        "get_weather": True,        # 中断，允许全部决策 (approve/reject/edit)
        "get_news": True,           # 中断，允许全部决策
        "read_email_tool": False,   # 放行，无风险读操作不中断
        "send_email_tool": {        # 精细化管控：高危写操作禁止编辑参数，定制描述
            "allowed_decisions": ["approve", "reject"],
            "description": "高危操作：向外部发送邮件必须经由人工授权确认"
        }
    }
)

# 4. 构建绑定检查点与中间件的智能体
model = ChatOpenAI(model="gpt-4o-mini")
agent = create_agent(
    model=model,
    tools=[get_weather, get_news, read_email_tool, send_email_tool],
    checkpointer=checkpointer,
    middleware=[hitl_middleware]
)

# 5. 指定唯一的会话线程 ID
session_config = {"configurable": {"thread_id": "session_hitl_001"}}

# 6. 发起复合多意图任务，同时触碰上述四个工具
prompt = "帮我查下北京天气，看看最新的科技新闻，查下收件箱邮件，然后给 manager@corp.com 发封邮件说明进度。"

phase1_response = agent.invoke(
    {"messages": [("user", prompt)]},
    config=session_config
)

# 观察阶段一返回的响应载荷
rprint(phase1_response)
```

在执行完阶段一的调用后，解析返回的响应字典：
- 在返回的 `AIMessage` 中，可以看到大模型已经成功将意图分解为四个工具调用（`tool_calls`），包含了各自的调用入参；
- 紧随其后的执行链条被硬性阻断，并没有立刻产生对应的 `ToolMessage`；
- 响应根节点中暴露出了 `interrupts` 数组：
  - `get_weather` 和 `get_news` 对应的挂起描述为全局前缀 `系统拦截：工具调用已挂起`；
  - `send_email_tool` 独立呈现了定制的描述文案 `高危操作：向外部发送邮件必须经由人工授权确认`；
  - `read_email_tool` 由于规则设定为 `False`，绝不出现在挂起中断列表中，保持平稳放行。

#### 阶段二：决策注入与 Command 状态恢复执行

智能体在完成阶段一挂起后，其执行状态已被安全保存在 `checkpointer` 的快照库中。此时系统可以等待外部 Web 管理后台、审批机器人或命令行交互获取操作员的审查指令。

在阶段二中，我们需要利用 LangGraph 的 `Command` 指令机制，将组装好的决策数据（Decisions）注入 `resume` 载荷中，带着**相同的 `thread_id`** 再次调用智能体，实现现场恢复。

```python
from langgraph.types import Command

# 1. 从阶段一响应中防御性提取待处理的挂起请求
interrupt_data = phase1_response.get("interrupts", [])
if not interrupt_data:
    raise RuntimeError("当前流程未发生任何挂起中断，无法执行决策恢复。")

# 提取挂起的动作列表
action_requests = interrupt_data[0]["value"]["action_requests"]

# 2. 模拟人工审查员根据不同工具定制生成的决策列表
decisions = []

for req in action_requests:
    tool_name = req["name"]
    
    if tool_name == "get_weather":
        # 人工干预：不直接放行，而是通过 edit 篡改模型原始规划的城市与参数
        decisions.append({
            "type": "edit",
            "edited_action": {
                "name": "get_weather",
                "args": {
                    "city": "上海市",
                    "is_forecast": True
                }
            }
        })
    elif tool_name == "get_news":
        # 批准放行
        decisions.append({
            "type": "approve"
        })
    elif tool_name == "send_email_tool":
        # 批准发送邮件
        decisions.append({
            "type": "approve"
        })

# 3. 构造 Command 恢复指令并恢复执行
resume_command = Command(resume={"decisions": decisions})

phase2_response = agent.invoke(
    resume_command,
    config=session_config  # 必须传入与阶段一完全一致的 thread_id
)

# 4. 打印恢复流转后的最终消息链条
for msg in phase2_response["messages"]:
    msg.pretty_print()
```

> **易错点**：构建 `edit` 决策字典时，所携带的嵌套配置键名必须严格书写为 `edited_action`（包含过去分词后缀 `-ed`）。若误写为 `edit_action`，中间件底层的反序列化解析会抛出字段缺失异常。同时，恢复调用时所传的 `config` 字典必须与第一阶段的 `thread_id` 严格保持一致，否则框架会误认为开启了全新空白会话，导致找不到中断检查点而报错。

在阶段二恢复调用顺利执行后，我们可以从输出日志中见证整个编排闭环：
- `get_weather` 工具实际接收并执行的入参被成功重写，查询结果返回的是“上海市天气：晴朗，气温 25℃，未来趋势良好”，证明 `edit` 行为完全覆盖了模型最初生成的“北京市”计划；
- `get_news` 与 `send_email_tool` 得到审批放行，分别输出了对应工具的执行结果；
- `read_email_tool` 此前已由模型静默安全放行；
- 最终，大模型统一汇聚所有工具执行结果，生成了一份条理清晰的最终总结 `AIMessage` 呈现给终端用户。人机协同拦截机制在确保绝对工程安全的同时，实现了业务流程的顺畅运转。

> **承前启后**：上一节讲完「中间件机制全景与人机协同、自动摘要」，下一节接着讲「隐私脱敏、任务清单与模型调用限制中间件」。

---

## 隐私脱敏、任务清单与模型调用限制中间件
> 对应块：BLK16 | 覆盖分集：P67-P69

在构建基于大语言模型的智能体时，如果直接将原始用户输入抛给模型，或者任由智能体在多步执行中自主发散，系统很快就会遭遇三大工程痛点：用户输入中夹带的敏感身份信息外泄至外部模型提供商；面对长链路、多步骤的复杂任务时，模型执行到中途遗忘初始目标或在报错后草率跳过验证；以及智能体陷入死循环导致调用次数与 API 成本彻底失控。

LangChain 针对这些工程落地中的真实风险，提供了一系列供应商中立的内置中间件（Middleware）。通过在 Agent 的调用生命周期前后注入拦截与编排逻辑，系统能够在不侵入业务核心代码的前提下，实现隐私合规脱敏、任务规划追踪与调用频次硬性熔断。

---

### 个人身份信息保护与脱敏中间件

在调用模型以及与外部环境交互的过程中，用户的提示词或上下文字段经常包含各类个人隐私与敏感数据。若未经处理直接发送给模型，将带来严重的合规风险与隐私泄露隐患。`PIIMiddleware` 正是用于拦截和保护个人身份信息（Personally Identifiable Information，PII）的专用中间件。

#### 敏感数据类型与核心脱敏策略

`PIIMiddleware` 的核心职责是在数据进入大模型之前完成特征捕获与策略替换。中间件内置了对常见敏感数据类型的识别能力，包括电子邮箱（email）、信用卡号（credit_card）、网页链接（URL）、MAC 地址（mac_address）、IP 地址（IP）以及各类 API 密钥（API key）。

针对检测到的敏感数据，中间件提供了四种截然不同的处置策略，分别对应不同的生产安全等级与业务交互场景：

| 保护策略 (`strategy`) | 处理机理 | 典型输出表现 | 适用业务场景 |
| :--- | :--- | :--- | :--- |
| `redact` | 整段抹除替换 | 替换为固定占位字符串，如 `redacted-email`、`redacted-url` | 日志持久化清洗、合规审计要求、对外公开输出时的完全隐藏 |
| `mask` | 掩码遮蔽保留后缀 | 核心字符替换为星号（`***`），仅保留末尾数位 | 客户端服务界面、前端回显，兼顾用户身份辨识与防窥探 |
| `hash` | 单向哈希计算 | 经不可逆哈希算法输出固定长度的散列值 | 调试分析、匿名追踪、多样本频次统计（同一原值哈希一致） |
| `block` | 拦截阻断抛异常 | 检测到敏感项后直接抛出运行时异常，中断执行 | 金融、军工等极高隐私安全场景，实行零容忍拦截 |

```text
+-------------------+      +------------------------------------------------------+
| 用户原始输入文本  | ---> | PIIMiddleware 隐私检测与拦截管线                     |
+-------------------+      +------------------------------------------------------+
                                      |
       +------------------------------+------------------------------+
       |                              |                              |
       v                              v                              v
[ redact 策略 ]                [ mask 策略 ]                  [ block 策略 ]
替换为专用占位符               部分字符置换为掩码             立即抛出中断异常
"redacted-email"               "4111********1111"             raise Exception
       |                              |                              |
       +------------------------------+------------------------------+
                                      |
                                      v
                        +----------------------------+
                        | 脱敏后安全文本发送至大模型 |
                        +----------------------------+
```
> 用户输入经过 `PIIMiddleware` 处理后，依据各字段配置的策略进行分流脱敏，消除敏感信息后再提交给底层大模型。

在中间件的触发时机配置上，核心参数包括 `apply_to_input`（在模型调用前检测）、模型调用后检测以及工具调用输出检测等。在绝大多数业务场景中，规范的做法是将 `apply_to_input` 设置为 `True`，其余检测时机保持 `False`。

> **设计考量**：引入该中间件的首要目的，就是杜绝敏感信息被发送到外部模型服务商的服务器。既然在进入模型之前已经完成了彻底的脱敏或阻断，大模型接收到的本身就是脱敏后的文本，那么在后续的模型响应和工具返回环节再次重复检测就失去了工程意义，反而会空耗算力。

#### 内置检测器的多策略装配

在常规场景下，可以直接利用中间件内置的识别器，针对不同的字段维度装配异构策略。在创建 Agent 时，`middleware` 参数接收一个中间件实例列表，针对邮箱选用 `redact`，针对银行卡与 MAC 地址选用 `mask`，针对 URL 选用 `hash`，而针对 IP 地址则选用严格的 `block`。

```python
from langchain.agents import create_agent
from langchain.messages import HumanMessage
from langchain.middleware import PIIMiddleware

# 装配具备复合脱敏策略的中间件列表
pii_middlewares = [
    PIIMiddleware("email", strategy="redact", apply_to_input=True),
    PIIMiddleware("credit_card", strategy="mask", apply_to_input=True),
    PIIMiddleware("URL", strategy="hash", apply_to_input=True),
    PIIMiddleware("mac_address", strategy="mask", apply_to_input=True),
    PIIMiddleware("IP", strategy="block", apply_to_input=True),
]

agent = create_agent(
    model=model,
    middleware=pii_middlewares
)
```

当向该 Agent 传入包含邮箱、银行卡、URL 与 MAC 地址的常规请求时，输入内容在抵达模型之前便完成了就地转换：

```python
# 构造包含多种敏感实体的内容
query = (
    "请协助处理账单：用户邮箱为 test_user@example.com，"
    "绑定卡号为 6222021000001234，"
    "访问来源于网址 https://secure.payment.com/gateway，"
    "客户端物理地址为 00:1A:2B:3C:4D:5E。"
)

response = agent.invoke({"messages": [HumanMessage(content=query)]})

for message in response["messages"]:
    message.pretty_print()
```

控制台打印出的输入消息中，`test_user@example.com` 被替换为标准字符串 `redacted-email`；银行卡号前中段被替换为星号并仅保留末四位；URL 被整体转换为哈希编码；MAC 地址的关键字段同样被掩码覆盖。

对于设置了 `block` 策略的 IP 地址，一旦输入中出现 IP 特征，中间件将立即中断管线并抛出异常。在工程落地时，需要使用异常处理结构进行防御性包裹：

```python
ip_query = "上报异常服务器节点，IP 地址为 192.168.1.100。"

try:
    response = agent.invoke({"messages": [HumanMessage(content=ip_query)]})
except Exception as e:
    print(f"检测到敏感 IP 信息，流程已阻断并抛出异常: {e}")
```

> **易错点**：`block` 策略的触发是在模型请求发起前的同步阶段。未捕获该异常会导致整个请求任务链挂起失败，因此面向不受信输入源使用 `block` 时，必须显式编写外部 `try...except` 容错逻辑。

#### 自定义敏感信息检测器扩展

当企业业务涉及非通用的特定敏感信息（如中国大陆 11 位手机号、企业自研系统的专用 API Key 格式）时，内置检测器无法直接覆盖。此时需要编写自定义检测函数并注入 `PIIMiddleware`。

自定义检测函数必须遵循严格的契约接口：接收待检测的字符串文本 `content`，返回一个包含所有匹配项详细位置信息的迭代器或列表。列表中每一个元素均为具备 `text`、`start` 和 `end` 三个标准字段的字典：

- `text`：从原文本中提取出的完整敏感子串；
- `start`：该子串在原文本中的起始字符索引；
- `end`：该子串在原文本中的结束字符索引。

```python
import re

def detect_phone_number(content: str):
    """检测 11 位大陆手机号的自定义匹配器"""
    pattern = r"\b1[3-9]\d{9}\b"
    results = []
    for match in re.finditer(pattern, content):
        results.append({
            "text": match.group(),
            "start": match.start(),
            "end": match.end()
        })
    return results

def detect_custom_api_key(content: str):
    """检测形如 sk-[a-zA-Z0-9]+ 的 API Key"""
    pattern = r"sk-[a-zA-Z0-9]{16,}"
    results = []
    for match in re.finditer(pattern, content):
        results.append({
            "text": match.group(),
            "start": match.start(),
            "end": match.end()
        })
    return results
```

自定义检测函数编写完成后，可直接在初始化中间件时通过 `detector` 参数完成绑定：

```python
custom_pii_middlewares = [
    PIIMiddleware(detector=detect_phone_number, strategy="mask", apply_to_input=True),
    PIIMiddleware(detector=detect_custom_api_key, strategy="hash", apply_to_input=True),
]

secure_agent = create_agent(
    model=model,
    middleware=custom_pii_middlewares
)
```

当输入文本 `"联络电话 13812345678，密钥 sk-abcdef1234567890，详情见 https://open.api.org"` 流经该 Agent 时，手机号将依循自定义正则被识别并加星遮蔽，API Key 被哈希化处理，而未声明任何检测策略的通用 URL 则原样保留，精确满足定制化的脱敏边界控制要求。

---

### 任务规划与进度追踪中间件

让通用大模型自主解决长流程工程任务时，模型极易表现出类似“经验不足的实习生”的行为特征：面对复杂的子任务链路，执行到第二或第三步时就遗忘了初始诉求并产生幻觉；当某个工具抛出运行报错时，模型往往产生应急反应，直接绕过必要的验证环节，给出草率且错误的最终答复。

`TodoListMiddleware` 为 Agent 引入了动态任务规划、状态看板维护与进度自愈追踪能力，使其表现提升至具备严密条理的“资深工程师”水准。

#### 复杂多步任务退化与中间件引入准则

在实际开发中，并非所有调用场景都需要加载 `TodoListMiddleware`。必须根据任务的复杂度与步骤的不确定性建立清晰的选型决策链路：

```text
                  +--------------------------+
                  | 当前任务是否需要多步拆解？ |
                  +--------------------------+
                                |
                +---------------+---------------+
                | 否                            | 是
                v                               v
    +-----------------------+     +-------------------------------+
    | 简单直接任务          |     | 步骤顺序固定且无需动态自愈？   |
    | (如单次问答、代码翻译)|     +-------------------------------+
    | 决策：坚决不用        |                   |
    | 避免浪费模型调用算力  |           +-------+-------+
    +-----------------------+           | 是            | 否
                                        v               v
                        +-------------------+   +-------------------------+
                        | 静态编排场景      |   | 动态长链路探索场景      |
                        | 决策：选用        |   | (如跨多文件协同排错)    |
                        | LangGraph 线性图  |   | 决策：引入              |
                        | 节点硬编码控制    |   | TodoListMiddleware      |
                        +-------------------+   +-------------------------+
```
> 任务编排技术选型决策流：只有步骤不确定且面临执行失败重试的多步骤任务，才适合引入待办任务清单中间件。

选型需要重点把握两个判别标准：

1. **不需要拆解的单步任务坚决不引入**：诸如简单的加减计算、文本翻译、单次接口查询，若强制模型为其制定待办清单，属于毫无意义的 Token 浪费；
2. **静态确定性的步骤优先交由图工作流处理**：若任务仅是机械的“第一步做 A，第二步做 B，第三步做 C”，无需因现场反馈动态改变策略，应直接选用 LangGraph 等框架的静态状态机节点完成；
3. **步骤动态发散且高度依赖中间反馈的场景必须引入**：当后续步骤取决于前序工具执行的成败，需要应对意外报错并重新制定策略时（例如 Deep Research 深度调研、跨工程代码修复与自动化回归测试），`TodoListMiddleware` 能够时刻锚定整体目标。

该中间件在初始化时提供了 `system_prompt`（自定义待办规范引导词）与 `tool_description`（自定义任务管理工具描述）两个参数。通常情况下直接采用系统内置的默认配置即可，无需额外显式指定。

#### 内置待办工具与状态驱动机制

`TodoListMiddleware` 的底层实现并非被动记录日志，而是在初始化阶段向 Agent 自动注入一个专用的内置工具：`write_todos`。

Agent 在执行任务时，首先调用的不是业务工具，而是向 `write_todos` 提交一份结构化的任务清单数组。每一个任务项包含两个核心数据字段：

- `content`：该待办子任务的具体目标描述；
- `status`：该任务项的生命周期状态，典型取值包括 `in_progress`（正在执行）、`pending`（排队待执行）以及完成状态。

```text
[用户目标输入]
      |
      v
(大模型决策) ------------> 触发内置工具 write_todos
                                |
                                v
                +-------------------------------+
                | 写入/更新结构化任务看板       |
                | 1. 定位缺陷代码  [in_progress]|
                | 2. 修正逻辑错误  [pending]    |
                | 3. 回归测试验证  [pending]    |
                +-------------------------------+
                                |
                                v
(大模型决策) ------------> 调用业务工具读取并修改文件
                                |
                                v
(大模型决策) ------------> 触发内置工具 write_todos
                                |
                                v
                +-------------------------------+
                | 刷新看板进度并转移状态        |
                | 1. 定位缺陷代码  [completed]  |
                | 2. 修正逻辑错误  [completed]  |
                | 3. 回归测试验证  [in_progress]|
                +-------------------------------+
```
> Agent 在多轮交互中持续通过 `write_todos` 刷新任务看板，驱动状态向后流转，避免在长文本上下文中偏离目标。

#### 代码自动测试与自愈修复实战

为了完整观察 `TodoListMiddleware` 的动态拆解与容错纠偏机制，构建一个典型的代码缺陷定位与回归修复场景。

首先在本地准备一个受限的工作区目录 `todo_workspace/`，并在其中写入故意写错的运算模块 `my_add.py` 及对应的单测脚本 `test_my_add.py`：

```python
# todo_workspace/my_add.py
def add(a, b):
    # 逻辑缺陷：加法误写为减法
    return a - b
```

```python
# todo_workspace/test_my_add.py
from my_add import add

def test_add():
    assert add(2, 3) == 5
```

在终端命令行中切换至该目录并执行 `pytest -q`，系统会立刻暴露断言失败错误：`-1 != 5`。

接着为 Agent 提供四个用于操作工作区的标准原子工具：

```python
import os
import subprocess
from langchain.tools import tool

WORKSPACE = "./todo_workspace"

@tool
def list_files() -> list[str]:
    """扫描指定工作区目录下的所有代码与测试文件"""
    return os.listdir(WORKSPACE)

@tool
def read_file(filename: str) -> str:
    """读取工作区内指定文件的完整源码文本"""
    path = os.path.join(WORKSPACE, filename)
    with open(path, "r", encoding="utf-8") as f:
        return f.read()

@tool
def write_file(filename: str, content: str) -> str:
    """覆盖写入修正后的代码到工作区指定文件中"""
    path = os.path.join(WORKSPACE, filename)
    with open(path, "w", encoding="utf-8") as f:
        f.write(content)
    return f"文件 {filename} 写入成功。"

@tool
def run_tests() -> str:
    """在工作区内执行 pytest 并捕获测试套件的输出结果"""
    result = subprocess.run(
        ["pytest", "-q"],
        cwd=WORKSPACE,
        capture_output=True,
        text=True
    )
    return result.stdout if result.returncode == 0 else result.stderr
```

接下来创建集成了 `TodoListMiddleware` 的代码修复 Agent。为了强化规划意识，可以在 `system_prompt` 中明确要求其在应对复杂工作时先列出规划：

```python
from langchain.agents import create_agent
from langchain.middleware import TodoListMiddleware
from langchain.messages import HumanMessage

system_prompt = (
    "你是一名专业代码修复助手。在处理多步骤任务时，必须优先使用 write_todos "
    "制定结构化待办事项，然后分步执行文件查阅、代码修复与测试运行。"
    "所有操作均在指定工作区下进行。"
)

tools = [list_files, read_file, write_file, run_tests]

agent = create_agent(
    model=model,
    tools=tools,
    middleware=[TodoListMiddleware()],
    system_prompt=system_prompt
)
```

发起任务调用并打印执行全流程日志：

```python
from rich import print as rprint

response = agent.invoke({
    "messages": [HumanMessage(content="请测试并修复工作区下的 my_add.py 文件中的代码。")]
})

rprint(response)
```

追踪控制台输出的完整交互帧序列，可以清晰看到 Agent 思维与行为的进化：

1. **首次响应（AI Message）**：模型未直接调用文件读取工具，而是发起了对内置工具 `write_todos` 的调用，生成包含四个步骤的待办列表：
   - 步骤一：扫描工作区结构并定位源码与测试（状态：`in_progress`）；
   - 步骤二：读取源码文件并分析逻辑问题（状态：`pending`）；
   - 步骤三：修改代码并运行回归测试（状态：`pending`）；
   - 步骤四：确认测试通过并输出修复总结（状态：`pending`）。
2. **工具执行与状态流转**：紧随其后的 `ToolMessage` 确认待办看板持久化成功；下一帧中，Agent 调用 `list_files` 获取文件列表，继而调用 `read_file` 阅读 `my_add.py`。
3. **定位缺陷并修改**：识别到 `return a - b` 之后，Agent 调用 `write_file` 将源码纠正为 `return a + b`。
4. **运行测试验证与动态修正**：Agent 紧接着调用 `run_tests` 执行测试。若测试环境返回失败，模型会再次唤醒 `write_todos` 更新待办看板，加入重新分析步骤；测试通过后，看板中所有项目状态被刷新为已完成。
5. **结束交互**：最终检查物理磁盘上的 `my_add.py`，减号已被成功覆写为加号，系统全自动化闭环完成修复。

---

### 模型调用频次与预算熔断中间件

在生产环境中，Agent 偶尔会遭遇极端异常：大模型未能准确理解工具返回结果导致在两三个工具间无休止地“拉扯”，或者输出的参数持续无法通过校验而不断重试。这种不可控的死循环会在极短时间内消耗海量 Token，导致巨额账单。

`ModelCallLimitMiddleware` 是 LangChain 提供的 16 个与特定模型厂商无关的通用内置中间件之一，专用于从框架层硬性限制模型调用频次，构建系统预算保护的最后一道安全防线。

#### 调用次数限制维度与退出行为

中间件从两个不同的生命周期粒度对调用次数实施配额管控，并允许开发者在配额耗尽时选择两种不同的退出策略：

| 关键配置属性 | 参数作用与取值 | 行为特征与机制说明 |
| :--- | :--- | :--- |
| **会话限制** | `thread_limit: int` | 跨轮次持久化限制。统计同一个 `thread_id` 会话线程内的模型累计调用总次数。 |
| **单次限制** | `run_limit: int` | 单次执行限制。仅约束单次 `invoke` 内部因工具调用重试、反思等触发的模型调用次数。 |
| **退出行为** | `exit_behavior="end"` | **优雅退出**。达到上限后不再向大模型发起网络请求，流程平稳截断并静默返回。 |
| **退出行为** | `exit_behavior="error"` | **异常阻断**。达到上限后立即抛出 `ModelCallLimitExceededError`，强行阻断流程。 |

```text
                            +--------------------------+
                            | Agent 发起模型调用请求   |
                            +--------------------------+
                                          |
                                          v
                    +----------------------------------------------+
                    | ModelCallLimitMiddleware 拦截器计数校验      |
                    | (检查 run_limit 或 thread_limit 阈值)        |
                    +----------------------------------------------+
                                          |
                        +-----------------+-----------------+
                        | 未超限                            | 达到阈值上限
                        v                                   v
             +--------------------+            +--------------------------+
             | 准予发送网络请求   |            | 检查 exit_behavior 配置  |
             | 调用外部大模型服务 |            +--------------------------+
             +--------------------+                         |
                                            +---------------+---------------+
                                            | "end"                         | "error"
                                            v                               v
                                +-----------------------+       +-----------------------+
                                | 优雅退出 / 静默结束   |       | 抛出运行时异常        |
                                | 终止调用并返回当前结果|       | ModelCallLimit...Error|
                                +-----------------------+       +-----------------------+
```
> 模型调用拦截与熔断控制流：中间件在每次底层请求发出前完成计数核对，依据策略决定放行、优雅截断或抛错熔断。

#### 会话级调用限制与状态持久化

当限制作用于整个用户会话（Thread）时，中间件需要依赖检查点组件（Checkpointer）来持久化跨轮次调用的计数状态。在下一阶段系统学习记忆机制之前，此处直接加载基础检查点即可支持该特性。

设定 `thread_limit=2`，并分别验证优雅退出与异常阻断两种模式。

在优雅退出模式下配置中间件：

```python
from langchain.agents import create_agent
from langchain.middleware import ModelCallLimitMiddleware
from langchain.checkpoint import MemorySaver

# 声明每个会话线程最多允许调用 2 次模型，超限后优雅退出
limit_middleware = ModelCallLimitMiddleware(thread_limit=2, exit_behavior="end")
checkpointer = MemorySaver()

agent = create_agent(
    model=model,
    middleware=[limit_middleware],
    checkpointer=checkpointer
)
```

在同一个会话配置 `thread_id="1"` 下连续发起三次独立调用：

```python
config = {"configurable": {"thread_id": "1"}}

# 轮次 1：调用模型 1 次，累计 1 次，正常返回
resp1 = agent.invoke({"messages": [{"role": "user", "content": "你好"}]}, config=config)

# 轮次 2：调用模型 1 次，累计 2 次，触碰上限，正常返回
resp2 = agent.invoke({"messages": [{"role": "user", "content": "你是谁"}]}, config=config)

# 轮次 3：已超出 2 次配额，触发优雅退出
resp3 = agent.invoke({"messages": [{"role": "user", "content": "你能帮我做什么"}]}, config=config)
```

在第三次调用时，由于前两轮对话已经消耗了全部的 2 次调用额度，中间件拦截了向模型发起的请求，不再产生额外的 API 交互，系统平稳结束。

若将参数切换为严格抛错模式：

```python
limit_middleware = ModelCallLimitMiddleware(thread_limit=2, exit_behavior="error")
```

当第三轮交互发起时，控制台将立即终止并抛出 `ModelCallLimitExceededError` 异常，精确提示该会话线程已超出模型调用的最大预算限制。

#### 单次运行调用限制与异常循环模拟

相较于会话限制，`run_limit` 针对的是单次运行过程中的死循环防护。在真实的 Agent 运行中，单次请求内要连续调用多次模型，通常发生在结构化输出解析失败或工具调用参数报错引起的多次自愈重试中。

为了在无外部真实死循环的受控环境中验证 `run_limit=3` 的熔断效果，可以通过“结构化输出联合类型解析冲突”制造连续报错循环，并配合本地 Mock 服务精确控制重试频次。

在大模型结构化输出设计中，当使用 `with_structured_output` 配合 `Union[ContactInfo, EventInfo]` 时，模型理论上应当单选其一输出。如果模型异常地同时返回了两个工具调用，客户端在反序列化联合类型时就会报错（报出 `multiple tool calls` 错误），进而驱动 Agent 不断向模型重新发起请求进行纠偏。

```python
from pydantic import BaseModel, Field
from typing import Union

class ContactInfo(BaseModel):
    name: str = Field(description="联系人姓名")
    phone: str = Field(description="联系方式")

class EventInfo(BaseModel):
    event_name: str = Field(description="事件名称")
    date: str = Field(description="发生日期")

# 联合类型：正常模型仅能二选一返回
TargetSchema = Union[ContactInfo, EventInfo]
```

编写一个独立的本地 Mock 服务（监听 `127.0.0.1:8889`），在接收到推理请求时，通过随机数机制以 80% 的高概率同时塞入两个工具调用，迫使客户端解析器持续报错重试；以 20% 的概率只返回一个工具调用完成正常解析：

```python
# 虚拟 Mock 服务核心响应构造逻辑片段（用于测试重试熔断）
# 随机数大于 2 则执行 append，产生 80% 的双工具冲突概率
if random.randint(1, 10) > 2:
    tool_calls.append(event_tool_call)
```

将 Agent 指向该本地虚拟服务，配置单次运行限制 `run_limit=3`，并分别测试两种退出行为：

```python
from langchain_openai import ChatOpenAI

# 指向本地 Mock 服务器，无需真实 API Key
mock_model = ChatOpenAI(
    base_url="http://127.0.0.1:8889",
    api_key="mock-key",
    model="ANY"
)

# 场景 A：优雅退出
run_limit_agent_end = create_agent(
    model=mock_model.with_structured_output(TargetSchema),
    middleware=[ModelCallLimitMiddleware(run_limit=3, exit_behavior="end")]
)

# 场景 B：异常阻断
run_limit_agent_error = create_agent(
    model=mock_model.with_structured_output(TargetSchema),
    middleware=[ModelCallLimitMiddleware(run_limit=3, exit_behavior="error")]
)
```

在测试执行时：

1. **概率重试链条**：当 80% 的异常概率连续命中时，Agent 遭遇第一次解析错误并自动发起第二次模型调用；若依然冲突，则发起第三次调用；
2. **达到上限截断**：当单次运行内的模型调用尝试累积达到 3 次上限时，中间件强制介入；
3. **行为差异验证**：在配置为 `exit_behavior="end"` 时，Agent 终止无限重试并安全退出；在配置为 `exit_behavior="error"` 时，控制台直接抛出 `ModelCallLimitExceededError`，彻底阻断了因为模型反复生成错误格式而造成无限死循环的风险。

> **承前启后**：上一节讲完「隐私脱敏、任务清单与模型调用限制中间件」，下一节接着讲「容灾重试、上下文编辑与 Node-style 钩子基础」。

---

## 容灾重试、上下文编辑与 Node-style 钩子基础
> 对应块：BLK17 | 覆盖分集：P70-P80 | 块标题（分集名）：《容灾重试、上下文编辑与Node-style钩子基础》

前面我们讲到过控制大模型调用次数的中间件，能够避免大模型自身失控循环消耗调用配额。但在实际的智能体应用中，大模型并不是唯一的调用源，外部工具同样存在不可控的调用风险。如果一个智能体判定需要调用某个查询工具，而该工具由于参数偏差或网络问题迟迟返回不了预期结果，智能体就极有可能陷入死循环，反复发起请求。更糟糕的是，如果调用的工具背后是高昂的计费接口、高并发的爬虫抓取或是核心数据库查询，这种无节制的循环调用会迅速烧光额度，甚至拖垮下游系统。

为此，我们需要构建一套完整的容灾防护体系：既包含工具调用频次限制、多级模型降级备用、大模型工具预筛选、指数退避重试，也涵盖未就绪工具模拟、多轮上下文瘦身裁剪以及文件系统检索中间件。在此基础之上，我们还要厘清多个中间件组合时的洋葱调用模型，并深入智能体的生命周期插槽，通过函数装饰器实现轻量级节点风格（Node-style）的自定义钩子扩展。

---

### 工具调用次数限制中间件

#### 核心机制与三种退出策略

`ToolCallLimitMiddleware` 的核心职责就是约束智能体执行期间的工具调用次数。它具备双重粒度的控制能力：既可以限制整个会话生命周期内所有工具调用的总次数，也可以针对某些昂贵工具设置独立的调用上限。

> **定义**：`ToolCallLimitMiddleware` 是 LangChain 体系中用于约束工具调用频次的防御型中间件，通过设定调用阈值与退出策略，防止 Agent 陷入无休止的工具调用循环，并控制外部 API 与底层存储的访问成本。

该中间件的核心参数包括调用次数限制（如传入整数上限）以及退出行为 `exit_behavior`。当工具调用累计达到设定的上限时，系统支持三种不同的应对行为：

| 退出策略 | 触发机制 | 行为表现 | 适用场景 |
| :--- | :--- | :--- | :--- |
| `end` | 达到调用上限 | 强行终止当前 Agent 会话，流程安全退出 | 生产环境中要求硬性止损、防止额度击穿的兜底场景 |
| `error` | 达到调用上限 | 主动抛出 `ToolCallLimitExceededError` 异常 | 上游调用方需要捕获异常并执行独立告警或事务回滚的场景 |
| `continue` | 达到调用上限（默认） | 不中断 Agent，将超限告警信息注入上下文交由模型决策 | 模型具备较强自我纠错能力，允许模型根据超限提示改换策略 |

> **注意**：`continue` 是该中间件的默认退出行为。当调用超限后，系统会将“工具调用已超出限制”的信息回传给大模型。此时如果底层模型推理能力较弱，它发现调用超限后仍然执拗地尝试发起同名工具调用，极有可能引发更深层次的逻辑死循环。因此在选用 `continue` 策略时，必须确保基座模型具备理解超限提示并主动换道推理的能力。

#### 策略执行表现验证

为了清晰观察三种策略的执行差异，我们构建一个具备概率返回特性的虚拟服务环境：大模型响应中有 80% 的概率返回两个工具调用需求，20% 的概率仅返回单工具调用需求。我们将工具调用上限限制为 2 次。

在 `end` 策略下，Agent 首次调用大模型，服务端按 80% 概率返回了两个工具调用任务。智能体在执行完这两个工具调用后，由于业务条件未完全满足，再次向大模型发起请求，大模型再次返回两个工具调用。此时中间件检测到总调用次数已达上限，触发 `end` 逻辑，整个 Agent 会话直接平稳结束，不再发起新的网络往返。

在 `error` 策略下，配置保持上限为 2 次不变，仅将行为参数调整为 `error`：

```python
from langchain.agents import create_agent
from langchain.agents.middleware import ToolCallLimitMiddleware

# 配置工具调用次数上限为 2，超出则抛出异常
tool_limit_middleware = ToolCallLimitMiddleware(
    max_tool_calls=2,
    exit_behavior="error"
)

agent = create_agent(
    model=model,
    tools=tools,
    middleware=[tool_limit_middleware]
)
```

当模型连续两次返回双工具调用、累计执行达到阈值时，控制台直接抛出 `ToolCallLimitExceededError` 异常中断执行。而如果在执行过程中，大模型命中了 20% 的分支只返回了一个工具调用且完成任务，则调用总量未达到 2 次，程序正常结束且不触发异常。

在 `continue` 策略下，即使工具调用累计达到上限，程序也不会硬性崩溃，而是将超限提示作为观察信息回传给大模型。大模型在感知到工具不可用后，尝试重新评估上下文并组织最终输出。若大模型连续多次仍然无法收敛，整个交互过程可能会拉长至数十秒，直至模型在提示约束下妥协并直接生成文本响应。

---

### 模型备用容灾机制

#### 故障转移设计原理

在分布式和公有云环境中，没有任何单一模型服务能够保证 100% 的高可用性。主模型可能因账户欠费、接口并发超限（HTTP 429）、云厂商宕机或网络专线抖动而突然不可用。如果业务链路直接依赖单一模型实例，主模型的故障将直接导致整个智能体瘫痪。

`ModelFallbackMiddleware` 提供了模型备用（容灾故障转移）机制。它的核心逻辑非常直观：当配置的主模型由于各种不可抗力无法访问时，自动将请求故障转移（Failover）至备用模型，保障业务的连续性。

> **定义**：`ModelFallbackMiddleware` 是用于模型高可用容灾的中间件，它维护一条由主模型、首选备用模型以及追加备用模型构成的降级链条，在前序模型发生异常时实现无缝自动切换。

该中间件在初始化时，主模型参数既可以传递模型实例，也可以直接传递模型标识字符串（底层自动调用 `init_chat_model` 完成装配）。其备用参数支持多级配置：

- `first_model`：主模型故障时，首选顶替上去的第一备用模型。
- `additional_models`：当第一备用模型同样故障或不可用时，依次跟进的后续追加模型列表。

#### 容灾切换验证

我们可以通过传递一个完全不存在的主模型标识，验证故障转移机制的生效情况：

```python
from langchain.agents import create_agent
from langchain.agents.middleware import ModelFallbackMiddleware

# 主模型配置为不可用的无效标识，备用模型配置为 deepseek-chat (V3 Flash)
fallback_middleware = ModelFallbackMiddleware(
    first_model="deepseek-ai/DeepSeek-V3-Flash",
    additional_models=["openai/gpt-4o-mini"]
)

agent = create_agent(
    model="invalid-nonexistent-model",
    tools=tools,
    middleware=[fallback_middleware]
)

response = agent.invoke({"messages": [("human", "你好，请介绍一下你自己")]})

# 验证实际响应模型的元数据
actual_model = response["messages"][-1].response_metadata.get("model_name")
print(f"实际执行推理的模型为: {actual_model}")
```

运行后可以观察到，虽然主模型传入的是不可解析的无效字符串，但程序并未发生崩溃中断。中间件自动捕获了主模型的初始化或调用失败，将请求切流至 `first_model`。通过解析末条响应消息中的 `response_metadata["model_name"]` 字段，可以清晰看到当前生成结果正是来自于备用模型 DeepSeek V3 Flash。

---

### 基于子模型的工具智能筛选

#### 工具膨胀与两级筛选架构

当智能体接入的企业工具库日益庞大时，若将成百上千个工具的接口描述（Schema）无差别地塞进每次模型调用的提示词中，会带来三大严重负面影响：

1. **上下文成本激增**：每个工具的入参说明、字段描述都会消耗大量 Token，使得哪怕一个简短的问候都需要支付高昂的上下文费用；
2. **推理干扰与误选**：过多的工具定义会稀释模型的注意力焦点，容易导致大模型在相似工具间产生幻觉或误选；
3. **首字延迟上升**：长上下文推理显著推高了 TTFT（首字到达时间），严重损害交互体验。

为了解决这一矛盾，`LLMToolSelectorMiddleware` 引入了两级智能筛选架构。其核心理念是：使用一个推理成本极低的小型子模型（`model_in`，如轻量化 Mini 模型）充当筛选专员，先根据用户当前的意图从大工具池中挑选出关联度最高的工具子集，再把过滤后的精简工具集提交给外层主业务模型（`model_out`）进行最终决策。

```text
+------------------+         +-------------------------------+
| 用户输入 Prompt   | ------> | 子模型筛选 (model_in)          |
+------------------+         | (基于语义匹配度评估工具集)       |
                             +---------------+---------------+
                                             |
                                             v 筛选出高相关工具子集 (最多 max_tools 个)
                                             | + 白名单工具 (always_include)
                                             v
                             +-------------------------------+
                             | 外层主业务模型 (model_out)      |
                             | (仅加载精炼后的工具子集执行调用) |
                             +-------------------------------+
```

*图：基于子模型预筛选的两级工具分发流程，先通过低成本子模型精简候选池，再由主模型执行精确调度。*

该中间件涉及三个核心参数：
- `model`：用于执行工具语义匹配与筛选的轻量子模型实例；
- `max_tools`：一个整型数值，限定子模型动态挑选工具的最大数量；
- `always_include`：白名单词典或列表，指定不论用户意图如何、都必须无条件保留的工具。

> **注意**：`always_include` 中指定的工具是**独立于** `max_tools` 额度之外的。也就是说，如果设置 `max_tools=5` 且 `always_include=["get_weather"]`，那么最终暴露给外层主模型的工具上限实际上是 $5 + 1 = 6$ 个。

#### 策略对比矩阵

为了验证筛选器的控制粒度，我们为 Agent 注册四个测试工具：`get_weather`（查天气）、`get_news`（查新闻）、`calculate`（数学计算）和 `query_stock`（查股票）。我们通过多组极端与典型配置，测试其在组合查询“北京天气如何？今日新闻概要？”时的行为表现：

| 测试场景 | `max_tools` | `always_include` | 最终注入主模型的工具集 | 意图执行表现 |
| :--- | :--- | :--- | :--- | :--- |
| 极端天气白名单 | `0` | `[get_weather]` | 仅 `get_weather` | 天气正常查出；新闻工具被硬性剔除，主模型提示无法查新闻 |
| 极端新闻白名单 | `0` | `[get_news]` | 仅 `get_news` | 新闻正常查出；天气工具未被注入，主模型提示无法查天气 |
| 强制双白名单 | `0` | `[get_weather, get_news]` | `get_weather` 与 `get_news` | 两者均被白名单保底注入，天气与新闻均能成功查出 |
| 单配额动态推荐 | `1` | `[get_weather]` | 保底 `get_weather` + 动态挑出 `get_news` | 子模型识别新闻需求并动态分配配额，两个工具均就位并执行 |
| 镜像动态推荐 | `1` | `[get_news]` | 保底 `get_news` + 动态挑出 `get_weather` | 子模型识别天气需求并动态分配配额，两个工具均就位并执行 |

```python
from langchain.agents import create_agent
from langchain.agents.middleware import LLMToolSelectorMiddleware

# 使用子模型过滤工具：白名单保留天气，动态配额设为 1
tool_selector = LLMToolSelectorMiddleware(
    model=model_in,  # 轻量筛选子模型
    max_tools=1,
    always_include=["get_weather"]
)

agent = create_agent(
    model=model_out,  # 主业务推理模型
    tools=[get_weather, get_news, calculate, query_stock],
    middleware=[tool_selector]
)
```

在第四个场景下，外层主模型接收到的工具定义精确锁定在 `get_weather` 和 `get_news` 这两个相关项上，其余无关的数学计算与股票工具被自动丢弃。这既保证了多意图复合任务的正常执行，又将大模型上下文中的工具 Schema 规模压缩到了最低限度。

---

### 容灾重试与指数退避策略

#### 指数退避算法与防惊群抖动

在复杂的网络拓扑和微服务调用中，瞬时网络超时或下游短时限流是不可避免的常见抖动。当工具执行或模型请求遭遇失败时，最忌讳的做法是立即发起固定间隔的高频重试。

设想一个典型场景：某个热门票务系统由于瞬间涌入超大流量而导致服务瘫痪。如果所有客户端在请求失败后，都在接下来的每一秒内立即重试，这就相当于在已经崩溃的服务器上持续发起全量 DDoS 攻击，彻底剥夺下游系统的喘息与自愈恢复机会。

因此，工业级容灾策略必须采用**指数退避（Exponential Backoff）算法**：操作失败后系统不立即重试，也不等待固定时长，而是让重试间隔时间按指数级逐级翻倍递增。

$$t_{\text{wait}} = \min(t_{\text{initial}} \times b^{n}, t_{\text{max}})$$

其中 $t_{\text{initial}}$ 为初始延时时间，$b$ 为退避底数因子（Backoff Factor），$n$ 为当前重试次数序号，$t_{\text{max}}$ 为单次等待延时的硬性上限。以 $t_{\text{initial}} = 1\text{s}, b = 2$ 为例，每次重试的等待时间依次为 $1\text{s} \to 2\text{s} \to 4\text{s} \to 8\text{s}$，为下游服务的负载释放争取充足的时间窗口。

然而，单纯的指数退避仍然存在潜在缺陷——**惊群效应（Thundering Herd Problem）**。如果在某个峰值时间点，有数千个并发请求由于同一原因同时失败，即使它们都遵循指数退避，在 2 秒后它们仍会极其整齐地同时发起第二轮冲击，在 4 秒后又会齐刷刷地发起第三轮冲击。

为了打破这种同频共振，系统引入了**抖动机制（Jitter）**。抖动是在计算出的理论退避时间基础上，额外叠加一个随机扰动偏移量。例如理论等待时间为 2 秒，加入抖动后各个客户端的实际重试点可能被打散到 1.8 秒、2.1 秒或 2.3 秒。通过随机化打散，所有失败请求在时域上被平滑错峰，从而真正消除并发重试带来的次生流量海啸。

```text
无抖动指数退避 (同频共振，引发惊群):
失败时刻 0s ---------> 重试波次 1 (+1s) ---------> 重试波次 2 (+2s) ---------> 重试波次 3 (+4s)
[所有失败并发请求]      [数千请求同时冲击]          [数千请求同时冲击]          [数千请求同时冲击]

加入抖动 Jitter (错峰平滑，打散流量):
失败时刻 0s ---------> 客户端 A: 0.9s 重试 ------> 客户端 A: 2.8s 重试
                      客户端 B: 1.1s 重试 ------> 客户端 B: 3.2s 重试
                      客户端 C: 1.3s 重试 ------> 客户端 C: 4.1s 重试
```

*图：指数退避与随机抖动对比示意图。加入随机抖动后，并发请求的时钟完全错开，避免了下游集群遭受周期性洪峰冲击。*

#### 重试中间件参数体系与实践

LangChain 针对工具执行与模型推理分别提供了两款功能对齐的重试中间件：`ToolRetryMiddleware`（工具调用重试）与 `ModelRetryMiddleware`（模型调用重试）。它们的参数设计完全统一，支持细粒度的容灾调优：

- `max_retries`：最大允许的重试次数。若设置为 6，意味着连同首次尝试在内，系统总共最多执行 7 次请求。如同俗称“猫有九条命”，此处系统一共具备 7 次尝试机会。
- `backoff_factor`：退避底数因子，默认为 2（即按 2 的次方倍增），也可调整为 3 或其他基数。
- `initial_delay`：首次重试前的初始等待时间，例如 1 秒。
- `max_delay`：单次延时上限阈值，例如 10 秒，防止因重试次数较多导致单次等待数分钟。
- `jitter`：布尔值，是否启用随机抖动打散机制。
- `retry_on`：异常捕获元组，明确指定只有发生哪些类型的异常时才触发重试（例如 `(TimeoutError, ConnectionError)`），非指定的致命异常则立即向上抛出。
- `on_failure`：当重试次数全部耗尽后系统最终的退出动作，支持 `"continue"`（不抛异常，把错误文本交给模型进行后续总结并平稳退出）与 `"error"`（直接向外抛出异常终止调用）。

在模型重试场景下，我们可以通过指定一个不存在的无效模型名称（如 `deepseek-cat`）来模拟接口的持续调用失败：

```python
from langchain.agents import create_agent
from langchain.agents.middleware import ModelRetryMiddleware

model_retry_middleware = ModelRetryMiddleware(
    max_retries=6,
    backoff_factor=2,
    initial_delay=1.0,
    max_delay=10.0,
    jitter=False,
    retry_on=(ValueError, RuntimeError),
    on_failure="continue"  # 耗尽后不抛异常，优雅收尾
)

agent = create_agent(
    model="deepseek-cat",
    tools=tools,
    middleware=[model_retry_middleware]
)
```

当 `on_failure="continue"` 时，程序在等待 30 多秒完成累计 7 次重试后平稳结束。模型返回的响应中清晰报告了具体错误信息，并指明支持的 API 模型应为 `deepseek-v3-pro` 或 `v3-flash`，并未抛出未捕获异常。而若将 `on_failure` 修改为 `"error"`，程序在耗尽 7 次尝试后，将直接抛出对应的异常堆栈中断运行。

---

### 工具模拟器中间件

在协同研发或原型验证阶段，常常遇到业务工具尚未开发完毕的状况。例如后台接口正在编写，负责 Agent 编排的工程师仅知道工具的入参结构、函数名称以及功能文档（Docstring），但无法获得真实的业务返回值。此时如果为了调通链路而手写大量 Mock 逻辑，不仅耗时费力，而且难以拟真多样化的业务返回。

`LLMToolEmulator` 中间件正是为该痛点量身打造的模拟组件。

> **提示**：需要特别注意命名规范。`LLMToolEmulator` 在类名末尾并没有带有常规的 `Middleware` 后缀，但它在设计与注册层面上是完全标准的中间件。

它的工作机制十分精巧：开发者只需要定义一个空壳工具函数，写清规范的函数签名与详细的中文或英文文档注释（Docstring）。在中间件内部，我们挂载一个专门负责模拟的子模型（`model_in`）。

```python
from langchain.agents import create_agent
from langchain.agents.middleware import LLMToolEmulator
from langchain.tools import tool

@tool
def get_weather(city: str) -> str:
    """根据输入的城市名称查询当天的实时天气情况，包含气温、湿度、风力风向及穿衣建议。"""
    # 业务细节尚未开发完成，函数体为空逻辑
    pass

# 配置工具模拟器中间件，挂载负责生成 Mock 数据的模型
tool_emulator = LLMToolEmulator(model=model_in)

agent = create_agent(
    model=model_out,
    tools=[get_weather],
    middleware=[tool_emulator]
)

response = agent.invoke({"messages": [("human", "查询北京的天气情况")]})
```

当外层 Agent 经过分析决定调用 `get_weather` 时，该调用请求会被 `LLMToolEmulator` 拦截。中间件内部的模拟模型读取该工具的 Docstring 和调用参数，实时编造出高度拟真、细节甚至比简单真实接口更详实的结构化文本返回给 Agent，最终驱动外层主模型完成整个端到端流程。这极大加速了前后端分离场景下智能体的预演测试。

---

### 上下文编辑与 Token 裁剪

#### 上下文膨胀与抽脂手术机制

在长生命周期的多轮对话中，或者在智能体高频调用会产生海量输出的工具（例如执行一段返回上千行日志的代码、或者抓取并解析了一整个网页的 HTML）时，历史消息列表中的字符量会发生指数级膨胀。

如果不加干预，后续每一轮哪怕最细微的追问，都需要把此前积累下来的巨幅工具输出原封不动地重新发送给大模型。这不仅急剧拉高了 Token 账单，还会导致有效信息被海量噪声淹没。

`ContextEditingMiddleware` 的作用就是扮演上下文管理的“抽脂手术医生”。它专注于对历史上下文中的冗余文本执行定向剪裁与剔除，在保留对话主线逻辑的前提下大幅压缩 Token 消耗。

其关键配置集中在 `edits` 规则列表中：
- `trigger`：触发上下文编辑的 Token 阈值门槛（默认数值较大，测试时可设为 50 等较低数值以便观察）；当累计 Token 超过该值时，自动激活裁剪；
- `keep`：指定保留的历史工具调用条数。例如设为 `keep=0`，表示一旦触发编辑，历史上的所有冗余工具输出消息将被彻底清除，只保留最新的必要状态。

#### 裁剪效果的量化观察

很多开发者在初次接触 `ContextEditingMiddleware` 时容易产生误解，认为只要中间件生效，消息列表中的消息条数（Message Count）就一定会变少。

> **易错点**：使用与不使用 `ContextEditingMiddleware`，对话列表中的消息物理条数可能完全相同。中间件并没有粗暴地从内存中删除消息对象，而是将其中的冗长文本（Content）进行了裁剪重写。要客观衡量其效果，必须通过每轮调用后 `response["messages"][-1].usage_metadata["input_tokens"]` 的增长幅度来判断。

为了量化这一表现，我们在包含 `InMemorySaver` 短期记忆的环境中，让 Agent 连续发起三轮对话。每轮对话均触发一个故意返回数千字符天气描述的超长工具。我们分别在开启中间件与关闭中间件（对照组）的情况下记录各轮的 `input_tokens` 消耗：

```python
from langchain.agents import create_agent
from langchain.agents.middleware import ContextEditingMiddleware
from langgraph.checkpoint.memory import InMemorySaver

# 配置上下文编辑：达到 50 tokens 即触发，清除全部历史工具冗余文本
context_editing = ContextEditingMiddleware(
    edits=[
        {"trigger": 50, "keep": 0}
    ]
)

checkpointer = InMemorySaver()

agent = create_agent(
    model=model,
    tools=[verbose_weather_tool],
    middleware=[context_editing],
    checkpointer=checkpointer
)
```

在对照组（未挂载中间件）中，由于多轮历史工具输出完整滞留在上下文中，第 1 轮到第 3 轮的输入 Token 呈现大幅线性递增（如 280 $\to$ 316 $\to$ 352）；而在启用上下文编辑后，超出阈值的旧工具输出被精准剔除，后续轮次的 `input_tokens` 增幅被显著压平。这充分证明了上下文编辑在长期多轮运行中的成本优化价值。

---

### 本地文件检索中间件

#### 文件检索能力封装

在很多软件工程智能体或知识库助手场景中，Agent 需要直接对本地文件系统进行检索与审查。常规做法是开发者自行编写 `os.walk` 或正则过滤函数并包装为工具，但容易面临效率低下、路径越界与大文件内存撑爆的问题。

`FilesystemFileSearchMiddleware` 为 Agent 原生赋予了底层文件检索与深度分析的能力。该中间件在初始化后，会自动向 Agent 隐式注入两款底层检索工具，开发者无需手动定义与注册：
- `glob_search`：基于文件路径和通配符进行文件目录检索；
- `grep`：基于文本内容在指定目录的文件中进行关键字深度检索。

该中间件的核心配置参数包括：
- 检索路径：指定允许检索的根目录（如 `../todo_workspace`）；
- 文件格式限定：过滤允许被检索的文件后缀（留空则表示全量检索）；
- `use_ripgrep`：布尔值。设置为 `True` 时，底层检索将采用用 Rust 编写的高性能搜索引擎 `ripgrep`，其检索速度远超原生 grep（前提是系统环境中已预装 ripgrep 工具）；
- `max_file_size_mb`：单次读取文件载入内存的最大上限（例如设置为 10 MB）。

> **注意**：`max_file_size_mb` 的数值设置需要权衡。如果设置过大，当检索到大型日志或二进制文件时，极易引发内存溢出（OOM）；如果设置过小，单次读取的文本块过于碎片化，导致 Agent 与底层文件系统交互过于频繁，整体检索性能显著下降。

当我们在提示词中要求“查找包含 add 函数的 Python 或 Jupyter 文件”时，中间件会自动路由触发 `glob_search` 定位符合扩展名的候选文件，或直接调用 `grep` 扫描代码体，并在控制台清晰输出匹配到的目标文件路径。

#### 官方内置中间件体系全景

至此，加上此前讲解的 4 个基础中间件，本教程已完整覆盖了 LangChain 官方体系中除特殊受限环境外的全部内置中间件：

1. `ModelCallLimitMiddleware`：模型调用次数限制；
2. `ToolCallLimitMiddleware`：工具调用次数限制；
3. `ModelFallbackMiddleware`：模型多级备用容灾；
4. `LLMToolSelectorMiddleware`：子模型两级工具筛选；
5. `ToolRetryMiddleware`：工具调用指数退避重试；
6. `ModelRetryMiddleware`：模型推理指数退避重试；
7. `LLMToolEmulator`：基于子模型的未就绪工具模拟器；
8. `ContextEditingMiddleware`：上下文编辑与 Token 瘦身；
9. `FilesystemFileSearchMiddleware`：本地文件系统 glob 与 grep 检索。

> **提示**：官方文档中提及的其余 3 个中间件属于特定受限场景——第 1 个需要具备命令行执行权限的专用沙盒环境（Sandbox），在常规本地 Windows 环境下无法直接测试；另 2 个与官方的深度研究（Deep Research）中间件强耦合，不具备通用单兵测试条件。因此掌握上述 9 款核心中间件，便足以应对生产级智能体绝大部分治理需求。

---

### 多中间件组合与洋葱执行架构

在实际生产项目中，我们往往不会孤立使用单一中间件，而是需要在一个 Agent 中组合挂载多个中间件（如先裁剪上下文，再进行多级备用，最后记录审计日志）。`create_agent` 的 `middleware` 参数接收一个中间件列表：

```python
middleware = [middleware_1, middleware_2, middleware_3]
```

当多个中间件同时生效时，它们的执行次序并不是随意的线性平铺，而是严格遵循经典的**洋葱模型（Onion Architecture）**。

```text
[请求进入阶段]
       |
       v
+-------------------------------------------------------------+
| Middleware 1 : before_model (最外层入口)                    |
|   +-----------------------------------------------------+   |
|   | Middleware 2 : before_model                         |   |
|   |   +---------------------------------------------+   |   |
|   |   | Middleware 3 : before_model (最内层入口)     |   |   |
|   |   |   +-------------------------------------+   |   |   |
|   |   |   |          LLM 模型推理 / 调用         |   |   |   |
|   |   |   +-------------------------------------+   |   |   |
|   |   | Middleware 3 : after_model (最内层退出)      |   |   |
|   |   +---------------------------------------------+   |   |
|   | Middleware 2 : after_model                         |   |
|   +-----------------------------------------------------+   |
| Middleware 1 : after_model (最外层退出)                     |
+-------------------------------------------------------------+
       |
       v
[响应返回阶段]
```

*图：多中间件组合时的洋葱调用模型示意图。请求进入时按列表正序执行，响应返回时按列表逆序执行。*

如上图所示，在模型调用发生前（进入阶段），中间件按照列表声明的**正序**依次执行：
$$\text{Middleware 1 (before)} \longrightarrow \text{Middleware 2 (before)} \longrightarrow \text{Middleware 3 (before)}$$

而在模型完成推理生成响应后（返回阶段），中间件则按照列表声明的**倒序**逐层向外退出：
$$\text{Middleware 3 (after)} \longrightarrow \text{Middleware 2 (after)} \longrightarrow \text{Middleware 1 (after)}$$

这一规律如同剥洋葱，层层剥入，再层层退出。如果我们调整列表顺序为 `[middleware_3, middleware_1, middleware_2]`，执行链条将严格对应重组为：
$$3_{\text{before}} \to 1_{\text{before}} \to 2_{\text{before}} \to \text{LLM} \to 2_{\text{after}} \to 1_{\text{after}} \to 3_{\text{after}}$$

> **结论**：中间件在列表中的编排顺序对系统的最终行为具有决定性影响。凡是涉及数据前置清洗、意图初筛的中间件应放置在列表前列（洋葱外层）；凡是涉及审计埋点、后置加工的逻辑，则要充分考虑其在逆序回溯时的执行时机。

---

### 自定义中间件与钩子函数原理

#### 钩子函数概念与流水线插槽模型

官方内置中间件虽然功能完备，但面对定制化的企业级需求（例如企业私有鉴权、敏感词拦截、自定义 Prometheus 监控埋点、数据库审计记录）时，必须依赖自定义中间件。

理解自定义中间件的核心，在于理解**钩子函数（Hook Function）**。

> **定义**：钩子函数是在特定系统流程的特定时机，由底层框架或主引擎自动触发回调的扩展函数。它不需要业务代码主动显式调用，只需挂载在系统预留的特定插槽上即可生效。

我们可以用现代化工业生产车间来生动比喻这一机制：

```text
[生产车间流水线插槽模型]

[主流程：投料准备] 
       |
       v
 [插槽 1: before_agent]  <---- (挂载：权限认证 / 会话初始化钩子)
       |
       v
[主流程：数据处理与提示词装配]
       |
       v
 [插槽 2: before_model]  <---- (挂载：参数校验 / 敏感词过滤钩子)
       |
       v
[主流程：核心大模型推理执行]
       |
       v
 [插槽 3: after_model]   <---- (挂载：Token 统计 / 格式合规检查钩子)
       |
       v
[主流程：结果分发与后处理]
       |
       v
 [插槽 4: after_agent]   <---- (挂载：持久化存储 / 审计日志记录钩子)
```

*图：主业务流水线与预留插槽示意图。主流程专注于核心业务推进，钩子函数嵌入插槽完成旁路扩展。*

主流程就好比车间里的自动化传送带，专注于核心部件的组装（提示词组装、模型推理、工具调度）。在传送带的关键节点之间，系统预留了一些标准化的“扩展插槽”。诸如零件质量检测、流水线喷码、重量复核等非核心主流程逻辑，不需要把代码焊死在传送带主电机里，而是以插件的方式挂载在这些插槽上。

钩子函数具备三个核心特征：
1. **被动回调**：开发者无需在业务运行代码中手动调用它，框架行进到对应生命周期节点时自动触发；
2. **依附主干**：它完全寄生并依附于一个更大的主执行流程（如 Agent 或 Graph 执行流）；
3. **无侵入解耦**：在完全不修改框架主流程源代码的前提下，灵活插入开发者私有的控制逻辑。

#### 两类钩子函数的架构分类

在基于 LangGraph 构建的 LangChain Agent 架构中，系统总共暴露了 **6 个核心钩子函数**。按其拦截与执行风格，严格划分为两大门派：

| 钩子门派 | 钩子名称 | 拦截时机与核心职责 |
| :--- | :--- | :--- |
| **Node-styled hooks**<br>（节点风格钩子） | `before_agent` | Agent 整体流程启动前触发，执行全局初始化、用户鉴权等 |
| | `before_model` | 每次调用 LLM 模型前触发，执行参数拦截、提示词微调等 |
| | `after_model` | 每次 LLM 模型推理完成返回后触发，执行结果审计、Token 统计等 |
| | `after_agent` | Agent 整体流程收尾前触发，执行资源释放、最终日志归档等 |
| **Wrap-styled hooks**<br>（包裹风格钩子） | `wrap_model_call` | 环绕包裹整个模型调用过程，具备类似 AOP 的控制权，可实现重试与熔断 |
| | `wrap_tool_call` | 环绕包裹底层具体工具的执行过程，可实现工具出入参篡改或 Mock 替换 |

节点风格（Node-style）的钩子类似于生命周期监听器，分别锚定在四个固定的时序节点上单向触发；而包裹风格（Wrap-style）的钩子则具有环绕执行特征，能够完整控制目标组件的调用与返回值接管。

---

### 基于装饰器定义 Node-style 钩子函数

#### 装饰器语法与签名规范

实现 Node-style 钩子函数有两种途径：基于函数的轻量装饰器方式与基于类的标准面向对象实现方式。对于逻辑紧凑、功能专一的钩子，基于函数装饰器的定义方式最为轻便直观。

框架提供了四个核心装饰器：`@before_agent`、`@before_model`、`@after_model` 以及 `@after_agent`。被这些装饰器修饰的函数，其形参签名必须严格遵循底层规范：

```python
from typing import Any
from langchain.agents.middleware import (
    before_agent,
    before_model,
    after_model,
    after_agent,
    AgentState,
    Runtime
)

@before_model
def my_before_model_hook(state: AgentState, runtime: Runtime) -> None:
    # 状态监控与非侵入处理
    return None
```

针对入参与返回值的设计规范，必须牢记以下核心机理：
- `state: AgentState`：表示当前 Agent 运行时的上下文状态实例。在节点风格钩子中，它承载着当前会话的消息历史列表（`state["messages"]`），允许钩子函数读取并检查当前最新的输入输出内容；
- `runtime: Runtime`：表示当前 Agent 运行时的环境实例，提供了运行时配置信息以及底层长期记忆存储的访问句柄；
- `return None`：**关键返回值规范**。在当前的无侵入钩子设计中，函数执行完毕后应明确返回 `None`。返回 `None` 明确向底层引擎传达一个信号：当前钩子仅执行旁路状态监控、埋点或非破坏性操作，绝不修改、覆盖或破坏主流程的状态数据。

#### 全链路节点钩子实战

我们将四个节点风格装饰器全部落地，并在每个钩子执行时，向当前消息列表中追加时序跟踪标记，以此实测四个生命周期节点的真实触发轨迹：

```python
from langchain.agents import create_agent
from langchain.agents.middleware import (
    before_agent,
    before_model,
    after_model,
    after_agent,
    AgentState,
    Runtime
)
from langchain_core.messages import HumanMessage

# 1. 注册 Agent 启动前钩子
@before_agent
def log_before_agent(state: AgentState, runtime: Runtime) -> None:
    print("[Lifecycle] -> before_agent 触发：Agent 流程初始化")
    return None

# 2. 注册 Model 调用前钩子
@before_model
def log_before_model(state: AgentState, runtime: Runtime) -> None:
    print("[Lifecycle] ----> before_model 触发：准备调用大模型")
    return None

# 3. 注册 Model 调用后钩子
@after_model
def log_after_model(state: AgentState, runtime: Runtime) -> None:
    print("[Lifecycle] <---- after_model 触发：大模型推理完毕")
    return None

# 4. 注册 Agent 收尾前钩子
@after_agent
def log_after_agent(state: AgentState, runtime: Runtime) -> None:
    print("[Lifecycle] <- after_agent 触发：Agent 流程即将退出")
    return None

# 创建 Agent 并将四个装饰器钩子统一装配至 middleware 列表
agent = create_agent(
    model=model,
    tools=[],
    middleware=[
        log_before_agent,
        log_before_model,
        log_after_model,
        log_after_agent
    ]
)

# 发起同步调用
response = agent.invoke({"messages": [HumanMessage(content="你好")]})

for message in response["messages"]:
    message.pretty_print()
```

运行上述代码，控制台完整打印出以下时序轨迹：

```text
[Lifecycle] -> before_agent 触发：Agent 流程初始化
[Lifecycle] ----> before_model 触发：准备调用大模型
================================== Human Message ===================================
你好
================================== Ai Message ===================================
你好！很高兴为你提供帮助。请问有什么我可以协助你的吗？
[Lifecycle] <---- after_model 触发：大模型推理完毕
[Lifecycle] <- after_agent 触发：Agent 流程即将退出
```

从日志执行顺序可以清晰验证：
1. 会话伊始，`before_agent` 首先执行，接管全局准备工作；
2. 随后在进入大模型推理前，`before_model` 触发；
3. 大模型完成内容生成后，`after_model` 紧随其后捕获到模型输出；
4. 最终在整个响应返回前，`after_agent` 执行终态收尾。

通过基于装饰器的函数式挂载，我们无需创建冗长的类定义，就能以极其优雅、解耦的方式将自定义监控、审计与拦截逻辑注入到 Agent 的生命周期全流程中。

> **承前启后**：上一节讲完「容灾重试、上下文编辑与 Node-style 钩子基础」，下一节接着讲「钩子函数进阶：类式中间件、拦截跳转与包裹执行机制」。

---

## 钩子函数进阶：类式中间件、拦截跳转与包裹执行机制
> 对应块：BLK18 | 覆盖分集：P81-P85 | 块标题（分集名）：《钩子函数进阶（模型/工具调用拦截与跳转控制）》

基于装饰器的写法我们在前一讲已经全面实践过，写起来轻快利落，非常适合快速验证逻辑。但如果回到严肃的框架设计与工程落地层面，单纯依赖装饰器并不足以支撑复杂的工业级应用。在 LangChain 的设计中，除了装饰器风格，还提供了第二套更核心、更具扩展性的实现方案——基于类（Class-based）的中间件。

掌握这套机制的关键在于理解两件事：其一，装饰器并非底层独立的新架构，它的底层本质正是类式中间件的一层轻量封装；其二，中间件不仅能做旁路观测，更能通过节点跳转（Jump）与环绕包裹（Wrap）真正介入控制流，改变图的执行轨迹。

---

### 基于类的节点风格中间件定义与实现

在 LangChain 框架内部，所有通过类定义的节点风格（Node-style）中间件都必须遵循严格的契约规范。这并不是随意写一个 Python 类就能生效的，必须满足三项硬性约束。

#### 核心规范与接口重写约束

> **定义**：类式中间件是通过显式继承框架基类并重写固定生命周期方法的组件，是 LangChain 组织多阶段治理逻辑的标准工程载体。

类式中间件的实现规则可以归纳为以下要点：

- **固定基类继承**：中间件类必须显式继承自 `AgentMiddleware`。
- **固定方法重写**：允许重写的方法名称是严格受限且固定的，包括 `before_agent`、`before_model`、`after_model` 以及 `after_agent`。
- **命名规范**：类名本身可以自由定义，但工程实践中必须做到见名知意（例如示例中的 `MyMiddleware`，或生产中的 `AuditLoggingMiddleware`）。
- **底层驱动机制**：LangChain 的智能体运行循环底层是基于 LangGraph 构建的。LangGraph 在初始化与调度执行图时，并不关心你的类叫什么名字，它只严格校验该对象是否继承自 `AgentMiddleware`，以及是否挂载了符合规范的重写方法。

```text
+-------------------------------------------------------------------------+
|                           AgentMiddleware (基类)                         |
+-------------------------------------------------------------------------+
       ^
       | 继承 (Inherit)
+------+------------------------------------------------------------------+
|                            MyMiddleware                                 |
+-------------------------------------------------------------------------+
| + before_agent(self, state, runtime) -> Optional[dict]                  |
| + before_model(self, state, runtime) -> Optional[dict]                  |
| + after_model(self, state, runtime)  -> Optional[dict]                  |
| + after_agent(self, state, runtime)  -> Optional[dict]                  |
+-------------------------------------------------------------------------+
```
上图展示了自定义类式中间件与框架基类 `AgentMiddleware` 之间的继承与接口重写拓扑关系。

#### 代码实现与生命周期时序

在代码层面，重写类式方法时必须注意补充实例方法的 `self` 形参，并显式接收状态数据 `state` 与上下文环境 `runtime`。

```python
from langchain.agents.middleware import AgentMiddleware
from langchain.agents import create_agent
from langchain_core.messages import HumanMessage

class MyMiddleware(AgentMiddleware):
    def before_agent(self, state, runtime):
        print("--- [before_agent] 智能体开始执行 ---")
        return None

    def before_model(self, state, runtime):
        print("--- [before_model] 准备调用模型 ---")
        return None

    def after_model(self, state, runtime):
        print("--- [after_model] 模型调用完毕 ---")
        return None

    def after_agent(self, state, runtime):
        print("--- [after_agent] 智能体执行结束 ---")
        return None

# 实例化中间件并挂载到 Agent
my_middleware = MyMiddleware()
agent = create_agent(
    model=model,
    tools=tools,
    middleware=[my_middleware]
)

response = agent.invoke({"messages": [HumanMessage(content="你好")]})
```

运行上述代码后，整个调用链路的执行顺序非常明确：在向底层模型发送 `HumanMessage` 之前，依次触发全局前置钩子 `before_agent` 与模型前置钩子 `before_model`；模型完成推理输出 `AIMessage` 后，再逆序触发模型后置钩子 `after_model` 与全局后置钩子 `after_agent`。

```text
[用户发起请求]
       |
       v
+------------------+
|   before_agent   |  <--- 1. Agent 全局前置检查
+------------------+
       |
       v
+------------------+
|   before_model   |  <--- 2. 模型调用前置拦截
+------------------+
       |
       v
+==================+
|    Model (LLM)   |  <--- 3. 大模型核心推理与响应生成
+==================+
       |
       v
+------------------+
|   after_model    |  <--- 4. 模型调用后置清洗与审计
+------------------+
       |
       v
+------------------+
|   after_agent    |  <--- 5. Agent 全局后置收尾
+------------------+
       |
       v
[返回最终结果]
```
上图展示了一个完整的无工具调用场景下，类式中间件四个生命周期钩子的标准执行时序。

#### 典型职责边界与场景归纳

`before_model` 与 `after_model` 是智能体交互中最核心的两个切面，它们各自承担着明确的职责分工：

- **`before_model` 典型场景**：
  - **消息修剪（Pruning）**：在发送给大模型之前裁剪历史滑动窗口，清除不必要的冗余消息。
  - **敏感信息脱敏（PII Masking）**：在数据离开本地边界前，对用户的身份信息、密钥、手机号进行正则替换与遮蔽。
  - **输入合规验证**：校验 Prompt 是否存在安全风险或格式违规。
  - **条件路由**：基于消息特征决定是否直接旁路处理。
- **`after_model` 典型场景**：
  - **输出内容验证**：校验大模型返回的文本是否包含幻觉或违规内容。
  - **响应格式化**：对大模型输出的内容进行标准化解析与后处理。
  - **指标统计**：统计本次调用的 Token 消耗量、耗时并上报可观测性平台。
  - **状态更新**：根据模型输出就地提取关键业务信息并写回全局状态。

#### 装饰器与类式实现的统一机制

许多初学者容易把装饰器写法与类式写法看作两套割裂的体系。实际上，装饰器写法只是一种语法糖，它的底层实现依然依托于类式架构。

以 `@after_model` 装饰器为例，查看源码可以发现，当使用该装饰器修饰一个独立函数时，框架内部会动态定义一个继承自 `AgentMiddleware` 的临时子类。该子类拥有 `state_schema` 和 `tools` 等必要属性，并将被装饰的函数直接挂载为该类的 `after_model` 方法。

> **结论**：一个被装饰器修饰的钩子函数，在底层被自动构造成一个仅包含单一钩子方法的 `AgentMiddleware` 子类实例。二者在运行时是完全等价且高度统一的。

关于两个输入参数与返回值的控制规则：
- **`state` 参数**：包含当前会话完整的上下文数据，其中核心字段为消息列表 `messages`。
- **`runtime` 参数**：提供了运行时的环境上下文句柄。
- **返回值行为**：
  - 返回 `None`：表示仅执行观察或只读操作，不会改变状态，也不会影响主流程的默认走向。
  - 返回字典 `dict`：框架会将返回的字典合并回全局状态。例如从 `state` 中提取计数器 `count` 进行修改后返回 `{"count": count + 1}`，即可完成状态更新。
  - 返回跳转控制：在返回字典中携带目标标识，直接改变智能体的图执行路径。

---

### 流程跳转控制与 can_jump_to 参数

在复杂的业务流中，智能体不能永远遵循固定的线性轨迹。当遇到某些特定业务规则或异常状态时，我们必须能够直接“拦截并跳转”。四个节点风格的钩子函数均支持接收 `can_jump_to` 参数，用于声明该钩子具备跳转到哪些下游节点的能力。

```text
                      +-------------------+
                      |   before_model    |
                      +-------------------+
                       /        |        \
  can_jump_to='tools' /         |         \ can_jump_to='end'
                     /          |          \
                    v           | normal    v
             +-----------+      v     +-----------+
             |   Tools   |   +-----+  |    End    |
             |  工具节点 |   | LLM |  |  流程终点 |
             +-----------+   +-----+  +-----------+
                    ^           |
                    |           | can_jump_to='model'
                    +-----------+ (via after_model retry)
```
上图展示了利用 `can_jump_to` 参数在 `before_model` 与 `after_model` 处改变智能体执行轨迹的核心拓扑流向。

#### 跳转目标与语义规范

`can_jump_to` 参数接收一个节点名称列表，允许跳转的目标值主要包括三个：

- **`end`**：直接跳转至智能体执行流程的终点，提前终止整个 Agent 的生命周期。
- **`tools`**：跳过大模型推理环节，直接跳转到工具执行节点。
- **`model`**：重新跳转回模型调用节点（常用于重试或二次追问）。

> **注意**：跳转并非瞬时抹除一切后续逻辑。如果跳转到 `end`，若中间件中还注册了 `after_agent` 钩子，该收尾钩子仍会被触发执行；若跳转到 `model`，在进入模型之前，前置的 `before_model` 钩子依然会按生命周期规范再次执行。

#### 基于装饰器的跳转实战

我们可以通过四个测试用例，在基于装饰器的钩子中完整验证不同的跳转分支。

```python
from langchain.agents.middleware import before_model, after_model
from langchain_core.messages import AIMessage, SystemMessage

# 案例一：直接跳过模型，伪造 AI 消息并直达工具
@before_model(can_jump_to=["tools"])
def bypass_model_to_tool(state, runtime):
    last_msg = state["messages"][-1].content
    if "direct_tool" in last_msg:
        # 伪造 AIMessage，直接注入 tool_calls，省去模型推理开销
        fake_ai_message = AIMessage(
            content="",
            tool_calls=[{
                "name": "get_news",
                "args": {},
                "id": "call_fake_news_001"
            }]
        )
        return {
            "messages": [fake_ai_message],
            "jump_to": "tools"
        }
    return None

# 案例二：模型回答后发现不满足要求，注入提示词并跳回模型重试
@after_model(can_jump_to=["model"])
def retry_model_with_restriction(state, runtime):
    user_query = state["messages"][0].content
    last_response = state["messages"][-1].content
    
    if "retry_model" in user_query:
        # 防死循环检查：如果已经包含二次回答前缀，则正常退出
        if last_response.startswith("二次回答"):
            return None
        
        # 注入约束性系统指令，要求模型重新作答
        sys_msg = SystemMessage(content="你必须以二次回答开头，并且只用一句话回答。")
        return {
            "messages": [sys_msg],
            "jump_to": "model"
        }
    return None

# 案例三：检测到上下文溢出，直接熔断跳转至结束
@before_model(can_jump_to=["end"])
def guard_overflow(state, runtime):
    last_msg = state["messages"][-1].content
    if "overflow" in last_msg:
        abort_message = AIMessage(content="上下文窗口溢出，终止执行。")
        return {
            "messages": [abort_message],
            "jump_to": "end"
        }
    return None
```

针对上述逻辑进行场景触发与验证：

- **场景 1（直达工具）**：用户提问“请帮我查询今日天气 direct_tool”。`bypass_model_to_tool` 捕获该关键词，人工构造了调用 `get_news` 工具的 `AIMessage`，直接跳转至工具节点。大模型一次都没被调用，直接输出了工具查询结果，节省了宝贵的响应时间与 Token。
- **场景 2（模型重试）**：用户输入“请随便介绍一下 LangChain retry_model”。大模型首先输出了大段详细介绍；随后进入 `after_model`，钩子识别到指令且尚未带有特定开头，立即追加 `SystemMessage` 并执行 `jump_to="model"`；模型被迫再次推理，输出了符合“二次回答开头且仅有一句话”的精简回答；第二次触发 `after_model` 时命中防御逻辑返回 `None`，流程平稳结束。
- **场景 3（熔断终止）**：用户输入“你好 overflow”。`guard_overflow` 在模型调用前生效，直接回传告警信息并跳转至 `end`，彻底阻断了后续一切计算。
- **场景 4（常规放行）**：正常问题不触发任何关键词分支，各钩子返回 `None`，智能体沿默认流水线自然流转。

#### 基于类的跳转实现与方法合并

在类式中间件中，声明跳转权限需要配合 `@hook_config` 装饰器。同时必须面对一个语法差异：类方法名是唯一的，不能像装饰器那样写多个 `before_model`。

> **易错点**：使用类式实现时，原本在装饰器中分散编写的多个 `before_model` 钩子，必须全部合并到同一个 `before_model(self, state, runtime)` 方法内部，并在上方用 `@hook_config` 集中声明所有可能用到的跳转目标。

```python
from langchain.agents.middleware import AgentMiddleware, hook_config
from langchain_core.messages import AIMessage

class FlowControlMiddleware(AgentMiddleware):
    
    # 集中声明该方法具有跳转到 tools 和 end 的权限
    @hook_config(can_jump_to=["tools", "end"])
    def before_model(self, state, runtime):
        content = state["messages"][-1].content
        
        # 分支一：直达工具
        if "direct_tool" in content:
            fake_ai = AIMessage(
                content="",
                tool_calls=[{"name": "get_news", "args": {}, "id": "call_001"}]
            )
            return {"messages": [fake_ai], "jump_to": "tools"}
        
        # 分支二：溢出熔断
        if "overflow" in content:
            abort_msg = AIMessage(content="上下文窗口溢出，终止")
            return {"messages": [abort_msg], "jump_to": "end"}
        
        return None

    # 声明跳转回 model 的权限
    @hook_config(can_jump_to=["model"])
    def after_model(self, state, runtime):
        user_msg = state["messages"][0].content
        last_resp = state["messages"][-1].content
        
        if "retry_model" in user_msg and not last_resp.startswith("二次回答"):
            sys_msg = SystemMessage(content="你必须以二次回答开头，并且只用一句话回答")
            return {"messages": [sys_msg], "jump_to": "model"}
        
        return None
```

通过 `@hook_config` 与内部条件分支的分流，类式中间件以更加结构化、内聚的方式完成了相同的多路跳转控制。

---

### 模型调用包裹钩子 wrap_model_call

与节点风格的前后拆分不同，包裹风格（Wrap-style）的钩子函数将整个调用视为一个独立的处理句柄（Handler），在单点上实现对调用过程的“环绕拦截”。

#### 接口规范与参数解析

`wrap_model_call` 接收两个核心参数：
- **`request`**：发送给大模型的完整请求对象。内部包含了 `messages` 消息列表、推理超参数（如 `temperature`）、绑定的工具定义 `tools` 以及当前的 `state` 数据。
- **`handler`**：模型调用的实际执行处理器。调用 `handler(request)` 即会触发实际的大模型网络通信，并返回 `response` 对象。

```text
           [调用 wrap_model_call]
                     |
+--------------------+--------------------+
| 1. 前置拦截处理 (修改 request.messages) |
+--------------------+--------------------+
                     |
                     v
           [handler(request) 调用]
                     |
           +---------+---------+
           | 真实 Model 推理通信|
           +---------+---------+
                     |
                     v
             [获得 response 响应]
                     |
+--------------------+--------------------+
| 2. 后置增强处理 (修改 response.result)  |
+--------------------+--------------------+
                     |
                     v
              [返回 response]
```
上图展示了 `wrap_model_call` 在模型调用外层形成的环绕拦截机制。

#### 装饰器与类式代码实现

在基于装饰器的实现中，可以直接拦截并对请求体与响应体双向打桩：

```python
from langchain.agents.middleware import wrap_model_call

@wrap_model_call
def wrap_model_call_middleware(request, handler):
    # 1. 模型调用前拦截：向最后一条输入消息中注入前缀标记
    request.messages[-1].content += " [wrap_model_call_before]"
    
    # 2. 触发核心模型推理
    response = handler(request)
    
    # 3. 模型调用后拦截：向响应的第一条消息中追加后缀标记
    response.result[0].content += " [wrap_model_call_after]"
    
    return response
```

如果要转换为工程级的类式写法，同样继承 `AgentMiddleware`，并在类内部定义同名方法：

```python
from langchain.agents.middleware import AgentMiddleware

class WrapModelCallMiddleware(AgentMiddleware):
    def wrap_model_call(self, request, handler):
        # 执行前序处理
        request.messages[-1].content += " [wrap_model_call_before]"
        
        # 调用处理器
        response = handler(request)
        
        # 执行后序处理
        response.result[0].content += " [wrap_model_call_after]"
        
        return response
```

两种方式挂载到 Agent 后，控制台输出的大模型响应文本前后均被成功包裹了对应的标记字符串，清晰印证了环绕处理器的执行轨迹。

#### 四大工业级落地场景

包裹模型调用在实际生产环境中有非常高频的工程应用：

- **场景一：失败重试机制（Retry）**  
  将 `handler(request)` 包裹在 `try-except` 或重试循环中。当大模型遇到偶发网络超时或 HTTP 429 限流时，无需上层重跑整个图，直接在底层就地进行指数退避重试。
- **场景二：大模型响应本地缓存（Response Caching）**  
  当前主流大模型服务商对于 Context Cache 与未命中缓存的收费差距巨大（例如 DeepSeek 对缓存命中的极低计费策略）。  
  在执行 `handler(request)` 前，先基于请求的 Prompt 计算哈希并查询本地/Redis 缓存。若命中，直接构造成 `response` 返回，无需向外发起任何网络调用；若未命中，再调用 `handler(request)`，并将得到的响应异步写入缓存。这不仅大幅压低了 Token 支出，更极大减轻了高并发下的后端压力。
- **场景三：动态注入系统提示词（Prompt Enrichment）**  
  在调用前根据环境变量或上下文，无侵入式地向 `request` 中拼装额外信息（例如当前精确系统时间、客户端地理位置、统一追加“请使用中文回答”等约束）。
- **场景四：响应安全过滤与规范化（Post-processing）**  
  在拿到 `response` 后，统一进行敏感词过滤清退、文本替换，或强制校验输出格式是否符合合规约束。

---

### 工具调用包裹钩子 wrap_tool_call

不仅大模型调用可以被包裹，工具（Tool）的执行同样支持通过 `wrap_tool_call` 进行环绕拦截。

#### 接口结构与参数剖析

`wrap_tool_call` 的参数模型与模型包裹高度对称，依然是 `(request, handler)`：

- **`request` 对象**：工具调用请求载荷。其内部最关键的属性是 `tool_call` 字典，包含了工具名称 `name`、大模型解析出的实参字典 `args`，以及调用标识 `id`。
- **`handler` 对象**：工具实际执行的操作句柄。调用 `handler(request)` 即可完成底层函数的执行并获取返回值。

```text
[大模型决定调用工具]
        |
        v
+-----------------------+
|    wrap_tool_call     |  <--- 拦截入口
+-----------------------+
| 读取/修改 request.    |
| tool_call['args']     |  <--- 参数校验与纠偏
+-----------------------+
        |
        v
+-----------------------+
|   handler(request)    |  <--- 执行本地/远程真实工具代码
+-----------------------+
        |
        v
+-----------------------+
| 后置审计 / 结果清洗    |  <--- 工具执行后拦截
+-----------------------+
        |
        v
[回传 ToolMessage]
```
上图展示了 `wrap_tool_call` 介入工具调用的全生命周期。

#### 工具入参拦截与治理实战

在实际调用工具之前，我们往往需要对大模型输出的参数进行微调纠偏或前置鉴权。

```python
from langchain.agents.middleware import wrap_tool_call

@wrap_tool_call
def wrap_tool_call_middleware(request, handler):
    # 提取当前调用的工具参数字典
    tool_call = request.tool_call
    tool_name = tool_call.get("name")
    args = tool_call.get("args", {})
    
    # 针对特定工具做参数治理
    if tool_name == "query_weather":
        # 强制检查或修改 is_forecast 参数
        if "is_forecast" not in args:
            args["is_forecast"] = False
        print(f"[Tool Guard] 正在调用天气工具，校准后参数: {args}")
        
    # 执行实际工具调用
    tool_result = handler(request)
    
    return tool_result
```

通过这种方式，大模型推导出的不稳定参数在真正落入核心业务代码前被彻底过滤和纠正，有效筑牢了智能体与外部系统交互的安全底线。

---

### 架构选型与中间件执行顺序

在深入掌握了装饰器与类式两套写法后，实际开发中如何做技术选型，以及多个中间件并存时它们的执行顺序如何推演，是构建复杂应用必须搞清楚的终极问题。

#### 装饰器与基于类的选型权衡

这两套写法并非优劣对立，而是针对不同规模复杂度的分工：

| 评估维度 | 装饰器方式（Decorator-based） | 类继承方式（Class-based） |
| :--- | :--- | :--- |
| **钩子数量** | 推荐用于**单钩子**场景；若要返回多个钩子需要通过闭包解包，语法晦涩不友好 | 天生适合**多钩子**协同；一个类中可清晰声明全套生命周期方法 |
| **配置复杂度** | 必须依赖**闭包**捕获外部变量；配置不易自省，调试排查成本高 | 通过 `__init__` 注入配置并挂载至 `self` 属性；面向对象，直观易维护 |
| **状态持有与检查** | 难以在外部获取内部属性字典，无法直观反射状态 | 类实例具备完整的 `__dict__`，便于序列化、反射与监控自省 |
| **工程化与复用** | 适合单个脚本、轻量实验、快速原型验证 | 适合跨模块封装、作为公共库发布、以及进行标准单元测试与 Mock |

> **提示**：选型黄金法则是——只要中间件仅包含单一钩子、逻辑极简且用于快速验证，果断选装饰器；一旦涉及多生命周期联动、有内部配置参数、需要维护持久状态或跨项目复用，毫不犹豫选择类。

#### 中间件执行顺序与洋葱模型

当一个 Agent 注册了多个中间件时，它们的执行顺序存在非常明确但容易反直觉的规律。假设我们在 `middleware` 列表中按顺序注册了 `[M1, M2, M3]`：

```python
middleware = [m1, m2, m3]
```

执行时，框架并不会机械地全流程正序走到底，而是严格遵循洋葱模型（Onion Model）与调用栈展开规律：

- **`before_*` 钩子**：按照注册列表的**正序（FIFO）**执行，即 `M1 -> M2 -> M3`。
- **`wrap_*` 钩子（前置段）**：作为洋葱外层向内层包裹，按**正序**进入，即 `M1 -> M2 -> M3`。
- **核心推理与执行**：处于最内层核心的 Model 或 Tool 触发。
- **`wrap_*` 钩子（后置段）**：离开核心向外退出，按**倒序（LIFO）**执行，即 `M3 -> M2 -> M1`。
- **`after_*` 钩子**：按照注册列表的**倒序（LIFO）**执行，即 `M3 -> M2 -> M1`。

```text
注册顺序: [ M1, M2, M3 ]

+-------------------------------------------------------------+
| M1 (before_model)                                           |
|   +-------------------------------------------------------+ |
|   | M2 (before_model)                                     | |
|   |   +-------------------------------------------------+ | |
|   |   | M3 (before_model)                               | | |
|   |   |   +-------------------------------------------+ | | |
|   |   |   | M1 (wrap 入: 1)                           | | | |
|   |   |   |   +-------------------------------------+ | | | |
|   |   |   |   | M2 (wrap 入: 2)                     | | | | |
|   |   |   |   |   +-------------------------------+ | | | | |
|   |   |   |   |   | M3 (wrap 入: 3)               | | | | | |
|   |   |   |   |   |   +=========================+ | | | | | |
|   |   |   |   |   |   |   LLM / Tool 核心调用   | | | | | | |
|   |   |   |   |   |   +=========================+ | | | | | |
|   |   |   |   |   | M3 (wrap 出: 3)               | | | | | |
|   |   |   |   |   +-------------------------------+ | | | | |
|   |   |   |   | M2 (wrap 出: 2)                     | | | | | |
|   |   |   |   +-------------------------------------+ | | | | |
|   |   |   | M1 (wrap 出: 1)                           | | | | |
|   |   |   +-------------------------------------------+ | | |
|   |   | M3 (after_model)                                | | |
|   |   +-------------------------------------------------+ | |
|   | M2 (after_model)                                      | |
|   +-------------------------------------------------------+ |
| M1 (after_model)                                            |
+-------------------------------------------------------------+
```
上图直观展示了中间件在不同生命周期阶段的正序进入与倒序解包（洋葱模型）全景。

> **易错点**：很多开发者直觉上会以为 `after_model` 也是正序的 `1 -> 2 -> 3`，这是最常见的认知陷阱！请务必牢记，所有成熟中间件体系（无论是 Web 框架中的 Koa、Django，还是 LangChain）的响应后置链路均为**逆序执行**。外层中间件最先捕获请求，必然最后处理响应，形成完整闭合的对称环。

> **承前启后**：上一节讲完「钩子函数进阶：类式中间件、拦截跳转与包裹执行机制」，下一节接着讲「记忆机制原理、短期内存持久化与云端数据库基础设施」。

---

## 记忆机制原理、短期内存持久化与云端数据库基础设施
> 对应块：BLK19 | 覆盖分集：P86-P91 | 块标题（分集名）：《记忆机制概念、短期内存存储与云数据库准备》

在完成智能体中间件的系统学习后，我们正式切入智能体研发中至关重要的一整套核心架构——上下文与记忆机制（Context and Memory）。智能体的业务能力越强，用户对其寄予的期望也越高，所要解决的现实问题自然就越来越复杂。一旦任务走向深度化，单次简单的人机问答就很难满足任务闭环的需求，必须依赖多轮次的持续交互。而一旦涉及多轮交互，一个根本性的工程挑战便浮出水面：智能体在后续的交互步骤中，是否能够拿到并理解先前交互过程中产生的上下文与中间状态？如果智能体无法调取历史交互信息，每一次对话都沦为孤立的全新开端，复杂任务的推理与执行便无从谈起。这种能够沉淀并提取过往互动信息的能力，正是我们所说的记忆。

---

### 大模型无状态特性与记忆机制需求

#### 多轮复杂交互与无状态断层

大语言模型（LLM）天然是无状态（Stateless）的。所谓无状态，是指模型本身就像一个纯函数，对于任何一次给定的 API 请求，它只基于当前传入的 Prompt 或消息序列完成概率推演并给出回复。底层模型不会在自身的显存或参数中残留关于这次调用的任何私有记忆；每一次向模型发起 `invoke` 调用，从模型的视角来看都是一场全新的、彼此毫无瓜葛的独立事件。

如果直接使用裸模型与用户交互，用户即便在上一秒完成了自我介绍，下一秒再向其询问身份信息，模型也完全无法获知。这种无状态的物理断层，构成了我们理解和开发智能体记忆系统的核心认知前提。

#### 实际应用中的会话记忆表现

在现实生活中，我们所使用的各类 AI 应用（例如字节跳动的“豆包”）却能够流畅地维持多轮会话上下文。

必须厘清的是，豆包并不是一个裸露的基座大模型，而是一个构建在多层软件工程体系之上的完整应用程序。当我们在会话中告诉豆包“我是康师傅”，紧接着追问“我是谁”时，它能够准确应答。更进一步，即便我们彻底关闭豆包的客户端界面，在操作系统后台将其进程强行终止（Kill Process），随后重新打开该应用并返回原先的会话窗口继续提问“我是谁”，它依然能够准确给出解答。

这种现象直观地表明：在成熟的 AI 应用程序中，上下文信息绝不仅仅停留在单次运行的内存中，而是通过某种持久化机制被同步沉淀到了外部的持久化存储介质（如数据库）中。每当用户重启会话，系统都会自动从持久化存储中反序列化提取历史记录，并无缝拼接为模型的推理上下文。

```text
+-------------------+       +-----------------------+       +-------------------+
|  裸大模型 (LLM)   | <---> |   单次独立调用通道    | <---> |  状态完全不留存   |
|   (天然无状态)    |       |   (无历史追踪机制)    |       |   (每次独立计算)  |
+-------------------+       +-----------------------+       +-------------------+

+-------------------+       +-----------------------+       +-------------------+
|  AI 应用系统      | <---> |   上下文与记忆管理层  | <---> |   内存 / 数据库   |
| (如豆包等智能体)  |       |   (读取、拼接、持久化)|       | (会话记录长期保留)|
+-------------------+       +-----------------------+       +-------------------+
```
上图对比了裸大模型单次无状态调用模式与完整 AI 应用程序外挂记忆管理层的数据流转差异。

智能体具备记忆机制不仅是为了满足复杂任务的逻辑闭环，还能显著改善人机交互的情感体验。一个能够记住用户习惯、工作偏好与历史事实的智能体，面对用户时展现出的是类似于朋友般的拟人化协作质感；反之，若每一次提问都如同面对一个失忆的冷漠机器，整个系统的可用性与人机信任感都将大打折扣。

#### 智能体无状态代码验证

为了在代码层直观验证大模型的无状态特性，我们通过 LangChain 的基础调用进行两组对比测试。

```python
from langchain.agents import create_agent
from langchain_core.messages import HumanMessage

# 初始化模型与智能体实例（基础配置）
agent = create_agent(model=model)

# 场景一：连续独立 invoke 调用，未配置任何记忆组件
messages_round_1 = [
    HumanMessage(content="你好，我叫小明。"),
    HumanMessage(content="我想学习大模型与智能体开发。"),
    HumanMessage(content="请根据我的背景给个建议。")
]

# 第一轮调用
response_1 = agent.invoke({"messages": messages_round_1})
print("第一轮回复：", response_1["messages"][-1].content)

# 第二轮调用：直接在同一个 agent 对象上发起全新的 invoke
messages_round_2 = [
    HumanMessage(content="我叫什么名字？")
]
response_2 = agent.invoke({"messages": messages_round_2})
print("第二轮回复：", response_2["messages"][-1].content)
```

在场景一中，尽管两次调用都作用在同一个 `agent` 实例上，但由于智能体底层默认没有挂载任何记忆持久化组件，第二次调用传入的 `messages_round_2` 是一份全新的消息列表。在这一次调用中，返回的消息列表中仅有 2 条记录（一条是刚传入的 `HumanMessage`，另一条是刚生成的 `AIMessage`），模型明确回复“我不知道你叫什么名字”。前一次调用中关于“小明”的信息早已随着调用生命周期的结束而彻底消散。

```python
# 场景二：手动将历史与当前消息合并到同一个消息列表中发起单次 invoke
messages_combined = [
    HumanMessage(content="你好，我叫小明。"),
    HumanMessage(content="我想学习大模型与智能体开发。"),
    HumanMessage(content="请问我叫什么名字？")
]
response_combined = agent.invoke({"messages": messages_combined})
print("合并调用回复：", response_combined["messages"][-1].content)
```

在场景二中，模型能够准确回答出“你叫小明”。然而这种“记忆”纯粹是开发者在代码层面通过手动拼接上下文硬塞给模型的，并不是系统自发具备的动态记忆能力。在真实生产场景中，面对不可预知的多轮交互，我们必须建立一套标准化的自动化机制，由框架负责在每次调用前后自动拉取历史、追加状态并完成存盘。

---

### 上下文工程架构与记忆分类辨析

#### 上下文工程与记忆的管理边界

在深入具体代码实现之前，必须准确区分两个高频概念：**记忆（Memory）**与**上下文工程（Context Engineering）**。

> **定义**：记忆是专门用于沉淀与存储历史交互信息的物理或逻辑数据集合；而上下文工程是负责合理组织、治理、压缩、筛选并动态组装这些记忆与任务信息，以支撑智能体应对多轮复杂交互的系统工程体系。

记忆就像一本写满历史记录的草稿本，随着对话轮次的演进，上面记录的信息往往杂乱无章、篇幅冗长。如果不对这些记录加以治理，而是粗暴地将它们一股脑全部作为 Prompt 塞进大模型，将会引发两个严重的工程缺陷：

1. **Token 成本与性能劣化**：模型调用的计费与耗时均与输入 Token 数量呈正相关。历史消息无限膨胀会导致 Token 成本呈指数级攀升，并带来不可接受的推理延迟，甚至直接击穿模型的最大上下文窗口限制（Context Window Limit）。
2. **注意力分散与精度衰减**：长文本中混杂大量无效对话、噪声与过时事实，容易导致模型的注意力机制被稀释（Lost in the Middle），进而输出偏离预期或不够精炼的答案。

上下文工程正是为了解决上述矛盾而生的中枢调度层。它既负责在底层将每一次调用的状态稳定保存为上下文，又负责在向模型递交输入时，执行消息裁剪、摘要压缩、语义过滤等治理策略，使递交给模型的上下文始终保持高密度与高精准度。

```text
+-------------------------------------------------------------+
|                      用户与智能体交互流                      |
+-------------------------------------------------------------+
                               |
                               v
+-------------------------------------------------------------+
|    记忆层 (Memory Storage)                                  |
|    - 原始历史消息、工具执行轨迹、状态流变 (Raw History)       |
+-------------------------------------------------------------+
                               |
                               v
+-------------------------------------------------------------+
|    上下文工程 (Context Engineering)                         |
|    - 消息截断 (Trimming)                                    |
|    - 滚动摘要 (Summarization)                               |
|    - 语义检索过滤 (Filtering)                               |
+-------------------------------------------------------------+
                               |
                               v
+-------------------------------------------------------------+
|    大语言模型推理核心 (LLM Engine)                           |
|    - 接收精简、高密度、高相关的上下文并输出高质量响应       |
+-------------------------------------------------------------+
```
上图展示了从底层原始记忆沉淀到中间上下文工程治理，再到最终递交大模型推理的完整分层架构。

#### 上下文类型与底层抽象对象

智能体技术全面向图状态机架构（LangGraph）演进后，上下文工程所纳管的场景被提炼为三类典型形态，分别对应框架层不同的抽象对象：

| 上下文类型 | 语义定义与演变规律 | 典型包含内容 | 框架底层承载对象 |
| :--- | :--- | :--- | :--- |
| **动态运行时上下文** | 单次运行或单次会话内动态演变、持续修改的可变状态 | 历史消息列表、中间工具调用结果、执行步骤计数器 | `state` 对象 |
| **跨会话上下文** | 超越单次会话线程、在不同会话及多轮运行间长期共享的数据 | 用户全局偏好、历史认知洞察、长期知识条目 | `store` 对象 |
| **静态运行时上下文** | 短时间内相对固定、通常只读或极低频变更的环境元数据 | 用户基础元数据、已注册工具清单、数据库连接凭证与 URL | `context` 对象 |

在上述三类划分中，动态运行时上下文直接映射为智能体的短期记忆机制，跨会话上下文映射为长期记忆机制，而静态运行时上下文主要用于运行期依赖注入。

#### 短期记忆与长期记忆的本质区别

关于短期记忆与长期记忆，开发者极易陷入一个普遍存在的认知误区。

> **易错点**：绝不能按照物理存储介质将记忆划分为“短期记忆对应内存存储，长期记忆对应数据库持久化”！这种理解是完全错误的。存储介质（内存还是外部介质）与记忆的作用域（线程内还是跨线程）是两个正交维度的概念。

根据权威规范定义，短期与长期记忆的本质分水岭在于**作用域范围（Scope）**：

- **短期记忆（Short-term Memory）**：亦称为**会话级记忆（Thread-scoped Memory）**。它的可见性被严格限制在单次会话线程（Thread）内部。只要处于同一个线程内，上下文即可持续流转；一旦开启了一个新的会话线程，该记忆就对新线程完全不可见。在实现介质上，短期记忆既可以使用内存进行极速读写（如开发测试常用的 `InMemorySaver`），也可以使用外部数据库实现防丢失的持久化（如生产环境的 `PostgresSaver` 或 `SqliteSaver`）。
- **长期记忆（Long-term Memory）**：亦称为**跨会话记忆（Cross-session Memory）**。它的可见性超越了单个会话线程，允许多个相互隔离的会话线程共同读写全局共享的数据。在实现介质上，长期记忆同样既有内存存储的实现，也有基于 PostgreSQL 等外部数据库的落地方案。

```text
+-------------------------------------------------------------------------+
|                              短期记忆 (会话级)                          |
|                       Scope: Thread-Scoped (以线程为界)                 |
|       +----------------------------+   +----------------------------+   |
|       |    基于内存实现            |   |    基于外部存储实现        |   |
|       |    (如 InMemorySaver)      |   |    (如 PostgresSaver)      |   |
|       +----------------------------+   +----------------------------+   |
+-------------------------------------------------------------------------+

+-------------------------------------------------------------------------+
|                              长期记忆 (跨会话级)                        |
|                       Scope: Cross-Session (全局共享)                   |
|       +----------------------------+   +----------------------------+   |
|       |    基于内存实现            |   |    基于外部数据库实现      |   |
|       |    (如 InMemoryStore)      |   |    (如 PostgresStore)      |   |
|       +----------------------------+   +----------------------------+   |
+-------------------------------------------------------------------------+
```
上图直观展示了记忆分类的真正维度：横向由会话作用域划分为短期与长期，纵向由存储介质划分为内存级与外部持久化级。

在早期版本中，框架曾提供大量以 `XXXMemory` 命名的封装类。但在现代智能体架构中，所有底层的状态流变与检查点机制均统一构建在 LangGraph 之上，以 `state`（会话内状态）与 `store`（跨会话存储）两套纯粹的原语作为事实标准。

---

### 基于内存的短期记忆实现与状态观测

#### 短期记忆的三大核心支柱

现代智能体体系中，要让短期记忆正确运转，必须由以下三项核心要素协同支撑：

1. **状态（`state`）**：会话内部的核心数据载体。在智能体中，它默认维护着对话消息的历史列表（`messages`）。
2. **检查点（`checkpointer`）**：某一特定执行时刻 `state` 的完整状态快照。它的机制类似于单机 RPG 游戏中的自动存档系统：系统在执行的关键节点自动将当前状态冻结存档，后续进入游戏时即可从上一个存档点无缝恢复。
3. **线程标识（`thread_id`）**：划定会话作用域边界的唯一键值。框架通过 `thread_id` 判定哪些调用属于同一上下文。相同的 `thread_id` 共享同一个检查点演进链条，不同的 `thread_id` 之间彼此物理隔离。

#### 内存检查点四步实现范式

在代码中基于内存实现短期记忆，需要遵循严格的四步工程范式：

- **第一步：实例化内存级持久化器**。创建 `InMemorySaver` 实例，该对象将在内存中开辟存储空间，担任检查点管理器。
- **第二步：将检查点注入智能体构造器**。在调用 `create_agent` 时，显式将持久化器赋值给 `checkpointer` 参数，赋予智能体状态留存能力。
- **第三步：构建会话配置字典**。构造携带 `thread_id` 的 `config` 字典，明确声明当前交互归属的具体会话线程。
- **第四步：在执行调用时透传配置**。调用 `agent.invoke` 时，通过 `config` 关键字参数将配置传入，确保框架在执行前后正确读写对应的检查点。

```python
from langchain.agents import create_agent
from langchain_core.messages import HumanMessage
from langgraph.checkpoint.memory import InMemorySaver
from rich import print as rprint

# 第一步：实例化内存级持久化器
checkpointer = InMemorySaver()

# 第二步：将 checkpointer 传入智能体创建方法
agent = create_agent(
    model=model,
    checkpointer=checkpointer
)

# 第三步：构建包含 thread_id 的 config 配置对象
config = {"configurable": {"thread_id": "1"}}

# 第四步：在 invoke 调用中透传 config
# 第一轮对话：注入身份信息
response_round_1 = agent.invoke(
    {"messages": [HumanMessage(content="你好，我叫张三。")]},
    config=config
)
print("第一轮回复：", response_round_1["messages"][-1].content)

# 第二轮对话：在同一个 thread_id 下发起身份追问
response_round_2 = agent.invoke(
    {"messages": [HumanMessage(content="我叫什么？")]},
    config=config
)
print("第二轮回复：", response_round_2["messages"][-1].content)
```

在上述代码中，当第二轮执行 `agent.invoke` 时，由于传入的 `config` 与上一轮保持完全一致的 `thread_id: "1"`，模型能够立即且明确地回答“你叫张三”。

#### 会话连续性与跨线程隔离验证

为了验证短期记忆的内部机理与隔离边界，我们可以通过 `agent.get_state(config)` 深入观测底层的快照结构，并进行跨轮次问答与跨线程测试。

```python
# 观测当前 thread_id = "1" 的底层状态快照
state_thread_1 = agent.get_state(config)
rprint(state_thread_1)
```

通过 `rprint` 打印输出的快照字典中，可以清晰看到 `values` 字段下的 `messages` 列表。此时该列表中包含整整 4 条消息：第一轮的用户输入、第一轮的 AI 回复、第二轮的用户提问，以及第二轮的 AI 回复。正是由于底层维护了这份完整的历史列表，模型才能获得连贯的记忆上下文。

```python
# 第三轮对话：测试基于前序对话过程的元认知问题
response_round_3 = agent.invoke(
    {"messages": [HumanMessage(content="我刚才问了什么问题？")]},
    config=config
)
print("第三轮回复：", response_round_3["messages"][-1].content)

# 再次查看状态，验证消息列表的递增累加
state_thread_1 = agent.get_state(config)
print("当前消息总条数：", len(state_thread_1.values["messages"]))  # 输出 6 条
```

在第三轮问答中，模型给出的回复为“你刚才问我：我叫什么”，再次准确捕捉了对话历史。此时状态快照中的消息记录已线性累加至 6 条。

接下来，我们构建一个全新的 `thread_id` 发起调用，验证会话隔离性：

```python
# 第四轮对话：切换全新的 thread_id 构建配置
config_new_thread = {"configurable": {"thread_id": "2"}}

response_round_4 = agent.invoke(
    {"messages": [HumanMessage(content="我叫什么？")]},
    config=config_new_thread
)
print("新会话回复：", response_round_4["messages"][-1].content)

# 观测 thread_id = "2" 的底层快照
state_thread_2 = agent.get_state(config_new_thread)
print("新会话消息总条数：", len(state_thread_2.values["messages"]))  # 仅输出 2 条
```

执行结果显示，在新会话线程中，模型给出的回复是“对不起，我不知道你的名字”。通过 `agent.get_state(config_new_thread)` 打印其内部快照，可以观察到其中仅有属于线程 2 自身的 2 条消息记录，线程 1 中积累的 6 条历史记录对线程 2 而言完全处于不可见的物理隔离状态。这充分证明了短期记忆以 `thread_id` 为边界的线程作用域隔离机制。

---

### 内存存储工作原理与常见工程问题

#### 内存检查点数据流转机理

当挂载了 `InMemorySaver` 的智能体被调用时，其底层的数据流转与状态推进遵循严谨的五步闭环链条：

1. **历史检索**：智能体接收到调用请求后，首先根据 `config` 中的 `thread_id`，从 `InMemorySaver` 维护的哈希结构中定位属于该线程的最新状态快照；
2. **消息追加**：将当前调用中传入的最新 `HumanMessage` 追加至读取出的历史 `messages` 列表末尾；
3. **模型推理**：将组装完毕的完整消息列表整体递交给底层大模型进行概率推演；
4. **接收响应**：模型完成推理计算，输出一个全新的 `AIMessage` 响应对象；
5. **写回快照**：系统将生成的 `AIMessage` 再次追加至消息列表，并在 `InMemorySaver` 中为当前线程写入一个全新的 `Checkpoint` 状态快照，等待下一次调用唤醒。

```text
+--------------------------------------------------------------------------+
| 步骤 1: 用户发起 invoke(messages=[HumanMessage], config={thread_id: "1"})|
+--------------------------------------------------------------------------+
                                    |
                                    v
+--------------------------------------------------------------------------+
| 步骤 2: 读取检查点 (从 InMemorySaver 中按 thread_id 提取已有历史消息)    |
+--------------------------------------------------------------------------+
                                    |
                                    v
+--------------------------------------------------------------------------+
| 步骤 3: 追加当前消息 (将最新的 HumanMessage 拼入消息序列末尾)             |
+--------------------------------------------------------------------------+
                                    |
                                    v
+--------------------------------------------------------------------------+
| 步骤 4: 递交模型推理 (将全量拼接后的消息序列发送至大语言模型)            |
+--------------------------------------------------------------------------+
                                    |
                                    v
+--------------------------------------------------------------------------+
| 步骤 5: 存盘快照写回 (将模型返回的 AIMessage 存入检查点，生成新 State)    |
+--------------------------------------------------------------------------+
```
上图展示了带有检查点的智能体在单次调用生命周期内完整的读取、组装、推理与写回流转过程。

#### 隔离维度与业务应用场景

理解基于 `thread_id` 的隔离机制，是构建生产级企业智能体系统的基石。在真实的业务架构中，`thread_id`（通常为字符串格式）通常承担以下两类典型场景的边界划分：

- **多用户并发会话隔离**：系统同时为海量终端用户提供问答服务。用户 Alice 拥有唯一的会话标识 `thread_id: "user_alice_session_001"`，用户 Bob 拥有标识 `thread_id: "user_bob_session_002"`。两人与智能体交互的历史记录各自单向累加，在底层内存中互不穿透，确保多租户数据安全与隐私隔离。
- **单用户多业务任务隔离**：即便是同一个用户，其在智能体系统内也可能并发执行不同的工作流。例如用户当前既在让智能体编写 Python 爬虫（`thread_id: "task_coding"`），又在让智能体整理技术文档（`thread_id: "task_writing"`）。将两类任务的 `thread_id` 显式解耦，可以避免写代码的技术上下文对文档撰写产生混乱干扰，使各个细分任务的推理状态保持高度纯净。

#### 常见问题与排错指南

在实际编码与调试过程中，基于内存的持久化器通常容易出现以下四类典型故障：

> **易错点**：**为什么智能体完全记不住之前的对话？**
> 1. **未创建或未传入 checkpointer**：仅在代码中实例化了 `InMemorySaver`，但未在 `create_agent` 的入参中指定 `checkpointer=checkpointer`；或者干脆遗漏了持久化器的创建。
> 2. **调用时遗漏 config**：虽然智能体注册了检查点，但在执行 `agent.invoke()` 时没有传递 `config` 参数，框架无法定位当前的会话归属。
> 3. **thread_id 发生跳变**：多轮调用中传递的 `config` 字典内，`thread_id` 拼写错误或取值动态变化，导致系统不断在开辟全新会话而非追加历史。

> **结论**：**`InMemorySaver` 会丢失数据吗？**
> **必然会**。`InMemorySaver` 顾名思义仅将快照保存在当前 Python 运行进程的宿主内存堆中。一旦当前进程退出、程序异常崩溃或重启，内存中的全部数据将彻底蒸发。此外，部署在多进程（如 Gunicorn / Uvicorn 多 Worker 模式）或分布式微服务环境下的智能体，多个独立进程之间无法共享本地内存，用户前后两次请求若被负载均衡路由到不同节点，记忆也会瞬间断裂。

> **注意**：**内存开销与上下文窗口是否会无限制增长？**
> 如果不对会话历史进行治理，每一次 `invoke` 都会导致消息列表无休止地递增追加。随之而来的是：
> 1. 提交给模型的 Prompt Token 数量暴增，API 计费呈阶梯式上涨；
> 2. 模型接口调用耗时大幅拉长，甚至因超出 LLM 最大上下文 Token 阈值而抛出异常崩溃。
> 生产环境必须配合上下文管理策略（如限制最近 $N$ 条滑动窗口裁剪、定期触发总结摘要合并）进行动态截流。

> **提示**：**如何主动清空或重置特定会话的历史？**
> 1. **业务逻辑层切换**：最轻量优雅的姿势是生成一个新的 `thread_id`（例如基于 UUID 重新派发），原先的旧线程自然被废弃不再访问。
> 2. **实例级别重构**：若要在当前内存中彻底抛弃所有历史，可以为智能体重新构造并挂载一个全新的 `InMemorySaver` 实例。

---

### 云端持久化基础设施部署

#### 外部存储介质演进与环境选型

既然基于内存的 `InMemorySaver` 存在进程生命周期短暂、易随重启丢失、无法跨进程跨机器共享等致命缺陷，那么将存储介质从易失性 RAM 升级为高可靠的外部持久化存储，就成为走向生产环境的必经之路。

在 LangGraph 官方提供的持久化方案矩阵中，针对短期记忆与长期记忆均有标准化的外部组件支持。对于短期会话级记忆，生产中最具代表性的是基于 PostgreSQL 数据库构建的 `PostgresSaver`。

要让智能体能够将状态快照安全落盘到 PostgreSQL 中，我们必须首先准备一套稳定可用的 PostgreSQL 运行环境。虽然开发者可以在本地 Windows 操作系统上直接安装单机版数据库，但在现实的商业化项目与工程团队中，几乎所有服务端应用与数据库均承载于 Linux 系统之上。为了抹平学习环境与工业实践之间的鸿沟，我们采用公有云（以腾讯云为例）部署 Linux 云服务器，作为后续承载数据持久化的基础设施。

#### 腾讯云 Linux 服务器选型与配置

在云厂商控制台中部署一台用于开发学习与测试的轻量级 Linux 服务器，可以遵循如下选型与配置流：

1. **产品与实例选型**：
   进入腾讯云控制台的“云服务器 CVM”板块，点击“立即选购”。在计费模式中选择“自定义配置”下的**竞价实例（Spot Instance）**。竞价实例在保证标准云服务器完整功能的前提下，价格极为低廉（本例中约 0.052 元/小时），非常适合个人学习与技术验证。
2. **机型与硬件配置**：
   - 地域选择：就近选择机房（例如南京等可用区）；
   - CPU 与内存：选择 **2 核 CPU、4GB 内存**的通用算力机型。这是流畅运行现代 Linux 发行版与部署 PostgreSQL 数据库的基础底线配置；
   - 处理器架构：选用标准 x86 架构。
3. **操作系统镜像**：
   公共镜像选择经典稳定的 **Ubuntu Server 24.04 LTS（64 位）**版本，系统盘配置 50GB 高性能云硬盘。
4. **网络与公网暴露配置**：
   - 必须确保勾选**“分配独立公网 IP”**，网络计费选择按流量计费并调满带宽峰值，确保具备公网可寻址能力；
   - 安全组（Security Group）：先绑定默认放通全部内网与常见 SSH 端口的基础安全组；
   - 实例命名与凭据：实例名称设置为 `langchain1.2-demo`，默认登录用户为 `ubuntu`，密码按规范设置为强密码（演示使用统一密码 `abcd_1234`）。

确认选购信息后勾选风险条款并确认开通。等待云平台后台调度分配硬件资源，片刻之后实例即进入“运行中”状态，并在控制台面板中分配到唯一的公网 IPv4 地址。

#### SSH 远程终端连接

服务器就绪后，我们使用本地终端工具（如 Xshell、FinalShell 或原生 OpenSSH 终端）建立远程连接会话：

```bash
# 原生 OpenSSH 命令行连接示例（替换为你自己的公网 IP）
ssh ubuntu@<云服务器公网IP>
```

若使用 Xshell 等 GUI 工具：
- 新建会话，协议选择 `SSH`；
- 主机（Host）填入云主机的公网 IP，端口号保持默认的 `22`；
- 认证环节输入用户名 `ubuntu`，密码填入创建时设定的 `abcd_1234`；
- 点击连接并选择“接受并保存”主机密钥指纹。

终端光标闪烁并打印出 Ubuntu 系统的欢迎信息时，即代表本地与云端 Linux 环境的基础设施链路已完全打通。

---

### PostgreSQL 数据库安装与权限配置

#### 数据库安装与服务生命周期验证

连接至云服务器终端后，我们在 Ubuntu 操作系统中执行标准 APT 包管理工具的安装流程。

```bash
# 1. 更新本地软件包索引缓存
sudo apt update

# 2. 安装 PostgreSQL 数据库内核及其核心扩展包（交互提示输入 Y 确认）
sudo apt install postgresql postgresql-contrib -y
```

安装完成后，依次校验数据库的版本、运行状态及端口绑定：

```bash
# 检查当前安装的 PostgreSQL 版本号
psql --version
```
命令输出显示 PostgreSQL 的具体发行版信息（如 PostgreSQL 16.x）。

```bash
# 查看 PostgreSQL 守护进程的运行状态
sudo systemctl status postgresql
```
检查输出终端中是否存在绿色的 `active (running)` 状态标识。若处于激活状态，表明数据库服务在安装后已由 systemd 自动拉起，具备服务承载能力。

```bash
# 查看系统端口占用情况，确认 5432 端口状态
sudo ss -tlpn | grep 5432
```
PostgreSQL 的官方默认通信端口为 **5432**。通过端口排查命令可以明确查看到 PostgreSQL 服务已在后台牢牢绑定了 5432 端口。

#### 专属用户与业务数据库初始化

为契合最小特权原则并保障数据隔离，智能体系统不能直接使用默认的超级管理员 `postgres` 进行日常业务读写，必须在数据库内部创建专属的业务用户与业务数据库。

```bash
# 以 postgres 系统管理员身份进入 psql 交互式控制台
sudo -u postgres psql
```
执行后终端提示符转变为 `postgres=#`，代表已成功进入数据库管理终端。在此执行标准 SQL 指令完成用户与库的初始化：

```sql
-- 1. 创建智能体专属业务用户，并设定访问密码
CREATE USER langchain_user WITH PASSWORD 'abcd1234';

-- 2. 创建用于存储智能体持久化状态的业务数据库，并将所有者指定为该用户
CREATE DATABASE langchain_db OWNER langchain_user;

-- 3. 将该数据库上的所有操作特权全面授予该业务用户
GRANT ALL PRIVILEGES ON DATABASE langchain_db TO langchain_user;
```

SQL 执行完毕后，使用元命令对元数据状态进行核查：

```text
-- 查看当前实例下的所有数据库列表及所有权属关系
\l
```
终端将渲染出一张数据库属性矩阵，确认 `langchain_db` 对应的 `Owner` 列已被准确指派为 `langchain_user`。核验无误后退出管理界面：

```text
-- 退出 psql 控制台返回 Linux Bash 终端
\q
```

#### 本地连接串验证测试

在退出超级管理员控制台后，我们需要使用刚创建的业务身份，通过完整的数据库连接统一资源定位符（Database URL）在服务器本机发起闭环测试。

```bash
# 使用标准连接字符串在本机访问新建的数据库
psql postgresql://langchain_user:abcd1234@localhost:5432/langchain_db
```

对该测试连接串的各个核心字段做逐一解析：

- `postgresql://`：标准协议头标识，声明使用 PostgreSQL 驱动连接；
- `langchain_user`：先前创建的具有合法读写权限的业务用户名；
- `abcd1234`：该用户的身份鉴权密码；
- `localhost`：主机地址。由于当前是在云主机本身的 Shell 终端内发起测试，请求属于本地回环链路，因此指定为 `localhost`；
- `5432`：PostgreSQL 默认监听端口；
- `langchain_db`：连接目标业务数据库。

执行回车后，终端提示符若成功转变为 `langchain_db=>`，即宣告 PostgreSQL 数据库的核心安装、用户开辟、数据库初始化与权限分配在本地层面已完全调通。

---

### 远程网络访问放通与安全配置

#### 跨网络远程连接的三大阻碍

虽然我们在云服务器本地使用 `localhost` 能够瞬时连通数据库，但如果此时急躁地直接将开发环境（如本地 PyCharm、Jupyter Notebook 或外网应用）中的连接地址由 `localhost` 替换为云服务器的公网 IP，连接必然会遭遇惨烈的超时或被拒绝报错。

从公网向 Linux 云服务器中的 PostgreSQL 发起直连，必须接连穿透三道层层设防的网络与安全拦截网：

1. **云厂商外围网络屏障（安全组防火墙）**：公有云厂商默认在实例外侧挂载虚拟防火墙，未在控制台规则中放通的非标准端口一律在硬件层直接丢弃数据包；
2. **PostgreSQL 内部监听屏障（`postgresql.conf`）**：出于安全防护设计，PostgreSQL 在出厂默认配置下仅绑定监听本地回环地址（`127.0.0.1` / `localhost`），任何到达物理网卡的外部网络包都不会被其接收；
3. **客户端认证与来源控制屏障（`pg_hba.conf`）**：PostgreSQL 具备独立的基于主机的主机安全认证控制机制（Host-Based Authentication），对于未经白名单授权的远端网段 IP，即使网络层可达，也会在应用层握手阶段直接强行中断握手。

只有按序将上述三道屏障全部解开，外部的智能体客户端才能真正跨越公网与云数据库建立持久化长连接。

#### 云厂商安全组端口放行

针对第一道屏障，必须进入云服务商的网页控制台修改实例外层防护规则：

1. 登录腾讯云控制台，进入对应云服务器的“实例详情”页，切换到“安全组”标签卡；
2. 点击当前绑定的安全组 ID 进入配置面板，切换至“入站规则（Inbound Rules）”；
3. 点击“添加规则”，新增一条端口放行策略：
   - **类型**：自定义；
   - **来源**：填写 `0.0.0.0/0`（代表允许所有来源 IPv4 地址发起握手）；
   - **协议端口**：选择 `TCP` 协议，端口指定为 `5432`；
   - **策略**：允许（Allow）。
4. 确认保存规则，安全组策略即刻全局生效。

通过这一配置，云厂商物理宿主层的网络过滤器便不再拦截发往 5432 端口的 TCP 流量包。

#### 监听地址与客户端认证配置

针对第二道与第三道屏障，我们需要直接修改 Ubuntu 系统中 PostgreSQL 的两份核心底层配置文件。

##### 修改网络监听配置（`postgresql.conf`）

首先定位并编辑主配置文件：

```bash
# 打开 postgresql.conf 配置文件（Ubuntu 下默认位于 /etc/postgresql/<版本号>/main/）
sudo vim /etc/postgresql/16/main/postgresql.conf
```

进入 Vim 后，输入 `/listen_addresses` 执行全文搜索，定位到如下配置行：

```text
#listen_addresses = 'localhost'         # what IP address(es) to listen on;
```

在默认状态下，该项即便取消注释，其绑定的取值也是 `'localhost'`。按下键盘 `i` 键进入插入编辑模式，取消该行开头的注释符 `#`，并将取值修改为全局通配符 `'*'`：

```text
listen_addresses = '*'
```

> **注意**：将 `listen_addresses` 设定为 `'*'` 意味着指示 PostgreSQL 的网络引擎监听宿主机上存在的所有网络适配器（包括内网物理网卡与公网弹性网卡），使其有能力捕获来自外网的数据通信。

修改完成后，按下 `Esc` 退出编辑模式，输入 `:wq` 并回车保存退出。

##### 修改客户端认证控制文件（`pg_hba.conf`）

接下来配置第三道防线中的主机授权策略：

```bash
# 打开 pg_hba.conf 访问控制配置文件
sudo vim /etc/postgresql/16/main/pg_hba.conf
```

进入文件后，按快捷键 `G` 直接跳转至配置文件的最末尾。按下 `i` 键在末行新追加一行授权规则：

```text
host    all             all             0.0.0.0/0               md5
```

对该策略各列的语义做技术解析：
- **`host`**：连接类型，表明该规则作用于普通的 TCP/IP 网络套接字连接（包括未加密与 SSL 加密通信）；
- **`all`（第一处）**：作用的数据库，声明该规则放行用户访问服务器内的任意数据库；
- **`all`（第二处）**：作用的用户身份，声明允许实例内的任意数据库用户进行匹配；
- **`0.0.0.0/0`**：允许接入的客户端 IP 网段掩码，此处配置为全网段放行；
- **`md5`**：身份校验方式，要求客户端在接入时提供通过 MD5/加盐哈希校验的密码进行身份认证。

配置追加完毕后，按下 `Esc`，输入 `:wq` 保存并退出。

#### 服务重载与公网端到端验证

修改上述两份核心配置后，设置并不会自动热加载生效，必须对 PostgreSQL 守护进程执行平滑重载与重启操作：

```bash
# 重启 postgresql 核心服务以重新加载底层配置
sudo systemctl restart postgresql

# 重新核验服务状态，确保重启后状态依然为健康的 active (running)
sudo systemctl status postgresql
```

确认系统服务正常运转后，我们在外部环境（可以在重新打开的终端会话中，或者本地开发机安装了 psql 客户端的环境中）执行端到端终验。将测试命令中的连接地址显式替换为**云服务器真实的公网 IP**：

```bash
# 跨越公网进行数据库连通性终验（将 <云服务器公网IP> 替换为真实的 IPv4 地址）
psql postgresql://langchain_user:abcd1234@<云服务器公网IP>:5432/langchain_db
```

```text
+-------------------------------------------------------------------------+
|                  本地开发环境 (PyCharm / CLI 终端)                      |
+-------------------------------------------------------------------------+
                                     |
                                     | 公网 TCP 请求 (Port: 5432)
                                     v
+-------------------------------------------------------------------------+
| 第一道关卡: 云厂商虚拟防火墙 (安全组开放 TCP 5432 入站)                  |
+-------------------------------------------------------------------------+
                                     |
                                     v
+-------------------------------------------------------------------------+
| 第二道关卡: postgresql.conf 监听配置 (listen_addresses = '*')            |
+-------------------------------------------------------------------------+
                                     |
                                     v
+-------------------------------------------------------------------------+
| 第三道关卡: pg_hba.conf 访问鉴权 (host all all 0.0.0.0/0 md5)            |
+-------------------------------------------------------------------------+
                                     |
                                     v
+-------------------------------------------------------------------------+
| 成功建立数据库连接长会话，就绪承接 PostgresSaver 状态持久化             |
+-------------------------------------------------------------------------+
```
上图总结了从客户端到数据库内核依次穿透三道安全与网络关卡的完整拓扑。

回车执行后，终端成功越过公网验证，并顺利呈现出 `langchain_db=>` 的交互式命令行，即标志着外部持久化基础设施的搭建大功告成。至此，我们不仅在理论层深刻理解了上下文与记忆机制的运作边界，在代码层掌握了基于内存的检查点实现与状态观测，更在基础设施层亲手构筑了企业级的 Linux 与 PostgreSQL 运行底座，为下一阶段将智能体短期会话记忆全面升级为具备生产级持久化能力的 `PostgresSaver` 铺平了道路。

> **承前启后**：上一节讲完「记忆机制原理、短期内存持久化与云端数据库基础设施」，下一节接着讲「PostgreSQL持久化存储、消息治理与长期记忆架构」。

---

## PostgreSQL持久化存储、消息治理与长期记忆架构
> 对应块：BLK20 | 覆盖分集：P92-P97

我们在前面的测试中已经在 Linux 云服务器上顺利安装了 PostgreSQL，并验证了基础连通性。但把数据库跑起来只是第一步，在实际工程中，真正核心的问题在于：我们如何通过代码把智能体的状态从挥发性的进程内存中剥离出来，安全地沉淀到外部持久化存储中？随着多轮交互的不断推进，暴涨的历史消息如何做精细化治理？当会话结束、线程切换后，跨越整个用户生命周期的偏好与经验又该如何通过长期记忆架构进行跨会话沉淀与精准读取？

围绕状态的生命周期演进，我们将系统拆解关系型数据库持久化存储、短期记忆与长期记忆的边界划分、基于中间件的消息治理策略，以及底层元数据在不同介质中的更新机理。

---

### 外部数据库状态持久化与检查点机制

在单机本地测试阶段，我们通常使用保存在内存中的暂存机制。但内存存储无法应对生产环境的服务重启、多实例横向扩展与分布式容灾。将短期记忆接入 PostgreSQL 是实现生产级会话管理的基础操作。

#### 关系型数据库存储连接与环境初始化

LangGraph 在持久化存储层提供了标准化的检查点器接口。针对 PostgreSQL，官方推荐的持久化类为 `PostgresSaver`。在建立数据库连接前，我们需要准备标准的连接字符串，并借助 Python 的上下文管理器（Context Manager）来管理数据库连接的生命周期。

```python
from langgraph.checkpoint.postgres import PostgresSaver

# 数据库连接 URI，格式为 postgresql://用户名:密码@主机地址:端口/数据库名
DB_URI = "postgresql://postgres:password@192.168.1.100:5432/langgraph_db"

with PostgresSaver.from_conn_string(DB_URI) as checkpointer:
    # 初始化数据库环境，创建底层必须的元数据表
    checkpointer.setup()
```

在这段初始化代码中，有两处关键的设计原则：

1. **上下文管理器语法**：必须使用 `with PostgresSaver.from_conn_string(DB_URI) as checkpointer:` 语法。它能确保在程序执行异常或进程退出时，数据库连接池资源能够被优雅释放，防止连接泄露导致数据库连接池耗尽。
2. **幂等性建表调用**：`checkpointer.setup()` 是一个至关重要的初始化方法。它的核心作用是向目标数据库中写入 LangGraph 运行时所必需的数据表结构。该方法内部具备幂等性判断：首次执行时，如果数据库中不存在相关表，它会自动执行建表 SQL（`CREATE TABLE IF NOT EXISTS`）；若表已存在，则静默跳过，不会重复执行或破坏已有数据。

> **提示**：在后续的实际生产部署中，`setup()` 方法通常在服务启动的初始化脚本中执行一次即可，后续的日常运行请求直接复用已建好的数据表结构。

#### 检查点写入与跨调用状态重载验证

完成检查点器的实例化与建表初始化后，我们即可将其装配进智能体中。智能体在执行任务时，依赖配置字典中的 `thread_id` 字段来标识当前的会话线程。

```python
from langchain_core.messages import HumanMessage
from langgraph.prebuilt import create_react_agent

# 构建智能体并将外部持久化检查点器注入
agent = create_react_agent(
    model=model,
    tools=[],
    checkpointer=checkpointer
)

# 为当前交互分配会话线程标识
config = {"configurable": {"thread_id": "1"}}

# 第一次交互：提供身份信息
response1 = agent.invoke(
    {"messages": [HumanMessage(content="你好，我是康师傅")]},
    config=config
)
for m in response1["messages"]:
    m.pretty_print()

# 第二次交互：测试短期记忆加载
response2 = agent.invoke(
    {"messages": [HumanMessage(content="你知道我是谁吗？")]},
    config=config
)
for m in response2["messages"]:
    m.pretty_print()
```

执行上述逻辑时，底层的状态流转如下：

```text
+-----------------------+     invoke (thread_id="1")     +-----------------------+
|  HumanMessage("你好") | -----------------------------> | Agent 执行推理生成响应 |
+-----------------------+                                +-----------------------+
                                                                     |
                                                           保存 State 快照
                                                                     v
                                                         +-----------------------+
                                                         | PostgreSQL 数据表     |
                                                         | (checkpoints)         |
                                                         +-----------------------+
                                                                     |
+-----------------------+     invoke (thread_id="1")                 |
| HumanMessage("我是谁")| -----------------------------> +-----------------------+
+-----------------------+   根据 thread_id 加载历史快照  | Agent 结合历史记忆响应 |
                                                         +-----------------------+
```

读图说明：每次通过 `invoke` 发起交互并携带相同的 `thread_id` 时，智能体首先从 PostgreSQL 的 `checkpoints` 数据表中拉取该线程的历史状态快照，将其注入到当前推理上下文，最后将更新后的消息全量快照写回数据库。因此在第二次提问时，模型无须显式重传上一轮输入即可准确识破用户身份。

#### PostgreSQL底层存储拓扑与模式分区检查

为了搞清楚持久化数据究竟存放在数据库的何处，我们需要深入到 PostgreSQL 底层的存储拓扑中进行结构审查。PostgreSQL 的数据组织遵循三层命名空间体系：

```text
+--------------------------------------------------------------+
| 数据库 (Database: 如 langgraph_db)                           |
|   +--------------------------------------------------------+ |
|   | 模式分区 (Schema: 如 public)                           | |
|   |   +--------------------------------------------------+ | |
|   |   | 数据表 (Tables: checkpoints, writes 等)          | | |
|   |   +--------------------------------------------------+ | |
|   +--------------------------------------------------------+ |
+--------------------------------------------------------------+
```

读图说明：最外层为独立的物理逻辑数据库，向下划分模式分区（Schema），最内层承载具体的实体业务表。

在 Linux 终端通过 `psql` 命令行工具进入 `langgraph_db` 数据库后，可以通过以下元命令进行层级检索：

1. **查看数据库清单**：
   输入 `\l`，输出中包含 `langgraph_db` 数据库，确认数据库创建成功。
2. **查看模式分区（Schema）**：
   输入 `\dn`，输出显示当前仅包含默认的 `public` 分区。
   也可以执行 SQL 语句进行精准验证：
   ```sql
   SELECT current_schema();
   ```
   返回结果为 `public`，证明当前所有的会话操作均落在该模式下。
3. **查看数据表清单**：
   在 `public` 模式下输入 `\dt`（`t` 即 table 的缩写），终端将列出系统初始化创建的四张核心表。

在生成的四张表中，核心表为 `checkpoints`。

> **定义**：`checkpoints` 是短期记忆的主表，用于持久化记录每一个 `thread_id` 在特定版本时序下的状态快照（State Snapshot）。它记录了线程编号、检查点版本号（Checkpoint ID）、父检查点标识以及序列化后的二进制或 JSON 格式状态载荷。

---

### 短期记忆的内存与数据库存储机制对比

不少人容易混淆“代码重跑”与“会话重载”的区别。为了厘清持久化存储与内存存储在生命周期上的根本边界，我们设计两组严格对照的实验。

#### 进程生命周期与内存状态挥发实验

首先考察基于 `InMemorySaver` 的内存短期记忆。

```python
from langgraph.checkpoint.memory import InMemorySaver
from langchain_core.messages import HumanMessage
from langgraph.prebuilt import create_react_agent

checkpointer = InMemorySaver()
agent = create_react_agent(model=model, tools=[], checkpointer=checkpointer)

config = {"configurable": {"thread_id": "1"}}

# 第一轮运行内的三次交互
agent.invoke({"messages": [HumanMessage(content="你好，我是谁？")]}, config=config)
agent.invoke({"messages": [HumanMessage(content="我是老王")]}, config=config)
agent.invoke({"messages": [HumanMessage(content="你好，我是谁？")]}, config=config)
```

在这一次进程运行过程中，第三次调用准确输出了“你是老王”，此时内存状态累积了 6 条消息（3 条人类提问与 3 条 AI 回复）。

现在保持 `thread_id = "1"` 完全不变，停止当前 Python 进程并重新点击运行：
- 第一步重新提问：“你好，我是谁？”
- 运行结果：模型回复“我不清楚你的身份”。
- 消息数量变化：第一次交互后为 2 条，第二次交互后为 4 条，第三次交互后为 6 条。

> **结论**：`InMemorySaver` 将状态完全维护在当前 Python 进程的堆内存中。一旦进程终止、脚本重启，或在代码中重新实例化了 `InMemorySaver` 对象，旧有的历史状态便彻底烟消云散。

#### 数据库连接复用与跨进程状态累加验证

对比之下，我们切换为 `PostgresSaver` 并设置一个全新线程 `thread_id = "3"`。

第一轮执行：
1. 发送“你好，我是谁？”，模型表示未知（2 条消息）；
2. 发送“我是老王”，模型表示已记录（4 条消息）；
3. 发送“你好，我是谁？”，模型回答“你是老王”（6 条消息）。

此时终止该 Python 进程。保持 `thread_id = "3"` 不做任何修改，重新启动脚本运行：
- 现象：第一轮运行产生的 6 条历史消息并没有随进程销毁。第二轮启动后，代码虽然重新构建了 `PostgresSaver` 实例，但该实例指向的物理数据库表完好无损。
- 刚进入第二轮第一次调用并提问“你好，我是谁？”，模型直接回答：“你好老王！”
- 消息计数变化：第二轮第一次调用完成后，系统消息量直接在原有 6 条基础上累加至 8 条（6 + 2）；第二次调用累加至 10 条；第三次调用累加至 12 条。

只要 `thread_id` 保持一致且未显式执行清理指令，每一次进程启动都会无缝衔接上一进程的历史数据并在线性维度持续累加。

#### 状态生命周期与持久化边界准则

| 维度 | InMemorySaver（内存存储） | PostgresSaver（外部关系型存储） |
| :--- | :--- | :--- |
| **存储介质** | 本地内存堆栈 | 外部 PostgreSQL 关系型数据库 |
| **进程退出表现** | 状态彻底挥发丢失 | 数据永久保存在磁盘与 WAL 日志中 |
| **实例重建表现** | 生成全新空存储，历史无法找回 | 重新建立连接池，按 `thread_id` 即刻还原 |
| **网络与 IO 开销** | 纯内存指针寻址，零网络 IO | 存在网络往返与数据库事务开销 |
| **典型适用场景** | 本地单元测试、轻量原型开发、一次性会话 | 生产环境微服务、跨实例共享会话、分布式部署 |

---

### 对话消息的生命周期治理策略

在 PostgreSQL 的支持下，会话状态可以永久留存并持续累加。但这种“无限累加”很快会引发严重的反噬。如果对历史消息听之任之，智能体系统将迅速滑向崩溃边缘。这就是上下文工程（Context Engineering）中不可或缺的消息治理。

#### 上下文累积带来的工程挑战

随着对话轮次的推进，存储在 `state["messages"]` 中的条目不断膨胀，给模型运行带来三重致命打击：

1. **上下文窗口硬限制被撑爆**：所有商业大模型均有严格的上下文窗口上限（如 4K、8K、32K、128K Token）。一旦历史消息长度超过模型阈值，接口将直接抛出超长异常，导致业务链路中断。
2. **注意力稀释与回复劣质化**：即使长文本模型在物理容量上能容纳庞大的消息列表，过多不相关、陈旧甚至互相冲突的冗余对话也会严重分散模型的注意力机制，导致回答的精确度断崖式下跌。这与 RAG 检索系统的逻辑如出一辙——检索召回成百上千个分块后，必须借助 Rerank 模型挑选排名前列、最相关的内容送入模型，而非一股脑全量灌入。
3. **Token 计费成本线性飙升**：大模型按输入与输出的 Token 总量计费。每次简单的日常寒暄都需要将此前数十轮历史全量打包计费，会导致 API 调用账单剧烈增加。

为此，LangGraph 提供了消息裁剪、消息删除与消息摘要三种核心治理策略。

#### 基于前置中间件的消息裁剪策略

消息裁剪的核心逻辑是：**在模型被调用之前（Before Model），有选择性地截断传入的上下文消息列表**。这种截断只控制送进模型计算的临时视图，不影响历史存储库的完整性。

我们利用 LangGraph 提供的 `@before_model` 装饰器编写裁剪中间件：

```python
from langgraph.prebuilt import before_model
from langgraph.prebuilt.chat_agent_executor import AgentState
from langchain_core.messages import RemoveMessage

@before_model
def trim_messages(state: AgentState, runtime) -> dict | None:
    messages = state["messages"]
    # 消息条数少于等于 3 条时，无需裁剪
    if len(messages) <= 3:
        return None
    
    # 保留初始系统设定或第一条首发消息
    first_message = messages[0]
    
    # 动态切片策略：偶数保留最近 3 条，奇数保留最近 4 条
    if len(messages) % 2 == 0:
        recent_messages = messages[-3:]
    else:
        recent_messages = messages[-4:]
        
    new_messages = [first_message, *recent_messages]
    
    # 清空上下文重排视图并写入新切片
    return {"messages": [RemoveMessage(id="__REMOVE_ALL_MESSAGES__"), *new_messages]}
```

为了彻底理解消息裁剪的时序推演，我们跟踪四次连续 `invoke` 下的推导链条：

```text
[第 1 轮 invoke]
输入: H1
进入 before_model 视图: [H1] (长度 1 <= 3) -> 不触发裁剪
模型响应并沉淀: [H1, A1] (总计 2 条)

[第 2 轮 invoke]
输入: H2 ("从现在起叫我小王")
进入 before_model 视图: [H1, A1, H2] (长度 3 <= 3) -> 不触发裁剪
模型响应并沉淀: [H1, A1, H2, A2] (总计 4 条)

[第 3 轮 invoke]
输入: H3 ("今天天气不错")
进入 before_model 视图: [H1, A1, H2, A2, H3] (长度 5 为奇数)
裁剪策略: 取首条 H1 与末尾 4 条 [-4:] -> 保留 [H1, A1, H2, A2, H3]
计算结果: 5 条全部保留，未产生实际丢弃
模型响应并沉淀: [H1, A1, H2, A2, H3, A3] (总计 6 条)

[第 4 轮 invoke]
输入: H4 ("告诉我你是谁，我是谁")
进入 before_model 视图: [H1, A1, H2, A2, H3, A3, H4] (总计 7 条，奇数)
裁剪策略: 取首条 H1 与末尾 4 条 [A2, H3, A3, H4]
丢弃条目: 中间的 A1 与 H2 遭到精准切除！
送入上下文: [H1, A2, H3, A3, H4] (共 5 条)
模型响应并输出: A4
```

读图推演说明：在第四轮交互发起时，传入模型的历史记录被修剪为最初的身份说明（H1）以及最近的两轮问答，成功避开了中间无关闲聊对模型注意力的干扰。

#### 基于后置中间件的消息删除策略

裁剪主要着眼于“模型调用前的上下文受控”，而消息删除则是在**模型调用完成之后（After Model），直接对状态列表中的冗余消息执行移除操作**。

```python
from langgraph.prebuilt import after_model
from langgraph.prebuilt.chat_agent_executor import AgentState
from langchain_core.messages import RemoveMessage

@after_model
def delete_messages(state: AgentState, runtime) -> dict | None:
    messages = state["messages"]
    # 当消息总数未达到 5 条时，不执行清理
    if len(messages) <= 5:
        return None
        
    # 计算需要剥离的陈旧消息数量
    to_delete = len(messages) - 5
    
    # 针对超出部分的消息 ID，分发墓碑删除指令
    return {"messages": [RemoveMessage(id=m.id) for m in messages[:to_delete]]}
```

在第四轮交互中，如果不启用删除策略，总消息数应为 8 条；而在 `after_model` 的干预下，系统仅保留最后 5 条。

这里存在一个极易踩坑的现象：若在首轮交互中用户输入了“我是老王”，而模型仅仅客套性回复了“你好”，随着删除中间件将最早的三条消息彻底剔除，第四轮提问“我是谁”时，模型将无法回答。因为包含“老王”这一事实的原始提问与回复已经被清理出状态。只有当第一轮模型的回复中显式包含“你好老王”且该回复条目在保留的窗口内时，模型才能获悉用户信息。

> **底层机制**：`RemoveMessage` 的底层工作原理与墓碑标记（Tombstone）。
> 当我们向状态返回 `RemoveMessage(id=m.id)` 时，LangGraph 底层并不会立刻执行 SQL 的物理 `DELETE` 语句去抹除数据库行。相反，它向消息流中追加了一条携带墓碑标记的特殊记录。当后续重新拉取状态时，回放机制（State Replay）会将原始消息与其对应的墓碑记录进行合并抵消，从而在向外输出时将该消息隐藏过滤掉。在逻辑体感上消息已被删除，但在底层存储中依然保留着完整的审计踪迹。

#### 双模型架构下的自动消息摘要

裁剪与删除虽然直接有效，但都会不可避免地导致部分历史信息彻底丢失。对于需要长程跟踪关键线索的业务场景，消息摘要（Summarization）是更为稳妥的架构方案。

摘要策略采用**双模型协同架构**：
- **外层主智能体模型**：负责处理用户高维复杂的业务推理与工具调用（如 GPT-4 等大参数模型）。
- **内层摘要辅助模型**：专门负责将溢出的历史消息压缩归纳为紧凑的综述文本。通常选用轻量模型（如 `gpt-4o-mini`），响应快且单价极低。

```python
from langgraph.prebuilt import create_react_agent
from langgraph.checkpoint.memory import InMemorySaver
from langchain_openai import ChatOpenAI

# 1. 声明主模型与轻量级内层摘要模型
main_model = ChatOpenAI(model="gpt-4o")
summary_model = ChatOpenAI(model="gpt-4o-mini")

# 2. 装配摘要中间件
# 当消息 Token 数超过 100 时触发归纳，并保留最近 2 条完整交互
agent = create_react_agent(
    model=main_model,
    tools=[],
    checkpointer=InMemorySaver(),
    # 注入模型内摘要机制，并设定阈值与保留窗口
    prompt="你是一个贴心的助理",
)
```

在第二轮交互执行完毕后，我们可以通过 `agent.get_state(config)` 获取当前的最终状态，并借助 Rich 库对其内部结构进行审查：

```python
from rich import print as rprint

final_state = agent.get_state(config)
rprint(final_state.values["messages"])
```

输出的结果清晰地展示出结构分化：原本长达数百 Token 的早期长篇输入被压缩替换为一条系统级摘要文本，紧随其后的则是保留下来的最近两轮完整对话。后续模型的所有决策，均基于“全量历史摘要 + 近期精确问答”的混合上下文展开。

在生产落地摘要策略时，需要关注以下四个关键设计问题：

1. **信息丢失边界**：摘要一定会带来一定程度的细节折损，但核心实体属性、用户身份与事实结论通常会被完整提炼；配合末尾的 `keep_messages` 保留区，能覆盖绝大多数连续交互诉求。
2. **Token 阈值设定依据**：`max_tokens` 不能拍脑袋决定。如果模型窗口是 4K，阈值应留出充足的缓冲余量，预留空间供系统提示词与复杂的工具调用定义（Tool Schemas）占用。
3. **经济成本控制**：使用内层轻量级模型进行后台异步摘要，其消耗的 API 资费往往仅为主模型的几十分之一，整体性价比极高。
4. **触发频率监控**：若监控指标显示系统频繁触发摘要，说明阈值定得过于激进，应适当上调 `max_tokens`；若长期从未触发，则说明上下文管理失效，需要下调阈值促成压缩。

> **提示**：智能体的 `AgentState` 字典不仅包含 `messages` 消息列表，在复杂工具编排或条件路由场景中，还包含诸如 `jump_to`（控制流跳转标签）和 `structured_response`（结构化输出结果）等系统保留字段，供高级中间件协同消费。

---

### 长期记忆体系与多维存储架构

短期记忆通过检查点器严格绑定在单条会话线程（Thread）上。但在真实的业务世界中，用户今天开辟新窗口向客服提问，明天再次发起全新的会话，智能体若表现得像完全不认识该用户，体验就会大打折扣。这就需要引入跨越所有线程的长期记忆（Long-term Memory）。

#### 长期记忆与短期记忆的维度差异

长期记忆与短期记忆并非互斥，而是在维度与生命周期上形成严密的互补体系：

```text
+-------------------------------------------------------------+
|                      长期记忆 (Store)                        |
|            用户画像、代码风格偏好、VIP 身份、业务规则       |
+-------------------------------------------------------------+
        ^                                            ^
        | 共享读取                                   | 共享读取
+-----------------------+                    +-----------------------+
|  短期记忆 (State)     |                    |  短期记忆 (State)     |
|  Thread A (会话 1)    |                    |  Thread B (会话 2)    |
|  [H1, A1, H2, A2...]  |                    |  [H1, A1...]          |
+-----------------------+                    +-----------------------+
```

读图说明：每一个独立的 `thread_id` 维护一份私有的 `state` 消息流水，而所有 `thread` 无论何时创建，均共同挂载在全局唯一的 `store` 实例之上，随时随地读取与更新沉淀在其中的全局信息。

- **短期记忆（Short-term Memory）**：以 `thread_id` 为边界的会话级隔离存储，管理单次任务连续对话的上下文。
- **长期记忆（Long-term Memory）**：以用户维度（User-level）或应用全局（App-level）为边界的共享存储。诸如“用户偏好简短回答”、“用户技术栈偏好 Python”、“该用户是付费 VIP 会员”等跨会话事实，均沉淀于此。

#### 认知架构的三类长期记忆划分

在智能体架构设计中，长期记忆通常对标认知科学的三大支柱分类：

1. **语义记忆（Semantic Memory）**：事实与偏好类记忆。例如记录用户的固有属性（“常用语言为中文”、“偏好极简代码风格”）、业务领域实体归属（“某客户属于金融科技行业”）。
2. **情景记忆（Episodic Memory）**：经验与历史事件类记忆。记录智能体过往解决某一类复杂问题的行动轨迹与成败经验。在工程上，情景记忆常被转化为动态的少样本案例库（Few-shot Examples），当智能体面临相似任务时，从记忆库中调出过往最成功的历史样本引导模型输出。
3. **程序性记忆（Procedural Memory）**：规则与做事章法。包括智能体系统提示词的动态优化版、标准化工作流的运转步骤，以及工具链调用的约束准则。

#### 长期记忆的四层核心概念拓扑

为了在系统层面实现高效存取，LangGraph 构建了一套对标短期记忆的长期记忆核心概念架构：

```text
+-------------------------------------------------------------+
| 长期记忆容器 (Store: InMemoryStore / PostgresStore)          |
|   +-------------------------------------------------------+ |
|   | 命名空间 (Namespace: tuple[str, ...])                 | |
|   | 类似于层级目录结构: ("users", "user_123", "profile")   | |
|   |   +-------------------------------------------------+ | |
|   |   | 唯一标识符 (Key: str)                           | | |
|   |   | 键: "preferences"                               | | |
|   |   |   +-------------------------------------------+ | | |
|   |   |   | 真实数据载荷 (Value: dict[str, Any])      | | | |
|   |   |   | 值: {"language": "zh", "theme": "dark"}   | | | |
|   |   |   +-------------------------------------------+ | | |
|   |   +-------------------------------------------------+ | |
|   +-------------------------------------------------------+ |
+-------------------------------------------------------------+
```

读图说明：长期记忆从外至内由 Store、Namespace、Key、Value 四个核心层级逐层嵌套。

- **Store（长期记忆容器）**：对标短期记忆的 Checkpointer。需要注意的是，**长期记忆不等于持久化存储**。它同样包含纯内存实现的 `InMemoryStore`（适用于开发测试与本地校验）和数据库持久化实现的 `PostgresStore`（适用于生产环境跨实例部署）。
- **Namespace（命名空间）**：层级检索路径，在 Python 中表现为字符串元组 `tuple[str, ...]`。它完美类比了操作系统的多级文件夹结构，或者现实世界中的户籍地址层级——如 `("河北省", "石家庄市", "裕华区")`。在跨应用或跨用户过滤时，命名空间提供了精准且结构化的范围界定能力。
- **Key（唯一键）**：特定命名空间下的唯一主键，必须为字符串类型（`str`），在同一目录下唯一标识一个实体记录。
- **Value（数据载荷）**：存储的具体内容，规范要求为字典类型（`dict[str, Any]`），能够灵活承载非结构化与半结构化的高维信息。

---

### 长期记忆基础操作与跨介质更新行为

在将长期记忆装配给智能体之前，我们需要从底层的原子 API 入手，探究数据在不同介质中的存取机理。长期记忆的核心 API 由 `put`、`get` 与 `search` 构成。

#### 基础操作接口定义与参数规范

数据读写的两项最基本操作是写入（`put`）与读取（`get`）。

- **`store.put(namespace, key, value, index=None, ttl=None)`**：
  - `namespace`: 必须传入字符串元组，指定存放的层级路径。
  - `key`: 字符串，指定当前命名空间下的唯一键。
  - `value`: 字典类型，实际存储的业务数据。
  - `index`: 语义检索索引配置项，设为 `None` 时继承 Store 的默认配置。
  - `ttl`: 存活时间（Time To Live），用于控制具备时效性记忆的过期回收机制。
- **`store.get(namespace, key, refresh_ttl=False)`**：
  - 基于命名空间与 Key 进行精确索引检索。
  - 核心返回值：并非直接返回原始的 `value` 字典，而是返回一个包装对象 `Item`。该对象完整包含了 `namespace`、`key`、`value`，以及系统自动维护的元数据时间戳 `created_at` 和 `updated_at`。

#### 内存存储模式下的全量覆盖行为

首先考察基于 `InMemoryStore` 的读写与覆盖逻辑：

```python
from langgraph.store.memory import InMemoryStore

store = InMemoryStore()

# 1. 首次写入数据
namespace = ("users",)
user_id = "user_1"
profile_data = {"user_name": "小明"}

store.put(namespace=namespace, key=user_id, value=profile_data)

# 2. 读取验证
item = store.get(namespace=namespace, key=user_id)
print(item)
# 输出包含: value={'user_name': '小明'}, created_at='2026-09-20T10:00:00.123', updated_at='2026-09-20T10:00:00.123'
```

此时 `created_at` 与 `updated_at` 的时间戳几乎完全一致。接下来我们在相同的 `namespace` 与 `key` 下执行更新写入，将姓名由“小明”变更为“小红”：

```python
# 3. 覆盖写入
updated_data = {"user_name": "小红"}
store.put(namespace=namespace, key=user_id, value=updated_data)

# 4. 再次读取
updated_item = store.get(namespace=namespace, key=user_id)
print(updated_item)
```

在 `InMemoryStore` 中，观察到的时间戳表现如下：
- `value` 成功变为小红；
- `created_at` 与 `updated_at` 均被刷新为当下的最新时间戳，两者仍然严格相等！

> **易错点**：在 `InMemoryStore` 的纯内存底层实现中，每次调用 `put` 都会在哈希表中生成一个全新的 `Item` 对象覆盖旧对象。因此它不会保留该条目历史上首次创建的时间戳，表现为 `created_at` 随同每次覆盖而一同被重置。

#### PostgreSQL持久化模式下的增量更新行为

切换为生产级的 `PostgresStore` 后，底层的时序表现呈现出显著差异。

```python
from langgraph.store.postgres import PostgresStore

DB_URI = "postgresql://postgres:password@192.168.1.100:5432/langgraph_db"

with PostgresStore.from_conn_string(DB_URI) as store:
    # 首次执行建表初始化
    store.setup()
    
    # 1. 首次写入
    namespace = ("users",)
    store.put(namespace, "user_1", {"user_name": "小明"})
    item1 = store.get(namespace, "user_1")
    # 首次写入后：created_at 等于 updated_at
    
    # 2. 执行更新写入
    store.put(namespace, "user_1", {"user_name": "小红"})
    item2 = store.get(namespace, "user_1")
```

在 PostgreSQL 持久化引擎中观察 `item2` 的输出：
- 数据载荷 `value` 正确更新为 `{"user_name": "小红"}`；
- **`created_at` 维持了首次写入小明时的历史时间戳不变**；
- **`updated_at` 被精确更新为当前写入小红的最新时刻时间戳**。

```text
[InMemoryStore 内存覆盖机制]
首次写入 -> 创建 Item(v1) [created_at = T1, updated_at = T1]
更新写入 -> 丢弃旧对象并新建 Item(v2) [created_at = T2, updated_at = T2]
结论: 内存模式无法保留记录的初创时序审计线索。

[PostgresStore 关系型数据库更新机制]
首次写入 -> INSERT 记录 [created_at = T1, updated_at = T1]
更新写入 -> UPDATE 记录 [created_at = T1 (保持原值), updated_at = T2 (更新为最新)]
结论: 数据库引擎具备完整的实体生命周期时序维护能力。
```

读图说明：PostgreSQL 底层依靠关系型数据表的行级更新能力，保证了数据记录首次进入系统的时间溯源性，这与基于 Python 字典简单赋值的内存实现形成了技术分水岭。

无论是在短期记忆还是长期记忆的设计中，架构选型的本质都是在“内存操作的高吞吐与轻量性”和“关系型数据库的持久一致性与时序严密性”之间寻求平衡。深刻理解检查点快照的生成机理、消息流的精细裁剪，以及多维命名空间在底层介质中的真实物理落盘行为，是构建工业级智能体记忆系统的坚实基础。

---

## 本册小结

本册主题为「中间件矩阵、Hook钩子拦截与持久化记忆架构」，整编了 6 个模块（P64-P66 ~ P92-P97，共 34 讲 / 315 分钟音频）。
建议配合 `notes/` 目录下的复习笔记复盘；模块长文原文保留在 `articles/`，需要查证细节时可直接回到原文。
