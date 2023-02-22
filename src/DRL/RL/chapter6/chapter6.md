## 六． 执行者/评论者方法

本章介绍带自益的策略梯度算法．这类算法将策略梯度和自益结合了起来：一方面，用一个含参函数近似价值函数，然后利用这个价值函数的近似值来估计回报值；另一方面，利用估计得到的回报值估计策略梯度，进而更新策略参数．这两方面又常常被称为**评论者** (critic) 和**执行者**（actor）．所以，带自益的策略梯度算法被称为**执行者 / 评论者算法**（actorcritic algorithm)．

### 6.1 执行者 / 评论者算法

同样用含参函数 $h(s, a ; \theta)$ 表示偏好，用其 $\operatorname{softmax}$ 运算的结果 $\pi(a|s ; \boldsymbol{\theta})$ 来近似最优策略．在更新参数 $\boldsymbol{\theta}$ 时，执行者 / 评论者算法依然也是根据策略梯度定理，取 $\mathrm{E}\left[\Psi_{t} \nabla \ln \pi\left(A_{t}|S_{t} ; \boldsymbol{\theta}\right)\right]$ 为梯度方向迭代更新．其中， $\Psi_{t}=\gamma^{t}\left(G_{t}-B(s)\right)\mathrm{J} .$ Schulman 等在文章《 High-dimensional continuous control using generalized advantage estimation 》中指出， $\Psi_{t}$ 并不拘泥于以上形式． $\Psi_{t}$ 可以是以下几种形式：

- （动作价值） $\Psi_{t}=\gamma^{t} q_{\pi}\left(S_{t}, A_{t}\right)$
- （优势函数） $\Psi_{t}=\gamma^{t}\left[q_{\pi}\left(S_{t}, A_{t}\right)-v_{\pi}\left(S_{t}\right)\right]$
- （时序差分） $\Psi_{t}=\gamma^{t}\left[R_{t+1}+\gamma v_{\pi}\left(S_{t+1}\right)-v_{\pi}\left(S_{t}\right)\right]$

在以上形式中，往往用价值函数来估计回报．例如，由于 $\mathrm{E}\left[\gamma^{t} G_{t} \nabla \ln \pi\left(A_{t}|S_{t} ; \boldsymbol{\theta}\right)\right]=\mathrm{E}\left[\gamma^{t} q_{\pi(\theta)}\left(S_{t}, A_{t}\right) \nabla \ln \pi\left(A_{t}|S_{t} ; \theta\right)\right]$ ，而且 $\mathrm{E}\left[q_{\pi(\theta)}\left(S_{t}, A_{t}\right) \nabla \ln \pi\left(A,|S_{t} ; \theta\right)\right]$ 也表征期望方向，所以
$\Psi_{t}=\gamma^{t} q_{\pi}\left(S_{t}, A_{t}\right)$ ，相当于用 $q_{\pi}\left(S_{t}, A_{t}\right)$ 表示期望．再例如，对于 $\Psi_{t}=\gamma^{t}\left[q_{\pi}\left(S_{t}, A_{t}\right)-v_{\pi}\left(S_{t}\right)\right]$ ，就相当于在回报 $q_{\pi}\left(S_{t}, A_{t}\right)$ 的基础上减去基线 $B(s)=v_{\pi}(s)$ 以减小方差．对于时序差分 $\Psi_{t}=\gamma^{t}\left[R_{t+1}+\gamma v_{\pi}\left(S_{t+1}\right)-v_{\pi}\left(S_{t}\right)\right]$ ，也是用 $R_{t+1}+\gamma v_{\pi}\left(S_{t+1}\right)$ 代表回报，再减去基线 $B(s)=v_{\pi}(s)$ 以减小方差．

不过在实际使用时，真实的价值函数是不知道的．但是，我们可以去估计这些价值函数．具体而言，我们可以用函数近似的方法，用含参函数 $v(s ; w)(s \in \mathcal{S})$ 或 $q(s, a ; \mathbf{w})$ $(s \in \mathcal{S}, a \in \mathcal{A}(s))$ 来近似 $v_{\pi}$ 和 $q_{\pi}$ ．在上一章中，带基线的简单策略梯度算法已经使用了含参函数 $v(\mathrm{~s} ; \mathbf{w})(s \in \mathcal{S})$ 作为基线函数．我们可以在此基础上进一步引人自益的思想，用价值的估计 $U_{t}$ 来代替 $\Psi_{t}$ 中表示回报的部分．例如，对于时序差分，用估计来代替价值函数可以得到 $\Psi_{t}=\gamma^{t}\left[R_{t+1}+\gamma v\left(S_{t+1} ; \mathbf{w}\right)-v\left(S_{t} ; \mathbf{w}\right)\right]$ ．这里的估计值 $v(\mathbf{w})$ 就是评论者，这样的算法就是执行者 / 评论者算法．
>注意：只有采用了自益的方法，即用价值估计来估计回报，并引入了偏差，才是执行者 / 评论者算法．用价值估计来做基线并没有带来偏差（因为基线本来就可以任意选择）．所以，带基线的简单策略梯度算法不是执行者 / 评论者算法．

#### 6.1.1 动作价值执行者 / 评论者算法

根据前述分析，同策执行者 / 评论者算法在更新策略参数 $\theta$ 时也应该试图减小 $-\Psi_{t} \ln \pi\left(A_{t}|S_{t} ; \theta\right)$ ，只是在计算 $\Psi_{t}$ 时采用了基于自益的回报估计．算法 6-1 给出了在回报估计为 $q\left(S_{t}, A_{t} ; \mathbf{w}\right)$ ，并取 $\Psi_{1}=\gamma^{\prime} q\left(S_{t}, A_{t} ; \mathbf{w}\right)$ 时的同策算法，称为动作价值执行者 / 评论者算法．算法一开始初始化了策略参数和价值参数．虽然算法中写的是可以将这个参数初始化为任意值，但是如果它们是神经网络的参数，还是应该按照神经网络的要求来初始化参数．在迭代过程中有个变量 $I$ ，用来存储策略梯度的表达式中的折扣因子 $\gamma^{t}$ ．在同一回合中，每一步都把这个折扣因子乘上 $\gamma$ ，所以第 $t$ 步就是 $\gamma^{t}$ ．

>**算法 6-1**：$\quad$ 动作价值同策执行者 / 评论者算法
***********************

输入：环境（无数学描述）
输出：最优策略的估计 $\pi(\boldsymbol{\theta})$
参数：优化器（隐含学习率 $\left.\alpha^{(\mathrm{w})}, \alpha^{(\theta)}\right)$ ，折扣因子 $\gamma$ ，控制回合数和回合内步数的参数．

