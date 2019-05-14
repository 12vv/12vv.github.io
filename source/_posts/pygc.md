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


### 参考资料
1. [python垃圾回收机制](https://my.oschina.net/hebianxizao/blog/57367?fromerr=KJozamtm)
2. [Garbage Collection in Python](https://www.geeksforgeeks.org/garbage-collection-python/)