# 幂级数与Weierstrass逼近

## 幂级数与收敛半径

   * **幂级数（power series）**

       * > **定义**：形如 $\sum\limits_{n=0}^{\infty}a_n(x-x_0)^n=a_0+a_1(x-x_0)+a_2(x-x_0)^2+\cdots$ 的函数项级数，通项 $a_n(x-x_0)^n$ 是幂函数。

       * 部分和 $S_N(x)$ 是 $N-1$ 次多项式，幂级数可看成「无限次的多项式」。

       * 不失一般性取 $x_0=0$ 只讨论 $\sum a_nx^n$：一般情形把 $x$ 换成 $t-x_0$ 即化回，讨论出的每条性质照样平移成立。

   * **收敛半径（radius of convergence）**

       * > **定义**：记 $A=\limsup\limits_{n\to\infty}\sqrt[n]{|a_n|}$，规定 $R=\dfrac1A$（$0<A<+\infty$）；$A=0$ 时 $R=+\infty$；$A=+\infty$ 时 $R=0$。

       * $A=0$ 的含义：$\sqrt[n]{|a_n|}$ 再乘任何 $|x|$ 仍趋 $0$，级数对一切 $x$ 绝对收敛；$A=+\infty$：只要 $x\neq0$ 就发散，级数仅在 $x=0$ 一点收敛。

   * **柯西—阿达玛定理（Cauchy–Hadamard theorem）**

       * 当 $|x|<R$ 时 $\sum a_nx^n$ 收敛且**绝对收敛**；当 $|x|>R$ 时发散。

       * 「绝对」是白捡的：柯西判别法作用在 $\sqrt[n]{|a_nx^n|}$ 这个非负量上，判出的收敛天然就是绝对收敛。

       * 证明靠两件事：$x^n$ 开 $n$ 次根正好是 $|x|$，连极限都不用求；常数因子从上极限里提出来是等号。

       * 一般形式 $\sum a_n(x-x_0)^n$ 同话平移：$|x-x_0|<R$ 绝对收敛、$|x-x_0|>R$ 发散；端点 $x=x_0\pm R$ 定理管不着，要具体讨论。

   * **达朗贝尔公式（d'Alembert formula）**

       * 前提 $a_n\neq0$（后项除前项，系数为零就没法除）。若 $\lim\limits_{n\to\infty}\left|\dfrac{a_{n+1}}{a_n}\right|=A$，则收敛半径同样是 $R=\dfrac1A$。

       * 依据不等式链 $\liminf\limits_{n\to\infty}\left|\frac{a_{n+1}}{a_n}\right|\le\liminf\limits_{n\to\infty}\sqrt[n]{|a_n|}\le\limsup\limits_{n\to\infty}\sqrt[n]{|a_n|}\le\limsup\limits_{n\to\infty}\left|\frac{a_{n+1}}{a_n}\right|$：比值的上、下极限都是 $A$，两头一夹，中间的 $\limsup\sqrt[n]{|a_n|}$ 只能也等于 $A$。

       * > **易错点**：不等式链里的比值必须加绝对值，否则左边完全可能是负的，不等式就没有意义。

   * **收敛半径与收敛域**

       * 半径只说「从中心往两边多远之内绝对收敛、之外必发散」；端点处收敛与否公式一概不管，必须逐个代回去单独验。

       * > **易错点**：收敛半径和收敛域是两个概念，对幂级数都要记住。

## 收敛域的端点判定

   * **端点结构随中心平移**

       * $\sum\frac{x^n}{n}$、$\sum\frac{(x-1)^n}{n}$、$\sum\frac{(x+1)^n}{n}$：系数都是 $\frac1n$，$R$ 统统是 $1$；一端代出 $\sum\frac1n$ 发散、另一端代出 $\sum\frac{(-1)^n}{n}$ 交错收敛，收敛域分别为 $[-1,1)$、$[0,2)$、$[-2,0)$——左闭右开，整体随中心平移。

   * **系数带指数增长的例子**

       * $\sum\limits_{n=1}^{\infty}n\left(3^{n}+1\right)\left(x-\tfrac12\right)^{n}$：$\sqrt[n]{n(3^n+1)}=1\times3=3$（$3^n+1$ 也好、$3^n-1$ 也好，开出来都是 $3$），$R=\tfrac13$；两端点 $\tfrac16$、$\tfrac56$ 处通项绝对值约等于 $n$、不趋于零，收敛域为开区间 $\left(\tfrac16,\tfrac56\right)$。

   * **要请斯特林公式的例子**

       * $\sum\limits_{n=1}^{\infty}\frac{n^{n}}{n!}x^{n}$（从 $n=1$ 起避免 $0^0$）：$\frac{a_{n+1}}{a_n}=\left(1+\frac1n\right)^{n}\to\mathrm e$，$R=\dfrac1{\mathrm e}$。

       * 右端点 $x=\tfrac1{\mathrm e}$ 发散：用斯特林公式（Stirling formula）$n!\sim\sqrt{2\pi n}\,n^{n}\mathrm e^{-n}$，通项 $\dfrac{n^{n}}{n!\,\mathrm e^{n}}\sim\dfrac{1}{\sqrt{2\pi n}}$，是 $p=\tfrac12$ 的发散 $p$ 级数。

       * 左端点 $x=-\tfrac1{\mathrm e}$ 收敛：$u_{n+1}/u_n=\dfrac1{\mathrm e}\left(1+\dfrac1n\right)^{n}<1$ 对每个 $n$ 成立（$\left(1+\frac1n\right)^{n}$ 单调增加收敛于 $\mathrm e$、永远比 $\mathrm e$ 小），$u_n$ 单调减少；$u_n\sim\frac{1}{\sqrt{2\pi n}}\to0$；莱布尼茨级数。收敛域 $\left[-\tfrac1{\mathrm e},\tfrac1{\mathrm e}\right)$。

       * > **易错点**：验单调性不能只盯比值极限——极限是 $1$ 什么也说明不了，不知道 $u_{n+1}$ 比 $u_n$ 大还是小；救场的是 $\left(1+\frac1n\right)^{n}$ 递增收敛于 $\mathrm e$ 这条具体的不等式知识。

