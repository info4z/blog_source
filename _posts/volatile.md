---
title: 使用volatile解决可见性问题及阻止指令重排序
date: 2020-03-27
categories:
- 高性能编程
tags: 
- 多线程并发编程
- 线程安全
---





> 使用 volatile 解决可见性问题及阻止指令重排序, 了解 volatile 在 jmm 中的具体实现



## 一 : 线程操作的定义

操作定义

- write : 要写的变量以及要写的值
- read : 要读的变量以及可见的写入值(由此, 我们可以确定可见的值)
- lock : 要锁定的管程(监视器 monitor)
- unlock : 要解锁的管程
- 外部操作(socket等等...)
- 启动和终止

程序顺序 : 如果一个程序没有数据竞争, 那么程序的所有执行看起来都是顺序一致的

## 二 : 对于同步的规则定义

1. 对于监视器 m 解锁与所有后续操作对于 m 的加锁同步
2. 对 volatile 变量 v 的写入, 与所有其他线程后续对 v 的读同步
3. 启动线程的操作与线程中的第一个操作同步
4. 对于每个属性写入默认值(0, false, null)与每个线程对其进行的操作同步
5. 线程 T1 的最后操作与线程 T2 发现线程 T1 已经结束同步(isAlive, join可以判断线程是否终结)
6. 如果线程 T1 中断了 T2, 那么线程 T1 的中断操作与其他所有线程发现 T2 被中断了同步; 通过抛出 InterruptedException 异常, 或者调用 Thread.interrupted 或 Thread.isInterrupted

## 三 : happens-before 先行发生原则

**happens-before 关系**主要用于强调两个有冲突的动作之间的顺序, 以及定义数据争用的发生时机

**具体的虚拟机实现**, 有必要确保以下原则的成立 : 

- 某个线程中的每个动作都 happens-before 该线程中该动作后面的动作
- 某个管程上的 unlock 动作 happens-before 同一个管程上后续的 lock 动作
- 对某个 volatile 字段的写操作 happens-before 每个后续对该 volatile 字段的读操作
- 在某个线程对象上调用 start() 方法 happens-before 该启动了线程中的任意动作
- 某个线程中的所有动作 happens-before 任意其他线程成功从该线程对象上的 join() 中返回
- 如果某个动作 a happens-before 动作 b, 且 b happens-before 动作 c, 则有 a happens-before c

当程序包含两个没有被 happens-before 关系排序的冲突访问时, 就称存在**数据争用**; **遵守了这个原则, 也就意味着有些代码不能进行重排序, 有些数据不能缓存**

## 四 : 使用 volatile 解决可见性问题及阻止指令重排序

**可见性问题 :** 让一个线程对共享变量的修改, 能够及时的被其他先成功看到

根据 jmm(Java Memory Model) 中规定的 happens-before 和同步原则

- 对某个 volatile 字段的写操作 happens-before 每个后续对该 volitile 字段的读操作
- 对 volatile 变量 v 的写入, 与所有其他线程后续对 v 的读同步

要满足这些条件, 所以 volatile 关键字就有这些功能

1. 禁止缓存 : volatile 变量的访问控制符会加 `ACC_VOLATILE`
2. 对 volatile 变量相关的指令不做重排序

## 五 : final 在 JMM 中的处理

final 在对象的构造函数中设置对象的字段, 当线程看到该对象时, 将始终看到该对象的 final 字段的正确构造版本; 伪代码实例 : 

```java
public class FinalDemo {
    final int x;
    int y;
    static FinalDemo f;
}

f = new finalDemo(); //读取到的 f.x 一定最新, x 为 final 字段
```

如果在构造函数中设置字段后发生读取, 则会看到该 final 字段分配的值, 否则它将看到默认值; 伪代码示例:

```java
public finalDemo(){
    x = 1;
    y = x; // y等于1
}
```

读取该共享对象的 final 成员变量之前, 先要读取共享对象; 伪代码示例:

```java
r = new ReferenceObj(); 
k = r.f; 
// 这两个操作不能重排序
```

通常 static final 是不可以修改的字段; 然而 System.in, System.out 和 System.err 是 static final 字段, 遗留原因, 必须允许通过 set 方法改变, 我们将这些字段称为写保护, 以区别与普通 final 字段

## 六 : word tearing 字节处理

一个字段或元素的更新不得与任务其他字段或元素的读取或者更新交互; 特别是, 分别更新字节数组的相邻元素的两个线程不得干涉或交互, 也不需要同步以确保顺序一致性

有些处理器(尤其是早期的 Alphas 处理器)没有提供写单个字节的功能; 在这样的处理器上更新 byte 数组, 若只是简单的读取整个额呢绒, 更新对应的字节, 然后将整个内容再写回内存, 将是不合法的

这个问题有时候被称为**"字分裂(word tearing)"**, 在单独更新单个字节有难度的处理器上, 就需要寻求其他方式了

基本不需要考虑这个, 了解就好

## 七 : double 和 long 的特殊处理

虚拟机规范中, 写64位的 double 和 long 分成了两次32位值的操作; 由于不是原子操作, 可能导致读取到某次写操作中 64 位的前 32 位, 以及另外一次写操作的后 32 位

**读写 volatile 的 long 和 double 总是原子的; 读写引用也总是原子的**

商业 jvm 不会存在这个问题, 虽然规范没有要求实现原子性, 但是考虑到实际应用, 大部分实现了原子性