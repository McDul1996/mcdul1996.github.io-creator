---
title: "关于我在Collection体系中学到的知识"
date: 2020-06-06T16:15:12+08:00
draft: false
---

Collection体系的常用类及其背后的数据结构
---
* List
1. ArrayList：
背后是一个动态数组，有序插入元素，可以根据下标取得对应元素，检索元素为线性查找，效率较低。当容量不足时会自动扩容，每次新建一个新的List，大小为原来的1.5倍，再将所有元素拷贝到新的List中去。

2. LinkedList：
背后是一个链表结构，有序插入元素，可以根据下标取得对应元素， 检索时从头部或者尾部不断向下标靠拢，所以头部和尾部的查找效率高，中间低。
* Set
1. HashSet：
计算出元素的哈希值，哈希值相同的元素放在同一个哈希桶里，每个哈希桶里维护一个链表。插入的元素是无序的且不允许出现重复元素，查找效率高。但当发生哈希碰撞时，整个结构退化为链表会导致性能急剧下降。
2. LinkedHashSet：
不同于HashSet，前面的数据结构为链表，所以是有序的，其他几乎一样。
3. TreeSet：
前面的数据结构为二叉树，可以对插入的元素进行排序。检索快，时间复杂度降为对数级。
* Map
1. HashMap：
实质上和HashSet一样，不过可以存储键（Key）到值（Value）的映射。JDK1.8之后当链表节点大于7时会裂变为红黑树。
2. TreeMap：
可以排序的HashMap。



ArrayList源码分析
---
---
---

* ArrayList是最常用的一种集合类型

```
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
```
表示ArrayList继承自AbstractList,实现了List接口、RandomAccess接口、Cloneable接口、Serializable接口。 说明它具备以下特点：
* 浅拷贝
* 序列化
---

常量
---
```
private static final int DEFAULT_CAPACITY = 10;
```
* DEFAULT_CAPACITY 表示默认的容量是10个元素
---

成员变量
---
```
transient Object[] elementData;
private int size;
```
* elementData为实际存储元素的数组，这里可以看到ArrayList的底层实际是一个数组。因此它具有数组的特点，随机读写快，插入删除慢。 transient关键字说明底层数组不能被序列化。
---
构造函数
---
```
public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }
```
* 无参构造函数，构造一个空的数组。
```
public ArrayList(int initialCapacity)
```
* 指定容量构造，构造一个指定初始容量大小的数组。
```
public ArrayList(Collection<? extends E> c) 
```
* 通过另外一个集合来构造。
---
添加元素方法
---

追加:
```
public boolean add(E e) 
```
add方法是线程不安全的，因为size++操作并非原子操作。因此也说明ArrayList类是线程不安全的类。 ensureCapacityInternal()方法保证数组容量可以容纳新增的这个元素，不会出现数组越界的情况。

插入:
```
public void add(int index, E element) 
```
指定index位置插入元素，其逻辑如下：

* 校验index是否合法，否则抛出数组越界异常
* 将index+1及以后的元素往前移一个位置
* 将最后一个元素置为null，同时size减一
* 返回删除的元素值。
---
删除元素
---
```
 public boolean remove(Object o) {
        if (o == null) {
            for (int index = 0; index < size; index++)
                if (elementData[index] == null) {
                    fastRemove(index);
                    return true;
                }
        } else {
            for (int index = 0; index < size; index++)
                if (o.equals(elementData[index])) {
                    fastRemove(index);
                    return true;
                }
        }
        return false;
    }
```
* 原理简单，就是通过遍历的方式。但是需要注意的是，只会移除第一个找到的元素。 fastRemove方法和remove(index) 方法作用相同，只是去掉了边界校验和返回值。
---

查询
---
通过索引获取元素
```
public E get(int index) {
        rangeCheck(index);

        return elementData(index);
    }
```
* 原理就是通过数组获取指定index的元素。

从前往后查找第一个出现的元素：
```
public int indexOf(Object o) {
        if (o == null) {
            for (int i = 0; i < size; i++)
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = 0; i < size; i++)
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }
```
从后往前查找第一个出现的元素：
```
public int lastIndexOf(Object o) {
        if (o == null) {
            for (int i = size-1; i >= 0; i--)
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = size-1; i >= 0; i--)
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }
```
---
集合运算
---------------------------
并集:
* 并集就是addAll方法

交集:
```
 public boolean retainAll(Collection<?> c) {
        Objects.requireNonNull(c);
        return batchRemove(c, true);
    }
```
差集:
```
public boolean removeAll(Collection<?> c) {
        Objects.requireNonNull(c);
        return batchRemove(c, false);
    }
```
* 交集和差集都调用了batchRemove方法，通过一个boolean标记来表示要留下的是相同的一部分还是不同的一部分。 batchRemove方法的写法还是值得学习。
---
batchRemove:
```
private boolean batchRemove(Collection<?> c, boolean complement) {
        final Object[] elementData = this.elementData;
        int r = 0, w = 0;
        boolean modified = false;
        try {
            for (; r < size; r++)
                if (c.contains(elementData[r]) == complement)
                    elementData[w++] = elementData[r];
        } finally {
            // Preserve behavioral compatibility with AbstractCollection,
            // even if c.contains() throws.
            if (r != size) {
                System.arraycopy(elementData, r,
                                 elementData, w,
                                 size - r);
                w += size - r;
            }
            if (w != size) {
                // clear to let GC do its work
                for (int i = w; i < size; i++)
                    elementData[i] = null;
                modCount += size - w;
                size = w;
                modified = true;
            }
        }
        return modified;
    }
```
双指针的思路，一个读指针，一个写指针。读指针每次往前增加一个位置，写指针遇到要保留的元素时，再往前移动一个位置。
最后，如果读指针没走到最后就异常了，那把剩下读指针剩下的元素拷贝到写指针之后。
如果写指针没到最后，说明写指针后面的元素都需要剔除，手动置为null，否则会内存泄露。
如果写指针走到了最后，说明一个元素都没有移除掉，所以返回false。

* 作者：大雨还在下
链接：[https://juejin.im/post/5ce509736fb9a07ea420544d]
来源：掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
---
---

# ArrayList是如何扩容的？

* 创建一个更大的空间，然后把原先所有的元素拷贝进去


# HashMap是如何扩容的？

* 创建一个更大的空间，把所有东西拷贝进去

# HashMap从Java7到Java8发生了哪些变化？

* 在发生hash碰撞的时候，hash桶就不会恶化成和链表一样的性能，
而是演变成了名叫红黑树的数据结构，避免性能恶化

为什么HashMap不是线程安全的？
---

* 在多线程的环境下可能会发生死循环，这个时候就需要使用ConcurrentHashMpa来代替
---
