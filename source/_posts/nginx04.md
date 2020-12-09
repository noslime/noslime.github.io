---
title: nginx反向代理与负载均衡
date: 2020-11-27 23:17:22
author: noslime
categories: 后端
tags: nginx
keywords: nginxn
---

nginx的反向代理与负载均衡是我们日常工作中经常用到的功能，并且给我们带来了较大的便利。



## 反向代理

### 概念

> ​		反向代理在计算机网络中是代理服务器的一种。服务器根据客户端的请求，从其关系的一组或多组后端服务器上获取资源，然后再将这些资源返回给客户端，客户端只会得知反向代理的IP地址，而不知道在代理服务器后面的服务器集群的存在。
>

### 作用

**1)提高了内部服务器的安全**

> ​		外部网络用户通过反向代理访问内部服务器，只能看到反向代理服务器的IP地址和端口号，内部服务器对于外部网络是完全不可见的。 此外，没有信息资源被保存在反向代理服务器上。 所有网页程序都存储在内部服务器上。 对反向代理服务器的攻击不会破坏真实的网页信息系统，从而提高了内部服务器的安全性。
>

**2)加快了对内部服务器的访问速度**

> ​		反向代理服务器的缓存功能还可以加快用户的访问速度。
>

**3)节约了有限的IP资源**

> ​		 公共网络分配的IP地址数量是有限的。 如果为每个服务器分配了一个公共网络地址，则不可能，反向代理技术可以解决IP地址不足的问题。
>

**4)应用场景**

> - 堡垒机，保护后端服务器的安全
> - 将多个服务器通过虚拟主机的方式发布到公网
> - 缓存服务器，CDN加速

### NGINX反向代理

#### 基本配置

> ```
> server {
> 	listen 80;
> 	server_name localhost;
> 	lacation / {
>     	index index.html index.htm;
>     	proxy_pass  http://target_server:port;  //代理的目标服务器
> 	}
> }
> ```
>

#### 可配参数

> ```
> server {
> listen 80;
> server_name localhost;
> 	location / {
>            index index.html index.htm;
>            proxy_pass  http://target_server:port;  //代理的目标服务器
>            
>            proxy_set_header Host      $host;       //设置请求主机名
>            proxy_set_header X-Real-IP $remote_addr;//设置上游访问IP
>            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for //客户端访问IP
>    
>            proxy_buffering  on       //默认为on，开启缓冲后端服务器响应
>            proxy_buffer_size 16k;   //代理服务器保存响应头信息的缓冲区大小，默认情况也为分页大小
>            proxy_buffers 4 64k;	 //用于读响应体的缓冲区数目和大小，默认情况也为分页大小, 网页平均64k以下
>            proxy_busy_buffers_size 128k;  //用来设置处于busy状态的buffer有多大，默认为proxy_buffers的两倍
>    
>            proxy_connect_timeout 60;   //和后端服务器连接超时时间
>            proxy_send_timeout 90;   //向后端服务器发送一次数据包的超时时间
>            proxy_read_timeout 90;   //定义从后端服务器读取（接收）数据的超时时间
>    
>            client_max_body_size    10m;    //单次通过nginx上传文件的大小
>            client_body_buffer_size 128k;	//用户端请求缓冲区的最大字节数
>                                                                      
>            proxy_max_temp_file  1024m      //开启响应缓冲后，临时文件的最大大小, 默认1G
>            proxy_temp_file_write_size 128k //开启响应缓冲后，每次写数据到临时文件的大小
>            proxy_temp_path /spool/nginx/proxy_temp 1 2;　　 //缓冲文件位置及命名规则
> 	}
> }
> ```
>

​		**proxy_busy_buffers_size**

> 配置项`proxy_busy_buffers_size` 用来设置处于busy状态的buffer有多大，默认为proxy_buffers的两倍。这个参数的作用是限制处于busy（开启响应缓存的情况下，读满数据的缓存响应客户端的过程不能被打断）状态的buffer大小，下述两条配置  {`proxy_buffers 4 64k;`	`proxy_busy_buffers_size 128k;`}  便是当用掉两个缓存区（2*64k=128k）的大小后，后面的缓存即使读到数据但不响应给客户端，若读满剩余缓存还不能读完后端服务器响应，则会写到临时文件中。

