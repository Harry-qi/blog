---
title: Micro-Frontends
date: 2021-02-08 17:28:54
tags:
---
# 微前端 （Micro Frontends）
## 1. 什么是微前端？

将 Web 应用由单一的单体应用转变为多个小型前端应用聚合为一的应用。各个前端应用可以独立运行、独立开发、独立部署。（概念类似于后端的微服务）

更多可以参考[链接](https://micro-frontends.org)

## 2. 能解决什么问题？
- 多种版本多个技术共存（例如`Vue1.0`、`Vue2.6`、`Vue3.0`、`React`、`jQuery`在一个应用中存在。
- 整合老系统
> 场景：例如一个上古时期的系统用`jQuery`+`jsp`开发，代码量很多。现在你接手了，你想用`Vue`/`React`来写，打算重构？这时候就可以用微前端来解决来。
## 3. 优点
- 独立开发，独立部署
- 框架无关
- 增量升级
- 独立运行时
## 4. 微前端的解决方案

自己能想到的是iframe嵌套
其他解决方案：
- Nginx路由转发
- Web Components
- 组合式应用路由分发

  组合式应用路由分发方案改造成本中等并且能满足大部分需求，也不影响各前端应用的体验，是当下各个业务普遍采用的一种方案
## 5. 微前端框架

- [single-spa](https://github.com/single-spa/single-spa) 最早的微前端框架，兼容多种前端技术栈。
- [qiankun](https://github.com/umijs/qiankun) 基于Single-Spa，阿里系开源微前端框架。**(推荐)**
- [icestark](https://github.com/ice-lab/icestark) 阿里飞冰微前端框架，兼容多种前端技术栈。

## 6. 微前端实践
自己参考`qiankun`官网写的[demo](https://github.com/Harry-qi/learn-qiankun)  

### 参考链接
- [记一次 微前端 qiankun 项目 实践 !!! 防踩坑指南](https://juejin.cn/post/6844904142880800775)
- [qiankun-demo](https://github.com/kristianmandrup/qiankun-demo)
- [微前端-最容易看懂的微前端知识](https://juejin.cn/post/6844904162509979662)

