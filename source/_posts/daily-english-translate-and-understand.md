---
title: 每日翻译一段话【坚持就是胜利】
date: 2020-01-19 12:49:23
tags:
  - English
  - learn
---
# Documentation

## Java Development Kit

### java.util.concurrent.DelayQueue

An unbounded **blocking queue** of `Delayed` elements, in which an element can only be taken when its delay has expired. The *head* of the queue is that `Delayed` element whose delay expired furthest in the past. If no delay has expired there is no head and `poll` will return `null`. Expiration occurs when an element's `getDelay(TimeUnit.NANOSECONDS)` method returns a value less than or equal to zero. Even though unexpired elements cannot be removed using `take` or `poll`, they are otherwise treated as normal elements. For example, the `size` method returns the count of both expired and unexpired elements. This queue does not permit null elements.

This class and its iterator implement all of the *optional* methods of the `Collection` and `Iterator` interfaces. The Iterator provided in method `iterator()` is *not* guaranteed to traverse the elements of the DelayQueue in any particular order.

This class is a member of the **Java Collections Framework**.

一个没有界限的、元素为Delayed实例的阻塞队列，其中的元素只有在到达指定延迟时间才能被取出。队列的头指针是在过去延迟时间最久的。如果没有元素延时过期那么就没有头指针且poll方法会返回null。当一个元素的getDelay方法返回一个小于等于0的值时，过期即触发。即使没有过期的元素不能通过take或poll方法被移除，在其他处理情况也会被认为是普通元素，比如size方法就同时返回过期和未过期的所有元素总和。此队列不允许空元素。

此类以及它的迭代器实现了所有的源自Collection与Iterator接口的optional方法。不保证iterator方法提供的迭代器能够以通常的顺序遍历。

此类为Java Collections Framework的成员。

### java.lang.Number

The abstract class `Number` is the superclass of platform classes representing numeric values that are convertible to the primitive types `byte`, `double`, `float`, `int`, `long` and `short`. The specific semantics of the conversion from the numeric value of a particular `Number` implementation to a given primitive type is defined by the `Number` implementation in question. For platform classes, the conversion is often analogous to a narrowing primitive conversion or a widening primitive conversion as defined in *The Java Language Specification* for converting between primitive types. Therefore, conversions may lose information about the overall magnitude of a numeric value, may lose precision, and may even return a result of a different sign then the input. See the documentation of a given `Number` implementation for conversion details.

抽象类Number是代表了可以转换为原始类型`byte`, `double`, `float`, `int`, `long`,  `short`的数值类型的平台类的超类。从特定的Number实现到给定的原始类型的转换的具体语义由所讨论的Number实现来定义。对于平台类来说，转换通常类似于在Java语言规范中定义的原始类型之间进行转换的变窄的原始转换或变宽的原始转换。因此，转换可能会丢失有关数值总大小的信息，可能会丢失精度，甚至可能返回一个与输入不同的符号结果。具体转换细节请查看给定Number实现的文档。