---
title: nginx的编译安装
categories: nginx
tags: [科技,nginx]
thumbnail: https://www.msbgn.cn/msbgn/0.jpg

---

---
![nginx实现原理](https://www.msbgn.cn/msbgn/0.jpg)
<!-- more -->
第一部分：在Nginx服务器上分别操作

一、关闭SElinux、配置防火墙
1、vi /etc/selinux/config
```config
#SELINUX=enforcing #注释掉
#SELINUXTYPE=targeted #注释掉
SELINUX=disabled #增加

:wq!  #保存退出
setenforce 0 #使配置立即生效
```
2、vi /etc/sysconfig/iptables  #编辑
```iptables
-A RH-Firewall-1-INPUT -d 224.0.0.18 -j ACCEPT  #允许组播地址通信
-A RH-Firewall-1-INPUT -p    vrrp    -j ACCEPT  #允许VRRP（虚拟路由器冗余协）通信
-A RH-Firewall-1-INPUT -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT  #允许80端口通过防火墙
:wq! #保存退出 
/etc/init.d/iptables restart #重启防火墙使配置生效
(或者)
service iptables stop
```

二、安装Nginx

1、安装编译工具包（使用CentOS yum命令安装，安装的包比较多，方便以后配置lnmp环境）
```
yum install -y make apr* autoconf automake curl curl-devel gcc gcc-c++ gtk+-devel zlib-devel openssl openssl-devel pcre-devel gd kernel keyutils patch perl kernel-headers compat*  cpp glibc libgomp libstdc++-devel keyutils-libs-devel libsepol-devel libselinux-devel krb5-devel  libXpm* freetype freetype-devel freetype* fontconfig fontconfig-devel  libjpeg* libpng* php-common php-gd gettext gettext-devel ncurses* libtool* libxml2 libxml2-devel patch policycoreutils bison
```
2、下载软件包
（1）http://nginx.org/download/nginx-1.9.0.tar.gz  #下载Nginx
（2）https://sourceforge.net/projects/pcre/files/pcre/8.35/pcre-8.35.tar.gz/download  #下载pcre （支持nginx伪静态）
上传以上软件包到/usr/local/ 目录

3、安装pcre
```
cd /usr/local/src
mkdir /usr/local/pcre #创建安装目录
tar -zxvf pcre-8.35.tar.gz
cd pcre-8.35
./configure --prefix=/usr/local/pcre 
make && make install
```
4、安装Nginx
```
cd /usr/local/
groupadd  www  
useradd -g www www -s /bin/false  
tar  -zxvf nginx-1.9.0.tar.gz 
cd nginx-1.9.0

*带参数编译根据自己使用nginx的具体情况安装对应模块*
./configure --prefix=/etc/nginx --sbin-path=/usr/sbin/nginx --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --pid-path=/var/run/nginx.pid --lock-path=/var/run/nginx.lock --http-client-body-temp-path=/var/cache/nginx/client_temp --http-proxy-temp-path=/var/cache/nginx/proxy_temp --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp --http-scgi-temp-path=/var/cache/nginx/scgi_temp --user=nginx --group=nginx --with-http_ssl_module --with-http_realip_module --with-http_addition_module --with-http_sub_module --with-http_dav_module --with-http_flv_module --with-http_mp4_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_random_index_module --with-http_secure_link_module --with-http_stub_status_module --with-http_auth_request_module  --with-mail_ssl_module --with-file-aio

make && make install

设置nginx开启启动
vi /etc/rc.d/init.d/nginx #编辑启动文件添加下面内容
#!/bin/sh
#
# nginx - this script starts and stops the nginx daemin
#

# Comments to support chkconfig on RedHat Linux
# chkconfig: 2345 64 36
# description: nginx

# Comments to support LSB init script conventions
### BEGIN INIT INFO
# Provides: nginx
# Required-Start: $local_fs $network $remote_fs
# Should-Start: ypbind nscd ldap ntpd xntpd
# Required-Stop: $local_fs $network $remote_fs
# Default-Start:  2 3 4 5
# Default-Stop: 0 1 6
# Short-Description: start and stop nginx
### END INIT INFO

# Source function library.
. /etc/init.d/functions

nginx="/usr/sbin/nginx "
prog=$(basename $nginx)

NGINX_CONF_FILE="/etc/nginx/nginx.conf"

lockfile=/var/lock/subsys/nginx

start() {
    [ -x $nginx ] || exit 5
    [ -f $NGINX_CONF_FILE ] || exit 6
    echo -n $"Starting $prog: "
    daemon $nginx -c $NGINX_CONF_FILE
    retval=$?
    echo
    [ $retval -eq 0 ] && touch $lockfile
    return $retval
}

stop() {
    echo -n $"Stopping $prog: "
    killproc $prog -QUIT
    retval=$?
    echo
    [ $retval -eq 0 ] && rm -f $lockfile
    return $retval
}

restart() {
    configtest || return $?
    stop
    sleep 1
    start
}

reload() {
    configtest || return $?
    echo -n $"Reloading $prog: "
    killproc $nginx -HUP
    RETVAL=$?
    echo
}

force_reload() {
    restart
}

configtest() {
  $nginx -t -c $NGINX_CONF_FILE
}

rh_status() {
    status $prog
}

rh_status_q() {
    rh_status >/dev/null 2>&1
}

case "$1" in
    start)
        rh_status_q && exit 0
        $1
        ;;
    stop)
        rh_status_q || exit 0
        $1
        ;;
    restart|configtest)
        $1
        ;;
    reload)
        rh_status_q || exit 7
        $1
        ;;
    force-reload)
        force_reload
        ;;
    status)
        rh_status
        ;;
    condrestart|try-restart)
        rh_status_q || exit 0
            ;;
    *)
        echo $"Usage: $0 {start|stop|status|restart|condrestart|try-restart|reload|force-reload|configtest}"
        exit 2
esac
*************************************************************************************************
:wq! #保存退出
chmod 775 /etc/rc.d/init.d/nginx #赋予文件执行权限
chkconfig nginx on #设置开机启动
/etc/rc.d/init.d/nginx restart  #重新启动Nginx
service nginx restart
*************************************************************************************************

```
三、配置Nginx
```
cp /etc/nginx/nginx.conf  /etc/nginx/nginx.conf.bak  #备份nginx配置文件
```
1、设置nginx运行账
```
vi  /etc/nginx/nginx.conf  #编辑，修改
找到server，在上面一行添加如下内容：
server {
listen       80 default;
server_name  _;
location / {
root   html;
return 404;
}
location ~ /.ht {

deny  all;
}
}
:wq! #保存退出
service nginx restart
这样设置之后，空主机头访问会直接跳转到nginx404错误页面。
```
3、添加nginx虚拟主机包含文件
```
cd /etc/nginx/
mkdir vhost   
vi /etc/nginx/nginx.conf
找到上一步添加的代码，在最后添加如下内容：
include  vhost/*.conf;
:wq! #保存退出
例如：
server {
listen       80 default;
server_name  _;
location / {
root   html;
return 404;
}
location ~ /.ht {
deny  all;
}
}
include  vhost/*.conf;
```
4、添加Web服务器列表文件
```
cd  /etc/nginx/   #进入目录 
touch  mysvrhost.conf  #建立文件
vi  /etc/nginx/nginx.conf   #编辑
找到上一步添加的代码，在下面添加一行
include  mysvrhost.conf;
:wq! #保存退出
```
5、设置nginx全局参数
```
vi  /etc/nginx/nginx.conf
worker_processes 2;   # 工作进程数,为CPU的核心数或者两倍
events
{
use epoll;   #增加
worker_connections 65535;    #修改为65535，最大连接数。
}
********以下代码在http { 部分增加与修改********
server_names_hash_bucket_size 128;   #增加
client_header_buffer_size 32k;       #增加
large_client_header_buffers 4 32k;   #增加
client_max_body_size 300m;           #增加
tcp_nopush     on;      #修改为on
keepalive_timeout  60;  #修改为60
tcp_nodelay on;        #增加
server_tokens off;     #增加，不显示nginx版本信息
gzip  on;  #修改为on
gzip_min_length  1k;      #增加
gzip_buffers     4 16k;   #增加
gzip_http_version 1.1;    #增加
gzip_comp_level 2;        #增加
gzip_types       text/plain application/x-javascript text/css application/xml;  #增加
gzip_vary on;  #增加
```
6、设置Web服务器列表
```
cd  /etc/nginx   #进入目录
vi mysvrhost.conf  #编辑，添加以下代码
upstream  osyunweihost {
server 192.168.21.127:80 weight=1 max_fails=2 fail_timeout=30s;
server 192.168.21.128:80 weight=1 max_fails=2 fail_timeout=30s;
ip_hash;
}
```
7、新建虚拟主机配置文件```
```
cd /etc/nginx/vhost   
touch osyunwei.conf #建立虚拟主机配置文件
vi  osyunwei.conf #编辑
log_format  access  '$remote_addr - $remote_user [$time_local] $request '
'"$status" $body_bytes_sent "$http_referer" '
'"$http_user_agent" "$http_x_forwarded_for"';
server
{
listen       80;
server_name www.a.com www.b.com;
location /
{
proxy_next_upstream http_502 http_504 error timeout invalid_header;
proxy_pass http://osyunweihost;
#proxy_redirect off;
proxy_set_header Host  $host;
proxy_set_header X-Forwarded-For  $remote_addr;
}
location /NginxStatus {
stub_status on;
access_log  on;
auth_basic  "NginxStatus";
######auth_basic_user_file  pwd;
}
access_log  /usr/local/nginx/logs/access.log  access;
}
:wq!  #保存配置
service nginx restart  #重启nginx
```
****至此nginx安装配置基本完成****




