# Fourier级数

## 从泰勒级数到傅里叶级数

* 用简单函数逼近复杂函数

    * 数学的常用手法：简单函数处理起来方便，若它跟一个复杂函数很接近，处理简单函数得到的结论就可能用到复杂函数上

    * 泰勒级数（Taylor series）是这条路上第一个著名成果：$f$ 条件较好（在一点邻域内任意阶可导）时可写 $f(x)=\sum_{n=0}^{\infty}a_n(x-x_0)^n$，求和号里全是幂函数这种最简单的函数

* 泰勒展开的局限

    * 条件苛刻：$f$ 得任意阶可导；实际只加有限项、加到第 $N$ 项时函数至少得有 $N$ 阶导数

    * 近似只在展开点 $x_0$ 附近吻合，离开 $x_0$ 一段距离近似就不好了——这是 Taylor 近似的根本局限

* 傅里叶级数（Fourier series）

    * > **定义**：19 世纪初傅里叶（Fourier）在研究热传导方程（一个偏微分方程）时建立的三角级数展开 $\sum_{n=1}^{\infty}\left(a_n\cos nx+b_n\sin nx\right)$，用三角级数近似代替一个函数

    * 右端是一串周期函数的和，合起来是周期 $2\pi$ 的周期函数，所以被展开的函数必须是周期 $2\pi$ 的函数

    * 对 $f$ 的要求低：不要求可导、甚至不要求连续，只要求黎曼可积或在反常积分意义下绝对可积

    * 逼近是总体上、整体上的逼近，不是"仅仅在某一点附近很好、别的点不行"——这一点比泰勒级数好得多

    * 应用面宽：声学、电学、光学、热力学里波的传播都有周期性；在偏微分方程里更是基本工具

| 对比 | 泰勒级数 | 傅里叶级数 |
| :--- | :--- | :--- |
| 用的"简单函数" | 幂函数 $(x-x_0)^n$ | 三角函数 $\cos nx$、$\sin nx$ |
| 对 $f$ 的要求 | 任意阶可导（取到第 $N$ 项也要 $N$ 阶可导） | 不要求可导，甚至不要求连续 |
| 逼近范围 | 只在展开点 $x_0$ 附近吻合 | 总体上、整体上的逼近 |

* 围绕傅里叶级数的三个问题

    * 如何把 $f$ 展开成傅里叶级数

    * 展开后的级数收敛吗？收敛的话收敛于 $f$ 吗？不收敛于 $f$ 时收敛于什么

    * 级数所表示的函数是否连续、可导、可积

## 基本前提与周期延拓

* 可积或绝对可积

    * > **定义**：$f$ 在 $[-\pi,\pi]$ 上要么黎曼可积、要么在反常积分意义下绝对可积，这两句话简写成"可积或绝对可积"

    * 条件已很低：允许有间断点（不连续的点不要太多总是黎曼可积的）；甚至允许无界（无界时积分是反常积分，要求绝对可积）

    * 周期函数在哪个周期区间上加条件都一样，取 $[-\pi,\pi]$ 只是为了方便

* 周期延拓

    * $f$ 只给在一个长度 $2\pi$ 的区间上，而展开右端定义在整条实轴上，所以要按周期把函数值铺满 $(-\infty,+\infty)$；做傅里叶展开的对象就是这个延拓后的周期函数

    * 区间左右端点函数值不一样时不能直接延拓；按"每个周期点一个函数值"的规则铺开，接不上的点扣掉一个值（扣哪个都行）

* 改动有限个点的函数值

    * 不影响可积性，也不影响任何一个傅里叶系数的计算；做积分时有限个点可以直接不管

## 三角函数系的正交性

* 三角函数系的正交性

    * > **定义**：函数列 $1,\ \cos x,\ \sin x,\ \cdots,\ \cos nx,\ \sin nx,\ \cdots$ 两两正交——"正交"按积分来说：任意两个不同的函数相乘在 $[-\pi,\pi]$ 上积分为零

    * 同名函数：$\int_{-\pi}^{\pi}\cos nx\cos mx\,dx=\begin{cases}0, & n\neq m\\ \pi, & n=m\end{cases}$；$\int_{-\pi}^{\pi}\sin nx\sin mx\,dx$ 完全同理

    * 交叉情形恒为零：$\int_{-\pi}^{\pi}\cos nx\sin mx\,dx=0$（$m,n$ 任意）；常数 $1$ 与任何 $\cos nx$、$\sin nx$ 的乘积积分也为零

    * 唯一例外：$\int_{-\pi}^{\pi}1\cdot1\,dx=2\pi$，推系数时它自己会冒出来

## 傅里叶系数与傅里叶级数

* 欧拉–傅里叶公式

    * 推导手法与幂级数求系数一模一样：不是先证明能展开，而是先假设展开成立，再把系数的表达式挤出来——幂级数那边先假设可逐项求导，这边先假设可逐项积分

    * 两边同乘 $\cos mx$ 在 $[-\pi,\pi]$ 上积分，正交性把右边筛得只剩一项：$m=0$ 时 $a_0\pi=\int_{-\pi}^{\pi}f(x)\,dx$；$m>0$ 时只有 $n=m$ 的项存活，$a_m\pi=\int_{-\pi}^{\pi}f(x)\cos mx\,dx$；同乘 $\sin mx$ 同理得 $b_n$

    * $$a_n=\frac{1}{\pi}\int_{-\pi}^{\pi}f(x)\cos nx\,dx\quad(n=0,1,2,\cdots),\qquad b_n=\frac{1}{\pi}\int_{-\pi}^{\pi}f(x)\sin nx\,dx\quad(n=1,2,\cdots)$$

    * > **定义**：这两个系数公式称为欧拉–傅里叶公式，由它们算出的 $a_n$、$b_n$ 称为 $f$ 的傅里叶系数

* 常数项写成 $\dfrac{a_0}{2}$ 的原因

    * 把常数项看成 $n=0$ 的余弦项（$\cos 0x=1$）的系数，$a_0$ 的公式就跟一般的 $a_n$ 的公式完全一致；直接写 $a_0$ 会破坏这套统一形式

* 傅里叶级数的定义

    * > **定义**：设 $f$ 在 $[-\pi,\pi]$ 上可积或绝对可积，按欧拉–傅里叶公式算出系数，对应的三角级数称为 $f$ 的傅里叶级数，记作 $f(x)\sim\dfrac{a_0}{2}+\sum_{n=1}^{\infty}\left(a_n\cos nx+b_n\sin nx\right)$

    * "$\sim$"只表示：这个三角级数是按 $f$ 的傅里叶系数写出来的；级数收不收敛、收敛了是否等于 $f(x)$，都不作任何承诺

* 三角级数不等于傅里叶级数

    * 收敛的三角级数未必是傅里叶级数：$\sum_{n=2}^{\infty}\dfrac{\sin nx}{\ln n}$ 因 $\dfrac1{\ln n}$ 单调趋于零而在 $(-\infty,+\infty)$ 点点收敛，但它的系数不满足必要条件，不是任何可积或绝对可积函数的傅里叶级数

    * 判定标准：看系数是不是来自某个可积或绝对可积函数的欧拉–傅里叶公式

    * > **易错点**：不要以为三角级数只要收敛就是傅里叶级数——"三角级数"与"傅里叶级数"是两个概念，前者收敛并有和函数，也不说明它是和函数的傅里叶级数

## 方波的展开

* 方波函数

    * > **定义**：$f(x)=1$（$-\pi\le x\le0$）、$f(x)=0$（$0<x<\pi$），图像上两个接不上的点要圈掉——$(0,0)$ 处因 $f(0)=1$，$(\pi,0)$ 处因周期延拓后 $x=\pi$ 应取值 $1$；这个函数在电学里叫方波

    * $x=\pi$ 处的函数值没给——给了也没用：改动一个点的函数值不影响可积性，也不影响任何一个系数的计算

