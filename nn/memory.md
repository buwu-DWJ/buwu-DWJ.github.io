# 循环神经网络

**循环神经网络**（Recurrent Neural Network，RNN）是一类具有短期记忆能力的神经网络．在循环神经网络中，神经元不但可以接受其他神经元的信息，也可以接受自身的信息，形成具有环路的网络结构．和前馈神经网络相比，循环神经网络更加符合生物神经网络的结构．循环神经网络已经被广泛应用在语音识别、语言模型以及自然语言生成等任务上．循环神经网络的参数学习可以通过**随时间反向传播算法**来学习．随时间反向传播算法即按照时间的逆序将错误信息一步步地往前传递．当输入序列比较长时，会存在梯度爆炸和消失问题，也称为长程依赖问题．为了解决这个问题，人们对循环神经网络进行了很多的改进，其中最有效的改进方式引入**门控机制**（Gating Mechanism）．  
此外，循环神经网络可以很容易地扩展到两种更广义的记忆网络模型：递归神经网络和图网络．  

## 一． 循环神经网络  

给定一个输入序列 $x_{1:T} = (x_1,x_2,...,x_t,...,x_T)$ ，循环神经网络通过下面公式更新带反馈边的隐藏层的活性值 $h_t$：
$$
h_t = f(h_{t-1},x_t),\tag{1.1}
$$
其中 $h_0 = 0$ ，$f(\cdot)$ 为一个非线性函数，可以是一个前馈网络．  
给出了循环神经网络的示例，其中“延时器”为一个虚拟单元，记录神经元的最近一次（或几次）活性值．  

![循环网络图例](img/5.JPG)  

从数学上讲，上式可以看成一个动力系统．因此，隐藏层的活性值 $h_t$ 在很多文献上也称为状态（State）或隐状态（Hidden State）．由于循环神经网络具有短期记忆能力，相当于存储装置，因此其计算能力十分强大．理论上，循环神经网络可以近似任意的非线性动力系统．前馈神经网络可以模拟任何连续函数，而循环神经网络可以模拟任何程序．  

## 二． 简单循环网络  

简单循环网络（Simple Recurrent Network，SRN）只有一个隐藏层．在一个两层的前馈神经网络中，连接存在于相邻的层与层之间，隐藏层的节点之间是无连接的．而简单循环网络增加了从隐藏层到隐藏层的反馈连接．  
令向量 $x_t\in \mathbb{R}^M$ 表示在时刻 $t$ 时网络的输入， $h_t\in \mathbb{R}^D$ 表示隐藏层状态（即隐藏层神经元活性值），则 $h_t$ 不仅和当前时刻的输入 $x_t$ 相关，也和上一个时刻的隐藏层状态 $h_{t-1}$ 相关．简单循环网络在时刻 $t$ 的更新公式为
$$
z_{t}=\boldsymbol{U} \boldsymbol{h}_{t-1}+\boldsymbol{W} \boldsymbol{x}_{t}+\boldsymbol{b}\tag{2.1}
$$
其中 $z_t$ 为隐藏层的净输入， $U\in \mathbb{R}^{D\times D}$ 为状态-状态权重矩阵， $W\in \mathbb{R}^{D\times M}$ 为状态-输入权重矩阵，$b\in \mathbb{R}^D$ 为偏置向量，$f(\cdot)$ 是非线性激活函数，通常为Logistic函数或Tanh函数．也经常直接写为
$$
\boldsymbol{h}_{t}=f\left(\boldsymbol{U} \boldsymbol{h}_{t-1}+\boldsymbol{W} \boldsymbol{x}_{t}+\boldsymbol{b}\right)\tag{2.2}
$$
下图给出了按时间展开的循环神经网络：  

![按时间展开的循环神经网络](img/2.PNG)  

### 循环神经网络的通用近似定理

一个完全连接的循环网络是任何非线性动力系统的近似器．  
**定理：循环神经网络的通用近似定理**[Haykin,2009]：如果一个完全连接的循环神经网络有足够数量的sigmoid型隐藏神经元，那么它可以以任意的准确率去近似任何一个非线性动力系统
$$
\begin{array}{l}
\boldsymbol{s}_{t}=g\left(\boldsymbol{s}_{t-1}, \boldsymbol{x}_{t}\right), \\
\boldsymbol{y}_{t}=o\left(\boldsymbol{s}_{t}\right),\tag{2.3}
\end{array}
$$
其中 $s_t$ 为每个时刻的隐状态， $x_t$ 是外部输入， $g(\cdot)$ 是可测的状态转换函数， $o(\cdot)$ 是连续输出函数，并且对状态空间的紧致性没有限制．  

