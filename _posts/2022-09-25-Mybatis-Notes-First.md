---
title: Mybatis Notes First
author: Travis <Hongxu Wei>
date: 2022-09-25 12:25:00 +0800
categories: [Java Learning Space]
tags: [Mybatis]
math: false
---

# Mybatis Notes First

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

# 二、MyBatis入门程序

只要你会JDBC，MyBatis就可以学。

## 2.1 版本

### 软件版本：

- IntelliJ IDEA：2022.1.4
- Navicat for MySQL：16.0.14
- MySQL数据库：8.0.30

### 组件版本：

- MySQL驱动：8.0.30
- MyBatis：3.5.10
- JDK：Java17
- JUnit：4.13.2
- Logback：1.2.11

## 2.2 MyBatis下载

- 从github上下载，地址：https://github.com/mybatis/mybatis-3

- 将框架以及框架的源码都下载下来，下载框架后解压，打开mybatis目录

    - <img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209291923168.png" style="zoom:50%"/>
    - 通过以上解压可以看到，框架一般都是以jar包的形式存在。我们的mybatis课程使用maven，所以这个jar我们不需要。
    - 官方手册需要。

## 2.3 MyBatis入门程序开发步骤

- 写代码前准备：

    - 准备数据库表：汽车表t_car，字段包括：

        - id：主键（自增）【bigint】

        - car_num：汽车编号【varchar】

        - brand：品牌【varchar】

        - guide_price：厂家指导价【decimal类型，专门为财务数据准备的类型】

        - produce_time：生产时间【char，年月日即可，10个长度，'2022-10-11'】

        - car_type：汽车类型（燃油车、电车、氢能源）【varchar】

    - 使用navicat for mysql工具建表

        <img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209291924829.png" style="zoom:45%"/>

    - 使用navicat for mysql工具向t_car表中插入两条数据，如下：

        <img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209291925094.png" style="zoom:45%"/>

    - 创建Project：建议创建Empty Project，设置Java版本以及编译版本等。

        <img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209291926220.png" style="zoom:40%"/>

        <img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209291927993.png" style="zoom:40%"/>

    - 设置IDEA的maven

        <img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209291939546.png" style="zoom:45%"/>

    - 创建Module：普通的Maven Java模块

        <img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209291940609.png" style="zoom:45%"/>

- 步骤1：打包方式：jar（不需要war，因为mybatis封装的是jdbc。）

    ```xml
    <groupId>com.powernode</groupId>
    <artifactId>mybatis-001-introduction</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>jar</packaging>
    ```

- 步骤2：引入依赖（mybatis依赖 + mysql驱动依赖）

    ```xml
    <!--mybatis核心依赖-->
    
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis</artifactId>
      <version>3.5.10</version>
    </dependency>
    <!--mysql驱动依赖-->
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>8.0.30</version>
    </dependency>
    ```

- 步骤3：在resources根目录下新建mybatis-config.xml配置文件（可以参考mybatis手册拷贝）

    ```xml
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE configuration
            PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
            "http://mybatis.org/dtd/mybatis-3-config.dtd">
    
    
    <configuration>
        <environments default="development">
            <environment id="development">
                <transactionManager type="JDBC"/>
                <dataSource type="POOLED">
                    <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                    <property name="url" value="jdbc:mysql://localhost:3306/powernode"/>
                    <property name="username" value="root"/>
                    <property name="password" value="root"/>
                </dataSource>
            </environment>
        </environments>
        <mappers>
            <!--sql映射文件创建好之后，需要将该文件路径配置到这里-->
            <mapper resource=""/>
        </mappers>
    </configuration>
    ```

注意1：mybatis核心配置文件的文件名不一定是mybatis-config.xml，可以是其它名字。

注意2：mybatis核心配置文件存放的位置也可以随意。这里选择放在resources根下，相当于放到了类的根路径下。

- 步骤4：在resources根目录下新建CarMapper.xml配置文件（可以参考mybatis手册拷贝）

    ```xml
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE mapper
            PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
            "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    
    
    <!--namespace先随意写一个-->
    
    <mapper namespace="car">
        <!--insert sql：保存一个汽车信息-->
        <insert id="insertCar">
            insert into t_car
                (id,car_num,brand,guide_price,produce_time,car_type) 
            values
                (null,'102','丰田mirai',40.30,'2014-10-05','氢能源')
        </insert>
    </mapper>
    ```

