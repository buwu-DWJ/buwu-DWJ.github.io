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