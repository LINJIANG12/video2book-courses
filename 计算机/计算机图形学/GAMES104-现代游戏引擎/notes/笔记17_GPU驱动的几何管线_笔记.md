# GPU驱动的几何管线

## 传统渲染管线的瓶颈

* **绘制调用（Draw Call）**

    * > **定义**：CPU 发起一次几何绘制的完整过程——先设置好纹理、alpha blend 等一整套渲染状态（Render State），提交顶点与索引数据，再进入漫长的着色器与光栅化管线。

    * Draw Call 非常昂贵：哪怕只画一个三角形，整条管线也要完整走一遍；渲染状态的设置本身代价也很高。

    * 把同材质的渲染 merge 在一起，正是为了把它们的 Draw Call 与渲染状态一并合并掉。

* **组合爆炸**

    * 现代游戏场景几何复杂、材质丰富：mesh、渲染状态、LOD 设置、材质的一大堆参数与各种纹理、skinning 与动画——每种组合都要单独 Draw Call 一次并准备一次渲染状态，3A 级场景按传统管线绘制基本跑不动。

* **CPU 成为瓶颈**

    * 所有渲染需求由 CPU 发起：视锥剔除（view frustum culling）在 CPU 算完还要提交给 GPU，期间 GPU 大量时间空等数据。

    * 多线程提交（DX12 的核心架构之一）只是 CPU 资源被大量浪费状态下的权宜之计。

---

## GPU 驱动渲染的目标

* **计算着色器（Compute Shader）**

    * > **定义**：可在 GPU 上执行通用计算的着色器，约十年前开始成熟。

    * 让视锥裁剪这类原本属于 CPU 的通用计算完全在 GPU 内部完成，免去 CPU 与 GPU 之间来回搬数据（数据要从 memory load 进 cache 再送运算单元）的开销，CPU 与 GPU 的通讯变得非常简单。

* **间接绘制（Indirect Draw）**

    * > **定义**：把 array of parameters（parameter buffer）整体录进去、一次 Draw 画出许多 mesh 的绘制方式。

    * 让"尽量减少 CPU 到 GPU 的 Draw Call"成为可能。

* **终极图景：一次 Draw Call 画完整个世界**

    * 整个场景数据放进显存（现代显卡显存动辄数 GB，装得下）；可见性判断、LOD 选择、几何裁剪全部在 GPU 内部完成；CPU 只发一条"把场景绘制一遍"的指令并设置相机参数。

    * 从一个个 draw primitive 走到理想中的一次 draw call；CPU 被释放出来做 AI、游戏逻辑、玩法、物理检测、网络通讯。

    * cluster-based rendering、visibility buffer 与 Nanite 的原始思想同出一源：整个世界的几何裁剪与可见性判断，尽量别让 CPU 拖后腿。

---

## Cluster-Based Rendering 的开创

* **历史定位**

    * 2015 年前后《刺客信条：大革命》（Unity 引擎）在 GDC 公开的 GPU 驱动管线，是这条路线最早的先驱实现，也是后续所有 cluster-based geometry rendering 的源头。

* **要解决的问题**

    * 大量精细的巴黎建筑浮雕：场景上亿级面片、上万个物体；传统做法把整个几何加载后逐 instance 实例渲染，大量 instance 不可见、建筑自身大量几何也被遮挡，渲染状态的设置无从优化。

### cluster 与 chunk 的两级颗粒

* **簇（Cluster）**

    * > **定义**：把一个 instance 的 mesh 切成的固定小尺寸片段——大革命取 64 个三角形一片（数字本身不重要，重点是一小片固定大小的 mesh），每片自带包围盒（bounding box），是可见性判断的最小颗粒。

    * 颗粒度从 instance 细化到 cluster 后：可见性、背面剔除、深度遮挡都可以逐片判断；"instance 只要有一个角可见，10 万个三角形就得整个重渲"的浪费被消除。

    * 对细密的高精度几何，这是三角形利用率最高、overdraw（同一像素被很多三角形反复绘制的浪费）最小的做法。

* **块（Chunk）**

    * > **定义**：把约 16~32 个 cluster（一般取 32）预先 group 成的一批，尺寸对齐 GPU 的 wavefront——它只是个加速结构。

    * 动机：GPU 一次性发出一批工作（NVIDIA 叫 warp、32 个并发，AMD 早期 64 个），把所有工作线程全部吃满效率最高；chunk 一批整扔出去。

### 从 CPU 到 GPU 的过滤管线

* **CPU 侧做得很薄**

    * 对 instance 做最简单的视锥剔除粗筛。

    * 按材质建哈希表，把相同材质、相同渲染状态的尽量归拢到一起提交。

    * 把每个 instance 的 transform 与整体选定的 LOD 层级（按离相机距离对整个 instance 计算——古典做法，与 Nanite 有本质不同）打包成 Buffer，一股脑交给 GPU。

* **GPU 侧层层细筛（全部在 Compute Shader 里跑）**

```text
Instance 细筛（更精细的视锥 + 深度剔除）
  -> Chunk 可见性测试
  -> Cluster 可见性测试（各自包围盒 + 深度测试）
  -> Cluster 内背面三角形剔除
  -> 可见三角形压实（Index Buffer Compaction）
  -> 一次（Multi）Draw Indirect 画完整个场景
```

### Index Buffer 压实与绘制顺序

* **压实（Index Buffer Compaction）**

    * 先分配约 8MB 的大 Index Buffer；所有工作线程并行遍历所有 instance、chunk、cluster、三角形，可见三角形的索引以原子写依次排进大 buffer。

    * 依赖 compute shader 的并行能力与写运算的原子性；写得稳、工作量均匀时效率极高。

    * 本质：一个超级 visibility culling——把不可见三角形尽可能全干掉，可见几何 group 成一个大 buffer，一次 draw 画完；性能最优的同时把 overdraw 降到最低。

