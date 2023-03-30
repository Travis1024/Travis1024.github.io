---
title: 过滤器+拦截器+ControllerAdvice+切片
author: Travis <Hongxu Wei>
date: 2023-03-29 11:25:00 +0800
categories: [Java Learning Space]
tags: [过滤器, 拦截器, 切片]
math: false
---

## 一、过滤器+拦截器

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



## 二、ControllerAdvice

增强controller可以用来做全局异常处理、全局数据绑定、全局数据预处理；

可以通过实现RequestBodyAdvice、ResponseBodyAdvice接口实现对传入controller接口的请求和接口响应进行全局统一处理

具体使用场景参考链接：[@ControllerAdvice注解的三种使用场景](https://www.cnblogs.com/lenve/p/10748453.html)

## 三、切片@Aspect

spring boot 拦截的方式

1. 过滤器filter

   可以获取http、http请求和响应，但无法获取与spring框架相关的信息，如哪个control处理，哪个方法处理，有哪些参数，这些都是无法获取的。主要用于内容上的过滤，敏感字替换成*等，也可用于非登入状态的非法请求过滤。

2. 拦截器interceptor

   除了获取http、http请求和响应对象，还可以获取请求的类名、方法名，但拦截器无法获取请求参数的值，从DispatcherServlet类源码分析。主要用于对公共的一些拦截获取，例如请求的IP 地址，IP黑白名单里的过过滤，非登入状态的接口请求拦截。

3. 增强controller：ControllerAdvice

   增强controller可以用来做全局异常处理、全局数据绑定、全局数据预处理；

   可以通过实现RequestBodyAdvice、ResponseBodyAdvice接口实现对传入controller接口的请求和接口响应进行全局统一处理

   具体使用场景参考链接：[@ControllerAdvice注解的三种使用场景](https://www.cnblogs.com/lenve/p/10748453.html)

4. 切片拦截Aspect

   能获取到方法请求的参数，方法名，以及方法返回的json数据，更多的是用于数据的处理，比如对操作进行记录，修改，新建，查询，审批等操作记录进行处理统计。对返回的json中的一些特殊数据，比如字典值替换成对应的数据，避免前端转化，等等。





## 四、执行顺序

正常情况：过滤器、拦截器、切片，
异常报错：切片、ControllerAdvice注解类、拦截器、过滤器



**Filter、Interceptor、controllerAdvice、切片--执行顺序**

<img src="https://travisnotes.oss-cn-shanghai.aliyuncs.com/mdpic/202303142037759.png" alt="image-20230314203733720" style="zoom:35%;" />



## 附录

[参考链接1](https://www.cnblogs.com/konglxblog/p/17001902.html)

[参考链接2](https://www.cnblogs.com/kuotian/p/13176186.html#%E6%89%A7%E8%A1%8C%E9%A1%BA%E5%BA%8F)

[springboot切片应用](https://huaweicloud.csdn.net/638754c2dacf622b8df8afdb.html)

[@ControllerAdvice注解的三种使用场景](https://www.cnblogs.com/lenve/p/10748453.html)
