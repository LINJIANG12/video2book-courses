# clip-path 灯光扫描文字效果

黑底上有一行灰色大字，一道白色灯光从左往右扫过去，被扫到的字瞬间变亮，扫过去又暗回去，像机场跑道灯一样来回跑。你们在很多科技官网的首页都见过，我第一眼就想：这是 Canvas 做的吧？

其实不用那么重，两层同样的文字叠在一起，上面那层用 `clip-path` 开一扇会动的窗，灯光效果就出来了。上一节学的 `inset` 终于派上用场。

## 1. 双层文字的叠放结构

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

## 2. 裁剪窗口的扫描动画

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

## 3. 发光质感的调参

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
