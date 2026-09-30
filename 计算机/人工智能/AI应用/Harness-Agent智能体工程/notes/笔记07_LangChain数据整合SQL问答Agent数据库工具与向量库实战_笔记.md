# LangChain 数据整合：SQL 问答、Agent 数据库工具与向量库实战

## 表格数据问答的整体流程

* **表格数据问答**

    * > **定义**：让大语言模型根据用户问题与数据库表字段自动生成 SQL 语句，执行后把问题与查询结果交回大模型，最终给出自然语言答案；即用 LangChain 把关系型数据库作为数据源，对数据做问答、摘要等文本处理。

    * 固定三步：大模型（问题 + 表字段）生成 SQL → 拿 SQL 去数据库执行 → 大模型（问题 + 查询结果）合成自然答案；中间的「执行」由链或代理代劳。

    * 用户视角是黑匣子：不需要知道数据库在哪、有哪些表、表里有哪些字段；问题能转成 SQL 就转 SQL 去查，不需要查库的问题由大模型直接回答。

    * 数据库的查询操作可执行，增删改其实也都可以，但要在数据库上执行各种操作需要代理（agent）实现。

* **链（chain）与代理的分工**

    * 链：把组件进行组合与串接，结构相对简单。

    * 代理：可以根据需要多次循环查询数据库来回答问题，能执行的次数、循环的次数更多，当然也更复杂。

---

## SQLAlchemy 连接数据库

* **SQLAlchemy**

    * > **定义**：LangChain 用它解决「连接关系型数据库」这个绕不开的第一步；总体就是调用一个函数、传一个数据库 URL 参数，函数本身是 SQLAlchemy 内部的，关键是把 URL 写对。

    * 数据库 URL 格式：

```text
dialect + driver :// username : password @ host : port / database ?charset=utf8
方言 + 驱动 :// 用户名 : 密码 @ 主机 : 端口 / 数据库名 ?charset=编码格式
```

    * 方言（dialect）标明连的是哪种数据库；用户名与密码之间用冒号；@ 后接主机名与端口号；MySQL 默认端口 3306，端口号可省略。

    * URL 记不住可直接从网上抄对应数据库的写法，复制粘贴过来也能用。

* **SQLite 连接**

    * `sqlite://` 后面直接接 SQLite 的 db 文件路径，且要用绝对路径；它没有什么方言讲究。

    * UNIX 或苹果电脑一种写法，Windows 是另一种，Windows 上还可在字符串前面加个 `r`。

* **MySQL 三种连接写法**

| 连接写法 | 用什么驱动 | 说明 |
| :--- | :--- | :--- |
| `mysql://` | 默认 DBAPI | 不加驱动，需额外装 MySQL 连接器，麻烦，一般不用 |
| `mysql+mysqldb://` | mysqlclient | 用哪个驱动就安装哪个 |
| `mysql+pymysql://` | PyMySQL | `pip install pymysql` 安装后使用 |

* **连接的建立与自测**

    * `SQLDatabase.from_url(mysql_url)` 根据 URL 初始化得到数据库连接（导包自 `langchain.sql_database`）；URL 常用 format 拼参数生成。

    * 自测两招：`db.get_usable_table_names()` 打印库里可用的表名；`db.run("SELECT * FROM t_emp LIMIT 10;")` 执行一条测试 SQL（`LIMIT 10` 表示最多返回十条，末尾分号表示语句结束）。

    * 两张表、三张表还是十张表，表里多少条数据、表名叫什么，AI 大模型都无所谓；LangChain 会自动去找你的表名，其实它也是调 db 去找的。

    * 这一步完全不涉及 AI 大模型，先确认关系数据库这边可用。

---

## 只生成 SQL 的查询链

* **create_sql_query_chain**

    * > **定义**：传入大语言模型与数据库连接两个参数，创建一条生成数据库查询 SQL 的链；链的本质就是把组件进行组合。

    * `test_chain.invoke({"question": "请问员工表中有多少条数据"})`：invoke 传入的 input 对象是一个字典，键取名叫 question，值是问题。

    * > **易错点**：invoke 不能直接传列表或字符串，传了会报错。

    * 它只根据问题生成 SQL 语句、并不执行，拿到 SQL 还得自己去库里跑；只适合先试试水。

