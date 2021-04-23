# numpy  
----------------------
`import numpy as np`  
## 基础  
- 数组：np.array([[1,2,3],[4,5,6],...])  
- 序列：np.arange(a,b,c)  初值a，终值b，步长c，不含终值  
- 序列：np.linspace(a,b,c)    初值a，终值b，个数c，含终值  
- 空数组：np.empty((a,b),np.int) shape&type  
- 零数组：np.zeros((a,b),np.int)  
- 单位阵：np.eye(N,M=None,k=0) 行数N，列数M，k为对角线上移(正)或下移(负)  
- 全1阵：np.ones((a,b),np.int)  
- 转置与内积：np.dot(A.T,A)  
## 切片与索引  
切片是视图而非副本，若要副本：`arr[5:8].copy()`  
布尔型索引：data[data<0]=0  
~可用来反转条件  

## 通用函数func  
### 一元函数：np. *f* (arr)  
- abs,fabs,sqrt开根,square平方  
- exp,log,log10,log2  
- sign,ceil向上取整,floor向下取整,rint四舍五入,modf拆成整数和小数  
- isnan,isfinite,isinf  
- cos,sin,cosh,sinh,tan,tanh  
### 二元函数：np. *f* (arr)  
- add,substract,multiply,divide,floor_divide除后取整  
- power,maximum,fmax,mod
- copysign得到第二个数组的符号  
- greater,greater_equal,less,less_equal,equal,not_equal返回布尔值  
  
  