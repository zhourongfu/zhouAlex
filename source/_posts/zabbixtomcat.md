---
title: zabbix 监控服务--------使用JMX监控tomcat
thumbnail: https://www.msbgn.cn/msbgn/tomcat211.png
categories: zabbix
tags: [监控,zabbix]
---

-------------------



###  工作原理

我的环境是centOS6.5.64位
比如：当Zabbix-Server需要知道java应用程序的某项性能的时候，会启动自身的一个Zabbix-JavaPollers进程去连接Zabbix-JavaGateway请求数据，而ZabbixJavagateway收到请求后使用“JMXmanagementAPI”去查询特定的应用程序，而前提是应用程序这端在开启时需要“-Dcom.sun.management.jmxremote”参数来开启JMX远程查询就行。Java程序会启动自身的一个简单的小程序端口12345向Zabbix-JavaGateway提供请求数据。

![ ](https://www.msbgn.cn/msbgn/tomcat211.png)


<!-- more -->

-------------------
 


-------------------
###   开始监控部署  

从上面的原理图中我们可以看出，配置Zabbix监控Java应用程序的关键点在于：配置Zabbix-JavaGateway、让Zabbix-Server能够连接Zabbix-JavaGateway、Tomcat开启JVM远程监控功能等


###  环境说明

安装方法:yum安装
Zabbix版本: Zabbix-2.4.8
JDK版本: jdk1.7.0_71
zabbix-server操作系统: CentOS 6.5 X86_64
zabbix-agent操作系统: CentOS 6.5 X86_64
tomcat:7.0

zabbix的安装这里不做介绍了。

###  服务端zabbix-server端
###  安装JDK
1.rpm安装jdk

```
rpm -ivh jdk-7u71-linux-x64.rpm

```
2.添加jdk的环境变量、查看设置是否成功

```

vi /etc/profile
JAVA_HOME=/usr/java/jdk1.8.0_20
PATH=$PATH:$JAVA_HOME
CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export JAVA_HOME
export PATH
export CLASSPATH
source /etc/profile
java -version

```



###  安装zabbix-java-gateway
1.编译安装

```

这里有编译安装跟yum安装方式，我这里采用的是yum的方式安装
yum install zabbix-java-gateway
systemctl start zabbix-java-gateway
netstat-lntup|grep10052

```

2.修改zabbix-java-gateway配置文件

```

vim /etc/zabbix/zabbix_java_gateway.conf
LISTEN_IP="0.0.0.0"  指定bind的地址,默认值为0.0.0.0
LISTEN_PORT=10052    指定bind的端口,默认值为10052
START_POLLERS=5      指定启动多少进程, 默认为5

```

3.重启zabbix-java-gateway服务


```

/etc/init.d/zabbix-server restart

```


4.修改zabbix_server添加以下文件内容

```

vim /etc/zabbix/zabbix_server.conf
JavaGateway=127.0.0.1   指定Zabbix Java GateWay地址
JavaGatewayPort=10052   指定Zabbix Java GateWay端口，默认为10052
StartJavaPollers=5      指定启动时启动的Java Pollers数量
注意：Zabbix Server/Proxy中的StartJavaPollers要小于等于Zabbix Java GateWay配置文件中的START_POLLERS

``` 

5.重启zabbix_server服务

``` 

/etc/init.d/zabbix-server restart

``` 

6.验证是否启动成功

``` 

ss -tunlp|grep 10052
tcp    LISTEN     0      50                     *:10052                 *:*      users:(("java",10175,12))


``` 

###  zabbix-agentd端
 cd /usr/local/tomcatweixin/bin/
1.修改tomcat catalina.sh

``` 
CATALINA_OPTS="$CATALINA_OPTS -Dcom.sun.management.jmxremote #开启远程监控
  -Dcom.sun.management.jmxremote.port=12345 #远程监控端口
  -Dcom.sun.management.jmxremote.ssl=false #远程ssl验证为false
  -Dcom.sun.management.jmxremote.authenticate=false #关闭权限认证
  -Djava.rmi.server.hostname=填写相应主机客户端的IP" #部署了tomcat的主机地址
``` 

2.下载catalina-jmx-remote.jar，并且放入 cd /usr/local/tomcatweixin/lib/

链接：https://pan.baidu.com/s/1kV4ot5H 密码：mzb3 

3.重启tomcat

``` 
/usr/local/tomcatweixin/bin/shutdown.sh
/usr/local/tomcatweixin/bin/startup.sh

``` 
4.验证是否启动jmx监听成功

``` 
lsof -i :12345

COMMAND  PID USER   FD   TYPE   DEVICE SIZE/OFF NODE NAME
java    2530 root   16u  IPv4 96921409      0t0  TCP *:italk (LISTEN)
``` 

5.导入cmdline-jmxclient-0.10.3.jar包在zabbix-server进行测试

``` 
java -jar cmdline-jmxclient-0.10.3.jar - 192.168.3.18:12346 java.lang:type=Memory NonHeapMemoryUsage
12/05/2016 15:31:16 +0800 org.archive.jmx.Client NonHeapMemoryUsage: 
committed: 54853632
init: 24576000
max: 136314880
used: 54294528
表示连接成功
``` 
###  配置zabbix监控页面端


1.导入监控模板，可以自定义模板，也可以使用zabbix自带的模板，这里我使用的是自定义模板



 ![ ](https://www.msbgn.cn/msbgn/tomcat1.png)

点击improt导入

 ![ ](https://www.msbgn.cn/msbgn/tomcat2.png)
 
选择文件导入，点击improt

 ![ ](https://www.msbgn.cn/msbgn/tomcat3.png)
 
 点击hostz主机，选择对应的主机JMX interfaces，填写IP ，客户端端口12345
 ![ ](https://www.msbgn.cn/msbgn/tomcat4.png)
选择模板，选择之前导入的模板，add,然后保存即可，update

2.过几分钟，等待一会去查看最新数据图形。

 ![ ](https://www.msbgn.cn/msbgn/toamct6.png)
 ![ ](https://www.msbgn.cn/msbgn/tomcat7.png)
 ![ ](https://www.msbgn.cn/msbgn/tomcat8.png)
  ![ ](https://www.msbgn.cn/msbgn/tomcat9.png)
