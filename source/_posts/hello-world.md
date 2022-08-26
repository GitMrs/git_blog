---
title: jenkins 项目配置
cover: "/img/cover.jpg"
---
jenkins 项目配置，默认已经配好nginx；项目目录都已经创建好！

## 开始配置

### 创建一个 item，选择 Freestyle project 模式

### 源码管理 git

- 填写git地址
- 密码
- 指定分支

### 增加构建步骤

- Execute shell
```bash
    rm -rf node_modules
    rm -rf yarn.lock
    yarn
    yarn build
    cp -r ./dist ./home
    tar -zcvf dist.tar.gz ./dist
    mv /home/frontend/xdzq/web/home/ /home/frontend/xdzq/history/home
    mv /home/frontend/xdzq/history/home/home  /home/frontend/xdzq/history/home/home$(date +%s)
    mv ./home /home/frontend/xdzq/web
```

### 推送到别的服务器
- 增加构建后操作步骤 -> Send build artifacts over SSH
- SSH Server
    · Name 选择对应的服务器
- Transfers
    · Source files ->  选择上面打包的dist.tar.gz
    · Remove prefix -> 空
    · Remote directory -> 指定服务器的目录
    · Exec command 服务的指令
    ```bash
    cd /home/frontend/xdzq/web
    mv ./home  /home/frontend/xdzq/history/home/
    mv /home/frontend/xdzq/history/home/home  /home/frontend/xdzq/history/home/home$(date +%s)
    tar -zxvf dist.tar.gz
    rm -rf  dist.tar.gz
    mv ./dist home
    ```

