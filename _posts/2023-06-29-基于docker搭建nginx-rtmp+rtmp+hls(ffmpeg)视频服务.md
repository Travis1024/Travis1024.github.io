---
title: 基于docker搭建nginx-rtmp+rtmp+hls(ffmpeg)视频服务
author: Travis <Hongxu Wei>
date: 2023-06-29 16:49:20 +0800
categories: [Java Learning Space]
tags: [nginx, rtmp, hls, ffmpeg]
math: false
---



## 启动 nginx-rtmp

- 拉取 nginx-rtmp 镜像

  ```shell
  docker pull tiangolo/nginx-rtmp
  ```

- 启动 nginx-rtmp 容器

  ```shell
  docker run -d --name nginx-rtmp tiangolo/nginx-rtmp:latest
  ```

- 把计划挂载的文件夹进行 cp, /mydata/nginx 文件夹下有 conf、log目录

  ```shell
  docker cp 2aa9:/etc/nginx /data/mydata/nginx/conf
  docker cp 2aa9:/var/log /data/mydata/nginx
  ```

- 删除 nginx 容器，重新运行新的容器，并进行挂载

  ```shell
  docker rm ***
  ```

  ```shell
  docker run -itd --name nginx-rtmp -p 5000:5000 \
  -v /home/travis/hlsvideo:/home/travis/hlsvideo  \
  -v /data/mydata/nginx-rtmp/conf/nginx:/etc/nginx \
  -v /data/mydata/nginx-rtmp/log:/var/log \
  tiangolo/nginx-rtmp:latest
  ```

  **⚠️ /home/travis/hlsvideo  下存放的视频切片文件（m3u8 + ts）, 必须进行挂载**




## 配置 nginx-rtmp

```ini
http {
  server {
    listen 5000;

    location /hlsvideo {
      types {
        application /vnd.apple.mpegurl m3u8;
        video /mp2t ts;
      }
      #访问权限开启，否则访问这个地址会报403
      autoindex on;
      #视频流存放地址，与上面的hls_path相对应，这里root和alias的区别可自行百度
      root /home/travis;
      expires -1;
      add_header Cache-Control no-cache;
      #防止跨域问题
      add_header 'Access-Control-Allow-Origin' '*';
      add_header 'Access-Control-Allow-Credentials' 'true';
      add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
      add_header 'Access-Control-Allow-Headers' 'DNT,X-CustomHeader,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type';
    }
  }
}
```



## 注意：docker 部署 nginx 问题

- docker安装的nginx相当于是在我们宿主机上又安装了一个linux系统，所以不能直接访问我们宿主机的文件夹，前面启动nginx容器时挂载了宿主机配置文件和日志目录，在nginx配置文件中配置的内容都是相对容器而言的，所以直接写宿主机的目录会出现404，这种情况需要我们启动时将我们的资源目录一并挂载，对应着容器的某个目录，也就是我们配置文件中配置的目录，挂载之后，其实就是容器可以直接访问宿主机对应的目录以及目录下的所有文件了，挂载的命令-v 宿主机资源目录:容器目录，修改配置文件之后需要重启nginx容器，docker restart 容器ID/名称。



## 参考

[docker搭建直播服务](http://123.57.164.21/?p=1384)