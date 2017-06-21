---
title: zabbix如何实现微信报警
thumbnail: https://www.msbgn.cn/msbgn/wexin1.png
categories: zabbix
tags: [监控,zabbix]
---

-------------------


![微信报警机制](https://www.msbgn.cn/msbgn/wexin1.png)

 
<!-- more -->

###  申请企业号

首先需要申请一个企业号，其实公众号也可以，不过脚本不一样。而且公众号任何人都可以关注，有泄密的风险。企业号只有指定的人可以关注，安全性较高。
申请企业号，需要一个绑定你本人开户银行卡的微信号。
申请网址https://qy.weixin.qq.com/
点击“立即注册”。
根据提示注册企业号，到“选择类型”时，选择最右边的企业号。

![zabbix agent](https://www.msbgn.cn/msbgn/wexin00.png)

“主体类型”中选择“团队”。
注意：企业描述中：“报警”是敏感词不能使用。


-------------------


-------------------
###  设置企业号

如果一切顺利，申请成功并登录企业号，进入如下界面：

![ ](https://www.msbgn.cn/msbgn/wexin1.png)

（没有头像的话，去“设置”里自己传一个）
进入“通讯录”，点击“组织架构”旁边的加号，点击“新增成员”

![ ](https://www.msbgn.cn/msbgn/wexin2.png)
![ ](https://www.msbgn.cn/msbgn/wexin3.png)

填写完成后点击“保存”，如果一切顺利则应该可以看到用户。

![ ](https://www.msbgn.cn/msbgn/wexin4.png)

注：这里的账号相当于你的企业账号，与微信号无关。
必须先在此处创建用户，并且填写正确的微信号或者手机号，才可通过扫描二维码关注该企业号（知道为何安全了吧）。

###  关注企业号的方法


点击左侧的“设置”-二维码，使用微信扫一扫扫描二维码。

![ ](https://www.msbgn.cn/msbgn/wexin5.png)


点击左侧列的“应用中心”，点击“我的应用”下面的加号。填写应用名称，描述。
一切正常的话，点击进入刚才创建的应用。


![ ](https://www.msbgn.cn/msbgn/wexin6.png)

这里要记住一个值：应用ID。

设置管理员：
设置-功能设置-权限管理-新建管理组,设置权限中，保证管理员可以读取访问通讯录，可以发消息。
添加管理员之后，屏幕上显示二维码，用你的微信扫一扫扫描二维码并且设置登录密码。
![ ](https://www.msbgn.cn/msbgn/wexin7.png)

注意：这里要记录下来下面的CorpID和Secret。
现在万事俱备，可以开始编写脚本了。


### 编写脚本


检查/usr/etc/zabbix/zabbix_server.conf 中，加入
 AlertScriptsPath=/usr/lib/zabbix/alertscripts
如果有的话，就确定好报警目录。
创建目录。
mkdir –p /usr/lib/zabbix/alertscripts
在目录下创建`wechat.sh` 脚本文件。

```
#!/bin/bash

CorpID=<刚才记下来的CorpID，不要包含尖括号>
Secret=<刚才记下来的Secret，不要包含尖括号>
GURL="https://qyapi.weixin.qq.com/cgi-bin/gettoken?corpid=$CorpID&corpsecret=$Secret"
Gtoken=$(/usr/bin/curl -s -G $GURL | awk -F\" '{print $4}')

PURL="https://qyapi.weixin.qq.com/cgi-bin/message/send?access_token=$Gtoken"

function body() {
        local int AppID=<刚才记下来的应用id，不要包含尖括号>                        
        local UserID=$1                        
        local PartyID=1                          
        local Msg=$(echo "$@" | cut -d" " -f3-)  
        printf '{\n'
        printf '\t"touser": "'"$User"\"",\n"
        printf '\t"toparty": "'"$PartyID"\"",\n"
        printf '\t"msgtype": "text",\n'
        printf '\t"agentid": "'" $AppID "\"",\n"
        printf '\t"text": {\n'
        printf '\t\t"content": "'"$Msg"\""\n"
        printf '\t},\n'
        printf '\t"safe":"0"\n'
        printf '}\n'
}
/usr/bin/curl --data-ascii "$(body $1 $2 $3)" $PURL

```

将上面文字部分替换成自己记录下来的值。保存退出

```
chown –R zabbix:zabbix /usr/lib/zabbix/alertscripts
chmod 750/usr/lib/zabbix/alertscripts/wechat.sh
```

执行`./wechat.sh 1 1 test` 看自己微信是否能收到东西。
如果能的话，继续下一步。反之检查上面有什么问题。


### 添加触发器

 ![ ](https://www.msbgn.cn/msbgn/weixin10.png)

点击管理(Administration)-媒体类型(Media types)-创建媒体类型(Create media type)

 ![ ](https://www.msbgn.cn/msbgn/weixin11.png)
 
类型（Type）中选择脚本（Script），脚本名称（Script name）里填写wechat.sh
点击旁边的用户(User)创建一个用户。

 ![ ](https://www.msbgn.cn/msbgn/weixin12.png)
 ![ ](https://www.msbgn.cn/msbgn/weixin13.png)
 在Media中点击Add
 ![ ](https://www.msbgn.cn/msbgn/weixin14.png)
 ![ ](https://www.msbgn.cn/msbgn/weixin15.png)

When active中按如图填写，默认为7*24小时
Use if severity选择通告何种等级的消息，从上往下分别是：未分类，资讯，警告，一般，高威胁，灾难。
Send to 填写你的微信号。
点击Add.

![ ](https://www.msbgn.cn/msbgn/weixin16.png)


下一步，点击许可权（Permissions），将用户类型（User type）改为Zabbix超级用户（Zabbix Super Admin）。
创建触发动作。

![ ](https://www.msbgn.cn/msbgn/weixin17.png)


如此填写：


![ ](https://www.msbgn.cn/msbgn/weixin18.png)


` Host:{HOST.NAME}`
`Time:{EVENT.DATE} {EVENT.TIME}`
`Status:{TRIGGER.STATUS}`
`Event:{TRIGGER.NAME}`

勾选下面的故障恢复信息(Recovery message)并在Recovery message中填写：
`Server recovered.`
`Host:{HOST.NAME}`
`Time:{EVENT.DATE} {EVENT.TIME}`
`Status:{TRIGGER.STATUS}`
`Event:{TRIGGER.NAME} `

这样当服务器恢复后，可以收到一条以Server recovered开头的信息，省去了大半夜打车跑公司之苦。
条件(Conditions)无视。

![ ](https://www.msbgn.cn/msbgn/weixin19.png)

点击Operations，点击New

![ ](https://www.msbgn.cn/msbgn/weixin20.png)

点击User中的Add. 

![ ](https://www.msbgn.cn/msbgn/weixin21.png)


点击刚才创建的用户。

![ ](https://www.msbgn.cn/msbgn/weixin22.png)


最后结果如图，点击Add。然后点击最下面的Add.
配置完毕，现在关闭被监控机。[root@Jewel ~]# service network stop
等一会，若微信收到消息，则配置成功。

![ 故障告警信息与故障恢复信息 ](https://www.msbgn.cn/msbgn/wwwx11.png)
反之检查Zabbix报警是否有问题。
至此监控报警已经配置完成。
