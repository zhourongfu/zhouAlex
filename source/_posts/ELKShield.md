---
title:  ELK添加Shield插件管理权限
thumbnail: https://www.msbgn.cn/msbgn/ELK1.png
categories: ELK
tags: [语言,ELK]
---
-------------------

![ ](https://www.msbgn.cn/msbgn/ELK1.png)




<!-- more -->



###   Shield
Shield作为插件安装到elasticsearch中，一旦安装，插件会拦截入栈API调用，以强制执行身份验证和授权，插件还可以使用Secure Sockets Layer/Transport Layer Security (SSL/TLS)为来自网络和elasticsearch的网络流量提供加密，该插件还是用API拦截层，该层使身份验证和授权能够提供审计日志记录功能。

###  为什么要使用Shield

ELK是一个开源的日志分析平台，可以对各类日志进行分析和研究，但是有一个缺陷就是无法对用户身份进行验证，造成的直接后果就是任何人都可以访问和查看数据，从而我们需要这样一个插件来对elk的访问做一个权限控制，elastic官方给出的是使用shield，当然也有开源的产品替代search-guard，下边我们就先来看下elastic官方给出的shield在elk中的使用。


###  环境说明

Centos6.5：16.04
Elasticsearch：2.4.0
Logstash：2.4.0
Kibana：4.6.1
Java：1.8.0_101


###  安装Shield到Elasticsearch


```
bin/plugin install license
bin/plugin install shield

```

测试安装是否成功
Curl -X GET http://loclahost:9200


如果报错类似于下面这样，说明安装成功
{"error":{"root_cause":[{"type":"security_exception","reason":"missing authentication token 


创建一个es_admin用户，用来登录elasticsearch,kiabna超级管理员，按提示输入密码


```
cd /usr/local/logstash-2.4.0/
bin/shield/esusers useradd es_admin -r admin

```

2.安装Logstash（日志收集、分析，并将其存储供以后使用）
这里安装遇到了下载的坑，最好去使用迅雷下载安装包	


```
wget https://download.elastic.co/logstash/logstash/logstash-2.4.0.tar.gz
tar –zxf logstash-2.4.0.tar.gz -C /usr/local/

```



用新建的用户名进行测试

curl -u es_admin:password -X GET http://localhost:9200

```
{
  "name" : "Mauvais",
  "cluster_name" : "elasticsearch",
  "version" : {
    "number" : "2.4.0",
    "build_hash" : "ce9f0c7394dee074091dd1bc4e9469251181fc55",
    "build_timestamp" : "2016-08-29T09:14:17Z",
    "build_snapshot" : false,
    "lucene_version" : "5.5.2"
  },
  "tagline" : "You Know, for Search"
}
OK


```

###  Kibana添加用户权限


为了给kibana加用户访问权限，同时为让kibana可以访问elasticsearch的数据，需要给kibana添加kibana server密码
创建一个用户用来访问elasticsearch


```
/usr/local/logstash-2.4.0
bin/shield/esusers useradd kibana4-server -r kibana4_server
Enter new password:  输入密码
Retype new password:
```


接下来是比较重要的一个环节


###  利用openssl创建 server.key ，server.crt ，serverpem

切换到kibana的目录，创建server_ssl目录，进入该目录创建所需的文件
Key生成


```
openssl genrsa -des3 -out server.key 2048
openssl rsa -in server.key -out server.key
```

生成CA的Crt：

```
openssl req -new -x509 -key server.key -out ca.crt -days 3650
openssl req -new -key server.key -out server.csr
```


Crt生成：


```
openssl x509 -req -days 3650 -in server.csr -CA ca.crt -CAkey server.key -CAcreateserial -out server.crt
cat server.key server.crt > server.pem
```


最后在server_sll目录下有这些文件


```
ca.crt ca.srl server.crt server.csr server.key server.pem
```


###  配置Kibana



进入kibana的安装目录，执行：vim config/kibana.yml


####   指定访问elasticsearch的用户和密码

```
elasticsearch.username: "kibana4-server"
elasticsearch.password: "kibana"
```


####  设置server.ssl.key /server.ssl.crt/serve.pem


```
server.ssl.cert: /usr/local/elasticsearch-2.2.0/server.crt
server.ssl.key: /usr/local/elasticsearch-2.2.0/server.key
elasticsearch.ssl.ca: /usr/local/elasticsearch-2.2.0/server.pem
```


####  设置elasticsearch.url

```
elasticsearch.url: http://192.168.10.110:9200
```


####  设置shield.encryptionKey

```
hield.encryptionKey: "something_at_least_32_characters"
```


####  设置shield.sessionTimeout


```
shield.sessionTimeout: 600000
```


###  在kibana中安装Shield

```
bin/kibana plugin –install kibana/shield/latest
```

如果提示安装不了，请去官网下载复制到kibana目录
我分享的云盘地址  http://pan.baidu.com/s/1hrThpUO  



###  Roles.yml文件的配置


roles.yml的位置

```
/usr/local/elasticsearch-2.4.0/config/shield/roles.yml
```

我们拿其中一个举例说一下



```
#The required permissions for the kibana 4 server
kibana4_server:   #用户组
  cluster:     
      - monitor
  indices:        #权限
    - names: '.kibana*'  #索引名称
      privileges:        #用户可对该索引执行的操作
        - all            #这里是给隶属于kibana4_server的用户所有的执行权限
    - names: '.reporting-*'
      privileges:
        - all
编辑roles.yml，添加用户组 my_kibana_user 用户组，这里只给了read的权限
my_kibana_user:
  cluster:
      - monitor
  indices:
    - names: 'logstash-*'
      privileges:
        - view_index_metadata
        - read
    - names: '.kibana*' 
      privileges:
        - manage
        - read
        - index

```



接下来创建一个kibana属于my_kibana_user用户组


```
Cd /usr/local/elasticsearch-2.4.0/config/shield/

/usr/local/elasticsearch-2.4.0/bin/shield/esusers useradd kibana -r my_kibana_user
/ usr / local /elasticsearch-2.4.0/bin/shield/esusers list
```


###  logstash配置Shield



创建一个logstash用户，用来连接elasticsearch





```
Cd /usr/ local /elasticsearch-2.4.0 
bin/shield/esusers useradd logstash -r logstash
Enter new password:
Retype new password:
```


接着我们依旧测试Rsyslog发送日志
在logstash目录下便捷logstash-indexer.conf，加入




```
input {
  file {
     type => "syslog"
     path => ["/var/log/messages", "/var/log/secure" ]
  }
  syslog {
     type => "syslog"
     port => "5542"
  }
}
output {
  elasticsearch {
	        hosts => 192.168.1.3"
           
			user => " es_admin "   #logstash与elasticsearch交互的用户名 这个用户名如果你不是单台部署的话用户名最好写
			password => "123456"   #这个是logsatsh对应的密码

}
  stdout { codec => rubydebug }
}


```


###  配置logstash-indexer的配置文件

###   启动ELK，进行测试


至此我们的整体环境已经编辑OK，使用es_admin登录，该用户所属用户组为admin，对elk有所有的执行权限



输入 https://192.168.10.110:5601


![ ](https://www.msbgn.cn/msbgn/shied1.png)



遇到问题是权限访问不到，后面配置logstash账号的时候使用es_admin就OK了


```
[403] {"error":{"root_cause":[{"type":"security_exception","reason":"action [indices:data/write/bulk] is unauthorized for user [logstash]"}],"type":"security_exception","reason":"action [indices:data/write/bulk] is unauthorized for user [logstash]"},"status":403} {:class=>"Elasticsearch::Transport::Transport::Errors::Forbidden", :backtrace=>["/usr/local/logstash/vendor/bundle/jruby/1.9/gems/elasticsearch-transport-1.1.0/lib/elasticsearch/transport/transport/base.rb:201:in `__raise_transport_error'", "/usr/local/logstash/vendor/bundle/jruby/1.9/gems/elasticsearch-transport-1.1.0/lib/elasticsearch/transport/transport/base.rb:312:in `perform_request'", "/usr/local/logstash/vendor/bundle/jruby/1.9/gems/elasticsearch-transport-1.1.0/lib/elasticsearch/transport/transport/http/manticore.rb:67:in `perform_request'", "/usr/local/logstash/vendor/bundle/jruby/1.9/gems/elasticsearch-transport-1.1.0/lib/elasticsearch/transport/client.rb:128:in `perform_request'", "/usr/local/logstash/vendor/bundle/jruby/1.9/gems/elasticsearch-api-1.1.0/lib/elasticsearch/api/actions/bulk.rb:93:in `bulk'", "/usr/local/logstash/vendor/bundle/jruby/1.9/gems/logstash-output-elasticsearch-2.7.1-java/lib/logstash/outputs/elasticsearch/http_client.rb:53:in `non_threadsafe_bulk'", "/usr/local/logstash/vendor/bundle/jruby/1.9/gems/logstash-output-elasticsearch-2.7.1-java/lib/logstash/outputs/elasticsearch/http_client.rb:38:in `bulk'", "org/jruby/ext/thread/Mutex.java:149:in `synchronize'", "/usr/local/logstash/vendor/bundle/jruby/1.9/gems/logstash-output-elasticsearch-2.7.1-java/lib/logstash/outputs/elasticsearch/http_client.rb:38:in `bulk'", "/usr/local/logstash/vendor/bundle/jruby/1.9/gems/logstash-output-elasticsearch-2.7.1-java/lib/logstash/outputs/elasticsearch/common.rb:172:in `safe_bulk'", "/usr/local/logstash/vendor/bundle/jruby/1.9/gems/logstash-output-elasticsearch-2.7.1-java/lib/logstash/outputs/elasticsearch/common.rb:101:in `submit'", "/usr/local/logstash/vendor/bundle/jruby/1.9/gems/logstash-output-elasticsearch-2.7.1-java/lib/logstash/outputs/elasticsearch/common.rb:86:in `retrying_submit'", "/usr/local/logstash/vendor/bundle/jruby/1.9/gems/logstash-output-elasticsearch-2.7.1-java/lib/logstash/outputs/elasticsearch/common.rb:29:in `multi_receive'", "org/jruby/RubyArray.java:1653:in `each_slice'", "/usr/local/logstash/vendor/bundle/jruby/1.9/gems/logstash-output-elasticsearch-2.7.1-java/lib/logstash/outputs/elasticsearch/common.rb:28:in `multi_receive'", "/usr/local/logstash/vendor/bundle/jruby/1.9/gems/logstash-core-2.4.0-java/lib/logstash/output_delegator.rb:130:in `worker_multi_receive'", "/usr/local/logstash/vendor/bundle/jruby/1.9/gems/logstash-core-2.4.0-java/lib/logstash/output_delegator.rb:114:in `multi_receive'", "/usr/local/logstash/vendor/bundle/jruby/1.9/gems/logstash-core-2.4.0-java/lib/logstash/pipeline.rb:301:in `output_batch'", "org/jruby/RubyHash.java:1342:in `each'", "/usr/local/logstash/vendor/bundle/jruby/1.9/gems/logstash-core-2.4.0-java/lib/logstash/pipeline.rb:301:in `output_batch'", "/usr/local/logstash/vendor/bundle/jruby/1.9/gems/logstash-core-2.4.0-java/lib/logstash/pipeline.rb:232:in `worker_loop'",
```



然后普通账号登录访问就是如下图


![ ](https://www.msbgn.cn/msbgn/shied2.png)
