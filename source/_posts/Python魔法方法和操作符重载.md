---
title: Python魔法方法和操作符重载
date: 2018-02-02 00:12:06
tags:
- 技术日志
- Python
- Magic Method
- Operator Overloading
categories:
- 技术日志
- Python

---

![魔法方法](/Python魔法方法和操作符重载/marvin_the_magician.png)

**关键词：** Python 魔法方法 操作符重载

# 0.前言

在阅读Python的代码时，经常会看到以双下划线`__`包裹起来的方法，最常见的就是在Python的类里的`__init__`方法。这些方法被称之为**魔法方法（Magic Method）**。网上看了很多介绍Python**魔法方法**的文章，有的写得太过啰嗦，有的写得太过深奥。我个人觉得[这篇文章](https://www.python-course.eu/python3_magic_methods.php)写得非常简洁明了，于是将之翻译，稍微对原文内容做了一些修改，算是Python**魔法方法**相关知识的总结和归档。

<!-- more -->

# 1.介绍

所谓的Python魔法方法与真正的魔法并没有关系。这种方法的语法很令人费解，如：在方法名的开始和结尾都要加上两道下划线。你也很难探讨它们，如何口头表达一个名字为`__init__`的方法？可能最好的口头表达方法就是"前后双下划init方法"，就是这样听起来也很难受。

那么关于`__init__`方法有何神奇之处？答案就是，你不用直接调用它，调用的实现机制是在后台完成。当你用语句`X = A()`创建一个A类的x实例时，Python会自动调用`__new__`和`__init__`方法。

关于操作符重载，我们可以使用`+`来进行加法运算、合并字符串或列表。

```python
>>> 4 + 5
9
>>> 3.8 + 9
12.8
>>> "Peter" + " " + "Pan"
'Peter Pan'
>>> [3, 6, 8] + [7, 11, 13]
[3, 6, 8, 7, 11, 13]
>>>
```

我们甚至可以在自己的类中重载`+`操作符乃至其它操作符。想要实现上述功能，我们就需要理解操作符重载的背后机制。每一个操作符号都有一个魔法方法（也称特殊方法）。`+`号的魔法方法是`__add__`方法，`-`号的魔法方法是`__sub__`方法。下面第2节表格显示的是全部操作符的魔法方法。

操作符重载的工作机制如下：如果我们有一个`x + y`的表达式，x是类K的实例，那么Python将会检查类K的定义。如果K有`__add__`方法，那么`x + y`将会调用`x.__add__(y)`，否则我们将会得到一个错误信息。

```python
>>> class K():
...     print "This is class K."
... 
This is class K.
>>> A = K()
>>> B = K()
>>> A + B
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: unsupported operand type(s) for +: 'instance' and 'instance'


>>> class M():
...     def __add__(self, other):
...         print "Hello, World!"
... 
>>> A = M()
>>> B = M()
>>> A + B
Hello, World!
```



# 2. 魔法方法总览

**二进制操作符**

|  操作符   |                   方法                   |
| :----: | :------------------------------------: |
|  `+`   |     `object.__add__(self, other)`      |
|  `-`   |     `object.__sub__(self, other)`      |
|  `*`   |     `object.__mul__(self, other)`      |
|  `//`  |   `object.__floordiv__(self, other)`   |
|  `/`   |   `object.__truediv__(self, other)`    |
|  `%`   |     `object.__mod__(self, other)`      |
|  `**`  | `object.__pow__(self, other[,modulo])` |
|  `<<`  |    `object.__lshift__(self, other)`    |
|  `>>`  |    `object.__rshift__(self, other)`    |
|  `&`   |     `object.__and__(self, other)`      |
|  `^`   |     `object.__xor__(self, other)`      |
| &#x7c; |      `object.__or__(self, other)`      |

**增强赋值操作符**

|   操作符    |                   方法                    |
| :------: | :-------------------------------------: |
|   `+=`   |     `object.__iadd__(self, other)`      |
|   `-=`   |     `object.__isub__(self, other)`      |
|   `*=`   |     `object.__imul__(self, other)`      |
|   `/=`   |     `object.__idiv__(self, other)`      |
|  `//=`   |   `object.__ifloordiv__(self, other)`   |
|   `%=`   |     `object.__imod__(self, other)`      |
|  `**=`   | `object.__ipow__(self, other[,modulo])` |
|  `<<=`   |    `object.__lshift__(self, other)`     |
|  `>>=`   |    `object.__rshift__(self, other)`     |
|   `&=`   |     `object.__iand__(self, other)`      |
| &#x7c; = |      `object.__ior__(self, other)`      |
**注：**在Markdown表格里输入`|`会出现很多问题，上述表格中的`|=`的中间并没有空格。

**一元操作符**

|     操作符     |             方法             |
| :---------: | :------------------------: |
|     `-`     |   `object.__neg__(self)`   |
|     `+`     |   `object.__pos__(self)`   |
|   `abs()`   |   `object.__abs__(self)`   |
|     `~`     | `object.__invert__(self)`  |
| `complex()` | `object.__complex__(self)` |
|   `int()`   |   `object.__int__(self)`   |
|  `long()`   |  `object.__long__(self)`   |
|  `float()`  |  `object.__float__(self)`  |
|   `oct()`   |   `object.__oct__(self)`   |
|   `hex()`   |   `object.__hex__(self)`   |

**比较操作符**

| 操作符  |              方法              |
| :--: | :--------------------------: |
| `<`  | `object.__lt__(self, other)` |
| `<=` | `object.__le__(self, other)` |
| `==` | `object.__eq__(self, other)` |
| `!=` | `object.__ne__(self, other)` |
| `>=` | `object.__ge__(self, other)` |
| `>`  | `object.__gt__(self, other)` |

# 3. 例程：Length类

在接下来的`Length`类里，我们将展示如何在自己的类中重载`+`操作符。要完成上述功能，我们要重载`__add__`方法。我们的类中也同时包含`__str__`和`__repr__`方法。`Length`类的实例也包含长度或距离信息。一个实例的属性包含`self.value`和`self.unit`。

`Length`类允许我们计算混合单元的表达式，如下例所示：

`2.56 m + 3 yd + 7.8 in + 7.03 cm`

`Length`类可以如下所示的代码所使用：

```python
>>> from unit_conversions import Length
>>> L = Length
>>> print(L(2.56, "m") + L(3, "yd") + L(7.8, "in") + L(7.03, "cm"))
5.57162
>>>
```

`Length`类的代码：

```python
class Length():
    __metric = {'mm':0.001, 'cm':0.01, 'm':1, 'km':1000,
                'in':0.0254, 'ft':0.3048, 'yd':0.9144, 'mi':1609.344}

    def __init__(self, value, unit="m"):
        self.value = value
        self.unit = unit

    def Converse2Metres(self):
        return self.value * Length.__metric[self.unit]

    def __add__(self, other):
        l = self.Converse2Metres() + other.Converse2Metres()
        return Length(l / Length.__metric[self.unit], self.unit)

    def __str__(self):
        return str(self.Converse2Metres())

    def __repr__(self):
        return "Length(" + str(self.value) + ", '" + self.unit + "')"


if __name__ == "__main__":
    x = Length(4)
    print(x)
    y = eval(repr(x))

    z = Length(4.5, "yd") + Length(1)
    print(repr(z))
    print(z)
```

**注：**补充关于Python中的前缀为单下划线、结尾为单下划线、前缀为双下划线、前缀和结尾为双下划线的各自含义，参照[stackoverflow上的解答](https://stackoverflow.com/questions/8689964/why-do-some-functions-have-underscores-before-and-after-the-function-name/8689983#8689983)。

* **_single_leading_underscore:** 以单下划线作为前缀的变量或者函数被默认当作是内部变量和函数。如果使用`from a_module import *`导入时，这部分变量和函数不会被导入。不过如果使用`import a_module`导入模块，仍可通过`a_module._some_var`进行访问。
* **single_trailing_underscore_：**避免与Python关键字冲突的一种习惯写法，举例：`Tkinter.Toplevel(master, class_='ClassName')`。
* **__double_leading_underscore：**以双下划线作前缀的名称对解释器有特定含义。Python会改写这些名称，以免与子类中定义的名称产生冲突。任何`__method`这种形式的标识符，都会在文本上被替换成`_classname__method`，其中`classname`是当前类名，并带上一个下划线作为前缀，具体理解可参考[例子](https://segmentfault.com/a/1190000002611411) 。
* **__double_leading_and_trailing_underscore__：**Python的魔法方法，但也是一种惯例，确保Python系统中的名称不会和用户自定义的名称发生冲突的方式。


如果我们运行程序，将会得到以下输出结果：

```
4
Length(5.593613298337708, 'yd')
5.1148
```

我们可以使用`__iadd__`方法来实现更多的功能：

```python
def __iadd__(self, other):
    l = self.Converse2Metres() + other.Converse2Metres()
    self.value = l / Length.__metric[self.unit]
    return self
```

现在我们可以写下以下程序：

```python
x += Length(1)
x += Length(4, "yd")
```

在上述的例子当中，我们通过语句`x += Length(1)`来实现加上1米的功能。可以肯定地说，你一定会同意`x += 1`这种写法更加方便。对于`Length(5, "yd") + 4.8`这类的表达式，我们也想实现类似的功能。如果有人使用整型数或者浮点型数，我们的`Length`类将会自动将其转换为单位是"metre"的`Length`对象。使用`__add__`和`__iadd__`方法来完成上述功能很简单，我们只需要检查参数"other"的类型即可。

```python
def __add__(self, other):
    if type(other) == int or type(other) == float:
        l = self.Converse2Metres() + other
    else:
        l = self.Converse2Metres() + other.Converse2Metres()
    return Length(l / Length.__metric[self.unit], self.unit)

def __iadd__(self, other):
    if type(other) == int or type(other) == float:
        l = self.Converse2Metres() + other
    else:
        l = self.Converse2Metres() + other.Converse2Metres()
    self.value = l / Length.__metric[self.unit]
    return self
```



如果一个人从右边加上整数或者浮点数，他也很有可能从左边执行相同的操作。让我们尝试一下：

```python
>>> from unit_conversions import Length
>>> x = Length(3, "yd") + 5
>>> x = 5 + Length(3, "yd")
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: unsupported operand type(s) for +: 'int' and 'Length'
>>> 
```

当然，表达式的左边必须是"Length"类，否则Python将会试图从`int`类型里使用`__add__`方法，这样就无法将`Length`对象作为第二个参数进行处理。

对于此类问题Python也提供了一套解决方案。这就是`__radd__`方法。它的工作原理是这样的：Python试图求得表达式`5 + Length(3, "yd")`的值。首先表达式将调用`int.__add__(5, Length(3, 'yd'))`，从而引发一个异常。然后表达式将试图调用`Length.__radd__(Length(3, "yd"), 5)`。很容易想象出`__radd__`的实现同`__add__`很类似。

```python
def __radd__(self, other):
    if type(other) == int or type(other) == float:
        l = self.Converse2Metres() + other
    else:
        l = self.Converse2Metres() + other.Converse2Metres()
    return Length(l / Length.__metric[self.unit], self.unit)
```

更为明智的做法是在`__radd__`方法中使用`__add__`方法：

```python
def __radd__(self, other):
    return Length.__add__(self, other)
```

下面的图标解释了方法`__add__`和`__radd__`之间的关系：

![radd和add的关系](/Python魔法方法和操作符重载/operator_overloading__radd__.png)



# 4. 标准类作为基类

我们有可能使用诸如`int`，`float`，`dict`或者是`list`的基类作为标准类。

我们将`list`列表类型新填一个`push`方法：

```python
class Plist(list):
    def __init__(self, l):
        list.__init__(self, l)
    
    def push(self, item):
        self.append(item)
        
if __name__ == "__main__":
    x = Plist([3,4])
    x.push(47)
    print(x)
```



# 5. 练习

**问题：**写一个类似于之前定义的`Length`类的名称为`Ccy`的类。`Ccy`类应该包含不同货币的值，诸如欧元、英镑、美元等。一个实例应该包含金额数量和金额单位。设计的类必须能够通过下列程序描述：

```python
>>> from currencies import Ccy
>>> v1 = Ccy(23.43, "EUR")
>>> v2 = Ccy(19.97, "USD")
>>> print(v1 + v2)
39.68 EUR
>>> print(v2 + v1)
48.76 USD
>>> print(v1 + 3) # an int or a float is considered to be a EUR value
26.43 EUR
>>> print(3 + v1)
26.43 EUR
>>> 
```

**注：**原文数据有误。

**我的解决方法：**

```python
class Ccy:

    __unit = {"EUR": 1.0, "USD": 1.2288, "GBD": 0.8857}

    def __init__(self, value, unit = "EUR"):
        self.value = value
        self.unit = unit

    def converse2eur(self):
        return self.value / Ccy.__unit[self.unit]

    def __add__(self, other):
        if type(other) == int or type(other) == float:
            l = self.converse2eur() + other
        else:
            l = self.converse2eur() + other.converse2eur()
        x = l * Ccy.__unit[self.unit]

        return Ccy(x, self.unit)

    def __radd__(self, other):
        return Ccy.__add__(self, other)

    def __str__(self):
        return "{0:5.2f}".format(self.value) + " " + self.unit
```





**参考链接：**

1. https://www.python-course.eu/python3_magic_methods.php
2. https://segmentfault.com/a/1190000002611411
3. https://stackoverflow.com/questions/1301346/what-is-the-meaning-of-a-single-and-a-double-underscore-before-an-object-name
4. https://ask.helplib.com/python/post_166801
