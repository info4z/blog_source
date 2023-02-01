---
title: Lock接口和ReentrantLock/ReadWriteLock
date: 2020-05-08
categories:
- 高性能编程
tags: 
- 多线程并发编程
- 线程安全
---



> JAVA 还提供了锁相关的一些API, 可以更加灵活的实现控制锁的获取和释放



## 一 : Lock 的核心API

synchronized 关键字使用固然简单, 但是我们**无法控制同步代码停止**, 除非抛异常; 



Lock 提供了很丰富的 API : 

| 方法              | 描述                                           |
| ----------------- | ---------------------------------------------- |
| lock              | 获取锁的方法, 若锁被其他线程获取, 则等待(阻塞) |
| lockInterruptibly | 在锁的获取过程中可以中断当前线程               |
| tryLock           | 尝试非阻塞的获取锁, 立即返回                   |
| unlock            | 释放锁                                         |

## 二 : ReentrantLock

**独享锁; 可重入锁**; 支持公平锁, 非公平锁两种模式

```java
public class Demo {
    
    Lock lock = new ReentrantLock();

    public void a() {
        /*
         * 如果需要正确中断等待锁的线程,必须将获取锁放在try{}外面;
         * 否则会抛出 IllegalMonitorStateException,
         * 由于finally却无论如何都要执行,而等待线程根本就没有拿到锁,也就是锁没有锁定监视器
         */
        lock.lock();
        try {
            // ...method body
            b(); // 此方法带锁, 同一线程可以重复进入
        } finally { // 这里必须用finally将锁释放掉,否则一旦出现异常容易出现死锁
            lock.unlock(); 
        }
    }

    public void b() {
        lock.lock();
        //...
        lock.unlock();
    }
}
```

**执行过程 :**

- 初始化 : 未锁, ReentrantLock, 持有者(null), 计数(0)
- 调用 a() : 第一次锁, ReentrantLock, 持有者(线程-0), 计数(1) 
- a() 中又调用 b() : 第二次锁, ReentrantLock, 持有者(线程-0), 计数(2)
- b() 执行完毕 : 解锁, ReentrantLock, 持有者(线程-0), 计数(1)
- a() 执行完毕 : 解锁, ReentrantLock, 持有者(null), 计数(0)

## 三 : ReadWriteLock

维护一对关联锁, 一个用于只读操作, 一个用于写入; 读锁可以由多个读线程同时持有, 写锁是排他的; 

* **适用场景 :** 适合读取线程比写入线程多的场景(读多写少), 改进互斥锁的性能
* **示例场景 :** 缓存组件, 集合的并发线程安全性改造

代码示例 : 

```java
public class Demo {
    ReentrantReadWriteLock readWriteLock = new ReentrantReadWriteLock();

    public static void main(String[] args) {
        final Demo demo = new Demo();
        // 多线程同时读/写
        new Thread(() -> {
            demo.read(Thread.currentThread());
        }).start();

        new Thread(() -> {
            demo.read(Thread.currentThread());
        }).start();

        new Thread(() -> {
            demo.write(Thread.currentThread());
        }).start();
    }

    // 多线程读,共享锁
    public void read(Thread thread) {
        readWriteLock.readLock().lock();
        try {
            long start = System.currentTimeMillis();
            while (System.currentTimeMillis() - start <= 1) {
                System.out.println(thread.getName() + "正在进行“读”操作");
            }
            System.out.println(thread.getName() + "“读”操作完毕");
        } finally {
            readWriteLock.readLock().unlock();
        }
    }

    // 写,独占锁
    public void write(Thread thread) {
        readWriteLock.writeLock().lock();
        try {
            long start = System.currentTimeMillis();
            while (System.currentTimeMillis() - start <= 1) {
                System.out.println(thread.getName() + "正在进行“写”操作");
            }
            System.out.println(thread.getName() + "“写”操作完毕");
        } finally {
            readWriteLock.writeLock().unlock();
        }
    }
}
```

**锁降级**指的是写锁降级成为读锁; 把持住当前拥有的写锁的同时, 再获取到读锁, 然后释放写锁的过程

写锁是线程独占, 读锁是共享, 所以, **写 => 读是升级**(读 => 写, 是不能实现的)

## 四 : Condition

Object 中的 wait(), notify(), notifyAll() 方法是和 synchronized 配合使用的, 可以唤醒一个或者全部(单个等待集);

Condition 是需要与 Lock 配合使用的, 提供多个等待集合, 更精确的控制; **底层是 park/unpark 机制**; 用于替代 wait/notify; 

经典场景 : JDK 中的队列实现

- 多线程读写队列 : 写入数据时, 唤醒读取线程继续执行; 读取数据后, 通知写入队列继续执行

代码示例 : 

```java
/**
 * condition 实现队列线程安全
 */ 
public class QueueDemo {
    final Lock lock = new ReentrantLock();
    // 指定条件的等待 - 等待有空位
    final Condition notFull = lock.newCondition();
    // 指定条件的等待 - 等待不为空
    final Condition notEmpty = lock.newCondition();

    // 定义数组存储数据
    final Object[] items = new Object[100];
    int putptr, takeptr, count;

    // 写入数据的线程,写入进来
    public void put(Object x) throws InterruptedException {
        lock.lock();
        try {
            while (count == items.length) // 数据写满了
                notFull.await(); // 写入数据的线程,进入阻塞
            items[putptr] = x;
            if (++putptr == items.length) putptr = 0;
            ++count;
            notEmpty.signal(); // 唤醒指定的读取线程
        } finally {
            lock.unlock();
        }
    }
    // 读取数据的线程,调用take
    public Object take() throws InterruptedException {
        lock.lock();
        try {
            while (count == 0)
                notEmpty.await(); // 线程阻塞在这里,等待被唤醒
            Object x = items[takeptr];
            if (++takeptr == items.length) takeptr = 0;
            --count;
            notFull.signal(); // 通知写入数据的线程,告诉他们取走了数据,继续写入
            return x;
        } finally {
            lock.unlock();
        }
    }
}
```

