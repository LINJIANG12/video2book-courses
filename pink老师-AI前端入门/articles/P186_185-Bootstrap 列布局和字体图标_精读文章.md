# Bootstrap 列布局控制与字体图标应用

你们有没有遇到过这种尴尬：栅格明明是 12 等分，我只想放 3 个盒子，每个占 4 份，铺满倒是铺满了，可我想让它居中、想让它错开、想让它在手机上两列、电脑上四列，一写就乱。我跟你们讲，前面我们把 `container`、`row`、`col` 跑通了，只能算进了 Bootstrap 的门，这一节我们把列玩活，再把图标这件事一次解决。

## 1.1 十二列轨道的再理解

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

## 1.2 列偏移与空白控制

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

## 1.3 整行快速等分与列排序

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

## 1.4 列的对齐与槽宽调节

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

## 1.5 字体图标的引入与使用

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
