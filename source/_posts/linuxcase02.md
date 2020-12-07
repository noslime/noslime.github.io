---
title: linux设置静态IP
date: 2020-11-28 14:19:22
author: noslime
categories: linux
tags: 
	- linux
	- case
keywords: linux
---

### CentOS  7  设置静态IP

1. 查看网卡信息

   执行`ifconfig`命令查看网卡名称

   ![](https://cdn.jsdelivr.net/gh/noslime/noslime.github.io@master/source/images/linuxifconfig.png)

   如图网卡名称为ens33

   在`/etc/sysconfig/network-scripts` 文件夹下查找ens33对应的配置文件，如在我的系统中为`ifcfg-ens33` ，首先备份配置文件（若文件不存在，可自行创建一个）

   ```shell
   cp /etc/sysconfig/network-scripts/ifcfg-ens33 /etc/sysconfig/network-scripts/ifcfg-ens33.bakup
   ```

2. 然后在其中添加以下语句设置静态IP，请勿复制注释信息

   ```shell
   DEVICE="ens33"            #描述网卡对应的设备别名
   BOOTPROTO="static"        #设置网卡获得ip地址的方式，可能的选项为static，默认dhcp动态获取
   IPADDR="192.168.0.20"       #设置的静态IP地址
   NETMASK="255.255.255.0"     #子网掩码
   GATEWAY="192.168.0.1"       #网关地址
   DNS1="223.5.5.5"		   #dns信息
   DNS2="8.8.8.8"			   #dns信息
   ONBOOT="yes"			  # 开机启用
   ```
   
3. 重启网络服务

   ```shell
   systemctl restart network  //centos 7
   或
   nmcli c reload		//centos 8  需重启
   ```

### Ubuntu 20.04 设置静态IP

​	ubuntu20版本系统设置静态IP需要修改`/etc/netplan` 下面的`1-network-manager-all.yaml`文件

1. 同样执行`ifconfig`查看当前网卡信息

   网卡依旧是ens33，就不截图了

2. 备份原配置文件

   ```shell
   sudo cp /etc/netplan/01-network-manager-all.yaml /etc/netplan/01-network-manager-all.yaml.backup
   ```

3. 修改`1-network-manager-all.yaml`文件为：

   ```yaml
   # Let NetworkManager manage all devices on this system
   network:
     version: 2
     renderer: NetworkManager
     ethernets:
     	ens33:		#网卡
     	  dhcp4: false #关闭动态获取ip
         addresses: [192.168.0.21/24]   #IP及掩码长度
         geteway4: 192.168.0.1    #网关
         nameservers:
         	addresses: [223.5.5.5, 8.8.8.8]  #DNS
   ```

4. 使更改生效

   ```shell
   sudo netplan --debug apply
   ```

   

