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
其中 $f^{\prime}$ 和 $f^{\prime \prime}$ 是任意的以 $a$ 为自变量的函数。(证明: 设 $a^{\prime}=\arg \max_{a} f^{\prime}(a)$, 则
$$
\max _{a} f^{\prime}(a)-\max_{a} f^{\prime \prime}(a)=f^{\prime}\left(a^{\prime}\right)-\max _{a} f^{\prime \prime}(a) \leqslant f^{\prime}\left(a^{\prime}\right)-f^{\prime \prime}\left(a^{\prime}\right) \leqslant \max_{a}\left|f^{\prime}(a)-f^{\prime \prime}(a)\right|
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

- **策略评估**（policy evaluation): 对于给定的策略 $\pi$ ，估计策略的价值, 包括动作价值和状态价值
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
2. 如果新策略 $\pi^{\prime}$ 和旧策略 $\pi$ 相同，则说明旧策略巳是最优; 否则，输出改进的新策略 $\pi^{\prime}$

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

#### 4.2.3 异策回合更新最优策略求解

接下来介绍最优策略的求解．算法 2-8 给出了每次访问加权重要性采样异策回合最优策略求解算法．它和其他最优策略求解算法一样，都是在策略估计算法的基础上加上策略改进得来的．算法 2-8 的迭代过程中，始终让 $\pi$ 是一个确定性策略．所以，在回合更新的过程中，任选一个策略 $b$ 都满足 $\pi \ll b$ ．这个柔性策略可以每个回合都分别选取，也可以整个程序共用一个．由于采用了确定性的策略，则对于每个状态 $s \in \mathcal{S}$ 都有一个 $a \in \mathcal{A}(s)$ 使得 $\pi(a|s)=1$ ，而其他 $\pi(\cdot|s)=0$ ．算法 2-8 利用这一性质来更新权重并判断权重是否为 0 ．如果 $A_{t} \neq \pi\left(S_{t}\right)$ ，则意味着 $\pi\left(A_{t}|S_{t}\right)=0$ ，更新后的权重为 0 ，需要退出循环以避免除零错误；若 $A_{t}=\pi\left(S_{t}\right)$ ，则意味着 $\pi\left(A_{t}|S_{t}\right)=1$ ，所以权重更新语句 $\rho \leftarrow \rho \frac{\pi\left(A_{t}|S_{t}\right)}{b\left(A_{t}|S_{t}\right)}$ 就可以简化为
$\rho \leftarrow \rho \frac{1}{b\left(A_{t}|S_{t}\right)}$．

