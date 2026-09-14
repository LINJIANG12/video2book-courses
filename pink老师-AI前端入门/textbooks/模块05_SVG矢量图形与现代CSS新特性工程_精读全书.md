# 模块 05：SVG矢量图形与现代CSS新特性工程 合辑教材

> **所属课程**：2025版pink老师最新AI+前端入门教程，零基础必看的html5、css3、grid、flex布局、响应式、移动端，bootstrap5框架，AI工具  
> **模块跨度**：P115 ~ P137（全模块共 23 讲系统重构）  
> **内容定位**：模块化系统学习教材，融合核心机制、架构全景、代码解析与思考自测。  
> **关联说明**：单集长文讲义同步保留于 `articles/` 目录供定向查阅。

---

## 模块导读与全景目录

- **第 1 章**：114-第五篇案例展示以及SVG基本语法使用
- **第 2 章**：115-SVG图片-描边效果以及仿北大官网效果
- **第 3 章**：116-SVG图片-利用偏移实现画心效果
- **第 4 章**：117-SVG图片-仿oppoAI里程碑案例
- **第 5 章**：118-SVG图片-无人机动画案例
- **第 6 章**：119-clip-path裁剪-基本语法使用
- **第 7 章**：120-clip-path裁剪-灯光扫描文字效果
- **第 8 章**：121-turn旋转一圈-仿华为官网花瓣旋转效果
- **第 9 章**：122-滤镜filter的使用
- **第 10 章**：123-背景滤镜backdrop-filter
- **第 11 章**：124-动画时间线timeline-滚动时间线
- **第 12 章**：125-动画时间线timeline-视图时间线
- **第 13 章**：126-CSS变量的定义以及使用
- **第 14 章**：127-CSS计算函数calc基本使用
- **第 15 章**：128-CSS变量和计算函数修改精灵图坐标效果
- **第 16 章**：129-综合案例1-动感菜单
- **第 17 章**：130-综合案例2-滑动导航栏效果
- **第 18 章**：131-综合案例3-炫酷导航栏上
- **第 19 章**：132-综合案例3-炫酷导航栏下
- **第 20 章**：133-综合案例4-滚动叠加卡片首屏以及vw和vh单位
- **第 21 章**：134-综合案例4-滚动叠加卡片主体制作
- **第 22 章**：135-综合案例4-滚动叠加卡片制作
- **第 23 章**：136-综合案例4-滚动叠加卡片添加视图时间线

---

## 第 1 章：114-第五篇案例展示以及SVG基本语法使用
> 对应分集：P115 | 原始标题：《114-第五篇案例展示以及SVG基本语法使用》

从本节开始，我们正式跨入全课程最具技术前瞻性的核心板块——**第五篇：前沿技术拓展**。

在这一篇章中，我们将系统攻克现代前端开发中最具表现力的前沿利器：
- **SVG 矢量图形与路径描边动画**；
- **现代 CSS 形状裁剪（`clip-path`）** 与 **滤镜系统（`filter`）**；
- **滚动驱动动画（Scroll-driven Animations）**：`scroll-timeline` 与 `view-timeline`；
- **CSS 变量（Custom Properties）** 与 `calc()` 动态响应式计算。

作为第五篇的奠基石，本讲我们将深入理解 **SVG 矢量图形的底层原理**、工程化代码级引入方式，并手把手实现 **B 站官方导航微动效图标**。

---

### 1. 为什么现代大厂全面拥抱 SVG？

在过去，网页图标经历了三个技术世代：
1. **位图切图时代（PNG/JPG）**：高分屏（Retina）上容易模糊，且体积大、无法动态变色；
2. **字体图标时代（Iconfont）**：单色易用，但在多色图标、精细路径以及复杂动态交互上面临瓶颈；
3. **SVG 矢量图时代（现代主流）**：彻底打通了设计与代码的边界。

```text
位图 vs SVG 矢量图在 4K/视网膜屏上的表现
【位图 PNG (基于像素点阵)】
放大 400% ────► [ 严重马赛克、边缘模糊锯齿 ]

【SVG 矢量图 (基于数学几何公式)】
放大 4000% ───► [ 边缘绝对锐利、丝毫不失真 ]
```

#### SVG 的四大核心杀手锏：
1. **矢量无限无损**：SVG 基于数学矢量方程（点、线、曲线）绘制，放大 10 倍、100 倍依旧如刀锋般锐利；
2. **代码级可编辑**：它不是黑盒图片，而是一串标准 XML 代码！你可以像操作普通 HTML 标签一样，用 CSS 去修改某一条路径的描边、填充、渐变，甚至为特定锚点绑定动画；
3. **极强交互能力**：支持 CSS 伪类与 JS 动态控制；
4. **加载开销极小**：大多数纯矢量图标的代码体积仅几百字节，配合 Gzip 压缩后性能极佳。

---

### 2. SVG 的获取与内联工程化引入

很多初学者用 `<img src="icon.svg">` 引入 SVG，随后发现用 CSS 修改颜色或做动画完全不起作用。

> **核心避坑要点**：
> 如果通过 `<img>` 标签引入，浏览器会把 SVG 当作沙箱闭包图片处理，外部 CSS **无法穿透修改其内部的 `<path>` 路径**！
> **要想用 CSS 深度控制动效，必须采用“内联代码引入（Inline SVG）”！**

#### 2.1 获取标准 SVG 源码
1. 访问国内最常用的矢量图标库（如阿里巴巴 Iconfont 或各大开源图标站）；
2. 搜索所需图标（如“收藏”、“历史记录”、“消息”）；
3. 点击“下载素材”时，**务必选择“SVG”格式下载**；
4. 下载后用浏览器或编辑器打开该 `.svg` 文件，右键查看源码，会看到一段以 `<svg>` 开头、`</svg>` 结尾的代码块。

---

#### 2.2 SVG 内部结构全解
```html
<svg class="icon" viewBox="0 0 1024 1024" width="24" height="24">
  <!-- path 路径：所有几何形状由 d 属性中的坐标点描绘 -->
  <path d="M512 64C264.6 64 64 264.6 64 512s200.6 448 448 448..." fill="#333" />
</svg>
```
- **`<svg>`**：最外层画布容器；
- **`viewBox`**：视区坐标系（相当于画纸尺寸，通常设计稿为 `0 0 1024 1024`）；
- **`<path d="...">`**：路径标签，`d` 属性内存储着直线、贝塞尔曲线的绘制指令；
- **`fill`**：填充颜色（在 CSS 中对应 `fill` 属性，等价于普通盒子的 `background-color`）；
- **`stroke`**：描边颜色（等价于 `border-color`）。

> **尺寸控制法则**：
> SVG 属于图形元素，**不能使用 `font-size` 改变大小**！必须使用 `width` 和 `height` 属性（或在 CSS 中直接写宽高）来定义尺寸。

---

### 3. 实战：仿 B 站导航图标悬停“弹跳微动效”

大家在刷 B 站时会发现，鼠标滑过顶部的“动态”、“消息”、“收藏”等小图标时，图标并不会死板地变色，而是会**轻快地向上弹跳一下再落回原处**，极富灵动感。

这正是利用内联 SVG 结合 `@keyframes` 触发的经典单次动画。

#### 3.1 完整代码编写

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <title>仿 B 站导航 SVG 弹跳微动效</title>
  <style>
    * {
      margin: 0;
      padding: 0;
      box-sizing: border-box;
    }

    body {
      height: 100vh;
      display: flex;
      justify-content: center;
      align-items: center;
      background-color: #f6f7f8;
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
    }

    /* 1. 导航条容器 */
    .bili-nav {
      list-style: none;
      display: flex;
      gap: 30px;
      padding: 12px 24px;
      background: #fff;
      border-radius: 12px;
      box-shadow: 0 4px 16px rgba(0, 0, 0, 0.06);
    }

    /* 2. 单个导航项：图标在上，文字在下 */
    .bili-nav li {
      display: flex;
      flex-direction: column;
      align-items: center;
      cursor: pointer;
      color: #61666d;
      transition: color 0.2s ease;
    }

    /* 3. 内联 SVG 图标样式约束 */
    .bili-nav li svg {
      width: 22px;
      height: 22px;
      margin-bottom: 4px;
      fill: currentColor; /* 核心技巧：fill 继承父级文字颜色 */
    }

    .bili-nav li span {
      font-size: 13px;
    }

    /* =======================================================
       4. 核心微动效：鼠标悬停触发小图标单次弹跳
       ======================================================= */
    
    /* 悬停时文字与图标整体高亮（B站标志性粉色） */
    .bili-nav li:hover {
      color: #00aeec;
    }

    /* 核心：只让内部的 SVG 图标执行弹跳动画 */
    .bili-nav li:hover svg {
      animation: iconBounce 0.35s ease-out;
    }

    /* 5. 弹跳关键帧：向上窜动然后落回 */
    @keyframes iconBounce {
      0% {
        transform: translateY(0);
      }
      50% {
        /* 上冲 4 像素 */
        transform: translateY(-4px);
      }
      100% {
        /* 落回原点 */
        transform: translateY(0);
      }
    }
  </style>
</head>
<body>
  <ul class="bili-nav">
    <li>
      <!-- 收藏图标 (SVG 源码) -->
      <svg viewBox="0 0 1024 1024">
        <path d="M854.6 288.6L671.4 262 589.6 96.3c-33.8-68.5-121.4-68.5-155.2 0L352.6 262 169.4 288.6c-75.6 11-105.8 103.8-51.1 157.1l132.6 129.2-31.3 182.5c-12.9 75.3 66.2 132.7 133.8 97.2L512 768.4l158.6 86.2c67.6 35.5 146.7-21.9 133.8-97.2l-31.3-182.5 132.6-129.2c54.7-53.3 24.5-146.1-51.1-157.1z"/>
      </svg>
      <span>收藏</span>
    </li>
    <li>
      <!-- 历史记录图标 (SVG 源码) -->
      <svg viewBox="0 0 1024 1024">
        <path d="M512 64C264.6 64 64 264.6 64 512s200.6 448 448 448 448-200.6 448-448S759.4 64 512 64zm0 820c-205.4 0-372-166.6-372-372s166.6-372 372-372 372 166.6 372 372-166.6 372-372 372zm28-564h-56c-4.4 0-8 3.6-8 8v288c0 4.4 3.6 8 8 8h208c4.4 0 8-3.6 8-8v-56c0-4.4-3.6-8-8-8H540V328c0-4.4-3.6-8-8-8z"/>
      </svg>
      <span>历史</span>
    </li>
  </ul>
</body>
</html>
```

---

### 4. 实战高阶技巧：`fill: currentColor`

在上面的代码中，隐藏着一个极其优雅的 CSS 高阶神技：
```css
.bili-nav li svg {
  fill: currentColor;
}
```
- `currentColor` 代表当前元素的文本颜色（即 `color` 的值）；
- 当我们鼠标悬停在 `li` 上将 `color` 改为 `#00aeec` 时，内部的文字立刻变蓝，而 SVG 由于设置了 `fill: currentColor`，其填充颜色也会**自动同步变蓝**！
- 我们无需为 SVG 单独写颜色切换规则，实现了文字与矢量图标的完美联动换色。

> 💡 **承前启后**：完成对「114-第五篇案例展示以及SVG基本语法使用」的理解后，下一章我们将深入探讨「115-SVG图片-描边效果以及仿北大官网效果」，进一步完善知识图谱体系。

---

## 第 2 章：115-SVG图片-描边效果以及仿北大官网效果
> 对应分集：P116 | 原始标题：《115-SVG图片-描边效果以及仿北大官网效果》

很多高校（如北京大学官网）、顶尖设计工作室以及科技发布会主页中，常常有这样惊艳的一幕：

当页面滚动到某个区域或鼠标滑过图标时，建筑轮廓、校徽、科技图案如同有一支无形的马良神笔一样，沿着矢量轮廓**一笔一划、行云流水地“画”出来**，随后填入饱满的色彩。

很多初学者误以为这是用 JS 逐帧计算 Canvas 贝塞尔曲线的复杂算法，其实完全不需要！只要掌握 SVG 的 **`stroke` 描边系统** 与 **`stroke-dasharray` 虚线生长魔法**，几十行纯 CSS 就能复刻大厂顶级动效。

本节我们将彻底拆解 SVG 独有的 CSS 属性体系，并手把手实现 **北京大学官网同款线条生长微动效**。

---

### 1. SVG 独占样式属性系统

SVG 是基于 XML 的矢量图形规范，它拥有一套完全独立于普通 HTML 盒模型的专属样式属性：

| 属性名 | 作用 | 对标常规 CSS 属性 | 关键取值说明 |
| :--- | :--- | :--- | :--- |
| **`fill`** | 闭合图形内部填充色 | `background-color` | 颜色值（`#ff0000`, `currentColor`）或 `none`（镂空透明） |
| **`stroke`** | 路径线条描边颜色 | `border-color` | 沿数学路径外延描边的色彩 |
| **`stroke-width`** | 描边线条粗细 | `border-width` | 数字或像素值（如 `2`, `3px`） |
| **`stroke-dasharray`** | 描边虚线分割模式 | 虚线间隔设置 | 控制“实线段”与“空白间隙”的交替步长 |
| **`stroke-dashoffset`**| 虚线绘制起始偏移量 | 位移控制 | 控制虚线起始点向前或向后滑动 |

> **避坑法则：颜色覆盖失效？**
> 从阿里矢量图标库下载的 SVG 源码中，`<path>` 标签内部通常默认硬编码了 `fill="#333333"`。
> 在外部 CSS 写 `fill: red;` 时，由于 HTML 行内属性的权重极高，外部样式往往无法生效。
> **两大解法**：
> 1. 打开 SVG 源码，手动将 `<path>` 里的 `fill="..."` 属性删掉；
> 2. 在 CSS 中使用提权：`fill: red !important;`。

---

### 2. 核心数学解密：`stroke-dasharray` 虚线机制

要做出线条从无到有的“手绘生长感”，关键就在于 **`stroke-dasharray`**。

#### 2.1 参数语法剖析
```css
/* 单值语法：实线与间隙等长 */
stroke-dasharray: 100; /* 实线 100px，空白间隙 100px，交替循环 */

/* 双值语法：实线与间隙独立定义 */
stroke-dasharray: 100 50; /* 实线 100px，空白间隙 50px */
```

```text
stroke-dasharray 虚线排布可视化
stroke-dasharray: 100 50;
|◄── 100px 实线 ──►|◄─ 50px 间隙 ─►|◄── 100px 实线 ──►|
██████████████████                 ██████████████████
```

---

#### 2.2 仿北大官网：线条“凭空画出”的魔法

如何让原本看不见的线条，像笔画一样被一笔划出？

让我们来看一组极具智慧的动态关键帧设计：

```css
@keyframes drawLine {
  0% {
    /* 实线长度为 0，间距为 300 */
    stroke-dasharray: 0 300;
  }
  100% {
    /* 实线长度为 300，间距为 0 */
    stroke-dasharray: 300 0;
  }
}
```

##### 底层运行逻辑推演：
1. **初始状态（0%）**：
   - 实线长度为 `0`，而间歇空白长达 `300px`（已超过整个图标线条的周长）；
   - 此时整张图上全都是看不见的空白间隙，**视觉上整个图标彻底消失隐形**！
2. **渐变推进过程**：
   - 浏览器在动画补间中，逐渐将实线长度从 `0` 递增到 `300`，空白间歇从 `300` 缩减到 `0`；
   - 视觉上，那段长度为 0 的实线一点一点向外延伸拉长，如同一根墨水笔在纸上疾驰！
3. **终点状态（100%）**：
   - 实线长度达到 `300px`，间隙缩为 `0`；
   - 此时整段路径 100% 被实线填满，完整的图标绘制完毕！

---

### 3. 完整代码实战开发

下面我们选用经典古建筑/徽标矢量图，完整还原北京大学官网图标悬停手绘生长效果。

#### 3.1 HTML 结构
```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <title>SVG 描边手绘动效</title>
  <link rel="stylesheet" href="style.css">
</head>
<body>
  <div class="pku-card">
    <!-- 经典建筑轮廓 SVG 图标 -->
    <svg class="building-icon" viewBox="0 0 1024 1024">
      <path d="M512 64L128 256v128h768V256L512 64zm-256 384v320h128V448H256zm192 0v320h128V448H448zm192 0v320h128V448H640zM128 832v64h768v-64H128z"/>
    </svg>
    <p class="title">燕园建筑群</p>
  </div>
</body>
</html>
```

---

#### 3.2 CSS 核心样式编码

```css
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #8c1515; /* 北大红经典底色 */
}

/* 1. 卡片展示区 */
.pku-card {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 30px 40px;
  background-color: rgba(255, 255, 255, 0.08);
  border-radius: 16px;
  cursor: pointer;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.2);
  transition: background-color 0.3s ease;
}

.pku-card:hover {
  background-color: rgba(255, 255, 255, 0.15);
}

/* 2. SVG 基础样式配置 */
.building-icon {
  width: 120px;
  height: 120px;
  /* 核心 A：清空内部填充，只保留镂空轮廓 */
  fill: none;
  /* 核心 B：配置金黄色描边，粗细为 4px */
  stroke: #ffd700;
  stroke-width: 4;
}

.title {
  margin-top: 16px;
  color: #fff;
  font-size: 18px;
  letter-spacing: 2px;
}

/* =======================================================
   3. 鼠标悬停触发描边手绘动画
   ======================================================= */
.pku-card:hover .building-icon {
  /* 0.8秒平滑匀速画完一圈 */
  animation: drawStroke 0.8s ease-in-out forwards;
}

/* 4. 关键帧：从全间隙 (0 300) 到全实线 (300 0) */
@keyframes drawStroke {
  0% {
    stroke-dasharray: 0 300;
  }
  100% {
    stroke-dasharray: 300 0;
  }
}
```

---

### 4. 重点避坑与素材甄选指南

1. **素材必须选择“单线条/实心轮廓”**：
   - 如果你从素材网下载了一个原本带有双边描边的“空心图标”，加了 `stroke` 之后，线条内外两圈都会同时描边，呈现出像蚯蚓一样的“双轨道”怪异效果；
   - 只有选用纯实心、单路径或者纯线稿素材，描边效果才最为纯粹洗练；
2. **`forwards` 停留在终点**：
   - 动画执行完毕后，必须加上 `forwards`（填充模式），保证动画结束时实线图案完整停留在画面中，不会瞬移变回未绘制状态；
3. **线帽优化（`stroke-linecap: round`）**：
   - 在写复杂描边时，强烈建议声明 `stroke-linecap: round;`（圆角线帽），它能让线段起止点由生硬的平头变成圆润饱满的胶囊圆弧，手绘流畅感倍增。

> 💡 **承前启后**：完成对「115-SVG图片-描边效果以及仿北大官网效果」的理解后，下一章我们将深入探讨「116-SVG图片-利用偏移实现画心效果」，进一步完善知识图谱体系。

---

## 第 3 章：116-SVG图片-利用偏移实现画心效果
> 对应分集：P117 | 原始标题：《116-SVG图片-利用偏移实现画心效果》

在上一节中，我们通过动态修改 `stroke-dasharray` 的虚线比例，实现了北京大学官网的线条绘制效果。

而在现代大厂生产环境、商业 H5（如各类节日互动、告白动态、签约手写签名）中，还有一种**更加规范、也是全行业通用标准**的线条手绘实现方式——**基于 `stroke-dashoffset` 路径偏移量**。

本节我们将彻底搞懂 `stroke-dashoffset` 的空间位移原理，掌握通过原生 JavaScript `getTotalLength()` 精确测量任意复杂矢量路径周长的方法，并手把手实现经典 **动态手绘爱心（Draw Heart）动效**。

---

### 1. 核心原理：`stroke-dashoffset` 是如何工作的？

`offset` 英文直译为“偏移量”。在 SVG 描边中，`stroke-dashoffset` 专门用来**控制虚线与间隙相对于路径起点的滑动偏移距离**。

为了讲透这个原理，我们用一条长度为 $L$ 的封闭爱心路径来做演示：

```text
stroke-dashoffset 路径滑块模型 (假设路径总长 L = 2783px)
【第一步：固定 dasharray 等于整条路径的长度】
stroke-dasharray: 2783;
┌─────────────────────── 实线 2783px ───────────────────────┐┌──────── 空白间隙 2783px ────────┐

【第二步：初始状态设置 offset = 2783px (向左推出视口)】
stroke-dashoffset: 2783;
           ┌─────── 视口所能看到的爱心轮廓 ───────┐
[ 实线 2783px (全被推出视口外) ] | [ 空白间隙 2783px (占满视口) ] |
                                └──────────────────────────────────┘
                                结果：线条完全隐形不可见！

【第三步：动画补间让 offset 逐渐减少到 0px (实线顺滑拉回视口)】
stroke-dashoffset: 0;
           ┌─────── 视口所能看到的爱心轮廓 ───────┐
           | [ 实线 2783px (100% 完整覆盖爱心) ] |
           └──────────────────────────────────┘
           结果：爱心从起点顺着线条被一笔一划“画”出来！
```

