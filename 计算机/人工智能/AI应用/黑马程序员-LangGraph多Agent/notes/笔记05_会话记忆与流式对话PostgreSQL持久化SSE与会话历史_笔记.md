# 会话记忆与流式对话：PostgreSQL 持久化、SSE 与会话历史

## 会话记忆的持久化替换

   * **检查点（Checkpointer）**

       * > **定义**：LangGraph 在每步状态之后落盘快照的组件，按 `thread_id` 读写会话记忆，是 agent 记忆的底座。

       * 内存实现 `InMemorySaver` 只活在进程内存里，进程重启即全部清零，在真实项目里等同没有记忆。

       * 换成 PostgreSQL 实现后，会话可跨进程、跨重启存活。

       * 替换实现只动两处：导入的包、checkpointer 的构造。`创建 agent → 绑定 checkpointer → 调用时传 config` 这套流程一行都不用改。

       * 依赖包为 `langgraph-checkpoint-postgres`，同时带入 `psycopg[binary]`。

   * **两套驱动与两条 URL**

       * 业务库走 SQLAlchemy，底层驱动 `asyncpg`，URL 的 scheme 写成「协议 + 驱动」。

       * checkpointer 走 `psycopg` 的异步版本，URL 只写协议、不允许写驱动名，驱动藏在库内部。

       * 两条 URL 的形状对照如下，注释按「汉字 2 列、ASCII 1 列」对齐：

         ```text
         postgresql+asyncpg://user:pass@host:5432/db   业务库：协议 + 驱动
         postgresql://user:pass@host:5432/db           checkpointer：只有协议
         ```

       * 同一个数据库、两个客户端：底层驱动不同、连接方式不同、URL 写法不同，因此 SQLAlchemy 那套连接池在 checkpointer 这里用不了，必须单独再配一套。

   * **scheme 与 schema**

       * > **定义**：`://` 前面那一段叫 scheme（方案、协议）；schema 是库内的命名空间，默认叫 `public`。

       * > **易错点**：两个词只差一个字母，含义毫不相干，配置里写错就连不上库。

   * **一套配置产出两条 URL**

       * `DatabaseSettings` 保留 `host`、`port`、`user`、`password`、`database`，把拼 URL 的动作抽成 `build_url(scheme)`，scheme 作参数传入。

       * `url` 属性返回 `build_url("postgresql+asyncpg")` 给 SQLAlchemy；`checkpointer_url` 属性返回 `build_url("postgresql")` 给 checkpointer。

       * 两条路径前半段不同、后半段完全一致，不必维护两份密码与端口。

       * > **结论**：同一数据库两套客户端，先比较 URL 差在哪，通常只差 scheme，抽一个参数化函数即可，别复制两份配置。

## 连接池与自动建表

   * **异步连接池的构造参数**

       * `conninfo` 读 `checkpointer_url`，是专供 checkpointer 的那条 URL，不是业务库那条。

       * `min_size=1`、`max_size=5`：池子创建时先放 1 个连接，用完允许再建，上限 5 个；这里给的是下限与上限，不同于 SQLAlchemy 的「池大小 + 池满后额外连接数」。

       * 池子只服务 checkpointer 一个消费者、写记忆频率不高，1 与 5 足够；不放心可调成 3 与 10。连接建立速度很快，池子不必开大常年占着。

       * `kwargs` 三项是 LangChain 要求的固定写法，要用连接池就必须这么写：`autocommit=True` 自动提交事务、`prepare_threshold=0` 不缓存预处理语句、`row_factory=dict_row` 查询结果以字典返回。

       * `open=False`：创建池对象的瞬间不建连接，池子是空的。

   * **open 与 wait 配套**

       * 真正的连接在 `await pool.open()` 时建立，随后必须 `await pool.wait()` 等池内连接就绪。

       * > **易错点**：`open=False` 与这两步是一整套，只 `open()` 不 `wait()`，第一次查询很可能撞上还没准备好的空池，报错位置离真正的原因很远。

   * **异步 Saver 与建表**

       * `AsyncPostgresSaver` 从 `langgraph.checkpoint.postgres.aio` 导入；名字里的 `aio` 即异步，只接受异步调用，同步接口全部不可用。

       * 构造只做一件事：把连接池塞进去。与内存版相比，多出来的唯一动作是连接池初始化。

       * `await checkpointer.setup()` 自动建表，生成 `checkpoints`、`checkpoint_blobs`、`checkpoint_writes`、`checkpoint_migrations` 四张表，建表语句一句都不用写。

       * > **定义**：`setup()` 是 checkpointer 的建表入口，负责补齐 LangGraph 需要的表结构；它只补缺、不覆盖已有数据，放在初始化流程里重复执行是安全的。

   * **数据落在哪**

       * `checkpoints` 存状态快照，每步状态一条；四轮对话留下的是四个状态快照，不是四条消息，消息挂在对应快照下面。

       * `checkpoint_blobs` 存真正的消息正文，`message` 字段是加密内容，看到乱码属设计如此。

   * **同步改异步的三处改动**

       * `stream` 换成 `astream_events`、`for` 换成 `async for`、该等的地方补 `await`。

       * > **提示**：这不是 PostgreSQL 版特有的坑，凡是 `aio` 包都这样；漏一个 `await` 拿到的是协程对象而不是结果。

