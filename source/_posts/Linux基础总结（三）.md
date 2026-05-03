---
title: Linux基础总结（三）
date: 2018-01-08 10:53:06
tags:
- Linux
- 技术日志
categories:
- 技术日志 
---

**关键词：** Linux  目录管理  find命令

# 0. 前言

想要熟练使用Linux，首先得过的坎便是熟悉Linux命令行。命令行中很多的操作都是关于如何管理文件和目录，包括创建和删除目录、在不同的目录间切换、查找文件等。本文将对常用的Linux目录与文件管理命令进行总结。

<!-- more -->

# 1. 文件与目录管理

## 1.1 绝对路径与相对路径

* **绝对路径：** 路径的写法必须从**根目录**写起，如`/home/snowball/code`这个目录。
* **相对路径：**从`/home/snowball/code`切换到`/home/snowball/caffe`这个目录下时，可以写成`cd ../caffe`，这就是相对路径的写法。

常用的特殊目录：
> .  代表此目录
> .. 代表上一层目录
> \-  代表上一步使用的工作目录
> ~ 代表当前用户身份所在的主文件夹
> ~account 代表account用户的主文件夹（我的是snowball）

**注：** 
1. 所有目录下面都会存在两个目录：`.`表示此目录，`..`表示上一层目录。
2. 根目录的上一层目录`..`与根目录本身`.`是同一个目录

## 1.2 常用目录命令

* **cd**：切换目录
* **pwd**：显示当前目录（-P 显示当前路径，而非链接（link）路径）
* **mkdir**：新建目录（-m 配置文件的权限，-p 递归创建新的目录）
* **rmdir**：删除空目录（-p 递归删除空目录）

## 1.3 $PATH变量

PATH变量是由一堆目录组成，每个目录中间用冒号(:)来隔开，每个目录有顺序之分。PATH的作用是用户在命令行敲入如`ls`指令时，通过PATH寻找`ls`所在路径，执行`ls`指令，相当于Windows下系统变量的设置。如果想要把目录添加加入PATH中，可执行以下命令：
```
$ PATH="$PATH":/home/snowball
```
## 1.4 常用文件命令

1. **ls**
```
$ ls [-aAdfFhilnrRSt] 目录名称
$ ls [--color={never,auto,always}] 目录名称
$ ls [--full-time] 目录名称
参数（加粗表示常用）：
-a: 列出全部文件，包含隐藏文件（常用）
-A: 列出全部文件，但不包括`.`和`..`这两个目录
-d: 仅列出目录本身，不列出目录内的文件数据（常用）
-f: 直接列出结果，不进行排序（ls默认以文件名排序）
-F: 根据文件、目录等信息给予附加数据结构
-h: 以人可读的方式显示文件容量（GB，KB）
-i: 列出inode号码
-l: 列出长数据串，包含文件属性与权限等数据（常用）
-n: 列出UID和GID
-r: 反向输出排序结果
-R: 递归输出目录下所有内容，注意和-a的区别
-S: 以文件容量大小排序
-t: 以时间（ctime）排序
--color=never: 不依据文件特性给予颜色显示
--color=always: 显示颜色
--color=auto: 系统自行判断
--full-time: 以完整时间模式输出
--time={atime, ctime}: 输出atime和ctime时间
常用组合参数:
-alh: 以人可读的方式列出全部文件的长数据
-clt: 按照ctime时间排序输出目录内容
```

`ls`最常用的的参数还是`-l`参数。Ubuntu在默认的情况下，可使用`ll`来表示`ls -l`。

2. **cp**

cp指令，顾名思义，即为copy复制。但cp指令还可以创建快捷方式，更新旧文件，以及复制整个目录等功能。

```
$ cp [-adfilprsu] 源文件（source） 目标文件(destination)
$ cp [options] source1 source2 source3 ... directory
参数：
-a: 相当于-pdr，即将文件的所有特性都复制过来（常用）
-d: 若源文件为链接文件（link file），则复制的是链接文件
-f: 强制复制
-i: 若目标文件已经存在，在覆盖时会先询问操作的进行（常用）
-l: 进行硬链接的链接文件
-p: 连同文件的属性一同复制
-r: 递归持续复制（常用）
-s: 复制为符号链接文件（symbolic link），即"快捷方式"
```

需要注意的是，`cp -a`可以复制文件的相关权限与时间属性，但是无法更改所有者、用户组的相关信息。

