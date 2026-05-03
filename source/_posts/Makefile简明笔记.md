---
title: Makefile简明笔记
date: 2023-08-28 22:45:39
tags:
- 技术日志
- C/C++
categories:
- 技术日志
- C/C++
---

## 0. 前言

一直想写一篇关于`VPP`构建系统的博客，但`VPP`的构建系统里包含着一个顶级目录下的`Makefile`，一个在`build-root`下由数据驱动的`Makefile`，以及一系列子目录下的`Makefile`。理解`VPP`的构建系统就要求对`Makefile`掌握熟练。本文就是自己在使用`Makefile`过程中的相关知识点的回顾和总结。

<!-- more -->

## 1. `Makefile`简介

### 1.1 `Makefile`的规则

```makefile
target ... : prerequisites ...
	command
	...
	...
```

`target`可以是一个目标文件，也可以是一个可执行文件，还可以是一个标签（伪目标）。

`prerequisites`是生成`target`所依赖的文件。

`command`是生成`target`所要执行的命令。

需要注意的是，`Makefile`中的命令，也就是上述的`command`，必须是以`Tab`键开始。

以上描述的三者相互构成依赖关系，一句话描述的话就是：**prerequisites中如果有文件比target要更新的话，command所定义的命令就会被执行**。

上述描述的就是`Makefile`的规则，也是`Makefile`的核心内容。

### 1.2 `make`命令自动推导

GNU `make`命令很强大，可以自动推导文件以及文件依赖后面的命令。只要`make`看到一个`.o`文件，它就会自动地把`.c`文件添加到依赖关系中。例如，如果`make`找到一个`example.o`，那么`example.c`就会是`example.o`的依赖文件，并且`cc -c example.c`也会被自动推导出来。这种方法，就是`make`的隐式推导规则。

### 1.3 `Makefile`文件包含的内容

`Makefile`文件里主要包含了五个部分的内容：显示规则、隐式规则、变量定义、文件指示和注释。

- 显示规则说明了如何生成一个或多个目标文件。程序员负责编写要生成的文件、文件的依赖以及生成的命令。
- 隐晦规则使用的是`make`自动推导的过程，不需要程序员像显示规则一样指定整个依赖关系。
- `Makefile`里的变量可以理解为C语言中的宏。当`Makefile`被执行时，其中的变量会被扩展到相应的引用位置上。
- 文件指示。`Makefile`可以引用另一个`Makefile`文件，类似于C中的`include`。`Makefile`中可以通过`#if`等条件语句来指定有效部分。同时，`Makefile`中还可以定义一个多行的命令。
- 注释。`Makefile`使用`#`字符作为注释符。

### 1.4 引入其它的`Makefile`

在`Makefile`中使用`include`关键字可以把别的`Makefile`包含进来，语法如下：

```makefile
include <filename>
```

`make`命令执行时，会搜索`include`所包含的所有的`Makefile`，并将其内容放置在当前位置。如果文件没有声明绝对路径或者相对路径的话，`make`会在当前目录下首先搜索。如果当前目录没有对应的文件，`make`继续在以下目录中搜索：

- 若`make`执行时，添加了`-I`或者`--include-dir`参数，那么`make`回去此参数指定的目录搜索。
- 若`<prefix>/include`包括`/usr/local/bin`或`/usr/include`存在的话，`make`也会在相应目录中搜索。

若文件没有被查找到，`make`会提示警告信息，但不会立即出错。如果想让`make`不理会无法读取的文件，继续执行接下来的命令，可在`include`前加一个`-`号。

```makefile
-include <filename>
```

### 1.5 `make`工作流程

GNU`make`工作流程如下：

1. 读入所有的`Makefile`文件。
2. 读入被`include`的其它`Makefile`文件。
3. 初始化文件中的变量。
4. 推导隐式规则，并分析所有规则。
5. 为所有的目标文件创建依赖关系链。
6. 根据依赖关系，决定哪些目标要重新生成。
7. 执行生成命令。

## 2. `Makefile`命令

