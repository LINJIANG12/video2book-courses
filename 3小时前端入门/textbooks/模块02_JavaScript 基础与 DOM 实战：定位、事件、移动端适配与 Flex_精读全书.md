# 3小时前端入门教程（HTML+CSS+JS）·JavaScript 基础与 DOM 实战：定位、事件、移动端适配与 Flex（第 2 册 / 共 2 册）

> **所属课程**：3小时前端入门教程（HTML+CSS+JS）  
> **本册内容**：JavaScript 基础与 DOM 实战：定位、事件、移动端适配与 Flex  
> **覆盖范围**：P13-P19 ~ P20-P23（共 11 讲 / 2 章 / 97 分钟音频）  
> **分册说明**：全书按内容分 2 册；本册为第 2 册  
> **整编说明**：正文逐字保留各块模块长文，仅补导读、目录与章间过渡。

---

## 导读与全景目录

1. 定位与JavaScript基础：变量、函数、事件与DOM（P13-P19）
2. DOM实战练习、移动端适配与Flex布局（P20-P23）

---

## 定位与JavaScript基础：变量、函数、事件与DOM
> 对应块：BLK03 | 覆盖分集：P13-P19

在网页排版布局的演进中，浮动布局虽然灵活，但并不容易精确控制。定位布局则恰恰相反，它能够让我们精准地将页面中的某一个元素锁定在指定的位置上。但任何技术都要辩证地看待：定位元素在获得精准控制力的同时，也丧失了一定的流式灵活性，一旦面对空间尺寸与排列位置不确定的复杂版面，全盘使用定位反而会带来维护上的麻烦。从视觉样式的精准布局，再到通过脚本实现页面的动态交互，正是前端技术从静态走向动态的分水岭。

---

### CSS定位机制与脱离文档流特性

CSS 的定位机制主要由 `position` 属性以及四个方向偏移属性（`top`、`bottom`、`left`、`right`）共同控制。在默认的静态流之外，核心定位模式分为相对定位、绝对定位与固定定位三种。理解它们的核心分水岭，在于两点：第一是**定位参照基准究竟是谁**，第二是**该元素是否脱离了正常的文档流**。

```text
+-------------------------------------------------------------+
| 普通文档流 (Normal Flow)                                     |
| [ 元素 1 ]                                                  |
| [ 元素 2 (原本位置) ]  -->  [ 相对定位: 偏移但保留原占位 ]   |
| [ 元素 3 (保持原位不动) ]                                   |
+-------------------------------------------------------------+
| 绝对定位 (Absolute) / 固定定位 (Fixed)                       |
| [ 元素 1 ]                                                  |
| [ 元素 2 (脱离文档流) ] ===> 浮出平面，空间释放              |
| [ 元素 3 (自动上浮顶替原位置) ] <--------------------------+
+-------------------------------------------------------------+
```
> **图示说明**：相对定位在移动时原物理占位依然保留，后序元素不会移动；而绝对定位与固定定位一旦生效便脱离文档流，后序元素会自动上浮填补其留下的空隙。

#### 相对定位与文档流占位保留

相对定位是通过设置 `position: relative;` 实现的。它的核心定义是：**相对于该元素在文档流中的正常原始位置进行偏移**，并且**绝对不会脱离正常文档流**。

当我们仅仅给元素添加 `position: relative;` 时，页面在视觉上不会产生任何变化。只有配合方向偏移属性时，元素才会发生位移：
- `left`：表示距离该元素原本位置的最左侧向右偏移多少距离；
- `right`：表示距离原本位置的最右侧向左偏移多少距离；
- `top`：表示距离原本位置的最顶端向下偏移多少距离；
- `bottom`：表示距离原本位置的最底端向上偏移多少距离。

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <title>相对定位演示</title>
    <style>
        .box1 {
            height: 350px;
            background-color: aqua;
        }
        .box-normal {
            width: 100px;
            height: 100px;
            background-color: purple;
        }
        .box-relative {
            width: 100px;
            height: 100px;
            background-color: pink;
            /* 开启相对定位 */
            position: relative;
            left: 120px;
            top: 40px;
        }
    </style>
</head>
<body>
    <div class="box1">
        <div class="box-normal"></div>
        <div class="box-relative"></div>
        <div class="box-normal"></div>
    </div>
</body>
</html>
```

> **注意**：在上面的结构中，中间的粉色盒子向右偏移了 `120px`、向下偏移了 `40px`。但紧随其后的第三个紫色盒子（`box-normal`）依然牢牢停留在它原本的位置上，并未向上顶替。这就是因为相对定位的元素虽然在视觉上发生了位移，但它在文档流中的原始物理空间依然被保留着。通常在实际开发中，相对定位多用于对元素进行微调，或者作为绝对定位子元素的参考基准容器。

#### 绝对定位与定位父级脱离文档流

绝对定位通过设置 `position: absolute;` 触发。与相对定位截然不同，绝对定位的元素会**完全脱离正常的文档流**，不再占据原有的空间位置。

绝对定位的参考基准遵循“就近定位祖先”原则：
- 它会相对于与它最近的、已经开启了定位（即 `position` 属性值不为 `static`）的父级元素进行偏移；
- 如果它的所有父级及祖先元素均未设置定位，它将最终退化为相对于最外层的 HTML 根标签进行定位。

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <title>绝对定位演示</title>
    <style>
        .box2 {
            height: 350px;
            background-color: yellow;
            margin-bottom: 300px;
        }
        .box-normal {
            width: 100px;
            height: 100px;
            background-color: purple;
        }
        .box-absolute {
            width: 100px;
            height: 100px;
            background-color: yellowgreen;
            /* 开启绝对定位 */
            position: absolute;
            left: 120px;
        }
    </style>
</head>
<body>
    <div class="box2">
        <div class="box-normal"></div>
        <div class="box-absolute"></div>
        <div class="box-normal"></div>
    </div>
</body>
</html>
```

> **易错点**：当我们刚给 `box-absolute` 加上 `position: absolute;` 且未设置偏移时，会发现原本排在它后面的第三个紫色盒子“凭空消失”了。这并非元素被删除了，而是因为 `box-absolute` 脱离了文档流，不再占据原来的空间，底下的紫色盒子便自动上浮填补空白，导致两个盒子重叠在了一起，绝对定位的绿黄色盒子恰好把底层的紫色盒子完全遮挡住了。只要为绝对定位盒子加上 `left: 120px;` 将其移开，底层被遮盖的紫色元素便会重新显露出来。

#### 固定定位与浏览器视口固定

固定定位通过声明 `position: fixed;` 实现。它与绝对定位的共同点在于：**两者都完全脱离了正常文档流**。而它们最根本的区别在于**参照物不同**：
- 绝对定位参照的是最近的已定位父元素或根标签；
- 固定定位则是**直接相对于浏览器窗口（Viewport 视口）**进行定位。

由于它是相对于视口本身进行测算的，因此无论网页怎样向下或向右滚动，固定定位的元素始终保持在屏幕窗口的固定位置上，绝不随滚动条产生任何位移。

```css
.box-fixed {
    width: 100px;
    height: 100px;
    background-color: brown;
    /* 开启固定定位 */
    position: fixed;
    right: 0;
    top: 300px;
}
```

在页面最下方创建一个 `.box-fixed` 盒子，并赋予 `right: 0; top: 300px;` 后，这个棕色盒子会稳稳停留在浏览器屏幕右侧偏居中的位置。无论如何滚动鼠标滚轮，盒子都纹丝不动。这正是各大网站右侧固定侧边栏、客服悬浮挂件、返回顶部按钮以及顶部常驻导航栏的通用实现机制。

| 定位模式 | 声明语法 | 是否脱离文档流 | 定位参照对象 | 典型应用场景 |
| :--- | :--- | :--- | :--- | :--- |
| **相对定位** | `position: relative;` | 否（原物理占位保留） | 元素在文档流中的正常原位 | 元素位置局部微调、作为绝对定位的父级容器 |
| **绝对定位** | `position: absolute;` | 是（原有占位被释放） | 最近的已定位祖先元素，无则退化为 HTML 标签 | 复杂图文重叠、下拉菜单浮层、弹窗组件 |
| **固定定位** | `position: fixed;` | 是（原有占位被释放） | 浏览器视口窗口本身 | 顶部固定导航栏、屏幕侧边栏挂件、吸底操作栏 |

---

### JavaScript语言定位与加载机制

在掌握了 HTML 的骨架与 CSS 的样式定位后，前端技术进入了赋予页面动态交互能力的领域——JavaScript（简称 JS）。

#### 脚本语言定位与前后端运行环境

> **注意**：首先必须明确辟谣：**JavaScript 与 Java 没有任何关系**。它们是两门完全不同的编程语言，其命名历史仅属于当年的商业推广策略。

JavaScript 是一种**轻量级、解释型、面向对象的脚本语言**。它的设计初衷是为了在网页上实现动态视觉效果并增加用户与页面的实时交互。起初它仅仅用于在浏览器中处理用户的表单输入，但随着浏览器技术与 JavaScript 引擎的高速迭代，它已经成长为极其强大的通用编程语言。

与 HTML 和 CSS 最大的不同在于：JS 的加入让网页彻底告别了纯静态陈列，能够根据用户的点击、悬浮、滑动等操作，产生实时的逻辑响应与数据渲染。

在现代软件开发体系中，JavaScript 的应用领域覆盖了三大主流场景：
1. **客户端脚本**：直接嵌入 HTML 中并在用户的浏览器里解释执行，负责界面动效、事件捕获与交互逻辑；
2. **现代网页开发**：与 HTML、CSS 深度协同，通过异步通信和数据驱动，为用户提供富交互体验；
3. **后端服务端开发**：借助 **Node.js** 运行环境，JavaScript 打破了浏览器的边界，可以直接在服务端接收网络请求、读写数据库，完成完整的后端应用程序开发。

#### 内联嵌入与外部脚本引入机制

要在网页中使用 JavaScript，常用的导入方式有两种：内联式（Inline）与外部引入（External）。

```text
内联脚本:
HTML 文件 ---> <script> console.log("内部代码"); </script>

外联脚本:
HTML 文件 ---> <script src="js/myscript.js"></script> ===> 独立的 myscript.js 文件
```
> **图示说明**：内联脚本直接写在 HTML 的 script 标签内；外联脚本通过 script 标签的 src 相对路径属性链接外部独立脚本文件。