* **帧间顺序确定性**

    * 并行写导致绘制顺序逐帧乱序（第一帧是 instance1、5、8，第二帧可能倒过来）；几何密度特别高、彼此贴得极近时，z-buffer 精度有限，顺序一变就 z-fighting。

    * 解法：multi draw indirect 可以锁死 instance、cluster、三角形的绘制顺序，避免 z-fighting。

    * > ⚠️ **易错点**：帧与帧之间的渲染顺序最好是 deterministic（确定）的。高密度贴面几何（如在墙上再叠一层贴花几何）相机稍远，z buffer 精度就不够——先画谁后画谁必须保证，否则画出来就是错的。

### 立方体位图背面剔除

* **6 面 × 64 bit 预编码**

    * > **定义**：在 cube 的六个面上分别记录"从这个方向看过去，cluster 的 64 个三角形各自可见与否"——6×64 个 bit 表达整个 cluster 的背面可见性。

    * 相机视角到来先确定可见的三个面；三个面的位图里只要有一位亮，cluster 就不会被背面剔除——速度非常快。

* **去向**

    * Nanite 未采用该技巧，但作为高速背面剔除手段值得记录。

---

## 遮挡剔除：低成本 Z 挡板

* **为什么必须做**

    * 不做遮挡剔除，现代游戏性能差得一塌糊涂——同样场景帧率直接下降一个量级是常态。

    * 心法：所有算法做的都是同一件事——用尽可能低的成本构建当前相机的 depth buffer，再 build 成一层层"幕布"，拿任何物体/cluster 的包围盒在 GPU 里快速测试，把大量看不见的东西踢掉。

### 大革命的启发式方案

* **上一帧深度重投影（Reprojection）**

    * depth buffer 里每个像素有深度与 XY，等价于世界坐标系中一个个"带深度的点"；换掉相机位置后每个像素都能重新投影到新相机位。

    * 核心观察：相机移动光滑、场景大部分物体静止时，上一帧的 z-buffer 可以重用。

* **启发式 Z 拼接**

    * 用启发式规则挑最可能遮挡别人的物体（又大又近优先）约 300 个，先画出它们的 z-buffer，down-sample 到 512×256；因为只画了少数物体，这张 Z 有很多洞，再用上一帧 z-buffer 重投影把洞填一填。

* **层级深度缓冲（Hierarchical Z-Buffer，HZB）**

    * > **定义**：对 z-buffer 一层层往上 mipmap 密铺，每一层取最近（最小）的那个 Z。

    * cluster、chunk、instance 都有包围盒，可在 GPU 上直接剔除：instance 不可见则其 chunk 不可见；chunk 可见再问 cluster 可不可见。

    * > **提示**：HZB 是保守测试——通过说明肯定没被遮挡；没通过只说明可能被遮挡。

* **边界**

    * 上一帧 Z 里高速移动的物体会造成误判；理论上存在 artifact，实际结果基本可接受。

### Two-Phase 遮挡剔除

```text
全部物体（如 1 万个）
  |  用上一帧 Z 测试（保守过滤，真实场景通过率极低，约 100 个通过）
  v
通过者按当前相机快速画一遍 -> 本帧新 Z（有洞，深度表达可能不准）
  |  用本帧新 Z 复查此前被过滤掉的全部物体
  v
被误杀的物体复活，重新进入绘制
```

* **机制**

    * 第一步用上一帧 Z 提前过滤：1 万个物体可能只有约 100 个通过——站在巴黎街道里满眼几十栋建筑，全城上万栋，真实场景通过率就是极低。

    * 幸存者按当前相机快速画出本帧新 Z；第二步拿这张当前帧、当前相机的真实 Z，把被过滤的物体全部复测一遍，捞回被误杀者。

    * 预过滤是保守的：不怕近处马车高速通过——大不了这一帧少画几个物体，复测一定能把它们画出来，结果准确。

* **验证**

    * 25 万颗随时在动的小行星/陨石场景下效果非常好；真实游戏里动的物体到不了 25 万，但场景物体总量可能真到这个数量。

* **两种方案对比**

| | 大革命启发式方案 | Two-Phase 方案 |
| :--- | :--- | :--- |
| Z 从哪来 | 启发式选约 300 个又大又近的遮挡物先画，降到 512×256，再用上一帧 Z 重投影填洞 | 上一帧 Z 预过滤，幸存者用当前相机现画新 Z |
| 正确性 | 高速移动物体会造成误判，"基本可接受" | 预过滤保守、二次复查兜底，结果准确 |
| 机制 | 一步到位，拼出的 Z 直接建 HZB 测试 | 先粗过滤，再对被过滤者全体复测 |

---

## 阴影贴图的几何裁剪

* **为什么贵**

    * cascade shadow map 的生成可占整个绘制时间的 1/10 到 1/5 还多。

    * 成本只由几何复杂度决定：材质上做任何简化都不影响它。

    * shadow caster 的几何精度要与相机空间看到的精度一致——用低精度几何向高精度几何投影，depth 信息无法匹配，会产生很多很脏的 artifact。

* **上一帧阴影深度重投影**

    * 把上一帧 shadow map 的 depth buffer 重投影成本帧的，精度只取 64×64 像素，用它快速剔除大量 caster——cascade 最后一层可能覆盖方圆几平方公里的物体，能挡多少挡多少。

* **相机深度延伸出的四棱柱**

    * 对 camera space 的 depth buffer 每 16×16 texel 取一个最近深度，沿光方向延伸出一个体积（严格讲不是 cube，是四棱柱）；逐物体检测与该体积有无交点。

    * 依据假设：建筑物平地而起、人走在街道上——近处遮挡物形成的 shadow volume 与相机视锥不相交，其背后的 caster 投不进画面，渲染 shadow map 时可整批跳过；特别高的物体仍会有交集，但这类物体在场景里是极少数。

---

## Visibility Buffer

