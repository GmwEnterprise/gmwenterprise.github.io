---
title: 学习Java I/O系统
date: 2019-12-22 00:27:09
tags:
  - java
  - learn
---

## 前言/简介

本文源自于笔者阅读《Java编程思想》这本书时所记录的知识点以及对于联想到的一些东西的思考。作为一个Java码农，毕业一年半了却一直没有好好了解一下java的IO系统，实在是不应该，故以此文来整理一下。

## 开始记录

### File类

> `java.io.File`这个类是一个工具类库，提供了很方便的本地文件操作。File既可以表示一个文件，也可以表示一个文件夹，虽然类名为File但其实用Path更为恰当，因为它实质上引用的是一个路径，而不是文件或文件夹。另外，有一个更方便的类可以替代File的操作：`java.nio.Path`

常用操作：

``` java
void processDirs() {
  File f1 = new File("C:\\Windows");
  System.out.println(f1.isFile()); // false
  System.out.println(f1.isDirectory()); // true

  File f2 = new File("C:\\Windows\\notepad.exe");
  System.out.println(f2.isFile()); // true
  System.out.println(f2.isDirectory()); // false
  System.out.println(f2.canRead()); // true
  System.out.println(f2.canWrite()); // true
  // 是否可执行
  System.out.println(f2.canExecute()); // true
  System.out.println(f2.length()); // 181248

  File f3 = new File("C:\\Windows\\nothing");
  System.out.println(f3.isFile()); // false
  System.out.println(f3.isDirectory()); // false
  
  File f4 = new File("../.");
  // 返回构造方法传入的路径（格式化为合法路径）
  System.out.println(f4.getPath()); // ..\.
  // 返回绝对路径（这个路径无法在win10资源管理器中的地址栏输入访问，会提示找不到）
  System.out.println(f4.getAbsolutePath()); // C:\Users\Me\AppData\Local\Temp\0000016f2c87ea13\..\.
  // 返回规范路径。规范路径会把'..', '.'转换成标准的绝对路径
  System.out.println(f4.getCanonicalPath()); // C:\Users\Me\AppData\Local\Temp
  Arrays.stream(Objects.requireNonNull(f4.listFiles()))
    .map(File::getName)
    .forEach(System.out::println); // 输出Temp文件夹下所有文件、文件夹的名称

  // 还有很多有用的方法这里不再一一赘述
}
```

### Path类



### In / Out简介

所谓的“流”是一个编程语言中的抽象概念，它表示任何有能力产出数据的数据源或有能力接收数据的接收器。“流”屏蔽了实际的I/O设备中处理数据的细节。对于Java语言来说，掌握流的操作就能基本地掌握数据地输入输出控制。

`java.io.InputStream`与`java.io.OutputStream`是I/O流中最顶级的两个抽象类，所有的I/O流相关的操作类都要直接或间接继承这两个类。其中，`InputStream`用于从流中读取数据到内存；而`OutputStream`则是从内存中输出数据到流。而`InputStream`的创建则必须指定一个数据源，用于为流提供数据；当然，`OutputStream`也需要一个接收数据的接收端用于初始化其输出数据到什么地方。

用一段代码来简单地描述一下这两个抽象类的使用：

``` java
public Serializable deepCopy(Serializable serializableObj) throws Exception {
    // 从内存中写入数据到输出流，最终目的地是字节流
    var byteArrayOutputStream = new ByteArrayOutputStream();
    // 使用对象输出流来装饰字节输出流，装饰器模式的实际应用
    var objectOutputStream = new ObjectOutputStream(byteArrayOutputStream);
    objectOutputStream.writeObject(serializableObj);

    // 从数据源中读取数据到内存，数据源则是先前的字节流数组
    var byteArrayInputStream = new ByteArrayInputStream(byteArrayOutputStream.toByteArray());
    var objectInputStream = new ObjectInputStream(byteArrayInputStream);

    var result = (Serializable) objectInputStream.readObject();

    // 关闭流
    objectInputStream.close();
    byteArrayInputStream.close();
    objectOutputStream.close();
    byteArrayOutputStream.close();
    return result;

    // 这种序列化方式存在一定的安全隐患，因为实例直接由JVM通过字节数组生成，不会执行构造方法
    // 仅适用于Java程序，如果要和其他程序通信必须使用通用的序列化方法，例如JSON、XML
}
```

这一段代码实现了一个java当中很实用的一个功能：深拷贝。原理是将内存中的对象序列化为字节流，再重新读取字节流，如此便实现了一个对象的深度拷贝，不论是其中的值还是引用，都具有完全不同 / 全新的地址。而`Object.clone()`只能实现浅拷贝，对于对象的引用成员只能拷贝其地址而无法拷贝真实数据。

实际上其中的`ByteArrayStream`可以替换为其他的数据源 / 接收器，比如文件流，而且文件流还能实现数据的持久化。

### In / Out当中的设计模式：装饰器模式

> 装饰器模式（Decorator Pattern）允许向一个现有的对象添加新的功能，同时又不改变其结构。这种类型的设计模式属于结构型模式，它是作为现有的类的一个包装。
> 这种模式创建了一个装饰类，用来包装原有的类，并在保持类方法签名完整性的前提下，提供了额外的功能。
> 
> 与代理模式的区别：*装饰器模式应当为所装饰的对象提供增强功能，而代理模式对所代理对象的使用施加控制，并不提供对象本身的增强功能。—— 阎宏：《java与模式》*