##### 内联脚本方式
与 CSS 使用 `<style>` 标签类似，JavaScript 代码必须书写在 `<script>` 标签之中。`<script>` 标签既可以放置在 `<head>` 头部内，也可以放置在 `<body>` 页面主体中。

```html
<script>
    console.log("你好，Hello，body 标签的内联样式");
</script>
```

##### 外部脚本引入方式
在项目工程中，推荐将逻辑代码抽离保存在独立的外部 `.js` 文件中（例如在项目根目录下建立 `js/` 目录并创建 `myscript.js` 文件）。在 HTML 中，我们通过 `<script>` 标签的 `src` 属性引入该外部文件：

```html
<script src="./js/myscript.js"></script>
```

其中 `./` 表示当前工作目录，通过相对路径层级定位至脚本文件。需要注意的是，当 `<script>` 标签指定了 `src` 属性后，该双标签内部不应再书写内联代码，否则内部代码将被浏览器忽略。

#### 浏览器开发者控制台与脚本调试

学习任何编程语言的第一课都是打印输出。在 JavaScript 中，最核心的输出与调试窗口不是页面本身，而是浏览器的**开发者工具控制台（Console）**。

在桌面浏览器（如 Google Chrome）中按下快捷键 **F12**（或右键点击页面检查），切换至 **Console** 标签页即可调出控制台面板。
- `console.log()`：用于向控制台打印调试日志、对象结构及计算结果。在主流代码编辑器中，通常可以输入 `log` 随后按下 `Tab` 键直接自动补全该方法。
- `alert()`：用于在浏览器窗口弹出一个模态警告对话框。它会阻断后续渲染直到用户点击确认。

```javascript
// 控制台输出
console.log("Hello, World!");

// 浏览器弹窗提示（必须使用英文标点符号包裹字符串）
alert("你好，内联样式弹窗");
```

##### 脚本解析执行顺序实验
浏览器在解析 HTML 文档时，是自上而下顺序读取并同步执行脚本的。如果在页面的不同位置引入脚本：

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <title>加载顺序测试</title>
    <!-- 头部内联脚本 -->
    <script>
        console.log("Hello，header 标签内联样式");
    </script>
    <!-- 头部外联脚本：内部执行 console.log("Hello，外联样式"); -->
    <script src="./js/myscript.js"></script>
</head>
<body>
    <h1>JavaScript 加载顺序</h1>
    <!-- 页面主体内联脚本 -->
    <script>
        console.log("Hello，body 标签的内联样式");
    </script>
</body>
</html>
```

打开 F12 控制台观察输出结果，其顺序严格遵循文档自上而下的扫描顺序：
1. `Hello，header 标签内联样式`
2. `Hello，外联样式`
3. `Hello，body 标签的内联样式`

---

### 变量声明与基础数据类型体系

#### 变量与常量声明关键字

在 JavaScript 中，声明一个变量或常量可以使用三个核心关键字：`var`、`let` 与 `const`。

```javascript
// var 关键字：声明传统变量（变量名后可不立即赋值）
var x;

// let 关键字：声明现代块级变量，并赋予初值
let y = 5;

// const 关键字：声明常量，必须在声明时初始化且后续不可更改
const π = 3.14;
```

`const` 代表常量（Constant），一旦赋值后在程序生命周期中一般不会轻易改变，适用于存储固定配置或数学常数（如圆周率 `3.14`）。如果后续尝试对其重新赋值，运行时将直接报错。

#### 作用域差异与变量提升规范

`var` 与 `let` 虽然都是用来声明变量的，但底层设计存在本质的区别：
- `var`（源自 variable 缩写）具有**函数作用域**（Function Scope），并且存在**变量提升**（Hoisting）现象，即变量可以在声明语句之前被访问（值为 undefined），极易引发状态混乱；
- `let` 具备**块级作用域**（Block Scope），仅在被 `{}` 包裹的代码块内部有效，且具备暂时性死区，不会发生非预期的变量提前暴露。

> **结论**：`let` 比 `var` 更安全、更灵活，它有效杜绝了变量污染与作用域逃逸。在现代 JavaScript 规范中，**统一推荐优先使用 `let` 与 `const`，全面避免使用 `var`**。

#### 基础数据类型与缺失值语义区分

打印上述声明的变量：

```javascript
console.log(x, y, π);
// 控制台输出：undefined 5 3.14
```

由此引出 JavaScript 中的核心数据类型：
1. **数值型（Number）**：在底层统一处理，既包含整型数值（如 `5`），也包含浮点型小数（如 `3.14`）；
2. **字符串型（String）**：用引号包裹的纯文本字符序列，例如 `let name = "如花";`；
3. **未定义型（Undefined）**：表示变量已被声明，但从未进行过初始化赋值的特殊状态；
4. **空值对象（Null）**：表示一个被明确赋值为空的特殊标记，例如 `let empty_value = null;`。

##### undefined 与 null 的深度辨析

初学者最容易将 `null` 与 `undefined` 混淆，虽然两者都用来表达“缺失的值”，但它们的语言学涵义和应用语境截然不同：

| 标识符 | 核心语义 | 产生机制 | 典型应用场景 |
| :--- | :--- | :--- | :--- |
| **`undefined`** | “未定义”状态 | 变量仅声明但未初始化赋值；访问对象中不存在的属性；函数缺少 `return` 语句时的默认返回值。 | 表示变量未来可能被计算出来并赋值，属于系统默认派发的初态。 |
| **`null`** | “空对象引用” | 由开发者**主动、明确赋值**给变量，表明该变量当前不指向任何有效对象或内存地址。 | 明确告知程序该对象资源目前为空，或用于手动解除对象的引用关系。 |

---

### 程序控制流语句与循环控制

控制语句赋予了程序根据条件产生分支判断与重复迭代的能力。在语法设计上，JavaScript 吸收了 C/Java 系列语言的通用语法，基本遵循“万变不离其宗”的编程共性。

#### 条件分支控制语句

条件语句允许程序根据逻辑表达式判断的真假结果（`true` 或 `false`），有选择地执行特定的代码块。最基础的形式由 `if`、`else if` 和 `else` 组合而成。
- `if (condition)`：当括号内的条件判定为真（true）时，执行大括号内的代码块；
- `else if (condition)`：当上一级条件判定为假时，顺延测试第二重或更多重备选条件（可以有任意多个）；
- `else`：当前面所有的条件判定全部为假时，执行此处的最终兜底代码块。在一个条件链中，`if` 和 `else` 只能有一个。

##### 案例 1：成年资格判定
```javascript
let age = 18;

if (age > 18) {
    console.log("你已经成年了");
} else {
    console.log("未成年");
}
```
当 `age = 18` 时，由于判断条件是严格大于（`>`），`18 > 18` 判定为 `false`，程序不会执行 if 分支，而是回退执行 else 分支输出 `"未成年"`；若将变量改为 `19`，则会触发 `"你已经成年了"`。

##### 案例 2：按时间段弹出问候窗
```javascript
let time = 12;

if (time < 12) {
    alert("上午好");
} else if (time >= 12 && time < 18) {
    alert("下午好");
} else {
    alert("晚上好");
}
```
> **提示**：书写前端代码时，引号、分号、小括号等语法标点符号**必须严格处于半角英文状态**。全角标点符号会导致脚本引擎抛出语法解析错误。当变量传入 `9` 时弹出“上午好”，传入 `12` 时命中第二个分支弹出“下午好”，传入 `22` 时命中兜底分支弹出“晚上好”。

#### 循环结构与迭代状态管理

循环语句用于重复执行一段逻辑，直到设定的终止条件不再满足。JavaScript 中常见的循环结构包括 `for` 循环、`while` 循环与 `do-while` 循环。

##### for 循环的三段式结构
`for` 循环是最经典的有限次数循环结构。它的括号内包含三个用分号分隔的标准表达式：
1. **初始化表达式**：定义循环计数变量并赋予起始值（例如 `let i = 1`）；
2. **循环条件判定**：每次执行前检查该条件，为真则进入循环体，为假则跳出循环（例如 `i <= 10`）；
3. **迭代步进器**：每次循环体执行完毕后触发的递增或递减运算（例如 `i++`，它是 `i = i + 1` 或 `i += 1` 的简写）。

```javascript
// 在控制台依次打印 1 到 10
for (let i = 1; i <= 10; i++) {
    console.log(i);
}
```

##### while 循环与无限死循环警示
`while` 循环仅在括号内条件为真时执行循环体。它内部没有内嵌的初始化与迭代器机制，循环变量必须在外层预先声明，且状态递增必须手动写在循环体内部。

```javascript
let count = 1;

while (count <= 10) {
    console.log(count);
    // 关键步骤：必须在循环体内部手动推进迭代变量
    count++;
}
```

> **易错点**：如果在 `while` 循环体内遗漏了 `count++` 这行步进代码，`count` 的值将永远停留在 `1`，使得 `count <= 10` 的条件永久为真。这将立即引发**无限死循环（Infinite Loop）**，导致浏览器主线程被彻底锁死，页面瞬间卡住无法响应，甚至引起计算机 CPU 满载风扇轰鸣。因此，面对有限次数的遍历，优先使用结构紧凑的 `for` 循环；面对未知终止次数或状态轮询时，使用 `while` 循环必须时刻警惕步进条件的闭环。

#### 循环跳转与流程中断指令

在循环执行过程中，若要临时变更执行流程，依赖于两个核心跳转关键字：`break` 与 `continue`。
- `break`：**立即强制终止并跳出整层循环**，循环体内后续的所有代码以及未来的所有迭代次数全部作废；
- `continue`：**仅跳过当前这一轮迭代中剩余的代码**，直接推进到下一次迭代的判定与执行阶段。

```javascript
// 循环控制关键字对比演示
for (let i = 0; i < 7; i++) {
    // 当 i 为 2 时，跳过当次剩余逻辑，直接进入下一轮
    if (i === 2) {
        continue;
    }
    // 当 i 为 4 时，彻底结束整个 for 循环
    if (i === 4) {
        break;
    }
    console.log(i);
}
```
运行上述脚本，控制台输出的序列为：
```text
0
1
3
```
分析可知：`i = 2` 时命中 `continue`，当次的 `console.log(2)` 被跳过；`i = 3` 正常输出；`i = 4` 时命中 `break`，循环彻底终止，因此后续的 `4`、`5`、`6` 均不会被执行。

---

### 函数定义与作用域边界

函数是将具备特定功能的代码组织在一起的复用单元。它能够接收外部输入的参数，在内部执行计算，并将最终结果返回。

#### 函数声明结构与调用方式

在 JavaScript 中，最标准的函数声明方式是使用 `function` 关键字：

```text
function 函数名(参数1, 参数2, ...) {
    // 函数体代码集合
    return 返回值;
}
```
- 函数名命名规范通常采用语义化的英文动词短语；
- 括号内为形参列表，多个参数以英文逗号分隔，无参数时小括号留空；
- 大括号内是具体的执行逻辑；
- `return` 用于将结果交还给调用方。若函数不需要回传结果，可省略 `return`。

```javascript
// 定义打招呼函数
function hello() {
    console.log("Hello, World!");
}