* **定位**

    * 一种越来越受关注的新渲染管线；查到的最早论文在 2013 年，自称 cache friendly 的 deferred shading 优化方法。

    * Nanite 最终就用 visibility buffer 机制对渲染出的几何做 shading——理解 Nanite 渲染的关键积木。

### Deferred Shading 的两个软肋

* **肥 G-Buffer（fat G-buffer）**

    * deferred 的收益：G-buffer 只存可见像素，避免大量无谓 pixel shading；动态光源可无限叠加；tiled rendering 还能按小 tile 统计光源数——上千个光源动来动去也没问题。

    * 代价：每像素要存 normal、albedo、roughness 等几十甚至接近 100 字节的中间数据；cache miss 是显卡性能命门，buffer 越肥、光写一下 fetch 一下就很废（4K 分辨率下尤甚）。

    * 手机端 memory 带宽非常紧张，fat G-buffer 根本用不起，只能用 TBDR 加各种 hack 模拟类似 deferred 的效果。

* **植被类场景的过绘（Overdraw）**

    * 叶子层层叠叠，几何提交到显卡的顺序不受控制、无法保证从靠近相机的开始绘制，屏幕上一个像素可能被绘制十几次。

    * 每次过绘都要全套纹理采样（三线性插值：mipmap 上下两层之间插值，每层 mip 内部再做双线性插值）并把近 100 字节写进 G-buffer；ALU 对现代 GPU 还好，大量 texture sampling 加 G-buffer 反复读写让性能急剧变慢。

### 两步渲染法

* **可见性缓冲（Visibility Buffer，VB）**

    * > **定义**：第一遍几何 pass 写出的屏幕空间缓冲，每像素只存一个 32-bit UINT——primitive id、instance id，外加 alpha mask 之类的标志位（产品级实现用 material id 区分材质），不含任何材质属性。

    * 第一步只打 ID：带 alpha mask 的材质走单独的设置通道；哪怕场景只是一个球，打出来也是一堆色彩斑斓、毫无可读性的索引。

* **手动重建着色**

    * 第二步由 instance id / triangle id 取回三角形的三个顶点；相机参数已知，把顶点投影到屏幕空间，由像素的深度与 XY 反算重心坐标（barycentric coordinates）。

    * 对三个顶点的 UV 与纹理做重心坐标插值，得到该像素的 albedo、roughness、specular，知道光的位置后即可 shading——原本由 vertex shader 与光栅器自动完成的插值，改为着色阶段手动完成。

* **为什么反而快**

    * 取顶点数据 cache 命中率高：屏幕上看到的一小块几何往往来自同一个三角形，反复取的就是那三个顶点的数据，cache miss 极低——而对性能影响最大的正是 cache miss。

    * 过绘的代价只剩"打 ID"（计算量极低）；shading pass 永远按"屏幕上有多少像素就采样多少次"进行——几乎没有 overdraw 的 cost。

* **三种着色路线对比**

| | Forward Rendering | Deferred Shading | Visibility Buffer |
| :--- | :--- | :--- | :--- |
| 中间缓冲 | 无 G-Buffer | Fat G-Buffer：每像素几十到近 100 字节 | 每像素一个 32-bit ID |
| 遇到过绘 | 被遮挡像素也做完整采样加光照 | 每次过绘都全套采样并写一遍 G-Buffer | 过绘只付"打 ID"的极低成本 |
| 着色次数 | 按绘制次数算 | 按绘制次数算 | 按屏幕像素数算，几乎零过绘 |
| 典型痛点 | 动态光源一多就得反复绘制叠加 | 植被等层叠场景过绘严重、带宽压力大 | 要手算 mip 梯度等细节，处理不好出 artifact |

### 与 Deferred 合流的双轨管线

* **汇合点在 G-Buffer**

    * visibility buffer pass 照常输出 v-buffer；material pass（G-buffer pass）按 v-buffer 里的 instance id / triangle id 取回数据，把 albedo、specular、roughness 照样打进 G-buffer。

    * 走传统管线的几何本来就在写 G-buffer；两路共享同一张 z-buffer，数据不会冲突；lighting pass 对数据来源完全无感，直接上光。

* **典型分工**

    * 密集植被（灌木丛、树林）：像素级剔除很难看准——树叶有很多洞，一眼望过去几十棵树不知道哪棵被挡住——用 visibility buffer 节约 overdraw 成本。

    * 主角与近处物件：没必要用复杂方法，走传统 deferred shading，最后在 G-buffer 汇合做 final lighting。

### 手算 Mip 梯度与适用边界

* **Mip 梯度细节**

    * 传统管线里光栅器自动生成 DDX/DDY，纹理采样时自动选正确的 mip 层；visibility buffer 里只能拿小三角形的三个顶点 UV 投影到屏幕空间，反算该像素 UV 在屏幕上的梯度来选 mip——注意是在屏幕空间，不是几何空间。

    * 细节处理不好会出 artifact（示例图里凳子拐角的小白边，就是两边密度不匹配所致）。

* **剪刀差**

    * 对 800 万三角形的场景，屏幕精度越大，visibility buffer 相对 deferred shading 的性能领先越大。

    * 原因：显卡算力是陡峭上升曲线、memory 速度是平缓曲线，二者不匹配；算力越强，visibility buffer 优势越大。

---

## Nanite 的路线选择

### 从虚拟纹理到虚拟几何

* **虚拟纹理（Virtual Texture，VT）**

    * > **定义**：John Carmack 在 Quake 3 前后提出的纹理虚拟化方案——给定预算，近处纹理精度高、远处精度低，形成 texture clip map，把当前视点需要的所有材质纹理缓存在有限预算里。

    * 额外收益：所有材质拍进一张巨大虚拟纹理后，"每换一次材质切一次渲染状态"的麻烦一次性解决；deferred rendering 用的也是这个思想。

