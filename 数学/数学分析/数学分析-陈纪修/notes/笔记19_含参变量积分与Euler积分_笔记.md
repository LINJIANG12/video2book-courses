# 含参变量积分与Euler积分

## 含参常义积分的分析性质

   * **含参变量的积分（integral with parameter）**

       * > **定义**：设 $f(x,y)$ 在矩形区域 $D=[a,b]\times[c,d]$ 上连续，称 $I(y)=\displaystyle\int_a^b f(x,y)\,dx$（$y\in[c,d]$）为含参变量的常义积分，它是以 $y$ 为参变量的函数。

       * 机理：固定 $y$ 后 $f(x,y)$ 成为 $x$ 的连续函数，闭区间上必可积；积分值随 $y$ 而变，$y$ 在整个积分过程中被当作参变量。

       * 几何图像：在 $D$ 上固定 $y=y_0$，积分沿这条水平线段从 $a$ 扫到 $b$；换一个 $y_0$ 换一条线段，积出一个新值。

   * **四条性质的条件梯度**

| 要问的问题 | 需要的条件 | 得到的结论 |
| :--- | :--- | :--- |
| $I(y)$ 连不连续 | $f$ 在 $D$ 上连续 | 连续，极限与积分交换次序 |
| 两个累次积分相等吗 | 仍只要 $f$ 连续 | 积分次序可交换 |
| $I(y)$ 可不可导 | $f$ 与 $f_y'$ 都连续 | 求导可进入积分号 |
| 上下限也随 $y$ 变 | 再加 $a(y)$、$b(y)$ 可导且不出 $[a,b]$ | 变限求导公式（三项） |

       * 记忆线索：条件一道比一道苛刻，与函数项级数"和函数连续—逐项积分—逐项求导"的梯度同构。

   * **连续性定理**

       * 结论：$f$ 在闭矩形 $D$ 上连续 $\Rightarrow$ $I(y)$ 在 $[c,d]$ 上连续。

       * 机理：闭矩形是紧集，$f$ 在其上一致连续；$|I(y)-I(y_0)|\le\displaystyle\int_a^b\left|f(x,y)-f(x,y_0)\right|dx<(b-a)\,\varepsilon$，前面乘常数 $b-a$ 不碍趋零。

       * 实质：$\displaystyle\lim_{y\to y_0}\int_a^b f(x,y)\,dx=\int_a^b\lim_{y\to y_0}f(x,y)\,dx$——左边先积分后取极限、右边先取极限后积分，极限运算与积分运算可交换次序。

   * **积分次序交换定理**

       * 条件：仍只要 $f$ 在 $D$ 上连续。

       * 结论：$I(y)$ 在 $[c,d]$ 上可积，且 $\displaystyle\int_c^d\left(\int_a^b f(x,y)\,dx\right)dy=\int_a^b\left(\int_c^d f(x,y)\,dy\right)dx$。

       * 机理：$f$ 在矩形上的二重积分存在，两个累次积分都等于同一个二重积分，彼此自然相等。

   * **积分号下求导定理**

       * 条件：$f(x,y)$ 与偏导数 $f_y'(x,y)$ 都在 $D$ 上连续——比前两条各多一条。

       * 结论：$I'(y)=\displaystyle\int_a^b\frac{\partial f}{\partial y}(x,y)\,dx$，求导运算可以搬进积分号。

       * 证明骨架：差商 $\dfrac{I(y+\Delta y)-I(y)}{\Delta y}=\displaystyle\int_a^b\frac{f(x,y+\Delta y)-f(x,y)}{\Delta y}\,dx$，分子固定 $x$ 对 $y$ 用拉格朗日中值定理，得 $\displaystyle\int_a^b f_y'\bigl(x,y+\theta\Delta y\bigr)dx$（$0<\theta<1$），再用连续性把极限送进积分号。

   * **变限含参积分的求导公式**

       * > **定义**：$F(y)=\displaystyle\int_{a(y)}^{b(y)}f(x,y)\,dx$——被积函数与上、下限都依赖 $y$。

       * 条件：$f$、$f_y'$ 在 $D$ 上连续；$a(y)$、$b(y)$ 可导，且始终落在 $[a,b]$ 内。

       * 结论：$F'(y)=\displaystyle\int_{a(y)}^{b(y)}\frac{\partial f}{\partial y}(x,y)\,dx+f\bigl(b(y),y\bigr)\,b'(y)-f\bigl(a(y),y\bigr)\,a'(y)$。

       * 机理：令 $I(u,v,y)=\int_u^v f(x,y)\,dx$，$F$ 经三条路径依赖 $y$——对参数 $y$ 的偏导给出积分号内求导项；对下限 $u$ 的偏导是 $-f(u,y)$（下限求导带负号），对上限 $v$ 的偏导是 $+f(v,y)$，再各乘 $a'(y)$、$b'(y)$。

       * > **易错点**：$a(y)$、$b(y)$ 必须始终不出 $[a,b]$——$f$ 只在这个矩形上有定义，积分限一出界被积函数就失去意义。⚠️

---

## 常义含参积分的计算实例

   * **极限放进积分号**

       * 目标：$\displaystyle\lim_{\alpha\to0}\int_0^1\frac{\cos\alpha x}{1+x^2}\,dx$——先积后取极限走不通，这个积分不好积。

       * 处理：参变量是 $\alpha$；题目没给范围时取一个包含 $0$ 的闭矩形即可，如 $[0,1]\times\left[-\tfrac12,\tfrac12\right]$，$f$ 在其上连续，连续性定理允许极限进积分号。

       * 结果：$\displaystyle\int_0^1\frac{dx}{1+x^2}=\frac{\pi}{4}$。

   * **换序求积**

       * 目标：$\displaystyle\int_0^1\frac{x^b-x^a}{\ln x}\,dx$（$b>a>0$）——$\ln x$ 在分子上好解、跑到分母上就麻烦。

       * 关键识别：$\dfrac{\partial}{\partial y}\left(\dfrac{x^y}{\ln x}\right)=x^y$，对 $y$ 从 $a$ 到 $b$ 用牛顿–莱布尼茨给出 $\dfrac{x^b-x^a}{\ln x}=\displaystyle\int_a^b x^y\,dy$——被积函数其实是某个关于 $y$ 的积分。

       * 换序：$x^y$ 在 $[0,1]\times[a,b]$ 上连续，$\displaystyle\int_0^1 dx\int_a^b x^y\,dy=\int_a^b dy\int_0^1 x^y\,dx=\int_a^b\frac{dy}{1+y}=\ln\frac{1+b}{1+a}$。

   * **变限公式走一遍**

       * 目标：$F(y)=\displaystyle\int_0^y\frac{\ln(1+xy)}{x}\,dx$（$y>0$），求 $F'(y)$。

       * 逐项：下限 $0$ 是常数，第三项消失；第一项 $\displaystyle\int_0^y\frac{dx}{1+xy}=\frac{1}{y}\ln(1+y^2)$；第二项 $f(y,y)\cdot1=\dfrac{\ln(1+y^2)}{y}$。

       * 结果：两项一模一样，$F'(y)=\dfrac{2}{y}\ln(1+y^2)$。

       * > **提示**：$\dfrac{\ln(1+xy)}{x}$ 在 $x=0$ 处无直接定义，但 $x\to0$ 时趋于 $y$，补上这一点的定义后就是连续函数，$f_y'=\dfrac{1}{1+xy}$ 也连续，定理条件满足。

   * **积分号下求导救活一个积分**

       * 目标：$I(\theta)=\displaystyle\int_0^{\pi}\ln(1+\theta\cos x)\,dx$（$|\theta|<1$）；分部积分一上手就出 $\dfrac{x}{1+\theta\cos x}$，做不动。

       * > **易错点**：$|\theta|<1$ 不是随手写的限制——$\theta=\pm1$ 时 $x=\pi$ 或 $x=0$ 成为瑕点，积分变反常；它是"此积分为常义积分"的保证。

       * 验条件：$f$ 与 $f_\theta'$ 在 $[0,\pi]\times[-1,1]$ 上不连续（$|\theta|=1$ 处分母可取零）；但对每个 $|\theta|<1$ 可配 $a<1$ 使 $|\theta|<a$，在收紧的 $[0,\pi]\times[-a,a]$ 上分母 $\ge1-a>0$，条件齐备——矩形开小一点定理就能用，且结论对整个 $(-1,1)$ 成立。

       * 求导与拆项：$I'(\theta)=\displaystyle\int_0^\pi\frac{\cos x}{1+\theta\cos x}\,dx$；分子加减 $1$ 拆成 $I'(\theta)=\dfrac{\pi}{\theta}-\dfrac{1}{\theta}J$，其中 $J=\displaystyle\int_0^\pi\frac{dx}{1+\theta\cos x}$。

       * 万能代换（universal substitution）：$t=\tan\dfrac x2$ 把 $J$ 化成 arctan 型积分，$J=\dfrac{\pi}{\sqrt{1-\theta^2}}$，故 $I'(\theta)=\dfrac{\pi}{\theta}-\dfrac{\pi}{\theta\sqrt{1-\theta^2}}$。

       * 积回：$\pi\ln|\theta|$ 与 $-\pi\ln|\theta|$ 正好抵消，$I(\theta)=\pi\ln\bigl(1+\sqrt{1-\theta^2}\bigr)+C$。

       * 定常数：$I(0)=0$ 给 $C=-\pi\ln2$，故 $\displaystyle I(\theta)=\pi\ln\frac{1+\sqrt{1-\theta^2}}{2}$（$|\theta|<1$）。

       * > **提示**：拆项同除了 $\theta$，只在 $\theta\neq0$ 时合法；$\theta=0$ 时原积分就是 $\int_0^\pi\ln1\,dx=0$，本来就平凡，定常数一步把它接上。

       * > **注意**：链条上没有一步可省——最容易被跳过的"验条件"是整条链的门票；求导再积回会丢一个常数，必须拿特殊值抓回来。