## 生命周期与 Agent 装配

   * **基础设施层的新模块**

       * 模块导入时就创建池对象与 saver，因 `open=False` 此时不连库，import 不会打数据库。

       * `init_checkpointer()` 依次执行 `open()`、`wait()`、`setup()`，打日志并返回 saver。

       * `close_checkpointer()` 只做一件事：关闭连接池。

       * 与业务库的 database 模块并列：写法套路相同，服务对象不同——一份给业务 ORM，一份给 agent。

   * **lifespan 的启动与关闭顺序**

       * 启动时 `yield` 之前：`init_db()` → `init_checkpointer()` → `init_insurance_agent(checkpointer)`。

       * 关闭时 `yield` 之后：`close_db()` → `close_checkpointer()`。

       * 顺序由依赖决定：agent 必须拿到 checkpointer 才能创建，所以 checkpointer 先就绪；关闭时反着走一遍。

         ```text
         启动
           ├─ init_db()                 业务库连接池（SQLAlchemy / asyncpg）
           ├─ init_checkpointer()      checkpointer 连接池（psycopg）
           │    ├─ open()   建连接
           │    ├─ wait()   等就绪
           │    └─ setup()  补表
           └─ init_insurance_agent()   绑定上述 checkpointer 的 agent

         关闭
           ├─ close_db()
           └─ close_checkpointer()
         ```

   * **Windows 事件循环报错**

       * 现象：启动到 checkpointer 初始化时报 `RuntimeError`，提示 `Please use a compatible event loop`。

       * 原因：psycopg 的异步连接要求事件循环支持 IO 事件的注册与监听，Windows 默认的循环类型不提供这套接口，连接一建就报错。代码没错，是循环选错了。

       * 解法：`loop = "asyncio" if sys.platform == "win32" else "auto"`，把该值塞进 uvicorn 启动参数的 `loop`，其余参数不动。

       * > **提示**：三元表达式拆成多行时要用一对圆括号把整个表达式包住，否则会被当成三条独立语句、直接语法报错。

   * **agents 包与三样绑定**

       * agent 不塞进业务包，单开 `agents` 包；用复数是故意的，将来不会只有一个智能体。

       * `create_agent` 要绑定三样：模型、工具、checkpointer（以前只有模型和工具两样）。

       * 工具先留空：`tools=[]`，等确定保险顾问要挂哪些工具再加。

       * `system_prompt` 写在 `create_agent` 参数里，角色设定跟着 agent 走，多少轮对话都不必每次请求重复传。

       * checkpointer 以参数传入，不在函数内 new；它已在 lifespan 初始化过，那是全项目唯一的一份，再 new 就会出现两个 checkpointer。

       * > **结论**：同类型 agent 全项目只初始化一次，所有请求共用同一个；只有确实需要不同类型的 agent 时才建多个实例。

   * **模型初始化时的 key 来源**

       * `init_chat_model` 的默认初始化靠模型名推断厂商，再自动去进程环境变量找 API key 与 base url。

       * `.env` 只被 Config 读进 `settings`，并没有写进进程环境变量，默认初始化什么也找不到。

       * 改为自定义初始化：显式传 `model`、`model_provider`、`api_key`，取值都从 `settings.LLM`；`model_provider` 可省，写上更明确。

       * 另需带上 `extra_body` 关闭思考：思考浪费时间，本场景不需要。