注意1：**sql语句最后结尾可以不写“;”**

注意2：CarMapper.xml文件的名字不是固定的。可以使用其它名字。

注意3：CarMapper.xml文件的位置也是随意的。这里选择放在resources根下，相当于放到了类的根路径下。

注意4：将CarMapper.xml文件路径配置到mybatis-config.xml：

```xml
<mapper resource="CarMapper.xml"/>
```

- 步骤5：编写MyBatisIntroductionTest代码

    ```java
    package com.powernode.mybatis;
    import org.apache.ibatis.session.SqlSession;
    import org.apache.ibatis.session.SqlSessionFactory;
    import org.apache.ibatis.session.SqlSessionFactoryBuilder;
    import java.io.InputStream;
    
    public class MyBatisIntroductionTest {
    	public static void main(String[] args) {
          // 1. 创建SqlSessionFactoryBuilder对象
           SqlSessionFactoryBuilder sqlSessionFactoryBuilder = new SqlSessionFactoryBuilder();
           // 2. 创建SqlSessionFactory对象
           InputStream is = Thread.currentThread().getContextClassLoader().getResourceAsStream("mybatis-config.xml");
           SqlSessionFactory sqlSessionFactory = sqlSessionFactoryBuilder.build(is);
           // 3. 创建SqlSession对象
           SqlSession sqlSession = sqlSessionFactory.openSession();
           // 4. 执行sql
           int count = sqlSession.insert("insertCar"); // 这个"insertCar"必须是sql的id
           System.out.println("插入几条数据：" + count);
           // 5. 提交（mybatis默认采用的事务管理器是JDBC，默认是不提交的，需要手动提交。）
           sqlSession.commit();
           // 6. 关闭资源（只关闭是不会提交的）
           sqlSession.close();
       }
    }
    ```
    

注意1：默认采用的事务管理器是：JDBC。JDBC事务默认是不提交的，需要手动提交。

- 步骤6：运行程序，查看运行结果，以及数据库表中的数据

    <img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209291942594.png" style="zoom:45%"/>

## 2.4 关于MyBatis核心配置文件的名字和路径详解

- 核心配置文件的名字是随意的，因为以下的代码：

    ```java
    // 文件名是出现在程序中的，文件名如果修改了，对应这里的java程序也改一下就行了。
    InputStream is = Thread.currentThread().getContextClassLoader().getResourceAsStream("mybatis-config.xml");
    ```

- 核心配置文件必须放到resources下吗？放到D盘根目录下，可以吗？测试一下：

    将mybatis-config.xml文件拷贝一份放到D盘根下，然后编写以下程序：

    ```java
    package com.powernode.mybatis;
    import org.apache.ibatis.session.SqlSession;
    import org.apache.ibatis.session.SqlSessionFactory;
    import org.apache.ibatis.session.SqlSessionFactoryBuilder;
    import java.io.FileInputStream;
    import java.io.InputStream;
    public class MyBatisConfigFilePath {
    	public static void main(String[] args) throws Exception{
          // 1. 创建SqlSessionFactoryBuilder对象
           SqlSessionFactoryBuilder sqlSessionFactoryBuilder = new SqlSessionFactoryBuilder();
           // 2. 创建SqlSessionFactory对象
           // 这只是一个输入流，可以自己new。
           InputStream is = new FileInputStream("D:/mybatis-config.xml"); 
           SqlSessionFactory sqlSessionFactory = sqlSessionFactoryBuilder.build(is);
           // 3. 创建SqlSession对象
           SqlSession sqlSession = sqlSessionFactory.openSession();
           // 4. 执行sql
           int count = sqlSession.insert("insertCar");
           System.out.println("插入几条数据：" + count);
           // 5. 提交（mybatis默认采用的事务管理器是JDBC，默认是不提交的，需要手动提交。）
           sqlSession.commit();
           // 6. 关闭资源（只关闭是不会提交的）
           sqlSession.close();
    
       }
    }
    ```

以上程序运行后，看到数据库表t_car中又新增一条数据，如下（成功了）：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209291943265.png" style="zoom:45%"/>

经过测试说明mybatis核心配置文件的名字是随意的，存放路径也是随意的。

虽然mybatis核心配置文件的名字不是固定的，但通常该文件的名字叫做：mybatis-config.xml

