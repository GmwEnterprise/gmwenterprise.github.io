---
title: 学习Java并发（02）Java中使用线程
date: 2020-01-06 10:27:38
categories: 并发
tags:
  - java
  - concurrency
---

## 基本说明

在Java中，Thread类表示一个线程实例。Java支持多线程开发，JVM中可存在多个执行线程。

每个线程都拥有一个优先级，高优先级的线程先执行的概率更大（被调度器分配时间片的概率更大）。每个线程都可以被设置为守护线程（也就是后台线程，其随着JVM的结束而自动结束）。当JVM启动时，会存在一个非守护线程，用于调用main方法从而开始执行程序。当线程任务执行完毕后，或者抛出异常且未在线程代码中处理，线程将结束运行。

JVM在以下两种情况下会结束运行：
- 主动调用`Runtime.exit`或`System.exit`.
- 所有非守护线程都已执行结束.

## Java中线程的状态

Java中线程有6种状态：
- NEW: 线程刚刚创建，未执行.
- RUNNABLE: 线程启动后，便处于这个状态；线程等待操作系统资源（比如I/O阻塞、CPU资源）时，也处于这个状态
- BLOCKED: 线程因等待锁阻塞
- WAITING: 
- TIMED_WAITING: 
- TERMINATED: 

## 创建一个线程

首先要新建一个任务。通过实现Runnable接口即可创建一个没有返回值的任务。
``` java
// 方法1
public Task implements Runnable {
  @Override
  public void run() { print("Task run !"); }
}
// 方法2
Runnable task = () -> print("Task run !");
```
创建好了任务后，将这个任务实例作为Thread的构造参数传入并构建Thread实例：
``` java
Thread t = new Thread(task);
// 调用start()启动线程
t.start();
```
调用了start方法后，该线程就启动了。

实际上也可以选择继承Thread类来覆盖其run()方法来定义任务，然而Java只允许继承一个类，但支持多接口实现，所以为了可扩展性通常会选择实现Runnable的方式。

## 设置线程的优先级

Java中线程的优先级分为10档，即数值1-10。Thread类本身提供了三档优先级常量，分别为`Thread.MIN_PRIORITY`, `Thread.NORM_PRIORITY`, `Thread.MAX_PRIORITY`，值分别为1，5，10。值越大，优先级越高，调度器为线程分配时间片概率也就更大。设置方法：
``` java
// 需要在启动前设置
t.setPriority(Thread.MAX_PRIORITY);
t.start();
```

## 设置线程为守护线程

设置为守护线程后，其会随着JVM的结束而结束。当JVM结束时，如果有守护线程正在执行代码，其会突然终止在当前行代码，且不会往下继续执行（哪怕有finally块）。设置的方式如下：
``` java
t.setDaemon(true); // true即开启守护线程，为false则不开启。非守护线程设置时默认为false
t.start();
```
作为守护线程，其创建 / 派生出的任何子线程都默认为守护线程，即：
``` java
Runnable daemon = () -> {
  print("该线程为守护线程 " + Thread.currentThread().isDaemon());
  new Thread(() -> print("派生线程默认为守护线程 " + Thread.currentThread().isDaemon())).start();
}
new Thread(daemon).start();
```

## 线程的休眠

使线程休眠，指线程放弃当前执行权，在指定的时间内进入阻塞状态。当指定时间过了以后，线程进入就绪状态，重新参与CPU资源的竞争。
使当前线程休眠的方式如下：
``` java
try {
  // 1
  Thread.sleep(1000); // 休眠1000ms也就是1s
  // 2
  TimeUnit.MILLISECONDS.sleep(100); // 休眠100ms
  // 3
  TimeUnit.SECONDS.sleep(1); // 休眠1s
} catch (InterruptedException e) {
  print("休眠阻塞被中断");
}
```
后两种方式更具有可读性。还有一种特别的使用方式：`Thread.sleep(0)`，休眠0秒，其作用是立刻使当前线程进入就绪态，重新参与CPU资源的分配。

线程休眠实际上是使线程阻塞的一种方式，同时休眠阻塞是可以被打断的，通过捕获`InterruptedException`来处理中断信号。

使线程休眠并不会使线程放弃当前持有的锁。

## 线程的让步

线程放弃当前CPU的使用，进入就绪状态，让其他线程得以执行。但调度器可能会忽略掉这个请求。
> 有种说法是`yield()`与`sleep(0)`的差别仅在于前者得以让优先级更低的线程能够执行，而后者只会把执行机会让给优先级大于等于自己的线程。但经测试发现二者的使用并无太大的差别。可能不同的平台会有一些调度上策略的不同。

使用方式：`Thread.yield()`。这是一个静态方法，直接调用使当前线程让出CPU。通常用于调试，测试。该方法不会释放锁。

## 线程的阻塞