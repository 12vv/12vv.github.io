---
title: 算法 -- 大数相乘
date: 2019-07-26
tag: [algorithm]
categories: [算法]
---

<!--more-->

### 概述

> “Perhaps the most important principle for the good algorithm designer is to refuse to be content.”
---Alfred Aho

最近在听`coursera`上的算法课，讲到乘法的算法，也许普通的乘法也能做运算，但是复杂度为$O(n^2)$，实在不适用于大数相乘。就像上面那句引言，对于算法的设计拒绝满足，时刻要问，“Can we do better?”。


### Karatsuba Multiplication

计算$x*y$，按照下面将x拆成$a,b$两部分，将y拆成$c,d$两部分，用$n$表示数字的位数，
$$
x = x1 * 10^{n/2} + x2 \\
y = y1 * 10^{n/2} + y2
$$

比如`1234`就被拆成`12`和`34`两部分，那么，
$$
x * y = (x1 * 10^{n/2}+ x2)(y1 * 10^{n/2}+y2) \\
= x1 * y1 * 10^{n}+(x1 * y2 + x2 * y1) * 10^{n/2}+x2 * y2
$$

其中，$(x1\*y2+x2\*y1)$可以由$(x1+x2)(y1+y2)-x1\*y1-x2\*y2$取得，这样又减少了一次乘法运算。

```python
import math

def karatsuba(x, y):
    if x<10 or y<10:
        return x*y
    else:
        lenx = len(str(x))
        leny = len(str(y))
        # the longest digits
        m = int(max(lenx, leny)/2)
        # print(m)
        x1 = int(x/math.pow(10, m))
        # print(x1)
        x0 = x-int(x1*math.pow(10, m))
        # print(x0)
        y1 = int(y/math.pow(10, m))
        y0 = y-int(y1*math.pow(10, m))
        z2 = karatsuba(x1, y1)
        z0 = karatsuba(x0, y0)
        z1 = karatsuba((x1+x0), (y1+y0))-z2-z0
        return z2*int(math.pow(10, 2*m)) + z1*int(math.pow(10, m))+z0
```

以上是`python`解法，自己测试了一些用例没有问题，但是在`Coursera`上`Divide and Conquer, Sorting and Searching, and Randomized Algorithms`的`Week1`课后测试无法通过。

```python
print(karatsuba(3141592653589793238462643383279502884197169399375105820974944592, 2718281828459045235360287471352662497757247093699959574966967627))
# Output: 8539734222673566239807039410319259860852243689421558415287808071184244153281951896239103984554011517738945511404005174800327280
# The correct answer: 8539734222673567065463550869546574495034888535765114961879601127067743044893204848617875072216249073013374895871952806582723184
```
我猜想可能时`python`对太大的数取`int`会不精确。大家可以参考[这里](http://abhi5631.blogspot.com/2017/04/divide-and-conquer-1karatsuba-algorithm.html)的`C语言`解法。

根据上面$x\*y$的式子，可以分析，
$$
T(n)=3T(n/2)+O(n) \\
T(n)=O(n^{log3})=O(n^{1.584})
$$

### Toom-Cook k-Way algorithm
上面`Karatsuba`算法用了`分而治之`的思想，将每个操作数分为两部分，而这个`Toom-Cook k-Way Multiplication`则是分为`k`部分。其复杂度为$\theta(c(k)n^{log(2k-1)/log(k)})$，(由主定理得到)。

下面以`Toom 3-Way Multiplication`为例子，同样是用了`分而治之`的思想，将每个操作数分成3个部分。

```
 high                         low
+----------+----------+----------+
|    x2    |    x1    |    x0    |
+----------+----------+----------+

+----------+----------+----------+
|    y2    |    y1    |    y0    |
+----------+----------+----------+
```

$$
X(t) = x2 * t^2 + x1 * t + x0
Y(t) = y2 * t^2 + y1 * t + y0
$$

用一个多项式来表示$W(t)=X(t)*Y(t)$，

$$
W(t) = w4 * t^4 + w3 * t^3 + w2 * t^2 + w1 * t + w0
$$

即最后的结果，确定了上式的参数$w4, w3, w2, w1, w0$后，错位相加。
```
 high                                        low
+-------+-------+
|       w4      |
+-------+-------+
       +--------+-------+
       |        w3      |
       +--------+-------+
               +--------+-------+
               |        w2      |
               +--------+-------+
                       +--------+-------+
                       |        w1      |
                       +--------+-------+
                                +-------+-------+
                                |       w0      |
                                +-------+-------+
```

为计算$w$系数，让$t$取5个不同的点($0, -1, 1, 2, inf$)，根据$W(t)=X(t)*Y(t)$，得

```
Point	    Value
t=0	    x0 * y0, which gives w0 immediately
t=1	    (x2+x1+x0) * (y2+y1+y0)
t=-1	    (x2-x1+x0) * (y2-y1+y0)
t=2	    (4*x2+2*x1+x0) * (4*y2+2*y1+y0)
t=inf	    x2 * y2, which gives w4 immediately
```

又根据$W(t) = w4\*t^4 + w3\*t^3 + w2\*t^2 + w1\*t + w0$，得

```
W(0)   =                              w0
W(1)   =    w4 +   w3 +   w2 +   w1 + w0
W(-1)  =    w4 -   w3 +   w2 -   w1 + w0
W(2)   = 16*w4 + 8*w3 + 4*w2 + 2*w1 + w0
W(inf) =    w4
```

联立方程即可求得`w`系数，回代即可得解。

`Toom-3`有渐进$O(N^{1.465})$的复杂度，指数部分是`1.465`是$log(5)/log(3)$，因为一次运算拆成5次递归，每个递归的大小是原来的 1/3。




### FFT Multiplication



### 参考资料
1. [Karatsuba Multiplication](https://www.coursera.org/learn/algorithms-divide-conquer/lecture/wKEYL/karatsuba-multiplication)
2. [Divide and Conquer - 1.Karatsuba Algorithm for Multiplication](http://abhi5631.blogspot.com/2017/04/divide-and-conquer-1karatsuba-algorithm.html)
3. [Multiplication](https://gmplib.org/manual/Multiplication-Algorithms.html#Multiplication-Algorithms)

