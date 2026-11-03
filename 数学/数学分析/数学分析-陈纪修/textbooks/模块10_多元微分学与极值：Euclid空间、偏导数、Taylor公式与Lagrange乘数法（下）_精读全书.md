# 数学分析 陈纪修老师 1080p高清版(全集)·多元微分学与极值：Euclid空间、偏导数、Taylor公式与Lagrange乘数法（下）（第 10 册 / 共 13 册）

> **所属课程**：数学分析 陈纪修老师 1080p高清版(全集)  
> **本册内容**：多元微分学与极值：Euclid空间、偏导数、Taylor公式与Lagrange乘数法（下）  
> **覆盖范围**：P145 ~ P157（共 12 讲 / 12 章 / 542 分钟音频）  
> **分册说明**：全书按内容分 13 册；本册为第 10 册  
> **整编说明**：正文逐字保留各块模块长文，仅补导读、目录与章间过渡。

---

## 导读与全景目录

1. 向量值函数的链式法则与一阶全微分形式不变性（P145）
2. 多元函数的中值定理与泰勒公式（P146）
3. 泰勒公式的应用与多元微分学例题（P147）
4. 空间曲线的切线与法平面（P149）
5. 从螺旋线到曲面的切平面与法线（P150）
6. 曲面的参数表示：法向量、切平面与正交曲面（P151）
7. 多元函数的无条件极值：定义、必要条件与判别法的铺垫（P152）
8. 无条件极值的判定：从驻点到二次型（P153）
9. n 元函数的极值判别与最值问题（P154）
10. 条件极值问题与 Lagrange 乘数法：从原点到一条直线的距离说起（P155）
11. 拉格朗日乘数法：一般理论与两个例题（P156）
12. 条件极值三例：二次型最值、幂乘积与最优价格模型（P157）

---

## 向量值函数的链式法则与一阶全微分形式不变性
> 对应块：BLK135 | 覆盖分集：P145 | 块标题（分集名）：《144.第12章 第2节 多元复合函数的求导法则》

上次我们已经讲了多元复合函数求导的链式法则，讲的是"多元函数与向量值函数复合"的情形。今天继续深入讨论：向量值函数与向量值函数的复合。先把上次的结论摆出来，看清两件"东西的形状"，再推今天的矩阵链式法则；下半场转到全微分，那里有一个非常重要的结论——一阶全微分的形式不变性，以及它在高阶微分上的失效。

### 从上次的链式法则说起

上次的设定是：$z = f(\boldsymbol{y})$，这里 $\boldsymbol{y} = (y_1, y_2, \ldots, y_m)$ 是一个向量，而 $\boldsymbol{y}$ 又是 $\boldsymbol{x}$ 的函数：$\boldsymbol{y} = g(\boldsymbol{x})$，$\boldsymbol{x} \in D_G \subset \mathbb{R}^n$。$f$ 是 $y$ 的函数，$y$ 又是 $x$ 的函数，合起来就构成 $z$ 关于 $x$ 的复合函数。复合以后求导怎么求？外层导数乘内层导数：

$$(f\circ g)' = f'(y)\cdot g'(x),$$

其中 $f'$ 取在 $y$ 这一点。这里要注意两件东西的形状。这个 $f'$ 是什么？是个横向量，对不对？$f$ 是数量值函数，它对 $m$ 个中间变量的偏导排成一行，是 $m$ 维的横向量；而 $g'(x)$ 呢，是 $m\times n$ 的一个矩阵。横向量乘矩阵，乘出来正好是 $n$ 维的横向量。

今天我们要讨论的，是向量值函数与向量值函数的复合。先从简单的考虑：取 $n = 2$，换句话说，这两个向量值函数都是两个自变量、两个应变量的情况。

### 两个向量值函数的复合：矩阵形式的链式法则

设外层的向量值函数为

$$F:\quad x = x(u,v),\qquad y = y(u,v),\qquad (u,v)\in D_F,$$

内层的为

$$G:\quad u = u(s,t),\qquad v = v(s,t),\qquad (s,t)\in D_G.$$

变量之间的关系是一条两段的链：

```text
(s,t) --G--> (u,v) --F--> (x,y)
```

读法：$(s,t)$ 是最外层的自变量，$G$ 把它送到中间变量 $(u,v)$，$F$ 再把 $(u,v)$ 送到因变量 $(x,y)$。

复合要有意义，需要 $G$ 的值域包含在 $F$ 的定义域里边；这样就有 $F$ 跟 $G$ 的复合函数，自变量是 $(s,t)$。问题是：这个复合函数的导数是怎么样的？

把 $G$ 带到 $F$ 里边以后，复合函数就是

$$x = x\big(u(s,t),\,v(s,t)\big),\qquad y = y\big(u(s,t),\,v(s,t)\big).$$

来求 $x$、$y$ 关于 $s$、$t$ 的偏导。$x$ 既通过 $u$ 依赖 $s,t$，又通过 $v$ 依赖 $s,t$，用上节课的链式法则：

$$\frac{\partial x}{\partial s} = \frac{\partial x}{\partial u}\frac{\partial u}{\partial s} + \frac{\partial x}{\partial v}\frac{\partial v}{\partial s},\qquad \frac{\partial x}{\partial t} = \frac{\partial x}{\partial u}\frac{\partial u}{\partial t} + \frac{\partial x}{\partial v}\frac{\partial v}{\partial t},$$

$$\frac{\partial y}{\partial s} = \frac{\partial y}{\partial u}\frac{\partial u}{\partial s} + \frac{\partial y}{\partial v}\frac{\partial v}{\partial s},\qquad \frac{\partial y}{\partial t} = \frac{\partial y}{\partial u}\frac{\partial u}{\partial t} + \frac{\partial y}{\partial v}\frac{\partial v}{\partial t}.$$

这四个式子，都是上节课讲的链式法则推出来的。现在把它们写成矩阵的形式：

$$\begin{pmatrix}\dfrac{\partial x}{\partial s} & \dfrac{\partial x}{\partial t}\\[3mm]\dfrac{\partial y}{\partial s} & \dfrac{\partial y}{\partial t}\end{pmatrix} = \begin{pmatrix}\dfrac{\partial x}{\partial u} & \dfrac{\partial x}{\partial v}\\[3mm]\dfrac{\partial y}{\partial u} & \dfrac{\partial y}{\partial v}\end{pmatrix}\begin{pmatrix}\dfrac{\partial u}{\partial s} & \dfrac{\partial u}{\partial t}\\[3mm]\dfrac{\partial v}{\partial s} & \dfrac{\partial v}{\partial t}\end{pmatrix}.$$

大家对应一下：左边矩阵第一行第一列，正好是第一个式子；第一行第二列是第二个式子……这个矩阵等式恰好就是把那四个式子打包在一起。而左边正是 $F$ 跟 $G$ 复合以后的导数；右边第一个因子是 $F$ 的导数——$(x,y)$ 关于 $(u,v)$ 的导数，第二个因子是 $(u,v)$ 关于 $(s,t)$ 的导数，是 $G$ 的导数。所以我们马上知道：

> **结论**：$F$ 与 $G$ 复合以后求导，等于 $F$ 的导数乘上 $G$ 的导数：$(F\circ G)' = F'(u,v)\cdot G'(s,t)$。

注意，像这样的结论，跟我们一元函数求导的链式法则形式上又一致——上次的"横向量乘矩阵"是如此，今天的"矩阵乘矩阵"也是如此：外层导数乘内层导数，只是乘法从数的乘法变成了矩阵乘法。

### 例 3：三角函数复合的求导

设

$$F:\quad x = \cos u\sin v,\qquad y = \sin u\cos v,$$

$$G:\quad u = s+t,\qquad v = s-t,$$

求复合以后 $(x,y)$ 关于 $(s,t)$ 的导数。按刚得到的结论：

$$\begin{pmatrix}\dfrac{\partial x}{\partial s} & \dfrac{\partial x}{\partial t}\\[3mm]\dfrac{\partial y}{\partial s} & \dfrac{\partial y}{\partial t}\end{pmatrix} = \begin{pmatrix}\dfrac{\partial x}{\partial u} & \dfrac{\partial x}{\partial v}\\[3mm]\dfrac{\partial y}{\partial u} & \dfrac{\partial y}{\partial v}\end{pmatrix}\begin{pmatrix}\dfrac{\partial u}{\partial s} & \dfrac{\partial u}{\partial t}\\[3mm]\dfrac{\partial v}{\partial s} & \dfrac{\partial v}{\partial t}\end{pmatrix}.$$

先把两个因子算出来。$F$ 的导数是

$$\begin{pmatrix}-\sin u\sin v & \cos u\cos v\\ \cos u\cos v & -\sin u\sin v\end{pmatrix},$$

$G$ 的导数更简单：$u_s = u_t = v_s = 1$，$v_t = -1$，恰好是

$$\begin{pmatrix}1 & 1\\ 1 & -1\end{pmatrix}.$$

两个矩阵相乘，乘出来的每个元素都是"两项相加或相减"，正好套三角公式：$\cos u\cos v - \sin u\sin v = \cos(u+v)$，$\cos u\cos v + \sin u\sin v = \cos(u-v)$。比如左上角这个元素，就是 $-\sin u\sin v + \cos u\cos v = \cos(u+v)$；右上角是两项都带负号，得 $-\cos(u-v)$。乘出来：

$$\begin{pmatrix}\cos(u+v) & -\cos(u-v)\\ \cos(u+v) & \cos(u-v)\end{pmatrix}.$$

到了这一步，想想看 $u+v$ 是什么？$2s$；$u-v$ 呢？$2t$。代进去，得到

$$\begin{pmatrix}\dfrac{\partial x}{\partial s} & \dfrac{\partial x}{\partial t}\\[3mm]\dfrac{\partial y}{\partial s} & \dfrac{\partial y}{\partial t}\end{pmatrix} = \begin{pmatrix}\cos 2s & -\cos 2t\\ \cos 2s & \cos 2t\end{pmatrix},$$

也就是 $\dfrac{\partial x}{\partial s} = \cos 2s$，$\dfrac{\partial x}{\partial t} = -\cos 2t$，$\dfrac{\partial y}{\partial s} = \cos 2s$，$\dfrac{\partial y}{\partial t} = \cos 2t$。

### 一阶全微分的形式不变性

下面讲全微分当中一个很重要的结论。一元函数里我们知道，一阶微分具有形式不变性；对多元函数来讲，一阶全微分仍然有这样的结论。

先把条件交代清楚。讨论这个问题时，我们讨论的函数都满足可微的条件——有时我们直接加条件，比如说一阶偏导连续，而一阶偏导连续实际上就隐含了可微。如果要讨论高阶微分，比如二阶微分，就要假设二阶偏导都连续：二阶偏导连续说明了什么？一阶微分还可以求全微分；不仅如此，还要注意一点，二阶偏导连续还保证混合导数相等。所以这一节里，函数所需要的可微性条件，我们都假设它满足。

设 $z = f(x,y)$。当 $x$、$y$ 是自变量时，可微函数的全微分是

$$dz = \frac{\partial z}{\partial x}dx + \frac{\partial z}{\partial y}dy.$$

现在假如 $x$、$y$ 是中间变量：

$$x = x(u,v),\qquad y = y(u,v),$$

那么 $z$ 应该看成 $u$、$v$ 的函数。这时候 $x$、$y$ 自己的全微分是

$$dx = \frac{\partial x}{\partial u}du + \frac{\partial x}{\partial v}dv,\qquad dy = \frac{\partial y}{\partial u}du + \frac{\partial y}{\partial v}dv.$$

而既然 $z$ 现在是 $u$、$v$ 的函数，它的全微分就该写成

$$dz = \frac{\partial z}{\partial u}du + \frac{\partial z}{\partial v}dv,$$

其中两个偏导用复合函数的链式法则展开：

$$\frac{\partial z}{\partial u} = \frac{\partial z}{\partial x}\frac{\partial x}{\partial u} + \frac{\partial z}{\partial y}\frac{\partial y}{\partial u},\qquad \frac{\partial z}{\partial v} = \frac{\partial z}{\partial x}\frac{\partial x}{\partial v} + \frac{\partial z}{\partial y}\frac{\partial y}{\partial v}.$$

把这两个式子往上带，按 $du$、$dv$ 归拢：

$$dz = \left(\frac{\partial z}{\partial x}\frac{\partial x}{\partial u} + \frac{\partial z}{\partial y}\frac{\partial y}{\partial u}\right)du + \left(\frac{\partial z}{\partial x}\frac{\partial x}{\partial v} + \frac{\partial z}{\partial y}\frac{\partial y}{\partial v}\right)dv.$$

把含 $\dfrac{\partial z}{\partial x}$ 的两项提出来，剩下的一大块 $\dfrac{\partial x}{\partial u}du + \dfrac{\partial x}{\partial v}dv$ 恰好就是刚才的 $dx$；再把 $\dfrac{\partial z}{\partial y}$ 提出来，剩下的恰好是 $dy$。于是

$$dz = \frac{\partial z}{\partial x}dx + \frac{\partial z}{\partial y}dy.$$

也就是说，不管 $x$、$y$ 是自变量也好，是中间变量也好，二元函数的全微分 $dz$ 都可以这样写。

> **结论**（一阶全微分形式不变性）：无论 $x,y$ 是自变量还是中间变量，$dz = \dfrac{\partial z}{\partial x}dx + \dfrac{\partial z}{\partial y}dy$ 在形式上总是成立。

这给我们以后的运算带来方便。为什么？你求 $z$ 的全微分时，不要去管这里的 $x$、$y$ 是不是还是其他变量的函数——不要管它是中间变量还是自变量，直接把这个式子写出来，不会错。假如这条性质仅仅对自变量成立，那你每次都得先掂量：这个 $x$、$y$ 能不能是其他变量的函数？它跟着其他变量变的时候公式还敢不敢用？那就会给我们带来不方便。

### 高阶微分没有形式不变性

一阶全微分具有形式不变性，那么高阶全微分呢？没有这个性质。来看二阶的。

二阶微分就是一阶微分的再微分：$d^2z = d(dz)$，而 $dz = \dfrac{\partial z}{\partial x}dx + \dfrac{\partial z}{\partial y}dy$。对它再求一次微分，这里多写一步，把乘积法则老老实实用上：

$$d^2z = d\!\left(\frac{\partial z}{\partial x}\right)dx + \frac{\partial z}{\partial x}\,d^2x + d\!\left(\frac{\partial z}{\partial y}\right)dy + \frac{\partial z}{\partial y}\,d^2y.$$

这四项怎么来的？$d\!\left(\dfrac{\partial z}{\partial x}\right)dx$ 是第一个因子求微分、$dx$ 不动；$\dfrac{\partial z}{\partial x}d^2x$ 是 $z_x$ 不动、$dx$ 求微分，而 $dx$ 的微分就是 $d^2x$。后两项同理。

先看关键的一点：如果 $x$、$y$ 是自变量，那么 $d^2x = 0$、$d^2y = 0$，后面两项是零。但如果 $x$、$y$ 不是自变量，而是中间变量——它可以看成另外两个变量比如说 $s,t$ 的函数——那么这两个二阶微分就不等于零。先把会算的算掉：

$$d\!\left(\frac{\partial z}{\partial x}\right) = \frac{\partial^2 z}{\partial x^2}dx + \frac{\partial^2 z}{\partial y\,\partial x}dy,\qquad d\!\left(\frac{\partial z}{\partial y}\right) = \frac{\partial^2 z}{\partial x\,\partial y}dx + \frac{\partial^2 z}{\partial y^2}dy.$$

代进去，前面部分归拢：

$$\frac{\partial^2 z}{\partial x^2}dx^2 + \frac{\partial^2 z}{\partial y\,\partial x}dy\,dx + \frac{\partial^2 z}{\partial x\,\partial y}dx\,dy + \frac{\partial^2 z}{\partial y^2}dy^2.$$

我们假设函数具有二阶连续偏导，混合导数相等，两个交叉项合并成一项：

$$\frac{\partial^2 z}{\partial x^2}dx^2 + 2\frac{\partial^2 z}{\partial x\,\partial y}dx\,dy + \frac{\partial^2 z}{\partial y^2}dy^2.$$

所以二阶全微分实际上是

$$d^2z = \frac{\partial^2 z}{\partial x^2}dx^2 + 2\frac{\partial^2 z}{\partial x\,\partial y}dx\,dy + \frac{\partial^2 z}{\partial y^2}dy^2 + \frac{\partial z}{\partial x}d^2x + \frac{\partial z}{\partial y}d^2y.$$

大家注意：$x$、$y$ 是自变量，最后两项没有；$x$、$y$ 是中间变量，最后两项有。所以高阶微分不具有形式不变性。

| 微分 | $x,y$ 是自变量 | $x,y$ 是中间变量 |
| :--- | :--- | :--- |
| 一阶 $dz$ | $\dfrac{\partial z}{\partial x}dx + \dfrac{\partial z}{\partial y}dy$ | 形式相同 |
| 二阶 $d^2z$ | $\dfrac{\partial^2 z}{\partial x^2}dx^2 + 2\dfrac{\partial^2 z}{\partial x\,\partial y}dxdy + \dfrac{\partial^2 z}{\partial y^2}dy^2$ | 上式再加 $\dfrac{\partial z}{\partial x}d^2x + \dfrac{\partial z}{\partial y}d^2y$ |

表中 $dx^2$、$dy^2$ 指微分的平方，即 $(dx)^2$、$(dy)^2$；$dx\,dy$ 是两个微分的乘积。

> **易错点**：一阶的形式不变性很容易被想当然地推广到高阶。二阶微分在中间变量之下多出来的 $\dfrac{\partial z}{\partial x}d^2x + \dfrac{\partial z}{\partial y}d^2y$ 一般不为零，丢掉它就错了。

### 例题：四次根式的全微分

求 $z = \sqrt[4]{\dfrac{x+y}{x-y}}$ 的全微分。

如果直接求——$z$ 对 $x$ 求偏导乘 $dx$，加上 $z$ 对 $y$ 求偏导乘 $dy$——这完全可以。但是大家看到这样一个求偏导，肯定觉得哎呀蛮烦的：它里边是个分式，分数还好，外头还罩着个四次根号。那么想想看，用什么简单的方法来做？

两边取对数：

$$\log z = \frac{1}{4}\big[\log(x+y) - \log(x-y)\big].$$

然后两边求全微分。左边求全微分，是 $\dfrac{1}{z}dz$。这一步注意：我们是对 $\log z$ 求全微分，得到 $\dfrac{1}{z}dz$——这里用的正是一阶微分形式不变性，因为 $z$ 不是自变量，$z$ 是 $x$、$y$ 的函数。右边是

$$\frac{1}{4}\left[\frac{dx+dy}{x+y} - \frac{dx-dy}{x-y}\right],$$

于是

$$\frac{dz}{z} = \frac{1}{4}\left[\frac{dx+dy}{x+y} - \frac{dx-dy}{x-y}\right].$$

把它通分，$\dfrac{1}{4}$ 搁在外面，分母是 $(x+y)(x-y) = x^2 - y^2$，分子是

$$(x-y)(dx+dy) - (x+y)(dx-dy).$$

展开来看：$x\,dx$ 与 $-x\,dx$ 消掉；$-y\,dy$ 与 $+y\,dy$ 也消掉；剩下 $x\,dy$ 和 $-y\,dx$ 两项，而且各自翻倍——所以 $\dfrac{1}{4}$ 变成了 $\dfrac{1}{2}$：

$$\frac{dz}{z} = \frac{1}{2}\cdot\frac{x\,dy - y\,dx}{x^2-y^2}.$$

那么 $dz$ 就是把这个 $z$ 乘过来：

$$dz = \frac{1}{2(x^2-y^2)}\left(x\,dy - y\,dx\right)\sqrt[4]{\frac{x+y}{x-y}}.$$

全微分好了。而且从这个全微分马上可以读出两个偏导——$dx$ 的系数、$dy$ 的系数就是偏导数：

$$\frac{\partial z}{\partial x} = -\frac{y}{2(x^2-y^2)}\sqrt[4]{\frac{x+y}{x-y}},\qquad \frac{\partial z}{\partial y} = \frac{x}{2(x^2-y^2)}\sqrt[4]{\frac{x+y}{x-y}}.$$

### 例 12.29：$z=\log(x+y)$ 的 $k$ 阶全微分

再来看一个例题：设 $z = \log(x+y)$，求它的高阶全微分，$k$ 是任意一个正整数，也就是要求 $d^kz$。

这里的处理大家注意：令 $u = x+y$。这个 $u$ 是什么？中间变量。那么

$$du = dx + dy.$$

再看 $u$ 的二阶微分：$x$、$y$ 是自变量，所以

$$d^2u = 0.$$

这一点记住，马上要用。先算一阶：

$$dz = \frac{1}{u}\,du.$$

对 $\log u$ 求微分得 $\dfrac{1}{u}du$，用的就是一阶微分形式不变性——$z = \log u$，$u$ 是中间变量。代进去就是

$$dz = \frac{dx+dy}{x+y}.$$

二阶微分就是一阶微分的再微分：

$$d^2z = d\!\left(\frac{1}{u}\,du\right) = d\!\left(\frac{1}{u}\right)du + \frac{1}{u}\,d^2u.$$

后面这项是什么？零——刚才 $d^2u = 0$ 在这里用上了。前面 $d\!\left(\dfrac{1}{u}\right) = -\dfrac{1}{u^2}du$（又一次用了形式不变性），所以

$$d^2z = -\frac{1}{u^2}\,du^2 = -\frac{(dx+dy)^2}{(x+y)^2}.$$

到这儿大家有没有发现一点：尽管 $u$ 是中间变量、不是自变量，但我们得到的 $d^2z = -\dfrac{1}{u^2}du^2$，跟 $u$ 是自变量时的公式一模一样。这说明什么？说明对这个题目来讲，它的高阶微分也有形式不变性。原因是什么？因为 $u$ 是自变量的一次项加加减减——一次式再微分，二阶微分照样是零。假如它不是一次项，而是二次项、三次项或者其他函数，那 $d^2u$ 就不等于零，高阶微分就没有这样好的形式了。

接着看三阶：$d^3z = d\!\left(-\dfrac{1}{u^2}du^2\right)$，还是老办法，乘积法则拆开：

$$d^3z = d\!\left(-\frac{1}{u^2}\right)du^2 + \left(-\frac{1}{u^2}\right)d(du^2).$$

后面这一项里 $d(du^2) = 2\,du\cdot d^2u = 0$，又是零；前面 $d\!\left(-\dfrac{1}{u^2}\right) = \dfrac{2}{u^3}du$。所以

$$d^3z = \frac{2}{u^3}\,du^3 = \frac{2!}{(x+y)^3}(dx+dy)^3.$$

写成 $x$、$y$ 的函数，就是 $x+y$ 的三次方、二的阶乘、$dx+dy$ 的三次方。

照这个规律，每升一阶：符号一正一负翻一次，阶乘多一档，幂次升一次。利用数学归纳法，从 $k$ 推到 $k+1$ 只要把这套操作再来一遍，这一步的验证大家可以当作练习自己做：

> **结论**：对任意正整数 $k$，
>
> $$d^kz = (-1)^{k-1}\frac{(k-1)!}{(x+y)^k}\,(dx+dy)^k.$$

这一讲就到这里。链式法则的矩阵形式、一阶全微分的形式不变性、以及高阶微分在什么情况下"碰巧"还保持形式——这些都是很基本的操作，回去大家在这方面还要加强练习。

> **承前启后**：上一节讲完「向量值函数的链式法则与一阶全微分形式不变性」，下一节接着讲「多元函数的中值定理与泰勒公式」。

---

## 多元函数的中值定理与泰勒公式
> 对应块：BLK136 | 覆盖分集：P146 | 块标题（分集名）：《145.第12章 第3节 中值定理与Taylor》

一元微分学里，中值定理我们已经打过交道：当时学了几个？三个——罗尔定理、拉格朗日定理、柯西定理。到了多元函数，我们要介绍的是拉格朗日中值定理和泰勒公式。为什么一元学过的东西，多元还要再立一遍？因为在微分学里，中值定理和泰勒公式的应用实在太重要了，一元微积分里它们撑起了一大片内容，多元函数同样绕不开，所以这两个定理必须重新建立起来。

不过动手之前，得先备一个新概念：凸区域。它是这一块所有定理的舞台。这一讲的路线是：先立凸区域，再证多元的中值定理，推出一个重要推论，然后把两件事都推广到 $n$ 元函数，最后讲泰勒公式。

### 凸区域：连线不出界的区域

我们平时说的"区域"，指的是连通开集。在区域的基础上再加一条要求：对 $D$ 内任意两点 $X$、$Y$，连接 $X$ 与 $Y$ 的直线段也整个落在 $D$ 里，这样的区域就叫凸区域。

> **定义**：设 $D$ 是区域（连通开集）。若对任意 $X,Y\in D$，连接两点的线段 $\{\,X+\lambda(Y-X)\mid 0\le\lambda\le 1\,\}$ 都含于 $D$，则称 $D$ 是凸区域。

在平面上画个图最直观。左边的区域，随便取两点连起来，连线全程都在区域里边，这是凸的；右边的区域中间有个凹口，我这里取一点、那里取一点，把两点连起来，直线有一部分跑到区域外面去了，这就不是凸区域。

```text
凸区域：任取两点，连线全程留在 D 内

      ____________
     /            \
    |   A*======*B |      ===== 就是连线 AB
     \____________/

非凸区域：有个凹口，连线有一段跑出 D 外

      ______        ______
     |      |      |      |
     |  A*  |      |  *B  |
     |______|______|______|

     A、B 的连线中段恰好从凹口上方穿过，那一段不在 D 内
```

那连接两点的直线段在式子上怎么表示？这个表示后面到处要用，一定要会写。以 $X$ 为起点，加上 $\lambda$ 倍的 $Y-X$：

$$X+\lambda(Y-X),\qquad 0\le\lambda\le 1$$

看 $\lambda$ 从 $0$ 变到 $1$：$\lambda=0$ 时这个点正好是 $X$，$\lambda=1$ 时正好是 $Y$，中间的每一步都正好落在这条线段上。这个式子还可以换一种写法，把括号拆开整理：

$$X+\lambda(Y-X)=(1-\lambda)X+\lambda Y$$

两种写法完全一样，用哪个都行。

### 多元函数的拉格朗日中值定理

定理叙述如下。设二元函数 $f(x,y)$ 在凸区域 $D$ 上可微，$(x_0,y_0)$ 与 $(x_0+\Delta x,\ y_0+\Delta y)$ 是 $D$ 上任意两点（第二点就写成起点的坐标加增量），则存在 $\theta$，$0<\theta<1$，使得

$$f(x_0+\Delta x,\ y_0+\Delta y)-f(x_0,\ y_0)=f_x(x_0+\theta\Delta x,\ y_0+\theta\Delta y)\,\Delta x+f_y(x_0+\theta\Delta x,\ y_0+\theta\Delta y)\,\Delta y$$

对照一元的 $f(b)-f(a)=f'(\xi)(b-a)$：形状一模一样，增量等于"某一点的导数"乘"增量"，只是这里的"某一点"换成了连接两点的线段上的一个中间点，两个方向的偏导各乘各的增量。

> **易错点**：等式右端一共出现了四个 $\theta$。这四个 $\theta$ 是**同一个** $\theta$——存在一个 $\theta$ 同时让四处成立，不是各配各的。

怎么证明？思路就一句话：把二元的问题化成一元的问题。

第一步，凸性在这里派上用场。$D$ 是凸区域，所以这两点之间的连线整段都在 $D$ 里。沿线段走，动点可以写成

$$(x_0+t\Delta x,\ y_0+t\Delta y),\qquad 0\le t\le 1$$

第二步，沿这条线定义一个新的函数：

$$\varphi(t)=f(x_0+t\Delta x,\ y_0+t\Delta y),\qquad t\in[0,1]$$

注意这个 $\varphi$：$t$ 在 $[0,1]$ 上变化时，动点沿线段走，$\varphi$ 就成了 $t$ 的一元函数。$f$ 是可微函数，所以马上可以知道 $\varphi$ 关于 $t$ 连续、可导——换句话说，$\varphi(t)$ 满足拉格朗日中值定理的全部条件。一元中值定理立刻给出：存在 $\theta\in(0,1)$，

$$\varphi(1)-\varphi(0)=\varphi'(\theta)$$

两头先看清楚：$\varphi(1)=f(x_0+\Delta x,\ y_0+\Delta y)$，$\varphi(0)=f(x_0,\ y_0)$。所以 $\varphi(1)-\varphi(0)$ 恰好就是我们要证的等式左端。

第三步，算 $\varphi'(t)$。这是复合函数求导：$f$ 先对 $x$、$y$ 求偏导，然后 $x$、$y$ 再对 $t$ 求导，分别得到 $\Delta x$ 和 $\Delta y$，链式法则串起来就是

$$\varphi'(t)=f_x(x_0+t\Delta x,\ y_0+t\Delta y)\,\Delta x+f_y(x_0+t\Delta x,\ y_0+t\Delta y)\,\Delta y$$

把式子里的 $t$ 换成 $\theta$，恰好就是中值定理等式的右端。证毕。

整个证明里分工很清楚：凸区域保证"线段不出界"，辅助函数负责"把多元压成一元"。

### 直接推论：偏导数恒为零，函数就是常数

一元的时候，中值定理证完马上有一个直接推论：若 $f'(x)$ 恒等于零，则 $f$ 为常数。当年学导数，我们知道"常数的导数是零"；但反过来——导数恒等于零，函数是不是常数？这一步就是靠拉格朗日中值定理立的。

多元函数有完全平行的结论。

> **结论**：设 $f(x,y)$ 的两个偏导数在区域 $D$ 上恒为零，则 $z=f(x,y)$ 在 $D$ 上恒等于常数。

