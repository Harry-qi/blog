---
title: node-sass升级到sass
date: 2022-02-15 09:47:29
tags:
---
> 最近准备把公司使用的node版本进行升级，我们的UI库是基于ElementUI基础上进行开发的，在升级node过程中遇到一些问题，记录如下

# 尝试1：直接升级node版本
发现一些c++的编译报错，如下图
![1](https://s2.loli.net/2022/02/15/IygMs2aEnQ7Lvtf.jpg)
# 尝试2: 写在`node-sass`，安装`sass`
在网上进行了一些搜索，https://zhuanlan.zhihu.com/p/168018388   
文章中使用了别名，相当于引用的是`node-sass`但是实际使用的是`sass`。  
全局搜索了下`node-sass`，然后发现UI库中还使用了`gulp-sass`，该包内部使用了`node-sass`。所以单单修改别名还是不行的，得把`gulp-sass`也要解决掉。  
# 最终解决方案
在elementUI的issue中看到看到`node-sass`升级到`sass`的讨论，最终看到相关的[pr](https://github.com/ElemeFE/element/commit/d6dedac2e2cb5f1f171f8ce32a9bbcdf6dab82d4)。根据了该Pr进行了升级。
升级了sass以及sass-loader，还需要修改相关语法。  
相关语法主要是 
- /deep/ -> ::v-deep
- 除法`/` 修改为 `math.div`并使用`@use "sass:math";`

参考：
- https://zhuanlan.zhihu.com/p/168018388
- https://github.com/ElemeFE/element/commit/d6dedac2e2cb5f1f171f8ce32a9bbcdf6dab82d4
