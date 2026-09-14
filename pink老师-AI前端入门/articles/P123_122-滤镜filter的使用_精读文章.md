# 滤镜 filter 的函数与实战

手机修图软件里一键变灰、变亮、变模糊的那种滤镜，CSS 里一行就能实现，而且作用对象可以是图片、文字、整个盒子，甚至鼠标放上去还能平滑过渡。

我第一次知道 `filter` 的时候，第一反应是：以后做图片置灰、悬停变彩这种需求，再也不用找设计师切两张图了。

## 1. filter 的作用模型

> **定义**：滤镜（`filter`）是对元素自身渲染结果再做一次像素级加工，写在谁身上，谁整体的外观就变，子元素跟着一起变。

```text
原始元素渲染 --> filter 加工 --> 显示到页面
   彩色图    --> grayscale(1) --> 灰度图
   清晰图    --> blur(5px)    --> 模糊图
   普通盒子  --> drop-shadow  --> 带投影盒子
```

读图说明：滤镜是渲染管线末端的一道工序，元素先画好，再整体过一遍滤镜。

```css
img {
  width: 300px;
  /* 默认灰掉,悬停恢复彩色 */
  filter: grayscale(1);
  transition: filter 0.5s ease;
}
img:hover {
  filter: grayscale(0);
}
```

> **提示**：`filter` 会作用于整个元素，里面的文字、边框、背景一个都跑不掉。只想动背景而不动文字，后面背景滤镜那一节才是正解，这里别用错。

## 2. 常用滤镜函数

### 2.1 灰度与怀旧

```css
/* 灰度:0 原色,1 全灰 */
.gray { filter: grayscale(1); }
/* 怀旧:0 原色,1 全黄褐 */
.sepia { filter: sepia(0.8); }
```

`grayscale(1)` 是全灰，`grayscale(0.5)` 是半灰，取值就是灰的程度。`sepia` 是老照片那种黄褐色，两个经常叠着用，先灰后黄，旧照片味一下就出来了。

### 2.2 亮度、对比度与饱和度

```css
/* 亮度:1 原样,大于1变亮,小于1变暗 */
.bright { filter: brightness(1.5); }
/* 对比度:1 原样,越大黑白越分明 */
.contrast { filter: contrast(1.4); }
/* 饱和度:1 原样,0 全灰,大于1更浓艳 */
.saturate { filter: saturate(1.8); }
```

这三个的基准都是 `1`，`1` 就是不动。你们记一个口诀：大于 `1` 往浓里走，小于 `1` 往淡里走。调产品图时 `brightness(1.1) saturate(1.2)` 轻轻一叠，图片通透不少。

### 2.3 模糊与色相旋转

```css
/* 高斯模糊,半径越大越糊 */
.blur { filter: blur(5px); }
/* 色相旋转,0~360deg 转一圈回到原色 */
.hue { filter: hue-rotate(90deg); }
/* 反色:0 原色,1 全反 */
.invert { filter: invert(1); }
/* 透明度:和 opacity 类似,但可参与滤镜叠加 */
.fade { filter: opacity(0.5); }
```

`blur(5px)` 注意单位是长度不是百分比。`hue-rotate` 最好玩，`90deg` 直接把红色拧成绿色，做主题一键换色经常靠它。

多个函数可以空格连写，按从左到右依次加工：

```css
.poster {
  filter: contrast(1.2) saturate(1.4) brightness(1.05);
}
```

| 函数 | 取值 | 基准与含义 |
| :--- | :--- | :--- |
| `grayscale()` | `0~1` | `0` 彩色，`1` 全灰 |
| `sepia()` | `0~1` | `1` 全怀旧黄褐 |
| `brightness()` | `0~` | `1` 原样，大于变亮 |
| `contrast()` | `0~` | `1` 原样，大于更分明 |
| `saturate()` | `0~` | `1` 原样，`0` 等于灰度 |
| `blur()` | 长度 | 半径越大越模糊 |
| `hue-rotate()` | 角度 | 色轮旋转，`360deg` 回原色 |
| `invert()` | `0~1` | `1` 全反色 |
| `opacity()` | `0~1` | 元素整体透明度 |

## 3. 投影滤镜 drop-shadow

```css
/* 横偏移 竖偏移 模糊半径 颜色 */
.icon {
  filter: drop-shadow(4px 4px 6px rgba(0, 0, 0, 0.4));
}
```

你们肯定要问：这和 `box-shadow` 有什么区别？记住这个画面：`box-shadow` 给盒子加影子，永远是方的；`drop-shadow` 给不透明像素加影子，镂空图标、异形 PNG 的影子会跟着轮廓走。

```text
box-shadow:      +--------+
                 | 图片盒 |::::影子(方形)
                 +--------+

drop-shadow:        /\
                   /  \::::影子(跟着三角形轮廓)
                  /____\
```

读图说明：方形图两者差不多，异形图 `drop-shadow` 的影子才是贴合形状的。

我做异形按钮、镂空图标时，投影一律用 `drop-shadow`，方方正正的卡片才用 `box-shadow`。两者还能叠加，但别在同一个元素上反复套，性能会吃紧。

## 4. 悬停切换的完整写法

```css
.card img {
  width: 100%;
  filter: grayscale(1) brightness(0.9);
  transition: filter 0.4s ease;
}
.card:hover img {
  filter: grayscale(0) brightness(1);
}
```

默认灰一点暗一点，鼠标放上去恢复彩色，`transition` 把 `filter` 的变化抹平滑。你们去试，滤镜过渡天生就是平滑的，不需要逐帧写关键帧。注意别把过渡写在 `hover` 那一行，要写在常态上，去和回才都有动画。