#### 两种手绘方案的对比与选型：
- **方案 A（动态改 `dasharray`）**：不断改变实线和间隙的长度，适合简单单一直线；
- **方案 B（固定 `dasharray`，动态移动 `offset`）**：保持实线长度恒等于总周长，仅让整条线在轨道上滑动。**这是业界绝大多数 SVG 动画库（如 GSAP、Anime.js、Lottie）底层的标准工业规范**！

---

### 2. 核心神技：如何获取任意 SVG 路径的精确周长？

不同矢量图的大小、曲率千差万别，我们怎么知道这个爱心、小汽车或文字的路径总长到底是多少？靠肉眼猜是绝对不可能准确的。

pink 老师教给大家一个任何资深前端都必备的调试神技——**利用原生 JS 的 `getTotalLength()` API**：

#### 测量四步法：
1. 打开内联 SVG 代码，在要测量的 `<path>` 标签上添加一个唯一的 `id`（例如 `id="myHeart"`）：
   ```html
   <path id="myHeart" d="M512 896s..." />
   ```
2. 在浏览器中打开页面，按下 `F12` 打开开发者工具，切换到 **Console（控制台）** 面板；
3. 直接输入以下命令并回车：
   ```javascript
   myHeart.getTotalLength()
   ```
   *(注意驼峰命名：`getTotalLength`，末尾带小括号执行)*；
4. 控制台会立刻精确返回该图形的数学周长像素值！
   本案例实测返回值：**`2783.35`**，取整为 **`2783`**！

---

### 3. 完整代码实战开发

得到精确周长 `2783` 之后，代码编写便如同行云流水。

#### 3.1 HTML 结构
```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <title>SVG 利用偏移实现动态画心</title>
  <link rel="stylesheet" href="style.css">
</head>
<body>
  <div class="heart-box">
    <!-- 爱心矢量图标 -->
    <svg class="heart-svg" viewBox="0 0 1024 1024">
      <path
        id="myHeart"
        d="M512 896s-384-230.4-384-486.4c0-140.8 115.2-256 256-256 89.6 0 166.4 51.2 204.8 128 38.4-76.8 115.2-128 204.8-128 140.8 0 256 115.2 256 256C896 665.6 512 896 512 896z"
      />
    </svg>
    <p class="txt">把我的心、你的心、串一串</p>
  </div>
</body>
</html>
```

---

#### 3.2 CSS 核心样式编码

```css
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #1a1a24;
}

.heart-box {
  display: flex;
  flex-direction: column;
  align-items: center;
}

/* 1. 爱心 SVG 画布尺寸与基础描边 */
.heart-svg {
  width: 200px;
  height: 200px;
}

#myHeart {
  /* 镂空无填充 */
  fill: none;
  /* 浪漫玫瑰粉色描边，粗细 8px */
  stroke: #ff2d55;
  stroke-width: 8;
  /* 线帽圆润化，连接点平滑 */
  stroke-linecap: round;
  stroke-linejoin: round;

  /* 核心 A：虚线长度严格设置为实测周长 2783 */
  stroke-dasharray: 2783;
  /* 核心 B：初始偏移 2783（完全推空隐形） */
  stroke-dashoffset: 2783;

  /* 核心 C：绑定画心动画，3秒匀速无限循环手绘 */
  animation: drawHeart 3s linear infinite;
}

/* 2. 关键帧动画：让偏移量从 2783 衰减到 0 */
@keyframes drawHeart {
  0% {
    /* 初始：实线被完全推开在视口外 */
    stroke-dashoffset: 2783;
  }
  100% {
    /* 终点：实线完全拉回，轮廓完整闭合 */
    stroke-dashoffset: 0;
  }
}

.txt {
  margin-top: 24px;
  color: #ff758c;
  font-size: 16px;
  letter-spacing: 2px;
}
```

---

### 4. 关键认知升级与知识延伸

1. **正向画心 vs 逆向擦除**：
   - `stroke-dashoffset` 从 `2783` 变到 `0`：**线条被一笔画出（Drawing Effect）**；
   - `stroke-dashoffset` 从 `0` 变到 `2783`：**线条被顺次擦除消解（Erasing Effect）**；
2. **配合填充色延迟变现（先画线后填色）**：
   - 很多高端动效在画完线条的一瞬间，心形内部会立刻由透明渐变填满实心红色；
   - 可以在关键帧中加入 `fill` 属性补间：
     ```css
     @keyframes drawAndFill {
       0% { stroke-dashoffset: 2783; fill: transparent; }
       80% { stroke-dashoffset: 0; fill: transparent; }
       100% { stroke-dashoffset: 0; fill: #ff2d55; }
     }
     ```
   - 这样就能实现先 80% 时间手绘线稿，最后 20% 时间色彩充盈升华的顶级视觉！

> 💡 **承前启后**：完成对「116-SVG图片-利用偏移实现画心效果」的理解后，下一章我们将深入探讨「117-SVG图片-仿oppoAI里程碑案例」，进一步完善知识图谱体系。

---

## 第 4 章：117-SVG图片-仿oppoAI里程碑案例
> 对应分集：P118 | 原始标题：《117-SVG图片-仿oppoAI里程碑案例》

在现代科技品牌（如 OPPO、苹果、华为）的官方产品页中，我们经常能看到极具科技感的时间线与里程碑动效：当页面加载或用户向下滚动时，一条精致的曲线轨迹像光纤一样向前延展绘制；当轨迹延伸到特定年份或里程碑节点时，对应的信息卡片由隐至显、伴随微动效浮现出来。

这一讲我们把前面学过的 SVG 描边动画（`stroke-dasharray` 与 `stroke-dashoffset`）融入真实的商业级项目，完整推导并实现这一套纯 CSS 驱动的里程碑时间线交互体系。

---

### 1. 案例效果与结构层次拆解

#### 1.1 动效时序与视觉流程

很多初学者看到官网这种复杂特效，第一反应往往是“这必须写很多复杂的 JavaScript 甚至是 WebGL 吧？”其实抛开后期结合滚轮监听的交互触发，其核心渲染与骨架动效完全由纯 HTML5 + CSS3 承载。整个视觉流包含两个核心阶段：

1. **导轨描边阶段**：底层的 SVG 曲线路径从起点向终点持续匀速绘制，耗时约 10 秒；
2. **节点卡片交错进场阶段**：里程碑信息卡片初始处于隐藏状态（`opacity: 0`）。随着时间线延伸，当轨迹经过卡片所在物理位置时，卡片分别以 1 秒、2 秒的延迟时间（`animation-delay`）向上或向下浮现到位，并保持最终显示状态（`animation-fill-mode: forwards`）。

```text
+-------------------------------------------------------------------------+
| .box (固定视口宽度 1300px, overflow: auto)                              |
|                                                                         |
|  .scroll (横向超宽容器 3858px)                                           |
|  +-------------------------------------------------------------------+  |
|  |  [卡片1: 2020]         [卡片3: 2024 (向下浮入, delay: 2s)]       |  |
|  |       *                      *                                    |  |
|  |  ~~~~~*~~~~~~~~~~~~~~~~~~~~~~*~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~>  |  |
|  |       \                     /      (SVG 曲线路径: getTotalLength) |  |
|  |        \                   /                                      |  |
|  |         *                 /                                       |  |
|  |   [卡片2: 2022 (向上浮入, delay: 1s)]                             |  |
|  +-------------------------------------------------------------------+  |
+-------------------------------------------------------------------------+
```

> **读图说明**：外层 `.box` 充当可视窗口，内层 `.scroll` 承载超宽的 SVG 轨迹与定位卡片。SVG 负责横向路径描边，定位卡片根据轨迹落点交错执行纵向位移与渐变进入。

#### 1.2 商业页面矢量素材提取技巧

在日常前端开发中，复杂曲线并非由程序员手写坐标，而是由 UI 设计师在 Figma、Illustrator 等软件中导出，或直接从现存优秀案例中提取：

1. 打开目标官网（OPPO AI 官方介绍页），按 `F12` 或右键点击页面选择“检查”；
2. 在 Elements 面板中定位到承载曲线的 `<svg>` 节点；
3. 右键该节点，选择 `Copy` -> `Copy element`，整段矢量代码即完整进入剪贴板；
4. 粘贴到项目对应的 HTML 结构容器中。

> **提示**：矢量素材复制到本地后，务必检查 `<svg>` 内的核心 `<path>` 标签，为了后续能够精准施加样式和动效，我们通常手动为该路径添加语义化标识，例如 `id="line"`。

---

### 2. 布局骨架搭建与定位层级

#### 2.1 容器架构与尺寸契约

为了实现未来可水平拖动或滚动的效果，容器采用典型的“固定视口 + 超宽内容载体”结构：

```html
<div class="box">
  <div class="scroll">
    <!-- SVG 路径代码 -->
    <svg class="timeline-svg" viewBox="0 0 3858 300">
      <path id="line" d="M0 150 C 300 50, 600 250, 900 150 ..." fill="none" stroke="#00c070" stroke-width="4" />
    </svg>

    <!-- 里程碑卡片 1 -->
    <div class="item item1">
      <img src="images/star.png" alt="star" />
      <h5>2020 OPPO AI 实验室成立</h5>
      <p>启动全栈式端侧模型与云端协同研发体系布局。</p>
    </div>

    <!-- 里程碑卡片 2 -->
    <div class="item item2">
      <img src="images/star.png" alt="star" />
      <h5>2022 安第斯大模型正式发布</h5>
      <p>全面赋能移动终端，提供智慧感知与影像算法支持。</p>
    </div>

    <!-- 里程碑卡片 3 -->
    <div class="item item3">
      <img src="images/star.png" alt="star" />
      <h5>2024 AI 手机时代来临</h5>
      <p>OPPO AI 战略全面升级，生态赋能千万移动用户。</p>
    </div>
  </div>
</div>
```

#### 2.2 绝对定位与坐标排布

外层 `.box` 宽度定为 `1300px`，`.scroll` 容器的宽度则由 SVG 实际长度支撑或显式声明为 `100%`。每个 `.item` 卡片必须脱离常规文档流，通过绝对定位固定于曲线上对应的锚点旁边：

```css
/* 视口外层盒子 */
.box {
  width: 1300px;
  height: 400px;
  margin: 50px auto;
  border: 1px solid #e0e0e0;
  position: relative;
}

/* 超长内容滑轨 */
.box .scroll {
  width: 100%;
  height: 100%;
  position: relative;
  overflow: auto; /* 超出宽度时显示横向滚动条，不破坏页面整体排版 */
}

/* 里程碑卡片公共样式 */
.box .scroll .item {
  position: absolute;
  width: 240px;
  background-color: #ffffff;
  padding: 12px 16px;
  border-radius: 8px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.08);
  box-sizing: border-box;
}

.box .scroll .item img {
  width: 12px;
  height: 12px;
  vertical-align: middle;
}

.box .scroll .item h5 {
  font-size: 15px;
  margin: 6px 0 4px 0;
  color: #1a1a1a;
}

.box .scroll .item p {
  font-size: 12px;
  line-height: 1.5;
  color: #666666;
  margin: 0;
}
```

每个卡片的坐标根据设计稿与曲折点手工调优：
- `.item1`：位于起始平直段，`top: 80px; left: 100px;`（保持常驻显示，引导视觉）；
- `.item2`：位于曲线波谷下方，`top: 120px; left: 800px;`；
- `.item3`：位于曲线后方波峰上方，`top: 40px; left: 1600px;`。

---

### 3. 测量路径周长与描边机理

#### 3.1 JavaScript 控制台实测路径总长

做 SVG 描边动画最核心的物理参数是**路径总长度（Path Total Length）**。很多同学在这里盲目猜测数字，导致动画要么在末端提前中断，要么在线条还没跑完时动画时间就耗尽了。

正确姿势是借助浏览器原生 DOM API 进行精密求值：
1. 确保目标路径具有唯一选择器（如 `<path id="line">`）；
2. 打开 Chrome DevTools 的 Console 面板；
3. 敲入求长指令并回车：

```javascript
line.getTotalLength()
```

控制台当场返回浮点数值 `3857.41...`。

> **结论**：实测数值为 3857 时，CSS 中取值建议微调进位为 `3858px`。宁可多给一个像素，绝不能少给！如果写小了（例如写 3850px），虚线偏移后尾部会残留 7~8 像素的断头可见痕迹，无法实现真正的“从无到有”。

#### 3.2 虚线间隔与偏移原理

我们将 `3858px` 同时赋给虚线段长 `stroke-dasharray` 与偏移量 `stroke-dashoffset`：

```text
初始状态 (offset = 3858px):
+------------------------- 3858px -------------------------+
|                  全空段 (透明/不可见)                     |  实线段 (被推出视野)
+---------------------------------------------------------+

终止状态 (offset = 0px):
+------------------------- 3858px -------------------------+
|                   实线段 (完整铺满可视区)                  |
+---------------------------------------------------------+
```

通过把偏移量从 `3858px` 线性渐变递减至 `0`，实线部分被逐步“拉回”到视口路径之内，产生平滑的自主绘制光效。

```css
/* 明确指定 SVG 画布的物理尺寸 */
.box .scroll svg {
  width: 3858px;
  height: 100%;
}

/* 对具体路径施加描边参数 */
.box .scroll svg path {
  stroke-dasharray: 3858px;
  stroke-dashoffset: 3858px;
  animation: drawLine 10s linear forwards;
}

@keyframes drawLine {
  0% {
    stroke-dashoffset: 3858px;
  }
  100% {
    stroke-dashoffset: 0;
  }
}
```

---

### 4. 节点卡片交错进场动效与时间控制

#### 4.1 初始可见性与位移动画设计

卡片不能一开始就杵在页面上，否则时间线延伸过来就没有任何惊喜感与仪式感。
因此，后续卡片（`.item2`、`.item3`）必须预设 `opacity: 0`。

同时，为了让动效富有层次，我们为不同卡片制定不同的进场轨迹：
- **卡片 2**：从下方向上微移 `40px` 浮入；
- **卡片 3**：从上方微移 `40px` 下落浮入。

```css
/* 向上浮动的关键帧 */
@keyframes translateUp {
  0% {
    transform: translateY(0);
    opacity: 0;
  }
  100% {
    transform: translateY(-40px);
    opacity: 1;
  }
}

/* 向下浮动的关键帧 */
@keyframes translateDown {
  0% {
    transform: translateY(0);
    opacity: 0;
  }
  100% {
    transform: translateY(40px);
    opacity: 1;
  }
}
```

#### 4.2 动画双重时间参数解析

在 CSS 的 `animation` 复合属性中，经常出现两个连续的时间单位（如 `1s ... 1s` 或 `1s ... 2s`）。这里极易混淆，请务必牢记规范约定：

```css
animation: 动画名 持续时间(duration) 速度曲线(timing-function) 填充模式(fill-mode) 延迟时间(delay);
```

> **易错点**：当复合写法中出现两个时间值时，**排在前面的必定是执行持续时间（Duration），排在后面的必定是延迟等待时间（Delay）**！

```css
/* 初始隐藏卡片 2 与卡片 3 */
.box .scroll .item2,
.box .scroll .item3 {
  opacity: 0;
}

/* 卡片 2：等待线条跑了 1 秒后开始向上浮动，用时 1 秒，保持停在最后一帧 */
.box .scroll .item2 {
  top: 160px; /* 初始落位靠下 40px，上浮后到达 top: 120px */
  left: 800px;
  animation: translateUp 1s linear forwards 1s;
}

/* 卡片 3：等待线条跑了 2 秒后开始向下浮动，用时 1 秒，保持停在最后一帧 */
.box .scroll .item3 {
  top: 0px;   /* 初始落位靠上 40px，下沉后到达 top: 40px */
  left: 1600px;
  animation: translateDown 1s linear forwards 2s;
}
```

> **重点参数解析**：`forwards` 属性至关重要。若不加 `forwards`，卡片在执行完 1 秒动画后，会瞬间闪回第一帧状态（`opacity: 0`），导致卡片再次消失。`forwards` 强行锁定最后一帧样式，使透明度稳定在 `1`，位移固定在目标位置。

---

### 5. 局部滚动体验与完整整合工程

#### 5.1 视口内滚动优化

商业网站通常避免因为子模块过长而导致整个浏览器窗口横向被撑爆出现系统级滚动条。我们在 `.box .scroll` 上配置：

```css
overflow: auto;
```

当内容（`3858px` 的 SVG 轨迹与卡片）超过外层父盒子（`1300px`）时，滚动条只出现在 `.scroll` 内部，保持整体页面栅格严谨。

#### 5.2 完整代码示例

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>仿 OPPO AI 里程碑案例</title>
  <style>
    * {
      margin: 0;
      padding: 0;
      box-sizing: border-box;
    }

    body {
      background-color: #0b0d10;
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
      padding-top: 60px;
    }

    /* 视口舞台容器 */
    .box {
      width: 1300px;
      height: 420px;
      margin: 0 auto;
      border: 1px solid #23272e;
      border-radius: 12px;
      background-color: #14171d;
      position: relative;
    }

    /* 内部横向滑轨容器 */
    .box .scroll {
      width: 100%;
      height: 100%;
      position: relative;
      overflow: auto;
    }

    /* 隐藏原生粗糙滚动条，保留滑动手感 */
    .box .scroll::-webkit-scrollbar {
      height: 6px;
    }
    .box .scroll::-webkit-scrollbar-thumb {
      background: #2a313d;
      border-radius: 3px;
    }

    /* SVG 画布设置 */
    .box .scroll svg {
      width: 3858px;
      height: 100%;
      display: block;
    }

    /* 路径描边动画核心实现 */
    .box .scroll svg #line {
      stroke-dasharray: 3858px;
      stroke-dashoffset: 3858px;
      animation: lineDraw 10s linear forwards;
    }

    @keyframes lineDraw {
      0% {
        stroke-dashoffset: 3858px;
      }
      100% {
        stroke-dashoffset: 0;
      }
    }

    /* 里程碑卡片公共样式 */
    .box .scroll .item {
      position: absolute;
      width: 250px;
      background: rgba(255, 255, 255, 0.05);
      border: 1px solid rgba(255, 255, 255, 0.12);
      backdrop-filter: blur(10px);
      padding: 16px;
      border-radius: 10px;
      color: #ffffff;
    }

    .box .scroll .item .icon-box {
      display: inline-block;
      width: 12px;
      height: 12px;
      background-color: #00d084;
      border-radius: 50%;
      box-shadow: 0 0 8px #00d084;
    }

    .box .scroll .item h5 {
      font-size: 15px;
      margin: 8px 0 6px 0;
      color: #ffffff;
    }

    .box .scroll .item p {
      font-size: 12px;
      line-height: 1.6;
      color: #9aa0a6;
    }

    /* 卡片各自坐标与交错进场 */
    .box .scroll .item1 {
      top: 90px;
      left: 80px;
      /* 卡片 1 常驻展现 */
    }

    .box .scroll .item2 {
      top: 180px;
      left: 750px;
      opacity: 0;
      animation: moveUp 1s linear forwards 1s;
    }

    .box .scroll .item3 {
      top: 20px;
      left: 1550px;
      opacity: 0;
      animation: moveDown 1s linear forwards 2s;
    }

    @keyframes moveUp {
      0% {
        transform: translateY(0);
        opacity: 0;
      }
      100% {
        transform: translateY(-40px);
        opacity: 1;
      }
    }

    @keyframes moveDown {
      0% {
        transform: translateY(0);
        opacity: 0;
      }
      100% {
        transform: translateY(40px);
        opacity: 1;
      }
    }
  </style>
</head>
<body>

  <div class="box">
    <div class="scroll">
      <!-- 矢量线条 -->
      <svg viewBox="0 0 3858 400">
        <path id="line" d="M 0 200 Q 400 80, 800 200 T 1600 200 T 2400 200 T 3200 200 L 3858 200" 
              fill="none" stroke="#00d084" stroke-width="4" stroke-linecap="round" />
      </svg>

      <!-- 里程碑卡片 -->
      <div class="item item1">
        <span class="icon-box"></span>
        <h5>2020 · 成立 AI 核心战略部</h5>
        <p>确立以智慧交互与影像算力为核心的研发战略路线。</p>
      </div>

      <div class="item item2">
        <span class="icon-box"></span>
        <h5>2022 · 潘塔纳尔跨端生态</h5>
        <p>智慧感知与跨设备服务无缝流转，技术能力全面落地。</p>
      </div>

      <div class="item item3">
        <span class="icon-box"></span>
        <h5>2024 · 旗舰级生成式 AI 手机</h5>
        <p>安第斯大模型入驻移动端，重塑移动生产力与智慧生活。</p>
      </div>
    </div>
  </div>

