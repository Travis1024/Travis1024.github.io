---
title: Mybatis Notes
author: Travis <Hongxu Wei>
date: 2022-09-25 00:25:00 +0800
categories: [Java Learning Space]
tags: [Mybatis]
math: false
---



# Mybatis Notes (1)

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



# 三、使用MyBatis完成CRUD

- 准备工作
    - 创建module（Maven的普通Java模块）：mybatis-002-crud

    - pom.xml
        - 打包方式jar

        - 依赖：
            - mybatis依赖

            - mysql驱动依赖

            - junit依赖

            - logback依赖

    - mybatis-config.xml放在类的根路径下

    - CarMapper.xml放在类的根路径下

    - logback.xml放在类的根路径下

    - 提供com.powernode.mybatis.utils.SqlSessionUtil工具类

    - 创建测试用例：com.powernode.mybatis.CarMapperTest


## 3.1 insert（Create）

分析以下SQL映射文件中SQL语句存在的问题

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!--namespace先随便写-->
<mapper namespace="car">
    <insert id="insertCar">
        insert into t_car(car_num,brand,guide_price,produce_time,car_type) values('103', '奔驰E300L', 50.3, '2022-01-01', '燃油车')
    </insert>
</mapper>
```

存在的问题是：SQL语句中的值不应该写死，值应该是用户提供的。之前的JDBC代码是这样写的：

```java
// JDBC中使用 ? 作为占位符。那么MyBatis中会使用什么作为占位符呢？
String sql = "insert into t_car(car_num,brand,guide_price,produce_time,car_type) values(?,?,?,?,?)";
// ......
// 给 ? 传值。那么MyBatis中应该怎么传值呢？
ps.setString(1,"103");
ps.setString(2,"奔驰E300L");
ps.setDouble(3,50.3);
ps.setString(4,"2022-01-01");
ps.setString(5,"燃油车");
```

在MyBatis中可以这样做：

​	**在Java程序中，将数据放到Map集合中**

​	**在sql语句中使用 #{map集合的key} 来完成传值，#{} 等同于JDBC中的 ? ，#{}就是占位符**

Java程序这样写：

```java
package com.powernode.mybatis;
import com.powernode.mybatis.utils.SqlSessionUtil;
import org.apache.ibatis.session.SqlSession;
import org.junit.Test;
import java.util.HashMap;
import java.util.Map;

public class CarMapperTest {
	@Test
    public void testInsertCar(){
    	// 准备数据
        Map<String, Object> map = new HashMap<>();
        map.put("k1", "103");
        map.put("k2", "奔驰E300L");
        map.put("k3", 50.3);
        map.put("k4", "2020-10-01");
        map.put("k5", "燃油车");
        // 获取SqlSession对象
        SqlSession sqlSession = SqlSessionUtil.openSession();
        // 执行SQL语句（使用map集合给sql语句传递数据）
        int count = sqlSession.insert("insertCar", map);
        System.out.println("插入了几条记录：" + count);
	}
}
```

SQL语句这样写：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<!--namespace先随便写-->
<mapper namespace="car">
    <insert id="insertCar">
        insert into t_car(car_num,brand,guide_price,produce_time,car_type) values(#{k1},#{k2},#{k3},#{k4},#{k5})
    </insert>
</mapper>
```

**#{} 的里面必须填写map集合的key，不能随便写。**运行测试程序，查看数据库：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209291947956.png" style="zoom:45%"/>

