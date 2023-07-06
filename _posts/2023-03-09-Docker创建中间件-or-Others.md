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



# Mysql8

## 1、下载镜像

```
docker pull mysql:8.0.32
```



## 2、创建本地挂载目录（4 个）

```
mkdir -p /data/mysql/conf
mkdir -p /data/mysql/data
mkdir -p /data/mysql/log
mkdir -p /data/mysql/mysql-files
```



## 3、创建配置文件

在 conf 文件夹下 vim my.cnf

```properties
[client]
port = 3306
default-character-set = utf8mb4
 
[mysql]
port = 3306
default-character-set = utf8mb4
 
[mysqld]
# bind-address = 0.0.0.0
# port = 3306

# 设置最大连接数
max_connections=10000

character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci
 
# 设置时区和字符集
# default-time-zone='+8:00'
character-set-client-handshake=FALSE
init_connect='SET NAMES utf8mb4 COLLATE utf8mb4_unicode_ci'
 
gtid-mode=ON
enforce-gtid-consistency = ON
```



## 4、启动容器

```shell
docker run --restart=always --name mysql8 -v /data/mydata/mysql/conf:/etc/mysql/conf.d -v /data/mydata/mysql/data:/var/lib/mysql -v /data/mydata/mysql/log:/var/log -v /data/mydata/mysql/mysql-files:/var/lib/mysql-files -p 3307:3306 -e MYSQL_ROOT_PASSWORD='123456' -d mysql:8.0.32
```



## 5、增加远程访问权限

```
mysql -u root -p
 
mysql> use mysql;
mysql> update user set host = '%' where user ='root';
mysql> flush privileges;
mysql> exit
```



## 6、修改两个 root 用户的密码

```
ALTER USER 'root'@'localhost' IDENTIFIED BY '新密码';
ALTER USER 'root'@'%' IDENTIFIED BY '123456';
```



## 7、查看 root 用户的相关信息

```
select host, user, authentication_string, plugin from user; 
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

版本号：mongo：4.4 / 6.0

## · 安装

1. 拉取镜像

   ```shell
    docker pull mongo:6.0
   ```

2. 创建mongo数据持久化目录

   ```shell
   mkdir -p /data/mydata/mongo/data
   ```

3. 运行容器

   ```shell
   docker run -itd --name mongo -v /data/mydata/mongo/data:/data/db -p 27017:27017 mongo:4.4 --auth
   ```

## · 创建用户

1. 进入容器

   ```
   docker exec -it mongo mongosh admin
   ```

2. 创建新用户

   ```shell
   db.createUser({ user:'root',pwd:'123456',roles:[ { role:'userAdminAnyDatabase', db: 'admin'},'readWriteAnyDatabase']});
   ```

3. 修改密码

   ```shell
   db.auth("root","123456")
   db.changeUserPassword('root','w1270278575')
   ```

## · springboot整合mongodb

[Docker安装mongoDB及使用](https://blog.csdn.net/packge/article/details/126539320)



# Sentinel

- 查询 Sentinel 版本

  ```shell
  docker search sentinel
  ```

- 拉取 Sentinel 镜像

  ```shell
  docker pull bladex/sentinel-dashboard
  ```

- 运行 Sentinel 容器

  ```shell
  docker run -itd --name sentinel -p 8858:8858 bladex/sentinel-dashboard
  ```

  

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

[elasticsearch + ik分词器 + kibana](https://juejin.cn/post/7141271047562592264)

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

# 保证权限
chmod -R 777 /mydata/elasticsearch/
```

```
docker stop + docker rm
```

```
docker run -itd --name elasticsearch \
-p 9200:9200 -p 9300:9300 \
-e "discovery.type=single-node" -e ES_JAVA_OPTS="-Xms512m -Xmx512m" \
-v /data/mydata/elasticsearch/config:/usr/share/elasticsearch/config \
-v /data/mydata/elasticsearch/data:/usr/share/elasticsearch/data \
-v /data/mydata/elasticsearch/plugins:/usr/share/elasticsearch/plugins \
-v /data/mydata/elasticsearch/logs:/usr/share/elasticsearch/logs \
--privileged=true elasticsearch:7.16.2
```



# kibana

- 拉取 kibana 镜像

  ```shell
  # 注意和 ElasticSearch 的版本
  docker pull kibana:7.16.2
  ```

- 创建并启动容器

  ```shell
  docker run -itd --name kibana -e ELASTICSEARCH_HOSTS=http://IP:9200 -p 5601:5601 kibana:7.16.2
  ```