* 系数与展开式

    * $a_0=\dfrac1\pi\int_{-\pi}^{0}1\cdot dx=1$；$a_n=\dfrac1\pi\int_{-\pi}^{0}\cos nx\,dx=0$；$b_n=\dfrac{(-1)^n-1}{n\pi}$——$n$ 为偶数时为零，$n$ 为奇数时为 $-\dfrac{2}{n\pi}$

    * $$f(x)\sim\frac{1}{2}-\frac{2}{\pi}\left(\sin x+\frac{1}{3}\sin 3x+\frac{1}{5}\sin 5x+\cdots\right)$$

* 收敛行为

    * 除 $0$、$\pm\pi$ 这些不连续点外，级数在其余点上都收敛于函数值

    * 不连续点上它不是不收敛——它收敛，只是不收敛于 $f(x)$：$x=0$ 处函数值为 $1$ 而所有正弦项为零，级数收敛于 $\dfrac12$；$x=\pi$ 处按周期延拓函数值应为 $1$，级数照样收敛于 $\dfrac12$

* 正弦波叠加出方波

    * 把 $\sin x$、$\dfrac13\sin3x$、$\dfrac15\sin5x$、… 按这套系数叠加就叠出方波——"方波可以用正弦波来叠加"的数学表述正是这个展开

## 正弦级数与余弦级数

* 展开类型不能随意指定

    * $f$ 在整个区间上给定时，$a_n$、$b_n$ 全被积分公式锁死，无法指定"在这个区间上展开成正弦级数或余弦级数"

    * 唯一办法：缩小给定的区间——只给半个周期 $[0,\pi]$ 的函数值，靠延拓实现两类展开

* 奇延拓得到正弦级数

    * 把 $[0,\pi]$ 上的 $f$ 按反号镜像补成 $[-\pi,\pi]$ 上的奇函数，再以 $2\pi$ 为周期铺满数轴；$f$ 是奇函数、$\cos nx$ 是偶函数，乘积还是奇函数，对称区间上积分为零，故 $a_0=a_n=0$，展开只剩正弦项

    * 实际要算的只有 $b_n=\dfrac{2}{\pi}\displaystyle\int_0^{\pi}f(x)\sin nx\,dx$

* 偶延拓得到余弦级数

    * 把 $[0,\pi]$ 上的 $f$ 按原样镜像补成偶函数再周期延拓；$f$ 是偶函数、$\sin nx$ 是奇函数，乘积是奇函数，$b_n=0$

    * 实际要算 $a_0=\dfrac{2}{\pi}\displaystyle\int_0^{\pi}f(x)\,dx$ 与 $a_n=\dfrac{2}{\pi}\displaystyle\int_0^{\pi}f(x)\cos nx\,dx$

    * 公式里多出的因子 $2$ 是延拓带来的好处：对称区间上的积分折一半、乘二，只剩半个周期上的积分

| 目标 | 延拓方式 | 消失的系数 | 保留的系数 | 实际要算的公式 |
| :--- | :--- | :--- | :--- | :--- |
| 正弦级数 | 奇延拓 | $a_0,\ a_n$ | $b_n$ | $b_n=\dfrac{2}{\pi}\displaystyle\int_0^{\pi} f(x)\sin nx\,dx$ |
| 余弦级数 | 偶延拓 | $b_n$ | $a_0,\ a_n$ | $a_0=\dfrac{2}{\pi}\displaystyle\int_0^{\pi} f(x)\,dx$，$a_n=\dfrac{2}{\pi}\displaystyle\int_0^{\pi} f(x)\cos nx\,dx$ |

* 常数项的归属

    * $\dfrac12+\sin x+\dfrac13\sin3x+\cdots$ 看不到余弦字母，但对应的 $f(x)$ 不是奇函数——常数项是 $n=0$ 的余弦项（$\cos0=1$，常数本来就在余弦家族里）

    * 把 $f(x)$ 减掉 $\dfrac12$、图像往下平移一半，剩下的才是奇函数和纯正弦级数

    * > **易错点**：判断"纯正弦级数 $\Rightarrow$ 奇函数"时必须先把常数项挪走

* 实战：延拓这道工序可以不做

    * 区间给 $[0,\pi]$、要余弦级数就直接写 $a_0=\frac2\pi\int_0^\pi f\,dx$、$a_n=\frac2\pi\int_0^\pi f\cos nx\,dx$；要正弦级数就直接写 $b_n=\frac2\pi\int_0^\pi f\sin nx\,dx$

    * 延拓的价值是想明白道理：为什么一半系数是零、为什么积分只要从 $0$ 到 $\pi$、为什么前面多一个 $2$

## 同一函数的两种展开

* $f(x)=x$（$x\in[0,\pi]$）的余弦级数

    * 偶延拓得到连续的锯齿形周期函数（$[-\pi,\pi]$ 上是 $|x|$，拼成 V 字形再周期复制）；延拓出来的函数连续、性质好，级数在每一点都收敛于 $f(x)$

    * $a_0=\dfrac{2}{\pi}\int_0^{\pi}x\,dx=\pi$；分部积分（把 $x\cos nx$ 写成 $\dfrac{x}{n}\,d(\sin nx)$，微分号里要带 $n$）得 $a_n=\dfrac{2}{n^2\pi}\left[(-1)^n-1\right]$——$n$ 偶为零、$n$ 奇为 $-\dfrac{4}{n^2\pi}$

    * $$f(x)=\frac{\pi}{2}-\frac{4}{\pi}\left(\cos x+\frac{1}{3^2}\cos 3x+\frac{1}{5^2}\cos 5x+\cdots\right)$$

* $f(x)=x$ 的正弦级数

    * 奇延拓成周期 $2\pi$ 的奇函数（每段从 $-\pi$ 升到 $\pi$，拼接处跳变），每个周期点上挤着两个值要扣掉一个；$b_n=\dfrac{2(-1)^{n+1}}{n}$

    * $$f(x)=2\left(\sin x-\frac{1}{2}\sin 2x+\frac{1}{3}\sin 3x-\cdots\right)=2\sum_{n=1}^{\infty}\frac{(-1)^{n+1}}{n}\sin nx$$

    * 分部积分要点：$\sin nx\,dx=-\dfrac1n\,d(\cos nx)$，后一个积分 $\int_0^\pi\cos nx\,dx=0$，只剩边界项 $\pi(-1)^n$

* 同一段函数值，两个迥异的展开

    * 两个级数分别对应奇函数版本与偶函数版本的延拓；在 $[0,\pi)$ 上函数值完全一样，两个级数都收敛于 $f(x)=x$

    * 区别在区间外：余弦级数在 $[-\pi,0]$ 收敛到偶镜像那支，正弦级数收敛到奇函数那支——延拓方式决定级数在给定区间外的表现

## 任意周期的傅里叶展开

* 周期 $2T$ 的化归

    * 做变量替换 $\dfrac xT=\dfrac t\pi$：$x$ 以 $2T$ 为周期，则 $t=\dfrac{\pi x}{T}$ 以 $2\pi$ 为周期；令 $\varphi(t)=f\!\left(\dfrac{Tt}{\pi}\right)$，$\varphi$ 的傅里叶展开是现成的

    * 把 $t=\dfrac{\pi x}{T}$ 换回去：$f(x)\sim\dfrac{a_0}{2}+\displaystyle\sum_{n=1}^{\infty}\left(a_n\cos\dfrac{n\pi x}{T}+b_n\sin\dfrac{n\pi x}{T}\right)$

