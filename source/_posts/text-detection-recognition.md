---
title: Chinese Text Detection and Recognition
date: 2017-03-03
categories: Projects
tags: Image
---

&emsp;&emsp;The task of Chinese text detection is to localize the regions in a 2D image which contain Chinese characters. The task of Chinese text recognition is, given the localized regions including text, to convert each region into machine-encoded text. It is an important technique for understanding text information in 2D images, and many other applications such as text-to-speech, machine translation, text mining, etc.  
<!--more-->

### Course
&emsp;&emsp;Image Analysis and Understanding

### Project
&emsp;&emsp;Chinese Text Detection and Recognition

### Description
&emsp;&emsp;The task of Chinese text detection is to localize the regions in a 2D image which contain Chinese characters. The task of Chinese text recognition is, given the localized regions including text, to convert each region into machine-encoded text. It is an important technique for understanding text information in 2D images, and many other applications such as text-to-speech, machine translation, text mining, etc.

### Deadline
&emsp;&emsp;April 11, Tuesday, 2017

### Overview
&emsp;&emsp;图像文本一般分为两大类，图形文本(Graphic text)和场景文本(Scene text)，图形文本通常是指覆盖在图像上的机器打印的文本，如图片水印、视频字幕等；而场景文本则是自然场景中物体上的文本，如墙壁上的标语、衣服上的文字等。最近大部分的研究都集中在自然场景中文本检测与识别上，由于受到场景复杂、光照变化、成像随意等因素的影响，这一研究任务具有较大的挑战性，但其应用领域十分广泛，包括文字转语音、多语言翻译、自动化生产等。
&emsp;&emsp;对于文本检测，需要在一幅2D图像上准确定位到文本区域，而识别则是在此基础上将其识别转换为机器编码的文字。此次项目设计，主要针对自然场景中文本图像，采用多种方法进行检测并作对比，在定位到文本区域的基础上，采用Tesseract的开源库做进一步的识别。

