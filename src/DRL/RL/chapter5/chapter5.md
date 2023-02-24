## 五． 回合更新策略梯度方法

本书前几章的算法都利用了价值函数，在求解最优策略的过程中试图估计最优价值函数，所以那些算法都称为**最优价值算法**（optimal value algorithm)．但是，要求解最优策略不一定要估计最优价值函数．本章将介绍不直接估计最优价值函数的强化学习算法，它们试图用含参函数近似最优策略，并通过迭代更新参数值．由于迭代过程与策略的梯度有关，所以这样的迭代算法又称为策略梯度算法（policy gradient algorithm）．

### 5.1 策略梯度算法的原理

基于策略的策略梯度算法有两大核心思想：

- 用含参函数近似最优策略
- 用策略梯度优化策略参数

本节介绍这两部分内容．

### 5.1.1 函数近似与动作偏好

用函数近似方法估计最优策略 $\pi_*(a|s)$ 的基本思想是用含参函数 $\pi(a|s ; \theta)$ 来近似最优策略．由于任意策略 $\pi$ 都需要满足对于任意的状态 $s \in \mathcal{S}$ ，均有 $\sum_{a} \pi(a|s)=1$ ，我们也希望 $\pi(a|s ; \theta)$ 满足对于任意的状态 $s \in \mathcal{S}$ ，均有 $\sum_{a} \pi(a|s ; \theta)=1$ ．为此引入动作偏好函数（action preference function） $h(s, a ; \theta)$ ，其 softmax 的值为 $\pi(a|s ; \theta)$ ，即
$$
\pi(a|s ; \theta)=\frac{\exp h(s, a ; \theta)}{\sum_{o^{\prime}} \exp h\left(s, a^{\prime} ; \theta\right)}, \quad s \in \mathcal{S}, a \in \mathcal{A}(s)
$$
在第 3~4 章中，从动作价值函数导出最优策略估计往往有特定的形式 (如 $\varepsilon$ 贪心策
略)．与之相比，从动作偏好导出的最优策略的估计不拘泥于特定的形式，其每个动作都可以有不同的概率值，形式更加灵活．如果采用迭代方法更新参数 $\boldsymbol{\theta}$ ，随着迭代的进行， $\pi(a|s ; \theta)$ 可以自然而然地逼近确定性策略，而不需要手动调节 $\varepsilon$ 等参数．

动作偏好函数可以具有线性组合、人工神经网络等多种形式．在确定动作偏好的形式中，只需要再确定参数 $\theta$ 的值，就可以确定整个最优状态估计．参数 $\boldsymbol{\theta}$ 的值常通过基于梯度的迭代算法更新，所以，动作偏好函数往往需要对参数 $\boldsymbol{\theta}$ 可导．

### 5.1.2 策略梯度定理

策略梯度定理给出了期望回报和策略梯度之间的关系，是策略梯度方法的基础．本节学习策略梯度定理．

在回合制任务中，策略 $\pi(\theta)$ 期望回报可以表示为 $\mathrm{E}_{\pi(\theta)}\left[G_{0}\right]$ ．**策略梯度定理**（policy gradient theorem) 给出了它对策略参数 $\boldsymbol{\theta}$ 的梯度为
$$
\nabla \mathrm{E}_{\pi(\theta)}\left[G_{0}\right]=\mathrm{E}\left[\sum_{t=0}^{+\infty} \gamma^{t} G_{t} \nabla \ln \pi\left(A_{t}|S_{t} ; \boldsymbol{\theta}\right)\right]
$$
其等式右边是和的期望，求和的 $\gamma^{\prime} G_{t} \nabla \ln \pi\left(A_{t}|S_{t} ; \theta\right)$ 中，只有 $\nabla \ln \pi\left(A_{t}|S_{t} ; \theta\right)$ 显式含有参数 $\theta$ ．

策略梯度定理告诉我们，只要知道了 $\nabla \ln \pi\left(A_{t}|S_{t} ; \boldsymbol{\theta}\right)$ 的值，再配合其他一些容易获得的 值（如 $\gamma^{t}$ 和 $\left.G_{t}\right)$ ，就可以得到期望回报的梯度．这样，我们也可以顺着梯度方向改变 $\theta$ 以增大期望回报．

