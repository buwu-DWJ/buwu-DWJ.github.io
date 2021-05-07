# 强化学习基础

## 一． 有模型数值迭代

### 1.1 度量空间与压缩映射

#### 1.1.1 度量空间及其完备性

**度量** ( metric，又称距离 )，是定义在集合上的二元函数．对于集合 $\mathcal{X}$ ，其上的度量
$d: \mathcal{X} \times \mathcal{X} \rightarrow \mathbb{R}$ ，需要满足 $:$

1. 非负性：对任意的 $x^{\prime}, x^{\prime \prime} \in \mathcal{X}$, 有 $d\left(x^{\prime}, x^{\prime \prime}\right) \geqslant 0 ;$
2. 同一性：对任意的 $x^{\prime}, x^{\prime \prime} \in \mathcal{X}$, 如果 $d\left(x^{\prime}, x^{\prime \prime}\right)=0$, 则 $x^{\prime}=x^{\prime \prime} ;$
3. 对称性：对任意的 $x^{\prime}, x^{\prime \prime} \in \mathcal{X}$, 有 $d\left(x^{\prime}, x^{\prime \prime}\right)=d\left(x^{\prime \prime}, x^{\prime}\right) ;$
4. 三角不等式: 对任意的 $x^{\prime}, x^{\prime \prime}, x^{\prime \prime \prime} \in \mathcal{X}$, 有 $d\left(x^{\prime}, x^{\prime \prime \prime}\right) \leqslant d\left(x^{\prime}, x^{\prime \prime}\right)+d\left(x^{\prime \prime}, x^{\prime \prime \prime}\right)$ ．

有序对 $(\mathcal{X}, d)$ 又称为**度量空间** (metric space)．我们来看一个度量空间的例子．考虑有限 Markov 决策过程状态函数 $v(s)(s \in \mathcal{S})$ ，其所有可能的取值组成集合 $\mathcal{V}=\mathbb{R}^{|\mathcal{S}|}$ ，定义 $d_{\infty}$ 如下:
$$
d_{\infty}\left(v^{\prime}, v^{\prime \prime}\right)=\max _{s \in S}\left|v^{\prime}(s)-v^{\prime \prime}(s)\right|, v^{\prime}, v^{\prime \prime} \in \mathcal{V}
$$
可以证明, $d_{\infty}$ 是 $\mathcal{V}$ 上的一个度量．（证明: 非负性、同一性、对称性是显然的．由于 $\forall s \in \mathcal{S}$ 有
$$
\begin{array}{l}
\left|v^{\prime}(s)-v^{\prime \prime \prime}(s)\right| \\
\quad=\left|\left[v^{\prime}(s)-v^{\prime \prime}(s)\right]+\left[v^{\prime \prime}(s)-v^{\prime \prime}(s)\right]\right| \\
\quad \leqslant\left|v^{\prime}(s)-v^{\prime \prime}(s)\right|+\left|v^{\prime \prime}(s)-v^{\prime \prime \prime}(s)\right| \\
\quad \leqslant \max _{s \in S}\left|v^{\prime}(s)-v^{\prime \prime}(s)\right|+\max_{s \in S}\left|v^{\prime \prime}(s)-v^{\prime \prime}(s)\right| \\
\end{array}
$$
可得三角不等式．）所以， $\left(\mathcal{V}, d_{\infty}\right)$ 是一个度量空间．对于一个度量空间，如果 Cauchy 序列都收敛在该空间内，则称这个度量空间是**完备**的(complete)．对于度量空间 $\left(\mathcal{V}, d_{\infty}\right)$ 也是完备的．(证明: 考虑其中任意 Cauchy 列 $\left\{v_{k}: k=0,1,2, \ldots\right\}$ ，即对任意的正实数 $\varepsilon>0$ ，存在正整数 $\kappa$ 使得任意的 $k^{\prime}, k^{\prime \prime}>\kappa$ ，均有 $d_{\infty}\left(v_{k^{\prime}}, v_{k^{\prime}}\right)<\varepsilon_{0}$ 对于 $\forall s \in \mathcal{S},\left|v_{k^{\prime}}(s)-v_{k^{\prime}}(s)\right| \leqslant d_{\infty}\left(v_{k^{\prime}}, v_{k^{\prime}}\right)<\varepsilon$ ，所以 $\left\{v_{k}(s): k=0,1,2, \ldots\right\}$ 是 Cauchy 列．由实数集的完备性，可以知道 $\left\{v_{k}(s): k=0,1,2, \ldots\right\}$ 收敛于某个实数，记这个实数为 $v_{\infty}(s)$ ．所以，对于 $\forall \varepsilon>0$ ，存在正整数 $\kappa(s)$ ，对于任意 $k>\kappa(s)$ ，有 $\left|v_{k}(s)-v_{\infty}(s)\right|<\varepsilon_{0}$ 取 $\kappa(\mathcal{S})=\max _{s \in S} \kappa(s)$ ，有 $d_{\infty}\left(v_{k}, v_{\infty}\right)<\varepsilon$ ，所以 $\left\{v_{k}: k=0,1,2, \ldots\right\}$ 收敛于
$v_{\infty}$ ，而 $v_{\infty} \in \mathcal{V}$ ，完备性得证)．  

#### 1.1.2 压缩映射与Bellman算子

本节介绍压缩映射的定义，并证明 Bellman 期望算子和 Bellman 最优算子是度量空间 $\left(\mathcal{V}, d_{\infty}\right)$ 上的压缩映射．  

对于一个度量空间 $(\mathcal{X}, d)$ 和其上的一个映射 $t: \mathcal{X} \rightarrow \mathcal{X}$ ，如果存在某个实数 $\gamma \in(0,1)$ ，使得对于任意的 $x^{\prime}, x^{\prime \prime} \in \mathcal{X}$ ，都有
$$
d\left(t\left(x^{\prime}\right), t\left(x^{\prime \prime}\right)\right)<\gamma d\left(x^{\prime}, x^{\prime \prime}\right)
$$
则称映射 $t$ 是**压缩映射** ( contraction mapping, 或 Lipschitzian mapping)．其中的实数 $\gamma$ 被称为 Lipschitz 常数．  

#### ※ Bellman期望方程

用状态价值函数表示状态价值函数：
$$
v_{\pi}(s)=\sum_{a} \pi(a|s)\left[r(s, a)+\gamma \sum_{s^{\prime}} p\left(s^{\prime}|s, a\right) v_{\pi}\left(s^{\prime}\right)\right], \quad s \in \mathcal{S}
$$
用动作价值函数表示动作价值函数：
$$
q_{\pi}(s, a)=\sum_{s^{\prime}, r} p\left(s^{\prime}, r|s, a\right)\left[r+\gamma \sum_{a^{\prime}} \pi\left(a^{\prime}|s^{\prime}\right) q_{\pi}\left(s^{\prime}, a^{\prime}\right)\right], \quad s \in \mathcal{S}, a \in \mathcal{A}
$$

#### ※ Bellman最优方程

