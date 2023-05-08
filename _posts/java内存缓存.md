---
title: java内存缓存
excerpt: 如果是单体服务其实也不是非用redis不可,毕竟搭个缓存服务器也要钱...
date: 2021-04-02
categories: 中间件
tags: [缓存中间件, java内存缓存]
---



## 一 : 缓存介绍

在计算中, 缓存是一个高速数据存储层, 其中存储了数据子集, 且通常是短暂性存储, 这样日后再次请求此数据时, 速度要比访问数据的主存储位置快。通过缓存, 您可以高效地重用之前检索或计算的数据。

**为什么要用缓存?**

* 提升应用程序性能 : 内存比磁盘性能高很多
* 降低数据库成本 : 以8c16g为例, mysql 可以支持 2w/s 的读取速度, 而使用缓存则可以支持 20w/s 甚至是 200w/s
* 减少后端负载 : 有效减少 Connection 的数量
* 可预测的性能
* 消除数据库热点 : 数据库中经常被访问的数据
* 提高读取吞吐量(IOPS) : 无论是数据库还是程序的吞吐量都会有明显提高



## 二 : java自研缓存

在Java应用中, 对于访问频率高, 更新少的数据, 通常的方案是将这类数据加入缓存中。相对从数据库中读取来说, 读缓存效率会有很大提升。

在集群环境下, 常用的分布式缓存有 Redis、Memcached 等。但在某些业务场景上, 可能不需要去搭建一套复杂的分布式缓存系统, 在单机环境下, 通常是会希望使用内部的缓存 (LocalCache) 。

**实现方案 :**

* 基于 JSR107 规范自研
* 基于 ConcurrentHashMap 实现数据缓存

代码示例 : 实现一个简单的缓存功能

```java
/**
 * 1.使用ConcurrentHashMap满足线程安全的要求
 * 2.使用SoftReference<Object>作为映射值,软引用可以保证在抛出OutOfMemory之前,如果缺少内存,将删除引用的对象
 * 3.设置缓存清理机制:定时清理
 */
public class CacheUtils {
    // 定义缓存容器
    private static final ConcurrentHashMap<String, SoftReference<CacheObject>> cache = new ConcurrentHashMap<>();
    // 每5秒扫描一次并清理过期的对象
    private static final int CLEAN_UP_PERIOD_IN_SEC = 5;
    static {
        Thread cleanerThread = new Thread(() -> {
            while (!Thread.currentThread().isInterrupted()) {
                try {
                    Thread.sleep(CLEAN_UP_PERIOD_IN_SEC * 1000);
                    cache.entrySet().removeIf(entry -> Optional.ofNullable(entry.getValue())
                                              .map(SoftReference::get)
                                              .map(CacheObject::isExpired)
                                              .orElse(false)
                                             );
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            }
        });
        cleanerThread.setDaemon(true);
        cleanerThread.start();
    }
    // 定义缓存对象,设置过期时间
    @Getter
    private static class CacheObject {
        private Object value;
        private long expiryTime;

        private CacheObject(Object value, long expiryTime) {
            this.value = value;
            this.expiryTime = expiryTime;
        }

        boolean isExpired() {
            return System.currentTimeMillis() > expiryTime;
        }
    }

    public static void add(String key, Object value, long periodInMillis) {
        if (key == null) return;
        if (value == null) {
            cache.remove(key);
        } else {
            long expiryTime = System.currentTimeMillis() + periodInMillis;
            cache.put(key, new SoftReference<>(new CacheObject(value, expiryTime)));
        }
    }

    public static void remove(String key) {
        cache.remove(key);
    }

    public static Object get(String key) {
        return Optional.ofNullable(cache.get(key))
            .map(SoftReference::get)
            .filter(cacheObject -> !cacheObject.isExpired())
            .map(CacheObject::getValue)
            .orElse(null);
    }

    public static void clear() {
        cache.clear();
    }

    public static long size() {
        return cache.entrySet().stream()
            .filter(entry -> Optional.ofNullable(entry.getValue())
                    .map(SoftReference::get)
                    .map(cacheObject -> !cacheObject.isExpired())
                    .orElse(false)
                   )
            .count();
    }

}
```



## 三 : guava缓存(推荐)

Guava Cache 是 google guava 中的一个内存缓存模块, 用于将数据缓存到JVM内存中。

实际项目开发中经常将一些公共或者常用的数据缓存起来方便快速访问, 例如 : 

* 愿意消耗一些内存空间来提升速度
* 预料到某些键会被查询一次以上
* 缓存中存放的数据总量不会超出内存容量

Guava Cache 是**单个应用运行时的本地缓存**; 不会把数据存放到文件或外部服务器, 分布式系统建议使用 Memcached、Redis 这类工具。

maven 引用

```xml
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>30.1.1-jre</version>
</dependency>
```

代码示例

```java
public class GuavaCacheUtils {
    
    public static LoadingCache<String, Object> getInstance(){
        //使用CacheBuilder构建即可
        return CacheBuilder.newBuilder()
            //设置并发级别为8,并发级别是指可以同时写缓存的线程数
            .concurrencyLevel(8)
            //设置写缓存后8秒钟过期
            .expireAfterAccess(8, TimeUnit.SECONDS)
            //设置写缓存后1秒钟刷新
            .refreshAfterWrite(1, TimeUnit.SECONDS)
            //设置缓存容器的初始容量为10
            .initialCapacity(10)
            //设置缓存最大容量为100,超过100之后就会按照LRU最近虽少使用算法来移除缓存项
            .maximumSize(100)
            //设置要统计缓存的命中率
            .recordStats()
            //设置缓存的移除通知
            .removalListener(new RemovalListener<Object, Object>() {
                @Override
                public void onRemoval(RemovalNotification<Object, Object> notification) {
                    System.out.println(notification.getKey() + " 被移除了，原因： " + notification.getCause());
                }
            })
            //build方法中可以指定CacheLoader,在缓存不存在时通过CacheLoader的实现自动加载缓存
            .build(new CacheLoader<String, Object>() {
                @Override
                public Object load(String key) throws Exception {
                    System.out.println("缓存没有时，从数据库加载" + key);
                    //省略JDBC过程
                    return "Tom-" + key;
                }
            });
    }
}
```

单元测试

```java
public class GuavaCacheTest {

    @Test
    public void test() throws ExecutionException {
        
        LoadingCache<String, Object> cache = GuavaCacheUtils.getInstance();

        // 第一次
        for (int i = 1; i <= 5; i++) {
            System.out.println(i + ": " + cache.get(String.valueOf(i)));
        }
        // 第二次
        for (int i = 1; i <= 5; i++) {
            System.out.println(i + ": " + cache.get(String.valueOf(i)));
        }
        // 打印缓存命中率
        System.out.println("cache stats: " + cache.stats().toString());
    }
}
```



