# 前馈神经网络  

## 一． 前馈神经网络

**前馈神经网络**（Feedforward Neural Network，FNN）是最早发明的简单人工神经网络．前馈神经网络也经常称为**多层感知器**（Multi-Layer Perceptron，MLP）．  

在前馈神经网络中，各神经元分别属于不同的层．每一层的神经元可以接收前一层神经元的信号，并产生信号输出到下一层．第 0 层称为**输入层**，最后一层称为**输出层**，其他中间层称为**隐藏层**．整个网络中无反馈，信号从输入层向输出层单向传播，可用一个有向无环图表示．  

![网络结构示意图](img/5.PNG)  

下面用到的记号：  

- $L$ ：神经网络的层数
- $M_{l}$：第 $l$ 层神经元的个数
- $f_{l}(\cdot)$：第 $l$ 层神经元的激活函数
- $\boldsymbol{W}^{(l)} \in \mathbb{R}^{M_{l} \times M_{l-1}} \quad$ ：第 $l-1$ 层到第 $l$ 层的权重矩阵
- $\boldsymbol{b}^{(l)} \in \mathbb{R}^{M_{l}}:\text{第 } l-1 \text {层到第 } l \text { 层的偏置 }$
- $z^{(l)} \in \mathbb{R}^{M_{l}}$：第 $l$ 层神经元的净输入 $($ 净活性值 $)$
- $\boldsymbol{a}^{(l)} \in \mathbb{R}^{M_{l}}$：第 $l$ 层神经元的输出 $($ 活性值 $)$  

令 $\boldsymbol{a}^{(0)}=\boldsymbol{x}$ ，前馈神经网络通过不断迭代下面公式进行信息传播:
$$
\begin{array}{l}
\boldsymbol{z}^{(l)}=\boldsymbol{W}^{(l)} \boldsymbol{a}^{(l-1)}+\boldsymbol{b}^{(l)} \\
\boldsymbol{a}^{(l)}=f_{l}\left(\boldsymbol{z}^{(l)}\right)
\end{array}
$$

首先根据第 $l-1$ 层神经元的**活性值** ( Activation ) $\boldsymbol{a}^{(l-1)}$ 计算出第 $l$ 层神经元的**净活性值** ( Net Activation ) $\boldsymbol{z}^{(l)}$ ，然后经过一个激活函数得到第 $l$ 层神经元的活性
值因此，我们也可以把每个神经层看作一个仿射变换 ( Affine Transformation )
和一个非线性变换．
上述两式也可以合并写为:
$$
z^{(l)}=\boldsymbol{W}^{(l)} f_{l-1}\left(z^{(l-1)}\right)+\boldsymbol{b}^{(l)}
$$
或者
$$
\boldsymbol{a}^{(l)}=f_{l}\left(\boldsymbol{W}^{(l)} \boldsymbol{a}^{(l-1)}+\boldsymbol{b}^{(l)}\right)
$$

>**仿射变换**：又称仿射映射，是指在几何中，一个向量空间进行一次线性变换并接上一个平移，变换为另一个向量空间．  

这样, 前馈神经网络可以通过逐层的信息传递，得到网络最后的输出 $\boldsymbol{a}^{(L)}$ ．整个网络可以看作一个复合函数 $\phi(\boldsymbol{x} ; \boldsymbol{W}, \boldsymbol{b})$ ，将向量 $\boldsymbol{x}$ 作为第 1 层的输入 $\boldsymbol{a}^{(0)}$ ．将第 $L$ 层的输出 $\boldsymbol{a}^{(L)}$ 作为整个函数的输出．
$$
x=\boldsymbol{a}^{(0)} \rightarrow z^{(1)} \rightarrow \boldsymbol{a}^{(1)} \rightarrow \boldsymbol{z}^{(2)} \rightarrow \cdots \rightarrow \boldsymbol{a}^{(L-1)} \rightarrow \boldsymbol{z}^{(L)} \rightarrow \boldsymbol{a}^{(L)}=\phi(\boldsymbol{x} ; \boldsymbol{W}, \boldsymbol{b})，
$$

其中 $\boldsymbol{W}, \boldsymbol{b}$ 表示网络中所有层的连接权重和偏置．  

