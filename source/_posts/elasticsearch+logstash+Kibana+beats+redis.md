---
title: elasticsearch+logstash+Kibana+beats+redis安装配置
thumbnail: https://www.msbgn.cn/msbgn/ELKR-log-platform.jpg
categories: ELK
tags: [ELK,redis]
---



### 简介

-------------------
 Logstash是一个开源的用于收集，分析和存储日志的工具。 Kibana4用来搜索和查看Logstash已索引的日志的web接口。这两个工具都基于Elasticsearch。 + Logstash: Logstash服务的组件，用于处理传入的日志。 + Elasticsearch: 存储所有日志 + Kibana 4: 用于搜索和可视化的日志的Web界面，通过nginx反代 + Logstash Forwarder: 安装在将要把日志发送到logstash的服务器上，作为日志转发的道理，通过 lumberjack 网络协议与 Logstash 服务通讯,logstash-forwarder要被beats替代。


-------------------


![kibana-count-log](https://www.msbgn.cn/msbgn/kibana-count-log.png)

<!-- more -->

###   1. 安装elasticsearch

```
[root@node4 ~]# yum localinstall jdk-8u101-linux-x64.rpm 
[root@node4 ~]# wget https://download.elastic.co/elasticsearch/release/org/elasticsearch/distribution/rpm/elasticsearch/2.3.4/elasticsearch-2.3.4.rpm
[root@node4 ~]# yum localinstall elasticsearch-2.3.4.rpm 
[root@node4 ~]# systemctl enable elasticsearch.service 
[root@node4 ~]# mkdir -p /data/elasticsearch/{data,logs}
[root@node4 ~]# chown -R elasticsearch.elasticsearch /data/elasticsearch/
```
elasticsearch配置文件位于ES_HOME/config目录中，yum安装的就在etc目录下。该目录中有两个文件elasticsearch.yml配置elasticsearch不同模块，logging.yml配置elasticsearch日志。配置格式是YAML。如果使用json格式，需要将elasticsearch.yml重命名为elasticsearch.json，同时，需要将文件中的配置参数转换成json格式。\

```
[root@node4 ~]# vim /etc/elasticsearch/elasticsearch.yml
cluster.name: elk-logs
node.name: ${HOSTNAME}
path.data: /data/elasticsearch/data
path.logs: /data/elasticsearch/logs
bootstrap.mlockall: true
bootstrap.max_open_files: true
network.host: 10.199.200.204
http.port: 9200

```

`注:`

数据目录可以设置为多个如：path.data: ["/data/elasticsearch/data1","/data/elasticsearch/data2"] 或者 path.data: /data/elasticsearch/data1, /data/elasticsearch/data2。 如果想以主机名命令节点名称，同时，该服务器上只运行单个elasticsearch实例，可以设置为${HOSTNAME}变量，将从环境变量中获取主机名。也可以设置成${prompt.text}，在启动时，需要键入名称。


```
[root@node4 ~]# systemctl start elasticsearch.service
[root@node4 ~]# curl -XGet 10.199.200.204:9200
    {
      "name" : "Jane Kincaid",
      "cluster_name" : "elasticsearch",
      "version" : {
        "number" : "2.3.4",
        "build_hash" : "e455fd0c13dceca8dbbdbb1665d068ae55dabe3f",
        "build_timestamp" : "2016-06-30T11:24:31Z",
        "build_snapshot" : false,
        "lucene_version" : "5.5.0"
      },
      "tagline" : "You Know, for Search"
    }
```


###  2. 安装kibana


```
[root@node4 ~]# wget https://download.elastic.co/kibana/kibana/kibana-4.5.3-1.x86_64.rpm
[root@node4 ~]# yum localinstall kibana-4.5.3-1.x86_64.rpm
[root@node4 ~]# vim /opt/kibana/config/kibana.yml 
  server.port: 5601
  server.host: "10.199.200.204"
  elasticsearch.url: "http://10.199.200.204:9200"

[root@node4 ~]# systemctl enable kibana.service
[root@node4 ~]# systemctl start kibana.service

```


###  3. 安装插件


```
[root@node4 ~]# /usr/share/elasticsearch/bin/plugin  install mobz/elasticsearch-head
[root@node4 ~]# /usr/share/elasticsearch/bin/plugin install lmenezes/elasticsearch-kopf
[root@node4 ~]# /usr/share/elasticsearch/bin/plugin install  license
[root@node4 ~]# /usr/share/elasticsearch/bin/plugin install  graph
[root@node4 ~]# /usr/share/elasticsearch/bin/plugin install marvel-agent
[root@node4 ~]# /opt/kibana/bin/kibana plugin --install elasticsearch/marvel/latest
[root@node4 ~]# /opt/kibana/bin/kibana plugin --install elasticsearch/graph/latest

浏览器访问
http://10.199.200.204:9200/_plugin/head/
http://10.199.200.204:9200/_plugin/kopf/        
http://10.199.200.204:5601/app/marvel   
http://10.199.200.204:5601/app/kibana

```


###   4. 安装logstash


```

[root@node4 ~]# wget https://download.elastic.co/logstash/logstash/packages/centos/logstash-2.3.4-1.noarch.rpm
[root@node4 ~]# yum localinstall logstash-2.3.4-1.noarch.rpm
[root@node4 ~]# usermod -a -G root logstash 
[root@node4 ~]# service logstash start
[root@node4 ~]# vim /etc/logstash/conf.d/topbeat-redis.conf

input {
    redis {
        host => "127.0.0.1"
        port => "6379"
        data_type => "list"
        key => "topbeat"
        threads => 10
    }

}

output {
    elasticsearch {
        hosts => "10.199.200.204:9200"
        sniffing => true
        index => "topbeat-%{+YYYY.MM.dd}"
        workers => 10
    }

}

[root@node4 ~]# vim /etc/logstash/conf.d/filebeat-redis.conf

input {
    redis {
        host => "127.0.0.1"
        port => "6379"
        data_type => "list"
        key => "filebeat"
        threads => 10
    }

}

output {
    elasticsearch {
        hosts => "10.199.200.204:9200"
        sniffing => true
        index => "filebeat-%{+YYYY.MM.dd}"
        workers => 10
    }

}


[root@node4 ~]# vim /etc/logstash/conf.d/packetbeat-redis.conf

input {
    redis {
        host => "127.0.0.1"
        port => "6379"
        data_type => "list"
        key => "packetbeat"
        threads => 10
    }

}

output {
    elasticsearch {
        hosts => "10.199.200.204:9200"
        sniffing => true
        index => "packetbeat-%{+YYYY.MM.dd}"
        workers => 10
    }

}

```

验证配置文件

```
[root@node4 ~]# /opt/logstash/bin/logstash --configtest -f /etc/logstash/conf.d/*

```


###  5. 安装redis

redis这里不详细说明，最好是用集群，由于这里是试验所以就单节点。


```
[root@node4 ~]# yum -y install epel-release
[root@node4 ~]# yum -y install redis
[root@node4 ~]# vim /etc/redis
    bind 127.0.0.1 10.199.200.204
    maxmemory 4G
[root@node4 ~]# systemctl enable redis.service
[root@node4 ~]# systemctl start redis.service

```


###  6. 客户端配置

####  6.1 安装filebeat


```
[root@node1 ~]# yum localinstall https://download.elastic.co/beats/filebeat/filebeat-1.2.3-x86_64.rpm
[root@node1 ~]# vim /etc/filebeat/filebeat.yml  
filebeat:
  prospectors:
    -
      paths:
        - "/opt/gis/tomcat/logs/catalina.out"
      fields:
        type: gis-tomcat-logs
output:
  redis:
    host: "10.199.200.204"
    port: 6379
    save_topology: true
    index: "filebeat"
    #redis keyname
    db: 0
    db_topology: 1
    reconnect_interval: 1

[root@node1 ~]# curl -XPUT 'http://10.199.200.204:9200/_template/filebeat?pretty' -d@/etc/filebeat/filebeat.template.json   
[root@node1 ~]# service filebeat restart

```


####  6.2 安装topbeat



```

[root@node1 ~]# yum localinstall https://download.elastic.co/beats/topbeat/topbeat-1.2.3-x86_64.rpm
[root@node1 ~]# curl -XPUT 'http://10.199.200.204:9200/_template/topbeat' -d@/etc/topbeat/topbeat.template.json
[root@node1 ~]# vim /etc/filebeat/topbeat.yml

input:
  # In seconds, defines how often to read server statistics
  period: 10

  # Regular expression to match the processes that are monitored
  # By default, all the processes are monitored
  procs: [".*"]

  # Statistics to collect (all enabled by default)
  stats:
    # per system statistics, by default is true
    system: true

    # per process statistics, by default is true
    process: true

    # file system information, by default is true
    filesystem: true

    # cpu usage per core, by default is false
    cpu_per_core: false


output:
  redis:
    # Set the host and port where to find Redis.
    host: "10.199.200.204"
    port: 6379

    # Uncomment out this option if you want to store the topology in Redis.
    # The default is false.
    save_topology: true

    # Optional index name. The default is topbeat and generates topbeat keys.
    index: "topbeat"

    # Optional Redis database number where the events are stored
    # The default is 0.
    db: 0

    # Optional Redis database number where the topology is stored
    # The default is 1. It must have a different value than db.
    db_topology: 1

    # Optional password to authenticate with. By default, no
    # password is set.
    # password: ""

    # Optional Redis initial connection timeout in seconds.
    # The default is 5 seconds.
    timeout: 5

    # Optional interval for reconnecting to failed Redis connections.
    # The default is 1 second.
    reconnect_interval: 1

[root@node1 ~]# service topbeat start


```


####  6.3 安装packetbeat


```

[root@node1 ~]# yum localinstall https://download.elastic.co/beats/packetbeat/packetbeat-1.2.3-x86_64.rpm   
[root@node1 ~]# curl -XPUT 'http://10.199.200.204:9200/_template/packetbeat' -d@/etc/packetbeat/packetbeat.template.json
[root@node1 ~]# vim /etc/filebeat/packetbeat.yml
protocols:
  dns:
    ports: [53]

    include_authorities: true
    include_additionals: true

  http:
    ports: [80, 8080, 8081, 5000, 8002]

  memcache:
    ports: [11211]

  mysql:
    ports: [3306]

  pgsql:
    ports: [5432]

  redis:
    ports: [6379]

  thrift:
    ports: [9090]

  mongodb:
    ports: [27017]

 tomcat:
    ports: [8080]

output:
  redis:
    # Set the host and port where to find Redis.
    host: "10.199.200.204"
    port: 6379

    # Uncomment out this option if you want to store the topology in Redis.
    # The default is false.
    save_topology: true

    # Optional index name. The default is packetbeat and generates packetbeat keys.
    index: "packetbeat"

    # Optional Redis database number where the events are stored
    # The default is 0.
    db: 0

    # Optional Redis database number where the topology is stored
    # The default is 1. It must have a different value than db.
    db_topology: 1

    # Optional password to authenticate with. By default, no
    # password is set.
    # password: ""

    # Optional Redis initial connection timeout in seconds.
    # The default is 5 seconds.
    timeout: 5

    # Optional interval for reconnecting to failed Redis connections.
    # The default is 1 second.
    reconnect_interval: 1
    [root@node1 ~]# service packetbeat start
```


####  7. 加载kibana Dashboards


```

官方提供了一些仪表盘样本方便我们查看监视，项目地址：https://github.com/elastic/beats-dashboards
加载方法如下：

[root@node1 ~]# wget http://download.elastic.co/beats/dashboards/beats-dashboards-1.0.0.tar.gz
[root@node1 ~]# tar xzvf beats-dashboards-1.0.0.tar.gz
[root@node1 ~]# cd beats-dashboards-1.0.0/
[root@node1 ~]#./load.sh http://10.199.200.204:9200

https://github.com/packetb-old

```


结束
