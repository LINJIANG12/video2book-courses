# 策略梯度与Actor-Critic方法

策略梯度的整体思路只有三步：先立目标函数（metric），再算目标函数的梯度，最后把梯度放进梯度上升算法里优化策略参数。目标函数有两类——平均状态值与平均奖励；梯度归结为一个统一公式，借助 $\log$ 技巧可写成期望形式，从而能用采样近似；用蒙特卡洛估计 $q$ 得到 REINFORCE，用 TD 估计 $q$ 就进入 Actor-Critic。由此一路推广：加 baseline 得 A2C，加重要性采样得 off-policy 版本，把随机策略换成确定性策略得 DPG。

## 目标函数：平均状态值与平均奖励

* **平均状态值（average state value）**

    * > **定义**：所有状态的 state value 按权重 $d(s)$ 加权平均得到的标量，记作 $\bar{v}_\pi=\sum_{s\in\mathcal{S}}d(s)\,v_\pi(s)$，上横线代表"平均"。

    * 它是策略的函数：不同策略对应的值不同，所以可以优化它、找让这个值最大的策略。

    * $d(s)$ 满足 $d(s)\ge 0$ 且所有权重之和为 1，因此也可理解成概率分布——状态 $s$ 被选中的概率；此时目标函数可写成期望 $\bar{v}_\pi=\mathbb{E}_{S\sim d}[v_\pi(S)]$，也可写成向量内积 $\bar{v}_\pi=d^T v_\pi$（内积形式在后面求梯度时用到）。

* **权重 $d$ 与策略无关的情形（$d_0$）**

    * $d$ 与 $\pi$ 无关时求梯度只需对 $v_\pi$ 求，不涉及 $d$ 的梯度，处理简单；此时记作 $d_0$，对应 metric 记作 $\bar{v}_{\pi_0}$。

    * 最简单的选法是均匀分布：$n$ 个状态各占权重 $1/n$，即把所有状态视为同等重要。

    * 另一类重要选法是只关心某个出发状态 $s_0$（如游戏固定开局画面）：取 $d_0(s_0)=1$、其余全为 0，此时 $\bar{v}_\pi$ 退化为 $v_\pi(s_0)$——最大化它就是最大化从 $s_0$ 出发能获得的 return。

* **权重 $d$ 依赖于策略的情形（平稳分布）**

    * **平稳分布（stationary distribution，$d_\pi$）**：跟随策略 $\pi$ 与环境交互很多步后，agent 出现在各状态的概率趋于平稳，这个依赖于 $\pi$ 的分布就是 $d_\pi$，满足 $d_\pi^T P_\pi=d_\pi^T$，$P_\pi$ 是状态转移概率矩阵。

    * 直观含义：按某策略执行，有的状态访问得多、有的访问得少；访问多的状态概率 $d_\pi(s)$ 大、权重就大，访问少的权重自然小。

* **平均奖励（average reward / average one-step reward）**

    * > **定义**：单步即时奖励的平均值 $\bar{r}_\pi=\sum_{s\in\mathcal{S}}d_\pi(s)\,r_\pi(s)$，其中 $d_\pi(s)$ 是平稳分布，$r_\pi(s)$ 是从 $s$ 出发所得即时奖励的平均。

    * $r_\pi(s)=\sum_{a\in\mathcal{A}}\pi(a\mid s)\,r(s,a)$，即对每个动作按选择概率加权；$r(s,a)$ 本身是在 $(s,a)$ 处得到的即时奖励的平均值。链条是：$r(s,a)$ 对 $a$ 求期望得 $r_\pi(s)$，再对 $s$ 求期望得 $\bar{r}_\pi$。

    * 论文中常见的等价写法：$\bar{r}_\pi=\lim_{n\to\infty}\frac{1}{n}\,\mathbb{E}\big[\sum_{t=1}^{n}R_{t+1}\big]$——沿 trajectory 跑无穷多步后奖励的平均；式子里没有出发点 $s_0$，因为跑了无穷多步后从哪开始已不重要。

    * $\bar{v}_\pi$ 同样有轨迹形式：$\mathbb{E}_{S_0\sim d}\big[\sum_{t=0}^{\infty}\gamma^{t}R_{t+1}\big]$——按 total expectation 展开后，内层是 $G_t$ 即 $v_\pi(s)$，整个式子就等于 $\sum_s d(s)v_\pi(s)$。读 policy gradient / actor-critic 论文时见到的这类式子都是这两个 metric。

