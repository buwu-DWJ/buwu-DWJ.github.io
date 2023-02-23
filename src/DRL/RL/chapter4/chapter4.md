## 四． 函数近似方法

第 $3 \sim 5$ 章中介绍的有模型数值迭代算法、回合更新算法和时序差分更新算法，在每次更新价值函数时都只更新某个状态（或状态动作对）下的价值估计．但是，在有些任务中，状态和动作的数目非常大，甚至可能是无穷大，这时，不可能对所有的状态 (或状态动作对) 逐一进行更新．函数近似方法用参数化的模型来近似整个状态价值函数（或动作价值函数），并在每次学习时更新整个函数．这样，那些没有被访问过的状态（或状态动作对）的价值估计也能得到更新．本章将介绍函数近似方法的一般理论，包括策略评估和最优策略求解的一般理论．再介绍两种最常见的近似函数：线性函数和人工神经网络．后者将深度学习和强化学习相结合，称为深度 Q 学习，是第一个深度强化学习算法，也是目前的热门算法．

### 4.1 函数近似原理

本节介绍用**函数近似**（function approximation）方法来估计给定策略 $\pi$ 的状态价值函数 $v_{\pi}$ 或动作价值函数 $q_{\pi}$ ．要评估状态价值，我们可以用一个参数为 $\mathbf{w}$ 的函数 $v(s ; \mathbf{w})$ $(s \in \mathcal{S})$ 来近似状态价值；要评估动作价值，我们可以用一个参数为 $\mathbf{w}$ 的函数 $q(s, a ; \mathbf{w})$ $(s \in \mathcal{S}, a \in \mathcal{A}(s))$ 来近似动作价值．在动作集 $\mathcal{A}$ 有限的情况下，还可以用一个矢量函数一个动作，而整个矢量函数除参数外只用状态作为输人．这里的函数 $v(s ; \mathbf{w})(s \in \mathcal{S})$ 、 $q(s, a ; w)(s \in \mathcal{S}, a \in \mathcal{A}(s)), \mathbf{q}(s ; \mathbf{w})(s \in \mathcal{S})$ 形式不限，可以是线性函数，也可以是神经网络．但是，它们的形式要事先给定，在学习过程中只更新参数 $\mathbf{w}$ ．一旦参数 $\mathbf{w}$ 完全确定，价值估计就完全给定．所以，本节将介绍如何更新参数 $\mathbf{w}$ ．更新参数的方法既可以用于策略价值评估，也可以用于最优策略求解．

#### 4.1.1 随机梯度下降

本节来看同策回合更新价值估计．将同策回合更新价值估计与函数近似方法相结合，可以得到函数近似回合更新价值估计算法（算法 4-1 ）．这个算法与第 2 章中回合更新算法的区别就是在价值更新时更新的对象是函数参数，而不是每个状态或状态动作对的价值估计．

>**算法 6-1**： $\quad$ 随机梯度下降函数近似评估策略的价值
***********************

1. （初始化）任意初始化参数 $\mathbf{w}$
2. 逐回合执行以下操作
 2.1 （采样）用环境和策略 $\pi$ 生成轨迹样本 $S_{0}, A_{0}, R_{1}, S_{1}, A_{1}, R_{2}, \ldots, S_{T-1}, A_{T-1}, R_{T}, S_{T}$
 2.2 （初始化回报） $G \leftarrow 0$
 2.3 （逐步更新）对 $t \leftarrow T-1, T-2, \ldots, 0$ ，执行以下步骤
    1. （更新回报） $G \leftarrow \gamma G+R_{t+1}$
    2. （更新价值）若评估的是动作价值则更新 $\mathbf{w}$ 以减小 $\left[G-q\left(S_{t}, A_{t} ; \mathbf{w}\right)\right]^{2}$ (如 $\left.\mathbf{w} \leftarrow \mathbf{w}+\alpha\left[G-q\left(S_{t}, A_{t} ; \mathbf{w}\right)\right] \nabla q\left(S_{t}, A_{t} ; \mathbf{w}\right)\right) ;$ 若评估的是状态价值则更新 $\mathbf{w}$ 以减小
 $\left[G-v\left(S_{t} ; \mathbf{w}\right)\right]^{2}\left(\text { 如 } \mathbf{w} \leftarrow \mathbf{w}+\alpha\left[G-v\left(S_{t} ; \mathbf{w}\right)\right] \nabla v\left(S_{t} ; \mathbf{w}\right)\right)$．

***********************

