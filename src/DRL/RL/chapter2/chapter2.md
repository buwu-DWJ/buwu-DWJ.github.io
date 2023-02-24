## 二． 回合更新价值迭代

本章开始介绍无模型的机器学习算法．无模型的机器学习算法在没有环境的数学描述的情况下，只依靠经验（例如轨迹的样本) 学习出给定策略的价值函数和最优策略．在现实生活中，为环境建立精确的数学模型往往非常困难．因此，无模型的强化学习是强化学习的主要形式．

根据价值函数的更新时机，强化学习可以分为回合更新算法和时序差分更新算法这两类．回合更新算法只能用于回合制任务，它在每个回合结束后更新价值函数．本章将介绍回合更新算法，包括同策回合更新算法和异策回合更新算法．

### 2.1 同策回合更新

本节介绍同策回合更新算法．与有模型迭代更新的情况类似，我们也是先学习同策策略评估，再学习最优策略求解．

#### 2.1.1 同策回合更新策略评估

本节考虑用回合更新的方法学习给定策略的价值函数．我们知道，状态价值和动作价值分别是在给定状态和状态动作对的情况下回报的期望值．回合更新策略评估的基本思路使用 Monte Carlo 方法来估计这个期望值．具体而言，在许多轨迹样本中，如果某个状态(或状态动作对) 出现了 $c$ 次，其对应的回报值分别为 $g_{1}, g_{2}, \ldots, g_{c}$ ，那么可以估计其状态价 (或动作价值) 为 $\frac{1}{c} \sum_{i=1}^{c} g_{i}$．

无模型策略评估算法有**评估状态价值函数**和**评估动作价值函数**两种版本．在有模型情况下，状态价值和动作价值可以互相表示；但是在无模型的情况下，状态价值和动作价值并不能互相表示．我们已经知道，任意策略的价值函数满足 Bellman 期望方程．借助于动力 $p$（某个状态转移分布）的表达式，我们可以用状态价值函数表示动作价值函数；借助于策略 $\pi$ 的表达式，我们可以用动作价值函数表示状态价值函数．所以，对于无模型的策略评估， $p$ 的表达式未知，只能用动作价值表示状态价值，而不能用状态价值表示动作价值．另外，由于策略改进可以仅由动作价值函数确定，因此在学习问题中，动作价值函数往往更加重要．

在同一个回合中，多个步骤可能会到达同一个状态 (或状态动作对)，即同一状态（或状态动作对）可能会被多次访问．对于不同次的访问，计算得到的回报样本值很可能不相 同．如果采用回合内全部的回报样本值更新价值函数, 则称为**每次访问回合更新**（every visit Monte Carlo update); 如果每个回合只采用第一次访问的回报样本更新价值函数，则称为**首次访问回合更新**（ first visit Monte Carlo update)．每次访问和首次访问在学习过程中的中间值并不相同，但是它们都能收敛到真实的价值函数．

首先来看每次访问回合更新策略评估算法．算法 2-1 给出了每次访问更新求动作价值的算法．我们来逐步看一下算法 2-1 ．算法 2-1 首先对动作价值 $q(s, a)$ 进行初始化． $q(s, a)$ 可以初始化为任意的值，因为在第一次更新后 $q(s, a)$ 的值就和初始化的值没有关系，所以将 $q(s, a)$ 初始化为什么数无关紧要．接着，算法 2-1 进行回合更新．与有模型迭代更新的情形类似，这里可以用参数来控制回合更新的回合数．例如，可以使用最大回合数 $k_{\max }$ 或者精度指标 $\vartheta_{\max }$ ．在生成好轨迹后，算法 2-1 采用逆序的方式更新 $q(s, a)$ ．这里采用逆序是为了使用 $G_{t}=R_{t+1}+\gamma G_{t+1}$ 这一关系来更新 $G$ 值，以减小计算复杂度．

>**算法 2-1**：$\quad$每次访问回合更新评估策略的动作价值
***********************
输入：环境（无数学描述)，策略 $\pi$
输出：动作价值函数 $q(s, a), s \in \mathcal{S}, a \in \mathcal{A}$

