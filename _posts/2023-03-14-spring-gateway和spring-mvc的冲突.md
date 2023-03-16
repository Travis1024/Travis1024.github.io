---
title: spring-gateway和spring-mvc的冲突
author: Travis <Hongxu Wei>
date: 2023-03-14 11:27:55 +0800
categories: [Java Learning Space]
tags: [gateway, mvc, spring]
math: false
---

- 报错信息

  ```
  WebMvcConfigurer.class] cannot be opened because it does not exist
  ```

- 原因

   gateway在其内部导入了webflux包，但是webmvc和webflux是不能同时出现的。

- 解决方法：

  - 1、除去gateway服务中spring-boot-starter-web和spring-webmvc的依赖

    ```xml
    <exclusions>
        <exclusion>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
        </exclusion>
    </exclusions>
    ```

  - 2、导入的时候忽略spring-boot-starter-web(一般来说，另一个项目中导入了web，一般是用到了web，所以这个方法可能会导致出现其他问题)

    第二种办法是在application.yml中加入配置

    ```yaml
    spring:
      #解决gateway 与 mvc 包冲突的问题
      main:
        web-application-type: reactive
    ```

    