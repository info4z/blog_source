---
title: JVM调优
excerpt: jdk版本不断更新,JVM参数和具体说明,建议需要时参考oracle官网的手册
date: 2020-08-28
categories: 高性能编程
tags: [java系统性能调优, 性能调优]
---



## 一 : 调优基本概念

在调整性能时, JVM有三个组件 :

1. 堆大小调整
2. 垃圾收集器调整
3. JIT编译器

通常, 在调优 Java 应用程序时, 重点是以下两个主要目标之一 :

* **响应性 :** 应用程序或系统对请求的数据进行响应的速度, 对于专注于响应性的应用程序, 长的暂停时间是不可接受的, 重点是在短时间内做出回应
* **吞吐量 :** 侧重于在特定时间段内最大化应用程序的工作量, 对于专注于吞吐量的应用程序, 高暂停时间是可接受的。由于高吞吐量应用程序在较长时间内专注于基准测试, 因此不需要考虑快速响应时间

**系统瓶颈核心还是在应用代码, 一般情况下无需过多调优, JVM本身在不断优化。**

**JDK版本不断更新, JVM参数和具体说明, 建议需要时参考oracle官网的手册。**

官方调优指南 : https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/



## 二 : 常用 JVM 参数

| 参数                               | 作用                              |
| ---------------------------------- | --------------------------------- |
| -XX:+AlwaysPreTouch                | jvm启动时分配内存, 非使用时再分配 |
| -XX: ErrorFile = filename          | 崩溃日志                          |
| -XX:+TraceClassLoading             | 跟踪类加载信息                    |
| -XX:+PrintClassHistogram           | 按下Ctrl+Break后, 打印类的信息    |
| -Xmx -Xms                          | 最大堆和最小堆                    |
| -xx:permSize、-xx:metaspaceSize    | 永久代/元数据空间                 |
| -XX:+HeapDumpOnOutOfMemoryError    | OOM时导出堆到文件                 |
| -XX:+HeapDumpPath                  | OOM时堆导出的路径                 |
| -XX:OnOutOfMemoryError             | 在OOM时, 执行一个脚本             |
| iava -XX:+PrintFlagsFinal -version | 打印所有的-XX参数和默认值         |



## 三 : GC 调优思路

| 步骤 | 目标     | 描述                                             |
| ---- | -------- | ------------------------------------------------ |
| 1    | 分析场景 | 启动速度慢; 偶尔出现响应慢于平均水平或者出现卡顿 |
| 2    | 确定目标 | 内存占用、低延时、吞吐量                         |
| 3    | 收集日志 | 通过参数配置收集GC日志;通过JDK工具查看GC状态     |
| 4    | 分析日志 | 使用工具辅助分析日志, 查看GC次数, GC时间         |
| 5    | 调整参数 | 切换垃圾收集器或者调整垃圾收集器参数             |



## 四 : 通用 GC 参数

GC线程数量

| 参数                  | 描述           |
| --------------------- | -------------- |
| -XX:ParallelGCThreads | 并行GC线程数量 |
| -XX:ConcGCThreads     | 并发GC线程数量 |

GC时间 : 意义不大, JVM只能尽量满足

| 参数                 | 描述                                                         |
| -------------------- | ------------------------------------------------------------ |
| -XX:MaxGCPauseMillis | 最大停顿时间, 单位毫秒; GC尽力保证回收时间不超过设定值       |
| -XX:GCTimeRatio      | 0-100的取值范围; 垃圾收集时间占总时间的比; 默认99, 即最大允许1%时间做GC |

内存占比 : 用默认值就好

| 参数              | 描述                                                         |
| ----------------- | ------------------------------------------------------------ |
| -XX:SurvivorRatio | 新生代Eden和Survivor大小的比例; 默认8, 表示2个Survivor:Eden=2:8, 即1个Survivor占新生代的1/10 |
| -XX:NewRatio      | 新生代和老年代的比例; 默认4, 表示新生代:老年代=1:4, 即年轻代占堆的1/5 |

GC信息

| 参数                      | 描述                                 |
| ------------------------- | ------------------------------------ |
| -verbose:gc、-XX:+printGC | 打印GC的简要信息                     |
| -XX:+PrintGCDetails       | 打印GC详细信息, **jdk9已经舍弃**     |
| -XX:+PrintGCTimeStamps    | 打印CG发生的时间戳, **jdk9已经舍弃** |
| **-Xloggc:log/gc.log**    | 指定Gc日志的位置, 以文件输出         |
| -XX:+PrintHeapAtGC        | 每次一次GC后, 都打印堆信息           |



