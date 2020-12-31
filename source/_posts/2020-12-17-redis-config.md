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

![图一](https://cdn.jsdelivr.net/gh/noslime/noslime.github.io@master/source/images/REDIS-CONFIG.png)

## 配置块介绍

Redis完整的配置文件（6.0版本）正如图一所示大致可以分为24部分，本文将根据复杂度，与本人理解详细介绍一部分，有些配置可能会一笔带过。开始之前，先介绍下redis的单位表示，其中符号对大小写不敏感

```text
1k => 1000 bytes
1kb => 1024 bytes
1m => 1000000 bytes
1mb => 1024*1024 bytes
1g => 1000000000 bytes
1gb => 1024*1024*1024 bytes
```

### Redis INCLUDES

**1、INCLUDE**

- 默认：未配置

- 取值：配置文件所在位置

- 说明：Redis可以包含进来其他的配置文件片段，其中Redis指令后出现的会覆盖先出现的指令，所以可以根据需要选择include块所在的位置。

- 示例：

  ```text
  include /path/to/local.conf
  include /path/to/other.conf
  ```

### Redis MODULES

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

**1、maxmemory**

- 默认：未设置

- 取值：正整数 单位字节bytes

- 说明：redis使用内存的最大限制

- 示例：

  ```text
  maxmemory 1024*1024*1024*8
  ```

**2、maxmemory-policy**

- 默认：noeviction

- 取值：volatile-lru | allkeys-lru | volatile-lfu | allkeys-lfu | volatile-random | allkeys-random |volatile-ttl

  | noeviction

- 说明：当达到设置的最大内存后，redis选择的移除策略，其中lru为最近最久未使用算法，lfu为最近最少使用算法。以下为每个设置的简要介绍。

  - volatile-lru：使用LRU算法清除设置了expire过期时间的数据。
  - allkeys-lru：使用LRU算法清除任意数据。
  - volatile-lfu：使用LFU算法清除设置了expire过期时间的数据。
  - allkeys-lfu：使用LFU算法清除任意数据。
  - volatile-random：随机清除设置了expire过期时间的数据。
  - allkeys-random：随机清除任意数据。
  - volatile-ttl：删除最接近过期时间的数据。
  - noeviction：不删除任何内容，只在写操作时返回一个错误。

  注意：对于上述任何策略，当没有合适的数据删除时，Redis都会在写操作时返回一个错误。

- 示例：

  ```text
  maxmemory-policy noeviction
  ```

**3、maxmemory-samples**

- 默认：5

- 取值：一个正整数

- 说明：上条配置指令中介绍的LRU、LFU和minimal-TTL算法不是精确算法，而是近似算法（为了节省内存），因此您可以调整它的速度或精度。默认情况下，Redis将检查五个键并选择最近使用过的一个，您可以使用以下配置指令更改样本大小。
  默认值为5会产生足够好的结果。10非常接近真实的LRU，但需要更多的CPU。3更快，但不是很准确。

- 示例：

  ```text
  maxmemory-samples 5
  ```

**4、replica-ignore-maxmemory**

- 默认：yes

- 取值：yes | no

- 说明：从redis版本5开始，redis从机默认忽略maxmemory的设置，而内存清除策略也只工作在master主机上，从机清除依赖主机传递的删除命令，从而保持一致性。但是如果你的从机时可写入的，且能保证所有写操作都是幂等的，可以更改此选项。

- 示例：

  ```text
  replica-ignore-maxmemory
  ```

**5、active-expire-effort**

- 默认：1
- 取值：[1-10]
- 说明：影响redis定期删除策略的一个参数，effort改变了取样基数、定期回收任务间隔，过期键可占内存的最大百分比，最大占用cpu的百分比等，effort越大，cpu负担越重，所以根据自己的需要设置effort的值。这是内存、CPU和延迟之间的权衡。

- 示例：

  ```text
  active-expire-effort 1
  ```

### Redis LAZY FREEING

Redis有两种方式来删除键。一种叫做DEL，是对对象的阻塞性删除。这意味着服务器停止处理新命令，以便以同步方式回收与对象关联的所有内存。如果删除的键与一个小对象关联，则执行DEL命令所需的时间非常小，与Redis中大多数其他O（1）或O（log_N）命令相比是非常小的。但是，如果键与一个包含数百万个元素的聚合值相关联，服务器可能会长时间（甚至几秒钟）阻塞以完成操作。

基于上述原因，Redis还提供了UNLINK（non-blocking DEL）和FLUSHDB命令的ASYNC选项等非阻塞删除原语，以便在后台回收内存。这些命令在固定时间内执行。另一个线程将以最快的速度增量释放后台的对象。

FLUSHALL和FLUSHDB的DEL、UNLINK和ASYNC选项由用户控制。这取决于应用程序的设计来理解何时使用其中一个。然而，由于其他操作的副作用，Redis服务器有时不得不删除密钥或刷新整个数据库。具体来说，在以下场景中，Redis独立于用户调用删除对象：

1） 回收内存时，由于maxmemory和maxmemory策略配置，为了给新数据腾出空间，而不超过指定的内存限制。

2） 因为expire：必须从内存中删除具有相关生存时间的密钥（请参阅expire命令）。

3） 因为命令的副作用是将数据存储在可能已经存在的键上。例如，重命名命令可能会删除旧的键内容，当它被另一个替换时。类似地，SUNIONSTORE或SORT with STORE选项可能会删除现有的键值。SET命令本身删除指定键的任何旧内容，以便用指定的字符串替换它。

4） 在复制过程中，当从机与其主机执行完全重新同步时，整个数据库的内容将被删除，以便加载刚刚传输的RDB文件。

在上述所有情况下，默认情况是以阻塞方式删除对象，就像调用DEL一样。但是，您可以具体地配置每种情况，以便使用以下配置指令以非阻塞方式释放内存，就像调用UNLINK一样。

```text
# 默认全为 no， 可取值：yes | no
lazyfree-lazy-eviction yes
lazyfree-lazy-expire yes
lazyfree-lazy-server-del yes
replica-lazy-flush yes
```

其中lazyfree可译为惰性删除或延迟释放；当删除键的时候，redis提供异步延时释放key内存的功能，把key释放操作放在bio(Background I/O)单独的子线程处理中，减少删除big key对redis主线程的阻塞。有效地避免删除big key等带来的性能和可用性问题。

在用UNLINK调用替换用户代码DEL调用并不容易的情况下，还可以使用以下配置指令修改DEL命令的默认行为，使其与UNLINK完全相同：

```text
# 默认为no，可取值：yes | no
lazyfree-lazy-user-del yes
```

### Redis THREADED I/O

输入输出线程，现在还可以在不同的I/O线程中处理Redis客户端的套接字读写。由于编写速度非常慢，通常Redis用户使用流水线来提高每个核心的Redis性能，并生成多个实例以扩展更多。使用I/O线程，可以轻松地将Redis的速度提高两倍，而无需使用流水线或实例切分。

默认情况下，线程设置被禁用，redis建议只在至少有4个或更多内核的计算机中启用它，并至少保留一个备用内核。使用超过8个线程不大可能有多大帮助。redis还建议仅当您实际存在性能问题时才使用线程I/O，因为Redis实例会占用相当大比例的CPU时间，否则使用此功能没有任何意义。

例如，如果你有一个4核的容器，尝试使用2或3个I/O线程，如果你有一个8核，尝试使用6个线程。要启用I/O线程，请使用以下配置指令：

```text
 io-threads 4
```



将`io-threads`设置为1只会像往常一样使用主线程。当启用I/O多线程时，我们只使用线程进行写入，即线程写入（2）系统调用并将客户机缓冲区传输到套接字。但是，也可以使用以下配置指令启用读取线程和协议解析，方法是将其设置为“yes”：

```text
io-threads-do-reads no
```

通常线程读取并没有多大帮助。

注1：此配置指令不能在运行时通过`CONFIG SET`进行更改。启用SSL时，此功能当前不起作用。

注2：如果你想用Redis benchmark来测试Redis的加速，请确保你也在线程模式下运行基准测试，使用--threads选项来匹配Redis线程的数量，否则你将无法测试出提升。

### Redis KERNEL OOM CONTROL

Linux 内核有个机制叫OOM killer（Out-Of-Memory killer），该机制会监控那些占用内存过大，尤其是瞬间很快消耗大量内存的进程，为了防止内存耗尽而内核会把该进程杀掉。

**1、oom-score-adj**

- 默认：no

- 取值：yes | no

- 说明：此功能启用，使得redis主动给自己所有的进程进行评分，提供OOM killer机制触发时，进程的灭杀顺序。

- 示例：

  ```text
  oom-score-adj no
  ```

**2、oom-score-adj-values**

- 指令模式及默认值：`oom-score-adj-values 0 200 800`
- 说明：当`oom-score-adj`指令启用时，这条指令指定主机，从机和后台子进程的分数。分数值在区间[-1000, 1000]内，值越大越先被杀掉。
- 示例同指令模式

### Redis APPEND ONLY MODE 

默认情况下，Redis将数据集异步转储到磁盘上。这种模式在许多应用程序中已经足够好了，但是Redis进程出现问题或断电可能会导致几分钟的写丢失（取决于配置的保存点）。
AOF模式是一种可选的持久性模式，它提供了更好的持久性。例如，如果使用默认的数据fsync策略，Redis在服务器断电等戏剧性事件中可能只会丢失一秒钟的写入操作，或者如果Redis进程本身发生了问题，但操作系统仍在正常运行，则只会丢失一次写入操作。

**1、appendonly**

- 默认：no

- 取值：yes | no

- 说明：是否开启AOF持久化，AOF和RDB两种持久化可以并存，并且redis启动时，会优先加载AOF持久化文件。

- 示例：

  ```text
  appendonly yes
  ```

**2、appendfilename**

- 默认：appendonly.aof

- 取值：文件名

- 说明：AOF持久化文件名称

- 示例：

  ```text
  appendfilename "appendonly.aof"
  ```

**3、appendfsync**

- 默认：everysec

- 取值：always | everysec | no

- 说明：追加同步的策略

  - no：不会调用fsync， 由系统决定什么时候刷磁盘。 速度最快。

  - always：每次写操作都调用fsync同步AOF文件。速度最慢，但安全性高。
  - everysec：每秒调用一次fsync同步AOF文件。以上两种方式的折中方案。

- 示例

  ```text
  appendfsync everysec
  ```

**4、no-appendfsync-on-rewrite**

- 默认：no

- 取值：yes | no

- 说明：当AOF fsync策略设置为always或everysec，并且后台保存进程（后台保存或AOF日志后台重写）正在对磁盘执行大量I/O时，在某些Linux配置中，Redis可能会在fsync（）调用上阻塞太长时间。请注意，目前还没有解决此问题的方法，因为即使在不同的线程中执行fsync，也会阻止我们的同步写入调用。
  为了缓解这个问题，可以使用此选项来防止在进行BGSAVE或bwriteAOF时在主进程中调用fsync（）。
  这意味着，当另一个子进程在存储时，Redis的持久性与“appendfsync none”相同。实际上，这意味着在最坏的情况下（使用默认的Linux设置），可能会丢失最多30秒的日志。

- 示例：

  ```text
  no-appendfsync-on-rewrite no
  ```

**5、auto-aof-rewrite-percentage & auto-aof-rewrite-min-size**

- 默认及示例：

  ```text
  auto-aof-rewrite-percentage 100
  auto-aof-rewrite-min-size 64mb
  ```

- 取值：第一条为百分比，100表示比原来大一倍；第二条表示触发重写规则的最小AOF文件大小

- 说明：AOF采用文件追加方式，文件会越来越大为避免出现此种情况，新增了重写机制，当AOF文件的大小超过所设定的阈值时，Redis就会启动AOF文件的内容压缩，调用指令`bgrewriteaof`进行重写，只保留可以恢复数据的最小指令集。

  Redis会记录上次重写时的AOF大小，默认配置是当AOF文件大小是上次rewrite后大小的一倍且文件大于64M时触发。其中AOF重写时会fork出一条新进程来将文件重写(也是先写临时文件最后再rename)，遍历新进程的内存中数据，每条记录有一条的Set语句。重写aof文件的操作，并没有读取旧的aof文件，而是将整个内存中的数据库内容用命令的方式重写了一个新的aof文件。

**6、aof-load-truncated**

- 默认：yes	

- 取值：yes | no

- 说明：redis发现AOF文件尾部命令错误的时候是否继续启动服务。当运行Redis的系统崩溃时，尤其是在没有data=ordered选项的情况下挂载ext4文件系统时，可能会发生这种情况（但是，当Redis本身崩溃或中止，但操作系统仍然正常工作时，这种情况就不会发生）。如果选择的是yes，当截断的aof文件被导入的时候，会忽视有错误的尾部，并且自动发布一个log给客户端然后load。如果是no，用户必须手动redis-check-aof修复AOF文件才可以。

- 示例：

  ```text
  aof-load-truncated yes
  ```

**7、aof-use-rdb-preamble yes**

- 默认：yes

- 取值：yes | no

- 说明：4.0以后出现的混合持久化模式，在重写AOF文件时，Redis能够在AOF文件中使用RDB前导码以加快重写和恢复。启用此选项时，重写的AOF文件由两个不同的节组成：[RDB文件] [AOF tail]，新的AOF文件前半段是RDB格式的全量数据后半段是AOF格式的增量数据。

  加载时，Redis识别出AOF文件以“Redis”字符串开头并加载带前缀的RDB文件，然后继续加载AOF尾部。

- 示例：

  ```text
  aof-use-rdb-preamble yes
  ```


### Redis LUA SCRIPTING

Redis从2.6.0开始支持lua脚本，通过内置的 Lua 解释器，可以使用 EVAL和EVALSHA命令对一次在服务端执行大量操作。Redis 使用单个 Lua 解释器去运行所有脚本，并且， Redis 也保证脚本会以原子性(atomic)的方式执行： 当某个脚本正在运行的时候，不会有其他脚本或 Redis 命令被执行。

`EVAL` 命令要求你在每次执行脚本的时候都发送一次脚本主体(script body)。Redis 有一个内部的缓存机制，因此它不会每次都重新编译脚本，不过在很多场合，付出无谓的带宽来传送脚本主体并不是最佳选择。

为了减少带宽的消耗， Redis 实现了 [EVALSHA](http://www.redis.cn/commands/evalsha.html) 命令，它的作用和 `EVAL` 一样，都用于对脚本求值，但它接受的第一个参数不是脚本，而是脚本的 SHA1 校验和(sum)。

EVALSHA 命令的表现如下：

如果服务器还记得给定的 SHA1 校验和所指定的脚本，那么执行这个脚本 如果服务器不记得给定的 SHA1 校验和所指定的脚本，那么它返回一个特殊的错误，提醒用户使用 EVAL 代替 EVALSHA

**1、lua-time-limit**

- 默认：5000

- 取值：毫秒

- 说明：lua 脚本的最大执行限制时间

  当一个脚本达到最大执行时间的时候，它并不会自动被 Redis 结束，因为 Redis 必须保证脚本执行的原子性，而中途停止脚本的运行意味着可能会留下未处理完的数据在数据集(data set)里面。

  因此，当脚本运行的时间超过最大执行时间后，以下动作会被执行：

  - Redis 记录一个脚本正在超时运行
  - Redis 开始重新接受其他客户端的命令请求，但是只有 [SCRIPT KILL](http://www.redis.cn/commands/script-kill.html) 和 `SHUTDOWN NOSAVE` 两个命令会被处理，对于其他命令请求， Redis 服务器只是简单地返回 BUSY 错误。
  - 可以使用 [SCRIPT KILL](http://www.redis.cn/commands/script-kill.html) 命令将一个仅执行只读命令的脚本杀死，因为只读命令并不修改数据，因此杀死这个脚本并不破坏数据的完整性
  - 如果脚本已经执行过写命令，那么唯一允许执行的操作就是 `SHUTDOWN NOSAVE` ，它通过停止服务器来阻止当前数据集写入磁盘

- 示例：

  ```text
  lua-time-limit 5000
  ```

### Redis REDIS CLUSTER

Resid集群相关的设置

**1、cluster-enabled**

- 默认：未配置

- 取值：yes | no

- 说明：普通的redis实例并不能成为redis集群的一部分，只有节点以集群节点启动才能加入，此处的配置便是让redis以集群节点的形式启动。

- 示例：

  ```text
  cluster-enabled yes
  ```

**2、cluster-config-file**

- 默认：未配置

- 取值：文件名称

- 说明：redis集群系统中，每个节点必须有自己独立的配置文件，此处为配置文件的名称。

- 示例：

  ```text
  cluster-config-file nodes-6379.conf
  ```

**3、cluster-node-timeout**

- 默认：未配置

- 取值：毫秒

- 说明：redis集群节点无法访问多长时间才被是为故障节点。

- 示例：

  ```text
  cluster-node-timeout 15000
  ```

**4、cluster-replica-validity-factor**

- 默认：未配置

- 取值：一个非负整数

- 说明：启动故障切换的时间系数。其中关于它应用场景的官方参考翻译如下：

  如果发生故障的主机的从机的数据看起来太旧，它将避免启动故障转移。

  对于从机来说，没有一种简单的方法来实际精确测量其“数据期限”，因此需要执行以下两种检查：

  1） 如果有多个从机能够进行故障转移，它们会交换消息，以便尝试以最佳复制偏移量（处理来自主服务器的更多数据）为从机提供优势。从机将尝试按偏移量获取其级别，并在故障转移开始时应用与其级别成比例的延迟。

  2） 每个副本都计算最后一次与主副本交互的时间。这可以是上一次接收到的ping或命令（如果主服务器仍处于“已连接”状态），也可以是自与主服务器断开连接后经过的时间（如果复制链接当前处于关闭状态）。如果最后一次交互太旧，从机根本不会尝试故障转移

  其中第二点可以有我们用户把控。指明一个从机是否能够进行故障迁移，看它上次和主机交互过去的时间有没有超过：`(node-timeout * cluster-replica-validity-factor) + repl-ping-replica-period`

  例如：节点超时时间30秒，时间系数为10， 从机检查周期10秒，则从机如果超过310秒没有与主机进行交互，则不能进行故障转移。

  较大的时间系数可能允许数据太旧的从机故障转移到主服务器，而太小的值可能会阻止群集选择从机。
  为了获得最大可用性，可以将时间系数设置为0，这意味着从机将始终尝试故障转移主服务器，而不管它们上次与主服务器交互的时间。（但是他们总是尝试应用与偏移量成比例的延迟）。
  零是唯一能够保证当所有分区恢复时，集群始终能够继续运行的值。

- 示例：

  ```text
  cluster-replica-validity-factor 10
  ```

**5、cluster-migration-barrier**

- 默认：1

- 取值：非负整数

- 说明：`cluster-migration-barrier`属性可以保证redis集群中不会出现孤立主机，当某个主节点的从节点挂掉裸奔后，会从其他富余的主节点分配一个从节点过来，确保每个主节点都有至少一个从节点，不至于因为主节点挂掉而没有相应从节点替换为主节点导致集群崩溃不可用。

  只有当旧主机的其他从机的数量至少为给定数量时，从机才会迁移到孤立主机。这个数字就是“移民壁垒”。迁移屏障为1意味着一个从机只有在其主副本至少有一个其他从机时才会迁移，以此类推。它通常反映了集群中每个主机所需的从机数量。

  默认值为1（仅当从机的主节点至少保留一个从机时，从机才会迁移）。要禁用迁移，只需将其设置为非常大的值。可以设置值0，但仅对调试有用，在生产中很危险。

- 示例：

  ```text
  cluster-migration-barrier 1
  ```


**6、cluster-require-full-coverage**

- 默认：yes

- 取值：yes | no

- 说明：默认情况下，如果Redis集群节点检测到至少有一个散列槽未覆盖（没有可用的节点为其提供服务），那么它们将停止接受查询。这样，如果集群部分关闭（例如，一系列哈希槽不再覆盖），那么所有集群最终都将不可用。一旦所有插槽再次被覆盖，它就会自动返回可用。
  但是，有时您希望正在工作的集群的子集继续接受对仍然被覆盖的密钥空间部分的查询。为此，只需将cluster require full coverage选项设置为no。

- 示例：

  ```text
  cluster-require-full-coverage yes
  ```

**7、 cluster-replica-no-failover**

- 默认：no

- 取值：yes | no

- 说明：此选项设置为“yes”时，可防止从机在主服务器出现故障时尝试故障转移其主服务器。但是，主服务器仍然可以执行手动故障转移。
  这在不同的场景中非常有用，特别是在多个数据中心操作的情况下，如果不是在整个DC故障的情况下，我们希望永远不会单独升级一端。

- 示例：

  ```text
   cluster-replica-no-failover no
  ```

**8、cluster-allow-reads-when-down**

- 默认：no

- 取值：yes | no

- 说明：

  默认为no, 表示当集群因主节点数量达不到最小值或有散列槽没有分配而被标记为失效时, 节点将停止所有的客户端通讯(stop serving all traffic). 这样可以避免潜在从一个不知道集群状态变化的节点读到不一致数据的危险. 设为yes则允许集群失效时仍可以由节点中读取数据. 这样既保证读操作的高可用性, 亦避免不一致写操作(inconsistent writes). 同时, 当Redis Cluster 仅包含1至2个节点, 而某个节点失效后无可用从节点替代, 且因节点数量不足, 无法自动重新分配散列槽, 则该参数设为yes可保证节点仍然可执行读操作.

- 示例：

  ```text
  cluster-allow-reads-when-down no
  ```



### Redis CLUSTER DOCKER/NAT support

在某些部署中，Redis集群节点地址发现失败，原因是地址是NAT的，或者是端口被转发（典型的情况是Docker和其他容器）。
为了使Redis集群在这样的环境中工作，需要一个静态配置，其中每个节点都知道自己的公共地址。以下两个选项用于此范围，分别是：
cluster-announce-ip	集群通知ip
cluster-announce-port	集群通知端口
cluster-announce-bus-port	集群通知总线端口
每个都指示节点关于其地址、客户机端口和集群消息总线端口。然后在总线包的报头中发布信息，以便其他节点能够正确地映射发布信息的节点的地址。
如果不使用上述选项，将使用正常的Redis集群自动检测。
请注意，当重新映射时，总线端口可能不在客户端端口+10000的固定偏移量处，因此您可以根据重新映射的方式指定任何端口和总线端口。如果没有设置总线端口，将像往常一样使用固定偏移量10000。
示例：

```text
cluster-announce-ip 10.1.1.5
cluster-announce-port 6379
cluster-announce-bus-port 6380
```



### Redis SLOW LOG

Redis Slow Log是一个记录超过指定执行时间的查询的系统。执行时间不包括与客户机对话、发送应答等I/O操作，而只是实际执行命令所需的时间（这是命令执行的唯一一个阶段，线程被阻塞，不能同时处理其他请求）

您可以使用两个参数来配置慢日志：一个参数告诉Redis为了记录命令要超过的执行时间（以微秒为单位），另一个参数是慢日志的长度。记录新命令时，最旧的命令将从记录的命令队列中删除。

**1、slowlog-log-slower-than**

- 默认：10000

- 取值：微秒

- 说明：此参数时间以微秒表示，因此1000000相当于1秒。请注意，负数将禁用慢速日志，而值为零将强制记录每个命令。

- 示例：

  ```text
   slowlog-log-slower-than 10000
  ```

**2、slowlog-max-len**

- 默认：128

- 取值：一个正整数

- 说明：慢日志的长度，没有限制，但会消耗内存。可以通过命令`SLOWLOG RESET`回收慢日志占用的内存。

- 示例：

  ```
  slowlog-max-len 128
  ```

### Redis LATENCY MONITOR

Redis 2.8.13引入**延迟监控（Latency Monitoring）**的新特性，帮助用户检查和排除可能的延迟问题。

Redis延迟监控子系统在运行时对不同的操作进行采样，以便收集与Redis实例的可能延迟源相关的数据。
通过LATENCY命令，用户可以打印图形并获取报告。
系统只记录在大于或等于通过延迟监视器阈值配置指令指定的毫秒数的时间内执行的操作。当其值设置为零时，将关闭延迟监视器。
默认情况下，延迟监视是禁用的，因为如果没有延迟问题，则通常不需要延迟监视，并且收集数据会对性能产生影响，虽然影响很小，但可以在大负载下进行测量。如果需要，可以使用命令`CONFIG SET Latency monitor threshold<millises>`在运行时轻松启用延迟监视。

**1、latency-monitor-threshold**

- 默认：0

- 取值：毫秒数

- 说明：延迟监控的阈值，只有超过延迟阈值的操作才会被延迟监控系统记录。其中特殊值0表示关闭延迟监控。

- 示例：

  ```text
  latency-monitor-threshold 0
  ```

### Redis EVENT NOTIFICATION

键空间通知功能自2.8.0版本开始可用。

键空间通知允许客户端订阅发布/订阅频道，以便以某种方式接收影响Redis数据集的事件。

可能接收的事件示例如下：

- 所有影响给定键的命令。
- 所有接收LPUSH操作的键。
- 所有在数据库0中到期的键。

事件使用Redis的普通发布/订阅层传递，因此实现了发布/订阅的客户端无需修改即可使用此功能。

由于Redis的发布/订阅是*fire and forget*，因此如果你的应用要求**可靠的事件通知**，目前还不能使用这个功能，也就是说，如果你的发布/订阅客户端断开连接，并在稍后重连，那么所有在客户端断开期间发送的事件将会丢失。

默认情况下，键空间事件通知是不启用的，因为虽然不太明智，但该功能会消耗一些CPU。可以使用redis.conf中的`notify-keyspace-events`或者使用**CONFIG SET**命令来开启通知。

将参数设置为空字符串会禁用通知。 为了开启通知功能，使用了一个非空字符串，由多个字符组成，每一个字符都有其特殊的含义，具体参见下表：

```text
K     键空间事件，以__keyspace@<db>__前缀发布。
E     键事件事件，以__keyevent@<db>__前缀发布。
g     通用命令（非类型特定），如DEL，EXPIRE，RENAME等等
$     字符串命令
l     列表命令
s     集合命令
h     哈希命令
z     有序集合命令
x     过期事件（每次键到期时生成的事件）
e     被驱逐的事件（当一个键由于达到最大内存而被驱逐时产生的事件）
A     g$lshzxe的别名，因此字符串AKE表示所有的事件。
```

字符串中应当至少存在`K`或者`E`，否则将不会传递事件，不管字符串中其余部分是什么。

例如，要为列表开启键空间事件，则配置参数必须设置为`Kl`，以此类推。

字符串`KEA`可以用于开启所有可能的事件。

配置指令示例：

```text
notify-keyspace-events ""
```

### Redis GOPHER SERVER

Redis包含了RFC1436中指定的Gopher协议的实现.

Gopher是Internet上一个非常有名的信息查找系统，它将Internet上文件组织成某种索引，很方便地将用户从Internet的一处带到另一处。允许用户使用层叠结构的菜单与文件，以发现和检索信息，它拥有世界上最大、最神奇的编目。

Redis Gopher支持使用Redis的内联协议，特别是两种无论如何都是非法的内联请求：空请求或任何以“/”开头的请求（没有以这样的斜杠开头的Redis命令）。正常的RESP2/RESP3请求完全脱离了Gopher协议实现的路径，也照常提供服务。

请注意，当启用“io-threads-do-reads”时，Gopher不可用。

启用命令示例：

```text
 gopher-enabled yes
```

### Redis ADVANCED CONFIG

Redis高级配置部分。

**1、hash-max-ziplist-entries & hash-max-ziplist-value**

- 默认与示例：

  ```
  hash-max-ziplist-entries 512
  hash-max-ziplist-value 64
  ```

- 取值：哈希表内字段数量与单个字段值的大小

- 说明：当散列有少量的条目，并且最大的条目不超过给定的阈值时，使用内存高效的数据结构对散列进行编码，以节约空间提高效率。以上示例中配置表示，当hash内元素数量不超过512个且单个元素的值不大于64个字节时，Redis以其特有的数据结构`ziplist`存储数据以加快查询速度。

**2、list-max-ziplist-size**

- 默认：-2

- 取值：-5 | -4 | -3 | -2 | -1 | 正数

- 说明：列表也以一种特殊的方式编码以节省大量空间。每个内部列表节点允许的条目数可以指定为固定的最大大小或最大元素数。

  对于固定的最大大小，请使用-5到-1，意思是：

  -  -5: max size: 64 Kb  <-- 不建议用于正常工作负载
  - -4: max size: 32 Kb  <-- 不建议的
  - -3: max size: 16 Kb  <-- 不太建议
  - -2: max size: 8 Kb   <-- 建议
  - -1: max size: 4 Kb   <-- 建议

  正数表示每个列表节点最多可存储的元素数。

- 示例：

  ```text
  list-max-ziplist-size -2
  ```

**3、list-compress-depth**

- 默认：0

- 取值：非负整数

- 说明：列表也可以压缩。
  Compress depth是从列表的每侧排除压缩的节点数。列表的头和尾总是未压缩的，以便进行快速推/弹出操作。其中数字的代表意义为：

  - 0： 特殊值，禁用所有列表压缩
  - 1： depth 1表示头部和尾部一个元素不压缩，如：[head]->node->node->...->node->[tail]，其中 [head], [tail] 将不会被压缩，压缩所有之间的节点。
  - 2：depth2 表示头部和尾部的两个元素不压缩。如：[head]->[next]->node->node->...->node->[prev]->[tail]，这里的意思是：不要压缩head或head->next或tail->prev或tail，而是压缩它们之间的所有节点。
  - 3 ...依次类推

- 示例：

  ```text
  list-compress-depth 0
  ```

**4、set-max-intset-entries**

- 默认：512

- 取值：正整数 元素个数

- 说明：集合只有一种特殊的编码方式：当一个集合由恰好是基数为10的64位有符号整数范围内的整数组成时。为了使用这种特殊的内存节省编码，此处的配置设置设置了集合大小的限制。

- 示例：

  ```text
  set-max-intset-entries 512
  ```

**5、zset-max-ziplist-entries  & zset-max-ziplist-value** 

- 默认及示例：

  ```text
  zset-max-ziplist-entries 128
  zset-max-ziplist-value 64
  ```

- 取值：有序集合中元素个数以及单个元素最大大小

- 说明：与散列和列表类似，排序集也经过特殊编码以节省大量空间。此编码仅在排序集的长度和元素低于指定限制时使用。示例表示有续集元素数量小于128且单个元素小于64字节时，使用特殊编码以节约空间。

**6、hll-sparse-max-byte**

- 默认： 3000

- 取值：0-16000

- 说明：HyperLogLog稀疏表示字节数限制。该限制包括16字节头。当使用稀疏表示的HyperLogLog超过此限制时，它将转换为密集表示。
  大于16000的值是完全无用的，因为在这一点上，密集表示更高效。
  建议值为~3000，以便在不减慢过多PFADD（稀疏编码为O（N））的情况下利用空间高效编码的优点。当CPU不是问题，但空间是问题，并且数据集由基数在0-15000范围内的许多HyperLogLog组成时，该值可以提高到~10000。

- 示例：

  ```text
  hll-sparse-max-bytes 3000
  ```

**7、stream-node-max-bytes  & stream-node-max-entries** 

- 默认及示例：

  ```text
  stream-node-max-bytes 4096
  stream-node-max-entries 100
  ```

- 取值：前者为流节点大小，后者为最大元素数量

- 说明：Streams宏节点最大大小/项。流数据结构是一个由大节点组成的基数树，其中对多个项进行编码。使用此配置，可以配置单个节点的大小（以字节为单位），以及在附加新的流条目时切换到新节点之前可能包含的最大项数。如果将以下任何设置设置为零，则会忽略该限制，因此，例如，可以通过将max bytes设置为0，将max entries设置为所需的值来设置max entires限制。

**8、activerehashing**

-  默认：yes

- 取值：yes | no

- 说明：是否重置Hash表,设置成yes后redis将每100毫秒使用1毫秒CPU时间来对redis的hash表重新hash，可降低内存的使用,当使用场景有较为严格的实时性需求,不能接受Redis时不时的对请求有2毫秒的延迟的话，把这项配置为no,如果没有这么严格的实时性要求,可以设置为 yes,以便能够尽可能快的释放内存

- 示例：

  ```text
  activerehashing yes
  ```

**9、client-output-buffer-limit**

- 默认及示例：

  ```text
  client-output-buffer-limit normal 0 0 0
  client-output-buffer-limit replica 256mb 64mb 60
  client-output-buffer-limit pubsub 32mb 8mb 60
  ```

- 配置语法：`client-output-buffer-limit <class> <hard limit> <soft limit> <soft seconds>`

- 说明：客户机输出缓冲区限制可用于强制断开由于某种原因从服务器读取数据速度不够快的客户机的连接（一个常见的原因是Pub/Sub客户机不能像发布服务器生成消息那样快地使用消息）。
  对于三种不同类型的客户端，可以设置不同的限制，对应配置的第一个字段：

  - normal ->普通客户端，包括监视客户端

  - replica ->从机客户端

  - pubsub->订阅了至少一个pubsub频道或模式的客户端

  后面的三个参数的含义时，硬限制，软限制以及软限制的持续时间。具体含义为：一旦达到硬限制，或者如果达到软限制并保持达到指定秒数（连续），对应客户端将立即断开连接。例如：`client-output-buffer-limit normal 32mb 16mb 60`的含义为，如果输出缓冲区的大小达到32mb，normal类型的客户端，会断开连接；或者输出缓冲区的大小达到16mb，而且持续时间超过60秒，则断开连接。

  默认情况下，普通客户机不受限制，因为它们不会在没有请求（以推送方式）的情况下接收数据，而是在请求之后接收数据，因此只有异步客户机可能会创建这样一种场景，即请求数据的速度比读取数据的速度快。
  相反，pubsub和从机客户端有一个默认限制，因为订阅者和副本以推送方式接收数据。
  硬限制或软限制都可以通过将其设置为零来禁用。

  

**10、client-query-buffer-limit**

- 默认：未配置

- 取值：缓存大小

- 说明：客户端查询缓冲区累积新指令。默认情况下，它们被限制为固定数量，以避免协议去同步（例如，由于客户端中的错误）将查询缓冲区中未绑定的内存使用。但是，如果您有非常特殊的需求，比如我们的multi/exec请求或类似请求，您可以在这里配置它。

- 示例：

  ```text
  client-query-buffer-limit 1gb
  ```

**11、proto-max-bulk-len**

- 默认：512mb

- 取值：请求大小

- 说明：在Redis协议中，批量请求（即表示单个字符串的元素）通常限制为512MB。不过，您可以在此处更改此限制，但必须为1mb或更大。

- 示例：

  ```text
  proto-max-bulk-len 512mb
  ```

**12、hz**

- 默认：10

- 取值：1-500

- 说明：Redis执行内部任务的频率，值越大，单位时间执行的次数越多。Redis调用一个内部函数来执行许多后台任务，比如在超时时关闭客户端的连接，清除从未请求的过期密钥，等等。并非所有任务都以相同的频率执行，但是Redis会根据指定的“hz”值检查要执行的任务。
  默认情况下，“hz”设置为10。当Redis空闲时，提高该值将使用更多的CPU，但同时当有许多键同时过期时，将使Redis更具响应性，并且可以更精确地处理超时。
  范围在1到500之间，但是值超过100通常不是一个好主意。大多数用户应该使用默认值10，并且仅在需要非常低延迟的环境中才将其提高到100。

- 示例：

  ```text
  hz 10
  ```

**13、dynamic-hz**

- 默认：yes

- 取值：yes | no

- 说明：通常，HZ值与连接的客户机数量成比例是有用的。例如，为了避免每次后台任务调用都要处理太多的客户机以避免延迟峰值，这一点非常有用。由于默认HZ值被保守地设置为10，Redis提供并启用了使用自适应HZ值的功能，当有许多连接的客户端时，该功能将临时提高。启用动态HZ时，实际配置的HZ将用作基线，但一旦连接更多客户端，实际将根据需要使用配置的HZ值的倍数。这样，空闲的实例将使用很少的CPU时间，而繁忙的实例将更具响应性。

- 示例：

  ```text
  dynamic-hz yes
  ```

**14、aof-rewrite-incremental-fsync**

- 默认：yes

- 取值：yes | no

- 说明：当子进程进行重写AOF文件时，如果启用此选项，则该文件将每生成32mb的数据进行一次文件同步。这对于以渐进的方式将文件提交到磁盘和避免较大的延迟峰值非常有用。

- 示例：

  ```text
  aof-rewrite-incremental-fsync yes
  ```

**15、rdb-save-incremental-fsync**

-  默认：yes

- 取值：yes | no

- 说明：当redis保存RDB文件时，如果启用了此选项，则文件将每生成32mb的数据进行一次文件同步。这对于以渐进的方式将文件提交到磁盘和避免较大的延迟峰值非常有用。

- 示例：

  ```text
  rdb-save-incremental-fsync yes
  ```

**16、 lfu-log-factor & lfu-decay-time**

- 默认及示例：

  ```text
  lfu-log-factor 10
  lfu-decay-time 1
  ```

- 取值：都为一个整数，其中后者单位为分钟

- 说明：

  Redis LFU清理算法（参见maxmemory设置）可以进行调优。但是，最好以默认设置开始，在研究如何提高性能以及键的LFU算法如何随时间变化后才建议进行更改，这可以通过OBJECT FREQ命令进行检查。
  Redis LFU实现中有两个可调参数：计数器对数因子(`lfu-log-factor`)和计数器衰减时间(`lfu-decay-time`)。在改变这两个参数之前，了解这两个参数的含义是很重要的。
  LFU计数器每个键只有8位，它的最大值是255，所以Redis使用了基于概率的对数计数器。给定旧计数器的值，当访问键时，计数器按以下方式递增：

  - 提取0到1之间的随机数R。

  - 概率P的计算公式为1/(old_value*`lfu_log_factor`+1)。
  - 只有当R<P时，计数器才递增。

  默认`lfu log factor`为10，值越大，频率增长越慢。这是一个频率计数器如何随不同对数因子的不同访问次数而变化的表：

  | factor | 100 hits | 1000 hits | 100k hits | 1M hits | 10M hits |
  | ------ | -------- | --------- | --------- | ------- | -------- |
  | 0      | 104      | 255       | 255       | 255     | 255      |
  | 1      | 18       | 49        | 255       | 255     | 255      |
  | 10     | 10       | 18        | 142       | 255     | 255      |
  | 100    | 8        | 11        | 49        | 143     | 255      |

  注：上表是通过运行以下命令获得的：

  ```text
  redis-benchmark -n 1000000 incr foo
  redis-cli object freq foo
  ```

  注2：计数器初始值为5，以便给新对象一个累积命中的机会。

  计数器衰减时间是键计数器除以2所必须经过的时间，单位为分钟（如果值小于等于10，则递减）衰减时间用于配置热点键访问频率衰减的速度，值越大，衰减越慢，热点数据保存的时间越长。
  `lfu-decay-time`的默认值为1。一个特殊的值0意味着每次扫描计数器时它都会衰减。



### Redis ACTIVE DEFRAGMENTATION

什么是活动碎片整理？

活动（在线）碎片整理允许Redis服务器压缩内存中数据碎片和释放之间的空间，从而允许回收内存。
碎片化是每个分配器（但幸运的是，Jemalloc的情况较好）和某些工作负载都会发生的自然过程。通常需要重新启动服务器以降低碎片，或者至少清除所有数据并重新创建。不过，由于Oran Agra for Redis4.0实现了这个特性，这个过程可以在服务器运行时以“热”的方式在运行时发生。

基本上，当碎片超过某个级别（见下面的配置选项）时，Redis将开始利用某些特定的Jemalloc特性在连续内存区域中创建值的新副本（以便了解分配是否导致碎片并将其分配到更好的位置），同时，将释放数据的旧拷贝。这个过程，对所有键进行增量重复，将使得碎片降回正常值。

需要了解的重要事项：

- 此功能在默认情况下是禁用的，并且仅当您编译Redis以使用我们随Redis源代码提供的Jemalloc副本时才起作用。这是Linux版本的默认设置。
- 如果没有碎片问题，就不需要启用此功能。
- 一旦遇到碎片，可以在需要时使用命令“CONFIG SET activedefrag yes”启用此功能。

配置参数能够微调碎片整理过程的行为。如果您不确定它们的含义，那么最好保持默认值不变。

**1、activedefrag**

- 默认：未配置

- 取值：yes | no

- 说明：是否开启碎片整理

- 示例：

  ```
  activedefrag no
  ```

**2、 active-defrag-ignore-bytes**

- 默认：未配置

- 取值：碎片浪费的内存大小

- 说明：启动活动碎片整理的最小碎片大小

- 示例：

  ```text
   active-defrag-ignore-bytes 100mb
  ```

**3、active-defrag-threshold-lower**

- 默认：未配置

- 取值：一个整数表示百分比

- 说明：启动活动碎片整理的最小碎片百分比

- 示例：

  ```text
  active-defrag-threshold-lower
  ```

**4、active-defrag-threshold-upper**

- 默认：未配置

- 取值：一个整数表示百分比

- 说明：内存碎片超过 100%，则尽最大努力整理

- 示例：

  ```text
  active-defrag-threshold-upper 100
  ```

**5、active-defrag-cycle-min**

- 默认：未配置

- 取值：百分比

- 说明：占用的CPU资源百分比，在达到较低阈值时使用

- 示例：

  ```text
  active-defrag-cycle-min 1
  ```

**6、active-defrag-cycle-max**

- 默认：未配置

- 取值：百分比

- 说明：占用的CPU资源百分比，在达到最高阈值时使用

- 示例：

  ```
  active-defrag-cycle-max 25
  ```

**7、active-defrag-max-scan-fields**

- 默认：未配置

- 取值：一个正整数

- 说明：碎片整理 扫描set/hash/zset/list时，仅当 set/hash/zset/list 的长度小于此阀值时，才会将此key加入碎片整理

- 示例

  ```text
  active-defrag-max-scan-fields 1000
  ```

**8、jemalloc-bg-thread**

- 默认：yes

- 取值：yes | no

- 说明：默认情况下，将启用用于清除的Jemalloc后台线程

- 示例：

  ```text
  jemalloc-bg-thread yestexy
  ```

**9、固定redis线程或进程所用的CPU核心**

- 默认：未配置

- 说明：可以将Redis的不同线程和进程固定到系统中的特定cpu上，以最大限度地提高服务器的性能。将不同的Redis线程固定在不同的cpu中非常有用，同时也可以确保在同一主机上运行的多个Redis实例将被固定到不同的cpu上。

  通常，您可以使用“taskset”命令来执行此操作，但是也可以通过Redis配置直接执行此操作，无论是在Linux还是FreeBSD中。
  您可以固定服务器/IO线程、bio线程、aof重写子进程和bgsave子进程。指定cpu列表的语法与taskset命令相同：

  将redis服务器/io线程与cpu核心0,2,4,6 绑定：

  ```text
  server_cpulist 0-7:2
  ```

  将bio线程与cpu核心1,3 绑定：

  ```text
  bio_cpulist 1,3
  ```

  将aof rewrite子进程与cpu核心8、9、10、11 绑定：

  ```text
  aof_rewrite_cpulist 8-11
  ```

  将bgsave子进程与cpu 核心 1,10,11 绑定：

  ```text
  bgsave_cpulist 1,10-11
  ```

## 官方完整配置

[=========================传送门=========================](https://raw.githubusercontent.com/redis/redis/6.0/redis.conf) 