* **Virtual Geometry 的梦想**

    * Nanite 的官方标签是 virtual geometry：纹理可以 virtualize，几何也要 virtualize；目标是在游戏引擎管线中实现无限几何细节、达到电影级（cinematic）精度。

    * 核心思想：屏幕像素数是硬上限，一个像素原则上一个三角形、最多两个——几何再海量，任意视角下真正要画的三角形数量可算、有界；用屏幕精度决定几何精度，正如虚拟纹理用视点决定纹理精度。

* **几何难过滤的本质障碍**

    * 几何是不规则数据（irregular data）：index buffer 的访问位置会大跨度跳变（画一个 mesh 可能从第 8 个顶点跳到很远），只能整块搂进来，没法一段一段 paging；LOD0 与 LOD1 的 mesh 可能完全无关，filter 不连续。

    * 可过滤的几何表达存在但各有短板：SDF 可以过滤、可做各种 mipmap，但太粗无法直接绘制；voxel、点云也可过滤——这正是选定三角形前逐一排查的方向。

### 备选路线的逐一排查

| 备选路线 | 好的一面 | 过不去的坎 |
| :--- | :--- | :--- |
| Voxel | 表达 uniform、过滤性极好、密铺方便 | 表达高频边界细节时数据量难压；美术资产管线（Max/Maya/ZBrush）全基于网格，逼所有资产转 voxel 的成本致命 |
| Subdivision（细分曲面） | GPU 上实时加密几何（hull/domain shader 到 mesh shader），电影工业大量使用 | 只能向上 refine：5000 面能采到 100 万面，却无法向下减到几百面；Nanite 要求每个对象能减到一百多个面甚至 impostor；细分要用四边形控制面，而实时渲染是三角形 |
| Displacement Map | 有机体表面效果好、可过滤 | 硬表面很难设置；从精细几何反推 displacement map 本身需要运算 |
| Point Cloud | 过滤方便 | 绘制靠在屏幕上 splat 小圆盘，overdraw 大，高精度下效果不好 |
| 三角形 | 整个 content pipeline 与硬件支持最成熟 | ——最终选择 |

* > **提示**：displacement 与"硬件加密几何"之争至今仍是 debate 最多的地方——NVIDIA 40 系推出的 Micromesh 走 GPU 自动加密几何、直接兼容 ray tracing 的路线；Nanite 这套软件方案与硬件路线两派都在快速往前走，胜负未分。

* 选择三角形的理由：它是最成熟、content pipeline（3ds Max、Maya、ZBrush）全面支持、硬件支持最完善的几何表达——整个无限几何方案都以三角形为地基。

---

## Cluster LOD 体系

### 逐层合并的朴素方案

* **Cluster 粒度与逐视角 LOD**

    * 把细密几何切成约 128 个三角形一片的 cluster；每个 cluster 可独立切到不同 LOD 层级——view dependent LOD transition。

    * 与大革命的本质区别：后者的 LOD 层级对整个 instance 锁死，同一 instance 内所有 cluster 只能同层；Nanite 内每个 cluster 层级可不同，把屏幕空间三角形数量的利用率最大化。

    * 效果案例：一条 200 多万三角形的龙，只用约 1/30 的三角形就做到几乎每个屏幕像素一个三角形。

* **朴素层级**

    * 每两个 cluster 合并做一次简化，三角形数量减半，形成上一层 cluster；4 个变 2 个、2 个变 1 个，一路合并到只剩一个 cluster 为止（即根）。

    * 绘制判定：按 cluster 包围体离相机的距离算出"误差不高于一个像素"的容差；简化误差小于一个 sub-pixel 即可绘制，大于则继续向更细层走。

    * 白送的能力——几何流送（streaming）：先加载层级顶端粗糙部分，相机推进发现不够细时逐层加载更细的 LOD，按需下载。

```text
LOD0: c0 c1 c2 c3  ->  LOD1: A(c0+c1) B(c2+c3)  ->  LOD2: R(A+B)
每向上合并简化一次：三角形数减半，几何 error 增加一档
```

### 裂缝与锁边困境

* **裂缝（Crack）**

    * 相邻 cluster 本来 watertight（水密）拼合；一片用 LOD0、相邻一片切到 LOD1 后，简化几何的边界与未简化的对不齐，中间出现裂缝。

* **锁边的两难**

    * 直觉解法：简化时锁住 cluster 边界上的顶点不动，保证与任何 LOD 的邻居都水密拼合。

    * 但锁的永远是 LOD0 那条边，锁边处三角形密度永远降不下来——两个后果：一是高层级（如 LOD10）一个 128 面的 cluster 要表达原始几何很大一片区域，光锁边 128 个面远不够用，实际上没法搞；二是干脆不锁，则锁边区域密度与整体不一致（non-uniform），产生采样 artifact。

    * 人眼对高频信号、特别是频率发生变化（frequency change）的位置侦测能力极强——数值上很小的 artifact 也逃不过眼睛。

### Cluster Group：锁边代价的摊薄

* **簇组（Cluster Group）**

    * > **定义**：把十来个 cluster（8~32 量级，具体数目可调）捆成的简化单元；锁边只锁整捆的外边界，捆内 cluster 之间的边界全部打碎放开，让简化跨 cluster 自由进行。

    * 锁边代价被 group 规模摊薄：一个两万多三角形的 group，锁边可能只占一千多个三角形；内部边界解放后一口气从两万多减到一万，简化变得更有效。

* **核心循环**

    * LOD0 全部 cluster 先聚成 group（约 10:1，如 2000 个 cluster → 200 个 group）→ group 内简化（外边界保持水密）→ 重新聚类出 1000 个新 cluster → 再按比例聚成 100 个 group，逐层向上。

    * 最核心的设定：相邻两层 group 的边界不保证一致——LOD 来回切换时不存在一条持续出现的 boundary；否则那条边界会一直出问题。