**通用近似定理**（Universal Approximation Theorem ) [Cy-
benko, 1989; Hornik et al., 1989]: 令 $\phi(\cdot)$ 是一个非常数、有界、单调递增的连续函数，$\mathcal{J}_{D}$ 是一个 $D$ 维的单位超立方体 $[0,1]^{D}，C\left(\mathcal{J}_{D}\right)$ 是定义在 $\mathcal{J}_{D}$ 上的连续函数集合．对于任意给定的一个函数 $f \in C\left(\mathcal{T}_{D}\right)$ ，存在一个整数 $M$ ，和一组实数 $v_{m}, b_{m} \in \mathbb{R}$ 以及实数向量 $\boldsymbol{w}_{m} \in \mathbb{R}^{D}, m=1, \cdots, M$ ，以至于我
们可以定义函数
$$
F(\boldsymbol{x})=\sum_{m=1}^{M} v_{m} \phi\left(\boldsymbol{w}_{m}^{\top} \boldsymbol{x}+b_{m}\right)
$$
作为函数 $f$ 的近似实现，即
$$
|F(\boldsymbol{x})-f(\boldsymbol{x})|<\epsilon, \quad \forall \boldsymbol{x} \in \mathcal{J}_{D}
$$
其中 $\epsilon>0$ 是一个很小的正数．  

## 二． 反向传播  

假设采用随机梯度下降进行神经网络参数学习，给定一个样本 $(\boldsymbol{x}, \boldsymbol{y})$ ，将其输入到神经网络模型中，得到网络输出为 $\hat{y}$ ．假设损失函数为 $\mathcal{L}(\boldsymbol{y}, \hat{\boldsymbol{y}})$ ，要进行参数学习就需要计算损失函数关于每个参数的导数．  

不失一般性，对第 $l$ 层中的参数 $\boldsymbol{W}^{(l)}$ 和 $\boldsymbol{b}^{(l)}$ 计算偏导数．因为 $\frac{\partial \mathcal{L}(\boldsymbol{y}, \hat{\boldsymbol{y}})}{\partial \boldsymbol{W}^{(l)}}$ 的计算
涉及向量对矩阵的微分，十分繁銷，因此我们先计算 $\mathcal{L}(\boldsymbol{y}, \hat{\boldsymbol{y}})$ 关于参数矩阵中每个元素的偏导数 $\frac{\partial \mathcal{L}(\boldsymbol{y}, \boldsymbol{y})}{\partial w_{i j}^{(l)}}$ ．根据链式法则，
$$
\begin{aligned}
\frac{\partial \mathcal{L}(\boldsymbol{y}, \hat{\boldsymbol{y}})}{\partial w_{i j}^{(l)}}=\frac{\partial \boldsymbol{z}^{(l)}}{\partial w_{i j}^{(l)}} \frac{\partial \mathcal{L}(\boldsymbol{y}, \hat{\boldsymbol{y}})}{\partial \boldsymbol{z}^{(l)}} \\
\frac{\partial \mathcal{L}(\boldsymbol{y}, \hat{\boldsymbol{y}})}{\partial \boldsymbol{b}^{(l)}}=\frac{\partial \boldsymbol{z}^{(l)}}{\partial \boldsymbol{b}^{(l)}} \frac{\partial \mathcal{L}(\boldsymbol{y}, \hat{\boldsymbol{y}})}{\partial \boldsymbol{z}^{(l)}}
\end{aligned}
$$
上式中的第二项都是目标函数关于第 $l$ 层的神经元 $z^{(l)}$
的偏导数，称为**误差项**，可以一次计算得到，这样我们只需要计算三个偏导数, 分 别为 $\frac{\partial z^{(l)}}{\partial w_{i j}^{(l)}}, \frac{\partial z^{(l)}}{\partial \boldsymbol{b}^{(l)}}$ 和 $\frac{\partial \mathcal{L}(\boldsymbol{y}, \hat{\boldsymbol{y}})}{\partial \boldsymbol{z}^{(l)}} .$
下面分别来计算这三个偏导数．  

