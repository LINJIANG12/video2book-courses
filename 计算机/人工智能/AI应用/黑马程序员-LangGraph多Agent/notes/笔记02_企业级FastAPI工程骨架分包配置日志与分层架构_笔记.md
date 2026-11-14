# 企业级 FastAPI 工程骨架：分包、配置、日志与分层架构

## 项目背景与系统架构

### 保险商城的业务定位

* **安心宝**

    * > **定义**：一个保险优选平台（保险商城），本身不是保险公司，而是把各家保险公司的产品集中到一个服务里，让用户一站式挑选并组合保障方案。

    * 只做面向人的保险，不卖车险；卖的是虚拟产品（保险），不是实物。

    * 核心痛点：普通用户不了解险种、条款与理赔流程，需要专业顾问科普知识、解释条款、筛选产品并组合方案；这个顾问由 AI 承担，功能是产品答疑、条款解释、理赔指引。

    * 保险从来不是单一产品，针对一个人或一个家庭可以组合出不同方案，组合起来才有更全面的保障。

* **四类人身险**

    | 险种 | 解决的问题 | 赔付方式 |
    | :--- | :--- | :--- |
    | 百万医疗险 | 社保报完之后剩下的那部分医疗费 | 报销型，通常有约 1 万元免赔额，超出部分最高可报几百万 |
    | 重疾险 | 癌症这类治疗费用极高的重症 | 确诊后直接赔一笔现金，50 万、200 万都可能 |
    | 寿险 | 家庭经济支柱突然身故 | 赔一笔钱给家人，保额可达几百万，保费也最贵 |
    | 意外险 | 突发意外造成的伤残或身故 | 覆盖「残而不死」带来的长期经济负担 |

    * 百万医疗险是报销型主力：社保只报一部分，甲类药可能报 70% 到 80%，乙类丙类比例不同，国外药不进社保；超过免赔额的部分由它接上。

    * 重疾险赔的是现金，不只可以治病，还房贷或干别的都行。

    * 产品本身有种类之分，四大险种就是业务里「分类」概念的来源，对应数据库中的 `category` 字段。

### 三层架构与前后端分离

* **用户体验层（frontend）**

    * > **定义**：用户直接看到的交互界面，采用 React + Ant Design，提供浏览产品、登录注册、进入智能客服的页面。

    * 前端是静态页面，自身没有数据，也实现不了具体功能，功能要靠后端。

* **后端服务层（agent-service）**

    * > **定义**：Python + FastAPI 实现的服务端，`agent` 在人工智能领域称智能体，`agent-service` 即智能体服务。

    * 传统保险业务：登录认证、产品查询、保费试算、保险方案生成、保单增删改查、理赔。

    * 智能体业务：意图识别与路由、保险组合推荐、条款问答、理赔咨询。

* **基础设施层**

    * > **定义**：后端依赖的外部设施，包括 PostgreSQL（关系型数据库）、Milvus（向量数据库）与各类模型。

* 请求主线：浏览器访问前端服务器拿到页面；页面需要数据时调后端服务层；后端再把活派给数据库和模型。

* 真实企业中传统保险业务与智能顾问业务是分离的，传统业务通常用 Java 写；当前阶段两者合并用 Python 实现，后续阶段再拆开。

### 技术选型

* **后端**

    * 框架 FastAPI，智能体编排 LangGraph + LangChain，大模型 DeepSeek。

    * 包管理 uv，部署 Docker Compose。

* **数据与前端**

    * 关系型数据库 PostgreSQL，向量数据库 Milvus。

    * 前端 React + Ant Design。

---

## 开发环境与基础设施

### Notebook 交互式环境

* **Jupyter Notebook（`.ipynb`）**

    * > **定义**：可交互的 Python 学习笔记，把笔记与可执行代码混在同一个文件里。

    * 后缀含义：I 是 interactive（交互），PY 是 Python，NB 是 notebook。

    * `cell`（单元格）分两类：code cell 写代码，markdown cell 写笔记。

    * 结果保留：运行结果就在代码正下方，关掉文件再打开，代码与结果都还在。

    * 变量可观察：Jupyter Variables 面板能列出已定义变量及其值，随时盯着它变。

    * 单元格联动：后一个单元格可以直接使用前一个单元格定义的变量，命名空间是共享的。

    * 运行方式：点单元格旁的三角形，或选中后按 `Shift + Enter`。

    * 用途：知识点验证、变量观察、项目演示；演示与测试放在 `notebooks` 目录，不写进项目代码包。

* **与传统 `.py` 文件的差异**

    * 传统 `.py` 文件为了看一眼变量的值，得把整个文件跑一遍，跑完窗口一关什么也不剩，变量下次无法再观察。

    * Notebook 把笔记与代码混在一起，代码可直接运行、结果可直接看，变量随时可见；写笔记的部分用 markdown cell，形成带层级、可跳转的结构。

### Docker Compose 基础设施

* **PostgreSQL**

    * 端口 `5432`，不是 MySQL 的 `3306`，写错就连不上；库名 `insurance`、用户名 `insurance`、密码 `insurance123`。

    * Python 领域与智能体领域通常用 PostgreSQL；SQL 语句有统一标准，SQLAlchemy 又是面向对象调用，换库不影响业务代码。

* **Milvus**

    * 向量数据库本体端口 `19530`。

    * 依赖 etcd（`2379`，集群管理与元数据存储）、MinIO（`9000` / `9001`，对象存储）、Attu（`3000`，图形化管理界面）。

    * 访问 `服务器IP:3000` 打开的是 Attu，不是 Milvus 本体，两个地址别搞混。

    * 首次访问要创建管理员账号，至少八位，且包含大小写字母、数字和符号。

* **部署步骤**

    * 前提：关闭防火墙。

    * 把部署目录整体上传到服务器 `/root` 下，进入该目录执行 `docker compose up -d`；它会自建网络、数据卷、容器，镜像已提前给好。

    * 启动顺序分三梯队：etcd + MinIO → Milvus → Attu；原因是 Milvus 依赖前两者，Attu 管理 Milvus。

    * 全部状态变成 `healthy` 或 `started` 后，用 `docker compose ps` 确认五个容器都在。

    * 验证 PostgreSQL：用客户端填服务器 IP、`5432`、用户名 `insurance`、密码 `insurance123`、库 `insurance`，能看到自动初始化好的表。

### 前端启动

