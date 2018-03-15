---
title: Deep Learning -- Optimization
date: 2017-09-15
categories: Algorithm
tags: DL
---
&emsp;&emsp;在机器学习中，模型优化通常会定义一个代价函数(Loss Function)，然后通过最小化代价函数，求得一组参数，例如Logistic Regression、SVM以及神经网络等都属于这类问题，而这类模型往往使用迭代法求解，其中梯度下降法(Gradient Descent)应用最为广泛。
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

### Gradient Descent
&emsp;&emsp;由一组模型参数$\theta$定义的目标函数$J(\theta)$，梯度下降法通过计算目标函数对所有参数的梯度$\nabla \_{\theta}J(\theta)$，然后往梯度相反的方向更新参数，使得目标函数逐步下降，最终趋于（局部）极小值。
$$\theta = \theta - \eta \cdot \nabla_\theta J( \theta)$$
学习率$\eta$决定每次更新的步长。梯度下降法有三种变体：Batch gradient descent、Stochastic gradient descent、Mini-batch gradient descent。不同之处在于每次迭代训练使用的样本数。
#### Batch Gradient Descent
&emsp;&emsp;BGD每次迭代计算所有训练样本的代价函数（均值），并求其梯度用于更新参数，这种方法在较小的学习率下能够保证收敛到全局最小（凸函数）或者局部最小（非凸函数）。但每次更新参数需要计算整个训练集的梯度，造成训练缓慢；样本数据较多的情况下无法全部放入内存；并且不能在线(Online)更新。
#### Stochastic Gradient Descent
&emsp;&emsp;SGD和BGD相反，每次迭代训练只计算一个样本的梯度，对参数进行更新。相比于Batch gradient descent对大数据集执行冗余计算，每次更新参数之前都要重复计算相似样本的梯度，SGD通过每次计算单个样本可以消除这种冗余，参数更新较快，可用于在线学习。而且SGD在迭代的过程中波动较大，可能从一个局部最优跳到另一个更好的局部最优，甚至是全局最优。但SGD没法利用矩阵运算加速计算过程。
#### Mini-batch Gradient Descent
&emsp;&emsp;Mini-batch Gradient Descent相当于是BGD和SGD的折衷，每次迭代训练从数据集中选取一部分样本(minibatch)计算梯度值。这种方法减少了参数更新时的波动，使代价函数整体上呈下降趋势，达到了平稳收敛的效果；而且可以有效利用矩阵计算工具加速训练，通常minibatch的取值范围为[50, 256]。由于minibatch的方法使用较为广泛，因此现在的SGD一般是指Mini-batch Gradient Descent。

#### Challenges
&emsp;&emsp;上述几种梯度下降法并不能保证模型达到最优的效果，还有一些比较有挑战性的问题需要解决：
- 选取合适的学习率是困难的。学习率较小导致训练缓慢；学习率较大造成收敛困难，代价函数在最小值附近波动甚至发散
- 通过Schedule的方式在训练过程中调整学习率，例如迭代到一定次数后降低学习率，但这些都需要提前设置好，不能适应训练数据的特性
- 所有参数更新的学习率是相同的，如果训练数据是稀疏的，而且特征出现的频率不一样，比较合适的做法是对出现频率低的特征采用较大的学习率更新
- 对于非凸函数，可能会陷入鞍点(saddle points)而导致训练困难，在鞍点区域某些维度呈上升趋势，而另外一些呈下降趋势，鞍点的梯度在所有维度都趋近于0，导致代价函数陷入其中，参数更新困难

### Momentum
&emsp;&emsp;SGD存在一个缺陷就是，当某一个方向(维度)梯度较大、另一个方向梯度较小时，代价函数可能会在大梯度方向以大步长来回振荡，而在小梯度方向缓慢前进，造成收敛缓慢。Momentum受到物理学中动量的启发，能够在相关方向加速SGD，并抑制振荡，加快收敛
<img src="https://i.loli.net/2017/12/23/5a3e02df1d688.jpg" alt="momentum.jpg" title="SGD without momentum(left) and with momentum(right)" />

\begin{split}
v\_t &= \gamma v\_{t-1} + \eta \nabla\_\theta J(\theta) \cr
\theta &= \theta - v\_t
\end{split}

