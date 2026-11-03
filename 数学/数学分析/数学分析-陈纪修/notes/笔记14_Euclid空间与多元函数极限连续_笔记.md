# Euclid空间与多元函数极限连续

## 多元函数的引入

* **为什么一个自变量不够用**

    * 自然界的现象里变量相互联系，说一个事物只受一个变量影响一般不可能——自变量、因变量都可能有多个，这是多元函数（及多元函数组）研究的对象。
    * 质点在空间中运动：$x=x(t),\ y=y(t),\ z=z(t)$，一个自变量（时间 $t$）、三个因变量（空间坐标），用方程组（函数组）描述。
    * 教室里的温度：$T$ 随空间位置（三个坐标）不同而不同，还随时间变化——四个自变量、一个因变量。
    * 河流流速：速度是有方向的向量，三个分量各写成 $f_i(x,y,z,t)$（$i=1,2,3$）——四个自变量、三个因变量。
    * 研究进程与一元函数相同：先极限，再微分，后积分。

* **距离是极限概念的前提**

    * 一元极限 $\lim_{x\to x_0}f(x)=A$ 的分析表述中两处用到绝对值，绝对值表示两点之间的距离："只要 $x$ 离开 $x_0$ 的距离充分小，$f(x)$ 与 $A$ 的距离就充分小"。
    * 一元函数的平台是数轴，两点距离就是绝对值；多元函数的平台是欧氏空间，距离要重新搭起来——没有距离，就没有极限的概念。

## 欧氏空间

* **欧几里得空间（Euclid space，欧氏空间）**

    * > **定义**：定义了内积的向量空间 $\mathbb{R}^n$；它是多元函数、多元函数组理论展开的舞台。

    * $\mathbb{R}^n=\mathbb{R}\times\mathbb{R}\times\cdots\times\mathbb{R}$ 是 $n$ 个实数集的笛卡尔乘积集合，元素是有序组 $\mathbf{X}=(x_1,x_2,\cdots,x_n)$，称为向量（点）——它不是一个实数；零向量 $\mathbf{0}=(0,0,\cdots,0)$；书写用黑体大写 $\mathbf{X},\mathbf{Y},\mathbf{A}$ 表示向量，小写字母表示分量。
    * 加法按分量相加 $\mathbf{X}+\mathbf{Y}=(x_1+y_1,\cdots,x_n+y_n)$；数乘 $\lambda\mathbf{X}=(\lambda x_1,\cdots,\lambda x_n)$。定义了加法与数乘，$\mathbb{R}^n$ 成为向量空间。
    * **内积（inner product）**：$\mathbf{X}\cdot\mathbf{Y}=\sum_{k=1}^n x_ky_k$——结果是数，不是向量；定义了内积，$\mathbb{R}^n$ 才成为欧几里得空间。
    * **范数（norm）**：$\|\mathbf{X}\|=\sqrt{\mathbf{X}\cdot\mathbf{X}}=\sqrt{\sum_{k=1}^n x_k^2}$。

* **内积的性质**

    * 正定性：$\mathbf{X}\cdot\mathbf{X}\ge0$，等号成立当且仅当 $\mathbf{X}=\mathbf{0}$。
    * 对称性：$\mathbf{X}\cdot\mathbf{Y}=\mathbf{Y}\cdot\mathbf{X}$。
    * 线性性：$(\lambda\mathbf{X}+\mu\mathbf{Y})\cdot\mathbf{Z}=\lambda(\mathbf{X}\cdot\mathbf{Z})+\mu(\mathbf{Y}\cdot\mathbf{Z})$。
    * **许瓦兹不等式（Schwarz inequality）**：$(\mathbf{X}\cdot\mathbf{Y})^2\le(\mathbf{X}\cdot\mathbf{X})(\mathbf{Y}\cdot\mathbf{Y})$，分量形式即 $\left(\sum_{i=1}^n x_iy_i\right)^2\le\left(\sum_{i=1}^n x_i^2\right)\left(\sum_{i=1}^n y_i^2\right)$。
    * 证明思路：按正定性 $(\lambda\mathbf{X}+\mathbf{Y})\cdot(\lambda\mathbf{X}+\mathbf{Y})\ge0$，用线性性与对称性展开成关于 $\lambda$ 的二次三项式；它对一切 $\lambda$ 非负，只可能是判别式 $\le0$。

## 距离、邻域与点列极限

* **距离（distance）**

    * > **定义**：两点 $\mathbf{X},\mathbf{Y}$ 的距离 $\rho(\mathbf{X},\mathbf{Y})=\|\mathbf{X}-\mathbf{Y}\|=\sqrt{(x_1-y_1)^2+\cdots+(x_n-y_n)^2}$——分量之差的平方和开根号，从 $\mathbb{R}^2$、$\mathbb{R}^3$ 的距离公式引申而来。

    * 与范数的关系：两个向量相减的范数就是这两个向量之间的距离；一个向量的范数就是它到零向量的距离。
    * 正定性：$\rho(\mathbf{X},\mathbf{Y})\ge0$，等于零当且仅当 $\mathbf{X}=\mathbf{Y}$。
    * 对称性：$\rho(\mathbf{X},\mathbf{Y})=\rho(\mathbf{Y},\mathbf{X})$。
    * 三角不等式：$\rho(\mathbf{X},\mathbf{Z})\le\rho(\mathbf{X},\mathbf{Y})+\rho(\mathbf{Y},\mathbf{Z})$——应用非常频繁的一条。
    * 验证思路：只需先证 $\|\mathbf{X}+\mathbf{Y}\|\le\|\mathbf{X}\|+\|\mathbf{Y}\|$（两边平方后用许瓦兹不等式），再把 $\mathbf{X},\mathbf{Y}$ 换成 $\mathbf{X}-\mathbf{Y},\mathbf{Y}-\mathbf{Z}$，左边即 $\rho(\mathbf{X},\mathbf{Z})$。

* **邻域（neighborhood）**

    * > **定义**：$\mathbf{A}$ 的 $\delta$ 邻域 $O(\mathbf{A},\delta)=\{\mathbf{X}:\rho(\mathbf{X},\mathbf{A})<\delta\}$——到 $\mathbf{A}$ 距离小于 $\delta$ 的那些向量的全体。

    * 形状：一维是以 $\mathbf{A}$ 为中心的开区间；二维是以 $\mathbf{A}$ 为圆心、$\delta$ 为半径的开圆盘；三维是开球。

* **点列的极限**

    * > **定义**：若存在向量 $\mathbf{A}$，对任意 $\varepsilon>0$ 存在 $K$，对一切 $k>K$ 都有 $\rho(\mathbf{X}_k,\mathbf{A})=\|\mathbf{X}_k-\mathbf{A}\|<\varepsilon$，则称点列 $\{\mathbf{X}_k\}$ 收敛于 $\mathbf{A}$，记 $\lim_{k\to\infty}\mathbf{X}_k=\mathbf{A}$。

    * 形式与一维 $\varepsilon$-$N$ 语言一模一样，只把数轴上的绝对值换成欧氏空间里的距离。
    * 下标用 $k$ 不用 $n$：$n$ 已被空间维数占用。

## 点与集合

* **补集（complement）**：$S^C=\mathbb{R}^n\setminus S$，即全空间挖掉 $S$ 剩下的部分；记号 $C$ 来自英文 complement。