// 必须添加括号进行显式调用
hello();
```
> **提示**：只写出函数定义，计算机并不会执行其中的代码。函数必须通过 `函数名()` 加上圆括号的形式进行显式调用，函数体内部的代码才会被激活执行。

#### 返回值传递与参数列表处理

##### 带返回值的函数
如果函数通过 `return` 关键字返回了数据，直接调用它并不会在控制台输出任何内容，因为数据只是被丢回给了调用处。要查阅或使用这个数据，需要用变量接收，或作为参数直接传入输出函数：

```javascript
function hello_with_return() {
    return "Hello, World! 返回值";
}

// 方式一：用变量承接返回值后输出
let result = hello_with_return();
console.log(result);

// 方式二：直接嵌套打印调用结果
console.log(hello_with_return());
```

##### 带形参的动态函数
为了让函数具备通用性，可以通过参数列表接收外部动态传入的实参：

```javascript
// name 为形参（Parameter）
function hello_with_params(name) {
    console.log("Hello, " + name);
}

// 传入具体实参字符串
hello_with_params("如花");  // 输出：Hello, 如花
hello_with_params("Alice"); // 输出：Hello, Alice
```

#### 全局作用域与函数局部作用域

作用域（Scope）决定了变量的可访问区域与生命周期。在基础阶段，核心划分为两类：
- **全局作用域**：在所有函数体外部声明的变量，拥有全局作用域。它在整个页面生命周期中常驻，任何函数内部都可以自由读取它；
- **局部作用域（函数作用域）**：在函数大括号内部声明的变量，只在当前函数体内有效。一旦函数执行完毕，其内部变量即被封闭隔离，外部环境绝无可能直接访问。

```javascript
// 全局作用域中的变量
let global_var = "全局变量";

function local_scope_demo() {
    // 局部作用域中的变量
    let local_var = "局部变量";
    
    // 函数内部可以同时访问全局变量与自身局部变量
    console.log("函数内打印全局变量:", global_var);
    console.log("函数内打印局部变量:", local_var);
}

// 调用函数：内部打印全部成功
local_scope_demo();

// 函数外部调用测试
console.log("全局打印全局变量:", global_var); // 正常输出

// 尝试在外部读取函数内部的变量
console.log("全局打印局部变量:", local_var);   // 运行时报错！
```

> **注意**：当我们在全局环境下试图打印 `local_var` 时，控制台将直接抛出引用异常：`Uncaught ReferenceError: local_var is not defined`。这正是局部作用域的物理屏障特性——它避免了各个功能模块之间的变量相互污染与命名冲突。

---

### 页面事件驱动模型与事件绑定

事件驱动是现代前端交互的核心灵魂。所谓**事件（Event）**，是指在 HTML 文档或浏览器窗口中发生的**特定瞬间**（如用户点击了鼠标、按下了键盘键位、或者页面完成了加载）。事件在本质上就是**驱动特定函数执行的触发开关**。通过事件，HTML（骨架）、CSS（外观）与 JavaScript（行为逻辑）才真正交织联动起来。

#### 常用交互事件类型与触发时机

在 Web 标准中存在上百种事件类型，但在日常交互开发中，高频使用的核心事件包括以下几类：

| 事件属性名 | 对应原生事件 | 触发时机与语义 | 典型应用场景 |
| :--- | :--- | :--- | :--- |
| `onclick` | `click` | 鼠标指针左键点击某个元素瞬间 | 按钮提交、链接跳转、模态框呼出 |
| `onmouseover` | `mouseover` | 鼠标光标移入元素的外边界内 | 下拉菜单悬停展开、卡片高亮浮起 |
| `onmouseout` | `mouseout` | 鼠标光标从元素内移出边界外 | 菜单收起、恢复卡片原本阴影样式 |
| `onchange` | `change` | 表单输入控件的值发生改变且失去焦点 | 下拉框切换选中项、复选框状态改变 |
| `onselect` | `select` | 文本输入框中的文本内容被光标选中 | 快捷复制内容、选中文本高亮分析 |
| `onfocus` | `focus` | 文本框等输入控件**获得光标焦点** | 输入框聚焦边框变色、展示输入提示信息 |
| `onblur` | `blur` | 文本框等输入控件**失去光标焦点** | 表单实时离焦格式验证、密码强度检验 |

#### HTML行内属性事件绑定机制

将事件绑定到页面元素上有三种途径：
1. **HTML 标签属性绑定**（最直观的基础入门方式）；
2. **DOM 对象属性赋值绑定**；
3. **`addEventListener` 事件监听器注册**。

后两种方式需要深入结合 DOM 树进行操作，而第一种方式是直接在 HTML 标签内，将事件名称作为属性写入，属性的值就是当事件触发时所要调用的 JavaScript 函数表达式。

##### 按钮点击事件绑定
```html
<button onclick="click_event()">这是一个点击事件按钮</button>

<script>
    function click_event() {
        alert("点击事件触发了");
    }
</script>
```
点击该按钮，浏览器便会立刻执行 `click_event()` 并弹出提示窗。

##### 文本框多事件组合绑定
一个 HTML 标签上并不是只能绑定单一事件，它可以同时承载多个不同的事件属性。例如在一个输入框上同时侦听聚焦与失焦：

```html
<input type="text" onfocus="focus_event()" onblur="blur_event()">

<script>
    function focus_event() {
        console.log("获取焦点");
    }

    function blur_event() {
        console.log("失去焦点");
    }
</script>
```
打开 F12 控制台，当鼠标光标点击该文本框开始输入时，触发 `onfocus`，控制台打印 `"获取焦点"`；当点击页面其他空白区域导致光标移出文本框时，触发 `onblur`，控制台打印 `"失去焦点"`。

---

### 文档对象模型与DOM操作接口

当浏览器把一份 HTML 网页文件下载并解析后，会在内存中构建出一棵结构严密的**文档对象模型（Document Object Model，简称 DOM）**。DOM 为整个 HTML 页面提供了面向对象的编程接口（DOM API），让 JavaScript 获得了动态读取、增删、改写页面元素、属性与样式的完整权力。

```text
理解“对象”概念的极简公式：
对象 = 属性（存储状态的变量） + 方法（执行动作的函数）
在调用时：对象.属性名； 对象.方法名()
```

#### DOM文档树层次与节点类型划分

每一个 HTML 文档都可以被抽象并视作一棵倒置的**文档树（Document Tree）**。树中的一切组成部分，在 DOM 的世界里统统被定义为**节点（Node）**。

```text
                         [ Document 文档根节点 ]
                                   |
                                [ <html> ]
                               /          \
                       [ <head> ]        [ <body> ]
                       /        \          /      \
               [ <title> ]   [ <link> ]  [ <h1> ]  [ <div> ]
                   |                        |         |
               "文本节点"                 "文本节点"  [ 属性: class="box" ]
                                                      |
                                                   "文本节点"
```
> **图示说明**：DOM 结构呈现自顶向下的层级树，顶层为 Document 节点，所有标签、属性和内部文本均派生为各层级节点，彼此构成父子与同级兄弟关系。

在 DOM 规范中，节点主要细分为四种形态：
1. **文档节点（Document Node）**：整棵树的根基点，在 JS 中表现为全局内置的 `document` 对象；
2. **元素节点（Element Node）**：HTML 页面中的所有标签（如 `<html>`, `<body>`, `<div>`, `<h1>`, `<a>` 等）；
3. **属性节点（Attribute Node）**：写在标签内部的修饰属性（如 `class`, `id`, `href` 等）；
4. **文本节点（Text Node）**：双标签内部实际包裹呈现的文本字符内容。

#### 元素节点查询与集合索引提取

JavaScript 操控界面的第一步，必须是在 DOM 树中精准找到目标元素节点。`document` 提供了多套内置查询 API。

```html
<div id="box1">这是一个 id 选择器标签</div>
<div class="box2">这是一个类选择器标签</div>
<div>普通的 div 标签</div>
```

针对上述三个具有不同特征的标签，获取它们的语法规范有着关键区别：

##### document.getElementById()
通过元素的唯一 ID 名获取单个节点对象：
```javascript
let element_id = document.getElementById("box1");
console.log(element_id); // 打印输出单一的 div#box1 DOM 节点
```
因为 HTML 规范中 ID 是全局唯一的，所以该方法直接返回**单一的元素节点对象**。

##### document.getElementsByClassName()
通过 CSS 类名寻找元素：
```javascript
let element_class = document.getElementsByClassName("box2");
console.log(element_class); // 打印输出一个 HTMLCollection 集合对象
```
> **易错点**：注意方法名中的 `Elements` 为复数形式。即便整个页面中带有该类名的标签只有一个，它返回的也不是单一元素，而是一个类数组集合——**`HTMLCollection`**。如果直接拿这个集合去修改文本或样式，操作将完全失效。要精确提取当中的某一个节点，必须通过**下标索引值**（从 `0` 开始）来获取：
> ```javascript
> // 通过索引 [0] 获取集合中的第一个元素节点
> let target_class_element = element_class[0];
> ```

##### document.getElementsByTagName()
通过标签名称在全文范围内检索所有匹配的元素：
```javascript
let element_tags = document.getElementsByTagName("div");
console.log(element_tags); // 包含三个 div 节点的 HTMLCollection
```
页面中共有三个 `<div>`（`#box1`、`.box2`、普通的 `div`）。该方法会按照它们在 HTML 文档中出现的物理先后顺序依次存入集合。因此，若要获取第三个普通 `div`，必须通过索引值 `2` 进行定位：
```javascript
let third_div = element_tags[2];
```

#### 节点内容更新与HTML标签解析

获取到具体的元素节点后，可以通过属性修改其内部呈现的文本。这里涉及两个核心属性：`innerHTML` 与 `innerText`。

