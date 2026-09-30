# LangChain 核心组件：模型调用、提示模板、部署与流式输出

## LangChain 的定位与价值

* **LangChain**

    * > **定义**：AI 领域里类似 JDBC 的框架，本质是连接大语言模型与 AI 产品、外部系统的桥梁和中介。

    * 类比理解：Java 程序员用 JDBC 连数据库，Python 程序员用 PyMySQL 连数据库，LangChain 在 AI 领域扮演同样的角色——连接和集成不同系统。

    * 直接好处：大语言模型不仅能够处理文本，还能在更广泛的环境中操作和响应，大大扩展了它的使用范围和有效性。

    * 国内背景：国内开发偏应用方向，底层算法优化受高端显卡制约（英伟达 H100/H200 一块卖几十万，40 万到 60 万美金甚至更高，且国内根本拿不到），走大模型落地开发更现实；模型再强最终也要落地——把大模型和人、和普通消费者的产品连起来就要开发功能性软件，开发软件就需要框架。

* **数据连接**

    * > **定义**：LangChain 允许把大语言模型链接到自己的数据源，并从中提取数据来回答问题。

    * 数据源可以是自有数据库、公司内部的 PDF 文件或 Word 文档；与大模型交互问答时，回答里的部分数据就来自私有数据库，更符合真实业务场景。

* **行动执行**

    * > **定义**：根据提取的信息帮助执行特定操作，比如发邮件——说一句话，LangChain 写好的程序就能帮我们做一些特定的事情。

    * 裸用普通的 OpenAI 做不到这件事，除非在它基础上再加一些代码，而且要加得比较多；用 LangChain 框架代码就写得少。

## 六个核心组件

* **模型封装（Models）**

    * > **定义**：大模型的包装器，提供统一的调用入口。

    * 0.2 版本之后支持的大模型至少有几十种：OpenAI 的 GPT-4、GPT-3，Hugging Face 的，GLM 提供的各式模型都支持，几乎涵盖市面上所有大公司比较著名的大模型。

    * 最关键的能力：写代码时想切换到另一个模型，代码一行不动，改配置即可——比如一开始用 GPT-4，后来想换更便宜的 GLM。

* **提示模板（Prompt Template）**

    * > **定义**：避免用硬编码文本进行输入、可动态把用户输入插入模板形成完整提示信息的组件。

    * 模板还可以自定义；提问时 prompt（提示）非常重要，LangChain 里专门有管理提示的组件。

* **链（Chains）**

    * > **定义**：把多个组件结合在一起、解决特定任务、构建完整语言模型应用程序的机制。

    * 到底联合哪些组件，由开发者自己按需决定。

* **代理（Agents）**

    * > **定义**：允许语言模型与外部 API 进行交互的组件。

* **嵌入与向量数据库（Embeddings / Vector Store）**

    * > **定义**：负责文本向量化与向量存储检索的组件。

    * 现成、稳定的向量数据库产品已经有很多；公司若有自己的向量数据库，开发时也可以直接连。

* **索引（Indexes）**

    * > **定义**：在搜索之前先对指定资料建立索引的组件。

    * 比如要把官网文档内容交给 LangChain 做搜索，必须先基于这些内容建立索引，然后才能在里面做搜索。

## 智能问答的底层链路

* **相似性搜索**

    * > **定义**：用户问题进入大模型之前，先在一个大型数据库或向量空间中找到与之相关信息的步骤。

    * 原理：向量之间的相似度，就是拿两个向量的夹角、用余弦距离来度量。

* **链路全景**

    * 步骤：用户问题 question 先不直接进大模型 → 相似性搜索在向量数据库中检索公司相关资料 → 检索结果与原始问题相结合、封装成一条非常复杂的 prompt → 交给大语言模型处理后得到 answer。

    * answer 两种去向：可能就是一段回答，直接传给用户；也可能需要执行某个动作（如模型判断该找专业技术服务人员来交互），由代理调用 AI 系统外部的 API，通过拨号、发邮件或消息转发完成任务。

    * 性质：数据驱动的决策过程，包含从信息检索、到处理、到最终行动的自动化步骤；到底是行动还是回答不确定，要根据用户问的是什么来决定。

    * 这类系统已经不是简单的人机交互对话：除对话外还包括 AI 自动执行的动作；用户问题背后的数据也可以来自公司内部的大型关系数据库或向量数据库。

    * 链路图示：

      ```text
      用户提问 Question
         |
         v
      相似性搜索（余弦距离度量向量相似度）
         |
         v
      在向量数据库 / 向量空间中检索公司相关资料
         |
         v
      检索结果 + 原始问题 --> 封装成一条完整的复杂 Prompt
         |
         v
      大语言模型（LLM）
         |
         v
      答案 Answer
         |
         +--> 只是一段回答 ----------> 直接交给用户
         |
         +--> 需要执行动作 --> Agent（代理）--> 调用外部 API
                                               （拨号 / 发邮件 / 转发信息）
      ```

