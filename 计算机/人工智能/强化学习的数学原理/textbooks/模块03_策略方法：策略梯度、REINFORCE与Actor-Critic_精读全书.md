# 【强化学习的数学原理】课程：从零开始到透彻理解（完结）·策略方法：策略梯度、REINFORCE与Actor-Critic（第 3 册 / 共 3 册）

> **所属课程**：【强化学习的数学原理】课程：从零开始到透彻理解（完结）  
> **本册内容**：策略方法：策略梯度、REINFORCE与Actor-Critic  
> **覆盖范围**：P46-P50 ~ P51-P54（共 9 讲 / 2 章 / 104 分钟音频）  
> **分册说明**：全书按内容分 3 册；本册为第 3 册  
> **整编说明**：正文逐字保留各块模块长文，仅补导读、目录与章间过渡。

---

## 导读与全景目录

1. 策略梯度方法：目标函数、梯度与 REINFORCE，兼看最简单的 Actor-Critic（P46-P50）
2. Actor-Critic 进阶：A2C、重要性采样与确定性策略梯度（P51-P54）

---

## 策略梯度方法：目标函数、梯度与 REINFORCE，兼看最简单的 Actor-Critic
> 对应块：BLK12 | 覆盖分集：P46-P50 | 块标题（分集名）：《第9课-策略梯度方法（该方法的目标函数1-Ave》

上一讲我们已经把 policy gradient 的基本思路讲清楚了，我相信这个思路本身还是比较简单的。下面就要基于这个基本思路展开比较具体的介绍。这件事分成三个部分：第一，我们选中的这个 metric，也就是 objective function，到底是什么；第二，这个 objective function 所对应的 gradient 是什么；第三，我们怎么用这个 gradient 来做优化，得到最优的策略。本讲就沿着这条线一路走下来——先立目标函数，再算梯度，最后套进梯度上升的算法里，得到第一个、也是最简单的 policy gradient 算法 REINFORCE；在结尾，我们会顺其自然地看到下一阶段的主角——Actor-Critic 方法是怎么从这个框架里长出来的。

### 目标函数之一：平均状态值

大家读书或者读论文的时候，经常会遇到两大类的 metric。第一大类叫 average state value，或者叫 average value。它其实就是 state value 的一个加权平均：

$$
\bar{v}_\pi=\sum_{s\in\mathcal{S}}d(s)\,v_\pi(s)
$$

右边 $v_\pi(s)$ 是状态 $s$ 对应的 state value，$d(s)$ 是给 $s$ 的一个权重，这个权重大于等于 0，并且所有权重之和等于 1。把它们加起来，得到的就是 $v_\pi(s)$ 的加权平均，这个数就是我们的 metric，用 $\bar{v}_\pi$ 表示——上面那一横线代表的就是"平均"的意思。

> **定义**：$\bar{v}_\pi$ 是所有状态的 state value 按权重 $d(s)$ 加权平均得到的标量。它是策略的函数——不同的策略对应的值不同——所以我们可以去优化它，找一个最优策略让这个值达到最大。用 state value 的加权平均来做 metric，是非常自然的一个选择。

$d(s)$ 不仅大于等于 0、加起来等于 1，在这种情况下它不仅可以理解成权重，也可以理解成 probability distribution——$d(s)$ 代表了状态 $s$ 被选中的概率。于是这个 objective function 就可以完全按照 expectation 的定义来写：

$$
\bar{v}_\pi=\mathbb{E}_{S\sim d}\big[v_\pi(S)\big]=\sum_{s}p(s)\,v_\pi(s)
$$

这里的 $p(s)$ 就是对应的 $d(s)$，两个式子实际上是一样的。进一步地，这样一个"相乘然后再相加"的式子，显然可以写成更简洁的形式——两个向量的内积：

$$
\bar{v}_\pi=d^T v_\pi
$$

$d$ 和 $v_\pi$ 都对应两个向量，每个元素对应一个状态：$d(s)$ 是状态 $s$ 的权重或概率，$v_\pi(s)$ 是 $s$ 的 state value。这个比较简洁的内积形式，在我们后面分析怎么求 $\bar{v}_\pi$ 的梯度时会用到。

不过这个 metric 还没有介绍完，为什么？因为里面有一个 $d$——$\bar{v}_\pi=\sum_s d(s)v_\pi(s)$ 里的这个 $d(s)$ 我们还没讲怎么选。选法有两种情况，我们一个一个来看。

#### d 与策略无关：d₀

第一种情况是 $d(s)$ 和 $\pi$ 没什么关系，它是独立的。这种情况相对来说比较简单。为什么简单？看内积形式 $\bar{v}_\pi=d^Tv_\pi$ 就明白了：如果 $d$ 和 $\pi$ 没有关系，那我待会儿求梯度的时候，$d$ 就不涉及任何梯度，只求一个 $v_\pi$ 的梯度就行；如果 $d$ 和 $\pi$ 有关系，那求梯度的时候当然也要求 $d$ 关于 $\pi$ 的梯度，麻烦一点。为了表明 $d$ 和 $\pi$ 没有关系，我们把它写成 $d_0$，相对应的 metric 就写成 $\bar{v}_{\pi_0}$。

那 $d_0$ 怎么选呢？最容易想到的情况，就是把所有的状态视为 equally important——每一个状态的权重或概率都相同。一共有 $n$ 个状态，每个状态的权重都是 $1/n$，相加恰好等于 1，这实际上就是一个均匀分布。

另外一个非常重要的情况是：我可能非常关心某一个状态 $s_0$。比如在一些游戏最开始的时候，画面总是从那个场景开始，那个画面就对应一个特定的状态 $s_0$。从那出发，我希望我能得到的 reward 越大越好。在这种情况下就不能对所有状态一视同仁了，要对一些状态有所偏好。极端情况下就是只关心 $s_0$：给 $d_0(s_0)=1$，其他所有状态的权重都是 0。这时 $\bar{v}_\pi$ 也就变成了 $v_\pi(s_0)$——最大化这个 metric，实际上就是最大化从 $s_0$ 出发我能得到多大的 return。

#### d 依赖于策略：平稳分布 dπ

第二种情况是 $d$ 和 $\pi$ 有关系，这是一种非常常见的选择：选 $d$ 为 $d_\pi(s)$，它叫 stationary distribution（平稳分布）。大家对 $d_\pi$ 应该比较熟悉，因为上节课我们做了比较详细的介绍，书里也有，这里只简要地说一下。

简单来说：我有一个策略，跟随这个策略不断地和环境交互，执行很多很多次之后，就可以预测 agent 在某一个状态的概率是多少——也就是逐渐达到一个平稳的状态。而且这个概率可以直接通过式子计算出来：

$$
d_\pi^T P_\pi = d_\pi^T
$$

$P_\pi$ 是 state transition probability matrix。知道了 $P_\pi$ 之后其实可以直接把 $d_\pi$ 求解出来。当然这个 $P_\pi$ 我们也是不知道的，反正它满足这样一个式子。更多的展开这里就不做了，一展开东西就多了去了——不熟悉或感兴趣的话，回到上节课或书里看细节。用一句话概括：$d_\pi$ 是依赖于 $\pi$ 的一个 distribution。

这种选法的直观含义是：根据某一个策略去执行，肯定有的状态访问得多一点，有的状态访问得少一点。访问多的那个状态，它对应的概率 $d_\pi(s)$ 相对大一点，我们给它的权重也就大一点；访问少的状态权重自然就少一点。

### 目标函数之二：平均奖励

下面说第二个 metric。刚才是 average value，现在是 average reward，或者叫 average one-step reward：

$$
\bar{r}_\pi=\sum_{s\in\mathcal{S}}d_\pi(s)\,r_\pi(s)
$$

和刚才对比，$v_\pi(s)$ 换成了 $r_\pi(s)$——它是从 $s$ 出发我所得到的单步的、也就是 immediate reward 的一个平均值；$d_\pi(s)$ 是 $s$ 所对应的权重，从下标就能看出来，它实际上是 stationary distribution，依赖于策略 $\pi$。把 $r_\pi(s)$ 做加权平均，得到的这个 metric 就是 $\bar{r}_\pi$，上面的横线同样是代表平均。因为 $d_\pi(s)$ 是一个概率分布，这个和式也可以写成 expectation 的形式。

那 $r_\pi(s)$ 的表达式具体是什么？我在状态 $s$ 其实有很多个 action 可以选择：选 action $a$ 的概率是 $\pi(a|s)$，选完 action $a$ 之后得到的 reward 是 $r(s,a)$，把它们加起来：

$$
r_\pi(s)=\sum_{a\in\mathcal{A}}\pi(a\mid s)\,r(s,a)
$$

这里的 $r(s,a)$ 代表什么呢？是我在 $s$ take action $a$ 之后，所得到的 immediate reward 的平均值——对这个 reward 求一个平均就是 $r(s,a)$，它也可以按期望的形式写出来。所以总的链条就是：$r(s,a)$ 是我在 $(s,a)$ 得到的 immediate reward，对 $a$ 求加权平均（求 expectation）得到 $r_\pi(s)$，这是在 $s$ 所得到的 immediate reward 的平均值；然后再对 $s$ 求期望、做加权平均，就得到了第二个 metric $\bar{r}_\pi$。

#### 论文里常见的另一种写法

$\bar{r}_\pi$ 还有另外一种形式，大家读论文和书的时候会经常遇到。它是这么来的：有一个策略，根据这个策略形成了一条 trajectory，沿着这条 trajectory 得到很多 reward：$R_{t+1}, R_{t+2}$，等等。把这些 reward 加起来，先求 expectation（因为这些都是随机变量），再除以 $n$ 求平均，最后取极限：

