## 七． 连续动作空间的确定性策略

简单易懂：[b站搬运ShuSenWang教程](https://www.bilibili.com/video/BV1rv41167yx?t=53)

>**如何理解**：两个网络：策略网络与价值函数网络( $q$ 函数 ) ，$t$ 时刻，先利用策略时序差分地更新价值函数，再更新策略网络，策略网络的梯度下降想法是：参数朝着使 $q$ 函数增大的方向走，即 $q$ 函数关于策略网络的参数求梯度，所以最后推得的关系式策略网络的更新式形式为连式法则的样子 $\nabla \mathrm{E}_{\pi(\theta)}\left[G_{0}\right]=\mathrm{E}\left[\sum_{t=0}^{+\infty} \gamma^{t} \nabla \pi\left(S_{t} ; \boldsymbol{\theta}\right)\left[\nabla_{\alpha} q_{\pi(\theta)}\left(S_{t}, a\right)\right]_{a=\pi\left(S_{t} ; \theta\right)}\right]$．

本章介绍在连续动作空间里的确定性执行者 / 评论者算法．在连续的动作空间中，动作的个数是无穷大的．如果采用常规方法，需要计算 $\max _{a} q(s, a ; \boldsymbol{\theta})$ ．而对于无穷多的动作，最大值往往很难求得．为此，D. Silver 等人在文章《 Deterministic Policy Gradient Algorithms 》中提出了确定性策略的方法，来处理连续动作空间情况．本章将针对连续动作空间，推导出确定性策略的策略梯度定理，并据此给出确定性执行者 / 评论者算法．

### 7.1 同策确定性算法

对于连续动作空间里的确定性策略， $\pi(a|s ; \theta)$ 并不是一个通常意义上的函数，它对策略参数 $\theta$ 的梯度． $\nabla \pi(a|s ; \theta)$ 也不复存在．所以，第 6 章介绍的执行者 / 评论者算法就不再适用．幸运的是，曾提到确定性策略可以表示为 $\pi(s ; \boldsymbol{\theta})(s \in \mathcal{S})$ ．这种表示可以绕过由于 $\pi(a|s ; \boldsymbol{\theta})$ 并不是通常意义上的函数而带来的困难．

本节介绍在连续空间中的确定性策略梯度定理，并据此给出基本的同策确定性执行者 / 评论者算法．

#### 7.1.1 策略梯度定理的确定性版本

当策略是一个连续动作空间上的确定性的策略 $\pi(s ; \boldsymbol{\theta})(s \in \mathcal{S})$ 时，策略梯度定理为
$$
\nabla \mathrm{E}_{\pi(\theta)}\left[G_{0}\right]=\mathrm{E}\left[\sum_{t=0}^{+\infty} \gamma^{t} \nabla \pi\left(S_{t} ; \boldsymbol{\theta}\right)\left[\nabla_{\alpha} q_{\pi(\theta)}\left(S_{t}, a\right)\right]_{a=\pi\left(S_{t} ; \theta\right)}\right]
$$
（证明：状态价值和动作价值满足以下关系
$$
\begin{aligned}
v_{\pi(0)}(s) &=q_{\pi(\theta)}(s, \pi(s ; \boldsymbol{\theta})), &  s \in \mathcal{S} \\
q_{\pi(\theta)}(s, \pi(s ; \boldsymbol{\theta})) &=r(s, \pi(s ; \boldsymbol{\theta}))+\gamma \sum_{s^{\prime}} p\left(s^{\prime}|s, \pi(s ; \boldsymbol{\theta})\right) v_{\pi(\theta)}\left(s^{\prime}\right), & s \in \mathcal{S}
\end{aligned}
$$
以上两式对 $\theta$ 求梯度，有
$$
\begin{array}{l}
\nabla v_{\pi(\theta)}(s)=\nabla q_{\pi(\theta)}(s, \pi(s ; \theta)), \quad s \in \mathcal{S} \\
\nabla q_{\pi(\theta)}(s, \pi(s ; \theta))=\left[\nabla_{a} r(s, a)\right]_{a=\pi(s ; \theta)} \nabla \pi(s ; \theta)+ \\
\quad\gamma \sum_{s^{\prime}}\left\{\left[\nabla_{a} p\left(s^{\prime}|s, a\right)\right]_{a=\pi(s ; \theta)}[\nabla \pi(s ; \theta)] v_{\pi(\theta)}\left(s^{\prime}\right)+p\left(s^{\prime}|s, \pi(s ; \theta)\right) \nabla v_{\pi(\theta)}\left(s^{\prime}\right)\right\} \\
\quad=\nabla \pi(s ; \theta)\left[\nabla_{a} r(s, a)+\gamma \sum_{s^{\prime}} \nabla_{a} p\left(s^{\prime}|s, a\right) v_{\pi(\theta)}\left(s^{\prime}\right)\right]_{a=\pi(s ; \theta)}+\gamma \sum_{s^{\prime}} p\left(s^{\prime}|s, \pi(s ; \theta)\right) \nabla v_{\pi(\theta)}\left(s^{\prime}\right) \\
\quad=\nabla \pi(s ; \theta)\left[\nabla_{a} q_{\pi(\theta)}(s, a)\right]_{a=\pi(s ; \theta)}+\gamma \sum_{s^{\prime}} p\left(s^{\prime}|s, \pi(s ; \theta)\right) \nabla v_{\pi(\theta)}\left(s^{\prime}\right), \quad s \in \mathcal{S}
\end{array}
$$
将 $\nabla q_{\pi(\theta)}(s, \pi(s ; \theta))$ 的表达式代人 $\nabla v_{\pi(\theta)}(s)$ 的表达式中，有
$$
\nabla v_{\pi(\theta)}(s)=\nabla \pi(s ; \theta)\left[\nabla_{a} q_{\pi(\theta)}(s, a)\right]_{a=\pi(s ; \theta)}+\gamma \sum_{s^{\prime}} p\left(s^{\prime}|s, \pi(s ; \theta)\right) \nabla v_{\pi(\theta)}\left(s^{\prime}\right), \quad s \in \mathcal{S}
$$
对上式求关于 $S_{t}$ 的期望，并考虑到 $p\left(s^{\prime}|s, \pi(s ; \theta)\right)=\operatorname{\mathbb{P}}\left[S_{t+1}=s^{\prime}|S_{t}=s ; \pi(\theta)\right]$ (其中 $t$ 任取)，有
$$
\begin{array}{l}
E\left[\nabla v_{\pi(\theta)}\left(S_{t}\right)\right] \\
=\sum_{s} \operatorname{\mathbb{P}}\left[S_{t}=s\right] \nabla v_{\pi(\theta)}\left(S_{t}\right) \\
=\sum_{s} \operatorname{\mathbb{P}}\left[S_{t}=s\right]\left[\nabla \pi(s ; \boldsymbol{\theta})\left[\nabla_{a} q_{\pi(\theta)}(s, a)\right]_{a=\pi(s ; \theta)}+\gamma \sum_{s^{\prime}} p\left(s^{\prime}|s, \pi(s ; \theta)\right) \nabla v_{\pi(\theta)}\left(s^{\prime}\right)\right] \\
=\sum_{s} \operatorname{\mathbb{P}}\left[S_{t}=s\right]\left[\nabla \pi(s ; \boldsymbol{\theta})\left[\nabla_{a} q_{\pi(\theta)}(s, a)\right]_{a=\pi(s ; \theta)}+\gamma \sum_{s^{\prime}} \operatorname{\mathbb{P}}\left[S_{t+1}=s^{\prime}|S_{t}=s ; \pi(\boldsymbol{\theta})\right] \nabla v_{\pi(\theta)}\left(s^{\prime}\right)\right] \\
=\sum_{s} \operatorname{\mathbb{P}}\left[S_{t}=s\right] \nabla \pi(s ; \boldsymbol{\theta})\left[\nabla_{a} q_{\pi(\theta)}(s, a)\right]_{a=\pi(s ; \theta)}+\gamma \sum_{s} \operatorname{\mathbb{P}}\left[S_{t}=s\right] \sum_{s^{\prime}} \operatorname{\mathbb{P}}\left[S_{t+1}=s^{\prime}|S_{t}=s ; \pi(\boldsymbol{\theta})\right] \nabla v_{\pi(\theta)}\left(s^{\prime}\right) \\
=\sum_{s} \operatorname{\mathbb{P}}\left[S_{t}=s\right] \nabla \pi(s ; \boldsymbol{\theta})\left[\nabla_{a} q_{\pi(\theta)}(s, a)\right]_{a=\pi(s ; \theta)}+\gamma \sum_{s^{\prime}} \operatorname{\mathbb{P}}\left[S_{t+1}=s^{\prime} ; \pi(\boldsymbol{\theta})\right] \nabla v_{\pi(\theta)}\left(s^{\prime}\right) \\
=\mathrm{E}\left[\nabla \pi(S ; \boldsymbol{\theta})\left[\nabla_{a} q_{\pi(\theta)}(S, a)\right]_{a=\pi(s ; \theta)}\right]+\gamma \mathrm{E}\left[\nabla v_{\pi(\theta)}\left(S_{t+1}\right)\right],
\end{array}
$$
这样就得到了从 $\mathrm{E}\left[\nabla v_{\pi(\theta)}\left(S_{t}\right)\right]$ 到 $\mathrm{E}\left[\nabla v_{\pi(\theta)}\left(S_{t+1}\right)\right]$ 的递推式．注意，最终关注的梯度值就是
$$
\nabla \mathrm{E}_{\pi(\theta)}\left[G_{0}\right]=\mathrm{E}\left[\nabla v_{\pi(\theta)}\left(S_{0}\right)\right]
$$
所以有
$$
\begin{array}{l}
\nabla \mathrm{E}_{\pi(\theta)}\left[G_{0}\right]=\mathrm{E}\left[\nabla v_{\pi(\mathbf{\theta})}\left(S_{0}\right)\right] \\
=\mathrm{E}\left[\nabla \pi\left(S_{0} ; \boldsymbol{\theta}\right)\left[\nabla_{\alpha} q_{\pi(\theta)}\left(S_{0}, a\right)\right]_{a=\pi\left(S_{0} ; \theta\right)}\right]+\gamma \mathrm{E}\left[\nabla v_{\pi(\theta)}\left(S_{1}\right)\right] \\
=\mathrm{E}\left[\nabla \pi\left(S_{0} ; \boldsymbol{\theta}\right)\left[\nabla_{a} q_{\pi(\theta)}\left(S_{0}, a\right)\right]_{a=\pi\left(S_{0} ; \theta\right)}\right]+\gamma \mathrm{E}\left[\nabla \pi\left(S_{1} ; \boldsymbol{\theta}\right)\left[\nabla_{0} q_{\pi(\theta)}\left(S_{1}, a\right)\right]_{a=\pi\left(S_{1} ; \theta\right)}\right]+\gamma^{2} \mathrm{E}\left[\nabla v_{\pi(\theta)}\left(S_{2}\right)\right] \\
=\cdots \\
=\sum_{t=0}^{+\infty} \mathrm{E}\left[\gamma^{t} \nabla \pi\left(S_{t} ; \boldsymbol{\theta}\right)\left[\nabla_{a} q_{\pi(\theta)}\left(S_{t}, a\right)\right]_{a=\pi\left(S_{t} ; \theta\right)}\right]
\end{array}
$$
就得到和之前梯度策略定理类似的形式 $_{0}$ )．