`Makefile`中每条规则中的命令和操作系统`Shell`的命令行是一致的。`make`会按照顺序一条一条地执行命令，每条命令的开头必须是以`Tab`键开头的。

### 2.1 命令显示

一般情况下，`make`会把其要执行的命令行在命令执行前输出到屏幕上。当我们在命令行前添加`@`字符时，这个命令将不被显示出来。

```
# Makefile1
echo "Compiling xxx module..."
# This will output the command and the result
echo "Compiling xxx module..."
Compiling xxx module...

# Makefile2
@echo "Compiling xxx module..."
# This will only output the result
compiling xxx module...
```

`make`执行时带入参数`-n`或者`--just-print`，只会显示命令，不会执行命令，可帮助调试`Makefile`，了解对应`Makefile`执行的具体步骤和内容。

`make`执行时带入参数`-s`或者`--silent`或者`--quiet`，则在执行时不会显示执行的命令。

### 2.2 命令执行

当在执行`Makefile`中的命令去更新目标文件时，执行命令的过程中，`shell`会为每一行命令创建一个子进程，在子进程中执行对应的命令。这就意味着一些如`cd`等`shell`命令的作用域只在当前行，不会影响到接下来命令的执行结果。如果想要`cd`的结果作用于接下来的命令，则需把两条命令写在同一行，中间用`;`隔开。

```
# Makefile1
foo:
    cd ~
    pwd
# This will output the directory where your Makefile is

# Makefile2
foo:
    cd ~; pwd
# This will output the home directory specified by $HOME
```

### 2.3 并行执行

GNU`make`可一次执行多条命令。通常情况下，`make`一次只执行一条命令，下一条命令需等待当前命令结束再执行。但是我们可以通过设置`-j`或者`--jobs`选项来显示地通知`make`并行执行命令，并行执行命令的数量即是`-j`或`--jobs`后接的数字。

### 2.4 错误处理

`make`会检查每行命令执行过后的退出码。如果其中一条命令的退出码有问题，`make`则不会继续执行那条命令对应接下来的命令，甚至可能直接退出执行。

当然有时候一些命令执行错误也无伤大雅，比如用`mkdir`创建一个已经存在的目录，`make`执行就会报错。解决这种问题可以在命令前添加`-`，如下例所示。即使`rm`无法删除文件，`make`也会继续执行。

```
clean:
	-rm -f *.o
```

另一种方法是在`make`选项中添加`-i`或`--ignore-errors`，显示地说明忽略`make`执行过程中所有的错误。

### 2.5 嵌套使用

我们可以在`Makefile`中把`make`当作命令使用。一般对于构建大型软件系统来说，我们会对不同的子项目编写单独的`Makefile`。举个例子，如果我们有个子目录`subdir`，对应`subdir`有自己的`Makefile`，我们可以在顶级`Makefile`里做如下编写：

```
# Method 1
subsystem:
	cd subdir && $(MAKE)

# Method 2
subsystem:
	$(MAKE) -C subdir
```

当我们嵌套使用`make`时，最好添加`.PHONY`对其目标声明为伪目标。

### 2.6 定义命令包

如果`Makefile`中出现一组相同的命令序列，我们则可以将这些相同的命令序列组合定义成一个变量。语法如下所示。`echo_var`即是这个命令包的名字，命令包还可以接受参数。我们可以直接把`echo_var`看作一个变量，用`$(echo_var)`。也可以把`echo_var`看作一个函数，用`call`调用`echo_var`。下述例子`t1`和`t2`的执行结果相同，都是输出`one`。

```makefile
define echo_var
@echo $(1)
endef

all: t1 t2

t1:
    $(echo_var) "one"

t2:
    $(call echo_var, "one")
```

## 3. `Makefile`变量

`Makefile`中定义的变量，类似于C/C++语言中的宏，本质上就是文本字符串，可以在`Makefile`执行过程中自动展开。变量在声明时需要赋初值，在使用变量时，需要给变量名前加`$`，而且最好使用`()`和`{}`将变量包起来。

### 3.1 变量赋值

在`Makefile`中，赋值方式有`=`、`:=`、`?=`和`+=`四种。

