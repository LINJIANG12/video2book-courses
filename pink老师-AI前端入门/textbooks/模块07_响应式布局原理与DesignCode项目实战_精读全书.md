# 模块 07：响应式布局原理与DesignCode项目实战 合辑教材

> **所属课程**：2025版pink老师最新AI+前端入门教程，零基础必看的html5、css3、grid、flex布局、响应式、移动端，bootstrap5框架，AI工具  
> **模块跨度**：P164 ~ P182（全模块共 19 讲系统重构）  
> **内容定位**：模块化系统学习教材，融合核心机制、架构全景、代码解析与思考自测。  
> **关联说明**：本册由各集精读讲义整编而成；单集讲义属生成侧中间产物，不随本仓库发布。

---

## 模块导读与全景目录

- **第 1 章**：163-第七篇案例展示以及响应式布局-媒体查询
- **第 2 章**：164-响应式布局-仿京东响应式效果
- **第 3 章**：165-designcode响应式项目-准备工作以及topbar模块制作
- **第 4 章**：166-designcode响应式项目-头部模块logo制作
- **第 5 章**：167-designcode响应式项目-头部模块导航栏制作
- **第 6 章**：168-designcode响应式项目-导航栏响应式效果制作
- **第 7 章**：169-designcode响应式项目-汉堡菜单响应式布局
- **第 8 章**：170-designcode响应式项目-课程模块制作
- **第 9 章**：171-designcode响应式项目-图标和搜索模块制作
- **第 10 章**：172-designcode响应式项目-课程模块响应式效果布局
- **第 11 章**：173-designcode响应式项目-课程卡片模块响应式布局
- **第 12 章**：174-designcode响应式项目-课程卡片模块小盒子布局
- **第 13 章**：175-designcode响应式项目-课程卡片模块鼠标经过效果
- **第 14 章**：176-designcode响应式项目-故事卡片模块搭建
- **第 15 章**：177-designcode响应式项目-故事卡片模块响应式布局
- **第 16 章**：178-designcode响应式项目-clip path裁剪波浪图形
- **第 17 章**：179-designcode响应式项目-底部模块制作以及响应式布局
- **第 18 章**：180-designcode响应式项目-小圆飘动动画效果
- **第 19 章**：181-designcode响应式项目-修复两个bug

---

## 第 1 章：163-第七篇案例展示以及响应式布局-媒体查询
> 对应分集：P164 | 原始标题：《163-第七篇案例展示以及响应式布局-媒体查询》

移动端那篇翻过去了，今天推开第七篇的大门。你们先别急着写代码，我先带你们把这一篇要做的案例整体过一遍，再讲响应式真正的看家本领——媒体查询。为什么是这个顺序？因为你们得先亲眼看到“同一套代码在手机和电脑上长得不一样”，才有动力学那几行判断条件。

---

### 1. 第七篇案例全貌与学习目标

这一篇我们只啃一件事：让同一个页面在不同宽度的屏幕上自动换一副面孔。案例是一个仿电商首页，手机上看是单列加汉堡菜单，电脑上看是多列加完整导航，中间的过渡全部自动发生，不需要准备两套页面。

```text
同一套代码的三种面孔
宽度 < 768px        768px ~ 992px        宽度 > 992px
+----------+       +--------------+     +------------------+
| 汉堡菜单  |       |  横向导航     |     | 完整导航 + 侧栏   |
| 单列商品  |       |  双列商品     |     |  四列商品         |
| 堆叠 footer|      |  双列 footer  |     |  横向 footer      |
+----------+       +--------------+     +------------------+
```

读图说明：屏幕越宽，列数越多、导航越完整，但 HTML 结构始终是同一套。

> **定义**：响应式布局（响应式布局，Responsive Layout）指一套 HTML 通过不同的层叠样式表（层叠样式表，CSS）规则，自适应多种屏幕宽度的布局方式。

### 2. 媒体查询的核心语法

#### 2.1 媒体查询的基本结构

媒体查询（媒体查询，Media Query）就是写在 CSS 里的一句“如果”。如果屏幕宽度满足某个条件，就执行大括号里那批样式，不满足就跳过。关键字记住四个：`screen` 指屏幕设备，`max-width` 是“宽度小于等于”，`min-width` 是“宽度大于等于”，`and` 用来拼接多个条件。

```css
@media screen and (max-width: 768px) {
  .goods {
    flex-direction: column;
  }
}
```

这段的意思直白得很：如果屏幕宽度小于等于 `768px`，商品区就从横排改成竖排。你们写的时候先写默认的电脑端样式，再用 `max-width` 一档一档往小屏覆盖，这是最不容易乱的顺序。

#### 2.2 常用断点与设备对照

断点不是随便拍脑袋定的，是跟着主流设备宽度走的。这一讲我给你们定的三档是：

| 断点条件 | 对应设备 | 本篇要干的事 |
| :--- | :--- | :--- |
| 小于 `768px` | 手机竖屏 | 单列，隐藏侧栏，导航折叠 |
| `768px` 到 `992px` | 平板竖屏和小笔记本 | 双列，导航精简 |
| 大于 `992px` | 桌面显示器 | 完整多列，全部展开 |

> **提示**：断点值宁可照抄主流框架的 `768px`、`992px`、`1200px`，也不要自创 `813px` 这种怪数，将来接手你代码的人会感谢你。

#### 2.3 外链样式的按需加载写法

除了写在同一个文件里，媒体查询还可以直接挂在 `link` 标签上，让浏览器按屏幕宽度决定下载哪份样式。手机访问就只下手机那份，省流量。

```html
<link rel="stylesheet" href="./css/pc.css" media="screen and (min-width: 993px)">
<link rel="stylesheet" href="./css/mobile.css" media="screen and (max-width: 768px)">
```

> **易错点**：`media` 属性里的条件和 `@media` 大括号里的条件写法完全一致，但它只控制“用不用这份文件”，文件内部的选择器该怎么写还怎么写，不要把两者混为一谈。

### 3. 响应式与自适应的本质区别

收尾必须把这对双胞胎掰开。你们之前做的移动端页面叫自适应：给手机单独做一版，地址都可能不一样。而响应式是一套代码通吃，靠媒体查询在内部切换。代价是响应式的 CSS 要写三档，调试量更大；好处是以后改结构只改一处，不用来回同步两个版本。

```css
/* 自适应：手机单独一版，条件写死 */
.container {
  width: 100%;
}

/* 响应式：同一选择器，随宽度切换 */
.container {
  width: 1170px;
}
@media screen and (max-width: 992px) {
  .container {
    width: 750px;
  }
}
```

下一讲我们就拿京东首页开刀，把这三档断点实打实写一遍，你们拖着浏览器窗口看列数自己变化，那一刻就全通了。

> 💡 **承前启后**：完成对「163-第七篇案例展示以及响应式布局-媒体查询」的理解后，下一章我们将深入探讨「164-响应式布局-仿京东响应式效果」，进一步完善知识图谱体系。

---

## 第 2 章：164-响应式布局-仿京东响应式效果
> 对应分集：P165 | 原始标题：《164-响应式布局-仿京东响应式效果》

上一讲媒体查询的语法都会背了，可语法会背不等于手会写。今天直接拿京东首页这种真实电商版式开刀：拖着浏览器窗口从宽拉到窄，看着导航折叠、商品从四列变两列，你们亲手把“如果宽度小于多少就换一套样式”写一遍，响应式才算真正上过手。

---

### 1. 仿京东版式的断点规划

动手前先定断点，不然写到一半一定乱。这一讲我们沿用经典三档：大屏 `1170px`、中屏 `970px`、小屏 `750px`，容器宽度跟着档位跳，而不是连续伸缩。为什么跳变而不是平滑变？因为商品卡片是固定比例的，连续缩放会把卡片压变形，按档跳才能保证每档都是整数列。

```text
容器宽度随断点跳变示意
屏幕宽度 ──────────────────────────>
          |         |         |
        <750px   750~992px  >992px
          |         |         |
       container  container container
        : 750px   : 970px   : 1170px
       商品 2 列   商品 3 列  商品 4 列
```

读图说明：屏幕跨过断点时，容器宽度和商品列数一起跳变，档内保持稳定。

```css
.container {
  width: 1170px;
  margin: 0 auto;
}
@media screen and (max-width: 992px) {
  .container {
    width: 970px;
  }
}
@media screen and (max-width: 768px) {
  .container {
    width: 750px;
  }
}
```

> **提示**：媒体查询的书写顺序必须是先大后小。用 `max-width` 往小覆盖时，小屏规则写在后面才能压住大屏规则，顺序写反小屏样式会被吞掉。

### 2. 导航栏的折叠与隐藏处理

#### 2.1 完整导航与汉堡按钮的切换

大屏下的导航是完整横排：Logo、搜索框、分类链接一字排开。宽度一掉到手机档，横排无论如何塞不下，做法是两步：把完整导航 `display: none` 藏起来，同时把事先准备好的汉堡按钮 `display: block` 放出来。两套导航的 HTML 是一直都在的，只是按宽度轮流露面。

```css
.nav-full {
  display: flex;
}
.nav-mini {
  display: none;
}
@media screen and (max-width: 768px) {
  .nav-full {
    display: none;
  }
  .nav-mini {
    display: block;
  }
}
```

#### 2.2 次要元素的按需隐藏

除了导航，页面上还有大量“大屏是加分、小屏是累赘”的元素：顶部促销条、侧边广告栏、冗长的筛选行。处理原则统一：小屏档一律 `display: none`，一个都不心疼。记住，手机用户要的是快和准，不是全。

```css
@media screen and (max-width: 768px) {
  .promo-bar,
  .sidebar-ad {
    display: none;
  }
}
```

> **易错点**：隐藏用 `display: none` 而不用 `visibility: hidden`，后者只是看不见，占位还在，会留下一大块空白，这是新手最容易踩的坑。

### 3. 商品列表的列数自适应

商品区是弹性盒（弹性布局，Flexible Box）排的，列数变化不需要改结构，只改每张卡片的宽度占比。大屏四列，每张 `25%`；中屏三列，每张 `33.33%`；小屏两列，每张 `50%`。配合 `flex-wrap: wrap` 自动换行，列数就变过来了。

```css
.goods {
  display: flex;
  flex-wrap: wrap;
}
.goods .item {
  width: 25%;
}
@media screen and (max-width: 992px) {
  .goods .item {
    width: 33.33%;
  }
}
@media screen and (max-width: 768px) {
  .goods .item {
    width: 50%;
  }
}
```

| 屏幕档位 | 容器宽度 | 商品列数 | 导航形态 |
| :--- | :--- | :--- | :--- |
| 大于 `992px` | `1170px` | 4 列 | 完整横排导航 |
| `768px` 到 `992px` | `970px` | 3 列 | 精简导航 |
| 小于 `768px` | `750px` | 2 列 | 汉堡按钮，次要元素隐藏 |

你们现在就拖着浏览器窗口从宽拉到窄看一遍：`1170px` 处跳一次，`992px` 处跳一次，`768px` 处导航“啪”地换成汉堡按钮，商品从四列压到两列。三次跳变都对上了，这一讲就过关了。响应式的整套手感你们已经拿到，后面接 Bootstrap 框架时，会发现它无非是把这三档断点提前封装好了而已。

> 💡 **承前启后**：完成对「164-响应式布局-仿京东响应式效果」的理解后，下一章我们将深入探讨「165-designcode响应式项目-准备工作以及topbar模块制作」，进一步完善知识图谱体系。

---

## 第 3 章：165-designcode响应式项目-准备工作以及topbar模块制作
> 对应分集：P166 | 原始标题：《165-designcode响应式项目-准备工作以及topbar模块制作》

