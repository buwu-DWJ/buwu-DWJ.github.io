# XPath

XPath (全称：XML Path Language) 即 XML 路径语言，它是一门在 XML 文档中查找信息的语言，最初被用来搜寻 XML 文档，同时它也适用于搜索 HTML 文档。因此，在爬虫过程中可以使用 XPath 来提取相应的数据。

可以将 Xpath 理解为在XML/HTML文档中检索、匹配元素节点的工具。

Xpath 使用路径表达式来选取XML/HTML文档中的节点或者节点集。Xpath 的功能十分强大，它除了提供了简洁的路径表达式外，还提供了100 多个内建函数，包括了处理字符串、数值、日期以及时间的函数。因此 Xpath 路径表达式几乎可以匹配所有的元素节点。

Python 第三方解析库 lxml 对 Xpath 路径表达式提供了良好的支持，能够解析 XML 与 HTML 文档。

## Xpath节点

XPath 提供了多种类型的节点，常用的节点有：元素、属性、文本、注释以及文档节点。如下所示：
```html
<?xml version="1.0" encoding="utf-8"?>
<website>

<site>
  <title lang="zh-CN">website name</title>
  <name>编程帮</name>
  <year>2010</year>
  <address>www.biancheng.net</address>
</site>

</website>
```

上面的 XML 文档中的节点例子：

```html
<website></website> (文档节点)
<name></name> (元素节点)
lang="zh-CN" (属性节点)
```

## 节点关系

XML 文档的节点关系和 HTML 文档相似，同样有父、子、同代、先辈、后代节点。如下所示：

```html
<?xml version="1.0" encoding="utf-8"?>
<website>

<site>
  <title lang="zh-CN">website name</title>
  <name>编程帮</name>
  <year>2010</year>
  <address>www.biancheng.net</address>
</site>

</website>
```

上述示例分析后，会得到如下结果：

```
title name year address 都是 site 的子节点
site 是 title name year address  父节点
title name year address  属于同代节点
title 元素的先辈节点是 site website
website 的后代节点是 site title name year address
```

## Xpath基本语法

### 1) 基本语法使用

Xpath 使用路径表达式在文档中选取节点，下表列出了常用的表达式规则：


| 表达式    | 描述                                                                                      |
| --------- | ----------------------------------------------------------------------------------------- |
| node_name | 选取此节点的所有子节点                                                                    |
| /         | 绝对路径匹配，从根节点选取                                                                |
| //        | 相对路径匹配，从所有节点中查找当前选择的节点，包括子节点和后代节点，其第一个 / 表示根节点 |
| .         | 选取当前节点                                                                              |
| ..        | 选取当前节点的父节点                                                                      |
| @         | 选取属性值，通过属性值选取数据。常用元素属性有 @id 、@name、@type、@class、@tittle、@href |

下面以下述代码为例讲解 Xpath 表达式的基本应用，代码如下所示：

```html
<ul class="BookList">
  <li class="book1" id="book_01" href="http://www.biancheng.net/">
        <p class="name">c语言小白变怪兽</p>
        <p class="model">纸质书</p>
        <p class="price">80元</p>
        <p class="color">红蓝色封装</p>
    </li>

    <li class="book2" id="book_02" href="http://www.biancheng.net/">
        <p class="name">Python入门到精通</p>
        <p class="model">电子书</p>
        <p class="price">45元</p>
        <p class="color">蓝绿色封装</p>
    </li>
</ul>
```

路径表达式以及相应的匹配内容如下：

```
xpath表达式：//li

匹配内容：
c语言小白变怪兽
纸质书
80元
红蓝色封装


Python入门到精通
电子书
45元
蓝绿色封装

xpath表达式：//li/p[@class="name"]
匹配内容：
c语言小白变怪兽
Python入门到精通


xpath表达式：//li/p[@class="model"]
匹配内容：
纸质书
电子书

xpath表达式：//ul/li/@href
匹配内容：
http://www.biancheng.net/
http://www.biancheng.net/

xpath表达式：//ul/li
匹配内容：
c语言小白变怪兽
纸质书
80元
红蓝色封装

Python入门到精通
电子书
45元
蓝绿色封装
```

注意：当需要查找某个特定的节点或者选取节点中包含的指定值时需要使用[]方括号。如下所示：

```
xpath表达式：//ul/li[@class="book2"]/p[@class="price"]
匹配结果：45元
```

### 2) xpath通配符

Xpath 表达式的通配符可以用来选取未知的节点元素，基本语法如下：

|通配符|描述说|
|*|匹配任意元素节点|
|@*|匹配任意属性节点|
|node()|匹配任意类型的节点|

示例如下：

```
xpath表达式：//li/*

匹配内容：
c语言小白变怪兽
纸质书
80元
红蓝色封装
Python入门到精通
电子书
45元
蓝绿色封装
```

### 3) 多路径匹配

多个 Xpath 路径表达式可以同时使用，其语法如下：

```
xpath表达式1 | xpath表达式2 | xpath表达式3
```

示例应用：

```
表达式：//ul/li[@class="book2"]/p[@class="price"]|//ul/li/@href

匹配内容：
45元
http://www.biancheng.net/
http://www.biancheng.net/
```

## Xpath内建函数

Xpath 提供 100 多个内建函数，这些函数给我们提供了很多便利，比如实现文本匹配、模糊匹配、以及位置匹配等，下面介绍几个常用的内建函数。

| 函数名称                | xpath表达式示例                                 | 示例说明                                          |
| :---------------------- | :---------------------------------------------- | :------------------------------------------------ |
| text()                  | ./text()                                        | 文本匹配，表示值取当前节点中的文本内容            |
| contains()              | //div[contains(@id,'stu')]                      | 模糊匹配，表示选择 id 中包含“stu”的所有 div 节点  |
| last()                  | //*[@class='web'][last()]                       | 位置匹配，表示选择@class='web'的最后一个节点      |
| position()              | //*[@class='site'][position()<=2]               | 位置匹配，表示选择@class='site'的前两个节点       |
| start-with()            | "//input[start-with(@id,'st')]"                 | 匹配 id 以 st 开头的元素                          |
| ends-with()             | "//input[ends-with(@id,'st')]"                  | 匹配 id 以 st 结尾的元素                          |
| concat(string1,string2) | concat('C语言中文网',.//*[@class='stie']/@href) | C语言中文与标签类别属性为"stie"的 href 地址做拼接 |