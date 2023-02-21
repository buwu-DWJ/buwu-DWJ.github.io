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


