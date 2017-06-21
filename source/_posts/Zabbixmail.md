---
title: zabbix邮件报警设置
thumbnail: https://www.msbgn.cn/msbgn/zabbixmail1.png
categories: zabbix
tags: [监控,zabbix]
---
![zabbixmaildex](https://www.msbgn.cn/msbgn/zabbixmail1.png)

邮件报警有两种情况：
1、Zabbix服务端只是单纯的发送报警邮件到指定邮箱，发送报警邮件的这个邮箱账号是Zabbix服务端的本地邮箱账号（例如：root@localhost)
2、使用一个可以在互联网上正常收发邮件的邮箱账号（例如：xxx@163.com），通过在Zabbix服务端中设置，使其能够发送报警邮件到指定邮箱。（这里我们使用第二种）

<!-- more -->

-------------------



## 1、使用外部邮箱账号发送报警邮件设置

一、关闭sendmail或者postfix
```
service sendmail stop #关闭
chkconfig sendmail off #禁止开机启动
service postfix stop
chkconfig postfix off
备注：
使用外部邮箱账号时，不需要启动sendmail或者postfix
如果在sendmail或者postfix启动的同时使用外部邮箱发送报警邮件，
首先会读取外部邮箱配置信息。
```

二、安装邮件发送工具mailx

```
yum install mailx #安装
```
CentOS 5.x 编译安装mailx，直接yum安装的mailx版本太旧，使用外部邮件发送会有问题。

```
yum remove mailx #卸载系统自带的旧版mailx
```
下载mailx：
http://nchc.dl.sourceforge.net/project/heirloom/heirloom-mailx/12.4/mailx-12.4.tar.bz2

```
tar jxvf mailx-12.4.tar.bz2 #解压
cd mailx-12.4 #进入目录
make #编译
make install UCBINSTALL=/usr/bin/install #安装
ln -s /usr/local/bin/mailx /bin/mail #创建mailx到mail的软连接
ln -s /etc/nail.rc /etc/mail.rc #创建mailx配置文件软连接
whereis mailx #查看安装路径
mailx -V #查看版本信息

```

三、配置Zabbix服务端外部邮箱
```
vi /etc/mail.rc #编辑，添加以下信息
set from=xxx@163.com smtp=smtp.163.com
set smtp-auth-user=xxx@163.com smtp-auth-password=123456
set smtp-auth=login
:wq! #保存退出
echo "zabbix test mail" |mail -s "zabbix" yyy@163.com
#测试发送邮件，标题zabbix，邮件内容：zabbix test mail，发送到的邮箱：yyy@163.com
#这时候，邮箱yyy@163.com会收到来自xxx@163.com的测试邮件
```
四、配置Zabbix服务端邮件报警


**1、打开Zabbix**
管理-示警媒介类型-Email
![zabbixmail1](https://www.msbgn.cn/msbgn/zabbixmail1.png)
![zabbixmail2](https://www.msbgn.cn/msbgn/zabbixmail2.png)
```
名称：Email
类型：电子邮件
SMTP伺服器：zabbix.sa.huanqiu.com
SMTP HELO：zabbix.sa.huanqiu.com
SMTP电邮：zabbix@zabbix.sa.huanqiu.com
已经用：勾选
存档
```
**2、设置Zabbix用户报警邮箱地址**
组态-用户-Admin (Zabbix Administrator)
![zabbixmail3](https://www.msbgn.cn/msbgn/zabbixmail3.png)
![zabbixmail4](https://www.msbgn.cn/msbgn/zabbixmail4.png)
切换到示警媒介-添加
![zabbixmail5](https://www.msbgn.cn/msbgn/zabbixmail5.png)
```
类型：Email
收件人：xxx@163.com
其他默认即可，也可以根据需要设置
状态：已启用
存档
```
**3、设置Zabbix触发报警的动作**

组态-动作-创建动作
![zabbixmail6](https://www.msbgn.cn/msbgn/zabbixmail6.png)
```
名称：Action-Email
默认接收人：故障{TRIGGER.STATUS},服务器:{Zabbix server}发生: {TRIGGER.NAME}故障!
默认信息：
告警主机:{Zabbix server}
告警时间:{EVENT.DATE} {EVENT.TIME}
告警等级:{TRIGGER.SEVERITY}
告警信息: {TRIGGER.NAME}
告警项目:{TRIGGER.KEY1}
问题详情:{ITEM.NAME}:{ITEM.VALUE}
当前状态:{TRIGGER.STATUS}:{ITEM.VALUE1}
事件ID:{EVENT.ID}
恢复信息：打钩
恢复主旨：恢复{TRIGGER.STATUS}, 服务器:{Zabbix server}: {TRIGGER.NAME}已恢复!
恢复信息：
告警主机:{Zabbix server}
告警时间:{EVENT.DATE} {EVENT.TIME}
告警等级:{TRIGGER.SEVERITY}
告警信息: {TRIGGER.NAME}
告警项目:{TRIGGER.KEY1}
问题详情:{ITEM.NAME}:{ITEM.VALUE}
当前状态:{TRIGGER.STATUS}:{ITEM.VALUE1}
事件ID:{EVENT.ID}
已启用：打钩
```
![zabbixmail7](https://www.msbgn.cn/msbgn/zabbixmail7.png)
切换到操作选项
新的

```
操作类型：送出信息
送到用户：添加
默认信息：打钩
```

```
用户：勾选Admin
选择
```
仅送到：Email
存档

**4、添加Zabbix服务端邮件发送脚本**
```
cd /usr/local/zabbix/share/zabbix/alertscripts #进入脚本存放目录
vi sendmail.sh #编辑，添加以下代码
#!/bin/sh
echo "$3" | mail -s "$2" $1
:wq! #保存退出
chown zabbix.zabbix /usr/local/zabbix/share/zabbix/alertscripts/sendmail.sh
#设置脚本所有者为zabbix用户
chmod +x /usr/local/zabbix/share/zabbix/alertscripts/sendmail.sh
#设置脚本执行权限
```
**五、测试Zabbix报警**
```
关闭Zabbix客户端服务
service zabbix_agentd stop
查看xxx@163.com邮箱，会收到报警邮件
再开启Zabbix客户端服务
service zabbix_agentd start
查看xxx@163.com邮箱，会收到恢复邮件
```

使用外部邮箱账号发送报警邮件设置完成。至此，Zabbix邮件报警设置完成。

» 文章出自链接：http://www.osyunwei.com/archives/8113.html
