---
title: 图像特征之SUSAN角点检测
date: 2017-07-01
categories: Algorithm
tags: Image
---
&emsp;&emsp;SUSAN(Small univalue segment assimilating nucleus)是一种基于灰度图像以及窗口模板的特征点获取方法，适用于图像中边缘和角点的检测，对噪声鲁棒，而且具有简单、有效、计算速度快等特点。
<!-- more -->

### 原理概述
&emsp;&emsp;SUSAN算子采用一种近似圆形的滑动窗口模板，邻域窗口内的每个像素点的灰度值和中心像素作比较，若两者的灰度差值小于一定阈值，则认为该像素点与中心像素(核)具有相似的灰度值，满足这一条件的像素组成的区域称为吸收核同值区(Univalue Segment Assimilating Nucleus, USAN)。

<img src="https://i.loli.net/2017/07/12/5965d644ac5c4.jpg" alt="different usan.jpg" title="模板在不同位置的USAN变化" />

如上图所示，当圆形模板处于灰度均匀区域(背景或目标内)，USAN区域面积最大；当模板移向图像边缘时，USAN面积逐渐变小，模板中心处于边缘位置时，USAN面积为最大值的1/2；当模板中心位于角点处时，USAN面积最小，约为最大值的1/4。因此，USAN面积越小，其中心像素为角点的概率就越大。通过计算每个像素的USAN值，并与给定阈值作比较，如果该像素的USAN值小于给定阈值，则认为是一个角点。USAN的三维显示如下图所示：

<img src="https://ooo.0o0.ooo/2017/07/12/5965d644d4320.jpg" alt="usan.jpg" title="USAN三维显示" />


### 算法步骤
- 定义一个半径为r(r=3)的圆形滑动模板，比较模板内像素与中心像素的灰度值差异，构成USAN区域。公式定义如下：
$$c(\overrightarrow{r}, \overrightarrow{r\_0}) = \begin{cases} 1,&emsp;|I(\overrightarrow{r}) - I(\overrightarrow{r\_0})| \leq t \\\ 0,&emsp;|I(\overrightarrow{r}) - I(\overrightarrow{r\_0})| > t\end{cases}$$
式中，$r\_0$表示模板核(中心像素)在图像中位置，$r$则是模板内其他像素的位置，$I(r)$表示图像灰度值。为了得到更稳定的结果，避免相似度函数在阈值边界处发生突变，亦可采用下式计算：
$$c(r,r\_0)=e^{-(\dfrac{I(r)-I(r\_0)}{t})^6}$$
- 计算USAN区域的面积
$$n(r\_0) = \Sigma\_r c(r, r\_0)$$
- 计算角点响应值
$$R(r\_0) = \begin{cases} g - n(r\_0),&emsp;n(r\_0) < g \\\ 0,&emsp;&emsp;&emsp;&emsp; n(r\_0) \geq g \end{cases}$$
阈值可取$g=n\_{max}/2$，USAN面积达到最小时，角点响应值达到最大。
- 在邻域内对角点响应值做非极大值抑制

### 参数分析
&emsp;&emsp;SUSAN算子采用的是圆形模板，窗口半径为r=3，窗口内包含37个像素，如下图所示：

<img src="https://i.loli.net/2017/07/12/5965d644c2903.png" alt="template.png" title="圆形模板" />

在进行角点检测时，需要确定两个重要的参数——g值和t值。阈值g决定了USAN区域面积的最大值，以及所检测角点的尖锐程度，g值越小，检测到的角点越尖锐。阈值t表示所能检测角点的最小对比度，决定了角点提取的数量，t值越小，可提取的角点数量越多。对于不同对比度和噪声的图像，应取不同的阈值。

### 算法特点
- 在对边缘和角点进行检测时，不涉及微分操作，因此对噪声的鲁棒性较好
- SUSAN算子比较的是邻域像素的灰度相似性，具有光强不变性、旋转不变性；而且检测算子不依赖模板尺寸，在一定程度具备尺度不变性
- 参数较少，计算较快，抗干扰能力强

### reference
- [Paper: SUSAN — A New Approach to Low Level Image Processing](http://www-2.dc.uba.ar/materias/ipdi/smith95susan.pdf)
- http://www.cnblogs.com/luo-peng/p/5615359.html
- http://blog.csdn.net/tostq/article/details/49305615
- http://xandl.cn/2017/03/23/extraction/