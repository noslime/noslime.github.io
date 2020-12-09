---
title: nginx cluster
date: 2020-12-08 14:01:35
author: noslime
categories: nginx
tags: 
    - nginx
    - cluster
keywords: nginx
---

nginx虽然实现了应用服务器的负载均衡，但其本身也有down掉的风险，所以为了避免nginx作为分发器时发生单点故障，搭建nginx的集群是有必要的。目前，搭建nginx集群主流方案，是由高可用监控软件Keepalived实现的，本文记录一次搭建过程。

# Nginx Cluster



## 一、主备模式

### 简介

使用两台服务器，一台主服务器和一台热备服务器，当主服务器发生故障时，热备服务器接管主服务器的公网虚拟IP，提供负载均衡服务。主备模式对外提供一个VIP。

### 实现过程

#### 安装keepalived软件

##### 安装指令

```shell
yum install keepalived -y
```

##### 操作指令

```shell
systemctl start keepalived
systemctl stop keepalived
systemctl restart keepalived
systemctl status keepalived
```

#### 修改keepalived配置文件

##### master机器

```text
! Configuration File for keepalived

global_defs {
   notification_email {
     nickname@qq.com
   }
   notification_email_from nickname@qq.com
   smtp_server 127.0.0.1
   smtp_connect_timeout 30
   router_id vrrp020
}

vrrp_script check_nginx {
    script "/etc/keepalived/nginx_check.sh"
    interval 3
    weight -20
    fall 2
    rise 1
}

vrrp_instance nginx {
    state MASTER
    interface ens33
    mcast_src_ip 192.168.0.20
    virtual_router_id 66
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.0.251
    }
	track_script {
        check_nginx
    }
}
```

##### backup机器

```text
! Configuration File for keepalived

global_defs {
   notification_email {
     nickname@qq.com
   }
   notification_email_from nickname@qq.com
   smtp_server 127.0.0.1
   smtp_connect_timeout 30
   router_id vrrp022
}

vrrp_script check_nginx {
    script "/etc/keepalived/nginx_check.sh"
    interval 3
    weight -20
    fall 2
    rise 1
}

vrrp_instance nginx {
    state BACKUP
    interface ens33
    mcast_src_ip 192.168.0.22
    virtual_router_id 66
    priority 80
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.0.251
    }
	track_script {
        check_nginx
    }
}
```

#### 编写检查脚本

此脚本的作用是配合keepalived监控nginx的状态，提供唤醒功能，并在nginx无法唤醒的情况下，结束故障服务器的keepalived进程，以实现VIP漂移。

```bash
#!/bin/bash
counter=$(ps -C nginx --no-heading|wc -l)
echo "$counter"
if [ "${counter}" = "0" ]; then
    nginx -c /usr/local/nginx/conf/nginx.conf
    sleep 2
    counter=$(ps -C nginx --no-heading|wc -l)
    if [ "${counter}" = "0" ]; then
        systemctl stop keepalived
    fi
fi
```

#### 启动服务

分别在主机和从机上启动nginx与keepalived。

```shell
nginx -c /usr/local/nginx/conf/nginx.conf
systemctl start keepalived
```

#### 测试服务

若顺利启动主备模式后则可以通过访问虚拟IP的地址得到想要的结果

```   shell
curl -i 192.168.0.251   #返回为nginx代理的服务即可
```

手动关闭主机nginx后，可以很快重新恢复服务

```shell
nginx -s stop
ps aux | grep nginx  #一两秒后nginx进程自动恢复则正常
```

手动关闭主机keepalived后，短暂延时后可以正常访问nginx代理的服务

```shell
systemctl stop keepalived
curl -i 192.168.0.251    #短暂延时后，可以正常根据VIP访问
```



### 遇到的问题

**第一次启动失败**

通过`sytemctl status keepalived`查看状态发现状态为**dead**，核心提示如下

```text
Keepalived_vrrp exited with permanent error CONFIG. Terminating
```

提示出现永久的配置出错，一时也发现不了错误，配置文件也没动过，所以第一时间去看日志。keepalived的日志文件默认在`/var/log/message`中，通过日志文件，得到线索如下

```
keepalived.service: Can't open PID file /var/run/keepalived.pid (yet?) after start: No such file or directory
```