接下来我们来证明这个定理．回顾，策略 $\pi(\boldsymbol{\theta})$ 满足 $\mathrm{Bellman}$ 期望方程，即
$$
\begin{array}{ll}
v_{\pi(\theta)}(s)=\sum_{a} \pi(a|s ; \theta) q_{\pi(\theta)}(s, a), & s \in \mathcal{S} \\
q_{\pi(\theta)}(s, a)=r(s, a)+\gamma \sum_{s^{\prime}} p\left(s^{\prime}|s, a\right) v_{\pi(\theta)}\left(s^{\prime}\right), & s \in \mathcal{S}, a \in \mathcal{A}(s)
\end{array}
$$
将以上两式对 $\boldsymbol{\theta}$ 求梯度，有
$$
\begin{array}{ll}
\nabla v_{\pi(\theta)}(s)=\sum_{a} q_{\pi(\theta)}(s, a) \nabla \pi(a|s ; \theta)+\sum_{a} \pi(a|s ; \theta) \nabla q_{\pi(\theta)}(s, a), & s \in \mathcal{S} \\
\nabla q_{\pi(\theta)}(s, a)=\gamma \sum_{s^{\prime}} p\left(s^{\prime}|s, a\right) \nabla v_{\pi(\theta)}\left(s^{\prime}\right), & s \in \mathcal{S}
\end{array}
$$
将 $\nabla q_{\pi(\theta)}(s, a)$ 的表达式代人 $\nabla v_{\pi(\theta)}(s)$ 的表达式中，有
$$
\begin{aligned}
\nabla v_{\pi(\theta)}(s) &=\sum_{a} q_{\pi(\theta)}(s, a) \nabla \pi(a|s ; \boldsymbol{\theta})+\sum_{a} \pi(a|s ; \boldsymbol{\theta}) \gamma \sum_{s^{\prime}} p\left(s^{\prime}|s, a\right) \nabla v_{\pi(\theta)}\left(s^{\prime}\right) \\
&=\sum_{a} q_{\pi(\theta)}(s, a) \nabla \pi(a|s ; \boldsymbol{\theta})+\sum_{s^{\prime}} \operatorname{\mathbb{P}}\left[S_{t+1}=s^{\prime}|S_{t}=s ; \boldsymbol{\theta}\right] \gamma v_{\pi(\theta)}\left(s^{\prime}\right), \quad s \in \mathcal{S}
\end{aligned}
$$
在策略 $\pi(\theta)$ 下，对 $S_{t}$ 求上式的期望，有
$$
\begin{array}{l}
\mathrm{E}\left[\nabla v_{\pi(\theta)}\left(S_{t}\right)\right] \\
\quad =\sum_{s} \operatorname{\mathbb{P}}\left[S_{t}=s\right] \nabla v_{\pi(\theta)}(s)\\
\quad =\sum_{s} \operatorname{\mathbb{P}}\left[S_{t}=s\right]\left[\sum_{a} q_{\pi(\theta)}(s, a) \nabla \pi(a|s ; \theta)+\sum_{s^{\prime}} \operatorname{\mathbb{P}}\left[S_{t+1}=s^{\prime}|S_{t}=s ; \theta\right] \gamma\nabla v_{\pi(\theta)}\left(s^{\prime}\right)\right] \\
\quad =\sum_{s} \operatorname{\mathbb{P}}\left[S_{t}=s\right] \sum_{a} q_{\pi(\theta)}(s, a) \nabla \pi(a|s ; \theta)+\sum_{s} \operatorname{\mathbb{P}}\left[S_{t}=s\right] \sum_{s^{\prime}} \operatorname{\mathbb{P}}\left[S_{t+1}=s^{\prime}|S_{t}=s ; \theta\right] \gamma\nabla v_{\pi(\theta)}\left(s^{\prime}\right) \\
\quad =\sum_{s} \operatorname{\mathbb{P}}\left[S_{t}=s\right] \sum_{a} q_{\pi(\theta)}(s, a) \nabla \pi(a|s ; \theta)+\gamma \sum_{s} \operatorname{\mathbb{P}}\left[S_{t+1}=s^{\prime} ; \theta\right] \nabla v_{\pi(\theta)}\left(s^{\prime}\right) \\
\quad =E\left[\sum_{\sigma} q_{\pi(\theta)}\left(S_{t}, a\right) \nabla \pi\left(a|S_{t} ; \theta\right)\right]+\gamma E\left[\nabla v_{\pi(\theta)}\left(S_{t+1}\right)\right]
\end{array}
$$
这样就得到了从 $\mathrm{E}\left[\nabla v_{\pi(\theta)}\left(S_{t}\right)\right]$ 到 $\mathrm{E}\left[\nabla v_{\pi(\theta)}\left(S_{t+1}\right)\right]$ 的递推式．注意到最终关注的梯度值就是
$$
\nabla \mathrm{E}_{\pi(\theta)}\left[G_{0}\right]=\nabla \mathrm{E}\left[v_{\pi(\theta)}\left(S_{0}\right)\right]=\mathrm{E}\left[\nabla v_{\pi(\theta)}\left(S_{0}\right)\right]
$$
所以有
$$
\begin{array}{l}
\nabla \mathrm{E}_{\pi(\theta)}\left[G_{0}\right]=\mathrm{E}\left[\nabla v_{\pi(\theta)}\left(S_{0}\right)\right] \\
\quad=\mathrm{E}\left[\sum_{a} q_{\pi(\theta)}\left(S_{0}, a\right) \nabla \pi\left(a|S_{0} ; \boldsymbol{\theta}\right)\right]+\gamma \mathrm{E}\left[\nabla v_{\pi(\theta)}\left(S_{1}\right)\right] \\
\quad=\mathrm{E}\left[\sum_{a} q_{\pi(\theta)}\left(S_{0}, a\right) \nabla \pi\left(a|S_{0} ; \boldsymbol{\theta}\right)\right]+\mathrm{E}\left[\sum_{a} \gamma q_{\pi(\theta)}\left(S_{1}, a\right) \nabla \pi\left(a|S_{1} ; \boldsymbol{\theta}\right)\right]+\gamma^{2} \mathrm{E}\left[\nabla v_{\pi(\theta)}\left(S_{1}\right)\right] \\
\quad=\cdots \\
\quad=\sum_{t=0}^{+\infty} \mathrm{E}\left[\sum_{a} \gamma^{t} q_{\pi(\theta)}\left(S_{t}, a\right) \nabla \pi\left(a|S_{t} ; \boldsymbol{\theta}\right)\right]
\end{array}
$$
考虑到
$$
\nabla \pi\left(a|S_{t} ; \boldsymbol{\theta}\right)=\pi\left(a|S_{t} ; \boldsymbol{\theta}\right) \nabla \ln \pi\left(a|S_{t} ; \boldsymbol{\theta}\right)
$$
所以
$$
\begin{array}{l}
\mathrm{E}\left[\sum_{a} \gamma^{t} q_{\pi(\theta)}\left(S_{t}, a\right) \nabla \pi\left(a|S_{t} ; \boldsymbol{\theta}\right)\right] \\
\quad=\mathrm{E}\left[\sum_{a} \pi\left(a|S_{t} ; \boldsymbol{\theta}\right) \gamma^{t} q_{\pi(\theta)}\left(S_{t}, a\right) \nabla \ln \pi\left(a|S_{t} ; \boldsymbol{\theta}\right)\right] \\
\quad=\mathrm{E}\left[\gamma^{t} q_{\pi(\theta)}\left(S_{t}, A_{t}\right) \nabla \ln \pi\left(A_{t}|S_{t} ; \boldsymbol{\theta}\right)\right]
\end{array}
$$
又由于 $q_{\pi(\theta)}\left(S_{t}, A_{t}\right)=\mathrm{E}\left[G_{t}|S_{t}, A_{t}\right]$ ，所以
$$
\mathrm{E}\left[\sum_{a} \gamma^{t} q_{\pi(\theta)}\left(S_{t}, a\right) \nabla \pi\left(a|S_{t} ; \boldsymbol{\theta}\right)\right]=\mathrm{E}\left[\gamma^{t} q_{\pi(\theta)}\left(S_{t}, A_{t}\right) \nabla \ln \pi\left(A_{t}|S_{t} ; \boldsymbol{\theta}\right)\right]=\mathrm{E}\left[\gamma^{t} G_{t} \nabla \ln \pi\left(A_{t}|S_{t} ; \boldsymbol{\theta}\right)\right]
$$
得证．