* **两个 metric 的关系与适用条件**

    * 两个 metric 全是策略参数 $\theta$ 的函数：不同 $\theta$ 得到不同值，从中找最优的 $\theta$ 就是 policy gradient 的基本思路。

    * $\bar{r}_\pi$ 对 immediate reward 求平均、不求 return，因此不需要 discount rate——它在 discounted case（$\gamma<1$）和 undiscounted case（$\gamma=1$）都成立；此前全书的推导都属 discounted case。

    * 直观上 $\bar{r}_\pi$ 只看单步、更"近视"（short sighted），$\bar{v}_\pi$ 看整条 return；但在 discounted case 下二者满足 $\bar{r}_\pi=(1-\gamma)\,\bar{v}_\pi$，优化其中一个等价于优化另一个。

    * > **易错点**：⚠️ "等价"不是说 $\bar{r}_\pi=\bar{v}_\pi$，而是差一个 $(1-\gamma)$ 的因子；等价的含义是优化其中一个时另一个也达到极值。

## 策略梯度公式与期望形式

* **统一梯度公式**

    * 计算这些 metric 的梯度是整个 policy gradient 方法中最复杂的部分：多个目标函数（$\bar{v}_\pi$、$\bar{r}_\pi$、$\bar{v}_{\pi_0}$）乘以 discounted / undiscounted 两种情况，每种组合的梯度形式都不同，但大同小异，可归结为一个式子：

    * $$\nabla_\theta J(\theta)=\sum_{s\in\mathcal{S}}\eta(s)\sum_{a\in\mathcal{A}}\nabla_\theta\pi(a\mid s,\theta)\,q_\pi(s,a)$$

    * 结构：外层对状态求和，权重是分布 $\eta(s)$；内层对动作求和，加的两项是策略的梯度 $\nabla_\theta\pi(a|s,\theta)$ 与 action value $q_\pi(s,a)$ 的乘积。

    * 对三个目标函数的对应关系：

    * | 目标函数 | 与公式的相等关系 | 权重分布 |
      | :--- | :--- | :--- |
      | $\bar{r}_\pi$ | discounted case 约等于，undiscounted case 严格等于 | $d_\pi$（平稳分布） |
      | $\bar{v}_\pi$ | 成比例等于（差一个比例因子） | $d_\pi$ |
      | $\bar{v}_{\pi_0}$ | 严格等于（最简单的情况） | $\rho_\pi$（另一个分布） |

* **log 技巧与期望形式**

    * 核心性质：$\nabla_\theta\ln\pi(a\mid s,\theta)=\dfrac{\nabla_\theta\pi(a\mid s,\theta)}{\pi(a\mid s,\theta)}$（$\log f(x)$ 求导先出 $1/f(x)$ 再乘 $f$ 的导数），两边整理得 $\nabla_\theta\pi=\pi\,\nabla_\theta\ln\pi$。

    * 把 $\nabla_\theta\pi$ 替换成 $\pi\,\nabla_\theta\ln\pi$ 后，对 $s$、对 $a$ 的两个求和都能写成期望，得到：

    * $$\nabla_\theta J(\theta)=\mathbb{E}_{S\sim\eta,\ A\sim\pi}\left[\nabla_\theta\ln\pi(A\mid S,\theta)\,q_\pi(S,A)\right]$$

    * 这个式子重要在：真实梯度含 expectation 而我们不知道 expectation，就可以用采样近似——这正是随机梯度上升（stochastic gradient ascent）的出发点。

