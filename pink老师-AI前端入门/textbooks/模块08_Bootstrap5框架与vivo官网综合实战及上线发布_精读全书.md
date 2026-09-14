# 模块 08：Bootstrap5框架与vivo官网综合实战及上线发布 合辑教材

> **所属课程**：2025版pink老师最新AI+前端入门教程，零基础必看的html5、css3、grid、flex布局、响应式、移动端，bootstrap5框架，AI工具  
> **模块跨度**：P183 ~ P200（全模块共 18 讲系统重构）  
> **内容定位**：模块化系统学习教材，融合核心机制、架构全景、代码解析与思考自测。  
> **关联说明**：本册由各集精读讲义整编而成；单集讲义属生成侧中间产物，不随本仓库发布。

---

## 模块导读与全景目录

- **第 1 章**：182-Bootstarp框架介绍和引用
- **第 2 章**：183-Bootstarp框架基本使用
- **第 3 章**：184-Bootstrap 栅格布局
- **第 4 章**：185-Bootstrap 列布局和字体图标
- **第 5 章**：186-vivo响应式官网-基础准备工作以及TDK三大优化标签
- **第 6 章**：187-vivo响应式官网-navbar导航栏制作（上)
- **第 7 章**：188-vivo响应式官网-navbar导航栏制作（下）
- **第 8 章**：189-vivo响应式官网-轮播图模块制作以及图片遮罩
- **第 9 章**：190-vivo响应式官网-热门产品模块搭建
- **第 10 章**：191-vivo响应式官网-热门产品卡片模块（上）
- **第 11 章**：192-vivo响应式官网-热门产品卡片模块（下）
- **第 12 章**：193-vivo响应式官网-未来科技模块制作
- **第 13 章**：194-vivo响应式官网-底部模块制作（上）
- **第 14 章**：195-vivo响应式官网-底部模块制作（中）
- **第 15 章**：196-vivo响应式官网-底部模块制作（下）
- **第 16 章**：197-vivo响应式官网-媒体查询和自定义滚动条样式
- **第 17 章**：198-前端网页托管-发布自己的网站
- **第 18 章**：199-结尾篇-致前端者mv

---

## 第 1 章：182-Bootstarp框架介绍和引用
> 对应分集：P183 | 原始标题：《182-Bootstarp框架介绍和引用》

手写完 DesignCode 那套响应式，我最大的感受就一个字：累。媒体查询、栅格、按钮、表单，每个都要自己从零调。你们是不是也在想：大厂天天做后台、官网，难道每次都手写一遍？当然不是，他们都站在框架肩膀上。这一讲我们就认识第一个重磅框架。

我刚入行那会儿也抵触框架，觉得用框架是“偷懒”。后来接了个三天上线的后台，才明白：框架不是偷懒，是把前人踩过的兼容坑一次性躲过去。`Bootstrap` 就是这类框架里资历最老、文档最全的一个。

---

### 1. 前端框架与 Bootstrap 的由来

> **定义**：前端框架（`Framework`）指封装好布局、组件与 `JS` 插件的半成品页面库，我们只写结构与类名，就能拼出风格统一、兼容良好的界面。

`Bootstrap`（简称 `bs`）最初是 `Twitter` 内部用的样式库，后来开源，一路做到第 5 版，也就是我们现在学的 `Bootstrap 5`。记住这一条就够了：它火不是因为花哨，而是因为稳：类名语义清晰，断点齐全，兼容性有人替你测过。

```text
手写响应式 vs Bootstrap 开发路径
手写: 写结构 -> 写栅格 -> 写组件 -> 调兼容 (全自己来)
Bootstrap: 写结构 + 套类名 -> 微调 -> 上线 (栅格组件现成)
```

读图说明：框架把重复度最高的栅格与组件沉淀下来，我们只剩业务结构与微调。

| 对比维度 | 手写 CSS | Bootstrap |
| :--- | :--- | :--- |
| 栅格系统 | 自己算百分比 + 媒体查询 | `row + col` 直接 12 等分 |
| 按钮表单 | 逐个调边框圆角 | `btn btn-primary` 一行搞定 |
| 兼容与文档 | 靠经验踩坑 | 官方文档即查即用 |

> **提示**：框架解决的是“快而稳”，不是“独一无二”。要做高度定制视觉稿，依然要自己写 `CSS` 覆盖，不要指望一套类名走天下。

---

### 2. 版本选择与两种引用方式

版本这里容易搞错。市面上还能搜到 `Bootstrap 3` 的老教程，里面用 `float` 做栅格，还依赖 `jQuery`。我们直接学 `Bootstrap 5`：栅格换成 `flex`，不再强依赖 `jQuery`，体积与写法都更现代。认准 `5.x`，别跟着老教程走偏了。

引用有两种，我两种都演示一遍，你们按场景选。

#### 2.1 本地包引入

去官网下好编译包，解压拿到 `css/bootstrap.min.css` 与 `js/bootstrap.bundle.min.js`，放进自己项目：

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Bootstrap 本地引入</title>
  <link rel="stylesheet" href="./bootstrap/css/bootstrap.min.css">
</head>
<body>
  <h1>本地引入成功</h1>
  <script src="./bootstrap/js/bootstrap.bundle.min.js"></script>
</body>
</html>
```

本地的好处是离线也能跑，上线稳定。注意 `CSS` 放头部，`JS` 放 `body` 尾部，这个顺序绝不能反。

#### 2.2 CDN 在线引用

学习阶段我更推荐 `CDN`（内容分发网络，`Content Delivery Network`），复制两行就行，不用下包：

```html
<link
  href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css"
  rel="stylesheet">
<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js"></script>
```

> **易错点**：`link` 缺了 `rel="stylesheet"`，样式静默失效，页面还是裸 `HTML`，很多人来回刷新还以为是 `CDN` 挂了，其实就差这一个属性。

验证是否引用成功很简单：写一个 `class="btn btn-primary"` 的按钮，刷新看到蓝色圆角按钮就是成了。若还是默认灰按钮，打开控制台看 `Network`，一定是路径或版本号写错了。

---

### 3. 第一个 Bootstrap 页面模板

引用成功后，标准起手式是这个模板，你们存成片段，以后每个页面都从它开始：

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>BS5 起手模板</title>
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet">
  <style>
    /* 自己的微调永远放框架之后, 才能覆盖生效 */
    .demo-box {
      background: #f1f5ff;
    }
  </style>
</head>
<body>
  <div class="container demo-box">
    <h1>你好, Bootstrap 5</h1>
    <button class="btn btn-primary">主要按钮</button>
  </div>
  <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
```

这里埋着一个关键习惯：自己的 `style` 一定要压在 `bootstrap.min.css` 后面。层叠规则摆着，谁后谁赢，你把覆盖写前面，写了也白写。

`container`（版心容器）是全套栅格的地基，下一讲我们就从它开始正式用组件。到这里你们先做到：能说清 `Bootstrap` 是干什么的，能用两种方式把它引进来并验证成功，后面学栅格才不飘。

> 💡 **承前启后**：完成对「182-Bootstarp框架介绍和引用」的理解后，下一章我们将深入探讨「183-Bootstarp框架基本使用」，进一步完善知识图谱体系。

---

## 第 2 章：183-Bootstarp框架基本使用
> 对应分集：P184 | 原始标题：《183-Bootstarp框架基本使用》

上一讲把 `Bootstrap 5` 引进来，页面还是光秃秃的。很多人到这步就懵了：包是引了，可到底怎么“用”？我的体会是，`Bootstrap` 的用法就一句话：结构自己写，样式靠类名堆。你们别急着背所有类，先把最常用的容器、按钮、表格、表单摸熟，就能应付八成页面。

---

### 1. 版心容器与查文档习惯

所有 `Bootstrap` 页面都从容器开始。容器（`Container`）就是居中的版心，替我们管最大宽度与两侧留白。

```html
<div class="container" style="background:#eef3ff;">
  版心容器: 随断点定宽, 居中
</div>
<div class="container-fluid" style="background:#fff3e0;">
  流式容器: 永远 100% 宽
</div>
```

```text
容器宽度行为 (由小到大拖动)
container-fluid : |---------------- 100% ----------------|
container       :   |--- 540 ---|--- 720 ---|--- 960 ---|--- 1140 ---|
                  576px      768px       992px       1200px 断点定宽
```

读图说明：`container-fluid` 永远铺满，`container` 在每个断点取固定最大宽度并居中。

> **定义**：断点（`Breakpoint`）指 `Bootstrap` 预设的响应宽度分界，如 `576px`、`768px`、`992px`、`1200px`，容器与栅格都依附它们切换。

我建议你们从第一天就养成查文档的习惯：`Bootstrap` 官网左侧即是组件目录，点进去每个组件都有可复制的示例。不要背，记住“容器管宽、行列管分、组件管样”这个分工，缺什么查什么，比死记硬背快得多。

> **提示**：自己的覆盖样式永远写在 `bootstrap.min.css` 之后，否则会被框架样式压住，改了也没效果。

---

### 2. 按钮与表格组件

按钮是出场率最高的组件。`Bootstrap` 把按钮拆成两层类：`btn` 管基础形状，`btn-*` 管颜色语义。

```html
<button class="btn btn-primary">主要操作</button>
<button class="btn btn-success">提交成功</button>
<button class="btn btn-danger">删除</button>
<button class="btn btn-outline-primary">描边按钮</button>
<button class="btn btn-primary btn-lg">大号按钮</button>
```

表格同理，一行类名就有斑马纹与悬停高亮，手写至少十几行：

```html
<table class="table table-striped table-hover table-bordered">
  <thead>
    <tr>
      <th>姓名</th>
      <th>方向</th>
      <th>进度</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>小白</td>
      <td>前端</td>
      <td>HTML 已完结</td>
    </tr>
    <tr>
      <td>小黑</td>
      <td>前端</td>
      <td>CSS 进行中</td>
    </tr>
  </tbody>
</table>
```

| 类名 | 作用 | 手写替代成本 |
| :--- | :--- | :--- |
| `table-striped` | 隔行变色 | `nth-child` 手写 |
| `table-hover` | 悬停高亮行 | `tr:hover` 手写 |
| `table-bordered` | 全边框 | 逐格调边框 |
| `btn-lg / btn-sm` | 尺寸档 | 逐个调 padding |

> **易错点**：只写 `btn-primary` 不写 `btn`，按钮形状出不来。`btn` 是地基，颜色类是油漆，缺了地基，油漆无处刷。

---

### 3. 表单控件与焦点样式

表单是 `Bootstrap` 最省事的地方。输入框加 `form-control`，下拉加 `form-select`，复选加 `form-check-input`，焦点光晕、圆角、间距全齐了。

```html
<form>
  <div class="mb-3">
    <label class="form-label">邮箱</label>
    <input type="email" class="form-control" placeholder="name@example.com">
  </div>
  <div class="mb-3">
    <label class="form-label">方向</label>
    <select class="form-select">
      <option>前端</option>
      <option>后端</option>
      <option>全栈</option>
    </select>
  </div>
  <div class="form-check mb-3">
    <input class="form-check-input" type="checkbox" id="agree">
    <label class="form-check-label" for="agree">记住我</label>
  </div>
  <button class="btn btn-primary">登录</button>
</form>
```

`mb-3` 是间距工具类，`m` 表外边距，`b` 表下方，`3` 表档位。你们刚开始记不住档位没关系，记住规律：`m/p + t/b/s/e/x/y + 0~5`，调间距再也不用随手写内联样式了。

```text
表单控件类名分工
form-label  -> 标题文本样式
form-control -> 文本框/密码框/邮箱框统一样式
form-select  -> 下拉框样式
form-check-* -> 复选/单选的框与文组合
```

读图说明：每种控件各有专属类，对号入座即可，不要把 `form-control` 套到 `select` 上。

这一讲你们亲手敲一遍：一个版心里放一行按钮、一个条纹表格、一个登录表单，全部只用类名实现，不写一行组件 `CSS`。能做到这一点，`Bootstrap` 的基本使用就过关了，下一讲我们啃它最核心的栅格布局。

