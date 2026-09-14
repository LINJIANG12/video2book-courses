# 头部 Logo 模块制作

上一讲我们把 topbar 跑通了，这一讲我带你们做 header 里的第一个零件：logo。你们别觉得 logo 就是放张图，两分钟搞定。我跟你们说，logo 是全站 SEO 权重最高的一张图，写法不对，搜索引擎根本不知道你们是谁。

## 1 Logo 语义结构选择

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

## 2 背景图与文字隐藏处理

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

## 3 Logo 尺寸与对齐调试

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
