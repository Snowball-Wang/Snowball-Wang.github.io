---
title: 理解Python函数参数*args和**kwargs
date: 2018-04-30 15:53:51
tags:
- 技术日志
- Python
categories:
- 技术日志
- Python
---

![*args和**kwargs](/理解Python函数参数-args和-kwargs/tricks.jpg)

**关键字：** Python `*args`  `**kwargs`



# 0. 前言

上周和室友讨论他在科大讯飞面试的问题时，谈到了Python装饰器的含义和用途。于是乎今天重新复习了一下关于Python装饰器的内容。但在阅读Python装饰器的代码中，函数经常会出现`*args`和`**kwargs`等参数，虽然之前了解这两个参数表示可以输入数量不定的变量的含义，但并没有深入理解其含义和用法。写下此文作为理解`*args`和`**kwargs`的学习笔记。

<!-- more -->

# 1. '*表达式'

在讨论`*args`和`**kwargs`之前，我们需要先介绍一下Python中的**'*表达式'**。

先看以下代码：

**说明：** `>>>`表示是Python运行终端，本文所用的Python版本是3.5.2。

```python
>>> user_info = ('snowball', 'USTC', '183xxxx1966', '188xxxx7604')
>>> name, school, phone_number = user_info
Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
ValueError: too many values to unpack (expected 3)
>>> name, school, *phone_number = user_info
>>> name
'snowball'
>>> school
'USTC'
>>> phone_number
['183xxxx1966', '188xxxx7604']
```

观察上述代码，我们可以看到，当我们用name、school、phone_number三个变量去对含有4个元素的user_info元组进行解包时，就会出现`too many values to unpack`的错误。但是元组中的第3个和第4个元素都是手机号码，我们又不想分别用两个变量对这两个号码进行保存，这时我们就可以通过**'*表达式'**来实现我们的功能。`*phone_number`表示的是一个列表，它将存储元组中剩下的所有元素。即使元组中剩下元素的个数为0，代码也不会报错，因为此时**'*表达式'**所对应的是一个空列表。

同时，**'*表达式'**也可以位于解包代码的第一个位置，举例说明：

```python
>>> *head, end = [0,1,2,3,4,5,6,7,8,9]
>>> head
[0,1,2,3,4,5,6,7,8]
>>> end
9
>>>
>>> a = [0,1,2,3,4,5,6,7,8,9]
>>> head = a[:9]
>>> head
[0, 1, 2, 3, 4, 5, 6, 7, 8]
>>> end = a[-1:]
>>> end
[9]
```

上述代码中，`*head`表示的是包含原列表中前9个元素的列表，变量`end`对应列表中的最后一个元素。在对列表的元素解包的过程中，`*head`一直匹配到下一个解包元素的所对应的原列表的位置的前一个位置。进一步深入的理解，**'*表达式'**的实现机制可和**切片**的实现机制进行类比。**'*表达式'**相当于实现了对原始可迭代对象的切片操作（上述代码中的可迭代对象为列表），只不过**'*表达式'**切片的长度是不定的，它的长度是由其它解包元素的个数来确定的。在上述代码中，`*head`之后只有一个解包变量`end`，所以`*head`的长度即为原始列表的长度减1，可将其类比成取原始列表前9个元素的**切片**操作。



# 2. `*args`和`**kwargs`

理解了**'*表达式'**的含义和用法，再对Python中的`*args`和`**kwargs`这两个魔法变量进行理解，就不会显得那么突兀。首先我们必须了解的是，`*args`和`**kwargs`的写法只是约定俗称的，我们完全可以按照自己的喜好来给参数变量命名，例如`*var`和`**vars`。换句话说，变量中`*`号是不可省略的，但`*`之后的变量名可以自定义。

`*args`和`**kwargs`都主要用于函数定义，表示的是可将不定数量的参数传递给一个函数。这里不定数量的含义是指，我们预先并不清楚，函数的使用者究竟会传递多少个参数给函数。这时，使用`*args`和`**kwargs`将会帮助我们处理这种情况。



## 2.1 `*args`的用法



