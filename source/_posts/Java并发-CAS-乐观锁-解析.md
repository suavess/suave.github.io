---
title: Java并发-CAS-乐观锁-解析
date: 2021-10-26 15:05:07
tags: 
    - Java
    - Java锁机制
category: 
    - Java
---
>  本文讲解CAS机制，主要是因为最近准备面试题，发现这个问题在面试中出现的频率非常的高，因此把自己学习过程中的一些理解记录下来。

### 乐观锁与悲观锁

- synchronized是悲观锁，这种线程一旦得到锁，其他需要锁的线程就挂起的情况就是悲观锁。

- CAS操作的就是乐观锁，每次不加锁而是假设没有冲突而去完成某项操作，如果因为冲突失败就重试，直到成功为止。

### 模拟线程安全问题

先看下面的代码，执行结果不言而喻，最终的结果一定是小于预期的1000

```java
/**
 * 并发时线程安全问题
 *
 * @author Suave
 * @author 2021/3/5 1:50 下午
 */
public class Test01 {

    static int count = 0;

    public static void main(String[] args) throws InterruptedException {
        long startTime = System.currentTimeMillis();
        int threadSize = 100;
        CountDownLatch countDownLatch = new CountDownLatch(threadSize);
        for (int i = 0; i < threadSize; i++) {
            new Thread(() -> {
                for (int j = 0; j < 10; j++) {
                    try {
                        request();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                countDownLatch.countDown();
            }).start();
        }
        countDownLatch.await();
        long endTime = System.currentTimeMillis();
        System.out.printf("消耗时长: %d, 结果: %d", endTime - startTime, count);
    }

    public static void request() throws InterruptedException {
        TimeUnit.MILLISECONDS.sleep(5);
        count++;
    }
}
```

!["执行结果"](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20210305154019790.png)

这段代码线程不安全的具体原因是 **count++** 并不是一个原子性的操作，count++实际会经过三个步骤

- 获取 count 的值，先记为 A	A = count
- 将 A 的值加一，得到B	B = A+1
- 将 B 的值赋给 count	count = B

在多线程的情况下，可能会有多个线程同时走到第一步，同时获取到了相同的A

此时新增后将值赋给B就会出现多个线程调用了这个方法结果count只加了一的情况

### 线程安全问题解决

- 线程安全的问题可以通过加锁的方式解决，先来试试最常用也最方便的synchronized关键字，即假设每次都会有冲突，一次只允许一个线程进入的悲观锁

```java
// 在request方法上面加上synchronized关键字
public synchronized static void request() throws InterruptedException {
        TimeUnit.MILLISECONDS.sleep(5);
        count++;
}
```

​	再来运行试一下

!["执行结果"](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20210305155226372.png)

​	可以看到，执行结果和我们预期的一致，线程已经安全了。

​	但是，这个运行效率就很感人了.........

- 用CAS乐观锁解决

增加一个CAS方法，同时更改request方法，每次赋值的时候去比较一下值是否发生了改变，改变了就重试

```java
		/**
     * CAS方法
     *
     * @param expectCount 期望值count
     * @param newCount    需要给count赋的新值
     * @return true 成功 false 失败
     */
    public static synchronized boolean compareAndSwap(int expectCount, int newCount) {
      if (count == expectCount) {
        count = newCount;
        return true;
      }
      return false;
    }

    public static void request() throws InterruptedException {
        TimeUnit.MILLISECONDS.sleep(5);
        int expectCount;
        // expectCount返回false时要再获取count的值并重新赋值
        while (!compareAndSwap(expectCount=count, expectCount + 1)) {}
    }
```

!["执行结果"](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20210305160816328.png)

可以看到，执行结果与我们预期的一致，性能也要强于synchronized悲观锁

***那么这段乐观锁的代码就一定不会出问题了吗？***

CPU执行某一段代码的时候，会先将数据读取到高速缓存中，极端情况下，当该线程将数据读取到高速缓存中时，其他线程更新了主存中的值，此时我们缓存中的值就不准确了

当我们以缓存中的值做CAS操作时，就会与预期值不一致了，因此，还要在 **count** 属性上加上 ***volatile*** 关键字，保证程序的可见性，每次获取该值时都去主存中获取

### CAS的缺点

当然，CAS也是有缺点的

1. CPU开销较大
   在并发量比较高的情况下，如果许多线程反复尝试更新某一个变量，却又一直更新不成功，循环往复，会给CPU带来很大的压力。

2. 不能保证代码块的原子性
   CAS机制所保证的只是一个变量的原子性操作，而不能保证整个代码块的原子性。比如需要保证3个变量共同进行原子性的更新，就不得不使用Synchronized了。