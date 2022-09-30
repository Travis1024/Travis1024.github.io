---
title: Mybatis Notes <2>
author: Travis <Hongxu Wei>
date: 2022-09-27 00:25:00 +0800
categories: [Java Learning Space]
tags: [Mybatis]
math: false
---

# Mybatis Notes <2>

# 七、使用javassist生成类

来自百度百科：

Javassist是一个开源的分析、编辑和创建Java字节码的类库。是由东京工业大学的数学和计算机科学系的 Shigeru Chiba （千叶 滋）所创建的。它已加入了开放源代码JBoss 应用服务器项目，通过使用Javassist对字节码操作为JBoss实现动态"AOP"框架。

## 7.1 Javassist的使用

我们要使用javassist，首先要引入它的依赖

```xml
<dependency>
  <groupId>org.javassist</groupId>
  <artifactId>javassist</artifactId>
  <version>3.29.1-GA</version>
</dependency>
```

样例代码：

```java
package com.powernode.javassist;

import javassist.ClassPool;
import javassist.CtClass;
import javassist.CtMethod;
import javassist.Modifier;

import java.lang.reflect.Method;

public class JavassistTest {
    public static void main(String[] args) throws Exception {
        // 获取类池
        ClassPool pool = ClassPool.getDefault();
        // 创建类
        CtClass ctClass = pool.makeClass("com.powernode.javassist.Test");
        // 创建方法
        // 1.返回值类型 2.方法名 3.形式参数列表 4.所属类
        CtMethod ctMethod = new CtMethod(CtClass.voidType, "execute", new CtClass[]{}, ctClass);
        // 设置方法的修饰符列表
        ctMethod.setModifiers(Modifier.PUBLIC);
        // 设置方法体
        ctMethod.setBody("{System.out.println(\"hello world\");}");
        // 给类添加方法
        ctClass.addMethod(ctMethod);
        // 调用方法
        Class<?> aClass = ctClass.toClass();
        Object o = aClass.newInstance();
        Method method = aClass.getDeclaredMethod("execute");
        method.invoke(o);
    }
}
```

运行要注意：加两个参数，要不然会有异常。

- --add-opens java.base/java.lang=ALL-UNNAMED
- --add-opens java.base/sun.net.util=ALL-UNNAMED

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292016617.png" style="zoom:45%"/>

运行结果：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292016373.png" style="zoom:45%"/>

## 7.2 使用Javassist生成DaoImpl类

使用Javassist动态生成DaoImpl类

```java
package com.powernode.bank.utils;

import org.apache.ibatis.javassist.CannotCompileException;
import org.apache.ibatis.javassist.ClassPool;
import org.apache.ibatis.javassist.CtClass;
import org.apache.ibatis.javassist.CtMethod;
import org.apache.ibatis.session.SqlSession;

import java.lang.reflect.Constructor;
import java.lang.reflect.Method;
import java.lang.reflect.Modifier;
import java.util.Arrays;

/**
 * 使用javassist库动态生成dao接口的实现类
 *
 * @author 老杜
 * @version 1.0
 * @since 1.0
 */
public class GenerateDaoByJavassist {

    /**
     * 根据dao接口生成dao接口的代理对象
     *
     * @param sqlSession   sql会话
     * @param daoInterface dao接口
     * @return dao接口代理对象
     */
    public static Object getMapper(SqlSession sqlSession, Class daoInterface) {
        ClassPool pool = ClassPool.getDefault();
        // 生成代理类
        CtClass ctClass = pool.makeClass(daoInterface.getPackageName() + ".impl." + daoInterface.getSimpleName() + "Impl");
        // 接口
        CtClass ctInterface = pool.makeClass(daoInterface.getName());
        // 代理类实现接口
        ctClass.addInterface(ctInterface);
        // 获取所有的方法
        Method[] methods = daoInterface.getDeclaredMethods();
        Arrays.stream(methods).forEach(method -> {
            // 拼接方法的签名
            StringBuilder methodStr = new StringBuilder();
            String returnTypeName = method.getReturnType().getName();
            methodStr.append(returnTypeName);
            methodStr.append(" ");
            String methodName = method.getName();
            methodStr.append(methodName);
            methodStr.append("(");
            Class<?>[] parameterTypes = method.getParameterTypes();
            for (int i = 0; i < parameterTypes.length; i++) {
                methodStr.append(parameterTypes[i].getName());
                methodStr.append(" arg");
                methodStr.append(i);
                if (i != parameterTypes.length - 1) {
                    methodStr.append(",");
                }
            }
            methodStr.append("){");
            // 方法体当中的代码怎么写？
            // 获取sqlId（这里非常重要：因为这行代码导致以后namespace必须是接口的全限定接口名，sqlId必须是接口中方法的方法名。）
            String sqlId = daoInterface.getName() + "." + methodName;
            // 获取SqlCommondType
            String sqlCommondTypeName = sqlSession.getConfiguration().getMappedStatement(sqlId).getSqlCommandType().name();
            if ("SELECT".equals(sqlCommondTypeName)) {
                methodStr.append("org.apache.ibatis.session.SqlSession sqlSession = com.powernode.bank.utils.SqlSessionUtil.openSession();");
                methodStr.append("Object obj = sqlSession.selectOne(\"" + sqlId + "\", arg0);");
                methodStr.append("return (" + returnTypeName + ")obj;");
            } else if ("UPDATE".equals(sqlCommondTypeName)) {
                methodStr.append("org.apache.ibatis.session.SqlSession sqlSession = com.powernode.bank.utils.SqlSessionUtil.openSession();");
                methodStr.append("int count = sqlSession.update(\"" + sqlId + "\", arg0);");
                methodStr.append("return count;");
            }
            methodStr.append("}");
            System.out.println(methodStr);
            try {
                // 创建CtMethod对象
                CtMethod ctMethod = CtMethod.make(methodStr.toString(), ctClass);
                ctMethod.setModifiers(Modifier.PUBLIC);
                // 将方法添加到类
                ctClass.addMethod(ctMethod);
            } catch (CannotCompileException e) {
                throw new RuntimeException(e);
            }
        });
        try {
            // 创建代理对象
            Class<?> aClass = ctClass.toClass();
            Constructor<?> defaultCon = aClass.getDeclaredConstructor();
            Object o = defaultCon.newInstance();
            return o;
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
}
```

**修改AccountMapper.xml文件：namespace必须是dao接口的全限定名称，id必须是dao接口中的方法名：**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.powernode.bank.dao.AccountDao">
    <select id="selectByActno" resultType="com.powernode.bank.pojo.Account">
        select * from t_act where actno = #{actno}
    </select>
    <update id="update">
        update t_act set balance = #{balance} where actno = #{actno}
    </update>
</mapper>
```

修改service类中获取dao对象的代码：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292016353.png" style="zoom:45%"/>

启动服务器：**启动过程中显示，tomcat服务器自动添加了以下的两个运行参数。所以不需要再单独配置。**

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292017248.png" style="zoom:45%"/>

测试前数据：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292017334.png" style="zoom:45%"/>

打开浏览器测试：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292018508.png" style="zoom:45%"/>



# 八、MyBatis中接口代理机制及使用

好消息！！！其实以上所讲内容mybatis内部已经实现了。直接调用以下代码即可获取dao接口的代理类：

```java
AccountDao accountDao = (AccountDao)sqlSession.getMapper(AccountDao.class);
```

使用以上代码的前提是：**AccountMapper.xml文件中的namespace必须和dao接口的全限定名称一致，id必须和dao接口中方法名一致。**

将service中获取dao对象的代码再次修改，如下：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292018278.png" style="zoom:45%"/>

测试前数据：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292019564.png" style="zoom:45%"/>

测试后数据：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292019028.png" style="zoom:45%"/>

# 九、MyBatis小技巧

## 9.1 #{}和${}

#{}：先编译sql语句，再给占位符传值，底层是PreparedStatement实现。可以防止sql注入，比较常用。

${}：先进行sql语句拼接，然后再编译sql语句，底层是Statement实现。存在sql注入现象。只有在需要进行sql语句关键字拼接的情况下才会用到。

需求：根据car_type查询汽车

模块名：mybatis-005-antic

### 使用#{}

依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.powernode</groupId>
    <artifactId>mybatis-005-antic</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>jar</packaging>

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
    </dependencies>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
    </properties>

</project>
```

jdbc.properties放在类的根路径下

```properties
jdbc.driver=com.mysql.cj.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/powernode
jdbc.username=root
jdbc.password=root
```

logback.xml，可以拷贝之前的，放到类的根路径下

utils

```java
package com.powernode.mybatis.utils;

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

pojo

```java
package com.powernode.mybatis.pojo;
/**
 * 普通实体类：汽车
 * @author 老杜
 * @version 1.0
 * @since 1.0
 */
public class Car {
    private Long id;
    private String carNum;
    private String brand;
    private Double guidePrice;
    private String produceTime;
    private String carType;
    // 构造方法
    // set get方法
    // toString方法
}
```

mapper接口

```java
package com.powernode.mybatis.mapper;