#### `=` 递归展开变量

用`=`或者`define`关键词都可以定义此类型变量。如果变量的定义援引了其它的变量，则引用会一直展开下去，直到找到被引用变量的最新定义，并以此作为变量的返回值。

```makefile
var = I love
distro = centos
var += $(distro)
distro = ubuntu

all:
    @echo $(var)
```

上述例子运行结果是：

```shell
I love ubuntu
```

`=`的赋值时没有先后顺序的，使用时要小心递归的出现，如`A = $(A)`这样的写法就会导致变量递归定义。

#### `:=`简单扩展变量

`:=`用此方式定义的变量，会在变量定义的位置，按照被引用的变量的当前值做一次展开。被引用的变量的值后续发生变化不会影响当前定义变量的值。

```makefile
var := I love
distro = centos
res := $(var) $(distro)
distro = ubuntu

all:
    @echo $(res)
```

上述例子运行结果是：

```shell
I love centos
```

但如果上述中的`res := $(var) $(distro)`改为`res = $(var) $(distro)`，运行的结果则为`I love ubuntu`。可以体会到两种赋值方式的差别。

#### `?=`条件变量

`?=`只在变量未被赋值的情况下起作用。如果变量没有初始化，变量的定义即为默认值。

```makefile
ARCH = arm
ARCH ?= amd64

all:
    @echo $(ARCH)
```

上述例子运行结果是：

```shell
arm
```

#### `+=`变量添加新的文本

当变量未被定义，则`+=`的作用和`=`是一致的，定义一个递归展开的变量。但如果变量之前已被定义，`+=`只是做字符的添加工作。如果一开始使用`:=`定义变量，则`+=`就是对变量添加新的文本。但如果一开始使用`:`定义变量，`+=`则不会立即进行变量展开，直到找到最新的变量定义。第一个例子中的`var`的最终值是`I love ubuntu`说明了这种用法。

### 3.2 自动化变量

`Makefile`的自动变量包括目标文件、依赖文件等，本质上是对一类变量的简写。

首先来看三个常用的自动化变量的含义：

- `$@` 代表目标文件名
- `$<`代表规则的第一个依赖文件
- `$^`代表规则的所有的依赖文件，以空格分离

举一个例子来说明三个自动化变量的使用。创建一个项目，包含如下所示的文件：

```c
/* func1.h */
void func1();

/* func1.c */
#include <stdio.h>
void func1()
{
    printf("func1: hello!\n");
}

/* func2.h */
void func2();

/* func2.c */
#include <stdio.h>
void func2()
{
    printf("func2: hello!\n");
}

/* main.c */
#include "func1.h"
#include "func2.h"

int main()
{
    func1();
    func2();
    return 0;
}
```

在这个示例项目中，`main`函数调用`func1`和`func2`，两个函数分别在`func1.h`和`func2.h`里声明。我们编写项目的`makefile`如下所示：

```makefile
CC = gcc

all: main

main: main.o func1.o func2.o
    $(CC) $^ -o $@

%.o: %.c
    $(CC) -c $< -o $@

.PHONY: run clean
run:
    ./main

clean:
    $(RM) main *.o
```

生成目标`main`，则需要`main.o`、`func1.o`和`func2.o`。所以`$^`指的是所有的依赖条件。而`%.o:%.c`的写法，则是采用了`Makefile`的静态模式，`$<`表示的是第一个依赖条件，所以其等价于如下所示的写法：

```makefile
main.o: main.c
    gcc -c main.c -o main.o
		
func1.o: func1.c
    gcc -c func1.c -o func1.o
		
func2.o: func2.c
    gcc -c func2.c -o func2.o
```

还有一些出现频率不高的自动化变量：

- `$?`所有比目标文件新的依赖文件的集合

```makefile
libxyz: x.o y.o z.o
    ar r libxyz $?
```

如果`x.o`、`y.o`或者`z.o`发生了更新，`$?`的含义则是将更新过后的目标文件插入到库`libxyz`中。