用最优状态价值函数表示最优状态价值函数：
$$
v_{*}(s)=\max_{a \in \mathcal{A}}\left[r(s, a)+\gamma \sum_{s^{\prime}} p\left(s^{\prime}|s, a\right) v_{*}\left(s^{\prime}\right)\right], \quad s \in \mathcal{S}
$$
用最优动作价值函数表示最优动作价值函数：
$$
q_{*}(s, a)=r(s, a)+\gamma \sum_{s^{\prime}} p\left(s^{\prime}|s, a\right) \max _{a^{\prime}} q_{*}\left(s^{\prime}, a^{\prime}\right), \quad s \in \mathcal{S}, a \in \mathcal{A}
$$

这两个方程都有用状态价值表示状态价值的形式．根据这个形式，我们可以为度量空间 $\left(\mathcal{V}, d_{\infty}\right)$ 定义 Bellman 期望算子 和 Bellman 最优算子．  

给定策略 $\pi(a|s)(s \in \mathcal{S}, a \in \mathcal{A}(s))$ 的 **Bellman 期望算子** $t_{\pi}: \mathcal{V} \rightarrow \mathcal{V}:$
$$
t_{n}(v)(s)=\sum_{a} \pi(a|s)\left[r(s, a)+\gamma \sum_{s^{\prime}} p\left(s^{\prime}|s, a\right) v\left(s^{\prime}\right)\right], \quad s \in \mathcal{S}
$$
**Bellman 最优算子** $t_{.}: \mathcal{V} \rightarrow \mathcal{V}$ :
$$
t_*(v)(s)=\max_{\operatorname{ces} A}\left[r(s, a)+\gamma \sum_{\text {ses }} p\left(s^{\prime}|s, a\right) v_{\cdot}\left(s^{\prime}\right)\right], \quad s \in \mathcal{S}
$$
下面我们就来证明，这两个算子都是压缩映射．  

首先来看 $\mathrm{Bellman}$ 期望算子 $t_{\pi}$ ．由 $t_{\pi}$ 的定义可知，对任意的 $v^{\prime}, v^{\prime \prime} \in \mathcal{V}$ ，有
$$
t_{\pi}\left(v^{\prime}\right)(s)-t_{\pi}\left(v^{\prime \prime}\right)(s)=\gamma \sum_{a} \pi(a|s) \sum_{s^{\prime}} p\left(s^{\prime}|s, a\right)\left[v^{\prime}\left(s^{\prime}\right)-v^{\prime \prime}\left(s^{\prime}\right)\right]
$$
所以
$$
\left|t_{\pi}\left(v^{\prime}\right)(s)-t_{\pi}\left(v^{\prime \prime}\right)(s)\right| \leqslant \gamma \sum_{a} \pi(a|s) \sum_{s^{\prime}} p\left(s^{\prime}|s, a\right) \max _{s^{\prime}}\left|v^{\prime}\left(s^{\prime}\right)-v^{\prime \prime}\left(s^{\prime}\right)\right|=\gamma d_{\infty}\left(v^{\prime}, v^{\prime \prime}\right)
$$
考虑到 $s$ 是任取的，所以有
$$
d_{\infty}\left(t_{\pi}\left(v^{\prime}\right), t_{\pi}\left(v^{\prime \prime}\right)\right) \leqslant \gamma d_{\infty}\left(v^{\prime}, v^{\prime \prime}\right)
$$
当 $\gamma<1$ 时， $t_{n}$ 就是压缩映射．接下来看 Bellman 最优算子 $t_{*}$ ．要证明 $t_{*}$ 是压缩映射，需要用到下列不等式:
$$
\left|\max_{a} f^{\prime}(a)-\max_{a} f^{\prime \prime}(a)\right| \leqslant \max_{a}\left|f^{\prime}(a)-f^{\prime \prime}(a)\right|
$$
其中 $f^{\prime}$ 和 $f^{\prime \prime}$ 是任意的以 $a$ 为自变量的函数。(证明: 设 $a^{\prime}=\arg \max_{o} f^{\prime}(a)$, 则
$$
\max _{o} f^{\prime}(a)-\max_{a} f^{\prime \prime}(a)=f^{\prime}\left(a^{\prime}\right)-\max _{a} f^{\prime \prime}(a) \leqslant f^{\prime}\left(a^{\prime}\right)-f^{\prime \prime}\left(a^{\prime}\right) \leqslant \max_{a}\left|f^{\prime}(a)-f^{\prime \prime}(a)\right|
$$
同理可证 $\max _{a} f^{\prime \prime}(a)-\max_{a} f^{\prime}(a) \leqslant \max _{a}\left|f^{\prime}(a)-f^{\prime \prime}(a)\right|$ ，于是不等式得证． $)$ 利用这个不等
式，对任意的 $v^{\prime}, v^{\prime \prime} \in \mathcal{V}$ ，有
$$
\begin{aligned}
\qquad t_{*}\left(v^{\prime}\right)(s)&-t_{*}\left(v^{\prime \prime}\right)(s) \\
&=\max _{a \in A}\left[r(s, a)+\gamma \sum_{s^{\prime} \in \mathcal{S}} p\left(s^{\prime}|s, a\right) v^{\prime}\left(s^{\prime}\right)\right]-\max_{a \in \mathcal{A}}\left[r(s, a)+\gamma \sum_{s^{\prime} \in S} p\left(s^{\prime}|s, a\right) v^{\prime \prime}\left(s^{\prime}\right)\right] \\
&\leqslant \max _{a^{\prime} \in \mathcal{A}}\left|\gamma \sum_{s^{\prime} \in \mathcal{S}} p\left(s^{\prime}|s, a^{\prime}\right)\left(v^{\prime}\left(s^{\prime}\right)-v^{\prime \prime}\left(s^{\prime}\right)\right)\right| \\
&\leqslant \gamma \max _{a^{\prime} \in A}\left|\sum_{s^{\prime} \in S} p\left(s^{\prime}|s, a^{\prime}\right)\right| \max _{s^{\prime} \in S}\left|v^{\prime}\left(s^{\prime}\right)-v^{\prime \prime}\left(s^{\prime}\right)\right| \\
&\leqslant \gamma d_{\infty}\left(v^{\prime}, v^{\prime \prime}\right)
\end{aligned}
$$
进而易知 $\left|t_{*}\left(v^{\prime}\right)(s)-t_{*}\left(v^{\prime \prime}\right)(s)\right| \leqslant \gamma d_{\infty}\left(v^{\prime}, v^{\prime \prime}\right)$ ，所以 $t_*$ 是压缩映射．

#### 1.1.3 Banach不动点定理

对于度量空间 $(\mathcal{X}, d)$ 上的映射 $t: \mathcal{X} \rightarrow \mathcal{X}$ ，如果 $x \in \mathcal{X}$ 使得 $t(x)=x$ ，则称 $x$ 是映射 $t$ 的**不动点** (fix point)．  

例如，策略 $\pi$ 的状态价值函数 $v_{\pi}(s)(s \in \mathcal{S})$ 满足 Bellman 期望方程，是 Bellman 期望算子 $t_{\pi}$ 的不动点．最优状态价值 $v_{*}(s)(s \in \mathcal{S})$ 满足 Bellman 最优方程，是 Bellman 最优算子 $t_*$ 的不动点．  

