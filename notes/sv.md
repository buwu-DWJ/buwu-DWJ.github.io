# 随机波动率摘要  

## 一． BS公式（Delta对冲下

BS部分参考[知乎专栏：Black-Scholes 模型学习框架](https://zhuanlan.zhihu.com/p/265727886)  

假设股价满足
$$
d S_{t}=\mu S_{t} d t+\sigma S_{t} d W_{t}
$$
自融资资产组合价值变化为
$$
d \Pi_{t}=r \gamma_{t} B_{t} d t+\Delta_{t} d S_{t}
$$
对 $V\left(S_{t}, t\right)$ （ $t$ 时刻衍生品的价值）做Ito公式，比较项的系数得到Delta 对冲下 的 **BS 偏微分方程**  
$$
\left\{\begin{array}{l}
V_{x}^{\prime}=\Delta_{t} \\
V_{t}^{\prime}+r S_{t} V_{x}^{\prime}+\frac{1}{2} \sigma^{2} S_{t}^{2} V_{x x}^{\prime \prime}-r V_{t}=0
\end{array}\right.
$$
终值条件（看涨期权）为
$$V\left(S_{T}, T\right)=\left(S_{T}-K\right)^{+}$$
解即为 **BS 公式**
$$
\begin{array}{l}
V\left(S_{t}, t\right)=S N\left(d_{1}\right)-K e^{-r(T-t)} N\left(d_{2}\right) \\
d_{1}=\frac{\ln \left(\frac{S_{t}}{K}\right)+\left(r+\frac{1}{2} \sigma^{2}\right)(T-t)}{\sigma \sqrt{T-t}} \\
d_{2}=d_{1}-\sigma \sqrt{T-t}
\end{array}
$$
注意到 $dS_t$ 的漂移项 $\mu$ 对期权定价没有影响．

BS model给出了期权价格的函数，作为一个波动率的函数．可以通过这个公式在给定期权价格时计算**隐含波动率**（implied volatility）．但事实是 BS 波动率强烈依赖于欧式期权的到期日和行权价．

波动率微笑是指给定到期日下，隐含波动率与行权价（maturity）的关系．  

## 二． Delta对冲

我们现在考虑用衍生品 $V(S_t,t)$ 和其标的资产 $S_t$ 构建一个“**无风险组合**”，考虑这样的自融资组合 $\Pi_{t}=\Delta_{t} S_{t}-V_{t}$，即每一单位的空头衍生品，我们用 $\Delta_{t}$ 单位多头的股票对其进行对冲 (Hedging)，由于其自融资的特性，根据定义，我们有 $d \Pi_{t}=\Delta_{t} d S_{t}-d V_{t}$ ，将股价的 SDE 和上一节中通过伊藤-德布林公式求出的 $d V_{t}$ 带入这个式子，我们可以得到:
$$
d \Pi_{t}=\left[\left(\Delta_{t}-V_{x}^{\prime}\right) \mu S_{t}-V_{t}^{\prime}-\frac{1}{2} \sigma^{2} S_{t}^{2} V_{x x}^{\prime \prime}\right] d t+\left(\Delta_{t}-V_{x}^{\prime}\right) \sigma S_{t} d W_{t}
$$
因为要使资产组合为无风险的，
>一个自融资组合 $V_{t}$ 如果是无风险的，则可以表示为 $d V_{t}=k_{t} V_{t} d t$ ，且 $k_{t} \equiv r$ ．  
$$
\left\{\begin{array}{l}
\Delta_{t}-V_{x}^{\prime}=0 \\
\left(\Delta_{t}-V_{x}^{\prime}\right) \mu S_{t}-V_{t}^{\prime}-\frac{1}{2} \sigma^{2} S_{t}^{2} V_{x x}^{\prime \prime}=r \Pi_{t}
\end{array}\right.
$$
1式即 Delta 对冲法则，将 $\Pi_{t}=V_{x}^{\prime} S_{t}-V_{t}$ 带入2式我们再次得到 BlackScholes 偏微分方程: $V_{t}^{\prime}+r S_{t} V_{x}^{\prime}+\frac{1}{2} \sigma^{2} S_{t}^{2} V_{x x}^{\prime \prime}-r V_{t}=0$ ．

## 三． BS公式（风险中性定价下）

### 3.1 鞅

**定义** 在 $(\Omega, \mathcal{F}, P)$ 上的随机过程 $X_{t}$ ， $t \in[0, T]$ ，称其是关于域流 $\mathcal{F}_{t}$ 的鞅，如果满足:

1. $X_{t}$ 是 $\mathcal{F}_{t}$ -适应的 (adapted)；
2. $E\left|X_{t}\right|<\infty, \forall t \in[0, T]$；
3. 对于 $s \leq t$ ，有 $E\left[X_{t} \mid \mathcal{F}_{s}\right]=X_{s}$．  

### 3.2 Radon-Nikodym导数  

**定义**  设 $P$ 和 $\tilde{P}$ 为 $(\Omega, \mathcal{F})$ 上的等价测度，若 $Z>0$，$E[Z]=1 ($a.s.$)$，且有 $\forall A \in \mathcal{F}， \tilde{P}(A)=\int_{A} Z(w) d P(w)$, 则称 $Z$ 是
$\tilde{P}$ 关于 $P$ 的 Radon-Nikodym 导数，记作: $Z=\frac{d \tilde{P}}{d P}$．  

$\int_{\Omega} X d \tilde{P}=\int_{\Omega} X Z d P$，即 $\tilde{E}[X]=E[Z X]$，其中 $\tilde{E}[\cdot]$ 表示在测度 $d \tilde{P}$ 下的期望．进一步的，可以用条件期望定义R-N导数过程: $Z_{t}=E\left[Z \mid \mathcal{F}_{t}\right]，t \in[0, T]$ ．用鞅和RN导数过程的定义，可以简单的证明，R-N导数过程 $Z_{t}$ 是一个 $\mathcal{F}_{t}$ -鞅．  

### 3.3 资产的现值

用 $V_t$ 表示风险资产的价值过程．首先要知道，在 $\mathrm{BS}$ 模型的假设下，市场是完备 (Complete) 的，即任意资产 $V_{t}$ 都可以被风险资产 $S_{t}$ 和无风险资产 $B_{t}$ 构成的组合所复制，即对任意一个 $V_{t}$ ，我们可以把它表示 为一个自融资组合:
$$
\begin{aligned}
d V_{t} &=\gamma_{t} d B_{t}+\Delta_{t} d S_{t} \\
&=\left(r \gamma_{t} B_{t}+\mu \Delta_{t} S_{t}\right) d t+\sigma \Delta_{t} S_{t} d W_{t} \\
&=\left[r V_{t}+(\mu-r) \Delta_{t} S_{t}\right] d t+\sigma \Delta_{t} S_{t} d W_{t}
\end{aligned}
$$
可以看到该组合的收益率部分由组合的时间价值 $r V_{t} d t$ 与风险资产的超额收益 $(\mu-r) \Delta_{t} S_{t} d t$ 构成．我们考虑该资产的折现价值过程 $V_{t} / B_{t}:$
$$
\begin{aligned}
d\left(\frac{V_{t}}{B_{t}}\right) &=\frac{1}{B_{t}} d V_{t}-\frac{V_{t}}{B_{t}^{2}} d B_{t} \\
&=(\mu-r) \Delta_{t} \frac{S_{t}}{B_{t}} d t+\sigma \Delta_{t} \frac{S_{t}}{B_{t}} d W_{t} \\
&=\sigma \Delta_{t} \frac{S_{t}}{B_{t}}\left(\frac{\mu-r}{\sigma} d t+d W_{t}\right)
\end{aligned}
$$

### 3.4 鞅表示

**定理(鞅表示)**  设 $\left\{W_{t}，\mathcal{F}_{t}^{\mathcal{W}}，t \in[0,T]\right\}$ 是 $(\Omega, \mathcal{F}, P)$ 上的布朗运动，而
$\left\{M_{t}，t \in[0, T]\right\}$ 为 $\mathcal{F}_{t}^{\mathcal{W}}$ -鞅, 且满足 $M_{t} \in \mathcal{L}^{2}(\Omega)，t \in[0, T]$ ，则存在一个 $\mathcal{F}_{t}^{\mathcal{W}}-$
适应的过程 $\left\{\Gamma_{t}, t \in[0, T]\right\} \in \mathcal{V}[0, T]$ ，使得 $M_{t}-M_{0}=\int_{0}^{t} \Gamma_{u} d W_{u}$ ．  

可以看到，如果 $M_{t}$ 是鞅，那么 $M_{t}$ 可以被表示为一个伊藤积分的形式，即没有 $d t$ 项而仅仅只有 $d W_{t}$ 项．再看我们的折现价值过程 $d\left(\frac{V_{t}}{B_{t}}\right)=\sigma \Delta_{t} \frac{S_{t}}{B_{t}}\left(\frac{\mu-r}{\sigma} d t+d W_{t}\right)$ ，如果想让它只有
$d W_{t}$ 项从而变成一个鞅，我们貌似只需要做变换: $d \tilde{W}_{t}=\frac{\mu-r}{\sigma} d t+d W_{t}$ ，这样折现价值过程就可以被表示为:
$d\left(\frac{V_{t}}{B_{t}}\right)=\sigma \Delta_{t} \frac{S_{t}}{B_{t}} d \tilde{W}_{t}$
但是，鞅表示定理有个非常非常重要的前提，就是你需要保证 $\int_{0}^{t} \Gamma_{u} d W_{u}$ 这玩意儿是个伊藤积分，即 $W_{t}$ 需要是一个布朗运动． $W_{t}$ 我们知道是布朗运动，但是经过这样变换过后的 $\tilde{W}_{t}$ 还是布朗运动么，或者说我们需要如何选择新的测度, 来保证经过变换之后的 $\tilde{W}_{t}$ 仍然是个布朗运动?

### 3.5 Girsanov定理

**定理(Grisanov)** 设 $\left\{W_{t}，\mathcal{F}_{t}^{\mathcal{W}}，t \in[0, T]\right\}$ 是 $(\Omega, \mathcal{F}, P)$ 上的布朗运动
$\left\{\theta_{t}, t \in[0, T]\right\}$ 为一个相适应的过程, 定义指数鞅过程，
$Z_{t}=\exp \left(X_{t}-\frac{1}{2}[X, X]_{t}\right)$
其中 $X_{t}$ 是初值 $X_{0}=0$ 的相适应的过程，$[\cdot, \cdot]$ 表示二次变差．则可以定义新的概率测度 $\left.\frac{d \tilde{P}}{d P}\right|_{\mathcal{F}_{t}}=Z_{t}$ ．如果在概率测度 $P$ 下 $W_{t}$ 是一个布朗运动，那么:
$\tilde{W}_{t}=W_{t}-[W, X]_{t}$
在新的概率测度 $\tilde{P}$ 下也是一个布朗运动．  

这样一来, 我们就找到了新的测度和两个测度之下布朗运动之间的关系．我们看新定义的这个布朗运动：$d \tilde{W}_{t}=\frac{\mu-r}{\sigma} d t+d W_{t}$，它的实质是把资产的风险溢价项给消除了．风险溢价是什么？是对承担单位风险的补偿，在新的测度下风险溢价是没有补偿的，所以说在这个世界里，风险是中性的，因此我们把这样定义的新测度 $\tilde{P}$ 称为**风险中性测度**，并且用 $Q$ 来表示．

### 3.6 风险定价公式

现在我们知道了变换公式 $d \tilde{W}_{t}=\frac{\mu-r}{\sigma} d t+d W_{t}$，那么在风险中性测度 $Q$ 下，风险资产 $S_{t}$ 所满足的 SDE 也需要进行相应的变化:
$$
\begin{aligned}
d S_{t} &=\mu S_{t} d t+\sigma S_{t}\left(d W_{t}^{Q}-\frac{\mu-r}{\sigma} d t\right) \\
&=r S_{t} d t+\sigma S_{t} d W_{t}^{Q}
\end{aligned}
$$
由此可见，在风险中性世界里，风险资产 (例如股票) 的收益率完全等于无风险收益率．  

此时任意资产的折现价值过程可以被表示为:
$d\left(\frac{V_{t}}{B_{t}}\right)=\sigma \Delta_{t} \frac{S_{t}}{B_{t}} d W_{t}^{Q}$
．我们知道 $\frac{V_{t}}{B_{t}}$ 在 $Q$ 下是一个鞅，那么由鞅的性贡我们可以知道:
$\frac{V_{t}}{B_{t}}=E^{Q}\left[\frac{V_{T}}{B_{T}} \mid \mathcal{F}_{t}\right]$ ．
常利率假设下有：
$V_{t}=e^{-r(T-t)} E^{Q}\left[V_{T} \mid \mathcal{F}_{t}\right]$ ．  

假设我们需要对一个欧式看涨期权进行定价，我们知道该期权在到期日 $T$ 的价值为 $\left(S_{T}-K\right)^{+}$，则有：
$$
\begin{aligned}
V_{t} &=e^{-r(T-t)} E^{Q}\left[\left(S_{T}-K\right) \cdot \mathbf{1}_{\left\{S_{T} \geq K\right\}} \mid \mathcal{F}_{t}\right] \\
&=S_{t} Q\left\{x \geq-d_{2}-\sigma \sqrt{T-t}\right\}-K e^{-r(T-t)} Q\left\{x \geq-d_{2}\right\} \\
&=S_{t} N\left(d_{1}\right)-K e^{-r(T-t)} N\left(d_{2}\right)
\end{aligned}
$$
其中:
$$
\begin{array}{l}
d_{1}=\frac{\ln \left(\frac{S_{t}}{K}\right)+\left(r+\frac{1}{2} \sigma^{2}\right)(T-t)}{\sigma \sqrt{T-t}} \\
d_{2}=d_{1}-\sigma \sqrt{T-t}
\end{array}
$$
与PDE方法一致．  

我们来总结一下 Risk-neutral Pricing 的几个步骤：

1. 找到资产的折现价值过程；
2. 作测度变换令这个折现价值在新的测度下为鞅；
3. 用 Girsanov 定理找到新的变换；
4. 利用鞅性质得到风险中性定价公式。

## 四． 局部波动率  

### 4.1 Dupire 的工作1994

**局部波动率**基于如下：
$$
\frac{d S_{t}}{S_{t}}=\mu_{t} d t+\sigma\left(S_{t}, t ; S_{0}\right) d W_{t}
$$
其中， $\sigma\left(S_{t}, t ; S_{0}\right)$ 是 $S_t$ 和 $t$ 的确定性函数， $S_0$ 是固定的参数．在风险中性测度时， $\mu_{t}=r_{t}-q_{t}$ 下． 一旦 $\sigma\left(S_{t}, t ; S_{0}\right)$ 给定，那么模型也就定了．  

Dupire（1994）证明了当给定了关于 $K$（行权价） 和 $T$（到期日） 的期权价格函数 $C\left(S_{0}, 0, K, T\right)$ 时，局部波动率 $\sigma\left(S_{t}, t ; S_{0}\right)$ 是唯一确定的．  

令 $\phi\left(S_{t}, t ; Y, T\right)$ 是如下定义的 $S_t$ 的转移密度函数（transitional density function）：
>转移密度函数：$\mathbb{P}\left(a<y^{\prime}<b\right.$ at $t^{\prime} \mid y$ at $\left.t\right)=\int_{a}^{b} p\left(y, t ; y^{\prime}, t^{\prime}\right) d y^{\prime}$ ，指在 $t$ 时刻 $y$ 条件下在 $t^{\prime}$ 时刻时 $y^{\prime}$ 的分布
$$
\begin{aligned}
\phi\left(S_{t}, t ; Y, T\right) &=E_{t}^{Q}\left[\phi\left(S_{T}, T ; Y, T\right)\right] \\
&=E_{t}^{Q}\left[\delta\left(S_{T}-Y\right)\right]
\end{aligned}
$$
其中 $Q$ 代表风险中性测度．众所周知 $\phi\left(S_{t}, t ; Y, T\right)$ 满足倒向的 Fokker-Planck 方程：
$$
\left\{\begin{array}{c}
\left(\frac{\partial}{\partial t}+\mathcal{L}_{t, S}\right) \phi\left(S_{t}, t ; Y, T\right)=0 \\
\phi\left(S_{T}, T ; Y, T\right)=\delta\left(S_{T}-Y\right)
\end{array}\right.
$$
其中
$$
\mathcal{L}_{t, S}=\frac{1}{2} \sigma_{t}^{2} S^{2} \frac{\partial^{2}}{\partial S^{2}}+\mu_{t} S \frac{\partial}{\partial S}
$$
一维情形，Fokker-Planck 方程有两个参数，一是拓扑参数 $D_{1}(x, t)$，另一是扩散 $D_{2}(x, t)$
$$
\frac{\partial}{\partial t} f(x, t)=-\frac{\partial}{\partial x}\left[D_{1}(x, t) f(x, t)\right]+\frac{\partial^{2}}{\partial x^{2}}\left[D_{2}(x, t) f(x, t)\right] .
$$

又可证它也满足前向的 Fokker-Planck 方程
$$
\left(\frac{\partial}{\partial T}-\mathcal{L}_{T, Y}^{*}\right) \phi\left(S_{t}, t ; Y, T\right)=0, \quad \forall T>t
$$
其中
$$
\mathcal{L}_{T, Y}^{*} \phi\left(S_{t}, t ; Y, T\right)=\frac{1}{2} \frac{\partial^{2}}{\partial Y^{2}}\left(\sigma_{T}^{2} Y^{2} \phi\right)-\frac{\partial}{\partial Y}\left(\mu_{T} Y \phi\right)
$$
且
$$
\phi\left(S_{t}, t ; Y, T\right)=\delta(Y-S)
$$
下面可推导 Dupire 方程，看涨期权的价格满足
$$
C\left(S_{0}, K, T\right)=e^{-r T} \int_{K}^{\infty} \phi\left(S_{0}, 0 ; S_{T}, T\right)\left(S_{T}-K\right) d S_{T}\tag{4.1}
$$
其中 $\phi\left(S_{0}, 0 ; S_{T}, T\right)$ 是 $S_T$ 的风险中性测度．对（4.1）关于 $K$ 做一次和二次微分，有
$$
\begin{aligned}
\frac{\partial C}{\partial K}&=-e^{-r T} \int_{K}^{\infty} \phi\left(S_{0}, 0 ; S_{T}, T\right) d S_{T} \\
\frac{\partial^{2} C}{\partial K^{2}}&=e^{-r T} \phi\left(S_{0}, 0 ; K, T\right)
\end{aligned}
$$

已知 $\phi$ 满足前向 Fokker-Planck 方程
$$
\frac{\partial \phi}{\partial T}=\frac{1}{2} \frac{\partial^{2}}{\partial S_{T}^{2}}\left(\sigma_{T}^{2} S_{T}^{2} \phi\right)-\frac{\partial}{\partial S_{T}}\left(\mu_{T} S_{T} \phi\right)
$$
（4.1）对 $T$ 微分，有
$$
\begin{aligned}
\frac{\partial C\left(S_{0}, K, T\right)}{\partial T}=& e^{-r T} \int_{K}^{\infty} d S_{T}\left\{\frac{\partial \phi\left(S_{0}, 0 ; S_{T}, T\right)}{\partial T}\right\}\left(S_{T}-K\right)-r C \\
=& e^{-r T} \int_{K}^{\infty} d S_{T}\left\{\frac{1}{2} \frac{\partial^{2}}{\partial S_{T}^{2}}\left(\sigma_{T}^{2} S_{T}^{2} \phi\right)-\frac{\partial}{\partial S_{T}}\left(\mu_{T} S_{T} \phi\right)\right\}\left(S_{T}-K\right)-r C \\
=&\left.e^{-r T}\left\{\frac{1}{2} \frac{\partial}{\partial S_{T}}\left(\sigma_{T}^{2} S_{T}^{2} \phi\right)-\frac{\partial}{\partial S_{T}}\left(\mu_{T} S_{T} \phi\right)\right\}\left(S_{T}-K\right)\right|_{S_{T}=K} ^{+\infty} \\
&-e^{-r T} \int_{K}^{\infty} d S_{T}\left\{\frac{1}{2} \frac{\partial}{\partial S_{T}}\left(\sigma_{T}^{2} S_{T}^{2} \phi\right)-\left(\mu_{T} S_{T} \phi\right)\right\}-r C \\
=&-\left.e^{-r T} \frac{1}{2}\left(\sigma_{T}^{2} S_{T}^{2} \phi\right)\right|_{S_{T}=K} ^{+\infty}+e^{-r T} \int_{K}^{\infty} d S_{T}\left(\mu_{T} S_{T} \phi\right)-r C \\
=& e^{-r T} \frac{1}{2} \sigma_{T}^{2} K^{2} \phi\left(S_{0}, 0 ; K, T\right)+\mu_{T} e^{-r T} \int_{K}^{\infty} d S_{T} S_{T} \phi-r C \\
=& \frac{1}{2} \sigma^{2} K^{2} \frac{\partial^{2} C}{\partial K^{2}}+e^{-r T} \mu_{T} \int_{K}^{\infty} d S_{T}\left(S_{T}-K+K\right) \phi-r C \\
=& \frac{1}{2} \sigma^{2} K^{2} \frac{\partial^{2} C}{\partial K^{2}}+\mu_{T}\left\{C+e^{-r T} K \int_{K}^{\infty} d S_{T} \phi\right\}-r C \\
=& \frac{1}{2} \sigma^{2} K^{2} \frac{\partial^{2} C}{\partial K^{2}}+\mu_{T}\left\{C-K \frac{\partial C}{\partial K}\right\}-r C
\end{aligned}\tag{0.3}
$$
这里我们假设 $\mu_T$ 与 $S_T$ 无关，故我们得到了 Dupire 方程：
$$
\frac{\partial C}{\partial T}=\frac{1}{2} \sigma_{T}^{2} K^{2} \frac{\partial^{2} C}{\partial K^{2}}+\mu_{T}\left\{C-K \frac{\partial C}{\partial K}\right\}-r C\tag{0.4}
$$
Dupire 方程最大的优点是**把局部波动率函数用期权价格和他们的微分表示出来**了
$$
\sigma^{2}\left(K, T, S_{0}\right)=\frac{\frac{\partial C}{\partial T}-\mu_{T}\left\{C-K \frac{\partial C}{\partial K}\right\}+r C}{\frac{1}{2} K^{2} \frac{\partial^{2} C}{\partial K^{2}}}
$$
以上结果扩展到了时间依赖的利率，我们只要将 $r$ 替换为 $r_T$ 即可．

#### 局限性

1. 可用的观测值是很少的
2. 数值微分不可靠，二阶的更甚

确定方程的传统做法是二元样条插值，再对模型进行校准（calibration）．  

#### 局部波动率作为瞬时方差的条件期望

考虑如下形式的一般的随机波动率模型
$$
\left\{\begin{array}{l}
d S_{t}=S_{t}\left(\mu_{t} d t+\sigma_{t} d W_{t}\right) \\
d \sigma_{t}=\alpha\left(S_{t}, \sigma_{t}, t\right) d t+\beta\left(S_{t}, \sigma_{t}, t\right) d Z_{t}
\end{array}\right.
$$
其中
$$
d W_{t} d Z_{t}=\rho_{t} d t
$$
远期价格
$$
F_{t}^{T}=S_{t} e^{\int_{0}^{t}\left(r_{s}-q_{*}\right) d s}
$$
资产价格过程变为
$$
d F_{t}^{T}=F_{t}^{T} \sigma_{t} d W_{t}
$$
考虑到 $F_{T}^{T}=S_{T}$ ， 欧式看涨期权的 $T$ -forward 价格为
$$
C\left(F_{0}^{T}, K, T\right)=E\left[\left(F_{T}^{T}-K\right)^{+}\right]
$$
上式两端对 $K$ 求微分
$$
\begin{array}{l}
\frac{\partial C}{\partial K}=E\left[H\left(F_{T}^{T}-K\right)\right] \\
\frac{\partial^{2} C}{\partial K^{2}}=E\left[\delta\left(F_{T}^{T}-K\right)\right]
\end{array}
$$
>其中 $H$ 是 Heaviside函数， $H(t)=\left\{\begin{array}{ll}0， & t<0， \\ 1, & t \geq 0．\end{array}\right.$ $\delta$ 是Dirac函数， $\delta(x)=0，(x \neq 0)$ ， $\int_{-\infty}^{\infty} \delta(x) d x=1$ ．

对最终的支付应用 Ito 公式并令 $t \rightarrow T$ 有
$$
d\left(F_{T}^{T}-K\right)^{+}=H\left(F_{T}^{T}-K\right) d F_{T}^{T}+\frac{1}{2} \sigma_{T}^{2}\left(F_{T}^{T}\right)^{2} \delta\left(F_{T}^{T}-K\right) d T
$$
两侧取条件期望 $F_{0}^{T}$，有
$$
\begin{aligned}
d C &=d E\left[\left(F_{T}^{T}-K\right)^{+}\right] \\
&=\frac{1}{2} E\left[\sigma_{T}^{2}\left(F_{T}^{T}\right)^{2} \delta\left(F_{T}^{T}-K\right)\right] d T
\end{aligned}
$$
上式用到了 $F_{t}^{T}$ 鞅的性质．接下来
$$
\begin{aligned}
E\left[\sigma_{T}^{2}\left(F_{T}^{T}\right) \delta\left(F_{T}^{T}-K\right)\right] &=E\left[\sigma_{T}^{2}\left(F_{T}^{T}\right) \mid F_{T}^{T}=K\right] E\left[\delta\left(F_{T}^{T}-K\right)\right] \\
&=K^{2} E\left[\sigma_{T}^{2} \mid F_{T}^{T}=K\right] \frac{\partial^{2} C}{\partial K^{2}}
\end{aligned}
$$
第二个等式是条件期望公式的一个推广：
$$
E[X \mid A]=\frac{E\left[X 1_{A}\right]}{E\left[1_{A}\right]}
$$
或者
$$
\left.E\left[X 1_{A}\right]=E[X \mid A] E\left[1_{A}\right]\right]
$$
因为
$$
d C=\frac{\partial C}{\partial T} d T
$$
故有
$$
\frac{\partial C}{\partial T}=\frac{1}{2} K^{2} E\left[\sigma_{T}^{2} \mid F_{T}^{T}=K\right] \frac{\partial^{2} C}{\partial K^{2}}
$$
进行比较，可知对应的局部波动率模型为
$$
\begin{aligned}
\sigma_{l o c}^{2}\left(K, T, S_{0}\right) &=E\left[\sigma_{T}^{2} \mid F_{T}^{T}=K\right] \\
&=E\left[\sigma_{T}^{2} \mid F_{0}^{T} e^{\int_{0}^{T}-\frac{1}{2} \sigma_{t}^{2} d t+\sigma_{t} d W_{t}}=K\right]
\end{aligned}
$$
也就是说，局部方差是以最终股票价格 $S_T=F_T^T$ 等于行权价 $K$ 为条件的瞬时方差的风险中性期望．

## 五． 随机局部波动率模型

### 5.1 Jex的随机局部波动率模型1999

$$
\begin{aligned}
\frac{d S}{S}&=\left(r_{d}-r_{f}\right) d t+\sqrt{X(S, t) Y} d Z_{1} \\
d Y&=\alpha\left(Y_{*}-Y\right) d t+\xi \sqrt{Y} d Z_{2} \\
X(K, T)&=\frac{\sigma_{D}^{2}(K, T)}{\mathrm{E}\left[Y^{2}(T) \mid S(T)=K\right]}
\end{aligned}
$$
其中
$r_d$ 和 $r_f$ 应该是无风险利率减股息收益，$Y$ 是随机波动率部分，$Y_*$ 是波动率的均值回复的平衡点．
>Heston随机波动率模型：$d S(t)=\mu S d t+\sqrt{v(t)} S d z_{1}(t)$，$d \sqrt{v(t)}=-\beta \sqrt{v(t)} d t+\delta d z_{2}(t)$．或者（$d v(t)=\left[\delta^{2}-2 \beta v(t)\right] d t+2 \delta \sqrt{v(t)} d z_{2}(t)$）  

注意到如果没有 $\sqrt{X(S, t)}$ 这一项，模型即为Heston模型，而当 $\xi$ 为 $0$ 时模型即为Dupire．

### 5.2 GAN基于LOCAL_STOCHASTIC_VOLATILITY

>This means parameterizing the model pool in a way which is accessible for machine learning techniques and interpreting the inverse problem as a training task of a generative network, whose quality is assessed by anadversary.We pursue this approach in the presentarticle  and  use  as  generative  models  so-called  neural  stochastic  differential  equations  (SDE),which just means to parameterize the drift and volatility of an Itˆo-SDE by neural networks.  

文中指的**neural SDE**即通过神经网络来对Ito-SDE的漂移项和波动率进行参数化．  
这里考虑的某资产的折现后价格过程（discounted price process） $(S_t)_{t\geq 0}$ ：
$$
dS_t = S_t L(t,S_t)\alpha_t dW_t
$$
其中 $(\alpha_t)_{t\geq 0}$ 是某个 $\mathbb{R}$ 中取值的随机过程，$L(t,s)$ 称为**杠杆函数**（Leverage function）取决于 $t$ 和资产当前价格．  
$L$ 的选取非常重要，需要很好地校准市场上观测到的隐含波动率．故 $L$ 需要满足如下条件：
$$
L^{2}(t, s)=\frac{\sigma_{\text {Dup }}^{2}(t, s)}{\mathbb{E}\left[\alpha_{t}^{2} \mid S_{t}=s\right]}\tag{1.1} ,
$$
其中 $\sigma_{\text {Dup }}$ 指 **Dupire** 的local volatility function．注意到（1.1）是 $L$ 的隐式方程，因为 $\mathbb{E}\left[\alpha_{t}^{2} \mid S_{t}=s\right]$ 中需要 $(S_t,\alpha_t)$ ．故此时 $(S_t)_{t\geq 0}$ 满足的SDE也成为了一个**McKean-Vlasov SDE**．  

本文采用了 an alternative，fully data-driven 方法，规避了其他计算 Dupire 局部波动率的方法中必须的对波动率曲面插值的做法，即此方法只需**离散数据**．  

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

对固定的 $\gamma$ ， $\ell^{\gamma}$ 是非线性非负凸函数满足 $\ell^{\gamma}(0) = 0$ 且 $\ell^{\gamma}(x)>0$ 对 $x\neq 0$ ，衡量模型和市场价的距离．$w_{j}^{\gamma}$ 某种权重，本文采用vega type．参数 $\gamma$ 扮演了**对抗**（adversarial）的部分，注意到 $\ell^{\gamma}$ 和 $w_{j}^{\gamma}$ 都受 $\gamma$ 控制．  
