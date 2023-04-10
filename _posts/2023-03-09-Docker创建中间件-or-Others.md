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