我一上来先不急着写头部，先把这个 DesignCode 项目的地基打牢。你们跟着我做过小兔鲜、做过优医问诊，应该有感觉了：凡是看起来复杂的页面，前面半小时都在搭架子，架子搭对了，后面每个模块都是填空。

这一讲我就干两件事：把项目文件和基础样式准备好，再把最顶上那条细细的 topbar 通告栏做出来。别小看这条小栏，它把后面整个响应式的思路都定调了。

### 1 项目文件结构与基础样式准备

我新建的文件夹就叫 `designcode`，里面很干净，只有三个东西：`index.html`、`css/index.css`、`images/`。你们千万别一上来就把所有图片往根目录一丢，后面找起来会疯掉。

基础样式我还是老套路，先清零，再定变量。我现在做响应式项目，颜色、字号、版心宽度一定先抽成变量，不然 768px、1024px 几个断点来回改，你会改到怀疑人生。

```css
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

:root {
  --main-color: #5457f5;
  --text-dark: #111827;
  --text-gray: #6b7280;
  --topbar-bg: #111827;
  --topbar-text: #e5e7eb;
  --wrap-width: 1200px;
}

body {
  font-family: "PingFang SC", "Microsoft YaHei", sans-serif;
  color: var(--text-dark);
  background-color: #fff;
}

a {
  text-decoration: none;
  color: inherit;
}

ul {
  list-style: none;
}
```

> **提示**：`box-sizing: border-box` 一定要在第一行就写死。我见过太多人后面盒子宽度对不上，回来查半天发现是 `padding` 把宽度撑爆了。

为什么这里要用 CSS 变量（`var()`）？你们想想，DesignCode 这种站，主色紫色会出现几十次，改一次设计稿你难道去全文替换？定义成 `--main-color`，后面改一行全站生效，这才是工程化的写法。

### 2 版心容器与整体分层

这一套页面的分层，我直接画给你们看：

```text
+-------------------------------+
|  .topbar  通告栏 (全宽,深色)  |
+-------------------------------+
|  .header  头部 (版心居中)      |
|  [logo] [nav........] [btn]   |
+-------------------------------+
|  .banner / .course / ...      |
+-------------------------------+
```

读图说明：只有 `topbar` 是通栏背景，里面再套版心；`header` 本身就是版心居中。记住这个区别，后面写背景色时就不会把宽度搞错。

版心类我这样写：

```css
.wrap {
  width: var(--wrap-width);
  max-width: 100%;
  margin: 0 auto;
  padding: 0 24px;
}
```

> **定义**：版心（容器，Container）指页面内容居中的最大宽度区域，本项目取 `1200px`。

这里 `max-width: 100%` 配 `padding: 0 24px` 就是给响应式埋的伏笔。桌面端它是 1200px 居中，窗口缩小后它自动变成满宽加左右留白，不需要再为它单独写媒体查询。你们记住这一条就够了：凡是版心，先写成流式的，别写死。

| 容器写法 | 桌面效果 | 窄屏效果 |
| :--- | :--- | :--- |
| `width: 1200px` 写死 | 居中正常 | 出现横向滚动条 |
| `width + max-width: 100%` | 居中正常 | 自动收缩，无滚动条 |

我为什么反复强调这个？后面导航栏、课程卡片全部依赖这个 `.wrap`，它要是写死了，响应式一个都做不成。

### 3 顶部通告栏结构搭建

topbar 这一条，你们看上去就是一行小字加一个小箭头，对吧？结构我拆成这样：

```html
<div class="topbar">
  <div class="wrap topbar-inner">
    <p>New: Learn SwiftUI 2025 course is live</p>
    <a href="#">Get access -></a>
  </div>
</div>
```

我故意把文字用 `p`、链接用 `a`，语义是“一段通知加一个行动入口”。千万别拿两个 `div` 一包了事，搜索引擎和读屏软件是靠标签理解你们页面的。

> **易错点**：`topbar` 和 `topbar-inner` 的分工不要混。外层管背景和高度，内层管版心和 flex 对齐。背景写到内层上，缩小窗口时两边会露白。

### 4 顶部通告栏样式与弹性布局

样式核心就三行：深色底、小字号、flex 居中对齐。

```css
.topbar {
  background-color: var(--topbar-bg);
  font-size: 13px;
  color: var(--topbar-text);
}

.topbar-inner {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 12px;
  height: 36px;
}

.topbar a {
  color: #fff;
  font-weight: 600;
}
```

你们问我：为什么是 `36px`？不是 `30px`、`40px`？我量的设计稿就是 36px，字号 13px 上下各留一点呼吸感，太高了喧宾夺主，太矮了手指点不到。做通告栏记住这个手感：高度在 `32px ~ 40px` 之间最舒服。

`gap: 12px` 是我现在最爱用的，文字和链接之间留 12px，比给 `p` 加 `margin-right` 干净多了。`flex` 布局下 `gap` 的兼容性现在完全没问题，放心用。

最后我调了一下小屏：当窗口小于 `768px`，这行英文太长会换行，我直接让它字号降到 `12px`，内层允许省略：

```css
@media (max-width: 768px) {
  .topbar-inner p {
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
  }
}
```

> **结论**：通告栏的响应式第一步不是隐藏，而是先保证不换行、不撑爆。`nowrap + ellipsis` 这三件套，后面所有单行标题都会用到。

这一讲我们就停在这里：架子有了，topbar 通了。下一讲我们直接啃头部最大的硬骨头之一——logo，你们会发现，一个 logo 里藏着 SEO 的大学问。

> 💡 **承前启后**：完成对「165-designcode响应式项目-准备工作以及topbar模块制作」的理解后，下一章我们将深入探讨「166-designcode响应式项目-头部模块logo制作」，进一步完善知识图谱体系。

---

## 第 4 章：166-designcode响应式项目-头部模块logo制作
> 对应分集：P167 | 原始标题：《166-designcode响应式项目-头部模块logo制作》

上一讲我们把 topbar 跑通了，这一讲我带你们做 header 里的第一个零件：logo。你们别觉得 logo 就是放张图，两分钟搞定。我跟你们说，logo 是全站 SEO 权重最高的一张图，写法不对，搜索引擎根本不知道你们是谁。

### 1 Logo 语义结构选择

我先把 header 的骨架摆出来，你们就知道 logo 卡在什么位置：

```text
.header (.wrap + flex)
+--------+ +----------------+ +--------+
| .logo  | | .nav (下一讲)  | | .btn   |
| 180x36 | |  ...           | | 登录   |
+--------+ +----------------+ +--------+
```

读图说明：header 本身是 flex 横向三段，logo 固定在最左，宽度写死，后面导航自适应。这就是为什么 logo 必须先做，它把左边界定死了。

结构我这样写，你们照抄就行：

```html
<header class="header wrap">
  <h1 class="logo">
    <a href="#">DesignCode</a>
  </h1>
</header>
```

为什么外面要套 `h1`？我跟你们反复讲过，一个页面只能有一个 `h1`，它告诉搜索引擎“我是干什么的”。把 logo 放进 `h1`，权重直接拉满。用 `div` 包 logo 的，功能上也能看，但语义上就丢分了。

> **定义**：语义化（Semantics）指用最贴切的标签表达内容含义，`h1` 表示页面主标题，权重大于普通 `div`。

| 写法 | 显示效果 | 语义效果 |
| :--- | :--- | :--- |
| `div.logo > a` | 正常显示 | 无权重，搜索引擎忽略 |
| `h1.logo > a` | 正常显示 | 主标题权重，推荐写法 |

### 2 背景图与文字隐藏处理

DesignCode 的 logo 是一张横向的图形加文字，我不直接用 `img`，而是用背景图。你们猜猜为什么？因为 `h1` 里面已经有文字“DesignCode”了，背景图盖上去，文字还在，只是看不见，搜索引擎还能读到，这叫一举两得。

```css
.logo {
  width: 180px;
  height: 36px;
}

.logo a {
  display: block;
  width: 100%;
  height: 100%;
  background: url("../images/logo.png") no-repeat left center / contain;
  font-size: 0;
}
```

这里 `font-size: 0` 就是隐藏文字的老办法。还有人用 `text-indent: -9999px` 把文字挤出屏幕，效果一样，我更喜欢 `font-size: 0`，一行搞定，不会产生超宽滚动条。

> **易错点**：`a` 是行内元素，不给 `display: block` 的话，`width`、`height` 全都失效，背景图只剩一行字的高度。每次 logo 不显示，先查这一行。

`background-size: contain` 也要记住。logo 图有大有小，`contain` 保证整张图按比例缩进 `180x36` 的盒子里，不裁剪、不变形。你们要是写成 `cover`，图的左右会被裁掉一半。

### 3 Logo 尺寸与对齐调试

尺寸我定的是 `180px` 宽、`36px` 高，这是量过设计稿的。高度 36px 跟后面导航文字行高对齐，header 整体才不抖。

垂直居中我交给父级 `.header` 的 flex：

```css
.header {
  display: flex;
  align-items: center;
  height: 72px;
}
```

你们看，header 高 72px，logo 高 36px，`align-items: center` 一写，上下各剩 18px，天生居中。千万别再给 logo 加 `margin-top` 去硬顶，那种写法窗口一缩就歪。

> **提示**：图片路径 `../images/logo.png` 里这个 `../` 不要漏。`index.css` 在 `css/` 文件夹里，要先退一层再进 `images/`。路径错了图裂开，十个里有八个是这里错的。

我最后拖动窗口验证了一下：从 1200px 一直缩到 360px，logo 始终固定 180px 不变形，右边的导航后面会隐藏，但 logo 纹丝不动。这就对了，品牌标识在任何屏幕下都不能缩水、不能换行，这是底线。下一讲我们接着往右走，把导航那一串链接做出来。

> 💡 **承前启后**：完成对「166-designcode响应式项目-头部模块logo制作」的理解后，下一章我们将深入探讨「167-designcode响应式项目-头部模块导航栏制作」，进一步完善知识图谱体系。

---

## 第 5 章：167-designcode响应式项目-头部模块导航栏制作
> 对应分集：P168 | 原始标题：《167-designcode响应式项目-头部模块导航栏制作》

logo 定住了，这一讲我们把 header 中间最长的一截——导航栏做出来。你们数一数，DesignCode 这一版有 5 个链接：Courses、Tutorials、Livestreams、Pricing、More，每一个的间距、字重、hover 状态都得抠到位。

### 1 导航列表结构搭建

结构没什么花活，就是无序列表。我跟你们说过，凡是并列的一组链接，闭眼用 `ul > li > a`，这是前端的肌肉记忆。

```html
<nav class="nav">
  <ul>
    <li class="active"><a href="#">Courses</a></li>
    <li><a href="#">Tutorials</a></li>
    <li><a href="#">Livestreams</a></li>
    <li><a href="#">Pricing</a></li>
    <li><a href="#">More</a></li>
  </ul>
</nav>
```

为什么外面还要包一层 `nav`？`ul` 只是列表，`nav` 才表示“这是导航区”。读屏软件遇到 `nav` 会直接播报“导航”，用户一键就能跳进来。少了这层，语义就塌了一半。

```text
nav.nav
  └─ ul (flex 横排)
       ├─ li > a  Courses
       ├─ li > a  Tutorials
       ├─ li > a  Livestreams
       ├─ li > a  Pricing
       └─ li > a  More
```

读图说明：flex 加在 `ul` 上，不是 `nav` 上。`nav` 只负责占位和语义，横排是列表自己的事，分工清楚后面做响应式才好隐藏。

> **提示**：第一个 `li` 我提前加了 `active` 类。当前栏目高亮是导航的标配，现在先埋好钩子，样式下一节直接挂上去。