* **softmax 策略**

    * 把 $\pi$ 放进 $\log$ 里要求 $\pi>0$，而 greedy / deterministic 策略对某些动作的 $\pi$ 值等于 0，不满足条件。

    * **softmax 函数（softmax function）**：把负无穷到正无穷的数归一化到 $(0,1)$ 区间且总和为 1，$z_i=\dfrac{e^{x_i}}{\sum_{j=1}^{n}e^{x_j}}$，既不会严格等于 0 也不会严格等于 1。

    * 据此强行令 $\pi(a\mid s,\theta)=\dfrac{e^{h(s,a,\theta)}}{\sum_{a\in\mathcal{A}}e^{h(s,a,\theta)}}$，$h$ 是类似特征函数的函数；实践中由神经网络输出：参数为 $\theta$、输入为 $s$ 的网络在输出层接 softmax，直接输出每个动作的 $\pi(a_i|s,\theta)$。

    * 这样得到的 $\pi$ 对每个动作都大于 0，因此是 stochastic 策略，天然具有探索性。

    * > **提示**：softmax 只适用于动作个数有限的情况（动作无穷多时行不通）；下一阶段的 deterministic policy gradient 不需要 $\pi>0$ 这个条件，且能处理无穷多动作。

## 梯度上升与 REINFORCE

* **从真实梯度到可用算法**

    * 梯度上升的基本形式：$\theta_{t+1}=\theta_t+\alpha\,\nabla_\theta J(\theta_t)$，$\alpha$ 是步长。

    * 真实梯度含 expectation，而环境的模型等信息不知道、无法计算期望，于是用随机梯度代替：$\theta_{t+1}=\theta_t+\alpha\,\nabla_\theta\ln\pi(a_t\mid s_t,\theta_t)\,q_t(s_t,a_t)$。

    * 这个式子仍不能用——里面有真实的 action value $q_\pi$；解决办法是用近似或采样把 $q_\pi$ 换成 $q_t$。用蒙特卡洛（从 $(s,a)$ 出发跑一个 episode，用其 return $g$ 当作 $q_t$）得到的就是 REINFORCE；用 TD 等其他方法则引出 actor-critic 系列算法。

* **on-policy 的由来**

    * 真实梯度中的随机变量是 $(S,A)$：理论上 $S$ 应服从长期平稳分布 $\eta$，但实际有数据就不错了，不会等到平稳状态才用数据，这一点实际中一般不考虑。

    * $A$ 必须服从 $\pi$：在 $s_t$ 按当前 $\pi$ 采 $a$，因此 behavior policy 与 target policy 都是 $\pi$（改进完立刻用它生成新数据再改进自己），所以 policy gradient 是 on-policy 算法；off-policy 需要额外技巧。

* **换个角度读算法：$\beta_t$ 在平衡什么**

    * 把 $\nabla_\theta\ln\pi$ 展开成 $\pi$ 的梯度除以 $\pi$，算法可重写为：$\theta_{t+1}=\theta_t+\alpha\,\beta_t\,\nabla_\theta\pi(a_t\mid s_t,\theta_t)$，其中 $\beta_t=\dfrac{q_t(s_t,a_t)}{\pi(a_t\mid s_t,\theta_t)}$。

    * 这说明算法实际在优化 $\pi(a_t|s_t)$ 这个量：它就是"步长 × 目标函数梯度"的梯度上升形式，通过改变 $\theta$ 去增大（或减小）$\pi(a_t|s_t)$。

    * 由一阶泰勒展开 $\pi(a_t|s_t,\theta_{t+1})\approx\pi(a_t|s_t,\theta_t)+\alpha\,\beta_t\,\big\|\nabla_\theta\pi(a_t\mid s_t,\theta_t)\big\|^2$（把 $\theta_{t+1}-\theta_t=\alpha\beta_t\nabla_\theta\pi$ 代入后梯度转置相乘得梯度的模平方），当 $\alpha\beta_t$ 较小时：$\beta_t>0$ 则下一时刻 $\pi(a_t|s_t)$ 增大，$\beta_t<0$ 则减小；步长越大增幅越大，但步长太大可能越过极值点，不保证上升。

    * 平衡机制：分子是 $q_t$——$q_t$ 大则 $\beta_t$ 大，给已知高价值的动作更大概率，这是 exploitation（充分利用）；分母是 $\pi$——$\pi(a_t|s_t)$ 小则 $\beta_t$ 大，下一时刻给这个动作更多机会，这是 exploration（探索）。

