# 黑马程序员Python+FastAPI+LangGraph多Agent搭建企业级AI应用，手把手用最新LangChain+Milvus搭建真实AI项目·FastAPI 企业级工程：分层架构与会话业务（第 2 册 / 共 6 册）

> **所属课程**：黑马程序员Python+FastAPI+LangGraph多Agent搭建企业级AI应用，手把手用最新LangChain+Milvus搭建真实AI项目  
> **本册内容**：FastAPI 企业级工程：分层架构与会话业务  
> **覆盖范围**：P32-P36 ~ P61-P63（共 32 讲 / 8 章 / 389 分钟音频）  
> **分册说明**：全书按内容分 6 册；本册为第 2 册  
> **整编说明**：正文逐字保留各块模块长文，仅补导读、目录与章间过渡。

---

## 导读与全景目录

1. 安心宝：从项目全貌到能跑起来的环境（P32-P36）
2. 先把地基打牢：分包、配置管理与统一日志（P37-P40）
3. 从启动模块到第一个接口：FastAPI 项目的骨架与业务规范（P41-P45）
4. 从表映射到路由：一条查询接口的四层落地（P46-P49）
5. 依赖注入链、Pydantic 转换与会话管理（P50-P52）
6. 会话创建：表结构、映射类与三层业务代码（P53-P56）
7. 会话接口：三个动作，一条从 router 到数据库的链路（P57-P60）
8. 会话重命名：从 ORM 自动更新到统一异常处理（P61-P63）

---

## 安心宝：从项目全貌到能跑起来的环境
> 对应块：BLK09 | 覆盖分集：P32-P36 | 块标题（分集名）：《企业级项目导览与开发环境准备》

### 先跟 Notebook 打个照面

我们之前写代码，习惯都是新建一个 `.py` 文件，在里边一行一行往下写。做项目开发，确实就该这么干。可换个场景：我不是做项目，我只是想学一个知识点，只想做个测试，那还在 `.py` 里写，行是行，但确实有点麻烦、有点多余——为了看一眼变量的值，得把整个文件跑一遍，跑完窗口一关，什么也不剩。

所以这里先介绍一个专门用来做开发测试的东西：Notebook。

随便找一个项目，新建一个 Jupyter Notebook 就行。我先建一个叫 `notebook` 的文件夹，在里面 new 一个 Jupyter Notebook，弹出一个窗口让我填名字，就叫 `hello`。创建好之后你会发现，这个文件和平时见的不一样——它的后缀是 `.ipynb`。

这个名字可以拆开看：`I` 是 interactive，交互；`P` 和 `Y` 就是 Python；`NB` 是 notebook。有人开玩笑说 NB 是很牛逼的意思——玩笑归玩笑，它真正的意思是 notebook。合起来，它是一个**可交互的 Python 学习笔记**。

那"可交互"到底体现在哪儿？打开这个文件，里面是一个一个的方框，这个方框叫 `cell`，中文叫单元格——拼写跟"细胞"是同一个词，但在这里它是单元格的意思。单元格里可以直接写代码：

```python
message = "hello world"
print(message)
```

写完这两行，代码旁边会冒出一个小三角形，点一下，它就开始跑。第一次运行需要下载依赖，中间还失败过一次——失败也不要紧，再点一次就成了。成功之后它会启动一个 Jupyter 服务，然后结果直接打印在单元格下面。

这件事有三个地方值得留意。

第一，结果就在代码正下方，而且**可保留**。把文件关掉再重新打开，代码还在，运行结果也在。将来你在这写了个笔记、试了段代码，过一阵想复习，点开就能看到当时跑出来的样子，不用重跑一遍。

第二，变量是**可观察**的。打开旁边的 Jupyter Variables 面板，刚才定义的 `message` 就列在那儿，值是多少一眼就能看到，还能随时盯着它变。传统 `.py` 文件做不到这一点：你写个变量，运行完，下次还想再观察它？观察不了了。

第三，单元格之间是**联动**的。鼠标放到单元格下方，会看到 Code 和 Markdown 两个选项，点 Code 再建一个代码单元格：

```python
message2 = message.upper()
print(message2)
```

`upper()` 就是转大写。这个单元格里根本没定义 `message`，但运行之后打出的是 `HELLO WORLD`——说明上一个单元格的变量顺延下来了。运行除了点三角形，还可以在选中单元格时按 `Shift + Enter`，这个快捷键要记住。

Markdown 单元格则是真正用来写笔记的：写标题、写层级，写完再看文件结构，是带层级的一棵树，复习的时候点一下就跳过去。

所以 Notebook 是什么？是把笔记和代码混在一起的东西，而且代码可以直接运行、结果可以直接看。这就像在纸质书上写笔记——写完是静态的；写在这里是动态的：有笔记的内容，有代码，代码还能执行，还能看结果。它里面的单元格就两大类：`code cell` 写代码，`markdown cell` 写笔记。

后面上课要演示什么东西，用 Notebook 就方便很多，变量怎么变随时看得见。

### 安心宝到底在卖什么

Notebook 说完了，接下来正式进项目。这一天要做五件事：先了解项目的背景和系统架构，对项目有个整体认识；学会用 Docker Compose 启动项目依赖的基础设施；理解项目结构和 Pydantic Settings 的配置管理；用 FastAPI 的生命周期管理数据库连接池（这个之前学过）；最后落一个小功能——把保险查询的接口做出来，让前后端能联调。目标很明确：把环境准备好，再跑通一个小功能。

项目叫**安心宝**，是一个保险优选平台，说白了就是一个保险商城。

大家以前买过保险吗？大多数应该没有，因为还没上班。等上了班，公司会给你买保险，但那种是普通的社保；这里说的是**商业保险**——保额更高，保的范围也更广。所以理论上，大多数人从出生就该开始买保险了。这不是空话，我家小孩一出生，我就给他买了：意外险、疾病险、寿险，全买了。小孩子买寿险其实也不要紧，买一段时间就行，不需要一直买。这些保险一方面是保障，另一方面也是理财，到时候是可以返还的。任何一个家庭，保险都是必备的东西。

要说清楚的是，我们这个商城**本身不是保险公司**。保险公司就那么几家，数都数得过来，平安、人寿，很多还是国企，不是随便什么人都能开的。我们做的事是把各大保险公司的产品集中到一个网站、一个服务里，让用户一站式地挑，组合自己的保险方案。普通商城卖实物，我们不卖实物，卖的是虚拟产品——保险。而且我们只做面向人的保险，车险不在里面。

面向人的保险主要是四大类。

| 险种 | 解决什么问题 | 赔付方式 |
| :--- | :--- | :--- |
| 百万医疗险 | 社保报完之后剩下的那部分医疗费 | 报销型，通常有 1 万左右免赔额，超出部分最高可报几百万 |
| 重疾险 | 癌症这类治疗费用极高的重症 | 确诊后直接赔一笔现金，50 万、200 万都可能 |
| 寿险 | 家庭经济支柱突然身故 | 赔一笔钱给家人，保额可达几百万，保费也最贵 |
| 意外险 | 突发意外造成的伤残或身故 | 覆盖"残而不死"带来的长期经济负担 |

**百万医疗险**是报销型的主力，解决的是平常看病的那笔医疗费。看病住院，社保肯定报一部分：买药如果是甲类，可能报 70% 到 80%；乙类丙类报的比例又不一样；要是用了国外的药，压根不进社保，那就得自费。社保只保一部分，剩下不保的怎么办？这时候百万医疗险接上——它一般有个免赔额，一万左右，超过一万的部分才报销，最高能报几百万。所以这个险用得最频繁。

**重疾险**看名字就知道，是专门针对大病、重症的。合同里会约定一批病种，比如各种癌症，那些治疗费用非常昂贵、动辄每个月要花好几万的病。这种保险是直接赔现金，赔你 50 万或者 200 万都可以。这笔钱不只可以拿去治病，还房贷、干别的都行。

**寿险**针对的是"人突然没了"。人走了，赔一笔钱给家人，防止家庭的经济支柱突然倒下。想想将来结婚生子：一家三口，一方在家看孩子，一方在外上班，家里还有房贷车贷——万一在外上班的那个人突然出了事，整个家就直接废了。所以寿险的赔付额非常高，往往能到好几百万；它也是四大险种里最贵的，一年可能要交大几千甚至上万。贵，但换来的是给家人的后续保障。

**意外险**针对突发的意外事故，比如出门坐车不小心出了车祸。它对应的是那句很关键的话：残而不死带来的长期经济负担。

讲这些不是在闲聊。你将来简历上写了这样一个项目，面试官很有可能就问这些东西——连基本常识都不知道，你怎么跟人聊？更重要的是，它跟业务开发直接相关：做业务就是要写这些产品的增删改查，而产品本身有种类之分，四大险种就是"分类"这个概念的来源。

险种一多，问题就来了：不同品种解决的问题不一样，买的时候注意事项也不一样。一个普通人，对保险没什么认知，到底该买哪些、有哪些产品、条款是什么意思、理赔该怎么走，基本都不知道。往前推个十几年，一提保险，大家第一反应是"骗人的""骗我花钱的"；保险销售也成了不太好做的职业，天天在线下推，见着亲戚朋友就说"买保险吧"，弄得大家都躲。这其实是误解，加上不了解——对保险产品本身不了解。

所以才需要一个专业的保险顾问：给你科普保险知识，带你看懂每一个产品的作用，再帮你筛，帮你组一套完整的方案。保险从来不是单一产品，针对一个人、一个家庭，是可以组合出不同方案的，组合起来才能实现更全面的保障。80% 的人不了解这件事，所以这是刚需。

这个顾问，我们就用 AI 来实现：协助用户选购保险，做产品答疑、条款解释、理赔指引。

### 前后端分离与三层架构

这个项目采用前后端分离的模式：前端和后端分开，各自独立开发。

前端叫 `frontend`，提供保险商城的页面——浏览产品、登录注册、进入智能客服，是用户直接看到的交互界面。但前端是静态页面，它自己没有数据，也没法实现具体功能，功能得靠后端。

后端叫 `agent-service`。`agent` 这个词传统翻译是"代理"，在人工智能领域叫**智能体**——我们要开发的是一个智能客服，它就是一个智能体，所以 `agent-service` 就是智能体服务。

后端内部的业务分两大部分。

一部分是传统保险业务：用户的登录认证、产品查询、保费试算、保险方案的生成、保单的增删改查、理赔等等。

另一部分是保险顾问自己的业务，也就是那个 AI 智能顾问：意图识别和路由、保险组合推荐、条款问答、理赔咨询。

整个系统从上到下是三层：

```text
[浏览器]
    |
    v
[用户体验层 / 用户交互层]   frontend：React + Ant Design
    |  浏览产品、登录注册、进入智能客服
    |  页面需要数据时发起请求
    v
[后端服务层]   agent-service：Python + FastAPI
    |  传统保险业务：登录认证、产品查询、保费试算、方案生成、保单管理、理赔
    |  智能体业务：意图识别与路由、组合推荐、条款问答、理赔咨询
    v
[基础设施层]   PostgreSQL（关系型数据库） + Milvus（向量数据库） + 各类模型
```

读这张图就一条主线：用户用浏览器访问前端服务器，拿到的是完整的页面，但页面里没有数据；页面要数据，就去调后端服务层；后端再把活派给下面的数据库和模型。

前端用 React 加 Ant Design 这套技术栈。前端技术栈你们现在应该还没学过，没关系，我已经准备好了，直接用就行，不需要开发。后端没完成，需要从零开始写，我会带着你们用 Python 加 FastAPI 一句一句写出来，里面既有业务部分也有 agent 部分。

技术栈整体是这样：

| 位置 | 选型 |
| :--- | :--- |
| 后端框架 | FastAPI |
| 智能体编排 | LangGraph + LangChain |
| 大模型 | DeepSeek |
| 向量数据库 | Milvus |
| 关系型数据库 | PostgreSQL |
| 前端 | React + Ant Design |
| 包管理 | uv |
| 部署 | Docker Compose |

有一点必须说明：现在这个架构里，传统保险业务和保险顾问业务是**合在一起**的。但在真实的企业开发中，这两块是分离的——传统业务通常不用 Python 写，用 Java。全球 90% 以上的业务后台都是 Java 写的：你访问的网站、医院在用的系统、银行取款机背后的服务，基本都是。理论上第一部分应该用 Java 来写，可咱们现在还没学 Java，所以暂时和保险顾问的业务一起用 Python 写。等这个阶段结束、进入下一个阶段学了 Java，这两部分就会分开。现在先知道真实的样子，我们先合着做。

### 把基础设施交给 Docker Compose

前端请求后端，后端要访问数据库。项目需要两个数据库：PostgreSQL 和 Milvus，这两个都得安装部署。

**PostgreSQL** 就是个数据库，类似 MySQL。你们之前学的是 MySQL，但在 Python 领域、智能体领域，大家通常用的是 PostgreSQL，所以接下来要从 MySQL 切到 PostgreSQL。常见数据库当然不止 MySQL：SQLite、SQL Server、MariaDB、Oracle 都是——Oracle 就是鼎鼎大名的甲骨文，收费的数据库，但活得很好，在全球用得都非常广，尤其银行这类地方。

不用因为"只学过 MySQL"而担心。学数据库，重点学的是 SQL 语句，而 SQL 语句在各大数据库之间是通用的，有统一的 SQL 标准；各家当然有自己的方言，但只要写的是标准 SQL，就能通用。更何况用 SQLAlchemy 的时候，你还需要自己写 SQL 吗？不需要，都是面向对象地调方法，SQL 由它自动生成。底层换成哪个数据库，跟你写代码还有关系吗？没有。所以切到 PostgreSQL，一点问题都没有。

**Milvus** 是向量数据库，这是个新单词，现在先认识有这么个东西就行。麻烦的地方在于它自己还依赖一堆组件：依赖 etcd 做集群管理和元数据存储，依赖 MinIO 做对象存储，依赖 Attu 做图形化界面的访问。这些是不是都得装？你会装吗？不会吧。所以我把 Docker Compose 准备好了——把对应的文件给你，你就能部署，前面学过的东西正好用上。

部署之前，先记住这套东西的端口和连接信息：

| 组件 | 端口 | 说明 |
| :--- | :--- | :--- |
| PostgreSQL | 5432 | 关系型数据库，库名 `insurance`，用户名 `insurance`，密码 `insurance123` |
| Milvus | 19530 | 向量数据库本体 |
| etcd | 2379 | Milvus 的集群管理与元数据存储 |
| MinIO | 9000 / 9001 | 对象存储 |
| Attu | 3000 | Milvus 的图形化管理界面 |

后几个端口不用背，将来用到的时候查表就行。但 PostgreSQL 那一行要记牢：`5432` 不是 MySQL 的 `3306`，写错了就连不上。

部署文件在课前资料的"课件代码"里，`04` 下面的 `hm-insurance` 目录（HM 就是黑马），里面只有两部分：`deploy` 是部署脚本，`frontend` 是前端源码。`deploy` 里最关键的是一份 `docker-compose.yml`，有了它就能一键部署；此外还有一个 `init` 目录，里面装的是 SQL 脚本，用来自动初始化项目需要的表——这些表都给你准备好了，不用自己建；`milvus` 那份配置现在先不用管，放在那儿就行。

部署就两步：

```bash
# 1. 把 deploy 目录整体上传到服务器的 /root 下，然后进入该目录
cd /root/deploy

# 2. 一键拉起全部容器
docker compose up -d
```

前提是先把防火墙关掉——这里再强调一遍，防火墙必须关；已经关过的就不用管了。`up -d` 跑起来之后，它会自己建网络、建数据卷、建容器，需要的镜像也都提前给好了，不用再下载，耐心等就行。

等待的时候可以留意一下启动顺序，它不是一股脑全起来的：

```text
第一梯队：etcd（2379） + MinIO（9000 / 9001）
                |
                v
第二梯队：Milvus（19530）
                |
                v
第三梯队：Attu（3000）
```

这个顺序是有道理的：Milvus 要靠 etcd 和 MinIO 才能工作，所以这两个先起来；Attu 是管 Milvus 的界面，自然要等 Milvus 好了再起。等所有状态都变成 `healthy` 或者 `started`，基础环境就部署好了，可以用 `docker compose ps` 看进程。

> **提示**：`docker compose ps` 列出的是当前这组容器的状态。看到 etcd、MinIO、Milvus、Attu、PostgreSQL 都在，状态正常，就说明整套基础设施齐了。

光起来还不够，得连一下才算验证通过。

先验 PostgreSQL。用 PyCharm 自带的数据库工具连就行，或者用任意一个客户端（我这儿用的是专门的客户端，效果一样）。要填的东西就是表里那几项：主机填服务器 IP（我这里是 `192.168.150.101`，你换成自己的），端口 `5432`，用户名 `insurance`，密码 `insurance123`，数据库 `insurance`。点开之后，所有表都在——这些表是自动创建好的，不用你管。

有人会问：到了企业里，我要用的表谁给我建？其实你进公司，公司的表早就建好了，业务都开发过很多轮了，不可能让你正好碰上一个从零开始的项目。退一步说，真让你碰上从零的项目，也轮不到一个刚拿一万月薪的人去建表——那是高级开发甚至架构师干的活。所以这些表我都给你准备好了，直接用。

再验 Milvus 的界面。在浏览器里访问 `192.168.150.101:3000`。注意这个 `3000` 不是 Milvus 的地址，而是它图形化界面 Attu 的地址；Milvus 自己是 `19530`，别搞混。第一次访问会让你创建一个管理员账号，要求至少八位，还要包含大小写字母、数字和符号，我图省事用了 `admin_123`。创建成功就自动登录进管理界面了。Milvus 具体怎么用现在还没学，能看到状态、能连上就够了。

两个都能连上，就说明部署成功，基础设施这一步完成。

### 把前端跑起来

基础设施好了，接着部署前端。前端源码也在 `hm-insurance` 里，就是那个 `frontend` 目录。

这一步要严格按顺序来。先在电脑上找一个专门放代码的地方，最好像我一样有个 `code` 目录，路径里**不要有中文、不要有特殊字符**，就是一个规规矩矩的目录。然后把整个 `hm-insurance` 目录拷过去，是整个拷，不是只拷 `frontend`。

为什么要整个拷？因为这里现在放的是前端代码，等一会儿后端也要建在里面。前后端放在同一个目录下，好处很实在：你可以在 `hm-insurance` 这个目录里直接打开一个 AI 工具，比如 Claude Code，让它以这个目录为根。将来你跟它交互——比如觉得前端某段代码写得不好想改——它既能看见前端，也能看见后端，前后端联调的代码它可以一起帮你改。

拷完先装依赖。前端项目也是一个独立项目，有自己的一套依赖，用 `npm install` 安装。你们的电脑上肯定有 npm——当初装 Claude 就是用 node 装的。在这个目录下打开命令行窗口，执行：

```bash
npm install
```

我这儿用了九秒钟，下了 230 个包，装完就成功了。你们执行的时候可能比较慢，耐心等一等。

依赖装好，启动它：

```bash
npm run dev
```

这条命令要记住，以后每次启动前端都用它。`npm run` 这个形式和 Python 项目里的 `uv run` 是一个意思：Python 项目用 uv 管理，前端项目用 npm 管理，所以是 `npm run`。run 谁？run `dev`。为什么叫 dev？因为现在做的是开发测试，`develop` 简称 `dev`，是开发模式下的本地部署，不是真实部署——真实部署是另外一套做法，通常交给专门的服务器软件去做。

启动成功后端口是 `5173`，所以访问地址是：

```text
http://localhost:5173
```

一定要用这个地址访问，别用别的。页面打开，商城的界面就出来了；同时你会看到"产品加载失败"之类的报错——这是正常的，因为后端还没写，前端只有页面样式，没有数据。数据要靠后端查出来。能看到这个页面，就说明前端部署成功了，报错先不用管。

### 初始化后端项目

前端页面有了，但看不到任何产品数据，因为后端还没开始开发。从这一节起，正式进入 `agent-service` 的开发：先认识项目结构、项目依赖和基础代码模块，第一步是把项目搭起来。

开发肯定用 PyCharm，但项目一定要建在 `hm-insurance` 目录下，原因前面说过——前端在这个目录里，后端也放进来，将来用 Claude Code 这类 AI 编程工具时，它前后端都能看到，帮我们写前后端联调的代码会方便很多。

在 PyCharm 里新建项目，有几个地方要盯住：

- 项目类型选普通的 Python 项目；
- 位置先选到 `hm-insurance`。注意选完之后路径默认就停在 `hm-insurance` 这一层，一定要在末尾手动拼上 `agent-service`，这个名字千万别打错，最好从笔记上直接拷贝；
- 依赖管理选 uv，Python 版本选 `3.14`（这两项会跟随你本机的环境自动带出来）；
- 点 Create。

创建完会有两个提示。一个是问要不要选一个数据库的连接，暂时不着急选，先跳过。另一个是 Windows 的 Defender 要接管这个目录，需要把它排除掉，点一下排除、选"是"就可以了。

项目建好之后，紧接着就是依赖管理。这个项目要用到的依赖非常多，我提前准备了一份 `pyproject.toml`，你把里面 `dependencies` 这一段拷过来覆盖掉项目里的同名部分就行。上半部分的项目名、版本这些，按你自己创建项目时的信息来，不用动；当然整份全拷也没问题。

拷进去之后，你会看到依赖全是黄色的——黄色表示这些依赖还没安装。用一句玩笑话说，这是"涉黄了"。文件下半部分还指定了一个 uv 的依赖库地址，指向清华的镜像源。uv 默认从国外下载，速度并不快，换成国内镜像以后会快很多。

安装有两种方式：一是点编辑器里那个黄色圆圈，它会自动去装；二是打开终端，直接执行：

```bash
uv sync
```

等一会儿，黄色标识全部消失，说明依赖都装好了。

到这儿，第一步就算完成：项目初始化加上依赖管理。环境齐了，后面才谈得上写接口、连数据库、做前后端联调。

> **承前启后**：上一节讲完「安心宝：从项目全貌到能跑起来的环境」，下一节接着讲「先把地基打牢：分包、配置管理与统一日志」。

---

## 先把地基打牢：分包、配置管理与统一日志
> 对应块：BLK10 | 覆盖分集：P37-P40 | 块标题（分集名）：《项目结构、配置管理与统一日志》

### 代码先别急着写

项目建好了，依赖也装完了，按说下一步就是噼里啪啦写业务代码。先别着急。咱们这个项目不是一个小脚本，业务比较复杂，业务领域也多，真正上手之前，得先把"东西放哪儿"这件事定下来。实际开发中，对这种复杂项目，通常都要按职责划分出不同的包和模块，图的就是方便管理。

职责大致可以这么分：`core` 核心模块、`infrastructure` 基础设施模块、`common` 通用模块，以及 `modules` 业务模块。业务模块主要负责数据库的增删改查，以及和前端的对接。当然，"业务模块"只是一个概括的叫法，它内部还得按处理前端请求的流程继续往下拆。

### 一个请求进来，要走完三层

前端发起一个请求，到了服务端，第一站是 `router`——也就是 FastAPI 的 router 接口。它只干两件事：接收请求、校验前端传来的参数；以及把处理好的结果返回给前端。

真正的业务逻辑不能全塞在 router 里。之前做 AI 伴侣的时候，我们把所有代码都写在 router 里，结果 router 越来越臃肿，这肯定不合理。真实开发中，router 只负责请求参数的处理和响应的返回，业务逻辑要往下放到 `service` 模块里。

service 在处理业务的过程中，免不了要碰数据库的增删改查。这一步同样不要自己写，而是再往下放一层 `repository`。这一层没有业务，只做单纯的 CRUD——写库、增库、查库。把增删改查组合起来形成业务，是 service 层的事。