import com.powernode.mybatis.pojo.Car;

import java.util.List;

/**
 * Car的sql映射对象
 * @author 老杜
 * @version 1.0
 * @since 1.0
 */
public interface CarMapper {

    /**
     * 根据car_num获取Car
     * @param carType
     * @return
     */
    List<Car> selectByCarType(String carType);

}
```

mybatis-config.xml，放在类的根路径下

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
        <mapper resource="CarMapper.xml"/>
    </mappers>
</configuration>
```

CarMapper.xml，放在类的根路径下：**注意namespace必须和接口名一致。id必须和接口中方法名一致**。

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.powernode.mybatis.mapper.CarMapper">
    <select id="selectByCarType" resultType="com.powernode.mybatis.pojo.Car">
        select
            id,car_num as carNum,brand,guide_price as guidePrice,produce_time as produceTime,car_type as carType
        from
            t_car
        where
            car_type = #{carType}
    </select>
</mapper>
```

测试程序

```java
package com.powernode.mybatis.test;

import com.powernode.mybatis.mapper.CarMapper;
import com.powernode.mybatis.pojo.Car;
import com.powernode.mybatis.utils.SqlSessionUtil;
import org.junit.Test;

import java.util.List;

/**
 * CarMapper测试类
 * @author 老杜
 * @version 1.0
 * @since 1.0
 */
public class CarMapperTest {

    @Test
    public void testSelectByCarType(){
        CarMapper mapper = (CarMapper) SqlSessionUtil.openSession().getMapper(CarMapper.class);
        List<Car> cars = mapper.selectByCarType("燃油车");
        cars.forEach(car -> System.out.println(car));
    }
}
```

执行结果：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292020646.png" style="zoom:45%"/>

通过执行可以清楚的看到，sql语句中是带有 ? 的，这个 ? 就是大家在JDBC中所学的占位符，专门用来接收值的。

把“燃油车”以String类型的值，传递给 ?

这就是 #{}，它会先进行sql语句的预编译，然后再给占位符传值

### 使用${}

同样的需求，我们使用${}来完成

CarMapper.xml文件修改如下：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.powernode.mybatis.mapper.CarMapper">
    <select id="selectByCarType" resultType="com.powernode.mybatis.pojo.Car">
        select
            id,car_num as carNum,brand,guide_price as guidePrice,produce_time as produceTime,car_type as carType
        from
            t_car
        where
            <!--car_type = #{carType}-->
            car_type = ${carType}
    </select>
</mapper>
```

再次运行测试程序：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292020587.png" style="zoom:45%"/>

出现异常了，这是为什么呢？看看生成的sql语句：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292021829.png" style="zoom:45%"/>

很显然，${} 是先进行sql语句的拼接，然后再编译，出现语法错误是正常的，因为 燃油车 是一个字符串，在sql语句中应该添加单引号

修改：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.powernode.mybatis.mapper.CarMapper">
    <select id="selectByCarType" resultType="com.powernode.mybatis.pojo.Car">
        select
            id,car_num as carNum,brand,guide_price as guidePrice,produce_time as produceTime,car_type as carType
        from
            t_car
        where
            <!--car_type = #{carType}-->
            <!--car_type = ${carType}-->
            car_type = '${carType}'
    </select>
</mapper>
```

再执行测试程序：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292021467.png" style="zoom:45%"/>

通过以上测试，可以看出，对于以上这种需求来说，还是建议使用 #{} 的方式。

原则：能用 #{} 就不用 ${}

### 什么情况下必须使用${}

当需要进行sql语句关键字拼接的时候。必须使用${}

需求：通过向sql语句中注入asc或desc关键字，来完成数据的升序或降序排列。

- **先使用#{}尝试：**

CarMapper接口：

```java
/**
 * 查询所有的Car
 * @param ascOrDesc asc或desc
 * @return
 */
List<Car> selectAll(String ascOrDesc);
```

CarMapper.xml文件：

```sql
<select id="selectAll" resultType="com.powernode.mybatis.pojo.Car">
  select
  id,car_num as carNum,brand,guide_price as guidePrice,produce_time as produceTime,car_type as carType
  from
  t_car
  order by carNum #{key}
</select>
```


测试程序

```java
@Test
public void testSelectAll(){
    CarMapper mapper = (CarMapper) SqlSessionUtil.openSession().getMapper(CarMapper.class);
    List<Car> cars = mapper.selectAll("desc");
    cars.forEach(car -> System.out.println(car));
}
```

运行：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292021481.png" style="zoom:45%"/>

报错的原因是sql语句不合法，因为采用这种方式传值，最终sql语句会是这样：

select id,car_num as carNum,brand,guide_price as guidePrice,produce_time as produceTime,car_type as carType from t_car order by carNum 'desc'

desc是一个关键字，不能带单引号的，所以在进行sql语句关键字拼接的时候，必须使用${}

- **使用${} 改造**

```xml
<select id="selectAll" resultType="com.powernode.mybatis.pojo.Car">
  select
  id,car_num as carNum,brand,guide_price as guidePrice,produce_time as produceTime,car_type as carType
  from
  t_car
  <!--order by carNum #{key}-->
  order by carNum ${key}
</select>
```


再次执行测试程序：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292022416.png" style="zoom:45%"/>

### 拼接表名

业务背景：实际开发中，有的表数据量非常庞大，可能会采用分表方式进行存储，比如每天生成一张表，表的名字与日期挂钩，例如：2022年8月1日生成的表：t_user20220108。2000年1月1日生成的表：t_user20000101。此时前端在进行查询的时候会提交一个具体的日期，比如前端提交的日期为：2000年1月1日，那么后端就会根据这个日期动态拼接表名为：t_user20000101。有了这个表名之后，将表名拼接到sql语句当中，返回查询结果。那么大家思考一下，拼接表名到sql语句当中应该使用#{} 还是 ${} 呢？

使用#{}会是这样：select * from 't_car'

使用${}会是这样：select * from t_car

```xml
<select id="selectAllByTableName" resultType="car">
  select
  id,car_num as carNum,brand,guide_price as guidePrice,produce_time as produceTime,car_type as carType
  from
  ${tableName}
</select>
```

```java
/**
 * 根据表名查询所有的Car
 * @param tableName
 * @return
 */
List<Car> selectAllByTableName(String tableName);
```

```java
@Test
public void testSelectAllByTableName(){
    CarMapper mapper = SqlSessionUtil.openSession().getMapper(CarMapper.class);
    List<Car> cars = mapper.selectAllByTableName("t_car");
    cars.forEach(car -> System.out.println(car));
}
```

执行结果：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292022314.png" style="zoom:45%"/>

### 批量删除

业务背景：一次删除多条记录。

对应的sql语句：

- delete from t_user where id = 1 or id = 2 or id = 3;
- delete from t_user where id in(1, 2, 3);

假设现在使用in的方式处理，前端传过来的字符串：1, 2, 3

如果使用mybatis处理，应该使用#{} 还是 ${}

使用#{} ：delete from t_user where id in('1,2,3') **执行错误：1292 - Truncated incorrect DOUBLE value: '1,2,3'**

使用${} ：delete from t_user where id in(1, 2, 3)

```java
/**
     * 根据id批量删除
     * @param ids
     * @return
     */
int deleteBatch(String ids);
```

```sql
<delete id="deleteBatch">
  delete from t_car where id in(${ids})
</delete>
```

```java
@Test
public void testDeleteBatch(){
    CarMapper mapper = SqlSessionUtil.openSession().getMapper(CarMapper.class);
    int count = mapper.deleteBatch("1,2,3");
    System.out.println("删除了几条记录：" + count);
    SqlSessionUtil.openSession().commit();
}
```

执行结果：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292023353.png" style="zoom:40%"/>

### 模糊查询

需求：查询奔驰系列的汽车。【只要品牌brand中含有奔驰两个字的都查询出来。】

#### 使用${}

```java
/**
     * 根据品牌进行模糊查询
     * @param likeBrank
     * @return
     */
List<Car> selectLikeByBrand(String likeBrank);
```

```xml
<select id="selectLikeByBrand" resultType="Car">
  select
  id,car_num as carNum,brand,guide_price as guidePrice,produce_time as produceTime,car_type as carType
  from
  t_car
  where
  brand like '%${brand}%'
