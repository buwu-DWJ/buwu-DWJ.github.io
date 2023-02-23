## 三． 时序差分价值迭代

本章介绍另外一种学习方法一时序差分更新．时序差分更新和回合更新都是直接采用经验数据进行学习，而不需要环境模型．时序差分更新与回合更新的区别在于，时序差 分更新汲取了动态规划方法中"自益"的思想，用现有的价值估计值来更新价值估计，不需要等到回合结束也可以更新价值估计．所以，时序差分更新既可以用于回合制任务，也可以用于连续性任务．

本章将介绍时序差分更新方法，包括同策时序差分更新方法和异策时序差分更新方法, 每种方法都先介绍简单的单步更新，再介绍多步更新．最后，本章会涉及基于资格迹的学习算法．

### 3.1 同策时序差分更新

本节考虑无模型同策时序差分更新．与无模型回合更新的情况相同，在无模型的情况下动作价值比状态价值更为重要，因为动作价值能够决定策略和状态价值，但是状态价值得不到动作价值．

本节考虑无模型同策时序差分更新。与无模型回合更新的情况相同，在无模型的情况下动作价值比状态价值更为重要，因为动作价值能够决定策略和状态价值，但是状态价值得不到动作价值。
从给定策略 $\pi$ 的情况下动作价值的定义出发，我们可以得到下式:
$$
\begin{aligned}
q_{\pi}(s, a) &=\mathrm{E}_{\pi}\left[G_{t}|S_{t}=s, A_{t}=a\right] \\
&=\mathrm{E}_{\pi}\left[R_{t+1}+\gamma G_{t+1}|S_{t}=S, A_{t}=a\right] \\
&=\mathrm{E}_{\pi}\left[R_{t+1}+\gamma q_{\pi}\left(S_{t+1}, A_{t+1}\right)|S_{t}=s, A_{t}=a\right], \quad s \in \mathcal{S}, a \in \mathcal{A}(s)
\end{aligned}
$$
在上一章的回合更新学习中，我们依据 $q_{\pi}(s, a)=\mathrm{E}_{\pi}\left[G_{t}|S_{t}=s, A_{t}=a\right]$ ，用 Monte Carlo 方法来估计价值函数．为了得到回报样本，我们要从状态动作对 $(s, a)$ 出发一直采样到回合结束．单步时序差分更新将依据 $q_{\pi}(s, a)=\mathrm{E}_{\pi}\left[R_{t+1}+\gamma q_{\pi}\left(S_{t+1}, A_{t+1}\right)|S_{t}=S, A,=a\right]$ ，只需要采样一步，进而用 $U_{t}=R_{t+1}+\gamma q_{\pi}\left(S_{t+1},\cdot \right)$ ，来估计回报样本的值．为了与由奖励直接计算得到的无偏回报样本 $G_{t}$ 进行区别，本书用字母 $U_{t}$ 表示使用自益得到的有偏回报样本．

基于以上分析，我们可以定义时序差分目标．时序差分目标可以针对动作价值定义，也可以针对状态价值定义．对于动作价值，其单步时序差分目标定义为
$$
U_{t:t+1}^{(q)}=R_{t+1}+\gamma q\left(S_{t+1}, A_{t+1}\right)
$$
其中 $U_{t:t+1}^{(g)}$ 的上标 $(q)$ 表示是对动作价值定义的，下标 $t:t+1$ 表示用 $\left(S_{t+1}, A_{t+1}\right)$ 的估计值来估计 $\left(S_{t}, A_{t}\right)$ ．如果 $S_{t+1}$ 是终止状态，默认有 $q\left(S_{t+1}, \cdot\right)=0$ ．这样的时序差分目标可以进一步扩展到多步的情况． n 步时序差分目标 $(n=1,2, \ldots)$ 定义为
$$
U_{t:t+n}^{(q)}=R_{t+1}+\gamma R_{t+2}+\cdots+\gamma^{n-1} R_{t+n}+\gamma^{n} q\left(S_{t+n}, A_{t+n}\right)
$$
在不强调步数的情况下， $U_{t t+n}^{(q)}$ 可以简记为 $U_{t}^{(q)}$ 或 $U_{t}$ ．对于回合制任务，如果回合的步数 $T \leqslant t+n$ ，则我们可以强制让
$$
\begin{array}{ll}
R_{t}=0, & t>T \\
S_{t}=S_{\text {终止}}, & t>T
\end{array}
$$
这样，上述时序差分目标的定义式依然成立，实际上
达到了
$$
U_{t:t+n}^{(q)}=\left\{\begin{array}{ll}
R_{t+1}+\gamma R_{t+2}+\cdots+\gamma^{n-1} R_{t+n}+\gamma^{n} q\left(S_{t+n}, A_{t+n}\right), & t+n<T \\
R_{t+1}+\gamma R_{t+2}+\cdots+\gamma^{T-t-1} R_{T}, & t+n \geqslant T
\end{array}\right.
$$
的效果。本书后文都做这样的假设。类似的是，对于 状态价值，定义 $n$ 步时序差分目标为
$$
U_{t:t+n}^{(v)}=R_{t+1}+\gamma R_{t+2}+\cdots+\gamma^{n-1} R_{t+n}+\gamma^{n} v\left(S_{t+n}\right)
$$
它也可以简记为 $U_{t}^{(\nu)}$ 或 $U_{t}$ ．

#### 3.1.1 时序差分更新策略评估

本节考虑利用时序差分目标来评估给定策略的价值函数．回顾在同策回合更新策略评估中，我们用形如
$$
q\left(S_{t}, A_{t}\right) \leftarrow q\left(S_{t}, A_{t}\right)+\alpha\left[G_{t}-q\left(S_{t}, A_{t}\right)\right]
$$
的增量更新来学习动作价值函数，试图减小 $\left[G_{t}-q\left(S_{t}, A_{t}\right)\right]^{2}$ ．在这个式子中， $G_{t}$ 是回报样本．在时序差分中，这个量就对应着 $U_{t}$ ．因此，只需在回合更新策略评估算法的基础上，将这个增量更新式中的回报 $G_{t}$ 替换为时序差分目标 $U_{t}$ ，就可以得到时序差分策略评估算法了．

时序差分目标既可以是单步时序差分目标，也可以是多步时序差分目标．我们先来看单步时序差分目标．

算法 3-1 给出了用单步时序差分更新评估策略的动作价值的算法．这个算法有一个 $\alpha$ ，它是一个正实数，表示学习率．在上一章的回合更新中，这个学习率往往是 $\frac{1}{c\left(S_{t}, A_t\right)}$ ，它和状态动作对有关，并且不断减小．在时序差分更新中，也可以采用这样不断减小的学习率．不过，考虑到在时序差分算法执行的过程中价值函数会越来越准确，进而基于价值函数估计得到的价值函数也会越来越准确，因此估计值的权重可以越来越大．所以，算法 3-1 采用了一个固定的学习率 $\alpha$ ．这个学习率一般在 $\alpha \in(0,1]$ ．当然，学习率也可以不是常数．在有些问题中，让学习率巧妙地变化能得到更好的效果．引入学习率 $\alpha$ 后，更新式可以表示为:
$$
q\left(S_{t}, A_{t}\right) \leftarrow q\left(S_{t}, A_{t}\right)+\alpha\left[U_{t}-q\left(S_{t}, A_{t}\right)\right] .
$$

>**算法 3-1**： $\quad$ 单步时序差分更新评估策略的动作价值
***********************
输入: 环境（无数学描述）、策略 $\pi$
输出：动作价值函数 $q(s, a), s \in \mathcal{S}, a \in \mathcal{A}$
参数：优化器（隐含学习率 $\alpha$），折扣因子 $\gamma$ ，控制回合数和回合内步数的参数．

1. （初始化） $q(s, a) \leftarrow$ 任意值， $s \in \mathcal{S}, a \in \mathcal{A}$ ．如果有终止状态，令 $q\left(S_{\text {终止 }}, a\right) \leftarrow 0, a \in \mathcal{A}$ ．
2. （时序差分更新）对每个回合执行以下操作
 2.1  （初始化状态动作对）选择状态 $S$ ，再根据输入策略 $\pi$ 确定动作 $A$
 2.2 如果回合未结東（例如未达到最大步数、S不是终止状态）, 执行以下操作:
    1. （采样）执行动作 $A$ ，观测得到奖励 $R$ 和新状态 $S^{\prime}$
    2. 用输入策略 $\pi$ 确定动作 $A^{\prime}$
    3. （计算回报的估计值） $U \leftarrow R+\gamma q\left(S^{\prime}, A^{\prime}\right)$
    4. （更新价值）更新 $q(S, A)$ 以减小 $[U-q(S, A)]^{2}($ 如 $q(S, A) \leftarrow q(S, A)+\alpha[U-q(S,A)])$
    5. $S\leftarrow S^{\prime},A\leftarrow A^{\prime}$．

***********************

在具体的更新过程中，除了学习率 $\alpha$ 和折扣因子 $\gamma$ 外，还有控制回合数和每个回合步数的参数．我们知道，时序差分更新不仅可以用于回合制任务，也可以用于非回合制任务．对于非回合制任务，我们可以自行将某些时段抽出来当作多个回合，也可以不划分回合当作只有一个回合进行更新．类似地，算法 3-2 给出了用单步时序差分方法评估策略状态价值的算法．

>**算法 3-2**$\quad$ 单步时序差分更新评估策略的状态价值
***********************
输入：环境（无数学描述）、策略 $\pi$
输出：状态价值函数 $v(s), s \in \mathcal{S}$
参数：优化器（隐含学习率 $\alpha$ )，折扣因子 $\gamma$ ，控制回合数和回合内步数的参数