</body>
</html>
```

---

### 6. 核心知识点与开发避坑总结

| 关注维度 | 关键技术实现 | 核心避坑指引 |
| :--- | :--- | :--- |
| **路径求长** | `line.getTotalLength()` | 必须在真实 DOM 节点渲染后调用；取整时向上微调 1px，切勿向下舍入。 |
| **描边隐藏** | `stroke-dasharray` + `stroke-dashoffset` | 两个属性必须设为相同的路径总长值，初始偏移让实线完全处于视线之外。 |
| **延时触发** | `animation: name 1s linear forwards 1s` | 明确区分持续时间与延迟时间，必须声明 `forwards` 保持最终展示状态。 |
| **滚动隔离** | `overflow: auto` 作用在超宽内层父级 | 避免超长 SVG（3858px）直接把主页面 body 撑宽造成横向布局溃散。 |

> 💡 **承前启后**：完成对「117-SVG图片-仿oppoAI里程碑案例」的理解后，下一章我们将深入探讨「118-SVG图片-无人机动画案例」，进一步完善知识图谱体系。

---

## 第 5 章：118-SVG图片-无人机动画案例
> 对应分集：P119 | 原始标题：《118-SVG图片-无人机动画案例》

在很多高端品牌官网（如大疆 DJI、科技教育网站、创意设计主页）中，我们经常能看到令人惊叹的插画交互：

一幅原本静止的矢量都市插画中，天上的无人机在云端忽高忽低地盘旋巡航，远处的风筝在微风中轻微摇曳，地面的人偶仿佛在眺望天际。

很多初学者看到这种效果，以为前端工程师必须让设计师切出几十张零散的 PNG 小图，再用绝对定位一个一个拼回去。

pink 老师告诉大家：**大错特错！在现代 SVG 工作流中，整张图就是一个完整的 `.svg` 文件！**

本节我们将彻底解密 **SVG 内部元素分组 `<g>`（Group）标签** 的运行机制，学习如何给复合 SVG 的局部组件施加独立动效，并攻克 **无人机天空巡航案例**。

---

### 1. 动效解构：复合 SVG 的“积木”世界

很多同学以前把 SVG 当作普通图片（`<img>`）来理解，以为它是一个无法拆解的整体。

但只要右键“查看网页源代码”就会发现：**SVG 本质上是一个具备完整 DOM 树的微型画布（Mini-DOM）**！

```text
复合 SVG 的 DOM 树结构剖析
<svg viewBox="0 0 1200 800">
├── <g id="background"> (背景草地、远山、云朵)
├── <g id="kite">       (风筝组件，包含骨架与飘带)
├── <g id="people">     (地面人物，包含身体与肢体)
└── <g id="drone">      (核心：无人机独立分组！)
    ├── <path d="..." class="blade" /> (螺旋桨)
    ├── <path d="..." class="body" />  (机身)
    └── <path d="..." class="light" /> (指示灯)
</svg>
```

#### 核心标签解析：`<g>`（Group）
- `<g>` 是 SVG 规范中的“分组容器”，等价于普通 HTML 中的 `<div>`；
- 设计师在 Figma 或 Illustrator 中编组的图层，导出为 SVG 时就会转化为对应的 `<g>` 标签；
- **只要我们为某个 `<g>` 添加 `class` 或 `id`，外部 CSS 就能像操作普通 `div` 一样，单独对这个局部组件进行缩放、旋转或位移动画！**

---

### 2. 核心技术避坑：`overflow: visible`

在为 SVG 内部组件添加位移动效时，几乎 100% 的开发者都会遇到这个诡异现象：
> **“我的无人机往上飞着飞着，突然像被一把无形的剪刀切掉了一半，再飞一点就彻底消失了！”**

#### 原因剖析：
- SVG 规范默认对画布之外的内容进行硬裁剪（隐式自带 `overflow: hidden` 特性）；
- 当无人机的航线超过了 `<svg>` 画布所规定的 `width/height` 视口边界时，飞出的部分就会被强行截断遮蔽！

#### 黄金解决方案：
必须在外部 CSS 中显式声明：
```css
svg {
  /* 核心：允许 SVG 内部的矢量组件飞出画布范围，依然保持可见 */
  overflow: visible;
}
```

---

### 3. 完整代码实战开发

下面我们以无人机巡航插画为蓝本，演示如何直接抓取内联 SVG 中的核心 `<g>` 分组并赋予其自由巡航动力。

#### 3.1 HTML 结构
```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <title>SVG 复合分组动画：无人机巡航</title>
  <link rel="stylesheet" href="style.css">
</head>
<body>
  <div class="stage">
    <!-- 复合矢量插画画布 -->
    <svg class="scene-svg" viewBox="0 0 1000 600">
      <!-- 1. 背景元素层 (静止不动) -->
      <g id="landscape">
        <path d="M0 500 Q 250 450, 500 500 T 1000 500 L 1000 600 L 0 600 Z" fill="#78c257" />
      </g>
      
      <!-- 2. 地面人物层 (静止不动) -->
      <g id="character">
        <circle cx="200" cy="460" r="16" fill="#fdbf6f" />
        <path d="M185 480 h30 v60 h-30 z" fill="#1f78b4" />
      </g>

      <!-- 3. 核心动态层：无人机独立分组 -->
      <g id="drone">
        <!-- 机身机翼几何矢量 -->
        <path d="M50 0 h100 v15 h-100 z" fill="#2c3e50" />
        <circle cx="100" cy="15" r="18" fill="#e74c3c" />
        <!-- 左右螺旋桨 -->
        <ellipse cx="45" cy="-2" rx="20" ry="4" fill="#95a5a6" />
        <ellipse cx="155" cy="-2" rx="20" ry="4" fill="#95a5a6" />
      </g>
    </svg>
  </div>
</body>
</html>
```

---

#### 3.2 CSS 核心样式编码

```css
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #ecf0f1;
}

.stage {
  width: 900px;
  height: 540px;
  background: linear-gradient(to bottom, #d4f1f9 0%, #eef9fc 100%);
  border-radius: 20px;
  box-shadow: 0 16px 40px rgba(0, 0, 0, 0.08);
  display: flex;
  justify-content: center;
  align-items: center;
}

/* 1. SVG 主画布样式 */
.scene-svg {
  width: 800px;
  height: 480px;
  /* 核心属性：允许飞出边界 */
  overflow: visible;
}

/* =======================================================
   2. 锁定无人机分组，注入独立飞行动效
   ======================================================= */
#drone {
  /* 12秒平滑多点循环巡航 */
  animation: droneCruise 12s ease-in-out infinite;
}

/* 3. 多航点自由飞行轨迹 (结合平移与微倾斜) */
@keyframes droneCruise {
  0% {
    /* 初始停泊位置 */
    transform: translate(150px, 180px) rotate(0deg);
  }
  25% {
    /* 爬升并加速前行，机身前倾 5° */
    transform: translate(320px, 80px) rotate(5deg);
  }
  50% {
    /* 悬停侦查，机身回正 */
    transform: translate(520px, 140px) rotate(0deg);
  }
  75% {
    /* 转向返航，机身反向倾斜 -6° */
    transform: translate(360px, 220px) rotate(-6deg);
  }
  100% {
    /* 回到起始航点，无缝闭环 */
    transform: translate(150px, 180px) rotate(0deg);
  }
}
```

---

### 4. 实战总结与工程认知升级

1. **从“切图仔”到“动效架构师”的跃迁**：
   - 传统思维：找设计师要图片切片；
   - 现代工程思维：让设计师给整份 SVG 文件，确保核心交互组件都在独立的 `<g>` 分组内；前端直接利用 CSS 选择器操作 `#drone`、`#blade` 或 `#cloud`；
2. **多层协同微动效**：
   - 不仅机身整体可以做位移，机身内部的螺旋桨（`ellipse`）还可以单独绑定 `rotate(360deg)` 极速旋转，配合指示灯的闪烁动画，整台无人机将变得无比生动逼真；
3. **SVG DOM 事件响应**：
   - 因为 `<g>` 标签是标准 DOM 节点，你还可以直接在 CSS 中书写 `#drone:hover`，或者在 JS 中给 `document.getElementById('drone').addEventListener('click')` 添加交互事件，可玩性与架构灵活性极其广阔！

> 💡 **承前启后**：完成对「118-SVG图片-无人机动画案例」的理解后，下一章我们将深入探讨「119-clip-path裁剪-基本语法使用」，进一步完善知识图谱体系。

---

## 第 6 章：119-clip-path裁剪-基本语法使用
> 对应分集：P120 | 原始标题：《119-clip-path裁剪-基本语法使用》

你们有没有遇到过这种无奈：我想做一个菱形头像、一个箭头按钮、一个六边形标签，翻遍 `border-radius` 也只能搓出圆角，再复杂的形状就得求设计师切图？

我以前也是这样，直到我学会 `clip-path`，才发现原来任意多边形、圆形、椭圆，前端自己一行 CSS 就能裁出来，而且裁掉的部分连点击事件都不响应。今天我们就把它的基本语法一次吃透。

### 1. clip-path 的裁剪模型

> **定义**：裁剪（`clip-path`）是在元素原有的盒子之上再蒙一层剪纸框，只有落在框里的像素会被保留并渲染，框外的像素直接丢弃，不占视觉、不响应鼠标。

```text
+----------------------+
|  原始盒子 (200x200)  |
|   +----------------+ |
|   |  clip-path 框  | |
|   |  保留显示      | |
|   +----------------+ |
|  框外丢弃,事件穿透   |
+----------------------+
```

读图说明：外层是元素原本的盒子，内层是裁剪路径，只有交集部分可见。

我刚学的时候总把它和 `overflow: hidden` 搞混，记住这一条就够了：`overflow` 是拿父盒子去藏子元素，而 `clip-path` 是拿一把剪刀直接剪自己，连自己都能剪成三角形。

> **提示**：被裁掉的区域真的消失了，把鼠标放上去不会触发 `hover`，这和用遮罩盖住是两回事。

基础写法只有一行：

```css
.box {
  width: 200px;
  height: 200px;
  background: pink;
  /* 裁成一个圆,后面会细讲参数 */
  clip-path: circle(50% at 50% 50%);
}
```

### 2. 圆形与椭圆裁剪

#### 2.1 圆形裁剪 circle

`circle()` 是最常用的入门函数，完整形态是 `circle(半径 at 圆心x 圆心y)`。

```css
/* 半径 50%,圆心在盒子正中心:正好内切 */
.avatar1 { clip-path: circle(50% at 50% 50%); }
/* 半径 60px,圆心偏右上:裁出月牙式错位圆 */
.avatar2 { clip-path: circle(60px at 70% 30%); }
```

半径写百分比时，参照的是盒子宽高计算出的特定基准，你们记住结论就行：`50%` 基本就是内切圆，想大一点就调大数字。圆心默认是 `50% 50%`，`at` 后面不写就居中。

我习惯拿它做头像：正方形 `img` 直接裁成圆形头像，比套一层 `border-radius: 50%` 更彻底，因为后者只是把角变圆，前者是把角删掉。

#### 2.2 椭圆裁剪 ellipse

椭圆就是把圆的横竖半径拆开写：`ellipse(横半径 纵半径 at 圆心)`。

```css
.banner {
  width: 400px;
  height: 200px;
  background: linear-gradient(135deg, #ff9a9e, #fecfef);
  /* 横向 200px,纵向 100px 的椭圆 */
  clip-path: ellipse(200px 100px at 50% 50%);
}
```

为什么是两个半径？你们想想，椭圆本来就是横向拉长或纵向压扁的圆，横半径管多宽，纵半径管多高。做那种横幅底部微微弧形的吸底效果，椭圆一裁就有味道了。

### 3. 内矩形裁剪 inset

`inset()` 是从四条边往里缩，语法和 `margin` 的方向一模一样：`inset(上 右 下 左)`，还可以带圆角。

```css
/* 四周各向内缩 20px,相当于留边裁 */
.card { clip-path: inset(20px); }
/* 只裁掉右侧一半:做文字扫描的底子,下一节会用到 */
.scan { clip-path: inset(0 50% 0 0); }
/* 内缩矩形再加 10px 圆角 */
.btn { clip-path: inset(0 round 10px); }
```

> **易错点**：`inset(20px)` 不是向外扩 20px，而是向内缩 20px，可见区变小了。方向记反，裁出来的和想要的正好相反。

我最喜欢用 `inset` 做的一件事是按钮 hover 展开：默认 `inset(0 100% 0 0)` 把背景藏起来，鼠标放上去过渡到 `inset(0 0 0 0)`，背景就像从左往右流进来一样，比改 `width` 流畅得多。

### 4. 多边形裁剪 polygon

前面三个都是规则形状，真正的大杀器是 `polygon()`：你给几个点，我就连成什么形。

```css
/* 三角形:上中、下左、下右三个点 */
.triangle {
  clip-path: polygon(50% 0, 0 100%, 100% 100%);
}
/* 菱形 */
.diamond {
  clip-path: polygon(50% 0, 100% 50%, 50% 100%, 0 50%);
}
/* 右箭头 */
.arrow {
  clip-path: polygon(0 0, 70% 0, 100% 50%, 70% 100%, 0 100%, 30% 50%);
}
```

坐标系一定要先立住：

```text
(0,0) ---------- (100%,0)
  |                  |
  |   点都落在        |
  |   这张百分比      |
  |   坐标纸上        |
  |                  |
(0,100%) ------ (100%,100%)
```

读图说明：左上角是原点，横轴向右、纵轴向下都是 `0~100%`，所有点都按这个坐标纸落子。

你们数一数，我上面菱形写了 4 个点，箭头写了 6 个点，浏览器会按顺序连线，最后自动闭合。点越多，形状越自由，六边形标签、五角星、不规则飘带都能手写。

> **提示**：手算多边形坐标太折磨人，我平时直接用 clippy 在线生成器拖拽出形状，再把那一行 `polygon(...)` 复制回来，自己微调数字就行。记住工具，但原理必须懂，不然复制回来的点你一个都看不懂。

| 函数 | 参数形态 | 典型用途 |
| :--- | :--- | :--- |
| `circle()` | `circle(半径 at x y)` | 圆形头像、热点圆 |
| `ellipse()` | `ellipse(横 纵 at x y)` | 椭圆横幅、弧形底部 |
| `inset()` | `inset(上 右 下 左 round 圆角)` | 矩形内缩、扫描显隐 |
| `polygon()` | `polygon(x1 y1, x2 y2, ...)` | 三角形、菱形、箭头、六边形 |

### 5. 裁剪的过渡与调试习惯

`clip-path` 是可以过渡和做动画的，前提是首尾两帧的函数类型和点的数量一致，浏览器才能插值。

```css
.box {
  width: 200px;
  height: 200px;
  background: #ff6b81;
  clip-path: circle(20% at 50% 50%);
  transition: clip-path 0.5s ease;
}
.box:hover {
  clip-path: circle(70% at 50% 50%);
}
```

鼠标放上去，圆从小变大，图片就像从中心晕开一样。你们去试，圆变圆可以，圆变多边形不行，点数对不上浏览器就直接跳变，没有中间帧。

我调试时有两个固定习惯：先给元素一个鲜艳的背景色，确认盒子本身的位置尺寸没问题，再上 `clip-path`；裁完发现全没了，八成是坐标写反或点连成了自交图形，回头一个点一个点检查。

> 💡 **承前启后**：完成对「119-clip-path裁剪-基本语法使用」的理解后，下一章我们将深入探讨「120-clip-path裁剪-灯光扫描文字效果」，进一步完善知识图谱体系。

---

## 第 7 章：120-clip-path裁剪-灯光扫描文字效果
> 对应分集：P121 | 原始标题：《120-clip-path裁剪-灯光扫描文字效果》

黑底上有一行灰色大字，一道白色灯光从左往右扫过去，被扫到的字瞬间变亮，扫过去又暗回去，像机场跑道灯一样来回跑。你们在很多科技官网的首页都见过，我第一眼就想：这是 Canvas 做的吧？

其实不用那么重，两层同样的文字叠在一起，上面那层用 `clip-path` 开一扇会动的窗，灯光效果就出来了。上一节学的 `inset` 终于派上用场。

### 1. 双层文字的叠放结构

> **结论**：底层放暗色文字做底，顶层放亮色文字做光，两层文字内容、字号、位置完全重合，动的是顶层那扇裁剪窗。

```text
+----------------------------------+
|  .wrap (relative)                |
|  +------------------------------+|
|  | 底层 h1: #333 灰字,静止       ||
|  +------------------------------+|
|  +------------------------------+|
|  |顶层 h1: 白色+发光,absolute盖住 ||
|  |只露 clip-path 窗内部分        ||
|  +------------------------------+|
+----------------------------------+
```

读图说明：容器相对定位，两层文字精确重叠，顶层绝对定位盖住底层，可见部分完全由裁剪窗决定。

```html
<div class="wrap">
  <h1 class="base">HELLO WORLD</h1>
  <h1 class="light">HELLO WORLD</h1>
</div>
```

```css
* { margin: 0; padding: 0; box-sizing: border-box; }

body {
  background: #000;
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
}

.wrap {
  position: relative;
  font-size: 60px;
  font-weight: 700;
  letter-spacing: 4px;
}

.wrap h1 {
  font-size: inherit;
  white-space: nowrap;
}

/* 底层:暗灰色底字 */
.base {
  color: #333;
}

/* 顶层:亮白色光字,精确盖住底层 */
.light {
  position: absolute;
  left: 0;
  top: 0;
  color: #fff;
  text-shadow: 0 0 10px #fff, 0 0 30px #0ff;
  /* 初始只露出最左侧一条缝 */
  clip-path: inset(0 100% 0 0);
  animation: scan 3s linear infinite;
}
```

你们注意，`.light` 必须 `left: 0; top: 0` 和底层严丝合缝，差 1px 都会重影。`white-space: nowrap` 也别漏，不然两行文字换行位置不一致，光就对不齐了。

> **易错点**：顶层文字一定要和底层共用同一套字体、字号、字距，我一般把字号写在 `.wrap` 上，两层 `inherit` 继承，改一处就全改，不会错位。

### 2. 裁剪窗口的扫描动画

核心就是让 `inset` 的右裁剪量从 `100%` 跑到 `0`，窗口从一条缝张大到全露，再收回去。

```css
@keyframes scan {
  0% {
    clip-path: inset(0 100% 0 0);
  }
  50% {
    clip-path: inset(0 0 0 0);
  }
  100% {
    clip-path: inset(0 0 0 100%);
  }
}
```

前一半灯光从左往右展开，后一半从左侧收起，合起来就是一道光扫过整行字。你们把时间线捋一下：`0%` 右边全裁掉只剩左边一条缝，`50%` 全展开整行都亮，`100%` 左边全裁掉只剩右边一条缝，光跑到头了。

想让光束带一点斜切的科技感，把矩形窗换成斜角多边形就行：

```css
@keyframes scanSlant {
  0% {
    clip-path: polygon(-10% 0, 0 0, -10% 100%, -20% 100%);
  }
  100% {
    clip-path: polygon(110% 0, 120% 0, 110% 100%, 100% 100%);
  }
}
```

这是一个宽度固定的斜平行四边形，从左侧外部一路平移到右侧外部，所过之处的字被点亮。我第一次试的时候直接写 `translateX` 去移整层字，结果整行字跟着跑，记住：移的是裁剪窗，不是字，字一动都不能动。

> **提示**：`clip-path` 做动画时首尾函数类型必须一致，`inset` 配 `inset`，`polygon` 配 `polygon`，且 `polygon` 的点数要相等，否则浏览器无法插值，会直接闪切一下。

### 3. 发光质感的调参

光扫过去了，亮得好不好看，全看顶层那几个属性：

```css
.light {
  color: #fff;
  /* 两层光晕:近处锐,远处 diffuse */
  text-shadow:
    0 0 8px rgba(255, 255, 255, 0.9),
    0 0 25px rgba(0, 255, 255, 0.8);
}
```

`color` 决定字母本体的亮度，`text-shadow` 决定光晕。底色是纯黑时，白色本体加青色外晕最出效果；底色偏蓝，就把光晕换成浅蓝。动画时长 `3s` 配 `linear` 最稳，`ease` 会让光在两头减速，像犹豫了一下，不够干脆。

| 结构层 | 角色 | 关键属性 |
| :--- | :--- | :--- |
| `.wrap` | 定位基准与字号源头 | `position: relative` |
| `.base` | 暗色底字，常亮 | `color: #333` |
| `.light` | 亮色光字，会动 | `position: absolute; clip-path; animation` |

> 💡 **承前启后**：完成对「120-clip-path裁剪-灯光扫描文字效果」的理解后，下一章我们将深入探讨「121-turn旋转一圈-仿华为官网花瓣旋转效果」，进一步完善知识图谱体系。

---

## 第 8 章：121-turn旋转一圈-仿华为官网花瓣旋转效果
> 对应分集：P122 | 原始标题：《121-turn旋转一圈-仿华为官网花瓣旋转效果》

`0deg` 到 `360deg` 转一圈，这种写法你们闭着眼都会。但我问你们：转三圈半怎么写？`1260deg`，每次都要按计算器，是不是很别扭？

CSS 里其实藏了一个专为整圈旋转准备的单位 `turn`，`1turn` 就是一圈。今天我们就用它复刻那种官网常见的花瓣缓缓旋转的效果，顺便把圈数表达彻底搞利索。

### 1. turn 圈数单位

> **定义**：圈数（`turn`）是旋转角度的圈数单位，`1turn = 360deg = 2πrad = 400grad`，数值上就是转了多少圈。

```css
/* 以下四行效果完全相同,都是转一圈 */
.a { transform: rotate(360deg); }
.b { transform: rotate(1turn); }
.c { transform: rotate(6.2832rad); }
.d { transform: rotate(400grad); }
```

你们看，表达一圈时 `1turn` 最直观，表达半圈时 `0.5turn` 就是 `180deg`，表达三圈半直接写 `3.5turn`，一眼就知道转了几圈，完全不用心算度数。

