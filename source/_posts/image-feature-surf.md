---
title: 图像特征之SURF特征匹配
date: 2017-06-27
categories: Algorithm
tags: Image
---
&emsp;&emsp;加速鲁棒特征(Speed Up Robust Feature, SURF)和SIFT特征类似，同样是一个用于检测、描述、匹配图像局部特征点的特征描述子。SIFT是被广泛应用的特征点提取算法，但其实时性较差，如果不借助于硬件的加速和专用图形处理器(GPUs)的配合，很难达到实时的要求。对于一些实时应用场景，如基于特征点匹配的实时目标跟踪系统，每秒要处理数十帧的图像，需要在毫秒级完成特征点的搜索定位、特征向量的生成、特征向量的匹配以及目标锁定等工作，SIFT特征很难满足这种需求。SURF借鉴了SIFT中近似简化(DoG近似替代LoG)的思想，将Hessian矩阵的高斯二阶微分模板进行了简化，借助于积分图，使得模板对图像的滤波只需要进行几次简单的加减法运算，并且这种运算与滤波模板的尺寸无关。SURF相当于SIFT的加速改进版本，在特征点检测取得相似性能的条件下，提高了运算速度。整体来说，SUFR比SIFT在运算速度上要快数倍，综合性能更优。
<!-- more -->

