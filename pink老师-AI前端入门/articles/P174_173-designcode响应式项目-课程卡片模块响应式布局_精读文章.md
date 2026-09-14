# 课程卡片的响应式布局

模块会变阵了，卡片自己还不会变形，照样翻车。我见过最多的翻车就是这个：三列变一列了，卡片里的图还是按桌面比例硬撑，手机上被拉成面条。这一集我就盯着单张卡片，看它从外框到配图，怎么在三档屏幕里都站得住。

## 1. 卡片外框与配图的基准写法

先把桌面端的卡片定死，后面响应式只改它，不改结构：

```html
<article class="card">
  <div class="card-cover">
    <img src="./images/course-figma.jpg" alt="Figma 课程封面" />
    <span class="tag">Design</span>
  </div>
  <div class="card-body">
    <h3>Design for Developers</h3>
    <p>12 lessons · 4.5 hours</p>
  </div>
</article>
```

卡片语义我用 `article`，它本身就是一篇独立内容，比 `div` 准。封面和正文上下两截，标签角标用绝对定位压在图上：

```css
.card {
  background-color: #fff;
  border-radius: 20px;
  overflow: hidden;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.08);
}
.card-cover {
  position: relative;
  aspect-ratio: 16 / 9;
  overflow: hidden;
}
.card-cover img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}
.card-cover .tag {
  position: absolute;
  left: 16px;
  top: 16px;
  padding: 6px 12px;
  font-size: 12px;
  color: #fff;
  background-color: rgba(0, 0, 0, 0.55);
  border-radius: 999px;
}
```

`aspect-ratio: 16 / 9` 是整张卡片的命根子：不管卡片多宽，封面永远按宽高比（Aspect Ratio）等比缩放，图永远不变形。`object-fit: cover` 让图片像背景图一样铺满裁掉多余部分，`overflow: hidden` 配 `border-radius: 20px` 把图片四个角一起切圆，不然图片直角会从圆角卡片里戳出来。

```text
+---------------+
| 16 : 9 封面图 |
| [Design]      |
+---------------+
| 标题 + 课时   |
+---------------+
```

读图说明：上半是等比封面加左上角角标，下半是标题正文，圆角外框一次包住。

> **提示**：纵横比（`aspect-ratio`）写在容器上，不是写在 `img` 上。容器定比例，图片 `100%` 铺满，这才是正顺序。反过来图片定死高度，卡片一变宽就露白边。

## 2. 卡片的三档响应式改法

```css
@media (max-width: 1024px) {
  .card {
    border-radius: 16px;
  }
}
@media (max-width: 768px) {
  .card {
    display: flex;
    gap: 16px;
    padding: 12px;
    border-radius: 16px;
  }
  .card-cover {
    flex: 0 0 128px;
    aspect-ratio: 1 / 1;
  }
  .card-body h3 {
    font-size: 16px;
  }
}
```

平板档我只收圆角，`20px` 变 `16px`，小卡片配小圆角，视觉才协调。手机档动大手术：卡片从上下结构翻成左右结构，封面从 `16:9` 横图压成 `1:1` 方图，固定 `128px` 宽，右边文字区自动伸缩。你们想想，手机单列时如果还上下堆，一张卡片占半屏，三张就得滚半天，改成左图右文，一屏能看两三个，刷课效率高得多。

> **易错点**：`flex: 0 0 128px` 三个值分别是放大、缩小、基准，`0 0` 就是锁死 `128px` 不许伸缩。只写 `width: 128px` 在弹性容器里会被挤小，锁死必须用这三个零。

| 屏幕档位 | 卡片形态 | 封面处理 | 原因 |
| :------- | :------- | :------- | :--- |
| 桌面 | 上下堆叠 | `16 / 9` 横图 | 宽卡片横图最舒展 |
| 平板 | 上下堆叠 | `16 / 9` 不变 | 只收圆角间隙 |
| 手机 | 左图右文 | `1 / 1` 方图 `128px` | 省纵向空间、一屏多看 |

> **结论**：卡片响应式的精髓是封面比例跟着形态走：上下堆用宽图，左右排用方图。比例用 `aspect-ratio` 锁，裁剪用 `cover` 铺，手机翻成横排时记得把图片锁死宽度。卡片外框站住了，下一集钻到卡片肚子里收拾那些小盒子。