* **不同模型的输出差异**

| 模型 | 生成的 SQL 长什么样 | 会不会执行 |
| :--- | :--- | :--- |
| GPT-4 | 带 `SQLQuery:` 前缀（还带反引号），SQL 本身写得比较标准 | 不会 |
| GPT-3.5-turbo | 直接给 SQL，没有前缀、没有反引号，同样正确可运行 | 不会 |

---

## 生成、执行、回答的完整链

* **回答提示模板（PromptTemplate）**

    * > **定义**：给链末段大模型的提示模板，要求把用户问题、SQL 语句、SQL 执行结果三者结合作答，占位符为 `{question}`、`{query}`、`{result}`。

    * SQL 与执行结果不用用户给定：SQL 本来就是大模型生成的，执行交给数据库工具。

    * 模板里的文字可以改，参数名对上就行。

* **SQL 执行工具（QuerySQLDataBaseTool）**

    * > **定义**：包住数据库连接、负责执行 SQL 语句的工具，让「先生成 SQL、再执行 SQL」全程自动。

    * > **易错点**：初始化必须指定参数名 `QuerySQLDataBaseTool(db=db)`；直接把 db 传进去会报参数不对。

* **链的组装与执行**

```python
chain = (
    RunnablePassthrough.assign(query=test_chain)       # 生成 SQL，命名为 query
    .assign(result=itemgetter("query") | execute_sql)  # 执行 SQL，结果命名为 result
    | answer_prompt                                    # 问题+SQL+结果进模板
    | model
    | StrOutputParser()                                # 输出纯文本答案
)
```

    * `RunnablePassthrough.assign(query=test_chain)`：先生成一条 SQL 并挂名为 query；query 这个名字不能乱写，模板就是用 query 作键名传 SQL 的。

    * `.assign(result=itemgetter("query") | execute_sql)`：itemgetter（Python 内置工具函数）从字典取出 query，经管道交给执行工具运行，结果用 result 存起来；result 同样要传到模型里。

    * 跑通后问「员工表里面有多少条数据」，得到的是完整回答「员工表中有五条数据」，而不是直接甩过来一条 SQL。

    * > **易错点**：`StrOutputParser` 后面必须加括号才表示创建实例；漏括号时报错信息指到 invoke 那一行，真凶在定义链的那一行。

* **GPT-4 的 SQLQuery 前缀坑**

    * 链是直接拿着返回的 SQL 去执行的；换 GPT-4 时生成的 SQL 带 `SQLQuery:` 前缀，执行即报 SQL 语法错误——报错信息自己就知道「员工表」对应 t_emp，并指出实际语句前面多了 `SQLQuery:` 前缀。

    * 结论：这种链式写法下只能使用 GPT-3.5-turbo；改用代理则 GPT-3.5、GPT-4 都没问题。

---

## 工具代理整合数据库

* **SQLDatabaseToolkit（数据库工具包）**

    * > **定义**：把数据库与模型打包成一组数据库工具的工具包生成器；SQL 语句的生成、执行、执行结果的拿取，全在工具内部、Agent 内部实现，外面不用再操心。

    * `toolkit = SQLDatabaseToolkit(db=db, llm=model)` 打包时传数据库 db 与模型 llm；`tools = toolkit.get_tools()` 空参数构造方法一调就把工具拿到手。

* **代理执行器（executor）**

    * 用 langgraph（专门初始化 Agent 的库）的 `create_react_agent` 创建工具调用的代理；核心就三样参数：模型、工具、`state_modifier=system_message`（系统 message，即给 AI 的定位提示模板），后面的参数都可以不传。

```python
from langgraph.prebuilt import create_react_agent

agent_executor = create_react_agent(model, tools, state_modifier=system_message)
resp = agent_executor.invoke({"messages": [HumanMessage(content="请问员工表中有多少条数据")]})
```

    * 创建出来的准确说是一个「代理执行器」，它本身就是可以直接调用的。

    * invoke 接的是一个字典，键一定是 `messages`，值是一个列表、里面放 `HumanMessage` 把问题写进去；返回值因为传入的键叫 messages，收到的也是 messages，用 `resp["messages"]` 取出。

