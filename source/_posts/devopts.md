---
title: devopts
date: 2023-04-17 17:02:00
tags:
---

# docker 集群化

```
$ docker swarm init
```

# 创建网络 public 集群网络

```
$ docker network create -d overlay public
```

# 安装 Portainer

```sh
$ mkdir -p /opt/portainer
$ curl -L https://downloads.portainer.io/portainer-agent-stack.yml -o /opt/portainer/portainer-agent-stack.yml
$ docker stack deploy -c /opt/portainer/portainer-agent-stack.yml portainer
```

# portainer-agent-stack.yml

```yml
version: "3.2"
services:
  agent:
    image: portainer/agent:2.11.1
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /var/lib/docker/volumes:/var/lib/docker/volumes
    networks:
      - agent_network
    deploy:
      mode: global
      placement:
        constraints: [node.platform.os == linux]

  portainer:
    image: portainer/portainer-ce:2.11.1
    command: -H tcp://tasks.agent:9001 --tlsskipverify
    ports:
      - "9443:9443"
      - "9000:9000"
      - "8000:8000"
    volumes:
      - portainer_data:/data
    networks:
      - agent_network
    deploy:
      mode: replicated
      replicas: 1
      placement:
        constraints: [node.role == manager]

networks:
  agent_network:
    driver: overlay
    attachable: true

volumes:
  portainer_data:
```

# 使用 portainer 的

- 创建 mysql 挂载区 infrastructure_mysql

# 创建 infrastructure 的 stack 编排 mysql,redis,gitea,registry:2,drone-server,drone-runner;

```yml
version: "3.4"
services:
  mysql:
    image: mysql:5.7
    command: --default-authentication-plugin=mysql_native_password --character-set-server=utf8mb4 --collation-server=utf8mb4_bin --default-storage-engine=INNODB --max_allowed_packet=256M --innodb_log_file_size=2GB --transaction-isolation=READ-COMMITTED --binlog_format=row
    networks:
      - public
    ports:
      - 3306:3306
    volumes:
      - infrastructure_mysql:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: joyadata
    security_opt:
      - seccomp:unconfined
    deploy:
      placement:
        constraints:
          - node.role==manager
  redis:
    image: redis:5
    deploy:
      placement:
        constraints:
          - node.role==manager
  gitea:
    image: gitea/gitea:1.16.8
    volumes:
      - infrastructure_gitea:/data
    networks:
      - public
    ports:
      - 80:3000
    environment:
      - APP_NAME=joyadata
      - RUN_MODE=prod
      - DOMAIN=192.168.80.81
      - ROOT_URL=http://192.168.80.81
      - DISABLE_SSH=true
      - ENABLE_GZIP=true
      - SSH_PORT=2222
      - DISABLE_REGISTRATION=true
      - REQUIRE_SIGNIN_VIEW=true
      - USER_UID=1000
      - USER_GID=1000
      - DB_TYPE=mysql
      - DB_HOST=mysql:3306
      - DB_NAME=gitea
      - DB_USER=root
      - DB_PASSWD=joyadata
    deploy:
      placement:
        constraints:
          - node.role==manager
  registry:
    image: registry:2
    networks:
      - public
    ports:
      - 5000:5000
    volumes:
      - infrastructure_registry:/var/lib/registry
    deploy:
      placement:
        constraints:
          - node.role==manager
  drone-server:
    image: drone/drone:latest
    environment:
      - DRONE_TLS_AUTOCERT=false
      - DRONE_AGENTS_ENABLED=true
      - DRONE_GITLAB_SERVER=http://192.168.80.81
      - DRONE_GITLAB_CLIENT_ID=40fa9b43d59e55f0647e74d02e936491794c938c735f756e027d0e867cff3417
      - DRONE_GITLAB_CLIENT_SECRET=ee4faa0bf5a4881c2279c0bfe6c0232a44e338bb996ef0451a81049bcf783fd0
      # - DRONE_GITEA_SERVER=http://192.168.90.227
      # - DRONE_GITEA_CLIENT_ID=c74770c2-6cfb-4029-b1e7-90f09da8a86b
      # - DRONE_GITEA_CLIENT_SECRET=spVZFjhiIk7XldgpI7V2qJU5zbSMu27R9rpmewgz9Jm1
      - DRONE_RPC_SECRET=839b342d807efefcefc4c38a4b5fc531
      - DRONE_SERVER_HOST=http://192.168.90.227:7300
      - DRONE_SERVER_PROTO=http
      - DRONE_GIT_ALWAYS_AUTH=true
    networks:
      - public
    ports:
      - 7300:80
    volumes:
      - infrastructure_drone:/data
    deploy:
      placement:
        constraints:
          - node.role==manager
  drone-runner:
    image: drone/drone-runner-docker:latest
    environment:
      - DRONE_RPC_PROTO=http
      - DRONE_RPC_HOST=192.168.90.227:7300
      - DRONE_RPC_SECRET=839b342d807efefcefc4c38a4b5fc531
      - DRONE_RUNNER_CAPACITY=2
      - DOCKER_API_VERSION=1.39
      - DRONE_RUNNER_NAME=AGENT-CCTOMATO-001
    networks:
      - public
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    deploy:
      placement:
        constraints:
          - node.role==manager
  gitlab:
      image: gitlab/gitlab-ce:latest
      volumes:
        - infrastructure_gitlab:/etc/gitlab
        - infrastructure_gitlab_log:/var/log/gitlab
        - infrastructure_gitlab_opt:/var/opt/gitlab
      networks:
        - public
      ports:
        - 80:80
        - 2224:22
      environment:
        GITLAB_OMNIBUS_CONFIG: "external_url 'http://192.168.80.81';gitlab_rails['gitlab_ssh_host'] = '192.168.80.81';gitlab_rails['gitlab_shell_ssh_port'] = 2224"
      deploy:
        placement:
          constraints:
            - node.role==manager
  jenkins:
    # image: jenkinsci/blueocean:1.25.6
    image: jenkins/jenkins
    networks:
      - public
    ports:
      - 8080:8080
      - 50000:50000
    volumes:
      - jenkins_home:/var/jenkins_home
      - /var/run/docker.sock:/var/run/docker.sock
    deploy:
      placement:
        constraints:
          - node.role==manager
networks:
  public:
    external: true
volumes:
  infrastructure_mysql:
    external: true
  infrastructure_gitea:
    external: true
  infrastructure_registry:
    external: true
  infrastructure_drone:
    external: true
  jenkins_home:
    external: true
```

