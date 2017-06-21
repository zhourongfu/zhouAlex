---
title:  elk实时日志分析平台搭建详细过程
thumbnail: https://www.msbgn.cn/msbgn/ELK1.png
categories: ELK
tags: [语言,ELK]
---
-------------------



![ ](https://www.msbgn.cn/msbgn/ELK1.png)


<!-- more -->


###   Elasticsearch
Elasticsearch 是个开源分布式搜索引擎，它的特点有：分布式，零配置，自动发现，索引自动分片，索引副本机制，restful风格接口，多数据源，自动搜索负载等。
Logstash 是一个完全开源的工具，他可以对你的日志进行收集、分析，并将其存储供以后使用（如，搜索）
kibana 也是一个开源和免费的工具，他Kibana可以为 Logstash 和 ElasticSearch 提供的日志分析友好的 Web 界面，可以帮助您汇总、分析和搜索重要数据日志。


###  服务端软件

Elasticsearch：负责日志检索和分析，它的特点有：分布式，零配置，自动发现，索引自动分片，索引副本机制，restful风格接口，多数据源，自动搜索负载等
Logstash：对日志进行收集、过滤，并将其存储供以后使用（如，搜索日志）
Kibana：为日志分析提供友好的Web界面，可以帮助汇总、分析和搜索重要数据日志


###  客户端软件

在需要收集日志的所有服务上部署logstash，作为logstash agent（logstash shipper）用于监控并过滤收集日志，将过滤后的内容发送到logstash indexer，logstash indexer将日志收集在一起交给全文搜索服务ElasticSearch，可以用ElasticSearch进行自定义搜索，然后通过Kibana来结合自定义搜索进行页面展示。
192.168.50.119：ELK+Nginx
192.168.50.120：Redis+Logstash

![ ](https://www.msbgn.cn/msbgn/ELK1.png)


###  部署流程：

1.安装JDK
Logstash的运行依赖于Java运行环境， Logstash 1.5以上版本不低于java 7推荐使用最新版本的Java，我这里使用了1.8版本
我们只需要Java的运行环境，所以可以只安装JRE，不过这里我依然使用JDK，请自行搜索安装。


```
#rpm -ivh jdk-8u25-linux-x64.rpm 

vim /etc/profile  #设置环境变量
export JAVA_HOME=/usr/local/jdk1.8.0_45
export PATH=$PATH:$JAVA_HOME/bin
export CLASSPATH=.:$JAVA_HOME/lib/tools.jar:$JAVA_HOME/lib/dt.jar:$CLASSPATH
source /etc/profile  #使环境变量生效

```

```
java -version
java version "1.8.0_25"
Java(TM) SE Runtime Environment (build 1.8.0_25-b17)
Java HotSpot(TM) 64-Bit Server VM (build 25.25-b02, mixed mode)
```

2.安装Logstash（日志收集、分析，并将其存储供以后使用）
这里安装遇到了下载的坑，最好去使用迅雷下载安装包	


```
wget https://download.elastic.co/logstash/logstash/logstash-2.4.0.tar.gz
tar –zxf logstash-2.4.0.tar.gz -C /usr/local/

```

验证logstash是否安装成功

```
[root@localhost ~]# /usr/local/logstash-2.4.0/bin/logstash -e 'input { stdin { } } output { stdout {} }'
Settings: Default pipeline workers: 1
Logstash startup completed
等待输入:hello world
2016-11-28T20:32:07.853Z localhost.localdomain hello world

```

3.部署nginx并收集日志


```
yum -y install nginx
设置nginx的log 格式
vim /etc/nginx/nginx.conf
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" $http_x_forwarded_for $request_length $msec $connection_requests $request_time';
```

系统安装了nginx就跳过此步骤

```
service nginx start
```


```
mkdir /usr/local/logstash-2.4.0/conf/   #创建logstash配置目录
定义logstash配置文件，用来收集nginx日志
[root@localhost conf]# cat logstash_nginx.conf 
input {
     file {
        path => ["/var/log/nginx/access.log"]
        type => "nginx_log"
     }
}
output {
     redis{
         host => "192.168.50.120"
         key => 'logstash-redis'
         data_type => 'list'
    }
    stdout {
codec => rubydebug
    }
}
```

4.安装部署redis 

`192.168.50.120` 服务器


```
yum -y install redis
vim /etc/redis.conf
bind 192.168.50.120


service redis start
```


5.启动Logstash


```
[root@localhost conf]# /usr/local/logstash-2.4.0/bin/logstash -f ./logstash_nginx.conf  --configtest   #检查配置文件
Configuration OK
```

```
[root@localhost conf]# /usr/local/logstash-2.4.0/bin/logstash agent  -f ./logstash_nginx.conf          #将日志信息输出到redis服务器

Settings: Default pipeline workers: 1
Logstash startup completed





{
       "message" => "192.168.50.114 - - [29/Nov/2016:00:58:43 +0800] \"GET / HTTP/1.1\" 304 0 \"-\" \"Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/54.0.2840.99 Safari/537.36\" \"-\"",
      "@version" => "1",
    "@timestamp" => "2016-11-28T18:55:49.587Z",
          "path" => "/var/log/nginx/access.log",
          "host" => "localhost.localdomain",
          "type" => "nginx_log"
}
{
       "message" => "192.168.50.114 - - [29/Nov/2016:00:58:43 +0800] \"GET /nginx-logo.png HTTP/1.1\" 304 0 \"http://192.168.50.119/\" \"Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/54.0.2840.99 Safari/537.36\" \"-\"",
      "@version" => "1",
    "@timestamp" => "2016-11-28T18:55:49.590Z",
          "path" => "/var/log/nginx/access.log",
          "host" => "localhost.localdomain",
          "type" => "nginx_log"
}
{
       "message" => "192.168.50.114 - - [29/Nov/2016:00:58:43 +0800] \"GET /poweredby.png HTTP/1.1\" 304 0 \"http://192.168.50.119/\" \"Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/54.0.2840.99 Safari/537.36\" \"-\"",
      "@version" => "1",
    "@timestamp" => "2016-11-28T18:55:49.590Z",
          "path" => "/var/log/nginx/access.log",
          "host" => "localhost.localdomain",
          "type" => "nginx_log"

```


6.安装部署Elasticsearch


`192.168.50.119` ELK服务器


创建安装用户

```
groupadd elk
useradd es -g elk
```


```
tar -xf elasticsearch-2.2.0.tar.gz -C /usr/local/
vim /usr/local/elasticsearch-2.2.0/config/elasticsearch.yml
   network.host: 192.168.50.119   # 端口绑定ip地址
   http.port: 9200
```

启动 ，这里遇到一个坑：es用户默认是不能用root用户启动的。所以要切到普通用户启动


```
chown -R es.elk /usr/local/elasticsearch-2.2.0
su - es
nohup  /usr/local/elasticsearch-2.2.0/bin/elasticsearch >/usr/local/elasticsearch-2.2.0/nohub &
```

```
[root@localhost ELK]# netstat -tunpl | grep 9200
tcp        0      0 ::ffff:192.168.50.119:9200  :::*                        LISTEN      2183/java

```


```
[root@localhost ELK]# curl http://192.168.50.119:9200   #查看状态
{
  "name" : "Blood Brothers",
  "cluster_name" : "elasticsearch",
  "version" : {
    "number" : "2.2.0",
    "build_hash" : "8ff36d139e16f8720f2947ef62c8167a888992fe",
    "build_timestamp" : "2016-01-27T13:32:39Z",
    "build_snapshot" : false,
    "lucene_version" : "5.4.1"
  },
  "tagline" : "You Know, for Search"
```

![ ](https://www.msbgn.cn/msbgn/ELK2.png)



安装kopf和head插件


```
[root@localhost conf]# cd /usr/local/elasticsearch-2.2.0/bin/
[root@localhost bin]# ./plugin  install lmenezes/elasticsearch-kopf
-> Installing lmenezes/elasticsearch-kopf...
Trying https://github.com/lmenezes/elasticsearch-kopf/archive/master.zip ...
Downloading ............................................................ DONE
Verifying https://github.com/lmenezes/elasticsearch-kopf/archive/master.zip checksums if available ...
NOTE: Unable to verify checksum for downloaded plugin (unable to find .sha1 or .md5 file to verify)
Installed kopf into /usr/local/elasticsearch-2.2.0/plugins/kopf



[root@localhost bin]# ./plugin install mobz/elasticsearch-head
-> Installing mobz/elasticsearch-head...
Trying https://github.com/mobz/elasticsearch-head/archive/master.zip ...
Downloading .........................................................DONE
NOTE: Unable to verify checksum for downloaded plugin (unable to find .sha1 or .md5 file to verify)
Installed head into /usr/local/elasticsearch-2.2.0/plugins/head
```


7.安装kibana 
`192.168.50.119` ELK服务器
安装


```
`[root@localhost ELK]# tar -xf kibana-4.4.0-linux-x64.tar.gz  -C /usr/local/
[root@localhost ELK]# cd /usr/local/kibana-4.4.0-linux-x64/

```

配置

```
[root@localhost kibana-4.4.0-linux-x64]# vim config/kibana.yml
elasticsearch.url: "http://192.168.50.119:9200"
server.port: 5601
server.host: "0.0.0.0"
```


启动


```
[root@localhost kibana-4.4.0-linux-x64]# nohup  /usr/local/kibana-4.4.0-linux-x64/bin/kibana > /usr/local/kibana-4.4.0-linux-x64/nohub.out &
```


```
[root@localhost ELK]# netstat -tunpl | grep 5601
tcp        0      0 0.0.0.0:5601                0.0.0.0:*
```


浏览器访问http://192.168.50.119:5601/



![ ](https://www.msbgn.cn/msbgn/ELK3.png)




8.安装logstash-server服务器 


192.168.50.120  服务器


安装jdk和logstash


```
tar -zxf jdk-8u45-linux-x64.tar.gz -C /usr/local/
vim /etc/profile  #设置环境变量
export JAVA_HOME=/usr/local/jdk1.8.0_45
export PATH=$PATH:$JAVA_HOME/bin
export CLASSPATH=.:$JAVA_HOME/lib/tools.jar:$JAVA_HOME/lib/dt.jar:$CLASSPATH
source /etc/profile  #使环境变量生效
```


```
tar –zxf logstash-2.4.0.tar.gz -C /usr/local/
mkdir /usr/local/logstash-2.4.0/conf
```



将redis 中的数据发送到elasticsearch中



```
[root@localhost conf]# cat logstash_server.conf 
input {
    redis {
        port => "6379"
        host => "192.168.50.120"
        data_type => "list"
        key => "logstash-redis"
        type => "redis-input"
   }
}
output {
    elasticsearch {
        hosts => "192.168.50.119"
        index => "logstash-%{+YYYY.MM.dd}"
   }
}
```


9.在Kibanda上创建nginx日志监控视图

![ ](https://www.msbgn.cn/msbgn/ELK4.png)
![ ](https://www.msbgn.cn/msbgn/ELK5.png)
![ ](https://www.msbgn.cn/msbgn/ELK6.png)


下载资料百度云盘地址


http://pan.baidu.com/s/1jHVKRMQ



案列：对nginxaccess日志的采集


logstash 端配置





```

input {
     file {
        path => ["/usr/local/nginx/logs/bc.access.log"]
        type => "nginx_log"
     }
}
output {
     redis{
         host => "192.168.1.33"
         key => 'logstash-redis'
         data_type => 'list'
    }
    stdout {
codec => rubydebug
    }
}
```

redis服务器端logstash


```
input {
    redis {
        port => "6379"
        host => "192.168.1.33"
        data_type => "list"
        key => "logstash-redis"
        type => "redis-input"
   }
}
output {
    elasticsearch {
        hosts => "192.168.1.6"
        user => "es_admin"
        password => "123456"
        index => "logstashqiantwo-%{+YYYY.MM.dd}"
   }
}
```

案列对tomcat的日志采集

logstash 端配置


```
input{
      file {
        path => "/usr/local/tomcatweixin/logs/catalina.out"
        start_position => "beginning"
  }
}
filter {
  grok {
      match => [ "message", "%{COMMONAPACHELOG}" ]
    }
      kv {
                source => "request"
                field_split => "&?"
                value_split => "="
        }
    urldecode {
        all_fields => true
    }
}

output{
    stdout { codec => rubydebug }

    redis{
        host=>"192.168.1.33"
        port=>6379
        data_type=>"list"
        key=>"logstash:redis"
    }

}

```


redis服务器端logstash




```
input {
    redis {
        port => "6379"
        host => "192.168.1.33"
        data_type => "list"
        key=>"logstash:redis"
   }
}
output {
    elasticsearch {
        hosts => ["192.168.1.6:9200"]

        user => "es_admin"
        password => "123456"
        index => "logstasweixin-%{+YYYY.MM.dd}"
   }
}
```

结束