如果我们用算法 4-1 评估动作价值，则更新参数时应当试图减小每一步的回报估计 $G$ 和动作价值估计 $q\left(S_{t}, A_{t} ; \mathbf{w}\right)$ 的差别．所以，可以定义每一步损失为 $\left[G_{t}-q\left(S_{t}, A_{t} ; \mathbf{w}\right)\right]^{2}$ ，而整个回合的损失为 $\sum_{t=0}^{T-1}\left[G_{t}-q\left(S_{t}, A_{t} ; \mathbf{w}\right)\right]^{2}$ ．如果我们沿着 $\sum_{t=0}^{T-1}\left[G_{t}-q\left(S_{t}, A_{t} ; \mathbf{w}\right)\right]^{2}$ 对 $\mathbf{w}$ 的梯度的反方向更新策略参数 $\mathbf{w}$ ，就有机会减小损失．这样的方法称为随机梯度下降（ stochastic gradient-descent, SGD ）算法．对于能支持自动梯度计算的软件包，往往自带根据损失函数更新参数的功能．如果不使用现成的参数更新软件包，也可以自己计算得到 $q\left(S_{t}, A_{t} ; \mathbf{w}\right)$ 的
梯度 $\nabla q\left(S_{t}, A_{t} ; \mathbf{w}\right)$ ，然后利用下式进行更新 :
$$
\mathbf{w} \leftarrow \mathbf{w}-\frac{1}{2} \alpha_{t} \nabla\left[G_{t}-q\left(S_{t}, A_{t} ; \mathbf{w}\right)\right]^{2}=\mathbf{w}+\alpha_{t}\left[G_{t}-q\left(S_{t}, A_t ; \mathbf{w}\right)\right] \nabla q\left(S_{t}, A_{t} ; \mathbf{w}\right)
$$
对于状态价值函数，也有类似的分析．定义每一步的损失为 $\left[G_{t}-v\left(S_{t} ; \mathbf{w}\right)\right]^{2}$ ，整个回合的损失为 $\sum_{t=0}^{T}\left[G_{t}-v\left(S_{t} ; \mathbf{w}\right)\right]^{2}$ ．可以在自动梯度计算并更新参数的软件包中定义这个损失来更新参数 $\mathbf{w}$, 也可以用下式更新：
$$
\mathbf{w} \leftarrow \mathbf{w}-\frac{1}{2} \alpha_{t} \nabla\left[G_{t}-v\left(S_{t} ; \mathbf{w}\right)\right]^{2}=\mathbf{w}+\alpha_{t}\left[G_{t}-v\left(S_{t} ; \mathbf{w}\right)\right] \nabla v\left(S_{t} ; \mathbf{w}\right)
$$
相应的回合更新策略评估算法与算法 4-1 类似，此处从略．

将策略改进引入随机梯度下降评估策略，就能实现随机梯度下降最优策略求解．算法 4-2 给出了随机梯度下降最优策略求解的算法．它与第 2 章回合更新最优策略求解算法的区别也仅仅在于迭代的过程中不是直接修改价值估计，而是更新价值参数 $\mathbf{w}$ ．

>**算法 4-2**：$\quad$ 随机梯度下降求最优策略
***********************

1. （初始化）任意初始化参数 $\mathbf{w}$
2. 逐回合执行以下操作
 2.1 （采样）用环境和当前动作价值估计 $q(\cdot, \cdot ; \mathbf{w})$ 导出的策略（如 $\varepsilon$ 柔性策略）生成轨迹样
本 $S_{0}, A_{0}, R_{1}, S_{1}, A_{1}, R_{2}, \ldots, S_{T-1}, A_{T-1}, R_{T}, S_{T}$
 2.2 （初始化回报） $G \leftarrow 0$
 2.3 （逐步更新）对 $t \leftarrow T-1, T-2, \ldots, 0$ ，执行以下步骤：
    1. （更新回报） $G \leftarrow \gamma G+R_{t+1}$
    2. （更新动作价值函数）更新参数 $\mathbf{w}$ 以减小 $\left[G-q\left(S_{t}, A_{t} ; \mathbf{w}\right)\right]^{2}($ 如 $\mathbf{w} \leftarrow \mathbf{w}+\alpha[G-$
$\left.\left.q\left(S_{t}, A_{t} ; \mathbf{w}\right)\right] \nabla q\left(S_{t}, A_{t} ; \mathbf{w}\right)\right)$．

***********************

#### 4.1.2 半梯度下降

动态规划和时序差分学习都用了“自益”来估计回报，回报的估计值与 $\mathbf{w}$ 有关，是存在偏差的．例如，对于单步更新时序差分估计的动作价值函数，回报的估计为 $U_{t}=R_{t+1}+\gamma q\left(S_{t+1}, A_{t+1} ; \mathbf{w}\right)$ ，而动作价值的估计为 $q\left(S_{t}, A_{t} ; \mathbf{w}\right)$ ，这两个估计都与权重 $\mathbf{w}$ 有关．在试图减小每一步的回报估计 $U_{t}$ 和动作价值估计 $q\left(S_{t}, A_{t} ; \mathbf{w}\right)$ 的差别时，可以定义每一步损失为 $\left[U_{t}-q\left(S_{t}, A_{t} ; \mathbf{w}\right)\right]^{2}$ ，而整个回合的损失为 $\sum_{t=0}^{T-1}\left[U_{t}-q\left(S_{t}, A_{t} ; \mathbf{w}\right)\right]^{2}$ ．在更新参数 $\mathbf{w}$ 以减小损失时，应当注意不对回报的估计 $U_{t}=R_{t+1}+\gamma q\left(S_{t+1}, A_{t+1} ; \mathbf{w}\right)$ 求梯度，只对动作价值的估计 $q\left(S_{t}, A_{t} ; \mathbf{w}\right)$ 求关于 $\mathbf{w}$ 的梯度，这就是**半梯度下降**（semi-gradient descent）算法．半梯度下降算法同样既可以用于策略评估，也可以用于求解最优策略（见算法 4-3 和算法 4-4 ）．