## 应用场景

* **个人助手**：帮助预订航班、自动转账、缴税——AI 接收信息后做出决策，不是只回答一句，而是自动帮用户实现预订、转账、缴税；不单有人机对话、还能执行动作的 AI 软件才更加智能。

* **学习辅助**：参考提供的整个课程大纲帮助更快学习材料；例如让 AI 先去指定地方搜索一百篇论文，再根据论文信息回答提问，甚至摘取论文内容自动保存 Word 文档或自动生成 PDF。

* **数据分析和数据科学**：公司的数据、客户的数据、市场的数据都能极大地促进数据分析的进展。

## 安装与版本

* **安装方式**

    * 用 pip 直接安装 LangChain 本身；要用 OpenAI 的模型还需安装 `langchain-openai`，它包括 OpenAI 所有的模型。

    ```bash
    pip install langchain
    pip install langchain-openai
    ```

    * 不指定版本默认只安装最新版；安装会连带装上 SQLAlchemy（访问本地尤其是关系数据库要用）、BeautifulSoup4（做爬虫爬数据）、JSON 解析工具以及 langchain-core、langchain-openai 等核心包；langchain-community 当时没装也没关系，后面用到发现缺了再装。

* **0.2 版本**

    * 官方文档只支持 JS 和 Python 两种语言；0.2 于 5 月 20 号正式发布，三四月份 beta 测试版就已出来，文档里可以点版本切换看 0.1。

    * 相比 0.1 代码主体差不多，大概改了 10% 到 15%；升级不在功能上，主要往两个方向：一个是稳定性（也就是安全性，避免了一些容易被拿去做 DoS 攻击的东西），一个是兼容性（兼容更多的大模型）；企业里未来一定用的是 0.2。

## LangSmith 监控平台

* **LangSmith**

    * > **定义**：构建生产级大语言应用程序的平台，提供调试、测试、评估、监控的能力，覆盖基于任何大模型框架构建的链和智能代理；它是 LangChain 内部的子项目，与 LangChain 无缝集成。

    * 存在理由：大模型按 token 计费，今天用这个模型明天用那个，想统计总共用了多长时间、总共花了多少钱得跑两个平台，太麻烦；它直接跟踪调用大模型 API 的次数以及总 token 数量。

    * 记录内容：找到 project 点进去，能看到每次调用的完整输入与输出、时间、token 数量，连一共花了多少钱都算出来。

    * 费用：针对单个用户免费，每个月 5000 次 trace（追踪）也免费；真做成产品、用户量非常多超出后要收费，页面上有收费标准；从来不存在「不用 LangSmith 就不能学 LangChain」这回事。

    * 主要作用：调试与测试（记录中间过程，调整提示词等中间环节优化模型响应）、评估效果、监控性能、数据管理（自动存储每次调用的提示输入并做分析，帮助理解模型提示、优化输入）、团队协作、可扩展性与维护性——核心提供的就是调试、测试、评估、监控四件事。

* **API key 获取**

    * 登录方式三种：GitHub 账号、谷歌账号、邮箱地址注册，任选一种；国内不翻墙就能访问，只是网速稍微慢点。

    * 路径：界面左下角点 Settings → 第一项就是 API Keys → 点 Create API Key → 填一个描述信息（随便写，个人用即可）→ 点 Create 生成。

    * > **易错点**：key 生成后要立刻复制保存到自己电脑的记事本里——复制过一次之后，列表里就看不到了，要点进去才能再看到；不想要旁边也有删除。

    * > **易错点**：不配置它也完全能跑，代价是没有代码的跟踪、没有提示数据的存储、也没有监控和大模型的数据分析；反正是免费的，建议配上。

## 模型调用与响应解析

* **模型支持范围**

    * 官网上可以看到支持了大概二三十种模型，常用的就有十几种，这十几种的代码全都写好了，复制粘贴过来就能用。

    * 各家写法不同：用 OpenAI 的模型是一套写法；用 Anthropic 的模型是另一套，要先装 `langchain-anthropic` 包；还有谷歌的模型等等。

