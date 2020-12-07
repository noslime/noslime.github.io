---
title: nginx限流
date: 2020-12-06 14:06:22
author: noslime
categories: nginx
tags: nginx
keywords: nginx
---

#  限流

---



## 一、限流介绍

<p>限流（rate limiting）是NGINX众多特性中的重要特性之一。该特性可以限制某个用户在一个给定时间段内能够产生的HTTP请求数以及请求的处理速率。用于安全目的上，比如减慢暴力密码破解攻击。通过限制进来的请求速率，并且（结合日志）标记出目标URLs来帮助防范DDoS攻击，保护带宽及服务器的IO资源。一般地说，限流是用在保护上游应用服务器不被在同一时刻的大量用户请求湮没</p>

## 二、相关模块

### 1) ngx_http_limit_conn_module

此模块用于限制每个定义的键的连接数，特别是来自单个IP地址的连接数。 不是所有的连接都被计算在内。 只有当一个连接被服务器处理并且整个请求头已经被读取时，它才会被计数。用来限制同一时间连接数，即并发限制。

**配置示例**

```
http {
    limit_conn_zone $binary_remote_addr zone=addr:10m;
    ...
    server {
        ...
        location /download/ {
            limit_conn addr 1;
}
```

**相关指令**

| Syntax:  | `limit_conn zone number;`    |
| :------- | ---------------------------- |
| Default: | —                            |
| Context: | `http`, `server`, `location` |

设置共享内存区域和给定键值的最大允许连接数。 当超过此限制时，服务器将返回定义的错误界面。 例如

```
limit_conn_zone $binary_remote_addr zone=addr:10m;
server {
    location /download/ {
        limit_conn addr 1;
}
```

每个IP每次只允许一个连接，在HTTP/2和SPDY中，每个并发请求都被认为是一个单独的连接。

可以有多个limit_conn指令。例如，以下配置将限制每个客户端IP与服务器的连接数量，同时限制与虚拟服务器的连接总数：

```
limit_conn_zone $binary_remote_addr zone=perip:10m;
limit_conn_zone $server_name zone=perserver:10m;

server {
    ...
    limit_conn perip 10;
    limit_conn perserver 100;
}
```



| Syntax:  | `limit_conn_dry_run on {%raw%}|{%endraw%} off;` |
| :------- | ----------------------------------------------- |
| Default: | `limit_conn_dry_run off;`                       |
| Context: | `http`, `server`, `location`                    |

启用干运行模式（ 1.17.6）。 在这种模式下，连接的数量不受限制，但在共享内存区域中，过度连接的数量照常计算。



| Syntax:  | `limit_conn_log_level info {%raw%}|{%endraw%} notice {%raw%}|{%endraw%} warn {%raw%}|{%endraw%} error;` |
| -------- | ------------------------------------------------------------ |
| Default: | `limit_conn_log_level error;`                                |
| Context: | `http`, `server`, `location`                                 |

设置服务器限制连接数量的情况下所需的日志记录级别（0.8.18）。



| Syntax:  | `limit_conn_status code;`    |
| :------- | ---------------------------- |
| Default: | `limit_conn_status 503;`     |
| Context: | `http`, `server`, `location` |

设置要返回的状态代码以响应被拒绝的请求(1.3.15)。



| Syntax:  | `limit_conn_zone key zone=name:size;` |
| :------- | ------------------------------------- |
| Default: | —                                     |
| Context: | `http`                                |

设置共享内存区域的参数，该区域将保留各种键的状态。 其中，状态包括当前连接数。 键可以包含文本，变量及其组合。 不计算带空键值的请求。在版本1.7.6之前，一个键只能包含一个变量。

使用示例

```
limit_conn_zone $binary_remote_addr zone=addr:10m;
```

其中内置参数`$binary_remote_addr`为客户端的IP地址，与 `$remote_addr`相比，前者为定长，后者不定长。addr 为共享内存区域的名字，10m为大小。

### 2)   ngx_http_limit_req_module

此模块（0.7.21）用于限制每个定义的键的请求处理速率，特别是来自单个IP地址的请求的处理速率。 限制是使用“漏桶”算法（缓存请求、匀速处理、多余的请求直接丢弃）完成的。单位时间的请求数，速率限制。

**配置示例**

```
http {
    limit_req_zone $binary_remote_addr zone=one:10m rate=1r/s;
    ...
    server {
        ...
        location /search/ {
            limit_req zone=one burst=5;
 }
```

**相关指令**

| Syntax:  | `limit_req zone=name [burst=number] [nodelay {%raw%}|{%endraw%} delay=number];` |
| :------- | ------------------------------------------------------------ |
| Default: | —                                                            |
| Context: | `http`, `server`, `location`                                 |

共享内存区的大小和请求的最大值。如果请求速率超过为区域配置的速率，则它们的处理将被延迟，以便以定义的速率处理请求。过多的请求会被延迟，直到它们的数量超过最大突发大小，在这种情况下，请求会因错误而终止。默认情况下，最大突发大小等于零。

```
limit_req_zone $binary_remote_addr zone=one:10m rate=1r/s;

server {
    location /search/ {
        limit_req zone=one burst=5;
}
```

