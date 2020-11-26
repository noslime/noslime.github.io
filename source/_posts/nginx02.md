---
title: nginx探索之旅二
date: 2020-11-24 21:23:22
author: noslime
categories: nginx
tags: nginx
keywords: nginx
---

# 配置文件介绍

nginx的配置文件大致可以分为三个部分：全局块、EVENTS块、HTTP块，示意图如下

<img src="https://cdn.jsdelivr.net/gh/noslime/noslime.github.io@master/source/images/nginx.conf.png" alt="n" style="zoom:80%;" />

### 全局块

从配置文件开始到events块之间的内容，主要会设置一些影响nginx服务器整体运行的配置指令，一般有运行nginx服务器的用户组，nginx进程pid存放路径，错误日志存放路径，允许生成worker process数等。

```config
#配置worker子进程用户或者组，默认nobody，安全问题，建议用nobody
#user  nobody;

#worker数和服务器的cpu数相等是最为适宜
worker_processes  2;

#配置Nginx worker进程最大打开文件数
worker_rlimit_nofile 65535;

#work绑定cpu(4 work绑定4cpu)
worker_cpu_affinity 0001 0010 0100 1000

#work绑定cpu (4 work绑定8cpu中的4个) 。
worker_cpu_affinity 0000001 00000010 00000100 00001000  

#error_log path(存放路径) level(日志等级)path表示日志路径，level表示日志等级，
#具体如下：[ debug | info | notice | warn | error | crit ]
# 从左至右，日志详细程度逐级递减，即debug最详细，crit最少，默认为crit。 

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;
#进程运行文件所在位置
#pid        logs/nginx.pid;
```

### EVENTS块

此部分涉及的主要是nginx工作模式，包括每个进程的最大连接数，选取哪种事件驱动模型处理连接请求，是否允许同时接受多个网路连接，开启网络连接序列化等。

```config
events {
    #这个值是表示每个worker进程所能建立连接的最大值，所以，一个nginx能建立的最大连接数，应该是				     #Max_client=worker_connections * worker_processes。
    #当然，这里说的是最大连接数，对于HTTP请求本地资源来说，能够支持的最大并发数量是worker_connections *         #worker_processes，
    #如果是支持http1.1的浏览器每次访问要占两个连接，
    #所以普通的静态访问最大并发数是： worker_connections * worker_processes /2，
    #而如果是HTTP作为反向代理来说，最大并发数量应该是worker_connections * worker_processes/4。
    #因为作为反向代理服务器，每个并发会建立与客户端的连接和与后端服务的连接，会占用两个连接。

    worker_connections  1024;  

    #Nginx支持的工作模式有select、poll、kqueue、epoll、rtsig和/dev/poll。     
    #其中select和poll都是标准的工作模式，kqueue和epoll是高效的工作模式
    #这个值是表示nginx要支持哪种多路io复用。
    #一般的Linux选择epoll, 如果是(*BSD)系列的Linux使用kquene。
    #windows版本的nginx不支持多路IO复用，这个值不用配。
    use epoll;

    #当一个worker抢占到一个链接时，是否尽可能的让其获得更多的连接,默认是off 。
    multi_accept on;

    #设置网路连接序列化，防止惊群现象发生，默认为on，这是一种保守的设置
    #但是如果你的网站访问量比较大，为了系统的吞吐量，我还是建议大家关闭它。
    accept_mutex  on;
}
```

### HTTP 块

这算是 Nginx 服务器配置中最频繁的部分，代理、缓存和日志定义等绝大多数功能和第三方模块的配置都在这里。 需要注意的是：http 块也可以包括 http全局块、[upstream] 、server 块。

#### HTTP全局块

http全局块配置的指令包括文件引入、MIME-TYPE 定义、日志自定义、连接超时时间、单链接请求数上限等。

以下给出一些参数设置，其中一些设置属于其他模块，但经常用到，文中为了方便描述，全写在了一起。其中很多参数可以设置在http、server、location处，实际应根据具体业务设置位置。 一般来说，设置的位置越在内层，优先级越高。 详见nginx [官网文档]

[官方文档]: http://nginx.org/en/docs/http/ngx_http_core_module.html



