---
title: Zabbix agent的安装部署
thumbnail: https://www.msbgn.cn/msbgn/zabbix1.png
categories: zabbix
tags: [监控,zabbix]
---
![zabbix agent](https://www.msbgn.cn/msbgn/zabbix1.png)
zabbix server可以通过SNMP，zabbix agent，ping，端口监视等方法提供对远程服务器/网络状态的监视，数据收集等功能，它可以运行在Linux，Solaris，HP-UX，AIX，Free BSD，Open BSD，OS X等平台上。这里讲的是zabbix_agent的功能。
<!-- more -->

##第一、安装环境

* 操作系统：CentOS6.5 X86_64 windows2008
* Yum源：163源
* IP地址：
* DNS：
* 主机名：

##第二、linux环境安装
###1.前期准备
```
[root@knode1 ~]# groupadd  -g 201  zabbix
[root@knode1 ~]# useradd -M -s /sbin/nologin -g zabbix zabbix
[root@knode1 ~]# yum  -y groupinstall  "Development Tools"
```
###2.安装zabbix_agentd
```
[root@knode1 ~]# tar xf zabbix-2.4.7.tar.gz 
[root@knode1 ~]# cd zabbix-2.4.7
[root@knode1 zabbix-2.4.7]# ./configure --prefix=/usr/local/zabbix_agent/ --enable-agent
可以增加--sysconfdir=/etc/zabbix/把配置文件放到指定位置
[root@knode1 zabbix-2.4.7]# make && make install
```
###3.编辑配置文件
```
[root@knode1 zabbix-2.4.7]# vim /usr/local/zabbix_agent/etc/zabbix_agentd.conf
修改为：
Server=10.199.200.15
ListenPort=10050
ServerActive=10.199.200.15
Hostname=knode1.allposs.com
User=zabbix
UnsafeUserParameters=1
```
###4.设置开机启动
```
[root@knode1 zabbix-2.4.7]# ln -s /usr/local/zabbix_agent/sbin/zabbix_agentd /usr/local/sbin/zabbix_agentd
[root@knode1 zabbix-2.4.7]# cp misc/init.d/fedora/core/zabbix_agentd /etc/init.d/
[root@knode1 zabbix-2.4.7]# chmod u+x /etc/init.d/zabbix_agentd 
[root@knode1 zabbix-2.4.7]# chkconfig --add zabbix_agentd
[root@knode1 zabbix-2.4.7]# chkconfig zabbix_agentd on
[root@knode1 zabbix-2.4.7]# service zabbix_agentd start
```
##widows端安装
###1.下载与解压
地址: https://www.zabbix.com/downloads/2.4.0/zabbix_agents_2.4.0.win.zip
解压zabbix_agents_2.4.0.win.zip
conf目录存放是agent配置文件 bin文件存放windows下32位和64位安装程序
###2.配置与安装
####2.1配置zabbix agent相关配置
找到conf下的配置文件 zabbix_agentd.win.conf，修改LogFile、Server、Hostname这三个参数。具体配置如下：
```
LogFile=c:\zabbix\zabbix_agentd.log #日志存放位置
Server=10.199.0.105 #zabbix服务器地址
Hostname=WHUAFFS    #本主机主机名
ServerActive=10.199.0.105 #zabbix server地址
```
####2.2 安装agent
在cmd下执行以下命令：
```
cd c:\zabbix\win64\
zabbix_agentd.exe  -c E:\zabbix\conf\zabbix_agentd.win.conf –i
```
####2.3 启动agent客户端
启动命令如下：
```
zabbix_agentd.exe  -c E:\zabbix\conf\zabbix_agentd.win.conf –s
```
启动的程序将以控制台的形式开启。判定是否启动可以查看启动的日志，或者使用netstat –an查看端口是否监听。