## 三． 参数学习

循环神经网络的参数可以通过梯度下降方法来进行学习．  

以随机梯度下降为例，给定一个训练样本 $(\boldsymbol{x}, \boldsymbol{y})$ ，其中 $\boldsymbol{x}_{1: T}=\left(\boldsymbol{x}_{1}, \cdots, \boldsymbol{x}_{T}\right)$
为长度是 $T$ 的输入序列， $y_{1: T}=\left(y_{1}, \cdots, y_{T}\right)$ 是长度为 $T$ 的标签序列．即在每个时刻 $t$ ，都有一个监督信息 $y_{t}$ ，我们定义时刻 $t$ 的损失函数为
$$
\mathcal{L}_{t}=\mathcal{L}\left(y_{t}, g\left(\boldsymbol{h}_{t}\right)\right)\tag{3.1}
$$
其中 $g\left(\boldsymbol{h}_{t}\right)$ 为第 $t$ 时刻的输出， $\mathcal{L}$ 为可微分的损失函数，比如交叉熵．那么整个序列的损失函数为
$$
\mathcal{L}=\sum_{t=1}^{T} \mathcal{L}_{t}\tag{3.2}
$$
整个序列的损失函数 $\mathcal{L}$ 关于参数 $\boldsymbol{U}$ 的梯度为
$$
\frac{\partial \mathcal{L}}{\partial \boldsymbol{U}}=\sum_{t=1}^{T} \frac{\partial \mathcal{L}_{t}}{\partial \boldsymbol{U}}\tag{3.3}
$$
即每个时刻损失 $\mathcal{L}_{t}$ 对参数 $\boldsymbol{U}$ 的偏导数之和．  

循环神经网络中存在一个递归调用的函数 $f(\cdot)$, 因此其计算参数梯度的方式和前贵神经网络不太相同．在循环神经网络中主要有两种计算梯度的方式：随
时间反向传播 ( BPTT ) 算法和实时循环学习 ( RTRL ) 算法．  

### 3.1 随时间反向传播

随时间反向传播 ( BackPropagation Through Time, BPTT ) 算法的主要思想是通过类似前馈神经网络的错误反向传播算法 [Werbos, 1990]来计算梯度．  

BPTT 算法将循环神经网络看作一个展开的多层前馈网络，其中“每一层”对应循环网络中的“每个时刻”．这样，循环神经网络就可以按照前馈网络中的反向传播算法计算参数梯度．在“展开”的前馈网络中，所有层的参数是共 享的，因此参数的真实梯度是所有“展开层”的参数梯度之和．  

**计算偏导数** $\frac{\partial \mathcal{L}_{t}}{\partial U} \quad$ 先来计算第 $t$ 时刻损失对参数 $\boldsymbol{U}$ 的偏导数 $\frac{\partial \mathcal{L}_{t}}{\partial U}$ ．  

因为参数 $\boldsymbol{U}$ 和隐藏层在每个时刻 $k(1 \leq k \leq t)$ 的净输入 $\boldsymbol{z}_{k}=\boldsymbol{U} \boldsymbol{h}_{k-1}+$
$W x_{k}+b$ 有关，因此第 $t$ 时刻的损失函数 $\mathcal{L}_{t}$ 关于参数 $u_{i j}$ 的梯度为：
$$
\frac{\partial \mathcal{L}_{t}}{\partial u_{i j}}=\sum_{k=1}^{t} \frac{\partial^{+} z_{k}}{\partial u_{i j}} \frac{\partial \mathcal{L}_{t}}{\partial z_{k}}\tag{3.1.1}
$$
其中 $\frac{\partial^{+} z_{k}}{\partial u_{i j}}$ 表示**“直接”偏导数**，即公式 $z_{k}=U \boldsymbol{h}_{k-1}+\boldsymbol{W} \boldsymbol{x}_{k}+\boldsymbol{b}$ 中保持 $\boldsymbol{h}_{k-1}$ 不变，对 $u_{i j}$ 进行求偏导数，得到
$$
\begin{aligned}
\frac{\partial^{+} z_{k}}{\partial u_{i j}}&=[0, \cdots, {\left[\boldsymbol{h}_{k-1}\right]}, \cdots, 0]\\
&{\triangleq \mathbb{I}_{i}\left(\left[\boldsymbol{h}_{k-1}\right]_{j}\right),}\tag{3.1.2}
\end{aligned}
$$
其中 $\left[\boldsymbol{h}_{k-1}\right]_{j}$ 为第 $k-1$ 时刻隐状态的第 $j$ 维; $\mathbb{I}_{l}(x)$ 是除了第 $i$ 行值为 $x$ 外，其余都为 0 的行向量．  