## 基础对话接口

   * **对话模块的构成**

       * 对话属于业务，不能放进专门放智能体的 `agents` 包，另建 `chat` 模块。

       * 常规业务模块是五件套：schemas、models、repository、service、router；对话不直接操作数据库，`models` 与 `repository` 去掉，只留 `schemas`、`service`、`router` 三件。

       * 会话记忆确实是数据库操作，但由 LangChain 内置完成——消息存储、消息拼接全自动，一行 SQL 都不用写。

       * 落地顺序仍是先 schema、再 service、最后 router。

   * **请求 Schema**

       * `ChatRequest` 继承 `BaseSchema`，只有两个字段。

       * `thread_id: UUID`：对应会话表的 `id`，本质是 UUID，发到前端变字符串、提交回来再转回 UUID；前端叫会话 id，LangChain 里叫 `thread_id`，一个东西两个名字。

       * `message: str = Field(..., min_length=1)`：用户这一轮说的话，不能为空。

       * 响应就是一个字符串，不写响应 schema。

   * **ChatService 的构造函数**

       * 需要两样：`AsyncSession`（创建 `ChatThreadRepository` 做归属校验）与 agent（做对话）。

       * agent 的类型就是 `create_agent` 的返回值类型，点进源码看返回值抄注解即可。

       * 自己没有 session，就由调用方传入——谁用它谁给，这就是依赖。

   * **ainvoke 调用**

       * FastAPI 是异步环境，用 `ainvoke` 不用同步阻塞的 `invoke`；用了就必须 `await`，函数必须是 `async def`。

       * `input = {"messages": [HumanMessage(content=request.message)]}`，`config = {"configurable": {"thread_id": request.thread_id}}`。

       * `input` 与 `config` 都是位置参数，按位置传即可；有了 config 才有会话记忆。

       * 返回仍是字典，取 `response["messages"][-1].content` 就是 AI 这段回复。

   * **会话归属校验**

       * 每个 `thread_id` 属于特定用户，而 id 在前端是明文（类似对话产品的 `chat_session_id`），不校验就会被人冒用写入、污染会话。

       * 复用会话管理里的 `required_owner(user_id, thread_id)`：查到即属于当前用户，查到的是会话对象；查不到返回空。

       * 为空则 `raise ChatThreadNotFoundException()`，语义正是「当前会话不存在或不属于当前用户」。

       * 顺序定死：先校验归属 → 再组织消息与 config → 再调 agent → 返回结果。

   * **Router 与全局状态**

       * `APIRouter(prefix="/api/v1/chat", tags=["与AI的对话接口"])`，用 `@router.post("")`；路径由 prefix 给出，响应不声明类型。

       * agent 在 lifespan 里挂到 `app.state.agent`；`app.state` 可理解为可动态加属性的字典，是全局 state，所有请求共享同一个 agent。

       * 接口参数声明 `Request`（变量名随意），用 `request.app.state.agent` 取回；也可写成 `app_request.app.state.agent`。

       * 取 service 抽成依赖 `get_chat_service(request, session=Depends(get_session))`，接口参数用 `service: ChatService = Depends(get_chat_service)` 注入。

       * `user_id: UUID = Header(...)` 从请求头取，与前面接口一致。

       * 写完必须在 main 里 `include_router(chat_router)`；此时共三个前缀：`/api/v1/chat`、`/api/v1/chat_threads`、`/api/v1/products`。

       * 实测：假 `thread_id` → 400「当前会话不存在或者不属于当前用户」；正确 id + `hi` → 返回完整问候语。

       * > **注意**：此时不能在前端联调，前端要的是 stream 模式，而这里是阻塞式、返回一个完整字符串。