> 💡 **承前启后**：完成对「183-Bootstarp框架基本使用」的理解后，下一章我们将深入探讨「184-Bootstrap 栅格布局」，进一步完善知识图谱体系。

---

## 第 3 章：184-Bootstrap 栅格布局
> 对应分集：P185 | 原始标题：《184-Bootstrap 栅格布局》

前面容器、按钮、表格都会套了，可一到“左边三分、右边七分，手机上又要上下堆”就卡壳。你们是不是也觉得，响应式最烦的就是算百分比？我当年也是拿着计算器算 `33.333%` 的，直到吃透栅格：以后再也不用算了，`Bootstrap` 把一行切成 12 份，我们只管说“占几份”。

---

### 1. 栅格三件套与 12 等分原理

栅格（`Grid`）的三件套是固定的：`container` 做版心，`row` 做行，`col-*` 做列。缺谁都不行。

> **定义**：栅格列（`Column`）指 `row` 内按 12 等分划分的纵向槽位，类名中的数字即占几份，如 `col-4` 占 `4/12`。

```html
<div class="container">
  <div class="row">
    <div class="col-4" style="background:#dbe7ff;">占 4 份</div>
    <div class="col-8" style="background:#ffe9d6;">占 8 份</div>
  </div>
  <div class="row">
    <div class="col-3" style="background:#e7f8ec;">3 份</div>
    <div class="col-6" style="background:#f3e8ff;">6 份</div>
    <div class="col-3" style="background:#fff4c2;">3 份</div>
  </div>
</div>
```

```text
12 等分切分示意 (一行 = 12 份)
|--- col-4 (4) ---|-------- col-8 (8) --------|
|--3--|----- 6 -----|--3--|
 0   3   4         10  12  刻度
```

读图说明：同属一个 `row` 的列份数相加为 12 即铺满一行，超过 12 会自动换行。

> **结论**：一行内份数总和按 12 配，`4+8`、`3+6+3`、`6+6` 都是满行。一定要如此，绝不能那般：列必须包在 `row` 里，`row` 必须包在 `container` 里，跨级嵌套必出多余边距。

`row` 自带负边距吃掉列的内边距，列与列之间才有整齐的槽（`gutter`）。你们要是把 `col` 直接丢进 `container`，两侧对不齐，罪魁祸首就是少了这层 `row`。

---

### 2. 响应断点与列类组合

光会等分还不够，响应式的灵魂是断点：大屏三列，小屏一列。`Bootstrap` 的列类可以按断点叠加，写法是 `col-断点-份数`。

| 断点前缀 | 适用宽度 | 记忆法 |
| :--- | :--- | :--- |
| `col-` / `col-xs` 隐含 | `< 576px` 超小屏 | 手机竖屏默认 |
| `col-sm-*` | `>= 576px` | 小屏手机横屏 |
| `col-md-*` | `>= 768px` | 平板 |
| `col-lg-*` | `>= 992px` | 小桌面 |
| `col-xl-*` | `>= 1200px` | 大桌面 |
| `col-xxl-*` | `>= 1400px` | 超宽屏 |

实战里最常用的就是 `col-md-*`：手机上堆叠，平板以上分栏。

```html
<div class="container">
  <div class="row">
    <div class="col-12 col-md-4" style="background:#dbe7ff;">课程介绍</div>
    <div class="col-12 col-md-4" style="background:#e7f8ec;">课程目录</div>
    <div class="col-12 col-md-4" style="background:#ffe9d6;">学员评价</div>
  </div>
</div>
```

这行代码你们要读出两层意思：手机下每个 `col-12` 独占一行，上下堆；宽度一过 `768px`，`col-md-4` 接管，三列并排。为什么是 `768px`？不是 `700`、`800`？因为框架的 `md` 断点就定在 `768px`，跟着框架走，团队协作才对得上。

```text
同一行在不同宽度下的表现
>=768px : [ 4份 ][ 4份 ][ 4份 ]  一行并排
< 768px : [     12份     ]  堆叠
          [     12份     ]
          [     12份     ]
```

读图说明：断点是开关，宽度跨过 `768px` 即切换排列，小屏优先写堆叠，大屏用断点类覆盖。

> **易错点**：只写 `col-md-4` 不写 `col-12`，手机下三列会被硬挤在一行，字都叠在一起。移动优先：先写小屏的份数，再用断点类往大屏覆盖。

---

### 3. 列偏移嵌套与对齐微调

分栏会了，再学三招收尾活：偏移、嵌套、 Space 控制。

偏移（`Offset`）是让列往右空出几份，常用来做居中：

```html
<div class="row">
  <div class="col-md-6 offset-md-3" style="background:#dbe7ff;">
    占 6 份, 左空 3 份, 天然居中
  </div>
</div>
```

嵌套是列里再套行列，做卡片内部分栏：

```html
<div class="row">
  <div class="col-md-8">
    <div class="row">
      <div class="col-6">左小图</div>
      <div class="col-6">右文字</div>
    </div>
  </div>
  <div class="col-md-4">右侧边栏</div>
</div>
```

```css
/* 间距不够时的微调: 宁用 gutter 与工具类, 少写死 margin */
.tight-row {
  --bs-gutter-x: 12px;
}
```

> **提示**：调列间距优先改 `--bs-gutter-x` 或加 `g-*` 工具类，直接给 `col` 写死 `margin` 会撑破 12 等分，最后一列被挤到下一行，有苦说不出。

验收照我这个来：桌面端三列并排，平板两列加一列换行，手机单列堆叠，居中区块在各宽度下都不贴边。跑通这一套，你们以后做官网、后台、商城列表，布局半小时就能搭出骨架，剩下的就是填内容与调细节了。

> 💡 **承前启后**：完成对「184-Bootstrap 栅格布局」的理解后，下一章我们将深入探讨「185-Bootstrap 列布局和字体图标」，进一步完善知识图谱体系。

---

## 第 4 章：185-Bootstrap 列布局和字体图标
> 对应分集：P186 | 原始标题：《185-Bootstrap 列布局和字体图标》

你们有没有遇到过这种尴尬：栅格明明是 12 等分，我只想放 3 个盒子，每个占 4 份，铺满倒是铺满了，可我想让它居中、想让它错开、想让它在手机上两列、电脑上四列，一写就乱。我跟你们讲，前面我们把 `container`、`row`、`col` 跑通了，只能算进了 Bootstrap 的门，这一节我们把列玩活，再把图标这件事一次解决。

### 1.1 十二列轨道的再理解

我一直跟你们强调，先别急着写类名，先在脑子里装一张图。Bootstrap 的一行就是 12 条轨道，你写 `col-4` 不是宽度 33%，而是占 4 条轨道。

```text
+----+----+----+----+----+----+----+----+----+----+----+----+
| 1  | 2  | 3  | 4  | 5  | 6  | 7  | 8  | 9  | 10 | 11 | 12 |
+----+----+----+----+----+----+----+----+----+----+----+----+
|        col-4        |        col-4        |      col-4     |
+----------------------------------------------------------+
|              col-6              |          col-6           |
+----------------------------------------------------------+
```

读图说明：一行共 12 格，`col-4 + col-4 + col-4` 恰好铺满，`col-6 + col-6` 也是铺满，超了就掉下去。

> **定义**：栅格轨道（网格轨道，英文 grid track）指 `row` 内按 12 等分划分的虚拟列，`col-*` 声明占几轨。

你们记住一条就够了：**同一行内数字加起来不要超 12**。超了，最后那个盒子会被挤到下一行，这不是 bug，这是换行规则。

```html
<div class="container">
  <div class="row">
    <div class="col-4">内容 A</div>
    <div class="col-4">内容 B</div>
    <div class="col-4">内容 C</div>
  </div>
</div>
```

上面这段你们闭眼都该写出来。`container` 定最大宽度并居中，`row` 消掉左右的槽宽，`col-4` 分轨道。三层缺一不可。

### 1.2 列偏移与空白控制

问题来了，我只想放一个 6 宽的盒子，还想让它居中，怎么办？你们千万别去写 `margin: 0 auto`，Bootstrap 给了偏移（偏移，英文 `offset`）。

```html
<div class="row">
  <div class="col-6 offset-3">居中的盒子</div>
</div>
```

`offset-3` 的意思是左边空出 3 条轨道。6 + 3 + 3，左右各剩 3，自然就居中了。为什么是 3？不是 2、不是 4？你们算一下：12 - 6 = 6，一边一半就是 3。这就是我反复问你们的那句：为什么是这个数，你得说得出来。

再看一个错开的效果，左边一个小图，右边一大段文字，右边我想往右再推一点：

```html
<div class="row">
  <div class="col-3">图片</div>
  <div class="col-7 offset-1">文字介绍</div>
</div>
```

3 + 1 + 7 = 11，还剩 1 格当呼吸。页面一下就不挤了。

> **易错点**：`offset` 是加在自己身上的，不是加在别人身上的。写在第二个盒子上，空的是它左边的缝。

偏移也有断点版，`offset-md-3` 的意思就是：手机上不偏移，平板以上才偏移。你们做“手机铺满、电脑居中”就靠它：

```html
<div class="row">
  <div class="col-12 col-md-6 offset-md-3">手机满行，电脑居中</div>
</div>
```

### 1.3 整行快速等分与列排序

一个一个数 `col-4` 太累了，Bootstrap 给了整行等分：`row-cols-*` 写在 `row` 上，子项自动平分。

```html
<div class="row row-cols-2 row-cols-md-4">
  <div class="col">卡片 1</div>
  <div class="col">卡片 2</div>
  <div class="col">卡片 3</div>
  <div class="col">卡片 4</div>
</div>
```

手机上一行 2 个，平板以上一行 4 个，子项只写 `col` 就够了。你们做商品列表、图标宫格，记住这一招就够了。

排序（排序，英文 `order`）是另一个救命的东西。SEO 要求文字在前、图片在后，可设计稿偏偏要图片在左。怎么办？结构不动，只调 `order`：

| 类名 | 作用 | 常用场景 |
| :--- | :--- | :--- |
| `order-1` ~ `order-5` | 数字越小越靠前 | 图文对调 |
| `order-first` | 抢到最前 | 重要内容提前 |
| `order-last` | 沉到最后 | 次要信息垫后 |
| `order-md-2` | 断点版排序 | 手机电脑顺序不同 |

```html
<div class="row">
  <div class="col-md-6 order-md-2">图片（结构在后，显示在右）</div>
  <div class="col-md-6 order-md-1">文字（结构在前，显示在左）</div>
</div>
```

> **提示**：`order` 只改变视觉顺序，不改变 DOM 顺序，读屏器和搜索引擎看到的还是源码顺序，这正是我们想要的。

### 1.4 列的对齐与槽宽调节

`row` 本质是弹性盒（弹性盒，英文 `flexbox`），所以对齐那一套全能用。水平方向用 `justify-content-*`，垂直方向用 `align-items-*`。

```html
<div class="row justify-content-center">
  <div class="col-3">居中的一小块</div>
</div>
<div class="row align-items-center" style="height: 200px;">
  <div class="col-4">高矮不一</div>
  <div class="col-4">也能垂直居中</div>
</div>
```

槽宽（槽宽，英文 `gutter`）用 `g-*`、`gx-*`、`gy-*` 调，`g-0` 去掉缝，`g-4` 拉开缝：

```text
g-0        紧贴，无缝
g-2        默认偏紧
g-3        常规卡片间距
g-4/g-5    海报式大呼吸
```

读图说明：`g` 同时管横竖，`gx` 只管左右，`gy` 只管上下，按需选用。

> **结论**：等分找 `row-cols`，留白找 `offset`，调序找 `order`，对齐找 `justify/align`，调缝找 `g`。五个各管一摊，别混着用。

### 1.5 字体图标的引入与使用

好，列玩活了，我们解决图标。以前你们切个搜索放大镜还得找 UI 要 png，现在一行类名就出来了。字体图标（字体图标，英文 `icon font`）本质是字体文件，每个字形长得像图标。

Bootstrap 官方那套叫 Bootstrap Icons，先引 CSS，再用 `i` 标签加 `bi-*` 类：