## 阿贝尔定理

   * **阿贝尔第一定理（Abel's first theorem）**

       * 若 $\sum a_nx^n$ 在 $x=x_c$ 收敛，则 $|x|<|x_c|$ 时**绝对收敛**；若在 $x=x_1$ 发散，则 $|x|>|x_1|$ 时**发散**。

       * 学过柯西—阿达玛后它是推论：在 $x_c$ 收敛说明 $|x_c|\le R$，在 $x_1$ 发散说明 $|x_1|\ge R$。

   * **阿贝尔第二定理（Abel's second theorem）**

       * 结论一：幂级数在 $(-R,R)$ 内每点收敛（且绝对收敛），并对**任意**闭区间 $[a,b]\subset(-R,R)$ 一致收敛——即**内闭一致收敛**。

       * 结论二：若级数在端点 $x=R$ 收敛，则对任意 $a\in(-R,R)$，级数在 $[a,R]$ 上一致收敛。

       * 结论一证明：取 $\xi=\max\{|a|,|b|\}<R$，则 $|a_nx^{n}|\le|a_n|\xi^{n}$，右端是与 $x$ 无关的收敛正项级数，魏尔斯特拉斯判别法立刻收案。

       * 结论二证明：拆 $a_nx^{n}=a_nR^{n}\cdot\left(\frac xR\right)^{n}$，逐条对号阿贝尔判别法（$\sum a_nR^{n}$ 收敛且与 $x$ 无关、$\left|\left(\frac xR\right)^{n}\right|\le1$ 一致有界、$\left(\frac xR\right)^{n}$ 关于 $n$ 单调），先得 $[0,R]$；$a<0$ 时把 $[a,R]$ 拆成 $[a,0]\cup[0,R]$，两段的 $N_1,N_2$ 取 max。

       * 对称结论：在 $x=-R$ 收敛则在任意 $[-R,b]$（$b\in(-R,R)$）上一致收敛；两头都收敛则在整个 $[-R,R]$ 上一致收敛。

       * 一句话概括：幂级数在包含于它的收敛域当中的闭区间上一致收敛。

       * > **易错点**：这个闭区间一定要「闭」——收敛域开的那头闭区间够不到；只有收敛域这一端本身是闭的，闭区间才可以顶到端点。

## 幂级数的分析性质

   * **和函数连续性**

       * $S(x)=\sum a_nx^n$ 在 $(-R,R)$ 上连续；若级数在 $x=R$ 收敛，则 $S$ 在 $R$ 处**左连续**；在 $x=-R$ 收敛则**右连续**。

       * 不用专门证：通项 $a_nx^n$ 个个连续，级数在含于收敛域的闭区间（含 $[a,R]$ 这类）上一致收敛，和函数当然连续。

   * **逐项积分**

       * $a,b$ 是收敛域中任意两点：$\int_a^b\Bigl(\sum_{n=0}^{\infty}a_nx^{n}\Bigr)dx=\sum_{n=0}^{\infty}\int_a^b a_nx^{n}\,dx$——「先求积分、再求和」与「先求和、再求积分」次序可交换。

       * 特别地 $\int_0^{x}\sum a_nt^{n}\,dt=\sum\limits_{n=0}^{\infty}\frac{a_n}{n+1}x^{n+1}$；凭据是幂级数在 $[a,b]$ 上一致收敛（阿贝尔第二定理）。

       * 逐项积分得到的新幂级数收敛半径**不变**：$\sqrt[n+1]{\frac{|a_n|}{n+1}}$ 与 $\sqrt[n]{|a_n|}$ 的上极限相等（根指数差一个趋于 $1$ 的因子）。

       * 但收敛域**可能扩大**：系数多除了一个 $n+1$ 变小，增加端点收敛的可能性——是「可能」，不是「肯定」。

   * **逐项求导**

       * 在收敛域内部 $(-R,R)$：$\left(\sum_{n=0}^{\infty}a_nx^{n}\right)'=\sum_{n=1}^{\infty}n\,a_nx^{n-1}$——常数项一求导就没有了，下标从 $n=1$ 起。

       * 新级数 $\sum na_nx^{n-1}$ 收敛半径也是 $R$：$\sqrt[n-1]{n|a_n|}$ 的上极限与 $\sqrt[n]{|a_n|}$ 相同（$n^{\frac1{n-1}}\to1$）。

       * > **注意**：逐项求导所要求的一致收敛，是对**逐项求导以后**的级数说的，不是原级数一致收敛就行；求导后的级数在 $(-R,R)$ 内闭一致收敛，条件自动满足。

       * > **提示**：内闭一致收敛，指在收敛区间内部的任何一个闭子区间上一致收敛；幂级数在 $(-R,R)$ 内总是内闭一致收敛。

   * **三条性质的作用范围对比**

| 性质 | 成立范围 | 端点 $R$ 处 |
| :--- | :--- | :--- |
| 和函数连续 | 整个收敛域 | $R$ 属于收敛域就连续 |
| 逐项积分 | 包含于收敛域的任意闭区间 $[a,b]$ | $a>-R$；$b$ 可以取到 $R$ |
| 逐项求导 | 收敛域内部 $(-R,R)$ | 不保证：求导以后的级数在 $R$ 处可能发散 |

## 和函数的认定与数项级数求和

   * **从 $S'=S$ 认出指数函数**

       * $S(x)=\sum_{n=0}^{\infty}\frac{x^n}{n!}$：达朗贝尔比值 $\frac{1/(n+1)!}{1/n!}\to0$，$R=+\infty$，处处可逐项求导，得 $S'(x)=S(x)$，且 $S(0)=1$。

       * 技巧：$\left(\mathrm e^{-x}S(x)\right)'=\mathrm e^{-x}\left(S'(x)-S(x)\right)\equiv0$，故 $\mathrm e^{-x}S(x)$ 恒为常数，代 $x=0$ 得常数 $1$，即 $S(x)=\mathrm e^{x}$。

       * 同款通法：$S'=aS$ 时乘 $\mathrm e^{-ax}$ 再求导，得 $S(x)=S(0)\,\mathrm e^{ax}$——数学分析版的常微分方程小技巧。

   * **两个关于 $\pi$ 与 $\ln 2$ 的级数和**

       * $\frac{1}{1+x^{2}}=\sum_{n=1}^{\infty}(-1)^{n-1}x^{2n-2}$ 只在 $(-1,1)$ 成立；逐项积分得 $\arctan x=\sum_{n=1}^{\infty}\frac{(-1)^{n-1}}{2n-1}x^{2n-1}$，新级数收敛域扩成 $[-1,1]$；由和函数在 $x=1$ 连续，$F(1)=\lim\limits_{x\to1^-}\arctan x=\dfrac\pi4$，即 $1-\frac13+\frac15-\frac17+\cdots=\dfrac\pi4$。

       * $\frac{1}{1+x}=\sum_{n=0}^{\infty}(-1)^{n}x^{n}$ 只在 $(-1,1)$ 成立；逐项积分得 $\ln(1+x)=\sum_{n=1}^{\infty}\frac{(-1)^{n-1}}{n}x^{n}$，新级数收敛域扩成 $(-1,1]$；$G(1)=\lim\limits_{x\to1^-}\ln(1+x)=\ln2$，即 $1-\frac12+\frac13-\frac14+\cdots=\ln2$。

       * > **易错点**：级数在端点收敛后，不能顺手把端点代进 $\arctan x$、$\ln(1+x)$——那个等式只在开区间里成立；正确做法是用和函数在收敛域上的连续性取极限 $x\to1^-$。

   * **数项级数求和例**

       * $\sum\limits_{n=1}^{\infty}\frac{2n+1}{3^{n}}$：拆成 $2\sum n\left(\frac13\right)^{n}+\sum\left(\frac13\right)^{n}$；由 $\sum nx^{n-1}=\frac{1}{(1-x)^{2}}$ 乘 $x$ 得 $\sum nx^{n}=\frac{x}{(1-x)^{2}}$，代 $x=\frac13$：$2\cdot\frac34+\frac12=2$。

   * **和函数例**

       * $\sum_{n=0}^{\infty}(n^{2}+1)\frac{x^{n}}{2^{n}n!}$：先把 $\frac{x^n}{2^n}$ 收成 $\left(\frac x2\right)^{n}$；$n^{2}$ 块从 $n=1$ 起且 $\frac{n^{2}}{n!}=\frac{n}{(n-1)!}$，由原型 $x\,\mathrm e^{x}=\sum_{n=1}^{\infty}\frac{x^{n}}{(n-1)!}$ 逐项求导再乘 $x$ 得 $\sum\frac{n(x/2)^{n}}{(n-1)!}=\frac x2\left(1+\frac x2\right)\mathrm e^{x/2}$；$1$ 块就是 $\mathrm e^{x/2}$；合计 $S(x)=\left(1+\frac x2+\frac{x^{2}}{4}\right)\mathrm e^{x/2}$。