```config
http {
    #当web服务器收到静态的资源文件请求时，依据请求文件的后缀名在服务器的MIME配置文件中
    #找到对应的MIME Type，再根据MIME Type设置HTTP Response的Content-Type，然后浏览器
    #根据Content-Type的值处理文件。
    include       mime.types;
    #如果 不能从mime.types找到映射的话，用以下作为默认值
    default_type  application/octet-stream;
    
    #保存服务器名字的hash表是由指令server_names_hash_max_size 和server_names_hash_bucket_size所控制的。参数hash bucket size总是等于hash表的大小，并且是一路处理器缓存大小的倍数。在减少了在内存中的存取次数后，使在处理器中加速查找hash表键值成为可能。如果hash bucket size等于一路处理器缓存的大小，那么在查找键的时候，最坏的情况下在内存中查找的次数为2。第一次是确定存储单元的地址，第二次是在存储单元中查找键 值。因此，如果Nginx给出需要增大hash max size 或 hash bucket size的提示，那么首要的是增大前一个参数的大小.
    server_names_hash_bucket_size 128;
    
    ###################################HTTP请求####################################
    #一个请求完成之后还要保持连接多久
    keepalive_timeout  5;
    
    #设定通过nginx上传文件的大小
	client_max_body_size 300m;
	#请求体缓存大小
	client_body_buffer_size 64k;
	#临时文件位置
	#小于client_body_buffer_size直接在内存中高效存储。如果大于client_body_buffer_size小于client_max_body_size会存储临时文件，临时文件位于client_body_temp下，一定要有权限。
	client_body_temp_path  /spool/nginx/client_body_temp 3 2;
	
	#客户端请求头部的缓冲区大小。这个可以根据你的系统分页大小来设置，一般一个请求的头部大小不会超过1k，不过由于一般系统分页都要大于1k，所以这里设置为分页大小。分页大小可以用命令getconf PAGESIZE取得。
    client_header_buffer_size 4k;
    #读取大型客户端请求头的缓冲区的最大数量和大小。nginx默认会用client_header_buffer_size这个buffer来读取header值，如果header过大，它会使用large_client_header_buffers来读取
	large_client_header_buffers 8 128k;
	#########################################################################
	
     ###################################日志####################################
     #访问日志位置  server、location中若有日志配置，优先级更高
     access_log  logs/host.access.log  main;
     access_log  logs/host.access.404.log  log404;
     #日志格式化
     log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
     '$status $body_bytes_sent "$http_referer" '
     '"$http_user_agent" "$http_x_forwarded_for"';
     #########################################################################

    #################################高效文件传输######################################
    #此配置项用来在两个文件描述符之间直接传递数据(完全在内核中操作)，从而避免了数据在内核     
    #缓冲区和用户缓冲区之间的拷贝，操作效率很高，被称之为零拷贝，适用于有大文件上传下载的情况     
    #如果用来进行下载等应用磁盘IO重负载应用，可设置为off，以平衡磁盘与网络IO处理速度，降低系统负载
    #可以在http块，server块，location块。
    sendfile  on;
    
	#此选项允许或禁止使用sock的TCP_CORK的选项，此选项仅在使用sendfile的时候使用
	#用户层可通过setsockopt系统调用设置TCP套接口的TCP_CORK选项。开启时，内核将阻塞不完整的报文，当关闭此选项时，发送阻塞的报文。此处的不完整指的是应用层发送的数据长度不足一个MSS长度。使用场景是在调用sendfile发送文件内容之前，提前发送一个描述文件信息的头部数据段，并且阻塞住此头部数据，与之后的sendfile数据一同发送。或者用于优化吞吐性能。但是，TCP_CORK最多只能将数据阻塞200毫秒，如果超过此时间值，内核将自动发送阻塞的数据
   	tcp_nopush on;
   	
   	#tcp_nodelay off 会增加通信的延时，但是会提高带宽利用率。适用于在高延时、数据量大的通信场景
    #tcp_nodelay on 会增加小包的数量，但是可以提高响应速度。适用于及时性高的通信场景
    #tcp_nodelay与tcp_nopush是互斥的
	tcp_nodelay off;
	#########################################################################
 
    #################################gzip###################################
    #开启或者关闭gzip模块
    #gzip  on ;
    #设置允许压缩的页面最小字节数，页面字节数从header头中的Content-Length中进行获取。
    #gzip_min_lenth 1k;
    # gzip压缩比，1 压缩比最小处理速度最快，9 压缩比最大但处理最慢（传输快但比较消耗cpu）
    #gzip_comp_level 4;
	
    #匹配MIME类型进行压缩，（无论是否指定）"text/html"类型总是会被压缩的。
    #gzip_types types text/plain text/css application/json  application/x-javascript text/xml  
    #########################################################################
    
    ##############################文件缓存###########################################
    #动静分离  指定缓存启用
    #服务器端静态资源缓存，最大缓存到内存中的文件数，非活动的（没要求到的文件）会在20秒后从缓存中释放
    open_file_cache max=102400 inactive=20s;   
    #活跃期限内最少使用的次数，否则视为不活跃。
    open_file_cache_min_uses 2;
    #验证缓存是否活跃的时间间隔
    open_file_cache_valid 30s;
    #文件错误是否缓存
    open_file_cache_errors on
    #########################################################################
    
	#################################代理服务器###################################
	#后端服务器连接的超时时间_发起握手等候响应超时时间
	proxy_connect_timeout 90;
    #连接成功后_等候后端服务器响应时间_其实已经进入后端的排队之中等候处理（也可以说是后端服务器处理请求的时间）
	proxy_read_timeout 180;
	#后端服务器数据回传时间_就是在规定时间之内后端服务器必须传完所有的数据
	proxy_send_timeout 180;
	
	#设置从被代理服务器读取的第一部分应答的缓冲区大小，通常情况下这部分应答中包含一个小的应答头，默认情况下这个值的大小为指令proxy_buffers中指定的一个缓冲区的大小，不过可以将其设置为更小
	proxy_buffer_size 256k;
	
	#设置用于读取应答（来自被代理服务器）的缓冲区数目和大小，默认情况也为分页大小，
根据操作系统的不同可能是4k或者8k
	proxy_buffers 4 256k;
	
	#proxy_busy_buffers_size不是独立的空间，他是proxy_buffers和proxy_buffer_size的一部分。
nginx会在没有完全读完后端响应就开始向客户端传送数据，所以它会划出一部分busy状态的buffer来专门向客户端传送数据(建议为proxy_buffers中单个缓冲区的2倍)，然后它继续从后端取数据。proxy_busy_buffer_size参数用来设置处于busy状态的buffer有多大。
1）如果完整数据大小小于busy_buffer大小，当数据传输完成后，马上传给客户端；
2）如果完整数据大小不小于busy_buffer大小，则装满busy_buffer后，马上传给客户端；
	proxy_busy_buffers_size 256k;
 
	#设置在写入proxy_temp_path时数据的大小，预防一个工作进程在传递文件时阻塞太长
	proxy_temp_file_write_size 256k;

	#设置内存缓存空间大小为200MB，1天没有被访问的内容自动清除，硬盘缓存空间大小为30GB。
	proxy_cache_path /data/proxy_cache_dir levels=1:2 keys_zone=cache_one:200m inactive=1d max_size=30g;
	#表示使nginx阻止HTTP应答代码为300或者更高的应答
	proxy_intercept_errors on;
	
    #######################################################################################    
    upstream streamname{...}
    server {...}
}
```



