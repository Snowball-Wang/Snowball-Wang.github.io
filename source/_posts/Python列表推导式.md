---
title: Python列表推导式
date: 2018-01-25 16:08:14
tags:
- 技术日志
- Python
- List Comprehension
categories:
- 技术日志

---

**关键字：** Python 列表推导式

# 0. 前言

最近重温Python的基础知识，发现了**列表推导式**（list comprehension）这种创建列表的高级方式，感觉非常有趣。谷歌发现了这篇文章[Python: List Comprehensions](http://www.secnetix.de/olli/Python/list_comprehensions.hawk)，文章作者用非常浅显易懂的语言介绍了**列表推导式**，于是我把文章翻译成中文，方便以后学习和查阅。

<!-- more -->

# 1. 正文

> **注：** 文章中以">>>"和"..."符号开头的行是Python的输入（这些符号都是Python交互式解释器默认的提示符），所有其它的内容都是Python的输出。

Python支持一个名为**"列表推导式"**的概念。它可以像数学家一样，以非常自然容易的方法来构建列表。

下面是常用的描述列表（或是集合、元组、向量）的数学表达式：

$$S = \lbrace x^2 : x \ in \  \lbrace0 ... 9\rbrace \rbrace $$

$$ V = \lbrace1, 2, 4, 8, ... , 2^{12}\rbrace $$

$$M = \lbrace x\ | x \ in \ S \ and \ x \ even \rbrace$$

你可能已经在学校的数学课里学过了上述内容。在Python中，你几乎完全可以像数学家一样写出这些表达式，并且不需要记住任何难懂的语法。

在Python中实现上述数学表达式：

```python
>>> S = [x**2 for x in range(10)]
>>> V = [2**i for i in range(13)]
>>> M = [x for x in S if x % 2 == 0]
>>>
>>> print S; print V; print M
[0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
[1, 2, 4, 8, 16, 32, 64, 128, 256, 512, 1024, 2048, 4096]
[0, 4, 16, 36, 64]
```



我知道你想要了解更复杂的例子:)。接下来我们将用列表推导式来计算质数。有趣的是我们要首先建一个非质数的列表，然后我们使用列表推导式来得到之前列表的"反转列表"，也就是我们想要得到的质数列表。

```python
>>> noprimes = [j for i in range(2, 8) for j in range(i*2, 50, i)]
>>> primes = [x for x in range(2,50) if x not in noprimes]
>>> print primes
[2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37, 41, 43, 47]
```

**注：**你可以在每一个列表推导式中再嵌套列表推导式，上述的例子用一行代码即可完成（不再需要中间变量"noprimes"）。但是，这样的代码容易变得冗长，导致可读性下降，所以不推荐。

当然，列表推导式不仅仅用于数字。列表可以包含任意类型的元素，包括字符串、嵌套列表和函数。你甚至可以在一个列表里混合多种类型的元素。

下面的代码处理一个字符串列表，生成一组列表。每个子列表都包含两个字符串和一个整型数。

```python
>>> words = 'The quick brown fox jumps over the lazy dog'.split()
>>> print words
['The', 'quick', 'brown', 'fox', 'jumps', 'over', 'the', 'lazy', 'dog']
>>> 
>>> stuff = [[w.upper(), w.lower(), len(w)] for w in words]
>>> for i in stuff:
...     print i
... 
['THE', 'the', 3]
['QUICK', 'quick', 5]
['BROWN', 'brown', 5]
['FOX', 'fox', 3]
['JUMPS', 'jumps', 5]
['OVER', 'over', 4]
['THE', 'the', 3]
['LAZY', 'lazy', 4]
['DOG', 'dog', 3]
>>> 
>>> stuff = map(lambda w: [w.upper(), w.lower(), len(w)], words)
>>> for i in stuff:
...     print i
... 
['THE', 'the', 3]
['QUICK', 'quick', 5]
['BROWN', 'brown', 5]
['FOX', 'fox', 3]
['JUMPS', 'jumps', 5]
['OVER', 'over', 4]
['THE', 'the', 3]
['LAZY', 'lazy', 4]
['DOG', 'dog', 3]
```

上述例子说明你完全可以通过`map()`和`lambda`函数做到列表推导式做的事。但是，存在你无法使用`map()`函数只能使用列表推导式的情况，反之亦然。如果两种方法都可以使用，则推荐使用列表推导式，因为它更高效，代码的可读性也更强。

如果构建列表的规则太过复杂，无法用简单的`for`或是`if`语句来表达时，抑或是构建列表的规则会在代码运行时动态修改，那你就无法使用列表推导式。在上述情况下，你最好使用`map()`或者`filter()`函数。当然，你也可以将函数结合列表推导式使用。



# 3. 总结



列表推导式的规范为：**`var = [ out_exp for out_exp in input_list if out_exp ...]`**



**参考链接：**

1. http://www.secnetix.de/olli/Python/list_comprehensions.hawk
2. https://eastlakeside.gitbooks.io/interpy-zh/content/Comprehensions/list-comprehensions.html



