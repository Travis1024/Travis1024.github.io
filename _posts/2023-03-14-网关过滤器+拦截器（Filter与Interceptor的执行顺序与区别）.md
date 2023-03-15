---
title: 网关过滤器+拦截器（Filter与Interceptor的执行顺序与区别）
author: Travis <Hongxu Wei>
date: 2023-03-14 14:25:00 +0800
categories: [Java Learning Space]
tags: [网关过滤器 拦截器]
math: false
---

[参考链接]: https://www.cnblogs.com/kuotian/p/13176186.html#%E6%89%A7%E8%A1%8C%E9%A1%BA%E5%BA%8F
[参考链接]: https://www.cnblogs.com/konglxblog/p/17001902.html

网关会碰到三类过滤器：

- 默认过滤器：DefaultFilter
- 自定义过滤器：GatewayFilter
- 全局过滤器：GlobalFilter

请求路由后，会将三者合并到一个过滤器链（集合）中，排序后依次执行每个过滤器。

<img src="https://travisnotes.oss-cn-shanghai.aliyuncs.com/mdpic/202303142030821.png" alt="image-20230314203032791" style="zoom:40%;" />

**排序规则：**

（1）每一个过滤器都必须指定一个 int 类型的 order 值，**order 值越小，优先级越高，执行顺序越靠前**。

（2）GlobalFilter 通过实现 Ordered 接口，或者使用 @Order 注解来指定 order 值，由我们自己指定。

（3）路由过滤器和 defaultFilter 的 order 由 Spring 指定，默认是按照声明顺序从1递增。



**Filter与Interceptor的执行顺序**

<img src="https://travisnotes.oss-cn-shanghai.aliyuncs.com/mdpic/202303142037759.png" alt="image-20230314203733720" style="zoom:35%;" />