* **内点（interior point）**

    * > **定义**：$\mathbf{X}\in S$，且存在 $\delta>0$ 使 $O(\mathbf{X},\delta)\subset S$。

    * 内点必属于 $S$；$S$ 的内点全体称为 $S$ 的内部，记 $S^\circ$。

* **外点（exterior point）**

    * > **定义**：存在 $\delta>0$ 使 $O(\mathbf{X},\delta)\subset S^C$——邻域里没有一个点属于 $S$。

* **边界点（boundary point）**

    * > **定义**：对任意 $\delta>0$，$O(\mathbf{X},\delta)$ 中既有 $S$ 的点又有 $S^C$ 的点——即不满足内点、外点两条性质的点。

    * 边界点全体称为 $S$ 的边界，记 $\partial S$。

* **孤立点（isolated point）**

    * > **定义**：存在 $O(\mathbf{X},\delta)$，其中只有中心 $\mathbf{X}$ 这一个点属于 $S$。

    * 孤立点一定是边界点：邻域里中心以外的点全是 $S^C$ 的点，而中心本身是 $S$ 的点。

* **聚点（accumulation point）**

    * > **定义**：$\mathbf{X}$ 的任意邻域中都含有 $S$ 中无穷多个点。

    * 等价说法一：任意邻域至少含一个异于 $\mathbf{X}$ 的 $S$ 中的点——逐次缩小邻域取点即得无穷多个互不相同的点；取的点必须非 $\mathbf{X}$ 自己，$\mathbf{X}$ 不算数。
    * 等价说法二（点列语言）：存在点列 $\{\mathbf{X}_k\}$，$\mathbf{X}_k\in S$ 且 $\mathbf{X}_k\ne\mathbf{X}$，使 $\lim_{k\to\infty}\mathbf{X}_k=\mathbf{X}$。
    * > **易错点**：⚠️ "点列收敛于 $\mathbf{X}$"推不出"$\mathbf{X}$ 是 $S$ 的聚点"——常向量点列（一个向量反复出现）就是反例；条件 $\mathbf{X}_k\ne\mathbf{X}$ 不能丢。

* **闭包（closure）**

    * > **定义**：$S$ 与它的聚点全体并起来所得的集合，记 $\bar{S}$。

    * 例：$S=\{(x,y):1<x^2+y^2\le4\}$ 的闭包为 $\{(x,y):1\le x^2+y^2\le4\}$——多出来的正是内圈 $x^2+y^2=1$，其上每一点的任何邻域里都有环带内无穷多个点，是聚点但不在 $S$ 中；外圈 $x^2+y^2=4$ 本来就在 $S$ 里。

## 开集与闭集

* **开集（open set）**

    * > **定义**：$S$ 当中的每一点都是 $S$ 的内点。

* **闭集（closed set）**

    * > **定义**：$S$ 包含了它所有的聚点。

    * 两个定义角度不同：开集从内点看，闭集从聚点看；两者的关系要靠补集定理打通。

* **邻域是开集**

    * 任取 $\mathbf{Q}\in O(\mathbf{A},\delta)$，则 $\rho(\mathbf{Q},\mathbf{A})<\delta$，取 $0<h<\delta-\rho(\mathbf{Q},\mathbf{A})$；对 $\mathbf{P}\in O(\mathbf{Q},h)$ 用三角不等式 $\rho(\mathbf{P},\mathbf{A})\le\rho(\mathbf{P},\mathbf{Q})+\rho(\mathbf{Q},\mathbf{A})<h+\rho(\mathbf{Q},\mathbf{A})<\delta$，故 $O(\mathbf{Q},h)\subset O(\mathbf{A},\delta)$——邻域中任何一点都是它的内点。

* **开球、闭球与 $n$ 维矩形**

| 集合 | 定义条件 | $n=2$ 直观 | $n=3$ 直观 |
| :--- | :--- | :--- | :--- |
| 开球 | $\rho(\mathbf{X},\mathbf{A})<R$ | 开圆盘 | 开球 |
| 闭球 | $\rho(\mathbf{X},\mathbf{A})\le R$ | 闭圆盘 | 闭球 |
| 开矩形 | $a_i<x_i<b_i$（$1\le i\le n$） | 开矩形 | 开长方体 |
| 闭矩形 | $a_i\le x_i\le b_i$（$1\le i\le n$） | 闭矩形 | 闭长方体 |

* 开球就是半径为 $R$ 的邻域——邻域是开集，开球自然是开集。

* **开与闭经补集互为镜像**

    * $S$ 是闭集的充要条件是补集 $S^C$ 是开集；对偶地，$S$ 是开集的充要条件是 $S^C$ 是闭集。
    * 必要性思路（$S$ 闭 $\Rightarrow$ $S^C$ 开）：$\mathbf{X}\in S^C$ 不是 $S$ 的聚点，故有邻域除 $\mathbf{X}$ 外不含 $S$ 的点，而 $\mathbf{X}$ 自己也不属于 $S$，整个邻域落在 $S^C$ 中。
    * 充分性思路（$S^C$ 开 $\Rightarrow$ $S$ 闭）：$S^C$ 中每点都有邻域含于 $S^C$，这些点都不是 $S$ 的聚点，于是 $S$ 的聚点一个都不落在 $S^C$ 里。

## 开集与闭集的运算

* **四条运算规则（定理 11.1.5）**

    * 任意一组开集的并 $\bigcup_{\alpha} S_{\alpha}$ 仍是开集——指标 $\alpha$ 可取有限个、可列无穷多个，甚至不可数无穷多个。
    * 有限个开集的交 $\bigcap_{i=1}^{K} S_i$ 是开集。
    * 任意一组闭集的交 $\bigcap_{\alpha} T_{\alpha}$ 是闭集。
    * 有限个闭集的并 $\bigcup_{i=1}^{K} T_i$ 是闭集。
    * 最要紧的不对称处：并这边多少个都行，交那边必须有限。
    * 任意并的证明：并中每点必落进某一个成员开集，向它借一个邻域即可，其余成员根本不用管——对不可数无穷多个开集照样成立。
    * 有限交的证明：每个 $S_i$ 各给出邻域半径 $r_i$，取 $R=\min_{1\le i\le K}r_i$——有限个数才有最小值。
    * > **易错点**：⚠️ "有限"二字丢不得——无穷多个正数 $r_i$ 可以没有正的最小值，下确界可能是 $0$，一个邻域都借不到。结论本身也确实会坏：$\bigcap_{n=1}^{\infty}\left(-\tfrac{1}{n},\tfrac{1}{n}\right)=\{0\}$ 不是开集；$\bigcup_{n=2}^{\infty}\left[\tfrac{1}{n},1\right]=(0,1]$ 不是闭集。

## 欧氏空间上的基本定理

* **实数系五大基本定理的归宿**

| 实数系基本定理 | 能否推广到 $\mathbb{R}^n$（$n\ge2$） | 原因 |
| :--- | :---: | :--- |
| 确界存在定理 | 不能 | 上界、上确界比的是大小 |
| 单调有界数列收敛定理 | 不能 | "单调"每一步都在比大小；$n\ge2$ 时点之间没有大小关系 |
| 闭区间套定理 | 能 | 搬成闭矩形套、Cantor 闭集套 |
| Bolzano–Weierstrass 定理 | 能 | 有界点列必有收敛子列 |
| 柯西收敛原理 | 能 | 点列收敛 $\iff$ 基本点列 |

