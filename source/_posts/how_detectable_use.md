---
title: 异常代码检测执行器Detectable（模仿java.utils.Optional类）
date: 2019-10-10 14:29:19
tags: java
---
工作中经常遇到一些代码，比如微服务调用的方法就是个很基本的增删改查功能，非要给你抛出一个`Exception`，很反感，我不是一个喜欢在代码里加try...catch...的人，也不太愿意抛异常，因为在我看来有一些错误是应该在开发过程中就完全避免的。但团队协作开发，，，以及一些公司自带的依赖实在是难以启齿。故仿照着`java.util.Optional`写了一个差不多模式的类：`Detectable`。没错，连名字风格都一样。

## 主要构成

1. `final class Detectable`，提供两个公开的静态方法，类似于`Optional.ofNullable`方法;
2. `functionalInterface Runner`，没有返回值的执行器；
3. `functionalInterface RunnerWithReturnValue`，有返回值的执行器
4. `functionalInterface ElseRun`，没有返回值的最终执行器。

## 源码

```java
@FunctionalInterface
public interface Runner {
  void run() throws Throwable;
}
```

```java
@FunctionalInterface
public interface RunnerWithReturnValue<R> {
  R run() throws Throwable;
}
```

```java
/**
 * 其实直接使用{@link java.lang.Runnable}也可以
 */
@FunctionalInterface
public interface ElseRun {
  void run();
}
```

```java
public final class Detectable<R> {
  private R returnValue;
  private Throwable throwable;

  private Detectable(R returnValue, Throwable throwable) {
    this.returnValue = returnValue;
    this.throwable = throwable;
  }

  public R orElse(R backup) {
    if (this.throwable == null) {
      return this.returnValue;
    }
    this.throwable.printStackTrace();
    return backup;
  }

  public void orElseRun(ElseRun runner) {
    if (this.throwable != null) {
      this.throwable.printStackTrace();
      runner.run();
    }
  }

  public Throwable getThrowableIfError() {
    return this.throwable;
  }

  // #########################################

  public static Detectable<?> ofThrowable(Runner runner) {
    try {
      runner.run();
      return new Detectable<>(null, null);
    } catch (Throwable t) {
      return new Detectable<>(null, t);
    }
  }

  public static <R> Detectable<R> ofThrowable(RunnerWithReturnValue<R> runner) {
    try {
      R value = runner.run();
      return new Detectable<>(value, null);
    } catch (Throwable t) {
      return new Detectable<>(null, t);
    }
  }
}
```

## 使用方式

```java
public class App {

  public static void main(String[] params) {
    // 执行普通方法
    Detectable
      .ofThrowable(() -> {
        throw new Exception("这个方法始终抛出一个异常");
      })
      .orElseRun(() -> System.err.println("产生异常！"));

    // 执行带有返回值的方法
    // 如果com.mysql.cj.jdbc.Driver不存在，则driverClass为null
    Class<?> driverClass = Detectable
      .ofThrowable(() -> Class.forName("com.mysql.cj.jdbc.Driver"))
      .orElse(null);

    // 获取异常
    // 如果代码没有异常则返回null
    Throwable throwable = Detectable
      .ofThrowable(() -> {
        Class.forName("java.error.ThisClassDoesNotExist")
      })
      .getThrowableIfError();

    // 异常向下转型
    ClassNotFoundException exp = (ClassNotFoundException) throwable;
  }
}
```

## 总结

其实也就模仿了Optional常用的使用方式，总体比较简单，但总算不用再写恶心的try...catch...了