对于连续动作空间中的确定性策略，更常使用的是另外一种形式：
$$
\nabla \mathrm{E}_{\pi(\theta)}\left[G_{0}\right]=\mathrm{E}_{S \sim \rho_{\pi(0)}}\left[\nabla \pi(S ; \boldsymbol{\theta})\left[\nabla_{\alpha} q_{\pi(\theta)}(S, a)\right]_{a=\pi(S ; \theta)}\right]
$$
其中的期望是针对折扣的状态分布 (discounted state distribution)
$$
\rho_{\pi}(s)=\int_{s_{0} \in S} p_{s_{0}}\left(s_{0}\right) \sum_{t=0}^{+\infty} \gamma^{t} \operatorname{\mathbb{P}}\left[S_{t}=s|S_{0}=s_{0} ; \boldsymbol{\theta}\right] \mathrm{d} s_{0}
$$
而言的。（证明：
$$
\begin{array}{l}
\nabla \mathrm{E}_{\pi(\theta)}\left[G_{0}\right]=\sum_{t=0}^{+\infty} \mathrm{E}\left[\gamma^{t} \nabla \pi\left(S_{t} ; \boldsymbol{\theta}\right)\left[\nabla_{a} q_{\pi(\theta)}\left(S_{t}, a\right)\right]_{a=\pi\left(S_{t} ; \theta\right)}\right] \\
=\sum_{t=0}^{+\infty} \int_{s} p_{S_{t}}(s) \gamma^{t} \nabla \pi(s ; \boldsymbol{\theta})\left[\nabla_{a} q_{\pi(\theta)}(s, a)\right]_{a=\pi\left(S_{t} ; \theta\right)} \mathrm{d} s \\
=\sum_{t=0}^{+\infty} \int_{s}\left(\int_{s_{0}} p_{s_{0}}\left(s_{0}\right) \operatorname{\mathbb{P}}\left[S_{t}=s|S_{0}=s_{0} ; \boldsymbol{\theta}\right] \mathrm{d} s_{0}\right) \gamma^{t} \nabla \pi(s ; \boldsymbol{\theta})\left[\nabla_{a} q_{\pi(\theta)}(s, a)\right]_{a=\pi(s ; \theta)} \mathrm{d} s \\
=\int_{s}\left(\int_{s_{0}} p_{s_{0}}\left(s_{0}\right) \sum_{t=0}^{+\infty} \gamma^{t} \operatorname{\mathbb{P}}\left[S_{t}=s|S_{0}=s_{0} ; \boldsymbol{\theta}\right] \mathrm{d} s_{0}\right) \nabla \pi\left(S_{t} ; \boldsymbol{\theta}\right)\left[\nabla_{a} q_{\pi(\theta)}(s, a)\right]_{a=\pi(s ; \theta)} \mathrm{d} s \\
=\int_{s} \rho_{\pi(\theta)}(s) \nabla \pi(s ; \boldsymbol{\theta})\left[\nabla_{a} q_{\pi(\theta)}(s, a)\right]_{a=\pi(s ; \theta)} \mathrm{d} s \\
=\mathrm{E}_{\rho_{\pi(\theta)}}\left[\nabla \pi(s ; \boldsymbol{\theta})\left[\nabla_{a} q_{\pi(\theta)}(s, a)\right]_{a=\pi(s ; \theta)}\right]
\end{array}
$$
得证．）

