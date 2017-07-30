---
title: 图像特征之SIFT特征匹配
date: 2017-06-24
categories: Algorithm
tags: Image
---
&emsp;&emsp;尺度不变特征变换(Scale-invariant feature transform, SIFT)是计算机视觉中一种检测、描述和匹配图像局部特征点的方法，通过在不同的尺度空间中检测极值点或特征点(Conrner Point, Interest Point)，提取出其位置、尺度和旋转不变量，并生成特征描述子，最后用于图像的特征点匹配。SIFT特征凭借其良好的性能广泛应用于运动跟踪(Motion tracking)、图像拼接(Automatic mosaicing)、3D重建(3D reconstruction)、移动机器人导航(Mobile robot navigation)以及目标识别(Object Recognition)等领域。
<!-- more -->

### 尺度空间极值检测
&emsp;&emsp;Harris特征进行角点检测时，不具备尺度不变性。为了能够在不同的尺度检测到尽可能完整的特征点或关键点，需要借助尺度空间理论来描述图像的多尺度特征。相关研究证明高斯卷积核是实现尺度变换的唯一线性核。因此可用图像的高斯金字塔表示尺度空间，而且尺度规范化的LoG算子具有尺度不变性，在具体实现中，可用高斯差分(DoG)算子近似LoG算子，在构建的尺度空间中检测稳定的特征点。

#### 构建尺度空间
&emsp;&emsp;尺度空间理论的基本思想是：在图像处理模型中引入一个被视为尺度的参数，通过连续变化尺度参数获取多尺度下的空间表示序列，对这些空间序列提取某些特征描述子，抽象成特征向量，实现图像在不同尺度或不同分辨率的特征提取。尺度空间中各尺度图像的模糊程度逐渐变大，模拟人在由近到远时目标在人眼视网膜上的成像过程。而且尺度空间需满足一定的不变性，包括图像灰度不变性、对比度不变性、平移不变性、尺度不变性以及旋转不变性等。在某些情况下甚至要求尺度空间算子具备仿射不变性。
&emsp;&emsp;一幅图像的尺度空间可定义为对原图像进行可变尺度的高斯卷积：
$$L(x,y,\sigma)=G(x,y,\sigma) \ast I(x,y)$$
$$G(x,y,\sigma)=\dfrac{1}{2 \pi \sigma^2} e^{-\dfrac{x^2+y^2}{2\sigma^2}}$$
式中，$G(x,y,\sigma)$是尺度可变的高斯函数，$(x,y)$是图像的空间坐标，$\sigma$是尺度坐标(尺度变化因子)，$\sigma$大小决定图像的平滑程度，值越大图像模糊得越严重。大尺度对应图像的概貌特征，小尺度对应图像的细节特征。一般根据$3\sigma$原则，高斯核矩阵的大小设为$(6\sigma+1) \times (6\sigma+1)$。
&emsp;&emsp;在使用高斯金字塔构建尺度空间时，主要分成两部分，对图像做降采样，以及对图像做不同尺度的高斯模糊。对图像做降采样得到不同尺度的图像，也就是不同的组(Octave)，后面的Octave(高一层的金字塔)为上一个Octave(低一层的金字塔)降采样得到，图像宽高分别为上一个Octave的1/2。每组(Octave)又分为若干层(Interval)，通过对图像做不同尺度的高斯模糊得到。为了有效地在尺度空间检测稳定的关键点，提出了高斯差分尺度空间(DoG Scale-Space)，利用不同尺度的高斯差分核与图像卷积生成。
$$D(x,y,\sigma)=(G(x,y,k\sigma)-G(x,y,\sigma)) \ast I(x,y) = L(x,y,k\sigma)-L(x,y,\sigma)$$
图像的高斯金字塔和高斯差分金字塔如下图所示，高斯差分图像由高斯金字塔中同一组(Octave)内相邻层(Interval)的图像作差得到。

<img src="https://ooo.0o0.ooo/2017/06/30/5956098d36da8.jpg" alt="Gaussian Pyramid.jpg" title="图像高斯金字塔" />