1. (初始化) 初始化动作价值估计 $q(s, a)\leftarrow$ 任意值， $s \in \mathcal{S}, a \in \mathcal{A}$ ，若更新价值需要使用计数 器，则初始化计数器 $c(s, a) \leftarrow 0, s \in \mathcal{S}, a \in \mathcal{A}$ 。
2. (回合更新) 对于每个回合执行以下操作
    2.1 (采样) 用策略 $\pi$ 生成轨迹 $S_{0}, A_{0}, R_{1}, S_{1}, \ldots, S_{T-1}, A_{T-1}, R_{T}, S_{T}$
    2.2 (初始化回报） $G \leftarrow 0$
    2.3 (逐步更新）对 $t \leftarrow T-1, T-2, \ldots, 0$ 执行以下步骤
        1. （更新回报） $G \leftarrow \gamma G+R_{t+1}$
        2. （更新动作价值）更新 $q\left(S_{t}, A_{t}\right)$ 以减小 $\left[G-q\left(S_{t}, A_{y}\right)\right]^{2}$ （如 $c\left(S_{t}, A_{t}\right) \leftarrow c\left(S_{t}, A_{t}\right)+1$ ， $\left.q\left(S_{t}, A_{t}\right) \leftarrow q\left(S_{t}, A_{t}\right)+\frac{1}{c\left(S_{t}, A_{t}\right)}\left[G-q\left(S_{t}, A_{t}\right)\right]\right)$．

***********************

算法 2-1 在更新动作价值时，可以采用增量法来实现 Monte Carlo 方法．增量法的原理如下：如前 $c-1$ 次观察到的回报样本是 $g_{1}, g_{2}, \ldots, g_{c-1}$ ，则前 $c-1$ 次价值函数的估计值为$\bar{g}_{c-1}=\frac{1}{c-1} \sum_{i=1}^{c-1} g_{i}$ ；如果第 $c$ 次的回报样本是 $g_{c}$ ，则前 $c$ 次价值函数的估计值为 $\bar{g}_{c}=\frac{1}{c} \sum_{i=1}^{c}g_i$ 可以证明， $\bar{g}_{c}=\bar{g}_{c-1}+\frac{1}{c}\left(g_{c}-\bar{g}_{c-1}\right)$ ．所以，只要知道出现的次数 $c$ ，就可以用新的观测 $g_{c}$ 把旧的平均值 $\bar{g}_{c-1}$ 更新为新的平均值 $\bar{g}_{c}$ ．因此，增量法不仅需要记录当前的价值估计 $\bar{g}_{c-1}$ 还需要记录状态动作对出现的次数 $c$ ．在算法 2-1 中，状态动作对 $(s, a)$ 的出现次数记录在 $c(s, a)$ 里，每次更新时将计数值加 1，再更新平均值 $q(s, a)$ ，这样就实现了增量法．

求得动作价值后，可以用 Bellman 期望方程求得状态价值．状态价值也可以直接用回合更新的方法得到．算法 2-2 给出了每次访问回合更新评估策略的状态价值的算法．它与算法 2-1 的区别在于将 $q(s, a)$ 替换为了 $v(s)$ ，计数也相应做了修改．

>**算法 4-2**：$\quad$ 每次访问回合更新评估策略的状态价值
***********************
输入: 环境（无数学描述)，策略 $\pi$
输出：状态价值函数 $v(s), s \in \mathcal{S}$

1. （初始化）初始化状态价值估计 $v(s) \leftarrow$ 任意值， $s \in \mathcal{S}$ ，若更新价值时需要使用计数器则更新初始化计数器 $c(s) \leftarrow 0, s \in \mathcal{S}$
2. （回合更新）对于每个回合执行以下操作
 2.1 （采样）用策略 $\pi$ 生成轨迹 $S_{0}, A_{0}, R_{1}, S_{1}, \ldots, S_{T-1}, A_{T-1}, R_{T}, S_{T}$
 2.2 （初始化回报） $G \leftarrow 0$ 。
 2.3 （逐步更新） 对 $t \leftarrow T-1, T-2, \ldots, 0 ，$ 执行以下步骤 :
    1. （更新回报） $G \leftarrow \gamma G+R_{t+1}$
    2. （更新状态价值）更新 $v\left(S_{t}\right)$ 以减小 $\left[G-v\left(S_{t}\right)\right]^{2}\left(\right.$ 如 $c\left(S_{t}\right) \leftarrow c\left(S_{t}\right)+1, v\left(S_{t}\right) \leftarrow$，$\left.v\left(S_{t}\right)+\frac{1}{c\left(S_{t}\right)}\left[G-v\left(S_{t}\right)\right]\right)$