* **REINFORCE 的完整流程**

    * REINFORCE 是最早、最简单的 policy gradient 算法，名称就来自 reinforcement learning；$q_t$ 用蒙特卡洛方法求——从 $(s_t,a_t)$ 出发跑完一个 episode，把该 episode 的 return $g_t$ 赋给 $q_t$。

    * 流程：

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

    * 因为蒙特卡洛是 offline（离线）的——必须把整条 episode 采集完毕才能开始算 return——所以 $\theta_{t+1}$ 不能立刻去产生数据，要等整条 episode 过完；换成 TD 方法后就可以变成 online：得到新的 $\theta_t$ 就立刻用它生成数据。

    * > **注意**：policy gradient 的基本思路就是一个优化问题——目标函数、梯度、梯度上升三步，抓不住这点就容易在细节里迷失。

## Actor-Critic 的基本结构与 QAC

* **actor 与 critic 分别是什么**

    * > **定义**：actor（演员）对应策略及更新策略的过程——策略用来 take action，所以叫 actor；critic（评论家）做 policy evaluation——通过估计 action value 或 state value 来评估当前策略好不好。

    * 从随机梯度上升式 $\theta_{t+1}=\theta_t+\alpha\,\nabla_\theta\ln\pi(a_t\mid s_t,\theta_t)\,q_t(s_t,a_t)$ 能直接看出二者的结合方式：整个算法是 actor——它更新策略参数 $\theta$，$\theta$ 变了策略就变；估计 $q_t$ 的部分是 critic——$q_t$ 是 $q_\pi$ 的近似，critic 负责评估当前策略的 action value。

    * Actor-Critic 不是与 policy gradient 不同的新方法，就是把基于 value 的方法（特别是 value function approximation）引入 policy gradient 后得到的。

* **QAC：最简单的 Actor-Critic**

    * **QAC（QAC，Q 指 q value / action value）**：critic 用 Sarsa 算法加值函数近似来估计 $q$，actor 用 policy gradient 更新策略。

    * 流程：在第 $t$ 步按当前策略得到 $a_t$，与环境交互得 $r_{t+1},s_{t+1}$，再在 $s_{t+1}$ 按策略取 $a_{t+1}$，凑出 $s_t,a_t,r_{t+1},s_{t+1},a_{t+1}$ 五元组——这正是 Sarsa 的经验样本；critic 用它更新值函数参数 $w$ 得到 $\widehat{q}$，actor 把 $\widehat{q}$ 代入 policy update 得到新策略，新策略再生成下一步数据：

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

    * QAC 是 on-policy 的：梯度是期望 $\mathbb{E}_{S\sim\eta,A\sim\pi}[\cdots]$，采样时 $A$ 必须按 $\pi$ 采，$\pi$ 既是 behavior policy 又是 target policy。

    * 不需要再把策略转成 $\varepsilon$-greedy：policy gradient 方法里 $\pi(a|s,\theta)$ 本来就必须大于 0、策略已是 stochastic 的，天然具有探索能力。

    * QAC 虽简单，但揭示了 actor-critic 最本质的想法，后续算法都在它基础上推广。

## 减掉一个 Baseline：不改变期望，改变方差

* **baseline 不改变梯度**

    * **基线（baseline，$b$）**：在梯度的期望里给 $q_\pi(s,a)$ 减去的一个标量函数 $b(s)$；不管 $b$ 是什么函数，减去它之后梯度与原来完全相同。

    * 证明的关键两步：$\nabla_\theta\log\pi=\nabla_\theta\pi/\pi$ 中分子分母的 $\pi$ 一抵消，多出来的那项只剩 $\pi$ 本身的梯度；$b(s)$ 只是 $s$ 的函数、与 $a$ 无关，才能作为公因子提到对 $a$ 的求和外面，剩下的 $\sum_a\pi(a|s,\theta)=1$，对 1 求梯度为 0，整项为零。

    * > **易错点**：⚠️ baseline 必须只是 $s$ 的函数，绝不能是 $a$ 的函数——一旦 $b$ 依赖 $a$，它提不出对 $a$ 的求和，梯度就真的被改变了。