```text
前端请求
   │
   ▼
router        接收请求、校验参数、返回响应
   │
   ▼
service       组织业务逻辑
   │
   ▼
repository    只做数据库增删改查
   │
   ▼
数据库
   │
   ▼
repository ──► service ──► router ──► 前端响应
```

看图时顺着箭头走一遍就够了：请求从上往下穿过 router、service、repository 到数据库；结果再从数据库回到 repository，交给 service 加工，最后经 router 返回前端。这就是 web 开发里常听说的三层架构——router 层管前端请求，service 层管业务，repository 层管数据库增删改查。

### 两类数据：schemas 与 models

三层之外，还有两个专门装数据的东西，容易混。

一个是 `schemas`。可以把它理解成一种约束——对请求数据和响应数据的约束。将来前后端是分离开发的，AI 伴侣那会儿也是前后端分离，前后端就得遵循同一套接口文档：前端发起的请求参数长什么样、后端返回的结果长什么样，都有固定要求。这份要求落到 Python 代码里，就是一个个写好的类，统一放在 `schemas` 模块里。

另一个是 `models`。repository 要访问数据库做增删改查，我们用的是 SQLAlchemy 这套 ORM。ORM 是一种关系映射：把数据库中的表映射成 Python 里的类，表中的列就是类里的字段，表中的每一行数据就是类对应的一个对象。既然表要被映射，那每张表都得有一个对应的映射类，这些映射类统一叫 `models`。

所以业务模块一共是这么五个部分：`schemas` 和 `models` 装数据，`router`、`service`、`repository` 构成正常业务处理的三层。

| 包 | 职责 | 对接谁 |
| :--- | :--- | :--- |
| `router` | 接收请求、校验参数、返回响应 | 前端 |
| `service` | 组织业务逻辑 | 业务 |
| `repository` | 单纯的增删改查 | 数据库 |
| `schemas` | 请求 / 响应的数据约束 | 前后端接口契约 |
| `models` | 数据库表的映射类 | 数据库表结构 |

### 按职责分包：够用，但会堵

模块的职责划分清楚了，真正分包的时候却有两种不同的分法。第一种就是 AI 伴侣项目里用过的：纯粹按职责分包，一个职责一个包。

```text
app/
├── main.py              项目入口，创建 FastAPI 实例
├── core/                核心配置、日志、统一异常处理
├── infrastructure/      数据库、Redis、模型等外部设施
├── common/              通用代码
├── api/
│   └── router/          各业务的接口文件（chat.py、user.py …）
├── service/             各业务的业务逻辑
├── repository/          各业务的数据库增删改查
├── schemas/             请求 / 响应的数据约束
└── models/              数据库表的映射类
```

总包叫 `app`，里面 `main.py` 是项目入口，负责创建 FastAPI 实例。往下就是刚才说的那些职责包：`core` 放项目核心的配置、日志、统一异常处理；`infrastructure` 就是基础设施，将来要连接的外部东西——数据库、Redis、各类模型——都可以在这个模块下初始化好；如果还有一些通用代码，可以再写一个 `common`。

业务侧则是五个包：`api/router` 专门放 router 文件，将来有对话业务、用户管理业务，就分别建不同的文件；router 往下是 `service` 包；service 往下是 `repository` 包；和前端交互的数据约束放 `schemas`；和数据库交互的表映射放 `models`。包分完了，再按具体业务分成一个个文件——一个业务一个单独的文件，这样就把它们分离开了。

这种方式在中小型项目里挺不错。但项目越来越复杂以后会出问题：一个项目里的业务模块可能达到十几个、几十个甚至上百个，`router` 包里就会有几十上百个 router 文件，service、repository、models 也一样。一个包里文件太多，就不好管理了。

### 按业务分包：一个业务一栋楼

业务大起来，就该换第二种：按业务功能分包。它和第一种大体差不多，差别在于多了一个 `modules` 包，把所有业务收进去，再在 `modules` 内部按业务一个业务一个包。

```text
app/
├── main.py
├── core/
├── infrastructure/
├── common/
└── modules/
    ├── chat/            对话
    │   ├── router.py
    │   ├── service.py
    │   ├── repository.py
    │   ├── models.py
    │   └── schemas.py
    ├── conversation/    会话管理
    ├── agent/           Agent 相关
    └── user/            用户管理
```

每个业务包里，再按刚才那五个模块切成 `router`、`service`、`repository`、`models`、`schemas`——相当于每个业务都自带一套自己的五件套，放在一起。conversation 相关的来五个，agent 相关的来五个，user 相关的再来五个。

这样一来，不管业务怎么复杂，将来扩展到十个、一百个还是更多，`modules` 下无非是多一个包，而每个包里最多就是五个文件。开发某个业务时，只专注于这个包内部的五个文件、它们相互之间的调用，不会影响到别人。业务之间的边界因此划得更清晰，这是一种更加高内聚的划分方式。

> **结论**：中小型项目用第一种，纯粹按职责分包；大型项目更推荐第二种，按业务功能分包。咱们这个项目就按第二种来。

### 配置为什么不能写死在代码里

包分好了，接下来可以动代码，但第一件要写的不是业务代码，而是配置管理。这个概念在 AI 伴侣项目里根本没考虑过。

一个项目里有很多信息都属于配置信息：项目的端口、项目名称、DeepSeek 的密钥、DeepSeek 的地址、数据库的地址端口用户名密码、日志级别，这些全都算。在 AI 伴侣里，除了密钥是配到环境变量里的，其余的基本都直接在代码中写死。

写死有几个麻烦。第一，它分散在项目的各个角落，这也有那也有，管理起来不方便。第二，开发时是一套值，到了线上要换一套值，那就得满项目去找、去改。第三，密钥这类东西写在代码里，还有泄露的风险。

那把这一堆配置全配到系统环境变量里行不行？一个密钥还行，配上环境变量也比较安全；但一堆配置全塞进去，配起来太繁琐，改起来也不方便，数量还太多。

企业里的通行做法，是在项目里放一个 `.env` 文件，把项目的关键参数都写进去——端口、名称、密钥、地址、数据库，全放这一个文件。将来项目读取这个文件拿参数；要修改也只改这个文件，不用再去项目代码里翻。

有人马上会问：那密钥放在文件里不还是有泄漏风险？不要着急。这个文件只在开发阶段用。真正部署上线的时候，密钥这类隐私性强的值不会放在文件里，而是用系统环境变量或命令行的临时参数，在部署那一刻替换掉。上线以后，文件里留的其实就是个假密码，自然也就不用担心泄漏了。

所以真实的企业开发，是 `.env` 文件结合系统环境变量、命令行参数，一起来完成项目核心参数的配置和管理。

### Pydantic Settings：六种来源，一套优先级

参数一会儿在文件里、一会儿在系统环境变量里、一会儿在命令行里，怎么统一管？靠 `pydantic-settings`。它能同时兼容上面说的所有配置方式。

依赖是 `uv add pydantic-settings`，不过创建项目时已经装过了，直接用就行。

它支持的来源一共六种，优先级从高到低是这样：

| 优先级 | 来源 | 典型用途 |
| :--- | :--- | :--- |
| 1（最高） | 项目执行时的 CLI 命令行参数 | 部署那一刻临时传密钥 |
| 2 | `Settings` 构造函数传参 | 代码里显式指定，几乎很少用 |
| 3 | 系统环境变量 | 线上存放密钥 |
| 4 | `.env` 文件 | 开发阶段放全部配置 |
| 5 | secrets 目录 | Linux 下专门存放密码的目录 |
| 6（最低） | 字段默认值 | 兜底 |

加载顺序是从上往下逐层看：某一层读到了值，后面再碰到相同的字段就不再用；只有这一层没有，才继续往下找。也就是说，优先级最高的那个为准。

举个例子。`.env` 文件里配了四个参数：host、port、debug、API 密钥。系统环境变量里也配了 port 和 debug，值还和文件里的不一样。由于系统环境变量优先级更高，最终的 port 一定是 9000、debug 一定是 false，而不是文件里的那两个值。

再看字段默认值。默认值里也写了 host `127.0.0.1`、port、debug、API 密钥，但它们全都不生效：host 会被文件里的 `0.0.0.0` 覆盖，API 密钥也会用文件里的，而不是默认值里的。最终结果就是 host 取 `0.0.0.0`、port 取 `9000`、debug 取 `false`、密钥取自 `.env`。

所以策略很清楚：开发阶段把配置全放 `.env`；线上部署时，大部分配置仍然留在文件里，只把密钥这类隐秘参数用命令行参数或系统环境变量覆盖掉。因为这两者优先级比 `.env` 高，文件一个字都不用改，覆盖就生效了。

### 把 .env 建在项目根目录

第一件事，在项目里新建一个普通文件，名字就叫 `.env`——前面那个点一定要有，`env` 是 environment 的缩写，表示环境配置。

文件里按四组来写：

| 配置组 | 内容 | 供谁使用 |
| :--- | :--- | :--- |
| 应用配置 | 应用名、是否开启 debug、host、端口 | 初始化 FastAPI |
| 日志配置 | 日志级别，默认 INFO | 初始化日志 |
| 数据库配置 | 主机、端口、库名、用户名、密码 | 初始化数据库 |
| 模型配置 | DeepSeek 的 API Key、模型名 | 初始化模型 |

```ini
# ---------- 应用配置 ----------
APP_NAME=黑马 AI 应用
APP_HOST=0.0.0.0
APP_PORT=8000
APP_DEBUG=true

# ---------- 日志配置 ----------
LOG_LEVEL=INFO

# ---------- 数据库配置 ----------
DB_HOST=192.168.100.128
DB_PORT=5432
DB_NAME=ai_app
DB_USER=postgres
DB_PASSWORD=postgres

# ---------- 模型配置 ----------
DEEPSEEK_API_KEY=sk-xxxxxxxxxxxxxxxx
DEEPSEEK_MODEL=deepseek-v4-flash
```

数据库用的是 PostgreSQL，装在虚拟机里，所以 `DB_HOST` 填的是虚拟机的地址；端口就是 PostgreSQL 的 `5432`；库名、用户名、密码按课程环境来。这些大家基本都一样，不用改。模型这边用的是 DeepSeek V4 的 Flash，便宜，速度也快，很适合这个项目。

整个文件里，真正需要你动手的只有 API Key 这一项。

### config.py：让配置自己找上门

配置属于核心代码，所以放在 `app/core/config.py`。这份代码不用手写，直接复制，但每一段在干什么得心里有数。

建目录的时候有个坑要避开：`app` 和 `core` 都要建成 Python package，不是普通的 directory。包的图标上带个圈，里面还会自动生成一个 `__init__.py`；directory 没有这些，导入时会出问题。建好 `app` 包，在它里面再建 `core` 包，然后在 `core` 里新建 `config.py` 文件，把代码粘进去。

```python
from pathlib import Path

from pydantic import Field, PostgresDsn, computed_field
from pydantic_settings import BaseSettings, SettingsConfigDict

# 当前文件位于 app/core/config.py，向上两级就是项目根目录
app_root = Path(__file__).resolve().parents[2]


class EnvSettings(BaseSettings):
    """所有配置类的公共父类，统一声明 .env 的位置与读取规则"""

    model_config = SettingsConfigDict(
        env_file=app_root / ".env",
        env_file_encoding="utf-8",
        extra="ignore",
    )


class AppSettings(EnvSettings):
    name: str = Field(default="AI 应用", alias="APP_NAME")
    host: str = Field(default="127.0.0.1", alias="APP_HOST")
    port: int = Field(default=8000, alias="APP_PORT")
    debug: bool = Field(default=False, alias="APP_DEBUG")


class DatabaseSettings(EnvSettings):
    host: str = Field(default="127.0.0.1", alias="DB_HOST")
    port: int = Field(default=5432, alias="DB_PORT")
    name: str = Field(default="postgres", alias="DB_NAME")
    user: str = Field(default="postgres", alias="DB_USER")
    password: str = Field(default="postgres", alias="DB_PASSWORD")

    @computed_field
    @property
    def url(self) -> str:
        return str(
            PostgresDsn.build(
                scheme="postgresql+asyncpg",
                username=self.user,
                password=self.password,
                host=self.host,
                port=self.port,
                path=self.name,
            )
        )


class LLMSettings(EnvSettings):
    api_key: str = Field(default="", alias="DEEPSEEK_API_KEY")
    model: str = Field(default="deepseek-v4-flash", alias="DEEPSEEK_MODEL")


class LogSettings(EnvSettings):
    level: str = Field(default="INFO", alias="LOG_LEVEL")


class Settings(BaseSettings):
    app: AppSettings = Field(default_factory=AppSettings)
    db: DatabaseSettings = Field(default_factory=DatabaseSettings)
    llm: LLMSettings = Field(default_factory=LLMSettings)
    log: LogSettings = Field(default_factory=LogSettings)


settings = Settings()
```

#### app_root：不写死盘符的路径定位

`__file__` 是当前模块的一个变量，表达的就是这个文件本身——在这里它指向 `config.py`。`Path(__file__)` 把文件路径包成 Path 对象，`.resolve()` 把它解析成绝对路径，`.parents` 则是它的父目录序列。

`parents[2]` 里的 2 是往上数两级。以 `config.py` 所在的位置为起点：它自己在 `core` 目录里，往上第一级是 `app`，再往上一级就是项目根目录。`parents[0]` 是 `core`，`parents[1]` 是 `app`，`parents[2]` 就是项目根目录——`.env` 正好放在那儿。

为什么不直接写死 `D:\...`？因为项目将来要上线部署，不一定跑在 Windows 上，项目所在的位置也会挪。位置会变，但项目的目录结构不会变。用"从当前文件往上找两级"的方式定位，项目挪到哪儿都能找到 `.env`，比写死盘符可靠得多。

反过来也有一条硬要求：你的项目结构必须和课程里一模一样。结构但凡不一样，这个相对层级就对不上，`.env` 也就找不到了。

#### EnvSettings：告诉 Pydantic 去哪儿读

`EnvSettings` 继承 `BaseSettings`，这一步不能省——只有继承了它，配置读取才会生效。

类里的 `model_config` 就是导入进来的 `SettingsConfigDict`，里面三个参数各管一件事：

- `env_file`：`.env` 文件的位置。`app_root / ".env"` 里的斜杠是 Path 的路径连接符，等价于"项目根目录下的 `.env`"。
- `env_file_encoding`：用 `utf-8` 读，因为文件里可能有中文。
- `extra="ignore"`：读取时忽略未定义的字段。

`extra="ignore"` 值得多说一句。Pydantic Settings 不只看 `.env`，还会读当前这个 Settings 的构造函数传参、系统环境变量、命令行参数。系统环境变量里的条目非常多，除了我们自己配的，还有电脑自带的。这些机器变量对项目毫无用处，如果全读进来，类里就会多出一堆没定义的东西。设为 `ignore` 以后，它只把 Settings 里将来定义过的那些变量读进来，没定义的直接忽略。

#### 四组配置类：别名才是对接 .env 的钥匙

`.env` 里分了四块，配置类也分成四个，一一对应：`AppSettings` 对应用应用配置，`DatabaseSettings` 对应数据库配置，`LLMSettings` 对应 DeepSeek 的模型配置，`LogSettings` 对应日志配置。

为什么不把四组全塞进一个类？塞得下，但类里的代码会越来越多，将来不方便管理。拆成四个，各管一块。

四个类都继承 `EnvSettings`，所以它们都共享同一份 `.env` 位置和读取规则。

字段名和 `.env` 里的键名对不上，这是故意的。比如 `AppSettings.name` 对应的是 `.env` 里的 `APP_NAME`，`host` 对应 `APP_HOST`，`port` 对应 `APP_PORT`，`debug` 对应 `APP_DEBUG`。这个映射靠 `Field(alias=...)` 声明：

```python
name: str = Field(default="AI 应用", alias="APP_NAME")
```

前面的 `name` 是给开发者用的变量名，小写、一目了然，写代码时 `settings.app.name` 顺手；后面的 `alias` 才是 Pydantic 去 `.env` 或系统环境变量里找值用的键。所以别名必须和配置文件里的键严格一致，变量名则可以随你起。

`DatabaseSettings` 的五个字段同样是一一映射：`host` 对 `DB_HOST`、`port` 对 `DB_PORT`、`name` 对 `DB_NAME`、`user` 对 `DB_USER`、`password` 对 `DB_PASSWORD`。

每个字段还带了 `default`。默认值的优先级是最低的：环境变量、命令行参数、`.env` 里都找不到时才会用它。现在 `.env` 里明明有值，所以这些默认值一个都不生效，最终填进来的全是文件里的值。

| 配置类 | 字段 | 别名 | 取自 `.env` 的哪一块 |
| :--- | :--- | :--- | :--- |
| `AppSettings` | `name` / `host` / `port` / `debug` | `APP_NAME` / `APP_HOST` / `APP_PORT` / `APP_DEBUG` | 应用配置 |
| `DatabaseSettings` | `host` / `port` / `name` / `user` / `password` | `DB_HOST` / `DB_PORT` / `DB_NAME` / `DB_USER` / `DB_PASSWORD` | 数据库配置 |
| `LLMSettings` | `api_key` / `model` | `DEEPSEEK_API_KEY` / `DEEPSEEK_MODEL` | 模型配置 |
| `LogSettings` | `level` | `LOG_LEVEL` | 日志配置 |

#### 数据库 URL：用计算字段拼出来

`DatabaseSettings` 里有一个特殊字段 `url`。它不是一个普通的赋值，而是一个函数，用 `@computed_field` 加 `@property` 两个装饰器标出来的。

函数体里调用 `PostgresDsn.build(...)`，把数据库连接的各个部分传进去：`scheme` 是协议前缀，这里是 `postgresql+asyncpg`，加号前面是数据库类型，加号后面是底层驱动；`username`、`password`、`host`、`port` 读的都是上面几个字段的值；`path` 是数据库名。`build` 会把这些东西拼成一条完整的连接 URL。

为什么要自动拼？回想 AI 伴侣那会儿，数据库地址是手动拼的。一条数据库连接 URL 的组成是固定的：

```text
postgresql+asyncpg://user:password@host:5432/dbname
└──────┬────────┘   └─┬─┘ └──┬───┘ └─┬─┘ └┬─┘ └──┬──┘
    协议+驱动       用户名   密码    主机  端口  数据库名
```

那时连的是 MySQL，协议部分是 `mysql+aiomysql`；现在换成 PostgreSQL，协议变成 `postgresql+asyncpg`，其余部分按用户名、密码、主机、端口、数据库名的顺序拼起来。以前这些字符要自己一段段接，现在把零件交给 `PostgresDsn.build`，它负责拼好。

之所以写成函数而不是直接 `url = "..."`，是因为拼接是一段逻辑，没法用一个等号表达完。这个函数返回的就是真正的 URL 路径，它表达的是"一个值"，所以加上 `@property`，让它能像属性一样被访问；再加 `@computed_field`，声明这个字段是计算出来的。

> **易错点**：两个装饰器的顺序不能反，`@computed_field` 要在上、`@property` 在下。写反了它就不是计算字段，序列化时也拿不到 `url`。

#### Settings：把四组配置收进一个对象

四组配置写完了，最后用一个 `Settings` 类把它们汇总。它同样继承 `BaseSettings`，内部四个字段分别是四个配置类的类型，用 `default_factory` 指定怎么构造：

```python
app: AppSettings = Field(default_factory=AppSettings)
```

`app`、`db`、`llm`、`log` 四个字段，值分别来自前面那四个类。这样 `Settings` 就相当于把四组配置集成到了一起。

文件末尾直接初始化：

```python
settings = Settings()
```

构造函数里没有传任何参数，所以"构造函数传参"这条配置来源就主动放弃了。`settings` 里的值全部来自 `.env`、系统环境变量、命令行参数这些外部来源。

这个模块一加载，`Settings()` 就会自动初始化：读 `.env`、读系统环境变量、读命令行参数，把配置全部装进对象。以后任何模块要用配置，导入这个 `settings` 就行，访问路径是一层层点下去的——`settings.app.name` 拿到应用名，`settings.app.host` 拿到主机地址，`settings.db.url` 拿到拼好的数据库连接串。

### 配置怎么取、怎么扩

验证配置有没有读进来，不要在 `app` 里写测试。`app` 是项目代码，测试和演示不属于项目代码。在项目根目录下新建一个 `notebooks` 目录，再在里面建 `day03` 目录，放一个 `01_settings_demo` 的 notebook。注意这两个都是目录，不是 Python 包——以后每天的演示、测试都往 `notebooks` 里放，项目代码才建成包。

notebook 里先导包：

```python
from app.core.config import settings
```

导进来就能直接用。`settings.app` 拿到的是 `AppSettings` 对象，它下面有 name、host、port、debug 四个属性；打印 `settings.app.name`，输出的就是 `.env` 里 `APP_NAME` 配的那个应用名。

想看全部配置，直接打印整个对象：

```python
print(settings.model_dump_json(indent=2))
```

`Settings` 继承了 `BaseSettings`，而 `BaseSettings` 本质上是 Pydantic 的 `BaseModel`，所有 BaseModel 都自带 `model_dump_json` 方法，能把类里的字段转成 JSON 打印出来，还能用 `indent` 设置缩进。缩进设成 2，输出就很漂亮：最外层是 `app`、`db`、`llm`、`log` 四个字段，`app` 里是 name、host、port、debug，`db` 里五个字段加一个拼好的 `url`，`llm` 和 `log` 各自的字段也都在。

以后要取值，顺着 `settings` 往下点就行：`settings.app.name`、`settings.db.url`、`settings.llm.api_key`、`settings.log.level`。

配置会经常加，加的时候记住两个动作就够了。

比如模型配置要多一个 `DEEPSEEK_BASE_URL`，先定位到跟模型配置相关的 `LLMSettings`，在里面加一个字段。不会写就复制一行现成的，把变量名改成 `base_url`——这个名字随你起，是给自己看的；但后面的 `alias` 不能乱，它才是去 `.env` 或系统环境变量里找值的键，必须写成 `DEEPSEEK_BASE_URL`，和配置文件里保持一致。至于默认值，想配就配，不想配删掉也没关系。

如果是要加一整组全新的配置，那就把这个类整体复制一份，改个类名，再到 `Settings` 里补一个对应的字段。这套代码不用会从头写到尾，会复制、会改，就够用了——到了企业里，这类配置代码其实大同小异。

### 统一日志：配置一次，到处取用

配置管好了，接着做统一日志。AI 伴侣里也记日志，但用的是系统自带的 logging，很简单。这里换一个工具：`structlog`，结构化日志。它是第三方包，需要单独装依赖，提前已经装好了，直接用。

日志代码放在 `app/core/logger.py`——日志也属于核心配置，所以跟 `config.py` 放在一起。

这份代码里有三块东西。第一块是 `configure_logging`，负责配置控制台日志：输出经过格式化，结构更清晰，还带颜色，看起来舒服。第二块是 `get_logger`，负责获取日志记录器：将来记日志不可能都在这个文件里记，代码会分成很多模块，每个模块都可能要记日志，所以提供一个函数，传一个名字进去，它就返回一个对应的日志记录器。第三块是一个下划线开头的私有函数，属于内部实现，不用管。

所以真正要用的只有两个函数：配置日志、获取日志。

```python
from app.core.config import settings
from app.core.logger import configure_logging, get_logger

# 项目启动时执行一次
configure_logging(level=settings.log.level)

# 各业务模块里按需获取，传入模块名
logger = get_logger(__name__)
logger.info("应用启动完成")
```

`get_logger` 传什么名字？通常就传模块名。比如将来写用户管理模块，就传跟用户相关的模块名。这样日志打印到控制台时会带上模块名，哪条日志是哪个模块输出的，一目了然。

日志级别默认是 `INFO`。调用 `configure_logging` 时可以传级别，但不要写死，要从 `settings.log.level` 里读——也就是从 `.env` 的 `LOG_LEVEL` 里读。这样以后要调日志级别，只改 `.env` 一个地方就够了。