```text
0turn ---- 0.25turn ---- 0.5turn ---- 0.75turn ---- 1turn
 0deg       90deg        180deg        270deg       360deg
  |            |            |             |            |
 初始        右转90     倒过来       再转90      回到起点
```

读图说明：圈数和度数一一对应，`turn` 的小数部分直接对应圈内的相位。

> **提示**：`turn` 只能用在旋转相关的属性里，比如 `transform: rotate()` 和动画关键帧中的角度，写在别处浏览器不认。

### 2. 花瓣盘的静态布局

我们要做的是一朵由 8 片花瓣围成的花，整朵花放在页面正中缓缓自转。结构很简单：一个圆盘容器，里面 8 个同样的花瓣。

```html
<div class="flower">
  <span></span>
  <span></span>
  <span></span>
  <span></span>
  <span></span>
  <span></span>
  <span></span>
  <span></span>
</div>
```

```css
* { margin: 0; padding: 0; box-sizing: border-box; }

body {
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  background: #0b0e1a;
}

.flower {
  position: relative;
  width: 300px;
  height: 300px;
  /* 整盘缓转,下面细讲 */
  animation: spin 12s linear infinite;
}

/* 单片花瓣:顶部收窄的椭圆 */
.flower span {
  position: absolute;
  left: 50%;
  top: 50%;
  width: 70px;
  height: 130px;
  margin-left: -35px;
  margin-top: -130px;
  background: linear-gradient(to top, #ff6b81, #ff9ff3);
  border-radius: 50% 50% 50% 50% / 60% 60% 40% 40%;
  opacity: 0.85;
  /* 每片以盘心为轴,先摆好角度 */
  transform-origin: 50% 130px;
}
```

关键在 `transform-origin: 50% 130px`，意思是旋转轴不在花瓣自己的中心，而在它正下方 `130px` 处，也就是圆盘的盘心。8 片花瓣都以盘心为轴，摆开不同角度，自然围成一圈。

8 个角度手写太啰嗦，我用变量加 `nth-child` 一次摆好：

```css
.flower span:nth-child(1) { transform: rotate(0turn); }
.flower span:nth-child(2) { transform: rotate(0.125turn); }
.flower span:nth-child(3) { transform: rotate(0.25turn); }
.flower span:nth-child(4) { transform: rotate(0.375turn); }
.flower span:nth-child(5) { transform: rotate(0.5turn); }
.flower span:nth-child(6) { transform: rotate(0.625turn); }
.flower span:nth-child(7) { transform: rotate(0.75turn); }
.flower span:nth-child(8) { transform: rotate(0.875turn); }
```

你们看用 `turn` 写等分多爽：8 片就是每次加 `0.125turn`，12 片就是每次加约 `0.0833turn`，用 `deg` 写就得 `45deg、90deg` 一路算上去。用圈数思考等分，比用度数思考等分更顺。

### 3. 整盘旋转的动画

静态花摆好了，让整盘转起来只需要一个关键帧：

```css
@keyframes spin {
  0% {
    transform: rotate(0turn);
  }
  100% {
    transform: rotate(1turn);
  }
}
```

从 `0turn` 到 `1turn`，不快不慢转一圈，配上 `linear infinite` 就是匀速永动。你们想转两圈就写 `2turn`，想反转让 `-1turn`，语义直接摆在脸上。

> **易错点**：花瓣自身的 `rotate` 和花盘的 `rotate` 不冲突，前者摆在 `span` 上管摆位，后者摆在 `.flower` 上管自转。千万别把自转动画写到 `span` 上，那样每片花瓣原地打转，花就散架了。

```text
        盘心(旋转轴)
          +
    \     |     /
     \  花瓣1  /
      \       /
   花瓣8 + + 花瓣2
      /       \
     /  花瓣... \
    /     |     \
        花瓣5
   整盘绕 + 匀速转 1turn
```

读图说明：8 片花瓣以盘心为轴均匀分布，动画旋转的是外层容器，所有花瓣跟着公转。

如果觉得转得太机械，可以叠第二层反向慢转的花盘，两层花瓣错开 `0.0625turn`，一正一反，层次感立刻出来。原理不变，还是 `turn` 加关键帧，只是多复制了一份结构。

> 💡 **承前启后**：完成对「121-turn旋转一圈-仿华为官网花瓣旋转效果」的理解后，下一章我们将深入探讨「122-滤镜filter的使用」，进一步完善知识图谱体系。

---

## 第 9 章：122-滤镜filter的使用
> 对应分集：P123 | 原始标题：《122-滤镜filter的使用》

手机修图软件里一键变灰、变亮、变模糊的那种滤镜，CSS 里一行就能实现，而且作用对象可以是图片、文字、整个盒子，甚至鼠标放上去还能平滑过渡。

我第一次知道 `filter` 的时候，第一反应是：以后做图片置灰、悬停变彩这种需求，再也不用找设计师切两张图了。

### 1. filter 的作用模型

> **定义**：滤镜（`filter`）是对元素自身渲染结果再做一次像素级加工，写在谁身上，谁整体的外观就变，子元素跟着一起变。

```text
原始元素渲染 --> filter 加工 --> 显示到页面
   彩色图    --> grayscale(1) --> 灰度图
   清晰图    --> blur(5px)    --> 模糊图
   普通盒子  --> drop-shadow  --> 带投影盒子
```

读图说明：滤镜是渲染管线末端的一道工序，元素先画好，再整体过一遍滤镜。

```css
img {
  width: 300px;
  /* 默认灰掉,悬停恢复彩色 */
  filter: grayscale(1);
  transition: filter 0.5s ease;
}
img:hover {
  filter: grayscale(0);
}
```

> **提示**：`filter` 会作用于整个元素，里面的文字、边框、背景一个都跑不掉。只想动背景而不动文字，后面背景滤镜那一节才是正解，这里别用错。

### 2. 常用滤镜函数

#### 2.1 灰度与怀旧

```css
/* 灰度:0 原色,1 全灰 */
.gray { filter: grayscale(1); }
/* 怀旧:0 原色,1 全黄褐 */
.sepia { filter: sepia(0.8); }
```

`grayscale(1)` 是全灰，`grayscale(0.5)` 是半灰，取值就是灰的程度。`sepia` 是老照片那种黄褐色，两个经常叠着用，先灰后黄，旧照片味一下就出来了。

#### 2.2 亮度、对比度与饱和度

```css
/* 亮度:1 原样,大于1变亮,小于1变暗 */
.bright { filter: brightness(1.5); }
/* 对比度:1 原样,越大黑白越分明 */
.contrast { filter: contrast(1.4); }
/* 饱和度:1 原样,0 全灰,大于1更浓艳 */
.saturate { filter: saturate(1.8); }
```

这三个的基准都是 `1`，`1` 就是不动。你们记一个口诀：大于 `1` 往浓里走，小于 `1` 往淡里走。调产品图时 `brightness(1.1) saturate(1.2)` 轻轻一叠，图片通透不少。

#### 2.3 模糊与色相旋转

```css
/* 高斯模糊,半径越大越糊 */
.blur { filter: blur(5px); }
/* 色相旋转,0~360deg 转一圈回到原色 */
.hue { filter: hue-rotate(90deg); }
/* 反色:0 原色,1 全反 */
.invert { filter: invert(1); }
/* 透明度:和 opacity 类似,但可参与滤镜叠加 */
.fade { filter: opacity(0.5); }
```

`blur(5px)` 注意单位是长度不是百分比。`hue-rotate` 最好玩，`90deg` 直接把红色拧成绿色，做主题一键换色经常靠它。

多个函数可以空格连写，按从左到右依次加工：

```css
.poster {
  filter: contrast(1.2) saturate(1.4) brightness(1.05);
}
```

| 函数 | 取值 | 基准与含义 |
| :--- | :--- | :--- |
| `grayscale()` | `0~1` | `0` 彩色，`1` 全灰 |
| `sepia()` | `0~1` | `1` 全怀旧黄褐 |
| `brightness()` | `0~` | `1` 原样，大于变亮 |
| `contrast()` | `0~` | `1` 原样，大于更分明 |
| `saturate()` | `0~` | `1` 原样，`0` 等于灰度 |
| `blur()` | 长度 | 半径越大越模糊 |
| `hue-rotate()` | 角度 | 色轮旋转，`360deg` 回原色 |
| `invert()` | `0~1` | `1` 全反色 |
| `opacity()` | `0~1` | 元素整体透明度 |

### 3. 投影滤镜 drop-shadow

```css
/* 横偏移 竖偏移 模糊半径 颜色 */
.icon {
  filter: drop-shadow(4px 4px 6px rgba(0, 0, 0, 0.4));
}
```

你们肯定要问：这和 `box-shadow` 有什么区别？记住这个画面：`box-shadow` 给盒子加影子，永远是方的；`drop-shadow` 给不透明像素加影子，镂空图标、异形 PNG 的影子会跟着轮廓走。

```text
box-shadow:      +--------+
                 | 图片盒 |::::影子(方形)
                 +--------+

drop-shadow:        /\
                   /  \::::影子(跟着三角形轮廓)
                  /____\
```

读图说明：方形图两者差不多，异形图 `drop-shadow` 的影子才是贴合形状的。

我做异形按钮、镂空图标时，投影一律用 `drop-shadow`，方方正正的卡片才用 `box-shadow`。两者还能叠加，但别在同一个元素上反复套，性能会吃紧。

### 4. 悬停切换的完整写法

```css
.card img {
  width: 100%;
  filter: grayscale(1) brightness(0.9);
  transition: filter 0.4s ease;
}
.card:hover img {
  filter: grayscale(0) brightness(1);
}
```

默认灰一点暗一点，鼠标放上去恢复彩色，`transition` 把 `filter` 的变化抹平滑。你们去试，滤镜过渡天生就是平滑的，不需要逐帧写关键帧。注意别把过渡写在 `hover` 那一行，要写在常态上，去和回才都有动画。

> 💡 **承前启后**：完成对「122-滤镜filter的使用」的理解后，下一章我们将深入探讨「123-背景滤镜backdrop-filter」，进一步完善知识图谱体系。

---

## 第 10 章：123-背景滤镜backdrop-filter
> 对应分集：P124 | 原始标题：《123-背景滤镜backdrop-filter》

上一节的 `filter` 是加工元素自己，这一节的 `backdrop-filter` 是加工元素背后的东西。只差两个字，效果天差地别：一个是给照片加滤镜，一个是隔着毛玻璃看风景。

你们手机控制中心那种半透明模糊面板、网站导航栏滚动时底下内容隐约透上来，都是它做的。我第一次亲手写出毛玻璃导航时，来回滚动页面玩了半天。

### 1. 背景滤镜的作用模型

> **定义**：背景滤镜（`backdrop-filter`）是对元素背后已经画好的内容再做像素加工，元素自己动不动，全看你给不给它加普通 `filter`。

```text
页面已渲染内容(图片/文字)
        |
        v
+-------------------+
| 半透明面板         |
| backdrop-filter:  |
|   blur(10px)      | --> 背后内容变模糊,面板文字保持清晰
+-------------------+
        |
        v
    显示到屏幕
```

读图说明：滤镜发生在面板这块区域，但加工对象是面板盖住的那片背景，面板自己的文字不受影响。

最小可用写法三件套缺一不可：

```css
.nav {
  /* 1. 必须半透明,全不透明就看不见背后了 */
  background: rgba(255, 255, 255, 0.3);
  /* 2. 背景滤镜:背后模糊 10px */
  backdrop-filter: blur(10px);
  -webkit-backdrop-filter: blur(10px);
}
```

> **易错点**：背景写成实色 `background: #fff`，背后全被盖住，`backdrop-filter` 写得再对也看不出任何变化。我排查毛玻璃失效，十次有九次是背景不透明。

| 特性 | `filter` | `backdrop-filter` |
| :--- | :--- | :--- |
| 加工对象 | 元素自身及子元素 | 元素背后的页面内容 |
| 面板自身文字 | 跟着变模糊变灰 | 保持清晰 |
| 是否需要半透明背景 | 不需要 | 需要，否则看不见效果 |
| 典型用途 | 图片置灰、悬停变彩 | 毛玻璃导航、弹窗、卡片 |

### 2. 毛玻璃导航栏制作

```html
<div class="page">
  <div class="hero"></div>
  <nav class="nav">毛玻璃导航栏</nav>
</div>
```

```css
* { margin: 0; padding: 0; box-sizing: border-box; }

.page {
  height: 200vh;
}

.hero {
  height: 400px;
  background: url("banner.jpg") center / cover no-repeat;
}

.nav {
  position: sticky;
  top: 0;
  height: 60px;
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 18px;
  color: #333;
  /* 半透明白底 + 背后模糊 */
  background: rgba(255, 255, 255, 0.4);
  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
  border-bottom: 1px solid rgba(255, 255, 255, 0.5);
}
```

页面往下滚，图片从导航底下经过时自动变模糊，导航上的字干干净净。`position: sticky` 让导航吸在顶部，`blur(12px)` 是我试下来最舒服的数值：`5px` 以下几乎看不出，`20px` 以上背后就糊成一片了。

> **提示**：苹果浏览器需要 `-webkit-` 前缀才认背景滤镜，生产环境两行都写上，先写带前缀的，再写标准的。

### 3. 常用背景滤镜函数

背景滤镜的函数和普通滤镜基本通用，最常用的就三个：

```css
/* 背后模糊:毛玻璃的本体 */
.glass { backdrop-filter: blur(10px); }
/* 背后提亮:暗图上托起浅色面板 */
.bright-panel { backdrop-filter: brightness(1.3); }
/* 背后加饱和:让透上来的颜色更浓 */
.vivid-panel { backdrop-filter: saturate(1.5); }
```

也可以空格连写叠效果：

```css
.card {
  background: rgba(255, 255, 255, 0.25);
  backdrop-filter: blur(8px) saturate(1.4);
  -webkit-backdrop-filter: blur(8px) saturate(1.4);
  border-radius: 16px;
  border: 1px solid rgba(255, 255, 255, 0.4);
}
```

这就是经典的毛玻璃卡片：压在一张风景图上，卡片里的字清晰，卡片外的风景透进来又被柔化。我做弹窗、登录框、图片上的说明条，基本都是这个配方，换张底图就是新皮肤。

背景滤镜是实时计算的，盖住的区域越大、页面同时启用的面板越多，开销越大。我的习惯是：导航、弹窗这种长期在线的用，列表里几十个卡片每个都加就免了，低端机会掉帧。

> 💡 **承前启后**：完成对「123-背景滤镜backdrop-filter」的理解后，下一章我们将深入探讨「124-动画时间线timeline-滚动时间线」，进一步完善知识图谱体系。

---

## 第 11 章：124-动画时间线timeline-滚动时间线
> 对应分集：P125 | 原始标题：《124-动画时间线timeline-滚动时间线》

不知道你们在浏览一些国际一线大厂或者跑车品牌的官网时，有没有注意过一个非常精致的交互细节。比如打开保时捷（Porsche）的官方网站，或者一些前沿的技术博客，页面的最顶部往往横亘着一根极细的彩色进度指示条。当你转动鼠标滚轮向下浏览网页时，这根彩色条会随着滚动的距离一点一点向右拉长变大；等我们一路拉到页面最底部的时候，它正好严丝合缝地占满视口的最右侧（达到 100%）；而当我们反向往上回滚时，它又会等比缩短回去。

这个小小的彩色条不仅极具科技感，而且能直观地告诉读者：当前内容你读了百分之多少，离整个网页的底部究竟还有多远。

放在以前，要实现这种“动画随着滚轮走”的效果，前端工程师必须在 JavaScript 里面监听 `window` 的 `scroll` 滚动事件，去动态获取 `window.scrollY`、页面的总高度 `scrollHeight` 以及可视窗口高度 `innerHeight`，经过一系列算术运算算出百分比，再频繁修改 DOM 元素的 `style.width`。这样不仅代码繁琐，高频触发的滚动计算还会不断打扰浏览器的主线程，极易造成页面掉帧和滚动卡顿。

而在现代 CSS3 标准中，浏览器为我们带来了一个里程碑式的全新特性——**动画时间线（Animation Timeline）**。我们完全不需要写半行 JavaScript，仅凭纯 CSS 的一句话，就能把动画的执行进度与页面的物理滚动深度进行硬件加速级别的精准绑定。

---

### 1. 动画时间线（Animation Timeline）的运行机制与架构定位

#### 1.1 从时间驱动到空间驱动的技术革新
在传统的 CSS 动画体系中，我们通过 `@keyframes` 规划好动画的关键帧之后，驱动动画前进的核心引擎是**物理时间**。

比如我们定义了 `animation: scrollBar 2s linear forwards;`，无论用户是在疯狂滚动页面，还是双手离开键盘静静发呆，浏览器内部的时钟只要走过 2 秒，这个动画就会自顾自地从 0% 跑到 100%。

> **定义**：
> **动画时间线（Animation Timeline）** 是 CSS 动画规范引入的进阶控制机制。它允许开发者将 CSS 动画的执行进度，从单纯依赖系统时钟推进，转为与**特定的外部事件轴（例如页面或容器的滚动行程、元素的视口可见性）** 进行强绑定。

一旦我们为动画挂载了时间线，动画的播放进度轴就不再由毫秒数主导，而是由滚动距离或视口交叉比例来绝对映射。

```text
传统时间动画 vs 滚动时间线映射模型

传统时间驱动模式：
[页面加载完毕] ──(经过 1 秒)──> 进度 50% ──(经过 2 秒)──> 进度 100% (与页面滚动完全脱节)

滚动时间线驱动模式：
[页面顶部 0%]  ──────────────> 关键帧 0%   (进度条宽度 0%)
      │
[滚至半程 50%] ──────────────> 关键帧 50%  (进度条宽度 50%)
      │
[滚至底部 100%] ─────────────> 关键帧 100% (进度条宽度 100%)
      ▲
      └─ 反向滚动向上拉动时，动画帧等比倒放回退！
```
*图 1：时间驱动模式与滚动空间驱动模式的映射关系比对*

#### 1.2 动画时间线的两大核心分支
CSS 的 `animation-timeline` 属性主要涵盖两种截然不同的应用场景：

| 时间线类型 | 关键字函数 | 驱动核心源 | 典型应用场景 |
| :--- | :--- | :--- | :--- |
| **滚动时间线（Scroll Timeline）** | `scroll()` | 页面或特定滚动容器的**绝对滚动距离** | 页面阅读进度条、全局视差背景、头部导航缩放 |
| **视图时间线（View Timeline）** | `view()` | 目标元素在当前**视口（Viewport）中的出入可见性** | 元素滚动入屏淡入放大、卡片滑入显现、画廊交错动画 |

本讲我们重点聚焦在第一种、也是最常用的全局进度指示神器——**滚动时间线**。

---

### 2. 滚动时间线的语法规则与绑定机制

想要在 CSS 中使用滚动时间线，它的语法简洁到了极致。我们依然按照常规方式定义动画与关键帧，仅仅是在动画属性下方追加一行时间线声明：

```css
/* 标准语法声明 */
animation-timeline: scroll();
```

#### 2.1 语法解构与书写规范
- 属性名称为 `animation-timeline`：单词之间用连字符隔开，顾名思义就是“动画的时间线”；
- 属性取值为 `scroll()`：`scroll` 表示滚动，建议严格书写带英文小括号的形式 `scroll()`，这是现代 CSS 规范中最推荐的函数式标准写法；
- 绑定效果：只要为元素声明了这一行，原本写在 `animation` 简写属性里的时长（例如 `2s`、`1s`）就会立刻退居幕后，动画的运行步伐彻底交由页面滚轮来统一指挥。

> **提示**：
> 在支持滚动时间线的现代浏览器中，`animation-duration` 虽然不能再控制动画播放的快慢，但在语法合规性上，基础的 `animation` 简写依然需要声明完整的动画名称与填充模式（如 `forwards`），然后再通过 `animation-timeline: scroll();` 将时间驱动重定向为滚动驱动。

---

### 3. 仿保时捷官网页面顶部滚动进度条实战

为了彻底吃透滚动时间线的落地过程，我们通过一个完整的商业级案例，完整模拟头部导航栏底部的彩色滚动进度条效果。

#### 3.1 页面 DOM 结构设计
我们需要三个核心结构：
1. 一个始终固定在页面视口顶部的导航栏大容器（`.nav-bar`）；
2. 嵌套在导航栏内部底部、初始宽度为 0 的渐变彩色细线（`.scroll-bar`）；
3. 导航栏下方承载长篇正文的页面内容大盒子（`.content`），用来撑开页面高度，产生长距离的垂直滚动条。

```html
<!-- 固定在页面顶部的导航栏 -->
<div class="nav-bar">
  <!-- 滚动进度条彩色细线 -->
  <div class="scroll-bar"></div>
</div>

<!-- 撑开页面高度的内容主体 -->
<div class="content">
  <p>前端前沿技术探索：现代 CSS 动画工程实践...</p>
  <p>向下滚动鼠标滚轮，观察顶部彩色进度条的延伸动态...</p>
  <!-- 内部放置多个段落文本以提供充分的垂直滚动高度 -->
</div>
```

#### 3.2 基础排版与导航布局
首先，我们把导航栏固定在页面最顶端，并为彩色进度条指定高度和渐变背景色：

