---
title: Ubuntu修改Docker镜像容器存储位置
author: Travis <Hongxu Wei>
date: 2023-06-28 17:47:11 +0800
categories: [Java Learning Space]
tags: [docker]
math: false
---



# Ubuntu修改Docker镜像容器存储位置



- Docker 默认的位置在/var/lib/docker,当前所有的镜像、容器都存储在这儿。如果你有任何在运行的容器，停止这些容器，并确保没有容器在运行，然后运行以下命令，确定当前Docker使用的存储驱动

  ```shell
  sudo docker info
  ```

- 在输出的信息中，查找Storage Driver那行，并记下：

  ```shell
  Storage Driver: overlay2
  docker存储根目录：
  
  Docker Root Dir: /var/lib/docker
  ```

- 关闭docker服务：

  ```shell
  sudo systemctl stop docker.service
  ```

- 将原来的 docker 文件夹拷贝到新的地方（比如新数据盘）

  ```shell
  sudo cp -r /var/lib/docker /data/docker
  ```

  这里不需要新建一个/data/docker目录，再进行复制，如果原来就存在、data/docker的话，复制的结果会变成`/data/docker/docker`

- 修改docker中默认镜像和容器的保存位置

  ```shell
  sudo vim /etc/docker/daemon.json
  ```

- 将里面的data-root改为新的docker容器存储位置

  ![image-20230628175223876](https://travisnotes.oss-cn-shanghai.aliyuncs.com/mdpic/202306281752951.png)

- 开启 docker 更新配置

  ```shell
  sudo systemctl daemon-reload
  sudo systemctl start docker
  ```

  