* **父子关系不是树**

    * 简化后重新聚类，新 cluster 与旧 cluster 不再一一对应：一个简化 cluster 可能同时关联下层 4 个 LOD0 cluster（三角形被复用）——一对多、多对多。

```text
LOD1:       P           Q
             \         /
              \       /
LOD0:      c0 -- c1 -- c2

P 由 c0、c1 简化而来；Q 由 c1、c2 简化而来
c1 同时关联 P 与 Q（两个 parent）
层与层是多对多连接——这构成有向无环图（Directed Acyclic Graph，DAG），不是树
```

    * 虽然一个底层 cluster 可能有 multiple parents，但它只和"由自己简化带上去了"的那几个上层 cluster 建立关系——影响是 localized 的。

* **每层换边界的抖动设计**

    * 锁边固定会让高频密度区被固定下来、被人眼抓住；Nanite 让被锁的边在每一层 LOD 都变换位置——类比 screen space AO 的采样 jitter：不做抖动，noise 就有 pattern，而人眼最会注意 pattern。

    * 相机移动中 cluster 的 LOD 不断 popping 切换，但高频锁边区每层都不一样，人眼抓不到一条固定清晰的高频缝。

```text
LOD0 锁边: |----A----|----B----|
LOD1 锁边: |------C------|-D-|
LOD2 锁边: |---------E------|--F|
每层的 group 边界都换位置，没有任何一条缝是"常驻"的
```

    * > ⚠️ **易错点**：层层套接的彩色边界示意图画的是 cluster group 的边界，不是 cluster 的边界——把这两个概念搞混，后面整张 DAG 图都读不懂。

### 简化误差：整个体系的地基

* **误差计算必须准确**

    * 每次简化的 error 都要做到 sub-pixel 级——error 不准，相机推来推去就会出现可察觉的 popping。

* **误差必须单调递增**

    * error 向上层传递时单向累加、单调上涨——这是整张 DAG 能并行选 LOD 的地基（数学上保证任意 threshold 下 cut 线唯一）。

* **落地工具链**

    * 生成 cluster → cluster 间链接关系建成 graph → 用 METIS 中间件把 graph 聚类成 group → group 内用二次误差度量（Quadric Error Metric，QEM）这套经典方法做简化 → 简化完再聚类形成新 group。

---

## 并行 LOD Selection

* **要解决的问题**

    * DAG 建好后给定相机位，要找出那条 cut line（画到哪一层为止）；上万个 cluster 连同层层 LOD 结构整棵 traverse 太慢、算法也难写。

    * 要求：决策并行进行且结果 deterministic——若 error 没有单调性约束，不同遍历方向的 cut 可能不稳定，引发 z-fighting、popping（并行提交的先后差之毫厘，数据就 fight，结果就不一样）。

    * 粒度：把同一个 cluster group 做成一个大的虚拟节点——整个节点有一个 error、一个包围盒；选了 LOD0 则组内 cluster 全按 LOD0 画，未选则整层不画；但判定精确到组内每个 cluster。

* **判定方程**

    * 父节点的 error 要大于给定阈值 $T$——说明父层已经不够精细；同时自身 error 不大于 $T$——说明本层足够精细；两个条件同时满足才绘制。

$$\text{render}(n) \iff \text{parentError}(n) > T \ \land\ \text{ownError}(n) \le T$$

    * 只要 error 严格单调上涨，任何 threshold 下这条 cut 线唯一。

* **拍平**

    * 每个节点存两组数：parentError（产生它的那次简化的最大 error）与 ownError；LOD0 的 cluster 是绝对正确的几何，ownError 强制写成 -1——既标记"绝对正确"，又因 -1 比任何阈值小而必过条件二。

    * 同一次向上合并简化出的兄弟 cluster 即使分属不同的上一层 group，error 也一模一样（都取该次简化的最大 error）。

    * 于是树的 traverse 变成数组 traverse：所有节点拍平成列表逐个套方程，任意顺序、任意并行度检测（一次可并行处理 32 个）——树的深度不再影响效率。

* **独立检测，没有联动**

    * > ⚠️ **易错点**：不存在"父节点没通过 → 激活孩子"的联动。所有节点拍平后各自独立检测；只要 error 严格单调向上传递、且下层 group 的 parentError 与上层对应 cluster 的 ownError 严格一致，所有节点的独立检测结果就严格一致——cut 唯一，几何不会被渲染两次。

    * 代码里"cluster error 小于 threshold"的字样容易误导：group 存的是 parentError，比较在 group 内逐 cluster 进行。

* **阈值算例（$T = 1.0$）**

| 节点 | 条件一：parentError > 1.0 | 条件二：ownError ≤ 1.0 | 判定 |
| :--- | :--- | :--- | :--- |
| LOD0 group 的 4 个 cluster | 1.1，通过 | -1，通过 | 绘制 |
| LOD1 红 group 的 cluster | 1.2，通过 | 1.1，不通过 | 不绘制 |
| LOD1 绿 group 的 cluster | 1.4，通过 | 1.1，不通过 | 不绘制 |

    * LOD0 的 4 个小 cluster 被画出，靠的不是任何联动，而是 error 严格向上传递：它们所属 group 的 parentError（1.1）恰好等于上层对应 cluster 的 ownError，两个条件同时满足。

---

## 超级 BVH

### 引入 BVH 的动机

* **数量问题**

    * 700 万三角形的龙生成 11 万个 cluster、几万个 cluster group（约 10:1：LOD0 1.1 万多个、LOD1 5000 多个、LOD2 2000 多个，各层合计约 2 万多个）；哪怕只有一个相机，每次也要把全部 group 拍平遍历一遍——即使用并行方法，效率也很低。

* **空间组织适配**

    * cluster group 是空间上一小片、大小相近的几何，各自有包围盒——最优组织方式就是层次包围体（Bounding Volume Hierarchy，BVH）。

### 结构与共同根

