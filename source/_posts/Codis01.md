---
title: Codis 高可用负载均衡群集的搭建与使用
thumbnail: https://www.msbgn.cn/msbgn/Codis1.png
categories: Codis
tags: [集群,负载,Codis]
---

* `codis-proxy` 提供连接集群`redis`服务的入口
* `codis-redis-group` 实现vredis`读写的水平扩展，高性能
* `codis-redis` 实现`redis`实例服务，通过`codis-ha`实现服务的高可用

## 一、实验环境：
网络拓扑图:
![Codis](https://www.msbgn.cn/msbgn/Codis.png)

<!-- more -->

群集架构图:
![Codis](https://www.msbgn.cn/msbgn/Codis1.png)



* 机器与应用列表

	操作系统：`System version: CentOS 6.7`
	
	`IP`: 192.168.43.130    `hostname`: vmware-130
	`apps`: keepalived + haproxy Master,  zookeeper_1, codis_proxy_1, codis_config, codis_server_master,slave 

    `IP`: 192.168.43.131    `hostname`: vmware-131 
   ` apps`: zookeeper_2, codis_proxy_2,   codis_server_master,slave

   `IP`: 192.168.43.132    `hostname`: vmware-132 
    `apps`: keepalived + haproxy Backup,   zookeeper_3, codis_proxy_3,   codis_server_master,slave

	`VIP`: 192.168.43.100  Port: 45001

`备注`：由于是虚拟测试环境，非生产环境，所以一台机器跑多个应用，如应用于生产环境，只需把应用分开部署到相应机器上即可。

####  1、初始化CentOS系统

*  使用镜像站点配置好的yum安装源配置文件

```
cd /etc/yum.repos.d/
/bin/mv CentOS-Base.repo CentOS-Base.repo.bak
wget https://mirrors.163.com/.help/CentOS6-Base-163.repo
```
* 接下来执行如下命令，检测yum是否正常

```
yum clean all      #清空yum缓存
yum makecache      #建立yum缓存
```

* 然后使用如下命令将系统更新到最新

```
rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY*     #导入签名KEY到RPM
yum  upgrade -y
```

###  2. 关闭不必要的服务

```
for sun in `chkconfig --list|grep 3:on|awk '{print $1}'`;do chkconfig --level 3 $sun off;done
for sun in `chkconfig --list|grep 5:on|awk '{print $1}'`;do chkconfig --level 5 $sun off;done
for sun in crond rsyslog sshd network;do chkconfig --level 3 $sun on;done
for sun in crond rsyslog sshd network;do chkconfig --level 5 $sun on;done
```

###  3. 安装依赖包

```
yum install -y gcc make gcc-c++ automake lrzsz openssl-devel zlib-* bzip2-* readline* zlib-* bzip2-*
```

###  4. 创建软件存放目录

```
mkdir /data/packages
```

###  5. 软件包版本以及下载地址

```
jdk1.8.0_45
zookeeper-3.4.8
go1.6.2
pcre-8.37
haproxy-1.4.22
keepalived-1.4.26

cd /data/packages
wget https://apache.fayea.com/zookeeper/zookeeper-3.4.6/zookeeper-3.4.6.tar.gz
wget https://golangtc.com/static/go/go1.6.2.linux-amd64.tar.gz
wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.37.tar.gz
wget https://www.keepalived.org/software/keepalived-1.2.16.tar.gz
```
通过浏览器自行下载

https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html
https://www.haproxy.org/download/1.4/src/haproxy-1.4.26.tar.gz 

###  6. 重启系统


```
[root@vmware-130 ~]# init 6
```


###  二、部署Zookeeper群集


* 配置hosts文件 ( zookeeper节点机器上配置 )

```
[root@vmware-130 ~]#  vim /etc/hosts
192.168.43.130    vmware-130
192.168.43.131    vmware-131
192.168.43.132    vmware-132
```

* 安装java 坏境  ( zookeeper节点机器上配置 )

```
[root@vmware-130 ~]# cd /data/packages
[root@vmware-130 packages ]# tar zxvf jdk-8u45-linux-x64.tar.gz -C /usr/local
[root@vmware-130 packages ]# cd /usr/local
[root@vmware-130 local ]# ln -s jdk1.8.0_45 java
```

 * 安装Zookeeper  ( zookeeper节点机器上配置 )

```
cd /data/packages
tar zxvf zookeeper-3.4.6.tar.gz -C /usr/local
ln -s zookeeper-3.4.6 zookeeper
cd /usr/local/zookeeper/
```

* 设置环境变量  ( `zookeeper`节点机器上配置 )

```
vim /etc/profile
JAVA_HOME=/usr/local/java
JRE_HOME=$JAVA_HOME/jre
ZOOKEEPER_HOME=/usr/local/zookeeper
JAVA_FONTS=/usr/local/java/jdk1.8.0_102/jre/lib/fonts 
CLASSPATH=$JAVA_HOME/lib:$JAVA_HOME/lib/tools.jar
PATH=$PATH:$JAVA_HOME/bin:$JAVA_HOME/jre/bin:$ZOOKEEPER_HOME/bin 
export JAVA_HOME PATH CLASSPATH JRE_HOME ZOOKEEPER_HOME

