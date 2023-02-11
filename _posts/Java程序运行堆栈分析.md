---
title: Java程序运行堆栈分析
date: 2020-01-03
categories:
- 高性能编程
tags: 
- 多线程并发编程
- java基础
---



> java 代码写起来很简单, 但具体是怎么运行的呢?



## 一 : 概述

jvm 运行时数据区大致可以分为两部分 : <u>线程共享</u>部分和<u>线程独占</u>部分

<img src="https://gcore.jsdelivr.net/gh/info4z/blog_images@main/images/0c33cb56-bffa-4b31-b8e1-df15e642f152-8352070.jpg" alt="image"  />

**线程共享** : 所有线程能访问这块内存数据, 随虚拟机或者 `GC` 而创建和销毁; **方法区**和**堆内存**皆属此列

**线程独占** : 每个线程都会有它的独立的空间, 随线程生命周期而创建和销毁; **虚拟机栈**, **本地方法栈**和**程序计数器**属于线程独占



## 二 : JVM运行时数据区

### (一) 方法区

jvm 启动时创建, 用来存储加载<u>类信息, 常量, 静态变量, 编译后的代码</u>等数据; 

![img](https://gcore.jsdelivr.net/gh/info4z/blog_images@main/images/7e154695-b8a0-468b-a587-9f308453d42e-8352070.jpg)

虚拟机规范中这是一个逻辑区划, 具体实现根据不同虚拟机来实现; 例如: 

* oracle 的 HotSpot 在 java7 中方法区放在**永久代**; 
* java8 放在**元数据空间** (metaspace), 并且通过 `GC` 机制对这个区域进行管理。

### (二) 堆内存

jvm 启动时创建, 存放<u>对象的实例</u>; **垃圾回收器主要就是管理堆内存;** 

![image](https://gcore.jsdelivr.net/gh/info4z/blog_images@main/images/7c0219ca-003b-40b2-9391-088efcc29ad6-8352070.jpg)

如果满了, 就会出现 `OutOfMemroyError`; 

堆内存还可以细分为 : **新生代**(`Eden`, `From Survivor(s0)` 和 `To Survivor(s1)`)和**老年代** (`Old`)

### (三) 虚拟机栈

随线程的生命周期创建和销毁, 每个线程都在在这个空间有一个私有的空间, 这个空间称为线程栈; 

![image](https://gcore.jsdelivr.net/gh/info4z/blog_images@main/images/bf4e950f-0afd-4be0-9215-8eadf2dbc0fe-8352070.jpg)

线程栈由多个<u>栈帧 (Stack Frame)</u> 组成; 栈帧内容包含 : 局部变量表, 操作数栈, 动态链接, 方法返回地址和附加信息等; 

**一个线程会执行一个或多个方法, 一个方法对应一个栈帧**; 

栈内存默认最大是 **1M**, 超出则抛出 `StackOverflowError`

### (四) 本地方法栈

和虚拟机栈功能类似, 虚拟机栈是为虚拟机执行java方法而准备的, 本地方法栈是为虚拟机使用 `Native` 本地方法而准备的;

![img](https://gcore.jsdelivr.net/gh/info4z/blog_images@main/images/907d972d-624b-4234-ad09-d3aa1488cb1e-8352070.jpg)

虚拟机规范没有规定具体的实现, 由不同的虚拟机厂商去实现;

HotSpot 虚拟机中虚拟机栈和本地方法栈的实现是一样的; 同样, 超出大小以后也会抛出 `StackOverflowError`

### (五) 程序计数器

程序计数器 (Program Counter Register) 记录当前线程执行字节码的位置, 存储的是<u>字节码指令地址</u>, 

![img](https://gcore.jsdelivr.net/gh/info4z/blog_images@main/images/68e21cbe-9300-41aa-b2b0-056e069b1a3e-8352070.jpg)

如果执行Native方法, 则计数器值为空;

**每个线程都在这个空间有一个私有的空间, 占用内存空间很少;**

CPU 同一时间, 只会执行一条线程中的指令; `jvm` 多线程会轮流切换并分配 `CPU` 执行时间; 线程切换后, 需要通过程序计数器, 来恢复正确的执行位置

## 三 : class文件内容

### (一) 概述

class 文件包含 java 程序执行的字节码; 数据严格按照格式紧凑排列在class文件中的二进制流,中间无任何分隔符; 文件开头有一个`ca fe ba be` (16进制)特殊的一个标志; 

![img](https://gcore.jsdelivr.net/gh/info4z/blog_images@main/images/5292090a-3c9a-49b4-b75d-6193f27cc906-8352070.jpg) 

这个文件具有复杂且严格的格式, 专门给 jvm 读取其中的内容, 人类可以借助工具查看; 其中包含 : 版本信息, 访问标志, 常量池, 当前类, 超级类, 接口, 字段, 方法, 属性等信息

### (二) 内容分析

* 示例代码

  ```java
  public class Demo {
      public static void main(String[] args) {
          int x = 500;
          int y = 100;
          int a = x / y;
          int b = 50;
          System.out.println(a + b);
      }
  }
  ```

* 反编译 class 文件并重定向到 txt

  ```shell
  # 编译命令
  $ javac Demo.java
  # 反编译将其写入txt文件
  $ javap -v Demo.class > Demo.txt
  ```

* 版本号/访问控制, 版本号规则 : JDK5,6,7,8 分别对应 49,50,51,52

  ```
  public class Demo
    minor version: 0 //次版本号
    major version: 52 //主版本号
    flags: ACC_PUBLIC, ACC_SUPER //访问标志
  ```

* 常量池

  ```
  Constant pool:
     #1 = Methodref          #5.#14         // java/lang/Object."<init>":()V
     #2 = Fieldref           #15.#16        // java/lang/System.out:Ljava/io/PrintStream;
     #3 = Methodref          #17.#18        // java/io/PrintStream.println:(I)V
     #4 = Class              #19            // Demo1
     #5 = Class              #20            // java/lang/Object
     #6 = Utf8               <init>
     #7 = Utf8               ()V
     #8 = Utf8               Code
     #9 = Utf8               LineNumberTable
    #10 = Utf8               main
    #11 = Utf8               ([Ljava/lang/String;)V
    #12 = Utf8               SourceFile
    #13 = Utf8               Demo1.java
    #14 = NameAndType        #6:#7          // "<init>":()V
    #15 = Class              #21            // java/lang/System
    #16 = NameAndType        #22:#23        // out:Ljava/io/PrintStream;
    #17 = Class              #24            // java/io/PrintStream
    #18 = NameAndType        #25:#26        // println:(I)V
    #19 = Utf8               Demo1
    #20 = Utf8               java/lang/Object
    #21 = Utf8               java/lang/System
    #22 = Utf8               out
    #23 = Utf8               Ljava/io/PrintStream;
    #24 = Utf8               java/io/PrintStream
    #25 = Utf8               println
    #26 = Utf8               (I)V
  ```

* 构造方法 : 示例中并没有写构造函数, 由此可见, 没有定义构造函数时, 会有隐式的无参构造函数

  ```
  public Demo();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
        0: aload_0
        1: invokespecial #1                  // Method java/lang/Object."<init>":()V
        4: return
      LineNumberTable:
        line 1: 0
  ```

* 程序入口 main 方法 : stack(操作数栈), locals(为本地变量表)

  ```
  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC	//访问控制
    Code:
      stack=3, locals=5, args_size=1
        /**
         * jvm执行引擎去执行这些源码编译过后的指令码
         * javap编译出来是操作符,class文件内存的是指令码
         * 前面的数字,是偏移量(字节),jvm根据这个去区分不同的指令(查看jvm指令码表)
         */
         0: sipush        500
         3: istore_1
         4: bipush        100
         6: istore_2
         7: iload_1
         8: iload_2
         9: idiv
        10: istore_3
        11: bipush        50
        13: istore        4
        15: getstatic     #2        // Field java/lang/System.out:Ljava/io/PrintStream;
        18: iload_3
        19: iload         4
        21: iadd
        22: invokevirtual #3        // Method java/io/PrintStream.println:(I)V
        25: return
      LineNumberTable:
        line 3: 0
        line 4: 4
        line 5: 7
        line 6: 11
        line 7: 15
        line 8: 25
  ```

## 四 : 程序完整运行分析

### (一) 加载信息到方法区

此时属于线程共享部分的**方法区**会存在大量的类信息, 同时还存在运行时常量池字符串常量。

HotSpot 虚拟机 : 1.7及之前称为永久代, 1.8开始称为元数据空间。

### (二) jvm 创建线程来执行代码

在虚拟机栈, 程序计数器内存区域中创建线程独占的空间。

虚拟机栈中存放**Thread栈帧**, 程序计数器中存放**Thread执行位置**(字节码指令地址)。

### (三) 方法区程序入口

main 方法栈帧初始化 : 5个本地变量, 变量0是方法参数 args

![](https://gcore.jsdelivr.net/gh/info4z/blog_images@main/execute/execute_00.png) 

### (四) 程序执行过程

1. 将500压入操作数栈

   ![](https://gcore.jsdelivr.net/gh/info4z/blog_images@main/execute/execute_01.png)

2. 弹出操作数栈栈顶500保存到本地变量表1

   ![](https://gcore.jsdelivr.net/gh/info4z/blog_images@main/execute/execute_02.png)

3. 将100压入操作数栈

   ![](https://gcore.jsdelivr.net/gh/info4z/blog_images@main/execute/execute_03.png)

4. 弹出操作数栈栈顶100保存到本地变量表2

   ![](https://gcore.jsdelivr.net/gh/info4z/blog_images@main/execute/execute_04.png)

5. 读取本地变量1压入操作数栈

   ![](https://gcore.jsdelivr.net/gh/info4z/blog_images@main/execute/execute_05.png)

6. 读取本地变量2压入操作数栈

   ![](https://gcore.jsdelivr.net/gh/info4z/blog_images@main/execute/execute_06.png)

7. 将栈顶两int类型数相除, 结果入栈 500/100=5

   ![](https://gcore.jsdelivr.net/gh/info4z/blog_images@main/execute/execute_07.png)

8. 将栈顶int类型值保存到局部变量3中

   ![](https://gcore.jsdelivr.net/gh/info4z/blog_images@main/execute/execute_08.png)

9. 将50压入操作数栈

   ![](https://gcore.jsdelivr.net/gh/info4z/blog_images@main/execute/execute_09.png)

10. 将栈顶int类型值保存到局部变量4中

    ![](https://gcore.jsdelivr.net/gh/info4z/blog_images@main/execute/execute_10.png)

11. 获取类或接口字段的值并将其压入操作数栈

    ![](https://gcore.jsdelivr.net/gh/info4z/blog_images@main/execute/execute_11.png)

12. 将本地变量3取出压入操作数栈

    ![](https://gcore.jsdelivr.net/gh/info4z/blog_images@main/execute/execute_12.png)

13. 将本地变量4取出压入操作数栈

    ![](https://gcore.jsdelivr.net/gh/info4z/blog_images@main/execute/execute_13.png)

14. 将栈顶两int类型数相加, 结果入栈

    ![](https://gcore.jsdelivr.net/gh/info4z/blog_images@main/execute/execute_14.png)

15. 调用静态方法; jvm会根据这个方法的描述,创建新栈帧, 方法的参数从操作数栈中弹出来,压入虚拟机栈, 然后虚拟机会开始执行虚拟机栈最上面的栈帧; 执行完毕后,再继续执行main方法对应的栈帧

    ![](https://gcore.jsdelivr.net/gh/info4z/blog_images@main/execute/execute_15.png)

16. void函数返回, main方法执行结束

    ![](https://gcore.jsdelivr.net/gh/info4z/blog_images@main/execute/execute_16.png)