### Method & Model
&emsp;&emsp;文本检测方法来源于ECCV 2016上的一篇文章——[Detecting Text in Natural Image with Connectionist Text Proposal Network](https://arxiv.org/pdf/1609.03605.pdf)，是一种基于Faster R-CNN的端到端的检测方法。R-CNN是目标检测领域中十分经典的方法，相比于传统的手工特征，R-CNN将卷积神经网络引入，用于提取深度特征，后接一个分类器预测搜索区域是否包含目标及其置信度，并取得了相对而言较为准确的检测结果，而Fast R-CNN和Faster R-CNN则是R-CNN的升级版本，在准确率和实时性方面都得到了较大的提升。在Fast R-CNN中，首先需要使用Selective Search的方法提取图像的proposals，也就是候选目标区域。而新提出的Faster R-CNN模型则引入了RPN网络(Region Proposal Network)提取proposals，RPN是一个全卷积网络，通过共享卷积层特征大大缩短了proposals提取的时间，Fast R-CNN基于RPN提取的proposal作进一步的检测和识别。因此Faster R-CNN模型主要由两大模块组成：RPN候选框提取模块和Fast R-CNN检测模块，如下图所示。

<img src="https://ooo.0o0.ooo/2017/05/14/5917b8d52ee44.jpg" alt="Faster R-CNN.jpg" title="Faster R-CNN模型图" width=50% height=50% align="center" />

&emsp;&emsp;在Faster R-CNN中，利用VGG-16进行特征提取，RPN网络结构如图2所示，在conv5卷积层用一个n x n的滑动窗口生成一个512维的全连接层，该层后接两个子连接层：分类层(cls-layer)和回归层(reg-layer)，cls-layer用于判断检测的proposal是目标还是背景，reg-layer则用于预测proposal的宽高和中心锚点的坐标，滑动窗口的处理方式保证了两个子连接层关联了conv5的全部特征空间。在RPN中有一个较为重要的概念——Anchor，对于一个滑动窗口，可以同时预测多个proposal，假设proposal最多为k个，k个proposal对应k个参考候选框(reference box)，这k个参考候选框即为Anchor，通过取不同的scale和aspect ratio，对应不同的Anchor，Faster R-CNN模型k值为9，即3种不同的scale和aspect ratio确定当前滑动窗口位置处对应的9个Anchor，最后cls-layer和reg-layer输出参数的数量分别为2k和4k。对于一幅W x H的卷积特征层，对应 W x H x k个Anchor，所有的Anchor都具有尺度不变性。

<img src="https://ooo.0o0.ooo/2017/05/14/5917b9c3ab2a9.jpg" alt="RPN.jpg" title="RPN网络结构" width=50% height=50% align="center"/>

&emsp;&emsp;训练RPN时，一个Mini-batch是由一幅图像中任意选取的256个proposal组成的，其中正负样本的比例为1 : 1。如果正样本数不足，则多补充一些负样本以满足有256个proposal可以用于训练。在RPN训练开始时，共享的VGG卷积层参数可以直接拷贝ImageNet中训练好的模型参数，剩余层参数用标准差为0.01的高斯分布初始化。RPN在提取到proposal后，通过训练Fast R-CNN进行检测和识别，而RPN和Fast R-CNN共用了VGG的卷积层，因此采用交替训练的方式实现卷积层特征共享。Faster R-CNN一个较为突出的贡献就是引入RPN，将proposal部分嵌入到了内部网络，因此整个网络模型即可完成端到端的检测任务，而不需要先执行proposal的搜索定位算法。  
&emsp;&emsp;Faster R-CNN在目标检测方面能取得良好的效果，但文本检测和一般的目标检测不同，文本是一系列字符、笔画或单词的序列集合，而不是一般目标检测中独立的目标，同一文本序列上的不同字符可能差异较大、距离长短不一，检测出一个完整的文本行可能比检测单个目标的难度要大。因此针对文本检测这种特殊的情况，论文中提出了一种Recurrent Connectionist Text Proposal Network (CTPN)的检测方法，算法思想和Faster R-CNN类似，通过搜索定位多个候选文本框(Text Proposal)，对每个proposal做分类判决，包括proposal为文本、非文本的概率以及proposal的坐标位置。不同的是，CTPN中定义的Anchor是fine-scale的，在水平方向固定宽度为16个像素，而在垂直方向考虑k(=10)种尺寸，这和RPN中同一个滑动窗口考虑9个Anchor，分别为3种不同的scale和aspect ratio有所不同，正是考虑了文本和一般目标检测的差异，作者认为预测文本的垂直位置比水平位置要更容易，在proposal生成网络中，最后的输出分别为proposal的分类概率以及proposal的高度、中心锚点的y坐标以及在水平方向的偏移量，因为宽度是固定的，不予考虑。CTPN与Faster R-CNN还有一个较大的不同之处就是将RPN换成了双向的长短时记忆网络(BLSTM)，VGG-16的conv5卷积层(W x H x C)，不是直接连接到全连接层，而是将每一行的所有滑动窗口对应的3 x 3 x C特征输入到BLSTM中，用于编码Text Proposal的上下文信息，更加准确地检测文本区域。单向LSTM的维数为128，因此得到W x 256的输出，然后将BLSTM连接到512维的全连接层，全连接层后接3个子连接层，分别用于预测proposal的类别信息、在垂直方向的高度和y坐标以及在水平方向的偏移量。CTPN的整体网络结构和提取的Text Proposal示意图如图3所示，在得到多个细长的Text Proposal后，文中利用一种文本行构造算法，将多个Text Proposal合并成一个完整的文本区域，其主要思想是根据一定的约束条件，将相邻的Text Proposal两两合并，直到没有公共元素为止。

<img src="https://ooo.0o0.ooo/2017/05/14/5917ba8f3a3ed.jpg" alt="CTPN.jpg" title="CTPN网络结构和Text Proposal" align="center"/>

### Detection Results

<!-- <figure class="third">
    <img src="https://ooo.0o0.ooo/2017/05/14/5917bc63897c8.jpg" alt="1.jpg" width=30% height=30%>
    <img src="https://ooo.0o0.ooo/2017/05/14/5917bc6386978.jpg" alt="2.jpg" width=30% height=30%>
    <img src="https://ooo.0o0.ooo/2017/05/14/5917bc638a778.jpg" alt="3.jpg" width=30% height=30%>
</figure>
<figure class="third">
    <img src="https://ooo.0o0.ooo/2017/05/14/5917bc63857a0.jpg" alt="4.jpg" width=30% height=30%>
    <img src="https://ooo.0o0.ooo/2017/05/14/5917bc638e4f5.jpg" alt="5.jpg" width=30% height=30%>
    <img src="https://ooo.0o0.ooo/2017/05/14/5917bc6387f6b.jpg" alt="6.jpg" width=30% height=30%>
</figure> -->

<img src="https://ooo.0o0.ooo/2017/05/14/5917cbb71e93b.png" alt="7.png" title="Detection Results" align="center" />

### Conclusion
&emsp;&emsp;在文本检测中用到的这种基于Faster R-CNN的算法模型，针对自然场景的多语言文本能取得良好的检测结果，比较适用于水平方向的文本检测，对于非水平方向的文本，该模型虽然也能有效检测，但不能获取文本的整体偏转信息，不利于后续的文本识别，因此在检测到文本区域后，对文本的整体偏转角度做了进一步的检测和校正。在识别任务中，主要用到了Tesseract的开源库，该OCR引擎对于中文的识别效果不是太理想。

### Reference
[Text Detection and Recognition in Imagery: A Survey](http://ieeexplore.ieee.org/stamp/stamp.jsp?arnumber=6945320)
[Course Website: Image Analysis and Understanding](http://lbmedia.ece.ucsb.edu/member/uweb/Teaching/website/index.htm)
[R-CNN,SPP-NET, Fast-R-CNN,Faster-R-CNN系列检测方法解读](http://www.cnblogs.com/venus024/p/5717766.html)
[Detecting Text in Natural Image with Connectionist Text Proposal Network](https://arxiv.org/pdf/1609.03605.pdf)
[Paper Reading: CTPN](http://www.cnblogs.com/lillylin/p/6277061.html)
[Source Code: CTPN](https://github.com/tianzhi0549/CTPN)
[Text Detection and Recognition Resource --1](https://github.com/chongyangtao/Awesome-Scene-Text-Recognition)
[Text Detection and Recognition Resource --2](https://handong1587.github.io/deep_learning/2015/10/09/ocr.html)
[Text Detection and Recognition Resource --2](http://blog.csdn.net/peaceinmind/article/details/51387367)  

关于Tesseract OCR引擎的安装和配置：  
[依赖库Leptonica的编译](http://www.leptonica.org/source/README.html)
[Tesseract-OCR的编译和配置](https://github.com/tesseract-ocr/tesseract/wiki/Compiling)
[OCR引擎的Python接口](https://github.com/openpaperwork/pyocr)



