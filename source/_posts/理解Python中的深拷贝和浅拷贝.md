---
layout: '[post]'
title: 理解Python中的深拷贝和浅拷贝
tags:
  - 技术日志
  - Python
categories:
  - 技术日志
date: 2018-11-13 17:10:06
---




**关键词：** Python  深拷贝  浅拷贝



# 0. 前言

之前在参加海康研究院图像算法工程师的面试时，面试官问到了关于Python中深拷贝和浅拷贝的问题。我只是回答出了直接赋值和浅拷贝的差异，并没有涉及到Python如何使用copy模块中的deepcopy()方法来实现深拷贝。这部分的知识明显有漏洞，并且网上有很多说明Python深浅拷贝的文章，本文就是基于自己的理解对Python的深浅拷贝做个总结。



# 1. 概念

* **直接赋值：**变量即为对象的别名（类比于C++的引用）。
* **浅拷贝：**拷贝父对象，不会拷贝对象内部的子对象，可通过copy模块的copy方法实现。
* **深拷贝：**完全拷贝父对象及其子对象，可通过copy模块的deepcopy方法实现。

<!-- more -->

# 2. 实例说明

## 2.1 直接赋值

**代码：**

```python3
snowball = ["Snowball", 22, ["Python", "C", "Shell"]]
wang = snowball
print("var snowball's id: ", id(snowball))
print("snowball's content: ", snowball)
print("each element's id in snowball: ", [id(elem) for elem in snowball])
print("var wang's id: ", id(wang))
print("wang's content: ", wang)
print("each element's id in wang: ", [id(elem) for elem in wang])

print()
print("Modify var snowball....")
snowball[0] = "wang"
snowball[2].append("C++")
print("var snowball's id: ", id(snowball))
print("snowball's content: ", snowball)
print("each element's id in snowball: ", [id(elem) for elem in snowball])
print("var wang's id:  ", id(wang))
print("wang's content: ", wang)
print("each element's id in wang: ", [id(elem) for elem in wang])
```

**运行结果：**

![test1](/理解Python中的深拷贝和浅拷贝/test1.png)

**代码分析：**

* 我首先创建了一个`snowball`变量，`snowball`变量指向一个list对象。从图1.1中我们可以看到所有对象的地址（每次运行显示的地址的值可能不同）。
* 将`snowball`变量赋值给`wang`变量，此时变量`wang`指向`snowball`变量指向的list对象，`wang`变量与`snowball`变量的内存地址也相同，如图1.2所示。可以这样去做理解，Python中对象的赋值都是针对对象的引用，即传递的并不是对象的值，传递的是对象的内存地址。
* 从图1.3可以看出，由于`snowball`变量和`wang`变量都指向同一个list对象，所以针对`snowball`变量的任何修改都会在`wang`变量上体现。

![图1](/理解Python中的深拷贝和浅拷贝/图1.png)

## 2.2 浅拷贝

**代码：**

```python3
import copy

snowball = ["Snowball", 22, ["Python", "C", "Shell"]]
wang = copy.copy(snowball)

print("var snowball's id: ", id(snowball))
print("snowball's content: ", snowball)
print("each element's id in snowball: ", [id(elem) for elem in snowball])
print("var wang's id: ", id(wang))
print("wang's content: ", wang)
print("each element's id in wang: ", [id(elem) for elem in wang])

print()
print("Modify var snowball....")
snowball[0] = "wang"
snowball[2].append("C++")
print("var snowball's id: ", id(snowball))
print("snowball's content: ", snowball)
print("each element's id in snowball: ", [id(elem) for elem in snowball])
print("var wang's id:  ", id(wang))
print("wang's content: ", wang)
print("each element's id in wang: ", [id(elem) for elem in wang])
```

**运行结果：**

![test2](/理解Python中的深拷贝和浅拷贝/test2.png)

**代码分析：**

* 同上述的步骤1相同，我创建一个`snowball`变量，所有对象的内存地址如图2.1所示。
* 通过copy模块中的copy()方法，对`snowball`变量指向的对象进行浅拷贝，然后将浅拷贝生成的新对象赋值给`wang`变量，则`wang`变量此时指向的对象不再是`snowball`变量指向的变量。但浅拷贝只完成父对象的拷贝，也就意味着对于被拷贝对象里的元素，浅拷贝只会使用原始元素的引用，即只对原始元素的内存地址进行拷贝。
* 当对`snowball`变量进行修改时，由于`snowball`变量所指向的list对象的第一个元素是`str`类型，`str`类型为不可变类型，所以新的`str`类型`wang`将会替换掉原来list对象的第一个元素。而原始list对象的最后一个元素是`list`类型，为可变类型，修改操作不会产生新的对象，所以对`snowball`变量的修改结果会在`wang`变量上得到体现。

![图2](/理解Python中的深拷贝和浅拷贝/图2.png)

## 2.3 深拷贝

**代码：**

```python3
import copy

snowball = ["Snowball", 22, ["Python", "C", "Shell"]]
wang = copy.deepcopy(snowball)

print("var snowball's id: ", id(snowball))
print("snowball's content: ", snowball)
print("each element's id in snowball: ", [id(elem) for elem in snowball])
print("var wang's id: ", id(wang))
print("wang's content: ", wang)
print("each element's id in wang: ", [id(elem) for elem in wang])

print()
print("Modify var snowball....")
snowball[0] = "wang"
snowball[2].append("C++")
print("var snowball's id: ", id(snowball))
print("snowball's content: ", snowball)
print("each element's id in snowball: ", [id(elem) for elem in snowball])
print("var wang's id:  ", id(wang))
print("wang's content: ", wang)
print("each element's id in wang: ", [id(elem) for elem in wang])
```

**运行结果：**

![test3](/理解Python中的深拷贝和浅拷贝/test3.png)

**代码分析：**

* 步骤1同上述内容一致。
* 通过copy模块中的deepcopy()方法，对`snowball`所指向的list对象进行深拷贝，将深拷贝生成的新的对象赋值给`wang`变量，同浅拷贝相同的是，此时`snowball`变量和`wang`变量所指向的list对象不是同一个对象，但不同于浅拷贝只复制对象元素的内存地址的方式，深拷贝将对象元素的内容也复制到新生成的对象中，这在图3.2中得到了体现。`snowball`变量指向的list对象的第三个元素的内存地址同`wang`变量指向的list对象的第三个元素的内存地址不同，也就是说在对原始list对象的深拷贝操作中，我们完成对list对象第三个元素列表内容的拷贝，即新开拓了一块内存存放原先list对象的第三个元素的值。
* 同上述浅拷贝一致，`snowball`变量指向的list对象的第一个元素发生了替换。但由于深拷贝复制了list对象第三个元素的值，所以当对`snowball`变量所指向的list对象的第三个元素修改时，这种修改不会在`wang`变量所指向的list对象上体现，也意味着`wang`变量所指向的list对象的第三个元素不会发生改变。



![图3](/理解Python中的深拷贝和浅拷贝/图3.png)



# 3. 总结

* Python中对象的直接赋值是对对象的引用。
* Python的浅拷贝只完成对象元素父对象的拷贝，而深拷贝则完成对象元素父对象及子对象的拷贝。
* 不可变类型（如int，str类型等）不存在拷贝这个概念。



# 参考文献

1.[图解Python深拷贝和浅拷贝](https://www.cnblogs.com/wilber2013/p/4645353.html)

2.[Python 直接赋值、浅拷贝和深度拷贝解析](http://www.runoob.com/w3cnote/python-understanding-dict-copy-shallow-or-deep.html)

