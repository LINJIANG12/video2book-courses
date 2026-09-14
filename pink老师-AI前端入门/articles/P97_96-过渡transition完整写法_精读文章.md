# CSS3 过渡 transition 完整规范与缓动曲线深度剖析

在前面学习基础动画时，大家最熟悉的过渡写法莫过于简写模式：`transition: all 0.5s;`。写上这短短一行，当元素的颜色、宽高或者位移发生变化时，浏览器就会在半秒钟之内平滑演变过去。

但是，当你打开淘宝、京东、苹果等一线大厂的官方网站，按下 F12 审查那些令人赏心悦目的悬浮微交互时，你经常会在样式面板里看到类似这样的代码：
```css
/* 淘宝箭头与卡片悬浮样式节选 */
transition: transform 0.2s ease-in-out;
/* 或者更具弹性质感的缓动函数 */
transition: all 0.8s cubic-bezier(0.25, 0.1, 0.25, 1);
```
很多小伙伴看到这些跟在时间后面的单词以及古怪的四参数函数，心里就会打鼓：“这后面的一长串单词到底是什么？为什么大厂不用我们最简单的写法？”

这一讲，我们就来彻底揭开 CSS3 过渡的完整书写体系，剖析过渡的四大核心分项，搞清楚时间参数双重出现时的底层判定法则，并深入挖掘速度曲线（Timing Function）与三次贝塞尔曲线（Cubic-bezier）的物理运行机制，让你的动效摆脱机械生硬，真正达到商业级产品的丝滑质感。

---

## 1. 过渡 transition 完整书写体系与四要素

### 1.1 复合写法与分项属性映射

一个标准的 CSS3 完整过渡属性由**四个核心部分**协同组成：

```css
/* 完整复合语法骨架 */
selector {
  transition: 要过渡的属性  持续时间  速度曲线(缓动函数)  延迟时间;
}
```

在 CSS 规范中，复合属性 `transition` 实际上是四个独立长命名属性的集合缩写。我们在实际工程中既可以写在一起，也可以按需单独配置：

| 复合项顺序 | 对应独立属性名 | 默认值 | 是否必填 | 属性功能说明 |
| :---: | :--- | :--- | :---: | :--- |
| 第 1 项 | `transition-property` | `all` | 可选 | 指定对哪些 CSS 属性应用过渡效果（如 `width`、`transform` 或 `all`） |
| 第 2 项 | `transition-duration` | `0s` | **必须声明** | 过渡动画从开始到结束所持续的时间总量（必须带单位 `s` 或 `ms`） |
| 第 3 项 | `transition-timing-function`| `ease` | 可选 | 规定过渡运动的速度曲线，控制加速度与运动节奏 |
| 第 4 项 | `transition-delay` | `0s` | 可选 | 状态触发后，等待多久才真正开始执行过渡动画 |

> **结论**：在过渡的四个参数中，**持续时间（duration）是唯一绝不可省略的参数**！因为持续时间的默认值为 `0s`，如果省略不写，意味着过渡在 0 秒内瞬间完成，视觉上与没有写过渡没有任何区别。其余三个参数均有稳健的默认缺省行为。

---

### 1.2 语法解析示例与双时间参数判定法则

我们来看一个包含四个完整参数的典型代码片段：

```css
.box {
  transition: all 1s linear 0.5s;
}
```

这段代码精准传达了四层业务含义：
1. `all`：元素身上所有能够产生渐变效果的 CSS 属性（背景色、边框、坐标等）全部参与过渡；
2. `1s`：整个过渡过程持续耗时 1 秒钟；
3. `linear`：运动过程全程保持匀速运动；
4. `0.5s`：当鼠标悬停或类名激活后，元素并不立即动作，而是静止等待 0.5 秒（延迟触发），随后才开始这 1 秒的匀速过渡。

```text
transition 双时间参数底层识别顺序

代码示例：transition: all 1s linear 0.5s;
         ┌─────┐   ┌───┐   ┌────┐ ┌──────┐ ┌────┐
         │ all │   │1s │   │line│ │0.5s  │ │ ;  │
         └─────┘   └───┘   └────┘ └──────┘ └────┘
            │        │       │       │
            │        │       │       └─ [第二个时间]：永远被解析为 delay（延迟等待）
            │        │       └───────── 速度曲线函数
            │        └───────────────── [第一个时间]：永远被解析为 duration（动画持续）
            └────────────────────────── 目标属性
```
*图 1：浏览器解析 transition 属性中两个时间值的严格先后顺序*

> **易错点**：当 `transition` 声明中同时出现两个时间数值时，**浏览器永远把第一个时间识别为“持续时间（duration）”，把第二个时间识别为“延迟时间（delay）”**！位置绝不可颠倒。如果你写 `transition: 0.5s 1s;`，代表持续时间是 0.5 秒，延迟等待 1 秒。

---

## 2. 速度曲线 transition-timing-function 全景剖析