#### 7.1.2 基本的同策确定性执行者 / 评论者算法

根据策略梯度定理的确定性版本，对于连续动作空间中的确定性执行者 / 评论者算法，梯度的方向变为
$$
\mathrm{E}\left[\sum_{t=0}^{+\infty} \gamma^{t} \nabla \pi\left(S_{t} ; \boldsymbol{\theta}\right)\left[\nabla_{a} q_{\pi(\theta)}\left(S_{t}, a\right)\right]_{a=\pi\left(S_{t} ; \theta\right)}\right]
$$
确定性的同策执行者 $/$ 评论者算法还是用 $q(s, a ; \mathbf{w})$ 来近似 $q_{\pi(\theta)}(s, a)$ ．这时， $\gamma^{t} \nabla \pi\left(S_{t} ; \theta\right)$ $\left[\nabla_{a} q_{\pi(\theta)}\left(S_{t}, a\right)\right]_{a=\pi\left(S_{t} ; \theta\right)}$ 近似为
$$
\gamma^{t} \nabla \pi\left(S_{t} ; \boldsymbol{\theta}\right)\left[\nabla_{a} q\left(S_{t}, a ; \mathbf{w}\right)\right]_{a=\pi\left(S_{t} ; \theta\right)}=\nabla_{\theta}\left[\gamma^{t} q\left(S_{t}, \pi\left(S_{t} ; \theta\right) ; \mathbf{w}\right)\right]
$$
所以，与随机版本的同策确定性执行者 / 评论者算法相比，确定性同策执行者 / 评论者算法在更新策略参数 $\boldsymbol{\theta}$ 时试图减小 $-\gamma^{t} q\left(S_{t}, \pi\left(S_{t} ; \boldsymbol{\theta}\right) ; \mathbf{w}\right)$ ．迭代式可以是
$$
\boldsymbol{\theta} \leftarrow \boldsymbol{\theta}+\gamma^{t} \nabla \pi\left(S_{t} ; \boldsymbol{\theta}\right)\left[\nabla_{a} q\left(S_{t}, a ; \mathbf{w}\right)\right]_{a=\pi\left(S_{t} ; \theta\right)}
$$
算法 7-1 给出了基本的同策确定性执行者 / 评论者算法．对于同策的算法，必须进行探索．连续性动作空间的确定性算法将每个状态都映射到一个确定的动作上，需要在动作空间添加扰动实现探索．具体而言，在状态 $S_{t}$ 下确定性策略 $\pi(\boldsymbol{\theta})$ 指定的动作为 $\pi\left(S_{t} ; \theta\right)$ ，则在同策算法中使用的动作可以具有 $\pi\left(S_{t} ; \boldsymbol{\theta}\right)+N_{t}$ 的形式，其中 $N_{t}$ 是扰动量．在动作空间无界的情况下（即没有限制动作有最大值和最小值)，常常假设扰动量 $N_{t}$ 满足正态分布．在动作空间有界的情况下，可以用 clip 函数进一步限制加扰动后的范围（如 $\operatorname{clip}\left(\pi\left(S_{t} ; \theta\right)+N_{t}, A_{\text {low }}, A_{\text {tigh }}\right)$ ，其中 $A_{\text {low }}$ 和 $A_{\text {high }}$ 是动作的最小取值和最大取值)，或用 sigmoid 函数将对加扰动后的动作变换到合适的区间里 $\left(\right.$ 如 $\left.A_{\text {low }}+\left(A_{\text {righ }}-A_{\text {low }}\right) \operatorname{expit}\left(\pi\left(S_{t} ; \theta\right)+N_{t}\right)\right)$．