由上式可以看到，当前梯度的方向要与之前的梯度方向进行加权平均，$\gamma$一般取值为0.9。在梯度方向不变的维度，momentum项增加；在梯度方向频繁改变的维度，momentum项更新减少。经过一段时间动量项的累加，可以在一定程度上抑制振荡，加速收敛。

### Nesterov accelerated gradient(NAG)
&emsp;&emsp;在梯度下降更新参数的过程中，有时候希望提前知道下一步到达位置的梯度，以便及时调整参数更新方向。NAG实现了这一想法，利用Momentum预测下一步的梯度，而不是直接使用当前的$\theta$

\begin{split}
v\_t &= \gamma v\_{t-1} + \eta \nabla\_\theta J( \theta - \gamma v\_{t-1} ) \cr
\theta &= \theta - v\_t
\end{split}

动量项系数$\gamma$取值为0.9。在上面提到的Momentum方法中，首先计算当前参数(位置)的梯度，然后更新动量项，并沿着动量项（更新后）的方向更新参数；而在NAG方法中，首先沿着之前累积的动量项的方向更新参数，然后基于更新后的参数计算梯度，并对参数进行调整。两种方法的对比如下图所示：
<img src="https://i.loli.net/2017/09/17/59be22509c283.jpeg" alt="nesterov.jpeg" title="Momentum and NAG" />
详细介绍可参考[cs231n-sgd](http://cs231n.github.io/neural-networks-3/#sgd-and-bells-and-whistles)

### Adagrad
&emsp;&emsp;Adagrad是一种在学习过程中自动调整学习率的参数优化方法，不同参数采用不同的学习率进行更新。出现频率低的参数更新较大，频率高的参数更新较小，因此特别适用于处理稀疏数据，而且Adagrad提高了SGD的鲁棒性

\begin{split}
g\_t &= \nabla\_\theta J( \theta) \cr
G\_t &= G\_{t-1} + g\_t^2 \cr
\theta\_{t+1} &= \theta\_t -  \dfrac{\eta}{\sqrt{G\_t + \varepsilon}} \cdot g\_t
\end{split}

$t$表示迭代次数，$G\_t是每个参数的梯度平方的累加和$，使用$\sqrt{G\_t + \varepsilon}$作为约束项自动调整每个参数在训练过程中的学习率。对于梯度较大的参数，有效学习率会降低；而梯度较小的参数，有效学习率则相对会增大。但对每个参数而言，由于梯度平方和的累加，学习率实际上一直是递减的，这也是Adagrad的一个缺陷，可能会导致学习率降低至0，过早结束训练过程。

### Adadelta
&emsp;&emsp;Adagrad方法比较激进，会过早结束训练过程。在对参数进行调整时，Adagrad使用的是每次迭代的梯度累加和，而在Adadelta中则通过衰减平均(decaying average)的方式计算梯度平方的加权均值
$$E[g^2]\_t = \gamma E[g^2]\_{t-1} + (1 - \gamma) g^2\_t$$
$\gamma$可取值为0.9(0.95)，使用$E[g^2]_t$调整学习率
$$\Delta \theta\_t = - \dfrac{\eta}{\sqrt{E[g^2]\_t + \epsilon}} g\_{t}$$
考虑到分母为梯度的均方根(root mean squared, RMS)，公式可替换为
$$\Delta \theta\_t = - \dfrac{\eta}{RMS[g]\_{t}} g\_t,&emsp;RMS[g]\_t=\sqrt{E\left[g^2\right]\_t+\epsilon}$$
将学习率$\eta$替换为$RMS[\Delta \theta]\_{t-1}$，利用之前的步长估计下一步的步长，公式如下：

\begin{split}
\Delta \theta\_t &= - \dfrac{RMS[\Delta \theta]\_{t-1}}{RMS[g]\_{t}} g\_{t} \\\
\theta\_{t+1} &= \theta\_t + \Delta \theta\_t 
\end{split}

式中，$RMS[\Delta \theta]\_{t} = \sqrt{E[\Delta \theta^2]\_t + \epsilon},E[\Delta \theta^2]\_t = \gamma E[\Delta \theta^2]\_{t-1} + (1 - \gamma) \Delta \theta^2\_t$，在Adadelta中，不需要设置默认学习率，其自身可通过迭代计算得到。详细讲解参考原论文[Adadelta](https://arxiv.org/pdf/1212.5701v1.pdf)

### RMSprop
&emsp;&emsp;RMSprop同样是用来解决Adagrad学习率趋于0、过早结束训练的方法，来源于Hinton大神的[slide](http://218.199.87.242/cache/9/03/www.cs.toronto.edu/a1dce074ab79c3b6a179c02438d76249/lecture_slides_lec6.pdf)，而且思想基本和Adadelta一致，公式就是上面Adadelta方法中的前两项：

\begin{split}
E[g^2]\_t &= \gamma E[g^2]\_{t-1} + (1-\gamma) g^2\_t \\\  
\theta\_{t+1} &= \theta\_{t} - \dfrac{\eta}{\sqrt{E[g^2]\_t + \epsilon}} g\_{t}
\end{split}

$\gamma$为权重衰减率，一般可设置为0.9、0.99、0.999，$\eta$为学习率，可取值为0.001。

### Adam
&emsp;&emsp;Adaptive Moment Estimation(Adam)是一种结合了Momentum和RMSprop优点的方法，利用指数衰减平均的方法(exponentially decaying average)计算梯度平方的加权均值$v\_t$，类似于Adadelta和RMSprop，并且基于同样的方法计算梯度值的加权均值$m\_t$，相当于引入动量项

\begin{split}
m\_t &= \beta\_1 m_{t-1} + (1 - \beta\_1) g\_t \\\ 
v\_t &= \beta\_2 v_{t-1} + (1 - \beta\_2) g\_t^2  
\end{split}

$m\_t$和$v\_t$分别是梯度的一阶矩（均值）和二阶矩（方差）估计，$\beta_1$和$\beta_2$为衰减率，一般取值为$\beta_1=0.9$、$\beta_2=0.999$。由于$m\_t$和$v\_t$在初始几步的取值较小，趋近于0，Adam对此进行了偏差校正(Bias Correction)：

\begin{split}
\hat{m}\_t &= \dfrac{m\_t}{1 - \beta^t\_1} \\\
\hat{v}\_t &= \dfrac{v\_t}{1 - \beta^t\_2} 
\end{split}

参数更新规则类似于RMSprop：
$$\theta\_{t+1} = \theta\_{t} - \dfrac{\eta}{\sqrt{\hat{v}\_t} + \epsilon} \hat{m}\_t$$
Adam集成了多种优化方法的优点，既适用于处理稀疏数据，又能够达到平稳快速收敛的效果，应用十分广泛。

### AdaMax
&emsp;&emsp;AdaMax是Adam的一种变体，在Adam中计算$v\_t$时考虑的是$\ell_2$范数(l2-norm)，AdaMax将这种约束推广到$\ell\_p$范数：
$$v\_t = \beta\_2^p v\_{t-1} + (1 - \beta\_2^p) |g\_t|^p$$
$\ell\_\infty$能够取得较为稳定的结果，因此在AdaMax中被采纳

\begin{split}
u\_t &= \beta\_2^\infty v\_{t-1} + (1 - \beta\_2^\infty) |g\_t|^\infty\\\ 
              & = \max(\beta\_2 \cdot v\_{t-1}, |g\_t|)\\\
              \theta\_{t+1} &= \theta\_{t} - \dfrac{\eta}{u\_t} \hat{m}\_t
\end{split}

$u\_t$取两项中的较大值，不需要再执行偏差校正操作，$\hat{m}\_t$仍然和Adam中一样，超参数取值分别为$\eta = 0.002,\beta\_1=0.9,\beta\_2=0.999$。

### Nadam
&emsp;&emsp;Nadam类似于带有Nesterov动量项的Adam，相当于NAG和Adam的组合，在NAG中，参数更新规则为：

\begin{split}
g\_t &= \nabla\_{\theta\_t}J(\theta\_t - \gamma m\_{t-1})\\\ 
m\_t &= \gamma m\_{t-1} + \eta g\_t\\\  
\theta\_{t+1} &= \theta\_t - m\_t
\end{split}

在NAG中既要更新梯度$g\_t$，又要更新参数$\theta\_{t+1}$，Nadam作者Dozat直接应用look-ahead momentum更新当前参数

\begin{split}
g\_t &= \nabla\_{\theta\_t}J(\theta\_t)\\\  
m\_t &= \gamma m\_{t-1} + \eta g\_t\\\  
\theta\_{t+1} &= \theta\_t - (\gamma m\_t + \eta g\_t)
\end{split}

注意到在更新参数时，用的不是previous momentum $m\_{t-1}$，而是current momentum $m\_t$。Adam的参数更新规则为

\begin{split}
m\_t &= \beta\_1 m\_{t-1} + (1 - \beta\_1) g\_t\\\  
\hat{m}\_t & = \frac{m\_t}{1 - \beta^t\_1}\\\
\theta\_{t+1} &= \theta\_{t} - \frac{\eta}{\sqrt{\hat{v}\_t} + \epsilon} \hat{m}\_t
\end{split}

为了将Nesterov momentum加入Adam中，需要扩展参数更新式：

\begin{split}
\theta\_{t+1} = \theta\_{t} - \dfrac{\eta}{\sqrt{\hat{v}\_t} + \epsilon} (\dfrac{\beta\_1 m\_{t-1}}{1 - \beta^t\_1} + \dfrac{(1 - \beta\_1) g\_t}{1 - \beta^t\_1}) \\
&= \theta\_{t+1} = \theta\_{t} - \dfrac{\eta}{\sqrt{\hat{v}\_t} + \epsilon} (\beta\_1 \hat{m}\_{t-1} + \dfrac{(1 - \beta\_1) g\_t}{1 - \beta^t\_1})
\end{split}

将$\hat{m}\_{t-1}$替换为$\hat{m}\_t$，即可得到Nadam的参数更新规则
$$\theta\_{t+1} = \theta\_{t} - \dfrac{\eta}{\sqrt{\hat{v}\_t} + \epsilon} (\beta\_1 \hat{m}\_t + \dfrac{(1 - \beta\_1) g\_t}{1 - \beta^t\_1})$$
详细内容参考论文[Nadam](http://cs229.stanford.edu/proj2015/054_report.pdf)

### Visualization of Algorithms
&emsp;&emsp;下面两幅图直观地展示了各种优化方法的性能，从左图中可以看出，Adagrad、Adadelta、RMSprop能够朝着正确的方向更新参数，而Momentum和NAG则在一开始偏离了正确路线，但最后均完成了收敛，而且NAG可以较为迅速地校正参数更新方向。右图展示了各种方法在鞍点处的表现，可以看出SGD、Momentum和NAG很难打破鞍点处的平衡(symmetry)，不过Momentum和NAG最终还是能够逃逸出来，而Adagrad、Adadelta、RMSprop则能够迅速朝代价函数的下降方向更新参数。
<table>  
  <tr>
    <td style="padding:1px">
      <figure>
      <img src="https://i.loli.net/2017/09/17/59be6494370fd.gif" style="width: 100%; height: 100%" title="SGD optimization on loss surface contours">
</figure>  
    </td>
    <td style="padding:1px">
      <figure>
      <img src="https://i.loli.net/2017/09/17/59be6493e3d8b.gif" style="width: 100%; height: 100%" title="SGD optimization on saddle point">
</figure>  
    </td>
  </tr>
</table>
### Which Optimizer to use
- 对于稀疏数据，尽可能地选择能够自适应调整学习率(adaptive learning rate)的优化方法，不需手动调节，在设置好默认值的情况下可以实现较好的结果
- RMSprop、Adadelta是Adagrad的扩展，解决了Adagrad学习率递减、过早结束训练的问题；RMSprop、Adadelta、Adam在相似的情况下表现差不多，相对而言Adam能够达到更好的效果
- SGD通常训练时间更长，可能陷入鞍点，但是在好的参数初始化和学习率调度方案的情况下，结果依然十分可靠
- 在训练深层或复杂的网络时，如果考虑快速收敛，一般选择Adam这类自适应调整学习率的优化方法

### reference
- http://ruder.io/optimizing-gradient-descent/
- [Paper: An overview of gradient descent optimization algorithms](https://arxiv.org/pdf/1609.04747.pdf)
- http://cs231n.github.io/optimization-1/
- http://ufldl.stanford.edu/wiki/index.php/%E5%8F%8D%E5%90%91%E4%BC%A0%E5%AF%BC%E7%AE%97%E6%B3%95
- http://cs231n.github.io/neural-networks-3/
- http://blog.mrtanke.com/2016/10/24/An-overview-of-gradient-descent-optimization-algorithms/
- http://shuokay.com/2016/06/11/optimization/
- https://blog.slinuxer.com/2016/09/sgd-comparison
- https://json0071.gitbooks.io/svm/content/sui_ji_ti_du_xia_jiang.html
- https://zhuanlan.zhihu.com/p/22252270