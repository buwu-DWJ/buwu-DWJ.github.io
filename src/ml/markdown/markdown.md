# Markdown与其支持的的LaTeX  

# Markdown

- 圆点突出：\- +文字
- 序号列表：数字+.
- 链接：\[文本\]\(链接\)  
- 图片：\!\[文本\]\(链接/本地\)  
- 斜体：\*文字\*  
- 加粗：\*\*文字\*\*  
- 粗斜体：\*\*\*文字\*\*\*  
- 分割线：*********或者----------

> 采用\>+文字

几个\#就代表几级标题  

`反引号`  
```python
import numpy as np  
利用三个反引号，后面加上代码使用的语言，如python可实现高亮
```  

#### 表格

```markdown
|  表头   | 表头  |
|  ----  | ----  |
| 单元格  | 单元格 |
| 单元格  | 单元格 |  
``` 
|  表头   | 表头  |
|  ----  | ----  |
| 单元格  | 单元格 |
| 单元格  | 单元格 |
 
还可设置表格的对齐方式  
- \:\- 实现左对齐
- \-\: 实现右对齐
- :-: 实现居中
```markdown
| 左对齐 | 右对齐 | 居中对齐 |
| :-----| ----: | :----: |
| 1 | 1 | 1 |
```
| 左对齐 | 右对齐 | 居中对齐 |
| :-----| ----: | :----: |
| 1 | 1 | 1 |

# markdown中的LaTeX  

支持的写法详情见[katex官方文档](https://katex.org/docs/supported.html)  

#### 多行公式

为公式加编号：\tag{number}  
$$
f(x) = x+1\tag{1}
$$
在VSCODE中必须使用`aligned`环境，如：  
$$
\begin{aligned}
a&=1\\
&=2
\end{aligned}
$$
```markdown
$$
\begin{aligned}
a&=1\\
&=2
\end{aligned}
$$
```  

#### 分段函数

使用`cases`环境
```LaTeX
$$
f(x) = 
\begin{cases}
a, a<1,\\
b, a\geq1.
\end{cases}
$$
效果如下
```
$$
f(x) = 
\begin{cases}
a&, a<1,\\
b&, a\geq1.
\end{cases}
$$

文内链接

```markdown
[能够点击的链接](#name)
<div id="name"></div>
```