完备度量空间上的压缩映射有非常重要的结论: Banach 不动点定理． **Banach 不动点定理**（Banach fixed-point theorem, 又称压缩映射定理， compressed mapping theorem) 的内容是: $(\mathcal{X}, d)$ 是非空的完备度量空间, $t: \mathcal{X} \rightarrow \mathcal{X}$ 是一个压缩映射，则映射 $t$ 在 $\mathcal{X}$ 内有且仅有一个不动点 $x_{+\infty}$ ．更进一步，这个不动点可以通过下列方法求出：从 $\mathcal{X}$ 内的任意一个元素 $x_{0}$ 开始，定义迭代序列 $x_{k}=t\left(x_{k-1}\right)(k=1,2,3, \ldots)$ ，这个序列收敛，且极限为 $x_{+\infty}$ ．  

证明：考虑任取的 $x_{0}$ 及其确定的列 $\left\{x_{k}: k=0,1, \ldots\right\}$ ，我们可以证明它是 Cauchy 序列．对于任意的 $k^{\prime}, k^{\prime \prime}$ 且 $k^{\prime}<k^{n}$ ，用距离的三角不等式和非负性可知，
$$
d\left(x_{k}, x_{k^{\prime}}\right) \leqslant d\left(x_{k^{\prime}}, x_{k^{\prime}+1}\right)+d\left(x_{k^{\prime}+1}, x_{k^{\prime}+2}\right)+\cdots+d\left(x_{k^{\prime\prime}-1}, x_{k^{\prime\prime}}\right) \leqslant \sum_{k=k^{\prime}}^{+\infty} d\left(x_{k+1}, x_{k}\right)
$$
再反复利用压缩映射可知，对于任意的正整数 $k$ 有 $d\left(x_{k+1}, x_{k}\right) \leqslant \gamma^{k} d\left(x_{1}, x_{0}\right)$ ，代人得：
$$
d\left(x_{k^{\prime}}, x_{k^{\prime\prime}}\right) \leqslant \sum_{k=k^{\prime}}^{+\infty} d\left(x_{k+1}, x_{k}\right) \leqslant \sum_{k=k^{\prime}}^{+\infty} \gamma^{k} d\left(x_{1}, x_{0}\right)=\frac{\gamma^{k^{\prime}}}{1-\gamma} d\left(x_{1}, x_{0}\right)
$$
由于 $\gamma \in(0,1)$ ，所以上述不等式右端可以任意小，得证．  

Banach 不动点定理给出了求完备度量空间中压缩映射不动点的方法：从任意的起点开始，不断迭代使用压缩映射，最终就能收敛到不动点．并且在证明的过程中，还给出了收敛速度，即迭代正比于 $\gamma^{k}$ 的速度收敛 $($ 其中 $k$ 是迭代次数)．在 $1.1.1$ 节我们已经证明 $\left(\mathcal{V}, d_{\infty}\right)$ 是完备的度量空间，而 $1.1.2$ 节又证明了 Bellman 期望算子和 $\mathrm{Bellman}$ 最优算子是压缩映射，那么就可以用迭代的方法求 Bellman 期望算子和 Bellman 最优算子的不动点．于 $\mathrm{Bellman}$ 期望算子的不动点就是策略价值，Bellman 最优算子的不动点就是最优价值，所以这就意味着我们可以用迭代的方法求得策略的价值或最优价值．在后面的小节中，就来具体看看求解的算法．  

### 1.2 有模型策略迭代

本节介绍在给定动力系统 $p$ 的情况下的策略评估、策略改进和策略迭代．策略评估、策略改进和策略迭代分别指以下操作．

- **策略评估**（policy evaluation): 对于给定的策略 $\pi$ ，估计策略的价值, 包括动作价
和状态价值
- **策略改进**（policy improvement): 对于给定的策略 $\pi$ ，在已知其价值函数的情况
找到一个更优的策略
- **策略迭代**（policy iteration): 综合利用策略评估和策略改进，找到最优策略

#### 1.2.1 策略评估

本节介绍如何用迭代方法评估给定策略的价值函数．如果能求得状态价值函数，那么就能很容易地求出动作价值函数．由于状态价值函数只有 $|S|$ 个自变量，而动作价值函数有
$|\mathcal{S}| \times|\mathcal{A}|$ 个自变量，所以存储状态价值函数比较节约空间．  

用迭代的方法评估给定策略的价值函数的算法如算法 1-1 所示．算法 1-1 一开始初始化状态价值函数 $v_{0}$ ，并在后续的迭代中用 $\mathrm{Bellman}$ 期望方程的表达式更新一轮所有状态的状态价值函数．这样对所有状态价值函数的一次更新又称为一次**扫描**（sweep)．在第 $k$ 次扫描时，用 $v_{k-1}$ 的值来更新 $v_{k}$ 的值，最终得到一系列的 $v_{0}, v_{1}, \nu_{2}, \ldots$ ．  

>**算法 1-1**：$\quad$有模型策略评估迭代算法
***********************
输入: 动力系统 $p$, 策略 $\pi$  
输出：状态价值函数 $v_{\pi}$ 的估计值  
参数: 控制迭代次数的参数（如误差容忍度 $\vartheta_{\max }$ 或最大迭代次数 $\left.k_{\max }\right)$  

1. (初始化) 对于 $s \in \mathcal{S}$ ，将 $v_{0}(s)$ 初始化为任意值 (比如 0 )．如果有终止状态，将终止状态初始化为 0，即 $v_{0}\left(S_{\text {终止}}\right) \leftarrow 0$  
2. (迭代) 对于 $k \leftarrow 0,1,2,3, \ldots$ ，迭代执行以下步骤  
2.1 对于 $s \in \mathcal{S}$, 逐一更新 $v_{k+1}(s) \leftarrow \sum_{a} \pi(a|s) q_{k}(s, a)$ ，其中$q_{k}(s, a) \leftarrow r(s, a)+\gamma \sum_{s^{\prime}} p\left(s^{\prime}|s, a\right) v_{k}\left(s^{\prime}\right)$．  
2.2 如果满足迭代终止条件 (如对 $s \in \mathcal{S}$ 均有 $\left|v_{k+1}(s)-v_{k}(s)\right|<\vartheta_{\max }$ ，或达到最大迭代次数
$\left.k=k_{\max }\right)$ ，则跳出循环

***********************
值得一提的是，算法 1-1 没必要为每次捉描都重新分配一套空间来存储．一种优化的方法是，设置奇数次迭代的存储空间和偶数次迭代的存储空间，一开始初始化偶数次存储空间，当 $k$ 是奇数时，用偶数次存储空间来更新奇数次存储空间; 当 $k$ 是偶数时, 用奇数次存储空间来更新偶数次存储空间．这样，一共只需要两套存储空间就可以完成算法．

#### 1.2.2 策略改进

对于给定的策略 $\pi$ ，如果得到该策略的价值函数，则可以用策略改进定理得到一个改进的策略．  

策略改进定理的内容如下：对于策略 $\pi$ 和 $\pi^{\prime}$ ，如果
$$
v_{\pi}(s) \leqslant \sum_{a} \pi^{\prime}(a|s) q_{\pi}(s, a), \quad s \in \mathcal{S}
$$
则 $\pi \leqslant \pi^{\prime}$ ，即
$$
v_{\pi}(s) \leqslant v_{\pi^{\prime}}(s), \quad s \in \mathcal{S}
$$
在此基础上，如果存在状态使得第一式的不等号是严格小于号，那么就存在状态使得第二式中的不等号也是严格小于号．  

