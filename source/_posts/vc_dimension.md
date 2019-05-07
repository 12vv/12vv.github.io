---
title: VC-Dimension
date: 2018-01-02
---

> 在看林轩田老师的《机器学习基石》视频的时候对`VC-dimension`那一节不是很理解，又看到了youtube上一个有关的视频，记录总结下来。

### 引子
`VC dimension`全称是`Vapnik–Chervonenkis dimension`。

<!--more-->

令 (其中`m`表示训练集规模，并假设数据是`独立同分布`的)$$TrainError=\frac{1}{m} \sum_{i}\delta(c^{i}\neq \hat{c}(x^{i};\theta))$$
$$TestError=E[\delta(c\neq \hat{c}(x;\theta))]$$

容易知道，如果训练模型太复杂，产生`过拟合(overfitting)`，那么`TrainError`很小，`TestError`却会很大。但是如果模型不够复杂，训练效果可能不会太好。

如果用`H`来表示这个模型的`VC-dimension`，这个理论告诉我们：$$TestError\leq TrainError+\sqrt\frac{Hlog(2m/H)+H-log(\eta/4)}{m}$$

它表明，如果`m（即训练集规模很大）`，或`H（即VC-dimension）`很小的话，`TestError` 和 `TrainError` 就会比较接近。这意味着通过训练可以`‘举一反三’`！即通过训练是能够做出很好的预测的。

### shattering
如果对于点$x^{(1)}, x^{(2)}, ...,x^{(h)}$，一个分类器`f(x)`对于所有$y^{(1)}, y^{(2)}, ...,y^{(h)}$可能情况，这个分类器都能成功分离$(x^{(1)},y^{(1)}),(x^{(2)},y^{(2)}),...,(x^{(h)},y^{(h)})$，那么称`f(x)`可以`shatter`这些点。换句话说，不管赋予这些点什么标签，这个分类器都有办法可以按照这个标签进行分类。

举个例子：
给定下面这两个点如下分布：
![两个点的分布](/images/VC/1.png)


下面是所有可能情况标签，现在如果这个分类器是$f(x;\theta)=sign(\theta_{0}+\theta_{1}x_{1}+\theta_{2}x_{2})$，一个线性分类器，那么
![所有可能情况](/images/VC/2.png)


可以观察到，这所有的可能情况这个分类器(给定不同的$\theta$时)都有办法根据标签来分离这些点。所以说这个这个分类器是可以`shatter`这些点的。

现在用另一个分类器$f(x;\theta)=sign(x^{T}x-\theta)$，看看它能不能`shatter`这两点：
![](/images/VC/3.png)

可以观察到对于第一种情况不管给这个函数怎么样的 $\theta$ 值，都没有办法按这个标签分类，因为这个函数规定圆内为正圆外为负。


### 定义
到这里，引入`VC-dimension`即`H`定义，它表示能被分类器`shatter`的点的个数的最大值。

比如上面这个例子的`VC-dimension`就是1，因为当有两个点的时候就有一种情况让它没法`shatter`了，如上图所示。当只有一个点的时候是可以`shatter`的，如下图。
![](/images/VC/4.png)

对于`线性分类器(linear classification)`即`感知机(perceptron)`对于`D维`分类器,`VC-dimension`为`d+1`，d同时也是这个函数参数个数，比如上上个例子中分类器$f(x;\theta)=sign(\theta_{0}+\theta_{1}x_{1}+\theta_{2}x_{2})$ ，它的`VC-dimension`是3（即d+1=2+1=3）。

### 使用
在上面`引子`中已经提到，模型过于简单或复杂效果都不太好，那么如何选择模型的复杂度呢？怎么权衡呢？

我们知道，当增加模型参数，增加了模型的复杂度，拟合训练集的效果会提升，对于训练集错误率降低，但是对于测试集错误率是先下降再升高的，如下图所示。

![](/images/VC/5.png)


我们的目的是找到一个分类器能够在预测上表现做好，而不是在训练集上。因此可以找`VC-dimension`来帮忙。

![](/images/VC/6.png)

如上图所示，结合`引子`中的公式，TrainError(公式右边的第一项)下降随参数增多而下降，第二项`VC-dimension`变大会增多，所以一种模型的选择方法就是选他们和最小的时候。与之类似，为了防止过拟合，很多模型会加上`惩罚项(penalty)`。


具体证明过程可以找论文看，上面只是简单介绍。


### Resampling methods

把数据集分成两部份，`训练集(training set)`和`验证集(validation set or hold out set)`, 数量上各占一半(随意抽取),每次的验证集不同，算出来的MSE变化会很大。



#### Cross-Validation


##### Leave-One-Out Cross-Validation(LOOCV)
> 留一验证

每次抽取一个样本作为验证集，剩下的`n-1`个样本用于训练，一共n个样本，每个样本都当一次验证集，一共进行n次，算出来的MSE取平均。

$$
CV_{n} = \frac{1}{n}\sum_{i=1}^{n}MSE_{i}
$$
缺点是当n很大的时候计算量很大。

##### k-Fold Cross-Validation
把数据集分成k个部分，每个部分的样本个数为`$\frac{n}{k}$`，每一个部分用于做验证集一次，每一次剩余的`k-1`个部分用于训练，总共进行`k`次。

> 这样就解决留一验证的计算量大的问题，当`k=n`时则等同与留一验证。


##### Bias-Variance Trade-Off for k-Fold Cross-Validation

> k-fold CV 往往比 LOOCV 对`test error`有更好的估计！！！

###### bias
由于`k-fold CV`每一次的训练集样本个数为`$(k-1)n/k$`比`LOOCV`中每一次训练集样本个数`k-1`要少，所以`k-fold CV`的bias是比较高的。

###### variance
但是，每一次的训练样本，`LOOCV`的重叠度，或者说样本间的相关度比较高，会造成`LOOCV`的variance比较高。
> the test error estimate resulting from LOOCV tends to have higher variance than does the test error estimate resulting from k-fold CV

一般来说选用`k-fold CV`取`k=5`或`k=10`是比较好的选择。



