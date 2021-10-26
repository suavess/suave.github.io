---
title: Java集合类-RandomAccess接口
date: 2021-10-26 14:59:56
tags: 
    - Java
    - Java集合
category: 
    - Java
---

## 发现RandomAccess接口

因为面试的时候问到了怎样使一个 **ArrayList** 变得线程安全

1. 使用老牌的Vetor
2. 使用CopyOnWriteArrayList
3. 使用Collections.syncronizedList()方法传入一个List，会返回一个线程安全的List

答完后引申到Collections.syncronizedList()的实现原理，当时没看过具体实现并未回答上来。复盘时研究了下，发现首先会先判断传过来的List是否实现了RandomAccess接口
![](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/20210302112045.png)

## RandomAccess接口详解

点进Random接口发现该接口并没有任何属性和任何方法

![](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/20210302112503.png)

于是查看jdk8官方文档

![](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/20210302113604.png)

官方文档写明了，RandomAccess接口主要是一个标记接口，用来注明这个List是否支持随机访问。

> ArrayList实现了Random接口而LinkedList并未实现

```java
public class ArrayList<E> extends AbstractList<E> implements List<E>, RandomAccess, Cloneable, java.io.Serializable {}
public class LinkedList<E> extends AbstractSequentialList<E> implements List<E>, Deque<E>, Cloneable, java.io.Serializable {}
```

ArrayList底层是数组，占用一片连续的空间，可以通过计算偏移量来直接获取元素，即支持随机访问，此时使用for (int i=0, n=list.size(); i < n; i++){}循环时的速度要快于iterator顺序循环访问

LinkedList底层是双向链表，遍历要按顺序一个元素节点一个元素节点查询下去，此时iterator速度要快于for (int i=0, n=list.size(); i < n; i++){}

## 性能测试

- 随机往数组和链表中插入100000个元素，并分别打印遍历时间

![image-20210302211040899](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20210302211040899.png)

- 遍历结果

![image-20210302211247553](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20210302211247553.png)

## 总结

可以看到ArrayList（即RandomAccess接口标识的类）使用普通for循环遍历速度较快

而LinkedList（即未被RandomAccess接口标识的类）使用迭代器遍历速度远快于普通for遍历

后续遍历时可以根据是否被RandomAccess接口标识选择更优的遍历方式
