# 笔记04 视觉动效与现代CSS新特性工程

> 整理自《2025版pink老师最新AI+前端入门教程》P92~P137单集精读长文
> 渲染支持：Typora / VS Code Markmap / XMind 一键脑图

```text
视觉动效与现代CSS新特性工程 (P92-P137)
├── [1] 2D变换与过渡 (P92-P98)
│   ├── 平移translate零回流        ──── 合成层零回流位移
│   ├── 旋转缩放倾斜基准           ──── 角度倍数切变与原点
│   └── 过渡缓动与复合顺序         ──── 四参数曲线与空格连写
├── [2] 3D变换与透视空间 (P99-P102)
│   ├── 左手法则与透视视距         ──── 定正向与800-1200px
│   ├── 双面翻转与背面剔除         ──── 预转180与hidden
│   └── 位移悬浮与空间保留         ──── translateZ与亲爸保留
├── [3] 关键帧动画与逐帧控制 (P103-P106)
│   ├── 定义调用与八属性           ──── keyframes与八参数
│   ├── 360首屏错峰复用            ──── 一套帧多时长交替
│   └── steps精灵图位移            ──── 步数等于帧数
├── [4] 七类商业动效模式 (P107-P113)
│   ├── 流光边框与旋转光盘         ──── 200宽与超大盘
│   ├── 折叠手风琴与层叠卡         ──── has与倾斜平移
│   └── 无缝滚动风琴相册与翻转导航 ──── 双组接力与镜像偏转
├── [5] SVG矢量与描边动画 (P115-P119)
│   ├── 内联引入与填充描边         ──── inline与fill电流
│   ├── 虚线生长与偏移画线         ──── 周长实测与偏移
│   └── 里程碑分组巡航             ──── g分组与溢出可见
├── [6] 裁剪滤镜与圈数单位 (P120-P124)
│   ├── clip-path形状与扫描窗      ──── 四函数与双层窗
│   ├── filter与backdrop-filter    ──── 自身与背后加工
│   └── turn圈数与花瓣布局         ──── 1圈与盘心原点
├── [7] 滚动驱动时间线 (P125-P126)
│   ├── scroll全局行程映射         ──── 页面进度条
│   └── view独立可见映射与区间     ──── 入场与entry区间
├── [8] 变量计算与精灵图公式 (P127-P129)
│   ├── 变量声明引用与换肤         ──── 双横线与var兜底
│   └── calc混算与序号公式         ──── 空格与乘数规则
└── [9] 综合导航与滚动叠加 (P130-P137)
    ├── 动感菜单与滑动滑块         ──── 变量位移与兄弟选择
    ├── 毛玻璃导航与圆点指示       ──── 模糊底座与伪元点
    └── 视口粘性层叠与视图收缩     ──── vh sticky与view缩放
```

---

## 1. 2D变换与过渡

> **一句话主旨**：规定平移旋转缩放倾斜的单位方向与原点，并用过渡曲线与复合顺序驱动悬停动效。

### 1.1 平移与悬停抽屉

* **平移（Translate）**
    * > 定义：`translate()`为改变元素渲染位置的2D函数，只改合成层投影，不改文档流物理占位。
    * 网页坐标系为X向右为正，Y向下为正，向上位移取负值。
    * 语法为`translate(x,y)`、`translateX(x)`、`translateY(y)`，轴名字母严格大写，长度带单位。
    * `translate`百分比相对元素自身尺寸，与父容器解耦，`translate(-50%,-50%)`配合`left:50%;top:50%`实现未知尺寸居中。
    * 修改`margin`或`top/left`触发回流与重绘，`translate`走合成线程与GPU加速，保持60FPS且不挤动兄弟元素。
    * > 易错点：`bottom`语义为距父底边距离，取正上移，取负下沉；抽屉高83px时初始须写`bottom:-83px`。
    * > 来源: P92, P93
* **透明边框与抽屉联动（Transparent Border， Drawer）**
    * > 定义：以预埋透明边框锁定悬停前后盒模型尺寸，以负位移抽屉实现无回流滑入的电商卡片模式。
    * 初始声明`border:2px solid transparent`，悬停只改`border-color`，尺寸不变。
    * 抽屉结构为父`li{position:relative;overflow:hidden}`包裹子`.mask{position:absolute;left:0;bottom:-83px}`。
    * 悬停选择器须写后代关系`.goods-list li:hover .mask`，动效为`transform:translateY(-83px)`。
    * 过渡写在动效元素原身，移入移出双向平滑；只写在`:hover`则移出瞬间复位。
    * 超链接（Anchor）不可嵌套，卡片主链接与找相似链接须为兄弟节点。
    * > 来源: P93

### 1.2 旋转缩放倾斜与原点

* **旋转（Rotate）**
    * > 定义：`rotate()`为绕变换原点偏转角度的2D函数，单位为度（Degree，deg）。
    * 参数必须带`deg`，`rotate(360)`非法；正值顺时针，负值逆时针。
    * 未加过渡时360度旋转在0秒完成，视觉等同未转，须配`transition`才可见过程。
    * `transition`写原身则往返平滑，写`:hover`则进入平滑、移出0秒瞬时复位。
    * 变换原点（Transform Origin）默认`50% 50%`，可用方位词`left top`、像素、百分比改枢纽。
    * 纯行内元素`display:inline`无盒模型边界，不响应`transform`，须转`display:inline-block`。
    * > 来源: P94
