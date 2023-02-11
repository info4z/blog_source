---
title: JAVA锁相关术语及同步关键字synchronized
date: 2020-04-17
categories:
- 高性能编程
tags: 
- 多线程并发编程
- 线程安全
---









> 要使用多线程技术, 能够合理的使用锁总是绕不开的



## 一 : java 中锁的概念

**自旋锁 :** 为了不放弃 CPU 执行事件, 循环的使用 CAS 技术对数据尝试进行更新, 直至成功

**悲观锁 :** 假定会发生并发冲突, 同步所有对数据的相关操作, 从读数据就开始上锁

**乐观锁 :** 假定没有冲突, 在修改数据时如果发现数据和之前获取的不一致, 则读最新的数据, 修改后重试修改

**独享锁(写) :** 给资源加上写锁, 线程可以修改资源, 其他线程不能再加锁;(单写)

**共享锁(读) :** 给资源加上读锁后只能读不能改, 其他线程也只能加读锁, 不能加写锁;(多读)

**可重入锁, 不可重入锁 :** 线程拿到一把锁之后, 可以自由进入同一把锁所同步的其他代码

**公平锁, 非公平锁 :** 争抢锁的顺序, 如果是按先来后到, 则为公平

几种重要的锁实现方式 : `synchronized`, `ReentrantLock`, `ReentrantReadWriteLock`

## 二 : 同步关键字 synchronized

属于最基本的线程通信机制, 基于**对像监视器**实现的; java中的每一个对象都与一个监视器相关联, 一个线程可以锁定或者解锁监视器; **一次只有一个线程可以锁定监视器;** 试图锁定该监视器的任何其他线程都会被阻塞, 直到它们可以获得该监视器上的锁定为止

**特性 :** 可重入, 独享, 悲观锁

**锁的范围 :** 类锁, 对象锁, 锁消除, 锁粗化

```java
public class SyncDemo {

    public synchronized /*static*/ void test() {
        try {
            System.out.println(Thread.currentThread() + " 我开始执行");
            Thread.sleep(3000L);
            System.out.println(Thread.currentThread() + " 我执行结束");
        } catch (InterruptedException e) {
            
        }
    }

    public static void main(String[] args) throws InterruptedException {
        new Thread(() -> new SyncDemo().test()).start();
        Thread.sleep(1000L); // 等1秒钟,让前一个线程启动起来
        new Thread(() -> new SyncDemo().test()).start();
    }
}
```

**运行结果 :** 线程不会同步

**原理分析 :** `synchronized` 的加锁原理是锁定**对象监视器**, 每个对象都对应各自的监视器

**解决方案 :** 

1. 加静态关键字 `static`, 提升为类锁
2. 为临界区添加同步代码块, 指定共用的锁, 但需要是类锁

**提示 :** 同步关键字, 不仅是实现同步, 根据 JMM 规定还能保证可见性(读取最新主内存数据, 结束后写入主内存)

## 三 : 同步关键字加锁原理

HotSpot 中, 对象在内存中存储的布局可以分为三块区域 : **对象头**(Header), **实例数据**(Instance Data)和**对齐填充**(Padding)

普通对象的**对象头**(Header)包括两部分 : **Mark Word** 和 **Class Metadata Address** (类型指针), 如果是数组对象还包括一个额外的 **Array length** 数组长度部分

| 长度     | 内容                       | 说明                                   |
| -------- | -------------------------- | -------------------------------------- |
| 32/64bit | **Mark Word**              | 存储对象hashCode或锁信息等运行时数据。 |
| 32/64bit | **Class Metadata Address** | 存储到对象类型数据的指针               |
| 32/64bit | **Array length**           | 数组的长度(如果当前对象是数组)         |

其中 **mark word** 用于存储对象自身的运行时数据, 如: 哈希码(HashCode), GC分代年龄(age), 是否偏向锁(0/1), 锁标志位(tag), 线程持有的锁状态(state), 偏向线程ID(Thread ID), 偏向时间戳等等(epoch); 占用内存大小与虚拟机位长一致(32/64bit)

```
|		Bitfield			|Tag|			State			|
|		HashCode	|age|0	|01	|	Unlocked				|
|	Lock record address		|00	|	Light-weight locked		|
|	Monitor address			|10	|	heavy-weight locked		|
|	Forwarding address, etc	|11	|	marked for GC			|
|	Thread ID|epoch	|age|1	|01	|	biased/biasable			|
```

**默认情况下 jvm 锁会经历 :** 偏向锁 => 轻量级锁 => 重量级锁

参考文献 : 

* https://www.cs.princeton.edu/picasso/mats/HotspotOverview.pdf 
* https://wiki.openjdk.java.net/display/HotSpot/Synchronization

### (一) 轻量级锁

对象初始化的时候是无锁的, mark word 记录的是 hashCode, age 和偏移状态

**加锁原理 :** 线程栈开辟一个空间, 存储当前锁定对象的 mark word 信息; 这个空间就是 lock record, 它可以存储多个锁定的对象信息, 如果对象被锁定了, mark word 的内容会变为 lock record address

加锁前

![](https://gcore.jsdelivr.net/gh/info4z/blog_images@main/images/image-20230120130607774.png) 

加锁后

![](https://gcore.jsdelivr.net/gh/info4z/blog_images@main/images/image-20230120130718713.png) 

使用 CAS 修改 mark word 完毕, 则 mark word 中的 tag 进入 00 状态

解锁的过程, 则是一个逆向恢复 mark word 的过程

### (二) 偏向锁到轻量级锁

偏向锁默认是开启的, **偏向锁其实也是无锁**, tag 值和无锁对象一样, 都是 01, 若出现锁竞争会升迁到轻量级锁

![](https://gcore.jsdelivr.net/gh/info4z/blog_images@main/images/image-20230120131237732.png)

未锁定或不可偏向对象经过自旋重锁之后, 会升迁为轻量级锁, mark word 的内容会修改为线程栈的 lock record address, 同时 tag 会修改为 00, 如果锁再次升级, 就会编程重量级锁, mark word 内容会修改为 monitor address , 同时 tag 会修改为 10

开启偏向锁的对象 mark word 内容为 thread ID|age|1 , tag 值仍然是 01, 这里需要注意的是 Thread ID 初始值为 0, 当第一个线程进行操作的时候, 会将 Thread ID 修改为自己的线程 ID, 一旦出现争抢会出现两种情况: 一种情况是如果对象没锁定, 则会变成未锁定且不偏向的对象, 也就是无锁对象; 如果对象已经锁定, 则会直接变成被轻量级锁定的对象

**总结 :**

- 偏向标记第一次有用, 出现过争用后就没有用了; 

  ```shell
  -XX: -UseBiasedLocking # 禁用使用偏置锁定
  ```

- 偏向锁, 本质就是无锁, 如果没有发生过任何多线程争抢锁的情况, jvm 认为就是单线程, 无需做同步(jvm 为了少干活: 同步在 jvm 底层是有很多操作来实现的, 如果是没有争用, 就不需要去做同步)

### (三) 重量级锁 - 监视器(monitor)

修改 mark word 如果失败, 会自旋 CAS 一定次数, 该次数可以通过参数配置

超过次数, 仍未抢到锁, 则锁升级为重量级锁, 进入阻塞

![](https://gcore.jsdelivr.net/gh/info4z/blog_images@main/images/image-20230120131530631.png)

monitor 也叫做管程, 计算机操作系统原理中有提及类似概念, 一个对象会有一个对应的 monitor