> **注意**：两件事记住就行。配置日志在项目启动时执行一次，放在 `main` 里；其他模块每次先 `get_logger` 拿到记录器，再去记日志。级别不要硬编码，从配置里取。

> **承前启后**：上一节讲完「先把地基打牢：分包、配置管理与统一日志」，下一节接着讲「从启动模块到第一个接口：FastAPI 项目的骨架与业务规范」。

---

## 从启动模块到第一个接口：FastAPI 项目的骨架与业务规范
> 对应块：BLK11 | 覆盖分集：P41-P45 | 块标题（分集名）：《FastAPI 启动模块与接口开发规范》

统一配置管理和日志管理做完，项目其实已经搭得差不多了——配置能读、日志能打，架子立住了。剩下的就是给它安一个入口，让它真的跑起来。这个入口就是 `main.py`。

位置有讲究：直接放在 `app` 包下，不要塞进 `core` 里。`core` 装的是配置、日志这类通用能力，启动模块是项目的门面，两者职责不是一回事。

### 启动模块：先让项目跑起来

启动模块要按固定的顺序做三件事：配日志、创建 FastAPI 对象、挂接口。最后才是用 uvicorn 把它拉起来。

#### 日志只在启动模块配一次

第一件事不是创建 FastAPI，而是配日志。

`core` 里的 logger 模块提供了两个函数：`configure_logging` 和 `get_logger`。这两个函数的分工必须分清：

- `configure_logging` 是一次性的。在这个进程里执行过一次，全局的日志格式、输出方式、级别就定下来了，别的地方不用再执行；
- `get_logger` 是随用随取的。哪个模块要打日志，就在哪个模块里取一个记录器。

配置时要传日志级别。这个级别千万不要写死——写死就意味着以后改级别得改代码、重新发布。从 `settings.logging.level` 读，要改就改 `.env` 里的 `LOG_LEVEL`。

取记录器要传一个名字，通常传模块名。Python 每个模块都有一个内置变量 `__name__`，它就是当前模块的名字。传进去以后，日志打出来会带着 `app.main`，一眼就知道这条日志是谁记的。

> **易错点**：`configure_logging` 只该在启动模块里执行一次，其余模块一律只调 `get_logger`。这一点在后面写数据库模块时会立刻用到——那里一个 `configure_logging` 都没有。

#### 创建 FastAPI 对象

初始化 FastAPI，就是创建 `FastAPI` 类的对象。

至少要传一个 `title`，让别人知道这个项目叫什么。这个值在 `.env` 里已经配成 `APP_NAME` 了，直接读 `settings.app.name`。

还有一个以前可能没用过的参数：`debug`。它控制 FastAPI 运行时要不要用 debug 模式。开了以后，记日志会记得更丰富，方便调试。

但这里有个容易混的点：`debug` 只决定"记不记、记多细"，不决定"你看不看得到"。你能看到多少，取决于日志级别——级别是 debug，看到的就多；级别是 info，看到的就少。两件事是分开的。

> **提示**：`debug` 管日志记录的详细程度，日志级别管日志的显示门槛。开了 debug 却没看到更多日志，先去看级别，别怀疑参数没生效。

`title` 和 `debug` 都从 `settings.app` 里读，将来改项目名、开关调试模式，都不用动代码。

#### 健康检查接口

FastAPI 定义接口就是 `app.get`、`app.post` 这种形式，点号后面跟请求方式。

先写一个健康检查接口，路径 `/health`，GET 请求，`description` 写清楚"健康检测接口"——这段描述会直接出现在自动生成的接口文档里。

函数写成 `async def`。这个项目里的接口统一都是异步的。

函数体只有一件事：返回 `{"status": "ok"}`。别人来调这个接口，看到 ok 就知道服务是正常的。返回值用字典就行，里面装字符串。

顺手再记一条日志，证明这个接口确实被调用了。日志有好几个级别——debug、info、warning 等等，健康检查属于正常事件，用 `info` 就够了。

#### 用 uvicorn 跑起来并验证

启动那几行是死代码，不用手写，直接从笔记里抄：

```python
if __name__ == "__main__":
    uvicorn.run(...)
```

关键是 `uvicorn.run` 的第一个参数。以前项目在根目录，写 `main:app` 就行；现在 `main.py` 在 `app` 包里，所以要写成 `app.main:app`——冒号前面是模块路径，冒号后面是 FastAPI 对象的变量名。uvicorn 靠它定位到要启动的应用。

主机名和端口号同样不写死，从 `settings.app.host` 和 `settings.app.port` 读。这个项目配的端口是 8001。

把前面几段拼起来，此时 `app/main.py` 是这样：

```python
import uvicorn
from fastapi import FastAPI

from app.core.config import settings
from app.core.logger import configure_logging, get_logger

configure_logging(settings.logging.level)
logger = get_logger(__name__)

app = FastAPI(
    title=settings.app.name,
    debug=settings.app.debug,
)

@app.get("/health", description="健康检测接口")
async def health():
    logger.info("健康检测接口正常调用")
    return {"status": "ok"}

if __name__ == "__main__":
    uvicorn.run(
        "app.main:app",
        host=settings.app.host,
        port=settings.app.port,
    )
```

启动之后，打开 `http://127.0.0.1:8001/docs`，这是 FastAPI 自动生成的接口文档。点开 `try it out` 发一次请求，能拿到 `{"status": "ok"}`。

也可以更省事：因为是 GET 请求，直接在浏览器地址栏敲 `http://localhost:8001/health`，一样能返回 ok。

回到控制台看日志：级别是 info，内容是健康检查正常，记录者是 `app.main`。这就是 `get_logger(__name__)` 的效果——模块名被带进了每一条日志。日志还带颜色，这是 structlog 的作用：结构化、清晰，一眼看得出是谁记的。

> **提示**：这个启动模块的代码基本是固定套路，抄下来就行。真正要记住的是日志的用法：启动模块里配一次，别的地方只 `get_logger`。

### 数据库连接池：把基础设施补上

启动类跑通，说明项目最核心的架构已经有了。但这时候先别急着写业务代码——基础设施还没搞定。

数据库连接池要在全局统一配置，还要在 FastAPI 的生命周期里做管理。连接池本身之前学过，AI 伴侣那个项目里用的是 MySQL；这次换成 PostgreSQL，代码大同小异。这类基础代码到了企业里通常也是写好的，所以目标不是背下来，而是看得懂、会改、会用。

#### 补齐连接池的四项配置

之前 `config` 里只配了数据库的基础信息，也就是连接地址；连接池的参数是空的。所以第一步是把这几项补进 `DatabaseSettings`：

| 配置项 | 作用 |
| :--- | :--- |
| `echo` | 是否让 SQLAlchemy 把执行的 SQL 语句输出出来 |
| `pool_size` | 连接池的大小，也就是常驻连接数 |
| `max_overflow` | 连接池溢出时最多还能再加多少连接 |
| `pool_recycle` | 连接的回收时间，到期就换新连接 |

这四项在配置类里都给了默认值，所以不必写进 `.env`。想改的时候，再按对应的别名去 `.env` 里覆盖就行。

> **提示**：`echo` 打开后控制台会刷出大量 SQL，调试时有用，平时建议关掉，否则日志会被淹没。

#### 基础设施包与 database.py

数据库连接属于基础设施。按项目的结构约定，基础设施要放在一个单独的模块里，叫 `infrastructure`，项目里简写成 `infra`。

所以在 `app` 下新建一个包 `infra`（一定要是包，不是普通目录），里面再建 `database.py`，把数据库连接的配置和工具都放进去。

这个文件开头先导入 SQLAlchemy 的依赖——引擎、会话、会话工厂这些；再导入项目自己的 `settings` 和日志。

注意这里的日志只调 `get_logger(__name__)`，没有 `configure_logging`。原因就是前面说的：日志配置在启动模块里已经做过一次了，这里只需要取出记录器，把当前模块名传进去，以后这个文件打的日志就会标成 `app.infra.database`。

#### 引擎与 session 工厂

初始化一上来先做了一次安全检查：如果没有配置数据库 URL，就直接 `raise` 一个错误。这相当于一道保险，避免带着空地址去连库，报出一堆看不懂的底层异常。项目里当然配了 `DB_URL`，所以这一步肯定能过。

接着是 `create_async_engine`，创建异步数据库引擎。和连 MySQL 的写法比，几乎没有区别，唯一的区别在 URL 的协议头：以前是 MySQL 协议，这次是 PostgreSQL 协议。

正因为 URL 放在配置文件里，代码里根本看不出用的是哪种数据库。这也带来一个很实用的结论：**换数据库时，这段代码一行都不用改，改的只是配置里 URL 的 scheme（协议）**。

顺便说一句，这个协议其实理论上也应该做成配置项，那样连"用哪种数据库"都不用进代码。不过目前它写在 URL 里，已经足够灵活了。

引擎的参数包括 URL、`echo`、连接池大小、溢出上限、回收时间，还有取连接的超时时间、每次取连接前要不要做检查。这些值全部从 `settings` 读，没有一个是写死的，将来都能通过配置文件覆盖。

引擎之后是 session 工厂：

```python
AsyncSessionFactory = async_sessionmaker(
    engine,
    autoflush=False,
    expire_on_commit=False,
)
```

名字故意用大写开头，是为了告诉读代码的人：这是一个工厂，不是一个普通变量。

多传的两个参数各自解决一个坑：

- `autoflush=False`：不自动刷新。执行查询前 SQLAlchemy 默认可能先把挂起的改动刷进数据库，这里关掉，将来需要的时候手动刷，行为更可控；
- `expire_on_commit=False`：提交后不把对象标记为过期。默认情况下，session 提交或关闭之后，之前查出来的对象属性会失效，再去访问就会报错；关掉这个行为，session 关闭后对象依然可用，做更新操作时省事很多。

> **提示**：这两个参数以前没学过也不要紧，先按这个配。企业项目里基本也是这么配的，照着用就行。

#### 三个工具函数

引擎和工厂都建好之后，文件里再提供三个工具函数。

第一个是 `get_session`。用 `async with` 上下文管理器创建 session，然后 `yield` 把它交出去——函数执行到这里暂停，谁拿到 session 谁去用；用完了控制权回来，上下文管理器自动把 session 关闭。

将来在 FastAPI 的路由里，用依赖注入的方式就能拿到这个 session。以前手写的时候，要自己 `commit`、异常时 `rollback`、最后 `close`；有了上下文管理器，这些收尾动作它全包了，代码干净很多。

第二个是 `close_database`。它调用 `engine.dispose()`，把整个连接池销毁，然后记一条日志。这个函数显然应该在项目关闭时执行。

第三个是 `check_database`。它用引擎 `connect()` 拿到一个连接，随便执行一条 `select 1`。语句本身不重要，重要的是让引擎真正建立连接。

这里有个很关键的认知：**模块被加载时，只是把引擎和 session 工厂创建出来了，连接池里其实一个连接都没有**。真正的连接是在第一次向数据库发请求时才创建的。所以 `check_database` 才是"初始化数据库连接"的那个函数——它一执行，引擎就会按 `pool_size` 把连接建好。

> **易错点**：不要以为 `database.py` 被 import 就等于连上了数据库。引擎是惰性的，连接要等第一次真正访问时才建立。

整份 `database.py` 的骨架如下：

```python
from sqlalchemy import text
from sqlalchemy.ext.asyncio import async_sessionmaker, create_async_engine

from app.core.config import settings
from app.core.logger import get_logger

logger = get_logger(__name__)

if not settings.database.url:
    raise ValueError("数据库连接地址未配置")

engine = create_async_engine(
    settings.database.url,
    echo=settings.database.echo,
    pool_size=settings.database.pool_size,
    max_overflow=settings.database.max_overflow,
    pool_recycle=settings.database.pool_recycle,
    pool_timeout=settings.database.pool_timeout,
    pool_pre_ping=settings.database.pool_pre_ping,
)

AsyncSessionFactory = async_sessionmaker(
    engine,
    autoflush=False,
    expire_on_commit=False,
)

async def get_session():
    async with AsyncSessionFactory() as session:
        yield session

async def close_database():
    await engine.dispose()
    logger.info("数据库连接已关闭")

async def check_database():
    async with engine.connect() as conn:
        await conn.execute(text("select 1"))
    logger.info("数据库连接初始化成功")
```

#### 用 lifespan 挂到 FastAPI 上

`check_database` 和 `close_database` 一个负责启动，一个负责关闭，正好对应 FastAPI 的启动和销毁。怎么让它们在这两个时刻自动执行？用生命周期钩子。

钩子函数用 `@asynccontextmanager` 装饰，这个装饰器从 `contextlib` 里导入。函数是异步的，名字叫 `lifespan`（生命周期的意思），参数接收 `app`，类型就是 `FastAPI`。

结构是 `try / finally` 夹一个 `yield`：

```python
@asynccontextmanager
async def lifespan(app: FastAPI):
    from app.infra.database import check_database, close_database

    logger.info("项目启动中，正在初始化……")
    try:
        await check_database()
        yield
    finally:
        await close_database()
```

`yield` 之前是启动阶段，`yield` 之后（写在 `finally` 里）是关闭阶段，中间这段时间 FastAPI 正常处理请求。

这里有个细节值得说：数据库函数的导入放在了函数内部，而不是文件顶部。原因是放到顶部的话，模块一被加载就会连带加载数据库模块；放进函数里，等于等到生命周期真正执行时才导入，是一种延迟加载。但完全塞进 `try` 里又会"看不见"，所以最终放在 `try` 之前、函数开头——既保持了延迟，又看得清楚。

钩子写好后还要让它生效：在创建 FastAPI 对象时加一个 `lifespan` 参数，值就是刚写的这个函数。

```python
app = FastAPI(
    title=settings.app.name,
    debug=settings.app.debug,
    lifespan=lifespan,
)
```

这样 FastAPI 创建时会调用它，并把 `app` 自己传进去；关闭时也会走到 `finally`。

验证方式很直接：重启项目，控制台先打印"项目启动中，正在初始化"，紧接着打印"数据库连接初始化成功"，记录者是 `app.infra.database`；把项目停掉，日志里会出现"数据库连接已经关闭"。一进一出都对上，说明初始化和关闭确实绑在了 FastAPI 的生命周期上。

到这里，这一轮做的事可以归成三件：在配置里补齐连接池参数、在 `app/infra/database.py` 里建好引擎与工厂并写出三个工具函数、用 `lifespan` 把初始化和关闭挂到项目启停上。

### 前后端分离：接口文档为什么是开发的前提

核心架构和基础设施都齐了，项目能跑起来，接下来才正式进入业务开发。但在写第一个接口之前，先把规范说清楚——不然写到一半就会发现，前端要的东西和后端给的东西对不上。

这个项目和之前的 AI 伴侣一样，采用前后端分离的架构。

#### 一次页面刷新背后的两条链路

所谓前后端分离，最直白的含义是：前端和后端各自独立部署，不在同一台服务器、不搅在一个工程里。

用户在浏览器里输入地址，浏览器先请求一堆静态资源——HTML、CSS、JS。这些请求到达前端服务器后，静态资源本来就在它本地，直接读文件返回就行。浏览器拿到这些文件，把页面渲染出来给用户看。

但这时候看到的只是"壳"，没有数据。首页刷新一下，布局、样式都在，产品列表是空的——因为产品数据还没查。

页面里那些动态数据（这个项目里就是保险产品数据）的请求，也要先发到前端服务器。打开浏览器控制台看 Network 会发现：请求地址还是 `localhost:5173`，和页面同源；但路径变成了 `/api/v1/...products` 这种形式。前端服务器一看到这个路径，就知道这是 API 请求，于是转给后端。

```text
浏览器
  │ ① 请求 html、css、js 等静态资源
  ▼
前端服务器（localhost:5173）
  │ ② 静态资源在本地，直接读文件返回
  ▼
浏览器：先把页面渲染出来（此时只有壳，没有数据）

浏览器
  │ ③ 页面发出的 /api/v1/... 请求，仍然先到前端服务器
  ▼
前端服务器：识别出这是 API 请求，转发给后端
  ▼
后端 FastAPI（localhost:8001）
  │ ④ 查询数据库
  ▼
数据库 → 后端 → 前端服务器 → 浏览器：渲染出动态数据
```

读图的关键在③：动态请求并不是浏览器直接打到后端，而是同源发给前端服务器，由它识别路径再转发。

#### 接口文档要写清四件事

前后端一旦分开，中间就必然有一次交互：前端把请求发给后端，后端把结果返回前端。这次交互不能靠口头约定，得有一套标准，这套标准通常就落成一份接口文档。

前端最关心的四个问题，文档里必须回答：

| 前端的问题 | 文档里的对应项 |
| :--- | :--- |
| 去哪里拿 | 请求的 URL 路径 |
| 怎么拿 | 请求方式（GET、POST 等） |
| 带什么条件 | 请求参数 |
| 拿到的东西长什么样 | 响应数据的格式 |

有了这份文档，前端知道自己该发什么请求，后端知道自己该实现什么接口，两边可以同时开工，不用互相等。开发效率的提升就来自这里——不是文档写得多漂亮，而是它把"等对方"这件事消掉了。

#### 项目里的两份文档

这个项目的笔记最下面放了两份接口文档：保险业务的接口文档和智能客服的接口文档。现在要做的是业务相关的部分，先看保险业务那份。

文档里不只有一个个接口，前面还有几块公共信息：接口地址、统一的路径前缀、结构格式说明、认证规则、数据字典。这些是读具体接口之前的地基——比如统一前缀决定了所有业务接口的 URL 长什么样，认证规则决定了请求要不要带 token。

再往下才是具体接口。每个接口都会列清楚请求方式、请求路径、请求参数、响应结果，照着实现就行。

> **提示**：接口文档不是参考读物，是这一阶段的施工图。实现接口时以文档为准，不要凭自己对"合理"的想象去改字段名或路径。

### 业务开发规范：按业务分包，按调用分层

文档层面的规范说完了，还有一层是代码组织上的规范。这一层决定了项目做大了以后是清清楚楚还是一团乱麻。

#### 每个业务一个包

整个项目按业务功能分包。所有跟业务功能有关的代码，都放在 `app/modules` 这个包下。

`modules` 里一个业务一个子包：

- 产品相关的业务 → `product` 包；
- 对话相关的业务 → `chat` 包；
- 用户管理相关的业务 → `user` 包。

每个业务包内部，再按职责建 `router`、`service`、`repository`、`schema`、`model` 这几类模块。

这样切的好处是"高内聚"：一个业务需要改什么，全在它自己的包里，不用满项目找。将来删掉一个业务，也是整个包一起删，不会留下半截代码。

#### 五类模块各司其职

这五类模块不是随便分的，它们对应一次请求从进来到出去的完整路径。

```text
前端请求
   │
   ▼
router       接收请求、校验参数、返回响应
   │
   ▼
service      业务逻辑处理
   │
   ▼
repository   只做数据库的 CRUD
   │
   ▼
数据库

schema  ──►  约束 router 的入参与出参
model   ──►  给 repository 提供数据库表映射
```

`router` 是第一道关口。请求进来先由它接住，它负责校验请求参数、以及最终把响应结果返回出去。接口的定义方式就是常见的 `app.get`、`app.post`，里面包含请求方式、请求路径、请求参数和响应结果。

但 `router` 不处理业务。校验完就把活交给 `service`——这一点很重要，路由层一旦开始写业务逻辑，分层就名存实亡了。

`service` 承担真正的业务逻辑。处理过程中如果需要查数据库，它也不自己查，而是调 `repository`。当然不是每个业务都要查库，有些纯计算的逻辑到 `service` 就结束了。

`repository` 只做一件事：数据库的增删改查。它用 SQLAlchemy 操作数据库，而 SQLAlchemy 需要知道 Python 类怎么对应数据库表，这就是 `model` 的职责——定义表映射类。

`schema` 则管住两头：`router` 收到的请求参数、返回的响应结果，都要有明确的约束。这个约束就是 schema。

> **易错点**：五类模块里最容易越界的是 `router`。只要在路由函数里出现了业务判断或数据库查询，就该往 `service` 挪。

#### 开发顺序为什么是从下往上

这五类模块有明确的先后关系，不能想到哪写到哪。

`schema` 和 `model` 是基础。`schema` 定义好之后，要一层层往 `service` 传；`model` 带着数据库表映射，最终查到数据时要往 `service` 返回。所以这两个必须先定义出来。

`router`、`service`、`repository` 之间是调用依赖：`router` 调 `service`，`service` 调 `repository`。既然上层依赖下层，开发就得从下往上——先写 `repository`，`service` 才有东西可调；`service` 写好了，`router` 才能调它。

所以完整的顺序是：先 `schema` 和 `model`，再 `repository` → `service` → `router`。看起来是"倒着做"，实际上顺着依赖的方向做，每一步都有已经存在的东西可以调用，不会出现写了一半只能干等的空壳。

### 接口分析：查询产品列表

要做的第一个接口是查询产品列表。首页现在刷不出产品，就是因为这个接口还没实现。

接口文档里产品相关的那一章，第一个就是查询产品列表。为了不用在文档和笔记之间来回切，笔记里已经把这段接口说明复制过来了，直接看笔记就行。

接口的定位是：查询有效的保险产品，可以按产品分类过滤，直接返回产品列表。这句话里有几个信息点，对应下面几段分析：怎么请求、按什么过滤、什么算有效、返回什么。

#### 请求方式与路径

请求方式是 GET，路径是 `/api/v1/products`。

拆开看：`/api/v1` 是统一的路径前缀，`products` 是请求的资源名称。这种"前缀 + 资源名"的写法完全符合 RESTful 风格——路径里出现的是名词（资源），而不是动词（动作），所以好记，也不容易起歧义。

文档里还给了一个请求示例：端口是 8001，路径是 `/api/v1/products`，后面按需要拼参数。

#### category 与四种险种

请求参数只有一个：`category`，产品分类。类型是字符串，而且不是必填。

这意味着两种用法：

- 不传：查全部有效产品；
- 传了：只查这个分类下的产品。

这里要弄清楚两件事。第一件是产品分类到底有哪几种。

回到首页看标签栏：默认是"全部"，后面跟着四个选项卡，对应项目里的四个险种——医疗险、重疾险、寿险、意外险。

点"全部"时，请求路径里只有 `products`，后面不拼任何参数；点某一个分类，路径后面才会拼上 `category`。四个分类对应的值分别是：

| 险种 | `category` 的值 |
| :--- | :--- |
| 医疗险 | `medical` |
| 重疾险 | `critical_illness` |
| 寿险 | `life` |
| 意外险 | `accident` |

这四个值不是凭空定的，数据库 `products` 表里就有 `category` 字段，存的就是这四个值。所以后端拿到参数以后，不需要做任何翻译或映射，直接拿它去表里过滤即可。

#### 什么叫"有效产品"

第二个要弄清楚的是"有效"。

数据库 `products` 表里有一个 `status` 字段，表示产品状态，取值是 `active` 和 `inactive`。`active` 是有效，另一种自然就是无效。

所以默认查产品时，必须加上 `status = 'active'` 这个条件。非 active 的产品，一律不返回。

> **易错点**：这个条件很容易漏。漏了不会报错，接口照样返回数据，但会把已经下架的产品一起返回给前端——数据看起来"能用"，其实是错的，这种 bug 最难发现。

#### 响应为什么是一个中括号

请求侧清楚了，剩下响应。

文档里的响应示例最外层是一对中括号。这个写法表示最终返回的是一个列表——Python 里的列表就是用中括号表示的。因为查询结果可能不止一个产品，所以外面必须是列表。

列表后面的 `.id`、`.name`、`.class_name`、`.category` 这些，描述的是列表里每个元素长什么样：一个元素要有 id、有 name、有 class_name、有 category，等等。

这些属性从哪来？看数据库表结构就明白了：接口文档要求的字段，和 `products` 表的字段完全一致——id 对 id、name 对 name、class_name 对 class_name、category 对 category，后面的介绍、图片地址、描述、最小值、最大值、目标人群、亮点、状态也都一一对应。

