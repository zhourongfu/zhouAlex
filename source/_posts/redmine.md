---
title: redmine的安装部署
thumbnail: https://www.msbgn.cn/msbgn/redmine1.jpg
categories: redmine
tags: [redmine,redmine]
---

-------------------



![redmine](https://www.msbgn.cn/msbgn/redmine2.png)
###  安装

-------------------
操作系统：centos6.3
redmine版本：2.3
虚拟机IP：192.168.1.44
mysql版本：mysql-5.1.69 
nginx版本: nginx/1.4.2
ruby版本：ruby 1.9.3p392 (2013-02-22 revision 39386) [i686-linux]
rubygems: 1.8.23

-------------------


<!-- more -->

###   安装ruby with libyaml

```
wget https://pyyaml.org/download/libyaml/yaml-0.1.4.tar.gz
tar -zxvf yaml-0.1.4.tar.gz
cd yaml-0.1.4
./configure --prefix=/usr/local
make && make install
```

###   安装ruby

```
wget ftp://ftp.ruby-lang.org/pub/ruby/1.9/ruby-1.9.3-p392.tar.gz
tar -zxvf ruby-1.9.3-p392
./configure --prefix=/usr/local --enable-shared-disable-install-doc-with-opt-dir=/usr/local/lib
make && make install
安装完成后查看ruby 版本：
ruby -v 
```

###  安装mysqlserver

```
yum -y install mysql-serverz
```

###  redmine安装

```
https://rubyforge.org/frs/?group_id=1850
mkdir data
cd data
mkdir web
cd web
tar -zxvf redmine-2.3.2.tar.gz
```


###   配置mysql数据库
```
CREATE DATABASE redmine CHARACTER SET utf8;
GRANT ALL PRIVILEGES ON redmine.* TO 'redmine'@'localhost' IDENTIFIED BY 'my_password';
```

###  数据库连接配置

```
cd /data/web/redmine-2.3.2
cp config/database.yml.example config/database.yml
打开database.yml
production:
  adapter: mysql2
  database: redmine
  host: localhost
  username: redmine
  password: my_password
```


###  安装依赖的bundler

```
gem install bundler
bundle install --without development test
bundle install --without development test rmagick
```

###  生成session存储加密信息和数据表

```
rake generate_secret_token
```

###  创建数据库对象 

```
RAILS_ENV=production rake db:migrate
```


###  数据库默认数据集

```
RAILS_ENV=production rake redmine:load_default_data
RAILS_ENV=production REDMINE_LANG=fr rake redmine:load_default_data
```

###  文件日志权限

```
groupadd redmine
useradd -g redmine redmine
mkdir -p /data/web/redmine-2.3.2 /public/plugin_assets
sudo chown -R redmine:redmine files log tmp public/plugin_assets
sudo chmod -R 755 files log tmp public/plugin_assets
```


###  测试安装


```
ruby script/rails server webrick -e production
login: admin
password: admin
http:localhost:3000
```

## 安装方式2
### 使用rbenv安装ruby

```
sudo apt-get install git

git clone git://github.com/sstephenson/rbenv.git ~/.rbenv
echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bash_profile
echo 'eval "$(rbenv init -)"' >> ~/.bash_profile
### 安装 ruby-build
mkdir -p ~/.rbenv/plugins
cd ~/.rbenv/plugins
git clone git://github.com/sstephenson/ruby-build.git

source ~/.bash_profile

```

## 安装 Ruby

```
 sudo apt-get install build-essential autoconf automake bison libtool \
openssl libreadline6 libreadline6-dev curl zlib1g zlib1g-dev libssl-dev \
libyaml-dev libsqlite3-dev sqlite3 libxml2-dev libxslt-dev libc6-dev ncurses-dev

rbenv install 1.9.3-p125
 rbenv rehash

 ```
## 安装mysql

```
sudo apt-get install mysql-server
```
h3. 下载安装redmine 
wget http://www.redmine.org/releases/redmine-2.4.1.tar.gz


```
tar -zxvf  redmine-2.4.1.tar.gz
sudo vi config/ databases.yml


CREATE DATABASE redmine CHARACTER SET utf8;
CREATE USER 'redmine'@'localhost' IDENTIFIED BY 'my_password';
GRANT ALL PRIVILEGES ON redmine.* TO 'redmine'@'localhost';

gem install bundler
bundle install --without development test
```
(报错1)
解决方式：sudo apt-get install libmysql-ruby libmysqlclient-dev
（报错2）
解决方式：gem install net-ldap --version 0.3.1
（报错3）
解决方式：sudo apt-get install imagemagick libmagickwand-dev


```
bundle install --without development test rmagick
rake generate_secret_token
RAILS_ENV=production rake db:migrate
RAILS_ENV=production rake redmine:load_default_data

mkdir -p tmp tmp/pdf public/plugin_assets
sudo chown -R redmine:redmine files log tmp public/plugin_assets
sudo chmod -R 755 files log tmp public/plugin_assets
ruby script/rails server webrick -e production
```



-------------------