## 柯西乘积

   * **幂级数的乘法**

       * 设 $f(x)=\sum_{n=0}^{\infty}a_nx^{n}$（收敛半径 $R_1$）、$g(x)=\sum_{n=0}^{\infty}b_nx^{n}$（收敛半径 $R_2$），则 $f(x)\,g(x)=\sum_{n=0}^{\infty}\Bigl(\sum_{i+j=n}a_ib_j\Bigr)x^{n}$，当 $|x|<\min(R_1,R_2)$ 时收敛且绝对收敛。

       * 这就是柯西乘积（Cauchy product）从 $0$ 编号的样子：$c_0=a_0b_0$，$c_1=a_1b_0+a_0b_1$，……每一块都是满足 $i+j=n$ 的 $a_ib_j$。

       * 为什么必须按斜对角线乘：$a_ix^{i}\cdot b_jx^{j}=a_ib_jx^{i+j}$，同一条对角线上次数相同，加起来才是干净的 $c_nx^{n}$——相当于合并同类项；正方形排列虽对一般收敛级数也成立、条件更宽松，但凑不出按 $x$ 的幂次排列的展开。

   * **去掉绝对收敛条件的柯西乘积**

       * 定理：设 $\sum a_n$、$\sum b_n$ 收敛，且它们的柯西乘积 $\sum c_n$（$c_n=a_1b_n+a_2b_{n-1}+\cdots+a_nb_1$）也收敛，则 $\sum c_n=\Bigl(\sum a_n\Bigr)\Bigl(\sum b_n\Bigr)$。

       * 证明骨架：造 $f(x)=\sum a_nx^{n}$、$g(x)=\sum b_nx^{n}$、$h(x)=\sum c_nx^{n}$，三者在 $x=1$ 都收敛，收敛域至少含 $(-1,1]$；在 $(-1,1)$ 内三幂级数都绝对收敛，柯西乘积可用，得 $f(x)g(x)=x\,h(x)$（首项 $a_1x\cdot b_1x=x^{2}$，乘积各项比 $h$ 的对应项高一次幂）；三个和函数在 $x=1$ 连续，令 $x\to1^-$ 取极限得 $f(1)g(1)=h(1)$。

## 泰勒级数与展开判据

   * **系数唯一性**

       * 只要 $f$ 能在 $x_0$ 的某邻域展开成 $\sum a_n(x-x_0)^{n}$，系数就被各阶导数定死：$a_K=\dfrac{f^{(K)}(x_0)}{K!}$（$K=0,1,2,\dots$，$K=0$ 即 $a_0=f(x_0)$）。

       * 推导：幂级数可逐项求导，求 $K$ 次得 $f^{(K)}(x)=\sum_{n=K}^{\infty}n(n-1)\cdots(n-K+1)\,a_n\,(x-x_0)^{n-K}$，代 $x=x_0$ 只剩 $n=K$ 项，得 $f^{(K)}(x_0)=K!\,a_K$。

   * **泰勒系数与泰勒级数**

       * > **定义**：$a_K=\dfrac{f^{(K)}(x_0)}{K!}$ 称为 $f$ 在 $x_0$ 点的**泰勒系数**（Taylor coefficients）；以泰勒系数为系数构造的幂级数 $\sum\limits_{n=0}^{\infty}\dfrac{f^{(n)}(x_0)}{n!}(x-x_0)^{n}$ 称为 $f$ 在 $x_0$ 点的**泰勒级数**（Taylor series）。

       * 构造前提是 $f$ 在 $x_0$ 某邻域上**任意阶可导**——系数是各阶导数除以阶乘，首先各阶导数都得存在。

       * 要害问题：泰勒级数收敛时，收到的值是不是 $f(x)$？**不一定**——完全可能不收敛到 $f$ 自己。

   * **展开的充要条件**

       * 定理：设 $f$ 在 $x_0$ 的某 $\rho$ 邻域上任意阶可导，则 $f(x)=\sum_{n=0}^{\infty}\frac{f^{(n)}(x_0)}{n!}(x-x_0)^{n}$ 在该邻域上成立的充要条件是：$\lim\limits_{N\to\infty}R_N(x)=0$ 对邻域内每一点 $x$ 成立。

       * 道理：泰勒公式是「有限和加余项」，让 $N\to\infty$，有限和变成泰勒级数，部分和收敛于 $f(x)$ 与余项趋于零是一回事。

| 余项 | 写法 | 特点 |
| :--- | :--- | :--- |
| 皮亚诺余项（Peano remainder） | $R_N(x)=o\bigl((x-x_0)^{N}\bigr)$ | 只给无穷小的阶，不给数量，判展开用不上 |
| 拉格朗日余项（Lagrange remainder） | $R_N(x)=\dfrac{f^{(N+1)}(\xi)}{(N+1)!}(x-x_0)^{N+1}$，$\xi=x_0+\theta(x-x_0)$，$\theta\in(0,1)$ | 带具体表达式，可以取极限 |