- `$+`和`$^`类似，表示所有依赖文件的集合（重复的依赖文件不剔除），主要用于链接过程的使用
- `$*`表示在模式匹配和静态模式规则中，代表目标模式中%的部分。比如`hello.c`，当匹配模式为`%.c`时，`$*`表示的是`hello`

## 4. `Makefile`条件判断

`Makefile`里有两种条件判断的使用方法： `ifeq`和`ifdef`。接下来我们就用例子加以说明。

### 4.1 `ifeq`

`ifeq` 和`ifneq`用于比较`Makefile`中变量是否相等。

```makefile
DEBUG = true

ifeq ($(DEBUG), true)
    CFLAGS = -g
else
    CFLAGS = -O2
endif

all:
    gcc $(CFLAGS) main.c -o my_program
```

在上述例子中，变量`DEBUG`的值与`true`比较，如果相等，则`CFLAGS`被设置成`-g`，为调试选项；如果不等，则`CFLAGS`被设置成`-O2`，为优化选项。

### 4.2 `ifdef`

`ifdef`和`ifndef`用于判断变量是否被定义。

```makefile
ifdef VERBOSE
	PRINT = echo
else
	PRINT = @echo
endif

all:
	$(PRINT) "Building my_program..."
	gcc main.c -o my_program
```

上述例子中，检查`VERBOSE`是否被定义（非空），如果已被定义，则`PRINT`设为`echo`，`makefile`运行时打印出`Building my_program...`。若未定义，则`PRINT`设为`@echo`，只展示编译的命令。

## 5. `Makefile`函数

`Makefile`中函数的基本调用语法如下：

```
$(function arguments)
```

或者是：

```
${function arguments}
```

`arguments`为函数的参数，参数间以逗号分隔，函数名和参数之间以空格分隔。接下来我们来具体看看`Makefile`里的函数。

### 5.1 字符串处理函数

#### `subst`字符串替换函数

```makefile
$(subst <from>,<to>,<text>)
```

把字符串`text`中的`from`全部替换成`to`。

示例：

```makefile
$(subst ee,EE,feet on the street)
```

上述返回结果为`fEEt on the strEEt`。

#### `patsubst`模式字符串替换函数

```makefile
$(patsubst <pattern>,<replacement>,<text>)
```

查找`text`中的单词（单词以空格、Tab、回车或换行分隔）是否符合模式`pattern`，如果匹配的话，则用`replacement`替换。

示例：

```makefile
$(patsubst %.c,%.o,x.c.c y.c)
```

上述返回结果为`x.c.o y.o`。

#### `strip`去空格函数

```makefile
$(strip <string>)
```

去除`string`字符串中开头和结尾的空字符。

#### `findstring`查找字符串函数

```makefile
$(findstring <find>,<in>)
```

在字符串`in`中查找`find`字符串，如果找到，则返回`find`。否则返回空字符串。

示例：

```makefile
$(findstring a,a b c)
$(findstring a,b c)
```

第一个函数返回结果`a`，第二个函数返回空字符串。

#### `filter`过滤函数

```makefile
$(filter <pattern>,<text>)
```

以`pattern`模式过滤`text`字符串中的单词，保留符合模式`pattern`的单词。匹配的模式可以有多个。

示例：

```makefile
sources := foo.c bar.c baz.s ugh.h
foo: $(sources)
		cc $(filter %.c %.s, $(sources)) -o foo
```

上述`$(filter %.c %.s, $(sources))`返回的值是`foo.c bar.c baz.s`。

#### `filter-out`反过滤函数

```makefile
$(filter-out <pattern>,<text>)
```

同`filter`函数的含义相反，以`pattern`模式过滤`text`字符串中的单词，去除符合模式`pattern`的单词。

#### `word`取单词函数

```makefile
$(word <n>,<text>)
```

取字符串`text`中的第`n`的单词（从1开始索引）。

示例：

```makefile
$(word 2, foo bar baz)
```

上述返回值是`bar`。

#### `wordlist`取单词串函数

```makefile
$(wordlist <start>,<end>,<text>)
```

从字符串`text`中取`start`到`end`的字符串。