##### 属性与方法的语法辨析
> **易错点**：很多初学者在更新内容时容易习惯性地写成：`element_id.innerHTML("内容");`，控制台会立刻抛出 `Uncaught TypeError: element_id.innerHTML is not a function`。必须牢记：**`innerHTML` 和 `innerText` 都是 DOM 属性，绝不是函数方法！** 不能加上小括号当函数去调用，必须使用等号赋值语法：
> ```javascript
> element_id.innerHTML = "修改后的文本内容";
> ```

##### innerHTML 与 innerText 的本质差异

```javascript
let target1 = document.getElementById("box1");
let target2 = document.getElementsByClassName("box2")[0];

// innerHTML 解析富文本与 HTML 标签语义
target1.innerHTML = '<a href="#">跳转链接</a>';

// innerText 忽略标签语义，强制按纯文本渲染
target2.innerText = '<a href="#">跳转链接</a>';
```
在浏览器中渲染上述两行代码：
- `target1` 中的内容被浏览器成功解析为一个真正可被鼠标点击跳转的超链接 `<a>` 锚点；
- `target2` 页面上则原原本本、冷冰冰地打印出纯文本字样：`"<a href="#">跳转链接</a>"`。

| 内容属性 | 标签解析能力 | 处理机制与安全性 |
| :--- | :--- | :--- |
| **`innerHTML`** | **能够解析**字符串中的 HTML 标记与标签语义 | 将字符串解析为真正的 DOM 子元素树并完成渲染，适用于动态装配富文本与组件结构。 |
| **`innerText`** | **忽略并不解析** HTML 标记 | 将传入的一切内容无差别视为纯文本字符，自动忽略或转义 HTML 标记，安全性更高。 |

#### 节点行内样式动态修改

通过元素节点的 `style` 对象属性，可以动态修改 CSS 样式。通过这种方式赋予的样式属于**行内样式（Inline Style）**，具有极高的渲染优先级。

```javascript
let third_div = document.getElementsByTagName("div")[2];

// 修改字体颜色
third_div.style.color = "red";

// 修改字体尺寸（属性名采用小驼峰写法，对应 CSS 中的 font-size）
third_div.style.fontSize = "20px";
```
代码执行后，第三个普通 `div` 内的字体将立即转变为红色，并且字号放大至 `20px`。

#### 基于DOM属性与监听器的事件绑定

在学习事件时，除了在 HTML 标签中内联写入属性外，生产环境中更为推荐的做法是将 HTML 骨架与 JS 行为代码解耦，完全在 JS 脚本内部基于 DOM 节点对象完成事件绑定。

假设页面存在一个纯净的按钮标签：
```html
<button>触发按钮</button>
```
首先通过 DOM 接口提取该节点对象：
```javascript
let button_element = document.getElementsByTagName("button")[0];
```

##### 方式一：基于 DOM 属性赋值绑定
直接为元素节点的事件属性（如 `onclick`）赋予一个函数。因为这个函数只需要在此处被该按钮调用，通常直接采用**匿名函数（Anonymous Function）**编写：

```javascript
button_element.onclick = function() {
    alert("DOM 属性按键触发");
};
```
点击该按钮，便会触发匿名函数体内包裹的弹窗逻辑。

##### 方式二：使用 addEventListener 事件监听器绑定
现代标准推荐使用 `addEventListener()` 方法。该方法接收两个核心参数：
1. **事件类型字符串**：指明要监听的具体事件（注意：**此处一律去掉 `on` 前缀**，例如点击事件写为 `'click'`，而不是 `'onclick'`）；
2. **回调处理函数**：当事件发生时交由浏览器调用的程序逻辑。

```javascript
// 使用匿名函数作为监听回调
button_element.addEventListener("click", function() {
    alert("通过 addEventListener 触发按键");
});

// 亦可使用外部预先声明好的命名函数作为监听回调
function click_event() {
    alert("通过外部命名函数触发");
}
button_element.addEventListener("click", click_event);
```
> **注意**：当把一个外部命名函数作为 `addEventListener` 的第二个参数传递时，只写函数名 `click_event` 即可，**绝对不能在后面加上小括号 `()`**。加了括号代表“立即执行该函数并把返回值传过去”，而不加括号才是将该函数本身的引用挂载为监听回调。

除了元素获取、文本改写、样式更替与事件绑定之外，DOM API 还涵盖了节点的动态创建（如创建元素节点、属性节点、文本节点）、子节点的增删插入（如 `appendChild`、`removeChild`）以及各类属性值的读取与设置。掌握了上述核心链路，便构建起了从静态排版到底层交互的完整技术基石。

> **承前启后**：上一节讲完「定位与JavaScript基础：变量、函数、事件与DOM」，下一节接着讲「DOM实战练习、移动端适配与Flex布局」。

---

## DOM实战练习、移动端适配与Flex布局
> 对应块：BLK04 | 覆盖分集：P20-P23

在掌握了 HTML 文本与表单标签、CSS 盒子模型与浮动定位，以及 JavaScript 变量、函数、事件和 DOM 节点操作的基本概念后，我们不再停留于孤立的语法小例子，而是进入一个真正具有综合性的前端实践阶段。通过动手构建一个完整的「表格数据的增删改查」案例，我们将 DOM 树的层级寻址、节点动态生成与挂载、销毁逻辑，以及事件触发中的执行上下文指针完整串联起来。

在此基础之上，面对当下多元化的终端设备生态，我们将视野从传统的 PC 桌面视窗拓展至移动端。通过拆解视口（Viewport）元标签的控制机制，剖析绝对单位与相对长度单位 `rem` 的本质差异，并借助 JavaScript 动态计算根元素字体大小建立起等比缩放响应式体系。最后，我们将全面进阶至现代 CSS 布局的核心支柱——Flex 弹性盒布局（Flexible Box Layout），从主轴与交叉轴的双轴坐标模型，逐一推导容器与项目的核心排版属性，并梳理从前端基础到 Vue.js、微信小程序及多端跨平台开发的进阶学习路线。

---

### DOM动态表格增删改查实战

很多人一听到“增删改查”（Create, Read, Update, Delete，简称 CRUD）就潜意识地觉得这一定是后端数据库的专属概念，其实在前端界面层，DOM 操作的本质就是对页面节点树的增删改查。我们先通过纯 HTML 和 CSS 搭建一个美观稳固的表格框架，随后通过外部 JavaScript 脚本分别实现动态新增数据行、整行删除以及行内数据编辑。

#### 表格骨架构建与CSS边框合并

我们首先从页面的结构和样式着手。构建一个包含姓名、联系方式与操作控制项的表格：

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <title>表格的增删改查</title>
    <style>
        /* 表格样式美化 */
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
    <button onclick="add_row()">新增数据</button>
    <table id="table">
        <tr>
            <th>姓名</th>
            <th>联系方式</th>
            <th>操作</th>
        </tr>
        <tr>
            <td>罗大富</td>
            <td>13800138000</td>
            <td>
                <button onclick="edit_row(this)">编辑</button>
                <button onclick="delete_row(this)">删除</button>
            </td>
        </tr>
    </table>
    <!-- 引入外部 JavaScript 业务脚本 -->
    <script src="./js/table.js"></script>
</body>
</html>
```

在这套基础骨架与样式的构建过程中，涉及四个关键排版与样式的技术细节：

**响应式宽度设定**：我们没有给 `<table>` 设置死板的像素宽度（如 `width: 800px;`），而是声明了 `width: 100%;`。这意味着表格将自动填满其父级容器（此处即浏览器窗口视口）的全部宽度。不论怎样拖拽缩放浏览器窗口，表格都能以 100% 的比例自适应横向展开。

**十六进制颜色的三位缩写规范**：在设置边框颜色时，选用浅灰色 `#ddd`。在 CSS 十六进制颜色表示法中，完整的六位颜色是 `#dddddd`。因为其第 1-2 位（红色通道）、第 3-4 位（绿色通道）和第 5-6 位（蓝色通道）的字符完全一致，根据 CSS 规范可以省略简写为三位 `#ddd`。这种写法在工业界样式表中极为普遍。

**单元格独立边框与边框合并**：在 HTML 默认渲染引擎中，表格的各个单元格（`<th>` 和 `<td>`）是作为独立的矩形框进行渲染的。如果不做额外处理，单元格与单元格之间的边框会彼此分离，产生明显的双重边框间距，影响视觉美观。

> **注意**：解决双边框问题的标准属性是为 `table` 添加 `border-collapse: collapse;`。`collapse` 的含义即为“折叠/合并”，它会命令浏览器将相邻单元格重合的边框合并为一条单线边框。

如果在日常开发中突然遗忘了某个生僻属性名，完全不需要死记硬背。直接在搜索引擎中输入“如何让表格中的边框合并”，检索结果会直接呈现 `border-collapse: collapse;` 及其用法示例。在现代前端开发中，面对浩如烟海的 CSS 属性，掌握检索关键词并快速转化为生产代码，是一项至关重要的核心素养。

**内外边距协调与视觉排版**：表头 `<th>` 默认具有文本加粗和居中特性，而普通数据单元格 `<td>` 则默认左对齐。为了让表格视觉整齐，我们给 `td` 显式设置了 `text-align: center;`；同时添加 `padding: 8px;`，撑开文字与边框的上下左右间隙；给 `table` 设置 `margin-top: 20px;`，拉开上方操作按钮与表格之间的垂直距离；给操作列中的按钮追加 `margin-left: 5px;`，避免编辑和删除两个按钮紧贴在一起。

#### 外部脚本解耦与文件挂载验证

随着前端页面的业务逻辑逐步丰富，如果将全部 JavaScript 脚本都堆砌在 HTML 文档的 `<script>` 标签内，会导致代码文件冗长杂乱、结构层与行为层高度耦合。因此，工程化的标准做法是将 JavaScript 单独剥离到独立的外部脚本文件中。

我们在项目工作区内建立名为 `js` 的子目录，在其中创建业务脚本文件 `table.js`，并在 HTML 页面中通过相对路径完成引入：

```html
<script src="./js/table.js"></script>
```

> **提示**：在引入外部脚本文件之后，先不要急着写大段复杂的业务逻辑。为了验证路径引用是否准确无误，可以在 `table.js` 的第一行写入一行简单的代码：
> ```javascript
> alert("table.js 引入成功");
> ```
> 保存并在浏览器中刷新页面。如果成功弹出了提示对话框，便证实了该外部文件已被当前 HTML 正确加载；此时即可将测试用的 `alert` 移除，继续编写核心逻辑。