* 系数公式

    * 换元时积分限从 $[-\pi,\pi]$ 变 $[-T,T]$、$dt$ 变 $dx$ 要乘 $\dfrac{\pi}{T}$，与前面的 $\dfrac1\pi$ 相消成 $\dfrac1T$：

    * $$a_n=\frac{1}{T}\int_{-T}^{T}f(x)\cos\frac{n\pi x}{T}\,dx,\qquad b_n=\frac{1}{T}\int_{-T}^{T}f(x)\sin\frac{n\pi x}{T}\,dx$$

    * 记法：把 $2\pi$ 情形里的 $\pi$ 全部换成 $T$、积分区间换成 $[-T,T]$

* 例子：以区间长度 $2$ 为周期展开

    * $f(x)=0$（$-1<x\le0$）、$f(x)=x^2$（$0<x\le1$），以区间长度 $2$ 作周期延拓，$2T=2$、$T=1$；级数里出现的是 $\cos n\pi x$、$\sin n\pi x$，不能惯性写成 $\cos nx$

    * $a_0=\displaystyle\int_0^{1}x^2\,dx=\dfrac13$（$[-1,0]$ 段恒为零没有贡献）；$a_n=\dfrac{2(-1)^n}{n^2\pi^2}$（$x^2$ 配 $\cos$ 要做两次分部积分）；$b_n=\dfrac{(-1)^{n+1}}{n\pi}+\dfrac{2\left[(-1)^n-1\right]}{n^3\pi^3}$，$n$ 为偶数时第二项自动消失

    * $$f(x)\sim\frac16+\frac{2}{\pi^2}\sum_{n=1}^{\infty}\frac{(-1)^n}{n^2}\cos n\pi x+\frac1\pi\sum_{n=1}^{\infty}\left[\frac{(-1)^{n+1}}{n}+\frac{2\left[(-1)^n-1\right]}{n^3\pi^2}\right]\sin n\pi x$$

## 收敛问题与Dirichlet积分

* 问题定位

    * 函数项级数先看点态收敛：固定 $x$，一切问题压在部分和 $S_M(x)=\dfrac{a_0}{2}+\displaystyle\sum_{n=1}^{M}\left(a_n\cos nx+b_n\sin nx\right)$ 身上——$M\to\infty$ 时有没有极限、极限是不是 $f(x)$

* 部分和的化归

    * 系数代入、合并积分：$S_M(x)=\dfrac1\pi\displaystyle\int_{-\pi}^{\pi}f(t)\left[\dfrac12+\sum_{n=1}^{M}\cos n(t-x)\right]dt$，中括号由积化和差的逆用 $\cos nt\cos nx+\sin nt\sin nx=\cos n(t-x)$ 凑出

    * 求和公式 $\dfrac12+\displaystyle\sum_{n=1}^{M}\cos n\theta=\dfrac{\sin\frac{2M+1}{2}\theta}{2\sin\frac{\theta}{2}}$：把分母乘过去，$2\sin\frac\theta2\cos n\theta$ 用积化和差拆成两个正弦相减、相邻错位相消，最后剩一个 $\sin\frac{2M+1}{2}\theta$

    * 令 $u=t-x$ 换元（$f(t)$ 变 $f(x+u)$，积分区间平移后仍是长度 $2\pi$ 的区间）；再把积分拆成 $[-\pi,0]$ 与 $[0,\pi]$ 两段——振荡因子 $\dfrac{\sin\frac{2M+1}2u}{2\sin\frac u2}$ 是偶函数（分子分母都是奇函数），负半段换向合并后 $f(x+u)$ 变成 $f(x-u)$

* Dirichlet积分（Dirichlet integral）

    * > **定义**：$S_M(x)=\dfrac1\pi\displaystyle\int_0^{\pi}\left[f(x+u)+f(x-u)\right]\dfrac{\sin\frac{2M+1}{2}u}{2\sin\frac{u}{2}}\,du$ 称为 Dirichlet 积分；换元后、合并正负半段之前的形式同样这么叫

    * "级数在 $x$ 点是否收敛、收敛于什么函数"完全翻译成一句话：看这个积分随 $M\to\infty$ 收敛不收敛、收敛到什么值

* 把收敛目标 $\sigma(x)$ 塞进积分

    * 关键恒等式：$\displaystyle\int_0^{\pi}\dfrac{\sin\frac{2M+1}{2}u}{2\sin\frac u2}\,du=\dfrac\pi2$ 恒成立、与 $M$ 无关——还原成 $\dfrac12+\sum\cos nu$ 逐项积分，每个 $\cos$ 项的积分都是零，只剩 $\dfrac12$ 积出 $\dfrac\pi2$

    * 于是任何函数值都可原封不动塞进积分：$\sigma(x)=\dfrac1\pi\displaystyle\int_0^{\pi}2\sigma(x)\dfrac{\sin\frac{2M+1}{2}u}{2\sin\frac u2}\,du$（对积分变量 $u$ 来说 $\sigma(x)$ 只是乘进去的常数）

    * 两积分同限同系数，减法在积分号里进行，记 $\varphi(\sigma,u,x)=f(x+u)+f(x-u)-2\sigma(x)$：

    * $$S_M(x)-\sigma(x)=\frac{1}{\pi}\int_0^{\pi}\varphi(\sigma,u,x)\,\frac{\sin\frac{2M+1}{2}u}{2\sin\frac{u}{2}}\,du$$

    * > **注意**：$S_M(x)\to\sigma(x)$（$M\to\infty$）当且仅当上式右端的积分趋于零

* 化归路线总览

    ```text
    Fourier级数在x点收敛吗？收敛到谁？
          │ 点态收敛 = 部分和序列 S_M(x) 收敛
          ▼
    系数代入、合并积分、积化和差、换元u、合并正负半段
          ▼
    Dirichlet积分：S_M(x)=(1/π)∫[f(x+u)+f(x−u)]·sin((2M+1)u/2)/(2sin(u/2)) du
          │ 关键积分恒等于 π/2，把预定目标 σ(x) 塞进积分号
          ▼
    S_M(x)−σ(x)=(1/π)∫φ·sin((2M+1)u/2)/(2sin(u/2)) du，它趋于 0 吗？
          │ 核心工具：黎曼引理（高频振荡因子 × 可积函数 → 0）
          ├─ 推论一（局部性原理）：积分区间可缩到 [0,δ]，只看 f 在 [x−δ,x+δ] 上的值
          └─ 推论二：分母 2sin(u/2) 可以换成 u
    ```

## 黎曼引理及其推论

* 黎曼引理（Riemann lemma）

    * 内容：设 $\varphi$ 在 $[a,b]$ 上可积或绝对可积，则 $\displaystyle\lim_{p\to+\infty}\int_a^b\varphi(x)\sin px\,dx=0$；把 $\sin px$ 换成 $\cos px$ 结论完全一样，证其中一个即可

    * 直观：$\varphi$ 可积，$\sin px$ 在 $p$ 很大时高频正负振荡，正负面积不断互相抵消，积出来的总量趋于零

    * 黎曼可积情形的证明：对任意 $\varepsilon$ 取划分使振幅和 $\sum\omega_i\Delta x_i<\dfrac\varepsilon2$（$\omega_i$ 为各小区间上振幅）；逐段加减下确界 $m_i$ 拆成两块——第一块被 $|\varphi-m_i|\le\omega_i$、$|\sin px|\le1$ 压住不超过 $\dfrac\varepsilon2$；第二块每段积分至多 $\dfrac2p$，合计不超过 $\dfrac2p\sum|m_i|$，取 $P=\dfrac4\varepsilon\sum|m_i|$ 即可

    * $\varepsilon$-$P$ 顺序：要找大 $P$，必须先有划分；有了划分才定得下各 $m_i$；$m_i$ 定下并固定之后才能取 $P$

    * 绝对可积情形的证明：设 $b$ 为瑕点，绝对可积保证存在 $\delta$ 使 $\int_{b-\delta}^{b}|\varphi|\,dx<\dfrac\varepsilon2$；主体 $[a,b-\delta]$ 已黎曼可积，按上一情形存在大 $p$ 使该段积分小于 $\dfrac\varepsilon2$；瑕点段用 $|\sin px|\le1$ 压回绝对积分

    * 引理不只是为傅里叶级数的收敛判别准备的，本身在整个分析中都有重要应用