> **注意**：这里对 $D$ 只要求是区域，**没有要求它是凸的**。不管这个区域凸还是不凸，结论都对。为什么可以这么宽，看到证明就明白了。

怎么证？先在小范围里把常数性立起来。在 $D$ 内任取一点 $(x',y')$，以它为中心、$R$ 为半径画一个圆盘 $O((x',y'),\,R)$，并设这个圆盘整个含于 $D$——$D$ 是开集，这一点办得到。注意：圆盘是凸的。

现在问：中值定理在这个圆盘里好用吗？条件过一遍——要用中值定理，得可微呀。$f$ 可微吗？两个偏导数恒为零，零函数当然连续，所以偏导数连续，$f$ 可微。条件齐了。于是对圆盘内任意一点 $(x,y)$，用刚证的中值定理，其中 $\Delta x=x-x'$，$\Delta y=y-y'$：

$$f(x,y)-f(x',y')=f_x\bigl(x'+\theta(x-x'),\ y'+\theta(y-y')\bigr)(x-x')+f_y\bigl(x'+\theta(x-x'),\ y'+\theta(y-y')\bigr)(y-y')$$

两个偏导数恒为零，右端两项全为零，于是

$$f(x,y)=f(x',y')$$

也就是说，在这个凸的圆盘里，任何一点的函数值都跟圆心处相等——偏导恒为零的函数，在任何凸的子区域上都是常数。

### 常数性怎么传遍整个区域：道路连通

麻烦在下一步。比如要比较的两点，一点在这里、一点在那里，隔得很远，一个圆盘罩不住它们俩——刚才"圆盘内套中值定理"的办法直接用不上了。为什么用不上？圆盘的半径有限制，不能随便放大，再大就跑到区域 $D$ 外面去了。这时候就要用到区域的性质：连通。

对开集来说，连通就是道路连通：$D$ 内任意两点之间，存在一条整个落在 $D$ 内的连续道路。把道路写成映射 $\gamma$，它是从 $[0,1]$ 到 $D\subset\mathbb{R}^2$ 的连续映射——说白了就是一个向量值函数。任取两点，记 $A=(x_0,y_0)$ 与 $B=(x,y)$，道路 $\gamma$ 满足

$$\gamma(0)=(x_0,y_0),\qquad \gamma(1)=(x,y)$$

考虑 $f$ 与 $\gamma$ 的复合 $f(\gamma(t))$，它是 $[0,1]$ 上的连续函数。两端的函数值：

$$f(\gamma(0))=f(x_0,y_0),\qquad f(\gamma(1))=f(x,y)$$

现在的问题就是：这两头等不等？光看还看不出来。直觉是这样：$t=0$ 时点在起点，函数值是 $f(x_0,y_0)$；让 $t$ 从 $0$ 往 $1$ 跑，点沿道路慢慢挪过去。$t$ 刚出发、还很小时，点还落在起点附近那个凸的小圆盘里，那里的函数值统统等于 $f(x_0,y_0)$；再往前跑出了圆盘，函数值变不变？不知道。那就把"等式能维持到多远"精确地写出来，令

$$T_0=\sup\{\,S:\ \forall\,t\in[0,S],\ f(\gamma(t))=f(\gamma(0))\,\}$$

这个 $T_0$ 的定义要停下来体会一下：集合里的 $S$，意思是"从 $0$ 到 $S$ 这一段上，$f(\gamma(t))$ 始终等于 $f(\gamma(0))$"；$T_0$ 取所有这样的 $S$ 的上确界，就是等式能维持到的最远时刻。

```text
 A = γ(0)                                 B = γ(1)
    ●-----●-----●-----●- - - - - - - - - -●
    t=0                           T0       t=1

   [0, T0] 上：f(γ(t)) = f(A) 恒成立
   T0 = 上述等式能维持的时刻 S 的上确界
```

三件事把 $T_0$ 钉死。

其一，$T_0>0$。$t$ 比 $0$ 大一点点的时候，$\gamma(t)$ 还在起点附近的凸圆盘里，等式成立，所以这样的 $S$ 确实存在，$T_0$ 当然大于零。

其二，$f(\gamma(T_0))=f(\gamma(0))$。$T_0$ 是上确界，等式在 $T_0$ 这一点本身达不达得到，单看定义不知道；但 $f(\gamma(t))$ 是连续函数，$T_0$ 之前它恒等于常数，取极限就把等式传到了 $T_0$ 这一点。

其三，反证 $T_0<1$ 会出矛盾。若 $T_0<1$，$\gamma(T_0)$ 是 $D$ 内的一点，$D$ 是开集，以它为中心存在一个圆盘整个含于 $D$；圆盘是凸的，由刚才的结论，$f$ 在这个圆盘上恒等于 $f(\gamma(T_0))=f(\gamma(0))$。而 $\gamma$ 连续，$t$ 超过 $T_0$ 一点点时，$\gamma(t)$ 就落进这个圆盘，于是那一小段上 $f(\gamma(t))$ 也等于 $f(\gamma(0))$——等式还能往右再延一段，这与 $T_0$ 是上确界矛盾。

所以 $T_0=1$，于是 $f(\gamma(1))=f(\gamma(0))$，即 $f(x,y)=f(x_0,y_0)$。既然对任意一点 $(x,y)$，函数值都跟起点处相等，那这个函数在整个 $D$ 上就是常数。推论证完。

回头看就清楚了：凸性在"每一小段"里干活——保证中值定理用得上；连通在"小段与小段之间"干活——保证常数性能一段一段传下去。所以最后的推论连"凸"都不必要求。

### n 元函数的中值定理

刚才的中值定理是在二维欧氏空间里说的，也就是二元函数的情形。$n$ 元函数 $f(x_1,x_2,\ldots,x_n)$ 对应的结论怎么写？记 $X=(x_1,\ldots,x_n)$，$X$ 与 $X+\Delta X$ 是凸区域 $D$ 上的两点，则存在同一个 $\theta\in(0,1)$，使得

$$f(X+\Delta X)-f(X)=f_{x_1}(X+\theta\Delta X)\,\Delta x_1+f_{x_2}(X+\theta\Delta X)\,\Delta x_2+\cdots+f_{x_n}(X+\theta\Delta X)\,\Delta x_n$$

其中 $X+\theta\Delta X$ 就是 $(x_1+\theta\Delta x_1,\ \ldots,\ x_n+\theta\Delta x_n)$。

用向量、矩阵的语言看这个式子很有意思：把 $n$ 个偏导数排成一行，是一个行向量；把 $n$ 个增量排成一列，是一个列向量；行向量乘列向量，乘出来正好是一个数。动手计算时老老实实把它写开——每一项都是"中间点的偏导数乘对应方向的增量"，而且 $\theta$ 全程是同一个。

### 一元泰勒公式：先换一种写法

接下来讲泰勒公式。一元的泰勒公式很熟了：

$$f(x)=f(x_0)+f'(x_0)(x-x_0)+\frac{1}{2!}f''(x_0)(x-x_0)^2+\cdots+\frac{1}{n!}f^{(n)}(x_0)(x-x_0)^n+R_n$$

余项 $R_n$ 可以取拉格朗日余项，也可以取皮亚诺余项；取拉格朗日余项时，要求 $f$ 在 $x_0$ 的邻域内有 $n+1$ 阶连续导数。

现在换一个写法。记 $x=x_0+\Delta x$，那么 $x-x_0$ 就是 $\Delta x$，同时把各阶导数写成算子的样子：

$$f(x_0+\Delta x)=f(x_0)+\Delta x\,\frac{d}{dx}f(x_0)+\frac{1}{2!}\Delta x^2\,\frac{d^2}{dx^2}f(x_0)+\cdots+\frac{1}{n!}\Delta x^n\,\frac{d^n}{dx^n}f(x_0)+R_n$$

数值上跟原来完全一样，只是外形变了。拉格朗日余项相应地为

$$R_n=\frac{1}{(n+1)!}\Delta x^{n+1}\,\frac{d^{n+1}}{dx^{n+1}}f(x_0+\theta\Delta x)$$

为什么要费劲换个写法？看到下面多元的形式就明白了——这样写，一元与多元长得一模一样，好记。

### 二元函数的泰勒公式

定理叙述如下。设 $f(x,y)$ 在 $(x_0,y_0)$ 的某个邻域 $O((x_0,y_0),\,R)$ 内有 $k+1$ 阶连续偏导数，则对该邻域内的任一点 $(x_0+\Delta x,\ y_0+\Delta y)$，$f$ 可以在 $(x_0,y_0)$ 这点展开到 $k$ 阶：

$$f(x_0+\Delta x,\ y_0+\Delta y)=f(x_0,y_0)+\left(\Delta x\frac{\partial}{\partial x}+\Delta y\frac{\partial}{\partial y}\right)f(x_0,y_0)+\frac{1}{2!}\left(\Delta x\frac{\partial}{\partial x}+\Delta y\frac{\partial}{\partial y}\right)^2 f(x_0,y_0)+\cdots+\frac{1}{k!}\left(\Delta x\frac{\partial}{\partial x}+\Delta y\frac{\partial}{\partial y}\right)^k f(x_0,y_0)+R_k$$

拉格朗日余项为

$$R_k=\frac{1}{(k+1)!}\left(\Delta x\frac{\partial}{\partial x}+\Delta y\frac{\partial}{\partial y}\right)^{k+1}f(x_0+\theta\Delta x,\ y_0+\theta\Delta y),\qquad 0<\theta<1$$

这式子初看唬人，其实规则很简单。把 $\Delta x\frac{\partial}{\partial x}+\Delta y\frac{\partial}{\partial y}$ 当成一个微分算子：它作用在 $f$ 上，就是对 $x$ 求偏导乘 $\Delta x$、对 $y$ 求偏导乘 $\Delta y$。它的平方按二项式展开：

$$\left(\Delta x\frac{\partial}{\partial x}+\Delta y\frac{\partial}{\partial y}\right)^2=\Delta x^2\frac{\partial^2}{\partial x^2}+2\Delta x\Delta y\frac{\partial^2}{\partial x\,\partial y}+\Delta y^2\frac{\partial^2}{\partial y^2}$$

一般地，$k$ 次幂展开以后，每一项都是"对 $x$ 求若干阶偏导、对 $y$ 求若干阶偏导"，总阶数恰好是 $k$，然后一起作用到 $f$ 上。$f$ 有 $k+1$ 阶连续偏导数，混合偏导与求导次序无关，算子的幂才有确定的意义。

> **提示**：这条公式好记。对照刚才换过写法的一元公式：把 $\Delta x\frac{d}{dx}$ 换成 $\Delta x\frac{\partial}{\partial x}+\Delta y\frac{\partial}{\partial y}$，把 $f(x_0)$ 换成 $f(x_0,y_0)$，余项的作用点从 $x_0+\theta\Delta x$ 换成 $(x_0+\theta\Delta x,\ y_0+\theta\Delta y)$，其余一字不改。一元的那个记住了，多元的这个自然就记住了——两个写法形式完全一致。

| 一元（$\Delta x$ 写法） | 二元 |
| :--------------------- | :--- |
| $f(x_0)$ | $f(x_0,\ y_0)$ |
| $\Delta x\frac{d}{dx}$ | $\Delta x\frac{\partial}{\partial x}+\Delta y\frac{\partial}{\partial y}$ |
| $\frac{1}{n!}\left(\Delta x\frac{d}{dx}\right)^n f(x_0)$ | $\frac{1}{k!}\left(\Delta x\frac{\partial}{\partial x}+\Delta y\frac{\partial}{\partial y}\right)^k f(x_0,\ y_0)$ |
| 余项作用点 $x_0+\theta\Delta x$ | 余项作用点 $(x_0+\theta\Delta x,\ y_0+\theta\Delta y)$ |

证明来了。还是那位老朋友——构造辅助函数

$$\varphi(t)=f(x_0+t\Delta x,\ y_0+t\Delta y)$$

这个辅助函数跟我们刚才证明中值定理用的 $\varphi$ 一致，对吧？完全同一个。$f$ 有 $k+1$ 阶连续偏导数，$\varphi$ 作为 $t$ 的函数就有 $k+1$ 阶连续导数。于是对 $\varphi(t)$ 用一元的泰勒公式，展开到 $k$ 阶、带拉格朗日余项：

$$\varphi(t)=\varphi(0)+\varphi'(0)\,t+\frac{1}{2!}\varphi''(0)\,t^2+\cdots+\frac{1}{k!}\varphi^{(k)}(0)\,t^k+\frac{1}{(k+1)!}\varphi^{(k+1)}(\theta t)\,t^{k+1}$$

剩下的工作是把 $\varphi$ 的各阶导数算出来。还是复合函数求导：先对 $x$ 求偏导，$x$ 再对 $t$ 求导得 $\Delta x$；再对 $y$ 求偏导，$y$ 对 $t$ 求导得 $\Delta y$。所以

$$\varphi'(t)=\left(\Delta x\frac{\partial}{\partial x}+\Delta y\frac{\partial}{\partial y}\right)f(x_0+t\Delta x,\ y_0+t\Delta y)$$

再求一次导，算子就再乘一次方：

$$\varphi''(t)=\left(\Delta x\frac{\partial}{\partial x}+\Delta y\frac{\partial}{\partial y}\right)^2 f(x_0+t\Delta x,\ y_0+t\Delta y)$$

一般地，$\varphi^{(m)}(t)$ 就是算子的 $m$ 次幂作用在 $f$ 于动点 $(x_0+t\Delta x,\ y_0+t\Delta y)$ 处的值；余项里 $\varphi^{(k+1)}$ 的作用点是 $(x_0+\theta t\,\Delta x,\ y_0+\theta t\,\Delta y)$。

现在令 $t=1$ 代进去：$\varphi(1)=f(x_0+\Delta x,\ y_0+\Delta y)$，$\varphi(0)=f(x_0,y_0)$；各阶导数项里的 $t$ 全都变成 $1$，消失了；余项里的 $\theta t$ 因为 $t$ 取 $1$ 而变成 $\theta$，作用点正好落在 $(x_0+\theta\Delta x,\ y_0+\theta\Delta y)$。整理一下，二元泰勒公式就得证。

### n 元函数的泰勒公式

$n$ 元的情形照这个模子写。设 $y=f(x_1,x_2,\ldots,x_n)$ 在 $(x_1^0,\ldots,x_n^0)$ 的邻域内有 $k+1$ 阶连续偏导数，则

$$f(x_1^0+\Delta x_1,\ \ldots,\ x_n^0+\Delta x_n)=f(x_1^0,\ldots,x_n^0)+\left(\sum_{i=1}^{n}\Delta x_i\frac{\partial}{\partial x_i}\right)f(x_1^0,\ldots,x_n^0)+\frac{1}{2!}\left(\sum_{i=1}^{n}\Delta x_i\frac{\partial}{\partial x_i}\right)^2 f(x_1^0,\ldots,x_n^0)+\cdots+\frac{1}{k!}\left(\sum_{i=1}^{n}\Delta x_i\frac{\partial}{\partial x_i}\right)^k f(x_1^0,\ldots,x_n^0)+R_k$$

余项为

$$R_k=\frac{1}{(k+1)!}\left(\sum_{i=1}^{n}\Delta x_i\frac{\partial}{\partial x_i}\right)^{k+1}f(x_1^0+\theta\Delta x_1,\ \ldots,\ x_n^0+\theta\Delta x_n)$$

第一项是起点处的函数值，然后算子的一次方、二次方，一路加到 $k$ 次方，每次幂前面配 $\frac{1}{m!}$；余项是 $k+1$ 次方，作用点在线段上的中间点。$n$ 元函数的泰勒公式就是这么长——当然，用向量记号把算子写成紧凑的形式，还能短不少。

至于这套泰勒公式的应用，下一节课再讲。

> **承前启后**：上一节讲完「多元函数的中值定理与泰勒公式」，下一节接着讲「泰勒公式的应用与多元微分学例题」。

---

## 泰勒公式的应用与多元微分学例题
> 对应块：BLK137 | 覆盖分集：P147 | 块标题（分集名）：《146.第12章 第3节 中值定理与Taylor》

上一讲我们已经建立了多元函数的泰勒公式。先把公式的样子重新写一遍，然后这一讲先举两个例子，看看它在计算、特别是近似计算当中怎么用；例题讲完之后，再用一组题目，把偏导数、全微分、复合函数求偏导这些运算综合练一遍。

### 多元泰勒公式的一般形式

设 $f(x,y)$ 在 $(x_0,y_0)$ 的邻域内具有直到 $k+1$ 阶的连续偏导数，那么可以写出 $f$ 在这点的 $k$ 阶泰勒公式：

$$
\begin{aligned}
f(x_0+\Delta x,\,y_0+\Delta y)={}&f(x_0,y_0)+\left(\Delta x\frac{\partial}{\partial x}+\Delta y\frac{\partial}{\partial y}\right)f(x_0,y_0)+\frac{1}{2!}\left(\Delta x\frac{\partial}{\partial x}+\Delta y\frac{\partial}{\partial y}\right)^{2}f(x_0,y_0)\\
&+\cdots+\frac{1}{k!}\left(\Delta x\frac{\partial}{\partial x}+\Delta y\frac{\partial}{\partial y}\right)^{k}f(x_0,y_0)+R_k
\end{aligned}
$$

第一项是一次项，即全微分那一项；记号 $(\cdot)^m$ 表示把括号里的算符像二项式那样正式展开、逐项作用到 $f$ 上。余项 $R_k$ 如果写成拉格朗日余项，则是

$$
R_k=\frac{1}{(k+1)!}\left(\Delta x\frac{\partial}{\partial x}+\Delta y\frac{\partial}{\partial y}\right)^{k+1}f\left(x_0+\theta\Delta x,\;y_0+\theta\Delta y\right),\qquad 0<\theta<1
$$

中间点取在 $(x_0,y_0)$ 与 $(x_0+\Delta x,\,y_0+\Delta y)$ 的连线上。

### 近似计算 $1.08^{3.96}$

要算 $1.08^{3.96}$，有同学说可以用计算器——那要手头没有计算器，怎么把这个值算得比较合理？这正是泰勒公式近似计算的用武之地。

**选函数与展开点。** 考虑二元函数 $f(x,y)=x^y$：底数是 $1.08$，指数是 $3.96$，所以要算的就是 $f(1.08,\,3.96)$。把展开点选在 $(x_0,y_0)=(1,4)$：自变量 $x$ 增加了 $0.08$，自变量 $y$ 减少了 $0.04$，也就是

$$
\Delta x=0.08,\qquad \Delta y=-0.04
$$

于是问题变成计算 $f(x_0+\Delta x,\,y_0+\Delta y)$。

> **易错点**：指数是从 $4$ 变到 $3.96$，是减少，所以 $\Delta y=-0.04$，负号不能丢。

**截断到二次项。** 泰勒公式取到二次项为止，设想这个值跟真值比较接近、误差不太大；当然，如果能算到三次项、四次项，那就更好：

$$
f(x_0+\Delta x,\,y_0+\Delta y)\approx f(x_0,y_0)+f_x\,\Delta x+f_y\,\Delta y+\frac{1}{2!}\left(f_{xx}\,\Delta x^{2}+2f_{xy}\,\Delta x\,\Delta y+f_{yy}\,\Delta y^{2}\right)
$$

其中各个偏导数都在 $(x_0,y_0)$ 处取值。

**先算函数值和五个偏导数。** $f(1,4)=1^{4}=1$，这个容易；五个偏导数逐一求出来：

| 偏导数 | 表达式 | 在 $(1,4)$ 的值 |
|:---:|:---:|:---:|
| $f_x$ | $y\,x^{y-1}$ | $4$ |
| $f_y$ | $x^{y}\ln x$ | $0$ |
| $f_{xx}$ | $y(y-1)\,x^{y-2}$ | $12$ |
| $f_{xy}$ | $x^{y-1}+y\,x^{y-1}\ln x$ | $1$ |
| $f_{yy}$ | $x^{y}(\ln x)^{2}$ | $0$ |

其中 $f_{xy}$ 是把 $f_x=yx^{y-1}$ 再对 $y$ 求导：第一项对 $y$ 求导得 $x^{y-1}$，第二项求导得 $yx^{y-1}\ln x$；在 $(1,4)$ 处 $\ln 1=0$，所以只剩下 $1$。有两个偏导数是零（$f_y$ 与 $f_{yy}$），说明展开式里对应的两项根本不需要算。

**代入求值。**

$$
1.08^{3.96}\approx 1+4\times 0.08+\frac{1}{2}\Big[12\times 0.08^{2}+2\times 1\times 0.08\times(-0.04)+0\Big]=1.3552
$$

精确值是 $1.3563\ldots$，两者相差约 $0.001$——第二位小数是精确的，这个误差已经比较小。

> **提示**：这道题选 $(1,4)$ 作展开点的好处很清楚：$1^{4}=1$、$\ln 1=0$，函数值与各阶偏导数都好算，还有两项直接归零，二次截断就够用了。

### 函数显式写不出来时：数值求偏导

实际当中经常要算偏导数在某一点的值，还包括二阶偏导在某点的值。但常常碰到两种情况：函数的显式根本写不出来；或者函数非常复杂。这时函数值倒不怕——给计算机一个点，它就能一点一点把值算出来——麻烦在算偏导：得先求出偏导函数，再代点求值，这就不太好算了。计算机里有这样一个办法，专治这个问题：用泰勒公式，从函数值把偏导数的值凑出来。

**一阶偏导。** 看 $f\left(x_0+\frac{h}{2},\,y_0\right)$，用泰勒公式展开。注意这里 $\Delta x=\frac{h}{2}$、$\Delta y=0$，所以 $f_y$ 那一项自动消失：

$$
f\left(x_0+\frac{h}{2},\,y_0\right)=f(x_0,y_0)+\frac{h}{2}f_x(x_0,y_0)+O(h^{2})
$$

（余项写成小 $o$ 或者大 $O(h^2)$ 都可以。）再对 $x_0-\frac{h}{2}$ 这边做同样的事：

$$
f\left(x_0-\frac{h}{2},\,y_0\right)=f(x_0,y_0)-\frac{h}{2}f_x(x_0,y_0)+O(h^{2})
$$

两式相减：函数值 $f(x_0,y_0)$ 抵消，两个 $\frac{h}{2}$ 相加变成 $h$，于是

$$
f_x(x_0,y_0)\approx\frac{f\left(x_0+\frac{h}{2},\,y_0\right)-f\left(x_0-\frac{h}{2},\,y_0\right)}{h}
$$

在计算机里，如果求 $f$ 在 $(x_0,y_0)$ 的偏导比较困难，就用这个式子算出偏导的近似值。同样地，关于 $y$ 的偏导有

$$
f_y(x_0,y_0)\approx\frac{f\left(x_0,\,y_0+\frac{h}{2}\right)-f\left(x_0,\,y_0-\frac{h}{2}\right)}{h}
$$

**二阶偏导。** $f_{xx}$ 是什么？就是一阶偏导再求一次偏导。按刚才的办法，把公式里的 $f$ 换成 $f_x$ 用一次：

$$
f_{xx}(x_0,y_0)\approx\frac{f_x\left(x_0+\frac{h}{2},\,y_0\right)-f_x\left(x_0-\frac{h}{2},\,y_0\right)}{h}
$$

然后右边的 $f_x$ 还可以再用上面的一阶公式代进去：$f_x\left(x_0+\frac{h}{2},y_0\right)\approx\dfrac{f(x_0+h,y_0)-f(x_0,y_0)}{h}$，$f_x\left(x_0-\frac{h}{2},y_0\right)\approx\dfrac{f(x_0,y_0)-f(x_0-h,y_0)}{h}$。整理出来就是

$$
f_{xx}(x_0,y_0)\approx\frac{f(x_0+h,\,y_0)-2f(x_0,y_0)+f(x_0-h,\,y_0)}{h^{2}}
$$

关于 $y$ 的二阶偏导也可以同样算。

**五点差分格式。** 实际中还经常要算 $\dfrac{\partial^{2}f}{\partial x^{2}}+\dfrac{\partial^{2}f}{\partial y^{2}}$ 这个表达式在某点的值，把上面两个二阶公式加起来：

$$
f_{xx}+f_{yy}\approx\frac{f(x_0+h,\,y_0)+f(x_0-h,\,y_0)+f(x_0,\,y_0+h)+f(x_0,\,y_0-h)-4f(x_0,y_0)}{h^{2}}
$$

看看公式里这些点在平面上的位置：以 $(x_0,y_0)$ 为中心画一个正方形，

```text
                (x0, y0+h)
                    |
                    |
  (x0-h, y0) ---- (x0, y0) ---- (x0+h, y0)
                    |
                    |
                (x0, y0-h)
```

读图：中心一点，加上下左右共五个点，这个公式恰好只用这五个点上的函数值。所以在近似计算当中，它称为五点差分格式。这些都是计算数学里面的内容，以后在计算数学里都会学到——数学分析的许多许多结果，都是可以适用于近似计算的。

### 四道例题：复合函数求偏导的综合练习

关于这一节——中值定理与泰勒公式——的内容到这里讲好了。下面举几道例题，把多元微分学里的偏导、全微分、复合函数求偏导的公式综合练一遍。

#### 单位圆周上至少两点

> **题目**：设 $f(x,y)$ 具有连续偏导数，且 $f(1,0)=f(0,1)$。证明：在单位圆周 $x^{2}+y^{2}=1$ 上至少存在两点，满足 $y f_x=x f_y$。

拿到题目先想：$f$ 是个二元函数，但到了圆周上，我们可以用什么方法来表达它？圆周的参数表示正好是 $x=\cos\theta$、$y=\sin\theta$。把参数表示代到 $f(x,y)$ 里去，得到 $f(\cos\theta,\sin\theta)$——这就是这个函数在圆周上的表示。$f$ 原是 $x,y$ 的函数，$x$ 又是 $\theta$ 的函数，$y$ 也是 $\theta$ 的函数，这是一个多元复合函数；但一复合，就变成一个一元函数，记为

$$
g(\theta)=f(\cos\theta,\,\sin\theta)
$$

看 $g$ 在几个特殊点的值：$\theta=0$ 时正好是 $f$ 在 $(1,0)$ 的值，$\theta=\frac{\pi}{2}$ 时正好是 $f$ 在 $(0,1)$ 的值，$\theta=2\pi$ 时又回到了 $f(1,0)$：

$$
g(0)=f(1,0),\qquad g\!\left(\frac{\pi}{2}\right)=f(0,1),\qquad g(2\pi)=f(1,0)
$$

条件 $f(1,0)=f(0,1)$ 恰好给出 $g(0)=g\!\left(\frac{\pi}{2}\right)=g(2\pi)$。$f$ 具有连续偏导，$g$ 肯定是可导的，于是对 $g$ 用罗尔定理：在 $\left[0,\frac{\pi}{2}\right]$ 上两端点函数值相等，存在 $\theta_1\in\left(0,\frac{\pi}{2}\right)$ 使 $g'(\theta_1)=0$；在 $\left[\frac{\pi}{2},2\pi\right]$ 上两端点函数值也相等，存在 $\theta_2\in\left(\frac{\pi}{2},2\pi\right)$ 使 $g'(\theta_2)=0$。

$g'$ 按复合函数求导公式来算：

$$
g'(\theta)=f_x\cdot(-\sin\theta)+f_y\cdot\cos\theta=x f_y-y f_x
$$

（在圆周上 $\sin\theta=y$、$\cos\theta=x$。）所以 $g'(\theta_1)=g'(\theta_2)=0$ 就是 $y f_x=x f_y$。$\theta_1$ 一定落在 $\left(0,\frac{\pi}{2}\right)$ 这段弧上，$\theta_2$ 一定落在 $\left(\frac{\pi}{2},2\pi\right)$ 这段弧上，对应圆周上两个不同的点——所以在圆周上至少有两个点满足等式。注意结论的分寸：不是说等式在圆周上处处成立，而是至少有两点使它成立。

#### 等式处处成立：$f$ 只与半径有关

> **题目**：设 $f(x,y)$ 满足 $y f_x=x f_y$，对平面上每一点都成立。证明：$f(x,y)=\varphi(r)$，其中 $r=\sqrt{x^{2}+y^{2}}$。

先看结论是什么意思。$f$ 定义在整个平面上，"只与 $r$ 有关"就是说：固定一个 $r$，函数在这个圆周上的值全相等；换一个 $r$，函数值可以不同；但只要 $r$ 相同，函数值就相同。

证明还是走极坐标：设 $x=r\cos\theta$、$y=r\sin\theta$，这里 $r$ 正好是 $\sqrt{x^{2}+y^{2}}$。这次让 $f$ 对 $\theta$ 求偏导：

$$
\frac{\partial f}{\partial\theta}=f_x\cdot(-r\sin\theta)+f_y\cdot(r\cos\theta)=-y f_x+x f_y=0
$$

最后一步用的正是条件。$\dfrac{\partial f}{\partial\theta}=0$ 说明什么？这个函数跟 $\theta$ 无关。跟 $\theta$ 无关，那就只跟 $r$ 有关，所以 $f(x,y)=\varphi(r)$。

#### $x f_x+y f_y=0$：$f$ 恒为常数

> **题目**：设 $f(x,y)$ 具有连续偏导数（可微等所需条件都满足），且满足 $x f_x+y f_y=0$。证明：$f$ 恒等于常数。

同样把 $f$ 写成 $f(r\cos\theta,\,r\sin\theta)$，这次对 $r$ 求偏导：

$$
\frac{\partial f}{\partial r}=f_x\cos\theta+f_y\sin\theta=\frac{1}{r}\left(x f_x+y f_y\right)=0
$$

提出一个 $\frac{1}{r}$，括号里恰好是条件的左边。这说明 $f$ 跟 $r$ 无关——$f$ 是幅角 $\theta$ 的函数，记 $f(x,y)=\psi(\theta)$。

到这里，条件已经用完了，只能得到"$f$ 是 $\theta$ 的函数"，还不能得到"$f$ 恒等于常数"。那怎么办？再想想看。

把式子写成 $f(r\cos\theta,\,r\sin\theta)=\psi(\theta)$，然后令 $r$ 取 $0$，$\theta$ 不动：左边就成了 $f(0,0)$。注意这个函数只跟幅角有关、跟 $r$ 没关系，而它在 $(0,0)$ 这一点还要连续，所以令 $r$ 取 $0$ 马上得到

$$
f(0,0)=\psi(\theta)\qquad\text{对一切 }\theta
$$

$\psi(\theta)$ 就是常数 $f(0,0)$，于是 $f(x,y)\equiv f(0,0)$，恒等于常数。能满足这个式子的条件，确实是很强的。

#### 从混合偏导等式到变量分离

> **题目**：设 $u(x,y)$ 满足 $u\,u_{xy}=u_x\,u_y$。证明：$u(x,y)=f(x)\,g(y)$，即 $x$ 与 $y$ 可以分离——当然 $f$、$g$ 是可导的函数，它们具体是什么样的函数我们不知道，但可以肯定 $u$ 能分成 $x$ 的函数与 $y$ 的函数相乘。

做这类题的关键，是看清楚条件该怎么用。这里考虑 $\dfrac{u_x}{u}$ 对 $y$ 求偏导，按商的求导法则，分子上"一个求导、一个不求导"相减：

$$
\frac{\partial}{\partial y}\left(\frac{u_x}{u}\right)=\frac{u_{xy}\cdot u-u_x\cdot u_y}{u^{2}}=0
$$

分子恰好就是条件的左边。有些式子就是这样，你要知道怎么去用它。

$\dfrac{u_x}{u}$ 关于 $y$ 的偏导数等于零，说明这个函数跟 $y$ 无关，是一个只含 $x$ 的函数，记作 $\varphi(x)$。而 $\dfrac{u_x}{u}$ 恰好是 $\ln u$ 关于 $x$ 的偏导数，于是两边对 $x$ 积分：

$$
\ln u=\int\varphi(x)\,dx=F(x)+G(y)
$$

注意积分出来的"常数"应该是 $y$ 的函数，记作 $G(y)$。两边取指数：

$$
u=e^{F(x)+G(y)}=e^{F(x)}\cdot e^{G(y)}=f(x)\,g(y)
$$

变量分离完成。

---

这几道例题用的全是同一套基本功：复合函数求偏导——参数化之后的 $g(\theta)$、极坐标下的 $\dfrac{\partial f}{\partial\theta}$ 与 $\dfrac{\partial f}{\partial r}$，都是链式法则的直接应用。微分学里求偏导、求全微分这些运算，大家要搞得很熟练。这一节到这里就结束了，下学期从第 4 节"隐函数存在定理"开始。作业：书上 170 页第 6 题（上次第 1、2、3 题已经做过），这次不交，练习一下即可。

> **承前启后**：上一节讲完「泰勒公式的应用与多元微分学例题」，下一节接着讲「空间曲线的切线与法平面」。

---

## 空间曲线的切线与法平面
> 对应块：BLK142 | 覆盖分集：P149 | 块标题（分集名）：《151.第12章 第5节 偏导数在几何中的应用（》

曲线画得出来，问题也跟着来了：这条割线的方程怎么写？入手点还是那个老想法——曲线也好，直线也好，把上面的动点记成 $(x, y, z)$，去求 $x, y, z$ 满足的关系式，而这个关系式就是这条线的方程。这一讲就沿着这条路走：从割线方程出发，把空间曲线的切线与法平面一次说清楚，最后落到一个很好用的结论——两张曲面交出的曲线，它的法平面可以由两个梯度张成。

### 从割线的方程到切线的方程

设曲线 $\gamma$ 用参数方程给出：

$$x = x(t), \qquad y = y(t), \qquad z = z(t)$$

在曲线上取定一点 $P_0$，坐标为 $(x(t_0),\, y(t_0),\, z(t_0))$；再取一个参数值 $t_1$，对应曲线上另一点 $P_1$。过 $P_0$、$P_1$ 的直线就是割线。现在问：割线上的动点 $P = (x, y, z)$ 要满足什么关系式？

连两个向量，起点都放在 $P_0$：一个以 $P_1$ 为终点，一个以 $P$ 为终点。这两个向量什么特点？方向相同，但模可以不一样——因为 $(x, y, z)$ 在直线上动。方向相同，用式子说就是

$$\vec{P_0P} = k\,\vec{P_0P_1}$$

$k$ 是任意实数，而且可以取负值——动点完全可能跑到 $P_0$ 的另一头去。

这个向量等式翻译成分量的语言：左右两个向量各有三个分量，等式表明的是三个分量**成比例**——第一个分量比第一个分量，第二个比第二个，第三个比第三个，三个比值相等：

$$\frac{x - x_0}{x(t_1) - x(t_0)} = \frac{y - y_0}{y(t_1) - y(t_0)} = \frac{z - z_0}{z(t_1) - z(t_0)}$$

这就是割线的方程：分子是动点坐标减去 $P_0$ 的坐标，分母是 $P_1$ 与 $P_0$ 对应坐标之差。

那么切线方程怎么来？令 $t_1 \to t_0$。可是一代进去，三个分母全变成零——这是 $0 \div 0$，没意义。怎么办？把三个分母同除以 $t_1 - t_0$：等式左、中、右在分母上除掉的是同一个非零数，比例关系不变，等式照样成立：

$$\frac{x - x_0}{\dfrac{x(t_1) - x(t_0)}{t_1 - t_0}} = \frac{y - y_0}{\dfrac{y(t_1) - y(t_0)}{t_1 - t_0}} = \frac{z - z_0}{\dfrac{z(t_1) - z(t_0)}{t_1 - t_0}}$$

现在让 $t_1 \to t_0$：$P_1$ 沿曲线向 $P_0$ 靠拢，割线绕着 $P_0$ 转，极限位置就是切线；三个分母分别趋于 $x'(t_0)$、$y'(t_0)$、$z'(t_0)$。于是

$$\frac{x - x_0}{x'(t_0)} = \frac{y - y_0}{y'(t_0)} = \frac{z - z_0}{z'(t_0)}$$

这就是曲线在 $P_0$ 点的切线方程。

### 分母为零时怎么办

> **易错点**：按光滑曲线的要求，$x'(t_0)$、$y'(t_0)$、$z'(t_0)$ 只是**不同时为零**——完全可以有一个等于零，甚至两个同时为零。分母一为零，上面的比例式就不能照写了，切线方程得换个写法。

先换个角度看清这个比例式：三个比式两两相等，实际只给出**两个等式**。在三维空间里，一个等式表示一张曲面，两个等式就是两张曲面的交线——空间里的一条直线，本来就应当这样来理解。

若恰有一个分母为零，比如 $z'(t_0) = 0$：第三个比式的分母已经是零，这个比要成立，只能要求分子也是零，也就是 $z = z_0$——若 $z \neq z_0$，这个等号根本不可能成立。而 $z = z_0$ 本身就表示一张平面，也就是一张曲面。于是切线方程写成

$$z = z_0, \qquad \frac{x - x_0}{x'(t_0)} = \frac{y - y_0}{y'(t_0)}$$

第一个等式是一张曲面，第二个等式是另一张曲面，切线就是它们的交线。

若恰有两个分母为零，比如 $z'(t_0) = 0$ 且 $x'(t_0) = 0$：同样地，$z$ 只能等于 $z_0$，$x$ 只能等于 $x_0$，中间那个比式 $(y - y_0)/y'(t_0)$ 随便等于什么都可以，不用管它。于是

$$x = x_0, \qquad z = z_0$$

也就是说，在 $P_0 = (x_0, y_0, z_0)$ 这一点，切线方程实际上就是两张平面的交线。

| 分母情况 | 切线的联立表示 |
| :-- | :-- |
| 三个全不为零 | $\frac{x-x_0}{x'(t_0)}=\frac{y-y_0}{y'(t_0)}=\frac{z-z_0}{z'(t_0)}$ |
| 恰有一个为零（如 $z'(t_0)=0$） | $z=z_0$ 与 $\frac{x-x_0}{x'(t_0)}=\frac{y-y_0}{y'(t_0)}$ 联立 |
| 恰有两个为零（如 $x'(t_0)=z'(t_0)=0$） | $x=x_0$ 与 $z=z_0$ 联立 |

### 切向量与法平面

把这三个导数拼成一个向量——

> **定义**：$\vec{\tau} = x'(t_0)\,\vec{i} + y'(t_0)\,\vec{j} + z'(t_0)\,\vec{k}$ 称为曲线 $\gamma$ 在点 $(x(t_0), y(t_0), z(t_0))$（也就是 $P_0$）处的**切向量**，往往用 $\tau$ 来表示。

这个向量非常重要：有了切向量，切线的方程马上就出来了。

曲线还有一个很重要的概念：法平面。

> **定义**：过 $P_0$、与切向量垂直（也就是与切线垂直）的平面，称为曲线在该点的**法平面**。

法平面的方程能不能求？套路跟刚才完全一样：把平面上的动点记为 $P = (x, y, z)$，求 $x, y, z$ 满足的关系式。$P$ 在法平面上，就意味着向量 $\vec{P_0P} = (x - x_0,\ y - y_0,\ z - z_0)$ 与切向量 $\vec{\tau} = (x'(t_0),\ y'(t_0),\ z'(t_0))$ 正交，点乘等于零：

$$x'(t_0)(x - x_0) + y'(t_0)(y - y_0) + z'(t_0)(z - z_0) = 0$$

这就是曲线 $\gamma$ 在 $P_0$ 点的法平面方程。

体会一下：空间曲线的切线与法平面，最关键的就是**切向量**。只要切向量出来了，切线方程、法平面方程马上就能写出来。而切向量要出来，首先三个分量不能同时为零；再就是我们讨论的是光滑曲线——三个导数 $x'(t)$、$y'(t)$、$z'(t)$ 连续，而且导向量 $\vec{r}\,'(t) = (x'(t), y'(t), z'(t))$（正是切向量那个表达式）不是零向量。在这个条件下，曲线是光滑曲线。

### 曲线写成两个柱面的交线

参数表示还没有把曲线的表示讲完——曲线的表示有好几种。比如 $\gamma$ 可以写成

$$y = y(x), \qquad z = z(x), \qquad x \in [a, b]$$

这两个式子各表示什么？先想清楚一件事：在三维空间里，一个等式表示一张曲面。$y = y(x)$ 里面没有 $z$，它就是母线平行于 $z$ 轴的柱面；$z = z(x)$ 里面没有 $y$，它是母线平行于 $y$ 轴的柱面。这两张柱面一交，交出的曲线就是 $\gamma$。

要它的切线与法平面，怎么办？往上套：在 $y = y(x)$、$z = z(x)$ 上面添一行 $x = x$，这就是刚才的参数方程——参数就取 $x$ 本身。切向量立刻写出：

$$\vec{\tau} = \left(1,\ y'(x_0),\ z'(x_0)\right)$$

第一个分量是 $1$，因为 $x$ 对 $x$ 求导当然是 $1$。于是切线方程

$$\frac{x - x_0}{1} = \frac{y - y_0}{y'(x_0)} = \frac{z - z_0}{z'(x_0)}$$

（分母最好都写成导数在 $x_0$ 处的值。）法平面方程

$$(x - x_0) + y'(x_0)(y - y_0) + z'(x_0)(z - z_0) = 0$$

一条曲线若是通过这样的两个方程来表示，它的切线与法平面同样可以得到。

### 一般情形：两张曲面的交线

曲线既然可以看成两张曲面的交线，最一般的写法就是

$$\gamma:\quad F(x, y, z) = 0, \qquad G(x, y, z) = 0$$

这是两张一般的曲面，一交交出一条曲线。设 $P_0 = (x_0, y_0, z_0)$ 属于 $\gamma$，也就是说 $F$、$G$ 在这一点都等于零。现在要求过 $P_0$ 的切线与法平面。

先说对 $F$、$G$ 加什么条件。第一，$F$、$G$ 有连续偏导数——这样一来，下面这个雅克比（Jacobian）矩阵的每个元素都是连续的。第二，这个矩阵

$$\begin{pmatrix} F_x & F_y & F_z \\ G_x & G_y & G_z \end{pmatrix}$$

**满秩**。它是几行几列？$F$、$G$ 是两个函数（因变量两个），自变量是三个 $x, y, z$，所以是两行三列。满秩就是说秩等于 $2$：它的二阶子行列式一共有三个，其中至少有一个不等于零。这个条件派什么用处？它可以保证这条曲线是一条光滑曲线。

$x$、$y$、$z$ 三个变量的地位是对称的，不妨就设在 $P_0$ 这一点，$F$、$G$ 关于 $y, z$ 的雅克比行列式不等于零：

$$\frac{\partial(F, G)}{\partial(y, z)} = \begin{vmatrix} F_y & F_z \\ G_y & G_z \end{vmatrix} \neq 0$$

那么马上可以应用**隐函数存在定理**：$F$、$G$ 关于 $y, z$ 的雅克比行列式不等于零，$y$、$z$ 就可以解成 $x$ 的函数——在 $(x_0, y_0, z_0)$ 附近、也就是 $x_0$ 的邻域里，存在唯一确定的隐函数

$$y = y(x), \qquad z = z(x)$$

注意：原来方程组里一共三个变量，现在 $y$、$z$ 都成了 $x$ 的函数。这个隐函数的表达式一般是写不出来的，但它的导数可以写出来——这在讲隐函数的时候就讲过了。

$\dfrac{dy}{dx}$ 与 $\dfrac{dz}{dx}$ 怎么求？用隐函数存在定理里那套导数公式：分母总是那个不等于零的 $\dfrac{\partial(F, G)}{\partial(y, z)}$，分子换成相应的雅克比行列式：

$$\frac{dy}{dx} = -\,\frac{\partial(F,G)/\partial(x,z)}{\partial(F,G)/\partial(y,z)}, \qquad \frac{dz}{dx} = -\,\frac{\partial(F,G)/\partial(y,x)}{\partial(F,G)/\partial(y,z)}$$

那个负号看着别扭，把它吸收进行列式里去：行列式对换两列差一个负号，把 $x$ 挪个位置，负号就没有了：

$$\frac{dy}{dx} = \frac{\partial(F,G)/\partial(z,x)}{\partial(F,G)/\partial(y,z)}, \qquad \frac{dz}{dx} = \frac{\partial(F,G)/\partial(x,y)}{\partial(F,G)/\partial(y,z)}$$

于是在 $x_0$ 附近，这条曲线就是 $y = y(x)$、$z = z(x)$ 这个表示，在 $P_0$ 点切向量是 $\left(1,\ y'(x_0),\ z'(x_0)\right)$。分母 $\dfrac{\partial(F,G)}{\partial(y,z)}$ 在 $P_0$ 不等于零，把三个分量同乘它，就得到一个对称的写法：

$$\vec{\tau} = \left( \frac{\partial(F,G)}{\partial(y,z)},\ \frac{\partial(F,G)}{\partial(z,x)},\ \frac{\partial(F,G)}{\partial(x,y)} \right)_{P_0}$$

> **提示**：这三个雅克比行列式不用死记。有同学觉得这一套蛮难记，其实记住一条就够：分母那一对必须是 $(y, z)$，**不能含有 $x$**——$x$ 是被单独拎出来的那个变量，这里放了 $x$，$y$、$z$ 那两对就摆不平了。分母取了 $(y, z)$，后面按轮换跟上：$(z, x)$、$(x, y)$。每一对内部的次序也能换，但一换就差一个负号，换的时候要留神。记住这一点，不用翻书，自己就能把公式写出来。

切线与法平面马上写出（所有雅克比行列式都取 $P_0$ 处的值）：

$$\frac{x - x_0}{\left.\dfrac{\partial(F,G)}{\partial(y,z)}\right|_{P_0}} = \frac{y - y_0}{\left.\dfrac{\partial(F,G)}{\partial(z,x)}\right|_{P_0}} = \frac{z - z_0}{\left.\dfrac{\partial(F,G)}{\partial(x,y)}\right|_{P_0}}$$

$$\left.\frac{\partial(F,G)}{\partial(y,z)}\right|_{P_0}(x - x_0) + \left.\frac{\partial(F,G)}{\partial(z,x)}\right|_{P_0}(y - y_0) + \left.\frac{\partial(F,G)}{\partial(x,y)}\right|_{P_0}(z - z_0) = 0$$

换句话说，对空间一条曲线，它的不同的表示我们都可以写出它的切向量；而写出了切向量，切线与法平面的方程都能写出来：

```text
曲线的三种表示与对应的切向量（均在 P0 处取值）

  参数式     x=x(t), y=y(t), z=z(t)   ->   τ = ( x'(t0), y'(t0), z'(t0) )
  柱面交线   y=y(x), z=z(x)           ->   τ = ( 1, y'(x0), z'(x0) )
  一般式     F=0, G=0                 ->   τ = ( D(y,z), D(z,x), D(x,y) )

  其中 D(y,z) 即 ∂(F,G)/∂(y,z) 在 P0 的值。切向量一旦写出：

  切线    (x-x0)/τ1 = (y-y0)/τ2 = (z-z0)/τ3
  法平面  τ1*(x-x0) + τ2*(y-y0) + τ3*(z-z0) = 0
  （τ 某个分量为零时，按前面"两张曲面联立"的方式写）
```

这张图把三种表示与切向量的对应收拢在一起：表示方式可以变，"先求切向量"这把钥匙不变。

### 两个梯度恰好张成法平面

再讲一个定理。曲线

$$\gamma:\quad F(x, y, z) = 0, \qquad G(x, y, z) = 0, \qquad P_0 \in \gamma$$

过 $P_0$ 作 $\gamma$ 的法平面——就是跟切线垂直的那张平面。定理说：过 $P_0$ 的这张法平面，可以由两个向量张成：一个是 $\operatorname{grad} F$ 在 $P_0$ 的值，一个是 $\operatorname{grad} G$ 在 $P_0$ 的值。换句话说，过这一点做两个向量，一个 gradient $F$，一个 gradient $G$，这两个向量张成的就是法平面。

条件还是刚才那两条：$F$、$G$ 有连续偏导数；雅克比矩阵 $\begin{pmatrix} F_x & F_y & F_z \\ G_x & G_y & G_z \end{pmatrix}$ 满秩。

证明来看一下。这个 $2 \times 3$ 的雅克比矩阵，两行恰好就是两个向量：第一行就是 $\operatorname{grad} F = (F_x, F_y, F_z)$，第二行就是 $\operatorname{grad} G = (G_x, G_y, G_z)$。满秩是什么意思？这两个向量**线性无关**。这里首先要排除一件事：要是这两个向量是平行的，它们能张成一个平面吗？张不成。满秩恰恰说明了这两个向量线性无关，所以它们可以张成一个平面。

接下来要说明：它们张成的平面恰好是法平面。怎么说？只要证明这两个向量都跟切向量垂直就可以了。切向量就是刚才那个

$$\vec{\tau} = \left( \frac{\partial(F,G)}{\partial(y,z)},\ \frac{\partial(F,G)}{\partial(z,x)},\ \frac{\partial(F,G)}{\partial(x,y)} \right)_{P_0}$$

算点积（以下各行列式都在 $P_0$ 取值）：

$$\operatorname{grad} F \cdot \vec{\tau} = F_x \frac{\partial(F,G)}{\partial(y,z)} + F_y \frac{\partial(F,G)}{\partial(z,x)} + F_z \frac{\partial(F,G)}{\partial(x,y)}$$

把这个式子认成三阶行列式按第一行的展开——高等代数里都学过：

$$F_x \begin{vmatrix} F_y & F_z \\ G_y & G_z \end{vmatrix} - F_y \begin{vmatrix} F_x & F_z \\ G_x & G_z \end{vmatrix} + F_z \begin{vmatrix} F_x & F_y \\ G_x & G_y \end{vmatrix} = \begin{vmatrix} F_x & F_y & F_z \\ F_x & F_y & F_z \\ G_x & G_y & G_z \end{vmatrix} = 0$$

为什么等于零？这个三阶行列式**前两行一模一样**——两行相同，行列式为零。（中间一项前面添的负号，恰好把 $\partial(F,G)/\partial(x,z)$ 翻成了 $\partial(F,G)/\partial(z,x)$——当初"去负号、换次序"的功夫在这里兑了现。）同理，$\operatorname{grad} G \cdot \vec{\tau}$ 也等于零。

于是两个梯度都与切向量垂直，而它们又线性无关，所以它们张成的平面恰好是过 $P_0$ 的法平面，定理得证。

一句话收尾：若曲线是 $F = 0$ 与 $G = 0$ 的交线，那么过 $P_0$ 的法平面可以由两个梯度张成，这两个梯度张成的平面恰好是曲线在 $P_0$ 点的法平面，而且这两个梯度都与曲线在 $P_0$ 点的切向量垂直。

> **承前启后**：上一节讲完「空间曲线的切线与法平面」，下一节接着讲「从螺旋线到曲面的切平面与法线」。

---

## 从螺旋线到曲面的切平面与法线
> 对应块：BLK143 | 覆盖分集：P150 | 块标题（分集名）：《152.第12章 第5节 偏导数在几何中的应用（》

上一讲把空间曲线在一点的切线与法平面讲完了，几种表示下切向量的公式都在手上。这一讲先用两道例题把"过一点求切线与法平面"练熟，再转入真正的重头戏——偏导数在曲面上的应用：曲面的切平面与法线。

### 例题：螺旋线的轨迹、速度与切平面

头一道例题是一个很好看的运动：一个质点绕 $z$ 轴以角速度 $\omega$ 逆时针旋转，同时以匀速 $C$ 上升。质点的运动是两个运动的合成——一个沿 $z$ 轴上升，一个绕 $z$ 轴转，转出来的轨迹就像一根弹簧。题目问三件事：

- 质点的运动轨迹，换句话说，把这条曲线的方程写出来；
- 质点运动的速度（注意它是个向量）；
- 当 $\omega=1$、$t=\pi/2$ 时，求对应点上曲线的切线方程与法平面方程。

动笔前先记住一句话：切线方程、法平面方程，说穿了就是把切向量求出来；切向量一出来，两个方程随手就能写。

#### 轨迹：两个运动的合成

设 $t=0$ 时质点位于 $P_0=(A,0,0)$，也就是它到 $z$ 轴的距离为 $A$。记时刻 $t$ 质点的位置向量为 $\boldsymbol r(t)=(x(t),\,y(t),\,z(t))$，把三个坐标一个一个求出来。

把质点竖直投影到 $xy$ 平面，记投影点为 $M$，设 $OM$ 与 $x$ 轴的夹角为 $\theta$。旋转是匀角速的，所以 $\theta=\omega t$，于是质点的横、纵坐标就是 $M$ 的坐标：

$$x=A\cos\omega t,\qquad y=A\sin\omega t.$$

再想 $z$ 等于什么——按题意以匀速 $C$ 上升，所以 $z=Ct$。合起来，这条曲线的方程也就是质点的运动轨迹：

$$\boldsymbol r(t)=\big(A\cos\omega t,\; A\sin\omega t,\; Ct\big).$$

转一圈、抬一段，弹簧的形状就出来了。

#### 速度就是切向量

第二个问题问速度。那么这个速度是什么？速度既有大小又有方向，是个向量；而轨迹的参变量 $t$ 本来就是时间，关于 $t$ 求导，得到的就是速度——换句话说，速度向量就是曲线的切向量：

$$\boldsymbol v=\boldsymbol r\,'(t)=\big(-A\omega\sin\omega t,\; A\omega\cos\omega t,\; C\big).$$

质点的运动速度就是这个向量。这一问顺带把几何量（切向量）与物理量（速度）接在了一起。

#### 代入 $\omega=1$、$t=\pi/2$：切线与法平面

第三问给了具体的参数值。先把对应点算出来——$\omega=1$、$t=\dfrac{\pi}{2}$ 代入轨迹，得

$$P=\left(0,\; A,\; \frac{\pi C}{2}\right).$$

再把这点处的切向量算出来——对 $\boldsymbol r\,'(t)$ 取 $\omega=1$、$t=\dfrac{\pi}{2}$：

$$\boldsymbol T=(-A,\; 0,\; C).$$

点有了、切向量有了，切线方程照对称式写：

$$\frac{x-0}{-A}=\frac{y-A}{0}=\frac{z-\dfrac{\pi C}{2}}{C}.$$

分母里出现了零，怎么办？这不是无解，而是提醒你这一项要单独处理：分母为零意味着 $y-A=0$ 恒成立，即 $y=A$；把这一项拎出来，剩下两个比值相等：

$$\frac{x}{-A}=\frac{z-\dfrac{\pi C}{2}}{C}.$$

> **易错点**：对称式里某个分母为 $0$，不代表直线不存在，而是该坐标恒等于定值。把它单独写成 $y=A$，再联立另外两个比值，才是完整的切线。

法平面以 $\boldsymbol T=(-A,0,C)$ 为法向量：

$$-A\,(x-0)+0\cdot(y-A)+C\left(z-\frac{\pi C}{2}\right)=0,$$

中间 $0\cdot(y-A)$ 那一项直接不写。这道题到此解完。回过头看会发现它很容易：曲线上的点知道，切向量知道，切线与法平面就都写出来了。

### 例题：两张曲面交线的切线与法平面

第二道例题的曲线不给参数方程，给的是两张曲面的交线：

$$x^2+y^2+z^2-2y=4,\qquad x+y+z=0.$$

算之前先看清楚它是一条什么曲线。第一个方程配方：

$$x^2+(y-1)^2+z^2=5,$$

是一个球面——注意球心不在原点；第二个方程是一张平面。一张平面去截球面，截出来的是空间的一个圆。

点 $P(1,1,-2)$ 代进两个方程都满足，所以它在曲线上。求过 $P$ 的切线与法平面。

这条曲线由两个方程共同决定，写成

$$F(x,y,z)=x^2+y^2+z^2-2y-4=0,\qquad G(x,y,z)=x+y+z=0.$$

曲线同时躺在这两张曲面上，它在一点的切向量必须同时垂直于两张曲面的法向量 $
abla F$ 与 $
abla G$，所以切向量就取这两个偏导向量的向量积：

$$\boldsymbol T=
abla F\times
abla G.$$

其中 $F_x=2x$，$F_y=2y-2=2(y-1)$，$F_z=2z$，而 $G_x=G_y=G_z=1$。三个分量依次算：

$$\begin{aligned}
T_x&=F_yG_z-F_zG_y=2(y-1)-2z=2(y-z-1),\\
T_y&=F_zG_x-F_xG_z=2z-2x=2(z-x),\\
T_z&=F_xG_y-F_yG_x=2x-2(y-1)=2(x-y+1).
\end{aligned}$$

> **易错点**：这三个分量表达式里的 $x,y,z$ 最后必须用 $P_0$ 的坐标代进去，得到的才是过 $P$ 点的那个具体切向量——这一步别漏。

代入 $P(1,1,-2)$：

$$\boldsymbol T=\big(2(1+2-1),\;2(-2-1),\;2(1-1+1)\big)=(4,\,-6,\,2).$$

切线方程：

$$\frac{x-1}{4}=\frac{y-1}{-6}=\frac{z+2}{2}.$$

三个分母有公因子 $2$ 可以约，这里不约，照原样写。法平面方程：

$$4(x-1)-6(y-1)+2(z+2)=0.$$

这两个方程都还能进一步化简，化简留给大家自己动手。

### 曲面的切平面与法线

两道例题算完，进入这一讲的正题。曲线那边，最重要的两个对象是切线与法平面；曲面这边，换成切平面与法线。曲线有好几种表示，对每一种我们都求出了切向量；曲面同样有多种表示，先从最通用的一种出发：

$$F(x,y,z)=0.$$

这一个方程在空间里就表示了一张曲面。要讨论它的切平面与法线，先对 $F$ 加两个条件：

- $F$ 有连续偏导数。曲线那边要求"光滑"，曲面这边同样希望它是光滑曲面——光滑曲面究竟指什么，一会儿用例子说清楚；
- 向量 $(F_x,F_y,F_z)$ 不是零向量，即三个分量不同时为零。把三个偏导数排成一行，也可以看成 $F$ 关于 $(x,y,z)$ 的雅可比矩阵——只有一行的雅可比矩阵。

这个向量到底代表什么？马上见分晓。

#### 曲面上所有曲线的切向量都垂直于同一个向量

在曲面上取定一点 $P_0(x_0,y_0,z_0)$。做这样一件事：在这张曲面上任意画一条光滑曲线 $\Gamma_1$，让它经过 $P_0$。设它的参数方程为

$$x=x(t),\quad y=y(t),\quad z=z(t),$$

并且 $x(t_0)=x_0,\ y(t_0)=y_0,\ z(t_0)=z_0$——这组式子的意思就是：$t=t_0$ 时曲线正好走到 $P_0$。曲线光滑，它在 $P_0$ 就有切向量

$$\boldsymbol\tau_1=\big(x'(t_0),\; y'(t_0),\; z'(t_0)\big).$$

这个切向量有什么性质？注意关键的一句话：这条曲线是画在曲面上的。所以把曲线方程代回曲面方程，得到的是关于 $t$ 的恒等式：

$$F\big(x(t),\,y(t),\,z(t)\big)\equiv 0.$$

两边关于 $t$ 求导（复合函数求导），得

$$F_x\,x'(t)+F_y\,y'(t)+F_z\,z'(t)=0.$$

这个式子说明什么？左边恰好是向量 $(F_x,F_y,F_z)$ 与切向量 $\boldsymbol\tau$ 的内积。注意 $(F_x,F_y,F_z)$ 本来是 $x,y,z$ 的函数，把点 $P_0$ 的坐标代进去，它就成了一个固定向量；它与过 $P_0$ 的这条曲线的切向量内积为零——两者垂直。

再画一条试试。过 $P_0$ 在曲面上另画一条曲线 $\Gamma_2$，它的切向量 $\boldsymbol\tau_2$ 照样垂直于这个固定向量。曲面上过 $P_0$ 的曲线可以画任意多条，而每一条的切向量都垂直于同一个向量 $(F_x,F_y,F_z)\big|_{P_0}$——于是所有这些切向量全落在同一张平面上：过 $P_0$、与这个向量垂直的那张平面。

#### 切平面方程与法线方程

既然过 $P_0$ 的一切切向量都躺在这张平面里，就给它一个名字。

> **定义**：曲面上过 $P_0$ 的一切曲线在 $P_0$ 处的切向量所在的平面，称为曲面在 $P_0$ 点的**切平面**；垂直于切平面的向量 $\boldsymbol n=(F_x,F_y,F_z)\big|_{P_0}$ 称为曲面在 $P_0$ 点的**法向量**。

有了法向量，两个方程都是现成的。

切平面：在平面上任取一点 $(x,y,z)$，从 $P_0$ 指向它的向量 $(x-x_0,\,y-y_0,\,z-z_0)$ 与法向量垂直，内积为零。这个关系式恰好是切平面上每一点的坐标都满足的关系式，也就是切平面的方程：

$$F_x(P_0)\,(x-x_0)+F_y(P_0)\,(y-y_0)+F_z(P_0)\,(z-z_0)=0.$$

法线：过 $P_0$、以法向量 $\boldsymbol n$ 为方向向量的直线：

$$\frac{x-x_0}{F_x(P_0)}=\frac{y-y_0}{F_y(P_0)}=\frac{z-z_0}{F_z(P_0)}.$$

一张曲面只要用 $F(x,y,z)=0$ 给出，切平面方程、法线方程就都出来了。

#### 两个条件为什么这样加

第一，为什么要求 $(F_x,F_y,F_z)$ 不是零向量？假如在 $P_0$ 点三个偏导数同时为零，法向量就没有了——我们不知道它是什么方向，法线的方向就写不出来了。所以这个向量不能是零向量。

第二，为什么要求偏导数连续？连续偏导保证一件事：让点在曲面上移动，法向量（连同切平面）的变化是连续的——这正是"光滑曲面"的含义。反过来看一个不光滑的例子：两张曲面沿一条缝拼接起来，就像一本书打开的样子，左页一张曲面，右页另一张曲面。在这条缝上，法向量从左边移动到右边时会突然跳一下——这里没有光滑性。这样的曲面叫**分片光滑**：每一片内部都光滑，拼缝处不光滑。我们要讨论的是性质好的光滑曲面，所以才加上"偏导连续"这一条。

### 显式曲面 $z=f(x,y)$ 的切平面与法线

曲面最常用的另一种表示是显式的：

$$z=f(x,y),\qquad (x,y)\in D,$$

其中 $D$ 是 $xy$ 平面上的一个区域。对付它非常容易：改写成

$$F(x,y,z)=f(x,y)-z=0,$$

套用刚才的一般公式。梯度是 $(f_x,\,f_y,\,-1)$，法向量就是它：

$$\boldsymbol n=(f_x,\,f_y,\,-1).$$

设 $P_0(x_0,y_0,z_0)$ 在曲面上，即 $z_0=f(x_0,y_0)$。过 $P_0$ 的切平面方程：

$$f_x(x_0,y_0)\,(x-x_0)+f_y(x_0,y_0)\,(y-y_0)-\left(z-z_0\right)=0.$$

法线方程：

$$\frac{x-x_0}{f_x(x_0,y_0)}=\frac{y-y_0}{f_y(x_0,y_0)}=\frac{z-z_0}{-1}.$$

注意第三个分母是 $-1$。曲面用 $z=f(x,y)$ 这种方式表示时，切平面与法线方程一目了然。

### 参数表示的曲面：化回显式

还有一种更复杂的表示。先想一个问题：曲线有参数表示，曲面有没有？曲线 $x=x(t),\,y=y(t),\,z=z(t)$，$t$ 从 $a$ 变到 $b$——$t$ 是一个实数，一个实数在变，描出来的是一条曲线；要在空间里描出一张曲面，一个变量不够，得要两个：

$$x=x(u,v),\quad y=y(u,v),\quad z=z(u,v),\qquad (u,v)\in D\subset\mathbb R^2.$$

$D$ 是 $uv$ 平面上的一个区域。在 $D$ 里取一点 $(u_0,v_0)$，代进去得到空间一点；这些点的像凑起来，就是空间里的一张曲面。

现在设曲面上一点 $P_0(x_0,y_0,z_0)$ 对应参数 $(u_0,v_0)$，即

$$x_0=x(u_0,v_0),\quad y_0=y(u_0,v_0),\quad z_0=z(u_0,v_0).$$

任务是求 $P_0$ 处的切平面与法线方程。关键还是那一步：求出法向量 $\boldsymbol n$——法向量一出来，切平面、法线方程谁都会写。那么 $\boldsymbol n$ 怎么求？

条件照旧加两条：$x,y,z$ 关于 $u,v$ 都有连续偏导；且 $x,y,z$ 关于 $(u,v)$ 的雅可比矩阵的秩为 $2$。秩为 $2$ 是什么意思？雅可比矩阵有三个二阶子行列式，至少有一个不等于零。不妨设

$$\frac{\partial(x,y)}{\partial(u,v)}\Big|_{P_0} \neq 0,$$

也完全可能是 $\dfrac{\partial(x,z)}{\partial(u,v)}$ 或 $\dfrac{\partial(y,z)}{\partial(u,v)}$ 不等于零，道理一样，我们就拿 $(x,y)$ 这一对来说。

把 $x=x(u,v),\ y=y(u,v)$ 看成一个向量值函数：从 $uv$ 平面映到 $xy$ 平面。它的雅可比行列式在该点不等于零，由逆映射定理（上一讲刚讲过），在 $(x_0,y_0)$ 的邻域内存在逆映射：

$$u=u(x,y),\qquad v=v(x,y).$$

把它代回第三个坐标：曲面在局部就成了

$$z=z\big(u(x,y),\,v(x,y)\big)\equiv\bar z(x,y),$$

也就是说，在 $P_0$ 附近，这张曲面就是一张显式曲面——上一节的全套公式直接搬过来用。而恒等式

$$x\big(u(x,y),v(x,y)\big)\equiv x,\qquad y\big(u(x,y),v(x,y)\big)\equiv y$$

两边对 $x,y$ 求导，就可以把 $u,v$ 的全部四个偏导数解出来；有了这些偏导数，用链式法则算出 $\bar z_x,\bar z_y$，法向量 $(\bar z_x,\bar z_y,-1)$ 就到手了，切平面与法线方程随即写出。

这条化归路线值得单独画一遍：

```text
参数曲面：(u, v) ∈ D ⊂ R²  →  (x, y, z) = (x(u,v), y(u,v), z(u,v))
        |
        |  秩条件：三个二阶子行列式至少一个非零
        |  不妨设 ∂(x,y)/∂(u,v) ≠ 0（在 P0 的对应点处）
        v
逆映射定理（局部）：u = u(x, y)，v = v(x, y)
        |
        |  代回第三个坐标：z = z(u(x,y), v(x,y))，记作 Z(x,y)
        v
局部化为显式曲面 z = Z(x, y)
        |
        |  恒等式两边求导，定出 u、v 的全部偏导数
        |  链式法则得 Zx、Zy
        v
法向量 n = (Zx, Zy, -1)  →  写出切平面与法线方程
```

读图说明：这张图把参数曲面的处理思路串成一条链——秩条件挑出一对好用的参数，逆映射定理把它们反解成 $x,y$ 的函数，曲面在局部化成显式 $Z(x,y)$（即正文中的 $\bar z(x,y)$），最终仍归结为 $z=f(x,y)$ 那一套。

到这里，曲面三种表示之间的关系可以并排放在一起看：

| 曲面表示 | 法向量怎么求 |
| :--- | :--- |
| $F(x,y,z)=0$ | 最一般的出发点，直接取梯度 $\boldsymbol n=(F_x,\,F_y,\,F_z)\big|_{P_0}$ |
| $z=f(x,y)$ | 改写成 $f(x,y)-z=0$ 套用上一行，得 $\boldsymbol n=(f_x,\,f_y,\,-1)$ |
| $x=x(u,v),\ y=y(u,v),\ z=z(u,v)$ | 秩为 $2$ 时经逆映射定理局部化成显式，得 $\boldsymbol n=(\bar z_x,\,\bar z_y,\,-1)$ |

参数曲面这一支，把四个偏导数真正解出来、给出法向量的完整公式与算例，就留到下一讲。作业布置在教材第 201 页：第 1 题的 (2)(3)、第 2 题、第 5、6 题。

> **承前启后**：上一节讲完「从螺旋线到曲面的切平面与法线」，下一节接着讲「曲面的参数表示：法向量、切平面与正交曲面」。

---

## 曲面的参数表示：法向量、切平面与正交曲面
> 对应块：BLK144 | 覆盖分集：P151 | 块标题（分集名）：《153.第12章 第5节 偏导数在几何中的应用（》

上节课我们讲了曲面的切平面和法线方程。要写出一张曲面的切平面和法线，关键就是求出这张曲面在一点处的**法向量**。曲面的方程有多种表示，上节课我们已经讲了两种：一种是 $F(x,y,z)=0$，这时法向量是 $(F_x,\,F_y,\,F_z)$；另一种是 $z=f(x,y)$，这时法向量是 $(f_x,\,f_y,\,-1)$。法向量一写出来，曲面上某一点的切平面方程和法线方程就都可以写了。

今天要讲曲面的另外一种表示——**参数表示**。

### 曲面的第三种表示：参数方程

参数表示写成

$$x=x(u,v),\qquad y=y(u,v),\qquad z=z(u,v),\qquad (u,v)\in D,$$

这里的 $D$ 是平面上的一个区域。

可以这样来看：给定一点 $(u,v)$，按照这组方程就得到空间里的一个点 $(x,y,z)$；随着 $(u,v)$ 在区域 $D$ 里变化，$(x,y,z)$ 就在空间中描出了一张曲面。

```text
      参数区域 D（u,v 平面）              曲面 S（x,y,z 空间）
           (u,v) ──────────────────────────► (x,y,z)
                      x = x(u,v)
                      y = y(u,v)      (u,v) 取遍 D，
                      z = z(u,v)      (x,y,z) 就描出整张曲面
```

读图：左边是参数平面上的动点，右边是它映到的空间点；这组三个式子就是这张曲面的**参数方程**。

现在的任务：求曲面上一点 $M_0(x_0,y_0,z_0)$ 处的切平面和法线方程。设这一点的参数是 $(u_0,v_0)\in D$，即

$$x_0=x(u_0,v_0),\qquad y_0=y(u_0,v_0),\qquad z_0=z(u_0,v_0).$$

剩下的问题只有一个：怎么求曲面在这一点的法向量？

### 法向量的推导：借道隐映射定理

两个前提条件先摆清楚。首先，$x,y,z$ 关于 $u,v$ 有**连续的偏导数**；其次，$x,y,z$ 关于 $u,v$ 的雅可比矩阵

$$\begin{pmatrix} x_u & x_v \\ y_u & y_v \\ z_u & z_v \end{pmatrix}$$

的**秩等于 $2$**。秩是 $2$ 是什么意思？代数里我们知道：这个 $3\times 2$ 矩阵可以取出三个二阶子行列式，秩为 $2$ 就是说这三个子行列式**不同时为零**，也就是

$$\frac{\partial(y,z)}{\partial(u,v)},\qquad \frac{\partial(z,x)}{\partial(u,v)},\qquad \frac{\partial(x,y)}{\partial(u,v)}$$

不全为零。我们不妨假设在 $(u_0,v_0)$ 这一点

$$\frac{\partial(x,y)}{\partial(u,v)} \neq 0 .$$

在这样的条件下，看怎么求法向量。

**第一步，反解出 $u,v$。** 参数方程有三个式子，先看前两个：$x=x(u,v)$，$y=y(u,v)$。既然 $\dfrac{\partial(x,y)}{\partial(u,v)} \neq 0$，回忆一下隐函数那一节讲过的**隐映射定理**：由这两个式子，可以唯一确定逆映射

$$u=u(x,y),\qquad v=v(x,y),$$

它在 $(x_0,y_0)$ 的某一个邻域里成立。

**第二步，曲面局部化成 $z=f(x,y)$。** 有了 $u=u(x,y)$、$v=v(x,y)$，代进第三个式子，得

$$z=z\bigl(u(x,y),\,v(x,y)\bigr)=f(x,y).$$

大家看好，这实际是一个复合形式：$z$ 本来是 $u,v$ 的函数，而 $u,v$ 现在是 $x,y$ 的函数。也就是说，在这点的附近，这张参数曲面就可以用 $z=f(x,y)$ 来表示，法向量就是

$$\vec{n}_0=(z_x,\ z_y,\ -1).$$

这里插一句大家都明白、但要点破的事：一个向量前面乘上一个常数 $k$，$k$ 是正的，方向不变；$k$ 是负的，只不过换一个反方向。我们要找的是切平面方程和法线方程，对法向量只关心**方向**——正着反着都没关系。所以后面我们会把这个法向量乘上一个适当的非零常数，让表达式更加规范。

**第三步，把 $z_x$、$z_y$ 算出来。** 这是整个推导的正戏。

把逆映射记成向量值函数 $g:\ (x,y)\mapsto\bigl(u(x,y),\,v(x,y)\bigr)$。把 $u(x,y)$、$v(x,y)$ 代回 $x=x(u,v)$、$y=y(u,v)$，得到两个恒等式：

$$x\bigl(u(x,y),\,v(x,y)\bigr)=x,\qquad y\bigl(u(x,y),\,v(x,y)\bigr)=y.$$

大家看：左边恰好是向量值函数的复合 $f\circ g$——$f$ 是 $(u,v)\mapsto(x,y)$，$g$ 是 $(x,y)\mapsto(u,v)$，两个复合起来，自变量是 $x,y$，作用的结果还是 $x,y$ 自己。这是个恒等式。

对左右两边求导。按向量值函数复合的求导公式，左边是 $f$ 的导数乘上 $g$ 的导数；右边关于 $x,y$ 求导——$x$ 对 $x$ 求导是 $1$、$x$ 对 $y$ 求导是 $0$、$y$ 对 $x$ 求导是 $0$、$y$ 对 $y$ 求导是 $1$——这是**单位矩阵**。所以

$$\begin{pmatrix} x_u & x_v \\ y_u & y_v \end{pmatrix}\begin{pmatrix} u_x & u_y \\ v_x & v_y \end{pmatrix}=\begin{pmatrix} 1 & 0 \\ 0 & 1 \end{pmatrix}.$$

马上得到

$$\begin{pmatrix} u_x & u_y \\ v_x & v_y \end{pmatrix}=\begin{pmatrix} x_u & x_v \\ y_u & y_v \end{pmatrix}^{-1}=\frac{1}{J}\begin{pmatrix} y_v & -x_v \\ -y_u & x_u \end{pmatrix},\qquad J=\frac{\partial(x,y)}{\partial(u,v)}=x_u y_v-x_v y_u .$$

也就是说 $u_x=\dfrac{y_v}{J}$，$u_y=-\dfrac{x_v}{J}$，$v_x=-\dfrac{y_u}{J}$，$v_y=\dfrac{x_u}{J}$：$u,v$ 关于 $x,y$ 的四个偏导数，都从这个式子来。

**第四步，链式法则收尾。** $z$ 是 $u,v$ 的函数，$u,v$ 又是 $x,y$ 的函数，按复合函数求导公式，

$$z_x=z_u u_x+z_v v_x=\frac{z_u y_v-z_v y_u}{J}=\frac{\partial(z,y)/\partial(u,v)}{\partial(x,y)/\partial(u,v)},$$

$$z_y=z_u u_y+z_v v_y=\frac{-z_u x_v+z_v x_u}{J}=\frac{\partial(x,z)/\partial(u,v)}{\partial(x,y)/\partial(u,v)}.$$

于是

$$\vec{n}_0=\left(\frac{\partial(z,y)/\partial(u,v)}{J},\ \frac{\partial(x,z)/\partial(u,v)}{J},\ -1\right).$$

为了公式统一起见，把 $\dfrac{\partial(z,y)}{\partial(u,v)}$ 换一个次序，写成 $-\dfrac{\partial(y,z)}{\partial(u,v)}$；再利用刚才说的"数乘不改变方向"，把分母 $J$ 一并乘进去（取常数 $k=-J \neq 0$），就得到干净的结果：

$$\vec{n}=\left(\frac{\partial(y,z)}{\partial(u,v)},\ \ \frac{\partial(z,x)}{\partial(u,v)},\ \ \frac{\partial(x,y)}{\partial(u,v)}\right).$$

> **结论**：参数曲面 $x=x(u,v),\,y=y(u,v),\,z=z(u,v)$ 在一点的法向量，就是三个二阶雅可比行列式 $\dfrac{\partial(y,z)}{\partial(u,v)}$、$\dfrac{\partial(z,x)}{\partial(u,v)}$、$\dfrac{\partial(x,y)}{\partial(u,v)}$，都在该点对应的 $(u_0,v_0)$ 处取值。

到这里，三种表示的法向量就凑齐了：

| 曲面的表示 | 法向量 |
| :--- | :--- |
| $F(x,y,z)=0$ | $(F_x,\ F_y,\ F_z)$ |
| $z=f(x,y)$ | $(f_x,\ f_y,\ -1)$ |
| $x=x(u,v),\ y=y(u,v),\ z=z(u,v)$ | $\left(\dfrac{\partial(y,z)}{\partial(u,v)},\ \dfrac{\partial(z,x)}{\partial(u,v)},\ \dfrac{\partial(x,y)}{\partial(u,v)}\right)$ |

### 切平面、法线方程与好记的规律

有了法向量，切平面方程马上写出（三个行列式都在 $(u_0,v_0)$ 取值）：

$$\frac{\partial(y,z)}{\partial(u,v)}(x-x_0)+\frac{\partial(z,x)}{\partial(u,v)}(y-y_0)+\frac{\partial(x,y)}{\partial(u,v)}(z-z_0)=0,$$

法线方程是

$$\dfrac{x-x_0}{\dfrac{\partial(y,z)}{\partial(u,v)}}=\dfrac{y-y_0}{\dfrac{\partial(z,x)}{\partial(u,v)}}=\dfrac{z-z_0}{\dfrac{\partial(x,y)}{\partial(u,v)}}.$$

> **注意**：这一切的前提是雅可比矩阵的秩为 $2$，即三个二阶行列式不同时为零。如果在某一点它们同时为零，那一点处的法向量就求不出来了。

那这个法向量好记还是不好记？我说还是好记的。大家注意：它的三个分量是三个雅可比行列式——三个变量 $x,y,z$，任取两个、关于 $u,v$ 求雅可比行列式。第一项不能从 $x$ 起头：你取了 $x$ 以后，剩下 $y,z$ 两个变量是对称的，到底 $y$ 在前还是 $z$ 在前就定不下来了；所以第一项肯定是 $\dfrac{\partial(y,z)}{\partial(u,v)}$，第二项是 $\dfrac{\partial(z,x)}{\partial(u,v)}$，第三项是 $\dfrac{\partial(x,y)}{\partial(u,v)}$——让 $(y,z)\to(z,x)\to(x,y)$ 轮着来。掌握了这个规律，它就是好记的。

### 隐式曲面求切平面与法线

曲面

$$\mathrm{e}^{z}-z+xy=3$$

求它在点 $(2,1,0)$ 处的切平面与法线方程。

这是第一种表示。令 $F(x,y,z)=\mathrm{e}^{z}-z+xy-3$，则

$$\vec{n}=(F_x,\ F_y,\ F_z)=(y,\ x,\ \mathrm{e}^{z}-1).$$

要在 $(2,1,0)$ 这一点取值：$y=1$，$x=2$，$\mathrm{e}^{0}-1=0$，所以

$$\vec{n}\big|_{(2,1,0)}=(1,\ 2,\ 0).$$

切平面方程：

$$(x-2)+2(y-1)=0.$$

法线方程：第三个分量是 $0$，写成

$$\frac{x-2}{1}=\frac{y-1}{2},\qquad z=0.$$

大家注意：解这类题，关键是一个法向量，对不对？法向量有了，切平面、法线都是照公式写。

### 参数曲面：先认识双曲函数

曲面由参数方程给出：

$$x=a\,\mathrm{ch}\,u\cos v,\qquad y=b\,\mathrm{ch}\,u\sin v,\qquad z=c\,\mathrm{sh}\,u,$$

求它在 $(u,v)=(0,\pi/4)$ 对应点处的切平面与法线方程。

这里出现两个函数：$\mathrm{sh}\,u$ 叫**双曲正弦**，$\mathrm{ch}\,u$ 叫**双曲余弦**，定义为

$$\mathrm{sh}\,u=\frac{\mathrm{e}^{u}-\mathrm{e}^{-u}}{2},\qquad \mathrm{ch}\,u=\frac{\mathrm{e}^{u}+\mathrm{e}^{-u}}{2}.$$

我不知道你们这个函数见过没有。它们的导数公式要记住：

$$(\mathrm{sh}\,u)'=\mathrm{ch}\,u,\qquad (\mathrm{ch}\,u)'=\mathrm{sh}\,u.$$

> **易错点**：$\cos$ 的导数是 $-\sin$，但双曲余弦的导数是 $+\mathrm{sh}$，没有负号。这个公式大家记住。

先把点定下来。$u=0$、$v=\pi/4$ 代进参数方程：

$$x_0=a\cos\frac{\pi}{4}=\frac{\sqrt{2}}{2}a,\qquad y_0=b\sin\frac{\pi}{4}=\frac{\sqrt{2}}{2}b,\qquad z_0=c\,\mathrm{sh}\,0=0.$$

要求法向量，就是那三个雅可比行列式。先把六个偏导数都算出来：

$$x_u=a\,\mathrm{sh}\,u\cos v,\qquad x_v=-a\,\mathrm{ch}\,u\sin v,$$

$$y_u=b\,\mathrm{sh}\,u\sin v,\qquad y_v=b\,\mathrm{ch}\,u\cos v,$$

$$z_u=c\,\mathrm{ch}\,u,\qquad z_v=0.$$

代进行列式，具体展开不细算了，把 $(0,\pi/4)$ 处的值写下来：

$\dfrac{\partial(y,z)}{\partial(u,v)}=y_u z_v-y_v z_u=-bc\,\mathrm{ch}^2u\cos v$，在该点等于 $-\dfrac{\sqrt{2}}{2}bc$；

$\dfrac{\partial(z,x)}{\partial(u,v)}=z_u x_v-z_v x_u=-ac\,\mathrm{ch}^2u\sin v$，在该点等于 $-\dfrac{\sqrt{2}}{2}ac$；

$\dfrac{\partial(x,y)}{\partial(u,v)}=x_u y_v-x_v y_u=ab\,\mathrm{sh}\,u\,\mathrm{ch}\,u(\cos^2v+\sin^2v)=ab\,\mathrm{sh}\,u\,\mathrm{ch}\,u$，在该点等于 $0$。

所以

$$\vec{n}\Big|_{(0,\pi/4)}=\left(-\frac{\sqrt{2}}{2}bc,\ -\frac{\sqrt{2}}{2}ac,\ 0\right).$$

切平面方程：

$$-\frac{\sqrt{2}}{2}bc\left(x-\frac{\sqrt{2}}{2}a\right)-\frac{\sqrt{2}}{2}ac\left(y-\frac{\sqrt{2}}{2}b\right)=0.$$

法线方程：第三个分量是 $0$，所以

$$\dfrac{x-\frac{\sqrt{2}}{2}a}{-\frac{\sqrt{2}}{2}bc}=\dfrac{y-\frac{\sqrt{2}}{2}b}{-\frac{\sqrt{2}}{2}ac},\qquad z=0.$$

利用"数乘不改变方向"，把公共因子 $-\dfrac{\sqrt{2}}{2}c$ 约掉，法线的方向就是 $(b,\,a,\,0)$，法线也可以写成 $\dfrac{x-\frac{\sqrt{2}}{2}a}{b}=\dfrac{y-\frac{\sqrt{2}}{2}b}{a}$，$z=0$。

### 证明一张曲面是柱面

有一张曲面，方程是

$$\mathrm{e}^{2x-z}=f(\pi y-\sqrt{2}\,z),$$

要你证明这张曲面 $\Sigma$ 是一张**柱面**。这里 $f$ 具体是什么不知道，但有一个条件：$f$ 具有连续导数。大家注意看记号：$f$ 括号里没有逗号，所以它是一元函数；这里的 $f'$ 是普通导数，**不是偏导数**，它作用的变量是 $\pi y-\sqrt{2}z$。

什么叫证明它是柱面？柱面是有母线的，而且所有母线的方向相同。所以先想清楚要证什么：需要证存在一个方向 $\vec{a}$，使得对曲面上**任意**一点 $P_0$，从这点出发沿 $\vec{a}$ 方向、随着 $t$ 变化的这条直线

$$P_0+t\,\vec{a}\qquad(t\in\mathbb{R})$$

整条都落在 $\Sigma$ 上。如果能证这样一件事，这张曲面就是柱面——上面任意一点拿来，从这个点开始沿这个方向画一条直线，都在曲面上。

那这个 $\vec{a}$ 应该有什么性质？大家想想：$\vec{a}$ 应该跟曲面的法向量**垂直**，对不对？我们来看法向量。把曲面写成 $F(x,y,z)=\mathrm{e}^{2x-z}-f(\pi y-\sqrt{2}z)=0$，则

$$\vec{n}=(F_x,\ F_y,\ F_z)=\bigl(2\mathrm{e}^{2x-z},\ -\pi f'(\pi y-\sqrt{2}z),\ -\mathrm{e}^{2x-z}+\sqrt{2}\,f'(\pi y-\sqrt{2}z)\bigr).$$

要找 $\vec{a}=(a_1,a_2,a_3)$，使得 $\vec{a}\cdot\vec{n}=0$ 在曲面上每一点都成立。把点积展开：

$$\vec{a}\cdot\vec{n}=(2a_1-a_3)\,\mathrm{e}^{2x-z}+(\sqrt{2}\,a_3-\pi a_2)\,f'(\pi y-\sqrt{2}z).$$

这里要看清楚：$\mathrm{e}^{2x-z}$ 和 $f'(\cdot)$ 是两块互相独立的东西——$f$ 具体不知道，$f'$ 的值跟着点变——所以要让点积恒为零，$f'$ 必须经过乘积运算被消掉，$\mathrm{e}^{2x-z}$ 的系数也得消掉。看系数：$F_y$ 里是 $-\pi$，$F_z$ 里是 $+\sqrt{2}$，第二个分量配上合适的倍数，这两个就没了；第一、第三个分量配成 $2a_1=a_3$，$\mathrm{e}^{2x-z}$ 这项也消掉。干脆取 $a_1=\pi$，那么 $a_3=2\pi$、$a_2=2\sqrt{2}$，即

$$\vec{a}=(\pi,\ 2\sqrt{2},\ 2\pi).$$

代回去验一下：$2\pi\,\mathrm{e}^{2x-z}-2\sqrt{2}\pi f'-2\pi\,\mathrm{e}^{2x-z}+2\sqrt{2}\pi f'=0$，两个系数全消掉了——这个向量跟曲面上所有的法向量都正交。

方向有了，现在验证整条直线在曲面上。设 $(x_0,y_0,z_0)\in\Sigma$，意思是

$$\mathrm{e}^{2x_0-z_0}=f(\pi y_0-\sqrt{2}z_0).$$

考虑直线

$$x=x_0+\pi t,\qquad y=y_0+2\sqrt{2}\,t,\qquad z=z_0+2\pi t.$$

代进方程两边看：左边 $2x-z=2x_0+2\pi t-z_0-2\pi t=2x_0-z_0$，两项消掉了；右边 $\pi y-\sqrt{2}z=\pi y_0+2\sqrt{2}\pi t-\sqrt{2}z_0-2\sqrt{2}\pi t=\pi y_0-\sqrt{2}z_0$，大家有没有看到，这两项也消掉了。所以沿整条直线，方程两边始终保持出发点的值，而出发点满足方程，于是这条直线上的点 $(x,y,z)$ 也都满足方程，全在曲面上。

所以 $\Sigma$ 是一张柱面，母线方向 $\vec{a}=(\pi,\,2\sqrt{2},\,2\pi)$。

### 曲线的夹角、曲面的夹角与正交

这里再引进一个概念。我们说两条曲线相交，有没有夹角？比如有一条曲线 $\gamma_1$、一条曲线 $\gamma_2$，它们相交。夹角是怎么定义的？我们在交点这一点做两条曲线的**切线**，这两条切线的夹角 $\alpha$，就称为 $\gamma_1$ 与 $\gamma_2$ 的**夹角**。

现在有两张曲面 $\Sigma_1$、$\Sigma_2$，它们有一条交线。在交线上取一点，在这一 点做两张曲面各自的法向量 $\vec{n}_1$、$\vec{n}_2$，这两个法向量有个夹角，比如说是 $\beta$，我们就说：这两张曲面在交线上这一点处相交的角是 $\beta$。

那么两张曲面从整体来讲，它的夹角可以讲吗？不能讲。为什么？因为两张曲面有一条交线，这条交线上有无穷多个点，而在每一个点上，两个法向量的夹角是不一样的。所以只有"交线上某一点处的夹角"，没有整体的夹角。

于是有下面的概念。

> **定义**：两张曲面 $\Sigma_1$、$\Sigma_2$，如果在**交线的每一点**上，它们相交的角都是 $\dfrac{\pi}{2}$——也就是每一点处两个法向量都正交——就称 $\Sigma_1$ 与 $\Sigma_2$ **正交**。

注意，是交线的每一点上两个法向量都正交，一个点都不能漏。

### 两张球面正交的证明

两张球面：

$$x^2+y^2+z^2=2ax,\qquad x^2+y^2+z^2=2by,$$

证明它们正交。

要证明正交，意思就是：在交线的每一点，这两张曲面的两个法向量的夹角都是 $\dfrac{\pi}{2}$。

设 $(x,y,z)$ 是两个球面交线上的任意一点。既然这个点在交线上，那它既在这张球面上、也在那张球面上——换个说法，把 $x,y,z$ 代到这两个方程里都成立等式。两张曲面都用第一种表示，令

$$F_1=x^2+y^2+z^2-2ax,\qquad F_2=x^2+y^2+z^2-2by,$$

法向量分别是

$$\vec{n}_1=(2x-2a,\ 2y,\ 2z),\qquad \vec{n}_2=(2x,\ 2y-2b,\ 2z).$$

现在要问：$\vec{n}_1\cdot\vec{n}_2$ 是不是等于零？算一下：

$$\vec{n}_1\cdot\vec{n}_2=4x^2-4ax+4y^2-4by+4z^2.$$

拆开来看，把 $4$ 拆成 $2+2$：

$$\vec{n}_1\cdot\vec{n}_2=\underbrace{\bigl(2x^2+2y^2+2z^2-4ax\bigr)}_{2\,(x^2+y^2+z^2-2ax)}+\underbrace{\bigl(2x^2+2y^2+2z^2-4by\bigr)}_{2\,(x^2+y^2+z^2-2by)}.$$

$(x,y,z)$ 在交线上，它一直在两张曲面上，直接满足这两个方程，所以第一个括号等于零、第二个括号等于零，点积为零。而 $(x,y,z)$ 是交线上的**任意**一点，所以这两张曲面正交。

这个正交的概念大家记住：是在交线的每一点上两张曲面都正交，每一点都要正交。

到这儿，第五节——偏导数在几何上的应用——就讲完了。

> **承前启后**：上一节讲完「曲面的参数表示：法向量、切平面与正交曲面」，下一节接着讲「多元函数的无条件极值：定义、必要条件与判别法的铺垫」。

---

## 多元函数的无条件极值：定义、必要条件与判别法的铺垫
> 对应块：BLK145 | 覆盖分集：P152 | 块标题（分集名）：《154.第12章 第6节 无条件极值（1）》

### 从一元到多元：只跟一个变量有关，其实是理想化的

一元函数 $y = f(x)$ 的极值、最值问题，应用面广得惊人。生产实践、社会活动，随便哪个方面都要碰它：追求原料最省、效率最高，或者追求生产的价值、得到的利益最大化——所有这些，都是极值问题和最值问题。

但以前我们手里只有一元函数的极值理论，也就是说，那个我们想追求最大效果、最大利益的应变量，只跟**一个**元素有关。仔细想想，"只跟一个元素有关"这种情况，在自然界里、在社会活动中，其实是理想化的——任何一件事物都同时跟许多变量发生关系。既然多元函数已经学了，就要把极值问题（或者叫最值问题）提到多元函数上来：

$$y = F(x), \qquad x = (x_1, x_2, \ldots, x_n) \in \mathbb{R}^n.$$

$x$ 是 $\mathbb{R}^n$ 空间里的一个点（向量），换一种写法就是 $f(x_1, x_2, \ldots, x_n)$。要讨论的就是它的极值或最值。

这一节叫**无条件极值**——不附加任何条件的极值问题。下一节讲的是附加了条件以后的条件极值，那是另外一个问题，这里先把无条件的讲清楚。

### 极值点与极值的定义

设 $D \subset \mathbb{R}^n$ 是一个区域，$x_0 = (x_1^0, x_2^0, \ldots, x_n^0) \in D$。

> **定义**：若存在 $x_0$ 的一个邻域 $U(x_0)$，使得对一切 $x \in U(x_0)$ 都有 $f(x) \le f(x_0)$，则称 $x_0$ 是 $f$ 的一个**极大值点**，$f(x_0)$ 称为 $f$ 的一个**极大值**；若把不等式换成 $f(x) \ge f(x_0)$，则 $x_0$ 是**极小值点**，$f(x_0)$ 是**极小值**。

直观地说：在 $x_0$ 的某个邻域里，函数在别处的值都不超过 $x_0$ 处的值，$x_0$ 就是一个"山顶"；反过来处处不低于它，就是"谷底"。极大值点与极小值点统称**极值点**，极大值与极小值统称**极值**——点与值分开记，别混成一句。

### 极值的必要条件：所有偏导数都等于零

一元函数里有一条熟悉的结论：函数在一个区间上可导，若某点是极值点，那一点的导数就等于零。多元函数有完全对应的结果。

> **结论（极值的必要条件）**：设 $x_0$ 是 $f$ 的一个极值点，且 $f$ 在 $x_0$ 处可偏导，则 $f$ 的所有一阶偏导数在 $x_0$ 处都等于零。

这里"可偏导"要说清楚：指对每一个变量——$x_1, x_2, \ldots, x_n$ 每一个——都能求偏导。结论写成式子就是

$$\frac{\partial f}{\partial x_1}(x_0) = \frac{\partial f}{\partial x_2}(x_0) = \cdots = \frac{\partial f}{\partial x_n}(x_0) = 0.$$

这个证明很容易，思路只有一句话：**把多元问题按回一元**。把 $x_2$ 固定在 $x_2^0$，$x_3$ 固定在 $x_3^0$，一直到 $x_n$ 固定在 $x_n^0$，只让 $x_1$ 自由变化——其余变量都焊死了，$f$ 就成了一个一元函数

$$g(x_1) = f(x_1, x_2^0, \ldots, x_n^0).$$

假如 $x_0$ 是 $f$ 的极值点，比方说在邻域内 $f(x) \le f(x_0)$（取最大值的情形），那么对这个一元函数来看，$x_1^0$ 也取到了最大值：在 $x_1^0$ 附近 $g(x_1) \le g(x_1^0)$。$f$ 可偏导，$g$ 在 $x_1^0$ 就可导，由一元函数的结论马上推出

$$g'(x_1^0) = \frac{\partial f}{\partial x_1}(x_1^0, x_2^0, \ldots, x_n^0) = 0.$$

$x_1$ 是任意的：把它换成 $x_2$，让 $x_2$ 变化、其余变量固定在各自的 $x_i^0$，同样得到 $\partial f/\partial x_2(x_0) = 0$；一个个换下去，$n$ 个偏导数全为零。

### 必要条件不是充分条件：xy 的反例

这条定理只是**必要**条件，不是充分的——偏导数全为零，不保证是极值点。为什么？举例最快。

取 $f(x, y) = xy$。算偏导：$f_x = y$，$f_y = x$，在原点 $(0, 0)$ 处 $f_x = f_y = 0$，必要条件满足了。但 $(0, 0)$ 对这个函数来说既不是极大值点，也不是极小值点。看原点附近函数值的分布：

```text
                y
                ^
       (-)      |      (+)
     xy < 0     |     xy > 0
    ------------+-------------> x
     xy > 0     |     xy < 0
       (+)      |      (-)
```

$f(0, 0) = 0$，而原点的任何一个小邻域里，第一、三象限 $xy > 0$，第二、四象限 $xy < 0$——函数值一会儿比 $f(0, 0)$ 大、一会儿比它小，极值无从谈起。

> **易错点**：偏导数全为零的点不一定是极值点；反过来，极值点处若可偏导，偏导数必定全为零。两个方向的蕴含关系别弄反。

### 不可偏导处也能取极值：z = |x|

定理条件里加了"可偏导"，这一条不能忘：**一个函数可以在不可偏导的点取得极大值或极小值**。一元函数的时候我们就知道这件事，多元函数同理。

例子：$f(x, y) = |x|$，跟 $y$ 完全没关系。一元函数 $|x|$ 在 $0$ 点不可导，所以作为二元函数，在 $y$ 轴上的每一点 $(0, y)$（$y$ 任意），它关于 $x$ 不可偏导；关于 $y$ 的偏导倒是存在的。

图像长什么样？函数里不含 $y$，整个图形沿 $y$ 方向平移不变，所以它是一张**柱面**：母线平行于 $y$ 轴，截面就是 $xz$ 平面上的曲线 $z = |x|$。

```text
      z
      ^
      |   \         /
      |    \       /        截面曲线 z = |x|
      |     \     /
      |      \   /
      |       \ /
      |        V
      +---------------------> x

  整条 V 形折线沿 y 方向平移不变（函数与 y 无关），
  扫出一张母线平行于 y 轴的柱面
```

关键结论：$y$ 轴上的任意一点，都是这个函数的**极小值点**。不可偏导，照样取极值。所以找极值点的时候，除了驻点，偏导不存在的点也一个都不能漏。

### 求极值的第一步：驻点与可疑点

有了必要条件，求极值的操作流程就定下来了。设 $y = f(x_1, x_2, \ldots, x_n)$，目标是把极值点找出来，再把点代回去算出极值——是极小值点就把极小值算出来，是极大值点就把极大值算出来。

第一步，让每个偏导数都等于零，解方程组：

$$
\begin{cases}
f_{x_1}(x_1, \ldots, x_n) = 0,\\
f_{x_2}(x_1, \ldots, x_n) = 0,\\
\quad\ \vdots\\
f_{x_n}(x_1, \ldots, x_n) = 0.
\end{cases}
$$

$n$ 个变量就求 $n$ 个偏导，得到 $n$ 个方程，把这个方程组解出来。解可能有好些好些组，记作 $(x_1^k, x_2^k, \ldots, x_n^k)$。这些偏导数全为零的点称为**驻点**——一元函数时导数等于零的点就叫驻点，现在是它的多元版本。

第二步，把**偏导不存在的点**也收进来：只要某一点关于某一个变量的偏导不存在，这个点也要拿来。驻点加上偏导不存在的点，合在一起称为极值的**可疑点**——可能是极值的点。

```text
   偏导数全为零的点(驻点)  +  偏导不存在的点
                  |
                  v
              极值的可疑点
                  |
                  v
      逐一判断: 哪些是极大值点, 哪些是极小值点,
                哪些不是极值点
```

> **易错点**：解出驻点不等于求出极值。驻点只是"可疑"，判断是下一步的事；列名单时别漏掉偏导不存在的点，漏一个就可能漏掉真正的极值。

### 回忆一元函数：二阶判别法是怎么来的

可疑点抓出来了，怎么判断？先回忆一元函数的做法。设 $f$ 有二阶连续导数，$f'(x_0) = 0$——$x_0$ 已经是极值的可疑点了，那么：

- $f''(x_0) > 0$：$x_0$ 是极小值点；
- $f''(x_0) < 0$：$x_0$ 是极大值点；
- $f''(x_0) = 0$：结论不一定——可能是极小值点，可能是极大值点，也可能不是极值点。

第三种情形三种可能都存在，例子都是老朋友：$x^4$ 在原点取极小值，$-x^4$ 在原点取极大值，$x^3$ 在原点不是极值点——三个函数在原点的二阶导数全为零。这三条你应当还记得，记不清的话正好现在补上。

这套结论当年是怎么得到的？靠**泰勒公式**：

$$f(x_0 + \Delta x) = f(x_0) + f'(x_0)\,\Delta x + \frac{1}{2!}\,f''(x_0 + \theta\,\Delta x)\,(\Delta x)^2, \qquad 0 < \theta < 1.$$

$x_0$ 是驻点，一阶项 $f'(x_0)\,\Delta x$ 没了；$(\Delta x)^2$ 恒正，差的符号就全看 $f''(x_0 + \theta\,\Delta x)$。$f''$ 连续：$f''(x_0) > 0$，则在 $x_0$ 附近它也大于零，右边整体为正，$f(x_0 + \Delta x) > f(x_0)$，极小值；$f''(x_0) < 0$，同理得极大值；$f''(x_0) = 0$ 时二阶项定不了号，判别失效。

记住这条推导路径——二元函数的判别法，就照着它一步一步走。

### 二元泰勒公式：把判别问题化成二次型

设 $z = f(x, y)$，$(x_0, y_0)$ 是驻点——关于 $x$、关于 $y$ 的偏导都为零，这一点已经是极值的可能点了；再加一个条件：$f$ 有二阶连续偏导。目标：建立一个判别方法，看完就能判断这一点是极大值点还是极小值点。

照一元的路子，先写二元泰勒公式：

$$
\begin{aligned}
f(x_0+\Delta x,\, y_0+\Delta y) &= f(x_0, y_0) + f_x(x_0, y_0)\,\Delta x + f_y(x_0, y_0)\,\Delta y \\
&\quad + \frac{1}{2!}\Big[f_{xx}(P)\,(\Delta x)^2 + 2f_{xy}(P)\,\Delta x\,\Delta y + f_{yy}(P)\,(\Delta y)^2\Big],
\end{aligned}
$$

其中 $P = (x_0 + \theta\,\Delta x,\ y_0 + \theta\,\Delta y)$，$0 < \theta < 1$。$(x_0, y_0)$ 是驻点，两个一阶偏导全为零，一阶项整体消失，把 $f(x_0, y_0)$ 移到左边：

$$f(x_0+\Delta x,\, y_0+\Delta y) - f(x_0, y_0) = \frac{1}{2}\Big[f_{xx}(P)\,(\Delta x)^2 + 2f_{xy}(P)\,\Delta x\,\Delta y + f_{yy}(P)\,(\Delta y)^2\Big].$$

把 $(x_0 + \Delta x,\, y_0 + \Delta y)$ 看成 $(x_0, y_0)$ 附近任取的一点 $(x, y)$，差的符号全由右边这个二次齐次式决定：它大于零就是极小值，小于零就是极大值。

下一步，把 $P$ 点的二阶偏导拉回 $(x_0, y_0)$ 来看。$P$ 随 $(\Delta x, \Delta y)$ 变化，当 $\rho \to 0$ 时 $P \to (x_0, y_0)$；二阶偏导连续，所以

$$f_{xx}(x, y) = f_{xx}(x_0, y_0) + \alpha, \qquad f_{xy}(x, y) = f_{xy}(x_0, y_0) + \beta, \qquad f_{yy}(x, y) = f_{yy}(x_0, y_0) + \gamma,$$

其中 $\alpha, \beta, \gamma$ 都是 $\rho \to 0$ 时的无穷小量，而

$$\rho = \sqrt{(\Delta x)^2 + (\Delta y)^2}$$

正是点 $(x_0+\Delta x,\, y_0+\Delta y)$ 到 $(x_0, y_0)$ 的距离。代回去，差分裂成"主项 + 无穷小"两块：

$$
\begin{aligned}
f(x_0+\Delta x,\, y_0+\Delta y) - f(x_0,\, y_0) &= \frac{1}{2}\Big[f_{xx}(x_0, y_0)\,(\Delta x)^2 + 2f_{xy}(x_0, y_0)\,\Delta x\,\Delta y + f_{yy}(x_0, y_0)\,(\Delta y)^2\Big] \\
&\quad + \frac{1}{2}\Big[\alpha\,(\Delta x)^2 + 2\beta\,\Delta x\,\Delta y + \gamma\,(\Delta y)^2\Big].
\end{aligned}
$$

现在把 $\rho^2$ 提出来。令

$$\xi = \frac{\Delta x}{\rho}, \qquad \eta = \frac{\Delta y}{\rho},$$

马上有一个关键性质：

$$\xi^2 + \eta^2 = \frac{(\Delta x)^2 + (\Delta y)^2}{\rho^2} = 1.$$

也就是说，$(\xi, \eta)$ 永远落在单位圆上，不可能两个同时为零。再记

$$A = f_{xx}(x_0, y_0), \qquad B = f_{xy}(x_0, y_0), \qquad C = f_{yy}(x_0, y_0),$$

差的表达式就整理成了干净的形状：

$$f(x_0+\Delta x,\, y_0+\Delta y) - f(x_0, y_0) = \frac{\rho^2}{2}\Big(A\xi^2 + 2B\xi\eta + C\eta^2\Big) + \frac{\rho^2}{2}\Big(\alpha\xi^2 + 2\beta\xi\eta + \gamma\eta^2\Big).$$

后一个括号是无穷小量：由 $\xi^2 + \eta^2 = 1$ 和 $|\xi\eta| \le \tfrac{1}{2}$，它的绝对值不超过 $|\alpha| + |\beta| + |\gamma|$，当 $\rho \to 0$ 时趋于零。

于是整个问题归结为讨论二次型

$$Q(\xi, \eta) = A\xi^2 + 2B\xi\eta + C\eta^2$$

的**正定、负定、不定**：

| 二次型的性态 | 在单位圆上的表现 | $\rho$ 充分小时差的符号 | 结论 |
| :---: | :---: | :---: | :---: |
| 正定 | 除原点外恒正，有正的最小值 | 恒正 | 极小值点 |
| 负定 | 除原点外恒负，有负的最大值 | 恒负 | 极大值点 |
| 不定 | 可正可负 | 可正可负 | 不是极值点 |

把理由说透。若 $Q$ 正定——意思是除原点外 $Q$ 处处为正——而单位圆上根本没有原点，所以圆上每一点都有 $Q > 0$；连续函数在有界闭集上必取到最小值，且最小值在某点取到、故严格大于零。于是第一项是 $\rho^2/2$ 乘一个正常数，第二项是 $\rho^2$ 乘无穷小量——$\rho$ 充分小时，正常数压倒无穷小，差恒正，得极小值。若 $Q$ 负定，同理，单位圆上有负的最大值压倒无穷小，差恒负，得极大值。若 $Q$ 不定——什么叫不定？可正可负——那么差一会儿大于零、一会儿小于零，$(x_0, y_0)$ 既不是极大值点，也不是极小值点。

还剩最后一个问题：二次型的正定、负定到底怎么判定？这个内容我们还没学过，下一讲专门来解决它。本讲到这里收尾，最后照例布置了作业，具体题目没有给出，这里如实注明，不代拟。

> **承前启后**：上一节讲完「多元函数的无条件极值：定义、必要条件与判别法的铺垫」，下一节接着讲「无条件极值的判定：从驻点到二次型」。

---

## 无条件极值的判定：从驻点到二次型
> 对应块：BLK146 | 覆盖分集：P153 | 块标题（分集名）：《155.第12章 第6节 无条件极值（2）》

上节课我们开始讲无条件极值，也就是多元函数的极值问题。设 $y=f(x)$，这里的 $x$ 是向量，具体写出来就是 $x=(x_1,x_2,\cdots,x_n)$，所以 $f$ 是一个 $n$ 元函数。上节课证过这样一个结论：假如 $X_0$，也就是 $(x_1^0,x_2^0,\cdots,x_n^0)$，是 $f$ 的一个极值点，并且 $f$ 在 $X_0$ 这一点关于各个变量的偏导数都存在，那么这些偏导数在 $X_0$ 处必须全部为零。

这样的点有一个专门的名字。

> **定义**：若 $f$ 在 $X_0$ 处满足 $f_{x_1}(X_0)=f_{x_2}(X_0)=\cdots=f_{x_n}(X_0)=0$，则称 $X_0$ 为 $f$ 的一个**驻点**。

知道了驻点，问题其实只解决了一半：接下来要判断这个驻点究竟是极大值点、极小值点，还是根本不是极值点。这一讲就把这个判定问题完整解决——先对二元函数把判别法从头建立起来，最后推广到一般的 $n$ 元函数。

### 驻点只是候选：判定问题从哪里来

为讨论方便，考虑二元函数 $z=f(x,y)$，设 $(x_0,y_0)$ 是驻点，换句话说 $f_x$、$f_y$ 在这一点都是零；再假设 $f$ 在 $(x_0,y_0)$ 的某个邻域内有连续的二阶偏导数——这是本讲从头用到尾的假设。

令 $x=x_0+\Delta x$，$y=y_0+\Delta y$。现在要考察的是差值

$$f(x,y)-f(x_0,y_0).$$

它的符号就是全部问题所在：如果 $(x,y)$ 在 $(x_0,y_0)$ 的小邻域内这个差值大于零，那 $(x_0,y_0)$ 就是极小值点；小于零，就是极大值点；可正可负，就不是极值点。所以判定极值，归结为判断这个差值的符号。

### 泰勒展开：增量的主部是一个二次型

把这个差值作泰勒展开，上节课已经得到了结果：

$$f(x,y)-f(x_0,y_0)=\frac{1}{2}\rho^{2}\left(A\xi^{2}+2B\xi\eta+C\eta^{2}\right)+o(\rho^{2}),$$

其中各个记号是：

- $\rho=\sqrt{\Delta x^{2}+\Delta y^{2}}$，即自变量增量的模；
- $\xi=\dfrac{\Delta x}{\rho}$，$\eta=\dfrac{\Delta y}{\rho}$，显然 $\xi^{2}+\eta^{2}=1$，也就是说 $(\xi,\eta)$ 落在单位圆上；
- $A=f_{xx}(x_0,y_0)$，$B=f_{xy}(x_0,y_0)$，$C=f_{yy}(x_0,y_0)$，都是二阶偏导数在驻点处的值。

> **提示**：这里的 $\rho$ 是增量的模，与邻域记号"$\rho$ 邻域"里的那个 $\rho$ 不是一回事。后面叙述定理时为了避让，邻域半径会改用别的字母来记。

$o(\rho^{2})$ 是 $\rho\to 0$ 时的无穷小量。把它折进括号里，展开式可以写成

$$f(x,y)-f(x_0,y_0)=\frac{1}{2}\rho^{2}\bigl[g(\xi,\eta)+o(1)\bigr],$$

其中

$$g(\xi,\eta)=A\xi^{2}+2B\xi\eta+C\eta^{2}.$$

这个 $g$ 是 $\xi,\eta$ 的二次齐次多项式，代数里称为**二次型**。它还可以写成矩阵形式：

$$g(\xi,\eta)=(\xi,\eta)\begin{pmatrix}A & B\\ B & C\end{pmatrix}\begin{pmatrix}\xi\\ \eta\end{pmatrix},$$

乘开验算一下，正好就是 $A\xi^{2}+2B\xi\eta+C\eta^{2}$。

于是判定极值的问题就变成了：这个二次型 $g$ 在什么条件下恒正、什么条件下恒负、什么条件下变号。

### 核心估计：单位圆上的下界决定一切

注意 $(\xi,\eta)$ 不是自由变量——按定义它被约束在单位圆 $\xi^{2}+\eta^{2}=1$ 上。判定的核心是下面三条估计。

第一条：假如 $g(\xi,\eta)$ 在单位圆上有**正的下界**，即存在 $M>0$，使得在 $\xi^{2}+\eta^{2}=1$ 上恒有 $g(\xi,\eta)\ge M$。那么当 $\rho$ 充分小——换句话说，$(x,y)$ 落在 $(x_0,y_0)$ 的一个小邻域内——就有

$$f(x,y)-f(x_0,y_0)=\frac{1}{2}\rho^{2}\bigl[g(\xi,\eta)+o(1)\bigr]\ge \frac{1}{2}\rho^{2}\bigl[M+o(1)\bigr]>0.$$

道理很简单：$o(1)$ 是无穷小量，只要 $\rho$ 充分小，它的绝对值就比 $M$ 还小，于是 $M+o(1)>0$。差值恒正，所以 $(x_0,y_0)$ 是极小值点。

第二条：反过来，假如 $g$ 在单位圆上有**负的上界** $-M$（$M>0$），同样地

$$f(x,y)-f(x_0,y_0)\le \frac{1}{2}\rho^{2}\bigl[-M+o(1)\bigr]<0,$$

差值恒负，$(x_0,y_0)$ 是极大值点。

第三条：假如 $g$ 在单位圆上**可正可负**，那么在 $g$ 取正值的那些方向上差值最终为正，在 $g$ 取负值的那些方向上差值最终为负——无穷小项改变不了这两个方向的符号——所以 $f(x,y)-f(x_0,y_0)$ 可正可负，$(x_0,y_0)$ 不是极值点。

把整个判定逻辑串起来，就是：

```text
驻点 (x0, y0)：f_x = f_y = 0
   |
   |  二阶泰勒展开
   v
f(x,y) - f(x0,y0) = (1/2)*rho^2 * g(xi,eta) + o(rho^2)
   |               其中 rho = sqrt(dx^2 + dy^2)，xi^2 + eta^2 = 1
   |
   |  考察 g 在单位圆 xi^2 + eta^2 = 1 上的取值
   v
有正的下界 M > 0   --->  f - f0 > 0   --->  (x0,y0) 是极小值点
有负的上界 -M < 0  --->  f - f0 < 0   --->  (x0,y0) 是极大值点
可正可负（变号）    --->  f - f0 变号  --->  (x0,y0) 不是极值点
```

读图：左侧是条件，中间是由展开式得出的差值符号，右侧是极值结论；全部判定只依赖于 $g$ 在单位圆上的符号行为。

这里要求的是"单位圆上有正的下界"，并不是"$g$ 处处为正"。但只要 $g$ 在整个平面上除原点外恒正，它在单位圆这个有界闭集上连续，最小值自动取得到而且是正的——所以"$g$ 恒正"是一个更强、也更好用的条件。这就引出二次型的正定性。

### 二次型的三副面孔：正定、负定、不定

二次型是高等代数里的内容，你们现在还没学到，这里先把需要的几条定义解释清楚。现在把单位圆的约束撤掉，让 $(\xi,\eta)$ 取遍整个平面。

> **定义**：对二次型 $g(\xi,\eta)=A\xi^{2}+2B\xi\eta+C\eta^{2}$，
> - 若对一切 $(\xi,\eta)
e(0,0)$ 都有 $g(\xi,\eta)>0$，称 $g$ **正定**；
> - 若对一切 $(\xi,\eta)
e(0,0)$ 都有 $g(\xi,\eta)<0$，称 $g$ **负定**；
> - 若 $g$ 在某些点上取正值、在另一些点上取负值，称 $g$ **不定**。

$(\xi,\eta)=(0,0)$ 时 $g=0$，这是显然的，所以定义只对非零点提要求。

这三种情况正好对应极值问题的三种结论：二次型正定，驻点是极小值点；负定，驻点是极大值点；不定，驻点不是极值点。前面已经说过，正定型在单位圆上必有正的下界，负定型必有负的上界，所以从"定号"走到"极值"这一步是通的。

### 正定的代数条件：判别式说了算

现在问：$g$ 在什么条件下正定？这用中学里学过的判别式就能回答。

先设 $\eta
e 0$，把 $\eta^{2}$ 提出来：

$$g=A\xi^{2}+2B\xi\eta+C\eta^{2}=\eta^{2}\left[A\left(\frac{\xi}{\eta}\right)^{2}+2B\left(\frac{\xi}{\eta}\right)+C\right].$$

把 $t=\dfrac{\xi}{\eta}$ 看成变量，方括号是关于 $t$ 的二次三项式 $At^{2}+2Bt+C$；而 $\eta^{2}>0$，所以 $g>0$ 当且仅当这个二次三项式对一切 $t$ 恒正。二次三项式恒正，当且仅当判别式小于零、首项系数为正。判别式是 $(2B)^{2}-4AC=4(B^{2}-AC)$，它的相反数正是 $AC-B^{2}$。于是得到条件：

$$AC-B^{2}>0,\qquad A>0.$$

再设 $\xi
e 0$，把 $\xi^{2}$ 提出来：

$$g=\xi^{2}\left[C\left(\frac{\eta}{\xi}\right)^{2}+2B\left(\frac{\eta}{\xi}\right)+A\right],$$

这是关于 $\eta/\xi$ 的二次三项式，判别式相同，条件同样归结为 $AC-B^{2}>0$，再配上 $C>0$。（至于 $\eta=0$ 或 $\xi=0$ 的情形，$g=A\xi^{2}$ 或 $g=C\eta^{2}$，符号由 $A$ 或 $C$ 单独决定，与上面两组合在一起是相容的。）

这两组条件其实是一回事：$AC-B^{2}>0$ 意味着 $AC>B^{2}\ge 0$，所以 $A$、$C$ 必定同号——$A>0$ 时 $C$ 必为正，$C>0$ 时 $A$ 必为正。

> **结论**：$g$ 正定 $\iff AC-B^{2}>0$ 且 $A>0$（等价地 $C>0$）；同理，$g$ 负定 $\iff AC-B^{2}>0$ 且 $A<0$。

### 二元函数的极值判别定理

把前面的准备合起来，就是书上关于二元函数无条件极值的充分条件（定理 12.6.2）。

设 $(x_0,y_0)$ 是 $f$ 的驻点，$f$ 在 $(x_0,y_0)$ 的某个邻域内有二阶连续偏导数（邻域半径记 $\alpha$，因为 $\rho$ 已经被用来记增量的模了）。记

$$A=f_{xx}(x_0,y_0),\qquad B=f_{xy}(x_0,y_0),\qquad C=f_{yy}(x_0,y_0),\qquad H=\det\begin{pmatrix}A & B\\ B & C\end{pmatrix}=AC-B^{2}.$$

| 条件 | 结论 |
| :--- | :--- |
| $H>0$ 且 $A>0$ | $(x_0,y_0)$ 是极小值点 |
| $H>0$ 且 $A<0$ | $(x_0,y_0)$ 是极大值点 |
| $H<0$ | $(x_0,y_0)$ 不是极值点 |
| $H=0$ | 判别法失效，需另行判断 |

$H>0$、$A>0$ 时为什么是极小？此时上一节的条件成立，二次型正定，于是在单位圆上有正的下界，核心估计立即给出 $f(x,y)-f(x_0,y_0)>0$。$H>0$、$A<0$ 时二次型负定，同理得极大。

$H<0$ 相当于判别式 $4(B^{2}-AC)>0$：二次三项式有两个不同的实根，可正可负，二次型不定，所以驻点不是极值点。

最后是 $H=0$。这时判别式等于零，二次三项式是一个完全平方——但要注意，完全平方在非零点上是可能等于零的：它要么恒 $\ge 0$、要么恒 $\le 0$，总存在非零方向使它取到零。在这些方向上，

$$f(x,y)-f(x_0,y_0)=\frac{1}{2}\rho^{2}\cdot 0+o(\rho^{2}),$$

符号完全由 $o(\rho^{2})$ 这一项决定，而它的符号我们无从知晓。所以 $H=0$ 时不能下结论：$(x_0,y_0)$ 可以是极值点，也可以不是极值点，必须对具体的函数另行判断。

### H=0 时判别法确实会失效

举两个例子，看看 $H=0$ 时结局可以有多不同。取两个不依赖 $y$ 的二元函数：

$$f(x,y)=x^{4}\qquad\text{与}\qquad f(x,y)=x^{3}.$$

对这两个函数，$(0,0)$ 都是驻点；而且二阶偏导数在 $(0,0)$ 处全是零，$A=B=C=0$，于是 $H=0$。但结局完全不同：

- 对 $f=x^{4}$，$(0,0)$ 是极小值点；
- 对 $f=x^{3}$，$(0,0)$ 不是极值点（$x<0$ 时函数值为负，$x>0$ 时为正）。

同样的 $H=0$，一个极小、一个非极值。所以 $H=0$ 时这个判别法给不出任何信息，只能回到函数本身去分析——下面的第二个例题就是这种"另行判断"的实战。

### 例题：四个驻点的完整判定

例题 12.6.1：设 $f(x,y)=xy(a-x-y)$，其中 $a
e 0$，求这个函数的极值。

第一步，找驻点。注意这个函数处处存在偏导数，没有偏导数不存在的点，所以要找极值的候选点，只需要找偏导数等于零的点：

$$f_x=y(a-x-y)-xy=y(a-2x-y)=0,\qquad f_y=x(a-x-y)-xy=x(a-x-2y)=0.$$

这是两个变量、二次的方程组，一般来讲最多可以有四组解。分情形解：$y=0$ 配 $x=0$ 得 $(0,0)$；$y=0$ 配 $a-x-2y=0$ 得 $(a,0)$；$a-2x-y=0$ 配 $x=0$ 得 $(0,a)$；两个括号同时为零，相减得 $x=y$，再代回得 $a-3x=0$，即 $\left(\dfrac{a}{3},\dfrac{a}{3}\right)$。恰好四组解：

$$(0,0),\qquad (a,0),\qquad (0,a),\qquad \left(\frac{a}{3},\frac{a}{3}\right).$$

第二步，算二阶偏导数：

$$f_{xx}=-2y,\qquad f_{xy}=a-2x-2y,\qquad f_{yy}=-2x.$$

四个驻点逐点代入，列成一张表：

| 驻点 | $A=f_{xx}$ | $B=f_{xy}$ | $C=f_{yy}$ | $H=AC-B^{2}$ |
| :---: | :---: | :---: | :---: | :---: |
| $(0,0)$ | $0$ | $a$ | $0$ | $-a^{2}$ |
| $(a,0)$ | $0$ | $-a$ | $-2a$ | $-a^{2}$ |
| $(0,a)$ | $-2a$ | $-a$ | $0$ | $-a^{2}$ |
| $(a/3,\,a/3)$ | $-\dfrac{2a}{3}$ | $-\dfrac{a}{3}$ | $-\dfrac{2a}{3}$ | $\dfrac{a^{2}}{3}$ |

这张表一看就清楚：前三个点的 $H=-a^{2}<0$，所以 $(0,0)$、$(a,0)$、$(0,a)$ 都不是极值点。最后一个点 $H=a^{2}/3>0$，接下来要看 $A=-2a/3$ 的符号，这就必须讨论参数 $a$ 的正负：$a>0$ 时 $A<0$，$\left(\dfrac{a}{3},\dfrac{a}{3}\right)$ 是极大值点；$a<0$ 时 $A>0$，是极小值点。

最后还缺一步：把 $(a/3,a/3)$ 代回去，极大值、极小值到底是多少？一算就出来：

$$f\left(\frac{a}{3},\frac{a}{3}\right)=\frac{a}{3}\cdot\frac{a}{3}\cdot\left(a-\frac{a}{3}-\frac{a}{3}\right)=\frac{a^{3}}{27}.$$

> **易错点**：小写的 $a$ 是题目给定的常数参数，大写的 $A$ 是 $f_{xx}$ 在驻点处的值，两者不是一回事。判别时先用 $H=AC-B^{2}$ 筛掉非极值点——它只含 $a^{2}$，与 $a$ 的符号无关；剩下的点再看 $A$ 的符号，把 $a$ 的正负讨论带进来。

### 例题：判别法失效时的直接分析

例题 12.6.2：设 $f(x,y)=x^{2}-2xy^{2}+y^{4}-y^{5}$，求这个函数的极值。

先找驻点：

$$f_x=2x-2y^{2}=0,\qquad f_y=-4xy+4y^{3}-5y^{4}=0.$$

由第一个方程得 $x=y^{2}$，代入第二个方程：$-4y^{3}+4y^{3}-5y^{4}=-5y^{4}=0$，立刻推出 $y=0$，同时 $x=0$。所以这个函数的驻点只有一点：$(0,0)$。

再上判别法。二阶偏导数：

$$f_{xx}=2,\qquad f_{xy}=-4y,\qquad f_{yy}=-4x+12y^{2}-20y^{3}.$$

把 $(0,0)$ 代进去：$A=2$，$B=0$，$C=0$，于是 $H=AC-B^{2}=0$。按刚才说的，$H=0$ 时判别不出任何结果，必须另行判断。

另行判断的办法，是把函数换一个写法：

$$f(x,y)=x^{2}-2xy^{2}+y^{4}-y^{5}=(x-y^{2})^{2}-y^{5}.$$

画个图来看。在 $xy$ 平面上，$x=y^{2}$ 是一条抛物线：

```text
        y
        ^
        |                 *
        |              *
        |           *        上半支（y > 0）：f = -y^5 < 0
        |        *
        |     *
        |  *
--------+-*-----------------> x
 (0,0)  |*   f(0,0) = 0
        |  *
        |     *
        |        *        下半支（y < 0）：f = -y^5 > 0
        |           *
        |              *
```

读图要点：如果 $(x,y)$ 恰好落在这条抛物线上，那么 $(x-y^{2})^{2}$ 这一项等于零，函数只剩下 $-y^{5}$。这一项有个鲜明的特点：上半平面（$y>0$）是负的，下半平面（$y<0$）是正的。而在驻点处 $f(0,0)=0$。

于是在 $(0,0)$ 的任何一个小邻域内：沿抛物线的上半支，函数值是负的，比 $f(0,0)=0$ 小；沿下半支，函数值是正的，比 $f(0,0)=0$ 大。假如 $(0,0)$ 是极大值点，邻域内的函数值都应当不超过它，可下半支上就偏有比它大的；假如是极小值点，上半支上就偏有比它小的。两头都说不通，所以 $(0,0)$ 不是极值点。

这个例子的手法值得记住：判别法只在 $H
e 0$ 时给答案；$H=0$ 时回到定义，在驻点附近找一条穿过它的曲线，看函数在这条曲线上是否变号——一旦既有比驻点值大的点、又有比它小的点，极值就没了。

### 推广到 n 元函数

前面从展开到判定，说的都是二元函数。但实际问题里很可能碰到三个变量甚至更多——作业当中就有这样的题：三元函数，怎么判断它是极大还是极小？这就需要一般的定理（定理 12.6.3）。

设 $y=f(x)$ 是 $n$ 元函数，$x=(x_1,x_2,\cdots,x_n)^{\mathsf{T}}$，$X_0=(x_1^0,x_2^0,\cdots,x_n^0)$ 是驻点，$f$ 有二阶连续偏导数。要判断 $X_0$ 是极大还是极小，考察二次型

$$G(\xi)=\sum_{i,j=1}^{n}\frac{\partial^{2}f}{\partial x_i\,\partial x_j}(X_0)\,\xi_i\xi_j=\sum_{i,j=1}^{n}a_{ij}\,\xi_i\xi_j,$$

其中 $\xi=(\xi_1,\xi_2,\cdots,\xi_n)^{\mathsf{T}}$，$a_{ij}$ 是 $f$ 关于 $x_i$、$x_j$ 的混合偏导数在驻点 $X_0$ 处的值。这个二次型同样有矩阵写法：

$$G(\xi)=(\xi_1,\cdots,\xi_n)\begin{pmatrix}a_{11} & \cdots & a_{1n}\\ \vdots & & \vdots\\ a_{n1} & \cdots & a_{nn}\end{pmatrix}\begin{pmatrix}\xi_1\\ \vdots\\ \xi_n\end{pmatrix},$$

系数矩阵 $A=(a_{ij})$ 是 $n\times n$ 的。结论与二元情形完全平行：

- 二次型 $G$ **正定**：驻点 $X_0$ 是极小值点；
- 二次型 $G$ **负定**：驻点 $X_0$ 是极大值点；
- 二次型 $G$ **不定**：驻点 $X_0$ 不是极值点。

回头看，这一讲处理的二元情形就是 $n=2$ 的特例：那里 $2\times 2$ 系数矩阵的行列式 $H=AC-B^{2}$ 配上 $A$ 的符号，恰好把正定、负定、不定区分开来。至于一般的 $n$ 元二次型怎么判定正定负定，那是高等代数要系统展开的内容。

### 本讲提到的题目

课上提到作业中有多元（三元）函数求极值的习题，但未给出具体题目，此处如实注明，不代拟。

> **承前启后**：上一节讲完「无条件极值的判定：从驻点到二次型」，下一节接着讲「n 元函数的极值判别与最值问题」。

---

## n 元函数的极值判别与最值问题
> 对应块：BLK147 | 覆盖分集：P154 | 块标题（分集名）：《156.第12章 第6节 无条件极值（3）》

先把前面留下的那件事说完整：$X_0$ 是多元函数 $f$ 的驻点、$f$ 有二阶连续偏导数时，要判断 $X_0$ 是极大值点、极小值点还是不是极值点，需要讨论一个二次型。为什么偏偏是二次型？因为这套判别法是从多元函数的泰勒公式里长出来的——二元的时候，我们就是用二元函数的泰勒公式得到了判断极值的充分条件，而这个充分条件，恰恰是对一个二次型的正定、负定来讲的。n 元函数一模一样。把这件事补齐之后，这一节的另一半是更实用的最值问题：极值的机器造好了，怎么求函数在一整块区域上的最大值、最小值。

### 从泰勒公式到二次型

处境先摆清楚。$X_0$ 是驻点，$X$ 是 $X_0$ 附近的一点，记 $X=X_0+\rho\xi$，其中 $\rho=\|X-X_0\|$ 是距离，$\xi$ 是归一化的方向向量：

$$\xi_1^2+\xi_2^2+\cdots+\xi_n^2=1$$

这个归一化条件跟二元情形的 $\cos^2\alpha+\sin^2\alpha=1$ 是同一件事——只看方向，不看远近。

对差 $f(X)-f(X_0)$ 用泰勒公式展开（这里用到了 $f$ 有二阶连续偏导数）：

$$f(X)-f(X_0)=\frac{\rho^2}{2}\sum_{i=1}^{n}\sum_{j=1}^{n}a_{ij}\,\xi_i\xi_j+o(\rho^2),\qquad a_{ij}=f_{x_ix_j}(X_0)$$

注意一次项整个不见了。为什么？因为 $X_0$ 正是驻点，一阶偏导全为零。于是真正干活的就是那个和式：$n\times n$ 个二阶偏导数值 $a_{ij}$ 排成一个矩阵，配上单位方向 $\xi$，构成一个二次型，记作

$$Q(\xi)=\sum_{i,j=1}^{n}a_{ij}\xi_i\xi_j$$

$o(\rho^2)$ 在 $\rho\to0$ 时是无穷小量，压不住前面的 $\rho^2$ 项。所以 $f(X)-f(X_0)$ 的正负，最终由二次型 $Q(\xi)$ 说了算：这个式子大于零，$X_0$ 是极小值点；小于零，是极大值点。于是问题归结为一件事：讨论这个二次型的正定与负定。

### 正定、负定、不定

> **定义**：对任意 $\xi \neq \mathbf 0$：
> - 若恒有 $Q(\xi)>0$，称二次型 $Q$ 正定，也称矩阵 $A_n=(a_{ij})_{n\times n}$ 正定；
> - 若恒有 $Q(\xi)<0$，称二次型 $Q$ 负定，也称矩阵 $A_n$ 负定；
> - 若 $Q$ 既取得到正值、又取得到负值，称二次型 $Q$ 不定。

"二次型正定"与"矩阵 $A_n$ 正定"是同一件事的两种说法，下面混着用，不加分。判定的结论跟二元时的讨论完全一样：

> **结论**：驻点 $X_0$ 处的二次型正定 $\Rightarrow$ $X_0$ 是极小值点；负定 $\Rightarrow$ $X_0$ 是极大值点；不定 $\Rightarrow$ $X_0$ 不是极值点。

道理过一遍：正定时任何方向上 $Q(\xi)>0$，而 $o(\rho^2)$ 是更高阶的小尾巴，只要 $X$ 离 $X_0$ 充分近，$f(X)-f(X_0)$ 就恒正——四周处处比 $X_0$ 高，所以是极小值点；负定反之，四周处处比它低，是极大值点；不定时，有的方向函数往上走、有的方向往下走，两边都刺穿，自然既非极大也非极小。

### 顺序主子式判别法

新问题来了：拿到一个具体的二次型（也就是拿到矩阵 $A_n$），怎么判断它正定、负定还是不定？高等代数里有一个现成的命题。

先把矩阵切片。记

$$A_k=\begin{pmatrix}a_{11}&a_{12}&\cdots&a_{1k}\\ a_{21}&a_{22}&\cdots&a_{2k}\\ \vdots&\vdots&\ddots&\vdots\\ a_{k1}&a_{k2}&\cdots&a_{kk}\end{pmatrix}\qquad(k=1,2,\dots,n)$$

也就是左上角那个 $k\times k$ 的小方块，称为 $k$ 阶顺序主子式；$k=n$ 时，$A_n$ 就是二次型的系数矩阵本身。

> **命题**：
> - 若 $\det A_k>0$ 对每个 $k=1,2,\dots,n$ 都成立，则 $A_n$ 正定（二次型正定）；
> - 若 $(-1)^k\det A_k>0$ 对每个 $k=1,2,\dots,n$ 都成立，则 $A_n$ 负定（二次型负定）。

> **提示**：$\det$ 是行列式的记号，来自英文 determinant，取前三个字母；$\det A_k$ 读作"$A_k$ 的行列式"。

还有一个马上要用到的性质：矩阵每个元素都变号，正定负定就互换——$A_n$ 正定当且仅当 $-A_n$ 负定，反过来 $-A_n$ 正定当且仅当 $A_n$ 负定。道理摆在明面上：二次型每一项的系数都添了个负号，整个二次型变号，原来恒正的现在恒负。这些结果以后你们学二次型的时候都会系统地见到，这里先拿来用。

于是 n 元函数求极值的路子全齐了：先找驻点，再在驻点处把二阶偏导排成矩阵 $A_n$，用顺序主子式定正定负定，落回"极大、极小、不是极值"。下面拿一个例子把这套流程实际走一遍。

### 一个 n 元指数函数的极值

$$f(x_1,x_2,\dots,x_n)=e^{-(x_1^2+x_2^2+\cdots+x_n^2)}$$

这个函数造得比较特殊。有的同学一眼就看出来：$x_1,\dots,x_n$ 全为零的时候指数为零，函数应该在这里达到极大值——图像像个山包，原点站在"肩膀"顶上。但极值点真是原点吗？这还不那么容易看。举这个例子的主要目的，是把刚给出的判别法实际用一遍。

第一步，求驻点。

$$\frac{\partial f}{\partial x_i}=-2x_i\,e^{-(x_1^2+\cdots+x_n^2)}=0\qquad(i=1,2,\dots,n)$$

指数因子恒正，$n$ 个方程联立只能得到 $x_1=x_2=\cdots=x_n=0$：驻点唯一，就是原点。

第二步，算二阶偏导。有两种：对同一个变量求两次的，和混合偏导。

$$\frac{\partial^2 f}{\partial x_i^2}=(4x_i^2-2)\,e^{-(x_1^2+\cdots+x_n^2)}$$

它落在矩阵的对角线上。混合偏导（$i \neq j$）则是

$$\frac{\partial^2 f}{\partial x_i\,\partial x_j}=4x_ix_j\,e^{-(x_1^2+\cdots+x_n^2)}$$

落在对角线之外。

第三步，把驻点坐标 $(0,\dots,0)$ 代进去：对角线上 $a_{ii}=-2$，对角线外 $a_{ij}=0$，所以

$$A_n=\begin{pmatrix}-2&&&\\&-2&&\\&&\ddots&\\&&&-2\end{pmatrix}=-2E$$

对应的二次型是

$$Q(\xi)=-2\xi_1^2-2\xi_2^2-\cdots-2\xi_n^2=-2\left(\xi_1^2+\cdots+\xi_n^2\right)$$

只要 $\xi \neq \mathbf 0$，它严格小于零。也可以这么看：$2E$ 显然正定，添个负号，$A_n=-2E$ 就负定。按判别法，驻点是极大值点。

再补一句：这个函数的驻点只有这一个，所以除了原点这个极大值点之外，它没有任何其他的极值点。

### 一元函数最值的老办法

极值讲到这里，接下来进一步，讲最值问题——函数的最值。

先回忆一元函数 $y=f(x)$，$x$ 属于闭区间 $[a,b]$ 时是怎么讲的。为什么总在闭区间上讲？因为假设了函数连续，而连续函数有一条定理：闭区间上的连续函数一定取到最大值和最小值。有了这条定理，"求最值"这个问题本身才有意义——最值确实存在，找就是了。

找的办法分两步。第一步在区间内部找候选：内部的最值点同时也是极值点，函数可导的话那里导数为零；不可导的点也得拿出来。第二步，拿端点来比。为什么端点必须比？看图说话：有一种图，最大值点压根不在内部——那一点导数不是零，可它就是最大值点，不比端点就把它漏了；还有一种图，内部冒出四个极值点（极大极小交替），这四个点还得跟 $a$、$b$ 一起，把函数值统统比一遍。数一数要比较几个点？内部候选点加两个端点，一个不落。

```text
 [a, b] 上找最值的候选清单：

   a ----•---------•------•---- b
        f'=0     不可导  f'=0     <- 区间内部：候选点

   候选总清单 = 内部候选点 + 端点 a、b
   所有函数值放在一起比大小，最大值、最小值就出来了
```

### 多元最值难在边界

换多元函数，比如 $z=f(x,y)$，定义在平面区域 $D$ 上。这里 $D$ 取闭区域，把边界也算在里面。为什么？多元函数里有与"闭区间定理"对应的那一条：紧集（有界闭区域）上的连续函数必有最大值最小值。一元是闭区间，多元是有界闭区域，边界必须收进来，最值的存在性才有保证。

内部的干法和一元完全一样：如果最值点落在区域内部，它同时就是极值点——能偏导的地方偏导数全为零（驻点），拿出来；不可偏导的点，也拿出来。

真正的差别在边界。一元函数的边界就是两个端点，比两下完事；二元函数的边界是一条闭曲线，上面有无穷多个点；三元函数的边界干脆是一张闭曲面。无穷多个点没法一个个比，必须先把函数在整个边界上的最大值、最小值求出来，再跟内部候选放到一起比较。所以从最值问题这个角度讲，多元函数要比一元函数复杂。

```text
 闭区域 D 上求 z = f(x,y) 的最值：

   |-- 内部：驻点（偏导全为零）+ 不可偏导的点 --> 算出函数值
   |-- 边界：一条闭曲线（二元）/ 一张闭曲面（三元），无穷多个点
   |         逐段代入化成一元函数，求出边界上的最大值与最小值
   `-- 比较：内部候选值 与 边界最值 放在一起 --> 全局最大值 / 最小值
```

流程说完了，下面两道例题把它完整走两遍。

### 三角形上的距离平方和

取一个直角三角形，顶点为 $O(0,0)$、$A(1,0)$、$B(0,1)$，三角形连同边界是闭的。在三角形上任取一点 $P(x,y)$，$P$ 到三个顶点有三段线段，三段的平方和跟 $P$ 的坐标有关。要问的是：$PO^2+PA^2+PB^2$ 的最大值与最小值是多少，在哪里取到。

```text
   B(0,1)
     |\
     | \
     |  *  P(x,y)
     |   \
     |    \
     +-----\
   O(0,0)   A(1,0)
```

两条直角边一条在 $y$ 轴上、一条在 $x$ 轴上，斜边是 $x+y=1$，$P$ 是闭三角形上的动点。这显然是一个最值问题。记

$$f(x,y)=PO^2+PA^2+PB^2=x^2+y^2+(x-1)^2+y^2+x^2+(y-1)^2=3x^2+3y^2-2x-2y+2$$

问题归结为：求这个二元函数在闭三角形上的最大值与最小值。

第一步，内部求驻点：$f_x=6x-2=0$，$f_y=6y-2=0$，马上得到驻点 $\left(\dfrac13,\dfrac13\right)$，函数值 $f\left(\dfrac13,\dfrac13\right)=\dfrac43$。但这个 $\dfrac43$ 是最大值还是最小值，或者两者都不是？现在谁也不知道，答案要靠比较——跟边界比。

第二步，边界分三段，逐段化成一元函数。

在 $OA$ 上 $y=0$，动点在这条边上跑，函数值就是 $3x^2-2x+2$：一条开口向上的抛物线，对称轴 $x=\dfrac13$。于是 $x=\dfrac13$ 处取最小值 $\dfrac53$；最大值在端点 $x=1$（顶点 $A$）取得，等于 $3$。

在 $OB$ 上 $x=0$，$f=3y^2-2y+2$：完全同理，$y=\dfrac13$ 处取最小值 $\dfrac53$，$y=1$（顶点 $B$）处取最大值 $3$。

在 $AB$ 上 $x+y=1$，把 $y=1-x$ 代进去，得 $f=6x^2-6x+3$（$x$ 仍在 $0$ 到 $1$ 之间）：对称轴 $x=\dfrac12$，最小值 $\dfrac32$ 在斜边中点取得；最大值 $3$ 在 $x=0$ 或 $x=1$ 取得——正好又是 $B$、$A$ 两个顶点。

| 边界段 | 化成的一元函数 | 最小值 | 最大值 |
| :---: | :---: | :---: | :---: |
| $OA$（$y=0$） | $3x^2-2x+2$ | $\dfrac53$，在 $x=\dfrac13$ | $3$，在 $x=1$ |
| $OB$（$x=0$） | $3y^2-2y+2$ | $\dfrac53$，在 $y=\dfrac13$ | $3$，在 $y=1$ |
| $AB$（$x+y=1$） | $6x^2-6x+3$ | $\dfrac32$，在 $x=\dfrac12$ | $3$，在 $x=0$ 或 $1$ |

第三步，把所有候选放到一起比：边界三段的最小值 $\dfrac53$、$\dfrac53$、$\dfrac32$ 都比内部的 $\dfrac43$ 大，最大值三段全是 $3$，都落在顶点上。

> **结论**：最小值 $\dfrac43$ 在内部驻点 $\left(\dfrac13,\dfrac13\right)$ 取得；最大值 $3$ 在两个顶点 $A(1,0)$、$B(0,1)$ 取得。

这道题讲完了，但里面还藏着一个值得多看一眼的东西：驻点 $\left(\dfrac13,\dfrac13\right)$ 正好是三角形的什么？重心。这件事可以推广：在平面上任意画一个三角形，要在其中找一点 $P$，使得 $PA^2+PB^2+PC^2$ 最小，这个点就应该在重心上。你们不妨回去自己算一下，非常容易的一件事情。

### 折铁板：让水槽截面最大

第二题有一块真实的铁板。铁板很长（图上用一串点表示），宽 24 cm。在两边各量出 $x$ cm，沿两条线折起来，截面折成一个等腰梯形——这实际上就成了一个水槽，让水从槽里流。

```text
         +--------------------+    <- top = (24 - 2x) + 2x*cos(a)
          \                  /
           \                /       side = x,  angle = a (to horizontal)
            \______________/        height = x * sin(a)

         bottom = 24 - 2x
```

读图：下底是板中间没折的部分，宽 $24-2x$；两条腰各长 $x$，与水平方向折起 $\alpha$ 角，每条腰的水平外推是 $x\cos\alpha$，所以上底比下底宽出 $2x\cos\alpha$；高为 $x\sin\alpha$。

目的是什么？让这个等腰梯形的面积最大——截面最大，水槽的流量就最大。这显然又是最值问题，而变量有两个：$x$ 和 $\alpha$，所以它不是一元函数的最值，是二元函数的最大值问题。

等腰梯形的面积：两底之和除以二，再乘高。上底 $(24-2x)+2x\cos\alpha$，下底 $24-2x$，高 $x\sin\alpha$，于是

$$A(x,\alpha)=\frac{\left[(24-2x)+2x\cos\alpha\right]+(24-2x)}{2}\cdot x\sin\alpha=24x\sin\alpha-2x^2\sin\alpha+x^2\sin\alpha\cos\alpha$$

范围先圈好：$x$ 从 $0$ 到 $12$——两条腰合计 $2x$ 不能超过板宽 $24$，这是肯定的；$\alpha$ 从 $0$ 到 $\dfrac{\pi}{2}$——折到两腰竖直为止，再往回翻，截面肯定更小。就在这块区域上求 $A$ 的最大值。

老流程：先在区域内部求驻点。

$$\frac{\partial A}{\partial x}=2\sin\alpha\,(12-2x+x\cos\alpha)=0$$

$$\frac{\partial A}{\partial \alpha}=x\left[24\cos\alpha-2x\cos\alpha+x\left(2\cos^2\alpha-1\right)\right]=0$$

注意现在是在内部求驻点：$\alpha \neq 0$，$\sin\alpha \neq 0$，可以约掉；$x>0$，也可以约掉。由第一式得

$$\cos\alpha=\frac{2x-12}{x}$$

代入第二式。式子看上去有点复杂，化起来很容易——含 $\dfrac{1}{x}$ 的项正好互相抵消，最后剩下

$$3x-24=0\qquad\Longrightarrow\qquad x=8$$

回代：$\cos\alpha=\dfrac{16-12}{8}=\dfrac12$，所以 $\alpha=\dfrac{\pi}{3}$。内部驻点唯一：$\left(8,\dfrac{\pi}{3}\right)$。代入 $A(x,\alpha)$ 算出这个驻点的函数值是 $48\sqrt3$，单位是 cm²。

关键一步来了：跟边界比。四条边界逐一检查。

$x=0$：面积肯定是零——两边不折，折起来的高度都没有了。$\alpha=0$：面积也是零——板摊平了，同样没有高度。

$x=12$：下底缩成零，两腰直接对接。代入化简：

$$A=24\cdot12\sin\alpha-2\cdot12^2\sin\alpha+12^2\sin\alpha\cos\alpha=144\sin\alpha\cos\alpha=72\sin2\alpha\le72$$

前两项正好消掉；由 $2\sin\alpha\cos\alpha=\sin2\alpha$ 知它不超过 $72$。

$\alpha=\dfrac{\pi}{2}$：两腰竖直，末项为零，

$$A=24x-2x^2=72-2(x-6)^2\le72$$

配方凑出来的：减 $72$ 正好是完全平方，而完全平方前面是负的，所以不超过 $72$。

| 边界 | 截面面积 | 上界 |
| :---: | :---: | :---: |
| $x=0$ | $0$ | 两腰不折，无高度 |
| $\alpha=0$ | $0$ | 板摊平，无高度 |
| $x=12$ | $72\sin2\alpha$ | $\le 72$ |
| $\alpha=\dfrac{\pi}{2}$ | $72-2(x-6)^2$ | $\le 72$ |

边界上的最大值是 $72$，内部那个驻点上则是 $48\sqrt3$。只要算一下就知道 $72<48\sqrt3$，这道题马上出来了：求的是最大值，内部驻点的函数值比边界上所有函数值都大，所以最大值就在这个驻点取得。

> **结论**：每边折起 $8$ cm，折角 $\dfrac{\pi}{3}$ 即 $60^\circ$，槽底宽 $24-16=8$ cm，此时水槽截面面积最大，为 $48\sqrt3\ \mathrm{cm}^2$。做成这样的水槽，水的流量最大——原因就是截面积最大。

到这里，多元函数的无条件极值就讲完整了：极值这一半，靠驻点加二次型的正定负定来判断；最值这一半，内部照极值的办法找候选，再把边界上的最值拉进来一起比。极值与最值，就讲到这里。

### 本讲提到的题目

课上留了一个"回去自己算一下"的问题，还原如下。

**题目**：在平面上任意画一个三角形 $ABC$，在其所在平面上找一点 $P$，使得 $PA^2+PB^2+PC^2$ 最小。这个 $P$ 应该在什么位置？

**解析**：设三顶点为 $A(x_1,y_1)$、$B(x_2,y_2)$、$C(x_3,y_3)$，记

$$f(x,y)=PA^2+PB^2+PC^2=\sum_{i=1}^{3}\left[(x-x_i)^2+(y-y_i)^2\right]$$

求偏导并令其为零：

$$f_x=6x-2(x_1+x_2+x_3)=0,\qquad f_y=6y-2(y_1+y_2+y_3)=0$$

解得唯一驻点

$$P_0=\left(\frac{x_1+x_2+x_3}{3},\ \frac{y_1+y_2+y_3}{3}\right)$$

这正是三条中线的交点——重心。再用本讲的判别法收尾：二阶偏导 $f_{xx}=6$、$f_{yy}=6$、$f_{xy}=0$，系数矩阵 $A=6E$ 的各阶顺序主子式 $\det A_k=6^k>0$，正定，所以 $P_0$ 是极小值点；又 $f$ 在全平面只有这一个驻点，且 $\|P\|\to\infty$ 时 $f\to+\infty$，这个极小值就是最小值。

**答案**：$P$ 取在三角形的重心上，此时 $PA^2+PB^2+PC^2$ 最小。

> **承前启后**：上一节讲完「n 元函数的极值判别与最值问题」，下一节接着讲「条件极值问题与 Lagrange 乘数法：从原点到一条直线的距离说起」。

---

## 条件极值问题与 Lagrange 乘数法：从原点到一条直线的距离说起
> 对应块：BLK148 | 覆盖分集：P155 | 块标题（分集名）：《157.第12章 第7节 条件极值问题与Lagr》

先看一个具体的几何问题。空间中有一条直线 $L$，由两张平面相交给出：

$$\begin{cases} x + y + z = 1, \\ x + 2y + 3z = 6. \end{cases}$$

两张平面相交，交出一条空间直线。现在要问：**原点到这条直线的距离是多少？**

垂直、投影，按解析几何的老办法当然能做。我们换一个角度来看这件事：直线上的点 $(x,y,z)$ 首先要满足这两个方程，其次它到原点有个距离——$\sqrt{x^2+y^2+z^2}$。于是整个问题可以等价地改写成：

> 在满足 $x+y+z=1$、$x+2y+3z=6$ 的那些点中，找使 $x^2+y^2+z^2$ 取最小值的点。

这个最小值开个根号，就是原点到直线的距离。我直接把根号去掉了，为什么敢去？开方是单调操作，不改变大小次序：平方和最小的点就是距离最小的点，前者的最小值开方就得到距离。

> **提示**：距离类问题先去根号。$x^2+y^2+z^2$ 与 $\sqrt{x^2+y^2+z^2}$ 在同一点取最小，前者的最小值开方即后者；求极值时带着根号只有麻烦没有好处。

改写之后，问题里的两个角色就有了名字：

> **定义**：要让它取最大或最小的那个函数叫**目标函数**，记作 $f(x,y,z)$；变量必须满足的等式叫**约束条件**，习惯写成 $g(x,y,z)=0$、$h(x,y,z)=0$ 的形式。

本题里 $f = x^2+y^2+z^2$，$g = x+y+z-1$，$h = x+2y+3z-6$（常数都挪到左边）。像这样"在约束条件下求目标函数的极值或最值"的问题，就叫**条件极值问题**（或条件最值问题）；换一个问题也可能是求最大值，框架完全一样。

### 两条假定：连续偏导，以及在约束点上满秩

推导之前，先把假定交代清楚，后面每一步都靠它们。

- 目标函数 $f$ 与约束函数 $g$、$h$ 都具有**连续偏导**；
- $g$、$h$ 关于 $(x,y,z)$ 的雅可比矩阵

$$\begin{pmatrix} \dfrac{\partial g}{\partial x} & \dfrac{\partial g}{\partial y} & \dfrac{\partial g}{\partial z} \\[6pt] \dfrac{\partial h}{\partial x} & \dfrac{\partial h}{\partial y} & \dfrac{\partial h}{\partial z} \end{pmatrix}$$

在**满足约束条件的点**处满秩（本题是 $2\times 3$ 矩阵，秩要达到 2，即约束的个数）。

第二个假定有个特别容易忽略的地方。"满秩"两个字，说的不是这个矩阵在全空间处处满秩——矩阵里的 $x,y,z$ 不是任意取的，它们必须满足约束条件。在本题里，约束交出一条直线，满秩只要求在这条直线上成立；直线外面的点满不满秩，跟我们毫无关系，因为我们讨论的条件极值，本来就只发生在约束集上。

> **易错点**：满秩只要求在"满足约束条件的点"处成立。脱离约束集去谈雅可比矩阵满不满秩，对象就搞错了。

### 消元降元：条件极值先变成无条件极值

接下来求**必要条件**：设 $(x_0,y_0,z_0)$ 是这个问题的条件极值点——它满足 $g=0$、$h=0$，并且使 $f$ 在约束下取到极大或极小。问：这样的点必须满足哪些关系式？

满秩意味着雅可比矩阵里至少有一个二阶行列式不为零。不妨就设

$$\frac{\partial(g,h)}{\partial(y,z)}\Bigg|_{(x_0,y_0,z_0)} = \begin{vmatrix} g_y & g_z \\ h_y & h_z \end{vmatrix}_{(x_0,y_0,z_0)} \neq 0$$

由隐函数存在定理，在 $x_0$ 的邻域内，约束方程组把 $y,z$ **唯一确定**为 $x$ 的函数：

$$y = y(x), \qquad z = z(x)$$

注意措辞：是"唯一确定"，不是"能解出表达式"。具体题目里 $y(x)$、$z(x)$ 很可能根本写不出来，但对推导来说，存在就够了。

把这两个函数代回目标函数：

$$\varphi(x) = f\big(x,\, y(x),\, z(x)\big)$$

想一想，现在成了几元函数？一元函数。原先"三个变量、两条约束"的条件极值，约束已经被 $y(x)$、$z(x)$ 吃了进去，变成了 $\varphi$ 的**无条件极值**问题：$(x_0,y_0,z_0)$ 是原问题的条件极值点，等价于 $x_0$ 是 $\varphi$ 的极值点。

无条件极值怎么办？一元函数极值的必要条件——一阶导数为零。而且因为 $f,g,h$ 都有连续偏导，$\varphi$ 确实可导，这条路是通的：

$$\varphi'(x_0) = 0$$

用链式法则展开（在 $x=x_0$ 处）：

$$f_x + f_y\,\frac{dy}{dx} + f_z\,\frac{dz}{dx} = 0$$

到这一步还只是机械的求导，真正有内容的是下一步：把式子**读出几何来**。

### φ′=0 的几何读法：梯度落进法平面

把 $\varphi'(x_0)=0$ 换个写法：

$$\big(f_x,\, f_y,\, f_z\big)\cdot\Big(1,\, \frac{dy}{dx},\, \frac{dz}{dx}\Big)\Bigg|_{(x_0,y_0,z_0)} = 0$$

这实际上是两个向量的点积。第一个向量是 $\operatorname{grad} f$（梯度）。第二个向量 $(1,\,y'(x),\,z'(x))$ 是什么？$g=0$ 与 $h=0$ 是两张曲面，两张曲面相交出一条曲线；沿着这条曲线以 $x$ 为参数写出的切向量，正是它。也就是说，$(1,\,y'(x_0),\,z'(x_0))$ 是约束曲线 $g=0\cap h=0$ 在点 $(x_0,y_0,z_0)$ 处的**切向量**，记作 $\mathbf{T}$。

于是条件说的是：$\operatorname{grad} f \cdot \mathbf{T} = 0$——**目标函数的梯度，与约束曲线的切向量垂直**。

```text
               grad g        grad h
                  \             /
                   \           /
                    +--- 法平面：由这两个向量张成
                               ^
               grad f ---------+   grad f 垂直于 T，必落进法平面
                               |
     ==========================●=========>  T = (1, y', z')
     （曲线 g = 0 与 h = 0 的交线，● 为条件极值点）
```

读图：竖直方向是过该点、与切向量 $\mathbf{T}$ 垂直的**法平面**（侧视成一条线）。$\operatorname{grad} g$ 与 $\operatorname{grad} h$ 是两张曲面的法向量，都与 $\mathbf{T}$ 垂直，所以都躺在法平面里；$\operatorname{grad} f$ 与 $\mathbf{T}$ 垂直，也只能落进这个平面。

为什么法平面恰好由 $\operatorname{grad} g$、$\operatorname{grad} h$ 张成？这是上一讲的定理 12.5.1：两曲面交出的曲线，在一点处的法平面由这两个向量张成。道理也直观：$g$ 沿这条曲线恒等于零，对曲线方向的方向导数为零，即 $\operatorname{grad} g\cdot\mathbf{T}=0$，$\operatorname{grad} g$ 与 $\mathbf{T}$ 垂直；$\operatorname{grad} h$ 同理。两个向量都进了法平面，而且（满秩保证了）线性无关，正好把这张平面张满。

这样一来，$\operatorname{grad} f$ 落在一个由 $\operatorname{grad} g$、$\operatorname{grad} h$ 张成的平面里——学过线性代数立刻知道，它必能表示成这两个向量的线性组合。于是**存在**两个数 $\lambda_0$、$\mu_0$（是多少暂时不知道，但一定存在），使得

$$\operatorname{grad} f(x_0,y_0,z_0) = \lambda_0\, \operatorname{grad} g(x_0,y_0,z_0) + \mu_0\, \operatorname{grad} h(x_0,y_0,z_0)$$

这就是条件极值点满足的核心关系，一句话：**在条件极值点处，目标函数的梯度落在诸约束函数的梯度张成的平面里。**

### Lagrange 函数：把五个方程打包成一个写法

上面的向量等式看着是一句话，展开是三个数量等式（都在 $(x_0,y_0,z_0)$ 处取值）：

$$\begin{aligned} f_x &= \lambda_0\, g_x + \mu_0\, h_x, \\ f_y &= \lambda_0\, g_y + \mu_0\, h_y, \\ f_z &= \lambda_0\, g_z + \mu_0\, h_z. \end{aligned}$$

再加上两条约束 $g=0$、$h=0$，一共**五个方程**。数一数不知道的量：$x_0$、$y_0$、$z_0$、$\lambda_0$、$\mu_0$，正好**五个未知数**。方程组解出来，把 $\lambda_0$、$\mu_0$ 一丢，剩下的 $(x,y,z)$ 就是**可能的条件极值点**——逻辑很清楚：真是条件极值点，就必定满足这五个方程；但满足方程组的点，未必真是极值点。

这五个方程背着写太累。换个说法，构造一个函数——**Lagrange 函数**（拉格朗日函数）：

$$L(x,y,z,\lambda,\mu) = f(x,y,z) - \lambda\, g(x,y,z) - \mu\, h(x,y,z)$$

写法上有讲究：第一项是目标函数；有几个约束条件，后面就接几项，每项是"一个乘数乘一个约束函数"。本题两个约束，就一个乘 $\lambda$、一个乘 $\mu$。这套做法也就叫 **Lagrange 乘数法**。注意 $L$ 的变量是**五个**：$x,y,z,\lambda,\mu$。

现在对 $L$ 求**无条件极值**——五个偏导全为零：

$$\begin{aligned} L_x &= f_x - \lambda g_x - \mu h_x = 0, \\ L_y &= f_y - \lambda g_y - \mu h_y = 0, \\ L_z &= f_z - \lambda g_z - \mu h_z = 0, \\ L_\lambda &= -g = 0, \\ L_\mu &= -h = 0. \end{aligned}$$

前三个偏导正好就是向量等式的三个分量，后两个正好就是约束条件——跟刚才那个五元方程组一对照，一模一样。于是得到本讲最要紧的一句话，先把它记住：

> **结论**：条件极值点 $(x_0,y_0,z_0)$ 必定包含在方程组 $
abla L = 0$ 的解 $(x,y,z,\lambda,\mu)$ 所对应的 $(x,y,z)$ 集合之中。求条件极值，先解这个方程组；解出来的只是**候选点**，不一定是真的极值点。

一看这个框架，很多人的第一反应是可怕：本来求个原点到直线的距离，怎么要解一个五元方程组？哪怕是五元一次方程组也够呛。两句话说清楚这件事。

其一，回代消元那条路往往走不通。推导里"从约束解出 $y(x)$、$z(x)$"靠的是隐函数存在定理，真到具体题目，大多数约束根本解不出表达式；表达式没有，代入就没得代。Lagrange 乘数法至少保证一件事：不管约束解不解得出来，方程组总能列出来。

其二，方程组看着大，解起来常有巧劲，究竟怎么巧，下面的例题一看便知。

|              | 直接消元代入                 | Lagrange 乘数法                 |
| :----------- | :--------------------------- | :------------------------------ |
| 变量个数     | 变少（每消一个约束少一元）   | 变多（每加一个约束多一个乘数）  |
| 前提         | 约束能显式解出某个变量       | 只需在约束点上满秩，不必显式可解 |
| 实际体验     | 式子少，但常常做不下去       | 方程组大，但常有巧妙解法        |

### 一般形式：n 个变量、m 个约束

把刚才的推导原样推广，就是书上的定理 12.7.1（必要条件）。目标函数是 $n$ 元函数 $f(x_1,\dots,x_n)$，约束条件是 $m$ 个：

$$g_i(x_1,\dots,x_n) = 0, \qquad i = 1, \dots, m$$

先问一句：$m$ 和 $n$ 什么关系？必须 $m < n$。要是 $m = n$，$m$ 个方程把 $n$ 个变量全部解死，变量没有任何自由余地，"极值"根本无从谈起，问题就没意思了。拿本题验算：$n=3$，$m=2$。约束还可以再少——把两张平面拿掉一张，问题就变成原点到一张平面的距离，照样是条件极值。总之约束个数可以是一个、两个，但必须比变量个数少。

条件照旧两条：$f$ 与各个 $g_i$ 都有连续偏导；雅可比矩阵

$$\left(\frac{\partial g_i}{\partial x_j}\right)_{\substack{i=1,\dots,m \\ j=1,\dots,n}}$$

（$m$ 行 $n$ 列）在**满足约束条件的点**处满秩。

结论的句式和三维情形一字不差。构造

$$L = f - \sum_{i=1}^{m} \lambda_i\, g_i$$

则条件极值点 $x^0 = (x_1^0,\dots,x_n^0)$ 必包含在下述方程组的解所对应的 $x$ 集合中：

$$\frac{\partial L}{\partial x_k} = \frac{\partial f}{\partial x_k} - \sum_{i=1}^{m}\lambda_i\,\frac{\partial g_i}{\partial x_k} = 0, \qquad k = 1,\dots,n$$

$$g_l(x_1,\dots,x_n) = 0, \qquad l = 1,\dots,m$$

未知量 $x_1,\dots,x_n,\lambda_1,\dots,\lambda_m$ 一共 $n+m$ 个，方程也正好 $n+m$ 个。刚才的 $x,y,z,\lambda,\mu$ 就是 $n=3$、$m=2$ 的特例。

### 充分条件：看 L 关于 x 的二阶矩阵

必要条件给的是候选——方程组的解只是"可能"的条件极值点，是不是真极值、是极大还是极小，得靠充分条件（定理 12.7.2）说话。

设 $(x^0, \lambda^0)$——即 $x_1^0,\dots,x_n^0,\lambda_1^0,\dots,\lambda_m^0$——是上面方程组的一组解。造一个 $n\times n$ 矩阵：

$$\left(\frac{\partial^2 L}{\partial x_k\,\partial x_l}\right)_{k,l=1,\dots,n}\Bigg|_{(x^0,\,\lambda^0)}$$

两个要点：求偏导**只对 $x$ 的变量做**，$\lambda$ 不参与（虽然 $\lambda$ 也是 $L$ 的变量）；取值要把乘数也一起代进去，就在方程组解出来的那一点。判定规则：

- 这个矩阵**正定** $\Longrightarrow$ $x^0$ 是条件**极小**值点；
- 这个矩阵**负定** $\Longrightarrow$ $x^0$ 是条件**极大**值点。

> **易错点**：这个二阶矩阵只对 $x_1,\dots,x_n$ 求偏导，$\lambda$ 不参与求导；但取值点必须连同乘数 $\lambda_1^0,\dots,\lambda_m^0$ 一起代入。

精神上与无条件极值的二阶判别法同款：一阶条件圈出候选，二阶形态定出极大极小——只是这里判别对象换成了 $L$（不是 $f$），而且要在乘数一并解出的点处取值。

### 例题精解：不解 x、y、z，直接读出距离

理论整了一套，回到开头那道题，看看"五元方程组"到底可不可怕。

目标函数照旧去掉根号：$f = x^2+y^2+z^2$（求出条件极小值后开方就是距离）；约束：$g = x+y+z-1$，$h = x+2y+3z-6$。两个约束，配两个乘数，Lagrange 函数是五个变量 $x,y,z,\lambda,\mu$ 的函数：

$$L = x^2 + y^2 + z^2 - \lambda(x+y+z-1) - \mu(x+2y+3z-6)$$

求无条件极值，五个偏导为零：

$$L_x = 2x - \lambda - \mu = 0, \qquad (1)$$

$$L_y = 2y - \lambda - 2\mu = 0, \qquad (2)$$

$$L_z = 2z - \lambda - 3\mu = 0, \qquad (3)$$

$$L_\lambda = 0:\ x+y+z = 1, \qquad L_\mu = 0:\ x+2y+3z = 6$$

硬解当然可以：先由 (1)(2)(3) 把 $x,y,z$ 用 $\lambda,\mu$ 表示，再代进两条约束。但我们来点巧的。

**盯住目标函数。** 它是 $x^2+y^2+z^2$，而 (1)(2)(3) 的左边恰好有 $2x$、$2y$、$2z$——把三式分别乘上 $x$、$y$、$z$ 再相加：

$$2x^2+2y^2+2z^2 - \lambda(x+y+z) - \mu(x+2y+3z) = 0$$

后两项被约束条件钉死：$x+y+z=1$、$x+2y+3z=6$。于是

$$2f - \lambda - 6\mu = 0 \quad\Longrightarrow\quad f = \frac{\lambda}{2} + 3\mu$$

先停一下，想一想这个式子说明了什么。它是从方程组推出来的：方程组的任何一组解都满足它。所以——如果解方程组时发现 $(\lambda,\mu)$ 只有唯一一组解，那 $\dfrac{\lambda}{2}+3\mu$ 这个数就不可能是别的值，只可能是**原点到直线距离的平方**。也就是说，$x_0,y_0,z_0$ 根本不用解！

那就只剩一件事：把 $\lambda$、$\mu$ 解出来。还是不碰 $x,y,z$，用加权求和把它们消掉。

(1)(2)(3) 三式直接相加（权重 $1,1,1$）：

$$2(x+y+z) - 3\lambda - 6\mu = 0 \;\Longrightarrow\; 3\lambda + 6\mu = 2$$

(1) 加 2 倍的 (2) 加 3 倍的 (3)（权重 $1,2,3$）：

$$2(x+2y+3z) - 6\lambda - 14\mu = 0 \;\Longrightarrow\; 6\lambda + 14\mu = 12$$

为什么挑这两组权重？因为 $1,1,1$ 与 $1,2,3$ 正是两条约束左边的系数——加权之后 $x,y,z$ 恰好凑成约束的左边，整个换成常数（$14 = 1\cdot 1 + 2\cdot 2 + 3\cdot 3$）。两个方程、两个未知数，立即解出

$$\mu = 4, \qquad \lambda = -\frac{22}{3}$$

代回枢纽式：

$$f = \frac{1}{2}\Big(-\frac{22}{3}\Big) + 3\cdot 4 = -\frac{11}{3} + 12 = \frac{25}{3}$$

所以原点到直线的距离

$$d = \sqrt{\frac{25}{3}} = \frac{5}{\sqrt{3}} = \frac{5\sqrt{3}}{3}$$

有同学想进一步问：垂足在哪？也容易。$\lambda$、$\mu$ 都已知了，代回 (1)(2)(3)：

$$x = \frac{\lambda+\mu}{2} = -\frac{5}{3}, \qquad y = \frac{\lambda+2\mu}{2} = \frac{1}{3}, \qquad z = \frac{\lambda+3\mu}{2} = \frac{7}{3}$$

垂足就是 $\big(-\tfrac{5}{3},\, \tfrac{1}{3},\, \tfrac{7}{3}\big)$。不过刚才的问题只问距离——距离到手，这道题就做完了。

回头看：整个方程组一个都没硬解。靠的是两眼——一眼盯着**目标函数的形状**（平方和，就用 $x,y,z$ 去乘三个一阶方程再相加，目标函数自己蹦出来）；一眼盯着**约束的系数**（按 $1,1,1$ 和 $1,2,3$ 加权，$x,y,z$ 全部换成常数）。很多条件极值问题都有这类巧劲，具体的下一讲接着看。

### 整套流程串起来

最后把做题流程钉死：

```text
目标函数 f，约束条件 g = 0、h = 0
        |
        v
构造 Lagrange 函数：L = f - λg - μh
        |
        v
令 L 的全部偏导为零（原变量各一个 + 乘数各一个）
        |
        v
解方程组，丢掉乘数，得候选点（可能的条件极值点）
        |
        v
用充分条件或问题的实际背景，定极大还是极小
```

读图：约束在构造 $L$ 时就"进"到了函数里，所以第三步求的是 $L$ 的**无条件**极值；乘数的个数跟着约束个数走——一个约束配一个 $\lambda$，两个约束用 $\lambda$ 和 $\mu$，三个就用 $\lambda_1,\lambda_2,\lambda_3$。

下节课继续讲其他类型的例题；作业课上已布置，具体题目此处不展开。

> **承前启后**：上一节讲完「条件极值问题与 Lagrange 乘数法：从原点到一条直线的距离说起」，下一节接着讲「拉格朗日乘数法：一般理论与两个例题」。

---

## 拉格朗日乘数法：一般理论与两个例题
> 对应块：BLK149 | 覆盖分集：P156 | 块标题（分集名）：《158.第12章 第7节 条件极值问题与Lagr》

上一讲我们开始讲条件极值问题，也就是用拉格朗日乘数法来解的这一类问题。当时做的例子是求一点到空间一条直线的距离：自变量 $x,y,z$ 三个，约束条件两个。具体题会做了，但那只是特例——今天先把一般情况的拉格朗日乘数法理论讲清楚，再拿例题练手。

### 从上节课的例子到一般框架

一般的目标函数是 $n$ 元函数

$$f(x_1,x_2,\dots,x_n),$$

约束条件有 $m$ 个：

$$g_i(x_1,x_2,\dots,x_n)=0,\qquad i=1,2,\dots,m,\quad m<n .$$

约束个数 $m$ 要小于变量个数 $n$——上节课的例子里 $n=3$、$m=2$，正是这个框架的特例。

前提假设有两条。其一，$f$ 与各 $g_i$ 都具有一阶连续偏导数。其二，把 $m$ 个约束函数对 $n$ 个变量求偏导，得到雅可比矩阵（Jacobian）

$$J=\left(\frac{\partial g_i}{\partial x_j}\right)_{m\times n},$$

它是 $m$ 行 $n$ 列的矩阵，要求它在**满足约束条件的点处满秩**。

> **注意**：一阶连续偏导，加上雅可比矩阵在约束点处满秩——这是使用拉格朗日乘数法首先需要的条件。

在这个框架下，求目标函数在 $m$ 个约束之下的最大值、最小值，做法是定义拉格朗日函数：

$$L(x_1,\dots,x_n;\lambda_1,\dots,\lambda_m)=f(x_1,\dots,x_n)-\sum_{i=1}^{m}\lambda_i\,g_i(x_1,\dots,x_n).$$

> **定义**：上式中的 $L$ 称为拉格朗日函数，$\lambda_1,\lambda_2,\dots,\lambda_m$ 称为拉格朗日乘数（Lagrange multiplier）。每个约束前面配一个乘数，$m$ 个约束共 $m$ 个。

上次已经讲了：求约束条件下的极大极小、最大最小，就转化为求 $L$ 这个函数的**无条件极值**。转化过来之后怎么办？偏导等于零。注意现在的偏导要对**所有**未知量求：除了 $x_1$ 到 $x_n$，对 $\lambda_1$ 到 $\lambda_m$ 的偏导也都要求等于零，一个都不能少。

### 必要条件：n+m 个方程锁定候选点

把偏导具体写出来。对 $x_k$ 求偏导（$k=1,\dots,n$）：

$$\frac{\partial L}{\partial x_k}=\frac{\partial f}{\partial x_k}-\sum_{i=1}^{m}\lambda_i\frac{\partial g_i}{\partial x_k}=0,$$

$k$ 从 1 取到 $n$，一共 $n$ 个方程。对 $\lambda_l$ 求偏导（$l=1,\dots,m$）——$L$ 里带 $\lambda_l$ 的只有一项，所以求出来的也只有一项：

$$\frac{\partial L}{\partial \lambda_l}=-g_l(x_1,\dots,x_n)=0 .$$

这最后一组方程带着个负号，实际使用时一般把它拿掉，直接写 $g_l=0$。

两部分合在一起，记这个方程组为（＊）：它一共有 $n+m$ 个方程，而未知量——$x_1$ 到 $x_n$、$\lambda_1$ 到 $\lambda_m$——恰好也是 $n+m$ 个。

> **结论**（教材定理 12.7.1，必要条件）：条件极值点可以有好多个，但每一个条件极值点——不管哪一个——都包含在方程组（＊）的解集合之中。解出来之后把 $\lambda_1,\dots,\lambda_m$ 扔掉，剩下的 $x$ 部分才是我们关心的点。

换句话说：解（＊）得到的只是候选名单，真正的条件极值点个个都藏在里面；至于名单上哪些点真是极值点、是极大还是极小，就要靠下面的充分条件来检验了。

### 充分条件：一个 n 阶矩阵的正负定

充分条件（教材定理 12.7.2）是这样说的：设从方程组（＊）里解出了一组解 $(x_0;\lambda_1,\dots,\lambda_m)$，其中 $x_0=(x_1^0,\dots,x_n^0)$。现在要判断的是这个点——$\lambda$ 不管它——是条件极值问题中的极大值点还是极小值点。除了（＊），还要添一个条件：$f$ 与各 $g_i$ 具有二阶连续偏导数。然后看一个矩阵

$$H=\left(\frac{\partial^2 L}{\partial x_k\,\partial x_l}\right)_{n\times n},$$

它是 $L$ 关于 $x_1,\dots,x_n$ 的二阶偏导排成的 $n\times n$ 矩阵。注意它在哪一点取值：就在 $(x_0;\lambda_1,\dots,\lambda_m)$ 这一点上——$L$ 的自变量有 $n+m$ 个，取值的时候 $x_0$ 与那组 $\lambda$ 一个都不能少。

| $H$ 在该点的正负性 | 能下的结论 |
| :--- | :--- |
| 正定 | $x_0$ 是条件极小值点 |
| 负定 | $x_0$ 是条件极大值点 |
| 不定 | 定理什么都没说——但别急着反着下结论，下面有反例 |

定理摆出来，我要问两个问题——大家对这两处有没有一点怀疑？

第一处：条件极值问题不是划成了 $L$ 的无条件极值吗？$L$ 有 $n+m$ 个自变量，那么按无条件极值的二阶判别，矩阵似乎应该是 $n+m$ 阶的；可现在这个矩阵里只有对 $x_k$、$x_l$ 的二阶偏导，没有对 $\lambda$ 的——这好像不对，是不是？

第二处：这个结论好像还缺一条。有同学会想：如果这个矩阵不定，那这个点应该既不是极大值点、也不是极小值点——对不对？翻翻书：书上没有这句话。没有这句话，它就成了一个疑问了。

#### 为什么判别矩阵是 n 阶而不是 n+m 阶

判断一个点 $x_0$ 是极大还是极小，标准只有一个：在约束曲面上看差 $f(x)-f(x_0)$ 的正负——当然，参与比较的 $x$ 与 $x_0$ 都得满足约束条件。第一步先做一个改写。设 $(x_0;\lambda_1,\dots,\lambda_m)$ 满足方程组（＊），取同一组 $\lambda$，那么

$$f(x)-f(x_0)=L(x,\lambda_1,\dots,\lambda_m)-L(x_0,\lambda_1,\dots,\lambda_m).$$

这个等式对不对？把 $L$ 的定义代进去验证一下：右边等于 $f(x)-f(x_0)-\sum_i\lambda_i\big[g_i(x)-g_i(x_0)\big]$，而 $x$ 与 $x_0$ 都满足约束、$g_i(x)=g_i(x_0)=0$，整串 $\lambda$ 项消掉，剩下的正是左边。注意两边用的是同一组、固定不动的 $\lambda$。

接下来对 $L$ 用泰勒公式。关键在这里：$\lambda_1,\dots,\lambda_m$ 两边相同，在泰勒展开里 $\lambda$ **没有变化**，变化的只有 $x$——$x_0$ 变到了 $x$。记 $\delta x=x-x_0$，$\rho=\|\delta x\|$。由于 $(x_0;\lambda)$ 是 $L$ 的驻点、一阶偏导全为零，一次项全部消失，展开式只剩二次项和余项：

$$L(x,\lambda)-L(x_0,\lambda)=\frac{1}{2}\sum_{k,l=1}^{n}\frac{\partial^2 L}{\partial x_k\,\partial x_l}\,\delta x_k\,\delta x_l+o(\rho^2)=\frac{1}{2}\,\delta x^{\mathsf T}H\,\delta x+o(\rho^2).$$

到这里第一个疑问就解开了：比较的两点 $\lambda$ 相同、只有 $x$ 在动，展开式里压根不出现 $\lambda$ 的增量，起作用的自然只有关于 $x$ 的那块 $n\times n$ 二阶偏导矩阵——这就是判别用的矩阵不是 $n+m$ 阶的原因。

再把 $\delta x=\rho\,\xi$ 代进去，其中 $\xi=\delta x/\rho$ 是范数为 1 的单位向量，式子换一种写法：

$$L(x,\lambda)-L(x_0,\lambda)=\rho^2\left(\frac{1}{2}\,\xi^{\mathsf T}H\,\xi+o(1)\right).$$

从这里就看得出定理的结论了。$H$ 正定，意味着 $\xi^{\mathsf T}H\xi$ 在单位球面这个紧集上有正的下界，于是 $\rho$ 充分小时括号里恒正、差恒正——$x_0$ 是条件极小值点；$H$ 负定同理，差恒负——条件极大值点。这一条书上没有证，这里作一个比较简单的补充说明。

#### 矩阵不定还能是极值点吗——一个反例

第二个疑问更要紧，答案要明确告诉你：书上没有"不定就非极值"这句话，而且这句话是不对的。要说明它不对，举个反例就够了。

取目标函数

$$f(x,y,z)=x^2+y^2-z^2,$$

约束条件为 $g(x,y,z)=z=0$。拉格朗日函数是

$$L=x^2+y^2-z^2-\lambda z .$$

求它的无条件极值，一阶偏导全为零：

$$2x=0,\qquad 2y=0,\qquad -2z-\lambda=0,\qquad z=0 .$$

马上推出：这个方程组只有一个解 $(0,0,0;\lambda=0)$。在这个点上算判别用的矩阵——现在是 $3\times 3$ 的（关于 $\lambda$ 的偏导进不了这个矩阵）：

$$H=\begin{pmatrix}2&0&0\\0&2&0\\0&0&-2\end{pmatrix}.$$

$x$ 方向上是 $2$，$z$ 方向上是 $-2$，这个矩阵显然不定。按"不定就非极值"的直觉，原点应该不是条件极值点。我们说：不对。看看约束 $z=0$ 之下目标函数变成了什么：

$$f(x,y,z)\big|_{z=0}=x^2+y^2\ge 0=f(0,0,0).$$

所以原点是条件极小值点——干脆就是条件最小值。

为什么"不定"还能是极值点？条件极值点怎么样，关键是看那个差。矩阵不定，说的是二次型 $\tfrac{1}{2}\,\xi^{\mathsf T}H\xi$ 对**一切**单位向量 $\xi$ 可正可负——这是 $\xi$ 不受任何限制时的说法。可在我们这个问题里，$\xi=\delta x/\rho$，而 $\delta x$ 是两个都满足约束的点之差：约束逼着 $z\equiv 0$，于是 $\delta z=0$，$\xi$ 的第三个分量 $\xi_3=\delta z/\rho=0$。把 $\xi_3=0$ 代回去，二次型只剩

$$2\xi_1^2+2\xi_2^2\ \ge\ 0,$$

它做不到既可正又可负了。换句话说：不定是对整个空间说的；在约束允许的方向上，这个二次型仍然是定的——所以原点照样是条件极小值点。

> **易错点**：定理 12.7.2 没有第三句话。$H$ 不定时定理保持沉默、什么都不能断言，但绝不能反过来写"不定则既非极大也非极小"——上面的反例里 $H$ 不定，那个点却是货真价实的条件极小值点。

到这里，必要条件与充分条件都齐了。把拉格朗日乘数法的完整操作流程串起来看：

```text
条件极值问题：最值化 f(x1,…,xn)，约束 gi(x1,…,xn)=0（i=1..m，m<n）
      │  构造拉格朗日函数 L = f − Σ λi·gi，引入乘数 λ1..λm（未知数共 n+m 个）
      ▼
解方程组（＊）：∂L/∂xk = 0（k=1..n）与 gl = 0（l=1..m），共 n+m 个方程
      │  求出全部解，把 λ 扔掉、只留 x —— 得到候选点
      ▼
逐点计算 n×n 矩阵 H = (∂²L/∂xk∂xl)（在“该点 + 对应 λ”处取值）
      │
      ▼
H 正定 → 条件极小值点；H 负定 → 条件极大值点；H 不定 → 不能下结论
```

一句话读图：$\lambda$ 是搭桥用的——先引进来把约束"吃"进方程组，解完再扔掉；最后交给正负定判别去检验的，只是那批候选点。

### 例题：平面截椭球的截面面积

理论摆完了，下面来举些例子。这道是教材上的例题 12.7.2：求平面 $x+y+z=0$ 去截椭球面 $x^2+y^2+4z^2=1$ 所得的截面面积。

先想清楚一件事：这张平面是斜的，它截椭球截出来的曲线到底是什么？是不是椭圆？为什么？能说清楚吗？

#### 为什么截面一定是椭圆

可以这样想：把坐标轴旋转一下。坐标轴旋转，实际上相当于对 $(x,y,z)$ 做一个正交变换

$$\begin{cases}x'=a_1x+b_1y+c_1z,\\ y'=a_2x+b_2y+c_2z,\\ z'=a_3x+b_3y+c_3z,\end{cases}$$

其中系数 $a_1,b_1,c_1,\dots,a_3,b_3,c_3$ 构成一个正交矩阵。转的目的只有一个：把这张斜平面变成新坐标系里的 $x'=0$（或者 $z'=0$，都行）。

新坐标系下椭球的方程怎么变？无非是把 $x,y,z$ 从变换里反解出来代回去——正交矩阵可逆，代进去得到的仍是一个二次方程。再令 $x'=0$，就得到新坐标里两个变量的方程。注意这个方程的特点：次数是**二次**的，所以截面是一条二次曲线。平面上的二次曲线总共三种：椭圆、双曲线、抛物线；而这条截线是椭球面上的闭曲线，椭球有界、它也有界——有界的二次曲线只能是椭圆。所以这个截面绝对是个椭圆。

#### 把面积问题化成距离的最值

椭圆的面积，数学分析里学过：长短半轴分别为 $a,b$ 时，$S=\pi ab$。于是问题变成：把这条椭圆的长半轴、短半轴找出来。

目标函数取

$$F(x,y,z)=x^2+y^2+z^2,$$

注意这实际上是到原点的距离的平方。这条椭圆的中心在哪里？在原点——平面 $x+y+z=0$ 过原点，椭球的中心也在原点。所以要求的就是原点到椭圆的距离的最大值与最小值：最大的是长半轴 $a$，最小的是短半轴 $b$，你说对不对？

约束条件两个，恰好就把这条椭圆表示出来：

$$g_1:\ x+y+z=0,\qquad g_2:\ x^2+y^2+4z^2-1=0 .$$

这样，求椭圆面积就转化为了条件极值问题：三个自变量、两个约束，点 $(x,y,z)$ 落在椭圆上，求它到原点距离的最大与最小。用什么方法？拉格朗日乘数法——两个约束，配两个乘数 $\lambda$ 与 $\mu$。

#### 解方程组的两个巧劲

拉格朗日函数：

$$L=x^2+y^2+z^2-\lambda(x+y+z)-\mu\left(x^2+y^2+4z^2-1\right).$$

求它的无条件极值，一阶偏导全为零，连同两条约束一起：

$$\begin{aligned}&(1)\quad 2x-\lambda-2\mu x=0,\\ &(2)\quad 2y-\lambda-2\mu y=0,\\ &(3)\quad 2z-\lambda-8\mu z=0,\\ &(4)\quad x+y+z=0,\\ &(5)\quad x^2+y^2+4z^2=1.\end{aligned}$$

有同学就说了：五个未知数的方程组，而且不是一次的，这不好解。上次就讲过，对这样的方程组**不要死解**，要想一些好的办法。这里有两个巧劲。

第一个巧劲：把 (1)(2)(3) 分别乘上 $x,y,z$ 再相加。$2x\cdot x=2x^2$、$2y\cdot y=2y^2$、$2z\cdot z=2z^2$ 凑在一起，$\lambda x+\lambda y+\lambda z$ 恰好是 $\lambda(x+y+z)$，$2\mu x^2+2\mu y^2+8\mu z^2$ 恰好是 $2\mu(x^2+y^2+4z^2)$，于是

$$2(x^2+y^2+z^2)-\lambda(x+y+z)-2\mu(x^2+y^2+4z^2)=0 .$$

大家现在想想：这个等式有什么用处？把 (4)(5) 代进去，第二个括号是零、第三个括号是 1，马上得到

$$x^2+y^2+z^2=\mu .$$

> **提示**：这一步是整道题的枢纽——乘数 $\mu$ 不是配角，它恰好就是椭圆上的点到原点距离的平方。所以整个方程组不必真解出 $x,y,z$，只要解出 $\mu$ 就够了；而 $\mu$ 会有两解，大的就是长半轴的平方 $a^2$，小的就是短半轴的平方 $b^2$。

第二个巧劲：$\mu$ 怎么解出来？看 (1)(2)(3) 的结构：$x,y$ 的系数都是 $2(1-\mu)$，$z$ 的系数是 $2(1-4\mu)$。让它们变整齐——$(1)$ 乘 $(1-4\mu)$、$(2)$ 乘 $(1-4\mu)$、$(3)$ 乘 $(1-\mu)$，再三式相加：三个方程中 $x,y,z$ 的系数全变成 $2(1-\mu)(1-4\mu)$，提出来后剩下的恰好是 $x+y+z$，由 (4) 它等于零。于是整个式子里只剩 $\lambda$ 与 $\mu$：

$$-\lambda\big[(1-4\mu)+(1-4\mu)+(1-\mu)\big]=0\quad\Longrightarrow\quad \lambda(1-3\mu)=0 .$$

这个等式包含两个因子，所以 $\mu=\dfrac{1}{3}$，或者 $\lambda=0$。

$\mu=\dfrac{1}{3}$ 直接给出一个值；另一个值要从 $\lambda=0$ 出发。此时 (1)(2)(3) 变成 $2(1-\mu)x=0$、$2(1-\mu)y=0$、$2(1-4\mu)z=0$。问一句：$\mu$ 能不能不等于 1？假如 $\mu \neq 1$，马上推出 $x=0$、$y=0$；再由 (4) 又推出 $z=0$。可是 $(0,0,0)$ 不在椭球面上——它不满足 (5)。矛盾，所以只能 $\mu=1$。

于是 $\mu$ 的两个值是 $\dfrac{1}{3}$ 与 $1$：大的 $1$ 给出 $a^2=1$，小的 $\dfrac{1}{3}$ 给出 $b^2=\dfrac{1}{3}$，即

$$a=1,\qquad b=\frac{1}{\sqrt3},\qquad S=\pi ab=\frac{\pi}{\sqrt3}=\frac{\sqrt3\,\pi}{3} .$$

你们不要看到"哎呀，这个方程组太可怕了"——拿到这种方程组，就要想个办法，用巧妙的方法把它解出来。

### 闭圆域上的二次函数：内部与边界两步走

再来一道，教材例题 12.7.3：求函数

$$f(x,y)=ax^2+2bxy+cy^2$$

的最大值与最小值，其中系数满足 $ac-b^2>0$——把它当作二次三项式来看，判别式 $b^2-ac<0$；另外 $a,b,c$ 都是大于零的。区域 $D$ 是闭圆域：

$$D:\qquad x^2+y^2\le 1 .$$

$D$ 是平面上的一个紧集——有界又闭，紧集上的连续函数一定取得最大值与最小值，所以这两个值必定存在，问题只在于怎么把它们找出来。做法分两步走。

第一步，先求 $f$ 在 $D$ 的内部（$x^2+y^2<1$）的极值点。这是普通的无条件极值问题，偏导为零解出来即可。

第二步，再求 $f$ 在边界 $x^2+y^2=1$ 上的最大值与最小值。圆周是一条约束曲线，边界上的最值恰恰是一个条件极值问题——这一节的拉格朗日乘数法，正是为这一步准备的。

> **承前启后**：上一节讲完「拉格朗日乘数法：一般理论与两个例题」，下一节接着讲「条件极值三例：二次型最值、幂乘积与最优价格模型」。

---

## 条件极值三例：二次型最值、幂乘积与最优价格模型
> 对应块：BLK150 | 覆盖分集：P157 | 块标题（分集名）：《159.第12章 第7节 条件极值问题与Lagr》

求一个函数在闭区域上的最大值与最小值，方案是两步走：第一步，求函数在区域内部的极值点——区域内部没有任何约束，这是无条件极值问题；第二步，到边界上去，边界自带约束方程，这是一个条件极值问题，得请出 Lagrange 乘数法。这一讲就把这套方案完整地走三遍：一道二次型在闭单位圆上的最值题，一道 $n$ 元幂乘积的条件极值题（解完顺手把均值不等式捡出来），最后是一道带数学模型味道的经济学题——电视机厂怎么定价。

> **提示**：闭区域上的最值 = 内部（无条件极值）+ 边界（条件极值）。两步各算各的，最后放在一起比大小。

### 先攻内部：一个现成的无条件极值问题

题目是：设 $a>0$ 且 $ac-b^2>0$，求

$$F(x,y) = ax^2 + 2bxy + cy^2$$

在闭区域 $x^2+y^2\le 1$ 上的最大值与最小值。

第一步，先在区域内部求极值点。内部没有约束，这就是无条件极值，令一阶偏导为零：

$$F_x = 2ax + 2by = 0, \qquad F_y = 2bx + 2cy = 0.$$

把二全划掉（约去公因子 2），得到齐次线性方程组

$$\begin{cases} ax + by = 0, \\ bx + cy = 0. \end{cases}$$

它的系数行列式

$$\begin{vmatrix} a & b \\ b & c \end{vmatrix} = ac - b^2,$$

由条件它大于零，当然不等于零，于是这个二元一次方程组只有零解：$x=0$，$y=0$。区域内部就这一个驻点。

那么这个点到底是极大值点还是极小值点？只要算一下二阶：记 $A = F_{xx}(0,0) = 2a$，$B = F_{xy}(0,0) = 2b$，$C = F_{yy}(0,0) = 2c$，于是

$$AC - B^2 = 4(ac-b^2) > 0, \qquad A = 2a > 0,$$

马上知道这个驻点是极小值点：$(0,0)$ 是极小值点，极小值 $F(0,0) = 0$。内部的问题到此做完。

### 再攻边界：圆周上的条件极值

第二步，求 $F$ 在条件 $x^2+y^2=1$ 下的极值——换句话说，就是在区域的边界上。约束摆在那里，这是一个条件极值问题，构造 Lagrange 函数：

$$L(x,y,\lambda) = ax^2 + 2bxy + cy^2 - \lambda\left(x^2 + y^2 - 1\right).$$

接下来求 Lagrange 函数的无条件极值：

$$L_x = 2ax + 2by - 2\lambda x = 0, \qquad L_y = 2bx + 2cy - 2\lambda y = 0.$$

把公因子 2 约掉，写成干净的形式：

$$(a-\lambda)x + by = 0, \qquad bx + (c-\lambda)y = 0,$$

再配上约束方程

$$x^2 + y^2 = 1,$$

三个方程联立，现在来解这个方程组。

### 为什么一门心思要解出 λ

先看前两个方程：它们构成一个关于 $x,y$ 的二元一次方程组。这个方程组能不能有零解？不能——因为约束条件说 $x^2+y^2=1$，$(0,0)$ 根本不在圆周上。

> **易错点**：齐次方程组要有非零解，系数行列式必须为零。这里"有非零解"不是假设出来的，而是约束直接送来的：$x^2+y^2=1$ 把零解排除在外，行列式才被迫等于零。

于是系数行列式必须等于零：

$$\begin{vmatrix} a-\lambda & b \\ b & c-\lambda \end{vmatrix} = 0,$$

展开就得到 $\lambda$ 的二次方程

$$\lambda^2 - (a+c)\lambda + (ac-b^2) = 0,$$

它有两个根：

$$\lambda = \frac{1}{2}\left[a+c \pm \sqrt{(a+c)^2 - 4(ac-b^2)}\right].$$

那为什么我们非要解出 $\lambda$ 不可？这里有一个很漂亮的观察。把第一个方程乘上 $x$，第二个方程乘上 $y$，再加起来：

$$ax^2 + bxy + bxy + cy^2 - \lambda x^2 - \lambda y^2 = 0,$$

也就是

$$ax^2 + 2bxy + cy^2 = \lambda\left(x^2 + y^2\right).$$

而在圆周上 $x^2+y^2=1$，于是

$$F(x,y) = \lambda.$$

> **结论**：边界驻点处目标函数的值恰好等于 $\lambda$ 本身。$\lambda$ 有两个解，大的那个就是 $F$ 在圆周上的极大值，小的那个就是极小值；圆周是整个边界，这里的极大极小就是最大最小。

### 内外合并，最大值最小值落定

圆周上这两个数——加号那支没问题，是正的；减号那支呢？用根与系数的关系看：两根之和 $a+c>0$（由 $a>0$、$ac-b^2>0$ 得 $ac>0$，进而 $c>0$），两根之积 $ac-b^2>0$——和与积都是正的，两个根只能都是正的。

现在把两步放在一起：

| 范围 | 问题类型 | 结果 |
| :--- | :--- | :--- |
| 圆内 $x^2+y^2<1$ | 无条件极值 | 唯一驻点 $(0,0)$，是极小值点，值为 $0$ |
| 圆周 $x^2+y^2=1$ | 条件极值（Lagrange） | 驻点处 $F=\lambda$，最大值取加号的大根，最小值取减号的小根 |

内部唯一的极值点是极小值点，最大值只可能在边界上取到；而边界上的最小值（小根）也是正的，比内部的 $0$ 还大，所以整个闭区域上的最小值就是 $0$。合并成一句话：

$$\max_{x^2+y^2\le 1} F = \frac{1}{2}\left[a+c+\sqrt{(a+c)^2-4(ac-b^2)}\right], \qquad \min_{x^2+y^2\le 1} F = F(0,0) = 0.$$

这个问题在提的时候，只让你求最大值、最小值，没有说最大值点、最小值点——所以我们没有必要把取得最大值时的 $x,y$ 算出来。

### 幂乘积求极值：先取对数，把乘法变加法

下一道例题（教材例题 12.7.4）。设 $a_1, a_2, \dots, a_n$ 都是正数，求目标函数

$$f(x_1, x_2, \dots, x_n) = x_1^{a_1}\, x_2^{a_2} \cdots x_n^{a_n}$$

在约束条件

$$x_1 + x_2 + \cdots + x_n = A, \qquad x_i > 0$$

下的极大值与极小值。也就是说，$n$ 个数的和确定的情况下，它的最大值是多少——把这一问做出来。

麻烦在于 $f$ 是一串乘积，要计算就要求导，这个式子求导蛮讨厌的。怎么办？换一个目标函数：令

$$g = \ln f = a_1\ln x_1 + a_2\ln x_2 + \cdots + a_n \ln x_n.$$

对数函数是单调的，求出 $g$ 的最大值，$f$ 的最大值也就跟着出来了；约束条件原封不动。求导从一串乘积变成逐项求和，这就好办了。

> **提示**：乘积型（或幂型）目标函数先取对数，是最优化里的常用招数——单调性保证最值点不变，而求导难度直线下降。

### 驻点只有一组：用 Hessian 认定最大值

一条约束，挂一个乘数，Lagrange 函数是

$$L(x_1,\dots,x_n,\lambda) = \sum_{i=1}^{n} a_i \ln x_i - \lambda\left(x_1 + x_2 + \cdots + x_n - A\right).$$

一阶条件：

$$\frac{\partial L}{\partial x_i} = \frac{a_i}{x_i} - \lambda = 0 \quad (i = 1, \dots, n), \qquad \frac{\partial L}{\partial \lambda} = -\left(x_1 + \cdots + x_n - A\right) = 0.$$

从前 $n$ 个方程立刻读出 $x_i = \dfrac{a_i}{\lambda}$；代进最后一条，$\dfrac{1}{\lambda}\left(a_1+\cdots+a_n\right) = A$，于是

$$\lambda = \frac{a_1 + a_2 + \cdots + a_n}{A}, \qquad x_i = \frac{A\,a_i}{a_1 + a_2 + \cdots + a_n} \quad (i=1,\dots,n).$$

方程组只解出这一组可能的点。但它对我们的条件极值问题来说是极大还是极小？这要看二阶。算 $\dfrac{\partial^2 L}{\partial x_k \partial x_l}$——这是一个 $n\times n$ 阶矩阵。它其实好算：$\ln x_i$ 求两次导只剩 $-\dfrac{a_i}{x_i^2}$，交叉偏导全是零，具体不算了，直接写出来：

$$\left(\frac{\partial^2 L}{\partial x_k \partial x_l}\right)_{n\times n} = \begin{pmatrix} -\dfrac{a_1}{x_1^2} & & 0 \\ & \ddots & \\ 0 & & -\dfrac{a_n}{x_n^2} \end{pmatrix}.$$

矩阵是对角的，非对角元全是零。这些 $a_i$ 都是正的，$x_i$ 也是正的，把负号拿到矩阵外面，剩下的是对角元全正的对角阵——正定；挂上负号，整个矩阵就是一个负定矩阵。负定说明什么问题？说明

$$(x_1^0, x_2^0, \dots, x_n^0) = \left(\frac{A\,a_1}{\sum_j a_j},\ \frac{A\,a_2}{\sum_j a_j},\ \dots,\ \frac{A\,a_n}{\sum_j a_j}\right)$$

是极大值点。

再问一句：这个函数的极大值点有几个？只有这一个，没有其他的极值点了。那么这个极大值点实际上就是最大值点。把 $x_i^0$ 代回 $f$，干脆用一个乘积的写法：

$$\max f = \prod_{i=1}^{n} \left(\frac{A\,a_i}{a_1 + a_2 + \cdots + a_n}\right)^{a_i}.$$

这个最大值公式看上去很复杂，用处可能也不大，不会用到这么复杂。那就考虑它的特例。

### 特例一转，均值不等式自己冒出来

取 $A=1$，同时所有的 $a_i=1$。$A=1$ 是什么意思？就是 $n$ 个数加起来等于 1；$a_i$ 全是 1，目标函数就成了 $n$ 个数相乘。问题变成：$n$ 个正数加起来等于 1，它们相乘最大是多少？把 $x_i^0 = \dfrac{1\cdot 1}{n} = \dfrac{1}{n}$ 代进公式——这里是 $n$ 分之一，但要相乘 $n$ 次——得到

$$x_1 x_2 \cdots x_n \le \left(\frac{1}{n}\right)^n \qquad \left(x_i > 0,\ \ x_1 + \cdots + x_n = 1\right).$$

再往前推一步。对任意 $n$ 个正数 $y_1, y_2, \dots, y_n$——它们没有任何约束条件——令

$$x_i = \frac{y_i}{y_1 + y_2 + \cdots + y_n}, \qquad i = 1, \dots, n.$$

这组 $x_i$ 是正的，而且加起来恰好等于 1，套用刚才的结论：

$$\frac{y_1}{y_1+\cdots+y_n}\cdot\frac{y_2}{y_1+\cdots+y_n}\cdots\frac{y_n}{y_1+\cdots+y_n} \le \left(\frac{1}{n}\right)^n,$$

也就是

$$\frac{y_1 y_2 \cdots y_n}{\left(y_1 + y_2 + \cdots + y_n\right)^n} \le \frac{1}{n^n}.$$

两边开 $n$ 次根号：分子是 $\left(y_1 y_2\cdots y_n\right)^{1/n}$；分母是同一个数乘了 $n$ 次，开出来还是它自己：

$$\frac{\sqrt[n]{y_1 y_2 \cdots y_n}}{y_1 + y_2 + \cdots + y_n} \le \frac{1}{n}.$$

> **结论**：对任意 $n$ 个正数 $y_1,\dots,y_n$，都有 $\dfrac{y_1+y_2+\cdots+y_n}{n} \ge \sqrt[n]{y_1 y_2 \cdots y_n}$——算术平均大于等于几何平均。

从一道条件极值题出发，我们顺手把这个经典不等式也证出来了。

### 把生意翻译成数学：最优价格模型

最后一道例题带点数学模型的味道，属于经济学方面的内容：最优价格模型。书上的这个模型，是针对一家电视机生产企业来说的。

先把变量立起来。产量用 $x$ 表示，并且把它同时看成销售量。按常规讲，产量是多少、销售量是多少，这两个是不相等的；但如果厂里价格定义得比较合理，产量多少正好能够销售完——积压了很多，厂子肯定容易造成亏损——所以至少产量跟销售量是很接近的，我们用同一个 $x$。价格用 $v$ 表示，成本用 $c$ 表示。

这些量之间相互都有制约关系。价格定得高，销售量肯定低下来；价格定得低，销售量就高，这时候要求的产量也高。成本跟产量也有关系：你生产一台电视机是一个价钱，生产 1 万台电视机，每台的成本就不一样了——越是产量大，成本就会减少。

按照经验公式，制约关系写成两条：

$$x = M e^{-\alpha v}, \qquad c = c_0 - k\ln x.$$

第一条里请大家注意这样一点：假如 $v$ 趋向于零——价格趋近零当然是不可能的，意思是电视机卖得非常非常便宜、简直等于送了——那么 $x$ 就趋向 $M$。这个 $M$ 是什么？是社会的总需求量：便宜到等于送，社会上需要多少台就能卖掉多少台。第二条里请注意：$x=1$ 时 $c=c_0$，所以 $c_0$ 就是生产一台电视机的成本，这是最高的；产量越来越大，成本在减少。

那么这个模型什么意思？生产的电视机投放到市场上，定一个价格，目标是什么？任何企业都是追求最大利润。利润是价格减掉成本，乘上产量（产量跟销售量近似相等）：

$$u(x,v,c) = (v-c)\,x.$$

```text
     x = M·e^(−αv)                    c = c0 − k·ln x
价格 v ─────────> 产量 x（≈ 销量 x）─────────> 单位成本 c
   │                   │                          │
   └───────────────────┼──────────────────────────┘
                       ↓
          利润 u = (v − c)·x，要它越大越好
```

一句话读图：价格通过需求公式压住销量，销量又通过规模效应压住单位成本，而利润 $u$ 把三个变量拴在一起；那两条经验公式，正是 Lagrange 乘数法要面对的两条约束。

### 三个条件，四个参数全部定死

模型里有四个待定参数：$M$、$\alpha$、$c_0$、$k$。书上给了一家电视机厂的三个条件，一个一个对号入座。

第一个条件：当地的总需求量是 100 万台——这就是 $M=100$ 万。意思是：电视机卖得非常非常便宜、几乎不要钱了，100 万台肯定卖得掉。

第二个条件：去年售出 10 万台，售价是 4000 块。回到 $x = Me^{-\alpha v}$：$M$ 已经知道，$x$ 取 10 万时 $v$ 是 4000，$\alpha$ 就解出来了。

第三个条件：生产一台电视机的成本 $c_0 = 4000$ 块——巧了，还是 4000 块一台；又说批量生产以后，产量到 1 万台时每台的成本是 3000 块。回到 $c = c_0 - k\ln x$：$c_0$ 已知，$x=1$ 万时 $c$ 是 3000，$k$ 也就解出来了。

| 给出的条件 | 数据 | 定出的参数 |
| :--- | :--- | :--- |
| 当地总需求量 | 100 万台 | $M$ |
| 去年销量与售价 | 10 万台、4000 块 | $\alpha$（代回 $x=Me^{-\alpha v}$） |
| 单台生产的成本 | 4000 块 | $c_0$ |
| 批量 1 万台时的单位成本 | 3000 块 | $k$（代回 $c=c_0-k\ln x$） |

到这一步，$c_0$、$k$、$M$、$\alpha$ 全部知道了。

### 两条约束、两个乘数：五个方程解出最优价格

现在追求最大利润：在上面两条约束条件下求目标函数的最大值。这个经济学上的模型称为最优价格模型，它归结到最后是什么？是一个在约束条件下求目标函数最大的条件极值问题。那怎么办？用 Lagrange 乘数法。未知量是 $x,v,c$，约束有两条，所以挂两个乘数 $\lambda_1, \lambda_2$：

$$L(x,v,c,\lambda_1,\lambda_2) = (v-c)x - \lambda_1\left(x - Me^{-\alpha v}\right) - \lambda_2\left(c - c_0 + k\ln x\right).$$

原来的问题就转化为求这个 Lagrange 函数的无条件极值。中间的计算过程大家都知道了，这里把消元的次序补上。令 $L_x = L_v = L_c = 0$，再与两条约束

$$x = Me^{-\alpha v}, \qquad c - c_0 + k\ln x = 0$$

放在一起，共五个方程，未知量 $x,v,c,\lambda_1,\lambda_2$ 也恰好五个。先由两个好下手的驻点方程把乘数定下来：

$$L_v = x - \lambda_1 \alpha x = 0 \ \Rightarrow\ \lambda_1 = \frac{1}{\alpha}, \qquad L_c = -x - \lambda_2 = 0 \ \Rightarrow\ \lambda_2 = -x.$$

代回 $L_x = (v-c) - \lambda_1 - \lambda_2\dfrac{k}{x} = 0$：

$$v - c - \frac{1}{\alpha} + k = 0 \quad\Longrightarrow\quad c = v + k - \frac{1}{\alpha}.$$

另一边，对 $x = Me^{-\alpha v}$ 取对数得 $\ln x = \ln M - \alpha v$，代进成本约束：

$$c = c_0 - k\ln x = c_0 - k\ln M + k\alpha v.$$

两个 $c$ 相等，解出

$$v^* = \frac{c_0 - k\ln M + \dfrac{1}{\alpha} - k}{1 - k\alpha}.$$

这就是这道题最后要的答案。$c_0$、$k$、$M$、$\alpha$ 刚才已经全知道，代进去就得到这个电视机厂的最优定价：生产的电视机投放到市场上，按照这个价格去出售，可以获得最大的利润。

这叫做最优价格模型，而这个模型在解决的过程当中，用到的正是 Lagrange 乘数法。

---

## 本册小结

本册主题为「多元微分学与极值：Euclid空间、偏导数、Taylor公式与Lagrange乘数法（下）」，整编了 12 个模块（P145 ~ P157，共 12 讲 / 542 分钟音频）。
建议配合 `notes/` 目录下的复习笔记复盘，需要查证细节时可直接回到本册正文。
