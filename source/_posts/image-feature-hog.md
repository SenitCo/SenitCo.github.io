---
title: 图像特征提取之HOG特征
date: 2017-06-10
categories: Algorithm
tags: Image
---

&emsp;&emsp;方向梯度直方图(Histogram of Oriented Gradient, HOG)特征是一种在计算机视觉和图像处理中用来进行物体检测的特征描述子。它通过计算和统计图像局部区域的梯度方向直方图来构成特征。Hog特征结合SVM分类器已经被广泛应用于图像识别中，尤其在行人检测中获得了极大的成功。
<!-- more -->

### 算法思想
&emsp;&emsp;HOG特征的核心思想是在一幅图像中，局部目标的表象和形状(appearance and shape)能够被梯度和边缘的方向密度（梯度的统计信息，而梯度主要存在于边缘地方）很好地描述。通过将整幅图像分为多个小的连通区域(cells)，并计算每个cell的梯度或边缘方向直方图，这些直方图的组合可用于构成特征描述子，为了提高准确率，可以将局部直方图在图像更大范围内(称为block)进行对比度归一化(constrast-normalized)。所采用的方法是：先计算各直方图在对应的block中的密度，然后根据这个密度对block中的所有cell做归一化(normalize)。归一化操作对光照变化和阴影具有更好的鲁棒性。

### 算法特点
- HOG特征是在图像的局部操作，对图像几何和光学的变化有较好的稳健性，这两种变化只会出现在更大的空域上。
- 在粗粒度的空域抽样、细粒度的方向抽样，以及较强的局部光学归一化条件下，只要行人大体保持直立的姿势，可以容许行人有一些细微的肢体动作，而不影响检测效果。

### 算法实现
HOG特征提取的流程如下：  
#### 图像预处理  
- 灰度化：HOG提取的是纹理特征，颜色信息不起作用，所以将彩色图转化为灰度图。
- Gamma校正(归一化)：对图像进行Gamma校正，完成对整个图像的标准化(归一化)，可以调节图像的对比度，降低局部光照不均匀或者阴影的影响，同时也可以在一定程度上降低噪声的干扰，提高特征描述器对光照等干扰因素的鲁棒性。校正公式如下：
 $$I(x, y) = I(x, y)^{\gamma}, \gamma = \dfrac{1}{2}$$

#### 计算图像梯度
&emsp;&emsp;分别求取图像水平方向和垂直方向的梯度，然后计算每个像素点的梯度幅值和方向，微分求图像梯度不仅可以捕获图像边缘和纹理信息，而且可以弱化光照不均匀的影响。
$$G\_x(x,y) = I(x+1,y) - I(x-1,y)$$
$$G\_y(x,y) = I(x,y+1) - I(x,y-1)$$
$$\nabla G(x,y) = \sqrt{G\_x(x,y)^2+G\_y(x,y)^2}$$
$$theta(x,y) = arctan(G\_y(x,y) / G\_x(x,y))$$
一般采用梯度算子对图像进行卷积运算求取图像梯度，例如用$[-1,0,1]$梯度算子对图像进行卷积操作得到水平方向的梯度分量，用$[-1,0,1]^T$梯度算子进行卷积操作得到竖直方向的梯度分量，然后求取图像的梯度幅值和方向。

#### 在cell中计算梯度方向直方图(Orientation binning)
&emsp;&emsp;将图像划分为若干个连通区域(cell)，例如每个cell为$8\times 8$个像素，相邻cell之间不重叠，将所有梯度方向划分为9个方向块(bin)，然后在每个cell内统计梯度方向直方图。在计算梯度方向时，可把方向的角度范围定位$(0, 180^{\circ})$或者$(0, 360^{\circ})$。最后每个cell都对应一个9维的特征向量。此外，还可以考虑梯度幅值作为bin的统计权重。
&emsp;&emsp;在行人检测中，通过给局部图像区域进行编码，可以保持对目标对象的姿势和外观的弱敏感性，更好地捕获图像的轮廓和纹理信息。

#### 在block中归一化梯度方向直方图(Block Normalization)
&emsp;&emsp;将多个cell组合成更大连通块(block)，将block内所有cell的特征向量串联起来便得到该block的HOG特征描述子，不同block之间可能相互重叠，可以有效地利用局部邻域信息。类比在卷积神经网络(CNN)中，掩码(Kernel)和步长(stride)的选择。在更大范围内(block)统计梯度直方图，并做归一化处理，能够更好地适应光照和对比度的变化。常用的归一化方法有以下几种：
- L2-norm
$$v = \dfrac{v}{\sqrt{\left|v\right\|\_{2}^{2} + \varepsilon^2}}$$
- L1-norm
$$v = \dfrac{v}{\left|v\right\|\_{1} + \varepsilon}$$
- L1-sqrt
$$v = \sqrt{\dfrac{v}{\left|v\right\|\_{1} + \varepsilon}}$$    
还有一种L2-Hys，即先做一次L2-norm,然后把大于特定值(0.2)的分量幅值为0.2再做一次L2-norm，一般在检测中采用L2-norm效果更好。在一个block中，如果cell的数量为$2\times 2$，那block的特征数为$2\times 2 \times 9 = 36$维特征。

#### 统计整幅图像(检测窗口)的HOG特征
&emsp;&emsp;在实际应用中，通常是选取固定大小的滑动窗口来提取HOG特征，对于一个$64\times 128$的图像窗口(window)，每$8\times 8$个像素组成一个cell，每$2\times 2$个cell组成一个block，一共有$(8-1)\times (16-1) = 105$个block，因此该图像的窗口特征维数为$105\times 36 = 3780$。当然也可以将整幅图像作为一个窗口来提取HOG特征。

#### HOG特征 + SVM分类器进行行人检测
&emsp;&emsp;训练过程中正样本为图片中包含有目标区域(行人)的boundingbox，尺寸统一为检测窗口的大小即$64\times 128$，负样本不需要统一尺寸，只需比检测窗口大，且图片中不包含检测目标，可任意截取图片中$64\times 128$大小的区域提取HOG特征作为负样本的特征向量，并与正样本图片中boundingbox区域提取出的HOG特征向量一起训练，得到SVM的分类模型。
&emsp;&emsp;检测过程中采用滑动窗口法，检测窗口尺寸固定不变，对待检测图片进行尺度缩放，在每一层的图像上，用固定大小的滑动窗口提取HOG特征，并根据训练好的分类模型判断检测窗口是否为目标(行人)。因此HOG + SVM进行行人检测的过程实际上就是对图像的检测窗口提取HOG特征进行分类判决的过程。  

### reference
- [paper: Histograms of Oriented Gradients for Human Detection](http://lear.inrialpes.fr/people/triggs/pubs/Dalal-cvpr05.pdf)
- [wikipedia: Histogram of oriented gradients](https://en.wikipedia.org/wiki/Histogram_of_oriented_gradients)
- http://blog.csdn.net/zouxy09/article/details/7929348
- http://blog.csdn.net/hujingshuang/article/details/47337707
- http://www.cnblogs.com/tornadomeet/archive/2012/08/15/2640754.html
- http://shuokay.com/2016/07/18/hog/
- http://www.jianshu.com/p/6f69c751e9e7
- http://blog.sina.com.cn/s/blog_60e6e3d50101bkpn.html





