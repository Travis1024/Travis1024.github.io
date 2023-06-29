---
title: RedissonRedLock被弃用
author: Travis <Hongxu Wei>
date: 2023-06-21 09:16:17 +0800
categories: [Java Learning Space]
tags: [redis]
math: false
---

- 官方文档：

<img src="https://travisnotes.oss-cn-shanghai.aliyuncs.com/mdpic/202306210916724.png" alt="image-20230621091649637" style="zoom:100%;" />

- 找一找 issue：

<img src="https://travisnotes.oss-cn-shanghai.aliyuncs.com/mdpic/202306210917038.png" alt="image-20230621091730003" style="zoom:50%;" />

- Redisson 的开发者认为 Redis 的红锁也存在争议，但是为了保证可用性，RLock 对象执行的每个 Redis 命令执行都通过 Redis 3.0 中引入的 WAIT 命令进行同步。

- WAIT 命令会阻塞当前客户端，直到所有以前的写命令都成功的传输并被指定数量的副本确认。如果达到以毫秒为单位指定的超时，则即使尚未达到指定数量的副本，该命令也会返回。 WAIT 命令同步复制也并不能保证强一致性，不过在主节点宕机之后，只不过会尽可能的选择最佳的副本（slaves）

  ![image-20230621091934130](https://travisnotes.oss-cn-shanghai.aliyuncs.com/mdpic/202306210919154.png)



- 【结论】

Redisson RedLock 是基于联锁 MultiLock 实现的，但是使用过程中需要自己判断 key 落在哪个节点上，对使用者不是很友好。

Redisson RedLock 已经被弃用，直接使用普通的加锁即可，会基于 wait 机制将锁同步到从节点，但是也并不能保证一致性。仅仅是最大限度的保证一致性。



- [掘金有关 redisson-redlock 介绍](https://juejin.cn/post/6983988197420171278)
