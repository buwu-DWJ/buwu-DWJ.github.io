# 机器学习  
### 支持向量机  
支持向量机，Support Vector Machine(SVM)，属于监督学习中的二分类。
#### 核方法  
有些问题是线性不可分的，即特征空间存在超平面(hypersurface)将正类和负类分开。考虑使用非线性函数将非线性可分问题从原始的特征空间映射至更高维的希尔伯特空间，从而转化为线性可分问题。  
核方法(kernal method)定义映射函数的内积为核函数(kernal function)，$\kappa (X_1,X_2)=\phi(X_1)^T \phi(X_2)$以回避内积的显示计算。 
##### 常用核函数  
- 径向基核函数(RBF kernal)：$\kappa (X_1,X_2)=exp\left({-\frac{||X_1-X_2||^2}{2\sigma^2}}\right)$
- 多项式核函数：$\kappa (X_1,X_2)=(X_1^TX_2)^2$
- 拉普拉斯核(Laplacian kernal)：$\kappa (X_1,X_2)=exp\left({-\frac{||X_1-X_2||}{\sigma}}\right)$ 
- Sigmoid核：$\kappa (X_1,X_2)=tanh[a(X_1^2X_2)-b]，a,b>0$
#### Mercer定理  
