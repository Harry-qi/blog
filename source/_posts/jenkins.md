---
title: Jenkins搭配gitlab的webhook实现自动化部署
date: 2020-11-11 16:08:22
tags: jenkins
---
## 背景
> 每次写完代码就要进行的机械工作：
> - 提交代码，push到分支。
> - 打包然后把dist目录下的文件用ftp上传到服务器 。  

> 这种机械工作，做久了，就觉得很繁琐，于是我就想做一些自动化部署的工作来简化工作。  

## 公司技术
- 代码使用gitlab管理(版本8.5.7)
- 前端vue 

## 步骤
- [安装jenkins](https://www.jenkins.io/zh/doc/book/installing/#war%E6%96%87%E4%BB%B6)
> 注意点: 
> - 默认是80端口，如果启动jenkins失败，要更换下端口启动，例如`java -jar jenkins.war --httpPort=8001`
> - 安装插件比较慢，如果部分插件没安装成功，可以先跳过，进入主页面
- 新建任务，选择第一个类型的
[![BXW3Yn.png](https://s1.ax1x.com/2020/11/11/BXW3Yn.png)](https://imgchr.com/i/BXW3Yn)
- 源码管理
  - `Credentials`这里我选择的是用户名和密码
  - `Branches to build` 这里我选择的是`master`，就是说`master`的代码变动后就会触发
[![BXWdw4.png](https://s1.ax1x.com/2020/11/11/BXWdw4.png)](https://imgchr.com/i/BXWdw4) 
- 构建触发器
 - 选择`Generic Webhook Trigger`
 - `Token`这里随意定义
 [![BXWwTJ.png](https://s1.ax1x.com/2020/11/11/BXWwTJ.png)](https://imgchr.com/i/BXWwTJ)
- 构建
 - 我选择的是执行`shell`
 [![BXWamF.png](https://s1.ax1x.com/2020/11/11/BXWamF.png)](https://imgchr.com/i/BXWamF)
 - `$WORKSPACE`是jenkins自带的变量，`git push`的代码回退到`$WORKSPACE`目录下，你可以`echo $WORKSPACE`打印下，看下具体在哪里。
 - `cd $WORKSPACE && npm i && npm run build` 进入目录，安装依赖，进行打包。
 - ` cp -R  $WORKSPACE/dist/.  /usr/share/nginx/html` 复制打包好的`dist`目录的文件到`nginx`的`html`目录下。这里`/usr/share/nginx/html`可以根据具体目录进行修改。
- 设置gitlab的webhook
 - 进入项目，选择settings，再选择Web hooks(这里根据gitlab版本不同，找到Web hooks的步骤可能不同)
 [![BXWtyT.png](https://s1.ax1x.com/2020/11/11/BXWtyT.png)](https://imgchr.com/i/BXWtyT)
 - `URL` 填写`http://服务器IP:端口号/generic-webhook-trigger/invoke?token=webhook_token` 这里 token填写就是当时我们在jenkins**构建触发器**步骤填写的`Token`，一定要保持一致。
 - `Trigger` 可以根据实际需求勾选,我只是按照默认的勾选`push`。
- 至此，jenkins和gitlab的设置已经全部设置完成，可以进行git push看看效果了 

## 效果
- 我们`push`代码后，可以看到jenkins已经在自动进行构建了。
[![BXWNOU.png](https://s1.ax1x.com/2020/11/11/BXWNOU.png)](https://imgchr.com/i/BXWNOU)
- 点进去可以看到git commit的信息
[![BXWBk9.png](https://s1.ax1x.com/2020/11/11/BXWBk9.png)](https://imgchr.com/i/BXWBk9)
- 点击`控制台输出`，可以看到在自动打包，并且构建完成，没有报错。 
[![BXWDYR.png](https://s1.ax1x.com/2020/11/11/BXWDYR.png)](https://imgchr.com/i/BXWDYR)

## 2022-01-25更新
这次是需要指定分支推送才触发自动化部署，具体操作如下
![q](https://s2.loli.net/2022/01/25/DaRYV8tpT4QxN2z.jpg)
![q1](https://s2.loli.net/2022/01/25/q53VTItumpnhbOy.jpg)
其实就是在jenkins中加入过滤分支即可，gitlab是不用操作的
## 总结
第一次接触jenkins，搭配gitlab的webhook实现自动打包，效果还是可以的。