### 闭矩形套定理

* **闭矩形套定理（定理 11.1.6）**

    * > **定义**：一列闭矩形 $\Delta_k=[a_k,b_k]\times[c_k,d_k]$ 满足一个套一个（$a_k\le a_{k+1}\le b_{k+1}\le b_k$，$c_k\le c_{k+1}\le d_{k+1}\le d_k$），且对角线长度的平方 $(b_k-a_k)^2+(d_k-c_k)^2\to0$，称为闭矩形套。

    * 结论：存在唯一的点 $(\xi,\eta)$ 属于每一个 $\Delta_k$。
    * 证明思路：两组边 $\{[a_k,b_k]\}$、$\{[c_k,d_k]\}$ 各自成一维闭区间套——套住的条件现成，"长度趋于零"由平方和趋于零且平方和非负逼出来；分别用闭区间套定理得唯一 $\xi$、$\eta$；若还有别的公共点，由唯一性只能 $(x,y)=(\xi,\eta)$。

### Cantor 闭集套定理

* **直径（diameter）**

    * > **定义**：$d(S)=\sup_{x,y\in S}|x-y|$——集合内任意两点距离的上确界，衡量集合最宽能有多宽。

    * 闭矩形的直径恰好就是对角线的长度。

* **Cantor 闭集套定理（定理 11.1.6′）**

    * > **定义**：$\mathbb{R}^n$ 上一列非空闭集（闭区域也可以）满足 $S_1\supseteq S_2\supseteq\cdots\supseteq S_k\supseteq\cdots$ 且 $d(S_k)\to0$，称为闭集套。

    * 结论：存在唯一的点 $x_0$ 属于一切 $S_k$。矩形形状不是本质——本质是"闭""一个套一个""直径趋于零"。
    * 证明四步：
        * 每个非空 $S_k$ 中任取一点 $\boldsymbol{x}_k$；$l>k$ 时两点同在 $S_k$ 中，故 $|\boldsymbol{x}_k-\boldsymbol{x}_l|\le d(S_k)$。
        * 由 $d(S_k)\to0$，$\{\boldsymbol{x}_k\}$ 是基本点列，用柯西收敛原理得 $\boldsymbol{x}_k\to\boldsymbol{x}_0$。
        * 固定 $k$，脚标靠后的点全落 $S_k$ 且趋于 $\boldsymbol{x}_0$：这些点要么有无穷多个互不相同（$\boldsymbol{x}_0$ 是 $S_k$ 的聚点，$S_k$ 闭、包含所有聚点），要么从某时刻起恒取同一点（该点在 $S_k$ 里且就是 $\boldsymbol{x}_0$）——两种情形都有 $\boldsymbol{x}_0\in S_k$。
        * 唯一性：若 $\boldsymbol{x}_0,\boldsymbol{y}_0$ 都属于一切 $S_k$，则 $|\boldsymbol{x}_0-\boldsymbol{y}_0|\le d(S_k)\to0$，故 $\boldsymbol{x}_0=\boldsymbol{y}_0$。
    * 应用：三角形三中线共点——三中线交点（不论一个还是三个）都在每层三角形内；取三边中点连成内层三角形，原三条中线仍是内层三角形的中线（中位线平行于对边且过其对边中点），逐层得闭区域套 $A_1B_1C_1\supset A_2B_2C_2\supset\cdots$，直径 $d\to\tfrac d2\to\tfrac d4\to\cdots\to0$；闭集套定理给出唯一公共点 $O$，若真有两个交点，它们都得同时是 $O$，矛盾——三线共点。

### Bolzano–Weierstrass 定理

* **Bolzano–Weierstrass 定理（定理 11.1.7）**

    * 内容：$\mathbb{R}^n$ 上的有界点列必有收敛子列。
    * 证明思路（$n=2$）：$\boldsymbol{x}_k=(x_k,y_k)$ 有界保证坐标 $x_k$ 是有界实数列，先用**实数系**的 Bolzano–Weierstrass 定理取收敛子列 $x_{k_i}\to x_0$；$y_{k_i}$ 仍有界，再取子列的子列 $y_{k_{i_j}}\to y_0$；合起来 $(x_{k_{i_j}},y_{k_{i_j}})\to(x_0,y_0)$。一般 $n$ 维要做 $n$ 次"子列的子列"——维数有限、固定，总做得成。

* **推论：有界无限点集必有聚点**

    * 内容：$\mathbb{R}^n$ 上有界的无限点集必有聚点（至少一个）。
    * 证明要点：无限点集保证能取出一列**互不相同**的点 $\boldsymbol{x}_{k_i}$——就怕取成常向量序列，"无限"排除它；这列点仍落在原球内有界，用定理 11.1.7 取收敛子列，极限记 $\boldsymbol{x}_0$——子列里无穷多个互不相同的点落入 $\boldsymbol{x}_0$ 的任何邻域，故 $\boldsymbol{x}_0$ 是原来点集的聚点。
    * > **易错点**：聚点属于**点集**，极限点属于**点列**。点列的极限点未必是它元素集合的聚点——就怕这列点是常向量序列（或者只有有限个不同元素）；推论里"无限"两个字就是用来堵这个漏洞的。

### 柯西收敛原理

* **基本点列（fundamental sequence）**

    * > **定义**：对任意 $\varepsilon>0$，存在正整数 $K$，对一切 $k,l>K$ 都有 $|\boldsymbol{x}_k-\boldsymbol{x}_l|<\varepsilon$——脚标够远之后，任何两项之间的距离都小于 $\varepsilon$。

* **柯西收敛原理（定理 11.1.8）**

    * 内容：$\mathbb{R}^n$ 上的点列收敛的充分必要条件是它是基本点列。
    * 必要性：$k,l>K$ 时 $|\boldsymbol{x}_k-\boldsymbol{x}_l|\le|\boldsymbol{x}_k-A|+|\boldsymbol{x}_l-A|<\tfrac\varepsilon2+\tfrac\varepsilon2=\varepsilon$（三角不等式）。
    * 充分性的关键不等式：$\big|x_i^{(k)}-x_i^{(l)}\big|\le|\boldsymbol{x}_k-\boldsymbol{x}_l|$——分量差的平方只是距离平方和里的一项；它把 $n$ 维问题拆回 $n$ 个一维基本数列，再用实数系的柯西收敛原理。

* **基本定理的全景**

    * 实数系（欧氏空间）的基本定理一般说有七个：确界存在、单调有界、闭区间套、Bolzano–Weierstrass、柯西收敛原理、海涅–博雷尔（有限覆盖）、戴德金切割。
    * 七个在实数系 $\mathbb{R}$ 中相互等价，都反映实数系的基本性质；到 $n\ge2$ 的欧氏空间，确界存在与单调有界因牵涉大小比较不能推广，戴德金切割是实数系专属——剩下四个相互等价，都反映欧氏空间的基本性质。

## 紧集与海涅–博雷尔定理

### 开覆盖与紧集

* **开覆盖（open cover）**

    * > **定义**：开集族 $\{U_\alpha\}$（取在 $\mathbb{R}^n$ 中）满足 $\bigcup_{\alpha} U_\alpha\supseteq S$，称 $\{U_\alpha\}$ 是 $S$ 的一个开覆盖。

