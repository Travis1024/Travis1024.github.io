---
title: Linux Commands
author: Travis <Hongxu Wei>
date: 2023-03-09 22:17:45 +0800
categories: [Linux Space]
tags: [linux]
math: false
---

## 1、查看Linux系统架构的命令

```bash
uname -a

Linux travis1024 5.4.0-139-generic #156~18.04.1-Ubuntu SMP Wed Jan 25 15:56:22 UTC 2023 x86_64 x86_64 x86_64 GNU/Linux
```


## 2、Mac查看端口调用

```bash
lsof -i tcp:8081
```



## 3、jar包增量修改

```
jar -xvf  [jar包名称]
```

```
jar -uvf0 [jar包名称] [文件夹绝对路径]
```

```
-u 更新现有档案
-v 在标准输出中生成详细输出
-f 指定档案文件名
-0 仅存储; 不使用任何 ZIP 压缩
```



## 4、动态观察命令的变化

```shell
watch -n 1 'virsh list --all'
```















jar -uvf0 VmControl-1.0-SNAPSHOT.jar BOOT-INF

jar -xvf VmControl-1.0-SNAPSHOT.jar

nmcli connection show --active

nmcli connection show uuid 33904e60-0188-4c92-b17d-bca2331c22c7 | grep connection.id