>**算法 7-1**：$\quad$ 基本的同策确定性执行者 / 评论者算法
***********************

输入: 环境（无数学描述）
输出：最优策略的估计 $\pi(\boldsymbol{\theta})$
参数：学习率 $\alpha^{(\mathrm{w})}, \alpha^{(0)}$ ，折扣因子 $\gamma$ ，控制回合数和回合内步数的参数．

1. （初始化） $\boldsymbol{\theta \leftarrow}$ 任意值，$W \leftarrow$任意值
2. （带自益的策略更新）对每个回合执行以下操作：
 2.1 （初始化累积折扣） $I \leftarrow 1$
 2.2 （初始化状态动作对）选择状态 $S$ ，对 $\pi(S ; \theta)$ 加扰动进而确定动作 $A$ (如用正态分布随机变量扰动)
 2.3 如果回合未结束，执行以下操作：
    1. （采样）根据状态 $S$ 和动作 $A$ 得到采样 $R$ 和下一状态 $S^{\prime}$
    2. （执行）对 $\pi\left(S^{\prime} ; \theta\right)$ 加扰动进而确定动作 $A^{\prime}$
    3. （估计回报） $U \leftarrow R+\gamma q\left(S^{\prime}, A^{\prime} ; \mathbf{w}\right)$
    4. （更新价值）更新 $\mathbf{w}$ 以减小 $[U-q(S, A ; \mathbf{w})]^{2} （$ 如 $\left.\mathbf{w} \leftarrow \mathbf{w}+\alpha^{(\mathbf{w})}[U-q(S, A ; \mathbf{w})] \nabla q(S, A ; \mathbf{w})\right)$
    5. （策略改进）更新 $\theta$ 以减小 $-I q(S, \pi(S ; \theta) ; \mathbf{w}) （$ 如 $\left.\boldsymbol{\theta} \leftarrow \boldsymbol{\theta}+\alpha^{(\theta)} I \nabla \pi(S ; \boldsymbol{\theta})\left[\nabla_{a} q(S, a ; \mathbf{w})\right]_{a=\pi(s ; \theta)}\right)$
    6. （更新累积折扣）$I\leftarrow\gamma I$
    7. （更新状态） $S \leftarrow S^{\prime}, A \leftarrow A^{\prime}$．