## 任意阶可导却展不开的反例

   * **反例函数**

       * $f(x)=\begin{cases}\mathrm e^{-\frac{1}{x^{2}}}, & x\neq 0,\\ 0, & x=0.\end{cases}$ 在整条实轴 $(-\infty,+\infty)$ 上任意阶可导，但它的泰勒级数在任何邻域里都不收敛于 $f$。

       * $x\neq0$ 处 $f^{(n)}(x)=P_n\Bigl(\frac{1}{x}\Bigr)\mathrm e^{-\frac{1}{x^{2}}}$，其中 $P_n(u)$ 是 $u$ 的 $3n$ 次多项式（核对：$f'(x)=\frac{2}{x^{3}}\mathrm e^{-1/x^{2}}$，$f''(x)=\Bigl(\frac{4}{x^{6}}-\frac{6}{x^{4}}\Bigr)\mathrm e^{-1/x^{2}}$）。

       * 地基极限：$\lim\limits_{x\to 0}\dfrac{\mathrm e^{-\frac{1}{x^{2}}}}{x^{k}}=0$ 对任意固定 $k$ 成立——令 $t=\frac{1}{x^{2}}$ 化为 $\frac{t^{k/2}}{\mathrm e^{t}}\to0$，指数增长压倒幂增长。

       * 按定义逐阶归纳：每一步都是 $\lim\limits_{x\to0}P_{3K-2}\bigl(\frac1x\bigr)\mathrm e^{-1/x^{2}}=0$ 的同一模式，得 $f^{(K)}(0)=0$ 对一切 $K=0,1,2,\dots$ 成立。

       * 泰勒系数全部是零，泰勒级数是地地道道的零级数：$x=0$ 处恰好等于 $f(0)$；只要 $x\neq0$，$f(x)>0$ 而级数和是零——反例坐实。

       * > **提示**：这条「指数压倒幂」的极限是整个反例的地基，后面每一步归纳都靠它放行。

   * **反例的用途：光滑拼接**

       * 实际问题：造一个左半（$x\le0$）恒为零、右半大于零、在接点光滑衔接的函数——右边接 $x^{2}$ 光滑度断在二阶，接 $x^{N}$ 断在 $N$ 阶，幂函数拼接的光滑度永远有天花板。

       * 右边接 $\mathrm e^{-1/x^{2}}$ 这一款：接点处任意阶导数都存在；以后要让曲线光滑衔接且任意阶可导，靠的就是这类函数。

## 标准函数的幂级数展开

   * **指数函数**

       * $\mathrm e^{x}=\sum\limits_{n=0}^{\infty}\dfrac{x^{n}}{n!}$，对一切 $x\in(-\infty,+\infty)$ 成立，收敛半径 $+\infty$。

       * 验收：$R_N(x)=\dfrac{\mathrm e^{\xi}}{(N+1)!}x^{N+1}$，$\xi=\theta x$；一头 $|\mathrm e^{\xi}|\le\mathrm e^{|x|}$ 有界，另一头 $(N+1)!$ 的增长压倒幂的增长，余项对一切 $x$ 趋于零。

       * 在任意点 $x_0$ 展开的变式：拆 $\mathrm e^{x}=\mathrm e^{x_0}\cdot\mathrm e^{\,x-x_0}$，得 $\sum\limits_{n=0}^{\infty}\dfrac{\mathrm e^{x_0}}{n!}(x-x_0)^{n}$。

   * **正弦与余弦**

       * $\sin x=\sum\limits_{n=0}^{\infty}\dfrac{(-1)^{n}}{(2n+1)!}x^{2n+1}$，$\cos x=\sum\limits_{n=0}^{\infty}\dfrac{(-1)^{n}}{(2n)!}x^{2n}$，均对一切实数成立。

       * $\sin x$ 只显式写到 $x^{2N+1}$，但 $x^{2N+2}$ 的系数是零，余项从 $2N+2$ 档编起；$R_{2N+2}(x)=\dfrac{\sin\Bigl(\xi+\frac{(2N+3)\pi}{2}\Bigr)}{(2N+3)!}x^{2N+3}$（每求一次导辐角加 $\frac\pi2$），正弦因子有界、阶乘压倒幂，余项趋于零。

   * **反正切与对数（逐项积分所得）**

       * $\arctan x=\sum\limits_{n=0}^{\infty}\dfrac{(-1)^{n}}{2n+1}x^{2n+1}$，$x\in[-1,1]$，两端都闭。

       * $\ln(1+x)=\sum\limits_{n=1}^{\infty}\dfrac{(-1)^{n-1}}{n}x^{n}$，$x\in(-1,1]$，$x=1$ 闭、$x=-1$ 开。

       * > **易错点**：函数有意义 $\neq$ 级数收敛。$|x|>1$ 时 $\arctan x$ 好好地有意义，可右边的级数已经发散——展开式只在收敛范围内说话。

   * **五个基本展开汇总**

| 函数 | 幂级数展开 | 收敛范围 |
| :--- | :--- | :--- |
| $\mathrm e^{x}$ | $\displaystyle\sum_{n=0}^{\infty}\frac{x^{n}}{n!}$ | $(-\infty,+\infty)$ |
| $\sin x$ | $\displaystyle\sum_{n=0}^{\infty}\frac{(-1)^{n}}{(2n+1)!}x^{2n+1}$ | $(-\infty,+\infty)$ |
| $\cos x$ | $\displaystyle\sum_{n=0}^{\infty}\frac{(-1)^{n}}{(2n)!}x^{2n}$ | $(-\infty,+\infty)$ |
| $\arctan x$ | $\displaystyle\sum_{n=0}^{\infty}\frac{(-1)^{n}}{2n+1}x^{2n+1}$ | $[-1,1]$ |
| $\ln(1+x)$ | $\displaystyle\sum_{n=1}^{\infty}\frac{(-1)^{n-1}}{n}x^{n}$ | $(-1,1]$ |

## 余项的积分形式与柯西余项

   * **余项的积分形式**

       * 定理：设 $f$ 在 $O(x_0,R)$ 内任意阶可导，则对 $x\in O(x_0,R)$，泰勒公式的余项可写成积分

       * $$R_n(x)=\frac{1}{n!}\int_{x_0}^{x}f^{(n+1)}(t)(x-t)^n\,\mathrm{d}t$$

       * 证明第一步：$R_n(x)=f(x)-\sum_{k=0}^{n}\frac{f^{(k)}(x_0)}{k!}(x-x_0)^k$ 逐次求导，每求一次幂次掉一格、起点抬一格，得 $R_n(x_0)=R_n'(x_0)=\cdots=R_n^{(n)}(x_0)=0$ 与 $R_n^{(n+1)}(x)=f^{(n+1)}(x)$。

       * > **易错点**：代 $x=x_0$ 不存在 $0^0$ 问题——$k=0$ 那一项是和式里的**常数项** $f(x_0)$，根本不带 $(x-x_0)$ 的幂。

       * 证明第二步：从牛顿—莱布尼茨公式 $R_n(x)=\int_{x_0}^{x}R_n'(t)\,\mathrm dt$ 起手，把 $\mathrm dt$ 写成 $\mathrm d(t-x)$（微分号里减常数微分不变），分部积分甩出的边界项两端全零（$t=x$ 处因子为零、$t=x_0$ 处 $R_n'$ 为零）；同款动作滚 $n$ 次得积分形式。

   * **两种余项的导出**

       * 都用第一积分中值定理从积分形式拿项：拿出 $f^{(n+1)}(\xi)$，剩 $\int_{x_0}^{x}(x-t)^n\mathrm dt=\frac{(x-x_0)^{n+1}}{n+1}$，回到拉格朗日余项。

       * 拿出整个 $f^{(n+1)}(t)(x-t)^{n}$，积分号里剩 $1$ 的积分，得**柯西余项**（Cauchy remainder）：$R_n(x)=\dfrac{f^{(n+1)}(\xi)}{n!}(1-\theta)^{n}(x-x_0)^{n+1}$。

       * > **注意**：第一积分中值定理要求留在积分号里的因子不变号；$(x-t)^{n}$ 作为 $t$ 的函数是定号的（积分变量是 $t$，$x$、$n$ 都固定），条件满足。

