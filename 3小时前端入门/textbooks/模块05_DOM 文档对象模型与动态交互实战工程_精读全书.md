# 模块 05：DOM 文档对象模型与动态交互实战工程 合辑教材

> **所属课程**：3小时前端入门教程（HTML+CSS+JS）  
> **模块跨度**：P19 ~ P20（全模块共 2 讲系统重构）  
> **内容定位**：模块化系统学习教材，融合核心机制、架构全景、代码解析与思考自测。  
> **关联说明**：单集长文讲义同步保留于 `articles/` 目录供定向查阅。

---

## 模块导读与全景目录

1. JavaScript DOM
2. 练习-表格的增删改查

---

## JavaScript DOM
> 对应分集：P19 | 原始标题：《19. JavaScript DOM》

在掌握了 JavaScript 的基础语法与事件之后，如果不能直接读取并改变网页中的 HTML 标签与 CSS 样式，脚本就依然无法在页面上产生实质性的视觉变化。连接 JavaScript 语言与网页界面的核心枢纽，就是 **DOM**。

DOM 的全称为 **文档对象模型**（Document Object Model，DOM）。当浏览器加载一个 HTML 页面时，会首先将纯文本的 HTML 解析并转化为内存中的对象树。借助这套模型，JavaScript 获得了与 HTML 文档实时交互的编程接口（DOM API），能够动态地增删改查页面中的标签结构、文本内容和样式表现。

> **定义**：所谓“对象”（Object），其核心作用是建立数据（变量）与功能（函数）之间的组织联系。在 DOM 对象中，**属性**对应着变量（保存状态，如文本、尺寸、颜色），**方法**对应着函数（执行动作，如查询、监听、添加）。调用时，只需在属性或方法前加上对应的对象变量名即可。

---

### DOM 树状层级结构与节点分类

在浏览器视角下，整个 HTML 或 XML 文档被抽象表示为一棵层级严格的**文档树**（DOM Tree）。每个页面组件在这棵树上都有明确的亲属关系：父节点（Parent Node）、子节点（Child Node）以及同级兄弟节点（Sibling Node）。

```text
                        [ document ] (根文档节点)
                             |
                         <html> (根元素)
                             |
            +----------------+----------------+
            |                                 |
         <head>                            <body>
            |                                 |
         <title>                     +--------+--------+
            |                        |                 |
     "网页标题" (文本)             <h1>              <a> (href="#")
                                     |                 |
                              "主标题" (文本)     "超链接" (文本)
```
*上图展示了 DOM 树的标准层级结构：document 为根，向下延伸出元素节点，元素节点内部包含属性节点与叶子层的文本节点。*

在 DOM 规范中，文档内的一切皆可称之为**节点**（Node），主要划分为四大核心类别：
- **文档节点**（Document Node）：整个 DOM 树的唯一顶层入口，代指整个网页文档对象（即全局的 `document`）；
- **元素节点**（Element Node）：HTML 文档中所有的双标签与单标签实体，例如 `<html>`、`<body>`、`<div>`、`<h1>`、`<button>` 等；
- **属性节点**（Attribute Node）：嵌套在元素标签内部的键值修饰，例如 `class`、`id`、`href` 等；
- **文本节点**（Text Node）：标签对之间所包裹的具体文字信息，位于树结构的末端叶子节点。

---

### 元素节点的查询获取方法

要对页面上的某个元素进行修改，第一步必须在 JavaScript 中定位并获取该元素对应的 DOM 节点对象。DOM API 提供了多种精准与批量的查询方法。

| 查询方法 | 参数要求 | 返回类型 | 返回特征 |
| :--- | :--- | :--- | :--- |
| `document.getElementById()` | 元素的 ID 字符串 | 单个 Element 节点 | ID 在页面中具有唯一性，直接返回目标对象 |
| `document.getElementsByClassName()` | 样式类名字符串 | `HTMLCollection` 集合 | 类名通常可复用，返回类数组，需用索引提取 |
| `document.getElementsByTagName()` | HTML 标签名字符串 | `HTMLCollection` 集合 | 同名标签往往大量存在，返回类数组，需用索引提取 |
| `document.getElementsByName()` | 元素的 `name` 属性值 | `NodeList` 集合 | 多用于单选框等表单元素，按名称批量返回集合 |