***********************

首次访问回合更新策略评估是比每次访问回合更新策略评估更为历史悠久、更为全面研究的算法．算法 2-3 给出了首次访问回合更新求动作价值的算法．这个算法和算法 2-1 的区别在于，在每次得到轨迹样本后，先找出各状态分别在哪些步骤被首次访问．在后续的更新过程中，只在那些首次访问的步骤更新价值函数的估计值．

>**算法 2-3**：$\quad$首次访问回合更新评估策略的动作价值
***********************
输入: 环境（无数学描述），策略 $\pi$
输出：动作价值函数 $q(s, a), s \in \mathcal{S}, a \in \mathcal{A}$ ．

1. （初始化）初始化动作价值估计 $q(s, a) \leftarrow$ 任意值， $s \in \mathcal{S}, a \in \mathcal{A}$ ，若更新动作价值时需要计数器，则初始化计数器 $c(s, a) \leftarrow 0, s \in \mathcal{S}, a \in \mathcal{A}$ ．
2. （回合更新）对于每个回合执行以下操作
 2.1  （采样）用策略 $\pi$ 生成轨迹 $S_{0}, A_{0}, R_{1}, S_{1}, \ldots, S_{T-1}, A_{T-1}, R_{T}, S_{T}$
 2.2 （初始化回报）$G \leftarrow 0$
 2.3 （初始化首次出现的步骤数）$f(s, a) \leftarrow-1, s \in \mathcal{S}, a \in \mathcal{A}$
 2.4 （统计首次出现的步骤数）对于 $t \leftarrow 0,1, \ldots, T-1$ ，执行以下步骤：如果 $f\left(S_{t}, A_{t}\right)<0$ ，则 $f\left(S_{t}, A_{t}\right) \leftarrow t$
 2.5 （逐步更新）对 $t \leftarrow T-1, T-2, \ldots, 0$ ，执行以下步骤：
   1. （更新回报） $G \leftarrow \gamma G+R_{t+1}$
   2. （首次出现则更新）如果 $f\left(S_{t}, A_{t}\right)=t$ ，则更新 $q\left(S_{t}, A_{t}\right)$ 以减小 $\left[G-q\left(S_{t}, A_{t}\right)\right]^{2}$如$\left.c\left(S_{t}, A_{t}\right) \leftarrow c\left(S_{t}, A_{t}\right)+1, \quad q\left(S_{t}, A_{t}\right) \leftarrow q\left(S_{t}, A_{t}\right)+\frac{1}{c\left(S_{t}, A_{t}\right)}\left[G-q\left(S_{t}, A_{t}\right)\right]\right)$．

***********************

与每次访问的情形类似，首次访问也可以直接估计状态价值，见算法 2-4 ．当然也可借助 Bellman 期望方程用动作价值求得状态价值．

>**算法 2-4**：$\quad$首次访问回合更新评估策略的状态价值
***********************
输入：环境（无数学描述），策略 $\pi$
输出：状态价值函数 $v(s), s \in \mathcal{S}$