```css
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
  background-color: #f4f5f7;
}

/* 顶部固定导航栏 */
.nav-bar {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 50px;
  background-color: #ffffff;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
  z-index: 999;
}

/* 进度条彩色细线：绝对定位贴合在导航栏底边 */
.scroll-bar {
  position: absolute;
  bottom: 0;
  left: 0;
  height: 4px;
  /* 炫彩渐变背景色 */
  background: linear-gradient(to right, #ff416c, #ff4b2b, #ffb300, #00c6ff);
  /* 初始状态宽度为 0，看不见 */
  width: 0%;
}

/* 内容区域：留出顶部 50px 间隙，防止被固定导航遮挡 */
.content {
  max-width: 900px;
  margin: 70px auto 40px;
  padding: 30px;
  background-color: #ffffff;
  border-radius: 8px;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.05);
}

.content p {
  line-height: 2;
  margin-bottom: 24px;
  color: #333333;
  font-size: 16px;
}
```

#### 3.3 动画关键帧与时间线绑定
接下来，我们定义进度条展开的关键帧动画。逻辑非常单纯：从宽度 0% 扩展到宽度 100%：

```css
/* 1. 定义进度条宽度拉伸的关键帧 */
@keyframes scrollProgress {
  0% {
    width: 0%;
  }
  100% {
    width: 100%;
  }
}

/* 2. 应用动画并挂载滚动时间线 */
.scroll-bar {
  position: absolute;
  bottom: 0;
  left: 0;
  height: 4px;
  background: linear-gradient(to right, #ff416c, #ff4b2b, #ffb300, #00c6ff);
  
  /* 传统动画声明：指定动画名称、线性匀速与保留终止状态 */
  animation: scrollProgress 2s linear forwards;
  
  /* 核心驱动行：绑定原生滚动时间线 */
  animation-timeline: scroll();
}
```

```text
DOM 与样式装配拓扑图
┌────────────────────────────────────────────────────────┐
│ .nav-bar (position: fixed; width: 100%; height: 50px)   │
│                                                        │
│   ┌──────────────────────────────────────────────────┐ │
│   │ .scroll-bar (height: 4px; animation-timeline:    │ │
│   │              scroll(); width: 0% -> 100%)        │ │
│   └──────────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────────┘
┌────────────────────────────────────────────────────────┐
│ .content (正文区域，撑开纵向滚动条，驱动 scroll() 进度)   │
│   ¶ 段落 1 ...                                         │
│   ¶ 段落 2 ...                                         │
│   ¶ 段落 N (拉到底部时刚好触发关键帧 100%)               │
└────────────────────────────────────────────────────────┘
```
*图 2：导航容器与滚动细线组件的层级拓扑*

#### 3.4 交互运行推导与对比验证
为了验证 `animation-timeline: scroll()` 的真实运行逻辑，我们可以做两组极有说服力的现场比对：

1. **未添加 `animation-timeline` 时**：
   保存代码刷新页面，你会发现进度条在页面刚刚打开的瞬间，自己就以 2 秒的时长从左向右“呲溜”一下自动拉满了。不管你滚不滚页面，它都走完了，完全起不到指示阅读进度的作用。
2. **添加 `animation-timeline: scroll();` 之后**：
   刷新页面，彩色细线静静地停在最左侧（宽度为 0），根本不自动播放。
   - 当我们轻轻向下滚动鼠标滚轮，彩色条开始随着滚动实时向右延伸；
   - 滚到正文中间位置时，彩色条正好停在视口中间；
   - 一直拉到页面最底部，彩色条不偏不倚刚好到达最右侧（100% 满格）；
   - 当我们把滚轮向上反向回滚时，彩色条又丝滑地倒退收缩。

> **易错点**：
> 有些初学者可能会疑惑：“如果我往 `.content` 里面再复制 10 个段落，把页面撑得更长，这个动画会不会在中途就提前拉满了？”
> **绝对不会！**
> 因为 `scroll()` 绑定的不是具体的滚动像素距离（不是按 500px 或 1000px 算的），而是**当前滚动容器的最大可滚动行程（Scroll Range）的相对百分比**。
> 无论你的页面只有 2000 像素高，还是有 20000 像素高，浏览器都会自动将总滚动行程的 0% 映射到动画 0 帧，100% 映射到动画最终帧。只要没滚到页面最底部，进度条就绝对不会提前到达终点。

---

### 4. 滚动时间线的工程总结与注意事项

1. **语法简洁高效**：
   无需引入复杂的第三方库，无需在 JS 循环和事件监听中与 DOM 重绘肉搏。只需标准动画搭配一行 `animation-timeline: scroll();`，就能构建出硬件级平滑的滚动交互。
2. **时间参数被空间覆盖**：
   在声明了滚动时间线之后，原本的 `2s` 或 `1s` 等时间参数不再代表真实的秒数，动画的每一个百分比关键帧完全对应滚动位置的百分比。
3. **书写格式严格对齐规范**：
   `animation-timeline` 后面的值必须使用小括号的函数形式 `scroll()`。切记不要随意更改关键字名称，严格保持官方标准定义。

随着现代主流浏览器对 CSS 滚动驱动动画（Scroll-driven Animations）的全面支持，滚动时间线已经成为现代 Web 页面提升交互体验、呈现高级质感的一项必备武器。

> 💡 **承前启后**：完成对「124-动画时间线timeline-滚动时间线」的理解后，下一章我们将深入探讨「125-动画时间线timeline-视图时间线」，进一步完善知识图谱体系。

---

## 第 12 章：125-动画时间线timeline-视图时间线
> 对应分集：P126 | 原始标题：《125-动画时间线timeline-视图时间线》

上一讲我们啃下了滚动时间线 `scroll()`，我当时第一反应就是：页面顶部的阅读进度条是有了，可页面里那一堆卡片、图片、标题，能不能也在它们滚进屏幕的一瞬间自己淡入、放大、上浮？以前这种"滚到哪、动到哪"的入屏动画，我的本能就是去写 `IntersectionObserver` 或者监听 `scroll` 算位置，代码又臭又长，性能还得自己操心。

这一讲我就跟着把另一半拼图补齐了：**视图时间线（View Timeline）**。它和 `scroll()` 是亲兄弟，区别就一句话：`scroll()` 看的是"整个页面滚了多远"，`view()` 看的是"某一个元素自己在视口里露出了多少"。搞懂这个区别，后面所有滚动入屏动效就全通了。

---

### 1. 视图时间线（View Timeline）的驱动原理

#### 1.1 从页面滚动到元素可见性的视角切换

滚动时间线回答的问题是"页面滚到哪了"，视图时间线回答的问题是"这个盒子露出来多少了"。

> **定义**：
> **视图时间线（View Timeline）** 是 CSS 动画时间线的一种，它把动画的播放进度绑定到**目标元素自身与视口（Viewport）的交叉可见比例**上。元素刚从视口边缘冒头时动画处于 `0%`，完全穿过视口离开后动画走到 `100%`，中间随滚动连续映射，反向滚动则等比倒放。

```text
滚动时间线 vs 视图时间线驱动源对比

scroll() 驱动源：整个滚动容器的行程
[页面顶部] ═════ 滚动距离 ═════> [页面底部]
  └─ 所有绑定元素共享同一条进度轴

view() 驱动源：每个元素各自的可见性
卡片A：[视口外] ─> [冒头0%] ─> [全显] ─> [离开100%]
卡片B：[视口外] ─> [冒头0%] ─> [全显] ─> [离开100%]
  └─ 每个元素拥有独立的进度轴，互不干扰
```

读图说明：`scroll()` 是一条全局进度轴大家共用，`view()` 是每个元素各走各的进度轴，这就是为什么长列表里每张卡片能依次入场而不是一起动。

#### 1.2 两类时间线的选型对照

| 对比维度 | 滚动时间线 `scroll()` | 视图时间线 `view()` |
| :--- | :--- | :--- |
| 绑定对象 | 滚动容器（页面或某盒子） | 被观察的目标元素自身 |
| 进度含义 | 滚动条从头到尾的行程百分比 | 元素进出视口的可见性百分比 |
| 典型效果 | 顶部进度条、视差背景 | 卡片入屏淡入、图片缩放显现 |
| 多元素表现 | 所有元素同步推进 | 每个元素依次独立触发 |

> **结论**：要做"页面级"的整体进度就用 `scroll()`，要做"元素级"的依次入场就用 `view()`。你们以后看到任何"滚下来一张张卡片浮出来"的网站，直接锁定 `view()` 就行。

---

### 2. 视图时间线的基础语法与默认行为

基础写法和上一讲几乎一模一样，定义关键帧、挂动画、再加一行时间线声明：

```css
.card {
  animation: fadeUp linear forwards;
  animation-timeline: view();
}

@keyframes fadeUp {
  from {
    opacity: 0;
    transform: translateY(60px) scale(0.9);
  }
  to {
    opacity: 1;
    transform: translateY(0) scale(1);
  }
}
```

我刚写完这段时心里其实打了个问号：动画时长 `linear` 后面没写秒数，浏览器怎么知道播多快？答案和上一讲一致——一旦挂上 `view()`，**时长就作废了，滚动位置就是时钟**。元素在视口下边缘刚露头，动画就卡在 `from`；滚到穿过视口，动画才走到 `to`。

> **易错点**：`animation-timeline: view();` 的括号不能省。我一开始图省事写成 `animation-timeline: view;`，结果动画直接不播，老老实实把小括号补上才正常，规范写法就是函数式 `view()`。

---

### 3. 动画区间（Animation Range）的精确控制

#### 3.1 默认区间的问题

只写 `view()` 有个让我别扭的地方：默认情况下动画的 `0%~100%` 会铺满元素从"进入视口"到"完全离开视口"的全过程。也就是说卡片刚冒个头就开始变，滚了老长一段才变完，入场感拖拖拉拉不够干脆。

我想要的是：卡片一露头就迅速淡入放大，进来之后就定住别再变了。这就要请出**动画区间（Animation Range）**。

#### 3.2 animation-range 的区间写法

```css
.card {
  animation: fadeUp linear forwards;
  animation-timeline: view();
  animation-range: entry 0% entry 100%;
}
```

> **定义**：
> **动画区间（Animation Range）** 由 `animation-range` 属性声明，用于限定时间线进度轴上的哪一段映射到关键帧的 `0%~100%`。常用关键字有 `entry`（进入阶段）、`contain`（完全包含阶段）、`exit`（离开阶段）、`cover`（覆盖全过程）。

最常用的就是上面这行 `entry 0% entry 100%`，意思是：只在元素"进入视口"这个阶段内播完整个动画，进来之后动画定格，不再随滚动反复横跳。

```text
animation-range 区间示意（元素自下而上穿过视口）

视口 ┌──────────────────┐
     │                  │
     │   contain 区间   │  ← 元素完全在视口内
     │                  │
     └──────────────────┘
     ▲ entry 区间         ▲ exit 区间
  刚冒头处            顶出视口处
  └─ animation-range: entry 0% entry 100%
     只在这段播完 from → to，干脆利落
```

读图说明：默认 `cover` 铺满全过程，`entry` 只取冒头这一截，做入场动画基本锁定 `entry` 就够了。

> **提示**：区间数值支持百分比也支持长度，`entry 0% entry 50%` 表示进入阶段的前一半就播完，动画会更"跟手"。你们多试几个数值，滚两下就有感觉了。

---

### 4. 长列表卡片依次入场实战

把上面几招拼起来，一个经典的图文列表入场效果三步落地：

```css
/* 1. 列表项：各自绑定视图时间线 */
.news-item {
  animation: itemIn linear forwards;
  animation-timeline: view();
  animation-range: entry 0% entry 100%;
}

/* 2. 关键帧：从透明下沉到清晰归位 */
@keyframes itemIn {
  from {
    opacity: 0;
    transform: translateY(80px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}
```

```html
<ul class="news-list">
  <li class="news-item">卡片 01</li>
  <li class="news-item">卡片 02</li>
  <li class="news-item">卡片 03</li>
  <li class="news-item">卡片 04</li>
</ul>
```

实际滚动时的表现是：第一张卡片冒头就浮起来，第二张还在视口外面纹丝不动，等滚到它才轮到它动。每张卡片各走各的时间线，天然就是依次入场，根本不用写错峰延迟。

> **结论**：视图时间线最迷人的地方就是"声明一次、处处生效"。列表有多长都不用加代码，新塞进来的卡片自动获得同样的入场行为。

---

### 5. 兼容性与降级思路

这么新的特性，兼容性必须心里有数。我现在写这类效果会默认叠两层保险：

```css
/* 基础状态：元素默认就是可见的终态 */
.news-item {
  opacity: 1;
}

/* 支持时间线的浏览器才叠加入场动画 */
@supports (animation-timeline: view()) {
  .news-item {
    animation: itemIn linear forwards;
    animation-timeline: view();
    animation-range: entry 0% entry 100%;
  }
}
```

> **提示**：用 `@supports` 做特性查询，老浏览器看到的就是静态列表，内容一个不少；新浏览器自动升级成滚动入场动画。这种渐进增强的写法，面试和实际项目都挑不出毛病。

---

### 6. 滚动时间线与视图时间线的收官对照

学完这两讲，我把动画时间线整个版图归拢成一张表，以后翻出来就能用：

| 时间线 | 声明 | 进度轴 | 一句话记忆 |
| :--- | :--- | :--- | :--- |
| 滚动时间线 | `animation-timeline: scroll();` | 容器滚动行程 | 页面滚多远，动画走多远 |
| 视图时间线 | `animation-timeline: view();` | 元素可见比例 | 盒子露多少，动画播多少 |
| 区间控制 | `animation-range` | 截取进度轴片段 | 只播其中一段，入场更干脆 |

从"时间说了算"到"滚轮说了算"，再到"每个盒子自己说了算"，CSS 动画这条线算是被我彻底拿下了。下一讲开始切入 CSS 变量，又是一个能让代码量砍半的神器。

> 💡 **承前启后**：完成对「125-动画时间线timeline-视图时间线」的理解后，下一章我们将深入探讨「126-CSS变量的定义以及使用」，进一步完善知识图谱体系。

---

## 第 13 章：126-CSS变量的定义以及使用
> 对应分集：P127 | 原始标题：《126-CSS变量的定义以及使用》

写了这么多年 CSS，我最头疼的场景就是换主题色。老板一句话"把全站的粉色换成蓝色"，我就得全局搜索 `#ff6b9d` 之类的色值，一个一个改，改完还得提心吊胆：漏了哪个角落没？ hover 态那一处改了没？这种"牵一发动全身"的苦，这一讲终于有解药了：**CSS 变量（Custom Properties，自定义属性）**。

它的核心诱惑就一句话：把反复出现的值起个名字存起来，后面全站引用这个名字，改一处就全站生效。

---

### 1. CSS 变量的基本定义

#### 1.1 用双横线声明变量

```css
:root {
  --main-color: #ff6b9d;
  --main-bg: #f5f5f5;
  --font-base: 16px;
}
```

> **定义**：
> **CSS 变量（Custom Properties）** 是以连续两条横线 `--` 开头的自定义属性，值可以是任何合法的 CSS 值（颜色、长度、字体、甚至一段完整声明片段）。它遵循 CSS 的级联与继承规则，定义在哪里、哪里 convenio 就能用哪里。

我第一次见 `--main-color` 这种写法时觉得别扭，好好的属性名怎么以横线开头？记住就行：**浏览器见到 `--` 开头就知道这是变量声明而不是普通属性**，这是死规定。

#### 1.2 为什么习惯定义在 :root 上

```css
:root {
  --main-color: #ff6b9d;
}
```

> **提示**：`:root` 就是 `html` 根元素的伪类版本，写在这里的变量全站所有元素都能继承到，相当于全局变量。它的优先级比直接写在 `html` 选择器里高一丢丢，所以业界统一约定全局变量放 `:root`。

```text
CSS 变量作用域层级示意

:root { --main-color: 粉色 }   ← 全局，所有后代可继承
  └─ .header { 引用 var(--main-color) → 粉色 }
  └─ .card { --main-color: 蓝色 }  ← 局部重定义
       └─ .card .btn { 引用 var(--main-color) → 蓝色 }
       └─ .footer { 引用 var(--main-color) → 粉色（不受影响）
```

读图说明：变量支持继承，也支持在任何后代里重新赋值覆盖，只影响自己的子树。

---

### 2. var() 函数的引用与兜底值

定义完变量，用的地方靠 `var()` 函数：

```css
.btn {
  background-color: var(--main-color);
  font-size: var(--font-base);
}
```

#### 2.1 兜底值的写法

```css
.btn {
  background-color: var(--main-color, #333);
}
```

`var()` 的第二个参数是兜底值：如果 `--main-color` 没定义、或者定义的值在当前位置非法，就退回用 `#333`，保证页面不裸奔。

> **易错点**：变量名大小写敏感，`--main-color` 和 `--Main-Color` 是两个东西。我就吃过这个亏，定义和引用差一个大小写，样式死活不生效，查了半天才发现。

#### 2.2 非法值时的表现

| 情况 | 示例 | 结果 |
| :--- | :--- | :--- |
| 变量未定义且无兜底 | `var(--nope)` | 声明无效，属性回退到继承或初始值 |
| 变量未定义但有兜底 | `var(--nope, red)` | 使用兜底值 `red` |
| 变量值类型不匹配 | `--gap: red; margin: var(--gap);` | 该条声明无效 |

> **结论**：凡是引用变量的地方，我都建议顺手写兜底值，多几个字符，换来的是线上少一颗雷。

---

### 3. 局部变量与就近覆盖

变量不是只能全局用，在任何选择器里都能重定义，而且**离谁近谁说了算**：

```css
:root {
  --theme: #ff6b9d;
}

.card {
  --theme: #3b82f6;
  border: 2px solid var(--theme);
}

.card .btn {
  background: var(--theme);
}
```

`.card` 里的按钮拿到的是蓝色，而页面其他引用 `--theme` 的地方依然是粉色。这种"全局定基准、局部按需覆盖"的玩法，做深色模式、多皮肤卡片特别顺手。

---

### 4. JavaScript 读写变量

CSS 变量真正的杀招是能和 JS 互通，这是 Sass/Less 编译期变量做不到的：

```js
const root = document.documentElement;

// 读取变量
const color = getComputedStyle(root).getPropertyValue('--main-color');

// 修改变量：全站引用处瞬间更新
root.style.setProperty('--main-color', '#3b82f6');
```

```css
:root {
  --main-color: #ff6b9d;
}
```

一行 `setProperty`，全站所有 `var(--main-color)` 的地方同时变色，不需要挨个操作 DOM。这就是"一键换肤"的全部秘密，下一讲的综合案例还会继续放大这一招。

> **提示**：`setProperty` 改的是内联样式层级，优先级高于 `:root` 里写的，所以一定生效；想恢复就 `removeProperty('--main-color')`，又会落回样式表的值。

---

### 5. 一键换肤实战

把上面的招式串起来，一个最小可用的换肤开关长这样：

```css
:root {
  --bg: #ffffff;
  --text: #333333;
  --accent: #ff6b9d;
}

body.dark {
  --bg: #1a1a1a;
  --text: #f5f5f5;
  --accent: #7c3aed;
}

body {
  background: var(--bg);
  color: var(--text);
}

.btn {
  background: var(--accent);
}
```

```js
document.querySelector('#themeToggle').addEventListener('click', () => {
  document.body.classList.toggle('dark');
});
```

点击按钮给 `body` 加减一个类，变量整组切换，全站配色跟着翻篇。以前要写两套样式表的事，现在就是多写三行变量。

> **结论**：CSS 变量的本质是"把值变成可运行时的参数"。凡是以后可能要批量改的东西——主题色、间距基数、圆角大小、字体 scale，先问自己一句：要不要抽成变量？十次里有九次答案是"要"。

---

下一讲轮到 `calc()` 计算函数出场，它和变量是天作之合：变量存数，`calc()` 算数，两者一配合，连精灵图坐标这种体力活都能自动化。

> 💡 **承前启后**：完成对「126-CSS变量的定义以及使用」的理解后，下一章我们将深入探讨「127-CSS计算函数calc基本使用」，进一步完善知识图谱体系。

---

## 第 14 章：127-CSS计算函数calc基本使用
> 对应分集：P128 | 原始标题：《127-CSS计算函数calc基本使用》

上一讲我刚把值抽成变量，新的烦恼马上就来了：侧边栏固定 `200px`，右边内容区想"占满剩下的所有宽度"，这个"剩下多少"在 CSS 里怎么写？百分比和像素混在一起，总不能让我用 JS 去量吧？

这一讲的 `calc()` 就是来解这道题的：**让 CSS 自己做四则运算**，而且是浏览器实时算、窗口一变就重算的那种活的运算。

---

### 1. calc() 的出场场景与基本语法

#### 1.1 一个让我开窍的例子

```css
.layout {
  display: flex;
}

.aside {
  width: 200px;
}

.main {
  width: calc(100% - 200px);
}
```

`100%` 是爹给的整行宽度，`200px` 是侧边栏占掉的，`calc()` 当场做减法，`.main` 永远不多不少正好吃掉剩下的。窗口随便拖，它自己重算，永远严丝合缝。