>**算法 4-3**：$\quad$ 半梯度下降算法估计动作价值或 $\mathrm{SARSA}$ 算法求最优策略
***********************

1. （初始化）任意初始化参数 $\mathbf{w}$
2. 逐回合执行以下操作
 2.1 （初始化状态动作对）选择状态 $S$ ．如果是策略评估，则用输入策略 $\pi(\cdot|S)$ 确定动作 $A$ ；如果是寻找最优策略，则用当前动作价值估计 $q(S, \cdot ; \mathbf{w})$ 导出的策略 $($ 如 $\varepsilon$ 柔性策略 $)$ 确定动作 $A$
 2.2 如果回合未结東，执行以下操作：
    1. （采样）执行动作 $A$ ，观测得到奖励 $R$ 和新状态 $S^{\prime}$
    2. 如果是策略评估，则用输入策略 $\pi\left(\cdot|S^{\prime}\right)$ 确定动作 $A^{\prime}$ ；如果是寻找最优策略，则用当前动作价值估计 $q\left(S^{\prime}, ; \mathbf{w}\right)$ 导出的策略 $\left(\right.$ 如 $\varepsilon$ 柔性策略）确定动作 $A_{0}^{\prime}$
    3. （计算回报的估计值） $U \leftarrow R+\gamma q\left(S^{\prime}, A^{\prime} ; \mathbf{w}\right)$
    4. （更新动作价值函数）更新参数 $\mathbf{w}$ 以减小 $[U-q(S, A ; \mathbf{w})]^{2}($ 如 $\mathbf{w} \leftarrow \mathbf{w}+\alpha[U-q(S, A ; \mathbf{w})] \nabla q(S, A ; \mathbf{w}))$ ．注意此步不可以重新计算 $U$
    5. $S\leftarrow S^{\prime}, \quad A \leftarrow A^{\prime}$

***********************

>**算法 4-4**：$\quad$ 半梯度下降估计状态价值或期望 SARSA 算法或 $\mathrm{Q}$ 学习

***********************

1. （初始化）任意初始化参数 $\mathbf{w}$
2. 逐回合执行以下操作
 2.1 （初始化状态）选择状态 $S$
 2.2 如果回合未结束，执行以下操作
    1. 如果是策略评估，则用输入策略 $\pi(\cdot|S)$ 确定动作 $A$ ；如果是寻找最优策略，则用当前动作价值估计 $q(S, \cdot ; \mathbf{w})$ 导出的策略（如 $\varepsilon$ 柔性策略）确定动作 $A$
    2. （采样）执行动作 $A$ ，观测得到奖励 $R$ 和新状态 $S_{0}^{\prime}$
    3. （计算回报的估计值）如果是状态价值评估，则 $U \leftarrow R+\gamma v\left(S^{\prime} ; \mathbf{w}\right)$ ．如果是期望 SARSA 算法，则 $U \leftarrow R+\gamma \sum_{a} \pi\left(a|S^{\prime} ; \mathbf{w}\right) q\left(S^{\prime}, a ; \mathbf{w}\right)$ ，其中 $\pi\left(\cdot|S^{\prime} ; \mathbf{w}\right)$ 是 $q\left(S^{\prime}, \cdot ; \mathbf{w}\right)$ 确定的策略（如 $\varepsilon$ 柔性策略）若是 $\mathrm{Q}$ 学习则 $U \leftarrow R+\gamma \max_{a} q\left(S^{\prime}, a ; \mathbf{w}\right)$
    4. （更新动作价值函数）若是状态价值评估则更新 $\mathbf{w}$ 以减小 $[U-v(S ; \mathbf{w})]^{2}$ (如 $\mathbf{w} \leftarrow \mathbf{w}+\alpha[U-v(S ; \mathbf{w})] \nabla v(S ; \mathbf{w}))$ ，若是期望 $\operatorname{SARSA}$ 算法或 $\mathrm{Q}$ 学习则更新参数 $\mathbf{w}$ 以减小 $[U-q(S, A ; \mathbf{w})]^{2}($ 如 $\mathbf{w} \leftarrow \mathbf{w}+\alpha[U-q(S, A ; \mathbf{w})] \nabla q(S, A ; \mathbf{w}))$ ．注意此步不可以重新计算 $U_{0}$
    5. $S \leftarrow S^{\prime}$．

***********************

如果采用能够自动计算微分并更新参数的软件包来减小损失，则务必注意不能对回报的估计求梯度．有些软件包可以阻止计算过程中梯度的传播，也可以在计算回报估计的表达式时使用阻止梯度传播的功能．还有一种方法是复制一份参数 $\mathbf{w}_{\text {目标 }} \leftarrow \mathbf{w}$ ，在计算回报估计的表达式时用这份复制后的参数 $\mathbf{w}_{\text {目标 }}$ 来计算回报估计，而在自动微分时只对原来的参数进行微分，这样就可以避免对回报估计求梯度．

#### 4.1.3 带资格迹的半梯度下降

在第 3 章中，我们学习了资格迹算法．资格迹可以在回合更新和单步时序差分更新之间进行折中，可能获得比回合更新或单步时序差分更新都更好的结果．回顾前文，在资格迹算法中，每个价值估计的数值都对应着一个资格迹参数，这个资格迹参数表示这个价值估计数值在更新中的权重．最近遇到的状态动作对（或状态）的权重大，比较久以前遇到的状态动作对（或状态）的权重小，从来没有遇到过的状态动作对（或状态）的权重为 0 ．每次更新时，都可以更新整条轨迹上的资格迹，再利用资格迹作为权重，更新整条轨迹上的价值估计．

资格迹同样可以运用在函数近似算法中，实现回合更新和单步时序差分的折中．这时，资格迹对应价值参数 $\mathbf{w}$ ．具体而言，资格迹参数 $\mathbf{z}$ 和价值参数 $\mathbf{w}$ 具有相同的形状大小，并且逐元素一一对应．资格迹参数中的每个元素表示了在更新价值参数对应元素时应当使用的权重乘以价值估计对该分量的梯度．也就是说，在更新价值参数 $\mathbf{w}$ 的某个分量 $w$ 对应着资格迹参数 $\mathrm{z}$ 中的某个分量 $z$ 时，那么在更新 $w$ 时应当使用以下迭代式更新：
$$
\begin{aligned}
&w \leftarrow w+\alpha z\left[U-q\left(S_{t}, A ; \mathbf{w}\right)\right], \quad &更新动作价值\\
&w \leftarrow w+\alpha z\left[U-v\left(S_{t} ; \mathbf{w}\right)\right], \quad &更新状态价值
\end{aligned}
$$
对价值参数整体而言，就有
$$
\begin{aligned}
&\mathbf{w} \leftarrow \mathbf{w}+\alpha\left[U-q\left(S_{t}, A_{t} ; \mathbf{w}\right)\right] \mathbf{z}, &更新动作价值\\
&\mathbf{w} \leftarrow \mathbf{w}+\alpha\left[U-v\left(S_{t} ; \mathbf{w}\right)\right] \mathbf{z}, \quad &更新状态价值
\end{aligned}
$$
当选取资格迹为累积迹时，资格迹的递推定义式如下：当 $t=0$ 时 $\mathbf{z}_{0}=\mathbf{0} ;$ 当 $t>0$ 时
$$
\begin{aligned}
&\mathbf{z}_{t}=\gamma \lambda \mathbf{z}_{t-1}+\nabla q\left(S_{t}, A_{t} ; \mathbf{w}\right) &更新动作价值对应的资格迹\\
&\mathbf{z}_{t}=\gamma \lambda \mathbf{z}_{t-1}+\nabla v\left(S_{t} ; \mathbf{w}\right) &更新状态价值对应的资格迹
\end{aligned}
$$
资格迹的递推式由 2 项组成．递推式的第一项是对前一次更新时使用的资格迹衰减而来，衰减系数是 $\gamma \lambda$ ，这是一个 0 到 1 之间的数．可以通过改变 $\lambda$ 的值，决定衰减的速度．
当 $\lambda$ 接近 0 时，衰减快；当 $\lambda$ 接近 1 时，衰减慢．递推式的第二项是加强项，它由动作价值的梯度值决定．动作价值的梯度值事实上确定了价值参数对总体价值估计的影响．对总体价值估计影响大的那些价值参数分量是当前比较重要的分量，应当加强它的资格迹．不过，梯度的分量值不一定是正数或 0 ，也可能是负数．所以，更新后的资格迹分量也可能是负值．当资格迹的某些分量是负值时，对应价值参数分量的权重值就是负值．进一步而言，在价值参数更新时，面对相同的时序差分误差，会出现价值参数的某些分量增大而另一些
分量减小的情况． 算法 4-5 和算法 4-6 给出了使用资格迹的价值估计和最优策略求解算法．这两个算法都
使用了累积迹．

>**算法 6-5** $\quad$ $\mathrm{TD}(\lambda)$ 算法估计动作价值或 $\operatorname{SARSA}(\lambda)$ 算法
***********************

1. （初始化）任意初始化参数 $\mathbf{w}$
2. 逐回合执行以下操作
 2.1 （初始化状态动作对）选择状态 $S_{0}$ 如果是策略评估，则用输入策略 $\pi(\cdot|S)$ 确定动作 $A$ ；如果是寻找最优策略，则用当前动作价值估计 $q(S, \cdot ; \mathbf{w})$ 导出的策略（如 $\varepsilon$ 柔性策略 $)$ 确定动作 $A$
 2.2 如果回合未结東，执行以下操作
    1. （采样）执行动作 $A$ ，观测得到奖励 $R$ 和新状态 $S^{\prime}$
    2. 如果是策略评估，则用输入策略 $\pi\left(\cdot|S^{\prime}\right)$ 确定动作 $A^{\prime}$ ；如果是寻找最优策略，用当前动作价值估计 $q\left(S^{\prime}, ; \mathbf{w}\right)$ 导出的策略 $($ 如 $\varepsilon$ 柔性策略 $)$ 确定动作 $A^{\prime}$；
    3. （计算回报的估计值） $U \leftarrow R+\gamma q\left(S^{\prime}, A^{\prime} ; \mathbf{w}\right)$
    4. （更新资格迹） $\mathbf{z} \leftarrow \gamma \lambda \mathbf{z}+\nabla q(S, A ; \mathbf{w})$
    5. （更新动作价值函数） $\mathbf{w} \leftarrow \mathbf{w}+\alpha[U-q(S, A ; \mathbf{w})] \mathbf{z}$
    6. $S \leftarrow S^{\prime}, A \leftarrow A^{\prime}$.

