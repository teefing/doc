---
title: nginx
category: 专业知识
date: 2019-05-20 22:53:35
tags: 
 - nginx
---
花了一周时间了解 nginx 相关的知识，主要内容有：**基础知识：** Nginx 的快速部署安装、模块、基础配置语法，Nginx 的日志输出、Nginx 默认配置模块、Nginx 做为 http 代理服务, 介绍代理服务的类型，正向反向代理配置，nginx 作为的应用层负载均衡服务的各种应用，hash 负载均衡策略, Nginx 缓存，**高级知识：** Nginx 常用配置模块, rewirte 的配置语法和规则，配置基于指定地域的规则访问, geoip 模块、https 的实现原理，配置 nginx 的 https 服务, secure_link_module 的防盗链实现，讲解，讲解 Lua 的开发语法、配合 Nginx 实现高效的认证系统和其他场景。

# 基础知识

## 环境

### 初始环境

**docker 启动**

```
docker run -d -p 8088:80 --name nginx_8088 nginx_80:latest /sbin/init

```

### 四项确认

*   确认系统网络（ping)
*   确认 yum 可用 (yum list | grep gcc
*   确认关闭 iptables (iptables -F)
*   确认停用 selinux

### 两项安装

```
yum -y install gcc gcc-c++ autoconf pcre pcre-devel make automake
yum -y install wget httpd-tools vim

```

### 一次初始化

```
cd /opt/ mkdir app download logs work backup
```

### nginx 安装

**确定 nginx 源**

```
cd /etc/yum.repos.d
vim nginx.repo 

添加：

name=nginx repo
baseurl=http://nginx.org/packages/centos/7/$basearch/
gpgcheck=0
enabled=1
```

**安装**

```
yum list | grep nginx
yum install nginx 


```

**查看版本**

```
nginx -v
```

**查看 nginx 编译的参数**

```
nginx -V
```

### nginx 启动

```
nginx -c /etc/nginx/nginx.conf 
```

**重启 nginx 服务**

```
systemctl restart nginx.service
```

**柔和重启**

```
nginx -s reload -c /etc/nginx/nginx.conf
```

**检查配置文件**

```
nginx -t -c /etc/nginx/nginx.conf
```

## 中间件架构

![](https://user-gold-cdn.xitu.io/2017/12/5/1602472a96bc918d?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### nginx 简述

> nginx 是一个开源且高性能、可靠的 http 中间件，代理服务。Nginx（发音同 engine x）是一个 Web 服务器，也可以用作反向代理，负载平衡器和 HTTP 缓存。该软件由 Igor Sysoev 创建，并于 2004 年首次公开发布。同名公司成立于 2011 年，以提供支持。

### 为什么选择 nginx

#### io 多路复用 epoll

> 多个描述符的 i/o 操作都能在一个线程内并发交替地顺序完成，这就教 i/o 多路复用，这里的” 复用 “指的是复用同一个线程。i/o 多路复用的实现方式为：select、poll、epoll

![](https://user-gold-cdn.xitu.io/2017/12/5/1602472a9800726b?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

**什么是 select**

![](https://user-gold-cdn.xitu.io/2017/12/5/1602472a984fc82e?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

**epoll 模型**

*   当 FD 就绪，采用系统的回调函数之间将 fd 放入，效率更高
*   最大连接无限制

#### 轻量级

*   功能模块少
*   代码模块化

#### cpu 亲和（affinity)

> 是一种 cpu 核心和 nginx 工作进程绑定方式，把每个 worker 进程固定在一个 cpu 上执行，减少切换 cpu 的 cache miss，获得更好的性能。
> 
> ![](https://user-gold-cdn.xitu.io/2017/12/5/1602472a9a564e2c?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

说白了就是减少 cpu 切换所损耗的性能

#### sendfile

![](https://user-gold-cdn.xitu.io/2017/12/5/1602472a9b5614e6?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

## nginx 版本

*   Mainline version 开发版
*   Stable version 稳定版
*   Legacy version 历史版本

## 基本参数使用

**rpm**

> rpm 命令是 RPM 软件包的管理工具。rpm 原本是 Red Hat Linux 发行版专门用来管理 Linux 各项套件的程序，由于它遵循 GPL 规则且功能强大方便，因而广受欢迎。逐渐受到其他发行版的采用。RPM 套件管理方式的出现，让 Linux 易于安装，升级，间接提升了 Linux 的适用度。

### 安装目录

**列出服务的安装目录**

`rpm -ql nginx`

列出下面目录

```
/etc/logrotate.d/nginx
/etc/nginx
/etc/nginx/conf.d
/etc/nginx/conf.d/default.conf
/etc/nginx/fastcgi_params
/etc/nginx/koi-utf
/etc/nginx/koi-win
/etc/nginx/mime.types
/etc/nginx/modules
/etc/nginx/nginx.conf
/etc/nginx/scgi_params
/etc/nginx/uwsgi_params
/etc/nginx/win-utf
/etc/sysconfig/nginx
/etc/sysconfig/nginx-debug
/usr/lib/systemd/system/nginx-debug.service
/usr/lib/systemd/system/nginx.service
/usr/lib64/nginx
/usr/lib64/nginx/modules
/usr/libexec/initscripts/legacy-actions/nginx
/usr/libexec/initscripts/legacy-actions/nginx/check-reload
/usr/libexec/initscripts/legacy-actions/nginx/upgrade
/usr/sbin/nginx
/usr/sbin/nginx-debug
/usr/share/doc/nginx-1.12.2
/usr/share/doc/nginx-1.12.2/COPYRIGHT
/usr/share/man/man8/nginx.8.gz
/usr/share/nginx
/usr/share/nginx/html
/usr/share/nginx/html/50x.html
/usr/share/nginx/html/index.html
/var/cache/nginx
/var/log/nginx 
```

**目录解释**

| 路径 | 类型 | 作用 |
| --- | --- | --- |
| /etc/logrotate.d/nginx | 配置文件 | nginx 日志轮转，用于 logrotate 服务的日志切割 |
| /etc/nginx
/etc/nginx/conf.d
/etc/nginx/conf.d/default.conf
/etc/nginx/nginx.conf | 目录、配置文件 | nginx 主配置文件 |
| /etc/nginx/fastcgi_params
/etc/nginx/scgi_params
/etc/nginx/uwsgi_params | 配置文件 | cgi 配置相关，fastcgi 配置 |
| /etc/nginx/koi-utf
/etc/nginx/koi-win
/etc/nginx/win-utf | 配置文件 | 编码映射转化文件 |
| /etc/nginx/mime.types | 配置文件 | 设置 http 协议的 Content-Type 与扩展名对应关系 |
| /etc/sysconfig/nginx
/etc/sysconfig/nginx-debug
/usr/lib/systemd/system/nginx-debug.service
/usr/lib/systemd/system/nginx.service | 配置文件 | 用于配置出系统守护进程管理器管理方式 |
| /etc/nginx/modules
/usr/lib64/nginx/modules | 目录 | nginx 模块目录 |
| /usr/sbin/nginx
/usr/sbin/nginx-debug | 命令 | nginx 服务的启动管理的终端命令 |
| /usr/share/doc/nginx-1.12.2
/usr/share/doc/nginx-1.12.2/COPYRIGHT
/usr/share/man/man8/nginx.8.gz | 文件目录 | nginx 的手册和帮助文件 |
| /var/cache/nginx | 目录 | nginx 的缓存目录 |
| /var/log/nginx | 目录 | nginx 的日志目录 |

## 编译参数

**列出编译参数的命令**

`nginx -V`

**结果**

```
nginx version: nginx/1.12.2
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-16) (GCC) 
built with OpenSSL 1.0.2k-fips  26 Jan 2017
TLS SNI support enabled
configure arguments: --prefix=/etc/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib64/nginx/modules --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --pid-path=/var/run/nginx.pid --lock-path=/var/run/nginx.lock --http-client-body-temp-path=/var/cache/nginx/client_temp --http-proxy-temp-path=/var/cache/nginx/proxy_temp --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp --http-scgi-temp-path=/var/cache/nginx/scgi_temp --user=nginx --group=nginx --with-compat --with-file-aio --with-threads --with-http_addition_module --with-http_auth_request_module --with-http_dav_module --with-http_flv_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_mp4_module --with-http_random_index_module --with-http_realip_module --with-http_secure_link_module --with-http_slice_module --with-http_ssl_module --with-http_stub_status_module --with-http_sub_module --with-http_v2_module --with-mail --with-mail_ssl_module --with-stream --with-stream_realip_module --with-stream_ssl_module --with-stream_ssl_preread_module --with-cc-opt='-O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector-strong --param=ssp-buffer-size=4 -grecord-gcc-switches -m64 -mtune=generic -fPIC' --with-ld-opt='-Wl,-z,relro -Wl,-z,now -pie'

```

** 参数解释 **

| 路径 | 类型 |
| --- | --- |
| --prefix=/etc/nginx
--sbin-path=/usr/sbin/nginx
--modules-path=/usr/lib64/nginx/modules
--conf-path=/etc/nginx/nginx.conf
--error-log-path=/var/log/nginx/error.log
--http-log-path=/var/log/nginx/access.log
--pid-path=/var/run/nginx.pid
--lock-path=/var/run/nginx.lock | 安装目的目录或路径 |
| --http-client-body-temp-path=/var/cache/nginx/client_temp
--http-proxy-temp-path=/var/cache/nginx/proxy_temp
--http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp
--http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp
--http-scgi-temp-path=/var/cache/nginx/scgi_temp | 执行对应模块时，nginx 所保留的临时性文件 |
| --user=nginx --group=nginx | 设定 nginx 进程启动的用户和组用户 |
| --with-cc-opt=parameters | 设置额外的参数将被添加到 CFLAGS 变量 |
| --with-ld-opt=parameters | 设置附加的参数，链接系统库 |

## nginx 基本配置语法

### http 相关

**展示每次请求的请求头**

```
curl -v http://www.baidu.com

```

**结果**

```
% Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0* About to connect() to www.baidu.com port 80 (#0)
*   Trying 61.135.169.121...
* Connected to www.baidu.com (61.135.169.121) port 80 (#0)
> GET / HTTP/1.1
> User-Agent: curl/7.29.0
> Host: www.baidu.com
> Accept: */*
> 
< HTTP/1.1 200 OK
< Server: bfe/1.0.8.18
< Date: Thu, 30 Nov 2017 02:14:02 GMT
< Content-Type: text/html
< Content-Length: 2381
< Last-Modified: Mon, 23 Jan 2017 13:27:32 GMT
< Connection: Keep-Alive
< ETag: "588604c4-94d"
< Cache-Control: private, no-cache, no-store, proxy-revalidate, no-transform
< Pragma: no-cache
< Set-Cookie: BDORZ=27315; max-age=86400; domain=.baidu.com; path=/
< Accept-Ranges: bytes
< 
{ [data not shown]
100  2381  100  2381    0     0  88266      0 --:--:-- --:--:-- --:--:-- 91576
* Connection #0 to host www.baidu.com left intact


```

### nginx 日志类型

*   error.log、 access.log
*   log_format

_格式_ *

```
syntax: log_format name [escape=default | json] string...;
default: log_format combined "...";
context:http

```

### nginx 变量

**nginx 配置的内容**

```
user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;

events {
    worker_connections  1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}


```

**变量类型**

*   http 请求变量：arg_PARAMETER,http_header,sent_http_header
*   内置变量：nginx 内置的
*   自定义变量： 自己定义

### nginx 模块

*   nginx 官方模块
*   第三方模块

**default.conf**

```
server {
    listen       80;
    server_name  localhost;

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
}


```

**nginx 开启的模块**

```
--with-compat --with-file-aio --with-threads --with-http_addition_module --with-http_auth_request_module --with-http_dav_module --with-http_flv_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_mp4_module --with-http_random_index_module --with-http_realip_module --with-http_secure_link_module --with-http_slice_module --with-http_ssl_module --with-http_stub_status_module --with-http_sub_module --with-http_v2_module --with-mail --with-mail_ssl_module --with-stream --with-stream_realip_module --with-stream_ssl_module --with-stream_ssl_preread_module

```

### 安装编译模块

| 编译选项 | 作用 |
| --- | --- |
| --with-http_stub_status_module | nginx 的客户端状态 |
| --with-http_random_index_module | 目录中选择一个随机主页 |
| --with-http_sub_module | http 内容替换 |
| --limit_conn_module | 连接频率限制 |
| --limit_req_module | 请求频率限制 |
| http_access_module | 基于 ip 的访问控制 |
| http_auth_basic_module | 基于用户的信任登录 |

#### http_stub_status_module 配置

**配置语法**

```
syntax: stub_status;
default:-
context:server, location

```

在 default.conf 中添加：

```
# my config
location /mystatus {
	stub_status;
}

```

**检查和重新启动配置**

```
nginx -tc /etc/nginx/nginx.conf 

```

**重启服务**

```
nginx -s reload -c /etc/nginx/nginx.conf 

```

**检查效果** 输入：http://127.0.0.1:8088/mystatus

![](https://user-gold-cdn.xitu.io/2017/12/5/1602472a9b9ec647?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

表示 nginx 的活跃连接数；握手的总次数、处理连接数；读、写、等待个数

#### http_random_index_module

**配置语法**

```
syntax: random_index on | off;
default:random_index off;
context:location

```

在 default.conf 中将下面的配置：

```
location / {
root   /usr/share/nginx/html;
index  index.html index.htm;
}

```

改为：

```
location / {
root   /usr/share/nginx/html;
random_index on;
#index  index.html index.htm;
}

```

重启后多刷几次网页：

![](https://user-gold-cdn.xitu.io/2017/12/5/1602472ac9cf8cb8?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

主页出现不同了。

![](https://user-gold-cdn.xitu.io/2017/12/5/1602472ac39dac25?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

#### http_sub_module （替换）

**配置语法**

```
syntax: sub_filter string replacement;
default:-
context:http,server,location

```

```
syntax: sub_filter_last_modified on | off (重要用户缓存)
default:sub_filter_last_modified off;
context:http,server,location

```

```
syntax: sub_filter_once on | off 
default:sub_filter_once on;
context:http,server,location

```

是否只替换一次

#### 连接限制

**压力测试**

```
ab -n 50 -c 20 http://127.0.0.1/index.html

```

-n 表示请求次数 -c 表示并发数

**配置语法**

**连接限制**

```
syntax: limit_conn_zone key zone=name:size;
default:-
context:http

```

```
syntax:limit_conn zone number;
default:-
context:http, server, location 

```

**请求限制**

```
syntax: limit_req_zone key zone=name:size rate=rate;
default:-
context:http

```

```
syntax: limit_req_zone name [burst=number] [nodelay];
default:-
context:http,server,location

```

在 default.conf 中将下面的配置：

```
server {
    listen       80;
    server_name  localhost;

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    # my config
    location /mystatus {
        stub_status;
    }
    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
}


```

**改为：**

```
limit_conn_zone $binary_remote_addr zone=conn_zone:1m;
limit_req_zone $binary_remote_addr zone=req_zone:1m rate=1r/s;
server {
    listen       80;
    server_name  localhost;

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /usr/share/nginx/html;
        #limit_conn conn_zone 1;
        limit_req zone=req_zone;
        # limit_req zone=req_zone burst nodelay;
        # limit_req zone=req_zone burst nodelay;
        index  index.html index.htm;
    }

    # my config
    location /mystatus {
        stub_status;
    }
    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}


```

**压测结果**

**限制前**

![](https://user-gold-cdn.xitu.io/2017/12/5/1602472ac80880b8?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

**限制后**

![](https://user-gold-cdn.xitu.io/2017/12/5/1602472ac7d78e55?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

#### http_access_module

**配置语法**

```
syntax: allow address | CIDR | unix： | all
default:-
context:http,server,location,limit_except

```

```
syntax:deny address | CIDR | unix： | all
default:-
context:http, server, location ,limit_except

```

**测试** 配置如下

```
location ~ ^/admin.html {
  root /opt/app/code;
  deny all;
  index index.html index.htm;
}

```

**结果**

![](https://user-gold-cdn.xitu.io/2017/12/5/1602472adfe918cc?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

##### 局限性

![](https://user-gold-cdn.xitu.io/2017/12/5/1602472ac4718a8b?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

**http_x_forwarded_for**

![](https://user-gold-cdn.xitu.io/2017/12/5/1602472aee3b0e9b?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

http_x_forwarded_for = client ip, proxy(1)ip,proxy(2)ip,...

**解决方法**

*   采用别的 http 头信息控制访问，如 http_x_forward_for
*   结合 geo 模块操作
*   通过 http 自定义变量传递

#### http_auth_basic_module

**配置语法**

```
syntax: auth_basic string | off;
default:-
context:http,server,location,limit_except

```

```
syntax:auth_basic_user_file file;
default:-
context:http, server, location ,limit_except

```

**生成 password 文件**

```
htpasswd -c ./auth_conf wujunqi

```

**修改 conf 文件**

```
location ~ ^/admin.html {
  root /opt/app/code;
  auth_basic "please input you user name and passwd";
  auth_basic_user_file /etc/nginx/auth_conf;
  index index.html index.htm;
}

```

**测试**

![](https://user-gold-cdn.xitu.io/2017/12/5/1602472b01d0b451?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

##### 局限性

*   用户信息依赖文件方式
*   操作管理机械、效率低下

##### 解决方案

*   nginx 结合 LUA 实现高效验证
*   nginx 和 LDAP 打通，利用 nginx-auth-ldap 模块

# 场景实践

## 静态资源 web 服务

### 静态资源

**定义**

> 非服务器动态生成的文件

![](https://user-gold-cdn.xitu.io/2017/12/5/1602472afa6bab43?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)![](https://user-gold-cdn.xitu.io/2017/12/5/1602472af28ccf54?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### 静态资源服务场景 - CDN

![](https://user-gold-cdn.xitu.io/2017/12/5/1602472b055de73c?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### 文件读取配置

#### sendfile

```
syntax: sendfile on | off;
default:sendfile off
context:http,server,location,if in location

```

**注**

--with-file-aio 异步文件读取

#### tcp_nopush

作用：sendfile 开启的情况下，提高网络包的传输效率 (等待，一次传输）

```
syntax:tcp_nopush on | off
default:tcp_nopush off
context:http, server, location 

```

**相反的**

```
syntax:tcp_nodelay on | off
default:tcp_nodelay on
context:http, server, location 

```

**作用** 在 keepalive 连接下，提高网络包的传输实时性

#### 压缩

**作用**

压缩传输

```
syntax:gzip on | off
default:gzip off
context:http, server, if in location 

```

```
syntax:gzip_comp_level level;
default:gzip_comp_level 1;
context:http, server, location 

```

#### 扩展 nginx 压缩模块

*   http_gzip_static_module: 预读 gzip 功能
*   http_gunzip_module: 应用支持 gunzip 的压缩方式

#### 配置截图

![](https://user-gold-cdn.xitu.io/2017/12/5/1602472b1059ef48?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### 浏览器缓存

> http 协议定义的缓存机制（如：expires，cache-control 等）

*   浏览器无缓存

![](https://user-gold-cdn.xitu.io/2017/12/5/1602472b203baa71?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

*   浏览器有缓存

![](https://user-gold-cdn.xitu.io/2017/12/5/1602472b1f2dd22d?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

**检测过期机制**

| 作用 | 请求头 |
| --- | --- |
| 检验是否过期 | expires, cache-control (max-age) |
| 协议中 Etag 头信息校验 | etag |
| last-modified 头信息校验 | last-modified |

** 浏览器请求服务器过程（缓存版本）**

![](https://user-gold-cdn.xitu.io/2017/12/5/1602472b3960b1f5?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

#### 相关配置

##### expires

**添加 cache-control、expires 头**

```
syntax: expires [modified] time;
		expires epoch | max | off;
default: expires off;
context:http, server, location 

```

**配置例子**

![](https://user-gold-cdn.xitu.io/2017/12/5/1602472b3485c5d2?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### 跨域访问

![](https://user-gold-cdn.xitu.io/2017/12/5/1602472b3aa45180?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

**为什么浏览器禁止跨域访问**

不安全，容易出现 CSRF 攻击

![](https://user-gold-cdn.xitu.io/2017/12/5/1602472b51e885d5?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

#### nginx 配置

```
syntax: add_header name value [always]
default: -
context:http, server, location, if in location 

```

添加请求头：Access-Control-Allow-Origin

**配置截图**

![](https://user-gold-cdn.xitu.io/2017/12/5/1602472b6f2d9646?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### 防盗链

**目的**

防止资源被盗用

** 防盗链设置思路 **

首要方式：区别哪些请求是非正常的用户请求

#### 基于 http_refer 防盗链配置模块

```
syntax: valid_referers none | blocked | server_names | string...;
default: -
context:server, location

```

配置截图

![](https://user-gold-cdn.xitu.io/2017/12/5/1602472b5c9e7865?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

none: 表示如果没带 refer blocked: 代表不是标准的 http 写过过来的

一个命令

```
curl -e "http://www.baidu.com" -I http://116.62.103.228/wei.png

```

-e: 表示 refer -i: 表示只显示请求头

## 代理服务

> 代理 - 代为办理（代理理财、代理收货等）

![](https://user-gold-cdn.xitu.io/2017/12/5/1602472b4cebced7?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

**代理服务**

![](https://user-gold-cdn.xitu.io/2017/12/5/1602472b5ff2420c?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### 代理区别

区别在于代理的对象不一样

*   正向代理的对象是客户端
*   反向代理代理的是服务器

**正向代理**

![](https://user-gold-cdn.xitu.io/2017/12/5/1602472b6c05e617?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

**反向代理**

![](https://user-gold-cdn.xitu.io/2017/12/5/1602472b7e86eddb?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### 代理的配置

```
syntax: proxy_pass URL;
default: -
context:location, if in location, limit_except

```

**url 一般为：**

*   http://localhost:8000/uri/
*   https://192.168.1.1:8000/uri/
*   http://unix:/tmp/backend.socket:/uri/;

#### 反向代理配置截图

![](https://user-gold-cdn.xitu.io/2017/12/5/1602472b819a4863?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

想访问 8080，只能访问到 80，通过 80 然后通过反向代理可以访问到 8080

#### 正向代理配置截图

![](https://user-gold-cdn.xitu.io/2017/12/5/1602472b8ef65063?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

116.62.103.228 的配置如下（其实和反向代理配置参不多）

![](https://user-gold-cdn.xitu.io/2017/12/5/1602472b8acf710a?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

**客户端配置**

![](https://user-gold-cdn.xitu.io/2017/12/5/1602472bc8e37aa9?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### 缓存区配置

```
syntax: proxy_buffering on | off
default: proxy_buffering on
context:location,http,server

```

**扩展**

*   proxy_buffer_size
*   proxy_buffers
*   proxy_busy_buffers size

### 跳转重定向配置

```
syntax: proxy_redirect default;proxy_redirect off;proxy_redirect redirect replacement;
default: proxy_redirect default;
context:location,http,server

```

### 头信息配置

```
syntax: proxy_set_header field value;
default: proxy_set_header host $proxy_host
		 proxy_set_header connection close;
context:location,http,server

```

### 超时配置

```
syntax: proxy_connect_timeout time;
default: proxy_connect_timeout 60s;
context:location,http,server

```

**扩展**

*   proxy_read_timeout
*   proxy_send_timeout

### 总的配置

![](https://user-gold-cdn.xitu.io/2017/12/5/1602472b9627f63e?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

## 负载均衡调度器 SLB

**nginx 负载均衡**

![](https://user-gold-cdn.xitu.io/2017/12/5/1602472bac3c31ba?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### GSLB

> GSLB 是英文 Global Server Load Balance 的缩写，意思是全局负载均衡。作用：实现在广域网（包括互联网）上不同地域的服务器间的流量调配，保证使用最佳的服务器服务离自己最近的客户，从而确保访问质量。

![](https://user-gold-cdn.xitu.io/2017/12/5/1602472bdfc9fee2?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### SLB

> 负载均衡（Server Load Balancer，简称 SLB）是一种网络负载均衡服务，针对阿里云弹性计算平台而设计，在系统架构、系统安全及性能，扩展，兼容性设计上都充分考虑了弹性计算平台云服务器使用特点和特定的业务场景。

### 4 层负载均衡

![](https://user-gold-cdn.xitu.io/2017/12/5/1602472bb99aaee3?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

在 iso 模型中的传输层（包的转发）

### 7 层负载均衡

![](https://user-gold-cdn.xitu.io/2017/12/5/1602472bae60a7d0?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

在应用层实现

#### nginx 实现的负载均衡（7 层）

![](https://user-gold-cdn.xitu.io/2017/12/5/1602472bbeda916a?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

#### 配置

```
syntax: upstream name{...}
default: -
context:http

```

#### 配置截图

![](https://user-gold-cdn.xitu.io/2017/12/5/1602472be92d4af4?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

#### upstream 举例

```
upstream backend {
	server backend1.example.com weight=5;
    server backend2.example.com:8080;
    server unix:/tmp/backend3;

    server backup1.exmple.com:8080 backup;
    server backup2.example.com:8080 backup;
}

```

#### 后端服务器在负载均衡调度中的状态

| 字段 | 作用 |
| --- | --- |
| down | 当前的 server 暂时不参与负载均衡 |
| backup | 预留的备份服务器 |
| max_fails | 允许请求失败的次数 |
| fail_timeout | 经过 max_fails 失败后，服务暂停的时间 |
| max_conns | 限制最大的接收的连接数 |

#### 配置截图

![](https://user-gold-cdn.xitu.io/2017/12/5/1602472c17478a95?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### 调度算法

| 字段 | 作用 |
| --- | --- |
| 轮询 | 按时间顺序逐一分配到不同的后端服务器 |
| 加权轮询 | weight 值越大，分配到的访问几率越高 |
| ip_hash | 每个请求按访问 ip 的 hash 结果分配，这样来自同一个 ip 的固定访问一个后端服务器 |
| least_conn | 最少链接数，那个机器连接数少就分发 |
| url_hash | 按照访问的 url 的 hash 结果来分配请求，是每个 url 定向到同一个后端服务器 |
| hash 关键值 | hash 自定义的 key |

#### iphash

![](https://user-gold-cdn.xitu.io/2017/12/5/1602472bf4dec7d7?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

#### url——hash

![](https://user-gold-cdn.xitu.io/2017/12/5/1602472bf94c7971?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

#### 配置语法

**url_hash**

```
syntax: hash key [consistent];
default:-
context:upstream
this directive appeared in version 1.7.2

```

**配置截图**

![](https://user-gold-cdn.xitu.io/2017/12/5/1602472c1e740b63?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

## 动态缓存

### 缓存的类型

![](https://user-gold-cdn.xitu.io/2017/12/5/1602472ca3c5d75e?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### 代理缓存

![](https://user-gold-cdn.xitu.io/2017/12/5/1602472c4280a61b?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### proxy_cache 配置语法

```
Syntax: proxy_cache_path path [levels=levels]
Default:-
context:http

```

**开关**

```
syntax: proxy_cache zone | off;
default:proxy_cache off;
context:http, sercer, location

```

**过期周期**

```
syntax: proxy_cache_valid[code] time;
default:-
context:http, sercer, location

```

**缓存的维度**

```
syntax: proxy_cache_key string;
default: proxy_cache_key $scheme$proxy_host$request_uri;
context:http,server,location

```

**配置截图**

![](https://user-gold-cdn.xitu.io/2017/12/5/1602472c2dacd26c?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

level：目录分级 inactive: 不活跃就清理

### 如何清理指定缓存

*   rm-rf 缓存目录内容
*   第三方扩展模块 ngx_cache_purge

### 如何让部分页面不缓存

```
syntax : proxy_no_cache string ...;
default: -;
context:http,server,location

```

**配置截图**

![](https://user-gold-cdn.xitu.io/2017/12/5/1602472cdd01ed8c?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### 大文件分片请求

```
syntax : slice size
default: slice o
context:http,server,location

```

**http_slice_module**

![](https://user-gold-cdn.xitu.io/2017/12/5/1602472c42d25cca?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

**优势**

*   每个子请求收到的数据都会形成一个独立的文件，一个请求断了，其它请求不受到影响

**缺点**

*   当文件很大或者 slice 很小的时候， 可能会导致文件描述符耗尽等情况。

# 深度学习

## 动静分离

> 通过中间件将动态请求和静态请求分离

**为什么**

*   分离资源，减少不必要的请求消耗，减少请求延时

![](https://user-gold-cdn.xitu.io/2017/12/5/1602472c6158c0d6?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

**场景**

![](https://user-gold-cdn.xitu.io/2017/12/5/1602472caacec62b?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

## rewrite 规则

### 应用场景

*   url 访问跳转，支持开发设计
    *   页面跳转、兼容性支持、展示效果等
*   seo 优化
*   维护
    *   后台维护、流量转发
*   安全

### 配置语法

```
syntax: rewrite regex replacement [flag];
default:-
context:server, location,if

```

**维护页面实例**

```
rewrite ^(.*)$ /pages/maintain.html break;

```

### flag

| 字段 | 作用 |
| --- | --- |
| last | 停止 rewrite 检测 |
| break | 停止 rewrite 检测 |
| redirect | 返回 302 临时重定向，地址栏会显示跳转后的地址 |
| permanent | 返回 301 永久重定向，地址栏会显示跳转后的地址（浏览器下次直接访问重定向后的地址 |

![](https://user-gold-cdn.xitu.io/2017/12/5/1602472c79b14fd8?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)![](https://user-gold-cdn.xitu.io/2017/12/5/1602472c474fec46?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### 一些实例

![](https://user-gold-cdn.xitu.io/2017/12/5/1602472cb03c4f7f?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### rewrite 规则优先级

*   执行 server 块的 rewrite 指令
*   执行 location 匹配
*   执行指定的 locaiton 中的 rewrite

## nginx 高级模块

### secure_link_module 模块

*   制定并允许检查请求的链接的真实性以及保护资源免遭未经授权的访问。
*   限制链接生效周期

```
syntax: secure_link expression
default:-
context:server, location,server

```

```
syntax: secure_link_md5 expression
default:-
context:server, location,http

```

**图示**

![](https://user-gold-cdn.xitu.io/2017/12/5/1602472d4f87e890?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)![](https://user-gold-cdn.xitu.io/2017/12/5/1602472d41906339?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

**配置例子**

![](https://user-gold-cdn.xitu.io/2017/12/5/1602472cf12bfc44?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### http_geoip_module 使用场景

> 基于 ip 地址匹配 MaxMind GeoIp 二进制文件，读取 ip 所在地域信息

```
yum install nginx-module-geoip

```

*   区别国内外做 http 访问规则
*   区别国内城市地域做 http 访问规则

**配置截图**

![](https://user-gold-cdn.xitu.io/2017/12/5/1602472cf3a89d56?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### https 服务

> 对传输内容进行加密以及身份验证。

### 为什么需要 https

*   传输数据被中间人盗用，信息泄露
*   数据内容劫持，篡改

### 对称加密和非对称加密

![](https://user-gold-cdn.xitu.io/2017/12/5/1602472d0ca3a2a2?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)![](https://user-gold-cdn.xitu.io/2017/12/5/1602472d26fb740d?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### https 加密协议原理

![](https://user-gold-cdn.xitu.io/2017/12/5/1602472d4480592b?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### 中间人伪造客户端和服务器

![](https://user-gold-cdn.xitu.io/2017/12/5/1602472d7b0425df?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

**如何解决**

![](https://user-gold-cdn.xitu.io/2017/12/5/1602472d403465d1?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### 生成密钥和 CA 证书

**安装 openssl 和 http_ssl_module 模块**

```
#openssl version
openSSL 1.0.1e-fips 11 feb 2013

#nginx -V
--with-http_ssl_module

```

**步骤**

*   生成 key 密钥

```
openssl genrsa -idea -out jesonc.key 1024

```

**结果**

![](https://user-gold-cdn.xitu.io/2017/12/5/1602472d73ce78c1?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

*   生成证书签名请求文件（csr 文件）

```
openssl req -new -key jesonc.key -out jesonc.csr

```

**结果**

![](https://user-gold-cdn.xitu.io/2017/12/5/1602472d8ab45d14?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

*   生成证书签名文件（CA 文件）

```
openssl x509 -req -days 3650 -in jesonc.csr -signkey jesonc.key -out jesonc.crt

```

**结果**

![](https://user-gold-cdn.xitu.io/2017/12/5/1602472da40d57a7?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### 相关实验

#### 配置 docker(443)

```
docker run -d -p 443:443 --name nginx_443 nginx_443:latest /sbin/init
nginx -c /etc/nginx/nginx.conf

```

### conf 文件配置

```
server {
    listen       443;
    server_name  localhost;
    ssl on;
    ssl_certificate /etc/nginx/ssl_key/jesonc.crt;
    ssl_certificate_key /etc/nginx/ssl_key/jesonc.key;

    location / {
        root   /usr/share/nginx/html;
        #limit_conn conn_zone 1;
        #limit_req zone=req_zone;
        # limit_req zone=req_zone burst nodelay;
        # limit_req zone=req_zone burst nodelay;
        index  index.html index.htm;
    }

    # my config
    location /mystatus {
        stub_status;
    }
    #error_page  404              /404.html;

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}

```

### 测试结果

![](https://user-gold-cdn.xitu.io/2017/12/5/1602472dbbf9ad47?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)![](https://user-gold-cdn.xitu.io/2017/12/5/1602472dbd46a04e?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### https 服务优化

*   激活 keepalive 长连接
*   设置 ssl session 缓存

### 配置截图

![](https://user-gold-cdn.xitu.io/2017/12/5/1602472dc008db89?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