> **定义**：
> **计算函数（calc()）** 是 CSS 提供的数学运算函数，允许在属性值的位置书写包含加减乘除的表达式，浏览器在渲染时实时求值。它支持不同单位混算，这是它最不可替代的一点。

```text
calc() 运算示意

父盒子宽度 100% ┌──────────────────────────┐
                │ 侧边栏 200px │  主体 calc  │
                │<─────>│<───── 100%-200px ─>│
                └──────────────────────────┘
窗口拉宽 → 100% 变大 → calc 结果自动变大
```

读图说明：减号左边是相对单位、右边是绝对单位，混算的结果随视口实时变化，纯手写固定值永远做不到这一点。

---

### 2. 四则运算的完整写法

| 运算符 | 示例 | 说明 |
| :--- | :--- | :--- |
| 加 `+` | `calc(100% + 20px)` | 两边必须留空格 |
| 减 `-` | `calc(100% - 200px)` | 两边必须留空格 |
| 乘 `*` | `calc(var(--n) * 40px)` | 至少一边是纯数字 |
| 除 `/` | `calc(100% / 3)` | 除数必须是纯数字 |

```css
.three-col {
  width: calc(100% / 3 - 20px);
}

.gap-box {
  margin-left: calc(var(--gap) * 2);
}
```

> **易错点**：加号和减号两边**必须各留一个空格**，`calc(100%-200px)` 会被浏览器当成非法值整条丢弃。我刚开始十次有八次栽在这个空格上，记住这个铁律能省大量调试时间。乘除则没有空格强制要求，但为可读性我也习惯留空格。

> **提示**：乘除有一条硬约束——乘法至少一边是数字，除法的除数必须是数字。`calc(100% * 50%)` 这种"百分比乘百分比"在物理上没意义，浏览器会直接拒绝。

---

### 3. 不同单位混算与嵌套

#### 3.1 单位混算

```css
.modal {
  width: calc(100vw - 40px);
  height: calc(100vh - 60px - 60px);
  font-size: calc(1rem + 1vw);
}
```

`vw`、`vh`、`rem`、`px`、`%` 随便混搭，最后一个 declaration  cover 所有：弹窗永远左右各留 `20px` 边距，字号随视口微微浮动。这在响应式布局里是刚需中的刚需。

#### 3.2 与变量联动

```css
:root {
  --sidebar: 200px;
  --gap: 16px;
}

.main {
  width: calc(100% - var(--sidebar) - var(--gap) * 2);
}
```

变量一改，`calc()` 的结果跟着变。我现在做后台管理系统，侧边栏折叠展开就是改一个 `--sidebar` 的值，右边内容区宽度自动重算，一行 JS 都不用碰布局。

---

### 4. 经典三栏与居中实战

```css
/* 左右定宽、中间自适应 */
.left {
  width: 200px;
}
.right {
  width: 240px;
}
.center {
  width: calc(100% - 200px - 240px);
}
```

```css
/* 未知尺寸盒子水平居中留白 */
.inner {
  width: calc(100% - 40px);
  margin: 0 auto;
}
```

> **结论**：凡是出现"占满剩下"、"两边留白"、"N 等分再扣间隙"这类需求，我的第一反应就是 `calc()`。它把原来要 JS 量的活收编回了 CSS，而且渲染时机更早，不会闪一下再归位。

---

### 5. 兼容与书写 checklist

`calc()` 的兼容性在现代浏览器里已经不是问题，真正要防的是自己的手误。我每次写完都会过一遍这三条：

1. 加减号两边有空格吗？
2. 除数是纯数字吗？
3. 括号成对闭合了吗？嵌套的 `var()` 括号别漏。

```css
/* 反面例子：缺空格，整条失效 */
.bad {
  width: calc(100%-200px);
}

/* 正面例子 */
.good {
  width: calc(100% - 200px);
}
```

> **注意**：`calc()` 里写错不会报错，只会静默失效，盒子宽度回退成 `auto`。以后凡是看到宽度"莫名其妙没生效"，先去检查 `calc()` 的空格和括号，八成就是它。

---

变量负责"存数"，`calc()` 负责"算数"，两兄弟已经到齐。下一讲我们就拿精灵图坐标开刀，看看它们联手是怎么把重复体力活消灭掉的。

> 💡 **承前启后**：完成对「127-CSS计算函数calc基本使用」的理解后，下一章我们将深入探讨「128-CSS变量和计算函数修改精灵图坐标效果」，进一步完善知识图谱体系。

---

## 第 15 章：128-CSS变量和计算函数修改精灵图坐标效果
> 对应分集：P129 | 原始标题：《128-CSS变量和计算函数修改精灵图坐标效果》

前两讲我分别拿下了 CSS 变量和 `calc()`，说实话，单独看每个都不难。但这一讲把两者往一块一拼，我才真正体会到什么叫"一加一大于二"——拿来开刀的正好是我最怕的体力活：**精灵图（Sprite）坐标**。

以前切精灵图是什么光景？一张雪碧图上横着排了 8 个图标，每个 `40px` 宽，我就得手写 8 套 `background-position`：`0`、`-40px`、`-80px`……写到第五个我就开始数错格，改个图标尺寸更是全军覆没。今天这个写法，能把这 8 套坐标压缩成一行公式。

---

### 1. 精灵图坐标的手工之痛

#### 1.1 传统写法的重复劳动

```css
.icon-1 { background-position: 0 0; }
.icon-2 { background-position: -40px 0; }
.icon-3 { background-position: -80px 0; }
.icon-4 { background-position: -120px 0; }
.icon-5 { background-position: -160px 0; }
```

每个图标的横坐标都是"序号乘以单格宽度再取负"，规律明显到刺眼，可 CSS 以前表达不出"序号乘宽度"这个意思，只能人肉展开。我数了一下，一个 12 图标的雪碧图，光坐标就要写 12 行，错一行就错位一个图标，调试全靠肉眼。

```text
精灵图坐标规律示意（每个图标 40px 宽）

雪碧图 ┌────┬────┬────┬────┬────┐
       │ 0  │ 1  │ 2  │ 3  │ 4  │  ← 第 n 个图标
       └────┴────┴────┴────┴────┘
坐标 x = -(n × 40px)：0, -40px, -80px, -120px, -160px
```

读图说明：横坐标就是序号与单格宽度的乘积取负，规律固定，完全可以公式化。

---

### 2. 用变量存序号、用 calc() 算坐标

#### 2.1 核心公式

```css
.icon {
  width: 40px;
  height: 40px;
  background-image: url("./sprites.png");
  background-position: calc(var(--i) * -40px) 0;
}

.icon-1 { --i: 0; }
.icon-2 { --i: 1; }
.icon-3 { --i: 2; }
.icon-4 { --i: 3; }
.icon-5 { --i: 4; }
```

> **定义**：
> 这种写法的本质是把"变化的部分"（图标序号 `--i`）抽象成变量，把"不变的规律"（乘以单格宽度取负）沉淀成 `calc()` 公式。新增图标时只加一行序号声明，坐标自动算对。

```html
<span class="icon icon-1"></span>
<span class="icon icon-2"></span>
<span class="icon icon-3"></span>
```

`--i` 是纯数字，乘以 `-40px` 得到合法的长度，`background-position` 照单全收。以后图标改成 `48px` 一格呢？只改公式里的一个数字：

```css
.icon {
  background-position: calc(var(--i) * -48px) 0;
}
```

一行改动，全站图标坐标集体修正。以前要改 12 行的地方，现在改 1 个数字，这就是公式化的威力。

> **提示**：`--i` 存的是纯数字所以能参与乘法。如果存的是带单位的值，乘法那一侧就必须换成数字，具体是哪边做数字，取决于公式怎么写，两种都合法，统一团队风格即可。

---

### 3. 行列双变量的二维扩展

横着排一排不够，真刀真枪的雪碧图往往是多行多列。这时把行列拆成两个变量：

```css
.icon {
  width: 40px;
  height: 40px;
  background-image: url("./sprites.png");
  background-position:
    calc(var(--x) * -40px)
    calc(var(--y) * -40px);
}

.icon-a { --x: 0; --y: 0; }
.icon-b { --x: 1; --y: 0; }
.icon-c { --x: 0; --y: 1; }
.icon-d { --x: 2; --y: 1; }
```

| 变量 | 含义 | 公式 |
| :--- | :--- | :--- |
| `--x` | 列序号（从 0 起） | `calc(var(--x) * -40px)` 控制横坐标 |
| `--y` | 行序号（从 0 起） | `calc(var(--y) * -40px)` 控制纵坐标 |

> **结论**：二维坐标就是两个一维公式的叠加。行列序号一目了然，新人接手代码不用数像素格子，看 `--x: 2; --y: 1` 就知道是第 3 列第 2 行。

---

### 4. 行内样式直传序号的灵活用法

变量还能直接写在行内，连类名都不用起：

```html
<span class="icon" style="--i: 0;"></span>
<span class="icon" style="--i: 1;"></span>
<span class="icon" style="--i: 5;"></span>
```

```css
.icon {
  width: 40px;
  height: 40px;
  background-image: url("./sprites.png");
  background-position: calc(var(--i) * -40px) 0;
}
```

后端循环输出图标时，直接把序号吐进 `style`，前端零新增代码。这种"数据进变量、样式出公式"的分工，是我在组件化开发里最高频的套路之一。

> **注意**：行内 `style="--i: 5;"` 只声明变量不写任何表现，表现全在类里。這种分离让模板和样式各干各的，改布局不动模板，改数据不动样式。

---

### 5. 方法论收官：从枚举到公式

这一讲表面在讲精灵图，实际在讲一种思维升级：

```text
思维升级路径

枚举思维：icon-1 是 0，icon-2 是 -40px，icon-3 是 -80px……
  └─ 每新增一个就多一行，线性膨胀，出错率线性上升

公式思维：坐标 = calc(var(--i) * -40px)
  └─ 新增只加序号，公式恒定不变，出错率趋近于零
```

读图说明：凡是"有规律的重复"，都可以试着把变量抽出来、把规律写成公式，精灵图只是最典型的一例，栅格偏移、轮播位移、阶梯延迟全都同理。

> **结论**：以后再遇到一长串"长得很像"的 CSS，先别急着复制粘贴，问自己一句：变化的是哪个数？能不能抽成变量、把剩下的写成 `calc()`？这一个习惯，能消灭你职业生涯里一半以上的体力代码。

---

下一讲是综合案例"动感菜单"，正好把变量、过渡、变换一起验收，是时候把这几讲的散装招式打成一套组合拳了。

> 💡 **承前启后**：完成对「128-CSS变量和计算函数修改精灵图坐标效果」的理解后，下一章我们将深入探讨「129-综合案例1-动感菜单」，进一步完善知识图谱体系。

---

## 第 16 章：129-综合案例1-动感菜单
> 对应分集：P130 | 原始标题：《129-综合案例1-动感菜单》

前面几讲我手里攒了一堆散装招式：弹性布局、过渡、变换、CSS 变量。这一讲终于到了验收环节——做一个鼠标滑过去会"动"起来的导航菜单。别小看一个菜单，它是检验"布局功底 + 动效手感"的试金石：排不齐显得业余，动得生硬显得廉价。

我的目标很明确：菜单平时端端正正，鼠标悬停到哪一项，哪一项就优雅地凸显出来，配一个跟随滑动的小指示块，整个过程丝滑不抖动。

---

### 1. 菜单的 DOM 结构与布局基座

#### 1.1 语义化结构

```html
<nav class="menu">
  <ul>
    <li class="active"><a href="#">首页</a></li>
    <li><a href="#">课程</a></li>
    <li><a href="#">故事</a></li>
    <li><a href="#">关于</a></li>
  </ul>
  <span class="indicator"></span>
</nav>
```

`nav` 包 `ul` 包 `li` 包 `a`，这是导航菜单最正统的语义结构，搜索引擎和读屏软件都认。最后那个 `.indicator` 是跟随滑动的小色块，靠绝对定位在菜单底部游走。

#### 1.2 弹性布局横排

```css
.menu {
  position: relative;
  background: #222;
}

.menu ul {
  display: flex;
  list-style: none;
  margin: 0;
  padding: 0;
}

.menu a {
  display: block;
  padding: 16px 32px;
  color: #fff;
  text-decoration: none;
}
```

> **提示**：`a` 标签默认是行内元素，上下 `padding` 撑不开点击区，务必先 `display: block` 再谈体验。这个细节做菜单时每次都要检查。

```text
菜单布局结构示意

┌──────────────────────────────────────────┐
│ 首页 │ 课程 │ 故事 │ 关于                 │
│━━━━━━                                    │ ← indicator 小色块（绝对定位在底部）
└──────────────────────────────────────────┘
  └─ ul: display:flex 横排，li 等宽排列
```

读图说明：`ul` 负责横排，`nav` 设 `relative` 给指示块当定位基准，指示块初始压在第一项下方。

---

### 2. 悬停凸显的动感效果

#### 2.1 文字变色与上浮

```css
.menu a {
  transition: color 0.3s, transform 0.3s;
}

.menu li:hover a {
  color: #ff6b9d;
  transform: translateY(-4px);
}
```

鼠标悬停时文字变色加微微上浮 `4px`，幅度虽小，但"动"和"不动"之间就是精致和粗糙的分水岭。`transition` 把两个属性一起过渡，避免了状态切换时的生硬跳变。

> **易错点**：`transition` 一定要写在常态选择器（`.menu a`）上，而不是 `:hover` 上。写在 `:hover` 里会导致"滑入有动画、滑出没动画"，移开鼠标时直接闪回原位，质感全毁。

#### 2.2 背景高光

```css
.menu li {
  transition: background-color 0.3s;
}

.menu li:hover {
  background-color: rgba(255, 255, 255, 0.08);
}
```

文字动的同时整项背景泛起一层淡高光，双重反馈叠加，悬停感立刻浓郁起来。透明度只用 `0.08`，点到为止，太浓会显脏。

---

### 3. 跟随滑动的指示块

这是整个案例最出彩的一笔：底部一根小色条，鼠标滑到哪一项，它就平滑地滑到哪一项下方。

#### 3.1 变量定位法

```css
.menu {
  --x: 0;
}

.indicator {
  position: absolute;
  bottom: 0;
  left: 0;
  width: 96px;
  height: 4px;
  background: #ff6b9d;
  transform: translateX(calc(var(--x) * 100%));
  transition: transform 0.3s;
}

.menu li:nth-child(1):hover ~ .indicator { --x: 0; }
```

配合每一项悬停时改 `--x` 的值：

```css
.menu ul {
  display: flex;
}

.menu li:nth-child(2):hover ~ .indicator,
.menu li:nth-child(2):hover {
  --x: 1;
}
```

更通用的做法是给每项单独声明序号（沿用上一讲精灵图的思路）：

```css
.menu li:nth-child(1) { --i: 0; }
.menu li:nth-child(2) { --i: 1; }
.menu li:nth-child(3) { --i: 2; }
.menu li:nth-child(4) { --i: 3; }

.menu li:hover {
  --x: var(--i);
}
```

> **定义**：
> 这里的本质是"变量做状态、位移做表现"：悬停改变的是无形的变量 `--x`，真正产生动画的是引用它的 `transform: translateX(calc(var(--x) * 100%))` 加上 `transition`。状态与表现解耦，代码立刻变清爽。

实际有个结构细节要注意：`~` 兄弟选择器要求 `.indicator` 和 `li` 是兄弟，所以指示块要放在 `ul` 外面、跟 `ul` 平级，或者干脆用 JS 改变量。两种路线都能跑通，按你们项目的结构选就行：

```html
<nav class="menu">
  <ul>
    <li style="--i: 0;"><a href="#">首页</a></li>
    <li style="--i: 1;"><a href="#">课程</a></li>
    <li style="--i: 2;"><a href="#">故事</a></li>
    <li style="--i: 3;"><a href="#">关于</a></li>
  </ul>
  <span class="indicator"></span>
</nav>
```

```css
.menu ul:hover .indicator,
.menu .indicator {
  transform: translateX(calc(var(--x, 0) * 100%));
}
```

> **提示**：`var(--x, 0)` 里带兜底 `0`，鼠标没进来时指示块老老实实待在第一项下方，不会乱飘。

---

### 4. 完整代码串联

把布局、悬停、指示块拼成一份可直接运行的代码：

```css
.menu {
  position: relative;
  --x: 0;
  background: #222;
}

.menu ul {
  display: flex;
  list-style: none;
  margin: 0;
  padding: 0;
}

.menu li {
  transition: background-color 0.3s;
}

.menu li:hover {
  background-color: rgba(255, 255, 255, 0.08);
  --x: var(--i);
}

.menu a {
  display: block;
  padding: 16px 32px;
  color: #fff;
  text-decoration: none;
  transition: color 0.3s, transform 0.3s;
}

.menu li:hover a {
  color: #ff6b9d;
  transform: translateY(-4px);
}

.indicator {
  position: absolute;
  bottom: 0;
  left: 0;
  width: 96px;
  height: 4px;
  background: #ff6b9d;
  transform: translateX(calc(var(--x) * 100%));
  transition: transform 0.3s;
}
```

```text
交互流程全景

鼠标滑入第 3 项
  → li:hover 触发
    → --x 被改写为 2
      → indicator 的 translateX 重算为 200%
        → transition 让色块平滑滑过去（0.3s）
      → a 的颜色与位移同步过渡
```

读图说明：一次悬停触发两条过渡链——色块横滑、文字上浮变色，两者同时播完，菜单就"动感"起来了。

---

### 5. 调试 checklist

做完之后我习惯滚一遍这三项：

| 检查项 | 症状 | 解法 |
| :--- | :--- | :--- |
| 每项宽度是否一致 | 指示块滑偏，对不齐 | `li` 定宽或 `a` 的横向 `padding` 统一 |
| `transition` 的位置 | 滑出时闪回 | 移到常态选择器上 |
| 变量兜底 | 初次加载指示块失踪 | `var(--x, 0)` 补默认值 |

> **结论**：这个菜单虽小，五脏俱全——弹性布局管排列，过渡管丝滑，变量加 `calc()` 管跟随位移。把它吃透，后面的滑动导航栏、炫酷导航栏都是换皮，骨架全是这一套。

> 💡 **承前启后**：完成对「129-综合案例1-动感菜单」的理解后，下一章我们将深入探讨「130-综合案例2-滑动导航栏效果」，进一步完善知识图谱体系。

---

## 第 17 章：130-综合案例2-滑动导航栏效果
> 对应分集：P131 | 原始标题：《130-综合案例2-滑动导航栏效果》

我跟你们说，导航栏你们都写过无数个了，无非就是一排 `a` 标签，鼠标放上去变个颜色就完事。但你们有没有刷到过那种很丝滑的效果：鼠标移到哪个菜单上，下方就有一条小色块嗖一下滑过去，稳稳停在你指的那个词下面，离开之后又弹回去。这种效果看起来很高级，其实就三板斧：定位、过渡加兄弟选择器，今天我们就把它一次性拿下。

我先把最终要做的东西给你们描述清楚。导航是一个深色圆角大条，里面均匀排着 5 个选项：首页、课程、故事、关于、联系。每个选项都是白字。默认情况下，第一项下面压着一个橙色小枕头一样的滑块。当我把鼠标移到第二个词上，这个橙色块不是闪现过去的，而是像抹了油一样平滑地滑到第二个词正下方。移到第五个，它就一路滑到最右边。鼠标彻底移出整个导航，这个滑块还会乖乖滑回起点。

这就是我们常说的滑动导航栏（滑动指示器），它的灵魂不是 JavaScript，而是纯 CSS 的位置计算加过渡。

---

### 1. 导航骨架与横向布局

#### 1.1 语义结构搭建

我习惯先把结构写干净。外层一个 `nav` 包着无序列表，滑块我单独放一个 `span`，千万不要把它塞进 `li` 里面，因为它是要在整条导航里自由游走的，它跟任何一个菜单项都不存在父子关系。

```html
<nav class="nav">
  <ul>
    <li><a href="#">首页</a></li>
    <li><a href="#">课程</a></li>
    <li><a href="#">故事</a></li>
    <li><a href="#">关于</a></li>
    <li><a href="#">联系</a></li>
    <span class="slider"></span>
  </ul>
</nav>
```

你们记住这个顺序：5 个 `li` 在前，`span.slider` 断后。这个顺序直接决定了后面我们能不能用 `~` 找到它，写反了就全崩了。

#### 1.2 弹性横向排布

导航条本身我给了固定宽度，比如 `600px`，高度 `60px`，背景深灰，圆角 `30px`。里面的 `ul` 我直接 `display: flex`，让 5 个 `li` 一字排开，每个 `flex: 1`，文字居中。

```css
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #1e1e2a;
}

.nav ul {
  position: relative;
  display: flex;
  width: 600px;
  height: 60px;
  background-color: #2c2c3a;
  border-radius: 30px;
  list-style: none;
}

.nav ul li {
  flex: 1;
  display: flex;
  justify-content: center;
  align-items: center;
}

.nav ul li a {
  color: #fff;
  text-decoration: none;
  font-size: 17px;
  z-index: 2;
}
```

> **注意**：`ul` 一定要 `position: relative`，它是滑块定位的基准。`a` 标签我给了 `z-index: 2`，就是为了让文字永远浮在滑块上面，不然滑块一过来就把字盖住了。