$$
\bar{r}_\pi=\lim_{n\to\infty}\frac{1}{n}\,\mathbb{E}\left[\sum_{t=1}^{n}R_{t+1}\right]
$$

这个式子代表什么？就是从某一个状态出发跑无穷多步，但这时不是求所有 reward 的和，而是跑无穷多步之后 reward 的平均是多少。注意这里前面那个出发点 $s_0$ 没有了，为什么？因为 $s_0$ 不起作用——跑了无穷多步之后，你最开始从哪开始其实已经不再重要了。

> **提示**：这是大家在论文当中可能会经常看到的一个式子。当你看到它的时候，希望你能明白：它就是 $\bar{r}_\pi$，也就是能够写成 $d_\pi(s)\,r_\pi(s)$ 加权平均的那个东西。为什么两种写法等价？具体细节涉及一些数学，但其实也没那么复杂，书里有；总而言之，未来当你看到这个 metric 时，不要忘了我们在课程里讲过它。

### 关于这两个 metric 的三点补充

刚刚介绍了两个 metric，下面对它们做进一步的补充，之后会有一个小小的例子。

第一点：这些 metric 全都是策略的函数。策略是一个函数，它的参数是 $\theta$，所以无论是 $\bar{v}_\pi$ 还是 $\bar{r}_\pi$，它们都是 $\theta$ 的函数。不同的 $\theta$ 会得到不同的 metric 值，里面就存在一个最优的——我们希望能够去优化，找到最优的 $\theta$ 去最大化这些 metric。这就是 policy gradient 的基本思路。

第二点：如果你仔细分析，这个 metric 还是会比较麻烦的，麻烦在它分两种情况。一种是 discounted case，里面有一个 discount rate $\gamma$，是小于 1 的数；另一种是 undiscounted case，$\gamma$ 等于 1。到目前为止整本书只介绍了 discounted case。为什么会出现 undiscounted case？就是因为刚才介绍的 $\bar{r}_\pi$ 这个 metric，它是对 immediate reward 求平均，不是对 return 求平均——不求 return，就不需要考虑 discount rate。所以对 discounted 和 undiscounted 两种情况它都成立，仔细分析也可以有 undiscounted 的情况存在。当然这个情况比较复杂，课程里不再介绍，感兴趣可以到书里看。

第三点：$\bar{r}_\pi$ 和 $\bar{v}_\pi$ 什么关系？先说直观：$\bar{r}_\pi$ 好像比 $\bar{v}_\pi$ 更近视（short sighted），因为它好像只关心 immediate reward，而 $\bar{v}_\pi$ 关心 return——跑了很多步之后把所有 reward 加起来。那是不是这种情况呢？实际上这两个 metric 是等价的。具体来说，在 discounted case 可以证明：

$$
\bar{r}_\pi=(1-\gamma)\,\bar{v}_\pi
$$

注意我说的"等价"不是说这俩相等，而是说它满足这样一个等式。那么自然地，对其中一个做优化的时候，另外那个也达到了极值。这个式子怎么得到的，书里可以看。

> **易错点**："等价"不是"$\bar{r}_\pi=\bar{v}_\pi$"，而是差一个 $(1-\gamma)$ 的因子。优化其中一个等价于优化另一个，这才是"等价"的含义。

### 一道小练习：认出论文里的另一种写法

下面有一个小小的练习题。如果在课堂上，我会让大家停下来去做一下；屏幕前的你如果感兴趣也可以这么做。

问题是这样的：你读论文和很多参考资料的时候，会经常见到一个 metric——在 policy gradient 的问题当中经常见到的这么一个式子。这个 metric 和我们刚才介绍的 metric 有什么联系吗？

我可以直接告诉大家（你当然也可以先自己想一想）：还记得吗？$\bar{r}_\pi$ 有两个定义，一个是 $\sum_s d_\pi(s)r_\pi(s)$，另一个是 $\lim_{n\to\infty}\frac{1}{n}\mathbb{E}\sum_{t=1}^n R_t$。实际上 $\bar{v}_\pi$ 也有两个定义：一个是 $\sum_s d(s)v_\pi(s)$，另一个就等于下面这个——沿着一条 trajectory 把 discounted return 求期望：

$$
\mathbb{E}_{S_0\sim d}\left[\sum_{t=0}^{\infty}\gamma^{t}R_{t+1}\right]
$$

你在读 policy gradient 或者 actor-critic 的论文的时候，一定会见到其中一个——要么是见这个，要么是见那个。所以希望大家见到的时候都明白：我们在这个课程当中都已经讲过了。这个式子实际上就是 $\bar{v}_\pi$。为什么？来看一下，其实非常简单——这会儿你可能会被蒙住，实际上很简单。

首先分析这里面的情况。这感觉好像是从一条 trajectory 得到的 reward 加起来，我假设这条 trajectory 从 $S_0$ 出发，$S_0$ 满足一个分布；然后从 $S_0$ 得到 $S_0, A_0, R_1, S_1, A_1,\dots$ 这样一条 trajectory。在每一个状态 $s$ 我怎么采取动作呢？根据 $\pi(s)$ 来采取；采取动作之后，得到的 $R_{t+1}$ 和 $S_{t+1}$ 都是由环境决定的。知道了这些之后，就可以把式子进一步拆开：根据 total expectation 的基本性质，因为可以从不同的 $s$ 出发，我写成——假如说从这个 $s$ 出发，从这出发的概率是 $d(s)$，从这出发得到的值是里面的那一项，把它们乘起来再相加，就等于原来的期望。

那里面那一项是什么？大家能看出来吗？从 $s$ 出发所得到的所有的 reward——一个 discounted return，这实际上就是一个 $G_t$。而这个是什么？这个就是 $v_\pi(s)$！所以整个式子等于 $\sum_s d(s)\,v_\pi(s)$，这等于什么？就是 $\bar{v}_\pi$。就这么简单。

### 目标函数的梯度怎么算

刚刚介绍了几个 metric，下面就对它们求 gradient，之后就可以把梯度应用到梯度上升的方法当中去进行优化，找到最优的策略。在给出 gradient 之前，我先强调一点：计算这些 metric 的 gradient，是整个 policy gradient 方法当中最复杂的一部分。为什么？复杂在两个地方：

- 第一个，我们有多个不同的 metric，像 $\bar{v}_\pi$、$\bar{r}_\pi$，还有 $\bar{v}_{\pi_0}$——$\bar{v}_{\pi_0}$ 指的是那里面的 $s$ 的 distribution 和 $\pi$ 没有关系，它是 $\bar{v}_\pi$ 的特殊情况；
- 第二个，我们要区分 discounted case 和 undiscounted case。

大家可以想一想：不同的目标函数、还有不同的情况，排列组合一下，每一种情况计算 gradient 都是不同的。虽然不同，但是大同小异。详细的细节课程里就不再介绍了，感兴趣可以到书里看——说实话这部分内容还是比较多的，但是比较复杂，所以这里做比较简要的介绍。

怎么个简要法呢？我只给出这样一个公式：

$$

abla_\theta J(\theta)=\sum_{s\in\mathcal{S}}\eta(s)\sum_{a\in\mathcal{A}}
abla_\theta\pi(a\mid s,\theta)\,q_\pi(s,a)
$$

$J$ 是目标函数，右边先对 $s$ 求和，每一个 $s$ 有一个权重（或者叫 probability distribution）$\eta(s)$；每一个 $s$ 又对应各个 action，再对 $a$ 求和，加的是两项的乘积：一边是策略的梯度 $
abla_\theta\pi(a|s,\theta)$，一边是 $q_\pi(s,a)$，也就是 $(s,a)$ 的 action value。

这个式子实际上是比较有意思的。首先 $J(\theta)$ 可以是 $\bar{v}_\pi$、$\bar{r}_\pi$ 还有 $\bar{v}_{\pi_0}$，这个等号实际上分三种情况：第一种是严格的相等，第二种是约等，第三种是成比例等于——中间差一个比例因子。还有一个：$\eta$ 对于不同的目标函数、在不同的情况下，会呈现出来不同的 distribution。具体对应过来就是：

| 目标函数 | 与公式的相等关系 | 权重分布 |
| :--- | :--- | :--- |
| $\bar{r}_\pi$ | discounted case 约等于，undiscounted case 严格等于 | $d_\pi$（stationary distribution） |
| $\bar{v}_\pi$ | 成比例等于（差一个比例因子） | $d_\pi$ |
| $\bar{v}_{\pi_0}$ | 严格等于（最简单的情况） | $\rho_\pi$（另外一个分布） |

总而言之，所有这些情况求出来的 gradient 是大同小异的，所以我就用这一个式子表示出来了。详细的推导下面我会稍微讲一下，但是详细的细节就不再讲了，感兴趣的同学可以到我的书里去看。我相信对绝大多数同学来说，这个式子就够了——除非你需要去研究、创新新的算法，那可以到书里看详细的内容。

### 化成期望形式：log 的技巧

上面这个梯度公式还有一个最重要的性质：它可以写成 expectation 的形式，也就是把所有的 $\sum$ 全都去掉：

$$

abla_\theta J(\theta)=\mathbb{E}_{S\sim\eta,\ A\sim\pi}\left[
abla_\theta\ln\pi(A\mid S,\theta)\,q_\pi(S,A)\right]
$$

这里边的 $(S,A)$ 是随机变量，$S$ 满足 $\eta$ 的分布，$A$ 满足 $\pi$ 这个分布。在讲怎么得到这个式子之前，我先讲一下为什么我们需要它——有的同学可能已经意识到了：这是一个真实的梯度，这个真实的梯度当中含有 expectation，而我不知道 expectation，所以就可以用采样来近似这个 expectation。这就是 stochastic gradient descent 或者 ascent 的基本思路。待会儿我就会用这个 stochastic gradient 来做优化——这就是这个式子重要的原因。

