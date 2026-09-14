# 课程模块的版心骨架与标题区

导航和汉堡菜单调通之后，页面往下滚，第一屏正文就是课程模块。我习惯先把这一块的骨架立住：标题在上、卡片在下，中间留足呼吸感。你们跟着我把这一集做完，课程区的大架子就有了，后面几集往里面填图标、搜索框、卡片细节，都是顺手的事。

## 1. 课程模块的三段结构

我把课程模块拆成三段：外层版心、标题区、卡片容器。为什么非要套三层？外层管背景和上下留白，版心管宽度居中，卡片容器管排列，三件事搅在一起，后面做响应式会改到崩溃。

```text
+-------------------------------+
| section.courses               |
|  .w (版心, max-width: 1200px) |
|   .courses-hd (标题区)        |
|   .courses-bd (卡片容器)      |
+-------------------------------+
```

读图说明：`section` 撑满全宽给背景和留白，`.w` 收窄居中，标题和卡片都装在版心里。

```html
<section class="courses">
  <div class="w">
    <div class="courses-hd">
      <h2>Featured Courses</h2>
      <p>Learn the latest design tools and workflows</p>
    </div>
    <div class="courses-bd">
      <!-- 卡片后面几集逐个填 -->
    </div>
  </div>
</section>
```

标题用 `h2` 不是 `h3`，一页只有一个 `h1` 留给站点名，模块标题从 `h2` 往下排，这个层级关系别搞反了。

## 2. 版心与上下留白的取值

```css
.w {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 24px;
}
.courses {
  padding: 96px 0;
  background-color: #f5f7fa;
}
```

三个数字你们记住：`1200px` 是这套设计稿的版心，大屏两边留白、小屏自动收窄；左右各 `24px` 内边距是给手机留的安全距离，卡片再宽也不会贴到屏幕边；上下 `96px` 是模块之间的呼吸，课程区内容密，留白必须大一号，挤在一起会显得很廉价。

> **定义**：版心（Container）指页面内容的最大宽度区域，`max-width` 加 `margin: 0 auto` 让它在大屏居中、在小屏自动撑满。

> **易错点**：`width: 1200px` 和 `max-width: 1200px` 是两回事。前者写死宽度，手机上必出横向滚动条；后者是上限不封下限，小屏自动收缩。响应式项目里一律用 `max-width`。

## 3. 标题区的字号节奏

```css
.courses-hd {
  text-align: center;
  margin-bottom: 56px;
}
.courses-hd h2 {
  font-size: 40px;
  font-weight: 700;
  line-height: 1.2;
  color: #0a0a0a;
}
.courses-hd p {
  margin-top: 16px;
  font-size: 17px;
  line-height: 1.6;
  color: #666;
}
```

我为什么标题 `40px`、副标题 `17px`？差出一倍多，层级一眼就分出来了。你们千万别标题 `24px`、副标题 `20px`，差 `4px` 肉眼根本分不清主次。`margin-bottom: 56px` 是标题区和卡片区的距离，比段内间距大一档，这是排版的老规矩：组内紧、组间松。

```css
.courses-bd {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 32px;
}
```

卡片容器我直接用网格布局（Grid）三等分，间隙（Gap，`gap`）取 `32px`。有人问为什么不用弹性盒子（Flexbox）？三列等宽、行列都有缝，网格一行代码搞定，弹性盒子还要算百分比、处理换行，纯属给自己加戏。这一集先把三列铺出来，下一集开始做响应式，该变两列变两列。

> **提示**：`repeat(3, 1fr)` 等价于写三遍 `1fr`，`fr` 是网格专用的等分单位，会自动吃掉版心剩下的全部宽度，比百分比省心，不用再减间隙。

| 结构层   | 管的事         | 关键属性                          |
| :------- | :------------- | :-------------------------------- |
| `section.courses` | 背景、上下留白 | `padding: 96px 0` |
| `.w`     | 宽度、居中     | `max-width: 1200px; margin: 0 auto` |
| `.courses-hd` | 标题对齐、组间距 | `text-align: center; margin-bottom: 56px` |
| `.courses-bd` | 卡片排列       | `display: grid; gap: 32px` |

> **结论**：这一集不碰任何卡片细节，只干一件事：把课程模块的架子搭正。版心、标题、网格容器三层各司其职，后面填图标、搜索框、卡片，全部往 `.courses-bd` 里加就行，骨架不用再动。