***********************

>**算法 4-6** $\quad$ TD(\lambda) 估计状态价值或期望 SARSA(\lambda) 算法或 $Q$ 学习
***********************

1. （初始化）任意初始化参数 $\mathbf{w}$
2. 逐回合执行以下操作
 2.1 （初始化资格迹） $\mathbf{z} \leftarrow \mathbf{0}$
 2.2 （初始化状态）选择状态 S
 2.3 如果回合未结束，执行以下操作
    1. 如果是策略评估，则用输入策略 $\pi(\cdot|S)$ 确定动作 $A$ ；如果是寻找最优策略，用当前动作价值估计 $q(S, \cdot ; \mathbf{w})$ 导出的策略 $($ 如 $\varepsilon$ 柔性策略 $)$ 确定动作 $A$
    2. （采样）执行动作 $A$ ，观测得到奖励 $R$ 和新状态 $S^{\prime}$
    3. （计算回报的估计值）如果是状态价值评估，则$U\leftarrow R+\gamma v(S^{\prime};\mathbf{w})$ ．如果是期望 SARSA 算法，则 $U \leftarrow R+\gamma \sum_{a} \pi\left(a|S^{\prime} ; \mathbf{w}\right) q\left(S^{\prime}, a ; \mathbf{w}\right)$ ，其中 $\pi\left(\cdot|S^{\prime} ; \mathbf{w}\right)$ 是 $q\left(S^{\prime},\cdot\:, \mathbf{w}\right)$ 确定的策略 (如 $\varepsilon$ 柔性策略 $)$ ．若是 $Q$ 学习则 $U \leftarrow R+\gamma \max_{a} q\left(S^{\prime}, a ; \mathbf{w}\right)$
    4. （更新资格迹）若是状态价值评估，则 $\mathbf{z} \leftarrow \gamma \lambda \mathbf{z}+\nabla v(S ; \mathbf{w})$ ；若是期望 $\mathrm{SARSA}$ 法或 $\mathrm{Q}$ 学习，则 $\mathbf{z} \leftarrow \gamma \lambda \mathbf{z}+\nabla q(S, A ; \mathbf{w})$
    5. （更新动作价值函数）若是状态价值评估，则 $\mathbf{w} \leftarrow \mathbf{w}+\alpha[U-v(S ; \mathbf{w})] \mathbf{z}$ ；若是期望 $\mathrm{SARSA}$ 算法或 $\mathrm{Q}$ 学习，则 $\mathbf{w} \leftarrow \mathbf{w}+\alpha[U-q(S, A ; \mathbf{w})] \mathbf{z}$
    6. $S \leftarrow S^{\prime}$.

