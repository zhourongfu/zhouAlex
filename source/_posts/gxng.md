---
title: 高性能WEB服务器nginx
categories: nginx
tags: [科技,nginx]
thumbnail: https://www.msbgn.cn/msbgn/gxng.png
---

-------------------


![](https://www.msbgn.cn/msbgn/gxng.png)


<!-- more -->


###  高性能WEB服务器nginx的介绍
Nginx（发音同 engine x）是一款轻量级的Web 服务器／反向代理服务器及电子邮件（IMAP/POP3）代理服务器，并在一个BSD-like 协议下发行。由俄罗斯的程序设计师Igor Sysoev所开发，最初供俄国大型的入口网站及搜寻引擎Rambler（俄文：Рамблер）使用。  其特点是占有内存少，并发能力强，事实上nginx的并发能力确实在同类型的网页伺服器中表现较好.目前中国大陆使用nginx网站用户有：新浪、网易、 腾讯,另外知名的微网志Plurk也使用nginx。

###  Nginx的组成原理与工作原理

Nginx由内核和模块组成，其中，内核的设计非常微小和简洁，完成的工作也非常简单，仅仅通过查找配置文件将客户端请求映射到一个location block（location是Nginx配置中的一个指令，用于URL匹配），而在这个location中所配置的每个指令将会启动不同的模块去完成相应的工作。
Nginx的模块从结构上分为核心模块、基础模块和第三方模块， HTTP模块、EVENT模块和MAIL模块等属于核心模块，HTTP Access模块、HTTP FastCGI模块、HTTP Proxy模块和HTTP Rewrite模块属于基础模块，而HTTP Upstream Request Hash模块、Notice模块和HTTP Access Key模块属于第三方模块，用户根据自己的需要开发的模块都属于第三方模块。正是有了这么多模块的支撑，Nginx的功能才会如此强大。
Nginx的模块从功能上分为如下三类。
Handlers（处理器模块）。此类模块直接处理请求，并进行输出内容和修改headers信息等操作。Handlers处理器模块一般只能有一个。
Filters （过滤器模块）。此类模块主要对其他处理器模块输出的内容进行修改操作，最后由Nginx输出。
Proxies （代理类模块）。此类模块是Nginx的HTTP Upstream之类的模块，这些模块主要与后端一些服务比如FastCGI等进行交互，实现服务代理和负载均衡等功能。
图1-1展示了Nginx模块常规的HTTP请求和响应的过程。

![](https://www.msbgn.cn/msbgn/gxng.png)


在工作方式上，Nginx分为单工作进程和多工作进程两种模式。在单工作进程模式下，除主进程外，还有一个工作进程，工作进程是单线程的；在多工作进程模式下，每个工作进程包含多个线程。Nginx默认为单工作进程模式。
Nginx的模块直接被编译进Nginx，因此属于静态编译方式。启动Nginx后，Nginx的模块被自动加载，不像Apache，首先将模块编译为一个so文件，然后在配置文件中指定是否进行加载。在解析配置文件时，Nginx的每个模块都有可能去处理某个请求，但是同一个处理请求只能由一个模块来完成。



###  Nginx的性能优势

Nginx的性能优势
作为WEB服务器，nginx处理静态文件，索引以及自动索引的效率非常高
作为负载均衡服务器，nginx既可以直接支持Rails和PHP，也可以支持HTTP反向代理服务器，对外进行服务，还支持简单的容错。
在性能方面，nginx采用了分阶段的资源分配技术，使用CPU跟内存非常低
高可用方面 我个人觉得这个很不错。在master管理进程与worker工作进程的分离设计，使的Nginx具有热部署的功能，那么在7×24小时不间断服务的前提下，升级Nginx的可执行文件。也可以在不停止服务的情况下修改配置文件，更换日志文件等功能。

Nginx的安装请见地址：http://www.msbgn.cn/2016/11/01/nginxbianyi/

###  Nginx的配置与调试

Nginx配置文件主要分成四部分：main（全局设置）、server（主机设置）、upstream（负载均衡服务器设置）和 location（URL匹配特定位置的设置）。main部分设置的指令将影响其他所有设置；server部分的指令主要用于指定主机和端口；upstream指令主要用于负载均衡，设置一系列的后端服务器；location部分用于匹配网页位置。这四者之间的关系式：server继承main，location继承server，upstream既不会继承其他设置也不会被继承。
         在这四个部分当中，每个部分都包含若干指令，这些指令主要包含Nginx的主模块指令、事件模块指令、HTTP核心模块指令，同时每个部分还可以使用其他HTTP模块指令，例如Http SSL模块、HttpGzip Static模块和Http Addition模块等。
下面通过一个Nginx配置实例，详细介绍下nginx.conf每个指令的含义。

```
user  nobody nobody;
 worker_processes  4;
 error_log  logs/error.log  notice;
 pid        logs/nginx.pid;
 worker_rlimit_nofile 65535; 
 events{
  use epoll;
  worker_connections      65536;
       }

```

`user`：是个主模块指令，指定Nginx Worker进程运行用户以及用户组，默认由nobody账号运行。
`worker_processes`：是个主模块指令，指定了Nginx要开启的进程数。每个Nginx进程平均耗费10M~12M内存。根据经验，一般指定一个进程足够了，如果是多核CPU，建议指定和CPU的数量一样的进程数即可。

`error_log`是个主模块指令，用来定义全局错误日志文件。日志输出级别有debug、info、notice、warn、error、crit可供选择，其中，debug输出日志最为最详细，而crit输出日志最少。

`pid`是个主模块指令，用来指定进程id的存储文件位置。
`worker_rlimit_nofile`用于指定一个nginx进程可以打开的最多文件描述符数目，这里是65535，需要使用命令“ulimit -n 65535”来设置。
`events`指令是设定Nginx的工作模式及连接数上限。


Nginx案列1

###  Nginx作为WEB缓存服务器应用
从0.7.48版本开始，Nginx支持类似Squid的缓存功能。Nginx的web缓存服务主要由proxy_cache相关命令集合fastcgi_cache相关命令集构成，前者用于反向代理时对后端内容源服务器进行缓存，后者主要用于对FastCGI的动态程序进行缓存。此外，如果不想使用Nginx自带的缓存功能，也可使用第三方模块ngx_slowfs_cache来实现缓存服务器的配置。


这里使用Nginx自带的缓存模块，通过proxy命令来实现数据的缓存,所以在编译的时候要加上ngx_cache_purge模块

下面介绍nginx作为缓存服务器的安装步骤
Prce的安装不再重复
`Wget http://labs.frickle.com/files/ngx_cache_purge-2.1.tar.gz`
`Tar –zxvf –C /app/ngx_cache_purge-2.1.tar.gz`


在网上下载ngx_cache_purge插件的最新版本，然后重新编译安装nginx。
`./configure --prefix=/usr/local/nginx --without-http_memcached_module --user=www --group=www --with-http_stub_status_module --with-openssl=/usr/ --with-pcre=/usr/local/src/pcre-8.31 --add-module=/usr/local/src/ngx_cache_purge-2.3`（文件解压后存放的位置）
 最后通过”/usr/local/nginx/nginx-v”查看是否已经安装的成功的版本和加载模块信息

配置nginx缓存服务器

Nginx缓存服务器的配置主要通过proxy_cache相关命令来实现

`proxy_cache_path /backup/proxy_cache_dir levels=1:2 keys_zone=cache_one:4096m inactive=1d max_size=3g;`
•	`poxy_cache_path`:用于设置缓存的目录，后面跟缓存路径。最好将缓存目录放在一个独立的硬盘上。
•	`levels=1:2`：levels用来设置目录深度，这里是两层目录深度，第一层是一个字符，第二层是两个字符。
•	`keys_zne`：用来设置web缓存区名称，这里的cache_one后面的4096，表示内存缓存空间大小为4GB
•	`inactive`：表示自动清除缓存文件的时间，这里的“d”表示1天没有被访问的内容自动清除，还可以使用分钟和小时计数，5m，5h。
•	`max_size`:表示硬盘缓存空间可使用的最大值，默认情况下经访问的文件常将被放到内存中进行缓存，而在内存缓存空间不足时，Nginx会将不经常访问的数据从内存写到磁盘。
 
`proxy_temp_path /backup/proxy_temp_dir`;
•	用于指定临时缓存文件的存储路径，这里需要注意的是，两个存放缓存文件的目录必须在同一磁盘分区。
•`	location / {
•	    root   html;
•	    index  index.html index.htm index.php;
•	    proxy_cache cache_one`;           #反向代理缓存设置命令，语法为“proxy_cache zone|off“，默认为off，需要将proxy_cache命令放在location字段，这样匹配以此location的url才能被缓存。
•	   ` proxy_cache_valid 200 304 12h`;       #对不懂HTTP状态码设置不同的缓存时间
•	   ` proxy_cache_key $host$uri$is_args$args`;      #这个命令是设置以什么样的参数得到缓存的文件名，默认为”$scheme$proxy_host$request_uri”,表示以协议，主机名，请求uri（包括参数）做MD5得出缓存的文件名。这里以域名，URI，参数组合成Web缓存的key值，nginx会根据key哈希，存储缓存内容到二级缓存目录内
•	    }
•	        

```
       下面配置手动清除缓存策略：
         location ~ /purge(/.*) {
       allow 127.0.0.1;
       allow 192.168.1.0/24;
        deny all;
	        proxy_cache_purge cache_one $host$1$is_args$args;
}

```
•	这里设置可以清除缓存的ip和网段，下面说的是清除的内容
```
•	         location ~ \.php?$ {
•	            root           html;
•	            fastcgi_pass   127.0.0.1:9000;
•	            fastcgi_index  index.php;
•	            fastcgi_param  SCRIPT_FILENAME  /usr/local/nginx/html/$fastcgi_script_name;
•	            include        fastcgi_params;
•	}

```
•	以.php结尾的文件不用缓存
•	手动清除缓存的方法
•	http://192.168.1.120/index.html                                                  访问
•	http://192.168.1.120/purge/index.html                                      清除缓存策略
•	 t
•	验证我们的缓存服务是否成功启动


```
[root@Goun conf]# ps -ef | grep nginx
root       9390      1  0 20:52 ?        00:00:00 nginx: master process /usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
www        9392   9390  0 20:52 ?        00:00:00 nginx: worker process                                         
www        9393   9390  0 20:52 ?        00:00:00 nginx: cache manager process
```

进程nginx：cache manager process这个进程是用来管理缓存服务和文件的。
如何清除指定的URL缓存

如果有时候修改了页面内容，如果不想等到缓存文件过期后自动清理，还可以手动方式清理缓存，清理方式在上面已经说明，只需叜清除缓存网页的URL地址钱增加`purge`即可
列如清除` http://localhost:8088/forestRight/1.html`;
那么：`http://localhost:8088/pirge/forestRight/1.html;`即可


##  Nginx作为负载均衡服务器应用 


nginx的负载均衡功能是通过upstream命令实现的，因此他的负载均衡机制比较简单，是一个基于内容和应用的7层交换负载均衡的实现。Nginx负载均衡默认对后端服务器有健康监测能力，但是监测能力较弱，仅限于端口监测，在后端服务器比较少的情况下（10台以下）负载均衡能力表现突出。而对于有大量后端节点的负载应用，由于所有访问请求都从一台服务器进出，容易发生请求堵塞进而引发连接失败，因此无法充分发挥后端服务器的性能。


## Nginx负载均衡算法


Nginx的负载均衡模块目前支持4中调度算法，下面分别进行介绍，其中，后两种属于第三方调度方法：


•	轮询（默认）：每个请求按时间顺序逐一分配到不同的后端服务器，如果后端某台服务器死机，自动剔除故障系统，使用户访问不受影响。
•	weight：指定轮询权值，weight值越大，分配到访问概率越高，主要用于后端每台服务器性能不均衡的情况下。
•	ip_hash:每个请求按照ip的哈希结果分配，这样来自同一个ip的访客固定访问一台后端服务器，有效解决动态网页存在的session共享问题。
•	fair：它是比上面两种更加智能的负载均衡算法。此种算法可以依据页面大小和加载时间长短智能的进行负载均衡，也就是根据后端服务器的响应时间来分配请求，响应时间短的优先分配。Nginx本身是不支持fair的，如果需要使用这种调度算法，必须下载nginx的upstream_fair模块。
•	url_hash：按访问URL的哈希结果来分配请求，使每个URL定向到同一台后端服务器，可以进一步提高后端缓存服务器的效率。Nginx本身不支持url_hash的，如果需要使用这种调度算法，必须安装Nginx的hash软件包。



在HTTP Upstream模块中，可以通过server命令指定后端服务器的IP地址和端口，同时还可以设定每台后端服务器在负载均衡调度中的状态。常用的状态有：
•	down：表示当前的server暂时不参与负载均衡。
•	backup：预留的备份机器。当其他所有的非backup机器出现故障或者忙的时候，才会请求backup机器，因此这台机器的访问压力最轻。
•	max_fails:允许请求失败的次数，默认为1.当超过最大次数时，返回proxy_next_upstream模块定义的错误。
•	fail_timeout:在经历了max_fails次失败后，暂停服务的时间。max_fails可以和fail_timeout一起使用。
注意：当调度算法为ip_hash时，后端服务器在负载均衡调度中的调度状态不能是weight和backup。
Nginx的负载均衡配置实例

```
http {
         upstream myserver {
    server 192.168.1.120:80 weight=1 max_fails=3 fail_timeout=20s;
    server 192.168.1.121:80 weight=1 max_fails=3 fail_timeout=20s;
}
    server {
        listen       80;
        server_name  localhost;
        index  index.html index.htm;
        #ciharset koi8-r;
        root html;
        #access_log  logs/host.access.log  main;
 
        location / {
        proxy_pass http://myserver;
        proxy_next_upstream http_500 http_502 http_503 error timeout invalid_header;
 
        }
}
```


在上面这段配置中，upstream关键字标识负载均衡配置开始，upstream是Nginx的HTTP Upstream模块，这个模块通过一个简单的调度算法来实现客户端IP到后端服务器的负载均衡。在上面。通过upstream制定一个负载均衡器的名称为myserver。这个名称可以任意指定，在后面需要的地方直接调用即可。
另外，proxy_next_upstream 参数用来定义故障转移策略，当后端服务节点返回500、502、503、504和执行超时等错误时，自动将请求转发到upstream负载均衡组中的另一台服务器，实现故障转移



###  服务器Nginx(Nginx性能优化技巧)



###  一、	编译安装过程优化


1.减小Nginx编译后的文件大小
在编译Nginx时，默认以debug模式进行，而在debug模式下会插入很多跟踪和ASSERT之类的信息，编译完成后，一个Nginx要有好几兆字节。在编译前取消Nginx的debug模式，编译完成后Nginx只有几百千字节，因此可以在编译之前，修改相关源码，取消debug模式，具体方法如下：
在Nginx源码文件被解压后，找到源码目录下的auto/cc/gcc文件，在其中找到如下几行：

```
1.	# debug  
2.	CFLAGS=”$CFLAGS -g”  
```

注释掉或删掉这两行，即可取消debug模式。

2.为特定的CPU指定CPU类型编译优化


在编译Nginx时，默认的GCC编译参数是“-O”，要优化GCC编译，可以使用以下两个参数：
--with-cc-opt='-O3'
--with-cpu-opt=CPU  #为特定的 CPU 编译，有效的值包括：pentium, pentiumpro, pentium3, pentium4, athlon, opteron, amd64, sparc32, sparc64, ppc64
要确定CPU类型，可以通过如下命令: 
[root@localhost home]#cat /proc/cpuinfo | grep "model name"

`TCMalloc`的全称是 Thread-Caching Malloc,是谷歌开发的开源工具google-perftools中的一个成员。与标准的glibc库的Malloc相比，TCMalloc库在内存分配效率和速度上要高很多，这在很大程度上提高了服务器在高并发情况下的性能，从而降低了系统的负载。下面简单介绍如何为Nginx添加TCMalloc库支持



要安装TCMalloc库，需要安装libunwind 和 gperftools两个软件包，libunwind库为基于64为CPU何操作系统的程序提供了基本函数调用链和函数调用寄存器功能，32位操作系统部需要安装。



1 安装libunwind库


可以从LinuxIDC.com的FTP下载libunwind-1.1.tar.gz
libunwind-1.1.tar.gz下载地址：
 
免费下载地址在 http://linux.linuxidc.com/
用户名与密码都是www.linuxidc.com
具体下载目录在 /2013年资料/4月/21日/利用TCMalloc优化Nginx的性能


安装过程如下：
tar -xvf libunwind-1.1.tar.gz
cd libunwind-1.1
CFLAGS=-fPIC ./configure
make CFLAGS=-fPIC
make CFLAGS=-fPIC install
2.安装gperftools 可以从 这里 下载2.0版本
tar -xvf gperftools-2.0.tar.gz
cd gperftools-2.0
./configure
make && make install
echo "/usr/local/lib" > /etc/ld.so.conf.d/usr_local_lib.conf
ldconfig
支持gperftools安装完成
3.重新编译Nginx.
./configure --prefix=/usr/local/nginx \  指定nginx的安装目录
--with-http_stub_status_module \  启用nginx的status功能，可以监控nginx当前状态
--with-http_gzip_static_module \  支持在线实时压缩输出数据流
--with-google_perftools_module \  支持TCMalloc对Nginx性能的优化
make && make install
到这里Nginx安装完成。
4.为gperftools添加线程目录
创建一个线程目录，这里讲文件放在/tmp/tcmalloc下。操作如下：
mkdir /tmp/tcmalloc
chmod 0777 /tmp/tcmalloc
5.修改nginx主配置文件，在pid这行的下面添加如下代码：
pid  logs/nginx.pid;
google_perftools_profiles /tmp/tcmalloc;
接着重启nginx即可完成gperftools的加载。
6.验证运行状态
为了验证gperftools已经正常加载，可以通过如下命令查看：

```
lsof -n | grep tcmalloc
nginx    2395 nobody    9w REG    8,8    0    1599440 /tmp/tcmalloc.2395
nginx    2396 nobody    11w REG    8,8    0    1599443 /tmp/tcmalloc.2396
nginx    2397 nobody    13w REG    8,8    0    1599441 /tmp/tcmalloc.2397
nginx    2398 nobody    15w REG    8,8    0    1599442 /tmp/tcmalloc.2398
```
由于在Nginx配置文件中设置worker_processes的值为4 ，因此开启了4个Nginx线程，每个线程都会有一行记录。每个线程文件后面的数字值就是启动Nginx的pid值。
至此，利用TCMalloc优化Nginx的操作完成。


###  Nginx内核参数的优化


内核参数的优化，主要是在linux系统中针对Nginx应用而进行的系统内核参数的优化。





```
下面给出的一个优化实例以供参考。
net.ipv4.tcp_max_tw_buckets = 6000
net.ipv4.ip_local_port_range = 1024 65000
net.ipv4.tcp_tw_recycle = 1
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_syncookies = 1
net.core.somaxconn = 262144
net.core.netdev_max_backlog = 262144
net.ipv4.tcp_max_orphans = 262144
net.ipv4.tcp_max_syn_backlog = 262144
net.ipv4.tcp_synack_retries = 1
net.ipv4.tcp_syn_retries = 1
net.ipv4.tcp_fin_timeout = 1
net.ipv4.tcp_keepalive_time = 30
将上面的内核参数值 加入到/etc/sysctl.conf文件中，然后执行如下命令使之生效
/sbin/sysctl -p
```


下面对上面的参数进行介绍一下：


net.ipv4.tcp_max_tw_buckets 用来设定timewait的数量 默认是180000，这里改为6000
net.ipv4.ip_local_port_range 用来设定允许系统打开的端口范围最小值1024
net.ipv4.tcp_tw_recycle 用来设置启动timewait快速回收。
net.ipv4.tcp_tw_reuse 用来设置开启重用，允许将time-wait sockets重新用于新的tcp连接
net.ipv4.tcp_syncookies 用来开启syn cookies,当出现syn等待队列一处时，启用cookies处理
net.core.somaxconn 默认是128，参数用于调节系统同时发起的tcp连接数，在高并发的请求中，默认的值可能会导致连接超时或者重传，因此，需要结合并发请求数来调节此值。
net.core.netdev_max_backlog 表示当每个网络接口接受数据包的速率比内核处理这些包的速率快时，允许发送到队列的数据包的最大数目。
net.ipv4.tcp_max_orphans  用于设定系统中最多有多少个tcp套接字不被关联到任何一个用户文件句柄上。如果超过这个数字，孤立连接将立即被复位并打印出警告信息。这个限制值是为了防止简单的DOS攻击。不能过分依靠这个限制甚至人为减小这个值，更多的情况下应该增加这个值。
net.ipv4.tcp_max_syn_backlog 用于记录那些尚未收到客户端确认信息的连接请求的最大值。对于有128MB内存的系统而言，次参数默认值是1024，对小内存的系统则是128
net.ipv4.tcp_synack_retries 参数的值决定了内核放弃连接之前发送SYN+ACK包的数量
net.ipv4.tcp_syn_retries 表示在内核放弃简历连接之前发送SYN包的数量
net.ipv4.tcp_fin_timeout 决定了套接字保持在FIN-WAIT-2 状态的时间。默认值是60秒。正确设置这个值非常重要，有时即使一个负载很小的web服务器，也会出现大量的死套接字而产生内存溢出的风险。
net.ipv4.tcp_keepalive_time 表示当keepalive启动的时候，tcp发送keepalive消息的频度。默认值是2（单位是小时）。


详细查看地址： http://linux.linuxidc.com/ 