* 代码目录路径不要有中文、不要有特殊字符；整个项目目录一起拷贝，不只拷前端，因为后端也要建在里面，前后端同根便于 AI 编程工具同时看到两边。

* `npm install` 安装依赖；`npm run dev` 启动开发模式——`develop` 简称 `dev`，是开发模式下的本地部署，不是真实部署。

* 启动成功后端口是 `5173`，访问 `http://localhost:5173`。

* 页面能打开但提示「产品加载失败」属正常：后端还没写，前端只有页面样式，没有数据。

### 后端项目初始化

* 在项目根目录下创建 `agent-service`；项目类型选普通 Python 项目，依赖管理选 uv，Python 版本 `3.14`。

* 依赖统一写进 `pyproject.toml` 的 `dependencies`，并把 uv 的依赖库地址指向清华镜像源，避免从国外慢速下载。

* 安装方式两种：点编辑器里的黄色提示自动安装，或在终端执行 `uv sync`；黄色标识全部消失即依赖装好。

---

## 分包策略与分层职责

### 五类模块

* **router（路由层）**

    * > **定义**：请求进入服务端的第一站，只负责接收请求、校验前端参数、把处理结果返回前端。

    * 接口用 `app.get`、`app.post` 等形式声明，包含请求方式、请求路径、请求参数、响应结果。

    * 不处理业务：路由层一旦出现业务判断或数据库查询，分层就名存实亡。

* **service（业务层）**

    * > **定义**：组织业务流程、承载业务逻辑的层。

    * 处理过程中需要查库时调 repository；纯计算的业务可以只停在 service，不必碰库。

* **repository（仓储层）**

    * > **定义**：只做数据库增删改查的层，没有业务。

    * 把若干 CRUD 组合起来形成业务，是 service 的职责。

* **schemas（数据契约）**

    * > **定义**：对请求数据与响应数据的约束，对应前后端共同遵循的接口文档。

    * 落到代码里就是继承 Pydantic `BaseModel` 的类，统一放在 `schemas` 模块。

* **models（表映射）**

    * > **定义**：SQLAlchemy ORM 的表映射类，一张表对应一个类，列对应字段，一行数据对应一个对象。

    * repository 靠它知道 Python 类怎么对应数据库表。

### 两种分包方式

* **按职责分包**

    * 总包 `app`，入口 `main.py`；`core` 放配置、日志、统一异常处理，`infrastructure` 放数据库、Redis、模型等外部设施，`common` 放通用代码。

    * 业务侧按职责平铺：`api/router`、`service`、`repository`、`schemas`、`models` 各一个包，包内再按业务分文件。

    * 中小型项目够用；业务达到十几个、几十上百个时，每个包里文件过多，不好管理。

* **按业务分包**

    * 保留 `core`、`infrastructure`、`common`，新增 `modules` 包，所有业务收进去，一个业务一个子包（如 `product`、`chat`、`user`）。

    * 每个业务包自带五件套：`router.py`、`service.py`、`repository.py`、`models.py`、`schemas.py`，最多五个文件。

    * 高内聚：改一个业务只动自己的包；删一个业务整包删除，不留半截代码；业务扩展只是多一个包，不影响别人。

* **结论**：中小型项目按职责分包，大型项目按业务功能分包；本项目采用按业务分包。

* **包与目录的区别**

    * `app`、`core`、`common`、`infrastructure`、`modules` 及各业务包都要建成 Python package，图标带圈并自动生成 `__init__.py`；建成普通 directory 导入时会出问题。

    * `notebooks` 反而要建成普通目录，用来放每天的演示与测试，测试和演示不属于项目代码。

### 开发顺序

* `schema` 与 `model` 是基础：schema 约束 router 的入参与出参，model 为 repository 提供表映射。

* 调用依赖是 router → service → repository，所以开发从下往上：schema / model → repository → service → router。

* 顺着依赖方向做，每一步都有已经存在的下层可调用，不会写出写了一半只能干等的空壳。

---

## 配置管理

### 配置外置与 .env

* 属于配置的信息：项目端口、项目名称、DeepSeek 的密钥与地址、数据库的地址端口用户名密码、日志级别。

* 写死在代码里的三个麻烦：分散在项目各个角落不好管理；开发一套值、线上一套值时要满项目找着改；密钥写在代码里有泄露风险。

* 全塞系统环境变量也不合适：一两个密钥还行，配置一多配起来繁琐、改起来不便。

* 企业通行做法：项目根目录放一个 `.env` 文件，把关键参数集中写进去，代码读取该文件，修改也只改这一处。

* 上线时密钥类隐私值用系统环境变量或命令行临时参数在部署那一刻替换，文件里留的是假值，因而不用担心泄露。

* 真实配置 = `.env` 文件 + 系统环境变量 + 命令行参数的组合。

### Pydantic Settings 的来源优先级

* **pydantic-settings**

    * > **定义**：能同时兼容文件、环境变量、命令行等多种来源的配置管理库，按固定优先级逐层取值。

    * 依赖名 `pydantic-settings`，创建项目时已安装。

| 优先级 | 来源 | 典型用途 |
| :--- | :--- | :--- |
| 1（最高） | 项目执行时的 CLI 命令行参数 | 部署那一刻临时传密钥 |
| 2 | `Settings` 构造函数传参 | 代码里显式指定，几乎很少用 |
| 3 | 系统环境变量 | 线上存放密钥 |
| 4 | `.env` 文件 | 开发阶段放全部配置 |
| 5 | secrets 目录 | Linux 下专门存放密码的目录 |
| 6（最低） | 字段默认值 | 兜底 |

* 加载规则：从上往下逐层看，某一层读到了值，后面碰到相同字段就不再使用；只有这一层没有，才继续往下找。

* 例：`.env` 里配了 host、port、debug、API 密钥，系统环境变量里也配了 port 与 debug 且值不同，最终 port 取 `9000`、debug 取 `false`，host 取 `.env` 的 `0.0.0.0`，密钥也取自 `.env`；字段默认值里的 `127.0.0.1` 等全部不生效。

* 策略：开发阶段配置全放 `.env`；线上大部分配置仍留在文件里，只把密钥用命令行参数或系统环境变量覆盖，文件一个字都不用改。

### 配置类结构