### 5.2 同策回合更新策略梯度算法

策略梯度定理告诉我们，沿着 $\nabla E_{\pi(\theta)}\left[G_{0}\right]=\mathrm{E}\left[\sum_{t=0}^{+\infty} \gamma^{t} G_{t} \nabla \ln \pi\left(A_{t}|S_{t} ; \theta\right)\right]$ 的方向改变策略参数 $\theta$ 的值，就有机会增加期望回报．基于这一结论，可以设计策略梯度算法．本节考虑同策更新算法

#### 5.2.1 简单的策略梯度算法

在每一个回合结束后，我们可以就回合中的每一步用形如
$$
\boldsymbol{\theta}_{t+1} \leftarrow \boldsymbol{\theta}_{t}+\alpha \gamma^{t} G_{t} \nabla \ln \pi\left(A_{t}|S_{t} ; \boldsymbol{\theta}\right), \quad t=0,1, \ldots
$$
的迭代式更新参数 $\boldsymbol{\theta}$ ．这样的算法称为简单的策略梯度算法（Vanilla Policy Gradient, VPG)．

R Willims 在文章《Simple statistical gradient-following algorithms for connectionist reinforcement learning 》中给出了该算法，并称它为“REward Increment = Nonnegative Factor $\times$ Offset Reinforcement $\times$ Characteristic Eligibility” ( $\mathrm{REINFORCE})$ ，表示增量 $\alpha \gamma^{t} G_{t} \nabla \ln \pi\left(A_{t}|S_{t} ; \theta_{t}\right)$ 是由三个部分的积组成的．这样迭代完这个回合轨迹就实现了
$$
\boldsymbol{\theta} \leftarrow \boldsymbol{\theta}+\alpha \sum_{t=0}^{+\infty} \gamma^{t} G_{t} \nabla \ln \pi\left(A_{t}|S_{t} ; \boldsymbol{\theta}\right)
$$
在具体的更新过程中，不一定要严格采用这样的形式．当采用 TensorFlow 等自动微分的软件包来学习参数时，可以定义单步的损失为 $-\gamma^{\prime} G_{t} \ln \pi\left(A_{t}|S_{t} ; \boldsymbol{\theta}\right)$ ，让软件包中的优化器减小整个回合中所有步的平均损失，就会沿着 $\sum_{t=0}^{+\infty} \gamma^{\prime} G_{t} \nabla \ln \pi\left(A,|S_{t} ; \theta\right)$ 的梯度方向改变 $\theta$ 的值．