---

## 含参反常积分一致收敛的概念

   * **收敛点（point of convergence）**

       * > **定义**：设 $f(x,y)$ 定义在 $[a,+\infty)\times[c,d]$ 上，若 $\displaystyle\int_a^{+\infty}f(x,y_0)\,dx$ 收敛，就称 $y_0$ 为该含参变量反常积分的一个收敛点。

       * 全体收敛点记为 $E\subseteq[c,d]$；$I(y)=\displaystyle\int_a^{+\infty}f(x,y)\,dx$ 是定义在 $E$ 上的函数——含参反常积分定义了收敛域上的一个函数。

   * **点态收敛不够用**

       * 类比：取定 $y_0$ 后积分收敛，相当于函数项级数的点态收敛；函数项级数的经验是点态收敛保证不了和函数的连续、可积、可导。

       * 结论：要讨论 $I(y)$ 的分析性质、要让极限与求导跟积分交换次序，点态收敛不行，需要一致收敛。

   * **一致收敛的定义（无限区间）**

       * > **定义**：若对任意 $y\in[c,d]$ 积分都收敛，且对任意 $\varepsilon>0$ 存在 $A_0>a$（$A_0$ 与 $y$ 无关），使得对一切 $A>A_0$ 与一切 $y\in[c,d]$ 都有 $\left|\displaystyle\int_A^{+\infty}f(x,y)\,dx\right|<\varepsilon$，则称其关于 $y$ 在 $[c,d]$ 上**一致收敛**（uniform convergence）。

       * 本质：积分收敛就是一次函数极限收敛 $\lim\limits_{A\to+\infty}\int_a^A f\,dx$；"一致"要求这一极限过程关于 $y$ 一致。

       * 要害：$\varepsilon$ 给定后找出的 $A_0$ 必须对一切 $y\in[c,d]$ 同时管用，不是每个 $y$ 各配各的。

   * **非一致收敛的定义**

       * 生成方式：把一致收敛逐句否定——"任意 $\varepsilon$"换"存在 $\varepsilon_0$"，"存在 $A_0$"换"任意 $A_0$"，"一切 $y$"换"某个 $y_0$"；证非一致收敛照这个否定结构去凑。

| 命题 | 量词链 | 直观含义 |
| :--- | :--- | :--- |
| 一致收敛 | $\forall\varepsilon>0,\ \exists A_0,\ \forall A>A_0,\ \forall y\in[c,d]$ | 一个 $A_0$ 同时管住所有 $y$ |
| 非一致收敛 | $\exists\varepsilon_0>0,\ \forall A_0,\ \exists A>A_0,\ \exists y_0\in[c,d]$ | 无论截到多远，总有某个 $y_0$ 掉队 |

   * **无界函数情形**

       * 设 $f(x,y)$ 定义在 $[a,b)\times[c,d]$ 上、$b$ 为瑕点：对任意 $\varepsilon>0$ 存在 $\delta>0$，对一切满足 $0<b-A<\delta$ 的 $A$ 与一切 $y\in[c,d]$，$\left|\displaystyle\int_A^b f(x,y)\,dx\right|<\varepsilon$，则称其关于 $y$ 在 $[c,d]$ 上一致收敛。

       * 对照：无限区间用 $A_0$ 控制尾巴，瑕点情形用 $\delta$ 控制瑕点左侧小区间——两个版本结构完全平行，非一致收敛版本照否定套路写出即可。

---

## 一致收敛的判别法

   * **柯西收敛原理（Cauchy convergence criterion）**

       * 地位：一致收敛的充分必要条件，一切判别法的出发点——只有它把"收敛"这个整体性质翻译成可动手验证的不等式，别的定理都只站在单边上。

       * 表述：对任意 $\varepsilon>0$，存在 $A_0>a$，对任意 $A',A''>A_0$ 与任意 $y\in[c,d]$，$\left|\displaystyle\int_{A'}^{A''}f(x,y)\,dx\right|<\varepsilon$；两个部分积分之差恰好接成中间区间上的积分。

       * > **注意**：$A_0$ 要大于下限 $a$——$A',A''$ 都得落在真正的"尾巴"上。

   * **判非一致收敛的两个推论**

       * 推论一（否定形式）：若存在 $\varepsilon_0>0$，对任意 $A_0$ 总存在 $A',A''>A_0$ 与 $y_0\in[c,d]$ 使 $\left|\displaystyle\int_{A'}^{A''}f(x,y_0)\,dx\right|\ge\varepsilon_0$，则关于 $y\in[c,d]$ 非一致收敛。

       * 推论二（序列形式，实战主力）：若存在 $A_n'\to+\infty$、$A_n''\to+\infty$ 与 $y_n\in[c,d]$，使数列 $\left|\displaystyle\int_{A_n'}^{A_n''}f(x,y_n)\,dx\right|$ 不趋于零，则非一致收敛。

       * 推论二的便宜处：$\varepsilon_0$ 不用凑——数列不趋零就有非零极限（甚至无穷大）的子列，$\varepsilon_0$ 对那个子列必然存在。

   * **魏尔斯特拉斯判别法（Weierstrass test，M 判别法）**

       * 条件：存在与参数无关的优函数（majorant）$F(x)$，使 $|f(x,y)|\le F(x)$ 对一切 $x\in[a,+\infty)$、一切 $y\in[c,d]$ 成立，且 $\displaystyle\int_a^{+\infty}F(x)\,dx$ 收敛（普通反常积分，不含参变量）。

       * 结论：$\displaystyle\int_a^{+\infty}f(x,y)\,dx$ 关于 $y\in[c,d]$ 一致收敛。

       * 机理：$\left|\int_{A'}^{A''}f\,dx\right|\le\int_{A'}^{A''}F\,dx<\varepsilon$——放缩全程 $y$ 不露面，优函数把每一个 $y$ 一起罩住。

   * **阿贝尔判别法（Abel test，参数版）**

       * 三条件：$\displaystyle\int_a^{+\infty}f(x,y)\,dx$ 关于 $y\in[c,d]$ 一致收敛；$g(x,y)$ 关于 $x$ 单调（固定每个 $y$ 看 $x$）；$|g(x,y)|\le L$ 对一切 $x$、$y$ 成立（一致有界，不再是个别 $y$ 底下的有界）。

       * 结论：$\displaystyle\int_a^{+\infty}f(x,y)\,g(x,y)\,dx$ 关于 $y\in[c,d]$ 一致收敛。

       * 来源：不带参数的版本是"$\int f$ 收敛 + $g$ 单调有界"；多了参数 $y$ 后，收敛升级为一致收敛、有界升级为一致有界。

   * **狄利克雷判别法（Dirichlet test，参数版）**

       * 三条件：$\left|\displaystyle\int_a^A f(x,y)\,dx\right|\le L$ 对一切 $A$、一切 $y\in[c,d]$ 成立（部分积分既关于 $A$ 又关于 $y$ 一致有界）；$g(x,y)$ 关于 $x$ 单调；$x\to+\infty$ 时 $g(x,y)$ 关于 $y\in[c,d]$ 一致趋于零。

       * 结论：同上，一致收敛。

       * 来源：不带参数的版本是"$\left|\int_a^A f\,dx\right|$ 有界 + $g$ 单调且趋于 $0$"；趋零升级为一致趋零。

       * 对照："$g$ 关于 $x$ 单调"两边共用；差别全在 $f$ 与 $g$ 各自强到什么程度——阿贝尔靠 $f$ 一致收敛，狄利克雷只要求部分积分一致有界、但压给 $g$ 一致趋零。

       * 证明引擎：第二积分中值定理（second mean value theorem for integrals）——固定 $y$，$g$ 关于 $x$ 单调才能用；阿贝尔放缩出 $2L\varepsilon$，狄利克雷放缩出 $4L\varepsilon$，$\varepsilon$ 乘常数照样要多小有多小。

   * 判别法地图：

    ```text
    柯西收敛原理（一致收敛的充要条件，一切判别法的出发点）
    ├── 正着用 → 一致收敛的充分条件
    │   ├── Weierstrass 判别法：找优函数 F(x) 压住 |f(x,y)|
    │   └── 阿贝尔 / 狄利克雷判别法：f、g 分开给条件，靠第二积分中值定理
    └── 反着用 → 非一致收敛
        ├── 推论一：否定命题（要凑出 ε₀）
        └── 推论二：找 A'_n→+∞、A''_n→+∞、y_n，让那一段积分不趋于零
    ```

---

