# RL 在金融中的应用

参见[Modern Perspectivs on RL in Finance](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=3449401)和[RL in economics and finance 2021](https://link.springer.com/article/10.1007/s10614-021-10119-4)．

本来应该通过动态规划方法解这些问题．用动态规划解优化问题通常需要下述三个条件：

1. 明确知道模型的状态转移概率
2. 有足够的算力来求解DP
3. Markov性质

RL，结合了DP，蒙特卡洛模拟，函数近似和机器学习．

RL在金融中主要有以下三个应用方向：

1. 衍生品定价/对冲
2. 投资组合/资产配置
3. 做市

## 一． RL for Risk Management

通常而言，学术中对衍生品定价和对冲都是基于随机环境下的有模型决策（model-driven decision rules in a stochastic environment），常规对冲策略都会用到希腊值 Greeks，代表模型对不同参数风险定价的敏感程度．

这种方法在高维情况时通常缺少有效的数值模拟方法．

### Deep Hedging

参考[Deep Hedging, Buehler et al.](https://arxiv.org/abs/1802.03042)

>**市场摩擦**(market frictions)：指金融资产在交易中存在的难度，如手续费(transaction costs)、买卖价差(bid/ask spread)、流动性约束(liquidity constraints)等．

把 trading decision 建模成一个网络，特征不仅仅有价格，还有交易信号，新闻分析(news analytics)，过去对冲决策等等．

算法是完全 model-free，不依赖对应市场的动力．我们只需确定下来市场的状态生成(scenario generator)，损失函数，市场摩擦和交易行为(trading instruments)．所以此方法 lends itself to a **statistically driven market dynamics**，我们不需要像传统方法那样计算单个衍生品的希腊值，应将建模的精力花在实现真实的市场动力和样本外表现．

#### 建模：带市场摩擦的离散市场

考虑有限时域的离散金融市场 $T$ 和交易时刻 $0=t_{0}<t_{1}<\ldots<t_{n}=T$. 固定一个有限概率空间 $\Omega=\left\{\omega_{1}, \ldots, \omega_{N}\right\}$ 和一个概率测度 $\mathbb{P}$ s.t. $\mathbb{P}\left[\left\{\omega_{i}\right\}\right]>0$ 对所有的 $i$. 定义所有 $\Omega$ 上的实值随机变量 $\mathcal{X}:=\{X: \Omega \rightarrow$ $\mathbb{R}\}$．

记 $I_{k}\in \mathbb{R}^{r}$ 在 $t_{k}$ available 的新市场数据， including market costs and mid-prices of liquid instruments-typically quoted in auxiliary terms such as implied volatilities-news, balance sheet information, any trading signals, risk limits etc. 过程 $I=\left(I_{k}\right)_{k=0, \ldots, n}$ 生成域流 $\mathbb{F}=\left(\mathcal{F}_{k}\right)_{k=0, \ldots, n}$, i.e. $\mathcal{F}_{k}$ 表示到 $t_{k}$ 时刻所有可用的信息. 注意到每个 $\mathcal{F}_{k}$ 可测的随机变量可以写成 $I_{0}, \ldots, I_{k}$的函数．

市场有 $d$ 个**hedging instruments** with mid-prices 取值于 $\mathbb{R}^{d}$ -valued $\mathbb{F}$ -adapted 随机过程 **$S=\left(S_{k}\right)_{k=0, \ldots, n}$**. 即可以用来做对冲的资产．

衍生品的投资组合即**负债**(liability)是一个 $\mathcal{F}_{T}$ 可测的随机变量 **$Z$**. 到期日 $T$ 是所有衍生品种最大的那一个．此为想要对冲掉的东西．

即 用$S 对冲 Z$．

想要在 $T$ 对冲掉 $Z$ , 我们要用 $\mathbb{R}^{d}$ -valued $\mathbb{F}$ -adapted stochastic process $\delta=\left(\delta_{k}\right)_{k=0, \ldots, n-1}$ with $\delta_{k}=\left(\delta_{k}^{1}, \ldots, \delta_{k}^{d}\right)$ 来交易 $S$．$\delta_{k}^{i}$ 表示智能体在 $t_{k}$ 时刻对第 $i$ 个资产的持有. 同样定义 $\delta_{-1}=\delta_{n}:=0$．

记 $\mathcal{H}^{u}$ 是这样的交易策略的无约束集合. 但是每个 $\delta_{k}$ 有其交易约束. 可能来自 liquidity, asset availability or trading restrictions. They are also used to restrict trading in a particular option prior to its availability. In the example above of an option which is listed in $3 \mathrm{M}$, the respective trading constraints would be $\{0\}$ until the $3 \mathrm{M}$ point. 所以我们假定 $\delta_{k}$ 受 $\mathcal{H}_{k}$ 约束，由一个连续 $\mathcal{F}_{k}$ 可测映射给出 $H_{k}: \mathbb{R}^{d(k+1)} \rightarrow \mathbb{R}^{d}$, i.e. $\mathcal{H}_{k}:=H_{k}\left(\mathbb{R}^{d(k+1)}\right) .$

对无约束策略 $\delta^{u} \in \mathcal{H}^{u}$, 我们定义它在 $\mathcal{H}_{k}$ 的有约束投影 $(H \circ \delta^{u})_{k}:=H_{k}((H \circ \delta^{u})_{0}, \ldots,(H \circ \delta^{u})_{k-1}, \delta_{k}^{u}$．记 $\mathcal{H}:=\left(H \circ \mathcal{H}^{u}\right) \subset \mathcal{H}^{u}$ 为受约束的交易策略相应的非空集．

**EXAMPLE 1** Assume that $S$ are a range of options and that $\mathcal{V}_{k}^{i}\left(S_{k}^{i}\right)$ computes the Black-Scholes Vega of each option using the various market parameters available at time $t_{k}$. The overall Vega traded with $\delta_{k}$ is then $\mathcal{V}_{k}\left(\delta_{k}-\delta_{k-1}\right):=$ $\left|\sum_{i=1}^{d} \mathcal{V}_{k}^{i}\left(S_{k}^{i}\right)\left(\delta_{k}^{i}-\delta_{k-1}^{i}\right)\right| .$ A liquidity limit of a maximum 
tradable Vega of $\mathcal{V}_{\max }$ could then be implemented by the map:
$$
\begin{aligned}
H_{k}\left(\delta_{0}, \ldots, \delta_{k}\right):&= \delta_{k-1}+\left(\delta_{k}-\delta_{k-1}\right) \\
& \times \frac{\mathcal{V}_{\max }}{\max \left\{\mathcal{V}_{k}\left(\delta_{k}-\delta_{k-1}\right), \mathcal{V}_{\max }\right\}} .
\end{aligned}
$$

#### 对冲

交易是自融资的．不带交易费用的 $T$ 时刻最终财富为 $-Z+p_{0}+(\delta \cdot S)_{T}$, 其中
$$
(\delta \cdot S)_{T}:=\sum_{k=0}^{n-1} \delta_{k} \cdot\left(S_{k+1}-S_{k}\right) .
$$
当考虑交易费用时，在$t_{k}$ 时刻买入 $n \in \mathbb{R}^{d}$ 的 $S$ 股票a产生费用 $c_{k}(n)$. $\delta$ 策略总的交易费用为
$$
C_{T}(\delta):=\sum_{k=0}^{n} c_{k}\left(\delta_{k}-\delta_{k-1}\right)
$$
回忆 $\delta_{-1}=\delta_{n}:=0$．所以智能体总的花费变为
$$
\mathrm{PL}_{T}\left(Z, p_{0}, \delta\right):=-Z+p_{0}+(\delta \cdot S)_{T}-C_{T}(\delta)
$$