* **app_root**

    * > **定义**：项目根目录的绝对路径，由当前文件位置向上两级得到。

    * `Path(__file__).resolve().parents[2]`：`parents[0]` 是 `core`，`parents[1]` 是 `app`，`parents[2]` 是项目根目录，`.env` 正好放在那里。

    * 不写死盘符：项目上线位置会变，但目录结构不变；用相对层级定位，项目挪到哪儿都能找到 `.env`。

    * 硬要求：项目目录结构必须与约定一致，结构不一样相对层级就对不上，`.env` 也就找不到。

* **EnvSettings**

    * > **定义**：所有配置类的公共父类，统一声明 `.env` 的位置与读取规则。

    * 必须继承 `BaseSettings`，不继承则配置读取不生效。

    * `model_config = SettingsConfigDict(env_file=app_root / ".env", env_file_encoding="utf-8", extra="ignore")`。

    * `env_file` 指定文件位置，`env_file_encoding` 用 `utf-8` 读（文件里可能有中文），`extra="ignore"` 忽略未定义字段。

    * `extra="ignore"` 的必要性：系统环境变量条目极多，除了自己配的还有机器自带的，不全读进来才不会多出一堆未定义字段。

* **四组配置类**（都继承 `EnvSettings`，共享同一份 `.env` 位置与读取规则）

    | 配置类 | 字段 | 别名 | 取自 `.env` 的哪一块 |
    | :--- | :--- | :--- | :--- |
    | `AppSettings` | `name` / `host` / `port` / `debug` | `APP_NAME` / `APP_HOST` / `APP_PORT` / `APP_DEBUG` | 应用配置 |
    | `DatabaseSettings` | `host` / `port` / `name` / `user` / `password` | `DB_HOST` / `DB_PORT` / `DB_NAME` / `DB_USER` / `DB_PASSWORD` | 数据库配置 |
    | `LLMSettings` | `api_key` / `model` | `DEEPSEEK_API_KEY` / `DEEPSEEK_MODEL` | 模型配置 |
    | `LogSettings` | `level` | `LOG_LEVEL` | 日志配置 |

    * 不把四组塞进一个类：塞得下，但类里代码会越来越多，拆开各管一块更方便管理。

* **别名（alias）**

    * > **定义**：`Field(alias=...)` 声明的、Pydantic 去 `.env` 或系统环境变量里查值用的键。

    * 变量名是给开发者用的，小写、写代码顺手；别名必须与配置文件里的键严格一致，变量名可以随你起。

    * 每个字段带 `default`，默认值优先级最低，只有环境变量、命令行、`.env` 都找不到时才生效；`.env` 有值时默认值一个都不生效。

### 计算字段与数据库 URL

* **数据库连接 URL 的组成**

    ```text
    postgresql+asyncpg://user:password@host:5432/dbname
    └──────┬────────┘   └─┬─┘ └──┬───┘ └─┬─┘ └┬─┘ └──┬──┘
        协议+驱动       用户名   密码    主机  端口  数据库名
    ```

* **@computed_field + @property**

    * > **定义**：把一段拼接逻辑声明为计算字段，访问时像普通属性一样取值。

    * 函数体调用 `PostgresDsn.build(scheme="postgresql+asyncpg", username=self.user, password=self.password, host=self.host, port=self.port, path=self.name)`，把零件拼成完整 URL。

    * 写成函数而不是直接赋值：拼接是多步逻辑，没法用一个等号表达完；`@property` 让它能像属性一样被访问，`@computed_field` 声明这个字段是计算出来的。

    * 协议头 `postgresql+asyncpg`：加号前是数据库类型，加号后是底层驱动；连 MySQL 时则是 `mysql+aiomysql`。

    * > **易错点**：装饰器顺序不能反，`@computed_field` 要在上、`@property` 在下；写反了它就不是计算字段，序列化时也拿不到 `url`。

    * 换数据库时这段代码一行不用改，改的只是配置里 URL 的 scheme（协议）。

### 配置读取与扩展

* **Settings 汇总**

    * `Settings` 继承 `BaseSettings`，四个字段 `app` / `db` / `llm` / `log` 分别用 `Field(default_factory=对应配置类)` 构造，把四组配置集成成一个对象。

    * 文件末尾 `settings = Settings()` 立即初始化；构造不传参，等于主动放弃「构造函数传参」这条来源，值全部来自 `.env`、系统环境变量、命令行参数。

    * 模块一加载就自动读取配置；任何模块要用配置，导入 `settings` 即可。

* **取用路径**

    * `settings.app.name` 取应用名，`settings.app.host` 取主机地址，`settings.db.url` 取拼好的连接串，`settings.llm.api_key` 取模型密钥，`settings.log.level` 取日志级别。

    * 查看全部配置：`print(settings.model_dump_json(indent=2))`；`BaseSettings` 本质是 Pydantic 的 `BaseModel`，自带 `model_dump_json`，`indent` 控制缩进。

* **扩展单项配置**

    * 定位到对应的配置类，加一个字段；变量名随意，`alias` 必须与 `.env` 或系统环境变量里的键一致，默认值想配就配、不想配可删。

* **扩展整组配置**

    * 把某个配置类整体复制一份，改类名，再到 `Settings` 里补一个对应的 `default_factory` 字段。

* **验证位置**

    * 不在 `app` 里写测试；在项目根目录建 `notebooks/<日期>/`，用 `from app.core.config import settings` 直接验证，这才是演示与测试的位置。

---

## 统一日志与启动模块

### structlog 的配置与取用

* **structlog**

    * > **定义**：第三方结构化日志库，输出经过格式化、结构更清晰、带颜色。

    * 代码属于核心配置，与配置模块放在一起；真正要用的只有两个函数：配置日志、获取日志。

    * `configure_logging` 是一次性的：进程里执行过一次，全局的日志格式、输出方式、级别就定下来，别处不再执行。

    * `get_logger` 是随用随取的：哪个模块要打日志，就在哪个模块里取一个记录器。

    * 取记录器传模块名，`__name__` 是 Python 每个模块的内置变量，日志会带上如 `app.main`、`app.infra.database` 的标识，一眼看出是谁记的。

* **级别从配置读**

    * `configure_logging(level=settings.log.level)`，不要写死；要改级别只改 `.env` 的 `LOG_LEVEL`，不用改代码、不用重新发布。

    * 日志级别有 debug、info、warning 等；健康检查属于正常事件，用 `info` 就够。

    * > **易错点**：`configure_logging` 只该在启动模块执行一次，其余模块一律只调 `get_logger`；数据库模块里一个 `configure_logging` 都没有。