只有两个字段不在响应里：`created_at` 和 `updated_at`，也就是创建时间和更新时间。前端不需要这两个时间，返回时去掉。

到这里，这个接口的实现思路就已经完整了：拿 `category`（如果有）加上 `status = 'active'` 去 `products` 表里查，查出来的记录去掉两个时间字段，组装成列表返回。

### 写 schema：ProductResponse

按前面定的顺序，先做 `schema` 和 `model` 这两个基础模块。这一节先把 `schema` 落地。

`schema` 是五类模块里最简单的：它的全部工作就是遵循接口文档，把请求参数和响应结果的约束写出来。所谓约束，就是一个类，继承 Pydantic 的 `BaseModel`，将来由它负责数据校验。

#### 请求参数不用单独建类

看接口文档，这个接口的请求参数只有一个 `category`。

只有一个参数的时候，没有必要单独定义一个类。直接在 `router` 的方法参数列表里接收 `category` 就行，FastAPI 会把它当成查询参数处理。多写一个只有单个字段的类，除了增加一层跳转，没有任何收益。

#### 响应落成一个类

响应这边就复杂了。最外层是一个列表，列表里的元素有很多字段，结构和数据库表类似——这种东西必须用类来描述。

规则很朴素：字段在文档里是数字，就定义成 `int`；是字符串，就定义成 `str`；以此类推。

文档最后还给了一个响应示例，看起来像一个字典，其实是 JSON。但代码里不能直接拼字典，还是要把结构定义成类，将来由它转成 JSON 返回给前端。

按分包规范，先在 `app` 下建 `modules` 包（注意是包），再在 `modules` 里建 `product` 包，代表产品业务；然后在 `product` 里新建 `schemas` 模块。

#### 字段类型怎么定

响应类叫 `ProductResponse`，一共十二个字段，和数据库表字段完全一致：

```python
from decimal import Decimal
from typing import List

from pydantic import BaseModel

class ProductResponse(BaseModel):
    id: int
    name: str
    class_name: str
    category: str
    insurer: str
    image_url: str
    description: str
    min: Decimal
    max: Decimal
    target_group: str
    highlights: List[str]
    status: str
```

继承只留 `BaseModel` 一个，后面多余的基类删掉。

类型上有三处值得单独说：

- `id` 是 `int`，其余大多数字段是字符串；
- `min` 和 `max` 是数字，用的是浮点数。这里没有用 `float`，而是用了 `Decimal`——高精度浮点数。需要精确表示的小数用普通浮点数会带出误差，`Decimal` 能避免这个问题。以前没接触过也不要紧，这里照着用就行；
- `highlights` 在数据库里是数组，在 Python 里对应列表，所以写成 `List[str]`，表示一个装字符串的列表。

> **提示**：`Decimal` 和 `float` 的区别不在"能不能算"，而在"算得准不准"。凡是不能容忍舍入误差的数值，优先用 `Decimal`。

`schema` 到这里就定义完了。基础模块有了，后面 `model`、`repository`、`service`、`router` 才有东西可以依赖。

> **承前启后**：上一节讲完「从启动模块到第一个接口：FastAPI 项目的骨架与业务规范」，下一节接着讲「从表映射到路由：一条查询接口的四层落地」。

---

## 从表映射到路由：一条查询接口的四层落地
> 对应块：BLK12 | 覆盖分集：P46-P49 | 块标题（分集名）：《分层架构：模型、仓储、服务与路由》

schema 一收工，第二个基础模块就轮到 models 了。models 是什么？就是数据库表的映射类。SQLAlchemy 这层框架干的活本来就是跟数据库做关系映射，所以一个 models 类代表的就是一张数据库表，它需要跟表的结构完全一致——表里有哪几列、每列什么类型、多长，类里就得是那个样子。

表结构我前面已经贴过来了，跟我们看过的那张截图一模一样。理论上讲，你只要参考这个表结构，把类写出来就行了。

不过这活儿还是稍微有点麻烦，因为 DeclarativeBase 这一套不只是把字段定义出来，还要把这些字段跟数据库做映射。这些代码你记不住——也不用去记。你只需要找到 SQLAlchemy 的官网，里面肯定会有一篇示例。示例里第一步永远是先定义一个基类，叫 `Base`，所有表映射类都去继承这个基类：它这边有个 `User` 继承 `Base`，下边 `Address` 也继承 `Base`。类里边再写字段映射，`id` 映射 `id`，`name` 映射 `name`，`full_name` 映射 `full_name`，还有 `address` 字段等等。

定义的时候，除了字段名，还有字段的数据类型，最关键的是 `mapped_column` 后面那一串东西：这个字段是不是主键、数据类型是什么、长度多少。这里的 API 非常多，你肯定记不住。

所以我也不建议你去手敲。像这种代码，一定是随便找一份示例——官网上的也行，你以前项目里的也行——复制过来，然后参考自己的字段一个一个改：它这里 `id`、`name` 跟我们一样，不用改；它叫 `full_name`，我们叫 `class_name`，那就改成 `class_name`；类型我们是字符串，那就写 `str`；后面 `mapped_column` 也是 `String`，把对应的长度写上去。

> **结论**：表映射类属于"固定代码"，靠"复制一份示例 + 逐字段替换"得到，不靠记忆。

不过这里有两点、三点需要你们注意，我一个一个说。

### 统一基类：为什么必须搬到 common

先看第一点。仔细观察，`User` 和 `Address` 都要继承这个 `Base`。那么这个 `Base` 将来写在哪里？

别忘了我们的项目遵循的是业务分包的方式：`app` 下的 `modules` 里有 `chat`、`conversation`、`rag`、`user` 这些业务模块，每个模块里都有自己的 models。`user` 的 models 放 `user` 下，`chat` 的 models 放 `chat` 下，但它们都要继承 `Base`。

那 `Base` 放哪？你如果放到 `user` 里面，`chat` 这个模块就没办法用了；如果放到 `chat` 的 models 里，`user` 又用不了。

所以这个 `Base` 是所有业务模块都要用的统一基类，是一个通用的东西，就不能放在具体业务模块里。放哪儿呢？放到 `common` 包下。然后不管是 `chat` 还是 `user`，都去 `common` 里找那个基类，继承它就可以了。

```text
app/
├── common/                  ← 谁都要用的东西放这里
│   └── models.py            Base / CreatedAtMixin / UpdatedAtMixin
├── infrastructure/
│   └── database.py          AsyncSessionFactory / get_session
└── modules/                 ← 业务分包，一个业务一个目录
    ├── chat/
    │   └── models.py        继承 common 里的 Base
    ├── user/
    │   └── models.py        继承 common 里的 Base
    └── product/
        ├── schemas.py
        ├── models.py
        ├── repository.py
        ├── service.py
        └── router.py
```

图里这条线是关键：业务模块之间横向互不依赖，公共的东西一律沉到 `common`，方向只从上往下。

统一基类的作用也就两条：第一，标记 SQLAlchemy 的表映射类；第二，把统一的函数写在里面。

### 混入类：公共字段的第二种去处

第二点，`User` 也好、`Address` 也好，都需要定义一个魔法函数 `__repr__`。它的作用是：当你去打印 `user` 或者 `address` 这样的对象时，能有一个友好的输出，而不是一串看不出内容的内存地址。

像这种模板函数，每一个类都要写一遍，是不是也挺麻烦？所以我们同样可以把它放到基类里定义。这样一来，不管是 `User` 还是 `Address`，都继承了 `Base`——你叫了爸爸不是白叫的，爸爸里面定义的函数，这些儿子们是不是都能用？

> **提示**：`__repr__` 是 Python 的"对象自述"钩子，`print(obj)`、调试器、日志里都会调它。放在基类里一次，所有表映射类都自动拥有可读的打印结果。

第三点，这个可能不太容易理解。看 `products` 这张表，里面有两个字段：`created_at` 和 `updated_at`，就是创建时间和更新时间。你去看一眼就会发现，很多很多的表都有这两个字段。上面那些是每张表的特有字段，但创建时间、更新时间真的是非常常见的字段。

翻一下库里的表：`user` 表有没有 `created_at` 和 `updated_at`？有。再往下这张表有没有 `created_at`？有，但它没有 `updated_at`。再往上一张有没有 `created_at`？也有。所以库里有非常多的表，会有 `created_at` 或者 `updated_at`，或者两者都有。

那将来定义表映射类的时候，每个类都得加这两个字段，这不就是公共字段吗？怎么处理？

注意，这回不要往基类里放了。因为你往基类里一放，他们一继承，意味着**所有**的映射类都有了 `created_at` 和 `updated_at`——可刚才我们明明看到，有些表有，有些表没有。

于是我们换一个做法：除了 `Base` 之外，再定义一个类，这个类里就放一个字段 `created_at`；谁想要这个字段，就继承我，从我这儿把它继承过去。相当于这些映射类不要自己再写创建时间和更新时间，而是通过继承来获取。

像这样的类也有个名字，叫**混入类**，英文 `Mixin`。名字听着有点奇怪，作用其实很简单：帮别人混入一些字段。其他映射类只要继承了它，相当于就继承到了对应的字段，代码复用性就上来了。

| 类 | 放什么 | 谁继承 | 为什么不能合并 |
| :--- | :--- | :--- | :--- |
| `Base` | 表映射的统一标记 + 通用函数（如 `__repr__`） | 所有表映射类，无例外 | 一旦并入时间字段，所有表都被迫带时间列 |
| `CreatedAtMixin` | 只放 `created_at` | 需要创建时间的表 | 有的表只有创建时间 |
| `UpdatedAtMixin` | 只放 `updated_at` | 需要更新时间的表 | 有的表两者都没有，也有两者都有的 |

所以 model 的开发顺序不是"一上来就照抄官方示例、把字段一个个改成我们的、整出 Product 的映射类"，而是先做出统一基类和混入类，再来开发具体的映射类。

### common 下的三个类

新建一个包叫 `common`，然后在 `common` 里再新建一个 Python 文件叫 `models.py`。注意它不是某个具体业务下的 model，而是一个公共的 model。

这里面我们写三个类，一个是基类，两个是混入类：

```python
from datetime import datetime

from sqlalchemy import DateTime, func
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column


class Base(DeclarativeBase):
    """所有数据库表模型类的统一基类。"""

    def __repr__(self) -> str:
        return f"<{type(self).__name__} {self.__dict__}>"


class CreatedAtMixin:
    """需要创建时间的表，继承我。"""

    created_at: Mapped[datetime] = mapped_column(
        DateTime, default=func.now()
    )


class UpdatedAtMixin:
    """需要更新时间的表，继承我。"""

    updated_at: Mapped[datetime] = mapped_column(
        DateTime, default=func.now(), onupdate=func.now()
    )
```

`Base` 继承 `DeclarativeBase`，这就是 SQLAlchemy 的统一基类，将来所有的数据库表模型类都要继承它。

两个混入类里，`mixin` 就是"混合"的意思：其他类继承了我，就有了我里面的东西。

`CreatedAtMixin` 里只有一个 `created_at` 字段，类型是时间，映射到数据库里也是时间。它还有默认值——有默认值的意思是，将来你往表里新增一条数据时，这个字段你根本不用管，它会自动往里面加，加的值就是 `func.now()`，也就是当前时间；这个函数是 SQLAlchemy 自带的。

`UpdatedAtMixin` 也是如此，字段叫 `updated_at`，类型是时间，也有默认值。但它除了默认值还有一个 `onupdate`：当你修改这张表里任意字段时——比如你改了 `category`，或者改了 `status`——这个 `updated_at` 都会自动更新。

> **结论**：这两个时间字段，将来你都不用再管了。往里插数据，它自动生成；修改任意字段，更新时间自动刷新。

这就是两个混入类的字段。那将来我们写一个 `Product` 类，这两个字段就不用写了，只要继承这两个混入类，就自带了。如果某张表只有 `created_at`、没有 `updated_at`，那就只继承第一个。

### Product 映射类

到这里，基础模块、也就是通用模块就搞定了。下边是 `product` 自己的 models：写一个类，继承 `common` 里的 `Base`，然后照着 `products` 表里的字段一个一个定义。

```python
from decimal import Decimal

from sqlalchemy import Integer, Numeric, String
from sqlalchemy.dialects.postgresql import ARRAY
from sqlalchemy.orm import Mapped, mapped_column

from app.common.models import Base, CreatedAtMixin, UpdatedAtMixin


class Product(Base, CreatedAtMixin, UpdatedAtMixin):
    """products 表的映射类。"""

    __tablename__ = "products"

    id: Mapped[int] = mapped_column(Integer, primary_key=True)
    name: Mapped[str] = mapped_column(String(64))
    class_name: Mapped[str] = mapped_column(String(64))
    category: Mapped[str] = mapped_column(String(32))
    # 其余列与 products 表一一对应：保险图片、描述、
    # 保额上下限、保障人群、status 等
```

继承了三个类：`Base` 是统一基类，两个混入类给我提供创建时间和更新时间字段。

表名是 `products`，对应到我们的数据库表——这个地方千万别写错了，回头看一眼那张表，它就叫 `products`。

下边就是表里的字段。时间字段我没写，从混入类继承；其他字段跟数据库完全一致：`id`、`name`、`class_name`、`category`，还有图片、描述等等。

几个类型对应关系记一下就够了：

| 数据库里的列类型 | 类里的写法 | 说明 |
| :--- | :--- | :--- |
| 整型主键 | `Mapped[int]` + `Integer, primary_key=True` | 主键信息写在 `mapped_column` 里 |
| 字符串 | `Mapped[str]` + `String(长度)` | 长度照表结构抄，不用加 `Optional` |
| `NUMERIC` | `Mapped[Decimal]` + `Numeric` | 保额上下限用 `Decimal`，不用 `float` |
| `ARRAY` | `Mapped[list]` + `ARRAY` | 保障人群是一组值，Python 侧就是 `list` |
| 时间（`TIMESTAMP`） | 不写 | 继承混入类即可 |

`status` 是字符串，并且带有默认值。到这里，这张表的映射类也搞定了，将来增删改查就都基于它。

> **易错点**：`NUMERIC` 一定要映射成 `Decimal` 而不是 `float`。浮点数存金额会在末位飘，`Decimal` 才是定点十进制，跟库里的 `NUMERIC` 语义一致。

回到文档，这一节要你做的就是两件事：第一，建 `app/common/models.py`，写公共的表映射基类和两个混入类；第二，定义 `app/modules/product/models.py`，写 `products` 表的映射类。动手做一下。

### 写 repository 之前，先回数据库把 SQL 敲出来

schema 和 model 准备好了，就可以正式进业务流程的开发了。正如前面所说，我们从最底层的 repository 开始，再往上做 service 和 router。

repository 负责的就是数据库操作——增删改查，别的什么都不管，你就只做数据库。

那我们当前这个查产品列表的接口，需要什么样的数据库操作？前面分析接口的时候都跟大家说过了：这个接口要做的事就是查询产品列表，查询的时候有一个条件 `category`（产品分类）。这个条件不是必须的，可以有也可以没有：没有，就查全部的有效产品；有，才查指定分类的有效产品。

所以这里边还有第二个条件：这个产品是否有效，也就是表里 `status` 字段的值是不是 `active`。综上所述，这个接口是个查询接口，将来要执行的就是一条数据库查询语句，条件有两个：`status` 必须有，`category` 可选。

到这里，repository 要做的事情就分析清楚了——一条查询语句。

那怎么落笔？我的建议是分两步走：**先回数据库把这条 SQL 语句写出来，再把 SQL 转成 SQLAlchemy 的代码**。以后你写任何增删改查都按这个流程来，先写 SQL，再写代码。

在数据库客户端里新建一个查询。它很智能，因为我刚打开的是 `products` 表，它自动给我建了 `select * from public.products`。这里的 `public` 指的是数据库默认的 schema，不用管它，直接写 `products` 就行。

先加第一个条件：

```sql
SELECT * FROM products WHERE status = 'active';
```

现在我们库里提供的所有商品，`status` 其实都是 `active`，但将来也有可能是非 `active`，也就是下架的产品，所以第一个条件一定要加上。

再补第二个条件：

```sql
SELECT *
FROM products
WHERE status = 'active'
  AND category = 'life'
ORDER BY id ASC;
```

`category` 有四种，用户可能会传其中一种。比如传 `life`，那就是寿险，执行一下，查到的就只有寿险了；把 `life` 换成 `medical`，再执行一次，查到的就全是医疗险；换成 `accident`，就是意外险。

这就是我们最终想要的效果。把 SQL 留着，接下来干的事就是把它翻译成 SQLAlchemy。

### repository：用一个类管住增删改查

在 `product` 业务包下新建一个 Python 文件，叫 `repository.py`。`repository` 这个单词指的就是持久化，讲的就是数据库这类操作。

写之前先说组织结构：不要一上来就摊开写增删改查。项目里的代码往往要有一定的组织方式——repository 里面将来写增删改查肯定不止一个，当前这个接口写一条，后面再来别的接口还要接着写，功能多了方法就会很多。

所以我的建议是：把这些方法写在一个类里，在类里整理和管理，对外用面向对象的思想让别人去调。

类名就按"业务名 + 层职责"来起，叫 `ProductRepository`，也就是产品的持久化操作、CRUD 操作。

类里第一个方法，作用是"根据产品的分类来查询产品"，那函数名就叫 `query_by_category`——见名知义。既然是按分类查，它一定接收一个参数 `category`，前端会把这个分类值传进来，类型是字符串；当然它也有可能是个 `None`，也就是前端压根没传。

```python
class ProductRepository:
    def __init__(self, session: AsyncSession):
        self.session = session

    async def query_by_category(
        self, category: str | None = None
    ) -> list[Product]:
        ...
```

### session 让别人递进来

SQLAlchemy 要做查询，需要用到 `session`。现在这个地方有没有 session？没有。

那怎么获取？有人说了：原来在 `app/infrastructure/database.py` 里不是有个 `get_session` 吗，调它不就把 session 拿到了？

没错。但前面讲过的规矩别忘了：`get_session` 一般是在 router 里靠依赖注入来注入的，不要直接去调。所以在 repository 这一层，非常不建议你手动获取 session，而是将来 router 调 service、service 调 repository 的时候，让上层把 session 传给我。

那在哪儿接收？不是在这个方法的参数列表里接收。因为这个类里将来定义的函数肯定不止一个，每一个函数都有可能要用 session，每个函数都自己定义一遍，那就多余了。

我们的做法是通过构造函数 `__init__` 来接收，然后把它保存到 `self` 上，作为一个类的成员变量。这样下边所有的函数都能用这个成员变量，同时也不用关心 session 是从哪来的——我只要接收就行。

session 的类型就是之前一直在用的 `AsyncSession`。这个包要导进来：一方面类型注释得写对，另一方面，导了包之后你写代码才会有提示；不导，编辑器什么都不知道。

> **提示**：`get_session` 是给 FastAPI 的依赖注入用的，不是给 repository 手动调的。谁在上层，谁负责把 session 递下来——这是这一层的边界。

### select、where、order by 的三处对应

有了 session，查询就能落到代码上。`self.session.execute(...)` 执行一条语句，查询要用 `select`——它是 SQLAlchemy 提供给我们的函数，同样要导包。

`select` 里填什么？回想关系映射那三条对应：数据库里的**表**对应 ORM 里的**类**，表里的**列**对应类里的**属性**。所以 SQL 里的 `select * from products`，在 ORM 里就是 `select(Product)`——填表所对应的那个映射类。`Product` 就在我们刚写的 models 里，从当前包导入：`from .models import Product`，前面那个点表示当前包。

接着是 `where` 条件，也可以链式往后写：类名点属性名，就是 `Product.status == "active"`。注意 Python 里判等是两个等号，不要写成一个。

第二个条件是 `Product.category == category`，这个值就不要写死了，别人传进来的 `category` 是什么，它就是什么。

再往后可以继续挂排序：`order_by` 也是按某个字段排，同样用类名点属性名，`Product.id.asc()` 是升序，`desc()` 是降序。

```python
stmt = (
    select(Product)
    .where(Product.status == "active")
    .where(Product.category == category)
    .order_by(Product.id.asc())
)

result = await self.session.execute(stmt)
return result.scalars().all()
```

这条语句翻译过来就类似 `select * from products where status = 'active' and category = ? order by id asc`。写法跟我们在 AI 伴侣里用 SQLAlchemy 时完全一致，是不是没什么问题？

但是——代码写到这儿，问题就来了。

`where` 现在是不是等于写死了？两个条件都得成立。可你千万别忘了，`category` 是有可能不传的。不传的时候你还去加一个 `category == None` 的条件，那就出问题了。所以 `category` 的值应该做一个判断：有值时，才把它加进来作为条件；没有值，就不该加。

也就是说，这个 `where` 不是一个固定条件，而是一个**动态条件**。动态条件怎么写？

### 动态条件：把 where 拆成列表再解构

这块放到一个单独的 notebook 里验证，把条件单独拎出来看看到底长什么样。

先把刚才 `execute` 里接收的那条语句拷过来——`execute` 接收的是一个 statement，也就是一条 SQL 语句，就是中间这一段，`order by` 先不要，只留条件。

以前大家都是直接执行，从来没打印过。这次打印一下 `stmt`，看看它长什么样：

```text
SELECT products.id, products.name, products.class_name, products.category, ...
FROM products
WHERE products.status = :status_1 AND products.category = :category_1
```

`select(Product)` 就是指 select 这张表里的所有字段，`from products` 就是 from 这个类对应的表，后面 where 两个条件都在。所以现在这条是**固定 SQL**，不是动态 SQL。

怎么变成动态的？定义一个 `conditions`，条件的意思，把它定义成一个列表。

为什么是列表？因为 `where` 里边现在传了两个参数，这就是两个条件，那就可以把它们都放进列表里。这个列表里装的不是字符串、不是数字，装的是"条件"。

```python
conditions = [
    Product.status == "active",
    Product.category == category,
]

stmt = select(Product).where(*conditions)
```

关键在 `*conditions` 这个星号，它是解构：把列表整体拿过来一解，等于把列表里的每一个条件逐个放进 `where`。打印一下，效果跟直接写在 `where` 里一模一样。

一旦条件能放进列表，动态就好办了：必须的那个条件直接放进初始列表，`category` 则用判断决定要不要追加。

```python
conditions = [Product.status == "active"]

if category:
    conditions.append(Product.category == category)

stmt = select(Product).where(*conditions)
```

判断 `category` 有没有值，不用 `is None` / `is not None` 去写。Python 里非空就是真：有值就是 `True`，是空字符串或者是 `None` 就是 `False`。所以直接 `if category:` 就代表"有值"，有值才 `conditions.append(...)` 追加这条分类条件。

于是 `category` 这一条就不再是固定的了，它变不变成条件，取决于你传没传：传 `life`，生成的就是两个条件；传 `None`，where 里就只剩一个条件。

> **结论**：动态条件的套路只有三步——先建一个条件列表，按传参用 `if` 往里追加，最后在 `where` 里用 `*` 解构铺开。

回到 repository，代码的落笔顺序也跟着变了：一上来不是直接查询，而是先组织动态条件——第一个条件是产品必须是上架（激活）的，下架的产品不要；第二个条件是如果 `category` 不为空，再追加分类查询条件。条件组织完，再去执行查询。

### 顺手省一步

笔记上对这个查询逻辑做了个简化。原来的链路是：

```text
execute(stmt) → result.scalars().all()
```

笔记里省掉了 `execute` 那一步，直接把 `stmt` 交给 `scalars`：

```python
products = await self.session.scalars(stmt)
return products.all()
```

也就是说，正常情况下你应该先执行、拿到 result 之后再做一个 `scalars` 收集、得到列表；现在 `scalars` 可以直接接收语句，所以省了一个步骤，调完 `scalars` 下边直接 `.all()` 就行，不用再 `scalars()` 一次。

