# 背景滤镜 backdrop-filter 与毛玻璃效果

上一节的 `filter` 是加工元素自己，这一节的 `backdrop-filter` 是加工元素背后的东西。只差两个字，效果天差地别：一个是给照片加滤镜，一个是隔着毛玻璃看风景。

你们手机控制中心那种半透明模糊面板、网站导航栏滚动时底下内容隐约透上来，都是它做的。我第一次亲手写出毛玻璃导航时，来回滚动页面玩了半天。

## 1. 背景滤镜的作用模型

> **定义**：背景滤镜（`backdrop-filter`）是对元素背后已经画好的内容再做像素加工，元素自己动不动，全看你给不给它加普通 `filter`。

```text
页面已渲染内容(图片/文字)
        |
        v
+-------------------+
| 半透明面板         |
| backdrop-filter:  |
|   blur(10px)      | --> 背后内容变模糊,面板文字保持清晰
+-------------------+
        |
        v
    显示到屏幕
```

读图说明：滤镜发生在面板这块区域，但加工对象是面板盖住的那片背景，面板自己的文字不受影响。

最小可用写法三件套缺一不可：

```css
.nav {
  /* 1. 必须半透明,全不透明就看不见背后了 */
  background: rgba(255, 255, 255, 0.3);
  /* 2. 背景滤镜:背后模糊 10px */
  backdrop-filter: blur(10px);
  -webkit-backdrop-filter: blur(10px);
}
```

> **易错点**：背景写成实色 `background: #fff`，背后全被盖住，`backdrop-filter` 写得再对也看不出任何变化。我排查毛玻璃失效，十次有九次是背景不透明。

| 特性 | `filter` | `backdrop-filter` |
| :--- | :--- | :--- |
| 加工对象 | 元素自身及子元素 | 元素背后的页面内容 |
| 面板自身文字 | 跟着变模糊变灰 | 保持清晰 |
| 是否需要半透明背景 | 不需要 | 需要，否则看不见效果 |
| 典型用途 | 图片置灰、悬停变彩 | 毛玻璃导航、弹窗、卡片 |

## 2. 毛玻璃导航栏制作

```html
<div class="page">
  <div class="hero"></div>
  <nav class="nav">毛玻璃导航栏</nav>
</div>
```

```css
* { margin: 0; padding: 0; box-sizing: border-box; }

.page {
  height: 200vh;
}

.hero {
  height: 400px;
  background: url("banner.jpg") center / cover no-repeat;
}

.nav {
  position: sticky;
  top: 0;
  height: 60px;
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 18px;
  color: #333;
  /* 半透明白底 + 背后模糊 */
  background: rgba(255, 255, 255, 0.4);
  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
  border-bottom: 1px solid rgba(255, 255, 255, 0.5);
}
```

页面往下滚，图片从导航底下经过时自动变模糊，导航上的字干干净净。`position: sticky` 让导航吸在顶部，`blur(12px)` 是我试下来最舒服的数值：`5px` 以下几乎看不出，`20px` 以上背后就糊成一片了。

> **提示**：苹果浏览器需要 `-webkit-` 前缀才认背景滤镜，生产环境两行都写上，先写带前缀的，再写标准的。

## 3. 常用背景滤镜函数

背景滤镜的函数和普通滤镜基本通用，最常用的就三个：

```css
/* 背后模糊:毛玻璃的本体 */
.glass { backdrop-filter: blur(10px); }
/* 背后提亮:暗图上托起浅色面板 */
.bright-panel { backdrop-filter: brightness(1.3); }
/* 背后加饱和:让透上来的颜色更浓 */
.vivid-panel { backdrop-filter: saturate(1.5); }
```

也可以空格连写叠效果：

```css
.card {
  background: rgba(255, 255, 255, 0.25);
  backdrop-filter: blur(8px) saturate(1.4);
  -webkit-backdrop-filter: blur(8px) saturate(1.4);
  border-radius: 16px;
  border: 1px solid rgba(255, 255, 255, 0.4);
}
```

这就是经典的毛玻璃卡片：压在一张风景图上，卡片里的字清晰，卡片外的风景透进来又被柔化。我做弹窗、登录框、图片上的说明条，基本都是这个配方，换张底图就是新皮肤。

背景滤镜是实时计算的，盖住的区域越大、页面同时启用的面板越多，开销越大。我的习惯是：导航、弹窗这种长期在线的用，列表里几十个卡片每个都加就免了，低端机会掉帧。
