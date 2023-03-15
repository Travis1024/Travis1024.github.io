---
title: docker安装mysql
author: Travis <Hongxu Wei>
date: 2023-03-09 22:22:37 +0800
categories: [DailyNotes Space]
tags: [docker]
math: false
---

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

