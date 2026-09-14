# Bootstrap 框架的定位与引用方法

手写完 DesignCode 那套响应式，我最大的感受就一个字：累。媒体查询、栅格、按钮、表单，每个都要自己从零调。你们是不是也在想：大厂天天做后台、官网，难道每次都手写一遍？当然不是，他们都站在框架肩膀上。这一讲我们就认识第一个重磅框架。

我刚入行那会儿也抵触框架，觉得用框架是“偷懒”。后来接了个三天上线的后台，才明白：框架不是偷懒，是把前人踩过的兼容坑一次性躲过去。`Bootstrap` 就是这类框架里资历最老、文档最全的一个。

---

## 1. 前端框架与 Bootstrap 的由来

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

## 2. 版本选择与两种引用方式

版本这里容易搞错。市面上还能搜到 `Bootstrap 3` 的老教程，里面用 `float` 做栅格，还依赖 `jQuery`。我们直接学 `Bootstrap 5`：栅格换成 `flex`，不再强依赖 `jQuery`，体积与写法都更现代。认准 `5.x`，别跟着老教程走偏了。

引用有两种，我两种都演示一遍，你们按场景选。

### 2.1 本地包引入

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

### 2.2 CDN 在线引用

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

## 3. 第一个 Bootstrap 页面模板

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