* **与链式写法的对比**

| 对比项 | 链式写法 | 代理写法 |
| :--- | :--- | :--- |
| 组合方式 | chain 套内嵌子 chain，模板、模型、解析器层层拼 | 一个工具代理全包 |
| 提示模板 | 要传 SQL、执行结果等一大堆参数 | 只给一个定位，其余全在工具内部 |
| 模型 | GPT-3.5（GPT-4 有前缀坑） | GPT-4 Turbo（GPT-3.5 也没问题） |

---

## 代理的系统提示词

* **定位条款**

    * 「你是一个被设计用来与 SQL 数据库交互的代理：给定一个输入的问题，创建一个语法正确的 SQL 语句并执行，然后查看查询结果，并返回答案」——给代理划定职责范围。

* **结果数上限**

    * 除非用户指定了想获取的具体数量，始终将 SQL 查询限制最多 10 个结果；防止生成的 SQL 一次性查回大量数据造成内存溢出——几十条、上百条没问题，上千条以上就有问题。

    * 可以按照相关列对结果进行排序，以返回数据库中最匹配的数据。

* **自查与纠错**

    * 在执行查询之前必须仔细检查查询语句；执行查询时出现错误，就重新编写查询语句再试——代理自己会纠错，这是它比一次性生成可靠的地方。

* **DML 禁令**

    * > **易错点**：明令禁止 AI 对数据库做任何 DML 语句（新增、修改、删除）。用户随口问一句「张三这个用户可能不需要了吧」，AI 要是开放了新增和删除权限，搞不好理解完就把张三那条数据删了，而用户并不是真的要删库里的数据。

* **固定工作流程**

    * 第一步先查看数据库中的表、看看可以查什么，不要跳过这一步；第二步查询最相关表的模式（schema）。

    * AI 通过「翻译」表名自行判断表里是什么：DEPT 判断是不是部门表，EMP 判断是不是员工表，person 判断里面是人、是用户还是什么。

---

## 代理的消息数组

* **三种消息类型**

| 消息类型 | 由谁产生 | 装的是什么 |
| :--- | :--- | :--- |
| HumanMessage | 用户 | 提出的问题 |
| AIMessage | 模型 | 下一步指令（比如去执行某条 SQL） |
| ToolMessage | 工具 | SQL 执行结果等信息 |

* **成对增长规律**

    * 代理根据需要多次循环查询数据库来回答问题，每一次查询至少产生两个对象：AIMessage（AI 给出的一条执行指令）+ ToolMessage（执行 SQL 后工具带回来的信息）。

    * 消息数组按「指令—结果」成对增长，一轮问答可包含若干对；一个简单问题的返回数组就有十个对象。

* **答案的取法**

    * 答案永远在数组最后一个元素里，下标就是 `len - 1`；把最后一个拿出来，取它的 `content` 属性即得答案字符串。

* **问题逐级加码的结果**

    * 「哪种性别的员工人数最多」：AI 内部先看员工表有没有性别字段、有哪些取值，再统计各性别人数、降序排序找最多；答「人数最多的性别是男性（man），共四名员工」，还自动把 man 翻译成「男性」，与人观察到的完全一样。

    * 「哪个部门下面的员工最多」：员工表只有 dept_id 字段跟部门关联，必须连表查询才能答出部门名字；数据里张七没有分配部门，其余四人中 id 为 3 的部门人数最多，查部门表确认是销售部——AI 把部门名字、共有几名员工全答了出来。

    * 「销售部下面有多少员工」「张三有几种角色」这类问题它都能识别出来；最基本的查询都能搞定，非常复杂、涉及复杂业务的暂时没办法。

---

## 从视频字幕到向量数据库

