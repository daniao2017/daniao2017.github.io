---
layout:     post
title:      项目申请 
date:       2019-10-06
author:     肥仔
catalog: true
tags:
    - 第九周
--- 
# 第九周
<font color =Coral>测试了arduino对实时ros的通信，主要是通过serial包来进行通讯。</font>
![第九周](https://img-blog.csdnimg.cn/20190813115346705.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RhbmlhbzIwMTc=,size_16,color_FFFFFF,t_70)
## 暴露的问题：
1.	arduino的存储能力有限，在执行端不可能写入太多代码，可能也就是数据的写入与pwm调速的功能。
2.	一块arduino板只能写一个节点，完成几个特定的任务？
![第九周（二）](https://img-blog.csdnimg.cn/2019081311562676.png)
3.小车的运动模型还没有建立好，主要是两轮驱动差分底盘的相关问题。即小车转动的角度与两个轮子的数学关系式的问题。
![第九周（三）](https://img-blog.csdnimg.cn/20190813115803140.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RhbmlhbzIwMTc=,size_16,color_FFFFFF,t_70)
