---
title: Sentinel-dashboard服务器部署-添加流控规则失败
author: Travis <Hongxu Wei>
date: 2023-06-29 10:24:19 +0800
categories: [Java Learning Space]
tags: [Sentinel]
math: false
---



## 错误信息

![img](https://travisnotes.oss-cn-shanghai.aliyuncs.com/mdpic/202306291024502.png)



## 错误分析

sentinel-dashboard是需要和微服务进行双向交互的，本地的微服务访问接口注册到sentinel-dashboard，同时会在本地起一个http service，端口默认是8719，如果被占用会依次向后尝试，管理页面如下图：

![img](https://travisnotes.oss-cn-shanghai.aliyuncs.com/mdpic/202306291025149.png)

可以看到，控制台保存的ip是内网的，云服务器是无法访问的我们本地的服务的，所以就会出错，上面红框中的ip和端口，我们是可以在本地设置的，参数如下：

```yml
spring:
  cloud:
    sentinel:
      transport:
        clientIp: 127.0.0.1
        port: 8719
```

所以如果sentinel和微服务部署在一条服务器上就没有这个问题，或者微服务部署在另外一台公网服务器上，这里的clientIp就可以直接指定服务器的公网ip



## 解决方案

综上所述，需要让sentinel控制台和微服务可以双向通信，那么方法有三个：

1、如果是本地起的微服务，那个就直接在本地运行sentinel的控制台，将clientIp直接配置成127.0.0.1

2、如果sentinel部署在云服务器上，那么就将本地的服务也放到同一个云服务器上运行

3、如果sentinel和微服务运行在不同的公有云服务器上，则需要指定clientIp为微服务运行服务器的ip

要让云服务器上的sentinel访问本地服务，可以尝试采用本地服务内网透传的方式，需要注意的是sentinel的访问携带了端口，所以不能是全域名映射，必须是域名指定端口映射，本人使用的natapp内网透传是全域名映射，无法指定端口，查阅了资料，nat123好像支持域名指定端口访问。