1. （初始化）初始化状态价值估计 $v(s) \leftarrow$ 任意值， $s \in \mathcal{S}$ ，若更新价值时需要使用计数器，更新初始化计数器 $c(s) \leftarrow 0, s \in \mathcal{S}$
2. （回合更新）对于每个回合执行以下操作
 2.1 （采样）用策略 $\pi$ 生成轨迹 $S_{0}, A_{0}, R_{1}, S_{1}, \ldots, S_{T-1}, A_{T-1}, R_{T}, S_{T}$
 2.2 （初始化回报） $G \leftarrow 0$
 2.3 （初始化首次出现的步骤数） $f(s) \leftarrow-1, s \in \mathcal{S}$
 2.4 （统计首次出现的步骤数）对于 $t \leftarrow 0,1, \ldots, T-1$ ，执行以下步骤：如果 $f\left(S_{t}\right)<0$ ，则 $f\left(S_{t}\right) \leftarrow t$
 2.5 （逐步更新）对 $t \leftarrow T-1, T-2, \ldots, 0$ ，执行以下步骤 :
    1. （更新回报） $G \leftarrow \gamma G+R_{t+1}$
    2. （首次出现则更新）如果 $f\left(S_{t}\right)=t$ ，则更新 $v\left(S_{t}\right)$ 以减小 $\left[G-v\left(S_{t}\right)\right]^{2}\left(\right.$ 如 $c\left(S_{t}\right) \leftarrow$，$\left.c\left(S_{t}\right)+1, \quad v\left(S_{t}\right) \leftarrow v\left(S_{t}\right)+\frac{1}{c\left(S_{t}\right)}\left[G-v\left(S_{t}\right)\right]\right)$．

***********************

TODO:起始探索与柔性策略

### 2.2 异策回合更新

本节考虑异策回合更新．异策算法允许生成轨迹的策略和正在被评估或被优化的策路不是同一策略．我们将引人异策算法中一个非常重要的概念——重要性采样，并用其进行 策略评估和求解最优策略．

#### 2.2.1 重要性采样

在统计学上，**重要性采样**（importance sampling）是一种用一个分布生成的样本来估计另一个分布的统计量的方法．在异策学习中，将要学习的策略 $\pi$ 称为**目标策略**（target policy），将用来生成行为的另一策略 $b$ 称为行为策略（behavior policy )．重要性采样可以用行为策略生成的轨迹样本生成目标策略的统计量．

现在考虑从 $t$ 开始的轨迹 $S_{t}, A_{t}, R_{t+1}, S_{t+1}, A_{t+1}, \ldots, S_{T-1}, A_{T-1}, R_{T}, S_{T}$ ．在给定 $S_{t}$ 的条件下，采用
策略 $\pi$ 和策略 $b$ 生成这个轨迹的概率分别为：
$$
\begin{array}{l}
\operatorname{\mathbb{P}}_{\pi}\left[A_{t}, R_{t+1}, S_{t+1}, A_{t+1}, \ldots, S_{T-1}, A_{T-1}, R_{T}, S_{T}|S_{t}\right] \\
\quad=\pi\left(A_t|S_{t}\right) p\left(S_{t+1}, R_{t+1}|S_{t}, A_{t}\right) \pi\left(A_{t+1}|S_{t+1}\right) \cdots p\left(S_{T}, R_{T}|S_{T-1}, A_{T-1}\right) \\
\quad=\prod_{\tau=t}^{T-1} \pi\left(A_{\tau}|S_{\tau}\right) \prod_{\tau=t}^{T-1} p\left(S_{\tau+1}, R_{\tau+1}|S_{r}, A_{r}\right), \\
\operatorname{\mathbb{P}}_{b}\left[A_{t}, R_{t+1}, S_{t+1}, A_{t+1}, \ldots, S_{T-1}, A_{T-1}, R_{T}, S_{T}|S_{t}\right] \\
=b\left(A_{t}|S_{t}\right) p\left(S_{t+1}, R_{t+1}|S_{t}, A_{t}\right) b\left(A_{t+1}|S_{t+1}\right) \cdots p\left(S_{T}, R_{T}|S_{T-1}, A_{T-1}\right) \\
=\prod_{\tau=t}^{T-1} b\left(A_{\tau}|S_{\tau}\right) \prod_{\tau=t}^{T-1} p\left(S_{\tau+1}, R_{\tau+1}|S_{\tau}, A_{\tau}\right)
\end{array}
$$
我们把这两个概率的比值定义为**重要性采样比率** (importance sample ratio)：
$$
\rho_{t:T-1}=\frac{\operatorname{\mathbb{P}}_{\pi}\left[A_{t}, R_{t+1}, S_{t+1}, A_{t+1}, \ldots, S_{T-1}, A_{T-1}, R_{T}, S_{T}|S_{t}\right]}{\operatorname{\mathbb{P}}_{b}\left[A_{t}, R_{t+1}, S_{t+1}, A_{t+1}, \ldots, S_{T-1}, A_{T-1}, R_{T}, S_{T}|S_{t}\right]}=\prod_{\tau=t}^{T-1} \frac{\pi\left(A_{\tau}|S_{\tau}\right)}{b\left(A_{\tau}|S_{\tau}\right)}
$$
这个比率只与轨迹和策略有关，而与动力无关．为了让这个比率对不同的轨迹总是有意义，我们需要使得任何满足 $\pi(a|s)>0$ 的 $s \in \mathcal{S}, a \in \mathcal{A}(s)$ ，均有 $b(a|s)>0$ 这样的关系可以记为$\pi \ll b$.

