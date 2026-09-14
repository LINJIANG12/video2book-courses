# vivo 官网导航栏骨架与折叠结构搭建

你们打开 vivo 官网看一眼，最顶上那条黑条：左边 logo，中间一排产品、商城、服务，右边搜索和人头图标，手机上缩成一个汉堡按钮，一点就展开。很多人觉得这个难，其实它就是 Bootstrap 的导航（导航，英文 `navbar`）组件换了层皮。这一节我们不管美化，先把骨头搭对，点开了、不断行、不错位，才有资格谈好看。

## 1.1 导航栏整体结构拆解

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

## 1.2 品牌标识与汉堡按钮配置

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

## 1.3 折叠菜单列表搭建

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