> **提示**：从命名规则可以看出清晰的设计约定：方法名中带有单数 `Element` 的（如 `getElementById`），返回的是单一确切的节点对象；而带有复数 `Elements` 的（如 `getElementsByClassName`、`getElementsByTagName`），返回的一律是类数组集合，即使整个页面中符合条件的标签只有一个，也必须通过 `[0]` 索引下标进行解构访问。

#### 节点查询实战演示

准备如下 HTML 结构：

```html
<div id="box1">这是一个 ID 选择器标签</div>
<div class="box2">这是一个类选择器标签</div>
<div>普通的 DIV 标签</div>
```

在 `<script>` 中分别使用不同方法获取这三个节点：

```javascript
// 1. 通过 ID 获取单个节点
var element_id = document.getElementById('box1');
console.log(element_id);

// 2. 通过类名获取节点集合，并取第一个元素
var element_class = document.getElementsByClassName('box2')[0];
console.log(element_class);

// 3. 通过标签名获取页面中所有的 div，并按索引提取第三个普通 div
var element_tag = document.getElementsByTagName('div')[2];
console.log(element_tag);
```

---

### 元素内容与内联样式的动态更新

获取到元素节点后，即可对元素包含的数据与表现形式进行重写。

#### innerHTML 与 innerText 的渲染差异

修改元素文本或嵌套结构时，最常用的属性是 `innerHTML` 与 `innerText`：

```javascript
element_id.innerHTML = '修改 ID 选择器标签文本内容';
element_class.innerText = '修改后的类选择器文本';
```

> **易错点**：`innerHTML` 与 `innerText` 是 DOM 节点的**属性**，而不是方法。绝对不能写成函数调用式的 `element_id.innerHTML(...)`，必须使用等号 `=` 进行直接赋值。

二者在处理 HTML 标记字符串时存在根本性的语义差异：
- **`innerHTML`**：不仅输出文字，还会对字符串中包含的 HTML 标签标记进行**语法解析**，将标签转换为真实的子 DOM 节点并挂载渲染；
- **`innerText`**：对 HTML 标记**不作任何解析**，将整个字符串统一当作纯文本处理，尖括号及标签字符会被原封不动地显示在界面上。

```javascript
// innerHTML 解析 HTML 标签，界面渲染出可点击的超链接
element_id.innerHTML = '<a href="#">跳转链接</a>';

// innerText 忽略标签语义，界面直接显示出纯文本字符串 "<a href="#">跳转链接</a>"
element_class.innerText = '<a href="#">跳转链接</a>';
```

#### style 属性的动态赋值与命名规范

通过节点的 `style` 对象，可以直接修改元素的内联 CSS 样式：

```javascript
// 修改字体颜色
element_tag.style.color = 'red';

// 修改字体大小
element_tag.style.fontSize = '20px';
```

> **提示**：在原生 CSS 中使用短横线连接的属性名（如 `font-size`、`background-color`），在 JavaScript 中必须统一转换为**驼峰命名法**（CamelCase，如 `fontSize`、`backgroundColor`），这是因为短横线 `-` 在 JavaScript 中会被视作减号减法运算符。

---

### DOM 事件绑定的高级机制

除了在 HTML 标签内部书写属性绑定事件之外，更专业、更易维护的做法是在 JavaScript 内部直接通过 DOM 对象实施事件绑定。

#### DOM 对象属性赋值绑定事件

准备一个按钮标签：

```html
<button>按钮</button>
```

首先获取该按钮的 DOM 对象，然后直接为其 `onclick` 属性分配一个事件处理函数：

```javascript
var btn_element = document.getElementsByTagName('button')[0];

btn_element.onclick = function() {
  alert('DOM 属性按键触发');
};
```

> **注意**：此处赋值给 `onclick` 的是一个**匿名函数**（Anonymous Function，没有名称的函数）。因为该函数仅在该特定按钮被点击时被事件调度器触发，不需要在程序其他地方重复引用，直接书写匿名函数既精简又避免了污染作用域。

#### addEventListener 事件监听器

