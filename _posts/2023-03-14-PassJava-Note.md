---
title: PassJava Note
author: Travis <Hongxu Wei>
date: 2023-03-14 22:14:54 +0800
categories: [Java Learning Space]
tags: [passjava project]
math: false
---

| 微服务名称          | 主要功能                                      | 数据库表 |
| ------------------- | --------------------------------------------- | -------- |
| passjava-gateway    | 网关服务                                      |          |
| passjava-auth       | 认证服务                                      | AUTH     |
| passjava-channel    | ####渠道####                                  | chms     |
| passjava-common     | 基础服务模块                                  |          |
| passjava-content    | 内容服务（包括资讯、广告服务）                | cms      |
| passjava-jwt        | JWT公共服务，认证服务和网关服务都会引用此项目 |          |
| passjava-member     | 会员服务                                      | ums      |
| passjava-question   | 题目服务                                      | qms      |
| passjava-study      | 学习服务                                      | sms      |
| passjava-thirdparty | 第三方功能服务（oss存储）                     |          |
| passjava-search     | 搜索服务（ElasticSearch）                     |          |
| Renren-fast         | 后台管理系统                                  | ADMIN    |
| Renren-fast-vue     | 后台管理系统页面                              |          |



| 微服务名称       | 主要功能         |      |
| ---------------- | ---------------- | ---- |
| passjava-miniApp | 小程序           |      |
| passjava-portal  | 门户网站         |      |
| renren-generator | renren代码生成器 |      |



- **鉴权流程图**

  ![image-20230314092415495](https://travisnotes.oss-cn-shanghai.aliyuncs.com/mdpic/202303140924554.png)