1. 计算偏导数 $\frac{\partial z^{(l)}}{\partial w_{i j}^{(l)}} \quad$ 因 $z^{(l)}=\boldsymbol{W}^{(l)} \boldsymbol{a}^{(l-1)}+\boldsymbol{b}^{(l)}$ ，偏导数
$$
\begin{aligned}
\frac{\partial \boldsymbol{z}^{(l)}}{\partial w_{i j}^{(l)}} &=[\frac{\partial z_{1}^{(l)}}{\partial w_{i j}^{(l)}}, \cdots, {\frac{\partial z_{i}^{(l)}}{\partial w_{i j}^{(l)}}}, \cdots, \frac{\partial z_{M_{l}}^{(l)}}{\partial w_{i j}^{(l)}}] \\
&=[0, \cdots, \frac{\partial\left(\boldsymbol{w}_{i:}^{(l)} \boldsymbol{a}^{(l-1)}+b_{i}^{(l)}\right)}{\partial w_{i j}^{(l)}}, \cdots, 0] \\
&=\left[0, \cdots, a_{j}^{(l-1)}, \cdots, 0\right] \\
& \triangleq \mathbb{I}_{i}\left(a_{j}^{(l-1)}\right) \in \mathbb{R}^{1 \times M_{l}},
\end{aligned}
$$
其中 $\boldsymbol{w}_{i:}^{(l)}$ 为权重矩阵 $\boldsymbol{W}^{(l)}$ 的第 $i$ 行， $\mathbb{I}_{i}\left(a_{j}^{(l-1)}\right)$ 表示第 $i$ 个元素为 $a_{j}^{(l-1)}$ ，其余为 0 的行向量．  

2. 计算偏导数 $\frac{\partial z^{(l)}}{\partial b^{(l)}} \quad$ 因为 $z^{(l)}$ 和 $\boldsymbol{b}^{(l)}$ 的函数关系为 $z^{(l)}=\boldsymbol{W}^{(l)} \boldsymbol{a}^{(l-1)}+$
$\boldsymbol{b}^{(l)}$ ，因此偏导数
$$
\frac{\partial \boldsymbol{z}^{(l)}}{\partial \boldsymbol{b}^{(l)}}=\boldsymbol{I}_{M_{l}} \in \mathbb{R}^{M_{l} \times M_{l}}
$$
为 $M_{l} \times M_{l}$ 的单位矩阵．  

3. 计算偏导数 $\frac{\partial \mathcal{L}(\boldsymbol{y}, \hat{y})}{\partial z^{(l)}} \quad$ 偏导数 $\frac{\partial \mathcal{L}(\boldsymbol{y}, \hat{\boldsymbol{y}})}{\partial \boldsymbol{z}^{(l)}}$ 表示第 $l$ 层神经元对最终损失
的影响，也反映了最终损失对第 $l$ 层神经元的敏感程度，因此一般称为第 $l$ 层神经元的**误差项**，用 $\delta^{(l)}$ 来表示．
$$
\delta^{(l)} \triangleq \frac{\partial \mathcal{L}(\boldsymbol{y}, \hat{\boldsymbol{y}})}{\partial \boldsymbol{z}^{(l)}} \in \mathbb{R}^{M_{l}}
$$  

误差项 $\delta^{(l)}$ 也间接反映了不同神经元对网络能力的贡献程度，从而比较好地解决
了贡献度分配问题 ( Credit Assignment Problem, CAP )．  