3. **rm**

```
$ rm [-fir] 文件或目录
参数：
-f: 强制删除，不会出现警告信息
-i: 互动模式，删除前会询问用户是否进行操作
-r: 递归删除，删除目录中的所有内容
```

如果文件名的开头含有`-`，诸如名为`-wang`的文件，使用`rm`指令会出现错误，这时可使用`rm ./-wang`来达到删除的目的。

4. **mv**

```
$ mv [-fiu] source destination
$ mv [options] source1 source2 source3 ... directory
参数：
-f: 强制
-i: 目标文件若存在，会询问是否直接覆盖
-u: 目标文件已经存在，source较新，则会更新
```

`mv`还可以用做对文件名进行重命名，`mv test test1`这条指令便可以完成对文件名test的修改。

## 1.5 常见文件查阅命令

* **cat**: 由第一行开始显示文件内容
* **tac**: 由最后一行开始显示（tac可以看作是cat的倒写）
* **nl**: 显示的时候，顺便输出行号
* **more**: 一页一页地显示文本内容
* **less**: 与more类似，但可向前翻页
* **head**: 只显示头几行文件（默认情况下只显示前10行）
* **tail**: 只显示结尾几行文件（默认情况下只显示结尾10行）
* **od**: 以二进制方式读取文件内容（-tc 可将数据按照ASCII方式显示）

# 2. 文件与目录权限

## 2.1 文件默认权限：umask

**umask**就是指定"目前用户在新建文件夹或目录时的权限默认值"。文件与目录的默认属性为：
* 若用户创建"文件"则默认没有可执行（x）权限，即只有r、w这两个选项，则默认权限如下：`-rw-rw-rw-`

* 若用户新建"目录"，则权限'x'与是否进入此目录有关，所以默认为所有权限开放，则默认权限如下：`drwxrwxrwx`

当umask为002时，用户新建文件的默认权限为`-rw-r--r--r`，用户新建目录的默认权限为`drwxrwxr-x`。
**那么umask有什么作用？**设想这样一种场景，用户A与用户B在公司的同一个小组工作，这时有新的研发任务，主管分别为用户A和用户B分配了两个Linux用户名，同时将他们分配在同一个用户组。此时如果umask为022的话，那么用户A编写的程序用户B便无法修改，这样就无法协同工作。这时，只需将umask修改为002即可解决问题。


## 2.2 文件隐藏属性：chattr, lsattr

`chattr`这个命令对于系统的数据安全尤其重要，而`lsattr`则可查看文件的隐藏权限。
```
$ chattr [+-=] [options] 文件或目录名称
参数：
+ : 增加某个特殊参数
- : 删除某个特殊参数
= : 仅有后面接的参数

a : 当设置a之后，这个文件将只能增加数据，而不能删除或者修改数据，root权限才能设置此属性
i : 让文件"不能被删除、改名，设置连接也无法写入或添加数据"，root才能设置此属性
```
## 2.3 文件特殊权限：SUID, SGID, SBIT

我们的Linux系统中，所有账号的密码都是记录在`/etc/shadow`文件夹内，而此文件的权限为`-r-------- root root`，表明只有root可读，那普通用户snowball是不是无法修改密码？并不是，这时就体现了SUID的作用，snowball用户当然可以修改自己的密码。snowball对`/usr/bin/passwd`这个程序是有x权限的，即snowball能够执行**passwd**，而**passwd**的拥有者是root，snowball会执行**passwd**，暂时获得root的权限，然后`/etc/shadow`可以被snowball执行的**passwd**程序所修改。

**注：**SUID仅可用在二进制程序上，不能用在shell script上。

SGID的作用同SUID类似，它是暂时获得用户组的权限。
SBIT(Sticky Bit)只针对目录有效，其含义是如果目录加上SBIT的权限项目时，普通用户只能针对自己创建的文件或目录进行删除、重命名、移动等操作，而无法删除他人的文件。


- **SUID**:4
- **SGID**:2
- **SBIT**:1

**注：**文件权限中可能出现**S**和**T**，表示对应用户或用户组都没有此文件的执行权限，即表明此**S**和**T**是空的含义。

# 3. 文件与目录查询

## 3.1 which命令

`which`命令是根据PATH环境变量所规范的路径去查询"执行文件"的文件名，加上`-a`参数，可列出所有的可以找到的同名执行文件。

```
$ which ifconfig
/sbin/ifconfig
```

