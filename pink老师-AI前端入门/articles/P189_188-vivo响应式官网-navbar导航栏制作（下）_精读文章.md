# vivo 官网导航栏样式覆盖与交互细节打磨

上一节骨头搭完了，折叠能点、不断行，可你们也看到了：默认的黑底白字跟 vivo 那种干净的深色差口气，字挤、没 hover 效果，右边还没有搜索和人头图标。记住我这句话：结构用库的，颜值必须自己的。这一节我们专门贴肉，把导航调成官网的样子。

## 1.1 导航背景与高度覆盖

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

## 1.2 菜单悬停与激活状态处理

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

## 1.3 右侧图标组与整体联调

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
