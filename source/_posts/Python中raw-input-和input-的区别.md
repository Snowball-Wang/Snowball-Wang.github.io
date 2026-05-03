---
title: Python中raw_input()和input()的区别
date: 2018-01-17 09:41:44
tags:
- 技术日志
- Python
- raw_input()
- input()
categories:
- 技术日志

---

**关键词：** Python  raw_input()  input()

# 0. 前言

之前在学习["笨办法学Python"](https://learnpythonthehardway.org/book/)时使用过`raw_input()`函数来提取用户输入的内容，今天在学习[Python基础教程](https://book.douban.com/subject/4866934/)又遇到了`input()`函数，`input()`函数也可以用来提取用户输入的内容，那么两者有什么差别？本文将对`raw_input()`和`input()`函数的差别进行总结。

<!-- more -->

# 1. 正文

## 1.1 Python2.x版本

我所用的是`Ubuntu16.04`系统，系统自带Python2.7版本，在命令行中敲入`python`命令，进入Python交互命令界面。由于`raw_input()`和`input()`是内置函数，可通过`help`函数来查看其具体用法。

先来看一看`raw_input()`的含义：

```python
>>> help(raw_input)

Help on built-in function raw_input in module __builtin__:

raw_input(...)
    raw_input([prompt]) -> string
    
    Read a string from standard input.  The trailing newline is stripped.
    If the user hits EOF (Unix: Ctl-D, Windows: Ctl-Z+Return), raise EOFError.
    On Unix, GNU readline is used if enabled.  The prompt string, if given,
    is printed without a trailing newline before reading.
```

`raw_input()`的含义是读取用户输入的字符串，并且用户输入的字符串中如果有**换行符（trailing newline）**，那么换行符将会被去除。如果用户在输入过程中输入**EOF（文件结束符）**，将会引发`EOFError`。



再来看一看`input()`的含义：

```python
>>> help(input)

Help on built-in function input in module __builtin__:

input(...)
    input([prompt]) -> value
    
    Equivalent to eval(raw_input(prompt)).
```



可以看到，`input()`函数实际上是通过调用`raw_input()`来实现的，`eval()`函数的功能是将字符串str当成有效的Python表达式来求值并返回计算结果。

这样子还是不能够理解两个函数的差别，下面通过具体例子来理解两者差异：

```python
>>> raw_input_A = raw_input(">")
>abc
>>> input_A = input(">")
>abc
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<string>", line 1, in <module>
NameError: name 'abc' is not defined
>>> input_A = input(">")
>'abc'
>>>
```

可以看出，`raw_input()`和`input()`都可以接收字符串，但`raw_input()`可以接受控制台任何类型的输入，而`input()`则要求输入必须是合法的Python表达式，否则就会引起错误。

```python
>>> raw_input_B = raw_input(">")
>123
>>> type(raw_input_B)
<type 'str'>
>>> input_B = input(">")
>123
>>> type(input_B)
<type 'int'>
```

可以看出，`raw_input()`将所有输入都当做字符串来看待，返回的是str类型。而`input()`在对待纯数字输入时，会把数字当作Python表达式来看待，它的返回类型就不再是str类型，将会变成int或float类型。同时，`input()`函数可以接收合法的Python表达式，如`input(1+1)`结果返回int类型的2。

## 1.2 	Python3.x版本



在Python3.x版本中，旧的`input()`函数功能被**删除**，**新的`input()`函数**的含义**完全相当于**Python2.x中的**`raw_input()`**。也就是说，在Python3.x版本中，不再有`raw_input()`函数，`input()`函数变为Python2.x中的`raw_input()`函数。



**参考链接：**

1. https://www.cnblogs.com/way_testlife/archive/2011/03/29/1999283.html
2. https://stackoverflow.com/questions/4915361/whats-the-difference-between-raw-input-and-input-in-python3-x