现代 Web 标准推荐使用 `addEventListener` 方法为 DOM 节点注册事件监听器。该方法接收两个核心参数：
1. **事件类型字符串**：指定要监听的事件名称，**不带 `on` 前缀**（例如直接写 `'click'`，而不是 `'onclick'`）；
2. **事件处理函数**：事件触发时执行的回调逻辑。

```text
node.addEventListener( 'click' , callbackFunction )
                          |             |
                 [ 事件类型无 on ]   [ 回调函数引用 ]
```
*上图展示了 addEventListener 的参数配置：事件名纯净解耦，函数以引用形式传递。*

##### 使用匿名函数注册监听器

```javascript
btn_element.addEventListener('click', function() {
  console.log('通过 addEventListener 触发按键');
});
```

##### 使用具名函数注册监听器

当事件处理逻辑较为庞大，或者希望该函数在其他地方复用时，可以抽离出具名函数进行绑定：

```javascript
function clickEvent() {
  console.log('通过 addEventListener 触发按键');
}

// 注意：传入的是函数名引用 clickEvent，绝不能在后面加圆括号 ()
btn_element.addEventListener('click', clickEvent);
```

> **易错点**：在将具名函数传递给 `addEventListener` 时，必须传递函数名本身 `clickEvent`。如果写成 `clickEvent()`，JavaScript 会在解析到该行时立即执行函数，并将函数的返回值（通常是 `undefined`）传给监听器，导致点击时无法正确触发。

---

### 常用 DOM 进阶操作概述

DOM API 除了基础的元素查询、内容修改与事件绑定外，还提供了一整套对文档树的动态管理能力：
- **节点增删**：通过 `document.createElement()` 创建新的元素节点，通过 `document.createTextNode()` 创建文本节点，并通过 `appendChild()`、`removeChild()` 将节点动态挂载到树上或从树中移除；
- **属性控制**：通过 `getAttribute()` 与 `setAttribute()` 动态读取或写入标签的任意自定义属性；
- **节点遍历**：通过 `parentNode`、`children`、`nextElementSibling` 等属性在树状层级中上下左右自由穿梭遍历。

掌握了 DOM 的基础查询、属性修改与事件绑定，前端开发便正式跨越了从“静态排版”到“动态应用”的鸿沟。

> 💡 **承前启后**：完成对「JavaScript DOM」的理解后，下一章我们将深入探讨「练习-表格的增删改查」，进一步完善知识图谱体系。

---

## 练习-表格的增删改查
> 对应分集：P20 | 原始标题：《20. 练习-表格的增删改查》

在前端基础阶段，掌握了 HTML 标签结构、CSS 盒子与排版属性，以及 JavaScript 的事件绑定和 DOM 操作之后，最有价值的学习方式就是通过一个经典的业务模型进行综合串联。动态表格的“增、删、改、查”就是最贴近实际开发、能够检验整体 DOM 操作熟练度的综合练习。

我们要实现的目标是一个带操作列的用户信息表格：页面顶部提供一个“新增数据”按钮，表格中展示“姓名”、“联系方式”和“操作”三列。每一行数据都配备“编辑”和“删除”按钮，能够动态向表格末尾追加行、删除指定行、以及通过弹窗修改对应行单元格的数据。

```text
+-------------------------------------------------------------+
|                      表格的增删改查                         |
|                                                             |
|  [ 新增数据 ]                                               |
|                                                             |
|  +------------+--------------------+---------------------+  |
|  |    姓名    |      联系方式      |        操作         |  |
|  +------------+--------------------+---------------------+  |
|  |   罗大富   |    13800000000     |  [编辑]   [删除]    |  |
|  +------------+--------------------+---------------------+  |
|  |   未命名   |     无联系方式     |  [编辑]   [删除]    |  |
|  +------------+--------------------+---------------------+  |
+-------------------------------------------------------------+
```
上图直观展示了整个综合案例的视觉布局结构与交互要素。

---

### 页面骨架与表格样式布局

在编写业务逻辑之前，必须先把语义化的 HTML 骨架和基础表格样式搭建妥当。合理的结构设计能大幅降低后续 DOM 节点的查找难度。

#### 表格结构搭建与初始数据标记