1. （初始化） $\boldsymbol{\theta} \leftarrow$ 任意值， $\mathbf{w} \leftarrow$ 任意值
2. （带自益的策略更新）对每个回合执行以下操作:
 2.1 （初始化累积折扣） $I \leftarrow 1$
 2.2 （决定初始状态动作对）选择状态 $S$ ，并用 $\pi(\cdot|S ; \boldsymbol{\theta})$ 得到动作 $A$
 2.3 如果回合未结束，执行以下操作：
    1. （采样）根据状态 $S$ 和动作 $A$ 得到奖励 $R$ 和下一状态 $S^{\prime}$
    2. （执行）用 $\pi\left(\cdot|S^{\prime} ; \boldsymbol{\theta}\right)$ 得到动作 $A^{\prime}$
    3. （估计回报） $U \leftarrow R+\gamma q\left(S^{\prime}, A^{\prime} ; \mathbf{w}\right)$
    4. （策略改进）更新 $\theta$ 以减小 $-I q(S, A ; \mathbf{w}) \ln \pi(A|S ; \boldsymbol{\theta})\left(\right.$ 如 $\boldsymbol{\theta} \leftarrow \boldsymbol{\theta}+\alpha^{(\theta)} I q(S, A ; \mathbf{w})\nabla \ln \pi(A|S ; \boldsymbol{\theta}))$
    5. （更新价值）更新 $\mathbf{w}$ 以减小 $[U-q(S, A ; \mathbf{w})]^{2}\left(\right.$ 如 $\mathbf{w} \leftarrow \mathbf{w}+\alpha^{(\mathbf{w})}[U-q(S, A ; \mathbf{w})]\nabla q(S, A ; w))$
    6. （更新累积折扣） $I \leftarrow \gamma I$
    7. （更新状态） $S \leftarrow S^{\prime}, \quad A \leftarrow A^{\prime}$．

***********************

#### 6.1.2 优势执行者 / 评论者算法

在基本执行者 / 评论者算法中引入基线函数 $B\left(S_{t}\right)=v\left(S_{t} ; \mathbf{w}\right)$ ，就会得到 $\Psi_{t}=\gamma^{t}\left[q\left(S_{t}, A_{t} ; \mathbf{w}\right)-\right.$
$\left.v\left(S_{t} ; \mathbf{w}\right)\right]$ ，其中， $q\left(S_{t}, A_{t} ; \mathbf{w}\right)-v\left(S_{t} ; \mathbf{w}\right)$ 是优势函数的估计．这样，我们就得到了优势执行者
评论者算法．不过，如果采用 $q\left(S_{t}, A_{t} ; \mathbf{w}\right)-v\left(S_{t} ; \mathbf{w}\right)$ 这样形式的优势函数估计值，我们就需搭建两个函数分别表示 $q(\mathbf{w})$ 和 $v(\mathbf{w})$ ．为了避免这样的麻烦，这里用了 $U_{t}=R_{t+1}+\gamma v\left(S_{t+1} ; w\right)$ 做目标，这样优势函数的估计就变为单步时序差分的形式 $R_{t+1}+\gamma v\left(S_{t+1} ; \mathbf{w}\right)-v\left(S_{t} ; \mathbf{w}\right)$．

如果优势执行者 / 评论者算法在执行过程中不是每一步都更新参数，而是在回合结束后用整个轨迹来进行更新，就可以把算法分为经验搜集和经验使用两个部分．这样的分隔可以让这个算法同时有很多执行者在同时执行．例如，让多个执行者同时分别收集很多经验，然后都用自己的那些经验得到一批经验所带来的梯度更新值．每个执行者在一定的时机更新参数，同时更新策略参数 $\boldsymbol{\theta}$ 和价值参数 $\mathbf{w}$ ．每个执行者的更新是异步的．所以，这样的并行算法称为**异步优势执行者 / 评论者算法**（ Asynchronous Advantage Actor-Critic, $\mathrm{A} 3 \mathrm{C}$ )．异步优势执行者 / 评论者算法中的自益部分，不仅可以采用单步时序差分，也可以使用多步时序差分．另外，还可以对函数参数的访问进行控制，使得所有执行者统一更新参数．这样的并行算法称为**优势执行者 / 评论者算法**（Advantage Actor-Critic, $\mathrm{A} 2 \mathrm{C}$ )．算法 $6-3$ 给出了异步优势执行者 / 评论者算法．异步优势执行者 / 评论者算法可以有许多执行者 (或称多个线程 )，所以除了有全局的价值参数 $\mathbf{w}$ 和策略参数 $\boldsymbol{\theta}$ 外，每个线程还可能有自己维护的价值参数 $\mathbf{w}^{\prime}$ 和 $\boldsymbol{\theta}^{\prime}$ ．执行者执行时，先从全局同步参数，然后再自己学习，最后统一同步全局参数．

>**算法 6-2**： $\quad$ 优势执行者 / 评论者算法
***********************

输入：环境（无数学描述）
输出：最优策略的估计 $\pi(\boldsymbol{\theta})$
参数：优化器（隐含学习率 $\left.\alpha^{(\theta)}, \alpha^{(\mathbf{w})}\right)$ ，折扣因子 $\gamma$ ，控制回合数和回合内步数的参数．

1. （初始化） $\boldsymbol{\theta} \leftarrow$ 任意值， $\mathbf{w} \leftarrow$ 任意值．
2. （带自益的策略更新）对每个回合执行以下操作：
 2.1 （初始化累积折扣） $I \leftarrow 1$
 2.2 （决定初始状态）选择状态 $S$
 2.3 如果回合未结束，执行以下操作：
    1. （采样）用 $\pi(\cdot|S ; \theta)$ 得到动作 $A$
    2. （执行）执行动作 $A$ ，得到奖励 $R$ 和观测 $S^{\prime}$
    3. （估计回报） $U \leftarrow R+\gamma v\left(S^{\prime} ; \mathbf{w}\right)$
    4. （策略改进）更新 $\boldsymbol{\theta}$ 以减小 $-I[U-v(S ; \mathbf{w})] \ln \pi(A|S ; \theta)\left(\right.$ 如 $\boldsymbol{\theta} \leftarrow \boldsymbol{\theta}+\alpha^{(\theta)} I[U-v(S ; \mathbf{w})]\nabla \ln \pi(A|S ; \theta))$
    5. （更新价值）更新 $\mathbf{w}$ 以减小 $[U-v(S ; \mathbf{w})]^{2}\left(\right.$ 如 $\left.\mathbf{w} \leftarrow \mathbf{w}+\alpha^{(\mathbf{w})}[U-v(S ; \mathbf{w})] \nabla v(S ; \mathbf{w})\right)$
    6. （更新累积折扣） $I \leftarrow \gamma I$
    7. （更新状态）$S\leftarrow S^\prime$．