* **追踪配置**

    * 两行核心环境变量：一行指定代码用 LangSmith 做追踪，一行放 LangSmith 的 API key——注意这个 key 是 LangSmith 的，不是 LangChain 的。

    ```python
    import os

    os.environ["LANGCHAIN_TRACING_V2"] = "true"   # 必须写 true：使用 V2 追踪
    os.environ["LANGCHAIN_API_KEY"] = "你的 LangSmith API Key"
    ```

    * 追踪开关一定要写 `true`：不写它会用 V1 的版本，而当前安装的环境不可能是 V1。

* **创建模型（ChatOpenAI）**

    * 从 `langchain_openai` 导入 `ChatOpenAI`，构造参数 `model` 传模型的名字字符串（如 `gpt-4-turbo`、`gpt-3.5-turbo`），名字从 OpenAI 官网查；未来换模型只需要改这个字符串。

    * 调 OpenAI 的模型需要网络可达（翻墙），而且要把 OpenAI 的 API key 配置到运行环境里，否则第一次运行就报错提示没有 key。

* **消息与角色**

    * prompt 可以理解为就是传 message，是一个中括号列表——因为可以传不同的角色；角色分三种：系统、用户、AI。

    * `SystemMessage` 给出任务设定（如「请将以下内容翻译成意大利语」）；`HumanMessage` 是普通用户说的话，即要被处理的内容。

* **调用与响应**

    * `model.invoke(messages)` 里传 messages，返回一个**非常复杂的对象**：除 `content` 外还包含响应的元数据——返回的 token 数量、具体用的哪个模型（如 2024 年 4 月 9 号快照的 gpt-4-turbo）。

    ```python
    from langchain_openai import ChatOpenAI
    from langchain_core.messages import SystemMessage, HumanMessage

    model = ChatOpenAI(model="gpt-4-turbo")
    messages = [
        SystemMessage(content="请将以下内容翻译成意大利语"),
        HumanMessage(content="请问你要去哪里"),
    ]
    result = model.invoke(messages)
    ```

* **输出解析器（OutputParser）**

    * > **定义**：给大模型返回的复杂响应对象配的解析器，只抽出最重要的核心结果，input token、返回 token 数量这些都不要。

    * 可解析成字符串、JSON 对象、自定义的 Python 类型（类似 Java 里 POJO 那种对象类型）；解析器有很多种，`StrOutputParser` 只是最简单的其中一种。

    * `StrOutputParser` 来自 `langchain_core` 的 `output_parsers` 模块，空构造函数直接创建；同样用 `invoke` 调用，里面传的是响应对象，返回解析之后的字符串。

* **链式写法：未来代码的标准形态**

    * 调大模型一共五步：创建模型 → 准备提示 → 创建返回数据的解析器 → 把模型和解析器用竖线连起来得到链 → 直接使用链调用。

    * 竖线 `|` 就是把各个组件连起来：这是解析器的组件、这是模型的组件，连起来得到一个 chain，链自己去 `invoke`；未来代码都是这样写的，裸调的写法很少采用了。

    ```python
    from langchain_core.output_parsers import StrOutputParser

    parser = StrOutputParser()          # 空构造函数直接创建
    chain = model | parser
    print(chain.invoke(messages))
    ```

## 提示模板

* **提示模板**

    * > **定义**：输入给大语言模型的模板，把写死的提示词变成活的。

    * 没有模板的痛点：提示词写死、一点也不灵活——今天翻意大利语、明天想翻英语就得改代码。

* **构建模板**

    * 用 `langchain_core` 里的 `ChatPromptTemplate` 类，通过它的 `from_messages` 函数从一个消息列表里构建模板。

    * `from_messages` 接收一个消息数组，数组里一个一个元组括起来，传的是不同角色说的话：system 元组放固定的话加变量，user 元组把整个用户输入当成一个变量。

    * 变量在模板里的写法：大括号里写名字（如 `{language}`、`{text}`），名字随你取，未来就根据这个参数名传参——`language` 传什么都行，意大利语、英语、阿拉伯语都 OK。

    ```python
    from langchain_core.prompts import ChatPromptTemplate

    prompt_template = ChatPromptTemplate.from_messages([
        ("system", "请将下面的内容翻译成{language}"),
        ("user", "{text}"),
    ])
    ```

