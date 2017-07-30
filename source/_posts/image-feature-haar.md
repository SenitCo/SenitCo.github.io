---
title: 图像特征提取之Haar特征
date: 2017-06-15
categories: Algorithm
tags: Image
---
&emsp;&emsp;Haar特征是一种用于目标检测或识别的图像特征描述子，Haar特征通常和AdaBoost分类器组合使用，而且由于Haar特征提取的实时性以及AdaBoost分类的准确率，使其成为人脸检测以及识别领域较为经典的算法。
<!-- more -->
### 多种Haar-like特征
&emsp;&emsp;在Haar-like特征提出之前，传统的人脸检测算法一般是基于图像像素值进行的，计算量较大且实时性较差。Papageorgiou等人最早将Harr小波用于人脸特征表示，Viola和Jones则在此基础上，提出了多种形式的Haar特征。Lienhart等人对Haar矩形特征做了进一步的扩展，加入了旋转$45^{\circ}$的矩形特征，因此现有的Haar特征模板主要如下图所示：

<img src="https://ooo.0o0.ooo/2017/06/25/594f76b0d6c65.jpg" alt="haar.jpg" title="Haar-like矩形特征" />

&emsp;&emsp;在计算Haar特征值时，用白色区域像素值的和减去黑色区域像素值的和，也就是说白色区域的权值为正值，黑色区域的权值为负值，而且权值与矩形区域的面积成反比，抵消两种矩形区域面积不等造成的影响，保证Haar特征值在灰度分布均匀的区域特征值趋近于0。Haar特征在一定程度上反应了图像灰度的局部变化，在人脸检测中，脸部的一些特征可由矩形特征简单刻画，例如，眼睛比周围区域的颜色要深，鼻梁比两侧颜色要浅等。
&emsp;&emsp;Haar-like矩形特征分为多类，特征模板可用于图像中的任一位置，而且大小也可任意变化，因此Haar特征的取值受到特征模板的类别、位置以及大小这三种因素的影响，使得在一固定大小的图像窗口内，可以提取出大量的Haar特征。例如，在一个$24\times 24$的检测窗口内，矩形特征的数量可以达到16万个。这样就需要解决两个重要问题，快速计算Haar矩形特征值——积分图；筛选有效的矩形特征用于分类识别——AdaBoost分类器。

### 快速计算——积分图
#### 积分图构建
&emsp;&emsp;在一个图像窗口中，可以提取出大量的Haar矩形特征区域，如果在计算Haar特征值时，每次都遍历矩形特征区域，将会造成大量重复计算，严重浪费时间。而积分图正是一种快速计算矩形特征的方法，其主要思想是将图像起始像素点到每一个像素点之间所形成的矩形区域的像素值的和，作为一个元素保存下来，也就是将原始图像转换为积分图(或者求和图)，这样在求某一矩形区域的像素和时，只需索引矩形区域4个角点在积分图中的取值，进行普通的加减运算，即可求得Haar特征值，整个过程只需遍历一次图像，计算特征的时间复杂度为常数(O(1))。因此可以大大提升计算效率。
积分图中元素的公式定义如下：
$$ii(x,y) = \Sigma\_{x'\leq x,y'\leq y} i(x',y')$$
上式含义是在$(x,y)$(第$x$行第$y$列)位置处，积分图中元素为原图像中对应像素左上角所有像素值之和。在具体实现时，可用下式进行迭代运算。
$$s(x,y)=s(x,y-1)+i(x,y)$$
$$ii(x,y)=ii(x-1,y)+s(x,y)$$
$s(x,y)$为行元素累加值，初始值$s(x,-1)=0,ii(-1,y)=0$

#### 矩形特征计算
&emsp;&emsp;构建好积分图后，图像中任何矩形区域的像素值累加和都可以通过简单的加减运算快速得到，如下图所示，矩形区域D的像素和值计算公式如下：
$$Sum(D)=ii(x\_4, y\_4)-ii(x\_2,y\_2)-ii(x\_3,y\_3)+ii(x\_1,y\_1)$$