***********************

>**算法 6-3**：$\quad$ 异步优势执行者 / 评论者算法 (演示某个线程的行为)
***********************

输入：环境（无数学描述）
输出：最优策略的估计 $\pi(\boldsymbol{\theta})$
参数：优化器（隐含学习率 $\left.\alpha^{(\theta)}, \alpha^{(\mathbf{w})}\right)$ ，折扣因子 $\gamma$ ，控制回合数和回合内步数的参数

1. （同步全局参数） $\boldsymbol{\theta}^{\prime} \leftarrow \boldsymbol{\theta}, \mathbf{w}^{\prime} \leftarrow \mathbf{w}$
2. 逐回合执行以下过程：
 2.1 用策略 $\pi\left(\boldsymbol{\theta}^{\prime}\right)$ 生成轨迹 $S_{0}, A_{0}, R_{1}, S_{1}, A_{1}, R_{1}, \ldots, S_{T-1}, A_{T-1}, R_{T}, S_{T}$ ，直到回合结束或执行步数达
到上限 $T$
 2.2 为梯度计算初始化：
    - （初始化目标 $U_{T}$）若 $S_{T}$ 是终止状态，则 $U \leftarrow 0$ ；否则 $U \leftarrow v\left(S_{T} ; \mathbf{w}^{\prime}\right)$
    - （初始化梯度）$\mathbf{g}^{(\theta)} \leftarrow \mathbf{0}, \mathbf{g}^{(\mathbf{w})} \leftarrow \mathbf{0}$
 2.3 （异步计算梯度）对 $t=T-1, T-2, \ldots, 0$ ，执行以下内容：
    - （估计目标）计算 $U \leftarrow \gamma U+R_{t+1}$
    - （估计策略梯度方向）$\mathbf{g}^{(\theta)} \leftarrow \mathbf{g}^{(\theta)}+\left[U-v\left(S_{t} ; \mathbf{w}^{\prime}\right)\right] \nabla \ln \pi\left(A_{t}|S_{t} ; \boldsymbol{\theta}^{\prime}\right)$
    - （估计价值梯度方向） $\mathbf{g}^{(w)} \leftarrow \mathbf{g}^{(\mathbf{w})}+\left[U-v\left(S_{t} ; \mathbf{w}^{\prime}\right)\right] \nabla v\left(S_{t} ; \mathbf{w}^{\prime}\right)$
3. （同步更新）更新全局参数
 3.1 （策略更新）用梯度方向 $\mathbf{g}^{(\theta)}$ 更新策略参数 $\boldsymbol{\theta}\left(\right.$ 如 $\left.\boldsymbol{\theta} \leftarrow \boldsymbol{\theta}+\alpha^{(\theta)} \mathbf{g}^{(\mathbf{\theta})}\right)$
 3.2 （价值更新）用梯度方向 $\mathbf{g}^{(\mathbf{w})}$ 更新价值参数 $\mathbf{w}\left(\right.$ 如 $\left.\mathbf{w} \leftarrow \mathbf{w}+\alpha^{(\mathbf{w})} \mathbf{g}^{(\mathbf{w})}\right)$．

***********************

#### 6.1.3 带资格迹的执行者 / 评论者方法

执行者 / 评论者算法引入了自益，那么它也就可以引入资格迹．算法 6-4 给出了带资格迹的优势执行者 / 评论者算法．这个算法里有两个资格迹 $\mathbf{z}^{(0)}$ 和 $\mathbf{z}^{(\mathbf{w})}$ ，它们分别与策略参数 $\theta$ 和价值参数 $\mathbf{w}$ 对应，并可以分别有自己的 $\lambda^{(\theta)}$ 和 $\lambda^{(\mathbf{w})}$ ．具体而言， $\mathbf{z}^{(\mathbf{w})}$ 与价值参数 $\mathbf{w}$ 对应，运用梯度为 $\nabla v(S ; \mathbf{w})$ ，参数为 $\lambda^{(\mathbf{w})}$ 的累积迹； $\mathbf{z}^{(\mathbf{\theta})}$ 与策略参数 $\boldsymbol{\theta}$ 对应，运用的梯度是 $\nabla \ln \pi(A|S ; \mathbf{w})$ 参数为 $\lambda^{(\theta)}$ 的累积迹，在运用中可以将折扣 $\gamma^{t}$ 整合到资格迹中．

>**算法 6-4**：$\quad$ 带资格迹的优势执行者 / 评论者算法
***********************
输入：环境（无数学描述）
输出：最优策略的估计 $\pi(\boldsymbol{\theta})$
参数：资格迹参数 $\lambda^{(\theta)}, \lambda^{(\mathbf{w})}$ ，学习率 $\alpha^{(0)}, \alpha^{(\mathbf{w})}$ ，折扣因子 $\gamma$ ，控制回合数和回合内步数的参数．

1. （初始化） $\boldsymbol{\theta} \leftarrow$ 任意值， $\mathbf{w} \leftarrow$ 任意值．
2. （带自益的策略更新）对每个回合执行以下操作：
 2.1 （初始化资格迹和累积折扣） $\mathbf{z}^{(\theta)} \leftarrow \mathbf{0}, \mathbf{z}^{(\mathbf{w})} \leftarrow \mathbf{0}, I \leftarrow 1$
 2.2 （决定初始状态）选择状态 $S$
 2.3 如果回合未结束，执行以下操作：
    1. （采样）用 $\pi(\cdot|S ；\boldsymbol{\theta})$ 得到动作 $A$
    2. （执行）执行动作 $A$ ，得到奖励 $R$ 和观测 $S^{\prime}$
    3. （估计回报） $U \leftarrow R+\gamma v\left(S^{\prime} ; \mathbf{w}\right)$
    4. （更新策略资格迹） $\mathbf{z}^{(\theta)} \leftarrow \gamma \lambda^{(\theta)} \mathbf{z}^{(\theta)}+I \nabla \ln \pi(A|S ; \mathbf{w})$
    5. （策略改进） $\boldsymbol{\theta} \leftarrow \boldsymbol{\theta}+\alpha^{(0)}[U-v(S ; \mathbf{w})] \mathbf{z}^{(\theta)}$
    6. （更新价值资格迹） $\mathbf{z}^{(\mathbf{w})} \leftarrow \gamma \lambda^{(\mathbf{w})} \mathbf{z}^{(\mathbf{w})}+\nabla v(S ; \mathbf{w})$
    7. （更新价值） $\mathbf{w} \leftarrow \mathbf{w}+\alpha^{(\mathbf{w})}[U-v(S ; \mathbf{w})] \mathbf{z}^{(\mathbf{w})}$
    8. （更新累积折扣） $I \leftarrow \gamma I$
    9. （更新状态） $S \leftarrow S^{\prime}$．

