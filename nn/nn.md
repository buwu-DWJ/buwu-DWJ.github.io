书可参考[邱锡鹏：神经网络与深度学习](https://nndl.github.io/nndl-book.pdf)

# 神经元  
### 1. Sigmoid型函数
Sigmoid型函数是指一类S型曲线函数，为两端饱和函数．常用的Sigmoid型函数有Logistic函数和Tanh函数.  
>对于函数 $f(x)$ ，若 $x\rightarrow -\infin$ 时，其导数 $f'(x)\rightarrow 0$ ，则称其为左饱和．若 $x\rightarrow +\infin$ 时，其导数 $f'(x)\rightarrow 0$ ，则称其为右饱和．当同时满足左、右饱和时，就称为两端饱和．  
#### 1.1 Logistic型函数  
$$
\sigma (x) = \frac{1}{1+exp(-x)}
$$
#### 1.2 Tanh函数  
$$
tanh(x) = \frac{exp(x)-exp(-x)}{exp(x)+exp(-x)}
$$
Tanh函数可以看作放大并平移的Logistic函数，其值域是(−1,1)  
$$
tanh(x) = 2\sigma(2x) -1
$$
下图给出了Logistic函数和Tanh函数的形状．Tanh函数的输出是零中心化的（Zero-Centered），而Logistic函数的输出恒大于0．非零中心化的输出会使得其后一层的神经元的输入发生偏置偏移（Bias Shift），并进一步使得梯度下降的收敛速度变慢．
![](images/1.jpg)  

### 2. ReLU函数  
ReLU（Rectified Linear Unit，修正线性单元），也叫Rectifier函数，是目前深度神经网络中经常使用的激活函数．ReLU实际上是一个斜坡（ramp）函数，定义为
$$
ReLU(x) = 
    \begin{cases}
        x&, x\geq0 \\
        0&, x<0
    \end{cases}
$$
ReLU神经元训练时比较容易“**死亡**”，在训练时，如果参数在一次不恰当的更新后，第一个隐藏层中的某个ReLU神经元在所有的训练数据上都不能被激活，那么这个神经元自身参数的梯度永远都会是0，在以后的训练过程中永远不能被激活．这种现象称为死亡ReLU问题（DyingReLU Problem）.故有以下变种  
#### 2.1 带泄露的ReLU  
带泄露的ReLU（Leaky ReLU）在输入 $x<0$ 时，保持一个很小的梯度 $\gamma$ ．这样当神经元非激活时也能有一个非零的梯度可以更新参数，避免永远不能被激活
$$
\begin{aligned}
LeakyReLU(x) &= 
    \begin{cases}
        x&if x>0\\
        \gamma x&if x\leq 0
    \end{cases}\\
&=max(0,x)+\gamma min(0,x).
\end{aligned}
$$
#### 2.2 带参数的ReLU
带参数的ReLU（Parametric ReLU，PReLU）引入一个可学习的参数，不同神经元可以有不同的参数[He et al.,2015]．对于第𝑖个神经元，其PReLU的定义为
$$
\begin{aligned}
PReLU_i(x) &= 
    \begin{cases}
        x&if x>0\\
        \gamma_i x&if x\leq 0
    \end{cases}\\
&=max(0,x)+\gamma_i min(0,x).
\end{aligned}
$$
#### 2.3 ELU函数
ELU（Exponential Linear Unit，指数线性单元）是一个近似的零中心化的非线性函数，其定义为
$$
\begin{aligned}
ELU(x) &= 
    \begin{cases}
        x&if x>0\\
        \gamma(exp(x)-1) &if x\leq 0
    \end{cases}\\
&=max(0,x)+ min(0,\gamma (exp(x)-1)).
\end{aligned}
$$
其中𝛾 ≥ 0是一个超参数，决定𝑥 ≤ 0时的饱和曲线，并调整输出均值在0附近.
#### 2.4 Softplus函数  
Softplus函数可以看作Rectifier函数的平滑版本，其定义为
$$
Softplus(x) = log(1+exp(x))
$$
Softplus函数其导数刚好是Logistic函数．Softplus函数虽然也具有单侧抑制、宽兴奋边界的特性，却没有稀疏激活性
![](images/2.jpg)  

### 3. Swish函数
Swish函数是一种自门控（Self-Gated）激活函数，定义为
$$
swish(x) = x\sigma(\beta x).
$$
其中$\sigma(\cdot)$为Logistic函数，$\beta$ 为可学习的参数或一个固定超参数．$\sigma(\cdot)$𝜎(⋅) ∈ (0,1)可以看作一种软性的门控机制．当𝜎(𝛽𝑥)接近于1时，门处于“开”状态，激活函数的输出近似于𝑥本身；当𝜎(𝛽𝑥)接近于0时，门的状态为“关”，激活函数的输出近似于0
![](images/3.jpg)  
当 $\beta=0$ 时，Swish函数变成线性函数 $\frac{x}{2}$ ．当 $\beta=1$ 时，Swish函数在 $x>0$ 时近似线性，在 $x<0$ 时近似饱和，同时具有一定的非单调性．当 $\beta\rightarrow+\infin$ 时， $\sigma(\beta x)$ 趋向于离散的0-1函数，Swish函数近似为ReLU函数．因此，Swish函数可以看作线性函数和ReLU函数之间的非线性插值函数，其程度由参数 $\beta$ 控制.  
### 4. GELU函数
### 5. Maxout单元  
# 网络结构
### 网络结构总述