根据 $\boldsymbol{z}^{(l+1)}=\boldsymbol{W}^{(l+1)} \boldsymbol{a}^{(l)}+\boldsymbol{b}^{(l+1)}$ ，有
$$
\frac{\partial z^{(l+1)}}{\partial \boldsymbol{a}^{(l)}}=\left(\boldsymbol{W}^{(l+1)}\right)^{\top} \in \mathbb{R}^{M_{l} \times M_{l+1}}
$$
根据 $\boldsymbol{a}^{(l)}=f_{l}\left(\boldsymbol{z}^{(l)}\right)$ ，其中 $f_{l}(\cdot)$ 为按位计算的函数，因此有
$$
\begin{aligned}
\frac{\partial \boldsymbol{a}^{(l)}}{\partial \boldsymbol{z}^{(l)}} &=\frac{\partial f_{l}\left(\boldsymbol{z}^{(l)}\right)}{\partial \boldsymbol{z}^{(l)}} \\
&=\operatorname{diag}\left(f_{l}^{\prime}\left(\boldsymbol{z}^{(l)}\right)\right) \quad \in \mathbb{R}^{M_{l} \times \boldsymbol{M}_{l}}
\end{aligned}
$$
因此，根据链式法则，第 $l$ 层的误差项为
$$
\begin{aligned}
\delta^{(l)} & \triangleq \frac{\partial \mathcal{L}(\boldsymbol{y}, \hat{\boldsymbol{y}})}{\partial \boldsymbol{z}^{(l)}} \\
&=\frac{\partial \boldsymbol{a}^{(l)}}{\partial \boldsymbol{z}^{(l)}} \cdot \frac{\partial \boldsymbol{z}^{(l+1)}}{\partial \boldsymbol{a}^{(l)}} \cdot {\frac{\partial \mathcal{L}(\boldsymbol{y}, \hat{\boldsymbol{y}})}{\partial \boldsymbol{z}^{(l+1)}}}] \\
&={\operatorname{diag}\left(f_{l}^{\prime}\left(\boldsymbol{z}^{(l)}\right)\right) \cdot\left(\boldsymbol{W}^{(l+1)}\right)^{\mathrm{T}} \cdot \cdot \delta^{(l+1)}} \\
&=f_{l}^{\prime}\left(\boldsymbol{z}^{(l)}\right) \odot\left(\left(\boldsymbol{W}^{(l+1)}\right)^{\top} \delta^{(l+1)}\right) \in \mathbb{R}^{M_{l}},
\end{aligned}
$$
其中 $\odot$ 是向量的点积运算符，表示每个元素相乘．  

从上式可以看出，第 $l$ 层的误差项可以通过第 $l+1$ 层的误差项计算得到，这就是误差的**反向传播** ( BackPropagation, BP )．反向传播算法的含义是:
第 $l$ 层的一个神经元的误差项 $($ 或敏感性 $)$ 是所有与该神经元相连的第 $l+1$ 层
的神经元的误差项的权重和．然后，再乘上该神经元激活函数的梯度．  

在计算出上面三个偏导数之后，最初的式子可以写为
$$
\begin{aligned}
\frac{\partial \mathcal{L}(\boldsymbol{y}, \hat{\boldsymbol{y}})}{\partial w_{i j}^{(l)}} &=\mathbb{l}_{i}\left(a_{j}^{(l-1)}\right) \delta^{(l)} \\
&=\left[0, \cdots, a_{j}^{(l-1)}, \cdots, 0\right]\left[\delta_{1}^{(l)}, \cdots, \delta_{i}^{(l)}, \cdots, \delta_{M_{l}}^{(l)}\right]^{\top} \\
&=\delta_{i}^{(l)} a_{j}^{(l-1)}
\end{aligned}
$$
其中 $\delta_{i}^{(l)} a_{j}^{(l-1)}$ 相当于向量 $\delta^{(l)}$ 和向量 $\boldsymbol{a}^{(l-1)}$ 的外积的第 $i, j$ 个元素．上式可以进一步写为
$$
\left[\frac{\partial \mathcal{L}(\boldsymbol{y}, \hat{\boldsymbol{y}})}{\partial \boldsymbol{W}^{(l)}}\right]_{i j}=\left[\delta^{(l)}\left(\boldsymbol{a}^{(l-1)}\right)^{\top}\right]_{i j} .
$$
因此， $\mathcal{L}(\boldsymbol{y}, \hat{\boldsymbol{y}})$ 关于第 $l$ 层权重 $\boldsymbol{W}^{(l)}$ 的梯度为
$$
\frac{\partial \mathcal{L}(\boldsymbol{y}, \hat{\boldsymbol{y}})}{\partial \boldsymbol{W}^{(l)}}=\delta^{(l)}\left(\boldsymbol{a}^{(l-1)}\right)^{\mathrm{T}} \in \mathbb{R}^{M_{l} \times M_{l-1}}．
$$