* **超级 BVH 结构**

    * 每一层 LOD 的所有 cluster group 各自按空间组织成一棵 BVH 子树；LOD0 到 LODn 的所有子树连到一个共同根节点。

    * 内部节点记录 children 包围盒的并集（全部 maximum 到一起），error 取所有 children 的最大值。

    * 尽量 build 成度为 4 的四叉树——树尽量平衡，traverse 效率才高。

    * > ⚠️ **易错点**：BVH 叶节点上挂的是一个个 cluster group，不是一个个 cluster。

* **共同根的意义**

    * 游戏场景里 5~20 米的距离很常见；一次根节点测试就能把 cluster group 数量最多的那一整层剪掉——物体够远时（bound 上离相机最近处都隔着 10 米，而该层最大 error 在这个距离下判定无需细化），整棵子树完全不用 traverse。

    * 一路往下，直到某一层的 error 判定说"到这里就够了"，才真正进树取要渲染的 cluster。

* **效果**

    * 600 万三角形合成约 11 万个 cluster：构建 BVH 后只检查 107 个 BVH 节点、4000 多个 cluster（比 11 万少二三十倍），最终真正可见约 2000 多个——这一步让算法真正可以实战。

### 遍历的任务队列

* **类 Job System 结构**

    * > **定义**：固定一组 worker thread（compute shader 一次发出 32 个并全部固定），配一个 multi-producer multi-consumer 的公共 task buffer——任何线程产生子节点任务就往里 append，谁空闲谁取，两个指针各管一头。

    * 并发 append 与取任务靠显卡支持的原子化锁定保证不出乱子。

* **定位**

    * 收益约 10%~30% 的加速，属于实现层面的小技巧；真正核心仍是 BVH 本身。

---

## 软件光栅化

### 硬件光栅化的两个假设

* 为处理纹理采样的 ddx/ddy，硬件每次光栅化至少以 2×2 像素为一组——四个像素一组才能算出导数。

* 古典算法为"大三角形"设计：scanline 对每一行像素算左右两端、填满中间，配合把屏幕按 4×4 分 tile、先整块剔除不含三角形的 tile——当年三角形数量远低于像素数量，这一下剪掉大量无谓计算。

* Nanite 里很多三角形接近一个像素大小：为一个像素的三角形生成 16 倍的像素去做光栅化，大量计算被浪费。

### 小三角形的软件光栅化

* **做法**

    * 用 compute shader 把小三角形的光栅化接管过来，这套 software rasterization 自身比硬件的还快 3 倍。

    * 三角形小于一个像素时：Nanite 知道每个三角形的边长与面积，投影只占一个像素——不做扫描，直接打一个像素上去。

    * ddx/ddy 不需要 2×2 quad 来凑：顶点上存着 UV，直接由顶点数据算出。

* **管线切换规则**

    * 以 cluster 为单位：cluster 内所有三角形投影后的边长都小于 16 个 screen pixel → 整个 cluster 切到软件光栅化管线；只要有一条边超过 16 pixel（离得近）→ 整个 cluster 回硬件管线，不做逐三角形混合。

    * 可视化里红色区域（硬件管线）零星分布，绝大部分区域走软件光栅化。

    * > **提示**：16 这个阈值看的是"该 cluster 里最大的那条投影边长"。

    * > ⚠️ **易错点**：光栅化管线的切换以 cluster 为单位，不是以 cluster group 为单位。

* **前景**

    * 硬件厂商很可能把这套机制硬件化——tiny triangle 直接走高效的硬件 single pixel 管线；NVIDIA 的 Micromesh 可能正是软件光栅器的硬件化方向。

### 64 位原子操作的深度测试模拟

* **问题**

    * 软件光栅化管线没有硬件的 early-z / z-testing 机制，而 Nanite 级几何的遮挡无处不在。

* **编码**

```text
一个 64 位原子字（InterlockedMax 的操作对象）：

[ 高 32 位：depth（比较依据） | 低 32 位：cluster id / triangle id ]

depth 更大（该编码里更大 = 离相机更近）-> 整个 64 位字被覆盖
depth 更小 -> 一个比特都写不进去
```

    * 用 shader 扩展直接写 64 位原子操作 `InterlockedMax`：深度占高位、cluster/triangle 信息占低位；比较只由高位深度决定，深度比较与"赢家附带的信息"在一次原子操作里全部完成。

    * buffer 里留下的永远是当前最近的片元，被遮挡几何在写入阶段就被过滤——本质上是在软件光栅化里手动模拟了一遍 z-testing，未来大概率会被硬件原生支持。

### 先可见性后着色

* **输出与消费**

    * 几何阶段输出 depth、cluster id、triangle id（伪彩色可视化中 cluster 极密、三角形几乎达到像素级）——即 visibility buffer 路线：先不生成 G-buffer，先把"每个像素属于哪个 cluster、哪个三角形"打对，着色放后面。

    * lighting pass 与 visibility buffer 渲染完全一致：由 id 找到三个顶点索引、取回顶点数据，按顶点 UV 插值出 albedo、specular 等，再上光。

* **与 Deferred 合流**

    * Nanite 这条路增加一个 material pass，把数据照样写进 G-buffer，与走传统 deferred 的几何共享，lighting pass 全管线统一——否则着色要跑两遍。

    * 实例：曼哈顿 demo 里绝大部分几何走的仍是传统 deferred pipeline，两种渲染毫无压力地混在一起。

    * 边界：Nanite 目前只能解决静态几何；带蒙皮的角色、奔跑的汽车等动态几何，现有技术解决不了。

---

## Imposter 远景替身

* **远距离替代**

    * 相机 20 米外的小雕像只占几个像素，即使简化到最小的 128 个三角形精度也过剩（简化到 128 面的兔子早已不像兔子，再往下简不行）。