#生效环境变量
source /etc/profile
```

*  修改zookeeper配置文件 ( `zookeeper`节点机器上配置 )

```
vi /usr/local/zookeeper/conf/zoo.cfg
tickTime=2000
initLimit=10
syncLimit=5
clientPort=2181
autopurge.snapRetainCount=500 
autopurge.purgeInterval=24
dataDir=/data/zookeeper/data
dataLogDir=/data/zookeeper/logs
server.1=192.168.43.130:2888:3888
server.2=192.168.43.131:2888:3888
server.3=192.168.43.132:2888:3888
#生效环境变量
source /etc/profile
```
* 创建数据目录和日志目录 ( `zookeeper`节点机器上配置 )

```
mkdir -p /data/zookeeper/data
mkdir -p /data/zookeeper/logs
```

* 在zookeeper节点机器上创建myid文件，节点对应id
在43.130机器上创建myid，并设置为1与配置文件`zoo.cfg`里面`server.1`对应。

```
echo "1" > /data/zookeeper/data/myid
```
* 在zookeeper节点机器上创建myid文件，节点对应id
	在43.131机器上创建myid，并设置为1与配置文件zoo.cfg里面server.2对应。

```
echo "2" > /data/zookeeper/data/myid
```

* 在zookeeper节点机器上创建myid文件，节点对应id
在43.132机器上创建myid，并设置为1与配置文件zoo.cfg里面server.3对应。

```
echo "3" > /data/zookeeper/data/myid
```


*  启动zookeeper服务, 以vmware-130为例：

```
[root@vmware-130 ~]# zkServer.sh start
JMX enabled by default
Using config: /usr/local/zookeeper/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
```

* 检查zookeeper所有节点状态

```
[root@vmware-130 ~]# zkServer.sh status
JMX enabled by default
Using config: /usr/local/zookeeper/bin/../conf/zoo.cfg
Mode: follower
 
[root@vmware-131 ~]# zkServer.sh status
JMX enabled by default
Using config: /usr/local/zookeeper/bin/../conf/zoo.cfg
Mode: follower
 
[root@vmware-132 ~]# zkServer.sh status
JMX enabled by default
Using config: /usr/local/zookeeper/bin/../conf/zoo.cfg
Mode: leader
```


###  三、部署Codis群集

* 安装` go `语言环境 ( 所有codis机器上配置 )

```
/data/packages
tar zxvf go1.4.2.linux-amd64.tar.gz -C /usr/local
```


* 添加GO环境变量，其他环境变量不变

```
vim /etc/profile
GOROOT=/usr/local/go
GOPATH=/usr/local/codis
PATH=$PATH:$JAVA_HOME/bin:$JAVA_HOME/jre/bin:$ZOOKEEPER_HOME/bin:$GOROOT/bin
export JAVA_HOME PATH CLASSPATH JRE_HOME ZOOKEEPER_HOME GOROOT GOPATH