上述配置表示平均每秒允许不超过1个请求，突发请求不超过5个。burst这个配置的意思是设置一个大小为5的缓冲区当有大量请求（爆发）过来时，超过了访问频次限制的请求可以先放到这个缓冲区内。

延迟参数（1.15.7）指定了过度请求被延迟的限制。 默认值为零，即所有超速的请求都被延迟了

如果不希望在请求受到限制时延迟过多的请求，则应该使用参数nodelay：

```
limit_req zone=one burst=5 nodelay;
```

如果设置，超过访问频次而且缓冲区也满了的时候就会直接返回错误503，如果没有设置，则所有请求会等待排队。

可以配置多个limit_req指令。 例如，以下配置将限制来自单个IP地址的请求的处理速率，同时限制虚拟服务器的请求处理速率：

```
limit_req_zone $binary_remote_addr zone=perip:10m rate=1r/s;
limit_req_zone $server_name zone=perserver:10m rate=10r/s;
server {
    ...
    limit_req zone=perip burst=5 nodelay;
    limit_req zone=perserver burst=10;
}
```



| Syntax:  | `limit_req_dry_run on {%raw%}|{%endraw%} off;` |
| :------- | ---------------------------------------------- |
| Default: | `limit_req_dry_run off;`                       |
| Context: | `http`, `server`, `location`                   |

启用干运行模式（1.17.1）。在这种模式下，请求处理速率不受限制，但是在共享内存区中，过量请求的数量照常计算



| Syntax:  | `limit_req_log_level info {%raw%}|{%endraw%} notice {%raw%}|{%endraw%} warn {%raw%}|{%endraw%} error;` |
| :------- | ------------------------------------------------------------ |
| Default: | `limit_req_log_level error;`                                 |
| Context: | `http`, `server`, `location`                                 |

为服务器因速率超过而拒绝处理请求或延迟请求处理的情况设置所需的日志记录级别（ 0.8.18）。延迟的日志记录级别比拒绝的记录级别低一分；例如，如果指定了“limit_req_log_level notice”，则延迟将用info级别记录。



| Syntax:  | `limit_req_status code;`     |
| :------- | ---------------------------- |
| Default: | `limit_req_status 503;`      |
| Context: | `http`, `server`, `location` |

设置要返回的状态代码以响应被拒绝的请求 (1.3.15)



| Syntax:  | `limit_req_zone key zone=name:size rate=rate [sync];` |
| :------- | ----------------------------------------------------- |
| Default: | —                                                     |
| Context: | `http`                                                |

为共享内存区域设置参数，该区域将保持各种键的状态。其中，状态存储当前过多请求的数量。键可以包含文本、变量及其组合。不计算键值为空的请求。在版本1.7.6之前，一个键只能包含一个变量。

指令示例

```
limit_req_zone $binary_remote_addr zone=one:10m rate=1r/s;
```

在这里，状态被保存在一个10兆字节的区域“one”，并且该区域的平均请求处理速率不能超过每秒1个请求

如果区域存储耗尽，则删除最近使用最少的状态。 如果在此之后无法创建新状态，则请求将以错误终止。

速率在每秒请求(r/s)中指定)。 如果要求每秒少于一个请求的速率，则在每分钟请求(r/m)中指定)。 例如，每秒半请求为30r/m

## 三、其他限制指令

### 1)限速

| Syntax:  | `limit_rate rate;`                             |
| :------- | ---------------------------------------------- |
| Default: | `limit_rate 0;`                                |
| Context: | `http`, `server`, `location`, `if in location` |

限制对客户端的响应传输速率。 速率以每秒字节为单位指定。 零值禁用速率限制。 限制是根据请求设置的，因此如果客户端同时打开两个连接，则总速率将是指定限制的两倍

参数值可以包含变量（1.17.0）。在根据特定条件限制速率的情况下，该方法可能有用：

```
map $slow $rate {
    1     4k;
    2     8k;
}
limit_rate $rate;
```

速率限制也可以在$limit_rate变量中设置，但是，由于版本1.17.0，不推荐使用此方法：

```
server {

    if ($slow) {
        set $limit_rate 4k;
    }
    ...
}
```



| Syntax:  | `limit_rate_after size;`                       |
| :------- | ---------------------------------------------- |
| Default: | `limit_rate_after 0;`                          |
| Context: | `http`, `server`, `location`, `if in location` |

设置初始数量，在此之后，对客户端的响应的进一步传输将受到速率限制。参数值可以包含变量（1.17.0）。

```
location /flv/ {
    flv;
    limit_rate_after 500k;
    limit_rate       50k;
}
```



### 2)限制HTTP方法

| Syntax:  | `limit_except method ... { ... }` |
| :------- | --------------------------------- |
| Default: | —                                 |
| Context: | `location`                        |

限制一个位置内允许的HTTP方法。方法参数可以是以下参数之一：GET、HEAD、POST、PUT、DELETE、MKCOL、COPY、MOVE、OPTIONS、PROPFIND、PROPPATCH、LOCK、UNLOCK或PATCH。允许GET方法使HEAD方法也被允许。可以使用ngx_http_Access_module、ngx_http_auth_basic_module和ngx_http_auth_jwt_module（1.13.10）modules指令来限制对其他方法的访问：

```
limit_except GET {
    allow 192.168.1.0/32;
    deny  all;
}
```

请注意，这将限制对所有方法的访问，除了GET和HEAD。