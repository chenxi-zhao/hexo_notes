---
title: markdown-基础语法
date: 2016-04-27 10:10:36
categories: tools
tags: [markdown]
---
Markdown 是一种方便记忆、书写的纯文本标记语言，用户可以使用这些标记符号以最小的输入代价生成极富表现力的文档：譬如您正在阅读的这份文档。它使用简单的符号标记不同的标题，分割不同的段落，**粗体** 或者 *斜体* 某些文字，更棒的是，它还可以


### <font color="red">基本语法：</font>

```
# 标题1
## 标题2
### 标题3

- 无序列表1
- 无序列表2
- 无序列表3

1. 有序列表1
2. 有序列表2
5. 顺序错了不用担心
3. 写错的列表，会自动纠正

分隔线
***

*我是斜体*
**我是粗体**

```
**实际效果如下**：
>- 无序列表1
>- 无序列表2
>- 无序列表3
>
>***
>1. 有序列表1
>2. 有序列表2
>5. 顺序错了不用担心
>3. 写错的列表，会自动纠正
>
>*我是斜体*
>**我是粗体**

<!-- more -->

### <font color="red">引用</font>
```
> 引用
> > 嵌套引用

```
> 引用
> > 嵌套引用


### <font color="red">代码高亮</font>
```python
//```python
@requires_authorization
def somefunc(param1='', param2=0):
    '''A docstring'''
    if param1 > param2: # interesting
        print 'Greater'
    return (param2 - param1 + 1) or None

class SomeClass:
    pass
>>> message = '''interpreter
... prompt'''

```

### <font color="red">链接</font>
#### (1)文字链接

```
Inline:
[谷歌](https://www.google.com)
Reference:
[谷歌][google_url]
[google_url]:https://www.google.com
```

>Inline:
>[谷歌](https://www.google.com)
>Reference:
>[谷歌][google_url]

#### (2)图片链接
```
![](http://www.google.rw/images/srpr/logo4w.png)
![][google_url]
[google_url]:http://www.google.rw/images/srpr/logo4w.png
```
![](http://www.google.rw/images/srpr/logo4w.png)
![][google_img_url]



### <font color="red">Mathjax公式表达</font>
```
$ 表示行内公式：

质能守恒方程可以用一个很简洁的方程式 $E=mc^2$ 来表达。

$$ 表示整行公式：

$$\sum_{i=1}^n a_i=0$$
```
$ 表示行内公式：

质能守恒方程可以用一个很简洁的方程式 $E=mc^2$ 来表达。

$$ 表示整行公式：

$$\sum_{i=1}^n a_i=0$$

----------------------------------------------------------

### <font color="red">更多教程请[戳这][markdown]查看</font>



[google_url]:https://www.google.com
[google_img_url]:http://www.google.rw/images/srpr/logo4w.png
[markdown]:https://www.zybuluo.com/mdeditor?url=https://www.zybuluo.com/static/editor/md-help.markdown#cmd-markdown-高阶语法手册