那怎么得到它？其实也比较简单。值得注意的是，这里边实际上有一个 $\ln\pi$。这个 $\ln\pi$ 是什么？是 $\pi$ 的自然对数——$\pi$ 是一个函数，我要在前面再加上一层函数，就是自然对数。自然对数也经常被写成 log；为了表示它的底数是自然对数，我写成 $\ln$ 的形式，后面我也会简称把它叫做 log。

$\log\pi$ 满足一个性质：

$$

abla_\theta\ln\pi(a\mid s,\theta)=\frac{
abla_\theta\pi(a\mid s,\theta)}{\pi(a\mid s,\theta)}
$$

为什么会有这个性质？比如 $\log x$ 对 $x$ 求导，它是什么呀？是 $1/x$。所以如果是 $\log f(x)$——这里 $f(x)$ 就对应了我们的 $\pi$——对它求梯度，首先会出来一个 $1/f(x)$，然后上面再乘上 $f(x)$ 的导数。这就是这个式子成立的原因。把它两边乘上 $\pi$、再左右调换一下，就得到了一个非常简洁的式子：

$$

abla_\theta\pi(a\mid s,\theta)=\pi(a\mid s,\theta)\,
abla_\theta\ln\pi(a\mid s,\theta)
$$

把这个式子代到刚才给出来的梯度表达式里边——刚才这里边是 $
abla_\theta\pi$，我们把它替换成 $\pi\,
abla_\theta\ln\pi$。为什么要做这样的替换？就是因为它能写成 expectation 的形式。我们看一下：首先把前面那个求和写成 expectation 的形式，$S$ 满足 $\eta$ 的分布；下面再把里面对 $a$ 的求和也写成 expectation 的形式，$A$ 属于 $\pi$ 这个分布。这样就得到了上面的式子。我们经常把下面的下标省略掉，直接写成：

$$

abla_\theta J(\theta)=\mathbb{E}\left[
abla_\theta\ln\pi(A\mid S,\theta)\,q_\pi(S,A)\right]
$$

#### softmax：让 log 有意义

介绍完梯度之后，做几个补充说明。第一个补充说明，我觉得有的同学可能已经意识到了：$\log\pi$ 要把 $\pi$ 放到 log 里边，$\pi$ 必须是大于 0 的。为什么？如果横坐标是 $x$、纵坐标是 $\log x$，那么它的曲线告诉我们 $x$ 必须大于 0，$x$ 等于 0 是不存在的。但是我们之前介绍过很多，比如 greedy 的 policy 或者 deterministic 的 policy，它们对某些 action 的 $\pi$ 值是等于 0 的——在这里边不能这样。

那我怎么样确保所有的 $\pi$ 对所有的 $a$ 全都大于 0 呢？这里就可以用一个 softmax function。softmax 的作用是什么？它实际上是在做归一化：把负无穷到正无穷上的一些数，归一化到 0 到 1 这个区间当中，而且它不会严格等于 0，也不会严格等于 1。具体来说，我现在有 $x_1$ 一直到 $x_n$ 这样一组数，把它们代到这样一个函数当中：

$$
z_i=\frac{e^{x_i}}{\sum_{j=1}^{n}e^{x_j}}
$$

这实际上就是一个函数 $z=f(x)$：$x$ 的取值可以从负无穷到正无穷任意，但这样计算之后，$z$ 一定是 0 到 1 上面的一个数，而且所有的 $z$ 加起来一定等于 1。

有了 softmax 函数，我们就可以写出 $\pi$ 的一个表达形式。反正 $\pi$ 都是一个函数，为了确保 $\pi$ 对每一个 $a$ 全都大于 0，我就可以强行让它满足这样的形式：

$$
\pi(a\mid s,\theta)=\frac{e^{h(s,a,\theta)}}{\sum_{a\in\mathcal{A}}e^{h(s,a,\theta)}}
$$

这里 $h$ 是另外一个函数。这类似于我们之前在介绍 value function approximation 当中的 feature vector——对应 $(s,a)$ 有一个 feature function。那怎么选取这个 feature function？如果是之前的话，选取起来还是比较麻烦的；现在全都是神经网络了。也就是说，假如我有一个神经网络，它的参数是 $\theta$、输入是 $s$，这个神经网络直接就可以把 $\pi(a_1|s,\theta)$ 一直到 $\pi(a_5|s,\theta)$ 全都输出出来（比如有 5 个的话），它的输出层用 softmax，自然就得到所有这些数都在 0 到 1 区间、并且相加等于 1。

因为这个 $\pi$ 对每一个 $a$ 都大于 0，所以这个策略是 stochastic 的，自然它也是探索性的。

> **提示**：下一节课我们会介绍 deterministic policy gradient 的方法，那时候就不需要这个条件了。它还有一个优势：这里我们输出的是每一个 action 所对应的 $\pi$，那如果 action 是无穷多个该怎么办？这时候 softmax 这种方法就不行了，但是 deterministic 的方法可以。

### 梯度上升与 REINFORCE

刚刚我们在第二节、第三节分别介绍了目标函数，还有目标函数所对应的梯度。下面来到第四部分：把梯度代入到梯度上升的算法当中去进行优化，并给出来第一个、也是最简单的 policy gradient 的算法——它有一个名字，叫做 REINFORCE。

梯度上升算法的基本思路就是：

$$
\theta_{t+1}=\theta_t+\alpha\,
abla_\theta J(\theta_t)
$$

$\theta$ 是参数，$\alpha$ 是步长，后面是目标函数的梯度——刚才我们已经计算出来了，就是那个 expectation 形式。但这个在实际当中是不能用的，为什么？因为这有一个 expectation，里面涉及到了状态它的分布。如果我们知道所有的信息的话，其实这个分布是能确定下来的；但是很可惜，比如环境的模型等等我们都是不知道的，这时候是无法计算这个 expectation 的。所以我们要用随机的梯度来代替真实的梯度：

$$
\theta_{t+1}=\theta_t+\alpha\,
abla_\theta\ln\pi(a_t\mid s_t,\theta_t)\,q_t(s_t,a_t)
$$

那这个式子其实也是不能用的，为什么？因为这里面有一个 $q_\pi$——策略 $\pi$ 所对应的真实的 action value。那怎么办呢？很简单：用一个方法来近似 $q_\pi$，或者对 $q_\pi$ 进行采样，把 $q_\pi$ 换成 $q_t$。怎么近似或者采样？有几种方法：

- 第一种方法是基于蒙特卡洛的方法，这也是最直观的：要估计 $q_\pi(s,a)$，就从 $(s,a)$ 出发得到一个 episode，然后计算这个 episode 的 return，假如说是 $g$，那这个 $g$ 实际上就是 $q_t(s,a)$，用它来近似 $q_\pi$。基于这个方法、并且和 policy gradient 算法相结合得到的算法，就叫做 REINFORCE，待会儿我会详细介绍。
- 除此之外还有其他方法，比如 TD 的方法等等。这些就引出了 actor-critic 一系列的算法，我们会在下一节课介绍。

所以 policy gradient 的算法其实到这里就介绍完了，思路还是非常简单的。下面补充几个非常重要的说明。

#### 怎么采样：on-policy 的由来

第一个补充说明是我们怎么样去做采样。真实的 gradient 里涉及到的随机变量是 $(S,A)$，那怎么对它采样？

先说怎么采 $S$。对于 $S$ 来说，它服从的分布是 $d$ 或者 $\eta$，这个分布代表了一种长期的情况——我跑了很久之后所得到的一种对 $S$ 的分布。但是在实际当中我们一般来说都不这么做，因为我们有数据就不错了，不会说要采很久、等它达到那种平稳的状态再去用这个数据。所以这一点在实际当中一般不太考虑，不用在意。

再说怎么采 $A$。理论上要求 $A$ 服从 $\pi$，所以我在 $s_t$ 应该根据当前的 $\pi$ 去采 $a$。因此 policy gradient 是 on-policy 的算法：它的 behavior policy 就是这个 $\pi$，target policy 也是这个 $\pi$——因为它在不断地改进，我改进完了之后就立刻要用它生成新的数据，然后再改进我自己。也有 off-policy 的情况，但是会需要额外的技巧，下节课我们会介绍。

#### 换个角度读这个算法：β 在平衡什么

第二个补充说明是关于我们怎么理解这个算法。有的同学可能会说：这不就是一个梯度上升的算法吗？它做的事情不就是在优化这个目标函数吗？这点是没错的，但是下面我们重新组织一下这个算法，以另外一种形式呈现出来，大家会发现有很多有意思的地方。

$\log\pi$ 的梯度等于 $\pi$ 的梯度除以 $\pi$，这个大家已经比较熟悉了。把之前算法里的 $
abla_\theta\ln\pi$ 换成右边这个，分母放到一边、$q_t$ 放到分子上，再把分子那一项单独拿出来，简写成一个 $\beta_t$：

$$
\theta_{t+1}=\theta_t+\alpha\,\beta_t\,
abla_\theta\pi(a_t\mid s_t,\theta_t),\qquad \beta_t=\frac{q_t(s_t,a_t)}{\pi(a_t\mid s_t,\theta_t)}
$$

$\alpha$ 和 $\beta_t$ 相乘作为一个新的步长。这个式子其实是非常有意思的——它在干嘛呢？其实它是在优化 $\pi(a_t|s_t)$。什么意思呢？假设我有一个目标函数 $f(\theta)$，希望 maximize 它，那算法不就是 $\theta_{t+1}=\theta_t+\text{步长}\times
abla f(\theta)$ 吗？所以这个算法实际上是要通过改变 $\theta$，去优化 $\pi(a_t|s_t)$ 的值——直观就很明显了。