## 判别法的正用与反用

   * **$e^{-\alpha x}$ 在两个区间上的对照**

       * 一致：$\displaystyle\int_0^{+\infty}e^{-\alpha x}\,dx$ 关于 $\alpha$ 在 $[\alpha_0,+\infty)$（$\alpha_0>0$）上一致收敛——尾积分 $\dfrac{1}{\alpha}e^{-\alpha A}$ 关于 $\alpha$ 单调递减，$\le\dfrac{1}{\alpha_0}e^{-\alpha_0 A}\to0$，$A_0$ 由 $\varepsilon$ 与取定的 $\alpha_0$ 决定、与 $\alpha$ 无关。

       * 非一致：同一积分在 $(0,+\infty)$ 上非一致收敛——每个 $\alpha>0$ 处都收敛（点态收敛处处成立），但任取固定 $A$，$\alpha\to0^+$ 时尾积分 $\dfrac{1}{\alpha}e^{-\alpha A}\to+\infty$；取 $\varepsilon_0=1$，对任意 $A_0$ 取 $A>A_0$ 再配使尾积分 $\ge1$ 的 $\alpha_A$ 即可。

       * > **易错点**：同一个积分在 $[\alpha_0,+\infty)$ 上一致、在 $(0,+\infty)$ 上非一致，差别全在参数范围——$\alpha_0>0$ 把参数与 $0$ 隔开一段距离才找得出公共的 $A_0$；允许 $\alpha$ 无限逼近 $0$，再大的 $A_0$ 也同时管不住所有 $\alpha$。点点收敛与一致收敛是两回事。⚠️

   * **优函数压住与抽掉分母的对照**

       * 一致：$\displaystyle\int_0^{+\infty}\frac{e^{-\alpha x}}{1+x^2}\,dx$ 关于 $\alpha$ 在 $[0,+\infty)$ 上一致收敛——$e^{-\alpha x}\le1$，优函数 $\dfrac{1}{1+x^2}$ 可积，Weierstrass 判别法一步到位（分子不超 $1$、分母带 $x^2$，一头一尾把条件喂齐）。

       * 非一致：$\displaystyle\int_0^{+\infty}\frac{e^{-\alpha x}}{1+x}\,dx$（$\alpha>0$，$\alpha=0$ 时成 $\dfrac{1}{1+x}$ 本身发散）关于 $\alpha$ 在 $(0,+\infty)$ 上非一致收敛——取 $A_n'=n$、$A_n''=2n$、$\alpha_n=\dfrac1n$，区间上 $e^{-x/n}\ge e^{-2}$，$\displaystyle\int_n^{2n}\frac{e^{-x/n}}{1+x}\,dx\ge e^{-2}\ln\frac{2n+1}{n+1}\to e^{-2}\ln2\ne0$。

       * > **易错点**：判一致收敛全程"放大"——把 $|f|$ 压到优函数底下，越小越好控制；证非一致收敛要证那段积分压不到零，只能缩小找下界，一放大结论就没了。⚠️

   * **$\dfrac{\sin x}{x}$ 带不带 $e^{-\alpha x}$ 的对照**

       * 一致：$\displaystyle\int_0^{+\infty}e^{-\alpha x}\frac{\sin x}{x}\,dx$ 关于 $\alpha$ 在 $[0,+\infty)$ 上一致收敛——拆成 $e^{-\alpha x}$ 与 $\dfrac{\sin x}{x}$：后者不含 $\alpha$，其收敛（狄利克雷判别法的老结果）自动一致；前者关于 $x$ 单调递减且 $0\le e^{-\alpha x}\le1$ 一致有界，阿贝尔判别法收工。

       * > **提示**："关于 $x$ 单调"是对每个固定参数说的，$\alpha=0$ 时退化为常数 $1$，常数也算单调——参数区间带着端点 $0$ 照样过判别法，不要下意识把 $0$ 挖掉。

       * 非一致：$\displaystyle\int_0^{+\infty}e^{-\alpha x}\sin x\,dx$ 关于 $\alpha$ 在 $(0,+\infty)$ 上非一致收敛（$\alpha=0$ 时 $\int_0^{+\infty}\sin x\,dx$ 发散，$0$ 必须挖掉）——取 $a_n'=n\pi$、$a_n''=(n+1)\pi$（$\sin x$ 在每个小区间上不变号）、$\alpha_n=\dfrac{1}{n+1}$，$e^{-\frac{x}{n+1}}$ 的最小值恰是干净的 $e^{-\pi}$，积分绝对值 $\ge2e^{-\pi}\not\to0$。

       * > **提示**：参数点 $\dfrac1{n+1}$ 是调出来的——先取 $\dfrac1n$ 会拖出 $e^{-\frac{(n+1)\pi}{n}}$ 的尾巴，换成 $\dfrac1{n+1}$ 最小值恰好落在 $\pi$ 的整倍数上；判非一致时参数点列总往出毛病的地方送（这里是 $0$ 附近），区间端点跟着被积函数的周期性取 $\pi$ 的整倍数。

   * **$\dfrac{\sin xy}{x}$ 的两个命题**

       * 一致：$\displaystyle\int_0^{+\infty}\frac{\sin xy}{x}\,dx$ 关于 $y$ 在 $[y_0,+\infty)$（$y_0>0$）上一致收敛——拆成 $\sin(xy)\cdot\dfrac1x$：$\dfrac1x\to0$ 与 $y$ 无关（一致），$\left|\displaystyle\int_0^A\sin(xy)\,dx\right|=\dfrac{|1-\cos(Ay)|}{y}\le\dfrac{2}{y}\le\dfrac{2}{y_0}$ 一致有界，狄利克雷判别法。

       * 非一致：关于 $y$ 在 $(0,+\infty)$ 上非一致收敛——取 $a_n'=n\pi$、$a_n''=\dfrac{3n\pi}{2}$、$y_n=\dfrac1n$，令 $t=\dfrac xn$（上下限一起除以 $n$）后积分 $=\displaystyle\int_\pi^{3\pi/2}\frac{\sin t}{t}\,dt$，$n$ 被约得一干二净，是个非零常数。

   * **$x^p\cos x^2$ 的内闭一致收敛**

       * > **定义**：内闭一致收敛——对任意闭区间 $[p_0,p_1]\subset(-1,1)$（$-1<p_0\le p\le p_1<1$），积分关于 $p$ 在 $[p_0,p_1]$ 上一致收敛，闭区间一个不漏。

       * 拆分：$p<0$ 时 $x^p$ 在 $x=0$ 处无界，$0$ 可能是瑕点，故拆成 $I_1=\displaystyle\int_0^1$（无界函数反常积分）与 $I_2=\displaystyle\int_1^{+\infty}$（无限区间反常积分）分头处理，两块都一致收敛原积分才一致收敛。

| 拆出的积分 | 自变量范围 | 放大时 $p$ 换成 | 控制手段 | 判别法 |
| :--- | :--- | :--- | :--- | :--- |
| $I_1=\int_0^1 x^p\cos x^2\,dx$ | $0<x<1$：指数越大幂越小 | $p_0$ | $\int_0^1 x^{p_0}dx$ 收敛（$p_0>-1$） | M 判别法 |
| $I_2=\int_1^{+\infty}x^p\cos x^2\,dx$ | $t\ge1$：指数越大幂越大 | $p_1$ | $\dfrac{1}{2t^{\frac{1-p_1}{2}}}$ 一致趋于零 | 狄利克雷判别法 |

       * $I_2$ 先换元 $t=x^2$ 把 $\cos x^2$ 捋直：$I_2=\dfrac12\displaystyle\int_1^{+\infty}\frac{\cos t}{t^{\frac{1-p}{2}}}\,dt$，$\left|\int_1^A\cos t\,dt\right|\le2$ 一致有界，$\dfrac{1}{2t^{\frac{1-p}{2}}}\le\dfrac{1}{2t^{\frac{1-p_1}{2}}}\to0$ 一致（$\dfrac{1-p_1}{2}>0$）。

       * > **易错点**："把 $p$ 换成 $p_0$ 还是 $p_1$"跟着自变量范围翻面：$0<x<1$ 时指数越大幂越小，放大换小的 $p_0$；$t>1$ 时指数越大幂越大，放大换大的 $p_1$。两半用的判别法也不同——瑕积分靠优积分压，无穷限靠狄利克雷。⚠️

---