| | 拉格朗日余项 | 柯西余项 |
| :--- | :---: | :---: |
| 表达式 | $\dfrac{f^{(n+1)}(\xi)}{(n+1)!}(x-x_0)^{n+1}$ | $\dfrac{f^{(n+1)}(\xi)}{n!}(1-\theta)^{n}(x-x_0)^{n+1}$ |
| 分母上的阶乘 | $(n+1)!$ | $n!$ |
| 分子上多出的因子 | 无 | $(1-\theta)^{n}$ |

## 二项式级数

### 广义二项式系数与收敛半径

   * **广义二项式系数（generalized binomial coefficient）**

       * > **定义**：$\dbinom{\alpha}{k}=\dfrac{\alpha(\alpha-1)\cdots(\alpha-k+1)}{k!}$（$k=1,2,\dots$），并约定 $\dbinom{\alpha}{0}=1$；写法跟组合数一模一样，$\alpha$ 是比 $k$ 大的正整数时就是 $\mathrm C_{\alpha}^{k}$。

       * $\alpha$ 为正整数 $m$：二项式展开只有穷多项 $(1+x)^{m}=\sum_{k=0}^{m}\mathrm C_m^{k}x^{k}$，成立范围 $(-\infty,+\infty)$，没有收敛性问题。

       * $\alpha$ 非正整数（$\alpha=0$ 平凡，不必考虑）：逐阶求导每求一次掉一个因子，$f^{(k)}(0)=\alpha(\alpha-1)\cdots(\alpha-k+1)$，泰勒级数为 $\sum\limits_{n=0}^{\infty}\dbinom{\alpha}{n}x^{n}$。

       * 收敛半径：相邻系数比 $\dfrac{\binom{\alpha}{n+1}}{\binom{\alpha}{n}}=\dfrac{\alpha-n}{n+1}$，绝对值的极限为 $1$，达朗贝尔给 $R=1$。

### 用柯西余项打通 $(-1,1)$

   * **拉格朗日余项卡壳**

       * $f^{(n+1)}(\xi)=(n+1)!\binom{\alpha}{n+1}(1+\xi)^{\alpha-n-1}$，以 $\xi=\theta x$ 代入得 $R_n=\binom{\alpha}{n+1}x^{n+1}(1+\theta x)^{\alpha-n-1}$。

       * $x>0$ 时 $1+\theta x>1$，$(1+\theta x)^{n}$ 越乘越大，分式被压住；$x<0$ 时 $1+\theta x$ 严格小于 $1$，而 $\theta$ 怎么随 $n$ 变一无所知——$(1+\theta x)^{-n}$ 既说不了有界也说不了无穷大量，路走不通。

   * **换柯西余项，$(1-\theta)^{n}$ 派上用场**

       * 合并后 $R_n=(n+1)\dbinom{\alpha}{n+1}x^{n+1}\cdot(1+\theta x)^{\alpha-1}\cdot\left(\dfrac{1-\theta}{1+\theta x}\right)^{n}$，三个因子逐个收拾。

           * 第一因子趋于零：建幂级数 $\sum(n+1)\binom{\alpha}{n}x^{n}$，相邻系数之比的极限不变，收敛半径仍是 $1$，$|x|<1$ 时通项趋于零。

           * $(1+\theta x)^{\alpha-1}$ 有界：$\theta\in(0,1)$、$|x|<1$ 给出 $1-|x|<1+\theta x<1+|x|$，故 $|1+\theta x|^{\alpha-1}\le\max\bigl\{(1-|x|)^{\alpha-1},\,(1+|x|)^{\alpha-1}\bigr\}$，是与 $n$、$\theta$ 都无关的常数。

           * $\left(\frac{1-\theta}{1+\theta x}\right)^{n}$ 有界：$(1+\theta x)-(1-\theta)=\theta(1+x)>0$（$x>-1$），故 $0<1-\theta<1+\theta x$，比值落在 $0$ 与 $1$ 之间。

       * > **易错点**：估 $(1+\theta x)^{\alpha-1}$ 不能随手写 $\le1$——$\theta$ 随 $n$ 变，且指数 $\alpha-1$ 是正是负不定，负的时候底数越小值反而越大；要老实用 $1\pm|x|$ 夹住。

       * 结论：对任意实数 $\alpha$，$(1+x)^{\alpha}=\sum\limits_{n=0}^{\infty}\dbinom{\alpha}{n}x^{n}$ 在 $(-1,1)$ 内成立；柯西余项里多出的 $(1-\theta)^{n}$ 正是压住 $(1+\theta x)^{-n}$ 这块硬骨头的关键。

### 端点 $x=\pm1$ 的收敛

   * **$\alpha\le-1$：通项不趋于零**

       * 端点处 $|x|=1$，$x$ 从通项里消失，只剩系数 $|u_n|=\frac{|\alpha|\cdot|\alpha-1|\cdots|\alpha-n+1|}{n!}$；第 $k$ 个因子 $|\alpha-k|\ge k+1$（$\alpha=-1$ 时取等号），每个分式不小于 $1$，累乘得 $|u_n|\ge1$——级数发散，两端全丢。

   * **$-1<\alpha<0$：$x=1$ 收敛，$x=-1$ 发散**

       * $x=1$ 处：新因子 $\alpha-n$ 永远是负的，每加一项翻一次号，是交错级数；$u_{n+1}=u_n\cdot\frac{\alpha-n}{n+1}$，绝对值因子 $\frac{n-\alpha}{n+1}<1$（因 $\alpha>-1$），$|u_n|$ 严格单调减少；$|u_n|=\prod_{k=1}^{n}\Bigl(1-\frac{1+\alpha}{k}\Bigr)\to0$（同号无穷乘积，因 $\sum\frac{1+\alpha}{k}$ 调和型发散而发散到零）；莱布尼茨条件齐了，收敛。

       * $x=-1$ 处：$n$ 个负因子的乘积本来带符号 $(-1)^n$，再乘 $(-1)^n$ 负负得正——每一项全是正的，用不上交错工具；与 $\frac1n$ 比较，每个因子 $\frac{k-\alpha}{k}=1+\frac{|\alpha|}{k}>1$，得 $|u_n|>\dfrac{|\alpha|}{n}$，比较判别法判发散。

   * **$\alpha>0$：拉贝判别法一并解决**

       * 拉贝判别法（Raabe test）：对正项级数考察 $\lim\limits_{n\to\infty}n\Bigl(\dfrac{|u_n|}{|u_{n+1}|}-1\Bigr)$，极限大于 $1$ 则收敛。

       * 这里 $n\Bigl(\frac{|u_n|}{|u_{n+1}|}-1\Bigr)=n\Bigl(\frac{n+1}{n-\alpha}-1\Bigr)=\dfrac{n(1+\alpha)}{n-\alpha}\to1+\alpha>1$，两端点都绝对收敛。

   * **端点汇总**