1. （初始化） $v(s)\leftarrow$ 任意值， $s\in \mathcal(S)$ ．如果有终止状态， $v(s_{\text{终止}})\leftarrow 0$
2. （时序差分更新）对每个回合执行以下操作
 2.1 （初始化状态）选择状态 $S$
 2.2 如果回合未结東（例如未达到最大步数、S不是终止状态)，执行以下操作:
    1. 根据输入策略 $\pi$ 确定动作 $A$
    2. （采样）执行动作 $A$ ，观测得到奖励 $R$ 和新状态 $S^{\prime}$
    3. （计算回报的估计值） $U \leftarrow R+\gamma v\left(S^{\prime}\right)$
    4. （更新价值）更新 $v(S)$ 以减小 $[U-v(S)]^{2}($ 如 $v(S) \leftarrow v(S)+\alpha[U-v(S)])$
    5. $S \leftarrow S^{\prime}$

***********************

在无模型的情况下，用回合更新和时序差分更新来评估策略都能渐近得到真实的价值函数.它们各有优劣.目前并没有证明某种方法就比另外一种方法更好.根据经验，学习 率为常数的时序差分更新常常比学习率为常数的回合更新更快收敛．不过时序差分更新对环境的 Markov 性要求更高．

我们通过一个例子来比较回合更新和时序差分更新．考虑某个 Markov 奖励过程，我们得到了它的 5 个轨迹样本如下（只显示状态和奖励):
$$
\begin{aligned}
&S_{A}, 0\\
&s_{B}, 0, s_{A}, 0\\
&S_{A}, 1\\
&S_{B}, 0, S_{A}, 0\\
&s_{A}, 1
\end{aligned}
$$
使用回合更新得到的状态价值估计值为 $v\left(s_{A}\right)=\frac{2}{5}, v\left(s_{B}\right)=0$ ，而使用时序差分更新得到的状态价值估计值为 $v\left(s_{A}\right)=v\left(s_{B}\right)=\frac{2}{5}$ ．这两种方法对 $v\left(s_{A}\right)$ 的估计是一样的，但是对于 $v\left(s_{B}\right)$ 的估计有明显不同：回合更新只考虑其中两个含有 $s_{B}$ 的轨迹样本，用这两个轨迹样本回报来估计状态价值；时序差分更新认为状态 $s_{B}$ 下一步肯定会到达状态 $s_{A}$ ，所以可以利用全部轨迹样本来估计 $v\left(s_{A}\right)$ ，进而由 $v\left(s_{A}\right)$ 推出 $v\left(s_{B}\right)$ ．试想，如果这个环境真的是 Markov 决策过程，并且我们正确地识别出了状态空间 $\mathcal{S}=\left\{s_{A}, s_{B}\right\}$ ，那么时序差分更新方法可以用更多的轨迹样本来帮助估计 $s_{B}$ 的状态价值，这样可以更好地利用现有的样本得到更精确的估计．但是，如果这个环境其实并不是 Markov 决策过程，或是 $\left\{s_{A}, s_{B}\right\}$ 并不是其真正的状态空间．那么也有可能 $s_{A}$ 之后获得的奖励值其实和这个轨迹是否到达过 $s_{B}$ 有关，例如如果达到过 $s_B$ 则奖励总是为 0 ．这种情况下，回合更新能够不受到这一错误的影响，只采用正确的信息，从而不受无关信息的干扰，得到正确的估计．这个例子比较了回合更新和时序差分更新的部分利弊。

接下来看如何用多步时序差分目标来评估策略价值．算法 3-3 和算法 3-4 分别给出了用多步时序差分评估动作价值和状态价值的算法．实际实现时，可以让 $S_{t}, A_{t}, R_{,}$ 和 $S_{t+n+1}, A_{t+n+1}, R_{t+n+1}$ 共享同一存储空间，这样只需要 $n+1$ 份存储空间．

>**算法 3-3** $\quad n$ 步时序差分更新评估策略的动作价值
***********************
输入：环境（无数学描述）、策略 $\pi$
输出：动作价值估计 $q(s, a), s \in \mathcal{S}, a \in \mathcal{A}(s)$
参数：步数 $n$ ，优化器 (隐含学习率 $\alpha$ )，折扣因子 $\gamma$ ，控制回合数和回合内步数的参数

