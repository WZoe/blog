---
layout: post
title: 'Hyperledger Cello部署步骤和问题阶段总结（一）'
subtitle: '在进行Hyperledger Cello及Fabric的产品级部署时遇到的一些问题和坑，在这里记录一下。'
date: 2019-04-01
categories: 学习笔记
cover: '/assets/img/cello.png'
tags: Hyperledger Cello 区块链
---
#前言
最近在开发一项用于商业保险联盟用的Permissioned Blockchain System工程。主要用到的是**HyperLedger Cello**。Cello是Hyperledger旗下的项目之一，主要的作用有：
>- 管理区块链网络的生命周期，即创建、开启、停止、删除等；从零开始构建一个BaaS服务
>- 自定义区块链网络配置
>- 支持在多种裸机、虚拟云端、容器等中管理区块链资源
>- 拥有一些扩展组件，如监控、账户管理、健康和分析等功能

简单来说，cello可以帮助我们管理链、管理智能合约，并提供一个图形化的UI方便运维人员操作。

目前cello还是一个不太成熟的项目，网络上可用的资源较少。一般可以通过官方的[文档](https://cello.readthedocs.io/en/latest/)和[答疑区](https://chat.hyperledger.org/channel/cello)找到问题的答案。这系列文章记录一下开发部署过程中的一些步骤和坑用于参考。

这篇文章是截止到master-node部署完成。部署环境为ubuntu16。

#部署步骤
##一、 安装依赖
###1. 安装docker
```
#1) 更新
sudo apt-get update

#2) 安装依赖包
sudo apt-get install apt-transport-https ca-certificates curl software-properties-common

#3) 秘钥和仓库
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

#4) 更新
sudo apt-get update

#5) 安装docker
sudo apt-get install docker-ce

#6) 确认是否安装成功
docker -v
```
###2. 安装docker-compose
```
#1) 安装
sudo curl -L https://github.com/docker/compose/releases/download/1.17.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose

#2) 确认是否安装成功
docker-compose --version
```
###3.启动docker
```
sudo service docker start
```
##二、部署Cello
###1.下载源码
```
git clone https://github.com/hyperledger/cello
```
###2.进入Cello目录下setup（这里是master）
```
cd cello/
make setup-master
```
这一步会花很长时间并拉取多个docker image到本地，若一次没有成功可以重复执行此条命令。
出现下面的log表示安装成功：
```
All Image downloaded 
Checking local mounted database path /opt/cello/mongo...
Local database path /opt/cello/mongo not existed, creating one
Setup done, please logout and login again.
It's safe to run this script repeatedly.
```
###3.重启terminal/连接

```
exit
```
###4.进入Cello目录下启动服务
```
SERVER_PUBLIC_IP=X.X.X.X make start
```
这里的IP是你的主机的IP。首次启动过程将花费几分钟。启动完成后，可以使用下面的命令查看日志：
```
make logs
```
若启动成功，执行`docker ps`和`netstat -tlnp`你将看到启动了的容器和端口：
```
CONTAINER ID        IMAGE                                  COMMAND                  CREATED             STATUS              PORTS                                        NAMES
ee830fc35423        hyperledger/cello-user-dashboard       "bash -c 'cd /usr/..."   3 seconds ago       Up 2 seconds        0.0.0.0:8081->8080/tcp                       cello-user-dashboard
0fe6f30624b3        hyperledger/cello-watchdog             "python watchdog.py"     4 seconds ago       Up 3 seconds                                                     cello-watchdog
d081ef0bbcfd        hyperledger/cello-mongo                "docker-entrypoint..."   4 seconds ago       Up 3 seconds        27017/tcp                                    cello_dashboard_mongo_1
6d6965cc0a8b        hyperledger/cello-nginx                "/bin/bash /tmp/do..."   4 seconds ago       Up 3 seconds        0.0.0.0:80->80/tcp, 0.0.0.0:8080->8080/tcp   cello-nginx
28349b09a597        hyperledger/cello-operator-dashboard   "/bin/sh -c 'if [ ..."   4 seconds ago       Up 3 seconds        8080/tcp                                     cello-operator-dashboard
2e2cc63dfbbd        hyperledger/cello-engine               "python restserver.py"   4 seconds ago       Up 3 seconds        80/tcp                                       cello-engine
600d26e414a6        hyperledger/cello-mongo                "docker-entrypoint..."   4 seconds ago       Up 3 seconds        127.0.0.1:27017-27018->27017-27018/tcp       cello-mongo

```
```
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 127.0.0.1:27017         0.0.0.0:*               LISTEN      13043/docker-proxy- 
tcp        0      0 127.0.0.1:27018         0.0.0.0:*               LISTEN      13026/docker-proxy- 
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      4880/sshd           
tcp6       0      0 :::80                   :::*                    LISTEN      13070/docker-proxy- 
tcp6       0      0 :::8080                 :::*                    LISTEN      13057/docker-proxy- 
tcp6       0      0 :::8081                 :::*                    LISTEN      13622/docker-proxy- 
tcp6       0      0 :::22                   :::*                    LISTEN      4880/sshd   
```
###5.访问管理界面
http://[Master_Node_IP]:8080          默认用户名：admin 密码：pass
![operator dashboard](cello-operator-dashboard.png)

#问题及解决
- **进入Master_Node_IP:8080，显示invalid parameter，logs中显示connectionrefused error errno111 econnrefused socket出问题**

	一般是主机网络错误。启动时记得加SERVER_PUBLIC_IP。
	
- **make setup-master等提示docker相关的错误**
	1. 检查docker-compose是否安装正确
	2. 确认docker是否在运行 `sudo service docker start`
	3. 把user添加到用户组 `sudo gpasswd -a ${USER} docker`
	4. 然后尝试重新`make setup-master`