在为三个按钮绑定事件之前，我们对增、删、改三类操作的技术复杂度做一个理性评估：
- **新增数据（Add）**：向表格末尾追加一个全新的行节点，并在行内生成三个单元格节点；
- **删除数据（Delete）**：定位到当前点击按钮所在的数据行，直接将其从 DOM 树中剔除；
- **编辑数据（Update）**：需要精准锁定该行中的具体单元格，调起交互弹窗获取用户输入的新文本，再将其回填进对应的单元格节点。

在这三个操作中，编辑逻辑涉及“节点寻址 $\to$ 数据获取 $\to$ 内容改写”的完整闭环，复杂度最高；而新增和删除相对更为直观清晰。因此，按照先易后难的原则，我们先实现新增与删除，最后攻坚编辑功能。

#### 动态行与列节点的构建与装配

新增数据操作的核心目标，是在用户每一次点击“新增数据”按钮时，向表格末尾无缝插入一个全新的数据行。我们为按钮绑定点击事件：`<button onclick="add_row()">新增数据</button>`。

在实现新增行的算法中，必须严格遵循 DOM 树的操作顺序：**先确定挂载目标与插入索引 $\to$ 插入新行 $\to$ 在新行中依次插入各列单元格 $\to$ 填充单元格文本与交互控件**。

```text
  table.insertRow(length)
  +-------------------------------------------------------------+
  | new_row (<tr> 节点)                                         |
  |                                                             |
  |   new_row.insertCell(0)  -->  <td>未命名</td>               |
  |   new_row.insertCell(1)  -->  <td>无联系方式</td>           |
  |   new_row.insertCell(2)  -->  <td><button>...</button></td> |
  +-------------------------------------------------------------+
```
> **图示说明**：新增数据时，首先在表格最末尾插入一个 `<tr>` 容器节点，随后在该行节点内部依次调用 `insertCell()` 分别装配姓名、电话和操作三个 `<td>` 单元格。

```javascript
// 新增数据业务函数
function add_row() {
    // 1. 获取目标表格 DOM 节点对象
    var table = document.getElementById("table");

    // 2. 计算新行插入的物理索引位置
    // table.rows 返回当前表格所有 <tr> 的集合，length 即为行数总和
    var length = table.rows.length;

    // 3. 在表格最末尾插入全新行节点
    var new_row = table.insertRow(length);

    // 4. 在新行中依次插入三个列单元格节点
    var name_column = new_row.insertCell(0);   // 姓名列（索引 0）
    var phone_column = new_row.insertCell(1);  // 联系方式列（索引 1）
    var action_column = new_row.insertCell(2); // 操作列（索引 2）

    // 5. 修改各单元格的文本与内部 HTML 内容
    name_column.innerHTML = "未命名";
    phone_column.innerHTML = "无联系方式";
    action_column.innerHTML = '<button onclick="edit_row(this)">编辑</button><button onclick="delete_row(this)">删除</button>';
}
```

> **易错点**：在调用表格插入行方法时，极容易手误漏打字母，例如将 `insertRow` 误写为少打一个 `t`。此时在控制台点击按钮，浏览器会立即抛出异常：`Uncaught TypeError: table.insertRow is not a function`。一旦遇到此类提示，说明浏览器在当前 DOM 节点对象的原型链上找不到该方法名，第一反应必须是检查 API 单词拼写。
> 另外，在 `action_column.innerHTML` 动态拼装 HTML 字符串时，新生成的编辑与删除按钮同样必须挂载 `onclick="edit_row(this)"` 和 `onclick="delete_row(this)"` 事件，否则新增出来的行将无法被编辑和删除。

除了利用表格专属的高层 API（`insertRow` 与 `insertCell`）之外，在通用 DOM 编程体系中，也可以使用更底层的核心方法组合来完成同样的创建与挂载：
- `document.createElement("tr")`：创建原生元素节点；
- `document.createElement("td")`：创建子级单元格节点；
- `parentElement.appendChild(childElement)`：将创建好的子节点追加至父节点末尾；
- `parentElement.insertBefore(newElement, referenceElement)`：在指定参考节点之前插入新节点。
表格高层 API 实际上正是对这一系列基础节点操作的高效封装。

#### 上下文指针与DOM父节点溯源删除

删除数据的功能需求是：无论表格中有多少行数据，点击某一行内的“删除”按钮，就必须只把**该按钮所在的这一行整行销毁**。

要实现这个效果，有两个核心技术难点必须突破：第一，函数如何知道用户点击的具体是哪一个按钮？第二，按钮本身只是单元格内部的一个子元素，如何通过按钮将整行节点彻底移除？

**上下文关键字 this 的指向机理**：在给删除按钮绑定事件时，我们传入了一个特殊的实参：
```html
<button onclick="delete_row(this)">删除</button>
```
> **定义**：在 JavaScript 中，`this` 是一个保留关键字，它始终指向**当前函数执行的上下文对象（Context）**。
> 当 `this` 出现在 HTML 标签的行内事件监听属性（如 `onclick`）中作为参数传递时，它精准且唯一地指向**当前触发该事件的 DOM 元素节点本身**。

因此，当点击第一行的删除按钮时，函数形参接收到的就是第一行里的 `<button>` 节点对象；点击第二行的删除按钮时，接收到的就是第二行的 `<button>` 节点。

**DOM 树的向上溯源寻址（parentNode）**：在 DOM 树模型中，节点与节点之间有着严格的父子层级网络。通过节点层级分析：
- 按钮自身是 `<button>` 节点；
- 按钮的直接父节点（`button.parentNode`）是包裹它的单元格 `<td>`；
- 单元格的父节点（`button.parentNode.parentNode`）则是包裹它的整行 `<tr>`。

```text
  table (或 tbody)  <=== row.parentNode
     |
    tr              <=== row = button.parentNode.parentNode
     |
    td              <=== button.parentNode
     |
   button           <=== 触发事件的 this 节点
```

**DOM 节点销毁的核心定律**：
> **注意**：在标准 DOM 操作中，**任何一个 DOM 元素节点都无法调用方法自己把自己从文档中删除**！
> 要销毁一个目标节点，必须先顺藤摸瓜找到它的直接父级节点，然后由父级节点调用 `removeChild()` 方法，将目标子节点移出文档树：
> $$\text{parentNode.removeChild(childNode)}$$

综合上述三步推导，我们的整行删除逻辑实现如下：

```javascript
// 删除数据行函数
function delete_row(button) {
    // 1. 获取按钮的父节点(td)的父节点(tr)，即定位到目标数据行
    var row = button.parentNode.parentNode;

    // 2. 通过行节点(tr)的父级节点(table 或自动包裹的 tbody)，将当前行节点剔除
    row.parentNode.removeChild(row);
}
```

#### 单元格提取与弹窗交互编辑

数据的编辑功能要求在点击“编辑”按钮后，弹出输入框允许用户分别修改姓名与联系方式，并在确定后将新的内容实时回显在当前行的单元格内。

与删除操作相同，编辑函数在 HTML 标签中同样借助 `this` 获取当前被点击的按钮引用：`<button onclick="edit_row(this)">编辑</button>`。

```javascript
// 编辑数据行函数
function edit_row(button) {
    // 1. 向上回溯两层，锁定当前数据行 tr
    var row = button.parentNode.parentNode;

    // 2. 通过行的 cells 集合属性精确提取指定列单元格
    // cells[0] 对应当前行的第 0 列（姓名列 td）
    var name_cell = row.cells[0];
    // cells[1] 对应当前行的第 1 列（联系方式列 td）
    var phone_cell = row.cells[1];

    // 3. 调用浏览器内置输入对话框接收用户输入
    var input_name = prompt("请输入姓名：", name_cell.innerHTML);
    var input_phone = prompt("请输入联系方式：", phone_cell.innerHTML);

    // 4. 边界校验：防止用户点击取消或未输入时覆盖原有内容
    if (input_name !== null && input_name.trim() !== "") {
        name_cell.innerHTML = input_name;
    }
    if (input_phone !== null && input_phone.trim() !== "") {
        phone_cell.innerHTML = input_phone;
    }
}
```

**row.cells 属性的高效寻址**：除了通过通用方法遍历获取子节点外，HTML DOM 表格行对象原生提供了一个专用的属性——`cells`。`row.cells` 是一个包含该行内部所有 `<td>` 和 `<th>` 单元格的有序集合。通过下标 `cells[0]` 和 `cells[1]`，可以直观地提取出第一列和第二列。

**window.prompt 模态对话框交互**：`prompt()` 是浏览器窗口对象提供的同步阻塞式输入弹窗。方法接收一个字符串作为输入提示语，同时可以传入第二个可选参数作为输入框内的默认初始值（我们直接把当前单元格原本的内容 `name_cell.innerHTML` 作为初始值传入，使用户交互更自然）。用户在弹出的窗口中输入文本并点击“确定”后，该方法会返回用户输入的字符串；如果用户点击了“取消”，方法则返回 `null`。

**边界容错与健壮性防护**：在实际编码中，如果直接执行 `name_cell.innerHTML = input_name;`，会暴露出明显的边界漏洞：当用户误触编辑按钮后点击弹窗上的“取消”，由于返回值为 `null`，原本存在的姓名和手机号会被赋值为字符串 `"null"` 或被冲刷为空白；如果用户什么都没输入直接点确认，也会把原数据清空。因此，我们在代码中追加了判空防御（`if (input_name !== null && input_name.trim() !== "")`），唯有用户真正输入了有效内容时才执行覆盖更新。

此外，当连续点击删除按钮直到表格中所有行都被清空时，表格只剩下一个表头。在更完善的业务逻辑中，还可以监测 `table.rows.length` 的数值，在只剩一行表头时动态插入一行“暂无数据”的空状态占位提示。

---

### 移动端视口机制与rem等比适配

完成了桌面端的表格实战，我们来探讨一个至关重要的现实课题——响应式设计与移动端页面适配。

在过去的练习中，代码几乎都是在 PC 端全屏浏览器中运行。在电脑大屏幕上，视窗尺寸充裕，各分辨率之间的视觉差异往往不易察觉。但当用户使用智能手机、平板电脑等移动设备打开同一个网页时，如果页面缺乏移动端适配，就会发生严重的排版问题：整个页面可能被等比极度缩小成微小的文字堆，或者页面内容因超出屏幕宽度而被截断，用户必须频繁用手指横向拖拽和放大缩小才能看清局部内容。

