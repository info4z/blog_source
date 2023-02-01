---
title: 线程封闭之ThreadLocal和栈封闭
date: 2020-02-21
categories:
- 高性能编程
tags: 
- 多线程并发编程
- java基础
---





> 并不是所有多线程都会用到共享数据, 如果数据都是独享的, 就可以使用线程封闭技术



## 一 : 线程封闭概念

多线程访问共享可变数据时, 涉及到线程间数据同步的问题, 并不是所有时候, 都要用到共享数据, 所以线程封闭概念就提出来了

数据都被封闭在各自的线程之中, 就不需要同步, 这种通过将数据封闭在线程中而避免使用同步的技术称为**线程封闭**

线程封闭的具体实现有: **ThreadLocal**, **局部变量**

## 二 : ThreadLocal

**ThreadLocal 是 java 里一种特殊的变量**, 它是一个**线程级别变量**, 每个线程都有一个 ThreadLocal, 就是每个线程都拥有了自己独立的一个变量, 竞争条件被彻底消除了, 在并发模式下是绝对安全的变量

**用法**: `ThreadLocal<T> var = new ThreadLocal<T>();` 会自动在每一个线程上创建一个 T 的副本, 副本之间彼此独立, 互不影响; 可以用 ThreadLocal 存储一些参数, 以便在线程中多个方法中使用, **用来代替方法传参的做法**; 

- 代码示例

  ```java
  public static ThreadLocal<String> value = new ThreadLocal<>();
  
  public void threadLocalTest() throws InterruptedException {
      value.set("123");
      String v = value.get();
      System.out.println("线程1执行之前,主线程取到的值: " + v);
  
      new Thread(()->{
          String s = value.get();
          System.out.println("线程1取到的值: " + s);
          value.set("456");
  
          s = value.get();
          System.out.println("重新设置后,线程1取到的值: " + s);
      }).start();
  
      Thread.sleep(5000L);
      v = value.get();
      System.out.println("线程1执行之后,主线程取到的值: " + v);
  }
  
  public static void main(String[] args) throws InterruptedException {
      new ThreadLocalDemo().threadLocalTest();
  }
  ```

实在难以理解的, 可以理解为, jvm 维护了一个 Map<Thread, T>, 每个线程要用这个 T 的时候, 用当前的线程去 Map 里面取, 仅作为一个概念理解

## 三 : 栈封闭

**局部变量**的固有属性之一就是封闭在线程中

它们位于执行线程的栈中, 其他线程无法访问这个栈