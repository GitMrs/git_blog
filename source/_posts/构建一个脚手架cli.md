---
title: 构建一个脚手架cli 
date: 2020-12-30 16:23:35
categories: "cli"
tags: "cli" 
cover: "/img/node_cli.jpg"
---
# 构建一个脚手
  记录一下自己构建的一个脚手架cli的步骤，使用node的ssh自动拉取一下文件，并给与用户已反馈！！！！

 ## 开始构件准备
  1. 电脑需要有node
  2. 编辑器
 ## 开始
 1. mkdir 创建一个文件夹 xxx-cli
 2. cd 进入该文件
 3. npm init -y 初始化一个项目

      在package.json 添加入口文件
      ```
        "bin": {
          "xxx-cli": "./bin/index"
        }
      ```
     
  4. mdir bin 在根目录下创建一个bin目录,并创建index文件不要后缀名;并添加

      ```
        #指定脚本解释器为node
        #!/usr/bin/env node
        console.log('cli.....')

      ```
  5. 在终端执行，npm link;

      就可以通过npm 建立一个link对于bin/index里面的文件；
      在终端输入xxx就可以打印里面的文件！
  6. 安装依赖
      - commander // node命令行操作
      - chakl // 打印彩色提示
      - clear // 清除
      - download-git-repo // 拉取git模板
      - handlebars // 语义化模板
      - inquirer // 终端交互
      - open // 自动打开浏览器
      - ora // 加载中
      - figlet // 字体样式
  7. 打开bin/index 编写入口
      ```
        #指定脚本解释器为node
        #!/usr/bin/env node
        const program = require('commander'); // 引入node模板终端
        program.version(require('../package.json').version); // 提示版本
        program.usage('<command> [option]') // 命令提示
          .command('create <name>') // create <xxx(为变量)> 
          .description('create project') // 描述
          .action(require('./init.js')) // 加载执行函数
        program.parse(process.argv); // 传入参数
      ```
  8. 建立执行函数 init.js；
 
      执行逻辑，先打印欢迎信息-在选择模板-下载模板
      - 打印欢迎页
        ```
          const {promisify} = require('util') // 引入异步函数
          const figlet = promisify(require('figlet')) // 打印欢迎页
          const clear = require('clear') // 清屏
          const chalk = require('chalk') // 粉笔
          const log = content => console.log(chalk.green(content))
          module.exports = async name => {
          // 打印欢迎画面
            clear()
            const data = await figlet('XXX Welcome')
            log(data)
          }

        ```
      - 选择模板这里使用[inquirer](https://www.npmjs.com/package/inquirer)
        ```
          // 构件模板数据
           const presetPrompt = { 
            name: 'preset', // 名字
            type: 'list', // 类型
            message: `请选择你要获取的模板类型:`, // 描述信息
            choices: [ // 选择项
              {
                name: 'vue2.0+ele', // 展示的名字
                value: 'vue2.0' // 获取的值
              },
              {
                name: 'vue3.0+and-vue',
                value: 'vue3.0'
              },
              {
                name: 'react+and',
                value: 'react'
              }
            ]
          }
          const answers = await inquirer.prompt (presetPrompt) // 将选择模板，对应到终端中
          // answers.preset 就是模板里面的value
          log(`创建${answers.preset}项目：${name}`)
          // 判断preset的值，对应获取下载文件地址
          let url = '';
          switch (answers.preset) {
            case 'vue2.0':
              url = 'xxx';
              break;
            case 'vue3.0':
              url = '';
              break;
            case 'react':
              url = '';
              break;
            default:
              break;
          }
          if (!url) {
            console.log(chalk.red('项目正在构件中。。。'));
            return false;
          } 
          // 下载模板
          let isSuccess = await clone(url, name)

        ```
      - 创建download.js,抛出clone方法,使用[download-git-repo](https://www.npmjs.com/package/download-git-repo)拉取模板
        ```
        const { promisify } = require('util'); // 引入异步
        module.exports.clone = async function (repo, desc) {
          const download = promisify(require('download-git-repo')); // 异步使用download-git-repo
          const ora = require('ora'); // 打印下载进度
          const process = ora(`下载中...`);
          process.start();
          // 容错处理
          try {
            await download(repo, desc);
            process.succeed('下载成功！');
            return true;
          } catch (error) {
            console.log(error);
            process.fail('下载失败！');
            return false;
          }
        }
        ```
      - 下载完成,打印提示信息
        ```
            if (isSuccess) {
              log(chalk.green(`
                拉取完成：
                To get Start：
                =========================
                  cd ${name}
                  npm install 或者 yarn add  下载依赖
                  npm run dev
                =========================
                  `))
                  } else {
                    log(chalk.red(`
                拉取失败：
                请检查网络或稍后再试
                  `))
            }
        ```
        