简单的策略梯度算法见算法 5-1．

>**算法 5-1**：$\quad$ 简单的策略梯度算法求解最优策略
***********************

输入：环境（无数学描述）
输出：最优策略的估计 $\pi(\boldsymbol{\theta})$
参数：优化器（隐含学习率 $\alpha$ ），折扣因子 $\gamma$ ，控制回合数和回合内步数的参数

1. （初始化） $\boldsymbol{\theta} \leftarrow$任意值
2. （回合更新）对每个回合执行以下操作
 2.1 （采样）用策略 $\pi(\theta)$ 生成轨迹 $S_{0}, A_{0}, R_{1}, S_{1}, \ldots, S_{T-1}, A_{T-1}, R_{T}, S_{T}$
 2.2 （初始化回报） $G \leftarrow 0$
 2.3 对 $t=T-1, T-2, \ldots, 0$ ，执行以下步骤：
    1. （更新回报） $G \leftarrow \gamma G+R_{t+1}$
    2. （更新策略）更新 $\boldsymbol{\theta}$ 以减小 $-\gamma^{\prime} G \ln \pi\left(A_{t}|S_{t} ; \boldsymbol{\theta}\right)\left(\text { 如 } \boldsymbol{\theta} \leftarrow \boldsymbol{\theta}+\alpha \gamma^{t} G \nabla \ln \pi\left(A_{t}|S_{t} ; \boldsymbol{\theta}\right)\right)$

