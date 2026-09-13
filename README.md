<div align="center">

<a name="readme-top"></a>

<h1>Video2Book Courses</h1>

<p>
  <strong>由 Video2Book 真实生成的课程重构产物归档：四门课程的精读教材长文、模块合辑全书与思维导图复习笔记。</strong>
  <br />
  <em>精读教材长文 · 模块合辑全书 · 思维导图复习笔记 · 逐字稿 · 385 篇长文 · 57 本模块全书 · 19 篇复习笔记</em>
</p>

<p>
  <a href="https://github.com/LINJIANG12/video2book"><img src="https://img.shields.io/badge/生成工具-Video2Book-4CAF50?style=for-the-badge" alt="生成工具：Video2Book" /></a>
  <img src="https://img.shields.io/badge/来源-Bilibili-00A1D6?style=for-the-badge" alt="内容来源：Bilibili" />
</p>

<p>
  <img src="https://img.shields.io/badge/语言-简体中文-lightgrey?style=flat" alt="语言：简体中文" />
  <img src="https://img.shields.io/badge/阅读器-Typora-3178C6?style=flat" alt="推荐阅读器：Typora" />
  <img src="https://img.shields.io/badge/脑图-Markmap_|_XMind-8B5CF6?style=flat" alt="脑图支持：Markmap 与 XMind" />
</p>

</div>

