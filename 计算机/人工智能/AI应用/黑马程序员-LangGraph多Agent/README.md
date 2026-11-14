# 黑马程序员：Python + FastAPI + LangGraph 多 Agent 企业级 AI 应用（193 讲）

从一台空虚拟机开始，把「企业级 AI 应用」从环境、后端工程一路搭到多 Agent 编排：先用 Linux 与 Docker 把部署链路打通，再用 FastAPI 的分层工程撑起业务骨架，随后接入 LangChain 的 Agent、工具调用、MCP 与会话记忆，补上 SSE 流式对话；后半程进入 RAG，从文档加载、切分、向量模型一路做到 Milvus 的稠密/稀疏/混合检索与重排，最终用父子块知识库和 LangGraph 把「保险顾问」这个真实项目做成主图 + 意图识别 + 闲聊子图 + 保险推荐的多 Agent 系统。

课程特点是「一条项目主线贯穿到底」：前半程的 FastAPI 分层与会话管理，后半程的 RAG 与图编排，最后都落回同一个保险顾问项目，不是分散的 API 演示。

- **原视频**：[黑马程序员Python+FastAPI+LangGraph多Agent搭建企业级AI应用，手把手用最新LangChain+Milvus搭建真实AI项目](https://www.bilibili.com/video/BV1M3Ws6LEVr) — Bilibili，BV1M3Ws6LEVr
- **全 193 集**，已产出 6 册模块全书、10 篇复习笔记、51 份逐字稿

## 怎么学

课程主线是「部署环境 → 后端工程 → Agent 应用 → RAG 知识库 → 多 Agent 编排」，每一段都为下一段准备零件：

- **先打通环境与部署** → 模块 01 覆盖 Linux 常用命令、Docker 镜像/容器/数据卷/Compose、Nginx 前后端分离与云服务器环境；不熟悉命令行的话，这一册不要跳
- **再吃透 FastAPI 分层骨架** → 模块 02 把分包、配置、日志、Models/Repository/Service/Router、依赖注入、统一异常串成一条完整链路，后面的会话业务与对话接口都复用它
- **Agent 部分边看边写** → 模块 03 从手写 Tool Calling 讲到 LangChain 的模型、消息、结构化输出、MCP 与 PostgreSQL 记忆，代码密度最高，建议对着教材逐个方法跑一遍
- **RAG 与 Milvus 抓两条线** → 一条是文档处理（加载、切分、向量化、VectorStore），一条是检索（稠密/稀疏/混合、加权融合、RRF、Cross-encoder、条件过滤）；模块 04 按这两条线组织
- **项目实战连续读** → 模块 05 的父子块知识库与 Retriever、模块 06 的主图与多 Agent 编排属于同一个保险顾问项目，建议从模块 05 开头一路读到模块 06 结尾，不要拆开
- **复习与速查用笔记** → 10 篇笔记按知识主题重组（工程骨架、会话业务、Agent 与记忆、RAG、Milvus、LangGraph、多 Agent），知识拓扑树可直接导入 Markmap 或 XMind

## 模块全书目录

整编为 6 册，按课程推进顺序：

| # | 模块 | 主要内容 |
| ---: | :--- | :--- |
| 01 | [环境与部署：Linux 命令、Docker 与云上发布](textbooks/模块01_环境与部署：Linux%20命令、Docker%20与云上发布_精读全书.md) | Linux 目录与文件操作、文本处理、软件安装；前后端分离与 Nginx、Docker 镜像容器数据卷、Compose 一键部署、云服务器与 Ollama 环境 |
| 02 | [FastAPI 企业级工程：分层架构与会话业务](textbooks/模块02_FastAPI%20企业级工程：分层架构与会话业务_精读全书.md) | 项目结构与分包、PydanticSettings 配置、统一日志、启动模块、开发规范、Models/Repository/Service/Router、依赖注入、会话创建/查询/重命名、统一异常 |
| 03 | [Agent 应用开发：LangChain、工具调用、记忆与流式对话](textbooks/模块03_Agent%20应用开发：LangChain、工具调用、记忆与流式对话_精读全书.md) | Agent 与 Tool Calling 原理、手写 Tool Calling、LangChain 模型初始化/消息/结构化输出/流式、工具描述、MCP stdio/HTTP、短期记忆、PostgreSQL Checkpointer、SSE 与会话历史 |
| 04 | [RAG 与 Milvus：文档处理、向量检索与混合召回](textbooks/模块04_RAG%20与%20Milvus：文档处理、向量检索与混合召回_精读全书.md) | RAG 原理与蓝图、PyPDFLoader/MinerU、切分策略、向量模型、VectorStore、2Step 与 Agentic RAG、Milvus 索引与 Collection/Entity、稠密/稀疏/混合检索、加权融合/RRF/Cross-encoder、条件过滤 |
| 05 | [LangGraph 图编排：从核心概念到保险顾问 RAG 实战](textbooks/模块05_LangGraph%20图编排：从核心概念到保险顾问%20RAG%20实战_精读全书.md) | 保险条款父子块设计、文档解析、切分入库、知识库 Pipeline、父块召回与 Retriever、RAG 写入 AgentTool；LangGraph 的 Node/State/Reducer/边/Command、构建 Agent、Workflow、路由与子图嵌套 |
| 06 | [多 Agent 编排：意图识别、闲聊子图与保险推荐接入](textbooks/模块06_多%20Agent%20编排：意图识别、闲聊子图与保险推荐接入_精读全书.md) | 保险顾问主图的 State/构建/生命周期、意图识别工具、寒暄识别与兜底、闲聊子 Agent、subgraph 流事件处理、保险推荐 Agent 接入 |

## 复习笔记目录

10 篇按知识主题重组，可跨模块检索：

| # | 笔记 | 覆盖内容 |
| ---: | :--- | :--- |
| 01 | [Linux 环境、Docker 容器与项目部署](notes/笔记01_Linux环境Docker容器与项目部署_笔记.md) | Linux 命令与文本处理、Nginx 部署、镜像与容器、数据卷、Dockerfile 与 Compose |
| 02 | [企业级 FastAPI 工程骨架：分包、配置、日志与分层架构](notes/笔记02_企业级FastAPI工程骨架分包配置日志与分层架构_笔记.md) | 项目分包、PydanticSettings、统一日志、启动模块、分层架构、依赖注入、Pydantic 转换 |
| 03 | [会话管理业务：数据建模、接口分层与统一异常处理](notes/笔记03_会话管理业务数据建模接口分层与统一异常处理_笔记.md) | 会话表 Models/schemas、Repository/Service/Router、创建/查询/重命名链路、统一异常 |
| 04 | [Agent 与 Tool Calling：原理、LangChain 基础与 MCP 接入](notes/笔记04_Agent与ToolCalling原理LangChain基础与MCP接入_笔记.md) | Tool Calling 原理与手写、模型初始化与消息、结构化输出与流式、工具描述、MCP stdio/HTTP |
| 05 | [会话记忆与流式对话：PostgreSQL 持久化、SSE 与会话历史](notes/笔记05_会话记忆与流式对话PostgreSQL持久化SSE与会话历史_笔记.md) | Checkpointer 与生命周期、基础对话接口、SSE 数据格式与流式输出、历史查询/删除 |
| 06 | [RAG 基础：原理、文档处理与向量检索](notes/笔记06_RAG基础原理文档处理与向量检索_笔记.md) | RAG/向量检索原理、文档加载与切分策略、向量模型、VectorStore、2Step 与 Agentic RAG |
| 07 | [Milvus 向量数据库：索引、读写与混合检索](notes/笔记07_Milvus向量数据库索引读写与混合检索_笔记.md) | 索引与 Collection、Entity 读写、稠密/稀疏检索、加权融合与 RRF、Cross-encoder、条件过滤 |
| 08 | [保险条款 RAG：父子块知识库构建与检索工具化](notes/笔记08_保险条款RAG父子块知识库构建与检索工具化_笔记.md) | 父子块设计、解析与切分入库、Pipeline、父块召回 Retriever、RAG 工具化 |
| 09 | [LangGraph 核心：State、控制流与 Agent 构建](notes/笔记09_LangGraph核心State控制流与Agent构建_笔记.md) | Node 与 State、默认/自定义 Reducer、普通边/条件边/Command、构建 Agent、Workflow 与子图 |
| 10 | [多 Agent 编排：意图识别、闲聊子图与保险推荐接入](notes/笔记10_多Agent编排意图识别闲聊子图与保险推荐接入_笔记.md) | 主图 State 与生命周期、意图识别、寒暄兜底、闲聊子图、保险推荐接入 |

笔记开头的知识拓扑树可直接导入 Markmap 或 XMind 生成脑图。

## 其他入口
- [逐字稿（51 份）](subtitles/) — 核对讲师原话，以及开发适合自己习惯的课程

---

> 本目录内容由 [Video2Book](https://github.com/LINJIANG12/video2book) 生成，属于对原课程的二次整理，
> 不代表原作者与所属机构的观点。原课程版权归原作者与所属机构所有，仅供个人学习使用。

[← 返回仓库首页](../../../../README.md)
