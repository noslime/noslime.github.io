---
title: redis安装
date: 2020-12-3 09:14:22
author: noslime
categories: redis
tags: redis
keywords: redis
---

# Redis的安装

### 官方安装命令

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



### 问题

1. 安装过程中可能出现因为gcc版本过低导致编译失败，需要升级gcc为较新的版本

```shell
sudo yum install centos-release-scl
sudo yum install devtoolset-8-gcc*
scl enable devtoolset-8 bash
//然后清理下错误编译
make distclean
```

2. 无法正常关闭redis服务

   错误信息

   > 127.0.0.1:6379> SHUTDOWN
   > (error) ERR Errors trying to SHUTDOWN. Check logs.

   2.1. 由于默认日志是输入到标准输出的， 创建日志文件并修改配置文件

   > mkdir /var/log/redis
   > touch /var/log/redis/redis.log
   >
   > 在redis.conf中的配置日志路径
   > logfile "/var/log/redis/redis.log"

   2.2. 查看日志后，发现错误信息如下

   > User requested shutdown...
   > Saving the final RDB snapshot before exiting.
   > Failed opening the RDB file dump.rdb (in server root dir /usr/local/redis-6.0.9) for saving: Permission denied
   > Error trying to save the DB, can't exit.
   >
   > 可见是由于dump.rdb文件没有写权限

   2.3. 修改权限信息

   ​		

   ```shell
   sudo chmod 777 dump.rdb
   sudo chmod 777 /usr/local/redisdir
   ```

   