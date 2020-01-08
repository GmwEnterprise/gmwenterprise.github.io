---
title: 学习Java并发（02）使用线程类Thread
date: 2020-01-06 10:27:38
categories: 并发
tags:
  - java
  - concurrency
---

## Thread基本说明

在Java中，Thread类表示一个线程实例。Java支持多线程开发，JVM中可存在多个执行线程。

每个线程都拥有一个优先级，高优先级的线程先执行的概率更大（被调度器分配时间片的概率更大）。每个线程都可以被设置为守护线程（也就是后台线程，其随着JVM的结束而自动结束）。当JVM启动时，会存在一个非守护线程，用于调用main方法从而开始执行程序。当线程任务执行完毕后，或者抛出异常且未在线程代码中处理，线程将结束运行。

JVM在以下两种情况下会结束运行：
- 主动调用`Runtime.exit`或`System.exit`.
- 所有非守护线程都已执行结束.

## 创建一个Thread

直接创建一个线程并启动的方式如下。

``` java
// 创建一个任务
Runnable task = () -> System.out.printf("Current thread: %s%n", Thread.currentThread());
// 创建一个线程实例
Thread thread = new Thread(task);
// 启动线程
thread.start();
System.out.printf("Current thread: %s%n", Thread.currentThread());
```

其中`Thread.currentThread()`用来获取当前线程，写在主线程的printf方法将打印main线程，而写在task中的printf方法将打印子线程。

## Thread常用的自带方法

### void run()

很简单，调用传入的Runnable的run方法，仅调用方法而不开启线程。

### void start()

开启线程。内部调用本地C代码实现的native方法start0()。当执行了start方法后，这个线程就已经创建成功并进入就绪状态了。

### void setPriority(int)

设置线程的优先级。Java中线程的优先级分为10级，即1-10。通常使用三档优先级来设置：
``` java
/**
  * The minimum priority that a thread can have.
  */
public static final int MIN_PRIORITY = 1;

/**
  * The default priority that is assigned to a thread.
  */
public static final int NORM_PRIORITY = 5;

/**
  * The maximum priority that a thread can have.
  */
public static final int MAX_PRIORITY = 10;
```

优先级的设置在start方法执行前设置。

### void setDaemon(boolean)

设置是否为守护线程。守护线程会在JVM结束时自动结束。结束时假如代码块中有finally块，并不会执行。（类似于线程突然中止在当前行代码，不会向下执行下去）

### void join()

使当前线程等待目标线程执行完毕：
``` java
public static void main(String[] param) throws InterruptedException {
    // 初始化一个睡眠3秒的任务
    Thread t = new Thread(() -> {
        try {
            TimeUnit.SECONDS.sleep(3);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    });
    t.start();
    t.join();
    System.out.println("done."); // 会在3秒后执行

    // 调用t.join()的是主线程
    // 随后主线程开始等待子线程的执行结束
    // 子线程睡眠3秒后即结束
    // 主线程等待完毕
}
```
假如在主线程开启了很多个子线程任务，想要等待所有的任务执行完毕，可以轮询调用所有线程的join方法。

### boolean isInterrupted()

查看当前线程是否为中断状态。详细用法见`void interrupt()`。

### void interrupt()

中断目标线程。调用了这个方法后，目标线程的`isInterrupted()`方法返回true，即中断标志设为true。对于某些方法，还要求捕获`InterruptedException`。对于要求捕获这个异常的方法来说，一旦补货到了这个异常，中断标志又将重置，所以不要在catch块里判断中断标志。

### static void yield()

### static Thread currentThread()

获取当前线程。

