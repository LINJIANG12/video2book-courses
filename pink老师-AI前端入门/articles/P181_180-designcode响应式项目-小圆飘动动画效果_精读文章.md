# DesignCode 响应式项目的小圆飘动动画实现

刚把 DesignCode 首页的头部和卡片区调顺，整个页面看着还是有点“死”，缺一口气。你们有没有这种感觉：布局都对了，配色也有了，可就是不像那些获奖站点那样有呼吸感？我就卡在这里，这才决定把装饰用的小圆点做成飘动的。

我和你们一样，一开始就想给页面加动效，但又怕动效太重拖慢加载。想来想去，这种小圆装饰最合适：实现就几行 `CSS`，不依赖 `JS`，还能让首屏一下子活起来。这一讲我们就把它一次做对。

---

## 1. 小圆装饰的定位结构

先想清楚结构。我在头部大标题区放了两三个小圆，有实心的，有半透明描边的，大小也不一样。它们不能占文档流，否则会把标题挤开，所以统一用绝对定位压在背景层。

```text
header .banner 层叠关系
+----------------------------+
|  .banner (relative)        |
|  +----------------------+  |
|  | h1 + p + btn (正常流) |  |
|  +----------------------+  |
|  (o) .dot1  absolute      |
|      (o) .dot2 absolute   |
|  +----------------------+  |
+----------------------------+
```

读图说明：`.banner` 做定位上下文，文字走正常流，小圆全部绝对定位浮在上层，互不干扰。

代码结构我习惯这样写：

```html
<div class="banner">
  <h1>DesignCode 创意设计课程</h1>
  <p>从 0 到 1 学会响应式与动效</p>
  <a class="btn" href="#">立即学习</a>
  <span class="dot dot1"></span>
  <span class="dot dot2"></span>
  <span class="dot dot3"></span>
</div>
```

```css
.banner {
  position: relative;
  overflow: hidden;
}
.dot {
  position: absolute;
  border-radius: 50%;
  pointer-events: none;
}
.dot1 {
  width: 18px;
  height: 18px;
  background: #5b8cff;
  left: 12%;
  top: 30%;
}
.dot2 {
  width: 32px;
  height: 32px;
  border: 3px solid #ff7a9e;
  opacity: 0.6;
  right: 15%;
  top: 22%;
}
.dot3 {
  width: 12px;
  height: 12px;
  background: #ffc94d;
  left: 48%;
  bottom: 18%;
}
```

> **提示**：`pointer-events: none` 一定要加，否则小圆会挡住按钮点击，你们点半天没反应，还以为是 `JS` 坏了。

> **易错点**：父级忘记写 `position: relative`，小圆就会满屏乱跑，直接以 `body` 为基准了。

---

## 2. 飘动关键帧与错峰控制

光摆在那还不够，得让它们动。我要的是那种轻轻上浮下沉的感觉，不是满屏乱飞，所以只动 `transform: translateY`，不动 `top`，性能也最好。

```css
@keyframes floatY {
  0% {
    transform: translateY(0);
  }
  50% {
    transform: translateY(-18px);
  }
  100% {
    transform: translateY(0);
  }
}
.dot1 {
  animation: floatY 3s ease-in-out infinite;
}
.dot2 {
  animation: floatY 4.2s ease-in-out 0.6s infinite;
}
.dot3 {
  animation: floatY 3.6s ease-in-out 1.1s infinite;
}
```

为什么三个圆我要写三种时长加延迟？你们要是写成一模一样，它们就会像阅兵一样齐上齐下，假得很。错开时长和延迟，每个圆的相位都不一样，看起来才是随意飘的。

```text
时间 -> 3s 周期内位移曲线
dot1 :  /\    /\    (快, 3s)
dot2 : /  \__/  \__ (慢, 4.2s + 延迟0.6s)
dot3 :  _/\_  _/\_  (中, 3.6s + 延迟1.1s)
```

读图说明：周期和起始相位错开后，三个圆在同一时刻处于波形的不同位置，视觉上自然错峰。

> **定义**：位移动画（`transform` animation）指只改变 `transform` 与 `opacity` 的动画，不触发重排，浏览器可以用合成器线程独立渲染。

> **结论**：装饰动效优先用 `transform` 和 `opacity`，绝不用 `top`、`left`、`margin` 做逐帧动画，那会引起整页重排，小圆一动，标题跟着抖。

我还加了一个细节：给大圆多加一层缩放呼吸，让它忽大忽小一点点：

```css
@keyframes floatScale {
  0%, 100% {
    transform: translateY(0) scale(1);
  }
  50% {
    transform: translateY(-22px) scale(1.12);
  }
}
.dot2 {
  animation: floatScale 4.2s ease-in-out 0.6s infinite;
}
```

记住这一条就够了：装饰动效幅度要小，`18px` 到 `22px` 足够了，太大就喧宾夺主，用户只看球不看字了。

---

## 3. 响应式下的收敛处理

做到这里在桌面端很漂亮，一缩到手机宽度就露馅：小圆压住标题，或者撑出横向滚动条。这里容易搞错，很多人只写桌面一套就交差了。

我的做法是两手：

```css
@media (max-width: 768px) {
  .dot2 {
    display: none;
  }
  .dot1 {
    width: 12px;
    height: 12px;
    left: 6%;
    top: 18%;
  }
  .dot3 {
    left: auto;
    right: 8%;
    bottom: 12%;
  }
}
```

一是藏起最碍事的大圆，二是把剩下的缩小并往边角赶。配合父级的 `overflow: hidden`，小屏下再也不会出现横向滚动条。

| 断点 | 处理策略 | 原因 |
| :--- | :--- | :--- |
| `> 992px` | 三圆全显示，原尺寸飘动 | 空间充足，装饰拉满 |
| `<= 768px` | 藏大圆，其余缩小靠边 | 防止压字与横向溢出 |
| `<= 480px` | 进一步减小动画幅度 | 小屏下大幅位移很晃眼 |

你们照着改完，刷新从大屏一路拖到手机宽度看一遍：标题始终可读，按钮始终可点，小圆始终是配角，这才算合格。下一讲我们回头收那两个遗留 `bug`，页面就真正能上线了。