>**算法 4-8**：$\quad$ 每次访问加权重要性采样异策回合更新最优策略求解

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
&=\mathrm{E}_{\pi}\left[R_{t+1}+\gamma q_{\pi}\left(S_{t+1}, A_{t+1}\right)|S_{t}=s, A_{1}=a\right], \quad s \in \mathcal{S}, a \in \mathcal{A}(s)
\end{aligned}
$$
在上一章的回合更新学习中，我们依据 $q_{\pi}(s, a)=\mathrm{E}_{\pi}\left[G_{t}|S_{t}=s, A_{1}=a\right]$ ，用 Monte Carlo 方法来估计价值函数．为了得到回报样本，我们要从状态动作对 $(s, a)$ 出发一直采样到回合结束．单步时序差分更新将依据 $q_{\pi}(s, a)=\mathrm{E}_{\pi}\left[R_{t+1}+\gamma q_{\pi}\left(S_{t+1}, A_{t+1}\right)|S_{t}=S, A,=a\right]$ ，只需要采样一步，进而用 $U_{t}=R_{t+1}+\gamma q_{\pi}\left(S_{t+1},\cdot \right)$ ，来估计回报样本的值．为了与由奖励直接计算得到的无偏回报样本 $G_{t}$ 进行区别，本书用字母 $U_{t}$ 表示使用自益得到的有偏回报样本．

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
$q^{(0)}\left(S_{t}, A_{1}\right)$ 之间的差别 (例如设定损失为 $\left[U_{t}^{(0)}-q^{(0)}\left(S_{t}, A_{t}\right)\right]^{2}$ ，或采用 $q^{(0)}\left(S_{t}, A_{t}\right) \leftarrow$
$q^{(0)}\left(S_{t}, A_{t}\right)+\alpha\left[U_{t}^{(0)}-q^{(0)}\left(S_{t}, A_{t}\right)\right]$ 更新 $)$
- 使用 $U_{t}^{(1)}=R_{t+1}+\gamma q^{(0)}\left(S_{t+1}, \arg \max_{a} q^{(1)}\left(S_{t+1}, a\right)\right)$ 来更新 $\left(S_{t}, A_{t}\right)$ ，以减小 $U_{t}^{(1)}$ 和 $q^{(1)}\left(S_{t}, A_{1}\right)$ 之间的差别 (例如设定损失为 $\left[U_{t}^{(1)}-q^{(1)}\left(S_{t}, A_{2}\right)\right]^{2}$ ，或采用 $q^{(1)}\left(S_{t}, A_{t}\right) \leftarrow q^{(1)}\left(S_{t}, A_{t}\right)+$
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
其中 $\beta \in[0,1]$ 是事先给定的参数．资格迹的表达式应该这么理解：对于历史上的某个状念动作对 $\left(S_{\tau}, A_{\tau}\right)$ ，距离第 $t$ 步间隔了 $t-\tau$ 步， $U_{\tau:t}$ 在 $\lambda$ 回报 $U_{\tau}^{\lambda}$ 中的权重为 $(1-\lambda) \lambda^{t-\tau-1}$ ，并且 $U_{\tau :t}=R_{\tau+1}+\cdots+\gamma^{t-\tau-1} U_{t-1:t}$ ，所以 $U_{t-1:t}$ 是以 $(1-\lambda)(\lambda \gamma)^{t-\tau-1}$ 的比率折算到 $U_{\tau}^{\lambda}$ 中．间隔的步数每增加一步，原先的资格迹大致需要衰减为 $\gamma \lambda$ 倍．对当前最新出现的状态动作对 $\left(S_{t}, A_{1}\right)$ ，大的更新权重则要进行某种强化．强化的强度 $\beta$ 常有以下取值：

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

#### 四． 函数近似方法

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
对于状态价值函数，也有类似的分析．定义每一步的损失为 $\left[G_{t}-v\left(S_{t} ; \mathbf{w}\right)\right]^{2}$ ，整个回合的损失为 $\sum_{t=0}^{T}\left[G_{t}-v\left(S_{i} ; \mathbf{w}\right)\right]^{2}$ ．可以在自动梯度计算并更新参数的软件包中定义这个损失来更新参数 $\mathbf{w}$, 也可以用下式更新：
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

## 五． 回合更新策略梯度方法

本书前几章的算法都利用了价值函数，在求解最优策略的过程中试图估计最优价值函数，所以那些算法都称为**最优价值算法**（optimal value algorithm)．但是，要求解最优策略不一定要估计最优价值函数．本章将介绍不直接估计最优价值函数的强化学习算法，它们试图用含参函数近似最优策略，并通过迭代更新参数值．由于迭代过程与策略的梯度有关，所以这样的迭代算法又称为策略梯度算法（policy gradient algorithm）．

### 5.1 策略梯度算法的原理

基于策略的策略梯度算法有两大核心思想：

- 用含参函数近似最优策略
- 用策略梯度优化策略参数

本节介绍这两部分内容．

### 7.1.1 函数近似与动作偏好

用函数近似方法估计最优策略 $\pi_*(a|s)$ 的基本思想是用含参函数 $\pi(a|s ; \theta)$ 来近似最优策略．由于任意策略 $\pi$ 都需要满足对于任意的状态 $s \in \mathcal{S}$ ，均有 $\sum_{a} \pi(a|s)=1$ ，我们也希望 $\pi(a|s ; \theta)$ 满足对于任意的状态 $s \in \mathcal{S}$ ，均有 $\sum_{a} \pi(a|s ; \theta)=1$ ．为此引入动作偏好函数（action preference function） $h(s, a ; \theta)$ ，其 softmax 的值为 $\pi(a|s ; \theta)$ ，即
$$
\pi(a|s ; \theta)=\frac{\exp h(s, a ; \theta)}{\sum_{o^{\prime}} \exp h\left(s, a^{\prime} ; \theta\right)}, \quad s \in \mathcal{S}, a \in \mathcal{A}(s)
$$
在第 3~4 章中，从动作价值函数导出最优策略估计往往有特定的形式 (如 $\varepsilon$ 贪心策
略)．与之相比，从动作偏好导出的最优策略的估计不拘泥于特定的形式，其每个动作都可以有不同的概率值，形式更加灵活．如果采用迭代方法更新参数 $\boldsymbol{\theta}$ ，随着迭代的进行， $\pi(a|s ; \theta)$ 可以自然而然地逼近确定性策略，而不需要手动调节 $\varepsilon$ 等参数．