</select>
```

```java
@Test
public void testSelectLikeByBrand(){
    CarMapper mapper = SqlSessionUtil.openSession().getMapper(CarMapper.class);
    List<Car> cars = mapper.selectLikeByBrand("奔驰");
    cars.forEach(car -> System.out.println(car));
}
```

执行结果：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292024892.png" style="zoom:45%"/>

#### 使用#{}

第一种：concat函数

```sql
<select id="selectLikeByBrand" resultType="Car">
  select
  id,car_num as carNum,brand,guide_price as guidePrice,produce_time as produceTime,car_type as carType
  from
  t_car
  where
  brand like concat('%',#{brand},'%')
</select>
```


执行结果：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292024783.png" style="zoom:45%"/>

第二种：双引号方式

```sql
<select id="selectLikeByBrand" resultType="Car">
  select
  id,car_num as carNum,brand,guide_price as guidePrice,produce_time as produceTime,car_type as carType
  from
  t_car
  where
  brand like "%"#{brand}"%"
</select>
```

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292025323.png" style="zoom:45%"/>

## 9.2 typeAliases

我们来观察一下CarMapper.xml中的配置信息：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.powernode.mybatis.mapper.CarMapper">

    <select id="selectAll" resultType="com.powernode.mybatis.pojo.Car">
        select
            id,car_num as carNum,brand,guide_price as guidePrice,produce_time as produceTime,car_type as carType
        from
            t_car
        order by carNum ${key}
    </select>

    <select id="selectByCarType" resultType="com.powernode.mybatis.pojo.Car">
        select
            id,car_num as carNum,brand,guide_price as guidePrice,produce_time as produceTime,car_type as carType
        from
            t_car
        where
            car_type = '${carType}'
    </select>
</mapper>
```

resultType属性用来指定查询结果集的封装类型，这个名字太长，可以起别名吗？可以。

在mybatis-config.xml文件中使用typeAliases标签来起别名，包括两种方式：

### 第一种方式：typeAlias

```xml
<typeAliases>
  <typeAlias type="com.powernode.mybatis.pojo.Car" alias="Car"/>
</typeAliases>
```

- 首先要注意typeAliases标签的放置位置，如果不清楚的话，可以看看错误提示信息。

- typeAliases标签中的typeAlias可以写多个。

- typeAlias：
    - type属性：指定给哪个类起别名

    - alias属性：别名。
        - alias属性不是必须的，如果缺省的话，type属性指定的类型名的简类名作为别名。

        - alias是大小写不敏感的。也就是说假设alias="Car"，再用的时候，可以CAR，也可以car，也可以Car，都行。


### 第二种方式：package

如果一个包下的类太多，每个类都要起别名，会导致typeAlias标签配置较多，所以mybatis用提供package的配置方式，只需要指定包名，该包下的所有类都自动起别名，别名就是简类名。并且别名不区分大小写。

```xml
<typeAliases>
  <package name="com.powernode.mybatis.pojo"/>
</typeAliases>
```

package也可以配置多个的。

### 在SQL映射文件中用一下

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.powernode.mybatis.mapper.CarMapper">

    <select id="selectAll" resultType="CAR">
        select
            id,car_num as carNum,brand,guide_price as guidePrice,produce_time as produceTime,car_type as carType
        from
            t_car
        order by carNum ${key}
    </select>

    <select id="selectByCarType" resultType="car">
        select
            id,car_num as carNum,brand,guide_price as guidePrice,produce_time as produceTime,car_type as carType
        from
            t_car
        where
            car_type = '${carType}'
    </select>
</mapper>
```

运行测试程序：正常。

## 9.3 mappers

SQL映射文件的配置方式包括四种：

- resource：从类路径中加载
- url：从指定的全限定资源路径中加载
- class：使用映射器接口实现类的完全限定类名
- package：将包内的映射器接口实现全部注册为映射器

### resource

这种方式是从类路径中加载配置文件，所以这种方式要求SQL映射文件必须放在resources目录下或其子目录下。

```xml
<mappers>
  <mapper resource="org/mybatis/builder/AuthorMapper.xml"/>
  <mapper resource="org/mybatis/builder/BlogMapper.xml"/>
  <mapper resource="org/mybatis/builder/PostMapper.xml"/>
</mappers>
```

### url

这种方式显然使用了绝对路径的方式，这种配置对SQL映射文件存放的位置没有要求，随意。

```xml
<mappers>
  <mapper url="file:///var/mappers/AuthorMapper.xml"/>
  <mapper url="file:///var/mappers/BlogMapper.xml"/>
  <mapper url="file:///var/mappers/PostMapper.xml"/>
</mappers>
```

### class

如果使用这种方式必须满足以下条件：

- SQL映射文件和mapper接口放在同一个目录下。
- SQL映射文件的名字也必须和mapper接口名一致。

```xml
<!-- 使用映射器接口实现类的完全限定类名 -->

<mappers>
  <mapper class="org.mybatis.builder.AuthorMapper"/>
  <mapper class="org.mybatis.builder.BlogMapper"/>
  <mapper class="org.mybatis.builder.PostMapper"/>
</mappers>
```

将CarMapper.xml文件移动到和mapper接口同一个目录下：

- 在resources目录下新建：com/powernode/mybatis/mapper【这里千万要注意：**不能这样新建 com.powernode.mybatis.dao**】
- 将CarMapper.xml文件移动到mapper目录下
- 修改mybatis-config.xml文件

```xml
<mappers>
  <mapper class="com.powernode.mybatis.mapper.CarMapper"/>
</mappers>
```

运行程序：正常！！！

### package

如果class较多，可以使用这种package的方式，但前提条件和上一种方式一样。

```xml
<!-- 将包内的映射器接口实现全部注册为映射器 -->

<mappers>
  <package name="com.powernode.mybatis.mapper"/>
</mappers>
```

## 9.4 idea配置文件模板

mybatis-config.xml和SqlMapper.xml文件可以在IDEA中提前创建好模板，以后通过模板创建配置文件。

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292025870.png" style="zoom:45%"/>

## 9.5 插入数据时获取自动生成的主键

前提是：主键是自动生成的。

业务背景：一个用户有多个角色。

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292026336.png" style="zoom:45%"/>

插入一条新的记录之后，自动生成了主键，而这个主键需要在其他表中使用时。

插入一个用户数据的同时需要给该用户分配角色：需要将生成的用户的id插入到角色表的user_id字段上。

第一种方式：可以先插入用户数据，再写一条查询语句获取id，然后再插入user_id字段。【比较麻烦】

第二种方式：mybatis提供了一种方式更加便捷。

```java
/**
     * 获取自动生成的主键
     * @param car
     */
void insertUseGeneratedKeys(Car car);
```

```sql
<insert id="insertUseGeneratedKeys" useGeneratedKeys="true" keyProperty="id">
  insert into t_car(id,car_num,brand,guide_price,produce_time,car_type) values(null,#{carNum},#{brand},#{guidePrice},#{produceTime},#{carType})
</insert>
```

```java
@Test
public void testInsertUseGeneratedKeys(){
    CarMapper mapper = SqlSessionUtil.openSession().getMapper(CarMapper.class);
    Car car = new Car();
    car.setCarNum("5262");
    car.setBrand("BYD汉");
    car.setGuidePrice(30.3);
    car.setProduceTime("2020-10-11");
    car.setCarType("新能源");
    mapper.insertUseGeneratedKeys(car);
    SqlSessionUtil.openSession().commit();
    System.out.println(car.getId());
}
```



# 十、MyBatis参数处理

模块名：mybatis-006-param

表：t_student

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292026676.png" style="zoom:45%"/>

表中现有数据：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292026583.png" style="zoom:45%"/>

pojo类：

```java
package com.powernode.mybatis.pojo;

import java.util.Date;

/**
 * 学生类
 * @author 老杜
 * @version 1.0
 * @since 1.0
 */
public class Student {
    private Long id;
    private String name;
    private Integer age;
    private Double height;
    private Character sex;
    private Date birth;
    // constructor
    // setter and getter
    // toString
}
```



## 10.1 单个简单类型参数

简单类型包括：

- byte short int long float double char
- Byte Short Integer Long Float Double Character
- String
- java.util.Date
- java.sql.Date

需求：根据name查、根据id查、根据birth查、根据sex查

```java
package com.powernode.mybatis.mapper;

import com.powernode.mybatis.pojo.Student;

import java.util.Date;
import java.util.List;

/**
 * 学生数据Sql映射器
 * @author 老杜
 * @version 1.0
 * @since 1.0
 */
public interface StudentMapper {
    /**
     * 根据name查询
     * @param name
     * @return
     */
    List<Student> selectByName(String name);

    /**
     * 根据id查询
     * @param id
     * @return
     */
    Student selectById(Long id);

    /**
     * 根据birth查询
     * @param birth
     * @return
     */
    List<Student> selectByBirth(Date birth);

    /**
     * 根据sex查询
     * @param sex
     * @return
     */
    List<Student> selectBySex(Character sex);
}
```

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.powernode.mybatis.mapper.StudentMapper">
    <select id="selectByName" resultType="student">
        select * from t_student where name = #{name}
    </select>
    <select id="selectById" resultType="student">
        select * from t_student where id = #{id}
    </select>
    <select id="selectByBirth" resultType="student">
        select * from t_student where birth = #{birth}
    </select>
    <select id="selectBySex" resultType="student">
        select * from t_student where sex = #{sex}
    </select>
</mapper>
```

```java
package com.powernode.mybatis.test;

import com.powernode.mybatis.mapper.StudentMapper;
import com.powernode.mybatis.pojo.Student;
import com.powernode.mybatis.utils.SqlSessionUtil;
import org.junit.Test;

import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.List;

public class StudentMapperTest {

    StudentMapper mapper = SqlSessionUtil.openSession().getMapper(StudentMapper.class);

    @Test
    public void testSelectByName(){
        List<Student> students = mapper.selectByName("张三");
        students.forEach(student -> System.out.println(student));
    }
    @Test
    public void testSelectById(){
        Student student = mapper.selectById(2L);
        System.out.println(student);
    }
    @Test
    public void testSelectByBirth(){
        try {
            Date birth = new SimpleDateFormat("yyyy-MM-dd").parse("2022-08-16");
            List<Student> students = mapper.selectByBirth(birth);
            students.forEach(student -> System.out.println(student));
        } catch (ParseException e) {
            throw new RuntimeException(e);
        }

    }
    @Test
    public void testSelectBySex(){
        List<Student> students = mapper.selectBySex('男');
        students.forEach(student -> System.out.println(student));
    }
}
```

通过测试得知，简单类型对于mybatis来说都是可以自动类型识别的：

- 也就是说对于mybatis来说，它是可以自动推断出ps.setXxxx()方法的。ps.setString()还是ps.setInt()。它可以自动推断。

其实SQL映射文件中的配置比较完整的写法是：

```sql
<select id="selectByName" resultType="student" parameterType="java.lang.String">
  select * from t_student where name = #{name, javaType=String, jdbcType=VARCHAR}
</select>
```


其中sql语句中的javaType，jdbcType，以及select标签中的parameterType属性，都是用来帮助mybatis进行类型确定的。不过这些配置多数是可以省略的。因为mybatis它有强大的自动类型推断机制。

- javaType：可以省略
- jdbcType：可以省略
- parameterType：可以省略

**如果参数只有一个的话，#{} 里面的内容就随便写了。对于 ${} 来说，注意加单引号。**

## 10.2 Map参数

需求：根据name和age查询

```java
/**
* 根据name和age查询
* @param paramMap
* @return
*/
List<Student> selectByParamMap(Map<String,Object> paramMap);
```

```java
@Test
public void testSelectByParamMap(){
    // 准备Map
    Map<String,Object> paramMap = new HashMap<>();
    paramMap.put("nameKey", "张三");
    paramMap.put("ageKey", 20);

    List<Student> students = mapper.selectByParamMap(paramMap);
    students.forEach(student -> System.out.println(student));
}
```

```xml
<select id="selectByParamMap" resultType="student">
  select * from t_student where name = #{nameKey} and age = #{ageKey}
</select>
```

测试运行正常。

**这种方式是手动封装Map集合，将每个条件以key和value的形式存放到集合中。然后在使用的时候通过#{map集合的key}来取值。**

## 10.3 实体类参数

需求：插入一条Student数据

```java
/**
 * 保存学生数据
 * @param student
 * @return
 */
int insert(Student student);
```

```sql
<insert id="insert">
  insert into t_student values(null,#{name},#{age},#{height},#{birth},#{sex})
</insert>
```

```java
@Test
public void testInsert(){
    Student student = new Student();
    student.setName("李四");
    student.setAge(30);
    student.setHeight(1.70);
    student.setSex('男');
    student.setBirth(new Date());
    int count = mapper.insert(student);
    SqlSessionUtil.openSession().commit();
}
```

运行正常，数据库中成功添加一条数据。

**这里需要注意的是：#{} 里面写的是属性名字。这个属性名其本质上是：set/get方法名去掉set/get之后的名字。**

## 10.4 多参数

需求：通过name和sex查询

```java
/**
* 根据name和sex查询
* @param name
* @param sex
* @return
*/
List<Student> selectByNameAndSex(String name, Character sex);
```

```java
@Test
public void testSelectByNameAndSex(){
    List<Student> students = mapper.selectByNameAndSex("张三", '女');
    students.forEach(student -> System.out.println(student));
}
```

```sql
<select id="selectByNameAndSex" resultType="student">
  select * from t_student where name = #{name} and sex = #{sex}
</select>
```

执行结果：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292027558.png" style="zoom:45%"/>

异常信息描述了：name参数找不到，可用的参数包括[arg1, arg0, param1, param2]

修改StudentMapper.xml配置文件：尝试使用[arg1, arg0, param1, param2]去参数

```sql
<select id="selectByNameAndSex" resultType="student">
  <!--select * from t_student where name = #{name} and sex = #{sex}-->
  select * from t_student where name = #{arg0} and sex = #{arg1}
</select>
```


运行结果：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292027253.png" style="zoom:45%"/>

再次尝试修改StudentMapper.xml文件

```sql
<select id="selectByNameAndSex" resultType="student">
  <!--select * from t_student where name = #{name} and sex = #{sex}-->
  <!--select * from t_student where name = #{arg0} and sex = #{arg1}-->
  <!--select * from t_student where name = #{param1} and sex = #{param2}-->
  select * from t_student where name = #{arg0} and sex = #{param2}
</select>
```


通过测试可以看到：

- arg0 是第一个参数
- param1是第一个参数
- arg1 是第二个参数
- param2是第二个参数

实现原理：**实际上在mybatis底层会创建一个map集合，以arg0/param1为key，以方法上的参数为value**，例如以下代码：

```java
Map<String,Object> map = new HashMap<>();
map.put("arg0", name);
map.put("arg1", sex);
map.put("param1", name);
map.put("param2", sex);
// 所以可以这样取值：#{arg0} #{arg1} #{param1} #{param2}
// 其本质就是#{map集合的key}
```

注意：**使用mybatis****3.4.2之前的版本时：要用#{0}和#{1}这种形式。**

## 10.5 @Param注解（命名参数）

可以不用arg0 arg1 param1 param2吗？这个map集合的key我们自定义可以吗？当然可以。使用@Param注解即可。这样可以增强可读性。

需求：根据name和age查询

```java
/**
* 根据name和age查询
* @param name
* @param age
* @return
*/
List<Student> selectByNameAndAge(@Param(value="name") String name, @Param("age") int age);
```

```java
@Test
public void testSelectByNameAndAge(){
    List<Student> stus = mapper.selectByNameAndAge("张三", 20);
    stus.forEach(student -> System.out.println(student));
}
```

```sql
<select id="selectByNameAndAge" resultType="student">
  select * from t_student where name = #{name} and age = #{age}
</select>
```

通过测试，一切正常。

核心：@Param("**这里填写的其实就是map集合的key**")

## 10.6 @Param源码分析

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292028971.png" style="zoom:45%"/>



# 十一、MyBatis查询语句专题

模块名：mybatis-007-select

打包方式：jar

引入依赖：mysql驱动依赖、mybatis依赖、logback依赖、junit依赖。

引入配置文件：jdbc.properties、mybatis-config.xml、logback.xml

创建pojo类：Car

创建Mapper接口：CarMapper

创建Mapper接口对应的映射文件：com/powernode/mybatis/mapper/CarMapper.xml

创建单元测试：CarMapperTest

拷贝工具类：SqlSessionUtil

## 11.1 返回Car

当查询的结果，有对应的实体类，并且查询结果只有一条时：

```java
package com.powernode.mybatis.mapper;

import com.powernode.mybatis.pojo.Car;

/**
 * Car SQL映射器
 * @author 老杜
 * @version 1.0
 * @since 1.0
 */
public interface CarMapper {

    /**
     * 根据id主键查询：结果最多只有一条
     * @param id
     * @return
     */
    Car selectById(Long id);
}
```

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.powernode.mybatis.mapper.CarMapper">
    <select id="selectById" resultType="Car">
        select id,car_num carNum,brand,guide_price guidePrice,produce_time produceTime,car_type carType from t_car where id = #{id}
    </select>
</mapper>
```

```java
package com.powernode.mybatis.test;

import com.powernode.mybatis.mapper.CarMapper;
import com.powernode.mybatis.pojo.Car;
import com.powernode.mybatis.utils.SqlSessionUtil;
import org.junit.Test;

public class CarMapperTest {

    @Test
    public void testSelectById(){
        CarMapper mapper = SqlSessionUtil.openSession().getMapper(CarMapper.class);
        Car car = mapper.selectById(35L);
        System.out.println(car);
    }
}
```

执行结果：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292028475.png" style="zoom:45%"/>

**查询结果是一条的话可以使用List集合接收吗？当然可以**。

```java
/**
* 根据id主键查询：结果最多只有一条，可以放到List集合中吗？
* @return
*/
List<Car> selectByIdToList(Long id);
```

```sql
<select id="selectByIdToList" resultType="Car">
  select id,car_num carNum,brand,guide_price guidePrice,produce_time produceTime,car_type carType from t_car where id = #{id}
</select>
```

```java
@Test
public void testSelectByIdToList(){
    CarMapper mapper = SqlSessionUtil.openSession().getMapper(CarMapper.class);
    List<Car> cars = mapper.selectByIdToList(35L);
    System.out.println(cars);
}
```

执行结果：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292031587.png" style="zoom:45%"/>

## 11.2 返回List<Car>

当查询的记录条数是多条的时候，必须使用集合接收。如果使用单个实体类接收会出现异常。

```java
/**
* 查询所有的Car
* @return
*/
List<Car> selectAll();
```

```xml
<select id="selectAll" resultType="Car">
  select id,car_num carNum,brand,guide_price guidePrice,produce_time produceTime,car_type carType from t_car
</select>
```

```java
@Test
public void testSelectAll(){
    CarMapper mapper = SqlSessionUtil.openSession().getMapper(CarMapper.class);
    List<Car> cars = mapper.selectAll();
    cars.forEach(car -> System.out.println(car));
}
```

执行结果：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292031546.png" style="zoom:45%"/>

如果返回多条记录，采用单个实体类接收会怎样？

```java
/**
* 查询多条记录，采用单个实体类接收会怎样？
* @return
*/
Car selectAll2();
```

```xml
<select id="selectAll2" resultType="Car">
  select id,car_num carNum,brand,guide_price guidePrice,produce_time produceTime,car_type carType from t_car
</select>
```

```java
@Test
public void testSelectAll2(){
    CarMapper mapper = SqlSessionUtil.openSession().getMapper(CarMapper.class);
    Car car = mapper.selectAll2();
    System.out.println(car);
}
```

执行结果：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292032443.png" style="zoom:45%"/>

## 11.3 返回Map

当返回的数据，没有合适的实体类对应的话，可以采用Map集合接收。字段名做key，字段值做value。

查询如果可以保证只有一条数据，则返回一个Map集合即可。

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292032062.png" style="zoom:45%"/>

```java
/**
 * 通过id查询一条记录，返回Map集合
 * @param id
 * @return
 */
Map<String, Object> selectByIdRetMap(Long id);
```

```xml
<select id="selectByIdRetMap" resultType="map">
  select id,car_num carNum,brand,guide_price guidePrice,produce_time produceTime,car_type carType from t_car where id = #{id}
</select>
```

**resultMap="map"，这是因为mybatis内置了很多别名。【参见mybatis开发手册】**

```java
@Test
public void testSelectByIdRetMap(){
    CarMapper mapper = SqlSessionUtil.openSession().getMapper(CarMapper.class);
    Map<String,Object> car = mapper.selectByIdRetMap(35L);
    System.out.println(car);
}
```

执行结果：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292033946.png" style="zoom:45%"/>

当然，如果返回一个Map集合，可以将Map集合放到List集合中吗？当然可以，这里就不再测试了。

反过来，如果返回的不是一条记录，是多条记录的话，只采用单个Map集合接收，这样同样会出现之前的异常：**TooManyResultsException**

## 11.4 返回List<Map>

查询结果条数大于等于1条数据，则可以返回一个存储Map集合的List集合。List<Map>等同于List<Car>

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292033179.png" style="zoom:45%"/>

```java
/**
     * 查询所有的Car，返回一个List集合。List集合中存储的是Map集合。
     * @return
     */
List<Map<String,Object>> selectAllRetListMap();
```

```xml
<select id="selectAllRetListMap" resultType="map">
  select id,car_num carNum,brand,guide_price guidePrice,produce_time produceTime,car_type carType from t_car
</select>
```

```java
@Test
public void testSelectAllRetListMap(){
    CarMapper mapper = SqlSessionUtil.openSession().getMapper(CarMapper.class);
    List<Map<String,Object>> cars = mapper.selectAllRetListMap();
    System.out.println(cars);
}
```

执行结果：

[
  {carType=燃油车, carNum=103, guidePrice=50.30, produceTime=2020-10-01, id=33, brand=奔驰E300L}, 
  {carType=电车, carNum=102, guidePrice=30.23, produceTime=2018-09-10, id=34, brand=比亚迪汉}, 
  {carType=燃油车, carNum=103, guidePrice=50.30, produceTime=2020-10-01, id=35, brand=奔驰E300L}, 
  {carType=燃油车, carNum=103, guidePrice=33.23, produceTime=2020-10-11, id=36, brand=奔驰C200},
  ......
]

## 11.5 返回Map<String,Map>

**拿Car的id做key，以后取出对应的Map集合时更方便。**

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292034269.png" style="zoom:45%"/>

```java
/**
     * 获取所有的Car，返回一个Map集合。
     * Map集合的key是Car的id。
     * Map集合的value是对应Car。
     * @return
     */
@MapKey("id")
Map<Long,Map<String,Object>> selectAllRetMap();
```

```xml
<select id="selectAllRetMap" resultType="map">
  select id,car_num carNum,brand,guide_price guidePrice,produce_time produceTime,car_type carType from t_car
</select>
```

```java
@Test
public void testSelectAllRetMap(){
    CarMapper mapper = SqlSessionUtil.openSession().getMapper(CarMapper.class);
    Map<Long,Map<String,Object>> cars = mapper.selectAllRetMap();
    System.out.println(cars);
}
```

执行结果：

{
64={carType=燃油车, carNum=133, guidePrice=50.30, produceTime=2020-01-10, id=64, brand=丰田霸道}, 
66={carType=燃油车, carNum=133, guidePrice=50.30, produceTime=2020-01-10, id=66, brand=丰田霸道}, 
67={carType=燃油车, carNum=133, guidePrice=50.30, produceTime=2020-01-10, id=67, brand=丰田霸道}, 
69={carType=燃油车, carNum=133, guidePrice=50.30, produceTime=2020-01-10, id=69, brand=丰田霸道},
......
}

## 11.6 resultMap结果映射

查询结果的列名和java对象的属性名对应不上怎么办？

- 第一种方式：as 给列起别名
- 第二种方式：使用resultMap进行结果映射
- 第三种方式：是否开启驼峰命名自动映射（配置settings）

### 使用resultMap进行结果映射

```java
/**
* 查询所有Car，使用resultMap进行结果映射
* @return
*/
List<Car> selectAllByResultMap();
```

```xml
<!--
        resultMap:
            id：这个结果映射的标识，作为select标签的resultMap属性的值。
            type：结果集要映射的类。可以使用别名。
-->
<resultMap id="carResultMap" type="car">
  <!--对象的唯一标识，官方解释是：为了提高mybatis的性能。建议写上。-->
  <id property="id" column="id"/>
  <result property="carNum" column="car_num"/>
  <!--当属性名和数据库列名一致时，可以省略。但建议都写上。-->
  <!--javaType用来指定属性类型。jdbcType用来指定列类型。一般可以省略。-->
  <result property="brand" column="brand" javaType="string" jdbcType="VARCHAR"/>
  <result property="guidePrice" column="guide_price"/>
  <result property="produceTime" column="produce_time"/>
  <result property="carType" column="car_type"/>
</resultMap>

<!--resultMap属性的值必须和resultMap标签中id属性值一致。-->
<select id="selectAllByResultMap" resultMap="carResultMap">
  select * from t_car
</select>
```

```java
@Test
public void testSelectAllByResultMap(){
    CarMapper carMapper = SqlSessionUtil.openSession().getMapper(CarMapper.class);
    List<Car> cars = carMapper.selectAllByResultMap();
    System.out.println(cars);
}
```

执行结果正常。

### 是否开启驼峰命名自动映射

使用这种方式的前提是：属性名遵循Java的命名规范，数据库表的列名遵循SQL的命名规范。

Java命名规范：首字母小写，后面每个单词首字母大写，遵循驼峰命名方式。

SQL命名规范：全部小写，单词之间采用下划线分割。

比如以下的对应关系：

| **实体类中的属性名** | **数据库表的列名**  |
| ------------ | ------------ |
| carNum       | car_num      |
| carType      | car_type     |
| produceTime  | produce_time |

如何启用该功能，在mybatis-config.xml文件中进行配置：

```xml
<!--放在properties标签后面-->
<settings>
  <setting name="mapUnderscoreToCamelCase" value="true"/>
</settings>
```

```java
/**
* 查询所有Car，启用驼峰命名自动映射
* @return
*/
List<Car> selectAllByMapUnderscoreToCamelCase();
```

```xml
<select id="selectAllByMapUnderscoreToCamelCase" resultType="Car">
  select * from t_car
</select>
```

```java
@Test
public void testSelectAllByMapUnderscoreToCamelCase(){
    CarMapper carMapper = SqlSessionUtil.openSession().getMapper(CarMapper.class);
    List<Car> cars = carMapper.selectAllByMapUnderscoreToCamelCase();
    System.out.println(cars);
}
```

执行结果正常。

## 11.7 返回总记录条数

需求：查询总记录条数

```java
/**
* 获取总记录条数
* @return
*/
Long selectTotal();
```

```xml
<!--long是别名，可参考mybatis开发手册。-->
<select id="selectTotal" resultType="long">
  select count(*) from t_car
</select>
```

```java
@Test
public void testSelectTotal(){
    CarMapper carMapper = SqlSessionUtil.openSession().getMapper(CarMapper.class);
    Long total = carMapper.selectTotal();
    System.out.println(total);
}
```

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292035165.png" style="zoom:45%"/>



# 十二、动态SQL

有的业务场景，也需要SQL语句进行动态拼接，例如：

- 批量删除

​		delete from t_car where id in(1,2,3,4,5,6,......这里的值是动态的，根据用户选择的id不同，值是不同的);

- 多条件查询

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292123479.png" style="zoom:45%"/>

select * from t_car where brand like '丰田%' and guide_price > 30 and .....;

创建模块：mybatis-008-dynamic-sql

打包方式：jar

引入依赖：mysql驱动依赖、mybatis依赖、junit依赖、logback依赖

pojo：com.powernode.mybatis.pojo.Car

mapper接口：com.powernode.mybatis.mapper.CarMapper

引入配置文件：mybatis-config.xml、jdbc.properties、logback.xml

mapper配置文件：com/powernode/mybatis/mapper/CarMapper.xml

编写测试类：com.powernode.mybatis.test.CarMapperTest

拷贝工具类：SqlSessionUtil

## 12.1 if标签

需求：多条件查询。

可能的条件包括：品牌（brand）、指导价格（guide_price）、汽车类型（car_type）

```java
package com.powernode.mybatis.mapper;

import com.powernode.mybatis.pojo.Car;
import org.apache.ibatis.annotations.Param;

import java.util.List;

public interface CarMapper {


    /**
     * 根据多条件查询Car
     * @param brand
     * @param guidePrice
     * @param carType
     * @return
     */
    List<Car> selectByMultiCondition(@Param("brand") String brand, @Param("guidePrice") Double guidePrice, @Param("carType") String carType);
}
```

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.powernode.mybatis.mapper.CarMapper">

    <select id="selectByMultiCondition" resultType="car">
        select * from t_car where
        <if test="brand != null and brand != ''">
            brand like #{brand}"%"
        </if>
        <if test="guidePrice != null and guidePrice != ''">
            and guide_price >= #{guidePrice}
        </if>
        <if test="carType != null and carType != ''">
            and car_type = #{carType}
        </if>
    </select>

</mapper>
```

```java
package com.powernode.mybatis.test;

import com.powernode.mybatis.mapper.CarMapper;
import com.powernode.mybatis.pojo.Car;
import com.powernode.mybatis.utils.SqlSessionUtil;
import org.junit.Test;

import java.util.List;

public class CarMapperTest {
    @Test
    public void testSelectByMultiCondition(){
        CarMapper mapper = SqlSessionUtil.openSession().getMapper(CarMapper.class);
        List<Car> cars = mapper.selectByMultiCondition("丰田", 20.0, "燃油车");
        System.out.println(cars);
    }
}
```

执行结果：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292125088.png" style="zoom:45%"/>

如果第一个条件为空，剩下两个条件不为空，会是怎样呢？

```java
List<Car> cars = mapper.selectByMultiCondition("", 20.0, "燃油车");
```

执行结果：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292126748.png" style="zoom:45%"/>

报错了，SQL语法有问题，where后面出现了and。这该怎么解决呢？

- 可以where后面添加一个恒成立的条件。

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292126880.png" style="zoom:45%"/>

执行结果：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292127842.png" style="zoom:45%"/>

如果三个条件都是空，有影响吗？

```java
List<Car> cars = mapper.selectByMultiCondition("", null, "");
```

执行结果：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292128226.png" style="zoom:45%"/>

三个条件都不为空呢？

```java
List<Car> cars = mapper.selectByMultiCondition("丰田", 20.0, "燃油车");
```

执行结果：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292128140.png" style="zoom:45%"/>

## 12.2 where标签

where标签的作用：让where子句更加动态智能。

- 所有条件都为空时，where标签保证不会生成where子句。
- 自动去除某些条件**前面**多余的and或or。

继续使用if标签中的需求。

```java
/**
* 根据多条件查询Car，使用where标签
* @param brand
* @param guidePrice
* @param carType
* @return
*/
List<Car> selectByMultiConditionWithWhere(@Param("brand") String brand, @Param("guidePrice") Double guidePrice, @Param("carType") String carType);
```

```sql
<select id="selectByMultiConditionWithWhere" resultType="car">
  select * from t_car
  <where>
    <if test="brand != null and brand != ''">
      and brand like #{brand}"%"
    </if>
    <if test="guidePrice != null and guidePrice != ''">
      and guide_price >= #{guidePrice}
    </if>
    <if test="carType != null and carType != ''">
      and car_type = #{carType}
    </if>
  </where>
</select>
```

```java
@Test
public void testSelectByMultiConditionWithWhere(){
    CarMapper mapper = SqlSessionUtil.openSession().getMapper(CarMapper.class);
    List<Car> cars = mapper.selectByMultiConditionWithWhere("丰田", 20.0, "燃油车");
    System.out.println(cars);
}
```

运行结果：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292129604.png" style="zoom:45%"/>

如果所有条件都是空呢？

```java
List<Car> cars = mapper.selectByMultiConditionWithWhere("", null, "");
```

运行结果：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292130046.png" style="zoom:45%"/>

它可以自动去掉前面多余的and，那可以自动去掉前面多余的or吗？

```java
List<Car> cars = mapper.selectByMultiConditionWithWhere("丰田", 20.0, "燃油车");
```

```sql
<select id="selectByMultiConditionWithWhere" resultType="car">
  select * from t_car
  <where>
    <if test="brand != null and brand != ''">
      or brand like #{brand}"%"
    </if>
    <if test="guidePrice != null and guidePrice != ''">
      and guide_price >= #{guidePrice}
    </if>
    <if test="carType != null and carType != ''">
      and car_type = #{carType}
    </if>
  </where>
</select>
```


执行结果：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292130717.png" style="zoom:45%"/>

它可以自动去掉前面多余的and，那可以自动去掉后面多余的and吗？

```xml
<select id="selectByMultiConditionWithWhere" resultType="car">
  select * from t_car
  <where>
    <if test="brand != null and brand != ''">
      brand like #{brand}"%" and
    </if>
    <if test="guidePrice != null and guidePrice != ''">
      guide_price >= #{guidePrice} and
    </if>
    <if test="carType != null and carType != ''">
      car_type = #{carType}
    </if>
  </where>
</select>
```

```xml
// 让最后一个条件为空
List<Car> cars = mapper.selectByMultiConditionWithWhere("丰田", 20.0, "");
```

运行结果：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292131123.png" style="zoom:45%"/>

很显然，后面多余的and是不会被去除的。

## 12.3 trim标签

trim标签的属性：

- prefix：在trim标签中的语句前**添加**内容
- suffix：在trim标签中的语句后**添加**内容
- prefixOverrides：前缀**覆盖掉（去掉）**
- suffixOverrides：后缀**覆盖掉（去掉）**

```java
/**
* 根据多条件查询Car，使用trim标签
* @param brand
* @param guidePrice
* @param carType
* @return
*/
List<Car> selectByMultiConditionWithTrim(@Param("brand") String brand, @Param("guidePrice") Double guidePrice, @Param("carType") String carType);
```

```sql
<select id="selectByMultiConditionWithTrim" resultType="car">
  select * from t_car
  <trim prefix="where" suffixOverrides="and|or">
    <if test="brand != null and brand != ''">
      brand like #{brand}"%" and
    </if>
    <if test="guidePrice != null and guidePrice != ''">
      guide_price >= #{guidePrice} and
    </if>
    <if test="carType != null and carType != ''">
      car_type = #{carType}
    </if>
  </trim>
</select>
```

```java
@Test
public void testSelectByMultiConditionWithTrim(){
    CarMapper mapper = SqlSessionUtil.openSession().getMapper(CarMapper.class);
    List<Car> cars = mapper.selectByMultiConditionWithTrim("丰田", 20.0, "");
    System.out.println(cars);
}
```

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292131563.png" style="zoom:45%"/>

如果所有条件为空，where会被加上吗？

```xml
List<Car> cars = mapper.selectByMultiConditionWithTrim("", null, "");
```

执行结果：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292132087.png" style="zoom:45%"/>

## 12.4 set标签

主要使用在update语句当中，用来生成set关键字，同时去掉最后多余的“,”

比如我们只更新提交的不为空的字段，如果提交的数据是空或者""，那么这个字段我们将不更新。

```java
/**
* 更新信息，使用set标签
* @param car
* @return
*/
int updateWithSet(Car car);
```

```sql
<update id="updateWithSet">
  update t_car
  <set>
    <if test="carNum != null and carNum != ''">car_num = #{carNum},</if>
    <if test="brand != null and brand != ''">brand = #{brand},</if>
    <if test="guidePrice != null and guidePrice != ''">guide_price = #{guidePrice},</if>
    <if test="produceTime != null and produceTime != ''">produce_time = #{produceTime},</if>
    <if test="carType != null and carType != ''">car_type = #{carType},</if>
  </set>
  where id = #{id}
</update>
```

```java
@Test
public void testUpdateWithSet(){
    CarMapper mapper = SqlSessionUtil.openSession().getMapper(CarMapper.class);
    Car car = new Car(38L,"1001","丰田霸道2",10.0,"",null);
    int count = mapper.updateWithSet(car);
    System.out.println(count);
    SqlSessionUtil.openSession().commit();
}
```

执行结果：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292132530.png" style="zoom:45%"/>

## 12.5 choose when otherwise

这三个标签是在一起使用的：

```sql
<choose>
  <when></when>
  <when></when>
  <when></when>
  <otherwise></otherwise>
</choose>
```

等同于：

```java
if(){

}else if(){

}else if(){

}else if(){

}else{
}
```

只有一个分支会被选择！！！！

需求：先根据品牌查询，如果没有提供品牌，再根据指导价格查询，如果没有提供指导价格，就根据生产日期查询。

```java
/**
* 使用choose when otherwise标签查询
* @param brand
* @param guidePrice
* @param produceTime
* @return
*/
List<Car> selectWithChoose(@Param("brand") String brand, @Param("guidePrice") Double guidePrice, @Param("produceTime") String produceTime);
```

```xml
<select id="selectWithChoose" resultType="car">
  select * from t_car
  <where>
    <choose>
      <when test="brand != null and brand != ''">
        brand like #{brand}"%"
      </when>
      <when test="guidePrice != null and guidePrice != ''">
        guide_price >= #{guidePrice}
      </when>
      <otherwise>
        produce_time >= #{produceTime}
      </otherwise>
    </choose>
  </where>
</select>
```

```java
@Test
public void testSelectWithChoose(){
    CarMapper mapper = SqlSessionUtil.openSession().getMapper(CarMapper.class);
    //List<Car> cars = mapper.selectWithChoose("丰田霸道", 20.0, "2000-10-10");
    //List<Car> cars = mapper.selectWithChoose("", 20.0, "2000-10-10");
    //List<Car> cars = mapper.selectWithChoose("", null, "2000-10-10");
    List<Car> cars = mapper.selectWithChoose("", null, "");
    System.out.println(cars);
}
```

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292133435.png" style="zoom:45%"/>

## 12.6 foreach标签

循环数组或集合，动态生成sql，比如这样的SQL：

```sql
delete from t_car where id in(1,2,3);
delete from t_car where id = 1 or id = 2 or id = 3;
```

```sql
insert into t_car values
  (null,'1001','凯美瑞',35.0,'2010-10-11','燃油车'),
  (null,'1002','比亚迪唐',31.0,'2020-11-11','新能源'),
  (null,'1003','比亚迪宋',32.0,'2020-10-11','新能源')
```

### 批量删除

- 用in来删除

```java
/**
* 通过foreach完成批量删除
* @param ids
* @return
*/
int deleteBatchByForeach(@Param("ids") Long[] ids);
```

```xml
<!--
collection：集合或数组
item：集合或数组中的元素
separator：分隔符
open：foreach标签中所有内容的开始
close：foreach标签中所有内容的结束
-->
<delete id="deleteBatchByForeach">
  delete from t_car where id in
  <foreach collection="ids" item="id" separator="," open="(" close=")">
    #{id}
  </foreach>
</delete>
```

```java
@Test
public void testDeleteBatchByForeach(){
    CarMapper mapper = SqlSessionUtil.openSession().getMapper(CarMapper.class);
    int count = mapper.deleteBatchByForeach(new Long[]{40L, 41L, 42L});
    System.out.println("删除了几条记录：" + count);
    SqlSessionUtil.openSession().commit();
}
```

执行结果：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292133420.png" style="zoom:45%"/>

- 用or来删除

```java
/**
* 通过foreach完成批量删除
* @param ids
* @return
*/
int deleteBatchByForeach2(@Param("ids") Long[] ids);
```

```sql
<delete id="deleteBatchByForeach2">
  delete from t_car where
  <foreach collection="ids" item="id" separator="or">
    id = #{id}
  </foreach>
</delete>
```

```java
@Test
public void testDeleteBatchByForeach2(){
    CarMapper mapper = SqlSessionUtil.openSession().getMapper(CarMapper.class);
    int count = mapper.deleteBatchByForeach2(new Long[]{40L, 41L, 42L});
    System.out.println("删除了几条记录：" + count);
    SqlSessionUtil.openSession().commit();
}
```

执行结果：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292134824.png" style="zoom:45%"/>

### 批量添加

```java
/**
* 批量添加，使用foreach标签
* @param cars
* @return
*/
int insertBatchByForeach(@Param("cars") List<Car> cars);
```

```sql
<insert id="insertBatchByForeach">
  insert into t_car values 
  <foreach collection="cars" item="car" separator=",">
    (null,#{car.carNum},#{car.brand},#{car.guidePrice},#{car.produceTime},#{car.carType})
  </foreach>
</insert>
```

```java
@Test
public void testInsertBatchByForeach(){
    CarMapper mapper = SqlSessionUtil.openSession().getMapper(CarMapper.class);
    Car car1 = new Car(null, "2001", "兰博基尼", 100.0, "1998-10-11", "燃油车");
    Car car2 = new Car(null, "2001", "兰博基尼", 100.0, "1998-10-11", "燃油车");
    Car car3 = new Car(null, "2001", "兰博基尼", 100.0, "1998-10-11", "燃油车");
    List<Car> cars = Arrays.asList(car1, car2, car3);
    int count = mapper.insertBatchByForeach(cars);
    System.out.println("插入了几条记录" + count);
    SqlSessionUtil.openSession().commit();
}
```

执行结果：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292134987.png" style="zoom:45%"/>

## 12.7 sql标签与include标签

sql标签用来声明sql片段

include标签用来将声明的sql片段包含到某个sql语句当中

作用：代码复用。易维护。

```xml
<sql id="carCols">id,car_num carNum,brand,guide_price guidePrice,produce_time produceTime,car_type carType</sql>

<select id="selectAllRetMap" resultType="map">
  select <include refid="carCols"/> from t_car
</select>

<select id="selectAllRetListMap" resultType="map">
  select <include refid="carCols"/> carType from t_car
</select>

<select id="selectByIdRetMap" resultType="map">
  select <include refid="carCols"/> from t_car where id = #{id}
</select>
```




# 十三、MyBatis的高级映射及延迟加载

模块名：mybatis-009-advanced-mapping

打包方式：jar

依赖：mybatis依赖、mysql驱动依赖、junit依赖、logback依赖

配置文件：mybatis-config.xml、logback.xml、jdbc.properties

拷贝工具类：SqlSessionUtil

准备数据库表：一个班级对应多个学生。班级表：t_clazz。学生表：t_student

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292135585.png" style="zoom:45%"/>



创建pojo：Student、Clazz

```java
package com.powernode.mybatis.pojo;

/**
 * 学生类
 * @author 老杜
 * @version 1.0
 * @since 1.0
 */
public class Student {
    private Integer sid;
    private String sname;
    //......
}
```

```java
package com.powernode.mybatis.pojo;

/**
 * 班级类
 * @author 老杜
 * @version 1.0
 * @since 1.0
 */
public class Clazz {
    private Integer cid;
    private String cname;
    //......
}
```

创建mapper接口：StudentMapper、ClazzMapper

创建mapper映射文件：StudentMapper.xml、ClazzMapper.xml

## 13.1 多对一

多种方式，常见的包括三种：

- 第一种方式：一条SQL语句，级联属性映射。
- 第二种方式：一条SQL语句，association。
- 第三种方式：两条SQL语句，分步查询。（这种方式常用：优点一是可复用。优点二是支持懒加载。）

### 第一种方式：级联属性映射

pojo类Student中添加一个属性：Clazz clazz; 表示学生关联的班级对象。

```java
package com.powernode.mybatis.pojo;

/**
 * 学生类
 * @author 老杜
 * @version 1.0
 * @since 1.0
 */
public class Student {
    private Integer sid;
    private String sname;
    private Clazz clazz;

    public Clazz getClazz() {
        return clazz;
    }

    public void setClazz(Clazz clazz) {
        this.clazz = clazz;
    }

    @Override
    public String toString() {
        return "Student{" +
                "sid=" + sid +
                ", sname='" + sname + '\'' +
                ", clazz=" + clazz +
                '}';
    }

    public Student() {
    }

    public Student(Integer sid, String sname) {
        this.sid = sid;
        this.sname = sname;
    }

    public Integer getSid() {
        return sid;
    }

    public void setSid(Integer sid) {
        this.sid = sid;
    }

    public String getSname() {
        return sname;
    }

    public void setSname(String sname) {
        this.sname = sname;
    }
}
```

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.powernode.mybatis.mapper.StudentMapper">

    <resultMap id="studentResultMap" type="Student">
        <id property="sid" column="sid"/>
        <result property="sname" column="sname"/>
        <result property="clazz.cid" column="cid"/>
        <result property="clazz.cname" column="cname"/>
    </resultMap>

    <select id="selectBySid" resultMap="studentResultMap">
        select s.*, c.* from t_student s join t_clazz c on s.cid = c.cid where sid = #{sid}
    </select>

</mapper>
```

```java
package com.powernode.mybatis.test;

import com.powernode.mybatis.mapper.StudentMapper;
import com.powernode.mybatis.pojo.Student;
import com.powernode.mybatis.utils.SqlSessionUtil;
import org.junit.Test;

public class StudentMapperTest {
    @Test
    public void testSelectBySid(){
        StudentMapper mapper = SqlSessionUtil.openSession().getMapper(StudentMapper.class);
        Student student = mapper.selectBySid(1);
        System.out.println(student);
    }
}
```

执行结果：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292136507.png" style="zoom:45%"/>

### 第二种方式：association

其他位置都不需要修改，只需要修改resultMap中的配置：association即可。

```xml
<resultMap id="studentResultMap" type="Student">
  <id property="sid" column="sid"/>
  <result property="sname" column="sname"/>
  <association property="clazz" javaType="Clazz">
    <id property="cid" column="cid"/>
    <result property="cname" column="cname"/>
  </association>
</resultMap>
```

association翻译为：关联。

学生对象关联一个班级对象。

### 第三种方式：分步查询

其他位置不需要修改，只需要修改以及添加以下三处：

第一处：association中select位置填写sqlId。sqlId=namespace+id。其中column属性作为这条子sql语句的条件。

```xml
<resultMap id="studentResultMap" type="Student">
  <id property="sid" column="sid"/>
  <result property="sname" column="sname"/>
  <association property="clazz"
               select="com.powernode.mybatis.mapper.ClazzMapper.selectByCid"
               column="cid"/>
</resultMap>

<select id="selectBySid" resultMap="studentResultMap">
  select s.* from t_student s where sid = #{sid}
</select>
```


第二处：在ClazzMapper接口中添加方法

```java
package com.powernode.mybatis.mapper;

import com.powernode.mybatis.pojo.Clazz;

/**
 * Clazz映射器接口
 * @author 老杜
 * @version 1.0
 * @since 1.0
 */
public interface ClazzMapper {

    /**
     * 根据cid获取Clazz信息
     * @param cid
     * @return
     */
    Clazz selectByCid(Integer cid);
}
```

第三处：在ClazzMapper.xml文件中进行配置

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.powernode.mybatis.mapper.ClazzMapper">
    <select id="selectByCid" resultType="Clazz">
        select * from t_clazz where cid = #{cid}
    </select>
</mapper>
```

执行结果，可以很明显看到先后有两条sql语句执行：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292136839.png" style="zoom:45%"/>

分步优点：

- 第一个优点：代码复用性增强。
- 第二个优点：支持延迟加载。【暂时访问不到的数据可以先不查询。提高程序的执行效率。】

## 13.2 多对一延迟加载

要想支持延迟加载，非常简单，只需要在association标签中添加fetchType="lazy"即可。

修改StudentMapper.xml文件：

```xml
<resultMap id="studentResultMap" type="Student">
  <id property="sid" column="sid"/>
  <result property="sname" column="sname"/>
  <association property="clazz"
               select="com.powernode.mybatis.mapper.ClazzMapper.selectByCid"
               column="cid"
               fetchType="lazy"/>
</resultMap>
```

我们现在只查询学生名字，修改测试程序：

```java
public class StudentMapperTest {
    @Test
    public void testSelectBySid(){
        StudentMapper mapper = SqlSessionUtil.openSession().getMapper(StudentMapper.class);
        Student student = mapper.selectBySid(1);
        //System.out.println(student);
        // 只获取学生姓名
        String sname = student.getSname();
        System.out.println("学生姓名：" + sname);
    }
}
```

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292137725.png" style="zoom:45%"/>

如果后续需要使用到学生所在班级的名称，这个时候才会执行关联的sql语句，修改测试程序：

```java
public class StudentMapperTest {
    @Test
    public void testSelectBySid(){
        StudentMapper mapper = SqlSessionUtil.openSession().getMapper(StudentMapper.class);
        Student student = mapper.selectBySid(1);
        //System.out.println(student);
        // 只获取学生姓名
        String sname = student.getSname();
        System.out.println("学生姓名：" + sname);
        // 到这里之后，想获取班级名字了
        String cname = student.getClazz().getCname();
        System.out.println("学生的班级名称：" + cname);
    }
}
```

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292137047.png" style="zoom:45%"/>

通过以上的执行结果可以看到，只有当使用到班级名称之后，才会执行关联的sql语句，这就是延迟加载。

在mybatis中如何开启全局的延迟加载呢？需要setting配置，如下：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292138903.png" style="zoom:45%"/>

```xml
<settings>
  <setting name="lazyLoadingEnabled" value="true"/>
</settings>
```

**把fetchType="lazy"去掉。**

执行以下程序：

```java
public class StudentMapperTest {
    @Test
    public void testSelectBySid(){
        StudentMapper mapper = SqlSessionUtil.openSession().getMapper(StudentMapper.class);
        Student student = mapper.selectBySid(1);
        //System.out.println(student);
        // 只获取学生姓名
        String sname = student.getSname();
        System.out.println("学生姓名：" + sname);
        // 到这里之后，想获取班级名字了
        String cname = student.getClazz().getCname();
        System.out.println("学生的班级名称：" + cname);
    }
}
```

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292140771.png" style="zoom:45%"/>

通过以上的测试可以看出，我们已经开启了全局延迟加载策略。

开启全局延迟加载之后，所有的sql都会支持延迟加载，如果某个sql你不希望它支持延迟加载怎么办呢？将fetchType设置为eager：

```xml
<resultMap id="studentResultMap" type="Student">
  <id property="sid" column="sid"/>
  <result property="sname" column="sname"/>
  <association property="clazz"
               select="com.powernode.mybatis.mapper.ClazzMapper.selectByCid"
               column="cid"
               fetchType="eager"/>
</resultMap>
```

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292140540.png" style="zoom:45%"/>

这样的话，针对某个特定的sql，你就关闭了延迟加载机制。

后期我们要不要开启延迟加载机制，主要看实际的业务需求是怎样的。

## 13.3 一对多

一对多的实现，通常是在一的一方中有List集合属性。

在Clazz类中添加List<Student> stus; 属性。

```java
public class Clazz {
    private Integer cid;
    private String cname;
    private List<Student> stus;
    // set get方法
    // 构造方法
    // toString方法
}
```

一对多的实现通常包括两种实现方式：

- 第一种方式：collection
- 第二种方式：分步查询

### 第一种方式：collection

```java
package com.powernode.mybatis.mapper;

import com.powernode.mybatis.pojo.Clazz;

/**
 * Clazz映射器接口
 * @author 老杜
 * @version 1.0
 * @since 1.0
 */
public interface ClazzMapper {

    /**
     * 根据cid获取Clazz信息
     * @param cid
     * @return
     */
    Clazz selectByCid(Integer cid);

    /**
     * 根据班级编号查询班级信息。同时班级中所有的学生信息也要查询。
     * @param cid
     * @return
     */
    Clazz selectClazzAndStusByCid(Integer cid);
}
```

```xml
<resultMap id="clazzResultMap" type="Clazz">
  <id property="cid" column="cid"/>
  <result property="cname" column="cname"/>
  <collection property="stus" ofType="Student">
    <id property="sid" column="sid"/>
    <result property="sname" column="sname"/>
  </collection>
</resultMap>

<select id="selectClazzAndStusByCid" resultMap="clazzResultMap">
  select * from t_clazz c join t_student s on c.cid = s.cid where c.cid = #{cid}
</select>
```

注意是ofType，表示“集合中的类型”。

```java
package com.powernode.mybatis.test;

import com.powernode.mybatis.mapper.ClazzMapper;
import com.powernode.mybatis.pojo.Clazz;
import com.powernode.mybatis.utils.SqlSessionUtil;
import org.junit.Test;

public class ClazzMapperTest {
    @Test
    public void testSelectClazzAndStusByCid() {
        ClazzMapper mapper = SqlSessionUtil.openSession().getMapper(ClazzMapper.class);
        Clazz clazz = mapper.selectClazzAndStusByCid(1001);
        System.out.println(clazz);
    }
}
```

执行结果：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292141475.png" style="zoom:45%"/>

### 第二种方式：分步查询

修改以下三个位置即可：

```xml
<resultMap id="clazzResultMap" type="Clazz">
  <id property="cid" column="cid"/>
  <result property="cname" column="cname"/>
  <!--主要看这里-->
  <collection property="stus"
              select="com.powernode.mybatis.mapper.StudentMapper.selectByCid"
              column="cid"/>
</resultMap>

<!--sql语句也变化了-->
<select id="selectClazzAndStusByCid" resultMap="clazzResultMap">
  select * from t_clazz c where c.cid = #{cid}
</select>
```

```java
/**
* 根据班级编号获取所有的学生。
* @param cid
* @return
*/
List<Student> selectByCid(Integer cid);
```

```xml
<select id="selectByCid" resultType="Student">
  select * from t_student where cid = #{cid}
</select>
```


执行结果：

<img src="https://cdn.jsdelivr.net/gh/Travis1024/PicGo_image/202209292141867.png" style="zoom:50%"/>

## 13.4 一对多延迟加载

一对多延迟加载机制和多对一是一样的。同样是通过两种方式：

- 第一种：fetchType="lazy"
- 第二种：修改全局的配置setting，**lazyLoadingEnabled=true，**如果开启全局延迟加载，想让某个sql不使用延迟加载：fetchType="eager"
