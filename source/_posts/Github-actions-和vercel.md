---
title: Github actions 和vercel
date: 2022-02-10 14:39:58
tags:
---
最近在部署一些node服务，不想麻烦公司同事搞服务器来部署了，于是想自己部署。  
之前有了解到`GitHub actions`和`vercel`，但是只是使用过`GitHub actions`来部署博客。这次部署node服务遇到了一些问题，特此记录下。
# 尝试使用`GitHub actions`部署node
1. 发现每次部署会变ip，尝试谷歌如何固定ip，得到[结果](https://stackoverflow.com/questions/66970875/is-it-possible-to-use-static-ip-when-using-github-actions)是需要自己搭建服务器
2. 执行`npm run start`一直在进行中的状态，无法结束这个状态。然后创建仓库进行测试，发现如果有setInterval会一直处于上述状态。自己使用express写个测试接口，也会处于上述状态  

最终决定更换node定时发送消息的方案，决定使用`GitHub actions`的定时服务。
还是发现了问题
1. `GitHub actions`的时间是`GMT`，北京时间比`GMT`时间快**8个小时**
![3](https://s2.loli.net/2022/02/11/nOgDBMlYQmZcub2.jpg)
![2](https://s2.loli.net/2022/02/10/JWQpGFSYNAZel3X.jpg)
2. `GitHub actions` 也**不是立即执行的**，会延迟执行，目前发现慢了14分钟。我这里只是测试一次发现是这个时间，不一定准确。

# 使用`vercel`部署express
经过一般测试，发现几个关键点
1. 根目录必须有`index.js`,相当于是`vercel`的入口文件
2. 根目录必须要有`vercel.json`
``` javascript
{
  "version": 2,
  "builds": [
    {
      "src": "./index.js",
      "use": "@vercel/node"
    }
  ],
  "routes": [
    {
      "src": "/(.*)",
      "dest": "/"
    }
  ]
}
```
用作测试的git仓库地址：https://github.com/Harry-qi/test-github-actions

# 总结
如果是定时任务可以使用github actions自带的schedule执行，不要使用node-schedule  
一次性的函数执行也可以使用github actions部署  

`vercel`适合部署接口服务，也可以当作部署博客之类的