### 积分图像
&emsp;&emsp;SURF算法中用到了积分图的概念，积分图(Integral Image)由Viola和Jones提出，在前面的博文[Haar特征提取](https://senitco.github.io/2017/06/25/image-feature-haar/)中做了详细的介绍。借助积分图，图像与高斯二阶微分模板的滤波转化为对积分图像的加减运算，从而在特征点的检测时大大缩短了搜索时间。    
&emsp;&emsp;积分图中任意一点$(i,j)$的值$ii(i,j)$，为原图像左上角到任意点$(i,j)$相应对角线区域灰度值的总和，即
$$ii(x,y) = \Sigma\_{x'\leq x,y'\leq y} i(x',y')$$
式中，$i(x',y')$表示原图像中的灰度值，具体实现时$ii(x,y)$可由下式迭代计算得到
$$s(x,y)=s(x,y-1)+i(x,y)$$
$$ii(x,y)=ii(x-1,y)+s(x,y)$$
求取积分图时，对图像所有像素遍历一遍，得到积分图后，计算任何矩形区域内的像素灰度和只需进行三次加减运算，如下图所示。

<img src="https://ooo.0o0.ooo/2017/07/03/5959f26a69af1.png" alt="integral image.png" />

### Hessian矩阵近似
&emsp;&emsp;图像点的二阶微分Hessian矩阵的行列式(Determinant of Hessian, DoH)极大值，可用于图像的斑点检测(Blob Detection)。Hessian矩阵定义如下：
$$H(x,y,\sigma)=\left( \begin{matrix} L\_{xx}& L\_{xy} \\\ L\_{xy} & L\_{yy}\end{matrix} \right)$$
式中，$L\_{xx}、L\_{yy}、L\_{xy}$分别是高斯二阶微分算子$\dfrac{\partial ^2 g}{\partial x^2}、\dfrac{\partial ^2 g}{\partial y^2}、\dfrac{\partial ^2 g}{\partial x \partial y}$与原图像的卷积，Hessian矩阵的行列式值DoH为
$$detH = L\_{xx} L\_{yy} - L\_{xy}^2$$
与LoG算子一样，DoH同样反映了图像局部的纹理或结构信息，与LoG相比，DoH对图像中细长结构的斑点有较好的抑制作用。LoG和DoH在利用二阶微分算子对图像进行斑点检测时，都需要利用高斯滤波平滑图像、抑制噪声，检测过程主要分为以下两步：
- 使用不同的$\sigma$生成$(\dfrac{\partial ^2 g}{\partial x^2} + \dfrac{\partial ^2 g}{\partial y^2})$或$\dfrac{\partial ^2 g}{\partial x^2}、\dfrac{\partial ^2 g}{\partial y^2}、\dfrac{\partial ^2 g}{\partial x \partial y}$高斯卷积模板，并对图像进行卷积运算。
- 在图像的位置空间和尺度空间搜索LoG或DoH的峰值，并进行非极大值抑制，精确定位到图像极值点。

三个高斯微分算子的响应图像如下图所示：

<img src="https://ooo.0o0.ooo/2017/07/03/5959f8e4d9702.png" alt="gaussian.png" />

由于二阶高斯微分模板被离散化和裁剪的原因，导致了图像在旋转奇数倍的$\pi/4$即模板对角线方向时，特征点检测的重复性(Repeatability)降低，即原来是特征点的地方在旋转后可能检测不到了；而旋转$\pi/2$时，特征点检测的重复性最高。不过这一不足并不影响Hessian矩阵检测特征点。

&emsp;&emsp;为了将模板与图像的卷积转化为盒子滤波器(Box Filter)运算，并能够使用积分图，需要对高斯二阶微分模板进行简化，使得简化后的模板只是由几个矩形区域组成，矩形区域内填充同一值，如下图所示，在简化模板中白色区域的值为1，黑色区域的值为-1或-2(由相对面积决定)，灰色区域的值为0。

<img src="https://ooo.0o0.ooo/2017/07/03/5959fd140cfe1.png" alt="simplify mask.png" title="高斯二阶微分模板及相应简化模板"/>

对于$\sigma=1.2$的高斯二阶微分滤波，设定模板的尺寸为$9 \times 9$的大小，并用它作为最小尺度空间值对图像进行滤波和斑点检测。使用$D\_{xx}、D\_{yy}、D\_{xy}$表示简化模板与图像进行卷积的结果，Hessian矩阵的行列式可进一步简化为：
$$Det(H)=L\_{xx}L\_{yy}-L\_{xy}^2=D\_{xx} \dfrac{L\_{xx}}{D\_{xx}} D\_{yy} \dfrac{L\_{yy}}{D\_{yy}} - D\_{xy} \dfrac{L\_{xy}}{D\_{xy}} D\_{xy} \dfrac{L\_{xy}}{D\_{xy}}$$
$$= D\_{xx}D\_{yy}(\dfrac{L\_{xx}}{D\_{xx}} \dfrac{L\_{yy}}{D\_{yy}}) - D\_{xy} D\_{xy} \dfrac{L\_{xy}}{D\_{xy}} \dfrac{L\_{xy}}{D\_{xy}} = (D\_{xx}D\_{yy} - D\_{xy} D\_{xy} Y)C$$
$$Y = (\dfrac{L\_{xy}}{D\_{xy}} \dfrac{L\_{xy}}{D\_{xy}}) (\dfrac{D\_{xx}}{L\_{xx}} \dfrac{D\_{yy}}{L\_{yy}})$$
$$C = \dfrac{L\_{xx}}{D\_{xx}} \dfrac{L\_{yy}}{D\_{yy}}$$
式中，$Y=(||L\_{xy}(1.2)||\_F ||D\_{xx}(9)||\_F)/(||L\_{xx}(1.2)||\_F ||D\_{xy}(9))||\_F)=0.912 \approx 0.9$，$||X||\_F$为Frobenius范数，$1.2$是LoG的尺度$\sigma$，$9$是box filter的尺寸。理论上说，对于不同的$\sigma$值和对应的模板尺寸，$Y$值应该是不同的，但为了简化起见，可将其视为一个常数，同样$C$也为一常数，且不影响极值求取，因此，DoH可近似如下：
$$Det(H\_{approx}) = D\_{xx} D\_{yy} - (0.9D\_{xy})^2$$
在实际计算滤波响应值时，需要使用模板中盒子(矩形)区域的面积进行归一化处理，以保证一个统一的Frobenius范数能适应所有的滤波尺寸。

&emsp;&emsp;使用近似的Hessian矩阵行列式来表示一个图像中某一点处的斑点响应值，遍历图像中的所有像素，便形成了在某一尺度下斑点检测的响应图像。使用不同的模糊尺度和模板尺寸，便形成了多尺度斑点响应的金字塔图像，利用这一金字塔图像，可以进行斑点响应极值点的搜索定位，其过程与SIFT算法类似。