***********************

在有些任务中，动作的效果经过低通滤波器处理后反映在系统中，而独立同分布的 Gaussian 噪声不能有效实现探索．例如，在某个任务中，动作的直接效果是改变一个质点的加速度．如果在这个任务中用独立同分布的 Gaussian 噪声叠加在动作上，那么对质点位置的整体效果是在没有噪声的位置附近移动．这样的探索就没有办法为质点的位置提供持续的偏移，使得质点到比较远的位置．在这类任务中，常常用 Ornstein Uhlenbeck 过 程作为动作噪声．Ornstein Uhlenbeck 过程是用下列随机微分方程定义的 (以一维的情况为例 )
$$
\mathrm{d} N_{t}=\theta\left(\mu-N_{t}\right) \mathrm{d} t+\sigma \mathrm{d} B_{t}
$$
其中 $\theta, \mu, \sigma$ 是参数 $(\theta>0, \sigma>0), B_{t}$ 是标准 Brownian 运动．当初始扰动是在原点的单点分布（即限定 $N_{0}=0$ ），并且 $\mu=0$ 时，上述方程的解为
$$
N_{t}=\sigma \int_{0}^{t} e^{\theta(\tau-t)} \mathrm{d} B_{t}
$$
（证明：将 $\mathrm{d} N_{t}=\theta\left(\mu-N_{t}\right) \mathrm{d} t+\sigma \mathrm{d} B_{t}$ 代入 $\mathrm{d}\left(N_{t} e^{\theta t}\right)=\theta N_{t} e^{\theta t} \mathrm{~d} t+e^{\theta t} \mathrm{~d} N_{t}, \quad$ 化简可得 $\mathrm{d}\left(N_{t} e^{\theta t}\right)=\mu \theta e^{\theta t} \mathrm{~d} t+\sigma e^{\theta t} \mathrm{~d} B_{t}$ ．将此式从 0 积到 $t$ ，得 $N_{t} e^{\theta t}-N_{0}=\mu\left(e^{\theta t}-1\right)+\sigma \int_{0}^{t} e^{\theta \tau} \mathrm{d} B_{\tau}$ ．当 $N_{0}=0$ 且 $\mu=0$
时化简可得结果）．

这个解的均值为 0 ，方差为 $\frac{\sigma^{2}}{2 \theta}\left(1-e^{-2 \theta t}\right)$ ，协方差为
$$
\operatorname{Cov}\left(N_{t}, N_{s}\right)=\frac{\sigma^{2}}{2 \theta}\left(e^{-\theta|t-s|}-e^{-\theta(t+s)}\right)
$$
（证明：由于均值为 0 ，所以 $\operatorname{Cov}\left(N_{t}, N_{s}\right)=\mathrm{E}\left[N_{t} N_{s}\right]=\sigma^{2} e^{-\theta(t+s)} \mathrm{E}\left[\int_{0}^{t} e^{\theta \tau} \mathrm{d} B_{t} \int_{0}^{s} e^{\theta \tau} \mathrm{d} B_{t}\right]$ ．另外，Ito Isometry 告诉我们 $\mathrm{E}\left[\int_{0}^{t} e^{\theta \tau} \mathrm{d} B_{\tau} \int_{0}^{s} e^{\theta \tau} \mathrm{d} B_{\tau}\right]=\mathrm{E}\left[\int_{0}^{\min (t, s)} e^{2 \theta \tau} \mathrm{d} \tau\right]$ ，所以 $\operatorname{Cov}\left(N_{t}, N_{s}\right)=\sigma^{2} e^{-\theta(t+s)}$
$\int_{0}^{\min (t, s)} e^{2 \theta \tau} \mathrm{d} \tau$ ，进一步化简可得结果．）

对于 $t \neq s$ 总有 $|t-s|<t+s$ ，所以 $\operatorname{Cov}\left(N_{t}, N_{s}\right)>0$ ．据此可知，使用 Ornstein Uhlenbeck 过程让相邻扰动正相关，进而让动作向相近的方向偏移．

### 7.2 异策确定性算法

