
---
title: Haporxy+Keepalived 配置与安装
thumbnail: https://www.msbgn.cn/msbgn/Haporxy.png
categories: HA
tags: [HA,Keepalived]
---


-------------------



###  简介

-------------------
 软件负载均衡一般通过两种方式来实现：基于操作系统的软负载实现和基于第三方应用的软负载实现。LVS就是基于Linux操作系统实现的一种软负载，HAProxy就是开源的并且基于第三应用实现的软负载。
 
 HAProxy相比LVS的使用要简单很多，功能方面也很丰富。当 前，HAProxy支持两种主要的代理模式:"tcp"也即4层（大多用于邮件服务器、内部协议通信服务器等），和7层（HTTP）。在4层模式 下，HAProxy仅在客户端和服务器之间转发双向流量。7层模式下，HAProxy会分析协议，并且能通过允许、拒绝、交换、增加、修改或者删除请求 (request)或者回应(response)里指定内容来控制协议，这种操作要基于特定规则。




-------------------

###   拓扑图



![Haporxy+Keepalived](https://www.msbgn.cn/msbgn/Haporxy.png)

<!-- more -->


###  正文


####  1. node1配置

1.1 安装keepalived+Haporxy
```
[root@node1 ~]# systemctl stop firewalld.service 
[root@node1 ~]# systemctl disable firewalld.service 
[root@node1 ~]# setsebool -P haproxy_connect_any on
```
这里最好禁用seliunx，我这边是因为selinux出问题，修改配置文件无效。

```
[root@node1 ~]# yum install psmisc.x86_64 haproxy.x86_64 keepalived.x86_64
```

`注:`

  因为需要使用killall命令，所以额外安装了psmisc包

配置keepalived

```
[root@node1 ~]# vim /etc/keepalived/keepalived.conf
```
改为


```
! Configuration File for keepalived
global_defs {  
    notification_email {  
        leekexi@gmail.com #可以多个地址  
    }  
    notification_email_from leekexi@gmail.com   
    smtp_server smtp.gmail.com   
    smtp_connect_timeout 30  
    router_id LVS_MASTER  
}

vrrp_script checkhaproxy
    {
        script "killall -0 haproxy"
        interval 3
        weight -20
    }

vrrp_instance VI_1 {
        interface eno16777728               
        state MASTER                  
        virtual_router_id 50          
        priority 101
        advert_int 1                  
        nopreempt
        debug
    authentication {
           auth_type PASS
           auth_pass 1111
            }

    virtual_ipaddress {
            10.199.200.10/24
    }

    track_script {
            chk_haproxy
    }

   notify_master "/etc/keepalived/scripts/notify.sh master 172.16.10.201"
   notify_backup "/etc/keepalived/scripts/notify.sh backup 172.16.10.201"
   notify_fault "/etc/keepalived/scripts/notify.sh fault 172.16.10.201"
   notify_stop   "/etc/keepalived/scripts/notify.sh stop 172.16.10.201"
}
```


配置相关脚本


```
[root@node2 ~]# vim /etc/keepalived/scripts/notify.sh
```

写入

```
#!/bin/bash
# Author: Jason.Yu <admin@lnmmp.com>
# description: An example of notify script
#
#contact='root@localhost'
#notify() {
#mailsubject="`hostname` to be $1: $2 floating"
#mailbody="`date '+%F %H:%M:%S'`: vrrp transition, `hostname` changed to be $1"
#echo$mailbody | mail -s "$mailsubject"$contact
#}
case "$1" in
    master)
#               notify master $2
                echo "`date +%c` success to get Vip,start-up haproxy." >> /etc/keepalived/scripts/master.log
                systemctl restart haproxy.service
                exit 0
                ;;
    backup)
#               notify backup $2
                # 在节点切换成backup状态时，无需刻意停止haproxy服务，防止chk_maintaince和chk_haproxy多次对haproxy服务操作；
                echo "`date +%c`  host "$2" is backup. " >> /etc/keepalived/scripts/backup.log
                exit 0
                ;;
    fault)
#               notify fault $2# 同上
                kepid=`pidof keepalived`
                if [ $kepid == "" ]
                then
                    echo "`date +%c` no keepalived process id"  >> /etc/keepalived/scripts/fault.log
                else
                    echo "`date +%c` will stop keepalived "  >> /etc/keepalived/scripts/fault.log
                    systemctl stop keepalived.service

                fi
                exit 0
                ;;

    stop)
            pid=`pidof haproxy`
            echo "`date +%c` stop host "$2" is haproxy" >> /etc/keepalived/scripts/stop.log
            kill -9 $pid
            ;;
    * )
            echo'Usage: `basename $0` {master|backup|fault}'
            exit 1
            ;;
    esac
###  2. 安装kibana

```

1.2 配置haproxy

```
[root@node1 ~]# vim /etc/haproxy/haproxy.cfg
```

改为


```
global
    # to have these messages end up in /var/log/haproxy.log you will
    # need to:
    #
    # 1) configure syslog to accept network log events.  This is done
    #    by adding the '-r' option to the SYSLOGD_OPTIONS in
    #    /etc/sysconfig/syslog
    #
    # 2) configure local2 events to go to the /var/log/haproxy.log
    #   file. A line like the following can be added to
    #   /etc/sysconfig/syslog
    #
    #    local2.*                       /var/log/haproxy.log
    #
    log         127.0.0.1 local2

    chroot      /var/lib/haproxy        #修改haproxy的工作目录至指定的目录并在放弃权限之前执行chroot()操作。
    pidfile     /var/run/haproxy.pid    
    maxconn     4000        #设定每个haproxy进程所接受的最大并发连接数。
    user        haproxy
    group       haproxy
    daemon                  #让haproxy以守护进程的方式工作于后台。

    # turn on stats unix socket
    stats socket /var/lib/haproxy/stats

#---------------------------------------------------------------------
# common defaults that all the 'listen' and 'backend' sections will
# use if not designated in their block
#---------------------------------------------------------------------
defaults
    mode                    http        #设定实例的运行模式或协议。
    log                     global      #为每个实例启用事件和流量日志，因此可用于所有区段。
    option                  httplog
    option                  dontlognull
    option http-server-close
    option forwardfor       except 127.0.0.1/8 如果服务器上的应用程序想记录发起请求的客户端的IP地址，需要在HAProxy上配置此选项，这样HAProxy会把客户端的IP信息发送给服务器，在HTTP请求中添加"X-Forwarded-For"字段。
    option                  redispatch      　#当serverId对应的服务器挂掉后，强制定向到其他健康的服务器
    retries                 3
    timeout http-request    10s
    timeout queue           1m
    timeout connect         10s
    timeout client          1m
    timeout server          1m
    timeout http-keep-alive 10s
    timeout check           10s         #心跳检测超时
    maxconn                 3000

listen stats
    mode http
    bind 0.0.0.0:1080
    stats enable
    stats hide-version
    stats uri     /haproxyadmin?stats
    stats realm   Haproxy\ Statistics
    stats auth    admin:admin
    stats admin if TRUE

frontend http-in #指定前端服务
    bind *:80    #指定监听端口
    mode http    
    log global
    option httpclose
    option logasap
    option dontlognull
    capture request  header Host len 20
    capture request  header Referer len 60
    default_backend servers

backend servers
    balance roundrobin  #使用roundrobin算法
    server websrv1 172.16.10.203:80 check maxconn 2000
    server websrv2 172.16.10.204:80 check maxconn 2000
```

测试


```

[root@node1 ~]# haproxy -c -f /etc/haproxy/haproxy.cfg
```

1.3 配置日志


```
[root@node1 ~]# vim /etc/rsyslog.conf

```

增加一条

```
local2.*                                                /var/log/haproxy.log

```

修改日志等级

```
[root@node1 ~]# vim /etc/sysconfig/rsyslog
```

改为

```
# Options for rsyslogd
# Syslogd options are deprecated since rsyslog v3.
# If you want to use them, switch to compatibility mode 2 by "-c 2"
# See rsyslogd(8) for more details
SYSLOGD_OPTIONS="-c 2 -r"

```


###  2. node2配置


2.1 安装keepalived+Haporxy

```
[root@node2 ~]# setsebool -P haproxy_connect_any on
#这里最好禁用seliunx，我这边是因为selinux出问题，修改配置文件无效。
[root@node2 ~]# systemctl disable firewalld.service 
[root@node2 ~]# yum install psmisc.x86_64 haproxy.x86_64 keepalived.x86_64

```
注：因为需要使用killall命令，所以额外安装了psmisc包 配置keepalived


```
[root@node2 ~]# vim /etc/keepalived/keepalived.conf
```

改为


```
! Configuration File for keepalived
global_defs {  
    notification_email {  
    leekexi@gmail.com #可以多个地址  
    }  
    notification_email_from leekexi@gmail.com   
    smtp_server smtp.gmail.com   
    smtp_connect_timeout 30  
    router_id LVS_BACKUP  
}

vrrp_script checkhaproxy
{
    script "killall -0 haproxy"
    interval 3
    weight -20
}

vrrp_instance VI_1 {
    interface eno16777728               
    state BACKUP                  
    virtual_router_id 50          
    priority 100
    advert_int 1                  
    nopreempt
    debug
    authentication {
            auth_type PASS
                    auth_pass 1111
    }

    virtual_ipaddress {
            10.199.200.10/24
    }

    track_script {
            chk_haproxy
    }

   notify_master "/etc/keepalived/scripts/notify.sh master 172.16.10.202"
   notify_backup "/etc/keepalived/scripts/notify.sh backup 172.16.10.202"
   notify_fault "/etc/keepalived/scripts/notify.sh fault 172.16.10.202"
   notify_stop   "/etc/keepalived/scripts/notify.sh stop 172.16.10.202"
}

```

配置相关脚本

```
[root@node2 ~]# vim /etc/keepalived/scripts/notify.sh
```
写入


```
#!/bin/bash
# Author: Jason.Yu <admin@lnmmp.com>
# description: An example of notify script
#
#contact='root@localhost'
#notify() {
#mailsubject="`hostname` to be $1: $2 floating"
#mailbody="`date '+%F %H:%M:%S'`: vrrp transition, `hostname` changed to be $1"
#echo$mailbody | mail -s "$mailsubject"$contact
#}
case "$1" in
    master)
#               notify master $2
                echo "`date +%c` success to get Vip,start-up haproxy." >> /etc/keepalived/scripts/master.log
                systemctl restart haproxy.service
                exit 0
                ;;
    backup)
#               notify backup $2
                # 在节点切换成backup状态时，无需刻意停止haproxy服务，防止chk_maintaince和chk_haproxy多次对haproxy服务操作；
                echo "`date +%c`  host "$2" is backup. " >> /etc/keepalived/scripts/backup.log
                exit 0
                ;;
    fault)
#               notify fault $2# 同上
                kepid=`pidof keepalived`
                if [ $kepid == "" ]
                then
                    echo "`date +%c` no keepalived process id"  >> /etc/keepalived/scripts/fault.log
                else
                    echo "`date +%c` will stop keepalived "  >> /etc/keepalived/scripts/fault.log
                    systemctl stop keepalived.service

                fi
                exit 0
                ;;

    stop)
            pid=`pidof haproxy`
            echo "`date +%c` stop host "$2" is haproxy" >> /etc/keepalived/scripts/stop.log
            kill -9 $pid
            ;;
    * )
            echo'Usage: `basename $0` {master|backup|fault}'
            exit 1
            ;;
    esac

```


2.2 配置haproxy

```
[root@node2 ~]# vim /etc/haproxy/haproxy.cfg
```

改为

```
global
    # to have these messages end up in /var/log/haproxy.log you will
    # need to:
    #
    # 1) configure syslog to accept network log events.  This is done
    #    by adding the '-r' option to the SYSLOGD_OPTIONS in
    #    /etc/sysconfig/syslog
    #
    # 2) configure local2 events to go to the /var/log/haproxy.log
    #   file. A line like the following can be added to
    #   /etc/sysconfig/syslog
    #
    #    local2.*                       /var/log/haproxy.log
    #
    log         127.0.0.1 local2

    chroot      /var/lib/haproxy        #修改haproxy的工作目录至指定的目录并在放弃权限之前执行chroot()操作。
    pidfile     /var/run/haproxy.pid    
    maxconn     4000        #设定每个haproxy进程所接受的最大并发连接数。
    user        haproxy
    group       haproxy
    daemon                  #让haproxy以守护进程的方式工作于后台。

    # turn on stats unix socket
    stats socket /var/lib/haproxy/stats

#---------------------------------------------------------------------
# common defaults that all the 'listen' and 'backend' sections will
# use if not designated in their block
#---------------------------------------------------------------------
defaults
    mode                    http        #设定实例的运行模式或协议。
    log                     global      #为每个实例启用事件和流量日志，因此可用于所有区段。
    option                  httplog
    option                  dontlognull
    option http-server-close
    option forwardfor       except 127.0.0.0/8
    option                  redispatch      　#当serverId对应的服务器挂掉后，强制定向到其他健康的服务器
    retries                 3
    timeout http-request    10s
    timeout queue           1m
    timeout connect         10s
    timeout client          1m
    timeout server          1m
    timeout http-keep-alive 10s
    timeout check           10s         #心跳检测超时
    maxconn                 3000

listen stats
    mode http
    bind 0.0.0.0:1080
    stats enable
    stats hide-version
    stats uri     /haproxyadmin?stats
    stats realm   Haproxy\ Statistics
    stats auth    admin:admin
    stats admin if TRUE

frontend http-in #指定前端服务
    bind *:80    #指定监听端口
    mode http    
    log global
    option httpclose
    option logasap
    option dontlognull
    capture request  header Host len 20
    capture request  header Referer len 60
    default_backend servers

backend servers
    balance roundrobin  #使用roundrobin算法
    server websrv1 172.16.10.203:80 check maxconn 2000
    server websrv2 172.16.10.204:80 check maxconn 2000

```

测试


```
[root@node2 ~]# haproxy -c -f /etc/haproxy/haproxy.cfg
```

2.3 配置日志

```
[root@node2 ~]# vim /etc/rsyslog.conf
```


增加一条

```
local2.*                                                /var/log/haproxy.log
```

修改日志等级


```
[root@node1 ~]# vim /etc/sysconfig/rsyslog

```

改为

```
# Options for rsyslogd
# Syslogd options are deprecated since rsyslog v3.
# If you want to use them, switch to compatibility mode 2 by "-c 2"
# See rsyslogd(8) for more details
SYSLOGD_OPTIONS="-c 2 -r"

```


####  3 node3配置



```
[root@node3 ~]# yum install httpd -y
[root@node3 ~]# firewall-cmd --permanent --add-service=http
[root@node3 ~]# firewall-cmd --reload 
[root@node3 ~]# systemctl enable httpd.service 
[root@node3 ~]# systemctl start httpd.service
[root@node3 ~]# vim /var/www/html/index.html

```

写入

```
<h1>WEB1/10.199.200.203/h1>
```


####  4 node4配置


```
[root@node4 ~]# yum install httpd -y
[root@node4 ~]# firewall-cmd --permanent --add-service=http
[root@node4 ~]# firewall-cmd --reload 
[root@node4 ~]# systemctl enable httpd.service 
[root@node4 ~]# systemctl start httpd.service
[root@node4 ~]# vim /var/www/html/index.html

```

写入

```
<h1>WEB2/10.199.200.204/h1>
```


####  5.测试

* 1.直接用浏览器访问172.16.0.10，看是否网页有变动。
* 2.down掉node1看VIP是否切换到node2上。

##  结束
