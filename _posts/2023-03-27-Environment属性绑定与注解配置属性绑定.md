---
title: Environment属性绑定与注解配置属性绑定
author: Travis <Hongxu Wei>
date: 2023-03-27 14:45:48 +0800
categories: [Java Learning Space]
tags: [Spring]
math: false
---

- **示例一：注解配置属性绑定**

<img src="https://travisnotes.oss-cn-shanghai.aliyuncs.com/mdpic/202303271434494.png" alt="image-20230327143447455" style="zoom:45%;" />

<img src="https://travisnotes.oss-cn-shanghai.aliyuncs.com/mdpic/202303271435611.png" alt="image-20230327143533581" style="zoom:45%;" />

- **示例二：Environment属性绑定**

<img src="https://travisnotes.oss-cn-shanghai.aliyuncs.com/mdpic/202303271437988.png" alt="image-20230327143718966" style="zoom:50%;" />

<img src="https://travisnotes.oss-cn-shanghai.aliyuncs.com/mdpic/202303271437378.png" alt="image-20230327143753341" style="zoom:50%;" />



- 在RpcClientAutoConfig中采用Environment属性绑定的原因？

  - 实际上在RPC-Client中的RpcClientProcessor 实现了 BeanFactoryPostProcessor，而在此时Bean的生命周期还没有开始，所以如果采用注解配置绑定，在执行到RpcClientProcessor时还没有完成绑定，获取不到配置信息。[Bean生命周期参考链接](https://travis1024.github.io/posts/SpringIOC%E8%AF%A6%E8%A7%A3+Bean%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F%E8%AF%A6%E8%A7%A3/)

    <img src="https://travisnotes.oss-cn-shanghai.aliyuncs.com/mdpic/202303271441510.png" alt="image-20230327144124465" style="zoom:50%;" />





**参考链接**

- [深入SpringBoot源码（十一）SpringApplication与Environment的绑定（下）](https://blog.csdn.net/BlackReimu/article/details/124332631)
- [SpringBoot自定义Environment属性及属性绑定](https://blog.csdn.net/qq_30038111/article/details/125400230)