### 2 横向布局与间距控制

布局代码你们看好了，核心就四行：

```css
.nav ul {
  display: flex;
  align-items: center;
  gap: 28px;
}

.nav a {
  font-size: 15px;
  font-weight: 500;
  color: var(--text-dark);
  line-height: 72px;
}
```

`gap: 28px` 这个数字是我量的设计稿，链接之间 28px，不多不少。你们千万别用 `margin-right` 一个个顶，最后一个还要去清零，麻烦死了。`gap` 写在父级，一行管全部。

`line-height: 72px` 是个小窍门：header 高 72px，链接行高也给 72px，文字自然垂直居中，而且整块 72px 高度都是可点击区，手指好点。你们要是只给 `padding`，点击区忽大忽小，体验差一截。

header 三段的分配，我是这样收的口：

```css
.header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  height: 72px;
}

.nav {
  flex: 1;
  margin-left: 40px;
}
```

> **结论**：logo 固定宽，导航 `flex: 1` 吃掉中间所有剩余空间，登录按钮顶到最右。`margin-left: 40px` 是 logo 和导航之间的安全距离，窗口缩小时先挤这个距离，不挤文字。

| 属性 | 取值 | 作用 |
| :--- | :--- | :--- |
| `display` | `flex` | 列表横排 |
| `gap` | `28px` | 链接等距 |
| `line-height` | `72px` | 垂直居中并扩大点击区 |

### 3 链接状态与当前项样式

最后把状态加上，不然导航看起来死气沉沉。两件事：鼠标经过变色，当前项加粗加主色。

```css
.nav a:hover {
  color: var(--main-color);
  transition: color 0.2s;
}

.nav li.active a {
  color: var(--main-color);
  font-weight: 700;
}
```

`transition: color 0.2s` 别省，0.2 秒的颜色渐变会让 hover 丝滑很多，没有它，颜色是生切的，一看就很廉价。

> **易错点**：选择器一定要写成 `.nav li.active a`，权重是类加元素，能稳稳盖过 `.nav a`。直接写 `.active` 的话，权重不够，颜色挂不上去，很多人这里调半天。

我点了一遍 5 个链接，hover 全变紫，Courses 常亮紫色加粗，横向无换行、无错位，这一讲就收工。你们现在把窗口往窄了拖会发现，导航挤到 900px 左右就开始顶按钮了。别慌，这正是我下一讲要解决的：到断点直接把整组导航藏起来，换汉堡菜单上场。

> 💡 **承前启后**：完成对「167-designcode响应式项目-头部模块导航栏制作」的理解后，下一章我们将深入探讨「168-designcode响应式项目-导航栏响应式效果制作」，进一步完善知识图谱体系。

---

## 第 6 章：168-designcode响应式项目-导航栏响应式效果制作
> 对应分集：P169 | 原始标题：《168-designcode响应式项目-导航栏响应式效果制作》

上一讲结尾我留了个问题：窗口缩到 900px，导航和按钮就打架了。这一讲我就带你们把这件事收掉，核心只有一句话：到断点，让桌面导航优雅退场。

你们记住，做响应式不是把页面压扁，而是到某个宽度直接换一套布局。这一讲换的就是导航这一截。

### 1 响应式断点选择

断点我定的是 `1024px` 和 `768px` 两档，你们看我是怎么做取舍的：

```css
/* 桌面：>1024px  完整导航 */
/* 平板：768px ~ 1024px  导航隐藏，汉堡出现 */
/* 手机：<768px  只留 logo + 汉堡 */
```

为什么是 1024？我们导航 5 个链接加登录按钮，一共差不多 700px 宽，加上 logo 180px 和间距，1024 以下必然挤爆。我实际拖了一遍，1000px 左右就开始重叠，所以 1024 就是那个临界点。你们以后定断点别背数字，就拖窗口，看哪里坏就定哪里。

| 视口宽度 | 导航状态 | 汉堡状态 |
| :--- | :--- | :--- |
| `> 1024px` | 显示 | 隐藏 |
| `<= 1024px` | 隐藏 | 显示 |
| `<= 768px` | 隐藏 | 显示并放大点击区 |

> **定义**：断点（Breakpoint）指布局切换的视口宽度阈值，写在媒体查询（Media Query）`@media` 的条件里。

### 2 桌面导航隐藏实现

隐藏的代码简单到你们不敢信，就三行：

```css
@media (max-width: 1024px) {
  .nav {
    display: none;
  }

  .header .btn-login {
    display: none;
  }
}
```

`display: none` 一写，元素直接从渲染树里摘掉，不占位、不响应点击，后面的汉堡按钮顶上来，header 瞬间清爽。我跟你们强调过，这里一定要用 `none`，用 `visibility: hidden` 或者 `opacity: 0` 的，盒子还占着地方，header 中间会空一大块，蠢得很。

顺序也要注意：基础样式写桌面端，媒体查询写在文件最下面。CSS 是后写的覆盖先写的，媒体查询放上面会被桌面样式盖掉，等于白写。你们就记死：桌面先行，断点垫后。

> **易错点**：`max-width: 1024px` 表示“宽度小于等于 1024 时生效”。写成 `min-width` 整个逻辑就反了，桌面端反而没导航。每次响应式失灵，先查 max 和 min 有没有写反。

```text
视口从宽拖到窄
1200px ── [logo][nav 5项][login] ── 正常
1024px ── [logo][nav 5项][login] ── 临界
1023px ── [logo]          [汉堡]  ── nav+login 隐藏
 768px ── [logo]          [汉堡]  ── 保持
```

读图说明：1024 是开关，过去之后桌面导航整体下线，没有中间态，不存在半藏半露。

我拖到 900px 验证了一下：导航和登录按钮同时消失，header 只剩 logo 和右边的汉堡，高度还是 72px，一点没塌。这就对了，隐藏不是把 header 压扁，而是把内容换掉。汉堡按钮长什么样、怎么摆，下一讲我们单独把它做漂亮。

> 💡 **承前启后**：完成对「168-designcode响应式项目-导航栏响应式效果制作」的理解后，下一章我们将深入探讨「169-designcode响应式项目-汉堡菜单响应式布局」，进一步完善知识图谱体系。

---

## 第 7 章：169-designcode响应式项目-汉堡菜单响应式布局
> 对应分集：P170 | 原始标题：《169-designcode响应式项目-汉堡菜单响应式布局》

导航藏起来了，空出来的位置得有人顶上，这个人就是汉堡菜单。你们在手机上天天点那个三横线，今天我们亲手把它做出来。

我跟你们说，汉堡按钮虽小，里面门道不少：三条线要等距，桌面端要彻底隐身，移动端要点得爽，还得给后面的下拉菜单留好钩子。

### 1 汉堡按钮结构与图标绘制

结构我用三根 `span`，干净又好控制：

```html
<button class="burger" aria-label="menu">
  <span></span>
  <span></span>
  <span></span>
</button>
```

为什么用 `button` 而不用 `div`？`button` 天生可聚焦、可回车触发，键盘用户也能操作。包上 `aria-label="menu"`，读屏软件会报“菜单按钮”，无障碍直接加分。用 `div` 模拟按钮的，还要自己补键盘事件，何苦呢。

三条线的画法，我不用图片，纯 CSS：

```css
.burger {
  display: none;
  flex-direction: column;
  justify-content: center;
  gap: 5px;
  width: 40px;
  height: 40px;
  background: none;
  border: 0;
  cursor: pointer;
}

.burger span {
  display: block;
  height: 2px;
  width: 22px;
  margin: 0 auto;
  background-color: var(--text-dark);
  border-radius: 2px;
}
```

默认 `display: none`，桌面端压根不存在它，这是关键。`flex-direction: column` 配 `gap: 5px`，三条线纵向等距，比一条线加上下边框的偏方稳多了，粗细也好调。

```text
.burger 40x40 点击区
+-------------+
|   =======   |  span 22x2
|   =======   |  gap 5px
|   =======   |  span 22x2
+-------------+
```

读图说明：可点击的是 40px 方块，线的视觉宽度只有 22px，线居中，四周留白都是点击区。记住这个关系：视觉可以小，点击区一定要大。

> **提示**：`span` 必须加 `display: block`，行内元素设宽高无效，三条线会挤成一团。这是每次汉堡做不出来的头号原因。

### 2 媒体查询切换显示

切换逻辑跟上一讲是镜像的：1024 以下汉堡现身，桌面端隐身。

```css
@media (max-width: 1024px) {
  .burger {
    display: flex;
  }
}
```

就这么一行，`none` 翻成 `flex`，三条线自动纵向排好。你们看，显示和隐藏一定要成对写：`.nav` 是桌面显示、移动隐藏，`.burger` 是桌面隐藏、移动显示，一藏一现，header 右边永远有人站岗，不会空。

| 元素 | 桌面 `>1024px` | 移动 `<=1024px` |
| :--- | :--- | :--- |
| `.nav` | `flex` 显示 | `none` 隐藏 |
| `.btn-login` | 显示 | `none` 隐藏 |
| `.burger` | `none` 隐藏 | `flex` 显示 |

> **结论**：响应式切换的本质就是两组 `display` 互斥。把这张表背下来，所有“桌面一套、移动一套”的组件都是这个套路。

header 右对齐我用 `margin-left: auto` 兜底。不管中间导航在不在，汉堡永远贴右，跟 logo 一左一右，72px 高度里垂直居中，拖到 360px 也不会乱跑。

### 3 移动端布局验证

我最后从 1200px 一路拖到 360px 验了三遍：桌面端 5 个链接加登录按钮整整齐齐，1024 一过瞬间切成 logo 加汉堡，无闪动、无错位、无横向滚动条。点汉堡虽然这一讲还没挂下拉面板，但 `button` 的按压态已经有了，`cursor: pointer` 一放上去就是小手，不会让用户觉得点不动。

> **易错点**：汉堡的 `width: 40px` 不要省。只画三条 22px 的线、不给父级尺寸的，点击区只剩几像素，手机上根本点不中。苹果的人机规范里可点击至少 `44px`，我们给 40px 是底线。

到这里，头部三件套齐了：logo 定左，导航居中，汉堡守右。下一讲我们离开头部，往下做课程模块，你们会发现，响应式真正的主战场其实在卡片列表。

> 💡 **承前启后**：完成对「169-designcode响应式项目-汉堡菜单响应式布局」的理解后，下一章我们将深入探讨「170-designcode响应式项目-课程模块制作」，进一步完善知识图谱体系。

---

## 第 8 章：170-designcode响应式项目-课程模块制作
> 对应分集：P171 | 原始标题：《170-designcode响应式项目-课程模块制作》

导航和汉堡菜单调通之后，页面往下滚，第一屏正文就是课程模块。我习惯先把这一块的骨架立住：标题在上、卡片在下，中间留足呼吸感。你们跟着我把这一集做完，课程区的大架子就有了，后面几集往里面填图标、搜索框、卡片细节，都是顺手的事。

### 1. 课程模块的三段结构

我把课程模块拆成三段：外层版心、标题区、卡片容器。为什么非要套三层？外层管背景和上下留白，版心管宽度居中，卡片容器管排列，三件事搅在一起，后面做响应式会改到崩溃。

```text
+-------------------------------+
| section.courses               |
|  .w (版心, max-width: 1200px) |
|   .courses-hd (标题区)        |
|   .courses-bd (卡片容器)      |
+-------------------------------+
```

读图说明：`section` 撑满全宽给背景和留白，`.w` 收窄居中，标题和卡片都装在版心里。