移动端页面开发必须具备**响应式设计（Responsive Web Design）**能力，确保同一套核心内容能够在不同尺寸、不同像素密度的屏幕终端上，均呈现出良好的排版与交互体验。

#### 浏览器调试工具与移动端模拟

要开发和调试移动端网页，并不需要每次都把代码部署到真实的手机上进行测试。主流现代浏览器内置了完善的设备仿真套件。

工欲善其事，必先利其器。在前端开发者的日常工作流中，**Google Chrome**、**Mozilla Firefox** 以及基于 Chromium 内核的**新版 Microsoft Edge** 是公认最强大、最好用的三款开发工具。

以 Chrome 浏览器为例，按下快捷键 `F12`（或右键菜单选择“检查”/ Inspect）调出开发者工具（DevTools）：

**控制台停靠布局切换**：点击 DevTools 窗口右上角的三点菜单图标，可以调整调试窗口的停靠位置：
- 独立窗口（Undock into separate window）：双屏开发时的首选；
- 停靠在左侧（Dock to left）；
- 停靠在底部（Dock to bottom）：适合排查长文本和控制台网络请求；
- 停靠在右侧（Dock to right）：在调试响应式与移动端页面时推荐采用，左侧留出的纵长区域正好与手机屏幕的纵向形态吻合。

**移动端仿真模拟器**：在开发者工具左上角，有一个形似手机与平板叠放的图标（快捷键 `Ctrl+Shift+M` / `Cmd+Shift+M`），点击即可切换为移动端设备仿真视图。

模拟器顶部工具栏提供了一系列核心仿真配置：
- **设备预设与型号选择**：下拉菜单中预置了大量真实机型配置（如 iPhone 12 Pro、iPhone SE、iPhone XR、iPad 等）。切换机型后，视口会自动模拟该机型的物理逻辑分辨率；
- **尺寸读数（如 390 × 844）**：标明当前模拟设备视口的逻辑宽度（Width）与逻辑高度（Height）；
- **缩放比例（Zoom）**：控制渲染视口在当前电脑屏幕上的显示比例。`100%` 表示 1:1 等倍呈现，`50%` 缩小一倍以便预览超长平板，`200%` 放大双倍以便检查微观像素对齐；
- **屏幕旋转（Rotate）**：一键切换设备的横屏（Landscape）与竖屏（Portrait）姿态。

**移动端自适应两大主流方案对比**：在现代 Web 前端开发中，实现跨设备屏幕适配的主流方案主要分为两大流派：

| 适配方案 | 核心技术载体 | 运作机理与特征 | 典型应用场景 |
| :--- | :--- | :--- | :--- |
| **等比缩放方案** | `rem` 相对长度单位<br>视口单位（`vw` / `vh`） | 页面元素的排版拓扑固定，所有字号、宽高、边距跟随屏幕物理宽度按严格数学比例等比缩放。 | 电商移动端页面（如小米商城移动端）、内容展示类移动 Web 站点。 |
| **媒体查询响应式方案** | CSS3 媒体查询（`@media`） | 维持一套 HTML DOM 结构不变，针对不同屏幕宽度断点（如手机、平板、桌面）分别挂载多套截然不同的 CSS 样式规则，在小屏下折叠单列，在大屏下展开多列。 | 官方教程网站、技术文档中心、企业门户官网、响应式博客。 |

#### Viewport元标签与视口缩放控制

在深入探讨等比缩放单位之前，我们必须先彻底搞清楚在 HTML 模板头部的这行关键代码：
```html
<meta name="viewport" content="width=device-width, initial-scale=1.0">
```

> **定义**：**视口（Viewport）**是指浏览器用来真正渲染和呈现网页内容的可见屏幕区域。它直接决定了网页在用户设备上的初始大小、布局边界以及缩放行为。

**有无 Viewport 的对比分析**：
我们将先前编写的包含 `width: 100%;` 的表格在 Chrome 移动端模拟器中打开：
1. **保留 Viewport 标签时**：表格宽度与手机屏幕宽度完全贴合，文字清晰可辨，排版完整且自然；
2. **将该 Viewport 标签完全注释或删除后**：再次刷新移动端模拟器，整个页面退化成极小的一块，文字被严重缩微，页面右侧被大片截断，必须在屏幕上手动左右滑动画布才能看清全貌。

**移动浏览器默认虚拟视口机理**：
在早期智能手机普及初期，绝大多数网站都是专为宽大的 PC 桌面屏幕设计的（典型版心宽度在 980px 到 1200px 之间）。手机浏览器为了让用户依然能够访问这些未经适配的 PC 网站，默认会采用一个宽达 `980px` 的**虚拟视口（Layout Viewport）**来排版页面，然后再把这整整 980px 的画面等比强行压缩进手机只有三百多像素的窄小物理屏幕中。

如果一个页面没有声明 Viewport 元标签，移动端浏览器就会将其一律视为传统的 PC 网站，自动启用 980px 虚拟视口并强制缩小，导致布局失真。

**Viewport 元标签属性参数配置**：
通过 `<meta name="viewport" content="...">` 中的 `content` 键值对，开发者可以控制浏览器的视口渲染规则：

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0, minimum-scale=1.0, maximum-scale=1.0, user-scalable=no">
```

- **`width=device-width`**：命令浏览器将当前视口的宽度设定为设备的物理逻辑宽度（Device Width）。这是移动端适配的核心，它确保页面直接以设备真实的逻辑像素作为基准来布局，摒弃 980px 的旧假定；
- **`initial-scale=1.0`**：设定页面初始加载时的缩放比率为 1.0（即 100% 原始比例），保证页面以原始尺寸展示；
- **`minimum-scale=1.0` 与 `maximum-scale=1.0`**：定义允许用户通过双指捏合手势进行手动缩放的最小倍率与最大倍率；
- **`user-scalable=no`**：明确指示浏览器禁止用户手动缩放页面，使 Web 页面呈现出类似于原生手机 App 般的固定视窗体验。

#### 绝对单位px与相对单位rem的差异

在初学 CSS 时，接触最早、使用最普遍的长度单位是像素 `px`。但到了移动端，单纯使用 `px` 会暴露出明显的局限性。

**绝对长度单位 px（Pixel）**：像素是一个固定的绝对长度度量单位。在 CSS 中写下 `width: 300px;`，无论把这个盒子放在宽度为 320px 的小屏手机上，还是宽度为 1920px 的桌面显示器上，该元素所消耗的屏幕宽度永远被定死在 300 个逻辑像素。它缺乏自我感知和按比例适配设备环境的能力。

**相对长度单位 em 与 rem 的参考系差异**：
为了赋予元素根据环境自适应伸缩的能力，CSS 规范引入了相对长度单位。而在相对单位中，最核心的区分在于 `em` 与 `rem`：

```text
  em 的参考基准：当前元素自身（或直接父级）的 font-size
  +-------------------------------------------------------------+
  | .parent (font-size: 20px)                                   |
  |    |                                                        |
  |    +---> .child { width: 5em; } ==> 5 * 20px = 100px        |
  +-------------------------------------------------------------+

  rem 的参考基准：永远严格只锁定 html 根标签的 font-size
  +-------------------------------------------------------------+
  | html (font-size: 20px)                                      |
  |    |                                                        |
  |    +---> .box1 { width: 5rem; } ==> 5 * 20px = 100px        |
  |    |                                                        |
  |    +---> div > p > span { width: 5rem; } ==> 5 * 20px = 100px|
  +-------------------------------------------------------------+
```
> **图示说明**：`em` 随父级层级嵌套容易出现多重继承的连乘放大风险；而 `rem`（Root em）则跳过了所有中间父级，参考系唯一指向最顶层的根节点 `<html>`。

- **`em`**：相对于当前元素自身的字体尺寸（若当前元素未设置，则继承父级元素的 `font-size`）。一旦 DOM 结构发生多层嵌套，每一层都设置了相对字体大小，`em` 的最终换算像素就会层层累乘，导致尺寸难以预测；
- **`rem`（Root em）**：**根相对长度单位**。无论当前元素被嵌套在页面 DOM 树的哪一层，它的换算基准永远且唯一指向最顶层的根元素——**`<html>` 标签的 `font-size`**。

**rem 的基本换算模型**：
在绝大多数主流浏览器中，根元素 `<html>` 的默认基准字体大小为 `16px`。在未经修改的情况下：
$$1\text{rem} = 16\text{px}$$

如果通过 CSS 覆盖根元素的字号：
```css
html {
    font-size: 20px;
}
```
那么页面中所有使用 `rem` 的元素尺寸计算关系为：
- `5rem = 5 \times 20\text{px} = 100\text{px}`
- `3rem = 3 \times 20\text{px} = 60\text{px}`

这引出一个核心推导：如果在 CSS 中把 `html` 的字号写死成固定数值（例如 `20px`），那么在任何设备下 `5rem` 依然等于固定的 `100px`。

> **结论**：**`rem` 能够实现移动端响应式等比缩放的关键，在于根元素 `html` 的 `font-size` 绝不能是一个静态常数，它必须跟随当前设备的屏幕宽度动态计算。**

#### 基于视口宽度的根字体动态计算与窗口监听

要让根元素的 `font-size` 随设备屏幕宽度动态改变，静态的 CSS 无法单独完成。我们必须借助 JavaScript 介入 DOM 运行时，在页面初始化以及窗口发生尺寸变化时，动态计算并重设 `<html>` 的字体大小。

**适配计算模型的建立**：
为了使换算简洁明了，我们设定一个基准规则：**将 `html` 根节点的 `font-size` 设定为当前屏幕宽度的十分之一（即 10%）**：

$$\text{html.fontSize} = \frac{\text{screen.width}}{10} + \text{"px"}$$

在这一算法设定下，`1rem` 就代表了当前设备屏幕总宽度的 10%：
$$1\text{rem} = 0.1 \times \text{screen.width}$$

如果定义一个盒子的宽度为 `5rem`，那么在任何移动设备上，它所实际渲染的像素宽度都等于：
$$\text{Width} = 5 \times 1\text{rem} = 5 \times (0.1 \times \text{screen.width}) = 0.5 \times \text{screen.width}$$
即无论将页面置于何种宽度的屏幕上，该盒子始终占据当前屏幕宽度的**一半（50%）**。

**JavaScript 动态设置根字体实现**：
在 HTML 文档中嵌入计算脚本：

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>rem移动端等比适配演示</title>
    <style>
        /* 绝对像素盒子对比组 */
        .box-pixel {
            width: 300px;
            height: 100px;
            background-color: purple;
            margin-bottom: 50px;
        }
        /* rem等比自适应盒子试验组 */
        .box-rem {
            width: 5rem;   /* 恒定占据屏幕宽度的 50% */
            height: 3rem;  /* 恒定占据屏幕宽度的 30% */
            background-color: aqua;
        }
    </style>
</head>
<body>
    <div class="box-pixel"></div>
    <div class="box-rem"></div>

    <script>
        // 动态根据设备屏幕宽度重新计算 html 根节点 font-size
        function reset_html_font_size() {
            // document.documentElement 即代表 <html> 根节点对象
            // 将其样式中的 fontSize 属性动态赋值为屏幕宽度的 1/10
            document.documentElement.style.fontSize = (screen.width / 10) + "px";
        }

        // 页面初始化时立即执行计算
        reset_html_font_size();

        // 关键事件绑定：监听设备尺寸重置与横竖屏旋转
        window.onresize = reset_html_font_size;
    </script>
</body>
</html>
```