```text
导航层叠关系侧视图
+------------------------------------------+
| ul (relative, flex横排)                  |
| [首页][课程][故事][关于][联系]           |
|  ____                                    |
| |slider| (absolute, z-index:1)            |
|  ----                                    |
|  文字层 (z-index:2, 永远在最上)          |
+------------------------------------------+
```

读图说明：滑块和文字是上下两层，文字层级更高，所以滑块只能在文字背后滑动。

---

### 2. 滑块定位与过渡质感

滑块本质就是一个绝对定位的小圆角矩形。我让它的宽高小于一个菜单格，比如宽 `100px`、高 `40px`，背景橙色渐变，圆角 `20px`，垂直居中，初始 `left` 对准第一个格子。

```css
.slider {
  position: absolute;
  top: 50%;
  left: 10px;
  width: 100px;
  height: 40px;
  transform: translateY(-50%);
  background: linear-gradient(to right, #ff8a00, #ff3d68);
  border-radius: 20px;
  z-index: 1;
  transition: left 0.35s ease;
}
```

> **定义**：过渡（过渡， `transition`）就是告诉浏览器，某个属性变化时不要瞬间跳变，而是用一段时间平滑地补间过去。

这里我只过渡 `left`，时长 `0.35s`，缓动 `ease`。为什么是 0.35 秒？你们自己可以试，0.1 秒太贼，看不出滑的感觉；0.8 秒太肉，鼠标都走了它还在路上。0.3 到 0.4 秒这个区间最跟手，这就是我反复调出来的经验值，你们直接记住就行。

> **易错点**：有人把 `transition` 写在 `:hover` 上，那样只有移入有动画，移出就瞬间弹回了。一定要写在 `.slider` 本体上，进出才都有过渡。

---

### 3. 兄弟选择器驱动滑块走位

重头戏来了，滑块怎么知道我指到哪了？靠的就是通用兄弟选择器（通用兄弟选择器， `~`）。

你们想，`span.slider` 跟 5 个 `li` 是亲兄弟，而且排在最后。那我是不是可以在 `li` 被悬停的时候，反过来指挥它后面的兄弟往哪走？

```css
.nav ul li:nth-child(1):hover ~ .slider {
  left: 10px;
}
.nav ul li:nth-child(2):hover ~ .slider {
  left: 130px;
}
.nav ul li:nth-child(3):hover ~ .slider {
  left: 250px;
}
.nav ul li:nth-child(4):hover ~ .slider {
  left: 370px;
}
.nav ul li:nth-child(5):hover ~ .slider {
  left: 490px;
}
```

我给你们算笔账：整条 600px，5 等分每份 120px。滑块宽 100px，两边各留 10px 间隙。所以第一个 `left: 10px`，第二个就是 `10 + 120 = 130px`，第三个 `250px`，以此类推，每次加 120。规律一旦摸清，10 个菜单项你们也会推了。

```text
滑块走位刻度尺 (单位 px)
0    10   130   250   370   490   600
|----|-----|-----|-----|-----|-----|
| 首页 | 课程 | 故事 | 关于 | 联系 |
| [==] |
|       [==]                     |
|              [==]              |
```

读图说明：滑块每次步进 120px，正好是一个菜单格的宽度。

> **提示**：`+` 只能找到紧挨着的下一个兄弟，`~` 能找到后面所有的兄弟。我们这里 `li` 和 `span` 中间隔着好几个 `li`，必须用 `~`，用 `+` 是找不到的。

还有人问我，为什么不用 JavaScript 算偏移？你们现在菜单是等宽的，我能手算出来。以后做不等宽菜单，或者要跟随点击锁定状态，那确实得请 JavaScript 出马，用 `offsetLeft` 动态量。但在入门阶段，你们先把这种纯 CSS 的思路吃透，它能帮你们真正理解选择器和层叠，不要一上来就依赖脚本。

---

### 4. 联调手感与复用建议

写完之后你们一定要亲手测三件事。第一，从头到尾慢慢划过去，看滑块是不是一个格一个格匀速跟随，有没有哪一项对不齐，差 5px 都要回去调 `left`。第二，快速来回甩鼠标，看过渡会不会卡顿，如果卡，把 `left` 换成 `transform: translateX()` 性能会更好，但计算方式同理。第三，把页面缩窄，看 600px 写死会不会溢出，真要做响应式，把 `px` 换成百分比或者 `flex` 配合 `calc()` 重算一遍就行。

我为什么这么喜欢拿这个案例给你们练手？因为它把我们前面学的弹性盒布局（弹性盒布局， `flex`）、子绝父相定位、结构伪类（结构伪类， `:nth-child()`）、兄弟选择器、过渡，一口气全串起来了。你们能独立把这个滑块调准，说明选择器和定位这两关算是过了，后面做更炫的导航，底子就有了。

> 💡 **承前启后**：完成对「130-综合案例2-滑动导航栏效果」的理解后，下一章我们将深入探讨「131-综合案例3-炫酷导航栏上」，进一步完善知识图谱体系。

---

## 第 18 章：131-综合案例3-炫酷导航栏上
> 对应分集：P132 | 原始标题：《131-综合案例3-炫酷导航栏上》

我跟你们讲，前面那个滑动条只能算开胃菜，今天这个才叫真家伙。你们去看那些设计师社区、游戏官网，导航栏早就不满足于变个颜色了，要的是通透感、悬浮感、图标会跳舞的感觉。我这一讲就带你们起这个炫酷导航栏的地基：先把毛玻璃底座搭起来，再把图标加文字的结构立住，上半部分做完，它静态就已经很好看了。

很多人一上来就想抄最终效果，结果底座都没打牢，图标东倒西歪。我跟你们强调，一定要先把布局做正，再谈动效，不然你后面加的每一行过渡都是在歪楼上装修。

---

### 1. 整体视觉与文件准备

#### 1.1 效果拆解

我们要做的是一条居中悬浮的胶囊导航。背景是半透明的深色，背后隐隐透出页面渐变，有毛玻璃的模糊感。里面 5 个按钮均匀分布，每个按钮由上面的小图标加下面的小字组成。默认状态图标是灰白的，选中项高亮。你看，它跟上一集最大的区别是：上一集是滑块在动，这一集是按钮自己在动，底座是不动的。

```text
目标导航正视图
+----------------------------------------------+
| ( o )  ( o )  ( o )  ( o )  ( o )            |
| 首页   发现   发布   消息   我的              |
|~~~~~~~~ 毛玻璃底座 (半透明+模糊) ~~~~~~~~~~~~|
+----------------------------------------------+
```

读图说明：底座负责通透质感，上层图标加文字负责信息结构，两层分离。

#### 1.2 图标与字体引入

这一集图标我用字体图标（字体图标， `iconfont`）。你们去 `iconfont` 站挑五个，首页、 compass、加号、铃铛、人头，下载下来，用 `link` 引 CSS，或者直接用 Unicode 字符也行。我这里为了讲课方便，先用最稳的 `link` 方式。

```html
<link rel="stylesheet" href="./fonts/iconfont.css">
<nav class="cool-nav">
  <ul>
    <li class="active">
      <a href="#">
        <i class="iconfont icon-home"></i>
        <span>首页</span>
      </a>
    </li>
    <li>
      <a href="#">
        <i class="iconfont icon-compass"></i>
        <span>发现</span>
      </a>
    </li>
    <li>
      <a href="#">
        <i class="iconfont icon-add"></i>
        <span>发布</span>
      </a>
    </li>
    <li>
      <a href="#">
        <i class="iconfont icon-bell"></i>
        <span>消息</span>
      </a>
    </li>
    <li>
      <a href="#">
        <i class="iconfont icon-user"></i>
        <span>我的</span>
      </a>
    </li>
  </ul>
</nav>
```

> **提示**：`i` 管图形，`span` 管文字，`a` 把它们竖着包起来。这个图标上、文字下的结构后面要做联动，必须从 HTML 上就分好层。

---

### 2. 毛玻璃底座制作

#### 2.1 居中悬浮布局

页面背景我先铺一个深色渐变，这样毛玻璃才有东西可透。导航用弹性盒布局（弹性盒布局， `flex`）居中，`ul` 也是 `flex`，5 个 `li` 平分。

```css
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  background: linear-gradient(135deg, #1a1a2e, #16213e, #0f3460);
}

.cool-nav ul {
  display: flex;
  width: 480px;
  height: 80px;
  list-style: none;
  background-color: rgba(255, 255, 255, 0.08);
  border-radius: 20px;
  border: 1px solid rgba(255, 255, 255, 0.15);
}
```

#### 2.2 背景滤镜关键行

毛玻璃的灵魂就一行：背景滤镜（背景滤镜， `backdrop-filter`）。

```css
.cool-nav ul {
  backdrop-filter: blur(15px);
  -webkit-backdrop-filter: blur(15px);
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.3);
}
```

> **定义**：背景滤镜（背景滤镜， `backdrop-filter: blur()`）是对元素背后透过来的内容做模糊，而不是模糊元素自己。一定配半透明背景用，背景写死了就不透了。

你们记住三要素：`background-color` 必须带透明度，比如 `rgba(255,255,255,0.08)`；`backdrop-filter: blur(15px)` 负责糊；再加一圈半透明白边 `border` 和阴影，玻璃的厚重感马上出来。少任何一个，都像塑料。

| 属性 | 取值 | 作用 |
| :--- | :--- | :--- |
| `background-color` | `rgba(255,255,255,0.08)` | 提供透光底色 |
| `backdrop-filter` | `blur(15px)` | 模糊背后背景 |
| `border` | `1px solid rgba(255,255,255,0.15)` | 玻璃描边高光 |

#### 2.3 图文竖排结构

每个 `a` 我让它竖排居中，图标大、文字小。

```css
.cool-nav ul li {
  flex: 1;
  display: flex;
  justify-content: center;
  align-items: center;
}

.cool-nav ul li a {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 4px;
  text-decoration: none;
  color: rgba(255, 255, 255, 0.6);
}

.cool-nav ul li a .iconfont {
  font-size: 24px;
  line-height: 1;
}

.cool-nav ul li a span {
  font-size: 12px;
}
```

`flex-direction: column` 一写，图标就乖乖跑到文字上面了。`gap: 4px` 控制他俩的间距，比写 `margin` 省事。默认颜色我故意调灰一点 `0.6` 透明度，就是为了衬托后面高亮项有多亮。

---

### 3. 上半程联调要点

到这里你们先刷新看一眼：一条透亮的胶囊横在屏幕中间，5 个灰色小图标加小字整整齐齐。如果背景糊 up 不起来，先查两件事：第一，`body` 背景是不是纯色，纯色是看不出毛玻璃的，必须有渐变或者图片垫底；第二，是不是忘了写 `-webkit-` 前缀，旧版 Chrome 只认前缀。

我把这一集停在这里是有意的。很多人学动效总想一口气吃完，结果结构一塌糊涂，动效越炫越乱。你们今天就把底座和结构练到闭眼能写，图标居中、文字大小、间距手感都调顺。下一集我们再请主角登场：悬浮小圆点、图标上跳、颜色点亮，到时候你们会发现，有了今天这个扎实的地基，加动效就是水到渠成的三五行代码。

> 💡 **承前启后**：完成对「131-综合案例3-炫酷导航栏上」的理解后，下一章我们将深入探讨「132-综合案例3-炫酷导航栏下」，进一步完善知识图谱体系。

---

## 第 19 章：132-综合案例3-炫酷导航栏下
> 对应分集：P133 | 原始标题：《132-综合案例3-炫酷导航栏下》

上半集我们把毛玻璃底座和图文结构搭好了，静态看已经很正了。这一集我们就来点真功夫：鼠标放上去图标往上跳一截，颜色从灰变白发光，下面还冒出一颗小圆点，整个导航一下就活了。你们跟着我把这三段动效串起来，这个案例就算彻底封口了。

我跟你们说透，这类炫酷导航看着花，其实就一个心法：默认藏一部分，悬停露一部分，中间用过渡连起来。你们学会藏和露，后面什么花活都能自己变。

---

### 1. 图标上跳与颜色点亮

#### 1.1 位移加变色组合

图标上跳靠的是 2D 变换中的位移（位移， `translate`）。默认 `translateY(0)`，悬停 `translateY(-8px)`。颜色从 `rgba(255,255,255,0.6)` 切到纯白，再加一点文字阴影发光。

```css
.cool-nav ul li a .iconfont {
  font-size: 24px;
  transition: transform 0.3s ease, color 0.3s ease;
}

.cool-nav ul li a span {
  font-size: 12px;
  transition: color 0.3s ease;
}

.cool-nav ul li:hover a .iconfont,
.cool-nav ul li.active a .iconfont {
  transform: translateY(-6px);
  color: #fff;
  text-shadow: 0 0 12px rgba(255, 255, 255, 0.8);
}

.cool-nav ul li:hover a span,
.cool-nav ul li.active a span {
  color: #fff;
}
```

> **结论**：位移管位置跳动，颜色管情绪点亮，阴影管发光质感，三者同时过渡，按钮才有弹跳加发光的复合感。

你们注意，我把过渡写在默认态的 `.iconfont` 上，不是写在 `:hover` 上，这样移入移出都有回弹。时长 `0.3s`，跟上一集滑块同理，太长就黏糊了。

#### 1.2 悬停与选中复用

我这里把 `:hover` 和 `.active` 写成并列选择器，共用一套高亮样式。为啥？鼠标悬停是临时体验，点击锁定是真实状态，他俩视觉必须一致，不然用户会 confused，不知道自己到底选没选中。你们以后做所有导航都照这个习惯写，临时和持久共用一套皮。

---

### 2. 底部指示小圆点

#### 2.1 伪元素藏露逻辑

小圆点我没在 HTML 里加标签，直接用伪元素（伪元素， `::after`）变出来。默认 `opacity: 0` 藏起来，悬停 `opacity: 1` 冒出来，再加一点缩放。

```css
.cool-nav ul li {
  position: relative;
}

.cool-nav ul li::after {
  content: "";
  position: absolute;
  bottom: 10px;
  left: 50%;
  width: 6px;
  height: 6px;
  border-radius: 50%;
  background-color: #00e5ff;
  box-shadow: 0 0 8px #00e5ff;
  transform: translateX(-50%) scale(0);
  opacity: 0;
  transition: opacity 0.3s ease, transform 0.3s ease;
}

.cool-nav ul li:hover::after,
.cool-nav ul li.active::after {
  opacity: 1;
  transform: translateX(-50%) scale(1);
}
```

```text
小圆点状态机
默认态: opacity:0 + scale(0)  -> 看不见
   | hover / .active 触发
   v
点亮态: opacity:1 + scale(1)  -> 青色光点
   | 移出 (transition 回退)
   v
默认态: 缩小+淡出
```

读图说明：藏露靠透明度，弹性靠缩放，两者同步过渡才有冒出来的感觉。

> **易错点**：`li` 必须 `position: relative`，不然 `::after` 的 `absolute` 会跑到 `ul` 甚至 `body` 上，小圆点满天飞。还有 `content: ""` 一定不能漏，漏了伪元素根本不渲染。

颜色我选了青色 `#00e5ff` 加同色辉光，跟白色图标形成对比。你们换成橙色粉色都行，但记住一个原则：底座是冷色，点就用暖色跳，底座是暗色，点就用亮色炸，对比越强，指示性越强。

---

### 3. 整体联调与经验收尾

到这里三件套齐了：图标跳、文字亮、圆点冒。你们连起来测：鼠标从左到右匀速划一遍，看每个按钮是不是依次点亮熄灭，有没有拖影。再点一下让 `.active` 锁住一个，看悬停别人时，锁住的那个还亮不亮，逻辑对不对。

| 动效模块 | 核心属性 | 时长 |
| :--- | :--- | :--- |
| `图标上跳` | `transform: translateY(-6px)` | `0.3s ease` |
| `颜色点亮` | `color + text-shadow` | `0.3s ease` |
| `圆点指示` | `opacity + scale` | `0.3s ease` |

我再跟你们唠两句心里话。你们看，这个案例从头到尾没用一行 JavaScript，全是 CSS 选择器加过渡加伪元素。这就是我想让你们体会的：CSS 的天花板远比你们想象的高。把定位、弹性盒、过渡、伪元素、滤镜这几样揉熟了，你们就能不靠框架做出让人眼前一亮的东西。下一章我们要进滚动叠加卡片，那就要请出视口单位和粘性定位了，你们先把这个导航敲三遍，敲到不用看代码能默写，今天就算过关。

> 💡 **承前启后**：完成对「132-综合案例3-炫酷导航栏下」的理解后，下一章我们将深入探讨「133-综合案例4-滚动叠加卡片首屏以及vw和vh单位」，进一步完善知识图谱体系。

---

## 第 20 章：133-综合案例4-滚动叠加卡片首屏以及vw和vh单位
> 对应分集：P134 | 原始标题：《133-综合案例4-滚动叠加卡片首屏以及vw和vh单位》

你们刷招聘官网、技术大会官网有没有见过这种场面：第一屏一张大海报铺满整个屏幕，你往下滚，下一张卡片像扑克牌一样压上来，把上一张盖住，再往下滚，第三张又压住第二张，一层叠一层，特别有高级感。我跟你们说，这个效果现在已经是高端官网的标配了，今天我们就从它的第一屏讲起，顺手把视口单位（视口单位， `vw/vh`）彻底讲透。

很多人做全屏还在用 `1000px` 写死，或者 `height: 100%` 写了半天没效果，根子就是没搞懂视口单位。这一集你们把 `vw` 和 `vh` 吃透，后面叠卡片就是搭积木。

---

### 1. 首屏全屏布局

#### 1.1 视口单位定义

什么叫视口？就是你浏览器当前能看见的那块玻璃。视口宽度（视口宽度， `vw`， `viewport width`）把这块玻璃的宽等分成 100 份，`1vw` 就是 1 份；视口高度（视口高度， `vh`， `viewport height`）把高分成 100 份，`1vh` 就是 1 份。

```css
.hero {
  width: 100vw;
  height: 100vh;
}
```

> **定义**：`100vw` 等于视口宽度，`100vh` 等于视口高度。它跟随窗口实时变化，窗口拉大它变大，窗口缩小它变小。

你们看，这跟百分比（百分比， `%`）最大的区别是：百分比要看爹，爹没高度你写 `100%` 就是 0；而 `100vh` 不看爹，直接看窗口，写上就满屏。这就是我让你们做首屏必须用 `vh` 的原因，稳，一句话满屏。

| 单位 | 基准 | 是否看父级 |
| :--- | :--- | :--- |
| `%` | 父元素尺寸 | 看，需要父级有高度 |
| `vw/vh` | 视口窗口尺寸 | 不看，直接相对窗口 |

> **提示**：`100vw` 包含滚动条宽度，在有垂直滚动条的页面容易撑出横向滚动条。首屏宽度我一般写 `width: 100%`，高度写 `height: 100vh`，这样最稳。

#### 1.2 首屏文案居中

首屏里面我放一句大标题加一句副标题加一个按钮，整体用弹性盒居中，背景用深色渐变加一点噪点，文字白色。

```css
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

.hero {
  height: 100vh;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  background: linear-gradient(180deg, #0f0f1a, #1e1e3a);
  color: #fff;
  text-align: center;
}

.hero h1 {
  font-size: 64px;
  letter-spacing: 4px;
}

.hero p {
  margin: 16px 0 32px;
  font-size: 18px;
  opacity: 0.7;
}

.hero .btn {
  padding: 12px 36px;
  border-radius: 30px;
  background-color: #fff;
  color: #111;
  text-decoration: none;
  font-weight: 700;
}
```

```text
首屏盒模型 (100vh)
+----------------------------------+
|                                  |
|          h1 大标题               |
|          p  副标题               |
|        [ btn 按钮 ]              |
|                                  |
|  height = 100vh (撑满一屏)       |
+----------------------------------+
```

读图说明：首屏独占一屏高度，内容三件套垂直居中，为后面卡片压上来留出舞台。

---

### 2. 叠加舞台预搭

首屏下面，我先把卡片舞台的架子搭出来，但细节留到下一集。舞台就是一个普通大盒子，里面摞 3 到 4 张卡片，每张卡片也是接近满屏高，圆角、不同背景色，这样滚起来才有扑克牌的感觉。

```html
<section class="hero">
  <h1>滚动叠加</h1>
  <p>向下滚动，看卡片逐层覆盖</p>
  <a class="btn" href="#">开始体验</a>
</section>

<section class="stack">
  <div class="card card1">CARD 01</div>
  <div class="card card2">CARD 02</div>
  <div class="card card3">CARD 03</div>
</section>
```

```css
.stack {
  padding: 40px 20px;
}

.card {
  height: 80vh;
  border-radius: 24px;
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 48px;
  font-weight: 800;
  color: #fff;
}

.card1 { background-color: #ff5e62; }
.card2 { background-color: #4e54c8; }
.card3 { background-color: #11998e; }
```

你们先写到这，刷新看：第一屏满屏海报，往下滚三张大色块依次出现，但现在还是首尾相接的普通流，还没有压住的效果。不要急，压住靠的是下一集的粘性定位（粘性定位， `sticky`）加层叠，剩下的戏肉我们下一集接着唱。

