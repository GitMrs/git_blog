---
title: vsCode插件开发
date: 2021-1-4 16:23:35
categories: "cli"
tags: "vscode" 
cover: "/img/vscode.jpg"
mp3: "/music/music_vscode.mp3"
---
# 记录一下vsCode 插件开发的流程

## 环境准备
- node 环境
- npm 一般安装完node就自带了
- vscode插件接口API
- yeoman npm install -g yo
- generator-code npm install -g generator-code
## 开始搭建一个vsCode插件，起始项目
 - 开始搭建一个vsCode插件，起始项目
```
$ yo code
```
 - 文件目录说明
 ```
    node_modules // 依赖
    test // 测试文件
    CHANGELOG.md // 文件变成说明
    extension.js // 入口文件说明
    jsconfig.json // javascript 配置
    README.md // 插件说明
    vsc-extension-quickstart.md // 使用说明
 ```
 - package.json
 ```
    {
  "name": "test", // 项目名
  "displayName": "test", // 发布后商店里的名称
  "description": "", // 描述
  "version": "0.0.1", // 版本
  "engines": {
    "vscode": "^1.52.0"  // 依赖环境
  },
  "categories": [
    "Other" // 插件分类
  ],
  "activationEvents": [ // 插件激活的事件数组
        "onCommand:test.helloWorld" 
  ],
  "main": "./extension.js", // 入口文件
  // 插件最多的配置项
  "contributes": { 
    // 命令
    "commands": [{
            "command": "test.helloWorld",
            "title": "Hello World"

    }],
    // 配置文件
    "configuration": [
      "type": "object",
      "title": "helloWorld",
      "properties": { // 具体的配置参数
        "helloWorld.api":{
            "type": "string",
            "default": "helloWorld",
            "description": "文字说明"
        }
      }
    ]
  },
  // 执行代码
  "scripts": {
    "lint": "eslint .",
    "pretest": "yarn run lint",
    "test": "node ./test/runTest.js"
  },
  // 依赖
  "devDependencies": {
        "@types/vscode": "^1.52.0",
        "@types/glob": "^7.1.3",
        "@types/mocha": "^8.0.4",
        "@types/node": "^12.11.7",
        "eslint": "^7.15.0",
        "glob": "^7.1.6",
        "mocha": "^8.1.3",
        "typescript": "^4.1.2",
        "vscode-test": "^1.4.1"
    }

}
 ```
## 开始编写一个基于hitokoto的插件
- 在package.json 中的contributes中配置
```
    "contributes": {
    "commands": [
      {
        "command": "extension.hitokoto",
        "title": "hitokoto"
      }
    ],
    "configuration": {
      "type": "object",
      "title": "hitokoto",
      "properties": {
        "hitokoto.api": {
          "type": "string",
          "default": "https://v1.hitokoto.cn",
          "description": "hitokotoAPI接口地址"
        },
        "hitokoto.fontColor": {
          "type": "string",
          "default": "#ffffff",
          "description": "字体颜色(状态栏显示时有效)"
        }
      }
    }
  }
```
- 代码编写
```
const vscode = require('vscode');
const axios = require('axios');
/**
* @param {vscode.ExtensionContext} context
*/
// 创建一个Bar存放文章
const hitokotoBarItem = vscode.window.createStatusBarItem(vscode.StatusBarAlignment.Left);

// 激活
function activate(context) {
  // 定义数据结构
  const hitokoto = {
    id: null,
    intervalTimeFlag: null
  };
  // 当ctrl+shift+p 输入时hitokoto时 都重新获取
  let disposable = vscode.commands.registerCommand('extension.hitokoto', function () {
    createHitokoto(hitokoto);
  });
  // 当ctrl+shift+p 输入时refreshHitokoto时 都重新获取
  const getText = vscode.commands.registerCommand('extension.refreshHitokoto', function () {
    createHitokoto(hitokoto);
  })
  // 将其放入context中
  context.subscriptions.push(disposable);
  context.subscriptions.push(getText)
}
// 获取数据
function createHitokoto(hitokoto) {
  const api = vscode.workspace.getConfiguration().get('hitokoto.api');
  axios.get(api).then(res => {
    hitokoto['id'] = res.data.id;
    showHitokoto(res.data);
  }).catch((err) => {
    vscode.window.showInformationMessage('API服务连接失败')
  })
}
// 展示出来
function showHitokoto(data) {
  hitokotoBarItem.color = vscode.workspace.getConfiguration().get('hitokoto.fontColor');
  hitokotoBarItem.text = `${data.hitokoto} --- ${data.from}`;
  hitokotoBarItem.command = `extension.refreshHitokoto`;
  hitokotoBarItem.tooltip = "点击刷新";
  hitokotoBarItem.show();
}
exports.activate = activate;
// 插件被停用的时候被调用
function deactivate() { }

module.exports = {
  activate,
  deactivate
}
```
- 发布详细见[vscode发布说明](https://code.visualstudio.com/api/working-with-extensions/publishing-extension)