**窗口尺寸变动监听**：
在上述代码中，如果仅仅在页面加载时调用一次 `reset_html_font_size()`，当用户在横屏和竖屏之间切换，或者调整浏览器窗口大小时，页面将无法感知尺寸变化，导致缩放比例失效。

通过给全局窗口绑定 `window.onresize = reset_html_font_size;`，一旦视窗几何尺寸发生变化，浏览器便会重新触发该函数，捕获最新的宽度并更新 `<html>` 的 `font-size`，实现实时的响应式等比适配。

**多设备实测数据对比**：
在 Chrome 模拟器中切换不同机型，观察 `<html>` 标签的实际 `font-size` 渲染值，可以印证该计算模型的准确性：

| 测试设备型号 | 设备视口宽度（Logic Width） | 动态计算出的根字体（html font-size） | 5rem 盒子实际渲染像素宽度 | 占当前屏幕宽度比例 |
| :--- | :--- | :--- | :--- | :--- |
| **iPhone SE** | 375px | $375 \div 10 = \mathbf{37.5\text{px}}$ | $5 \times 37.5\text{px} = \mathbf{187.5\text{px}}$ | 50.0% |
| **iPhone 12 Pro** | 390px | $390 \div 10 = \mathbf{39.0\text{px}}$ | $5 \times 39.0\text{px} = \mathbf{195.0\text{px}}$ | 50.0% |
| **iPhone 14 Pro Max** | 430px | $430 \div 10 = \mathbf{43.0\text{px}}$ | $5 \times 43.0\text{px} = \mathbf{215.0\text{px}}$ | 50.0% |
| **Surface Pro 7** | 912px | $912 \div 10 = \mathbf{91.2\text{px}}$ | $5 \times 91.2\text{px} = \mathbf{456.0\text{px}}$ | 50.0% |

与此同时，采用绝对单位 `300px` 的紫色盒子，在小屏 iPhone SE 上占据过大版面，在宽屏 Surface Pro 7 上则显得局促；相比之下，`rem` 天蓝色盒子在任何设备上都保持在屏幕宽度的 50%，展现出优秀的跨设备适应能力。

---

### Flex弹性盒模型与容器布局系统

解决了长度单位在不同设备上的自适应缩放问题后，我们进入现代 CSS 中应用最广泛的排版利器——**Flex 弹性盒布局（Flexible Box Layout，简称 Flexbox）**。

在传统网页排版体系中，开发者若想将块级元素横向并排陈列，通常依赖 `float: left;`，紧接着就要处理高度塌陷与清除浮动的复杂问题；而要想实现“水平与垂直居中”，则往往需要依赖相对定位配合绝对定位或位移偏移。Flexbox 的出现彻底革新了排版逻辑。

在弹性盒模型下，父容器内的子元素能够自动进行空间分配、对齐与折叠，与 `rem` 或百分比单位配合使用，构成了现代前端界面工程的核心。在 CSS 中，**任何一个容器元素都可以通过设置 `display: flex;` 开启弹性盒上下文**。

#### 弹性容器与主轴交叉轴坐标体系

理解 Flexbox 的第一步，是建立起其特有的空间几何坐标概念：

```text
                      Cross Axis (交叉轴)
                          cross start
                              |
        +---------------------v---------------------+
        | Flex Container (弹性容器)                 |
        |                                           |
  main  |   +---------+   +---------+   +---------+ |  main
  start |   | Item 1  |   | Item 2  |   | Item 3  | |  end
  ----->|   |         |   |         |   |         | |-----> Main Axis
        |   +---------+   +---------+   +---------+ |        (主轴)
        |     main size                             |
        |                                           |
        +---------------------^---------------------+
                              |
                          cross end
```
> **图示说明**：采用 Flex 布局的元素被称为弹性容器，其子元素自动成为弹性项目。容器内存在相互垂直的正交双轴：**主轴（Main Axis）**与**交叉轴（Cross Axis）**。项目的排列、缩放与对齐完全由这两根轴线决定。

**弹性容器（Flex Container）与弹性项目（Flex Item）**：
- **弹性容器**：显式声明了 `display: flex;`（块级弹性盒）或 `display: inline-flex;`（行内弹性盒）的父级 HTML 元素。
- **弹性项目**：弹性容器的**直接子元素**自动成为弹性盒的成员，被称为 Flex Item。项目原本的块级或行内属性（如 `div` 的独占一行特性、`span` 的不可设宽高特性）在进入弹性盒上下文后均转为受弹性规则支配。

**双轴坐标体系与几何边界**：
Flex 布局并非依据传统的物理方位排版，而是依托两根正交的轴线：
- **主轴（Main Axis）**：弹性项目沿着排列的主要轴线。在默认状态下，主轴呈水平横向；
- **交叉轴（Cross Axis）**：垂直于主轴的副轴线。在默认状态下，交叉轴呈垂直纵向；
- **主轴起点（main start）与终点（main end）**：主轴排布的起始坐标边界与终止坐标边界；
- **交叉轴起点（cross start）与终点（cross end）**：交叉轴排布的起始坐标边界与终止坐标边界；
- **项目尺寸（main size 与 cross size）**：单个弹性项目在主轴方向上占据的宽度或高度称为 main size；在交叉轴方向上占据的尺寸称为 cross size。

#### 主轴方向与换行控制

作用在弹性容器上的属性，负责控制主轴的朝向以及空间耗尽时的换行策略。

**主轴方向控制（flex-direction）**：
`flex-direction` 用于定义主轴的方向，直接决定了弹性项目在容器内的先后排布顺序：
- **`row`（默认值）**：主轴为水平方向，起点在容器左端（main start 在左），项目从左往右依次排列；
- **`row-reverse`**：主轴为水平方向，起点在容器右端（main start 在右），项目从右往左反向排列（原本的 1、2、3 项会倒置显示在右侧）；
- **`column`**：主轴切换为垂直方向，起点在容器顶端（main start 在上），项目从上往下依次排列；
- **`column-reverse`**：主轴为垂直方向，起点在容器底端（main start 在下），项目从下往上反向排列。

**轴线换行行为（flex-wrap）**：
在默认情况下，Flex 容器内的所有项目都会排在同一条单轴线上。如果容器的主轴空间不足以容纳所有项目，默认不会换行，而是会压缩项目的宽度。

`flex-wrap` 属性用于定义当单条轴线空间不足时是否换行以及换行的方向：
- **`nowrap`（默认值）**：不换行。所有项目保持在同一条轴线上，项目空间被自动挤压变窄；
- **`wrap`**：允许正常换行。当单行排满后，多余的项目自动折入下一行：
  - 当主轴为水平方向时，从上到下产生新行；
  - 当主轴为垂直方向时，从左到右产生新列；
- **`wrap-reverse`**：反向换行：
  - 当主轴为水平方向时，新行从下方往上方反向折叠排列；
  - 当主轴为垂直方向时，新列从右侧往左侧反向折叠排列。

**方向与换行的复合简写（flex-flow）**：
`flex-flow` 是 `flex-direction` 与 `flex-wrap` 两个属性的简写形式：
```css
flex-flow: <flex-direction> <flex-wrap>;
```
例如 `flex-flow: row wrap;` 等同于主轴横向排列且允许向下换行。

#### 主轴空间对齐与项目分布

当所有项目自身的尺寸相加后未填满整根主轴时，容器在主轴方向上剩余的空白空间需要通过 `justify-content` 属性进行分配。该属性用于定义**项目在主轴上的对齐与间隙分布方式**。

- **`flex-start`（默认值）**：项目紧靠主轴起点（main start）排列，空白空间留在主轴尾部；
- **`flex-end`**：项目紧靠主轴终点（main end）排列，空白空间留在主轴头部；
- **`center`**：项目向主轴中央居中对齐，空白空间均分在首尾两端；
- **`space-between`**：两端对齐。第一个项目紧贴主轴起点，最后一个项目紧贴主轴终点，剩余空白空间在项目之间均匀平分；
- **`space-around`**：环绕对齐。每个弹性项目的**两侧均分配相等的空白间隔**。导致相邻项目之间的间距（左项右间距 + 右项左间距），是首尾两端项目距离容器边框间距的**两倍**；
- **`space-evenly`**：均分对齐。容器内各个项目之间的间隙，以及首尾项目与外层容器边框之间的间隙，全部绝对相等。

| 对齐属性值 | 首尾项目与容器边框的距离 | 相邻项目之间的间隙距离 | 视觉核心特征 |
| :--- | :--- | :--- | :--- |
| **`space-between`** | $0$（紧贴边框） | 剩余空间在 $N-1$ 个间隙中均分 | 首尾顶格，中间间隙均等。 |
| **`space-around`** | 设为 $x$（单侧留白） | $2x$（两倍于两端留白） | 两头留白较窄，项目之间留白较宽。 |
| **`space-evenly`** | 设为 $x$ | $x$（与两端留白完全相等） | 所有缝隙绝对均等。 |

#### 交叉轴对齐与多轴线对齐机制

交叉轴上的对齐与空间分配由 `align-items` 与 `align-content` 共同管理。两者的作用范围有着清晰的边界划分。