对于给定状态动作对 $\left(S_{t}, A_{t}\right)$ 的条件概率也有类似的分析．在给定 $\left(S_{t}, A_{t}\right)$ 的条件下，采用策略 $\pi$ 和策略 $b$ 生成这个轨迹的概率分别为：
$$
\begin{array}{l}
\operatorname{\mathbb{P}}_{\pi}\left[R_{t+1}, S_{t+1}, A_{t+1}, \ldots, S_{T-1}, A_{T-1}, R_{T}, S_{T}|S_{t}, A_{t}\right] \\
\quad=p\left(S_{t+1}, R_{t+1}|S_{t}, A_{t}\right) \pi\left(A_{t+1}|S_{t+1}\right) \cdots p\left(S_{T}, R_{T}|S_{T-1}, A_{T-1}\right) \\
\quad=\prod_{\tau=t+1}^{T-1} \pi\left(A_{\tau}|S_{\tau}\right) \prod_{\tau=t}^{T-1} p\left(S_{\tau+1}, R_{\tau+1}|S_{\tau}, A_{\tau}\right) \\
\operatorname{\mathbb{P}}_{b}\left[R_{t+1}, S_{t+1}, A_{t+1}, \ldots, S_{T-1}, A_{T-1}, R_{T}, S_{T}|S_{t}, A\right] \\
\quad=p\left(S_{t+1}, R_{t+1}|S_{t}, A_t\right) b\left(A_{t+1}|S_{t+1}\right) \cdots p\left(S_{T}, R_{T}|S_{T-1}, A_{T-1}\right) \\
\quad=\prod_{\tau=t+1}^{T-1} b\left(A_{\tau}|S_{\tau}\right) \prod_{\tau=t}^{T-1} p\left(S_{\tau+1}, R_{\tau+1}|S_{\tau}, A_{\tau}\right)
\end{array}
$$
其概率的比值为
$$
\rho_{t+1: T-1}=\prod_{\tau=t+1}^{T-1} \frac{\pi\left(A_{\tau}|S_{\tau}\right)}{b\left(A_{\tau}|S_{\tau}\right)}
$$
回合更新总是使用 Monte Carlo 估计价值函数的值．同策回合更新得到 $c$ 个回报 $g_{1}, g_{2}, \ldots, g_{c}$ 后，用平均值 $\frac{1}{c} \sum_{i=1}^{c} g_{i}$ 来作为价值函数的估计．这样的方法实际上默认了这 $c$ 个回报是等概率出现的．类似的是，异策回合更新用行为策略 $b$ 得到 $c$ 个回报 $g_{1}, g_{2}, \ldots, g_{c}$ ，这个回报值对于行为策略 $b$ 是等概率出现的．但是这 $c$ 个回报值对于目标策略 $\pi$ 不是等概率出现的．对于目标策略 $\pi$ 而言，这 $c$ 个回报值出现的概率正是各轨迹的重要性采样比率．这样，我们可以用加权平均来完成 Monte Carlo 估计．具体而言，若 $\rho_{i}(1 \leqslant i \leqslant c)$ 是回报样本 $g_{i}$ 对应的权重（即轨迹的重要性采样比率)，可以有以下两种加权方法．

- **加权重要性采样**（weighted importance sampling ), 即 $\frac{\sum_{i=1}^{c} \rho_{i} g_{i}}{\sum_{i=1}^{c} \rho_{i}}$
- **普通重要性采样**（ordinary importance sampling), 即 $\frac{1}{c} \sum_{i=1}^{c} \rho_{i} g_{i}$