道理完全一样，效果也完全一样，以后就用这个新写法。整个 repository 的方法最终长这样：

```python
from sqlalchemy import select
from sqlalchemy.ext.asyncio import AsyncSession

from .models import Product


class ProductRepository:
    def __init__(self, session: AsyncSession):
        self.session = session

    async def query_by_category(
        self, category: str | None = None
    ) -> list[Product]:
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

这里要 `await`，因为 PostgreSQL 的驱动是异步的，查询本身是个异步操作；对应的函数也得是 `async def`。查询多条用 `scalars`，它的作用是把结果集收成一列实体，再 `.all()` 转成列表。

返回值就是这个列表，里面装的是 `Product`。

### 在 notebook 里先跑通

写完就测。测试还在 notebook 里做，因为要创建 repository 对象，就得给它一个 session。

session 从哪来？`database.py` 里确实提供了 `get_session`，但这个地方同样不要调它——它还是留给 router 的依赖注入用。现在不在 FastAPI 的运行环境里，就手动通过 session 工厂来拿：

```python
from app.infrastructure.database import AsyncSessionFactory
from app.modules.product.repository import ProductRepository

async with AsyncSessionFactory() as session:
    repo = ProductRepository(session)
    products = await repo.query_by_category("life")

    for product in products:
        print(product)
```

传 `life` 查寿险，跑出来三条，这三条的 `category` 都是寿险，没问题。

再把参数换成 `None`——注意这个地方不能不传，它得是个"空"，那就传 `None`：结果库里 12 条产品全查出来了，id 从 1 到 12，一条不落。

既然调用方经常会不传，那就给参数一个默认值 `category: str | None = None`，这样不传也能跑，代码更健壮。改完记得重启 notebook 的内核，从头执行一遍。

这一层的两个关键点：第一是动态条件的写法，以前没写过——把条件放进列表，按传参不同动态往里加；第二是查询从"`execute` 再 `scalars`"变成"直接 `scalars` 再 `all`"，省一步。

### service：业务很薄，地基要留

repository 完成以后，进入 service，也就是业务层。

service 作为业务层，主要负责组织业务流程，有必要的话去调 repository 完成数据库的增删改查。注意是"有必要的话"——有些业务可能根本不用碰库，直接在业务层就做完了。

我们当前这个业务是查询产品列表，所以肯定要调 repository。而且因为仅仅是一个查询，没有其他逻辑，理论上这一层的代码其实没有什么特殊的东西，属于一个非常简单的业务层。

在 `app/modules/product/` 下新建 `service.py`，同样写一个类。service 层将来也会有很多不同的业务功能，会封装不同的函数，所以跟 repository 一样，把方法整理在类里。命名沿用"业务名 + 层"：`ProductService`。以后你写用户管理相关的模块，就是 `UserService`、`UserRepository`，一路套下去。

函数就叫 `list_products`，查产品列表的意思，参数还是 `category`。

注入的问题：不管业务再简单，你要调 repository，就得先有它。而这个注入同样不要写在方法里——现在写在方法里，以后方法多了，每个方法都要来一遍。跟 repository 一样，用构造函数注入。

但构造函数接收的不是 repository，我们只要 `session`：repository 我们自己就能创建，有了 session，创建 repository 对象还不是一句话的事。

```python
from sqlalchemy.ext.asyncio import AsyncSession

from .models import Product
from .repository import ProductRepository


class ProductService:
    def __init__(self, session: AsyncSession):
        self.session = session
        self.repository = ProductRepository(session)

    async def list_products(
        self, category: str | None = None
    ) -> list[Product]:
        return await self.repository.query_by_category(category)
```

除了把 repository 存成类里的变量，session 也要记下来。为什么？因为后续如果有增删改的业务，业务比较复杂的情况下，一个 service 里可能要调用多次数据库写操作。这种时候就需要在 service 层统一管理事务，把多次写操作归到同一个事务里；而要开启事务，就得用到 session。

> **结论**：repository 只管单次怎么读写库；"这一串写操作算不算一件事"，由 service 决定。事务的边界在 service，不在 repository。

当前这个业务简单，没有事务，所以方法体就是一句话：`return await self.repository.query_by_category(category)`。返回值还是列表，里面装 `Product`，类型注释同样要导包。

### router：照接口文档声明接口

最后是 router 层。它负责接收前端的请求、校验前端传过来的参数，并且封装最终的响应结果发给前端；中间要查数据，就调 service。

router 要做的事就三件：接收、校验、返回。而怎么接收，要严格按接口文档来——文档规定这个接口的方式、路径是什么样，你就得照着声明。

我们这个接口：请求方式是 `GET`，路径是 `/api/v1/products`，请求参数只有 `category` 一个。正因为只有一个参数，当时我们没有定义 schema，直接在方法的参数列表里接收就行。

响应格式是一个列表，里面装 `ProductResponse`。这个讲得比较早了，可能有点忘。接口文档规定返回的是查询到的产品列表，最终要以 JSON 返回；而产品的字段就是数据库里的那些字段——**但没有 `created_at` 和 `updated_at`**。所以当时封装了一个对应的 schema，在 `app/modules/product/schemas.py` 里，叫 `ProductResponse`，去掉这两个时间字段，剩下的都有。

在业务包下新建 `router.py`。这里要注意：在 router 里是看不到 `app` 的，不能像在 `main.py` 里那样直接 `app.get`。所以要先单独定义出一个 router，将来在 main 里挂载。

```python
from fastapi import APIRouter, Depends
from sqlalchemy.ext.asyncio import AsyncSession

from app.infrastructure.database import get_session
from app.modules.product.schemas import ProductResponse
from app.modules.product.service import ProductService

router = APIRouter(
    prefix="/api/v1/products",
    tags=["产品管理相关接口"],
)


@router.get("", response_model=list[ProductResponse])
async def list_products(
    category: str | None = None,
    session: AsyncSession = Depends(get_session),
) -> list[ProductResponse]:
    service = ProductService(session)
    return await service.list_products(category)
```

`APIRouter` 是 FastAPI 提供的，直接导。它有几个参数：

第一个是 `prefix`，统一的路径前缀。因为这个 router 里将来肯定不止一个接口，跟产品有关的可能有好多个，它们会有统一的路径前缀。我们的业务遵循 RESTful，所以查询路径的前缀其实都是统一的 `/api/v1` 再加上资源名称 `products`；不管是增删改查，将来都用这个前缀，所以把它抽到 `prefix` 里，个别接口路径不一样，在装饰器里再补。

第二个是 `tags`，一个列表，是对当前这组接口的说明。写上"产品管理相关接口"，将来 FastAPI 动态生成的接口文档里就能看到这个分组说明。

接下来用装饰器 `router.get` 定义请求方式，`get`、`post`、`delete` 都可以，我们这里是查询，所以是 `get`。装饰器里第一个是位置参数，就是这个接口的路径。这次路径跟统一前缀是同一个，所以值不写，但位置参数不能省，空在这里就行，写成 `""`。后面的参数都是可选的，其中 `response_model` 建议写上——我们要的格式是列表，里面装 `ProductResponse`。

函数本身声明成异步函数。参数就 `category` 一个，字符串，也可以是 `None`，默认给 `None`，这样前端可传可不传。

到这里，请求方式、请求路径、请求参数、响应格式，全部声明完毕，一个 FastAPI 接口的声明就完成了。

实现只有两步：第一步创建 service，第二步执行查询。

创建 service 需要 session。这时就轮到 `database.py` 里的 `get_session` 了，它就是给 FastAPI 用的。但不是我们自己调——还记得依赖注入吧：需要什么，就在参数列表里写什么。我现在需要 session，类型是 `AsyncSession`，它不是我自己创建，而是 `Depends(get_session)`，依赖来源于 `get_session` 这个函数。

FastAPI 在执行时会发现你需要 session，并且看到你是基于 `get_session` 来取的，它就去调 `get_session`，把 session 交出来，代码再走到我们这里。我们拿到 session，传给 `ProductService(session)`，然后 `await service.list_products(category)`，把结果返回。

> **提示**：`get_session` 是靠 `yield` 把 session 交出来的依赖，请求进来时打开、请求结束时关闭，中间这段生命周期由 FastAPI 管。所以它只能出现在 `Depends(...)` 里，不能像普通函数那样随手调。

细心的同学会发现一个问题：`list_products` 返回的是 `list[Product]`，可这里要的是 `list[ProductResponse]`，这能行吗？

没问题。FastAPI 很强大：它执行的时候看到你要的是 `ProductResponse`，会自动把 `Product` 转成 `ProductResponse`——转换时按目标模型要的字段，去源对象里找，有就转过来，没有就算了。`ProductResponse` 的字段跟 `Product` 相比大部分一样，只少了 `created_at` 和 `updated_at`，那转换的时候正好把这两个丢掉。所以你完全不用操心，只管查询，转换交给 FastAPI。

### 挂载、联调，再回头看这条路线

现在这个 router 还没生效，因为它没被挂到应用上。到 `main.py` 里注册：

```python
from app.modules.product.router import router as product_router

app.include_router(product_router)
```

导入的时候起个别名 `product_router`，因为将来 router 会有很多个，不起别名容易冲突。

启动项目，初始化都成功。打开浏览器，先看接口文档：刷新一下，除了 `default` 之外，上面多了一组"产品管理相关接口"，就是 `/api/v1/products`，在这里就能直接完成查询。

再回前端刷新页面：所有产品都查到了。做一个过滤，查医疗险、查重疾险、查寿险、查意外险，都没有问题。接口开发完成。

回头看看整个接口是怎么做出来的——它是**倒过来**开发的：

先是按接口文档，确定请求方式是 `GET`、请求路径是 `/api/v1/products`、请求参数是 `category`、返回值是 response 的列表；而这个 response 的信息其实就是数据库表信息，所以第一步先做 schema，严格照文档要求定义其中每一个字段。

第二步，定义数据库表的映射类 models：统一的基类 `Base`，两个混入类，然后照表写 `Product` 的映射模型，字段一一对应，只有两个时间字段靠继承获得。

第三步，往下编写增删改查：定义 repository。由于 `category` 是动态条件，可以有可以没有，所以先做动态条件组装，然后才是最终的查询实现。

第四步，service 层调 repository，完成一次列表的批量查询，没有其他逻辑。

第五步，router 层按接口文档声明、定义路径前缀、定义整个接口、调 service 返回结果。

最后在 main 里注册 router，到前端测试，一切正常。

```text
浏览器 / Swagger 文档
        │  GET /api/v1/products?category=life
        ▼
┌────────────────────────────────────────────────┐
│ router      modules/product/router.py           │
│   收请求 · 声明参数与响应格式 · 转调 service      │
└────────────────────────────────────────────────┘
        │  ProductService(session)
        ▼
┌────────────────────────────────────────────────┐
│ service     modules/product/service.py          │
│   组织业务流程 · 事务边界放在这一层               │
└────────────────────────────────────────────────┘
        │  ProductRepository(session)
        ▼
┌────────────────────────────────────────────────┐
│ repository  modules/product/repository.py       │
│   只做增删改查（动态条件就在这里组装）            │
└────────────────────────────────────────────────┘
        │  select(Product).where(*conditions)
        ▼
┌────────────────────────────────────────────────┐
│ models      modules/product/models.py           │
│   products 表映射 · 字段与列一一对应              │
└────────────────────────────────────────────────┘
        │
        ▼
    PostgreSQL
```

读这张图要注意两个方向：请求从上往下走，一层只认下一层；而 session 是从上往下**递**下来的——router 用 `Depends(get_session)` 把 session 拿到手，然后作为构造参数传给 service，service 再传给 repository，越往下越不关心它从哪来。

| 层 | 文件 | 只负责这一件事 | session 从哪来 |
| :--- | :--- | :--- | :--- |
| router | `router.py` | 收请求、校验参数、封装响应 | `Depends(get_session)` |
| service | `service.py` | 组织流程、管事务边界 | 构造函数接收 |
| repository | `repository.py` | 增删改查 | 构造函数接收 |
| models | `models.py` | 表结构映射 | 不涉及 |

再补一句整体的感受：越往下越不知道"业务"，越往上越不碰 SQL。repository 里只有 `select`、`where`、`order_by`，它压根不知道什么叫"上架的产品列表"；router 里只有参数和响应模型，它压根不知道数据是 join 出来的还是缓存里取的。这条界限守住，后面再加接口就是照着这个模子往里填。

这一整条链路跑通了，你自己动手再走一遍，比看十遍都管用。

> **承前启后**：上一节讲完「从表映射到路由：一条查询接口的四层落地」，下一节接着讲「依赖注入链、Pydantic 转换与会话管理」。

---

## 依赖注入链、Pydantic 转换与会话管理
> 对应块：BLK13 | 覆盖分集：P50-P52 | 块标题（分集名）：《依赖注入与 Pydantic 进阶》

上一节我们把产品列表的接口做出来了，请求能通，数据也能返回。但有个地方不太对劲：router 里的那个 service，是我手动创建的。

### 把 service 的创建抽成依赖

只有一个接口的时候，手动创建没问题。可接口将来会越来越多——不光查产品列表，还会有别的接口，它们都要用到 service。每个接口里都写一遍创建 service 的代码，就有点多余了。

那能不能把「获取 service」这段代码也抽成一个方法？抽出来以后，谁需要 service，谁调这个方法拿，就不用每个接口都自己写一遍了。

抽法很直接：定义一个函数，名字叫 `get_product_service`，在函数里写创建 service 的代码，然后把它 `yield` 出去。

```python
async def get_product_service(session: AsyncSession = Depends(get_session)):
    service = ProductService(session)
    yield service
```

问题紧跟着就来了：创建 service 需要 session，session 从哪来？

同样依赖注入。所以在 `get_product_service` 里，我不自己去取 session，而是声明「我需要 session」，由 FastAPI 通过 `Depends(get_session)` 交给我。我们定义的函数需要 session，就继续往下依赖——这叫什么？这叫依赖注入的链。

这样一来，以后写业务，需要 service 就不要自己创建了，直接依赖注入：

```python
@router.get("/products")
async def list_products(service: ProductService = Depends(get_product_service)):
    ...
```

需要 service，就依赖注入 service；这个 service 是产品的 service，它从依赖注入来，依赖的是 `get_product_service` 这个函数。以后再有更多业务，同样道理，都不要再自己取 session、手动创建 service，统一通过这条链来拿。这么一改，就方便太多了。

### 依赖注入链是怎么跑起来的

这条链真正执行的时候，走的是一条「一层套一层」的路径。

```text
[前端请求]
    |
    v
list_products()           参数 service 声明为 Depends(get_product_service)
    |
    |  FastAPI 解析签名：这个接口需要一个 service
    v
get_product_service()     参数 session 声明为 Depends(get_session)
    |
    |  FastAPI 继续解析：这个函数需要一个 session
    v
get_session()
    |
    |  从会话管理器取 session（创建 session）
    |  yield session  --->  挂起，把 session 交给调用者（FastAPI）
    |                        FastAPI 把它赋给 get_product_service 的 session 参数
    v  （恢复执行）
get_product_service() 继续
    |
    |  把 session 传给 service，创建 service
    |  yield service  --->  挂起，把 service 交给调用者（FastAPI）
    |                        FastAPI 把它赋给 list_products 的 service 参数
    v  （恢复执行）
list_products() 拿到 service
    |
    |  执行查询逻辑，返回响应，函数结束
    v
get_product_service() 回到挂起点：没有后续，结束
    |
    v
get_session() 回到挂起点：上下文结束，关闭 session
```

从上往下读，是「FastAPI 一层层解析依赖、把资源造出来」的过程；从下往上读，是「每一层 `yield` 交出去的值原路返回，最后资源被回收」的过程。整条链的入口是一次前端请求，出口是 session 被关闭。

拆开看，一共七步。

第一步，前端请求进来，调用的是 `list_products`。

第二步，FastAPI 一看它的签名，这个地方需要一个 service，而这个 service 是基于依赖注入得来的、依赖于 `get_product_service` 这个函数。于是将来 FastAPI 就会帮我们来调用 `get_product_service`。

第三步，一调 `get_product_service`，它又发现：我这个函数依赖于谁？依赖于 session。session 又是哪里来的？它依赖于 `get_session`，而 `get_session` 就是从会话管理器里取 session 的那个函数。于是又去调用 `get_session`。

第四步，进入 `get_session`，创建 session，然后 `yield` 暂停执行，把 session 交出去。交给谁了？交给调用这个函数的人——也就是 FastAPI。等号右侧赋给左侧，这一交，`get_product_service` 这边就拿到了 `yield` 出来的 session。

第五步，session 到手，继续执行 `get_product_service`：把 session 传给 service、创建 service，再 `yield` 暂停，把 service 交出去。调用它的还是 FastAPI，拿到 service 以后注给等号左边，也就是 `list_products` 的参数。

第六步，`list_products` 拿到 service，执行自己的查询逻辑。

第七步，查询逻辑结束，`list_products` 返回，回到 `get_product_service` 刚才暂停的地方——它已经执行完了；再回到 `get_session` 暂停的地方，一结束，上下文结束，上下文结束就关闭 session。

> **提示**：`yield` 在这里是「借用与归还」的分界线。`yield` 之前是准备资源，`yield` 之后是回收资源；session 的关闭没人手动调 `close()`，是这层上下文结束时自动发生的。这也是为什么这里用 `yield` 而不是 `return`——用 `return` 交出去，函数当场就结束了，回头没有任何地方能把 session 关掉。

说到底还是那句话：把创建 service 的逻辑封装到一个函数里，只不过这个函数本身又依赖于 session，那么它就再去做一层依赖注入。`list_products` 依赖 service，获取 service 的代码又依赖 session，链式依赖，一层套一层。

### model 和 schema 是两套类

刚才那个查询接口里，我们做了一件事：把 service 查询的结果直接作为 `ChatResult` 里的 `items` 返回了。这里其实有问题。

从 service 里查出来的 products，是一个 list，里面装的是 `Product`——从数据库查回来的，类型是 model 里定义的那个 `Product`。前面讲过：数据库中放的是 model，前端要的是 schema。model 是数据库表映射类，schema 是前端需要的约束类，这两个不一样。

再看接口这一侧：我们调 service 的 query 方法，返回 products，这 product 是从数据库查回来的，所以它的类型是 model 中定义的那个 `Product` 类型；而前端要的 `ChatResult` 里的 `items`，人家想要的是 `ProductResponse` 类型。把 `Product` 的集合直接放到 `items` 上，等于把 `Product` 的集合赋给了 `ProductResponse` 的集合，这种做法其实是不对的。

可程序运行的时候并没有报错，看着好像可以。这是怎么回事？

因为 FastAPI 很强大：它发现你返回的是这个类型、它要的是那个类型，两边不相符，它就帮你把 `Product` 转成了 `ProductResponse`，做了一次自动转换。

> **提示**：FastAPI 帮你转的前提是「它知道你声明了什么」。响应模型是写进注解里的，FastAPI 在返回响应前按这个模型重新校验一遍，顺手就完成了从 model 到 schema 的字段搬运。所以那段代码不报错，不代表类型是对的，只是有人替你兜住了。

但我想说的是：在 FastAPI 里它能自动转，如果我们的场景不是 FastAPI 呢？那这种情况下你就必须得手动做这个转换了。

而手动做转换的时候，不好意思，它转不了。我们的 `Product` 类是一个普通类，`ProductResponse` 类是一个 Pydantic 的 BaseModel 类。一个普通类想转成 Pydantic 的 BaseModel 类，默认情况下不能直接转。

> **定义**：没有继承 `BaseModel` 的类，这里统称普通类；继承了 `pydantic.BaseModel` 的类，统称 BaseModel 类。在这门课的分工里，前者是数据库 model，后者是接口 schema。

### 把普通类对象转成 BaseModel

没有 FastAPI 帮忙自动转，我们就得自己转。要做两件事。

#### 第一步：声明从属性读取

给 BaseModel 类加一个属性：

```python
model_config = ConfigDict(from_attributes=True)
```

这句话的意思是：让 Pydantic 的 BaseModel 类对象能够从别的类、也就是普通类对象中读它的属性。`from_attributes` 字面就是「从属性读取」。一旦加上这个配置，它就能从普通类对象里读属性，把属性转到自己里边去，从而实现一次转换——把普通类对象转成 BaseModel 类对象。

但这里有一个前提：两种类型的属性名得是一样的。它读的是同名属性，名字对不上，就搬不过去。

我们回头看自己的两个类：`ProductResponse` 和 `Product`，字段名是 `id`、`name`、`class_name`、`category`，完全一致。所以在这种情况下，加上这段配置就能读了——告诉 BaseModel 说：你将来要从属性里读，由于这两个类的属性长得一样，它就会顺着属性名去读里边的值，然后交给 response。

#### 第二步：用 model_validate 完成转换

光加配置还不够，还要调一个函数：Pydantic BaseModel 里的 `model_validate`。

所有继承了 BaseModel 的类都会有这样一个工具函数，是个静态方法，作用是拿这个模型去做一次校验；你只需要把普通类对象传进去，它的返回值就会是一个 BaseModel 对象。这是新的 API，要记住它——转换就靠它。

我们在 notebook 里把这件事完整走一遍。先写一个 `UserResponse`，它继承 BaseModel，它就是一个 Pydantic 的 BaseModel 类，类似于项目里用来做请求和响应约束的 schema；叫这个名字，是假设将来要做用户查询，它就是查询用户的响应结果。

```python
class UserResponse(BaseModel):
    id: int
    name: str
    age: int
```

一开始先不加 `model_config`。再写一个普通类 `User`，普通类当然不继承 BaseModel。普通类要写三个属性，不能像 BaseModel 那样直接「字段名冒号类型」，得手写 `__init__`：

```python
class User:
    def __init__(self, id: int, name: str, age: int):
        self.id = id
        self.name = name
        self.age = age
```

两个类的属性名一样。创建普通类对象，传 `id=1`、`name="杰克"`、`age=21`，然后调转换：

```python
user = User(id=1, name="杰克", age=21)
print(UserResponse.model_validate(user))
```

第一次运行，报错了。报的是在做数据校验时出错：`UserResponse` 的输入必须得是一个字典，或者是 `UserResponse` 的一个实例（实例就是对象）。也就是说，你想得到一个 `UserResponse` 的对象，传进来的只有两种可能——要么是 dict，要么是 `UserResponse` 本身的对象。我们传进去的是一个普通类对象，不符合要求。

那就回到第一步，把这个配置加到 `UserResponse` 里：

```python
class UserResponse(BaseModel):
    model_config = ConfigDict(from_attributes=True)

    id: int
    name: str
    age: int
```

再执行，又报错了。这一次的原因不一样：`User` 这个类缺少成员，缺少对外获取成员的方法——这个类没法把字段值交出来，所以还做不了转换。

普通类如果只写 `__init__`，那些用来对外获取字段的魔法函数是缺的，得自己一个个补，很麻烦。

#### dataclass 补上缺失的魔法函数

这里换一种定义类的办法：`dataclass`。

```python
from dataclasses import dataclass

@dataclass
class User:
    id: int
    name: str
    age: int
