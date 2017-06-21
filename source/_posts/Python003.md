---
title: Python的数据类型和运算
thumbnail: https://www.msbgn.cn/msbgn/python01.jpg
categories: Python
tags: [语言,Python]
---
-------------------



###  语法要求


-------------------
 
缩进统一
变量
* 标识符第一个字符必须是字母表中的字母（大写或者小写）或者一个下划线('_')
* 标识符名称的组成部分可以由字母(大写或小写)、下划线（'_'）或数字0-9组成
* 标识符名称是对大小写敏感的，列如，myname和Myname不是一个标识符。注意前者中的小写m和后者中的大写M.
* 有效标识符名称的列子有 i、_my_name、name_32和alb2_c3。
* 无效的标识符名称的列子有 2thongs、this is spaced out和my-name.



-------------------

<!-- more -->

###  定义变量

```
task_detail=1（变量写法一）
TaskDetail=1 （变量写法二）
taskDetail=1 （驼峰写法）
```

###  数据类型(特征划分)

####  数字类型

*  布尔型（bool）
判断真假
```
false
true

if False:print 'ddd'
...

if True: print 'dddd'
...
ddd
```

###  整型
在Python2里，一个int型包含32位，可以存储从-2147483648到214483647的整数
一个long型会占用更多的空间，64为可以存储-922372036854775808到922372036854775808的整数
python3里long型已经不存在了，而int型可以存储到任意大小的整型，甚至超过64为。
Python内部对整数的处理分为普通整数和长整数，普通整数长度为机器位长，通常都是32位，超过这个范围的整数就自动当长整数处理，而长整数的范围几乎完全没限制，如下：


```
a = 2**64
a
18446744073709551616L
>>> type(a)
<type 'long'>
a = 2**12
>>> a = 2**12
>>> type(a)
<type 'int'>
```

###  非整型
列如：浮点型,序列类型,字符串，列表，元组,字典

```
>>> type(3.14)
<type 'float'>

name='alex'
type(name)
<type 'str'>
>>> if type(name) is str: print 'ddd'
... 
ddd

name_list=['sdss','alexzhou','zhoualx']
>>> type(name_list)
<type 'list'>

name={'alexzhou':[28,'IT']}
name['alexzhou']
[28,'IT']
<type 'dict'>
```


###  数学运算

```
1+1*3/2
2**32


A=14
B=12
A>B,A<=B,A!=B
```


###  字符串(str)

字符串类型是python的序列类型，他的本质就是字符序列，而且python的字符串类型是不可以改变的，你无法将原字符串进行修改，但是可以将字符串的一部分复制到新的字符串中，来达到相同的修改效果。


###  使用引号创建字符串
创建字符串类型可以使用单引号或者双引号又或者三引号来创建，实例如下

* 单引号

```
>>> string = 'alxe'
# type是查看一个变量的数据类型
>>> type(string)
<class 'str'>

```

* 双引号

```
>>> string = "alxe"
# type是查看一个变量的数据类型
>>> type(string) 
<class 'str'>

```


三引号

```
>>> string = """alxe"""
>>> type(string)
<class 'str'>
```


###  字符串所具备的方法

 * capitalize(self):

把值得首字母变大写


```
>>> name="alxe"
>>> name.capitalize()
'alxe'
```

* center(self, width, fillchar=None):
内容居中，width：字符串的总宽度；fillchar：填充字符，默认填充字符为空格。


```
# 定义一个字符串变量，名为"string"，内容为"hello word"
>>> string="hello word"
# 输出这个字符串的长度，用len(value_name)
>>> len(string)
10
# 字符串的总宽度为10，填充的字符为"*"
>>> string.center(10,"*")
'hello word'
# 如果设置字符串的总产都为11，那么减去字符串长度10还剩下一个位置，这个位置就会被*所占用
>>> string.center(11,"*")
'*hello word'
# 是从左到右开始填充
>>> string.center(12,"*")
'*hello word*'

```


* count(self, sub, start=None, end=None):


`sub` –> 搜索的子字符串
`start` –> 字符串开始搜索的位置。默认为第一个字符,第一个字符索引值为0。
`end` –> 字符串中结束搜索的位置。字符中第一个字符的索引为 0。默认为字符串的最后一个位置。

用于统计字符串里某个字符出现的次数,可选参数为在字符串搜索的开始与结束位置。

```
>>> string="hello word"
# 默认搜索出来的"l"是出现过两次的
>>> string.count("l")
2
# 如果指定从第三个位置开始搜索，搜索到第六个位置，"l"则出现过一次
>>> string.count("l",3,6)
1
```


* decode(self, encoding=None, errors=None):
解码

```
# 定义一个变量内容为中文
temp = "中文"
# 把变量的字符集转化为UTF-8
temp_unicode = temp.decode("utf-8")
```


* encode(self, encoding=None, errors=None):
编码，针对unicode

```
# 定义一个变量内容为中文,字符集为UTF-8
temp = u"中文"
# 编码，需要指定要转换成什么编码
temp_gbk = temp_unicode.encode("gbk")
```

* endswith(self, suffix, start=None, end=None):

`suffix` –> 后缀，可能是一个字符串，或者也可能是寻找后缀的tuple。
`start` –> 开始，切片从这里开始。
`end` –> 结束，片到此为止。


于判断字符串是否以指定后缀结尾，如果以指定后缀结尾返回True，否则返回False。


```
>>> string="hello word"
# 判断字符串中是否已"d"结尾，如果是则返回"True"
>>> string.endswith("d")
True
# 判断字符串中是否已"t"结尾，不是则返回"False"
>>> string.endswith("t")
False
# 制定搜索的位置，实则就是从字符串位置1到7来进行判断，如果第七个位置是"d"，则返回True，否则返回False
>>> string.endswith("d",1,7)
False
```

* expandtabs(self, tabsize=None):

tabsize –> 指定转换字符串中的 tab 符号(‘\t’)转为空格的字符数。
把字符串中的tab符号(‘\t’)转为空格，tab符号(‘\t’)默认的空格数是8。


```
>>> string="hello       word"
# 输出变量"string"内容的时候会发现中间有一个"\t"，这个其实就是一个`tab`键
>>> string
'hello\tword'
# 把`tab`键换成一个空格
>>> string.expandtabs(1)
'hello word'
# 把`tab`键换成十个空格
>>> string.expandtabs(10)
'hello     word'
```

* find(self, sub, start=None, end=None):

`str` –> 指定检索的字符串
`beg` –> 开始索引，默认为0。
`end` –> 结束索引，默认为字符串的长度。


检测字符串中是否包含子字符串str，如果指定beg(开始)和end(结束)范围，则检查是否包含在指定范围内，如果包含子字符串返回开始的索引值，否则返回-1。


```
>>> string="hello word"
# 返回`o`在当前字符串中的位置，如果找到第一个`o`之后就不会再继续往下面寻找了
>>> string.find("o")
4
# 从第五个位置开始搜索，返回`o`所在的位置
>>> string.find("o",5)
7
```


* format(args, *kwargs):

未完，待续…..