### 启动模块的三件事

* **位置**

    * 启动模块 `main.py` 直接放在 `app` 包下，不要塞进 `core`：`core` 装配置、日志这类通用能力，启动模块是项目的门面，职责不同。

* **固定顺序**

    * 配日志 → 创建 FastAPI 对象 → 挂接口；最后用 uvicorn 拉起来。

    * 创建实例：`FastAPI(title=settings.app.name, debug=settings.app.debug, lifespan=lifespan)`；`title` 让接口文档知道项目叫什么。

* **debug 参数与日志级别的分工**

    * `debug` 控制 FastAPI 运行时是否用调试模式，决定日志「记不记、记多细」。

    * 日志级别决定「看不看得到」：级别是 debug 看到的多，级别是 info 看到的少。

    * > **提示**：`debug` 管记录详细程度，日志级别管显示门槛；开了 debug 却没看到更多日志，先去看级别，别怀疑参数没生效。

### 健康检查与 uvicorn 验证

* **健康检查接口**

    * `@app.get("/health", description="健康检测接口")`，函数写成 `async def`，项目里的接口统一是异步的。

    * 函数体返回 `{"status": "ok"}`，返回值用字典装字符串即可。

    * `description` 会直接出现在自动生成的接口文档里；顺手用 `info` 记一条日志，证明接口确实被调用。

* **uvicorn 启动**

    * `if __name__ == "__main__": uvicorn.run("app.main:app", host=settings.app.host, port=settings.app.port)`。

    * 第一个参数格式是「模块路径:FastAPI 对象变量名」；`main.py` 在 `app` 包里，所以必须写 `app.main:app`，不能写 `main:app`。

    * 主机与端口不写死，从 `settings.app.host` 与 `settings.app.port` 读；本项目端口为 `8001`。

* **验证**

    * 打开 `http://127.0.0.1:8001/docs`，这是 FastAPI 自动生成的接口文档，点 `try it out` 发请求能拿到 `{"status": "ok"}`。

    * GET 请求也可以直接在浏览器地址栏敲 `http://localhost:8001/health`。

    * 控制台日志级别 info、内容是健康检查正常、记录者是 `app.main`，说明 `get_logger(__name__)` 生效。

---

## 数据库连接与生命周期

### 连接池配置项

* **四项核心参数**

    | 配置项 | 作用 |
    | :--- | :--- |
    | `echo` | 是否让 SQLAlchemy 把执行的 SQL 语句输出出来 |
    | `pool_size` | 连接池大小，即常驻连接数 |
    | `max_overflow` | 连接池溢出时最多还能再加多少连接 |
    | `pool_recycle` | 连接的回收时间，到期就换新连接 |

    * 这四项在配置类里都有默认值，不必写进 `.env`，要改时按别名在 `.env` 里覆盖。

    * > **提示**：`echo` 打开后控制台会刷出大量 SQL，调试时有用，平时建议关掉，否则日志会被淹没。

* **另外两项由引擎读取**

    * `pool_timeout`：取连接的超时时间。

    * `pool_pre_ping`：每次取连接前是否先做检查。

### 引擎与会话工厂

* **安全前置**

    * 初始化一上来先检查数据库 URL，没配置就直接 `raise` 一个错误，避免带着空地址去连库、报出一堆看不懂的底层异常。

* **异步引擎**

    * `create_async_engine(url, echo, pool_size, max_overflow, pool_recycle, pool_timeout, pool_pre_ping)`，参数全部从 `settings` 读，没有一个是写死的。

    * 与连 MySQL 的写法几乎没区别，唯一区别在 URL 协议头；URL 在配置里，代码根本看不出用的是哪种数据库。

* **会话工厂**

    * `AsyncSessionFactory = async_sessionmaker(engine, autoflush=False, expire_on_commit=False)`；名字大写开头，表示它是工厂而不是普通变量。

    * `autoflush=False`：执行查询前不自动把挂起的改动刷进数据库，需要时手动刷，行为更可控。

    * `expire_on_commit=False`：提交后不把对象标记为过期；默认行为下 session 提交或关闭后对象属性会失效，关掉后 session 关闭仍可访问对象，更新操作省事很多。

### 三个工具函数

* **get_session**

    * `async with AsyncSessionFactory() as session: yield session`；函数执行到 `yield` 暂停，谁拿到谁用，用完控制权回来，上下文管理器自动关闭 session。

    * 专供 FastAPI 依赖注入；以前手写要自己 `commit`、异常 `rollback`、最后 `close`，上下文管理器把这些收尾全包了。

* **close_database**

    * `await engine.dispose()` 销毁整个连接池，再记一条日志；应该在项目关闭时执行。

* **check_database**

    * 用 `engine.connect()` 拿连接，随便执行一条 `select 1`；语句本身不重要，重要的是让引擎真正建立连接。

    * > **定义**：引擎是惰性的——模块加载时只把引擎与 session 工厂创建出来，连接池里一个连接都没有，真正的连接在第一次向数据库发请求时才建立。

    * `check_database` 才是「初始化数据库连接」的函数，一执行引擎就按 `pool_size` 把连接建好。

    * > **易错点**：不要以为数据库模块被 import 就等于连上了数据库。

### lifespan 生命周期

* **钩子结构**

    * 用 `@asynccontextmanager` 装饰（从 `contextlib` 导入），函数异步、名为 `lifespan`、参数接收 `app`，类型是 `FastAPI`。

    * 结构是 `try / yield / finally`：`yield` 之前是启动阶段（`await check_database()`），`yield` 之后写在 `finally` 里是关闭阶段（`await close_database()`），中间这段时间 FastAPI 正常处理请求。

    * 数据库函数的导入放在函数内部、`try` 之前：放文件顶部会在模块加载时连带加载数据库模块，放函数里实现延迟加载；放 `try` 里又会「看不见」，所以取这个位置。

    * 创建 FastAPI 对象时加 `lifespan=lifespan` 参数，FastAPI 创建时调用它并把 `app` 传进去，关闭时也会走到 `finally`。

* **验证**

    * 重启项目：先打印「项目启动中，正在初始化……」，紧接着「数据库连接初始化成功」，记录者是 `app.infra.database`。

    * 停掉项目：日志出现「数据库连接已关闭」，一进一出都对上即绑定成功。