> [!IMPORTANT]
> **本仓库内容由 [Video2Book](https://github.com/LINJIANG12/video2book) 生成**。所有长文与笔记均为 AI 按讲师音频内容重构所得，不是对原课程视频的转录稿，也不代表原作者观点。原课程版权归其作者与所属机构所有。

## 目录

- [项目概述](#项目概述)
- [效果预览](#效果预览)
- [收录课程](#收录课程)
- [如何使用](#如何使用)
- [生成流程](#生成流程)
- [目录结构](#目录结构)
- [生成工具链](#生成工具链)
- [常见问题](#常见问题)
- [版权与免责声明](#版权与免责声明)

## 项目概述

本仓库是 [Video2Book](https://github.com/LINJIANG12/video2book) 的产物归档，收录四门真实课程经完整流水线处理后的全部交付物。它的用途有两个：一是让想了解 Video2Book 输出质量的人直接读到成品，而不必先自己跑一遍；二是为这些课程本身留下一份可检索、可离线通读的复习资料。

仓库里的每一篇长文都来自对应分集的音频：模型听完该集音频后直接成文，因此正文保留了讲师的推导过程、板书案例与课堂比喻，而不是把视频压成要点列表。课程处理完后，多集长文再被整编为按知识模块分册的合辑教材，并被归并成跨模块的复习笔记。

三类产物各有各的粒度，这是刻意的：长文按集拆，便于定向查阅某一讲；教材按模块合，便于体系化通读；笔记按跨模块主题并，便于考前速查与脑图导入。

<div align="right">

[![返回顶部][badge-top]](#readme-top)

</div>

## 效果预览

复习笔记开头的知识拓扑树，首行写主题名与覆盖的分集范围，末级注释纵向对齐：

```text
└── Python 入门路线、开发环境搭建与基础语法体系（P01-P13）
  └── AI 时代的 Python 学习路线与职业前景 (P01)
    ├── AI 岗位爆发、DeepSeek 与政策依据 ────         纵览
    ├── 六阶段课程主线与四块实战方向 ────             纵览
    ├── 四项课程特色与学习收获 ────                   纵览
    └── 阶段化学习方法与适用人群 ────                 纵览
  └── Python 语言本体：出身、定位与应用领域 (P02)
    ├── 作者、发布年份与名字由来 ────                 纵览
    └── 人类语言与编程语言的对比 ────                 纵览
```

模块合辑教材按知识体系分册，一节一课，例如浙江大学软件工程被整编为 14 册：

```text
模块01_软件工程导论与学科范式_精读全书.md
模块02_软件过程模型体系与演进_精读全书.md
模块03_系统工程原理与需求工程基础_精读全书.md
...
模块14_敏捷软件开发与软件度量工程_精读全书.md
```

单集长文以内容为纲，保留课堂推导与类比。例如《软件工程》第一讲开篇即还原了讲师用来解释课程定位的比喻：

> 课程刻意站在一个很高的抽象高度来俯瞰全局。一个贴切的类比是：这就像"站在月球上看地球"——先看到它整体是个球体，有固体、有液体；再拉近一点，才看到山川、河流、树木、城市。

<div align="right">

[![返回顶部][badge-top]](#readme-top)

</div>

## 收录课程

| 课程 | 分集 | 精读长文 | 模块全书 | 复习笔记 | 逐字稿 |
| :--- | ---: | ---: | ---: | ---: | ---: |
| [浙江大学-软件工程-陈越](浙江大学-软件工程-陈越/) | 34 | 34 | 14 | — | 35 |
| [数据库系统概论](数据库系统概论/) | 75 | 75 | 11 | — | — |
| [黑马程序员-AI大模型NLP](黑马程序员-AI大模型NLP/) | 189 | 189 | 16 | 11 | 189 |
| [黑马程序员-Python-AI](黑马程序员-Python-AI/) | 87 | 87 | 16 | 8 | 87 |
| **合计** | **385** | **385** | **57** | **19** | **311** |

四门课程均处理自 Bilibili，原始链接与 BV 号见各课程目录下的 `README.md`。「—」表示该课程未经该阶段处理，对应目录不存在。

<div align="right">

[![返回顶部][badge-top]](#readme-top)

</div>

## 如何使用

**在线阅读**：直接在 GitHub 上点开任意 `.md` 文件即可阅读，文档内标题自动生成目录锚点。

**克隆到本地**：仓库体积约 10 MB（音频与中间件未入库），全量克隆很快。

```bash
git clone https://github.com/LINJIANG12/video2book-courses.git
```

**用 Typora 通读**：三类产物默认按 Typora 排版。若长文含行内公式，请在 Typora 的「偏好设置 → Markdown」中勾选「内联公式」，否则公式会原样显示 `$…$` 源码。

**生成脑图**：复习笔记的标题层级与拓扑树可直接导入脑图工具。VS Code 装 Markmap 插件后打开笔记文件即可预览；XMind 可通过导入 Markdown 大纲生成导图。

**检索特定知识点**：用编辑器的全库搜索（如 VS Code 的 `Ctrl+Shift+F`）在四门课程间跨库检索，长文密度高、术语统一，适合定位到具体讲解。

<div align="right">

[![返回顶部][badge-top]](#readme-top)

</div>

## 生成流程

本仓库的产物由 Video2Book 的两阶段流水线生成，全程不落中间逐字稿文件：

1. **摄取与切片**：解析 B 站合集结构，用 ffmpeg 把每集音频提取为 16kHz 单声道切片（每片不超过 60 分钟）。
2. **阶段一听音成文**：宿主模型具备原生音频模态，因此走 `read_audio` 通道——模型直接聆听音频切片，按 `learning` 风格提示词逐集写成精读长文，落盘至 `articles/`。
3. **阶段二整编归并**：先按知识体系的语义边界把集号切成模块，再跨模块归并成若干篇笔记；教材按模块分册整编至 `textbooks/`，笔记落至 `notes/`。
4. **交付前质检**：笔记与渲染各自过一遍机器门禁，套话填充、空壳标题、分集平铺标题、行内残缺引用、分集口吻五类致命项清零后方可交付。

各课程的账本字段显示 `asr_engine` 与 `doc_engine` 均为 `agent-native`，即听音与写作都由宿主原生完成，未经外部转录服务。

<div align="right">

[![返回顶部][badge-top]](#readme-top)

</div>

## 目录结构

```
video2book-courses/
├── 浙江大学-软件工程-陈越/
│   ├── README.md          # 课程导读与原始出处
│   ├── articles/          # PXX_*_精读文章.md    单集精读长文
│   ├── textbooks/         # 模块XX_*_精读全书.md  模块合辑教材
│   └── subtitles/         # PXX_*_clean.txt      逐字稿
├── 数据库系统概论/
│   ├── README.md
│   ├── articles/
│   └── textbooks/
├── 黑马程序员-AI大模型NLP/
│   ├── README.md
│   ├── articles/
│   ├── textbooks/
│   ├── notes/             # 笔记XX_*_笔记.md      跨模块复习笔记
│   └── subtitles/
└── 黑马程序员-Python-AI/
    ├── README.md
    ├── articles/
    ├── textbooks/
    ├── notes/
    └── subtitles/
```

未被收录的内容保留在生成端工作区，未进入本仓库：`audio/` 音频原件（约 3.7 GB）、`manifest.json` / `parts.json` 账本、`topic_plan.json` / `note_plan.json` 语义规划、`*_TASK.md` 任务书、`subtitles/kernels/` 知识元与各类备份文件。忽略规则见 [`.gitignore`](.gitignore)。

<div align="right">

[![返回顶部][badge-top]](#readme-top)

</div>

## 生成工具链

- **Video2Book** — 技能本体与流水线工具链，负责摄取、听音编排、两趟语义聚合与交付门禁。仓库地址 <https://github.com/LINJIANG12/video2book>。
- **omni-media** — 提供 `read_audio` / `read_media` 听音通道的 MCP 服务，本仓库产物使用的是宿主原生听音的 `read_audio` 通道。仓库地址 <https://github.com/LINJIANG12/omni-media>。

<div align="right">

[![返回顶部][badge-top]](#readme-top)

</div>

## 常见问题

### 这些内容是我可以自己复现的吗

可以。用 [Video2Book](https://github.com/LINJIANG12/video2book) 对同一课程执行一条命令即可重跑，但重跑产物不会与这里逐字相同——长文由模型撰写，措辞会随模型与提示词版本变化。

### 为什么有的课程没有笔记或逐字稿

各课程的推进进度不同。逐字稿只在部分课程保留；复习笔记需要走完阶段二的两趟语义归并，尚未处理的课程就没有 `notes/` 目录。缺失的目录不会以空壳形式占位。

### 音频文件在哪里

未入库。四门课程的音频原件合计约 3.7 GB，且可由 Video2Book 从原始链接重新提取，因此只保留文本产物。

### 笔记能直接导成脑图吗

可以。笔记开头的拓扑树使用缩进层级表达结构，Markmap 与 XMind 都能直接解析；笔记正文的标题层级同样可作为脑图节点。

<div align="right">

[![返回顶部][badge-top]](#readme-top)

</div>

## 版权与免责声明

- **内容生成方式**：本仓库全部文本由 [Video2Book](https://github.com/LINJIANG12/video2book) 基于课程音频自动生成，属于对原课程的二次整理，**不代表原作者、讲师或所属机构的观点**，也不构成对原课程内容的官方版本。
- **原始版权归属**：各课程的视频、音频、讲义与其中涉及的一切原创内容，版权均归原作者与所属机构所有。本仓库不主张对原始课程内容的任何权利。
- **使用范围**：本仓库内容仅供个人学习、复习与检索使用，**不得用于商业用途**，亦不得作为原课程的替代品传播。
- **纠错与下架**：若生成内容存在事实性错误，或原作者/权利人认为本仓库内容侵犯其权益，请通过 [Issues](https://github.com/LINJIANG12/video2book-courses/issues) 联系，将及时更正或删除相应内容。
- **本仓库无 LICENSE 文件**：文本产物不适用开源许可证；如需引用，请注明来源为原课程，而非本仓库。

<div align="right">

[![返回顶部][badge-top]](#readme-top)

</div>

<!-- LINKS & IMAGES -->

[badge-top]: https://img.shields.io/badge/-返回顶部-151515?style=flat-square