```html
<link rel="stylesheet" href="./css/bootstrap-icons.min.css">
<i class="bi bi-search"></i>
<i class="bi bi-person"></i>
<i class="bi bi-cart3"></i>
<i class="bi bi-list"></i>
```

你们看，`bi` 是前缀，`bi-search` 是具体图标。记不住名字？去官网搜英文单词，搜 `search`、`phone`、`heart`，点进去复制类名就行。

图标大小和颜色怎么调？记住：**它就是文字**，`font-size` 调大小，`color` 调颜色：

```css
.icon-search {
  font-size: 20px;
  color: #333;
}
.icon-search:hover {
  color: #415fff;
}
```

```html
<a href="#" class="nav-icon"><i class="bi bi-search"></i></a>
<a href="#" class="nav-icon"><i class="bi bi-person"></i></a>
```

> **易错点**：图标不显示，九成是 CSS 路径错了，或者 `fonts` 文件夹没一起拷过来。`bootstrap-icons.min.css` 引用的字体文件是相对路径，整个文件夹要原样搬，别只拷一个 CSS。

实战里我们常把图标和列结合：四个等分列，每列一个图标加一句文案，手机两列、电脑四列：

```html
<div class="container">
  <div class="row row-cols-2 row-cols-md-4 g-3 text-center">
    <div class="col"><i class="bi bi-truck"></i><p>顺丰包邮</p></div>
    <div class="col"><i class="bi bi-shield-check"></i><p>正品保障</p></div>
    <div class="col"><i class="bi bi-arrow-repeat"></i><p>七天退换</p></div>
    <div class="col"><i class="bi bi-headset"></i><p>在线客服</p></div>
  </div>
</div>
```

这一节你们回去练两件事：第一，不用计算器，`offset` 手算一次居中；第二，把你们页面里所有 png 小图标换成 `bi` 图标，鼠标悬停变一次色。下一节我们就开新项目，用这些本事搭一个真正的响应式官网。

> 💡 **承前启后**：完成对「185-Bootstrap 列布局和字体图标」的理解后，下一章我们将深入探讨「186-vivo响应式官网-基础准备工作以及TDK三大优化标签」，进一步完善知识图谱体系。

---

## 第 5 章：186-vivo响应式官网-基础准备工作以及TDK三大优化标签
> 对应分集：P187 | 原始标题：《186-vivo响应式官网-基础准备工作以及TDK三大优化标签》

我跟你们讲，做官网最忌讳一上来就写导航、写轮播。我见过太多人，HTML 写了 200 行，忽然发现没引 Bootstrap 的 JS，折叠菜单点不动，又回去翻。听我的，先花 10 分钟把地基打牢：文件怎么放、库怎么引、标题和优化标签怎么写。地基稳了，后面几节我们搭导航、搭轮播都是往上堆积木。

### 1.1 项目目录与基础文件准备

我们先定规矩。一个官网，哪怕是练习，也得有个像样的架子：

```text
vivo-project/
  |- index.html
  |- css/
  |    |- bootstrap.min.css
  |    |- bootstrap-icons.min.css
  |    +- index.css
  |- js/
  |    +- bootstrap.bundle.min.js
  +- images/
       +- logo.png
```

读图说明：`index.html` 只管结构，`css` 放样式，`js` 放行为，`images` 放素材，自己写的永远和库文件分开。

为什么要把 `bootstrap.min.css` 下到本地？你们用 CDN 当然也能跑，可我建议你们第一遍跟着我用本地文件。断网了也能写，不受网速影响，出了问题你知道去哪个文件夹找。

`index.css` 是我们自己的覆盖样式，一定要放在 Bootstrap 样式之后引入。谁在后，谁说了算，这就是层叠（层叠，英文 `cascade`）最直白的一条。

```html
<link rel="stylesheet" href="./css/bootstrap.min.css">
<link rel="stylesheet" href="./css/bootstrap-icons.min.css">
<link rel="stylesheet" href="./css/index.css">
```

JS 也一样，`bootstrap.bundle.min.js` 是包含了弹窗、折叠、轮播行为的那一份，放在 `body` 结束之前引入：

```html
<script src="./js/bootstrap.bundle.min.js"></script>
```

> **易错点**：只引 CSS 不引 JS，页面看着正常，一点汉堡菜单、一点轮播就没反应。新手九成卡在这里，先把 JS 引上。

### 1.2 视口标签与字符编码声明

骨架第二件事，把 `head` 里的三行写对。先看完整开头：

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>vivo 官方网站</title>
</head>
```

`charset="UTF-8"` 管中文不乱码，`lang="zh-CN"` 告诉浏览器这是中文页。视口（视口，英文 `viewport`）这一行决定了手机端按不按 375 宽度渲染，少了它，响应式全白费。

> **结论**：`width=device-width` 让布局视口等于设备宽度，`initial-scale=1.0` 让初始缩放为 1。这行是响应式的命根子，做移动和响应式必写，背下来。

### 1.3 TDK 三大优化标签写法

好，重头戏来了。什么叫 TDK？就是 `Title`、`Description`、`Keywords` 三个字母开头，搜索引擎看排名先看它们。你们做官网不写这个，等于开了店不挂招牌。

```html
<title>vivo官网-vivo X100系列新品|拍照旗舰手机官方正品</title>
<meta name="description" content="vivo官方网站提供X系列、S系列、Y系列智能手机，官方正品保障，顺丰包邮，七天无理由退换。">
<meta name="keywords" content="vivo,vivo官网,vivo手机,X100,拍照手机,智能手机">
```

三者的分工不一样，我给你们拆开：

| 标签 | 给谁看 | 写法要求 |
| :--- | :--- | :--- |
| `title` 标题 | 用户和搜索结果 | 品牌词 + 核心产品 + 卖点，30 字内 |
| `description` 描述 | 搜索结果摘要 | 一句话说清卖什么、有何保障，80 字内 |
| `keywords` 关键词 | 搜索引擎分类 | 6~8 个词，逗号分隔，别堆砌 |

你们看我这个标题：`vivo官网` 是品牌，`X100系列新品` 是核心词，`拍照旗舰` 是卖点。为什么这么排？用户搜“vivo X100”，标题里正好全命中，排名和点击率都高。

> **提示**：关键词（关键词，英文 `keywords`）现在权重不如以前了，但照写不吃亏。切忌堆二十个词，搜素引擎会判定作弊，得不偿失。

描述也别写空话。“欢迎光临本站”这种等于没写。要写人话：卖什么系列、正品吗、包邮吗、几天退换。用户在搜索结果里瞄一眼就决定点不点你家。

### 1.4 网站图标与版心注释分区

还有两件小事，顺手做了。网站图标（图标，英文 `favicon`）就是浏览器标签页左边那个小 logo：

```html
<link rel="shortcut icon" href="./images/favicon.ico">
```

一张 `32x32` 的 `ico` 文件就行。有了它，你的网站在收藏夹里一眼就能认出来。

然后在 `body` 里先把分区注释写好，人还没开工，楼层先标清：

```html
<body>
  <!-- 01 顶部导航栏 -->
  <!-- 02 轮播图模块 -->
  <!-- 03 热门产品模块 -->
  <!-- 04 未来科技模块 -->
  <!-- 05 底部模块 -->
</body>
```

> **定义**：版心（容器，英文 `container`）指页面内容的最大宽度居中区，vivo 官网一般取 `1200px` 左右，Bootstrap 里用 `container` 类实现。

```css
/* index.css 顶部先写三件事 */
* { margin: 0; padding: 0; box-sizing: border-box; }
a { text-decoration: none; }
ul { list-style: none; margin: 0; padding: 0; }
```

初始化、去下划线、去小圆点，这三板斧先抡下去，后面覆盖 Bootstrap 默认样式才顺手。记住，`index.css` 永远在 Bootstrap 之后引入，我们写的才压得住库里的。

这一节没什么花活，但缺一样后面都得返工。你们回去照着把目录建好、TDK 写好，标题别空着，描述别糊弄。下一节，我们正式搭导航栏。

> 💡 **承前启后**：完成对「186-vivo响应式官网-基础准备工作以及TDK三大优化标签」的理解后，下一章我们将深入探讨「187-vivo响应式官网-navbar导航栏制作（上)」，进一步完善知识图谱体系。

---

## 第 6 章：187-vivo响应式官网-navbar导航栏制作（上)
> 对应分集：P188 | 原始标题：《187-vivo响应式官网-navbar导航栏制作（上)》

你们打开 vivo 官网看一眼，最顶上那条黑条：左边 logo，中间一排产品、商城、服务，右边搜索和人头图标，手机上缩成一个汉堡按钮，一点就展开。很多人觉得这个难，其实它就是 Bootstrap 的导航（导航，英文 `navbar`）组件换了层皮。这一节我们不管美化，先把骨头搭对，点开了、不断行、不错位，才有资格谈好看。

### 1.1 导航栏整体结构拆解

我带你们先把结构背下来，一共四层，少一层都转不动：

```text
nav.navbar
  +- div.container
       +- a.navbar-brand        logo
       +- button.toggler        汉堡按钮
       +- div.collapse
            +- ul.navbar-nav    菜单项
```

读图说明：`nav` 是外壳，`container` 定宽居中，品牌、按钮、折叠区三者并列，菜单列表藏在折叠区里。

你们记住这个顺序：品牌在前，按钮在中，折叠在后。顺序写反了，小屏下按钮会跑到奇怪的位置。

先写外壳：

```html
<nav class="navbar navbar-expand-lg navbar-dark bg-dark">
  <div class="container">
  </div>
</nav>
```

`navbar-expand-lg` 是关键：大屏（大屏，英文 `large`，`>=992px`）展开成横排，小屏收起来。为什么选 `lg`？vivo 这种菜单多，`md` 就展开会挤，`lg` 最稳。记住这一条就够了。

> **定义**：断点展开（展开，英文 `expand`）指导航在大于某宽度时横向铺开、小于时折叠，`navbar-expand-lg` 即 `992px` 为界。

### 1.2 品牌标识与汉堡按钮配置

logo 区用 `navbar-brand`，里面放图片：

```html
<a class="navbar-brand" href="#">
  <img src="./images/logo.png" alt="vivo" height="30">
</a>
```

`alt` 别空着，图片挂了还有文字兜底，搜索也认它。

汉堡按钮是小屏的命门，三个属性一个都不能少：

```html
<button class="navbar-toggler" type="button"
  data-bs-toggle="collapse" data-bs-target="#vivoNav">
  <span class="navbar-toggler-icon"></span>
</button>
```

| 属性 | 作用 |
| :--- | :--- |
| `data-bs-toggle="collapse"` | 声明这是折叠开关 |
| `data-bs-target="#vivoNav"` | 指向要折叠的盒子 id |
| `navbar-toggler-icon` | 三条杠的图标本体 |

> **易错点**：按钮点了没反应，先查两处：JS 引了没；`data-bs-target` 的 id 和折叠盒子的 `id` 是否一字不差，`#` 不能丢，大小写不能错。

这里容易搞错的还有 `data-bs-toggle` 和老版本的 `data-toggle`。Bootstrap 5 全带 `bs` 前缀，照旧版本抄必跪，一定要如此，绝不能那般。

### 1.3 折叠菜单列表搭建

折叠区长这样，`id` 和按钮的指向对上：

```html
<div class="collapse navbar-collapse" id="vivoNav">
  <ul class="navbar-nav mx-auto">
    <li class="nav-item"><a class="nav-link active" href="#">产品</a></li>
    <li class="nav-item"><a class="nav-link" href="#">商城</a></li>
    <li class="nav-item"><a class="nav-link" href="#">服务</a></li>
    <li class="nav-item"><a class="nav-link" href="#">社区</a></li>
    <li class="nav-item"><a class="nav-link" href="#">企业</a></li>
  </ul>
</div>
```

`collapse` 管收起，`navbar-collapse` 管展开后的排版，两个类都得有。`navbar-nav` 让列表横排，`nav-link` 给链接加上合适的行高和颜色。

`mx-auto` 是什么意思？左右外边距自动，菜单就被顶到中间了。你们想要菜单居中、logo 在左，记住这一招就行。

