# Rough Volatility

## Bergomi's model revisited

### Variance swap

A variance swap with maturity $\mathrm{T}$ is a contract which pays out the realized variance of the logarithmic total returns up to $\mathrm{T}$ less a strike called the variance swap rate $V_{0}^{T}$, determined in such a way that the contract has zero value today.

The annualized realized variance of a stock price process $\mathrm{S}$ for the period $[0, T]$ with business days $0=t_{0}<\ldots<t_{n}=T$ is usually defined as
$$
R V^{0, T}:=\frac{d}{n} \sum_{i=1}^{n}\left(\log \frac{S_{t_{i}}}{S_{t_{i-1}}}\right)^{2}
$$
The constant $d$ denotes the number of trading days per year and is usually fixed to 252 so that $\frac{d}{n} \approx \frac{1}{T}$. We assume the market is arbitrage-free and prices of traded instruments are represented as conditional expectations with respect to an equivalent pricing measure $\mathbb{Q}$.

A standard result gives that as $\sup_{i=1, \ldots, n}\left|t_{i}-t_{i-1}\right| \longrightarrow 0$, we have

$$
\sum_{i=1}^{n}\left(\log \frac{S_{t_{i}}}{S_{t_{i-1}}}\right)^{2} \longrightarrow\langle\log S\rangle_{T} \quad \text { in probability }
$$

when $\left(S_{t}\right)_{t \geq 0}$ is a continuous semimartingale.

Approximating the realized variance by the quadratic variation of the log returns works very well for variance swaps, but care should be taken in practise if we price short dated non-linear payoffs on realized variance. Denote by $V_{t}^{T}$, the price at time $t$ of a variance swap with maturity $T<\infty$. It is given under $\mathbb{Q}$ by
$$
V_{t}^{T}=\mathbb{E}_{t}^{\mathbb{Q}}\left[R V^{0, T}\right]=\mathbb{E}_{t}^{\mathbb{Q}}\left[\langle\log S\rangle_{T}\right]
$$

We define the ***forward variance curve*** $\left(\xi^{T}\right)_{T \geq 0}$ as
$$
\xi_{t}^{T}:=\partial_{T} V_{t}^{T}, \quad T \geq t \geq 0
$$
Note that, if we assume that the S\&PX index follows a diffusion process, $d S_{t}=$ $\mu_{t} S_{t} d t+\sigma_{t} S_{t} d W_{t}$ with a general stochastic volatility process, $\sigma$, the forward variance is given by
$$
\xi_{t}^{T}=\mathbb{E}_{t}^{\mathbb{Q}}\left(\sigma_{T}^{2}\right)
$$
It can be seen as the forward instantaneous variance for date $\mathrm{T}$, observed at $t$. In particular
$$
\xi_{t}^{t}=\sigma_{t}^{2}, \quad \forall t \geq 0
$$

The current price of a variance swap, $V_{t}^{T}$, is given in terms of the forward variances as
$$
V_{t}^{T}=\langle\log S\rangle_{t}+\int_{t}^{T} \xi_{t}^{u} d u
$$
The models used in practice are based on diffusion dynamics where forward variance curves are given as a functional of a finite-dimensional Markov-process:
$$
\xi_{t}^{T}=G\left(T ; t, Z_{t}\right)
$$
where the function $\mathrm{G}$ and the m-dimensional Markov-process $\mathrm{Z}$ satisfy some consistency
condition, which essentially ensures that for every fixed maturity $T>0$, the forward variance $\left(\xi_{t}^{T}\right)_{t \leq T}$ is a martingale.

## ※ Pricing under rough volatility ※

### ATM volatility skew

$$
\psi(\tau):=\left|\frac{\partial}{\partial k} \sigma_{\mathrm{BS}}(k, \tau)\right|_{k=0}
$$
其中 $\tau=T-t$ 是离到期日的时间， $k=\log K / S_{t}$ 是log-strike．在传统随机波动率模型中， $\psi(\tau)$ 对短期时间是常数，对长时间与 $\tau$ 成反比．经验上观测到 $\psi(\tau)$ 对某些 $0<\alpha<1 / 2$ 与 $1 / \tau^{\alpha}$ 成比例．

### forward variance curve

$v_{u}=\sigma_{u}^{2}$ 表示 $u>t$ 时刻瞬时方差．则 forward variance curve 为：
$$
\xi_{t}(u)=\mathbb{E}\left[v_{u} \mid \mathcal{F}_{t}\right], \quad u \geq t
$$

### Wick exponential

对零均值的 Gaussian R.V. ，其 Wick exponential 为
$$
\mathcal{E}(\Psi)=\exp \left(\Psi-\frac{1}{2} \mathbb{E}\left[|\Psi|^{2}\right]\right)
$$

这里只作记号使用，不涉及其运算．

### 模型推导

Gatheral et al. (2014) 发现已实现方差（realized variance） 与如下模型一致
$$
\log \sigma_{t+\Delta}-\log \sigma_{t}=v\left(W_{t+\Delta}^{H}-W_{t}^{H}\right) \tag{1}
$$

其中 $W^{H}$ 是 fBm．This relationship was found to hold for all 21 equity indices in the Oxford-Man database, Bund futures, Crude Oil futures, and Gold futures. Perhaps this feature of the time series of volatility is universal?

考虑 fBm $W^{H}$ 的 Mandelbrot-Van Ness 表示
$$
W_{t}^{H}=C_{H}\left\{\int_{-\infty}^{t} \frac{\mathrm{d} W_{s}^{\mathbb{P}}}{(t-s)^{\gamma}}-\int_{-\infty}^{0} \frac{\mathrm{d} W_{s}^{\mathbb{P}}}{(-s)^{\gamma}}\right\}\tag{2}
$$

其中 $\gamma=1 / 2-H$ ， $C_{H}=\sqrt{\frac{2 H \Gamma(3 / 2-H)}{\Gamma(H+1 / 2) \Gamma(2-2 H)}}$ 这样选取是为了保证
$$
\mathbb{E}\left[W_{t}^{H} W_{s}^{H}\right]=\frac{1}{2}\left\{t^{2 H}+s^{2 H}-|t-s|^{2 H}\right\}
$$

将 （2）带入（1）并由 $v_{t}=\sigma_{t}^{2}$ ，可以得到 $v_{u}$ 基于 physical measure $\mathbb{P}$ 的变化:
$$
\begin{aligned}
\log v_{u}-\log v_{t} \\
=& 2 v C_{H}\left\{\int_{-\infty}^{u} \frac{\mathrm{d} W_{s}^{\mathbb{P}}}{(u-s)^{\gamma}}-\int_{-\infty}^{t} \frac{\mathrm{d} W_{s}^{\mathbb{P}}}{(t-s)^{\gamma}}\right\} \\
=& 2 \nu C_{H}\left\{\int_{t}^{u} \frac{1}{(u-s)^{\gamma}} \mathrm{d} W_{s}^{\mathbb{P}}\right.\\
&\left.+\int_{-\infty}^{t}\left[\frac{1}{(u-s)^{\gamma}}-\frac{1}{(t-s)^{\gamma}}\right] \mathrm{d} W_{s}^{\mathbb{P}}\right\} \\
=&: 2 v C_{H}\left\{M_{t}(u)+Z_{t}(u)\right\}
\end{aligned}
$$

注意到 $Z_{t}(u)$ 是 $\mathcal{F}_{t}$-可测，而 $M_{t}(u)$ 与 $\mathcal{F}_{t}$ 独立且是 Gaussian with mean zero and variance $(u-t)^{2 H} /(2 H)$． 用如下记号：
$$
\tilde{W}_{t}^{\mathbb{P}}(u):=\sqrt{2 H} \int_{t}^{u} \frac{\mathrm{d} W_{s}^{\mathbb{P}}}{(u-s)^{\gamma}}\tag{3}
$$
和 $M_{t}(u)$ 有相同分布，仅仅方差变为 $(u-t)^{2 H}$．记 $\eta:=2 v C_{H} / \sqrt{2 H}$ 则有 $2 v C_{H} M_{t}(u)=$ $\eta \tilde{W}_{t}^{\mathbb{P}}(u)$ 且
$$
\mathbb{E}^{\mathbb{P}}\left[v_{u} \mid \mathcal{F}_{t}\right]=v_{t} \exp \left\{2 v C_{H} Z_{t}(u)+\frac{1}{2} \eta^{2} \mathbb{E}\left|\tilde{W}_{t}^{\mathbb{P}}(u)\right|^{2}\right\}
$$
结合 Wick expenential
$$
\begin{aligned}
v_{u} &=v_{t} \exp \left\{\eta \tilde{W}_{t}^{\mathbb{P}}(u)+2 \nu C_{H} Z_{t}(u)\right\} \\
&=\mathbb{E}^{\mathbb{P}}\left[v_{u} \mid \mathcal{F}_{t}\right] \mathcal{E}\left(\eta \tilde{W}_{t}^{\mathbb{P}}(u)\right)\tag{4}
\end{aligned}
$$
这里，由 1 式可知 $Z_{t}(u)$ 依赖 $W^{\mathbb{P}}$ 的整个历史，所以 $v_{u}$ 是 non-Markovian．而 2 式表示 the conditional distribution of $v_{u}$ depends on $\mathcal{F}_{t}$ only through the instantaneous variance forecasts $\mathbb{E}^{\mathbb{P}}\left[v_{u} \mid \mathcal{F}_{t}\right], u>t.$

总结，**得到如下模型**基于实际概率测度 $\mathbb{P}$ :
$$
\begin{aligned}
\frac{\mathrm{d} S_{u}}{S_{u}} &=\mu_{u} \mathrm{~d} u+\sqrt{v_{u}} \mathrm{~d} Z_{u}^{\mathbb{P}} \\
v_{u} &=v_{t} \exp \left\{\eta \tilde{W}_{t}^{\mathbb{P}}(u)+2 v C_{H} Z_{t}(u)\right\}\tag{5}
\end{aligned}
$$
其中，两个布朗运动 $Z^{\mathbb{P}}$ 和 $W^{\mathbb{P}}$ 相关系数为 $\rho$．

### Pricing under Q

期权在 t 时刻的定价基于等价鞅测度 $\mathbb{Q} \sim \mathbb{P}$ on $[t, T]$ s.t. 资产价格过程 $S_{t}$ 在 $\mathbb{Q}$ 下是一个鞅．

在固定的时间域 $[t, T]$ 中，通过 Girsanov 变换，
$$
\mathrm{d} Z_{u}^{\mathbb{Q}}=\mathrm{d} Z_{u}^{\mathbb{P}}+\frac{\mu_{u}}{\sqrt{v_{u}}} \mathrm{~d} u, \quad t \leq u \leq T
$$
使得
$$
\frac{\mathrm{d} S_{u}}{S_{u}}=\sqrt{v_{u}} \mathrm{~d} Z_{u}^{\mathbb{Q}}, \quad t \leq u \leq T
$$

另一方面 $\tilde{W}_{t}^{\mathbb{P}}$ 由 $W^{\mathbb{P}}$ 而来，而 $W^{\mathbb{P}}$ 是一个布朗运动与 $Z_{u}^{\mathbb{P}}$ 以如下关系相关，
$$
W^{\mathbb{P}}=\rho Z^{\mathbb{P}}+\bar{\rho} \bar{Z}^{\mathbb{P}}, \quad \rho^{2}+\bar{\rho}^{2}=1
$$
其中 $\left(Z^{\mathbb{P}}, \bar{Z}^{\mathbb{P}}\right)$ 是一对独立的标准布朗运动．对第二项的一个标准的测度变换为
$$
d \bar{Z}_{u}^{\mathbb{Q}}=d \bar{Z}_{u}^{\mathbb{P}}+\gamma_{u} \mathrm{~d} u
$$
其中 $\gamma=\gamma(u, \omega)$，for $u \in[t, T]$，是一个合适的适应过程，称为波动率风险的市场价格．所以有
$$
\mathrm{d} W_{u}^{\mathbb{Q}}=\mathrm{d} W_{u}^{\mathbb{P}}+\left[\rho \mu_{u} / \sqrt{v_{u}}+\bar{\rho} \gamma_{u}\right] \mathrm{d} u, \quad t \leq u \leq T，
$$
将其重写为
$$
\mathrm{d} W_{s}^{\mathbb{P}}=\mathrm{d} W_{s}^{\mathbb{Q}}+\lambda_{s} \mathrm{~d} s
$$
由 4 ，在 $\mathbb{P}:$ 下，
$$
v_{u}=\mathbb{E}^{\mathbb{P}}\left[v_{u} \mid \mathcal{F}_{t}\right] \mathcal{E}\left(\eta \tilde{W}_{t}^{\mathbb{P}}(u)\right)
$$
特别的， $\mathbb{E}^{\mathbb{P}}\left[v_{u} \mid \mathcal{F}_{t}\right]$ 适应于由 $W^{\mathbb{P}}$ 生成的域流（和由 $W^{\mathbb{Q}}$ 生成的域流一致）．把上式重写为
$$
\begin{aligned}
v_{u}=\mathbb{E}^{\mathbb{P}} &\left[v_{u} \mid \mathcal{F}_{t}\right] \exp \{\eta \sqrt{2 H}\\
&\left.\times \int_{t}^{u} \frac{1}{(u-s)^{\gamma}} \mathrm{d} W_{s}^{\mathbb{P}}-\frac{\eta^{2}}{2}(u-t)^{2 H}\right\} \\
=\mathbb{E}^{\mathbb{P}} &\left[v_{u} \mid \mathcal{F}_{t}\right] \mathcal{E}\left(\eta \tilde{W}_{t}^{\mathbb{Q}}(u)\right) \\
& \times \exp \left\{\eta \sqrt{2 H} \int_{t}^{u} \frac{\lambda_{s}}{(u-s)^{\gamma}} \mathrm{d} s\right\}
\end{aligned}\tag{6}
$$
指数中的最后一项明显改变了 $v_{u}$ 的边缘分布．虽然 $v_{u}$ 在 $\mathbb{P}$ 下的条件分布是对数正态的，它在 $\mathbb{Q}$ 下不是对数正态．

### rBergomi model

