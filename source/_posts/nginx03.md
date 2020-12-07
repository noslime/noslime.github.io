---
title: nginx的web服务
date: 2020-11-27 23:17:22
author: noslime
categories: nginx
tags: nginx
keywords: nginx
---

# nginx的web服务

nginx这款优秀的web服务器，作为后起之秀，在日益拉近与服务器老大哥Apathe的距离，自然有启独特之处，所以值得认真学习一下。本文是一次自己学习nginx的课程记录，课程来源：[baism] 

[baism]: https://www.bilibili.com/video/BV1fr4y1c7Gz

## 

#### **默认网站**

如果nginx只有一个server，这个server就是默认的网站，会默认访问index.html， 以下为初始的默认网站

```
server {
    listen       80;
    server_name  localhost;
    location / {
        root   html;
        index  index.html index.htm;
    }
    error_page   500 502 503 504  /50x.html;
    	location = /50x.html {
        root   html;
    }
}
```

#### **目录浏览**

显示服务器根目录的目录列表，需要删除默认的index.html文件

```
server {
    listen       80;
    server_name  localhost;
    location / {
        autoindex on;  //开启目录浏览
        autoindex_exact_size off;  //关闭确切大小，便于人眼识别。默认为bytes.
        autoindex_localtime on;   //显示文件的服务器时间
    }
    error_page   500 502 503 504  /50x.html;
    	location = /50x.html {
        root   html;
    }
}
```

####  **访问控制**

见文知意，访问控制就是对访问者进行限制，以下根据具体IP控制访问

```config
#只允许本机访问a目录，其他返回百度
location /a {
    autoindex on;
    allow 127.0.0.1;
    deny all;
    return http://www.biadu.com;
}

```

#### **登录验证**

需要输入用户及密码才能访问，可以使用htpasswd或openssl等生成用户密码

1. 安装htpasswod工具，在http-tools包下

   ```shell 
   yum -y install httpd-tools
   ```

2. 生成用户密码

   ```shell
   mkdir /usr/local/nginx/passwd
   htpasswd -c /usr/local/nginx/passwd/htpasswd username
   ```

   

3. 配置pass目录的验证需要登录

   设置登录后的显示信息

   ```shell
   echo "Welcome to login" > index.html
   ```

   配置文件

   ```
   location /pass {
       auth_basic "Login verification";
       auth_basic_user_file /usr/local/nginx/passwd/htpasswd;
   }
   
   ```

#### **防止盗链**

防盗链功能基于HTTP协议支持的Referer机制，通过Referer跟踪来源，对来源进行识别和判断。

```
location ~* \.(png|gif|bmp)$ {
alias /data/images/;
	valid_referers none blocked *.images.com;
    if ($invalid_referer) {
       return 403;
    }
}
```

#### 虚拟主机

1. 基于IP的虚拟主机 

   ```
server {
       listen       192.168.0.20:80;
       location / {
           root   html/app1;
           index  index.html index.htm index.php;
       }
   }
   server {
       listen       192.168.0.21:80;
   	location / {
           root   html/app2;
           index  index.html index.htm;
       }
   }
   
   ```
   
2. 基于端口的虚拟主机

   ```
   基于端口
   server {
       listen       80;
       server_name  www.demo.com;
       location / {
           root   html/app1;
           index  index.html index.htm index.php;
       }
   }
   server {
       listen       8080;
       server_name  www.demo.com;
       location / {
           root   html/app2;
           index  index.html index.htm;
       }
   }
   ```

3. 基于域名的虚拟主机

   ```
   server {
       listen       80;
       server_name  app1.demo.com;
       location / {
           root   html/app1;
           index  index.html index.htm index.php;
       }
   }
   server {
       listen       80;
       server_name  app2.demo.com;
       location / {
           root   html/app2;
           index  index.html index.htm;
       }
   }
   ```

   