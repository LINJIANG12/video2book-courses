# 汉堡菜单按钮与移动端切换布局

导航藏起来了，空出来的位置得有人顶上，这个人就是汉堡菜单。你们在手机上天天点那个三横线，今天我们亲手把它做出来。

我跟你们说，汉堡按钮虽小，里面门道不少：三条线要等距，桌面端要彻底隐身，移动端要点得爽，还得给后面的下拉菜单留好钩子。

## 1 汉堡按钮结构与图标绘制

结构我用三根 `span`，干净又好控制：

```html
<button class="burger" aria-label="menu">
  <span></span>
  <span></span>
  <span></span>
</button>
```

为什么用 `button` 而不用 `div`？`button` 天生可聚焦、可回车触发，键盘用户也能操作。包上 `aria-label="menu"`，读屏软件会报“菜单按钮”，无障碍直接加分。用 `div` 模拟按钮的，还要自己补键盘事件，何苦呢。

三条线的画法，我不用图片，纯 CSS：

```css
.burger {
  display: none;
  flex-direction: column;
  justify-content: center;
  gap: 5px;
  width: 40px;
  height: 40px;
  background: none;
  border: 0;
  cursor: pointer;
}

.burger span {
  display: block;
  height: 2px;
  width: 22px;
  margin: 0 auto;
  background-color: var(--text-dark);
  border-radius: 2px;
}
```

默认 `display: none`，桌面端压根不存在它，这是关键。`flex-direction: column` 配 `gap: 5px`，三条线纵向等距，比一条线加上下边框的偏方稳多了，粗细也好调。

```text
.burger 40x40 点击区
+-------------+
|   =======   |  span 22x2
|   =======   |  gap 5px
|   =======   |  span 22x2
+-------------+
```

读图说明：可点击的是 40px 方块，线的视觉宽度只有 22px，线居中，四周留白都是点击区。记住这个关系：视觉可以小，点击区一定要大。

> **提示**：`span` 必须加 `display: block`，行内元素设宽高无效，三条线会挤成一团。这是每次汉堡做不出来的头号原因。

## 2 媒体查询切换显示

切换逻辑跟上一讲是镜像的：1024 以下汉堡现身，桌面端隐身。

```css
@media (max-width: 1024px) {
  .burger {
    display: flex;
  }
}
```

就这么一行，`none` 翻成 `flex`，三条线自动纵向排好。你们看，显示和隐藏一定要成对写：`.nav` 是桌面显示、移动隐藏，`.burger` 是桌面隐藏、移动显示，一藏一现，header 右边永远有人站岗，不会空。

| 元素 | 桌面 `>1024px` | 移动 `<=1024px` |
| :--- | :--- | :--- |
| `.nav` | `flex` 显示 | `none` 隐藏 |
| `.btn-login` | 显示 | `none` 隐藏 |
| `.burger` | `none` 隐藏 | `flex` 显示 |

> **结论**：响应式切换的本质就是两组 `display` 互斥。把这张表背下来，所有“桌面一套、移动一套”的组件都是这个套路。

header 右对齐我用 `margin-left: auto` 兜底。不管中间导航在不在，汉堡永远贴右，跟 logo 一左一右，72px 高度里垂直居中，拖到 360px 也不会乱跑。

## 3 移动端布局验证

我最后从 1200px 一路拖到 360px 验了三遍：桌面端 5 个链接加登录按钮整整齐齐，1024 一过瞬间切成 logo 加汉堡，无闪动、无错位、无横向滚动条。点汉堡虽然这一讲还没挂下拉面板，但 `button` 的按压态已经有了，`cursor: pointer` 一放上去就是小手，不会让用户觉得点不动。

> **易错点**：汉堡的 `width: 40px` 不要省。只画三条 22px 的线、不给父级尺寸的，点击区只剩几像素，手机上根本点不中。苹果的人机规范里可点击至少 `44px`，我们给 40px 是底线。

到这里，头部三件套齐了：logo 定左，导航居中，汉堡守右。下一讲我们离开头部，往下做课程模块，你们会发现，响应式真正的主战场其实在卡片列表。
