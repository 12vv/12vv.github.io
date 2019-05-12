---
title: 自动调参---Automated Hyperparameter Tuning
date: 2019-05-10
tag: [Machine Learning]
categories: [机器学习]
---


## 超参数(Hyperparameter)
在机器学习中，超参数是在开始学习过程之前设置值的参数，例如`学习率(learning rate)` , `决策树最大深度`等等，这些超参数不是通过训练取得。

<!--more-->

## 超参数优化算法
超参数的优化可以表示成：
$$
x^{*} = arg \underset{x\epsilon \chi }{min}f(x)
$$

其中，`f(x)`代表在验证集要优化(最小化)的目标，可以是`RMSE`或错误率等等，`$x^{*}$`是使目标最小化的参数集，x又属于域`$\chi$`.

> 超参数调优遇到的问题？

当优化目标函数时，代价太高，因为每次一改动参数，就要重新在整个训练集上训练模型，然后在验证集评估。(当参数很多或模型较复杂就尤其困难)


### 网格搜索(Grid Search)
搜索整个`超参数空间(Domain)`, 及所有参数的所有取值。可以想像其代价之高。

### 随机搜索(Random search)
在整个`超参数空间(Domain)中`，随机选取训练。

### 贝叶斯优化(Bayesian optimization)
不同于`网格搜索`和`随机搜索`, 贝叶斯优化会根据之前的结果来进行参数的优化，学习一个概率模型，比目标函数容易优化，步骤可以归纳为以下：
1. 为目标函数建立一个概率模型
2. 找到超参数在这个概率模型上表现最好
3. 将这些超参数运用到真正的目标函数上
4. 根据结果更新概率模型
5. 重复2-4步

因为每次更新用了之前的知识，所以通常使用贝叶斯优化会比`网格搜索`和`随机搜索`用更少的迭代就得到较好的结果。


***

下面主要讨论贝叶斯优化(Bayesian optimization)，内容主要来自，[这篇文章](https://towardsdatascience.com/automated-machine-learning-hyperparameter-tuning-in-python-dfda59b72f8a)

## Parts of Bayesian Optimization

1. 目标函数(Objective Function)：输入超参数
2. Domain space: 输入的超参数的范围
3. 优化算法(Optimization algorithm): 建立概率模型并选取下一组超参数的算法
 
> 下面几张图帮助理解`Bayesian optimization`是如何进行优化的

![](/images/auto/p1.jpg)

紫色的区域表示`variance`, 优化尽量选择`variance`大的地方，因为想要搜到更多可能的优化的组合，但是同时也想找到目标函数的最优解(这里假设要找极大值，但贝叶斯优化中通常找极小)，Acquisition Functions (绿色的线) 用来做两者的权衡的。告知下一次选取的点在`Acquisition Functions`最大点的地方，就是红色那里。

![](/images/auto/p2.jpg)

这样就多sample了一些点，又得到了新的知识，图像更新，发现在左边的点和点`x3`间的`variance`变得很小了，所以下一个点的选取就肯定不会在那里。

就这样一直进行下去，不需要所有的点就能够找到目标函数的最优解。

### Domain
举例，要优化随机森林的参数：
对于`网格搜索`和`随机搜索`，`domain`看作是`grid`。

```
hyperparameter_grid = {
    'n_estimators': [100, 200, 300, 400, 500, 600],
    'max_depth': [2, 5, 10, 15, 20, 25, 30, 35, 40],
    'min_samples_leaf': [1, 2, 3, 4, 5, 6, 7, 8]
}
```
对于基于模型的方法，如贝叶斯优化，domain看作是又概率分布组成。如下图![](/images/auto/2.png)


### 目标函数
目标函数接收超参数为输入，输出一个分数(score)，这就是我们要最小化的目标。一个标准的目标函数模型如下：
```
def objective(hyperparameters):
    """Returns validation score from hyperparameters"""
    
    model = Classifier(hyperparameters)
    validation_loss = cross_validation(model, training_data)    
    return validation_loss
```
> 在 python 的开源库 Hyperopt 中，将目标函数视作`'黑盒子'(black box)`，不在乎内部的损失函数。

举随机森林为例，我们想要最小化`RMSE(Root Mean Squared Error)`，可以定义目标函数如下。

```
import numpy as np
from sklearn.ensemble improt RandomForestRegressor

# Function mapping hyperparameters to a real-valued scpre
def objective(hyperparameters):
    
    # Machine learning model
    rf = RandomForestRegressor(**hyperparameters)
    
    # Training 
    rf.fit(X_train, y_train)
    
    # Making predictions and evaluating
    predictions = rf.predict(X_valid)
    rmse = np.sqrt(np.mean(np.square(prediction - y_valid)))
    
    return rmse
```

### 概率模型(Probability Model)
有很多中选择，比如`GP(Gaussian Processes)`或`Random Forest regression`等。

### Acquisition Functions
这是我们基于过去的数据选择新一轮的超参数的标准，在贝叶斯优化中通常选择` Expected Improvement`作为`Acquisition Functions`。

$$
EI_{y^{*}}(x) = \int_{ - \infty}^{y^{*}}(y^{*}-y)p(y|x)dy
$$

#### Tree-structured Parzen Estimator (TPE)

根据`Bayes rule`

$$
P(y \mid x) = \frac{P(x\mid y) P(y)}{x}
$$

其中，`$p(x|y) $`是给定目标函数的参数，取这些超参数的概率

$$
p(x|y) = \left\{\begin{matrix}
l(x) , \quad if \  y<y^{*}
\\g(x) , \quad if \  y\geqslant y^{*}
\end{matrix}\right.
$$

分成两段建立不同的概率分布，例如评估随机森林的`n_estimators`超参数，
![](/images/auto/3.png)

![](/images/auto/4.png)

结合贝叶斯公式与`(EI) Expected Improvement`，得到

$$
EI_{y^{*}}(x) = \frac{\gamma y*l(x) - l(x)\int_{ - \infty}^{y^{*}}p(y)dy }{\gamma l(x) + (1-\gamma)g(x)} \ \propto  \ (\gamma + \frac{g(x)}{l(x)}(1-\gamma))^{-1}
$$

### 局限
贝叶斯优化(Bayesian optimization) 是两个过程的切换和权衡。
- exploration：可以理解为探索或搜索，尝试新的超参数取值
- exploitation: 选取尽可能减小目标函数返回值的参数值

所以，如果达到了目标函数局部最优(local minimum)，搜索可能会定位在局部最优解的周围。

另外，贝叶斯优化(Bayesian optimization)的表现也和数据集的大小有关。


#### 参考资料
1. [A Conceptual Explanation of Bayesian Hyperparameter Optimization for Machine Learning](https://towardsdatascience.com/a-conceptual-explanation-of-bayesian-model-based-hyperparameter-optimization-for-machine-learning-b8172278050f)
2. [Bayesian optimization and multi-armed bandits](https://www.youtube.com/watch?v=vz3D36VXefI)