***********************

### 5.2.2 带基线的简单策略梯度算法

本节介绍简单的策略梯度算法的一种改进一带基线的简单的策略梯度算法（REINFOCE with baselines)．为了降低学习过程中的方差，可以引人基线函数 $B(s)(s \in \mathcal{S})$ ．基线函数 $B$ 可以是任意随机函数或确定函数，它可以与状态 $s$ 有关，但是不能和动作 $a$ 有关．满足这样的条件后，基线函数 $B$ 自然会满足
$$
\mathrm{E}\left[\gamma^{t}\left(G_{t}-B\left(S_{t}\right)\right) \nabla \ln \pi\left(A_{t}|S_{t} ; \boldsymbol{\theta}\right)\right]=\mathrm{E}\left[\gamma^{t} G_{t} \nabla \ln \pi\left(A_{t}|S_{t} ; \boldsymbol{\theta}\right)\right]
$$
证明如下：由于 $B$ 与 $a$ 无关，所以
$$
\sum_{a} B\left(S_{t}\right) \nabla \pi\left(a|S_{t} ; \boldsymbol{\theta}\right)=B\left(S_{t}\right) \nabla \sum_{a} \pi\left(a|S_{t} ; \boldsymbol{\theta}\right)=B\left(S_{t}\right) \nabla 1=0
$$
进而
$$
\begin{aligned}
\mathrm{E} &\left[\gamma^{t}\left(G_{t}-B\left(S_{t}\right)\right) \nabla \ln \pi\left(A_{t}|S_{t} ; \boldsymbol{\theta}\right)\right] \\
&=\sum_{a} \gamma^{t}\left(G_{t}-B\left(S_{t}\right)\right) \nabla \pi\left(a|S_{t} ; \boldsymbol{\theta}\right) \\
&=\sum_{a} \gamma^{t} G_{t} \nabla \pi\left(a|S_{t} ; \boldsymbol{\theta}\right) \\
&=\mathrm{E}\left[\gamma^{t} G_{t} \nabla \ln \pi\left(A_{t}|S_{t} ; \boldsymbol{\theta}\right)\right]
\end{aligned}
$$
得证．
基线函数可以任意选择，例如以下情况

1) 选择基线函数为由轨迹确定的随机变量 $B\left(S_{t}\right)=-\sum_{\tau=0}^{t-1} \gamma^{\tau-t} R_{\tau+1}$ ，这时 $\gamma^{t}\left(G_{t}-B\left(S_{t}\right)\right)=G_{0}$ ，梯度的形式为 $\mathrm{E}\left[G_{0} \nabla \ln \pi\left(A_{t}|S_{t} ; \boldsymbol{\theta}\right)\right]$
2) 选择基线函数为 $B\left(S_{t}\right)=\gamma^{t} v_{*}\left(S_{t}\right)$ ，这时梯度的形式 为 $\mathrm{E}\left[\gamma^{t}\left(G_{t}-v_{*}\left(S_{t}\right)\right) \nabla \ln \pi\right.$
$\left.\left(A_{t}|S_{t} ; \theta\right)\right]$

但是，在实际选择基线时，应当参照以下两个思想．

- 基线的选择应当有效降低方差．一个基线函数能不能降低方差不容易在理论上判别， 往往需要通过实践获知．- 基线函数应当是可以得到的．例如我们不知道最优价值函数，但是可以得到最优价值函数的估计．价值函数的估计也可以随着迭代过程更新．

一个能有效降低方差的基线是状态价值函数的估计．算法 5-2 给出了用状态价值函数的估计作为基线的算法．这个算法有两套参数 $\boldsymbol{\theta}$ 和 $\mathbf{w}$ ，分别是最优策略估计和最优状态价值函数估计的参数．每次迭代时，它们都以各自的学习算法进行学习．算法 5-2 采用了随机梯度下降法来更新这两套参数（事实上也可以用其他算法)，在更新过程中都用到了 $G-v\left(S_{t} ; \mathbf{w}\right)$ ，可以在更新前预先计算以减小计算量．

