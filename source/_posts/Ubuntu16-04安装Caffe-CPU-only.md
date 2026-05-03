---
title: Ubuntu16.04安装Caffe(CPU_only)
date: 2017-12-04 09:22:17
tags: 
- Ubuntu16.04
- Caffe
- 技术日志
categories:
- 技术日志
---

**关键词：** Ubuntu16.04  Caffe  CPU_only

# 0. 概述

在Linux系统下安装Caffe是一个很繁琐的过程，而且很有可能由于安装文档内容的欠缺或者用户在安装过程中的疏忽，将会导致各种各样令人崩溃的问题。我逛遍了CSDN等各大博客网站，尝试了各种版本的安装教程，几乎没有人把安装流程写得清晰明确。于是我决定把自己在Ubuntu16.04成功安装CPU_only版本的Caffe的流程记录下来，以便以后不时之需。虽然不是学习Caffe的最好版本（一般情况下肯定是GPU版本的Caffe的运行速度更快），但对于我等想要了解Caffe的初学者已经足够。

**推荐：**最好使用洁净版的Ubuntu16.04系统，这里是[安装教程](https://www.linuxtechi.com/install-ubuntu-16-04-with-screenshots/  "Ubuntu16.04安装教程")。

<!-- more -->

# 1. 安装OpenCV

Caffe需要图像处理库OpenCV，所以在安装Caffe之前我们需要安装OpenCV。

## 1.1 安装OpenCV所需包

* GCC 4.4.x或更新版本
* CMake 2.6或更高版本
* Git
* GTK+2.x或更新版本，包含头部（libgtk2.0-dev）
* pkg-config
* Python 2.6或者更新版本，Numpy 1.5或者更新版本，两者都需要开发者包（python-dev，python-numpy）
* ffmpeg或者libav开发工具包：libavcodec-dev，libavformat-dev，libswscale-dev
* [optional] libtbb2 libtbb-dev
* [optional] libdc1394 2.x
* [optional] libjpeg-dev，libpng-dev，libtiff-dev，libjasper-dev，libdc1394-22-dev

这些包的安装可通过在终端敲入以下命令实现：

```
sudo apt-get install build-essential
sudo apt-get install cmake git libgtk2.0-dev pkg-config libavcodec-dev libavformat-dev libswscale-dev
sudo apt-get install python-dev python-numpy libtbb2 libtbb-dev libjpeg-dev libpng-dev libtiff-dev libjasper-dev libdc1394-22-dev
```
## 1.2 获取OpenCV源代码

在1.1中我们已经完成了Git的安装，可在终端敲入以下命令将OpenCV的源代码克隆到本地：

```
cd ~/<相应的工作目录> #以我的为例，我的目录为/home/snowball
git clone https://github.com/opencv/opencv.git
```

## 1.3 使用CMake编译OpenCV

1. 创建一个临时文件夹（比如取名为<build>目录），这个文件夹会放置你编译产生的Makefiles、项目文件（project files）、目标文件（object files）以及输出二进制文件（output binaries）。
2. 进入<build>目录文件夹，敲入以下命令：
```
cmake [<可选参数>] <OpenCV源代码目录>
```
举例来说：

```
cd ~/opencv #以我的为例，我的目录是/home/snowball/opencv-master
mkdir build
cd build
cmake -D CMAKE_BUILD_TYPE=RELEASE -D CMAKE_INSTALL_PREFIX=/usr/local ..
```

3. 在`build`目录下（以我的为例，即为/home/snowball/opencv-master/build），执行以下命令：

```
make
sudo make install
```

**注意：** 如果上述第2步中的-D不起作用的话，可使用以下命令代替：
```
cmake -DCMAKE_BUILD_TYPE=RELEASE -DCMAKE_INSTALL_PREFIX=/usr/local ..
```



# 2. 安装Caffe依赖包

Caffe的安装依赖于以下命令行中的包，所以要在安装Caffe之前完成这些包的安装。一般来说安装这些包不会出现错误提示，但如果之前安装过Caffe导致操作系统留有破损的依赖包，就很有可能出现错误。如果出现错误，推荐使用洁净版的Ubuntu16.04从头开始。相关命令如下：

```
sudo apt-get install -y --no-install-recommends libboost-all-dev
sudo apt-get install libprotobuf-dev libleveldb-dev libsnappy-dev libopencv-dev libboost-all-dev libhdf5-serial-dev libgflags-dev libgoogle-glog-dev liblmdb-dev protobuf-compiler

sudo apt-get install libopenblas-dev
sudo apt-get install libatlas-base-dev
```

# 3. 克隆Caffe源代码

接下来需要从GitHub上将Caffe的源代码克隆到本地，终端中执行以下命令：

```
cd ~/<相应的工作目录> #以我的为例，我的目录是/home/snowball
git clone https://github.com/BVLC/caffe
cd caffe
cp Makefile.config.example Makefile.config
```

# 4. 安装Python包

这一步完成pycaffe的配置，终端中执行以下命令：

```
sudo apt-get install python-pip python-dev build-essential

sudo pip install scikit-image protobuf
cd python
for req in $(cat requirements.txt); do sudo pip install $req; done
```

# 5. 修改配置文件

由于我们是工作在一个CPU_only版本下的Caffe，我们必须通过修改配置文件来通知Caffe我们没有GPU。同时我们也要在配置文件里说明我们所使用的OpenCV的版本。打开配置文件：

```
cd .. #回到上一目录级
sudo gedit Makefile.config #用gedit打开配置文件
```

这时配置文件已经打开，我们把相应的注释部分给去掉：

```
# CPU-only switch (uncomment to build without GPU support).
CPU_ONLY := 1
...
# Uncomment if you're using OpenCV 3
OPENCV_VERSION := 3
```

同时，我们也需要将hdf5的库添加进INCLUDE_DIRS和LIBRARY_DIRS的变量中，这样会避免随后编译Caffe时出错。

Makefile.config原来的INCLUDE_DIRS和LIBRARY_DIRS:
```
# Whatever else you find you need goes here.
INCLUDE_DIRS := $(PYTHON_INCLUDE) /usr/local/include
LIBRARY_DIRS := $(PYTHON_LIB) /usr/local/lib /usr/lib
```

修改为：
```
# Whatever else you find you need goes here.
INCLUDE_DIRS := $(PYTHON_INCLUDE) /usr/local/include /usr/include/hdf5/serial
LIBRARY_DIRS := $(PYTHON_LIB) /usr/local/lib /usr/lib /usr/lib/x86_64-linux-gnu /usr/lib/x86_64-linux-gnu/hdf5/serial
```

# 6. 修改Makefile文件

接下来我们需要修改Makefile文件，打开Makefile文件：
```
sudo gedit Makefile
```

对文件内的两处进行如下修改：
将
```
NVCCFLAGS += -ccbin=$(CXX) -Xcompiler -fPIC $(COMMON_FLAGS) 
```
修改为：
```
NVCCFLAGS += -D_FORCE_INLINES -ccbin=$(CXX) -Xcompiler -fPIC $(COMMON_FLAGS)
```
将
```
LIBRARIES += glog gflags protobuf boost_system boost_filesystem m hdf5_hl hdf5
```
修改为：
```
LIBRARIES += glog gflags protobuf boost_system boost_filesystem m hdf5_serial_hl hdf5_serial
```

# 7. 对文件进行链接

```
sudo ln -s /usr/lib/x86_64-linux-gnu/libhdf5_serial.so.10.1.0 /usr/lib/x86_64-linux-gnu/libhdf5.so
sudo ln -s /usr/lib/x86_64-linux-gnu/libhdf5_serial_hl.so.10.0.2 /usr/lib/x86_64-linux-gnu/libhdf5_hl.so
```

**注意：**我的版本是`libhdf5_serial.so`对应的是10.1.0版本，`libhdf5_serial_hl.so`对应的是10.0.2版本。如果版本号和我的不同，则需调整相应的版本号。

# 8. 编译Caffe

终于到了最后一步--编译。返回到Caffe的根目录下，在终端执行以下命令：

```
make all
make test
make runtest
```

如果一切正常，将会看到同下图相同的结果：

![make_runtest_success](/Ubuntu16-04安装Caffe-CPU-only/make_runtest_success.png "Caffe编译成功")

# 9. 编译Caffe-Python接口

配置pycaffe，在终端中输入以下命令：

```
make pycaffe
echo export PYTHONPATH=/path/to/caffe/python:$PYTHONPATH >> ~/.bashrc
source ~/.bashrc
```

接下来在Python终端导入Caffe模块：
```
python
>>>import caffe
```

![import_success](/Ubuntu16-04安装Caffe-CPU-only/import_success.png "导入Caffe成功")

至此，Ubuntu16.04安装Caffe(CPU_only)完成。

**参考链接：**

1. https://mohanadkaleia.com/install-caffe-on-ubuntu-with-no-gpu/
2. https://docs.opencv.org/2.4/doc/tutorials/introduction/linux_install/linux_install.html