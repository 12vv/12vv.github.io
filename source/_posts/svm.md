---
title: SVM理解
date: 2018-02-01
tag: [Machine Learning]
categories: [机器学习]
---
      

## 简述

在《统计学习方法》中，这样描述：

> `支持向量机（support vector machines，SVM）`是一种二类分类模型，其基本模型定义为特征空间上的间隔最大的线性分类器，其学习策略便是间隔最大化，可形式化为一个`凸二次规划（convex quadratic programming）`问题的求解。


<!--more-->

## 函数距离与几何距离

### 函数间隔(function margin)
对于给定训练数据集T和超平面$(w, b)$,定义超平面$(w,b)$关于样本点$(x_{i},y_{i})$的函数间隔为

$$\widehat{\gamma_{i}}=y_{i}(w\cdot x_{i}+b)$$


定义超平面$(w,b)$关于数据集T的几何间隔为超平面$(w,b)$关于T中所有样本点$(x_{i},y_{i})$的函数间隔之最小值，即
$$\widehat{\gamma}=min\widehat{\gamma _{i}}$$

### 几何间隔(geimetric margin)

对于给定训练数据集T和超平面$(w, b)$,定义超平面$(w,b)$关于样本点
$(x_{i},y_{i})$的几何间隔为$$\gamma_{i}=y_{i}(\frac{w}{\left \| w \right \|}\cdot x_{i}+\frac{b}{\left \| w \right \|})$$
定义超平面$(w,b)$关于数据集T的几何间隔为超平面$(w,b)$T中所有样本点$(x_{i},y_{i})$的几何间隔之最小值，即
$$\gamma=min\gamma _{i}$$
因为如果超平面参数$w$和$b$成比例改变，虽然超平面不变，但是函数间隔离变了。因此使用几何间隔，并且令$\left | w \right |=1$，下图为《机器学习》中的一张插图。
![](/images/svm/2-1.png)

## 对偶问题

得到的目标函数如下

$$max\frac{1}{\left \|w \right \|} \hspace{0.5cm} s.t., \gamma_{i}(w^{T}+b)\geq 1$$

由于求$\frac{1}{\left |w \right |}$的最大值相当于求$\frac{1}{2}\left |w \right |^{2}$的最小值，所以上面的目标函数等价于

$$min\frac{1}{2}\left \|w \right \|^{2} \hspace{0.5cm} s.t., \gamma_{i}(w^{T}+b)\geq 1$$

* * *