### 尺度空间表示
&emsp;&emsp;要想检测不同尺度的极值点，必须建立图像的尺度空间金字塔。一般的方法是通过采用不同$\sigma$的高斯函数，对图像进行平滑滤波，然后重采样获得更高一层(Octave)的金字塔图像。Lowe在SIFT算法中就是通过相邻两层(Interval)高斯金字塔图像相减得到DoG图像，然后在DoG金字塔图像上进行特征点检测。与SIFT特征不同的是，SURF算法不需要通过降采样的方式得到不同尺寸大小的图像建立金字塔，而是借助于盒子滤波和积分图像，不断增大盒子滤波模板，通过积分图快速计算盒子滤波的响应图像。然后在响应图像上采用非极大值抑制，检测不同尺度的特征点。SIFT算法的LoG金字塔和SURF算法的近似DoH金字塔如下图所示：

<img src="https://ooo.0o0.ooo/2017/07/03/595a2141bea36.jpg" alt="pyramid.jpg" title="LoG金字塔与盒状滤波金字塔" />

&emsp;&emsp;如前所述，使用$9 \times 9$的模板对图像滤波，其结果作为最初始的尺度空间层，后续层将通过逐步增大滤波模板尺寸，以及放大后的模板与图像卷积得到。由于采用了box filter和积分图，滤波过程并不随着滤波模板尺寸的增大而增加运算量。
&emsp;&emsp;在建立盒状滤波金字塔时，与SIFT算法类似，需要将尺度空间划分为若干组(Octaves)。每组又由若干固定层组成，包括不同尺寸的滤波模板对同一输入图像进行滤波得到的一系列响应图。由于积分图像的离散特性，两个相邻层之间的最小尺度变化量，是由高斯二阶微分滤波模板在微分方向上对正负斑点响应长度(波瓣长度)$l\_0$决定的，它是盒子滤波模板尺寸的1/3。对于$9 \times 9$的滤波模板，$l\_0$为3。下一层的响应长度至少应该在$l\_0$的基础上增加2个像素，以保证一边一个像素，即$l\_0=5$，这样模板的尺寸为$15 \times 15$，如下图所示。依次类推，可以得到一个尺寸逐渐增大的模板序列，尺寸分别为$9 \times 9$、$15 \times 15$、$21 \times 21$、$27 \times 27$。显然，第一个模板和最后一份模板产生的Hessian响应图像只作为比较用，而不会产生最后的响应极值。这样，通过插值计算，可能的最小尺度值为$\sigma=1.2 \times \dfrac{(15+9)/2}{9}=1.6$，对应的模板尺寸为$12 \times 12$；可能的最大尺度值为$\sigma=1.2 \times \dfrac{(27+21)/2}{9}=3.2$，对应的模板尺寸为$24 \times 24$。

<img src="https://ooo.0o0.ooo/2017/07/03/595a295515d31.jpg" alt="filter template.jpg" title="滤波模板尺寸变化" />

&emsp;&emsp;采用类似的方法处理其他组的模板序列，其方法是将滤波器尺寸增加量按Octave的组数$m$翻倍，即$6 \times 2^{m-1} $，序列依次为$(6, 12, 24, 48, ...)$，这样，在盒状滤波金字塔中，每组滤波器的尺寸如下图所示，滤波器的组数可由原始图像的尺寸决定。对数水平轴代表尺度，组之间有相互重叠，其目的是为了覆盖所有可能的尺度。在通常尺度分析情况下，随着尺度的增大，被检测的特征点数迅速衰减。

<img src="https://ooo.0o0.ooo/2017/07/03/595a2b5a57f41.png" alt="scale-octaves.png" title="不同组滤波器尺寸" />
滤波器的尺寸$L$、滤波响应长度$l$、组索引$o$、层索引$s$、尺度$\sigma$之间的相互关系如下：
$$L=3 \times (2^{o+1}(s+1)+1)$$
$$l=\dfrac{L}{3}=2^{o+1}(s+1)+1$$
$$\sigma=1.2 \times \dfrac{L}{9} = 1.2 \times \dfrac{l}{3}$$