* **缩放（Scale）**
    * > 定义：`scale()`为以变换中心为基准缩放视觉位图的函数，不改变文档流占位。
    * 直接改`width/height`触发回流并挤动下方兄弟元素，悬停放大禁用该方案。
    * 单参数`scale(1.5)`为等比150%，小于1为缩小；双参数`scale(x,y)`分轴控制。
    * 参数为无单位纯数字，禁带`px/%/deg`；负值附带镜像翻转。
    * 默认以中心向四周辐射膨胀，视觉遮盖邻位但不推动布局。
    * 企业级图片放大须用相框结构：父`.img{overflow:hidden}`定尺寸裁剪，子`img{transition:all 0.3s}`承接`scale(1.2)`，悬停绑在卡片`li`上。
    * > 来源: P95
* **倾斜（Skew）**
    * > 定义：`skew()`为沿X或Y轴切变矩形的函数，参数单位须为`deg`。
    * `skew(x-angle,y-angle)`双轴复合，单参数`skew(x)`等价`skew(x,0)`，不等同`scale`的等比同步。
    * `skewX(20deg)`顶部向左倒，`skewX(-16deg)`顶部向右倒，对应游戏按钮标准形态。
    * 默认原点为中心时`skewY`两侧一沉一翘，左上顶点脱位；折叠场景须定`transform-origin:top left`锁定装订线。
    * 折叠卡片结构为`.card{position:relative}`内叠`.front/.back{position:absolute;top:0;left:0}`，非对称圆角`border-radius:0 30px 40px 40px`。
    * 后层初始`skewY(8deg)`，悬停增至`skewY(15deg)`并收窄宽度，前层`translateY(-3px)`形成视差。
    * 父倾斜会带偏内部文字，解法为子反向`skewX(16deg)`抵消，或伪元素`::before`承载倾斜背景。
    * > 来源: P96

### 1.3 过渡曲线与复合顺序

* **过渡（Transition）**
    * > 定义：`transition`为声明属性渐变耗时、曲线与延迟的复合属性，挂载位置决定进退是否平滑。
    * 完整顺序为`属性 持续时间 速度曲线 延迟时间`，对应`transition-property/duration/timing-function/delay`。
    * 持续时间必填，缺省`0s`等同无过渡；其余缺省为`all/ease/0s`。
    * 双时间值中第一个解析为持续时间，第二个解析为延迟时间。
    * 五种预设为`ease`慢快慢默认、`linear`匀速、`ease-in`慢快、`ease-out`快慢、`ease-in-out`对称慢快慢。
    * 三次贝塞尔曲线（Cubic Bezier）为`cubic-bezier(x1,y1,x2,y2)`，x限于0到1，y可超1回弹、可负向蓄力。
    * 工程取参依靠开发者工具曲线面板、`cubic-bezier.com`、`easings.net`或复用大厂调参。
    * > 来源: P97
* **复合变换（Multiple Transforms）**
    * > 定义：复合变换为同一`transform`内空格连写多函数的写法，后写覆盖前写的分行声明非法。
    * 函数间严格空格分隔，禁用逗号；同属性分行写仅末行生效。
    * 执行顺序从右向左，先执行最右侧函数；变换会改变本地坐标系。
    * `translate(...) rotate(...)`为先自转后平移，轨迹为平滑滚动；`rotate(...) translate(...)`为先平移后旋转，轨迹为绕远点大弧甩动。
    * 汽车案例须用容器包裹法：父`.car`只做`translateX`宏观位移，子`.wheel`只做`rotate`原地自转，避免三元素各自位移脱节。
    * 初始含`translateY(-50%)`垂直居中时，悬停须全量保留写作`translateX(-500px) translateY(-50%)`，否则居中丢失下坠。
    * 向左行驶配逆时针负角度如`rotate(-720deg)`，车身与车轮过渡时长与曲线须一致，如`3s cubic-bezier(0.25,0.1,0.25,1)`。
    * ```css
      .viewport-box:hover .car { transform: translateX(-500px) translateY(-50%); }
      .viewport-box:hover .wheel { transform: rotate(-720deg); }
      ```
    * > 来源: P98

---

## 2. 3D变换与透视空间

> **一句话主旨**：以左手法则定三轴正向，以透视与空间保留构建翻转悬浮所需的立体渲染链。

### 2.1 坐标透视与旋转正向

* **三维坐标系（3D Coordinate System）**
    * > 定义：Z轴为垂直屏幕平面的法线轴，正向穿出屏幕指向观察者，负向深入屏幕内部。
    * X正向向右，Y正向向下，Z正向向外；Y向下为正区别于数学坐标系。
    * > 来源: P99, P100
* **左手法则（Left-Hand Rule）**
    * > 定义：以左手拇指对准轴正向、四指弯曲指向判定旋转正向的空间法则。
    * 拇指指右判`rotateX`，拇指朝下判`rotateY`，拇指指自己判`rotateZ`；顺四指为正，逆四指为负。
    * `rotateX(30deg)`为上沿后仰、下沿前迎；`rotateY(45deg)`为右侧后退、左侧前凸；`rotateZ`等同2D顺时针为正。
    * 生活模型为X单杠翻滚、Y钢管旋转门、Z风扇扇叶自转。
    * > 来源: P99, P100