#### 尺度空间的参数确定
&emsp;&emsp;在由图像金字塔表示的尺度空间中，图像的组数(Octave)由原始图像的大小和塔顶图像的大小决定。
$$Octave = log\_2(min(width\_0, height\_0)) - log\_2(min(width, height)) $$
式中，$width\_0、height\_0$分别为原始图像的宽高，$width、height$为塔顶图像的宽高。对于一幅大小为$512 \times 512$的图像，当塔顶图像大小为$4 \times 4$时，图像的组数为$Octave = 7$。
尺度参数$\sigma$的取值与金字塔的组数和层数相关，设第一组第一层的尺度参数取值为$\sigma(1,1)=\sigma\_0$，则第$m$组第$n$层的$\sigma$取值为
$$\sigma(m,n)=\sigma\_0 \cdot 2^{m-1} \cdot k^{n-1}, &emsp;k=2^{\dfrac{1}{S}}$$
式中，$S$为金字塔中每组的有效层数。为了得到更多的特征点，将图像扩大为原来的两倍，原始图像的尺度参数为$\sigma=0.5$，扩大两倍后的尺度为$\sigma=1.6$。在检测极值点前对原始图像的高斯平滑会导致图像高频信息的丢失，所以在建立尺度空间前先将图像扩大为原来的两倍，以保留原始图像信息，增加特征点数量。

#### DoG算子检测极值点
&emsp;&emsp;为了寻找DoG尺度空间的极值点，每一个采样点要和其所有邻域像素相比较，如下图所示，中间检测点与其同尺度的$8$个邻域像素点以及上下相邻两层对应的$9 \times 2$个像素点一共$26$个点作比较，以确保在图像空间和尺度空间都能检测到极值点。一个像素点如果在DoG尺度空间本层及上下两层的26邻域中取得最大或最小值时，就可以认为该点是图像在该尺度下的一个特征点。

<img src="https://ooo.0o0.ooo/2017/06/30/59562da5c15d9.png" alt="extremum.png" title="DoG尺度空间极值检测" />

&emsp;&emsp;在极值比较的过程中，每一组差分图像的首末两层是无法比较的，为了在每组中检测$S$个尺度的极值点，则DoG金字塔每组须有$S+2$层图像，高斯金字塔每组须有$S+3$层图像。另外，在降采样时，高斯金字塔中一组(Octive)的底层图像是由前一组图像的倒数第3张图像(S+1层)隔点采样得到。这样也保证了尺度变化的连续性，如下图所示：

<img src="https://ooo.0o0.ooo/2017/06/30/59562da5c399e.png" alt="parameter.png" title="尺度参数的变化" />

假设每组的层数$S=3$，则$k=2^{1/S}=s^{1/3}$，在高斯金字塔中，第一个Octave中第$S+1$层图像尺度为$k^3\sigma=2\sigma$，经降采样后得到第二个Octave的第$1$层图像，尺度仍为$2\sigma$。在DoG尺度空间中，第一组(1st-Octave)图像中间三项的尺度分别为$(k\sigma, k^2\sigma, k^3\sigma)$，下一组中间三项为$(2k\sigma, 2k^2\sigma, 2k^3\sigma)$，其“首项”$2k\sigma=2^{4/3}\sigma$，与上一组“末项”$k^3\sigma=2^{3/3}\sigma$尺度变化连续，变化尺度为$k=2^{1/S}=2^{1/3}$。

### 关键点定位
&emsp;&emsp;在DoG尺度空间检测到的极值点是离散的，通过拟合三元二次函数可以精确定位关键点的位置和尺度，达到亚像素精度。同时去除低对比度的检测点和不稳定的边缘点，以增强匹配稳定性，提高抗噪声能力。
#### 关键点精确定位
&emsp;&emsp;DoG函数$D(X)=D(x,y,\sigma)$在尺度空间的的Taylor展开式为
$$D(X)=D+\dfrac{\partial D^T}{\partial X} X + \dfrac{1}{2} X^T \dfrac{\partial^2 D}{\partial X^2}X$$
令$D(X)$导数为0，得到极值点的偏移量
$$\hat {X} = -(\dfrac{\partial^2 D}{\partial X^2})^{-1} \dfrac{\partial D}{\partial X}$$
若$\hat {X}=(x,y,\sigma)^T$在任意一个维度大于$0.5$，说明极值点精确位置距离另一个点更近，应该改变当前关键点的位置，定位到新点后执行相同操作，若迭代5次仍不收敛，则认为该检测点不为关键点。精确关键点处函数值为
$$D(\hat {X})=D+\dfrac{1}{2} \dfrac{\partial D^T}{\partial X} \hat {X}$$
$|D(\hat {X})|$过小易受噪声点的干扰而变得不稳定，若其小于某个阈值(例如$0.03$或者$0.04/S$)，则将该极值点删除。