1. （初始化） $q(s, a) \leftarrow$ 任意值 $, s \in \mathcal{S}, a \in \mathcal{A}$ ．如果有终止状态，令 $q\left(S_{\text {终止 }}, a\right) \leftarrow 0, a \in \mathcal{A}$
2. （时序差分更新）对每个回合执行以下操作
 2.1 （生成 n 步）用策略 $\pi$ 生成轨迹 $S_0,A_0,R_1,...,R_n,S_n$ （若遇到终止状态，则令后续奖励均为0，状态均为 $s_{\text{终止}}$
 2.2 对于 $t=0,1,2, \ldots$ 依次执行以下操作，直到 $S_{t}=S_{\text{终止}}$ ：
    1. 若 $S_{t+n} \neq S_{\text {终止 }}$ ，则根据 $\pi\left(\cdot|S_{t+n}\right)$ 决定动作 $A_{t+n}$
    2. （更新时序差分目标） $U_{t:t+n}^{(q)} U \leftarrow R_{t+1}+\gamma R_{t+2}+\cdots+\gamma^{n-1} R_{t+n}+\gamma^{n} q\left(S_{t+n}, A_{t+n}\right)$
    3. （更新价值）更新 $q\left(S_{t}, A_{t}\right)$ 以减小 $\left[U-q\left(S_{t}, A_{t}\right)\right]^{2}$
    4. 若 $S_{t+n} \neq S_{\text {终止 }}$ ，则执行 $A_{t+n}$ ，得到奖励 $R_{t+n+1}$ 和下一状态 $S_{t+n+1}$ ；若 $S_{t+n}=S_{\text {终止 }}$, $R_{t+n+1} \leftarrow 0, \quad S_{t+n+1} \leftarrow S_{\text {终止 }}$

***********************

>算法 3-4 $n$ 步时序差分更新评估策略的状态价值
***********************
输入: 环境（无数学描述）、策略 $\pi$
输出：状态价值估计 $v(s), s \in \mathcal{S}$
参数：步数 $n$ ，优化器（隐含学习率 $\alpha$ ），折扣因子 $\gamma$ ，控制回合数和回合内步数的参数

1. (初始化） $v(s) \leftarrow$ 任意值， $s \in \mathcal{S}$ 如果有终止状态，令 $v\left(S_{\text {終止 }}\right) \leftarrow 0$
2. (时序差分更新) 对每个回合执行以下操作
 2.1（生成 $n$ 步）用策略 $\pi$ 生成轨迹 $S_{0}, A_{0}, R_{1}, \ldots, R_{n}, S_{n}$ （若遇到终止状态，则令后续奖励均为 0 ，状态均为 $S_{\text {终止 }}$ ）．
 2.2 对于 $t=0,1,2, \ldots$ 依次执行以下操作，直到 $S_{t}=S_{\text {终止 }}$ :
    1. （更新时序差分目标 $U_{t:t+n}^{(\nu)}$）$U \leftarrow R_{t+1}+\gamma R_{t+2}+\cdots+\gamma^{n-1} R_{t+n}+\gamma^{n} v\left(S_{t+n}\right)$
    2. （更新价值）更新 $v\left(S_{t}\right)$ 以减小 $\left[U-v\left(S_{t}\right)\right]^{2}$
    3. 若 $S_{t+n} \neq S_{\text {终止 }}$ ，则根据 $\pi\left(\cdot|S_{t+n}\right)$ 决定动作 $A_{t+n}$ 并执行，得到奖励 $R_{t+n+1}$ 和下一状态 $S_{t+n+1}$ ；若 $S_{t+n}=S_{\text {终止 }}$ ，令 $R_{t+n+1} \leftarrow 0, S_{t+n+1} \leftarrow S_{\text {终止 }}$．

***********************

#### 3.1.2 SARSA算法

本节我们采用同策时序差分更新来求解最优策略．首先我们来看 “状态 / 动作 / 奖励 状态 / 动作”（State-Action-Reward-State-Action, SARSA）算法．这个算法得名于更新涉及的随机变量 $\left(S_{t}, A_{t}, R_{t+1}, S_{t+1}, A_{t+1}\right)$ ．该算法利用 $R_{t+1}+\gamma q_{t}\left(S_{t+1}, A_{t+1}\right)$ 得到单步时序差分目标 $U_{t}$ ，进而更新 $q\left(S_{t}, A_{t}\right)$ ．该算法的更新式为：
$$
q\left(S_{t}, A_{t}\right) \leftarrow q\left(S_{t}, A_{t}\right)+\alpha\left[U_{t}-q\left(S_{t}, A_{t}\right)\right]
$$
其中 $\alpha$ 是学习率．算法 3-5 给出了用 SARSA 算法求解最优策略的算法．SARSA 算法就是在单步动作价值估计的算法的基础上，在更新价值估计后更新策略．在算法 3-5 中，每当最优动作价值函数的估计 $q$ 更新时，就进行策略改进，修改最优策略的估计 $\pi$ ．策略的提升方法可以采用 $\varepsilon$ 贪心算法，使得 $\pi$ 总是柔性策略．更新结束后，就得到最优动作价值估计和最优策略估计．

>**算法 3-5** $\quad$SARSA 算法求解最优策略（显式更新策略）
***********************
**与 Q-learning 区别在于，SARSA 每次算得的 $a^{\prime}$ 在下一步会使用，也就是这一步通过 Q 函数预测得到的下一步采取的动作 $a^{\prime}$ 也是在下一步更新时真正使用的，而 Q-learning 在下一步更新时的 $a^{\prime}$ 是重新算的（用这一步更新后的 Q 函数算得）**

输入：环境（无数学描述）
输出：最优策略估计 $\pi(a|s)(s \in \mathcal{S}, a \in \mathcal{A}(s))$ 和最优动作价值估计 $q(s, a)(s \in \mathcal{S}, a \in \mathcal{A}(s))$
参数：优化器（隐含学习率 $\alpha$ ），折扣因子 $\gamma$ ，策略改进的参数（如 $\varepsilon$ ），其他控制回合数和回合步数的参数

1. （初始化） $q(s, a) \leftarrow$ 任意值， $s \in \mathcal{S}, a \in \mathcal{A}(s)$ ．如果有终止状态，令 $q\left(s_{\text {终止}}, a\right) \leftarrow 0, a_{\in \mathcal{A}}$ ．用动作价值 $q(s, a)(s \in \mathcal{S}, a \in \mathcal{A}(s))$ 确定策略 $\pi($ 如使用 $\varepsilon$ 贪心策略 $)$
2. （时序差分更新）对每个回合执行以下操作
 2.1 （初始化状态动作对）选择状态 $S$ ，再用策略 $\pi$ 确定动作 $A$
 2.2 如果回合未结東（比如未达到最大步数、$S$ 不是终止状态），执行以下操作:
    1. （采样）执行动作 $A$ ，观测得到奖励 $R$ 和新状态 $S^{\prime}$
    2. 用策略 $\pi$ 确定动作 $A^{\prime}$
    3. （计算回报的估计值） $U \leftarrow R+\gamma q\left(S^{\prime}, A^{\prime}\right)$
    4. （更新价值）更新 $q(S, A)$ 以减小 $[U-q(S, A)]^{2}($ 如 $q(S, A) \leftarrow q(S, A)+\alpha [U-q(S, A)])$
    5. （策略改进）根据 $q(S, \cdot)$ 修改 $\pi(\cdot|S)($ 如 $\varepsilon$ 贪心策略 $)$
    6. $S \leftarrow S^{\prime}, A \leftarrow A^{\prime}$

***********************
其实，在同策迭代的过程中，最优策略也可以不显式存储．另外多步 SARSA 此处省略．

#### 3.1.3 期望SARSA算法

SARSA 算法有一种变化一一期望 SARSA 算法（Expected SARSA）．期望 SARSA算法与 SARSA 算法的不同之处在于，它在估计 $U_{t}$ 时，不使用基于动作价值的时序差分目标 $U_{t+1}^{(q)}=R_{t+1}+\gamma q\left(S_{t+1}, A_{t+1}\right)$ ，而使用基于状态价值的时序差分目标 $U_{t=1}^{(v)}=R_{t+1}+\gamma v\left(S_{t+1}\right)$ ．利用
Bellman 方程，这样的目标又可以表示为
$$
U_{t}=R_{t+1}+\gamma \sum_{a \in A\left(S_{t+1}\right)} \pi\left(a|S_{t+1}\right) q\left(S_{t+1}, a\right)
$$
与 SARSA 算法相比，期望 SARSA 需要计算 $\sum_{a} \pi\left(a|S_{t+1}\right) q\left(S_{t+1}, a\right)$ ，所以计算量比 SARSA 大．但是，这样的期望运算减小了 SARSA 算法中出现的个别不恰当决策．这样，可以避免在更新后期极个别不当决策对最终效果带来不好的影响．因此，期望 SARSA 常常有比 SARSA 更大的学习率．在很多情况下，期望 SARSA 的效果会比 SARSA 稍微好一些．

算法 3-6 给出了期望 SARSA 求解最优策略的算法，它可以视作在单步时序差分状态价值估计算法上修改得到的．期望 SARSA 对回合数和回合内步数的控制方法等都和 SARSA 相同，但是由于期望 SARSA 在更新 $q\left(S_{t}, A_{t}\right)$ 时不需要 $A_{t+1}$ ，所以其循环结构有所简化．算法中让 $\pi$ 保持为 $\varepsilon$ 柔性策略．如果 $\varepsilon$ 很小，那么这个 $\varepsilon$ 柔性策略就很接近于确定性策略，则期望 SARSA 计算的 $\sum_{a} \pi\left(a|S_{t+1}\right) q\left(S_{t+1}, a\right)$ 就很接近于 $q\left(S_{t+1}, A_{t+1}\right)$ ．

>**算法 3-6** $\quad$ 期望 SARSA 求解最优策略
***********************

1. （初始化） $q(s, a) \leftarrow$ 任意值， $s \in \mathcal{S}, a \in \mathcal{A}$ ．如果有终止状态，令 $q\left(S_{\text{终止}}, a\right) \leftarrow 0, a \in \mathcal{A}$ ．用动作价值 $q(\cdot, \cdot)$ 确定策略 $\pi$ (如使用 $\varepsilon$ 贪心策略)
2. （时序差分更新）对每个回合执行以下操作
 2.1 （初始化状态）选择状态 $S$
 2.2 如果回合未结東（比如未达到最大步数、S不是终止状态），执行以下操作：
    1. 用动作价值 $q(S, \cdot)$ 确定的策略（如 $\varepsilon$ 贪心策略）确定动作 $A$
    2. （采样）执行动作 $A$ ，观测得到奖励 $R$ 和新状态 $S^{\prime}$
    3. （用期望计算回报的估计值） $U \leftarrow R+\gamma \sum_{a \in \mathcal{A}\left(s^{\prime}\right)} \pi\left(a|S^{\prime}\right) q\left(S^{\prime}, a\right)$
    4. （更新价值）更新 $q(S, A)$ 以减小 $[U-q(S, A)]^{2}$ （如 $q(S, A) \leftarrow q(S, A)+\alpha\left[U-q(S, A)\right])$
    5. $S \leftarrow S^{\prime}$

