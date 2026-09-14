# Bootstrap 栅格布局的 12 列与断点实践

前面容器、按钮、表格都会套了，可一到“左边三分、右边七分，手机上又要上下堆”就卡壳。你们是不是也觉得，响应式最烦的就是算百分比？我当年也是拿着计算器算 `33.333%` 的，直到吃透栅格：以后再也不用算了，`Bootstrap` 把一行切成 12 份，我们只管说“占几份”。

---

## 1. 栅格三件套与 12 等分原理

栅格（`Grid`）的三件套是固定的：`container` 做版心，`row` 做行，`col-*` 做列。缺谁都不行。

> **定义**：栅格列（`Column`）指 `row` 内按 12 等分划分的纵向槽位，类名中的数字即占几份，如 `col-4` 占 `4/12`。

```html
<div class="container">
  <div class="row">
    <div class="col-4" style="background:#dbe7ff;">占 4 份</div>
    <div class="col-8" style="background:#ffe9d6;">占 8 份</div>
  </div>
  <div class="row">
    <div class="col-3" style="background:#e7f8ec;">3 份</div>
    <div class="col-6" style="background:#f3e8ff;">6 份</div>
    <div class="col-3" style="background:#fff4c2;">3 份</div>
  </div>
</div>
```

```text
12 等分切分示意 (一行 = 12 份)
|--- col-4 (4) ---|-------- col-8 (8) --------|
|--3--|----- 6 -----|--3--|
 0   3   4         10  12  刻度
```

读图说明：同属一个 `row` 的列份数相加为 12 即铺满一行，超过 12 会自动换行。

> **结论**：一行内份数总和按 12 配，`4+8`、`3+6+3`、`6+6` 都是满行。一定要如此，绝不能那般：列必须包在 `row` 里，`row` 必须包在 `container` 里，跨级嵌套必出多余边距。

`row` 自带负边距吃掉列的内边距，列与列之间才有整齐的槽（`gutter`）。你们要是把 `col` 直接丢进 `container`，两侧对不齐，罪魁祸首就是少了这层 `row`。

---

## 2. 响应断点与列类组合

光会等分还不够，响应式的灵魂是断点：大屏三列，小屏一列。`Bootstrap` 的列类可以按断点叠加，写法是 `col-断点-份数`。

| 断点前缀 | 适用宽度 | 记忆法 |
| :--- | :--- | :--- |
| `col-` / `col-xs` 隐含 | `< 576px` 超小屏 | 手机竖屏默认 |
| `col-sm-*` | `>= 576px` | 小屏手机横屏 |
| `col-md-*` | `>= 768px` | 平板 |
| `col-lg-*` | `>= 992px` | 小桌面 |
| `col-xl-*` | `>= 1200px` | 大桌面 |
| `col-xxl-*` | `>= 1400px` | 超宽屏 |

实战里最常用的就是 `col-md-*`：手机上堆叠，平板以上分栏。

```html
<div class="container">
  <div class="row">
    <div class="col-12 col-md-4" style="background:#dbe7ff;">课程介绍</div>
    <div class="col-12 col-md-4" style="background:#e7f8ec;">课程目录</div>
    <div class="col-12 col-md-4" style="background:#ffe9d6;">学员评价</div>
  </div>
</div>
```

这行代码你们要读出两层意思：手机下每个 `col-12` 独占一行，上下堆；宽度一过 `768px`，`col-md-4` 接管，三列并排。为什么是 `768px`？不是 `700`、`800`？因为框架的 `md` 断点就定在 `768px`，跟着框架走，团队协作才对得上。

```text
同一行在不同宽度下的表现
>=768px : [ 4份 ][ 4份 ][ 4份 ]  一行并排
< 768px : [     12份     ]  堆叠
          [     12份     ]
          [     12份     ]
```

读图说明：断点是开关，宽度跨过 `768px` 即切换排列，小屏优先写堆叠，大屏用断点类覆盖。

> **易错点**：只写 `col-md-4` 不写 `col-12`，手机下三列会被硬挤在一行，字都叠在一起。移动优先：先写小屏的份数，再用断点类往大屏覆盖。

---

## 3. 列偏移嵌套与对齐微调

分栏会了，再学三招收尾活：偏移、嵌套、 Space 控制。

偏移（`Offset`）是让列往右空出几份，常用来做居中：

```html
<div class="row">
  <div class="col-md-6 offset-md-3" style="background:#dbe7ff;">
    占 6 份, 左空 3 份, 天然居中
  </div>
</div>
```

嵌套是列里再套行列，做卡片内部分栏：

```html
<div class="row">
  <div class="col-md-8">
    <div class="row">
      <div class="col-6">左小图</div>
      <div class="col-6">右文字</div>
    </div>
  </div>
  <div class="col-md-4">右侧边栏</div>
</div>
```

```css
/* 间距不够时的微调: 宁用 gutter 与工具类, 少写死 margin */
.tight-row {
  --bs-gutter-x: 12px;
}
```

> **提示**：调列间距优先改 `--bs-gutter-x` 或加 `g-*` 工具类，直接给 `col` 写死 `margin` 会撑破 12 等分，最后一列被挤到下一行，有苦说不出。

验收照我这个来：桌面端三列并排，平板两列加一列换行，手机单列堆叠，居中区块在各宽度下都不贴边。跑通这一套，你们以后做官网、后台、商城列表，布局半小时就能搭出骨架，剩下的就是填内容与调细节了。