定义**误差项** $\delta_{t, k}=\frac{\partial \mathcal{L}_{t}}{\partial z_{k}}$ 为第 $t$ 时刻的损失对第 $k$ 时刻隐藏神经层的净输入 $z_{k}$ 的导数，则当 $1 \leq k<t$ 时
$$
\begin{aligned}
\delta_{t, k} &=\frac{\partial \mathcal{L}_{t}}{\partial z_{k}} \\
&=\frac{\partial \boldsymbol{h}_{k}}{\partial z_{k}} \frac{\partial \boldsymbol{z}_{k+1}}{\partial \boldsymbol{h}_{k}} \frac{\partial \mathcal{L}_{t}}{\partial \boldsymbol{z}_{k+1}} \\
&=\operatorname{diag}\left(f^{\prime}\left(\boldsymbol{z}_{k}\right)\right) \boldsymbol{U}^{\top} \delta_{t, k+1} .\tag{3.1.3}
\end{aligned}
$$
由上面三式得到
$$
\frac{\partial \mathcal{L}_{t}}{\partial u_{i j}}=\sum_{k=1}^{t}\left[\delta_{t, k}\right]_{i}\left[\boldsymbol{h}_{k-1}\right]_{j}\tag{3.1.4}
$$
将上式写成矩阵形式为
$$
\frac{\partial \mathcal{L}_{t}}{\partial \boldsymbol{U}}=\sum_{k=1}^{t} \delta_{t, k} \boldsymbol{h}_{k-1}^{\top} .\tag{3.1.5}
$$
下图给出了误差项随时间进行反向传播算法的示例．  

![误差项随时间反向传播](img/18.PNG)  

**参数梯度** $\quad$ 由几式，得到整个序列的损失函数 $\mathcal{L}$ 关于参数 $\boldsymbol{U}$ 的梯度
$$
\frac{\partial \mathcal{L}}{\partial \boldsymbol{U}}=\sum_{t=1}^{T} \sum_{k=1}^{t} \delta_{t, k} \boldsymbol{h}_{k-1}^{\top}\tag{3.1.6}
$$
同理可得， $\mathcal{L}$ 关于权重 $\boldsymbol{W}$ 和偏置 $\boldsymbol{b}$ 的梯度为
$$
\begin{aligned}
\frac{\partial \mathcal{L}}{\partial \boldsymbol{W}}&=\sum_{t=1}^{T} \sum_{k=1}^{t} \delta_{t, k} \boldsymbol{x}_{k}^{\top} \\
\frac{\partial \mathcal{L}}{\partial \boldsymbol{b}}&=\sum_{t=1}^{T} \sum_{k=1}^{t} \delta_{t, k}\tag{3.1.7}
\end{aligned}
$$
**计算复杂度** $\quad$ 在BPTT算法中，参数的梯度需要在一个完整的“前向”计算和“反向”计算后才能得到并进行参数更新．

### 3.2 实时循环学习

与反向传播的 BPTT 算法不同的是, **实时循环学习** ( Real-Time Recurrent
Learning, RTRL ) 是通过前向传播的方式来计算梯度 [Williams et al., 1995]．  

假设循环神经网络中第 $t+1$ 时刻的状态 $\boldsymbol{h}_{t+1}$ 为
$$
\boldsymbol{h}_{t+1}=f\left(\boldsymbol{z}_{t+1}\right)=f\left(\boldsymbol{U} \boldsymbol{h}_{t}+\boldsymbol{W} \boldsymbol{x}_{t+1}+\boldsymbol{b}\right),\tag{3.2.1}
$$
其关于参数 $u_{i j}$ 的偏导数为
$$
\begin{aligned}
\frac{\partial \boldsymbol{h}_{t+1}}{\partial u_{i j}} &=\left(\frac{\partial^{+} z_{t+1}}{\partial u_{i j}}+\frac{\partial \boldsymbol{h}_{t}}{\partial u_{i j}} \boldsymbol{U}^{\top}\right) \frac{\partial \boldsymbol{h}_{t+1}}{\partial \boldsymbol{z}_{t+1}} \tag{3.2.2}\\
&=\left(\mathbb{l}_{i}\left(\left[\boldsymbol{h}_{t}\right]_{j}\right)+\frac{\partial \boldsymbol{h}_{t}}{\partial u_{i j}} \boldsymbol{U}^{\top}\right) \operatorname{diag}\left(f^{\prime}\left(\boldsymbol{z}_{t+1}\right)\right)\\
&=\left(\mathbb{l}_{i}\left(\left[\boldsymbol{h}_{t}\right]_{j}\right)+\frac{\partial \boldsymbol{h}_{t}}{\partial u_{i j}} \boldsymbol{U}^{\top}\right) \odot\left(f^{\prime}\left(\boldsymbol{z}_{t+1}\right)\right)^{\mathrm{T}},
\end{aligned}
$$
其中 $\mathbb{I}_{i}(x)$ 是除了第 $i$ 行值为 $x$ 外，其余都为 0 的行向量．  