* **紧集（compact set）**

    * > **定义**：$S$ 的**任意**一个开覆盖都存在有限子覆盖——能从原开集族中挑出有限个 $U_{\alpha_1},\cdots,U_{\alpha_P}$ 仍盖住 $S$——则称 $S$ 是紧集。

    * "子"覆盖的意思：挑出的 $P$ 个对整个开集族而言是个子集。
    * > **易错点**：紧集要求对"任意"开覆盖成立；要说明某个集合不是紧集，只需找出**一个**没有有限子覆盖的开覆盖就够了——一票否决。

### 两个一维例子

* **不紧的例子**：$S=\{1,\tfrac12,\tfrac13,\cdots\}$ 不是紧集。

    * 相邻两点间距 $\big|\tfrac1n-\tfrac1{n+1}\big|=\tfrac1{n(n+1)}$。
    * 取 $U_n$ 为以 $\tfrac1n$ 为中心、$\tfrac1{(n+1)^2}$ 为半径的邻域——半径比间距小，每个 $U_n$ 在 $S$ 里只盖住 $\tfrac1n$ 一个点。
    * $\{U_n\}_{n=1}^\infty$ 盖住 $S$，但每个开集只盖一个点，有限个开集盖不住无穷多个点——没有有限子覆盖。

* **紧的例子**：$S_0=S\cup\{0\}$ 是紧集。

    * 任一开覆盖先要盖住 $0$；盖住 $0$ 的是开集，罩住 $0$ 就连 $0$ 附近一小圈一起罩住——存在 $\varepsilon>0$ 使邻域 $O(0,\varepsilon)$ 整个落在它里面。
    * $n>1/\varepsilon$ 时 $\tfrac1n$ 全落在这个邻域里——盖住 $0$ 的同时把充分大的 $n$ 对应的点全盖住，剩下的点只有有限个，各挑一个开集伺候即可。
    * 同样一串点，添不添上聚点 $0$，紧与不紧立刻分野。

### 海涅–博雷尔定理

* **海涅–博雷尔定理（Heine–Borel theorem）**

    * 内容：$S$ 是紧集的充分必要条件是 $S$ 为有界闭集。
    * 必要性之有界：对每点 $x\in S$ 配邻域 $O(x,1)$，构成开覆盖；紧性挑出有限个 $O(x_1,1),\cdots,O(x_P,1)$，$S$ 被 $\max_{1\le i\le P}|x_i|+1$ 一个数整个框住——全靠"有限"两个字把局面定死。
    * 必要性之闭：反证 $S$ 有聚点 $A\notin S$，取 $U_n=\{x\in\mathbb{R}^n:|x-A|>\tfrac1n\}$（不等号必须**严格大于**才得到开集，写成大于等于就成了闭集），$\{U_n\}$ 盖住 $S$；取 $S$ 中互异点列 $x_k\to A$，对固定的 $U_n$，点列后段满足 $|x_k-A|<\tfrac1n$、全在 $U_n$ 外面——每个 $U_n$ 只装点列的有限项，有限个 $U_n$ 连这点列都盖不住，与紧性矛盾。
    * 充分性：反证 $S$ 不紧，存在没有有限子覆盖的开覆盖 $\{U_\alpha\}$。用闭矩形 $I_1$ 把有界的 $S$ 整个抱住，四等分（高维即每个坐标方向对半切），四块中必有一块 $I_k\cap S$ 仍不能被有限个 $U_\alpha$ 覆盖（否则四份有限个开集并起来还是有限个），取出对应矩形记 $I_2$，一直做下去，得闭矩形套、直径趋于零。由闭矩形套定理存在唯一点 $A$ 属于一切 $I_k$；$I_k\cap S$ 非空，取 $x_k$，则 $|x_k-A|\le\operatorname{diam}(I_k)\to0$，即 $x_k\to A$，$A$ 是 $S$ 的聚点，$S$ 闭故 $A\in S$。盖住 $A$ 的某个 $U_{\alpha^*}$ 内含邻域 $O(A,R)$，$k$ 充分大时 $\operatorname{diam}(I_k)<R$ 且 $A\in I_k$，整个 $I_k\cap S$ 落进 $O(A,R)$——嘴硬盖不住的一块被**一个**开集盖住，矛盾。

### 紧集的三条等价刻画

* **定理 11.1.4**

    * 内容：$S\subset\mathbb{R}^n$，则下列三条命题等价：(1) $S$ 是有界闭集；(2) $S$ 是紧集；(3) $S$ 的任何一个无限子集在 $S$ 中有聚点。
    * 有界闭 $\Rightarrow$ 无限子集有聚点：$S$ 的无限子集 $T$ 也有界，必有聚点；$T$ 的聚点也是 $S$ 的聚点，$S$ 闭故聚点落进 $S$。
    * 无限子集有聚点 $\Rightarrow$ 闭：对 $S$ 的任一聚点 $A$，取 $S$ 中互异点列 $x_k\to A$，$\{x_k\}$ 是 $S$ 的无限子集，由条件它在 $S$ 中有聚点；而一列两两不同、收敛到 $A$ 的点聚点只可能是 $A$（任何子列都收敛到 $A$），故 $A\in S$。
    * 无限子集有聚点 $\Rightarrow$ 有界：反证 $S$ 无界，可取 $\lim_{k\to\infty}|x_k|=+\infty$ 的点列（趋于无穷的点列不可能从某项起是常向量列），它构成 $S$ 的无限子集，但对任何点 $B$，从某项起 $|x_k|>|B|+1$、全跑出 $B$ 的单位邻域——它在整个空间里都没有聚点，矛盾。直观例：$1,2,3,\cdots$ 没有聚点。

## 多元函数与它的图像

* **多元函数（function of several variables）**

    * > **定义**：$D$ 是 $n$ 维欧氏空间 $\mathbb{R}^n$ 中的点集，$f$ 是从 $D$ 到 $\mathbb{R}$ 的映射，记 $z=f(\boldsymbol{x})=f(x_1,x_2,\cdots,x_n)$；$D$ 称为定义域，$f(D)=\{z:z=f(\boldsymbol{x}),\boldsymbol{x}\in D\}$ 称为值域。

    * 图像：点集 $\{(\boldsymbol{x},z):\boldsymbol{x}\in D,\ z=f(\boldsymbol{x})\}\subset\mathbb{R}^{n+1}$——一元函数图像是平面上的曲线，$n$ 元函数图像落在 $n+1$ 维空间。
    * 例：$z=\sqrt{1-\tfrac{x^2}{a^2}-\tfrac{y^2}{b^2}}$ 的定义域是椭圆区域 $\tfrac{x^2}{a^2}+\tfrac{y^2}{b^2}\le1$，图像是三维空间里张在这个椭圆上的上半椭球面（$z\ge0$ 的那一半）。

## 多元函数的极限

