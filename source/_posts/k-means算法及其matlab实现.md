---
title: k-means算法及其matlab实现
date: 2018-04-25 20:26:53
tags:
- 技术日志
- 机器学习
- matlab
- k-means
categories:
- 技术日志
---

**关键字：** K-means算法



# 0. 前言

K-means算法是常见的聚类算法。研一的时候在吴桂兴老师的"信息论"课上接触过K-means算法，但并没有深入理解，也没有自己写代码去实现。最近在看[斯坦福CS231n课程](http://cs231n.stanford.edu/syllabus.html)的lecture2里又涉及到K-means算法，正好也想把Coursera里吴恩达机器学习课程里的代码复习一遍，以此文作为学习总结。



<!-- more -->

# 1. 聚类算法概述

在机器学习算法中，按照**学习方式**来划分，分为**有监督学习**、**无监督学习**、**半监督学习**、**强化学习**。其中，在**无监督学习（supervised learning）**中，训练的样本的标记信息是未知的，我们的目标是对无标记训练样本的学习来揭示输入数据的内在性质和规律，从而可对数据进行下一步的分析。**"聚类算法"**是**无监督学习**中经常使用的算法。

那么**聚类**是如何实现的了？通常来说，聚类是将数据集中的样本划分为若干个不相交的子集，每个子集称为一个**簇（cluster）**，并且每个子集可能都对应于一些潜在的类别。举个例子，某招聘会上有一群毕业生，有的是"交大毕业生"，有的是"科大毕业生"，有的是"上大毕业生"，这三个子集互不相交，且都是这群毕业生的组成部分。但我们需要注意的是，这些类别，或者说聚类产生的子集的概念，对于聚类算法而言事先是未知的。聚类过程仅能自动形成簇结构，簇所对应的概念语义需要人为的把握和命名。

聚类算法既能够作为一个单独的过程，用于寻找数据内在的分布结构，也可作为分类等其他学习任务的前驱过程。举个例子，在一些商业应用中，我们需要对新用户的用户类型进行判别，然而"用户类型"的概念很难全面准确地被商家定义，于是我们可以针对已有的用户数据进行聚类，将聚类结果的每一个簇定义为一个类，然后再根据这些类来训练分类模型，达到判别新用户的类型的目标。



# 2. k-means算法

## 2.1 k-means算法概述

k-means算法，也称为k均值聚类算法，是最广泛使用的聚类算法。它实现的是，将数据集中的n个点划分到k个聚类中，使得每个点都属于离此点最近均值点所对应的聚类。k-means算法优点在于简单、快速，但其缺点也很明显。使用k-means算法就必须要求事前给出k值，也就是预先确定好想要把数据集分成几类。其次，不同的初始化点，最后通过k-means得出的聚类结果也有可能产生差异。最后一点，k-means对于"噪声点"是极其敏感的，可能极少的"噪声点"都会对最后的结果产生很大的影响。

## 2.2 k-means算法流程

![k-means算法流程图](/k-means算法及其matlab实现/k-means算法流程.jpg)

**算法流程说明：**第1行对均值向量进行初始化，再第4-8行与第9-16行依次对当前簇划分及均值向量迭代更新，若迭代更新后聚类结果保持不变，则在第18行将当前簇划分结果返回。



# 3. matlab代码实现

在Coursera的吴恩达的机器学习的课程作业里，ex7中有关于k-means的matlab实现。我将其代码进行合并和改进，源码如下：

```matlab
function kmeans()

clear; close all; clc;
fprintf('\nRunning K-Means clustering example dataset.\n\n');

%加载数据集
X = load('data.mat');
X = cell2mat(struct2cell(X));

%设定K值和最大迭代次数
K = 3;
max_iter = 10;


initial_centroids = kMeansInitCentroid(X, K);
fprintf('The initial centroids is:\n');
fprintf(' %f %f \n', initial_centroids);
[centroids, idx] = runkMeans(X, initial_centroids, max_iter, true);
fprintf('The final centroids: \n');
fprintf(' %f %f \n', centroids);
fprintf('The class of the first 3 points: \n');
fprintf(' %d \n', idx(1:3));
fprintf('\nK-Means Done.\n\n');

end

% 实现两个坐标点的连线
function drawLine(p1, p2)
plot([p1(1) p2(1)],[p1(2) p2(2)]);
end
% 初始化K个中心点
function centroids = kMeansInitCentroid(X, K)

centroids = zeros(K, size(X, 2));
% 将1-n个数随机排列
randidx = randperm(size(X, 1));
centroids = X(randidx(1:K), :);

end

% 找到最近的中心点
function index = findClosestCentroids(X, centroids)
index = zeros(size(X,1), 1);
length = size(X,1);
K = size(centroids, 1);
distance = zeros(1,K);
for i=1:length
    for j=1:K
        distance(j) = norm(X(i,:) - centroids(j,:));
    end
    [~, index(i)] = min(distance);
end
end

% 更新中心点
function centroids = computeCentroids(X, index, K)
centroids = zeros(K, size(X,2));

for i=1:K
    idx = find(index==i);
    number = length(idx);
    centroids(i,:) = sum(X(idx, :)) / number;
end
end

% 绘制k-means迭代图
function plotProgresskMeans(X, centroids, previous, idx, K, i)

% 创建画板，将颜色和类别一一映射
palette = hsv(K);
colors = palette(idx,:);

% 绘制散点图
scatter(X(:,1),X(:,2),15,colors);

% 将中心点绘制成黑色的'x'
plot(centroids(:,1), centroids(:,2), 'x', 'MarkerEdgeColor', 'k', 'MarkerSize', 10, 'LineWidth', 3);

% 绘制中心点更新的轨迹
for j=1:size(centroids, 1)
    %plot([centroids(j,:)(1), previous(j,:)(1)],[centroids(j,:)(2), previous(j,:)(2)]);
    drawLine(centroids(j,:), previous(j,:));
end
title(sprintf('Iteration number %d',i));
end

% 实现k-means
function [centroids, idx] = runkMeans(X, initial_centroids, max_iter, plot_progress)
if ~exist('plot_progress', 'var') || isempty(plot_progress)
    plot_progress = false;
end

if plot_progress
    figure;
    hold on;
end

% 初始化
[m, ~] = size(X);
K = size(initial_centroids, 1);
centroids = initial_centroids;
previous_centroids = initial_centroids;
idx = zeros(m, 1);


% 运行k-means
for i=1:max_iter
    fprintf('K-Means iteration %d/%d...\n', i, max_iter);
    idx = findClosestCentroids(X, centroids);
    
    if plot_progress
        plotProgresskMeans(X, centroids, previous_centroids, idx, K, i);
        previous_centroids = centroids;
        fprintf('Press enter to continue.\n')
        pause;
    end
    centroids = computeCentroids(X, idx, K);   
end

if plot_progress
    hold off;
end
end
```

在matlab命令行中敲入`run kmeans()`命令，即可执行脚本程序。

实验的效果图如下：

![1](/k-means算法及其matlab实现/1.jpg)

![2](/k-means算法及其matlab实现/2.jpg)

![3](/k-means算法及其matlab实现/3.jpg)

![4](/k-means算法及其matlab实现/4.jpg)

![5](/k-means算法及其matlab实现/5.jpg)

![10](/k-means算法及其matlab实现/10.jpg)

观察图可以看出，中心点在迭代第四次之后便不再更新，所以第四次迭代之后的图便不再发生变化。

对应的代码和数据集的下载链接：https://pan.baidu.com/s/14yIoSxEiIeir2VaRR2qJlQ 密码：ts8l。

**参考文献：**

1. 周志华 《机器学习》
2. https://www.coursera.org/learn/machine-learning
3. https://blog.csdn.net/weiyongle1996/article/details/77925325



