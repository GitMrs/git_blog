---
title:  通过hexo-cli + gitPage 构件博客项目
date: 2020-12-16 11:45:45
categories: "hexo"
tags: "hexo" 
cover: "/img/welcome-cover.jpg"
---
  1. mkdir blog  建立一个文件夹
  ``` 
    mkdir blog
  ```
  2. npm install hexo-cli -g 全局安装hexo-cli
  ```
    cd mkdir
    npm install hexo-cli -g 
  ```
  3. hexo init client 初始化一个hexo项目，项目名为client
  ```
    hexo init client
  ```
  4. hexo new 第一篇文章 编写一个文章，存在source\_posts文件下，生成一个xxx.md
  ```
    hexo new 第一篇文章 
  ```
  5. source\_posts文件就是博客的文章内容，自己可以创建，或者直接删除
  ```
    // 文章格式
    ---
    title: 第一篇文章 // 标题
    date: 2020-12-16 11:45:45 // 时间
    categories: // 文件分类
    tags: // 分类标签
    ---
  ```

  6. 进入[hexo主题官网](https://hexo.io/themes/)

  7. 选择喜欢主题

  8. 点击进入github里面

  9. 在自己的blog里面，git clone https://github.com/Fechin/hexo-theme-diaspora  themes/diaspora 执行该命令！（感谢，[Fechin](https://github.com/Fechin)的无私开源）

  10. 这个主题的具体配置可参考[hexo-theme-diaspora](https://github.com/Fechin/hexo-theme-diaspora)


  11. 首先在git上创建一个自己的仓库，仓库名称为GitMrs(自己的用户名).github.io；

  12. 进入到hexo的配置文件_config.yml配置git信息

  ```
    // 找到这个配置
    deploy
      type: 'git' // 类型
      repo: git@github.com:GitMrs/GitMrs.github.io.git // 仓库地址
      branch: master // 分支
  ```
  13. npm install hexo-deployer-git -s 安装hexo发布模块

  14. hexo generate // 打包单页应用

  15. 执行 hexo deploy // 发布上线（一般两次发布时间需要间隔2分钟！否则会提示没有权限）