Java的I/O系统在设计时，将类型分成了两部分：一部分是确定能够访问数据源的基础`InputStream`；而另一部分则是提供额外功能的扩展`InputStream`:

![InputStream结构](https://i.loli.net/2019/12/24/z4fcZj1NTX3xaHK.png)

`OutputStream`也是如此：

![OutputStream结构](https://i.loli.net/2019/12/24/eylOEWi5d9Zw6AQ.png)

使用方法大致如下：
1. 首先确定一个数据源，比如想要从文件中读取数据的话，就使用文件流：`var fileIn = new FileInputStream(new File(targetPath))`；
2. 想要使用缓冲流来为文件流添加额外的读取缓冲能力，就是用缓冲流包装文件流：`var bufIn = new BufferedInputStream(fileIn)`;
3. 想要格式化读取文件中的基础数据：`var dataIn = new DataInputStream(bufIn)`。

`DataInputStream`提供了平台无关的基础数据读取功能，可以很方便地读取基本类型数据如`char`, `int`, `long`, `boolean`等。
使用地过程就是这么简单，想要添加新的扩展方法只需要用新的扩展类包装原本的类就好了；层层嵌套，便能够添加各种功能。

### `InputStream`类型

**`InputStream`的作用是用来表示那些从不同数据源产生输入的类**。这些数据源包括：
- 字节数组：`ByteArrayInputStream`
- String对象：`StringBufferInputStream` *注:这个类已过时，被`StringReader`所取代*
- 文件：`FileInputStream`
- 管道（一端输入，另一端输出）：`PipedInputStream`
- 其他数据源
- 其他种类的流组成的序列

`FilterInputStream`这个类被描述为一个抽象类，它是`BufferedInputStream`、`DataInputStream`、`LineNumberInputStream`、`PushbackInputStream`的基类。其他的输入流都可以作为这几个类的数据源，它可以增强其他几个`InputStream`类的功能；但实际上代码中`FilterInputStream`并没有被`abstract`关键字修饰，它的构造函数仅接受一个`InputStream`作为参数，其他几个主要的方法都是直接调用接收的`InputStream`来执行。

简单一点的说法：输入流从数据源读取数据到JVM(read)。

### `OutputStream`类型

**`OutputStream`表示了数据即将去往的目的地/接收端**：字节数组、文件、管道。当然，输出流也具有一个装饰器类的基类：`FilterOutputStream`。

也可以这么说：数据从JVM经输出流发往接收端(write)。

#### `OutputStream.flush` 

`OutputStream.flush()`这个方法的作用是将缓冲区的内容真正输出到数据目的地，以保证数据全部输出。因为在实际使用输出流时，通常会搭配缓存以减少真正物理写入的操作。

> 为什么要有`flush()`？因为向磁盘、网络写入数据的时候，出于效率的考虑，操作系统并不是输出一个字节就立刻写入到文件或者发送到网络，而是把输出的字节先放到内存的一个缓冲区里（本质上就是一个`byte[]`数组），等到缓冲区写满了，再一次性写入文件或者网络。对于很多IO设备来说，一次写一个字节和一次写1000个字节，花费的时间几乎是完全一样的，所以`OutputStream`有个`flush()`方法，能强制把缓冲区内容输出。
> *（本段内容取自[廖雪峰官方java教程](https://www.liaoxuefeng.com/wiki/1252599548343744/1298069169635361)）*

### I/O的阻塞

**`InputStream.read()`以及`OutputStream.write()`这两个方法都是阻塞的**。阻塞是指当代码执行到这一行时，必须等待`read()`或`write()`执行完毕才能够执行下一行，当前线程也会进入阻塞状态。

### `Reader`和`Writer`

`Reader`和`Writer`在Java的I/O系统中提供了兼容Unicode与面向字符的功能。设计这两个类继承层次结构主要是为了国际化，因为老的I/O流无法很好地处理16位的Unicode字符（Java本身的`char`就是16位的Unicode），所以设计这两个类就是为了在所有的I/O操作中都支持Unicode。

### `RandomAccessFile`

支持对文件**随机访问**的类。

> *In computer science, random access (more precisely and more generally called direct access) is the ability to access an arbitrary element of a sequence in equal time or any datum from a population of addressable elements roughly as easily and efficiently as any other, no matter how many elements may be in the set.*
> 
> *-- From Wikipedia*

它不继承于`InputStream`或`OutputStream`，仅实现了`DataOutput`、`DataInput`、`Closeable`这三个接口，直接由`Object`派生而来，本质上的功能类似于将`DataInputStream`和`DataOutputStream`组合起来使用，且添加了其他有用的方法。

`RandomAccessFile`很适合网络中的文件断点续传、多线程下载。

### 新I/O

也就是NIO，位于`java.nio.*`。引入NIO的目的在于充分提高速度。实际上旧的I/O包已经使用nio重新实现过，所以即便不显示使用NIO我们也能从中受益。