```text
大屏 >=992px                  小屏 <992px
[logo][产品 商城 服务 社区]    [logo]        [汉堡]
                                点汉堡后：
                                [产品]
                                [商城]
                                [服务]
                                [社区]
```

读图说明：同一套结构，大屏横排，小屏纵向堆叠，靠 `expand-lg` 一行类名切换。

写到这里先停手验证：浏览器拖到手机宽度，看汉堡出没出现；点一下，看菜单展不展开；再拉回电脑宽度，看菜单横不横。骨头正了我们再贴肉，下一节专门调颜色、间距和右侧图标。

> 💡 **承前启后**：完成对「187-vivo响应式官网-navbar导航栏制作（上)」的理解后，下一章我们将深入探讨「188-vivo响应式官网-navbar导航栏制作（下）」，进一步完善知识图谱体系。

---

## 第 7 章：188-vivo响应式官网-navbar导航栏制作（下）
> 对应分集：P189 | 原始标题：《188-vivo响应式官网-navbar导航栏制作（下）》

上一节骨头搭完了，折叠能点、不断行，可你们也看到了：默认的黑底白字跟 vivo 那种干净的深色差口气，字挤、没 hover 效果，右边还没有搜索和人头图标。记住我这句话：结构用库的，颜值必须自己的。这一节我们专门贴肉，把导航调成官网的样子。

### 1.1 导航背景与高度覆盖

先把那身默认皮肤换了。别去改 Bootstrap 的源码，在 `index.css` 里覆盖：

```css
.navbar {
  background-color: #000;
  height: 60px;
  padding: 0;
}
.navbar .nav-link {
  color: #fff;
  font-size: 14px;
  padding: 0 18px;
  line-height: 60px;
}
```

为什么 `line-height` 写 60px？导航高 60，字行高也 60，文字自然垂直居中。你们以前用 `padding` 硬撑、算半天，用行高一行就搞定。

```html
<nav class="navbar navbar-expand-lg navbar-dark">
  <div class="container">
  </div>
</nav>
```

看到没，我把 `bg-dark` 摘了。`navbar-dark` 只管字是浅色系，背景交还给我们自己的 `#000`。分工弄清，覆盖才不打架。

> **提示**：覆盖顺序永远是库在前、自己在后。`index.css` 压不住？先看引入顺序，再看选择器权重，别一上来就 `!important`。

### 1.2 菜单悬停与激活状态处理

vivo 的菜单有个细节：鼠标放上去，字变亮，底下冒一条小横线。做法是边框预留位：

```css
.navbar .nav-link {
  border-bottom: 2px solid transparent;
}
.navbar .nav-link:hover {
  color: #fff;
  border-bottom-color: #fff;
}
.navbar .nav-link.active {
  color: #fff;
  border-bottom-color: #fff;
}
```

为什么平时是 `transparent`？先占住 2px 的位置，hover 时只换颜色不换尺寸，整条导航不会抖。你们直接 `hover` 加边框，菜单会跳一下，这里容易搞错。

当前栏目高亮靠 `active` 类，哪个页面就给哪个链接加：

```html
<li class="nav-item"><a class="nav-link active" href="#">产品</a></li>
<li class="nav-item"><a class="nav-link" href="#">商城</a></li>
```

> **结论**：悬停（悬停，英文 `hover`）管手感，激活（激活，英文 `active`）管位置，一个管一时，一个管此刻，两个样式保持一致，导航才有整体感。

### 1.3 右侧图标组与整体联调

右边那组搜索、人头图标，用字体图标做，上一节学的 `bi` 派上用场了：

```html
<div class="nav-icons d-flex align-items-center">
  <a href="#"><i class="bi bi-search"></i></a>
  <a href="#"><i class="bi bi-person"></i></a>
</div>
```

```css
.nav-icons a {
  color: #fff;
  font-size: 18px;
  margin-left: 22px;
}
.nav-icons a:hover {
  opacity: 0.7;
}
```

图标就是文字，`font-size` 控大小，`color` 控颜色，`margin-left` 拉开距离。悬停我没换色，做了个半透明，干净。

小屏下图标组要不要进折叠区？我建议你们放外面，折叠区只装菜单。手机上 logo、图标、汉堡三者一排，最像官网：

```text
小屏顶栏 (三段分布)
[logo]          [搜索][人头][汉堡]
----------------------------------
点汉堡展开后
  产品
  商城
  服务
  社区
```

读图说明：图标常驻顶栏不折叠，菜单纵向展开，互不抢位。

最后联调三件事，你们跟着我一项项过：第一，拖到 `992px` 前后，汉堡出现和消失的时机对不对；第二，点汉堡三下，开合顺不顺；第三，把菜单加到 8 个，看大屏挤不挤，挤就把 `padding` 从 18px 收到 14px。都顺了，导航才算交工。下一节我们做轮播图，导航的高度会决定轮播顶到哪，别乱改 60 这个数。

> 💡 **承前启后**：完成对「188-vivo响应式官网-navbar导航栏制作（下）」的理解后，下一章我们将深入探讨「189-vivo响应式官网-轮播图模块制作以及图片遮罩」，进一步完善知识图谱体系。

---

## 第 8 章：189-vivo响应式官网-轮播图模块制作以及图片遮罩
> 对应分集：P190 | 原始标题：《189-vivo响应式官网-轮播图模块制作以及图片遮罩》

导航做好了，往下就是官网最抢眼的那块：全屏大轮播。X100 的海报一张张过，底下小圆点跟着走。你们别觉得这是 JS 高手才写得出来的，Bootstrap 把轮播（轮播，英文 `carousel`）做成了填空题，结构填对，图就能转。这一节我们填结构，再把官网那层高级感的遮罩加上。

### 1.1 轮播图三件套结构搭建

轮播一共三件套：图片区、指示点、左右箭头，全包在一个 `carousel` 里：

```text
div.carousel
  +- div.indicators      小圆点
  +- div.carousel-inner  图片轨道
  |    +- div.item.active  图1
  |    +- div.item         图2
  |    +- div.item         图3
  +- button.prev           左箭头
  +- button.next           右箭头
```

读图说明：轨道里一次只亮一张（`active`），圆点和箭头都指向轨道，靠 `id` 关联。

```html
<div id="vivoBanner" class="carousel slide" data-bs-ride="carousel">
  <div class="carousel-indicators">
    <button data-bs-target="#vivoBanner" data-bs-slide-to="0" class="active"></button>
    <button data-bs-target="#vivoBanner" data-bs-slide-to="1"></button>
    <button data-bs-target="#vivoBanner" data-bs-slide-to="2"></button>
  </div>
  <div class="carousel-inner">
    <div class="carousel-item active">
      <img src="./images/banner1.jpg" class="d-block w-100" alt="X100">
    </div>
    <div class="carousel-item">
      <img src="./images/banner2.jpg" class="d-block w-100" alt="S19">
    </div>
    <div class="carousel-item">
      <img src="./images/banner3.jpg" class="d-block w-100" alt="Y78">
    </div>
  </div>
</div>
```

`data-bs-ride="carousel"` 是自动播放的开关，不加它，轮播就定在第一张。`active` 只能有一张 consort，你们复制粘贴最容易多贴一个，图就叠在一起了。

图片那两个类 `d-block w-100` 是什么意思？`w-100` 让图宽顶满轨道，`d-block` 去掉图片底下那几像素的白缝。做轮播，这两个一个不能少。

> **易错点**：圆点点了没反应，箭头点了不动，九成是 `data-bs-target` 的 `#vivoBanner` 和外层 `id` 对不上。复制我的代码，先把 id 改成自己的，再全篇统一改。

### 1.2 切换按钮与播放参数配置

箭头两个按钮长这样：

```html
<button class="carousel-control-prev" data-bs-target="#vivoBanner" data-bs-slide="prev">
  <span class="carousel-control-prev-icon"></span>
</button>
<button class="carousel-control-next" data-bs-target="#vivoBanner" data-bs-slide="next">
  <span class="carousel-control-next-icon"></span>
</button>
```

`data-bs-slide="prev/next"` 管方向，图标是 Bootstrap 自带的半透明白箭头，放在深色海报上正好。

自动播放的速度和悬停行为可以调：

```html
<div id="vivoBanner" class="carousel slide"
  data-bs-ride="carousel" data-bs-interval="4000" data-bs-pause="hover">
```

| 参数 | 作用 | 取值建议 |
| :--- | :--- | :--- |
| `data-bs-interval` | 每张停留毫秒数 | `4000`~`5000`，太快看不清 |
| `data-bs-pause` | 鼠标悬停是否暂停 | `hover`，方便用户细看 |
| `data-bs-wrap` | 到末尾是否循环 | 默认循环，别关 |

你们记住，`interval` 写在外层是全局，写在某个 `carousel-item` 上是单张特例，海报想多停一会儿就单独加。

### 1.3 图片自适应与暗色遮罩叠加

官网的图为什么怎么拉都不变形？两个手段：图自适应，遮罩压光。

先让图片在不同屏幕下都好看：

```css
.carousel-item img {
  height: 520px;
  object-fit: cover;
}
```

> **定义**：对象贴合（贴合，英文 `object-fit`）指图片在固定框内的填充方式，`cover` 即等比铺满并裁掉多余，框多大图多大，不留白不变形。

`cover` 会裁图，人物别放太靠边，安全区留中间。手机上 520 太高了，加一行媒体查询压下来：

```css
@media (max-width: 768px) {
  .carousel-item img {
    height: 320px;
  }
}
```

再说遮罩（遮罩，英文 `mask/overlay`）。白色字压在浅色海报上看不清，官网就在图上蒙了一层由下而上的黑渐变：

```html
<div class="carousel-item active">
  <img src="./images/banner1.jpg" class="d-block w-100" alt="X100">
  <div class="banner-mask"></div>
  <div class="carousel-caption">
    <h3>X100 系列</h3>
    <p>蔡司影像，旗舰之光</p>
  </div>
</div>
```

```css
.carousel-item {
  position: relative;
}
.banner-mask {
  position: absolute;
  left: 0; bottom: 0;
  width: 100%; height: 45%;
  background: linear-gradient(transparent, rgba(0, 0, 0, 0.65));
}
.carousel-caption {
  bottom: 60px;
}
```

渐变从透明到六成五的黑，字坐在黑的上头，怎么都清楚。为什么只压 45%？全蒙就闷了，上面透原图的光，下面托字的底，这个比例最透气。

```text
轮播项层叠 (自下而上)
+-- img 底层图片 ---------+
+-- .banner-mask 渐变蒙层-+
+-- .carousel-caption 文字+
```

读图说明：三层同属一个 `item`，图片打底，遮罩居中过渡，文字浮顶，缺了定位（定位，英文 `position`）就叠不上。

> **结论**：轮播转不转看 `id` 和 `data`，图美不美看 `cover` 和遮罩。你们回去把三张图换成自己的，手机和电脑各拉一遍，不变形、字清楚，这一节才算过。下一节我们进热门产品模块，轮播的 520 高度别再动了。

> 💡 **承前启后**：完成对「189-vivo响应式官网-轮播图模块制作以及图片遮罩」的理解后，下一章我们将深入探讨「190-vivo响应式官网-热门产品模块搭建」，进一步完善知识图谱体系。

---

## 第 9 章：190-vivo响应式官网-热门产品模块搭建
> 对应分集：P191 | 原始标题：《190-vivo响应式官网-热门产品模块搭建》

轮播图转起来了，我往下翻，空了一大片，心里就不踏实。官网这个位置一定是最能卖货的那一排：四张产品图整整齐齐，标题价格清清楚楚。你们想想，用户刚被轮播勾住眼，往下一滑就看到能下单的东西，这才叫承接。我问你们一句：这一排怎么做才省事？答案就四个字，栅格先行。这一讲我们不填卡片的肉，先把热门产品这个盘子搭结实。

### 1.1 产品区容器与背景划分

我先把这一块在页面里的位置定死。轮播是全屏的，产品区不是，它要收进版心里，两边留白，背景跟轮播切开。你们记住我做页面的习惯：先划段，再定宽，最后才写里面。