**证明**: 考虑到第一个不等式等价于
$$
v_{\pi}(s)=\mathrm{E}_{\pi^{\prime}}\left[v_{\pi}\left(S_{t}\right)|S_{t}=s\right] \leqslant \mathrm{E}_{\pi^{\prime}}\left[q_{\pi}\left(S_{t}, A_{t}\right)|S_{t}=s\right], \quad s \in \mathcal{S}
$$
其中的期望是针对用策略 $\pi^{\prime}$ 生成的轨迹中，选取 $S_{t}=s$ 的那些轨迹而言的．进而有
$$
\begin{array}{l}
\mathrm{E}_{\pi^{\prime}}\left[\nu_{\pi}\left(S_{t+\tau}\right)|S_{t}=s\right] \\
\quad=\mathrm{E}_{\pi^{\prime}}\left[\mathrm{E}_{\pi^{\prime}}\left[v_{\pi}\left(S_{t+\tau}\right)|S_{t+\tau}\right]|S_{t}=s\right] \\
\quad \leqslant \mathrm{E}_{\pi^{\prime}}\left[\mathrm{E}_{\pi^{\prime}}\left[q_{\pi}\left(S_{t+\tau}, A_{t+\tau}\right)|S_{t+\tau}\right]|S_{t}=s\right] \\
\quad=\mathrm{E}_{\pi^{\prime}}\left[q_{\pi}\left(S_{t+\tau}, A_{t+\tau}\right)|S_{t}=s\right], \quad s \in \mathcal{S}, \tau=0,1,2, \ldots
\end{array}
$$
考虑到
$$
\mathrm{E}_{\pi^{\prime}}\left[q_{\pi}\left(S_{t+\tau}, A_{t+\tau}\right)|S_{t}=s\right]=\mathrm{E}_{\pi^{\prime}}\left[R_{t+\tau+1}+\gamma v_{\pi}\left(S_{t+\tau+1}\right)|S_{t}=s\right], \quad s \in \mathcal{S}, \tau=0,1,2, \ldots
$$
所以
$$
\mathrm{E}_{\pi^{\prime}}\left[v_{\pi}\left(S_{t+\tau}\right)|S_{t}=s\right] \leqslant \mathrm{E}_{\pi^{\prime}}\left[R_{t+\tau+1}+\gamma v_{\pi}\left(S_{t+\tau+1}\right)|S_{t}=s\right], \quad s \in \mathcal{S}, \tau=0,1,2, \ldots
$$
进而有
$$
\begin{array}{l}
v_{\pi}(s)=\mathrm{E}_{\pi^{\prime}}\left[v_{\pi}\left(S_{t}\right)|S_{t}=s\right] \\
\leqslant \mathrm{E}_{\pi^{\prime}}\left[R_{t+1}+\gamma v_{\pi}\left(S_{t+1}\right)|S_{t}=s\right] \\
\leqslant \mathrm{E}_{\pi^{\prime}}\left[R_{t+1}+\gamma \mathrm{E}_{\pi^{\prime}}\left[R_{t+2}+\gamma v_{\pi}\left(S_{t+2}\right)|S_{t}=s\right]|S_{t}=s\right] \\
\leqslant \mathrm{E}_{\pi^{\prime}}\left[R_{t+1}+\gamma R_{t+2}+\gamma^{2} v_{\pi}\left(S_{t+2}\right)|S_{t}=s\right] \\
\leqslant \mathrm{E}_{\pi^{\prime}}\left[R_{t+1}+\gamma R_{t+2}+\gamma^{2} R_{t+3}+\gamma^{3} v_{\pi}\left(S_{t+4}\right)|S_{t}=s\right] \\
\ldots \\
\leqslant \mathrm{E}_{\pi^{\prime}}\left[R_{t+1}+\gamma R_{t+2}+\gamma^{2} R_{t+3}+\gamma^{3} R_{t+4}+\cdots|S_{t}=s\right] \\
=\mathrm{E}_{\pi^{\prime}}\left[G_{t}|S_{t}=s\right] \\
=v_{\pi^{\prime}}(s), \quad s \in \mathcal{S}
\end{array}
$$
严格不等号的证明类似．  

对于一个确定性策略 $\pi$ ，如果存在着 $s \in \mathcal{S}, a \in \mathcal{A}$ ，使得 $q_{\pi}(s, a)>v_{\pi}(s)$ ，那么我们可以构造一个新的确定策略 $\pi^{\prime}$ ，它在状态 $s$ 做动作 $a$ ，而在除状态 $s$ 以外的状态的动作都和策略一样．可以验证，策略 $\pi$ 和 $\pi^{\prime}$ 满足策略改进定理的条件．这样，我们就得到了一个比策略 $\pi$ 更好的策略 $\pi^{\prime}$ ．这样的策略更新算法可以用算法 1-2 来表示．  

>**算法 1-2**：$\quad$有模型策略改进算法
***********************
输入: 动力系统 $p$ ，策略 $\pi$ 及其状态价值函数 $v_{\pi}$  
输出：改进的策略 $\pi^{\prime}$ ，或策略 $\pi$ 已经达到最优的标志  

1. 对于每个状态 $s \in \mathcal{S}$ ，执行以下步骤:  
 1.1 为每个动作 $a \in \mathcal{A}$ ，求得动作价值函数 $q_{\pi}(s, a) \leftarrow r(s, a)+\gamma \sum_{s^{\prime}} p\left(s^{\prime}|s, a\right) v_{\pi}\left(s^{\prime}\right)$  
 1.2 找到使得 $q_{\pi}(s, a)$ 最大的动作 $a$ ，即 $\pi^{\prime}(s)=\arg\max_{a}q(s, a)$
2. 如果新箱略 $\pi^{\prime}$ 和旧策略 $\pi$ 相同，则说明旧策略巳是最优; 否则，输出改进的新策略 $\pi^{\prime}$

***********************
值得一提的是，在算法 1-2 中，旧策略 $\pi$ 和新策略 $\pi^{\prime}$ 只在某些状态上有不同的动作值, 新策略 $\pi^{\prime}$ 可以很方便地在旧策略 $\pi$ 的基础上修改得到．所以，如果在后续不需要使用旧策略的情况下，可以不为新策略分配空间．

#### 1.2.3 策略迭代

策略迭代是一种综合利用策略评估和策略改进求解最优策略的迭代方法．  

见算法 1-3 ，策略迭代从一个任意的确定性策略 $\pi_{0}$ 开始，交替进行策略评估和策略改进．这里的策略改进是严格的策略改进，即改进后的策略和改进前的策略是不同的 对于状态空间和动作空间均有限的 Markov 决策过程，其可能的确定性策略数是有限的．由于确定性策略总数是有限的，所以在迭代过程中得到的策略序列 $\pi_{0}, \pi_{1}, \pi_{2}, \ldots$ 一定能收敛，使得到某个 $k$ ，有 $\pi_{k}=\pi_{k+1}$ (即对任意的 $s \in \mathcal{S}$ 均有 $\left.\pi_{k+1}(s)=\pi_{k}(s)\right)$ ．由于在 $\pi_{k}=\pi_{k+1}$ 的情况下， $\pi_{k}(s)=\pi_{k+1}(s)=\arg \max _{a} q_{\pi_{k}}(s, a)$ ，进而 $v_{\pi_{k}}(s)=\max_{a} q_{\pi_{k}}(s, a)$ ，满足 Bellman 最优方程．因此， $\pi_{k}$ 就是最优策略．这样就证明了策略迭代能够收敛到最优策略．