其他指令：

​       **proxy_redirect**

> *Context: http,server, location*
>
> 设置后端服务器“Location”响应头和“Refresh”响应头的替换文本。 假设后端服务器返回的响应头是 “Location: http://localhost:8000/two/some/uri/”，那么指令
>
> ```
> proxy_redirect http://localhost:8000/two/ http://frontend/one/;
> ```
>
> 会被重写为  “`Location: http://frontend/one/some/uri/`”.
>
> 以下指令`replacement`字符串将省略服务器名
>
> ```
> proxy_redirect http://localhost:8000/two/ /;
> ```
>
> 此时将使用代理服务器的主域名和端口号来替换。如果端口是80，可以不加。
>
> 默认的参数`default`替换使用了location和proxy_pass指令的参数，使用以下两种配置是等价的:
>
> ```
> location /one/ {
>     proxy_pass     http://upstream:port/two/;
>     proxy_redirect default;
> ```
>
> ```
> location /one/ {
>     proxy_pass     http://upstream:port/two/;
>     proxy_redirect http://upstream:port/two/ /one/;
> ```
>
> 如果proxy_pass包含变量，则不能使用`default` ，但是proxy_redirect中可以使用变量，如下：
>
> ```
> proxy_redirect http://localhost:8000/ http://$host:$server_port/;
> proxy_redirect http://$proxy_host:8000/ /;
> ```
>
> 在1.11及以后的版本里，可以使用正则匹配，''~" 大小写敏感 ，"~*" 不区分大小写开始匹配。
>
> ```
> proxy_redirect ~^(http://[^:]+):\d+(/.+)$ $1$2;
> proxy_redirect ~*/user/([^/]+)/(.+)$      http://$1.example.com/$2;
> ```
>
> 可以定义多条
>
> ```
> proxy_redirect default;
> proxy_redirect http://localhost:8000/  /;
> proxy_redirect http://www.example.com/ /;
> ```
>
> 如果多条指令可以应用到代理服务器的响应头中，只有第一个匹配的被选择。
>
> 使用以下指令，也可以将主机名添加到代理服务器发出的相对重定向：
>
> ```
> proxy_redirect / /;
> ```

​       **proxy_next_upstream**

> 当你使用Nginx proxy代理时，如果是代理到后端是使用upstream，那么这个指令就是指定在何种情况下，一个失败的请求应该被发送到下一台后端服务器。
>
> error – 和后端服务器建立连接时，或向后端服务器发送请求时，或从后端服务器读取响应时，出现错误；
>
> timeout – 和后端服务器建立连接时，或向后端服务器发送请求时，或从后端服务器读取响应时，出现超时；
>
> invalid_header – 后端服务器返回空响应或者非法响应头；
>
> http_500 – 后端服务器返回的响应状态码为500；
>
> http_502 – 后端服务器返回的响应状态码为502；
>
> http_503 – 后端服务器返回的响应状态码为503；
>
> http_504 – 后端服务器返回的响应状态码为504；
>
> http_404 – 后端服务器返回的响应状态码为404；
>
> http_403 – 后端服务器返回的响应状态码为403；
>
> http_429 – 后端服务器返回的响应状态码为429；(1.11.13)
>
> non_idempotent–通常，如果请求已发送到上游服务器（1.9.13），则使用非幂等方法(POST、LOCK、PATCH)的请求不会传递给下一个服务器；启用此选项显式允许重试此类请求；
>
> off – 关闭proxy_next_upstream功能—出错就选择另一台上游服务器再次转发。
>
> 只有在没有向客户端发送任何请求的情况下，才有可能将请求传递给下一个服务器。 也就是说，如果在传输响应的过程中发生错误或超时，那么修复这个错误是不可能的。
>
> 将请求传递给下一个服务器可以受到尝试次数和时间的限制，默认均为0， 即失败满足条件立即重试，以下为指令定义 （1.7.5）
>
> | Syntax:  | `proxy_next_upstream_timeout time;` |
> | :------- | ----------------------------------- |
> | Default: | `proxy_next_upstream_timeout 0;`    |
> | Context: | `http`, `server`, `location`        |
>
> ----
>
> | Syntax:  | `proxy_next_upstream_tries number;` |
> | :------- | ----------------------------------- |
> | Default: | `proxy_next_upstream_tries 0;`      |
> | Context: | `http`, `server`, `location`        |

