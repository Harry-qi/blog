---
title: linux命令
date: 2020-11-11 16:01:31
tags: linux
---
# 记录下在部署项目时候用到的命令
>  使用linux时间不长，特意记录下平时用到的命令

|  命令   | 含义  |
|  ----  | ----  |
|   ps -ef | grep nginx |  查看进程nginx   |
|  kill -9 <PID>       |  彻底杀死进程   |
|  chmod +x /usr/local/src/jenkins/jenkins.sh   |  设置可执行权限  |
|  rm -rf   |    递归强制删除文件夹  |
|  cp -r /home/packageA/* /home/cp/packageB/  |    递归复制 packageA 下的所有文件拷贝到 packageB  |
|  rm -rf   |    递归强制删除  |
|  cat      |    查看文件  |
|  whereis  |    用于程序名的搜索  |
|  :q!      |    不保存并且强制退出  |
|  :wq      |    保存后离开 vi  |
| scp output.txt root@192.168.1.0:/data/ |  本地拷贝文件到远程服务器 |
| netstat -lnp &#124; grep 88 |   检查端口被哪个进程占用 |
| ssh-copy-id root@192.168.1.0 | 将本机的公钥复制到远程机器的authorized_keys文件中，这样输入一次密码后以后就不用输密码了  |
| tail -fn 100 common-error.log   |    查看日志，100为倒数100行     |
| top       |    查看cpu占用    |