```html
<section class="courses">
  <div class="w">
    <div class="courses-hd">
      <h2>Featured Courses</h2>
      <p>Learn the latest design tools and workflows</p>
    </div>
    <div class="courses-bd">
      <!-- 卡片后面几集逐个填 -->
    </div>
  </div>
</section>
```

标题用 `h2` 不是 `h3`，一页只有一个 `h1` 留给站点名，模块标题从 `h2` 往下排，这个层级关系别搞反了。

### 2. 版心与上下留白的取值

```css
.w {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 24px;
}
.courses {
  padding: 96px 0;
  background-color: #f5f7fa;
}
```

三个数字你们记住：`1200px` 是这套设计稿的版心，大屏两边留白、小屏自动收窄；左右各 `24px` 内边距是给手机留的安全距离，卡片再宽也不会贴到屏幕边；上下 `96px` 是模块之间的呼吸，课程区内容密，留白必须大一号，挤在一起会显得很廉价。

> **定义**：版心（Container）指页面内容的最大宽度区域，`max-width` 加 `margin: 0 auto` 让它在大屏居中、在小屏自动撑满。

> **易错点**：`width: 1200px` 和 `max-width: 1200px` 是两回事。前者写死宽度，手机上必出横向滚动条；后者是上限不封下限，小屏自动收缩。响应式项目里一律用 `max-width`。

### 3. 标题区的字号节奏

```css
.courses-hd {
  text-align: center;
  margin-bottom: 56px;
}
.courses-hd h2 {
  font-size: 40px;
  font-weight: 700;
  line-height: 1.2;
  color: #0a0a0a;
}
.courses-hd p {
  margin-top: 16px;
  font-size: 17px;
  line-height: 1.6;
  color: #666;
}
```

我为什么标题 `40px`、副标题 `17px`？差出一倍多，层级一眼就分出来了。你们千万别标题 `24px`、副标题 `20px`，差 `4px` 肉眼根本分不清主次。`margin-bottom: 56px` 是标题区和卡片区的距离，比段内间距大一档，这是排版的老规矩：组内紧、组间松。

```css
.courses-bd {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 32px;
}
```

卡片容器我直接用网格布局（Grid）三等分，间隙（Gap，`gap`）取 `32px`。有人问为什么不用弹性盒子（Flexbox）？三列等宽、行列都有缝，网格一行代码搞定，弹性盒子还要算百分比、处理换行，纯属给自己加戏。这一集先把三列铺出来，下一集开始做响应式，该变两列变两列。

> **提示**：`repeat(3, 1fr)` 等价于写三遍 `1fr`，`fr` 是网格专用的等分单位，会自动吃掉版心剩下的全部宽度，比百分比省心，不用再减间隙。

| 结构层   | 管的事         | 关键属性                          |
| :------- | :------------- | :-------------------------------- |
| `section.courses` | 背景、上下留白 | `padding: 96px 0` |
| `.w`     | 宽度、居中     | `max-width: 1200px; margin: 0 auto` |
| `.courses-hd` | 标题对齐、组间距 | `text-align: center; margin-bottom: 56px` |
| `.courses-bd` | 卡片排列       | `display: grid; gap: 32px` |

> **结论**：这一集不碰任何卡片细节，只干一件事：把课程模块的架子搭正。版心、标题、网格容器三层各司其职，后面填图标、搜索框、卡片，全部往 `.courses-bd` 里加就行，骨架不用再动。

> 💡 **承前启后**：完成对「170-designcode响应式项目-课程模块制作」的理解后，下一章我们将深入探讨「171-designcode响应式项目-图标和搜索模块制作」，进一步完善知识图谱体系。

---

## 第 9 章：171-designcode响应式项目-图标和搜索模块制作
> 对应分集：P172 | 原始标题：《171-designcode响应式项目-图标和搜索模块制作》

课程标题下面空荡荡的不好看，真实的产品页会在这里放两样东西：一排分类图标帮我快速跳转，一条搜索框让我直接搜课。这一集我就把这两样加上去。你们注意，我做图标从来不是为了装饰，每一个图标背后都对应一个跳转，点上去必须有反应。

### 1. 图标分类行的结构与排列

图标行我用无序列表写，语义上它就是一组并列的导航：

```html
<ul class="cate">
  <li>
    <a href="#">
      <i class="iconfont icon-figma"></i>
      <span>Figma</span>
    </a>
  </li>
  <li>
    <a href="#">
      <i class="iconfont icon-swiftui"></i>
      <span>SwiftUI</span>
    </a>
  </li>
  <li>
    <a href="#">
      <i class="iconfont icon-react"></i>
      <span>React</span>
    </a>
  </li>
  <li>
    <a href="#">
      <i class="iconfont icon-blender"></i>
      <span>3D</span>
    </a>
  </li>
</ul>
```

为什么图标不用 `img` 而用字体图标（Iconfont）？四个图标四张图就是四次请求，字体图标一次请求全搞定，还能跟着字号（`font-size`）随意放大缩小不发虚。`i` 管图形、`span` 管文字，各干各的。

```text
+------+------+------+------+
|  O   |  O   |  O   |  O   |
|Figma |SwiftU|React | 3D   |
+------+------+------+------+
```

读图说明：四个小格横向均分，图标在上、文字在下，每格都是一个可点击的 `a`。

```css
.cate {
  display: flex;
  justify-content: center;
  gap: 48px;
  margin-bottom: 40px;
}
.cate a {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 12px;
  font-size: 14px;
  color: #333;
}
.cate .iconfont {
  width: 56px;
  height: 56px;
  line-height: 56px;
  text-align: center;
  font-size: 28px;
  color: #fff;
  background-color: #0a0a0a;
  border-radius: 50%;
}
```

`flex-direction: column` 让图标和文字上下叠，`align-items: center` 让两者左右居中，这两句配在一起就是图标导航的固定写法。圆形底我用 `border-radius: 50%` 一步到位，宽高必须相等，不等就是椭圆，你们量一下 `56px` 对 `56px`，差 `1px` 都不行。

> **易错点**：`line-height` 等于盒子高度只能让单行内容垂直居中，图标字体正好是单行，所以 `line-height: 56px` 能把它顶到正中间。换成多行文字这招立刻失效，别乱套用。

### 2. 搜索框的胶囊造型

```html
<div class="search">
  <i class="iconfont icon-search"></i>
  <input type="search" placeholder="Search courses, topics, mentors…" />
  <button>Search</button>
</div>
```

输入框类型我用 `search` 不用 `text`，移动端弹出的键盘会带“搜索”按钮，语义也更准。放大镜图标放在输入框左边，按钮放在右边，整条做成胶囊形：

```css
.search {
  display: flex;
  align-items: center;
  max-width: 560px;
  margin: 0 auto;
  padding: 6px 6px 6px 20px;
  background-color: #fff;
  border: 1px solid #e5e5e5;
  border-radius: 999px;
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.06);
}
.search input {
  flex: 1;
  border: 0;
  outline: 0;
  font-size: 15px;
  background-color: transparent;
}
.search button {
  padding: 12px 28px;
  font-size: 15px;
  color: #fff;
  background-color: #5e5ce6;
  border: 0;
  border-radius: 999px;
  cursor: pointer;
}
```

三处细节你们照抄：第一，`border-radius: 999px` 是胶囊的通用写法，半径给得足够大，两头自动全圆，不用算高度的一半是多少；第二，输入框的边框和轮廓线（`border: 0; outline: 0`）必须清掉，否则胶囊里面再套一个方框，丑到没眼看；第三，`flex: 1` 让输入框吃掉中间全部剩余宽度，按钮宽度固定，两边放大镜和按钮一挤，输入框永远是自适应的。

> **提示**：占位文本（Placeholder）`placeholder` 里的提示语要写具体场景，“搜课程、主题、导师”比光写“请输入”好得多。用户看到例子才知道这个框能搜什么。

| 部件     | 布局手段              | 关键取值                      |
| :------- | :-------------------- | :---------------------------- |
| `.cate`  | 横向弹性、居中散开    | `justify-content: center; gap: 48px` |
| 图标圆底 | 等宽高等行高加全圆角  | `56px; border-radius: 50%`    |
| `.search`| 胶囊弹性条            | `border-radius: 999px; flex: 1` 输入框 |

> **结论**：图标行负责分流，搜索框负责直达，两者都是课程模块的入口。样式上记住两组固定搭配：图标导航用纵向弹性加居中，胶囊搜索用横向弹性加 `flex: 1`。下一集我们让整个课程模块在不同屏幕下自动变阵。

> 💡 **承前启后**：完成对「171-designcode响应式项目-图标和搜索模块制作」的理解后，下一章我们将深入探讨「172-designcode响应式项目-课程模块响应式效果布局」，进一步完善知识图谱体系。

---

## 第 10 章：172-designcode响应式项目-课程模块响应式效果布局
> 对应分集：P173 | 原始标题：《172-designcode响应式项目-课程模块响应式效果布局》

桌面端三列排得漂漂亮亮，一拖到平板就挤成一团，再拖到手机直接溢出横滚。怎么办？我给你们的答案就一句话：版心不动，列数动。这一集专门收拾课程模块在不同宽度下的变阵，断点（Breakpoint）取哪几个、每一档改什么，一次讲透。

### 1. 断点划分与变阵总览

我用两个断点把屏幕切成三档，这是我做响应式最常用的切法：

```text
桌面 (>1024px)     平板 (768~1024px)     手机 (<768px)
+---+---+---+      +-----+-----+        +---------+
| 1 | 2 | 3 |      |  1  |  2  |        |    1    |
+---+---+---+      +-----+-----+        +---------+
                   |  3  |  4  |        |    2    |
                   +-----+-----+        +---------+
```

读图说明：桌面三列，平板两列自动换行，手机单列 vertical 堆叠，列数只减不加。

```css
.courses-bd {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 32px;
}
@media (max-width: 1024px) {
  .courses-bd {
    grid-template-columns: repeat(2, 1fr);
    gap: 24px;
  }
}
@media (max-width: 768px) {
  .courses-bd {
    grid-template-columns: 1fr;
    gap: 20px;
  }
  .courses {
    padding: 64px 0;
  }
  .courses-hd h2 {
    font-size: 28px;
  }
}
```

你们看，媒体查询（Media Query）里我只改四样：列数、间隙、模块留白、标题字号。版心 `.w` 的 `max-width` 一行都不用动，它自己会缩。记住这个顺序：先写桌面默认样式，再从大往小写 `max-width` 覆盖，断点越小越往后写，后写的优先级自然盖住前面的。

> **定义**：断点指布局发生质变的屏幕宽度，本集取 `1024px` 和 `768px`，分别对应平板横屏和手机竖屏的上限。

### 2. 每一档改什么

第一档桌面端什么都不加，原样三列，这是基准。第二档 `1024px` 以下变两列，间隙从 `32px` 收到 `24px`，平板宽度本来就紧张，缝太大卡片会被挤扁。

第三档 `768px` 以下事情最多：

```css
@media (max-width: 768px) {
  .cate {
    gap: 24px;
    flex-wrap: wrap;
  }
  .search {
    max-width: 100%;
  }
}
```

图标行在手机上必须允许换行（`flex-wrap: wrap`），四个图标一排摆不下，硬撑只会溢出；搜索框的 `max-width: 560px` 在手机上要放开到 `100%`，让它撑满版心，按钮才按得舒服。

> **提示**：`flex-wrap: wrap` 和网格自动换行是一个意思：空间不够就往下一行掉。凡是横向一排、数量固定的东西，进手机前先问自己一句：要不要加换行？

