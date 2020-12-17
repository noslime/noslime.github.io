---
title: REDIS安装
date: 2020-12-3 09:14:22
author: noslime
top: false
cover: false
categories: 数据库
tags: REDIS
keywords:  redis安装
summary: 简要介绍redis数据库在linux下的安装
---

# Redis的安装

### 安装命令

```shell
$ wget https://download.redis.io/releases/redis-6.0.9.tar.gz
$ tar xzf redis-6.0.9.tar.gz
$ cd redis-6.0.9
$ make
$ make install 
```

运行服务端

```shell
reids-server  /usr/local/redis/redis.conf
```

测试客户端

```shell
$ redis-cli
redis> ping
PONG
```

### 可能遇到的问题

#### 未安装GCC

```shell
# cc: command not found
sudo yum install gcc
```

#### 默认内存分配器错误

```shell
# fatal error: jemalloc/jemalloc.h: No such file or directory
sudo make MALLOC=libc
```

#### gcc版本过低

```shell
# error: expected specifier-qualifier-list before ‘_Atomic’
sudo yum install centos-release-scl
sudo yum install devtoolset-8-gcc*
scl enable devtoolset-8 bash
# 然后清理下错误编译
make distclean
```

#### 无法正常关闭redis服务

- 客户端报错

> 127.0.0.1:6379> SHUTDOWN
> (error) ERR Errors trying to SHUTDOWN. Check logs.

- 日志信息


> User requested shutdown...
> Saving the final RDB snapshot before exiting.
> Failed opening the RDB file dump.rdb (in server root dir /usr/local/redis-6.0.9) for saving: Permission denied
> Error trying to save the DB, can't exit.

- 修改权限信息

```shell
sudo chmod 777 dump.rdb
sudo chmod 777 /usr/local/redisdir
```

