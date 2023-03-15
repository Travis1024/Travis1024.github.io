---
title: Dubbo与Feign
author: Travis <Hongxu Wei>
date: 2023-03-11 22:12:25 +0800
categories: [Java Learning Space]
tags: [dubbo feign]
math: false
---

## 一、相同点

Dubbo 与 Feign 都依赖注册中心、负载均衡，作用是提供远程接口调用。

常见的 实现远程调用的方式： Http接口（web接口、RestTemplate+Okhttp）、Feign、RPC调用（Dubbo、Socket编程）、Webservice。

## 二、区别

Dubbo除了注册中心需要进行整合，其它功能都自己实现了，而Feign大部分功能都是依赖全家桶的组件来实现的。

### 1、协议

- Dubbo：支持多传输协议(Dubbo、Rmi、http、redis等等)，可以根据业务场景选择最佳的方式。非常灵活。  默认的Dubbo协议：利用Netty，TCP传输，单一、异步、长连接，适合==数据量小、高并发、服务提供者远远少于消费者的场景。==

- Feign：基于HTTP传输协议、短连接、不适合高并发的访问

### 2、负载均衡

- Dubbo：支持4种算法（随机、轮询、活跃度、Hash一致性），而且算法里面引入权重的概念。配置的形式不仅支持代码配置，还支持Dubbo控制台灵活动态配置。负载均衡的算法可以精准到某个服务接口的某个方法。

- Feign：只支持N种策略：轮询、随机、ResponseTime加权。负载均衡算法是Client级别的。

Nacos注册中心很好的兼容了Feign，Feign默认集成了Ribbon，所以在Nacos下使用Fegin默认就实现了负载均衡的效果。（当然Nacos也能够整合Dubbo进行使用）

### 3、容错策略

- Dubbo：支持多种容错策略：failover、failfast、brodecast、forking等，也引入了retry次数、timeout等配置参数。

- Feign：利用熔断机制来实现容错的，处理的方式不一样。

## 三、总结

Dubbo支持更多功能、更灵活、支持高并发的RPC框架。

SpringCloud全家桶里面（Feign、Ribbon、Hystrix），特点是非常方便。Ribbon、Hystrix、Feign在服务治理中，配合Spring Cloud做微服务，使用上有很多优势，社区也比较活跃，看将来更新发展。

业务发展影响着架构的选型，当服务数量不是很大时，使用普通的分布式RPC架构即可，当服务数量增长到一定数据，需要进行服务治理时，就需要考虑使用流式计算架构。Dubbo可以方便的做更精细化的流量调度，服务结构治理的方案成熟，适合生产上使用，虽然Dubbo是尘封后重新开启，但这并不影响其技术价值。

==如果项目对性能要求不是很严格，可以选择使用Feign，它使用起来更方便。
如果需要提高性能，避开基于Http方式的性能瓶颈，可以使用Dubbo。==

Dubbo Spring Cloud的出现，使得Dubbo既能够完全整合到Spring Cloud的技术栈中，享受Spring Cloud生态中的技术支持和标准化输出，又能够弥补Spring Cloud中服务治理这方面的短板