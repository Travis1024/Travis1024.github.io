---
title: obsidian git：ssh and https
author: Travis <Hongxu Wei>
date: 2023-03-01 22:25:58 +0800
categories: [DailyNotes Space]
tags: [obsidian]
math: false
---

在使用obsidian-git插件时，能够commit，但是push时一直报错，如下：

<img src="https://gitee.com/awtaling/images_repository/raw/master/202303082159061.jpg" alt="4421678277162_.pic" style="zoom:50%;" />

提示信息是未能读取到用户名（注意此路径为https协议），按照{username}:{password}配置后，出现其他报错信息：

```
remote: Support for password authentication was removed on August 13, 2021.
```

**---破案---**

原因为：从 21 年 8 月 13 后不再支持用户名密码的方式验证，而需要创建个人访问令牌(personal access token)当创建完个人访问令牌后

准备一键push到远端仓库的时候，一模一样的报错又发生了，检查了ssh密钥的位置用法都没有问题，原因？？？

**---再次破案---**

由于之前并没有单独使用ssh keys的习惯，在git clone的时候很多时候都是按照git init repo上的指示，采用了https协议，而非git协议。通过以下命令查看这个项目的remote到底是什么，使用以下命令：

```
git config --get remote.origin.url
```

返回的结果为：

```
https://github.com/Travis1024/TravisNotes.git
```

这证实了我们在clone的时候采用的是https协议。https协议会每次要求你输入账户密码，而git协议才可以使用ssh-keys文件，实现git push自由。所以需要更改remote协议：

```
git remote set-url origin git@github.com:Travis1024/TravisNotes.git
```

更改完成之后再次查看remote

```
git@github.com:Travis1024/TravisNotes.git
```

再次尝试git push，提示需要输入Enter passphrase for key，在命令行进行输入没有任何问题，但是obsidian-git插件并不会进行输入的交互，所以会报错：

```
fatal: Could not read from remote repository.
```

**---破案破的人麻了---**

解决办法也很简单，就是通过命令将《无知》的我配置ssh密码设置为空，就解决了：

```
$ ssh-keygen -p [-P old_passphrase] [-N new_passphrase] [-f keyfile]
eg：
$ ssh-keygen -p -P 123456 -N '' -f ~/.ssh/id_rsa
```

