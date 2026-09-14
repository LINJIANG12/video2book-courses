# 104-CSS3 实战：仿 360 极速浏览器首屏星空动效

在前面的章节中，我们分别掌握了关键帧 `@keyframes` 的定制方式以及 `animation` 的 8 个高阶复合属性。俗话说“光说不练假把式”，技术最终是要服务于真实产品的。

很多同学在访问 360 极速浏览器或各大大厂科技感官网时，都会被其震撼的开屏动效所吸引：浩瀚深邃的星空背景铺满整个视口，大小两个发光球体以不同节奏在空中缓缓浮动，右上角的小行星则以极其缓慢的周期来回偏转旋转，科技感与空间感瞬间拉满。

本讲我们将以 **360 极速浏览器首屏实战** 为蓝本，手把手带大家解构并还原这套由背景视口适配、错峰垂直悬浮动画以及超慢速星体旋转所组成的经典企业级动效。

---

## 1. 案例效果解构与动效拆解

在动手写代码之前，先对整体视觉进行模块化拆解：

```text
360 极速浏览器首屏视觉架构
+-------------------------------------------------------------+
| 全屏视口背景 (background: center / cover, overflow: hidden)  |
|                                                             |
|        (○) 大球 (ball2)                                     |
|         │  慢速错峰垂直浮动 (6s alternate)                   |
|         ▼                                                   |
|                                       [★] 卫星/行星 (planet)|
|                                        │  逆时针极慢偏转      |
|    (o) 小球 (ball1)                    ▼  (30s alternate)   |
|     │  匀速垂直浮动 (5s alternate)                          |
|     ▼                                                       |
+-------------------------------------------------------------+
```

### 核心动效要点：
1. **全屏沉浸式背景**：利用 `html, body { width: 100%; height: 100%; }` 铺满窗口，配合 `cover` 缩放自适应任何分辨率屏幕；
2. **多元素复用同一动画**：大小两个球体运动轨迹基本一致，我们**只定义一套关键帧**，通过在不同元素上分配不同的动画时长（如 5s 与 6s），极简实现“错峰摆动”的视差层次；
3. **性能最优解**：坚决摒弃老式的 `margin-top` 位移方案，全面采用 `transform: translateY()`，走 GPU 独立合成层，避免触发全屏重排与卡顿。

---

## 2. 结构搭建与全屏视口背景适配

首先完成 HTML 结构与页面防溢出配置：

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <title>仿 360 极速浏览器首屏动效</title>
  <style>
    /* 1. 视口铺满重置：必须给 html 和 body 同时设置 100% */
    * {
      margin: 0;
      padding: 0;
      box-sizing: border-box;
    }

    html, body {
      width: 100%;
      height: 100%;
    }

    body {
      /* 全屏深色星空背景：居中对齐、等比覆盖、禁止平铺 */
      background: #000 url("images/bg.jpg") no-repeat center center / cover;
      /* 核心避坑：裁剪溢出，防止球体运动到边缘时出现烦人的浏览器滚动条 */
      overflow: hidden;
      position: relative;
    }
  </style>
</head>
<body>
  <!-- 动效主体结构 -->
  <img src="images/ball1.png" alt="小球" class="ball1">
  <img src="images/ball2.png" alt="大球" class="ball2">
  <img src="images/satellite.png" alt="卫星" class="satellite">
</body>
</html>
```

> **重点避坑提示**：
> 1. 单纯给 `body` 设置 `height: 100%` 是无效的，因为其父级 `html` 默认高度由内容撑开，必须同时给 `html, body` 设定 `100%`；
> 2. `background-size: cover` 必须与 `background-position: center` 配套使用，书写为复合形式时中间用斜杠隔开：`center / cover`。

---

## 3. 球体悬浮与行星旋转关键帧开发

### 3.1 定义球体上下悬浮动画（`ballMove`）
```css
@keyframes ballMove {
  0% {
    transform: translateY(0);
  }
  100% {
    /* 向上浮动 45 像素（原版参数） */
    transform: translateY(-45px);
  }
}
```

### 3.2 定义星体缓速旋转动画（`planetRotate`）
```css
@keyframes planetRotate {
  0% {
    transform: rotate(0deg);
  }
  100% {
    /* 逆时针偏转 45 度 */
    transform: rotate(-45deg);
  }
}
```

---

## 4. 元素定位与动画调用（错峰层次设计）

通过绝对定位将三个元素固定在视口对应坐标，并注入动画：

```css
/* 小球定位与动画 */
.ball1 {
  position: absolute;
  left: 20%;
  top: 25%;
  width: 130px;
  /* 5秒 匀速 无限循环 交替往返(跑马灯式) */
  animation: ballMove 5s linear infinite alternate;
}

/* 大球定位与动画：复用同一套动画，但修改时长实现错开 */
.ball2 {
  position: absolute;
  left: 35%;
  top: 45%;
  width: 220px;
  /* 耗时 6秒，比小球稍慢，形成视觉上的深度错落感 */
  animation: ballMove 6s linear infinite alternate;
}

/* 卫星/行星定位与动画 */
.satellite {
  position: absolute;
  right: 15%;
  top: 15%;
  width: 180px;
  /* 30秒 极缓速 匀速 往返旋转 */
  animation: planetRotate 30s linear infinite alternate;
}
```

### 细节对比：为什么不用 `margin-top`？
原版部分老旧代码曾使用 `margin-top` 做位移，但审查元素会发现每一帧都在强制 CPU 重算布局；而我们使用 `transform: translateY`：
- **零重排重绘**：只在合成器线程更新图层坐标；
- **更加平滑**：无论在高性能 PC 还是低配笔记本上，均能稳定维持 60fps 满帧运行。

---

## 5. 本讲小结与下节预告

- **核心要点回顾**：
  - 全屏背景标准写法：`html, body { width: 100%; height: 100%; }` + `background: ... center / cover` + `overflow: hidden`；
  - 动画设计核心思想：**一套关键帧，多处灵活复用**。通过为不同节点配置差异化的 `duration`（5s vs 6s），无需重复写冗余代码即可轻松打造视差层次感；
  - 往返循环黄金组合：`linear infinite alternate`。
- **下一讲预告**：除了这种平滑插值补间动画，我们平时看到的打字机动效、马赛克跳跃、时钟走针以及游戏精灵奔跑是如何实现的？下一讲我们将深入学习 **CSS 动画的步进神器——逐帧动画（`steps()`）**！
