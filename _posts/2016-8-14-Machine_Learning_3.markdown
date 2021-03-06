---
layout: post
title:  机器学习（三）——生成学习算法, 朴素贝叶斯方法
category: ML 
---

## 广义线性模型（续）

最大似然估计对数函数：

$$\ell(\theta)=\sum_{i=1}^m\log p(y^{(i)}\mid x^{(i)};\theta)=\sum_{i=1}^m\log\prod_{l=1}^k\left(\frac{\exp(\theta_{l}^Tx^{(i)})}{\sum_{j=1}^k\exp(\theta_j^Tx^{(i)})}\right)^{1\{y^{(i)}=l\}}$$

参考：

http://statmath.wu.ac.at/courses/heather_turner/

INTRODUCTION TO GENERALIZED LINEAR MODELS

https://mp.weixin.qq.com/s/jeloJDgfa3eFXUPduhesVA

logistic函数和softmax函数

https://mp.weixin.qq.com/s/iGJ7Xt4_QSZOC0fG0yJfhg

从最大似然估计开始，你需要打下的机器学习基石

## 机器学习的优化问题

优化理论和算法是机器学习用于处理问题的重要工具，但是机器学习有自己独特的看待问题的视角，并且其中也有很多和 Optimization 并不直接相关的部分，反过来Machine Learning也对Optimization产生影响。

例如，Interior Point Method的发明被认为是Optimization中的重要里程碑，这一类的方法能够保证在多项式次迭代内收敛。但是在机器学习中，特别是现在的所谓“大数据”的趋势下，这类算法却没法工作，一方面由于机器学习中所处理的数据通常维度非常高，从而相应的优化问题的变量个数变得很巨大，传统的方法虽然保证多项式迭代收敛，但是其中每一步迭代的计算代价却是随着变量个数的平方甚至三次方增长，结果是连算法的一次迭代都无法在可接受的时间内完成。于是（机器学习方面的）人们逐渐将注意力集中到主要基于first-order oracle的单次迭代计算量非常小的算法上。另一方面，数据点的个数的爆炸性增长也使得stochastic类的算法受到更多的关注——同样是降低单次迭代的计算复杂度。

# 生成学习算法

比如说，要确定一只羊是山羊还是绵羊。从历史数据中学习到模型，然后通过提取这只羊的特征，来预测出这只羊是山羊还是绵羊。这种方法叫做判别学习算法（DLA，Discriminative Learning Algorithm）。其形式化的写法是：$$p(y\mid x)$$。

换一种思路，我们可以根据山羊的特征首先学习出一个山羊模型，然后根据绵羊的特征学习出一个绵羊模型。然后从这只羊中提取特征，放到山羊模型中看概率是多少，再放到绵羊模型中看概率是多少，哪个大就是哪个。这种方法叫做生成学习算法（GLA，Generative Learning Algorithms）。其形式化的写法是：建立模型——$$p(x\mid y)$$，应用模型——$$p(y)$$。

由贝叶斯（Bayes）公式可知：

$$p(y\mid x)=\frac{p(x\mid y)p(y)}{p(x\mid y=1)p(y=1)+p(x\mid y=0)p(y=0)}=\frac{p(x\mid y)p(y)}{p(x)} \tag{6}$$

其中，$$p(x\mid y)$$称为后验概率，$$p(y)$$称为先验概率。

>注：Thomas Bayes，1701~1761，英国统计学家。

由于我们关注的是y的离散值结果中哪个概率大（比如山羊概率和绵羊概率哪个大），而并不是关心具体的概率，因此公式6可改写为：

$$\arg\max_yp(y\mid x)=\arg\max_y\frac{p(x\mid y)p(y)}{p(x)}=\arg\max_yp(x\mid y)p(y) \tag{7}$$

先验/后验概率还可从参数估计的角度来考虑：

$$p(\theta\mid x)=\frac{p(x\mid \theta)p(\theta)}{p(x)}$$

这里的$$\theta$$表示模型的参数。先验概率是根据经验设定的参数值，后验概率是样本实测值，代入Bayes公式即可得到参数的真实值。

常见的判别式模型有Logistic Regression，Linear Regression，SVM，Traditional Neural Networks，Nearest Neighbor，CRF等。

常见的生成式模型有Naive Bayes，Mixtures of Gaussians， HMMs，Markov Random Fields等。

判别式模型，优点是分类边界灵活，学习简单，性能较好；缺点是不能得到概率分布。

生成式模型，优点是收敛速度快，可学习分布，可应对隐变量；缺点是学习复杂，分类性能较差。

![](/images/img2/Discriminative_Generative.jpg)

上面是一个分类例子，可知判别式模型，有清晰的分界面，而生成式模型，有清晰的概率密度分布。生成式模型，可以转换为判别式模型，反之则不能。

参考：

https://zhuanlan.zhihu.com/p/42726550

为什么要对参数设先验(Prior)