## 负载均衡

###  概念

> 负载均衡，英文名称为Load Balance，其含义就是指将互联网或内网的流量（工作任务）进行平衡、分摊到多个操作单元上进行运行，例如FTP服务器、Web服务器、企业核心应用服务器和其它主要任务服务器等，从而协同完成工作任务。

###  作用

> 负载均衡构建在原有网络结构之上，它提供了一种透明且廉价有效的方法扩展服务器和网络设备的带宽、加强网络数据处理能力、增加吞吐量、提高网络的可用性和灵活性。它可以通过流量分发，隐藏实际服务端口，消除服务的单点故障，达到提高应用系统可用性，增强安全性，提升可靠性的要求

### NGINX负载均衡

#### 基本配置

默认情况下，使用加权轮询平衡方法在服务器之间分配请求。 在下面的示例中，每7个请求将被分发如下：5个请求转到backend1.example.com，一个请求发送到第二和第三个服务器中的每个服务器。 如果在与服务器通信期间发生错误，请求将被传递到下一个服务器，以此类推，直到所有功能服务器都被尝试为止。 如果无法从任何服务器获得成功的响应，客户端将收到与最后一台服务器通信的结果。

> ```
> upstream backend {
>     server backend1.example.com       weight=5;
>     server backend2.example.com:8080;
>     server unix:/tmp/backend3;
> 
>     server backup1.example.com:8080   backup;
>     server backup2.example.com:8080   backup;
> }
> 
> server {
>     location / {
>         proxy_pass http://backend;
>     }
> }
> ```

> 关于 `server unix:/tmp/backend3;` “unix：”前缀之后指定的UNIX域套接字路径
>
> Unix domain socket也叫 IPC socket (inter-process communication socket)，即进程间通信 socket，主要用于同一主机上的进程间通信。与主机间的进程通信不同，它不是通过 "IP地址 + TCP或UDP端口号" 的方式进程通信，而是使用 socket 类型的文件来完成通信。

##### SERVER配置项

SERVER指定了一台上游服务器的名字，这个名字可以是域名、IP地址端口、UNIX句柄等，在其后还可以跟下列参数:

> - **weight=number**：
>
>   ​	设置向这台上游服务器转发的权重，默认为1。
>
> - **max_fails=number**：
>
>   ​	该选项与fail_timeout配合使用，指在fail_timeout时间段内，如果向当前的上游服务器转 发失败次数超过number，则认为在当前的fail_timeout时间段内这台上游服务器不可用。max_fails默认为1，如果设置为0，则表示 不检查失败次数。
>
> - **fail_timeout=time：fail_timeout**
>
>   ​	表示该时间段内转发失败多少次后就认为上游服务器暂时不可用，用于优化反向代理功能。它与向上游服务器建立连接的超时时间、读取上游服务器的响应超时时间等完全无关。fail_timeout默认为10秒。
>
> - **down**：
>
>   ​	表示所在的上游服务器永久下线，只在使用ip_hash配置项时才有用。
>
> - **backup**：
>
>   ​	在使用ip_hash配置项时它是无效的。它表示所在的上游服务器只是备份服务器，只有在所有的非备份上游服务器都失效后，才会向所在的上游服务器转发请求。

#####  keepalive

