---
title: Austin-Note
author: Travis <Hongxu Wei>
date: 2023-03-22 14:36:45 +0800
categories: [Java Learning Space]
tags: [project]
math: false
---



| 工程模块                    | 作用                                                 |
| --------------------------- | ---------------------------------------------------- |
| **austin-common**           | **项目公共包：存储着项目公共常量/枚举/Bean**         |
| **austin-cron**             | **定时任务模块：对xxl-job封装和项目定时任务逻辑**    |
| **austin-data-house**       | **数据仓库模块：消费MQ数据写入hive**                 |
| **austin-handler**          | **消息处理逻辑层：消费MQ下发消息**                   |
| **austin-service-api**      | **消息接入层接口定义模块：只有接口和必要的入参依赖** |
| **austin-service-api-impl** | **消息接入层具体实现模块：真实处理请求**             |
| **austin-stream**           | **实时处理模块：利用flink实时处理下发链路数据**      |
| **austin-support**          | **项目工具包：对接中间件/组件**                      |
| **austin-web**              | **后台管理模块：提供接口给前端调用**                 |



![image-20230329102333850](https://travisnotes.oss-cn-shanghai.aliyuncs.com/mdpic/202303291023900.png)