***********************

### 6.2 基于代理优势的同策算法

本节介绍面向代理优势的执行者 / 评论者算法．这些算法在迭代的过程中并没有直接优化期望目标，而是试图优化期望目标近似一代理优势．在很多问题上，这些算法会比简单的执行者 / 评论者算法得到更好的性能．

#### 6.2.1 代理优势

考虑采用迭代的方法更新策略 $\pi(\boldsymbol{\theta})$ ．在某次迭代后，得到了策略 $\pi\left(\boldsymbol{\theta}_{k}\right)$ ．接下来我们希望得到一个更好的策略 $\pi(\boldsymbol{\theta})$ ．Kakade 等在文章 《 Approximately optimal approximate reinforcement learning 》中证明了策略 $\pi(\boldsymbol{\theta})$ 和策略 $\pi\left(\boldsymbol{\theta}_{k}\right)$ 的期望回报满足**性能差别引理** (Performance Difference Lemma):
$$
\mathrm{E}_{\pi(\theta)}\left[G_{0}\right]=\mathrm{E}_{\pi\left(\theta_{k}\right)}\left[G_{0}\right]+\mathrm{E}_{\pi(\theta)}\left[\sum_{t=0}^{+\infty} \gamma^{t} a_{\pi\left(\theta_{k}\right)}\left(S_{t}, A_{t}\right)\right]
$$
（证明：
$$
\begin{array}{l}
\mathrm{E}_{\pi(\theta)}\left[\sum_{t=0}^{+\infty} \gamma^{t} a_{\pi\left(\theta_{k}\right)}\left(S_{t}, A_{t}\right)\right] \\
\quad=\mathrm{E}_{\pi(0)}\left[\sum_{t=0}^{+\infty} \gamma^{t}\left(R_{t+1}+\gamma v_{\pi\left(\theta_{k}\right)}\left(S_{t+1}\right)-v_{\pi\left(\theta_{k}\right)}\left(S_{t}\right)\right)\right] \\
\quad=\mathrm{E}_{\pi(0)}\left[-v_{\pi\left(\theta_{k}\right)}\left(S_{0}\right)+\sum_{t=0}^{+\infty} \gamma^{t} R_{t+1}\right] \\
\quad=-\mathrm{E}_{S_{0}}\left[v_{\pi\left(\theta_{k}\right)}\left(S_{0}\right)\right]+\mathrm{E}_{\pi(\theta)}\left[\sum_{t=0}^{+\infty} \gamma^{t} R_{t+1}\right] \\
\quad=-\mathrm{E}_{\pi\left(\theta_{k}\right)}\left[G_{0}\right]+\mathrm{E}_{\pi(\theta)}\left[G_{0}\right]
\end{array}
$$
得证．）

所以，要最大化 $\mathrm{E}_{\pi(\theta)}\left[G_{0}\right]$ ，就是要最大化优势的期望 $\mathrm{E}_{\pi(\theta)}\left[\sum_{t=0}^{+\infty} \gamma^{t} a_{\pi\left(\theta_{k}\right)}\left(S_{t}, A_{t}\right)\right]$ ．这个期望是对含参策略而言的．要优化这样的期望，可以利用以下形式的重采样，将其中对 $A_{t} \sim \pi(\boldsymbol{\theta})$ 求期望转化为对 $A_{t} \sim \pi\left(\boldsymbol{\theta}_{k}\right)$ 求期望：
$$
\mathrm{E}_{S_{t}, A_{t}\sim\pi(\theta)}\left[a_{\pi_{k}}\left(S_{t}, A_{t}\right)\right]=\mathrm{E}_{S_{t}\sim\pi(\theta), \mathcal{A}_t \sim \pi\left(\theta_{k}\right)}\left[\frac{\pi\left(A_{t}|S_{t} ; \theta\right)}{\pi\left(A_{t}|S_{t} ; \theta_{k}\right)} a_{\pi\left(\theta_{k}\right)}\left(S_{t}, A_{t}\right)\right]
$$
但是，对 $S_{t} \sim \pi(\boldsymbol{\theta})$ 求期望无法进一步转化．**代理优势**（ surrogate advantage）就是在上述重采样的基础上，将对 $S_{t} \sim \pi(\boldsymbol{\theta})$ 求期望近似为对 $S_{t} \sim \pi\left(\boldsymbol{\theta}_{k}\right)$ 求期望：
$$
\mathrm{E}_{S_{t}, A_{t} \sim \pi(\theta)}\left[a_{\pi_{k}}\left(S_{t}, A_{t}\right)\right] \approx \mathrm{E}_{S_{t}, A_{t}\sim \pi\left(\theta_{k}\right)}\left[\frac{\pi\left(A_{t}|S_{t} ; \theta\right)}{\pi\left(A_{t}|S_{t} ; \theta_{k}\right)} a_{\pi\left(\theta_{k}\right)}\left(S_{t}, A_{t}\right)\right]
$$
这样得到了 $\mathrm{E}_{\pi(\theta)}\left[G_{0}\right]$ 的近似表达式 $l(\boldsymbol{\theta})$ ，其中
$$
l(\boldsymbol{\theta})=\mathrm{E}_{\pi\left(\theta_{k}\right)}\left[G_{0}\right]+\mathrm{E}_{S_{t}, \mathcal{A}_t \sim \pi\left(\theta_{k}\right)}\left[\sum_{t=0}^{+\infty} \gamma^{t} \frac{\pi\left(A_{t}|S_{t} ; \boldsymbol{\theta}\right)}{\pi\left(A_{t}|S_{t} ; \boldsymbol{\theta}_{k}\right)} a_{\pi\left(\theta_{k}\right)}\left(S_{t}, A_{t}\right)\right]
$$
可以证明， $\mathrm{E}_{\pi(\theta)}\left[G_{0}\right]$ 和 $l(\boldsymbol{\theta})$ 在 $\boldsymbol{\theta}=\boldsymbol{\theta}_{k}$ 处有相同的值 $\mathrm{E}_{\pi\left(\theta_{k}\right)}\left[G_{0}\right]$ 和梯度．