***********************

TODO:线性近似

TODO:4.3函数近似的收敛性

### 4.4 深度Q学习

本节介绍一种目前非常热门的函数近似方法——深度 Q 学习．深度 Q 学习将深度学习和强化学习相结合，是第一个深度强化学习算法．深度 Q 学习的核心就是用一个人工神经网络 $q(s, a ; \mathbf{w}), s \in \mathcal{S}, a \in \mathcal{A}$ 来代替动作价值函数．由于神经网络具有强大的表达能力，能够自动寻找特征，所以采用神经网络有潜力比传统人工特征强大得多．最近基于深度 Q 网络的深度强化学习算法有了重大的进展，在目前学术界有非常大的影响力．当同时出现异策、自益和函数近似时，无法保证收敛性，会出现训练不稳定或训练困难等问题．针对出现的各种问题，研究人员主要从以下两方面进行了改进．

- **经验回放**（experience replay): 将经验（即历史的状态、动作、奖励等）存储起来，再在存储的经验中按一定的规则采样．
- **目标网络**（target network): 修改网络的更新方式，例如不把刚学习到的网络权重马上用于后续的自益过程．

本节后续内容将从这两条主线出发，介绍基于深度 Q 网络的强化学习算法．

#### 4.4.1 经验回放

V. Mnih 等在 2013 年发表文章《Playing Atari with deep reinforcement learning 》，提出了基于经验回放的深度 Q 网络，标志着深度 Q 网络的诞生，也标志着深度强化学习的诞生．在 4.2 节中我们知道，采用批处理的模式能够提供稳定性．经验回放就是一种让经验的概率分布变得稳定的技术，它能提高训练的稳定性．经验回放主要有“存储”和“采样回放”两大关键步骤．