对于连续的动作空间，我们希望能够找到一个确定性策略，使得每条轨迹的回报最大．同策确定性算法利用策略 $\pi(\boldsymbol{\theta})$ 生成轨迹，并在这些轨迹上求得回报的平均值，通过让平均回报最大，使得每条轨迹上的回报尽可能大．事实上，如果每条轨迹的回报都要最大，那么对于任意策略 $b$ 采样得到的轨迹，我们都希望在这套轨迹上的平均回报最大．所以异策确定性策略算法引入确定性行为策略 $b$ ，将这个平均改为针对策略 $b$ 采样得到的轨迹，得到异策确定性梯度为
$$
\nabla \mathrm{E}_{\rho_{b}}\left[q_{\pi(\theta)}(S, \pi(S ; \boldsymbol{\theta}))\right]=\mathrm{E}_{\rho_{b}}\left[\nabla \pi(S ; \boldsymbol{\theta})\left[\nabla_{a} q_{\pi(\theta)}(S, a)\right]_{a=\pi(S ; \theta)}\right]
$$
这个表达式与同策的情形相比，期望运算针对的表达式相同．所以，异策确定性算法的迭代式与同策确定性算法的迭代式相同．

异策确定性算法可能比同策确定性算法性能好的原因在于，行为策略可能会促进探索，用行为策略采样得到的轨迹能够更加全面的探索轨迹空间．这时候，最大化对轨迹分布的平均期望时能够同时考虑到更多不同的轨迹，使得在整个轨迹空间上的所有轨迹的回报会更大．

#### 7.2.1 基本的异策确定性执行者 / 评论者算法

基于上述分析，我们可以得到**异策确定性执行者 / 评论者算法** (Off-Policy Deterministic Actor-Critic, $\mathrm{OPDAC}$ )，见算法 7-2 ．

值得一提的是，虽然异策算法和同策算法有相同形式的迭代式，但是在算法结构上并不完全相同．在同策算法迭代更新时，目标策略的动作可以在运行过程中直接得到；但是在异策算法迭代更新策略参数时，对环境使用的是行为策略决定的动作，而不是目标策略决定的动作，所以需要额外计算目标策略的动作．在更新价值函数时，采用的是 $\mathrm{Q}$ 学习，依然需要计算目标策略的动作．

>**算法 9-2**：$\quad$ 基本的异策确定性执行者 / 评论者算法

输入：环境（无数学描述）
输出：最优策略的估计 $\pi(\boldsymbol{\theta})$
参数：学习率 $\alpha^{(\mathbf{w})}, \alpha^{(0)}$ ，折扣因子 $\gamma$ ，控制回合数和回合内步数的参数．

1. （初始化） $\boldsymbol{\theta} \leftarrow$ 任意值， $\mathbf{w} \leftarrow$ 任意值．
2. （带自益的策略更新）对每个回合执行以下操作：
 2.1 （初始化累积折扣） $I \leftarrow 1$
 2.2 （初始化状态）选择状态 $S$
 2.3 如果回合未结束，执行以下操作：
    1. （执行）用 $b(\cdot|S)$ 得到动作 $A$
    2. （采样）根据状态 $S$ 和动作 $A$ 得到采样 $R$ 和下一状态 $S^{\prime}$
    3. （估计回报） $U \leftarrow R+\gamma q\left(S^{\prime}, \pi\left(S^{\prime} ; \boldsymbol{\theta}\right) ; \mathbf{w}\right)$
    4. （更新价值）更新 $\mathbf{w}$ 以减小 $[U-q(S, A ; \mathbf{w})]^{2}($ 如 $\left.\mathbf{w} \leftarrow \mathbf{w}+\alpha^{(\mathbf{w})}[U-q(S, A ; \mathbf{w})] \nabla q(S, A ; \mathbf{w})\right)$
    5. （策略改进）更新 $\boldsymbol{\theta}$ 以减小 $-\operatorname{Iq}(S, \pi(S ; \boldsymbol{\theta}) ; \mathbf{w})$ (如$\left.\boldsymbol{\theta} \leftarrow \boldsymbol{\theta}+\alpha^{(\theta)} I \nabla \pi(S ; \boldsymbol{\theta})\left[\nabla_{a} q(S, a ; \mathbf{w})\right]_{a=\pi(s ; \theta)}\right)$
    6. （更新累积折扣） $I \leftarrow \gamma I$
    7. （更新状态） $S \leftarrow S_{0}^{\prime}$．

***********************

#### 7.2.2 深度确定性策略梯度算法

**深度确定性策略梯度算法**（Deep Deterministic Policy Gradient, $\mathrm{DDPG}$）将基本的异策 确定性执行者 / 评论者算法和深度 Q 网络中常用的技术结合．具体而言，确定性深度策略梯度算法用到了以下技术．

- 经验回放：执行者得到的经验 $\left(S, A, R, S^{\prime}\right)$ 收集后放在一个存储空间中，等更新参数时批量回放，用批处理更新．
- 目标网络：在常规价值参数 $\mathbf{w}$ 和策略参数 $\boldsymbol{\theta}$ 外再使用一套用于估计目标的目标价值参数 $\mathbf{w}_{\text {目标 }}$ 和目标策略参数 $\boldsymbol{\theta}_{\text {目标。 }}$ 在更新目标网络时，为了避免参数更新过快，还引入了目标网络的学习率 $\alpha_{\text {目标 }} \in(0,1)$