* **透视（Perspective）**
    * > 定义：`perspective`为模拟人眼到屏幕视距的投影属性，数值即视距，产生近大远小。
    * 无透视为正交平行投影，旋转只现挤扁；有透视远边缩短、近边放大。
    * 视距越小透视越夸张，越大越扁平；工程推荐`800px~1200px`或`500px`。
    * 父级写法`.parent{perspective:500px}`建立统一相机空间，多子共享视点。
    * 单元素写法`transform:perspective(500px) rotateY(45deg)`，`perspective()`必须为首函数，写后则失效。
    * ```css
      .stage { perspective: 500px; }
      .single-test { transform: perspective(400px) rotateY(30deg); }
      ```
    * > 来源: P100

### 2.2 翻转盒子与悬浮纵深

* **两面翻转盒子（Flip Card）**
    * > 定义：以前后双绝对定位子层背靠背贴合、协同偏转实现正反内容切换的3D组件。
    * 结构为`.box{position:relative;perspective:1000px}`内含`.front/.back{position:absolute;top:0;left:0;width:100%;height:100%}`。
    * 背面初始须预转`transform:rotateY(180deg)`，否则翻至正面呈镜像倒字。
    * 悬停目标为正面`rotateY(-180deg)`、背面`rotateY(0deg)`，双层同向同步构成无缝翻面。
    * 后写元素默认压前写元素，初始须给`.front{z-index:1}`保正面可见。
    * 背面可见性（Backface Visibility）取`visible`默认可见、`hidden`背向隐藏；须挂子层自身，挂父容器无效。
    * ```css
      .box .front, .box .back { transition: all 0.7s; backface-visibility: hidden; }
      .box .back { transform: rotateY(180deg); }
      .box:hover .front { transform: rotateY(-180deg); }
      .box:hover .back { transform: rotateY(0deg); }
      ```
    * > 来源: P101
* **三维位移与空间保留（TranslateZ， Preserve-3D）**
    * > 定义：`translateZ/translate3d`为沿深度轴的位移函数，`transform-style:preserve-3d`为保留子层独立立体坐标的开关。
    * `translate3d(x,y,z)`三参缺一不可，无位移轴补`0`；大厂常用`translate3d(0,-2px,0)`强制GPU合成层，绕开回流重绘。
    * `translateZ(60px)`向前扑近放大，负值后退缩小；无祖先透视则无近大远小感知。
    * 默认`transform-style:flat`将子层拍扁于父平面，子层`translateZ`失效。
    * `preserve-3d`须加直接父级，文字之父为`.front/.back`而非最外`.box`，只加最外层不生效。
    * 悬浮文字须同步`backface-visibility:hidden`，否则翻至背面时高处文字穿透外露。
    * 配合关系为外舞台定`perspective`，旋转层定`rotate`，悬浮层定`translateZ/3d`，亲父定`preserve-3d`，旋转图元定`hidden`。
    * > 来源: P102

---

## 3. 关键帧动画与逐帧控制

> **一句话主旨**：用关键帧定义多节点状态，用复合属性控循环方向定格，再用步进函数驱动精灵图。

### 3.1 定义调用与完整属性

* **关键帧动画（Animation， Keyframes）**
    * > 定义：关键帧为时间轴状态锚点集合，浏览器对锚点间自动插值，突破过渡仅首尾两态限制。
    * 相对脚本动画具备合成线程加速、免手写缓动、多节点、无需交互即可自动播放特征。
    * 标准两步为先`@keyframes 动画名{0%{}50%{}100%{}}`定义，后`animation:动画名 时长`调用。
    * 百分比可任意多节点，两态可用`from/to`简写；`100%`同`0%`可构成闭环归位。
    * 帧内可复合`transform/border-radius/background-color/opacity`协同插值。
    * 调用两必填为名称大小写敏感匹配、时长带`s/ms`，缺时长默认为0不播放。
    * ```css
      @keyframes squareToCircle { 0% { transform: translate(0,0); } 100% { transform: translate(0,0); } }
      .target-box { animation: squareToCircle 4s; }
      ```
    * > 来源: P103
* **动画复合属性（Animation Properties）**
    * > 定义：动画复合属性为八项控制的集合，前两项位置固定，暂停项须独立书写的控制体系。
    * 八项为`name/duration/timing-function/delay/iteration-count/direction/fill-mode/play-state`。
    * 复合前两值须为名称与时长，后续顺序可凭取值特征识别；`paused`禁塞复合，须单独配`:hover`切换。
    * `infinite`为无限循环，`alternate`为往返摆动区别于`normal`瞬移回起点。
    * `forwards`定格100%帧，`none`播完回初始；`paused`悬停暂停后从断点续播。
    * ```css
      .box { animation: move 2s linear 0.5s infinite alternate forwards; }
      .circle:hover { animation-play-state: paused; }
      ```
    * > 来源: P104

### 3.2 首屏复用与逐帧原理

* **首屏悬浮复用（First-Screen Floating）**
    * > 定义：以同一套位移关键帧配差异时长实现错峰视差的首屏多元素调度方法。
    * 全屏须同时`html,body{width:100%;height:100%}`，背景用`center/cover`配`overflow:hidden`防球体溢出滚动条。
    * 球体关键帧为`translateY(0)`至`translateY(-45px)`，大小球复用同一`ballMove`但时长取`5s`与`6s`。
    * 行星关键帧为`rotate(0deg)`至`rotate(-45deg)`，调用`30s linear infinite alternate`极缓偏转。
    * 位移禁用`margin-top`，须用`translateY`保合成层60FPS。
    * > 来源: P105