虽然 $\mathrm{E}_{\pi(\theta)}\left[G_{0}\right]$ 没有直接的表达式而很难直接优化，但是只要沿着它的梯度方向改进策略参数，就有机会增大它．由于 $\mathrm{E}_{\pi(\theta)}\left[G_{0}\right]$ 和 $l(\boldsymbol{\theta})$ 在 $\boldsymbol{\theta}=\boldsymbol{\theta}_{k}$ 处有着相同的值和梯度方向， $\mathrm{E}_{\pi(\theta)}\left[G_{0}\right]$ 和代理优势有着相同的梯度方向．所以，沿着
$$
\mathrm{E}_{S_{t}, A_t\sim \pi\left(\theta_{k}\right)}\left[\sum_{t=0}^{+\infty} \gamma^{t} \frac{\pi\left(A_{t}|S_{t} ; \theta\right)}{\pi\left(A_{t}|S_{t} ; \theta_{k}\right)} a_{n\left(\theta_{k}\right)}\left(S_{t}, A_{t}\right)\right]
$$
的梯度方向就有机会改进 $\mathrm{E}_{\pi(\theta)}\left[G_{0}\right]$ ．据此，我们可以得到以下结论：通过优化代理优势，有希望找到更好的策略．

#### 6.2.2 邻近策略优化

我们已经知道代理优势与真实的目标相比，在 $\theta=\theta_{k}$ 处有相同的值和梯度．但是，如果 $\theta$ 和 $\theta_{k}$ 差别较远，则近似就不再成立．所以针对代理优势的优化不能离原有的策略太远．基于这一思想，J. Schulman 等在文章 《 Proximal policy optimization algorithms 》中提出了**邻近策略优化** (Proximal Policy Optimization) 算法，将优化目标设计为
$$
\mathrm{E}_{\pi\left(\theta_{k}\right)}\left[\min \left(\frac{\pi\left(A_{t}|S_{t} ; \theta\right)}{\pi\left(A_{t}|S_{t} ; \theta_{k}\right)} a_{\pi\left(\theta_{k}\right)}\left(S_{t}, A_{t}\right), a_{\pi\left(\theta_{k}\right)}\left(S_{t}, A_{t}\right)+\varepsilon\left|a_{\pi\left(\theta_{k}\right)}\left(S_{t}, A_{t}\right)\right|\right)\right]
$$
其中 $\varepsilon \in(0,1)$ 是指定的参数．采用这样的优化目标后，优化目标至多比 $a_{\pi\left(\theta_{k}\right)}\left(S_{t}, A_{t}\right)$ 大 $\varepsilon\left|a_{\pi\left(\theta_{k}\right)}\left(S_{t}, A_{t}\right)\right|$ ，所以优化问题就没有动力让代理优势 $\frac{\pi\left(A_{t}|S_{t} ; \theta\right)}{\pi\left(A_{t}|S_{t} ; \theta_{k}\right)} a_{\pi\left(\theta_{k}\right)}\left(S_{t}, A_{2}\right)$ 变得非常大，可以避免迭代后的策略与迭代前的策略差距过大．

算法 6-5 给出了邻近策略优化算法的简化版本．

>**算法 6-5**： $\quad$ 邻近策略优化算法 (简化版本 )
***********************

输入：环境（无数学描述）
输出：最优策略的估计 $\pi(\boldsymbol{\theta})$
参数：策略更新时目标的限制参数 $\varepsilon(\varepsilon>0)$ ，优化器，折扣因子 $\gamma$ ，控制回合数和回合内步数的参数．

1. （初始化） $\boldsymbol{\theta} \leftarrow$ 任意值， $\mathbf{w} \leftarrow$ 任意值
2. （时序差分更新）对每个回合执行以下操作：
 2.1 用策略 $\pi(\boldsymbol{\theta})$ 生成轨迹
 2.2 用生成的轨迹由 $\mathbf{w}$ 确定的价值函数估计优势函数 $\left(\right.$ 如 $a\left(S_{t}, A_{t}\right) \leftarrow \sum_{\tau=t}^{T-1}(\gamma \lambda)^{\tau-t}\left[U_{\tau: \tau+1}^{(v)}-\right.\left.\left.v\left(S_{\tau} ; \mathbf{w}\right)\right]\right)$
 2.3 （策略更新）更新 $\theta$ 以增大 $\min \left(\frac{\pi\left(A_{t}|S_{t} ; \theta\right)}{\pi\left(A_{t}|S_{t} ; \theta_{k}\right)} a_{\pi\left(\theta_{k}\right)}\left(S_{t}, A_{t}\right), a_{\pi\left(\theta_{k}\right)}\left(S_{t}, A_{t}\right)+\varepsilon\left|a_{\pi\left(\theta_{k}\right)}\left(S_{t}, A_{t}\right)\right|\right)$
 2.4 （价值更新）更新 $\mathbf{w}$ 以减小价值函数的误差（如最小化 $\left.\left[G_{t}-v\left(S_{t} ; \mathbf{w}\right)\right]^{2}\right)$

***********************

在实际应用中，常常加人经验回放．具体的方法是，每次更新策略参数 $\boldsymbol{\theta}$ 和价值参数 $\mathrm{w}$ 前得到多个轨迹，为这些轨迹的每一步估计优势和价值目标，并存储在经验库 $\mathcal{D}$ 中．接着多次执行以下操作：从经验库 $\mathcal{D}$ 中抽取一批经验 $\mathcal{B}$ ，并利用这批经验回放并学习，即从经验库中随机抽取一批经验并用这批经验更新策略参数和价值参数．

>注意：邻近策略优化算法在学习过程中使用的经验都是当前策略产生的经验，所以使用了经验回放的邻近策略优化依然是同策学习算法．

### 6.3 信任域算法

**信任域方法**（Trust Region Method, TRM）是求解非线性优化的常用方法，它将一个复杂的优化问题近似为简单的信任域子问题再进行求解．

本节将介绍三种同策执行者 / 评论者算法：

- 自然策略梯度算法
- 信任域策略优化算法
- Kronecker 因子信任域执行者 / 评论者算法

这三个算法十分接近，它们都是以试图通过优化代理优势，迭代更新策略参数，进而找到最优策略的估计．在优化的过程中，也需要让新的策略和旧的策略不能相差太远．和上节介绍的邻近策略优化相比，它们在代理优势的基础上可以进一步引入信任域，要求新的策略在一个信任域内．本节将介绍信任域的定义（包括用来定义信任域的 $\mathrm{KL}$ 散度的定义），再介绍如何利用信任域实现这些算法．

#### 6.3.1 KL 散度