* 局部性原理

    * 内容：$f$ 可积或绝对可积时，其傅里叶级数在 $x$ 点是否收敛（连同收敛的话收敛于什么值），只与 $f$ 在 $[x-\delta,\,x+\delta]$ 上的性质有关；$\delta$ 是任意小的正数，$0.1$、$0.001$ 都可以

    * 证明要点：$[\delta,\pi]$ 上分母 $2\sin\frac u2$ 无零点，$\dfrac{f(x+u)+f(x-u)}{2\sin\frac u2}$ 可积或绝对可积，黎曼引理（取 $p=\dfrac{2M+1}{2}\to+\infty$）让这一段积分的极限为零；段内再减常数 $2\sigma(x)$ 不影响结论

    * $u$ 从 $0$ 走到 $\delta$ 时 $x+u$ 从 $x$ 走到 $x+\delta$、$x-u$ 从 $x$ 走到 $x-\delta$，所以极限只牵涉邻近值；$[x-\delta,x+\delta]$ 之外的函数值对收敛性不起任何作用

* 分母可以换成 $u$

    * 内容：$\varphi$ 在 $[0,\delta]$ 上可积或绝对可积，则 $\displaystyle\lim_{M\to\infty}\int_0^{\delta}\varphi(u)\dfrac{\sin\frac{2M+1}{2}u}{2\sin\frac u2}\,du=\displaystyle\lim_{M\to\infty}\int_0^{\delta}\varphi(u)\dfrac{\sin\frac{2M+1}{2}u}{u}\,du$

    * 依据：两分母之差 $\dfrac{1}{2\sin\frac u2}-\dfrac1u$ 通分后，分子 $u-2\sin\frac u2$ 从三次方项开始（等价于 $Au^3$）、分母 $2u\sin\frac u2$ 是两阶无穷小，商 $\sim Au\to0$——差函数补上一个函数值后在零点连续，乘可积的 $\varphi$ 仍可积，黎曼引理给出差的极限为零

    * > **易错点**：$2\sin\frac u2$ 与 $u$ 只是等价无穷小，积分号里不能见"等价"就替换——这里能换是因为"差的极限为零"被黎曼引理证出来，不是默认约分

## 收敛值的确定与收敛判别法

* 收敛值的必要条件

    * 设 $f$ 在 $x$ 点连续或为第一类不连续点（左、右极限都存在；连续点极限就等于 $f(x)$）；要考察 $\varphi_\sigma(u)=\dfrac{f(x+u)+f(x-u)-2\sigma(x)}{u}$ 在 $[0,\delta]$ 上的可积性，毛病出在分母 $u$ 的零点上

    * 若分子减 $2\sigma(x)$ 在 $u\to0$ 时不等于零，整个函数就是 $\dfrac1u$ 乘一个趋向非零常数的量，在 $[0,\delta]$ 上肯定不可积，黎曼引理条件根本不成立

    * 由此，要用黎曼引理处理这个积分，必要条件是 $\sigma(x)=\dfrac{f(x+0)+f(x-0)}{2}$：连续点上级数只能收敛于 $f(x)$ 本身，第一类间断点上只能收敛于左右极限的平均

    * 这个取法只是必要条件：取定之后并不保证 $\varphi_\sigma$ 就可积或绝对可积

* Dini引理（Dini lemma）

    * 内容：$g(u)$ 在 $[0,\delta]$ 上单调，则 $\displaystyle\lim_{p\to\infty}\int_0^{\delta}\frac{g(u)-g(0+)}{u}\sin pu\,du=0$；条件只讲单调、没讲可积或绝对可积，这正是它厉害的地方

    * 证明骨架：在 $\delta_1$ 处切开——近段 $[0,\delta_1]$ 用积分第二中值定理（单调因子放 $G$ 位，$G(0+)=0$ 使一项消失），剩下 $[g(\delta_1)-g(0+)]\displaystyle\int_\xi^{\delta_1}\frac{\sin pu}{u}du$；换元 $t=pu$ 后用 Dirichlet 积分 $\int_0^{+\infty}\frac{\sin x}{x}dx$ 收敛的 Cauchy 准则（存在 $K$ 压住任意部分区间上的积分），这一项对一切 $p$ 一致地小；远段 $[\delta_1,\delta]$ 分母远离零，黎曼引理放掉

    * 骨架值得记住：近点用单调性 + 第二中值定理硬啃，远点用黎曼引理放掉——后面两个判别法的证明都是这个套路的翻版

* 分段单调

    * > **定义**：$f$ 在 $[a,b]$ 上每点有定义，若存在有限划分 $a=x_0<x_1<\cdots<x_n=b$ 使 $f$ 在每个开区间 $(x_{i-1},x_i)$ 上单调（增减随意），称 $f$ 在 $[a,b]$ 上分段单调

    * 用在判别法里只要求 $x$ 的小邻域分段单调：$x$ 是分点则左右各一小段单调，不是分点则整个邻域单调——条件很宽厚

* 收敛判别法

    * 设 $f$ 在 $[-\pi,\pi]$ 上可积或绝对可积，若在点 $x$ 处下列两条件之一成立，则傅里叶级数在 $x$ 点收敛于 $\dfrac{f(x+0)+f(x-0)}{2}$

    * 狄利克雷–若当判别法（Dirichlet 判别法）：$f$ 在 $x$ 点的某个邻域内分段单调且是有界函数；证明即用 Dini 引理——$f(x+u)$、$f(x-u)$ 关于 $u$ 在 $[0,\delta]$ 上单调，两个积分的极限都为零

    * 迪尼–利普希茨判别法（Dini–Lipschitz 判别法）：$f$ 在 $x$ 处满足指数 $\alpha\in(0,1]$ 的 Hölder 条件

* Hölder条件（Hölder condition）

    * > **定义**：$|f(x+u)-f(x+)|\le Lu^{\alpha}$ 且 $|f(x-u)-f(x-)|\le Lu^{\alpha}$ 对一切 $0<u<\delta$ 成立（$L$ 为常数，左右两侧各一个估计）；$\alpha=1$ 时即 Lipschitz 条件

    * 证明思路：$\left|\dfrac{f(x\pm u)-f(x\pm)}{u}\right|\le L\,u^{\alpha-1}=\dfrac{L}{u^{1-\alpha}}$，幂指数 $1-\alpha<1$ 的奇性积分收敛，被积函数至少可积（绝对可积），黎曼引理直接开火

* 条件强弱链

    * 指数 $\alpha$ 越大 Hölder 条件越强；函数在一点可导比满足 Lipschitz 条件更强——可导推出该点附近 Lipschitz（用微分中值定理），反例 $|x|$ 在零点不可导却处处 Lipschitz

    * 从强到弱：可导 $\Rightarrow$ Lipschitz（$\alpha=1$ 的 Hölder）$\Rightarrow$ 较小指数的 Hölder；验证强条件就自动满足弱条件