* **逐帧动画（Frame-by-Frame Animation， Steps）**
    * > 定义：逐帧动画为以`steps(n)`瞬时切换代替平滑补间，透过定尺寸视口逐格露出精灵图的动画。
    * 三要素为视口盒等于单帧尺寸、超宽横排精灵图、`background-position-x`阶梯左移。
    * `steps(n)`中n严格等于横向帧数，25帧写`steps(25)`；补间会将整图拖影拉过。
    * 案例尺寸为视口`548px×513px`，全图`13700px×513px`，`13700/548=25`，原图高度不符须`background-size:13700px 513px`校准。
    * 关键帧为`background-position:0 0`至`-13700px 0`，负值左拉，调用`animation:playSprite 1s steps(25) infinite`。
    * 相对GIF具备32位Alpha细腻边缘、时长可改、支持暂停反向跳转、单图减少请求特征。
    * 步数错位导致帧偏移，盒尺寸须与单帧严丝合缝。
    * > 来源: P106

---

## 4. 七类商业动效模式

> **一句话主旨**：以遮罩错位与选择器联动实现边框折叠滚动相册导航，覆盖七个高频商业动效。

### 4.1 边框折叠与层叠展开

* **流光渐变边框（Gradient Border Flow）**
    * > 定义：以外层留缝、夹层双倍渐变位移、内层遮罩三层叠合伪造边框流动的障眼结构。
    * 外层`a{padding:3px;overflow:hidden}`以边距充当3px边框厚度并裁剪溢出。
    * 夹层`::before{width:200%;background-size:50% 100%}`配`translateX(-50%)`实现无缝循环，走一半留一半不断档。
    * 动画为`0.75s linear infinite`，悬停可缩至`0.4s`加速；内层`span{position:relative;z-index:1;background:#000}`压住定位伪元素。
    * 圆角遵从内圆角等于外圆角减厚度，如外`6px`内`3px`；文字`line-height`取总高减双倍边距，如60px取54px。
    * ```css
      @keyframes streamMove { to { transform: translateX(-50%); } }
      ```
    * > 来源: P107
* **旋转流光边框（Rotating Glow Border）**
    * > 定义：以超大旋转光盘加内衬遮罩仅露边缘光晕的四层旋转边框结构。
    * 容器`.card{position:relative;overflow:hidden}`定尺寸圆角，`::before{width:500px;height:500px}`须远大于卡片防转角露怯。
    * 光盘背景取锥形渐变如`conic-gradient(transparent 70%,#00ffff,#ff007f)`，动画`rotate 3s linear infinite`。
    * 遮罩`::after{position:absolute;inset:3px;background:#12131a}`等价四边3px，`inset`多值规则同`margin/padding`。
    * `::before/::after`加`z-index:-1`沉于标准流文字下，免逐个提权文字。
    * 超大盘居中依赖父`flex`吸附几何中心，硬算偏移在响应式下偏心。
    * > 来源: P108
* **折叠手风琴（Accordion Fold， Has）**
    * > 定义：以网格比例切换加父选择器逆向命中实现纯CSS手风琴展开的布局动效。
    * 父网格默认`grid-template-columns:2fr 1fr 1fr 1fr`，悬停目标列切为`2fr`，配`transition:grid-template-columns 0.5s`。
    * `:has()`语义为命中含特定状态子的父级，如`ul:has(.item:nth-child(2):hover)`改整表比例。
    * 标题初始`bottom:-100px`藏于`overflow:hidden`卡片下，悬停切`bottom:0`上浮。
    * 卡片须`background-position:center;background-size:cover`防展宽拉伸变形。
    * 兼容为Chrome105+、Safari15.4+、Edge105+、Firefox121+。
    * > 来源: P109
* **倾斜层叠卡片（Tilted Stack Cards）**
    * > 定义：以统一倾斜初态加悬停抹平错位平移实现多卡协同展开的空间动效。
    * 通配初态为`.box>*{transform:rotateY(-20deg) rotateX(20deg)}`，顺序先Y后X不可颠倒。
    * 悬停五卡分别`translate3d(-30px,-30px,0)`、`translate3d(0,-30px,0)`、`translate3d(0,0,0)`、`translate3d(-120px,60px,0)`、`translate3d(30px,30px,0)`。
    * 位移优先`translate3d`开GPU层，过渡取弹簧`cubic-bezier(0.34,1.56,0.64,1)`。
    * 毛玻璃卡取`background:rgba(255,255,255,0.12)`配`backdrop-filter:blur(16px)`及`-webkit-`前缀。
    * > 来源: P110

### 4.2 滚动相册与立体导航

* **无缝滚动品牌墙（Seamless Marquee）**
    * > 定义：以AB双组克隆加精确位移实现跳帧不可见的无限横向滚动结构。
    * 外视口`.box{width:1180px;overflow:hidden}`，轨道`.scroll{display:flex;width:max-content;animation:seamlessMove 20s linear infinite}`。
    * 单组`.ad{width:1180px;flex-shrink:0}`禁压缩，组间距`margin-right:20px`。
    * 总位移须含间距，公式为单组宽加间距，即`1180px+20px=1200px`，写1180px交接抽搐20px。
    * 悬停暂停为`.box:hover .scroll{animation-play-state:paused}`；位移须`translateX`禁`margin-left`。
    * $$总位移=单组宽度+组间距=1200px$$
    * > 来源: P111
