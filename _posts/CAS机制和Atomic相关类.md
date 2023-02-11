---
title: CAS机制和Atomic相关类
date: 2020-04-10
categories:
- 高性能编程
tags: 
- 多线程并发编程
- 线程安全
---



> CAS 虽然可以保证原子性操作, 但同时也存在其他的问题, 并非十全十美, 用的时候要结合具体情况 



## 一 : CAS 机制

Compare and swap 比较和交换; 属于硬件同步原语, 处理器提供了基本内存操作的原子性保证

CAS 操作需要输入两个数值, 一个旧值 A (期望操作前的值)和一个新值 B, 在操作期间先比较下旧值有没有发生变化, 如果没有发生变化, 才交换成新值, 发生了变化则不交换

![](https://gcore.jsdelivr.net/gh/info4z/blog_images@main/images/image-20230116002157950.png) 

JAVA 中的 sun.misc.Unsafe 类, 提供了 `compareAndSwapInt()` 和 `compareAndSwapLong()` 等几个方法实现 CAS

## 二 : JUC包内的原子操作封装类

数值
- AtomicBoolean : 原子更新布尔类型
- AtomicInteger : 原子更新整型
- AtomicLong : 原子更新长整型

数组
- AtomicIntegerArray : 原子更新整型数组数组里的元素
- AtomicLongArray : 原子更新长整型数组里的元素
- AtomicReferenceArray : 原子更新引用类型数组里的元素

字段
- AtomicIntegerFieldUpdate : 原子更新整形的字段的更新器
- AtomicLongFieldUpdate : 原子更新长整型字段的更新器
- AtomicReferenceFieldUpdate : 原子更新引用类型里的字段

引用类型
- AtomicReference : 原子更新引用类型
- AtomicStampedReference : 原子更新带有版本号的引用类型
- AtomicMarkableReference : 原子更新带有标记为的引用类型

jdk1.8 更新
- 更新器 : DoubleAccumulator, LongAccumulator
- 计数器 : DoubleAdder, LongAdder
- 计数器增强版, 高并发下性能更好
- 频繁更新但不太频繁读取的汇总统计信息时使用
- 分成多个操作单元, 不同线程更新不同的单元
- 只有需要汇总的时候才计算所有单元的操作

## 三 : CAS 的三个问题

1. 循环 + CAS, 自旋的实现让所有线程都处于高频运行, 争抢 CPU 执行事件的状态; 如果操作长时间不成功, 会带来很大的 CPU 资源消耗
2. 仅针对单个变量的操作, 不能用于多个变量来实现原子操作
3. ABA 问题 : 简单来说就是 value=A, 线程1将A改成B, 线程2又将B改成A, 这是线程3将A改成C, 这时线程3是不知道 value 的值曾经有过变动的