`*args`用来向函数传递一个**非键值对的可变长度的参数列表**。

```python
def test_var_args(f_arg, *argv):
    print("first normal arg:", f_arg)
    for arg in argv:
        print("another arg through *argv:", arg)
        
test_var_args('snowball', 'emily', 'tom', 'jerry')
```

运行结果：

```python
first normal arg: snowball
another arg through *argv: emily
another arg through *argv: tom
another arg through *argv: jerry
```

通过上述代码，我们可以对`*args`的用法有了较为深刻的理解。



## 2.2 `**kwargs`的用法

首先我们要知道的是，`**kwargs`中的`kw`的含义是**key-word pairs**，也就是键值对。所以`**kwargs`的用法是，向函数传递一个不定长度的**键值对**。如果我们想要我们的函数能够处理**带名字的参数**，我们应该使用`**kwargs`。

举个例子：

```python
>>> def welcome(**kwargs):
	for key, value in kwargs.items():
		print("{0} = {1}".format(key, value))

		
>>> welcome(name="snowball")
name = snowball
>>> a_dict = {"name1":"snowball", "name2":"emily"}
>>> welcome(**a_dict)
name1 = snowball
name2 = emily
>>> 
```

所以，当函数的形参是`**kwargs`时，传入函数的实参必须是**键值对**。

## 2.3 使用`*args`和`**kwargs`调用函数 

我们写一个小函数，代码如下：

```python
>>> def test_args_kwargs(arg1, arg2, arg3):
	print("arg1:", arg1)
	print("arg2:", arg2)
	print("arg3:", arg3)
```

然后我们分别使用`*args`和`**kwargs`来将参数传递给上述函数，代码如下：

```python
>>> args = ("snowball", [1,2,3], 5)
>>> test_args_kwargs(*args)
arg1: snowball
arg2: [1, 2, 3]
arg3: 5
>>> kwargs = {"arg3":3, "arg2":2, "arg1":1}
>>> test_args_kwargs(**kwargs)
arg1: 1
arg2: 2
arg3: 3
```

使用标准参数和`*args`、`**kwargs` 的顺序如下：

```python
some_func(fargs, *args, **kwargs)
```

举个例子：

```python
>>> def test_formal_args_kwargs(fargs, *args, **kwargs):
	print("First argument:", fargs)
	for arg in args:
		print("arg through *args:", arg)
	for key, value in kwargs.items():
		print("{0}: {1}".format(key, value))

		
>>> a = 4
>>> a_list = [1,2,3, "a", [4,5,6]]
>>> a_dict = {"a":1, "b":2, "c":3}
>>> test_formal_args_kwargs(a, *a_list, **a_dict)
First argument: 4
arg through *args: 1
arg through *args: 2
arg through *args: 3
arg through *args: a
arg through *args: [4, 5, 6]
a: 1
b: 2
c: 3
```



# 3. `*args`和`**kwargs`的使用场景

在*Python Cookbook*第三版的第9章的元编程中，涉及到了Python装饰器的相关代码，`*args`和`**kwargs`在装饰器中经常会出现，举例说明：

```python
import time
from functools import wraps

def timethis(func):
    '''
    Decorator that reports the execution time
    '''
    @wraps
    def wrapper(func):
        start = time.time()
        result = func(*args, **kwargs) # use *args and **kwargs to represent the input data
        end = time.time()
        print(func.__name__, end - start)
        return result
    return wrapper

@timethis
def countdown(n):
    '''
    Counts down
    '''
    while n > 0:
        n -= 1
        
def main():
    countdown(10000)
    countdown(10000000)
    
if __name__ == '__main__':
    main()
```

上述中的代码是一个用于计算函数执行时间的装饰器。在装饰器内部的代码里，我们使用语句`result = func(*args, **kwargs)`来表示传入被装饰函数的参数的所有可能的表达形式。这种写法在Python装饰器函数中极其常见。

代码的运行结果为：

```python
countdown 0.001141786553173828
countdown 0.7821342945098877
```





**参考文献：**

1. *Python Cookbook* 第三版
2. http://book.pythontips.com/en/latest/args_and_kwargs.html