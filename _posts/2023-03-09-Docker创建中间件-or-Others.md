---
title: Docker创建中间件-or-Others
author: Travis <Hongxu Wei>
date: 2023-03-09 22:22:37 +0800
categories: [DailyNotes Space]
tags: [docker]
math: false
---

# Mysql

## 1、下载镜像

```bash
sudo docker pull mysql:5.7

sudo docker images
```

## 2、创建mysql实例并启动

```bash
sudo docker run -p 3306:3306 --name mysql \
-v /mydata/mysql/log:/var/log/mysql \
-v /mydata/mysql/data:/var/lib/mysql \
-v /mydata/mysql/conf:/etc/mysql \
-e MYSQL_ROOT_PASSWORD=root \
-d mysql:5.7
参数说明
-p 3306:3306 将容器的3306端口映射到主机
-v /mydata/mysql/log:/var/log/mysql\ 将日志文件挂载到主机
-v /mydata/mysql/data:/var/lib/mysql\ 将数据文件挂载到主机
-v /mydata/mysql/conf:/etc/mysql\ 将配置文件挂载到主机
```

## 3、进入mysql容器

```bash
sudo docker exec -it mysql /bin/bash
```

## 4、修改mysql账号密码

```bash
1.进入mysql容器
docker exec -it mysql /bin/bash

2.登录mysql
mysql -u root -p
输入密码：root

3.切换数据库
use mysql

4.查询root用户
select * from user where user = root;

5.修改密码
update user set authentication_string = password('新的密码'), password_expired = 'N', password_last_changed = now() where user = 'root';

6.修改加密方式
update user set plugin="mysql_native_password";

7.刷新权限
flush privileges;

8.退出
quit;

9.重新登录
mysql -u root -p 

输入新的密码，登录成功

```

## 5、设置容器在机器重启后自动启动

```bash
docker update {容器id} --restart=always
```



# Zookeeper

## 1、下载镜像

```bash
# 查看本地镜像
docker images
# 检索ZooKeeper 镜像
docker search zookeeper
# 拉取ZooKeeper镜像最新版本
docker pull zookeeper:latest
```

## 2、创建挂载目录

```bash
mkdir -p /mydata/zookeeper/data # 数据挂载目录
mkdir -p /mydata/zookeeper/conf # 配置挂载目录
mkdir -p /mydata/zookeeper/logs # 日志挂载目录
```

## 3、创建并启动镜像实例

```bash
docker run -d --name zookeeper --privileged=true -p 2181:2181  -v /mydata/zookeeper/data:/data -v /mydata/zookeeper/conf:/conf -v /mydata/zookeeper/logs:/datalog zookeeper
```

参数说明：

```bash
-e TZ="Asia/Shanghai" # 指定上海时区 
-d # 表示在一直在后台运行容器
-p 2181:2181 # 对端口进行映射，将本地2181端口映射到容器内部的2181端口
--name # 设置创建的容器名称
-v # 将本地目录(文件)挂载到容器指定目录；
--restart always #始终重新启动zookeeper，看需求设置不设置自启动
```

## 4、修改配置文件

添加ZooKeeper配置文件，在挂载配置文件目录(/mydata/zookeeper/conf)下，新增zoo.cfg 配置文件，配置内容如下：（⚠️注意去掉中文注释）

可能主要需要添加：**clientPort=2181**

```
dataDir=/data  # 保存zookeeper中的数据
clientPort=2181 # 客户端连接端口，通常不做修改
dataLogDir=/datalog
tickTime=2000  # 通信心跳时间
initLimit=5    # LF(leader - follower)初始通信时限
syncLimit=2    # LF 同步通信时限
autopurge.snapRetainCount=3
autopurge.purgeInterval=0
maxClientCnxns=60
standaloneEnabled=true
admin.enableServer=true
server.1=localhost:2888:3888;2181
```

## 5、进入容器，查看状态

```bash
# 进入zookeeper 容器内部
docker exec -it zookeeper /bin/bash
# 检查容器状态
docker exec -it zookeeper /bin/bash ./bin/zkServer.sh status
# 进入控制台
docker exec -it zookeeper zkCli.sh
```

