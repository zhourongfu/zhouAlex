---
title: python 的文件操作
categories: Python
tags: [语言,Python]
---
-------------------



###  对文件操作流程
1.打开文件，得到文件句柄并赋值给一个变量
2.通过句柄对文件进行操作
3.关闭文件 

现在有如下文件：alexzhou.txt

```
伶仃九泉挂相思，寂寞百载谁曾知。
三世回眸两相望，几成追忆几成痴。
血染江山碧如画，凭栏追忆经年别。
      多情应是无情月，青丝白雪，化作千千结。
忍听残鸣，望断秋月，心与谁连。
红尘凄切，繁华尽落，亦殇恋。
此情，君欲有语，均欲无音。
此景，柔情万缕，斜影千丝。
忆往昔，与伊伴，思卿碎碎念。

```


<!-- more -->


```
#coding=utf-8
import  random

f = open("alexzhou")#打开文件
'''
for i in f.readlines():#读取文件多行
     # print i,#输出文件
'''
'''
fis = f.readline()
print 'fisL:',fis#读取一行
print '我是分割线'.center(50,'-')#分隔线
data = f.read()#读取剩下的文件内容
print data
f.close#关闭文件
'''
with open('alexzhou') as f:#直接打开文件并且赋值文件名为f
    fis = f.readline()#读取文件一行
    print fis
    print '我是分隔线'.center(50,'-')#分隔线
    for i in f.readlines():#读取剩下的文件多行,优点是不需要关闭文件
        print i,

```
打开文件的模式有：

`•r`，只读模式（默认）。
`•w`，只写模式。【不可读；不存在则创建；存在则删除内容；】
`•a`，追加模式。【可读；   不存在则创建；存在则只追加内容；】
`"+"` 表示可以同时读写某个文件

`•r+`，可读写文件。【可读；可写；可追加】
`•w+`，写读
`•a+`，同a
`"U"`表示在读取时，可以将 \r \n \r\n自动转换成 \n （与 r 或 r+ 模式同使用）

`•rU`
`•r+U`
`"b"`表示处理二进制文件（如：FTP发送上传ISO镜像文件，linux可忽略，windows处理二进制文件时需标注）

`•rb`
`•wb`
`•ab`


其它语法

with语句

为了避免打开文件后忘记关闭，可以通过管理上下文，即：

```
 with open('log','r') as f:
 
```
 
如此方式，当with代码块执行完毕时，内部会自动关闭并释放文件资源。

在Python 2.7 后，with又支持同时对多个文件的上下文进行管理，即：

```

with open('log1') as obj1, open('log2') as obj2:
    pass
```


## 字符编码与转码

###  需知:
1.在python2默认编码是ASCII, python3里默认是utf-8
2.unicode 分为 utf-32(占4个字节),utf-16(占两个字节)，utf-8(占1-4个字节)， so utf-8就是unicode
3.在py3中encode,在转码的同时还会把string 变成bytes类型，decode在解码的同时还会把bytes变回string

```
#-*-coding:utf-8-*-
'''
__author__ = 'Alex zhou'

import sys
print(sys.getdefaultencoding())


msg = "我爱北京天安门"
msg_gb2312 = msg.decode("utf-8").encode("gb2312")
gb2312_to_gbk = msg_gb2312.decode("gbk").encode("gbk")

print(msg)
print(msg_gb2312)
print(gb2312_to_gbk)
'''

#-*-coding:gb2312 -*-   #这个也可以去掉
__author__ = 'Alex zhou'

import sys
print(sys.getdefaultencoding())


msg = "我爱北京天安门"
#msg_gb2312 = msg.decode("utf-8").encode("gb2312")
msg_gb2312 = msg.encode("gb2312") #默认就是unicode,不用再decode,喜大普奔
gb2312_to_unicode = msg_gb2312.decode("gb2312")
gb2312_to_utf8 = msg_gb2312.decode("gb2312").encode("utf-8")

print(msg)
print(msg_gb2312)
print(gb2312_to_unicode)
print(gb2312_to_utf8)

```
