---
title: 降维打击之LLE算法
date: 2017-09-29 21:55:34
tags: 
    - 机器学习
    - 降维算法
    - 随笔
mathjax: true
---

## 背景
---
　　最近在温习之前了解过的spectral clustering 的时候接触到一个看着bigger满满的术语：manifold learing，然后顺藤摸瓜找到一篇2000年的science，不过他们当时并没有大肆炒这个manifold learning的概念，而是提出了一种叫做“local linear embedding”的降维算法，在处理所谓的流形的降维的时候，效果比PCA要好很多。于是我了解了下这个“简约而不简单”的LLE算法的具体实现方法，写一下我目前的理解。
　　
   ![Swiss roll](http://instudio.mabangapp.com/img/037/WJ058/WJ058g08.jpg)
　　
　　[图片来源](http://instudio.mabangapp.com/img/037/WJ058/WJ058g08.jpg)
　　首先，所谓流形，我脑海里最直观的印象就是上图这种Swiss roll,我吃的时候喜欢把它整个摊开成一张饼再吃，其实这个过程就实现了对瑞士卷的降维操作，即从三维降到了两维。降维前，我们看到相邻的卷层之间看着距离很近，但其实摊开成饼状后才发现其实距离很远，所以如果不进行降维操作，如果根据近邻原则去判断相似性其实是不准确的。


## 原理
---
　　LLE的原理其实是这样的：

- 所谓局部线性，就是认为在整个数据集的某小范围内，数据是线性的，就比如虽然地球是圆的，但我们还是可以认为我们的篮球场是个平面；而这个“小范围”，最直接的办法就是k-近邻原则。这个“近邻”的判断也可以是不同的依据：比如欧氏距离，测地距离等。

- 既然是线性，那么对每个数据点$x_i$(D维数据，即$D\times 1$的列向量)，可以用其k近邻的数据点的线性组合来表示，即$x_i = \sum_{j=1}^{k}w_{ji}x_{ji}$。其中，$w_i$是$k\times 1$的列向量，$w_{ji}$是$w_i$的第$j$行,$x_{ji}$是$x_i$的第j个近邻点。


- 通过使loss function最小化： 
$$ \arg\min \limits_{W}\sum_{i=1}^{N}(\|x_i-\sum_{j=1}^{k}w_{ji}x_{ji}\|_2^2)$$
得到权重系数$w = [w_1, w_2, ..., w_N]$,一共N个数据点，对应$N$列$w_i$。

- 接下来是LLE中又一个假设：即认为将原始数据从D维降到d维后，$x_i(D\times 1) \rightarrow y_i(d\times 1)$,其依旧可以表示为其k近邻的线性组合，且组合系数不变，即$w_i$,再一次通过最小化loss function:

$$ \arg\min \limits_{Y}\sum_{i=1}^{N}(\|y_i-\sum_{j=1}^{k}w_{ji}y_{ji}\|_2^2)$$

得到降维后的数据。



## 算法推导
---
### step 1：
---
运用k近邻算法得到每个数据的k近邻点：


$$N_i = KNN(x_i,k), N_i = [x_{1i}, ..., x_{ki}]$$


### step 2: 求解权重系数矩阵
---
即求解：


$$ \arg\min \limits_{W}\sum_{i=1}^{N} \|x_i - \sum_{j=1}^{k}w_{ji}x_{ji}\|^2, s.t. \sum_{j=1}^k$$


在推导之前，我们首先统一下数据的矩阵表达形式

　　输入：$X = [x_1, x_2, ..., x_N], D\times N$

　　权重：$w = [w_1, w_2, ..., w_N], k\times N$

　　


$$\Phi(w)=\sum_{i=1}^{N} \|x_i - \sum_{j=1}^{k}w_{ji}x_{ji}\|^2$$
$$=\sum_{i=1}^{N} \|\sum_{j=1}^k (x_i-x_{ji})w_{ji}\|^2$$
$$=\sum_{i=1}^N \|(X_i-N_i)w_i\|^2, X_i =\underbrace {[x_i, ..., x_i]}_k, N_i = [x_{1i}, ..., x_{ki}]$$
$$= \sum_{i=1}^N w_i^T(X_i-N_i)^T(X_i-N_i)w_i$$
$let\ S_i = (X_i-N_i)^T(X_i-N_i)$ being the covariance matrix, then:
$$\Phi(w)=\sum_{i=1}^N w_i^T S_i w_i$$
拉格朗日乘子法：
$$L(w_i) = w_i^T S_i w_i + \lambda(w_i^T-1)$$
求导：
$$\frac{\partial L(w_i)}{\partial w_i} = 2S_iw_i+\lambda I = 0$$
$$w_i = \frac{S_i^{-1}I}{I^T S_i^{-1}I}$$
　　
　　就上述表达式而言，局部协方差矩阵$S_i$是个$k\times k$的矩阵，其分母其实是$S_i$的逆矩阵的所有元素之和，而其分子是$S_i$的逆矩阵对行求和后得到的列向量。
 
 <!-- more -->

### step 3: 映射到低维空间($d<D$)
---

即求解：
$$\arg \min \limits_{Y} \Psi(Y) = \sum_{i=1}^N\|y_i - \sum_{j=1}^k w_{ji}y_{ji}\|^2, s.t. \sum_{i=1}^N y_i = 0, \sum_{i=1}^N y_i y_i^T = NI_{d\times d}$$

　　

　　低维空间向量：$Y = [y_1, y_2, ..., y_N], d\times N$


首先，用一个$N\times N$的稀疏矩阵(sparse matrix)$W$来表示$w$:

　　　　　对$i$的近邻点j：   $W_{ji} = w_{ji}$;

　　　　　否则:   $W_{ji} = 0$

因此：
$$\sum_{j=1}^k w_{ji}y_{ji} = YW_i$$
$W_i$是$W$方阵的第$i$列。所以
$$\Psi(Y) = \sum_{i=1}^N\|Y(I_i-W_i)\|^2$$
由矩阵论可知：对矩阵$A = [a_1, a_2, ..., a_N]$
$$\sum_i (a_i)^2 = \sum_i a_i^T a_i = tr(AA^T)$$
所以：
$$\Psi(Y) = tr(Y(I_i-W_i)(I_i-W_i)^TY^T)$$
$$= tr(YMY^T),    M = (I_i-W_i)(I_i-W_i)^T$$
再一次拉格朗日乘子法：
$$L(Y) = YMY^T+\lambda (YY^T-NI)$$
求导：
$$\frac{\partial L}{\partial Y} = 2MY^T+2\lambda Y^T = 0$$
即
$$MY^T = \lambda'Y^T$$

　　可见Y其实是M的特征向量构成的矩阵，为了将数据降到d维，我们只需要取M的最小的d个非零特征值对应的特征向量，而一般第一个最小的特征值接近0，我们将其舍弃，取前$[2， d+1]$个特征值对应的特征向量。

## 思考
---
1. 在step 3中为什么是取d个最小的非零特征值，而不是最大的d个特征值呢？
2. 怎样理解每一步的约束条件呢？


输公式好累呀~~~~