>**算法 5-2**：$\quad$ 带基线的简单策略梯度算法求解最优策略
***********************

输入：环境（无数学描述）
输出：最优策略的估计 $\pi(\boldsymbol{\theta})$
参数：优化器（隐含学习率 $\left.\alpha^{(\mathbf{w})}, \alpha^{(0)}\right)$ ，折扣因子 $\gamma$ ，控制回合数和回合内步数的参数．

1. （初始化） $\boldsymbol{\theta} \leftarrow$ 任意值， $\mathbf{w} \leftarrow$ 任意值．
2. （回合更新）对每个回合执行以下操作：
 2.1 （采样）用策略 $\pi(\boldsymbol{\theta})$ 生成轨迹 $S_{0}, A_{0}, R_{1}, S_{1}, \ldots, S_{T-1}, A_{T-1}, R_{T}, S_{T}$
 2.2 （初始化回报） $G \leftarrow 0$
 2.3 对 $t=T-1, T-2, \ldots, 0$ ，执行以下步骤：
    1. （更新回报） $G \leftarrow \gamma G+R_{t+1}$
    2. （更新价值）更新 $\mathbf{w}$ 以减小 $\left[G-v\left(S_{t} ; \mathbf{w}\right)\right]^{2}\left(\right.$ 如 $\left.\mathbf{w} \leftarrow \mathbf{w}+\alpha^{(\mathbf{w})}\left[G-v\left(S_{t} ; \mathbf{w}\right)\right] \nabla v\left(S_{t} ; \mathbf{w}\right)\right)$
    3. （更新策略）更新 $\boldsymbol{\theta}$ 以减小 $-\gamma^{t}\left[G-v\left(S_{t} ; \mathbf{w}\right)\right] \ln \pi\left(A_{t}|S_{t} ; \boldsymbol{\theta}\right)\left(\right.$ 如 $\boldsymbol{\theta} \leftarrow \boldsymbol{\theta}+\alpha^{(\theta)} \gamma^{t}\left.\left[G-v\left(S_{t} ; \mathbf{w}\right)\right] \nabla \ln \pi\left(A_{t}|S_{t} ; \boldsymbol{\theta}\right)\right)_{0}$

***********************

接下来，我们来分析什么样的基线函数能最大程度地减小方差．考虑 $\mathrm{E}\left[\gamma^{t}\left(G_{t}-B\left(S_{t}\right)\right)\right.$ $\left.\nabla \ln \pi\left(A_{t}|S_{t} ; \theta\right)\right]$ 的方差为
$$
\mathrm{E}\left[\left[\gamma^{t}\left(G_{t}-B\left(S_{t}\right)\right) \nabla \ln \pi\left(A_{t}|S_{t} ; \theta\right)\right]^{2}\right]-\left[\mathrm{E}\left[\gamma^{t}\left(G_{t}-B\left(S_{t}\right)\right) \nabla \ln \pi\left(A_{t}|S_{t} ; \theta\right)\right]\right]^{2}
$$
其对 $B\left(S_{t}\right)$ 求偏导数为
$$
\mathrm{E}\left[-2 \gamma^{2 t}\left(G_{t}-B\left(S_{t}\right)\right)\left[\nabla \ln \pi\left(A_{t}|S_{t} ; \boldsymbol{\theta}\right)\right]^{2}\right]
$$
（求偏导数时用到了 $\frac{\partial}{\partial B\left(S_{t}\right)} \mathrm{E}\left[\gamma^{t}\left(G_{t}-B\left(S_{t}\right)\right) \nabla \ln \pi\left(A_{t}|S_{t} ; \boldsymbol{\theta}\right)\right]=0$）．令这个偏导数为 0 ，并假设
$$
\mathrm{E}\left[B\left(S_{t}\right)\left[\nabla \ln \pi\left(A_{t}|S_{t} ; \theta\right)\right]^{2}\right]=\mathrm{E}\left[B\left(S_{t}\right)\right] \mathrm{E}\left[\left[\nabla \ln \pi\left(A_{t}|S_{t} ; \theta\right)\right]^{2}\right]
$$
可知
$$
\mathrm{E}\left[B\left(S_{t}\right)\right]=\frac{\mathrm{E}\left[G_{t}\left[\nabla \ln \pi\left(A_{t}|S_{t} ; \theta\right)\right]^{2}\right]}{E\left[\left[\nabla \ln \pi\left(A_{t}|S_{t} ; \theta\right)\right]^{2}\right]}
$$
这意味着，最佳的基线函数应当接近回报 $G_{t}$ 以梯度 $\left[\nabla \ln \pi\left(A_{t}|S_{t} ; \boldsymbol{\theta}\right)\right]^{2}$ 为权重加权平均的结果．但是，在实际应用中，无法事先知道这个值，所以无法使用这样的基线函数．