```text
页面纵向切分 (自上而下)
+----------------------------+
|  navbar  导航             |
+----------------------------+
|  banner  轮播 全屏         |
+----------------------------+
|  .hot    热门产品 (本讲)   |
|   .container 1200px 居中   |
+----------------------------+
|  .tech   未来科技 (后讲)   |
+----------------------------+
```

读图说明：热门产品区是收窄居中的一段，上下用内边距跟轮播和后段隔开。

```html
<section class="hot">
  <div class="container">
    <!-- 标题行 + 四列产品放这里 -->
  </div>
</section>
```

```css
.hot {
  background-color: #f7f8fa;
  padding: 60px 0;
}
.hot .container {
  width: 1200px;
  margin: 0 auto;
}
```

为啥背景给个浅灰 `#f7f8fa`？白色卡片坐在浅灰上，一眼就能浮起来。你们要是全页都用纯白，卡片跟底就糊在一起了。记住这一条就够了：区块之间要么靠间距分，要么靠底色分，两个至少占一个。

`padding: 60px 0` 又是啥讲究？上下各 `60px`，这个数字不是拍脑袋的。大屏上太薄显得挤，太厚显得空，`60px` 刚好够标题喘气。后面小屏我们再压，现在先按这个抄。

> **易错点**：`section` 是语义化标签，有主题的区块就用它，不要满屏 `div`。搜索引擎看到 `section` 就知道这是一块独立内容。

### 1.2 标题行版式处理

盘子有了，先把盘子上的名牌摆好。热门产品这四个字，加上右边一个查看更多的入口，这是标配。你们别小看这一行，它决定用户一眼知不知道这块卖的是啥。

```html
<div class="hot-head">
  <h2>热门产品</h2>
  <a href="#">查看更多 &gt;</a>
</div>
```

```css
.hot-head {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 30px;
}
.hot-head h2 {
  font-size: 28px;
  font-weight: 700;
  margin: 0;
}
.hot-head a {
  font-size: 14px;
  color: #666;
  text-decoration: none;
}
.hot-head a:hover {
  color: #415fff;
}
```

`space-between` 在这里就是左标题右链接，一行撑开，你们以后写这种头，闭眼用这个。为啥标题是 `28px` 加粗？不是 `24px`、不是 `32px`？`28px` 在 `1200px` 版心里最压得住，又不至于抢产品的风头，我试过好几版，就这个看着最舒服。

链接为啥默认灰、悬停变蓝？你们想想，默认状态它是个配角，灰一点不抢戏，鼠标上去了，它得告诉用户能点，变蓝就是给反馈。记住：凡是能点的东西，悬停一定要有变化，不然用户不敢点。

> **提示**：`h2` 自带上下外边距，不清掉的话标题行会比设计稿高出一截，先 `margin: 0` 再调自己的 `margin-bottom`。

### 1.3 四列栅格骨架搭建

标题行下面，就是四个空位。这一讲我们先占位，不填图，让四个格子先站稳。你们猜我用啥？直接上 Bootstrap 的栅格（栅格，英文 `grid`），`row` 包 `col`，一行四等分。

```html
<div class="row hot-list">
  <div class="col-6 col-lg-3">
    <div class="hot-card">卡片1占位</div>
  </div>
  <div class="col-6 col-lg-3">
    <div class="hot-card">卡片2占位</div>
  </div>
  <div class="col-6 col-lg-3">
    <div class="hot-card">卡片3占位</div>
  </div>
  <div class="col-6 col-lg-3">
    <div class="hot-card">卡片4占位</div>
  </div>
</div>
```

```text
栅格占位 (大屏一行四列)
+------+------+------+------+
| col  | col  | col  | col  |
| lg-3 | lg-3 | lg-3 | lg-3 |
+------+------+------+------+
<- - gutter 间隙自动留白 - ->
```

读图说明：大屏每列占 `3/12`，小屏每列占 `6/12` 自动折成两行，间隙由栅格的槽宽承担。

为啥是 `col-6 col-lg-3`？你们算一下：`12 / 3 = 4` 列，`12 / 6 = 2` 列。大屏四列一排，小屏两列两行，手机上一个手指划得动。这就是响应式（响应式，英文 `responsive`）的第一步：列数跟着屏宽换。

| 类名 | 生效宽度 | 一行几列 |
| :--- | :--- | :--- |
| `col-6` | 默认全宽度 | 2 列 |
| `col-lg-3` | `>=992px` | 4 列 |

你们记住这个搭配，以后产品列表闭眼这么写就行。这里容易搞错的是槽宽，有人觉得格子贴太紧，自己加 `margin`，千万别。栅格的间隙靠 `gutter`（槽宽，英文 `gutter`），默认就有，你们要是再加 `margin`，一行就塞不下四个了，直接掉下去一个。

```css
.hot-card {
  background-color: #fff;
  border-radius: 12px;
  height: 320px;
}
```

占位卡我先给白底圆角 `12px`，高度随便撑 `320px`，就是让你们看清四个格子站没站齐。你们回去把浏览器从宽往窄拖：宽的时候一行四个，窄到 `992px` 以下变成两行两个，不断就是对的。卡片里的图和字我们下一讲再填，这一讲盘子稳了就行。

> **结论**：热门产品区就三板斧：浅灰底拉开层次，标题行左右撑开，四列栅格先占位。你们把这三样跑通，下一讲我们往格子里装图装价，心里才不慌。

> 💡 **承前启后**：完成对「190-vivo响应式官网-热门产品模块搭建」的理解后，下一章我们将深入探讨「191-vivo响应式官网-热门产品卡片模块（上）」，进一步完善知识图谱体系。

---

## 第 10 章：191-vivo响应式官网-热门产品卡片模块（上）
> 对应分集：P192 | 原始标题：《191-vivo响应式官网-热门产品卡片模块（上）》

盘子上一讲搭好了，四个白格子站得整整齐齐。这一讲我们往第一个格子里装东西。我先问你们：用户点一张产品卡，先看啥？图，再看名，最后看价。就三眼，多一眼都不给。所以卡片也只做三层：图、名、价。你们把这三层写利索，后面三个卡复制改字就行。

### 1.1 卡片骨架与图片处理

卡片我直接用 Bootstrap 的卡片（卡片，英文 `card`），`card` 包图片加身体，这是人家定好的结构，你们别自己发明。

```text
卡片纵向分层 (自上而下)
+-------------------+
| card-img-top 图片 |
+-------------------+
| card-body 身体    |
|  title 名称       |
|  price 价格       |
+-------------------+
```

读图说明：图在上，文字在下，身体部分只装两行，多的不装。

```html
<div class="col-6 col-lg-3">
  <div class="card hot-card h-100">
    <img src="./images/hot1.jpg" class="card-img-top" alt="X100">
    <div class="card-body">
      <h3 class="card-title">X100 系列</h3>
      <p class="card-price">￥3999起</p>
    </div>
  </div>
</div>
```

`h-100` 是啥意思？就是高度顶满父列。你们想想，四张卡的字有长有短，不加这个，卡有高有低，底边对不齐，强迫症当场就犯了。加上 `h-100`，四个一样高，底边一条线。

图片这里有个关键活：图有大有小，框是固定的，咋办？我给你们结论：

```css
.hot-card .card-img-top {
  height: 200px;
  object-fit: cover;
}
```

> **定义**：对象贴合（贴合，英文 `object-fit`）指图片在固定框内的填充方式，`cover` 即等比铺满并裁掉多余，框多大图多大，不留白不变形。

为啥是 `200px`？卡片总高 `320px` 左右，图占 `200px`，剩下 `120px` 给字，图大字小，符合用户先看图的习惯。你们要是图给 `150px`，卡就显得头轻脚重，不好看。还有，人物和 logo 别放图边上，`cover` 会裁边，安全区留中间。

> **易错点**：`img` 是行内替换元素，底下有几像素白缝，看着跟字贴不上。两种治法：加 `display: block`，或者父级字号清零。卡片里直接让它块级最省事。

### 1.2 标题价格文本层写法

图铺好了，写下面的两行字。标题一行，价格一行，就这么简单，但字号颜色差一档都不能错。

```css
.hot-card .card-body {
  padding: 16px;
  text-align: center;
}
.hot-card .card-title {
  font-size: 16px;
  font-weight: 600;
  color: #333;
  margin: 0 0 8px 0;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
.hot-card .card-price {
  font-size: 14px;
  color: #415fff;
  margin: 0;
}
```

价格用品牌蓝

标题那三行溢出处理是啥意思？`nowrap` 不换行，`hidden` 藏多余，`ellipsis` 出省略号，三句一套，标题再长也只占一行。你们记住，卡片标题只能一行，多一行，四个卡就对不齐了。

| 元素 | 字号 | 颜色 | 说明 |
| :--- | :--- | :--- | :--- |
| `card-title` | `16px` | `#333` | 产品名，一行省略 |
| `card-price` | `14px` | `#415fff` | 价格，跳色引导 |

为啥标题 `16px`、价格 `14px`？标题是主，价格是辅，主大辅小。但价格颜色跳，虽小也抢眼，这叫以色补形。你们细品，大厂的卡全是这个路数。

这一讲我们先把第一个卡写死写透，你们在浏览器里看：图铺满不变形，标题居中一行，价格蓝色一行，卡底对齐。有这三样，这一讲就算过了。剩下三个卡咋批量铺，悬停咋浮起来，我们下一讲接着干。

> **结论**：卡片上半身就记住三件事：结构用 `card` 套 `body`，图片用 `cover` 定高铺满，标题价格一大一小一深一亮。你们回去把第一张卡换成自己的图，手机电脑各看一遍，不变形、不串行，就可以往下走了。

> 💡 **承前启后**：完成对「191-vivo响应式官网-热门产品卡片模块（上）」的理解后，下一章我们将深入探讨「192-vivo响应式官网-热门产品卡片模块（下）」，进一步完善知识图谱体系。

---

## 第 11 章：192-vivo响应式官网-热门产品卡片模块（下）
> 对应分集：P193 | 原始标题：《192-vivo响应式官网-热门产品卡片模块（下）》

上一个卡写顺了，这一讲我们把它复制成四个，再让它活起来。你们想想，四个白卡干摆着，用户为啥要点？得给点反馈：鼠标一上去，卡浮起来，阴影出来，购买的口子露出来。还有，电脑上四列，手机上不能硬挤，得自动变成两列。这一讲就干这两件事：悬停动效和列数切换。

### 1.1 四卡复制与悬停阴影

先把一个卡复制出四个，图和字换掉。你们复制的时候记住：结构一个字别改，只改 `src`、`alt`、标题和价格。结构一改，四个卡高低立马不一样。

```html
<div class="row g-3 hot-list">
  <div class="col-6 col-lg-3">
    <div class="card hot-card h-100">
      <img src="./images/hot1.jpg" class="card-img-top" alt="X100">
      <div class="card-body">
        <h3 class="card-title">X100 系列</h3>
        <p class="card-price">￥3999起</p>
        <a href="#" class="buy">立即购买</a>
      </div>
    </div>
  </div>
  <!-- hot2 / hot3 / hot4 结构相同，只换图文 -->
</div>
```

`g-3` 是啥？栅格槽宽（槽宽，英文 `gutter`）的档位，`3` 就是横竖都留 `16px` 左右的缝。你们之前问格子太贴咋办，答案就是这个，不要自己加 `margin`。

悬停怎么写？我只加两样：上浮和阴影。

```css
.hot-card {
  border: none;
  border-radius: 12px;
  overflow: hidden;
  transition: transform 0.3s, box-shadow 0.3s;
}
.hot-card:hover {
  transform: translateY(-6px);
  box-shadow: 0 12px 30px rgba(0, 0, 0, 0.12);
}
.hot-card .buy {
  display: inline-block;
  margin-top: 10px;
  font-size: 13px;
  color: #415fff;
  text-decoration: none;
  border: 1px solid #415fff;
  border-radius: 20px;
  padding: 4px 16px;
  opacity: 0;
  transition: opacity 0.3s;
}
.hot-card:hover .buy {
  opacity: 1;
}
```