* **极限的 $\varepsilon$-$\delta$ 定义**

    * > **定义**：设 $D\subset\mathbb{R}^n$ 是开集，$\boldsymbol{x}_0\in D$，$f$ 定义在 $D$ 去掉 $\boldsymbol{x}_0$ 上，$A$ 是实数。若对任意 $\varepsilon>0$，存在 $\delta>0$，对一切满足 $0<\|\boldsymbol{x}-\boldsymbol{x}_0\|<\delta$ 的 $\boldsymbol{x}$ 都有 $|f(\boldsymbol{x})-A|<\varepsilon$，则称 $\boldsymbol{x}\to\boldsymbol{x}_0$ 时 $f(\boldsymbol{x})$ 以 $A$ 为极限，记 $\lim_{\boldsymbol{x}\to\boldsymbol{x}_0}f(\boldsymbol{x})=A$。

    * $0<\|\boldsymbol{x}-\boldsymbol{x}_0\|<\delta$ 即 $\boldsymbol{x}\in O(\boldsymbol{x}_0,\delta)\setminus\{\boldsymbol{x}_0\}$：求极限的过程中 $\boldsymbol{x}\ne\boldsymbol{x}_0$；$f$ 在 $\boldsymbol{x}_0$ 有没有定义、有的话值是多少，统统不考虑——极限与该点取值无关，一元、多元同一套逻辑。
    * 坐标写法：$|x_i-x_i^0|<\delta_i$（$i=1,\cdots,n$）且 $\boldsymbol{x}\ne\boldsymbol{x}_0$；坐标写法的 $\delta_i$ 与圆盘写法的 $\delta$ 不必相同——圆盘邻域和方块邻域要互相容纳，半径与边长差一个倍数。
    * > **易错点**：⚠️ "$\boldsymbol{x}\ne\boldsymbol{x}_0$"只能对整个点写一次，绝不能拆到每个坐标上写成 $0<|x_i-x_i^0|$——那会把过 $\boldsymbol{x}_0$ 的坐标超平面全扣掉（平面上扣掉两条线，空间里扣掉三个互相垂直的平面），邻域都不完整了。
    * 定义证明示范：$\lim_{(x,y)\to(0,0)}(x+y)\sin\tfrac1{x^2+y^2}=0$——因子 $\sin\tfrac1{x^2+y^2}$ 有界、$x+y$ 趋零（有界量乘无穷小），估计 $|f|\le|x|+|y|$，取 $\delta=\tfrac\varepsilon2$ 即得 $|f|<\tfrac\varepsilon2+\tfrac\varepsilon2=\varepsilon$。

* **趋近方式：多元特有的麻烦**

    * 一元函数 $x\to x_0$ 只有左、右两种方式，左极限右极限都等于 $A$ 就行。
    * 二元极限要求 $(x,y)$ 以**任何方式**趋向 $(x_0,y_0)$ 都得到同一个 $A$：直线方向（斜率 $k$ 一变就是一条新的射线）、曲线（以相切的角度滑向）、螺线（圈越转越小地逼近）——方式根本列不完。

    * 路径形态：

    ```text
    一元函数：x 趋近 x0 只有两种方式

          左极限 -------->  x0  <-------- 右极限

    二元函数：(x, y) 趋近 O 的方式无法穷举
      (a) 直线方向：斜率 k 一变，就是一条新的射线
      (b) 沿曲线：以相切的角度滑向 O
      (c) 沿螺线：圈越转越小地收缩逼近 O
    ```

    * 判不存在的标准：只要有两种方式，$(x,y)$ 以这两种方式趋向 $(x_0,y_0)$ 得到的极限不一样，二重极限就不存在。
    * > **注意**："沿所有直线极限都相同"远远不够——直线只是众多趋近方式里的一类。

* **两个判不存在的标准例子**

    * $f(x,y)=\dfrac{xy}{x^2+y^2}$：沿直线 $y=kx$ 代入得 $\dfrac{k}{1+k^2}$，随 $k$ 不同而不同——极限不存在。
    * $f(x,y)=\dfrac{(y^2-x)^2}{y^4+x^2}$：沿一切直线 $y=kx$ 极限都是 $1$（分子提出 $x^2$ 后剩 $(k^2x-1)^2\to1$，与 $k$ 无关），沿 $y$ 轴（$x=0$，$y^4$ 比 $y^4$）极限也是 $1$；但沿抛物线 $y^2=x$，分子恒为零、分母 $2y^4\ne0$，极限是 $0$——两种方式极限不同，极限仍不存在。

* **极限的性质与四则运算**

    * 唯一性：极限若存在必唯一。
    * 局部有界性：极限存在则函数在该点的小邻域内有界；"局部"——邻域不能开得很大，开大了函数可以在较大邻域里的某些点附近无界。
    * 局部保序性：在去心邻域内 $f(\boldsymbol{x})\ge g(\boldsymbol{x})$ 且两极限都存在，则 $\lim f\ge\lim g$。
    * 局部夹逼性：去心邻域内 $h(\boldsymbol{x})\le f(\boldsymbol{x})\le g(\boldsymbol{x})$ 且 $\lim h=\lim g=A$，则 $\lim f=A$。
    * 四则运算法则与一元一致；用除法法则前必须先确认**分母的极限不等于零**——分母极限为零，法则就不能用。

## 二重极限与二次极限

* **两个概念的分工**

    * > **定义**：二重极限（double limit）$\lim_{(x,y)\to(x_0,y_0)}f(x,y)$——$x$、$y$ 处于平等地位，$(x,y)$ 作为一个点整体趋向，任何方式都得同一极限。

    * > **定义**：二次极限（iterated limit，累次极限）——先 $x$ 后 $y$：对每个 $y\ne y_0$，$\varphi(y)=\lim_{x\to x_0}f(x,y)$ 存在，且 $\lim_{y\to y_0}\varphi(y)$ 存在；先 $y$ 后 $x$ 的二次极限同理。

    * 两者写法完全不同：二重极限没有先后次序；二次极限每一步按一元极限来求，有先后。
    * 读二次极限定义的两处停顿：里层极限不存在，整个二次极限就没有意义——定义分两步走，先里层后外层；定义只要求 $\varphi(y)$ 对每个 $y\ne y_0$ 存在——外层极限在 $y\to y_0$ 的过程中根本用不上 $y=y_0$ 这一点。

* **三组反例：谁也推不出谁**

| 例子 | 二重极限 | 两个二次极限 | 说明 |
| :--- | :--- | :--- | :--- |
| $\dfrac{xy}{x^2+y^2}$ | 不存在 | 都存在，均为 $0$ | 二次存在推不出二重存在 |
| $(x+y)\sin\tfrac1x\sin\tfrac1y$（坐标轴上取 $0$） | 存在，为 $0$ | 都不存在 | 二重存在推不出二次存在 |
| $\dfrac{x^2(1+x^2)-y^2(1+y^2)}{x^2+y^2}$ | — | 都存在：$-1$ 与 $1$ | 二次极限的次序不可交换 |

    * 反例一的算法：固定 $y\ne0$ 时分子趋于 $0$、分母趋于 $y^2\ne0$，内层极限为 $0$，再让 $y\to0$ 仍得 $0$；换次序同样 $0$——但二重极限不存在。
    * 反例二的机关：$\tfrac1x$、$\tfrac1y$ 在坐标轴上无意义，故规定 $x=0$ 或 $y=0$ 时函数值取 $0$；其余地方 $\sin\tfrac1x$、$\sin\tfrac1y$ 有界而 $(x+y)\to0$，二重极限为 $0$。求二次极限要对每个 $y\ne0$ 算内层极限：$y=\tfrac1{n\pi}$ 时 $\sin\tfrac1y=\sin n\pi=0$，沿这条横线函数恒为零，内层极限存在；$y\ne\tfrac1{n\pi}$ 时 $\sin\tfrac1y$ 是固定非零常数，$\lim_{x\to0}(x+y)\sin\tfrac1x$ 振荡不收敛——内层极限要求对每个点都存在，缺一个就挂。
    * 反例三的计算：先 $x$ 后 $y$ 得 $\lim_{y\to0}\big[-(1+y^2)\big]=-1$；先 $y$ 后 $x$ 得 $\lim_{x\to0}\big(1+x^2\big)=1$——两个二次极限都存在但不相等，次序不能交换。