* **字幕获取（youtube_transcript_api，YouTube Transcript API）**

    * > **定义**：GitHub 上的一个 Python 开源项目，专注于获取 YouTube 视频的自动字幕，并提供方便的 API 接口。

    * 能拿到字幕的核心是它利用了 YouTube 的公开接口；不用这个库、直接调公开接口拿字幕，再手动把它们变成一个个 Document 也可以。

    * 覆盖范围有限：只有一部分视频可以拿，最新的一些视频肯定不行，稍微老一点（比如三个月之前）的倒是有可能；接口每更新一次，能获取到的可能就更多。

    * 它只针对 YouTube（不是 B 站，也不是优酷）；国内平台要有公开接口才行，优酷没有提供这样的公开接口。

* **Document（文档）**

    * > **定义**：LangChain 中构建向量空间、向量数据库的最基本单位。

    * 每个 Document 分为两部分：`page_content`（正文，即字幕内容，可能很长）与 `metadata`（元数据）。

```text
Document
├── page_content   正文内容（视频字幕全文）
└── metadata       元数据
    ├── source / url / title / author
    ├── description / length / view_count
    ├── publish_date   发布时间（精确到年月日时分秒）
    └── publish_year   发布年份（自加字段）
```

* **YoutubeLoader 爬取**

    * `YoutubeLoader.from_youtube_url(url, add_video_info=True).load()`：按 URL 把字幕拿出来，`add_video_info=True` 表示把视频元数据（标题、播放时长、发布时间）也拿出来。

    * 一个视频对应一个 Document，全部 `extend` 进 docs 列表，打印 `len(docs)` 知道总数；要先装 youtube_transcript_api 等两个包，不装拿不到工具。

* **给元数据补发布年份**

    * 给每个 Document 额外添加 `publish_year` 字段，便于后续按年份做元数据检索；抽年份用 `strftime`。

| 函数 | 方向 | 参数 |
| :--- | :--- | :--- |
| `strptime` | 字符串 → 日期时间对象 | 两个：待解析的字符串、格式 |
| `strftime` | 日期时间对象 → 指定格式的字符串 | 一个：格式 |

```python
doc.metadata["publish_year"] = int(doc.metadata["publish_date"].strftime("%Y"))
```

    * > **易错点**：抽年份只需 `strftime("%Y")` 拿到年份字符串、再套一层 `int()` 做类型转换；错用 `strptime` 会报「要给两个参数，实际只给了一个」——目标根本不是得到日期对象，`publish_date` 本身已是日期时间对象。

    * > **注意**：键名写错（如把 `publish_date` 写成 `publish_da`）时，爬数据其实已经成功，程序是卡在元数据转换这一步断掉的；正确键名可从打印出来的 Document 元数据里找。

* **持久化的动机**

    * 向量数据库放内存里，程序一停内存就没了，下次还得重新读文档；正确做法是第一次从文档读文本、转 Document、存进向量数据库并持久化到磁盘，第二次直接从磁盘加载。

```text
第一次（建库 + 持久化）：
  字幕/文档 → Document → 切割成片段 → 嵌入入库 → 保存到磁盘目录
第二次（直接用库）：
  磁盘目录 → 加载向量数据库
  （不再爬字幕、不再读 Word/PDF、不再重新做嵌入）
```

---

## 文本切分与向量库落盘

* **文本切分（RecursiveCharacterTextSplitter）**

    * 手上已经是 Document 时调用 `split_documents(docs)`，切完得到很多小片段；片段按 2000 个字符切、上下片段重复 30 个字符（`chunk_size=2000`、`chunk_overlap=30`，数字可调）。

```python
text_splitter = RecursiveCharacterTextSplitter(chunk_size=2000, chunk_overlap=30)
split_docs = text_splitter.split_documents(docs)
```

* **嵌入模型（OpenAIEmbeddings）**

    * 按道理应指定嵌入模型的名字，用 `OpenAIEmbeddings(model="text-embedding-3-small")`；small 是小的嵌入模型，速度快一些。

* **建库即持久化（Chroma.from_documents）**

```python
vectorstore = Chroma.from_documents(
    split_docs,                          # 切完之后的片段
    embedding,                           # 嵌入模型
    persist_directory=chroma_data_dir,   # 持久化目录，要持久化就必须传
)
```

    * 持久化目录用相对路径，运行时会在当前工程目录下自动创建。

    * 第一次运行速度很慢，因为要从网络把十几个视频的字幕全部加载下来，中途报错照文档处理即可。

    * 落库后一个视频对应一个 document：13 条视频就是 13 个 document。

