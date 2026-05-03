---
title: 新版My CS4 Challenge
date: 2020-04-12 23:33:58
tags:
- 日常规划
- 自我学习
categories:
- CS4挑战
---

**关键词：** MIT挑战  自我学习  CS4挑战



# 0.前言

又看了一遍以前写的[My CS4 Challenge](http://wangjieqiang.com/2018/09/28/My-CS4-Challenge/)，才觉得当初制定的目标是多么的不切实际。当然也要考虑到当时的境遇，那种急迫地想要通过做事情来证明自己还行的想法，潜意识里高估了自己执行计划的能力。当发现事情的发展完全不是预期的那样，自己就会变得很佛系，就会安慰自己：做人嘛，不要给自己太大压力:)。

原始版的[My CS4 Challenge](http://wangjieqiang.com/2018/09/28/My-CS4-Challenge/)的一个不足之处在于，计划的拟定完全参照[MIT 挑战](https://www.scotthyoung.com/blog/myprojects/mit-challenge-2/)和自己在研究生阶段的学习兴趣。但是在工作了大半年之后才发现，只有把工作的内容和学习的计划结合起来，自己才会有更有动力去执行这些计划。否则按照原始计划去学习*CS139P Developing Apps for IOS*，我不相信自己真的有时间有精力去完成这门课程。

认真思考了一下，以前的我是想通过这些美帝高校的公开课资源来搭建自己的计算机基础框架，方便自己秋招春招找工作。现在的需求是想要搭建自己的技能树，让自己的职业道路走得更顺一些。目前来看（至少五年内）自己的技术领域方向还是在**网络**、**数据中心**、**云计算**、**虚拟化**、**Arm架构**、**Linux内核**等等。所以做个简单的排除法，之前的课程列表里诸如*CS231n Convolutional Neural Networks for Visual Recognition*之类的课就可以剔除了，毕竟自己从事的方向基本与AI不相关了。而且课程列表里的有些课程的内容是重叠的，如果真的能够把握一门课的精髓，诸如老老实实地把CMU的*15-213 Introduction to Computer Systems*过一遍，我认为是没有必要再去看MIT的*6.004 Computer Structure*了。当初课程列表里放这么多课程可能是让自己有个心理满足感，但新一版的课程列表必须更加精简，更加符合我学习的实际情况。

其实再回头一想，当初毕业后来Arm做OSS也是机缘巧合，冥冥之中感觉这是自己喜欢的方向。但是自己确实基础薄弱，开始工作的前几个月不说是以泪洗面，也感到特别强烈的挫折感。这版新的CS4挑战，给自己一个3~5年的时间，配合着工作中的积累，我相信还是基本满足工作上的需求的。

<!-- more -->

# 1. 正文

## 1.1 新版CS4挑战的目标

在Arm工作大半年了，扪心自问一下，自己在技术领域有了多少的长进？是否按照设定好的想法和规划向前推进？进步当然是有的，我想去年的自己可能还很害怕使用`gdb`调试程序，今年可以用`gdb`来找bug了。去年对计算机体系一无所知，今年的CSAPP已经进展到第五章了，而且完成了前四个实验，今年肯定要把这门课学完。不足还有很多，计算机网络的知识学习迟迟没有行动起来，基本的数据结构和算法掌握不到位，工作中听到的很多操作系统的概念都不甚了解，这些都需要自己花费大量的时间与精力去学习。更新CS4挑战的目的正是通过精选的课程学习，系统化地完成相关知识体系的构建和巩固。

1. 语言：完成Shell中Bash Script的学习、巩固C/C++/Python语言，包括GCC相关的延展特性，Python相应的测试框架（Robot Framework）。
2. 数学：完成计算机领域中的基本数学概念学习。
3. 计算机基础和体系：数据结构与算法、计算机体系结构、计算机网络、操作系统，建立起计算机自底向上的认知视角。
4. 实际应用：云计算、Linux内核开发等，辅助理解工作中的VPP项目。

## 1.2 新版CS4挑战内容

### 1.2.1 课程列表

**MIT：**

1. ~~6.0001 Introduction to Computer Science Programming in Python~~
2. ~~6.0002 Introduction to Computational Thinking and Data Science~~
3. 6.006 Introduction to Algorithms
4. 6.042 Mathematics for Computer Science 
5. ~~6.087 Practical Programming in C~~
6. 6.033 Computer System Engineering
7. 6.S081/6.828 Operating System Engineering
8. 6.824 Distributed System

**CMU:**

1. 15-213 Introduction to Computer Systems
2. 15-441 Computer Networks
3. 15-418 Parallel Computer Architecture and Programming

**Stanford & UCB:**

1. CS143 Compilers

### 1.2.2 课程关系

![learning path](/course_graph.PNG)

**注：**绿色表示已完成课程，黄色表示正在进行课程，白色表示还未开始课程。

### 1.2.3 新版CS4挑战计划细节

不像原版计划把计划订的很僵硬，对工作的我来说，**我必须保持自己对于学习新技术的热情和坚持**。自己的学习习惯还是先看视频，再过一遍课程PPT，最后完成项目assignment或者project的学习效果最好。根据[今年的计划](http://wangjieqiang.com/2020/01/01/2019%E5%B9%B4%E5%B9%B4%E7%BB%88%E5%A4%8D%E7%9B%98/)，把图标的黄色课程变成绿色，就已经非常不错啦^。^

### 1.2.4 新版CS4挑战外的事宜

同以前一样，CS4挑战只是我生活中的一部分。首先肯定还是以工作为主，保住饭碗才是硬道理。其次也包括下半年的半程马拉松破2h计划、今年的结婚安排、买房子的事情。同理，还是要**锻炼自己上下文切换的能力**，向前方继续走吧。

# 2. 愿景

新版的CS4计划是自己在工作大半年后又一次的整理和规划，只是希望自己还是按照自己的节奏继续推进。确认好自己的技术路线和方向，确定好自己要学的课程和内容，那就要落到实处。人世间难得的不是天衣无缝的计划，而是**知行合一**的脚步。

借以在CSAPP第四章中的一句话结束。愿自己对技术的热情始终饱满如初。

>There is an intrinsic value in learning how things work.

# 3. 更新

* 2022/10/10 updated: 完成MIT 6.087，更换UCB CS124为MIT 6.S081，添加MIT 6.824课程，并更新课程关系表。