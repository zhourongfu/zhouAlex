---
title: 我和python的第一次接触
thumbnail: https://www.msbgn.cn/msbgn/python01.jpg
categories: python
tags: [语言,python]
---


## Python诞生
Python是著名的'龟叔`Guido van Rossum(吉多·范罗苏`在1989年圣诞节期间，为了打发无聊的圣诞
节而编写的一个编程语言。

![python01](https://www.msbgn.cn/msbgn/python01.jpg)

<!-- more -->

Python语法很多来自C，但又受到ABC语言的强烈影响。来自ABC语言的一些规定直到今天还富有
争议比如强制缩进。但这些语法规定让Python容易读。
Guido van Rossum著名的一句话就是`Life is short, you need Python`，译为：`人生苦短，我用Python.`
![python02](https://www.msbgn.cn/msbgn/python02.jpg)

截至到目前`2016年4月28日`，`Python在Tiobe`的排名还是很靠前的，而且近几年来说Python上升
的趋势已经超过PHP紧随C#。
![python03](https://www.msbgn.cn/msbgn/python03.png)
查询网站：http://www.tiobe.com/tiobe_index?page=index
### Python实现方式
Python身为一门编程语言，但是他是有多种实现方式的，这里的实现指的是符合Python语言规范的Python解释程序以及标准库等。
* `Python的实现方式主要分为三大类`
1. Cpython
2. PyPy
3. Jpython、IronPython等等类似的实现方式
* `CPython`

 当Python执行代码的时候，会启用一个Python解释器，将源码(.py)文件读取到内存当中，然后编译成字节码(.pyc)文件，最后交给Python的虚拟机逐行解释并执行其内容，然后释放内存，退出程序。
 
 当第二次在执行当前程序的时候，会现在当前目录下寻找有没有同名的pyc文件，如果找到了，则直接进行运行，否则重复上面的工作。

	pyc文件的目的其实就是为了实现代码的重用，为什么这么说呢？因为Python认为只要是import导入过来的文件，就是可以被重用的，那么他就会将这个文件编译成pyc文件。
	
	python会在每次载入模块之前都会先检查一下py文件和pyc文件的最后修改日期，如果不一致则重新生成一份pyc文件，否则就直接读取运行。
* `PyPy`

	Python（RPython Python）的Python实现版本，原理是这样的，PyPy运行在CPython（或者其它实现）之上，用户程序运行在PyPy之上。它的一个目标是成为Python语言自身的试验场，因为可以很容易地修改PyPy解释器的实现（因为它是使用Python写的），PyPy的代码性能是Cpython的五倍以上。
* 性能对比图

![python05](https://www.msbgn.cn/msbgn/python005.png)

* Jython
ython是个Python的一种实现方式，Jython编译Python代码为Java字节码，然后由JVM（Java虚拟机）执行，这意味着此时Python程序与Java程序没有区别，只是源代码不一样。此外，它能够导入和使用任何Java类像Python模块。

* IronPython
IronPython是Python的C#实现，并且它将Python代码编译成C#中间代码（与Jython类似），然后运行，它与.NET语言的互操作性也非常好。

### 安装Python
####  Windows安装Python2.7.11

* 下载Python软件
64位下载地址：https://www.python.org/ftp/python/2.7.11/python-2.7.11.amd64.msi
32位下载地址：https://www.python.org/ftp/python/2.7.11/python-2.7.11.msi

* 安装Python软件
下载下来之后双击安装，安装时一路下一步下一步即可，默认的安装路径是：C:\Python27

* 配置环境变量
`右键计算机` —> `属性` —> `高级系统设置` —> `高级` —> `环境变量 —> `在第二个内容框中找到变量名为Path的一行，双击 —> Python安装目录追加到变值值中，`用;分割`
如图：
![python06](https://www.msbgn.cn/msbgn/python06.png)

如：原来的值``;C:\python27，切记前面有分号,Windows 10安装后默认会把环境变量配置好。

* cmd测试
`win+r`调出运行窗口，然后输入`cmd`

![python07](https://www.msbgn.cn/msbgn/pythond07.png)

在`cmd`窗口中输入`python`指令
![python08](https://www.msbgn.cn/msbgn/python08.png)

如果你得到的结果和我一样，那么你就安装好了`windows`下的`python`环境

####  `CentOS`升级到`Python2.7.11`

CentOS6.7默认自带Python2.6.6版本，如果你不需要升级到2.7.11请跳过这段，如果需要升级请继续往下看

* 下载python2.7.11源码包
下载地址：https://www.python.org/ftp/python/2.7.11/Python-2.7.11.tgz


* 安装开发工具包
```
# yum -y groupinstall "debugging Tools"
```

* 编译安装
```
# tar xf Python-2.7.11.tgz
# cd Python-2.7.11
# ./configure -prefix=/usr/local/python2.7.11
# make && # make install
```
* 更改配置
创建链接来使系统默认python变为python2.7
```
# ln -fs /usr/local/python2.7.11/bin/python2.7
```
* 查看Python版本
```
# python -V
Python 2.7.11
```
* 修改`yum`配置（否则`yum`无法正常运行）
```
# vim /usr/bin/yum
# !/usr/bin/python
# 将第一行改为
# !/usr/bin/python2.6
```
###  `Python`简单入门
####  `Hello Word`
一般情况下程序猿的第一个小程序都是简单的输出`Hello Word!`，当然Python也不例外，下面就让我们来用`Python`输出一句`Hello Word!`吧！
* 创建一个以py结尾的文件
```
# touch hello.py
```

* 其内容为
```
#!/usr/vin/env python
print "Hello Word!"
```

* 用Python执行
```
# python hello.py
Hello Word!

```
输出的内容为`Hello Word!`，`OK`

####  Python代码执行流程

![python07](https://www.msbgn.cn/msbgn/python09.png)
`释意`：当Python执行代码的时候，会先把源码读取到内存当中，然后由Cpython进行编译，编译成字节码，最后由Cpython的虚拟机一行一行的解释其内容，再输出到屏幕上，然后释放内存，退出程序。

####  pyc文件

执行`Python`代码时，如果导入了其他的.py文件，那么，执行过程中会自动生成一个与其同名的.pyc文件，该文件就是Python解释器编译之后产生的字节码。
代码经过编译可以产生字节码；字节码通过反编译也可以得到代码
####  指定Python解释器

在Python文件的开头加入以下代码就制定了解释器。

* 第一种方式

```
#!/usr/bin/python
```

告诉`shell`这个脚本用`/usr/bin/python`执行

* 第二种方式

```	
#!/usr/bin/env python

```
在操作系统环境不同的情况下制定这个脚本用python来解释。
####  执行Python文件
执行Python文件的方式有两种

* 例如hello.py的文件内容为

```
#!/usr/bin/env python
print "Life is short, you need Pytho"

```
* 第一种执行方式

```
# python my.py
Life is short, you need Pytho
```
如果使用`python my.py`这种方式执行，那么`#!/usr/bin/python`会被忽略，等同于注释。
* 第二种执行方式

```
# chmod +x my.py 
# ./my.py 
Life is short, you need Pytho
```
如果使用`./my.py`来执行，那么`#!/usr/bin/python`则是指定解释器的路径，在执行之前`my.py`这个文件必须有执行权限。

`python my.py `实则就是在my.py文件顶行加入了`#!/usr/bin/python`

###  字符编码

python解释器在加载`.py`文件中的代码时，会对内容进行编码，python2.7内部默认`ascii`字符，而最新的`python3.5`默认使用UTF-8。
`ascii`字符(26个字母，8位表示所有，字符，英文 数字 )
万国码(unicode，最少两个字节，所有的字符，一个中文汉字3个字节表示，占用字节多，浪费空间)
UTF-8（unicode：英文特殊字符用八位表示，中文–24位….,解决unicode所占用的内存）

####  指定字符编码
python制定字符编码的方式有多种，而编码格式是要写在解释器的下面的，常用的如下面三种:
* 第一种

```
#!/usr/bin/env python
# _*_ coding:utf-8 _*_
```
* 第二种

```
#!/usr/bin/env python
# -*- coding:utf-8 -*-
```

* 第三种

```
#!/usr/bin/env python
# coding:utf-8
```
###  代码注释

####  单行注释

```
# 注释内容
```

####  多行注释
多行注释用三个单引号或者三个双引号躲起来

单行注释只需要在代码前面加上#号

```
"""
注释内容
"""
```

####  实例
`py`脚本原文件内容

```
#!/usr/bin/env python
# _*_ coding:utf-8 _*_
print "My name isAlexzhou"
print "I'm a Python developer"
print "My blog URL: http://msbgn.cn"
print "Life is short, you need Pytho"
```
源文件输出的内容
```
# python my.py 
My name is Alexzhou
I'm a Python developer
My blog URL: https://www.msbgn.cn
Life is short, you need Pytho
```
* 单行注释演示
* 代码改为

```
#!/usr/bin/env python
# _*_ coding:utf-8 _*_
print "My name is Alexzhou"
print "I'm a Python developer"
print "My blog URL: https://www.msbgn.cn"
#print "Life is short, you need Pytho"
```

执行结果

```
# python my.py 
My name is Alexzhou
I'm a Python developer
My blog URL: https://www.msbgn.cn
```
结果`Life is short, you need Pythoprint`出来
* 多行注释演示
* 代码改为

```
#!/usr/bin/env python
# _*_ coding:utf-8 _*_
print "My name is Alexzhou"
"""
print "I'm a Python developer"
print "Life is short, you need Pytho"
"""
```
* 执行结果

```
# python my.py 
My name is Alexzhou
```

print输出多行
既然用单个单引号或者多引号可以注释多行，那么能不能print多行呢？
* 代码

```
#!/usr/bin/env python
# _*_ coding:utf-8 _*_
print """
My name is Alexzhou
I'm a Python developer
My blog URL: https://www.msbgn.cn
Life is short, you need Python.
"""
```

执行结果
```

# python my.py 
My name is Alexzhou
I'm a Python developer
My blog URL: https://www.msbgn.cn
Life is short, you need Python.
```

变量

1.变量名只能包含数字、字幕、下划线
2.不能以数字开头
3.变量名不能使python内部的关键字

* Python内部已占用的关键字
['and', 'as', 'assert', 'break', 'class', 'continue', 'def', 'del', 'elif', 'else', 'except', 'exec', 'finally', 'for', 'from', 'g]
* 定义变量

```
>>> name = "Alexzhou" 
>>> print name
Alexzhou
```
###  基本的数据类型

* 字符串(str)
定义字符串类型是需要用单引号或者双引号包起来的

```
>>> name = "Alexzhou"
>>> print(type(name))
<type 'str'>
```
 或者

```
>>> name = 'Alexzhou'
>>> print(type(name))
<type 'str'>
```
* 数字(int)
证书类型定义的时候变量名后面可以直接跟数字，不要用双引号包起来。

```
>>> age = 20
>>> print(type(age))
<type 'int'>
```
* 布尔值()
布尔值就只有`True`(真)，`Flash`(假)

```
>>> if True:
...  print("0")
... else:
...  print("1")
...
0
```

解释：如果为真则输出`0`，否则输出`1`

###  流程控制

####  if语句
if语句是用来检查一个条件：如果条件为真(true)，我们运行一个语句块（你为if块），否则(else)，我们执行另一个语句块（称为else块）。else子语句是可选的。
####  单条件

例题：如果num变量大于1，那么就输出num大，否则就输出num小，num值为5。
* 代码

```
#!/usr/bin/env python
# -*- coding:utf-8 -*-
num = 5
if num > 1:     
 print("num大")
else:
 print("num小")
```

结果
```
# python num.py
num大
```
####  多条件
例题：如果num变量大于5，那么就输出num大于5，如果num变量小于5，那么就输出num小于5，否则就输出num等于5，num值为5。
```
#!/usr/bin/env python
# -*- coding:utf-8 -*-
num = 5
if num > 5:
 print("num大于5")
elif num < 5:
 print("num小于5")
else:
 print("num等于5")
```
结果
```
python num.py
num等于5
```
» python基础教程分享百度云盘链接: https://pan.baidu.com/s/1eSyXdx4 密码: 3ehk
» 文章出自链接：https://blog.ansheng.me/2016/04/python-full-stack-way-basics.html