* **磁盘上存了什么**

    * 目录里存着一个 Chroma 数据库，还有一堆文件；可理解为数据库本身存的是元数据，真正的数据存在目录里那一堆 bin 文件中。

    * > **易错点**：持久化不是说只存一个文件，整个目录要原封不动地留着；bin 文件千万不能删，删完之后加载数据就会出问题。

    * 元数据里能看到：source（来源）、title、description、view_count（观看次数）、视频预览图、publish_date，以及自加的 publish_year。

---

## 加载向量库与相似度搜索

* **从磁盘加载**

    * 把建库那段代码注释掉，直接用 Chroma 的构造函数加载，传两个参数：向量数据库所在的目录、嵌入模型（embedding function）。

```python
vector_store = Chroma(
    persist_directory=PERSIST_DIR,   # 建库时定义好的向量库目录常量
    embedding_function=embeddings,   # 嵌入模型
)
```

    * > **易错点**：这里的 embedding 必须跟之前创建向量数据库时用到的 embedding 保持一致，这样才能从磁盘中把向量数据库正确加载起来。

* **不接大模型的相似度搜索**

    * `similarity_search_with_score` 与 `similarity_search` 调谁都行，默认都是根据分数来进行搜索，只是传的参数不一样：用带 with_score 的那个，结果中会带分数。

    * 相似度搜索会把最相似的放在最前面，拿第一个即可；查询语句要写一句英语，因为字幕里面全是英语单词、没有中文。

    * > **易错点**：带分数搜索返回的是 `(Document, score)` 元组，直接对结果取 `.metadata` 会报「tuple 对象没有 metadata 属性」；应 `doc, score = result[0]` 解包，不想看到分数就去掉 with_score 用 `similarity_search`。

    * 搜回来的就是当初往库里存的 document：`.metadata` 里可继续取 source、URL、title、author、description、length、view_count、publish_date、publish_year；`page_content` 是整篇字幕内容（这个属性在 IDE 里没有提示）。

    * 这种搜索没有和大模型进行整合、不智能，定位是测试向量数据库相似搜索。

---

## 用 Pydantic 定义检索数据模型

* **Pydantic**

    * > **定义**：Python 里专门做数据管理的一个非常综合的库，包括数据的验证、数据的定义、模型的定义与序列化操作；web 开发常用它构建接收/响应数据的模型，自动完成校验、类型转换与接收。

    * 这里要定义的「模型」是数据模型——相当于 Java 程序员经常讲的 POJO 类，不是大语言模型。

* **为什么要数据模型**

    * 直接相似度搜索只能检索内容，没办法做到精细化检索：做不到按元数据字段查，比如「2024 年发布的所有教程」「长度不小于 1024 的教程」这类专基于元数据的条件。

    * 基于元数据的搜索条件要先验证、再序列化，所以要用 Pydantic 构建一个模型来接收搜索条件。

* **Search 模型的字段**

    * 继承 `BaseModel`；导包统一使用 `pydantic.v1`（编辑器会给 pydantic 与 pydantic.v1 两个选择，这是版本的区别）。

    * 字段一 `query: str`：要检索的那条语句，按内容做相似度搜索；写 `Field(default=None, description="搜索我们视频中的字幕")`，default 是默认值、description 是描述。

    * 字段二 `publish_year: Optional[int]`：Optional 来自 typing、表示可选类型（实际类型 int，可传可不传）；写 `Field(default=None, description="视频发布的年份")`。

    * 字段名不一定要跟 document 元数据里的字段名保持一致，保持一致只是代码看起来好看。

```python
from typing import Optional
from pydantic.v1 import BaseModel, Field

class Search(BaseModel):
    query: str = Field(default=None, description="搜索我们视频中的字幕")
    publish_year: Optional[int] = Field(default=None, description="视频发布的年份")
```

* **模型的用途**

    * 与大模型整合后，大模型根据检索时传入的 question 自动把数据信息抽取出来、封装成 Search 模型；再拿着 Search 模型去向量数据库里搜索。