* **三者相等的条件（定理 11.2.1）**

    * 内容：若二重极限 $\lim_{(x,y)\to(x_0,y_0)}f(x,y)=A$ 存在，且当 $x\ne x_0$ 时内层极限 $\lim_{y\to y_0}f(x,y)$ 存在（记 $\varphi(x)$），则 $\lim_{x\to x_0}\varphi(x)=A$——先 $y$ 后 $x$ 的二次极限存在且等于二重极限；把条件对称地换到另一边（当 $y\ne y_0$ 时 $\lim_{x\to x_0}f(x,y)$ 存在，记 $\psi(y)$），则 $\lim_{y\to y_0}\psi(y)=A$。
    * 证明三步：二重极限给出全部本钱——去心圆盘内 $|f(x,y)-A|<\tfrac\varepsilon2$；固定满足 $0<|x-x_0|<\delta$ 的 $x$，让 $y\to y_0$ 对不等式取极限得 $|\varphi(x)-A|\le\tfrac\varepsilon2<\varepsilon$；收口正是 $\lim_{x\to x_0}\varphi(x)=A$ 的 $\varepsilon$-$\delta$ 定义。
    * 取 $\tfrac\varepsilon2$ 的用意：严格的小于号取极限后退化为 $\le$，留出的余地保证不等式依旧有效，而 $\tfrac\varepsilon2$ 本身小于 $\varepsilon$。
    * 推论：若 $f$ 在一点的二重极限与两个二次极限都存在，则三者相等——两个二次极限可以交换次序。

## 多元连续函数

* **连续的定义**

    * > **定义**：$f$ 定义在整个 $D$ 上（$\boldsymbol{x}_0\in D$），若 $\lim_{\boldsymbol{x}\to\boldsymbol{x}_0}f(\boldsymbol{x})=f(\boldsymbol{x}_0)$，则称 $f$ 在 $\boldsymbol{x}_0$ 连续；若 $f$ 在 $D$ 的任意点都连续，则称 $f$ 在 $D$ 上连续。

    * 比"极限存在"多两点要求：$f$ 在 $\boldsymbol{x}_0$ 必须有定义（算极限时不要求）；定义的值必须正好等于极限值。
    * $\varepsilon$-$\delta$ 表述：对任意 $\varepsilon>0$ 存在 $\delta>0$，$\|\boldsymbol{x}-\boldsymbol{x}_0\|<\delta$ 时 $\|f(\boldsymbol{x})-f(\boldsymbol{x}_0)\|<\varepsilon$——前面**没有"大于零"**：$\boldsymbol{x}=\boldsymbol{x}_0$ 时 $\|f(\boldsymbol{x}_0)-f(\boldsymbol{x}_0)\|=0<\varepsilon$ 自动成立，挖与不挖一个样。连续性不挖点：函数在该点有定义，且取的值恰好就是极限值。

* **多元初等函数连续性**

    * 多元初等函数在其定义域上连续——以后看到初等函数，不必逐点用 $\varepsilon$-$\delta$ 语言验证。
    * 多元初等函数的构成：基本初等函数（幂函数、指数函数、对数函数、三角函数、反三角函数，变量可以不止一个）经有限次加减乘除与复合得到。
    * 一元连续函数的那套运算——和、差、积、商与复合函数的性质——同样推广到多元函数。

* **求极限的两个示范**

    * 定义域内直接代：$\lim_{(x,y)\to(1,0)}\dfrac{\ln(x+\mathrm{e}^y)}{\sqrt{x^2+y^2}}=\dfrac{\ln2}{1}=\ln2$——先检查 $(1,0)$ 在定义域内（分母 $\sqrt{1+0}=1\ne0$、分子 $\ln 2$ 有意义），再借多元初等函数的连续性代值。
    * > **易错点**：⚠️ 代值之前先确认这个点在函数的定义域里——这是连续性代值的入场券。
    * 未定型化归一元经验：$\lim_{(x,y)\to(0,0)}\dfrac{\sin\big[(1+y)\sqrt{x^2+y^2}\big]}{\sqrt{x^2+y^2}}=1$——$(0,0)$ 不在定义域（分母为零），是 $\tfrac00$ 型；凑 $\dfrac{\sin u}{u}\to1$（$u=(1+y)\sqrt{x^2+y^2}\to0$），乘上极限为 $1$ 的因子 $(1+y)$，得 $1\times1=1$。

* **全程 $\varepsilon$-$\delta$ 示范**

    * 例：$f(x,y)=\sin\sqrt{x^2+y^2}$ 在整个平面上连续。
    * 放缩链：和差化积 $\sin u-\sin v=2\sin\tfrac{u-v}2\cos\tfrac{u+v}2$，用 $\big|\cos\tfrac{u+v}2\big|\le1$ 与 $|\sin t|\le|t|$ 得 $|f(x,y)-f(x_0,y_0)|\le\big|\sqrt{x^2+y^2}-\sqrt{x_0^2+y_0^2}\big|$。
    * 几何收网：右边是 $P,Q$ 两点到原点的距离之差，三角形"两边之差不超过第三边"给出 $\le\sqrt{(x-x_0)^2+(y-y_0)^2}$（$P$ 到 $Q$ 的距离）；对任意 $\varepsilon>0$ 取 $\delta=\varepsilon$，全程收口。

## 向量值函数

* **向量值函数（vector-valued function）**

    * > **定义**：映射 $\mathbf{F}:D\subset\mathbb{R}^n\to\mathbb{R}^m$，称为 $n$ 元 $m$ 维向量值函数（向量函数），记 $\mathbf{z}=\mathbf{F}(\mathbf{x})$；$m$ 个分量 $z_i=f_i(x_1,\cdots,x_n)$ 各是一个普通 $n$ 元函数，也记作 $\mathbf{F}=(f_1,f_2,\cdots,f_m)$。

    * 与多元函数的区别：多元函数值域在 $\mathbb{R}^1$，函数值是数；向量值函数值域在 $\mathbb{R}^m$，函数值是向量。
    * 几何原型一：空间曲线的参数方程 $x=x(t),\ y=y(t),\ z=z(t)$，$t\in[a,b]$——一个自变量、三个因变量。
    * 几何原型二：空间曲面的参数表示 $x=x(u,v),\ y=y(u,v),\ z=z(u,v)$，$(u,v)\in D\subset\mathbb{R}^2$——两个自变量、三个因变量。

