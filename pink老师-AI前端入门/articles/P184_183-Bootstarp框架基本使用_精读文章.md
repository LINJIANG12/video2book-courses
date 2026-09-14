# Bootstrap 框架的容器与常用组件用法

上一讲把 `Bootstrap 5` 引进来，页面还是光秃秃的。很多人到这步就懵了：包是引了，可到底怎么“用”？我的体会是，`Bootstrap` 的用法就一句话：结构自己写，样式靠类名堆。你们别急着背所有类，先把最常用的容器、按钮、表格、表单摸熟，就能应付八成页面。

---

## 1. 版心容器与查文档习惯

所有 `Bootstrap` 页面都从容器开始。容器（`Container`）就是居中的版心，替我们管最大宽度与两侧留白。

```html
<div class="container" style="background:#eef3ff;">
  版心容器: 随断点定宽, 居中
</div>
<div class="container-fluid" style="background:#fff3e0;">
  流式容器: 永远 100% 宽
</div>
```

```text
容器宽度行为 (由小到大拖动)
container-fluid : |---------------- 100% ----------------|
container       :   |--- 540 ---|--- 720 ---|--- 960 ---|--- 1140 ---|
                  576px      768px       992px       1200px 断点定宽
```

读图说明：`container-fluid` 永远铺满，`container` 在每个断点取固定最大宽度并居中。

> **定义**：断点（`Breakpoint`）指 `Bootstrap` 预设的响应宽度分界，如 `576px`、`768px`、`992px`、`1200px`，容器与栅格都依附它们切换。

我建议你们从第一天就养成查文档的习惯：`Bootstrap` 官网左侧即是组件目录，点进去每个组件都有可复制的示例。不要背，记住“容器管宽、行列管分、组件管样”这个分工，缺什么查什么，比死记硬背快得多。

> **提示**：自己的覆盖样式永远写在 `bootstrap.min.css` 之后，否则会被框架样式压住，改了也没效果。

---

## 2. 按钮与表格组件

按钮是出场率最高的组件。`Bootstrap` 把按钮拆成两层类：`btn` 管基础形状，`btn-*` 管颜色语义。

```html
<button class="btn btn-primary">主要操作</button>
<button class="btn btn-success">提交成功</button>
<button class="btn btn-danger">删除</button>
<button class="btn btn-outline-primary">描边按钮</button>
<button class="btn btn-primary btn-lg">大号按钮</button>
```

表格同理，一行类名就有斑马纹与悬停高亮，手写至少十几行：

```html
<table class="table table-striped table-hover table-bordered">
  <thead>
    <tr>
      <th>姓名</th>
      <th>方向</th>
      <th>进度</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>小白</td>
      <td>前端</td>
      <td>HTML 已完结</td>
    </tr>
    <tr>
      <td>小黑</td>
      <td>前端</td>
      <td>CSS 进行中</td>
    </tr>
  </tbody>
</table>
```

| 类名 | 作用 | 手写替代成本 |
| :--- | :--- | :--- |
| `table-striped` | 隔行变色 | `nth-child` 手写 |
| `table-hover` | 悬停高亮行 | `tr:hover` 手写 |
| `table-bordered` | 全边框 | 逐格调边框 |
| `btn-lg / btn-sm` | 尺寸档 | 逐个调 padding |

> **易错点**：只写 `btn-primary` 不写 `btn`，按钮形状出不来。`btn` 是地基，颜色类是油漆，缺了地基，油漆无处刷。

---

## 3. 表单控件与焦点样式

表单是 `Bootstrap` 最省事的地方。输入框加 `form-control`，下拉加 `form-select`，复选加 `form-check-input`，焦点光晕、圆角、间距全齐了。

```html
<form>
  <div class="mb-3">
    <label class="form-label">邮箱</label>
    <input type="email" class="form-control" placeholder="name@example.com">
  </div>
  <div class="mb-3">
    <label class="form-label">方向</label>
    <select class="form-select">
      <option>前端</option>
      <option>后端</option>
      <option>全栈</option>
    </select>
  </div>
  <div class="form-check mb-3">
    <input class="form-check-input" type="checkbox" id="agree">
    <label class="form-check-label" for="agree">记住我</label>
  </div>
  <button class="btn btn-primary">登录</button>
</form>
```

`mb-3` 是间距工具类，`m` 表外边距，`b` 表下方，`3` 表档位。你们刚开始记不住档位没关系，记住规律：`m/p + t/b/s/e/x/y + 0~5`，调间距再也不用随手写内联样式了。

```text
表单控件类名分工
form-label  -> 标题文本样式
form-control -> 文本框/密码框/邮箱框统一样式
form-select  -> 下拉框样式
form-check-* -> 复选/单选的框与文组合
```

读图说明：每种控件各有专属类，对号入座即可，不要把 `form-control` 套到 `select` 上。

这一讲你们亲手敲一遍：一个版心里放一行按钮、一个条纹表格、一个登录表单，全部只用类名实现，不写一行组件 `CSS`。能做到这一点，`Bootstrap` 的基本使用就过关了，下一讲我们啃它最核心的栅格布局。
