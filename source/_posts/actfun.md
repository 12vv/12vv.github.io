---
title: 激活函数 --- Activation function
date: 2019-05-24
tag: [Deep Learning]
categories: [机器学习]
---


## 什么是激活函数
在计算网络中， 一个节点的激活函数定义了该节点在给定的输入或输入的集合下的输出。标准的计算机芯片电路可以看作是根据输入得到`开(1)`或`关(0)`输出的数字电路激活函数。这与神经网络中的线性感知机的行为类似。然而，只有非线性激活函数才允许这种网络仅使用少量节点来计算非平凡问题。 在人工神经网络中，这个功能也被称为传递函数。

<!--more-->

## 为什么需要激活函数

激活函数在神经网络中起非常大的作用，如果不用激活函数，每一层输出都是上层输入的线性函数，无论神经网络有多少层，输出都是输入的线性组合，激活函数给神经元引入了非线性因素，使得神经网络可以任意逼近任何非线性函数，这样神经网络就可以应用到众多的非线性模型中。

如图，这是一个简单的浅神经网络。
![神经网络](/images/actfun/1.jpg)

如果不考虑非线性激活函数
$$
z^{[1]} = W^{[1]}X + b^{[1]}
\\
a^{[1]} = z^{[1]}
$$

- $z^{[L]} $是每一层的输出
- $W^{[1]} $是每一层的神经元的权重
- X 是输入向量
- $b^{[1]} $是每一层的神经元的偏置

>以上都经过向量化处理

那么， 

$$
z^{[2]} = W^{[2]}a^{[1]} + b^{[2]}
\\
a^{[2]} = z^{[2]}
$$

代入，得到
$$
z^{[2]} = (W^{[2]} * [ W^{[1]}X + b^{[1]}]) + b^{[2]}
\\
        = [ W^{[2]} * W^{[1]}] * X + [ W^{[2]}*b^{[1]} + b^{[2]}]
$$

