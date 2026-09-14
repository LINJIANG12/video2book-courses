# SVG描边动效实战：仿OPPO AI里程碑时间线案例

在现代科技品牌（如 OPPO、苹果、华为）的官方产品页中，我们经常能看到极具科技感的时间线与里程碑动效：当页面加载或用户向下滚动时，一条精致的曲线轨迹像光纤一样向前延展绘制；当轨迹延伸到特定年份或里程碑节点时，对应的信息卡片由隐至显、伴随微动效浮现出来。

这一讲我们把前面学过的 SVG 描边动画（`stroke-dasharray` 与 `stroke-dashoffset`）融入真实的商业级项目，完整推导并实现这一套纯 CSS 驱动的里程碑时间线交互体系。

---

## 1. 案例效果与结构层次拆解

### 1.1 动效时序与视觉流程

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

### 1.2 商业页面矢量素材提取技巧

在日常前端开发中，复杂曲线并非由程序员手写坐标，而是由 UI 设计师在 Figma、Illustrator 等软件中导出，或直接从现存优秀案例中提取：

1. 打开目标官网（OPPO AI 官方介绍页），按 `F12` 或右键点击页面选择“检查”；
2. 在 Elements 面板中定位到承载曲线的 `<svg>` 节点；
3. 右键该节点，选择 `Copy` -> `Copy element`，整段矢量代码即完整进入剪贴板；
4. 粘贴到项目对应的 HTML 结构容器中。

> **提示**：矢量素材复制到本地后，务必检查 `<svg>` 内的核心 `<path>` 标签，为了后续能够精准施加样式和动效，我们通常手动为该路径添加语义化标识，例如 `id="line"`。

---

## 2. 布局骨架搭建与定位层级

### 2.1 容器架构与尺寸契约

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

### 2.2 绝对定位与坐标排布

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

## 3. 测量路径周长与描边机理

### 3.1 JavaScript 控制台实测路径总长

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

### 3.2 虚线间隔与偏移原理

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

## 4. 节点卡片交错进场动效与时间控制

### 4.1 初始可见性与位移动画设计

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

### 4.2 动画双重时间参数解析

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

## 5. 局部滚动体验与完整整合工程

### 5.1 视口内滚动优化

商业网站通常避免因为子模块过长而导致整个浏览器窗口横向被撑爆出现系统级滚动条。我们在 `.box .scroll` 上配置：

```css
overflow: auto;
```

当内容（`3858px` 的 SVG 轨迹与卡片）超过外层父盒子（`1300px`）时，滚动条只出现在 `.scroll` 内部，保持整体页面栅格严谨。

### 5.2 完整代码示例

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

## 6. 核心知识点与开发避坑总结

| 关注维度 | 关键技术实现 | 核心避坑指引 |
| :--- | :--- | :--- |
| **路径求长** | `line.getTotalLength()` | 必须在真实 DOM 节点渲染后调用；取整时向上微调 1px，切勿向下舍入。 |
| **描边隐藏** | `stroke-dasharray` + `stroke-dashoffset` | 两个属性必须设为相同的路径总长值，初始偏移让实线完全处于视线之外。 |
| **延时触发** | `animation: name 1s linear forwards 1s` | 明确区分持续时间与延迟时间，必须声明 `forwards` 保持最终展示状态。 |
| **滚动隔离** | `overflow: auto` 作用在超宽内层父级 | 避免超长 SVG（3858px）直接把主页面 body 撑宽造成横向布局溃散。 |
