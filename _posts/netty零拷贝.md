---
title: netty零拷贝
date: 2020-07-31
categories:
- 高性能编程
tags: 
- 高并发网络编程
- Netty
---



> 使用ByteBuf是Netty高性能很重要的一个原因 !



## 一 : ByteBuf

ByteBuf 是为解决 ByteBuffer 的问题和满足网络应用程序开发人员的日常需求而设计的。

JDK ByteBuffer的缺点︰

* **无法动态扩容 :** 长度是固定, 不能动态扩展和收缩, 当数据大于ByteBuffer容量时, 会发生索引越界异常。
* **API使用复杂 :** 读写的时候需要手工调用flip()和rewind()等方法, 使用时需要非常谨慎的使用这些api,否则很容出现错误

ByteBuf 做了哪些增强

* API操作便捷性
* 动态扩容
* 多种 ByteBuf实现
* 高效的零拷贝机制



## 二 : ByteBuf 操作

ByteBuf 三个**重要属性** : capacity容量、readerIndex读取位置、writerIndex写入位置。

提供了两个指针变量来支持顺序读和写操作, 分别是readerIndex和写操作writerIndex

常用方法定义

* 随机访问索引 getByte
* 顺序读read*
* 顺序写write*
* 清除已读内容discardReadBytes
* 清除缓冲区clear
* 搜索操作
* 标记和重置
* 引用计数和释放

下图显示了一个缓冲区是如何被两个指针分割成三个区域的

