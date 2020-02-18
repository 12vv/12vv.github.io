---
title: Ray Tracing in One Weekend 总结
date: 2019-12-08
tag: [cg, 杂]
categories: [Computer Graphics]
---

## 概述
这两天在看[Ray Tracing in One Weekend](https://raytracing.github.io/books/RayTracingInOneWeekend.html), 中间遇到了一些问题，查资料记录下来，特此总结。

<!--more-->

## 关于PPM

`PPM(Portable Pixmap Format)`

## c++ virtual function

看下面这个没有虚函数的例子。

```c++
class Animal
{
    public:
        void eat() { std::cout << "I'm eating generic food."; }
};

class Cat : public Animal
{
    public:
        void eat() { std::cout << "I'm eating a rat."; }
};


```

在`main`主函数中，

```c++
Animal *animal = new Animal;
Animal *catty = new Cat;
Cat *cat = new Cat;

animal->eat(); // Outputs: "I'm eating generic food."
catty->eat();  // outputs: "I'm eating generic food."
cat->eat();    // Outputs: "I'm eating a rat."
```

从上面例子可以看到，由于没有虚函数，第二行`catty`子类无法实现自己的函数。使用了虚函数，函数的调用就是在运行时才确定，基于一个类型选择调用哪一个函数，而不是编译时。

### 纯虚函数

```c++
class Abstract {
public:
   virtual void pure_virtual() = 0;
};
```
定义了纯虚函数的类为`抽象类`。


## 关于头文件

这里是看到作者有函数直接写在了头文件中，想起来在书上看见过`头文件声明，源文件定义`，但是不记得原因，查找资料回顾一下。

首先，预编译指令`#include`相当于宏，把文件包含内容全部复制到`#include`处。而`c++`允许多次声明，一次实现，如果编译时多个cpp文件中include了同一个头文件，就会报错---找到一个或多个重定义的符号。

另外，就算用`#pragma once`或是`#ifndef`避免重复定义，这里引用轮子哥的解释，#ifndef的意思只是说，同一个cpp里面不能被复制两遍，但是不同的cpp是分开编译的。


## gamma correction


## 全反射
`全内反射(Total Internal Reflection)`, 当光线经过两个不同折射率的介质时，部分的光线会于介质的界面被折射，其余的则被反射。但是，当入射角比临界角大时（光线远离法线），光线会停止进入另一界面，反之会全部向内面反射。


## 反射与折射

### Schlick's approximation

$$
R(\theta) = R_{0} + (1-R_{0})(1-cos\theta)^{5}
$$

$R(\theta)$ - probability of reflection when the incident ray angle is $\theta$
$R_{0}$ - probability of reflection on normal incidence, $\theta=0$

$$
R_{0} = (\frac{n_1-n_2}{n_1+n_2})^{2}
$$