* **悬停风琴相册（Hover Organ Album）**
    * > 定义：以否定与兄弟选择器分区实现主角放大、两侧镜像偏转的向心相册。
    * 舞台须父级`perspective:500px`统一视点，写子层则各自分割失效。
    * 倒影为`-webkit-box-reflect:below 2px linear-gradient(transparent 60%,rgba(0,0,0,0.7))`。
    * 非主角默认为`.box:hover .item:not(:hover){transform:scale(0.9) rotateY(45deg)}`。
    * 右侧覆盖为`.box .item:hover~.item{transform:scale(0.9) rotateY(-45deg)}`，主角为`scale(1.25) rotateY(0deg);z-index:999`。
    * 进入聚拢用`.box:hover .item{margin:0 -15px}`负边距重叠。
    * > 来源: P112
* **翻转立体导航（Flip Navbar）**
    * > 定义：以前底双面半高偏移建模、整体绕X上翻90度切换的长方体导航。
    * 单项尺寸如`90px×44px`半高22px，父`li{position:relative;transform-style:preserve-3d;transition:transform 0.4s}`。
    * 前面`translateZ(22px)`，底面`translateY(22px) rotateX(-90deg)`，先平移后旋转不可颠倒。
    * 悬停整体`li:hover{transform:rotateX(90deg)}`，前面上翻顶部、底面翻至正面。
    * 双面须父相子绝重叠，缺`preserve-3d`则压扁失效。
    * > 来源: P113
* **作业通用法则（Homework Rules）**
    * > 定义：以合成层属性与几何推导沉淀动效作业的通用验收约束。
    * 微浮取`translateY(-6px)`配`box-shadow:0 12px 24px rgba(0,0,0,0.15)`，纵深取`translateZ(60px)`。
    * 显隐禁用`display`过渡，须`opacity:0;pointer-events:none`切`opacity:1`配`transition:opacity 0.3s`。
    * 气泡尖角为`12px`正方形`rotate(45deg)`半嵌底部，如`.bubble::after{bottom:-6px;left:50%;transform:translateX(-50%) rotate(45deg)}`。
    * 悬浮弹窗初始`translateX(-50%) translateY(10px);opacity:0;visibility:hidden`，悬停归位显现。
    * > 来源: P114

---

## 5. SVG矢量与描边动画

> **一句话主旨**：以内联结构取得路径控制权，以实测周长驱动虚线生长与偏移绘制。

### 5.1 内联引入与描边生长

* **矢量引入（Scalable Vector Graphics， SVG）**
    * > 定义：SVG为基于数学方程的矢量规范，放大不失真且支持代码级修改填充描边与动画。
    * 相对位图无马赛克，相对字体图标支持多色与路径交互，体积多为数百字节。
    * `img`引入为沙箱闭包不可穿透控`path`，动效须内联粘贴`svg`源码。
    * 结构为`svg`画布、`viewBox`视区、`path d`路径、`fill`填充、`stroke`描边；尺寸须`width/height`，`font-size`无效。
    * 图标库`path`常硬编码`fill`，外部改色须删源码属性或`fill:red !important`。
    * B站弹跳为`li:hover svg{animation:iconBounce 0.35s ease-out}`，帧为`translateY(0/-4px/0)`。
    * 联动换色为`svg{fill:currentColor}`，父`color`变则填充同步变。
    * > 来源: P115
* **描边生长（Stroke Drawing）**
    * > 定义：以虚线实部长短变化模拟手绘生长的SVG描边动效，终态须定格保留。
    * 专有属性为`fill/stroke/stroke-width/stroke-dasharray/stroke-dashoffset`。
    * 单值`dasharray:100`为实线间隙各100px，双值`100 50`为实线100间隙50。
    * 生长帧为`0%{stroke-dasharray:0 300}`至`100%{stroke-dasharray:300 0}`，初始全间隙隐形，终态全实线。
    * 素材须单线条实心轮廓，空心双边会现双轨道；收尾须`forwards`，线帽宜`stroke-linecap:round`。
    * ```css
      .building-icon { fill: none; stroke: #ffd700; stroke-width: 4; }
      .pku-card:hover .building-icon { animation: drawStroke 0.8s ease-in-out forwards; }
      ```
    * > 来源: P116

### 5.2 偏移画线与商业巡航

* **偏移画线（Dash Offset Drawing）**
    * > 定义：以固定虚线长等于路径周长、滑动偏移量实现的标准手绘绘制方法。
    * 三步为`dasharray`定周长L，初始`dashoffset:L`推出隐形，动画减至0拉回显形。
    * 反向0至L为擦除；80%画线配20%`fill`补间可先线后填色。
    * 周长须`id`加路径后控制台执行`getTotalLength()`实测，爱心案例取`2783`，CSS取整向上宁多勿少。
    * ```css
      #myHeart { fill: none; stroke: #ff2d55; stroke-width: 8; stroke-dasharray: 2783; stroke-dashoffset: 2783; animation: drawHeart 3s linear infinite; }
      @keyframes drawHeart { 0% { stroke-dashoffset: 2783; } 100% { stroke-dashoffset: 0; } }
      ```
    * > 来源: P117