## SSE 协议与数据格式

   * **服务端发送事件（Server-Sent Events，SSE）**

       * > **定义**：服务器主动向浏览器持续推送数据的协议；连接建立后不返回一次就结束，而是不断推送，从而实现流式输出。

       * 底层依然是 HTTP，只是在 HTTP 基础上做特殊处理；响应格式不是 `application/json`，而是 `text/event-stream`（事件流）。

       * 前端看到这个格式就用 `EventSource` 这套 API 接收；后端只要按格式响应即可，不需要写前端。

       * > **结论**：前端发起一次请求，服务端以流式不断推送，前端边收边渲染。

   * **一条消息的文本结构**

       * 标准 SSE 返回纯文本，应包含 `event`、`data`、`id`、`retry` 四个字段，不要求全含，至少要有 `data`。

       * 每行形如 `字段名: 字段值\n`，字段名后跟一个冒号加一个空格，行尾一个换行；一条消息的最后再加一个空行。

       * > **定义**：SSE 的一条消息 = 若干行 `字段名: 字段值\n`，再加一个额外的 `\n` 收尾。

       * 单 `\n` 只是换行，双 `\n` 才代表这条消息结束；一条消息里可以包含多个「字段名: 字段值」。

   * **三种常见形态**

       * 只有数据：`data: 你好` 加空行，最简形式。

       * 带事件类型：`event: message` + `data: {"content": "你"}` 加空行。

       * 结束信号：`event: done` + `data: end` 加空行；前端收到 `done` 就知道消息完事、断开连接。

       * 项目里统一用带 `event` 的写法：普通消息事件名 `message`，最后一条 `done`；不发纯数据，否则前端不知道何时结束。

       * 手拼字符串（拼 event、拼换行、拼 data、补 `\n\n`）可行但极其繁琐，这件事可交给框架。

## 流式对话的实现

   * **FastAPI 的两个改动**

       * 第一件：装饰器上把 `response_class` 改成 `EventSourceResponse`（事件流响应），不是 `response_model`。

       * 第二件：返回值不再用字符串，改用 `ServerSentEvent` 封装；该类自带 `id`、`event`、`data`、`retry` 四个字段，需要哪几个传哪几个，序列化成 SSE 文本由框架完成。

       * 导入位置：`from fastapi.sse import EventSourceResponse, ServerSentEvent`。

       * > **提示**：`response_model` 与 `response_class` 只差一个词，前者约束响应体里的数据结构，后者换掉整个响应类型；做 SSE 认准 `response_class`。

   * **return 与 yield 的分水岭**

       * `return` 交出去一次函数就结束，只能给一个结果；流式要不断产生、不断前送，必须用 `yield`。

       * `yield` 把一个东西临时交出、函数就地暂停，外部处理完再回到暂停处继续；一路产出就得到生成器，这就是「动态列表」。

   * **假数据先跑通一条流**

       * 循环 10 次 `yield ServerSentEvent(event="message", data=f"message_{i}")`，循环结束后再 `yield` 一条 `event="done"`。

       * `done` 必须写在循环外面，代表「没有了」；它的 `data` 写什么都无所谓，前端只认 `event` 类型。

       * 输出就是一行 event、一行 data、一个空行，一条接一条，与协议格式完全对得上。

       * > **易错点**：接口写成普通 `def` 却使用 `async for`，运行时报「不能用在非异步函数里」；`async for` 只对异步可迭代对象有意义，`range(10)` 这种同步序列用普通 `for` 就够。

   * **astream_events 换成真回答**

       * 方法名换成 `astream_events`，前两个参数（输入与 config）不动，加 `version="v3"`。

       * 返回的是流对象，要两层遍历：外层 `async for message in stream` 一条条拿消息，内层 `async for text in message.text` 从消息里抠文本片段。

       * 每抠出一段就封 `ServerSentEvent(event="message", data=text)` 并 `yield`；两层循环都结束后补 `yield ServerSentEvent(event="done", data="")`。

       * 与假数据版只差两处：数据从 `message_i` 变成真文本片段，`yield` 的位置从 router 挪到 service。

   * **router 只做转发**

       * service 是异步生成器，router 必须再套一层 `async for event in chat_service.chat_stream(...): yield event`，service 吐多少次就转多少次。

       * 整条通路是一条 yield 链，每层都不改内容、只做传递。

         ```text
         浏览器
           │  POST /chat/stream
           ▼
         router.chat_stream
           │  async for event in chat_service.chat_stream(...)
           ▼
         service.chat_stream
           │  agent.astream_events(..., version="v3")
           │  每抠出一个文本片段
           │  yield ServerSentEvent(event="message", data=片段)
           ▼
         router 原样 yield 出去
           │
           ▼
         EventSourceResponse 按 SSE 规范写给浏览器
           │  循环结束后 yield ServerSentEvent(event="done", data="")
           ▼
         浏览器逐条收到 event: message
         ```

       * 同一条链上埋着两个坑：接口函数必须是 `async def`；外层与内层的循环都必须是 `async for`。同一个知识点前后绊两次。