* 拟单侧导数推论

    * 两个单侧导数 $f'_+(x)$、$f'_-(x)$ 都存在：级数在 $x$ 点收敛于 $f(x)$——单侧导数存在蕴含该点连续（左右极限都得对上 $f(x)$）

    * 更宽一步，两个拟单侧导数 $\displaystyle\lim_{h\to0^\pm}\frac{f(x+h)-f(x\pm)}{h}$ 存在：$f$ 在 $x$ 点可以不连续，级数收敛于 $\dfrac{f(x+0)+f(x-0)}{2}$

    * > **易错点**：拟单侧导数不是严格的单侧导数——差商分子用的是单侧极限 $f(x\pm)$ 而不是函数值 $f(x)$；正因放宽这一点，间断点上也有"导数"可言，收敛值随之从 $f(x)$ 变成左右极限的平均

## 和函数的确定与级数和

* 方法链

    * 判别法定出和函数 → 特殊点代入 → 掉出级数和等式 → 拿去算积不动的积分与零点的和；"圈掉、补点"的操作就是把 $f$ 的图像换成和函数的图像

* 方波的和函数

    * 开区间内部 $f$ 是常数（单调、可导都成立），两个判别法随便用，级数收敛于 $f$ 本身；$x=0,\pm\pi$ 处不连续但分段单调仍成立、两个单侧拟导数都存在（等于零），级数收敛于左右平均 $\dfrac{1+0}{2}=\dfrac12$

    * 和函数与 $f$ 几乎处处重合，唯独 $x=0,\pm\pi$ 及周期重复出去的 $\pm2\pi,\pm3\pi,\dots$ 处把原函数值圈掉、各补一个高度 $\dfrac12$ 的孤立点

    * 代 $x=\dfrac\pi2$（落在 $(0,\pi)$、和函数值为 $0$；$\sin\frac\pi2=1$、$\sin\frac{3\pi}2=-1$ 正负交替）：$1-\dfrac13+\dfrac15-\dfrac17+\cdots=\dfrac\pi4$

* $f(x)=x$ 余弦级数：等号写到闭区间

    * 偶延拓是 $|x|$、处处连续：内部可导单调，$x=0$ 连续且左右导数存在，$x=\pm\pi$ 单侧导数存在、分段单调也成立——没有间断点捣乱，等式在整个闭区间 $[0,\pi]$ 上处处成立

    * 代 $x=0$（右边每项 $\cos$ 全为 $1$）：$1+\dfrac1{3^2}+\dfrac1{5^2}+\dfrac1{7^2}+\cdots=\dfrac{\pi^2}{8}$

    * > **易错点**：括号里的分母是 $3^2,\ 5^2,\ \ldots$，是平方——这一处最容易看漏、抄错

* $f(x)=x$ 正弦级数：端点收敛到零

    * 开区间 $(-\pi,\pi)$ 内延拓函数就是 $y=x$，单调可导，级数收敛于 $x$；$x=\pm\pi$ 处不连续（右极限 $-\pi$、左极限 $\pi$），分段单调仍成立、单侧拟导数也存在，级数收敛于平均 $0$

    * 和函数图像：$(-\pi,\pi)$ 上是直线 $y=x$、往两边周期重复成斜锯齿，$\pm\pi,\pm3\pi,\dots$ 处端点圈掉、各补一个高度 $0$ 的孤立点——这张图已经是级数和函数的图像，不再是 $f$ 自己的图像

* 周期为 $2$ 的展开：一次代入收三个级数和

    * 逐段检查：$(0,1)$ 上收敛于 $x^2$；$(-1,0)$ 上是常数 $0$，收敛于 $0$；$x=0$ 处连续（左右极限与函数值都是 $0$），收敛于 $0$；$x=\pm1$ 不连续，但分段单调成立、拟导数存在（$x=1$ 处右拟导数为 $0$），收敛于 $\dfrac{0+1}{2}=\dfrac12$

    * 代 $x=1$（$\sin n\pi=0$ 正弦部分整个消失，$\cos n\pi=(-1)^n$ 与系数里的 $(-1)^n$ 相乘变 $1$）：$\dfrac12=\dfrac16+\dfrac{2}{\pi^2}\sum_{n=1}^{\infty}\frac1{n^2}$，移项得 $\displaystyle\sum_{n=1}^{\infty}\frac1{n^2}=\frac{\pi^2}{6}$

    * 代 $x=0$：$0=\dfrac16+\dfrac{2}{\pi^2}\sum_{n=1}^{\infty}\frac{(-1)^n}{n^2}$，改写成正项开头得 $\displaystyle\sum_{n=1}^{\infty}\frac{(-1)^{n+1}}{n^2}=\frac{\pi^2}{12}$

    * 前两式相加除二（偶数项正负抵消、奇数项翻倍）：$\displaystyle\sum_{k=0}^{\infty}\frac1{(2k+1)^2}=\frac{\pi^2}{8}$——与余弦级数那条路走到同一个和，相互印证

| 级数 | 和 |
| :--- | :--- |
| $\displaystyle\sum_{n=1}^{\infty}\frac{1}{n^2}$ | $\dfrac{\pi^2}{6}$ |
| $\displaystyle\sum_{n=1}^{\infty}\frac{(-1)^{n+1}}{n^2}$ | $\dfrac{\pi^2}{12}$ |
| $\displaystyle\sum_{k=0}^{\infty}\frac{1}{(2k+1)^2}$ | $\dfrac{\pi^2}{8}$ |

## 级数和的应用

* 一个"积不动"的积分

    * $\displaystyle\int_0^1\frac{\ln(1-x)}{x}\,dx$：$x=0$ 那头被积函数极限存在（等于 $-1$），根本不是反常积分，但要真下手去积一时做不出来

    * 展开 $\ln(1-x)=-\sum_{n=1}^{\infty}\dfrac{x^n}{n}$（没有 $(-1)^n$：减号拎在最前面，系数全是负的），除以 $x$ 逐项积分，每项凑成 $\dfrac1{n^2}$

    * 结果 $=-\displaystyle\sum_{n=1}^{\infty}\frac1{n^2}=-\frac{\pi^2}{6}$

    * > **易错点**：负号别丢——$\ln(1-x)$ 展开的系数全是负的，负号要一路拎到底，丢了答案整个错

* $\cos x$ 零点的倒数平方和

    * 零点全体 $x=\pm\dfrac{(2k-1)\pi}{2}$（$k=1,2,3,\dots$）；正零点组平方取倒数求和 $=\dfrac{4}{\pi^2}\cdot\dfrac{\pi^2}{8}=\dfrac12$

    * 负零点组平方后与正零点组完全一样，全体求和 $=2\cdot\dfrac{4}{\pi^2}\cdot\dfrac{\pi^2}{8}=1$——$\cos x$ 全体零点的倒数平方和恰好等于 $1$

## 系数趋于零与逐项积分定理

* 傅里叶系数趋于零

    * 内容：$f$ 在 $[-\pi,\pi]$ 上可积或绝对可积，则 $\displaystyle\lim_{n\to\infty}a_n=0$，$\displaystyle\lim_{n\to\infty}b_n=0$

    * 证明即把黎曼引理往上套（振荡频率取 $p=n$），不需要单独证

* 逐项积分定理的叙述

    * 条件只要 $f$ 在 $[-\pi,\pi]$ 上可积或绝对可积——级数收不收敛、收敛了收不收敛于 $f$，一概不要求

    * 对任意 $c,x\in[-\pi,\pi]$：

    * $$\int_c^x f(t)\,dt=\frac{a_0}{2}(x-c)+\sum_{n=1}^{\infty}\int_c^x\left(a_n\cos nt+b_n\sin nt\right)dt$$

    * 亮点：原来的"$\sim$"换成等号——不管原级数收敛与否，右边因积分上限是 $x$ 的这个级数一定收敛于 $\int_c^x f(t)dt$

    * 证明范围：只对 $f$ 在 $[-\pi,\pi]$ 上只有有限个第一类不连续点（左右极限都存在的间断点）的情形证明；第二类间断点（左、右极限至少一个不存在）的情形未覆盖

