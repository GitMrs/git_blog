---
layout: vmware
title: ESXi给欧拉系统扩容
date: 2023-06-26 11:59:06
tags:
---
# ESXi给欧拉系统扩容从100G~200G

## 登录管理平台，找到虚拟机，关闭电源

## 点击编辑 -> 硬盘100G 改为200G（如果报错，请先删除快照）

## 开机执行一下命令
- lsblk [查看磁盘分配]
- fdisk - l [查看现有磁盘空间]
## 创建新的磁盘

```
   # 创建分区命令
    fdisk /dev/sda
    再输入p
    新增分区输入：n
    回车（默认为主分区primary）
    分区号，起始扇区，结束扇区都默认（回车）
    设置分区格式输入：t
    分区号默认（回车）
    Hex 代码为 8e （8e代表Linux LVM分区类型）
    w （写入分区表）
    等待分区完成
```
## 格式化新分区
- lsblk -f [查看分区格式]
- mkfs.ext4 /dev/sda3 [这里新建几就是几]

## 合并分区
```
    # lvm  
    # 创建pv，输入y确认 
    lvm> pvcreate /dev/sda3
    # 这里的名字，可以通过，下面的命令，找到 
    lvm> vgextend centos /dev/sda3
    # 查看一下当前的Volume卷详情   
    lvm> vgdisplay -v
    # 磁盘融合，下面的指定容量和路劲，上面的命令可以看到
    lvm> lvextend -l+136446 /dev/mapper/centos-root
    # 推出
    lvm> quit
    # 系统识别新的存储
    # 如果使用xfs文件系统
    xfs_growfs /dev/root_vg/root
    # 如果使用ext4文件系统
    resize2fs /dev/root_vg/root
```

## df -h 查看应该就成功了
