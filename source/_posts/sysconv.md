---
title: 信号与系统中的卷积
date: 2019-05-25
tag: [信号与系统]
categories: [信号与系统]
---


## 脉冲函数

### 离散时间(discrete-time)

<!--more-->

$$
\delta [n] = \left\{\begin{matrix}
1,  \quad  n=0\\ 
0, \quad n=1
\end{matrix}\right.
$$

![Discrete-time form of the unit impulse](/images/sysconv/impulse.jpg)

### 连续时间(continuous-time)

$$
\delta (t) = 0  \quad for \quad t\neq 0

\int_{-\infty }^{\infty } \delta(t) = 1
$$

![](/images/sysconv/impulse2.jpg)


## The Convolution Sum
> 以下仅适用于 `LTI(Linear Time Invariant)`系统。离散信号(discrete-time)输入。

令信号 $x[n]$ 乘以脉冲序列 $ \delta[n] $，即
$$ x[n]\delta[n] = x[0]\delta[n] $$

这个关系也可以概括为信号 $x[n]$ 和一个时移脉冲序列的乘积，即

$$x[n]\delta[n-k] = x[k]\delta[n-k]$$

这是因为在上一节中可以观察到，对于$ \delta[n] $，当且仅当`n`为0时值为一，其他情况为0，所以对于$ \delta[n-k] $，当且仅当`n`为k时值为一，其他情况为0。

`n`表示时间的索引，`x[n]`表示整个信号，`x[k]`表示信号 $x[n]$ 在`k`时间的值。

$$ x[n] = ... + x[-2]\delta[n+2] + x[-1]\delta[n+1] + x[0]\delta[n] 
                + x[1]\delta[n-1] + x[2]\delta[n-2] + ...
$$

即可以表示成函数权重乘积的和

$$ x[n] = \sum^{\infty}_{k=-\infty} x[k]\delta[n-k] $$

![](/images/sysconv/1.jpg)


令`H`表示以信号 $x[n]$ 为输入的系统，那么

$$ y[n] = H\{x[n]\}
        = H\{\sum^{\infty}_{k=-\infty} x[k]\delta[n-k]\}
$$

由于系统是线性(linearity)的，可以得到

$$ y[n] = \sum^{\infty}_{k=-\infty} H\{x[k]\delta[n-k]\} $$

又因为在时间k，$x[k]$为常数，所以

$$ y[n] = \sum^{\infty}_{k=-\infty}x[k] H\{\delta[n-k]\} $$

应用`LTI`系统的时间不变性，得到

$$ H\{\delta[n-k]\} = h[n-k]$$

这里$h[n] = H{\delta[n]}$ 表示`系统H`的`脉冲响应(impulse response)`，可得


$$ y[n] = \sum^{\infty}_{k=-\infty}x[k]h[n-k] $$

![](/images/sysconv/3.jpg)

> the output of an LTI system is given by a weighted sum of time-shifted impulse responses.

使用卷积符号 $ * $ 来表示，

$$ x[n]*h[n] = \sum^{\infty}_{k=-\infty}x[k]h[n-k] $$

![](/images/sysconv/conv2.jpg)



## 卷积的过程

假设，我们定义中间信号

$$ w_{n}[k] = x[k]h[n-k] $$

其实，$h[n-k] = h[-(n-k)]$可以由$h[k]$反射再向左平移动`n`个单位得到(其中n>0)。

将上式代入$ y[n] $ 可得

$$ y[n] = \sum^{\infty}_{k=-\infty}w_{n}[k] $$

便于后续推导。

### 例题

![](/images/sysconv/e1.jpg)

![](/images/sysconv/e2.jpg)

![](/images/sysconv/e3.jpg)

## The Convolution Integral
> 以下仅适用于 `LTI(Linear Time Invariant)`系统。连续信号(continuous-time)输入。

类似于上面离散信号的例子。用积分来取代求和。

$$ x(t) = \int_{-\infty }^{\infty}x(\tau)\delta(t-\tau)d\tau $$

$$ y(t) = H{x(t)}
        = H{\int_{-\infty }^{\infty}x(\tau)\delta(t-\tau)}d\tau
$$

同样利用`linearity`，得到

$$ y(t) = \int_{-\infty }^{\infty}x(\tau)H{\delta(t-\tau})d\tau $$

定义脉冲响应 $h(t) = H{\delta(t)}$

代入上式得到，
$$ y(t) = \int_{-\infty }^{\infty}x(\tau)h(t-\tau)d\tau $$

卷积积分(convolution integral)如下，

$$ x(t)*h(t) = \int_{-\infty }^{\infty}x(\tau)h(t-\tau)d\tau$$


## 反卷积(Deconvolution)

将$x(t)$从 $h(t)*x(t)$ 中恢复出来成为反卷积。

$$ x(t)*(h(t) *h^{inv}(t)) = x(t) $$

需要

$$ h(t)*h^{inv}(t) = \delta(t) $$

对于离散时间LTI可逆系统，

$$ h[t]*h^{inv}[n] = \delta[n] $$

![](/images/sysconv/inv.jpg)


## 课后练习

#### 2.33 
> Evaluate the discrete-time convolution sums given below.

a) $ y[n] = u[n + 3] * u[n − 3]$

令 $u[n + 3] = x[n]$ 且 $u[n − 3] = h[n]$

![](/images/sysconv/2.jpg)

当 $n-3 < -3$，$n<0，\quad y[n]=0$

当 $n-3 \geq -3$，$n \geq 0，\quad y[n]=\sum^{n-3}_{k=-3}1 = n+1$

即
$$ y[n] = \left\{\begin{matrix}
n+1, \quad n\geq 0\\ 
0, \quad n < 0
\end{matrix}\right.$$


#### 2.35
> At the start of the first year 10,000 is deposited in a bank account earning 5% per year. At the
start of each succeeding year 1000 is deposited. Use convolution to determine the balance at the start of each year (after the deposit). Initially 10000 is invested.

![](/images/sysconv/235_1.jpg)

![Yearly balance of the account](/images/sysconv/235_2.jpg)


#### 2.39
> Evaluate the continuous-time convolution integrals given below.

![](/images/sysconv/239_1.jpg)

![](/images/sysconv/239_2.jpg)

![](/images/sysconv/239_3.jpg)


## 参考资料

1. Signals and Systems 2nd(Simon Haykin)
2. [Lecture 4, Convolution | MIT RES.6.007 Signals and Systems, Spring 2011](https://www.youtube.com/watch?v=_vyke3vF4Nk)