---

## 接口规范与产品列表契约

### 前后端分离的请求链路

* **部署形态**

    * 前端与后端各自独立部署，不在同一台服务器、不搅在一个工程里。

* **静态资源链路**

    * 浏览器请求 HTML、CSS、JS 等静态资源 → 前端服务器本地就有这些文件，直接读文件返回 → 浏览器渲染出页面。

    * 此时页面只有「壳」，布局样式都在，产品列表是空的，因为产品数据还没查。

* **动态数据链路**

    * 页面发出的动态数据请求仍然先发到前端服务器，地址与页面同源（`localhost:5173`），只是路径变成 `/api/v1/...` 这种形式。

    * 前端服务器看到这个路径就知道是 API 请求，转给后端 FastAPI（`localhost:8001`），后端查数据库后原路返回。

    ```text
    浏览器
      │ ① 请求 html、css、js 等静态资源
      ▼
    前端服务器（localhost:5173）
      │ ② 静态资源在本地，直接读文件返回
      ▼
    浏览器：先渲染页面（只有壳，没有数据）

    浏览器
      │ ③ /api/v1/... 请求仍先到前端服务器
      ▼
    前端服务器：识别出 API 请求，转发给后端
      ▼
    后端 FastAPI（localhost:8001）
      │ ④ 查询数据库
      ▼
    数据库 → 后端 → 前端服务器 → 浏览器：渲染动态数据
    ```

    * 读图关键在③：动态请求不是浏览器直接打到后端，而是同源发给前端服务器，由它识别路径再转发。

### 接口文档四要素

* **四个必答问题**

    | 前端的问题 | 文档里的对应项 |
    | :--- | :--- |
    | 去哪里拿 | 请求的 URL 路径 |
    | 怎么拿 | 请求方式（GET、POST 等） |
    | 带什么条件 | 请求参数 |
    | 拿到的东西长什么样 | 响应数据的格式 |

* 文档前置的公共信息：接口地址、统一路径前缀、结构格式说明、认证规则、数据字典；统一前缀决定所有业务接口的 URL 长什么样，认证规则决定请求要不要带 token。

* 文档把「等对方」这件事消掉了：前端知道自己该发什么请求，后端知道自己该实现什么接口，两边可以同时开工。

* > **提示**：接口文档不是参考读物，是施工图；实现时以文档为准，不要凭自己对「合理」的想象去改字段名或路径。

### RESTful 路径约定

* `/api/v1` 是统一的路径前缀，`products` 是请求的资源名称；「前缀 + 资源名」的写法符合 RESTful 风格。

* 路径里出现的是名词（资源）而不是动词（动作），所以好记，也不容易起歧义。

* router 的 `prefix` 抽出公共前缀，个别接口路径不同时再在装饰器里补。

### 产品列表接口契约

* **请求**

    * 方式 `GET`，路径 `/api/v1/products`；定位是查询有效的保险产品，可按产品分类过滤，直接返回产品列表。

    * 请求参数只有 `category`，类型字符串，非必填：不传查全部有效产品，传了只查该分类。

    * 四个分类值：`medical`（医疗险）、`critical_illness`（重疾险）、`life`（寿险）、`accident`（意外险）。

    * 这四个值不是凭空定的，数据库 `products` 表的 `category` 字段存的就是它们；后端拿到参数后不做任何翻译或映射，直接拿去过滤。

* **有效产品**

    * `products` 表有 `status` 字段，取值 `active` 与 `inactive`；`active` 是有效。

    * 默认查询必须加 `status = 'active'` 条件，非 active 的产品一律不返回。

    * > **易错点**：这个条件很容易漏；漏了不会报错，接口照样返回数据，但会把已下架产品一起返回，数据看起来「能用」其实是错的，这种 bug 最难发现。

* **响应**

    * 最外层是一对中括号，表示返回的是一个列表；查询结果可能不止一个产品。

    * 列表里每个元素包含 12 个字段：`id`、`name`、`class_name`、`category`、`insurer`、`image_url`、`description`、`min`、`max`、`target_group`、`highlights`、`status`。

    * 字段与 `products` 表完全一致，只去掉 `created_at` 与 `updated_at` 两个时间字段，因为前端不需要。

    * 实现思路由此完整：拿 `category`（如果有）加上 `status = 'active'` 去表里查，查出的记录去掉两个时间字段，组装成列表返回。

### ProductResponse

* **定义方式**

    * 继承 Pydantic 的 `BaseModel`，一个字段一行；字段在文档里是数字就定 `int`，是字符串就定 `str`，以此类推。

    * 类型上有三处要点：`id` 是 `int`，其余大多是字符串；`min` 与 `max` 用 `Decimal` 而不是 `float`；`highlights` 在数据库里是数组，Python 侧写 `List[str]`。

    * 继承只留 `BaseModel` 一个，多余基类删掉；后续统一改为继承 `BaseSchema`。

* **单参数不建类**

    * 请求参数只有一个 `category`，没有必要单独定义类；直接在 router 方法参数列表接收，FastAPI 会把它当查询参数处理。

    * 多写一个只有单个字段的类，除了增加一层跳转没有任何收益。

* > **提示**：`Decimal` 与 `float` 的区别不在「能不能算」，而在「算得准不准」；凡是不能容忍舍入误差的数值，优先用 `Decimal`。

---

## 数据模型与仓储层

### 统一基类与混入类

* **Base**

    * > **定义**：所有数据库表模型类的统一基类，继承 SQLAlchemy 的 `DeclarativeBase`。

    * 作用两条：标记 SQLAlchemy 的表映射类；把统一的函数写在里面，例如 `__repr__` 返回 `<类名 {self.__dict__}>`，让打印对象有可读输出而不是内存地址。

    * 必须放 `common`：`chat`、`user`、`product` 等业务模块都要继承它，放进任何一个业务模块都会让其他模块用不了；公共物一律沉到 `common`，业务模块之间横向互不依赖。

    * `__repr__` 是 Python 的「对象自述」钩子，`print(obj)`、调试器、日志都会调它；放在基类里一次，所有表映射类自动拥有可读的打印结果。