> **易错点**：有人把 `100vh` 写在 `html, body` 上撑高度，那是多余的。`body` 高度由内容撑开就行，只有需要满屏的那一节才给 `100vh`。还有手机浏览器地址栏伸缩会导致 `vh` 跳动，真要做移动端精调可以用 `100dvh`，但入门阶段先把 `100vh` 用熟。

这一集你们就记住两句话：要满屏找 `vh`，不要找百分比；首屏先做漂亮，叠加先搭架子。地基打正了，下一集我们让卡片飞起来的时候，你们才不会手忙脚乱。

> 💡 **承前启后**：完成对「133-综合案例4-滚动叠加卡片首屏以及vw和vh单位」的理解后，下一章我们将深入探讨「134-综合案例4-滚动叠加卡片主体制作」，进一步完善知识图谱体系。

---

## 第 21 章：134-综合案例4-滚动叠加卡片主体制作
> 对应分集：P135 | 原始标题：《134-综合案例4-滚动叠加卡片主体制作》

上一集我们把首屏海报铺满了，也把三张大卡片的架子搭好了，但它们现在还是像火车车厢一样一节接一节，压根没有叠起来的感觉。这一集我们就来动真格的：请出粘性定位，让每张卡片滚到顶部就粘住，后面的卡片再盖上来，扑克牌式叠加一下就成了。

我跟你们说，这个案例是检验你们定位功底的试金石。相对、绝对、固定、粘性，前面都讲过了，今天就是综合验收，谁粘谁盖，谁先谁后，全在细节里。

---

### 1. 卡片主体结构精修

#### 1.1 内容分区

光秃秃的色块太假了，我给每张卡片里塞真东西：左文右图。左边序号加标题加描述，右边一张图。这样叠起来的时候，层次感才真实。

```html
<section class="stack">
  <div class="card card1">
    <div class="text">
      <span>01</span>
      <h2>设计语言</h2>
      <p>圆角、阴影与留白，构成第一张卡片的呼吸感。</p>
    </div>
    <div class="pic">IMG</div>
  </div>
  <div class="card card2">
    <div class="text">
      <span>02</span>
      <h2>交互动效</h2>
      <p>过渡与变换让页面从静态变为可触摸。</p>
    </div>
    <div class="pic">IMG</div>
  </div>
  <div class="card card3">
    <div class="text">
      <span>03</span>
      <h2>工程落地</h2>
      <p>粘性定位加层叠，把高级感做进业务里。</p>
    </div>
    <div class="pic">IMG</div>
  </div>
</section>
```

#### 1.2 双栏弹性排布

卡片本身用弹性盒，左右两栏，圆角 `24px`，内边距拉开，高度保持 `80vh`，这样三张高度一致，叠起来边角才对齐。

```css
.card {
  height: 80vh;
  border-radius: 24px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 60px;
  color: #fff;
  margin-bottom: 40px;
}

.card .text span {
  font-size: 20px;
  opacity: 0.6;
}

.card .text h2 {
  font-size: 48px;
  margin: 12px 0;
}

.card .text p {
  font-size: 18px;
  opacity: 0.85;
  max-width: 420px;
  line-height: 1.8;
}

.card .pic {
  width: 38%;
  height: 100%;
  border-radius: 16px;
  background-color: rgba(255, 255, 255, 0.2);
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 32px;
}
```

> **提示**：`margin-bottom: 40px` 先留着，它是卡片之间的透气缝。等粘住之后，这个缝会变成下一张露出来的小边，可有可无，但留一点更透气。

---

### 2. 粘性定位实现叠加

#### 2.1 粘住顶部

核心就两行：每张卡片都 `position: sticky`，`top` 给一个值，比如 `20px`。

```css
.card {
  position: sticky;
  top: 20px;
}
```

> **定义**：粘性定位（粘性定位， `sticky`）是相对加固定的混合体。在容器内滚动时它像相对定位一样跟着走，一旦滚到 `top` 指定的阈值就粘在视口上，直到父容器滚完才放行。

你们想想这个过程：第一张卡滚到离顶部 20px，啪，粘住了。第二张继续往上走，自然就盖到第一张身上了。第二张滚到 20px，也粘住了，第三张再盖上来。一层压一层，叠加就成了。

```text
粘性叠加时序图
滚动前: [card1]
        [card2]
        [card3]

滚到card1粘住: [card1*粘住 top:20px]
               [card2 上移盖住card1下半]
滚到card2粘住: [card1 被盖住]
               [card2*粘住]
               [card3 上移盖住card2]
```

读图说明：先粘的被后来的盖住，后来居上，这就是扑克牌的秘密。

#### 2.2 层叠顺序控制

光粘还不够，还得让后来的能盖住先来的。正常文档流里，后面的元素本来就压住前面的，只要你们别乱写 `z-index`，顺序就是对的。我这里干脆显式写清楚，省得以后加东西乱套。

```css
.card1 { background-color: #ff5e62; z-index: 1; }
.card2 { background-color: #4e54c8; z-index: 2; }
.card3 { background-color: #11998e; z-index: 3; }
```

> **易错点**：`sticky` 一定要写在卡片自己身上，不要写在父容器 `.stack` 上。写在父容器上，整个舞台一起粘，里面就不叠了。还有父容器千万不要写 `overflow: hidden`，一写粘性全废，这是新手最容易踩的坑。

| 定位 | 粘性阈值 | 叠加角色 |
| :--- | :--- | :--- |
| `card1 sticky` | `top: 20px` | 最先粘，最先被盖 |
| `card2 sticky` | `top: 20px` | 盖住 card1，再被盖 |
| `card3 sticky` | `top: 20px` | 盖住前两张，收尾 |

你们还可以玩点进阶：让 `top` 逐张递增，比如 `20px、40px、60px`，叠住之后会露出上面卡片的一小截边，形成阶梯式书签感。或者给每张加一点缩放过渡，下一集我们配合视图时间线，还能让卡片在被盖时微微缩小，更有纵深。先把今天这种等 `top` 的压住练稳，再去变花样。

---

### 3. 联调手感

写完你们慢速滚三遍。第一遍看粘不粘：每张到顶部有没有停一下，不停就是 `sticky` 没生效，回去查父级 `overflow`。第二遍看盖不盖：后一张有没有严丝合缝盖住前一张，露缝就调 `margin-bottom` 和圆角。第三遍快速甩，看有没有抖动，抖就把卡片高度从 `80vh` 微调到 `85vh`，留足行程。

我跟你们说，这个叠加卡片加上首屏的 `100vh`，再加上之前的毛玻璃、滑动导航，你们现在手里已经攒了一套能直接写进简历的高端组件了。不要小看这三张卡片，大厂官网的滚动叙事，原理跟它一模一样。你们今天把它敲到闭眼能写，后面我们再加一行视图时间线让它自动缩放，那高级感直接拉满，咱们下一集见。

> 💡 **承前启后**：完成对「134-综合案例4-滚动叠加卡片主体制作」的理解后，下一章我们将深入探讨「135-综合案例4-滚动叠加卡片制作」，进一步完善知识图谱体系。

---

## 第 22 章：135-综合案例4-滚动叠加卡片制作
> 对应分集：P136 | 原始标题：《135-综合案例4-滚动叠加卡片制作》

你们有没有在一些大厂官网刷到过这种效果：页面往下滚，一张大卡片吸在屏幕上不动，下一张卡片从底下升上来，把上一张一点一点盖住，一张压一张，像扑克牌一样叠起来。我第一次见这个效果是在苹果的产品页上，当时心里就一个念头——这得写多少 JavaScript 监听滚动才能做出来？

今天我们就把这个念头彻底打掉。上一讲我们已经把首屏和整组卡片的空架子搭好了，这一讲只干一件事：让这几张卡片真正叠起来。而且我提前告诉你们结论：不需要一行 JavaScript，靠粘性定位（粘性定位，sticky）几行 CSS 就够了。你们跟着我把这一讲走完，以后再看到任何"滚动层叠"的页面，心里都会咯噔一下：不就是几个 `sticky` 加不同的 `top` 值吗？

---

### 1. 层叠滚动的视觉规律与结构拆解

先别急着写代码，我们先把眼睛看到的东西翻译成布局语言。你们现在慢慢往下滚，盯着看，回答我三个问题：第一，每张卡片是不是都曾经"吸"在屏幕上停留过一段时间？第二，后一张卡片是不是从前一张的底下钻出来往上走的？第三，最先被盖住的是哪一张？

全答对了：每张卡片都会吸顶停留，后一张压前一张，最底层的第一张最先消失。把这个规律画出来就是这样：

```text
滚动层叠卡片的运动过程示意（从上往下滚动）

状态 A（刚进入卡片区）：        状态 B（继续下滚）：          状态 C（再往下滚）：

+------------------+            +------------------+            +------------------+
|  卡片 1（吸顶）   |            |  卡片 1（被压住） |            |  卡片 2（被压住） |
+------------------+            +------------------+            +------------------+
        |                               |                               |
   卡片 2 跟随上移                  卡片 2 吸顶、盖住卡片 1           卡片 3 吸顶、盖住卡片 2
        |                               |                               |
   卡片 3 在更下方                 卡片 3 跟随上移                  卡片 4 跟随上移
```

读图说明：箭头方向是滚动方向，每张卡片依次经历"跟随上移 → 吸顶停留 → 被下一张盖住"三个阶段，盖住的顺序严格等于文档流顺序。

> **结论**：层叠的本质不是"动画"，而是"每一张卡片都吸顶，后来者居上"。文档流顺序本身就是层叠顺序，浏览器会自动让后面的卡片压住前面的卡片。

所以这一讲的结构拆解只有一句话：一个装全部卡片的大容器，里面依次放四张结构相同的子卡片，每张卡片一个背景色、一张图、一段文字。结构上一讲已经有了，这一讲我们动样式。

---

### 2. 粘性定位实现吸顶层叠的核心写法

#### 2.1 每张卡片的粘性定位声明

还记得粘性定位的脾气吗？它平时是相对定位（相对定位，relative），滚到阈值位置就变成固定定位（固定定位，fixed）。我们给每张卡片统一写：

```css
.card {
  position: sticky;
  top: 0;
  height: 100vh;
}
```

你们看，就三行。`height: 100vh` 让每张卡片恰好占满一屏视口（视口，viewport）高度；`position: sticky` 加 `top: 0` 让它滚到视口顶部时吸住。第一张卡片先吸住，第二张从下面升上来把它盖住，第二张再吸住，第三张再盖上来……层叠就这么发生了。

> **易错点**：`sticky` 生效有两个死条件，缺一不可。第一，卡片自身必须写 `top`（或者 `bottom`）阈值，不写阈值它永远是相对定位，吸不起来；第二，父容器必须足够高、且不能写 `overflow: hidden` 裁掉它。多少人复制了三行代码发现吸不住，回头一查全是父级被加了 `overflow: hidden`，你们一定要记住这一条。

#### 2.2 错开顶边距的叠放层次

如果每张卡片的 `top` 都是 `0`，叠起来后前面的卡片会被盖得严严实实，底下压了几张根本看不出来，效果就打了折扣。我们想要的是每张卡片的头顶都露出一小截，像台阶一样。所以四张卡片的 `top` 要错开：

```css
.card:nth-child(1) { top: 0; }
.card:nth-child(2) { top: 40px; }
.card:nth-child(3) { top: 80px; }
.card:nth-child(4) { top: 120px; }
```

为什么是这个数？不是 70、不是 80 固定死的？你们记住思路就行：错开的距离就是你想让上一张卡片露出的"帽檐"高度。我这里取 40 像素一档，四张卡片就是 0、40、80、120。露出的帽檐连起来看，正好是一道彩色的台阶边，这就是很多官网层叠卡片顶部那条彩虹边的由来。

```text
错开 top 值后的吸顶状态（侧视示意）

视口顶部 ─────────────────────
  top:0   [ 卡片 1 露出 40px 帽檐 ]
  top:40  [ 卡片 2 露出 40px 帽檐 ]
  top:80  [ 卡片 3 露出 40px 帽檐 ]
  top:120 [ 卡片 4 铺满剩余视口   ]
```

读图说明：四张卡片吸顶位置逐级下移，每张恰好露出一截头顶，层叠关系一目了然。

> **提示**：`top` 的级差值要和卡片圆角、内边距一起看，级差太小帽檐挤在一起看不清，级差太大又浪费视口高度。40 像素左右是实践里最顺眼的一档，你们做自己的页面时上下微调就行。

---

### 3. 卡片内部版式与视觉打磨

#### 3.1 卡片内部的图文结构

骨架叠起来了，接下来是每张卡片肚子里的东西。我定的版式很简单：左边文字区，右边图片区，文字区里一个编号、一个大标题、一段描述。结构长这样：

```html
<section class="cards">
  <div class="card">
    <div class="card-text">
      <span class="num">01</span>
      <h2>第一张卡片标题</h2>
      <p>卡片的描述文字写在这里……</p>
    </div>
    <div class="card-img">
      <img src="./images/1.jpg" alt="卡片配图">
    </div>
  </div>
  <!-- 卡片 2、3、4 结构相同 -->
</section>
```

卡片自身用弹性盒（弹性盒，flex）左右排，文字和图片各占一半，垂直居中。图片定宽高加 `object-fit: cover`，保证不同尺寸的图裁出来不变形。圆角我给了 `24px`，卡片和视口边缘之间留一点呼吸空间，叠起来才有"一张一张独立卡片"的感觉，而不是几块颜色糊在一起。

#### 3.2 背景色与文字对比

四张卡片的背景色我取了深浅交错的四个色号。为什么强调深浅交错？你们想，层叠的时候上下两张卡片是贴在一起的，如果颜色接近，交界线就糊掉了，滚动时根本看不出"盖住"的动作。颜色反差拉开，盖住的瞬间才干脆利落。

文字颜色跟着背景走：深底配白字，浅底配深字。行高（行高，line-height）给到 1.8，描述文字字号 18 像素左右，这个密度在 100vh 的大卡片里读起来最舒服。编号数字我用了超大字号加半透明，你们别小看这个编号，它是层叠区最重要的路标——用户滚到哪一张，余光扫到编号就知道进度了。

> **结论**：这一讲做完，纯 CSS 的滚动层叠就已经成立了：滚下去一张压一张，滚上来一张一张退回去，全程不需要 JavaScript。但现在的切换是"硬盖"，下一讲我们给它加上视图时间线（视图时间线，view timeline），让被盖住的卡片一边被压一边缩小变暗，那个味道一下就高级了。

> 💡 **承前启后**：完成对「135-综合案例4-滚动叠加卡片制作」的理解后，下一章我们将深入探讨「136-综合案例4-滚动叠加卡片添加视图时间线」，进一步完善知识图谱体系。

---

## 第 23 章：136-综合案例4-滚动叠加卡片添加视图时间线
> 对应分集：P137 | 原始标题：《136-综合案例4-滚动叠加卡片添加视图时间线》

上一讲结束的时候我卖了个关子：现在的层叠是"硬盖"，后一张"哐"地一下把前一张糊住，动作是有了，味道还差一截。你们去刷苹果官网的层叠区就会发现，人家被盖住的那张不是傻站着等盖的，它会一边被压，一边自己往里缩一点、变暗一点，像是主动往后退。就是这后退的一小步，高级感全出来了。

那问题来了：这个"被盖住时缩小变暗"怎么写？用 JavaScript 监听每一张卡片的位置再算缩放？能写，但上一讲我们就说过，高频滚动加频繁改样式，主线程迟早被拖卡。今天我们用纯 CSS 的视图时间线（视图时间线，view timeline）把它收了，两三行代码，浏览器硬件加速直接给办了。

---

### 1. 视图时间线的驱动逻辑与适用判断

#### 1.1 滚动时间线与视图时间线的分工

之前我们学过滚动时间线（滚动时间线，scroll timeline），`animation-timeline: scroll()` 是拿整个页面的滚动距离当进度条的：滚到顶是 0%，滚到底是 100%。它回答的是"页面滚了多远"。

视图时间线回答的是另一个问题："这个元素在视口里处在什么位置"。`animation-timeline: view()` 把动画进度绑到单个元素和视口的交叉关系上：元素刚从视口底部冒头是 0%，完全进入是中间，顶着视口顶部消失是 100%。

| 时间线类型 | 声明写法 | 进度来源 | 回答的问题 |
| :--- | :--- | :--- | :--- |
| 滚动时间线 | `scroll()` | 页面整体滚动距离 | 页面滚了多远 |
| 视图时间线 | `view()` | 单个元素的视口可见性 | 这个元素走到哪了 |

> **定义**：视图时间线是以目标元素相对视口（视口，viewport）的进出过程为进度轴的动画时间线。元素进入视口、穿过视口、离开视口的完整行程，被映射为动画的 0% 到 100%。

#### 1.2 为什么层叠卡片必须用视图时间线

你们想想，我们要的是"每一张卡片在自己被盖住的时候缩小"。四张卡片被盖住的时机各不相同：第一张最先被盖，第四张到最后都没人盖它。如果用滚动时间线，四张卡片共用同一根进度条，动作必然整齐划一，那就错了。

视图时间线是每张卡片各走各的进度：每张卡片从自己吸顶、到被下一张顶上去、再到离开视口，完整走一遍自己的 0% 到 100%。各走各的，互不干扰，这正是层叠动效要的。记住这条判断：同一动效要在不同元素上错峰发生，就用 `view()`；全页面统一的一个进度，就用 `scroll()`。

---

### 2. 被覆盖卡片的缩放与压暗写法

#### 2.1 关键帧与时间线绑定

写法上还是老三样：先写 `@keyframes`，再写 `animation`，最后把时间线从时钟切换成视图。以前一张卡片被盖住时"缩小加变暗"为例：

```css
.card {
  animation: scaleDown linear both;
  animation-timeline: view();
}

@keyframes scaleDown {
  to {
    transform: scale(0.9);
    filter: brightness(0.6);
  }
}
```

你们看，关键帧里就两行：`scale(0.9)` 缩到九成，`brightness(0.6)` 亮度压到六成。`animation-timeline: view()` 把这段动画的进度交给卡片自己的视口行程：卡片吸顶不动时进度几乎不走，下一张顶上来、它开始离开视口时进度飞涨，缩和暗就发生了。滚回去呢？进度倒放，卡片自己弹回来变亮，全程可逆。

> **提示**：`animation` 简写里那个时长（比如 `1s`）在时间线接管后不再控制快慢，但语法上必须保留一位，写 `linear` 保证进度和滚动严格等比，不要带缓动函数，否则滚动和动画会脱节，看着发飘。

#### 2.2 填充模式与细节修正

`both` 这个填充模式（填充模式，fill-mode）一定要写。什么意思呢？动画进度还没开始（卡片还没进入行程）时保持首帧，进度走完（卡片离开视口）后保持尾帧。没有它，卡片在行程之外会闪回无动画状态，层叠区边缘会出现一闪而过的跳变。

还有一个细节：缩放默认是以元素中心为基准（`transform-origin: center`）向内缩的，这正是我们要的"往后退"的感觉，千万不要手贱改成顶部基准，那样卡片会一边缩一边错位，反而露馅。亮度值 0.6 是我试出来的，压得太狠底下的字看不见了，压得太轻又没"退下去"的感觉，你们在 0.5 到 0.7 之间调就行。

```text
单张卡片在视图时间线上的状态映射

视口底部冒头 ──→ 吸顶停留 ──→ 被下一张顶起 ──→ 离开视口
   0%              进度平缓          进度飞涨           100%
    │                 │                 │                 │
 scale(1)          scale(1)        scale(1→0.9)       scale(0.9)
 brightness(1)   brightness(1)  brightness(1→0.6)  brightness(0.6)
```

读图说明：动效几乎全部发生在"被顶起离开"的后半程，前半程保持原样，这正是视图时间线自动算出来的节奏。

---

### 3. 整组卡片的联调与兼容处理

四张卡片统一样式，动效自然错峰：第一张先缩先暗，第四张因为没有下一张盖它，行程里几乎不动，一直保持原样收尾，结尾干净。这个"不用单独处理最后一张"的省心，恰恰证明选 `view()` 选对了。

最后说兼容。视图时间线是比较新的 CSS 特性，老浏览器不认 `animation-timeline` 会直接整段忽略。忽略的结果是什么？卡片回到上一讲的"硬盖"状态，层叠功能本身不受影响，只是少了缩放压暗的味道。这种"新的浏览器有动效、旧的浏览器保功能"的退化，恰恰是最健康的渐进增强（渐进增强，progressive enhancement）。所以你们放心写：底线是上一讲的粘性层叠，上限是这一讲的视图动效，两头都站得住。

> **结论**：到这一讲，滚动叠加卡片这个综合案例彻底收官了。回头看我们用的东西：`100vh` 定屏高，`sticky` 加错开的 `top` 做层叠，`view()` 时间线做退场动效，全程零 JavaScript。下一讲我们翻开第六篇，换个战场，去啃移动端。

---

## 模块 05 全景总结与技术沉淀

本全书系统整合了 SVG矢量图形与现代CSS新特性工程 模块的 23 个核心专题（P115 ~ P137）。
建议读者在学完本章后，对照 `notes/` 目录下的思维导图树状笔记进行复盘与知识自测，巩固底层机理与工程实践能力。