我们先来看 $\mathrm{KL}$ 散度的定义．回顾重要性采样的章节，我们知道，如果两个分布 $p(x)(x \in \mathcal{X})$ 和 $q(x)(x \in \mathcal{X})$ ，满足对于任意的 $p(x)>0$ ，均有 $q(x)>0$ ，则称分布 $p$ 对 分布 $q$ 绝对连续，记为 $p \ll q$ ．在这种情况下，我们可以定义从分布 $q$ 到分布 $p$ 的 KL 散度 (Kullback-Leibler divergence)：
$$
d_{\mathrm{KL}}(p \| q)=\mathrm{E}_{X \sim p}\left[\ln \frac{p(X)}{q(X)}\right]
$$
当$p$ 和 $q$ 是离散分布时，
$$
d_{\mathrm{KL}}(p \| q)=\sum_{x} p(x) \ln \frac{p(x)}{q(x)}
$$
当 $p$ 和 $q$ 是连续分布时，
$$
d_{\mathrm{KL}}(p \| q)=\int_{x} p(x) \ln \frac{p(x)}{q(x)} \mathrm{d} x
$$
$\mathrm{KL}$ 散度有个性质：相同分布的 $\mathrm{KL}$ 散度为 0 ，即 $d_{\mathrm{KL}}(p \| p)=0$ ．

TODO:信任域A-C算法

#### 重要性采样异策执行者 / 评论者算法

执行者 / 评论者算法可以和重要性采样结合，得到异策执行者 / 评论者算法．本节介绍基于重要性采样的异策执行者 / 评论者算法．

#### 6.4.1 基本的异策算法

本节介绍基于重要性采样的**异策的执行者/评论者算法**（Off-Policy Actor-Critic, OffPAC )．

用 $b(\cdot|\cdot)$ 表示行为策略，则梯度方向可由 $\mathrm{E}_{\pi(\theta)}\left[\Psi_{t} \nabla \ln \pi\left(A_{t}|S_{t} ; \theta\right)\right]$ 变为 $\mathrm{E}_{b}\left[\frac{\pi\left(A_{t}|S_{t} ; \theta\right)}{b\left(A_{t}|S_{t}\right)}\right.\left.\Psi_{t} \nabla \ln \pi\left(A_{t}|S_{t} ; \boldsymbol{\theta}\right)\right]=\mathrm{E}_{b}\left[\frac{1}{b\left(A_{t}|S_{t}\right)} \Psi_{t} \nabla \pi\left(A_{t}|S_{t} ; \boldsymbol{\theta}\right)\right]$ ．这时，更新策略参数 $\boldsymbol{\theta}$ 时就应该试图减小
$-\frac{1}{b\left(A_{t}|S_{t}\right)} \Psi_{t} \nabla \pi\left(A_{t}|S_{t} ; \theta\right)$ ．据此，可以得到异策执行者 / 评论者算法，见算法 6-10．

>**算法 8-10**：$\quad$ 异策动作价值执行者 / 评论者算法
***********************

输入：环境（无数学描述）
输出：最优策略的估计 $\pi(\boldsymbol{\theta})$
参数：优化器（隐含学习率 $\left.\alpha^{(\theta)}, \alpha^{(\mathbf{w})}\right)$ ，折扣因子 $\gamma$ ，控制回合数和回合内步数的参数．

1. （初始化） $\boldsymbol{\theta \leftarrow}$ 任意值， $\boldsymbol{W \leftarrow}$ 任意值
2. （带自益的策略更新）对每个回合执行以下操作：
 2.1 （初始化累积折扣） $I \leftarrow 1$
 2.2 （初始化状态动作对）选择状态 $S$ ，用行为策略 $b(.|S)$ 得到动作 $A$
 2.3 如果回合未结束，执行以下操作：
    1. （采样）根据状态 $S$ 和动作 $A$ 得到采样 $R$ 和下一状态 $S^{\prime}$
    2. （执行）用 $b\left(\cdot|S^{\prime}\right)$ 得到动作 $A^{\prime}$
    3. （估计回报） $U \leftarrow R+\gamma q\left(S^{\prime}, A^{\prime} ; \mathbf{w}\right)$
    4. （策略改进）更新 $\theta$ 以减小 $-\frac{1}{b(A|S)} I q(S, A ; \mathbf{w}) \pi(A|S ; \theta) \quad\left(\right.$ 如 $\boldsymbol{\theta} \leftarrow \boldsymbol{\theta}+\alpha^{(\theta)} I \frac{1}{b(A|S)}q(S, A ; \mathbf{w}) \nabla \pi(A|S ; \boldsymbol{\theta}))$
    5. （更新价值）更新 $\mathbf{w}$ 以减小 $\frac{\pi(A|S ; \boldsymbol{\theta})}{b(A|S)}[U-q(S, A ; \mathbf{w})]^{2}\left(\right.$ 如 $\left.\mathbf{w} \leftarrow \mathbf{w}+\alpha^{(w)} \frac{\pi(A|S ; \boldsymbol{\theta})}{b(A|S)}\right)[U-q(S, A ; \mathbf{w})] \nabla q(S, A ; \mathbf{w})$
    6. （更新累积折扣）$I\leftarrow\gamma I$
    7. （更新状态） $S \leftarrow S^{\prime}, A \leftarrow A^{\prime}$

***********************

#### 6.4.2 带经验回放的异策算法

本节介绍 $\mathrm{Z} .$ Wang 等在文章 《 Sample efficient actor-critic with experience replay》中提出的**带经验回放的执行者 / 评论者算法** ( Actor-Critic with Experiment Replay, $\mathrm{ACER})$ ．如果说 $6.4.1$ 节介绍的基本异策执行者 / 评论者算法是 $6.1.1$ 节介绍的基本同策执行者 / 评论者算法的异策版本，那么本节介绍的带经验回放的异策执行者 / 评论者算法就相当于 $6.1.2$ 节介绍的 $\mathrm{A} 3 \mathrm{C}$ 算法的异策版本．它同样可以支持多个线程的异步学习：每个线程在执行前先同步全局参数，然后独立执行和学习，再利用学到的梯度方向进行全局更新．

