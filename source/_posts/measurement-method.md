---
title: 数据挖掘中的度量方法
date: 2017-05-24
categories: Algorithm
tags: ML
---

&emsp;&emsp;在数据挖掘中，无论是对数据进行分类、聚类还是异常检测、关联性分析，都建立在数据之间相似性或相异性的度量基础上。通常使用距离作为数据之间相似性或相异性的度量方法，常用的度量方法有欧式距离、曼哈顿距离、切比雪夫距离、闵可夫斯基距离、汉明距离、余弦距离、马氏距离、Jaccard系数、相关系数、信息熵。   
<!-- more -->

<!-- mathjax config similar to math.stackexchange -->
<script type="text/x-mathjax-config">
MathJax.Hub.Config({
    jax: ["input/TeX", "output/HTML-CSS"],
    tex2jax: {
        inlineMath: [ ['$', '$'] ],
        displayMath: [ ['$$', '$$']],
        processEscapes: true,
        skipTags: ['script', 'noscript', 'style', 'textarea', 'pre', 'code']
    },
    messageStyle: "none",
    "HTML-CSS": { preferredFont: "TeX", availableFonts: ["STIX","TeX"] }
});
</script>
<script type="text/javascript" src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script> 

### 欧式距离
&emsp;&emsp;$n$维空间中两个样本点$x$和$y$之间的欧几里得距离定义如下：
$$d(x,y)=\sqrt{\Sigma\_{k=1}^n (x\_k-y\_k)^2}$$
标准化欧式距离公式如下：
$$d(x,y)=\sqrt{\Sigma\_{k=1}^n (\dfrac{x\_k-y\_k}{s\_k})^2}$$
式中，$s\_k$为数据每一维的方差，标准化欧式距离考虑了数据各维分量的量纲和分布不一样，相当于对每维数据做了标准化处理。欧式距离适用于度量数据属性无关、值域或分布相同的数据对象。

### 曼哈顿距离
&emsp;&emsp;曼哈顿距离也称为街区距离，计算公式如下：
$$d(x,y)=\Sigma\_{k=1}^n \left|x\_k-y\_k\right|$$

### 切比雪夫距离
$$d(x,y) = \lim\_{n\rightarrow \infty} (\Sigma\_{k=1}^n (\left|x\_k-y\_k\right|)^r)^\dfrac{1}{r} = max\_k (\left|x\_k-y\_k\right|)$$
上面两个公式是等价的。

### 闵可夫斯基距离
$$d(x,y)=(\Sigma\_{k=1}^n (\left|x\_k-y\_k\right|)^r)^\dfrac{1}{r}$$
式中，r是一个可变参数，根据参数r取值的不同，闵可夫斯基距离可以表示一类距离
&emsp;&emsp;r = 1时，为曼哈顿距离  
&emsp;&emsp;r = 2时，为欧式距离  
&emsp;&emsp;r →∞时，为切比雪夫距离  
闵可夫斯基距离包括欧式距离、曼哈顿距离、切比雪夫距离都假设数据各维属性的量纲和分布（期望、方差）相同，因此适用于度量独立同分布的数据对象。

### 汉明距离
&emsp;&emsp;两个等长字符串之间的汉明距离是两个字符串对应位置的不同字符的个数，也就是将一个字符串变换为另一个字符串所需要替换的最小字符个数，例如
$$Hamming Distance(1001001, 0101101) = 3$$
汉明距离常用于信息编码中。

### 余弦距离
&emsp;&emsp;余弦相似度公式定义如下：
$$cos⁡(x,y)=\dfrac{xy}{\left\|x\right\|\left\|y\right\|} = \dfrac{\Sigma\_{k=1}^n x\_k y\_k}{\sqrt{\Sigma\_{k=1}^n x\_k^2} \sqrt{\Sigma\_{k=1}^n y\_k^2}}$$
余弦相似度实际上是向量$x$和$y$夹角的余弦度量，可用来衡量两个向量方向的差异。如果余弦相似度为$1$，则$x$和$y$之间夹角为$0°$，两向量除模外可认为是相同的；如果预先相似度为$0$，则$x$和$y$之间夹角为$90°$，则认为两向量完全不同。在计算余弦距离时，将向量均规范化成具有长度$1$，因此不用考虑两个数据对象的量值。  
余弦相似度常用来度量文本之间的相似性。文档可以用向量表示，向量的每个属性代表一个特定的词或术语在文档中出现的频率，尽管文档具有大量的属性，但每个文档向量都是稀疏的，具有相对较少的非零属性值。

