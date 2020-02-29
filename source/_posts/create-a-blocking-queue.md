---
title: 自己实现一个阻塞队列
date: 2020-02-29 06:44:41
tags:
  - java
  - concurrent
  - container
---
### 前言
不继承jdk提供的接口，只实现概念中的十个重要方法，目的在于学习：
| 处理方式 | 抛出异常 | 返回特殊值 | 一直阻塞 | 超时退出 |
| :----: | :----: | :----: | :----: | :----: |
| 插入 | add(e) | offer(e) | put(e) | offer(e,time,unit) |
| 移除 | remove() | poll() | take() | poll(time, unit) |
| 检查 | element() | peek() | 

### LinkedList简析

我将要实现的阻塞队列内部用到了`LinkedLink`，只使用了其两个核心方法：`add`与`remove`。为了避免一些并发错误所以首先分析这两个方法，关键讨论是否可以在并发环境下同时执行这两个方法，即并发环境同时出入队列。

#### LinkedList.add方法简析：
``` java
// 代码经过优化，便于阅读
public boolean add(E e) {
  // 尾插法

  // this.last为尾指针，指向链表的尾节点
  // 新建指针lp指向当前尾指针指向的地址
  final Node<E> lp = this.last;

  // 构建即将插入的节点
  final Node<E> newNode = new Node<>(l, e, null);

  // 使尾指针指向新节点
  this.last = newNode;

  if (lp == null)
    // lp为空则意味着插入之前尾指针指向null，即队列无元素
    // 使头指针也指向新节点
    this.first = newNode;
  else
    // 插入之前队列已有元素
    // 当前lp已为尾节点的父节点，使其父节点的后继指针指向尾节点
    lp.next = newNode;

  // size自增
  this.size++;
  // modCount自增，该值记录此对象从结构上修改过的次数
  this.modCount++;

  // 插入成功返回true
  return true;
}
```

#### LinkedList.remove方法简析：
``` java
// 代码经过优化，便于阅读
public E remove() {
  // 移除头节点，即先进后出FIFO
  
  // 声明f指针指向头节点
  final Node<E> f = this.first;
  if (f == null)
    // 头节点为空则抛异常
    throw new NoSuchElementException();

  // 获取将要移除的元素
  final E element = f.item;

  // 声明next指针指向第二节点，移除后将变为头节点
  final Node<E> next = f.next;

  // 清空元素引用
  f.item = null;

  // 头节点后继指针断链
  f.next = null; // help GC

  // 对象头指针移位
  this.first = next;

  if (next == null)
    // 此时队列无元素，使头尾指针都指向null
    this.last = null;
  else
    // 使头节点前驱指针锻炼
    next.prev = null;

  // 长度自减
  size--;

  // 结构修改次数自增
  modCount++;

  // 返回从链表删除后的元素
  return element;
}

```

在源码中，二者都在方法最后统计了长度与修改次数。我将通过size方法来判断队列元素个数从而决定是否出入队列，所以并发的关键点在这里。当执行一次插入、删除操作时，只有当修改完了链表的结构后，size才会更新，所以对于阻塞队列的出入队列方法，可以分别使用不同的锁来控制，插入方法使用插入锁，移除方法使用获取锁。

### 代码实现

``` java
package cn.gmwenterprise.demo;

import java.util.LinkedList;
import java.util.NoSuchElementException;
import java.util.Optional;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

/**
 * 实现一个阻塞队列
 */
public class MyBlockingQueue<E> {
    private int maxSize;

    /**
     * 通过构造函数决定该队列的边界
     *
     * @param maxSize 最大容量
     */
    public MyBlockingQueue(int maxSize, long maxWaiting, TimeUnit timeUnit) {
        this.maxSize = maxSize;
    }

    /**
     * 核心队列
     */
    private LinkedList<E> queue = new LinkedList<>();
    /**
     * 锁
     */
    private Lock putLock = new ReentrantLock();
    private Condition putCondition = putLock.newCondition();
    private Lock getLock = new ReentrantLock();
    private Condition getCondition = getLock.newCondition();

    /**
     * 插入，若队列已满则抛出异常
     *
     * @param e 待插入元素
     * @return 插入成功返回true
     * @throws IllegalStateException 如果队列已满
     */
    public boolean add(E e) {
        if (offer(e)) {
            return true;
        }
        throw new IllegalStateException("队列已满");
    }

    /**
     * 插入，若队列已满则等待
     *
     * @param e 待插入元素
     * @return 插入成功返回true
     * @throws InterruptedException 阻塞等待过程中被意外中断
     */
    public boolean put(E e) throws InterruptedException {
        putLock.lock();
        try {
            // 使用while的原因是，当该线程被唤醒后必须再次进行一次容量判断
            while (queue.size() == maxSize) {
                putCondition.await();
            }
            // 此时容量未满
            queue.add(e);
            getCondition.signalAll();
            return true;
        } finally {
            putLock.unlock();
        }
    }

    /**
     * 插入，若队列已满则返回特殊值
     *
     * @param e    待插入元素
     * @param time 等待时间
     * @param unit 时间单位
     * @return 插入成功返回true，插入失败返回false
     */
    public boolean offer(E e, long time, TimeUnit unit) throws InterruptedException {
        putLock.lock();
        try {
            while (queue.size() == maxSize) {
                if (!putCondition.await(time, unit)) {
                    // 超时退出
                    return false;
                }
            }
            // 此时容量未满
            queue.add(e);
            getCondition.signal(); // 插入一个元素，唤醒一个取除线程
            return true;
        } finally {
            putLock.unlock();
        }
    }

    /**
     * 插入，若队列已满则返回特殊值
     *
     * @param e 待插入元素
     * @return 插入成功返回true，插入失败返回false
     */
    public boolean offer(E e) {
        putLock.lock();
        try {
            if (queue.size() == maxSize) {
                return false;
            }
            // 此时容量未满
            queue.add(e);
            getCondition.signal(); // 插入一个元素，唤醒一个取除线程
            return true;
        } finally {
            putLock.unlock();
        }
    }

    /**
     * 移除，若队列为空返回{@code null}
     *
     * @return 等同于 {@link LinkedList#poll}
     */
    public E poll() {
        getLock.lock();
        try {
            if (queue.size() == 0) {
                return null;
            }
            return queue.removeFirst();
        } finally {
            getLock.unlock();
        }
    }

    /**
     * 移除，若队列为空返回{@code null}
     *
     * @param time 等待时间
     * @param unit 时间单位
     * @return 等同于 {@link LinkedList#poll}
     */
    public E poll(long time, TimeUnit unit) throws InterruptedException {
        getLock.lock();
        try {
            while (queue.size() == 0) {
                if (!getCondition.await(time, unit)) {
                    return null;
                }
            }
            return queue.removeFirst();
        } finally {
            getLock.unlock();
        }
    }

    /**
     * 移除元素，若队列为空抛异常
     *
     * @return 移除后的元素
     * @throws NoSuchElementException 为空则抛出
     */
    public E remove() {
        return Optional.ofNullable(poll()).orElseThrow(NoSuchElementException::new);
    }

    public E take() throws InterruptedException {
        getLock.lock();
        try {
            while (queue.size() == 0) {
                getCondition.await();
            }
            return queue.removeFirst(); // 理论上来说不会抛出NoSuchMethodException
        } finally {
            getLock.unlock();
        }
    }

    // TODO element() & peek()
}

```