```

一旦加上 `@dataclass` 装饰器，定义类就可以跟上边 BaseModel 一样写：字段名冒号类型，不用自己写 `__init__`，也不用写那些获取属性的模板函数。因为 dataclass 装饰器会自动提供 `__init__` 魔法函数，还有其他各种各样的魔法函数，都给我们准备好了。

这样的类仍然是普通类——判断标准就一条：没有继承 BaseModel，都叫普通类。dataclass 只是简化写法，不用我们手写一堆模板代码。

再试一次转换，成功了：打印出来的类型是 `UserResponse`，属性值也全都转过去了。

> **结论**：想把普通类对象转成 BaseModel 类对象，必须做两件事——给 BaseModel 类加上 `model_config = ConfigDict(from_attributes=True)`，然后用 `model_validate` 做转换。两个类的属性名还必须一致。

> **提示**：`dataclass` 以前没学过没关系。它在这里只是为了造一个「能被读出属性」的普通类来模拟真实场景，项目里用不到，知道有这么个东西、知道它会给类自动生成各种魔法函数就够了。效果上它和 BaseModel 很像——BaseModel 在定义时也没让我们写魔法函数，因为它继承之后就帮着生成了；dataclass 则是用装饰器生成。

### 给所有 schema 抽一个统一基类

道理清楚了，接下来是落到项目里：所有的 BaseModel 类都得加上这个配置。项目里有哪些 BaseModel 类？所有 schema 都是 Pydantic 的 BaseModel，都要加；`ChatResult` 也要加；将来还会写更多。每个类里都手动加一遍，麻烦。

那就定义一个统一的父类，也叫基类，在基类里加这个属性，让以后所有 schema 都继承这个基类，就不用在每个 schema 里手动加了。

在 `common/schema` 里定义它：

```python
class BaseSchema(BaseModel):
    model_config = ConfigDict(from_attributes=True)
```

这个 `BaseSchema` 一继承 BaseModel，它自己就是一个 BaseModel 类，配置加在它身上。

然后项目里所有 schema 不再直接继承 BaseModel，改继承 `BaseSchema`。这样它们间接继承了 BaseModel，仍然是 BaseModel 类、仍然拥有 BaseModel 的全部功能；同时又继承了 `BaseSchema` 上的这个配置，于是都具备了对象转换的能力。`ProductResponse` 一样改，以后不要继承 BaseModel，去继承 `app.common.schema` 里的 `BaseSchema`。

一句话收口：定义一个统一的基类，在它里边定义这个属性，就能让所有 BaseModel 类不仅可以从字典里取属性，还能基于类的字段来取属性——只要两个类的属性一样，就能把类 A 的属性转给类 B，实现自动转换。以后所有的类都去继承 `BaseSchema`，而不再直接继承 BaseModel。这就是 Pydantic 的这个拓展功能，后面要用到。

### 会话管理要解决什么问题

接下来进入新的内容：会话管理。先看前端页面。

首页展示的是一堆保险产品。但对普通人来讲，这些保险产品他看了以后，即便看到了，能知道它们是干什么的吗？不知道，所以他也不知道该怎么选。因此我们提供了一个功能：AI 保险顾问，帮用户来筛选产品。

不过可惜的是，保险顾问要想使用，必须先登录。其实一个正常项目里，一定会有登录注册的功能，我们这个登录已经模拟好了，也算是个假登录——只要点了登录，就登录成功。

登录成功以后，去聊天、去新建咨询，前端都会把这个用户的信息发送到服务端。这个请求里有一个参数，叫 `x-user-id`，就是用户的 id，默认写死成 1。将来后端拿到这个请求头，就能知道当前登录的用户是谁了。学过 HTTP 请求的话，请求头的概念应该不陌生，这就是请求头——前端相当于把用户给写死了，就固定是 id 为 1 的用户。

所以登录功能咱们不用实现，接下来重点实现智能保险顾问的功能。

作为一个保险顾问，其实就是一个聊天的客服。跟客服聊天，就要有对话的管理功能：可以创建会话——点「新建咨询」就是在创建一个会话；创建完以后，在会话里跟顾问聊天；过一段时间，还可以再新建新的会话，再做聊天。将来这里就会出现很多会话，形成一个列表，这就是会话管理。

会话管理的功能主要有哪些？新增会话、查询会话列表、删除会话，还有重命名会话——会话的名字通常也要能改。这些功能就是接下来要实现的。之前做 AI 伴侣的时候做过类似的，应该比较熟悉，这次在项目里再做一遍，顺便把 FastAPI 再熟悉一下。

这里有个说法要统一。在当前这个智能客服的会话管理里，每一个用户都可以同时拥有多个会话，而每一个会话都会用一个唯一的 id 来区分。这个 id 有一个专业的称呼，叫 `thread_id`。以前做 AI 伴侣的时候叫 `session_id`，名字不一样，道理是一样的，都是指一个会话，都是会话的唯一标识。后续跟 AI 对话时，就要用到这个会话 id。

会话管理其实分成两部分：

| 部分 | 内容 | 本期安排 |
| :--- | :--- | :--- |
| 会话元数据 | 会话的标题、所属用户、创建与更新时间等信息，也就是左侧那个会话列表 | 本次实现 |
| 会话历史 | 会话里一条一条的消息，用户一条、AI 一条 | 先不做 |

所以这次先做的是会话列表的管理，也就是会话元数据的管理。四个功能：创建会话、查询会话、重命名会话、删除会话。

### 创建会话接口的请求与响应

实现任何一个接口，都要参考接口文档来做。会话管理属于智能客服相关的功能，不属于保险业务功能，所以要看智能客服的接口文档，里面有一组会话管理的接口，第一个就是创建会话：为当前用户创建一个新的对话会话。

| 项目 | 内容 |
| :--- | :--- |
| 请求方法 | `POST` |
| 请求路径 | `/api/v1/chat/threads` |
| 是否需要登录 | 必须登录 |
| 请求体字段 | `title` |

请求体字段为什么只有一个 `title`？因为创建一个会话，会话里的信息主要就是名字。点新建会话的时候，界面上其实就是一个名字，别的没有。所以新建会话最关键的就是起个名；剩下的「这个会话属于谁」，从请求头里的用户 id 就知道了。因此前端提交的就只有标题本身。

> **易错点**：这个字段不是拼在问号后面的查询参数，而是放在请求体里。以前的接口是在路径后面拼接参数，这个接口是在请求体中发送——这是不一样的地方。

什么叫请求体，打开浏览器的 Network 面板就清楚了：点一下新建会话，发出一个请求，能看到请求方式、请求路径，而请求体是放在 payload 里的，以 JSON 风格传递。前端发 JSON，后端就要用一个对象来接收 JSON，对应的就是后边那个 schema 对象。

响应结果里有四个字段：

| 字段 | 含义 |
| :--- | :--- |
| `thread_id` | 新建会话的唯一标识，后面跟 AI 对话时要用到，也就是后面学 LangGraph 时会遇到的那个 `thread_id` |
| `title` | 把创建时提交的标题原样返回 |
| `created_at` | 创建时间 |
| `updated_at` | 更新时间 |

最终也是 JSON 格式：有 id、有标题、有两个时间。

到这儿，这个接口就看完了：请求怎么发、字段放哪里、响应长什么样，都清楚了。接口弄明白之后，接下来才是照着它去开发——先分析，再动手。

> **承前启后**：上一节讲完「依赖注入链、Pydantic 转换与会话管理」，下一节接着讲「会话创建：表结构、映射类与三层业务代码」。

---

## 会话创建：表结构、映射类与三层业务代码
> 对应块：BLK14 | 覆盖分集：P53-P56 | 块标题（分集名）：《会话创建：数据模型与业务分层》

创建会话这个接口，请求方式、请求路径、请求参数、响应结果，前面已经一条条过完了。接口长什么样，心里已经有数了，接下来就该动手把它做出来。

写一个接口，不管它是哪门子业务，永远都是那几个部分：Model、Schema、Repository、Service、Router。这五个部分合起来，才构成一个完整的业务模块。

那现在写的是什么业务？会话管理。既然不是产品管理，就不能再往 `product` 里面塞了。回到开发工具，在 `modules` 下面新建一个包，包名叫 `chat_thread`——`chat` 是聊天，`thread` 在这里就是会话。它跟 `product` 是平级的，一个是产品相关业务，一个是会话管理业务。

```text
app/modules/
├── product/                产品相关业务
│   ├── models/
│   ├── schemas/
│   ├── repository/
│   ├── service/
│   └── router/
└── chat_thread/            会话管理业务
    ├── models/
    ├── schemas/
    ├── repository/
    ├── service/
    └── router/
```

新的业务模块照抄 `product` 的骨架，五样东西一个不少，业务代码自己重写。

先做哪一块？先做 Model 和 Schema。因为这两块是基础，后面的业务全都要用它们；而且这次要写的是表的增删改查，表长什么样都还没定，增删改查就无从谈起。所以顺序是 Model → Schema → Repository → Service → Router。

### 会话表该有哪五个字段

表长什么样，取决于一个会话需要记住哪些信息。

平时用对话产品，左边那一栏一条一条的，每一条就是一个会话。新建一个会话，第一眼看得到的是什么？标题。然后是它什么时候建出来的，要有创建时间。再然后，每个会话都得有自己的一串编号。

为什么光靠标题不够？你们之前那个 AI 伴侣项目里，区分不同会话靠的就是标题——这不准。数据库设计里，每张表都该有主键，区分不同的数据、不同的对象，最靠得住的永远是 id。所以每一个会话都必须有自己的 id。

打开对话产品，随便点一个会话，地址栏里那一长串就是会话 id；点不同的会话，那一串就跟着变。将来用户点开某一个会话，后台是拿着这个 id 去把对应的聊天记录查出来的。所以这个 id 非常要紧。

再往下，为什么有了创建时间，还要有一个更新时间？

改标题会更新，这个好理解。更主要的是聊天——你每聊一句，这个会话就变了，更新时间就该跟着变。变了有什么用？按时间倒序排的时候，最近聊过的那个会被顶到最前面。列表里七年前建的会话沉在底下，你随手跟它说个「hi」，它立刻回到顶上。这就是更新时间的作用。

最后还有一层。我们这个项目里是有用户概念的，用户要登录。现在登录的是黑马虎哥，那这些会话就都属于黑马虎哥；换一个人登录，看到的会话就该是另一个人的。所有会话都存在同一张表里，不给一个字段标明归属，将来表里成千上万条数据，你根本分不清哪条是谁的。

> **结论**：会话表最少要有五个字段——`id`、`title`、`create_at`、`update_at`，再加一个标明归属的 `user_id`。

| 字段 | 类型 | 作用 |
| :--- | :--- | :--- |
| `id` | UUID | 会话主键，唯一标识一条会话 |
| `user_id` | int | 标记这条会话属于哪个用户 |
| `title` | varchar(200) | 会话标题 |
| `create_at` | 时间 | 创建时间 |
| `update_at` | 时间 | 更新时间，每次聊天都会变 |

### 表映射类 ChatThread

字段定下来，表映射类就能照着抄了。有了表字段，才有映射类；映射类是表的影子，不是反过来。

```python
from uuid import UUID, uuid4

from sqlalchemy import Integer, String
from sqlalchemy.dialects.postgresql import UUID as PGUUID
from sqlalchemy.orm import Mapped, mapped_column


class ChatThread(Base, CreateAtMixin, UpdateAtMixin):
    __tablename__ = "chat_thread"

    id: Mapped[UUID] = mapped_column(PGUUID, primary_key=True, default=uuid4)
    user_id: Mapped[int] = mapped_column(Integer)
    title: Mapped[str] = mapped_column(String(200))
```

类名叫 `ChatThread`，继承 `Base`，另外再继承 `CreateAtMixin` 和 `UpdateAtMixin`。这两个混入类前面已经写过了，一个提供创建时间，一个提供更新时间；表里刚好就要这两个字段，继承过来，自己就不用再写一遍。

> **提示**：既然认了这两个混入类当爸爸，就不能白叫——`create_at` 和 `update_at` 会随继承自动落到这张表上。所以下面三个字段里看不到时间，不是漏了。

`__tablename__` 就是表名，定为 `chat_thread`。

剩下三个字段，一个是 `id`。这里要特别注意，它不再是 `int`，而是 `UUID`——从 Python 自带的 `uuid` 包里导进来，不用额外安装，标准库自己就有。数据库那边也有 uuid 类型，从 `sqlalchemy.dialects.postgresql` 里导出来的类同样叫 `UUID`，两个名字撞车了，所以给它起个别名 `as PGUUID`，意思是「Postgres 里的 uuid」。

于是就有了这样一层对应：类里的类型是 `UUID`，落到数据库里是 `PGUUID`。再配三件事：

- `primary_key=True`，声明它是主键；
- `default=uuid4`，给它一个默认值；
- `uuid4` 是第四代生成算法，`uuid` 还有 1、2、3 代，这里用的是第 4 代。

设了默认值意味着什么？意味着将来往数据库里写这一条的时候，`id` 不用你自己给，它会自动生成一个填进去。生成一次也好，生成一亿次也好，重复的概率低到工程上可以当成不会发生——正因为它不重复，拿它当主键才刚刚好。UUID 不是自增计数器排出来的，它是按算法算出来的：不同代算法取的要素不一样，有的取机器码、网卡地址和时间，`uuid4` 取的是随机数。

> **提示**：`uuid4` 靠 128 位随机数保证唯一，撞车的概率小到可以忽略，但它不是数学意义上的「绝对不重复」。工程上把它当唯一标识用，完全够。

接下来是 `user_id`，在 Python 里就是个 `int`，映射到数据库是 `Integer`；再是 `title`，Python 里是 `str`，对应数据库里的字符串，长度 200，也就是 `String(200)`。

笔记里 `title` 这一行还带过一个 `server_default="新会话"`，意思是在数据库层面给个默认值。这个值其实可以不写——前端创建会话时一定会把标题传过来，写了也用不上，`String(200)` 就够了。当然，写上也不算错。

映射类写到这里就齐活了。类里的 `id` 对应表里的 `id`，类里的 `user_id` 对应表里的 `user_id`，类里的 `title` 对应表里的 `title`，两个时间字段靠继承拿到。这种代码是死代码，格式固定，以后再来一个业务，复制一遍改个名就行——重点该花在业务逻辑上。

### 两个 Schema：进来的和出去的

Model 是照着数据库表写的，Schema 不是。Schema 照着什么写？照着请求和响应写。

前面看接口文档的时候，请求长什么样、响应长什么样，都已经看过了。现在把它们各写成 Pydantic 模型。

#### 请求体：ChatThreadCreate

请求只有一个字段，`title`。

只有一个字段，还要专门写一个类吗？要。因为它是请求体，而且请求是以 JSON 风格提交的。只要是 JSON 风格的请求体，就必须有一个类来接它，哪怕类里只有一个字段。

```python
from pydantic import Field

from app.common.schemas import BaseSchema


class ChatThreadCreate(BaseSchema):
    title: str
```

类名 `ChatThreadCreate`：`ChatThread` 是业务，`Create` 是动作，合起来就是「新增会话」这个请求。

继承谁？理论上继承 Pydantic 的 `BaseModel` 就行。但我们自定义过一个 `BaseSchema`，前面讲过，所有 Pydantic 模型最好都继承它。为什么？因为这样一来，将来做对象转换才能自动转换。直接继承 `BaseModel` 也不算错，只是继承 `BaseSchema` 更好一点。

字段只有一个：`title`，字符串。这样前端提交 JSON 的时候，`{"title": "..."}` 就能被这个对象接住。

笔记里这一行还多了一串东西：

```python
title: str = Field(description="会话标题", min_length=1, max_length=200)
```

`Field` 以前没加过，它是干什么的？是对这个字段的描述和约束——`description` 说明它是个标题，`min_length` 和 `max_length` 限定长度。

这其实就是请求参数校验。Pydantic 的模型天生带数据校验能力，你一旦写上这些约束，校验就自动生效了：前端传过来的值不符合要求，接口直接报错，轮不到业务逻辑去处理脏数据。

> **结论**：定义 Pydantic Schema 的时候，除了字段名和类型，还可以定义约束。约束写在这里，等于在校验这件事上「一次写清、处处生效」。

#### 响应体：ChatThreadResponse

响应有四个字段：`id`、`title`、`create_at`、`update_at`。

表里明明是五个字段，为什么返回的时候少了 `user_id`？

因为用户 id 属于敏感信息。查会话，把会话自己的信息返回给前端就够用了，没有必要把「这条数据属于谁」也带出去。`user_id` 是给后台做数据隔离用的，不是给前端展示用的。

```python
class ChatThreadResponse(BaseSchema):
    id: str
    title: str
    create_at: str
    update_at: str
```

有意思的是两个类型。

`id` 为什么是 `str`，不是 UUID？因为库里存的 UUID，本质上就是一长串字母和数字的混合，它本来就是个字符串。返回给前端的时候，没必要把 UUID 这个类型也带出去，直接用字符串返回；中间需要做一次从 UUID 到字符串的转换。

时间同理。`create_at` 不是时间吗？是时间，但时间可以转成字符串，转成字符串返回没有任何问题。所以两个时间字段也都是 `str`。

> **易错点**：库里的类型和接口的类型是两回事。库里 `id` 是 UUID、时间是时间类型，但对外传输时统一成字符串——前端不需要知道你怎么存的，只关心拿到的是什么。

笔记里这两个字段原先写的是 `UUID` 和 `datetime`，跟接口文档对不上——对了一下，是文档当时没写准，这里按文档统一成 `str`。

### 校验实测：空串挡得住，空格挡不住

`Field` 加上去到底管不管用？嘴上说不算，跑一遍才算。这地方值得单独开个 Notebook 试，因为校验是 Pydantic 的地基，后面每个 Schema 都要靠它。

新建一个文件夹 `day04`，里面建一个 Notebook，叫「测试 Pydantic 模型的校验」。先把刚才写的那个请求体导进来：

```python
from app.modules.chat_thread.schemas import ChatThreadCreate
```

它里面只有一个字段 `title`。现在造一个字典，试试能不能转成这个类的对象。所有 Pydantic 模型都有 `model_validate`，作用是把接收到的数据转成自己的对象——前端提交上来的是 JSON，转到 Python 里就是一个字典，正好喂给它：

```python
data = {"title": "我的第一个会话"}
obj = ChatThreadCreate.model_validate(data)
print(obj)
```

跑一遍，没报错，打印出来是一个 `ChatThreadCreate` 对象。传进去的是字典，出来的是对象，转成功了。

那如果值不符合要求呢？把 `title` 改成空字符串：

```python
ChatThreadCreate.model_validate({"title": ""})
```

立刻报错，意思很清楚：这个字符串至少要有一个字符，你传的太短了。因为我们约束了 `min_length=1`。校验就在 `model_validate` 这一步自动发生了，一行校验代码都没写。

可是——如果我传一堆空格呢？

```python
ChatThreadCreate.model_validate({"title": "        "})
```

它竟然通过了。

从校验规则上讲没毛病：八个空格，长度是 8，大于 1 小于 200，符合要求。但从业务上讲这明显不对：一个全是空格的标题，跟空标题有什么区别？用户看到的就是一条没有名字的会话。

那怎么办？把空格 `strip` 掉再校验。

问题是，这个值不是我们造的，是别人传进来的；校验逻辑也不是我们手写的，是框架内部执行的。你没法在中间插一句「先 strip 一下」。所以这里要用 Pydantic 的**前置处理**：在赋值和校验之前，先对传进来的值动手。

这套写法在 `product` 模块里已经用过一次了——当时的场景是担心传进来空值，所以要么返回默认值，要么返回空字符串、空列表。格式是固定的，换个字段名就能用：

```python
from pydantic import Field, field_validator

from app.common.schemas import BaseSchema


class ChatThreadCreate(BaseSchema):
    title: str = Field(description="会话标题", min_length=1, max_length=200)

    @field_validator("title", mode="before")
    @classmethod
    def validate_title(cls, value: str) -> str:
        return value.strip()
```

拆开看三件事：

- 导入的是 `field_validator`，顾名思义，字段校验器；
- `field_validator("title", ...)` 指明要校验哪个字段——这里叫 `title`；
- `mode="before"` 是关键，意思是**在赋值之前**做处理。

方法里收到一个值，做了什么？`strip()`。就是你们学字符串时学过的那个 `strip`，去掉首尾空白。去掉之后再交给后面的赋值和校验。

所以顺序是：先处理值，再校验，再赋值。这就是「前置处理」四个字的分量。

改完代码，Notebook 要重启一次。因为 `ChatThreadCreate` 已经被导进内存了，源文件改了，内存里那份还是旧的。重启之后再拿八空格试一遍——这次报错了，还是那句「至少得有一个字符」。

明明八个字符那么长，怎么说太短？因为这个长度是假的。`strip` 一过，八个空格全被削掉，长度归零，自然不满足 `min_length=1`。

> **结论**：一个完整的 Schema，三样东西缺一不可——字段（名字和类型）、约束（`Field` 里的长度限制）、前置处理（`field_validator(mode="before")`）。只有字段和约束，形似空标题的脏数据照样能进来。

以后碰到类似的字段校验，直接把这套搬过去，改三处就行：校验的字段名、函数名、以及你想怎么处理这个值。处理方案完全由你定，`product` 里当时是填空值，这里是去空格，逻辑可以自己写。代码是死代码，格式是固定的。

### Repository：只做增删改查

Model 和 Schema 是数据模块，搞定了它们，接下来就是增删改查。三层里 Router 调 Service，Service 调 Repository，所以从最底层的 Repository 写起。

Repository 层负责什么？只负责 CRUD，别的不管。

那「创建会话」翻译成数据库语言是什么？就是往 `chat_thread` 表里插入一条数据，一条 `insert` 语句。

五个字段里，`id` 是自动生成的，两个时间是继承来的、也是自动的，真正需要填的只有 `user_id` 和 `title`。所以要传进来的其实只有两个值。

#### 类的骨架：把 session 拿进来

跟 `product` 里的写法一样，所有业务封装到一个类里，类里定义方法。类名是 `ChatThreadRepository`——前缀是业务名，后缀是 `Repository`。

要增删改查，就必须要拿到 `session`。所以构造函数里接收一个，然后记下来：

```python
from sqlalchemy.ext.asyncio import AsyncSession


class ChatThreadRepository:
    def __init__(self, session: AsyncSession):
        self.session = session
```

构造函数把 `session` 存成实例属性，这个类里定义的**所有**方法就都能直接用 `self.session` 了，不用每个方法再传一遍。

#### add 的两种写法

第一批代码是这么写的：自己创建对象、塞进去、再返回。

```python
async def add(self, user_id: int, title: str) -> ChatThread:
    chat_thread = ChatThread(user_id=user_id, title=title)
    self.session.add(chat_thread)
    return chat_thread
```

这里有个很多人会卡住的点：`session.add()` 到底在 add 什么？

SQLAlchemy 是 ORM，对象关系映射。它干的事就一句话：**把数据库表映射成 Python 的类，把表里的数据映射成类的对象**。`chat_thread` 这张表，对应 `ChatThread` 这个类；表里的一行数据，对应一个 `ChatThread` 对象。

所以「向表里插入一条数据」，在 ORM 里就是「往 session 里放一个对象」。`add` 收的是对象，不是字典，也不是几个字段值。你把这条记住，`add` 就再也不会写错了。

还有两个细节。

第一，`self.session.add(...)` 前面没有 `await`。因为 `AsyncSession.add()` 本身就是同步方法，它只是把对象挂到 session 上，并没有真的发 SQL；真正跟数据库打交道的是后面的 `flush`、`commit`，那些才需要 `await`。查询那类操作要 `await`，新增这里不用。

第二，`return chat_thread` 是因为响应里要用——`id`、`title`、`create_at`、`update_at` 都得返回给前端，所以得把对象交出去。

不过笔记里的最终版本不是这样，它更纯粹：

```python
async def add(self, chat_thread: ChatThread) -> None:
    self.session.add(chat_thread)