首先在 HTML 中放置一级标题，通过行内样式 `text-align: center;` 让文本在页面中水平居中。紧随其后放置一个触发新增操作的按钮 `<button>`，并在下方构建核心的 `<table>` 元素。

表格首行为表头 `<tr>`，内部使用加粗且默认居中的表头单元格 `<th>`，分别对应姓名、联系方式和操作三列。为了在页面初次加载时有参照物，我们先手工写入一行示例数据作为初始测试行：

```html
<h1 style="text-align: center;">表格的增删改查</h1>
<button onclick="addRow()">新增数据</button>
<br />
<table id="table">
    <tr>
        <th>姓名</th>
        <th>联系方式</th>
        <th>操作</th>
    </tr>
    <tr>
        <td>罗大富</td>
        <td>13800000000</td>
        <td>
            <button onclick="deleteRow(this)">删除</button>
            <button onclick="editRow(this)">编辑</button>
        </td>
    </tr>
</table>
```

> **提示**：为 `<table>` 元素显式标注 `id="table"` 是最稳健的工程做法。如果使用 `document.getElementsByTagName("table")` 获取，返回的是节点集合（数组），每次访问都需要通过索引下标 `[0]` 取值，稍有不慎就容易引发越界或找不到属性的错误；而通过 ID 则可以直接精确定位到唯一的 DOM 对象。

#### CSS 边框合并与单元格排版

原生 HTML 表格在未加修饰时非常粗糙，单元格彼此分离且带有空隙。我们需要在 `<head>` 的 `<style>` 标签中添加样式，让表格具有工业级的工整外观：

```css
table {
    width: 100%;
    border-collapse: collapse;
    margin-top: 20px;
}

th, td {
    border: 1px solid #ddd;
    text-align: center;
    padding: 8px;
}

th {
    background-color: #ddd;
}

button {
    margin-left: 5px;
}
```

这段样式包含了几个核心排版技巧：

| 样式选择器与属性 | 核心作用与原理解析 |
| :--- | :--- |
| `table { width: 100%; }` | 让表格自适应撑满整个浏览器窗口宽度，拉伸窗口时自适应排版 |
| `table { border-collapse: collapse; }` | 合并单元格默认分离的双重边框，消除间隙，使相邻边框合为一条单线 |
| `table { margin-top: 20px; }` | 为表格上方留出间距，避免与“新增数据”按钮挤在一起 |
| `th, td { border: 1px solid #ddd; }` | 设置灰白色实线细边框。`#ddd` 是十六进制颜色 `#dddddd` 的三位等价简写 |
| `th, td { text-align: center; padding: 8px; }` | 单元格文本统一水平居中，同时设置 8 像素内边距撑开单元格空间 |
| `th { background-color: #ddd; }` | 表头底色铺设浅灰色，与边框颜色一致，强化表头识别度 |
| `button { margin-left: 5px; }` | 操作列中并列的按钮默认紧挨，增加左外边距可使其产生呼吸间隙 |

---

### 脚本分离与 DOM 增删逻辑

随着交互逻辑复杂度的上升，将 JavaScript 逻辑直接混杂在 HTML 页面中会导致代码冗长、维护困难。最佳实践是在外部创建独立的 `.js` 脚本文件进行解耦管理。

我们在项目同级的 `js` 文件夹下新建 `table.js`，并在 HTML 文件末尾引入：

```html
<script src="js/table.js"></script>
```

> **提示**：在引入外部脚本后，第一步应当在脚本开头编写一句 `alert("table.js");` 或 `console.log("loaded");` 进行连通性测试。验证浏览器成功弹窗或打印日志后，再注释测试代码开始写业务，避免因路径错误排查半天。

#### 动态计算行索引与行节点追加

在表格中新增一行，底层对应的就是生成一个 `<tr>` 节点并挂载到 `<table>` 中。DOM 原生为表格对象封装了极其便利的方法：`table.insertRow(index)`。

```text
table.rows: [
  0: <tr> <th>姓名</th> <th>联系方式</th> <th>操作</th> </tr>
  1: <tr> <td>罗大富</td> ... </tr>
]  ==> table.rows.length = 2

table.insertRow(2)  ==> 新增在索引 2 的位置（恰好紧贴末尾追加）
```
上面的数据模型直观展现了表格行数组与动态插入索引的关系。