#### 消除边缘效应
&emsp;&emsp;高斯差分函数DoG有较强的边缘响应，需要剔除不稳定的边缘响应点。边缘点的特征表现在某个方向有较大的主曲率，而在与其垂直方向主曲率较小。主曲率可通过一个$2 \times 2$的$Hessian$矩阵求出。
$$H=\left( \begin{matrix} D\_{xx}& D\_{xy} \\\ D\_{xy} & D\_{yy}\end{matrix} \right)$$
$D$的主曲率和$H$的特征值成正比，令$\alpha$为较大特征值，$\beta$为较小特征值，且$\alpha / \beta = r$，则
$$Tr(H)=D\_{xx}+D\_{yy}=\alpha + \beta, &emsp;Det(H)=D\_{xx}D\_{yy}-D\_{xy}^2=\alpha \beta$$
$$\dfrac{Tr(H)^2}{Det(H)}=\dfrac{(\alpha + \beta)^2}{\alpha \beta} = \dfrac{(r+1)^2}{r}$$
$(r+1)^2/r$在两个特征值相等时最小，随着$r$的增大而增大，$r$值越大，说明两个特征值的比值越大，正好对应边缘的情况。因此，设定一个阈值$r\_t$，若满足
$$\dfrac{Tr(H)^2}{Det(H)} < \dfrac{(r\_t+1)^2}{r\_t}$$
则认为该关键点不是边缘，否则予以剔除。

### 关键点方向分配
&emsp;&emsp;为了使特征描述子具有旋转不变性，需要利用关键点邻域像素的梯度方向分布特性为每个关键点指定方向参数。对于在DoG金字塔中检测出的关键点，在其邻近高斯金字塔图像的$3\sigma$邻域窗口内计算其梯度幅值和方向，公式如下：
$$m(x,y)=\sqrt{[L(x+1,y)-L(x-1,y)]^2 + [L(x,y+1)-L(x,y-1)]^2}$$
$$\theta(x,y)=tan^{-1}{[L(x,y+1)-L(x,y-1)] / [L(x+1,y)-L(x-1,y)]}$$
式中，$L$为关键点所在尺度空间的灰度值，$m(x,y)$为梯度幅值，$\theta(x,y)$为梯度方向。对模值$m(x,y)$按$\sigma=1.5\sigma\_{oct}$、邻域窗口为$3\sigma=3 \times 1.5\sigma\_{oct}$的高斯分布加权。在完成关键点的梯度计算后，使用直方图统计邻域内像素的梯度和方向，梯度直方图将梯度方向$(0, 360^{\circ })$分为36柱(bins)，如下图所示，直方图的峰值所在的方向代表了该关键点的主方向。

<img src="https://ooo.0o0.ooo/2017/06/30/59564f338a59c.png" alt="main direction.png" title="梯度方向直方图" />

梯度方向直方图的峰值代表了该特征点处邻域梯度的主方向，为了增强鲁棒性，保留峰值大于主方向峰值80%的方向作为该关键点的辅方向，因此，在相同位置和尺度，将会有多个关键点被创建但方向不同，可以提高特征点匹配的稳定性。

### 关键点特征描述
&emsp;&emsp;在经过上述流程后，检测到的每个关键点有三个信息：位置、尺度以及方向，接下来要做的就是抽象出一组特征向量描述每个关键点。这个特征描述子不但包括关键点，还包括其邻域像素的贡献，而且需具备较高的独特性和稳定性，以提高特征点匹配的准确率。SIFT特征描述子是关键点邻域梯度经过高斯加权后统计结果的一种表示。通过对关键点周围图像区域分块，计算块内的梯度直方图，得到表示局部特征点信息的特征向量。例如在尺度空间$4 \times 4$的窗口内统计8个方向的梯度直方图，生成一个$4 \times 4 \times 8 = 128$维的表示向量。

#### 确定特征描述子采样区域
&emsp;&emsp;特征描述子与特征点所在的尺度空间有关，因此对梯度的求取应该在特征点对应的高斯图像上进行。将关键点附近的邻域划分为$d \times d(d=4)$个子区域，每个子区域的大小与关键点方向分配时相同，即边长为$3\sigma\_{oct}$，考虑到实际计算时需要进行三线性插值，邻域窗口边长设为$3\sigma\_{oct}(d+1)$，又考虑到旋转因素(坐标轴旋转至关键点主方向)，最后所需的图像区域半径为
$$radius=\dfrac{3\sigma\_{oct} \times \sqrt{2} \times (d+1)}{2}$$
<img src="https://ooo.0o0.ooo/2017/06/30/595656c99be22.jpg" alt="radius.jpg" />

