---
title: 理解Shell脚本中'2>&1'的含义
date: 2020-04-27 21:49:35
tags:
- 技术日志
- Shell编程
categories:
- 技术日志
---

**关键词：** Bash脚本  重定向  文件描述符  

## 0. 前言

最近因为要理解VPP中的CSIT项目，里面很多Bash Shell脚本，于是过了一遍[Bash的教程](https://wangdoc.com/bash/)。又在CSIT脚本里看到有`2>&1`这个符号，以前不求甚解不知其含义，现在谷歌搜索发现一篇[文章](https://www.brianstorti.com/understanding-shell-script-idiom-redirect/)介绍得相当具体，翻译之作为技术总结。

<!-- more -->

## 1. 概述

在编程时，我们经常会有一些习惯用法。这些习惯用法要么是程序员约定俗成的编码方式，要么是某个问题的通用解决方法。Shell脚本也不例外，然而有一个相当通用的Shell习惯用法没有被程序员们充分理解，那就是`2>&1`。举个例子，`ls foo > /dev/null 2>&1`。本篇文章就来说明一下`2>&1`背后的故事。

## 2. I/O重定向简介

简而言之，重定向就是用来将一个命令的输出发送到另一个位置的机制。举个例子，如果我们用`cat`去查看一个文件，输出默认会打印在屏幕上。

```shell
$ cat foo.txt
foo
bar
baz
```

但是我们可以把`cat`的输出重定向到别的地方。举个例子，我们可以把上述的结果重定向到`output.txt`文件中。

```shell
$ cat foo.txt > output.txt

$ cat output.txt
foo
bar
baz
```

注意到第一个`cat`命令我们没有在屏幕上见到任何输出。我们把标准输出（`stdout`）的位置改成了`output.txt`文件，所以屏幕上没有任何输出。

需要注意的是，我们还有标准错误`stderr`，程序可以将错误信息发送到这个位置。比如说，我们想用`cat`去查看一个不存在的文件，如下所示：

```shell
$ cat nop.txt > output.txt
cat: nop.txt: No such file or directory
```

即使我们把`stdout`重定向到`output.txt`文件，我们依然在屏幕上看到了错误信息。这是因为我们只是重定向了标准输出，并没有对标准错误进行重定向。

## 3. 文件描述符简介

文件描述符就是一个表示打开文件的正整数。如果你打开了100个文件，那么你就有100个文件描述符。需要注意的是，Unix系统的设计哲学中有一条是，**万物皆文件**。对于标准输出（`stdout`）和标准错误（`stderr`），也有其相应的文件描述符。

## 4. 汇总

回到我们一开始的例子，当我们将`cat foo.txt`的输出重定向到`output.txt`中时，我们可在命令行中输入以下命令：

```shell
$ cat foo.txt 1> output.txt
```

这里的`1`就是`stdout`的文件描述符。重定向操作的语法是`[FILE_DESCRIPTOR]>`，所以对于将`stdout`重定向到其它地方，我们可以写成`1>`。

因此，对于重定向标准错误`stderr`而言，我们只需要将文件描述符放在正确的位置上：

```shell
# Using stderr file descriptor (2) to redirect the errors to a file
$ cat nop.txt 2> error.txt

$ cat error.txt
cat: nop.txt: No such file or directory
```

到此为止，我们可能已经清楚`2>&1`这个习惯用法的含义了，不过我们还是系统地说明一下。

我们用`&1`来引用文件描述符`1`（`stdout`）的值。因此当我们用`2>&1`时，我们的意思是"把`stderr`的输出重定向到和`stdout`重定向输出相同的位置"。我们这么做的原因就是把`stdout`和`stderr`的输出重定向到同一个地方。

```shell
$ cat foo.txt > output.txt 2>&1

$ cat output.txt
foo
bar
baz

$ cat nop.txt > output.txt 2>&1

$ cat output.txt
cat: nop.txt: No such file or directory
```

## 5. 回顾

- 程序可将输出发送到标准输出（`stdout`）和标准错误（`stderr`）两个位置；
- 我们可以将程序的输出重定向到不同的位置（比如重定向到一个文件）；
- `command > output`是`command 1> output`的简写；
- 我们可以使用`&[FILE_DESCRIPTOR]`来引用一个文件描述符的值；
- 用`2>&1`将会把`stderr`的输出重定向到`stdout`中（`1>&2`的含义相反）。



## 参考链接

1. https://www.brianstorti.com/understanding-shell-script-idiom-redirect/