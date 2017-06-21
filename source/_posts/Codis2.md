---
title: Codis 高可用负载均衡群集的搭建与使用2
thumbnail: https://www.msbgn.cn/msbgn/Codis1.png
categories: Codis
tags: [集群,负载,Codis]
---

![Codis](https://www.msbgn.cn/msbgn/Codis1.png)


<!-- more -->

## 一、部署Keepalived + haproxy 高可用负载均衡

安装`haproxy`、`keepalived` (43.130、43.132 机器上操作)

* 1.查看系统内核是否支持` tproxy`

```
[root@vmware-130 ~]# grep TPROXY /boot/config-`uname -r` 
CONFIG_NETFILTER_TPROXY=m
CONFIG_NETFILTER_XT_TARGET_TPROXY=m
内核为2.6.32-220.el6.x86_64，支持TPROXY；
```

* 2.源码安装`pcre-8.01`

```
[root@vmware-130 ~]# rpm -qa|grep pcre
pcre-7.8-6.el6.x86_64
pcre-devel-7.8-6.el6.x86_64
```

系统已经`rpm`形式安装了`pcre`，但安装`haproxy`时，提示找不到`pcre`的库文件，看了`haproxy`的`Makefile`文件，指定`pcre`的为`/usr/local`下，故再源码安装一个`pcre-8.01`，如下(如果不重新安装，可以改`makefile`文件或把库文件软链到`makefile`文件指定的路径)

```
[root@vmware-130 ~]# cd /data/packages
[root@vmware-130 ~]# tar -zxf pcre-8.37.tar.gz && cd pcre-8.37
[root@vmware-130 pcre-8.36 ]# ./configure --disable-shared --with-pic
[root@vmware-130 pcre-8.36 ]# make && make install
```

* 3.安装 haproxy-1.4.22

```
[root@vmware-130 ~]# cd /data/packages
[root@vmware-130 ~]# tar xf haproxy-1.4.26.tar.gz
[root@vmware-130 ~]# cd haproxy-1.4.26
[root@vmware-130 haproxy-1.4.26 ]# make TARGET=linux26 CPU=x86_64 USE_STATIC_PCRE=1 USE_LINUX_TPROXY=1
[root@vmware-130 haproxy-1.4.26 ]# make install target=linux26
[root@vmware-130 haproxy-1.4.26 ]# mkdir -p /usr/local/haproxy/sbin
[root@vmware-130 haproxy-1.4.26 ]# mkdir -p /data/haproxy/{conf,run,logs}
[root@vmware-130 haproxy-1.4.26 ]# ln -s /usr/local/sbin/haproxy /usr/local/haproxy/sbin
```

 * 4.创建haproxy启动脚本
 
```
[root@vmware-130 ~]# vim /etc/init.d/haproxy 
#!/bin/sh
# haproxy
# chkconfig: 35 85 15
# description: HAProxy is a free, very fast and reliable solution \
# offering high availability, load balancing, and \
# proxying for TCP and HTTP-based applications
# processname: haproxy
# config: /data/haproxy/conf/haproxy.cfg
# pidfile: /data/haproxy/run/haproxy.pid

# Source function library.
. /etc/rc.d/init.d/functions

# Source networking configuration.
. /etc/sysconfig/network

# Check that networking is up.
[ "$NETWORKING" = "no" ] && exit 0

config="/data/haproxy/conf/haproxy.cfg"
exec="/usr/local/haproxy/sbin/haproxy"
prog=$(basename $exec)

[ -e /etc/sysconfig/$prog ] && . /etc/sysconfig/$prog

lockfile=/var/lock/subsys/haproxy

check() {
    $exec -c -V -f $config
}

start() {
    $exec -c -q -f $config
    if [ $? -ne 0 ]; then
        echo "Errors in configuration file, check with $prog check."
        return 1
    fi
 
    echo -n $"Starting $prog: "
    # start it up here, usually something like "daemon $exec"
    daemon $exec -D -f $config -p /data/haproxy/run/$prog.pid
    retval=$?
    echo
    [ $retval -eq 0 ] && touch $lockfile
    return $retval
}

stop() {
    echo -n $"Stopping $prog: "
    # stop it here, often "killproc $prog"
    killproc $prog 
    retval=$?
    echo
    [ $retval -eq 0 ] && rm -f $lockfile
    return $retval
}

restart() {
    $exec -c -q -f $config
    if [ $? -ne 0 ]; then
        echo "Errors in configuration file, check with $prog check."
        return 1
    fi
    stop
    start
}

reload() {
    $exec -c -q -f $config
    if [ $? -ne 0 ]; then
        echo "Errors in configuration file, check with $prog check."
        return 1
    fi
    echo -n $"Reloading $prog: "
    $exec -D -f $config -p /data/haproxy/run/$prog.pid -sf $(cat /data/haproxy/run/$prog.pid)
    retval=$?
    echo
    return $retval
}

force_reload() {
    restart
}

fdr_status() {
    status $prog
}

case "$1" in
    start|stop|restart|reload)
        $1
        ;;
    force-reload)
        force_reload
        ;;
    checkconfig)
        check
        ;;
    status)
        fdr_status
        ;;
    condrestart|try-restart)
      [ ! -f $lockfile ] || restart
    ;;
    *)
        echo $"Usage: $0 {start|stop|status|checkconfig|restart|try-restart|reload|force-reload}"
        exit 2
esac
```

备注：此脚本stop的时候有问题，有待解决。

```
#添加haproxy服务#添加haproxy服务
[root@vmware-130 ~]# echo "net.ipv4.ip_nonlocal_bind = 1" >> /etc/rsysctl.conf
[root@vmware-130 ~]# sysctl -p
[root@vmware-130 ~]# chmod 755 /etc/init.d/haproxy
[root@vmware-130 ~]# chkconfig --add haproxy
[root@vmware-130 ~]# chkconfig haproxy on
```

* 5.安装keepalived

```
[root@vmware-130 ~]# cd /data/packages
[root@vmware-130 ~]# tar zxvf keepalived-1.2.16.tar.gz
[root@vmware-130 ~]# cd keepalived-1.2.16
[root@vmware-130 keepalived-1.2.16 ]# ./configure --with-kernel-dir=/usr/src/kernels/2.6.32-504.16.2.el6.x86_64/
\\若/usr/src/kernels/目录下为空，那么安装kernel-headers和kernel-devel包 yum install -y kernel-header kernel-devel
 [root@vmware-130 keepalived-1.2.16 ]# make && make install
```

* 6.配置keepalived,添加keepalived 服务

```
[root@vmware-130 ~]# cp /usr/local/etc/rc.d/init.d/keepalived /etc/rc.d/init.d/
[root@vmware-130 ~]# cp /usr/local/etc/sysconfig/keepalived /etc/sysconfig/
[root@vmware-130 ~]# mkdir -p /data/keepalived/{conf,scripts}
[root@vmware-130 ~]# cp /usr/local/sbin/keepalived /usr/sbin/
[root@vmware-130 ~]# chkconfig --add keepalived
[root@vmware-130 ~]# chkconfig keepalived on
```

 * 7 配置haproxy.cfg配置文件( 43.130, 43.132 配置，haproxy.cfg配置文件完全一样 )

```
[root@vmware-130 ~]# vim /usr/local/haproxy/conf/haproxy.cfg
########### 全局配置 #########
global
log 127.0.0.1 local0 err
chroot /usr/local/haproxy
daemon
nbproc 1
group nobody
user nobody
pidfile /usr/local/haproxy/run/haproxy.pid
ulimit-n 65536
#spread-checks 5m 
#stats timeout 5m
#stats maxconn 100


######## 默认配置 ############
defaults
mode tcp                     #默认的模式mode { tcp|http|health }，tcp是4层，http是7层，health只会返回OK
retries 3                    #两次连接失败就认为是服务器不可用，也可以通过后面设置
option redispatch            #当serverId对应的服务器挂掉后，强制定向到其他健康的服务器
option abortonclose          #当服务器负载很高的时候，自动结束掉当前队列处理比较久的链接
maxconn 32000                #默认的最大连接数
timeout connect 5000ms       #连接超时
timeout client 30000ms       #客户端超时
timeout server 30000ms       #服务器超时
#timeout check 2000          #心跳检测超时
log 127.0.0.1 local3 err     #[err warning info debug]
 
######## proxy 配置#################
listen proxy_status 
bind 0.0.0.0:45001
mode tcp
balance roundrobin
server codis_proxy_1 192.168.43.130:19000 weight 1 maxconn 10000 check inter 10s
server codis_proxy_2 192.168.43.131:19000 weight 1 maxconn 10000 check inter 10s
server codis_proxy_3 192.168.43.132:19000 weight 1 maxconn 10000 check inter 10s
 
######## 统计页面配置 ########
listen admin_stats
bind 0.0.0.0:8099     #监听端口
mode http             #http的7层模式
option httplog        #采用http日志格式
#log 127.0.0.1 local0 err
maxconn 10
stats refresh 30s     #统计页面自动刷新时间
stats uri /stats      #统计页面url
stats realm XingCloud\ Haproxy     #统计页面密码框上提示文本
stats auth admin:admin             #统计页面用户名和密码设置
stats hide-version                 #隐藏统计页面上HAProxy的版本信息
stats admin if TRUE

```

* 8 配置keepalived.conf配置文件 ( 43.130 上配置，43.132备用配置主要修改参数已经标注 )


```
[root@vmware-130 ~]# vim /data/keepalived/conf/keepalived.conf
! Configuration File for keepalived  
  
global_defs {  
   notification_email {  
         lwz_benet@163.com 
   }  
   notification_email_from lwz_benet@1163.com
   smtp_connect_timeout 30  
   smtp_server 127.0.0.1  
   router_id HAProxy_DEVEL 
}  

vrrp_script chk_haproxy {  
    script "killall -0 haproxy"  
    interval 2  
}  

vrrp_instance HAProxy_HA {  
    state BACKUP     
    interface eth0  
    virtual_router_id 80   
    priority 100      #备用为90
    advert_int 2
    nopreempt        #设置不强占，防止业务来回切换。

    authentication {  
        auth_type PASS  
        auth_pass KJj23576hYgu23IP  
    }  
    track_interface {  
       eth0  
    }  
    virtual_ipaddress {  
        192.168.43.100
    }  
    track_script {  
        chk_haproxy  
    }  
  
    #状态通知  
    notify_master "/data/keepalived/scripts/mail_notify.py master"  
    notify_backup "/data/keepalived/scripts/mail_notify.py backup"  
    notify_fault  "/data/keepalived/scripts/mail_notify.py fault"  
}
\\拷贝主上面的keepalived.conf到从上，只需修改priority值参数即可。
 
创建/data/keepalived/scripts/mail_notify.py邮件通知程序：
详细请访问：http://liweizhong.blog.51cto.com/1383716/1639917
\\最后修改下通知信息为英文，中文内容可能会投递失败。
 
# 配置haproxy日志
[root@vmware-130 ~]# vim /etc/rsyslog.d/haproxy.conf
$ModLoad imudp
$UDPServerRun 514
local3.* /data/haproxy/logs/haproxy.log
local0.* /data/haproxy/logs/haproxy.log
[root@vmware-130 ~]# vim /etc/sysconfig/rsyslog
SYSLOGD_OPTIONS="-c 2 -r -m 0"
[root@vmware-130 ~]# service rsyslog restart
```

* 8启动haproxy、keepalived服务。(先启动两个haproxy服务，然后在依次启动master、backup上的keepalived服务)

```
[root@vmware-130 ~]# service haproxy start   ( 先启动 haproxy 服务 )
[root@vmware-130 ~]# service keepalived start
```

* 10测试redis-cli客户端访问

```
[root@vmware-130 ~]# redis-cli -h 192.168.43.130 -p 45001
```
备注：redis-cli 命令，codis里面是没有的，我是安装redis服务的，只是用codis而已。

####  到这里，整个架构已经全部部署完成啦！！！


##  二、Codis 群集架构故障测试


* codis-server的HA安装

```
export GOPATH=/data/go
/usr/local/go/bin/go get github.com/ngaut/codis-ha
cd  /data/go/src/github.com/ngaut/codis-ha
go build
cp codis-ha /data/go/src/github.com/wandoulabs/codis/bin/
使用方法：
codis-ha --codis-config=dashboard地址:18087 --productName=集群项目名称

* 10测试redis-cli客户端访问
```

* 使用supervisord管理codis-ha进程

```
yum -y install supervisord
/etc/supervisord.conf中添加如下内容：
[program:codis-ha]
autorestart = True
stopwaitsecs = 10
startsecs = 1
stopsignal = QUIT
command = /data/go/src/github.com/wandoulabs/codis/bin/codis-ha --codis-config=10.10.32.17:18087 --productName=zh_news
user = root
startretries = 3
autostart = True
exitcodes = 0,2

/etc/init.d/supervisord start
chkconfig supervisord  on
```
1.停止任意zookeeper节点，检查codis-proxy，dashboard是否正常.


```
[root@vmware-132 scripts]# zkServer.sh status
JMX enabled by default
Using config: /usr/local/zookeeper/bin/../conf/zoo.cfg
Mode: leader                                     \\目前此节点提供服务
[root@vmware-132 scripts]# zkServer.sh stop           \\停止此服务，模拟leader挂掉。
JMX enabled by default
Using config: /usr/local/zookeeper/bin/../conf/zoo.cfg
Stopping zookeeper ... STOPPED

检查zookeeper其他节点是否重新选取 leader。
[root@vmware-131 ~]# zkServer.sh status
JMX enabled by default
Using config: /usr/local/zookeeper/bin/../conf/zoo.cfg
Mode: leader                                     \\可以看到，vmware-131已经选举为leader.
[root@vmware-130 ~]# zkServer.sh status
JMX enabled by default
Using config: /usr/local/zookeeper/bin/../conf/zoo.cfg
Mode: follower
 
redis客户端是否能正常访问到codis-proxy。
[root@vmware-130 logs]# redis-cli -h 192.168.43.100 -p 45001
192.168.43.100:45001> get mike
"liweizhong"
192.168.43.100:45001> get benet
"lwz"
192.168.43.100:45001> get id
"27"
192.168.43.100:45001> exit
[root@vmware-130 logs]#
 
dashboard管理界面是否正常。
打开浏览器，访问 http://192.168.43.130:18087/admin/

```
2.停止group master,检查group slave是否自动切换主

接下来，我们开始来模拟vmware-130机器上的codis-server master 6379端口挂掉
![Codis](https://www.msbgn.cn/msbgn/Codis22.png)

停止codis-master后，检查codis-ha日志输出如下信息:
![Codis](https://www.msbgn.cn/msbgn/Codis23.png)


打开管理界面，查看到如下信息：

![Codis](https://www.msbgn.cn/msbgn/Codis24.png)

客户端写入新数据，切换后的主是否有新key增加。


```
[root@vmware-130 ~]# redis-cli -h 192.168.43.100 -p 45001
192.168.43.100:45001> set abc 123
OK
192.168.43.100:45001> set def 456
OK
192.168.43.100:45001> get abc
"123"
192.168.43.100:45001> get def
"456"
192.168.43.100:45001> exit
```

打开管理界面，查看到keys增加两个。

![Codis](https://www.msbgn.cn/msbgn/Codis25.png)

接下来我们恢复vmware-130 codis-server 6379

```
[root@vmware-130 ~]# /usr/local/codis/bin/codis-server /data/codis_server/conf/6379.conf 
[root@vmware-130 ~]# ps -ef |grep codis-server
root      2121     1  0 Apr30 ?        00:02:15 /usr/local/codis/bin/codis-server *:6380                           
root      7470     1 21 16:58 ?        00:00:00 /usr/local/codis/bin/codis-server *:6379                           
root      7476  1662  0 16:58 pts/0    00:00:00 grep codis-server
```

这时，我们在管理界面看到如下情况：

![Codis](https://www.msbgn.cn/msbgn/Codis26.png)


备注：当master挂掉时候，redis-ha检测到自动将slave切换为master,但是master恢复后，仍为offline，需要将其删除在添加，就可以成为slave.

按备注那样，我们需要将原来的master 6379先删除，然后再次添加。操作完成后，如下图所示

![Codis](https://www.msbgn.cn/msbgn/Codis27.png)


3.通过dashboard管理界面添加codis-server组，在线迁移、扩展等。

添加新组，添加master,slave . \\此步省略，之前已经添加好group_3
通过Migrate Slot(s)选项，我们来迁移group_1组到group_3组：
为了模拟迁移是否会影响到业务，我在一台机器开启插入数据脚本，
[root@vmware-132 scripts]# sh redis-key.sh      \\脚本里面连接codis群集请修改为虚拟IP.
现在又客户端在实时插入数据，接下来通过管理界面操作步骤如下：

![Codis](https://www.msbgn.cn/msbgn/Codis28.png)




目前客户端在不断插入新数据，后端我们又在迁移组数据，那么我们现在在来get数据看看是否正常。


```
[root@vmware-130 ~]# redis-cli -h 192.168.43.100 -p 45001
192.168.43.100:45001> get abc
"123"
192.168.43.100:45001> get benet
"lwz"
192.168.43.100:45001> get mike
"liweizhong"
192.168.43.100:45001> get id
"27"
192.168.43.100:45001> exit
```

可以看到后端在迁移数据，对业务访问不受影响。这点非常赞。
迁移完成后，如下图所示：
![Codis](https://www.msbgn.cn/msbgn/Codis29.png)


4.模拟codis-proxy节点挂掉，看haproxy服务是否会剔除节点。
我们仍继续用脚本插入数据，然后停止vmware-131上面的codis-proxy服务。
```
[root@vmware-132 scripts]# sh redis-key.sh 
```

![Codis](https://www.msbgn.cn/msbgn/Codis30.png)

```
[root@vmware-130 ~]# redis-cli -h 192.168.43.100 -p 45001
192.168.43.100:45001> get mike
"liweizhong"
192.168.43.100:45001> get benet
"lwz"
192.168.43.100:45001> get id
"27"
192.168.43.100:45001> get abc
"123"
```

codis-proxy代理节点挂掉一个，haproxy自动剔除此节点，插入数据脚本由于之前连接的socket挂掉，会中断重新连接新的socket.  业务正常访问，以下为haproxy监控页面信息：

![Codis](https://www.msbgn.cn/msbgn/Codis31.png)

codis管理界面我们可以看到codis_proxy_2已经没有显示出来：

![Codis](https://www.msbgn.cn/msbgn/Codis32.png)

 
当codis_proxy_2恢复的时候，haproxy又自动加入此节点，并正常提供服务。

![Codis](https://www.msbgn.cn/msbgn/Codis33.png)

codis管理界面又正常显示codis_proxy_2节点。


![Codis](https://www.msbgn.cn/msbgn/Codis34.png)

5.keepalived+haproxy群集故障测试
一、停止haproxy-master ,  观察/var/log/message日志
照样启动redis-key.sh插入数据脚本

```
[root@vmware-132 scripts]# sh redis-key.sh 
停止 haproxy master
```

![Codis](https://www.msbgn.cn/msbgn/Codis35.png)

以下为截取到的日志信息:
keepalived master 130  tail -f /var/log/message
![Codis](https://www.msbgn.cn/msbgn/Codis36.png)


插入数据脚本会出现中断，然后又正常插入数据。
虚拟IP出现一次掉包，然后马上恢复了。
![Codis](https://www.msbgn.cn/msbgn/Codis37.png)


二、恢复haproxy-master, 观察/var/log/message日志，看是否被抢占，正常情况主haproxy恢复后，不会进行切换，防止业务来回切换。。。
接下来我们恢复haproxy-master

```
[root@vmware-130 logs]# service haproxy start
Starting haproxy:                                          [  OK  ]
tail -f /var/log/message
```
![Codis](https://www.msbgn.cn/msbgn/Codis38.png)

以上截图我们可以看到恢复haproxy-master后，VIP不会进行漂移，keepalived进入BACKUP状态，这是因为设置了nopreempt参数，不抢占，防止业务来回切换。。。
 
三、停止haproxy-backup, 观察 /var/log/message日志，是否进行切换。
以上我们模拟了haproxy-master故障和恢复，现在我们再次模拟现在的haproxy-master也就是原先的haproxy-backup.
模拟haproxy-backup进程挂掉：
 
keepalived master 130  tail -f /var/log/message
 
keepalived backup 132  tail -f /var/log/message
VIP进行了漂移，keepalived也切换身份。
再次恢复haproxy-backup ,VIP不进行漂移，与以上类似，不在描述。


四、模拟keepalived进程挂掉
 
`keepalived-master `挂掉，`keepalived`主备切换，VIP进行漂移。
当`keepalived-master`恢复时，直接进入BACKUP状态，不进行主备切换，VIP不漂移。

»至此已经完成。