>**算法 1-3**：$\quad$有模型策略迭代
***********************
输入: 动力系统 $p$  
输出：最优策略  

1. (初始化）将策略 $\pi_{0}$ 初始化为一个任意的确定性策略．
2. (迭代) 对于 $k \leftarrow 0,1,2,3, \ldots$, 执行以下步骤  
 2.1 （策略评估）使用策略评估算法，计算策略 $\pi_{k}$ 的状态价值函数 $v_{\pi_{k}}$  
 2.2 （策略更新）利用状态价值函数 $v_{\pi_{k}}$ 改进确定性策略 $\pi_{k}$ ，得到改进的确定性策略 $\pi_{k+1}$ ．如果 $\pi_{k+1}=\pi_{k}$ ．（即对任意的 $s \in \mathcal{S}$ 均有 $\pi_{k+1}(s)=\pi_{k}(s)$），则迭代完成，返回策略 $\pi_{k}$ 为最终的最优策略．

***********************
策略迭代也可以通过重复利用空间来节约空间．为了节约空间，在各次迭代中用相同的空间 $v(s)(s \in \mathcal{S})$ 来存储状态价值函数，用空间 $\pi(s)(s \in \mathcal{S})$ 来存储确定性策略．

### 1.3 有模型价值迭代

价值迭代是一种利用迭代求解最优价值函数进而求解最优策略的方法．在 $1.2.1$ 节介绍的策略评估中，迭代算法利用 Bellman 期望方程迭代求解给定策略的价值函数．与之相对，本节将利用 Bellman 最优方程迭代求解最优策略的价值函数，并进而求得最优策略．

与策略评估的情形类似，价值迭代算法有参数来控制迭代的终止条件，可以是误差容忍度 $\vartheta_{\max }$ 或是最大迭代次数 $k_{\max }$ ．

算法 1-4 给出了一个价值迭代算法．这个价值迭代算法中先初始化状态价值函数，然后用 Bellman 最优方程来更新状态价值函数．根据第 1.1 节的证明，只要迭代次数足够多，最终会收敘到最优价值函数．得到最优价值函数后，就能很轻易地给出确定性的最优策略．

>**算法 1-4**：$\quad$ 有模型价值迭代算法
***********************
输入：动力系统 $p$  
愉出：最优策略估计 $\pi$  
参数：策略评估需要的参数

1. (初始化) $v_{0}(s) \leftarrow$ 任意值， $s \in \mathcal{S}$ ．如果有终止状态， $v_{0}\left(s_{\text {终止 }}\right) \leftarrow 0$
2. (迭代) 对于 $k \leftarrow 0,1,2,3, \ldots$ ，执行以下步骤  
 2.1 对于 $s \in \mathcal{S}$ ，逐一更新 $v_{k+1}(s) \leftarrow \max_{a}\left\{r(s, a)+\gamma \sum_{s^{\prime}} p\left(s^{\prime}|s, a\right) v_{k}\left(s^{\prime}\right)\right\}$  
 2.2 如果满足误差容忍度 (即对于 $s \in \mathcal{S}$ 均有 $\left.\left|v_{k+1}(s)-v_{k}(s)\right|<\vartheta\right)$ 或达到最大迭代次数 (即
$\left.k=k_{\max }\right)$ ，则跳出循环
3. (策略) 根据价值函数输出确定性策略 $\pi_{*}$ ，使得$\pi_{*}(s) \leftarrow \underset{a}{\arg \max }\left\{r(s, a)+\gamma \sum_{s^{\prime}} p\left(s^{\prime}|s, a\right) v_{k+1}\left(s^{\prime}\right)\right\}, \quad s \in \mathcal{S}_{0}$

***********************
与策略评估的迭代求解类似，价值迭代也可以在存储状态价值函数时重复使用空间．算法 1-5 给出了重复使用空间以节约空间的版本．

>**算法 1-5**：$\quad$有模型价值迭代 (节约空间的版本 )
***********************
输入: 动力系统 $p$  
输出：最优策略  
参数：策略评估需要的参数

1. (初始化) $v_{0}(s) \leftarrow$ 任意值, $s \in \mathcal{S}$ ．如果有终止状态， $v_{0}\left(s_{\text {终止 }}\right) \leftarrow 0$
2. (迭代) 对于 $k \leftarrow 0,1,2,3, \ldots$ ，执行以下步骤  
    2.1 对于使用误差容忍度的情况，初始化本次迭代观测到的最大误差 $\vartheta \leftarrow 0$  
    2.2 对于 $s \in \mathcal{S}$ 执行以下操作:
     1. 计算新状态价值 $v_{\text {新}}\leftarrow\max_{a}\left\{r(s, a)+\gamma \sum_{s^{\prime}} p\left(s^{\prime}|s, a\right) v\left(s^{\prime}\right)\right\}$  
     2. 对于使用误差容忍度的情况，更新本次迭代观测到的最大误差 $\vartheta \leftarrow \max \left\{\vartheta,\left|v_{\text {新}}-v(s)\right|\right\}$  
     3. 更新状态价值函数 $v(s) \leftarrow v_{\text {新}}$  
 2.3 如果满足误差容忍度（即 $\left.\vartheta<\vartheta_{\max }\right)$ 或达到最大迭代次数 $\left(\right.$ 即 $\left.k=k_{\max }\right)$ ，则跳出循环
3. (策略) 根据价值函数输出确定性策略:
$$
\pi(s)=\underset{a}{\arg \max }\left\{r(s, a)+\gamma \sum_{s} p\left(s^{\prime}|s, a\right) v\left(s^{\prime}\right)\right\}．
$$

***********************

## 二． 回合更新价值迭代

本章开始介绍无模型的机器学习算法．无模型的机器学习算法在没有环境的数学描述的情况下，只依靠经验（例如轨迹的样本) 学习出给定策略的价值函数和最优策略．在现实生活中，为环境建立精确的数学模型往往非常困难．因此，无模型的强化学习是强化学习的主要形式．

根据价值函数的更新时机，强化学习可以分为回合更新算法和时序差分更新算法这两类．回合更新算法只能用于回合制任务，它在每个回合结束后更新价值函数．本章将介绍回合更新算法，包括同策回合更新算法和异策回合更新算法．

### 2.1 同策回合更新

本节介绍同策回合更新算法．与有模型迭代更新的情况类似，我们也是先学习同策策略评估，再学习最优策略求解．

#### 2.1.1 同策回合更新策略评估

本节考虑用回合更新的方法学习给定策略的价值函数．我们知道，状态价值和动作价值分别是在给定状态和状态动作对的情况下回报的期望值．回合更新策略评估的基本思路使用 Monte Carlo 方法来估计这个期望值．具体而言，在许多轨迹样本中，如果某个状态(或状态动作对) 出现了 $c$ 次，其对应的回报值分别为 $g_{1}, g_{2}, \ldots, g_{c}$ ，那么可以估计其状态价 (或动作价值) 为 $\frac{1}{c} \sum_{i=1}^{c} g_{i}$．