* **向量值函数的极限**

    * > **定义**：若存在向量 $\boldsymbol{A}\in\mathbb{R}^m$，对任意 $\varepsilon>0$ 存在 $\delta>0$，对一切满足 $0<\|\boldsymbol{x}-\boldsymbol{x}_0\|<\delta$ 的 $\boldsymbol{x}$ 都有 $\|\mathbf{F}(\boldsymbol{x})-\boldsymbol{A}\|<\varepsilon$，则称 $\mathbf{F}$ 在 $\boldsymbol{x}_0$ 的极限为 $\boldsymbol{A}$，记 $\lim_{\boldsymbol{x}\to\boldsymbol{x}_0}\mathbf{F}(\boldsymbol{x})=\boldsymbol{A}$。

    * 与多元函数极限的唯一差别：极限值是向量；式中"绝对值"是 $\mathbb{R}^m$ 空间中两点的距离。
    * 用箭头记号 $f(\boldsymbol{x})\to\boldsymbol{A}$ 时，必须把自变量是谁、它趋向什么写清楚，否则记号不完整。

* **向量值函数的连续**

    * > **定义**：$\lim_{\boldsymbol{x}\to\boldsymbol{x}_0}\mathbf{F}(\boldsymbol{x})=\mathbf{F}(\boldsymbol{x}_0)$，则 $\mathbf{F}$ 在 $\boldsymbol{x}_0$ 连续；$\varepsilon$-$\delta$ 表述里同样不挖 $\boldsymbol{x}_0$（$\boldsymbol{x}=\boldsymbol{x}_0$ 时 $\|f(\boldsymbol{x}_0)-f(\boldsymbol{x}_0)\|=0<\varepsilon$ 自动成立，写不写"大于零"都一样，通常省掉）。

    * **分量判据（定理 11.2.2）**：$f:D\to\mathbb{R}^m$ 的 $m$ 个分量为 $f_1,\cdots,f_m$，则 $f$ 在 $\boldsymbol{x}_0$ 连续的充要条件是 $f_1,\cdots,f_m$ 都在 $\boldsymbol{x}_0$ 连续——判断连续只需逐个看分量，配上"初等多元函数在定义域内连续"即得现成判别法。
    * 证明核心不等式链：$\big|f_j(\boldsymbol{x})-f_j(\boldsymbol{x}_0)\big|\le\|f(\boldsymbol{x})-f(\boldsymbol{x}_0)\|=\sqrt{\sum_{i=1}^m\big(f_i(\boldsymbol{x})-f_i(\boldsymbol{x}_0)\big)^2}\le\sum_{i=1}^m\big|f_i(\boldsymbol{x})-f_i(\boldsymbol{x}_0)\big|$——两头一夹：正方向每分量差不超过距离；反方向让和式每项小于 $\varepsilon/M$。
    * > **易错点**：⚠️ 开根号里求和的下标 $1$ 到 $m$ 是函数值的分量个数，别跟自变量的维数 $n$ 混了：$\boldsymbol{x}\in\mathbb{R}^n$ 有 $n$ 个分量，$f(\boldsymbol{x})\in\mathbb{R}^m$ 有 $m$ 个分量，平方加起来开根号的是这 $m$ 个分量。
    * **复合映射（composite mapping）的连续性（定理 11.2.3）**：前提 $g(D)\subseteq\omega$——$g$ 的值域落在 $f$ 的定义域内，有了它才谈得上复合 $h=f\circ g:D\to\mathbb{R}^m$；若 $g$ 在 $D$ 上连续、$f$ 在 $\omega$ 上连续，则 $f\circ g$ 在 $D$ 上连续（证明同一元复合函数）。

## 点集上的连续性

* **推广到任意点集**

    * > **定义**：$K\subset\mathbb{R}^n$ 是任意点集（对 $K$ 不加任何开或闭的要求），$\boldsymbol{x}_0\in K$；若对任意 $\varepsilon>0$ 存在 $\delta>0$，对一切属于 $\boldsymbol{x}_0$ 的 $\delta$ 邻域**且属于 $K$** 的向量 $\boldsymbol{x}$（即 $\boldsymbol{x}\in U(\boldsymbol{x}_0,\delta)\cap K$），都有 $\|f(\boldsymbol{x})-f(\boldsymbol{x}_0)\|<\varepsilon$，则称 $f$ 在 $\boldsymbol{x}_0$ 连续；在 $K$ 的每一点连续则称 $f$ 在 $K$ 上连续。

    * 关键在"$\cap K$"这一下：邻域里可能既有 $K$ 中的点又有 $K$ 以外的点，只考虑属于 $K$ 的那些点——$K$ 以外的点 $f$ 根本没有定义，不管它。
    * 为什么要推广：紧集（有界闭集）的每一点未必是内点——它有边界点，原先只在区域内点上给的连续定义覆盖不了边界。
    * 一元对应：老定义只够处理开区间内部（每点都是内点）；"在 $[a,b]$ 上连续"老说法要补"端点 $a$ 右连续、端点 $b$ 左连续"，新定义一句话统一——$x_0$ 在内部还是端点一律适用。这个推广对紧集、闭集、非开非闭的集合都成立。

## 紧集上的连续函数

* **连续映射将紧集映成紧集（定理 11.3.1）**

    * 内容：$K\subset\mathbb{R}^n$ 是紧集，$f:K\to\mathbb{R}^m$ 是连续映射（每个分量都是连续函数），则 $f(K)$ 是 $\mathbb{R}^m$ 上的紧集。
    * 论证抓手：紧集的三个等价命题——列紧（集合中任何点列必有收敛子列，且极限属于这个集合）、有界闭集、任何无限点集在集合中有聚点；任何无限点集必含一个无限点列，所以只需证 $f(K)$ 中任一无限点列在 $f(K)$ 里有收敛到集内点的子列。
    * 证明路线：取 $\{\boldsymbol{y}_k\}\subset f(K)$，各取原象 $\boldsymbol{x}_k\in K$ 使 $f(\boldsymbol{x}_k)=\boldsymbol{y}_k$；$K$ 列紧给收敛子列 $\boldsymbol{x}_{k_i}\to\boldsymbol{a}$，且 $\boldsymbol{a}\in K$（$K$ 是有界闭集，闭集把极限点留在自己家里）；$f$ 连续（序列语言）给 $f(\boldsymbol{x}_{k_i})\to f(\boldsymbol{a})\in f(K)$——即 $\{\boldsymbol{y}_k\}$ 有子列收敛到 $f(K)$ 中的点，$f(K)$ 是紧集。

* **有界性定理与最值定理（定理 11.3.2）**

    * 内容：$K$ 是紧集，$f$ 是 $K$ 上的连续函数（实值），则 (1)（有界性定理）$f$ 在 $K$ 上有界；(2)（最值定理）$f$ 在 $K$ 上可取到最大值与最小值——存在 $\xi_1,\xi_2\in K$，对一切 $\boldsymbol{x}\in K$ 有 $f(\xi_1)\le f(\boldsymbol{x})\le f(\xi_2)$。
    * 机理：由定理 11.3.1，$f(K)$ 是实轴上的紧集——有界闭集；$\sup f(K)$ 是有限值，且属于 $f(K)$（上确界不是 $f(K)$ 中的点就是它的聚点，闭集包含所有聚点）——于是 $\sup$ 可以改写成 $\max$、$\inf$ 可以改写成 $\min$。

