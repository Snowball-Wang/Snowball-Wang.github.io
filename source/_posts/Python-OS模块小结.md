---
title: Python OS模块小结
date: 2019-04-23 21:47:41
tags:
- 技术日志
- Python
categories:
- 技术日志
---

**关键字：** Python  OS模块

# 0.前言

前些日子在看PyQt5的项目代码时，遇到了使用OS模块实现目标文件夹中的文件遍历的功能。接下来的工作也会接触到与操作系统相关的操作，本文就是对Python的OS模块中相关函数的小结。

<!-- more -->

# 1. 基本函数

首先通过一些样例代码了解OS模块。

导入OS模块：

```python
import os
```

查看OS模块中的函数列表：

```python
print(dir(os))
```

得到的输出结果：

```python
['DirEntry', 'F_OK', 'MutableMapping', 'O_APPEND', 'O_BINARY', 'O_CREAT', 'O_EXCL', 'O_NOINHERIT', 'O_RANDOM', 'O_RDONLY', 'O_RDWR', 'O_SEQUENTIAL', 'O_SHORT_LIVED', 'O_TEMPORARY', 'O_TEXT', 'O_TRUNC', 'O_WRONLY', 'P_DETACH', 'P_NOWAIT', 'P_NOWAITO', 'P_OVERLAY', 'P_WAIT', 'PathLike', 'R_OK', 'SEEK_CUR', 'SEEK_END', 'SEEK_SET', 'TMP_MAX', 'W_OK', 'X_OK', '_Environ', '__all__', '__builtins__', '__cached__', '__doc__', '__file__', '__loader__', '__name__', '__package__', '__spec__', '_execvpe', '_exists', '_exit', '_fspath', '_get_exports_list', '_putenv', '_unsetenv', '_wrap_close', 'abc', 'abort', 'access', 'altsep', 'chdir', 'chmod', 'close', 'closerange', 'cpu_count', 'curdir', 'defpath', 'device_encoding', 'devnull', 'dup', 'dup2', 'environ', 'errno', 'error', 'execl', 'execle', 'execlp', 'execlpe', 'execv', 'execve', 'execvp', 'execvpe', 'extsep', 'fdopen', 'fsdecode', 'fsencode', 'fspath', 'fstat', 'fsync', 'ftruncate', 'get_exec_path', 'get_handle_inheritable', 'get_inheritable', 'get_terminal_size', 'getcwd', 'getcwdb', 'getenv', 'getlogin', 'getpid', 'getppid', 'isatty', 'kill', 'linesep', 'link', 'listdir', 'lseek', 'lstat', 'makedirs', 'mkdir', 'name', 'open', 'pardir', 'path', 'pathsep', 'pipe', 'popen', 'putenv', 'read', 'readlink', 'remove', 'removedirs', 'rename', 'renames', 'replace', 'rmdir', 'scandir', 'sep', 'set_handle_inheritable', 'set_inheritable', 'spawnl', 'spawnle', 'spawnv', 'spawnve', 'st', 'startfile', 'stat', 'stat_float_times', 'stat_result', 'statvfs_result', 'strerror', 'supports_bytes_environ', 'supports_dir_fd', 'supports_effective_ids', 'supports_fd', 'supports_follow_symlinks', 'symlink', 'sys', 'system', 'terminal_size', 'times', 'times_result', 'truncate', 'umask', 'uname_result', 'unlink', 'urandom', 'utime', 'waitpid', 'walk', 'write']
```

通过`getcwd`函数，我们可获取当前的工作路径：

```python
print(os.getcwd())
```

得到的输出结果：

```python
C:\Users\jiewan01\test\MIT_6.0001_Introduction_to_CS_Using_Python-master
```



# 2. 列出目录与目录内的文件

可通过`listdir`函数列出当前目录下中的目录与文件。

```python
print(os.listdir())
```

得到的输出结果是：

```python
['ps0', 'ps1', 'ps2', 'ps3', 'ps4', 'ps5', 'README.md']
```

从输出的结果可以看出，在当前目录下，我一共有6个文件夹和1个`README.md`文件。

如果想要获取整个项目文件的目录树结构，我们通过使用`os.walk()`函数对当前路径下每个目录内的文件进行迭代，其代码如下所示：

```python
# -*- coding: utf-8 -*-
# 使用os.walk()遍历当前文件目录下的所有文件
def list_files(startpath):
    for root, dirs, files in os.walk(startpath):
        level = root.replace(startpath, '').count(os.sep)
        indent = ' ' * 4 * level
        print('{}{}/'.format(indent, os.path.basename(root)))
        subindent = ' ' 4 * (level + 1)
        for f in files:
            print('{}{}'.format(subindent, f))
```

在上述代码中，`os.walk()`函数返回的是包含迭代到当前的根目录、根目录下所有子目录、子目录中的所有文件的生成器。代码通过设置`indent`和`subindent`两个参数来实现目录层级的显示。`os.path.basename()`返回文件路径最后的文件名。调用`list_files`函数，实现对当前目录的遍历，代码为：

```python
startpath = os.getcwd()
list_files(startpath)
```

得到的输出结果：