| $\alpha$ 的范围 | 在 $x=-1$ | 在 $x=1$ | 展开式成立的范围 |
| :--- | :--- | :--- | :--- |
| $\alpha\le-1$ | 发散 | 发散 | $-1<x<1$ |
| $-1<\alpha<0$ | 发散 | 收敛 | $-1<x\le1$ |
| $\alpha>0$ | 收敛 | 收敛 | $-1\le x\le1$ |

       * 规律：$\alpha$ 越大，端点保住得越多。

## 间接展开的方法

   * **先展开再逐项求导 / 积分**

       * 求 $\dfrac{1}{x^{2}}$ 在 $x=1$ 的展开（$x=0$ 处函数没定义）：凑熟面孔 $\dfrac1x=\dfrac{1}{1+(x-1)}=\sum_{n=0}^{\infty}(-1)^{n}(x-1)^{n}$，在收敛区间内逐项求导得 $\dfrac{1}{x^{2}}=\sum_{n=0}^{\infty}(-1)^{n}(n+1)(x-1)^{n}$，$x\in(0,2)$（端点通项绝对值 $n+1$ 不趋于零）。

       * > **易错点**：换求和指标是两头的事——$n-1\to n$ 的同时，式子里其余的 $n$ 全要改成 $n+1$，起点也要跟着从 $1$ 挪到 $0$；只换一处，级数就换错了。

   * **因式分解 + 部分分式**

       * $\dfrac{1}{3+5x-2x^{2}}$ 在 $x=0$ 展开：老老实实求各阶导数苦不堪言，先分解 $3+5x-2x^{2}=(3-x)(1+2x)$，拆 $\dfrac17\Bigl(\dfrac{1}{3-x}+\dfrac{2}{1+2x}\Bigr)$，两块分别套几何级数得 $\dfrac{1}{7}\sum\limits_{n=0}^{\infty}\Bigl(\dfrac{1}{3^{n+1}}+(-1)^{n}2^{n+1}\Bigr)x^{n}$。

       * 收敛范围取小的：一块要 $|x|<3$、另一块要 $|x|<\frac12$，两个幂级数相加范围取小的，展开在 $\left(-\frac12,\frac12\right)$ 成立。

   * **柯西乘积**

       * $\mathrm e^{x}\sin x$ 展到五次：$x+x^{2}+\dfrac13x^{3}-\dfrac1{30}x^{5}+\cdots$，四次方两路 $\frac16x^4$ 与 $-\frac16x^4$ 正好抵消。

       * 各写多少项有讲究：$\sin x$ 最低次是一次，$\mathrm e^{x}$ 写到四次就够（五次以上的项乘过来至少六次），$\sin x$ 写到五次。

       * 两因子收敛半径都是 $+\infty$，乘积展开对一切 $x$ 成立；这类乘积通项难写，通常按题目要求展到某一项为止。

   * **待定系数法（method of undetermined coefficients）**

       * 前提 $b_0\neq0$（$b_0=0$ 则商在 $x=0$ 无从展开；且每解一个系数最后都要除以 $b_0$）。设 $\dfrac{f}{g}=\sum c_nx^n$，比较 $(\sum b_nx^{n})(\sum c_nx^{n})=\sum a_nx^{n}$ 的同次幂系数：$c_0=\dfrac{a_0}{b_0}$，通式 $c_n=\dfrac{1}{b_0}\Bigl(a_n-\sum_{k=1}^{n}b_kc_{n-k}\Bigr)$，每步只用已求出的系数。

       * $\tan x=\dfrac{\sin x}{\cos x}$ 展到五次：先看奇偶性——奇函数在零点的值为零给 $c_0=0$；奇函数的导数是偶函数、偶函数的导数是奇函数，二、四阶导数在零点全为零给 $c_2=c_4=0$；比较 $x^1,x^3,x^5$ 的系数解得 $\tan x=x+\dfrac13x^{3}+\dfrac2{15}x^{5}+\cdots$。

       * > **提示**：动手前先看奇偶性——奇函数只设奇次项、偶函数只设偶次项，能省掉一半未知数。

   * **代入法（substitution method）**

       * 把整块表达式当成一个 $U$ 代进已知展开式：$\tan x=\sin x\cdot\dfrac{1}{\cos x}$，先单独拿下 $\dfrac{1}{\cos x}$——$\cos x=1-U$（$U=\frac{x^{2}}{2!}-\frac{x^{4}}{4!}+\cdots$），$\dfrac{1}{\cos x}=\dfrac{1}{1-U}=1+U+U^{2}+\cdots=1+\dfrac12x^{2}+\dfrac5{24}x^{4}+\cdots$；再与 $\sin x$ 做柯西乘积，同样得到 $x+\frac{x^{3}}{3}+\frac{2x^{5}}{15}+\cdots$，两法互相印证。

       * 合法性两步走：$|x|$ 充分小时 $U$ 的最低次是 $x^{2}$、随 $x\to0$ 趋于 $0$，$|U|<1$ 成立，展开至少在小邻域内是对的；而幂级数系数 $a_n=\dfrac{f^{(n)}(0)}{n!}$ 完全由 $f$ 在零点的各阶导数决定，故只要收敛半径是 $R$，整个 $|x|<R$ 上展开都对。

       * > **注意**：代入法自带的空白——这样得到的展开其收敛半径在本课程里求不出来；以后学了复变函数可知 $\dfrac{1}{\cos x}$ 展开的收敛半径是 $\dfrac{\pi}{2}$。