* **组装与调用**

    * 链的顺序：模板一般放在前面——`prompt_template | model | output_parser`，`model` 和 `output_parser` 是之前定义好的对象，不用动。

    * `chain.invoke` 传参不能像以前那么干：要用大括号括起来，实际上是用字典的方式传参——参数名就是模板大括号里的名字；要翻别的语言，只要把 `language` 的值换一下就可以。

    ```python
    chain = prompt_template | model | output_parser

    response = chain.invoke({
        "language": "italian",
        "text": "下午还有一节课，不能去打球了",
    })
    ```

## LangServe 部署服务

* **LangServe**

    * > **定义**：把写好的 LangChain 应用程序部署成服务器，让公司里其他人、或者别的程序（其他 API、安卓程序、iOS 程序）通过发 HTTP 请求来调用的工具。

    * 安装要装它的所有东西，后面加 `[all]`；踩坑提醒：第一次装死活装不上，其实是翻墙的锅——把代理去掉马上就装好，而真正调模型跑程序时代理又得开回来，两码事别搞混。

    ```bash
    pip install "langserve[all]"
    ```

    * 底层用到 WebSocket，还用到 FastAPI——FastAPI 是目前为止 Python 里性能最好的 Web 框架，可以把任何服务部署成一个个的接口。

* **创建 FastAPI 应用**

    * `title` 是打开接口文档时能看到的应用程序名称；`version` 表示部署的这个服务的版本；`description` 是描述——值都由你决定。

    ```python
    from fastapi import FastAPI

    app = FastAPI(
        title="我的LangChain服务",
        version="v1.0",
        description="使用LangChain翻译任何语句的服务器",
    )
    ```

* **添加路由与启动**

    * `add_routes(app, chain, path="/chain_demo")`：`path` 表示请求路径，可以随便改（默认叫 chain）；未来别人请求接口就按照这个 path 来。

    * 直接右键运行，启动后监听在 localhost 上——写 localhost 就表示只监听 `127.0.0.1`，端口是 8000，日志里写得清清楚楚。

    * > **提示**：真要给别人调，监听地址写具体的服务器 IP 最合适；只写 localhost 的话就只监听在 127.0.0.1 上，外面访问不到。

    ```python
    from langserve import add_routes

    add_routes(app, chain, path="/chain_demo")
    ```

### HTTP 客户端调用

* **请求规则**

    * 一定是 POST 请求；传参类型首先要设置成 JSON 格式（`Content-Type` 用 `application/json`），不能是表单提交的方式。

    * 地址拼法：服务器地址 `127.0.0.1` + 端口 `8000` + 路由路径 `/chain_demo` + 固定后缀 `/invoke`——这个后缀是固定死的，表示「调用」。

    * 参数结构：`input` 是固定死的整个参数的名字；`input` 里面传什么跟提示模板有关——模板里定义了哪些变量就传哪两个。

    * 返回结果里有个 `output` 字段，装的就是翻译之后的结果；客户端工具用 ApiPost、Postman 都行，哪个都可以。

    ```json
    {
      "input": {
        "language": "english",
        "text": "我要去上课了，不能和你聊天了"
      }
    }
    ```

### Python 客户端调用

* **RemoteRunnable**

    * > **定义**：langserve 提供的远程调用客户端类，构造时接一个服务路由地址。

    * 地址要把 `/invoke` 去掉——接下来真正调用的是它的 `invoke` 函数，调 invoke 时它会自动在后面加一个 `/invoke`。

    * 传参用字典：HTTP 客户端里叫 JSON，Python 里就不叫 JSON 了，叫字典，一样的。

    * 服务端必须先跑起来而且不要关，关了之后别人肯定访问不了；客户端代码自己并没有实现 AI 功能——服务器实现了，调服务器就行。

    ```python
    from langserve import RemoteRunnable

    client = RemoteRunnable("http://localhost:8000/chain_demo")
    response = client.invoke({
        "language": "italian",
        "text": "你好",
    })
    print(response)
    ```

## 聊天机器人与会话记忆

* **为什么要存聊天记录**

    * 用大语言模型做人和机器的聊天，最重要的一个环节就是把聊天记录全部存起来；要不然 AI 没办法把整个聊天历史作为上下文，有些跟上下文有联系的问题就回答不上。

* **ChatMessageHistory**

    * > **定义**：LangChain 子项目 `langchain-community` 里保存所有人和机器聊天历史记录的类。

    * 需要先安装 `pip install langchain-community`；这个历史会话记录不但能存进去，还可以从它里面拿出来，都存在 store 里。

