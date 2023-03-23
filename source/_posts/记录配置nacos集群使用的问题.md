---
title: 记录配置nacos集群使用的问题
date: 2023-03-23 15:45:35
tags: nginx
cover: "/img/11186.jpg"
mp3: "/music/C400002FDEqn0UGW1N.m4a"
---

# nacos集群+nginx代理

nacos之前单个部署,服务读取配置文件没有问题。现在是 nacos 集群又加 nginx 转换发现读取不到信息！需要nginx使用stream 多代理出来 7848，9848，9849；

# 前置条件

- 126 8848 nginx
- 126 18848 nacos
- 101 18848 nacos
- 102 18848 nacos

---

以上 126，101，102 是组成 nacos 集群，126 使用 nginx 负载代理作为 nacos 的入口

### nginx 配置如下

- nginx.conf

```
user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;
events {
    worker_connections  1024;
}
# tcp服务
stream {
    log_format  main  '$remote_addr [$time_local] '
                 '$protocol $status $bytes_sent $bytes_received '
                 '$session_time';
    access_log /var/log/nginx/stream-access.log main;
    include /etc/nginx/conf.d/*.stream; #stream的配置目录
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
     server_tokens off; #隐藏服务器的版本
    tcp_nopush on;
    fastcgi_buffers 8 102400k;
    tcp_nodelay on;  #提高i/o性能，可以设置在http，server，location字段标签里
    client_max_body_size 50m;  #默认是1m，上传文件大小的限制
    client_header_timeout 1000; #默认60s，读取客户端请求头数据的超时时间，若超过这个时间，客户端还没有发送完整的header数据，服务器端将返回"Request timo out(408)"错误。
    client_body_timeout 1000; #默认60s，用于设置读取客户端请求主体的超时时间，仅仅为俩次成功的读取操作之间的一个超时，非请求整个主体数据的超时时间。如果在这个超时时间内，客户端没有发 人和数据，nginx将返回"Request timo out(408)"错误
    send_timeout 1000;  #默认是60s，服务器传送http响应信息到客户端的超时时间，这个超时仅仅为俩次成功握手后的一个超时，非请求整个响应数据的超时时间，如果在这个超时时间内，客户端没有接收任何数据，链接将被关闭。
    gzip on; #开启gzip
    gzip_disable "msie6"; #IE6不使用gzip
    gzip_vary on; #设置为on会在Header里增加 "Vary: Accept-Encoding"
    gzip_proxied any; #代理结果数据的压缩
    gzip_comp_level 6; #gzip压缩比（1~9），越小压缩效果越差，但是越大处理越慢，所以一般取中间值
    gzip_buffers 16 8k; #获取多少内存用于缓存压缩结果
    gzip_http_version 1.1; #识别http协议的版本
    gzip_min_length 1k; #设置允许压缩的页面最小字节数，超过1k的文件会被压缩
    gzip_types text/plain application/javascript application/json text/css; #对特定的MIME类型生效,js和css文件会被压缩
    proxy_read_timeout 1240s; #默认值是 60s ，我们可以设置为240s,或者300s。来应对上游服务器处理请求慢的问题
    keepalive_timeout  65;
    #gzip  on;
  include /etc/nginx/conf.d/*.conf;
}
```

- conf.d 里面的 *.conf 配置

```
upstream myServer {
	server  192.168.90.126:18848 max_fails=1 fail_timeout=10s;
	server  192.168.90.101:18848 max_fails=1 fail_timeout=10s ;
	server  192.168.90.102:18848 max_fails=1 fail_timeout=10s ;
}
server {
         listen       8848;
         server_name  192.168.90.126;
         client_max_body_size 1024m;

        location / {
		proxy_pass http://myServer;
	}
}
```

- conf.d 里面的 *.stream 配置
```
# nacos7848.stream
upstream myServer7848 {
    server  192.168.90.126:17848 max_fails=1 fail_timeout=10s;
    server  192.168.90.101:17848 max_fails=1 fail_timeout=10s;
    server  192.168.90.102:17848 max_fails=1 fail_timeout=10s;
}
server {
    listen       7848;
    proxy_pass myServer7848;
}
# nacos9848.stream
upstream myServer9848 {
    server  192.168.90.126:19848 max_fails=1 fail_timeout=10s;
    server  192.168.90.101:19848 max_fails=1 fail_timeout=10s;
    server  192.168.90.102:19848 max_fails=1 fail_timeout=10s;
}
server {
    listen       9848;
    proxy_pass myServer9848;
}
#nacos9849.stream
upstream myServer9849 {
    server  192.168.90.126:19848 max_fails=1 fail_timeout=10s;
    server  192.168.90.101:19848 max_fails=1 fail_timeout=10s;
    server  192.168.90.102:19848 max_fails=1 fail_timeout=10s;
}
server {
    listen       9849;
    proxy_pass myServer9848;
}
```

## 