# registry2 配置

- 创建 vi /etc/docker/daemon.json
- 写入 { "insecure-registries":["192.168.80.81:5000"] }
- systemctl daemon-reload
- systemctl restart docker
- docker pull alpine:latest
- docker tag alpine:latest 192.168.90.227:5000/alpine:v1
- docker push 192.168.90.227:5000/alpine:v1
- docker push 192.168.90.227:5000/alpine:v1

# .drone.yml 测试文件

```
kind: pipeline
type: docker
name: default

steps:
- name: greeting
  image: alpine
  commands:
  - echo hello
  - echo world
```

# 前端文件.drone.yml

```yml
kind: pipeline
name: default

steps:
  - name: frontend
    image: node:14.16.1
    commands:
      - npm i --registry=http://192.168.90.126:8081/repository/joyadata-npm-group/
      - npm config set registry http://192.168.90.126:8081/repository/joyadata-npm-group/
      - npm run build
  - name: docker
    image: plugins/docker
    settings:
      repo: 192.168.90.227:5000/project/pdweb
      registry: 192.168.90.227:5000
      insecure: true
      force_tag: true
      tags: latest
```

# 前端配套的 Dockerfile

```Dockerfile
FROM nginx
COPY ./dist /usr/share/nginx/html
COPY ./nginx.conf /etc/nginx
COPY ./default.conf /etc/nginx/conf.d/default.conf
ENTRYPOINT ["nginx","-g","daemon off;"]
```

# default.conf

```conf
server {
  listen       80;
  server_name  127.0.0.1;
  absolute_redirect off;
  # 页面路径
  location /pd/page  {
    alias /usr/share/nginx/html/;
  }
  # 后台接口
  location /pd/ {
    proxy_pass   http://192.168.90.101:9200/pd/;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarder-For $proxy_add_x_forwarded_for;
  }
}

```

# nginx.conf

```conf
user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;
events {
    worker_connections  1024;
}
#stream服务
stream {
    log_format  main  '$remote_addr [$time_local] '
                 '$protocol $status $bytes_sent $bytes_received '
                 '$session_time';
    access_log /var/log/nginx/stream-access.log main;
    include /etc/nginx/conf.d/*.stream;
}
#tcp服务
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

# nginx ssl 配置

去 nginx 官网，下载最新文件
tar -xzvf nginx-1.23.4.tar.gz
cd /nginx-1.23.4

# 校验

./configure --prefix=/usr/local/nginx1.23.4 --with-http_ssl_module

# 编译&&安装

make && make install

# 页面禁止复制，去除
$("\*").css("user-select", "text");