由于现有表格中有行索引 `0`（表头）和行索引 `1`（罗大富），行数 `table.rows.length` 的值为 `2`。我们要把新行追加在最后，传入的目标索引恰好就是 `length`。

```javascript
function addRow() {
    var table = document.getElementById("table");
    var length = table.rows.length;
    var newRow = table.insertRow(length);
}
```

> **易错点**：调用 DOM 原生方法时必须严格遵循驼峰命名。若不小心漏写字母写成了 `table.inserRow(length)`，浏览器控制台会报出 `table.inserRow is not a function` 的 TypeError 错误。

#### 单元格拆分创建与默认内容注入

空行 `newRow` 建立之后，不能直接往里面写纯文本，必须包含三个列单元格（`<td>`）。同样地，DOM 的行节点提供了 `row.insertCell(index)` 方法，用于在当前行的指定列索引处插入单元格：

```javascript
// 创建三列单元格节点
var nameCol = newRow.insertCell(0);
var phoneCol = newRow.insertCell(1);
var actionCol = newRow.insertCell(2);

// 为各列赋初始默认内容
nameCol.innerHTML = "未命名";
phoneCol.innerHTML = "无联系方式";
actionCol.innerHTML = '<button onclick="deleteRow(this)">删除</button><button onclick="editRow(this)">编辑</button>';
```

我们在新增数据时，姓名列默认赋给文本 `"未命名"`，联系方式赋给 `"无联系方式"`。最关键的是操作列：我们需要在单元格内部构建两个可点击的按钮，并通过字符串模板注入对应的 `onclick` 事件绑定。

#### 上下文传递与父节点回溯删除

删除操作的核心在于：**如何精准确定用户点的是哪一行？**

如果在绑定事件时不传参数，函数内部将无法得知是哪个具体的按钮触发了点击。在 HTML 的行内事件中，JavaScript 提供了一个特殊的上下文关键字：`this`。

```html
<button onclick="deleteRow(this)">删除</button>
```

当按钮被点击时，`this` 就会直接指向当前触发事件的那个 `<button>` DOM 对象本身。

```text
[button] (触发源: this)
   │  .parentNode
   ▼
  [td]   (操作单元格)
   │  .parentNode
   ▼
  [tr]   (整行节点 row)
   │  .parentNode
   ▼
[tbody / table] (表格主体/表格容器) ──.removeChild(row)──> 移除整行
```
通过节点的父子包含关系自底向上回溯，即可安全找到整行及其父容器。

DOM 中规定：**DOM 节点无法直接“自我销毁”，删除一个节点必须先找到其父节点，由父节点调用 `removeChild()` 将子节点移除**。

实现删除函数的完整流程如下：

```javascript
function deleteRow(button) {
    // 1. 回溯两次 parentNode 定位到所在行 <tr>
    var row = button.parentNode.parentNode;
    // 2. 由行节点的父节点调用 removeChild 删除该行
    row.parentNode.removeChild(row);
}
```

---

### 数据更新与边界交互优化

在实现了新增与删除之后，增删改查中最核心、交互链路最长的是“改”（编辑）功能。

#### 单元格集合索引与弹窗数据回填

点击编辑按钮时，同样传入 `this` 作为实参定位当前行。一旦拿到行对象 `row`，行内所有的单元格就可以通过行自带的属性集合 `row.cells` 按索引提取：

* `row.cells[0]`：第 0 列，即姓名单元格 `<td>`。
* `row.cells[1]`：第 1 列，即联系方式单元格 `<td>`。

获取到目标单元格后，借助浏览器内置的交互输入框 `prompt(message)` 引导用户输入新信息，并将返回值重新写入到单元格的 `innerHTML` 中：

```javascript
function editRow(button) {
    var row = button.parentNode.parentNode;
    var name = row.cells[0];
    var phone = row.cells[1];

    var inputName = prompt("请输入名字");
    var inputPhone = prompt("请输入联系方式");

    name.innerHTML = inputName;
    phone.innerHTML = inputPhone;
}
```

#### 空值拦截与取消输入保护

