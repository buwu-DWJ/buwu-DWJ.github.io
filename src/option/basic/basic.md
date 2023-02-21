## 一．基础概念

### 1.1 BS公式（Delta对冲下）

BS部分参考[知乎专栏：Black-Scholes 模型学习框架](https://zhuanlan.zhihu.com/p/265727886)

假设股价满足
$$
d S_{t}=\mu S_{t} d t+\sigma S_{t} d W_{t}
$$
自融资资产组合（此处为期权价格的一个复制）价值变化为
$$
d \Pi_{t}=r \gamma_{t} B_{t} d t+\Delta_{t} d S_{t}
$$
对 $V\left(S_{t}, t\right)$ （ $t$ 时刻衍生品的价值）做Ito公式，比较项的系数得到Delta 对冲下 的 **BS 偏微分方程**  （在比较系数过程中会使用到$\gamma_{t} B_{t}$替换为$V_{t}-V_{x}^{\prime} S_{t}$，即为持有的债券份额为自融资总份额减去持有的标的份额，标的的持有量已经使用为$\Delta_t$）
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

#### BS公式解法（欧式call）

$$
\frac{\partial C}{\partial t}+\frac{1}{2} \sigma^{2} S^{2} \frac{\partial^{2} C}{\partial S^{2}}=r C-r S \frac{\partial C}{\partial S}
$$

有考虑边界条件
$$
\begin{aligned}
C(0, t) &=0 \text { for all } t \\
C(S, t) & \rightarrow S \text { as } S \rightarrow \infty \\
C(S, T) &=\max \{S-K, 0\}
\end{aligned}
$$

注意到这是一个 Cauchy-Euler 方程，能通过下述变量代换将其转化为一个扩散方程
The solution of the PDE gives the value of the option at any earlier time, $\mathbb{E}[\max \{S-K, 0\}] .$
$$
\begin{aligned}
&\tau=T-t \\
&u=C e^{r \tau} \\
&x=\ln \left(\frac{S}{K}\right)+\left(r-\frac{1}{2} \sigma^{2}\right) \tau
\end{aligned}
$$
Black-Scholes PDE 变为一个扩散方程：
$$
\frac{\partial u}{\partial \tau}=\frac{1}{2} \sigma^{2} \frac{\partial^{2} u}{\partial x^{2}}
$$
终值条件 $C(S, T)=\max \{S-K, 0\}$ 现在变为了初值条件
$$
u(x, 0)=u_{0}(x):=K\left(e^{\max \{x, 0\}}-1\right)=K\left(e^{x}-1\right) H(x)
$$
其中 $H(x)$ 是 Heaviside 阶梯函数，

使用解给定了初值函数 $u(x, 0)$ 的扩散方程的标准卷积法，得到
$$
u(x, \tau)=\frac{1}{\sigma \sqrt{2 \pi \tau}} \int_{-\infty}^{\infty} u_{0}(y) \exp \left[-\frac{(x-y)^{2}}{2 \sigma^{2} \tau}\right] d y
$$
经过处理，得到：
$$
u(x, \tau)=K e^{x+\frac{1}{2} \sigma^{2} \tau} N\left(d_{1}\right)-K N\left(d_{2}\right)
$$
其中 N 为正态分布累计密度函数，
$$
\begin{aligned}
&d_{1}=\frac{1}{\sigma \sqrt{\tau}}\left[\left(x+\frac{1}{2} \sigma^{2} \tau\right)+\frac{1}{2} \sigma^{2} \tau\right] \\
&d_{2}=\frac{1}{\sigma \sqrt{\tau}}\left[\left(x+\frac{1}{2} \sigma^{2} \tau\right)-\frac{1}{2} \sigma^{2} \tau\right]
\end{aligned}
$$

BS model给出了期权价格的函数，作为一个波动率的函数．可以通过这个公式在给定期权价格时计算**隐含波动率**（implied volatility）．但事实是 BS 波动率强烈依赖于欧式期权的到期日和行权价．

波动率微笑是指给定到期日下，隐含波动率与行权价（maturity）的关系．

### 1.2 Delta对冲

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

### 1.3 BS公式（风险中性定价下）

#### 1.3.1 鞅

**定义** 在 $(\Omega, \mathcal{F}, P)$ 上的随机过程 $X_{t}$ ， $t \in[0, T]$ ，称其是关于域流 $\mathcal{F}_{t}$ 的鞅，如果满足:

1. $X_{t}$ 是 $\mathcal{F}_{t}$ -适应的 (adapted)；
2. $E\left|X_{t}\right|<\infty, \forall t \in[0, T]$；
3. 对于 $s \leq t$ ，有 $E\left[X_{t} \mid \mathcal{F}_{s}\right]=X_{s}$．

#### 1.3.2 Radon-Nikodym导数

**定义**  设 $P$ 和 $\tilde{P}$ 为 $(\Omega, \mathcal{F})$ 上的等价测度，若 $Z>0$，$E[Z]=1 ($a.s.$)$，且有 $\forall A \in \mathcal{F}， \tilde{P}(A)=\int_{A} Z(w) d P(w)$, 则称 $Z$ 是
$\tilde{P}$ 关于 $P$ 的 Radon-Nikodym 导数，记作: $Z=\frac{d \tilde{P}}{d P}$．

$\int_{\Omega} X d \tilde{P}=\int_{\Omega} X Z d P$，即 $\tilde{E}[X]=E[Z X]$，其中 $\tilde{E}[\cdot]$ 表示在测度 $d \tilde{P}$ 下的期望．进一步的，可以用条件期望定义R-N导数过程: $Z_{t}=E\left[Z \mid \mathcal{F}_{t}\right]，t \in[0, T]$ ．用鞅和RN导数过程的定义，可以简单的证明，R-N导数过程 $Z_{t}$ 是一个 $\mathcal{F}_{t}$ -鞅．

#### 1.3.3 资产的现值

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

#### 1.3.4 鞅表示

**定理(鞅表示)**  设 $\left\{W_{t}，\mathcal{F}_{t}^{\mathcal{W}}，t \in[0,T]\right\}$ 是 $(\Omega, \mathcal{F}, P)$ 上的布朗运动，而 $\left\{M_{t}，t \in[0, T]\right\}$ 为 $\mathcal{F}_{t}^{\mathcal{W}}$ -鞅, 且满足 $M_{t} \in \mathcal{L}^{2}(\Omega)，t \in[0, T]$ ，则存在一个 $\mathcal{F}_{t}^{\mathcal{W}}-$ 适应的过程 $\left\{\Gamma_{t}, t \in[0, T]\right\} \in \mathcal{V}[0, T]$ ，使得 $M_{t}-M_{0}=\int_{0}^{t} \Gamma_{u} d W_{u}$ ．

可以看到，如果 $M_{t}$ 是鞅，那么 $M_{t}$ 可以被表示为一个伊藤积分的形式，即没有 $d t$ 项而仅仅只有 $d W_{t}$ 项．再看我们的折现价值过程 $d\left(\frac{V_{t}}{B_{t}}\right)=\sigma \Delta_{t} \frac{S_{t}}{B_{t}}\left(\frac{\mu-r}{\sigma} d t+d W_{t}\right)$ ，如果想让它只有 $d W_{t}$ 项从而变成一个鞅，我们貌似只需要做变换: $d \tilde{W}_{t}=\frac{\mu-r}{\sigma} d t+d W_{t}$ ，这样折现价值过程就可以被表示为: $d\left(\frac{V_{t}}{B_{t}}\right)=\sigma \Delta_{t} \frac{S_{t}}{B_{t}} d \tilde{W}_{t}$

但是，鞅表示定理有个非常非常重要的前提，就是你需要保证 $\int_{0}^{t} \Gamma_{u} d W_{u}$ 这玩意儿是个伊藤积分，即 $W_{t}$ 需要是一个布朗运动． $W_{t}$ 我们知道是布朗运动，但是经过这样变换过后的 $\tilde{W}_{t}$ 还是布朗运动么，或者说我们需要如何选择新的测度, 来保证经过变换之后的 $\tilde{W}_{t}$ 仍然是个布朗运动?

#### 1.3.5 Girsanov定理

**定理(Grisanov)** 设 $\left\{W_{t}，\mathcal{F}_{t}^{\mathcal{W}}，t \in[0, T]\right\}$ 是 $(\Omega, \mathcal{F}, P)$ 上的布朗运动．$\left\{\theta_{t}, t \in[0, T]\right\}$ 为一个相适应的过程, 定义指数鞅过程，$Z_{t}=\exp \left(X_{t}-\frac{1}{2}[X, X]_{t}\right)$．其中 $X_{t}$ 是初值 $X_{0}=0$ 的相适应的过程，$[\cdot, \cdot]$ 表示二次变差．则可以定义新的概率测度 $\left.\frac{d \tilde{P}}{d P}\right|_{\mathcal{F}_{t}}=Z_{t}$ ．如果在概率测度 $P$ 下 $W_{t}$ 是一个布朗运动，那么: $\tilde{W}_{t}=W_{t}-[W, X]_{t}$ 在新的概率测度 $\tilde{P}$ 下也是一个布朗运动．

这样一来, 我们就找到了新的测度和两个测度之下布朗运动之间的关系．我们看新定义的这个布朗运动：$d \tilde{W}_{t}=\frac{\mu-r}{\sigma} d t+d W_{t}$，它的实质是把资产的风险溢价项给消除了．风险溢价是什么？是对承担单位风险的补偿，在新的测度下风险溢价是没有补偿的，所以说在这个世界里，风险是中性的，因此我们把这样定义的新测度 $\tilde{P}$ 称为**风险中性测度**，并且用 $Q$ 来表示．

#### 1.3.6 风险定价公式

现在我们知道了变换公式 $d \tilde{W}_{t}=\frac{\mu-r}{\sigma} d t+d W_{t}$，那么在风险中性测度 $Q$ 下，风险资产 $S_{t}$ 所满足的 SDE 也需要进行相应的变化:
$$
\begin{aligned}
d S_{t} &=\mu S_{t} d t+\sigma S_{t}\left(d W_{t}^{Q}-\frac{\mu-r}{\sigma} d t\right) \\
&=r S_{t} d t+\sigma S_{t} d W_{t}^{Q}
\end{aligned}
$$
由此可见，在风险中性世界里，风险资产 (例如股票) 的收益率完全等于无风险收益率．

此时任意资产的折现价值过程可以被表示为: $d\left(\frac{V_{t}}{B_{t}}\right)=\sigma \Delta_{t} \frac{S_{t}}{B_{t}} d W_{t}^{Q}$．我们知道 $\frac{V_{t}}{B_{t}}$ 在 $Q$ 下是一个鞅，那么由鞅的性贡我们可以知道: $\frac{V_{t}}{B_{t}}=E^{Q}\left[\frac{V_{T}}{B_{T}} \mid \mathcal{F}_{t}\right]$．常利率假设下有：$V_{t}=e^{-r(T-t)} E^{Q}\left[V_{T} \mid \mathcal{F}_{t}\right]$ ．

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