## python之禅

**print**写法

```python
name = 'ROSE'
country = 'China'
age = 20

print('hi, my name is {}. im from {}, and im {}'.format(name,country,age))

最简单写法
print(f'hi, my name is {name}, im from {country}, and im {age+1}')
```

for 循环时使用 enumerate 可返回两个参数，前一个是 index ，第二个是对应参数

```python
for idx,step in enumerate(range(10))
```

@staticmethod
静态方法, 不强制要求传递参数

@classmethod
类方法, 不需要实例化, 不需要self参数, 但第一个参数需要是表示自身类的cls参数, 可以用来调用类的属性, 类的方法, 实例化对象等

### 类特殊方法

class Test():
    def __init__():
    	pass

    def __enter__():
        '''使用with语句创建示例时会自动运行此方法'''
        pass

    ‘’‘
    with Test() as t:
        pass
    ’‘’

    def __exit__():
        '''使用with语句创建实例, 在结束时自动调用该方法'''
        pass

    def __str__():
        '''可print(实例)'''
        return ‘我是Test类’

    def __setattr__, __getattr__, __getattribute__, __delattr__:
        '''对属性进行操作'''
        pass

    def __call__():
        '''能让把实例化对象直接当做函数来调用'''
        print(1)
    '''
    a = Test()
    in: a()
    out: 1
    '''

    def __contains__, __len__():
        '''类作为容器'''
        pass

# HDFStore`
with pd.HDFStore('iv_hv.h5') as store:
    c = store.keys()

'''matplotlib
':'  点虚线
'-'  实线
'--' 破折线
'-.' 点划线

添加水平垂直线
plt.axhline(y=0,ls=":",c="yellow") 水平直线
plt.axvline(x=4,ls="-",c="green")  垂直直线
'''