为啥上浮只给 `-6px`？不是 `-10px`、`-20px`？`6px` 是人眼觉得浮起来了、又不觉得在跳的那个点，大了就飘，小了没感觉。阴影 `12px` 模糊加一成二的黑，也是同理：托得住卡，又不脏页面。

购买按钮默认 `opacity: 0` 藏起来，悬停才出来，这是啥心思？平时卡面干净，用户一有兴趣，口子递到手边。你们记住：卖货的按钮，平时藏，悬停露，转化才高。

> **提示**：`transition` 一定要写在 `.hot-card` 本体上，不是写在 `:hover` 上。写在悬停上，鼠标移入有动画，移开就硬跳回去了，两边都要顺才写本体。

```text
悬停状态切换
常态: 平面卡片 无阴影 按钮隐藏
  |  hover
  v
悬停: 上浮6px 柔阴影 按钮显现
```

读图说明：位移和阴影同步 `0.3s` 过渡，按钮单独做透明度淡入。

### 1.2 多端列数与等高处理

四卡齐了，把屏幕从宽往窄拖，检查列数。`col-6 col-lg-3` 的意思上一讲讲过，这里再落一次：

| 宽度 | 类名生效 | 一行几列 |
| :--- | :--- | :--- |
| `<992px` | `col-6` | 2 列两行 |
| `>=992px` | `col-lg-3` | 1 行 4 列 |

```css
@media (max-width: 768px) {
  .hot {
    padding: 40px 0;
  }
  .hot-head h2 {
    font-size: 22px;
  }
  .hot-card .card-img-top {
    height: 160px;
  }
}
```

小屏三处要压：区块 padding 从 `60px` 压到 `40px`，标题从 `28px` 压到 `22px`，图高从 `200px` 压到 `160px`。为啥全要压？手机一屏才 `375px` 宽，东西一样大，用户滑三屏还看不完产品，早就跑了。小屏的核心就一个字：省。

等高再强调一次：`h-100` 别丢，标题一行省略别丢，这两个丢一个，手机两列就一边高一边低，像豁了牙。你们检查就看底边，四张卡底边一条线，购买按钮齐刷刷，才算过。

> **易错点**：`overflow: hidden` 配 `border-radius: 12px` 是一对，丢了前者，图片的方角会把卡片的圆角顶穿，四个角冒尖，最容易在这里翻车。

到这里，热门产品四个卡就算闭环了。你们回去数三遍：常态齐，悬停浮，小屏两列不断。都对了，我们下一讲进未来科技，那块是深色大图，写法跟这块完全反过来，正好换换脑子。

> **结论**：下半身记住两招：悬停只做上浮加阴影加按钮显现，多端只做三处压尺寸加两列切换。招不在多，灵就行。

> 💡 **承前启后**：完成对「192-vivo响应式官网-热门产品卡片模块（下）」的理解后，下一章我们将深入探讨「193-vivo响应式官网-未来科技模块制作」，进一步完善知识图谱体系。

---

## 第 12 章：193-vivo响应式官网-未来科技模块制作
> 对应分集：P194 | 原始标题：《193-vivo响应式官网-未来科技模块制作》

产品卡那块是白底小卡，密密麻麻讲卖货。这一块我要反过来：深色、大图、一句话。你们想想，官网光卖货不行，还得讲梦想，不然跟电商详情页有啥区别？未来科技这块就是干这个的：黑底一张大图，中间一句话加一个按钮，用户一看就觉得这牌子有未来。我带你们把这块的气质做出来。

### 1.1 深色通栏容器搭建

这一块跟热门产品最大的不同是：不收版心，全屏通栏，黑底。你们记住：讲科技感，底色先黑一半。

```text
科技区纵向结构 (自上而下)
+----------------------------+
| .tech  黑底通栏 全屏       |
|  +----------------------+  |
|  | .tech-media 大图     |  |
|  |  + overlay 暗蒙层   +  |
|  |  + .tech-text 文字  +  |
|  +----------------------+  |
+----------------------------+
```

读图说明：图片打底，暗层过渡，文字浮顶，三层同属一个媒体盒。

```html
<section class="tech">
  <div class="tech-media">
    <img src="./images/tech.jpg" alt="未来影像">
    <div class="tech-mask"></div>
    <div class="tech-text">
      <h2>未来影像，超越所见</h2>
      <p>自研芯片 × 蔡司光学，夜景如白昼</p>
      <a href="#" class="more">探索科技</a>
    </div>
  </div>
</section>
```

```css
.tech {
  background-color: #000;
  padding: 0 0 60px 0;
}
.tech-media {
  position: relative;
  height: 480px;
  overflow: hidden;
}
.tech-media img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}
```

高度为啥定 `480px`？大屏上这个高度刚好一屏露个头，用户不用滑就知道下面有大片。太矮了没气势，太高了手机上三屏滑不完。`overflow: hidden` 是保险：图再大也不准撑破盒子。

> **提示**：通栏区段就不要再套 `1200px` 的 `container` 了，套了两边留白，黑底的气势直接断掉，要通就通到底。

### 1.2 文字叠压与按钮处理

白字压在照片上，最怕看不清。治法跟轮播那讲一样：蒙一层暗的，再把字坐上去。但这一块蒙得比轮播狠，因为字在正中间，不在底边。

```css
.tech-mask {
  position: absolute;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  background: linear-gradient(rgba(0, 0, 0, 0.15), rgba(0, 0, 0, 0.65));
}
.tech-text {
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
  text-align: center;
  color: #fff;
}
.tech-text h2 {
  font-size: 36px;
  font-weight: 700;
  margin: 0 0 12px 0;
}
.tech-text p {
  font-size: 15px;
  color: rgba(255, 255, 255, 0.8);
  margin: 0 0 24px 0;
}
.tech-text .more {
  display: inline-block;
  font-size: 14px;
  color: #fff;
  text-decoration: none;
  border: 1px solid rgba(255, 255, 255, 0.6);
  border-radius: 24px;
  padding: 10px 32px;
  transition: background-color 0.3s, border-color 0.3s;
}
.tech-text .more:hover {
  background-color: #415fff;
  border-color: #415fff;
}
```

居中这行 `translate(-50%, -50%)` 你们一定要背下来：先 `left/top` 走到中点，再往回挪自身一半，横竖都中，比 flex 还稳，专门治这种浮层字。

字号为啥标题 `36px`、副文案 `15px`？差出一倍多，层级一眼分明。副文案颜色为啥是八成白 `rgba(255,255,255,0.8)`？纯白太扎眼，灰了又看不清，八成白是 vừa 好：退半步，把 C 位让给标题。

按钮平时是描边透明，悬停变品牌蓝，这是啥逻辑？平时它不能抢图的风头，描边就行；鼠标上去了，用户想点了，它变实心蓝，告诉用户跟我走。记住：深色区按钮，默认虚，悬停实。

| 元素 | 常态 | 悬停 |
| :--- | :--- | :--- |
| `.more` 按钮 | 透明底白描边 | 蓝底蓝边 |
| 切换 | `transition 0.3s` | 同上反向 |

小屏也要压，不然 `36px` 在手机上直接怼到边：

```css
@media (max-width: 768px) {
  .tech-media {
    height: 320px;
  }
  .tech-text h2 {
    font-size: 24px;
  }
  .tech-text p {
    font-size: 13px;
  }
}
```

你们回去把这一块跟热门产品连起来看：上面白底小卡讲现在能买啥，下面黑底大图讲以后能成啥，一白一黑，一密一疏，节奏就出来了。下一讲我们进页脚，黑底还能接上，正好顺下去。

> **结论**：科技区就三句话：通栏黑底定调，暗层加居中让字站住，按钮平时虚悬停实。你们把高度和字号在手机上压一遍，不溢出、不顶边，这一讲就过了。

> 💡 **承前启后**：完成对「193-vivo响应式官网-未来科技模块制作」的理解后，下一章我们将深入探讨「194-vivo响应式官网-底部模块制作（上）」，进一步完善知识图谱体系。

---

## 第 13 章：194-vivo响应式官网-底部模块制作（上）
> 对应分集：P195 | 原始标题：《194-vivo响应式官网-底部模块制作（上）》

主体两块都做完了，热门产品讲现在，未来科技讲远方，页面到这里气是足的。但你们往底下一拉，空了，直接露底，这就像话说一半人走了。页脚就是收尾的人，这一讲我们先收第一层：顶部服务条。别小看这一条：正品保障、快速配送、售后无忧，四个词一摆，用户信任直接拉满。

### 1.1 页脚外层与服务条结构

我做页脚的习惯是先定外层，再分三段：服务条、链接区、版权区。这一讲只写第一段，但外层和分段一次定死，后面两讲直接往里填。

```text
页脚纵向分段 (自上而下)
+----------------------------+
| footer 黑底                |
| +------------------------+ |
| | .service 服务条 本讲   | |
| |------------------------| |
| | .footer-link 链接 下讲 | |
| |------------------------| |
| | .footer-bottom 版权 后讲| |
| +------------------------+ |
+----------------------------+
```

读图说明：三段同属一个 `footer`，本讲只写最上面一段，下面两段先占位。

```html
<footer class="footer">
  <div class="container service">
    <a href="#"><i class="icon">✓</i>官方正品</a>
    <a href="#"><i class="icon">✈</i>快速配送</a>
    <a href="#"><i class="icon">♥</i>售后无忧</a>
    <a href="#"><i class="icon">◉</i>门店体验</a>
  </div>
  <div class="container footer-link">
    <!-- 下一讲填五列链接 -->
  </div>
  <div class="footer-bottom">
    <!-- 再下一讲填版权 -->
  </div>
</footer>
```

```css
.footer {
  background-color: #222;
  color: #999;
}
.footer .container {
  width: 1200px;
  margin: 0 auto;
}
```

为啥页脚底色是 `#222`？你们看，科技区是纯黑 `#000`，页脚 `#222` 比它亮一档，页面到底了反而透口气。要是页脚也用纯黑，两块糊在一起，分不清哪到哪。记住：相邻深色块，差一档灰，才有层次。

外层为啥用 `footer` 标签？语义化（语义化，英文 `semantics`）四个字你们听到起茧了吧？这里就是 payoff：搜索引擎看到 `footer` 就知道这是页脚，跟正文分开算。以后凡是页脚，闭眼 `footer`，不要 `div`。

> **提示**：图标我这里先用字符占位，你们后面换成字体图标（字体图标，英文 `iconfont`）也行，换图片也行，结构不用动，大小我下面定死。

### 1.2 四项横排与分隔处理

服务条四个入口，一行摆开，中间用竖线隔开。写法跟标题行一样：弹性盒（弹性盒，英文 `flexbox`）横排，两端对齐，垂直居中。

```css
.service {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 28px 0;
  border-bottom: 1px solid #333;
}
.service a {
  flex: 1;
  text-align: center;
  font-size: 14px;
  color: #ccc;
  text-decoration: none;
}
.service a + a {
  border-left: 1px solid #333;
}
.service a:hover {
  color: #fff;
}
.service .icon {
  display: inline-block;
  width: 28px;
  height: 28px;
  line-height: 28px;
  text-align: center;
  border: 1px solid #555;
  border-radius: 50%;
  margin-right: 8px;
  font-style: normal;
}
```

`a + a` 是啥意思？相邻兄弟选择器（兄弟选择器，英文 `sibling selector`），从第二个 `a` 开始，左边加一条线。好处是以后加第五个入口，不用改样式，自动有分隔线。你们要是手敲 `|` 符号，间距永远对不齐，记住这一招，专治分隔线。

图标为啥是 `28px` 圆？字 `14px`，图标大一倍，眼睛先看到圈，再看到字。圆圈描边 `#555`，比底 `#222` 亮，比字 `#ccc` 暗，隐约托一下就行，太实了会抢戏。

| 属性 | 取值 | 作用 |
| :--- | :--- | :--- |
| `display` | `flex` | 四项横排 |
| `justify-content` | `space-between` | 均分撑开 |
| `border-bottom` | `1px solid #333` | 跟链接区分段 |

底下那条 `border-bottom` 是分段线，颜色 `#333` 比底亮一点点，隐约看得见就行。你们别用纯黑，纯黑在 `#222` 上等于没有；也别用太亮，太亮了把页面切成两截。