动作偏好函数可以具有线性组合、人工神经网络等多种形式．在确定动作偏好的形式中，只需要再确定参数 $\theta$ 的值，就可以确定整个最优状态估计．参数 $\boldsymbol{\theta}$ 的值常通过基于梯度的迭代算法更新，所以，动作偏好函数往往需要对参数 $\boldsymbol{\theta}$ 可导．

### 7.1.2 策略梯度定理

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

### 7.2 同策回合更新策略梯度算法

策略梯度定理告诉我们，沿着 $\nabla E_{\pi(\theta)}\left[G_{0}\right]=\mathrm{E}\left[\sum_{t=0}^{+\infty} \gamma^{t} G_{t} \nabla \ln \pi\left(A_{t}|S_{t} ; \theta\right)\right]$ 的方向改变策略参数 $\theta$ 的值，就有机会增加期望回报．基于这一结论，可以设计策略梯度算法．本节考虑同策更新算法

#### 7.2.1 简单的策略梯度算法

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
    2. （更新策略）更新 $\boldsymbol{\theta}$ 以减小 $-\gamma^{\prime} G \ln \pi\left(A_{1}|S_{t} ; \boldsymbol{\theta}\right)\left(\text { 如 } \boldsymbol{\theta} \leftarrow \boldsymbol{\theta}+\alpha \gamma^{t} G \nabla \ln \pi\left(A_{t}|S_{t} ; \boldsymbol{\theta}\right)\right)$

***********************

### 7.2.2 带基线的简单策略梯度算法

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

>**算法 7-2**：$\quad$ 带基线的简单策略梯度算法求解最优策略
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
接下来，我们来分析什么样的基线函数能最大程度地减小方差。考虑 $\mathrm{E}\left[\gamma^{\prime}\left(G_{t}-B\left(S_{t}\right)\right)\right.$ $\left.\nabla \ln \pi\left(A_{t}|S_{t} ; \theta\right)\right]$ 的方差为
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

### 7.3 异策回合更新策略梯度算法

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

>**算法7-3**：$\quad$ 重要性采样简单策略梯度求解最优策略
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

## 六． 执行者/评论者方法

本章介绍带自益的策略梯度算法．这类算法将策略梯度和自益结合了起来：一方面，用一个含参函数近似价值函数，然后利用这个价值函数的近似值来估计回报值；另一方面，利用估计得到的回报值估计策略梯度，进而更新策略参数．这两方面又常常被称为**评论者** (critic) 和**执行者**（actor）．所以，带自益的策略梯度算法被称为**执行者 / 评论者算法**（actorcritic algorithm)．

### 8.1 执行者 / 评论者算法

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