### 马氏距离
&emsp;&emsp;马氏距离的计算公式如下：
$$mahalanobis(x,y)=(x-y)\Sigma^{-1}(x-y)^T$$
式中，$\Sigma^{-1}$是数据协方差矩阵的逆。
前面的距离度量方法大都假设样本独立同分布、数据属性之间不相关。马氏距离考虑了数据属性之间的相关性，排除了属性间相关性的干扰，而且与量纲无关。若协方差矩阵是对角阵，则马氏距离变成了标准欧式距离；若协方差矩阵是单位矩阵，各个样本向量之间独立同分布，则变成欧式距离。

### Jaccard系数
&emsp;&emsp;Jaccard系数定义为两个集合A和B的交集元素在其并集中所占的比例，即
$$J(A,B)=\dfrac{A\cap B}{A\cup B}$$
对于两个数据对象$x$和$y$，均由$n$个二元属性组成，则
$$J=\dfrac{f\_{11}}{f\_{01}+f\_{10}+f\_{11}}$$
式中，$f\_{11}$为$x$取$1$且$y$取$1$的属性个数，$f\_{01}$为$x$取$0$且$y$取$1$的属性个数，$f\_{10}$为$x$取$1$且$y$取$0$的属性个数。
Jaccard系数适用于处理仅包含非对称的二元属性的对象。
广义Jaccard系数定义如下：
$$EJ(x,y)=\dfrac{xy}{‖x‖^2+‖y‖^2-xy}$$
广义Jaccard系数又称为Tanimoto系数，可用于处理文档数据，并在二元属性情况下归约为Jaccard系数。

### 相关系数
&emsp;&emsp;两个数据对象之间的相关性是对象属性之间线性关系的度量，计算公式如下
$$\rho\_{xy}=\dfrac{s\_{xy}}{s\_x s\_y}$$
$$s\_{xy}=\dfrac{1}{n-1} \Sigma\_{k=1}^n (x\_k-\overline{x})(y\_k-\overline{y})$$
$$s\_x=\sqrt{\dfrac{1}{n-1} \Sigma\_{k=1}^n (x\_k-\overline{x})^2}$$
$$s\_y=\sqrt{\dfrac{1}{n-1} \Sigma\_{k=1}^n (y\_k-\overline{y})^2}$$
$$\overline{x}=\dfrac{1}{n} \Sigma\_{k=1}^n x\_k ,&emsp;\overline{y}=\dfrac{1}{n} \Sigma\_{k=1}^n y\_k$$ 
相关系数是衡量数据对象相关程度的一种方法，取值范围为$[-1,1]$。相关系数的绝对值越大，则表明$x$和$y$相关度越高。当$x$和$y$线性相关时，相关系数取值为$1$（正线性相关）或$-1$（负线性相关）；线性无关时取值为$0$。在线性回归中，使用直线拟合样本点，可利用相关系数度量其线性性。

### 信息熵
&emsp;&emsp;信息熵描述的是整个系统内部样本之间的一个距离，是衡量分布的混乱程度或分散程度的一种度量。样本分布越分散（或者说分布越平均），信息熵越大；分布越有序（或者说分布越集中），信息熵就越小。给定样本集$X$的信息熵公式定义如下：
$$Entropy(X)=\Sigma\_{i=1}^n -p\_i log\_2⁡(p\_i)$$
式中，$n$为样本集的分类数，$p\_i$为第$i$类元素出现的概率。当$S$中$n$个分类出现的概率一样大时，信息熵取最大值$log2(n)$。当$X$只有一个分类时，信息熵取最小值$0$。信息熵用于度量不确定性，在决策树分类中，信息熵可用于计算子树划分前后的信息增益作为选择最佳划分的度量。