* **混入类（Mixin）**

    * > **定义**：只提供若干公共字段、供其他映射类按需继承的类；继承它即获得对应字段，用于代码复用。

    * `CreatedAtMixin`：`created_at: Mapped[datetime] = mapped_column(DateTime, default=func.now())`；新增数据时该字段不用管，自动填当前时间。

    * `UpdatedAtMixin`：`updated_at` 同样默认 `func.now()`，并加 `onupdate=func.now()`；修改表里任意字段时，更新时间自动刷新。

    * 不能把时间字段并入 `Base`：不同表需要的字段不同，有的只有 `created_at`，有的两者都有，有的都没有；并入基类会强迫所有表都带时间列。

    * 表需要哪个字段就继承哪个混入类；两个都需要就同时继承两个。

### Product 映射类

* **类声明**

    * `class Product(Base, CreatedAtMixin, UpdatedAtMixin)`：`Base` 提供统一标记与通用函数，两个混入类提供创建时间与更新时间。

    * `__tablename__ = "products"`，表名必须与数据库完全一致，写错就映射不到。

    * 其余字段与表一一对应，时间字段不手写，靠继承获得。

* **类型对应**

    | 数据库里的列类型 | 类里的写法 | 说明 |
    | :--- | :--- | :--- |
    | 整型主键 | `Mapped[int]` + `Integer, primary_key=True` | 主键信息写在 `mapped_column` 里 |
    | 字符串 | `Mapped[str]` + `String(长度)` | 长度照表结构抄，不用加 `Optional` |
    | `NUMERIC` | `Mapped[Decimal]` + `Numeric` | 保额上下限用 `Decimal`，不用 `float` |
    | `ARRAY` | `Mapped[list]` + `ARRAY` | 保障人群或亮点是一组值，Python 侧就是 `list` |
    | 时间（`TIMESTAMP`） | 不写 | 继承混入类即可 |

    * `status` 是字符串并带默认值；`mapped_column` 里还要写字段是否主键、数据类型、长度，这套 API 很多。

* > **易错点**：`NUMERIC` 一定要映射成 `Decimal` 而不是 `float`；浮点数存金额会在末位飘，`Decimal` 才是定点十进制，与库里的 `NUMERIC` 语义一致。

* **写法来源**

    * 表映射类属于「固定代码」，靠「复制一份示例 + 逐字段替换」得到，不靠记忆；示例可以来自 SQLAlchemy 官网或以前项目里的代码。

    * 开发顺序不是一上来就照抄官方示例写 `Product`，而是先做统一基类与混入类，再写具体映射类。

### 先写 SQL 再转 ORM

* **流程**

    * 写任何增删改查都分两步：先回数据库把 SQL 语句写出来，再把 SQL 翻译成 SQLAlchemy 代码。

    * repository 只做数据库操作，先分析接口需要什么样的查询，再落笔。

* **目标 SQL**

    ```sql
    SELECT *
    FROM products
    WHERE status = 'active'
      AND category = 'life'
    ORDER BY id ASC;
    ```

    * 两个条件：`status` 必须有（只查有效产品），`category` 可选（不传就查全部有效产品）。

    * 把 `life` 换成 `medical`、`critical_illness`、`accident`，分别查到对应分类；`ORDER BY id ASC` 按 id 升序。

* **ORM 三条对应**

    * 表对应类：SQL 的 `select * from products` 在 ORM 里是 `select(Product)`，填表所对应的映射类。

    * 列对应属性：`Product.status == "active"`、`Product.category == category`；类名点属性名，Python 判等用两个等号。

    * 排序对应方法：`Product.id.asc()` 升序，`Product.id.desc()` 降序。

    * 打印 `stmt` 能看到生成的 SQL：`SELECT products.id, products.name, ... FROM products WHERE products.status = :status_1 AND products.category = :category_1`，说明此时是固定 SQL。

### 动态条件

* **问题**

    * `where` 里写死两个条件，两个都必须成立；但 `category` 可能不传，不传时还加 `category == None` 就查错了。

    * 需要的是动态条件：有值才加，没有值不加。

* **三步套路**

    * 建条件列表：`conditions = [Product.status == "active"]`；列表里装的是「条件」，不是字符串或数字。

    * 按传参追加：`if category: conditions.append(Product.category == category)`。

    * 解构铺开：`select(Product).where(*conditions)`，星号把列表里每个条件逐个放进 `where`。

* **真假判断**

    * 不用 `is None` / `is not None`；Python 里非空即真，有值为 `True`，`None` 或空字符串为 `False`。

    * 直接 `if category:` 就代表「有值」，有值才追加分类条件；传 `life` 生成两个条件，传 `None` 只剩一个条件。

* **结论**：动态条件只有三步——先建条件列表，按传参用 `if` 追加，最后在 `where` 里用 `*` 解构铺开。

### repository 类与查询执行

* **类结构**

    * 类名按「业务名 + 层职责」：`ProductRepository`，即产品的持久化操作、CRUD 操作。

    * 方法写在类里统一管理，对外用面向对象的方式调用；将来增删改查方法会越来越多，不要一上来就摊开写。

    * `__init__(self, session: AsyncSession)` 把 session 存成 `self.session`；不是每个方法都定义一个 session 参数。

    * session 由上层递进来，不在 repository 里手动调 `get_session`；`get_session` 是给 FastAPI 依赖注入用的。

    * `async def query_by_category(self, category: str | None = None) -> list[Product]`，见名知义，默认 `None` 让调用方不传也能跑。

* **查询实现**

    ```python
    conditions = [Product.status == "active"]

    if category:
        conditions.append(Product.category == category)

    stmt = (
        select(Product)
        .where(*conditions)
        .order_by(Product.id.asc())
    )

    products = await self.session.scalars(stmt)
    return products.all()
    ```

    * `scalars` 可以直接接收语句，省掉先 `execute` 再 `scalars` 一步；调完直接 `.all()` 转成列表。

    * PostgreSQL 驱动是异步的，查询要 `await`，函数必须是 `async def`。

    * 查多条用 `scalars`，作用是把结果集收成一列实体，再 `.all()` 转成列表；返回值里装的是 `Product`。

* **notebook 验证**

    * 测试在 notebook 里做，因为创建 repository 对象要先有 session。

    * 不在 FastAPI 运行环境里，就手动用 `async with AsyncSessionFactory() as session:` 取 session，不要调 `get_session`。

    * 传 `"life"` 查出三条寿险；传 `None` 查出库里全部 12 条产品，id 从 1 到 12。

    * 改完代码记得重启 notebook 内核，从头执行一遍。

---

## 服务层与路由层

