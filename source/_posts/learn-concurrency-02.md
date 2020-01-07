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

## Thread的基本用法