### 关键点定位 
&emsp;&emsp;和LoG、DoG类似，建立尺度空间后，需要搜索定位关键点。将经过box filter处理过的响应图像中每个像素点
与其3维邻域中的26个像素点进行比较，若是最极大值点，则认为是该区域的局部特征点。然后，采用3维线性插值法得到亚像素级的特征点，同时去掉一些小于给定阈值的点，使得极值检测出来的特征点更稳健。和DoG不同的是，不需要剔除边缘导致的极值点，因为Hessian矩阵的行列式已经考虑了边缘的问题。

<img src="https://ooo.0o0.ooo/2017/07/03/595a2ee656e20.png" alt="maximum detectoin.png" title="极值检测" />

### 特征点方向分配
&emsp;&emsp;为了保证特征描述子具有旋转不变性，与SIFT一样，需要对每个特征点分配一个主方向。为此，在以特征点为中心，以$6s$(s为特征点的尺度)为半径的区域内，计算图像的Haar小波响应，实际上就是对图像进行梯度运算，只不过需要利用积分图，提高梯度计算效率。求Haar小波响应的图像区域和Haar小波模板如下图所示，用于计算梯度的Haar小波的尺度是$4s$，扫描步长为$s$。

<img src="https://ooo.0o0.ooo/2017/07/03/595a34455c031.jpg" alt="haar response.jpg" title="Haar小波响应计算" />

使用$\sigma=2s$的高斯函数对Haar小波的响应值进行加权。为了求取主方向，设计一个以特征点为中心，张角为$\pi/3$的扇形窗口，如下图所示，以一定旋转角度$\theta$旋转窗口，并对窗口内的Haar小波响应值$dx、dy$进行累加，得到一个矢量$(m\_w, \theta\_w)$
$$m\_w=\Sigma\_w dx + \Sigma\_w dy$$
$$\theta\_w=arctan(\Sigma\_w dy / \Sigma\_w dx) $$
主方向为最大Haar响应累加值所对应的方向，即$\theta=\theta\_w|max{m\_w}$

<img src="https://ooo.0o0.ooo/2017/07/03/595a3745a0dc7.png" alt="main direction.png" title="统计旋转窗口内的Haar小波响应和值" />

仿照SIFT求主方向时策略，当存在大于主峰值80%以上的峰值时，则将对应方向认为是该特征点的辅方向。一个特征点可能会被指定多个方向，可以增强匹配的鲁棒性。

### 特征描述子生成
&emsp;&emsp;生成特征点描述子时，同样需要计算图像的Haar小波响应。与确定主方向不同的是，这里不再使用圆形区域，而是在一个矩形区域计算Haar小波响应。以特征点为中心，沿主方向将$20s \times 20s$的邻域划分为$4 \times 4$个子块，每个子块利用尺寸为$2s$的Haar模板计算响应值，然后对响应值统计$\Sigma dx、\Sigma |dx|、\Sigma dy、\Sigma |dy|$形成特征向量，如下图所示：

<img src="https://ooo.0o0.ooo/2017/07/03/595a3c15098a0.png" alt="feature descriptor.png" title="特征描述子生成" />

将$20s$的窗口划分为$4 \times 4$个子窗口，每个子窗口大小为$5s \times 5s$，使用尺寸为$2s$的Haar小波计算子窗口的响应值；然后，以特征点为中心，用$\sigma=10s/3=3.3s$的高斯核函数对$dx、dy$进行加权计算；最后，分别对每个子块的加权响应值进行统计，得到每个子块的向量：
$$V\_i=[\Sigma dx,\Sigma |dx|,\Sigma dy,\Sigma |dy|]$$
由于共有$4 \times 4$个子块，特征描述子的特征维数为$4 \times 4 \times 4 = 64$。SURF描述子不仅具有尺度和旋转不变性，还具有光照不变性，这由小波响应本身决定，而对比度不变性则是通过将特征向量归一化来实现。下图为3种简单模式图像及其对应的特征描述子，可以看出，引入Haar小波响应绝对值的统计和是必要的，否则只计算$\Sigma dx、\Sigma dy$的话，第一幅图和第二幅图的特征表现形式是一样的，因此，采用4个统计量描述子区域使特征更具有区分度。

