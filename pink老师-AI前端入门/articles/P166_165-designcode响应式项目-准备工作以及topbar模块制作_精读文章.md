# DesignCode响应式项目准备与顶部通告栏

我一上来先不急着写头部，先把这个 DesignCode 项目的地基打牢。你们跟着我做过小兔鲜、做过优医问诊，应该有感觉了：凡是看起来复杂的页面，前面半小时都在搭架子，架子搭对了，后面每个模块都是填空。

这一讲我就干两件事：把项目文件和基础样式准备好，再把最顶上那条细细的 topbar 通告栏做出来。别小看这条小栏，它把后面整个响应式的思路都定调了。

## 1 项目文件结构与基础样式准备

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

## 2 版心容器与整体分层

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

## 3 顶部通告栏结构搭建

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

## 4 顶部通告栏样式与弹性布局

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