- 进入容器修改语言配置

  ```shell
  docker exec -it kibana /bin/bash
  
  # 进入 config 文件夹，找到 kibana.yml 配置文件，在kibana.yml文件添加中文语言设置
  i18n.locale: "zh-CN"
  ```

- 重新启动 kibana 容器





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
  docker run -d -p 5000:80 --name nginx --restart=always -v /mydata/nginx/conf:/etc/nginx -v /mydata/nginx/html:/usr/share/nginx/html -v /mydata/nginx/log:/var/log nginx:latest
  ```

  



```
configure arguments: 
--sbin-path=/usr/local/sbin/nginx 
--conf-path=/etc/nginx/nginx.conf 
--error-log-path=/var/log/nginx/error.log 
--pid-path=/var/run/nginx/nginx.pid 
--lock-path=/var/lock/nginx/nginx.lock 
--http-log-path=/var/log/nginx/access.log 
--http-client-body-temp-path=/tmp/nginx-client-body 
--with-http_ssl_module 
--with-threads 
--with-ipv6 
--add-module=/tmp/build/nginx-rtmp-module/nginx-rtmp-module-1.2.1


configure arguments: 
--prefix=/etc/nginx 
--sbin-path=/usr/sbin/nginx 
--modules-path=/usr/lib/nginx/modules 
--conf-path=/etc/nginx/nginx.conf 
--error-log-path=/var/log/nginx/error.log 
--http-log-path=/var/log/nginx/access.log 
--pid-path=/var/run/nginx.pid 
--lock-path=/var/run/nginx.lock 
--http-client-body-temp-path=/var/cache/nginx/client_temp 
--http-proxy-temp-path=/var/cache/nginx/proxy_temp 
--http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp 
--http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp 
--http-scgi-temp-path=/var/cache/nginx/scgi_temp 
--user=nginx 
--group=nginx 
--with-compat 
--with-file-aio 
--with-threads 
--with-http_addition_module 
--with-http_auth_request_module 
--with-http_dav_module 
--with-http_flv_module 
--with-http_gunzip_module 
--with-http_gzip_static_module 
--with-http_mp4_module 
--with-http_random_index_module 
--with-http_realip_module 
--with-http_secure_link_module 
--with-http_slice_module 
--with-http_ssl_module 
--with-http_stub_status_module 
--with-http_sub_module 
--with-http_v2_module 
--with-mail 
--with-mail_ssl_module 
--with-stream 
--with-stream_realip_module 
--with-stream_ssl_module 
--with-stream_ssl_preread_module 
--with-cc-opt='-g -O2 -ffile-prefix-map=/data/builder/debuild/nginx-1.21.5/debian/debuild-base/nginx-1.21.5=. -fstack-protector-strong -Wformat -Werror=format-security -Wp,-D_FORTIFY_SOURCE=2 -fPIC' 
--with-ld-opt='-Wl,-z,relro -Wl,-z,now -Wl,--as-needed -pie'
```





# RocketMq

1. 在宿主机创建需要挂载的目录

   ```shell
   mkdir -p /data/mydata/rocketmq/nameserver/logs
   mkdir -p /data/mydata/rocketmq/nameserver/store
   
   mkdir -p /data/mydata/rocketmq/broker/logs
   mkdir -p /data/mydata/rocketmq/broker/store
   mkdir -p /data/mydata/rocketmq/broker/conf
   ```

2. 在 conf 路径下创建需要挂载的 broker.conf 配置文件

   ```properties
   # mq集群名称，注意这里进行了更改
   brokerClusterName = FilesBottleCluster
   #broker名称，master和slave使用相同的名称，表明他们的主从关系
   brokerName = broker-master
   #0表示Master，大于0表示不同的slave
   brokerId = 0
   #表示几点做消息删除动作，默认是凌晨4点
   deleteWhen = 00
   #在磁盘上保留消息的时长，单位是小时
   fileReservedTime = 72
   #有三个值：SYNC_MASTER，ASYNC_MASTER，SLAVE；同步和异步表示Master和Slave之间同步数据的机制；
   brokerRole = ASYNC_MASTER
   #刷盘策略，取值为：ASYNC_FLUSH，SYNC_FLUSH表示同步刷盘和异步刷盘；SYNC_FLUSH消息写入磁盘后才返回成功状态，ASYNC_FLUSH不需要；
   flushDiskType = ASYNC_FLUSH
   #设置broker节点所在服务器的ip地址(公网IP)，win系统下，用ipconfig查一下你的主机ip
   brokerIP1 = xxx.xxx.xxx.xxx
   # 是否允许 Broker 自动创建 Topic，建议线下开启，线上关闭 ！！！这里仔细看是 false，false，false
   autoCreateTopicEnable=true
   # 是否允许 Broker 自动创建订阅组，建议线下开启，线上关闭
   autoCreateSubscriptionGroup=true
   # Broker 对外服务的监听端口
   listenPort=10911
   # 补充
   # 磁盘使用达到95%之后,生产者再写入消息会报错 CODE: 14 DESC: service not available now, maybe disk full diskMaxUsedSpaceRatio=95
   ```

3. 拉取镜像 

   ```shell
   # nameserver + broker
   docker pull apache/rocketmq:4.9.4
   # 可视化
   docker pull apacherocketmq/rocketmq-dashboard:1.0.0
   ```

4. 启动 docker 容器

   ```shell
   # nameserver
   docker run -d --restart=always --name rmq_nameserver -p 9876:9876 -v /data/mydata/rocketmq/nameserver/logs:/home/rocketmq/logs -v /data/mydata/rocketmq/nameserver/store:/home/rocketmq/store -e "MAX_POSSIBLE_HEAP=100000000" apache/rocketmq:4.9.4 sh mqnamesrv
   ```

   ```shell
   # broker
   docker run -d --restart=always --name rmq_broker --link rmq_nameserver:nameserver -p 10911:10911 -p 10909:10909 -v /data/mydata/rocketmq/broker/logs:/home/rocketmq/logs -v /data/mydata/rocketmq/broker/store:/home/rocketmq/store -v /data/mydata/rocketmq/broker/conf/broker.conf:/home/rocketmq/rocketmq-4.9.4/conf/broker.conf -e "NAMESRV_ADDR=nameserver:9876" -e "MAX_POSSIBLE_HEAP=200000000" apache/rocketmq:4.9.4 sh mqbroker -c ../conf/broker.conf
   ```

   ```shell
   # dashboard
   docker run -d --name rocketmq-dashboard -e "JAVA_OPTS=-Drocketmq.namesrv.addr=服务器公网 ip:9876" -p 9999:8080 -t apacherocketmq/rocketmq-dashboard:1.0.0
   ```

5. ⚠️ 注意需要开放服务器的 9876、10911、10909、9999（自定）端口

6. [参考：rocketmq 4.9.4 docker部署](https://cloud.tencent.com/developer/article/2157853?from=15425&areaSource=102001.1&traceId=qkkLPmu9xMHLB14XQUM2y)

​	

# Redis

- 在宿主机创建需要挂载的目录

  ```shell
  mkdir -p /data/mydata/redis/conf
  mkdir -p /data/mydata/redis/data
  ```

- conf下创建配置文件redis.conf

  ```ini
  requirepass 123456 # 设置密码
  appendonly yes # 持久化
  ```

- 启动 redis

  ```shell
  docker run --name redis -p 3796:6379 -v /data/mydata/redis/conf/redis.conf:/etc/redis/redis.conf -v /data/mydata/redis/data:/data -d redis redis-server /etc/redis/redis.conf --appendonly yes
  ```




# Minio

- [Docker 搭建 Minio 容器](https://blog.csdn.net/BThinker/article/details/125412751)

- 创建 minio 容器并运行

  ```shell
  docker run -p 9000:9000 -p 9090:9090 \
       --net=host \
       --name minio \
       -d --restart=always \
       -e "MINIO_ACCESS_KEY=admin" \
       -e "MINIO_SECRET_KEY=admin" \
       -v /home/minio/data:/data \
       -v /home/minio/config:/root/.minio \
       minio/minio server \
       /data --console-address ":9090" -address ":9000"
  ```

  

# xxl-job-admin

- github中下载 xxl-job-admin/docs/db下的 sql 文件， 运行 sql 文件

- 下载镜像文件（⚠️一定要指定版本号，因为官网没有设置默认的 lastest 版本）

  ```shell
  docker pull xuxueli/xxl-job-admin:2.4.0
  ```

- 创建文件夹

  ```shell
  mkdir -p /data/mydata/xxl-job-admin/applogs
  ```

- 创建容器（主要是指定 mysql 地址及账号密码）

  ```shell
  docker run -itd -e PARAMS="--spring.datasource.url=jdbc:mysql://x.x.x.x:x/xxl_job?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&serverTimezone=Asia/Shanghai --spring.datasource.username=root --spring.datasource.password=xxxx" -p 9080:8080 -v /mydata/xxl-job-admin/applogs:/data/applogs --name xxl-job-admin xuxueli/xxl-job-admin:2.4.0
  ```

- 访问网页

  ```
  http://localhost:9080/xxl-job-admin
  admin
  123456
  ```

  