令, 
$$
[ W^{[2]} * W^{[1]}] = W
\\
[[ W^{[2]}*b^{[1]} + b^{[2]}] = b
$$
得到

$$ z(2) = W*X + b $$

仍然是线性函数！

> 一般在输出层可能会使用线性的激活函数。

## 激活函数的性质

激活函数通常都有这些性质：
- 非线性：当激活函数是线性的时候，无法拟合复杂的情况。
- 可微性：当优化方法是基于梯度的时候，这个性质是必须的。
- 单调性：当激活函数是单调的时候，单层网络能够保证是凸函数。

## 常见的激活函数

### 1.Sigmoid

$$ \sigma(x; a) = \frac{1}{1 + \mathrm{e}^{-ax}} $$


![sigmoid and its gradient](/images/actfun/sigmoid1.jpg)

#### 优点

- 此函数范围为 [0, 1]，因此很适合二分类问题，设定一个阈值(如0.5)，那么大于这个阈值划为正类，反之分为负类。也可直接输出概率。

- 便于求导，在向前传播的运算中较为方便。


#### 缺点

从图上可以发现，激活函数接近饱和区时，变化太缓慢，导数接近0，即容易出现梯度消失(gradient  vanishing)，神经网络在学习的时候，在反向传播更新参数的过程中，需要计算梯度，据是微积分求导的链式法则，当前导数需要之前各层导数的乘积，几个比较小的数相乘，导数结果很接近0，从而无法完成深层网络的训练。甚至如果初始值没有选取好就直接饱和了，导致网络变的难以学习。

另外，Sigmoid的输出不是0均值(zero-centered)的，这会导致后层的神经元的输入是非0均值的信号，这会对梯度产生影响。以 f=sigmoid(wx+b)为例， 假设输入均为正数（或负数），那么对w的导数总是正数（或负数），这样在反向传播过程中要么都往正方向更新，要么都往负方向更新，导致有一种捆绑效果，使得收敛缓慢。


> 为什么`非zero-centered`会产生问题？

根据反向传播权重的更新法则，
$$
\frac{\partial L}{\partial w_i} = \frac{\partial L}{\partial f}\frac{\partial f}{\partial w_i} = x_i \cdot \frac{\partial L}{\partial f}
\\
w_i \gets w_i - \eta x_i\cdot \frac{\partial L}{\partial f}
$$

可以看出，其更新方向又当层的输入$ x^{(i)} $决定！除了第0层，隐藏层的输入均为前一层线性处理后再激活，当激活函数为`sigmoid`，处理过后送给下一层神经元的输入均为同号，导致权重的更新方向都相同，可能会产生如下(ZIG ZAG)情形。

![](/images/actfun/zig.jpg)


### 2.Tanh(hyperbolic tangent)

$$ 
\tanh(x) = 2\sigma(2x) - 1 = \frac{\mathrm{e}^{x} - \mathrm{e}^{-x}}{\mathrm{e}^{x} + \mathrm{e}^{-x}} 
$$

![tanh and its gradient](/images/actfun/tanh1.jpg)

#### 优点

`tanh函数`将输入值压缩到 [-1, 1] 的范围，因此它是0均值(zero-centered)的，解决了`Sigmoid函数`的`非zero-centered`问题。

#### 缺点

同`Sigmoid函数`，仍然存在梯度消失问题。

### 3.ReLU(Rectified Linear Unit)

$$ f(x) = max(0, x) $$

#### 优点

- 梯度不会饱和，解决了梯度消失问题。
- 计算复杂度低，不需要进行指数运算。
- ReLU会使很多的隐藏层输出为0，即网络变得稀疏，起到了类似L1的正则化作用，可以在一定程度上缓解过拟合。

#### 缺点

- ReLU的输出不是zero-centered；
- ReLU不会对数据做幅度压缩，所以数据的幅度会随着模型层数的增加不断扩张。
- Dead ReLU Problem：某些神经元可能永远不会被激活，导致相应参数永远不会被更新(在负数部分，梯度为0)。


#### Leaky ReLU

$$ f(x) = max(ax, x) $$

![](/images/actfun/leaky.jpeg)

负数部分梯度也不为0，可以解决 `Dead ReLU Problem`。

### 4.Softmax

$$
a_j^L = \frac{e^{z_j^L}}{\sum_k e^{z_k^L}}
$$

通常用于多分类。


### 5. Maxout
$$
h_i(x)=\max_{j\in[1,k]}z_{ij}
\\
z_{ij}=x^TW_{...ij}+b_{ij}
$$

#### 优点
- Maxout的拟合能力非常强，可以拟合任意的凸函数。
- 具有`ReLU`的所有优点，线性、不饱和性。
- 解决`Dead ReLU Problem`。

#### 缺点

每个神经元中有两组(w,b)参数，那么参数量就增加了一倍，导致整体参数的数量激增。

![Maxout & ReLU](/images/actfun/maxall.jpg)


## 补充

![Activation Function Cheetsheet](/images/actfun/all.png)


## 总结
现今，通常使用`ReLU`来作为隐藏层的激活函数，如果存在`Dead ReLU Problem`则使用`Leaky ReLU`或 Maxout, 由于`Sigmoid`和`Tanh`的梯度消失问题，一般不使用这两种激活函数。

## 参考资料
1. [Activation functions in Neural Networks](https://www.geeksforgeeks.org/activation-functions-neural-networks/)
2. [激活函数](https://baike.baidu.com/item/%E6%BF%80%E6%B4%BB%E5%87%BD%E6%95%B0/2520792?fr=aladdin)
3. [深度学习中激活函数的优缺点](https://blog.csdn.net/NOT_GUY/article/details/78749509)
4. [sigmoid和tanh](https://www.zhihu.com/question/50396271)
5. [Activation Function: ReLU & Maxout](https://medium.com/%E5%AD%B8%E4%BB%A5%E5%BB%A3%E6%89%8D/activation-function-relu-maxout-f958f066fbfa)