- 存储：将轨迹以 $\left(S_{t}, A_{t}, R_{t+1}, S_{t+1}\right)$ 等形式存储起来；
- 采样回放：使用某种规则从存储的 $\left(S_{t}, A_{t}, R_{t+1}, S_{t+1}\right)$ 中随机取出一条或多条经验

算法 4-8 给出了带经验回放的 Q 学习最优策略求解算法

>**算法 6-8**$\quad$ 带经验回放的 $\mathrm{Q}$ 学习最优策略求解
***********************

1. （初始化）任意初始化参数 $\mathbf{w}$
2. 逐回合执行以下操作
 2.1 （初始化状态）选择状态 $S$
 2.2 如果回合未结東，执行以下操作
    1. （采样）根据 $q(S, \cdot ; \mathbf{w})$ 选择动作 $A$ 并执行，观测得到奖励 $R$ 和新状态 $S^{\prime}$
    2. （存储）将经验 $\left(S, A, R, S^{\prime}\right)$ 存入经验库中
    3. （回放）从经验库中选取经验 $\left(S_{i}, A_{t}, R_{i}, S_{i}^{\prime}\right)$
    4. （计算回报的估计值） $U_{i} \leftarrow R_{i}+\gamma \max _{a} q\left(S_{i}^{\prime}, a ; \mathbf{w}\right)$
    5. （更新动作价值函数）更新 $\mathbf{w}$ 以减小 $\left[U_{i}-q\left(S_{i}, A_{t} ; \mathbf{w}\right)\right]^{2}$ (如 $\mathbf{w \leftarrow w}+\alpha\left[U_{i}-\right.\left.\left.q\left(S_{i}, A_{t} ; \mathbf{w}\right)\right] \nabla q\left(S_{i}, A_{t} ; \mathbf{w}\right)\right)$
    6. $S \leftarrow S^{\prime}$

***********************

经验回放有以下好处．

- 在训练 $\mathrm{Q}$ 网络时，可以消除数据的关联，使得数据更像是独立同分布的（独立同分布是很多有监督学习的证明条件）这样可以减小参数更新的方差，加快收敛．
- 能够重复使用经验，对于数据获取困难的情况尤其有用．

从存储的角度，经验回放可以分为集中式回放和分布式回放．

- **集中式回放**：智能体在一个环境中运行，把经验统一存储在经验池中．
- **分布式回放**：智能体的多份拷贝（worker) 同时在多个环境中运行，并将经验统一存储于经验池中．由于多个智能体拷贝同时生成经验，所以能够在使用更多资源的同
时更快地收集经验

从采样的角度，经验回放可以分为均匀回放和优先回放．

- **均匀回放**：等概率从经验集中取经验，并且用取得的经验来更新最优价值函数
- **优先回放**（Prioritized Experience Replay, PER）：为经验池里的每个经验指定一个优先级，在选取经验时更倾向于选择优先级高的经验．

T. Schaul 等于 2016 年发表文章 《Prioritized experience replay》，提出了优先回放．优先回放的基本思想是为经验池里的经验指定一个优先级，在选取经验时更倾向于选择优先级高的经验．一般的做法是，如果某个经验（例如经验 $i$ ）的优先级为 $p_{i}$ ，那么选取该经验的概率为
$$
\frac{p_{i}}{\sum_{k} p_{k}}
$$
经验值有许多不同的选取方法，最常见的选取方法有成比例优先和基于排序优先．

- **成比例优先**（proportional priority）：第 $i$ 个经验的优先级为 $p_{i}=\left(\delta_{i}+\varepsilon\right)^{\alpha}$ ．其中 $\delta_{i}$ 是时序差分误差 $\left(\right.$ 定义为 $\delta_{i}=U_{t}-q\left(S_{t}, A_{t} ; \mathbf{w}\right)$ 或 $\left.\delta_{i}=U_{t}-v\left(S_{t} ; \mathbf{w}\right)\right), \quad \varepsilon$ 是预先选择
的一个小正数， $\alpha$ 是正参数．
- **基于排序优先**（rank-based priority): 第 $i$ 个经验的优先级为 $p_{i}=\left(\frac{1}{\operatorname{rank}_{i}}\right)^{\alpha}$ ．其中 $\mathrm{rank}_{i}$ 是第 $i$ 个经验从大到小排序的排名，排名从 1 开始．

D. Horgan 等在 2018 发表文章 《 Distributed prioritized experience replay》，将分布式经
签回放和优先经验回放相结合，得到分布式优先经验回放（distributed prioritized experience replay）．

#### 4.4.2 带目标网络的深度Q学习