考虑最简单的测度变换，
$$
\mathrm{d} W_{s}^{\mathbb{P}}=\mathrm{d} W_{s}^{\mathbb{Q}}+\lambda(s) \mathrm{d} s
$$
assuming for simplicity, resp. as a first approximation,  $\lambda(s)$ 是关于 $s$ 的确定性函数．则由 6 我们有
$$
\begin{aligned}
v_{u}&= \mathbb{E}^{\mathbb{P}}\left[v_{u} \mid \mathcal{F}_{t}\right] \mathcal{E}\left(\eta \tilde{W}_{t}^{\mathbb{Q}}(u)\right) \\
& \quad \times \exp \left\{\eta \sqrt{2 H} \int_{t}^{u} \frac{1}{(u-s)^{\gamma}} \lambda(s) \mathrm{d} s\right\} \\
&=\xi_{t}(u) \mathcal{E}\left(\eta \tilde{W}_{t}^{\mathbb{Q}}(u)\right)
\end{aligned}\tag{7}
$$
其中 $\xi_{t}(u)=\mathbb{E}^{\mathbb{Q}}\left[v_{u} \mid \mathcal{F}_{t}\right]$ ．进一步有 forward variance curve
$$
\xi_{t}(u)=\mathbb{E}^{\mathbb{P}}\left[v_{u} \mid \mathcal{F}_{t}\right] \exp \left\{\eta \sqrt{2 H} \int_{t}^{u} \frac{1}{(u-s)^{\gamma}} \lambda(s) \mathrm{d} s\right\}
$$

是如下两项的乘积： $\mathbb{E}^{\mathbb{P}}\left[v_{u} \mid \mathcal{F}_{t}\right]$ 依赖于驱动布朗运动的历史；另一项依赖于风险价格 $\lambda(s)$.

模型 7 是 non-Markovian 因为 $v_{t}: \mathbb{E}^{\mathbb{Q}}\left[v_{u} \mid \mathcal{F}_{t}\right] \neq$ $\mathbb{E}^{\mathbb{Q}}\left[v_{u} \mid v_{t}\right]$．

## ※on deep calibration of rough sv model※

### 一．介绍