## 由连续性反推一致收敛

   * **定理（定号 + 连续推出一致收敛）**

       * > **定义**：设 $f(x,y)$ 在 $[a,+\infty)\times[c,d]$ 上连续且保持定号（如 $f\ge0$），若 $I(y)=\displaystyle\int_a^{+\infty}f(x,y)\,dx$ 在 $[c,d]$ 上连续，则该含参变量反常积分关于 $y$ 在 $[c,d]$ 上一致收敛。

       * 逻辑方向：$f$ 连续、积分点态收敛、$I(y)$ 连续——光这些保证不了一致收敛；加上"$f$ 保持定号"这一条，一致收敛就自动到位。

       * 对应：函数项级数里同款结果——$u_n(x)$ 连续、级数点态收敛、和函数连续，一般推不出一致收敛；各项保持定号（如非负）时结论成立。

   * **反证法的五步链条**

       * 第一步翻译否定式：存在 $\varepsilon_0>0$，对任意 $N>a$ 存在 $y_N\in[c,d]$ 使 $\displaystyle\int_N^{+\infty}f(x,y_N)\,dx\ge\varepsilon_0$（$f\ge0$ 免绝对值；严格的否定式里还有 $A>N$，把下限从 $A$ 挪回 $N$ 积分只会更大，简化合法）。

       * 第二步抽子列：$y_N$ 全落在闭区间 $[c,d]$ 里，必有收敛子列，极限 $y_0\in[c,d]$（闭区间跑不出去）；不妨设 $y_N\to y_0$。

       * 第三步造小尾巴：$y_0$ 处积分收敛，存在 $A$ 使 $\displaystyle\int_A^{+\infty}f(x,y_0)\,dx<\frac{\varepsilon_0}{2}$——关键在这个 $A$ 从此固定，而 $N$ 可以任意大。

       * 第四步两边顶牛：取 $N>A$，由 $f\ge0$ 得 $\displaystyle\int_A^{+\infty}f(x,y_N)\,dx\ge\int_N^{+\infty}f(x,y_N)\,dx\ge\varepsilon_0$；再把极限送进去需要 $J_A(y)=\displaystyle\int_A^{+\infty}f(x,y)\,dx$ 关于 $y$ 连续。

       * 第五步收网：取极限得 $\displaystyle\int_A^{+\infty}f(x,y_0)\,dx\ge\varepsilon_0$，与第三步的 $<\dfrac{\varepsilon_0}{2}$ 完全顶死，矛盾。

       * $J_A(y)$ 连续的来历：$J_A(y)=I(y)-\displaystyle\int_a^A f\,dx$——前者是定理条件给的连续函数，后者是含参常义积分（$f$ 二元连续）也连续；两个条件各司其职："$f$ 保持定号"负责丢绝对值、挪下限保不等号，"$I(y)$ 连续"加"常义含参积分连续"负责把极限真正送到 $y_0$ 手里。

       * 反证链条：

    ```text
    反证：假设关于 y 非一致收敛
      ├─ ∃ε₀>0：∀N>a，∃y_N∈[c,d]，使 ∫_N^{+∞} f(x,y_N)dx ≥ ε₀
      ├─ {y_N}⊂[c,d] 有收敛子列，极限 y₀∈[c,d]（不妨设 y_N→y₀）
      ├─ y₀ 处积分收敛 ⇒ ∃固定 A：∫_A^{+∞} f(x,y₀)dx < ε₀/2
      ├─ 取 N>A：∫_A^{+∞} f(x,y_N)dx ≥ ∫_N^{+∞} f(x,y_N)dx ≥ ε₀
      └─ J_A(y)=∫_A^{+∞}f(x,y)dx 关于 y 连续 ⇒ 极限号进去
            ⇒ ∫_A^{+∞} f(x,y₀)dx ≥ ε₀，与 < ε₀/2 矛盾
    ```

---