## arcsin 展开与双阶乘

   * **双阶乘（double factorial）**

       * $(2n-1)!!=1\cdot3\cdot5\cdots(2n-1)$，$(2n)!!=2\cdot4\cdot6\cdots(2n)$。

       * 化简 $\dbinom{-1/2}{n}$：分子是 $-\frac12,-\frac32,\dots,-\frac{2n-1}{2}$，提出 $(-1)^{n}$ 剩奇数连乘；分母的 $2^{n}\cdot n!$ 恰是 $2\cdot4\cdots(2n)$，故 $\dbinom{-1/2}{n}=(-1)^{n}\dfrac{(2n-1)!!}{(2n)!!}$。

   * **$\arcsin x$ 的展开**

       * 路线：$\arcsin x$ 手头没有现成展开，先求导 $(\arcsin x)'=\dfrac{1}{\sqrt{1-x^{2}}}=\bigl(1+(-x^{2})\bigr)^{-\frac12}$，套 $(1+x)^{\alpha}$ 的展开，再逐项积分回去。

       * $\dfrac{1}{\sqrt{1-x^{2}}}=1+\sum\limits_{n=1}^{\infty}\dfrac{(2n-1)!!}{(2n)!!}x^{2n}$，$x\in(-1,1)$——$(-x^{2})^{n}$ 里的 $(-1)^{n}$ 与系数里的 $(-1)^{n}$ 全部抵消，系数都是正的。

       * > **易错点**：求和必须从 $n=1$ 写起、常数项 $1$ 单独放前面——若从 $n=0$ 开始，通项里的 $(2n-1)!!$ 就成了 $(-1)!!$，没有意义。

       * 逐项积分：$\arcsin x=x+\sum\limits_{n=1}^{\infty}\dfrac{(2n-1)!!}{(2n)!!}\cdot\dfrac{x^{2n+1}}{2n+1}$；逐项积分收敛半径不变仍为 $1$，端点 $x=\pm1$ 处该级数收敛（拉贝判别法），再用幂级数在收敛范围内连续，把等式推到整个闭区间 $[-1,1]$。

       * 令 $x=1$（$\arcsin 1=\dfrac\pi2$）：$\dfrac{\pi}{2}=1+\sum\limits_{n=1}^{\infty}\dfrac{(2n-1)!!}{(2n)!!}\cdot\dfrac{1}{2n+1}$——当年只用拉贝判别法判过它收敛，如今知道它收敛到 $\dfrac\pi2$。

## 由展开求级数和

   * **$\log\dfrac{\sin x}{x}$ 的展开**

       * $x=0$ 不在定义域里，视为补定义的函数：$f(0)=0$（$x\to0$ 时 $\frac{\sin x}{x}\to1$，$\log 1=0$），在 $0$ 的附近展开到 $x^{4}$。

       * 代入法：$\dfrac{\sin x}{x}=1-\dfrac{x^{2}}{3!}+\dfrac{x^{4}}{5!}-\cdots=1-U$，用 $\log(1-U)=-\Bigl(U+\dfrac{U^{2}}{2}+\dfrac{U^{3}}{3}+\cdots\Bigr)$（每一项都是负的，负号提前），得 $\log\dfrac{\sin x}{x}=-\dfrac{x^{2}}{6}-\dfrac{x^{4}}{180}-\cdots$。

       * 补定义后是偶函数、常数项为零，奇次项全没有；$x^{4}$ 系数来自两处：$-U$ 给 $+\frac1{120}$，$-\frac{U^{2}}{2}$ 给 $-\frac12\cdot\bigl(\frac16\bigr)^{2}=-\frac1{72}$，合起来 $\frac1{120}-\frac1{72}=-\frac1{180}$。

   * **从无穷乘积对出两个级数和**

       * $\sin x=x\prod\limits_{n=1}^{\infty}\Bigl(1-\dfrac{x^{2}}{n^{2}\pi^{2}}\Bigr)$；两边除 $x$ 取对数，乘积化求和，每项再用 $\log(1-U)$ 展开（$U=\dfrac{x^{2}}{n^{2}\pi^{2}}$，负号照旧提前）。

       * 得 $\log\dfrac{\sin x}{x}=-\sum\dfrac{x^{2}}{n^{2}\pi^{2}}-\sum\dfrac{x^{4}}{2n^{4}\pi^{4}}-\sum\dfrac{x^{6}}{3n^{6}\pi^{6}}-\cdots$，这与逐点代入法求出的展开是**同一个函数的两个幂级数展开**，同次幂系数应该相等，立即推出

       * $\sum\limits_{n=1}^{\infty}\dfrac{1}{n^{2}}=\dfrac{\pi^{2}}{6}$，$\sum\limits_{n=1}^{\infty}\dfrac{1}{n^{4}}=\dfrac{\pi^{4}}{90}$；把 $x^{6}$ 的项也求出来，$\sum\frac1{n^{6}}$ 跟着出来——多求几项还能得到更多等式，幂级数展开成了一条求级数和的路。

## 幂级数近似计算

   * **积不出的定积分**

       * $\displaystyle\int_0^1\mathrm e^{-x^{2}}\,\mathrm dx$：被积函数的原函数不是初等函数，先求原函数再走牛顿—莱布尼茨的路走不通，只能幂级数展开逐项积分。

       * $\mathrm e^{-x^{2}}=1-x^{2}+\frac{1}{2!}x^{4}-\frac{1}{3!}x^{6}+\cdots$，逐项积分得交错级数 $1-\dfrac13+\dfrac1{10}-\dfrac1{42}+\dfrac1{216}-\dfrac1{1320}+\dfrac1{9360}-\dfrac1{75600}+\cdots$，通项绝对值单调递减，是莱布尼茨型级数。

       * 误差估计：从某项起丢掉，误差不超过该项的绝对值；$\dfrac1{75600}<1.5\times10^{-5}$，取前七项相加误差 $<10^{-4}$，$\displaystyle\int_0^1\mathrm e^{-x^{2}}\,\mathrm dx\approx0.7468$，至少小数点后第四位精确。

   * **算 $\pi$：选对代入点**

       * $\arctan x=x-\dfrac{x^{3}}{3}+\dfrac{x^{5}}{5}-\dfrac{x^{7}}{7}+\cdots$（$x\in[-1,1]$）：代 $x=1$（$\arctan 1=\frac\pi4$）得 $\dfrac\pi4=1-\dfrac13+\dfrac15-\cdots$，缺点是收敛慢——误差不超过通项乘 $4$，$\dfrac{4}{2n+1}<10^{-5}$ 要 $n>2\times10^{5}$，约二十万项。

       * 代 $x=\dfrac1{\sqrt3}$（$\arctan\dfrac1{\sqrt3}=\dfrac\pi6$）：$\pi=2\sqrt3\,\Bigl(1-\dfrac19+\dfrac1{45}-\dfrac1{189}+\cdots\Bigr)$，通项 $\dfrac{2\sqrt3}{(2n+1)\,3^{n}}$ 每项多除了一个 $3$ 的幂，收敛快得多。

       * 取前九项，误差 $\dfrac{2\sqrt3}{19\cdot3^{9}}<10^{-5}$，小数点后第四位肯定精确：$\pi\approx3.1416$。

| 代入点 | 通项 | 误差 $<10^{-5}$ 所需项数 |
| :--- | :--- | :--- |
| $x=1$ | $\dfrac{4}{2n+1}$ | 约 $2\times10^{5}$ |
| $x=\dfrac{1}{\sqrt3}$ | $\dfrac{2\sqrt3}{(2n+1)3^{n}}$ | $9$ |

## Weierstrass 第一定理与 Bernstein 多项式