1. 访问日志格式化参数说明

   注：log_format可设为json格式以方便后续分析处理

> ​    #$http_x_forwarded_for 用以记录客户端的ip地址；
>
> ​	#$remote_addr  上一级代理的IP地址；
>
> ​	#$remote_user：用来记录客户端用户名称；
> ​	#$time_local： 用来记录访问时间与时区；
> ​	#$request： 用来记录请求的url与http协议；
> ​	#$status： 用来记录请求状态；成功是200，
> ​	#$body_bytes_sent ：记录发送给客户端文件主体内容大小；
> ​	#$http_referer：用来记录从那个页面链接访问过来的；
> ​	#$http_user_agent：记录客户浏览器的相关信息；

2. 临时文件存储位置 	

> Syntax: client_body_temp_path path [level1 [level2 [level3]]]; 
>
> Default:    client_body_temp_path client_body_temp; 
>
> Context:    http, server, location
>
> client_body_temp_path  /spool/nginx/client_body_temp 3 2 1;     //3 2 1代表目录名称长度
>
> 以上设置可能产生缓存文件格式：/spool/nginx/client_temp/456/78/9/20201127
>
> chown -R username:usergroup /spool/nginx/client_body_temp  //为防止权限问题，可为文件设置所有者

3. 文件引入

