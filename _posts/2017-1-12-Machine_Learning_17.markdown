---
layout: post
title:  机器学习（十七）——主成分分析
category: ML 
---

# 主成分分析

真实的训练数据总是存在各种各样的问题。

比如拿到一个汽车的样本，里面既有以“千米/每小时”度量的最大速度特征，也有“英里/小时”的最大速度特征。显然这两个特征有一个是多余的，我们需要找到，并去除这个冗余。

再比如，针对飞行员的调查，包含两个特征：飞行的技能水平和对飞行的爱好程度。由于飞行员是很难培训的，因此如果没有对飞行的热爱，也就很难学好飞行。所以这两个特征实际上是强相关的（strongly correlated）。如下图所示：

![](/images/article/PCA.png)

我们的目标就是找出上图中所示的向量$$u_1$$。

为了实现这两个目标，我们可以采用PCA（Principal components analysis）算法。

## 数据的规则化处理

在进行PCA算法之前，我们首先要对数据进行预处理，使之规则化。其方法如下：

>1.$$\mu=\frac{1}{m}\sum_{i=1}^mx^{(i)}$$   
>2.$$x^{(i)}:=x^{(i)}-\mu$$   
>3.$$\sigma_j^2=\frac{1}{m}\sum_i(x^{(i)})^2$$   
>4.$$x_j^{(i)}:=x_j^{(i)}/\sigma_j$$   

多数情况下，特征空间中，不同特征向量所代表的维度之间，并不能直接比较。

比如，摄氏度和华氏度，虽然都是温度的单位，但两种温标的原点和尺度都不相同，因此需要规范化之后才能比较。

步骤1和2，用于消除原点偏差（常数项偏差）。步骤3和4，用于统一尺度（一次项偏差）。

虽然上面的办法，对于二次以上的偏差无能为力，然而多数情况下，这种处理，已经比原始状态好多了。

## PCA算法推导

回到之前的话题，为了找到主要的方向u，我们首先观察一下，样本点在u上的投影应该是什么样子的。

![](/images/article/PCA_2.png) | ![](/images/article/PCA_3.png)

上图所示是5个样本在不同向量上的投影情况。其中，X表示样本点，而黑点表示样本在u上的投影。

很显然，左图中的u就是我们需要求解的主成分的方向。和右图相比，左图中各样本点x在u上的投影点比较分散，也就是投影点之间的方差较大。

由《机器学习（十二）》的公式1，可知样本点x在单位向量u上的投影为：$$x^Tu$$。

因此，这个问题的代价函数为：

$$\begin{align}
&\frac{1}{m}\sum_{i=1}^m\left(x^{(i)^T}u\right)^2=\frac{1}{m}\sum_{i=1}^m\left(x^{(i)^T}u\right)^T\left(x^{(i)^T}u\right)
\\&=\frac{1}{m}\sum_{i=1}^mu^Tx^{(i)}x^{(i)^T}u=u^T\left(\frac{1}{m}\sum_{i=1}^mx^{(i)}x^{(i)^T}\right)u=u^T\Sigma u
\end{align}$$

即：

$$\begin{align}
&\operatorname{max}_{u}& & u^T\Sigma u\\
&\operatorname{s.t.}& & u^Tu=1
\end{align}$$

其中的矩阵$$\Sigma$$实际上是一个对角阵，对角线元素$$\sigma_{ii}$$表示x的第i维的方差。

其拉格朗日函数为：

$$\mathcal{L}(u)=u^T\Sigma u-\lambda(u^Tu-1)$$

对u求导可得：

$$\nabla_u\mathcal{L}(u)=\Sigma u-\lambda u$$

这里的矩阵求导步骤，参见《机器学习（十）》中的公式5.12的推导过程。

令导数为0可得，当$$\lambda$$为$$\Sigma$$的特征值的时候，该代价函数得到最优解。

>注：这里的推导过程，求解的是1维的PCA，但结论对于k维的PCA也是成立的。

一个n阶矩阵有n个特征值，这些特征值可按绝对值大小排序，绝对值越大的，越重要。其中最大的k个特征值，被称作k principal components，这就是主成分分析（Principal components analysis，PCA）算法的命名来历。

我们以最大的k个特征值所对应的特征向量，构建样本空间Y：

$$y^{(i)}=\begin{bmatrix}
u_1^Tx^{(i)}\\
u_2^Tx^{(i)}\\
\cdots \\
u_k^Tx^{(i)}
\end{bmatrix}\in R^k\tag{1}$$