> | Syntax:  | `keepalive connections;` |
> | :------- | ------------------------ |
> | Default: | —                        |
> | Context: | `upstream`               |
>
> This directive appeared in version 1.1.4.
>
> 连接参数设置到上游服务器的空闲keepalive连接的最大数量，这些连接保存在每个工作进程的缓存中。 当超过此数字时，关闭最近使用最少的连接。
>
> 应该特别注意的是，keepalive指令没有限制nginx工作进程可以打开的连接到上游服务器的总数。 连接参数应该设置为足够小的数字，以便上游服务器处理新的传入连接。
>
> 当使用默认轮询方法以外的负载平衡方法时，必须在keepalive指令之前激活它们。
>
> 对于HTTP，proxy_http_version指令应该设置为“1.1”，并且应该清除“连接”头字段：
>
> ```
> upstream http_backend {
>     server 127.0.0.1:8080;
>     keepalive 16;
> }
> 
> server {
>     ...
>     location /http/ {
>         proxy_pass http://http_backend;
>         proxy_http_version 1.1;
>         proxy_set_header Connection "";
>         ...
>     }
> }
> ```
>
> 或者，HTTP/1.0持久连接可以通过将“Connection：Keep-Alive”头字段传递给上游服务器来使用，尽管不推荐使用此方法。
>
> 对于FastCGI服务器，需要为保持连接设置fastcgi_keep_conn以工作：
>
> ```
> upstream fastcgi_backend {
>     server 127.0.0.1:9000;
>     keepalive 8;
> }
> server {
>     ...
>     location /fastcgi/ {
>         fastcgi_pass fastcgi_backend;
>         fastcgi_keep_conn on;
>         ...
>     }
> }
> ```
>
> SCGI和uwsgi协议没有保持连接的概念。

##### keepalive_requests

> | Syntax:  | `keepalive_requests number;` |
> | :------- | ---------------------------- |
> | Default: | `keepalive_requests 100;`    |
> | Context: | `upstream`                   |
>
> This directive appeared in version 1.15.3.
>
> 设置可以通过一个保持连接服务的请求的最大数量。 请求的最大数量后，连接关闭。
> 定期关闭连接是必要的，以释放每个连接内存分配。 因此，使用过高的最大请求数可能导致内存使用过多，不推荐使用。

##### keepalive_timeout

> | Syntax:  | `keepalive_timeout timeout;` |
> | :------- | ---------------------------- |
> | Default: | `keepalive_timeout 60s;`     |
> | Context: | `upstream`                   |
>
> This directive appeared in version 1.15.3.
>
> 设置一个到上游服务器的空闲保持连接超时时间。

#### 负载均衡算法

##### 1、轮询（默认）

每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器down掉，能自动剔除。

> upstream myapp1 {
>         server srv1.example.com;
>         server srv2.example.com;
>         server srv3.example.com;
> }

##### 2、weight

指定轮询几率，weight和访问比率成正比，用于后端服务器性能不均的情况。

> upstream myapp1 {
>         server srv1.example.com weight=3;
>         server srv2.example.com weight=2;
>         server srv3.example.com weight=2;
>     }

##### 3、least_conn

指定组应该使用负载均衡方法，其中将请求传递给活动连接数量最少的服务器，同时考虑到服务器的权重。 如果有几个这样的服务器，则使用加权循环平衡方法依次尝试。

> upstream myapp1 {
>     least_conn;
>     server srv1.example.com;
>     server srv2.example.com;
>     server srv3.example.com;
> }

##### 4、ip_hash

每个请求按访问ip的hash结果分配，这样每个访客固定访问一个后端服务器，可以解决session的问题
如果其中一个服务器需要暂时移除，则应该用向下参数标记，以保持客户端IP地址的当前散列。

> upstream myapp1 {
>         ip_hash;
>         server srv1.example.com;
>         server srv2.example.com  down;
>         server srv3.example.com;
> 	}

##### 5、**hash** key [`consistent`]

  客户机-服务器映射基于散列键值。 键可以包含文本，变量及其组合。 请注意，从组中添加或删除服务器可能会导致将大多数键重映射到不同的服务器。 该方法与缓存：Memcached Perl库兼容。如果指定了一致参数，则将使用ketama一致性散列方法。 该方法确保当一个服务器被添加到或从组中删除时，只有少数密钥将被重新映射到不同的服务器。 这有助于为缓存服务器实现更高的缓存命中率。 该方法与缓存：Memcached：：Fast Perl库兼容，ketama_points参数设置为160。

> upstream myapp1 {
>         hash $request_uri;
>         server srv1.example.com;
>         server srv2.example.com;
>         server srv3.example.com;
> 	}



##### 6、fair（第三方）

​    按后端服务器的响应时间来分配请求，响应时间短的优先分配。 需要安装fair模块

> ​    upstream myapp1 {
> ​        fair;
> ​        server srv1.example.com;
> ​        server srv2.example.com;
> ​        server srv3.example.com;
> ​	}

​    .............................................................................................................................................