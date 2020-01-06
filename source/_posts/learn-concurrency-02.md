---
title: 学习Java并发（02）Java提供的API
date: 2020-01-06 10:27:38
categories: 并发
tags:
  - java
  - concurrency
---

## 前言

本文用于记录Java中最基本的线程类使用，如`Thread`，`Runnable`等。

## Java对任务的定义：`Runnable`

使用多线程，往往是用于处理多个任务，追求的是最大化利用CPU资源。Java中把单个的任务抽象为一个接口：`java.lang.Runnable`。这个接口只有一个`run()`方法，用于定义一个没有返回值的任务。直接实现这个接口便定义了一个任务，不过在开发中定义`Runnable`的通常方式是使用匿名内部类或Lambda表达式。

``` java
public interface Task implements Runnable {
  @Override
  public void run() {
    print("任务开始");
    // do something...
  }
}
```

这个任务定义好了，其本身却不能用于执行。要执行任务，得依靠Java提供的线程类：`java.lang.Thread`。

## `Thread`的基本使用

使用方式为：`new Thread(() -> print("任务开始"))).start()`。

使用构造器创建一个线程类实例，构造参数为一个Runnable的实现，然后调用其start方法，此时一个新的线程就进入就绪状态了，且会开始和主线程竞争CPU使用权。

### 线程优先权设置

Java中定义了1-10的不同等级的优先级，数值越高优先级越高，但不推荐使用这些全部的优先级。

``` java
public static final int MIN_PRIORITY = 1;

public static final int NORM_PRIORITY = 5;

public static final int MAX_PRIORITY = 10;
```

Thread内部定义了三种优先级，最高、最低、一般，使用这三种优先级来定义即可处理大多数情况。优先级越高的线程，获取到CPU时间片的概率也就越大。但也不是绝对说优先级能够决定执行顺序。

设置的方式如下：

``` java
Thread t = new Thread(() -> print("start..."));
t.setPriority(Thread.MAX_PRIORITY);
t.start();
```

### 守护线程

守护线程是指，这个线程会随着JVM的生命周期结束而结束。通常开启子线程后，主线程结束后，子线程会继续执行，每一个线程都必须执行完毕后JVM才会结束运行。守护线程适合用来设置一些周期性的任务，比如日志定时生产、缓存定时更新、后台监控等等。非守护线程全部执行完毕后，守护线程也会立即结束生命周期，不会再继续运行。守护线程所创建的所有线程都默认为守护线程，也就是说不需要担心在守护线程中的线程使用。

设置方式如下：

``` java
Thread t = new Thread(() -> {
  while (true) {
    print("Daemon thread...");
  }
});
t.setDaemon(true);
t.start();
```

守护线程的结束，其结束方式是突然结束，也就是说程序突然终止，不会执行下一行代码，哪怕设置了finally块，也不会执行。所以在守护线程中设置finally是不合适的选择。（类似于电脑断电与电脑自动关机的区别）