https://mp.weixin.qq.com/s/6_BSs7SK2HWq0-7RgNeuzA

理解生成模型与判别模型

## 高斯分布的向量形式

高斯分布的向量形式$$N(\mu,\Sigma)$$的概率密度函数为：

$$p(x;\mu,\Sigma)=\frac{1}{(2\pi)^{n/2}\lvert\Sigma\rvert^{1/2}}\exp\left(-\frac{1}{2}(x-\mu)^T\Sigma^{-1}(x-\mu)\right)$$

其中，$$\mu$$表示均值向量（Mean Vector），$$\Sigma$$表示协方差矩阵（Covariance Matrix），$$\lvert\Sigma\rvert$$表示协方差矩阵的行列式。

## 矩阵行列式计算

对于高阶矩阵行列式，一般采用莱布尼茨公式（Leibniz Formula）或拉普拉斯公式（Laplace Formula）计算。

首先，定义排列A的反序向量V（Inversion Vector）。下面举一个包含6个元素的例子：

| 序列 | 4 1 5 2 6 3 |
| 反序向量 | 0 1 0 2 0 3 |

$$V_i=\sum_{j=1}^{i-1}f(i,j),
f(i,j)=\begin{cases}
1, & A_i<A_j \\
0, & A_i>A_j \\
\end{cases}$$

反序向量的模被称为总序数（Total Order），例如上面例子的总序数为$$1+2+3=6$$。

总序数为奇数的排列被称为奇排列（Odd Permutations），为偶数的排列被称为偶排列（Even Permutations）。

定义勒维奇维塔符号(Levi-Civita symbol)如下：

$$\varepsilon_{a_1 a_2 a_3 \ldots a_n} =
\begin{cases}
+1 & \text{if }(a_1 , a_2 , a_3 , \ldots , a_n) \text{ is an even permutation of } (1,2,3,\dots,n) \\
-1 & \text{if }(a_1 , a_2 , a_3 , \ldots , a_n) \text{ is an odd permutation of } (1,2,3,\dots,n) \\
0 & \text{otherwise}
\end{cases}$$

>注：Tullio Levi-Civita，1873~1941，意大利数学家。他在张量微积分领域的贡献，帮助了相对论的确立。

莱布尼茨公式：

$$\det(A) = \sum_{i_1,i_2,\ldots,i_n=1}^n \varepsilon_{i_1\cdots i_n}  a_{1,i_1} \cdots a_{n,i_n}$$

## 高斯判别分析

高斯判别分析（GDA，Gaussian Discriminant Analysis）模型需要满足以下条件：

$$y\sim Bernoulli(\phi)$$

$$x\mid y=0\sim N(\mu_0,\Sigma)$$

$$x\mid y=1\sim N(\mu_1,\Sigma)$$

>注：这里只讨论y有两种分类的情况，且假设两种分类的$$\Sigma$$相同。

相应的概率密度函数为：

$$p(y)=\phi^y(1-\phi)^{1-y}$$

$$p(x\mid y=0)=\frac{1}{(2\pi)^{n/2}\lvert\Sigma\rvert^{1/2}}\exp\left(-\frac{1}{2}(x-\mu_0)^T\Sigma^{-1}(x-\mu_0)\right)=\frac{1}{A}\exp(f(\mu_0,\Sigma,x))$$

$$p(x\mid y=1)=\frac{1}{(2\pi)^{n/2}\lvert\Sigma\rvert^{1/2}}\exp\left(-\frac{1}{2}(x-\mu_1)^T\Sigma^{-1}(x-\mu_1)\right)=\frac{1}{A}\exp(f(\mu_1,\Sigma,x))$$

将上面三个分布的概率密度函数代入《机器学习（二）》公式7，可求得$$\arg\max_yp(y\mid x)$$，然后进行最大似然估计，可得该GDA的最大似然估计参数为：（过程略）

$$\phi=\frac{1}{m}\sum_{i=1}^m1\{y^{(i)}=1\}$$

$$\mu_0=\frac{\sum_{i=1}^m1\{y^{(i)}=0\}x^{(i)}}{\sum_{i=1}^m1\{y^{(i)}=0\}}$$

$$\mu_1=\frac{\sum_{i=1}^m1\{y^{(i)}=1\}x^{(i)}}{\sum_{i=1}^m1\{y^{(i)}=1\}}$$

$$\Sigma=\frac{1}{m}\sum_{i=1}^m(x^{(i)}-\mu_{y^{(i)}})(x^{(i)}-\mu_{y^{(i)}})^T$$

![](/images/article/GDA.png)

上图中的直线就是分界线$$p(y=1\mid x)=0.5$$。

## GDA vs 逻辑回归

