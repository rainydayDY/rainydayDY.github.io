---
title: nginx配置
date: 2019-04-10 18:07:56
tags:
- https服务
---
> nginx起本地https服务
<p hidden><!--more--></p>

### 需求

>1. 当线上出现了一个bug，只有在App里才能复现，这时我们直接打开App，使用本地代码调试
>2. 调试验证的时候，App中使用的是https链接，我们本地配置成https，就可以直接在App中调试了

实际上和使用charles的mapRemote是一个作用，区别是线上使用的是https协议，本地是http，没法直接调试。所以可以使用nginx在本地提供https服务。
### 方法

#### 安装nginx
```
brew install nginx
```
#### 修改配置文件

查看配置文件的位置
```
nginx -t        
```
按照如下配置：
```

#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;



    include servers/*;
}
```
#### 创建配置文件
同级servers目录下，创建对应项目的配置文件，比如我现在想对项目名称为expample的项目做映射
创建expample.conf，修改server_name为项目的线上域名，下载证书和密钥到ssl目录下，proxy_pass为本地服务的地址加端口号

```
server {
        listen 80;
        listen 443 ssl;
        server_name 项目的线上域名;
        ssl_session_cache shared:SSL:1m;
        ssl_session_timeout 5m;
        server_tokens off;
        ssl_ciphers HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers on;
        ssl_certificate /usr/local/etc/nginx/ssl/expample.pem;
        ssl_certificate_key /usr/local/etc/nginx/ssl/expample.key;
        location / {
                proxy_pass http://127.0.0.1:3004;
                proxy_set_header Host $host;
                proxy_set_header Remote_Addr $remote_addr;
                proxy_set_header X-REAL-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-Forwarded-Proto $scheme;
        }
}
```
#### 本地配host
```
127.0.0.1 项目的线上域名
```
上述的目的是，将发送到线上的请求发送到本地
#### 启动nginx
配置完成之后，如果之前已经启动了nginx，则在命令行运行 sudo nginx -s reload，如果之前没有启动过nginx，则在命令行运行 sudo nginx

