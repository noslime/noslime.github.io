---
title: keepalived's log
date: 2020-12-09 17:01:36
author: noslime
top: false
cover: false
categories: 后端
tags: 
	- KEEPALIVED
	- LOG
keywords: keepalived
summary: 上次搭建nginx集群，并没有处理keepalived的日志，比较混乱。今天实现日志文件自定义输出位置，以便查看。
---

上次搭建nginx集群，并没有处理keepalived的日志，而keepalived的配置文件默认输出到系统日志`/var/log/message`中，比较混乱。今天实现日志文件自定义输出位置，以便查看。

---

s

**1.修改/etc/sysconfig/keepalived**

将`KEEPALIVED_OPTIONS="-D" `修改为`KEEPALIVED_OPTIONS="-D -S 0"`

```text
# Options for keepalived. See `keepalived --help' output and keepalived(8) and
# keepalived.conf(5) man pages for a list of all options. Here are the most
# common ones :
#
# --vrrp               -P    Only run with VRRP subsystem.
# --check              -C    Only run with Health-checker subsystem.
# --dont-release-vrrp  -V    Dont remove VRRP VIPs & VROUTEs on daemon stop.
# --dont-release-ipvs  -I    Dont remove IPVS topology on daemon stop.
# --dump-conf          -d    Dump the configuration data.
# --log-detail         -D    Detailed log messages.
# --log-facility       -S    0-7 Set local syslog facility (default=LOG_DAEMON)
#
# KEEPALIVED_OPTIONS="-D"
KEEPALIVED_OPTIONS="-D -S 0"
~                                
```

**2.在/etc/rsyslog.conf里添加**

```text
local0.*  /var/log/keepalived.log 
```

**3.重新启动keepalived和rsyslog服务** 

```shell
systemctl restart rsyslog
systemctl restart keepalived
```

**4.查看日志信息**

```shell
tail -f -n 150 /var/log/keepalived.log
```