算法 7-3 给出了深度确定性策略梯度算法．

>**算法 7-3**：$\quad$ 深度确定性策略梯度算法 (假设 $\pi(S ; \theta)+N$ 总是在动作空间内)
***********************

输入：环境（无数学描述）
输出：最优策略的估计 $\pi(\boldsymbol{\theta})$
参数：学习率 $\alpha^{(\mathbf{w})}, \alpha^{(\theta)}$ ，折扣因子 $\gamma$ ，控制回合数和回合内步数的参数，目标网络学习率 $\alpha_{\text {目标}}$

1. （初始化） $\boldsymbol{\theta \leftarrow}$ 任意值， $\boldsymbol{\theta}_{\text {目标 }} \leftarrow \boldsymbol{\theta}, \mathbf{w} \leftarrow$ 任意值， $\mathbf{w}_{\text {目标 }} \leftarrow \mathbf{w}$
2. 循环执行以下操作：
 2.1 （累积经验）从起始状态 $S$ 出发，执行以下操作，直到满足终止条件：
    - 用对 $\pi(S ; \boldsymbol{\theta})$ 加扰动进而确定动作 $A$ (如用正态分布随机变量扰动)
    - 执行动作 $A$ ，观测到收益 $R$ 和下一状态 $S^{\prime}$
    - 将经验 $\left(S, A, R, S^{\prime}\right)$ 存储在经验存储空间 $\mathcal{D}$
2.2 （更新）在更新的时机，执行一次或多次以下更新操作：
    - （回放）从存储空间 $\mathcal{D}$ 采样出一批经验 $\mathcal{B}$
    - （估计回报）为经验估计回报 $U \leftarrow R+\gamma q\left(S^{\prime}, \pi\left(S^{\prime} ; \theta_{\text {目标 }}\right) ; \mathbf{w}_{\text {目标 }}\right)\left(\left(S, A, R, S^{\prime}\right) \in \mathcal{B}\right)$
    - （价值更新)更新 $\mathbf{w}$ 以减小 $\frac{1}{|\mathcal{B}|} \sum_{\left(S, A, R, S^{\prime}\right) \in \mathcal{B}}[U-q(S, A ; \mathbf{w})]^{2}$
    - （策略更新）更新 $\boldsymbol{\theta}$ 以减小 $-\frac{1}{|\mathcal{B}|} \sum_{\left(s, A, R, S^{\prime}\right) \in \mathcal{B}} q(S, \pi(S ; \boldsymbol{\theta}) ; \mathbf{w})$ (如 $\left.\boldsymbol{\theta} \leftarrow \boldsymbol{\theta}+\alpha^{(\theta)} \frac{1}{|\mathcal{B}|} \sum_{\left(S, A, R, S^{\prime}\right) \in \mathcal{B}} \nabla \pi(S ; \boldsymbol{\theta})\left[\nabla_{a} q(S, a ; \mathbf{w})\right]_{a=\pi(S ; \theta)}\right)$
    - （更新目标）在恰当的时机更新目标网络和目标策略， $\mathbf{w}_{\text {目标 }} \leftarrow\left(1-\alpha_{\text {目标 }}\right) \mathbf{w}_{\text {目标 }}+$ $\alpha_{\text {目标 }} \mathbf{w}, \quad \boldsymbol{\theta}_{\text {目标 }} \leftarrow\left(1-\alpha_{\text {目标 }}\right) \boldsymbol{\theta}_{\text {目标 }}+\alpha_{\text {目标 }} \boldsymbol{\theta}$．

***********************

#### 7.2.3 双重延迟深度确定性策略梯度算法

S. Fujimoto 等人在文章 《 Addressing function approximation error in actor-critic methods》 中给出了**双重延迟深度确定性策略梯度算法**（Twin Delay Deep Deterministic Policy Gradient, TD3 )，结合了深度确定性策略梯度算法和双重 Q 学习．