6.1.2 节中介绍的执行者 / 评论者算法是基于整个轨迹进行更新的．对于引入行为策略和重采样后，对于目标 $U_{t}$ 的重采样系数变为 $\prod_{\tau=0}^{t} \rho_{\tau}$, 其中 $\rho_{\tau}=\frac{\pi\left(A_{\tau}|S_{\tau} ; \theta\right)}{b\left(A_{\tau}|S_{\tau}\right)}$ ．在这个表达式中，每个 $\rho_{\tau}$ 都有比较大的方差，最终乘积得到的方差会特别大．一种限制方差的方法是控制重采样比例的范围，例如给定一个常数 $c$ ，将重采样比例截断为 $\min \left\{\rho_{\tau}, c\right\}$ ．但是，如果直接将梯度方向中的重采样系数进行截断（例如从 $\mathrm{E}_{b}\left[\rho_{t} \Psi_{t} \nabla \ln \pi\left(A_{t}|S_{t} ; \theta\right)\right]$ 修改为
$\mathrm{E}_{b}\left[\min \left\{\rho_{t}, c\right\} \Psi_{t} \nabla \ln \pi\left(A_{t}|S_{t} ; \boldsymbol{\theta}\right)\right]$ ），会带来偏差．这时候我们可以再加一项来弥补这个偏差．利用恒等式 $\rho=\min \{\rho, c\}+\max \{\rho-c, 0\}$ ，我们可以把梯度 $\mathrm{E}_{b}\left[\rho_{t} \Psi_{t} \nabla \ln \pi\left(A_{t}|S_{t} ; \boldsymbol{\theta}\right)\right]$ 拆成以下两项:
$\mathrm{E}_{b}\left[\min \left\{\rho_{t}, c\right\} \Psi_{t} \nabla \ln \pi\left(A_{t}|S_{t} ; \boldsymbol{\theta}\right)\right]:$ 期望针对行为策略 $b$ ，此项方差是可控的；
$\mathrm{E}_{b}\left[\max \left\{\rho_{t}-c, 0\right\} \Psi_{t} \nabla \ln \pi\left(A_{t}|S_{t} ; \boldsymbol{\theta}\right)\right]$ 即 $\mathrm{E}_{\pi(\theta)}\left[\max \left\{1-c / \rho_{t}, 0\right\} \Psi_{t} \nabla \ln \pi\left(A_{t}|S_{t} ; \theta\right)\right]:$ 采用针对原有目标策略 $\pi(\boldsymbol{\theta})$ 的期望后， $\max \left\{1-c / \rho_{t}, 0\right\}$ 也是有界的 (即 $\max \left\{1-c / \rho_{t}, 0\right\} \leqslant 1$ )．

采用这样的拆分后，两项的方差都是可控的．但是，这两项中其中一项针对的是行为策略，另外一项针对的是原策略，这就要求在执行过程中兼顾这两种策略．

得到梯度方向后，我们希望对这个梯度方向做修正，以免超出范围．为此，用 $\mathrm{KL}$ 散度增加了约束。记在迭代过程中策略参数的指数滑动平均值为 $\theta^{\mathrm{EMA}}$ ，对应的平均策略为 $\pi\left(\mathbf{\theta}^{\mathrm{EMA}}\right)$ ．我
们可以希望迭代得到的新策略参数不要与这个平均策略 $\pi\left(\mathbf{\theta}^{\mathrm{EMA}}\right)$ 参数差别太大．所以，可以限定这两个策略在当前状态 $S_{t}$ 下的动作分布不要差别太大．考虑到 KL 散度可以刻画两个分布直接的差别，所以可以限定新得到的梯度方向（记为 $\mathbf{z}_{t}$ ) 与 $\nabla_{\theta} d_{\mathrm{KL}}\left(\pi\left(\cdot|S_{t} ; \boldsymbol{\theta}^{\mathrm{EMA}}\right) \| \pi\left(\cdot|S_{t} ; \boldsymbol{\theta}\right)\right)$ 的内积不要太大．值得一提的是， $\nabla_{\theta} d_{\mathrm{KL}}\left(\pi\left(\cdot|S_{t} ; \theta^{\mathrm{EMA}}\right) \| \pi\left(\cdot|S_{t} ; \boldsymbol{\theta}\right)\right)$ 实际上有和重采样比例类似的形式：
$$
\begin{aligned}
\nabla_{\theta} & d_{\mathrm{KL}}\left(\pi\left(\cdot|S_{t} ; \theta^{\mathrm{EMA}}\right) \| \pi\left(\cdot|S_{t} ; \theta\right)\right) \\
&=\nabla_{\theta} \sum_{a} \pi\left(a|S_{t} ; \theta^{\mathrm{EMA}}\right) \ln \frac{\pi\left(a|S_{t} ; \theta^{\mathrm{EMA}}\right)}{\pi\left(a|S_{t} ; \theta\right)} \\
&=-\nabla_{\theta} \sum_{a} \pi\left(a|S_{t} ; \theta^{\mathrm{EMA}}\right) \ln \pi\left(a|S_{t} ; \boldsymbol{\theta}\right) \\
&=-\sum_{a} \frac{\pi\left(a|S_{t} ; \theta^{\mathrm{EMA}}\right)}{\pi\left(a|S_{t} ; \theta\right)} \nabla_{\theta} \pi\left(a|S_{t} ; \boldsymbol{\theta}\right)
\end{aligned}
$$
至此，我们可以得到一个确定新的梯度方向的优化问题．记新的梯度方向为 $\mathbf{z}_{t}$ ，定义
$$
\begin{array}{l}
\mathbf{g}_{t}=\min \left\{\rho_{t}, c\right\}\left(U_{t}-v\left(S_{t} ; \mathbf{w}\right)\right) \nabla \ln \pi\left(A_{t}|S_{t} ; \theta\right) \\
\quad+E_{A_t \sim \pi(\theta)}\left[\max \left\{1-\frac{\rho_{t}}{c}\right\}\left(q\left(S_{t}, A_{t} ; \mathbf{w}\right)-v\left(S_{t} ; \mathbf{w}\right)\right) \nabla \ln \pi\left(A_{t}|S_{t} ; \boldsymbol{\theta}\right)\right] \\
\mathbf{k}_{t}=\nabla_{\theta} d_{\mathrm{KL}}\left(\pi\left(\cdot|S_{t} ; \boldsymbol{\theta}^{\mathrm{EMA}}\right) \| \pi\left(\cdot|S_{t} ; \boldsymbol{\theta}\right)\right)
\end{array}
$$
我们一方面希望新的梯度方向 $\mathbf{z}_{t}$ 要和 $\mathbf{g}_{t}$ 尽量接近，另外一方面要满足 $\mathbf{k}_{t}^{\mathrm{T}}\mathbf{z}_t$ ，不超过一个给定的参数 $\delta$ ．这样这个优化问题为
$$
\begin{array}{ll}
\text { minimize } & \frac{1}{2}\left\|\mathbf{g}_{t}-\mathbf{z}\right\|_{2}^{2} \\
\text { over } & \mathbf{z} \\
\text { s.t. } & \mathbf{k}_{t}^{\mathrm{T}} \mathbf{z} \leqslant \delta ．
\end{array}
$$
接下来求解这个优化问题．使用 Lagrange 乘子法，构造函数：
$$
l(\mathbf{z}, \lambda)=\frac{1}{2}\left\|\mathbf{g}_{t}-\mathbf{z}\right\|_{2}^{2}+\lambda\left(\mathbf{k}_{t}^{\mathrm{T}} \mathbf{z}-\delta\right)
$$
将前式代人后式可得 $\lambda_{t}=\frac{\mathbf{k}_{t}^{\mathrm{T}} \mathbf{g}_{t}-\delta}{\mathbf{k}_{t}^{\mathrm{T}} \mathbf{k}_{t}}$ ．由于 Lagrange 乘子应大等于 0 ，所以，Lagrange 乘子应为 $\max \left\{\frac{\mathbf{k}_{t}^{\mathrm{T}} \mathbf{g}_{t}-\delta}{\mathbf{k}_{t}^{T} \mathbf{k}_{t}}, 0\right\}$ ，优化问题的最优解为
$$
\mathbf{z}_{t}=\mathbf{g}_{t}-\max \left\{\frac{\mathbf{k}_{t}^{\mathrm{T}} \mathbf{g}_{t}-\delta}{\mathbf{k}_{t}^{\mathrm{T}} \mathbf{k}_{t}}, 0\right\} \mathbf{k}_{t}
$$
这个方向才是我们真正要用的梯度方向．综合以上分析，我们可以得到带经验回放的执行者 / 评论者算法的一个简化版本．这个算法可以有一个回放因子，可以控制每次运行得到的经验可以回放多少次．算法 6-11 给出了经验回放的线程的算法．对于经验回放的线程所回放的经验是从其他线程已经执行过的线程生成并存储的，这个过程在算法 6-11 中没有展示，但是是这个算法必需的．在存储和回放的时候，不仅要存储和回放状态 $S_{t}$, 动作 $A_{t}$ 、奖励 $R_{t+1}$ 等，还需要存储和回放在状态 $S_{t}$ 产生动作 $A_{t}$ 的概率 $b\left(A_{t}|S_{t}\right)$ ．有了这个概率值，才能计算重采样系数．在价值网络的设计方面，只维护动作价值网络．在需要状态价值的估计时，由动作价值网络计算得到．