```python
test/
    MIT_6.0001_Introduction_to_CS_Using_Python-master/
        README.md
        ps0/
            MIT6_0001F16_Getting_Start.pdf
            MIT6_0001F16_ProblemSet0.pdf
            pkgtest.py
            ps0.py
        ps1/
            MIT6_0001F16_ps1.pdf
            MIT6_0001F16_StyleGuide.pdf
            ps1a.py
            ps1b.py
            ps1c.py
        ps2/
            hangman.py
            MIT6_0001F16_Pset2.pdf
            words.txt
        ps3/
            MIT6_0001F16_ProblemSet3.pdf
            ps3.py
            test_ps3.py
            words.txt
        ps4/
            MIT6_0001F16_Pset4.pdf
            ps4a.py
            ps4b.py
            ps4c.py
            story.txt
            words.txt
        ps5/
            feedparser.py
            MIT6_0001F16_ps5.pdf
            mtTkinter.py
            my_own_triggers.png
            my_own_triggers.txt
            project_util.py
            ps5.py
            ps5_test.py
            test_is_phrase_in.py
            triggers.txt
```

从输出结果可以看到，每个目录后都以`/`结束，目录内的文件缩进4个空格。

# 3. 更改工作目录

我们通过`os.chdir`函数来更改工作目录，进入到`ps5`工作目录下。

```python
os.chdir('MIT_6.0001_Introduction_to_CS_Using_Python-master/ps5')
```

再次使用`list_files`函数，得到输出结果：

```python
ps5/
    feedparser.py
    MIT6_0001F16_ps5.pdf
    mtTkinter.py
    my_own_triggers.png
    my_own_triggers.txt
    project_util.py
    ps5.py
    ps5_test.py
    test_is_phrase_in.py
    triggers.txt
```



# 4. 创建单个或嵌套目录结构

我们在当前目录下创建一个名为`testdir`的新目录。

```python
os.mkdir('testdir')
```

再调用`list_files`函数显示当前目录下的所有文件，得到的输出结果为：

```python
ps5/
    feedparser.py
    MIT6_0001F16_ps5.pdf
    mtTkinter.py
    my_own_triggers.png
    my_own_triggers.txt
    project_util.py
    ps5.py
    ps5_test.py
    test_is_phrase_in.py
    triggers.txt
    testdir/
```

由输出结果可以看到，该函数在当前目录下创建了一个新目录。

我们再来创建一个2级的嵌套目录：

```python
os.mkdir('level1dir/level2dir')
```

得到的输出结果是：

```python
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
FileNotFoundError: [WinError 3] 系统找不到指定的路径。: 'level1dir/level2dir'
```

我们得到了一个`FileNotFoundError`的错误，原因在于Python会在创建`level2dir`之前先调用`level1dir`，而`level1dir`一开始就没有创建，由此抛出了`FileNotFoundError`错误。

想要创建多级嵌套目录，我们可使用`makekdirs()`函数，来递归地创建多级目录。

```python
os.makedirs('level1dir/level2dir')
```

再次调用`list_files`函数来显示当前目录的目录树结构，得到的输出结果为：

```python
ps5/
    feedparser.py
    MIT6_0001F16_ps5.pdf
    mtTkinter.py
    my_own_triggers.png
    my_own_triggers.txt
    project_util.py
    ps5.py
    ps5_test.py
    test_is_phrase_in.py
    triggers.txt
    level1dir/
       level2dir/
    testdir/
```

输出结果显示，再`ps5`目录下有我们创建的`level1dir`和`testdir`两个新的文件夹。在`level1dir`目录下有`level2dir`目录。



# 5. 递归地删除单个或多个目录

我们可通过`rmdir`指令删除目录：

```python
os.rmdir('testdir')
```

调用`list_files`函数严重目录是否已被删除，得到的输出结果是：

```python
ps5/
    feedparser.py
    MIT6_0001F16_ps5.pdf
    mtTkinter.py
    my_own_triggers.png
    my_own_triggers.txt
    project_util.py
    ps5.py
    ps5_test.py
    test_is_phrase_in.py
    triggers.txt
    level1dir/
        level2dir/
```

从输出结果可以看出，`testdir`已经被删除。

接下来我们删除`level1dir`的嵌套目录：

```python
os.rmdir('level1dir')
```

得到的输出结果是：

```python
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
OSError: [WinError 145] 目录不是空的。: 'level1dir'
```

输出结果抛出`OSError`错误，报错的信息说明`level1dir`目录不是空的，因为`level1dir`下有`level2dir`目录。同Unix系统类似，`os.rmdir`函数不能删除非空目录。

同`makedirs()`函数一样，我们使用`removedirs()`来实现目录树结构中目录的递归删除。

```python
os.removedirs('level1dir/level2dir')
```

再次调用`list_files`函数来显示目录树结构，得到的输出结果为：

```python
ps5/
    feedparser.py
    MIT6_0001F16_ps5.pdf
    mtTkinter.py
    my_own_triggers.png
    my_own_triggers.txt
    project_util.py
    ps5.py
    ps5_test.py
    test_is_phrase_in.py
    triggers.txt
```

当前显示的目录树结构与先前相同，没有发生变化。



# 6. 其它常用OS模块函数表

|          函数名          |                           函数含义                           |
| :----------------------: | :----------------------------------------------------------: |
|   os.rename(old, new)    |                   将old文件名重新命名为new                   |
|  os.path.abspath(file)   |                      获得file的绝对路径                      |
|  os.path.dirname(file)   | 若file是文件，则获取文件所在的目录；若file是目录，获取file目录的上一层目录 |
|  os.path.basename(file)  |             获取file所在文件路径最后的目录或文件             |
|   os.path.split(path)    |                 将path所在的路径与文件名分离                 |
| os.path.join(path, file) |               将path和file两者组合形成文件路径               |



# 7. 总结

Python的OS模块提供了与操作系统交互的接口，在与操作系统中的文件打交道时非常有用。记住本文中使用到的OS模块函数，已经足够的使用。



## 参考链接

1. https://docs.python.org/3/library/os.html
2. https://stackabuse.com/introduction-to-python-os-module/