## 3.2 whereis命令和locate命令

Linux系统会将系统内的所有文件都记录在一个数据库文件里面，当使用`whereis`或`locate`指令时，都会以此数据库文件的内容为准，所以要比直接搜索硬盘的`find`指令要快很多。

```
$ whereis filename
$ locate [-ir] keyword
参数：
-i : 忽略大小写的差异
-r : 可接正则表达式
```

如果要更新记录数据库，可使用`updatedb`指令，它将根据`/etc/updatedb.conf`来查找系统硬盘内的文件名，并更新`/var/lib/mlocate`内的数据库文件。

## 3.3 find命令

`find`命令是Unix/Linux中非常常用的命令，它可以进行多种方式的搜索。其命令为：
```
$ find [PATH] [option] [action]
```

### 3.3.1 指定时间搜索

```
参数：
可接atime、ctime、mtime，以mtime为例
-mtime n: n是数字，表示n天前与n+1天前之间被修改过的文件
-mtime +n: 表示n天之前（不含n天本身）被修改过的文件
-mtime -n： 表示n天之内（包含那天本身）被修改过的文件
```

举例：查找当前目录下7天之前修改过的文件。
```
snowball@snowball-wang:~/code$ find . -mtime +7
.
./learningC
./learningC/List
./learningC/List/1.c
./learningLinux
./learningLinux/acc_iter.txt
./learningLinux/1.txt
./learningLinux/blog.txt
./learningLinux/result.txt
./learningLinux/acc_iter.py
./learningLinux/acc_iter.sh
./python
./python/test1.py
./python/readpara.py
./python/lfw_test.py
./python/autoLFWtest.sh
```

### 3.3.2 指定用户或用户组名搜索

```
参数：
- user name: name为用户账号名称，如snowball
- group name: name为用户组名，如users
- nouser: 寻找文件的所有者不存在/etc/passwd的人
```
举例：`find . -user snowball`查找当前目录下用户名为snowball的文件；`find . -group root`查找当前目录下用户组名为root的文件；`find . -nouser`查找/etc/passwd里不存在的用户的文件。
```
snowball@snowball-wang:~/code$ find . -user snowball
.
./learningC
./learningC/List
./learningC/List/1.c
./learningLinux
./learningLinux/acc_iter.txt
./learningLinux/blog_01_12.txt
./learningLinux/1.txt
./learningLinux/blog.txt
./learningLinux/result.txt
./learningLinux/acc_iter.py
./learningLinux/acc_iter.sh
./python
./python/test1.py
./python/readpara.py
./python/autoLFWtest.sh
snowball@snowball-wang:~/code$ find . -group root
./python/lfw_test.py
snowball@snowball-wang:~/code$ find . -nouser
snowball@snowball-wang:~/code$ 
```
### 3.3.3 指定文件权限及名称搜索

```
参数：
-name filename: 查找文件名为filename的文件
-size [+-]SIZE: 查找比SIZE还要大（+）或小（-）的文件。"-size +50k"的含义为找比50KB还要大的文件
-type TYPE: TYPE主要有f（一般文件）、b、c（设备文件）、d（目录）、l（连接文件）、s（socket）等
-perm mode: 查找文件权限恰好等于"mode"的文件
-perm -mode: 查找文件权限"必须要包括mode的权限"的文件
-perm +mode: 查找文件权限"包含任一mode的权限"的文件
```

### 3.3.4 其它可执行的操作

find的特殊功能就是能够进行额外的动作（action）。
```
$ find / -perm 7000 -exec ls -l {} \;
```
`{}`代表的是`find / -perm 7000`搜索到的内容，其结果会被放在`{}`内。`-exec`和`\;`中间的内容是find命令的额外命令，上述中就是`ls -l {}`。

**练习：**
1. 找出/etc下面，文件大小介于50KB到60KB之间的文件，并且将权限完整地列出来
2. 找出/etc下面，文件容量大于50KB且文件所有者不是root的文件名，且将权限完整地列出来
3. 找出/etc下面，容量大于1500KB以及容量等于0的文件

**答案：**
1. `find /etc -size +50k -size -60k -exec ls -l {} \;`
2. `find /etc -size +50k -a ! -user root -exec ls -l {} \;`
3. `find /etc -size + 1500k -o -size 0k`


**参考链接：**
1. http://man.linuxde.net/ls
2. http://blog.csdn.net/qq_38880380/article/details/78356372