***********************

### 3.2 异策时序差分更新

本节介绍异策时序差分更新．异策时序差分更新是比同策差分更新更加流行的算法．特别是 Q 学习算法，已经成为最重要的基础算法之一．

#### 3.2.1 基于重要性采样的异策算法

时序差分策略评估也可以与重要性采样结合，进行异策的策略评估和最优策略求解．对于 $n$ 步时序差分评估策略的动作价值和 SARSA 算法，其时序差分目标 $U_{t:t+n}^{(q)}$ 依赖于轨迹 $\mathrm{S}_{t}, A_{t}, R_{t+1}, S_{t+1}, A_{t+1}, \ldots, S_{t+n}, A_{t+n}$ ．在给定 $S_{t}, A_{t}$ 的情况下，采用策略 $\pi$ 和另外的行为策略 $b$ 生成这个轨迹的概率分别为：
$$
\begin{array}{l}
\operatorname{\mathbb{P}}_{\pi}\left[R_{t+1}, S_{t+1}, A_{t+1}, \ldots, S_{t+n}|S_{t}, A_{t}\right]=\prod_{\tau=t+1}^{t+n-1} \pi\left(A_{\tau}|S_{\tau}\right) \prod_{\tau=t}^{t+n-1} p\left(S_{\tau+1}, R_{\tau+1}|S_{\tau}, A_{\tau}\right) \\
\operatorname{\mathbb{P}}_{b}\left[R_{t+1}, S_{t+1}, A_{t+1}, \ldots, S_{t+n}|S_{t}, A_{t}\right]=\prod_{\tau=t+1}^{t+n-1} b\left(A_{\tau}|S_{\tau}\right) \prod_{\tau=t}^{t+n-1} p\left(S_{\tau+1}, R_{\tau+1}|S_{\tau}, A_{\tau}\right)
\end{array}
$$
它们的比值就是重要性采样比率：
$$
\rho_{t+1:t+n-1}=\frac{\operatorname{\mathbb{P}}_{\pi}\left[R_{t+1}, S_{t+1}, A_{t+1}, \ldots, S_{t+n}|S_{t}, A_{t}\right]}{\operatorname{\mathbb{P}}_{b}\left[R_{t+1}, S_{t+1}, A_{t+1}, \ldots, S_{t+n}|S_{t}, A_{t}\right]}=\prod_{\tau=t+1}^{t+n-1} \frac{\pi\left(A_{\tau}|S_{\tau}\right)}{b\left(A_{\tau}|S_{\tau}\right)}
$$
也就是说，通过行为策略 $b$ 拿到的估计，在原策略 $\pi$ 出现的概率是在策略 $b$ 中出现概率的 $\rho_{t+1:t+n-1}$ 倍．所以，在学习过程中，这样的时序差分目标的权重为 $\rho_{t+1:t+n-1}$ ．将这个权重整合到时序差分策略评估动作价值算法或 SARSA算法中，就可以得到它们的重要性采样的版本．算法 3-7 给出了多步时序差分的版本，单步版本请自行整理．

>**算法 3-7**： $\quad n$ 步时序差分策略评估动作价值或 SARSA 算法
***********************
输入：环境（无数学描述）、策略 $\pi$
输出：动作价值函数 $q(s, a), s \in \mathcal{S}, a \in \mathcal{A}(s)$ ，若是最优策略控制则还要输出策略 $\pi$
参数：步数 $n$ ，优化器 (隐含学习率 $\alpha$ )，折扣因子 $\gamma$ ，控制回合数和回合内步数的参数