* **store 字典**

    * 结构：以每个用户的 session_id 作为 key、value 就是历史聊天记录对象；这个程序针对很多用户，每一个用户每一次会话都会有一个新的 session_id，session id 可以自己去定义。

* **get_session_history 回调函数**

    * 参数是字符串 session_id；里面只做一个判断：看这个 session_id 是不是不在 store 里，不在就往里放一个新的 `ChatMessageHistory()`；最后一句 `return store[session_id]`——其他情况都是直接从 store 里返回，不用写 else。

* **RunnableWithMessageHistory**

    * > **定义**：负责每次运行聊天时携带聊天历史记录的包装器——名字里的 with message history 就是「携带消息历史」的意思。

    * 三个参数：第一个传 chain（没有 chain 的话传模型也行）；第二个传刚写的回调函数；第三个 `input_messages_key` 定义每次聊天输入消息的键，名字随你定（如 `my_msg`），这个参数必须在这里定义好。

    * > **易错点** ⚠️：回调函数千万不要加括号——写 `get_session_history`，不要写 `get_session_history()`，因为这个函数是由 RunnableWithMessageHistory 这个类负责去调用的，不是你在这里手动调的。

    ```python
    from langchain_community.chat_message_histories import ChatMessageHistory
    from langchain_core.runnables.history import RunnableWithMessageHistory

    store = {}

    def get_session_history(session_id: str) -> ChatMessageHistory:
        if session_id not in store:
            store[session_id] = ChatMessageHistory()   # 第一次聊：新建一份记录
        return store[session_id]

    with_message_history = RunnableWithMessageHistory(
        chain,
        get_session_history,      # 回调函数：千万不要加括号
        input_messages_key="my_msg",
    )
    ```

* **会话调用与验证**

    * 模板的 system 给当前 AI 机器人一个定位（如「你是一个非常乐于助人的助手，用{language}尽你所能地回答所有的问题」），用什么语言回答由 `language` 参数传进来。

    * `invoke` 要传两个参数：第一个字典里以 `my_msg` 为键放 `HumanMessage` 消息、同时把模板变量 `language` 传进去；第二个参数 `config` 传会话配置。

    * config 就是一个字典，前面的键固定叫 `configurable`，值里放配置项 session_id，值随便写（如「张三123」）；响应内容通过 `.content` 拿到——这个案例里输出解析器暂时不需要。

    * 验证记忆：第一轮聊天一般没有所谓的历史记录；从第二轮开始，每次都会把历史记录发给大语言模型——第一轮自我介绍「我是老肖」，第二轮问「请问我的名字是什么」，单独问 AI 肯定不知道，结合了聊天记录它就能答出来。

    ```python
    config = {"configurable": {"session_id": "张三123"}}

    response = with_message_history.invoke(
        {
            "my_msg": HumanMessage(content="你好！我是老肖"),
            "language": "中文",
        },
        config=config,
    )
    print(response.content)
    ```

* **历史消息占位符（MessagesPlaceholder）**

    * > **易错点** ⚠️：提示模板中若根本没有「要发送的历史记录」这一项，历史记录就发不出去，每次对话都是单独的，AI 完全不记得你前面说过什么。

    * 特殊写法：`MessagesPlaceholder(variable_name="messages")`——声明真正要发送的所有历史消息的键是什么；`variable_name` 必须和前面定义历史消息地方的键保持一致，两边名字对不上，历史记录就接不进来。

    * 补好之后，模板里除了 system，还有你的所有历史聊天记录。

    * 图示如下：

      ```text
      用户调用 with_message_history.invoke(输入, config)
              |
              v
      RunnableWithMessageHistory（负责每次携带历史记录）
              |
              |-- 从 config 里取出 session_id（如 "张三123"）
              |        |
              |        v
              |   get_session_history("张三123")   <-- 回调，由它自动调用
              |        |
              |        v
              |   store 字典：{"张三123": ChatMessageHistory}
              |   （没有这份记录就新建，有就直接取）
              |
              v
      chain（模板 + 模型）拿着 [历史记录 + 本轮消息] 生成回答
      ```

## 流式输出