控制过渡动画是生硬机械还是轻快灵动，核心灵魂在于**速度曲线（缓动函数）**。速度曲线决定了物体在单位时间内的加速度分布。

### 2.1 常见预设关键字及其加速度机理

CSS3 为我们内置了 5 种最常用的标准速度曲线关键字：

```text
5 种预设速度曲线运动速率随时间走势对比

速度(v)
  ↑
  │        ease (慢-快-慢，默认值)              linear (匀速直线，速率恒定)
  │      .-----.                             │------------------
  │     /       \                            │
  │    /         \                           │
  └───'───────────'───→ 时间(t)              └──────────────────→ 时间(t)

  │    ease-in (慢-快，持续加速)             │    ease-out (快-慢，持续减速)
  │              .---                        │    ---.
  │            /                             │        \
  │          /                               │         \
  └───......'─────────→ 时间(t)              └───'──────'───────→ 时间(t)

  │    ease-in-out (慢-快-慢，对称性加减速)
  │         .---.
  │        /     \
  │    ..-'       '-..
  └───────────────────→ 时间(t)
```
*图 2：CSS3 预设速度曲线的加速度变化特征直观示意*

#### 1. `ease`（默认缺省值）
- **物理机理**：慢速起步 -> 迅速加速 -> 慢速缓冲停止（“慢 - 快 - 慢”）。
- **设计场景**：最贴合人类肉眼观察现实世界物体受重力与阻力作用的视觉习惯。如果不写第三项，浏览器默认采用此函数，覆盖了 80% 以上的基础 UI 变换。

#### 2. `linear`（匀速）
- **物理机理**：从头到尾速度完全恒定一致，加速度为 0。
- **设计场景**：没有任何阻尼感与人情味。适合机械化运动场景，如无缝滚动跑马灯、音频播放进度条、雷达匀速扫描线或时钟秒针走动。

#### 3. `ease-in`（加速进场）
- **物理机理**：起始阶段极其缓慢，随后速度不断攀升，以最高速度戛然而止（“慢 - 快”）。
- **设计场景**：符合重力落体或蓄力冲刺的物理特性。适合弹窗关闭、元素坠落离场等动作。

#### 4. `ease-out`（减速刹车）
- **物理机理**：瞬间达到最高初速度，随后开始平滑刹车，最终极其柔和地滑行就位（“快 - 慢”）。
- **设计场景**：非常符合用户“即点即响应”的心理预期。大量用于弹窗滑入、通知栏弹出、下拉菜单展开，给用户极为敏捷的交互反馈。

#### 5. `ease-in-out`（平滑加减速）
- **物理机理**：慢速启动、中段全速加速、终点慢速缓冲。
- **对比辨析**：它与默认的 `ease` 极其相似，都属于“慢-快-慢”，但 `ease-in-out` 的曲线在两端的停滞感更均匀、对称性更强，中间的加速区间也更为紧凑。淘宝、天猫在按钮悬停、箭头朝向翻转时非常青睐 `ease-in-out`，视觉效果比默认 `ease` 更加稳重。

---

## 3. 进阶核心武器：三次贝塞尔曲线 cubic-bezier

### 3.1 贝塞尔曲线的数学构成与物理效果

虽然预设的 `ease`、`linear` 等关键字很方便，但在追求极致微交互的高级 UI 设计中，这些预设曲线往往无法满足特殊的品牌动效需求——比如像弹簧一样的**超调回弹效果（Bounce / Overshoot）**，或者像蓄力一样的**反向倒退启动（Anticipate）**。

这时，我们就必须请出终极武器：`cubic-bezier(x1, y1, x2, y2)`（三次贝塞尔曲线函数）。

> **定义**：CSS 中的贝塞尔曲线是由起点 $(0, 0)$、终点 $(1, 1)$ 以及两个动态控制点 $P_1(x_1, y_1)$ 和 $P_2(x_2, y_2)$ 确定的空间样条曲线。函数接受 4 个浮点数参数，其中 $x$ 坐标必须在 $[0, 1]$ 之间，而 $y$ 坐标可以突破 $1$（产生超越目标的弹性回弹）或小于 $0$（产生先向后退再冲向终点的蓄力）。

```text
Cubic-Bezier 控制点与弹性超调机理

  Y 轴 (动画进度)
   ↑               P1(x1, y1)  [y1 > 1 突破边界产生回弹冲量！]
   │                 *
1.2│               /   \
1.0├─────────────/───────* 终点 (1, 1)
   │           /
   │         /
   │       /
   │     /   * P2(x2, y2)
0.0O───/───────────────→ X 轴 (时间轴 0~1)
  起点(0,0)
```
*图 3：三次贝塞尔曲线中 y 轴坐标突破 1.0 时产生的回弹几何机理*

---

### 3.2 实际开发中的工程作业心法

在生产环境中，没有人会傻乎乎地坐在工位上用数学公式手算这 4 个浮点数。面对 `cubic-bezier`，讲师传授大家三条行之有效的顶级工程工作流：