1. （初始化） $q(s, a) \leftarrow$ 任意值， $s \in \mathcal{S}, a \in \mathcal{A}$ ．如果有终止状态，令 $q\left(S_{\text {终止 }}, a\right) \leftarrow 0, \quad a \in \mathcal{A}$ ．若是最优策略控制，还应该用 $q$ 决定 $\pi$ （如 $\varepsilon$ 贪心策略）
2. （时序差分更新）对每个回合执行以下操作
 2.1 （行为策略）指定行为策略 $b$ ，使得 $\pi \ll b$
 2.2 （生成 $n$ 步）用策略 $b$ 生成轨迹 $S_{0}, A_{0}, R_{1}, \ldots, R_{n}, S_{n}$ （若遇到终止状态，则令后续奖励均为 0 , 状态均为 $S_{\text {终止}}$ ）
 2.3 对于 $t=0,1,2, \ldots$ 依次执行以下操作，直到 $S_{t}=S_{\text{终止}}$ ：
    1. 若 $S_{t+n} \neq S_{\text{终止}}$ ，则根据 $\pi\left(\cdot|S_{t+n}\right)$ 决定动作 $A_{t+n}$
    2. （更新时序差分目标 $U_{t:t+n}^{(q)}$ ） $U\leftarrow R_{t+1}+\gamma R_{t+2}+\cdots+\gamma^{n-1} R_{t+n}+\gamma^{n} q\left(S_{t+n}, A_{i+n}\right)$
    3. （计算重要性采样比率 $\rho_{t+1:t+n-1}$）$\rho \leftarrow \prod_{\tau=t+1}^{\min \{t+n, T\}-1} \frac{\pi\left(A_{\tau}|S_{\tau}\right)}{b\left(A_{\tau}|S_{\tau}\right)}$
    4. （更新价值）更新 $q\left(S_{t}, A_{t}\right)$ 以减小 $\rho\left[U-q\left(S_{t}, A_{t}\right)\right]^{2}$
    5. （更新策略）如果是最优策略求解算法，需要根据 $q(S, \cdot)$ 修改 $\pi(.|S)$
    6. 若 $S_{t+n} \neq S_{\text {终止 }}$ ，则执行 $A_{t+n}$ ，得到奖励 $R_{t+n+1}$ 和下一状态 $S_{t+n+1}$ ；若 $S_{t+n}=S_{\text {终止}}$ ，则令$R_{t+n+1} \leftarrow 0, \quad S_{t+n+1} \leftarrow S_{\text{终止}}$ ．

***********************

我们可以用类似的方法将重要性采样运用于时序差分状态价值估计和期望 SARSA 算法中．具体而言，考虑从 $t$ 开始的 $n$ 步轨迹 $S_{t}, A_{t}, R_{t+1}, S_{t+1}, A_{t+1}, \ldots, S_{t+n}$ ．在给定 $S_{t}$ 的条件下，采用策略 $\pi$ 和策略 $b$ 生成这个轨迹的概率分别为:
$$
\begin{array}{l}
\operatorname{\mathbb{P}}_{\pi}\left[A_{t}, R_{t+1}, S_{t+1}, A_{t+1}, \ldots, S_{t+n}|S_{t}\right]=\prod_{\tau=t}^{t+n-1} \pi\left(A_{\tau}|S_{\tau}\right) \prod_{\tau=t}^{t+n-1} p\left(S_{\tau+1}, R_{\tau+1}|S_{\tau}, A_{\tau}\right) \\
\operatorname{\mathbb{P}}_{b}\left[A_{t}, R_{t+1}, S_{t+1}, A_{t+1}, \ldots, S_{t+n}|S_{t}\right]=\prod_{\tau=t}^{t+n-1} b\left(A_{\tau}|S_{\tau}\right) \prod_{\tau=t}^{t+n-1} p\left(S_{\tau+1}, R_{\tau+1}|S_{\tau}, A_{\tau}\right)
\end{array}
$$
它们的比值就是时序差分状态评估和期望 SARSA 算法用到的重要性采样比率:
$$
\rho_{t:t+n-1}=\frac{\operatorname{\mathbb{P}}_{\pi}\left[A_{t}, R_{t+1}, S_{t+1}, A_{t+1}, \ldots, S_{t+n}|S_{t}\right]}{\operatorname{\mathbb{P}}_{b}\left[A_{t}, R_{t+1}, S_{t+1}, A_{t+1}, \ldots, S_{t+n}|S_{t}\right]}=\prod_{\tau=t}^{t+n-1} \frac{\pi\left(A_{\tau}|S_{\tau}\right)}{b\left(A_{\tau}|S_{\tau}\right)}
$$

#### 3.2.2 Q学习

3.1.3 节的期望 SARSA 算法将时序差分目标从 SARSA 算法的 $U_{t}=R_{t+1}+\gamma q\left(S_{t+1}, A_{t+1}\right)$ 改为 $U_{t}=R_{t+1}+\gamma \mathrm{E}\left[q\left(S_{t+1}, A_{t+1}\right)|S_{t+1}\right]$ ，从而避免了偶尔出现的不当行为给整体结果带来的负面影响． Q 学习则是从改进后策略的行为出发，将时序差分目标改为
$$
U_{t}=R_{t+1}+\gamma \max _{a \in \mathcal{A}\left(S_{i+1}\right)} q\left(S_{t+1}, a\right)
$$
Q 学习算法认为，在根据 $S_{t+1}$ 估计 $U_{t}$ 时，与其使用 $q\left(S_{t+1}, A_{t+1}\right)$ 或 $v\left(S_{t+1}\right)$ ，还不如使用根据 $q\left(S_{t+1}, \cdot\right)$ 改进后的策略来更新，毕竟这样可以更接近最优价值．因此 Q 学习的更新式不是基于当前的策略，而是基于另外一个并不一定要使用的确定性策略来更新动作价值．从这个意义上看，Q 学习是一个异策算法．算法 3-8 给出了 Q 学习算法．Q 学习算法和期望 SARSA 有完全相同的程序结构，只是在更新最优动作价值的估计 $q\left(S_{t}, A_{t}\right)$ 时使用了不同的方法来计算目标．

>**算法 3-8**：$\quad$ Q 学习算法求解最优策略
***********************

1. （初始化） $q(s, a) \leftarrow$ 任意值， $s \in \mathcal{S}, a \in \mathcal{A}$ ．如果有终止状态，令 $q\left(s_{\text {终止}}, a\right) \leftarrow 0, a \in \mathcal{A}$
2. （时序差分更新）对每个回合执行以下操作
 2.1 （初始化状态）选择状态 $S$
 2.2 如果回合未结東（例如未达到最大步数、S不是终止状态)，执行以下操作：
    1. 用动作价值估计 $q(S, \cdot)$ 确定的策略决定动作 $A($ 如 $\varepsilon$ 贪心策略 $)$
    2. （采样）执行动作 $A$ ，观测得到奖励 $R$ 和新状态 $S^{\prime}$
    3. （用改进后的策略计算回报的估计值） $U \leftarrow R+\gamma \max _{a \in \mathcal{A}\left(s^{\prime}\right)} q\left(S^{\prime}, a\right)$
    4. （更新价值和策略）更新 $q(S, A)$ 以减小 $[U-q(S, A)]^{2}($ 如 $q(S, A) \leftarrow q(S, A)+\alpha[U-q(S, A)])$
    5. $S \leftarrow S^{\prime}$