* **baseline 影响方差**

    * 把梯度括号里的随机变量记作 $X$，梯度就是 $\mathbb{E}[X]$；$b$ 不改变期望，但 $X$ 的方差与 $b$ 有关（$X$ 是向量时方差是矩阵，用矩阵的 trace 衡量大小，可算出 $\mathbb{E}[X^TX]-\bar{x}^T\bar{x}$，$\bar{x}=\mathbb{E}[X]$）。

    * 方差展开后含这样一项：$\mathbb{E}\big[\|\nabla_\theta \log \pi\|^2\,(q_\pi(s,a)-b(s))^2\big]$——$b$ 取 0 还是取 100 万，这一项天差地别，所以 baseline 实实在在影响方差。

    * 找好 baseline 的动机：方差越小，采样误差越小——方差小时随便采一个样本都紧贴 $\mathbb{E}[X]$，方差大时采到的值可能与真实期望差很远。

    * REINFORCE 和 QAC 都没有 baseline，数学上相当于 $b=0$，而 $b=0$ 并不是一个好的 baseline。

    * > **结论**：减 baseline 是"免费的午餐"——期望一点不变，方差却可以大幅下降，采样自然更准。

* **最优 baseline 与常用替代**

    * 使方差最小的最优 baseline 是：$b^*(s)=\dfrac{\mathbb{E}\big[\|\nabla_\theta\log\pi\|^2\,q_\pi(s,a)\big]}{\mathbb{E}\big[\|\nabla_\theta\log\pi\|^2\big]}$——上下各一个期望，把 $\|\nabla_\theta\log\pi\|^2$ 看成权重。

    * 这个式子太复杂，平常不用；常用做法是把权重去掉、只求 $q$ 的期望：$b(s)=\mathbb{E}_{a\sim\pi}[q_\pi(s,a)]=v_\pi(s)$，即 state value。

    * > **易错点**：⚠️ 有些文献说 $v_\pi(s)$ 是最优 baseline——它不是最优的，只是因为真正的最优太复杂才退而求其次、经常用它而已。

## A2C：优势函数与 TD error

* **优势函数（advantage function，$\delta_\pi$）**

    * 把 baseline 设为 $v_\pi(s)$，梯度里的 $q_\pi-v_\pi$ 记作新函数：$\delta_\pi=q_\pi(s,a)-v_\pi(s)$，称为优势函数，常用符号是 $A$（因 $A$ 已被 action 占用，这里用 $\delta$ 表示）。

    * 名字由来：$v_\pi$ 是 $q_\pi$ 在某状态下对所有动作的平均值；某动作的 $q_\pi$ 比这个平均值大，说明这个动作有优势。

    * 引入后 A2C 的更新式为：$\theta_{t+1}=\theta_t+\alpha_t\,\dfrac{\delta_t}{\pi(a_t|s_t,\theta_t)}\,\nabla_\theta\pi(a_t\mid s_t,\theta_t)$——分子是 $\delta$（优势）、分母是 $\pi$。

    * 分子分母的平衡与之前分析 $\beta_t$ 完全一样：$\delta_t$ 大说明该动作 action value 大，下一时刻增大 $\pi(a_t|s_t)$，这是充分利用；分母 $\pi(a_t|s_t)$ 小则步长被推大、下一时刻多给它机会，这是增加探索。

    * $q_t$ 好（减过 baseline 的 $\delta_t$ 好）还是原 $q_t$ 好？显然是 $\delta_t$ 好：在乎的不是 action value 的绝对值而是相对值——一个动作 value 是 1 亿但同状态其他动作都是 2 亿，它仍不是好动作；value 是 0 而其他都是 $-1$，0 就是好动作。$q$ 减 $v$ 衡量的正是"比平均好多少"。