#### 方案 1：善用 Chrome DevTools 可视化调试手柄（实时所见即所得）
在 Chrome 或 Edge 浏览器中按下 F12 打开开发者工具：
1. 选中带有 `transition` 的元素；
2. 在右侧 Styles 面板中，属性值前会出现一个小巧的“曲线图标”；
3. 点击它，就会弹出一个直观的可视化贝塞尔曲线调节面板；
4. 鼠标直接拖拽控制杆手柄，页面上的动效实时重绘，调出满意的弹性手感后，直接复制面板底部生成的代码。

#### 方案 2：借助专业动效预设工具网站
动效设计师与前端工程师经常使用如下专业网站获取现成的神仙曲线：
- **cubic-bezier.com**：在线拖动手柄并支持多元素实时竞速对比；
- **easings.net**：收集了各种经典弹性算法（如 EaseInBack、EaseOutBack、EaseInOutCubic）的标准贝塞尔参数。

#### 方案 3：大厂优秀动效直接“Ctrl+C / Ctrl+V”
在大厂开发中，优秀的交互细节随处可见。当你在苹果官网、淘宝活动页看到极其惊艳的弹性卡片或翻页效果时：
1. 右键 -> 检查；
2. 找到对应的 `:hover` 规则；
3. 将对方精心调校好的 `cubic-bezier(0.68, -0.55, 0.27, 1.55)` 复制下来，直接粘贴到你的项目样式表中；
4. 建立属于你自己的**动效参数库**。这就是真正高效的前端开发范式！

---

## 4. 速度曲线与贝塞尔弹性对比实战验证

我们通过一段多轨道对比代码，把 5 种预设曲线以及带有回弹效果的三次贝塞尔曲线同台竞技，直观感受它们在运动节奏上的根本区别：

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <title>CSS3 过渡速度曲线与贝塞尔回弹对比</title>
  <style>
    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
    }

    body {
      padding: 40px;
      background-color: #1e272e;
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
      color: #ffffff;
    }

    h2 {
      margin-bottom: 20px;
      font-size: 20px;
      text-align: center;
    }

    .track-wrapper {
      width: 700px;
      margin: 0 auto;
      border: 1px solid #485460;
      padding: 20px;
      border-radius: 8px;
      background-color: #2f3542;
    }

    .track {
      position: relative;
      height: 45px;
      margin-bottom: 15px;
      background-color: #1e272e;
      border-radius: 4px;
    }

    .box {
      position: absolute;
      left: 0;
      top: 0;
      width: 120px;
      height: 45px;
      line-height: 45px;
      text-align: center;
      font-size: 12px;
      font-weight: bold;
      color: #ffffff;
      border-radius: 4px;
      cursor: pointer;
    }

    /* 各方块分别绑定不同的速度曲线（持续时间均为 2 秒） */
    .box-ease {
      background-color: #ff4757;
      transition: transform 2s ease;
    }

    .box-linear {
      background-color: #2ed573;
      transition: transform 2s linear;
    }

    .box-ease-in {
      background-color: #1e90ff;
      transition: transform 2s ease-in;
    }

    .box-ease-out {
      background-color: #ffa502;
      transition: transform 2s ease-out;
    }

    .box-ease-in-out {
      background-color: #9b59b6;
      transition: transform 2s ease-in-out;
    }

    /* 三次贝塞尔曲线：突破边界产生冲出后弹回的果冻感 */
    .box-bezier {
      background-color: #e056fd;
      transition: transform 2s cubic-bezier(0.68, -0.6, 0.32, 1.6);
    }

    /* 悬停整个赛道时，所有方块同步向右平移 550px */
    .track-wrapper:hover .box {
      transform: translateX(550px);
    }
  </style>
</head>
<body>

  <h2>鼠标悬停下方容器观察各速度曲线竞速特征</h2>

  <div class="track-wrapper">
    <div class="track"><div class="box box-ease">ease (默认)</div></div>
    <div class="track"><div class="box box-linear">linear (匀速)</div></div>
    <div class="track"><div class="box box-ease-in">ease-in (加速)</div></div>
    <div class="track"><div class="box box-ease-out">ease-out (减速)</div></div>
    <div class="track"><div class="box box-ease-in-out">ease-in-out</div></div>
    <div class="track"><div class="box box-bezier">cubic-bezier 弹性</div></div>
  </div>

</body>
</html>
```

### 现象观察与总结
1. `linear` 像机器人一样恒速向前，没有丝毫情绪；
2. `ease-in` 刚开始极慢，后面猛地冲刺；
3. `ease-out` 起步瞬间像箭一样射出，在接近终点时缓缓贴合；
4. `box-bezier` 最具看点：它的终点并不是直接停住，而是在到达终点时先向后猛地一拉蓄力，冲过头之后再优雅地弹回目标位置，这种带有“弹性回弹”特质的阻尼动画，正是现代前沿 UI 交互的精髓所在。
