---
title: Java中的值传递和引用传递
date: 2021-10-26 15:06:05
tags: 
    - Java
    - Java基础
category: 
    - Java
---
> Java中的参数传递分为值传递和引用传递，基本类型是值传递，封装类是引用传递。

# 值传递

```java
public class Test01 {
    public static void change(int num) {
        num = 1;
    }

    public static void main(String[] args) {
        int num = 0;
        change(num);
        System.out.println("num=" + num);
    }
}
```

上面这段代码的输出结果是`num=0`。

当调用`change`方法时，jvm实际上是复制了一份参数传入方法中，无论这个复制后的参数怎么变，`main`方法中的`num`参数都是不会发生变化的。

从内存分配的角度来看，栈中是有两个变量的。一个是`main`方法中定义的`num`，另外一个是在传参时jvm复制的一份`num`副本，我们就叫它`temp`。不论`temp`值怎么改变，`num`是不会变的。

# 引用传值

```java
public class Test01 {

    public static void changeAge(Student student) {
        student.setAge(20);
    }

    public static void main(String[] args) {
        Student student = new Student();
        student.setAge(18);
        changeAge(student);
        System.out.println("student.age=" + student.getAge());
    }

    static class Student {
        int age;

        public int getAge() {
            return age;
        }

        public void setAge(int age) {
            this.age = age;
        }
    }

}
```

上面这段代码的输出结果是`student.age=20`。

`main`方法`new Student`时，jvm在堆中分配了一块内存用于存放`Student`对象。并且在栈中存放一个`student`对象，值是指向堆中`Student`对象的地址。

把`student`变量传入`changeAge()`方法时，jvm复制了一份`student`变量。为了便于理解我们叫它`temp`，`temp`的值和`student`是一样的，都是指向堆中`Student`对象的地址。

也就是说两个指针都指向了堆中`Student`对象的地址，此时修改的也就是堆中的该对象，因此`main`中的`student`对象也发生了变化

# 特殊的String

> 按照上面的说法，String不是基本类型，按理来说应该是引用传递

```java
public class Test01 {

    public static void change(String str) {
        str = "456";
    }

    public static void main(String[] args) {
        String str = "123";
        change(str);
        System.out.println("str = " + str);
    }

}
```

实际上上面输出的结果是`str = 123`。

jvm在实例化字符串时会使用字符串常量池，把`str`作为参数传入`change()`方法。jvm复制了一份`str`变量，为了便于理解我们叫它`temp`。这个时候`str`和`temp`都指向字符串常量池中的`"123"`。

后续给`temp`重新赋值为`"456"`，可以理解为在常量池中新建了`"456"`这个字符串，并将`temp`指向常量池中的`"456"`，此时`str`的指向是没有发生变化的，因此还是`"123"`。

