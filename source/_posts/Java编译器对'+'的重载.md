---
title: Java编译器对'+'的重载
date: 2022-06-16 10:03:21
tags: 
  - Java
  - 反编译
category: 
  - Java
---

Java中对字符串进行拼接一般有两种方式，通过Stirng.concat()方法进行拼接，或者直接使用``+``号拼接，一般来说，我们都会使用第二种方式。

有人把Java中使用+拼接字符串的功能理解为运算符重载。其实并不是，Java是不支持运算符重载的。这其实只是Java提供的一个语法糖。

> 运算符重载：在计算机程序设计中，运算符重载（英语：operator overloading）是多态的一种。运算符重载，就是对已有的运算符重新进行定义，赋予其另一种功能，以适应不同的数据类型。

> 语法糖：语法糖（Syntactic sugar），也译为糖衣语法，是由英国计算机科学家彼得·兰丁发明的一个术语，指计算机语言中添加的某种语法，这种语法对语言的功能没有影响，但是更方便程序员使用。语法糖让程序更加简洁，有更高的可读性。

这样的一段代码，让我们看看反编译后的结果

![image-20220616102447895](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20220616102447895.png)

使用IDEA直接查看编译后的class文件

![image-20220616102539916](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20220616102539916.png)

通过查看反编译以后的代码，我们可以发现，原来字符串常量在拼接过程中，是将String转成了StringBuilder后，使用其append方法进行处理的。

那么也就是说，Java中的+对字符串的拼接，其实现原理是使用StringBuilder.append。



但是，String的使用+字符串拼接也不全都是基于StringBuilder.append，还有种特殊情况，那就是如果是两个固定的字面量拼接，如：

```java
String ab = "a" + "b";
```

反编译后

![image-20220616102923574](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20220616102923574.png)

这主要是因为两个字符串都是编译期常量，编译期可知，因此编译器会进行常量折叠，直接变成 String s = "ab"。
