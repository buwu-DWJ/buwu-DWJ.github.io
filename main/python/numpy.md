# numpy  

`import numpy as np`  
### 基础  
- 数组：np.array([[1,2,3],[4,5,6],...])  
- 序列：np.arange(a,b,c)  初值a，终值b，步长c，不含终值  
- 序列：np.linspace(a,b,c)    初值a，终值b，个数c，含终值  
- 空数组：np.empty((a,b),np.int) shape&type  
- 零数组：np.zeros((a,b),np.int)  
- 单位阵：np.eye(N,M=None,k=0) 行数N，列数M，k为对角线上移(正)或下移(负)  
- 全1阵：np.ones((a,b),np.int)  
- 转置与内积：np.dot(A.T,A)  
### 切片与索引  
切片是视图而非副本，若要副本：`arr[5:8].copy()`  
布尔型索引：data[data<0]=0, \~可用来反转条件  

### 通用函数func  
#### 一元函数：np. *f* (arr)  
- abs,fabs,sqrt开根,square平方  
- exp,log,log10,log2  
- sign,ceil向上取整,floor向下取整,rint四舍五入,modf拆成整数和小数  
- isnan,isfinite,isinf  
- cos,sin,cosh,sinh,tan,tanh  
#### 二元函数：np. *f* (arr)  
- add,substract,multiply,divide,floor_divide 除后取整  
- power,maximum,fmax,mod
- copysign 得到第二个数组的符号  
- greater,greater_equal,less,less_equal,equal,not_equal返回布尔值  
- meshgrid 接受两个一维数组，产生两个二维数组，对应所有(x,y)对  
  
np.where()是 x if condition else y的矢量化版本  
np.where(arr>2,2,-2)  
np.where(arr>2,2,arr)  
### 数组统计方法  
- sum,mean,std,var,min,max
- argmax,argmin 索引,cumsum,cumprod  
- 查询数组中是否有true:all,any(示例:`(a=b).all()`)
### 排序  
sort  
### 集合运算,数字1  
- unique(x)
- intersect1d(x,y) 交集
- union1d(x,y) 并集  
- in1d(x,y) 包含于  
- setdiff1d(x,y) 差集  
- setxor1d(x,y) 对称差  

### 常用numpy.linalg函数(npl)  
- diag 对角阵和一维数组转化  
- dot,trace,det  
- eig 特征值特征向量  
- inv 逆  
- pinv Moore-Penrose 伪逆  
- qr QR分解  
- svd 奇异值分解  
- solve 解Ax=b ,A方针  
- lstsq Ax=b最小二乘解  
### 部分numpy.random函数  
- seed 确定随机数生成器种子  
- permutation 返回新的打乱的x,x不变  
- shuffle 原地打乱x  
- rand 均匀分布  
- randint 给定范围内随机取整数  
- randn 标准正态分布  
- binomial 二项分布  
- normal 正态分布  
- beta,gamma
- chisquare 卡方分布  
- uniform [0,1)均匀分布  
### 数组合并与拼接
- append(arr,values,axis=None)  