---
title: servlet、tomcat、spring关系
author: Travis <Hongxu Wei>
date: 2023-03-13 14:25:00 +0800
categories: [Java Learning Space]
tags: [servlet tomcat spring]
math: false
---

[Servlet／Tomcat/ Spring 之间的关系\_servlet和spring框架的关系\_gb4215287的博客-CSDN博客](https://blog.csdn.net/gb4215287/article/details/115586213)

[附加：【Servlet】和【Spring MVC】的关系；【Servlet】体系简述；\_servlet和spring框架的关系\_红薯丸子炸糖糕的博客-CSDN博客](https://wgy-coder.blog.csdn.net/article/details/121529374?)

<img src="https://gitee.com/awtaling/images_repository/raw/master/202303141057003.png" alt="image-20230314105745984" style="zoom:50%;" />

<img src="https://gitee.com/awtaling/images_repository/raw/master/202303141057250.png" alt="image-20230314105704194" style="zoom:60%;" />

客户端的请求直接打到tomcat，它监听端口，请求过来后，根据url等信息，确定要将请求交给哪个servlet去处理，然后调用那个servlet的service方法，service方法返回一个response对象，tomcat再把这个response返回给客户端。

**SpringMVC组件和流程图**

<img src="https://gitee.com/awtaling/images_repository/raw/master/202303141101231.png" alt="image-20230314110117204" style="zoom: 50%;" />

**实例组件和运行流程**

<img src="https://gitee.com/awtaling/images_repository/raw/master/202303141102897.png" alt="image-20230314110206875" style="zoom:50%;" />