小屏咋办？四个一排在手机上肯定挤，我让它换行，一行两个：

```css
@media (max-width: 768px) {
  .service {
    flex-wrap: wrap;
  }
  .service a {
    flex: none;
    width: 50%;
    margin-bottom: 16px;
  }
}
```

`flex-wrap: wrap` 就是允许掉行，配上 `width: 50%`，刚好两行两个。你们回去拖一下：宽屏一行四个带竖线，窄屏两行四个，图标不挤，字不换行，就算对了。

> **易错点**：`i` 标签默认斜体，做图标一定要 `font-style: normal` 掰直，不然圈里的符号全是歪的，第一眼就露怯。

服务条写完，页脚的门面就立住了。你们记住这一讲的顺序：外层 `footer` 定色，服务条横排加分隔，底下一条线跟后段切开。链接区和版权区我们后面两讲再填，架子已经给你们留好了。

> **结论**：页脚上半身就三样：黑底比上段亮一档，四项横排加自动分隔，小屏换行一行两个。你们把这三样跑通，后面填链接才不慌。

> 💡 **承前启后**：完成对「194-vivo响应式官网-底部模块制作（上）」的理解后，下一章我们将深入探讨「195-vivo响应式官网-底部模块制作（中）」，进一步完善知识图谱体系。

---

## 第 14 章：195-vivo响应式官网-底部模块制作（中）
> 对应分集：P196 | 原始标题：《195-vivo响应式官网-底部模块制作（中）》

刚把主体几个大模块啃下来，我长舒一口气，问你们一句：一个官网，什么地方最能看出做得细不细？不是轮播图，是页脚。用户拉到最底下，密密麻麻全是链接，排得整整齐齐，字号、颜色、间距都恰到好处，这才叫大厂的样子。这一讲我们就专心干这件事，把 vivo 官网底部那一片链接区先搭出来。

我先带你们看一下最终效果：最外层是深色底，里面分好几列，每一列最上面是一个小标题，下面跟着四五个链接。你们注意看，标题是白色的，链接是灰色的，鼠标放上去链接会变白。这就是我们这一讲要拿下的第一部分。

### 1. 底部模块整体结构划分

我做页面有个习惯，先把大架子定死，再往里填肉。底部这一块，我把它拆成上下两层：上面是链接区，下面是版权区。这一讲我们只做上面这一层。

```text
+--------------------------------------+
|  footer                              |
|  +-------------------------------+   |
|  | .footer-link (本讲)           |   |
|  |  col1  col2  col3  col4  col5 |   |
|  +-------------------------------+   |
|  | .footer-bottom (下一讲)       |   |
|  +-------------------------------+   |
+--------------------------------------+
```

读图说明：`footer` 是最外层容器，上下两块各管一摊，这一讲只碰 `.footer-link`。

为什么要这么拆？你们想想，链接区是多列排布，用弹性盒（英文：Flexible Box，缩写：Flex）最顺手；版权区是单行居中，写法完全不一样。混在一起写，后面改媒体查询（英文：Media Query）的时候你们会哭的。

```html
<footer class="footer">
  <div class="container footer-link">
    <!-- 五列链接放这里 -->
  </div>
  <div class="footer-bottom">
    <!-- 下一讲再写 -->
  </div>
</footer>
```

> **提示**：`footer` 用语义化标签，不要再拿 `div` 糊弄了，搜索引擎看到 `footer` 就知道这是页脚。

### 2. 单列链接的盒子写法

你们别一上来就想五列，先把一列写利索。我带你们写第一列，其他四列就是复制粘贴改字。

每一列的结构，我用的是 `dl` 包 `dt` 加 `dd`。有人问我，为啥不用 `ul`？你们记住我这句话：标题加列表这种结构，用 `dl（描述列表，Description List）` 最对味，`dt` 放标题，`dd` 放每一项，天生就是一对多的关系。

```html
<dl class="footer-col">
  <dt>选购及了解</dt>
  <dd><a href="#">智能手机</a></dd>
  <dd><a href="#">智能硬件</a></dd>
  <dd><a href="#">配件</a></dd>
  <dd><a href="#">以旧换新</a></dd>
</dl>
```

```css
.footer {
  background-color: #222;
}
.footer-col dt {
  color: #fff;
  font-size: 14px;
  margin-bottom: 16px;
}
.footer-col dd {
  margin: 0 0 10px 0;
}
.footer-col dd a {
  color: #999;
  font-size: 12px;
  text-decoration: none;
}
.footer-col dd a:hover {
  color: #fff;
}
```

你们看这几个数字：标题 `14px`，链接 `12px`，标题底下留 `16px`，每一项之间留 `10px`。为啥标题要比链接大两号？不是随便定的，就是要让眼睛一眼分出层级。记住这一条就够了：页脚的字一定要小，小才显得精致，但对比一定要清楚。

> **易错点**：`dd` 自带左边距，不清掉的话五列对不齐，一定要 `margin: 0` 先清零。

### 3. 五列横排的弹性盒布局

一列写好了，复制出五列，然后让它们横着排开。这里我直接上 `display: flex`，你们千万别再用浮动（英文：Float）了，那都是老黄历了。

```css
.footer-link {
  display: flex;
  justify-content: space-between;
  padding: 40px 0;
}
.footer-col {
  flex: 1;
}
```

`justify-content: space-between` 是啥意思？就是五列之间自动留空，两端顶到容器边上。你们用这个，就不用自己算每一列多宽了，省心得很。

```text
+------+------+------+------+------+
| col1 | col2 | col3 | col4 | col5 |
+------+------+------+------+------+
       <- space-between 自动均分 ->
```

读图说明：五列等分容器宽度，列间空隙由弹性盒自动分配。

这里容易搞错的地方是 `padding`。我给 `.footer-link` 上下各留了 `40px`，为啥是 `40px`？不是 `70px`、`80px`？页脚不需要太厚，太厚了显得笨。`40px` 刚好透气，你们照着抄就行。

| 属性 | 取值 | 作用 |
| :--- | :--- | :--- |
| `display` | `flex` | 五列横排 |
| `justify-content` | `space-between` | 列间均分空隙 |
| `padding` | `40px 0` | 上下透气 |

### 4. 在线客服与电话入口处理

你们看原站，最右边那一列不太一样，没有那么多链接，而是一个客服入口，有电话号码，还有个按钮。我问你们，为啥要特殊处理？因为页脚不光是摆链接的，还要留转化的口子，用户有问题得找得到人。

这一列我单独给它一个类名，文字右对齐也好，或者保持左对齐但加上图标也好，关键是把电话号码字号放大加粗，让它跳出来。

```html
<dl class="footer-col footer-service">
  <dt>在线服务</dt>
  <dd class="tel">400-678-9688</dd>
  <dd><a class="btn" href="#">在线客服</a></dd>
</dl>
```

```css
.footer-service .tel {
  color: #fff;
  font-size: 20px;
  font-weight: 700;
}
.footer-service .btn {
  display: inline-block;
  border: 1px solid #999;
  border-radius: 20px;
  padding: 6px 20px;
  color: #fff;
  font-size: 12px;
}
```

> **结论**：页脚右侧服务列的核心就是三样东西：大字号电话、白字标题、描边按钮，别的都不用加。

> 💡 **承前启后**：完成对「195-vivo响应式官网-底部模块制作（中）」的理解后，下一章我们将深入探讨「196-vivo响应式官网-底部模块制作（下）」，进一步完善知识图谱体系。

---

## 第 15 章：196-vivo响应式官网-底部模块制作（下）
> 对应分集：P197 | 原始标题：《196-vivo响应式官网-底部模块制作（下）》

上一讲我们把上面那片链接区搭完了，五列排得整整齐齐，我看着挺舒服。这一讲我们把底下那一条收尾：版权、备案、隐私协议，还有那一排小图标。你们别小看这一条，做不好，整个页脚就像没穿鞋，看着漏风。

我先说个判断：版权区就一个要求，稳。字要小，颜色要灰，居中或者左右分开，反正不能抢戏。你们记住，页脚越往下越要安静。

### 1. 版权区容器与背景处理

链接区和版权区之间，我加了一条分割线。你们猜我用的是啥？不是 `hr`，就是一个 `border-top`。为啥？省一个标签，少一件事。

```text
+--------------------------------------+
|  .footer-link  (上一讲已完成)        |
|--------------------------------------|
|  .footer-bottom (本讲)               |
|   左:版权  中:协议  右:图标          |
+--------------------------------------+
```

读图说明：一条 `1px` 分割线上下分区，本讲只写下半部分。

```html
<div class="footer-bottom">
  <div class="container bottom-inner">
    <p class="copyright">Copyright © 2025 vivo 版权所有</p>
    <p class="links">
      <a href="#">隐私政策</a>
      <a href="#">法律声明</a>
      <a href="#">营业执照</a>
    </p>
    <p class="social">
      <a href="#">微博</a>
      <a href="#">微信</a>
    </p>
  </div>
</div>
```

```css
.footer-bottom {
  border-top: 1px solid #333;
}
.bottom-inner {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 20px 0;
  font-size: 12px;
  color: #666;
}
.bottom-inner a {
  color: #666;
  text-decoration: none;
  margin: 0 8px;
}
.bottom-inner a:hover {
  color: #fff;
}
```

你们注意这三个数字：字号 `12px`，行内留白 `20px`，链接间距 `8px`。为啥全是小数字？因为版权区就是要缩着，字大了就喧宾夺主了。

> **易错点**：分割线颜色不要用纯黑，用 `#333`，比背景 `#222` 亮一点点，隐约看得见就行，太实了会把页面切成两截。

### 2. 备案号与协议链接排布

国内官网，备案号是一定要放的，还有营业执照、隐私政策，一个都不能少。我把它们放在中间，用竖线隔开。竖线咋来的？两种写法，一种是直接敲 `|`，一种是用 `border-right`。我教你们用最省事的：直接在 `a` 之间加左右内边距，中间那个分隔符用边框做。

```css
.links a + a {
  border-left: 1px solid #444;
  padding-left: 16px;
}
```

这行代码啥意思？第一个 `a` 不管，从第二个 `a` 开始，左边加一条线，左边再垫 `16px`。这样写的好处是，以后加一个链接，不用改样式，自动就有分隔线了。

| 写法 | 缺点 | 是否推荐 |
| :--- | :--- | :--- |
| 手敲 `\|` 符号 | 间距不好控，对不齐 | 不推荐 |
| `border-left` 做法 | 自动适配新增链接 | 推荐 |

> **提示**：`a + a` 这种相邻兄弟选择器（英文：Adjacent Sibling Selector），做分隔线特别好使，你们一定要记住这一招。

### 3. 社交图标与收尾检查

最右边那几个微博微信的小图标，我这一讲用文字先占位，后面你们换成字体图标（英文：Iconfont）也行，换成图片也行。关键是大小要统一，我给的是 `24px` 见方，圆角一半，直接就是小圆点。

```css
.social a {
  display: inline-block;
  width: 24px;
  height: 24px;
  line-height: 24px;
  text-align: center;
  border: 1px solid #444;
  border-radius: 50%;
}
```

写到这里，整个 `footer` 就算闭环了。我带你们最后检查三件事：第一，链接区五列加底下一行，在 `1200px` 容器里有没有溢出；第二，所有 `a` 的 `hover` 是不是都变白了；第三，把页面缩到 `768px` 看一眼，挤成一团是正常的，下一讲我们就用媒体查询收拾它。

```text
检查清单
[1] 容器不溢出
[2] hover 变色一致
[3] 字号层级 20 / 14 / 12
```

读图说明：三项都打钩，这一讲才算过关，缺一项都不要往下走。

> 💡 **承前启后**：完成对「196-vivo响应式官网-底部模块制作（下）」的理解后，下一章我们将深入探讨「197-vivo响应式官网-媒体查询和自定义滚动条样式」，进一步完善知识图谱体系。

---

## 第 16 章：197-vivo响应式官网-媒体查询和自定义滚动条样式
> 对应分集：P198 | 原始标题：《197-vivo响应式官网-媒体查询和自定义滚动条样式》

