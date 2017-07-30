---
title: 图像特征描述子之BRIEF
date: 2017-07-05
categories: Algorithm
tags: Image
---
&emsp;&emsp;BRIEF(Binary Robust Independent Elementary Features)是一种对已检测到的特征点进行表示和描述的特征描述方法，和传统的利用图像局部邻域的灰度直方图或梯度直方图提取特征的方式不同，BRIEF是一种二进制编码的特征描述子，既降低了存储空间的需求，提升了特征描述子生成的速度，也减少了特征匹配时所需的时间。
<!-- more -->

### 原理概述
&emsp;&emsp;经典的图像特征描述子SIFT和SURF采用128维(SIFT)或者64维(SURF)特征向量，每维数据一般占用4个字节(Byte)，一个特征点的特征描述向量需要占用512或者256个字节。如果一幅图像中包含有大量的特征点，那么特征描述子将占用大量的存储，而且生成描述子的过程也会相当耗时。在SIFT特征的实际应用中，可以采用PCA、LDA等特征降维的方法来减少特征描述子的维度，例如PCA-SIFT；此外还可以采用一些局部敏感哈希(Locality-Sensitive Hashing, LSH)的方法将特征描述子编码为二进制串，然后使用汉明距离(Hamming Distance)进行特征点的匹配，汉明距离计算的是两个二进制比特串中同一位置不同值的个数，可通过异或操作快速实现，大大提升了特征匹配的效率。

&emsp;&emsp;BRIEF正是这样一种基于二进制编码生成特征描述子，以及利用汉明距离进行特征匹配的算法。由于BRIEF只是一种特征描述子，因此事先得检测并定位特征点，可采用Harris、FAST或者是SIFT算法检测特征点，在此基础上利用BRIEF算法建立特征描述符，在特征点邻域Patch内随机选取若干点对$(p,q)$，并比较这些点对的灰度值，若$I(p)>I(q)$，则编码为1，否则编码为0。这样便可得到一个特定长度的二进制编码串，即BRIEF特征描述子。

### 算法步骤
- 利用Harris或者FAST等方法检测特征点
- 确定特征点的邻域窗口Patch，并对该邻域内像素点进行$\sigma=2$、窗口尺寸为9的高斯平滑，以滤除噪声(也可直接对整幅图像做高斯平滑)
- 在邻域窗口内随机选取n对(n可取128、256等)像素点，并根据灰度值大小编码成二进制串，生成n位(bit)的特征描述子

### 采样方式
&emsp;&emsp;论文原作者Calonder提供了5种在$S \times S$的邻域Patch内随机选取点对$(X,Y)$的方法，如下图所示，一条线段的两个端点表示一个随机点对$(x\_i,y\_i)$。
- $X、Y$为均匀分布$[-S/2,S/2]$
- $X、Y$均为高斯分布$[0, S^2 / 25]$，采样准则服从各向同性的同一高斯分布
- $X$服从高斯分布$[0, S^2 / 25]$，$Y$服从高斯分布$(x\_i, S^2 / 100)$，即采样分为两步，首先在原点处为$x\_i$进行高斯采样，然后在中心为$x\_i$处为$y\_i$进行高斯采样
- $X、Y$在空间量化极坐标下的离散位置处进行随机采样
- $X$固定为$(0,0)$，$Y$在空间量化极坐标下的离散位置处进行随机从采样。

<img src="https://ooo.0o0.ooo/2017/07/14/596875e06003f.jpg" alt="sample.jpg" title="随机点对选取方式" />

### 算法特点
&emsp;&emsp;BRIEF算法通过检测随机响应，并采用二进制编码方式建立特征描述子，减少了特征的存储空间需求，并提升了特征生成的速度；Hamming距离的度量方式便于进行特征点的快速匹配，而且大量实验数据表明，不匹配特征点的Hamming距离为128左右(特征维数为256)，而匹配点的Hamming距离则远小于128。
&emsp;&emsp;BRIEF算法的缺点是不具备尺度不变性和旋转不变性，在图像的旋转角度超过$30^{\circ}$时，特征点匹配的准确率快速下降。

### Experiment & Result
OpenCV实现BRIEF特征检测与匹配
```
#include <opencv2/core/core.hpp> 
#include <opencv2/highgui/highgui.hpp> 
#include <opencv2/imgproc/imgproc.hpp> 
#include <opencv2/features2d/features2d.hpp>

using namespace cv;

int main(int argc, char** argv) 
{ 
    Mat img_1 = imread("box.png"); 
    Mat img_2 = imread("box_in_scene.png");

    // -- Step 1: Detect the keypoints using STAR Detector 
    std::vector<KeyPoint> keypoints_1,keypoints_2; 
    StarDetector detector; 
    detector.detect(img_1, keypoints_1); 
    detector.detect(img_2, keypoints_2);

    // -- Stpe 2: Calculate descriptors (feature vectors) 
    BriefDescriptorExtractor brief; 
    Mat descriptors_1, descriptors_2; 
    brief.compute(img_1, keypoints_1, descriptors_1); 
    brief.compute(img_2, keypoints_2, descriptors_2);

    //-- Step 3: Matching descriptor vectors with a brute force matcher 
    BFMatcher matcher(NORM_HAMMING); 
    std::vector<DMatch> mathces; 
    matcher.match(descriptors_1, descriptors_2, mathces); 
    // -- dwaw matches 
    Mat img_mathes; 
    drawMatches(img_1, keypoints_1, img_2, keypoints_2, mathces, img_mathes); 
    // -- show 
    imshow("Mathces", img_mathes);

    waitKey(0); 
    return 0; 
}
```

<img src="https://ooo.0o0.ooo/2017/07/14/596879b800c06.jpg" alt="match result.jpg" title="BRIEF特征描述与匹配" />

### reference
- [Paper: BRIEF: Binary Robust Independent Elementary Features](http://cvlabwww.epfl.ch/~lepetit/papers/calonder_eccv10.pdf)
- http://www.cnblogs.com/ronny/p/4081362.html
- http://blog.csdn.net/songzitea/article/details/18272559
- http://blog.csdn.net/luoshixian099/article/details/48338273
- http://blog.csdn.net/hujingshuang/article/details/46910259
- http://blog.csdn.net/icvpr/article/details/12342159