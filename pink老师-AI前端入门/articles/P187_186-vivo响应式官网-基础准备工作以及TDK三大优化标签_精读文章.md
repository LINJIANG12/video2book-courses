# vivo 响应式官网基础骨架与 TDK 标签配置

我跟你们讲，做官网最忌讳一上来就写导航、写轮播。我见过太多人，HTML 写了 200 行，忽然发现没引 Bootstrap 的 JS，折叠菜单点不动，又回去翻。听我的，先花 10 分钟把地基打牢：文件怎么放、库怎么引、标题和优化标签怎么写。地基稳了，后面几节我们搭导航、搭轮播都是往上堆积木。

## 1.1 项目目录与基础文件准备

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

## 1.2 视口标签与字符编码声明

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

## 1.3 TDK 三大优化标签写法

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

## 1.4 网站图标与版心注释分区

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
