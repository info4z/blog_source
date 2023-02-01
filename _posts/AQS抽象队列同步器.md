---
title: AQS抽象队列同步器
date: 2020-05-15
categories:
- 高性能编程
tags: 
- 多线程并发编程
- J.U.C并发编程包
---



> AQS : abstract queue synchronizer, 抽象队列同步器



## 一 : 同步锁的本质 - 排队

同步的方式 : 独享锁 - 单个队列窗口, 共享锁 - 多个队列窗口

抢锁的方式 : 公平锁(先来后到抢锁), 非公平锁(插队抢)

没抢到锁的处理方式 : 快速尝试多次(CAS自旋), 阻塞等待

唤醒阻塞线程的方式(叫号器) : 全部通知, 通知下一个

## 二 : AQS 抽象队列同步器

提供了对资源占用, 释放; 线程的等待, 唤醒等等接口和具体实现

可以用在各种需要控制资源争用的场景中(ReentrantLock/CountDownLatch/Semahpore)

![](https://gcore.jsdelivr.net/gh/info4z/blog_images@main/images/image-20230128175042201.png)

其中定义的接口可以分为**独占资源**接口和**共享资源**接口两类; 其中独占资源接口包括 acquire, release, tryAcquire(未实现), tryRelease(未实现); 而共享资源接口则包括 acquireShared, releaseShared, tryAcquireShared(未实现), tryReleaseShared(未实现)

- acquire, acquireShared : 定义了资源争用的逻辑, 如果没拿到, 则等待
- **tryAcquire, tryAcquireShared :** 实际执行占用资源的操作, 如何判定一个由使用者具体去实现
- release, releaseShared : 定义释放资源的逻辑, 释放之后, 通知后续节点进行争抢
- **tryRelease, tryReleaseShared :** 实际执行资源释放的操作, 具体的 AQS 使用者去实现

AQS 内部主体 : state(状态), exclusiveOwner(占有者), Node(锁的等待者链表, 从head => tail)

代码示例

```java
public class AQSdemo {
    // 同步资源状态
    volatile AtomicInteger state = new AtomicInteger(0);
    // 当前锁的拥有者
    protected volatile AtomicReference<Thread> owner = new AtomicReference<>();
    // java q 线程安全
    public volatile LinkedBlockingQueue<Thread> waiters = new LinkedBlockingQueue<>();

    public void acquire() {
        boolean addQ = true;
        while (!tryAcquire()) {
            if (addQ) {
                // 塞到等待锁的集合中
                waiters.offer(Thread.currentThread());
                addQ = false;
            } else {
                // 挂起这个线程
                LockSupport.park();
                // 后续，等待其他线程释放锁，收到通知之后继续循环
            }
        }
        waiters.remove(Thread.currentThread());
    }

    public void release() {
        // cas 修改 owner 拥有者
        if (tryRelease()) {
            Iterator<Thread> iterator = waiters.iterator();
            while (iterator.hasNext()) {
                Thread waiter = iterator.next();
                LockSupport.unpark(waiter); // 唤醒线程继续 抢锁
            }
        }
    }

    // 判断量够不够
    public void acquireShared() {
        boolean addQ = true;
        while (tryAcquireShared() < 0) {
            if (addQ) {
                // 塞到等待锁的集合中
                waiters.offer(Thread.currentThread());
                addQ = false;
            } else {
                // 挂起这个线程
                LockSupport.park();
                // 后续，等待其他线程释放锁，收到通知之后继续循环
            }
        }
        waiters.remove(Thread.currentThread());
    }

    public void releaseShared() {
        // cas 修改 owner 拥有者
        if (tryReleaseShared()) {
            Iterator<Thread> iterator = waiters.iterator();
            while (iterator.hasNext()) {
                Thread waiter = iterator.next();
                LockSupport.unpark(waiter); // 唤醒线程继续 抢锁
            }
        }
    }

    public boolean tryAcquire() {
        throw new UnsupportedOperationException();
    }

    public boolean tryRelease() {
        throw new UnsupportedOperationException();
    }

    public int tryAcquireShared() {
        throw new UnsupportedOperationException();
    }

    public boolean tryReleaseShared() {
        throw new UnsupportedOperationException();
    }

    public AtomicInteger getState() {
        return state;
    }

    public void setState(AtomicInteger state) {
        this.state = state;
    }
}
```



## 三 : 资源占用流程(acquire)

![](https://gcore.jsdelivr.net/gh/info4z/blog_images@main/images/94287e08-e18f-4bac-a9c7-0abe7af2d160-8352070.jpg)