```

对象不由 Repository 创建了，由调用方传进来；塞进去之后也不返回。

为什么这么改？因为得替 Service 想一想：如果 Repository 连对象组装都干了，那 Service 还有什么事可干？就没事干了。所以 Repository 要更纯粹一点——它不该关心数据从哪来、对象怎么拼，只该管一件事：**别人给我一个 thread，我把它塞进表里**。

对象由谁来初始化？Service。将来谁调 `add`，谁负责造这个对象。

> **结论**：Repository 的边界就是「接收实体、执行 CRUD」。判断一段代码该不该放这儿，看它是否只跟数据库打交道——凡是带业务判断的，都不属于这一层。

### Service：组装对象、管理事务

Repository 写完了，上一层就是 Service。有了上面的分析，Service 该长什么样，其实已经能想出来了。

#### 构造函数：把 repository 建出来

Service 里也要一个类，类名 `ChatThreadService`，类里定义方法。

Service 干活要用的东西有两样：`session` 和 `repository`。`repository` 在构造函数里创建，因为它需要 `session`；可 `session` 从哪来？Service 自己不知道，那就让别人注入进来。

```python
from sqlalchemy.ext.asyncio import AsyncSession

from app.modules.chat_thread.models import ChatThread
from app.modules.chat_thread.repository import ChatThreadRepository


class ChatThreadService:
    def __init__(self, session: AsyncSession):
        self.session = session
        self.repository = ChatThreadRepository(session)
```

构造函数收到 `session`，一手交给 `repository`，一手自己留一份。这样 Service 里同时握着 `session` 和 `repository`，后面任何方法都能直接取用。

这套写法是固定的：**Service 依赖 Repository，Repository 依赖 Session，依赖谁就在构造函数里注入谁**。

#### add：把对象组装出来

新增方法的骨架跟 Repository 那边一样，也叫 `add`，参数是 `user_id` 和 `title`——这两个值最终来自前端，前端调 Service，Service 通过参数列表接住。

```python
    async def add(self, user_id: int, title: str) -> ChatThread:
        async with self.session.begin():
            chat_thread = ChatThread(user_id=user_id, title=title)
            await self.repository.add(chat_thread)
            return chat_thread
```

方法里三件事：组装对象、交给 Repository、把对象返回。

`id` 不用管，`create_at`、`update_at` 也不用管，它们要么有默认值、要么靠混入类自动填。这里只需要准备两个字段。

`await self.repository.add(chat_thread)` 要带 `await`，因为 Repository 那个 `add` 声明成了 `async def`。同理，这个模块里的方法统一都写成 `async def`。

返回对象，是因为响应体要 `id`、`title` 和两个时间——外面的 Router 拿这个对象转成 `ChatThreadResponse` 就完事了。

> **易错点**：组装对象时要把字段名写全，`ChatThread(user_id=user_id, title=title)`，不能图省事写成 `ChatThread(user_id, title)`。
>
> 构造函数的参数是按字段顺序接的：第一个值会被当成 `id`，第二个当成 `user_id`，`title` 反而没值——因为你只传了两个，剩下的位置对不上。只有把五个字段一个不落地按顺序传齐，才能省略字段名。少传一个，就必须写关键字。

#### 事务为什么开在 Service

上面那段代码里有一层 `async with self.session.begin():`，它是在开启事务。用上下文管理器包起来，正常跑完自动 `commit`，中途出异常自动 `rollback`。

有同学问：这个事务为什么非得开在 Service 里，开在 Repository 里不行吗？

就现在这个业务来说，看不出差别——一共就一次新增，在哪儿开都一样。但把业务想复杂一点，答案就出来了。

假设 Service 里这个方法不只新增，还要再做一个删除：先 `add`，再 `delete`，两次数据库操作。这两次操作应不应该在同一个事务里？当然应该。它们是一次业务动作的两个步骤，必须要么都成功、要么都失败，绝不能出现「新增落库了、删除却没生效」的中间状态。

事务开在 Service，`async with` 把两次调用一起包住，它们就落在同一个事务里。

事务开在 Repository，那就是 `add` 开一次、`delete` 又开一次，两个独立事务各自提交。第一次成功了，第二次失败了，你还能保证「同时成功或同时失败」吗？保证不了了。

```text
Router        接收请求，按 Schema 校验入参
   │
   ▼
Service       组装 ChatThread 对象
   │          事务边界在这里：async with session.begin()
   ▼
Repository    只做一件事：session.add(对象)
   │
   ▼
chat_thread   落库，插入一条记录
```

一次创建会话的调用链就是这样：事务的开合发生在 Service，它把整个业务动作圈在里面，Repository 只在圈内做最朴素的一次插入。

> **结论**：事务的边界由业务决定，而业务逻辑住在 Service。所以只要一个业务动作包含多次数据操作，事务就必须开在 Service——这是分层里最容易放错位置的一件事。

Repository 只管一条 SQL 干不干净，Service 才管这一次业务是不是一个整体。这个分工定下来，后面写更复杂的业务时，才不会出现「两个事务各管一半」的窟窿。

> **承前启后**：上一节讲完「会话创建：表结构、映射类与三层业务代码」，下一节接着讲「会话接口：三个动作，一条从 router 到数据库的链路」。

---

## 会话接口：三个动作，一条从 router 到数据库的链路
> 对应块：BLK15 | 覆盖分集：P57-P60 | 块标题（分集名）：《会话接口：创建、列表查询与重命名》

service 写完了，下一步就轮到 router 了。写 router 说白了就是定义 FastAPI 的接口，而定义接口的关键全压在接口文档上：文档里写的什么请求方式、什么请求路径，决定你这个接口挂什么装饰器、路径怎么拼；文档里列了哪些请求参数，决定等会儿函数参数列表里摆什么；文档里写的响应值类型，决定 `response_model` 填谁。这几样对上了，接口的骨架就立住了，剩下的只是把业务往里填。

会话管理这块有三个动作要落地：创建会话、查询会话列表，以及把重命名会话的接口分析清楚。前两个要写到能跑通，第三个要把参数从哪来、为什么不能上来就改说明白。

### 写接口之前，先把接口文档读成四件事

拿到一份接口文档，我不急着敲代码，先把里面的信息翻译成四件事：

| 接口文档里的信息 | 落到代码的哪里 |
| :--- | :--- |
| 请求方式（POST / GET） | 装饰器 `@router.post` / `@router.get` |
| 请求路径 | `APIRouter` 的 `prefix`，加上装饰器里的相对路径 |
| 请求参数 | 函数参数列表，还要看它放在哪：请求体、请求头还是路径 |
| 响应值类型 | 装饰器的 `response_model` |

这四件事里最容易被看漏的是第三件的后半句——参数放在哪儿。同样是"有一个参数"，放在请求体里要用一个类去接，放在请求头里要用 `Header` 去接，放在路径里只要写一个同名参数。位置不同，写法就完全不同，后面重命名会话时这一点会体现得特别清楚。

路径这一栏还有个小算盘：创建会话和查询会话列表的路径是同一个 `/api/v1/chat/threads`，只是请求方式一个 POST、一个 GET。既然路径重复，就把它提到 `APIRouter` 的 `prefix` 上，装饰器里不用再写一遍。

### 用 APIRouter 立起会话管理模块

先在模块里造一个 router 出来：

```python
from fastapi import APIRouter

router = APIRouter(
    prefix="/api/v1/chat/threads",
    tags=["会话管理"],
)
```

`prefix` 是路径前缀，把这段公共路径固定下来。`tags` 是这个模块的接口在接口文档里的分组名，写"会话管理"，将来文档上这一堆接口就归在一组。

> **易错点**：`tags` 一定要加中括号，它是个列表，不是字符串。

这个 router 不是只服务一两个接口的。将来这个模块里写的所有会话管理接口——创建会话、查询会话列表、重命名会话、删除会话——全都往这里放。

### 创建会话：装饰器、响应模型、请求体

创建会话是 POST，路径和 prefix 完全一致，所以装饰器里的路径直接留空：

```python
@router.post("", response_model=ChatThreadResponse)
async def create_chat_thread(request: ChatThreadCreate):
    # 业务逻辑，参数补齐后往里填
    ...
```

`response_model` 填 `ChatThreadResponse`，这个类在 schema 里早就写好了——返回给前端的就是 id、标题、创建时间、更新时间那个格式。导入时我图省事，直接 `from .schema import *`：schema 里的东西本来就是用来定义接口的，后面每个接口都要用，一个个点名没必要。

接下来是请求参数。前端提交的是 JSON，JSON 在后台不能拿一个裸的 `title` 去接，得用一个类来接。schema 里现成有个 `ChatThreadCreate`，里面就一个 `title` 字段，正好对上：

```python
request: ChatThreadCreate
```

前端把 `{"title": "..."}` 发过来，FastAPI 按类解析成对象，字段名一样就自动塞进去。传到我手上之前，校验工具已经先过了一遍，`title` 不合规的请求根本进不到函数里。

参数名叫 `request`，叫什么都行，重点是类型是 `ChatThreadCreate`。取字段的时候写 `request.title` 就拿到了。

### 依赖注入：需要什么就声明什么

创建会话的业务逻辑只有三步：拿到 service、调新增、返回结果。第一步就卡住了——service 怎么拿？

理论上讲就是 `ChatThreadService(session)`，创建对象嘛。可 session 又从哪来？基础设施模块的 `database` 里有个 `get_session`，调一下就有。那是不是就在函数体里写一行，调完把 session 拿过来？

不这么干。FastAPI 有依赖注入：你需要什么，就在参数列表里声明什么，框架负责把它送进来。所以不是我主动去调 `get_session`，而是我声明"我要一个 session"：

```python
session: AsyncSession = Depends(get_session)
```

等号后面的 `Depends(get_session)` 是在告诉 FastAPI：这个参数的来源是 `get_session`，你调它，把返回值给我。前半句是类型，后半句是来源，这就是依赖注入的全部套路。

#### 把 service 的创建也抽出来

一个接口这么写没问题，问题是后面还有查询列表、重命名、删除。每个接口都要 `ChatThreadService(session)` 一遍，这段创建代码就是纯重复。既然 session 能注入，service 凭什么不能？

第一步，把创建 service 的过程封装成一个函数：

```python
def get_service(session: AsyncSession = Depends(get_session)) -> ChatThreadService:
    return ChatThreadService(session)
```

注意这个函数自己也在用依赖注入：它要 session，所以参数里写 `Depends(get_session)`。session 从哪来的问题，在这个函数里就解决了。

第二步，接口里不再手动创建 service，改成声明：

```python
service: ChatThreadService = Depends(get_service)
```

类型是 `ChatThreadService`，来源是 `get_service`。就这么两步，以后不管写几个接口，拿 service 都是同一句话。

> **提示**：这套写法可以记成三句话——把要的东西的创建过程封装成函数、在参数列表里声明它的类型、用 `Depends` 告诉 FastAPI 它从哪来。第一次写会觉得绕，写着写着就成肌肉记忆了。

### 用户 id 藏在请求头里

service 有了，接下来就是 `service.add(user_id, request.title)`。标题好办，`request.title` 直接取。可用户 id 上哪整？

回浏览器看这个请求，id 是跟着请求头走的，名字叫 `x-user-id`。

> **提示**：企业开发里用户信息基本都走请求头，不走请求参数。走参数别人一眼就看见你的 id，走请求头相对隐蔽，也安全一些。

那就得会从请求头取参数。这个东西以前没学过——没学过怎么办？问 AI。别笑，企业开发里碰到从没接触过的需求，第一反应就该是问 AI。早些年没有 AI 的时候，碰到一个陌生需求只能自己去网上人肉搜，搜到方案再照着改，那才叫费劲。

我上午就提前问了一句"FastAPI 如何获取请求头参数"，它给了两种推荐方式，第一种就是用 `Header` 函数，也是最常用的，我们就看这一种。`Header` 落到代码里又有两种写法，都值得记一下。

#### 原始写法：类型加来源

```python
user_id: int = Header(alias="x-user-id")
```

冒号后面是类型 `int`，等号后面是来源 `Header(...)`，括号里的别名就是前端发请求时那个头的名字。这句话翻译过来就是：这个参数是 int，从请求头里取，取的是 `x-user-id` 这个头。

#### Annotated 写法：把类型和来源分开

```python
user_id: Annotated[int, Header(alias="x-user-id")]
```

`Annotated` 来自 `typing`，直译是"带注释的"，作用是给一个类型挂上额外说明。中括号里两段：第一段 `int` 是数据类型，第二段 `Header(...)` 是数据来源。它跟原始写法表达的是同一件事，只是把类型和来源分开放，读起来更清楚，也更优雅。

| 写法 | 声明 | 读法 |
| :--- | :--- | :--- |
| 原始写法 | `user_id: int = Header(alias="x-user-id")` | 类型在冒号后，来源在等号后 |
| Annotated 写法 | `user_id: Annotated[int, Header(alias="x-user-id")]` | 中括号第一段是类型，第二段是来源 |

两种都能用，喜欢哪种写哪种。后面统一用 `Annotated` 这种。

> **易错点**：没有默认值的参数必须排在带默认值的参数前面。`user_id` 没有默认值，`service` 后面挂着 `Depends(...)`，所以 `user_id` 要往前挪，不能顺手写到最后。

请求头的名字大小写不敏感，写 `X-User-Id` 和 `x-user-id` 是一回事。以后要是前端换了头名，改哪里？只改别名，数据类型不动，变量名跟着语义走。

到这儿，创建会话的接口就齐了：

```python
@router.post("", response_model=ChatThreadResponse)
async def create_chat_thread(
    request: ChatThreadCreate,
    user_id: Annotated[int, Header(alias="x-user-id")],
    service: ChatThreadService = Depends(get_service),
):
    return await service.add(user_id, request.title)
```

有一处看着别扭：`service.add` 返回的是 `ChatThread`（model），而接口声明的 `response_model` 是 `ChatThreadResponse`（schema），类型对不上，怎么敢直接 return？

因为转换这件事交给了 FastAPI。它拿到返回值会按 `response_model` 做序列化；而我们的 model 和 schema 都继承自 `BaseSchema`，基类里已经打开了"允许从对象属性构造"的开关，所以任意一个对象都能转成 pydantic 模型。业务层返回 model、接口层声明 schema，中间这层转换不用我们动手。

### 两次报错：404 和 405

写完不测等于没写。刷新页面，清空，直接 404。

路径没错、方法没错，为什么找不到？因为 router 定义了，但没挂载。这件事特别容易忘：定义完 router 一定要在入口文件里挂上去，不然它就是一个躺在模块里的对象，FastAPI 根本不知道它的存在。

到 `main.py` 里照猫画虎加两行：

```python
from app.api.chat_thread import router as chat_thread_router

app.include_router(chat_thread_router)
```

导入时起个别名。因为每个模块里都有个叫 `router` 的变量，直接导会打架，叫 `chat_thread_router` 一眼就知道是谁。

再刷新，这回变成 405 Method Not Allowed。

405 不是路径找不到，是方法不对。浏览器地址栏直接访问发的是 GET，而这个接口只注册了 POST，FastAPI 一看路径对得上、方法不允许，就回 405。点页面上的"新建"按钮，发的是 POST，200，返回值里 id、标题、创建时间、更新时间全都有，左侧会话列表也刷出来了。

> **结论**：404 先查挂载，405 先查请求方式。一个在入口文件，一个在装饰器，都不在业务代码里。

### 查询会话列表：无参的接口，隐含的参数

新增能用之后我发现一个问题：一刷新，列表又没了。因为只有新增接口、没有查询接口——页面重新加载时去拉列表，拉不到东西，自然就空了。

那就看查询会话列表的接口文档。请求方式是 GET，路径还是 `/api/v1/chat/threads`，跟新增一模一样；请求参数一栏写着：无。

无参？那我怎么查。难道把这张表里所有人的会话全捞出来返回？这显然不合理。

那按 id 查？可人家压根没传 id。

关键在这一句接口说明上：**查询当前用户的会话**。现在页面是登录状态，登录的是哪个用户，就该查哪个用户的会话，别人的不能给。所以这个接口看起来没有参数，其实有个隐含参数 `user_id`——它跟新增一样，藏在请求头里。拿这个 `user_id` 去表里查 `user_id` 等于该值的所有会话，这才是这个接口真正干的事。

> **提示**：接口文档写"无参"，说的是没有请求体和查询字符串参数，不代表没有身份信息。凡是"查当前用户的某某"，参数一定在请求头里。

响应格式是一个列表，列表里每个对象有四个字段：id、title、create_at、update_at。这四个字段是不是似曾相识？新增会话时返回的也是这四个。所以查询列表的返回值和新增的返回值同构，`ChatThreadResponse` 这个 schema 直接复用，一行都不用再写。

| 对比项 | 创建会话 | 查询会话列表 |
| :--- | :--- | :--- |
| 请求方式 | POST | GET |
| 请求路径 | `/api/v1/chat/threads` | 同左 |
| 请求参数 | 请求体 `title` + 请求头 `user_id` | 只有请求头 `user_id` |
| 响应模型 | `ChatThreadResponse` | `list[ChatThreadResponse]` |

文档里还有一条更细的要求：要按时间倒序，先按 `update_at`，再按 `create_at`。这样最近动过的会话永远排在最前面。

### 三层落地：repository、service、router

按老规矩本该先写 schema 和 model，可刚才分析完发现两样都有现成的，那就直接从下往上写：repository 写查询，service 转一手，router 对外开接口。

```text
router  →  service  →  repository  →  session  →  chat_threads 表
  收参        转发        拼 SQL        执行         取数据
```

链条上每一段各管一摊，谁都不越界：参数从请求头进来，条件由 repository 拼成 SQL，数据从表里捞出来再原路返回。

#### 仓储层：一条带条件的查询

先把 SQL 想清楚，再去写代码：从 `chat_threads` 表里 `select *`，条件是 `user_id` 等于前端传进来的那个值。

```python
from sqlalchemy import select


async def list_by_user(self, user_id: int):
    result = await self.session.scalars(
        select(ChatThread)
        .where(ChatThread.user_id == user_id)
        .order_by(ChatThread.update_at.desc(), ChatThread.create_at.desc())
    )
    return result.all()
```

传统写法是 `execute`，这里用 `scalars()` 简化——查出来的每一行是 ORM 对象，`scalars()` 直接把这些对象摊开给你，省掉一层 `row[0]` 的取值。

SQLAlchemy 的映射关系要记牢：查表就传类，写数据就传对象，字段就是类的属性。所以 `select(ChatThread)` 里的 `ChatThread` 是那张表，`ChatThread.user_id` 是表里的字段，`.desc()` 是降序。排序按文档要求先 `update_at` 再 `create_at`，两个都降序，`order_by` 里用逗号隔开就是"先按前者，前者相同再按后者"。

> **易错点**：第一遍写的时候光想着条件，排序漏了，跑通之后回文档才补上。接口文档里那些"附加说明"，往往就是最容易漏的部分。

#### 服务层：这一层只做转手

```python
async def list_by_user(self, user_id: int):
    return await self.repository.list_by_user(user_id)
```

查询没有任何额外业务，service 不需要加工，拿到 `user_id` 直接交给 repository，再把结果原样返回。不是每个 service 方法都得有逻辑——它的职责是"对上层统一提供业务入口"，哪怕这一层薄得像张纸，也不能让 router 越过它去碰 repository。

#### 接口层：复制一个再改

新接口和创建接口骨架几乎一样，所以直接把创建接口复制一份来改，比从零敲快得多，也不容易漏东西。

```python
@router.get("", response_model=list[ChatThreadResponse])
async def list_by_user(
    user_id: Annotated[int, Header(alias="x-user-id")],
    service: ChatThreadService = Depends(get_service),
):
    return await service.list_by_user(user_id)
```

改了四处：装饰器从 `post` 换成 `get`；`response_model` 从单个对象换成 `list[ChatThreadResponse]`，表示返回一个列表、列表里装的是会话对象；请求体参数 `request` 整个删掉，因为这个接口没有请求体；最后调用的方法换成 `service.list_by_user`，把 `user_id` 传进去。请求头那一行和 `service` 那一行原封不动。

整个接口还是一行代码。别小看这一行——`await` 千万别漏，异步函数不 await，拿到的就是个协程对象，返回给前端直接报错。我重启前就差点漏掉，回头补上的。

刷新页面，列表回来了，返回值是一个列表，里面装着两个对象。到这儿查询会话列表就通了。

### 重命名会话：三个参数分居三处

列表回来了，页面上还有个重命名。点一下，改个名字，按保存——请求发出去了，报错。因为这个功能还没做。

重命名听起来最简单：不就是把标题更新到数据库里，把 `title` 字段的值改掉吗？对，但有个前提问题——我有两条会话，改的是哪一条？

所以还得传一个会话 id 来标识目标。数据库表里的数据本来就是用 id 区分的，前面查列表时那两条各自带着自己的 id，点哪一条改，就带哪一条的 id。

回浏览器看这个请求，路径最后一段是 `2932`，正好是列表里某一条会话的 id；新名字在请求体里，是 JSON。也就是说会话 id 走请求路径，标题走请求体。

再看接口说明：**修改当前用户指定会话的标题**。这一句话里有三个关键点——当前用户、指定会话、标题。落到参数上就是三个：

| 参数 | 放在哪 | 作用 |
| :--- | :--- | :--- |
| `thread_id` | 请求路径的占位符 | 指明改哪一条会话 |
| `title` | 请求体（JSON） | 改成什么标题 |
| `user_id` | 请求头 | 证明这条会话是你的 |

```text
/api/v1/chat/threads/{thread_id}
                     └─ 路径占位符：要改哪一条会话

请求头  x-user-id: 1         ── 你是谁（隐含参数）
请求体  {"title": "新名字"}   ── 改成什么标题
```

三个参数分居三处，接收方式也各不相同，正好把前面学的一起复习了：路径占位符最省事，参数列表里写一个同名参数就能接住，`thread_id` 就是这么来的；请求体还是用一个类接，类里放 `title` 字段；请求头继续用 `Header` 加别名。

响应也不用新东西，返回的还是 `ChatThreadResponse`——id、title、create_at、update_at 四个字段，跟新增、跟列表里的元素完全一致。schema 到这里已经被三个接口复用了三次，这就是当初把响应结构抽成 schema 的好处。

### 动手改之前，先确认这条会话是你的

接口文档里还压着一句容易被跳过的说明：修改会话之前，一定要校验当前会话是否属于当前用户。

为什么要专门校验？因为 `thread_id` 是公共的、肉眼可见的。你跟 DeepSeek 聊天，会话地址里那串 id 挂在浏览器上，谁都能看到。假如别人拿到了你的会话 id，手动构造一个请求，带上这个 id 和一个新标题发过去，服务端二话不说就改了——可这个请求根本不是你发的。改的却是你的会话，这就不安全了。

所以请求头里的 `user_id` 不是摆设，它是用来做验证的。比如现在传的 `user_id` 是 1，要改的会话是 2932，那就验证一下：2932 这条会话，是不是属于 1 这个用户？

数据库里 `id` 和 `user_id` 两个字段都有，这件事查一下就清楚——写一条同时带两个条件的查询，`id` 等于传进来的会话 id、`user_id` 等于当前用户 id。

```text
拿 thread_id + user_id 查一次
        │
        ├─ 查得到 → 这条会话属于当前用户 → 允许更新 title
        └─ 查不到 → 不是你的会话 → 非法请求，拒绝