* **一致连续（uniform continuity）**

    * > **定义**：$K\subset\mathbb{R}^n$，$f:K\to\mathbb{R}^m$；若对一切 $\varepsilon>0$ 存在 $\delta>0$，对任意 $\boldsymbol{x}',\boldsymbol{x}''\in K$，只要 $\|\boldsymbol{x}'-\boldsymbol{x}''\|<\delta$ 就有 $\|f(\boldsymbol{x}')-f(\boldsymbol{x}'')\|<\varepsilon$，则称 $f$ 在 $K$ 上一致连续。

    * "一致"的含义：$\delta$ 对整个 $K$ 通用，不管两点落在 $K$ 的哪里；逐点连续的 $\delta$ 是跟着点走的（记作 $\delta(a)$，换一个点换一个值）——一致连续要消灭的正是这一点。
    * > **提示**：定义中的 $\|f(\boldsymbol{x}')-f(\boldsymbol{x}'')\|$ 是 $\mathbb{R}^m$ 空间中两个向量的距离，不是两个数的绝对值。

* **一致连续性定理（定理 11.34，康托定理的推广）**

    * 内容：$K\subset\mathbb{R}^n$ 是紧集，$f:K\to\mathbb{R}^m$ 是连续映射，则 $f$ 在 $K$ 上一致连续。
    * 证明四步：
        * 逐点连续给每点 $a\in K$ 一个 $\delta(a)>0$：$x\in K\cap U(a,\delta(a))$ 时 $|f(x)-f(a)|<\tfrac\varepsilon2$——取半个 $\varepsilon$ 是刻意留的一手，待会儿把 $f(x')$ 到 $f(x'')$ 的差距拆成两段、用三角不等式拼回去各摊一半。
        * 以 $\tfrac{\delta(a)}2$ 为半径的开球全体盖住 $K$，构成开覆盖；紧性挑出有限子覆盖 $O_i=U\big(a_i,\tfrac{\delta(a_i)}2\big)$，$i=1,\cdots,p$。
        * 取 $\delta=\min_{1\le i\le p}\tfrac{\delta(a_i)}2>0$——有限个正数必有最小者，这个 $\delta$ 已不跟任何具体的点走。
        * 验证：$x'\in O_i$ 时 $|x'-a_i|<\tfrac{\delta(a_i)}2$；$x''$ 未必在小球里，但 $|x''-a_i|\le|x''-x'|+|x'-a_i|<\delta+\tfrac{\delta(a_i)}2\le\delta(a_i)$（最后一步用"$\delta$ 是最小值"）——$x''$ 可能出小球 $O_i$，却出不了大球 $U(a_i,\delta(a_i))$；两点同在 $K\cap U(a_i,\delta(a_i))$ 内，各自与 $f(a_i)$ 的差都小于 $\tfrac\varepsilon2$，三角不等式一拼小于 $\varepsilon$。

    * 几何图景：

    ```text
        +--------------------------------+
        |        U(a_i, δ(a_i))          |
        |    +--------------------+      |
        |    | U(a_i, δ(a_i)/2)   |      |
        |    |         x'         | x''  |
        |    +--------------------+      |
        +--------------------------------+
    ```

    * 半径砍成一半的作用：给 $x''$ 留出"可能出小球、出不了大球"的余量。
    * > **注意**：整个证明的命门在紧性——若 $K$ 不是紧集，盖住它就得用无限个开球，$\delta(a)$ 有无限多个，"取最小值"实际成了取下确界，而下确界完全可能是 $0$；$\delta$ 一旦是 $0$，一致连续性就无从证起。

## 连通性与中间值定理

* **闭区间的本质**

    * 闭区间 $[a,b]$ 剥掉形式后的三条本质性质：有界、闭、连通——它就是"一段"，没有分成两段。
    * 有界加闭集正好是紧集；紧集加连通，相当于一维实数范围内的闭区间。
    * 一元闭区间上连续函数的四件家底及用途：有界性——闭区间上的连续函数一定有界；最值定理——最大值最小值一定存在，一看是闭区间就知道最值问题有解，开区间则可能没解；介值定理——函数值取遍介于两个函数值之间的一切值，包含零点存在定理，可先判断方程有没有根、根大致在什么范围；康托定理——闭区间上的连续函数一定一致连续，定积分的存在性就靠它（闭区间上连续函数必可积）。

* **道路（path）**

    * > **定义**：连续映射 $\gamma:[0,1]\to\mathbb{R}^n$ 且 $\gamma([0,1])\subset S$，称 $\gamma$ 是 $S$ 中的一条道路；$\gamma(0)$ 称为起点，$\gamma(1)$ 称为终点——也说 $\gamma$ 是 $S$ 中连接 $\gamma(0)$ 与 $\gamma(1)$ 两点的一条道路。

    * 直观：一条不许离开 $S$ 的连续曲线；严格讲 $\gamma$ 是一个映射，"道路"按直观本该指它的像——空间里那条曲线。
    * > **提示**：定义只要求道路**存在**，道路可以有好几条——证明里能找出一条就够。

* **道路连通（path connected）**

    * > **定义**：若 $S$ 中任意两点 $x$ 与 $y$，都存在 $S$ 当中的道路以 $x$ 为起点、以 $y$ 为终点，则称 $S$ 是道路连通的，简称连通，这样的 $S$ 称为连通集（connected set）。

    * 欧氏空间里，道路连通与拓扑学里的连通等价、一致，不打架。
    * 不连通为何没有路：$S$ 分成互不相连的两块时，连接左块 $x$ 与右块 $y$ 的映射走到半路必须离开左块再落到右块——中间这一下必须"跳"，$\gamma$ 就不连续了；"既要连续，又要像全部包含在 $S$ 里"两个条件合在一起，逼着 $S$ 连成一片。

* **连续映射把连通集映成连通集**

    * 内容：$D\subset\mathbb{R}^n$ 是连通集，$f:D\to\mathbb{R}^m$ 是连续映射，则 $f(D)$ 是连通集。
    * 证明三步：任取 $z_1,z_2\in f(D)$，取原像 $x,y\in D$ 使 $f(x)=z_1$、$f(y)=z_2$；$D$ 连通给出道路 $\gamma$（连续、像含于 $D$、连接 $x$ 与 $y$）；复合 $f\circ\gamma$ 连续、像含于 $f(D)$、两端点恰为 $z_1,z_2$——它就是 $f(D)$ 中连接 $z_1$ 与 $z_2$ 的一条道路。

* **中间值定理（定理 11.36）**

    * 铺垫：$K$ 紧，则 $f(K)$ 紧——实轴上的有界闭集；$K$ 连通，则 $f(K)$ 连通——实数轴上连通集就是区间；紧又连通，$f(K)$ 恰是一个有界闭区间。反过来，$K$ 若只有紧没有连通，$f(K)$ 仍是有界闭集，却不一定是区间。
    * 推论：多元连续函数将连通紧集映射成闭区间。
    * 内容：$K\subset\mathbb{R}^n$ 是连通紧集，$f$ 在 $K$ 上连续，则 $f$ 可取到最大值 $M$ 与最小值 $m$ 之间的一切值。
    * 最大最小值提得出来靠的是紧性（紧集上的连续函数必取到最值）；$f(K)$ 恰是闭区间 $[m,M]$，区间里的每一个值自然都被取到。
    * 零点存在定理一并包入：只要最大值 $M>0$、最小值 $m<0$，$0$ 就夹在中间，$f$ 在 $K$ 上必有零点。
    * 分工收尾：有界性、最大最小值、一致连续靠**紧性**；中间值定理靠**紧性加连通性**。