* 积出来不再是傅里叶级数

    * $\int_c^x\cos nt\,dt=\dfrac{\sin nx-\sin nc}{n}$ 出 $\sin nx$、$\int_c^x\sin nt\,dt=-\dfrac{\cos nx-\cos nc}{n}$ 出 $\cos nx$，这部分还是三角级数；但 $\dfrac{a_0}{2}(x-c)$ 是 $x$ 的一次项，三角级数里没有这种东西——积完之后严格说已不是傅里叶级数的形状

    * 为证题方便把一次项移到左边：$\displaystyle\int_c^x f(t)dt-\frac{a_0}{2}(x-c)=\sum_{n=1}^{\infty}\int_c^x\left(a_n\cos nt+b_n\sin nt\right)dt$

* 证明的关键构造：辅助函数

    * 令 $F(x)=\displaystyle\int_c^x f(t)dt-\frac{a_0}{2}x$：被积函数可积则变上限积分连续（牛顿–莱布尼茨公式那一套），故 $F$ 连续

    * $F$ 以 $2\pi$ 为周期：$F(x+2\pi)-F(x)=\displaystyle\int_x^{x+2\pi}f(t)dt-\frac{a_0}{2}\cdot2\pi=0$——周期函数在任一长度为一个周期的区间上积分都等于 $a_0\pi$

    * $f$ 的连续点处 $F'(x)=f(x)-\dfrac{a_0}{2}$；$f$ 的第一类间断点处 $F$ 不可导但单侧导数存在（把 $F$ 在该点的值按右极限规定、右连续化后）：$F'_+(x)=f(x+0)-\dfrac{a_0}{2}$，$F'_-(x)=f(x-0)-\dfrac{a_0}{2}$

    * 于是 $F$ 连续且逐段光滑——凭收敛定理那两组条件撑着，$F$ 的傅里叶展开敢写等号且收敛于 $F$ 本身：$F(x)=\dfrac{A_0}{2}+\sum_{n=1}^{\infty}\left(A_n\cos nx+B_n\sin nx\right)$

* 系数计算与还原

    * 把 $\cos nx$ 收进微分号（$\cos nx\,dx=\frac1n\,d(\sin nx)$）：边界项 $[F(x)\sin nx]_{-\pi}^{\pi}$ 因 $\sin(\pm n\pi)=0$ 消失；$F'=f-\frac{a_0}2$ 中常数部分乘 $\sin nx$ 的积分也为零（$\cos n\pi=\cos(-n\pi)$）——得 $A_n=-\dfrac{b_n}{n}$，同理 $B_n=\dfrac{a_n}{n}$；$A_0$ 不需要算

    * 令 $x=c$ 得 $0=\dfrac{A_0}{2}+\sum_{n=1}^{\infty}\left(-\frac{b_n}{n}\cos nc+\frac{a_n}{n}\sin nc\right)$，与展开式相减消掉常数项；再用牛顿–莱布尼茨公式把 $\frac1n(\sin nx-\sin nc)$、$-\frac1n(\cos nx-\cos nc)$ 还原成 $\int_c^x\cos nt\,dt$、$\int_c^x\sin nt\,dt$，逐项积分公式到手

* 逐项积分白送的两个推论

    * 级数 $\displaystyle\sum_{n=1}^{\infty}\frac{b_n}{n}$ 收敛：取 $c=0$ 得 $\sum\frac{b_n}{n}=\frac{A_0}{2}$——$A_0$ 没算过、不知道是多少，但它是 $F$ 的傅里叶级数常数项，是一个确定的数

    * $\displaystyle\sum_{n=2}^{\infty}\frac{\sin nx}{\ln n}$ 点点收敛（$\frac1{\ln n}$ 单调下降趋于零、$\sum\sin nx$ 部分和有界，Dirichlet 判别法一套就收敛），但不是任何函数的傅里叶级数：若 $b_n$ 来自傅里叶系数，$\sum\frac{b_n}{n}$ 必须收敛，而 $\sum_{n=2}^{\infty}\frac{1}{n\ln n}$ 发散

    * > **易错点**：幂级数"点点收敛就必然是和函数的幂级数"，三角级数没有这种好事——可以处处收敛、有和函数 $S(x)$，却不是任何可积或绝对可积函数的傅里叶级数

## 逐项求导定理

* 定理的条件与结论

    * 傅里叶级数逐项求导的要求很强，一般的傅里叶级数不能逐项求导，必须加条件：① $f$ 在闭区间 $[-\pi,\pi]$ 上连续（两头都连续）；② $f(-\pi)=f(\pi)$；③ 除有限个点外 $f$ 可导，且 $f'(x)$ 在 $[-\pi,\pi]$ 上可积或绝对可积

    * 结论：$f'(x)\sim\displaystyle\sum_{n=1}^{\infty}\left(-na_n\sin nx+nb_n\cos nx\right)$——导函数的傅里叶级数可以从 $f$ 的傅里叶级数逐项求导得到

    * 这里只有波浪号、绝对没有等号：得到的是 $f'$ 的傅里叶级数，并没说它收敛于 $f'$

* "两头相等"为什么非加不可

    * $f'$ 的常数项系数按定义 $a'_0=\dfrac1\pi\displaystyle\int_{-\pi}^{\pi}f'(x)\,dx=\dfrac{f(\pi)-f(-\pi)}{\pi}$；而逐项求导不产生常数项（$\frac{a_0}2$ 这一项的导数是零），所以必须 $a'_0=0$，即 $f(\pi)=f(-\pi)$

    * 不能靠把 $f(\pi)$ 的值改成与 $f(-\pi)$ 一样来凑：条件里有闭区间连续，一改连续性条件就不成立

* 系数关系与证明

    * $a'_n=\dfrac1\pi\displaystyle\int_{-\pi}^{\pi}f'(x)\cos nx\,dx=\dfrac1\pi\int_{-\pi}^{\pi}\cos nx\,df(x)$，分部积分：边界项 $\dfrac1\pi[f(x)\cos nx]_{-\pi}^{\pi}$ 因 $\cos$ 的偶性与 $f(\pi)=f(-\pi)$ 相消为零；$d(\cos nx)=-n\sin nx\,dx$ 变号带出 $n$

    * 结果 $a'_n=nb_n$，同理 $b'_n=-na_n$——恰好是对 $a_n\cos nx+b_n\sin nx$ 逐项求导的结果

* 与函数项级数逐项求导的对比

    * 函数项级数逐项求导的条件加在"求导以后"那边（导数级数的行为）；这里同样要求 $f'$ 可积或绝对可积，且多出两端相等这一傅里叶特有的条件——一般函数不满足，故一般傅里叶级数不可逐项求导

## 平方可积与最佳平方逼近

* 平方可积强于绝对可积

    * 柯西–施瓦茨不等式：$\left(\int_a^b|f(x)|\,dx\right)^2\le\int_a^bf^2(x)\,dx\cdot\int_a^b1\,dx=(b-a)\int_a^bf^2\,dx$——平方可积推出绝对可积，反过来不行

    * 反例 $f(x)=\dfrac1{\sqrt x}$：$(0,\pi]$ 上 $\int_0^{\pi}\dfrac{dx}{\sqrt x}$ 收敛，平方后 $\int_0^{\pi}\dfrac{dx}{x}$ 发散

    * > **易错点**：平方可积 $\Rightarrow$ 绝对可积，反向不成立，强弱关系不要记反

    * 用词约定：这里说"可积"专指黎曼可积；黎曼可积的函数一定平方可积；非黎曼可积、积分只在反常意义下存在的要单独要求平方可积——前提合起来是"可积或平方可积"