## 会话历史读取

   * **快照与 thread_id**

       * 会话历史有两条线：一条查出有哪些会话（会话列表），一条查出某个会话里聊过什么；后者不做，页面点进会话就是空的。

       * 每轮交互（用户一条、AI 一条）形成一个 state 快照；一个会话就是同一 `thread_id` 下摞起来的一堆快照。

       * 最后一份快照里已包含完整历史消息，取它一个就够，不必逐个翻。

       * 消息的写入全程没写过一行 SQL：创建 checkpointer、交给 agent、请求时 config 带 `thread_id`，LangChain 自动存用户与 AI 两条、下次自动捞历史拼上最新一句。

   * **读取 API 与 StateSnapshot**

       * `agent.get_state(config)` 取最后一个快照；`agent.get_state_history(config)` 取所有快照。

       * 异步 checkpointer 必须用 `aget_state` 并 `await`；漏写 await 会报「协程没有被等待」，看到 `coroutine` 就该想到补 await。

       * config 必须与对话时同一个 `thread_id`，对上了才查得到对应消息。

       * 返回对象是 `StateSnapshot`，历史消息路径为 `snapshot.values["messages"]`，先点 `values` 再按 key 取，两段缺一不可。

       * `pretty_print()` 是 LangChain 给所有 Message 自带的调试方法，真实接口不用它——要把消息返回给前端。

       * 不能直接去库里查：checkpointer 的表里内容经过加密与序列化，读不出可用的消息，想取历史只能找 LangChain 的接口。

   * **消息类型翻译成 role**

       * LangChain 不用字段标角色，用消息类型标；前端只认 `role` 与 `content` 两个字段。

       * 判断类型用 `isinstance`（返回 True/False，能进分支）；`type()` 只能打印类型名，做不了判断。

         | 消息类型 | 对应 role |
         | :--- | :--- |
         | `HumanMessage` | `user` |
         | `AIMessage` | `assistant` |
         | `SystemMessage` | `system` |
         | `ToolMessage` | `tool` |

       * 本接口只展示 `user` 与 `assistant`，判不出角色的一律 `continue` 跳过。

       * 每条转成 `{"role": role, "content": message.content}`，逐条 append 到列表。

   * **接口契约**

       * 请求方式 `GET`，路径 `/api/v1/chat/threads/{thread_id}/messages`，路径参数 `thread_id` 指明查哪条会话。

       * 响应体是一个对象，含 `thread_id` 与 `messages` 列表；列表每条只有 `role` 与 `content`。

       * 错误码：401 未登录；404 会话不存在或不属于当前用户。

       * 接口描述里还提到 interrupt 状态，本次只做历史消息，interrupt 不处理。

       * 路径必须带 `thread_id` 的理由与读快照的 config 相同：先定位会话，才能读它的状态。

       * 归属校验必须做：`thread_id` 在页面上是明文，任何人拿到 id 就能查历史等于泄露他人聊天记录。

   * **响应 Schema**

       * 落在会话管理侧的 schema 文件（历史属于会话管理，不属于对话）。

       * `ChatMessage`：`role: Literal["user", "assistant"]`、`content: str`。

       * `ChatHistoryResponse`：`thread_id: str`、`messages: list[ChatMessage]`。

       * 两个类都继承项目统一的 `BaseSchema`，不用裸 `BaseModel`。

       * > **提示**：`role` 用 `Literal` 而不是普通 `str`，两种之外的取值在校验阶段就被拦下，不用等前端发现角色不对。

   * **Service 三步走**

       * ① 归属校验：`await self.repository.find_owned(user_id, thread_id)`，为空抛 404「会话不存在或不属于当前用户」。

       * ② 组 `config = {"configurable": {"thread_id": thread_id}}`，`await self.agent.aget_state(config)` 拿快照。

       * ③ 逐条 `isinstance` 判类型、构造 `ChatMessage` 塞进列表，最后连同 `thread_id` 一起装进 `ChatHistoryResponse` 返回。

       * 有 schema 之后直接构造对象，不再像演示那样塞字典；`role` 与 `content` 两个属性一一对应。

       * > **易错点**：`continue` 不是「跳过这条消息的内容」，而是直接结束本轮循环进入下一条；在这里正合适——不是用户也不是 AI 的消息连角色都给不出，留着没用。

   * **agent 注入会话服务**

       * `ChatThreadService.__init__(session, agent)` 新增 agent 参数，赋值 `self.agent`。

       * 创建它的是 `ChatThreadRouter.get_service`，改为从 `request.app.state.agent` 取 agent 并传入：`ChatThreadService(session, request.app.state.agent)`。

       * 两步缺一不可：构造函数收下它、router 递进去；否则 service 里拿不到 agent。

       * 这也是后面删除能一行搞定的前提——agent 上挂着 checkpointer。

   * **空会话的 500**

       * 新建未说话、或消息被清空的会话：`values` 存在，但里面没有 `messages` 键，下标取值当场 KeyError → 500。

       * 第一次点进页面往往正是空会话，属于必然出现的情况，不能靠运气。

       * 改法：`snapshot.values.get("messages", [])`，缺失按空列表处理，判断与遍历合成一步，返回 `messages` 为空的响应。

       * > **易错点**：`values` 存在不等于 `messages` 存在；快照对象永远有 `values`，里面装什么取决于这条会话走到哪一步。

   * **content 的两种形态**

       * 报错为「输入字段无效」：要求 `string`、实际是 `list`，定位到 `content=message.content` 这一行，Pydantic 校验不通过 → 500。

       * 问题出在 `AIMessage`：它的 `content` 可能是内容块列表，一段文本包在块里，块内字段名为 `text`；`HumanMessage` 的 `content` 通常是字符串。

         ```text
         字符串   "我是保险顾问，可以帮你解答各种保险相关的问题"
         列表     [ { ... "text": "我是保险顾问，可以帮你解答……" ... } ]
         ```

         | `content` 的实际形态 | 直接取 `content` | 取 `text` |
         | :--- | :--- | :--- |
         | 纯字符串 | 得到字符串，能用 | 得到同一段字符串 |
         | 内容块列表 | 得到列表，校验失败报 500 | 自动遍历各块、拼成字符串 |

       * 不能写 `content[0]["text"]`：列表里可能有多个块，写死下标会丢内容，块结构一变又得改。

       * `message.text` 表面是属性、实际是包装成属性的函数：content 是字符串就原样返回，是列表就遍历拼成字符串；判空也一并换成 `text`。

       * > **提示**：只要把 LangChain 的消息往外送——接口返回、写日志、前端展示——一律用 `message.text`，别直接碰 `content`。

   * **Router 的四处改动**

       * 复制现成路由（如重命名那条）改四处：`@router.post` 改 `@router.get`、路径末尾补 `/messages`、`response_model` 换成 `ChatHistoryResponse`、函数体只收路径参数 `thread_id`。

       * 函数体只做转调：`return await service.get_chat_history(user_id, thread_id)`，无任何业务逻辑。

