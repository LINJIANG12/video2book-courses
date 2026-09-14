# 103-CSS3 动画 Animation 复合写法与高阶属性全解

在上一讲中，我们通过最基础的 `animation: 动画名 持续时间` 让元素成功动了起来。但大家在实际浏览各类知名网站时会发现，真实场景中的动效要精致得多：
- 比如页面右下角的客服小头像、加载中的 loading 圈，它们一直在不停地匀速旋转；
- 比如手机展示页上的下指指示小箭头，像钟摆一样在空中来回交替往返；
- 又比如某些抽奖转盘或动效卡片，鼠标一放上去（`:hover`）它就立刻定住，鼠标一拿开又继续旋转。

要想随心所欲地控制动画的**循环次数、运动速度、来回方向、停留状态乃至交互暂停**，我们就必须系统掌握 `animation` 的完整属性写法。

---

## 1. 动画八大属性全景清单

CSS3 为 `animation` 提供了 8 个独立属性，也可以简写到单行声明中：

| 属性名 | 作用 | 常见取值 | 默认值 | 复合写法中是否必填 |
| :--- | :--- | :--- | :--- | :--- |
| **`animation-name`** | 指定调用的关键帧动画名称 | 自定义名称（如 `rotate`、`move`） | 无 | **必需（排第 1 位）** |
| **`animation-duration`** | 动画单次播放耗时 | 时间单位（如 `1s`、`500ms`） | `0s` | **必需（排第 2 位）** |
| **`animation-timing-function`** | 速度过渡曲线 | `linear`（匀速）、`ease`（慢-快-慢）、`steps()` | `ease` | 可选 |
| **`animation-delay`** | 动画开始前的延迟等待时间 | 时间单位（如 `0s`、`2s`） | `0s` | 可选 |
| **`animation-iteration-count`** | 动画播放的循环次数 | 数字（如 `1`、`3`）或 **`infinite`（无限循环）** | `1` | 可选 |
| **`animation-direction`** | 播放方向（是否往返交替） | `normal`（正常）、**`alternate`（交替反向）** | `normal` | 可选 |
| **`animation-fill-mode`** | 动画结束或等待时的停滞状态 | `none`、**`forwards`（停在最后一帧）** | `none` | 可选 |
| **`animation-play-state`** | 控制动画运行与暂停 | `running`（运行中）、**`paused`（暂停）** | `running` | **通常单独书写** |

---

## 2. 复合写法核心规则与避坑指南

平时在企业级开发中，我们几乎不会把这 8 个属性拆成 8 行去写，而是采用复合简写：
```css
/* 标准复合书写范例 */
.box {
  animation: move 2s linear 0.5s infinite alternate forwards;
}
```

> **书写顺序铁律**：
> 1. **前两个值的位置必须固定**：第一个必须是 `动画名称`（name），第二个必须是 `持续时间`（duration）；
> 2. 后续的属性（速度曲线、延迟、循环次数、方向、停滞模式）顺序可以任意颠倒，浏览器会根据取值特征（如关键字、时间单位）自动识别；
> 3. **`animation-play-state` 特例**：控制动画暂停的 `paused` 属性**坚决不要塞进复合写法里**，因为它通常伴随 `:hover` 伪类或 JS 交互动态切换，必须单独拆开书写！

---

## 3. 核心高阶属性深度剖析

### 3.1 `infinite` —— 无限循环播放
默认情况下动画只跑 1 次就停下来了。如果要让旋转小太阳、等待指示器永远转下去，只需在复合写法中加上 `infinite`：
```css
/* 让元素永远匀速旋转 */
animation: rotate 1s linear infinite;
```

### 3.2 `alternate` —— 跑马灯交替往返
默认情况下（`normal`），当一次动画从 0% 跑到 100% 结束后，下一次播放会“啪”地一下瞬间瞬移回起点 0%，视觉非常生硬。
而设置了 `alternate` 之后，动画会像钟摆一样：
`0% ➔ 100%（正向） ➔ 0%（反向折返） ➔ 100% ➔ 0%`，来回流畅摆动！

```text
动画播放方向对比
normal:    [起点 0%] ──────────> [终点 100%] | (突变瞬移回起点) [起点 0%] ──────────> [终点 100%]
alternate: [起点 0%] ──────────> [终点 100%] ──────────> [起点 0%] ──────────> [终点 100%]
```

### 3.3 `forwards` —— 动画停留在最后一帧
很多时候我们希望按钮滑入、弹窗出现后就定格在最终状态，不要缩回原点。这时设置 `forwards`：
- 默认 `none`：播放完毕后立刻还原到元素原始初始 CSS 状态；
- `forwards`：播放完毕后，元素将直接**固定在 100% 关键帧的样式**，不会倒退复原。

### 3.4 `paused` —— 鼠标悬停暂停动画
这是大厂网站在轮播图、旋转相册、粒子动效中最常用的交互技巧：
```css
/* 正常状态下无限旋转 */
.circle {
  animation: spin 3s linear infinite;
}

/* 鼠标悬停时暂停，移开后从暂停处继续播放 */
.circle:hover {
  animation-play-state: paused;
}
```

---

## 4. 两个高频工程实战案例

### 4.1 案例一：呼吸指示圈（匀速无限旋转）
```html
<style>
  @keyframes rotateCircle {
    from { transform: rotate(0deg); }
    to { transform: rotate(360deg); }
  }

  .loading-ring {
    width: 60px;
    height: 60px;
    border: 4px solid rgba(0, 0, 0, 0.1);
    border-top-color: #3498db;
    border-radius: 50%;
    /* 核心：1.2秒 匀速(linear) 无限循环(infinite) */
    animation: rotateCircle 1.2s linear infinite;
  }
  
  .loading-ring:hover {
    animation-play-state: paused;
  }
</style>

<div class="loading-ring"></div>
```

### 4.2 案例二：网页下滑引导提示箭头（上下往返浮动）
```html
<style>
  @keyframes floatUpDown {
    0% { transform: translateY(0); }
    100% { transform: translateY(18px); }
  }

  .scroll-down-arrow {
    width: 32px;
    height: 32px;
    background: url("arrow.png") no-repeat center;
    /* 核心：0.8秒 匀速 无限交替往返(alternate) */
    animation: floatUpDown 0.8s ease-in-out infinite alternate;
  }
</style>

<div class="scroll-down-arrow"></div>
```

---

## 5. 本讲小结与下节预告

- **核心规范**：
  - `animation` 前两个参数必须为：`动画名` 和 `时长`；
  - 核心属性值速记：`infinite`（无限转）、`alternate`（来回摇）、`forwards`（停在终点不回来）、`paused`（单写悬停暂停）；
- **下一讲预告**：掌握了动画完整属性后，我们将综合运用 2D 变换与动画，还原经典互联网产品——**仿 360 极速浏览器炫酷首屏开场动效**！