---

## 结构化输出的智能检索链

* **系统提示的关键内容**

    * 定位「你是一个将用户问题转化为数据库查询的专家」（向量数据库的查询专家），可访问一个「构建大语言模型驱动的应用程序的软件库」的教程视频数据库——库里字幕都是跟 RAG 相关的构建教程。

    * > **提示**：如果有不熟悉的缩略语或者单词，不要试图改变它，直接把它作为要检索的向量去数据库里找相似度——RAG 在计算机编程语境里代表检索增强，换到另外的语境就是另一个意思，一「热心改写」检索就偏了。

* **with_structured_output（结构化输出）**

    * `structured_llm = model.with_structured_output(Search)`：把大语言模型与 Search 数据模型整合；这个数据模型既可以理解为结构化的搜索，也可以作为结构化的输出模型。

    * 链的输出是 Search 对象——一条「检索指令」，不是检索结果；这一步只是生成了要去向量数据库检索的指令，还没有真正去搜。

* **链的构造与效果**

```python
chain = {"question": RunnablePassthrough()} | prompt | structured_llm
```

    * `RunnablePassthrough` 表示先占住位置、值后面再传；模板里参数名固定叫 question。

    * 问「我怎么样去构建一个 RAG 的 Agent?」→ 解析出只按内容相似度检索的 Search（publish_year 为 None）；问「查找视频中关于 RAG 的部分，发布时间是 2023 年」→ 解析出内容条件加 2023 年的年份条件。

    * 大模型能把用户输入的问题自动变成搜索条件，靠的是用户输入的内容、提示模板，最重要的是大语言模型和数据模型的整合（`with_structured_output`）；前面若直接给一个模型，只会做内容搜索，做不了精细化。

* **retrieve 检索函数**

```python
def retrieve(search: Search) -> list[Document]:
    _filter = None                                   # 先给空值
    if search.publish_year:                          # 年份存在才拼过滤条件
        _filter = {"publish_year": {"$eq": search.publish_year}}
    return vector_store.similarity_search(
        query=search.query,      # 内容相似度检索
        filter=_filter,          # 额外的元数据过滤条件
    )
```

    * query 与 publish_year 分开处理：query 一定是根据内容做相似度检索；publish_year 不是根据内容，是判断元数据是否与检索值具有相等关系。

    * `$eq` 是 Chroma 的固定语法，代表「等于」；过滤条件的键要跟 document 元数据里的字段保持一致。

    * > **易错点**：`_filter` 必须在 if 之前先置为 None；否则问题里不带年份时这个变量根本没被定义，后面引用直接报错。

    * 这才叫真正的向量数据库检索：不纯粹只根据向量之间的相似度，还可以基于特定的值、额外的搜索条件，两者结合。

* **链尾接函数与验证**

    * `new_chain = chain | retrieve`：原来这条链根据传入问题搜索完得到的指令，直接传给 retrieve 函数执行真正的检索；invoke 里面仍传自然语言问题。

    * 问「2023 年关于 RAG 的一个教程」→ 返回 3 条且排名第一个就是 2023 年的：其他字幕里很可能也含 RAG 教程，但发布时间不是 2023 年，被过滤掉了。

    * 问题里本身不含年份（如「检索一下所有 RAG 的教程」）→ 大模型抽不出任何年份，不满足 if 条件、`_filter` 保持 None，只按 query 相似度搜索，结果多了：一共四条。

```text
用户问题 → prompt + model.with_structured_output(Search) → Search 检索指令
        → retrieve：publish_year 有值拼 $eq 过滤，没值 filter=None
        → vector_store.similarity_search(query, filter) → list[Document]
```

* **两种检索方式对比**

| 对比项 | 直接相似度搜索 | 结构化智能检索 |
| :--- | :--- | :--- |
| 检索依据 | 只有内容相似度 | 内容相似度 + 元数据过滤 |
| 检索条件来源 | 手写死在代码里 | 大模型从用户问题里自动抽取 |
| 能力边界 | 不智能，做不到精细化 | 可按发布年份等元数据字段精细过滤 |