<img src="https://ooo.0o0.ooo/2017/07/03/595a3f7e23db4.png" alt="pattern feature.png" title="不同模式图像的描述子" />

&emsp;&emsp;为了充分利用积分图像计算Haar小波的响应值，在具体实现中，并不是直接通过旋转Haar小波模板求其响应值，而是在积分图像上先使用水平和垂直的Haar模板求得响应值$dx、dy$，对$dx、dy$进行高斯加权处理，并根据主方向的角度，对$dx、dy$进行旋转变换，从而得到旋转后的$dx'、dy'$。

&emsp;&emsp;SURF在求取描述子特征向量时，是对一个子块的梯度信息进行求和，而SIFT是依靠单个像素计算梯度的方向。在有噪声的干扰下，SURF描述子具有更好的鲁棒性。一般而言，特征向量的长度越长，所承载的信息量就越大，特征描述子的独特性就越好，但匹配时所付出的时间代价也越大。对于SURF描述子，可以将其扩展到128维。具体方法就是在求Haar小波响应值的统计和时，区分$dx \geq 0$和$dx < 0$的情况，以及$dy \geq 0$和$dy < 0$的情况。为了实现快速匹配，SURF在特征向量中增加了一个新的元素，即特征点的拉普拉斯响应正负号。在特征点检测时，将Hessian矩阵的迹(Trace)的正负号记录下来，作为特征向量中的一个变量。在特征匹配时可以节省运算时间，因为只用具有相同正负号的特征点才可能匹配，对于不同正负号的特征点不再进行相似性计算。
下图是用SURF进行特征点匹配的实验结果

<img src="https://ooo.0o0.ooo/2017/07/04/595b05c548ec2.png" alt="result.png" title="SURF特征点匹配" />

### SURF与SIFT的对比
- 尺度空间：SIFT使用DoG金字塔与图像进行卷积操作，而且对图像有做降采样处理；SURF是用近似DoH金字塔(即不同尺度的box filters)与图像做卷积，借助积分图，实际操作只涉及到数次简单的加减运算，而且不改变图像大小。
- 特征点检测：SIFT是先进行非极大值抑制，去除对比度低的点，再通过Hessian矩阵剔除边缘点。而SURF是计算Hessian矩阵的行列式值(DoH)，再进行非极大值抑制。
- 特征点主方向：SIFT在方形邻域窗口内统计梯度方向直方图，并对梯度幅值加权，取最大峰对应的方向；SURF是在圆形区域内，计算各个扇形范围内$x、y$方向的Haar小波响应值，确定响应累加和值最大的扇形方向。
- 特征描述子：SIFT将关键点附近的邻域划分为$4 \times 4$的区域，统计每个子区域的梯度方向直方图，连接成一个$4 \times 4 \times 8 = 128$维的特征向量；SURF将$20s \times 20s$的邻域划分为$4 \times 4$个子块，计算每个子块的Haar小波响应，并统计4个特征量，得到$4 \times 4 \times 4 = 64$维的特征向量。   

&emsp;&emsp;总体来说，SURF和SIFT算法在特征点的检测取得了相似的性能，SURF借助积分图，将模板卷积操作近似转换为加减运算，在计算速度方面要优于SIFT特征。

### reference
- [Paper: SURF: Speeded Up Robust Features](http://www.vision.ee.ethz.ch/~surf/eccv06.pdf)
- http://blog.csdn.net/songzitea/article/details/16986423
- http://www.cnblogs.com/ronny/p/4045979.html
- http://www.cnblogs.com/ronny/p/4048213.html
- http://www.cnblogs.com/YiXiaoZhou/p/5903690.html
- http://www.cnblogs.com/tornadomeet/archive/2012/08/17/2644903.html