---
title: "HashMap源码解读"
date: 2020-07-18T16:47:04+08:00
draft: false
---

# HashMap源码解读

* 为什么数组的长度一定要是2的N次方
  * 只有这样，我们在对它进行-1操作的时候才能拿到全是1的值，可以非常快速的按照位运算的方式拿到数组的下标，并且桶的分布还是均匀的

java8HashMap的改进
---
* 数组+链表/红黑树
* 扩容时插入顺序的改进
* 函数方法
  * forEach
  * compute系列
* Mapde的新API
  * merge
  * replace