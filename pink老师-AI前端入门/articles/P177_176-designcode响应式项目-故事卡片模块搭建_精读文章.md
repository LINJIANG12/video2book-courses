# 故事卡片模块结构搭建

你们发现没有，前面课程卡片做得再漂亮，用户也只是觉得能用。真正让人记住这个页面的，是故事区那几张人物卡片。我跟你们讲，故事模块拼的不是特效，是结构：左边一张大卡讲主角，右边几张小卡讲配角，图片人物、引言文字、署名信息，一层都不能少。这一讲我们先把骨架搭结实，响应式下一讲再拆。

我上来先带你们看布局，很多同学一看到一左一右就想用浮动，那是老办法了。现在我直接用网格（Grid，栅格布局）起手，外层 `.stories` 定两列，左边大卡占一列，右边小卡列表占一列，思路一下就顺了。

## 1 故事区整体结构

整个模块我分成三段：标题区、大卡片区、小卡片列表区。标题区就是一句大标题加一句副文案，大卡是人物大图加引言，小卡是横向的小图文。你们先把语义想清楚再写标签， section 包 h2 加 p，卡片用 article，这都是有含义的。

```html
<section class="stories">
  <div class="stories-head">
    <h2>学员故事</h2>
    <p>看看他们是怎么从零基础走到独立做项目的</p>
  </div>
  <div class="stories-body">
    <article class="story-main">
      <div class="pic"><img src="./images/story-1.jpg" alt="学员作品展示" /></div>
      <div class="info">
        <p class="quote">从第一行标签到上线自己的响应式页面，我只用了两个月。</p>
        <div class="user"><img src="./images/avatar-1.jpg" alt="学员头像" /><span>林小白 · 前端学员</span></div>
      </div>
    </article>
    <div class="story-list">
      <article class="story-item">
        <div class="pic"><img src="./images/story-2.jpg" alt="学员作品展示" /></div>
        <div class="info">
          <p>以前最怕布局，现在 grid 和 flex 随手就来。</p>
          <span>阿哲 · 转行学员</span>
        </div>
      </article>
    </div>
  </div>
</section>
```

> **定义**：引言（quote）指卡片里那句第一人称的学员原话，它是故事卡的灵魂，字号和行高要单独给，不能跟普通正文混用。

你们写的时候一定注意，图片必须给 `alt`，这是习惯问题。今天少写一个 `alt` 没人罚你，哪天做企业站，缺 `alt` 就是硬伤。

## 2 大卡片样式

大卡我给了圆角 `24px`、白色底、深阴影，图片在上、文字在下。图片高度我定死 `320px`，配 `object-fit: cover`，这样不管原图是横是竖，裁出来都一样齐。

```css
.story-main {
  background-color: #fff;
  border-radius: 24px;
  overflow: hidden;
  box-shadow: 0 16px 40px rgba(0, 0, 0, 0.08);
}
.story-main .pic img {
  width: 100%;
  height: 320px;
  object-fit: cover;
}
.story-main .info {
  padding: 28px;
}
.story-main .quote {
  font-size: 20px;
  line-height: 1.6;
  color: #111827;
}
.story-main .user {
  display: flex;
  align-items: center;
  gap: 12px;
  margin-top: 20px;
}
.story-main .user img {
  width: 44px;
  height: 44px;
  border-radius: 50%;
}
```

头像为什么是正圆？`border-radius: 50%` 配等宽等高，那就是圆。你们要是宽高写得不一样，出来就是椭圆，特别容易被看出不专业。

整体层级关系是这样的：

```text
.stories
  |-- .stories-head (h2 + p)
  |-- .stories-body [grid: 2列]
        |-- .story-main (大卡: 大图 + 引言 + 署名)
        |-- .story-list (小卡列)
              |-- .story-item x N (小图 + 短引言)
```

读图说明：标题独占一行，主体两列中左列一张大卡，右列纵向排列多张小卡，大小卡共用圆角和阴影语言。

## 3 小卡片列表样式

小卡跟大卡是反过来的：横向放，左边小图、右边文字。图片我给了 `120px` 宽，固定不伸缩，文字区用 `flex: 1` 把剩下宽度吃满。小卡之间用 `gap: 20px` 拉开，比写 `margin` 省事得多。

```css
.story-item {
  display: flex;
  gap: 16px;
  align-items: center;
  background-color: #fff;
  border-radius: 18px;
  padding: 16px;
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.06);
}
.story-item .pic img {
  width: 120px;
  height: 96px;
  border-radius: 12px;
  object-fit: cover;
  flex-shrink: 0;
}
```

> **易错点**：小图一定要加 `flex-shrink: 0`。不加的话，屏幕稍窄一点，弹性盒（flex）就会先压缩图片，人物脸直接被压扁。我见过太多同学在这里翻车，记住这一条就够了。

这一讲我们把故事区的大卡小卡都立起来了，桌面端看已经像模像样。下一讲我们做它的响应式，到平板和手机上，大卡小卡该堆叠堆叠，该满宽满宽。