值得一提的是，当策略参数和价值参数同时需要学习的时候，算法的收敛性需要通过双时间轴 Robbins-Monro 算法（two timescale Robbins-Monro algorithm）来分析．

### 5.3 异策回合更新策略梯度算法

在简单的策略梯度算法的基础上引入重要性采样，可以得到对应的异策算法．记行为策略为 $b(a|s)$ ，有
$$
\begin{array}{c}
\sum_{a} \pi(a|s ; \boldsymbol{\theta}) \gamma^{t} G_{t} \nabla \ln \pi(a|s ; \boldsymbol{\theta}) \\
\qquad=\sum_{\sigma} b(a|s) \frac{\pi(a|s ; \boldsymbol{\theta})}{b(a|s)} \gamma^{t} G_{t} \nabla \ln \pi(a|s ; \boldsymbol{\theta}) \\
=\sum_{a} b(a|s) \frac{1}{b(a|s)} \gamma^{t} G_{t} \nabla \pi(a|s ; \boldsymbol{\theta})
\end{array}
$$
即
$$
E_{\pi(\theta)}\left[\gamma^{t} G_{t} \nabla \ln \pi\left(A_{t}|S_{t} ; \boldsymbol{\theta}\right)\right]=\mathrm{E}_{b}\left[\frac{1}{b\left(A_{t}|S_{t}\right)} \gamma^{t} G_{t} \nabla \pi\left(A_{t}|S_{t} ; \boldsymbol{\theta}\right)\right]
$$
所以，采用重要性采样的离线算法，只需要把用在线策略采样得到的梯度方向 $\gamma^{\prime} G_{t} \nabla \ln \pi\left(A_{t}|S_{t} ; \theta\right)$ 改为用行为策略 $b$ 采样得到的梯度方向 $\frac{1}{b\left(A_{t}|S_{t}\right)} \gamma^{t} G_{t} \nabla \pi\left(A_{t}|S_{t} ; \theta\right)$ 即可．这就意味着，在更新参数 $\boldsymbol{\theta}$ 时可以试图增大 $\frac{1}{b\left(A_{t}|S_{t}\right)} \gamma^{t} G_{t} \pi\left(A_{t}|S_{t} ; \theta\right)$ ．

>**算法5-3**：$\quad$ 重要性采样简单策略梯度求解最优策略
***********************

1. （初始化） $\boldsymbol{\theta \leftarrow}$任意值
2. （回合更新）对每个回合执行以下操作：
 2.1 （行为策略）指定行为策略 $b$ ，使得 $\pi(\boldsymbol{\theta}) \ll b$
 2.2 （采样）用策略 $b$ 生成轨迹：$S_{0}, A_{0}, R_{1}, S_{1}, \ldots, S_{T-1}, A_{T-1}, R_{T}, S_{T}$
 2.3 （初始化回报和权重） $G \leftarrow 0$
 2.4 对 $t=T-1, T-2, \ldots, 0$ ，执行以下步骤：
    1. （更新回报）$G \leftarrow \gamma G+R_{t+1}$
    2. （更新策略）更新参数 $\theta$ 以减小 $-\frac{1}{b\left(A_{t}|S_{t}\right)} \gamma^{t} G_{t} \pi\left(A_{t}|S_{t} ; \theta\right)\left(\right.$ 如 $\boldsymbol{\theta} \leftarrow \boldsymbol{\theta}+\alpha \frac{1}{b\left(A_{t}|S_{t}\right)}\left.\gamma^{t} G \nabla \pi\left(A_{t}|S_{t} ; \boldsymbol{\theta}\right)\right)_{0}$