无模型策略评估算法有**评估状态价值函数**和**评估动作价值函数**两种版本．在有模型情况下，状态价值和动作价值可以互相表示；但是在无模型的情况下，状态价值和动作 值并不能互相表示．我们已经知道，任意策略的价值函数满足 Bellman 期望方程．借助于动力 $p$（某个状态转移分布）的表达式，我们可以用状态价值函数表示动作价值函数；借助于策略 $\pi$ 的表达式，我们可以用动作价值函数表示状态价值函数．所以，对于无模型的策略评估， $p$ 的表达式未知，只能用动作价值表示状态价值，而不能用状态价值表示动作价值．另外，由于策略改进可以仅由动作价值函数确定，因此在学习问题中，动作价值函数往往更加重要．

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
    2. （更新动作价值）更新 $q\left(S_{t}, A_{i}\right)$ 以减小 $\left[G-q\left(S_{t}, A_{y}\right)\right]^{2}$ （如 $c\left(S_{t}, A_{t}\right) \leftarrow c\left(S_{t}, A_{1}\right)+1$ ， $\left.q\left(S_{t}, A_{i}\right) \leftarrow q\left(S_{t}, A_{t}\right)+\frac{1}{c\left(S_{i}, A_{i}\right)}\left[G-q\left(S_{t}, A_{1}\right)\right]\right)$．

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
\operatorname{Pr}_{\pi}\left[A_{t}, R_{t+1}, S_{t+1}, A_{t+1}, \ldots, S_{T-1}, A_{T-1}, R_{T}, S_{T}|S_{t}\right] \\
\quad=\pi\left(A_t|S_{t}\right) p\left(S_{t+1}, R_{t+1}|S_{t}, A_{t}\right) \pi\left(A_{t+1}|S_{t+1}\right) \cdots p\left(S_{T}, R_{T}|S_{T-1}, A_{T-1}\right) \\
\quad=\prod_{\tau=t}^{T-1} \pi\left(A_{\tau}|S_{\tau}\right) \prod_{\tau=t}^{T-1} p\left(S_{\tau+1}, R_{\tau+1}|S_{r}, A_{r}\right), \\
\operatorname{Pr}_{b}\left[A_{t}, R_{t+1}, S_{t+1}, A_{t+1}, \ldots, S_{T-1}, A_{T-1}, R_{T}, S_{T}|S_{t}\right] \\
=b\left(A_{t}|S_{t}\right) p\left(S_{t+1}, R_{t+1}|S_{t}, A_{t}\right) b\left(A_{t+1}|S_{t+1}\right) \cdots p\left(S_{T}, R_{T}|S_{T-1}, A_{T-1}\right) \\
=\prod_{\tau=t}^{T-1} b\left(A_{\tau}|S_{\tau}\right) \prod_{\tau=t}^{T-1} p\left(S_{\tau+1}, R_{\tau+1}|S_{\tau}, A_{\tau}\right)
\end{array}
$$
我们把这两个概率的比值定义为**重要性采样比率** (importance sample ratio)：
$$
\rho_{t:T-1}=\frac{\operatorname{Pr}_{\pi}\left[A_{t}, R_{t+1}, S_{t+1}, A_{t+1}, \ldots, S_{T-1}, A_{T-1}, R_{T}, S_{T}|S_{t}\right]}{\operatorname{Pr}_{b}\left[A_{t}, R_{t+1}, S_{t+1}, A_{t+1}, \ldots, S_{T-1}, A_{T-1}, R_{T}, S_{T}|S_{t}\right]}=\prod_{\tau=t}^{T-1} \frac{\pi\left(A_{\tau}|S_{\tau}\right)}{b\left(A_{\tau}|S_{\tau}\right)}
$$
这个比率只与轨迹和策略有关，而与动力无关．为了让这个比率对不同的轨迹总是有意义，我们需要使得任何满足 $\pi(a|s)>0$ 的 $s \in \mathcal{S}, a \in \mathcal{A}(s)$ ，均有 $b(a|s)>0_{\circ}$ 这样的关系可以记为$\pi \ll b$.

对于给定状态动作对 $\left(S_{t}, A_{t}\right)$ 的条件概率也有类似的分析．在给定 $\left(S_{t}, A_{t}\right)$ 的条件下，采用策略 $\pi$ 和策略 $b$ 生成这个轨迹的概率分别为：
$$
\begin{array}{l}
\operatorname{Pr}_{\pi}\left[R_{t+1}, S_{t+1}, A_{t+1}, \ldots, S_{T-1}, A_{T-1}, R_{T}, S_{T}|S_{t}, A_{t}\right] \\
\quad=p\left(S_{t+1}, R_{t+1}|S_{t}, A_{t}\right) \pi\left(A_{t+1}|S_{t+1}\right) \cdots p\left(S_{T}, R_{T}|S_{T-1}, A_{T-1}\right) \\
\quad=\prod_{\tau=t+1}^{T-1} \pi\left(A_{\tau}|S_{\tau}\right) \prod_{\tau=t}^{T-1} p\left(S_{\tau+1}, R_{\tau+1}|S_{\tau}, A_{\tau}\right) \\
\operatorname{Pr}_{b}\left[R_{t+1}, S_{t+1}, A_{t+1}, \ldots, S_{T-1}, A_{T-1}, R_{T}, S_{T}|S_{t}, A\right] \\
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

基于 2.2.1 节给出的重要性采样，算法 2-7 给出了每次访问加权重要性采样回合更新策略评估算法．这个算法在初始化环节初始化了权重和 $c(s, a)$ 与动作价值 $q(s, a)$ $(s \in \mathcal{S}, a \in \mathcal{A}(s))$ ，然后进行回合更新．回合更新需要借助行为策略 $b$ ．行为策略 $b$ 可以每个回合都单独设计，也可以为整个算法设计一个行为策略，而在所有回合都使用同一个行为策略．用行为策略生成轨迹样本后，逆序更新回报、价值函数和权重值．一开始权重值 $\rho$ 设为 1 ，以后会越来越小．如果某次权重值变为 0 (这往往是因为 $\left.\pi\left(A_{t} \mid S_{t}\right)=0\right)$ ，那么以 后的权重值就都为 0 ，再循环下去没有意义．所以这里设计了一个检查机制．事实上，这个检查机制保证了在更新 $q(s, a)$ 时权重和 $c(s, a)>0$ 是必需的．如果没有检查机制，则可能在更新 $c(s, a)$ 时，更新前和更新后的 $c(s, a)$ 值都是 0 ，进而在更新 $q(s, a)$ 时出现除零错误．加这个检查机制避免了这样的错误．

>**算法 2-7**：$\quad$ 每次访问加权重要性采样异策回合更新评估策略的动作价值
***********************