***********************
当然，Q学习也有多步的版本，其目标为：
$$
U_{t}=R_{t+1}+\gamma R_{t+2}+\cdots+\gamma^{n-1} R_{t+n}+\gamma^{n} \max _{a \in \mathcal{A}\left(S_{t+n}\right)} q\left(S_{t+n}, a\right)
$$
具体省略

#### 3.2.3 双重Q学习

上一节介绍的 Q 学习用 $\max_{a} q\left(S_{t+1}, a\right)$ 来更新动作价值，会导致“最大化偏差 (maximization bias)，使得估计的动作价值偏大．

我们来看一个最大化偏差的例子．下所示的回合制任务中，Markov决策过程的状态空间为 $\mathcal{S}=\left\{s_{\text{开始}}, S_{\text{中间}}\right\}$ ，回合开始时总是处在 $s_{\text {开始 }}$ 状态，可以选择的动作空间 $\mathcal{A}\left(s_{\text{开始}}\right)=\left\{a_{\text {去中间 }}, a_{\text {去终止 }}\right\}$ ．如果选择动作 $a_{\text {去中间 }}$ ，则可以到达状态 $s_{\text {中间 }}$ ，该步奖励为 0 ；如果选择动作 $s_{\text {去终止 }}$ ，则可以达到终止状态并获得奖励 $+1$ ．从状态 $s_{\text {中间 }}$ 出发，有很多可选的动作（例如有 1000 个可选的动作），但是这些动作都指向终止状态，并且奖励都服从均值为 0 、方差为 100 的正态分布．从理论上说，这个例子的最优价值函数为: $v_{*}\left(s_{\text {中间 }}\right)=q_{*}\left(S_{\text {中间 }}, \cdot\right)=0$ ，$v_{*}\left(S_{\text {开始 }}\right)=q_{*}\left(S_{\text {开始 }}, a_{\text {去终止 }}\right)=1$ ，最优策略应当是 $\pi_{*}\left(S_{\text {开始 }}\right)=a_{\text {去终止 }}$ ．但是，如果采用 Q 学习，在中间过程中会走一些弯路：在学习过程中，从 $s_{\text {中间 }}$ 出发的某些动作会采样到比较大的奖励值，从而导致 $\max _{a \in \mathcal{A}\left(s_{\text {中苗 }}\right)} q\left(S_{\text {中伺 }}, a\right)$ 会比较大，使得从 $s_{\text {开始更倾向于选择 }} a_{\text {去中间}}$ ．这样的错误需要大量的数据才能纠正．为了解决这一问题，**双重 Q 学习** ( Double Q Learning）算法使用两个独立的动作价值估计值 $q^{(0)}(\cdot, \cdot)$ 和 $q^{(1)}(\cdot, \cdot)$ ，用 $q^{(0)}\left(S_{t+1}, \arg \max _{a} q^{(1)}\left(S_{t+1}, a\right)\right)$ 或 $q^{(1)}\left(S_{t+1}, \arg \max _{a} q^{(0)}\left(S_{t+1}, a\right)\right)$ 来 代替 Q 学习中的 $\max _{a} q\left(S_{t+1}, a\right)$ ．由于 $q^{(0)}$ 和 $q^{(1)}$ 是相互独立的估计，所以 $\mathrm{E}\left[q^{(0)}\left(S_{t+1}, A^{*}\right)\right]=q\left(S_{t+1}, A^{*}\right)$ ，其中 $A^{*}=\arg \max _{a} q^{(1)}\left(S_{t+1}, a\right)$ ，这样就消除了偏差．在双重学习的过程中， $q^{(0)}$ 和 $q^{(1)}$ 都需要逐渐更新．所以，每步学习可以等概率选择以下两个更新 中的任意一个：

- 使用 $U_{t}^{(0)}=R_{t+1}+\gamma q^{(1)}\left(S_{t+1}, \arg \max_{a} q^{(0)}\left(S_{t+1}, a\right)\right)$ 来更新 $\left(S_{t}, A_{t}\right), \quad$ 以减小 $U_{t}^{(0)}$ 和
$q^{(0)}\left(S_{t}, A_{t}\right)$ 之间的差别 (例如设定损失为 $\left[U_{t}^{(0)}-q^{(0)}\left(S_{t}, A_{t}\right)\right]^{2}$ ，或采用 $q^{(0)}\left(S_{t}, A_{t}\right) \leftarrow$
$q^{(0)}\left(S_{t}, A_{t}\right)+\alpha\left[U_{t}^{(0)}-q^{(0)}\left(S_{t}, A_{t}\right)\right]$ 更新 $)$
- 使用 $U_{t}^{(1)}=R_{t+1}+\gamma q^{(0)}\left(S_{t+1}, \arg \max_{a} q^{(1)}\left(S_{t+1}, a\right)\right)$ 来更新 $\left(S_{t}, A_{t}\right)$ ，以减小 $U_{t}^{(1)}$ 和 $q^{(1)}\left(S_{t}, A_{t}\right)$ 之间的差别 (例如设定损失为 $\left[U_{t}^{(1)}-q^{(1)}\left(S_{t}, A_{2}\right)\right]^{2}$ ，或采用 $q^{(1)}\left(S_{t}, A_{t}\right) \leftarrow q^{(1)}\left(S_{t}, A_{t}\right)+$
$\alpha\left[U_{t}^{(1)}-q^{(1)}\left(S_{t}, A_{t}\right)\right]$ 更新 $)_{0}$

算法 3-9 给出了双重 Q 学习求解最优策略的算法．这个算法中最终输出的动作价值函数是 $q^{(0)}$ 和 $q^{(1)}$ 的平均值，即 $\frac{1}{2}\left(q^{(0)}+q^{(1)}\right)$ ．在算法的中间步骤，我们用这两个估计的和 $q^{(0)}+q^{(1)}$ 来代替平均值 $\frac{1}{2}\left(q^{(0)}+q^{(1)}\right)$ ，在略微简化的计算下也可以达到相同的效果．

>**算法 3-9**： $\quad$ 双重 Q 学习算法求解最优策略
***********************

1. （初始化） $q^{(i)}(s, a) \leftarrow$ 任意值， $s \in \mathcal{S}, a \in \mathcal{A}(s), i \in\{0,1\}$ ．如果有终止状态，则令 $q^{(i)}\left(S_{\text {终止 }}, a\right) \leftarrow$
$0, a \in \mathcal{A}, i \in\{0,1\}$
2. （时序差分更新）对每个回合执行以下操作．
 2.1 （初始化状态）选择状态 $S$
 2.2 如果回合未结束（比如未达到最大步数、 $S$ 不是终止状态），执行以下操作：
    1. 用动作价值 $\left(q^{(0)}+q^{(1)}\right)(S, \cdot)$ 确定的策略决定动作 $A$ (如 $\varepsilon$ 贪心策略)
    2. （采样）执行动作 $A$ ，观测得到奖励 $R$ 和新状态 $S^{\prime}$
    3. （随机选择更新 $q^{(0)}$ 或 $q^{(1)}$）以等概率选择 $q^{(0)}$ 或 $q^{(1)}$ 中的一个动作价值函数作为更新对象，记选择的是 $q^{(i)}, i \in\{0,1\}$
    4. （用改进后的策略更新回报的估计) $U\leftarrow R+\gamma q^{(1-i)}\left(S^{\prime}, \arg \max _{a} q^{(i)}\left(S^{\prime}, a\right)\right)$
    5. （更新动作价值）更新 $q^{(i)}(S, A)$ 以减小 $\left[U-q^{(i)}(S, A)\right]^{2}\left(\right.$ 如 $q^{(i)}(S, A) \leftarrow q^{(i)}(S, A)+\left.\alpha\left[U-q^{(i)}(S, A)\right]\right)$
    6. $S \leftarrow S^{\prime}$

