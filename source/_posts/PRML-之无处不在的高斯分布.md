---
title: PRML 之无处不在的高斯分布
date: 2017-10-21 23:50:57
tags: 
    - 机器学习
    - 读书笔记
mathjax: true
---

　　之前零零散散看了一些统计学习相关的书和paper时就发现高斯老爷子的分布真是无处不在，可惜每次看别人运用高斯分布的时候，由于对高斯分布理解不深，所以他们的推导过程看着也是一知半解。最近看PRML这本书的时候，发现其中对高斯分布，主要是多元高斯分布，做了很详细地阐述，为了加深自己的理解和记忆，觉得有必要写篇笔记记录一下。

导读：




* [1. 多元高斯分布](#1)

    * [1.1 期望和方差](#1.1)

    * [1.2 条件高斯分布](#1.2)

    * [1.3 边缘高斯分布](#1.3)

    * [1.4 贝叶斯定理](#1.4)

* [2. 参数估计]

    * [2.1 极大似然估计](#2.1)
    
    * [2.2 贝叶斯推断](#2.2)

    
    

<h2 id="1">1. 多元高斯分布</h2>


我们假设一个$D$维向量$x = (x_1,x_2,...,x_D)^T$,意味着有$D$个变量，它们整体服从多元高斯分布：
$$N(x|\mu,\Sigma) = (2\pi)^{-D/2}|\Sigma|^{-1/2}exp(-\frac{1}{2}(x-\mu)^T\Sigma^{-1}(x-\mu))$$ 
其中：

$\mu$：$D$维均值向量(mean vector)，即$\mu = (\mu_1, \mu_2,...,\mu_D)^T$

$\Sigma$：$D\times D$的协方差矩阵(covariance matrix)，$\Sigma_{ij}=(x_i-\mu_i)(x_j-\mu_j)$。


我们可以从$N(x|\mu, \Sigma)$中的指数项出发，理解多元高斯分布的几何意义。

首先，我们来看一下这个指数项  $\Delta^2 = (x-\mu)^T\Sigma^{-1}(x-\mu)$，发现其中主要包含两部分：一个是协方差矩阵$\Sigma$，一个是平移后的$x$。

若$\Sigma$有D个特征值$\lambda_1,\lambda_2,...,\lambda_D$，其对应的特征向量为$u_1, u_2, ..., u_D$，则$$\Sigma = \sum_{i=1}^{D}\lambda_iu_iu_i^T$$
其中，若所有$\lambda_i$都大于0，则说$\Sigma$是正定的(positive definite);若存在$\lambda_i = 0$,则说该分布是奇异的，说明其可以在低维子空间表达；若所有$\lambda$非负，则说明$\Sigma$是半正定的(positive semidefinite)。

$u_i$满足正交归一条件，即:

$$
u_i^Tu_j=I_{ij}=
\begin{cases}
1& {i=j}\\
0& {i\neq j}
\end{cases}
$$




令$U=(u_1,u_2,...,u_D)$，则$UU^T = I$。因此：
$$
\begin{equation}
\Delta^2 = \sum_{i=1}^{D}(x-\mu)^T\lambda_i^{-1}u_iu_i^T(x-\mu)
\end{equation}
$$
令$y_i = u_i^T(x-\mu)$,可以看出这是一个数，那么$y_i^T = (x-\mu)^Tu_i$也是个数。因此，
$$
\begin{equation}
\Delta^2 = \sum_{i=1}^{D}\lambda_i^{-1}y_i^Ty_i=\sum_{i=1}^{D}\lambda_i^{-1}y_i^2
\end{equation}
$$

可见指数项由向量计算变成了普通的数值计算。可以看出，$y_i$就是$u_i$与$x-\mu$的向量内积，也可以看作向量$x$先进行平移将重心移到原点变成$(x-\mu)$，而后在基矢$u_i$上的投影值。因此，$y = (y_1,y_2,...,y_D)^T$ 可看作$x-\mu$在基矢$U$张成的线性空间的新坐标值。所以从$x$到$y$是做了个线性变换，$y = U^T(x-\mu), x= Uy+\mu$。

从$x$到$y$的坐标系统，其Jacobi矩阵为$J_{ij} = \frac{\partial x_i}{\partial y_j}=U_{ij}$,$|J|^2 = |U|^2 = 1$, $|J| =1$。

在$y$的坐标系统中，其分布为:
$$
\begin{equation}
p(y) = p(x)|J| = \prod_{i=1}^{D}(2\pi\lambda_i)^{-1/2}exp(-\frac{y_1^2}{2\lambda_i})
\end{equation}
$$

可见其中没有了矩阵的操作，而是简化成了$D$个普通的单变量高斯分布的相乘，且这个单变量高斯分布中：$\mu = 0, \sigma^2 = \lambda$。
 <!-- more -->

<h3 id="1.1">1.1 期望和方差</h3>

由上节可知，多元高斯分布可以表示为：
$$
\begin{equation}
p(x)=(2\pi)^{-D/2}\Sigma^{-1/2}\int exp(-\frac{1}{2}\Delta^2))
\end{equation}
$$
$$
\Delta^2 = (x-\mu)^T\Sigma^{-1}(x-\mu)
$$

因此，计算其期望(expectation)可先作变换$z=x-\mu$,则
$$
\begin{equation}
E[x] = (2\pi)^{-D/2}\Sigma^{-1/2}\int exp(-\frac{1}{2}z^T\Sigma^{-1}z)(z+\mu)dz
\end{equation}
$$

可见指数项是关于$z$的偶函数，而积分区间是$(-\infty,\infty)$,由对称性可知,与$z$的相乘项为0，积分简化为
$$
\begin{equation}
 E[x]=\mu \cdot (2\pi)^{-D/2}\Sigma^{-1/2}\int exp(-\frac{1}{2}z^T\Sigma^{-1}z)dz
\end{equation}
$$

$\mu$后面的部分由归一化条件可知为1，所以，$E[x]=\mu$。

对多元高斯分布，我们对其方差，或者说协方差的求法和单变量高斯分布的类似，对单变量高斯分布，其一阶矩$E[x]=\mu$，二阶矩$E[x^2]=\mu^2+\sigma^2$,即方差$\sigma^2 = E[x^2]-E[x]^2$。因此，对多元高斯分布，我们首先求其二阶矩$E[x_ix_j]$,即$E[xx^T]$。

$$
\begin{equation}  
E[xx^T] = (2\pi)^{-D/2}\Sigma^{-1/2}\int exp(-\frac{1}{2}z^T\Sigma^{-1}z)(z+\mu)(z+\mu)^Tdz
\end{equation}
$$

由对称性可知，所有的$\mu z^T,z\mu^T$项都为0，而$\mu\mu^T$为常数项，事实上，由上一步求期望可知，该项值其实就为$\mu\mu^T$。所以，在这里，我们需要考虑的只有 $zz^T$ 项。

由上一部分的内容可知，我们可以通过将$z$投影到新的$y$坐标系,即$z=\sum_{j=1}^{D}y_ju_j$，使计算大大简化。依此，$zz^T$项可表示为：

$$
\begin{eqnarray}
E[zz^T]&=&(2\pi)^{-D/2}\Sigma^{-1/2}\int exp(-\frac{1}{2}z^T\Sigma^{-1}z)zz^Tdz\\
& = &(2\pi)^{-D/2}\Sigma^{-1/2}\sum_{i=1,j=1}^{D}u_iu_j^T\int exp(-\frac{1}{2}\sum_{k=1}^D \frac{y_k^2}{\lambda_k})y_iy_jdy
\end{eqnarray}
$$
由对称性可知，只有$i=j=k$的项保留，且由于此处可运用单变量高斯分布中的结论$E[x^2]=\mu^2+\lambda^2$,而此积分项中，对应的$\mu=0,\sigma^2=\lambda_k$，因此
$$
\begin{equation}
   E[zz^T]=\sum_{i=1}^D u_iu_i^T\lambda_i = \Sigma 
\end{equation}
$$
即$E[xx^T]=\Sigma+\mu\mu^T$
因此
$$
\begin{equation}
cov[x] = E[xx^T]-E[x]E[x]^T= \Sigma
\end{equation}
$$
可见，上一节中多变量高斯分布表达式$N(x|\mu,\Sigma)$中的两个参数$\mu$和$\Sigma$即为该分布的期望和协方差。

先写到这，剩下的慢慢更~~~


<h3 id="1.2">1.2 条件高斯分布</h3>

假设 $x$ 是服从高斯分布 $N(x|\mu,\Sigma)$ 的 D 维向量，把 $x$ 划分为两个不相交的子集 $x_a,x_b$。不失一般性的，令 $x_a$ 为 $x$ 的前 M个分量，令 $x_b$ 为剩余的 D-M 个分量，得到：

$$
        \begin{pmatrix}
        x_a \\
        x_b
        \end{pmatrix}
$$
对应的均值向量$\mu$的划分：
$$
        \begin{pmatrix}
        \mu_a \\
        \mu_b
        \end{pmatrix}
$$
协方差矩阵为：
$$
        \begin{pmatrix}
        \Sigma_{aa} & \Sigma_{ab} \\
        \Sigma_{ba} & \Sigma_{bb}
        \end{pmatrix}
$$
这里我们通过取协方差的倒数来定义精度矩阵：$\Lambda=\Sigma^{-1}$，
$$
\begin{pmatrix}
\Lambda_{aa} & \Lambda_{ab} \\
\Lambda_{ba} & \Lambda_{bb}
\end{pmatrix}
$$

定义条件概率$p(x)=p(x_a,x_b)$，即在$x_b$固定的前提下求$x_a$的分布，可知其也是一个高斯分布，因此我们可通过分析其指数项的特征来获悉具体的参数表达。

---分割线-------

首先，我们来看下任意高斯分布的指数项：
$$
\begin{equation}
   -\frac{1}{2}(x-\mu)^T\Sigma^{-1}(x-\mu)=-\frac{1}{2}x^T\Sigma^{-1}x+x^T\Sigma^{-1}\mu+const(\text{indenpendent on x}) 
\end{equation}
$$
又由于
$$
\begin{equation}
    -\frac{1}{2}(x-\mu)^T\Sigma^{-1}(x-\mu)=\\
    -\frac{1}{2}(x_a-\mu_a)^T\Sigma^{-1}(x_a-\mu_a)-\frac{1}{2}(x_a-\mu_a)^T\Sigma^{-1}(x_b-\mu_b)\\
    -\frac{1}{2}(x_b-\mu_b)^T\Sigma^{-1}(x_a-\mu_a)-\frac{1}{2}(x_b-\mu_b)^T\Sigma^{-1}(x_a-\mu_a)
\end{equation}
$$



<h3 id="1.3">1.3 边缘高斯分布</h3>

<h3 id="1.4">1.4 贝叶斯定理</h3>

<h2 id="2">2. 参数估计</h2>

<h3 id="2.1">2.1 极大似然估计</h3>

<h3 id="2.2">2.2 贝叶斯估计</h3>