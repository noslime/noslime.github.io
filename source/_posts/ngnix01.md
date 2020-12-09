---
title: nginx简介及安装
date: 2020-11-23 14:42:22
author: noslime
top: true
cover: true
categories: 后端
tags: nginx
keywords: nginx
---

nginx这款优秀的反向代理服务器，在网站的运行中占的比例越来越高，不管工作中是否用到，都值得去学习一下，一个好的开头必不可少，本文记录一下nginx的安装。

### 一、nginx概述

*Nginx* (engine x) 是一个高性能的[HTTP](https://baike.baidu.com/item/HTTP)和[反向代理](https://baike.baidu.com/item/反向代理/7793488)web服务器，同时也提供了IMAP/POP3/SMTP服务，nginx能够达到50000次并发，性能优异，且占用内存较小。国内大型的站点，例如百度、京东、新浪、网易、腾讯、淘宝等，都使用了[nginx] .

[nginx]: http://nginx.org/

### 二、常用功能

1. 反向代理

   -  正向代理：局域网中的客户端(如浏览器)访问Internet，通过配置代理访问，这种代理用户访问网站代理就是正向代理。
   -  反向代理：向代理服务器发送请求，反向代理转发到实际服务器，客户端不需要做任何配置。目标服务器对客户端是透明的。

2. 负载均衡

   nginx服务器收到请求后，分发到不同的服务器上去处理，提高系统负载的过程，也就是我们所说的负载均衡。

3. Web缓存

   Nginx可以对不同的文件做不同的缓存处理，配置灵活，并且支持FastCGI_Cache，主要用于对FastCGI的动态程序进行缓存。配合着第三方的ngx_cache_purge，对制定的URL缓存内容可以的进行增删管理


### 三、nginx安装

#### YUM安装

1. 安装先决条件：

> ```
> sudo yum install yum-utils
> ```

2. 若要设置 yum 存储库，请创建具有以下内容命名的文件：`/etc/yum.repos.d/nginx.repo`

> ```shell
> [nginx-stable]
> name=nginx stable repo
> baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
> gpgcheck=1
> enabled=1
> gpgkey=https://nginx.org/keys/nginx_signing.key
> module_hotfixes=true
> 
> [nginx-mainline]
> name=nginx mainline repo
> baseurl=http://nginx.org/packages/mainline/centos/$releasever/$basearch/
> gpgcheck=1
> enabled=0
> gpgkey=https://nginx.org/keys/nginx_signing.key
> module_hotfixes=true
> ```

3. 默认情况下，使用稳定 nginx 包的存储库。如果要使用主线 nginx 包，请运行以下命令：

> ```shell
> sudo yum-config-manager --enable nginx-mainline
> ```

4. 要安装 nginx，请运行以下命令：

> ```shell
> sudo yum install nginx
> ```

5. 当提示接受 GPG 密钥时，请验证指纹是否匹配，如果匹配，请接受它。

   `573B FD6B 3D8F BC64 1079 A6AB ABF5 BD82 7BD9 BF62`

6. 命令手册 例：/usr/bin/nginx -v 查看版本情况

> ```shell
> -?,-h         : this help
> -v            : show version and exit
> -V            : show version and configure options then exit
> -t            : test configuration and exit
> -T            : test configuration, dump it and exit
> -q            : suppress non-error messages during configuration testing
> -s signal     : send signal to a master process: stop, quit, reopen, reload
> -p prefix     : set prefix path (default: NONE)
> -c filename   : set configuration file (default: conf/nginx.conf)
> -g directives : set global directives out of configuration file
> ```

7. YUM安装默认位置：

> ```shell
> /usr/sbin/nginx   //执行程序位置
> /etc/nginx/  // nginx.conf所在
> /usr/share/nginx  //静态文件
> /var/log/nginx/  //访问记录日志 错误日志
> /var/run/nginx.pid   //进程ID文件
> ```

#### 从源包手动编译安装

1. 安装所需依赖

   - gcc是编程语言编译器，支持c、c++等，而nginx是由C语言编写而成，所以手动安装的编译过程要依赖gcc编译器。

   - pcre库是一组函数，它们使用与 Perl 5 相同的语法和语义实现正则表达式模式匹配，nginx的请求分发需要用到正则匹配。 其中devel代表供开发使用，包括头文件链接库等， 如果安装基于pcre开发的程序，只需要安装pcre包就行了，但编译使用了PCRE库的源代码，则需要pcre-devel。

   - zlib 是通用的压缩库，nginx的压缩模块需要用到，主要对网络传输的资源进行压缩以提高传输效率。

   - nginx涉及到很多加密、解密的功能。如https中需要用到加密解密，所以编译的时候需要使用openssl

> ```shell
> yum install -y gcc pcre-devel zlib-devel openssl-devel  //解决依赖问题
> ```

2. 下载并解压nginx程序包

> ```shell
> wget -P /usr/src https://nginx.org/download/nginx-1.18.0.tar.gz
> tar zxvf nginx-1.18.0.tar.gz
> ```

3. 进入解压文件夹构建程序

   configure命令作用是分析程序建立环境，检查是否安装了必要的外部工具和组件，

   并创建配置文件makefile，此文件描述了包括最终完成的程序各组件之间的关系和依赖性
   configure命令有多个可选项，如下所用的--prefix是告诉程序的安装位置
   如不指定 则./configure --help里面有个path，就是默认安装路径
   关于nginx的configure更多选项设置 详见 [Configure]

   [Configure]: http://nginx.org/en/docs/configure.html

> ```shell
> cd nginx-1.18.0
> ./configure  --prefix=/usr/local/nginx    
> ```

4. 编译与安装

> ```shell
> $ make && make install 
> ```

5. 常用命令

> ```shell
> /usr/local/nginx/sbin/nginx    //启动
> /usr/local/nginx/sbin/nginx -c 具体nginx.conf配置文件路径   //根据指定配置文件启动
> /usr/local/nginx/sbin/nginx -v //查看版本
> /usr/local/nginx/sbin/nginx -s stop //停止
> /usr/local/nginx/sbin/nginx -s reload  //热启动，重新加载配置文件
> ```

   6. 使用systemctl管理nginx

      创建一个nginx.service文件

> ```shell
> touch /usr/lib/systemd/system/nginx.service
> ```

​	编辑文件内容为：

> ```shell
> [Unit]
> Description=nginx - high performance web server
> Documentation=http://nginx.org/en/docs/
> After=network-online.target remote-fs.target nss-lookup.target
> Wants=network-online.target
> 
> [Service]
> Type=forking
> PIDFile=/var/run/nginx.pid
> ExecStart=/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
> ExecReload=/bin/sh -c "/bin/kill -s HUP $(/bin/cat /var/run/nginx.pid)"
> ExecStop=/bin/sh -c "/bin/kill -s TERM $(/bin/cat /var/run/nginx.pid)
> PrivateTmp=true    //设置是否使用私有的tmp目录
> 
> [Install]
> WantedBy=multi-user.target
> ```

​	常用命令

> ```shell
> systemctl start nginx
> systemctl stop nginx
> systemctl reload nginx
> systemctl status nginx
> 
> systemctl enable nginx  //设置开机自启
> systemctl disable nginx  //关闭开机自启
> ```

### 四、开放必要的端口

1. 为nginx开放80端口

    由于linux默认开启了防火墙，nginx是访问不成功的，如果是个人学习，可以直接关闭防火墙

> ```shell
> systemctl stop firewalld
> ```

​		否则只开启指定的端口供外部访问，以下为一些端口操作命令

> ```shell
> firewall-cmd --zone=public --add-port=80/tcp --permanent   //永久开放80端口
> firewall-cmd --reload   //重载使之生效
> firewall-cmd --zone= public --query-port=80/tcp   //查看是否开放
> firewall-cmd --zone= public --remove-port=80/tcp --permanent   //删除80端口
> 
> firewall-cmd --zone=public --add-port=40000-45000/tcp --permanent  开放一段端口
> firewall-cmd --zone=public --list-ports  //查看开放的端口列表
> ```

​		若服务器使用的iptables，则：

> ```shell
> iptables -I INPUT -p tcp --dport 80 -j ACCEPT   //开放80
> service iptables save  //保存策略
> service iptables restart  //重启防火墙
> iptables -D INPUT 2  // 删除规则，通过 iptables -L -n --line-number 可以显示规则和相对应的编号
> ```

### 五、赋予用户权限

大多情况下并不建议使用root用户去操作数据，因此在使用普通用户时可能会遇到一些权限问题，以下步骤帮助用户获取nginx相关的操作权限。

1. 创建用户组

   ```shell
   sudo groupadd nginx
   ```

2. 将当前用户假如nginx用户组

   ```shell
   sudo usermod -g nginx username
   ```

3. 将nginx的权限给nginx用户组的用户

   ```shell
   sudo chown -R username:nginx /usr/local/nginx
   ```

4. 如有必要，给nginx用户组写权限

   ```shell
   sudo chmod g+w -R /usr/local/nginx
   ```

5. 如有必要，授予nginx目录segid权限，做用户组控制

   ```shell
   sudo chmod g+s -R /usr/local/nginx
   ```

   