这两种方法的区别在于分母部分．对于加权重要性采样，如果某个权重 $\rho_{i}=0$ ，那么它不会让对应的 $g_{i}$ 参与平均，并不影响整体的平均值；对于普通重要性采样，如果某个权重 $\rho_{i}=0$ ，那么它会让 0 参与平均，使得平均值变小．无论是加权重要性采样还是普通重要性采样，当回报样本数增加时，仍然可以用增量法将旧的加权平均值更新为新的加权平均值．对于加权重要性采样，需要将计数值替换为权重的和，以
$$
\begin{array}{l}
c \leftarrow c+\rho \\
v \leftarrow v+\frac{\rho}{c}(g-v)
\end{array}
$$
的形式作更新．对于普通重要性采样而言，实际上就是对 $\rho_{i} g_{i}(1 \leqslant i \leqslant c)$ 加以平均，与直接没有加权情况下对 $g_{i}(1 \leqslant i \leqslant c)$ 加以平均没有本质区别．它的更新形式为 :
$$
\begin{array}{l}
c \leftarrow c+1 \\
v \leftarrow v+\frac{1}{c}(\rho g-v)
\end{array}
$$

#### 2.2.2 异策回合更新策略评估

基于 2.2.1 节给出的重要性采样，算法 2-7 给出了每次访问加权重要性采样回合更新策略评估算法．这个算法在初始化环节初始化了权重和 $c(s, a)$ 与动作价值 $q(s, a)$ $(s \in \mathcal{S}, a \in \mathcal{A}(s))$ ，然后进行回合更新．回合更新需要借助行为策略 $b$ ．行为策略 $b$ 可以每个回合都单独设计，也可以为整个算法设计一个行为策略，而在所有回合都使用同一个行为策略．用行为策略生成轨迹样本后，逆序更新回报、价值函数和权重值．一开始权重值 $\rho$ 设为 1 ，以后会越来越小．如果某次权重值变为 0 (这往往是因为 $\left.\pi\left(A_{t}|S_{t}\right)=0\right)$ ，那么以 后的权重值就都为 0 ，再循环下去没有意义．所以这里设计了一个检查机制．事实上，这个检查机制保证了在更新 $q(s, a)$ 时权重和 $c(s, a)>0$ 是必需的．如果没有检查机制，则可能在更新 $c(s, a)$ 时，更新前和更新后的 $c(s, a)$ 值都是 0 ，进而在更新 $q(s, a)$ 时出现除零错误．加这个检查机制避免了这样的错误．

>**算法 2-7**：$\quad$ 每次访问加权重要性采样异策回合更新评估策略的动作价值
***********************

1. （初始化）初始化动作价值估计 $q(s, a) \leftarrow$ 任意值， $s \in \mathcal{S}, a \in \mathcal{A}$ ，如果需要使用权重和，则初始化权重和 $c(s, a) \leftarrow 0, s \in \mathcal{S}, a \in \mathcal{A}$
2. （回合更新）对每个回合执行以下操作
 2.1 （行为策略）指定行为策略 $b$ ，使得 $\pi \ll b$
 2.2 （采样）用策略 $b$ 生成轨迹: $S_{0}, A_{0}, R_{1}, S_{1}, \ldots, S_{T-1}, A_{T-1}, R_{T}, S_{T}$
 2.3 （初始化回报和权重） $G \leftarrow 0, \rho \leftarrow 1$
 2.4 对于 $t \leftarrow T-1, T-2, \ldots, 0$ 执行以下操作：
    1. （更新回报） $G \leftarrow \gamma G+R_{t+1}$
    2. （更新价值）更新 $q\left(S_{t}, A_{t}\right)$ 以减小 $\rho\left[G-q\left(S_{t}, A_{t}\right)\right]^{2}\left(\right.$ 如 $c\left(S_{t}, A_{t}\right) \leftarrow c\left(S_{t}, A_t\right)+\rho$，$\left.q\left(S_{t}, A_{t}\right) \leftarrow q\left(S_{t}, A_{t}\right)+\frac{\rho}{c\left(S_{t}, A_{t}\right)}\left[G-q\left(S_{t}, A_{t}\right)\right]\right)$
    3. （更新权重） $\rho \leftarrow \rho \frac{\pi\left(A_{t}|S_{t}\right)}{b\left(A_{t}|S_{t}\right)}$
    4. （提前终止）如果 $\rho=0$ ，则结東步骤 2.4 的循环