![](https://gcore.jsdelivr.net/gh/info4z/blog_images@main/images/image-20230211142430013.png) 

代码示例

```java
public class ByteBufDemo {
    @Test
    public void apiTest() {
        // 1.创建一个非池化的ByteBuf, 大小为10个字节
        ByteBuf buf = Unpooled.buffer(10);
        System.out.println("原始ByteBuf为====================>" + buf.toString());
        System.out.println("1.ByteBuf中的内容为===============>" + Arrays.toString(buf.array()) + "\n");

        // 2.写入一段内容
        byte[] bytes = {1, 2, 3, 4, 5};
        buf.writeBytes(bytes);
        System.out.println("写入的bytes为====================>" + Arrays.toString(bytes));
        System.out.println("写入一段内容后ByteBuf为===========>" + buf.toString());
        System.out.println("2.ByteBuf中的内容为===============>" + Arrays.toString(buf.array()) + "\n");

        // 3.读取一段内容
        byte b1 = buf.readByte();
        byte b2 = buf.readByte();
        System.out.println("读取的bytes为====================>" + Arrays.toString(new byte[]{b1, b2}));
        System.out.println("读取一段内容后ByteBuf为===========>" + buf.toString());
        System.out.println("3.ByteBuf中的内容为===============>" + Arrays.toString(buf.array()) + "\n");

        // 4.将读取的内容丢弃
        buf.discardReadBytes();
        System.out.println("将读取的内容丢弃后ByteBuf为========>" + buf.toString());
        System.out.println("4.ByteBuf中的内容为===============>" + Arrays.toString(buf.array()) + "\n");

        // 5.清空读写指针
        buf.clear();
        System.out.println("将读写指针清空后ByteBuf为==========>" + buf.toString());
        System.out.println("5.ByteBuf中的内容为===============>" + Arrays.toString(buf.array()) + "\n");

        // 6.再次写入一段内容, 比第一段内容少
        byte[] bytes2 = {1, 2, 3};
        buf.writeBytes(bytes2);
        System.out.println("写入的bytes为====================>" + Arrays.toString(bytes2));
        System.out.println("写入一段内容后ByteBuf为===========>" + buf.toString());
        System.out.println("6.ByteBuf中的内容为===============>" + Arrays.toString(buf.array()) + "\n");

        // 7.将ByteBuf清零
        buf.setZero(0, buf.capacity());
        System.out.println("将内容清零后ByteBuf为==============>" + buf.toString());
        System.out.println("7.ByteBuf中的内容为================>" + Arrays.toString(buf.array()) + "\n");

        // 8.再次写入一段超过容量的内容
        byte[] bytes3 = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11};
        buf.writeBytes(bytes3);
        System.out.println("写入的bytes为====================>" + Arrays.toString(bytes3));
        System.out.println("写入一段内容后ByteBuf为===========>" + buf.toString());
        System.out.println("8.ByteBuf中的内容为===============>" + Arrays.toString(buf.array()) + "\n");
        //  随机访问索引 getByte
        //  顺序读 read*
        //  顺序写 write*
        //  清除已读内容 discardReadBytes
        //  清除缓冲区 clear
        //  搜索操作
        //  标记和重置
        //  完整代码示例：参考
        // 搜索操作 读取指定位置 buf.getByte(1);
        //
    }

}
```

## 三 : ByteBuf 动态扩容

capacity默认值:256字节、最大值:Integer.MAX_VALUE ( 2GB)

`write*` 方法调用时, 通过 `AbstractByteBuf.ensureWritable0` 进行检查。

容量计算方法 : AbstractByteBufAllocator. calculateNewCapacity (新capacity的最小要求, capacity最大值), 根据新capacity的最小值要求, 对应有两套计算方法:

1. **没超过4兆 :** 从64字节开始, 每次增加一倍, 直至计算出来的newCapacity满足新容量最小要求。示例 : 当前大小256, 已写250, 继续写10字节数据, 需要的容量最小要求是261, 则新容量是  64 * 2 * 2 * 2=512
2. **超过4兆 :** 新容量 = 新容量最小要求/4兆 * 4兆＋4兆; 示例:当前大小3兆, 已写3兆, 继续写2兆数据, 需要的容量最小要求是5兆, 则新容量是9兆(不能超过最大值)。

## 四 : ByteBuf 的具体实现

3个维度, 8种实现

![](https://gcore.jsdelivr.net/gh/info4z/blog_images@main/images/image-20230211150118950.png) 

unsafe意味着不安全的操作。但是更底层的操作会带来性能提升和特殊功能, Netty中会尽力使用unsafe。

Java语言很重要的特性是“一次编写到处运行”, 所以它针对底层的内存或者其他操作, 做了很多封装。而unsafe提供了一系列我们操作底层的方法, 可能会导致不兼容或者不可知的异常。

* Info.仅返回一些低级的内存信息 : addressSize, pageSize
* Objects.提供用于操作对象及其字段的方法 : allocatelnstance, objectFieldOffset
* Classes.提供用于操作类及其静态字段的方法 : staticFieldoffset, defineClass, defineAnonymousClass, ensureClassInitialized
* Synchronization.低级的同步原语 : monitorEnter, tryMonitorEnter, monitorExit, compareAndSwaplnt, putOrderedInt
* Memory.直接访问内存方法 : allocateMemory, copyMemory, freeMemory, getAddress, getln, tputlnt
* Arrays.操作数组 : arrayBaseOffset, arrayIndexScale

在使用中, 都是通过 ByteBufAllocator 分配器进行申请, 同时分配器具备有内存管理的功能

代码示例

```java
/**
 * 堆外内存的常规API
 */
public class DirectByteBufDemo {
    @Test
    public void apiTest() {
        // 1.创建一个非池化的ByteBuf, 大小为10个字节
        ByteBuf buf = Unpooled.directBuffer(10);
        System.out.println("原始ByteBuf为====================>" + buf.toString());
        // System.out.println("1.ByteBuf中的内容为===============>" + Arrays.toString(buf.array()) + "\n");

        // 2.写入一段内容
        byte[] bytes = {1, 2, 3, 4, 5};
        buf.writeBytes(bytes);
        System.out.println("写入的bytes为====================>" + Arrays.toString(bytes));
        System.out.println("写入一段内容后ByteBuf为===========>" + buf.toString());
        //System.out.println("2.ByteBuf中的内容为===============>" + Arrays.toString(buf.array()) + "\n");

        // 3.读取一段内容
        byte b1 = buf.readByte();
        byte b2 = buf.readByte();
        System.out.println("读取的bytes为====================>" + Arrays.toString(new byte[]{b1, b2}));
        System.out.println("读取一段内容后ByteBuf为===========>" + buf.toString());
       //System.out.println("3.ByteBuf中的内容为===============>" + Arrays.toString(buf.array()) + "\n");

        // 4.将读取的内容丢弃
        buf.discardReadBytes();
        System.out.println("将读取的内容丢弃后ByteBuf为========>" + buf.toString());
        //System.out.println("4.ByteBuf中的内容为===============>" + Arrays.toString(buf.array()) + "\n");

        // 5.清空读写指针
        buf.clear();
        System.out.println("将读写指针清空后ByteBuf为==========>" + buf.toString());
        //System.out.println("5.ByteBuf中的内容为===============>" + Arrays.toString(buf.array()) + "\n");

        // 6.再次写入一段内容, 比第一段内容少
        byte[] bytes2 = {1, 2, 3};
        buf.writeBytes(bytes2);
        System.out.println("写入的bytes为====================>" + Arrays.toString(bytes2));
        System.out.println("写入一段内容后ByteBuf为===========>" + buf.toString());
       // System.out.println("6.ByteBuf中的内容为===============>" + Arrays.toString(buf.array()) + "\n");

        // 7.将ByteBuf清零
        buf.setZero(0, buf.capacity());
        System.out.println("将内容清零后ByteBuf为==============>" + buf.toString());
       // System.out.println("7.ByteBuf中的内容为================>" + Arrays.toString(buf.array()) + "\n");

        // 8.再次写入一段超过容量的内容
        byte[] bytes3 = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11};
        buf.writeBytes(bytes3);
        System.out.println("写入的bytes为====================>" + Arrays.toString(bytes3));
        System.out.println("写入一段内容后ByteBuf为===========>" + buf.toString());
       // System.out.println("8.ByteBuf中的内容为===============>" + Arrays.toString(buf.array()) + "\n");
    }

}
```

## 五 : PooledByteBuf 对象、内存复用

**PoolThreadCache :** PooledByteBufAllocator 实例维护的一个线程变量。

多种分类的 MemoryRegionCache **数组**用作内存缓存，MemoryRegionCache内部是链表，队列里面存Chunk

PoolChunk里面维护了内存引用，内存复用的做法就是把 buf 的 memory 指向 chunk 的 memory

PooledByteBufAllocator.ioBuffer**运作过程**梳理

![](https://gcore.jsdelivr.net/gh/info4z/blog_images@main/images/image-20230211154920014.png) 

## 六 : 零拷贝机制

Netty的零拷贝机制，是一种应用层的实现。和底层JVM、操作系统内存机制并无过多关联。

CompositeByteBuf，将多个ByteBuf合并为一个逻辑上的ByteBuf，避免了各个ByteBuf之间的拷贝

![](https://gcore.jsdelivr.net/gh/info4z/blog_images@main/images/image-20230211155954409.png)  

```java
CompositeByteBuf compositeByteBuf = Unpooled.compositeBuffer();
ByteBuf newBuffer = compositeByteBuf.addComponents(true, buffer1, buffer2);
```

wrapedBuffer()方法，将byte[]数组包装成ByteBuf对象。

![](https://gcore.jsdelivr.net/gh/info4z/blog_images@main/images/image-20230211155705843.png) 

```java
ByteBuf newBuffer = Unpooled.wrappedBuffer(new byte[]{1,2,3,4,5]);
```

slice()方法。将一个ByteBuf对象切分成多个ByteBuf对象。

![](https://gcore.jsdelivr.net/gh/info4z/blog_images@main/images/image-20230211155745627.png) 

```java
ByteBuf buffer1 = Unpooled.wrappedBuffer("hello".getBytes());ByteBuf newBuffer = buffer1.slice(1,2);
```

代码示例

```java
/**
 * 零拷贝
 */
public class ZeroCopyTest {
    @org.junit.Test
    public void wrapTest() {
        byte[] arr = {1, 2, 3, 4, 5};
        ByteBuf byteBuf = Unpooled.wrappedBuffer(arr);
        System.out.println(byteBuf.getByte(4));
        arr[4] = 6;
        System.out.println(byteBuf.getByte(4));
    }

    @org.junit.Test
    public void sliceTest() {
        ByteBuf buffer1 = Unpooled.wrappedBuffer("hello".getBytes());
        ByteBuf newBuffer = buffer1.slice(1, 2);
        newBuffer.unwrap();
        System.out.println(newBuffer.toString());
    }

    @org.junit.Test
    public void compositeTest() {
        ByteBuf buffer1 = Unpooled.buffer(3);
        buffer1.writeByte(1);
        ByteBuf buffer2 = Unpooled.buffer(3);
        buffer2.writeByte(4);
        CompositeByteBuf compositeByteBuf = Unpooled.compositeBuffer();
        CompositeByteBuf newBuffer = compositeByteBuf.addComponents(true, buffer1, buffer2);
        System.out.println(newBuffer);
    }
}
```

