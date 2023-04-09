---
title: js-garbage-collection
date: 2023-3-24 10:30
categories: javaScript
---

# JS垃圾回收

- **可达性**，以某些方式可以访问或者可用的值，它们被保存在内存中，无需垃圾清理，反之，不可访问则需要回收
- js的垃圾回收机制，简单来说就是定期找出哪些不再用到的内存或者变量，然后将其释放
- 关于垃圾回收机制，最常用的两个算法策略为：**标记清理算法**和**引用计数算法**

## 标记清理算法

此算法是js中最常见的，算法大致过程如下：

- 垃圾收集器在运行时会给内存中的所有变量都加上一个标记，假如内存中的所有对象都是垃圾，则将它们全部标记为0
- 从根对象向上遍历，把不是垃圾的节点变为0
- 清除所有标记为0的垃圾，销毁并回收它们所占用的内存空间
- 把所有内存中的对象标记都修改为0，等待下一轮的垃圾回收

**优点：**

实现起来比较简单，只有标记与不标记两种情况，这使得一位二进制既可以为其标记，非常简单

**缺点：**

它最大的缺点就是再清楚之后，剩余对象的内存位置是不变的，这会导致空闲空间是不连续的，出现了**内存碎片**。

**解决**：

标记清理法可以有效地解决上述问题，它的标记阶段和清除阶段与标记清除算法没有什么不同，只是标记结束后，标记整理算法会将活着的对象向内存的一端移动，最后清除掉位于边界上的内存，这样就可以得到一整块的空闲内存了。

## 引用计数算法

引用计数算法是最早先用到的一种**垃圾回收算法**，它把对象是否不再需要简化定义为**对象有没有被其他对象所引用**，如果没有引用该对象，对象就将被垃圾回收机制回收。

它的算法策略时跟踪记录每个变量值被使用的次数，过程如下：

- 当声明了一个变量并且将一个引用类型赋值给该变量的时候，这个值的引用次数为1
- 如果同一个值又被赋给另一个变量，那么引用数加1
- 如果该变量的值又被其他值覆盖了，则引用次数减1
- 当这个值的引用次数为0的时候，说明没有变量再使用，这个值没法被访问。垃圾回收器会在运行时候清理掉引用次数为0的值所占用的内存

优点：

- 清晰简单很多

缺点：

- 此方法需要一个计数器，而技术器需要占一个比较大的位置
- 它无法解决循环引用问题。比如在一个函数作用域中，两个变量一直在相互引用，那么即使函数执行完毕，这两个变量也不会被清除，这样就会占据大量的无用的内存

# v8引擎对垃圾回收(GC)的优化

谷歌的v8引擎也是基于标记清理算法，但是做了相应的优化过程。垃圾清理算法在每次垃圾回收时都要检查内存中所有的变量，但是对一些比较大的，存活时间长的对象来说，它不需要频繁清理。但是对于另外一些小的，存活时间很短的，比较新的对象来说，却需要进行一个更频繁的清理。而v8的垃圾回收策略表示其余**分代式垃圾回收机制**，v8将堆内存中分为新生代和老生代两区域，并采用不同的垃圾回收策略。

## 新生代垃圾回收

新生代区域的对象都是存活时间较短的对象，v8将新生代区域一分为二，一个处于使用状态，为使用区，另一个是空闲状态，为空闲区。新加入的对象都会被放入使用去，当使用区快写满时，就执行一次垃圾清理操作。当开始进行垃圾回收时，新生代垃圾回收器会对使用区的活动对象做标记，标记后复制一份活动对象到空闲区(这里做了排序，避免出现内存随便)。然后清除使用区中的数据对象，把原来的使用区变为空闲区，再把原来的空闲区变为使用区。这样的话新的使用区就是空的，继续存放数据。当存放快要满的时候再进行下一轮垃圾回收，重复上面步骤。这个时候假如发现了上次就存在的对象这次还是活动对象，那么这个对象就会晋级，扔到老生代里面去。

## 老生代垃圾回收

老生代垃圾回收就是标记清理算法，先标记时从一组根元素开始，递归遍历这组根元素，遍历过程中能达到的元素被称为活动对象，没有达到的元素可以判断为非活动对象。清楚阶段老生代垃圾回收器会直接将非活动对象，也就是数据清理掉。