虽然mybatis核心配置文件的路径不是固定的，但通常该文件会存放到**类路径**当中，这样让项目的移植更加健壮。

- 在mybatis中提供了一个类：Resources【org.apache.ibatis.io.Resources】，该类可以从类路径当中获取资源，我们通常使用它来获取输入流InputStream，代码如下:

    ```java
    // 这种方式只能从类路径当中获取资源，也就是说mybatis-config.xml文件需要在类路径下。
    InputStream is = Resources.getResourceAsStream("mybatis-config.xml");
    ```

## 2.5 MyBatis第一个比较完整的代码写法

```java
package com.powernode.mybatis;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import java.io.IOException;
public class MyBatisCompleteCodeTest {
	public static void main(String[] args) {
    	SqlSession sqlSession = null;
    	try {
            // 1.创建SqlSessionFactoryBuilder对象
            SqlSessionFactoryBuilder sqlSessionFactoryBuilder = new SqlSessionFactoryBuilder();
            // 2.创建SqlSessionFactory对象
            SqlSessionFactory sqlSessionFactory = sqlSessionFactoryBuilder.build(Resources.getResourceAsStream("mybatis-config.xml"));
            // 3.创建SqlSession对象
            sqlSession = sqlSessionFactory.openSession();
            // 4.执行SQL
            int count = sqlSession.insert("insertCar");
            System.out.println("更新了几条记录：" + count);
            // 5.提交
            sqlSession.commit();
       } catch (Exception e) {
           // 回滚
           if (sqlSession != null) {
               sqlSession.rollback();
           }
           e.printStackTrace();
       } finally {
           // 6.关闭
           if (sqlSession != null) {
               sqlSession.close();
           }
       }
   }
}
```

运行后数据库表的变化：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209291943617.png" style="zoom:45%"/>

## 2.6 引入JUnit

- JUnit是专门做单元测试的组件。

    - 在实际开发中，单元测试一般是由我们Java程序员来完成的。

    - 我们要对我们自己写的每一个业务方法负责任，要保证每个业务方法在进行测试的时候都能通过。

    - 测试的过程中涉及到两个概念：
        - 期望值

        - 实际值

    - 期望值和实际值相同表示测试通过，期望值和实际值不同则单元测试执行时会报错。

- 这里引入JUnit是为了代替main方法。

- 使用JUnit步骤：

    - 第一步：引入依赖

        ```xml
        <!-- junit依赖 -->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.13.2</version>
            <scope>test</scope>
        </dependency>
        ```

    - 第二步：编写单元测试类【测试用例】，测试用例中每一个测试方法上使用@Test注解进行标注。

        - 测试用例的名字以及每个测试方法的定义都是有规范的：

            - 测试用例的名字：XxxTest
            - 测试方法声明格式：public void test业务方法名(){}
    
            ```java
            // 测试用例
            public class CarMapperTest{
            	// 测试方法
            	@Test
            	public void testInsert(){}
            
            	@Test
            	public void testUpdate(){}
            }
            ```
    
    - 第三步：可以在类上执行，也可以在方法上执行
    
        - 在类上执行时，该类中所有的测试方法都会执行。
        - 在方法上执行时，只执行当前的测试方法。


- 编写一个测试用例，来测试insertCar业务

    ```java
    package com.powernode.mybatis;
    import org.apache.ibatis.io.Resources;
    import org.apache.ibatis.session.SqlSession;
    import org.apache.ibatis.session.SqlSessionFactory;
    import org.apache.ibatis.session.SqlSessionFactoryBuilder;
    import org.junit.Test;
    public class CarMapperTest {
        @Test
        public void testInsertCar(){
            SqlSession sqlSession = null;
            try {
                // 1.创建SqlSessionFactoryBuilder对象
                SqlSessionFactoryBuilder sqlSessionFactoryBuilder = new SqlSessionFactoryBuilder();
                // 2.创建SqlSessionFactory对象
                SqlSessionFactory sqlSessionFactory = sqlSessionFactoryBuilder.build(Resources.getResourceAsStream("mybatis-config.xml"));
                // 3.创建SqlSession对象
                sqlSession = sqlSessionFactory.openSession();
                // 4.执行SQL
                int count = sqlSession.insert("insertCar");
                System.out.println("更新了几条记录：" + count);
                // 5.提交
                sqlSession.commit();
            } catch (Exception e) {
                // 回滚
                if (sqlSession != null) {
                    sqlSession.rollback();
                }
                e.printStackTrace();
            } finally {
                // 6.关闭
                if (sqlSession != null) {
                    sqlSession.close();
                }
            }
        }
    }
    ```

