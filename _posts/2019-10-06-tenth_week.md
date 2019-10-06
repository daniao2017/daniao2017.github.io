---
layout:     post
title:      第10周
date:       2019-10-06
author:     肥仔
catalog: true
tags:
    - 项目开发
    
--- 
# 项目陷入了瓶颈期
### 具体体现在如下方面：
**1.	大概有300块钱的东西被烧了（一个stm32和一个驱动板）
2.	控制思路不清晰，小车定位有点问题。**
### 控制思路：
![第十周](https://img-blog.csdnimg.cn/20190813120718766.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RhbmlhbzIwMTc=,size_16,color_FFFFFF,t_70)

<font color=red size =5>大意如下：</font>
![第十周（二）](https://img-blog.csdnimg.cn/20190813121135247.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RhbmlhbzIwMTc=,size_16,color_FFFFFF,t_70)
### 小车定位：
**以完成申请书上的东西为主，可以负重一点点，但重点不是落在负重上。而是小车的机械结构与控制上。**
![第十周（三）](https://img-blog.csdnimg.cn/20190813121547700.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RhbmlhbzIwMTc=,size_16,color_FFFFFF,t_70)
## 一些问题：
<font color=red size=3>a.电机的选型出了问题，100w/2500转，导致转矩为0.4n*m，不能承重？解决方法？</font>
1.增加减速机构，以增大转矩。
2.换电机，底层机械设计重新改进
  个人偏向第二种解决方法：第一种方案可以保留，如果有人想继续做下去的话，思路是增加减速结构，选一个好一点的驱动器（2个500以上？），能读出转速发送给单片机的那种，然后整合到小车上。
<font color=red size=3>b.时间，时间，时间，这个项目很有挑战性，花的时间有点多，需要克服一下。而且马上就期末了，我不可能揽下所有事情，所以会有分工。</font>
c.	干活毅力很重要，现在出现的一些问题都是小问题，不要乱了阵脚。
#### d.	小车的一些性能指标:
1.	最大线速度1m/s 
2.	转矩6n*m以上，可能还要计算一下
3.	预期称重30kg

**e	购买东西：预算3000左右，等上海市那边的钱批下来，先看。**
电池：500  	
树莓派：300-400 // 32g内存，3b（3b+）
直流无刷减速电机*2    300  //AB相编码器
Stm32：200
机械结构 500 
Arduino：200-250 //uno mege2560
Kinetic  300（不知道有发票吗）
Imu 200
语言识别模块 200
硬盘*2  100