* **流式输出**

    * > **定义**：让 AI 返回的数据一个 token 一个 token 地往外蹦的输出方式；现在基于 OpenAI 的大部分中转网站都采用它。

    * 关键改动就一处：调用的函数不再是 `invoke`，而是 `stream`——在 LangChain 里想要流式返回，就必须用 `stream`。

    * `stream` 返回来的东西是一个循环（可循环对象），代码不能再像 `invoke` 那样一次接住，要换成 `for` 循环一次拿一个结果，循环里的每一次响应都是一个 token。

    * 传参跟之前的 `invoke` 完全一样，只是还要传一个 `config`；`print` 要写在 `for` 循环里面——每来一个 token 就打印一个，可以在每个 token 后面加一个 `-` 分隔符，清楚看到 token 的边界。

    | 对比项 | `invoke` | `stream` |
    | :--- | :--- | :--- |
    | 调用方式 | 一次调用，等完整结果 | 调用后拿到一个可循环的对象 |
    | 每次得到什么 | 一个完整的响应 | 循环里的每一次响应都是一个 token |

    ```python
    resp3 = chain.stream(
        {"question": "请给我讲一个笑话，请使用英文回答"},
        config=config3,
    )
    for resp in resp3:
        print(resp.content + "-")   # 循环里的每一次响应 resp 都是一个 token
    ```

* **会话与语言的表现**

    * 沿用旧会话要求「用英文讲笑话」：流式机制正常、一个 token 一个 token 带分隔符返回了，但回答却是中文——因为这个会话里前面聊的都是中文，它不可能突然改用英文。

    * 解决：把之前的 config 复制一份，给这一轮一个全新的会话、定义一个新的 session id（如「李四」），其他地方都不变，再运行返回的就是英文了。

    * > **提示**：历史记录跟着 session id 走；换了会话就是一张白纸，之前说过的名字就找不到了——想让一段对话「重新开始」，给它换一个新的 session id 就行。

## 向量数据库与相似度搜索

* **案例目标与四步**

    * 用 LangChain 构建向量数据库和检索器：检索增强（RAG）里面肯定会用到，做智能化的搜索、智能化的检索也需要；它支持从向量数据库和其他来源检索数据，与大语言模型的工作流集成——应用程序需要获取数据，作为模型推理的一部分。

    * 数据源：本案例用的是自己设置的一组定死的数据（狗、猫、金鱼、鹦鹉、兔子五篇文档）；数据从数据库里来、或从网络网页爬下来也完全可以。

    * 图示如下：

      ```text
      准备数据（一个个 Document 文档）
              │
              ▼
      向量化，存进向量数据库（得到向量空间）
              │
              ▼
      根据向量空间得到检索器（一个 Runnable 对象）
              │
              ▼
      检索器与大语言模型结合，用 chain 串起来
      ```

* **Document**

    * > **定义**：LangChain 核心库（core）里的文档数据单元——大语言处理的就是文本，不是关系型数据库那种结构化数据（关系型数据库某个字段里存着大段文本，也一样可以拿来处理）。

    * 属性 `page_content`：文本内容——比如要针对某一篇论文做向量数据库检索，论文里所有文字（假设两千多字）全都放进这个属性。

    * 属性 `metadata`：元数据，是一个字典——文档的摘要、文章的作者、文档的来源，键随你设置，爱填多少填多少（如 `author` 表示作者，甚至设个日期表示发布时间）。

    * > **定义**：元数据——除了这篇文章核心内容之外的数据，都叫元数据。

    ```python
    from langchain_core.documents import Document

    documents = [
        Document(
            page_content="猫是独立的宠物，通常喜欢自己的空间。",
            metadata={"author": "老肖"},    # 元数据：字典，键随意设置
        ),
        # 另外四篇：狗、金鱼、鹦鹉、兔子，写法完全相同
    ]
    ```

* **Chroma 向量数据库**

    * > **定义**：LangChain 里专门帮我们存储向量数据库的东西，可以理解为 LangChain 内置的一个向量数据库；外面公司提供的向量数据库服务器同样可以用。

    * 需要先安装 `pip install langchain-chroma`；`from_documents` 基于文档创建向量数据库：第一个参数是文档列表，所有文档都放这里；第二个参数是 embedding——采用哪种技术帮我们向量化。

    * `OpenAIEmbeddings` 是一个类，要把实例创建出来再传进去；这句话的意思就是：使用 OpenAI 的向量化工具，把所有文档变成一个向量空间。

    * 有了向量空间的好处：可以计算每一个向量之间夹角的余弦值，根据余弦值知道每个向量的相似度，接下来就可以做相似度搜索了。

    * embedding 是要调 OpenAI 接口的，运行前一样要在运行配置里把 OpenAI 的 key 配好。

    ```python
    from langchain_chroma import Chroma
    from langchain_openai import OpenAIEmbeddings

    vector_store = Chroma.from_documents(
        documents,              # 第一个参数：文档列表
        OpenAIEmbeddings(),     # 第二个参数：向量化技术
    )
    ```