***********************

### 3.3 资格迹

资格迹是一种让时序差分学习更加有效的机制．它能在回合更新和单步时序差分更新之间折中，并且实现简单，运行有效．

#### 3.3.1 $\lambda$ 回报

在正式介绍资格迹之前，我们先来学习 $\lambda$ 回报和基于 $\lambda$ 回报的离线 $\lambda$ 回报算法．给定 $\lambda \in[0,1]$ ，**$\lambda$ 回报** $(\lambda$ return $)$ 是时序差分目标 $U_{t:t+1}, U_{t:t+2}, U_{t:t+3}, \ldots$ 按 $(1-\lambda),(1-\lambda) \lambda,(1-\lambda)$
$\lambda^{2}, \ldots$ 加权平均的结果．对于连续性任务，有
$$
U_{t}^{\lambda}=(1-\lambda) \sum_{n=1}^{+\infty} \lambda^{n-1} U_{t:t+n}
$$
对于回合制任务，则有
$$
U_{t}^{\lambda}=(1-\lambda) \sum_{n=1}^{T-t-1} \lambda^{n-1} U_{t:t+n}+\lambda^{T-t-1} G_{t}
$$
$\lambda$ 回报 $U_{t}^{\lambda}$ 可以看作是回合更新中的目标 $G_{t}$ 和单步时序差分目标 $U_{t:t+1}$ 的推广：当 $\lambda=1$ 时， $U_{t}^{1}=G_{t}$ 就是回合更新的回报；当 $\lambda=0$ 时， $U_{t}^{0}=U_{t:t+1}$ 就是单步时序差分目标．

**离线 $\lambda$ 回报算法**（ offline $\lambda$ -return algorithm ）则是在更新价值（如动作价值 $q\left(S_{t}, A_{t}\right)$ 或状态价值 $\left.v\left(S_{t}\right)\right)$ 时，用 $U_{t}^{\lambda}$ 作为目标，试图减小 $\left[U_{t}^{\lambda}-q\left(S_{t}, A_{t}\right)\right]^{2}$ 或 $\left[U_{t}^{\lambda}-v\left(S_{t}\right)\right]^{2}$ ．它与回合更新算法相比，只是将更新的目标从 $G_{t}$ 换为了 $U_{t}^{\lambda}$ ．对于回合制任务，在回合结束后为每一步 $t=0,1,2, \ldots$ 计算 $U_{t}^{\lambda}$ ，并统一更新价值．因此，这样的算法称为离线算法（ offline algorithm ）．对于连续性任务，没有办法计算 $U_{t}^{\lambda}$ ，所以无法使用离线 $\lambda$ 算法．

由于离线 $\lambda$ 回报算法使用的目标在 $G_{t}$ 和 $U_{t:t+1}$ 间做了折中，所以离线 $\lambda$ 回报算法的效果可能比回合更新和单步时序差分更新都要好．但是，离线 $\lambda$ 回报算法也有明显的缺点:
其一，它只能用于回合制任务，不能用于连续性任务；其二，在回合结束后要计算 $U_{t}^{\lambda}$ $(t=0,1, \ldots, T-1)$ ，计算量巨大．在下一节我们将采用资格迹来弥补这两个缺点．

#### 3.3.2 TD（$\lambda$）

TD ($\lambda$) 是历史上具有重要影响力的强化学习算法，在离线 $\lambda$ 回报算法的基础上改进而来．以基于动作价值的算法为例，在离线 $\lambda$ 回报算法中，对任意的 $n=1,2, \ldots$ ，在更新 $q\left(S_{t-n}, A_{t-n}\right)\left(\right.$ 或 $\left.v\left(S_{t-n}\right)\right)$ 时，时序差分目标 $U_{t-n:t}$ 的权重是 $(1-\lambda) \lambda^{n-1}$ ．虽然需要等到回合结束才能计算 $U_{t-n}^{\lambda}$ ，但是在知道 $\left(S_{t}, A_{t}\right)$ 后就能计算 $U_{t-n:t}$ ．所以我们在知道 $\left(S_{t}, A_{t}\right)$ 后，就可
以用 $q\left(S_{t}, A_{\imath}\right)$ 去更新所有的 $q\left(S_{\tau}, A_{\tau}\right)(\tau=0,1, \ldots, t-1)$ ，并且更新的权重与 $\lambda^{t-\tau}$ 成正比．

据此，给定轨迹 $S_{0}, A_{0}, R_{1}, S_{1}, A_{1}, R_{2}, \ldots$ ，可以引入资格迹 $e_{t}(s, a), s \in \mathcal{S}, a \in \mathcal{A}(s)$ 来表示第 $t$ 步的状态动作对 $\left(S_{t}, A_{t}\right)$ 的单步自益结果 $U_{t:t+1}=R_{t+1}+\gamma q\left(S_{t+1}, A_{t+1}\right)$ 对每个状态动作对 $(s, a)$ $(s \in \mathcal{S}, a \in \mathcal{A}(s))$ 需要更新的权重．**资格迹**（eligibility）用下列递推式定义：当 $t=0$ 时，
$$
e_{0}(s, a)=0, s \in \mathcal{S}, a \in \mathcal{A}
$$
当 $t>0$ 时，
$$
e_{t}(s, a)=\left\{\begin{array}{ll}
1+\beta \gamma \lambda e_{t-1}(s, a), & S_{t}=s, A_{t}=a \\
\gamma \lambda e_{t-1}(s, a), & \text { 其他 }
\end{array}\right.
$$
其中 $\beta \in[0,1]$ 是事先给定的参数．资格迹的表达式应该这么理解：对于历史上的某个状念动作对 $\left(S_{\tau}, A_{\tau}\right)$ ，距离第 $t$ 步间隔了 $t-\tau$ 步， $U_{\tau:t}$ 在 $\lambda$ 回报 $U_{\tau}^{\lambda}$ 中的权重为 $(1-\lambda) \lambda^{t-\tau-1}$ ，并且 $U_{\tau :t}=R_{\tau+1}+\cdots+\gamma^{t-\tau-1} U_{t-1:t}$ ，所以 $U_{t-1:t}$ 是以 $(1-\lambda)(\lambda \gamma)^{t-\tau-1}$ 的比率折算到 $U_{\tau}^{\lambda}$ 中．间隔的步数每增加一步，原先的资格迹大致需要衰减为 $\gamma \lambda$ 倍．对当前最新出现的状态动作对 $\left(S_{t}, A_{t}\right)$ ，大的更新权重则要进行某种强化．强化的强度 $\beta$ 常有以下取值：

- $\beta=1$, 这时的资格迹称为**累积迹**（accumulating trace）
- $\beta=1-\alpha$ （其中 $\alpha$ 是学习率）,这时的资格迹称为**荷兰迹**（dutch trace)
- $\beta=0$ ，这时的资格迹称为**替换迹**（replacing trace）．

