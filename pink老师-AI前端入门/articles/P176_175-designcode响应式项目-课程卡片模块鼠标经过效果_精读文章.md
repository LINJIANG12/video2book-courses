# 课程卡片鼠标经过效果

我跟你们说，静态页面和有生命页面的分界线，就在鼠标放上去那 0.3 秒。你们前面已经把课程卡片的大盒子、小盒子都摆好了，位置是对的，可鼠标划过去一点反应都没有，那就还是 demo 的样子。这一讲我们就干一件事：把 `hover` 加上，让卡片会浮起来、图片会放大、箭头会冒出来。

我做这类效果有个习惯，先想清楚用户能感知到哪三层变化：整张卡片的位置和阴影、里面图片的缩放、还有文字区小元素的位移。你们记住这一条就够了，hover 永远是分层做的，一层一个 `transition`，别指望一个属性包打天下。

## 1 卡片整体上浮与阴影

最先要改的是 `.course-card` 本身。我让它默认就带一个很淡的阴影，鼠标经过时往上走 `8px`，阴影加深、扩散更大。这里容易搞错的是，`transition` 一定要写在卡片本身上，而不是写在 `:hover` 上。

写在 `:hover` 上会怎样？移入有动画，移出唰一下弹回去，特别生硬。你们一定要把过渡写在常态，我反复强调过，动画写常态、变化写 hover。

```css
.course-card {
  background-color: #fff;
  border-radius: 20px;
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.06);
  transition: transform 0.3s ease, box-shadow 0.3s ease;
}
.course-card:hover {
  transform: translateY(-8px);
  box-shadow: 0 20px 48px rgba(0, 0, 0, 0.14);
}
```

> **结论**：`transform: translateY(-8px)` 只做视觉位移，不占据文档流，所以不会把旁边的卡片挤开，这正是它比改 `margin` 好的地方。

为什么是 8px？不是 4px、不是 20px？我试过，4px 用户几乎感觉不到，20px 整排卡片跳得太厉害，8px 是刚好能感知又不乱的那个点。你们以后调 hover 位移，就在 6px 到 12px 之间试，别贪大。

## 2 图片缩放与溢出裁剪

第二层是图片。我希望鼠标放上去，图片轻轻放大到 1.08 倍，有一种镜头推近的感觉。这里有两个配套动作缺一不可：图片容器要 `overflow: hidden`，图片本身要 `transition: transform`。

```css
.course-card .pic {
  overflow: hidden;
  border-radius: 20px 20px 0 0;
}
.course-card .pic img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: transform 0.5s ease;
}
.course-card:hover .pic img {
  transform: scale(1.08);
}
```

> **易错点**：图片放大用 `scale`，时间我给了 `0.5s`，比卡片上浮的 `0.3s` 慢一拍。这样快慢错开，层次感一下就出来了。如果全写成一样的时长，画面会发闷。

你们自己试一下，把 `overflow: hidden` 去掉再 hover，图片会直接撑破圆角漫出来，整个卡片就露馅了。所以裁剪和缩放永远是一对，记住这一条就够了。

## 3 文字区箭头与信息显现

第三层藏在文字区。课程卡片右下角那个小箭头，默认是半透明、往左缩一点的，hover 时它滑出来、变实。我用的是 `opacity` 加 `translateX` 的组合。

```css
.course-card .more {
  opacity: 0.4;
  transform: translateX(-6px);
  transition: opacity 0.3s ease, transform 0.3s ease;
}
.course-card:hover .more {
  opacity: 1;
  transform: translateX(0);
}
```

这个细节很多人会漏掉：小元素位移的距离一定要小，6px 就够了。你给 20px，它就像弹射出去一样，特别廉价。

三层叠在一起，整个 hover 的结构是这样的：

```text
+----------------------------------+
|  .course-card                    |
|   translateY(-8px) + shadow加深  |
|  +------------------------------+|
|  | .pic (overflow:hidden)       ||
|  |  img scale(1.0 -> 1.08)      ||
|  +------------------------------+|
|  | .info                        ||
|  |  .more opacity .4 -> 1       ||
|  +------------------------------+|
+----------------------------------+
```

读图说明：外框是卡片整体位移，中框是图片裁剪区内的缩放，底框是文字区小元素的淡入滑出，三层各管各的过渡。

## 4 过渡时长与性能取舍

最后说个当年踩过的坑。我以前给所有 hover 都加 `transition: all 0.3s`，省事是省事，可浏览器要对每个属性都做插值，卡片一多，低端机直接掉帧。后来我就只写要动的两个属性，从来不写 `all`。

| 写法 | 效果 | 建议 |
| :--- | :--- | :--- |
| `transition: all 0.3s` | 全属性过渡，方便但费性能 | 不用 |
| `transition: transform 0.3s, box-shadow 0.3s` | 只过渡变化项，流畅 | 采用 |
| 常态写过渡，hover 写变化 | 进出都有动画 | 必须如此 |

> **提示**：`transform` 和 `opacity` 是合成器能直接处理的属性，不触发重排，所以 hover 动画优先用它俩。阴影变化会引起重绘，但卡片数量不大时完全可以接受。

这一讲你们亲手把 hover 加完，再把鼠标来回划几遍，感受一下 0.3 秒和 0.5 秒错开的节奏。下一讲我们进入故事卡片模块，那是整页最出彩的一屏。