* **相似度搜索与分值**

    * `similarity_search`：参数是 query——你需要查什么东西；去向量空间里查询哪一个向量和查询向量相似度最高，不带分数。

    * `similarity_search_with_score`：不但查出结果，还要把相似度的分值一起返回；不想看分数就直接调 `similarity_search`。

    * 你看到的 0.几其实就是所谓的距离：距离越大越不相似，距离越小相似度越高；文档排得靠不靠前，看的就是这个分值——Chroma 向量空间里分数越低、相似度越高。

    * 实测搜「咖啡猫」：排第一的是猫那篇「猫是独立的宠物，通常喜欢自己的空间」，分值 0.27；后面依次兔子 0.41、狗 0.413、金鱼 0.43——第一名与后续分值明显拉开（猫也是动物，和兔子、狗都有一定相似度）。

## 检索器与检索链

* **Runnable 前提**

    * 之所以能用竖线 `|` 把这些对象隔开、链接起来，是因为这些对象必须都是 Runnable 对象——可以随便追踪，比如 `ChatOpenAI` 一层层往上追父类，最终实现的就是一个 Runnable。

    * 刚创建的 `vector_store` 不是一个 Runnable 对象，所以未来不方便用 chain 把它链接起来。

* **检索器**

    * > **定义**：根据向量空间得到的一个 Runnable 对象，方便后面和 LangChain 里的聊天模型链接起来。

    * 写法：`RunnableLambda(vector_store.similarity_search).bind(k=1)`——lambda 可以理解为匿名函数，把 `similarity_search` 函数传进去；不需要看分值，所以只调不带 score 的版本。

    * > **易错点** ⚠️：函数后面千万千万记住不要加括号——你不是在这里直接调用它，而是把它封装到一个 Runnable 对象中，后面运行的时候它会自动帮我们调用。

    * `bind(k=1)`：搜索时它会把所有有一定相似度的结果全部输出，其实只要相似度最高的那个——k=1 就是选取相似度最高的第一个返回。

    * 单独调用：一看到 `invoke` 其实你就知道了——它也是一个 Runnable 对象；`batch` 可理解为匹配或批处理，一次性检索多个——`batch(["咖啡猫", "鲨鱼"])` 返回一个数组：「咖啡猫」最相似的是猫那篇，「鲨鱼」最相似的是金鱼那篇，符合人类语义。

    * > **提示**：检索器做的不是像百度那样的全文检索，本质上是一个相似度搜索。

    ```python
    from langchain_core.runnables import RunnableLambda

    retriever = RunnableLambda(vector_store.similarity_search).bind(k=1)
    results = retriever.batch(["咖啡猫", "鲨鱼"])
    ```

* **检索器与模型结合**

    * 为什么要结合：接下来的应用程序要把给定的问题与检索到的上下文结合起来，未来大语言模型给你的结果才相对准确、符合特殊需求。

    * 模板最重要的两个变量：`{question}` 是要回答的问题；`{context}` 是上下文——其实就是我们的向量空间，可以理解为就是检索器；消息比较复杂时用三引号多行字符串单独定义。

    * `from_messages` 的传参是一个 sequence（序列）：Python 中元组、列表、字符串这三个是序列，LangChain 里面 BaseMessage、提示模板这些也算；因为要设置角色，所以用二元组 `("human", message)`。

    * 链的拼装：`{"question": RunnablePassthrough(), "context": retriever} | prompt_template | model`——模板不能直接放在最前面，一开始应该先把模板里面的参数准备好。

    * `RunnablePassthrough` 是占位：允许将用户的提问之后再传递给提示模板和模型；构建 chain 的时候并不知道要提的问题是啥，真正把问题传进去是在调用 `invoke` 的时候——所以 `invoke` 只传问题，`context` 不用传（组链时就已经把检索器传过去了）。

    * 效果：问「请介绍一下猫」，直接问 OpenAI 得到的介绍一定非常笼统、非常一般；传了上下文之后回答是「猫是独立的宠物，通常喜欢自己的空间」——很明显来自我们提供的数据；如果提的问题跟上下文一点关系都没有，那它可能就直接通过 OpenAI 回答了。

    * 这个案例最重要的两件事：学会怎么构建一个向量空间；构建好的向量空间怎么通过 chain 和其他组件组合起来。

    ```python
    from langchain_core.prompts import ChatPromptTemplate
    from langchain_core.runnables import RunnablePassthrough

    message = """
    使用提供的上下文仅回答这个问题：

    {question}

    上下文：{context}
    """
    prompt_template = ChatPromptTemplate.from_messages([("human", message)])

    chain = (
        {"question": RunnablePassthrough(), "context": retriever}
        | prompt_template
        | model
    )
    resp = chain.invoke("请介绍一下猫")
    print(resp.content)
    ```