* **里程碑时间线（Milestone Timeline）**
    * > 定义：以超宽SVG路径描边加延迟卡片浮现构成的纯CSS里程碑叙事结构。
    * 布局为外`.box{width:1300px}`视口加内`.scroll{overflow:auto}`滑轨，卡片绝对定位于曲线锚点旁。
    * 素材由检查目标页`svg`节点复制，补`id="line"`后施样式。
    * 实测`line.getTotalLength()`得3857，CSS取`3858px`同时赋`dasharray/dashoffset`，动画`lineDraw 10s linear forwards`由3858减至0。
    * 卡片初始`opacity:0`，`item2{animation:translateUp 1s linear forwards 1s}`上浮40px，`item3{animation:translateDown 1s linear forwards 2s}`下沉40px。
    * 复合双时间中前者为持续后者为延迟，缺`forwards`则播完闪回消失；超宽内容须内层滚动防撑爆body。
    * > 来源: P118
* **分组巡航（Group Cruise， G）**
    * > 定义：以`g`分组将复合SVG拆为可独立动画的积木层，对动态组直接绑定变换的工程方法。
    * `g`等价`div`，设计软件编组导出即该标签，加`id/class`即可单控位移旋转。
    * 动态飞出画布会被默认裁剪，须`svg{overflow:visible}`放行。
    * 无人机组`#drone{animation:droneCruise 12s ease-in-out infinite}`，帧含多航点`translate`加`rotate(0/5deg/-6deg)`闭环。
    * 桨叶可另绑`rotate`极速自转，`g`支持`:hover`与JS事件直绑。
    * > 来源: P119

---

## 6. 裁剪滤镜与圈数单位

> **一句话主旨**：用裁剪窗造异形与扫描光，用两类滤镜分加工对象，再用圈数简化旋转表达。

### 6.1 裁剪形状与扫描光窗

* **裁剪（Clip Path）**
    * > 定义：裁剪为在元素上蒙剪纸框的像素丢弃机制，框外不渲染且不响应鼠标，区别于父藏子的溢出隐藏。
    * 四函数为`circle(半径 at x y)`、`ellipse(横 纵 at x y)`、`inset(上 右 下 左 round 圆角)`、`polygon(各点)`。
    * 百分比坐标纸原点左上，横右纵下0至100%；菱形取4点，箭头取6点，顺序连线自动闭合。
    * `inset(20px)`为内缩可见变小；`inset(0 100% 0 0)`藏背景配合悬停展为`inset(0 0 0 0)`。
    * 过渡要求首尾同函数同点数，圆变圆可插值，圆变多边形跳变；复杂形宜在线生成器拖拽后复制。
    * ```css
      .triangle { clip-path: polygon(50% 0, 0 100%, 100% 100%); }
      .diamond { clip-path: polygon(50% 0, 100% 50%, 50% 100%, 0 50%); }
      ```
    * > 来源: P120
* **扫描文字（Scan Light Text）**
    * > 定义：以双层同字重叠、顶层裁剪窗移动点亮的灯光扫描结构。
    * 容器`.wrap{position:relative}`定字号，底层灰字静止，顶层光字`position:absolute;left:0;top:0`精确重合。
    * 两层须同字体字号字距，字号挂容器继承，`white-space:nowrap`防换行错位。
    * 矩形窗帧为`inset(0 100% 0 0)`至`inset(0 0 0 0)`再至`inset(0 0 0 100%)`，前半展开后半收起。
    * 斜光窗为等点数`polygon`斜四边形由左侧外部移至右侧外部；移动为窗非字，字禁动。
    * 光质为`color:#fff`配`text-shadow:0 0 8px/25px`青白双层，动画`3s linear infinite`。
    * > 来源: P121

### 6.2 滤镜圈数与花瓣旋转

* **滤镜（Filter）**
    * > 定义：滤镜为对元素自身渲染结果的末端像素加工，子内容同步受影响的整体外观函数。
    * 基准1系列为`brightness/contrast/saturate`，1为原样，大于浓小于淡；`grayscale/speia/invert/opacity`区间0至1。
    * `blur`单位为长度，`hue-rotate`转色轮360回原色；多函数空格连写自左向右叠加。
    * 投影（Drop Shadow）为`drop-shadow(4px 4px 6px rgba(0,0,0,0.4))`，相对`box-shadow`恒方形的差异为跟随镂空轮廓。
    * 悬停范式为常态`filter:grayscale(1) brightness(0.9);transition:filter 0.4s`，悬停切彩亮；过渡须写常态。
    * | 函数 | 取值 | 含义 |
      | :--- | :--- | :--- |
      | `grayscale()` | `0~1` | 1全灰 |
      | `brightness()` | `0~` | 1原样 |
      | `blur()` | 长度 | 半径越大越糊 |
      | `hue-rotate()` | 角度 | 360回原色 |
    * > 来源: P123
* **背景滤镜（Backdrop Filter）**
    * > 定义：背景滤镜为对元素背后已渲染内容的区域加工，自身文字保持清晰的隔玻璃观察机制。
    * 三件套为半透明`background:rgba(255,255,255,0.3)`、`backdrop-filter:blur(10px)`、`-webkit-`前缀。
    * 背景写实色则背后被盖无效果；导航舒适值`blur(12px)`，5px以下无感，20px以上糊化。
    * 常用叠加为`blur(8px) saturate(1.4)`配圆角描边，长期在线导航弹窗宜用，数十卡片全加低端机掉帧。
    * ```css
      .card { background: rgba(255,255,255,0.25); backdrop-filter: blur(8px) saturate(1.4); }
      ```
    * > 来源: P124, P110, P132
