---
layout:     post
title:      RSICV_MILK zephyr 移植
date:       2024-05-25
author:     肥仔
catalog: true
tags:
    - SOC
    - 个人学习
    - Linux
    - 项目总体Note


--- 

## 芯片资料

https://milkv.io/zh/docs/duo/overview
CV1800B

## 提交链接
https://github.com/plctlab/rvspoc-p2307-zephyr

## 操作系统资料

https://zephyr-doc.readthedocs.io/zh-cn/latest/drivers/drivers.html

https://docs.zephyrproject.org/latest/develop/getting_started/index.html

https://zhuanlan.zhihu.com/p/53421288

https://zhuanlan.zhihu.com/p/609159141#Zephyr%E6%98%AF%E4%BB%80%E4%B9%88?

### 添加新的board

https://docs2.listenai.com/x/7ri3aClW-jv

### qemu 安装

https://blog.csdn.net/qq_36035382/article/details/125308044

### Linux 系统 移植

https://community.milkv.io/t/milk-v-duo-rtos-rt-thread-rt-smart/233

(重要)

https://club.rt-thread.org/ask/article/22fad61ec62a7d8b.html

### rtos 小核运行

https://mp.weixin.qq.com/s/CAW3ME9ZpIW8HItXseLPHQ

https://zhuanlan.zhihu.com/p/663581390

### 官方sdk

https://github.com/milkv-duo/duo-buildroot-sdk


https://www.hw100k.com/play?id=195&chapterId=2310

指路Duo的datasheet: https://forum.sophgo.com/t/about-the-duo-category/41

还有论坛:https://community.milkv.io/

### openocd 调试

https://www.cnblogs.com/zongzi10010/p/10023535.html

https://liangkangnan.gitee.io/2020/03/21/%E6%B7%B1%E5%85%A5%E6%B5%85%E5%87%BARISC-V%E8%B0%83%E8%AF%95/

https://gitee.com/liangkangnan/tinyriscv


## 移植zephy bsp
### 是否需要移植bootelf 20240215
- 移植完成，但启动相关elf 文件之后，系统异常重启


## chatgpt 说明

<img src ="https://daniao2017.github.io/img/in_post/复杂驱动/gt_1.png">

<img src ="https://daniao2017.github.io/img/in_post/复杂驱动/gt_2.png">