### service 与事务边界

* **职责**

    * service 作为业务层，负责组织业务流程，有必要时调 repository 完成数据库增删改查；有些业务根本不碰库，直接在业务层做完。

    * 当前业务只是查询产品列表，没有其他逻辑，所以这一层非常薄。

* **类结构**

    * 类名「业务名 + 层」：`ProductService`；以后用户管理就是 `UserService`、`UserRepository`，一路套下去。

    * `__init__(self, session: AsyncSession)`：`self.repository = ProductRepository(session)`，同时保存 `self.session`。

    * 注入写在构造函数而不是方法里：方法会越来越多，逐方法注入重复；构造函数只接收 session，repository 自己就能创建。

    * 保留 session 的原因：复杂业务中一个 service 可能要调用多次写库，需要在 service 层统一管理事务，把多次写操作归入同一事务；开事务要用 session。

* **方法**

    * `async def list_products(self, category: str | None = None) -> list[Product]`，方法体一句：`return await self.repository.query_by_category(category)`。

    * 返回值仍是列表，里面装 `Product`，类型注释同样要导包。

* **结论**：repository 只管单次怎么读写库；「这一串写操作算不算一件事」由 service 决定，事务的边界在 service，不在 repository。

### router 声明

* **为什么不能直接 `app.get`**

    * router 文件里看不到 `app`，不能像启动模块那样直接 `app.get`；要先单独定义 `APIRouter`，将来在 `main` 里挂载。

* **APIRouter 参数**

    * `prefix="/api/v1/products"`：统一的路径前缀，本 router 里所有产品接口共用；个别接口路径不同，在装饰器里再补。

    * `tags=["产品管理相关接口"]`：对当前这组接口的说明，会出现在 FastAPI 自动生成的接口文档里。

* **装饰器**

    * `@router.get("", response_model=list[ProductResponse])`：第一个是位置参数，即接口路径；本次路径与统一前缀相同，值写 `""`，但位置参数不能省。

    * `response_model` 建议写上，声明响应格式是列表，里面装 `ProductResponse`。

    * `get`、`post`、`delete` 都可以，本接口是查询所以用 `get`。

* **函数签名与实现**

    * `category: str | None = None`：字符串，也可以是 `None`，默认 `None`，前端可传可不传。

    * `session: AsyncSession = Depends(get_session)`：需要什么就在参数列表里声明什么，由 FastAPI 调 `get_session` 交出来。

    * 实现两步：`service = ProductService(session)`；`return await service.list_products(category)`。

    * `get_session` 靠 `yield` 交出 session，请求进来时打开、请求结束时关闭，生命周期由 FastAPI 管，所以只能出现在 `Depends(...)` 里。

* **声明完成即四要素齐备**

    * 请求方式、请求路径、请求参数、响应格式全部声明完毕，一个 FastAPI 接口的声明就完成了。

### 挂载与自动转换

* **挂载**

    * `from ... import router as product_router`，再 `app.include_router(product_router)`。

    * 导入时起别名，因为将来 router 会有很多个，不起别名容易冲突。

    * 启动后刷新 `/docs`，除 `default` 外多出「产品管理相关接口」分组，可以直接在里面完成查询；前端刷新后产品全部查到，按四个分类过滤都正常。

* **model 到 schema 的自动转换**

    * router 返回的是 `list[Product]`，声明要的是 `list[ProductResponse]`，类型并不一致。

    * FastAPI 执行时看到目标模型，会自动把 `Product` 转成 `ProductResponse`：按目标模型要的字段去源对象里找，有就转过来，没有就丢掉。

    * `ProductResponse` 比 `Product` 少 `created_at` 与 `updated_at`，转换时正好把这两个丢掉，所以不用手写转换。

    * > **提示**：FastAPI 自动转换的前提是「它知道你声明了什么」；响应模型写进注解，它返回前按模型重新校验一遍，顺手完成字段搬运。代码不报错不代表类型对，只是有人替你兜住了。

### 全链路方向

* **开发顺序回顾**

    * 先按接口文档确定请求方式、路径、参数、响应，然后做 schema（响应字段就是数据库表字段）。

    * 再做 models：统一基类、两个混入类、`Product` 映射类。

    * 再写 repository：先组装动态条件，再执行查询。

    * 再写 service：调 repository 完成列表查询。

    * 再写 router：按文档声明路径前缀与接口，调 service 返回结果。

    * 最后在 `main` 注册 router，到前端测试。

* **层的职责与 session 来源**

    | 层 | 只负责这一件事 | session 从哪来 |
    | :--- | :--- | :--- |
    | router | 收请求、校验参数、封装响应 | `Depends(get_session)` |
    | service | 组织流程、管事务边界 | 构造函数接收 |
    | repository | 增删改查 | 构造函数接收 |
    | models | 表结构映射 | 不涉及 |

    * 请求从上往下走，一层只认下一层；session 从上往下递：router 用 `Depends(get_session)` 拿到，作为构造参数给 service，service 再给 repository，越往下越不关心它从哪来。

    * 越往下越不知道「业务」，越往上越不碰 SQL：repository 里只有 `select`、`where`、`order_by`，压根不知道什么叫「上架的产品列表」；router 里只有参数与响应模型，压根不知道数据是 join 出来的还是缓存取的。

---

## 依赖注入与 Pydantic 对象转换

### 依赖注入链

* **get_product_service**

    * > **定义**：把「创建 service」这段代码抽成的依赖函数，需要 service 的地方依赖它即可，不必每个接口手写一遍创建逻辑。

    * `async def get_product_service(session: AsyncSession = Depends(get_session)): service = ProductService(session); yield service`。

    * 函数自己不取 session，而是声明「我需要 session」，由 FastAPI 通过 `Depends(get_session)` 交进来；这就是依赖注入的链。

    * router 改为 `service: ProductService = Depends(get_product_service)`，不再自己创建 service。

* **链的执行过程**

    * 前端请求进入 `list_products`，FastAPI 解析签名：这个接口需要一个 service，来源于 `get_product_service`。

    * 调 `get_product_service`，它又声明需要 session，来源于 `get_session`；FastAPI 继续解析并调用 `get_session`。

    * `get_session` 创建 session 后 `yield` 挂起，把 session 交给调用者 FastAPI，FastAPI 把它赋给 `get_product_service` 的 session 参数。

    * `get_product_service` 恢复执行，把 session 传给 service、创建 service，再 `yield` 挂起交出 service，FastAPI 赋给 `list_products` 的 service 参数。

    * `list_products` 拿到 service，执行查询逻辑并返回。

    * 回到 `get_product_service` 的挂起点：没有后续，函数结束。

    * 回到 `get_session` 的挂起点：上下文结束，session 自动关闭。

