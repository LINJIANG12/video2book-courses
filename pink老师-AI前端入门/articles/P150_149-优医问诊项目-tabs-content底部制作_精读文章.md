# Tabs Content 卡片底部的标签价格与问诊按钮

头部把“这是谁”讲清了，底部就要回答“贵不贵、怎么约”。你们看设计稿，卡片最底下永远是三样东西：擅长标签、问诊价格、问诊按钮。这一块我带你们一次做完，做完一张完整的医生卡片就闭环了，直接能拿去复用十几张。

## 1. 底部两行的结构划分

底部我拆成上下两行，上面放标签，下面放价格加按钮。为什么不全塞一行？标签数量不确定，一行会被价格挤乱，分两行最稳：

```text
+----------------------------------------+
| .card-foot                             |
|  .tags: [儿科] [发热] [咳嗽]            |
|  .price-row: ￥39起  图文问诊  [去问诊]  |
+----------------------------------------+
```

读图说明：标签行只管横向排列，价格行左边是价格文字、右边是实心按钮，两行上下叠放。

```html
<div class="card-foot">
  <div class="tags">
    <span>儿科</span>
    <span>发热</span>
    <span>咳嗽调理</span>
  </div>
  <div class="price-row">
    <p class="price"><em>￥39</em>起 · 图文问诊</p>
    <a class="consult" href="#">去问诊</a>
  </div>
</div>
```

价格数字用 `em` 包起来，语义上表示强调（Emphasis），样式上正好给它单独放大变色，一举两得。

## 2. 擅长标签的写法

```css
.card-foot .tags {
  display: flex;
  flex-wrap: wrap;
  gap: 6px;
  margin-top: 10px;
}
.card-foot .tags span {
  padding: 3px 8px;
  font-size: 11px;
  color: #16c2a3;
  background-color: #e8f8f5;
  border-radius: 4px;
}
```

三个点你们记一下。第一，浅底深字是标签最经典的配色，背景 `#e8f8f5`、文字 `#16c2a3`，同色系一浅一深，看着干净；第二，`flex-wrap: wrap` 允许标签换行，医生擅长十个八个也不会撑破卡片；第三，标签间距用间隙（Gap，`gap`）属性，比挨个写 `margin-right` 省事，最后一个还不会多出一截空隙。

> **提示**：`gap` 是弹性布局和网格布局（Grid）通用的间距属性，行列间距一次搞定。你们以前用 `margin` 凑间距，总要处理最后一个元素的尾巴，从现在起换 `gap`，代码少一半。

## 3. 价格行与问诊按钮的写法

```css
.card-foot .price-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-top: 10px;
}
.card-foot .price {
  font-size: 12px;
  color: #999;
}
.card-foot .price em {
  font-style: normal;
  font-size: 18px;
  font-weight: 700;
  color: #f66;
}
.card-foot .consult {
  padding: 7px 18px;
  font-size: 13px;
  color: #fff;
  background-color: #16c2a3;
  border-radius: 16px;
}
```

价格这里有个小反差：数字又大又红（`#f66`），“起、图文问诊”又小又灰，用户的眼睛会本能地先落在数字上，这就是价格行要的效果。记得把 `em` 默认的斜体用 `font-style: normal` 掰正，不然价格数字斜着很别扭。

按钮和上一讲的“关注”正好一对：关注是描边空心，问诊是实心填充，主次分明。你们以后做任何列表，最高频的操作永远给实心按钮，次要操作给描边按钮，这个轻重关系别颠倒了。

> **易错点**：`justify-content: space-between` 是把两端顶到头、中间留空，前提是父级是弹性盒子。有人把这个属性写给普通块级元素，写了跟没写一样，先检查 `display: flex` 在不在。

> **结论**：到这里，一张医生卡片的头、中、尾全齐了。你们复制三四个 `.card`，换换名字头像，就是一屏医生列表。下一讲我们把整个切换模块改成粘性定位（Sticky Position），滑动时选项卡吸在顶部，那个效果一加上，整个首页才算真正像个 App。
