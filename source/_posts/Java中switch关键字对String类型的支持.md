---
title: Java中switch关键字对String类型的支持
date: 2022-06-16 20:09:14
tags: 
  - Java
  - 反编译
category:
  - Java
---

在java7以后，switch的参数可以是String类型了，到目前为止switch支持这样几种数据类型：`byte` `short` `int` `char` `String` 。接下来我们就看一下，switch到底是如何实现的。

> ps: 本文使用的反编译工具为jadx

### switch对整型的支持

一段很简单的通过整型进行switch的代码

```java
public class Test {
    public static void main(String[] args) {
        int a = 1;
        switch (a) {
            case 1:
                System.out.println("hello");
                break;
            case 2:
                System.out.println("world");
                break;
            default:
                break;
        }
    }
}
```

经过反编译后

```java
package com.example.demo;
/* loaded from: Test.class */
public class Test {
    public static void main(String[] args) {
        switch (1) {
            case 1:
                System.out.println("hello");
                return;
            case 2:
                System.out.println("world");
                return;
            default:
                return;
        }
    }
}
```

可以看到，经过反编译后的代码基本没有变化，**因此switch对int的判断是直接比较整数的值**。



### switch对字符类型的支持

```java
public class Test {
    public static void main(String[] args) {
        char a = 'b';
        switch (a) {
            case 'a':
                System.out.println("hello");
                break;
            case 'b':
                System.out.println("world");
                break;
            default:
                break;
        }
    }
}
```

经过反编译之后

```java
package com.example.demo;
/* loaded from: Test.class */
public class Test {
    public static void main(String[] args) {
        switch (98) {
            case 97:
                System.out.println("hello");
                return;
            case 98:
                System.out.println("world");
                return;
            default:
                return;
        }
    }
}
```

通过以上的代码作比较我们发现：对char类型进行比较的时候，实际上比较的是ascii码，编译器会把char型变量转换成对应的int型变量



### switch对字符串支持的实现

```java
public static void main(String[] args) {
    String str = "world";
    switch (str) {
        case "hello":
            System.out.println("hello");
            break;
        case "world":
            System.out.println("world");
            break;
        default:
            break;
    }
}
```

经过反编译之后

```java
package com.example.demo;
/* loaded from: Test.class */
public class Test {
    public static void main(String[] args) {
        char c = 65535;
        switch ("world".hashCode()) {
            case 99162322:
                if ("world".equals("hello")) {
                    c = 0;
                    break;
                }
                break;
            case 113318802:
                if ("world".equals("world")) {
                    c = 1;
                    break;
                }
                break;
        }
        switch (c) {
            case 0:
                System.out.println("hello");
                return;
            case 1:
                System.out.println("world");
                return;
            default:
                return;
        }
    }
}
```

原来字符串的switch是通过`equals()`和`hashCode()`方法来实现的。**记住，switch中只能使用整型**，比如`byte`。`short`，`char`(ackii码是整型)以及`int`。还好`hashCode()`方法返回的是`int`，而不是`long`。通过这个很容易记住`hashCode`返回的是`int`这个事实。仔细看下可以发现，进行`switch`的实际是哈希值，然后通过使用equals方法比较进行安全检查，这个检查是必要的，因为哈希可能会发生碰撞。因此它的性能是不如使用枚举进行switch或者使用纯整数常量，但这也不是很差。因为Java编译器只增加了一个`equals`方法，如果你比较的是字符串字面量的话会非常快，比如”abc” ==”abc”。如果你把`hashCode()`方法的调用也考虑进来了，那么还会再多一次的调用开销，因为字符串一旦创建了，它就会把哈希值缓存起来。因此如果这个`switch`语句是用在一个循环里的，比如逐项处理某个值，或者游戏引擎循环地渲染屏幕，这里`hashCode()`方法的调用开销其实不会很大。

总结一下我们可以发现，**其实switch只支持一种数据类型，那就是整型，其他数据类型都是转换成整型之后再使用switch的。**