>**算法 6-11**：$\quad$ 带经验回放的执行者 / 评
论者算法 (异策简化版本)
***********************

参数: 学习率 $\alpha^{(\theta)}, \alpha^{(\mathrm{w})}$ ，指数滑动平均系数 $\alpha^{\mathrm{EMA}}$ ，重采样因子截断系数 $c$ ，折扣因子 $\gamma$ ，控制回合数和回合内步数的参数．

1. （同步全局参数） $\boldsymbol{\theta}^{\prime} \leftarrow \boldsymbol{\theta}, \quad \mathbf{w}^{\prime} \leftarrow \mathbf{w}$
2. （经验回放）回放存储的经验轨迹 $S_{0}, A_{0}, R_{1}, S_{1}, \ldots, S_{T-1}, A_{T-1}, R_{T}, S_{T}$ ，以及经验对应的行为策略概率 $b\left(A_{t}|S_{t}\right)(t=0,1, \ldots)$
3. 梯度估计
 3.1 为梯度计算初始化:
    1. （初始化目标 $U_{T}$ ）若 $S_{T}$ 是终止状态，则 $U \leftarrow 0$ ；否则 $U \leftarrow \sum_{a} \pi\left(a|S_{t} ; \boldsymbol{\theta}^{\prime}\right)q\left(S_{t}, a ; \mathbf{w}^{\prime}\right)$
    2. （初始化梯度） $\mathrm{g}^{(\mathrm{w})} \leftarrow \mathbf{0}, \mathrm{g}^{(0)} \leftarrow \mathbf{0}$
 3.2 （异步计算梯度） 对 $t=T-1, T-2, \ldots, 0$ ，执行以下内容:
    - （估计目标）计算 $U \leftarrow \gamma U+R_{t+1}$
    - （估计价值梯度方向） $\mathbf{g}^{(\mathbf{w})} \leftarrow \mathbf{g}^{(\mathbf{w})}+\left[U-q\left(S_{t}, A_{t} ; \mathbf{w}^{\prime}\right)\right] \nabla q\left(S_{t}, A_{t} ; \mathbf{w}^{\prime}\right)$
    - （估计策略梯度方向）计算动作价值 $V \leftarrow \sum_{a} \pi\left(a|S_{t} ; \boldsymbol{\theta}^{\prime}\right) q\left(S_{t}, a ; \mathbf{w}^{\prime}\right)$ ，重采样系数 $\rho \leftarrow \frac{\pi\left(A_{t}|S_{t} ; \boldsymbol{\theta}^{\prime}\right)}{b\left(A_{t}|S_{t}\right)}, \quad$ 以及 $\mathbf{g}, \quad \mathbf{k} \leftarrow \nabla_{\mathbf{\theta}} d_{\mathrm{KL}}\left(\pi\left(\cdot|S_{t} ; \boldsymbol{\theta}^{\mathrm{EMA}}\right) \| \pi\left(\cdot|S_{t} ; \boldsymbol{\theta}^{\prime}\right)\right), \quad \mathbf{z} \leftarrow \mathbf{g}-\max \left\{\frac{\mathbf{k}^{\mathrm{T}} \mathbf{g}-\delta}{\mathbf{k}^{\mathrm{T}} \mathbf{k}}, 0\right\} \mathbf{k} .\mathbf{g}^{(\theta)} \leftarrow \mathbf{g}^{(\theta)}+\mathbf{z}$
    - （更新回溯目标） $U \leftarrow \min \{\rho, 1\}\left[U-q\left(S_{t}, A_{t} ; \mathbf{w}^{\prime}\right)\right]+V$
4. （同步更新）更新全局参数．
 4.1 （价值更新） $\mathbf{w} \leftarrow \mathbf{w}+\alpha^{(\mathbf{w})} \mathbf{g}^{(\mathbf{w})}$
 4.2 （策略更新） $\boldsymbol{\theta} \leftarrow \boldsymbol{\theta}+\alpha^{(\theta)} \mathbf{g}^{(\theta)}$
 4.3 （更新平均策略） $\theta^{\mathrm{EMA}} \leftarrow\left(1-\alpha^{\mathrm{EMA}}\right) \theta^{\mathrm{EMA}}+\alpha^{\mathrm{EMA}} \boldsymbol{\theta}$

***********************

TODO:柔性A-C