***********************

重要性采样使得我们可以利用其他策略的样本来更新策略参数，但是可能会带来较大的偏差，算法稳定性比同策算法差．

### 5.4 策略梯度更新和极大似然估计的关系

至此，本章已经介绍了各种各样的策略梯度算法．这些算法在学习的过程中，都是通过更新策略参数 $\theta$ 以试图增大形如 $\mathrm{E}\left[\Psi_{t} \ln \pi\left(A_{t}|S_{t} ; \boldsymbol{\theta}\right)\right]$ 的目标（考虑单个条目则为
$\left.\Psi_{t} \ln \pi\left(A_{t}|S_{t} ; \theta\right)\right)$ ，其中 $\Psi_{t}$ 可取 $G_{0}, G_{t}$ 等值．将这一学习过程与下列有监督学习最大似然问题的过程进行比较，如果已经有一个表达式未知的策略 $\pi$ ，我们要用策略 $\pi(\boldsymbol{\theta})$ 来近似它，这时可以考虑用最大似然的方法来估计策略参数 $\boldsymbol{\theta}$ ．具体而言，如果已经用未知策略 $\pi$ 生成了很多样本，那么这些样本对于策略 $\pi(\boldsymbol{\theta})$ 的对数似然值正比于 $\mathrm{E}\left[\ln \pi\left(A_{t}|S_{t} ; \boldsymbol{\theta}\right)\right]$ ．用这些样本进行有监督学习，需要更新策略参数 $\theta$ 以增大 $\mathrm{E}\left[\ln \pi\left(A_{t}|S_{t} ; \theta\right)\right]$ (考虑单个条目则为 $\left.\ln \pi\left(A_{t}|S_{t} ; \theta\right)\right)$ ．可以看出，$\mathrm{E}\left[\ln \pi\left(A_{t}|S_{t} ; \boldsymbol{\theta}\right)\right]$ 可以通过 $\mathrm{E}\left[\Psi_{t} \ln \pi\left(A_{t}|S_{t} ; \boldsymbol{\theta}\right)\right]$ 中取 $\Psi_{t}=1$ 得到，在形式上具有相似性．策略梯度算法在学习的过程中巧妙地利用观测到的奖励信号决定每步对数似然值 $\ln \pi\left(A_{t}|S_{t} ; \theta\right)$ 对策略奖励的贡献，为其加权 $\Psi_{t}$ (这里的 $\Psi_{t}$ 可能是正数，可能是负数，也可 能是 0 )，使得策略 $\pi(\boldsymbol{\theta})$ 能够变得越来越好．注意，如果取 $\Psi$ ，在整个回合中是不变的（例如 $\left.\Psi_{t}=G_{0}\right)$ ，那么在单一回合中的 $\mathrm{E}\left[G_{0} \ln \pi\left(A_{t}|S_{t} ; \boldsymbol{\theta}\right)\right]=G_{0} \mathrm{E}\left[\ln \pi\left(A_{t}|S_{t} ; \boldsymbol{\theta}\right)\right]$ 就是对整个回合的对数似然值进行加权后对策略的贡献，使得策略 $\pi(\boldsymbol{\theta})$ 能够变得越来越好．试想，如果有的回合表现很好 (比如 $G_{0}$ 是很大的正数 )，在策略梯度更新的时候这个回合的似然值 $\mathrm{E}\left[\ln \pi\left(A_{t}|S_{t} ; \theta\right)\right]$ 就会有一个比较大的权重 $\Psi_{t}\left(\right.$ 例如 $\left.\Psi_{t}=G_{0}\right)$ ，这样这个表现比较好的回合就会更倾向于出现；如果有的回合表现很差（比如 $G_{0}$ 是很小的负数，即绝对值很大的负数）则策略梯度更新时这个回合的似然值就会有比较小的权重，这样这个表现较差的回合就更倾向于不出现．