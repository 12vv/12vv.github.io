---
title: 一些常见的分布
date: 2018-02-05
tag: [Math]
---
       

## 分布

一些不同的分布的总结。
<!--more-->

### 正态分布

![](/images/dis/3.png)

(图摘自wikipedia)

### Beta分布

不同于正态分布，Beta分布的取值范围在0到1之间，所以可以作为概率的分布。

#### 密度函数

$$P(\mu|\alpha,\beta)=\frac{1}{B(\alpha,\beta)} \mu^{\alpha-1} (1-\mu)^{\beta-1}  \propto  \mu^{\alpha-1} (1-\mu)^{\beta-1}$$ 

在这里$B(\alpha,\beta) = \frac{\Gamma(\alpha)\Gamma(\beta)}{\Gamma(\alpha+\beta)}$ 并且 $\Gamma(\cdot)$ 是`Gamma函数` ：$\Gamma(x)=\int_0^\infty t^{x-1} e^{-t}dt$。

看起来有点像`二项分布`：$ p(x=\theta|n,\mu) = C_n^\theta \mu^{\theta} (1-\mu)^{1-\theta} $ ，`二项分布即n重伯努利实验`，需要注意的是这里 $\mu$ 的取值只能是0或1，而在`Beta分布`中可以取0和1之间的数(包括0和1)。

![](/images/dis/1.png)

这张图表明了改变两个参数 $\alpha$ 或 $\beta$ 的值，或同时改变，函数的形态会很不一样。

#### 均值和方差
$$E[\mu] = \frac{\alpha}{\alpha+\beta} \qquad \qquad  var(\mu) = \frac{\alpha\beta}{(\alpha+\beta)^2(\alpha+\beta+1)}$$

由均值可以看出，如果参数$\alpha$ 或 $\beta$ 的值`相等`，那么均值为 1/2，密度函数图像的对称轴为 1/2 。

> 补充：`共轭先验(Conjugate Prior)`
> 在贝叶斯统计理论中，如果某个随机变量 $\theta$ 的后验概率 $p(θ|x)$和先验概率 $p(θ)$ 属于同一个分布簇的，那么称 $p(θ|x)$ 和 $p(θ)$ 为共轭分布，同时，也称 $p(θ)$ 为似然函数 $p(x|θ)$ 的共轭先验。
> 换句话说，如果先验概率满足Beta分布，那么后验概率也满足Beta分布！

在贝叶斯统计中，Beta分布作为

### 分布

如果说Beta分布可以看做`一维`的概率分布，那么可以将`Dirichlet分布`理解成`K维`概率分布

#### 密度函数
$$p(\boldsymbol{\mu}|\boldsymbol{\alpha}) = Dir(\boldsymbol{\mu}|\boldsymbol{\alpha}) = \frac{\Gamma(\hat\alpha)}{\Gamma(\alpha_1) \cdots \Gamma(\alpha_d)} \prod_{i=1}^d \mu_i^{\alpha_i-1}$$

### 总结

（摘自Youtube）这个比喻便于理解
![](/images/dis/2.png)