## 五 : 垃圾收集器 Parallel 参数

Parallel 是 JDK1.8 的默认收集器, 适用于**吞吐量优先**

| 参数                       | 描述                          |
| -------------------------- | ----------------------------- |
| -XX:+UseParallelGC         | 新生代使用并行回收收集器      |
| -XX:+UseParallelOldGC      | 老年代使用并行回收收集器      |
| **-XX:ParallelGCThreads**  | 设置用于垃圾回收的线程数      |
| -XX:+UseAdaptiveSizePolicy | 打开自适应GC策略,**默认开启** |



## 六 : 垃圾收集器 CMS 参数调优

**响应时间优先;** ParallelGC 无法满足应用程序延迟要求时再考虑使用 CMS 垃圾收集器; 新版建议用 G1 垃圾收集器

| 参数                                | 描述                                                         |
| ----------------------------------- | ------------------------------------------------------------ |
| -XX:+UseConcMarkSweepGC             | 新生代使用并行收集器;老年代使用CMS+串行收集器                |
| -XX:+UseParNewGC                    | 在新生代使用并行收集器;CMS下默认开启                         |
| -XX:CMSInitiatingOccupancyFraction  | 设置触发GC的阈值,默认68%;如果内存预留空间不够,就会引起concurrentmode failure |
| -XX:+ UseCMSCompactAtFullCollection | Full GC后,进行一次整理,整理过程是独占的,会引起停顿时间变长   |
| -XX:+CMSFullGCsBeforeCompaction     | 设置进行几次Full GC后,进行一次碎片整理                       |
| -XX:+CMSClassUnloadingEnabled       | 允许对类元数据进行回收                                       |
| -XX:+UseCMSInitiatingOccupancyOnly  | 表示只在到达阀值的时候,才进行CMS回收                         |
| -XX:+CMSIncrementalMode             | 使用增量模式,比较适合单 CPU                                  |



## 七 : 垃圾收集器 G1 参数调优

**兼顾吞吐量和响应时间;** 超过50%的Java堆被实时数据占用; 建议大堆(大小约为6GB更大)

GC延迟要求有限的应用(稳定且可预测的暂停时间低于0.5秒)。

| 参数                                  | 描述                                                    |
| ------------------------------------- | ------------------------------------------------------- |
| -XX:G1HeapRegionSize=<N,例如 16>M     | 设置region大小,默认heap/2000                            |
| -XX:G1MixedGCLiveThresholdPercent     | 老年代依靠Mixed GC, 触发闻值                            |
| -XX:G1OldCSetRegionThresholdPercent   | 定多被包含在一次Mixed GC中的region比例                  |
| -XX:+ClassUnloadingWithConcurrentMark | G1增加并默认开启,在并发标记阶段结束后,JVM即进行类型卸载 |
| -XX:GINewSizePercent                  | 新生代的最小比例                                        |
| -XX:G1MaxNewSizePercent               | 新生代的最大比例                                        |
| -XX:G1MixedGCCountTarget              | Mixed GC 数量控制                                       |



## 八 : 运行时 JIT 编译器优化参数

JIT编译指的是字节码编译为本地代码(汇编)执行, 只有热点代码才会编译为本地代码。

解释器执行节约内存, 反之可以使用编译执行来提升效率

| 参数                                        | 描述                                                         |
| ------------------------------------------- | ------------------------------------------------------------ |
| -XX:+AggressiveOpts                         | 允许ivm使用积极的性能优化功能                                |
| -XX:-TieredCompilation                      | 分层编译idk8默认开启, idk7默认关闭client                     |
| -Xmaxjitcodesize、-XX:ReservedCodeCacheSize | 指定JIT编译代码的最大代码高速缓存最大值                      |
| -Xmixed                                     | 除了热方法之外, 解释器执行所有字节码, 热方法被编译为本机代码 |
| -XX:lnitialCodeCacheSize                    | 初始化编译后的汇编指令, 缓存区大小, 字节                     |
| -XX:+PrintCompilation                       | 打开编译日志istat -compiler pid                              |
| -XX:CICompilerCount                         | JIT编译所使用的线程数量                                      |
| -XX:+DoEscapeAnalysis                       | 逃逸分析, 默认打开。对代码的深度优化                         |
| -XX:-Inline                                 | 方法内联, 默认打开。                                         |

**很少需要对较新版本的JVM进行JIT调优**