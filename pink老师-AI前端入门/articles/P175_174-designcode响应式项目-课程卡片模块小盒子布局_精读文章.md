# 课程卡片内部小盒子布局

外框会变形了，卡片肚子里那堆小东西还挤在一起：头像压着名字，时长顶着难度，价格和星级打架。这一集专门收拾 `.card-body` 里面的小盒子。你们记住我的口诀：小盒子一律横向弹性，左中右三段，两端顶头、中间自适应。

## 1. 正文区的小盒子拆分

一个卡片正文我拆三行：标题行、讲师行、数据行，每行都是一个横向小盒子：

```html
<div class="card-body">
  <h3>Design for Developers</h3>
  <div class="meta">
    <img class="avatar" src="./images/mentor.jpg" alt="导师头像" />
    <span class="name">Meng To</span>
    <span class="lessons">12 lessons</span>
  </div>
  <div class="foot">
    <span class="price">$49</span>
    <span class="rating">★ 4.9</span>
  </div>
</div>
```

```text
+----------------------+
| 标题 (单行省略)      |
+----------------------+
| (头像) 名字 ... 课时 |
+----------------------+
| $49           ★ 4.9  |
+----------------------+
```

读图说明：标题独占一行，讲师行左中右散开，底部价格和评分顶到两端。

```css
.card-body {
  padding: 20px;
}
.card-body h3 {
  font-size: 18px;
  line-height: 1.4;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
```

标题三件套 `nowrap + hidden + ellipsis` 是单行省略的固定写法，标题再长也只显示一行，多余的变省略号，卡片高度永远整齐。你们要是谁的卡片忽高忽低，八成是标题没做省略，长标题把盒子撑高了。

## 2. 讲师行与底部行的弹性写法

```css
.meta {
  display: flex;
  align-items: center;
  gap: 8px;
  margin-top: 12px;
  font-size: 13px;
  color: #666;
}
.meta .avatar {
  width: 28px;
  height: 28px;
  border-radius: 50%;
  object-fit: cover;
}
.meta .lessons {
  margin-left: auto;
}
.foot {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-top: 16px;
  padding-top: 16px;
  border-top: 1px solid #eee;
}
.foot .price {
  font-size: 18px;
  font-weight: 700;
  color: #0a0a0a;
}
.foot .rating {
  font-size: 13px;
  color: #f5a623;
}
```

讲师行里 `margin-left: auto` 是个巧劲：头像名字靠左，课时数被 `auto` 外边距一顶，自动弹到最右边，中间留白自动消化，比算宽度、比 `space-between` 都省事。头像 `28px` 等宽高加全圆，`object-fit: cover` 保证方图切圆不变形。

底部行用 `space-between` 把价格和评分顶到两端，顶上加一条 `1px` 分隔线，数据区和内容区一切为二。价格又黑又大，评分金色点缀，眼睛先看价格再看分，这个轻重别颠倒。

> **定义**：自动外边距（Auto Margin）在弹性布局里会吃掉全部剩余空间，`margin-left: auto` 等于说“把我推到最右边”，是单侧顶边的标准做法。

> **易错点**：头像只写 `border-radius: 50%` 不锁宽高，长方形图片切出来是椭圆。宽高相等加 `object-fit: cover` 三件套缺一不可。

| 小盒子 | 手法 | 关键属性 |
| :----- | :--- | :------- |
| 标题行 | 单行省略 | `white-space: nowrap; text-overflow: ellipsis` |
| 讲师行 | 左贴右弹 | `display: flex; margin-left: auto` |
| 底部行 | 两端对齐加分隔线 | `justify-content: space-between; border-top` |

> **结论**：卡片内部没有大布局，全是小盒子的横向弹性：标题防撑高用省略，讲师行用自动边距弹开两端，底部用两端对齐加线。到这里课程卡片从外框到内胆全部收工，下一集给卡片加上鼠标经过的浮起和阴影，整个课程模块才算真正活起来。