| 屏幕档位 | 断点条件 | 课程列数 | 附带调整 |
| :------- | :------- | :------- | :------- |
| 桌面 | 默认 | `repeat(3, 1fr)` | 基准 |
| 平板 | `max-width: 1024px` | `repeat(2, 1fr)` | 间隙收至 `24px` |
| 手机 | `max-width: 768px` | `1fr` 单列 | 模块留白 `64px`、标题 `28px`、图标换行 |

> **易错点**：媒体查询里漏写 `meta viewport`（视口元标签），手机浏览器会按 `980px` 假宽度渲染，断点永远进不去。先检查 `width=device-width, initial-scale=1.0` 在不在，再调断点。

> **结论**：响应式不是重写一套页面，而是桌面基础上做减法：列数减、间隙减、字号减。你们把 `1024px` 变两列、`768px` 变一列这套背下来，任何卡片列表都能套。下一集钻进卡片内部，看卡片自己怎么跟着屏幕变形。

> 💡 **承前启后**：完成对「172-designcode响应式项目-课程模块响应式效果布局」的理解后，下一章我们将深入探讨「173-designcode响应式项目-课程卡片模块响应式布局」，进一步完善知识图谱体系。

---

## 第 11 章：173-designcode响应式项目-课程卡片模块响应式布局
> 对应分集：P174 | 原始标题：《173-designcode响应式项目-课程卡片模块响应式布局》

模块会变阵了，卡片自己还不会变形，照样翻车。我见过最多的翻车就是这个：三列变一列了，卡片里的图还是按桌面比例硬撑，手机上被拉成面条。这一集我就盯着单张卡片，看它从外框到配图，怎么在三档屏幕里都站得住。

### 1. 卡片外框与配图的基准写法

先把桌面端的卡片定死，后面响应式只改它，不改结构：

```html
<article class="card">
  <div class="card-cover">
    <img src="./images/course-figma.jpg" alt="Figma 课程封面" />
    <span class="tag">Design</span>
  </div>
  <div class="card-body">
    <h3>Design for Developers</h3>
    <p>12 lessons · 4.5 hours</p>
  </div>
</article>
```

卡片语义我用 `article`，它本身就是一篇独立内容，比 `div` 准。封面和正文上下两截，标签角标用绝对定位压在图上：

```css
.card {
  background-color: #fff;
  border-radius: 20px;
  overflow: hidden;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.08);
}
.card-cover {
  position: relative;
  aspect-ratio: 16 / 9;
  overflow: hidden;
}
.card-cover img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}
.card-cover .tag {
  position: absolute;
  left: 16px;
  top: 16px;
  padding: 6px 12px;
  font-size: 12px;
  color: #fff;
  background-color: rgba(0, 0, 0, 0.55);
  border-radius: 999px;
}
```

`aspect-ratio: 16 / 9` 是整张卡片的命根子：不管卡片多宽，封面永远按宽高比（Aspect Ratio）等比缩放，图永远不变形。`object-fit: cover` 让图片像背景图一样铺满裁掉多余部分，`overflow: hidden` 配 `border-radius: 20px` 把图片四个角一起切圆，不然图片直角会从圆角卡片里戳出来。

```text
+---------------+
| 16 : 9 封面图 |
| [Design]      |
+---------------+
| 标题 + 课时   |
+---------------+
```

读图说明：上半是等比封面加左上角角标，下半是标题正文，圆角外框一次包住。

> **提示**：纵横比（`aspect-ratio`）写在容器上，不是写在 `img` 上。容器定比例，图片 `100%` 铺满，这才是正顺序。反过来图片定死高度，卡片一变宽就露白边。

### 2. 卡片的三档响应式改法

```css
@media (max-width: 1024px) {
  .card {
    border-radius: 16px;
  }
}
@media (max-width: 768px) {
  .card {
    display: flex;
    gap: 16px;
    padding: 12px;
    border-radius: 16px;
  }
  .card-cover {
    flex: 0 0 128px;
    aspect-ratio: 1 / 1;
  }
  .card-body h3 {
    font-size: 16px;
  }
}
```

平板档我只收圆角，`20px` 变 `16px`，小卡片配小圆角，视觉才协调。手机档动大手术：卡片从上下结构翻成左右结构，封面从 `16:9` 横图压成 `1:1` 方图，固定 `128px` 宽，右边文字区自动伸缩。你们想想，手机单列时如果还上下堆，一张卡片占半屏，三张就得滚半天，改成左图右文，一屏能看两三个，刷课效率高得多。

> **易错点**：`flex: 0 0 128px` 三个值分别是放大、缩小、基准，`0 0` 就是锁死 `128px` 不许伸缩。只写 `width: 128px` 在弹性容器里会被挤小，锁死必须用这三个零。

| 屏幕档位 | 卡片形态 | 封面处理 | 原因 |
| :------- | :------- | :------- | :--- |
| 桌面 | 上下堆叠 | `16 / 9` 横图 | 宽卡片横图最舒展 |
| 平板 | 上下堆叠 | `16 / 9` 不变 | 只收圆角间隙 |
| 手机 | 左图右文 | `1 / 1` 方图 `128px` | 省纵向空间、一屏多看 |

> **结论**：卡片响应式的精髓是封面比例跟着形态走：上下堆用宽图，左右排用方图。比例用 `aspect-ratio` 锁，裁剪用 `cover` 铺，手机翻成横排时记得把图片锁死宽度。卡片外框站住了，下一集钻到卡片肚子里收拾那些小盒子。

> 💡 **承前启后**：完成对「173-designcode响应式项目-课程卡片模块响应式布局」的理解后，下一章我们将深入探讨「174-designcode响应式项目-课程卡片模块小盒子布局」，进一步完善知识图谱体系。

---

## 第 12 章：174-designcode响应式项目-课程卡片模块小盒子布局
> 对应分集：P175 | 原始标题：《174-designcode响应式项目-课程卡片模块小盒子布局》

外框会变形了，卡片肚子里那堆小东西还挤在一起：头像压着名字，时长顶着难度，价格和星级打架。这一集专门收拾 `.card-body` 里面的小盒子。你们记住我的口诀：小盒子一律横向弹性，左中右三段，两端顶头、中间自适应。

### 1. 正文区的小盒子拆分

一个卡片正文我拆三行：标题行、讲师行、数据行，每行都是一个横向小盒子：

```html
<div class="card-body">
  <h3>Design for Developers</h3>
  <div class="meta">
    <img class="avatar" src="./images/mentor.jpg" alt="导师头像" />
    <span class="name">Meng To</span>
    <span class="lessons">12 lessons</span>
  </div>
  <div class="foot">
    <span class="price">$49</span>
    <span class="rating">★ 4.9</span>
  </div>
</div>
```

```text
+----------------------+
| 标题 (单行省略)      |
+----------------------+
| (头像) 名字 ... 课时 |
+----------------------+
| $49           ★ 4.9  |
+----------------------+
```

读图说明：标题独占一行，讲师行左中右散开，底部价格和评分顶到两端。

```css
.card-body {
  padding: 20px;
}
.card-body h3 {
  font-size: 18px;
  line-height: 1.4;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
```

标题三件套 `nowrap + hidden + ellipsis` 是单行省略的固定写法，标题再长也只显示一行，多余的变省略号，卡片高度永远整齐。你们要是谁的卡片忽高忽低，八成是标题没做省略，长标题把盒子撑高了。

### 2. 讲师行与底部行的弹性写法

```css
.meta {
  display: flex;
  align-items: center;
  gap: 8px;
  margin-top: 12px;
  font-size: 13px;
  color: #666;
}
.meta .avatar {
  width: 28px;
  height: 28px;
  border-radius: 50%;
  object-fit: cover;
}
.meta .lessons {
  margin-left: auto;
}
.foot {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-top: 16px;
  padding-top: 16px;
  border-top: 1px solid #eee;
}
.foot .price {
  font-size: 18px;
  font-weight: 700;
  color: #0a0a0a;
}
.foot .rating {
  font-size: 13px;
  color: #f5a623;
}
```

讲师行里 `margin-left: auto` 是个巧劲：头像名字靠左，课时数被 `auto` 外边距一顶，自动弹到最右边，中间留白自动消化，比算宽度、比 `space-between` 都省事。头像 `28px` 等宽高加全圆，`object-fit: cover` 保证方图切圆不变形。

底部行用 `space-between` 把价格和评分顶到两端，顶上加一条 `1px` 分隔线，数据区和内容区一切为二。价格又黑又大，评分金色点缀，眼睛先看价格再看分，这个轻重别颠倒。

> **定义**：自动外边距（Auto Margin）在弹性布局里会吃掉全部剩余空间，`margin-left: auto` 等于说“把我推到最右边”，是单侧顶边的标准做法。

> **易错点**：头像只写 `border-radius: 50%` 不锁宽高，长方形图片切出来是椭圆。宽高相等加 `object-fit: cover` 三件套缺一不可。

| 小盒子 | 手法 | 关键属性 |
| :----- | :--- | :------- |
| 标题行 | 单行省略 | `white-space: nowrap; text-overflow: ellipsis` |
| 讲师行 | 左贴右弹 | `display: flex; margin-left: auto` |
| 底部行 | 两端对齐加分隔线 | `justify-content: space-between; border-top` |

> **结论**：卡片内部没有大布局，全是小盒子的横向弹性：标题防撑高用省略，讲师行用自动边距弹开两端，底部用两端对齐加线。到这里课程卡片从外框到内胆全部收工，下一集给卡片加上鼠标经过的浮起和阴影，整个课程模块才算真正活起来。

> 💡 **承前启后**：完成对「174-designcode响应式项目-课程卡片模块小盒子布局」的理解后，下一章我们将深入探讨「175-designcode响应式项目-课程卡片模块鼠标经过效果」，进一步完善知识图谱体系。

---

## 第 13 章：175-designcode响应式项目-课程卡片模块鼠标经过效果
> 对应分集：P176 | 原始标题：《175-designcode响应式项目-课程卡片模块鼠标经过效果》

我跟你们说，静态页面和有生命页面的分界线，就在鼠标放上去那 0.3 秒。你们前面已经把课程卡片的大盒子、小盒子都摆好了，位置是对的，可鼠标划过去一点反应都没有，那就还是 demo 的样子。这一讲我们就干一件事：把 `hover` 加上，让卡片会浮起来、图片会放大、箭头会冒出来。

我做这类效果有个习惯，先想清楚用户能感知到哪三层变化：整张卡片的位置和阴影、里面图片的缩放、还有文字区小元素的位移。你们记住这一条就够了，hover 永远是分层做的，一层一个 `transition`，别指望一个属性包打天下。

### 1 卡片整体上浮与阴影

最先要改的是 `.course-card` 本身。我让它默认就带一个很淡的阴影，鼠标经过时往上走 `8px`，阴影加深、扩散更大。这里容易搞错的是，`transition` 一定要写在卡片本身上，而不是写在 `:hover` 上。

写在 `:hover` 上会怎样？移入有动画，移出唰一下弹回去，特别生硬。你们一定要把过渡写在常态，我反复强调过，动画写常态、变化写 hover。

```css
.course-card {
  background-color: #fff;
  border-radius: 20px;
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.06);
  transition: transform 0.3s ease, box-shadow 0.3s ease;
}
.course-card:hover {
  transform: translateY(-8px);
  box-shadow: 0 20px 48px rgba(0, 0, 0, 0.14);
}
```

> **结论**：`transform: translateY(-8px)` 只做视觉位移，不占据文档流，所以不会把旁边的卡片挤开，这正是它比改 `margin` 好的地方。

为什么是 8px？不是 4px、不是 20px？我试过，4px 用户几乎感觉不到，20px 整排卡片跳得太厉害，8px 是刚好能感知又不乱的那个点。你们以后调 hover 位移，就在 6px 到 12px 之间试，别贪大。