执行单元测试，查看数据库表的变化：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209291946162.png" style="zoom:45%"/>

## 2.7 引入日志框架logback

- 引入日志框架的目的是为了看清楚mybatis执行的具体sql。

- 启用标准日志组件，只需要在mybatis-config.xml文件中添加以下配置：【可参考mybatis手册】

    ```xml
    <settings>
      <setting name="logImpl" value="STDOUT_LOGGING" />
    </settings>
    ```

标准日志也可以用，但是配置不够灵活，可以集成其他的日志组件，例如：log4j，logback等。

- logback是目前日志框架中性能较好的，较流行的，所以我们选它。

- 引入logback的步骤：

    - 第一步：引入logback相关依赖

        ```xml
        <dependency>
          <groupId>ch.qos.logback</groupId>
          <artifactId>logback-classic</artifactId>
          <version>1.2.11</version>
          <scope>test</scope>
        </dependency>
        ```

    - 第二步：引入logback相关配置文件（文件名叫做logback.xml或logback-test.xml，放到类路径当中）

        ```xml
        <?xml version="1.0" encoding="UTF-8"?>
        <configuration debug="false">
            <!-- 控制台输出 -->
            <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
                <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
                    <!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符-->
                    <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
                </encoder>
            </appender>
            <!-- 按照每天生成日志文件 -->
            <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
                <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
                    <!--日志文件输出的文件名-->
                    <FileNamePattern>${LOG_HOME}/TestWeb.log.%d{yyyy-MM-dd}.log</FileNamePattern>
                    <!--日志文件保留天数-->
                    <MaxHistory>30</MaxHistory>
                </rollingPolicy>
                <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
                    <!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符-->
                    <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
                </encoder>
                <!--日志文件最大的大小-->
                <triggeringPolicy class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy">
                    <MaxFileSize>100MB</MaxFileSize>
                </triggeringPolicy>
            </appender>
            <!--mybatis log configure-->
            <logger name="com.apache.ibatis" level="TRACE"/>
            <logger name="java.sql.Connection" level="DEBUG"/>
            <logger name="java.sql.Statement" level="DEBUG"/>
            <logger name="java.sql.PreparedStatement" level="DEBUG"/>
            <!-- 日志输出级别,logback日志级别包括五个：TRACE < DEBUG < INFO < WARN < ERROR -->
            <root level="DEBUG">
                <appender-ref ref="STDOUT"/>
                <appender-ref ref="FILE"/>
            </root>
        </configuration>
        ```


- 再次执行单元测试方法testInsertCar，查看控制台是否有sql语句输出

    <img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209291946122.png"/>

## 2.8 MyBatis工具类SqlSessionUtil的封装

- 每一次获取SqlSession对象代码太繁琐，封装一个工具类

    ```java
    package com.powernode.mybatis.utils;
    import org.apache.ibatis.io.Resources;
    import org.apache.ibatis.session.SqlSession;
    import org.apache.ibatis.session.SqlSessionFactory;
    import org.apache.ibatis.session.SqlSessionFactoryBuilder;
    public class SqlSessionUtil {
    	private static SqlSessionFactory sqlSessionFactory;
    	//类加载时初始化sqlSessionFactory对象
    	static {
     		try {
     			SqlSessionFactoryBuilder sqlSessionFactoryBuilder = new SqlSessionFactoryBuilder();
    			sqlSessionFactory = sqlSessionFactoryBuilder.build(Resources.getResourceAsStream("mybatis-config.xml"));
    		} catch (Exception e) {
    			e.printStackTrace();
    		}
    	}
    	//没调用一次openSession()可获取一个新的会话，该会话支持自动提交
    	public static SqlSession openSession() {
     		return sqlSessionFactory.openSession(true);
    	}
    }
    ```

- 测试工具类，将testInsertCar()改造

    ```java
    @Test
    public void testInsertCar(){
        SqlSession sqlSession = SqlSessionUtil.openSession();
        // 执行SQL
        int count = sqlSession.insert("insertCar");
        System.out.println("插入了几条记录:" + count);
        sqlSession.close();
    }
    ```


