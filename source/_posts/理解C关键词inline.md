---
title: 理解C关键词inline
date: 2022-01-17 23:05:50
tags:
- 技术日志
- C/C++
categories:
- 技术日志
- C/C++
---

**关键词：** C/C++ inline

# 0. 前言

最近在看VPP`ip4-frag`的代码，尤其是同事关于`ip4-frag`多架构支持的patch，其中的一个改动是将核心调用函数的`static`改为`always_inline`，查看`always_inline`在VPP里的定义，发现`always_inline`是一个宏，其在VPP的定义如下。那么这个`always_inline`到底是什么含义？`inline`关键词到底有什么作用？参考油管上的这个[教学视频](https://www.youtube.com/watch?v=t6Jfrmdg5rk&list=PL-Ov9uEUQe2uQ1Tr5Oo4lDXi1EERTt-qP&index=35&ab_channel=JacobSorber)，用例子对C关键词`inline`进行一次总结。

<!-- more -->

```c
#if CLIB_DEBUG > 0
#define always_inline static inline
#define static_always_inline static inline
#else
#define always_inline static inline __attribute__ ((__always_inline__))
#define static_always_inline static inline __attribute__ ((__always_inline__))
#endif
```

# 1. 正文

视频包含一个例子，这个例子一个有四个文件，分别是`Makefile`、`test.c`、`nums.c`、`nums.h`。目前`nums.c`和`nums.h`是空文件，`test.c`的内容如下。可以看到，我们定义了两个非常简单的函数`max`和`two`，函数`max`比较两个整型数的大小，函数`two`直接返回2。`main`函数比较命令行传入的参数与2进行比较，返回较大的值。

``` c
#include <stdio.h>
#include <stdlib.h>

int max(int a, int b)
{
    return a > b ? a : b;
}

int two()
{
    return 2;
}

int main(int argc, char *argv[])
{
    if (argc != 2)
    {
        printf("One argument needed!\n");
        exit(1);
    }
    
    int arg = atoi(argv[1]);
    int result;
    
    result = max(two(), arg);
    printf("result = %d\n", result);
    return 0;
}
```

对应的`Makefile`文件：

```makefile
CC=clang
CFLAGS=

all: test

test: test.c nums.o
	$(CC) $(CFLAGS) -S $< -o $@.opt0.s
	$(CC) $(CFLAGS) -O1 -S $< -o $@.opt1.s
	$(CC) $(CFLAGS) -O2 -S $< -o $@.opt2.s
	$(CC) $(CFLAGS) $< nums.o -o $@0
	$(CC) $(CFLAGS) -O1 $< nums.o -o $@1
	$(CC) $(CFLAGS) -O2 $< nums.o -o $@2
	
.phony: clean
clean:
	$(RM) *.o *.s a.out test0 test1 test2
	$(RM) -r *.dSYM
```

`Makefile`文件中通过设置不同的优化选项来生成不同的汇编代码和二进制文件。编译器使用的`clang`，对应的版本是`clang version 10.0.0-4ubuntu1`。当然可以用`gcc`或者其它编译器，不过不同编译器生成的汇编代码可能会有差异。

命令行敲入`make`指令，可以看到生成以下文件：

```shell
$ make
clang    -c -o nums.o nums.c
clang  -S test.c -o test.opt0.s
clang  -O1 -S test.c -o test.opt1.s
clang  -O2 -S test.c -o test.opt2.s
clang  test.c nums.o -o test0
clang  -O1 test.c nums.o -o test1
clang  -O2 test.c nums.o -o test2
$ ls
Makefile  nums.c  nums.h  nums.o  test0  test1  test2  test.c  test.opt0.s  test.opt1.s  test.opt2.s
```

执行`test0/1/2`文件，查看程序执行是否符合预期。

```shell
$ ./test0 1
result = 2
$ ./test0 3
result = 3
$ ./test1 1
result = 2
$ ./test1 3
result = 3
$ ./test2 0
result = 2
$ ./test2 4
result = 4
```

## 1.1 编译器会自动将简单函数优化成内联汇编代码

以上述例子生成的三个汇编代码中调用`two`和`max`函数对应生成的汇编代码为例。当编译器选项为`-O0`时，可以看到`main`函数中调用了`two`和`max`函数。当编译器选项为`-O1`时，对应的`two`函数直接通过`movl $2, %edi`语句实现，`max`函数依旧被调用。当编译器选项为`-O2`时，编译器直接将`two`和`max`函数通过内联汇编代码实现。

```assembly
//test.opt0.s
        callq   two
        movl    -20(%rbp), %esi
        movl    %eax, %edi
        callq   max

//test.opt1.s
        movl    $2, %edi
        movl    %eax, %esi
        callq   max

//test.opt2.s
        movl    $2, %esi
        cmovgl  %eax, %esi
```

## 1.2 `inline`关键词只是对编译器的提示

上述的代码还没有涉及`inline`关键词。我们将代码作如下小修改，将`two`函数改为`inline`函数。

```c
#include <stdio.h>
#include <stdlib.h>

int max(int a, int b)
{
    return a > b ? a : b;
}

inline int two()
{
    return 2;
}

int main(int argc, char *argv[])
{
    if (argc != 2)
    {
        printf("One argument needed!\n");
        exit(1);
    }
    
    int arg = atoi(argv[1]);
    int result;
    
    result = max(two(), arg);
    printf("%result = %d\n", result);
    return 0;
}
```

敲入`make`编译发现如下报错。怎么回事？不是说`inline`关键词会让函数变成内联汇编代码？

```
/usr/bin/ld: /tmp/test-4ea072.o: in function `main':
test.c:(.text+0x7f): undefined reference to `two'
clang: error: linker command failed with exit code 1 (use -v to see invocation)
make: *** [Makefile:11: test] Error 1
```

再进一步观察，发现当编译器优化选项是`-O0`或`-O1`时编译出错，而`-O2`选项则编译正常。查阅C99标准文档发现以下一段说明。

>An inline definition provides an alternative to an external definition, which a translator may use to implement any call to the function in the same translation unit. It is unspecified whether a call to the function uses the inline definition or the external definition.

啥意思呢？我的理解就是说，对应一个编译单元（如例子中的`test.c`文件），其中的函数如果被定义成`inline`，则必须外部有一个同名的函数定义，并且程序在运行时并不确定最后究竟调用的是哪个函数定义。

继续使用上述例子，我们在`nums.h`和`nums.c`里新建一个`two`函数。

```c
// nums.h
#ifndef NUMS_H
#define NUMS_H
int two();
#endif

// nums.c
#include <stdio.h>
int two()
{
    int thirty = 30;
    printf("calling a broken version of two.\n");
    return thirty;
}
```

再次编译，发现不论编译器的优化选项是多少，原先的编译错误都消失了。运行程序，得到以下结果。

```sh
$ ./test0 1
calling a broken version of two.
result = 30
$ ./test1 1
calling a broken version of two.
result = 30
$ ./test2 1
result = 2
```

可以看到，不同的优化选项的程序在调用`two`函数时选用的定义是不同的。对于`-O0`和`-O1`，调用的是`nums.c`里定义的`two`函数；对于`-O2`，调用的则是`test.c`中的`inline`函数。这种不确定性是我们不想看到的，并且函数的多重定义不仅影响代码阅读，也极易出错，是我们在写代码时应极力避免的。那有什么替代方案吗？

有的，给`inline`函数添加`static`关键词，显式地告诉编译器函数的定义就在此编译单元。

```c
// test.c
static inline int two()
{
    return 2;
}
```

重新编译，再次查看对应生成的汇编代码，发现同之前的汇编代码一样。

```assembly
//test.opt0.s
        callq   two
        movl    -20(%rbp), %esi
        movl    %eax, %edi
        callq   max

//test.opt1.s
        movl    $2, %edi
        movl    %eax, %esi
        callq   max

//test.opt2.s
        movl    $2, %esi
        cmovgl  %eax, %esi
```

上述现象也就印证了标题，同时也如同wikipedia对`inline function`第一个目的介绍的一样：

>It serves as a compiler directive that suggests (but does not require) that the compiler substitude the body of the function inline by performing inline expansion, i.e. by inserting the function code at the address of each function call, thereby saving the overhead of a function call.

## 1.3 使用`__attribute__ ((always_inline))`强制内联

既然`inline`只是对编译器是否内联的提示，那么有没有一种方式，显式地告诉编译器函数就是要内联呢？

答案是有的，使用编译器的属性`__attribute__((always_inline))`，即使在没有编译器优化的情况下（`-O0`），函数也会生成内联汇编代码。

继续以原来例子为例，将`two`函数的定义添加上以上函数属性。

```c
// test.c
__attribute__((always_inline)) inline int two()
//static inline int two()
{
    return 2;
}
```

查看生成的汇编代码，可以看到所有的`two`函数都被生成了内联汇编代码，不再发生函数调用。

```assembly
//test.opt0.s
        movl    $2, %edi
        callq   max

//test.opt1.s
        movl    $2, %edi
        movl    %eax, %esi
        callq   max

//test.opt2.s
        movl    $2, %esi
        cmovgl  %eax, %esi
```

# 2. 总结

* `inline`函数只是建议编译器生成内联汇编代码，并不是强制。
* 使用`static inline`函数可避免定义`inline`函数的同时还需要外部同名函数的定义。
* 如需函数强制生成内联汇编代码，可添加编译器函数属性`__attribute__((always_inline))`。

`inline`函数本质上还是适合执行较为简单但调用频率高的函数，省去了函数调用入栈出栈的开销。但凡事都有利有弊，在程序调试过程中`inline`函数的信息就会丢失，这就可以看到为什么开头VPP的源码里针对`debug`和`release`版本的`inline`宏定义做了不同的处理。

# 3. 参考

1. https://www.youtube.com/watch?v=t6Jfrmdg5rk&list=PL-Ov9uEUQe2uQ1Tr5Oo4lDXi1EERTt-qP&index=35&t=133s&ab_channel=JacobSorber
2. https://stackoverflow.com/questions/6312597/is-inline-without-static-or-extern-ever-useful-in-c99