当步长不大的时候，我们能够有一些结论。为什么优化的时候要强调步长不是很大？这也很简单：假如这里是一个极值，我从一边出发，沿着梯度确实能朝极值的方向前进；但如果步长太大的话，我可能过了，比如一下子跑到另一头去了，那显然不能保证上升。只有在比较小的范围内，才能确保沿着梯度确实能上升（或者下降）。所以我们假设 $\alpha\beta_t$ 是比较小的。

这时候直观上很容易理解：下一个时刻 $\pi(a_t|s_t,\theta_{t+1})$ 是大于 $\pi(a_t|s_t,\theta_t)$ 的——经过算法更新得到的 $\theta_{t+1}$ 会让这个 $\pi$ 的值增大，并且步长越大，增大的幅度也越大（当然步长不能太大，还是要在一定范围内）。相反，如果 $\beta_t$ 小于 0（刚才 $\beta_t$ 是大于 0 的），那它在干嘛？做梯度下降——得到的新的 $\theta$ 会让 $\pi(a_t|s_t)$ 的值减小。

为什么确实是这样？其实就把这种直觉转换成数学，很简单。由微分（一阶泰勒展开）：

$$
\pi(a_t\mid s_t,\theta_{t+1})\approx\pi(a_t\mid s_t,\theta_t)+
abla_\theta\pi(a_t\mid s_t,\theta_t)^T(\theta_{t+1}-\theta_t)
$$

为什么这个式子成立？它和 $\theta_t$、$\theta_{t+1}$ 具体取什么值没有任何关系，它成立是因为微分：当 $\theta_{t+1}-\theta_t$ 趋向于 0 的时候，这个近似就变成等式；两者离得越来越远，近似就越来越不精确。它对任意的、只要离得不太远的 $\theta_{t+1}-\theta_t$ 都成立。现在我有一个特殊的 $\theta_{t+1}$——就是算法给出的那个——把它代进去：把 $\theta_{t+1}-\theta_t$ 换成 $\alpha\beta_t
abla_\theta\pi$，梯度和梯度的转置相乘，就得到了这个梯度的模的平方：

$$
\pi(a_t\mid s_t,\theta_{t+1})\approx\pi(a_t\mid s_t,\theta_t)+\alpha\,\beta_t\,\big\|
abla_\theta\pi(a_t\mid s_t,\theta_t)\big\|^2
$$

这就很清晰了：这一项肯定是大于等于 0 的。所以当 $\beta_t>0$ 时，$\theta_{t+1}$ 时刻的 $\pi$ 值比 $\theta_t$ 时刻要大；当 $\beta_t<0$ 时肯定比它要小。这就是刚才那个直觉的数学表示，仅此而已。

说这些是为什么呢？因为它能更好地帮助我们理解这个算法在干嘛——这个 $\beta_t$ 能够很好地去 balance exploitation 和 exploration（一个是剥削、充分地利用，一个是探索）。为什么这么说？很有意思，我们来看：

- $\beta_t$ 的分子是 $q_t$，所以 $\beta_t$ 和 $q_t$ 成正比。当 $q_t$ 比较大的时候 $\beta_t$ 也比较大；$\beta_t$ 大意味着 $\pi(a_t|s_t)$ 这个值会被增大。这是什么意思？如果这个 action 之前它的 action value 是比较大的，那么我会给它更大的概率去选择它。这是在干嘛？这是在做 exploitation——我都已经知道那个 action 能给我更大的 action value 了，我干嘛不给它更大的概率去选择它呢？
- $\beta_t$ 的分母是 $\pi$，所以 $\beta_t$ 和 $\pi$ 成反比。很显然，当 $\pi(a_t|s_t)$ 比较小时 $\beta_t$ 比较大；大的 $\beta_t$ 又会得到更大的 $\pi(a_t|s_t)$（在 $\theta_{t+1}$ 的情况下）。这是什么意思？如果之前我选择 $a_t$ 的概率是比较小的，那么我会在下个时刻给它更大的概率去选择它。这是在干嘛？这是在做 exploration。

所以 $\beta_t$ 通过分子和分母上的这两个项，就很好地平衡了探索和充分利用——既做到了 exploration，又做到了 exploitation。

#### REINFORCE 的完整流程

介绍了算法的性质之后（还是非常有意思的），我们来看一个具体的算法。就是刚才给出来的那个算法，其中 $q_t$ 是用来近似 $q_\pi$ 的，而求 $q_t$ 有一些方法。如果我们用蒙特卡洛的方法——从 $(s_t,a_t)$ 出发得到一个 episode，把这个 episode 的 return $g_t$ 赋给 $q_t$——那么这个 policy gradient 的算法就被称为 REINFORCE。它有一个特殊的名称，这个 REINFORCE 就是 reinforcement learning 当中的那个 REINFORCE。REINFORCE 是最早的、也是最简单的一个 policy gradient 的算法，之后类似的算法还会被提出来，特别是下节课介绍 actor-critic 的时候。

它的基本流程是这样过的：在第 $k$ 个 iteration 当中，我们选定一个初始的 state，根据当前的策略——假设当前的参数是 $\theta_t$，所以当前的策略就是 $\pi(a|s,\theta_t)$——用它和环境进行交互，得到一个 episode。针对这个 episode 当中的每一个元素要过一遍，对每一个元素其实要操作两步：

```text
第 k 轮迭代：
  选定初始状态 s0
        │
        ▼
  用当前策略 π(a|s,θt) 与环境交互，采完一整条 episode
        │
        ▼
  逐个访问 episode 中的 (st, at)，每个做两步：
    ① value update：从 (st,at) 出发，把后面得到的
       所有 reward 相加 → 得到 qt
    ② policy update：把 qt 代进去，立刻更新 θt → θt+1
        │
        ▼
  episode 结束，把最后得到的 θ 作为新的 θk，
  用它去生成数据 → 回到开头，形成循环
```

读图说明：一轮迭代先采完一整条 episode，再对 episode 里每个 $(s_t,a_t)$ 先做值估计、再做策略更新，整条过完后的参数作为下一轮的起点。

有的同学可能已经注意到了：为什么 $\theta_t$ 更新成 $\theta_{t+1}$ 之后，$\theta_{t+1}$ 没有立刻去产生数据呢？就是因为这个方法是基于蒙特卡洛的方法，蒙特卡洛是 offline 的、离线的——你必须把所有的 episode 全部采集完毕之后才能开始跑，所以不能一边更新 $\theta_t$ 一边采集。之后我们介绍基于 TD 的方法的时候，就可以变成 online 的：我得到新的 $\theta_t$，就立刻可以用它去生成数据。

### 本讲小结

到这里，这次课的主要内容就介绍完了。其实这次课介绍的 policy gradient，基本的想法实际上就是三步：我要有一个目标函数；我求出目标函数的梯度；然后把它放到一个梯度上升的算法当中，对这个目标函数进行优化。所以它的思路是非常非常简单，简单得不能再简单了。但是你如果去看每一项——看它的目标函数、看它的 gradient、看它的算法——都会比较复杂。不过不管怎么样，希望大家能够牢牢地抓住它的基本思路：它就是一个优化问题。明白了这一点之后，你就不会轻易地迷失在里面了。今天介绍的 REINFORCE 是第一个 policy gradient 的算法，之后在下一节 actor-critic 当中，我们会介绍更多的算法。

### 下一步：Actor-Critic 与最简单的 QAC

欢迎来到强化学习课程的最后一站。这是我们的第 10 次课，介绍 actor-critic 的方法。经过长途的跋涉，我们终于来到了课程的最后一节——虽然是最后一节课，但是它的重要性却是非常高的。当然整个课程每一部分都很重要，但前面部分和后面部分的重要性体现在不同的方面：前面的部分（比如贝尔曼公式、贝尔曼最优公式等等）重要是因为它们非常基础，这些东西不理解清楚的话，学习后面会比较吃力，很多东西会一知半解；越到后边，重要性就体现在越来越接近前沿、越来越接近实践——比如今天介绍的 actor-critic，它实际上是现在强化学习当中最流行的方法之一。

那么 actor-critic 究竟是一个什么样的方法呢？其实它不是一个完全不一样的方法：它和我们上节课学习的 policy gradient 实际上是一种方法。那它为什么叫了另外一个名字？因为它把基于 value 的方法——特别是 value function approximation——引入到 policy gradient 当中，我们就得到了 actor-critic。具体怎么引入、细节是什么，今天来详细介绍。

#### actor 和 critic 分别是什么

actor 实际上对应的是策略：它是来更新策略的一个东西（或一个过程）。为什么称为 actor？因为策略是用来 take action、采取行动的，take action 的那个东西就可以对应着 actor——演员。

critic 的字面意思是评论家或者批评家。它在这里实际上是做 policy evaluation 的：我有一个策略，要评估一下这个策略到底好还是不好。怎么评估？实际上就是根据 value estimation——估计它的 action value 或者 state value。

这节课的路线是：首先介绍一个非常简单的 actor-critic 算法（actor-critic 简称 AC），这个 AC 算法虽然很简单，但是对揭示整个 AC 算法的思想起到了非常关键的作用；在这个基础之上，进一步介绍 advantage actor-critic，简称 A2C——它的思想是通过引入一个偏置量来减少估计的方差；然后再进一步，学习怎么把一个 on-policy 的 AC 方法转换成 off-policy 的 AC 方法，这里会用到非常重要的技术，叫做 importance sampling（重要性采样）。这三节实际上是一环扣一环，后面的是前面的推广。它们还有一个共同特点：和上节课一样，policy 全都是 stochastic 的。那能不能把 deterministic 的 policy 引进来呢？实际上是可以的——在最后一节，我们会介绍 deterministic actor-critic 的方法。