如果优势执行者 / 评论者算法在执行过程中不是每一步都更新参数，而是在回合结束后用整个轨迹来进行更新，就可以把算法分为经验搜集和经验使用两个部分．这样的分隔可以让这个算法同时有很多执行者在同时执行．例如，让多个执行者同时分别收集很多经验，然后都用自己的那些经验得到一批经验所带来的梯度更新值．每个执行者在一定的时机更新参数，同时更新策略参数 $\boldsymbol{\theta}$ 和价值参数 $\mathbf{w}$ ．每个执行者的更新是异步的．所以，这样的并行算法称为**异步优势执行者 / 评论者算法**（ Asynchronous Advantage Actor-Critic, $\mathrm{A} 3 \mathrm{C}$ )．异步优势执行者 / 评论者算法中的自益部分，不仅可以采用单步时序差分，也可以使用多步时序差分．另外，还可以对函数参数的访问进行控制，使得所有执行者统一更新参数．这样的并行算法称为**优势执行者 / 评论者算法**（Advantage Actor-Critic, $\mathrm{A} 2 \mathrm{C}$ )．算法 $8-3$ 给出了异步优势执行者 / 评论者算法．异步优势执行者 / 评论者算法可以有许多执行者 (或称多个线程 )，所以除了有全局的价值参数 $\mathbf{w}$ 和策略参数 $\boldsymbol{\theta}$ 外，每个线程还可能有自己维护的价值参数 $\mathbf{w}^{\prime}$ 和 $\boldsymbol{\theta}^{\prime}$ ．执行者执行时，先从全局同步参数，然后再自己学习，最后统一同步全局参数．

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
其中 $\varepsilon \in(0,1)$ 是指定的参数．采用这样的优化目标后，优化目标至多比 $a_{\pi\left(\theta_{k}\right)}\left(S_{t}, A_{1}\right)$ 大 $\varepsilon\left|a_{\pi\left(\theta_{k}\right)}\left(S_{t}, A_{t}\right)\right|$ ，所以优化问题就没有动力让代理优势 $\frac{\pi\left(A_{t}|S_{t} ; \theta\right)}{\pi\left(A_{t}|S_{t} ; \theta_{k}\right)} a_{\pi\left(\theta_{k}\right)}\left(S_{t}, A_{2}\right)$ 变得非常大，可以避免迭代后的策略与迭代前的策略差距过大．

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
$-\frac{1}{b\left(A_{1}|S_{t}\right)} \Psi_{t} \nabla \pi\left(A_{t}|S_{t} ; \theta\right)$ ．据此，可以得到异策执行者 / 评论者算法，见算法 6-10．

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

#### 6.4.2 带经驳回放的异策算法

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
这个方向才是我们真正要用的梯度方向．综合以上分析，我们可以得到带经验回放的执行者 / 评论者算法的一个简化版本．这个算法可以有一个回放因子，可以控制每次运行得到的经验可以回放多少次．算法 6-11 给出了经验回放的线程的算法．对于经验回放的线程所回放的经验是从其他线程已经执行过的线程生成并存储的，这个过程在算法 6-11 中没有展示，但是是这个算法必需的．在存储和回放的时候，不仅要存储和回放状态 $S_{t}$, 动作 $A_{1}$ 、奖励 $R_{t+1}$ 等，还需要存储和回放在状态 $S_{t}$ 产生动作 $A_{1}$ 的概率 $b\left(A_{t}|S_{t}\right)$ ．有了这个概率值，才能计算重采样系数．在价值网络的设计方面，只维护动作价值网络．在需要状态价值的估计时，由动作价值网络计算得到．

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

## 七． 连续动作空间的确定性策略

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
算法 7-1 给出了基本的同策确定性执行者 / 评论者算法．对于同策的算法，必须进行探索．连续性动作空间的确定性算法将每个状态都映射到一个确定的动作上，需要在动作空间添加扰动实现探索．具体而言，在状态 $S_{t}$ 下确定性策略 $\pi(\boldsymbol{\theta})$ 指定的动作为 $\pi\left(S_{i} ; \theta\right)$ ，则在同策算法中使用的动作可以具有 $\pi\left(S_{t} ; \boldsymbol{\theta}\right)+N_{t}$ 的形式，其中 $N_{t}$ 是扰动量．在动作空间无界的情况下（即没有限制动作有最大值和最小值)，常常假设扰动量 $N_{t}$ 满足正态分布．在动作空间有界的情况下，可以用 clip 函数进一步限制加扰动后的范围（如 $\operatorname{clip}\left(\pi\left(S_{t} ; \theta\right)+N_{t}, A_{\text {low }}, A_{\text {tigh }}\right)$ ，其中 $A_{\text {low }}$ 和 $A_{\text {high }}$ 是动作的最小取值和最大取值)，或用 sigmoid 函数将对加扰动后的动作变换到合适的区间里 $\left(\right.$ 如 $\left.A_{\text {low }}+\left(A_{\text {righ }}-A_{\text {low }}\right) \operatorname{expit}\left(\pi\left(S_{t} ; \theta\right)+N_{t}\right)\right)$．

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
