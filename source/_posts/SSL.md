---
title: StartSSL免费https证书申请安装
thumbnail: https://www.msbgn.cn/msbgn/startssl-index.png
categories: SSL
tags: [科技,StartSSL]
---
StartSSL是比较早的免费https提供商，由于之前的沃通SSL证书被禁用后，目前只有StartSSL免费提供着第三方证书，当然你也可以选择购买StartSSL的收费版证书，由于本人自己测试以及使用着免费的StartSSL证书，所以废话不多说直接说明StartSSL的配置安装说明。

第一、StartSSL官方网站
https://www.startssl.com/

![startssl](https://www.msbgn.cn/msbgn/startssl-index.png)
*打开startssl官网新用户进行注册，老用户直接跳过注册，进行登录。*
<!-- more -->
第二、新注册StartSSL账号
![startssl](https://www.msbgn.cn/msbgn/startssl-Signup.png)
根据说明选择国家，填写注册自己的邮箱进行注册操作，验证码会发送至对应的邮箱。(*按照注册流程下一步进行操作，楼主是已经注册的用户，直接点击LOGIN即可*)

第三、选择SSL证书的应用类型、选择验证方式
![startssl](https://www.msbgn.cn/msbgn/startss-validation.png)
这里我们使用的是域名的方式进行验证
![startssl](https://www.msbgn.cn/msbgn/validation-code.png)
填写好需要验证的域名或者子域名，然后点击确定，进行校验，选择需要验证的邮箱，并且受到邮箱的验证码填入validation code 点击验证

第四、申请免费域名SSL证书
![startssl](https://www.msbgn.cn/msbgn/startss-DV.png)
选择要申请的证书的合适类型，这里我们选择，Free User (Not Validated)，DV SSLCertificate证书类型
![startssl](https://www.msbgn.cn/msbgn/startss-CRS.png)
填写申请的域名(最多支持5个子域名免费版的证书，否则需要购买更多)，填写CSR，有两种方式：
1.选择下载tartComTool.exe 客户端去生成CSR
2.使用命令行生成openssl req -newkey rsa:2048 -keyout yourname.key -out yourname.csr
选择由IE Browser获得pfx格式证书，点击提交

第五、StartSSL免费SSL证书的下载和使用
![startssl](https://www.msbgn.cn/msbgn/startss-cests.png)
打开Certificate List这时候已经生成了刚刚申请的证书，说明下，这里我们默认注册的证书是1年如果需要增加证书使用年限在生成csr的时候修改证书的使用年限。
![startssl](https://www.msbgn.cn/msbgn/startss-yingyong.png)
证书的应用场景比较广泛比如nginx、ISS、ApacheServer等多服务器上部署，至此证书的申请使用已经完成，接下来就是各位部署到自己对应的服务器使用了，有兴趣的朋友可以自己申请部署试一试。