对于基于自益的 Q 学习，其回报的估计和动作价值的估计都和权重 $\mathbf{w}$ 有关．当权重值变化时，回报的估计和动作价值的估计都会变化．在学习的过程中，动作价值试图追逐一个变化的回报，也容易出现不稳定的情况．在 4.1.2 节中给出了半梯度下降的算法来解决 这个问题．在半梯度下降中，在更新价值参数 $\mathbf{w}$ 时，不对基于自益得到的回报估计 $U_{t}$ 求梯 度．其中一种阻止对 $U_{t}$ 求梯度的方法就是将价值参数复制一份得到 $\mathbf{w}_{\text {目标 }}$ ，在计算 $U_{t}$ 时用 $\mathbf{w}_{\text {目标 }}$ 计算．基于这一方法，V. Mnih 等在 2015 年发表了论文《Human-level control through deep reinforcement learning》，提出了目标网（target network）这一概念．目标网络是在原有的神经网络之外再搭建一份结构完全相同的网络．原先就有的神经网络称为评估网络（evaluation network）．在学习的过程中，使用目标网络来进行自益得到回报的评估值，作为学习的目标．在权重更新的过程中，只更新评估网络的权重，而不更新目标网络的权重．这样，更新权重时针对的目标不会在每次迭代都变化，是一个固定的目标．在完成一定次数的更新后，再将评估网络的权重值赋给目标网络，进而进行下一批更新．这样，目标网络也能得到更新．由于在目标网络没有变化的一段时间内回报的估计是相对固定的，目标网络的引入增加了学习的稳定性．所以，目标网络目前已经成为深度 Q 学习的主流做法．

算法 4-9 给出了带目标网络的深度 Q 学习算法．算法开始时将评估网络和目标网络初始化为相同的值．为了得到好的训练效果，应当按照神经网络的相关规则仔细初始化神经网络的参数．

>**算法 4-9**$\quad$ 带经验回放和目标网络的深度 $\mathrm{Q}$ 学习最优策略求解
***********************

1. （初始化）初始化评估网络 $q(\cdot, \cdot ; \mathbf{w})$ 的参数 $\mathbf{w}$ ；目标网络 $q\left(\cdot, \cdot ; \mathbf{w}_{\text {目标 }}\right)$ 的参数 $\mathbf{w}_{\text {目标 }} \leftarrow \mathbf{w}$
2. 逐回合执行以下操作
 2.1 （初始化状态）选择状态 $S$
 2.2 如果回合未结束，执行以下操作：
    1. （采样）根据 $q(S, \cdot ; \mathbf{w})$ 选择动作 $A$ 并执行，观测得到奖励 $R$ 和新状态 $S^{\prime}$
    2. （经验存储）将经验 $\left(S, A, R, S^{\prime}\right)$ 存入经验库 $\mathcal{D}$ 中
    3. （经验回放）从经验库 $\mathcal{D}$ 中选取一批经验 $\left(S_{i}, A_{t}, R_{i}, S_{i}^{\prime}\right)(i \in \mathcal{B})$
    4. （计算回报的估计值） $U_{i} \leftarrow R_{i}+\gamma \max _{a} q\left(S_{i}^{\prime}, a ; \mathbf{w}_{\text {目标 }}\right)(i \in \mathcal{B})$
    5. （更新动作价值函数）更新 $\mathbf{w}$ 以减小 $\frac{1}{|\mathcal{B}|} \sum_{i \in \mathcal{B}}\left[U_{i}-q\left(S_{i}, A_{t} ; \mathbf{w}\right)\right]^{2}$ (如 $\mathbf{w} \leftarrow \mathbf{w}+\alpha \frac{1}{|\mathcal{B}|}\left.\sum_{i \in \mathcal{B}}\left[U_{i}-q\left(S_{i}, A_{t} ; \mathbf{w}\right)\right] \nabla q\left(S_{i}, A_{t} ; \mathbf{w}\right)\right)$
    6. $S \leftarrow S^{\prime}$
    7. （更新目标网络）在一定条件下（例如访问本步若千次）更新目标网络的权重 $\mathbf{w}_{\text {目标 }} \leftarrow \mathbf{w}$

***********************

在更新目标网络时，可以简单地把评估网络的参数直接赋值给目标网络 $\left(\right.$ 即 $\left.\mathbf{w}_{\text {目标 }} \leftarrow \mathbf{w}\right)$ ，也可以引人一个学习率 $\alpha_{\text {目标 }}$ 把旧的目标网络参数和新的评估网络参数直接做加权平均后的值赋值给目标网络 $\left(\right.$ 即 $\left.\mathbf{w}_{\text {目标 }} \leftarrow\left(1-\alpha_{\text {目标 }}\right) \mathbf{w}_{\text {目标 }}+\alpha_{\text {目标 }} \mathbf{w}\right)$ ．事实上，直接赋值的版本是带学习率版本在 $\alpha_{\text {目标 }}=1$ 时的特例．对于分布式学习的情形，有很多独立的拷贝（worker）同时会修改目标网络，则就更常用学习率 $\alpha_{\text {目标 }} \in(0,1)$ ．

