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
\quad \leqslant \max _{s \in S}\left|v^{\prime}(s)-v^{\prime \prime}(s)\right|+\max _{s \in S}\left|v^{\prime \prime}(s)-v^{\prime \prime}(s)\right| \\
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
v_{\pi}(s)=\sum_{a} \pi(a \mid s)\left[r(s, a)+\gamma \sum_{s^{\prime}} p\left(s^{\prime} \mid s, a\right) v_{\pi}\left(s^{\prime}\right)\right], \quad s \in \mathcal{S}
$$
用动作价值函数表示动作价值函数：
$$
q_{\pi}(s, a)=\sum_{s^{\prime}, r} p\left(s^{\prime}, r \mid s, a\right)\left[r+\gamma \sum_{a^{\prime}} \pi\left(a^{\prime} \mid s^{\prime}\right) q_{\pi}\left(s^{\prime}, a^{\prime}\right)\right], \quad s \in \mathcal{S}, a \in \mathcal{A}
$$

#### ※ Bellman最优方程

用最优状态价值函数表示最优状态价值函数：
$$
v_{*}(s)=\max _{a \in \mathcal{A}}\left[r(s, a)+\gamma \sum_{s^{\prime}} p\left(s^{\prime} \mid s, a\right) v_{*}\left(s^{\prime}\right)\right], \quad s \in \mathcal{S}
$$
用最优动作价值函数表示最优动作价值函数：
$$
q_{*}(s, a)=r(s, a)+\gamma \sum_{s^{\prime}} p\left(s^{\prime} \mid s, a\right) \max _{a^{\prime}} q_{*}\left(s^{\prime}, a^{\prime}\right), \quad s \in \mathcal{S}, a \in \mathcal{A}
$$

这两个方程都有用状态价值表示状态价值的形式．根据这个形式，我们可以为度量空间 $\left(\mathcal{V}, d_{\infty}\right)$ 定义 Bellman 期望算子 和 Bellman 最优算子．  

给定策略 $\pi(a \mid s)(s \in \mathcal{S}, a \in \mathcal{A}(s))$ 的 **Bellman 期望算子** $t_{\pi}: \mathcal{V} \rightarrow \mathcal{V}:$
$$
t_{n}(v)(s)=\sum_{a} \pi(a \mid s)\left[r(s, a)+\gamma \sum_{s^{\prime}} p\left(s^{\prime} \mid s, a\right) v\left(s^{\prime}\right)\right], \quad s \in \mathcal{S}
$$
**Bellman 最优算子** $t_{.}: \mathcal{V} \rightarrow \mathcal{V}$ :
$$
t_*(v)(s)=\max_{\operatorname{ces} A}\left[r(s, a)+\gamma \sum_{\text {ses }} p\left(s^{\prime} \mid s, a\right) v_{\cdot}\left(s^{\prime}\right)\right], \quad s \in \mathcal{S}
$$
下面我们就来证明，这两个算子都是压缩映射．  

首先来看 $\mathrm{Bellman}$ 期望算子 $t_{\pi}$ ．由 $t_{\pi}$ 的定义可知，对任意的 $v^{\prime}, v^{\prime \prime} \in \mathcal{V}$ ，有
$$
t_{\pi}\left(v^{\prime}\right)(s)-t_{\pi}\left(v^{\prime \prime}\right)(s)=\gamma \sum_{a} \pi(a \mid s) \sum_{s^{\prime}} p\left(s^{\prime} \mid s, a\right)\left[v^{\prime}\left(s^{\prime}\right)-v^{\prime \prime}\left(s^{\prime}\right)\right]
$$
所以
$$
\left|t_{\pi}\left(v^{\prime}\right)(s)-t_{\pi}\left(v^{\prime \prime}\right)(s)\right| \leqslant \gamma \sum_{a} \pi(a \mid s) \sum_{s^{\prime}} p\left(s^{\prime} \mid s, a\right) \max _{s^{\prime}}\left|v^{\prime}\left(s^{\prime}\right)-v^{\prime \prime}\left(s^{\prime}\right)\right|=\gamma d_{\infty}\left(v^{\prime}, v^{\prime \prime}\right)
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
&=\max _{a \in A}\left[r(s, a)+\gamma \sum_{s^{\prime} \in \mathcal{S}} p\left(s^{\prime} \mid s, a\right) v^{\prime}\left(s^{\prime}\right)\right]-\max_{a \in \mathcal{A}}\left[r(s, a)+\gamma \sum_{s^{\prime} \in S} p\left(s^{\prime} \mid s, a\right) v^{\prime \prime}\left(s^{\prime}\right)\right] \\
&\leqslant \max _{a^{\prime} \in \mathcal{A}}\left|\gamma \sum_{s^{\prime} \in \mathcal{S}} p\left(s^{\prime} \mid s, a^{\prime}\right)\left(v^{\prime}\left(s^{\prime}\right)-v^{\prime \prime}\left(s^{\prime}\right)\right)\right| \\
&\leqslant \gamma \max _{a^{\prime} \in A}\left|\sum_{s^{\prime} \in S} p\left(s^{\prime} \mid s, a^{\prime}\right)\right| \max _{s^{\prime} \in S}\left|v^{\prime}\left(s^{\prime}\right)-v^{\prime \prime}\left(s^{\prime}\right)\right| \\
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