#### 回顾：actor 和 critic 在公式里各站在哪

先回顾上节课的 policy gradient 方法——因为在最开始已经提过，actor-critic 本身就是 policy gradient。它的基本思路三步：第一，有一个目标函数，可以是 $\bar{v}_\pi$ 也可以是 $\bar{r}_\pi$，分别对应 average value 和 average reward；第二，有了目标函数就用最简单的梯度方法去优化——梯度上升，$\theta_{t+1}=\theta_t+\alpha\times$ 梯度；第三，把这样一个含有 expectation 的算法，通过采样转化成一个不含有 expectation 的算法，也就是 stochastic gradient ascent：

$$
\theta_{t+1}=\theta_t+\alpha\,
abla_\theta\ln\pi(a_t\mid s_t,\theta_t)\,q_t(s_t,a_t)
$$

这个算法的表达式其实是非常重要的。为什么？通过这个表达式，我们就会知道哪些是 actor、哪些是 critic，actor 和 critic 究竟是怎么样结合起来的。非常直观：

- 哪个是 actor？这个算法就是 actor。这个算法在干嘛？它在更新策略——$\theta$ 是策略的参数，$\theta$ 变化了，策略自然也就变化了。所以 actor 是更新策略的这样一个算法。
- critic 是什么？critic 就是来估计这个 $q_t$ 的这样一个算法。因为要更新策略需要 $q_t$，而 $q_t$ 是 $q_\pi$ 的近似，所以 critic 的作用就是对当前的策略进行评估，计算出来它所对应的 action value。

通过这样一个式子，我们就非常清晰地看到了 actor 和 critic、policy based 的方法和 value based 的方法，是怎么样结合起来的。

那 $q_t$ 怎么计算？到目前为止，估计一个 action value 我们介绍了两种方法。第一种是蒙特卡洛：要计算 $q_t(s_t,a_t)$，就从后面得到一个 episode，计算那个 episode 所对应的 return，让那个 return 作为 $q_t(s_t,a_t)$ 的近似值。如果 $q_t$ 是这么计算的，再配合上刚才那个算法，整个算法就被称为 REINFORCE，或者叫蒙特卡洛 policy gradient——这是上节课已经介绍过的。这节课重点介绍的是：用 temporal-difference learning（TD 方法）来估计这个 $q_t$，这时候算法一般就被称为 actor-critic。

#### QAC：Sarsa 加值函数近似

做好准备了，来看第一个 actor-critic 的算法。它的目标就是优化目标函数 $J(\theta)$。怎么做？在第 $t$ 步：假设当前状态是 $s_t$，当前策略对应 $\theta_t$，根据这个策略生成一个 action $a_t$ 和环境交互，得到 $r_{t+1}$ 和 $s_{t+1}$；到了 $s_{t+1}$，再根据在那的策略得到一个新的 $a_{t+1}$。于是我就有了 $s_t, a_t, r_{t+1}, s_{t+1}, a_{t+1}$ 这 5 个量。

当大家看到这 5 个量的时候，可能就会反应过来——这就是 Sarsa 的算法！有了这 5 个量组成的 experience sample，就可以放到一个算法当中去估计 action value。所以这里的 critic（括号里写的就是 value update）对应的就是 Sarsa 算法，并且结合上了 value function approximation——Sarsa 对应一个函数，它的参数是 $w$，$w$ 被更新之后就得到了一个新的 $\widehat{q}$；我们把这个 $\widehat{q}$ 代到 actor 里边（就是前面介绍的那个 policy update 算法），得到一个新的策略；这个新的策略之后就会用到下一步当中去生成新的数据。整个结构如下：

```text
            (st, at, rt+1, st+1, at+1)   ← 一次交互的 experience sample
                        │
        ┌───────────────┴───────────────┐
        ▼                               ▼
  critic（value update）          actor（policy update）
  Sarsa + 值函数近似，参数 w       θt+1 = θt + α·∇lnπ(at|st,θt)·q̂t
  估计出 q̂(st,at)                        │
        └──────────► q̂ ─────────────────┘
                                        ▼
                            新策略 π(a|s,θt+1)
                                        │
                                        ▼
                          生成下一步的交互数据（循环）
```

读图说明：critic 负责用 Sarsa 加值函数近似估计出 $\widehat{q}$，actor 拿这个 $\widehat{q}$ 去更新策略参数 $\theta$，新策略再回头生成下一份数据。

对这个算法再做几点简要的补充说明。首先，critic 对应的就是 Sarsa 加上 value function approximation，actor 就是对应那个 policy update 的算法。值得指出的是，这个算法是 on-policy 的。为什么 policy gradient 的方法、包括这个 actor-critic，是 on-policy 的？告诉大家为什么：因为它的梯度 $J(\theta)$ 等于 $\mathbb{E}_{S\sim\eta,\,A\sim\pi}[\cdots]$——后边有一串东西我省略不写了。在随机梯度上升的方法当中，因为有 expectation，要对它进行随机采样，采样的时候肯定要符合这些分布，所以 $A$ 一定要按照 $\pi$ 的分布进行采样——$\pi$ 就是我们的 behavior policy；$\pi$ 又是我们不断改进的策略，所以它也是 target policy。两个 policy 是相同的，因此 policy gradient 包括刚才介绍的 actor-critic 都是 on-policy 的。

那有的同学可能会说：这个 on-policy，我要不要把策略做一些处理，比如把它变成 $\varepsilon$-greedy、让它变成随机策略？实际上是不用的。因为在 policy gradient 的方法当中，这个策略它已经是 stochastic 的——大家还记得吗？$\pi(a|s,\theta)$ 必须大于 0、不能等于 0——所以它本身就具有一定的探索能力，因此不需要再把它转成 $\varepsilon$-greedy 这种策略了。

刚才这个算法实际上是最简单的 AC 方法之一，有时候会被称为 QAC——这个 Q 实际上就对应的 q value，也就是 action value。虽然这个算法非常简单，但是它揭示了 actor-critic 非常本质的想法是什么。接下来我们会在它的基础之上做进一步的推广，得到其他不同的 AC 方法。

> **承前启后**：上一节讲完「策略梯度方法：目标函数、梯度与 REINFORCE，兼看最简单的 Actor-Critic」，下一节接着讲「Actor-Critic 进阶：A2C、重要性采样与确定性策略梯度」。

---

## Actor-Critic 进阶：A2C、重要性采样与确定性策略梯度
> 对应块：BLK13 | 覆盖分集：P51-P54 | 块标题（分集名）：《第10课-Actor-Critic方法（Adva》

上一讲的 QAC 把 actor 和 critic 拼在了一起，这一讲接着往前走三步：先给 QAC 引入一个 baseline（偏置量），得到 A2C；再用重要性采样把 on-policy 的 actor-critic 转成 off-policy 的版本；最后把随机策略换成确定性策略，得到 deterministic actor-critic（DPG）。每一步的动机都很朴素：baseline 是为了降低估计的方差，重要性采样是为了复用旧经验，确定性策略是为了应对动作个数无限的情况。

### 减掉一个 baseline，梯度为什么不变

A2C 的基本思想就一句话：在 QAC 当中引入一个新的 baseline，也就是偏置量，来减少估计的方差。要理解它，先要弄清一个性质——我们所推导的 policy gradient，对引入一个新的偏置是不敏感的。

回顾上一讲的梯度：

$$
abla_\theta J(\theta) = \mathbb{E}_{s\sim d,\, a\sim\pi}\Big[
abla_\theta \log \pi(a|s,\theta)\, q_\pi(s,a)\Big]$$

如果在这个 $q_\pi$ 后边减去一个新的 baseline $b$，得到的式子和原来完全相同——不管 $b$ 是什么样的函数都相同。这里 $b$ 是状态 $s$ 的函数，是一个标量函数。

为什么成立？把期望按定义展开：先对 $s$ 求和（每个 $s$ 遵从它的分布），再对 $a$ 求和（遵从 $\pi(a|s,\theta)$），多出来的那一项就是：

$$\sum_s d(s) \sum_a \pi(a|s,\theta)\, b(s)\, 
abla_\theta \log \pi(a|s,\theta)$$

关键一步在这里：$\log\pi$ 求梯度实际上就是 $\pi$ 求梯度再除以 $\pi$，分子分母的 $\pi$ 一抵消，剩下的就是 $\pi$ 本身的梯度。而 $b(s)$ 只是 $s$ 的函数，和 $a$ 没有关系，所以它可以作为一个公因子提到求和外面去——**也正是因为如此才能提到前面**。如果 $b$ 是 $a$ 的函数，那就提不出来，引入它就会改变期望。

提到外面之后，里面对所有 $a$ 的概率求和，概率和自然是 1；对 1 求梯度，任何常数的导数都是 0。所以整项为零，baseline 不影响梯度。

> **易错点**：baseline 必须只是 $s$ 的函数，绝不能是 $a$ 的函数。一旦 $b$ 依赖 $a$，它提不出对 $a$ 的求和，梯度就真的被改变了。

### baseline 不改变期望，却实实在在改变方差

那为什么要关心这个 $b$？它究竟有什么用？答案藏在方差里。

把梯度括号里那长串记成一个随机变量 $X$，梯度就是 $\mathbb{E}[X]$。我们已经知道 $\mathbb{E}[X]$ 和 $b(S)$ 没有任何关系——不论取什么样的函数，期望总是保持不变。但是 $X$ 的方差（variance）可是和 $b$ 有关系的：取不同的 $b$，方差可能差得很大。

