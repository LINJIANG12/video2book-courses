# 图标分类行与搜索框的制作

课程标题下面空荡荡的不好看，真实的产品页会在这里放两样东西：一排分类图标帮我快速跳转，一条搜索框让我直接搜课。这一集我就把这两样加上去。你们注意，我做图标从来不是为了装饰，每一个图标背后都对应一个跳转，点上去必须有反应。

## 1. 图标分类行的结构与排列

图标行我用无序列表写，语义上它就是一组并列的导航：

```html
<ul class="cate">
  <li>
    <a href="#">
      <i class="iconfont icon-figma"></i>
      <span>Figma</span>
    </a>
  </li>
  <li>
    <a href="#">
      <i class="iconfont icon-swiftui"></i>
      <span>SwiftUI</span>
    </a>
  </li>
  <li>
    <a href="#">
      <i class="iconfont icon-react"></i>
      <span>React</span>
    </a>
  </li>
  <li>
    <a href="#">
      <i class="iconfont icon-blender"></i>
      <span>3D</span>
    </a>
  </li>
</ul>
```

为什么图标不用 `img` 而用字体图标（Iconfont）？四个图标四张图就是四次请求，字体图标一次请求全搞定，还能跟着字号（`font-size`）随意放大缩小不发虚。`i` 管图形、`span` 管文字，各干各的。

```text
+------+------+------+------+
|  O   |  O   |  O   |  O   |
|Figma |SwiftU|React | 3D   |
+------+------+------+------+
```

读图说明：四个小格横向均分，图标在上、文字在下，每格都是一个可点击的 `a`。

```css
.cate {
  display: flex;
  justify-content: center;
  gap: 48px;
  margin-bottom: 40px;
}
.cate a {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 12px;
  font-size: 14px;
  color: #333;
}
.cate .iconfont {
  width: 56px;
  height: 56px;
  line-height: 56px;
  text-align: center;
  font-size: 28px;
  color: #fff;
  background-color: #0a0a0a;
  border-radius: 50%;
}
```

`flex-direction: column` 让图标和文字上下叠，`align-items: center` 让两者左右居中，这两句配在一起就是图标导航的固定写法。圆形底我用 `border-radius: 50%` 一步到位，宽高必须相等，不等就是椭圆，你们量一下 `56px` 对 `56px`，差 `1px` 都不行。

> **易错点**：`line-height` 等于盒子高度只能让单行内容垂直居中，图标字体正好是单行，所以 `line-height: 56px` 能把它顶到正中间。换成多行文字这招立刻失效，别乱套用。

## 2. 搜索框的胶囊造型

```html
<div class="search">
  <i class="iconfont icon-search"></i>
  <input type="search" placeholder="Search courses, topics, mentors…" />
  <button>Search</button>
</div>
```

输入框类型我用 `search` 不用 `text`，移动端弹出的键盘会带“搜索”按钮，语义也更准。放大镜图标放在输入框左边，按钮放在右边，整条做成胶囊形：

```css
.search {
  display: flex;
  align-items: center;
  max-width: 560px;
  margin: 0 auto;
  padding: 6px 6px 6px 20px;
  background-color: #fff;
  border: 1px solid #e5e5e5;
  border-radius: 999px;
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.06);
}
.search input {
  flex: 1;
  border: 0;
  outline: 0;
  font-size: 15px;
  background-color: transparent;
}
.search button {
  padding: 12px 28px;
  font-size: 15px;
  color: #fff;
  background-color: #5e5ce6;
  border: 0;
  border-radius: 999px;
  cursor: pointer;
}
```

三处细节你们照抄：第一，`border-radius: 999px` 是胶囊的通用写法，半径给得足够大，两头自动全圆，不用算高度的一半是多少；第二，输入框的边框和轮廓线（`border: 0; outline: 0`）必须清掉，否则胶囊里面再套一个方框，丑到没眼看；第三，`flex: 1` 让输入框吃掉中间全部剩余宽度，按钮宽度固定，两边放大镜和按钮一挤，输入框永远是自适应的。

> **提示**：占位文本（Placeholder）`placeholder` 里的提示语要写具体场景，“搜课程、主题、导师”比光写“请输入”好得多。用户看到例子才知道这个框能搜什么。

| 部件     | 布局手段              | 关键取值                      |
| :------- | :-------------------- | :---------------------------- |
| `.cate`  | 横向弹性、居中散开    | `justify-content: center; gap: 48px` |
| 图标圆底 | 等宽高等行高加全圆角  | `56px; border-radius: 50%`    |
| `.search`| 胶囊弹性条            | `border-radius: 999px; flex: 1` 输入框 |

> **结论**：图标行负责分流，搜索框负责直达，两者都是课程模块的入口。样式上记住两组固定搭配：图标导航用纵向弹性加居中，胶囊搜索用横向弹性加 `flex: 1`。下一集我们让整个课程模块在不同屏幕下自动变阵。
