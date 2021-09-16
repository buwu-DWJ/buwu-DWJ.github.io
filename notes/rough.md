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
所以，由 Wick expenential
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

模型 7 是 non-Markovian 因为 $v_{t}: \mathbb{E}^{\mathbb{Q}}\left[v_{u} \mid \mathcal{F}_{t}\right] \neq$ $\mathbb{E}^{\mathbb{Q}}\left[v_{u} \mid v_{t}\right]$ ，但对于无限维的状态空间是 Markovian 的 $\mathbb{E}^{\mathbb{Q}}\left[v_{u} \mid \mathcal{F}_{t}\right]=\xi_{t}(u)$．

## on deep calibration of rough sv model

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

