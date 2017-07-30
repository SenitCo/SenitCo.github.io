---
title: 图像特征之FAST角点检测
date: 2017-06-30
categories: Algorithm
tags: Image
---
&emsp;&emsp;前面已经介绍多种图像特征点(角点、斑点、极值点)的检测算法，包括Harris、LoG、HoG以及SIFT、SURF等，这些方法大多涉及图像局部邻域的梯度计算和统计，相比较而言，FAST(Features From Accelerated Segment Test)在进行角点检测时，计算速度更快，实时性更好。
<!-- more -->

### FAST角点定义
&emsp;&emsp;FAST角点定义为：若某像素点与周围邻域足够多的像素点处于不同区域，则该像素可能为角点。考虑灰度图像，即若某像素点的灰度值比周围邻域足够多的像素点的灰度值大或小，则该点可能为角点。

### 算法步骤
- 对于图像中一个像素点$p$，其灰度值为$I\_p$
- 以该像素点为中心考虑一个半径为3的离散化的Bresenham圆，圆边界上有16个像素(如下图所示)
- 设定一个合适的阈值$t$，如果圆上有n个连续像素点的灰度值小于$I\_p-t$或者大于$I\_p+t$，那么这个点即可判断为角点(n的值可取12或9)

<img src="https://ooo.0o0.ooo/2017/07/10/596386112222c.jpg" alt="circle.jpg" />

一种快速排除大部分非角点像素的方法就是检查周围1、5、9、13四个位置的像素，如果位置1和9与中心像素P点的灰度差小于给定阈值，则P点不可能是角点，直接排除；否则进一步判断位置5和13与中心像素的灰度差，如果四个像素中至少有3个像素与P点的灰度差超过阈值，则考察邻域圆上16个像素点与中心点的灰度差，如果有至少9个超过给定阈值则认为是角点。

### 角点分类器
- 选取需要检测的场景的多张图像进行FAST角点检测，选取合适的阈值n(n<12)，提取多个特征点作为训练数据
- 对于特征点邻域圆上的16个像素$x \in \{1,2,...,16 \}$，按下式将其划分为3类
$$S\_{p\rightarrow x} = \begin{cases} d, &emsp;I\_{p\rightarrow x} \leq I\_p-t \\\ s, &emsp;I\_p-t \leq I\_{p\rightarrow x} \leq I\_p+t \\\ b, &emsp;I\_p+t \leq  I\_{p\rightarrow x} \end{cases}$$
- 对每个特征点定义一个bool变量$K\_p$，如果$p$是一个角点，则$K\_p$为真，否则为假
- 对提取的特征点集进行训练，使用ID3算法建立一颗决策树，通过第$x$个像素点进行决策树的划分，对集合$P$，得到熵值为
$$H(P)=(c+\hat{c})log\_2 (c+\hat{c})-clog\_2 c - \hat{c}log\_2 \hat{c} $$
其中$c$为角点的数目，$\hat{c}$为非角点的数目。由此得到的信息增益为
$$\Delta H = H(P) - H(P\_d) - H(P\_s) - H(P\_b)$$
选择信息增益最大位置进行划分，得到决策树
- 使用决策树对类似场景进行特征点的检测与分类

### 非极大值抑制
&emsp;&emsp;对于邻近位置存在多个特征点的情况，需要进一步做非极大值抑制(Non-Maximal Suppression)。给每个已经检测到的角点一个量化的值$V$，然后比较相邻角点的$V$值，保留局部邻域内$V$值最大的点。$V$值可定义为
- 特征点与邻域16个像素点灰度绝对差值的和
- $V = max(\Sigma\_{x \in S\_{bright}} |I\_{p\rightarrow x} - I\_p| - t, \Sigma\_{x \in S\_{dark}} |I\_{p\rightarrow x} - I\_p| - t)$
式中，$S\_{bright}$是16个邻域像素点中灰度值大于$I\_p+t$的像素点的集合，而$S\_{dark}$表示的是那些灰度值小于$I\_p−t$的像素点。

### 算法特点
- FAST算法比其他角点检测算法要快
- 受图像噪声以及设定阈值影响较大
- 当设置$n<12$时，不能用快速方法过滤非角点
- FAST不产生多尺度特征，不具备旋转不变性，而且检测到的角点不是最优

### reference
- [Paper: Machine learning for high-speed corner detection](https://www.edwardrosten.com/work/rosten_2006_machine.pdf)
- [Paper: Faster and better: a machine learning approach to corner detection](https://pdfs.semanticscholar.org/a963/288ffecda4fd2bc475efe7cfb59ab094e7c1.pdf)
- http://www.cnblogs.com/ronny/p/4078710.html
- http://blog.csdn.net/hujingshuang/article/details/46898007
- http://blog.csdn.net/lql0716/article/details/65662648
- https://liu-wenwu.github.io/2016/10/08/fast-corners/
- http://blog.csdn.net/skeeee/article/details/9405531