### 补充解释
> 为了更好地理解接下来的内容，这里插入一段有关`对偶性(duality)`的补充。详情请见[这篇文章](http://blog.pluskid.org/?p=702)，已经清楚的伙伴可以跳过。

简单来说，对于任意一个带约束的优化都可以写成这样的形式：

$$
\begin{aligned} 
\min&f_0(x) \\ 
s.t. &f_i(x)\leq 0, \quad i=1,\ldots,m\\ 
&h_i(x)=0, \quad i=1,\ldots,p 
\end{aligned}$$

虽然约束条件能够帮助我们减小搜索空间，但是如果约束条件本身就是比较复杂的形式的话，其实是一件很让人头痛的问题，为此我们希望把带约束的优化问题转化为无约束的优化问题。为此，我们定义 `Lagrangian` 如下:

$$L(x,\lambda,\nu)=f_0(x)+\sum_{i=1}^m\lambda_if_i(x)+\sum_{i=1}^p\nu_ih_i(x)$$

令：

$$z(x)=\max_{\lambda\succeq 0, \nu}L(x,\lambda,\nu)$$

容易证明，对于满足约束条件的 x，有$f_0(x)=z(x)$，因为约束条件$h_i(x)=0$，即式子最后一项为0，又因为$\lambda\geq 0$且约束条件$f_i(x)\leq 0$，因此式子的第二项最大值为0，所以L的最大值即$z(x)=f_0(x)$.
所以，带约束条件的`原问题(primal problem)`转换为不带约束条件的优化问题，即：

$$\min_x z(x)$$

也即(记为$p^*$)：

$$p^*=\min_x\ \max_{\lambda\succeq 0, \nu} L(x, \lambda, \nu)$$

因为如果原始问题有最优值，那么肯定是在满足约束条件的某个 $x$ 取得，而对于所有满足约束条件的$ x$ ，$z(x)$ 和 $f_0(x)$ 都是相等的。
这个`原问题(primal problem)`的`对偶问题(dual problem)`将$min$和$max$调换了位置(记为$d^*$)：

$$d^*=\max_{\lambda\succeq 0, \nu}\ \min_x L(x, \lambda, \nu)$$

可以证明$d^{\*}\leq p^{*}$，此这个性质叫做`弱对偶(weak duality)` ，对于所有的优化问题都成立。注意，无论原问题是什么形式，它的对偶问题总是`凸优化问题(convex optimization)`。
`强对偶(strong duality)`即$d^{*}=p^{\*}$，在SVM中满足`KTT(Karush-Kuhn-Tucker)`条件，通过求解对偶问题间接求解原始问题。

* * *

根据上面的补充，继续如下推导。
引入`拉格朗日乘子(Lagrange multiplier)`构造拉格朗日函数 ,( 其中拉格朗日乘子$\alpha=(\alpha_{1},\alpha_{2},…\alpha_{n})^{T}$ )

$$L(w, b, \alpha)=\frac{1}{2}\left \|w \right \|^{2}-\sum_{i=1}^n\alpha_{i}(\gamma_{i}(w^{T}+b)-1)$$

要求解：

$$\min_{w,b}\ \max_{\alpha_{i}\succeq 0} L(w, b, \alpha)=p^*$$

转换为对偶问题：

$$\max_{\alpha_{i}\succeq 0}\ \min_{w,b}\  L(w, b, \alpha)=d^*$$

### 对偶问题求解
> 先求$L(w, b, \alpha)$对 $w$,$b$的极小，再求对$\alpha$的极大。

求$\min_{w,b}\  L(w, b, \alpha)$ ：将拉格朗日函数$L(w, b, \alpha)$分别对$w$,$b$求偏导数并令其等于0。

$$\nabla_{w}L(w, b, \alpha)=0 \hspace{0.6cm}和 \hspace{0.6cm} \nabla_{b}L(w, b, \alpha)=0$$

得到
$$w=\sum_{i=1}^n\alpha_{i}y_{i}x_{i} \hspace{0.6cm}和 \hspace{0.6cm}\sum_{i=1}^n\alpha_{i}y_{i}=0$$

将上面两式带入拉格朗日函数L，得到：

$$\min_{w,b}\  L(w, b, \alpha)=-\frac{1}{2}\sum_{i,j=1}^n\alpha_{i}\alpha_{j}y_{i}y_{j}x_{i}^Tx_{j}+\sum_{i=1}^n\alpha_{i}$$

详细推导补充如下：

![](/images/svm/2-2.png)

![](/images/svm/2-3.png)

接下来求$\min_{w,b}\  L(w, b, \alpha)$对 $\alpha$的极大

$$max-\frac{1}{2}\sum_{i,j=1}^n\alpha_{i}\alpha_{j}y_{i}y_{j}x_{i}^Tx_{j}+\sum_{i=1}^n\alpha_{i} \\ s.t.,\ \alpha_{i}\geq 0, i=1,...,n \\ \sum_{i=1}^n\alpha_{i}y_{i}=0$$


将 $\alpha^{\*}=(\alpha_{1},\alpha_{2},…\alpha_{n})^{T}$ 求解出来之后，即可求出 $w^{\*}$ 和 $b^{*}$

$$w^{*}=\sum_{i=1}^n\alpha_{i}y_{i}x_{i}$$

$$b^{\*}=y_{i}- \sum_{i=1}^n \alpha_{i}^{*}y_{i}(x_{i} \cdot x_{j})$$

二次规划求解可以使用更加优化的`SMO(Sequential Minimal Optimization)`替代，更加高效，暂时自己还没有看懂，先放着。


### Kernels

当数据在当前维度(`or features`)不是线性可分的，我们想要进行如下转换

> for vector $\mathbf{x}\in\mathbb{R}^d$, apply transformation $\mathbf{x} \rightarrow \phi(\mathbf{x})$ where $\phi(\mathbf{x})\in\mathbb{R}^D$, usually $D \gg d$。


$$
\mathbf{x}=\begin{pmatrix}x_1\\ x_2\\ \vdots \\ x_d \end{pmatrix} \;\;\;\phi(\mathbf{x})=\begin{pmatrix}1\\ x_1\\ \vdots \\x_d \\ x_1x_2 \\ \vdots \\ x_{d-1}x_d\\ \vdots \\x_1x_2\cdots x_d \end{pmatrix}
$$

这样$\phi(x)$的维度达到了$2^d$，难以计算。

#### The Kernel Trick


##### Gradient Descent with Squared Loss

$$
\ell(\mathbf{w}) = \sum_{i=1}^n (\mathbf{w}^\top  \mathbf{x}_i-y_i)^2
$$
使用梯度下降更新得到
$$
w_{t+1} \leftarrow w_t - s(\frac{\partial \ell}{\partial \mathbf{w}})\  where: 
  \frac{\partial \ell}{\partial \mathbf{w}}=\sum_{i=1}^n2(w^Tx_i-y_i)x_i=\sum_{i=1}^n\gamma_i \mathbf{x}_i
$$

把$w$表示成所有`input vector x`的线性组合：
$$
  \mathbf{w}=\sum_{i=1}^n \alpha_i {\mathbf{x}}_i
$$
因为损失是凸函数，结果与初始点的选取无关，方便起见取$\mathbf{w}_0=\begin{pmatrix}0 \\ \vdots \\ 0\end{pmatrix}$


$$

\mathbf{w}_1=\mathbf{w}_0-s\sum_{i=1}^n2(\mathbf{w}_0^\top  \mathbf{x}_i-y_i)\mathbf{x}_i=\sum_{i=1}^n \alpha_i^0 {\mathbf{x}}_i-s\sum_{i=1}^n\gamma_i^0\mathbf{x}_i=\sum_{i=1}^n\alpha_i^1\mathbf{x}_i

(with \;\;\;  \alpha_i^1=\alpha_i^0-s\gamma_i^0)


\mathbf{w}_2=\mathbf{w}_1-s\sum_{i=1}^n2(\mathbf{w}_1^\top  \mathbf{x}_i-y_i)\mathbf{x}_i=\sum_{i=1}^n \alpha_i^1\mathbf{x}_i-s\sum_{i=1}^n\gamma_i^1\mathbf{x}_i=\sum_{i=1}^n\alpha_i^2\mathbf{x}_i

(with \;\;\;\alpha_i^2=\alpha_i^1\mathbf{x}_i-s\gamma_i^1)


\mathbf{w}_3=\mathbf{w}_2-s\sum_{i=1}^n2(\mathbf{w}_2^\top  \mathbf{x}_i-y_i)\mathbf{x}_i=\sum_{i=1}^n \alpha_i^2\mathbf{x}_i-s\sum_{i=1}^n\gamma_i^2\mathbf{x}_i=\sum_{i=1}^n\alpha_i^3\mathbf{x}_i

(with \;\;\;\alpha_i^3=\alpha_i^2-s\gamma_i^2)

... \qquad\qquad\qquad ... 


\mathbf{w}_t=\mathbf{w}_{t-1}-s\sum_{i=1}^n2(\mathbf{w}_{t-1}^\top  \mathbf{x}_i-y_i)\mathbf{x}_i=\sum_{i=1}^n \alpha_i^{t-1}\mathbf{x}_i-s\sum_{i=1}^n\gamma_i^{t-1}\mathbf{x}_i=\sum_{i=1}^n\alpha_i^t\mathbf{x}_i

(with \;\;\;\alpha_i^t=\alpha_i^{t-1}-s\gamma_i^{t-1})

$$

数学归纳法得出$\alpha$的更新规则

$$
\alpha_i^t=\alpha_i^{t-1}-s\gamma_i^{t-1}, 


\alpha_i^t=-s\sum_{r=0}^{t-1}\gamma_i^{r}.

$$

就可以把计算高维度`w`的任务转换成$x$的内积。

$$
\mathbf{w}^\top {\mathbf{x}}_j=\sum_{i=1}^n \alpha_i {\mathbf{x}}_i^\top{\mathbf{x}}_j
$$
损失函数变为:
$$
\ell(\mathbf{\alpha}) = \sum_{i=1}^n \left(\sum_{j=1}^n\alpha_j\mathbf{x}_j^\top  \mathbf{x}_i-y_i\right)^2
$$

计算内积的`trick`。
The sum of $2^d$ terms becomes the product of `d` terms. 
$$
\phi(\mathbf{x})^\top \phi(\mathbf{z})=1\cdot 1+x_1z_1+x_2z_2+\cdots +x_1x_2z_1z_2+ \cdots +x_1\cdots x_dz_1\cdots z_d=\prod_{k=1}^d(1+x_kz_k)
$$




## 补充

个人感觉SVM挺难理解的，前前后后参考了很多资料，感谢大神们的总结和指导，自己仍有不足，若有错漏欢迎指出。有引用部分，侵删。
参考如下：
1.[SVM三重境界](https://blog.csdn.net/v_july_v/article/details/7624837)
2.《统计学习方法》 李航
3.[支持向量机：Duality](http://blog.pluskid.org/?p=702)
4.《机器学习》 周志华
5. [cornellCS4780/CS5780] (http://www.cs.cornell.edu/courses/cs4780/2018fa/syllabus/index.html)