* **yield 的含义**

    * `yield` 是「借用与归还」的分界线：之前是准备资源，之后是回收资源。

    * session 的关闭没人手动调 `close()`，是上下文结束时自动发生的。

    * 用 `return` 交出去则函数当场结束，回头没有任何地方能把 session 关掉，所以这里必须用 `yield`。

* 整条链的入口是一次前端请求，出口是 session 被关闭；从上往下读是「一层层解析依赖、把资源造出来」，从下往上读是「每层 yield 的值原路返回、资源被回收」。

### model 与 schema 两套类

* **普通类与 BaseModel 类**

    * > **定义**：没有继承 `BaseModel` 的类统称普通类；继承了 `pydantic.BaseModel` 的类统称 BaseModel 类。

    * 分工：普通类是数据库 model，BaseModel 类是接口 schema。

* **类型不匹配的隐患**

    * service 查出的 products 是 `list[Product]`（model），前端要的 `items` 是 `list[ProductResponse]`（schema）；把 model 集合直接赋给 schema 集合，类型上是不对的。

    * 程序运行不报错，是因为 FastAPI 发现返回类型与声明类型不符，自动帮你把 `Product` 转成 `ProductResponse`。

    * 换成非 FastAPI 场景就没有人兜底，必须手动转换；而手动转换时普通类不能直接转成 BaseModel 类。

### 普通类转 BaseModel

* **第一步：声明从属性读取**

    * 给 BaseModel 类加 `model_config = ConfigDict(from_attributes=True)`，允许 Pydantic 从其他类（普通类）对象的属性里读值。

    * 前提：两种类型的属性名必须一致；它读的是同名属性，名字对不上就搬不过去。

* **第二步：调用 model_validate**

    * `model_validate` 是所有 BaseModel 类都有的静态方法，传普通类对象进去，返回一个 BaseModel 对象。

    * 两个类属性名一致（如 `id`、`name`、`class_name`、`category`），加上配置后用 `model_validate` 就能完成转换。

* **报错演进（验证过程）**

    * 只写 `model_validate`、不加配置：报数据校验错误，输入必须是字典或该 BaseModel 的实例，普通类对象不符合要求。

    * 加了 `from_attributes`、普通类只写 `__init__`：报缺少成员、缺少对外获取成员的魔法函数，类没法把字段值交出来。

    * 普通类改用 `@dataclass` 装饰：自动生成 `__init__` 及各类魔法函数，再转换成功，打印类型是 `UserResponse`，属性值全部转过去。

* **dataclass**

    * `@dataclass` 让定义类可以像 BaseModel 一样写「字段名冒号类型」，不用手写 `__init__` 和取属性的模板函数。

    * 它定义的类仍然是普通类——判断标准只有一条：没继承 `BaseModel` 就是普通类；dataclass 只是简化写法。

* **结论**：普通类对象转 BaseModel 类对象必须做两件事——给 BaseModel 类加 `model_config = ConfigDict(from_attributes=True)`，再用 `model_validate` 转换；两个类的属性名还必须一致。

### BaseSchema 统一基类

* **问题**

    * 项目里所有 schema 都是 BaseModel，`ChatResult` 也是，将来还会更多；每个类手动加一遍 `from_attributes` 配置太麻烦。

* **做法**

    * 在 `common/schema` 定义 `class BaseSchema(BaseModel): model_config = ConfigDict(from_attributes=True)`。

    * 所有 schema 不再直接继承 `BaseModel`，改继承 `BaseSchema`：间接继承 BaseModel，保留全部功能，同时继承转换配置。

    * `ProductResponse`、`ChatResult` 以及后续所有 schema 一律继承 `BaseSchema`。

* **效果**

    * 所有 BaseModel 类不仅能从字典取属性，还能基于类的字段取属性；只要两个类属性一样，就能把类 A 的属性转给类 B，实现自动转换。

---

## 会话管理接口

### 会话与元数据

* **登录前提**

    * AI 保险顾问必须先登录才能使用；登录是模拟的，点了登录就成功，不需要实现。

    * 登录后前端把用户信息发到服务端，请求头里有 `x-user-id`，即用户 id，默认写死成 `1`；后端拿到请求头就知道当前登录用户是谁。

* **thread_id**

    * > **定义**：一个会话的唯一标识；每个用户可以同时拥有多个会话，每个会话用一个唯一 id 区分。

    * 以前做 AI 伴侣时叫 `session_id`，名字不同道理一样；后续跟 AI 对话、学 LangGraph 时都会用到 `thread_id`。

* **会话管理的两部分**

    | 部分 | 内容 | 本期安排 |
    | :--- | :--- | :--- |
    | 会话元数据 | 会话标题、所属用户、创建与更新时间，即左侧会话列表 | 本次实现 |
    | 会话历史 | 会话里一条条消息，用户一条、AI 一条 | 先不做 |

    * 本期实现会话列表的四个功能：创建会话、查询会话、重命名会话、删除会话。

### 创建会话接口契约

* **请求**

    * 请求方法 `POST`，请求路径 `/api/v1/chat/threads`，必须登录。

    * 请求体字段只有 `title`：新建会话时界面上只有一个名字，其余信息（会话属于谁）从请求头的用户 id 得知。

    * > **易错点**：`title` 不是拼在问号后面的查询参数，而是放在请求体里以 JSON 传递——请求体在浏览器 Network 面板的 payload 里。

* **响应**

    | 字段 | 含义 |
    | :--- | :--- |
    | `thread_id` | 新建会话的唯一标识，后续与 AI 对话要用到 |
    | `title` | 把创建时提交的标题原样返回 |
    | `created_at` | 创建时间 |
    | `updated_at` | 更新时间 |

    * 响应最终也是 JSON：有 id、有标题、有两个时间。

* **接收方式**

    * 前端发 JSON，后端就要用一个对象来接收 JSON，对应的是 schema 对象，而不是方法参数列表里的单个查询参数。

    * 接口实现的固定顺序：先看文档分析请求与响应，再动手写代码。