$$\begin{align}p(y=1\mid x)&=\frac{p(x\mid y=1)p(y=1)}{p(x\mid y=1)p(y=1)+p(x\mid y=0)p(y=0)}
\\&=\frac{\frac{1}{A}\exp(f(\mu_0,\Sigma,x))\phi}{\frac{1}{A}\exp(f(\mu_0,\Sigma,x))\phi + \frac{1}{A}\exp(f(\mu_1,\Sigma,x))(1-\phi)}
\\&=\frac{1}{1+\frac{\exp(f(\mu_1,\Sigma,x))(1-\phi)}{\exp(f(\mu_0,\Sigma,x))\phi}}
\\&=\frac{1}{1+\exp(f(\mu_1,\Sigma,x)+\log(1-\phi)-f(\mu_0,\Sigma,x)-\log(\phi))}
\\&=\frac{1}{1+\exp(-\theta^Tx)}
\end{align}$$

从上面的变换可以看出，GDA是逻辑回归的特例。

一般来说，GDA的条件比逻辑回归严格。因此，如果模型符合GDA的话，采用GDA方法，收敛速度（指所需训练集的数量）比较快。

而逻辑回归的鲁棒性较好，对于非GDA模型或者模型不够准确的情况，仍能收敛。

## 互不相容事件、独立事件与条件独立事件

如果$$P(AB)=0$$，则事件A、B为互不相容事件。

如果$$P(AB)=P(A)P(B)或P(A)=P(A\mid B)$$，则事件A、B为独立事件。

如果$$P(AB\mid R)=P(A\mid R)P(B\mid R)$$，则事件A、B为条件R下的独立事件。

可见，这三者是完全不同的数学概念，不要搞混了。

## 朴素贝叶斯方法

这里以文本分析（Text Classification）为例，讲解一下朴素贝叶斯（Naive Bayes）方法。

文本分析的一个常用方法是根据词典建立特征向量x。x中的每个元素代表词典里的一个单词，如果该单词在文本中出现，则对应的元素值为1,否则为0。

这里假设我们的目标是通过文本分析，判别一个email是否是垃圾邮件。$$y=1$$表示是垃圾邮件，$$y=0$$表示不是垃圾邮件。显然，在这里一封邮件就是一个样本集。

相比之前的应用领域，文本分析的特殊之处在于词典中的单词数量十分庞大，从而导致x的维数巨大（比如50000个单词，就是50000维），以至于之前的方法，根本无法计算。

为了简化问题，我们假设$$p(x_i\mid y)$$是条件独立的。这个假设被称为朴素贝叶斯假设（Naive Bayes (NB) assumption）。使用这个假设的算法被称为朴素贝叶斯分类器（Naive Bayes classifier）。

从数学角度，NB假设是个很严格的条件，但是实际使用中，即使样本集不满足NB假设，使用NB方法的效果一般还是不错的。

$$\begin{align}p(x_1,\dots,x_{50000}\mid y)&=p(x_1\mid y)p(x_2\mid y,x_1)p(x_3\mid y,x_1,x_2)\cdots p(x_{50000}\mid y,x_1,\dots,x_{49999})(条件概率的乘法公式)
\\&=p(x_1\mid y)p(x_2\mid y)p(x_3\mid y)\cdots p(x_{50000}\mid y)(NB假设)=\prod_{i=1}^np(x_i\mid y)
\end{align}$$

因此：

$$\begin{align}p(y=1\mid x)&=\frac{p(x\mid y=1)p(y=1)}{p(x)}
\\&=\frac{(\prod_{i=1}^np(x_i\mid y=1))p(y=1)}{(\prod_{i=1}^np(x_i\mid y=1))p(y=1)+(\prod_{i=1}^np(x_i\mid y=0))p(y=0)}\tag{1}
\end{align}$$

最大似然估计值为：

$$\phi_{j\mid y=1}=p(x_j=1\mid y=1)=\frac{\sum_{i=1}^m1\{x_j^{(i)}=1\land y^{(i)}=1\}}{\sum_{i=1}^m1\{y^{(i)}=1\}}$$

$$\phi_{j\mid y=0}=p(x_j=1\mid y=0)=\frac{\sum_{i=1}^m1\{x_j^{(i)}=1\land y^{(i)}=0\}}{\sum_{i=1}^m1\{y^{(i)}=0\}}$$

$$\phi_y=p(y=1)=\frac{\sum_{i=1}^m1\{y^{(i)}=1\}}{m}$$

NB算法还可以扩展到y有多个取值的情况。对于y为连续函数的情况，可以采用区间离散化操作，将之离散化为多个离散值。

参考：

https://mp.weixin.qq.com/s/e5Sa66HZly2N0sCqoYNSeA

一文读懂贝叶斯分类算法

https://mp.weixin.qq.com/s/xNZ0x3_o77C1KL1WTdYkfQ

解读实践中最广泛应用的分类模型：朴素贝叶斯算法

https://mp.weixin.qq.com/s/KNV0KfWktmsPUtQ93a7cDQ

这个男人嫁还是不嫁？懂点朴素贝叶斯(Naive Bayes)原理让你更幸福