* **做法**

    * 对 instance 取 12×12 共 144 个 view 采样，每个 view 渲染一张 12×12 像素的小图，存 albedo、normal 甚至深度。

    * 距离足够远时整个 Nanite 管线不启动，直接把 imposter 往上一贴：deferred 是对的，G-buffer 数据也是对的，后续着色照常。

    * 结论：经典 LOD 方法在这么高级的现代管线里依然是实战的组成部分。

---

## 三种三角形的瓶颈

| 三角形尺寸 | 瓶颈在哪里 | 为什么 |
| :--- | :--- | :--- |
| 很小 | 几何侧：顶点变换与三角形 setup | 再小的三角形也要取三个顶点、做插值、每个 pixel 都得折腾一遍；三角形过小时一个 pixel 里可能叠了好几个 |
| 中不溜 | 像素侧：coverage | 光栅化覆盖这一步本身比较费 |
| 很大 | 深度写入：原子操作争用 | 手搓的 64 位 depth buffer 靠原子操作维持 Max 语义——读写时别人不能动，只能排队 |

* 即使用了整套体系，真实 Nanite 场景里 overdraw 依然存在且昂贵（高亮区域即过绘重灾区）；小三角形情形下，visibility buffer 前面那一把几何处理（vertex transformation + triangle setup）是很费的一步。

---

## 材质绘制

* **难点**

    * 一个场景由五花八门的材质合成（把材质 ID 画成伪彩色一目了然），每种材质的 texture 与数据都不一样。

* **重要性**

    * 材质不解决，几何再漂亮也到不了商业级引擎的画面——artist 绝对不接受"限制你只用一种材质"。

### 早期方案：材质 ID 当深度

* **机制**

    * 把每个材质 ID 变成一个专属 depth 值；屏幕上有多少种材质就全屏绘制多少遍，绘制时做深度等于（EQUAL）测试——depth 里记录的材质 ID 等于当前材质的像素才被画出来。

* **代价与收益**

    * 50 种材质要全屏扫 50 遍，上百种材质就是上百个 fullscreen pass，overhead 并不小（与粒子系统里叠几十上百个全屏透明 quad 拖垮性能同理）。

    * 不费的一面：每个像素真正复杂的 pixel shading（用 visibility buffer 渲染像素是很麻烦的事）只对屏幕像素数量做一遍。

### 新方案：Tile 化材质表

* **流程**

    * 把屏幕分成 64×64 的 tile；用 compute shader 扫全部 tile，得到当前屏幕真正用到的材质表——理论上千上万种 material，当前 screen 可能只用 100 种，每种是一个 material slot。

    * 对每个 slot 标记每个 tile 是否用到该材质（tile 里只要有一个 pixel 用到就标 yes）；用 32 个比特编码 32 个 tile 的存在性——每种可见材质得到一张 tile 位图数组。

    * 对每种材质发起一次 indirect draw：在 compute shader 里把不含该材质的 tile 直接跳掉，一批一次处理 32 个 tile；一个 64×64 tile 四千来个像素一次带过，远快于逐像素处理。

* **推广与展望**

    * 同一思想适用于光源：整个屏幕 100 个光源，单个 tile 里可能只见两三个——tile-based 分而治之是 compute shader 时代降低复杂绘制复杂度的通用手段。

    * 若所有纹理与材质进一步走 virtual texture（全部拍进一张虚拟纹理），材质数量还能再降——Nanite 官方列出的 future work 方向。

---

## Virtual Shadow Map

### 阴影的难点

* 全局光照（GI）是低频信号，几何粗一点无所谓——这正是 Nanite 能与 Lumen 做朋友的原因；直射光的 shadow 是高频信号，cast shadow 的几何精度必须与渲染精度一致，否则出现各种很脏的 artifact。

* ray tracing 帮不上忙：Nanite 的几何表达高度定制化，现代 RT 硬件无法在自己的 BVH 架构里表达它；NVIDIA 强调用 Micromesh 结构加密几何也能构建 BVH 做 ray tracing，与 Nanite 的说法针锋相对——至少目前实时 ray tracing 无法承担 Nanite 几何的阴影计算。

### 阴影的采样本质

* **本质**

    * shadow map 是"根据相机视空间的精度去采样光空间"；相机空间对几何的采样与几何在光空间对可见性的采样频率不一致，才产生各种 artifact——加 bias 就是为采样不准留容错。

    * 近处希望 shadow map 的一个 texel 与屏幕一个 pixel 接近 1:1；远处一个 pixel 对应光空间一大片，可用更低精度——按 view dependency 生成不同精度的多层 shadow map 就是级联阴影贴图（Cascaded Shadow Map，CSM），现代游戏最经典的方法，数学本质是 view-dependent sampling。

* **CSM 与 SDSM**

    * CSM 围绕相机无脑扩圈：最远一层覆盖方圆 2 km，真正被看到的只是视锥切出的窄带，大量分辨率浪费。

    * 采样分布阴影贴图（Sample Distribution Shadow Map，SDSM）按视锥在 shadow map 空间的实际分布分配精度，把更多精度留给相机较近的地方——分布更合理，但仍是整块分配。

    * 更早的 perspective shadow map 试图在两个空间之间找平衡，是早已无人使用的古老算法。

### 按视空间切页

* **虚拟阴影贴图（Virtual Shadow Map，VSM）**

    * > **定义**：把相机看到的世界按视空间分辨率组织（近处采样密度高、远处低，形如 clipmap），每个区域在 light space 分到一小块 shadow map 页——页的大小不以世界空间尺寸、而以视空间尺寸决定：远处一大片区域只需一小块页，近处小区域反而拿得到足够精细的页。

    * 页像 volumetric clipmap 一样一格一格、以 world space 角度划分；Nanite 的 shadow 用的就是这套方法。

* **分配粒度**

    * 每个光源给死一个 16k×16k 的大 virtual shadow map，按视空间需求在其中找 tile 判断光源可见性；点光源对世界投影六个面，会生成六个 16k×16k 的 virtual shadow map（很废，但确实解决问题）。

    * 不同光源类型（spot light、点光源、方向光）对 shadow map 空间 tile 的划分方式各不相同。

