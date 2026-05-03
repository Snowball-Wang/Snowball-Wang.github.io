---
title: CS4 Challenge之MIT-6.087
date: 2021-02-14 11:34:14
tags:
- 日常规划
- 自我学习
- MIT 6.087
categories:
- CS4挑战
---

**关键词**：CS4挑战  MIT-6.087



# 0. 前言

今年过年没有回家，留在了上海。趁着这还有三四天的假期，把遗留的MIT-6.087的*Pratical Programming in C*的部分看完，并把之前看过的部分复习巩固一遍，也算是给[新版CS4挑战](http://wangjieqiang.com/2020/04/12/My-CS4-Challenge-Version1/)起个头。看了一下这个[Repo](https://github.com/Snowball-Wang/MIT_6.087_Practical_Programming_in_C)最早的commit的记录是2018年12月份，可以说是历史遗留问题了。借着再巩固一下C语言基本知识、数据结构以及Unix相关概念的想法，以本文作为契机对MIT-6.087课程进行回顾和总结。

# 1. 课程介绍

## 1.1 课程简介

*MIT 6.087 Practical Programming in C*是MIT 2010年的IAP(Independent Activities Period)课程。MIT的IAP课程总长四周，有点类似于上海大学的夏季小学期。但是MIT-6.087的内容还是很丰富的，感觉四周时间学习完所有的内容还是非常有挑战性的。直接摘录课程主页的课程介绍来看看这门课的内容。

>This course provides a thorough introduction to the C programming language, the workhorse of the UNIX operating system and lingua franca of embedded processors and micro-controllers. 
>
>The first two weeks will cover basic syntax and grammer, and expose students to practical programming techniques. The remaining lectures will focus on more advanced concepts, such as dynamic memory allocation, concurrency and synchronization, UNIX signals and process control, library development and usage. Daily programming assignments and weekly laboratory exercises are required. Knowledge of C is highly marketable for summer internships, UROPs, and full-time positions in software and embedded systems development.

正如课程介绍所说，这门课程在我看来很不错的地方在于不仅仅介绍了C语言的基础用法和一些基本的数据结构，而且还在后面的章节涉及到了很多Unix的高级概念。而这些概念很多是计算机体系或者是操作系统的内容，这也是当初为什么我看完前面的C基础就没法继续推进下去的原因😂，自己当时对计算机体系和操作系统一无所知。

<!-- more -->

## 1.2 课程资源

课程主页在[这里](https://ocw.mit.edu/courses/electrical-engineering-and-computer-science/6-087-practical-programming-in-c-january-iap-2010/)，包括了课程的讲义、练习和项目说明。遗憾的是，这个课程并没有相关视频，所以我的学习方式是看课程PPT，不懂的地方谷歌或者在油管上找相关视频。课程推荐的书籍是经典的[K&R](https://book.douban.com/subject/1139336/)，可以作为辅助参考资料。

课程一共有7个*Problem set*和2个*Lab*。每个*Problem set*都有官方的答案和解释，每个*Project*就没有官方的参考了。同样，我也把自己的Solution放在了[Github Repo](https://github.com/Snowball-Wang/MIT_6.087_Practical_Programming_in_C)，以供参考。

# 2. 课程总结

## 2.1 课程概述

本课程首先是介绍了C语言的基本语法，然后结合链表、二叉树、哈希表等数据结构，用C语言解决实际的问题。在此过程中，还引入了库开发、动态内存分配、多线程编程、进程间通信等系统编程的概念，虽然很多内容也是浅浅地介绍一下。即便如此，这门课对我的帮助依旧很大。正如其英文名`Practical Programming in C`表达的一样，认真地学完这门课，可以掌握C语言的基本语法和常用的数据结构的C实现，并对操作系统的相关概念有了入门的了解，同时能够体会系统编程的趣味和魅力（debug的痛并快乐着🤣）。

## 2.2 课程详述

**Lecture 1：**主要介绍了如何编写、编译和调试C程序，同时覆盖了C语言的基本概念。本节主要分为三个部分。首先简要地介绍了C语言的历史，包括C语言的诞生、C语言的版本演进、C语言的应用范围、C语言同其它类型计算机语言的差别，最关键的就是明确C语言是底层语言（low-level language）。第二部分介绍如何编写C程序，包括编辑器、编译器（课程用的是gcc）的选择，了解gcc相应的编译选项，C程序调试工具gdb的使用，以及内存检查工具valgrind。第三部分用一个具体的*Hello World*例子来介绍C语言的基本语法。包括一个**.c**文件结构所包含的内容，如何注释，如何使用宏，C语言算数运算规则，C语言函数原型，如何声明、初始化变量，变量的作用域，如何使用I/O函数打印字符串等等。在宏的部分，还着重介绍了预处理宏在条件编译的使用，包括`#if,#ifdef,#ifndef,#else,#elif,#endif,#pragma,#error,#warning,#undef`等等。

**Lecture 2：**主要介绍了C语言的变量、数据类型以及运算符。首先说明了数据类型、运算符、表达式、变量的相关概念，了解C是一个[弱类型静态类型语言](https://www.zhihu.com/question/19918532)。接着介绍了C语言变量的命名规范和C包含的数据类型。在介绍C数据类型中，还插入介绍了一下大端（Big Endian）和小端（Little Endian）的[区别](https://www.cnblogs.com/wxdlut/p/9200720.html)。除了C常规的数据结构（int, float, struct, union），还对C常量（Constants）进行了介绍，包括字符类型，字符串类型，[C枚举](https://www.geeksforgeeks.org/enumeration-enum-c/)（enumeration）。最后是C中的运算符的使用，包括算数运算符（+, -, *, /, %），关系运算符（>, >=, <, <=, ==, !=），逻辑运算符（&&, ||, !），自增自减运算符（++, --），位运算符（&, |, ^, <<, >>），三元条件运算符（?:）。其中关于逻辑运算符要注意其中的短路机制，即条件表达式一旦被满足就不会再执行后续的表达式。关于自增自减表达式要注意类似`y=x++`与`y=++x`的区别。本节的结尾还提及C类型转换和运算符优先级。

**Lecture 3：**主要介绍了C语言的程序控制流、函数及模块化思想和变量作用域。在程序控制流部分，说明了C语言条件语句`if...else if... else`和`switch-case`使用，以及C语言中的几种循环语句的表达，包括`while`循环、`do-while`循环和`for`循环。同时介绍了`break`和`continue`关键字在循环语句中的作用。在函数及模块化思想部分，主要通过一个具体的例子，来说明如何用C语言通过模块化的思想构造函数来解决实际问题，在`euclid.h`文件中自己额外引申关于**include guard**的问题。在最后一部分介绍了关于C变量作用域的话题，包括`extern`、`static`和`register`关键词对变量作用域范围的影响。

**Lecture 4：**进一步介绍了C语言的程序控制流以及C语言的I/O控制。在C语言的程序控制流部分，主要说明了`goto`语句的用法以及它在C语言**error handling**如何使用。在I/O控制部分，首先介绍了最简单的标准输入输出`putchar()`和`getchar()`函数，然后详细说明了一下格式化输出`printf()`函数的使用，通过具体的例子来理解其格式化的规则`%[flags][width][.precision][length]<type>`。中间还提了一下`strcmp()`函数的用法。对应`printf()`函数也介绍了`scanf()`函数，对应的还有针对字符串类型的输入输出`sprintf()`与`sscanf()`函数，文件输入输出`fprintf()`与`fscanf()`函数等。也可以通过`fgets()`和`fputs()`函数读写文件一行的数据。最后说明了一下C语言如何在类Unix系统环境下使用命令命令行参数，`int main(int argc, char *argv)`中的`argc`表示的就是参数的数量，`argv[]`就是每一个参数字符串数组的指针。

**Lecture 5：**主要引入了C语言指针的概念，介绍了操作字符串的相关函数，最后简单说明了几种排序算法。在引入指针的概念之前，说明了一下虚拟内存和物理内存的区别。也借用之前模块化函数的例子，来说明函数的参数是指针的效果。指针的算数运算不同于常规的数值算数运算，指针的加减运算等于其加减值乘以指针对应的数据类型。在操作字符串函数的部分，提及了相关常用的字符串函数，包括`strcpy`、`strcmp`、`strcat`等等。

**Lecture 6：**介绍了C语言中三种用户自定义的数据类型，并引入动态内存分配的概念，介绍了链表和二叉树两种数据结构的简单实现。C语言中的用户自定义的复杂数据类型包括结构体（struct），联合体（union）和位域bitfield。在结构体部分，着重说明了结构体变量如何访问成员变量，`.`操作符和`->`操作符的不同，以及结构体变量的大小需考虑内存对齐等细节。联合体最需要注意的就是其大小等同于成员变量中最大元素的大小。位域部分需了解其在C语言中的语法与使用。动态内存分配部分介绍了三个API，`malloc`、`calloc`和`free`，并基于这三个API，引入对链表linkedlist和二叉树binary tree的C实现的介绍。二叉树部分还涉及了遍历二叉树的三种方法，前序遍历（preorder）、中序遍历（inorder）和后序遍历（postorder）。

**Lecture 7：**介绍了C语言指针的进阶用法，并引入了栈（stack）和队列（queue）数据结构的概念和实现。关于指针的进阶用法，用指针数组和字符串数组两个例子来说明指向指针的指针的用法，顺带也说了一下如何用C语言定义多维数组。然后引入了栈和队列两种数据结构的介绍，分别用数组和链表对两种数据结构进行设计和实现，最后结合两种数据结构设计一个简单的表达式计算器，计算器将前序计算表达式转换成后序计算表达式，并计算后序表达式的结果。计算器具体的实现可参照本章的练习题。

**Lecture 8：**继续介绍C语言指针的进阶用法，并引入哈希表（hash）数据结构的概念和实现。首先介绍`void`指针的使用，需注意`void`指针不能够被解引用。其次介绍了函数指针（function pointer）的用法，声明的格式同`int (*fn)(void *, void *)`类似。C的函数指针主要应用在回调函数（callback）上，以`qsort`函数为例说明了函数指针的用法，及其灵活性。C的函数指针也可以构成指针数组。哈希表可以简单理解成链表和数组的结合体，最后介绍了哈希表的简单实现。

**Lecture 9：**介绍了C语言外部库的使用和创建，同时引入了`B-tree`和优先队列`priority queues`两种数据结构。在外部库的使用介绍中，首先说明符号（symbol）的定义，包括全局变量、静态变量和函数等。然后通过一个简单`hello.c`来阐述静态链接和动态链接的差异及其各自的优缺点。程序也可以通过定义在头文件`dlfcn.h`里的`dlopen/dlsys/dlclose`函数在运行态中加载动态库。对于静态库的创建，可以先编译但不链接源代码，然后将生成的可重定位目标文件通过`ar`命令打包成一个后缀为`.a`的静态库文件。对于动态库的创建，则可通过`-shared -fPIC`编译器选项生成自定义的后缀为`.so`的动态库文件。`B-tree`是更通用版本的二叉搜索树，这里的通用指的是每个节点的值和对应的子节点不止一个，并且`B-tree`始终是平衡的。对于`B-tree`和`priority queues`，分别介绍了相应的搜索（search）、插入（insertion）和删除（deletion）等操作。具体的用法和实践在**Problem Set 7**中有更好的体现。

**Lecture 10：**介绍了C语言标准库，并列出了常用标准库中的使用频率较高的函数。本章中介绍的标准库有`stdio.h`、`ctype.h`、`string.h`、`stdlib.h`、`assert.h`、`stdarg.h`以及`time.h`。`stdio.h`主要是包含文件以及输入输出的操作。`ctype.h`主要是包含字符处理的相关的API。`string.h`主要是包含字符串处理和内存相关的API。`stdlib.h`定义了很多常用的功能函数，包括随机函数`rand()`等。`assert.h`包含关于断言的相关函数。`stdarg.h`定义了C语言可变参数函数的相关API。`time.h`是C语言关于实践相关的库函数的头文件定义。

**Lecture 11：**介绍了C语言动态内存分配的实现，`valgrind`工具的使用以及垃圾回收（garbage collection）的机制。这个章节的内容基本和[CSAPP](https://csapp.cs.cmu.edu/)中第九章虚拟内存的内容一致，包括本章中介绍`malloc`不同的实现也是第九章的课后实验。个人觉得详细的内容还是**CSAPP**（深入理解计算机系统）这本书介绍得比较详细。

**Lecture 12：** 介绍了多线程编程的基本概念，以及`pthread`相应的API的使用。首先要区别两组组概念：并行`Parallelism`和并发`Concurrency`、进程`Process`和线程`Thread`，然后说明多线程编程给程序员带来的好处。后半部分介绍了`pthread`相关的API，着重介绍了线程创建`pthread_create`、线程退出`pthread_exit`、线程同步`thread_join`等。也对`pthread mutex`锁的API进行说明，并结合实例阐述其作用。最后对条件变量`condition variable`的API进行介绍，并给出实例说明。

**Lecture 13：** 继续介绍多线程编程，同时引入套接字`sockets`和异步IO的说明。首先说明多线程编程中会出现的竞争条件`race condition`的问题，然后介绍信号量`semaphore`的使用，以及如何用信号量来实现`生产者-消费者`模型。接着介绍了线程安全、可重入函数`reentrant function`、死锁`deadlock`以及优先级反转`priority inversion`等多线程编程中的概念和问题。后半部分介绍了`Client-Server`模型中的套接字相关的API，最后引入异步I/O中的`select`、`poll`的说明和使用。

**Lecture 14：**介绍了进程间通信的几种方式。本节从以下方式中介绍了信号、`Fork`、管道、`FIFO`四种方式进行了简单的说明。

* 信号`signal`
* 管道`pipes`
* FIFO队列
* 共享内存
* 信号量
* 套接字`sockets`



## 2.3 编程练习

**Problem Set 1：**练习1主要是针对**Lecture 1**的讲到的一些C语言基本知识点进行考察。比如区分`7`、`'7'`和`"7"`在C语言中含义的不同，要警惕使用宏定义进行算数运算时表达式是否符合预期。最后就是亲自上手编写、编译、调试C程序*Hello World*，熟悉编写C语言的整体流程。

**Problem Set 2：**练习2主要是通过一些小例子，对**Lecture 2**中讲到的C各个数据类型的大小，逻辑运算符、位运算符、运算符优先级进行练习。

**Problem Set 3：**练习3.1通过一个具体的code profiling代码的例子，来说明C语言中`register`关键词对执行程序速度的影响。练习3.2锻炼C模块化编程的思想，着重强调了`extern`关键词的作用。练习3.3练习如何在`do...while`、`while`、`for`循环之间进行代码转换。练习3.4训练的是C语言实现`wc`命令的功能，其中关于实现单词的计数需要用到一些小技巧。练习3.5个人觉得最有意思，用C语言实现打开文件读写文件内容，并针对实际需求对处理结果进行格式化输出。

**Problem Set 4：**练习4.1要求将Lecture5中插入排序的数组实现改为用指针去实现，强化了C语言数组与指针之间的联系。练习4.2很有意思，根据提示实现字符串数组`strspn`和`strcspn`，其中的关键在于如何使用`strpos`函数定位相应的字符位置。练习4.3以插入排序为基础，引入了插入排序的改进版---希尔排序（shell sort），结合插入排序的代码完成希尔排序的实现。

**Problem Set 5：**练习5.1熟悉关于链表数据结构的相关操作，包括打印链表数据，添加/删除节点，查询节点，删除链表等操作。练习5.2熟悉关于二叉树数据结构的相关操作，包括添加树节点，使用前序/中序/后续遍历二叉树，计算二叉树节点数，删除二叉树等操作。

**Problem Set 6a：**练习6.1使用栈和队列两种数据结构完成一个简单的计算器，主要完成其中`infix_to_postfix`（前缀表达式转换成后缀表达式）和`evaluate_postfix`（由后缀表达式得出计算结果）两个函数的实现。可参照**Lecture 7**中的调度场算法（shunting yard algorithm）来进行编码，题目看似很简单但我觉得很锻炼编程能力。练习6.2引入`trie`树数据结构，通过`trie`树来实现一个单向的英语转法语的字典，主要是实现其中的`add_node`（添加节点）、`delete_node`（删除节点）、`add_word`（将单词添加进`trie`树）以及`lookup_word`（查找对应英文单词的法语单词）函数。关于`trie`树的详细介绍可参照此[文章](https://medium.com/basecs/trying-to-understand-tries-3ec6bede0014)。

**Problem Set 6b：**练习6.1给出一组学生的信息，然后使用`qsort`函数对信息进行排序打印，练习主要加深对函数指针的理解和使用。练习6.2使用哈希表对文本单词数量进行统计，完成`lookup`和`cleartable`两个函数的实现。练习6.2需注意的是，在更新每个`bucket`对应的链表时，头节点必须使用全局变量`table[hash]`而不能使用函数内的局部变量，即使局部变量的初始值等同于`table[hash]`。

**Problem Set 7：**练习7.1a主要是熟悉`sqlite3`库相应的API，包括如何用`sqlite3_open`打开一个数据库，如何用`sqlite3_exec`执行相应的SQL语句并将数据库条目保存到自定义的`B-tree`中，以及如何用`sqlite3_close`关闭数据库。练习7.1b主要完成`find_index`、`inorder_traversal`以及`find_value`三个函数的实现，并在`main`函数中添加将`B-tree`中序遍历结果输出到命令行参数指定的文件的逻辑。练习7.1c完成用`find_value`函数实现对`B-tree`的搜索，同时需要在`main`函数中添加查询的命令行交互实现。练习7.1d则是训练如何创建静态库、动态库，并且使用创建好的静态库和动态库，本质上就是把前三个练习实现的功能抽出来放到`initialize_db`、`locate_movie`、`dump_sorted_list`以及`cleanup`四个函数中，并且将四个函数作为库中的函数被其他程序调用。

**Lab 1：** Lab1是对`Game of Life`游戏的实现。在`Part A`部分，我们对`num_neighbors`,`get_next_state`,`next_generation`函数实现并调用，从而得到50轮迭代后的细胞状态。`Part B`部分则是从文件读取细胞的初始化状态，并将迭代后的细胞状态存放在文件里。通过此实验我们更加了解C函数的调用流程，并对C语言的文件I/O操作的API（`fopen`、`fputs`）等的使用更加熟悉。

**Lab 2：** Lab2是对霍夫曼编码的实现。实现霍夫曼编码就要理解其流程：

1. 计算每个字符在文件中的频率
2. 根据每个字符的频率将字符及其对应的数量添加进优先队列
3. 利用优先队列创建霍夫曼树
4. 遍历霍夫曼树查找字符和二进制编码的映射关系
5. 遍历文件字符，将其转换成二进制编码

所以在此实验中实现霍夫曼的编码，我们熟悉了相应的文件读写操作（我在`feof`和`EOF`的区别上纠结了很长时间），并理解了优先队列数据结构，以及如何利用优先队列创建霍夫曼树。实验最后部分是通过一个具体的文件`book.txt`来完成霍夫曼的编码和解码，并计算出相应的压缩率。我得到的结果是`1.7496`。

# 3. 课程总结与评价

查看对应repo的提交记录，发现最早提交的时间点是2018年12月份。可以说，`MIT 6.087`这门课我是断断续续用了4年时间才完整地过完。其实准确来说这门课我一共推进了三次。第一次是刚刚来Arm实习，C语言一窍不通的时候开始的。不过这一次只是看了前面C语言的基本语法部分，后面稍微涉及操作系统的概念时就放弃了。第二次是2021年中，当时看CSAPP有了一定的系统编程的了解，再去复习了一遍相应的assignment，感觉自己对很多C数据结构的实现应用地就更加熟练，研究生期间算法课上的那种数据结构恐惧症的魔咒算是打破了。第三次就是2022年的国庆，完成了剩下的Lab，并对最后的多线程编程等内容结合PPT自己扩展找资源加深理解。

可以夸张一点地说，这门课帮助我在Arm工作，乃至在系统编程方向上奠定了基础。可能最重要的一点是，用C语言去解决问题对我来言，难点不再是以前的C语言的使用，而在于问题解决的思路了。这种小小的自信在四年前之前是不敢奢想的。

总之，这门课最大的遗憾是没有视频，并且后半部分针对多线程编程、进程间通信等话题也只是点到为止。不过过完这门课，可以大胆地宣称自己是个**初级系统工程师**了。