* 内积空间

    * 内积不论怎么定义都要满足三条：线性性、对称性、正定性（$\langle x,x\rangle\ge0$，且等于零当且仅当 $x=0$）

    * 范数 $\|x\|=\sqrt{\langle x,x\rangle}$ 表示点（向量）到原点的距离，$\|x-y\|=\sqrt{\langle x-y,\ x-y\rangle}$ 是两个元素之间的距离

* 最佳平方逼近元素

    * > **定义**：固定 $x\in S$（$S$ 是定义了内积的线性空间，$T$ 是其 $n$ 维子空间，$\varphi_1,\varphi_2,\dots,\varphi_n$ 为 $T$ 的一组正交基），若 $x_T=c_1\varphi_1+c_2\varphi_2+\cdots+c_n\varphi_n\in T$ 使 $\Vert x-x_T\Vert=\min_{y \in T} \Vert x-y\Vert$，称 $x_T$ 为 $x$ 在 $T$ 中的最佳平方逼近元素

    * 一般不能在 $T$ 里找到恰好等于 $x$ 的元素（$S$ 是大空间、$T$ 只是子空间），退而求 $T$ 中跟 $x$ 尽量接近的元素；$\min$ 写成 $\inf$ 也行，但下面会证明下确界确实被取到

    * 几何直观：从 $x$ 垂直落到 $T$ 上的线最短，落点是垂足；误差向量 $x-x_T$ 要跟整个子空间 $T$ 垂直，到任何别的 $y$ 的斜线都更长

* 引理的三条结论

    * 对任意 $x\in S$，$x$ 在 $T$ 中的最佳平方逼近元素存在并且唯一

    * $x_T$ 是最佳平方逼近元素的充要条件：$x-x_T$ 垂直于 $T$，即 $\langle x-x_T,\ \varphi_k\rangle=0$ 对 $k=1,2,\dots,n$ 每个基元素成立（$n$ 个式子）

    * 误差公式：$\Vert x-x_T\Vert^2=\Vert x\Vert^2-\Vert x_T\Vert^2$，其中 $\Vert x_T\Vert^2=\sum_{k=1}^{n} c_k^2\, \Vert \varphi_k\Vert^2$

* 配方法证明

    * 取 $c_k=\dfrac{\langle x,\varphi_k\rangle}{\Vert \varphi_k\Vert^2}$，对任意 $y=\sum d_k\varphi_k$ 展开：$\Vert x-y\Vert^2=\left(\Vert x\Vert^2-\sum c_k^2\Vert \varphi_k\Vert^2\right)+\sum(c_k-d_k)^2\Vert \varphi_k\Vert^2$——前块与 $y$ 完全无关、后块永远非负，故每个 $d_k=c_k$ 时最小且最小点唯一；代回去最小值恰是误差公式——存在性、唯一性、误差公式一次到手

    * 垂直特征两个方向：正方向按基的正交性求和里只有 $j=k$ 项存活，$\langle x-x_T,\varphi_k\rangle=\langle x,\varphi_k\rangle-c_k\Vert \varphi_k\Vert^2=0$；反方向设 $x-y\perp T$，由 $\langle x,\varphi_k\rangle-d_k\Vert \varphi_k\Vert^2=0$ 解出 $d_k=c_k$，即垂直于 $T$ 的那个元素就是最佳逼近元

## 傅里叶部分和的最佳性

* 函数空间的内积

    * 取 $S$ 为 $[-\pi,\pi]$ 上可积或平方可积的函数全体：$\langle f,g\rangle=\displaystyle\int_{-\pi}^{\pi}f(x)g(x)\,dx$，$\Vert f\Vert=\sqrt{\langle f,f\rangle}=\sqrt{\int_{-\pi}^{\pi}f^2(x)\,dx}$

    * 反常积分情形出问题：两个绝对可积函数乘起来不一定可积（$\frac1{\sqrt x}$ 自乘发散，范数根本定义不出来），所以平方逼近必须以平方可积为前提；黎曼可积情形有界函数的乘积与平方仍可积，自动没问题

| 前提 | 内积与范数 | 说明 |
| :--- | :--- | :--- |
| 黎曼可积 | 有意义 | 有界函数的乘积与平方仍可积，平方可积自动满足 |
| 反常意义下绝对可积 | 不一定有意义 | $f=\dfrac1{\sqrt x}$：自身可积，平方发散 |
| 平方可积 | 有意义 | 平方逼近采用的前提 |

* 三角多项式子空间

    * $T$ 取 $N$ 阶三角多项式全体 $\left\{\dfrac{a_0}{2}+\sum_{k=1}^{N}(a_k\cos kx+b_k\sin kx)\right\}$；基为 $1,\cos x,\sin x,\dots,\cos Nx,\sin Nx$ 共 $2N+1$ 个且正交，故 $T$ 是 $S$ 的 $2N+1$ 维子空间

    * 基元素的范数平方：$\Vert 1\Vert^2=\int_{-\pi}^{\pi}1\,dx=2\pi$，$\Vert \cos kx\Vert^2=\Vert \sin kx\Vert^2=\pi$（$k=1,\dots,N$）——除了常数元是 $2\pi$，其余全是 $\pi$

* 部分和就是最佳平方逼近

    * 按引理系数公式老实算：$\dfrac{a_0}{2}=\dfrac{\langle f,1\rangle}{\Vert 1\Vert^2}=\dfrac1{2\pi}\int_{-\pi}^{\pi}f\,dx$，$a_k=\dfrac{\langle f,\cos kx\rangle}{\Vert \cos kx\Vert^2}=\dfrac1\pi\int_{-\pi}^{\pi}f\cos kx\,dx$，$b_k$ 同理——与欧拉–傅里叶公式完全一致

    * 定理内容：$f$ 在 $[-\pi,\pi]$ 上可积或平方可积时，$f$ 在 $N$ 阶三角多项式全体 $T$ 中的最佳平方逼近元素恰好是 $f$ 的傅里叶级数的部分和 $S_N(x)=\dfrac{a_0}{2}+\sum_{k=1}^{N}(a_k\cos kx+b_k\sin kx)$

    * 换句话说：在所有 $N$ 阶三角多项式里按误差范数平方 $\int_{-\pi}^{\pi}|f(x)-t(x)|^2\,dx$ 找跟 $f$ 靠得最近的，挑出来的一定是傅里叶部分和；$N$ 随意，取定哪个 $N$，相应部分和就是该阶里的最佳

## Bessel不等式与Parseval等式

* 部分和的范数平方与误差公式

    * 正交性下交叉内积全零，只剩每项自身：$\Vert T_N\Vert^2=\dfrac1\pi\displaystyle\int_{-\pi}^{\pi}T_N^2\,dx=\dfrac{a_0^2}{2}+\sum_{k=1}^{N}\left(a_k^2+b_k^2\right)$

    * 最佳逼近的误差即范数平方之差：$\Vert f-T_N\Vert^2=\dfrac1\pi\displaystyle\int_{-\pi}^{\pi}f^2\,dx-\dfrac{a_0^2}{2}-\sum_{k=1}^{N}\left(a_k^2+b_k^2\right)$

* Bessel不等式（Bessel inequality）

    * > **定义**：$\dfrac{a_0^2}{2}+\displaystyle\sum_{k=1}^{N}\left(a_k^2+b_k^2\right)\le\dfrac1\pi\int_{-\pi}^{\pi}f^2\,dx$ 称为 Bessel 不等式——左边是范数平方永远大于等于零，移项即得

    * $N$ 是任意的：左边部分和对所有 $N$ 被右端固定的数卡住，单调有界，令 $N\to\infty$ 也不越界

    * 意义：$\sum_{n=1}^{\infty}a_n^2$ 与 $\sum_{n=1}^{\infty}b_n^2$ 都收敛——比"可积或绝对可积"条件下的 $a_n\to0$、$b_n\to0$ 与 $\sum\frac{b_n}{n}$ 收敛强得多；级数收敛蕴含通项趋于零、反之不然——条件强了，结论就要更强