RTRL 算法从第 1 个时刻开始，除了计算循环神经网络的隐状态之外，还利用上式依次前向计算偏导数 $\frac{\partial \boldsymbol{h}_{1}}{\partial u_{i j}}, \frac{\partial \boldsymbol{h}_{2}}{\partial u_{i j}}, \frac{\partial \boldsymbol{h}_{3}}{\partial u_{i j}}, \cdots$ ．  

这样，假设第 $t$ 个时刻存在一个监督信息，其损失函数为 $\mathcal{L}_{t}$ ，就可以同时计
算损失函数对 $u_{i j}$ 的偏导数
$$
\frac{\partial \mathcal{L}_{t}}{\partial u_{i j}}=\frac{\partial \boldsymbol{h}_{t}}{\partial u_{i j}} \frac{\partial \mathcal{L}_{t}}{\partial \boldsymbol{h}_{t}}\tag{3.2.3}
$$
这样在第 $t$ 时刻，可以实时地计算损失 $\mathcal{L}_{t}$ 关于参数 $\boldsymbol{U}$ 的梯度，并更新参数．参数 $\boldsymbol{W}$ 和 $\boldsymbol{b}$ 的梯度也可以同样按上述方法实时计算.  

**两种算法比较** $\quad$ RTRL算法和 BPTT 算法都是基于梯度下降的算法，分别通过前向模式和反向模式应用链式法则来计算梯度．在循环神经网络中，一般网络输出维度远低于输入维度，因此 BPTT 算法的计算量会更小，但是 BPTT 算法需要保存所有时刻的中间梯度，空间复杂度较高．RTRL算法不需要梯度回传，因此非常
适合用于需要在线学习或无限序列的任务中．  

## 四． 长程依赖问题

循环神经网络在学习过程中的主要问题是由于梯度消失或爆炸问题，很难建模**长时间间隔** ( Long Range ) 的状态之间的依赖关系．  

在 BPTT 算法中，将公式(6.36)展开得到
$$
\delta_{t, k}=\prod_{\tau=k}^{t-1}\left(\operatorname{diag}\left(f^{\prime}\left(\boldsymbol{z}_{\tau}\right)\right) \boldsymbol{U}^{\top}\right) \delta_{t, t} .\tag{4.1}
$$
如果定义 $\gamma \cong\left\|\operatorname{diag}\left(f^{\prime}\left(\boldsymbol{z}_{\tau}\right)\right) \boldsymbol{U}^{\top}\right\|$ ，则
$$
\delta_{t, k} \cong \gamma^{t-k} \delta_{t, t}\tag{4.2}
$$
若 $\gamma>1$ ，当 $t-k \rightarrow \infty$ 时， $\gamma^{t-k} \rightarrow \infty$ ．当间隔 $t-k$ 比较大时，梯度也变得很大，会造成系统不稳定，称为**梯度爆炸问题** ( Gradient Exploding Problem )．  

相反，若 $\gamma<1$ ，当 $t-k \rightarrow \infty$ 时， $\gamma^{t-k} \rightarrow 0$ ．当间隔 $t-k$ 比较大时，梯度也变得非常小，会出现和深层前馈神经网络类似的梯度消失问题 ( Vanishing
Gradient Problem )．  

>要注意的是，在循环神经网络中的梯度消失不是说 $\frac{\partial \mathcal{L}_{t}}{\partial U}$ 的梯度消失了，而是 $\frac{\partial \mathcal{L}_{t}}{\partial \boldsymbol{h}_{k}}$ 的梯度消失了 (当间隔 $t-k$ 比较大时 $)$ ．也就是说，参数 $U$ 的更新主要靠当前时刻 $t$ 的几个相邻状态 $\boldsymbol{h}_{k}$ 来更新，长距离的状态对参数 $\boldsymbol{U}$ 没有影响．  

