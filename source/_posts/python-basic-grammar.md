---
title: Python基础语法—基础数据类型
date: 2016-04-27 09:40:29
categories: python
tags: [python, grammar, 基础语法]
---
## Python基础语法

#### Python是一个高层次的结合了解释性、编译性、互动性和面向对象的脚本语言。

- Python是一种解释型语言： 这意味着开发过程中没有了编译这个环节。类似于PHP和Perl语言。

- Python是交互式语言： 这意味着，您可以在一个Python提示符，直接互动执行写你的程序。

- Python是面向对象语言: 这意味着Python支持面向对象的风格或代码封装在对象的编程技术。

- Python是初学者的语言：Python对初级程序员而言，是一种伟大的语言，它支持广泛的应用程序开发，从简单的文字处理到WWW浏览器再到游戏。

<!-- more -->

### Python中文编码

Python中编写中文代码需加入# -\*- coding: UTF-8 -*- 或者 #coding=utf-8以指定编码，不然会报编码错误,如下所示

```Python
#coding=utf-8
# -\*- coding: UTF-8 -*-
#!/usr/bin/python
print "你好，世界";
```
### Python保留字符
![](https://static.tmaczhao.cn/images/python/python%E4%BF%9D%E7%95%99%E5%AD%97.png)
![](https://static.tmaczhao.cn/images/python/python%E4%BF%9D%E7%95%99%E5%AD%972.png)


### Python行缩进
Python中没有大括号来控制类、函数以及各种逻辑判断，Python中需要通过缩进写模块，但是所有代码块需要包含相同的缩进量,如下所示代码会发生错误
```python
if True:
    print "Answer"
    print "True"
else:
    print "Answer"
  print "False"
```

### Python空行
函数之间或类的方法之间用空行分隔，表示一段新的代码的开始。类和函数入口之间也用一行空行分隔，以突出函数入口的开始。

空行与代码缩进不同，空行并不是Python语法的一部分。书写时不插入空行，Python解释器运行也不会出错。但是空行的作用在于分隔两段不同功能或含义的代码，便于日后代码的维护或重构。

**空行也是程序代码的一部分。**

### Python变量及数据类型
Python中的变量不需要声明，对变量进行赋值是包含了声明和定义的过程。

Python中存在五种标准的数据类型：
- Numbers（数字）
- String（字符串）
- List（列表）
- Tuple（元组）
- Dictionary（字典）

**数字类型**包含四种不同的数值类型
- int（有符号整型）
- long（长整型[也可以代表八进制和十六进制]）
- float（浮点型）
- complex（复数）

**Python字符串**可以用""、''、"''"三种符号直接赋值，Python中的字符串可以用下标访问，str[0]是第一个字符，str[-1]是最后一个字符,Python中字符串的基础操作如下所示：
```Python
#coding=utf-8
#!/usr/bin/python

str = 'Hello World!'

print str # 输出完整字符串
print str[0] # 输出字符串中的第一个字符
print str[2:5] # 输出字符串中第三个至第五个之间的字符串
print str[2:] # 输出从第三个字符开始的字符串
print str * 2 # 输出字符串两次
print str + "TEST" # 输出连接的字符串
```

**Python列表(List)**是Python使用非常频繁的数据类型，用[]标识
```Python
#coding=utf-8
#!/usr/bin/python

list = [ 'abcd', 786 , 2.23, 'john', 70.2 ]
tinylist = [123, 'john']

print list # 输出完整列表
print list[0] # 输出列表的第一个元素
print list[1:3] # 输出第二个至第三个的元素
print list[2:] # 输出从第三个开始至列表末尾的所有元素
print tinylist * 2 # 输出列表两次
print list + tinylist # 打印组合的列表
```

**Python元祖Tuple**类似于List但是Tuple中的元素不能进行二次赋值，无法更新

**Python字典(dictionary)**是Python中除了List以外最灵活的内置数据结构，列表是有序的对象集合，字典是无序对象集合。

字典使用{}作为标识，字典类似于Java中的map使用key、value存取。

```Python
#coding=utf-8
#!/usr/bin/python

dict = {}
dict['one'] = "This is one"
dict[2] = "This is two"

tinydict = {'name': 'john','code':6734, 'dept': 'sales'}


print dict['one'] # 输出键为'one' 的值
print dict[2] # 输出键为2的值
print tinydict # 输出完整的字典
print tinydict.keys() # 输出所有键
print tinydict.values() # 输出所有值
```

### Python数据类型转换

int(x [,base])   将x转换为一个整数
long(x [,base])  将x转换为一个长整数
float(x)  将x转换到一个浮点数
complex(real [,imag])创建一个复数
str(x)    将对象x转换为字符串
repr(x)   将对象x转换为表达式字符串
eval(str) 用来计算在字符串中的有效Python表达式,并返回一个对象
tuple(s)  将序列s转换为一个元组
list(s)   将序列s转换为一个列表
set(s)    转换为可变集合
dict(d)   创建一个字典。d必须是一个序列 (key,value)元组。
frozenset(s)转换为不可变集合
chr(x)    将一个整数转换为一个字符
unichr(x) 将一个整数转换为Unicode字符
ord(x)    将一个字符转换为它的整数值
hex(x)    将一个整数转换为一个十六进制字符串
oct(x)    将一个整数转换为一个八进制字符串

### Python中的运算符
Python中大部分运算符都与java中类似。但也有一些特殊的运算符

**逻辑运算符** and（与&&） or（或||） not（非！）
**成员运算符** in（在序列中，如某数在list中） not in（不在序列中）
**身份运算符** is（判断两标识符是否引用自同一对象） is not
