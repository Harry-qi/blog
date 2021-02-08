---
title: nvm
date: 2021-02-08 17:36:53
tags:
---
nvm安装和使用
nvm是用来切换不同的node版本的

1. 先下载
>  想要安装最新版，可以在GitHub上查看[最新版本](https://github.com/nvm-sh/nvm/releases) 再修改下面命令的版本号
```
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.2/install.sh | bash
```
2. 在.zshrc配置变量
```
vim ~/.zshrc
```
```
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion
```
3. 更新配置 （每次修改.zshrc就要更新不然不生效）
```
source ~/.zshrc
```
4. 查看是否安装成功
   help命令是每个node写的命令行工具都有的命令
```
nvm help 
```
5. 常用命令
   `nvm install <version> [arch]` 安装node，arch默认是系统位数。在 [历史版本](https://nodejs.org/zh-cn/download/releases/) 中查看想要下载的版本
   `nvm list [available]` 显示已安装的列表。list可简化为ls。
   `nvm uninstall <version>`  卸载指定版本node。
   `nvm use [version]`  使用制定版本node。
   自己的使用过程
```
nvm install 14.15.4
nvm install 13.7.0
nvm ls
nvm use 14.15.4
```   