#### 4.4.3 双重深度Q学习

第 3 章曾提到 $\mathrm{Q}$ 学习会带来最大化偏差，而双重 $\mathrm{Q}$ 学习却可以消除最大化偏差．基于查找表的双重 Q 学习引入了两个动作价值的估计 $q^{(0)}$ 和 $q^{(1)}$ ，每次更新动作价值时用其中的一个网络确定动作，用确定的动作和另外一个网络来估计回报．

对于深度 Q 学习也有同样的结论．Deepmind 于 2015 年发表论文 《 Deep reinforcement learning with double Q-learning 》，将双重 Q 学习用于深度 Q 网络，得到了双重深度 Q 网络（Double Deep Q Network, Double $\mathrm{DQN}$ ）．考虑到深度 Q 网络已经有了评估网络和目标网络两个网络，所以双重深度 Q 学习在估计回报时只需要用评估网络确定动作，用目标网络确定回报的估计即可．所以，只需要将算法 4-10 中的
$$
U_{i} \leftarrow R_{i}+\gamma \max _{a} q\left(S_{i}^{\prime}, a ; \mathbf{w}_{\text {目标 }}\right)
$$
更换为
$$
U_{i} \leftarrow R_{i}+\gamma q\left(S_{i}^{\prime}, \arg \max _{a} q\left(S_{i}^{\prime}, a ; \mathbf{w}\right) ; \mathbf{w}_{\text {目标 }}\right)
$$
就得到了带经验回放的双重深度 Q 网络算法．

#### 4.4.4 对偶深度 Q 网络

Z. Wang 等在 2015 年发表论文《Dueling network architectures for deep reinforcement learning 》，提出了一种神经网络的结构——对偶网络（ duel network）．对偶网络理论利用动作价值函数和状态价值函数之差定义了一个新的函数——优势函数（advantage function)：
$$
a(s, a)=q(s, a)-v(s), \quad s \in \mathcal{S}, a \in \mathcal{A}
$$
$$
a(s, a)=q(s, a)-v(s), \quad s \in \mathcal{S}, a \in \mathcal{A}
$$
对偶 $\mathrm{Q}$ 网络仍然用 $q(\mathbf{w})$ 来估计动作价值，只不过这时候 $q(\mathbf{w})$ 是状态价值估计 $v(s ; \mathbf{w})$ 和优
势函数估计 $a(s, a ; \mathbf{w})$ 的叠加，即
$$
q(s, a ; \mathbf{w})=v(s ; \mathbf{w})+a(s, a ; \mathbf{w})
$$
其中 $v(\mathbf{w})$ 和 $a(\mathbf{w})$ 可能都只用到了 $\mathbf{w}$ 中的部分参数．在训练的过程中， $v(\mathbf{w})$ 和 $a(\mathbf{w})$ 是共
同训练的，训练过程和单独训练普通深度 Q 网络并无不同之处．不过，同一个 $q(\mathbf{w})$ 事实上存在着无穷多种分解为 $v(\mathbf{w})$ 和 $a(\mathbf{w})$ 的方式．如果某个 $q(s, a ; w)$ 可以分解为某个 $v(s ; \mathbf{w})$ 和 $a(s, a ; \mathbf{w})$ ，那么它也能分解为 $v(s ; \mathbf{w})+c(s)$ 和 $a(s, a ; \mathbf{w})-c(s)$ ，其中 $c(s)$ 是任意一个只和状态 $s$ 有关的函数．为了不给训练带来不必要的麻烦，往往可以通过增加一个由优势函数导出的量，使得等效的优势函数满足固定的特征，使得分解唯一．常见的方法有以下两种:

- 考虑优势函数的最大值，令
$$
q(s, a ; \mathbf{w})=v(s ; \mathbf{w})+a(s, a ; \mathbf{w})-\max _{a \in \mathcal{A}} a(s, a ; \mathbf{w})
$$
使得等效优势函数 $a_{\text {等效 }}(s, a ; \mathbf{w})=a(s, a ; \mathbf{w})-\max _{a \in \mathcal{A}} a(s, a ; \mathbf{w})$ 满足
$$
\max_{\sigma \in \mathcal{A}} a_{\text {等效 }}(s, a ; \mathbf{w})=0, \quad s \in \mathcal{S}
$$
- 考虑优势函数的平均值，令
$$
q(s, a ; \mathbf{w})=v(s ; \mathbf{w})+a(s, a ; \mathbf{w})-\frac{1}{|\mathcal{A}|} \sum_{a \in \mathcal{A}} a(s, a ; \mathbf{w})
$$
使得等效优势函数 $a_{\text {等效 }}(s, a ; \mathbf{w})=a(s, a ; \mathbf{w})-\frac{1}{|\mathcal{A}|} \sum_{a \in \mathcal{A}} a(s, a ; \mathbf{w})$ 满足
$$
\sum_{a \in \mathcal{A}} a_{\text {等效 }}(s, a ; \mathbf{w})=0, \quad s \in \mathcal{S}
$$