***********************

在算法 2-7 的基础上略作修改，可以得到首次访问的算法、普通重要性采样的算法和估计状态价值的算法，此处略过．

#### 2.2.3 异策回合更新最优策略求解

接下来介绍最优策略的求解．算法 2-8 给出了每次访问加权重要性采样异策回合最优策略求解算法．它和其他最优策略求解算法一样，都是在策略估计算法的基础上加上策略改进得来的．算法 2-8 的迭代过程中，始终让 $\pi$ 是一个确定性策略．所以，在回合更新的过程中，任选一个策略 $b$ 都满足 $\pi \ll b$ ．这个柔性策略可以每个回合都分别选取，也可以整个程序共用一个．由于采用了确定性的策略，则对于每个状态 $s \in \mathcal{S}$ 都有一个 $a \in \mathcal{A}(s)$ 使得 $\pi(a|s)=1$ ，而其他 $\pi(\cdot|s)=0$ ．算法 2-8 利用这一性质来更新权重并判断权重是否为 0 ．如果 $A_{t} \neq \pi\left(S_{t}\right)$ ，则意味着 $\pi\left(A_{t}|S_{t}\right)=0$ ，更新后的权重为 0 ，需要退出循环以避免除零错误；若 $A_{t}=\pi\left(S_{t}\right)$ ，则意味着 $\pi\left(A_{t}|S_{t}\right)=1$ ，所以权重更新语句 $\rho \leftarrow \rho \frac{\pi\left(A_{t}|S_{t}\right)}{b\left(A_{t}|S_{t}\right)}$ 就可以简化为
$\rho \leftarrow \rho \frac{1}{b\left(A_{t}|S_{t}\right)}$．

>**算法 2-8**：$\quad$ 每次访问加权重要性采样异策回合更新最优策略求解

1. （初始化）初始化动作价值估计 $q(s, a) \leftarrow$ 任意值， $s \in \mathcal{S}, a \in \mathcal{A}$ ，如果需要使用权重和，初始化权重和 $c(s, a) \leftarrow 0, s \in \mathcal{S}, a \in \mathcal{A} ; \pi(s) \leftarrow \arg \max _{a} q(s, a), s \in \mathcal{S}$
2. （回合更新）对每个回合执行以下操作
 2.1 （柔性策略）指定 $b$ 为任意柔性策略
 2.2 （采样）用策略 $b$ 生成轨迹： $S_{0}, A_{0}, R_{1}, S_{1}, A_{1}, R_{2}, \ldots, S_{T-1}, A_{T-1}, R_{T}, S_{T}$
 2.3 （初始化回报和权重） $G \leftarrow 0, \rho \leftarrow 1$
 2.4 对 $t \leftarrow T-1, T-2, \ldots, 0:$
    1. （更新回报） $G \leftarrow \gamma G+R_{t+1}$
    2. （更新价值）更新 $q\left(S_{t}, A_{t}\right)$ 以减小 $\rho\left[G-q\left(S_{t}, A_{t}\right)\right]^{2}\left(\right.$ 如 $c\left(S_{t}, A_{t}\right) \leftarrow c\left(S_{t}, A_{t}\right)+\rho$，$\left.q\left(S_{t}, A_{t}\right) \leftarrow q\left(S_{t}, A_{t}\right)+\frac{\rho}{c\left(S_{t}, A_{t}\right)}\left[G-q\left(S_{t}, A_{t}\right)\right]\right)$
    3. （策略更新） $\pi\left(S_{t}\right) \leftarrow \arg \max_{a} q\left(S_{t}, a\right)$
    4. （提前终止）若 $A_{t} \neq \pi\left(S_{t}\right)$ 则退出步骤 2.4
    5. （更新权重） $\rho \leftarrow \rho \frac{1}{b\left(A_{t}|S_{t}\right)}$

***********************
算法 2-8 也可以修改得到首次访问的算法和普通重要性采样的算法，此处略过．