---
title: FutureTask源码剖析
excerpt: 通过Future了解到Callable和Runnable的内在区别
date: 2020-06-12
categories: 高性能编程
tags: [多线程并发编程, J.U.C并发编程包]
---



## 一 : Future

Future 表示异步计算的结果, 提供了用于检查计算是否完成, 等待计算完成以及获取结果的方法

![image-2020061201](../java/image-2020061201.png)

Future 和 Callable

1. Callable 和 Runnable 一样的业务定义, 但本质上有区别(返回值, 异常定义), 表面的区别并不是最重要的, 重要的是我们要看内在, **内在的一个本质上的点是 Callable 是被 Runnable 调用的, 也就是说 `call()` 运行在 `run()` 里面**

   ```java
   // Thread并不能传入Callable, 但是我们可以通过FutureTask来使用callable
   Callable<Object> callable = new Callalbe<Object>{
       @Override
       public Object call throws Exception {
           return null;
       }
   }
   FutureTask<Object> futureTask = new FutureTask<>(callable);
   new Thread(futureTask).start();
   ```

   ```java
   // 当然也可以用线程池直接使用
   ExecutorService executorService = Executors.newCachedThreadPool();
   executorService.submit(callable);
   ```

2. 在线程执行完后可以直接在 futureTask 中拿结果

   ```java
   futureTask.get();
   ```



## 二 : FutureTask 应用

![image-2020061202](../java/image-2020061202.png) 

总的执行时间, 取决于执行最慢的逻辑

逻辑之间无依赖关系, 可同时执行, 则可以应用**多线程技术进行优化**

代码示例

```java
@Service
public class UserServiceFutureTask {
    ExecutorService executorService = Executors.newCachedThreadPool();
    @Autowired
    private RestTemplate restTemplate;

    /**
     * 查询多个系统的数据，合并返回
     */
    public Object getUserInfo(String userId) throws ExecutionException, InterruptedException {

        // Future < >  Callable
        // 和runnable一样的业务定义,但是本质上是有区别的: 返回值 异常 call run.
        Callable<JSONObject> callable = new Callable<JSONObject>() {
            @Override
            public JSONObject call() throws Exception {
                // 1. 先从调用获取用户基础信息的http接口
                long userinfoTime = System.currentTimeMillis();
                String value = restTemplate.getForObject("http://www.tony.com/userinfo-api/get?userId=" + userId, String.class);
                JSONObject userInfo = JSONObject.parseObject(value);
                System.out.println("userinfo-api用户基本信息接口调用时间为" + (System.currentTimeMillis() - userinfoTime));
                return userInfo;
            }
        };

        // 通过多线程运行callable
        FutureTask<JSONObject> userInfoFutureTask = new FutureTask<>(callable);
        new Thread(userInfoFutureTask).start();

        FutureTask<JSONObject> intergralInfoTask = new FutureTask(() -> {
            // 2. 再调用获取用户积分信息的接口
            long integralApiTime = System.currentTimeMillis();
            String intergral = restTemplate.getForObject("http://www.tony.com/integral-api/get?userId=" + userId,
                    String.class);
            JSONObject intergralInfo = JSONObject.parseObject(intergral);
            System.out.println("integral-api积分接口调用时间为" + (System.currentTimeMillis() - integralApiTime));
            return intergralInfo;
        });
        new Thread(intergralInfoTask).start();

        // 3. 合并为一个json对象
        JSONObject result = new JSONObject();
        result.putAll(userInfoFutureTask.get()); // 会等待任务执行结束
        result.putAll(intergralInfoTask.get());
        return result;
    }
}
```



## 三 : 线程安全性级别

《Effective Java》- Joshua J.Block 提到, 工具类需要显示的说明它的安全性级别 :

1. **不可变的** : 这个类的实例是不可变的; 这样的例子包括 String, Long, BigInteger
2. **无条件的线程安全** : 这个类的实例是可变的, 但是这个类有组偶的内部同步; 例子包括 Random, ConcurrentHashMap
3. **有条件的线程安全** : 除了有些方法为进行安全的并发使用而需要外部同步之外, 这种线程安全级别与无条件相同; 例如包括: Collections.synchronized 包装返回的集合, 他们的迭代器要求外部同步
4. **非线程安全** : 这个类的实例是可变的; 为了并发使用它们, 客户必须利用自己选择的外部同步包围每个方法调用, 例子包括 ArrayList
5. **线程对立** : 这个类不能安全地被多个线程并发使用, 即使所有的方法调用都被外围同步包围



## 四 : JDK 学习思路汇总

积累 : 由基层知识再到封装的工具类, 足够多的 "因" 才能推理出 "果"; 基层不代表基础, 不代表简单;

思路 : 从顶层看使用, 从底层看原理

结语 : 多线程编程中, 不变的是**内存模型**和**线程通信**两个核心技术点, 变化的是各种程序设计想法(算法)