* **用 TD error 代替优势函数**

    * 把 $\delta_t=q_t-v_t$ 中的 $q_t$ 替换成 $r_{t+1}+\gamma v_t(s_{t+1})-v_t(s_t)$，得到：

    * $$\delta_t = r_{t+1} + \gamma v_t(s_{t+1}) - v_t(s_t)$$

    * 这就是 TD error；替换的合理性：从期望角度，对 $q_\pi-v_\pi$ 求期望等于对 $R+\gamma v_\pi(S')-v_\pi(S)$ 求期望，两者是一回事。

    * 好处：直接用 $q-v$ 需要两个网络（一个近似 $q_t$、一个近似 $v_t$）；换成 TD error 后只需一个神经网络近似 $v_t$。

    * 引入 baseline + 用 TD error 估计值函数后得到的算法就是 A2C（advantage actor-critic），也叫 TD actor-critic——因为其中用 TD error 来估计 value function。

* **A2C 的算法流程**

    * 每一步：策略 $\pi(a|s_t,\theta_t)$ 采样出 $a_t$，与环境交互得 $r_{t+1},s_{t+1}$；用这一步数据算 TD error $\delta_t$；同一个 $\delta_t$ 先送进 critic（TD 算法 + 值函数近似，更新 $v_t$），再复用到 actor 的 policy update 里更新 $\theta_t$，然后进入下一轮循环：

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

    * A2C 是 on-policy 的：$\pi$ 是 stochastic 的，采样本身有探索性，不需要 $\varepsilon$-greedy 这类方法。

## 重要性采样

* **动机：off-policy 想复用旧经验**

    * 到目前为止的 AC 与 policy gradient 方法都是 on-policy 的，本质原因：梯度是一个 expectation，其中的 $a$ 要服从 $\pi$，采样时 $a$ 必须按 $\pi$ 产生——behavior policy 与 target policy 相同。

    * 想 off-policy 的动机：手上已有通过其它方法得到的经验，想复用这些经验去优化 target policy；实现这一转换的技术就是重要性采样（importance sampling）。

    * 重要性采样不专属 AC 方法：任何要求 expectation 的算法（包括蒙特卡洛方法、TD 方法）都可以用它。

* **问题设定：样本来自另一个分布**

    * 最简单的例子：$X$ 取值只有 $+1$ 和 $-1$。在 $p_0$ 下各以概率 0.5 出现，$\mathbb{E}_{p_0}[X]=0$；在 $p_1$ 下 $+1$ 概率 0.8、$-1$ 概率 0.2，$\mathbb{E}_{p_1}[X]=0.6$。

    * 老路（样本服从 $p_0$ 时）：直接对样本求平均得 $\bar{x}$，它是 $\mathbb{E}[X]$ 的无偏估计、方差是 $X$ 方差的 $1/n$（背后是大数定理），样本越多越精确。

    * 新问题：样本是在 $p_1$ 下采的，直接求平均最后一定趋向 0.6 而不是 0——**用 $p_1$ 下的样本估计 $p_0$ 下的期望**。对应到 off-policy：behavior policy $\beta$ 产生数据（对应 $p_1$），要估计 target policy $\pi$ 分布下的期望（对应 $p_0$）。

* **核心公式：一个加权平均**

    * 推导只需在分子分母上同乘同除一个 $p_1(x)$：

    * $$\mathbb{E}_{p_0}[X] = \sum_x p_0(x)\, x = \sum_x p_1(x)\, \frac{p_0(x)}{p_1(x)}\, x = \mathbb{E}_{p_1}\big[f(X)\big], \quad f(x) \triangleq \frac{p_0(x)}{p_1(x)}\, x$$

    * 用法：对一批服从 $p_1$ 的样本 $x_i$ 算 $f(x_i)$ 再求平均，$\bar{f}=\dfrac{1}{n}\sum_{i=1}^{n}\dfrac{p_0(x_i)}{p_1(x_i)}\,x_i$；$\bar{f}$ 的期望等于 $\mathbb{E}_{p_1}[f(X)]$，方差是 $f(X)$ 方差的 $1/n$，而后者等于最初要求的 $\mathbb{E}_{p_0}[X]$——分布对上了，就能用样本近似。

    * 每个样本携带的权重 $\dfrac{p_0(x_i)}{p_1(x_i)}$ 称为 importance weight（重要性权重）；$p_1=p_0$ 时权重全为 1，退化为普通平均。

    * 权重大于 1 的直观解释：$p_0(x_i)$ 大说明它在 $p_0$ 下容易采到，$p_1(x_i)$ 小说明它在 $p_1$ 下很难采到；既然要算 $p_0$ 下的期望，就要珍惜这个难得的样本、给它大权重，把结果"拉回来"。

    * > **注意**：常见疑问——"都知道 $p_0(x)$ 了，为什么不直接对 $p_0(x)\,x$ 求和？"答案是：能那么做就不需要重要性采样；用它的前提正是不能这么做。具体而言：能给单个 $x$ 算出 $p_0(x)$ 的值，但求不出期望——连续情况要求积分、对 $p_0$ 表达式要求很高；更关键的情况是 $p_0$ 由神经网络表示（如 $\pi(a|s,\theta)$），只有数值输出、根本没有解析表达式，无从积分。

* **两条线各归各的期望**

    * 同一批 $p_1$ 样本、两个统计量，结论直白：在哪个分布下采样，直接平均就得到哪个分布的期望；要另一个分布的期望就得靠 importance weight 加权。

    * | 统计量 | 计算方式 | 收敛到 |
      | :--- | :--- | :--- |
      | $\bar{x}$ | 直接对样本求平均 | $\mathbb{E}_{p_1}[X]=0.6$ |
      | $\bar{f}$ | 带 importance weight 的加权平均 | $\mathbb{E}_{p_0}[X]=0$ |

## Off-policy Actor-Critic

* **目标函数与梯度**

    * 设 $\beta$ 是产生经验采样的 behavior policy，目标函数取 $j(\theta)=\sum_s d_\beta(s)\,v_\pi(s)$，其中 $d_\beta(s)$ 是 $\beta$ 下的平稳分布、$v_\pi(s)$ 是 target policy $\pi$ 下的 state value。

    * 对应的梯度是：

    * $$\nabla_\theta j(\theta) = \mathbb{E}_{s\sim\rho,\, a\sim\beta}\left[\frac{\pi(a|s,\theta)}{\beta(a|s)}\, \nabla_\theta \log\pi(a|s,\theta)\, q_\pi(s,a)\right]$$

    * 与 on-policy 版本的关键区别：$a$ 改为服从 $\beta$（样本按 $\beta$ 采），括号内其余与 on-policy 相同，前面多出的权重 $\frac{\pi}{\beta}$ 就是重要性采样——$\pi$ 对应 $p_0$，$\beta$ 对应 $p_1$。

* **算法形式与分析**

    * baseline 照样能加而不改变梯度，常见做法仍是取 $b(s)=v_\pi(S)$；true gradient 的 expectation 求不出来，仍用 stochastic gradient 代替；$q_t-v_t$ 照样用 TD error $r_{t+1}+\gamma v_t(s_{t+1})-v_t(s_t)$ 代替，也可理解成 advantage function。

    * 把 $\nabla_\theta\log\pi$ 展开成 $\pi$ 的梯度除以 $\pi$，与外面的 $\frac{\pi}{\beta}$ 相约，更新式变成：

    * $$\theta_{t+1} = \theta_t + \alpha_t\, \frac{\delta_t}{\beta(a_t|s_t)}\, \nabla_\theta \pi(a_t|s_t,\theta_t)$$

    * 分子 $\delta_t$ 大时下一时刻增大 $\pi(a_t|s_t)$，是充分利用；但分母从 $\pi$ 换成了 $\beta$——$\beta$ 是固定值，$\pi(a_t|s_t)$ 小也不会自动增大，**所以这个版本里没有探索机制，只有充分的利用**。

    * 伪代码与 A2C 一模一样，区别只有两处：多了一个 behavior policy $\beta$，产生经验时要按 $\beta$ 采样；式子里多了两个 importance weight。

## 确定性策略梯度（DPG）

* **为什么要确定性策略**

    * 前面所有方法（QAC、A2C、off-policy AC）的策略 $\pi(a|s,\theta)$ 全都严格大于 0，一定是 stochastic 的；stochastic 策略由网络输出每个动作的概率，**对动作个数有要求——只能是有限个**，网络不可能输出无限个数。

    * 一个状态下有无限（连续）个 action 时，就要换成 deterministic 策略。

* **从输出概率到直接输出动作**

    * 之前的 $\pi(a|s,\theta)$ 表示"在 $s$ 取 $a$ 的概率"，是 0 到 1 之间的数；deterministic 时直接写成 $a=\mu(s,\theta)$——$\mu$ 的输出就是动作 $a$ 本身（如动作的索引是向上还是向下），不是任何概率。

    * $\mu$ 是从状态空间到动作空间的映射：状态的点确定时动作的点就确定；实践中用神经网络实现，参数 $\theta$、输入 $s$、输出 $a$，常简写为 $\mu(s)$（$\theta$ 仍是它的参数）。

* **目标函数与梯度**

    * 目标函数：$J(\theta)=\sum_s d_0(s)\,v_\mu(s)$，$v_\mu(s)$ 是策略 $\mu$ 下的 state value，$d_0$ 是与 $\mu$ 无关的概率分布。两种重要的特殊取法：

        * 只关心某个出发状态：每次任务都从 $s_0$ 开始，取 $d_0(s_0)=1$、其余为 0，目标函数成为 $v_\mu(s_0)$——最大化从 $s_0$ 出发的 return。

        * $d_0$ 取某个 behavior policy 的平稳分布：这与 off-policy 相关——deterministic policy gradient **天然就是 off-policy 的**，不需要用 importance sampling 去转换。

    * discounted 情况下的梯度：

    * $$\nabla_\theta J(\theta) = \mathbb{E}_{s\sim\rho^\mu}\Big[\nabla_\theta \mu(s,\theta)\; \nabla_a q_\mu(s,a)\Big|_{a=\mu(s)}\Big]$$

    * 两项相乘：$\mu$ 对 $\theta$ 的梯度，以及 $q_\mu$ 对 $a$ 的梯度；竖线标记 $a=\mu(s)$ 表示**先**让 $q_\mu$ 对 $a$ 求梯度、**然后**把所有 $a$ 替换成 $\mu(S)$——不能直接写成 $\mu(S)$，否则式子里没有 $a$、看不出对谁求梯度。

    * 与 stochastic 情形最重要的区别：梯度里**没有涉及 action $a$**（$a$ 最终被替换成 $\mu(S)$），也就没有 $a$ 的所谓梯度；expectation 没有要求 $a$ 必须按哪个策略产生，所以采样时 $a$ 可以由任意 behavior policy 给出——这直接得出 DPG 是 off-policy 算法。

* **算法流程与实现要点**

    * 流程：behavior policy $\beta$ 生成数据，用它计算 TD error；TD error 送进 critic（TD + 值函数近似，更新 $q$）；$q$ 更新后代入 actor 的 $\theta$ 更新式，进入下一轮采样循环：

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

    * off-policy / on-policy 双实现：behavior policy 是 $\beta$、target policy 是 $\mu$，本质上 off-policy；$\mu$ 是确定性的、不能探索，可以给它加噪音（确定往某方向走但左右稍微摇摆）获得探索性，此时 $\beta$ 与 $\mu$ 一致，就成了 on-policy 实现——两种实现方式都可用。

    * $\mu+$noise 与 $\varepsilon$-greedy 很像，但这里用不了 $\varepsilon$-greedy：动作是连续的，没办法在其它有限个动作上加小概率。

    * $q$ 的选取有两种：一是线性函数 $q(s,a)=\phi^T w$（$\phi$ 是 feature vector，提出 DPG 的原论文用的就是它），问题是要自己选特征、线性逼近能力有限；二是之后推广出的基于神经网络的方法——DPG 论文被推广成了 DDPG 论文。