#### 旋转坐标轴至关键点主方向
&emsp;&emsp;将坐标轴旋转至关键点主方向，以确保旋转不变性。旋转后采样点的新坐标为
$$\left[ \begin{matrix} x'\\\ y'\end{matrix} \right] = \left( \begin{matrix} cos\theta & -sin\theta \\\ sin\theta & cos\theta \end{matrix} \right) \left[ \begin{matrix} x\\\ y \end{matrix} \right], &emsp;x,y \in [-radius, radius]$$

<img src="https://ooo.0o0.ooo/2017/06/30/595656c9f07bc.png" alt="rotation.png" />

#### 三线性插值计算权值
&emsp;&emsp;在图像半径区域内对每个像素点求其梯度幅值和方向，并对每个梯度幅值乘以高斯权重参数
$$w=m(a+x,b+y) \times e^{-\dfrac{(x')^2+(y')^2}{2\sigma\_w^2}}$$
式中，$x'、y'$分别表示像素点与关键点的行、列距离，$m(x,y)$表示像素梯度幅值，高斯尺度因子为$\sigma\_w=3\sigma \times 0.5d$。
将旋转后的采样点坐标分配到对应的子区域，计算影响子区域的采样点的梯度和方向，分配到8个方向上。旋转后的采样点$(x',y')$落在子区域的下标为
$$\left[ \begin{matrix} u\\\ v\end{matrix} \right] = \dfrac{1}{3\sigma\_{oct}} \left[ \begin{matrix} x'\\\ y'\end{matrix} \right] + \dfrac{d}{2}, &emsp;u,v \in [0,d]$$
将采样点在子区域的下标进行三线性插值，根据三维坐标计算与周围子区域的距离，按距离远近计算权重，最终累加在相应子区域的相关方向上的权值为
$$weight = w \cdot dr^i \cdot (1-dr)^{1-i} \cdot dc^j \cdot (1-dc)^{1-j} \cdot do^k \cdot (1-do)^{1-k}$$
式中$i、j、k$取0或者1.

<img src="https://ooo.0o0.ooo/2017/06/30/59565c422740a.jpg" alt="grad hist.jpg" title="梯度方向统计直方图" />

#### 向量归一化生成描述子
&emsp;&emsp;得到128维特征向量后，为了去除光照变化的影响，需要对向量进行归一化处理。非线性光照变化仍可能导致梯度幅值的较大变化，但对梯度方向影响较小。因此对于超过阈值0.2的梯度幅值设为0.2，然后再进行一次归一化。最后将特征向量按照对应高斯金字塔的尺度大小排序。至此，SIFT特征描述子形成。

### SIFT特征匹配
&emsp;&emsp;对两幅图像中检测到的特征点，可采用特征向量的欧式距离作为特征点相似性的度量，取图像1中某个关键点，并在图像2中找到与其距离最近的两个关键点，若最近距离与次近距离的比值小于某个阈值，则认为距离最近的这一对关键点为匹配点。降低比例阈值，SIFT匹配点数量会减少，但相对而言会更加稳定。阈值ratio的取值范围一般为0.4~0.6。

### SIFT特征的特点
&emsp;&emsp;SIFT是一种检测、描述、匹配图像局部特征点的算法，通过在尺度空间中检测极值点，提取位置、尺度、旋转不变量，并抽象成特征向量加以描述，最后用于图像特征点的匹配。SIFT特征对灰度、对比度变换、旋转、尺度缩放等保持不变性，对视角变化、仿射变化、噪声也具有一定的鲁棒性。但其实时性不高，对边缘光滑的目标无法准确提取特征点。

### reference 
- [Paper: Distinctive Image Features from Scale-Invariant Keypoints](http://www.cs.ubc.ca/~lowe/papers/ijcv04.pdf)
- [Paper: Object Recognition from Local Scale-Invariant Features](http://cgit.nutn.edu.tw:8080/cgit/PaperDL/iccv99.pdf)
- [PPT: Object Recognition from Local Scale-Invariant Features (SIFT)](https://people.cs.umass.edu/~elm/Teaching/ppt/SIFT.pdf)
- [Code: RobHess OpenSIFT](https://github.com/robwhess/opensift)
- http://blog.csdn.net/abcjennifer/article/details/7639681
- http://blog.csdn.net/zddblog/article/details/7521424
- http://www.sun11.me/blog/2016/sift-implementation-in-matlab/
- http://masikkk.com/article/RobHess-SIFT-Source-Code-Analysis-Overview/
- http://masikkk.com/article/RANSAC-SIFT-Image-Match/
- http://www.cnblogs.com/letben/p/5510976.html
- http://blog.csdn.net/fzthao/article/details/62424271