同理, $\mathcal{L}(\boldsymbol{y}, \hat{\boldsymbol{y}})$ 关于第 $l$ 层偏置 $\boldsymbol{b}^{(l)}$ 的梯度为
$$
\frac{\partial \mathcal{L}(\boldsymbol{y}, \hat{\boldsymbol{y}})}{\partial \boldsymbol{b}^{(l)}}=\delta^{(l)} \in \mathbb{R}^{M_{l}}
$$
在计算出每一层的误差项之后，我们就可以得到每一层参数的梯度．因此，使用误差反向传播算法的前贵神经网络训练过程可以分为以下三步：

1. 前馈计算每一层的净输入 $z^{(l)}$ 和激活值 $\boldsymbol{a}^{(l)}$ ，直到最后一层；
2. 反向传播计算每一层的误差项 $\delta^{(l)}$ ；
3. 计算每一层参数的偏导数，并更新参数．  

![反向传播算法](img/6.PNG)  

## 三． 自动微分

**自动微分**（Automatic Differentiation，AD）．  

为简单起见，这里以一个神经网络中常见的复合函数的例子来说明自动微分的过程．令复合函数 $f(x ; w, b)$ 为
$$
f(x ; w, b)=\frac{1}{\exp (-(w x+b))+1},
$$
其中 $x$ 为输入标量， $w$ 和 $b$ 分别为权重和偏置参数．  

首先，我们将复合函数 $f(x ; w, b)$ 分解为一系列的基本操作，并构成一个计算图 ( Computational Graph )．计算图是数学运算的图形化表示．计算图中的每个非叶子节点表示一个基本操作，每个叶子节点为一个输入变量或常量．下图给
出了当 $x=1, w=0, b=0$ 时复合函数 $f(x ; w, b)$ 的计算图，其中连边上的红色数字表示前向计算时复合函数中每个变量的实际取值．

![自动微分计算图](img/7.PNG)  

从计算图上可以看出，复合函数 $f(x ; w, b)$ 由 6 个基本函数 $h_{i}, 1 \leq i \leq 6$ 组成．如下图所示，每个基本函数的导数都十分简单，可以通过规则来实现．  

![函数导数](img/8.PNG)  

整个复合函数 $f(x ; w, b)$ 关于参数 $w$ 和 $b$ 的导数可以通过计算图上的节点
$f(x ; w, b)$ 与参数 $w$ 和 $b$ 之间路径上所有的导数连乘来得到，即
$$
\begin{aligned}
\frac{\partial f(x ; w, b)}{\partial w}=\frac{\partial f(x ; w, b)}{\partial h_{6}} \frac{\partial h_{6}}{\partial h_{5}} \frac{\partial h_{5}}{\partial h_{4}} \frac{\partial h_{4}}{\partial h_{3}} \frac{\partial h_{3}}{\partial h_{2}} \frac{\partial h_{2}}{\partial h_{1}} \frac{\partial h_{1}}{\partial w}, \\
\frac{\partial f(x ; w, b)}{\partial b}=\frac{\partial f(x ; w, b)}{\partial h_{6}} \frac{\partial h_{6}}{\partial h_{5}} \frac{\partial h_{5}}{\partial h_{4}} \frac{\partial h_{4}}{\partial h_{3}} \frac{\partial h_{3}}{\partial h_{2}} \frac{\partial h_{2}}{\partial b}
\end{aligned}
$$
以 $\frac{\partial f(x ; w, b)}{\partial w}$ 为例，当 $x=1, w=0, b=0$ 时，可以得到
$$
\begin{aligned}
\left.\frac{\partial f(x ; w, b)}{\partial w}\right|_{x=1, w=0, b=0} &=\frac{\partial f(x ; w, b)}{\partial h_{6}} \frac{\partial h_{6}}{\partial h_{5}} \frac{\partial h_{5}}{\partial h_{4}} \frac{\partial h_{4}}{\partial h_{3}} \frac{\partial h_{3}}{\partial h_{2}} \frac{\partial h_{2}}{\partial h_{1}} \frac{\partial h_{1}}{\partial w} \\
&=1 \times-0.25 \times 1 \times 1 \times-1 \times 1 \times 1 \\
&=0.25 .
\end{aligned}
$$
如果函数和参数之间有多条路径，可以将这多条路径上的导数再进行相加，得到最终的梯度．  

按照计算导数的顺序，自动微分可以分为两种模式：前向模式和反向模式．  

