# 机器学习  
**机器学习**（Machine Learning，ML）是指从有限的观测数据中学习（或“猜测”）出具有一般性的规律，并利用这些规律对未知数据进行预测的方法。  
传统的机器学习主要关注如何学习一个预测模型．一般需要首先将数据表示为一组**特征**（Feature），特征的表示形式可以是连续的数值、离散的符号或其他形式．然后将这些特征输入到预测模型，并输出预测结果．这类机器学习可以看作**浅层学习**（Shallow Learning）．浅层学习的一个重要特点是不涉及特征学习，其特征主要靠人工经验或特征转换方法来抽取．  
在实际任务中使用机器学习模型一般会包含以下几个步骤（如图1.2所示）：
1. 数据预处理：经过数据的预处理，如去除噪声等．比如在文本分类中，去除停用词等．
2. 特征提取：从原始数据中提取一些有效的特征．比如在图像分类中，提取边缘、尺度不变特征变换（Scale Invariant Feature Transform，SIFT）特征等．
3. 特征转换：对特征进行一定的加工，比如降维和升维．降维包括特征抽取（Feature Extraction）和特征选择（Feature Selection）两种途径．常用的特征转换方法有主成分分析（Principal Components Analysis，PCA）、线性判别分析（Linear Discriminant Analysis，LDA）等.
4. 预测：机器学习的核心部分，学习一个函数并进行预测．
![传统机器学习步骤](images/1.JPG)  

### 支持向量机  
支持向量机，Support Vector Machine(SVM)，属于监督学习中的二分类。
#### 核方法  
有些问题是线性不可分的，即特征空间存在超平面（hypersurface）将正类和负类分开。考虑使用非线性函数将非线性可分问题从原始的特征空间映射至更高维的希尔伯特空间，从而转化为线性可分问题。  
核方法(kernal method)定义映射函数的内积为核函数(kernal function)，$\kappa (X_1,X_2)=\phi(X_1)^T \phi(X_2)$以回避内积的显示计算。 
##### 常用核函数  
- 径向基核函数(RBF kernal)：$\kappa (X_1,X_2)=exp\left({-\frac{||X_1-X_2||^2}{2\sigma^2}}\right)$
- 多项式核函数：$\kappa (X_1,X_2)=(X_1^TX_2)^2$
- 拉普拉斯核(Laplacian kernal)：$\kappa (X_1,X_2)=exp\left({-\frac{||X_1-X_2||}{\sigma}}\right)$ 
- Sigmoid核：$\kappa (X_1,X_2)=tanh[a(X_1^2X_2)-b]，a,b>0$
#### Mercer定理  