页脚两层都堆完了，我把浏览器从大屏往小了拖，你们猜发生啥了？五列链接挤成一坨，版权区三段叠在一起，字都压变形了。这就是不做适配的下场。这一讲我们就干两件事：让页面在手机上能看，再把那条灰不溜秋的滚动条打扮一下。

我先问你们一个问题：啥叫响应式（英文：Responsive）？不是做三套页面，而是一套代码，靠媒体查询在不同宽度下换写法。记住这一条就够了。

### 1. 断点选择与版心处理

断点（英文：Breakpoint）定多少？我直接给你们结论：`1200px`、`992px`、`768px` 三档。为啥是这三个？这是 Bootstrap 里用了多少年的老数字，大屏、中屏、小屏刚好卡住，你们不要自己发明 `813px` 这种怪数字。

```css
/* 大屏默认：版心 1200px */
.container {
  width: 1200px;
  margin: 0 auto;
}

/* 中屏：版心收窄 */
@media (max-width: 992px) {
  .container {
    width: 750px;
  }
}

/* 小屏：满屏留边 */
@media (max-width: 768px) {
  .container {
    width: 100%;
    padding: 0 15px;
    box-sizing: border-box;
  }
}
```

你们看，小屏下我把 `width` 直接干到 `100%`，再加 `15px` 的内边距。为啥是 `15px`？拇指按得住，眼睛看得下，留白不大不小。还有，`box-sizing: border-box` 一定要加，不然 `100%` 加上 `padding` 直接溢出，页面就左右晃了。

> **易错点**：媒体查询一定要从大往小写，后写的覆盖先写的，顺序反了，小屏样式会被大屏盖掉。

### 2. 底部模块的多端切换

页脚那五列，在手机上绝对不能硬撑。我给你们的方案是：`768px` 以下，五列直接换行，一行排两列，服务列单独占一行。

```css
@media (max-width: 768px) {
  .footer-link {
    flex-wrap: wrap;
  }
  .footer-col {
    flex: none;
    width: 50%;
    margin-bottom: 24px;
  }
  .footer-service {
    width: 100%;
  }
  .bottom-inner {
    flex-direction: column;
    text-align: center;
    gap: 12px;
  }
}
```

`flex-wrap: wrap` 是啥意思？就是允许换行，原来挤在一行的五列，现在放不下就掉到下一行。再配上 `width: 50%`，刚好一行两个，整整齐齐。

```text
大屏 ( >768px )
[col1][col2][col3][col4][col5]

小屏 ( <=768px )
[col1  ][col2  ]
[col3  ][col4  ]
[  service 100% ]
[  copyright 居中 ]
```

读图说明：大屏一行摆完，小屏自动折成两列加一个通栏，版权区改成竖排居中。

这里容易搞错的是 `.bottom-inner`。大屏是 `space-between` 左右撑开，小屏一定要改成 `column` 居中，不然版权、协议、图标三段挤在一行，手机上根本点不到。

| 断点 | `.footer-col` | `.bottom-inner` |
| :--- | :--- | :--- |
| `>768px` | `flex: 1` 横排 | 横排两端对齐 |
| `<=768px` | `width: 50%` 换行 | 竖排居中加 `12px` 间隙 |

### 3. 自定义滚动条样式

好，适配做完了，我们来干个开心的活：把滚动条变好看。默认那个灰条多丑啊，我们给它换个深色细条，跟我们这个黑色页脚正好配一脸。

滚动条这套伪元素（英文：Pseudo-element），就三个件：`::-webkit-scrollbar` 管整体宽度，`::-webkit-scrollbar-track` 管轨道，`::-webkit-scrollbar-thumb` 管滑块。你们背下来就行。

```css
::-webkit-scrollbar {
  width: 8px;
}
::-webkit-scrollbar-track {
  background: #f1f1f1;
}
::-webkit-scrollbar-thumb {
  background: #888;
  border-radius: 4px;
}
::-webkit-scrollbar-thumb:hover {
  background: #555;
}
```

为啥滑块是 `8px`？太粗了笨，太细了点不到，`8px` 是个手感和美观的平衡点。圆角给 `4px`，正好是宽度的一半，滑块两头就是圆的了。

> **提示**：这套写法前面都带 `webkit`，在 Chrome 和 Edge 里好使，火狐认 `scrollbar-width`，你们做项目时知道有兼容差异就行，考试和练习里写这一套就够了。

写完你们自己拖一拖看，从 `1200px` 一路拖到 `375px`，五列变两列，版权竖排，滚动条又细又黑，顺滑得很。到这一步，这个 vivo 官网才算真正做完了。

> 💡 **承前启后**：完成对「197-vivo响应式官网-媒体查询和自定义滚动条样式」的理解后，下一章我们将深入探讨「198-前端网页托管-发布自己的网站」，进一步完善知识图谱体系。

---

## 第 17 章：198-前端网页托管-发布自己的网站
> 对应分集：P199 | 原始标题：《198-前端网页托管-发布自己的网站》

vivo 这个站我们从头写到尾，媒体查询也加上了，现在它还躺在你们自己电脑里，只有你们自己能看。我问你们，学前端最爽的一刻是啥？就是把链接甩到朋友圈，别人点开说一句：这是你做的？这一讲我们就干这件事，把网站发到网上去。

我先把话说透：发布（英文：Deploy）不神秘，就是把你的 `html`、`css`、`images` 搬到一台一直开机的电脑上，那台电脑叫服务器（英文：Server）。别人访问网址，服务器就把文件吐回去，浏览器一渲染，页面就出来了。

### 1. 发布前的工程整理

发布之前，先把家里收拾干净。你们现在文件夹里是不是一堆 `新建文件夹`、`最终版2`、`真的最终版`？全删掉，只留要上线的东西。

```text
vivo-site/            <- 就发这个文件夹
  index.html          <- 首页，必须叫这个名
  css/
    style.css
  images/
    logo.png
  js/
    index.js
```

读图说明：只留四样东西，首页必须叫 `index.html`，缺了它服务器不知道先吐哪个。

为啥必须叫 `index.html`？这是约定，服务器收到 `www.xxx.com` 这种没写文件名的请求，默认就找 `index.html`。你叫 `首页.html`，服务器直接两手一摊，说找不到。

还有路径，发布前一定要检查一遍：凡是 `C:\Users\xxx\Desktop` 这种绝对路径，一个都不能留，全改成相对路径（英文：Relative Path）。`images/logo.png` 这种才对，换到服务器上照样找得到。

> **易错点**：图片在本地能出来、发上去裂了，十有八九是路径里带了中文文件夹名或者用了绝对路径，发布前全局搜一遍 `C:` 和 `D:`。

### 2. 静态托管平台操作流程

我们零基础发第一个站，不租服务器，直接用静态托管（英文：Static Hosting），免费，拖进去就行。我带你们走一遍通用流程，哪家平台都这四步。

```text
[本地文件夹] --> [拖拽上传] --> [等待构建] --> [拿到网址]
     1              2              3              4
```

读图说明：四步走完，那个 `https` 开头的网址就是你们的站了。

具体操作我一步步说：第一步，注册登录托管平台，进控制台点新建站点；第二步，把刚才整理好的 `vivo-site` 整个拖进上传框；第三步，等它转圈，一般十几秒，提示构建成功；第四步，复制它给你的二级域名，比如 `xxx.github.io` 或者平台送的测试域名，直接粘到浏览器看效果。

| 步骤 | 动作 | 检查点 |
| :--- | :--- | :--- |
| 1. 新建 | 选静态站点 | 别选成动态应用 |
| 2. 上传 | 拖整个文件夹 | 首页在根目录 |
| 3. 等待 | 看构建日志 | 显示成功再走 |
| 4. 访问 | 复制网址打开 | 手机电脑都试一遍 |

> **提示**：第一次打开是空白的多半是缓存，按 `Ctrl + F5` 强刷一下，十有八九就出来了。

### 3. 换内容与二次更新方法

网址拿到了，你们肯定不满足用我的 vivo 素材，换成你们自己的名字、照片、作品，这才叫你们的站。更新也简单，改完本地文件，重新上传覆盖，平台会重新构建，等一分钟再刷新就行。

我给你们三个建议：第一，先把标题里的 vivo 换掉，换成你们的名字缩写；第二，把页脚电话换成你们的邮箱，这才是简历里能用的东西；第三，把这个网址记下来，后面学了 JavaScript 加了轮播，再传一版，你们的站是越养越值钱的。

```html
<title>我的第一个响应式网站</title>
```

你们看，就改一个 `title`，浏览器标签页上显示的就是你们自己的字了。小不小？小。但就是从这一小步开始，你们才算真正进了前端的门。去发吧，把链接甩出去，让别人点开看看。

> 💡 **承前启后**：完成对「198-前端网页托管-发布自己的网站」的理解后，下一章我们将深入探讨「199-结尾篇-致前端者mv」，进一步完善知识图谱体系。

---

## 第 18 章：199-结尾篇-致前端者mv
> 对应分集：P200 | 原始标题：《199-结尾篇-致前端者mv》

这一集没有新代码，没有新标签，点开来就是一段音乐和一些画面。我坐在剪辑好的成片前看了两遍，心里挺平静的。从第一个 `html` 标签，到弹性盒，到这个黑底的 vivo 站，我们一起走了整整一百九十九讲。这一篇，我就想跟你们聊聊天，不讲技术，讲讲你们接下来该咋走。

我先跟你们说句实话：你们现在会的东西，找工作还不够，但学东西的方法，已经够了。你们想想，我们是怎么过来的？每一个属性，我都让你们先看效果，再改数字，再猜规律。以后你们碰到没见过的框架、没见过的 AI 工具，也这么干就行。前端这行，变的就是这么快，谁也不是靠背下来的，都是靠试出来的。

### 1. 这趟旅程我们带走了什么

我帮你们数一下行李。第一件，是 `HTML（超文本标记语言，HyperText Markup Language）` 的语义感，你们现在写得出 `header`、`footer`、`dl`，知道啥时候该用哪个，这比会背一百个标签重要得多。第二件，是 `CSS（层叠样式表，Cascading Style Sheets）` 的盒子感和布局感，盒模型（英文：Box Model）、弹性盒、媒体查询，这三样捏在手里，什么样的页面你们都敢拆。第三件，是发布上线的完整感，你们的站已经在网上了，别人点得开，这就是证据。

```text
+----------------------------------+
|  我的行李箱                      |
|  [语义] [盒子] [弹性盒] [响应式] |
|  [一个能上线的作品网址]          |
+----------------------------------+
```

读图说明：前面四格是本事，最后一格是证明，少了哪格心里都不踏实。

你们要是觉得 articles 文件夹里那一堆精读笔记太散，我建议你们就守着那个 vivo 站，逢人就讲一遍：这里我为啥用 `flex`，那里断点为啥定 `768px`，滚动条那 `8px` 咋来的。讲得顺了，面试就不用慌了。

> **提示**：作品不用多，一个能从头讲到尾的响应式站，顶十个拼凑的半成品。

### 2. 给继续赶路的你们几句叮嘱

第一句，别停。前端是手艺，断一个月就生了。后面还有 JavaScript、框架、AI 提效，那才是真正的深水区，你们现在刚学会走，别急着夸自己跑得快。

第二句，别只看。这一套课你们可以二刷，但二刷不是再看一遍，是关掉声音自己敲，敲裂了再回来看。我带过的学生里，凡是能把这个站不看源码默写出来的，后面学啥都快。

第三句，别一个人闷着。把你们发布的那个网址发出去，让朋友点，让同行挑刺。被挑刺是好事，页脚对不齐、手机上溢出，这些刺早挑早好。

音乐到这里就淡出去了，我的话也说完了。山高路远，我们顶峰相见。你们去吧，带着这个站，往前走，别回头。

---

## 模块 08 全景总结与技术沉淀

本全书系统整合了 Bootstrap5框架与vivo官网综合实战及上线发布 模块的 18 个核心专题（P183 ~ P200）。
建议读者在学完本章后，对照 `notes/` 目录下的思维导图树状笔记进行复盘与知识自测，巩固底层机理与工程实践能力。