这里有个技术细节：如果 $X$ 是标量，方差很简单；如果 $X$ 是向量，它的方差是一个矩阵，我们通常用矩阵的 trace（对角线元素之和）作为评价方差大小的工具。它可以算出来等于 $\mathbb{E}[X^TX] - \bar{x}^T\bar{x}$，其中 $\bar{x} = \mathbb{E}[X]$。前一项和 $b$ 无关，后一项里的期望虽不变，但整体的方差表达式展开后含有这样一项：

$$\mathbb{E}\Big[\|
abla_\theta \log \pi(a|s,\theta)\|^2 \,\big(q_\pi(s,a) - b(s)\big)^2\Big]$$

直观想一想：如果 $b$ 等于 0，或者 $b$ 等于 100 万、1 亿，$\big(q-b\big)^2$ 这一项的值显然天差地别。所以结论很清楚：**baseline 对方差是有影响的**。

于是目标就出来了：既然 baseline 影响方差，那就要找一个最好的 baseline，让方差达到最小。为什么要让方差最小？因为采样时误差更小。想象两个随机变量：一个均值是那条线、方差非常小，随便采一个样本都紧贴着 $\mathbb{E}[X]$；另一个均值同样是那条线，但方差非常大，采到的 $x$ 和真实的 $\mathbb{E}[X]$ 可能差得很远。显然我们希望 $X$ 的方差越小越好。

之前介绍的 REINFORCE 和刚介绍的 QAC，当中都没有 baseline——从数学上来说，就是 baseline $b=0$。但没人说过 $b=0$ 是一个好的 baseline，实际上它也不是。

> **结论**：减 baseline 是"免费的午餐"——期望一点不变，方差却可以大幅下降，采样自然更准。

### 最优 baseline 长什么样，为什么我们不用它

最好的 baseline 有一个表达式：对任何一个状态 $s$，它对应的

$$b^*(s) = \frac{\mathbb{E}\big[\|
abla_\theta\log\pi\|^2\, q_\pi(s,a)\big]}{\mathbb{E}\big[\|
abla_\theta\log\pi\|^2\big]}$$

上面一个期望、下面一个期望，蓝色的 $\|
abla_\theta\log\pi\|^2$ 可以看成一个权重，红色的是对应的 action value $q_\pi(s,a)$。在这个 $b^*(s)$ 下，$X$ 的方差最小。

但是这个式子虽然最优，却太复杂了，所以我们平常不用它。常用的做法是把权重直接去掉，只求 $q$ 的期望：

$$b(s) = \mathbb{E}_{a\sim\pi}\big[q_\pi(s,a)\big] = v_\pi(s)$$

这不就是 $v_\pi(s)$ 吗——$s$ 所对应的 state value。

> **易错点**：一些文献当中有人说这个 $v_\pi(s)$ 是最优的 baseline。实际上它不是最优的，只不过我们经常用它而已——真正的最优是上面带权重的那个，太复杂了，才退而求其次。

### 优势函数：A2C 的核心

现在把 baseline 引入 actor-critic 算法，看看会得到什么有意思的结论。做法很简单：把 baseline 设为 $v_\pi(s)$，也就是 state value。于是梯度里的 $q_\pi - v_\pi$ 被写成一个新函数 $\delta_\pi$：

$$\delta_\pi = q_\pi(s,a) - v_\pi(s)$$

它有一个名字，叫 advantage function（优势函数），常用的符号是 $A$——但因为这里字母 $A$ 已经被 action 占了，所以用 $\delta$ 来表示。

为什么叫"优势函数"？回忆 $v_\pi$ 的定义：它是 $\pi(a|s)$ 遍历所有的 $a$，把它们对应的 $q_\pi$ 求平均。所以 $v_\pi$ 是 $q_\pi$ 在某一个状态下的平均值。那如果某个 action 对应的 $q_\pi$ 比这个平均值大，就说明这个 action 比较好、有一定的优势——这就是名字的由来。

有了期望形式的梯度，很自然地取随机版本（通过采样得到），把 $\log\pi$ 的梯度写成 $\pi$ 的梯度除以 $\pi$，整理成分数形式：

$$\theta_{t+1} = \theta_t + \alpha_t\, \frac{\delta_t}{\pi(a_t|s_t,\theta_t)}\, 
abla_\theta \pi(a_t|s_t,\theta_t)$$

大括号里的分子是 $\delta$（优势函数），分母是 $\pi$。

#### 分子分母如何平衡探索与利用

这个分子分母的结构上一讲已经详细分析过，这里再稍微回顾一下。当 $\delta_t$ 比较大的时候，step size 就比较大——现在这个算法在更新 $\pi(a_t|s_t)$，步子迈得大，下一个时刻 $\pi(a_t|s_t)$ 就会变得比较大。意思就是：这个时刻我已经知道这个 action 有比较大的 action value，下个时刻就要给它更多的概率去选择它。这是充分利用。

反过来，如果分母上的 $\pi(a_t|s_t)$ 本身比较小，step size 也会被推得比较大，下一时刻的概率就会增大。也就是说：上一个时刻我选这个 action 的概率很小，那么下个时刻就多给它一点机会——这是增加探索。

一个分子、一个分母，就这样达到了比较好的平衡。

#### 我们在乎的是相对值，不是绝对值

那 $q_t$ 好还是 $\delta_t$ 好？显然是 $\delta_t$ 好。为什么？因为我们在乎的不是 action value 的绝对值，而是它的相对值。比如一个 action 的 action value 是 1 亿，但那个状态下其它 action 的 value 都是 2 亿，我还是不会觉得它是好 action；相反，一个 action 的 value 是 0，但其它都是 $-1$，那 0 就是一个好 action。

> **结论**：$q$ 减去 $v$ 之后，衡量的是"比平均好多少"，这正是选动作真正该看的东西。

#### 最后一步替换：用 TD error 代替优势函数

还有一个技巧。刚才定义的 $\delta$ 是 $q - v$，其中 $q_\pi, v_\pi$ 已经换成 $q_t, v_t$ 了。现在把 $q_t$ 替换成 $r_{t+1} + \gamma v_t(s_{t+1}) - v_t(s_t)$：

$$\delta_t = r_{t+1} + \gamma v_t(s_{t+1}) - v_t(s_t)$$