source /etc/profile
```

* 安装`codis` ( 所有`codis`机器上配置 )

```
yum install -y git
go get -u -d github.com/CodisLabs/codis
cd $GOPATH/src/github.com/CodisLabs/codis
#执行编译测试脚本，编译go和reids。 
make
make gotest
# 将编译好后，把bin目录和一些脚本复制过去/usr/local/codis目录下：
mkdir -p /usr/local/codis/{logs,conf,scripts}        #创建codis日志，配置文件，脚本目录
mkdir -p /data/codis_server/{logs,conf,data}         #创建codis_server 日志，配置文件，数据目录
cp -rf /usr/local/codis/src/github.com/CodisLabs/codis/bin /usr/local/codis/                         #复制bin目录到自定义的安装目录
cp /usr/local/codis/src/github.com/CodisLabs/codis/config.ini /usr/local/codis/conf/          #复制模板配置文件到安装目录
cp /usr/local/codis/src/github.com/CodisLabs/codis/sample/redis_conf/6381.conf /data/codis_server/conf/            #复制codis_server配置文件到配置目录
cp -rf /usr/local/codis/src/github.com/CodisLabs/codis/sample/usage.md /usr/local/codis/scripts/   #复制模板启动流程文件到脚本目录下
 
```

* 配置`codis_proxy_1`  (` vmware-130` 机器上配置)

```
cd /usr/local/codis
vim config.ini 
zk=vmware-130:2181,vmware-131:2181,vmware-132:2181                              
product=codis                                         
proxy_id=codis_proxy_1                                   
net_timeout=5
dashboard_addr=192.168.43.130:18087         
coordinator=zookeeper
```

配置codis_proxy_1  (` vmware-131` 机器上配置)

```
cd /usr/local/codis
vim config.ini 
zk=vmware-130:2181,vmware-131:2181,vmware-132:2181                              
product=codis                                         
proxy_id=codis_proxy_2                                   
net_timeout=5
dashboard_addr=192.168.43.130:18087         
coordinator=zookeeper
```

配置codis_proxy_1  (` vmware-132` 机器上配置)

```
cd /usr/local/codis
vim config.ini 
zk=vmware-130:2181,vmware-131:2181,vmware-132:2181                              
product=codis                                         
proxy_id=codis_proxy_3                                   
net_timeout=5
dashboard_addr=192.168.43.130:18087         
coordinator=zookeeper
```


* 修改配置文件，启动codis-server服务. ( 所有`codis-server`机器上 )

```
cd /data/codis_server/conf/
mv 6381.conf 6379.conf
vim 6379.conf
修改如下参数: (生产环境，参数适当进行调整)
daemonize yes
pidfile /var/run/redis_6379.pid
port 6379
logfile "/data/codis_server/logs/codis_6379.log"
save 900 1
save 300 10
save 60 10000
dbfilename 6379.rdb
dir /data/codis_server/data
```

复制6380配置文件

```
cp 6379.conf 6380.conf
sed -i 's/6379/6380/g' 6380.conf
```

添加内核参数

```
echo "vm.overcommit_memory = 1" >>  /etc/sysctl.conf
sysctl -p
```

启动codis-server服务  ( 所有codis-server机器上 )

```
/usr/local/codis/bin/codis-server /data/codis_server/conf/6379.conf
/usr/local/codis/bin/codis-server /data/codis_server/conf/6380.conf
```

* 查看一下启动流程：( 以`vmware-130`机器为例 )

```
[root@vmware-130 ~]# cat /usr/local/codis/scripts/usage.md
0. start zookeeper                               //启动zookeeper服务
1. change config items in config.ini   //修改codis配置文件
2. ./start_dashboard.sh                       //启动 dashboard
3. ./start_redis.sh                                //启动redis实例
4. ./add_group.sh                               //添加redis组，一个redis组只能有一个master
5. ./initslot.sh                                     //初始化槽
6. ./start_proxy.sh                              //启动codis_proxy
7. ./set_proxy_online.sh                     //上线proxy项目
8. open browser to https://localhost:18087/admin     //访问管理界面
```
这只是一个参考，有些顺序不是必须的，但启动dashboard前，必须启动zookeeper服务，这是必须的，后面有很多操作，都可以在管理页面完成，例如添加/删除组、数据分片、添加/删除redis实例等
 
 创建dashboard启动脚本。可参考/usr/local/codis/src/github.com/wandoulabs/codis/sample/模板脚本( 只需在一台机器上启动即可。43.130上启动 )

```
[root@vmware-130 ~]# vim /usr/local/codis/scripts/start_dashboard.sh
#!/bin/sh
CODIS_HOME=/usr/local/codis 
nohup $CODIS_HOME/bin/codis-config -c $CODIS_HOME/conf/config.ini -L $CODIS_HOME/logs/dashboard.log dashboard --addr=:18087 --http-log=$CODIS_HOME/logs/requests.log &>/dev/null &
```
启动dashboard


```
[root@vmware-130 ~]# cd /usr/local/codis/scripts/
[root@vmware-130 scripts ]# sh start_dashboard.sh
```

创建初始化槽脚本，可参考`/usr/local/codis/src/github.com/wandoulabs/codis/sample/`模板脚本( 在任一台机器上机器上配置，此环境在`43.130`机器上配置 )

```
[root@vmware-130 ~]# vim /usr/local/codis/scripts/initslot.sh
#!/bin/sh
CODIS_HOME=/usr/local/codis
echo "slots initializing..."
$CODIS_HOME/bin/codis-config -c $CODIS_HOME/conf/config.ini slot init -f
echo "done"
```
执行初始化槽脚本：

```
[root@vmware-130 ~]#  cd /usr/local/codis/scripts
[root@vmware-130 scripts ]# sh initslot.sh 
```

配置`codis-server`,启动`codis-server master` ,` slave `实例 ，以上步骤已经启动，不在描述。


通过管理页面添加组`ID`，为组添加主从实例，一个组里只能有一个`master`，设置`slot`分片数据等。

https://192.168.43.130:18087（最好用Firefox浏览器或者谷歌浏览器，别的浏览器比较坑爹！！！）
如下图所示：
![Codis3](https://www.msbgn.cn/msbgn/Codis3.png)

接下来，依次添加 Server Group 1,2,3 ( 共添加3组 )

![Codis4](https://www.msbgn.cn/msbgn/Codis4.png)

添加好后，图为下:
![Codis5](https://www.msbgn.cn/msbgn/Codis5.png)

接下来添加`codis-server`实例包括`master`,`slave`

![Codis6](https://www.msbgn.cn/msbgn/Codis6.png)

全部添加完成后，如下图所示：

![Codis7](https://www.msbgn.cn/msbgn/Codis7.png)

为组分配Slot(槽)范围

```
group_1   0 - 511
group_2   512 - 1023
group_3   暂时不分配，以下测试中，用来迁移其他组使用。
```

如下图操作所示：group_1  ( 0 - 511 )

![Codis8](https://www.msbgn.cn/msbgn/Codis8.png)

添加成功后，页面会显示success窗口，如下图所示。

![Codis9](https://www.msbgn.cn/msbgn/Codis9.png)

如下图操作所示：group_2  ( 512 - 1023 )

![Codis10](https://www.msbgn.cn/msbgn/Codis10.png)

添加成功后，页面会显示success窗口，如下图所示。

![Codis11](http://www.msbgn.cn/msbgn/Codis11.png)

查看整个Slots分布情况: 选择 Slots Status 或者 右上角那个 Slots 都可以看到分布情况。

![Codis12](https://www.msbgn.cn/msbgn/Codis12.png)

![Codis13](https://www.msbgn.cn/msbgn/Codis13.png)

配置codis-ha服务，主从自动切换。( 随便找个节点机器上配置即可,此环境中在43.131机器上配置 )

```
[root@vmware-131 ~]# go get github.com/ngaut/codis-ha
[root@vmware-131 ~]# cd /usr/local/codis/src/github.com/ngaut
[root@vmware-131 ~]# cp -r codis-ha /usr/local/
[root@vmware-131 ~]# cd /usr/local/codis-ha
[root@vmware-131 codis-ha ]# go build
```

创建启动脚本，启动codis-ha服务

```
[root@vmware-131 ~]# vim /usr/local/codis-ha/start_codis_ha.sh
#!/bin/sh
./codis-ha --codis-config=192.168.43.130:18087 -log-level="info" --productName=vmware-Codis &> ./logs/codis-ha.log &
```

创建日志目录

```
[root@vmware-131 ~]# mkdir /usr/local/codis-ha/logs
[root@vmware-131 ~]# cd /usr/local/codis-ha/
[root@vmware-131 codis-ha ]# sh start_codis_ha.sh
```

修改start_proxy.sh，启动codis-proxy服务 ( 以130机器配置为例，其余codis-proxy只需修改下名称即可。)

```
[root@vmware-130 scripts]# vim /usr/local/codis/scripts/start_proxy.sh 
#!/bin/sh
CODIS_HOME=/usr/local/codis
echo "shut down codis_proxy_1..."
$CODIS_HOME/bin/codis-config -c $CODIS_HOME/conf/config.ini proxy offline codis_proxy_1
echo "done"
echo "start new codis_proxy_1..."
nohup $CODIS_HOME/bin/codis-proxy --log-level error -c $CODIS_HOME/conf/config.ini -L $CODIS_HOME/logs/codis_proxy_1.log  --cpu=8 --addr=0.0.0.0:19000 --http-addr=0.0.0.0:11000 &
echo "done"
echo "sleep 3s"
sleep 3
tail -n 30 $CODIS_HOME/logs/codis_proxy_1.log
[root@vmware-130 scripts]# vim /usr/local/codis/scripts/set_proxy_online.sh 
#!/bin/sh
CODIS_HOME=/usr/local/codis
echo "set codis_proxy_1 online"
$CODIS_HOME/bin/codis-config -c $CODIS_HOME/conf/config.ini proxy online codis_proxy_1
echo "done"
启动codis-proxy
./start_proxy.sh
上线codis_proxy_1
./set_proxy_online.sh
备注：其他codis_proxy只需修改start_proxy.sh和set_proxy_online.sh启动脚本里面的codis_proxy_1名称即可。
```

通过redis-cli客户端直接访问codis-proxy,写入数据，看组里面的master和slave 是否同步。

```
[root@vmware-130 scripts]# redis-cli -p 19000
127.0.0.1:19000> set mike liweizhong
OK
127.0.0.1:19000> set benet lwz
OK
127.0.0.1:19000> exit
[root@vmware-130 scripts]#
```

通过管理界面看到如下图所示：

![Codis14](https://www.msbgn.cn/msbgn/Codis14.png)

codis-server master,slave 同步数据正常，slots槽分片数据正常

接下来在通过codis-proxy去取数据看看

```
[root@vmware-130 scripts]# redis-cli -p 19000
127.0.0.1:19000> get mike
"liweizhong"
127.0.0.1:19000> get benet
"lwz"
127.0.0.1:19000> exit
[root@vmware-130 scripts]# 
```

以下用shell简单的写了个插入redis数据脚本，此脚本会插入20W个key,每运行一次，需要调整INSTANCE_NAME参数里面的数字，才可重新插入新数据。仅供测试使用：

```
[root@vmware-132 scripts]# cat redis-key.sh 
#!/bin/bash
REDISCLI="redis-cli -h 192.168.43.131 -p 19000 -n 0 SET"
ID=1
while [ $ID -le 50000 ]
do
  INSTANCE_NAME="i-2-$ID-VM"
  UUID=`cat /proc/sys/kernel/random/uuid`
  CREATED=`date "+%Y-%m-%d %H:%M:%S"`
  $REDISCLI vm_instance:$ID:instance_name "$INSTANCE_NAME"
  $REDISCLI vm_instance:$ID:uuid "$UUID"
  $REDISCLI vm_instance:$ID:created "$CREATED"
  $REDISCLI vm_instance:$INSTANCE_NAME:id "$ID"
ID=`expr $ID + 1`
done
```

执行插入脚本

```
[root@vmware-132 scripts]# sh redis-key.sh 
```

通过管理界面，我们可以看到如下图所示

![Codis15](https://www.msbgn.cn/msbgn/Codis15.png)

数据插完后，最终如下图所示

![Codis16](https://www.msbgn.cn/msbgn/Codis16.png)


»第三部分部，署Keepalived + haproxy 高可用负载均衡请看下一节，至此第一、二部分已经完成。