<img src="https://ooo.0o0.ooo/2017/06/25/594fb4dceae3c.jpg" alt="rectangle.jpg" title="矩形区域求和示意图" />
在下图中，以水平向右为x轴正方向，垂直向下为y轴正方向，可定义积分图公式Summed Area Table($SAT(x,y)$)
$$SAT(x,y)=\Sigma\_{x'\leq x,y'\leq y} i(x',y')$$
以及迭代求解式
$$SAT(x,y)=SAT(x,y-1)+SAT(x-1,y)-SAT(x-1,y-1)+I(x,y)$$
$$SAT(-1,y)=0, SAT(x,-1)=0$$
对于左上角坐标为$(x,y)$，宽高为$(w,h)$的矩形区域$r(x,y,w,h,0)$，可利用积分图$SAT(x,y)$求取像素和值
$$RecSum(r)=SAT(x+w-1, y+h-1)+SAT(x-1, y-1)-SAT(x+w-1, y-1)-SAT(x-1,y+h-1)$$
<img src="https://ooo.0o0.ooo/2017/06/25/594fb9abd4e14.jpg" alt="integer.jpg" title="积分图求矩形区域和值" />

#### 旋转矩形特征的计算
&emsp;&emsp;对于旋转矩形特征，相应的有$45^{\circ}$倾斜积分图用于快速计算Haar特征值，如下图所示，倾斜积分图的定义为像素点左上角$45^{\circ}$区域和左下角$45^{\circ}$区域的像素和，公式表示如下：
$$RSAT(x,y)=\Sigma\_{x'\leq x,x'\leq x-\left|y-y'\right|} i(x',y')$$
其递推公式计算如下：
$$RSAT(x,y)=RSAT(x-1,y-1)+RSAT(x-1,y)-RSAT(x-2,y-1)+I(x,y)$$
$$RSAT(x,y)=RSAT(x,y)+RSAT(x-1,y+1)-RSAT(x-2,y)$$
其中$RSAT(-1,y)=RSAT(-2,y)=RSAT(x,-1)=0$
也可直接通过下式递归计算：
$$RSAT(x,y)=RSAT(x-1,y-1)+RSAT(x+1,y-1)-RSAT(x,y-2)+I(x-1,y)+I(x,y)$$
以上3个积分图计算公式是等价的。
<img src="https://ooo.0o0.ooo/2017/06/25/594fb9abd4190.jpg" alt="titled interger.jpg" title="倾斜积分图求倾斜矩形区域和值" />
如下图所示，构建好倾斜积分图后，可快速计算倾斜矩形区域$r=(x,y,w,h,45^{\circ})$的像素和值
$$RecSum(r)=RSAT(x+w,y+w)+RSAT(x-h,y+h)-RSAT(x,y)-RSAT(x+w-h,y+w+h)$$
<img src="https://ooo.0o0.ooo/2017/06/25/594fb9abe7aa1.jpg" alt="rotated rectangle.jpg" title="倾斜矩形区域求和示意图" />

### AdaBoost分类器
&emsp;&emsp;由输入图像得到积分图，通过取不同种类、大小的Haar特征模板，并在不同位置处，利用积分图提取Haar矩形特征，可快速得到大量Haar特征值，AdaBoost分类器可用于对提取的Haar特征(通常需要做归一化处理)进行训练分类，并应用于人脸检测中。AdaBoost是一种集成分类器，由若干个强分类级联而成，而每个强分类器又由若干个弱分类器(例如决策树)组合训练得到。
弱分类器的定义如下：
$$h\_j(x)=\begin{cases} 1,&emsp;p\_j f\_j(x) < p\_j \theta\_j\\\ 0,&emsp;otherwise\end{cases}$$
上式中$p\_j$是为了控制不等式的方向而设置的参数。$x$表示一个图像窗口，$f\_j(x)$表示提取的Haar特征，阈值$\theta$用于判断该窗口是否为目标区域(人脸)。
算法流程：
- 假设训练样本为$(x\_i,y\_i),i=0,1,...,n$，$y\_i$取值为0(负样本)、1(正样本)。
- 初始化权重$w\_1,i=\dfrac{1}{2m},y\_i=\dfrac{1}{2l}$，其中$m$表示负样本的个数，$l$表示正样本的个数。
- For $t =1,2,...,T$
1.归一化权值：$w\_{t,i} = \dfrac{w\_{t,i}}{\Sigma\_{j=1}^n w\_{t,j}} $
2.对于每个(种)特征，训练一个分类器($h\_j$)，每个分类器只使用一种Haar特征进行训练。分类误差为$\varepsilon\_j=\Sigma\_i w\_i \left|h\_j(x\_i)-y\_i\right|$，$h\_j$为特征分类器，$x\_i$为训练图像样本。
3.选择最低误差的分类器$h\_t$
4.更新训练样本的权值$w\_{t+1,i}=w\_{t,i}\beta\_t^{1-e\_i}$，分类正确$e\_i=0$，分类错误$e\_i=1$，$\beta\_t=\dfrac{\varepsilon\_t}{1-\varepsilon\_t}$
- 最后的强分类器为
$$h(x)=\begin{cases} 1,&emsp;\Sigma\_{t=1}^T \alpha\_t h\_t \geq \dfrac{1}{2}\Sigma\_{t=1}^T \alpha\_t \\\ 0,&emsp;otherwise\end{cases}$$
其中$\alpha\_t=log(\dfrac{1}{\beta\_t})$。

&emsp;&emsp;在训练多个弱分类器得到强分类器的过程中，采用了两次加权的处理方法，一是对样本进行加权，在迭代过程中，提高错分样本的权重；二是对筛选出的弱分类器$h\_t$进行加权，弱分类器准确率越高，权重越大。此外，还需进一步对强分类器进行级联，以提高检测正确率并降低误识率。级联分类器如下所示：

<img src="https://ooo.0o0.ooo/2017/06/26/59506e87bf0f5.jpg" alt="cascade.jpg" title="级联分类器" />

&emsp;&emsp;首先将所有待检测的子窗口输入到第一个分类器中，如果某个子窗口判决通过，则进入下一个分类器继续检测识别，否则该子窗口直接退出检测流程，也就是说后续分类器不需要处理该子窗口。通过这样一种级联的方式可以去除一些误识为目标的子窗口，降低误识率。例如，单个强分类器，99%的目标窗口可以通过，同时50%的非目标窗口也能通过，假设有20个强分类器级联，那么最终的正确检测率为$0.99^{20}=98\%$，而错误识别率为$0.50^{20} \approx 0.0001\%$，在不影响检测准确率的同时，大大降低了误识率。当然前提是单个强分类器的准确率非常高，这样级联多个分类器才能不影响最终的准确率或者影响很小。

&emsp;&emsp;在一幅图像中，为了能够检测到不同位置的目标区域，需要以一定步长遍历整幅图像；而对于不同大小的目标，则需要改变检测窗口的尺寸，或者固定窗口而缩放图像。这样，最后检测到的子窗口必然存在相互重叠的情况，因此需要进一步对这些重叠的子窗口进行合并，也就是非极大值抑制(NMS,non-maximum suppression)，同时剔除零散分布的错误检测窗口。

### reference
- [Paper: Rapid Object Detection using a Boosted Cascade of Simple Features](http://wearables.cc.gatech.edu/paper_of_week/viola01rapid.pdf)
- [Paper: Empirical Analysis of Detection Cascades of Boosted Classifiers for Rapid Object Detection](https://link.springer.com/content/pdf/10.1007%2F978-3-540-45243-0_39.pdf)
- [Paper: An Extended Set of Haar-like Features for Rapid Object Detection](https://pdfs.semanticscholar.org/72e0/8cf12730135c5ccd7234036e04536218b6c1.pdf)
- http://blog.csdn.net/xizero00/article/details/46929261
- http://www.cnblogs.com/bhlsheji/p/4726376.html
- http://blog.csdn.net/zy1034092330/article/details/50383097
- http://blog.csdn.net/zouxy09/article/details/7929570