---
title: 开发CLI
date: 2021-02-10 10:21:12
tags:
---
## 1. 先来一个`hello world`

在`package.json`中添加 

```javascript
{
  "bin": {
    "hellocli": "./index.js" 
  }
}
```

作用是申明命令叫做`hellocli`，入口文件是`index.js`。 

我们想用类似`vue create hello-world`这种全局命令，怎么办呢？ 

使用`npm install . -g `或者`npm  link`来把命令注册为全局命令。
![1](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/976bc7f1f671460b9f67d7b863ee486b~tplv-k3u1fbpfcp-zoom-1.image)
在`index.js`中写入

```javascript
#!/usr/bin/env node
console.log('hello myCLI')
```

注意一定要写`#!/usr/bin/env node`不然不生效。现在我们测试下命令，在终端运行`hellocli`
![2](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7e444c648a134e5eaa8922b5eb7794ca~tplv-k3u1fbpfcp-zoom-1.image) 

## 2. 使用命令行交互

2.1 首先安装`npm i commander -S`

> commander: 完整的 node.js 命令行解决方案。更多可以参考[文档](https://github.com/tj/commander.js/blob/master/Readme_zh-CN.md)

在`index.js`中添加

```javascript
/**
 * 捕获命令行输入的参数
 */
program
    .command("create <name>")
    .action((res)=>{
      console.log(res)
    })

program.parse(process.argv);
```

> 解释：
>
> 1. `command`是申明一个命令，即你可以使用`hellocli create`这种命令；
>
> 2. `<name>`是用户输入的名称变量；
>
> 3. `program.parse(process.argv)`是用来解析命令行输入的参数；
>
> 4. `action`可以获取到`<name>`，这里把`<name>`输出

![2](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/479cd118043d449e98f3483499a4fe64~tplv-k3u1fbpfcp-zoom-1.image) 
2.2 安装 `npm i inquirer -S` 

> inquirer: 通用的命令行用户界面集合，用于和用户进行交互。更多参考[文档](https://github.com/SBoudrias/Inquirer.js) 

在index.js里面添加

```javascript
const inquirer = require('inquirer');
let list = [
  {
    type: 'list',
    name: 'single',
    message: '选择其中一个',
    choices: [
      '苹果',
      '香蕉',
      '橘子',
    ]
  },
  {
    type: 'checkbox',
    message: '多选',
    name: 'multiple',
    choices: [
      {
        name: '跑步',
      },
      {
        name: '举哑铃',
      },
      {
        name:"俯卧撑"
      }]
  }
]
program
    .command("create <name>")
    .action(async(res)=>{
      let listRes = await inquirer.prompt(list);
      console.log(listRes)
      console.log(res)
    })
```

> 说明：
>
> 1. `type`可选值有input, number, confirm, list, rawlist, expand, checkbox, password, editor
> 2. `name`这里定义的，会在结果中体现如下图 ![6](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6bad9467b1ba41edbb9db68d73fb5e55~tplv-k3u1fbpfcp-zoom-1.image)
> 3. `message` 是提示语
> 4. `choices`是选项列表的数组
> 5. 更多可以参考[文档](https://github.com/SBoudrias/Inquirer.js)  

效果图如下：
![4](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3b03a642e1e14207ad8b68709a5d81e0~tplv-k3u1fbpfcp-zoom-1.image)
![5](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/393ad61ef3064b41bd2052b1d166f5b9~tplv-k3u1fbpfcp-zoom-1.image)
最终打印结果
![6](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6bad9467b1ba41edbb9db68d73fb5e55~tplv-k3u1fbpfcp-zoom-1.image)
至此，我们可以通过`commander`获取到用户输出的参数，可以通过`inquirer`获取到用户选择的选项。其他的可以根据`node`的一些语法对这些获取的参数进行处理。

可以参考下我写的[CLI](https://www.npmjs.com/package/cmq_cli)