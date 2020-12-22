---
title: Redis 配置文件
date: 2020-12-17 14:02:03
author: noslime
top: false
cover: false
categories: 数据库
tags: 
	- REDIS
	- NOSQL
	- CONFIG
keywords:  redis配置文件
summary: 本文总结下redis常用配置项,现在软件开发中好多技术都是约定先于配置，而配置又大于编码。而想要掌握一门技术，掌握其配置文件是极其必要的。
---

本文总结下redis常用配置项, 现在软件开发中好多技术都是约定先于配置，而配置又大于编码。而想要掌握一门技术，掌握其配置文件是极其必要的。同时在实际生产环境中，正确的配置往往可以大大提高系统的可用性。

---

## 配置简介

Redis可以在没有配置文件的情况下通过内置的配置来启动，但是这种启动方式只适用于开发和测试。合理的配置Redis的方式是提供一个Redis配置文件，这个文件通常叫做 `redis.conf`，一般就在Redis安装目录下。 

对于配置文件的修改配置，除手动修改配置文件外，从Redis 2.8开始，可以使用[CONFIG REWRITE](https://redis.io/commands/config-rewrite)，它会自动扫描redis.conf版归档并更新与当前配置值不匹配的字段。不添加不存在但设置为默认值的字段。配置文件中的注释将被保留。

以下为**6.0**版本的配置文件概览（依据配置块在配置文件中出现先后排序）

![](C:\Users\silen\Desktop\REDIS-CONFIG.png)

## 配置块介绍

Redis完整的配置文件正如上图所示大致可以分为24部分，本文将根据复杂度，与本人理解详细介绍一部分，有些配置可能会一笔带过。开始之前，先介绍下redis的单位表示，其中符号对大小写不敏感

```text
1k => 1000 bytes
1kb => 1024 bytes
1m => 1000000 bytes
1mb => 1024*1024 bytes
1g => 1000000000 bytes
1gb => 1024*1024*1024 bytes
```

### Redis include

**1、include**

- 默认：未配置

- 取值：配置文件所在位置

- 说明：Redis可以包含进来其他的配置文件片段，其中Redis指令后出现的会覆盖先出现的指令，所以可以根据需要选择include块所在的位置。

- 示例：

  ```text
  include /path/to/local.conf
  include /path/to/other.conf
  ```

### Redis modules

**1、loadmodule**

- 默认：未配置

- 取值：加载功能模块文件的具体位置

- 说明：Redis 模块功能通过使用外部模块来扩展 Redis 功能，Redis可以根据需要在启动时加载模块。

- 示例：

  ```text
  loadmodule /path/to/my_module.so
  loadmodule /path/to/other_module.so
  ```

### Redis NETWORK

**1、bind** 

- 默认：`127.0.0.1`
- 取值：本机网卡所在的IP地址，其中两个特殊值`127.0.0.1`代表只能本机访问，`0.0.0.0`代表本机所有地址

- 说明：绑定指令设置redis接受请求来源于哪个网络接口（网卡），可以绑定多个。若不设置则来自任意网络接口的请求均可访问。默认设置为只能本机访问。

- 示例：

  ```text
  # 本机访问
  bind 127.0.0.1
  # 全都能访问
  bind 0.0.0.0
  ```

- 补充：由于redis侧重点在于性能响应速度，此配置并不能细粒度地控制外网指定IP访问，只能设置本机能访问或全部都能访问。若要设置指定IP，可通过添加防火墙规则进行控制

  ```shell
  firewall-cmd  --add-rich-rule="rule family="ipv4" source address="192.168.1.25" port protocol="tcp" port="80" accept" --permanent
  systemctl restart firewalld.service
  ```

**2、protected-mode**

- 默认：yes 即开启
- 取值：yes 或 no

- 说明：保护模式默认是开启的，但只有没有绑定网络接口，而且没有设置访问密码的情况下，保护模式才发挥效用

- 示例：

  ```text
  protected-mode yes
  ```

**3、port**

- 默认：6379
- 取值：1024 - 65535之间的可用端口即可

- 顾名思义，即redis对外提供服务的端口号

  ```text
  port 6379
  ```

**4、tcp-backlog**

- 默认：511
- 取值：合适的队列长度即可

- 说明：此参数用来配置完成tcp三次握手队列的长度，用于应对高并发的环境，虽然默认设置为511，但会受到Linux内核参数somaxconn的限制。若不进行修改，redis启动会有以下警告，提示该参数未发挥作用

  ```text
  WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
  ```

  可以手动修改somaxconn的值，以使配置生效

  *临时设置*：

  `echo 1024 > /proc/sys/net/core/somaxconn `

  *永久设置*：

  在`/etc/sysctl.conf`中添加一行配置：`net.core.somaxconn = 1024`

  然后执行命令`sysctl -p`，使之永久生效

- 示例：

  ```text
  tcp-backlog 511
  ```

**5、unixsocket**

- 默认：未配置

- 取值：redis sock文件的存储位置

- 说明：默认redis的连接方式是TCP，我们还可以让 Redis 以 Unix Socket 的方式运行，以避免 TCP/IP 的性能瓶颈，在高访问场景往往能够实现不错的性能提升。

- 示例：

  ```text
  unixsocket /tmp/redis.sock
  ```

**6、unixsocketperm**

- 默认：未配置
- 取值：linux权限码

- 说明：Unix socket文件的权限

  ```text
  unixsocketperm 700
  ```

**7、timeout**

- 默认：0 
- 取值：一个合适的整数值

- 说明：客户端空闲时，是否设置超时时间，0表示不设置。

- 示例：

  ```text
  # Close the connection after a client is idle for N seconds (0 to disable)
  timeout 0
  ```

**8、tcp-keepalive**

- 默认：300
- 取值：一个合适的整数值

- 设置redis客户端tcp的保活时间，在指定时间内会一直探测连接是否存活，官方建议300.

- 示例：

  ```text
  tcp-keepalive 300
  ```

### Redis TLS/SSL

配置redis加密传输，此部分需要掌握SSL协议，熟悉对称与非对称加解密算法等。详细可见[encryption](https://redis.io/topics/encryption)，文档描写的很详细。

### Redis GENERAL

redis的通用模块

**1、daemonize**

- 默认：no

- 取值：yes|no

- 说明：redis服务默认是运行在前台的，可以通过设置让他以守护进程运行在后台。

- 示例：

  ```text
  daemonize yes
  ```

**2、supervisor**

- 默认：no
- 取值：no|upstart|systemd|auto

- 说明：设置redis进程管理程序，可选项有`upstart` 和`systemd`以及根据系统环境自动检测，默认关闭。

- 示例：

  ```text
  supervised no
  ```

**3、pidfile**

- 默认：`/var/run/redis_6379.pid`

- 取值：pid文件的具体位置

- 说明：进程id文件所在位置。如果指定，服务启动时创建，停止时销毁。

- 示例：

  ```text
  pidfile /var/run/redis_6379.pid
  ```

**4、loglevel**

- 默认： notice

- 取值：debug|verbose|notice|warning

- 说明：redis服务日志的级别，以上四个取值日志详实度从左到右递减，一般生产环境，notice即可。

- 示例：

  ```text
  loglevel notice
  ```

**5、logfile**

- 默认：""

- 取值：日志文件的位置

- 说明：日志文件的名称，可以带有绝对路径

- 示例：

  ```text
   logfile "/var/log/redis/redis.log"
  ```

**6、syslog-enabled**

- 默认：未配置

- 取值：yes|no

- 说明：是否将日志记录进系统日志

- 示例：

  ```text
  syslog-enabled no
  ```

**7、syslog-ident**

- 默认：未配置

- 取值：系统日志标识ID，一般为应用程序名称

- 说明：指定系统日志标识。

- 示例：

  ```text
  syslog-ident redis
  ```

**8、syslog-facility**

- 默认：未配置

- 取值：Must be USER or between LOCAL0-LOCAL7

- 说明：指定redis系统日志的设备

- 示例

  ```text
  syslog-facility local0
  ```

**9、databases**

- 默认：16

- 取值：合适的整数

- 说明：redis数据库的数量，默认16个，一般一个业务用一个库。

- 示例：

  ```text
  databases 16
  ```

**10、always-show-logo**

- 默认：yes

- 取值：yes | no

- 说明：是否在redis启动时显示logo艺术字，只有输出到标准输出时才显示。

- 示例：

  ```
  always-show-logo yes
  ```

### Redis SNAPSHOTTING

redis快照部分

**1、save**

- 默认：

  ```text
  save 900 1
  save 300 10
  save 60 10000
  ```

- 取值：save seconds changes

- 说明：保存redis数据快照的策略，第一个参数为时间，单位为秒，第二个参数为key更新的次数，默认设置即为900秒内有一个键更新，则触发建立快照；300秒内有10个键值更新，触发建立快照；60秒内10000个键值更新，触发建立快照。若要禁用快照功能，save设空字符串即可。

- 示例即默认

**2、stop-writes-on-bgsave-error**

- 默认：yes

- 取值：yes | no

- 说明：是否停止写RDB文件，在后台发生错误后。若设置为yes则发生错误后，只有重新启动后，才会重新开启写操作。

- 示例：

  ```text
  stop-writes-on-bgsave-error yes
  ```

**3、rdbcompression**

- 默认： yes

- 取值：yes | no

- 说明：是否需要压缩RDB快照文件，默认使用`LAF`算法进行压缩。

- 示例：

  ```
  rdbcompression yes
  ```

**4、rdbchecksum**

- 默认：yes

- 取值：yes | no

- 说明：redis 5以后的版本允许在RDB快照文件尾部加入一个`CRC64`校验和, 用以防止文件损坏，提高可靠性。但会消耗10%左右的性能。

- 示例：

  ```text
  rdbchecksum yes
  ```

**5、dbfilename** 

- 默认：dump.rdb

- 取值：自己定义的快照文件名称

- 说明：快照文件的名称

- 示例：

  ```text
  dbfilename dump.rdb
  ```

**6、rdb-del-sync-files**

- 默认：no

- 取值：yes | no

- 说明：在没有启用持久性的实例中删除复制使用的RDB文件。默认情况下，此选项处于禁用状态。请注意，此选项仅适用于同时禁用AOF与RDB持久性的情况，否则将完全忽略

- 示例：

  ```text
  rdb-del-sync-files no
  ```

**7、dir**

- 默认：./

- 取值：一个目录的路径

- 说明：快照文件RDB，与AOF文件都会存储在这里

- 示例：

  ```text
  dir ./
  ```

### Redis REPLICATION

redis 主从复制部分

**1、replicaof**

- 默认：未配置

- 取值：masterip masterport

- 说明：配置该redis数据库主机IP与端口号

- 示例：

  ```text
  replicaof 127.0.0.1 6380
  ```

**2、masterauth**

- 默认：未配置

- 取值：主机需要的认证密码

- 说明：如果在主从复制的主机上设置了`requirepass`，从机在此处必须设置正确才能完成同步的请求获得数据。

- 示例：

  ```text
  masterauth master-password
  ```

**3、masteruser**

- 默认：未配置

- 取值：用户名

- 说明：如果在redis 6以后的版本中使用了[ACLs](https://redis.io/topics/acl)，仅仅设置上个配置项中的访问密码不够，还需要设置访问的用户，即本设置项的作用。

- 示例：

  ```text
  masteruser username
  ```

**4、replica-serve-stale-data**

- 默认：yes

- 取值：yes | no

- 说明：如果设置为是的话，则从机会一直回复客户端的请求，哪怕数据过期，或者返回一个空串当第一次同步的时候；如果设置为否的话，绝大部分命令请求会直接返回客户端错误信息**SYNC with master in progress**，除了：`INFO, REPLICAOF, AUTH, PING, SHUTDOWN, REPLCONF, ROLE, CONFIG, SUBSCRIBE,UNSUBSCRIBE, PSUBSCRIBE, PUNSUBSCRIBE, PUBLISH, PUBSUB, COMMAND, POST,
  HOST and LATENCY`这些命令。

- 示例：

  ```text
  replica-serve-stale-data yes
  ```

**5、replica-read-only**

- 默认：yes

- 取值：yes | no

- 说明：设置从机是否只读

- 示例

  ```text
  replica-read-only yes
  ```

**6、repl-diskless-sync**

- 默认：no

- 取值：yes | no

- 说明：新增从机或从机恢复连接时，需要一次向主机完全同步的请求。而主机应对这种请求，有两种策略：一种是先写到硬盘上，然后传输到指定从机，适合多个从机同时请求；另一种则是无盘传输，这种适合慢硬盘，大带宽的情形，后来的请求需要进入队列。

- 示例：

  ```text
  repl-diskless-sync no
  ```

**7、repl-diskless-sync-delay**

- 默认：5

- 取值：整数，单位为秒

- 说明：主机部分设置，当无盘传输一旦开始的情况下，主服务器不能为新的从机提供服务，因此服务器会设置一个等待延时，以便让更多从机加入。

- 示例

  ```
  repl-diskless-sync-delay 5
  ```

**8、repl-diskless-load**

- 默认：disabled

- 取值：disabled | on-empty-db |swapdb

- 说明：从机部分设置，当主机开始无盘传输时，我们可以选择，先保存到本地完整的RDB快照文件，然后再同步；或者只有当从机数据为空时，直接读取同步；或者先将完整的RDB快照保存到内存中，只有当主机传输完毕时，再从内存同步完整的快照。

- 示例：

  ```text
  repl-diskless-load disabled
  ```

**9、repl-ping-replica-period**

- 默认：10

- 取值： 合适的时间间隔，单位秒

- 说明：从机周期性PING主机的时间间隔，定期检查主机状态。

- 示例：

  ```text
  repl-ping-replica-period 3
  ```

**10、repl-timeout**

- 默认：60

- 取值：时间，单位为秒

- 说明：主备同步的超时时间

- 示例：

  ```text
  repl-timeout 60
  ```

**11、repl-disable-tcp-nodelay**

- 默认：no

- 取值：yes | no

- 说明：是否开启主从机器简单tcp延时，默认不开启，适合主从机间通信良好，最好在同一机房。而当网络高负荷，主从机再不同机房时，可以开启。降低传输频率。

- 示例：

  ```text
  repl-disable-tcp-nodelay no
  ```

**12、repl-backlog-size**

- 默认：未配置

- 取值：累计缓存大小，单位mb

- 说明：复制缓冲区大小，这是一个环形复制缓冲区，用来保存最新复制的命令。当redis存在从机时，若从机断开连接，重新连接时不一定需要进行全量同步。设置了累计缓存大小后，从机首先获取累计缓存，进行部分同步。若部分同步不能解决问题，再进行全量同步。

- 示例：

  ```text
  repl-baklog-size 1mb
  ```

**13、repl-backlog-ttl** 

- 默认：未配置

- 取值：合适的正整数，单位秒

- 说明：复制缓冲区的过期时间，0表示永不释放这块的内存。

- 示例：

  ```text
  repl-backlog-ttl 3600
  ```

**14、replica-priority** 

- 默认：100

- 取值：一个整数

- 说明：从机的优先级，数字越小，优先级越高。例如有三个从机优先级分别为10，25，100，则再主机不可用后，redis通过哨兵机制，选择优先为10的从机顶替主机。 其中特殊值 0 代表永不能成为主机。

- 示例：

  ```text
  replica-priority 100
  ```

**15、min-replicas-to-write  & min-replicas-max-lag**

- 默认： min-replicas-to-write 0   min-replicas-max-lag 10

- 取值：均为一个整数值，前者单位为个，后者为秒

- 说明：这两条配置限制主机是否继续接受写命令，前者表示最少的从机节点为几个，后者表示根据从副本接收的最后一次ping计算的延迟时间。只有存在符合延迟时间的指定个数的健康从机，主机才继续写命令。

- 示例：

  ```text
  min-replicas-to-write 3
  min-replicas-max-lag 10
  ```

**16、replica-announce-ip  & replica-announce-port**

- 默认： 未配置

- 取值： 前者为IP地址，后者为端口号

- 说明： 当从及IP及端口固定时，从机上报的IP与端口号，供哨兵发现各个从机。

- 示例：

  ```text
  replica-announce-ip 192.168.0.20
  replica-announce-port 6379
  ```

### Redis KEYS TRACKING

Redis辅助支持的客户端缓存，详情可见[Redis server-assisted client side caching]( https://redis.io/topics/client-side-caching)

**1、tracking-table-max-keys**

- 默认：未配置

- 取值：缓存键的数量

- 说明：redis可以帮助实现客户端缓存，并维持一个无效表，用以跟踪键值，当键值改变时，通知客户端重新获取，而此参数定义无效表的大小。需要注意的是，若使用的是广播模式的键值追踪，则此参数无效。

- 示例：

  ```text
  tracking-table-max-keys 1000000
  ```

### Redis SECURITY

Redis安全模块，由于Redis速度很快，外部用户可以尝试 每秒100万个密码。这意味着你应该使用很强的密码，否则很容易被破解。 请注意，因为密码实际上是客户端与服务器之间的共享机密，不应该被任何人记住密码可以是/dev/urandom中的一个长字符串，所以使用长而不可预测的密码避免暴力攻击将是可能的。

**1、user**

- 命令模式： `user <username> ... acl rules ...`
- 默认：未配置

- 说明：指定用户名，是否启用规则，访问控制规则及认证密码，可在客户端通过`ACL LIST`指令获取当前的权限规则列表，具体的权限赋予规则详情可见[Redis ACL](https://redis.io/topics/acl)。

- 示例：

  ```text
  # 允许默认用户所有频道所有命令的权限，且不需要密码认证
  user default on nopass ~* &* +@all
  ```

**2、acllog-max-len**

- 默认：128

- 取值：一个正整数

- 说明：ACL日志跟踪与ACL关联的失败命令和身份验证事件。此配置设置日志的长度，ACL日志存储在内存中。

- 示例：

  ```text
  acllog-max-len 128
  ```

**3、aclfile** 

- 默认：未配置

- 取值：ACL文件所在位置

- 说明：ACL文件所在位置，可以在外部配置user规则，只需通过此配置引入即可，但需要注意的是，配置文件中配置与ACL文件是冲突的，只能在一处配置。

- 示例：

  ```text
  aclfile /etc/redis/users.acl
  ```

**4、requirepass**

- 默认：未配置

- 取值：密码值

- 说明：此处的密码仅仅是设置默认用户`default`的密码

- 示例：

  ```text
  requirepass foobared
  ```

~~**5、rename-command**~~ 

- 命令模式：rename-command commandname  newcommandname

- 默认：未配置

- 说明：重命名命令名称，已经弃用，新版本建议用ACL进行权限控制。需要注意的是，更改命令记录到AOF文件或传输给从机可能会导致错误。

- 示例：

  ```text
  rename-command CONFIG "SecretConfig"
  ```

### Redis CLIENTS

Redis客户端配置

**1、maxclients** 

- 默认：10000

- 取值：客户端连接数量

- 说明：允许连接的客户端数量，最小为32（作为内部文件描述符使用），而一旦超过连接数量后的连接返回错误`max number of clients reached`。重要提示：当使用Redis集群时，最大连接数也与集群总线共享：集群中的每个节点将使用两个连接，一个传入，另一个传出。在非常大的簇的情况下，相应地调整限制的大小是很重要的。

- 示例：

  ```text
  maxclients 100000
  ```

### Redis MEMORY MANAGEMENT

Redis内存管理配置

## 官方完整配置

[=========================传送门=========================](https://raw.githubusercontent.com/redis/redis/6.0/redis.conf) 