* Parseval等式

    * > **定义**：$f$ 在 $[-\pi,\pi]$ 上可积或平方可积，则 $\dfrac{a_0^2}{2}+\displaystyle\sum_{n=1}^{\infty}\left(a_n^2+b_n^2\right)=\dfrac1\pi\int_{-\pi}^{\pi}f^2\,dx$

    * 背景：平方可积函数空间 $S$ 里 $1,\cos x,\sin x,\dots,\cos nx,\sin nx,\dots$ 是一组极大正交基——不可能再往里添加一个与所有元素都正交的新元素；若 $f$ 与每个基元素都正交，立刻推出 $f$ 就是零

    * $a_n,b_n$ 恰是 $f$ 按这组基展开的坐标；正交基既已极大，展开的坐标平方和就把 $f$ 的"长度"用完——Parseval 等式正是这件事的定量表达；完整证明牵涉泛函分析，此处承认

* 平方收敛

    * > **定义**：$\displaystyle\lim_{N\to\infty}\int_{-\pi}^{\pi}\left[f(x)-T_N(x)\right]^2dx=0$，称 $T_N$ 平方收敛（均方收敛）于 $f(x)$

    * 由 Parseval 等式保证：误差公式右端随 $N\to\infty$ 趋于零

    * 这个收敛模式跟点态收敛、一致收敛都不一样：不是逐点函数值逼近，而是差函数的平方积分趋于零

## Weierstrass第二逼近定理与等周问题

* Weierstrass第二逼近定理

    * 内容：设 $f(x)$ 是以 $2\pi$ 为周期的连续函数，则存在三角多项式序列 $\varphi_N(x)=\dfrac{a_0}{2}+\sum_{k=1}^{N}(a_k\cos kx+b_k\sin kx)$ 一致收敛于 $f(x)$

    * 与 Weierstrass 第一逼近定理对应：那里是代数多项式序列一致逼近闭区间上的连续函数，这里是三角多项式；两个逼近定理只叙述不证明

* 等周问题

    * 内容：平面上所有的简单闭曲线中，若周长一定，则圆所围的面积最大——设周长 $L$、面积 $A$，周长为 $L$ 的圆半径 $\dfrac{L}{2\pi}$、面积 $\pi\left(\dfrac{L}{2\pi}\right)^2=\dfrac{L^2}{4\pi}$，要证 $A\le\dfrac{L^2}{4\pi}$，且等号成立当且仅当曲线是圆

* Wirtinger引理（Wirtinger lemma）

    * 内容：设 $f$ 在 $[-\pi,\pi]$ 上连续，且 $f(-\pi)=f(\pi)$、$\displaystyle\int_{-\pi}^{\pi}f(x)\,dx=0$、除有限个点外可导且不可导点有单侧导数、$f'$ 可积或平方可积，则 $\displaystyle\int_{-\pi}^{\pi}f^2\,dx\le\int_{-\pi}^{\pi}f'^2\,dx$，等号成立当且仅当 $f(x)=a\cos x+b\sin x$

    * 条件用途一：$\int f=0$ 即 $a_0=0$，Parseval 里 $\dfrac{a_0^2}{2}$ 这一项就没有了，且让 $f'$ 的展开从 $f$ 的级数逐项求导名正言顺（$f$ 的常数项求导为零，$a'_0=0$ 自动成立）——没有这个条件傅里叶级数不能逐项求导

    * 条件用途二："除有限点可导 + 不可导点单侧导数"正是 Dini–Lipschitz 判别法的条件，保证 $f$ 连续时级数收敛于 $f$ 本身（左右极限的平均就是 $f(x)$）

    * 证明：对 $f$ 与 $f'$ 分别用 Parseval 等式（$f'$ 的系数是 $na_n$ 与 $nb_n$，求导各多出一个 $n$），相减得 $\dfrac1\pi\displaystyle\int_{-\pi}^{\pi}\left(f'^2-f^2\right)dx=\sum_{n=1}^{\infty}(n^2-1)\left(a_n^2+b_n^2\right)\ge0$；等号要求 $n\ge2$ 的系数全为零，只剩 $a_1\cos x+b_1\sin x$

* 等周问题的证明主线

    * 参数化：以弧长 $s$ 为参数（由 $\left(\frac{ds}{dt}\right)^2=x'^2(t)+y'^2(t)$ 可解出 $s$ 与 $t$ 一一对应），$x=\varphi(s),y=\psi(s)$，$s\in[0,L]$，闭曲线给 $\varphi(0)=\varphi(L)$、$\psi(0)=\psi(L)$；再换元 $s=\dfrac{L}{2\pi}t+\dfrac{L}{2}$（$s=0$ 时 $t=-\pi$，$s=L$ 时 $t=\pi$）把参数区间移到 $[-\pi,\pi]$，端点条件变成 $\varphi(-\pi)=\varphi(\pi)$、$\psi(-\pi)=\psi(\pi)$——Wirtinger 引理第一个条件到手

    * 不妨设 $\displaystyle\int_{-\pi}^{\pi}\psi\,dt=0$：若积分为 $K\ne0$，把曲线平移 $\dfrac{K}{2\pi}$——长度与面积都不变，而新曲线的积分归零

    * 关键恒等式：由 $\dfrac{ds}{dt}=\dfrac{L}{2\pi}$ 得 $\varphi'^2(t)+\psi'^2(t)=\dfrac{L^2}{4\pi^2}$，两边积分出手里的积分等式 $\dfrac{L^2}{2\pi}=\displaystyle\int_{-\pi}^{\pi}\left[\varphi'^2(t)+\psi'^2(t)\right]dt$

    * 面积：格林公式（第二类曲线积分，取逆时针方向）$A=\oint_C x\,dy=\displaystyle\int_{-\pi}^{\pi}\varphi(t)\psi'(t)\,dt$

    * 组装：$\dfrac{L^2}{2\pi}-2A=\displaystyle\int_{-\pi}^{\pi}\left[\varphi'^2+\psi'^2-2\varphi\psi'\right]dt$，减去 $\int\varphi^2$ 加上 $\int(\varphi-\psi')^2$ 凑完全平方数值不变——$\dfrac{L^2}{2\pi}-2A=\displaystyle\int_{-\pi}^{\pi}\left[\varphi'^2-\varphi^2\right]dt+\int_{-\pi}^{\pi}\left(\varphi-\psi'\right)^2dt\ge0$：第二项是平方的积分非负，第一项因 $\varphi$ 满足 Wirtinger 引理全部条件而非负；移项即 $A\le\dfrac{L^2}{4\pi}$，圆围出的面积最大

    * 等号情形：要求 $\int_{-\pi}^{\pi}(\varphi-\psi')^2dt=0$ 即 $\psi'=\varphi$，且 Wirtinger 引理等号成立即 $\varphi(t)=a\cos t+b\sin t$；积分出 $\psi(t)=a\sin t-b\cos t+c$，验证 $x^2+(y-c)^2=a^2+b^2$，恰好是圆

    * 主线两块支撑：Parseval 等式把"函数平方的积分"变成"系数平方和"，Wirtinger 引理靠它比较 $f$ 与 $f'$ 才立起来；把几何量（周长、面积）翻译成参数函数的积分（弧长微分、格林公式），再靠一个完全平方把两者配上
