# 头部导航栏制作

logo 定住了，这一讲我们把 header 中间最长的一截——导航栏做出来。你们数一数，DesignCode 这一版有 5 个链接：Courses、Tutorials、Livestreams、Pricing、More，每一个的间距、字重、hover 状态都得抠到位。

## 1 导航列表结构搭建

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

## 2 横向布局与间距控制

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

## 3 链接状态与当前项样式

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