1. （初始化）初始化动作价值估计 $q(s, a) \leftarrow$ 任意值， $s \in \mathcal{S}, a \in \mathcal{A}$ ，如果需要使用权重和，则初始化权重和 $c(s, a) \leftarrow 0, s \in \mathcal{S}, a \in \mathcal{A}$
2. （回合更新）对每个回合执行以下操作  
 2.1 （行为策略）指定行为策略 $b$ ，使得 $\pi \ll b$  
 2.2 （采样）用策略 $b$ 生成轨迹: $S_{0}, A_{0}, R_{1}, S_{1}, \ldots, S_{T-1}, A_{T-1}, R_{T}, S_{T}$  
 2.3 （初始化回报和权重） $G \leftarrow 0, \rho \leftarrow 1$  
 2.4 对于 $t \leftarrow T-1, T-2, \ldots, 0$ 执行以下操作：
    1. （更新回报） $G \leftarrow \gamma G+R_{t+1}$
    2. （更新价值）更新 $q\left(S_{t}, A_{t}\right)$ 以减小 $\rho\left[G-q\left(S_{t}, A_{i}\right)\right]^{2}\left(\right.$ 如 $c\left(S_{t}, A_{1}\right) \leftarrow c\left(S_{t}, A\right)+\rho$，$\left.q\left(S_{t}, A_{t}\right) \leftarrow q\left(S_{t}, A_{t}\right)+\frac{\rho}{c\left(S_{t}, A_{t}\right)}\left[G-q\left(S_{t}, A_{t}\right)\right]\right)$
    3. （更新权重） $\rho \leftarrow \rho \frac{\pi\left(A_{t} \mid S_{t}\right)}{b\left(A_{t} \mid S_{t}\right)}$
    4. （提前终止）如果 $\rho=0$ ，则结東步骤 2.4 的循环

***********************

在算法 2-7 的基础上略作修改，可以得到首次访问的算法、普通重要性采样的算法和估计状态价值的算法，此处略过．

#### 4.2.3 异策回合更新最优策略求解

接下来介绍最优策略的求解．算法 2-8 给出了每次访问加权重要性采样异策回合最优策略求解算法．它和其他最优策略求解算法一样，都是在策略估计算法的基础上加上策略改进得来的．算法 2-8 的迭代过程中，始终让 $\pi$ 是一个确定性策略．所以，在回合更新的过程中，任选一个策略 $b$ 都满足 $\pi \ll b$ ．这个柔性策略可以每个回合都分别选取，也可以整个程序共用一个．由于采用了确定性的策略，则对于每个状态 $s \in \mathcal{S}$ 都有一个 $a \in \mathcal{A}(s)$ 使得 $\pi(a \mid s)=1$ ，而其他 $\pi(\cdot \mid s)=0$ ．算法 2-8 利用这一性质来更新权重并判断权重是否为 0 ．如果 $A_{t} \neq \pi\left(S_{t}\right)$ ，则意味着 $\pi\left(A_{t} \mid S_{t}\right)=0$ ，更新后的权重为 0 ，需要退出循环以避免除零错误；若 $A_{t}=\pi\left(S_{t}\right)$ ，则意味着 $\pi\left(A_{t} \mid S_{t}\right)=1$ ，所以权重更新语句 $\rho \leftarrow \rho \frac{\pi\left(A_{t} \mid S_{t}\right)}{b\left(A_{t} \mid S_{t}\right)}$ 就可以简化为
$\rho \leftarrow \rho \frac{1}{b\left(A_{t} \mid S_{t}\right)}$．

>**算法 4-8**：$\quad$ 每次访问加权重要性采样异策回合更新最优策略求解

1. （初始化）初始化动作价值估计 $q(s, a) \leftarrow$ 任意值， $s \in \mathcal{S}, a \in \mathcal{A}$ ，如果需要使用权重和，初始化权重和 $c(s, a) \leftarrow 0, s \in \mathcal{S}, a \in \mathcal{A} ; \pi(s) \leftarrow \arg \max _{a} q(s, a), s \in \mathcal{S}$
2. （回合更新）对每个回合执行以下操作  
 2.1 （柔性策略）指定 $b$ 为任意柔性策略  
 2.2 （采样）用策略 $b$ 生成轨迹： $S_{0}, A_{0}, R_{1}, S_{1}, A_{1}, R_{2}, \ldots, S_{T-1}, A_{T-1}, R_{T}, S_{T}$  
 2.3 （初始化回报和权重） $G \leftarrow 0, \rho \leftarrow 1$  
 2.4 对 $t \leftarrow T-1, T-2, \ldots, 0:$
    1. （更新回报） $G \leftarrow \gamma G+R_{t+1}$
    2. （更新价值）更新 $q\left(S_{t}, A_{t}\right)$ 以减小 $\rho\left[G-q\left(S_{t}, A_{t}\right)\right]^{2}\left(\right.$ 如 $c\left(S_{t}, A_{t}\right) \leftarrow c\left(S_{t}, A_{i}\right)+\rho$，$\left.q\left(S_{t}, A_{t}\right) \leftarrow q\left(S_{t}, A_{t}\right)+\frac{\rho}{c\left(S_{t}, A_{t}\right)}\left[G-q\left(S_{t}, A_{t}\right)\right]\right)$
    3. （策略更新） $\pi\left(S_{t}\right) \leftarrow \arg \max_{a} q\left(S_{t}, a\right)$
    4. （提前终止）若 $A_{t} \neq \pi\left(S_{t}\right)$ 则退出步骤 2.4
    5. （更新权重） $\rho \leftarrow \rho \frac{1}{b\left(A_{t} \mid S_{t}\right)}$

***********************
算法 2-8 也可以修改得到首次访问的算法和普通重要性采样的算法，此处略过．

## 三． 时序差分价值迭代

本章介绍另外一种学习方法一时序差分更新．时序差分更新和回合更新都是直接采用经验数据进行学习，而不需要环境模型．时序差分更新与回合更新的区别在于，时序差 分更新汲取了动态规划方法中"自益"的思想，用现有的价值估计值来更新价值估计，不需要等到回合结束也可以更新价值估计．所以，时序差分更新既可以用于回合制任务，也可以用于连续性任务．

本章将介绍时序差分更新方法，包括同策时序差分更新方法和异策时序差分更新方法, 每种方法都先介绍简单的单步更新，再介绍多步更新．最后，本章会涉及基于资格迹的学习算法．

### 3.1 同策时序差分更新

本节考虑无模型同策时序差分更新．与无模型回合更新的情况相同，在无模型的情况下动作价值比状态价值更为重要，因为动作价值能够决定策略和状态价值，但是状态价值得不到动作价值．

本节考虑无模型同策时序差分更新。与无模型回合更新的情况相同，在无模型的情 下动作价值比状态价值更为重要，因为动作价值能够决定策略和状态价值，但是状态价倍 得不到动作价值。
从给定策略 $\pi$ 的情况下动作价值的定义出发，我们可以得到下式:
$$
\begin{aligned}
q_{\pi}(s, a) &=\mathrm{E}_{\pi}\left[G_{t} \mid S_{t}=s, A_{t}=a\right] \\
&=\mathrm{E}_{\pi}\left[R_{t+1}+\gamma G_{t+1} \mid S_{t}=S, A_{i}=a\right] \\
&=\mathrm{E}_{\pi}\left[R_{t+1}+\gamma q_{\pi}\left(S_{t+1}, A_{t+1}\right) \mid S_{t}=s, A_{1}=a\right], \quad s \in \mathcal{S}, a \in \mathcal{A}(s)
\end{aligned}
$$
在上一章的回合更新学习中，我们依据 $q_{\pi}(s, a)=\mathrm{E}_{\pi}\left[G_{t} \mid S_{t}=s, A_{1}=a\right]$ ，用 Monte Carlo 方法来估计价值函数．为了得到回报样本，我们要从状态动作对 $(s, a)$ 出发一直采样到回合结束．单步时序差分更新将依据 $q_{\pi}(s, a)=\mathrm{E}_{\pi}\left[R_{t+1}+\gamma q_{\pi}\left(S_{t+1}, A_{i+1}\right) \mid S_{t}=S, A,=a\right]$ ，只需要采样一步，进而用 $U_{t}=R_{t+1}+\gamma q_{\pi}\left(S_{t+1},\cdot \right)$ ，来估计回报样本的值．为了与由奖励直接计算得到的无偏回报样本 $G_{t}$ 进行区别，本书用字母 $U_{t}$ 表示使用自益得到的有偏回报样本．

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
的增量更新来学习动作价值函数，试图减小 $\left[G_{t}-q\left(S_{t}, A_{i}\right)\right]^{2}$ ．在这个式子中， $G_{t}$ 是回报样本．在时序差分中，这个量就对应着 $U_{t}$ ．因此，只需在回合更新策略评估算法的基础上，将这个增量更新式中的回报 $G_{t}$ 替换为时序差分目标 $U_{t}$ ，就可以得到时序差分策略评估算法了．