上述日志提示进程ID文件不存在，手动创建即可。

```text
WARNING - interface eth0 for vrrp_instance VI_1 doesn't exist
Non-existent interface specified in configuration
```

上述日志提示配置文件中网卡 **eht0** 不存在，通过`ip addr`命令发现，虚拟机中的网卡名为**ens33**。修改keepalived配置文件，将网卡名称替换为实际存在的即可。其中keepalived的配置文件的默认位置为`/etc/keepalived/keepalived.conf`

---

**第二次启动失败**

将日志提示的错误修改完成后重新启动keepalived发现，还是启不来，并提示使用`journalctl -Ex`命令查看系统日志

系统日志如下

```text
SELinux is preventing keepalived from read access on the file keepalived.pid. For complete SELinux messages run: seal>
SELinux is preventing keepalived from read access on the file keepalived.pid.
```

提示安全系统**SELinux** 阻止读取进程ID文件，此时可以根据提示生成本地策略以允许此访问

```shell
# ausearch -c 'keepalived' --raw | audit2allow -M my-keepalived
# semodule -X 300 -i my-keepalived.pp
```

或者也可以关闭SELinux的Enforcing模式，开启宽容模式

```shell
# setenforce 0
```

---

**主机与从机均绑定了VIP**

1. 首先执行命令` tcpdump -i ens33 vrrp -n`查看下网卡`ens33`上的组播报文

   ![](https://cdn.jsdelivr.net/gh/noslime/noslime.github.io@master/source/images/tcpdumvrrp.png)

   可见，主机20和备机22都在发送组播报文

2. 主从模式中，备机收到报文后是不会再发的，可见备机没有收到组播消息。初步判断防火墙问题。

3. 首先直接关闭防火墙，以*centos7*为例，

   ```shell
   # systemctl stop firewalld 
   ```

   可见关闭后只有主机在发报文

   ![](https://cdn.jsdelivr.net/gh/noslime/noslime.github.io@master/source/images/tcpdumpvrrp2.png)

4. 通过`ip add` 发现只有主机绑定了 *192.168.0.251*这个虚拟IP，符合了我们的预期

   主机部分

   ![](https://cdn.jsdelivr.net/gh/noslime/noslime.github.io@master/source/images/keepalivevip2.png)

   备机部分

   ![](https://cdn.jsdelivr.net/gh/noslime/noslime.github.io@master/source/images/keepalivevip1.png)

5. 更优雅一点，通过防火墙开启vrrp组播通信的权限，而不是直接关闭防火墙，命令如下

   ```bash
   # firewall-cmd --direct --permanent --add-rule ipv4 filter INPUT 0 --in-interface ens33 --destination 224.0.0.18 --protocol vrrp -j ACCEPT
   # firewall-cmd --reload
   ```

   其中**224.0.0.18** 是vrrp组播通信的默认广播地址。

   ---

## 二、双主模式

### 简介

主从模式中，一台做主，一台做备。虽然一定程度上实现了高可用，但备机大多数情况下处于浪费状态。为了解决备机的浪费问题，可以让两台机器互为主备，即每一台机器既担当主机的角色，又拥有备机的身份，这就是双主模式。双主模式对外提供两个VIP。

### 实现过程

双主模式只要在*keepalived*的配置文件中配置两个实例即可，在主备模式的基础上，只需要以下两步：

1、在主机配置文件中增加备机实例

```text
vrrp_instance nginx02 {
    state BACKUP
    interface ens33
    mcast_src_ip 192.168.0.20
    virtual_router_id 68
    priority 80
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.0.252
    }
	track_script {
        check_nginx
    }
}
```

2、在备机配置文件中增加主机实例

```text
vrrp_instance nginx02 {
    state MASTER
    interface ens33
    mcast_src_ip 192.168.0.22
    virtual_router_id 68
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.0.252
    }
	track_script {
        check_nginx
    }
}
```

> 其中，同原实例相比，需要修改的部分有：
>
> 实例名称：vrrp_instance
>
> 实例初始状态：state
>
> 节点优先级：priority
>
> 虚拟IP地址：virtual_ipaddress
>
> 虚拟路由ID：virtual_router_id  相同vrid的为一组