**单轴线交叉轴对齐（align-items）**：
`align-items` 定义弹性项目在**当前所处的单条轴线内部**，如何沿着交叉轴进行对齐：
- **`stretch`（默认值 / 拉伸）**：如果项目未显式设置高度（或高度为 `auto`），项目将被拉伸填满整个容器在交叉轴上的可用高度；若项目自身设置了固定高度，则保持设定高度不被拉伸；
- **`flex-start`**：与交叉轴的起点（默认是容器顶端）对齐；
- **`flex-end`**：与交叉轴的终点（默认是容器底端）对齐；
- **`center`**：在交叉轴上垂直居中对齐；
- **`baseline`**：项目依照内部第一行文字的文本基线对齐（当各个子项内的字体大小不同时，基线对齐能够确保文字底部在视觉上处于同一水平线）。

**多轴线交叉轴全局对齐（align-content）**：
> **注意**：`align-content` 属性生效的**前提条件**是——**容器开启了换行（`flex-wrap: wrap` 或 `wrap-reverse`），且子项目在实际渲染中确实产生了多根轴线（即折成了两行或两行以上）！**
> 如果容器内的项目全部处于单根轴线上，无论怎样修改 `align-content` 的值，都不会产生任何视觉效果。

**align-items 与 align-content 的行为差异**：
当折行产生多条轴线时，两者的行为表现明显不同：
- 产生两行项目时，如果仅设置 `align-items: center;`，浏览器会将容器的垂直高度均分为两部分；第一行的项目仅在第一行的垂直空间内居中，第二行的项目仅在第二行的垂直空间内居中，两行之间依然存在空白间隙。
- 如果需要**让多行项目作为一个整体，在容器的垂直正中央对齐**，则必须使用 `align-content: center;`。它将所有轴线作为一个整体统一分配在交叉轴上。

`align-content` 的属性值包括：`flex-start`、`flex-end`、`center`、`space-between`、`space-around`、`space-evenly` 以及 `stretch`。

#### 项目伸缩弹性与综合代码实操

除了给容器赋予控制属性外，Flex 项目本身也具备精细控制自身伸缩能力的属性：
- **`flex-grow`**：定义项目的**放大比例**。默认值为 `0`（即使容器在主轴方向有剩余空间，该项目也不放大）；若设为 `1`，则项目会自动吸收多余空间放大；
- **`flex-shrink`**：定义项目的**缩小比例**。默认值为 `1`（当主轴空间不足且容器不换行时，项目会自动等比收缩）；若将其设为 `0`，则表明该项目不被压缩，保持原始尺寸；
- **`flex-basis`**：定义在分配多余空间之前，项目在主轴上占据的基准尺寸。默认值为 `auto`（即项目本身的原本尺寸）；
- **`flex` 复合简写**：上述三者的简写形式，语法为：
  ```css
  flex: <flex-grow> <flex-shrink> <flex-basis>;
  ```
  默认值为 `0 1 auto`。日常开发中最常用的写法是 `flex: 1;`（等价于 `1 1 0%`），常用于让子项目均分容器的剩余宽度。

为了验证上述容器属性，我们搭建完整的实操代码沙箱：

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <title>Flex弹性布局综合实战</title>
    <style>
        /* 设定根基准字体，便于配合 rem 观察弹性尺寸 */
        html {
            font-size: 10px;
        }

        /* 弹性容器 */
        .container {
            width: 100%;
            height: 40rem;
            background-color: aqua;
            
            /* 1. 开启弹性盒布局上下文 */
            display: flex;

            /* 2. 主轴方向与换行控制 */
            flex-direction: row;
            flex-wrap: wrap;

            /* 3. 主轴对齐方式（可选：flex-start, flex-end, center, space-between, space-around, space-evenly） */
            justify-content: space-evenly;

            /* 4. 单轴交叉轴对齐 */
            align-items: center;

            /* 5. 多轴线全局对齐（换行产生多行时生效） */
            align-content: center;
        }

        /* 弹性项目基础样式 */
        .item {
            width: 20rem;       /* 20rem * 10px = 200px */
            height: 10rem;      /* 10rem * 10px = 100px */
            font-size: 4rem;
            text-align: center;
            line-height: 10rem;
            color: #fff;
            margin: 5px;
        }

        /* 为各个子项目赋予不同背景色以便肉眼追踪位置移动 */
        .item:nth-child(1) { background-color: yellowgreen; }
        .item:nth-child(2) { background-color: red; }
        .item:nth-child(3) { background-color: gray; }
        .item:nth-child(4) { background-color: gold; }
        .item:nth-child(5) { background-color: brown; }
        .item:nth-child(6) { background-color: #4aa; }
    </style>
</head>
<body>
    <!-- 使用 Emmet 语法一键生成结构：.container>.item*6{$} -->
    <div class="container">
        <div class="item">1</div>
        <div class="item">2</div>
        <div class="item">3</div>
        <div class="item">4</div>
        <div class="item">5</div>
        <div class="item">6</div>
    </div>
</body>
</html>
```

> **提示**：在编写 HTML 结构时，可以使用 Emmet 缩写：输入 `.container>.item*6{$}` 然后按下 `Tab` 键，编辑器会自动展开为一个类名为 `container` 的外层容器，内部包裹 6 个带有 `item` 类名且内容分别为 `1` 到 `6` 的子 `<div>`。在前端日常开发中，熟练使用这类缩写能够显著提升代码输入效率。

通过在上述样式中切换 `flex-direction: row-reverse;`（1 到 6 的顺序倒置），调整 `flex-wrap: nowrap;` 与 `wrap;`（观察宽度 200px 的盒子在窗口变窄时是被压缩还是顺畅折行），以及试验 `align-content: center;` 与未设置时的留白差异，可以直观掌握 Flex 布局的各种渲染效果。

---

### 前端技术进阶路径与跨端选型

学到这里，我们已经完整走完了前端基础三剑客的核心主干：
- **HTML**：搭建语义化的网页文档树骨架；
- **CSS**：借助盒子模型、定位机制、rem 等比缩放与 Flex 弹性盒，实现视觉美化与移动端响应式排版；
- **JavaScript**：通过变量控制、分支循环、函数封装、事件监听与 DOM 节点动态操作，为静态页面注入灵活的用户交互逻辑。

但必须清醒地认识到，这三大基础知识点只是推开了 Web 开发技术大门的第一步。在真实的工业级前端工程实践中，由于页面交互复杂度大幅上升，如果继续停留在手写原生 DOM 操作（频繁调用 `getElementById`、手动拼接 `innerHTML` 字符串、人工回溯 `parentNode`），不仅代码维护成本极高，而且容易出现数据与视图不一致的难以排查的问题。为了应对更庞大的应用开发，我们需要对后续的进阶路线有一个清晰的认知。

#### 现代MVVM前端框架选型

在夯实了原生基础之后，深入学习的首推方向是现代组件化前端框架——**Vue.js**。

**Vue.js 的核心优势**：Vue.js 是一套构建用户界面的渐进式 JavaScript 框架。它通过“数据驱动视图”的双向绑定与虚拟 DOM 机制，让开发者从繁琐的底层 DOM 节点创建、属性赋值与父子寻址中解脱出来。在 Vue 中，我们只需要在 JavaScript 中改变底层的业务数据变量，界面的 HTML 视图便会自动完成局部的精准更新。

**中文友好与平缓的学习曲线**：在当今三大主流前端框架（React、Vue、Angular）中，Vue.js 以其平缓的学习曲线、详尽的官方文档和优雅的 API 设计受到广泛欢迎。加之 Vue 的作者尤雨溪为华人，其官方文档提供了高水准的中文技术文档与活跃的社区支持，对于中文开发者来说，Vue 是迈入现代工程化前端开发的优秀起点。

#### 小程序生态与跨端开发架构

如果除了传统的 PC 与移动 Web 网站开发之外，对移动端应用、手机 App 软件开发抱有浓厚兴趣，那么**微信小程序（WeChat Mini Programs）**是极为推荐的学习方向。

**原生 App 开发的现实痛点**：在移动 App 开发领域，主流操作系统划分为苹果 iOS 与谷歌 Android 两大阵营（以及华为鸿蒙系统）。如果采用传统的原生语言开发 App，通常需要使用不同语言（Swift/Objective-C 与 Kotlin/Java）针对不同平台分别开发与维护；多端屏幕碎片化导致适配成本高昂，且用户的下载与安装门槛也影响了产品的推广。

**小程序生态的优势**：而在国内移动互联网环境下，微信拥有庞大的用户基础。微信小程序无需下载安装、即用即走、体积轻量，不仅抹平了 iOS 与 Android 底层系统的差异，降低了多端分别维护的成本，而且能够依托微信的社交与支付生态，是中小型项目与初创团队的高效选择。

**uni-app 多端跨平台开发架构**：当掌握了 Vue.js 的组件化开发思想，并了解了小程序的运行机制之后，我们可以将两者结合，采用跨端开发框架——**uni-app**。

uni-app 是一个基于 Vue.js 开发所有前端应用的统一框架。开发者编写一套遵循 Vue 语法的业务源码，通过编译器工具链，可以自动化打包编译并发布到：
- 微信小程序、支付宝小程序、百度小程序、抖音小程序等各大小程序平台；
- 运行于手机浏览器的移动端 Web 应用（H5）；
- 原生编译为可直接安装上架的 Android App 与 iOS App。

真正实现“一套代码，多端运行”的高效开发流程。

**学习进阶的非线性认知**：需要强调的是，在学习进阶的路径上，**Vue.js、微信小程序以及使用 Vue 进行跨端开发之间，不存在绝对死板的先后依赖顺序**。

在学完本教程的前端基础之后：
- 可以直接学习原生微信小程序开发，快速体验移动端产品的上线流程；
- 也可以直接学习 Vue.js 探索单页面应用（SPA）的组件化开发；
- 甚至可以在掌握 Vue 基础后跳过原生小程序，直接借助 uni-app 跨端框架开发小程序与 App。

根据自己的实际需求与兴趣方向，灵活选择、顺势而为，持续将技术积累转化为完整的数字化应用。

---

## 本册小结

本册主题为「JavaScript 基础与 DOM 实战：定位、事件、移动端适配与 Flex」，整编了 2 个模块（P13-P19 ~ P20-P23，共 11 讲 / 97 分钟音频）。
建议配合 `notes/` 目录下的复习笔记复盘，需要查证细节时可直接回到本册正文。