[Bayer C, Horvath B, Muguruza A, et al. On deep calibration of (rough) stochastic volatility models[J]. arXiv preprint arXiv:1908.08806, 2019.](https://arxiv.org/abs/1908.08806)

![1](pics/1.PNG)

从隐含波动率按 moneyness 和 maturity 的变化可以观察到存在着著名的 smiles 和 at-the-money(ATM) skews 现象，与 BS 公式相悖．特别的，Bayer, Friz, and Gatheral 经验性地表明 ATM skew 符合如下形式：

$$
\left|\frac{\partial}{\partial m} \sigma_{\mathrm{iv}}(m, T)\right| \sim T^{-0.4}, \quad T \rightarrow 0\tag{1}
$$
其中 $m$ 为 $\log$ moneynessand ，$T$ 为 time to maturity .

根据 Gatheral ，扩散的随机波动率模型不能复现当 time to maturity 趋于零时 volatility skew 的幂指数爆炸现象，反而表现为常数现象．

RSV 可定义为一族连续路径的随机波动率模型，其瞬时波动率由一个 Holder 正则性比布兰运动小的随机过程驱动，通常刻画为 Hurst 系数 H<1/2 的分形布朗运动．

这种范式转变的证据现在是 overwhelming ，一方面在物理测度下，时间序列分析表明对数已实现波动率的 Holder 正则为 0.1 阶；另一方面，在定价测度下经验性观察也表明在零附近由模型能够生成 power-law behaviour 的 volatility skew．

模型的一大难点来自于分形布朗运动的**非马尔科夫性**．

本文介绍两种方法

- **one-step approach** : 直接学习从隐含波动率曲面到模型参数的映射，
- **two-step approach** : 第一步学习从模型参数到期权价格的映射，然后根据实际市场价格校准模型．又分为 **point-wise approach** 和 **grid-wise approach**，前者将行权价和到期日作为输入，后者事先设定好这两项．

### 二．模型校准概述（未使用神经网络）

校准（calibration）意思是调整模型参数以使得模型曲面符合由欧式期权通过BS公式计算出的经验隐含波动率曲面．

假设模型有一个参数集 $\Theta$ 决定， i.e.，由 $\theta \in \Theta$．进一步，我们假设期权由参数集 $\zeta \in Z$ 决定．E.g.，对看涨看跌期权我们有 $\zeta=(T, k)$，分别为到期日和 log-moneyness．有些参数由市场观测得到，如现价、利率等，不在校准过程中．定价映射为
$$
(\theta, \zeta) \mapsto P(\theta, \zeta)
$$
带参数 $\theta$ 的模型中带参数 $\zeta$ 的期权的价格．我们通过 $\mathcal{P}(\zeta)$ 给定了有限子集 $\zeta \in Z^{\prime} \subset Z$ 以及所有可能的期权参数对应的期权价格．**校准**是决定模型参数以使模型价格 $(P(\theta, \zeta))_{\zeta \in Z^{\prime}}$ 和市场价格 $(\mathcal{P}(\zeta))_{\zeta \in Z^{\prime}}$ 在给定距离度量下最小，i.e.:
$$
\widehat{\theta}=\underset{\theta \in \Theta}{\operatorname{argmin}} \delta\left((P(\theta, \zeta))_{\zeta \in Z^{\prime}},(\mathcal{P}(\zeta))_{\zeta \in Z^{\prime}}\right).
$$

事实上，最常用的 $\delta$ 是加权最小二乘：
$$
\widehat{\theta}=\underset{\theta \in \Theta}{\operatorname{argmin}} \sum_{\zeta \in Z^{\prime}} w_{\zeta}(P(\theta, \zeta)-\mathcal{P}(\zeta))^{2}
$$
这里的权重 $w_{\zeta}$ 反映了 $\zeta$ 对应期权的重要性以及 $\mathcal{P}(\zeta)$ 的可靠性．例如可以选择 bid-ask spread 的倒数．

只要模型参数比 $\left|Z^{\prime}\right|$ 少，此时就是超定(overdetermined)的非线性最小二乘问题，通常采用数值迭代的方法解决，如 Levenberg-Marquardt（LM）算法．

**rBergomi** ：表示为 $\mathcal{M}^{\text {rBergomi }}\left(\Theta^{\text {rBergomi }}\right)$ ，参数 $\theta=\left(\xi_{0}, \eta, \rho, H\right) \in \Theta^{\text {rBergomi }}$ ，例如可以设为
$$
\left.\Theta^{\mathrm{rBergomi}}=\mathbb{R}_{>0} \times \mathbb{R}_{>0} \times[-1,1] \times\right] 0,1 / 2[
$$
模型基于如下系统
$$
\begin{aligned}
d X_{t} &=-\frac{1}{2} V_{t} d t+\sqrt{V_{t}} d W_{t}, \quad \text { for } t>0, \quad X_{0}=0 \\
V_{t} &=\xi_{0}(t) \mathcal{E}\left(\sqrt{2 H} \eta \int_{0}^{t}(t-s)^{H-1 / 2} d Z_{s}\right), \quad \text { for } t>0, \quad V_{0}=v_{0}>0
\end{aligned}
$$
其中 $H$ 是 Hurst 系数，$\eta>0$，$\mathcal{E}(\cdot)$ 是 Wick exponential，$\xi_{0}(\cdot)>0$ 表示初始forward variance curve， $W$ 和 $Z$ 是以 $\rho \in[-1,1]$ 相关的布朗运动．

### 三．深度校准

#### 3.1 one-step approach

[Hernandez A. Model calibration with neural networks[J]. Available at SSRN 2812140, 2016.](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=2812140)

直接学习校准过程，即将模型参数视作市场价格（隐含波动率）的函数，i.e.
$$
\Pi^{-1}:(\mathcal{P}(\zeta))_{\zeta \in Z^{\prime}} \mapsto \widehat{\theta}
$$
更具体地，训练神经网络基于标签数据 $\left(x_{i}, y_{i}\right)，i=1， \ldots, N$，
$$
x_{i}=(\mathcal{P}(\zeta))_{\zeta \in Z_{i}^{\prime}}
$$
及其对应标签
$$
y_{i}=\widehat{\theta}_{i}
$$

#### 3.2 two-step approach

首先学习定价映射，将模型参数映射为市场价格（或隐含波动率），然后使用标准校准方法进行校准．我们用 $\widetilde{P}(\theta, \zeta) \approx P(\theta, \zeta)$ 表示 $\widetilde{P}$ 是 $P$ 的通过神经网络得到的近似．然后第二步我们进行校准
$$
\widehat{\theta}=\underset{\theta \in \Theta}{\operatorname{argmin}} \delta\left((\widetilde{P}(\theta, \zeta))_{\zeta \in Z^{\prime}},(\mathcal{P}(\zeta))_{\zeta \in Z^{\prime}}\right)\tag{2}
$$

两步方法相较而言最大的好处如下：

- 神经网络只负责期权定价，所以能用人工数据来训练．
- 自然地将误差分为定价误差和模型误差．神经网络表现和模型对市场适应性做出的调整相互独立．

##### 3.2.1 two-step approach: 逐点训练（pointwise）和基于网格(grid-based)训练

In this section, we examine its advantages and present an analysis of the objective function with the goal to enhance learning performance. Within this framework, the pointwise approach has the ability to asses the quality of $\widetilde{P}$ using Monte Carlo or PDE methods, and indeed it is superior training in terms of robustness.

##### Pointwise learning

**step 1**:学习映射 $\widetilde{P}(\theta, T, k)=\widetilde{\sigma}^{\mathcal{M}(\theta)}(T, k)$ 即上述（2）式令 $\zeta=(T, k)$．在标准化期权（vanilla option，$(\zeta=(T, k))$）情况下，我们可以直接学习隐含波动率映射 $\operatorname{map} \tilde{\sigma}^{\mathcal{M}(\theta)}(T, k)$，而不是期权定价的映射 $\widetilde{P}(\theta, T, k)$．用 $F(w ; \theta, \zeta)$ 表示神经网络，最优化问题如下：
$$
\widehat{\omega}:=\underset{w \in \mathbb{R}^{n}}{\operatorname{argmin}} \sum_{i=1}^{N_{\text {Train }}} \eta_{i}\left(\widetilde{F}\left(w ; \theta_{i}, T_{i}, k_{i}\right)-\widetilde{\sigma}^{\mathcal{M}}\left(\theta_{i}, T_{i}, k_{i}\right)\right)^{2}
$$

**Step 2**: 解决经典的模型校准步骤：
$$
\widehat{\theta}:=\underset{\theta \in \Theta}{\operatorname{argmin}} \sum_{j=1}^{m} \beta_{j}\left(\widetilde{\sigma}^{\mathcal{M}(\theta)}\left(\theta, T_{j}, k_{j}\right)-\sigma_{\mathrm{BS}}^{\mathrm{MKT}}\left(k_{j}, T_{j}\right)\right)^{2}
$$

这里 $\widetilde{P}(\theta, T, k)$ 或者 $\widetilde{\sigma}^{\mathcal{M}(\theta)}(T, k)$ 被替换成 step1 中的近似网络 $\widetilde{F}(\widehat{\omega} ; \theta, T, k)$ ．

第一步中，关键在于训练数据和网络结构的选择．训练数据在于选择
$\theta$ 和 $\zeta$ $(=(T, k)$ 的‘先验’的、有实际意义的分布．

##### Implicit \& grid-based learning

用 $\Delta:=\left\{k_{i}, T_{j}\right\}_{i=1, j=1}^{n,} m_{i=1}$ 记关于到期日和行权价的网格．则

**step 1**：学习映射 $\widetilde{F}(\theta)=\left\{\sigma_{B S}^{\mathcal{M}(\theta)}\left(T_{i}, k_{j}\right)\right\}_{i=1, j=1}^{n,m}$，输入是 $\theta \in \Theta$，输出是 $\left\{\sigma_{\mathrm{BS}}^{\mathcal{M}(\theta)}\left(T_{i}, k_{j}\right)\right\}_{i=1, j=1}^{n, m}$ 这样的  $n \times m$ 网格．$\widetilde{F}$ 取值在 $\mathbb{R}^{L}$ 中，其中 $L=$ strikes $\times$ maturities $=n m .$最优化问题变为如下：
$$
\widehat{\omega}:=\underset{w \in \mathbb{R}^{n}}{\operatorname{argmin}} \sum_{i=1}^{N_{\text {Train }}^{\text {reduced }}} \sum_{j=1}^{L} \eta_{j}\left(\widetilde{F}\left(\theta_{i}\right)_{j}-\sigma^{\mathcal{M}}\left(\theta_{i}, T_{j}, k_{j}\right)\right)^{2}
$$
其中 $N_{\text {Train }}=N_{\text {Train }}^{\text {reduced }} \times L$．
**Step 2**:
$$
\widehat{\theta}:=\underset{\theta \in \Theta}{\operatorname{argmin}} \sum_{i=1}^{L} \beta_{j}\left(\widetilde{F}(\theta)_{i}-\sigma_{\mathrm{BS}}^{\mathrm{MKT}}\left(T_{i}, k_{i}\right)\right)^{2}
$$

这里期权的参数 $\zeta=(T, k)$ 是固定了的，不再是学习的一部分．

##### 3.2.2 pointwise versus grid-based

- 最大的不同在于 grid-based 在遇到不在网格上的 T,K 时需要手动插值
- grid-based 方法自然地有 reduction of variance ，
- pointwise 中对使样本符合实际金融数据的操作更简单，改变采样的分布．而 grid-wise 则是通过改变权重或者网格密度．
- grid-based 方法可以看做是一种降低维度的操作，将输入的维度转移到了输出的维度．

### 四．Pratical implementation

#### 4.1 网络结构与训练

1. 隐藏层为 3 层的全连接前馈神经网络，每层 30 个结点
2. 输入维度记 $=n$
3. 输出维度为 $=11\times 8$
4. 总共有 $(n+1) \times 30+3 \times(1+30) \times 30+(30+1) \times 88=30 n+5548$ 个参数．
5. 激活函数选择 Elu， $\sigma_{\text {Elu }}=\alpha\left(e^{x}-1\right)$，梯度下降选择 Adam．

#### 4.2 校准

使用第二节中讲的 LM 等算法．

### 五．数值实验

#### 5.1定价近似网络的速度和精确度

## ※Deep learning volatility: a deep neural network perspective on pricing and calibration in (rough）volatility models※

[Horvath B, Muguruza A, Tomas M. Deep learning volatility: a deep neural network perspective on pricing and calibration in (rough) volatility models[J]. Quantitative Finance, 2021, 21(1): 11-27.
](https://arxiv.org/abs/1901.09647)

[Github 代码](https://github.com/amuguruza/NN-StochVol-Calibrations)

## fBm的 Monte-Carlo 模拟

### 1.理论基础

[Horvath B, Jacquier A J, Muguruza A. Functional central limit theorems for rough volatility[J]. Available at SSRN 3078743, 2017.](https://arxiv.org/abs/1711.03078)

**Notations**: 在单位区间 $\mathbb{I}:=[0,1]上, \mathcal{C}(\mathbb{I})$ 表示连续函数空间， $\mathcal{C}^{\alpha}(\mathbb{I})$ 表示 $\alpha$-Hölder 连续函数空间， $\alpha \in(0,1)$ ． $\mathcal{C}^{1}(\mathbb{I})$ 和 $\mathcal{C}_{b}^{1}(\mathbb{I})$ 是 $\mathbb{I}$ 上连续可微和有界连续可微函数空间．

#### 1.1. Hölder spaces and fractional operators

For $\beta \in(0,1]$, the **$\beta$-Hölder space** $\mathcal{C}^{\beta}(\mathbb{I})$, with the norm
$$
\|f\|_{\beta}:=|f|_{\beta}+\|f\|_{\infty}=\sup_{t, s \in \mathbb{I} \atop t \neq s} \frac{|f(t)-f(s)|}{|t-s|^{\beta}}+\max _{t \in \mathbb{I}}|f(t)|
$$
is a non-separable Banach space. Following the spirit of Riemann-Liouville fractional operators recalled in Appendix $\mathrm{A}$, we introduce the class of Generalised Fractional Operators (GFO). For any $\lambda \in(0,1)$ we introduce the intervals
$$
\Re^{\lambda}:=\{\alpha \in(-1,1) \text { such that } \alpha+\lambda \in(0,1)\}
$$
$\Re_{+}^{\lambda}:=\Re^{\lambda} \cap(0,1), \Re_{-}^{\lambda}:=\Re^{\lambda} \cap(-1,0)$, and the space $\mathcal{L}^{\alpha}:=\left\{u \mapsto u^{\alpha} L(u): L \in \mathcal{C}_{b}^{1}(\mathbb{I})\right\}$, for any $\alpha \in \Re^{\lambda}$

**Definition 1.1.** For any $\lambda \in(0,1)$ and $\alpha \in \mathfrak{R}^{\lambda}$, the GFO associated to $g \in \mathcal{L}^{\alpha}$ is defined on $\mathcal{C}^{\lambda}(\mathbb{I})$ as
$$
\left(\mathcal{G}^{\alpha} f\right)(t):= \begin{cases}\int_{0}^{t} f(s) \frac{\mathrm{d}}{\mathrm{d} t} g(t-s) \mathrm{d} s, & \text { if } \alpha \in[0,1-\lambda) \\ \frac{\mathrm{d}}{\mathrm{d} t} \int_{0}^{t} f(s) g(t-s) \mathrm{d} s, & \text { if } \alpha \in(-\lambda, 0)\end{cases}\tag{1.1}
$$
We shall further use the notation $G(t):=\int_{0}^{t} g(u) \mathrm{d} u$, for any $t \in \mathbb{I}$. Of particular interest in mathematical finance are the following kernels and operators:
$$
\begin{array}{lll}
Riemann-Liouville: \quad &g(u)=u^{\alpha}, \quad &for \alpha \in(-1,1)\\
Gamma fractional:  &g(u)=u^{\alpha} \mathrm{e}^{\beta u}, &for \alpha \in(-1,1), \beta<0 \\  Power-law:  &g(u)=u^{\alpha}(1+u)^{\beta-\alpha},  &for \alpha \in(-1,1), \beta<-1
\end{array}\tag{1.2}
$$

**Proposition 1.2.** For any $\lambda \in(0,1)$ and $\alpha \in \mathfrak{R}^{\lambda}$, the operator $\mathcal{G}^{\alpha}: \mathcal{C}^{\lambda}(\mathbb{I}) \rightarrow \mathcal{C}^{\lambda+\alpha}(\mathbb{I})$ is continuous.

We develop here an approximation scheme for the following system, generalising the concept of rough volatility in the context of mathematical finance, where the process $X$ represents the dynamics of the logarithm of a stock price process:
$$
\begin{aligned}
\mathrm{d} X_{t} &=-\frac{1}{2} V_{t} \mathrm{~d} t+\sqrt{V_{t}} \mathrm{~d} B_{t}, & X_{0}=0 \\
V_{t} &=\Phi\left(\mathcal{G}^{\alpha} Y\right)(t), & V_{0}>0
\end{aligned}\tag{1.3}
$$
with $\alpha \in\left(-\frac{1}{2}, \frac{1}{2}\right)$, and $Y$ the (strong) solution to the stochastic differential equation $(1.4)$
$$
\mathrm{d} Y_{t}=b\left(Y_{t}\right) \mathrm{d} t+a\left(Y_{t}\right) \mathrm{d} W_{t}, \quad Y_{0} \in \mathcal{D}_{Y}\tag{1.4}
$$

where $\mathcal{D}_{Y}$ denotes the state space of the process $Y$, usually $\mathbb{R}$ or $\mathbb{R}_{+} .$The two Brownian motions $B$ and $W$, defined on a common filtered probability space $\left(\Omega, \mathcal{F},\left(\mathcal{F}_{t}\right)_{t \in \mathbb{I}}, \mathbb{P}\right)$, are correlated by the parameter $\rho \in[-1,1]$, and the functional $\Phi$ is assumed to be smooth on $\mathcal{C}^{1}(\mathbb{I}) .$ This is enough to ensure that the first stochastic differential equation is well defined. It remains to formulate the precise definition for $\mathcal{G}^{\alpha} Y$ (Proposition 1.4) to fully specify the system (1.3) and clarify the existence of solutions. Existence and (strong) uniquess of a solution to the second $\mathrm{SDE}$ in (1.4) is guaranteed by the following standard assumption :

**Assumption 1.3.** There exist $C_{b}, C_{a}>0$ such that, for all $x, y \in \mathcal{D}_{Y}$
$$
|b(x)-b(y)| \leq C_{b}|x-y| \quad \text { and } \quad|a(x)-a(y)| \leq C_{a} \sqrt{|x-y|}
$$

**Proposition 1.4.** For any $\alpha \in \mathfrak{R}^{1 / 2}$ ，the equality $\left(\mathcal{G}^{\alpha} W\right)(t)=\int_{0}^{t} g(t-s) \mathrm{d} W_{s}$ holds almost surely for $t \in \mathbb{I}$．

**Example 1.5.** This example is the rough Bergomi model introduced by Bayer, Friz and Gatheral, where
$$
V_{t}=\xi_{0}(t) \mathcal{E}\left(2 \nu C_{H} \int_{0}^{t}(t-s)^{\alpha} \mathrm{d} W_{s}\right)
$$
with $V_{0}, \nu, \xi_{0}(\cdot)>0, \alpha \in \Re^{1 / 2}$ and $\mathcal{E}(\cdot)$ is the Wick stochastic exponential. This corresponds exactly to $(1.3)$ with $g(u) \equiv u^{\alpha}, Y=W$ and
$$
\Phi(\varphi)(t):=\xi_{0}(t) \exp \left(2 \nu C_{H} \varphi(t)\right) \exp \left\{-2 \nu^{2} C_{H}^{2} \int_{0}^{t}(t-s)^{2 \alpha} \mathrm{d} s\right\}
$$

#### 1.2 The approximation scheme

We now move on to the core of the project, namely an approximation scheme for the system (1.3). The basic ingredient to construct approximating sequences is a family of iid random variables, which satisfies the following assumption:
**Assumption 1.6.** The family $\left(\xi_{i}\right)_{i \geq 1}$ forms an iid sequence of centered random variables with finite moments of all orders and $\mathbb{E}\left[\xi_{1}^{2}\right]=\sigma^{2}>0$

Following Donsker and Lamperti, we first define, for any $\omega \in \Omega, n \geq 1, t \in \mathbb{I}$, the approximating sequence for the driving Brownian motion $B$ as
$$
B_{n}(t):=\frac{1}{\sigma \sqrt{n}} \sum_{k=1}^{\lfloor n t\rfloor} \xi_{k}+\frac{n t-\lfloor n t\rfloor}{\sigma \sqrt{n}} \xi_{\lfloor n t\rfloor+1}\tag{1.5}
$$
As will be explained later, a similar construction holds to approximate the process $Y$ :
$$
Y_{n}(t):=\frac{1}{n} \sum_{k=1}^{\lfloor n t\rfloor} b\left(Y_{n}^{k-1}\right)+\frac{n t-\lfloor n t\rfloor}{n} b\left(Y_{n}^{\lfloor n t\rfloor}\right)+\frac{1}{\sigma \sqrt{n}} \sum_{k=1}^{\lfloor n t\rfloor} a\left(Y_{n}^{k-1}\right) \zeta_{k}+\frac{n t-\lfloor n t\rfloor}{\sigma \sqrt{n}} a\left(Y_{n}^{\lfloor n t\rfloor}\right) \zeta\lfloor n t\rfloor+1\tag{1.6}
$$
where $Y_{n}^{k}:=Y_{n}\left(t_{k}\right)$ and $\mathcal{T}_{n}:=\left\{t_{k}=\frac{k}{n}\right\} .$ Here $\{\xi\}_{i=1}^{\lfloor n t\rfloor}$ and $\{\zeta\}_{i=1}^{\lfloor n t\rfloor}$ satisfy Assumption $1.6$, with appropriate correlation structure between the pairs $\left\{\left(\zeta_{i}, \xi_{i}\right)\right\}_{i=1}^{\lfloor n t\rfloor}$ that will be made precise later. We shall always use $\left(\xi_{i}\right)$ to denote the sequence generating $B$ and $\left(\zeta_{i}\right)$ the one generating $W$. Consequently, we deduce an approximating scheme (up to the interpolating term which decays to zero by Chebyshev's inequality) for $X$ as
$$
X_{n}(t):=-\frac{1}{2 n} \sum_{k=1}^{\lfloor n t\rfloor} \Phi\left(\mathcal{G}^{\alpha} Y_{n}\right)\left(t_{k}\right)+\frac{1}{\sigma \sqrt{n}} \sum_{k=1}^{\lfloor n t\rfloor} \sqrt{\Phi\left(\mathcal{G}^{\alpha} Y_{n}\right)\left(t_{k}\right)}\left(B_{n}^{k+1}-B_{n}^{k}\right)\tag{1.7}
$$
All the approximations above, as well as all the convergence statements below should be understood pathwise, but we omit the $\omega$ dependence in the notations for clarity. The main result here is a convergence statement about the approximating sequence $\left(X_{n}\right)_{n \geq 1}$． *As usual in weak convergence analysis, convergence is stated in the **Skorokhod space** $\left(\mathcal{D}(\mathbb{I}),\|\cdot\|_{\mathcal{D}}\right)$ of càdlàg processes equipped with the Skorokhod topology.*
**Theorem 1.7.** The sequence $\left(X_{n}\right)_{n \geq 1}$ converges weakly to $X$ in $\left(\mathcal{D}(\mathbb{I}),\|\cdot\|_{\mathcal{D}}\right)$.
The construction of the proof allows to extend the convergence to the case where $Y$ is a $d$-dimensional diffusion without additional work. The proof of the theorem requires a certain number of steps: we start with the convergence of the approximation $\left(Y_{n}\right)$ in some Hölder space, which we translate, first into convergence of the stochastic integral in $(1.3)$, then, by continuity of the mapping $\Phi$, into convergence of the sequence $\left(\Phi\left(\mathcal{G}^{\alpha} Y_{n}\right)\right)$. All these ingredients are detailed in Section 1.3 below. Once this is achieved, the proof of the theorem itself is relatively straightforward.

#### 1.3. Monte-Carlo. 

Theorem 1.7 introduces the theoretical foundations of Monte-Carlo methods (in particular for path-dependent options) for rough volatility models. In this section we give a general and easy-to-understand recipe to implement the class of rough volatility models (1.3). For the numerical recipe to be as general as possible, we shall consider the general time partition $\mathcal{T}:=\left\{t_{i}=\frac{i T}{n}\right\}_{i=0, \ldots, n}$ on $[0, T]$ with $T>0$.

**Algorithm 1.8** (Simulation of rough volatility models).
(1) Simulate two $\mathcal{N}(0,1)$ matrices $\left\{\xi_{j, i}\right\}_{j=1, \ldots, M} \\{i=1, \ldots, n}$ and $\left\{\zeta_{j, i}\right\}_{j=1, \ldots, M} \\{i=1, \ldots, n}$ with $\operatorname{corr}\left(\xi_{j, i}, \zeta_{j, i}\right)=\rho$;
(2) simulate M paths of $Y_{n}$ viad
$$
Y_{n}^{j}\left(t_{i}\right)=\frac{T}{n} \sum_{k=1}^{i} b\left(Y_{n}^{j}\left(t_{k-1}\right)\right)+\frac{T}{\sqrt{n}} \sum_{k=1}^{i} a\left(Y_{n}^{j}\left(t_{k-1}\right)\right) \zeta_{j, k}, \quad i=1, \ldots, n \text { and } j=1, \ldots, M
$$
and also compute
$$
\Delta Y_{n}^{j}\left(t_{i}\right):=Y_{n}^{j}\left(t_{i}\right)-Y_{n}^{j}\left(t_{i-1}\right), \quad i=1, \ldots, n \text { and } j=1, \ldots, M
$$
(3) Simulate $M$ paths of the fractional driving process $\left(\left(\mathcal{G}^{\alpha} Y_{n}\right)(t)\right)_{t \in \mathcal{T}}$ using
$$
\left(\mathcal{G}^{\alpha} Y_{n}\right)^{j}\left(t_{i}\right):=\sum_{k=1}^{i} g\left(t_{i-k+1}\right) \Delta Y_{n}^{j}\left(t_{k}\right)=\sum_{k=1}^{i} g\left(t_{k}\right) \Delta Y_{n}^{j}\left(t_{i-k+1}\right), \quad i=1, \ldots, n \text { and } j=1, \ldots, M
$$
The complexity of this step is in general of order $\mathcal{O}\left(n^{2}\right)$ (see Appendix $[\mathrm{B}$ for details). However, this step is easily implemented using discrete convolution with complexity $\mathcal{O}(n \log n)$ (see Algorithm [B.4 in Appendix $\left[\mathrm{B}\right.$ for details in the implementation). With the vectors $\mathfrak{g}:=\left(g\left(t_{i}\right)\right)_{i=1, \ldots, n}$ and $\Delta Y_{n}^{j}:=$ $\left(\Delta Y_{n}^{j}\left(t_{i}\right)\right)_{i=1, \ldots, n}$ for $j=1, \ldots, M$, we can write $\left(\mathcal{G}^{\alpha} Y_{n}\right)^{j}(\mathcal{T})=\sqrt{\frac{T}{n}}\left(\mathfrak{g} * \Delta Y_{n}^{j}\right)$, for $j=1, \ldots, M$, where $*$ represents the discrete convolution operator.
(4) Use the forward Euler scheme to simulate the log-stock process, for all $i=1, \ldots, n, j=1, \ldots, M$, as
$$
X^{j}\left(t_{i}\right)=X^{j}\left(t_{i-1}\right)-\frac{1}{2} \frac{T}{n} \sum_{k=1}^{i} \Phi\left(\mathcal{G}^{\alpha} Y_{n}\right)^{j}\left(t_{k-1}\right)+\sqrt{\frac{T}{n}} \sum_{k=1}^{i} \sqrt{\Phi\left(\mathcal{G}^{\alpha} Y_{n}\right)^{j}\left(t_{k-1}\right)} \xi_{j, k}
$$

Remark:

- When $Y=W$, we may skip step (2) and replace $\Delta Y_{n}^{j}\left(t_{i}\right)$ by $\sqrt{T / n} \zeta_{i, j}$ on step (33).
- Step (3) may be replaced by the Hybrid scheme algorithm 11 only when $Y=W$.

Antithetic variates in Algorithm 1.8 are easy to implement as it suffices to consider the uncorrelated random vectors $\zeta_{j}:=\left(\zeta_{j, 1}, \zeta_{j, 2}, \ldots, \zeta_{j, n}\right)$ and $\xi_{j}:=\left(\xi_{j, 1}, \xi_{j, 2}, \ldots, \xi_{j, n}\right)$, for $j=1, \ldots, M .$ Then $\left(\rho \xi_{j}+\bar{\rho} \zeta_{j}, \xi_{j}\right),\left(\rho \xi_{j}-\right.$ $\left.\bar{\rho} \zeta_{j}, \xi_{j}\right),\left(-\rho \xi_{j}-\bar{\rho} \zeta_{j},-\xi_{j}\right)$ and $\left(-\rho \xi_{j}+\bar{\rho} \zeta_{j},-\xi_{j}\right)$, for $j=1, \ldots, M$, constitute the antithetic variates, which significantly improves the performance of the Algorithm 1.8 by reducing memory requirements, reducing variance and accelerating execution by exploiting symmetry of the antithetic random variables.

*1.3.1 Enhancing performance.* A standard practice in Monte-Carlo simulation is to match moments of the approximating sequence with the target process. In particular, when the process is Gaussian, matching first and second moments suffices. We only illustrate this approximation for Brownian motion: the left-point approximation may be modified to match moments as
$$
\left(\mathcal{G}^{\alpha} W\right)\left(t_{i}\right) \approx \frac{1}{\sigma \sqrt{n}} \sum_{k=1}^{i} g\left(t_{k}^{*}\right) \zeta_{k}, \quad \text { for } i=0, \ldots, n\tag{1.8}
$$
where $t_{k}^{*}$ is chosen optimally. Since the kernel $g(\cdot)$ is deterministic, there is no confusion with the Stratonovich stochastic integral, and the resulting approximation will always converge to the Itô integral. The first two moments of $\mathcal{G}^{\alpha} W$ read
$$
\mathbb{E}\left(\left(\mathcal{G}^{\alpha} W\right)(t)\right)=0 \quad \text { and } \quad \mathbb{V}\left(\left(\mathcal{G}^{\alpha} W\right)(t)\right)=\int_{0}^{t} g(t-s)^{2} \mathrm{~d} s
$$
The first moment of the approximating sequence 1.8 is always zero, and the second moment reads
$$
\mathbb{V}\left(\frac{1}{\sigma \sqrt{n}} \sum_{k=1}^{j-1} g\left(t_{k}^{*}\right) \zeta_{k}\right)=\frac{1}{n} \sum_{k=1}^{j-1} g\left(t_{k}^{*}\right)^{2}
$$
Equating the theoretical and approximating quantities we obtain $\frac{1}{n} g\left(t_{k}^{*}\right)^{2} \mathrm{~d} s=\int_{t_{k-1}}^{t_{k}} g(t-s)^{2} \mathrm{~d} s$ for $k=1, \ldots, n$, so that the optimal evaluation point can be computed as
$$
g\left(t_{k}^{*}\right)=\sqrt{n \int_{t_{k-1}}^{t_{k}} g(t-s)^{2} \mathrm{~d} s}, \quad \text { for any } k=1, \ldots, n
$$
In the Riemann-Liouville fractional Brownian motion case, $g(u)=u^{H-1 / 2}$, and the optimal point can be computed in closed form as
$$
t_{k}^{*}=\left(\frac{n}{2 H}\left[\left(t-t_{k-1}\right)^{2 H}-\left(t-t_{k}\right)^{2 H}\right]\right)^{1 /(2 H-1)}, \quad \text { for each } k=1, \ldots, n
$$

#### 1.3.2 Reducing Variance. 

As Bayer, Friz and Gatheral, a major drawback in simulating rough volatility models is the very high variance of the estimators, so that a large number of simulations are needed to produce a decent price estimate. Nevertheless, the rDonsker scheme admits a very simple conditional expectation technique which reduces both memory requirements and variance while also admitting antithetic variates. This approach is best suited for calibrating European type options. We consider $\mathcal{F}_{t}^{B}=\sigma\left(B_{s}: s \leq t\right)$ and $\mathcal{F}_{t}^{W}=\sigma\left(W_{s}: s \leq t\right)$ the natural filtrations generated by the Brownian motions $B$ and $W .$ In particular the conditional variance process $V_{t} \mid \mathcal{F}_{t}^{W}$ is deterministic. As discussed by Romano and Touzi, and recently adapted to the rBergomi case by McCrickerd and Pakkanen, we can decompose the stock price process as
$$
\mathrm{e}^{X_{t}}=\mathcal{E}\left(\rho \int_{0}^{t} \sqrt{\Phi\left(\mathcal{G}^{\alpha} Y\right)(s)} \mathrm{d} B_{s}\right) \mathcal{E}\left(\sqrt{1-\rho^{2}} \int_{0}^{t} \sqrt{\Phi\left(\mathcal{G}^{\alpha} Y\right)(s)} \mathrm{d} B_{s}^{\perp}\right):=\mathrm{e}^{X_{t}^{1}} \mathrm{e}^{X_{t}^{2}}
$$
and notice that
$$
X_{t} \mid\left(\mathcal{F}_{t}^{W} \wedge \mathcal{F}_{0}^{B}\right) \sim \mathcal{N}\left(X_{t}^{1}-\left(1-\rho^{2}\right) \int_{0}^{t} \Phi\left(\mathcal{G}^{\alpha} Y\right)(s) \mathrm{d} s,\left(1-\rho^{2}\right) \int_{0}^{t} \Phi\left(\mathcal{G}^{\alpha} Y\right)(s) \mathrm{d} s\right)
$$
Thus $\exp \left(X_{t}\right)$ becomes log-normal and the Black-Scholes closed-form formulae are valid here (European, Barrier options, maximum,...). The advantage of this approach is that the orthogonal Brownian motion $B^{\perp}$ is completely unnecessary for the simulation, hence the generation of random numbers is reduced to a half, yielding proportional memory saving. Not only this, but also this simple trick reduces the variance of the Monte-Carlo estimate, hence fewer simulations are needed to obtain the same precision. We present a simple algorithm to implement the rDonsker with conditional expectation and assuming that $Y=W$.

**Algorithm 1.9** (Simulation of rough volatility models with Brownian drivers). Consider the equidistant grid $\mathcal{T}$.
(1) Draw a random matrix $\left\{\zeta_{j, i}\right\}_{j=1, \ldots, M / 2}$ with unit variance, and create antithetic variates $\left\{-\zeta_{j, i}\right\}_{j=1, \ldots, M / 2}$;
(2) Create a correlated matrix $\left\{\xi_{j, i}\right\}$ as above;
(3) Simulate $M$ paths of the fractional driving process $\mathcal{G}^{\alpha} W$ using discrete convolution:
$$
\left(\mathcal{G}^{\alpha} W\right)^{j}(\mathcal{T})=\sqrt{\frac{T}{n}}\left(\mathfrak{g} * \zeta_{j}\right), \quad j=1, \ldots, M
$$
and store in memory $\left(1-\rho^{2}\right) \int_{0}^{T}\left(\mathcal{G}^{\alpha} W\right)^{j}(s) \mathrm{d} s \approx\left(1-\rho^{2}\right) \frac{T}{n} \sum_{k=0}^{n-1}\left(\mathcal{G}^{\alpha} W\right)^{j}\left(t_{k}\right)=: \Sigma^{j}$ for each $j=1, \ldots, M$
(4) use the forward Euler scheme to simulate the log-stock process, for each $i=1, \ldots, n, j=1, \ldots, M$, as
$$
X^{j}\left(t_{i}\right)=X^{j}\left(t_{i-1}\right)-\frac{\rho^{2}}{2} \frac{T}{n} \sum_{k=1}^{i} \Phi\left(\mathcal{G}^{\alpha} W\right)^{j}\left(t_{k-1}\right)+\rho \sqrt{\frac{T}{n}} \sum_{k=1}^{i} \sqrt{\Phi\left(\mathcal{G}^{\alpha} W\right)^{j}\left(t_{k-1}\right)} \xi_{j, i}
$$
(5) Finally, we have $X^{j}(T) \sim \mathcal{N}\left(X_{T}^{j}-\Sigma^{j}, \Sigma^{j}\right)$ for $j=1, \ldots, M ;$ we may compute any option using the Black-Scholes formula. For instance a Call option with strike $K$ would be given by $C^{j}(K)=$ $\exp \left(X_{T}^{j}\right) \mathcal{N}\left(d_{1}^{j}\right)-K \mathcal{N}\left(d_{2}^{j}\right)$ for $j=1, \ldots, M$, where $d_{1}^{j}:=\frac{1}{\sqrt{\Sigma^{j}}}\left(X_{T}^{j}-\log (K)+\frac{1}{2} \Sigma^{j}\right)$ and $d_{2}^{j}=d_{1}^{j}-\sqrt{\Sigma^{j}}$ Thus, the output of the model would be $C(K)=\frac{1}{M} \sum_{k=1}^{M} C^{j}(K)$

The algorithm is easily adapted to the case of general diffusions $Y$ as drivers of the volatility (see Algorithm 1.8 step 2). Algorithm 1.8 is obviously faster than 1.9, especially when using control variates. Nevertheless, with the same number of paths, Algorithm 1.9 remarkably reduces the Monte-Carlo variance, meaning in turn that fewer simulations are needed, making it very competitive for calibration.

### 2.传统cholesky分解法模拟

>If you need to generate $n$ correlated Gaussian distributed random variables
$$
\mathbf{Y} \sim \mathcal{N}(\mu, \boldsymbol{\Sigma})
$$
where $\mathbf{Y}=\left(Y_{1}, \ldots, Y_{n}\right)$ is the vector you want to simulate, $\mu=\left(\mu_{1}, \ldots, \mu_{n}\right)$ the vector of means and $\Sigma$ the given covariance matrix,
1.you first need to simulate a vector of uncorrelated Gaussian random variables, $\mathbf{Z}$
2.then find a square root of $\Sigma$, i.e. a matrix $\mathbf{C}$ such that $\mathbf{C C}^{\top}=\mathbf{\Sigma}$.
Your target vector is given by
$$
\mathbf{Y}=\mu+\mathbf{C Z}
$$
A popular choice to calculate $\mathbf{C}$ is the Cholesky decomposition.

而对于本 rBergomi 模型，
$$
\begin{aligned}
S_{t} &=S_{0} \mathcal{E}\left(\int_{0}^{t} \sqrt{v_{u}} \mathrm{~d} Z_{u}\right) \\
v_{u} &=\xi_{0}(u) \mathcal{E}\left(\eta \sqrt{2 H} \int_{0}^{u} \frac{1}{(u-s)^{\gamma}} \mathrm{d} W_{s}\right)=\xi_{0}(u) \mathcal{E}\left(\eta \tilde{W}_{u}\right)
\end{aligned}
$$

where $\tilde{W}$ is a Volterra processt with the scaling property $\operatorname{Var}\left[\tilde{W}_{u}\right]=u^{2 H}$. So far $\tilde{W}$ behaves just like $\mathrm{fBm}$. However, the dependence structure is different. Specifically, for $v>u$
$$
\mathbb{E}\left[\tilde{W}_{v} \tilde{W}_{u}\right]=u^{2 H} G\left(\frac{v}{u}\right)
$$
where, for $x \geq 1$ and with $\gamma=1 / 2-H$,
$$
\begin{aligned}
G(x) &=2 H \int_{0}^{1} \frac{\mathrm{d} s}{(1-s)^{\gamma}(x-s)^{\gamma}} \\
&=\frac{1-2 \gamma}{1-\gamma} x^{\gamma}{ }_{2} F_{1}(1, \gamma, 2-\gamma, x)
\end{aligned}
$$
where ${ }_{2} F_{1}(\cdot)$ denotes the confluent hypergeometric function.
Remark $4.1$ The dependence structure of the Volterra process $\tilde{W}$ is markedly different from that of $\mathrm{fBm}$ with the MolchanGolosov kernel $K(u, s)$ given by
$$
\begin{gathered}
K(u, s)=c_{H} \frac{1}{(u-s)^{\gamma}}_{2} F_{1}\left(1 / 2-H, H-1 / 2, H+1 / 2, \frac{s-u}{s}\right) \\
0<s<t
\end{gathered}
$$
for some constant $c_{H} .$ In particular, for small $H$, correlations drop precipitously as the ratio $u / v$ moves away from 1 .

We also need covariances of the Brownian motion $Z$ with the Volterra process $\tilde{W}$. With $v \geq u$, these are given by
$$
\mathbb{E}\left[\tilde{W}_{v} Z_{u}\right]=\rho D_{H}\left\{v^{H+1 / 2}-(v-u)^{H+1 / 2}\right\}
$$
and
$$
\mathbb{E}\left[Z_{v} \tilde{W}_{u}\right]=\rho D_{H} u^{H+1 / 2}
$$
where for future convenience, we have defined the constant,
$$
D_{H}=\frac{\sqrt{2 H}}{H+1 / 2}
$$
These two formulae may be conveniently combined as
$$
\mathbb{E}\left[\tilde{W}_{v} Z_{u}\right]=\rho D_{H}\left\{v^{H+1 / 2}-(v-\min (u, v))^{H+1 / 2}\right\}
$$
Lastly, of course, for $v \geq u, \mathbb{E}\left[Z_{v} Z_{u}\right]=u$.
With $m$ the number of time steps and $n$ the number of simulations, our rBergomi model simulation algorithm may then be summarized as follows.

- Construct the joint covariance matrix for the Volterra process $\tilde{W}$ and the Brownian motion $Z$ and compute its Cholesky decomposition.
- For each time, generate iid normal random vectors and multiply them by the lower triangular matrix obtained by the Cholesky decomposition to get a $m \times 2 n$ matrix of paths of $\tilde{W}$ and $Z$ with the correct joint marginals.
- With these paths held in memory, we may evaluate the expectation under $\mathbb{Q}$ of any payoff of interest.

we simulate the process $\int_0^t (t-s)^{H-1/2}dW_s$
```python
import numpy as np
import matplotlib.pyplot as plt
import scipy.special as special
def fBm_path_chol(grid_points, M, H, T):
    """
    @grid_points: # points in the simulation grid
    @H: Hurst Index
    @T: time horizon
    @M: # paths to simulate
    """
    
    assert 0<H<1.0
    
    ## Step1: create partition 
    
    X=np.linspace(0, 1, num=grid_points)
    
    # get rid of starting point
    X=X[1:grid_points]
    
    ## Step 2: compute covariance matrix
    Sigma=np.zeros((grid_points-1,grid_points-1))
    for j in range(grid_points-1):
        for i in range(grid_points-1):
            if i==j:
                Sigma[i,j]=np.power(X[i],2*H)/2/H
            else:
                s=np.minimum(X[i],X[j])
                t=np.maximum(X[i],X[j])
                Sigma[i,j]=np.power(t-s,H-0.5)/(H+0.5)*np.power(s,0.5+H)*special.hyp2f1(0.5-H, 0.5+H, 1.5+H, -s/(t-s))
        
    ## Step 3: compute Cholesky decomposition
    
    P=np.linalg.cholesky(Sigma)
    
    ## Step 4: draw Gaussian rv
    
    Z=np.random.normal(loc=0.0, scale=1.0, size=[M,grid_points-1])
    
    ## Step 5: get V
    
    W=np.zeros((M,grid_points))
    for i in range(M):
        W[i,1:grid_points]=np.dot(P,Z[i,:])
        
    #Use self-similarity to extend to [0,T] 
    
    return W*np.power(T,H)
```

### 3.rDonker方法

```python
def fBm_path_rDonsker(grid_points, M, H, T, kernel="optimal"):
    """
    @grid_points: # points in the simulation grid
    @H: Hurst Index
    @T: time horizon
    @M: # paths to simulate
    @kernel: kernel evaluation point use "optimal" for momen-match or "naive" for left-point
    """
    
    assert 0<H<1.0
    
    ## Step1: create partition 
    dt=1./(grid_points-1)
    X=np.linspace(0, 1, num=grid_points)
    
    # get rid of starting point
    X=X[1:grid_points]
    
    ## Step 2: Draw random variables
    
    dW = np.power(dt, H) *np.random.normal(loc=0, scale=1, size=[M, grid_points-1])
        
    ## Step 3: compute the kernel evaluation points
    i=np.arange(grid_points-1) + 1
    # By default use optimal moment-matching
    if kernel=="optimal":
        opt_k=np.power((np.power(i,2*H)-np.power(i-1.,2*H))/2.0/H,0.5)
    # Alternatively use left-point evaluation
    elif kernel=="naive" : 
        opt_k=np.power(i,H-0.5)
    else:
        raise NameError("That was not a valid kernel")
    
    
    ## Step 4: Compute the convolution
    
    Y = np.zeros([M, n])
    for i in range(int(M)):
        Y[i, 1:n] = np.convolve(opt_k, dW[i, :])[0:n - 1]
        
    #Use self-similarity to extend to [0,T] 
    
    return Y*np.power(T,H)
```

[Github 代码](https://github.com/amuguruza/RoughFCLT)

## ※使用GAN对LSV模型的校准※

[Cuchiero C, Khosrawi W, Teichmann J. A generative adversarial network approach to calibration of local stochastic volatility models[J]. Risks, 2020, 8(4): 101.](https://www.mdpi.com/2227-9091/8/4/101)

>This means parameterizing the model pool in a way which is accessible for machine learning techniques and interpreting the inverse problem as a training task of a generative network, whose quality is assessed by anadversary.We pursue this approach in the presentarticle  and  use  as  generative  models  so-called  neural  stochastic  differential  equations  (SDE),which just means to parameterize the drift and volatility of an Itˆo-SDE by neural networks.  

### 1．介绍

文中指的**neural SDE**即通过神经网络来对Ito-SDE的漂移项和波动率进行参数化．  
这里考虑的某资产的折现后价格过程（discounted price process） $(S_t)_{t\geq 0}$ ：
$$
dS_t = S_t L(t,S_t)\alpha_t dW_t \tag{1.1}
$$
其中 $(\alpha_t)_{t\geq 0}$ 是某个 $\mathbb{R}$ 中取值的随机过程，$L(t,s)$ 称为**杠杆函数**（Leverage function）取决于 $t$ 和资产当前价格．  
$L$ 的选取非常重要，需要很好地校准市场上观测到的隐含波动率．故 $L$ 需要满足如下条件：
$$
L^{2}(t, s)=\frac{\sigma_{\text {Dup }}^{2}(t, s)}{\mathbb{E}\left[\alpha_{t}^{2} \mid S_{t}=s\right]}\tag{1.1} ,
$$
其中 $\sigma_{\text {Dup }}$ 指 **Dupire** 的local volatility function．注意到（1.1）是 $L$ 的隐式方程，因为 $\mathbb{E}\left[\alpha_{t}^{2} \mid S_{t}=s\right]$ 中需要 $(S_t,\alpha_t)$ ．故此时 $(S_t)_{t\geq 0}$ 满足的SDE也成为了一个**McKean-Vlasov SDE**．  

本文采用了 fully data-driven 方法，规避了其他计算 Dupire 局部波动率的方法中必须的对波动率曲面插值的做法，即此方法只需**离散数据**．  

令 $T_0 = 0$ , $T_1<T_2<...<T_n$ 为不同期权的到期日．使用神经网络族 $F^i:\mathbb{R}\rightarrow \mathbb{R}$  将杠杆函数参数化，参数为 $\theta_i\in \Theta_i$ ，i.e.
$$
L(s,t,\theta) = 1+F^i(s,\theta_i)， t\in [T_{i-1},T_i)，i\in{1,...,n}．
$$  

于是有了neural SDE的生成模型组（generative model class），即使用带参数 $\theta$ 的神经网络来参数化漂移项 $\mu(·,·,\theta)$ 和波动率项 $\sigma(·,·,\theta)$ ，i.e.
$$
d X_{t}(\theta)=\mu\left(X_{t}(\theta), t, \theta\right) d t+\sigma\left(X_{t}(\theta), t, \theta\right) d W_{t}, \quad X_{0}(\theta)=x
$$  
本文中，没有漂移项，波动率项如下所示：
$$
\sigma\left(S_{t}(\theta), t, \theta\right)=S_{t}(\theta)\left(1+\sum_{i=1}^{n} F^{i}\left(S_{t}(\theta), \theta_{i}\right) 1_{\left[T_{i-1}, T_{i}\right)}(t)\right) \alpha_{t} .
$$  

依次对每个到期日，**参数优化**采用如下的校准法则：
$$
\inf _{\theta} \sup_{\gamma} \sum_{j=1}^{J} w_{j}^{\gamma} \ell^{\gamma}\left(\pi_{j}^{\bmod }(\theta)-\pi_{j}^{\mathrm{mkt}}\right)
$$
其中 $J$ 是期权的数目， $\pi_{j}^{\bmod}$ 和 $\pi_{j}^{\mathrm{mkt}}$ 是模型与市场分别的价格．  

对固定的 $\gamma$ ， $\ell^{\gamma}$ 是非线性非负凸函数满足 $\ell^{\gamma}(0) = 0$ 且 $\ell^{\gamma}(x)>0$ 对 $x\neq 0$ ，衡量模型和市场价的距离．$w_{j}^{\gamma}$ 某种权重，参数 $\gamma$ 扮演了**对抗**（adversarial）的部分，注意到 $\ell^{\gamma}$ 和 $w_{j}^{\gamma}$ 都受 $\gamma$ 控制．本文中 $w_{j}^{\gamma}$ 采用的是 [Cont R, Ben Hamida S. Recovering volatility from option prices by evolutionary optimization[J]. 2004.](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=546882)中的 *vega-type*．

### 2.VARIANCE REDUCTION FOR PRICING AND CALIBRATION VIA HEDGING AND DEEP HEDGING

介绍在蒙特卡洛定价和校准中利用对冲投资组合作为控制变量的方差缩减技术．在 LSV 校准中非常重要．

考虑有限时域 $T>0$，已折现的市场中有 $r$ 个交易中的金融产品 $\left(Z_{t}\right)_{t \in[0, T]}$ ，它是在某个概率空间 $\left(\Omega,\left(\mathcal{F}_{t}\right)_{t \in[0, T]}, \mathcal{F}, \mathbb{Q}\right)$ 上在 $\mathbb{R}^{r}$ 中取值的随机变量． $\mathbb{Q}$ 是风险中性测度， $\left(\mathcal{F}_{t}\right)_{t \in[0, T]}$ 假设是右连续的．特别的，假设 $\left(Z_{t}\right)_{t \in[0, T]}$ 是有右连左极路径的 $r$ 维平方可积鞅．

令 $C$ 是 $\mathcal{F}_{T}$ 可测的随机变量，表示表示某个欧式期权在到期日 $T>0$ 的支付．那么通常的对这个期权价格的 Monte Carlo 估计是：
$$
\pi=\frac{1}{N} \sum_{n=1}^{N} C_{n}．\tag{2.1}
$$
其中， $\left(C_{1}, \ldots, C_{N}\right)$ 是以分布 $C$，$N \in \mathbb{N}$  i.i.d 的．可以简单改造这个估计，加上关于 $Z$ 的随机积分．考虑一个策略 $\left(h_{t}\right)_{t \in[0, T]} \in L^{2}(Z)$ 和某个常数 $c$．用 $I=(h \bullet Z)_{T}$ 记关于 $Z$ 的随机积分，考虑如下估计：
$$
\widehat{\pi}=\frac{1}{N} \sum_{n=1}^{N}\left(C_{n}-c I_{n}\right)．\tag{2.2}
$$
其中， $\left(I_{1}, \ldots, I_{N}\right)$ 是以分布 $I$ i.i.d 的．则对于任意的 $\left(h_{t}\right)_{t \in[0, T]} \in L^{2}(Z)$ 和 $c$，这个估计仍是期权价格的无偏估计，因为随机积分的期望消失了．记
$$
H=\frac{1}{N} \sum_{n=1}^{N} I_{n}．
$$
则 $\widehat{\pi}$ 的方差为：
$$
\operatorname{Var}(\widehat{\pi})=\operatorname{Var}(\pi)+c^{2} \operatorname{Var}(H)-2 c \operatorname{Cov}(\pi, H)，
$$
在以下取法下达到最小
$$
c=\frac{\operatorname{Cov}(\pi, H)}{\operatorname{Var}(H)}．
$$
此时
$$
\operatorname{Var}(\widehat{\pi})=\left(1-\operatorname{Corr}^{2}(\pi, H)\right) \operatorname{Var}(\pi)．
$$
特别地，在沿路径完美对冲的情形下，$\pi=H$ a.s.，有 $\operatorname{Corr}(\pi, H)=1$ 和 $\operatorname{Var}(\widehat{\pi})=0$，此时
$$
\operatorname{Var}(\pi)=\operatorname{Var}(H)=\operatorname{Cov}(\pi, H)
$$
因此，找到一个好的近似对冲投资组合使得 $\operatorname{Corr}^{2}(\pi, H)$ 大是很重要的．

#### 2.1 Black&Scholes Delta Hedge

In many cases, of local stochastic volatility models as of form (1.1) and options depending only on the terminal value of the price process, a Delta hedge of the BlackâĂŞScholes model works well.

令 $C=g\left(S_{T}\right)$ ， $\pi_{\mathrm{BS}}^{g}(t, T, s, \sigma)$ 是 BS 模型下 $t$ 时刻的价格．对冲策略为：
$$
h_{t}=\partial_{s} \pi_{\mathrm{BS}}^{g}\left(t, T, S_{t}, L\left(t, S_{t}\right) \alpha_{t}\right)
$$

#### 2.2 Hedging Strategies as Neural Networks-Deep Hedging

在对冲产品数很多等情况下时，可以将对冲策略用神经网络参数化．令期权的支付是对冲产品最终价值的函数，i.e.，$C=g\left(Z_{T}\right)$．在**马尔科夫模型**中，可以用函数表示对冲策略：
$$
h: \mathbb{R}_{+} \times \mathbb{R}^{r} \rightarrow \mathbb{R}^{r}, h_{t}=h(t, z)
$$
对应这样一个神经网络：$(t, z) \mapsto h(t, z, \delta) \in \mathcal{N} \mathcal{N}_{r+1, r}$．$\delta$ 是网络参数．根据[Buehler H, Gonon L, Teichmann J, et al. Deep hedging[J]. Quantitative Finance, 2019, 19(8): 1271-1291.](https://www.tandfonline.com/doi/full/10.1080/14697688.2019.1571683) 给定 $\pi^{\mathrm{mkt}}$ 的最优对冲可以如下计算
$$
\inf _{\delta \in \Delta} \mathbb{E}\left[u\left(-C+\pi^{\mathrm{mkt}}+\left(h\left(\cdot, Z_{--}, \delta\right) \bullet Z \cdot\right)_{T}\right)\right]
$$
$u: \mathbb{R} \rightarrow \mathbb{R}_{+}$ 是凸的损失函数．

为了解决这个最优问题，采用随机梯度下降，随机目标函数 $Q(\delta)(\omega)$ 为：
$$
Q(\delta)(\omega)=u\left(-C(\omega)+\pi^{\mathrm{mkt}}+\left(h\left(\cdot, Z_{.-}, \delta\right)(\omega) \bullet Z .(\omega)\right)_{T}\right)
$$
记最优的参数 $\delta^{*}$ 和最优对冲策略 $h\left(\cdot, \cdot, \delta^{*}\right)$ ．

假定激活函数和凸损失函数是光滑的．下面要证明 $Q(\delta)$ 的梯度是：
$$
\nabla_{\delta} Q(\delta)(\omega)=u^{\prime}\left(-C(\omega)+\pi^{\mathrm{mkt}}+\left(h\left(\cdot, Z_{--}, \delta\right)(\omega) \bullet Z \cdot(\omega)\right)_{T}\right)\left(\nabla_{\delta} h\left(\cdot, Z_{.-}, \delta\right)(\omega) \bullet Z .(\omega)\right)_{T}
$$
i.e.，我们可以把梯度移到随机积分中．为此，我们要使用下述定理．

**定理 2.1**：$\forall\varepsilon \geq 0$，令 $Z^{\varepsilon}$ 是

<div id="2_1"></div>

Theorem 2.1. For ling, let $Z^{\varepsilon}$ be a solution of a stochastic differential equation as described in Theorem $A .3$ with drivers $Y=\left(Y^{1}, \ldots, Y^{d}\right)$, functionally Lipschitz operators $F_{j}^{\varepsilon, i}, i=1, \ldots, r$, $j=1, \ldots, d$ and a process $\left(J^{\varepsilon, 1}, \ldots J^{\varepsilon, r}\right)$, which is here for all $\varepsilon \geq 0$ simply $J 1_{\{t=0\}}(t)$ for some constant vector $J \in \mathbb{R}^{r}=J$, i.e.
$$
Z_{t}^{\varepsilon, i}=J^{i}+\sum_{j=1}^{d} \int_{0}^{t} F_{j}^{\varepsilon, i}\left(Z^{\varepsilon}\right)_{s-} d Y_{s}^{j}, \quad t \geq 0
$$
Let $(\varepsilon, t, z) \mapsto f^{\varepsilon}(t, z)$ be a map, such that the bounded càglàd process $f^{\varepsilon}:=f^{\varepsilon}\left(.-, Z_{.-}^{0}\right)$ converges $u c p$ to $f^{0}:=f^{0}\left(.-, Z_{.-}^{0}\right)$, then
$$
\lim _{\varepsilon \rightarrow 0}\left(f^{\varepsilon} \bullet Z^{\varepsilon}\right)=\left(f^{0} \bullet Z^{0}\right)
$$
holds true.

[证明过程](#proof_2_1)

**推论 2.2**：$\forall\varepsilon>0$，令 $Z^{\varepsilon}$ 为对冲产品过程 $Z \equiv Z^{0}$ 的离散，使得定理 2.1 中的条件都满足．对应的对冲策略 $(t, z, \delta) \mapsto h^{\varepsilon}(t, z, \delta)$ 由神经网络 $\mathcal{N} \mathcal{N}_{r+1, r}$ 给出，其中网络的激活函数有界 $C^{1}$，且导数有界．那么

(i) 随机积分在 $\delta_{0}$ 点关于 $\delta$  导数 $\nabla_{\delta}(h(\cdot, Z .-, \delta) \bullet Z)$ 满足
$$
\nabla_{\delta}\left(h\left(\cdot, Z_{--}, \delta_{0}\right) \bullet Z\right)=\left(\nabla_{\delta} h\left(\cdot, Z_{-}, \delta_{0}\right) \bullet Z\right)
$$

(ii) 若当 $\varepsilon \rightarrow 0$ 时，$\nabla_{\delta} h^{\varepsilon}\left(\cdot, Z_{--}, \delta_{0}\right)$ ucp 收敛到 $\nabla_{\delta} h\left(\cdot, Z_{--}, \delta_{0}\right)$，则离散积分的方向导数，i.e. $\nabla_{\delta}\left(h^{\varepsilon}\left(\cdot, Z_{-}^{\varepsilon}, \delta_{0}\right) \bullet Z^{\varepsilon}\right)$ 随着离散刻度 $\varepsilon \rightarrow 0$ 收敛到
$$
\lim _{\varepsilon \rightarrow 0}\left(\nabla_{\delta} h^{\varepsilon}\left(\cdot, Z_{--}^{\varepsilon}, \delta_{0}\right) \bullet Z^{\varepsilon}\right)=\left(\nabla_{\delta} h\left(\cdot, Z_{\cdot-}, \delta_{0}\right) \bullet Z\right)
$$
> ucp means *uniform convergence on compacts in probability*,i.e.，if
$$
\mathbb{P}\left(\sup_{s<t}\left|X_{s}^{n}-X_{s}\right|>\epsilon\right) \rightarrow 0
$$
for all $\epsilon, t>0$. The notation $X^{n} \stackrel{\text { ucp }}{\longrightarrow} \mathrm{X}$ is sometimes used, and $X^{n}$ is said to converge ucp to $X$.

### 3. LSV的校准

考虑定义在某个概率空间 $\left(\Omega,\left(\mathcal{F}_{t}\right)_{t \in[0, T]}, \mathcal{F}, \mathbb{Q}\right)$ 上的（1.1）LSV模型，$\mathbb{Q}$ 是风险中性测度．假定随机过程 $\alpha$ 固定．**所以实际中我们可以先令 $L\equiv 1$ 来近似校准其他参数并固定他们**．

主要目标是确定符合市场数据的杠杆函数 $L$，根据通用近似定理（universal approximation properties），对其参数化．令 $T_{0}=0，0<T_{1} \cdots<T_{n}=T$ 为欧式看涨期权的到期日．将 $L(t, s)$ 用如下神经网络近似
$$
L(t, s, \theta)=\left(1+\sum_{i=1}^{n} F^{i}\left(s, \theta_{i}\right) 1_{\left[T_{i-1}, T_{i}\right)}(t)\right)\tag{3.1}
$$
其中 $F^{i} \in \mathcal{N} \mathcal{N}_{1,1}$， $i=1, \ldots, n$．方便起见，通常省略 $\theta_{i} \in \Theta_{i}$．当我们写 $S_{t}(\theta)$ 时，$\theta$ 表示 $t$ 时刻前所有的参数 $\theta_{i}$． 

训练过程中，我们需要计算 LSV 过程关于 $\theta$ 的导数．以下结果可以看做 $\nabla_{\theta} S(\theta)$ 对应的链式法则．从附录 A 推导而来．

<div id="3_1"></div>

**定理 3.1**：令 $(t, s, \theta) \mapsto L(t, s, \theta)$ 为（3.1）形式，神经网络 $\left(s, \theta_{i}\right) \mapsto F^{i}\left(s, \theta_{i}\right)$ 有界且 $C^{1}$，导数有界且 Lipschitz 连续．则关于 $\theta$ 在 $\widehat{\theta}$ 点处的导数满足：
$$
\begin{aligned}
d\left(\nabla_{\theta} S_{t}(\widehat{\theta})\right)=&\left(\nabla_{\theta} S_{t}(\widehat{\theta}) L\left(t, S_{t}(\widehat{\theta}), \widehat{\theta}\right)+S_{t}(\widehat{\theta}) \partial_{s} L\left(t, S_{t}(\widehat{\theta}), \widehat{\theta}\right) \nabla_{\theta} S_{t}(\widehat{\theta})\right.\\
&\left.+S_{t}(\widehat{\theta}) \nabla_{\theta} L\left(t, S_{t}(\widehat{\theta}), \widehat{\theta}\right)\right) \alpha_{t} d W_{t}
\end{aligned}\tag{3.2}
$$
初值为 0．这个可以通过常数变易来解，i.e.
$$
\nabla_{\theta} S_{t}(\widehat{\theta})=\int_{0}^{t} P_{t-s} S_{s}(\widehat{\theta}) \nabla_{\theta} L\left(s, S_{s}(\widehat{\theta}), \widehat{\theta}\right) \alpha_{s} d W_{s}\tag{3.3}
$$
其中
$$
P_{t}=\mathcal{E}\left(\int_{0}^{t}\left(L\left(s, S_{s}(\widehat{\theta}), \widehat{\theta}\right)+S_{s}(\widehat{\theta}) \nabla_{s} L\left(s, S_{s}(\widehat{\theta}), \widehat{\theta}\right)\right) \alpha_{s} d W_{s}\right)
$$
$\mathcal{E}$ 表示随机指数（stochastic exponential）．

[证明过程](#proof_3_1)
<div id="3_1"></div>

*Remark $3.2$*

(i) 只看存在唯一性的话，
$$
d S_{t}(\theta)=S_{t}(\theta) L\left(t, S_{t}(\theta), \theta\right) \alpha_{t} d W_{t}
$$
$L(t, s, \theta)$ 为 (3.1) 形式，那么神经网络 $s \mapsto F^{i}\left(s, \theta_{i}\right)$ 有界以及 Lipschitz 足够了，$\forall i=1, \ldots, n$．

(ii) 公式 (3.3) 可以用来倒向传播．

定理 3.1 保证了导数过程的存在唯一性．这也保证了基于梯度搜索的学习算法的建立．

下面叙述如何具体优化．为了记号方便，省略权重 $w$ 和损失函数 $\ell$ 对应的参数 $\gamma$．对每个到期日 $T_{i}$，我们假定有 $J_{i}$ 个期权，行权价为 $K_{i j}, j \in\left\{1, \ldots, J_{i}\right\}$．对第 $i$ 个到期日，校准函数的形式为
$$
\underset{\theta_{i} \in \Theta_{i}}{\operatorname{argmin}} \sum_{j=1}^{J_{i}} w_{i j} \ell\left(\pi_{i j}^{\bmod }\left(\theta_{i}\right)-\pi_{i j}^{\mathrm{mkt}}\right), \quad i \in\{1, \ldots, n\}\tag{3.5}
$$
回忆 $\pi_{i j}^{\bmod }\left(\theta_{i}\right)$ 指的是对应到期日 $T_{i}$ 和行权价 $K_{i j}$ 的模型期权价格．$\ell: \mathbb{R} \rightarrow \mathbb{R}_{+}$ 是某个非负非线性凸的损失函数满足 $\ell(0)=0，\ell(x)>0$ 对 $x \neq 0$．$w_{i j}$ 是权重．

我们通过迭代地计算最优化问题（3.5），从 $T_{1}$ 和 $\theta_{1}$ 出发，计算 $\pi_{2 j}^{\bmod }\left(\theta_{2}\right)$，然后解决对应 $T_{2}$ 的（3.5）．为了简便记号，去掉 $i$ ，考虑一般的到期日 $T>0$ ，(3.5)变为
$$
\underset{\theta \in \Theta}{\operatorname{argmin}} \sum_{j=1}^{J} w_{j} \ell\left(\pi_{j}^{\bmod }(\theta)-\pi_{j}^{\mathrm{mkt}}\right)
$$
模型价格由下式给出
$$
\pi_{j}^{\bmod }(\theta)=\mathbb{E}\left[\left(S_{T}(\theta)-K_{j}\right)^{+}\right]\tag{3.6}
$$
我们有 $\pi_{j}^{\bmod }(\theta)-\pi_{j}^{\mathrm{mkt}}=\mathbb{E}\left[Q_{j}(\theta)\right]$ ，其中
$$
Q_{j}(\theta)(\omega):=\left(S_{T}(\theta)(\omega)-K_{j}\right)^{+}-\pi_{j}^{\mathrm{mkt}}\tag{3.7}
$$
那么校准问题变为寻找最小的
$$
f(\theta):=\sum_{j=1}^{J} w_{j} \ell\left(\mathbb{E}\left[Q_{j}(\theta)\right]\right)\tag{3.8}
$$
因为 $\ell$ 是非线性函数，不是 B.1 中的期望形式，标准的随机梯度下降方法不能直接用．我们通过第二节中讲的对冲控制变量 (hedge control variates) 解决这个问题．

#### 3.1 极小化校准方程

考虑标准的对（3.8） $\mathbb{E}\left[Q_{j}(\theta)\right]$ 的 Monte-Carlo 模拟：
$$
f^{\mathrm{MC}}(\theta):=\sum_{j=1}^{J} w_{j} \ell\left(\frac{1}{N} \sum_{n=1}^{N} Q_{j}(\theta)\left(\omega_{n}\right)\right)\tag{3.9}
$$
对 i.i.d 的样本 $\left\{\omega_{1}, \ldots, \omega_{N}\right\} \in \Omega$ ．Monte-Carlo 误差以 $\frac{1}{\sqrt{N}}$ 递减．模拟次数 $N$ 必须很大 $\left(\approx 10^{8}\right)$ ．因为由于 $\ell$ 非线性，随机梯度下降不能直接使用，所以看起来要计算整个函数 $\widehat{f}(\theta)$ 的梯度来最小化（3.9）．但 $N \approx 10^{8}$ ，这一做法计算成本太大且不稳定，因为要计算 $10^{8}$ 项的和的导数．

一个方便的做法是应用对冲控制变量来降低方差，可以将 Monte-Carlo 的样本数 $N$ 降为大约 $5 \times 10^{4}$ ．

假定我们有 $r$ 个对冲产品（包含价格过程 $S$ ），用 $\left(Z_{t}\right)_{t \in[0, T]}$ 表示，为 $\mathbb{Q}$ 下的平方可积鞅，在 $\mathbb{R}^{r}$ 下取值．对 $j=1, \ldots, J$ ，策略 $h_{j}:[0, T] \times \mathbb{R}^{r} \rightarrow \mathbb{R}^{r}$ 使得 $h(\cdot, Z .) \in L^{2}(Z)$ ，$c$ 为常数，定义
$$
X_{j}(\theta)(\omega):=Q_{j}(\theta)(\omega)-c\left(h_{j}\left(\cdot, Z_{.-}(\theta)(\omega)\right) \bullet Z .(\theta)(\omega)\right)_{T}\tag{3.10}
$$
则校准函数（3.8）和（3.9）可以通过替换 $Q_{j}(\theta)(\omega)$ 为 $X_{j}(\theta)(\omega)$ 来定义，变为最小化
$$
\widehat{f}(\theta)\left(\omega_{1}, \ldots, \omega_{N}\right)=\sum_{j=1}^{J} w_{j} \ell\left(\frac{1}{N} \sum_{n=1}^{N} X_{j}(\theta)\left(\omega_{n}\right)\right)\tag{3.11}
$$
对此，我们应用如下梯度下降的变种：从初始猜测 $\theta^{(0)}$ 出发，迭代计算
$$
\theta^{(k+1)}=\theta^{(k)}-\eta_{k} G\left(\theta^{(k)}\right)\left(\omega_{1}^{(k)}, \ldots, \omega_{N}^{(k)}\right)\tag{3.12}
$$
对某个学习率 $\eta_{k}$ ，i.i.d 样本 $\left(\omega_{1}^{(k)}, \ldots, \omega_{N}^{(k)}\right)$ ．其中
$$
G\left(\theta^{(k)}\right)\left(\omega_{1}^{(k)}, \ldots, \omega_{N}^{(k)}\right)
$$
是基于梯度待确定的量，样本在每次迭代中可以一样，可以另取．本文中另取．

最简单情形下，可以令
$$
G\left(\theta^{(k)}\right)\left(\omega_{1}^{(k)}, \ldots, \omega_{N}^{(k)}\right)=\nabla \widehat{f}(\theta)\left(\omega_{1}^{(k)}, \ldots, \omega_{N}^{(k)}\right)\tag{3.13}
$$

注意到（3.10）中随机积分项的导数计算通常是昂贵的．我们进行下述改造．令
$$
\begin{aligned}
\omega^{N} &=\left(\omega_{1}, \ldots, \omega_{N}\right) \\
Q_{j}^{N}(\theta)\left(\omega^{N}\right) &=\frac{1}{N} \sum_{n=1}^{N} Q_{j}(\theta)\left(\omega_{n}\right) \\
Q^{N}(\theta)\left(\omega^{N}\right) &=\left(Q_{1}^{N}(\theta)\left(\omega^{N}\right), \ldots, Q_{J}^{N}(\theta)\left(\omega^{N}\right)\right)
\end{aligned}
$$
定义 $\tilde{f}: \mathbb{R}^{J} \rightarrow \mathbb{R}$ ：
$$
\tilde{f}(x)=\sum_{j=1}^{J} w_{j} \ell\left(x_{j}\right)
$$
然后令
$$
G(\theta)\left(\omega^{N}\right)=D_{x}(\tilde{f})\left(X^{N}(\theta)\left(\omega^{N}\right)\right) D_{\theta}\left(Q^{N}\right)(\theta)\left(\omega^{N}\right)
$$
注意到基于倒向传播，这一项计算起来是很简单的．Moreover, leaving the stochastic integral away in the inner derivative is justified by its vanishing expectation. During the forward pass, the stochastic integral terms are included in the computation; however the contribution to the gradient (during the backward pass) is partly neglected, which can e.g. be implemented via the tensorflow stop_gradient function.

关于对冲策略的选择，我们可以按照 2.2 节中的方法将其用神经网络参数化，并通过下式计算最优的权重 $\delta$ ：
$$
\underset{\delta \in \Delta}{\operatorname{argmin}} \frac{1}{N} \sum_{n=1}^{N} u\left(-X_{j}(\theta, \delta)\left(\omega_{n}\right)\right)
$$
对 i.i.d 样本 $\left\{\omega_{1}, \ldots, \omega_{N}\right\} \in \Omega$ 和损失函数 $u$ ．此处
$$
X_{j}(\theta, \delta)(\omega)=\left(S_{T}(\theta)(\omega)-K_{j}\right)^{+}-\left(h_{j}\left(\cdot, Z_{--}(\theta)(\omega), \delta\right) \bullet Z .(\theta)(\omega)\right)_{T}-\pi_{j}^{\mathrm{mkt}}
$$
这意味着迭代两个优化步骤，i.e.，优化（3.11）中的 $\theta$ （固定 $\delta$ ） 和（3.14）中的 $\delta$ （固定 $\theta$ ）．

### 4. 数值实验流程

实际使用的 **SABR-LSV** 模型如下
$$
\begin{aligned}
d S_{t} &=S_{t} L\left(t, S_{t}\right) \alpha_{t} d W_{t} \\
d \alpha_{t} &=\nu \alpha_{t} d B_{t} \\
d\langle W, B\rangle_{t} &=\varrho d t
\end{aligned}
$$
参数为 $\nu \in \mathbb{R}, \varrho \in[-1,1]$，初值有 $\alpha_{0}>0, S_{0}>0$．$B$ 和 $W$ 是两个相关的布朗运动．

Remark：一般使用的是关于 $S$ 的对数价格 $X:=\log S$．故模型也可写为：
$$
\begin{aligned}
d X_{t} &=\alpha_{t} L\left(t, X_{t}\right) d W_{t}-\frac{1}{2} \alpha_{t}^{2} L^{2}\left(t, X_{t}\right) d t \\
d \alpha_{t} &=\nu \alpha_{t} d B_{t} \\
d\langle W, B\rangle_{t} &=\varrho d t
\end{aligned}
$$
注意到 $\alpha$ 是一个几何布朗运动，也就是说它有表达式：
$$
\alpha_{t}=\alpha_{0} \exp \left(-\frac{\nu^{2}}{2} t+\nu B_{t}\right)
$$

#### 生成样本

在已有文献中，有推荐的局部波动率函数族 $\widetilde{a}_{\xi}$ 如下：
$$
\xi=\left(p_{1}, p_{2}, \sigma_{0}, \sigma_{1}, \sigma_{2}\right)
$$
其中 $p_{0}=1-\left(p_{1}+p_{2}\right)$ 且参数满足如下约束：
$$
\sigma_{0}, \sigma_{1}, \sigma_{2}, p_{1}, p_{2}>0 \text { and } p_{1}+p_{2} \leq 1．
$$
令 $k(t, x, \sigma)=\exp \left(-x^{2} /\left(2 t \sigma^{2}\right)-t \sigma^{2} / 8\right)$，$\widetilde{a}_{\xi}$ 如下定义：
$$
\widetilde{a}_{\xi}^{2}(t, x)=\frac{\sum_{i=0}^{2} p_{i} \sigma_{i} k\left(t, x, \sigma_{i}\right)}{\sum_{i=0}^{2}\left(p_{i} / \sigma_{i}\right) k\left(t, x, \sigma_{i}\right)}
$$
文中作者修改为：
$$
a_{\xi}^{2}(t, x)=\frac{1}{4} \times \min \left(2,\left|\frac{\left(\sum_{i=0}^{2} p_{i} \sigma_{i} k\left(t, x, \sigma_{i}\right)+\Lambda(t, x)\right)\left(1-0.6 \times \mathbb{1}_{(t>0.1)}\right)}{\sum_{i=0}^{2}\left(p_{i} / \sigma_{i}\right) k\left(t, x, \sigma_{i}\right)+0.01}\right|\right)
$$
其中
$$
\Lambda(t, x):=\left(\frac{\mathbb{1}_{(t \leq 0.1)}}{1+0.1 t}\right)^{\lambda_{2}} \min \left\{\left(\gamma_{1}\left(x-\beta_{1}\right)_{+}+\gamma_{2}\left(-x-\beta_{2}\right)_{+}\right)^{\kappa}, \lambda_{1}\right\}
$$
注意到 $a_{\xi}^{2}$ 与 $t$ 有关．所以在做 Monte Carlo 模拟时，我们将 $a_{\xi}^{2}(0, x)$ 替换为 $a_{\xi}^{2}\left(\Delta_{t}, x\right)$，$\Delta_{t}$ 是 Monte Carlo 模拟的时间间隔．
What is left to be specified are the parameters
$$
\xi=\left(p_{1}, p_{2}, \sigma_{0}, \sigma_{1}, \sigma_{2}\right)
$$
模型变为：
$$
d X_{t}=-\frac{1}{2} a_{\xi}^{2}\left(t, X_{t}\right) d t+a_{\xi}\left(t, X_{t}\right) d W_{t}\tag{1}
$$
上式是用来生成人工市场价格样本的．

所以我们实际的做法是随机对 $a_{\xi}$ 中的 $\xi$ 采样再根据 (1) 计算出价格，然后对 SABR-LSV 模型进行校准，i.e. 寻找使模型符合上述价格的参数 $\nu，\varrho$，$\alpha_{0}$ 以及 $L$．

到期日 $T_{1}<\cdots<T_{n}$，每个到期日 $T_{i}$ 对应行权价为 $K_{i j}, j \in\left\{1, \ldots, J_{i}\right\}$．用 Monte-Carlo 模拟以 $\Delta_{t}=1 / 100$ 间隔计算价格．

具体如下：

- 在 $m=1, \ldots, 200$ 下对 $\xi_{m}$ 以给定分布进行模拟．
- 对每个 $m$，根据（1）式计算 $T_{i}$ and strikes $K_{i j}$ for $i=1, \ldots, n=4$ and $j=1, \ldots, 20$ 对应的欧式期权的价格．每个 $m$ 分别使用不同的 $10^{7}$ 条布朗运动轨道．
- 保存这些价格数据

## $\dagger$准备做的工作$\dagger$（弃案）

寻找最适合市场波动率曲面的“**复合**”模型，即假设市场波动率曲面实际是由**一些**波动率模型的**凸组合**决定的．

>回忆：波动率曲面即隐含波动率以 $T$：time to maturity 和 $\log(K/S_0)$：log-moneyness 为自变量构成的曲面．
例如，我们可以假设当前 $\{\sigma^{T,K}_{market}=a\sigma^{T,K}_{Heston}+b\sigma^{T,K}_{rough}\}_{T,K}$，其中 $a+b=1，a\geq0，b\geq0$．

### 大致做法

**记号**：分别以 $\xi_1$、$\xi_2$ 记 Heston 和 rough 模型的参数集，以 $N_1$、$N_2$ 记该两者通过神经网络训练得到的从模型参数到市场价格（隐含波动率）的映射，$a$、$b$ 为前述凸组合系数．

我们这里考虑**直接通过神经网络来学习市场波动率曲面 $\{\sigma_{market}^{T,K}\}$ 到凸组合系数 $a,b$ 的映射**．

我们对两个模型的参数以及 $a,b$ 分别均匀采样，然后根据两者的模型分别模拟出不同凸组合下两者的复合波动率曲面，*但要注意的是两者采用同一个参数 $\rho$（即两个标准布朗运动的相关系数）并且一个凸组合下两模型使用同一条 Monte-Carlo 轨道*．这时，忽略掉模型的参数，我们有了带有 $\{a,b\}$ 标签的许多波动率曲面样本，我们利用前述 grid-based 的方法通过神经网络**学习从波动率曲面到凸组合系数的映射**．

知道了 $a,b$ 后，如何校准出两个模型分别的参数？

## $\dagger$LSV-ROUGH 模型的校准$\dagger$

模型：
$$
\begin{aligned}
d S_{t} &=S_{t} L\left(t, S_{t}\right)\sqrt{V_t} d W_{t} \\
V_{t} &=\xi_{0}(t) \mathcal{E}\left(\sqrt{2 H} \eta \int_{0}^{t}(t-s)^{H-1 / 2} d Z_{s}\right), \quad \text { for } t>0, \quad V_{0}=v_{0}>0\\
d\langle W, B\rangle_{t} &=\varrho dt
\end{aligned}\tag{1.1}
$$

杠杆函数：$L^{2}(t, s)=\frac{\sigma_{\text {Dup }}^{2}(t, s)}{\mathbb{E}\left[\alpha_{t}^{2} \mid S_{t}=s\right]}$

**主要共有两个神经网络**，一个负责 Rough 的部分，一个负责 LSV 的部分．

一方面，Rough 部分的网络对应的即 Bayer 提出的 two-step 校准方法，即如下模型 （（1.1）中 $L \equiv 1$ 时）： 
$$
\begin{aligned}
d S_{t} &=S_{t} \sqrt{V_t} d W_{t} \\
V_{t} &=\xi_{0}(t) \mathcal{E}\left(\sqrt{2 H} \eta \int_{0}^{t}(t-s)^{H-1 / 2} d Z_{s}\right), \quad \text { for } t>0, \quad V_{0}=v_{0}>0\\
d\langle W, B\rangle_{t} &=\varrho dt
\end{aligned}\tag{1.1}
$$
对应的从模型参数到模型对应价格的映射的网络．只需用人工模拟数据训练一次后，网络就固定住了，在校准等步骤中是不会再变动的．

>**回忆**：用 $\Delta:=\left\{k_{i}, T_{j}\right\}_{i=1, j=1}^{n,} m_{i=1}$ 记关于到期日和行权价的网格．则
**step 1**：学习映射 $\widetilde{F}(\theta)=\left\{\sigma_{B S}^{\mathcal{M}(\theta)}\left(T_{i}, k_{j}\right)\right\}_{i=1, j=1}^{n,m}$，输入是 $\theta \in \Theta$，输出是 $\left\{\sigma_{\mathrm{BS}}^{\mathcal{M}(\theta)}\left(T_{i}, k_{j}\right)\right\}_{i=1, j=1}^{n, m}$ 这样的  $n \times m$ 网格．$\widetilde{F}$ 取值在 $\mathbb{R}^{L}$ 中，其中 $L=$ strikes $\times$ maturities $=n m .$最优化问题变为如下：
$$
\widehat{\omega}:=\underset{w \in \mathbb{R}^{n}}{\operatorname{argmin}} \sum_{i=1}^{N_{\text {Train }}^{\text {reduced }}} \sum_{j=1}^{L} \eta_{j}\left(\widetilde{F}\left(\theta_{i}\right)_{j}-\sigma^{\mathcal{M}}\left(\theta_{i}, T_{j}, k_{j}\right)\right)^{2}
$$
其中 $N_{\text {Train }}=N_{\text {Train }}^{\text {reduced }} \times L$．
**Step 2**:
$$
\widehat{\theta}:=\underset{\theta \in \Theta}{\operatorname{argmin}} \sum_{i=1}^{L} \beta_{j}\left(\widetilde{F}(\theta)_{i}-\sigma_{\mathrm{BS}}^{\mathrm{MKT}}\left(T_{i}, k_{i}\right)\right)^{2}
$$

另一方面，我们将 $L\left(t, S_{t}\right)$ 这个函数用网络近似，这个网络中的参数是随着校准不断变动的．具体地，令 $T_{0}=0，0<T_{1} \cdots<T_{n}=T$ 为欧式看涨期权的到期日．将 $L(t, s)$ 用如下神经网络近似
$$
L(t, s, \theta)=\left(1+\sum_{i=1}^{n} F^{i}\left(s, \theta_{i}\right) 1_{\left[T_{i-1}, T_{i}\right)}(t)\right)\tag{1.2}
$$
其中 $F^{i} \in \mathcal{N} \mathcal{N}_{1,1}$， $i=1, \ldots, n$．

为了记号方便，省略权重 $w$ 和损失函数 $\ell$ 对应的参数 $\gamma$．对每个到期日 $T_{i}$，我们假定有 $J_{i}$ 个期权，行权价为 $K_{i j}, j \in\left\{1, \ldots, J_{i}\right\}$．对第 $i$ 个到期日，校准函数的形式为
$$
\underset{\theta_{i} \in \Theta_{i}}{\operatorname{argmin}} \sum_{j=1}^{J_{i}} w_{i j} \ell\left(\pi_{i j}^{\bmod }\left(\theta_{i}\right)-\pi_{i j}^{\mathrm{mkt}}\right), \quad i \in\{1, \ldots, n\}\tag{1.3}
$$
回忆 $\pi_{i j}^{\bmod }\left(\theta_{i}\right)$ 指的是对应到期日 $T_{i}$ 和行权价 $K_{i j}$ 的模型期权价格．$\ell: \mathbb{R} \rightarrow \mathbb{R}_{+}$ 是某个非负非线性凸的损失函数满足 $\ell(0)=0，\ell(x)>0$ 对 $x \neq 0$．$w_{i j}$ 是权重．

我们通过迭代地计算最优化问题（1.3），从 $T_{1}$ 和 $\theta_{1}$ 出发，计算 $\pi_{2 j}^{\bmod }\left(\theta_{2}\right)$，然后解决对应 $T_{2}$ 的（1.3）．为了简便记号，去掉 $i$ ，考虑一般的到期日 $T>0$ ，(1.3)变为
$$
\underset{\theta \in \Theta}{\operatorname{argmin}} \sum_{j=1}^{J} w_{j} \ell\left(\pi_{j}^{\bmod }(\theta)-\pi_{j}^{\mathrm{mkt}}\right)
$$
模型价格由下式给出
$$
\pi_{j}^{\bmod }(\theta)=\mathbb{E}\left[\left(S_{T}(\theta)-K_{j}\right)^{+}\right]\tag{1.4}
$$
我们有 $\pi_{j}^{\bmod }(\theta)-\pi_{j}^{\mathrm{mkt}}=\mathbb{E}\left[Q_{j}(\theta)\right]$ ，其中
$$
Q_{j}(\theta)(\omega):=\left(S_{T}(\theta)(\omega)-K_{j}\right)^{+}-\pi_{j}^{\mathrm{mkt}}\tag{1.5}
$$
那么校准问题变为寻找最小的
$$
f(\theta):=\sum_{j=1}^{J} w_{j} \ell\left(\mathbb{E}\left[Q_{j}(\theta)\right]\right)\tag{1.6}
$$
我们通过第二节中讲的对冲控制变量 (hedge control variates) 解决这个问题．

考虑标准的对（3.8） $\mathbb{E}\left[Q_{j}(\theta)\right]$ 的 Monte-Carlo 模拟：
$$
f^{\mathrm{MC}}(\theta):=\sum_{j=1}^{J} w_{j} \ell\left(\frac{1}{N} \sum_{n=1}^{N} Q_{j}(\theta)\left(\omega_{n}\right)\right)\tag{1.7}
$$
对 i.i.d 的样本 $\left\{\omega_{1}, \ldots, \omega_{N}\right\} \in \Omega$ ．Monte-Carlo 误差以 $\frac{1}{\sqrt{N}}$ 递减．模拟次数 $N$ 必须很大 $\left(\approx 10^{8}\right)$ ．因为由于 $\ell$ 非线性，随机梯度下降不能直接使用，所以看起来要计算整个函数 $\widehat{f}(\theta)$ 的梯度来最小化（3.9）．但 $N \approx 10^{8}$ ，这一做法计算成本太大且不稳定，因为要计算 $10^{8}$ 项的和的导数．

一个方便的做法是应用对冲控制变量来降低方差，可以将 Monte-Carlo 的样本数 $N$ 降为大约 $5 \times 10^{4}$ ．

假定我们有 $r$ 个对冲产品（包含价格过程 $S$ ），用 $\left(Z_{t}\right)_{t \in[0, T]}$ 表示，为 $\mathbb{Q}$ 下的平方可积鞅，在 $\mathbb{R}^{r}$ 下取值．对 $j=1, \ldots, J$ ，策略 $h_{j}:[0, T] \times \mathbb{R}^{r} \rightarrow \mathbb{R}^{r}$ 使得 $h(\cdot, Z .) \in L^{2}(Z)$ ，$c$ 为常数，定义
$$
X_{j}(\theta)(\omega):=Q_{j}(\theta)(\omega)-c\left(h_{j}\left(\cdot, Z_{.-}(\theta)(\omega)\right) \bullet Z .(\theta)(\omega)\right)_{T}\tag{1.8}
$$
则校准函数（3.8）和（3.9）可以通过替换 $Q_{j}(\theta)(\omega)$ 为 $X_{j}(\theta)(\omega)$ 来定义，变为最小化
$$
\widehat{f}(\theta)\left(\omega_{1}, \ldots, \omega_{N}\right)=\sum_{j=1}^{J} w_{j} \ell\left(\frac{1}{N} \sum_{n=1}^{N} X_{j}(\theta)\left(\omega_{n}\right)\right)\tag{1.9}
$$

-----------------------

### 算法1：模型的校准步骤

1. *\# 初始化网络参数*

2. $\quad initialize\quad \theta_{1}, \ldots, \theta_{4}$

3. *\# 定义初始模拟轨道数和初始步骤值*

4. $\quad N, \quad k=500,\quad 1$

5. *\# 定义时间离散间隔和误差容忍度*

6. $\quad \Delta_{t},\quad tol=0.01,\quad 0.001$

7. $\quad \bm{for} \quad i=1, \ldots, 4$:

8. $\qquad nextslice = False$

9. *$\qquad$\# 计算此次切片的初始正规化权重*

10. $\qquad w_{j}=\tilde{w}_{j} / \sum_{l=1}^{20} \tilde{w}_{l}$

11. $\qquad \bm{while}\quad nextslice==False$

12. $\quad \qquad do:$

13. $\qquad \qquad 模拟 N 条模型到T_i时刻的轨道，计算支付$

14. $\qquad \quad do:$

15. $\qquad \qquad 计算这些轨道对应的到 T_i 时刻的对冲的随机积分$

16. $\qquad \quad do:$

17. $\qquad \qquad 计算(1.9)式的校准函数$

18. $\qquad \quad do:$

19. $\qquad \qquad 进行一步优化\ \theta_{i}^{(k-1)}\ 到\ \theta_{i}^{(k)}$

20. $\qquad \quad do:$

21. $\qquad \qquad 利用算法\ 2\ 更新参数 N，nextslice 并计算模型价格 \pi_{model}$

22. $\qquad \quad do:$

23. $\qquad \qquad k=k+1$

### 算法2：超参的更新



-----------------------










## 附录

[定理3.1](#3_1)
<div id="proof_3_1"></div>

**证明：**

首先定理 A.2 暗示了
$$
d S_{t}(\theta)=S_{t}(\theta) L\left(t, S_{t}(\theta), \theta\right) \alpha_{t} d W_{t}
$$
$\forall \theta$ 的解存在唯一性．这里驱动过程是一维的 $Y=\int_{0}^{*} \alpha_{s} d W_{s}$．事实上，若 $(t, s) \mapsto L(t, s, \theta)$ 有界，对 $t$ 左极右连，对 $s$ Lipschitz 连续以一个与 $t$ 无关的 Lipschitz 常数．$S\mapsto S \cdot(\theta) L(\cdot, S \cdot(\theta), \theta)$ 为 functionally Lipschitz，得到结论．这些条件由 $L(t, s, \theta)$ 的形式和 $F^{i}$ 的条件保证．

为了证明导数过程的形式，我们对如下系统应用定理 A.3：
$$
d S_{t}(\widehat{\theta})=S_{t}(\widehat{\theta}) L\left(t, S_{t}(\widehat{\theta}), \widehat{\theta}\right) \alpha_{t} d W_{t}
$$
和
$$
d S_{t}(\widehat{\theta}+\varepsilon \theta)=S_{t}(\widehat{\theta}+\varepsilon \theta) L\left(t, S_{t}(\widehat{\theta}+\varepsilon \theta), \widehat{\theta}+\varepsilon \theta\right) \alpha_{t} d W_{t}
$$
以及
$$
\begin{aligned}
d \frac{S_{t}(\widehat{\theta}+\varepsilon \theta)-S_{t}(\widehat{\theta})}{\varepsilon}=& \frac{S_{t}(\widehat{\theta}+\varepsilon \theta) L\left(t, S_{t}(\widehat{\theta}+\varepsilon \theta), \widehat{\theta}+\varepsilon \theta\right)-S_{t}(\widehat{\theta}) L\left(t, S_{t}(\widehat{\theta}), \widehat{\theta}\right)}{\varepsilon} \alpha_{t} d W_{t} \\
=&\left(\frac{S_{t}(\widehat{\theta}+\varepsilon \theta)-S_{t}(\widehat{\theta})}{\varepsilon} L\left(t, S_{t}(\widehat{\theta}+\varepsilon \theta), \widehat{\theta}+\varepsilon \theta\right)\right.\\
&\left.+S_{t}(\widehat{\theta}) \frac{L\left(t, S_{t}(\widehat{\theta}+\varepsilon \theta), \widehat{\theta}+\varepsilon \theta\right)-L\left(t, S_{t}(\widehat{\theta}), \widehat{\theta}\right)}{\varepsilon}\right) \alpha_{t} d W_{t}
\end{aligned}
$$
在定理 A.3 中，$Z^{\varepsilon, 1}=S(\widehat{\theta}), Z^{\varepsilon, 2}=S(\widehat{\theta}+\varepsilon \theta), Z^{\varepsilon, 3}=\frac{S_{t}(\widehat{\theta}+\varepsilon \theta)-S_{t}(\widehat{\theta})}{\varepsilon}$．$F^{\varepsilon, 3}$ 为
$$
\begin{aligned}
F^{\varepsilon, 3}\left(Z_{t}^{0}\right)=& Z_{t}^{0,3} L\left(t, Z_{t}^{0,2}, \widehat{\theta}+\varepsilon \theta\right)+Z_{t}^{0,1} \partial_{s} L\left(t, Z_{t}^{0,1}, \widehat{\theta}\right) Z_{t}^{0,3}+\mathcal{O}(\varepsilon) \\
&+Z_{t}^{0,1} \frac{L\left(t, Z_{t}^{0,1}, \widehat{\theta}+\varepsilon \theta\right)-L\left(t, Z_{t}^{0,1}, \widehat{\theta}\right)}{\varepsilon}
\end{aligned}\tag{3.4}
$$
ucp 收敛到
$$
F^{0,3}\left(Z_{t}^{0}\right)=Z_{t}^{0,3} L\left(t, Z_{t}^{0,2}, \widehat{\theta}\right)+Z_{t}^{0,1} \partial_{s} L\left(t, Z_{t}^{0,1}, \widehat{\theta}\right) Z_{t}^{0,3}+Z_{t}^{0,1} \nabla_{\theta} L\left(t, Z_{t}^{0,1}, \widehat{\theta}\right)
$$
事实上，$\forall t$，$\{s \mapsto L(t, s, \widehat{\theta}+\varepsilon \theta), \mid \varepsilon \in[0,1]\}$ 等度连续．因此，点点收敛暗示对 $s$ 的一致连续．This together with $L(t, s, \theta)$ being piecewise constant in $t$ yields:
$$
\lim _{\varepsilon \rightarrow 0} \sup_{(t, s)}|L(t, s, \widehat{\theta}+\varepsilon \theta)-L(t, s, \widehat{\theta})|=0
$$
whence ucp convergence of the first term in (3.4). The convergence of term two is clear. The one of term three follows again from the fact that the family $\left\{s \mapsto \nabla_{\theta} L(t, s, \widehat{\theta}+\varepsilon \theta) \mid \varepsilon \in[0,1]\right\}$ is equicontinuous, which is again a consequence of the form of the neural networks.

By the assumptions on the derivatives, $F^{0,3}$ is functionally Lipschitz. Hence Theorem A.2 yields the existence of a unique solution to (3.2) and Theorem A.3 implies convergence.$\square$

[定理2.1](#2_1)
<div id="proof_2_1"></div>

Proof. Consider the extended system
$$
d\left(f^{\varepsilon} \bullet Z^{\varepsilon}\right)=\sum_{j=1}^{d} f^{\varepsilon}\left(t-, Z_{t-}^{\varepsilon}\right) F_{j}^{\varepsilon, i}\left(Z^{\varepsilon}\right)_{t-} d Y_{t}^{j}
$$
and
$$
d Z_{t}^{\varepsilon, i}=\sum_{j=1}^{d} F_{j}^{\varepsilon, i}\left(Z^{\varepsilon}\right)_{t-} d Y_{t}^{j}
$$
where we obtain existence, uniqueness and stability for the second equation by Theorem A.3, and from where we obtain ucp convergence of the integrand of the first equation: since stochastic integration is continuous with respect to the ucp topology we obtain the result.

## 文献

1. 首次在波动率校准中运用神经网络
[Hernandez A. Model calibration with neural networks[J]. Available at SSRN 2812140, 2016.](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=2812140)
2. rough波动率模型的神经网络校准
[Bayer C, Horvath B, Muguruza A, et al. On deep calibration of (rough) stochastic volatility models[J]. arXiv preprint arXiv:1908.08806, 2019.](https://arxiv.org/abs/1908.08806)和
[Horvath B, Muguruza A, Tomas M. Deep learning volatility: a deep neural network perspective on pricing and calibration in (rough) volatility models[J]. Quantitative Finance, 2021, 21(1): 11-27.
](https://arxiv.org/abs/1901.09647)
[Github 代码](https://github.com/amuguruza/RoughFCLT)
3. LSV模型GAN校准
[Cuchiero C, Khosrawi W, Teichmann J. A generative adversarial network approach to calibration of local stochastic volatility models[J]. Risks, 2020, 8(4): 101.](https://www.mdpi.com/2227-9091/8/4/101)
[Github 代码](https://github.com/wahido/neural_locVol)
4. 损失函数中不同期权权重取法
[Cont R, Ben Hamida S. Recovering volatility from option prices by evolutionary optimization[J]. 2004.](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=546882)
5. fBm的MC模拟
[Horvath B, Jacquier A J, Muguruza A. Functional central limit theorems for rough volatility[J]. Available at SSRN 3078743, 2017.](https://arxiv.org/abs/1711.03078)
[Github 代码](https://github.com/amuguruza/RoughFCLT)
6. rBergomi提出
[Bayer C, Friz P, Gatheral J. Pricing under rough volatility[J]. Quantitative Finance, 2016, 16(6): 887-904.](https://www.tandfonline.com/doi/abs/10.1080/14697688.2015.1099717)