**前向模式** $\quad$ 前向模式是按计算图中计算方向的相同方向来递归地计算梯度．以 $\frac{\partial f(x ; w, b)}{\partial w}$ 为例，当 $x=1, w=0, b=0$ 时，前向模式的累积计算顺序如下:
$$
\begin{aligned}
\frac{\partial h_{1}}{\partial w}=x&=1, \\
\frac{\partial h_{2}}{\partial w}=\frac{\partial h_{2}}{\partial h_{1}} \frac{\partial h_{1}}{\partial w}=1 \times 1&=1, \\
\frac{\partial h_{3}}{\partial w}=\frac{\partial h_{3}}{\partial h_{2}} \frac{\partial h_{2}}{\partial w}&=-1 \times 1\\
\vdots \qquad \qquad \qquad \vdots\\
\frac{\partial h_{6}}{\partial w}=\frac{\partial h_{6}}{\partial h_{5}} \frac{\partial h_{5}}{\partial w}&=-0.25 \times-1=0.25 \\
\frac{\partial f(x ; w, b)}{\partial w}=\frac{\partial f(x ; w, b)}{\partial h_{6}} \frac{\partial h_{6}}{\partial w}&=1 \times 0.25=0.25
\end{aligned}
$$
**反向模式** $\quad$ 反向模式是按计算图中计算方向的相反方向来递归地计算梯度．以 $\frac{\partial f(x ; w, b)}{\partial w}$ 为例，当 $x=1, w=0, b=0$ 时，反向模式的累积计算顺序如下：
$$
\begin{aligned}
\frac{\partial f(x ; w, b)}{\partial h_{6}}&=1 \\
\frac{\partial f(x ; w, b)}{\partial h_{5}}=\frac{\partial f(x ; w, b)}{\partial h_{6}} \frac{\partial h_{6}}{\partial h_{5}}&=1 \times-0.25 \\
\frac{\partial f(x ; w, b)}{\partial h_{4}}=\frac{\partial f(x ; w, b)}{\partial h_{5}} \frac{\partial h_{5}}{\partial h_{4}}&=-0.25 \times 1=-0.25, \\
\vdots \qquad \qquad \qquad \vdots \\
\frac{\partial f(x ; w, b)}{\partial w}=\frac{\partial f(x ; w, b)}{\partial h_{1}} \frac{\partial h_{1}}{\partial w}&=0.25 \times 1=0.25
\end{aligned}
$$
前向模式和反向模式可以看作应用链式法则的两种梯度累积方式．从反向模式的计算顺序可以看出，反向模式和反向传播的计算梯度的方式相同．对于一般的函数形式 $f: \mathbb{R}^{N} \rightarrow \mathbb{R}^{M}$ ，前向模式需要对每一个输入变量都进行一遍遍历，共需要 $N$ 遍．而反向模式需要对每一个输出都进行一个遍历，共需要 $M$ 遍．当 $N>M$ 时，反向模式更高效．在前馈神经网络的参数学习中，风险函数为 $f: \mathbb{R}^{N} \rightarrow \mathbb{R}$ ，输出为标量，因此采用反向模式为最有效的计算方式，只需要一遍计算．  

>静态计算图和动态计算图计算图按构建方式可以分为静态计算图（StaticCom-putational Graph）和动态计算图（Dynamic Computational Graph）．在目前深度学习框架里，Theano和Ten-sorflow采用的是静态计算图，而DyNet、Chainer和PyTorch采用的是动态计算图．Tensorflow 2.0也支持了动态计算图．静态计算图是在编译时构建计算图，计算图构建好之后在程序运行时不能改变，而动态计算图是在程序运行时动态构建．两种构建方式各有优缺点．静态计算图在构建时可以进行优化，并行能力强，但灵活性比较差．动态计算图则不容易优化，当不同输入的网络结构不一致时，难以并行计算，但是灵活性比较高．

## 四． 卷积神经网络  

**卷积神经网络**（Convolutional Neural Network，CNN或ConvNet）是一种具有局部连接、权重共享等特性的深层前馈神经网络．卷积神经网络最早主要是用来处理图像信息．在用全连接前馈网络来处理图像时，会存在以下两个问题：  