* **缓存与失效**

    * 光与相机都不动时，页完全不用更新——游戏主光源（如太阳）基本不动，页的缓存命中率非常高。

    * allocation 每次只分配需要的部分；相机 smooth 移动时只更新部分 tile——从 cascade 时代"画一大片几何"变成"只画与光锥相关的一小片"，大量物体被 cut 掉，更新效率极高。

    * 失效场景：相机移动（smooth 时仅部分页更新）；光一动所有 shadow map 作废——所以 VSM 场景尽量选主光不动的设定（demo 里主光常固定直射）；几何移动导致阴影变化，可把变化做得更 local；另有其他失效原因。

    * 实测：主光不变推动相机，山体相关的 tile 基本不更新，只有少量 tile 需要更新。

* **质量**

    * 阴影质量比传统 cascade 明显更高；CSM 层级过渡处 shadow 会 popping，VSM 的 shadow 基本稳定；与 Nanite 管线柔和自然地结合。

    * 从数学原理上讲是对采样问题的本质解，大概率是取代 cascade shadow map 的下一代 shadow 技术。

* **三代方法对比**

| 方法 | 核心思路 | 局限或进步 |
| :--- | :--- | :--- |
| CSM | 按视距离分层，近处的层精度高 | 围绕相机无脑扩圈，大量分辨率浪费；层间过渡有 popping |
| SDSM | 按视锥在 shadow map 空间的实际分布分配精度 | 分布更合理，但仍是整块分配 |
| Virtual Shadow Map | 按视空间大小切页、按需分配，光不动可整页缓存 | 采样问题的本质解，质量最高，与 Nanite 天然契合 |

---

## 流送与压缩

* **几何流送（Streaming）**

    * 有了 BVH 树形结构，可按 view dependency 自然流送：视空间需要哪块高精度就只加载那块的 page，page 组织成一块块单位加载——几何像 virtual texture 一样"要用即下载，不用即不下载"；做基于 Nanite 的开放世界的必备技术。

* **内存压缩：量化（Quantization）**

    * 浮点转定点：顶点位置相对 cluster 的 bbox 定点化，normal 不需要太高精度可定点化，UV 也可定点化；节约存储，且访问与反向解包效率都很高。

* **磁盘压缩：硬件 LZ 解压**

    * 磁盘数据用 LZ 系压缩（效率最高、众多硬件支持）；DirectStorage（DirectX 提供，AMD/NVIDIA 的 SDK 亦有对应支持）让数据从 SSD 不经过 CPU 与主内存直接读进显存，读取途中自动完成解压。

    * 小技巧：在数据里加 padding 提高 LZ 字典命中率，换取更高压缩效率。

---

## Nanite 的边界与前景

* **技术边界**

    * 只支持静态几何：带蒙皮的角色（skinned geometry）与动态几何无法表达；算法本身做了大量工程妥协、非常复杂。

    * 但它第一次让游戏看到了实时的 cinematic 级几何细节。

* **路线之争**

    * Nanite 软件方案与硬件加密几何（Micromesh 类）两条腿赛跑，胜负未分；引擎行业几十年的规律是：一个新思想要经过好几代开发者共同迭代才能形成共识性 pipeline——deferred shading 也是多代迭代才有今天的成熟度，现在断言大局已定为时尚早。

* **应用空间**

    * 游戏之外同样成立：虚拟拍摄（LED 屏上绘制上亿像素的真实山水，演员在屏前实景表演）、智慧城市、建筑光照仿真中的自然场景表达。

* **一句话定位**

    * 下一代游戏必须有一套能 handle 复杂世界的全新几何管线；Nanite 是了不起的工程思想，把这一代硬件的性能压榨到了极致。

---

## 渲染之外的工程问答

* **ECS 的 Entity 删除**

    * Entity 删除并不真的释放内存（每次堆上 allocate/deallocate 都很慢）：只把 index 置空并挂入空闲链表，下次分配直接复用。

    * 用 salt 区分新旧 handle：salt 累加，index 相同而 salt 对不上即 handle 已失效；访问频率极高的系统，第一条纪律是尽可能减少 memory allocation。

* **Cache Miss 的测量**

    * 程序内自测很难、只能估算；实际靠硬件厂商工具把应用挂上去抓 CPU 的 L1/L2/L3 各级 miss（如 Intel 的 VTune）；GPU 侧同样有厂商提供的 profiling 工具——最底层的 spec 与接口函数只有厂商能拿到。

* **给设计师的 DOP 工具**

    * artist、designer、programmer 思维方式完全不同：designer 天然要表达自我、打破范式（要 novelty 与创意），与"每个 component 功能明确单一"的 DOP 原则天然冲突。

    * 工具原则两条：允许设计师自由组合组件（把 component 做成积木）；不替他做选择，但把每个选择的成本讲清楚（如深度定制走不了 DOP pipeline，性能可能只有十分之一）——给他总性能预算，让他自己权衡做一辆酷车还是 100 辆高效的车。

    * 预算控制（budget control）与预算可视化在实战团队极其重要——预算指 CPU/GPU 性能而非钱；配合自动化 QA：设计师改动后大量性能测试自动跑，并让他自己收到 report。

* **Lumen 对自发光材质的处理**

    * 传统做法常给自发光挂一个假光源（fake lighting）；Lumen 把自发光材质的 radiance 写进 surface cache 的 emission channel 纹理。

    * 采样 volumetric lighting 时这些 radiance 全部被采入，生成 screen space probe 时被带进屏幕空间；这一帧的光到下一帧变成 indirect lighting——自发光的 multi-bounce 逐帧传递，形成多次反弹。

    * 待验证：自发光能否支持 HDR、效果是否足够明显和准确。