* **圈数（Turn）**
    * > 定义：圈数为旋转圈数单位，`1turn=360deg=6.2832rad=400grad`，小数即圈内相位。
    * 适用限旋转角度，多圈直写`3.5turn`免算度数；半圈`0.5turn`即180度。
    * 花盘8瓣以盘心为轴等分步进`0.125turn`，容器`transform-origin:50% 130px`，瓣样式为椭圆渐变半透明。
    * 整盘动画`0turn`至`1turn`配`12s linear infinite`，负值反转；摆位写瓣身，自转写盘身不可混。
    * ```css
      @keyframes spin { 0% { transform: rotate(0turn); } 100% { transform: rotate(1turn); } }
      ```
    * > 来源: P122

---

## 7. 滚动驱动时间线

> **一句话主旨**：将动画进度由时钟切换为滚动行程或元素可见性，实现双向可逆的滚动叙事。

### 7.1 全局行程与独立可见

* **滚动时间线（Scroll Timeline）**
    * > 定义：滚动时间线为以容器绝对滚动行程百分比映射关键帧进度的全局时间线，反向滚动等比倒放。
    * 声明为`animation-timeline:scroll()`，括号禁省；挂载后时长退居合规占位，不控快慢。
    * 进度条案为`@keyframes scrollProgress{0%{width:0%}100%{width:100%}}`配`animation:scrollProgress 2s linear forwards`。
    * 结构为固定`.nav-bar{position:fixed;top:0}`底贴`.scroll-bar{height:4px}`渐变细线，长正文撑出滚动行程。
    * 未挂载则加载2秒自动拉满，挂载后顶部为0、半程50%、底部100%；总行程自适应2000或20000px高度。
    * > 来源: P125
* **视图时间线（View Timeline）**
    * > 定义：视图时间线为以元素自身进出视口可见比例驱动进度的独立时间线，每元素各走其轴。
    * 选型为页面整体进度用`scroll()`，多元素依次入场用`view()`；声明为`animation-timeline:view()`。
    * 基础帧为`from{opacity:0;transform:translateY(60px) scale(0.9)}`至`to{opacity:1;transform:none}`，时长作废由滚动充当时钟。
    * 默认区间铺满进出全程显拖沓，须`animation-range:entry 0% entry 100%`限定进入段播完定格。
    * 区间关键字含`entry/contain/exit/cover`，支持百分比长度微调跟手度；长列表声明一次自动错峰。
    * 降级为默认终态可见加`@supports(animation-timeline:view())`叠加入场，旧浏览器保静态列表。
    * | 类型 | 声明 | 进度轴 |
      | :--- | :--- | :--- |
      | 滚动时间线 | `scroll()` | 容器行程 |
      | 视图时间线 | `view()` | 元素可见比例 |
    * > 来源: P126, P137

---

## 8. 变量计算与精灵图公式

> **一句话主旨**：以变量存可变序号与主题值，以计算函数实时混算单位，实现批量换肤与坐标公式化。

### 8.1 声明引用与混算规则

* **层叠变量（Custom Properties）**
    * > 定义：层叠变量为双横线开头的运行时自定义属性，遵继承级联，可被引用兜底与脚本改写的参数。
    * 声明如`:root{--main-color:#ff6b9d}`，全局放`:root`伪类高于`html`，局部可在任意后代覆盖仅影响子树。
    * 引用为`var(--main-color)`，兜底为`var(--main-color,#333)`；未定义无兜底声明无效，有兜底取兜底，类型错配声明无效。
    * 名称大小写敏感；引用处建议常带兜底防裸奔。
    * 读写为`getComputedStyle(root).getPropertyValue('--main-color')`与`root.style.setProperty('--main-color','#3b82f6')`，后者内联层级必生效，`removeProperty`恢复。
    * 换肤为`body.dark`整组切`--bg/--text/--accent`，按钮类切换全站同步。
    * > 来源: P127
* **计算函数（Calc）**
    * > 定义：计算函数为渲染时实时求值的四则函数，支持异单位混算，窗口一变即重算的布局运算。
    * 范式为侧栏200px配主体`width:calc(100%-200px)`恒吃剩余；三栏为`calc(100%-200px-240px)`。
    * 加减须两侧空格，`calc(100%-200px)`整条失效；乘须一侧纯数字，除数须纯数字，禁百分比乘百分比。
    * 支持`vw/vh/rem/px/%`混搭，如`calc(100vw-40px)`、`calc(1rem+1vw)`；联动变量如`calc(100%-var(--sidebar)-var(--gap)*2)`。
    * 错写静默失效回`auto`，排查先查空格括号；现代浏览器兼容无碍。
    * ```css
      .main { width: calc(100% - var(--sidebar) - var(--gap) * 2); }
      ```
    * > 来源: P128

### 8.2 精灵图序号公式

* **精灵图公式（Sprite Formula）**
    * > 定义：以序号变量承载变化量、`calc()`沉淀乘宽规律，将枚举坐标压缩为公式的坐标管理方法。
    * 单行公式为`.icon{background-position:calc(var(--i)*-40px) 0}`，类只声明序号`--i:0/1/2`，改格宽只改一数。
    * 二维扩展为`--x/--y`双变量分控横纵，如`calc(var(--x)*-40px) calc(var(--y)*-40px)`。
    * 行内可直传`style="--i:5"`，后端循环吐序号零新增类，表现留类、数据进变量。
    * 方法论为遇规律重复先抽变量写公式，栅格偏移轮播位移阶梯延迟同理。
    * ```css
      .icon { width: 40px; height: 40px; background-position: calc(var(--i) * -40px) 0; }
      ```
    * > 来源: P129