1. 参数太多：如果输入图像大小为 100×100×3（即图像高度为 100 ，宽度为 100 以及RGB 3 个颜色通道），在全连接前馈网络中，第一个隐藏层的每个神经元到输入层都有 100 × 100 × 3 = 30000 个互相独立的连接，每个连接都对应一个权重参数．随着隐藏层神经元数量的增多，参数的规模也会急剧增加．这会导致整个神经网络的训练效率非常低，也很容易出现过拟合．
2. 局部不变性特征：自然图像中的物体都具有局部不变性特征，比如尺度缩放、平移、旋转等操作不影响其语义信息．而全连接前馈网络很难提取这些局部不变性特征，一般需要进行数据增强来提高性能．  

卷积神经网络是受生物学上感受野机制的启发而提出的．**感受野**（Recep-tive Field）机制主要是指听觉、视觉等神经系统中一些神经元的特性，即神经元只接受其所支配的刺激区域内的信号．在视觉神经系统中，视觉皮层中的神经细胞的输出依赖于视网膜上的光感受器．视网膜上的光感受器受刺激兴奋时，将神经冲动信号传到视觉皮层，但不是所有视觉皮层中的神经元都会接受这些信号．一个神经元的感受野是指视网膜上的特定区域，只有这个区域内的刺激才能够激活该神经元．  

目前的卷积神经网络一般是由卷积层、汇聚层和全连接层交叉堆叠而成的前馈神经网络．卷积神经网络有三个结构上的特性：**局部连接**、**权重共享**以及**汇聚**．这些特性使得卷积神经网络具有一定程度上的平移、缩放和旋转不变性．和前馈神经网络相比，卷积神经网络的参数更少．卷积神经网络主要使用在图像和视频分析的各种任务（比如图像分类、人脸识别、物体识别、图像分割等）上，其准确率一般也远远超出了其他的神经网络模型．近年来卷积神经网络也广泛地应用到自然语言处理、推荐系统等领域．  

## 4.1 卷积

### 4.1.1 卷积的定义

卷积（Convolution），也叫褶积，是分析数学中一种重要的运算．在信号处理或图像处理中，经常使用一维或二维卷积．  

#### 4.1.1.1 一维卷积

一维卷积经常用在信号处理中，用于计算信号的延迟累积．假设一个信号发生器每个时刻 $t$ 产生一个信号 $x_{t}$ ，其信息的衰减率为 $w_{k}$ ，即在 $k-1$ 个时间步长后，信息为原来的 $w_{k}$ 倍．假设 $w_{1}=1, w_{2}=1 / 2, w_{3}=1 / 4$ ，那么在**时刻 $t$ 收到的信号 $y_{t}$** 为当前时刻产生的信息和以前时刻延迟信息的叠加．
$$
\begin{aligned}
y_{t} &=1 \times x_{t}+1 / 2 \times x_{t-1}+1 / 4 \times x_{t-2} \\
&=w_{1} \times x_{t}+w_{2} \times x_{t-1}+w_{3} \times x_{t-2} \\
&=\sum_{k=1}^{3} w_{k} x_{t-k+1}
\end{aligned}
$$
我们把 $w_{1}, w_{2}, \cdots$ 称为**滤波器** ( Filter ) 或**卷积核** ( Convolution Kernel )．假设滤波器长度为 $K$ ，它和一个信号序列 $x_{1}, x_{2}, \cdots$ 的卷积为
$$
y_{t}=\sum_{k=1}^{K} w_{k} x_{t-k+1}
$$
为了简单起见，这里假设卷积的输出 $y_{t}$ 的下标 $t$ 从 $K$ 开始．  

信号序列 $x$ 和滤波器 $\boldsymbol{w}$ 的卷积定义为
$$
y=w *x
$$
其中 $*$ 表示卷积运算．一般情况下滤波器的长度 $K$ 远小于信号序列 $\boldsymbol{x}$ 的长度．  

我们可以设计不同的滤波器来提取信号序列的不同特征．比如，当令滤波器 $\boldsymbol{w}=[1 / K, \cdots, 1 / K]$ 时，卷积相当于信号序列的简单移动平均 $($ 窗口大小为 $K)$；当令滤波器 $\boldsymbol{w}=[1,-2,1]$ 时，可以近似实现对信号序列的二阶微分，即
$$
x^{\prime \prime}(t)=x(t+1)+x(t-1)-2 x(t) .
$$
下图给出了两个滤波器的一维卷积示例．可以看出，两个滤波器分别提取了输入序列的不同特征．滤波器 $\boldsymbol{w}=[1 / 3,1 / 3,1 / 3]$ 可以检测信号序列中的低频信息，而滤波器 $\boldsymbol{w}=[1,-2,1]$ 可以检测信号序列中的高频信息．（高低频指信号变化的强烈程度）  