时序差分目标既可以是单步时序差分目标，也可以是多步时序差分目标．我们先来看单步时序差分目标．

算法 3-1 给出了用单步时序差分更新评估策略的动作价值的算法．这个算法有一个 $\alpha$ ，它是一个正实数，表示学习率．在上一章的回合更新中，这个学习率往往是 $\frac{1}{c\left(S_{t}, A_t\right)}$ ，它和状态动作对有关，并且不断减小．在时序差分更新中，也可以采用这样不断减小的学习率．不过，考虑到在时序差分算法执行的过程中价值函数会越来越准确，进而基于价值函数估计得到的价值函数也会越来越准确，因此估计值的权重可以越来越大．所以，算法 5-1 采用了一个固定的学习率 $\alpha$ ．这个学习率一般在 $\alpha \in(0,1]$ ．当然，学习率也可以不是常数．在有些问题中，让学习率巧妙地变化能得到更好的效果．引入学习率 $\alpha$ 后，更新式可以表示为:
$$
q\left(S_{t}, A_{t}\right) \leftarrow q\left(S_{t}, A_{t}\right)+\alpha\left[U_{t}-q\left(S_{t}, A_{t}\right)\right] .
$$

>**算法 5-1**： $\quad$ 单步时序差分更新评估策略的动作价值
***********************
输入: 环境（无数学描述）、策略 $\pi$  
输出：动作价值函数 $q(s, a), s \in \mathcal{S}, a \in \mathcal{A}_{\circ}$  
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

>**算法 5-2**$\quad$ 单步时序差分更新评估策略的状态价值
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

接下来看如何用多步时序差分目标来评估策略价值．算法 3-3 和算法 3-4 分别给出了用多步时序差分评估动作价值和状态价值的算法．实际实现时，可以让 $S_{t}, A_{4}, R_{,}$ 和 $S_{t+n+1}, A_{t+n+1}, R_{t+n+1}$ 共享同一存储空间，这样只需要 $n+1$ 份存储空间．

>**算法 5-3** $\quad n$ 步时序差分更新评估策略的动作价值  
***********************
输入：环境（无数学描述）、策略 $\pi$  
输出：动作价值估计 $q(s, a), s \in \mathcal{S}, a \in \mathcal{A}(s)$  
参数：步数 $n$ ，优化器 (隐含学习率 $\alpha$ )，折扣因子 $\gamma$ ，控制回合数和回合内步数的参数

1. （初始化） $q(s, a) \leftarrow$ 任意值 $, s \in \mathcal{S}, a \in \mathcal{A}$ ．如果有终止状态，令 $q\left(S_{\text {终止 }}, a\right) \leftarrow 0, a \in \mathcal{A}$
2. （时序差分更新）对每个回合执行以下操作  
 2.1 （生成 n 步）用策略 $\pi$ 生成轨迹 $S_0,A_0,R_1,...,R_n,S_n$ （若遇到终止状态，则令后续奖励均为0，状态均为 $s_{\text{终止}}$  
 2.2 对于 $t=0,1,2, \ldots$ 依次执行以下操作，直到 $S_{t}=S_{\text{终止}}$ ：
    1. 若 $S_{t+n} \neq S_{\text {终止 }}$ ，则根据 $\pi\left(\cdot \mid S_{t+n}\right)$ 决定动作 $A_{t+n}$
    2. （更新时序差分目标） $U_{t:t+n}^{(q)} U \leftarrow R_{t+1}+\gamma R_{t+2}+\cdots+\gamma^{n-1} R_{t+n}+\gamma^{n} q\left(S_{t+n}, A_{t+n}\right)$
    3. （更新价值）更新 $q\left(S_{t}, A_{t}\right)$ 以减小 $\left[U-q\left(S_{t}, A_{t}\right)\right]^{2}$
    4. 若 $S_{t+n} \neq S_{\text {终止 }}$ ，则执行 $A_{t+n}$ ，得到奖励 $R_{t+n+1}$ 和下一状态 $S_{t+n+1}$ ；若 $S_{t+n}=S_{\text {终止 }}$, $R_{t+n+1} \leftarrow 0, \quad S_{t+n+1} \leftarrow S_{\text {终止 }}$

***********************

>算法 5-4 $n$ 步时序差分更新评估策略的状态价值
***********************
输入: 环境（无数学描述）、策略 $\pi$  
输出：状态价值估计 $v(s), s \in \mathcal{S}$  
参数：步数 $n$ ，优化器（隐含学习率 $\alpha$ ），折扣因子 $\gamma$ ，控制回合数和回合内步数的参数

1. (初始化） $v(s) \leftarrow$ 任意值， $s \in \mathcal{S}_{\circ}$ 如果有终止状态，令 $v\left(S_{\text {終止 }}\right) \leftarrow 0$
2. (时序差分更新) 对每个回合执行以下操作  
 2.1（生成 $n$ 步）用策略 $\pi$ 生成轨迹 $S_{0}, A_{0}, R_{1}, \ldots, R_{n}, S_{n}$ （若遇到终止状态，则令后续奖励均为 0 ，状态均为 $S_{\text {终止 }}$ ）．  
 2.2 对于 $t=0,1,2, \ldots$ 依次执行以下操作，直到 $S_{t}=S_{\text {终止 }}$ :
    1. （更新时序差分目标 $U_{t:t+n}^{(\nu)}$）$U \leftarrow R_{t+1}+\gamma R_{t+2}+\cdots+\gamma^{n-1} R_{t+n}+\gamma^{n} v\left(S_{t+n}\right)$
    2. （更新价值）更新 $v\left(S_{t}\right)$ 以减小 $\left[U-v\left(S_{t}\right)\right]^{2}$
    3. 若 $S_{t+n} \neq S_{\text {终止 }}$ ，则根据 $\pi\left(\cdot \mid S_{t+n}\right)$ 决定动作 $A_{t+n}$ 并执行，得到奖励 $R_{t+n+1}$ 和下一状态 $S_{t+n+1}$ ；若 $S_{t+n}=S_{\text {终止 }}$ ，令 $R_{t+n+1} \leftarrow 0, S_{t+n+1} \leftarrow S_{\text {终止 }}$．

***********************

#### 3.1.2 SARSA算法