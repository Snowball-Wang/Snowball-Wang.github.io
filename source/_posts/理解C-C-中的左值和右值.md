---
title: 理解C/C++中的左值和右值
date: 2018-03-29 15:39:43
tags:
- 技术日志
- C/C++
categories:
- 技术日志
- C/C++
---



# 前言

最近在执行**100天C++ Primer计划**，书中第四章里的左值和右值的概念让我困惑不已。CSDN博客里关于左值和右值的文章介绍的都不够全面，每次看了一些都没有耐心继续再看下去。个人认为这篇[英文博客](https://eli.thegreenplace.net/2011/12/15/understanding-lvalues-and-rvalues-in-c-and-c)把左值和右值的概念说明的非常清晰，且方式也是循循善诱。翻译之作为学习的存档。

<!-- more -->

# 正文

我们在C/C++编程中很少会遇到术语**左值（lvalue）**和**右值（rvalue）**，但是一旦接触到这两个概念，我们很难立即弄清楚他们的含义是什么。最常遇到这两个术语的地方便是编译错误和警告信息。举个例子，用`gcc`编译以下代码：

```c
int foo() {return 2;}

int main()
{
    foo() = 2;
    return 0;
}
```

命令行会显示以下错误信息：

```
test.c: In function ‘main’:
test.c:5:11: error: lvalue required as left operand of assignment
     foo() = 2;
           ^
```

当然，这段代码有些故意写错的成分，你也不会写出这样的代码，但是在错误信息中涉及了**左值**这个在C/C++教程中不经常出现的术语。再举一个例子，这次用`g++`来编译：

```C++
int& foo()
{
    return 2;
}
```

这一次的错误代码显示如下：

```
testcpp.cpp: In function ‘int& foo()’:
testcpp.cpp:3:12: error: invalid initialization of non-const reference of type ‘int&’ from an rvalue of type ‘int’
     return 2;
            ^
```

同样，错误信息中涉及了一个神秘的术语**右值**。那么**左值**和**右值**到底在C和C++中的含义是什么了？这就是这篇文章所要探索的内容。



## 一个简单的定义

这部分的文章将首先呈现一个故意简化版的**左值**和**右值**的定义。接下来的文章内容将对此定义进行详细叙述。

**左值（lvalue locator value）**表示的是占有某可被标识的内存位置的对象（它有地址）。

**右值**的定义是**左值**定义的补集，因为每个表达式不是**左值**就是**右值**。由此，根据上述的**左值**的定义，**右值**即为不表示占有某可被标识的内存位置对象的表达式。

**注：**这里的定义有点绕，简单理解就是**左值**和**右值**构成表达式，且**左值**和**右值**概念上不存在交集。**左值**的表达式在内存中占有位置，**右值**的表达式则不在内存中占有位置。



## 基础的例子

上述定义的术语可能显得比较抽象，现在就让我们来看一些简单的例程。

假定我们定义了一个整型变量，并且赋值给它：

```c
int var;
var = 4;
```

赋值操作期待左值作为它的左操作符，因为`var`是拥有可被标识的内存位置的对象，所以`var`是一个左值。但是，下面的代码就是无效的：

```C
4 = var;       //Error
(var + 1) = 4; //Error
```

常量`4`和表达式`var + 1`都不是左值（所以它们都是右值）。因为它们只是表达式的中间结果，没有相应的内存位置，只是在计算过程中存在临时的寄存器中，所以它们不是左值。因此，对它们进行赋值没有任何语义上的含义，因为没有赋值的空间。

现在我们就能明白第一段代码中的错误信息的含义。`foo`函数返回的是一个右值的临时值。试图给这个临时值赋值显然是错误的，因此当读到`foo() = 2;`这条语句时，编译器便会提示错误，因为它预期的是在赋值语句的左手侧读到一个左值。

也不是所有的对函数调用结果的赋值是无效的。C++的引用使之成为可能：

```C++
int globalvar = 20;
int& foo()
{
    return globalvar;
}

int main()
{
    foo() = 10;
    return 0;
}
```

上述代码中，`foo`函数返回了一个引用，**引用是左值类型**，因此可以被赋值。实际上，C++从函数中返回左值的能力在实现某些操作符重载的机制上有着非常重要的作用。一个常见的例子便是对类的中括号`[]`操作符进行重载，来实现某种查询控制。`std::map`可以这样做：

```c++
std::map<int, float> mymap;
mymap[10] = 5.6;
```

`mymap[10]`赋值有效，是因为`std::map::operator[]`的非const重载返回了一个可以被赋值的引用。



## 可修改的左值

一开始左值是为了C语言定义的，它字面上的意思是"适合在赋值语句左边的值"。一段时间后，当IOS C添加了`const`关键字时，原先的含义必须重新定义。毕竟：

```C++
const int a = 10; //'a'是个左值
a = 10;           //但是'a'不能够被赋值
```

由此必须添加对左值的定义的改良。不是所有的左值都能够被赋值。那些可以被赋值的左值被称为**可修改的左值**。正式来说，C99标准定义可修改的左值为：

>一个左值没有数组类型，没有[不完整类型](https://msdn.microsoft.com/en-us/library/200xfxh6.aspx)，没有const限定类型，而且如果它是一个结构体或者联合体，没有任何一个成员（递归的包括所有集合和联合的组成元素）含有const限定类型。



## 左值和右值的转换

通常来说，对象之间的运算，对象是以右值的形式参与的。例如，二元加法运算符`+`两边的参数以右值传入，计算的结果返回一个右值。

```C++
int a = 1;     //a是一个左值
int b = 2;     //b是一个左值
int c = a + b; //+需要右值，因此a和b被转换成右值，同时返回一个右值
```

正如之前我们所见，`a`和`b`都是左值。因此，在第三行中，`a`和`b`经历了隐式的**左值转右值**的变换。所有不是数组类型、函数类型或者定义不完整类型的左值，都可以被转换成右值。

那么可以将右值转换成左值嘛？当然不行！这会违背左值定义的本质。

但是这不意味这左值不能通过显式的方式从右值中产生。例如，单目`*`（解引用）运算符把右值作为参数但是返回的结果是左值。考虑以下有效代码：

```C++
int arr[] = {1, 2};
int* p = &arr[0];
*(p + 1) = 10;   //OK：p + 1是个右值，但是*(p + 1)是个左值
```

相反地，单目地址运算符`&`把左值作为参数生成的是个右值。

```C++
int var = 10;
int* bad_addr = &(var + 1); //ERROR:单目运算符要求参数是左值
int* addr = &var;           //OK:var是个左值
&var = 40;                  //ERROR:赋值语句的左操作符要求的是左值
```

`&`符号在C++中有着别的用途--它能够定义引用类型。这种引用类型被称为"左值引用"。非const左值引用不能够使用右值进行赋值，因为右值转左值的变换是无效的。

```C++
std::string& sref = std::string(); //ERROR: 从右值类型'std::string'对非const引用类
                                   //型'std::string'的初始化是无效的
```

常量左值引用能够被赋值右值。因为它们是常量，无法通过引用来修改值，所以也就不存在修改右值的问题。所以我们在C++中经常使用const引用作为函数的参数类型，由此可以减少不必要的临时对象的创建和复制。



## 带CV限定符（CV-qualified）的右值

如果我们仔细阅读C++标准里关于左值转右值的讨论，我们注意到它是这么说的：

> 类型T的左值（非函数、非数组类型）可以被转换成一个右值。[...]如果T不是一个类（class）类型，转换后的右值的类型便是不带CV限定符（cv-unqualified）的T类型，否则转换后的右值类型为T。

什么叫做不带CV限定符？CV限定符是指**const**和**volatile**关键字。

> 每个不带CV限定符的完整或者不完整的对象类型，亦或者是void类型，都有三种相对应的CV限定符的类型：**const限定**，**volatile限定**和**const-volatile限定**。[...]带CV限定符的类型和不带CV限定符的类型是截然不同的类型，但是它们依旧有着相同的表示方式和对齐要求。

但是上述的与右值又有什么关系？在C语言中，右值永远不会拥有CV限定符的类型。只有左值才有CV限定符的类型。在C++中，类的右值可以拥有CV限定符的类型，但是诸如`int`的内置类型是不带CV限定符的。举个例子：

```c++
#include <iostream>

class A {
public:
    void foo() const { std::cout << "A::foo() const\n";}
    void foo() { std::cout << "A::foo()\n";}
};

A bar() { return A(); }
const A cbar() { return A(); }

int main()
{
    bar().foo();  // calls foo
    cbar().foo(); // calls foo const
}
```

在`main`中的第二个函数调用，调用了`A`的`foo() const`方法。由于`cbar`返回的类型是`const A`，与`A`完全不同。这就是之前引用内容的最后一句话的含义。同时我们也注意到`cbar`的返回值类型是个右值。因此这就是个实际应用中带有CV限定符的右值的例子。



## 右值引用（C++11）

右值引用和相关的**move**语义是C++11标准中新引入的最强的特性之一。对此特性的详细说明超出了本文的范围，但是我依然会举一些简单的例子，这些例子将展示对左值和右值概念的理解会帮助我们加深对C++中一些重要语言概念的理解能力。

举个例子，我们看一下一个简单的动态"整型vector"的具体实现。在这里我只显示了相关方法：

```c++
class Intvec
{
public:
    explicit Intvec(size_t num = 0)
        : m_size(num), m_data(new int[m_size])
    {
        log("constructor");
    }
    
    ~Intvec()
    {
        log("destructor");
        if (m_data) {
            delete[] m_data;
            m_data = 0;
        }
    }
    
    Intvec(const Intvec& other)
        : m_size(other.m_size), m_data(new int[m_size])
    {
        log("copy constructor");
        for (size_t i = 0; i < m_size; ++i)
            m_data[i] = other.m_data[i];
    }
    
    Intvec& operator=(const Intvec& other)
    {
        log("copy assignment operator");
        Intvec tmp(other);
        std::swap(m_size, tmp.m_size);
        std::swap(m_data, tmp.m_data);
        return *this;
    }
private:
    void log(const char* msg)
    {
        cout << "[" << this << "]" << msg << "\n";
    }
    
    size_t m_size;
    int* m_data;
};
```

上述代码中，我们定义了构造函数、析构函数、拷贝构造函数和拷贝复制运算符，所有的定义中都加入了logging函数来明确被调用时的身份。

让我们来运行一些简单的代码，来实现将`v1`的内容复制到`v2`中。

```c++
Intvec v1(20);
Intvec v2;

cout << "assigning lvalue...\n";
v2 = v1;
cout << "ended assigning lvalue...\n";
```

打印出来的结果是：

```
assigning lavlue...
[0x28fef8] copy assignment operator
[0x28fec8] copy constructor 
[0x28fec8] destructor
ended assigning lvalue...
```

上述结果能够说得通，表明了在`operator=`里的运行过程究竟是什么样的。但是假设我们想要把右值赋给`v2`：

```c++
cout << "assigning rvalue...\n";
v2 = Intvec(33);
cout << "ended assigning rvalue...\n";
```

尽管在这里我赋的值是一个新构造的vector，但这个例子想对`v2`赋值的案例更加通用化，即新建某个临时右值，然后将它赋值给`v2`。得到的结果如下所示：

```
assigning rvalue...
[0x28ff08] constructor
[0x28fef8] copy assignment operator
[0x28fec8] copy constructor
[0x28fec8] destructor
[0x28ff08] destructor
ended assigning rvalues...
```

哇，这看上去工作量很大。尤其是有多余的一对用来创建和销毁临时对象的构造函数和析构函数。这种方式很不友好，因为在拷贝赋值运算符中，创建和销毁了另一个临时拷贝。这纯粹是多余无用的工作。

但在C++11中，上述的情况不在存在。C++11提出了右值引用这个概念，通过它，我们可以实现"move语义"和"move赋值运算符"。让我们对`Intvec`重载另一个`operator=`：

```C++
Intvect& operator=(Intvec&& other)
{
    log("move assignment operator");
    std::swap(m_size, other.m_size);
    std::swap(m_data, other.m_data);
    return *this;
}
```

`&&`语法就是新的**右值引用**。它实现对一个即将在调用后销毁的右值的引用。我们可以通过这种方式来"偷走"右值里的内容，反正这些也不再需要用到这些右值。结果如下显示：

```
assigning rvalue...
[0x28ff08] constructor
[0x28fef8] move assignment operator
[0x28ff08] destructor
ended assigning rvalue...
```

这里显示的是，当`v2`被一个右值赋值时，新的move赋值运算符便被调用。创建`Intvec(33)`的临时对象仍需调用构造和析构函数，但是在赋值运算符里，我们不再需要新的临时对象了。运算符只是把自己的内容和右值的内部缓冲区交换了一下，因此右值的析构函数将会释放掉我们对象中自己不再使用的缓冲区，整个操作干净明了。

我不得不再说明一遍，上述的例子仅仅是move语义和右值引用的冰山一角。你很容易猜到，这是个有着各种各样特殊情况的及其复杂的主题。我在这篇文章中想要说明的是C++中左值和右值不同之处的有趣的应用。编译器显然知道什么时候实体是个右值，并且能在编译时调用相应正确的构造函数。



## 结论

我们可以在不考虑左值和右值的情况下编写C++代码，仅仅把左值和右值看作是特定错误信息里的编译器的行话。但是，正如这篇文章想要说明的那样，对此主题的深刻理解，能够帮助我们对特定C++代码的构造有着更深的认识，同时也能使C++语言专家之间的专业讨论更加清楚明白。

同时，在新的C++里，由于C++11对右值引用和move语义的介绍，这个主题就显得更为重要。想要完全理解C++的这个新特性，扎实的理解左值和右值就变得极其重要。



## 参考文献

1. https://eli.thegreenplace.net/2011/12/15/understanding-lvalues-and-rvalues-in-c-and-c#id6
2. https://segmentfault.com/a/1190000003793498

