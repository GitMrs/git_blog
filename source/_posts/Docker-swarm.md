---
title: Docker-swarm
date: 2023-03-30 16:55:24
tags:
---
基于docker swarm 构建 docker 集群

# 前置条件docker服务器三台，一台ntp时间服务器116
- 116 ntp 时间服务器
- 224 docker 主节点
- 225 docker 从节点
- 226 docker 从节点
# docker swarm 常用命令
- docker swarm init 生成主网络
- docker node ls 查看进入的网络
- docker swarm join-token worker 产看主网络的，进入方式 
# 搭建Portainer 服务部署
```sh
$ mkdir -p /opt/portainer
$ curl -L https://downloads.portainer.io/portainer-agent-stack.yml -o /opt/portainer/portainer-agent-stack.yml
$ docker stack deploy -c /opt/portainer/portainer-agent-stack.yml portainer
```

# 构建自动化docker 构建
- docker swarm init --advertise-addr=192.168.90.224
- 依据上面的提示操作
- docker pull alpine:latest
- docker network create -d overlay  --attachable --subnet=174.18.0.0/24  mynetwork1
- docker run --privileged -itd --name alpine1 --network=mynetwork1 --ip=174.18.0.10  alpine:latest
- docker run --privileged -itd --name alpine1 --network=mynetwork1 --ip=174.18.0.11  alpine:latest
- docker run --privileged -itd --name alpine1 --network=mynetwork1 --ip=174.18.0.12  alpine:latest
