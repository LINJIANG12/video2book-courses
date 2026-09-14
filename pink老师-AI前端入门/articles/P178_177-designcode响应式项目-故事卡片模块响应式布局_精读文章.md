# 故事卡片模块响应式布局

上一讲我们把故事卡片搭起来了，在宽屏上看挺漂亮。可你们把浏览器往窄里一拖就露馅了：大卡把小卡挤得没地方站，文字被压成一竖行。这就是响应式（responsive）要解决的事。我跟你们说，响应式不是重写一套页面，而是让同一套结构在不同宽度下换一种排法，这一讲我们就给故事区换排法。

我的思路很直接：宽屏两列、平板前后堆、手机全部单列。断点（breakpoint）我就用两个，`1024px` 和 `768px`，别一上来定五六个，把自己绕晕。记住这一条就够了，断点越少越好维护。

## 1 两列转单列的断点划分

桌面端 `.stories-body` 是 `grid` 两列，大卡小卡左右放。到 `1024px` 以下，我让它变成一列，大卡在上、小卡列表在下。到 `768px` 以下，小卡内部再从横排变竖排，图片顶满全宽。

```css
.stories-body {
  display: grid;
  grid-template-columns: 1.2fr 1fr;
  gap: 28px;
}
@media (max-width: 1024px) {
  .stories-body {
    grid-template-columns: 1fr;
  }
  .story-main .pic img {
    height: 260px;
  }
}
@media (max-width: 768px) {
  .story-item {
    flex-direction: column;
    align-items: stretch;
  }
  .story-item .pic img {
    width: 100%;
    height: 180px;
  }
}
```

> **提示**：媒体查询（media query）一定要按从宽到窄的顺序写，后写的窄断点才能覆盖宽断点。顺序写反了，手机样式会被平板样式盖掉，怎么调都不生效。

为什么大卡列给 `1.2fr` 而不是 `1fr`？因为主角就该比配角宽一点，视觉重心才稳。`fr`（fraction，剩余空间份数）这个单位你们现在应该很熟了，它分的是剩下的空间，不是整个宽度。

## 2 小卡片列表的换行处理

有个细节你们在平板上一定会遇到：转成单列后，小卡列表如果还是纵向三张，会把页面拉得很长。我在这里让小卡在平板上先变成两列网格，到手机再变回单列，这样长度和密度都刚刚好。

```css
@media (max-width: 1024px) {
  .story-list {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 20px;
  }
}
@media (max-width: 768px) {
  .story-list {
    grid-template-columns: 1fr;
  }
}
```

断点前后的对照关系是这样的：

```text
宽度 >1024px        1024px以下         768px以下
.stories-body       .stories-body      .stories-body
[大卡 | 小卡列]  -> [大卡]          -> [大卡]
                    [小卡 小卡]       [小卡]
                    [小卡 ... ]       [小卡]
                                      [小卡]
```

读图说明：从左到右宽度递减，主体从左右两列变为上下堆叠，小卡列表先是两列网格最后变为单列，图片由固定小图变为全宽大图。

| 宽度区间 | 大卡 | 小卡列表 | 小卡内部 |
| :--- | :--- | :--- | :--- |
| 大于 1024px | 左列 1.2fr | 右列 1fr 纵排 | 横向图文 |
| 768px 到 1024px | 独占一行 | 两列网格 | 横向图文 |
| 小于 768px | 独占一行 | 单列 | 竖向图文，图片全宽 |

## 3 图片高度与文字缩放

最后别忘了字。桌面端引言 `20px`，到手机上不减到 `17px`，一行装不下几个字，读起来特别累。图片高度也要跟着降，大卡从 `320px` 降到 `220px`，不然手机一屏全是图，看不到字。

```css
@media (max-width: 768px) {
  .story-main .info {
    padding: 20px;
  }
  .story-main .quote {
    font-size: 17px;
  }
  .story-main .pic img {
    height: 220px;
  }
}
```

> **易错点**：改响应式只改列数不改字号和图高，是新手最常见的半截子工程。布局换了，内容尺度也要换，这两个永远一起调。

你们调完一定要亲手从 1400px 一路拖到 360px 看三遍，重点看 1024 和 768 附近有没有横向滚动条。一旦出现横向滚动条，百分之百是有固定宽度没改掉，顺着找就行。下一讲我们玩 `clip-path`，给页面裁出波浪分界线。
