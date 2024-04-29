---
layout:     post
title:      node-red 及emqx的使用
date:       2020-11-20
author:     肥仔
catalog: true
tags:
    - mqtt
    - stm32
    - node-red

--- 
# 前言
主要是完成STM32,OTA在线升级的一些工作，本次是基础篇，包括node-red界面，emqx下发指令等。
## docker & docker-compose安装
[参考daoCloud链接](https://get.daocloud.io/#install-compose)
代码如下:
```
#在 Linux上安装Docker 
curl -sSL https://get.daocloud.io/docker | sh
#安装Docker Compose 
curl -L https://get.daocloud.io/docker/compose/releases/download/1.25.5/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
#测试安装是否成功
docker-compose -v
```
## node-red 和emqx安装
### emqx安装

[EMQX docker安装及运行](https://blog.csdn.net/u011089760/article/details/89892591)

**注意**
- 推荐更换阿里源的docker镜像
- http://ip:18083  默认账号：admin 密码：public

### node-red安装

[搭建自己的物联网平台-NodeRed篇之NodeRed安装部署到Linux云服务器](https://blog.csdn.net/qq_41790078/article/details/108052795)

[node-red官网教程](https://nodered.org/docs/)

## 基本使用

如下图所示

<img src ="https://daniao2017.github.io/img/in_post/mqtt.png">

基本上就是一个mqtt节点,之后则是发送与接收部分。注意`数据格式`即可

## dashboard ui呈现

效果如下,粉丝数过少ing

<img src ="https://daniao2017.github.io/img/in_post/dash_ui.png">


- 注意dashboard是node-red的插件需要自己下载
- **折线图**使用的时候记得要看清楚**输入**