当 $\beta=1$ 时，直接将其资格迹加 1 ；当 $\beta=0$ 时，资格迹总是取值在 $[0,1]$ 范围内，所以让其资格迹直接等于 1 也实现了增加，只是增加的幅度没有 $\beta=1$ 时那么大；当 $\beta \in(0,1)$ 时，增加的幅度在 $\beta=0$ 和 $\beta=1$ 之间．

![不同资格迹](img/1.jpg)

利用资格迹，可以得到 $\mathrm{TD}(\lambda)$ 策略评估算法．算法 3-10 给出了用 $\mathrm{TD}(\lambda)$ 评估动作价值的算法．它是在单步时序差分的基础上，加入资格迹来实现的．资格迹也可以和最优策略求解算法结合，例如和 $\mathrm{SARSA}$ 算法结合得到 $\operatorname{SARSA}(\lambda)$ 算法．算法 3-10 中如果没有策略输入，在选择动作时按照当前动作价值来选择最优动作，就是 $\operatorname{SARSA}(\lambda)$ 算法．

>**算法 3-10** $\quad \mathrm{TD}(\lambda)$ 的动作价值评估或 $\operatorname{SARSA}(\lambda)$ 学习
***********************

输入：环境（无数学描述），若评估动作价值则需输入策略 $\pi$ ．
输出：动作价值估计 $q(s, a), s \in \mathcal{S}, a \in \mathcal{A}$
参数：资格迹参数 $\beta$ ，优化器（隐含学习率 $\alpha$ )，折扣因子 $\gamma$ ，控制回合数和回合内步数的参数

1. （初始化）初始化价值估计 $q(s, a) \leftarrow$ 任意值， $s \in \mathcal{S}, a \in \mathcal{A}$ ．如果有终止状态，令 $q\left(s_{\text {终止 }}, a\right) \leftarrow 0, a \in \mathcal{A}$
2. 对每个回合执行以下操作：
 2.1 （初始化资格迹） $e(s, a) \leftarrow 0, s \in \mathcal{S}, a \in \mathcal{A}$
 2.2 （初始化状态动作对）选择状态 $S$ ，再根据输入策略 $\pi$ 确定动作 $A$
 2.3 如果回合未结東（比如未达到最大步数、S不是终止状态)，执行以下操作:
    1. （采样）执行动作 $A$ ，观测得到奖励 $R$ 和新状态 $S^{\prime}$
    2. 根据输入策略 $\pi\left(\cdot|S^{\prime}\right)$ 或是迭代的最优价值函数 $q\left(S^{\prime}, \cdot\right)$ 确定动作 $A^{\prime}$
    3. （更新资格迹） $e(s, a) \leftarrow \gamma \lambda e(s, a), s \in \mathcal{S}, a \in \mathcal{A}(s), e(S, A) \leftarrow 1+\beta e(S, A)$
    4. （计算回报的估计值） $U \leftarrow R+\gamma q\left(S^{\prime}, A^{\prime}\right)$
    5. （更新价值） $q(s, a) \leftarrow q(s, a)+\alpha e(s, a)[U-q(S, A)], s \in \mathcal{S}, a \in \mathcal{A}(s)$
    6. 若 $S^{\prime}=S_{\text {终止 }}$ ，则退出 2.2 步；否则 $S \leftarrow S^{\prime}, A \leftarrow A^{\prime}$．

***********************

资格迹也可以用于状态价值．给定轨迹 $S_{0}, A_{0}, R_{1}, S_{1}, A_{1}, R_{2}, \ldots$ ，资格迹 $e_{t}(s), s \in \mathcal{S}$ 来表示第 $t$ 步的状态动作对 $S_{t}$ 对的单步自益结果 $U_{t:t+1}=R_{t+1}+\gamma v\left(S_{t+1}\right)$ 对每个状态 $s(s \in \mathcal{S})$ 需要更新的权重，其定义为：当 $t=0$ 时，
$$
e_{0}(s)=0, s \in \mathcal{S}
$$
当 $t>0$ 时，
$$
e_{t}(s)=\left\{\begin{array}{ll}
1+\beta \gamma \lambda e_{t-1}(s), & S_{t}=s \\
\gamma \lambda e_{t-1}(s), & \text { 其他. }
\end{array}\right.
$$
算法 3-11 给出了用资格迹评估策略状态价值的算法．

>**算法 5-14** $\quad \operatorname{TD}(\lambda)$ 更新评估策略的状态价值
***********************

输入：环境（无数学描述）、策略 $\pi$
输出：状态价值函数 $v(s), s \in \mathcal{S}$
参数：资格迹参数 $\beta$ ，优化器（隐含学习率 $\alpha$ )，折扣因子 $\gamma$ ，控制回合数和回合内步数的参数

1. （初始化）初始化价值 $v(s) \leftarrow$ 任意值， $s \in \mathcal{S}$ ．如果有终止状态， $v\left(S_{\text {终止 }}\right) \leftarrow 0$
2. 对每个回合执行以下操作:
 2.1 （初始化资格迹） $e(s) \leftarrow 0, s \in \mathcal{S}$
 2.2 （初始化状态）选择状态 S．
 2.3 如果回合未结東（比如未达到最大步数、 $S$ 不是终止状态），执行以下操作:
    1. 根据输入策略 $\pi$ 确定动作 $A$
    2. （采样）执行动作 $A$ ，观测得到奖励 $R$ 和新状态 $S^{\prime}$
    3. （更新资格迹） $e(s) \leftarrow \gamma \lambda e(s), s \in \mathcal{S}, e(S) \leftarrow 1+\beta e(S)$
    4. （计算回报的估计值） $U \leftarrow R+\gamma v\left(S^{\prime}\right)$
    5. （更新价值） $v(s) \leftarrow v(s)+\alpha e(s)[U-v(S)], s \in \mathcal{S}$
    6. $S \leftarrow S^{\prime}$

***********************

$\mathrm{TD}(\lambda)$ 算法与离线 $\lambda$ 回报算法相比，具有三大优点：

- $\mathrm{TD}(\lambda)$ 算法既可以用于回合制任务，又可以用于连续性任务
- $\mathrm{TD}(\lambda)$ 算法在每一步都更新价值估计，能够及时反映变化
- $\mathrm{TD}(\lambda)$ 算法在每一步都有均匀的计算，而且计算量都较小