可以看出PCA算法实际上是个**降维（dimensionality reduction）**算法。

## PCA相关系数的推导

$$\rho(y_k,x_i)$$表示第k个主成分与第i个变量的相关系数，也被称为因子负荷或主成分负荷。

**推导**：

相关系数的定义为：

$$\rho_{XY}=\frac{Cov(X,Y)}{\sqrt{D(X)}\sqrt{D(Y)}}$$

由特征值的定义可得：

$$u_i^T\Sigma u_i=u_i^T\lambda_i u_i=\lambda_iu_i^Tu_i$$

因为u是单位正交阵，所以：

$$u_i^T\Sigma u_i=\lambda_i$$

$$Var(y_k)=Var(u_k^Tx)=(u_k^Tx)(u_k^Tx)^T=u_k^Txx^Tu_k=u_k^T\Sigma u_k=\lambda_k$$

令$$x=Ay$$，则：

$$Cov(y_k,x_i)=Cov(y_k,a_{ik}y_k)=a_{ik}Cov(y_k,y_k)=a_{ik}Var(y_k)=a_{ik}\lambda_k$$

$$Var(x_i)=\sigma_{ii}$$

综上可得：

$$\rho(y_k,x_i)=\frac{a_{ik}\lambda_k}{\sqrt{\lambda_k}\sqrt{\sigma_{ii}}}=\frac{a_{ik}\sqrt{\lambda_k}}{\sqrt{\sigma_{ii}}}$$

## PCA的用途

为了便于理解PCA算法，我们以如下图片的处理过程为例，进行说明。

![](/images/article/svd.png)

**第一排从左往右依次为：原图（450*333）、k=1、k=5；第二排从左往右依次为：k=20、k=50。**

从中可以看出，k=50时，图像的效果已经和原图相差无几。而原图是个450*333的高阶矩阵。

在图像处理领域，奇异值不仅可以应用在数据压缩上，还可以对图像去噪。如果一副图像包含噪声，我们有理由相信那些较小的奇异值就是由于噪声引起的。当我们强行令这些较小的奇异值为0时，就可以去除图片中的噪声。

PCA的一个很大的优点是，它是完全无参数限制的。在PCA的计算过程中完全不需要人为的设定参数或是根据任何经验模型对计算进行干预，最后的结果只与数据相关，与用户是独立的。

但是，这一点同时也可以看作是缺点。如果用户对观测对象有一定的先验知识，掌握了数据的一些特征，却无法通过参数化等方法对处理过程进行干预，可能会得不到预期的效果，效率也不高。

## PCA和ALS的联系与区别

令：

$$Y_{k\times m}=\begin{bmatrix}y^{(1)} & \cdots & y^{(m)}\end{bmatrix}$$

则由公式1可得：

$$Y_{k\times m}=U_{n\times k}^TX_{n\times m}$$

变换可得：

$$X_{n\times m}\approx U_{n\times k}Y_{k\times m}$$

由于PCA是个降维算法，因此这个变换实际上也是个近似变换。

从上面可以看出，PCA和ALS实际上都是矩阵的降维分解算法。它们的差别在于：

1.PCA的U矩阵是单位正交矩阵，而ALS的分解矩阵则没有这个限制。

2.ALS从原理上虽然也是矩阵的奇异值或特征值的应用，然而其求解过程，却并不涉及矩阵的奇异值或特征值的运算，因此运算效率非常高。

3.ALS是个迭代算法，容易掉入局部最优陷阱中。

## PCA和特征选择的区别

两者虽然都是降维算法，但特征选择是在原有的n个特征中选择k个特征，而PCA是重建k个新的特征。

## 参考

https://mp.weixin.qq.com/s/tJ_FbL2nFQfkvKqpQJ8kmg

从特征分解到协方差矩阵：详细剖析和实现PCA算法

https://mp.weixin.qq.com/s/dDdyaA7Nxqa8tBE_qQ80Dw

典型相关性分析(CCA)详解

https://mp.weixin.qq.com/s/JDWgw3OOdBurDAShrPHJ7Q

从最大方差来看主成分分析PCA

https://mp.weixin.qq.com/s/ZDipXGPOxKhxtAx2Dc9RjA

主成分分析（PCA）以及在图像上的应用

https://mp.weixin.qq.com/s/9-nNNhhDWSYWy46u0hTazQ

降维：PCA真的能改善分类结果吗？