![一维滤波器](img/9.PNG)  

#### 4.1.1.2 二维卷积  

卷积也经常用在图像处理中．因为图像为一个二维结构，所以需要将一维卷积进行扩展．给定一个图像 $\boldsymbol{X} \in \mathbb{R}^{M \times N}$ 和一个滤波器 $\boldsymbol{W} \in \mathbb{R}^{U \times V}$ ，一般
$U<<M, V<<N$ ，其卷积为
$$
y_{i j}=\sum_{u=1}^{U} \sum_{v=1}^{V} w_{u v} x_{i-u+1, j-v+1}\tag{4.1}
$$
为了简单起见，这里假设卷积的输出 $y_{i j}$ 的下标 $(i, j)$ 从 $(U, V)$ 开始．  

输入信息 $\boldsymbol{X}$ 和滤波器 $\boldsymbol{W}$ 的二维卷积定义为
$$
\boldsymbol{Y}=\boldsymbol{W} *\boldsymbol{X}
$$
其中*表示二维卷积运算. 下图给出了二维卷积示例.  

![二维卷积示意图](img/10.PNG)  

在图像处理中常用的**均值滤波** ( Mean Filter ) 就是一种二维卷积，将当前位置的像素值设为滤波器窗口中所有像素的平均值，即 $w_{u v}=\frac{1}{U V}$ ．  

在图像处理中，卷积经常作为特征提取的有效方法．一幅图像在经过卷积操作后得到结果称为**特征映射**（Feature Map）．下图给出在图像处理中几种常用的滤波器，以及其对应的特征映射．图中最上面的滤波器是常用的**高斯滤波器**，可以用来对图像进行平滑去噪；中间和最下面的滤波器可以用来提取边缘特征．

![常用滤波器示意](img/11.PNG)  

### 4.1.2 互相关

在机器学习和图像处理领域，卷积的主要功能是在一个图像 ( 或某种特征 ) 居滑动一个卷积核 ( 即滤波器 $),$ 通过卷积操作得到一组新的特征．在计算卷积的过程中，需要进行**卷积核翻转**．在具体实现上，一般会以互相关操作来代替卷积，从而会减少一些不必要的操作或开销．互相关 ( Cross-Correlation ) 是一个
衡量两个序列相关性的函数，通常是用滑动窗口的点积计算来实现．给定一个图像 $X \in \mathbb{R}^{M \times N}$ 和卷积核 $\boldsymbol{W} \in \mathbb{R}^{U \times V}$ ，它们的互相关为
$$
y_{i j}=\sum_{u=1}^{U} \sum_{v=1}^{V} w_{u v} x_{i+u-1, j+v-1}\tag{4.2}
$$
和公式 (4.1) 对比可知，互相关和卷积的区别仅仅在于卷积核是否进行翻转．因此互相关也可以称为**不翻转卷积**．  

公式 (4.2) 可以表述为
$$
\begin{aligned}
\boldsymbol{Y}&=\boldsymbol{W} \otimes \boldsymbol{X}
&=\operatorname{rot} 180(\boldsymbol{W}) * \boldsymbol{X}
\end{aligned}
$$
其中 $\otimes$ 表示互相关运算， $\operatorname{rot} 180(\cdot)$ 表示旋转 180 度，$\boldsymbol{Y} \in \mathbb{R}^{M-U+1, N-V+1}$ 为输出
矩阵．  

在神经网络中使用卷积是为了进行特征抽取，卷积核是否进行翻转和其特征抽取的能力无关．特别是当卷积核是可学习的参数时，卷积和互相关在能力上是等价的．因此，为了实现上 (或描述上 ) 的方便起见，我们用互相关来代替卷积．事实上，很多深度学习工具中卷积操作其实都是互相关操作．  

### 4.1.3 卷积的变种