如果#{}里写的是map集合中不存在的key会有什么问题？

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="car">
    <insert id="insertCar">
        insert into t_car(car_num,brand,guide_price,produce_time,car_type) values(#{kk},#{k2},#{k3},#{k4},#{k5})
    </insert>
</mapper>
```

运行程序：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209291948504.png" style="zoom:50%"/>

通过测试，看到程序并没有报错。正常执行。不过 #{kk} 的写法导致无法获取到map集合中的数据，最终导致数据库表car_num插入了NULL。

在以上sql语句中，可以看到#{k1} #{k2} #{k3} #{k4} #{k5}的可读性太差，为了增强可读性，我们可以将Java程序做如下修改：

```java
Map<String, Object> map = new HashMap<>();
// 让key的可读性增强
map.put("carNum", "103");
map.put("brand", "奔驰E300L");
map.put("guidePrice", 50.3);
map.put("produceTime", "2020-10-01");
map.put("carType", "燃油车");
```

SQL语句做如下修改，这样可以增强程序的可读性：

```sql
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="car">
    <insert id="insertCar">
        insert into t_car(car_num,brand,guide_price,produce_time,car_type) values(#{carNum},#{brand},#{guidePrice},#{produceTime},#{carType})
    </insert>
</mapper>
```

运行程序，查看数据库表：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209291948935.png" style="zoom:45%"/>

使用Map集合可以传参，那使用**pojo**（简单普通的java对象）可以完成传参吗？测试一下：

- 第一步：定义一个pojo类Car，提供相关属性。

    ```java
    package com.powernode.mybatis.pojo;
    	//POJOs，简单普通的Java对象。封装数据用的。
        public class Car {
        private Long id;
        private String carNum;
        private String brand;
        private Double guidePrice;
        private String produceTime;
        private String carType;
        @Override
        public String toString() {
        return "Car{" +
             "id=" + id +
             ", carNum='" + carNum + '\'' +
             ", brand='" + brand + '\'' +
             ", guidePrice=" + guidePrice +
             ", produceTime='" + produceTime + '\'' +
             ", carType='" + carType + '\'' +
             '}';
    	}
    	public Car() {}
        public Car(Long id, String carNum, String brand, Double guidePrice, String produceTime, String carType) {
            this.id = id;
            this.carNum = carNum;
            this.brand = brand;
            this.guidePrice = guidePrice;
            this.produceTime = produceTime;
            this.carType = carType;
    	}
    	public Long getId() {
            return id;
        }
    
        public void setId(Long id) {
            this.id = id;
        }
    
        public String getCarNum() {
            return carNum;
        }
    
        public void setCarNum(String carNum) {
            this.carNum = carNum;
        }
    
        public String getBrand() {
            return brand;
        }
    
        public void setBrand(String brand) {
            this.brand = brand;
        }
    
        public Double getGuidePrice() {
            return guidePrice;
        }
    
        public void setGuidePrice(Double guidePrice) {
            this.guidePrice = guidePrice;
        }
    
        public String getProduceTime() {
            return produceTime;
        }
    
        public void setProduceTime(String produceTime) {
            this.produceTime = produceTime;
        }
    
        public String getCarType() {
            return carType;
        }
    
        public void setCarType(String carType) {
            this.carType = carType;
        }
    }
    ```

- 第二步：Java程序

    ```java
    @Test
    public void testInsertCarByPOJO(){
        // 创建POJO，封装数据
        Car car = new Car();
        car.setCarNum("103");
        car.setBrand("奔驰C200");
        car.setGuidePrice(33.23);
        car.setProduceTime("2020-10-11");
        car.setCarType("燃油车");
        // 获取SqlSession对象
        SqlSession sqlSession = SqlSessionUtil.openSession();
        // 执行SQL，传数据
        int count = sqlSession.insert("insertCarByPOJO", car);
        System.out.println("插入了几条记录" + count);
    }
    ```

- 第三步：SQL语句

    ```sql
    <insert id="insertCarByPOJO">
    	<!--#{} 里写的是POJO的属性名-->
    	insert into t_car(car_num,brand,guide_price,produce_time,car_type) values(#{carNum},#{brand},#{guidePrice},#{produceTime},#{carType})
    </insert>
    ```

- 运行程序，查看数据库表：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209291949173.png" style="zoom:45%"/>

#{} 里写的是POJO的属性名，如果写成其他的会有问题吗？

```sql
<insert id="insertCarByPOJO">
	insert into t_car(car_num,brand,guide_price,produce_time,car_type) values(#{a},#{brand},#{guidePrice},#{produceTime},#{carType})
</insert>
```

运行程序，出现了以下异常：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209291950756.png" style="zoom:45%"/>

错误信息中描述：在Car类中没有找到a属性的getter方法。

修改POJO类Car的代码，**只将getCarNum()方法名修改为getA()，其他代码不变**，如下：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209291950269.png" style="zoom:40%"/>

再运行程序，查看数据库表中数据：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209291951234.png" style="zoom:45%"/>

**经过测试得出结论：**

​	**如果采用map集合传参，#{} 里写的是map集合的key，如果key不存在不会报错，数据库表中会插入NULL。**

​	**如果采用POJO传参，#{} 里写的是get方法的方法名去掉get之后将剩下的单词首字母变小写（例如：getAge对应的是#{age}，getUserName对应的是#{userName}），如果这样的get方法不存在会报错。**

注意：其实传参数的时候有一个属性parameterType，这个属性用来指定传参的数据类型，不过这个属性是可以省略的.

```sql
<insert id="insertCar" parameterType="java.util.Map">
  insert into t_car(car_num,brand,guide_price,produce_time,car_type) values(#{carNum},#{brand},#{guidePrice},#{produceTime},#{carType})
</insert>

<insert id="insertCarByPOJO" parameterType="com.powernode.mybatis.pojo.Car">
  insert into t_car(car_num,brand,guide_price,produce_time,car_type) values(#{carNum},#{brand},#{guidePrice},#{produceTime},#{carType})
</insert>
```

## 3.2 delete（Delete）

需求：根据car_num进行删除。

SQL语句这样写：

```xml
<delete id="deleteByCarNum">
  delete from t_car where car_num = #{SuiBianXie}
</delete>
```

Java程序这样写：

```java
@Test
public void testDeleteByCarNum(){
    // 获取SqlSession对象
    SqlSession sqlSession = SqlSessionUtil.openSession();
    // 执行SQL语句
    int count = sqlSession.delete("deleteByCarNum", "102");
    System.out.println("删除了几条记录：" + count);
}
```

运行结果：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209291951943.png" style="zoom:45%"/>

**注意：当占位符只有一个的时候，${} 里面的内容可以随便写。**

## 3.3 update（Update）

需求：修改id=34的Car信息，car_num为102，brand为比亚迪汉，guide_price为30.23，produce_time为2018-09-10，car_type为电车

修改前：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209291952810.png" style="zoom:45%"/>

SQL语句如下：

```sql
<update id="updateCarByPOJO">
  update t_car set 
    car_num = #{carNum}, brand = #{brand}, 
    guide_price = #{guidePrice}, produce_time = #{produceTime}, 
    car_type = #{carType} 
  where id = #{id}
</update>
```

Java代码如下：

```java
@Test
public void testUpdateCarByPOJO(){
    // 准备数据
    Car car = new Car();
    car.setId(34L);
    car.setCarNum("102");
    car.setBrand("比亚迪汉");
    car.setGuidePrice(30.23);
    car.setProduceTime("2018-09-10");
    car.setCarType("电车");
    // 获取SqlSession对象
    SqlSession sqlSession = SqlSessionUtil.openSession();
    // 执行SQL语句
    int count = sqlSession.update("updateCarByPOJO", car);
    System.out.println("更新了几条记录：" + count);
}
```

运行结果：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209291953267.png" style="zoom:45%"/>

当然了，如果使用**map**传数据也是可以的。

## 3.4 select（Retrieve）

select语句和其它语句不同的是：查询会有一个结果集。来看mybatis是怎么处理结果集的！！！

### 查询一条数据

需求：查询id为1的Car信息

SQL语句如下：

```sql
<select id="selectCarById">
  select * from t_car where id = #{id}
</select>
```


Java程序如下：

```java
@Test
public void testSelectCarById(){
    // 获取SqlSession对象
    SqlSession sqlSession = SqlSessionUtil.openSession();
    // 执行SQL语句
    Object car = sqlSession.selectOne("selectCarById", 1);
    System.out.println(car);
}
```

运行结果如下：

```java
### Error querying database.  Cause: org.apache.ibatis.executor.ExecutorException: 
    A query was run and no Result Maps were found for the Mapped Statement 'car.selectCarById'.  【翻译】：对于一个查询语句来说，没有找到查询的结果映射。
    It's likely that neither a Result Type nor a Result Map was specified.						 【翻译】：很可能既没有指定结果类型，也没有指定结果映射。
```

以上的异常大致的意思是：对于一个查询语句来说，你需要指定它的“结果类型”或者“结果映射”。

所以说，你想让mybatis查询之后返回一个Java对象的话，至少你要告诉mybatis返回一个什么类型的Java对象，可以在<select>标签中添加resultType属性，用来指定查询要转换的类型：

```sql
<select id="selectCarById" resultType="com.powernode.mybatis.pojo.Car">
  select * from t_car where id = #{id}
</select>
```


运行结果：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209291954854.png" style="zoom:45%"/>

运行后之前的异常不再出现了，这说明添加了resultType属性之后，解决了之前的异常，可以看出resultType是不能省略的。

仔细观察控制台的日志信息，不难看出，结果查询出了一条。并且每个字段都查询到值了：Row: 1, 100, 宝马520Li, 41.00, 2022-09-01, 燃油车

但是奇怪的是返回的Car对象，只有id和brand两个属性有值，其它属性的值都是null，这是为什么呢？我们来观察一下查询结果列名和Car类的属性名是否能一一对应：

查询结果集的列名：id, car_num, brand, guide_price, produce_time, car_type

Car类的属性名：id, carNum, brand, guidePrice, produceTime, carType

通过观察发现：只有id和brand是一致的，其他字段名和属性名对应不上，这是不是导致null的原因呢？我们尝试在sql语句中使用as关键字来给查询结果列名起别名试试：

```sql
<select id="selectCarById" resultType="com.powernode.mybatis.pojo.Car">
  select 
    id, car_num as carNum, brand, guide_price as guidePrice, produce_time as produceTime, car_type as carType 
  from 
    t_car 
  where 
    id = #{id}
</select>
```


运行结果如下：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209291954621.png" style="zoom:45%"/>

通过测试得知，如果当查询结果的字段名和java类的属性名对应不上的话，可以采用as关键字起别名，**当然还有其它解决方案，我们后面再看**。

### 查询多条数据

需求：查询所有的Car信息。

SQL语句如下：

```sql
<!--虽然结果是List集合，但是resultType属性需要指定的是List集合中元素的类型。-->
<select id="selectCarAll" resultType="com.powernode.mybatis.pojo.Car">
  <!--记得使用as起别名，让查询结果的字段名和java类的属性名对应上。-->
  select
    id, car_num as carNum, brand, guide_price as guidePrice, produce_time as produceTime, car_type as carType
  from
    t_car
</select>
```


Java代码如下：

```java
@Test
public void testSelectCarAll(){
    // 获取SqlSession对象
    SqlSession sqlSession = SqlSessionUtil.openSession();
    // 执行SQL语句
    List<Object> cars = sqlSession.selectList("selectCarAll");
    // 输出结果
    cars.forEach(car -> System.out.println(car));
}
```

运行结果如下：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209291955877.png" style="zoom:45%"/>

## 3.5 关于SQL Mapper的namespace

在SQL Mapper配置文件中<mapper>标签的namespace属性可以翻译为命名空间，这个命名空间主要是为了防止sqlId冲突的。

创建CarMapper2.xml文件，代码如下：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="car2">
    <select id="selectCarAll" resultType="com.powernode.mybatis.pojo.Car">
        select
            id, car_num as carNum, brand, guide_price as guidePrice, produce_time as produceTime, car_type as carType
        from
            t_car
    </select>
</mapper>
```

不难看出，CarMapper.xml和CarMapper2.xml文件中都有 id="selectCarAll"

将CarMapper2.xml配置到mybatis-config.xml文件中。

```xml
<mappers>
  <mapper resource="CarMapper.xml"/>
  <mapper resource="CarMapper2.xml"/>
</mappers>
```

编写Java代码如下：

```java
@Test
public void testNamespace(){
    // 获取SqlSession对象
    SqlSession sqlSession = SqlSessionUtil.openSession();
    // 执行SQL语句
    List<Object> cars = sqlSession.selectList("selectCarAll");
    // 输出结果
    cars.forEach(car -> System.out.println(car));
}
```

运行结果如下：

```java
org.apache.ibatis.exceptions.PersistenceException: 
### Error querying database.  Cause: java.lang.IllegalArgumentException: 
  selectCarAll is ambiguous in Mapped Statements collection (try using the full name including the namespace, or rename one of the entries) 
  【翻译】selectCarAll在Mapped Statements集合中不明确（请尝试使用包含名称空间的全名，或重命名其中一个条目）
  【大致意思是】selectCarAll重名了，你要么在selectCarAll前添加一个名称空间，要有你改个其它名字。
```

Java代码修改如下：

```java
@Test
public void testNamespace(){
    // 获取SqlSession对象
    SqlSession sqlSession = SqlSessionUtil.openSession();
    // 执行SQL语句
    //List<Object> cars = sqlSession.selectList("car.selectCarAll");
    List<Object> cars = sqlSession.selectList("car2.selectCarAll");
    // 输出结果
    cars.forEach(car -> System.out.println(car));
}
```

运行结果如下：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209291956197.png" style="zoom:45%"/>

# 四、MyBatis核心配置文件详解

```xml
<!--mybatis-config.xml-->
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
        <mapper resource="CarMapper.xml"/>
        <mapper resource="CarMapper2.xml"/>
    </mappers>
</configuration>
```

- configuration：根标签，表示配置信息。

- environments：环境（多个），以“s”结尾表示复数，也就是说mybatis的环境可以配置多个数据源。
    - default属性：表示默认使用的是哪个环境，default后面填写的是environment的id。**default的值只需要和environment的id值一致即可**。

- environment：具体的环境配置（**主要包括：事务管理器的配置 + 数据源的配置**）
    - id：给当前环境一个唯一标识，该标识用在environments的default后面，用来指定默认环境的选择。

- transactionManager：配置事务管理器
    - type属性：指定事务管理器具体使用什么方式，可选值包括两个
        - **JDBC**：使用JDBC原生的事务管理机制。**底层原理：事务开启conn.setAutoCommit(false); ...处理业务...事务提交conn.commit();**

        - **MANAGED**：交给其它容器来管理事务，比如WebLogic、JBOSS等。如果没有管理事务的容器，则没有事务。**没有事务的含义：只要执行一条DML语句，则提交一次**。

- dataSource：指定数据源
    - type属性：用来指定具体使用的数据库连接池的策略，可选值包括三个
        - **UNPOOLED**：采用传统的获取连接的方式，虽然也实现Javax.sql.DataSource接口，但是并没有使用池的思想。
            - property可以是：
                - driver 这是 JDBC 驱动的 Java 类全限定名。

                - url 这是数据库的 JDBC URL 地址。

                - username 登录数据库的用户名。

                - password 登录数据库的密码。

                - defaultTransactionIsolationLevel 默认的连接事务隔离级别。

                - defaultNetworkTimeout 等待数据库操作完成的默认网络超时时间（单位：毫秒）

        - **POOLED**：采用传统的javax.sql.DataSource规范中的连接池，mybatis中有针对规范的实现。
            - property可以是（除了包含**UNPOOLED**中之外）：
                - poolMaximumActiveConnections 在任意时间可存在的活动（正在使用）连接数量，默认值：10

                - poolMaximumIdleConnections 任意时间可能存在的空闲连接数。

                - 其它....

        - **JNDI**：采用服务器提供的JNDI技术实现，来获取DataSource对象，不同的服务器所能拿到DataSource是不一样。如果不是web或者maven的war工程，JNDI是不能使用的。
            - property可以是（最多只包含以下两个属性）：
                - initial_context 这个属性用来在 InitialContext 中寻找上下文（即，initialContext.lookup(initial_context)）这是个可选属性，如果忽略，那么将会直接从 InitialContext 中寻找 data_source 属性。

                - data_source 这是引用数据源实例位置的上下文路径。提供了 initial_context 配置时会在其返回的上下文中进行查找，没有提供时则直接在 InitialContext 中查找。

- mappers：在mappers标签中可以配置多个sql映射文件的路径。

- mapper：配置某个sql映射文件的路径
    - resource属性：使用相对于类路径的资源引用方式

    - url属性：使用完全限定资源定位符（URL）方式


## 4.1 environment

mybatis-003-configuration

```xml
<!--mybatis-config.xml-->
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!--默认使用开发环境-->
    <!--<environments default="dev">-->
    <!--默认使用生产环境-->
    <environments default="production">
        <!--开发环境-->
        <environment id="dev">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/powernode"/>
                <property name="username" value="root"/>
                <property name="password" value="root"/>
            </dataSource>
        </environment>
        <!--生产环境-->
        <environment id="production">
            <transactionManager type="JDBC" />
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/mybatis"/>
                <property name="username" value="root"/>
                <property name="password" value="root"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <mapper resource="CarMapper.xml"/>
    </mappers>
</configuration>
```

```xml
<!--CarMapper.xml-->
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="car">
    <insert id="insertCar">
        insert into t_car(id,car_num,brand,guide_price,produce_time,car_type) values(null,#{carNum},#{brand},#{guidePrice},#{produceTime},#{carType})
    </insert>
</mapper>
```

```java
//ConfigurationTest.testEnvironment
package com.powernode.mybatis;

import com.powernode.mybatis.pojo.Car;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.Test;

public class ConfigurationTest {

    @Test
    public void testEnvironment() throws Exception{
        // 准备数据
        Car car = new Car();
        car.setCarNum("133");
        car.setBrand("丰田霸道");
        car.setGuidePrice(50.3);
        car.setProduceTime("2020-01-10");
        car.setCarType("燃油车");

        // 一个数据库对应一个SqlSessionFactory对象
        // 两个数据库对应两个SqlSessionFactory对象，以此类推
        SqlSessionFactoryBuilder sqlSessionFactoryBuilder = new SqlSessionFactoryBuilder();

        // 使用默认数据库
        SqlSessionFactory sqlSessionFactory = sqlSessionFactoryBuilder.build(Resources.getResourceAsStream("mybatis-config.xml"));
        SqlSession sqlSession = sqlSessionFactory.openSession(true);
        int count = sqlSession.insert("insertCar", car);
        System.out.println("插入了几条记录：" + count);

        // 使用指定数据库
        SqlSessionFactory sqlSessionFactory1 = sqlSessionFactoryBuilder.build(Resources.getResourceAsStream("mybatis-config.xml"), "dev");
        SqlSession sqlSession1 = sqlSessionFactory1.openSession(true);
        int count1 = sqlSession1.insert("insertCar", car);
        System.out.println("插入了几条记录：" + count1);
    }
}
```

执行结果：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209291957618.png" style="zoom:45%"/>

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209291957173.png" style="zoom:45%"/>

## 4.2 transactionManager

```xml
<!--mybatis-config2.xml-->
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <environments default="dev">
        <environment id="dev">
            <transactionManager type="MANAGED"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/powernode"/>
                <property name="username" value="root"/>
                <property name="password" value="root"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <mapper resource="CarMapper.xml"/>
    </mappers>
</configuration>
```

```java
//ConfigurationTest.testTransactionManager
@Test
public void testTransactionManager() throws Exception{
    // 准备数据
    Car car = new Car();
    car.setCarNum("133");
    car.setBrand("丰田霸道");
    car.setGuidePrice(50.3);
    car.setProduceTime("2020-01-10");
    car.setCarType("燃油车");
    // 获取SqlSessionFactory对象
    SqlSessionFactoryBuilder sqlSessionFactoryBuilder = new SqlSessionFactoryBuilder();
    SqlSessionFactory sqlSessionFactory = sqlSessionFactoryBuilder.build(Resources.getResourceAsStream("mybatis-config2.xml"));
    // 获取SqlSession对象
    SqlSession sqlSession = sqlSessionFactory.openSession();
    // 执行SQL
    int count = sqlSession.insert("insertCar", car);
    System.out.println("插入了几条记录：" + count);
}
```

当事务管理器是：JDBC

- 采用JDBC的原生事务机制：
    - 开启事务：conn.setAutoCommit(false);

    - 处理业务......

    - 提交事务：conn.commit();


当事务管理器是：MANAGED

- 交给容器去管理事务，但目前使用的是本地程序，没有容器的支持，**当mybatis找不到容器的支持时：没有事务**。也就是说只要执行一条DML语句，则提交一次。

## 4.3 dataSource

```xml
<!--mybatis-config3.xml-->
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <environments default="dev">
        <environment id="dev">
            <transactionManager type="JDBC"/>
            <dataSource type="UNPOOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/powernode"/>
                <property name="username" value="root"/>
                <property name="password" value="root"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <mapper resource="CarMapper.xml"/>
    </mappers>
</configuration>
```

```java
//ConfigurationTest.testDataSource
@Test
public void testDataSource() throws Exception{
    // 准备数据
    Car car = new Car();
    car.setCarNum("133");
    car.setBrand("丰田霸道");
    car.setGuidePrice(50.3);
    car.setProduceTime("2020-01-10");
    car.setCarType("燃油车");
    // 获取SqlSessionFactory对象
    SqlSessionFactoryBuilder sqlSessionFactoryBuilder = new SqlSessionFactoryBuilder();
    SqlSessionFactory sqlSessionFactory = sqlSessionFactoryBuilder.build(Resources.getResourceAsStream("mybatis-config3.xml"));
    // 获取SqlSession对象
    SqlSession sqlSession = sqlSessionFactory.openSession(true);
    // 执行SQL
    int count = sqlSession.insert("insertCar", car);
    System.out.println("插入了几条记录：" + count);
    // 关闭会话
    sqlSession.close();
}
```

当type是UNPOOLED，控制台输出：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209291958085.png" style="zoom:45%"/>

修改配置文件mybatis-config3.xml中的配置：

```xml
<dataSource type="POOLED">
```

Java测试程序不需要修改，直接执行，看控制台输出：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209291958817.png" style="zoom:45%"/>

通过测试得出：UNPOOLED不会使用连接池，每一次都会新建JDBC连接对象。POOLED会使用数据库连接池。【这个连接池是mybatis自己实现的。】

```xml
<dataSource type="JNDI">
```

JNDI的方式：表示对接JNDI服务器中的连接池。这种方式给了我们可以使用第三方连接池的接口。如果想使用dbcp、c3p0、druid（德鲁伊）等，需要使用这种方式。

这种再重点说一下type="POOLED"的时候，它的属性有哪些？

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <environments default="dev">
        <environment id="dev">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/powernode"/>
                <property name="username" value="root"/>
                <property name="password" value="root"/>
                <!--最大连接数-->
                <property name="poolMaximumActiveConnections" value="3"/>
                <!--这是一个底层设置，如果获取连接花费了相当长的时间，连接池会打印状态日志并重新尝试获取一个连接（避免在误配置的情况下一直失败且不打印日志），默认值：20000 毫秒（即 20 秒）。-->
                <property name="poolTimeToWait" value="20000"/>
                <!--强行回归池的时间-->
                <property name="poolMaximumCheckoutTime" value="20000"/>
                <!--最多空闲数量-->
                <property name="poolMaximumIdleConnections" value="1"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <mapper resource="CarMapper.xml"/>
    </mappers>
</configuration>
```

poolMaximumActiveConnections：最大的活动的连接数量。默认值10

poolMaximumIdleConnections：最大的空闲连接数量。默认值5

poolMaximumCheckoutTime：强行回归池的时间。默认值20秒。

poolTimeToWait：当无法获取到空闲连接时，每隔20秒打印一次日志，避免因代码配置有误，导致傻等。（时长是可以配置的）

当然，还有其他属性。对于连接池来说，以上几个属性比较重要。

最大的活动的连接数量就是连接池连接数量的上限。默认值10，如果有10个请求正在使用这10个连接，第11个请求只能等待空闲连接。

最大的空闲连接数量。默认值5，如何已经有了5个空闲连接，当第6个连接要空闲下来的时候，连接池会选择关闭该连接对象。来减少数据库的开销。

需要根据系统的并发情况，来合理调整连接池最大连接数以及最多空闲数量。充分发挥数据库连接池的性能。【可以根据实际情况进行测试，然后调整一个合理的数量。】

下图是默认配置：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209291959532.png" style="zoom:45%"/>

在以上配置的基础之上，可以编写java程序测试：

```java
@Test
public void testPool() throws Exception{
    SqlSessionFactoryBuilder sqlSessionFactoryBuilder = new SqlSessionFactoryBuilder();
    SqlSessionFactory sqlSessionFactory = sqlSessionFactoryBuilder.build(Resources.getResourceAsStream("mybatis-config3.xml"));
    for (int i = 0; i < 4; i++) {
        SqlSession sqlSession = sqlSessionFactory.openSession();
        Object selectCarByCarNum = sqlSession.selectOne("selectCarByCarNum");
    }
}
```

```sql
<select id="selectCarByCarNum" resultType="com.powernode.mybatis.pojo.Car">
  select id,car_num carNum,brand,guide_price guidePrice,produce_time produceTime,car_type carType from t_car where car_num = '100'
</select>
```

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209291959788.png" style="zoom:45%"/>

## 4.4 properties

mybatis提供了更加灵活的配置，连接数据库的信息可以单独写到一个属性资源文件中，假设在类的根路径下创建jdbc.properties文件，配置如下：

```properties
jdbc.driver=com.mysql.cj.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/powernode
```

在mybatis核心配置文件中引入并使用：

```xml
<?xml version="1.0" encoding="UTF-8" ?>

<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!--引入外部属性资源文件-->
    <properties resource="jdbc.properties">
        <property name="jdbc.username" value="root"/>
        <property name="jdbc.password" value="root"/>
    </properties>
    <environments default="dev">
        <environment id="dev">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <!--${key}使用-->
                <property name="driver" value="${jdbc.driver}"/>
                <property name="url" value="${jdbc.url}"/>
                <property name="username" value="${jdbc.username}"/>
                <property name="password" value="${jdbc.password}"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <mapper resource="CarMapper.xml"/>
    </mappers>
</configuration>
```

编写Java程序进行测试：

```java
@Test
public void testProperties() throws Exception{
    SqlSessionFactoryBuilder sqlSessionFactoryBuilder = new SqlSessionFactoryBuilder();
    SqlSessionFactory sqlSessionFactory = sqlSessionFactoryBuilder.build(Resources.getResourceAsStream("mybatis-config4.xml"));
    SqlSession sqlSession = sqlSessionFactory.openSession();
    Object car = sqlSession.selectOne("selectCarByCarNum");
    System.out.println(car);
}
```

**properties两个属性：**

​	**resource：这个属性从类的根路径下开始加载。【常用的。】**

​	**url：从指定的url加载，假设文件放在d:/jdbc.properties，这个url可以写成：file:///d:/jdbc.properties。注意是三个斜杠哦。**

注意：如果不知道mybatis-config.xml文件中标签的编写顺序的话，可以有两种方式知道它的顺序：

- 第一种方式：查看dtd约束文件。
- 第二种方式：通过idea的报错提示信息。【一般采用这种方式】

## 4.5 mapper

mapper标签用来指定SQL映射文件的路径，包含多种指定方式，这里先主要看其中两种：

第一种：resource，从类的根路径下开始加载【比url常用】

```xml
<mappers>
  <mapper resource="CarMapper.xml"/>
</mappers>
```

如果是这样写的话，必须保证类的根下有CarMapper.xml文件。

如果类的根路径下有一个包叫做test，CarMapper.xml如果放在test包下的话，这个配置应该是这样写：

```xml
<mappers>
  <mapper resource="test/CarMapper.xml"/>
</mappers>
```

第二种：url，从指定的url位置加载

假设CarMapper.xml文件放在d盘的根下，这个配置就需要这样写：

```xml
<mappers>
  <mapper url="file:///d:/CarMapper.xml"/>
</mappers>
```

**mapper还有其他的指定方式，后面再看！！！**

# 五、手写MyBatis框架（掌握原理）

警示：该部分内容有难度，基础较弱的程序员可能有些部分是听不懂的，如果无法跟下来，可直接跳过，不影响后续知识点的学习。当然，如果你要能够跟下来，必然会让你加深对MyBatis框架的理解。

**我们给自己的框架起个名：GodBatis（起名灵感来源于：my god!!! 我的天呢！）**

## 5.1 dom4j解析XML文件

该部分内容不再赘述，不会解析XML的，请观看老杜前面讲解的dom4j解析XML文件的视频。

模块名：parse-xml-by-dom4j（普通的Java Maven模块）

第一步：引入dom4j的依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>org.group</groupId>
    <artifactId>parse-xml-by-dom4j</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>jar</packaging>
    <dependencies>
        <!--dom4j依赖-->
        <dependency>
            <groupId>org.dom4j</groupId>
            <artifactId>dom4j</artifactId>
            <version>2.1.3</version>
        </dependency>
        <!--jaxen依赖-->
        <dependency>
            <groupId>jaxen</groupId>
            <artifactId>jaxen</artifactId>
            <version>1.2.0</version>
        </dependency>
        <!--junit依赖-->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.13.2</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
    </properties>
</project>
```

第二步：编写配置文件godbatis-config.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>

<configuration>
    <environments default="dev">
        <environment id="dev">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/powernode"/>
                <property name="username" value="root"/>
                <property name="password" value="root"/>
            </dataSource>
        </environment>
        <mappers>
            <mapper resource="sqlmapper.xml"/>
        </mappers>
    </environments>
</configuration>
```

第三步：解析godbatis-config.xml

```java
package com.powernode.dom4j;

import org.dom4j.Document;
import org.dom4j.Element;
import org.dom4j.Node;
import org.dom4j.io.SAXReader;
import org.junit.Test;

import java.util.HashMap;
import java.util.List;
import java.util.Map;

/**
 * 使用dom4j解析XML文件
 */
public class ParseXMLByDom4j {
    @Test
    public void testGodBatisConfig() throws Exception{

        // 读取xml，获取document对象
        SAXReader saxReader = new SAXReader();
        Document document = saxReader.read(Thread.currentThread().getContextClassLoader().getResourceAsStream("godbatis-config.xml"));

        // 获取<environments>标签的default属性的值
        Element environmentsElt = (Element)document.selectSingleNode("/configuration/environments");
        String defaultId = environmentsElt.attributeValue("default");
        System.out.println(defaultId);

        // 获取environment标签
        Element environmentElt = (Element)document.selectSingleNode("/configuration/environments/environment[@id='" + defaultId + "']");

        // 获取事务管理器类型
        Element transactionManager = environmentElt.element("transactionManager");
        String transactionManagerType = transactionManager.attributeValue("type");
        System.out.println(transactionManagerType);

        // 获取数据源类型
        Element dataSource = environmentElt.element("dataSource");
        String dataSourceType = dataSource.attributeValue("type");
        System.out.println(dataSourceType);

        // 将数据源信息封装到Map集合
        Map<String,String> dataSourceMap = new HashMap<>();
        dataSource.elements().forEach(propertyElt -> {
            dataSourceMap.put(propertyElt.attributeValue("name"), propertyElt.attributeValue("value"));
        });

        dataSourceMap.forEach((k, v) -> System.out.println(k + ":" + v));

        // 获取sqlmapper.xml文件的路径
        Element mappersElt = (Element) document.selectSingleNode("/configuration/environments/mappers");
        mappersElt.elements().forEach(mapper -> {
            System.out.println(mapper.attributeValue("resource"));
        });
    }
}
```

执行结果：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292000368.png" style="zoom:40%"/>

第四步：编写配置文件sqlmapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>

<mapper namespace="car">
    <insert id="insertCar">
        insert into t_car(id,car_num,brand,guide_price,produce_time,car_type) values(null,#{carNum},#{brand},#{guidePrice},#{produceTime},#{carType})
    </insert>
    <select id="selectCarByCarNum" resultType="com.powernode.mybatis.pojo.Car">
        select id,car_num carNum,brand,guide_price guidePrice,produce_time produceTime,car_type carType from t_car where car_num = #{carNum}
    </select>
</mapper>
```

第五步：解析sqlmapper.xml

```java
@Test
public void testSqlMapper() throws Exception{
    // 读取xml，获取document对象
    SAXReader saxReader = new SAXReader();
    Document document = saxReader.read(Thread.currentThread().getContextClassLoader().getResourceAsStream("sqlmapper.xml"));

    // 获取namespace
    Element mapperElt = (Element) document.selectSingleNode("/mapper");
    String namespace = mapperElt.attributeValue("namespace");
    System.out.println(namespace);

    // 获取sql id
    mapperElt.elements().forEach(statementElt -> {
        // 标签名
        String name = statementElt.getName();
        System.out.println("name:" + name);
        // 如果是select标签，还要获取它的resultType
        if ("select".equals(name)) {
            String resultType = statementElt.attributeValue("resultType");
            System.out.println("resultType:" + resultType);
        }
        // sql id
        String id = statementElt.attributeValue("id");
        System.out.println("sqlId:" + id);
        // sql语句
        String sql = statementElt.getTextTrim();
        System.out.println("sql:" + sql);
    });
}
```

执行结果：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292000532.png" style="zoom:45%"/>

## 5.2 GodBatis

手写框架之前，如果没有思路，可以先参考一下mybatis的客户端程序，通过客户端程序来逆推需要的类，参考代码：

```java
@Test
public void testInsert(){
    SqlSession sqlSession = null;
    try {
        // 1.创建SqlSessionFactoryBuilder对象
        SqlSessionFactoryBuilder sqlSessionFactoryBuilder = new SqlSessionFactoryBuilder();
        // 2.创建SqlSessionFactory对象
        SqlSessionFactory sqlSessionFactory = sqlSessionFactoryBuilder.build(Resources.getResourceAsStream("mybatis-config.xml"));
        // 3.创建SqlSession对象
        sqlSession = sqlSessionFactory.openSession();
        // 4.执行SQL
        Car car = new Car(null, "111", "宝马X7", "70.3", "2010-10-11", "燃油车");
        int count = sqlSession.insert("insertCar",car);
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

@Test
public void testSelectOne(){
    SqlSession sqlSession = null;
    try {
        // 1.创建SqlSessionFactoryBuilder对象
        SqlSessionFactoryBuilder sqlSessionFactoryBuilder = new SqlSessionFactoryBuilder();
        // 2.创建SqlSessionFactory对象
        SqlSessionFactory sqlSessionFactory = sqlSessionFactoryBuilder.build(Resources.getResourceAsStream("mybatis-config.xml"));
        // 3.创建SqlSession对象
        sqlSession = sqlSessionFactory.openSession();
        // 4.执行SQL
        Car car = (Car)sqlSession.selectOne("selectCarByCarNum", "111");
        System.out.println(car);
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
```

### 第一步：IDEA中创建模块

模块：godbatis（创建普通的Java Maven模块，打包方式jar），引入相关依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.god</groupId>
    <artifactId>godbatis</artifactId>
    <version>1.0.0</version>
    <packaging>jar</packaging>

    <dependencies>
        <!--dom4j依赖-->
        <dependency>
            <groupId>org.dom4j</groupId>
            <artifactId>dom4j</artifactId>
            <version>2.1.3</version>
        </dependency>
        <!--jaxen依赖-->
        <dependency>
            <groupId>jaxen</groupId>
            <artifactId>jaxen</artifactId>
            <version>1.2.0</version>
        </dependency>
        <!--junit依赖-->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.13.2</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
    
    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
    </properties>

</project>
```

### 第二步：资源工具类，方便获取指向配置文件的输入流

```java
package org.god.core;

import java.io.InputStream;

/**
 * 资源工具类
 * @author 老杜
 * @version 1.0
 * @since 1.0
 */
public class Resources {

    /**
     * 从类路径中获取配置文件的输入流
     * @param config
     * @return 输入流，该输入流指向类路径中的配置文件
     */
    public static InputStream getResourcesAsStream(String config){
        return Thread.currentThread().getContextClassLoader().getResourceAsStream(config);
    }
}
```

### 第三步：定义SqlSessionFactoryBuilder类

提供一个无参数构造方法，再提供一个build方法，该build方法要返回SqlSessionFactory对象

```java
package org.god.core;

import java.io.InputStream;

/**
 * SqlSessionFactory对象构建器
 * @author 老杜
 * @version 1.0
 * @since 1.0
 */
public class SqlSessionFactoryBuilder {

    /**
     * 创建构建器对象
     */
    public SqlSessionFactoryBuilder() {
    }


    /**
     * 获取SqlSessionFactory对象
     * 该方法主要功能是：读取godbatis核心配置文件，并构建SqlSessionFactory对象
     * @param inputStream 指向核心配置文件的输入流
     * @return SqlSessionFactory对象
     */
    public SqlSessionFactory build(InputStream inputStream){
        // 解析配置文件，创建数据源对象
        // 解析配置文件，创建事务管理器对象
        // 解析配置文件，获取所有的SQL映射对象
        // 将以上信息封装到SqlSessionFactory对象中
        // 返回
        return null;
    }
}
```

### 第四步：分析SqlSessionFactory类中有哪些属性

- 事务管理器
    - GodJDBCTransaction

- SQL映射对象集合
    - Map<String, GodMappedStatement>


### 第五步：定义GodJDBCTransaction

事务管理器最好是定义一个接口，然后每一个具体的事务管理器都实现这个接口。

```java
package org.god.core;

import java.sql.Connection;

/**
 * 事务管理器接口
 * @author 老杜
 * @version 1.0
 * @since 1.0
 */
public interface TransactionManager {
    /**
     * 提交事务
     */
    void commit();

    /**
     * 回滚事务
     */
    void rollback();

    /**
     * 关闭事务
     */
    void close();

    /**
     * 开启连接
     */
    void openConnection();
    
    /**
     * 获取连接对象
     * @return 连接对象
     */
    Connection getConnection();
}
```

```java
package org.god.core;

import javax.sql.DataSource;
import java.sql.Connection;
import java.sql.SQLException;

/**
 * 事务管理器
 * @author 老杜
 * @version 1.0
 * @since 1.0
 */
public class GodJDBCTransaction implements TransactionManager {
    /**
     * 连接对象，控制事务时需要
     */
    private Connection conn;

    /**
     * 数据源对象
     */
    private DataSource dataSource;

    /**
     * 自动提交标志：
     * true表示自动提交
     * false表示不自动提交
     */
    private boolean autoCommit;

    /**
     * 构造事务管理器对象
     * @param autoCommit
     */
    public GodJDBCTransaction(DataSource dataSource, boolean autoCommit) {
        this.dataSource = dataSource;
        this.autoCommit = autoCommit;
    }

    /**
     * 提交事务
     */
    public void commit(){
        try {
            conn.commit();
        } catch (SQLException e) {
            throw new RuntimeException(e);
        }
    }

    /**
     * 回滚事务
     */
    public void rollback(){
        try {
            conn.rollback();
        } catch (SQLException e) {
            throw new RuntimeException(e);
        }
    }

    @Override
    public void close() {
        try {
            conn.close();
        } catch (SQLException e) {
            throw new RuntimeException(e);
        }
    }

    @Override
    public void openConnection() {
        try {
            this.conn = dataSource.getConnection();
            this.conn.setAutoCommit(this.autoCommit);
        } catch (SQLException e) {
            throw new RuntimeException(e);
        }
    }

    @Override
    public Connection getConnection() {
        return conn;
    }
}
```



### 第六步：事务管理器中需要数据源，定义GodUNPOOLEDDataSource

```java
package org.god.core;

import java.io.PrintWriter;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;
import java.sql.SQLFeatureNotSupportedException;
import java.util.logging.Logger;

/**
 * 数据源实现类，不使用连接池
 * @author 老杜
 * @version 1.0
 * @since 1.0
 */
public class GodUNPOOLEDDataSource implements javax.sql.DataSource{
    private String url;
    private String username;
    private String password;

    public GodUNPOOLEDDataSource(String driver, String url, String username, String password) {
        try {
            // 注册驱动
            Class.forName(driver);
        } catch (ClassNotFoundException e) {
            throw new RuntimeException(e);
        }
        this.url = url;
        this.username = username;
        this.password = password;
    }

    @Override
    public Connection getConnection() throws SQLException {
        return DriverManager.getConnection(url, username, password);
    }

    @Override
    public Connection getConnection(String username, String password) throws SQLException {
        return null;
    }

    @Override
    public PrintWriter getLogWriter() throws SQLException {
        return null;
    }

    @Override
    public void setLogWriter(PrintWriter out) throws SQLException {

    }

    @Override
    public void setLoginTimeout(int seconds) throws SQLException {

    }

    @Override
    public int getLoginTimeout() throws SQLException {
        return 0;
    }

    @Override
    public Logger getParentLogger() throws SQLFeatureNotSupportedException {
        return null;
    }

    @Override
    public <T> T unwrap(Class<T> iface) throws SQLException {
        return null;
    }

    @Override
    public boolean isWrapperFor(Class<?> iface) throws SQLException {
        return false;
    }
}
```



### 第七步：定义GodMappedStatement

```java
package org.god.core;

/**
 * SQL映射实体类
 * @author 老杜
 * @version 1.0
 * @since 1.0
 */
public class GodMappedStatement {
    private String sqlId;
    private String resultType;
    private String sql;
    private String parameterType;

    private String sqlType;

    @Override
    public String toString() {
        return "GodMappedStatement{" +
                "sqlId='" + sqlId + '\'' +
                ", resultType='" + resultType + '\'' +
                ", sql='" + sql + '\'' +
                ", parameterType='" + parameterType + '\'' +
                ", sqlType='" + sqlType + '\'' +
                '}';
    }

    public String getSqlId() {
        return sqlId;
    }

    public void setSqlId(String sqlId) {
        this.sqlId = sqlId;
    }

    public String getResultType() {
        return resultType;
    }

    public void setResultType(String resultType) {
        this.resultType = resultType;
    }

    public String getSql() {
        return sql;
    }

    public void setSql(String sql) {
        this.sql = sql;
    }

    public String getParameterType() {
        return parameterType;
    }

    public void setParameterType(String parameterType) {
        this.parameterType = parameterType;
    }

    public String getSqlType() {
        return sqlType;
    }

    public void setSqlType(String sqlType) {
        this.sqlType = sqlType;
    }

    public GodMappedStatement(String sqlId, String resultType, String sql, String parameterType, String sqlType) {
        this.sqlId = sqlId;
        this.resultType = resultType;
        this.sql = sql;
        this.parameterType = parameterType;
        this.sqlType = sqlType;
    }
}
```



### 第八步：完善SqlSessionFactory类

```java
package org.god.core;

import javax.sql.DataSource;
import java.util.List;
import java.util.Map;

/**
 * SqlSession工厂对象，使用SqlSessionFactory可以获取会话对象
 * @author 老杜
 * @version 1.0
 * @since 1.0
 */
public class SqlSessionFactory {
    private TransactionManager transactionManager;
    private Map<String, GodMappedStatement> mappedStatements;

    public SqlSessionFactory(TransactionManager transactionManager, Map<String, GodMappedStatement> mappedStatements) {
        this.transactionManager = transactionManager;
        this.mappedStatements = mappedStatements;
    }

    public TransactionManager getTransactionManager() {
        return transactionManager;
    }

    public void setTransactionManager(TransactionManager transactionManager) {
        this.transactionManager = transactionManager;
    }

    public Map<String, GodMappedStatement> getMappedStatements() {
        return mappedStatements;
    }

    public void setMappedStatements(Map<String, GodMappedStatement> mappedStatements) {
        this.mappedStatements = mappedStatements;
    }
}
```



### 第九步：完善SqlSessionFactoryBuilder中的build方法

```java
package org.god.core;

import org.dom4j.Document;
import org.dom4j.DocumentException;
import org.dom4j.Element;
import org.dom4j.io.SAXReader;

import javax.sql.DataSource;
import java.io.InputStream;
import java.util.HashMap;
import java.util.Map;

/**
 * SqlSessionFactory对象构建器
 *
 * @author 老杜
 * @version 1.0
 * @since 1.0
 */
public class SqlSessionFactoryBuilder {

    /**
     * 创建构建器对象
     */
    public SqlSessionFactoryBuilder() {
    }


    /**
     * 获取SqlSessionFactory对象
     * 该方法主要功能是：读取godbatis核心配置文件，并构建SqlSessionFactory对象
     *
     * @param inputStream 指向核心配置文件的输入流
     * @return SqlSessionFactory对象
     */
    public SqlSessionFactory build(InputStream inputStream) throws DocumentException {
        SAXReader saxReader = new SAXReader();
        Document document = saxReader.read(inputStream);
        Element environmentsElt = (Element) document.selectSingleNode("/configuration/environments");
        String defaultEnv = environmentsElt.attributeValue("default");
        Element environmentElt = (Element) document.selectSingleNode("/configuration/environments/environment[@id='" + defaultEnv + "']");
        // 解析配置文件，创建数据源对象
        Element dataSourceElt = environmentElt.element("dataSource");
        DataSource dataSource = getDataSource(dataSourceElt);
        // 解析配置文件，创建事务管理器对象
        Element transactionManagerElt = environmentElt.element("transactionManager");
        TransactionManager transactionManager = getTransactionManager(transactionManagerElt, dataSource);
        // 解析配置文件，获取所有的SQL映射对象
        Element mappers = environmentsElt.element("mappers");
        Map<String, GodMappedStatement> mappedStatements = getMappedStatements(mappers);
        // 将以上信息封装到SqlSessionFactory对象中
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactory(transactionManager, mappedStatements);
        // 返回
        return sqlSessionFactory;
    }

    private Map<String, GodMappedStatement> getMappedStatements(Element mappers) {
        Map<String, GodMappedStatement> mappedStatements = new HashMap<>();
        mappers.elements().forEach(mapperElt -> {
            try {
                String resource = mapperElt.attributeValue("resource");
                SAXReader saxReader = new SAXReader();
                Document document = saxReader.read(Resources.getResourcesAsStream(resource));
                Element mapper = (Element) document.selectSingleNode("/mapper");
                String namespace = mapper.attributeValue("namespace");

                mapper.elements().forEach(sqlMapper -> {
                    String sqlId = sqlMapper.attributeValue("id");
                    String sql = sqlMapper.getTextTrim();
                    String parameterType = sqlMapper.attributeValue("parameterType");
                    String resultType = sqlMapper.attributeValue("resultType");
                    String sqlType = sqlMapper.getName().toLowerCase();
                    // 封装GodMappedStatement对象
                    GodMappedStatement godMappedStatement = new GodMappedStatement(sqlId, resultType, sql, parameterType, sqlType);
                    mappedStatements.put(namespace + "." + sqlId, godMappedStatement);
                });

            } catch (DocumentException e) {
                throw new RuntimeException(e);
            }
        });
        return mappedStatements;
    }


    private TransactionManager getTransactionManager(Element transactionManagerElt, DataSource dataSource) {
        String type = transactionManagerElt.attributeValue("type").toUpperCase();
        TransactionManager transactionManager = null;
        if ("JDBC".equals(type)) {
            // 使用JDBC事务
            transactionManager = new GodJDBCTransaction(dataSource, false);
        } else if ("MANAGED".equals(type)) {
            // 事务管理器是交给JEE容器的
        }
        return transactionManager;
    }

    private DataSource getDataSource(Element dataSourceElt) {
        // 获取所有数据源的属性配置
        Map<String, String> dataSourceMap = new HashMap<>();
        dataSourceElt.elements().forEach(propertyElt -> {
            dataSourceMap.put(propertyElt.attributeValue("name"), propertyElt.attributeValue("value"));
        });

        String dataSourceType = dataSourceElt.attributeValue("type").toUpperCase();
        DataSource dataSource = null;
        if ("POOLED".equals(dataSourceType)) {

        } else if ("UNPOOLED".equals(dataSourceType)) {
            dataSource = new GodUNPOOLEDDataSource(dataSourceMap.get("driver"), dataSourceMap.get("url"), dataSourceMap.get("username"), dataSourceMap.get("password"));
        } else if ("JNDI".equals(dataSourceType)) {

        }
        return dataSource;
    }
}
```



### 第十步：在SqlSessionFactory中添加openSession方法

```java
public SqlSession openSession(){
    transactionManager.openConnection();
    SqlSession sqlSession = new SqlSession(transactionManager, mappedStatements);
    return sqlSession;
}
```



### 第十一步：编写SqlSession类中commit rollback close方法

```java
package org.god.core;

import java.sql.SQLException;
import java.util.Map;

/**
 * 数据库会话对象
 * @author 老杜
 * @version 1.0
 * @since 1.0
 */
public class SqlSession {
    private TransactionManager transactionManager;
    private Map<String, GodMappedStatement> mappedStatements;

    public SqlSession(TransactionManager transactionManager, Map<String, GodMappedStatement> mappedStatements) {
        this.transactionManager = transactionManager;
        this.mappedStatements = mappedStatements;
    }

    public void commit(){
        try {
            transactionManager.getConnection().commit();
        } catch (SQLException e) {
            throw new RuntimeException(e);
        }
    }

    public void rollback(){
        try {
            transactionManager.getConnection().rollback();
        } catch (SQLException e) {
            throw new RuntimeException(e);
        }
    }

    public void close(){
        try {
            transactionManager.getConnection().close();
        } catch (SQLException e) {
            throw new RuntimeException(e);
        }
    }
}
```



### 第十二步：编写SqlSession类中的insert方法

```java
/**
 * 插入数据
 *
 * @param sqlId 要执行的sqlId
 * @param obj   插入的数据
 * @return
 */
public int insert(String sqlId, Object obj) {
    GodMappedStatement godMappedStatement = mappedStatements.get(sqlId);
    Connection connection = transactionManager.getConnection();
    // 获取sql语句
    // insert into t_car(id,car_num,brand,guide_price,produce_time,car_type) values(null,#{carNum},#{brand},#{guidePrice},#{produceTime},#{carType})
    String godbatisSql = godMappedStatement.getSql();
    // insert into t_car(id,car_num,brand,guide_price,produce_time,car_type) values(null,?,?,?,?,?)
    String sql = godbatisSql.replaceAll("#\\{[a-zA-Z0-9_\\$]*}", "?");

    // 重点一步
    Map<Integer, String> map = new HashMap<>();
    int index = 1;
    while (godbatisSql.indexOf("#") >= 0) {
        int beginIndex = godbatisSql.indexOf("#") + 2;
        int endIndex = godbatisSql.indexOf("}");
        map.put(index++, godbatisSql.substring(beginIndex, endIndex).trim());
        godbatisSql = godbatisSql.substring(endIndex + 1);
    }

    final PreparedStatement ps;
    try {
        ps = connection.prepareStatement(sql);

        // 给?赋值
        map.forEach((k, v) -> {
            try {
                // 获取java实体类的get方法名
                String getMethodName = "get" + v.toUpperCase().charAt(0) + v.substring(1);
                Method getMethod = obj.getClass().getDeclaredMethod(getMethodName);
                ps.setString(k, getMethod.invoke(obj).toString());
            } catch (Exception e) {
                throw new RuntimeException(e);
            }
        });
        int count = ps.executeUpdate();
        ps.close();
        return count;
    } catch (Exception e) {
        throw new RuntimeException(e);
    }
}
```



### 第十三步：编写SqlSession类中的selectOne方法

```java
/**
 * 插入数据
 *
 * @param sqlId 要执行的sqlId
 * @param obj   插入的数据
 * @return
 */
public int insert(String sqlId, Object obj) {
    GodMappedStatement godMappedStatement = mappedStatements.get(sqlId);
    Connection connection = transactionManager.getConnection();
    // 获取sql语句
    // insert into t_car(id,car_num,brand,guide_price,produce_time,car_type) values(null,#{carNum},#{brand},#{guidePrice},#{produceTime},#{carType})
    String godbatisSql = godMappedStatement.getSql();
    // insert into t_car(id,car_num,brand,guide_price,produce_time,car_type) values(null,?,?,?,?,?)
    String sql = godbatisSql.replaceAll("#\\{[a-zA-Z0-9_\\$]*}", "?");

    // 重点一步
    Map<Integer, String> map = new HashMap<>();
    int index = 1;
    while (godbatisSql.indexOf("#") >= 0) {
        int beginIndex = godbatisSql.indexOf("#") + 2;
        int endIndex = godbatisSql.indexOf("}");
        map.put(index++, godbatisSql.substring(beginIndex, endIndex).trim());
        godbatisSql = godbatisSql.substring(endIndex + 1);
    }

    final PreparedStatement ps;
    try {
        ps = connection.prepareStatement(sql);

        // 给?赋值
        map.forEach((k, v) -> {
            try {
                // 获取java实体类的get方法名
                String getMethodName = "get" + v.toUpperCase().charAt(0) + v.substring(1);
                Method getMethod = obj.getClass().getDeclaredMethod(getMethodName);
                ps.setString(k, getMethod.invoke(obj).toString());
            } catch (Exception e) {
                throw new RuntimeException(e);
            }
        });
        int count = ps.executeUpdate();
        ps.close();
        return count;
    } catch (Exception e) {
        throw new RuntimeException(e);
    }
}
```



## 5.3 GodBatis使用Maven打包

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292001787.png" style="zoom:45%"/>

查看本地仓库中是否已经有jar包：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292001166.png" style="zoom:45%"/>

## 5.4 使用GodBatis

使用GodBatis就和使用MyBatis是一样的。

第一步：准备数据库表t_user

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292002708.png" style="zoom:45%"/>

第二步：创建模块，普通的Java Maven模块：godbatis-test

第三步：引入依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  
  <groupId>com.powernode</groupId>
  <artifactId>godbatis-test</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>jar</packaging>
  
  <dependencies>
    <!--godbatis依赖-->
    <dependency>
      <groupId>org.god</groupId>
      <artifactId>godbatis</artifactId>
      <version>1.0.0</version>
    </dependency>
    <!--mysql-->
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>8.0.30</version>
    </dependency>
    <!--junit-->
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.13.2</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
  
  <properties>
    <maven.compiler.source>17</maven.compiler.source>
    <maven.compiler.target>17</maven.compiler.target>
  </properties>
  
</project>
```

第四步：编写pojo类

```java
package com.powernode.godbatis.pojo;

public class User {
    private String id;
    private String name;
    private String email;
    private String address;

    @Override
    public String toString() {
        return "User{" +
                "id='" + id + '\'' +
                ", name='" + name + '\'' +
                ", email='" + email + '\'' +
                ", address='" + address + '\'' +
                '}';
    }

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }

    public User() {
    }

    public User(String id, String name, String email, String address) {
        this.id = id;
        this.name = name;
        this.email = email;
        this.address = address;
    }
}
```

第五步：编写核心配置文件：godbatis-config.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>

<configuration>
    <environments default="dev">
        <environment id="dev">
            <transactionManager type="JDBC"/>
            <dataSource type="UNPOOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/powernode"/>
                <property name="username" value="root"/>
                <property name="password" value="root"/>
            </dataSource>
        </environment>
        <mappers>
            <mapper resource="UserMapper.xml"/>
        </mappers>
    </environments>
</configuration>
```

第六步：编写sql映射文件：UserMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>

<mapper namespace="user">
    <insert id="insertUser">
        insert into t_user(id,name,email,address) values(#{id},#{name},#{email},#{address})
    </insert>
    <select id="selectUserById" resultType="com.powernode.godbatis.pojo.User">
        select * from t_user where id = #{id}
    </select>
</mapper>
```

第七步：编写测试类

```java
package com.powernode.godbatis.test;

import com.powernode.godbatis.pojo.User;
import org.god.core.Resources;
import org.god.core.SqlSession;
import org.god.core.SqlSessionFactory;
import org.god.core.SqlSessionFactoryBuilder;
import org.junit.Test;

public class GodBatisTest {
    
    @Test
    public void testInsertUser() throws Exception{
        User user = new User("1", "zhangsan", "zhangsan@1234.com", "北京大兴区");
        SqlSessionFactoryBuilder sqlSessionFactoryBuilder = new SqlSessionFactoryBuilder();
        SqlSessionFactory sqlSessionFactory = sqlSessionFactoryBuilder.build(Resources.getResourcesAsStream("godbatis-config.xml"));
        SqlSession sqlSession = sqlSessionFactory.openSession();
        int count = sqlSession.insert("user.insertUser", user);
        System.out.println("插入了几条记录：" + count);
        sqlSession.commit();
        sqlSession.close();
    }
    
    @Test
    public void testSelectUserById() throws Exception{
        SqlSessionFactoryBuilder sqlSessionFactoryBuilder = new SqlSessionFactoryBuilder();
        SqlSessionFactory sqlSessionFactory = sqlSessionFactoryBuilder.build(Resources.getResourcesAsStream("godbatis-config.xml"));
        SqlSession sqlSession = sqlSessionFactory.openSession();
        Object user = sqlSession.selectOne("user.selectUserById", "1");
        System.out.println(user);
        sqlSession.close();
    }
}
```

第八步：运行结果

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292003145.png" style="zoom:45%"/>



## 5.5 总结MyBatis框架的重要实现原理

```xml
<?xml version="1.0" encoding="UTF-8" ?>

<mapper namespace="user">
  <insert id="insertUser">
    insert into t_user(id,name,email,address) values(#{id},#{name},#{email},#{address})
  </insert>
  <select id="selectUserById" resultType="com.powernode.godbatis.pojo.User">
    select id,name,email,address from t_user where id = #{id}
  </select>
</mapper>
```

思考两个问题：

- 为什么insert语句中 #{} 里填写的必须是属性名？
- 为什么select语句查询结果列名要属性名一致？

# 六、在WEB中应用MyBatis（使用MVC架构模式）

**目标：**

- 掌握mybatis在web应用中怎么用
- mybatis三大对象的作用域和生命周期
- ThreadLocal原理及使用
- 巩固MVC架构模式
- 为学习MyBatis的接口代理机制做准备

**实现功能：**

- 银行账户转账

**使用技术：**

- HTML + Servlet + MyBatis

**WEB应用的名称：**

- bank

## 6.1 需求描述

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292004400.png" style="zoom:40%"/>

## 6.2 数据库表的设计和准备数据

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292005984.png" style="zoom:45%"/>



## 6.3 实现步骤

### 第一步：环境搭建

- IDEA中创建Maven WEB应用（**mybatis-004-web**）

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292005087.png" style="zoom:45%"/>

- IDEA配置Tomcat，这里Tomcat使用10+版本。并部署应用到tomcat。

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292006973.png" style="zoom:45%"/>

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292006334.png" style="zoom:45%"/>

- 默认创建的maven web应用没有java和resources目录，包括两种解决方案

    - 第一种：自己手动加上。

        <img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292007215.png" style="zoom:45%"/>

    - 第二种：修改maven-archetype-webapp-1.4.jar中的配置文件

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292010006.png" style="zoom:45%"/>

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292011528.png" style="zoom:45%"/>

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292011016.png" style="zoom:45%"/>

- web.xml文件的版本较低，可以从tomcat10的样例文件中复制，然后修改

```xml
<?xml version="1.0" encoding="UTF-8"?>

<web-app xmlns="https://jakarta.ee/xml/ns/jakartaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="https://jakarta.ee/xml/ns/jakartaee
                      https://jakarta.ee/xml/ns/jakartaee/web-app_5_0.xsd"
         version="5.0"
         metadata-complete="true">
</web-app>
```

- 删除index.jsp文件，因为我们这个项目不使用JSP。只使用html。

- 确定pom.xml文件中的打包方式是war包。

- 引入相关依赖

    - 编译器版本修改为17

    - 引入的依赖包括：mybatis，mysql驱动，junit，logback，servlet。

        ```xml
        <?xml version="1.0" encoding="UTF-8"?>
        
        <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                 xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
            <modelVersion>4.0.0</modelVersion>
            <groupId>com.powernode</groupId>
            <artifactId>mybatis-004-web</artifactId>
            <version>1.0-SNAPSHOT</version>
            <packaging>war</packaging>
            <name>mybatis-004-web</name>
            <url>http://localhost:8080/bank</url>
            <properties>
                <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
                <maven.compiler.source>17</maven.compiler.source>
                <maven.compiler.target>17</maven.compiler.target>
            </properties>
            <dependencies>
                <!--mybatis依赖-->
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
                <!--junit依赖-->
                <dependency>
                    <groupId>junit</groupId>
                    <artifactId>junit</artifactId>
                    <version>4.13.2</version>
                    <scope>test</scope>
                </dependency>
                <!--logback依赖-->
                <dependency>
                    <groupId>ch.qos.logback</groupId>
                    <artifactId>logback-classic</artifactId>
                    <version>1.2.11</version>
                </dependency>
                <!--servlet依赖-->
                <dependency>
                    <groupId>jakarta.servlet</groupId>
                    <artifactId>jakarta.servlet-api</artifactId>
                    <version>5.0.0</version>
                    <scope>provided</scope>
                </dependency>
            </dependencies>
            <build>
                <finalName>mybatis-004-web</finalName>
                <pluginManagement>
                    <plugins>
                        <plugin>
                            <artifactId>maven-clean-plugin</artifactId>
                            <version>3.1.0</version>
                        </plugin>
                        <plugin>
                            <artifactId>maven-resources-plugin</artifactId>
                            <version>3.0.2</version>
                        </plugin>
                        <plugin>
                            <artifactId>maven-compiler-plugin</artifactId>
                            <version>3.8.0</version>
                        </plugin>
                        <plugin>
                            <artifactId>maven-surefire-plugin</artifactId>
                            <version>2.22.1</version>
                        </plugin>
                        <plugin>
                            <artifactId>maven-war-plugin</artifactId>
                            <version>3.2.2</version>
                        </plugin>
                        <plugin>
                            <artifactId>maven-install-plugin</artifactId>
                            <version>2.5.2</version>
                        </plugin>
                        <plugin>
                            <artifactId>maven-deploy-plugin</artifactId>
                            <version>2.8.2</version>
                        </plugin>
                    </plugins>
                </pluginManagement>
            </build>
        </project>
        ```


- 引入相关配置文件，放到resources目录下（全部放到类的根路径下）
    - mybatis-config.xml

    - AccountMapper.xml

    - logback.xml

    - jdbc.properties


```properties
jdbc.driver=com.mysql.cj.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/powernode
jdbc.username=root
jdbc.password=root
```

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <properties resource="jdbc.properties"/>
    <environments default="dev">
        <environment id="dev">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="${jdbc.driver}"/>
                <property name="url" value="${jdbc.url}"/>
                <property name="username" value="${jdbc.username}"/>
                <property name="password" value="${jdbc.password}"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <!--一定要注意这里的路径哦！！！-->
        <mapper resource="AccountMapper.xml"/>
    </mappers>
</configuration>

<?xml version="1.0" encoding="UTF-8" ?>

<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="account">
</mapper>
```

### 第二步：前端页面index.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>银行账户转账</title>
</head>
<body>
<!--/bank是应用的根，部署web应用到tomcat的时候一定要注意这个名字-->
<form action="/bank/transfer" method="post">
    转出账户：<input type="text" name="fromActno"/><br>
    转入账户：<input type="text" name="toActno"/><br>
    转账金额：<input type="text" name="money"/><br>
    <input type="submit" value="转账"/>
</form>
</body>
</html>
```

### 第三步：创建pojo包、service包、dao包、web包、utils包

- com.powernode.bank.pojo
- com.powernode.bank.service
- com.powernode.bank.service.impl
- com.powernode.bank.dao
- com.powernode.bank.dao.impl
- com.powernode.bank.web.controller
- com.powernode.bank.exception
- com.powernode.bank.utils：**将之前编写的SqlSessionUtil工具类拷贝到该包下。**

### 第四步：定义pojo类：Account

```java
package com.powernode.bank.pojo;

/**
 * 银行账户类
 * @author 老杜
 * @version 1.0
 * @since 1.0
 */
public class Account {
    private Long id;
    private String actno;
    private Double balance;

    @Override
    public String toString() {
        return "Account{" +
                "id=" + id +
                ", actno='" + actno + '\'' +
                ", balance=" + balance +
                '}';
    }

    public Account() {
    }

    public Account(Long id, String actno, Double balance) {
        this.id = id;
        this.actno = actno;
        this.balance = balance;
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getActno() {
        return actno;
    }

    public void setActno(String actno) {
        this.actno = actno;
    }

    public Double getBalance() {
        return balance;
    }

    public void setBalance(Double balance) {
        this.balance = balance;
    }
}
```



### 第五步：编写AccountDao接口，以及AccountDaoImpl实现类

分析dao中至少要提供几个方法，才能完成转账：

- 转账前需要查询余额是否充足：selectByActno
- 转账时要更新账户：update

```java
//AccountDao.java
package com.powernode.bank.dao;

import com.powernode.bank.pojo.Account;

/**
 * 账户数据访问对象
 * @author 老杜
 * @version 1.0
 * @since 1.0
 */
public interface AccountDao {

    /**
     * 根据账号获取账户信息
     * @param actno 账号
     * @return 账户信息
     */
    Account selectByActno(String actno);

    /**
     * 更新账户信息
     * @param act 账户信息
     * @return 1表示更新成功，其他值表示失败
     */
    int update(Account act);
}
```

```java
//AcountDaoImpl.java
package com.powernode.bank.dao.impl;

import com.powernode.bank.dao.AccountDao;
import com.powernode.bank.pojo.Account;
import com.powernode.bank.utils.SqlSessionUtil;
import org.apache.ibatis.session.SqlSession;

public class AccountDaoImpl implements AccountDao {
    @Override
    public Account selectByActno(String actno) {
        SqlSession sqlSession = SqlSessionUtil.openSession();
        Account act = (Account)sqlSession.selectOne("selectByActno", actno);
        sqlSession.close();
        return act;
    }

    @Override
    public int update(Account act) {
        SqlSession sqlSession = SqlSessionUtil.openSession();
        int count = sqlSession.update("update", act);
        sqlSession.commit();
        sqlSession.close();
        return count;
    }
}
```



### 第六步：AccountDaoImpl中编写了mybatis代码，需要编写SQL映射文件了

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="account">
    <select id="selectByActno" resultType="com.powernode.bank.pojo.Account">
        select * from t_act where actno = #{actno}
    </select>
    <update id="update">
        update t_act set balance = #{balance} where actno = #{actno}
    </update>
</mapper>
```




### 第七步：编写AccountService接口以及AccountServiceImpl

```java
package com.powernode.bank.exception;

/**
 * 余额不足异常
 * @author 老杜
 * @version 1.0
 * @since 1.0
 */
public class MoneyNotEnoughException extends Exception{
    public MoneyNotEnoughException(){}
    public MoneyNotEnoughException(String msg){ super(msg); }
}
```

```java
package com.powernode.bank.exception;

/**
 * 应用异常
 * @author 老杜
 * @version 1.0
 * @since 1.0
 */
public class AppException extends Exception{
    public AppException(){}
    public AppException(String msg){ super(msg); }
}
```

```java
package com.powernode.bank.service;

import com.powernode.bank.exception.AppException;
import com.powernode.bank.exception.MoneyNotEnoughException;

/**
 * 账户业务类。
 * @author 老杜
 * @version 1.0
 * @since 1.0
 */
public interface AccountService {

    /**
     * 银行账户转正
     * @param fromActno 转出账户
     * @param toActno 转入账户
     * @param money 转账金额
     * @throws MoneyNotEnoughException 余额不足异常
     * @throws AppException App发生异常
     */
    void transfer(String fromActno, String toActno, double money) throws MoneyNotEnoughException, AppException;
}
```

```java
package com.powernode.bank.service.impl;

import com.powernode.bank.dao.AccountDao;
import com.powernode.bank.dao.impl.AccountDaoImpl;
import com.powernode.bank.exception.AppException;
import com.powernode.bank.exception.MoneyNotEnoughException;
import com.powernode.bank.pojo.Account;
import com.powernode.bank.service.AccountService;

public class AccountServiceImpl implements AccountService {

    private AccountDao accountDao = new AccountDaoImpl();

    @Override
    public void transfer(String fromActno, String toActno, double money) throws MoneyNotEnoughException, AppException {
        // 查询转出账户的余额
        Account fromAct = accountDao.selectByActno(fromActno);
        if (fromAct.getBalance() < money) {
            throw new MoneyNotEnoughException("对不起，您的余额不足。");
        }
        try {
            // 程序如果执行到这里说明余额充足
            // 修改账户余额
            Account toAct = accountDao.selectByActno(toActno);
            fromAct.setBalance(fromAct.getBalance() - money);
            toAct.setBalance(toAct.getBalance() + money);
            // 更新数据库
            accountDao.update(fromAct);
            accountDao.update(toAct);
        } catch (Exception e) {
            throw new AppException("转账失败，未知原因！");
        }
    }
}
```



### 第八步：编写AccountController

```java
package com.powernode.bank.web.controller;

import com.powernode.bank.exception.AppException;
import com.powernode.bank.exception.MoneyNotEnoughException;
import com.powernode.bank.service.AccountService;
import com.powernode.bank.service.impl.AccountServiceImpl;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;

/**
 * 账户控制器
 * @author 老杜
 * @version 1.0
 * @since 1.0
 */
@WebServlet("/transfer")
public class AccountController extends HttpServlet {

    private AccountService accountService = new AccountServiceImpl();

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        // 获取响应流
        response.setContentType("text/html;charset=UTF-8");
        PrintWriter out = response.getWriter();
        // 获取账户信息
        String fromActno = request.getParameter("fromActno");
        String toActno = request.getParameter("toActno");
        double money = Integer.parseInt(request.getParameter("money"));
        // 调用业务方法完成转账
        try {
            accountService.transfer(fromActno, toActno, money);
            out.print("<h1>转账成功！！！</h1>");
        } catch (MoneyNotEnoughException e) {
            out.print(e.getMessage());
        } catch (AppException e) {
            out.print(e.getMessage());
        }
    }
}
```

启动服务器，打开浏览器，输入地址：http://localhost:8080/bank，测试：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292012768.png" style="zoom:45%"/>



## 6.4 MyBatis对象作用域以及事务问题

### MyBatis核心对象的作用域

#### SqlSessionFactoryBuilder

这个类可以被实例化、使用和丢弃，一旦创建了 SqlSessionFactory，就不再需要它了。 因此 SqlSessionFactoryBuilder 实例的最佳作用域是方法作用域（也就是局部方法变量）。 你可以重用 SqlSessionFactoryBuilder 来创建多个 SqlSessionFactory 实例，但最好还是不要一直保留着它，以保证所有的 XML 解析资源可以被释放给更重要的事情。

#### SqlSessionFactory

SqlSessionFactory 一旦被创建就应该在应用的运行期间一直存在，没有任何理由丢弃它或重新创建另一个实例。 使用 SqlSessionFactory 的最佳实践是在应用运行期间不要重复创建多次，多次重建 SqlSessionFactory 被视为一种代码“坏习惯”。因此 SqlSessionFactory 的最佳作用域是应用作用域。 有很多方法可以做到，最简单的就是使用单例模式或者静态单例模式。

#### SqlSession

每个线程都应该有它自己的 SqlSession 实例。SqlSession 的实例不是线程安全的，因此是不能被共享的，所以它的最佳的作用域是请求或方法作用域。 绝对不能将 SqlSession 实例的引用放在一个类的静态域，甚至一个类的实例变量也不行。 也绝不能将 SqlSession 实例的引用放在任何类型的托管作用域中，比如 Servlet 框架中的 HttpSession。 如果你现在正在使用一种 Web 框架，考虑将 SqlSession 放在一个和 HTTP 请求相似的作用域中。 换句话说，每次收到 HTTP 请求，就可以打开一个 SqlSession，返回一个响应后，就关闭它。 这个关闭操作很重要，为了确保每次都能执行关闭操作，你应该把这个关闭操作放到 finally 块中。 下面的示例就是一个确保 SqlSession 关闭的标准模式：

```java
try (SqlSession session = sqlSessionFactory.openSession()) {
  // 你的应用逻辑代码
}
```

### 事务问题

在之前的转账业务中，更新了两个账户，我们需要保证它们的同时成功或同时失败，这个时候就需要使用事务机制，在transfer方法开始执行时开启事务，直到两个更新都成功之后，再提交事务，我们尝试将transfer方法进行如下修改：

```java
package com.powernode.bank.service.impl;

import com.powernode.bank.dao.AccountDao;
import com.powernode.bank.dao.impl.AccountDaoImpl;
import com.powernode.bank.exception.AppException;
import com.powernode.bank.exception.MoneyNotEnoughException;
import com.powernode.bank.pojo.Account;
import com.powernode.bank.service.AccountService;
import com.powernode.bank.utils.SqlSessionUtil;
import org.apache.ibatis.session.SqlSession;

public class AccountServiceImpl implements AccountService {

    private AccountDao accountDao = new AccountDaoImpl();

    @Override
    public void transfer(String fromActno, String toActno, double money) throws MoneyNotEnoughException, AppException {
        // 查询转出账户的余额
        Account fromAct = accountDao.selectByActno(fromActno);
        if (fromAct.getBalance() < money) {
            throw new MoneyNotEnoughException("对不起，您的余额不足。");
        }
        try {
            // 程序如果执行到这里说明余额充足
            // 修改账户余额
            Account toAct = accountDao.selectByActno(toActno);
            fromAct.setBalance(fromAct.getBalance() - money);
            toAct.setBalance(toAct.getBalance() + money);
            // 更新数据库（添加事务）
            SqlSession sqlSession = SqlSessionUtil.openSession();
            accountDao.update(fromAct);
            // 模拟异常
            String s = null;
            s.toString();
            accountDao.update(toAct);
            sqlSession.commit();
            sqlSession.close();
        } catch (Exception e) {
            throw new AppException("转账失败，未知原因！");
        }
    }
}
```

运行前注意看数据库表中当前的数据：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292012247.png" style="zoom:45%"/>

执行程序：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292013971.png" style="zoom:45%"/>

再次查看数据库表中的数据：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292013535.png" style="zoom:45%"/>

**傻眼了吧！！！事务出问题了，转账失败了，钱仍然是少了1万。这是什么原因呢？主要是因为service和dao中使用的SqlSession对象不是同一个。**

怎么办？为了保证service和dao中使用的SqlSession对象是同一个，可以将SqlSession对象存放到ThreadLocal当中。修改SqlSessionUtil工具类：

```java
package com.powernode.bank.utils;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;

/**
 * MyBatis工具类
 *
 * @author 老杜
 * @version 1.0
 * @since 1.0
 */
public class SqlSessionUtil {
    private static SqlSessionFactory sqlSessionFactory;

    /**
     * 类加载时初始化sqlSessionFactory对象
     */
    static {
        try {
            SqlSessionFactoryBuilder sqlSessionFactoryBuilder = new SqlSessionFactoryBuilder();
            sqlSessionFactory = sqlSessionFactoryBuilder.build(Resources.getResourceAsStream("mybatis-config.xml"));
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    private static ThreadLocal<SqlSession> local = new ThreadLocal<>();

    /**
     * 每调用一次openSession()可获取一个新的会话，该会话支持自动提交。
     *
     * @return 新的会话对象
     */
    public static SqlSession openSession() {
        SqlSession sqlSession = local.get();
        if (sqlSession == null) {
            sqlSession = sqlSessionFactory.openSession();
            local.set(sqlSession);
        }
        return sqlSession;
    }

    /**
     * 关闭SqlSession对象
     * @param sqlSession
     */
    public static void close(SqlSession sqlSession){
        if (sqlSession != null) {
            sqlSession.close();
        }
        local.remove();
    }
}
```

修改dao中的方法：AccountDaoImpl中所有方法中的提交commit和关闭close代码全部删除。

```java
package com.powernode.bank.dao.impl;

import com.powernode.bank.dao.AccountDao;
import com.powernode.bank.pojo.Account;
import com.powernode.bank.utils.SqlSessionUtil;
import org.apache.ibatis.session.SqlSession;

public class AccountDaoImpl implements AccountDao {
    @Override
    public Account selectByActno(String actno) {
        SqlSession sqlSession = SqlSessionUtil.openSession();
        Account act = (Account)sqlSession.selectOne("account.selectByActno", actno);
        return act;
    }

    @Override
    public int update(Account act) {
        SqlSession sqlSession = SqlSessionUtil.openSession();
        int count = sqlSession.update("account.update", act);
        return count;
    }
}
```

修改service中的方法：

```java
package com.powernode.bank.service.impl;

import com.powernode.bank.dao.AccountDao;
import com.powernode.bank.dao.impl.AccountDaoImpl;
import com.powernode.bank.exception.AppException;
import com.powernode.bank.exception.MoneyNotEnoughException;
import com.powernode.bank.pojo.Account;
import com.powernode.bank.service.AccountService;
import com.powernode.bank.utils.SqlSessionUtil;
import org.apache.ibatis.session.SqlSession;

public class AccountServiceImpl implements AccountService {

    private AccountDao accountDao = new AccountDaoImpl();

    @Override
    public void transfer(String fromActno, String toActno, double money) throws MoneyNotEnoughException, AppException {
        // 查询转出账户的余额
        Account fromAct = accountDao.selectByActno(fromActno);
        if (fromAct.getBalance() < money) {
            throw new MoneyNotEnoughException("对不起，您的余额不足。");
        }
        try {
            // 程序如果执行到这里说明余额充足
            // 修改账户余额
            Account toAct = accountDao.selectByActno(toActno);
            fromAct.setBalance(fromAct.getBalance() - money);
            toAct.setBalance(toAct.getBalance() + money);
            // 更新数据库（添加事务）
            SqlSession sqlSession = SqlSessionUtil.openSession();
            accountDao.update(fromAct);
            // 模拟异常
            String s = null;
            s.toString();
            accountDao.update(toAct);
            sqlSession.commit();
            SqlSessionUtil.close(sqlSession);  // 只修改了这一行代码。
        } catch (Exception e) {
            throw new AppException("转账失败，未知原因！");
        }
    }
}
```

当前数据库表中的数据：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292014993.png" style="zoom:45%"/>

如果余额不足呢：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292015942.png" style="zoom:45%"/>

账户的余额依然正常：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292015667.png" style="zoom:45%"/>

## 6.5 分析当前程序存在的问题

我们来看一下DaoImpl的代码

```java
package com.powernode.bank.dao.impl;

import com.powernode.bank.dao.AccountDao;
import com.powernode.bank.pojo.Account;
import com.powernode.bank.utils.SqlSessionUtil;
import org.apache.ibatis.session.SqlSession;

public class AccountDaoImpl implements AccountDao {
    @Override
    public Account selectByActno(String actno) {
        SqlSession sqlSession = SqlSessionUtil.openSession();
        Account act = (Account)sqlSession.selectOne("account.selectByActno", actno);
        return act;
    }

    @Override
    public int update(Account act) {
        SqlSession sqlSession = SqlSessionUtil.openSession();
        int count = sqlSession.update("account.update", act);
        return count;
    }
}
```

我们不难发现，这个dao实现类中的方法代码很固定，基本上就是一行代码，通过SqlSession对象调用insert、delete、update、select等方法，这个类中的方法没有任何业务逻辑，既然是这样，**这个类我们能不能动态的生成**，以后可以不写这个类吗？答案：可以。
