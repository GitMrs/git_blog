---
title: 使用docker搭建企业版orcal
date: 2023-03-22 10:23:35
categories: "软件"
tags: "软件" 
cover: "/img/11536.jpg"
mp3: "/music/11536.m4a"
---
## 拉取镜像并启动
```bash
docker run -d --name oracle19c -p 1521:1521 -p 5500:5500 --privileged=true -it -v /opt/soft/db/oracle/:/var/opt/oracle/data -e TZ=Asia/Chongqing registry.cn-beijing.aliyuncs.com/zhouchaoyi/oracle19c:19.3 
```
## 进入镜像内部
```bash
docker exec -it oracle19c /bin/bash 
```
## 修改密码
```bash
./setPassword.sh 123456
```
## 配置
```
grep $ORACLE_HOME /etc/oratab | cut -d: -f1 
export ORACLE_SID=ORCLCDB 
```
## 登录orcal
```
sqlplus / as sysdba 
show pdbs; 
alter session set container=ORCLPDB1;
```
## 重启一下docker 
```
docker restart oracle19c
```
## 部署完成，使用Navicat 连接测试
 - 主机 127.0.0.1
 - 端口 1521
 - 服务名 ORCLPDB1
 - 用户名 system
 - 密码 123456

## 登录orcal管理页面验证
 - https://127.0.0.1:5500/em/
