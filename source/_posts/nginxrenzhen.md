---
title:  nginx配置SSL双向认证
thumbnail: https://www.msbgn.cn/msbgn/gup12ks.png
categories: nginx
tags: [ssl,nginx]
---
![nginxSSL](https://www.msbgn.cn/msbgn/gup12ks.png)


**https双向认证** 双向认证SSL 协议的具体通讯过程，这种情况要求服务器和客户端双方都有证书。 单向认证SSL 协议不需要客户端拥有CA证书，以及在协商对称密码方案，对称通话密钥时，服务器发送给客户端的是没有加过密的（这并不影响SSL过程的安全性）密码方案。这样，双方具体的通讯内容，就是加密过的数据。如果有第三方攻击，获得的只是加密的数据，第三方要获得有用的信息，就需要对加密的数据进行解密，这时候的安全就依赖于密码方案的安全。而幸运的是，目前所用的密码方案，只要通讯密钥长度足够的长，就足够的安全。这也是我们强调要求使用128位加密通讯的原因。
一般Web应用都是采用单向认证的，原因很简单，用户数目广泛，且无需做在通讯层做用户身份验证，一般都在应用逻辑层来保证用户的合法登入。但如果是企业应用对接，情况就不一样，可能会要求对客户端（相对而言）做身份验证。这时就需要做双向认证。
<!-- more -->

## 1、安装nginx

参考《nginx安装》：https://www.msbgn.cn/2016/11/01/nginxbianyi/

-------------------

## 2、使用openssl实现证书中心

由于是使用openssl架设私有证书中心，因此要保证以下字段在证书中心的证书、服务端证书、客户端证书中都相同

### 代码块
``` python
Country Name
 State or Province Name
 Locality Name
 Organization Name
 Organizational Unit Name
```
编辑证书中心配置文件

``` 
vim /etc/pki/tls/openssl.cnf
[ ca ]
default_ca      = CA_default            # The default ca section
####################################################################
[ CA_default ]
dir             = /etc/pki/CA           # Where everything is kept
certs           = $dir/certs            # Where the issued certs are kept
crl_dir         = $dir/crl              # Where the issued crl are kept
database        = $dir/index.txt        # database index file.
#unique_subject = no                    # Set to 'no' to allow creation of                                      # several ctificates with same subject.
new_certs_dir   = $dir/newcerts         # default place for new certs.
certificate     = $dir/cacert.pem       # The CA certificate
serial          = $dir/serial           # The current serial number
crlnumber       = $dir/crlnumber        # the current crl number                                      # must be commented out to leave a V1 CRL
crl             = $dir/crl.pem          # The current CRL
private_key     = $dir/private/cakey.pem# The private key
RANDFILE        = $dir/private/.rand    # private random number file
x509_extensions = usr_cert              # The extentions to add to the cert
# Comment out the following two lines for the "traditional"
# (and highly broken) format.
name_opt        = ca_default            # Subject Name options
cert_opt        = ca_default            # Certificate field options                                                                    
```
创建证书私钥
```
cd /etc/pki/CA/private
(umask 077;openssl genrsa -out cakey.pem 2048)
```
生成自签证书（我这里申请的自签证书为10年，如有报错，请检查vim /etc/pki/tls/openssl.cnf）
```
cd /etc/pki/CA/
openssl req -new -x509 -key private/cakey.pem -out cacert.pem -days=3655
```

## 3、创建服务器证书
```
mkdir /usr/local/nginx/ssl
 cd /usr/local/nginx/ssl
 (umask 077;openssl genrsa -out nginx.key 1024)
 openssl req -new -key nginx.key -out nginx.csr
 openssl ca -in nginx.csr -out nginx.crt -days=3650
```

##4、创建客户端浏览器证书

```
(umask 077;openssl genrsa -out client.key 1024)
 openssl req -new -key client.key -out client.csr
 openssl ca -in client.csr -out client.crt -days=3650
 将文本格式的证书转换成可以导入浏览器的证书，最好是设置客户端证书访问密码，并且报存好客户端证书p12
 openssl pkcs12 -export -clcerts -in client.crt -inkey client.key -out client.p12
```
## 5、配置nginx服务器验证
```
vim /usr/local/nginx/conf/nginx.conf
 ssl on;
 ssl_certificate         /usr/local/nginx/ssl/nginx.crt;
 ssl_certificate_key     /usr/local/nginx/ssl/nginx.key;
 ssl_client_certificate  /usr/local/nginx/ssl/cacert.pem;
 ssl_session_timeout     5m;
 #ssl_verify_client       on;                         服务器验证客户端，暂时不开启，让没有证书的客户端可以访问，先完成单向验证
 ssl_protocols           SSLv2 SSLv3 TLSv1;
 
```
## 6、打开浏览器安装客户端证书
![client1](https://www.msbgn.cn/msbgn/toiss.png)
![client2](https://www.msbgn.cn/msbgn/mima1.png)
点击安装生成的证书下一步输入证书密码(密码就是之前生成证书时候设置的密码)，下一步安装成功。
## 7、验证证书的使用
 
### 没有导入证书访问
![client3](https://www.msbgn.cn/msbgn/catsd.png)
没有导入浏览器安装的证书显示为400
### 导入证书访问
![client4](https://www.msbgn.cn/msbgn/nhkk.png)
注意:安装证书的时候，记得安装完成证书关闭浏览器重新打开访问。
感谢阅读这份帮助文档。请点击右上角开启分享体验吧。
