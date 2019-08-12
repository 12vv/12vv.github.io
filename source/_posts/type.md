---
title: 弱类型、强类型、动态类型、静态类型语言的区别
date: 2019-06-10
tag: [Python]
categories: [python]
---

大部分来自[知乎](https://www.zhihu.com/question/19918532/answer/21647195)

<!--more-->

## 基础概念

### Program Errors

#### trapped errors
导致程序终止执行。如除以0，`Java`中的数组越界访问。

#### untrapped errors
出错后继续执行，但可能出现任意行为。如`C`中的缓冲区溢出、`Jump`到错误地址。

### Forbidden Behaviours
语言设计时，可以定义一组`Forbidden Behaviours`。它必须包括所有`untrapped errors`，但可能包含`trapped errors`。

### Well behaved
程序执行不可能出现`forbidden behaviors`。

### Ill behaved 
程序执行可能出现`forbidden behaviors`。


![](/images/type/2.jpg)


## 语言的类型

### 强类型(strongly typed)
如果一种语言的所有程序都是`well behaved`，即不可能出现`forbidden behaviors`，则该语言为`strongly typed`。

> 强制数据类型定义的语言。也就是说，一旦一个变量被指定了某个数据类型，如果不经过强制转换，那么它就永远是这个数据类型了。如果两个不同类型相加，会出现 $typeerror$。

### 弱类型(weekly typed)
可能出现`forbidden behaviors`即为弱类型，比如C语言的缓冲区溢出，属于`untrapped errors`，即属于`forbidden behaviors`，故C是弱类型。

> 弱类型语言，类型检查更不严格，如偏向于容忍隐式类型转换。数据类型可以被忽略，它与强类型定义语言相反, 一个变量可以赋不同数据类型的值。譬如说C语言的`int`可以变成`double`。

### 静态类型(statically typed)
> 数据类型是在编译其间检查的，写程序时要声明所有变量的数据类型
在**编译**时拒绝`ill behaved`程序，则是`statically typed`。

- 如果类型时语言语法的一部分，则是`explicitly typed`显式类型。
- 如果类型通过编译时推导，是`umolicity typed`隐式类型，比如`ML`和`Haskell`。

### 动态类型(dynamiclly typed)
> 在运行期间才去做数据类型检查的语言
在**运行**时拒绝`ill behaved`程序，则是`dynamiclly typed`。


### 解释性语言
不需要编译，在运行程序的时候才翻译，每个语句都是执行的时候才翻译。这样解释性语言每执行一次就需要逐行翻译一次，效率比较低。
现代解释性语言通常把源程序编译成中间代码，然后用解释器把中间代码一条条翻译成目标机器代码，一条条执行。

### 编译性语言
编译性语言写的程序在被执行之前，需要一个专门的编译过程，把程序编译成为机器语言的文件，比如exe文件，以后要运行的话就不用重新翻译了，直接使用编译的结果就行了`（exe文件）`，因为翻译只做了一次，运行时不需要翻译，所以编译型语言的程序执行效率高。

![](/images/type/1.jpg)