---

## 9. 综合导航与滚动叠加

> **一句话主旨**：以变量位移与兄弟选择驱动指示跟随，以视口粘性与视图收缩完成层叠叙事。

### 9.1 跟随指示导航

* **动感菜单（Dynamic Menu）**
    * > 定义：以变量做状态、位移做表现实现文字上浮与指示横滑双链并发的导航模式。
    * 语义为`nav>ul>li>a`加平级`.indicator`，`ul{display:flex}`横排，`a{display:block}`保点击区。
    * 悬停为`li:hover a{color:#ff6b9d;transform:translateY(-4px)}`，背景`rgba(255,255,255,0.08)`淡高光。
    * 指示为`.indicator{width:96px;height:4px;transform:translateX(calc(var(--x)*100%));transition:transform 0.3s}`，序号由`--i`经`--x`传递，初态`var(--x,0)`防乱飘。
    * 过渡须写常态，`~`要求指示与项为兄弟；等宽不对齐先查统一`padding`定宽。
    * > 来源: P130
* **滑动导航栏（Sliding Navbar）**
    * > 定义：以通用兄弟选择器按格步进驱动圆角滑块背后滑动的等宽导航。
    * 结构为5`li`加尾`span.slider`，顺序不可反；`ul{position:relative;display:flex;width:600px;height:60px;border-radius:30px}`。
    * 项`flex:1`居中，文字`z-index:2`浮滑块上，滑块`100px×40px;left:10px;transform:translateY(-50%);z-index:1`渐变圆角。
    * 走位为10、130、250、370、490px，步进120px即格宽；规则为`li:nth-child(n):hover~.slider{left:...}`。
    * `~`命中后方全部兄弟，`+`仅紧邻不可代用；过渡`left 0.35s ease`写本体，0.1过贼0.8过肉。
    * > 来源: P131
* **炫酷导航栏（Cool Navbar）**
    * > 定义：以毛玻璃底座承载图标文字竖排、悬停点亮加伪元圆点指示的无脚本导航。
    * 底座为`ul{width:480px;height:80px;background:rgba(255,255,255,0.08);backdrop-filter:blur(15px);border:1px solid rgba(255,255,255,0.15)}`，须渐变图片垫底方显通透。
    * 项为`a{flex-direction:column;gap:4px;color:rgba(255,255,255,0.6)}`，图标24px文字12px。
    * 点亮为悬停与`.active`共用`translateY(-6px);color:#fff;text-shadow:0 0 12px`，过渡0.3s写默认态。
    * 圆点为`li{position:relative}`上`::after{width:6px;height:6px;border-radius:50%;background:#00e5ff;transform:translateX(-50%) scale(0);opacity:0}`，点亮切`scale(1);opacity:1`；`content`禁漏。
    * > 来源: P132, P133

### 9.2 视口层叠与视图收缩

* **视口单位与首屏（Viewport Units， VW/VH）**
    * > 定义：视口单位为相对窗口等分100份的长度单位，不依赖父尺寸即满屏的首屏定高工具。
    * `100vw`为视口宽，`100vh`为视口高，随窗口实时变；`%`看父无高则0，`vh`直看窗口。
    * 宽度宜`100%`防`100vw`含滚动条撑横向溢出，高度取`100vh`；撑高禁写`html,body`，仅满屏节取用。
    * 首屏为`.hero{height:100vh;flex-direction:column;justify-content:center}`承载标题副题按钮，舞台卡片取`80vh`圆角24px三色交错。
    * > 来源: P134
* **粘性层叠（Sticky Stack）**
    * > 定义：以逐卡粘性吸顶加后者居上实现扑克牌压盖的纯CSS层叠机制，每卡经历跟随吸顶被盖三段。
    * 核心为`.card{position:sticky;top:20px}`或`top:0;height:100vh`，滚至阈值吸住后卡上移盖前卡。
    * 等`top`全盖严看不出层数，错开如0、40、80、120px露帽檐台阶边；级差40px兼顾圆角与视口。
    * 顺序即层叠，后卡天然压前卡，显式`z-index:1/2/3`防乱；`sticky`须写卡身禁写父舞台，父禁`overflow:hidden`否则失效。
    * 卡内左右双栏弹性排布，图`object-fit:cover`，深浅背景交错保盖线干脆。
    * > 来源: P135, P136
* **层叠视图收缩（Stack View Shrink）**
    * > 定义：以视图时间线绑定每卡独立行程实现被盖时缩小压暗退场的层叠增强动效。
    * 同效错峰须`view()`而非共轴`scroll()`，末卡无盖行程不动自然收尾。
    * 写法为`.card{animation:scaleDown linear both;animation-timeline:view()}`，帧`to{transform:scale(0.9);filter:brightness(0.6)}`。
    * 时长占位须`linear`保等比，禁缓动；`both`保行程外首尾不闪回；原点保持中心，亮度0.5至0.7可调。
    * 兼容为渐进增强，旧浏览器回硬盖层叠保功能，新浏览器得收缩味道。
    * ```css
      @keyframes scaleDown { to { transform: scale(0.9); filter: brightness(0.6); } }
      ```
    * > 来源: P137