### 2 图片缩放与溢出裁剪

第二层是图片。我希望鼠标放上去，图片轻轻放大到 1.08 倍，有一种镜头推近的感觉。这里有两个配套动作缺一不可：图片容器要 `overflow: hidden`，图片本身要 `transition: transform`。

```css
.course-card .pic {
  overflow: hidden;
  border-radius: 20px 20px 0 0;
}
.course-card .pic img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: transform 0.5s ease;
}
.course-card:hover .pic img {
  transform: scale(1.08);
}
```

> **易错点**：图片放大用 `scale`，时间我给了 `0.5s`，比卡片上浮的 `0.3s` 慢一拍。这样快慢错开，层次感一下就出来了。如果全写成一样的时长，画面会发闷。

你们自己试一下，把 `overflow: hidden` 去掉再 hover，图片会直接撑破圆角漫出来，整个卡片就露馅了。所以裁剪和缩放永远是一对，记住这一条就够了。

### 3 文字区箭头与信息显现

第三层藏在文字区。课程卡片右下角那个小箭头，默认是半透明、往左缩一点的，hover 时它滑出来、变实。我用的是 `opacity` 加 `translateX` 的组合。

```css
.course-card .more {
  opacity: 0.4;
  transform: translateX(-6px);
  transition: opacity 0.3s ease, transform 0.3s ease;
}
.course-card:hover .more {
  opacity: 1;
  transform: translateX(0);
}
```

这个细节很多人会漏掉：小元素位移的距离一定要小，6px 就够了。你给 20px，它就像弹射出去一样，特别廉价。

三层叠在一起，整个 hover 的结构是这样的：

```text
+----------------------------------+
|  .course-card                    |
|   translateY(-8px) + shadow加深  |
|  +------------------------------+|
|  | .pic (overflow:hidden)       ||
|  |  img scale(1.0 -> 1.08)      ||
|  +------------------------------+|
|  | .info                        ||
|  |  .more opacity .4 -> 1       ||
|  +------------------------------+|
+----------------------------------+
```

读图说明：外框是卡片整体位移，中框是图片裁剪区内的缩放，底框是文字区小元素的淡入滑出，三层各管各的过渡。

### 4 过渡时长与性能取舍

最后说个当年踩过的坑。我以前给所有 hover 都加 `transition: all 0.3s`，省事是省事，可浏览器要对每个属性都做插值，卡片一多，低端机直接掉帧。后来我就只写要动的两个属性，从来不写 `all`。

| 写法 | 效果 | 建议 |
| :--- | :--- | :--- |
| `transition: all 0.3s` | 全属性过渡，方便但费性能 | 不用 |
| `transition: transform 0.3s, box-shadow 0.3s` | 只过渡变化项，流畅 | 采用 |
| 常态写过渡，hover 写变化 | 进出都有动画 | 必须如此 |

> **提示**：`transform` 和 `opacity` 是合成器能直接处理的属性，不触发重排，所以 hover 动画优先用它俩。阴影变化会引起重绘，但卡片数量不大时完全可以接受。

这一讲你们亲手把 hover 加完，再把鼠标来回划几遍，感受一下 0.3 秒和 0.5 秒错开的节奏。下一讲我们进入故事卡片模块，那是整页最出彩的一屏。

> 💡 **承前启后**：完成对「175-designcode响应式项目-课程卡片模块鼠标经过效果」的理解后，下一章我们将深入探讨「176-designcode响应式项目-故事卡片模块搭建」，进一步完善知识图谱体系。

---

## 第 14 章：176-designcode响应式项目-故事卡片模块搭建
> 对应分集：P177 | 原始标题：《176-designcode响应式项目-故事卡片模块搭建》

你们发现没有，前面课程卡片做得再漂亮，用户也只是觉得能用。真正让人记住这个页面的，是故事区那几张人物卡片。我跟你们讲，故事模块拼的不是特效，是结构：左边一张大卡讲主角，右边几张小卡讲配角，图片人物、引言文字、署名信息，一层都不能少。这一讲我们先把骨架搭结实，响应式下一讲再拆。

我上来先带你们看布局，很多同学一看到一左一右就想用浮动，那是老办法了。现在我直接用网格（Grid，栅格布局）起手，外层 `.stories` 定两列，左边大卡占一列，右边小卡列表占一列，思路一下就顺了。

### 1 故事区整体结构

整个模块我分成三段：标题区、大卡片区、小卡片列表区。标题区就是一句大标题加一句副文案，大卡是人物大图加引言，小卡是横向的小图文。你们先把语义想清楚再写标签， section 包 h2 加 p，卡片用 article，这都是有含义的。

```html
<section class="stories">
  <div class="stories-head">
    <h2>学员故事</h2>
    <p>看看他们是怎么从零基础走到独立做项目的</p>
  </div>
  <div class="stories-body">
    <article class="story-main">
      <div class="pic"><img src="./images/story-1.jpg" alt="学员作品展示" /></div>
      <div class="info">
        <p class="quote">从第一行标签到上线自己的响应式页面，我只用了两个月。</p>
        <div class="user"><img src="./images/avatar-1.jpg" alt="学员头像" /><span>林小白 · 前端学员</span></div>
      </div>
    </article>
    <div class="story-list">
      <article class="story-item">
        <div class="pic"><img src="./images/story-2.jpg" alt="学员作品展示" /></div>
        <div class="info">
          <p>以前最怕布局，现在 grid 和 flex 随手就来。</p>
          <span>阿哲 · 转行学员</span>
        </div>
      </article>
    </div>
  </div>
</section>
```

> **定义**：引言（quote）指卡片里那句第一人称的学员原话，它是故事卡的灵魂，字号和行高要单独给，不能跟普通正文混用。

你们写的时候一定注意，图片必须给 `alt`，这是习惯问题。今天少写一个 `alt` 没人罚你，哪天做企业站，缺 `alt` 就是硬伤。

### 2 大卡片样式

大卡我给了圆角 `24px`、白色底、深阴影，图片在上、文字在下。图片高度我定死 `320px`，配 `object-fit: cover`，这样不管原图是横是竖，裁出来都一样齐。

```css
.story-main {
  background-color: #fff;
  border-radius: 24px;
  overflow: hidden;
  box-shadow: 0 16px 40px rgba(0, 0, 0, 0.08);
}
.story-main .pic img {
  width: 100%;
  height: 320px;
  object-fit: cover;
}
.story-main .info {
  padding: 28px;
}
.story-main .quote {
  font-size: 20px;
  line-height: 1.6;
  color: #111827;
}
.story-main .user {
  display: flex;
  align-items: center;
  gap: 12px;
  margin-top: 20px;
}
.story-main .user img {
  width: 44px;
  height: 44px;
  border-radius: 50%;
}
```

头像为什么是正圆？`border-radius: 50%` 配等宽等高，那就是圆。你们要是宽高写得不一样，出来就是椭圆，特别容易被看出不专业。

整体层级关系是这样的：

```text
.stories
  |-- .stories-head (h2 + p)
  |-- .stories-body [grid: 2列]
        |-- .story-main (大卡: 大图 + 引言 + 署名)
        |-- .story-list (小卡列)
              |-- .story-item x N (小图 + 短引言)
```

读图说明：标题独占一行，主体两列中左列一张大卡，右列纵向排列多张小卡，大小卡共用圆角和阴影语言。

### 3 小卡片列表样式

小卡跟大卡是反过来的：横向放，左边小图、右边文字。图片我给了 `120px` 宽，固定不伸缩，文字区用 `flex: 1` 把剩下宽度吃满。小卡之间用 `gap: 20px` 拉开，比写 `margin` 省事得多。

```css
.story-item {
  display: flex;
  gap: 16px;
  align-items: center;
  background-color: #fff;
  border-radius: 18px;
  padding: 16px;
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.06);
}
.story-item .pic img {
  width: 120px;
  height: 96px;
  border-radius: 12px;
  object-fit: cover;
  flex-shrink: 0;
}
```

> **易错点**：小图一定要加 `flex-shrink: 0`。不加的话，屏幕稍窄一点，弹性盒（flex）就会先压缩图片，人物脸直接被压扁。我见过太多同学在这里翻车，记住这一条就够了。

这一讲我们把故事区的大卡小卡都立起来了，桌面端看已经像模像样。下一讲我们做它的响应式，到平板和手机上，大卡小卡该堆叠堆叠，该满宽满宽。

> 💡 **承前启后**：完成对「176-designcode响应式项目-故事卡片模块搭建」的理解后，下一章我们将深入探讨「177-designcode响应式项目-故事卡片模块响应式布局」，进一步完善知识图谱体系。

---

## 第 15 章：177-designcode响应式项目-故事卡片模块响应式布局
> 对应分集：P178 | 原始标题：《177-designcode响应式项目-故事卡片模块响应式布局》

上一讲我们把故事卡片搭起来了，在宽屏上看挺漂亮。可你们把浏览器往窄里一拖就露馅了：大卡把小卡挤得没地方站，文字被压成一竖行。这就是响应式（responsive）要解决的事。我跟你们说，响应式不是重写一套页面，而是让同一套结构在不同宽度下换一种排法，这一讲我们就给故事区换排法。

我的思路很直接：宽屏两列、平板前后堆、手机全部单列。断点（breakpoint）我就用两个，`1024px` 和 `768px`，别一上来定五六个，把自己绕晕。记住这一条就够了，断点越少越好维护。

### 1 两列转单列的断点划分

桌面端 `.stories-body` 是 `grid` 两列，大卡小卡左右放。到 `1024px` 以下，我让它变成一列，大卡在上、小卡列表在下。到 `768px` 以下，小卡内部再从横排变竖排，图片顶满全宽。

```css
.stories-body {
  display: grid;
  grid-template-columns: 1.2fr 1fr;
  gap: 28px;
}
@media (max-width: 1024px) {
  .stories-body {
    grid-template-columns: 1fr;
  }
  .story-main .pic img {
    height: 260px;
  }
}
@media (max-width: 768px) {
  .story-item {
    flex-direction: column;
    align-items: stretch;
  }
  .story-item .pic img {
    width: 100%;
    height: 180px;
  }
}
```

> **提示**：媒体查询（media query）一定要按从宽到窄的顺序写，后写的窄断点才能覆盖宽断点。顺序写反了，手机样式会被平板样式盖掉，怎么调都不生效。

为什么大卡列给 `1.2fr` 而不是 `1fr`？因为主角就该比配角宽一点，视觉重心才稳。`fr`（fraction，剩余空间份数）这个单位你们现在应该很熟了，它分的是剩下的空间，不是整个宽度。

### 2 小卡片列表的换行处理

有个细节你们在平板上一定会遇到：转成单列后，小卡列表如果还是纵向三张，会把页面拉得很长。我在这里让小卡在平板上先变成两列网格，到手机再变回单列，这样长度和密度都刚刚好。

```css
@media (max-width: 1024px) {
  .story-list {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 20px;
  }
}
@media (max-width: 768px) {
  .story-list {
    grid-template-columns: 1fr;
  }
}
```

断点前后的对照关系是这样的：

```text
宽度 >1024px        1024px以下         768px以下
.stories-body       .stories-body      .stories-body
[大卡 | 小卡列]  -> [大卡]          -> [大卡]
                    [小卡 小卡]       [小卡]
                    [小卡 ... ]       [小卡]
                                      [小卡]
```

读图说明：从左到右宽度递减，主体从左右两列变为上下堆叠，小卡列表先是两列网格最后变为单列，图片由固定小图变为全宽大图。

