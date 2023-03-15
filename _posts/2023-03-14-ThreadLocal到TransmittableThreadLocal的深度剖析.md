---
title: ThreadLocal到TransmittableThreadLocal的深度剖析
author: Travis <Hongxu Wei>
date: 2023-03-14 14:25:00 +0800
categories: [Java Learning Space]
tags: [ThreadLocal]
math: false
---

## 1、ThreadLocal，InheritableThreadLocal，TransmittableThreadLocal简述

当需要变量只生效于线程单位时，使用 TheadLocal 能很好的把变量在不同线程中隔离开来。浅浅分析一下 ThreadLocal --> InheritableThreadLocal --> TransmittableThreadLocal，这三者是依次推进的关系。`InheritableThreadLocal`对`ThreadLocal`做了拓展，`TransmittableThreadLocal`对`InheritableThreadLocal`做了拓展

<img src="https://travisnotes.oss-cn-shanghai.aliyuncs.com/mdpic/202303151448002.png" alt="image-20230314201409023" style="zoom: 25%;" />

## 2、ThreadLocal

在主线程设置了值，然后子线程去获取值的时没有获取到对应的内容，说明主线程上设置的值在子线程看不到。然后我们再在子线程设置了其对应的值，主线程去查看时也只是看到了自己设置的值，没有看到子线程设置的值。这样我们就可以判断，对于主线程和子线程，他们各自设置的值，都不会影响到对方，也就是 ThreadLocal 做到了变量对线程隔离。

==总结==

不同的线程上拥有自己的变量来保存这个 `ThreadLocal` 值，所以他能够做到对于线程隔离的。


**如何解决ThreadLocal的内存泄漏问题（内存泄漏问题参考链接）**

```java
private static final ThreadLocal<String> CONTEXT = new ThreadLocal<>()
```

## 3、InheritableThreadLocal

ThreadLocal在对于父子线程的情况下，无能为力，仍然是当作两个单独的线程进行处理。InheritableThreadLocal就是解决父子线程问题的，**即子线程能够取到父线程的值。**

## 4、TransmittableThreadLocal

TransmittableThreadLocal的存在是因为我们现在一般都不是直接创建线程了，而是直接通过**线程池**，那么我们在调用线程池中的线程时，也需要将主线程中的数据传递到这个子线程中，处理方式就是在TransmittableThreadLocal中。

TransmittableThreadLocal能够保证在线程池运行程序时获取到主线程设置的值。

[参考链接]: https://blog.csdn.net/weixin_36488231/article/details/123768881