# 独立成分分析

这一节我们将讲述独立成分分析（Independent Components Analysis，ICA）算法。

首先，我们介绍一下经典的鸡尾酒宴会问题(cocktail party problem)。

假设在party中有n个人，他们可以同时说话，我们也在房间中放置了n个声音接收器(Microphone)用来记录声音。宴会过后，我们从n个麦克风中得到了m组数据$$x^{(i)}$$，其中的i表示采样的时间顺序。由于宴会上人们的说话声是混杂在一起的，因此，采样得到的声音也是混杂不清的，那么我们是否有办法从混杂的数据中，提取出每个人的声音呢？

为了更为正式的描述这个问题，我们假设数据$$s\in R^n$$是由n个独立的源生成的。我们接收到的信号可写作：$$x=As$$。其中，A被称为混合矩阵（mixing matrix）。在这个问题中，$$s^{(i)}$$是一个n维向量，$$s_j^{(i)}$$表示第j个说话者在i时刻的声音。同理，$$x_j^{(i)}$$表示第j个麦克风在i时刻的记录下的数据。

我们把$$W=A^{-1}$$称作unmixing matrix。我们的目标就是找到W，然后利用$$s=Wx$$，求得s。我们使用$$w_i^T$$表示W矩阵的第i行，因此：$$s_j^{(i)}=w_j^Tx^{(i)}$$。

## ICA的不确定性

不幸的是，在没有源和混合矩阵的先验知识的情况下，仅凭$$x^{(i)}$$是没有办法求出W的。为了说明这一点，我们引入置换矩阵的概念。

置换矩阵（permutation matrix）是一种元素只由0和1组成的方块矩阵。置换矩阵的每一行和每一列都恰好只有一个1，其余的系数都是0。它的例子如下：

$$P=\begin{bmatrix}0 & 1 & 0 \\ 1 & 0 & 0 \\ 0 & 0 & 1 \end{bmatrix};
P=\begin{bmatrix}0 & 1 \\ 1 & 0 \end{bmatrix};
P=\begin{bmatrix}1 & 0 \\ 0 & 1 \end{bmatrix}$$

在线性代数中，每个n阶的置换矩阵都代表了一个对n个元素（n维空间的基）的置换。当一个矩阵乘上一个置换矩阵时，所得到的是原来矩阵的横行（置换矩阵在左）或纵列（置换矩阵在右）经过置换后得到的矩阵。

ICA的不确定性(ICA ambiguities)包括以下几种情形：

1.无法区分W和WP。比如改变说话人的编号，会改变$$s^{(i)}$$的值，但却不会改变$$x^{(i)}$$的值，因此也就无法确定$$s^{(i)}$$的值了。

2.无法确定W的尺度。比如$$x^{(i)}$$还可以写作$$x^{(i)}=2A \cdot (0.5)s^{(i)}$$，因此在不知道A的情况下，同样无法确定$$s^{(i)}$$的值。

3.信号不能是高斯分布的。

假设两个人发出的声音信号符合多值正态分布$$s\sim \mathcal{N}(0,I)$$，这里的I是一个2阶单位阵，则$$E[xx^T]=E[Ass^TA^T]=AA^T$$。

假设R是正交矩阵，$$A'=AR,x'=A's$$，则：

$$E[xx^T]=E[A'ss^T(A')^T]=E[ARss^T(AR)^T]=ARR^TA^T=AA^T$$

可见，无论是A还是A'，观测值x都是一个$$\mathcal{N}(0,AA^T)$$的正态分布，也就是说A的值无法确定，那么W和s也就无法求出了。

## 密度函数和线性变换

在讨论ICA的具体算法之前，我们先来回顾一下概率和线性代数里的知识。

假设我们的随机变量s有概率密度（probability density）函数$$p_s(s)$$。为了简单，我们再假设s是实数，还有一个随机变量$$x=As$$，A和x都是实数。令$$p_x$$是x的概率密度，那么怎么求$$p_x$$呢？

令$$W=A^{-1}$$，则$$s=Wx$$。然而$$p_x(x)\neq p_s(Wx)$$。

这里以均匀分布（Uniform）为例讨论一下。令$$s\sim \text{Uniform}[0,1]$$，则$$p_s(s)=1$$。令$$A=2$$，则$$W=0.5$$，$$x=2s\sim \text{Uniform}[0,2]$$，因此$$p_x(x)=p_s(Wx)\lvert W \rvert$$。