在测试编辑交互时，会发现一个明显的逻辑漏洞：如果用户弹窗后点击了“取消”或没有输入任何字符直接按确定，`prompt()` 会返回 `null` 或空字符串 `""`。按照上述直接赋值的代码，单元格原本的姓名和手机号就会立刻被覆盖成空白或 `"null"`。

为了提升交互体验与程序的鲁棒性，我们需要加入数据有效性校验：只有当用户真正输入了有效内容时，才执行赋值更新；若用户取消或输入全空格，则保留原内容不作变动：

```javascript
function editRow(button) {
    var row = button.parentNode.parentNode;
    var name = row.cells[0];
    var phone = row.cells[1];

    var inputName = prompt("请输入名字", name.innerHTML);
    var inputPhone = prompt("请输入联系方式", phone.innerHTML);

    // 仅在用户输入了非空字符串时更新数据
    if (inputName !== null && inputName.trim() !== "") {
        name.innerHTML = inputName;
    }
    if (inputPhone !== null && inputPhone.trim() !== "") {
        phone.innerHTML = inputPhone;
    }
}
```

> **提示**：为 `prompt(message, defaultValue)` 传入第二个参数（原单元格的内容 `name.innerHTML`），可以在弹窗出现时直接带出原有的数据，极大方便用户在原有文字的基础上进行微调修改，而不需要重新完整输入整串手机号。

---

### 完整业务代码整合

经过结构设计、样式调整、DOM 操作以及边界判断，我们最终整理出整套表格增删改查的完整实现。

#### 页面结构代码（index.html）

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <title>表格的增删改查练习</title>
    <style>
        table {
            width: 100%;
            border-collapse: collapse;
            margin-top: 20px;
        }
        th, td {
            border: 1px solid #ddd;
            text-align: center;
            padding: 8px;
        }
        th {
            background-color: #ddd;
        }
        button {
            margin-left: 5px;
        }
    </style>
</head>
<body>
    <h1 style="text-align: center;">表格的增删改查</h1>
    <button onclick="addRow()">新增数据</button>
    <br />
    <table id="table">
        <tr>
            <th>姓名</th>
            <th>联系方式</th>
            <th>操作</th>
        </tr>
        <tr>
            <td>罗大富</td>
            <td>13800000000</td>
            <td>
                <button onclick="deleteRow(this)">删除</button>
                <button onclick="editRow(this)">编辑</button>
            </td>
        </tr>
    </table>

    <script src="js/table.js"></script>
</body>
</html>
```

#### 业务交互逻辑（table.js）

```javascript
// 新增表格行数据
function addRow() {
    var table = document.getElementById("table");
    var length = table.rows.length;
    var newRow = table.insertRow(length);

    // 创建对应列单元格
    var nameCol = newRow.insertCell(0);
    var phoneCol = newRow.insertCell(1);
    var actionCol = newRow.insertCell(2);

    // 注入初始内容与带上下文引用的操作按钮
    nameCol.innerHTML = "未命名";
    phoneCol.innerHTML = "无联系方式";
    actionCol.innerHTML = '<button onclick="deleteRow(this)">删除</button><button onclick="editRow(this)">编辑</button>';
}

// 删除指定行数据
function deleteRow(button) {
    var row = button.parentNode.parentNode;
    row.parentNode.removeChild(row);
}

// 编辑指定行数据
function editRow(button) {
    var row = button.parentNode.parentNode;
    var name = row.cells[0];
    var phone = row.cells[1];

    var inputName = prompt("请输入名字", name.innerHTML);
    var inputPhone = prompt("请输入联系方式", phone.innerHTML);

    if (inputName !== null && inputName.trim() !== "") {
        name.innerHTML = inputName;
    }
    if (inputPhone !== null && inputPhone.trim() !== "") {
        phone.innerHTML = inputPhone;
    }
}
```

---

## 模块 05 全景总结与技术沉淀

本全书系统整合了 DOM 文档对象模型与动态交互实战工程 模块的 2 个核心专题（P19 ~ P20）。
建议读者在学完本章后，对照 `notes/` 目录下的思维导图树状笔记进行复盘与知识自测，巩固底层机理与工程实践能力。