这就是 TD error。为什么这个替换是合理的？从期望的角度来说，对 $q_\pi - v_\pi$ 求期望，就等于对 $R + \gamma v_\pi(S') - v_\pi(S)$ 求期望，两者是一回事。

这么做的好处是什么呢？如果直接用 $q - v$，我需要一个函数或神经网络去近似 $q_t$，还需要另一个函数和神经网络去近似 $v_t$——两个网络。现在只需要一个神经网络，近似 $v_t$ 就可以了。

到这里，A2C 就得到了。它也被称为 TD actor-critic，因为其中用 TD error 来估计 value function。

#### A2C 算法怎么跑起来

简要过一遍流程：

```text
π(a|st,θt) ──采样──> at ──与环境交互──> (rt+1, st+1)
                                            │
                                            ▼
              TD error: δt = rt+1 + γ·vt(st+1) − vt(st)
                                            │
                  ┌─────────────────────────┴─────────────────────────┐
                  ▼                                                   ▼
          critic：用 δt 更新 vt                              actor：用同一个 δt 更新 θt
        （TD 算法 + 值函数近似）                          （一步梯度上升）
                  │                                                   │
                  └──────────────> 新数据进入下一轮循环 <───────────────┘
```

读图：在第 $t$ 个时刻，策略 $\pi(a|s_t,\theta_t)$ 产生一个动作 $a_t$，与环境交互得到 $r_{t+1}$ 和 $s_{t+1}$；用这一步数据计算 TD error $\delta_t$，它先送进 critic（所谓 critic 就是 value update，即之前介绍的 TD 算法和值函数近似相结合的那一套），然后同一个 $\delta_t$ 复用到 actor 里，用于 policy update。更新之后的数据再进入下一个循环。

这个算法是 on-policy 的：因为 $\pi$ 在这里是 stochastic 的，采样本身就有探索性，我们不需要再用 $\varepsilon$-greedy 这类方法。

### 重要性采样：用别的分布采的样，估计这个分布的期望

到目前为止，AC 方法和 policy gradient 方法都是 on-policy 的。本质上原因就是：它的梯度是一个 expectation，这个 expectation 当中的 $a$ 要服从 $\pi$ 这个策略。所以采样的时候 $a$ 必须服从 $\pi$——$\pi$ 是产生数据的 behavior 策略，同时 $\pi$ 也是我们要更新的策略，即 target 策略。两个策略相同，所以是 on-policy。

能不能转成 off-policy？动机刚才说过了：如果之前已经有一些经验了（通过其它方法得到的），我们想复用这些经验，该怎么办？答案是可以的，用 importance sampling（重要性采样）这个技术。

这里要强调一点：虽然 importance sampling 是在这里介绍的，但并不是说它只应用于 AC 方法。实际上任何你要求 expectation 的算法，都可以用到 importance sampling——包括前面的蒙特卡洛方法、TD 方法，都是可以的。

#### 两个分布的直观例子

考虑一个最简单的情况。随机变量 $X$ 的取值集合只有两个元素：$+1$ 和 $-1$。

**第一种情况**：$X$ 服从分布 $p_0$，即 $X=+1$ 的概率是 0.5，$X=-1$ 的概率也是 0.5。它的期望：

$$\mathbb{E}_{p_0}[X] = (+1)\times 0.5 + (-1)\times 0.5 = 0$$

问题来了：如果不知道 $p_0$ 的具体表达式，或者 $p_0$ 很复杂、求不出 $\mathbb{E}[X]$，能不能通过采样来求？

这就是我们在蒙特卡洛那一节熟悉的老路：我有一些根据 $p_0$（真实分布）采样得到的样本，每一个样本的期望和 $\mathbb{E}[X]$ 相等、方差和 $X$ 的方差也相等，于是用平均值 $\bar{x}$ 来近似期望。当样本数越来越多，近似越来越精确——因为 $\bar{x}$ 是 $\mathbb{E}[X]$ 的无偏估计，同时 $\bar{x}$ 的方差是 $X$ 方差的 $1/n$，$n$ 趋向无穷时方差趋向 0。这背后就是大数定理。取 200 个随机采样，$+1$ 差不多 100 个、$-1$ 差不多 100 个，求平均：最开始不太精确，最后平均值会越来越接近真实的期望 0。

**第二种情况**是新的：我手里有一些样本，但它们不是在 $p_0$ 下采的，而是在另一个分布 $p_1$ 下采的——$X=+1$ 的概率是 0.8，$X=-1$ 的概率是 0.2。在这个分布下期望显然是：

$$\mathbb{E}_{p_1}[X] = (+1)\times 0.8 + (-1)\times 0.2 = 0.6$$

而 $p_0$ 下它是 0，两者不相等。因为样本服从 $p_1$，直接求平均得到的 $\bar{x}$ 最后一定趋向 0.6，不会趋向 0。

所以问题就变成了：**我有一个概率分布 $p_1$，在它下面产生了一些样本，我想用这些样本来估计 $p_0$ 下的期望。** 为什么要做这件事？因为我们想做 off-policy：off-policy 里有一个 behavior policy $\beta$ 产生数据——$\beta$ 就对应 $p_1$；还有一个 target policy $\pi$，我要估计 $\pi$ 分布下的期望——$\pi$ 就对应 $p_0$。直接对样本求平均，最后一定到 0.6 而不是 0，所以要用其它的方法，就是重要性采样。

先看一个直观的图景：$p_1$ 下取 $+1$ 的概率是 0.8、取 $-1$ 是 0.2，做 200 次采样，$+1$ 的样本非常非常多、$-1$ 的比较少。对这些样本直接求平均，得到的那条虚线最后趋向 0.6；而同样用这批数据，套上重要性采样技术之后，那条红色的线就会趋向 $p_0$ 对应的期望，也就是 0。

#### 核心推导：一个加权平均

重要性采样最核心的式子是下面这个。我们要求的目标是 $X$ 在 $p_0$ 分布下的期望，根据定义展开，再在分子分母上同乘同除一个 $p_1(x)$：

$$\mathbb{E}_{p_0}[X] = \sum_x p_0(x)\, x = \sum_x p_1(x)\, \frac{p_0(x)}{p_1(x)}\, x = \mathbb{E}_{p_1}\big[f(X)\big], \quad f(x) \triangleq \frac{p_0(x)}{p_1(x)}\, x$$

中间乘上 $p_1(x)$ 除以 $p_1(x)$，两者抵消，式子仍然相等，只不过把 $\frac{p_0(x)}{p_1(x)}x$ 整体看成一个函数 $f(x)$，整个期望就变成了"$f(X)$ 在 $p_1$ 下的期望"。

这个式子有什么用？我们的样本就是服从 $p_1$ 的，而右边期望里的 $X$ 也服从 $p_1$——分布对上了，就可以用样本来近似这个期望。这就是 importance sampling 最最基本的核心想法。

具体做法非常简单：得到一批服从 $p_1$ 的样本 $x_i$，求出 $f(x_i)$，再求和求平均：

$$\bar{f} = \frac{1}{n}\sum_{i=1}^{n} f(x_i) = \frac{1}{n}\sum_{i=1}^{n} \frac{p_0(x_i)}{p_1(x_i)}\, x_i$$

$\bar{f}$ 的期望等于 $\mathbb{E}_{p_1}[f(X)]$，它的方差是 $f(X)$ 方差的 $1/n$——又是大数定理。于是：$\bar{f}$ 近似 $\mathbb{E}_{p_1}[f(X)]$，而后者又等于我们最初要求的 $\mathbb{E}_{p_0}[X]$。

注意这个式子是一个**加权的平均数**：还是对 $x_i$ 求平均，但每个样本带一个权重 $\frac{p_0(x_i)}{p_1(x_i)}$，这个权重被称为 importance weight。

- 如果 $p_1 = p_0$，权重一除就是 1，$\bar{f}$ 和 $\bar{x}$ 就一样了——两个分布完全相同，在哪个下采样没有区别。
- 当 $p_0(x_i) > p_1(x_i)$ 时，权重大于 1。为什么这是合理的？给一个直观解释：我得到一个 $x_i$，发现 $p_0(x_i)$ 比较大，说明在 $p_0$ 下采到它的概率比较大；而 $p_1(x_i)$ 比较小，说明在 $p_1$ 下很难采到它。可我现在要算的是 $p_0$ 下面的期望，所以要很**珍惜**这个 $x_i$ 才对——它在 $p_1$ 下很难采到，在 $p_0$ 下却很容易采到，就给它比较大的权重，通过这样才能把结果"拉回来"。

#### "都知道 p₀ 了，为什么不直接求和？"——初学者必问的问题

有同学眼尖的话，可能已经发现一个问题：计算 $\bar{f}$ 的时候，分子是 $p_0(x_i)$，也就是说我知道 $x_i$ 的分布。那有人会说：我知道 $p_0$ 的话，直接对所有的 $p_0(x)\,x$ 一求和，这不就是期望的定义吗？还在这儿麻烦什么呢？

这个问题初学者经常问，答案其实很简单：**如果你能那么做，你就不需要用 importance sampling；你用 importance sampling，正是因为你不能这么做。** 也就是说，我给一个 $x$，能够算出 $p_0(x)$ 的值，但是我不能够去求它的期望。

为什么知道 $p_0(x)$ 了还不能求期望？举几种情况就明白了。首先，这里考虑的是离散情况，但也可能是连续的——连续时就不能求和，要求积分，而求积分对 $p_0$ 的表达式有比较高的要求，表达式很复杂的话可能算不出来。更重要的情况是：$p_0$ 是用**神经网络**来表示的——给我一个 $x$，我能输出一个 $p_0(x)$ 的数值，但 $p_0$ 根本就没有表达式。没有表达式，你怎么去求积分呢？

为什么会出现用神经网络代表一个分布的情况？回到强化学习里：我们不是有 $\pi(a|s,\theta)$ 吗？$\theta$ 就是那个神经网络的参数——给我 $s$ 和 $a$，通过网络我能告诉你对应的概率是多少，但它没有解析表达式。

#### 小结：两条线，各归各的期望

现在手上有同一批服从 $p_1$ 的样本，和两个统计量：

| 统计量 | 计算方式 | 收敛到 | 对应图中 |
| :--- | :--- | :--- | :--- |
| $\bar{x}$ | 直接对样本求平均 | $\mathbb{E}_{p_1}[X] = 0.6$ | 蓝色线 |
| $\bar{f}$ | 带 importance weight 的加权平均 | $\mathbb{E}_{p_0}[X] = 0$ | 红色线 |

道理很直白：你在哪个分布下采样，直接求平均最后一定得到那个分布下的期望；想得到另一个分布的期望，就得靠 importance weight 加权。

### Off-policy actor-critic

有了重要性采样，就把它应用到 policy gradient 当中，实现 off-policy 的学习。套路还是两步：先得到梯度的表达式，再把它应用到梯度上升方法中进行优化。

假设 $\beta$ 是一个 behavior policy，用来生成很多经验采样；我们的目标是优化 $j(\theta)$，$\theta$ 是 target policy $\pi$ 的参数：

$$j(\theta) = \sum_s d_\beta(s)\, v_\pi(s)$$

其中 $d_\beta(s)$ 是策略 $\beta$ 下面的 stationary distribution，$v_\pi(s)$ 是策略 $\pi$ 下面对应的 state value。这个目标函数对应的梯度是：

$$
abla_\theta j(\theta) = \mathbb{E}_{s\sim\rho,\, a\sim\beta}\left[\frac{\pi(a|s,\theta)}{\beta(a|s)}\, 
abla_\theta \log\pi(a|s,\theta)\, q_\pi(s,a)\right]$$

这里有一个很重要的区别：之前 on-policy 的时候，$a$ 是服从 $\pi$ 的；现在 $a$ 服从 $\beta$。括号里面这一部分和 on-policy 情况一样，只不过前面多了一个权重 $\frac{\pi}{\beta}$——这个权重就是重要性采样。对应关系一目了然：$\pi$ 就是前面的 $p_0$（要估计它分布下的期望），$\beta$ 就是 $p_1$（样本是按它采的）。

#### baseline 照样能加，TD error 照样能换

有了梯度的表达式，立刻应用到梯度上升。首先说明：这里仍然可以加上一个 baseline 而不改变梯度，常见做法就是把 baseline 设为 $v_\pi(S)$。true gradient 里的 expectation 求不出来，就用 stochastic gradient 代替。和 A2C 当中一样，$q_t - v_t$ 用 TD error $r_{t+1} + \gamma v_t(s_{t+1}) - v_t(s_t)$ 代替，它也可以理解成 advantage function。

再把算法变形看看它在干什么：$\log\pi$ 求梯度写成 $\pi$ 的梯度除以 $\pi$，和外面的 $\frac{\pi}{\beta}$ 一约，剩下分子上是 $\delta$、分母上是 $\beta$，再乘 $\pi$ 的梯度（这一项对应 step size）：

$$\theta_{t+1} = \theta_t + \alpha_t\, \frac{\delta_t}{\beta(a_t|s_t)}\, 
abla_\theta \pi(a_t|s_t,\theta_t)$$

分析方法和之前一样：$\delta$ 比较大的时候，下一个时刻就增大 $\pi(a_t|s_t)$，使得选择这个 action 的概率大一些——这是充分利用。但注意：之前分母上是 $\pi$，当 $\pi(a_t|s_t)$ 比较小时下一时刻它会增大，从而实现探索；现在分母上不是 $\pi$ 而是 $\beta$，它是一个固定的值，**所以这里面就没有什么探索，它就是充分的利用**——分母是不可变的。

对应的伪代码和前面的 A2C 一模一样，区别只有两处：多了一个 behavior policy $\beta$，产生经验采样时要根据 $\beta$ 进行产生；后边的式子里多了两个 importance weight。其余完全类似，不再展开。

### 确定性策略梯度（DPG）

QAC 引入了 actor-critic 的思想，A2C 引入 baseline 来减少估计的方差，off-policy actor-critic 引入了重要性采样——它们的共同特点是：涉及的策略 $\pi(a|s,\theta)$ 全都是严格大于 0 的，不会等于 0，也就是说策略一定是 stochastic 的。那么问题来了：能不能用 deterministic 的策略？答案是可以的。

#### 为什么要关心确定性策略

画一个图就明白了。之前的策略 $\pi$ 以 $s$ 为输入，输出 $\pi(a_1|s,\theta)$ 一直到 $\pi(a_5|s,\theta)$（比如有五个 action），输出层约束每一个输出全都是严格大于 0 的、并且它们的和等于 1。

这种表达方式的缺点是：**它对 action 的个数是有要求的，只能是有限个**——网络不可能输出无限个数。如果我在一个状态 $s$ 下有无限个 action，该怎么办？这时候就要用 deterministic 的情况。

#### 从"输出概率"到"直接输出动作"

之前所有的策略都用 $\pi(a|s,\theta)$ 表示：在状态 $s$ 下取 action $a$ 的概率是多少，是一个 0 到 1 之间的数值。这种写法既能表示 stochastic 策略，也能表示 deterministic 策略（deterministic 时概率就等于 1）。现在做一个变化——既然要求它是 deterministic 的，就直接写成：

$$a = \mu(s, \theta)$$

$\mu$ 是一个函数，输入是 $s$，参数是 $\theta$，输出直接就是 $a$。注意：$\mu$ 不是输出"在 $s$ 采取 action $a$ 的概率是多少"，它的输出就是 $a$ 本身——比如直接告诉你动作的索引是向上还是向下。所以 $\mu$ 是从状态空间到动作空间的一个映射：状态空间的点确定时，动作空间的点也就确定下来了。实际当中它就可以用一个神经网络实现，参数是 $\theta$，输入 $s$，输出直接是 $a$。有时候我们会把 $\mu(s,\theta)$ 简写成 $\mu(s)$，把 $\theta$ 隐藏掉，但要知道 $\theta$ 仍然是它的参数。

#### 目标函数：两种特殊的 d₀

还是老套路，分两步：先介绍梯度怎么计算，再把梯度用到梯度上升方法中进行优化——这和前面的算法一模一样。梯度优化的方法在强化学习当中使用非常广泛，了解了这个思路之后，学习很多算法都会非常清晰。

之前的梯度是针对 stochastic policy 的；现在强行要求 policy 是 deterministic 的，就要推导这种情况下的 policy gradient。目标函数定义为：

$$J(\theta) = \sum_s d_0(s)\, v_\mu(s)$$

$v_\mu(s)$ 是策略 $\mu$ 下面的 state value，$d_0$ 是一个概率分布——每一项都大于等于 0、和等于 1。这里 $d_0$ 比较简单的选取方式是让它和 $\mu$ 没有关系。有两种重要而特殊的情况：

- **只关心某一个出发点**。比如有一个任务，每次开始都从状态 $s_0$ 出发，其它状态我都无所谓，只要最大化从这个状态出发的 return。那 $d_0(s_0)=1$、$d_0(s \neq s_0)=0$，目标函数就成了 $v_\mu(s_0)$。
- **$d_0$ 取某个 behavior policy 的 stationary distribution**。比如取另一个 behavior policy $\beta$ 下面的 stationary distribution。这和 off-policy 有关系——待会儿就会发现，deterministic policy gradient 天然就是 off-policy 的，不需要用 importance sampling 把它转成 off-policy，它自己就是。

#### 梯度长什么样：连 action 都不见了

在 discounted 情况下，$J(\theta)$ 的梯度等于：

$$
abla_\theta J(\theta) = \mathbb{E}_{s\sim\rho^\mu}\Big[
abla_\theta \mu(s,\theta)\; 
abla_a q_\mu(s,a)\Big|_{a=\mu(s)}\Big]$$

前面是一个 expectation，$s$ 服从一个 $\rho^\mu$ 分布（细节到书里去看）。括号里分两项：第一项是 $\mu$ 对 $\theta$ 的梯度，第二项是 $q_\mu$ 对 $a$ 的梯度。注意书写上的细节：竖线外面标着"$a$ 替换成 $\mu(S)$"，意思是**先**让 $q_\mu$ 对 $a$ 求梯度，**然后**再把式子里所有的 $a$ 全部替换成 $\mu(S)$。

有的同学可能会问：为什么不直接把 $a$ 写成 $\mu(S)$？因为如果直接替换，式子里就没有 $a$ 了——那对谁去求 $a$ 的梯度呢？这就不明显了，所以要这么写。

这个梯度和 stochastic 情况的梯度非常不一样，一个最重要的区别是：**里面没有涉及 action $a$**——因为 $a$ 最后会被替换成 $\mu(S)$，所以没有 $a$，自然也没有 $a$ 的所谓梯度。

由此直接得出一个结果：这是一个 off-policy 的算法。为什么？之后要对 true gradient 求 stochastic gradient，也就是采样：给定一个 $s$，我要得到一个 $a$，再得到 $r_{t+1}$ 等等。这时候我**不需要关心这个 $a$ 究竟是哪个策略得到的**——expectation 没有要求 $a$ 必须按哪一个策略产生，它可以是任意的。所以我可以用一个 behavior policy，其它任何一个策略都可以。

#### 算法流程与三点补充

有了梯度，应用到梯度上升：$\theta_{t+1} = \theta_t + \alpha 
abla_\theta J$，其中 $\alpha$ 是步长。expectation 求不出来，还是用 stochastic gradient 代替，得到下面的算法：

```text
behavior policy β ──生成数据──> 计算 TD error δt
                                     │
                                     ▼
                        critic：value update
                      （TD + 值函数近似，更新 q）
                                     │
                                     ▼
                    actor：用更新后的 q 代入 θ 更新式
                    θt+1 = θt + α·∇θμ(s,θ)·∇a qμ(s,a)|a=μ(s)
                                     │
                                     ▼
                        进入下一轮采样循环
```

读图：behavior policy $\beta$ 每次生成一些数据，先用它计算一个 TD error；TD error 送进 critic（value update step），这就是之前学过的 TD 加值函数近似的算法；$q$ 更新之后，再代入刚刚给出的 $\theta$ 更新算法。

下面有三点补充说明。

**第一，这是一个 off-policy 的实现。** behavior policy 是 $\beta$，target policy 是 $\mu$。注意 $\beta$ 其实也可以和 $\mu$ 一致：$\mu$ 是确定性的，它不能探索，所以可以给它加上一些噪音——比如它确定性地让我往这边走，加上噪音之后它可以左右稍微摇摆一点，就有了一定的探索性，再去执行下一时刻的采样任务。这种情况下它就变成了 on-policy 的实现。当然本质上来说它是 off-policy 的算法，所以它既可以用 off-policy，也可以用 on-policy 的实现方式。

**第二，$\mu+$noise 和 $\varepsilon$-greedy 很像，但在这里 $\varepsilon$-greedy 用不了。** $\mu+$noise 实际上和我们之前的 $\varepsilon$-greedy 方法非常类似，但这里不能用 $\varepsilon$-greedy，因为它的 action 是连续的——没办法在其它有限的 action 上面加一些比较小的概率。

**第三，$q$ 怎么选取？两种方法。** 第一种是之前应用非常广的线性函数——包括提出 DPG 的那篇论文当中也是这么用的：

$$q(s,a) = \phi^T w$$

$w$ 是它的参数，$\phi$ 是一个 feature vector。它的问题在于：一要自己选取 feature vector，二因为它是线性的，逼近真实 action value 的能力还是有限的。之后就已经推广到了基于神经网络的方法——DPG 这篇论文被推广成了 DDPG 那篇论文。

### 课程落幕：打牢基础，江湖再见

到这里，这次课的全部内容介绍完了，整个课程也就介绍完了。认真学完这门课的同学，你们的认真学习是对这些时间和精力付出最好的回报。还有同学留言问能不能讲更多其它的算法——强化学习里还有很多很好的算法，但时间和精力有限，没办法一一讲给大家；而且大家不会在这里学到所有的东西，网上有很多资料和论文原文，没有必要重复造轮子。这门课的想法就是给大家打好一个非常牢固的基础：认真学完它，未来读论文的时候会如鱼得水，不会有拦路虎。

课程虽然是终点，但对你们而言，是强化学习领域里一个全新的起点。没有下次课了，所以没有下次再见——希望有机会，江湖再见。

---

## 本册小结

本册主题为「策略方法：策略梯度、REINFORCE与Actor-Critic」，整编了 2 个模块（P46-P50 ~ P51-P54，共 9 讲 / 104 分钟音频）。
建议配合 `notes/` 目录下的复习笔记复盘，需要查证细节时可直接回到本册正文。