| 宽度区间 | 大卡 | 小卡列表 | 小卡内部 |
| :--- | :--- | :--- | :--- |
| 大于 1024px | 左列 1.2fr | 右列 1fr 纵排 | 横向图文 |
| 768px 到 1024px | 独占一行 | 两列网格 | 横向图文 |
| 小于 768px | 独占一行 | 单列 | 竖向图文，图片全宽 |

### 3 图片高度与文字缩放

最后别忘了字。桌面端引言 `20px`，到手机上不减到 `17px`，一行装不下几个字，读起来特别累。图片高度也要跟着降，大卡从 `320px` 降到 `220px`，不然手机一屏全是图，看不到字。

```css
@media (max-width: 768px) {
  .story-main .info {
    padding: 20px;
  }
  .story-main .quote {
    font-size: 17px;
  }
  .story-main .pic img {
    height: 220px;
  }
}
```

> **易错点**：改响应式只改列数不改字号和图高，是新手最常见的半截子工程。布局换了，内容尺度也要换，这两个永远一起调。

你们调完一定要亲手从 1400px 一路拖到 360px 看三遍，重点看 1024 和 768 附近有没有横向滚动条。一旦出现横向滚动条，百分之百是有固定宽度没改掉，顺着找就行。下一讲我们玩 `clip-path`，给页面裁出波浪分界线。

> 💡 **承前启后**：完成对「177-designcode响应式项目-故事卡片模块响应式布局」的理解后，下一章我们将深入探讨「178-designcode响应式项目-clip path裁剪波浪图形」，进一步完善知识图谱体系。

---

## 第 16 章：178-designcode响应式项目-clip path裁剪波浪图形
> 对应分集：P179 | 原始标题：《178-designcode响应式项目-clip path裁剪波浪图形》

你们看原站故事区和底部之间那条波浪分界线，是不是比一条死直线高级得多？很多同学以为那是一张图片，其实我告诉你们，一行 `clip-path` 就能裁出来，根本不用切图。这一讲我们就把裁剪（clip-path）这个属性玩透，顺手把这道波浪做出来。

我先问你们一个问题：一个矩形的 section，怎么让它的底边变成弯的？答案是别想着加东西，而是把多余的部分剪掉。`clip-path` 就是那把剪刀，你给它一串坐标，它只留下坐标圈住的区域。

### 1 裁剪属性基本写法

裁剪属性里最常用的是 `polygon`（多边形），括号里一对一对写 `x y` 坐标，用百分比。我先拿梯形举例，你们一下就明白坐标是怎么走的。

```css
.wave {
  clip-path: polygon(0 0, 100% 0, 100% 85%, 0 100%);
}
```

这四个点连起来，底边就是一条斜线。你们记住，坐标是顺时针绕一圈，起点回到起点，圈住的留下，圈外的剪掉。

> **定义**：裁剪路径（clip-path）指用几何函数圈定元素可见区域的属性，被裁掉的部分不占视觉显示，但元素本身的盒子还在，只是看不见。

那波浪呢？光靠直线点是画不出弧线的，这时候就换 `ellipse`（椭圆）或者直接用路径。我这一版用的是椭圆裁法，一行就出大圆弧，特别适合做区块之间的柔和过渡。

```css
.stories {
  clip-path: ellipse(75% 100% at 50% 0%);
}
```

这句话翻译过来就是：以顶部中心为圆心，画一个宽 75%、高 100% 的椭圆，只留椭圆里的内容。这样底边自然就是一道弧线了。

### 2 波浪分界线实现

真正的波浪要起伏两次，光一个椭圆不够，我最后用的是多点 `polygon` 加密点拟合。你们别被点多吓到，规律很简单：顶边两个点固定，底边从左到右摆一串高低错开的点。

```css
.divider {
  height: 120px;
  background-color: #0b0d12;
  clip-path: polygon(0 60%, 10% 40%, 20% 55%, 30% 35%, 40% 55%, 50% 40%, 60% 60%, 70% 45%, 80% 60%, 90% 45%, 100% 60%, 100% 100%, 0 100%);
}
```

你们看，前十一个点是波浪的顶边，最后两个点把底边封口，这样深色块的上沿就是波浪，下半截是实心的，正好压在两个区块之间做衔接。

裁剪前后的对比是这样的：

```text
裁剪前 (矩形)              裁剪后 (波浪顶边)

+--------------+          .-._.-._.-._.-.
|              |         /               \
|   .divider   |   -->  |                 |
|              |         |                 |
+--------------+         +-----------------+
```

读图说明：左侧是完整矩形，右侧是顶边被波浪线重裁后的形状，多余的白色三角都被剪掉，露出下一区块的底色。

> **提示**：波浪点的 `y` 值我都在 `35%` 到 `60%` 之间摆动，振幅控制在 `25%` 以内。振幅太大，波浪像锯齿；振幅太小，远看又像直线，这个区间你们直接拿去用。

### 3 响应式下的裁剪适配

还有一个坑我必须提醒你们：`clip-path` 用百分比写，天然就是响应式的，宽度变了波浪会跟着缩放。可高度我写死了 `120px`，到手机上这个高度会显得太厚，所以小屏要把高度压到 `70px`。

```css
@media (max-width: 768px) {
  .divider {
    height: 70px;
  }
}
```

| 方案 | 做法 | 问题 |
| :--- | :--- | :--- |
| 切一张波浪图片做背景 | 图片定位到底边 | 拉伸变形，还多一次请求 |
| 用 `clip-path` 百分比裁剪 | 纯 CSS，一行属性 | 缩放自适应，无额外资源 |

> **结论**：能用 CSS 裁出来的形状，就别用图片。少一张图，少一次请求，缩放还不变形，这笔账怎么算都划算。

这一讲的波浪你们亲手裁出来，再拖动窗口看看它跟着伸缩，就知道百分比坐标的好处了。下一讲我们收尾，把底部模块搭完，整个 DesignCode 响应式项目就闭环了。

> 💡 **承前启后**：完成对「178-designcode响应式项目-clip path裁剪波浪图形」的理解后，下一章我们将深入探讨「179-designcode响应式项目-底部模块制作以及响应式布局」，进一步完善知识图谱体系。

---

## 第 17 章：179-designcode响应式项目-底部模块制作以及响应式布局
> 对应分集：P180 | 原始标题：《179-designcode响应式项目-底部模块制作以及响应式布局》

终于到最后一块了，底部（footer）模块。你们别小看页脚，用户滑到这里，意味着他把整页都看完了，肯不肯点个订阅、关不关注社交账号，全看这一屏。我跟你们说，页脚拼的是信息分组：左边品牌加订阅，中间几列链接，底部一行版权加社交图标，分组清楚，一点都不乱。

这一讲我们两件事一起干：先把四列结构搭出来，再把响应式收尾。到手机上四列变一列，订阅框顶满，链接两两并排，这就是标准答案。

### 1 页脚四列结构搭建

我用的还是语义化标签，外层 `footer`，里面 `.footer-top` 放四列，`.footer-bottom` 放版权行。四列分别是品牌订阅、课程链接、公司链接、资源链接。订阅那一列最宽，给 `1.4fr`，后面三列各 `1fr`。

```html
<footer class="footer">
  <div class="footer-top">
    <div class="brand">
      <img src="./images/logo.png" alt="网站标志" />
      <p>从零开始，做出能上线的响应式页面。</p>
      <form class="subscribe">
        <input type="email" placeholder="输入你的邮箱" />
        <button type="submit">订阅</button>
      </form>
    </div>
    <dl class="links">
      <dt>课程</dt>
      <dd><a href="#">前端入门</a></dd>
      <dd><a href="#">响应式布局</a></dd>
      <dd><a href="#">实战项目</a></dd>
    </dl>
    <dl class="links">
      <dt>公司</dt>
      <dd><a href="#">关于我们</a></dd>
      <dd><a href="#">加入我们</a></dd>
      <dd><a href="#">联系方式</a></dd>
    </dl>
    <dl class="links">
      <dt>资源</dt>
      <dd><a href="#">帮助中心</a></dd>
      <dd><a href="#">社区</a></dd>
      <dd><a href="#">更新日志</a></dd>
    </dl>
  </div>
  <div class="footer-bottom">
    <p>© 2025 DesignCode · 保留所有权利</p>
    <div class="social"><a href="#">B站</a><a href="#">微博</a><a href="#">微信</a></div>
  </div>
</footer>
```

> **提示**：链接组我用 `dl` 加 `dt` 加 `dd` 写，标题是 `dt`，链接是 `dd`。这比全用 `div` 加 `a` 语义准得多，读屏软件也能直接读出分组关系。

订阅表单那一行，输入框和按钮我包在一个圆角胶囊里，输入框去掉边框、背景透明，按钮给深色底。你们记住这一条就够了，页脚的订阅框永远做成胶囊形，用户一眼就知道这是要填的。

```css
.footer-top {
  display: grid;
  grid-template-columns: 1.4fr 1fr 1fr 1fr;
  gap: 32px;
}
.subscribe {
  display: flex;
  align-items: center;
  background-color: #f3f4f6;
  border-radius: 999px;
  padding: 6px 6px 6px 18px;
}
.subscribe input {
  flex: 1;
  border: 0;
  outline: none;
  background-color: transparent;
}
.subscribe button {
  border: 0;
  border-radius: 999px;
  padding: 10px 22px;
  background-color: #111827;
  color: #fff;
  cursor: pointer;
}
```

整体分块关系是这样的：

```text
.footer
  |-- .footer-top [grid 4列]
  |     |-- .brand (logo + 简介 + 订阅胶囊) 1.4fr
  |     |-- .links 课程  1fr
  |     |-- .links 公司  1fr
  |     |-- .links 资源  1fr
  |-- .footer-bottom [flex 左右]
        |-- 版权文字  +  社交链接
```

读图说明：顶部四列中品牌列最宽含订阅表单，三组链接等宽并排，底部一行左右分布版权与社交入口。

### 2 页脚响应式布局

到 `1024px` 以下，四列先收成两列，品牌独占第一行通栏，后面三组链接三变两加一，刚好两行。到 `768px` 以下，全部单列，链接组改成两列小网格，省高度。

```css
@media (max-width: 1024px) {
  .footer-top {
    grid-template-columns: 1fr 1fr;
  }
  .brand {
    grid-column: 1 / -1;
  }
}
@media (max-width: 768px) {
  .footer-top {
    grid-template-columns: 1fr;
  }
  .footer-bottom {
    flex-direction: column;
    gap: 12px;
    text-align: center;
  }
  .subscribe {
    width: 100%;
  }
}
```

> **易错点**：`grid-column: 1 / -1` 是让品牌列横跨整行，从第一条网格线直接跨到最后一条。很多同学在这里写 `span 2`，列数一变就错位，写 `1 / -1` 不管几列都通吃。

| 宽度区间 | 顶部排法 | 底部排法 |
| :--- | :--- | :--- |
| 大于 1024px | 四列，品牌 1.4fr | 左右分布 |
| 768px 到 1024px | 两列，品牌通栏 | 左右分布 |
| 小于 768px | 单列堆叠 | 上下居中堆叠 |

订阅框到手机上一定要给 `width: 100%`，不然它缩在左上角，右边空一大截，特别难看。按钮的字也别太多，订阅两个字就够了，写立即订阅四个字，窄屏下输入框会被挤得只剩一半。

做到这里，DesignCode 从顶栏、导航、课程卡、故事卡、波浪到页脚，整条线就串起来了。你们回头从 1400px 拖到 360px 完整走一遍，没有横向滚动、没有图片变形、hover 进出都有动画，这个项目就算真正过关了。

> 💡 **承前启后**：完成对「179-designcode响应式项目-底部模块制作以及响应式布局」的理解后，下一章我们将深入探讨「180-designcode响应式项目-小圆飘动动画效果」，进一步完善知识图谱体系。

