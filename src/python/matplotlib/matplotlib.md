## matplotlib

```python
import matplotlib.pyplot as plt

data1 = np.linspace(1,200,2000)
data2 = np.random.randn(2000)
fig = plt.figure()
ax1 = fig.add_subplot(2,2,1)
plt.plot(data1, label='first')
plt.plot(data2,'.', label='second')
ax1.set_xticks([1,2,40])
ax1.legend(loc='best')
ax1.set_title('first plot')
ax1.set_xlabel('index')
ax2 = fig.add_subplot(2,2,2)
```

'-'实线 '--'短划线 '-.'点划线 ':'虚线 '.'点 'v'倒三角 等
color参数 'b'蓝 'g'绿 'r'红 'c'青 'm'品红 'y'黄 'k'黑 'w'白