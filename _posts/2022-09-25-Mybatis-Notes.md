---
title: Mybatis Notes
author: Travis <Hongxu Wei>
date: 2022-09-25 23:25:00 +0800
categories: [Java Learning Space]
tags: [Mybatis]
math: true
---

# Mybatis Notes

# 一、MyBatis概述

## 1.1 框架

- 在文献中看到的framework被翻译为框架

- Java常用框架：
    - SSM三大框架：Spring + SpringMVC + MyBatis

    - SpringBoot

    - SpringCloud

    - 等。。

- 框架其实就是对通用代码的封装，提前写好了一堆接口和类，我们可以在做项目的时候直接引入这些接口和类（引入框架），基于这些现有的接口和类进行开发，可以大大提高开发效率。

- 框架一般都以jar包的形式存在。(jar包中有class文件以及各种配置文件等。)

- SSM三大框架的学习顺序：MyBatis、Spring、SpringMVC（仅仅是建议）

## 1.2 三层架构

<div align=center><img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209261948668.png" style="zoom:40%"/>

- 表现层（UI）：直接跟前端打交互（一是接收前端ajax请求，二是返回json数据给前端）

- 业务逻辑层（BLL）：一是处理表现层转发过来的前端请求（也就是具体业务），二是将从持久层获取的数据返回到表现层。

- 数据访问层（DAL）：直接操作数据库完成CRUD，并将获得的数据返回到上一层（也就是业务逻辑层）。

- Java持久层框架：
    - MyBatis

    - Hibernate（实现了JPA规范）

    - jOOQ

    - Guzz

    - Spring Data（实现了JPA规范）

    - ActiveJDBC

    - ......


## 1.3 JDBC不足

- 示例代码1：

    ```java
    // ......
    // sql语句写死在java程序中
    String sql = "insert into t_user(id,idCard,username,password,birth,gender,email,city,street,zipcode,phone,grade) values(?,?,?,?,?,?,?,?,?,?,?,?)";
    PreparedStatement ps = conn.prepareStatement(sql);
    // 繁琐的赋值：思考一下，这种有规律的代码能不能通过反射机制来做自动化。
    ps.setString(1, "1");
    ps.setString(2, "123456789");
    ps.setString(3, "zhangsan");
    ps.setString(4, "123456");
    ps.setString(5, "1980-10-11");
    ps.setString(6, "男");
    ps.setString(7, "zhangsan@126.com");
    ps.setString(8, "北京");
    ps.setString(9, "大兴区凉水河二街");
    ps.setString(10, "1000000");
    ps.setString(11, "16398574152");
    ps.setString(12, "A");
    // 执行SQL
    int count = ps.executeUpdate();
    // ......
    ```

- 示例代码2：

    ```java
    // ......
    // sql语句写死在java程序中
    String sql = "select id,idCard,username,password,birth,gender,email,city,street,zipcode,phone,grade from t_user";
    PreparedStatement ps = conn.prepareStatement(sql);
    ResultSet rs = ps.executeQuery();
    List<User> userList = new ArrayList<>();
    // 思考以下循环中的所有代码是否可以使用反射进行自动化封装。
    while(rs.next()){
        // 获取数据
        String id = rs.getString("id");
        String idCard = rs.getString("idCard");
        String username = rs.getString("username");
        String password = rs.getString("password");
        String birth = rs.getString("birth");
        String gender = rs.getString("gender");
        String email = rs.getString("email");
        String city = rs.getString("city");
        String street = rs.getString("street");
        String zipcode = rs.getString("zipcode");
        String phone = rs.getString("phone");
        String grade = rs.getString("grade");
        // 创建对象
        User user = new User();
        // 给对象属性赋值
        user.setId(id);
        user.setIdCard(idCard);
        user.setUsername(username);
        user.setPassword(password);
        user.setBirth(birth);
        user.setGender(gender);
        user.setEmail(email);
        user.setCity(city);
        user.setStreet(street);
        user.setZipcode(zipcode);
        user.setPhone(phone);
        user.setGrade(grade);
        // 添加到集合
        userList.add(user);
    }
    // ......
    ```

- JDBC不足：
    - SQL语句写死在Java程序中，不灵活。改SQL的话就要改Java代码。违背开闭原则OCP。

    - 给?传值是繁琐的。能不能自动化？？？

    - 将结果集封装成Java对象是繁琐的。能不能自动化？？？


## 1.4 了解MyBatis

- MyBatis本质上就是对JDBC的封装，通过MyBatis完成CRUD。

- MyBatis在三层架构中负责持久层的，属于持久层框架。

- MyBatis的发展历程：【引用百度百科】

    - MyBatis本是apache的一个开源项目iBatis，2010年这个项目由apache software foundation迁移到了google code，并且改名为MyBatis。2013年11月迁移到Github。

    - iBATIS一词来源于“internet”和“abatis”的组合，是一个基于Java的持久层框架。iBATIS提供的持久层框架包括SQL Maps和Data Access Objects（DAOs）。

- 打开mybatis代码可以看到它的包结构中包含：ibatis

    <img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209261953924.png" style="zoom:50%"/>

- ORM：对象关系映射

    - O（Object）：Java虚拟机中的Java对象
    - R（Relational）：关系型数据库
    - M（Mapping）：将Java虚拟机中的Java对象映射到数据库表中一行记录，或是将数据库表中一行记录映射成Java虚拟机中的一个Java对象。
    - ORM图示

    <img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209261954098.png" style="zoom:40%"/>

    - MyBatis属于半自动化ORM框架。
    - Hibernate属于全自动化的ORM框架。
    
- MyBatis框架特点：

    - 支持定制化 SQL、存储过程、基本映射以及高级映射
    - 避免了几乎所有的 JDBC 代码中手动设置参数以及获取结果集
    - 支持XML开发，也支持注解式开发。【为了保证sql语句的灵活，所以mybatis大部分是采用XML方式开发。】
    - 将接口和 Java 的 POJOs(Plain Ordinary Java Object，简单普通的Java对象)映射成数据库中的记录
    - 体积小好学：两个jar包，两个XML配置文件。
    - 完全做到sql解耦合。
    - 提供了基本映射标签。
    - 提供了高级映射标签。
    - 提供了XML标签，支持动态SQL的编写。
    - ......