由于循环神经网络经常使用非线性激活函数为 Logistic 函数或 Tanh 函数作为非线性激活函数，其导数值都小于 1 ，并且权重矩阵 $\|\boldsymbol{U}\|$ 也不会太大，因此如果时间间隔 $t-k$ 过大， $\delta_{t, k}$ 会趋向于 0 ，因而经常会出现梯度消失问题．  

虽然简单循环网络理论上可以建立长时间间隔的状态之间的依赖关系，但是由于梯度爆炸或消失问题，实际上只能学习到短期的依赖关系．这样，如果时刻 $t$ 的输出 $y_{t}$ 依赖于时刻 $k$ 的输入 $\boldsymbol{x}_{k}$ ，当间隔 $t-k$ 比较大时 ，简单神经网络很难建模这种长距离的依赖关系，称为**长程依赖问题** ( Long-Term Dependencies Problem )．  

### 4.1 改进方法

为了避免梯度爆炸或消失问题，一种最直接的方式就是选取合适的参数，同时使用非饱和的激活函数，尽量使得 $\operatorname{diag}\left(f^{\prime}(\boldsymbol{z})\right) \boldsymbol{U}^{\top} \approx 1$ ，这种方式需要足够的人
工调参经验，限制了模型的广泛应用．比较有效的方式是通过改进模型或优化方法来缓解循环网络的梯度爆炸和梯度消失问题．  

**梯度爆炸** $\quad$ 一般而言，循环网络的梯度爆炸问题比较容易解决，一般通过**权重衰减**或**梯度截断**来避免．  

权重衰减是通过给参数增加 $\ell_{1}$ 或 $\ell_{2}$ 范数的正则化项来限制参数的取值范围，从而使得 $\gamma \leq 1$ ．梯度截断是另一种有效的启发式方法，当梯度的模大于一定阈值时，就将它截断成为一个较小的数．  

**梯度消失** $\quad$ 梯度消失是循环网络的主要问题．除了使用一些优化技巧外，更有效的方式就是改变模型，比如让 $\boldsymbol{U}=\boldsymbol{I}$ ，同时令 $\frac{\partial \boldsymbol{h}_{t}}{\partial \boldsymbol{h}_{t-1}}=\boldsymbol{I}$ 为单位矩阵，即
$$
\boldsymbol{h}_{t}=\boldsymbol{h}_{t-1}+g\left(\boldsymbol{x}_{t} ; \theta\right),\tag{4.1.1}
$$
其中 $g(\cdot)$ 是一个非线性函数， $\theta$ 为参数．
公式(6.49)中， $\boldsymbol{h}_{t}$ 和 $\boldsymbol{h}_{t-1}$ 之间为线性依赖关系，且权重系数为 1 ，这样就不存在梯度爆炸或消失问题．但是，这种改变也丢失了神经元在反贵边上的非线性
激活的性质，因此也降低了模型的表示能力．  

为了避免这个缺点, 我们可以采用一种更加有效的改进策略:
$$
\boldsymbol{h}_{t}=\boldsymbol{h}_{t-1}+g\left(\boldsymbol{x}_{t}, \boldsymbol{h}_{t-1} ; \theta\right),\tag{4.1.2}
$$
这样 $\boldsymbol{h}_{t}$ 和 $\boldsymbol{h}_{t-1}$ 之间为既有线性关系，也有非线性关系，并且可以缓解梯度消失问题．但这种改进依然存在两个问题：

1. 梯度爆炸问题：令 $z_{k}=\boldsymbol{U} \boldsymbol{h}_{k-1}+\boldsymbol{W} \boldsymbol{x}_{k}+\boldsymbol{b}$ 为在第 $k$ 时刻函数 $g(\cdot)$ 的输入，在计算公式(6.34)中的误差项 $\delta_{t, k}=\frac{\partial \mathcal{L}_{t}}{\partial z_{k}}$ 时，梯度可能会过大，从而导致梯度爆炸问题．
2. 记忆容量 ( Memory Capacity ) 问题：随着 $\boldsymbol{h}_{t}$ 不断累积存储新的输入信息，会发生饱和现象．假设 $g(\cdot)$ 为 Logistic 函数，则随着时间 $t$ 的增长，$\boldsymbol{h}_{t}$ 会变得越来越大，从而导致 $\boldsymbol{h}$ 变得饱和．也就是说，隐状态 $\boldsymbol{h}_{t}$ 可以存储的信息是有限的，随着记忆单元存储的内容越来越多，其丢失的信息也越来越多．  

为了解决这两个问题,可以通过引入门控机制来进一步改进模型．

## 基于门控的循环神经网络