## 代理与搜索工具

* **代理（Agent）**

    * > **定义**：用大语言模型来作为推理引擎，确定要执行的操作、以及这些操作的输入应该是什么。

    * 为什么需要：大语言模型本身是无法执行动作的，它只能输出文本；训练用的数据可能是几个月前甚至几年前的，实时数据类问题（如「北京最近的天气怎么样」）它回答不了——要给出正确答案就必须执行相关动作，比如实时搜索天气网站的数据。

    * 执行顺序：用户问一个问题 Q，问给大语言模型 → 能直接回答就直接返回 → 答不了时得到一个推理结果（让你去执行一些动作）→ 自动去调用代理，代理执行相关动作、得到一些信息 → 信息再交回大语言模型，最终返回给用户。

    * 在 LangChain 里创建代理还需要一个新的包：`pip install langgraph`。

    * 无代理对照：`model.invoke` 的输入可以是一个字符串、也可以是一个序列，传 `HumanMessage` 列表问「北京最近天气怎么样」，模型答「很抱歉，我无法提供实时数据，包括当前的天气。我建议查看可靠的天气预报网站」——它没办法针对这个问题给出答案。

    * 图示如下：

      ```text
      用户提问 Q
         │
         ▼
      大语言模型 LLM（推理引擎）
         │
         ├─ 能直接回答 ──► 直接把答案返回给用户
         │
         └─ 答不了（需要实时信息 / 要执行动作）
                │
                ▼
           得到推理结果：去执行相关动作
                │
                ▼
           自动调用代理，代理执行动作拿到信息
                │
                ▼
           信息交回大语言模型，整合后返回最终答案
      ```

* **Tavily 搜索工具**

    * > **定义**：LangChain 里面内置的工具，可以轻松使用 Tavily API 帮我们做搜索引擎、搜索一些数据。

    * 类 `TavilySearchResults` 来自 `langchain_community` 里专门的 `tools` 库——这个库里有很多工具。

    * > **易错点** ⚠️：类名结尾有个 s——`TavilySearchResults`，少写了这个 s，自动导入都出不来。

    * 初始化要传参数 `max_results=2`：它会搜很多很多跟查询有关的网站，每个网站都搜回一些相关数据，但最终只返回两个值（写三个也行）。

    * 工具本身也是 Runnable 对象——一看到 `invoke` 你就知道了；它自动做文本匹配、向量匹配，匹配之后知道你要查的是天气、而且是北京的天气。

    * 它不是说去百度里搜：里面存了一些元数据，常用的公共网站它都有；会优先去国外的网站上搜索，所以返回内容是英文的（国外网站也有北京的天气预报）；搜国内网站返回的肯定就是中文。

    * 第一次运行会报缺 `TAVILY_API_KEY`：到它的官网点获取 API key 的按钮、用谷歌或其他网站的用户登录后就能看到；复制下来放到运行配置的环境变量里，名字是大写的 `TAVILY_API_KEY`；这个工具可以免费用，一个月用量到了之后才会收费。

    * 返回的是一个数组（因为最多返回两个结果），每一个结果都有一个 `url`（去哪个网址上搜到的）、一个 `content`（搜到的内容是什么），另外还有 `name`。

    ```python
    from langchain_community.tools import TavilySearchResults

    search = TavilySearchResults(max_results=2)
    result = search.invoke("北京的天气")
    print(result)
    ```

* **绑定工具**

    * 做法：调用 model 的 `bind_tools` 把工具绑定到模型上——`llm_with_tools = model.bind_tools([search])`。

    * `bind_tools` 接的参数是一个 sequence 序列——要么是列表、要么是元组、要么是字符串；要直接用中括号把 `search` 括起来再传，直接放 `search` 不是序列。

    * 绑定之后，大语言模型就会自动根据模型里的工具来做一些选择；到这里还没用到代理——现在只是在模型里面绑定工具，它有很多种灵活的用法，怎么在模型里面使用代理由后续内容详细展开。
