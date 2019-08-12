---
title: Python 的垃圾回收机制
date: 2019-03-10
tag: [Python]
categories: [python]
---


### 垃圾回收(Garbage Collection)
`垃圾回收(Garbage Collection, GC)`的动态存储器不再需要时，就应该予以释放，以让出存储器，这种存储器资源管理，称为垃圾回收。

<!--more-->

### python中的GC
> python采用的是`引用计数`机制为主，`标记-清除`和`分代收集`机制为辅的策略

#### 引用计数(Reference counting)

`PyObject`结构体如下

```python
typedef struct_object {
 int ob_refcnt;
 struct_typeobject *ob_type;
} PyObject
```

`PyObject`是每个对象必有的内容，其中`ob_refcnt`就是做为引用计数。创建一个对象时，python将引用计数设置为1。当一个对象有新的引用时，它的`ob_refcnt`就会增加。
![](/images/gc/1.jpg)

现在创建一个新的`Node`实例。
![](/images/gc/2.jpg)

由于改变了 n1 的指向，所以"ABC"的引用数便置为0了，当引用它的对象被删除，它的`ob_refcnt`就会减少。

当对象的引用数减为0，`Python`将立即释放内存。


##### 引用计数加一
1. 对象被创建，如：`a=23`。
2. 对象被引用，如：`b=a`。
3. 对象被作为参数，传入一个函数中，如：`func(a)`。
4. 对象作为一个元素，存储在容器中，如：`list1=[a,a]`。

##### 引用计数减一
1. 对象的别名被显示销毁，如：`del a`。
2. 对象的别名被赋予新的对象，如：`a=24`。
3. 一个对象离开它的作用域，如f函数执行完毕时，func函数中的`局部变量`。
4. 对象所在的容器被销毁，或从容器中删除对象。


##### 优点

1. 简单
2. 实时性：一旦没有引用，内存就直接释放了。不用像其他机制等到特定时机。


##### 缺点
1. 维护引用计数消耗资源
2. 无法解决循环引用
```python
def create_cycle(): 
  
    # create a list x 
    x = [ ] 
  
    # A reference cycle is created 
    # here as x contains reference to 
    # to self. 
    x.append(x) 
   
create_cycle() 
```
上面这个函数`create_cycle()`创造了x指向它自己。所以即使函数返回了x的空间也不会被释放。


可以使用`Python`的`gc`模块进行手动垃圾回收。
```python
import gc 
i = 0

# create a cycle and on each iteration x as a dictionary 
# assigned to 1 
def create_cycle(): 
	x = { } 
	x[i+1] = x 
	print x 

# lists are cleared whenever a full collection or 
# collection of the highest generation (2) is run 
collected = gc.collect() # or gc.collect(2) 
print "Garbage collector: collected %d objects." % (collected) 

print "Creating cycles..."
for i in range(10): 
	create_cycle() 

collected = gc.collect() 

print "Garbage collector: collected %d objects." % (collected) 

# output:
# Garbage collector: collected 0 objects.
# Creating cycles...
# {1: {...}}
# {2: {...}}
# {3: {...}}
# {4: {...}}
# {5: {...}}
# {6: {...}}
# {7: {...}}
# {8: {...}}
# {9: {...}}
# {10: {...}}
# Garbage collector: collected 10 objects.
```

#### 分代收集(Generational GC)

`Python`使用一种不同的链表来持续追踪活跃的对象，Python的内部C代码将其称为`零代(Generation Zero)`。

每次创建一个对象时，`Python`将其加入零代链表。
![](/images/gc/3.jpg)

可以看到创建节点 "ABC", `Python`将其加入零代链表。这个链表并不能直接使用代码访问。

![](/images/gc/4.jpg)
创建节点 "DEF", `Python`同样将其加入零代链表。(链表中还将包含Python创建的每个其他值，与一些`Python`自己使用的内部值。)


随后，Python会循环遍历零代列表上的每个对象，检查列表中每个互相引用的对象，根据规则减掉其引用计数。


在这个过程中，Python会一个接一个的统计内部引用的数量以防过早地释放对象。

![](/images/gc/5.jpg)

由上图，"ABC" 和 "DEF" 节点包含的引用数为1.有三个其他的对象同时存在于零代链表中，蓝色的箭头指示了有一些对象正在被零代链表之外的其他对象所引用。

下图显示`Python`识别内部(循环)引用并减小他们的引用计数。

![](/images/gc/6.jpg)

通过识别内部引用，`Python` 能够减少许多零代链表对象的引用计数。在上图的第一行中你能够看见 "ABC" 和 "DEF" 的引用计数已经变为零了，这意味着收集器可以释放它们并回收内存空间了。

剩下的活跃的对象则被移动到一个新的链表：一代链表。


### python的gc模块
[Garbage Collector interface](https://docs.python.org/2/library/gc.html)
有三种情况触发垃圾回收：
1. 调用gc.collect()。
2. 当gc模块的计数器达到阈值的时候。
3. 程序退出的时候。

#### 常用函数
1. gc.set_debug(flags): 设置gc的debug日志，一般设置为gc.DEBUG_LEAK
2. gc.collect([generation]): 显式进行垃圾回收，可以输入参数，0代表只检查第一代的对象，1代表检查一，二代的对象，2代表检查一，二，三代的对象，如果不传参数，执行一个full collection，也就是等于传2。返回不可达（unreachable objects）对象的数目。
3. gc.set_threshold(threshold0[, threshold1[, threshold2]): 设置自动执行垃圾回收的频率。
4. gc.get_count(): 获取当前自动执行垃圾回收的计数器，返回一个长度为3的列表。

##### 补充
get_count()长度为3的列表，第一位代表距离上一次一代垃圾检查，python分配内存数目减去释放内存的数目。同理第二位代表距离上一次二代垃圾检查，一代垃圾检查的次数，第三位代表距离上一次三代垃圾检查，二代垃圾检查的次数。

gc模快有一个自动垃圾回收的阀值，即通过`gc.get_threshold`函数获取到的长度为3的元组，例如`(700,10,10)`

每一次计数器的增加，gc模块就会检查增加后的计数是否达到阀值的数目，如果是，就会执行对应的代数的垃圾检查，然后重置计数器。

例如，假设阀值是`(700,10,10)`：

- 当计数器从`(699,3,0)`增加到`(700,3,0)`，gc模块就会执行`gc.collect(0)`,即检查一代对象的垃圾，并重置计数器为`(0,4,0)`
- 当计数器从(699,9,0)增加到`(700,9,0)`，gc模块就会执行`gc.collect(1)`,即检查一、二代对象的垃圾，并重置计数器为`(0,0,1)`
- 当计数器从`(699,9,9)`增加到`(700,9,9)`，gc模块就会执行`gc.collect(2)`,即检查一、二、三代对象的垃圾，并重置计数器为`(0,0,0)`

### 总结
- 应避免循环引用
- 引入gc模块，启动gc模块的自动清理循环引用的对象机制
- 由于分代收集，所以把需要长期使用的变量集中管理，并尽快移到二代以后，减少GC检查时的消耗



### 参考资料
1. [python垃圾回收机制](https://my.oschina.net/hebianxizao/blog/57367?fromerr=KJozamtm)
2. [Garbage Collection in Python](https://www.geeksforgeeks.org/garbage-collection-python/)
3. [Python垃圾回收机制详解](https://www.cnblogs.com/Xjng/p/5128269.html)