## 会话删除与快照清理

   * **删除的不对称**

       * 查历史只查最后一个快照；删历史必须删掉该 `thread_id` 下的所有快照。

       * 只删会话表中的记录，会话从列表消失，但 checkpoints 里的快照一条没动：既占空间，也让「删除」名不副实。

       * 残留快照还会被当成历史：下次带同一个 `thread_id` 来查，仍能读出旧消息，属于脏数据。

   * **删除 API 与调用位置**

       * 删除不走 agent 的对话接口，走 checkpointer：`delete_thread(thread_id)`，异步版为 `adelete_thread`，参数就是要删的会话 id。

       * 在 `remove` 里删完业务表记录后补一行：`await self.agent.checkpointer.delete_thread(thread_id)`；异步方法前面的 `await` 不能省。

       * 通过 agent 拿 checkpointer，是因为前面已把 agent 注进会话服务；router 不用改，原有删除流程原样保留。

       * 验证：删除返回 200 后，会话表对应记录消失，checkpoints 里该 thread 的快照也一并清空。

       * > **结论**：删除会话要动两处——业务表里的会话记录、LangGraph 里这条 thread 的全部快照；少删一处就是脏数据。

   * **查与删的分工**

         | 动作 | 用谁的方法 | 方法 | 作用范围 |
         | :--- | :--- | :--- | :--- |
         | 读取会话历史 | agent | `aget_state(config)` | 最后一个快照（含完整消息） |
         | 读取全部快照 | agent | `aget_state_history(config)` | 该 thread 的所有快照 |
         | 删除会话 | checkpointer | `adelete_thread(thread_id)` | 该 thread 的所有快照 |

       * 一句话记住：查用 agent 的方法，删用 checkpointer 的方法。
