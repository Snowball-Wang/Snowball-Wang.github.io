---
title: 理解C语言中的Stringizing操作符和Token Pasting操作符
date: 2019-09-01 13:52:41
tags:
- 技术日志
- C/C++
categories:
- 技术日志
- C/C++
---

**关键词：** C/C++ Stringizing Operator  Token Pasting Operator



# 0.前言

这些天在阅读[VPP](https://wiki.fd.io/view/VPP)源码时，发现VPP的源码在很多宏的定义中使用了诸如`#`和`##`的操作符，这在以前C语言的学习中从未接触过。Google了一下发现`#`叫做**Stringizing Operator**，`##`叫做**Token Pasting Operator**，借以此文做一个对这两个C语言知识点的梳理和总结。

<!-- more -->

# 1. Stringizing操作符

Stringizing操作符`#`的作用，简单来说就是把宏定义中的参数转换成字符串值（string literals）。当宏定义的参数前有一个`#`时，C语言的预处理器（preprocessor）会将参数转换成一个字符串常量。不同于常规的宏参数替代，这个过程中宏参数不会首先进行展开，这个过程就叫做**stringizing**。

举个例子：

```c
#include <stdio.h>
#define sum(xy) printf(#xy " = %f\n", xy)

int main()
{
    float a = 1.1;
    float b = 1.2;
    sum(a+b);
    printf("a+b" " = %f\n", a+b);

    return 0;
}
```

上述代码的输出结果为：

```
a+b = 2.300000
a+b = 2.300000
```

可以看到，**stringizing**操作符的效果和`printf`的效果相同，只是在宏定义中把原来的`a+b`计算表达式转换成了字符串。

在使用**stringizing**操作符时，我们需要注意：

* **stringizing**操作符会删除参数前后包含的空格（leading and trailing white space）。
* 如果传给**stringizing**操作符的参数是宏的话，该宏不会被展开。
* 诸如`,``)`之类的符号不能被**stringized**。

举个例子说明一下上述的注意要点：

```c
#include <stdio.h>

#define print(var) #var
#define GREETING "welcome" 

int main()
{
    char *s = NULL;
    //s = print(,);    //,不能作为操作符
    s = print(",");  
    puts(s);
    puts(print( Testing Program  ));  //前后空格被去除
    puts(print(GREETING));            //此处的GREETING宏不被展开
    puts(GREETING);                   //此处的GREETING宏正常展开
    return 0;
}
```

输出结果为：

```
","                            
Testing Program                                                                         GREETING                                                                              welcome  
```

如果使用注释的`s = print(,)`，程序编译会出错，同上述的第三点吻合。`puts(print( Testing Program  )`输出的结果为`Testing Program`，验证了上述关于前后空格删除的注意点。接下来的两条语句就是比较**stringizing**操作符传入的参数是宏不被展开的情况，输出的结果也验证了第二条注意点。

# 2. Token Pasting操作符

要理解**token pasting**操作符就要首先理解**token**在C语言中的含义。类比于英语句子中的单词，**token**在C语言中就是构成一个程序的最小组成单元，也是C语言编译器能够理解的C中的最小单元。C语言的**token**主要包括以下6种类型：

* C中的关键字（char、struct、if等）
* C中的标识符（即变量的变量名）
* C中的常量（整型常量、字符常量等）
* C中的字符串
* C中的特殊符号（诸如`[]`数组括号、`#`预处理指令符号等）
* C中的操作符（单元操作符、二元操作符、三元操作符）

**token pasting**操作符能够在一个宏定义里将两个**token**合并成一个**token**。换句话说，**token pasting**操作符`##`的作用就是消除**token**的空格然后把**token**中非空格的字符的字符连接在一起。

举个例子：

```c
#include <stdio.h>
#define MY_DEVICE_BASE_ADDR  0x40004000
#define MY_DEVICE_REG_CTRL_OFS  0x20
#define MY_DEVICE_REG_STATUS_OFS  0x30
#define MY_DEVICE_REG_INTR_OFS  0x40

#define REG(name)  (MY_DEVICE_BASE_ADDR + MY_DEVICE_REG_##name##_OFS)

int main()
{
    printf("Example to use ## Pre-Processor!\n");
    printf("*** My-Device Register ***\n");
    printf("Control-Register Addr = 0x%x\n", REG(CTRL));
    printf("Status-Register Addr = 0x%x\n", REG(STATUS));
    printf("Interrupt-Register Addr = 0x%x\n", REG(INTR));
    return 0;
}
```

程序的输出结果是：

```
Example to use ## Pre-Processor!
*** My-Device Register ***
Control-Register Addr = 0x40004020
Status-Register Addr = 0x40004030
Interrupt-Register Addr = 0x40004040
```

可以看到，上述程序利用**token pasting**操作符将不同寄存器的名字对应的值与基值相加，得到相应的结果。我们不用针对三种不同的寄存器定义三个不同的宏。**token pasting**操作符的使用使上述实现既简单也清晰，大大降低了重复编码的操作。

# 3. 总结

本文主要小结了**stringizing**操作符和**token-pasting**操作符的使用方法，并结合代码说明两种方法的实际用途。



# 参考文献

1. https://embetronicx.com/tutorials/p_language/c/stringizing-and-token-pasting-operators-in-c-programming/
2. https://data-flair.training/blogs/tokens-in-c/
3. https://kunaliiita.wordpress.com/2007/09/01/what-is-the-token-pasting-operator-and-stringizing-operator-in-c/