---

## 第 18 章：180-designcode响应式项目-小圆飘动动画效果
> 对应分集：P181 | 原始标题：《180-designcode响应式项目-小圆飘动动画效果》

刚把 DesignCode 首页的头部和卡片区调顺，整个页面看着还是有点“死”，缺一口气。你们有没有这种感觉：布局都对了，配色也有了，可就是不像那些获奖站点那样有呼吸感？我就卡在这里，这才决定把装饰用的小圆点做成飘动的。

我和你们一样，一开始就想给页面加动效，但又怕动效太重拖慢加载。想来想去，这种小圆装饰最合适：实现就几行 `CSS`，不依赖 `JS`，还能让首屏一下子活起来。这一讲我们就把它一次做对。

---

### 1. 小圆装饰的定位结构

先想清楚结构。我在头部大标题区放了两三个小圆，有实心的，有半透明描边的，大小也不一样。它们不能占文档流，否则会把标题挤开，所以统一用绝对定位压在背景层。

```text
header .banner 层叠关系
+----------------------------+
|  .banner (relative)        |
|  +----------------------+  |
|  | h1 + p + btn (正常流) |  |
|  +----------------------+  |
|  (o) .dot1  absolute      |
|      (o) .dot2 absolute   |
|  +----------------------+  |
+----------------------------+
```

读图说明：`.banner` 做定位上下文，文字走正常流，小圆全部绝对定位浮在上层，互不干扰。

代码结构我习惯这样写：

```html
<div class="banner">
  <h1>DesignCode 创意设计课程</h1>
  <p>从 0 到 1 学会响应式与动效</p>
  <a class="btn" href="#">立即学习</a>
  <span class="dot dot1"></span>
  <span class="dot dot2"></span>
  <span class="dot dot3"></span>
</div>
```

```css
.banner {
  position: relative;
  overflow: hidden;
}
.dot {
  position: absolute;
  border-radius: 50%;
  pointer-events: none;
}
.dot1 {
  width: 18px;
  height: 18px;
  background: #5b8cff;
  left: 12%;
  top: 30%;
}
.dot2 {
  width: 32px;
  height: 32px;
  border: 3px solid #ff7a9e;
  opacity: 0.6;
  right: 15%;
  top: 22%;
}
.dot3 {
  width: 12px;
  height: 12px;
  background: #ffc94d;
  left: 48%;
  bottom: 18%;
}
```

> **提示**：`pointer-events: none` 一定要加，否则小圆会挡住按钮点击，你们点半天没反应，还以为是 `JS` 坏了。

> **易错点**：父级忘记写 `position: relative`，小圆就会满屏乱跑，直接以 `body` 为基准了。

---

### 2. 飘动关键帧与错峰控制

光摆在那还不够，得让它们动。我要的是那种轻轻上浮下沉的感觉，不是满屏乱飞，所以只动 `transform: translateY`，不动 `top`，性能也最好。

```css
@keyframes floatY {
  0% {
    transform: translateY(0);
  }
  50% {
    transform: translateY(-18px);
  }
  100% {
    transform: translateY(0);
  }
}
.dot1 {
  animation: floatY 3s ease-in-out infinite;
}
.dot2 {
  animation: floatY 4.2s ease-in-out 0.6s infinite;
}
.dot3 {
  animation: floatY 3.6s ease-in-out 1.1s infinite;
}
```

为什么三个圆我要写三种时长加延迟？你们要是写成一模一样，它们就会像阅兵一样齐上齐下，假得很。错开时长和延迟，每个圆的相位都不一样，看起来才是随意飘的。

```text
时间 -> 3s 周期内位移曲线
dot1 :  /\    /\    (快, 3s)
dot2 : /  \__/  \__ (慢, 4.2s + 延迟0.6s)
dot3 :  _/\_  _/\_  (中, 3.6s + 延迟1.1s)
```

读图说明：周期和起始相位错开后，三个圆在同一时刻处于波形的不同位置，视觉上自然错峰。

> **定义**：位移动画（`transform` animation）指只改变 `transform` 与 `opacity` 的动画，不触发重排，浏览器可以用合成器线程独立渲染。

> **结论**：装饰动效优先用 `transform` 和 `opacity`，绝不用 `top`、`left`、`margin` 做逐帧动画，那会引起整页重排，小圆一动，标题跟着抖。

我还加了一个细节：给大圆多加一层缩放呼吸，让它忽大忽小一点点：

```css
@keyframes floatScale {
  0%, 100% {
    transform: translateY(0) scale(1);
  }
  50% {
    transform: translateY(-22px) scale(1.12);
  }
}
.dot2 {
  animation: floatScale 4.2s ease-in-out 0.6s infinite;
}
```

记住这一条就够了：装饰动效幅度要小，`18px` 到 `22px` 足够了，太大就喧宾夺主，用户只看球不看字了。

---

### 3. 响应式下的收敛处理

做到这里在桌面端很漂亮，一缩到手机宽度就露馅：小圆压住标题，或者撑出横向滚动条。这里容易搞错，很多人只写桌面一套就交差了。

我的做法是两手：

```css
@media (max-width: 768px) {
  .dot2 {
    display: none;
  }
  .dot1 {
    width: 12px;
    height: 12px;
    left: 6%;
    top: 18%;
  }
  .dot3 {
    left: auto;
    right: 8%;
    bottom: 12%;
  }
}
```

一是藏起最碍事的大圆，二是把剩下的缩小并往边角赶。配合父级的 `overflow: hidden`，小屏下再也不会出现横向滚动条。

| 断点 | 处理策略 | 原因 |
| :--- | :--- | :--- |
| `> 992px` | 三圆全显示，原尺寸飘动 | 空间充足，装饰拉满 |
| `<= 768px` | 藏大圆，其余缩小靠边 | 防止压字与横向溢出 |
| `<= 480px` | 进一步减小动画幅度 | 小屏下大幅位移很晃眼 |

你们照着改完，刷新从大屏一路拖到手机宽度看一遍：标题始终可读，按钮始终可点，小圆始终是配角，这才算合格。下一讲我们回头收那两个遗留 `bug`，页面就真正能上线了。

> 💡 **承前启后**：完成对「180-designcode响应式项目-小圆飘动动画效果」的理解后，下一章我们将深入探讨「181-designcode响应式项目-修复两个bug」，进一步完善知识图谱体系。

---

## 第 19 章：181-designcode响应式项目-修复两个bug
> 对应分集：P182 | 原始标题：《181-designcode响应式项目-修复两个bug》

小圆飘起来之后我挺得意，拖着浏览器从大屏缩到小屏来回测，结果一下子测出两个毛病：一个是导航在小屏下切来切去状态全乱了，一个是页面底部莫名多出一截横向滚动条。你们做响应式肯定也撞过这种“桌面好好的，一缩就崩”的情况，这一讲我们就把这两个坑填平。

---

### 1. 导航切换状态残留问题

先说导航。我用的是经典做法：大屏显示横排菜单，小屏收成汉堡按钮，点击展开下拉。逻辑本身不难，难在窗口缩放时状态没同步。

复现路径很固定：先把窗口缩到 `768px` 以下，点汉堡把菜单打开，再把窗口拉回大屏，菜单就卡在展开的 mobile 样式上，横排全乱了。反过来也一样：大屏好好的，缩到小屏菜单却默认摊开着，把首屏顶下去一大截。

```text
状态残留复现路径
大屏(菜单横排) --缩小--> 小屏(菜单应收起, 实则摊开)
小屏(菜单展开) --放大--> 大屏(菜单应横排, 实则竖排残留)
```

读图说明：断点切换时若只靠点击事件控制显隐，不监听宽度变化，旧状态会带到新断点。

根因在我自己：我用 `JS` 给导航加了 `open` 类做展开收起，但媒体查询里没把这个类的影响对冲掉。修复要两端一起改。

```css
/* 默认 mobile 下菜单收起 */
.nav-list {
  display: none;
}
.nav-list.open {
  display: flex;
  flex-direction: column;
}
/* 回到大屏, 强制横排, 无视 open */
@media (min-width: 769px) {
  .nav-list {
    display: flex;
    flex-direction: row;
  }
}
```

```javascript
const toggleBtn = document.querySelector('.nav-toggle');
const navList = document.querySelector('.nav-list');

toggleBtn.addEventListener('click', () => {
  navList.classList.toggle('open');
});

// 关键补丁: 跨断点时清掉残留状态
window.addEventListener('resize', () => {
  if (window.innerWidth > 768 && navList.classList.contains('open')) {
    navList.classList.remove('open');
  }
});
```

> **易错点**：只写点击切换、不写 `resize` 复位，是响应式导航最常见的半截工程。一定要如此，绝不能那般：断点切换必须有状态兜底。

> **提示**：`resize` 里不要做重活，只做类名清理。菜单的显隐全部交给 `CSS`，`JS` 只负责换类，这样缩放时才不会闪。

改完再按刚才的路径测两遍：小屏点开再拉大，大屏缩到小，菜单都老老实实回到各自断点该有的样子，这个 `bug` 才算修死。

---

### 2. 装饰溢出导致的横向滚动条

第二个 `bug` 更隐蔽。页面在手机宽度下左右能轻微晃动，底部出现横向滚动条。我一开始还以为是某张图没压住，逐段删着查，最后定位到上一讲加的小圆和卡片区的装饰条：绝对定位的元素一半探出父盒子，把 `body` 撑宽了。

```text
溢出示意 (俯视宽度方向)
+-------- body 375px --------+
| .banner (overflow可见)     |
|   文字区                   |
|              (o) dot2 ->| 探出 20px
|                         |  ==> body 被撑到 395px
+----------------------------+
```

读图说明：探出父盒子的装饰部分会计入页面滚动宽度，手机上就会左右晃。

修复分两层。第一层是治标：给可能溢出的父级和 `body` 加保险。

```css
html, body {
  overflow-x: hidden;
}
.banner,
.cards {
  overflow: hidden;
  position: relative;
}
```

第二层是治本：小屏下把探出去的装饰收回来或藏掉，而不是靠裁剪硬盖。

```css
@media (max-width: 768px) {
  .dot2 {
    display: none;
  }
  .card-deco {
    right: 0;
    transform: none;
  }
  .cards img {
    max-width: 100%;
    height: auto;
  }
}
```

| 排查手段 | 发现的问题 | 修复归属 |
| :--- | :--- | :--- |
| 删段二分法定位晃动来源 | `dot2` 右探出约 `20px` | 小屏隐藏 + 父级裁剪 |
| 检查图片与卡片定宽 | 卡片 `width: 320px` 在 `320px` 屏下加边距溢出 | 改 `max-width: 100%` |
| 缩放复测 `375px / 360px` | 装饰条 `right: -30px` 探出 | 小屏归零 |

> **结论**：`overflow-x: hidden` 是保险丝，不是修法。真正要改的是溢出的那个元素本身，否则桌面端看着没事，换个更窄的机型又冒出来。

两个修完，我的验收动作是固定的，你们也照做：`1200px`、`768px`、`375px` 三个宽度各刷新一次，点一遍汉堡开合，再左右拖一下看有没有横向滚动。三遍全过，DesignCode 这个响应式项目才算收尾，后面我们进 `Bootstrap`，很多这类布局就不用手写了。

---

## 模块 07 全景总结与技术沉淀

本全书系统整合了 响应式布局原理与DesignCode项目实战 模块的 19 个核心专题（P164 ~ P182）。
建议读者在学完本章后，对照 `notes/` 目录下的思维导图树状笔记进行复盘与知识自测，巩固底层机理与工程实践能力。
