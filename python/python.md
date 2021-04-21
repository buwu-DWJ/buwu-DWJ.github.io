# numpy  
----------------------
`import numpy as np`  
- 数组：np.array([[1,2,3],[4,5,6],...])  
- 序列：np.arange(a,b,c)  初值a，终值b，步长c，不含终值  
- 序列：np.linspace(a,b,c)    初值a，终值b，个数c，含终值  
- 空数组：np.empty((a,b),np.int) shape&type  
- 零数组：np.zeros((a,b),np.int)  
- 单位阵：np.eye(N,M=None,k=0) 行数N，列数M，k为对角线上移(正)或下移(负)  
- 