回顾前文，双重 $\mathrm{Q}$ 学习可以消除最大偏差．基于查找表的双重 Q 学习用了两套动作价值函数 $q^{(0)}(s, a)$ 和 $q^{(1)}(s, a)(s \in \mathcal{S}, a \in \mathcal{A})$ ，其中一套动作价值函数用来计算最优动作（如 $\left.A^{\prime}=\arg \max _{a} q^{(0)}\left(S^{\prime}, a\right)\right)$ ，另外一套价值函数用来估计回报（如 $\left.q^{(1)}\left(S^{\prime}, A^{\prime}\right)\right)$ ；双重 $\mathrm{Q}$ 网络则考虑到有了目标网络后已经有了两套价值函数的参数 $\mathbf{w}$ 和 $\mathbf{w}_{\text {目标， }}$ 所以用其中一套参数 $\mathbf{w}$ 计算最优动作（如 $A^{\prime}=\arg \max _{a} q\left(S^{\prime}, a ; \mathbf{w}\right)$ ），再用目标网络的参数 $\mathbf{w}_{\text {目标 }}$ 估计目标 （如 $q\left(S^{\prime}, A^{\prime} ; \mathbf{w}_{\text {目标 }}\right)$ ）．
但是对于确定性策略梯度算法，动作已经由含参策略 $\pi(\boldsymbol{\theta})$ 决定了 (如 $\pi\left(S^{\prime} ; \boldsymbol{\theta}\right)$ )，双重网络则要由双重延迟深度确定性策略梯度算法维护两份学习过程的价值网络参数 $\mathbf{w}^{(i)}$ 和目标网络参数 $\mathbf{w}_{\text {目标 }}^{(i)}(i=0,1)$ ．在估计目标时，选取两个目标网络得到的结果中较小的那个，即 $\min _{i=0,1} q\left(\cdot, ; ; \mathbf{w}_{\text {目标 }}^{(i)}\right)$．

算法 7-4 给出了双重延迟深度确定性策略梯度算法．

>**算法 7-4** ：$\quad$双重延迟深度确定性策略梯度
***********************

输入: 环境（无数学描述）
榆出：最优策略的估计 $\pi(\boldsymbol{\theta})$
参数：学习率 $\alpha^{(\mathrm{w})}, \alpha^{(\theta)}$ ，折扣因子 $\gamma$ ，控制回合数和回合内步数的参数，目标网络学习率 $\alpha_{\text {目标 }}$．

1. （初始化） $\boldsymbol{\theta} \leftarrow$ 任意值， $\boldsymbol{\theta}_{\text {目标 }} \leftarrow \boldsymbol{\theta}, \mathbf{w}^{(i)} \leftarrow$ 任意值， $\mathbf{w}_{\text {目标 }}^{(i)} \leftarrow \mathbf{w}^{(i)}, i \in\{0,1\}$
2. 循环执行以下操作：
 2.1 （累积经验）从起始状态 $S$ 出发，执行以下操作，直到满足终止条件：
    - 用对 $\pi(S ; \boldsymbol{\theta})$ 加扰动进而确定动作 $A$ (如用正态分布随机变量扰动)
    - 执行动作 $A$ ，观测到收益 $R$ 和下一状态 $S^{\prime}$
    - 将经验 $\left(S, A, R, S^{\prime}\right)$ 存储在经验存储空间 $\mathcal{D}$
 2.2 （更新）一次或多次执行以下操作：
    - （回放）从存储空间 $\mathcal{D}$ 采样出一批经验 $\mathcal{B}$
    - （扰动动作）为目标动作 $\pi\left(S^{\prime} ; \theta_{\text {目标 }}\right)$ 加受限的扰动，得到动作 $A^{\prime}\left(\left(S, A, R, S^{\prime}\right) \in \mathcal{B}\right)$
    - （估计回报）为经验估计回报 $U=R+\gamma \min _{i=0,1} q\left(S^{\prime}, A^{\prime} ; w^{(i)}\right)\left(\left(S, A, R, S^{\prime}\right) \in \mathcal{B}\right)$
    - （价值更新）更新 $\mathbf{w}^{(i)}$ 以减小 $\frac{1}{|\mathcal{B}|} \sum_{\left(S, A, R, S^{\prime}\right) \in \mathcal{B}}\left[U-q\left(S, A ; \mathbf{w}^{(i)}\right)\right]^{2}(i=0,1)$
    - （策略更新）在恰当的时机，更新 $\theta$ 以减小 $-\frac{1}{|\mathcal{B}|_{\left(S, A, R, S^{\prime}\right) \in \mathcal{B}}} q\left(S, \pi(S ; \boldsymbol{\theta}) ; \mathbf{w}^{(0)}\right)($ 如 $\boldsymbol{\theta} \leftarrow \left.\boldsymbol{\theta}+\alpha^{(0)} \frac{1}{|\mathcal{B}|} \sum_{\left(S, A, R, S^{\prime}\right) \in \mathcal{B}} \nabla \pi(S ; \boldsymbol{\theta})\left[\nabla_{a} q\left(S, a ; \mathbf{w}^{(0)}\right)\right]_{a=\pi(s ; \theta)}\right)$
    - （更新目标）在恰当的时机，更新目标网络和目标策略， $\mathbf{w}_{\text {目标 }}^{(i)} \leftarrow\left(1-\alpha_{\text {目标 }}\right)\mathbf{w}_{\text {甘标 }}^{(i)}+\alpha_{\text {目标 }} \mathbf{W}^{(i)}(i=0,1), \quad \boldsymbol{\theta}_{\text {目标 }} \leftarrow\left(1-\alpha_{\text {目标 }}\right) \boldsymbol{\theta}_{\text {目标 }}+\alpha_{\text {目标 }} \boldsymbol{\theta}$．

***********************

TODO:AlphaZero算法