```

> **结论**：更新本身很简单，`update` 这张表、`set` 一下 `title`、`where` 跟上 `id` 就完事；难的是"能不能改"这一步——不能谁来了都放行，必须先把 `id` 和 `user_id` 两个条件凑在一起查一遍，匹配了才允许改，不匹配就是冒牌货。

这也是分析接口文档时最该练的地方：文档给出的不只是路径和字段，还有业务规则。接口文档不是拿来抄路径的，是拿来把业务想明白的——规则想清楚了再动手，代码才写得踏实。

> **承前启后**：上一节讲完「会话接口：三个动作，一条从 router 到数据库的链路」，下一节接着讲「会话重命名：从 ORM 自动更新到统一异常处理」。

---

## 会话重命名：从 ORM 自动更新到统一异常处理
> 对应块：BLK16 | 覆盖分集：P61-P63 | 块标题（分集名）：《会话重命名与统一异常处理》

接口文档摆在旁边，前面的分析也做完了，接下来该动手。先别急着新建文件——把"重命名会话"这个接口拆开看，它需要的东西少得有点出乎意料：请求体里只有一个 `title`，跟新增会话的请求体长得一模一样；返回的还是那条会话信息，跟新增会话的返回结构也一模一样。既然一模一样，就没有必要再抄一份。

于是这一节的起点不是"新建 schema"，而是"确认哪些东西可以直接复用"。确认完了，剩下的活就集中在三处：仓储层的一次归属校验、业务层的一次查询加一次修改、路由层的一个 `PATCH` 接口。等这三处写完，真正的麻烦才浮出来——非法请求该回什么？答案是 500，而 500 恰恰是最不该给的答案。后面大半段时间，其实都在收拾这个"不该给的答案"。

### 先盘点：这个接口几乎不用新建东西

重命名会话的请求参数是直接接收的，路径上带一个会话 ID；请求体呢，跟新增会话的请求体完全一致，就是一个 `title`。这一点在 schema 文件里能直接对上：`ChatThreadCreate` 里装的就是 `title`，把它拿来当重命名的请求体，一个字都不用改。

返回值也一样。重命名成功之后要返回重命名后的会话信息，而这个结构跟新增会话返回的结构是同一个。所以响应侧同样不用重新定义。

再往下看 model。今天要做的所有业务——新增、查询、重命名——操作的都是同一张 `chat_thread` 表，表结构没有任何变化，model 自然也不用动。

> **结论**：schema 跳过，model 跳过，直接从 repository 往上写功能。

把这几层摊开看，需要动手的地方其实只有三个：

| 层 | 要不要新写 | 原因 |
| :--- | :--- | :--- |
| schema | 不用 | 请求体只有一个 `title`，与新增会话完全一致，响应结构也一致 |
| model | 不用 | 全天业务共用 `chat_thread` 这一张表 |
| repository | 要 | 需要按 `user_id` 加 `thread_id` 校验会话归属 |
| service | 要 | 先查再改，还要处理"查不到"的情况 |
| router | 要 | `PATCH` 方法、三个参数、路径多一段 |

### Repository：先证明会话属于当前用户

写这个接口之前，有一件事必须先想清楚：前端传过来的 `thread_id`，凭什么认定它是当前用户的？它完全可能是别人的会话 ID，甚至是随手编的一个。所以在改标题之前，得先做一次校验——拿会话 ID 和用户 ID 两个条件，一起去 `chat_thread` 表里查。

查得到，说明这两者是匹配的，这条会话确实属于当前用户；查不到，说明要么会话不存在，要么存在但不属于你。两种情况在业务上是一回事，都不该让你改。

这个函数我取名 `require_owner`。`require` 是"要求"的意思，`owner` 是"拥有者"——不是"老板"，翻译成老板是不对的。合起来就是：要求你必须是这条会话的拥有者，才有资格往下走。

它需要两个入参：`user_id` 和 `thread_id`。`thread_id` 是字符串形式的 UUID，查询条件里传字符串完全可以——按主键查的时候也是这么传的。这两个值从哪来？当然是调用方传进来，仓储层自己不生产数据。

查询用 `scalar`：因为我们只查一条记录，用 `scalar` 比 `execute` 再取 `scalars().first()` 省事得多。写起来是这样：

```python
from sqlalchemy import select

async def require_owner(self, user_id: int, thread_id: str) -> ChatThread | None:
    stmt = select(ChatThread).where(
        ChatThread.id == thread_id,
        ChatThread.user_id == user_id,
    )
    return await self.session.scalar(stmt)
```

这段代码里有三条对应关系值得说透，因为它们决定了 ORM 写法和 SQL 写法的换算方式：

- `select` 后面写的是**类**，不是表名。表对应到 Python 里就是类，类名 `ChatThread` 就是那张 `chat_thread` 表；
- `where` 里面写的是**类属性**，不是列名。字段对应到类里就是属性，`ChatThread.id`、`ChatThread.user_id` 分别对应表里的 `id` 和 `user_id`；
- 两个条件用逗号隔开，SQLAlchemy 会把它们当成"并且"拼进 `WHERE`，等价于 `WHERE id = ? AND user_id = ?`。

还有一处细节：返回值类型我标了 `ChatThread | None`。不标会怎么样？在 service 里调用这个函数的时候，IDE 就不知道拿回来的是什么东西，改 `title` 的时候既没有提示也没有类型检查，很容易写错。所以这个标注不是装饰，是给下游用的。

至于 `scalar` 查不到时会返回什么，当时还真不确定——`execute` 能返回空，`scalar` 会不会不一样？先按"可能为空"处理，反正下游本来就要判空；后面拿一个不匹配的 `user_id` 一测，走进的正是空值分支。所以这个返回值标成 `ChatThread | None` 是对的。

> **提示**：`require_owner` 严格来说不算增删改查里的"改"。它干的仍然是"查"，只不过这个查的目的是校验。持久层只负责执行语句，业务判断留给 service。

### 顺手验证：ORM 改完字段就自动更新

在继续写 service 之前，得先补一个容易被忘掉的前提：SQLAlchemy 有 ORM 功能，一条数据查出来之后，只要修改这个对象的任意字段，它就会自动把改动写回数据库——不需要你手写 `UPDATE` 语句。

这条特性如果记不清，后面 service 的写法就看不懂。所以当场演示一遍。思路很简单：拿到 session，按主键 `get` 一条记录出来，改它的 `title`，然后去数据库里看有没有变化。

```python
from app.core.database import async_session_factory

async with async_session_factory() as session:
    async with session.begin():
        chat_thread = await session.get(ChatThread, "会话ID")
        print(chat_thread)
        chat_thread.title = "测试二"
```

按主键查询可以直接用 `get`：第一个参数传表对应的类，第二个参数传主键值，拿回来的就是那个 `ChatThread` 对象。

但光有 `get` 和赋值还不够，外面必须套一层 `session.begin()`。为什么？因为有修改。有修改就要开事务，这是硬要求，不是可选项。

跑完之后去数据库里翻一眼：原来是"测试"，现在变成了"测试二"。

> **结论**：查出来、改字段、提交，数据库就更新了。`UPDATE` 语句由 ORM 帮你发，你不需要自己拼。

这条结论直接决定了 repository 的写法——里面只需要一个查询函数就够了，不用再补一个"根据 ID 修改标题"的方法。改标题这件事，在 service 里改对象属性就完成了。

### Service：先查再修，两步走在一个事务里

仓储层写好了，往上就是 service。这个函数叫 `rename`，重命名。

它的逻辑只有两步，顺序不能反：先查询会话、校验它是否属于当前用户，再修改标题。先查再修，先查再修——这两步必须放在同一个事务里，因为中间涉及了修改。开启事务还是那个写法：`self.session.begin()`，然后把这两步放进去。

查询不用自己写，仓储层刚刚已经提供了 `require_owner`，把 `user_id` 和 `thread_id` 传进去就行。这两个值又是从哪来的？service 自己不知道，谁调用它谁传给它——这个"责任往上推"的思路，从 repository 到 service 一路都是一致的。

查回来的是一个 `ChatThread` 对象，也就是这张表里的一条数据。接下来改它的 `title`。改成什么？前端传过来的新标题。所以 `title` 也得作为参数传进来。

```python
async def rename(self, user_id: int, thread_id: str, title: str) -> ChatThread:
    async with self.session.begin():
        chat_thread = await self.repository.require_owner(user_id, thread_id)
        if not chat_thread:
            raise ChatThreadNotFound()
        chat_thread.title = title
        await self.session.flush()
        await self.session.refresh(chat_thread)
        return chat_thread
```

这段代码看着短，但每一行都有来由。`if not chat_thread` 是后面要专门讲的一道闸门，`ChatThreadNotFound` 这会儿还没定义、等会儿补上；`flush` 和 `refresh` 是这一节最重要的一个坑；`return` 则是因为接口要求响应里带上重命名后的会话信息——写完数据库还不算完，得把这条数据返回给前端。

只是这个 `return` 一开始并不是这个样子，它是踩了一个坑之后才补成现在这样的。

### 为什么返回的数据是旧的

第一次跑这段演示的时候，改动确实写进数据库了，但打印出来的那个对象，`update_at` 跟修改前一模一样。

这不合理。既然刚刚改过，更新时间就应该变。于是去数据库里核对：表里的 `update_at` 明明已经变了。也就是说，数据库是新的，返回出去的是旧的。

原因不复杂。`chat_thread` 这个变量，是查询那一刻从数据库读回来的对象，它在内存里存的是旧值。后面改 `title`，触发的是 ORM 往数据库发 `UPDATE`，数据库那一行变了，但内存里这个对象并不会自动重新加载一遍。你打印它、返回它，拿到的还是查出来时的那份数据。

```text
查出来（内存里是旧对象，update_at = 旧时间）
        │
        ├── 改 title ──▶ 内存对象的 title 变了，update_at 没变
        │
        └── 提交事务 ──▶ 数据库那一行被 UPDATE，update_at = 新时间
                              │
                              └── 内存对象不跟着变，返回出去仍是旧值
```

图里那条分叉就是问题的全部：数据库和内存各走各的，谁也没通知谁。

那怎么拿到新数据？两条路。

第一条，把读取放到事务外面。事务结束了再取，理论上就能取到最新的。但这条路不一定有效——因为我们之前配过一个东西：提交后不过期（`expire_on_commit=False`）。有了这个配置，事务提交之后对象不会被标记为过期，再取也不会重新查库。实测的结果更直接：`update_at` 这个字段干脆没了，对象里只剩下 `create_at`。这条路走不通。

第二条，手动刷新。两个动作配合：

```python
await self.session.flush()
await self.session.refresh(chat_thread)
```

`flush` 把当前事务里挂起的改动写回数据库（注意，它不是提交事务，事务还是那个事务，只是把 SQL 发出去）；`refresh` 拿着这个对象重新查一次库，把最新的值灌回对象。两个动作合起来，`update_at` 就变成最新时间了。

> **易错点**：业务里改了数据、又要在同一个事务里立刻把这条数据返回给前端，必须做一次刷新。少这一步，接口会稳定地返回旧值，而且数据库里的数据是对的，排查起来特别费劲——你会以为是更新没生效，其实是返回的对象没刷新。

到这里 service 才算写完，但还差一个分支没处理：查不到的情况。

### 非法请求要抛异常

前面那个 `if not chat_thread`，挡的就是非法请求。

想想这个场景：请求里的 `user_id` 和 `thread_id` 不匹配。`require_owner` 按这两个条件去查，什么也查不到，返回空。如果不做判断直接往下走，下一行就是 `chat_thread.title = title`——对着一个空值改属性，立刻报错。这种错误是"撞"出来的，报错信息也说明不了业务含义。

所以要先判断，没值就抛异常。`raise` 一个异常，代码会立刻终止，后面的修改和返回都不会执行。

抛什么异常？直接抛一个 `Exception` 当然可以，但不够专业。将来业务里这种场景会非常常见——会话不存在、用户不存在、权限不够——每一种都该有自己明确的类型，方便上层分辨。这类自己定义的异常，就叫自定义异常。

写法比想象的简单：定义一个类，继承 `Exception`。

```python
class ChatThreadNotFound(Exception):
    pass
```

一个普通类，一旦继承了 `Exception`，它就不再是普通类，而是异常类。这个异常表达的含义很明确：当前会话不存在，或者不属于当前用户。这两种情况在业务上归为一类——反正你都改不了它。

这个类不应该写在 service 里。service 是写业务的地方，异常定义塞进去会越堆越乱。惯例是单独放一个文件，叫 `exceptions.py`，把这个模块里所有的异常都收在里面，service 需要的时候导入即可。

抛出去之后，控制台里能直接看到这个自定义异常的类名，还带着抛出位置所在的文件和行号——自己扔的异常，自己看得见扔在哪。

不过要让前端真的收到它，得先把路由接上。异常有了，接口还没有。

### Router：复制一份新增接口，改三处

路由这一层不用从零写。新增会话和重命名会话这两个接口非常像，直接把新增那个复制一份过来改，比对着空白文件敲要快，也不容易漏东西。

要改的地方其实只有三处。

第一处是请求方式。重命名用的是 `PATCH`，新增用的是 `POST`。`PATCH` 以前没怎么用过没关系——`GET`、`POST`、`PUT`、`DELETE` 这些方法本来就是按语义分工的，前端要求用 `PATCH`，那就用 `PATCH`。

第二处是请求路径。上层的 `/api/v1/chat/thread` 已经有了，缺的只是后面那一段参数，把它补上：

```text
PATCH  /api/v1/chat/thread/{thread_id}
```

第三处是函数本身：函数名从 `create` 改成 `rename`，参数从原来的两个变成三个。这三个参数正好覆盖了三种来源：

| 参数 | 来源 | 说明 |
| :--- | :--- | :--- |
| `thread_id` | 路径参数 | 从 URL 的 `{thread_id}` 段接收，字符串形式的 UUID |
| `request` | 请求体 | 类型还是 `ChatThreadCreate`，里面只有 `title` |
| `user_id` | 请求头 | 从 header 里取，新增接口里已有这一份，直接沿用 |

```python
@router.patch("/thread/{thread_id}")
async def rename_thread(
    thread_id: str,                    # 路径参数
    request: ChatThreadCreate,         # 请求体，里面只有 title
    user_id: int = Header(),           # 请求头
):
    # service 的获取方式与新增会话接口一致，沿用复制过来的那一行
    return await service.rename(user_id, thread_id, request.title)
```

装饰器上的响应模型不用动——重命名的返回结构和新增是同一个。路由的挂载也不用动：这两个接口属于同一个 router，前面已经挂载过了，现在只是给这个 router 加了一个新方法而已。

重启服务，进接口文档，会话管理里能看到那个 `PATCH` 的重命名接口。测试就在这一页做——别的页面里用户 ID 和会话 ID 天然是配套的，根本造不出"不匹配"这种场景。从数据库里拷一条真实数据出来：某条会话的 `thread_id`，它对应的 `user_id` 是 1。

先传正确的一对，执行，状态码 200，返回的是最新数据，没问题。

再故意把 `user_id` 换成一个不相干的，执行——500。

### 500 是甩锅，错在参数就该回 400

500 这个状态码的含义是"服务内部错误"。也就是说，前端看到 500，第一反应是"服务端挂了"。

可这次是谁造成的？不是服务端。是有人拿着不属于自己的会话 ID 来改标题，传了非法参数。服务端一点毛病没有，它只是拒绝了这次请求。把这种情况报成 500，等于把责任揽到自己身上，前端拿着 500 也没法判断到底是该重试、该提示用户，还是该找后端。

按 RESTful 的约定，请求参数有误应该给 400。400 的含义很明确：你传的东西不对。

顺手把这几个常见状态码理一遍：

| 状态码 | 含义 | 典型场景 |
| :--- | :--- | :--- |
| 200 | 成功 | 正常返回数据 |
| 400 | 请求参数有误 | 参数不合法、归属校验不通过 |
| 404 | 资源不存在 | 访问的路径或资源找不到 |
| 405 | 方法不允许 | 路径对，但用的 HTTP 方法不对 |
| 500 | 服务内部错误 | 服务端代码抛了未处理的异常 |

所以这里不该让异常直接往外冒，而应该捕获它，返回一个我们自己指定的状态码和消息。

### 全局统一异常处理：把异常交给 FastAPI

要捕获异常，先得弄清楚它被抛给了谁。

`service` 里 `raise` 出来的异常，抛给了它的调用者——router。router 没有再处理，又往上抛，抛给了 FastAPI。所以这个异常最终是 FastAPI 接住的，处理它的人应该是 FastAPI。

FastAPI 在哪定义的？在 `main` 里，那一行 `app = FastAPI()`。既然 `app` 就是 FastAPI 的实例，异常处理的代码就该写在 `main` 里。

写法是固定的：一个函数，上面加一个装饰器。这份代码也不是新东西，是之前讲统一异常处理时就写过的一份，直接拿过来用即可。

```python
from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse

from app.chat_thread.exceptions import ChatThreadNotFound

app = FastAPI()


@app.exception_handler(ChatThreadNotFound)
async def chat_thread_not_found_handler(request: Request, exc: ChatThreadNotFound):
    return JSONResponse(
        status_code=400,
        content={
            "code": "chat_thread_not_found",
            "message": "当前会话不存在或者不属于当前用户",
        },
    )
```

拆开看这几个部件：

- `app.exception_handler` 是装饰器，`exception` 是异常，`handler` 是处理器，合起来就是"异常处理程序"。加上它，这个普通函数就变成了全局异常处理器；
- 装饰器的参数写要捕获的异常类型。这里写 `ChatThreadNotFound`，意思是：只要有人抛这个异常，就交给下面这个函数；
- 函数有两个参数，第一个是请求对象，第二个是被捕获到的异常对象。两个都得有，顺序不能反；
- 函数体返回一个 `JSONResponse`，`status_code` 指定 HTTP 状态码，`content` 是响应体。

响应体里放了两个字段，这两个概念要分清楚：

- `code` 是**错误码**，业务层面的标识，给前端做判断用，比如 `chat_thread_not_found`；
- `message` 是**错误消息**，给人看的说明，比如"当前会话不存在或者不属于当前用户"。

而 `status_code=400` 那个 400，是 **HTTP 状态码**，属于协议层。三者各司其职：HTTP 状态码告诉客户端这一类请求出了什么性质的问题，错误码告诉前端具体是哪一种业务异常，错误消息用来直接展示或记录。前端拿到这一份响应，既能看到状态码，又能拿到错误码和错误消息，判断和展示都不缺东西。

`Request` 和 `JSONResponse` 都要手动导入，IDE 不一定自动补。重启服务，再传一次错误的 `user_id`：状态码变成 400，响应体里就是刚才写的那两个字段。

> **提示**：如果暂时还不会写异常处理，这一步可以先跳过——让异常直接抛出去变 500，接口照样能跑通。等功能都通了再回头补，顺序不影响结果。

### 异常一多，处理器就写不过来了

现在的写法能跑，但它只解决了"这一个"异常。

往后业务里还会有别的异常。比如某天加了一个 `UserNotFound`，它要返回的就不是 400 而是 404，错误码也不是 `chat_thread_not_found` 而是 `user_not_found`，消息是"用户不存在"。处理逻辑跟刚才那个几乎一样，但三样东西全变了：状态码、错误码、错误消息。

那是不是每多一个异常，就往 `main` 里再抄一个处理器？业务复杂起来有上百个异常，难道写上百个异常处理器？这显然不合理。

不合理的原因，归纳起来是两条：

- **异常类型不统一**。每个自定义异常都是独立的类，类型各不相同。而捕获异常必须写类型，类型不同就只能一个一个分开捕获，一个处理器管一个；
- **返回内容不统一**。每个异常想返回的状态码、错误码、错误消息都不一样，所以处理逻辑没法共用。

这两条卡在一起，就成了"异常越多、处理器越多"的死循环。要破它，也得从这两条分别下手。

### 自定义异常的最终形态：统一基类加自带信息

针对上面两条原因，要做两件事：

1. 给所有自定义异常找一个共同的父类，让它们变成同一种类型。这样捕获的时候只写父类，所有子类异常都会被接住；
2. 让异常自己带上状态码、错误码和错误消息。这样处理器不用把返回值写死，从异常对象里取就行。

先做第一件。这个父类放在核心模块里，文件叫 `exceptions.py`，类叫 `ApplicationError`——应用异常的意思，代表这个应用内部的业务异常。

```python
class ApplicationError(Exception):
    def __init__(
        self,
        status_code: int = 500,
        code: str = "application_error",
        message: str = "服务器内部错误",
    ):
        self.status_code = status_code
        self.code = code
        self.message = message
```

它继承 `Exception`，所以它是一个异常类；它带三个字段，所以它能装下每一种异常各自想返回的东西。三个默认值分别是 500、`application_error`、"服务器内部错误"——这是一份兜底：万一某个子类忘了填，至少还能给出一个像样的响应。

> **注意**：`ApplicationError` 只是父类，它的默认值不等于最终答案。真正决定返回什么的，是具体抛出来的那个子类。

然后是 `main` 里的处理器。捕获的类型从具体的 `ChatThreadNotFound` 换成 `ApplicationError`，返回值不再写死，全部从异常对象里取：

```python
@app.exception_handler(ApplicationError)
async def application_error_handler(request: Request, exc: ApplicationError):
    return JSONResponse(
        status_code=exc.status_code,
        content={"code": exc.code, "message": exc.message},
    )
```

这里有个容易写错的地方：函数第二个参数的类型标注，必须跟装饰器里捕获的类型一致。装饰器写着捕获 `ApplicationError`，参数却标成别的类型，类型检查会报错——这个参数代表的就是被捕获的那个异常，两者天然应该对得上。

最后做第二件事：让 `ChatThreadNotFound` 认祖归宗。它不再继承 `Exception`，改成继承 `ApplicationError`，然后在构造的时候把自己的那份信息填进去：

```python
from app.core.exceptions import ApplicationError


class ChatThreadNotFound(ApplicationError):
    def __init__(self):
        super().__init__(
            status_code=400,
            code="chat_thread_not_found",
            message="当前会话不存在或者不属于当前用户",
        )
```

现在链路闭合了。业务层抛的是 `ChatThreadNotFound`，它是 `ApplicationError` 的子类，所以会被那个唯一的处理器接住。接住的时候形参看着是 `ApplicationError`，实际传进来的是子类实例，从里面取出来的 `status_code` 是 400、`code` 是 `chat_thread_not_found`、`message` 是"当前会话不存在或者不属于当前用户"。

换成别的异常也一样：假设哪天有个地方抛了 `ApplicationError` 本身，取出来的就是默认的 500、"application_error"、"服务器内部错误"。

```text
service  raise ChatThreadNotFound（400 / chat_thread_not_found / 当前会话不存在或不属于当前用户）
   │
   ▼
router   不处理，继续往上抛
   │
   ▼
FastAPI  @app.exception_handler(ApplicationError) 接住
   │
   ├── exc.status_code ──▶ HTTP 状态码 400
   ├── exc.code        ──▶ 错误码 chat_thread_not_found
   └── exc.message     ──▶ 错误消息 当前会话不存在或者不属于当前用户
   │
   ▼
前端     JSONResponse，三者一次拿全
```

图里能看出这套方案的价值：一个处理器，管住了所有业务异常；每个异常返回什么，由它自己在定义的时候决定。

重启服务再测一遍。传正确的 `user_id`，返回 200；传错误的，返回 400，错误消息跟刚才一样。自定义异常，返回自定义结果——到这一步，全局统一异常处理才算真正落地。

### 剩下那些异常，我们选择不管

有人会问：那系统层面的异常要不要也写一个处理器？

这次不写。只处理业务里自己抛的那一种。

举个系统异常的例子：某个参数要的是整数，你偏传一个字母。这不是我们的自定义异常，是 FastAPI 在参数校验阶段就拦下来的，它会直接告诉你类型不对。这种异常归框架管，我们不插手。

再比如传个 0。0 能通过类型校验，会一路走到查询，查不到，于是走业务分支抛 `ChatThreadNotFound`，最后还是 400——这属于业务异常，本来就在我们的处理范围里。

还有更极端的情况：传一个格式很怪的值，一路到了数据库，SQL 直接报参数错误。这是数据库层的语句异常，理论上也能捕获，但意义不大——它说明的是调用方把数据搞坏了，不是业务规则拦下来的。

这些异常我们就放任它往前抛。界线划得很清楚：**只处理自己业务里主动抛的异常**。业务异常是我们定义的、我们知道该怎么向用户解释；框架异常和数据库异常有它们自己的处理方式，硬接过来反而会把真正的问题盖住。

---

## 本册小结

本册主题为「FastAPI 企业级工程：分层架构与会话业务」，整编了 8 个模块（P32-P36 ~ P61-P63，共 32 讲 / 389 分钟音频）。
建议配合 `notes/` 目录下的复习笔记复盘，需要查证细节时可直接回到本册正文。