[docker配置zookeeper参考链接](https://blog.csdn.net/duyun0/article/details/128437451)



# Nacos

启动命令：

```shell
docker run --name nacos -p 8848:8848 -p 9848:9848 -p 9849:9849 -e MODE=standalone -e JVM_XMS=128m -e JVM_XMX=128m -v /mydata/nacos/logs:/home/nacos/logs -v /mydata/nacos/conf/application.properties:/home/nacos/conf/application.properties -d nacos/nacos-server:latest
```



# MongoDB

版本号：mongo：4.4

## · 安装

1. 拉取镜像

   ```shell
    docker pull mongo:4.4
   ```

2. 创建mongo数据持久化目录

   ```shell
   mkdir -p /mydata/mongo/data
   ```

3. 运行容器

   ```shell
   docker run -itd --name mongo -v /mydata/mongo/data:/data/db -p 27017:27017 mongo:4.4 --auth
   ```

## · 创建用户

## · 连接测试

## · springboot整合mongodb

[Docker安装mongoDB及使用](https://blog.csdn.net/packge/article/details/126539320)



# Sentinel



# Seata

```shell
docker run -d -p 8091:8091 -p 7091:7091  --name seata-serve seataio/seata-server:1.6.1
```

```shell
docker cp b1b237:/seata-server/resources /mydata/seata/config
```

```shell
docker stop + docker rm
```

```shell
docker run --name seata-serve -p 8091:8091 -p 7091:7091 -e SEATA_IP=宿主机IP -v /mydata/seata/config/resources:/seata-server/resources -v /mydata/seata/sessionStore:/seata-server/sessionStore -d seataio/seata-server:1.6.1
```

⚠️⚠️ 需要设置 -e SEATA_IP=宿主机IP，否则在nacos上注册的地址是局域网的地址

- 修改application.yml配置文件

  ```yaml
  server:
    port: 7091
  
  spring:
    application:
      name: seata-server
  
  logging:
    config: classpath:logback-spring.xml
    file:
      path: ${user.home}/logs/seata
    extend:
      logstash-appender:
        destination: 127.0.0.1:4560
      kafka-appender:
        bootstrap-servers: 127.0.0.1:9092
        topic: logback_to_logstash
  
  console:
    user:
      username: seata
      password: seata
  
  seata:
    config:
      # support: nacos 、 consul 、 apollo 、 zk  、 etcd3
      type: nacos
      nacos:
        server-addr: 宿主机IP:8848
        namespace: 64e29a8b-edd7-4f60-b941-24111fb1081c
        group: DEFAULT_GROUP
        username: nacos
        password: nacos
        context-path:
        ##if use MSE Nacos with auth, mutex with username/password attribute
        #access-key:
        #secret-key:
        data-id: seataServer.properties
    registry:
      # support: nacos 、 eureka 、 redis 、 zk  、 consul 、 etcd3 、 sofa
      type: nacos
      preferred-networks: 30.240.*
      nacos:
        application: seata-server
        server-addr: 宿主机IP:8848
        group: DEFAULT_GROUP
        namespace: 64e29a8b-edd7-4f60-b941-24111fb1081c
        cluster: default
        username: nacos
        password: nacos
        context-path:
        ##if use MSE Nacos with auth, mutex with username/password attribute
        #access-key:
        #secret-key:
    server:
      service-port: 8091 #If not configured, the default is '${server.port} + 1000'
      max-commit-retry-timeout: -1
      max-rollback-retry-timeout: -1
      rollback-retry-timeout-unlock-enable: false
      enable-check-auth: true
      enable-parallel-request-handle: true
      retry-dead-threshold: 130000
      xaer-nota-retry-timeout: 60000
      enableParallelRequestHandle: true
      recovery:
        committing-retry-period: 1000
        async-committing-retry-period: 1000
        rollbacking-retry-period: 1000
        timeout-retry-period: 1000
      undo:
        log-save-days: 7
        log-delete-period: 86400000
      session:
        branch-async-queue-size: 5000 #branch async remove queue size
        enable-branch-async-remove: false #enable to asynchronous remove branchSession
    store:
      # support: file 、 db 、 redis
      mode: file
      session:
        mode: file
      lock:
        mode: file
      file:
        dir: sessionStore
        max-branch-session-size: 16384
        max-global-session-size: 512
        file-write-buffer-cache-size: 16384
        session-reload-read-size: 100
        flush-disk-mode: async
    security:
      secretKey: SeataSecretKey0c382ef121d778043159209298fd40bf3850a017
      tokenValidityInMilliseconds: 1800000
      ignore:
        urls: /,/**/*.css,/**/*.js,/**/*.html,/**/*.map,/**/*.svg,/**/*.png,/**/*.ico,/console-fe/public/**,/api/v1/auth/login
  ```

- maven依赖版本

  ```xml
  <dependency>
      <groupId>com.alibaba.cloud</groupId>
      <artifactId>spring-cloud-starter-alibaba-seata</artifactId>
      <version>2021.0.5.0</version>
  </dependency>
  ```

- nacos注册中心要配置的内容

  <img src="https://travisnotes.oss-cn-shanghai.aliyuncs.com/mdpic/202304140926486.png" alt="image-20230414092611286" style="zoom:50%;" />



[分布式框架seata的使用](https://blog.csdn.net/qq_15717719/article/details/123087819)

[官网部署文档](https://seata.io/zh-cn/docs/ops/deploy-guide-beginner.html)



# ElasticSearch

[docker安装配置elasticSearch](https://blog.csdn.net/huanglu0314/article/details/124535763)

[docker安装配置elasticSearch2](https://blog.csdn.net/qq_40942490/article/details/111594267)

```
docker run --name elasticsearch \
-p 9200:9200 -p 9300:9300 \
-e "discovery.type=single-node" -e ES_JAVA_OPTS="-Xms512m -Xmx512m" \
-d elasticsearch:7.16.2
```

```
docker cp elasticsearch:/usr/share/elasticsearch/config/ /mydata/elasticsearch/
docker cp elasticsearch:/usr/share/elasticsearch/data/ /mydata/elasticsearch/
docker cp elasticsearch:/usr/share/elasticsearch/logs/ /mydata/elasticsearch/
docker cp elasticsearch:/usr/share/elasticsearch/plugins/ /mydata/elasticsearch/
```

```
docker stop + docker rm
```

```
docker run --name elasticsearch \
-p 9200:9200 -p 9300:9300 \
-e "discovery.type=single-node" -e ES_JAVA_OPTS="-Xms512m -Xmx512m" \
-v /mydata/elasticsearch/config:/usr/share/elasticsearch/config \
-v /mydata/elasticsearch/data:/usr/share/elasticsearch/data \
-v /mydata/elasticsearch/plugins:/usr/share/elasticsearch/plugins \
-v /mydata/elasticsearch/logs:/usr/share/elasticsearch/logs \
--privileged=true -d elasticsearch:7.16.2
```



# kibana





# Nginx

- Docker 拉取 nginx 镜像

  ```shell
  docker pull nginx
  ```

- 启动 nginx 容器

  ```shell
  docker run -d --name nginx nginx:latest
  ```

- 把计划挂载的文件夹进行 cp, /mydata/nginx 文件夹下有 conf、log、html 三个目录

  ```shell
  docker cp 2aa9:/etc/nginx /mydata/nginx/conf
  docker cp 2aa9:/usr/share/nginx/html /mydata/nginx
  docker cp 2aa9:/var/log /mydata/nginx
  ```

- 删除 nginx 容器，重新运行新的容器，并进行挂载

  ```shell
  docker rm ***
  ```

  ```
  docker run -d -p 5000:80 --name nginx --restart=always -v /mydata/nginx/conf/nginx:/etc/nginx -v /mydata/nginx/html:/usr/share/nginx/html -v /mydata/nginx/log:/var/log nginx:latest
  ```

  
