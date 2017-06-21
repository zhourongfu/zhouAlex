---
title: Zabbix 配置Fping
thumbnail: https://www.msbgn.cn/msbgn/Fping.png
categories: zabbix
tags: [监控,zabbix]
---

![zabbix Fping](https://www.msbgn.cn/msbgn/Fping.png)
本文是继之前安装后做的一些相关工作，所有前提在前文提到。这次是安装Fping利用ICMP协议进行判断网络主机是否存活。

[TOC]

##第一、安装环境

* 操作系统：CentOS6.5 X86_64
* Yum源：163源
* IP地址：Server:10.199.255.200
* DNS：10.199.255.15
* 主机名：moniter

###1. 源码包
```
[root@Moniter ~]# wget http://www.fping.org/dist/fping-3.10.tar.gz

```

<!-- more -->

##第三、开始安装
###1.下载Fping
```
[root@Moniter ~]# wget http://www.fping.org/dist/fping-3.10.tar.gz
```
###2.安装Fping

```
[root@Moniter ~]# tar -zxvf fping-3.10.tar.gz 
[root@Moniter ~]# cd fping-3.10
[root@Moniter fping-3.10]# ./configure --prefix=/usr/local/Fping
[root@Moniter fping-3.10]# make&&make install
```
###3.修改fping的权限
```
[root@Moniter ~]# chown root:root /usr/local/Fping/sbin/fping
[root@Moniter ~]# chmod u+s /usr/local/Fping/sbin/fping
```
###4.zabbix配置
```
[root@Moniter fping-3.10]# vim /etc/zabbix/zabbix_server.conf
```
修改 FpingLocation=/usr/local/Fping/sbin/fping

![Fping1](https://www.msbgn.cn/msbgn/Fping1.png)
###5.添加zabbix模板
配置监控项ICMP_Loss

![Fping2](https://www.msbgn.cn/msbgn/Fping2.png)
配置监控项ICMP_Ping

![Fping3](https://www.msbgn.cn/msbgn/Fping3.png)

配置监控项ICMP_Response_Time

![Fping4](https://www.msbgn.cn/msbgn/Fping4.png)

配置触发器ping失效
![Fping5](https://www.msbgn.cn/msbgn/Fping5.png)

{Ping:icmpping.max(#3)}=0的意思是：在ping子集下的icmpping的最大值等于0
配置触发器ping失效后平均值小于0.15

![Fping6](https://www.msbgn.cn/msbgn/Fping6.png)

配置触发器Ping失效后最大时间为20

![Fping7](https://www.msbgn.cn/msbgn/Fping7.png)

三者之间的依赖关系