> ​    若server较多，为保持配置文件结构清晰，可以每个虚拟主机另起一个配置文件，如：
> ​    include /etc/nginx/conf.d/*.conf;  //配置在http全局块

#### Upstream

upstream是nginx的HTTP Upstream模块，此模块不是必须的，这个模块通过一个指定的调度算法来实现客户端IP到后端服务器的负载均衡。

upstream中服务器的常见参数

> weight`=`*number*   设置服务器的权重，默认为1
>
> max_conns=number  限制到代理服务器的最大同时连接数，默认为0
>
> fail_timeout=time     服务器不可用的时间 默认为10秒
>
> max_fails=number   设置在fail_timeout参数设置的持续时间内与服务器通信的失败尝试次数
>
> backup	备用服务器，主服务器不可用时启用，不能和hash和random负载平衡方法一起使用
>
> down  标记服务器永久不可用

```config
http {
	...
    # 1、轮询（默认）
    # 每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器down掉，能自动剔除。
     upstream myapp1 {
        server srv1.example.com;
        server srv2.example.com;
        server srv3.example.com;
    }
    # 2、最小连接负载均衡
    # nginx将尝试不使用过多的请求使繁忙的应用服务器过载，而是将新请求分发到不太繁忙的服务器
    upstream myapp1 {
        least_conn;
        server srv1.example.com;
        server srv2.example.com;
        server srv3.example.com;
    }
    # 3、指定权重
    # 指定轮询几率，weight和访问比率成正比，用于后端服务器性能不均的情况。
    upstream myapp1 {
        server srv1.example.com weight=3;
        server srv2.example.com weight=2;
        server srv3.example.com weight=2;
    }
    #4、IP绑定 ip_hash
    # 每个请求按访问ip的hash结果分配，这样每个访客固定访问一个后端服务器，可以解决session的问题。
    upstream myapp1 {
        ip_hash;
        server srv1.example.com;
        server srv2.example.com;
        server srv3.example.com;
	}
    #5、备机方式 backup
    # 正常情况不访问设定为backup的备机，只有当所有非备机全都宕机的情况下，服务才会进备机。
    # backup 不能和ip_hash一起使用
    upstream myapp1 {
        server srv1.example.com;
        server srv2.example.com;
        server srv3.example.com backup;
	}
    #6、fair（第三方）
    #按后端服务器的响应时间来分配请求，响应时间短的优先分配。 需要安装fair模块
    upstream myapp1 {
        fair;
        server srv1.example.com;
        server srv2.example.com;
        server srv3.example.com;
	}
    #7、url_hash（第三方） 需要安装url_hash模块
    #按访问url的hash结果来分配请求，使每个url定向到同一个后端服务器，后端服务器为缓存时比较有效。
    upstream myapp1 {
        server srv1.example.com;
        server srv2.example.com;
        server srv3.example.com;
        hash $request_uri
	}
      server {
    	listen 80;
    	location / {
        	proxy_pass http://myapp1;
      }
}
```

#### Server

server块：配置虚拟主机的相关参数，一个http中可以有多个server。而server块又包含多个**location块**

location块：配置请求的路由，以及各种页面的处理情况。

```config
http {
	...
    upstream myserver{}
    server {
        #监听端口号
        listen       80;
        #服务名
        server_name  192.168.161.130;
        #字符集
        #charset utf-8;
	#location [=|~|~*|^~] /uri/ { … }   
	# = 精确匹配
	# ~ 正则匹配，区分大小写
	# ~* 正则匹配，不区分大小写
	# ^~  关闭正则匹配
	
	#匹配原则：
	# 1、所有匹配分两个阶段，第一个叫普通匹配，第二个叫正则匹配。
	# 2、普通匹配，首先通过“=”来匹配完全精确的location
        # 2.1、 如果没有精确匹配到， 那么按照最大前缀匹配的原则，来匹配location
        # 2.2、 如果匹配到的location有^~,则以此location为匹配最终结果，如果没有那么会把匹配的结果暂存，继续进行正则匹配。
        # 3、正则匹配，依次从上到下匹配前缀是~或~*的location, 一旦匹配成功一次，则立刻以此location为准，不再向下继续进行正则匹配。
        # 4、如果正则匹配都不成功，则继续使用之前暂存的普通匹配成功的location.

	    # 匹配任何查询，因为所有请求都以 / 开头。但是正则表达式规则和长的块规则将被优先和查询匹配。
        location / {   
	    #定义服务器的默认网站根目录位置
        root   html;
            
	    #默认访问首页索引文件的名称
	    index  index.html index.htm;

	    #反向代理路径
         proxy_pass http://myapp1;

	    #反向代理的超时时间
         proxy_connect_timeout 10;
         proxy_redirect default;
         }

         location  /images/ {    
	    	root images ;
	    	deny 127.0.0.1;  #拒绝的ip
         	 allow 172.18.5.54; #允许的ip
	     }
         # 匹配任何以/images/jpg/ 开头的任何查询并且停止搜索。任何正则表达式将不会被测试。 
         location ^~ /images/jpg/ {  
            root images/jpg/ ;
         }
         location ~*.(gif|jpg|jpeg)$ { 
	      #所有静态文件直接读取硬盘
              root pic ;
	      
	      #expires定义用户浏览器缓存的时间为3天，如果静态页面不常更新，可以设置更长，
	      #这样可以节省带宽和缓解服务器的压力
              expires 3d; #缓存3天
         }
        #error_page  404              /404.html;
        # redirect server error pages to the static page /50x.html
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
 
}
```