### 定理的叙述与构造

   * **一致逼近（uniform approximation）**

       * 这里的「逼近」专指一致逼近：不是某个点上慢慢靠近，而是在整个区间上步调一致地靠近。

   * **Weierstrass 第一定理**

       * 设 $f(x)$ 在闭区间 $[a,b]$ 上连续，则存在多项式序列 $\{P_n(x)\}$ 在 $[a,b]$ 上一致收敛于 $f(x)$：任给 $\varepsilon>0$，存在 $N$，使得 $n>N$ 时对一切 $x\in[a,b]$ 成立 $|P_n(x)-f(x)|<\varepsilon$。

       * 证明要做两件事：多项式序列要真的构造出来（定理说的是「存在」）；找出后还要证它在 $[a,b]$ 上一致收敛。一般 $[a,b]$ 作线性代换可化到 $[0,1]$，逼近出来的多项式换回原变量仍是多项式，不失一般性。

       * Weierstrass 第二定理：周期连续函数一定可以用三角多项式一致逼近，放在 Fourier 级数一章讲。

   * **Bernstein 多项式（Bernstein polynomial）**

       * > **定义**：设 $f$ 在 $[0,1]$ 上连续，称 $B_n(f,x)=\sum\limits_{k=0}^{n}f\Bigl(\dfrac kn\Bigr)\mathrm C_n^k\,x^{k}(1-x)^{n-k}$ 为 $f$ 的 $n$ 次 **Bernstein 多项式**，它是次数不超过 $n$ 的多项式。

       * 构造思路：$\bigl[x+(1-x)\bigr]^{n}=1$ 二项式展开成 $n+1$ 个 $n$ 次多项式之和；在每项前插系数 $a_k=f\bigl(\frac kn\bigr)$——把 $[0,1]$ $n$ 等分，$n+1$ 个分点 $0,\frac1n,\dots,1$ 上的函数值分别插到 $n+1$ 项前面。

       * 权重视角：每个 $\mathrm C_n^kx^{k}(1-x)^{n-k}$ 在 $[0,1]$ 上非负、全体之和恒为 $1$，$B_n(f,x)$ 相当于对 $n+1$ 个采样值做加权平均，权重随 $x$ 变化——「总和为 1」是整个构造能够成立的支点。

### Bernstein 映射的性质与特殊像

   * **映射的两条基本性质**

       * $B_n$ 把 $[0,1]$ 上连续函数全体映到多项式全体；**线性性**：$B_n(\alpha f+\beta g)=\alpha\,B_n(f)+\beta\,B_n(g)$（线性组合从求和号里拆出来）。

       * **单调性**：若对一切 $t$ 有 $f(t)\ge g(t)$，则 $B_n(f,x)\ge B_n(g,x)$——组合数与 $x^{k}(1-x)^{n-k}$ 在 $[0,1]$ 上都非负，逐项比过去不等号保持。

       * 这两条是对不等式整体求 Bernstein 像的两件兵刃：线性性拆项，单调性保号。

   * **特殊函数的像**

| 函数 $f(t)$ | 它的 $n$ 次 Bernstein 多项式 $B_n(f,x)$ |
| :---: | :---: |
| $1$（常数 $A$ 同理） | $A$ |
| $t$ | $x$ |
| $t^{2}$ | $x^{2}+\dfrac{x(1-x)}{n}$ |
| $(t-s)^{2}$（$s$ 视为常数） | $(x-s)^{2}+\dfrac{x(1-x)}{n}$ |

       * $B_n(t,x)=x$ 用组合公式 $\dfrac kn\mathrm C_n^{k}=\dfrac{(n-1)!}{(k-1)!\,(n-k)!}=\mathrm C_{n-1}^{k-1}$，剩下的凑成 $\bigl[x+(1-x)\bigr]^{n-1}=1$。

       * $B_n(t^{2},x)$ 的关键一招是把 $k$ 拆成 $(k-1)+1$，拆成 $\dfrac{n-1}{n}x^{2}$ 与 $\dfrac1nx$ 两块，各自凑成二项式展开。

       * $(t-s)^{2}$ 的像靠线性性拆 $B_n(t^{2})-2s\,B_n(t)+s^{2}B_n(1)$，合起来配成完全平方加误差项。

       * > **提示**：$B_n$ 对常数与一次函数「原样通过」（$1\mapsto1$，$t\mapsto x$），到 $t^{2}$ 才出现偏差 $\dfrac{x(1-x)}{n}$；偏差随 $n$ 增大而消失，是证明里唯一需要收拾的误差。

### 一致收敛的 Korovkin 证明

```text
f 在 [0,1] 连续 ⇒ 有界 |f|≤M 且一致连续（|t−s|<δ 时 |f(t)−f(s)|<ε/2）
  ↓ 两段放大：|t−s|≥δ 时改用 2M(t−s)²/δ²
|f(t)−f(s)| ≤ ε/2 + 2M(t−s)²/δ²        （对一切 t, s 成立）
  ↓ 去绝对值成三段，对不等式整体求 Bernstein 像（s 视为常数，单调性保号）
|B_n(f,x)−f(s)| ≤ ε/2 + 2M(x−s)²/δ² + 2M·x(1−x)/(nδ²)
  ↓ 令 s = x，交叉项 (x−s)² 自动消失
|B_n(f,x)−f(x)| ≤ ε/2 + M/(2nδ²)
  ↓ x(1−x) ≤ 1/4，取 N = [M/(δ²ε)]
n > N 时对一切 x ∈ [0,1]：|B_n(f,x)−f(x)| < ε
```

   * **五步要点**

       * 第一步：闭区间上连续函数必有界且一致连续；一致连续的 $\varepsilon$ 故意只取一半 $\dfrac\varepsilon2$——另一半腾给误差项，两半合成 $\varepsilon$。

       * 第二步：一致连续只管得住 $|t-s|<\delta$；$|t-s|\ge\delta$ 时改用 $|f(t)-f(s)|\le2M\le2M\dfrac{(t-s)^{2}}{\delta^{2}}$；两段合成对一切 $t,s$ 成立的一条二次上界。

       * 第三步：把绝对号去掉写成三段不等式，对关于 $t$ 的左、中、右三个函数分别求 Bernstein 像（常数 $f(s)$ 的像还是 $f(s)$，$(t-s)^{2}$ 的像是 $(x-s)^{2}+\frac{x(1-x)}{n}$），单调性保住不等号。

       * 第四步：所得不等式对一切 $x$ 与一切 $s$ 成立，令 $s=x$——$f(s)$ 变 $f(x)$，真正失控的 $(x-s)^{2}$ 项消失。

       * 第五步：$x$ 与 $1-x$ 非负、和为 $1$，两数相等时乘积最大，故 $x(1-x)\le\dfrac14$；取 $N=\left[\dfrac{M}{\delta^{2}\varepsilon}\right]$（取整数部分），$n>N$ 时第二项 $<\dfrac\varepsilon2$，合起来对一切 $x$ 有 $|B_n(f,x)-f(x)|<\varepsilon$——一致收敛证毕。

       * 方法属前苏联数学家科罗夫金（Korovkin，1953）；妙处在于：逐点估计无从下手，两段放大统一成一条上界，保号性把整条不等式原封搬进多项式世界，剩下的误差算得出、压得下去。