示例：

```makefile
$(wordlist 2,3,foo bar baz)
```

上述返回值是`bar baz`。

#### `words`单词个数统计函数

```makefile
$(words <text>)
```

统计字符串中`text`的单词个数。

示例：

```makefile
$(word $(words, foo bar baz), foo bar baz)
```

上述返回值是`baz`。

### 5.2 文件名操作函数

#### `dir`取目录函数

```makefile
$(dir <names...>)
```

从文件名`names`取出目录部分。目录部分指最后一个反斜杠之前的部分。若无反斜杠，则返回`./`。

示例：

```makefile
$(dir src/foo.c main)
```

上述返回值是`src/ ./`。

#### `notdir`取文件函数

```makefile
$(notdir <names...>)
```

从文件名`names`取出非目录部分。

示例：

```makefile
$(notdir src/foo.c main)
```

上述返回值是`foo.c main`。

#### `suffix`取后缀函数

```makefile
$(suffix <names...>)
```

从文件名`names`取出各个文件名的后缀。返回文件名`names`的后缀。若无后缀，则返回空字符串。

示例：

```makefile
$(suffix src/foo.c src-1.0/bar.c hacks)
```

上述返回值`.c .c`。

#### `basename`取前缀函数

```makefile
$(basename <names...>)
```

`suffix`函数的对立函数，从文件`names`取出每个文件名的前缀。

示例：

```makefile
$(basename src/foo.c src-1.0/bar.c hacks)
```

上述返回值是`src/foo src-1.0/bar hacks`。

#### `addsuffix`加后缀函数

```makefile
$(addsuffix <suffix>,<names...>)
```

把`suffix`添加到`names`的每个单词后。

示例：

```makefile
$(addsuffix .c, foo bar)
```

上述返回值是`foo.c bar.c`。

#### `addprefix`加前缀函数

```makefile
$(addprefix <prefix>,<names...>)
```

把`prefix`添加到`names`的每个单词之前。

示例：

```makefile
$(addprefix src, foo.c bar.c)
```

上述返回值是`src/foo.c src/bar.c`。

### 5.3 `foreach`函数

```makefile
$(foreach <var>,<list>,<text>)
```

`foreach`函数的含义是，把参数`list`中的单词逐一取出放到参数`var`所指定的变量中，然后再执行`text`所包含的表达式。每一次执行`text`都会返回一个字符串，最终的返回值就是每次字符串的组合。

示例：

```makefile
names := a b c d
files := $(foreach n, $(names),$(n).o)
```

上述返回值是`a.o b.o c.o d.o`。

需要注意的一点是，`foreach`函数中的`var`参数是一个临时的局部变量，作用域只在`foreach`函数中。

### 5.4 `if`函数

```makefile
$(if <condition>,<then-part>)
```

或者

```makefile
$(if <condition>,<then-part>,<else-part>)
```

使用含义直接明了，不再赘述。

### 5.5 `call`函数

```makefile
$(call <expression>,<parm1>,<parm2>,...,<parmN>)
```

`call`函数执行时，`expression`参数中的变量，`$(1)`、`$(2)`等，会被`parm1`、`parm2`等取代。`expression`的返回值就是`call`的返回值。

`call`函数使用中需要注意的一点是，传递参数时不要有多余的空格。

### 5.6 `shell`函数

```makefile
$(shell <command>)
```

`shell`命令能够把执行操作系统命令后的输出作为函数返回。

### 5.7 `origin`函数

```makefile
$(origin <variable>)
```

`orign`函数用作判断变量的来源。这里不作细究，需要时查文档。

### 5.8 `eval`函数

`eval`函数执行时会对参数进行两次展开。第一次展开过程是由函数本身完成的，第二次是函数展开过后的结果会被认为是`Makefile`的内容由`make`执行。

## 总结

了解`Makefile`的基本语法规则和工作流程，熟悉`makefile`的自动变量，静态模式等技巧，会使用基本的条件判断语句和`Makefile`中相应的函数，我觉得对`Makefile`的掌握也就可以算是入了门。