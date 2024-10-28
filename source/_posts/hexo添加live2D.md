---
title: hexo添加live2D
date: 2020-12-31 13:52:45
tags: hexo
cover: "/img/live2d.jpg"
categories: hexo
---

# 给 hexo 博客添加 live2d，可爱的看板娘！

## 使用官方简单办法

1. 安装 hexo-helper-live2d 插件
   ```
   $ yarn add hexo-helper-live2d
   ```
2. 安装 live2d 模型，模型有肯定可供选择具体可以[查看](https://huaji8.top/post/live2d-plugin-2.0/)

   ```
   $ yarn add live2d-widget-model-shizuku
   ```

3. 在 hexo 配置\_\_config.yml 项中新增 live2d 配置

   ```
   live2d:
       enable: true
       scriptFrom: local
       pluginRootPath: live2dw/
       pluginJsPath: lib/
       pluginModelPath: assets/
       tagMode: false
       debug: false
       model:
         use: live2d-widget-model-shizuku
       display:
         position: right
         width: 300
         height: 600
       mobile:
         show: true

   ```

### 根据[hexo-theme-diaspora](https://github.com/Fechin/hexo-theme-diaspora)这个主题使用大神定制版的[看板娘](https://github.com/stevenjoezhang/live2d-widget)

### 简单用法

1. 找到 themes/diaspora/layout/\_partial/head.ejs 头部引入模板,再 head 标签里面引入字体样式，和 live2d 线上 js 即可

```
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/font-awesome/css/font-awesome.min.css">
  <script src="https://cdn.jsdelivr.net/gh/stevenjoezhang/live2d-widget@latest/autoload.js"></script>

```

2. 重启服务，即可

## 定制用法

1. 找到 themes/diaspora/layout/\_partial/head.ejs 头部引入模板,再 head 标签里面引入字体样式，本地自己引入配置文件
   ```
   <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/font-awesome/css/font-awesome.min.css">
   ```
2. 打开[live2d-widget](https://github.com/stevenjoezhang/live2d-widget),将该项目下载下来，修改名字为 live2d 放到
   /themes/diaspora/source 目录中
3. 打开/themes/diaspora/source/live2d;文件中的 autoload.js,修改文件 live2d_path
   ```
     // 注意：live2d_path 参数应使用绝对路径
     // const live2d_path = "https://cdn.jsdelivr.net/gh/stevenjoezhang/live2d-widget@latest/";
     const live2d_path = "/live2d/"; // 此处修改的名字和文件名保持一致！
   ```
4. 找到 themes/diaspora/layout/\_partial/scripts.ejs 脚本引入模板,在 js 引入列表中添加刚才修改的文件路径,下面 live2d/autoload.js 是新增的！
   ```
     <% if (theme.gitalk.enable){ %><%- js(['//cdn.jsdelivr.net/npm/gitalk@1/dist/gitalk.min.js'])%><%}%>
     <%- js(['//lib.baomitu.com/jquery/1.8.3/jquery.min.js', 'js/plugin.js', 'js/typed.js', 'js/diaspora.js','live2d/autoload.js'])%>
     <%- partial('photoswipe') %>
   ```
5. 重启服务既可以看到了！

## 部分小问题

- 在[hexo-theme-diaspora](https://github.com/Fechin/hexo-theme-diaspora)这个主题中，看板娘被图片挡住！

  修改/themes/diaspora/source/live2d 下的 waifu.css;

  ```
    #waifu {
      bottom: -1000px;
      left: 0;
      line-height: 0;
      margin-bottom: -10px;
      position: fixed;
      transform: translateY(3px);
      transition: transform .3s ease-in-out, bottom 3s ease-in-out;
      z-index: 6; // 原本是1，修改大于6即可
    }
  ```

- 移动端不显示

  修改 themes/diaspora/sourcelive2d 下的 autoload.js 文件夹

  ```
    // 此处判断屏幕宽度，大于768显示；注释掉判断就可全部显示
    // if (screen.width >= 768) {
      Promise.all([
        loadExternalResource(live2d_path + "waifu.css", "css"),
        loadExternalResource(live2d_path + "live2d.min.js", "js"),
        loadExternalResource(live2d_path + "waifu-tips.js", "js")
      ]).then(() => {
        initWidget({
          waifuPath: live2d_path + "waifu-tips.json",
          //apiPath: "https://live2d.fghrsh.net/api/",
          cdnPath: "https://cdn.jsdelivr.net/gh/fghrsh/live2d_api/"
        });
      });
    // }
  ```