## 一致收敛下的运算交换

   * **切段化级数的路线**

       * 取分点 $a_0=a<a_1<a_2<\cdots$，$a_n\to+\infty$，把 $[a,+\infty)$ 切成可列段，令 $u_n(y)=\displaystyle\int_{a_{n-1}}^{a_n}F(x,y)\,dx$——每段是含参常义积分，$F$ 连续则每个 $u_n(y)$ 连续。

       * 前提：含参反常积分本身收敛，才有 $I(y)=\displaystyle\int_a^{+\infty}F\,dx=\sum_{n=1}^\infty u_n(y)$ 可谈。

       * 引理：若反常积分关于 $y$ 在 $[c,d]$ 上一致收敛，则级数 $\sum u_n(y)$ 关于 $y$ 也一致收敛——对级数尾段 $\sum_{k=n+1}^m u_k=\displaystyle\int_{a_n}^{a_m}F\,dx$ 套柯西表述，取 $N$ 使 $a_N>A_0$ 即可。

       * 效果：函数项级数一章的连续性、逐项积分、逐项求导三件套，经这条引理整套平移给 $I(y)$。

    ```text
    含参反常积分 I(y) = ∫[a,+∞) F(x,y) dx
          │ 取分点 a0=a<a1<a2<…，an→+∞，切成可列段
          ▼
    I(y) = Σ u_n(y)，其中 u_n(y) = ∫[a_{n-1}, a_n] F(x,y) dx
          │
          │ 引理：反常积分关于 y 一致收敛 ⟹ 级数关于 y 一致收敛
          ▼
    函数项级数 Σ u_n(y)
          └─ 连续性 / 逐项积分 / 逐项求导：级数一章的结论整套平移过来
    ```

   * **连续性定理**

       * 条件：$F$ 在 $[a,+\infty)\times[c,d]$ 上连续，且积分关于 $y$ 在 $[c,d]$ 上一致收敛。

       * 结论：$I(y)$ 在 $[c,d]$ 上连续，即 $\displaystyle\lim_{y\to y_0}\int_a^{+\infty}F(x,y)\,dx=\int_a^{+\infty}\lim_{y\to y_0}F(x,y)\,dx$——极限号与积分号交换。

   * **积分次序交换定理**

       * 条件：同上（$F$ 连续 + 一致收敛）。

       * 结论：$I(y)$ 在 $[c,d]$ 上可积，且 $\displaystyle\int_c^d\mathrm{d}y\int_a^{+\infty}F\,dx=\int_a^{+\infty}\mathrm{d}x\int_c^d F\,dy$——左边先对 $x$（无穷限反常积分）后对 $y$，右边先对 $y$（有限区间）后对 $x$。

       * 证明骨架：逐项积分把 $\int_c^d I\,dy$ 化成 $\sum\int_c^d u_n\,dy$，内层是有限段上的常义含参积分、次序本来可换，最后一段段拼回无穷区间。

   * **推广的积分次序交换定理**

       * 场景：两个方向都反常，$F$ 在 $[a,+\infty)\times[c,+\infty)$ 上连续。

       * 三条件：$\displaystyle\int_a^{+\infty}F\,dx$ 关于 $y$ 在任意 $[c,C]$ 上一致收敛；$\displaystyle\int_c^{+\infty}F\,dy$ 关于 $x$ 在任意 $[a,a']$ 上一致收敛；两个带绝对值的累次积分 $\displaystyle\int_a^{+\infty}dx\int_c^{+\infty}|F|\,dy$ 与 $\displaystyle\int_c^{+\infty}dy\int_a^{+\infty}|F|\,dx$ 至少有一个收敛。

       * 结论：$\displaystyle\int_a^{+\infty}dx\int_c^{+\infty}F\,dy=\int_c^{+\infty}dy\int_a^{+\infty}F\,dx$；两个方向的一致收敛都退成了内闭要求，另加绝对值累次积分兜底，完整证明见菲赫金哥尔茨《微积分学教程》，叙述会用即可。

   * **积分号下求导定理**

       * 条件（与逐项求导定理逐条对应）：$F$ 与 $\dfrac{\partial F}{\partial y}$ 在 $[a,+\infty)\times[c,d]$ 上连续；$\displaystyle\int_a^{+\infty}F\,dx$ 对每个 $y\in[c,d]$ 收敛；$\displaystyle\int_a^{+\infty}\dfrac{\partial F}{\partial y}\,dx$ 关于 $y$ 在 $[c,d]$ 上一致收敛。

       * 结论：$I(y)$ 在 $[c,d]$ 上可导，且 $I'(y)=\displaystyle\int_a^{+\infty}\frac{\partial F}{\partial y}(x,y)\,dx$，求导符号钻进积分号。

       * > **易错点**：三条里最容易记错的是中间那条——原积分只要求**点态**收敛，并没有要求一致收敛；一致收敛的条件只加在偏导数的积分上。⚠️

       * 证明骨架：记 $\varphi(y)=\displaystyle\int_a^{+\infty}\frac{\partial F}{\partial y}\,dx$，由连续性定理 $\varphi$ 在 $[c,d]$ 上连续；在 $[c,y]\subseteq[c,d]$ 上换序得 $\displaystyle\int_c^y\varphi(t)\,dt=\int_a^{+\infty}\bigl[F(x,y)-F(x,c)\bigr]dx=I(y)-I(c)$；$\varphi$ 连续保证变上限积分可导，两边对 $y$ 求导即得结论。

   * **连续范围的求法：先定义域后内闭**

       * 例：确定 $I(\alpha)=\displaystyle\int_0^{+\infty}\frac{\ln(1+x^3)}{x^{\alpha}}\,dx$ 的连续范围——函数在哪里有定义都没搞清楚，谈何连续。

       * 定义域：区间无限且 $x=0$ 可能是瑕点，以 $1$ 为界拆两段——$I_1$（瑕积分）当 $\alpha<4$ 收敛（$x\to0^+$ 时被积函数等价于 $\dfrac{1}{x^{\alpha-3}}$，$p$ 积分判据），$I_2$（无穷限）当 $\alpha>1$ 收敛（对数无穷大量在幂函数面前不起作用，幂次说了算）；交集 $1<\alpha<4$。

       * 连续范围：定义域是开区间、两端点处积分恰好发散，整体一致收敛无从谈起——只需内闭一致收敛（连续是逐点性质，开区间里每个点都装得进某个闭子区间）；取任意 $[A,B]\subset(1,4)$，$I_1$ 用控制函数 $\dfrac{\ln(1+x^3)}{x^{B}}$（$x<1$ 时指数越大幂越小，放大分式把指数放大到 $B$），$I_2$ 用 $\dfrac{\ln(1+x^3)}{x^{A}}$（$x>1$ 时指数越小幂越小，指数缩小到 $A$），两个控制函数分别等价于 $x^{3-B}$（$3-B>-1$）与幂次 $A>1$ 的可积函数，M 判别法各自生效。

       * 结果：$I(\alpha)$ 在整个定义域 $(1,4)$ 上连续——路线固定：先定定义域（各段收敛条件取交集），再定连续范围（内闭一致收敛 + 连续性定理）。

---

## 经典反常积分战例

   * **狄利克雷积分（Dirichlet integral）**

       * 目标：$I=\displaystyle\int_0^{+\infty}\frac{\sin x}{x}\,dx$——收敛（狄利克雷判别法）但 $\dfrac{\sin x}{x}$ 的原函数不是初等函数，老办法积不出。

       * 造参：令 $I(\alpha)=\displaystyle\int_0^{+\infty}e^{-\alpha x}\frac{\sin x}{x}\,dx$（$\alpha\ge0$），乘进的 $e^{-\alpha x}$ 称收敛因子（convergence factor），作用是提高收敛性——$x\to+\infty$ 时把被积函数压得非常小，$\alpha=0$ 时退化为 $1$ 不起作用。

       * 连续铺垫：被积函数在 $x=0$ 处补值为 $1$（接住 $\lim\limits_{x\to0^+}\dfrac{\sin x}{x}=1$）后在 $[0,+\infty)\times[0,+\infty)$ 上连续；积分关于 $\alpha$ 在 $[0,+\infty)$ 一致收敛，故 $I(\alpha)$ 在 $[0,+\infty)$ 连续，$I=I(0)=\lim\limits_{\alpha\to0^+}I(\alpha)$——$\alpha=0$ 那点正是原题困难所在，只能用 $\alpha>0$ 的表达式取极限。

       * 求导进积分号：需要的是**偏导之后那个积分** $\displaystyle\int_0^{+\infty}e^{-\alpha x}\sin x\,dx$ 的一致收敛——一求导出来的 $x$ 正好抵消分母的 $\dfrac1x$，剩下好积的 $e^{-\alpha x}\sin x$；它在 $(0,+\infty)$ 非一致（$\alpha=0$ 处 $\int\sin x\,dx$ 发散），但在任意 $[\alpha_0,+\infty)$ 上有优函数 $e^{-\alpha_0x}$（$\left|e^{-\alpha x}\sin x\right|\le e^{-\alpha_0 x}$，$\int_0^{+\infty}e^{-\alpha_0x}dx=\dfrac{1}{\alpha_0}$）一致收敛；$\alpha_0$ 任意，故 $I'(\alpha)=-\displaystyle\int_0^{+\infty}e^{-\alpha x}\sin x\,dx=-\frac{1}{1+\alpha^2}$ 对一切 $\alpha>0$ 成立。

       * > **易错点**：积分号下求导要验的是求了偏导之后那个积分的一致收敛，不是原积分的——两者不是一回事，而前者偏偏在含 $0$ 的开区间上不成立，只能在每个 $[\alpha_0,+\infty)$ 上证、再靠 $\alpha_0$ 的任意性铺满 $(0,+\infty)$。

       * 积回并定常数：$I(\alpha)=-\arctan\alpha+C$；由 $|\sin x|\le x$ 得 $|I(\alpha)|\le\displaystyle\int_0^{+\infty}e^{-\alpha x}dx=\frac1\alpha\to0$（$\alpha\to+\infty$），与表达式右端 $-\dfrac\pi2+C$ 两头一对，$C=\dfrac\pi2$。

       * 收网：$I=I(0)=\lim\limits_{\alpha\to0^+}\Bigl(\dfrac\pi2-\arctan\alpha\Bigr)=\dfrac\pi2$。

       * 推论（符号函数）：$\displaystyle\int_0^{+\infty}\frac{\sin xt}{t}\,dt=\frac{\pi}{2}\operatorname{sgn}x$——$a>0$ 时换元 $t=ax$ 上下限不变仍得 $\dfrac\pi2$；$a=0$ 被积函数恒为零；$a<0$ 前面多出负号得 $-\dfrac\pi2$。

       * > **提示**：连续性、可导性这类结论，参数区间是无限的根本不碍事；真正对区间有要求的只有积分次序交换——那边一个是 $\int_c^d$、一个是 $\int_c^{+\infty}$，条件不一样。

   * **指数平方乘余弦**

       * 目标：$I(x)=\displaystyle\int_0^{+\infty}e^{-t^2}\cos 2xt\,dt$，参变量 $x$；已知普阿松积分（Poisson integral）$\displaystyle\int_0^{+\infty}e^{-t^2}dt=\frac{\sqrt\pi}{2}$，且 $I(0)$ 恰为它。

       * 合法性：$\left|e^{-t^2}\cos2xt\right|\le e^{-t^2}$ 可积，关于 $x\in(-\infty,+\infty)$ 一致收敛，$I(x)$ 连续；求偏导后 $\left|-2te^{-t^2}\sin2xt\right|\le2te^{-t^2}$ 可积，求导可进积分号。

       * 微分方程：$I'(x)=-2\displaystyle\int_0^{+\infty}te^{-t^2}\sin2xt\,dt$，注意 $-2te^{-t^2}dt=d\bigl(e^{-t^2}\bigr)$，分部积分后边界项归零（$t\to+\infty$ 处 $e^{-t^2}\to0$、$t=0$ 处 $\sin0=0$），出来的是 $I'(x)=-2x\,I(x)$。

       * 求解：$\bigl(\ln I(x)\bigr)'=-2x$ 给 $I(x)=Ce^{-x^2}$，代 $I(0)=\dfrac{\sqrt\pi}{2}$ 定出 $C$，故 $\displaystyle I(x)=\frac{\sqrt\pi}{2}\,e^{-x^2}$。

       * 套路沉淀：求这类含参反常积分——求导进积分号，转出一个含 $I'$ 与 $I$ 的关系式（简单的微分方程），解出再定常数；全程的门槛始终是"求了偏导之后关于参变量是否一致收敛"。

   * **余弦差除以平方**

       * 目标：$I=\displaystyle\int_0^{+\infty}\frac{\cos ax-\cos bx}{x^2}\,dx$（$b>a>0$），直接求原函数不好求。

       * 分子的来历：$\displaystyle\int_a^b\sin(xy)\,dy=\left[-\frac{\cos(xy)}{x}\right]_{y=a}^{y=b}=\frac{\cos ax-\cos bx}{x}$，于是 $I=\displaystyle\int_0^{+\infty}dx\int_a^b\frac{\sin(xy)}{x}\,dy$。

       * 换序依据：$\displaystyle\int_0^{+\infty}\frac{\sin(xy)}{x}\,dx$ 关于 $y$ 在 $[a,b]$ 上一致收敛（前款已证），积分次序可交换。

       * 内层现成：$y\in[a,b]$ 时 $y>0$，$\displaystyle\int_0^{+\infty}\frac{\sin(xy)}{x}\,dx=\frac\pi2$，故 $I=\displaystyle\int_a^b\frac\pi2\,dy=\frac\pi2(b-a)$。

| 战例 | 积分 | 用哪件工具 | 结果 |
| :--- | :--- | :--- | :--- |
| 狄利克雷积分 | $\int_0^{+\infty}\dfrac{\sin x}{x}\,dx$ | 造收敛因子，积分号下求导 | $\dfrac{\pi}{2}$ |
| 指数平方乘余弦 | $\int_0^{+\infty}e^{-t^2}\cos 2xt\,dt$ | 积分号下求导，解微分方程 | $\dfrac{\sqrt\pi}{2}e^{-x^2}$ |
| 余弦差 | $\int_0^{+\infty}\dfrac{\cos ax-\cos bx}{x^2}\,dx$ | 积分次序交换 | $\dfrac{\pi}{2}(b-a)$ |

---

## Beta函数

   * **定义与定义域**

       * > **定义**：$B(p,q)=\displaystyle\int_0^1 x^{p-1}(1-x)^{q-1}\,dx$，称为 Beta 函数（Beta function），$p$、$q$ 是它的两个参变量。

       * 类型：区间 $[0,1]$ 有限但被积函数可能无界——$p-1<0$ 时 $x=0$ 是潜在瑕点，$q-1<0$ 时 $x=1$ 是潜在瑕点，两头都得盯。

       * 收敛判据：在 $x=\dfrac12$ 处劈成两半，$I_1$（奇点只可能在 $0$）当 $x\to0^+$ 时等价于 $x^{p-1}$、收敛 $\iff p>0$；$I_2$（奇点只可能在 $1$）当 $x\to1^-$ 时等价于 $(1-x)^{q-1}$、收敛 $\iff q>0$。

       * 结论：$B(p,q)$ 收敛 $\iff p>0$ 且 $q>0$——定义域是 $pq$ 平面第一象限的开区域，两条坐标轴 $p=0$、$q=0$ 都不在内。

   * **连续性**

       * 结论：$B(p,q)$ 关于 $p$、$q$ 在整个定义域 $p>0,\ q>0$ 上连续。

       * 手法：开区域上不能一口要一致收敛，任意取定 $p_0>0$、$q_0>0$ 退到角状区域 $\{p\ge p_0,\ q\ge q_0\}$——优函数 $x^{p_0-1}(1-x)^{q_0-1}$ 的积分恰是收敛的 $B(p_0,q_0)$，M 判别法给出该区域上一致收敛、从而连续；$p_0,q_0$ 任意，角状区域盖满定义域。

       * > **易错点**：不等号方向最容易绊倒人——$0<x<1$ 上指数越大幂值越小（如 $x^2<x$），$p-1\ge p_0-1$ 给出的是 $x^{p-1}\le x^{p_0-1}$，右边那个才是控制项。

   * **对称性**

       * 结论：$B(p,q)=B(q,p)$。

       * 证明：代换 $x=1-t$，$x^{p-1}$ 与 $(1-x)^{q-1}$ 交换座位，积分限翻回后即得——看着不起眼，它直接递出递推的姊妹公式。

   * **递推公式**

| 递推公式 | 成立条件 |
| :--- | :--- |
| $B(p,q)=\dfrac{q-1}{p+q-1}\,B(p,q-1)$ | $p>0,\ q>1$ |
| $B(p,q)=\dfrac{p-1}{p+q-1}\,B(p-1,q)$ | $p>1,\ q>0$ |
| $B(p,q)=\dfrac{(p-1)(q-1)}{(p+q-1)(p+q-2)}\,B(p-1,q-1)$ | $p>1,\ q>1$ |

       * > **易错点**：递推公式不是对整个定义域成立——右边站着 $B(p,q-1)$ 就要求 $q-1>0$；每条公式的适用范围都要看清楚，不能拿来就套。

       * 证明骨架（第一条）：把 $x^{p-1}dx=\dfrac1p\,d(x^p)$ 放进微分号，分部积分后边界项靠 $q>1$（$x=1$ 处 $(1-x)^{q-1}=0$）与 $p>0$（$x=0$ 处 $x^p=0$）消为零；再把多出来的 $x$ 按 $x=1-(1-x)$ 拆成 $B(p,q-1)-B(p,q)$，解出两边都含 $B(p,q)$ 的方程即得。

       * 第三条来历：前两条连用——先用第一条把 $q$ 降一格，再用第二条把 $p$ 降一格，$p+q$ 的和每递推一次掉 $1$，条件也相应收紧成 $p>1$、$q>1$。

   * **三角形式与白送的积分**

       * 公式：代换 $x=\cos^2\varphi$，多余的幂次被微分配平，$B(p,q)=2\displaystyle\int_0^{\pi/2}\cos^{2p-1}\varphi\sin^{2q-1}\varphi\,d\varphi$。

       * 白送一：取 $p=q=\dfrac12$，两个幂次全消，$B\!\left(\dfrac12,\dfrac12\right)=\pi$，即 $\displaystyle\int_0^1\frac{dx}{\sqrt{x(1-x)}}=\pi$——平时要硬算的根式积分从 B 函数里一眼读出。

       * 白送二：$\displaystyle\int_0^{\pi/2}\sin^nx\cos^mx\,dx=\frac12\,B\!\left(\frac{n+1}{2},\frac{m+1}{2}\right)$——让 $2p-1$、$2q-1$ 分别对上两个幂次解出参数，系数 $2$ 挪到右边成 $\dfrac12$；$B$ 对称，两参数谁前谁后无所谓。

   * **无穷区间形式**

       * 公式：代换 $x=\dfrac{1}{1+t}$（$x$ 从 $0$ 到 $1$ 时 $t$ 从 $+\infty$ 到 $0$），$B(p,q)=\displaystyle\int_0^{+\infty}\frac{t^{q-1}}{(1+t)^{p+q}}\,dt$。

       * > **易错点**：这个形式前面**没有**系数 $2$——跟三角形式对比着记，别记混。

       * 孪生形式：利用对称性 $B(p,q)=B(q,p)$，$B(p,q)=\displaystyle\int_0^{+\infty}\frac{t^{p-1}}{(1+t)^{p+q}}\,dt$。

       * 折回形式：把 $[0,+\infty)$ 拆成 $[0,1]$ 与 $[1,+\infty)$，后一段作代换 $t=\dfrac1u$，得 $B(p,q)=\displaystyle\int_0^1\frac{t^{p-1}+t^{q-1}}{(1+t)^{p+q}}\,dt$。

       * 用途：碰到"幂函数除以 $(1+t)$ 的幂"的无穷区间积分，或 $[0,1]$ 上分子为两幂相加的积分，往 B 函数上靠。

---

## Gamma函数

   * **定义与定义域**

       * > **定义**：$\Gamma(s)=\displaystyle\int_0^{+\infty}x^{s-1}e^{-x}\,dx$，称为 Gamma 函数（Gamma function），积分值跟 $s$ 有关。

       * 两处险情：区间无限；$s<1$ 时 $x^{s-1}$ 在 $x=0$ 附近无界。拆成 $\int_0^1$ 与 $\int_1^{+\infty}$ 两段。

       * 收敛判据：$I_2$ 段有 $e^{-x}$ 压阵，对一切 $s$ 收敛；$I_1$ 段 $e^{-x}$ 在 $(0,1]$ 上是有界因子，收敛与否全看 $x^{s-1}$，$\iff s>0$。

       * 结论：$\Gamma$ 的定义域是 $s>0$。

   * **连续与无穷可导**

       * 钥匙：定义域 $(0,+\infty)$ 是开区间，不需要整体一致收敛，只需内闭一致收敛——对任意 $[a,b]\subset(0,+\infty)$（$a>0$ 正是"内闭"的含义）证一致收敛。

       * 连续性：$I_1$ 段 $\left|x^{s-1}e^{-x}\right|\le x^{a-1}$（$x<1$ 时指数越小幂越大，$e^{-x}\le1$ 放掉），$I_2$ 段 $x^{s-1}e^{-x}\le x^{b-1}e^{-x}$——这一段 $e^{-x}$ 不能丢，收敛全靠它；两个控制函数都可积，M 判别法给出 $[a,b]$ 上一致收敛。

       * 可导性：对 $s$ 求导多出一个 $\ln x$；$I_1$ 段控制函数 $x^{a-1}|\ln x|$（$\ln x$ 在 $0$ 附近虽是无穷大量，从无穷小 $x^{a-1}$ 里拿一点出来就抵消）可积，$I_2$ 段 $e^{-x}$ 继续压阵；故 $\Gamma'(s)=\displaystyle\int_0^{+\infty}x^{s-1}e^{-x}\ln x\,dx$。

       * 推广：再求导只是 $\ln x$ 变 $(\ln x)^2$，一致收敛的论证一字不改照搬——$\Gamma$ 在 $(0,+\infty)$ 上无穷次可导，$\Gamma^{(n)}(s)=\displaystyle\int_0^{+\infty}x^{s-1}e^{-x}(\ln x)^n\,dx$（$n=0,1,2,\dots$）。

   * **递推公式与阶乘**

       * 公式：$\Gamma(s+1)=s\,\Gamma(s)$（$s>0$），伽马函数最重要的性质。

       * 证明：$\Gamma(s+1)=\int_0^{+\infty}x^se^{-x}dx$ 中把 $e^{-x}$ 放进微分号分部积分；边界项 $x^se^{-x}$ 在 $+\infty$ 处即 $\dfrac{x^s}{e^x}\to0$（指数压倒幂）、在 $0$ 处因 $s>0$ 为零，剩下 $s\displaystyle\int_0^{+\infty}x^{s-1}e^{-x}dx=s\,\Gamma(s)$。

       * 阶乘推广：$\Gamma(n+1)=n\,\Gamma(n)=\cdots=n!\cdot\Gamma(1)$，而 $\Gamma(1)=\displaystyle\int_0^{+\infty}e^{-x}dx=1$，故 $\Gamma(n+1)=n!$。

       * > **提示**：错位关系——$\Gamma(n+1)=n!$，等价地正整数 $s$ 处 $\Gamma(s)=(s-1)!$，别把 $\Gamma(n)$ 直接当成 $n!$。

       * 意义：阶乘本来只对正整数有定义，$\Gamma$ 把"阶乘"推广到整个正实轴——$s$ 可以是分数，还可以是无理数。

   * **延拓到负半轴**

       * 引擎：递推公式改写成 $\Gamma(s)=\dfrac{\Gamma(s+1)}{s}$——左边要求 $s>0$，右边 $\Gamma(s+1)$ 只要求 $s>-1$、再除去分母的 $s=0$，右边在 $-1<s<0$ 上有意义，就用右边定义左边，$\Gamma$ 延拓到 $(-1,0)\cup(0,+\infty)$。

       * 推进：右边的 $\Gamma(s+1)$ 再用一次递推得 $\Gamma(s)=\dfrac{\Gamma(s+2)}{s(s+1)}$，又覆盖 $(-2,-1)$；一般地 $\Gamma(s)=\dfrac{\Gamma(s+k+1)}{s(s+1)\cdots(s+k)}$，最终定义到全实轴、挖去 $s=0,-1,-2,\dots$。

       * > **定义**：解析延拓（analytic continuation）的思想预演——一个函数初始定义的范围比较小，利用一条现成的关系式，把定义延伸到比原先大得多的范围；到复变函数里它是核心内容。

       * 瑕点：$s=-k$（$k=0,1,2,\dots$）都是延拓后 $\Gamma$ 的瑕点，函数在这些点附近趋于无穷大；无穷大的符号逐点、逐侧判断——$s\to0^+$ 时 $+\infty$、$s\to0^-$ 时 $-\infty$、$s\to-1^+$ 时 $-\infty$、$s\to-1^-$ 时 $+\infty$。

       * 终点：这些点上极限不存在，延拓推不动了，到此为止。

       * 推进路线：

    ```text
    Γ(s) 定义域的推进（每用一次 Γ(s) = Γ(s+1)/s，向左扩一格）：

      原始定义：                    (0, +∞)
      第一次延拓：              (-1, 0) ∪ (0, +∞)
      第二次延拓：      (-2, -1) ∪ (-1, 0) ∪ (0, +∞)
      ……
      最终：全实轴，挖去 s = 0, -1, -2, ……
    ```

   * **两种常用变形**

       * 平方代换：令 $x=t^2$，$\Gamma(s)=2\displaystyle\int_0^{+\infty}t^{2s-1}e^{-t^2}\,dt$——被积函数带 $e^{-t^2}$ 的积分都可以表示成 $\Gamma$。

       * 半整数值：取 $s=\dfrac12$，幂次 $2s-1=0$ 整个消失，$\Gamma\!\left(\dfrac12\right)=2\displaystyle\int_0^{+\infty}e^{-t^2}dt=\sqrt\pi$；递推一开动半整数点一个接一个全知道：$\Gamma\!\left(\dfrac32\right)=\dfrac{\sqrt\pi}{2}$，$\Gamma\!\left(\dfrac52\right)=\dfrac{3\sqrt\pi}{4}$，……

       * 尺度代换：令 $x=\alpha t$（$\alpha>0$），$\Gamma(s)=\alpha^s\displaystyle\int_0^{+\infty}t^{s-1}e^{-\alpha t}\,dt$。

   * **例：平方代换收编一个无穷积分**

       * 目标：$I=\displaystyle\int_0^{+\infty}x^{2n}e^{-x^2}\,dx$（$n$ 为自然数）。

       * 对照认形：盯住 $2\displaystyle\int_0^{+\infty}t^{2s-1}e^{-t^2}dt=\Gamma(s)$，$2s-1=2n$ 给 $s=\dfrac{2n+1}{2}$，$I=\dfrac12\Gamma\!\left(\dfrac{2n+1}{2}\right)$。

       * 递推剥层：每剥一层从幂次上下来一个 $\dfrac{2k-1}{2}$、同时给 $2$ 的幂次添一个因子，剥到 $\Gamma\!\left(\dfrac12\right)$ 为止，得 $I=\dfrac{(2n-1)!!}{2^{n+1}}\sqrt\pi$。

       * > **定义**：双阶乘（double factorial）$(2n-1)!!=(2n-1)(2n-3)\cdots3\cdot1$——隔一个乘一个。

---

## B与Γ的关系式

   * **主关系式**

       * 结论：$B(p,q)=\dfrac{\Gamma(p)\,\Gamma(q)}{\Gamma(p+q)}$（$p>0,\ q>0$）——本讲一切计算的引擎。

       * 思路：把乘积 $\Gamma(p)\Gamma(q)$ 化成一个二重积分，然后上极坐标。

       * 第一步：$\Gamma$ 用平方代换写成 $2\displaystyle\int_0^{+\infty}t^{2p-1}e^{-t^2}dt$；两个积分的变量不能重名，把其中一个改名 $s$，乘积是"每个 $s$ 配每个 $t$"，铺满第一象限 $\Omega=\{s>0,\ t>0\}$：$\Gamma(p)\Gamma(q)=4\iint_\Omega s^{2p-1}t^{2q-1}e^{-(s^2+t^2)}\,ds\,dt$。

       * 第二步：极坐标 $s=r\cos\theta$、$t=r\sin\theta$，$\theta$ 从 $0$ 到 $\dfrac\pi2$、$r$ 从 $0$ 到 $+\infty$；径向幂次合计 $2p+2q-1$，角向、径向各管各的，二重积分拆成两个单积分的乘积——角向那块恰是 $B$ 的三角形式 $\bigl(=\frac12B(p,q)\bigr)$，径向那块令 $u=r^2$ 后是 $\frac12\Gamma(p+q)$。

       * 配平：$4\cdot\dfrac12\cdot\dfrac12=1$，系数正好吃光，$\Gamma(p)\Gamma(q)=B(p,q)\,\Gamma(p+q)$。

       * > **易错点**：前面那个系数 $4=2\times2$ 最容易丢——随后被两个 $\dfrac12$ 各吃掉一个，账要一笔一笔对。

       * 手法沉淀：想把两个定积分的乘积变成二重积分，就给它们用不同的积分变量名；想凑出 $e^{-t^2}$，就在 $\Gamma$ 里令 $x=t^2$——整条证明是在 $B$ 与 $\Gamma$ 各自最舒服的表达式之间架桥。

   * **试刀：偶次幂三角积分**

       * 目标：$I=\displaystyle\int_0^{\pi/2}\sin^6x\cos^4x\,dx$——两个幂次都是偶数，定积分递推硬算相当繁琐。

       * 对照三角形式：$2p-1=6$ 给 $p=\dfrac72$，$2q-1=4$ 给 $q=\dfrac52$，$I=\dfrac12\,B\!\left(\dfrac72,\dfrac52\right)=\dfrac12\cdot\dfrac{\Gamma(7/2)\,\Gamma(5/2)}{\Gamma(6)}$。

       * 数值：$\Gamma(6)=5!=120$；沿递推链 $\Gamma\!\left(\dfrac52\right)=\dfrac34\sqrt\pi$、$\Gamma\!\left(\dfrac72\right)=\dfrac{15}{8}\sqrt\pi$；$I=\dfrac{3\pi}{512}$——原来要递推好几层的题两行收工。

   * **试刀：根式积分**

       * 目标：$I=\displaystyle\int_0^1x^8\sqrt{1-x^3}\,dx$。

       * 换元认形：令 $x^3=t$（$dx=\dfrac13t^{-2/3}dt$），幂次合并 $t^{8/3-2/3}=t^2$，$I=\dfrac13\displaystyle\int_0^1t^{2}(1-t)^{1/2}dt=\dfrac13\,B\!\left(3,\dfrac32\right)$。

       * 省力诀窍：分子里的 $\Gamma\!\left(\dfrac32\right)$ 不要急着算成数值——分母 $\Gamma\!\left(\dfrac92\right)=\dfrac72\cdot\dfrac52\cdot\dfrac32\,\Gamma\!\left(\dfrac32\right)$ 沿递推链降下来正好吐出一个，上下直接消掉；$I=\dfrac{16}{315}$。

   * **正弦幂的一般公式**

       * 公式：$\displaystyle\int_0^{\pi/2}\sin^\alpha x\,dx=\frac{\sqrt\pi}{2}\cdot\frac{\Gamma\!\left(\frac{\alpha+1}{2}\right)}{\Gamma\!\left(\frac{\alpha+2}{2}\right)}$，适用全体 $\alpha>-1$，$\alpha$ 不必是整数；$\cos^\alpha$ 的积分同值（换元 $x\to\dfrac\pi2-x$ 互变）。

       * 范围来历：$x\to0^+$ 时 $\sin^\alpha x$ 与 $x^\alpha$ 同阶，$\alpha\le-1$ 在零点发散；从 $B$ 函数那头看，识别出的参数 $q=\dfrac{\alpha+1}{2}$ 必须为正，恰好就是 $\alpha>-1$。

       * 识别参数：令 $\cos$ 的幂次为零，$2p-1=0$ 给 $p=\dfrac12$；$\sin$ 的幂次 $\alpha$ 给 $q=\dfrac{\alpha+1}{2}$——一般的 $\alpha$ 这两个 $\Gamma$ 值化不动，答案就停在 $\Gamma$ 上。

       * Wallis 校验（$\alpha=2n$）：用 $\Gamma\!\left(n+\dfrac12\right)=\dfrac{(2n-1)!!}{2^n}\sqrt\pi$ 与 $(2n)!!=2^n n!$，公式还原出 $\dfrac{(2n-1)!!}{(2n)!!}\cdot\dfrac\pi2$，与定积分一章的 Wallis 积分一致——新公式把老结果原封不动包含进来。

       * Wallis 校验（$\alpha=2n+1$）：用 $\Gamma\!\left(n+\dfrac32\right)=\dfrac{(2n+1)!!}{2^{n+1}}\sqrt\pi$，得 $\dfrac{(2n)!!}{(2n+1)!!}$，老结果再次被完整接住。

---

## 勒让德、余元与斯特林公式

   * **勒让德公式（Legendre formula）**

       * 公式：$\Gamma(s)\,\Gamma\!\left(s+\dfrac12\right)=2^{1-2s}\sqrt\pi\;\Gamma(2s)$（$s>0$）。

       * 证明入口：$B(s,s)$ 一条式子两种算法——B–Γ 关系取 $p=q=s$ 给 $B(s,s)=\dfrac{\Gamma(s)^2}{\Gamma(2s)}$；按定义合并两个相同幂次给 $B(s,s)=\displaystyle\int_0^1\bigl[x(1-x)\bigr]^{s-1}dx$。

       * 配方与换元：$x(1-x)=\dfrac14-\left(x-\dfrac12\right)^2$ 关于 $x=\dfrac12$ 对称，积分折半乘 $2$；再令 $\left(\dfrac12-x\right)^2=\dfrac t4$，整理出 $B(s,s)=2^{1-2s}\,B\!\left(\dfrac12,s\right)$。

       * 合流：$\dfrac{\Gamma(s)^2}{\Gamma(2s)}=2^{1-2s}\cdot\dfrac{\sqrt\pi\,\Gamma(s)}{\Gamma\left(s+\frac12\right)}$，消去一个 $\Gamma(s)$ 移项即得——全程只动用 B–Γ 关系、一次配方、一次换元。

   * **余元公式（reflection formula）**

       * 公式：$\Gamma(s)\,\Gamma(1-s)=\dfrac{\pi}{\sin\pi s}$（$0<s<1$），课上不证、直接叙述。

       * 用途：单个 $\Gamma$ 值求不动时，"互补的一对"乘起来可以求——如 $\Gamma\!\left(\dfrac14\right)\Gamma\!\left(\dfrac34\right)=\dfrac{\pi}{\sin\frac\pi4}=\sqrt2\,\pi$。

       * > **注意**：公式只给乘积、"没有一半"——单凭它，$\Gamma\!\left(\dfrac14\right)$ 自己等于多少仍然求不出来。

| $\Gamma(s)$ 的落点 | 具体值怎么求 |
| :--- | :--- |
| 正整数 $s=n+1$ | $\Gamma(n+1)=n!$ |
| 半整数 $s=n+\dfrac12$ | 递推链一路降到 $\Gamma\!\left(\dfrac12\right)=\sqrt\pi$ |
| 其余 $0<s<1$ | 单个求不出；余元公式给出乘积 $\Gamma(s)\Gamma(1-s)=\dfrac{\pi}{\sin\pi s}$ |

   * **斯特林公式（Stirling formula）**

       * 公式：$\Gamma(s+1)=\sqrt{2\pi s}\,\left(\dfrac{s}{e}\right)^{s}e^{\frac{\theta}{12s}}$（$0<\theta<1$），只叙述不证。

       * 与老熟人接轨：令 $s=n$ 且 $n\to\infty$，$0<\theta<1$ 保证 $e^{\frac{\theta}{12n}}\to1$，就还原出无穷乘积一章的 $n!\sim\sqrt{2\pi}\;n^{\,n+\frac12}\,e^{-n}$——阶乘的等价式被它整个包含。

       * 强在哪：原来的等价式只说"两个一除极限是 $1$"，靠近得多快没说；现在的比值恰为 $e^{\frac{\theta}{12s}}$，夹在 $1$ 与 $e^{\frac{1}{12s}}$ 之间——不只等价，连趋向 $1$ 的速度（无穷小量的阶）都给了，且 $s$ 不必是正整数。一句话：等价管趋势，它管趋势加余项。

---

## Euler积分的综合应用

   * **无穷区间上的四次根式**

       * 目标：$I=\displaystyle\int_0^{+\infty}\frac{\sqrt[4]{x}}{(1+x)^2}\,dx$。

       * 认形：对照 $B(p,q)=\displaystyle\int_0^{+\infty}\frac{x^{p-1}}{(1+x)^{p+q}}\,dx$——分母给 $p+q=2$，分子给 $p-1=\dfrac14$ 即 $p=\dfrac54$，从而 $q=\dfrac34$。

       * 计算：$I=B\!\left(\dfrac54,\dfrac34\right)=\dfrac{\Gamma(3/4)\,\Gamma(5/4)}{\Gamma(2)}=\dfrac14\,\Gamma\!\left(\dfrac14\right)\Gamma\!\left(\dfrac34\right)=\dfrac14\cdot\dfrac{\pi}{\sin\frac\pi4}=\dfrac{\sqrt2\,\pi}{4}$——递推与余元公式接力。

   * **极坐标曲线围出的面积**

       * 目标：曲线 $r^4=\sin^3\theta\,\cos\theta$ 所围图形的面积。

       * 定位：左边 $r^4$ 永远非负，要求 $\sin\theta$ 与 $\cos\theta$ 同号——曲线只在第一、第三象限，是关于原点对称的两瓣闭圈，面积取一瓣再乘 $2$。

       * 化简：$r^2=\sin^{3/2}\theta\cos^{1/2}\theta$，两瓣的 $2$ 与面积公式 $S=\dfrac12\int_\alpha^\beta r^2\,d\theta$ 里的 $\dfrac12$ 正好抵消，$S=\displaystyle\int_0^{\pi/2}\sin^{3/2}\theta\cos^{1/2}\theta\,d\theta$。

       * 收尾：三角形式给 $p=\dfrac54$、$q=\dfrac34$，$S=\dfrac12\,B\!\left(\dfrac54,\dfrac34\right)=\dfrac18\,\Gamma\!\left(\dfrac14\right)\Gamma\!\left(\dfrac34\right)=\dfrac{\sqrt2\,\pi}{8}$——上一题的乘积直接复用。

   * **二重积分的收敛判据**

       * 目标：$I=\displaystyle\iint_{D}\frac{dx\,dy}{1+x^\alpha+y^\beta}$（$D$ 为第一象限 $\{x>0,\ y>0\}$，$\alpha,\beta$ 为正数），问 $\alpha,\beta$ 满足什么关系时收敛。

       * 换元链：令 $x^\alpha=u^2$、$y^\beta=v^2$（反解 $x=u^{2/\alpha}$、$y=v^{2/\beta}$）把分母洗成最舒服的 $1+u^2+v^2$，雅可比行列式 $\dfrac{\partial(x,y)}{\partial(u,v)}=\dfrac{4}{\alpha\beta}\,u^{\frac{2}{\alpha}-1}v^{\frac{2}{\beta}-1}$；再上极坐标；径向令 $t=r^2$。

    ```text
    (x, y) --令 x^α=u^2, y^β=v^2--> (u, v) --极坐标--> (r, θ) --令 t=r^2--> 化成两个 B 函数
     第一象限                        第一象限          θ∈[0,π/2]      角向 B(1/α, 1/β)
                                                                    径向 B(1-1/α-1/β, 1/α+1/β)
    ```

       * 角向：$\displaystyle\int_0^{\pi/2}\cos^{\frac{2}{\alpha}-1}\theta\sin^{\frac{2}{\beta}-1}\theta\,d\theta=\frac12\,B\!\left(\frac1\alpha,\frac1\beta\right)$——老流程，$2p-1=\dfrac2\alpha-1$、$2q-1=\dfrac2\beta-1$。

       * 径向：$\displaystyle\int_0^{+\infty}\frac{r^{\frac{2}{\alpha}+\frac{2}{\beta}-1}}{1+r^2}\,dr=\frac12\int_0^{+\infty}\frac{t^{\frac{1}{\alpha}+\frac{1}{\beta}-1}}{1+t}\,dt=\frac12\,B\!\left(1-\frac{1}{\alpha}-\frac{1}{\beta},\ \frac{1}{\alpha}+\frac{1}{\beta}\right)$——分子读成 $t^{q-1}$ 给 $q=\dfrac1\alpha+\dfrac1\beta$，分母 $(1+t)^1$ 给 $p+q=1$（$B$ 对称，两种读法殊途同归）。

       * 合并：$\dfrac{4}{\alpha\beta}$ 里的 $4$ 被两个 $\dfrac12$ 各吃一个，翻成 $\Gamma$ 后中间的 $\Gamma\!\left(\dfrac1\alpha+\dfrac1\beta\right)$ 上下正好消掉，$I=\dfrac{1}{\alpha\beta}\,\Gamma\!\left(\dfrac1\alpha\right)\Gamma\!\left(\dfrac1\beta\right)\Gamma\!\left(1-\dfrac{1}{\alpha}-\dfrac{1}{\beta}\right)$。

       * 收敛条件：$\alpha,\beta>0$ 保证前两个 $\Gamma$ 的参数为正；最后一个要求 $1-\dfrac{1}{\alpha}-\dfrac{1}{\beta}>0$，即 $\dfrac{1}{\alpha}+\dfrac{1}{\beta}<1$ 时积分收敛。

       * > **易错点**：变量代换时雅可比行列式绝对不能漏——被积函数、积分区域、面积元三处都要跟着变，漏掉任何一处整道题就垮。

       * 通法沉淀：见到 $x^\alpha$、$y^\beta$ 就令它们等于 $u^2$、$v^2$，把分母洗成 $1+u^2+v^2$，极坐标一上角向、径向自动分家，各自落进 B 函数的形状——第一象限上带幂的积分通法就是这一套。
