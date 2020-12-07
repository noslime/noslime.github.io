---
title: linux创建用户并赋予root权限
date: 2020-11-28 14:09:22
author: noslime
categories: linux
tags: 
	- linux
	- case
keywords: linux
---

###  一、创建用户

创建一个带有家目录，并且可以登录bash的用户

```shell
su		//切换root用户创建用户
useradd -m -s /bin/bash noslime	
```

给创建的用户分配密码，并切换到noslime用户

```shell
passwd noslime
su - noslime
```

### 二、赋予root权限

方案一

修改/etc/sudoers文件，使得用户noslime免密使用sudo命令

```shell
echo "nolime ALL=(ALL:ALL)  NOPASSWD:ALL" >> /etc/sudoers
```

方案二

将用户添加到管理员用户组wheel中去，有的linux发行版，需要将`/etc/sudoers`文件中以下一行的注释打开：

```shell
%wheel  ALL=(ALL)       ALL  //centos 7 8 ubuntu20版本默认打开,看一下不费事
```

将用户添加到wheel管理员用户组中

```shell
usermod -g wheel noslime   //给用户指定主用户组
usermod -G wheel noslime   //给用户指定用户组，添加操作，但会去除原有附加组

gpasswd -a user_name group_name   //添加用户到组 建议操作
gpasswd -d user_name group_name   //将用户从指定组中删除
```

修改成功后通过`groups username` 或 `id username`查看用户的组是否添加成功，添加成功后如无作用，可**重启系统**使之生效，即可使用sudo命令获得管理员权限。

