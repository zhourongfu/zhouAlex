title: python的字符串续及列表
thumbnail: https://www.msbgn.cn/msbgn/python01.jpg
categories: python
tags: [语言,python]

----------



* index(self, sub, start=None, end=None):
`str` –> 指定检索的字符串
`beg` –> 开始索引，默认为0。
`end` –> 结束索引，默认为字符串的长度。
检测字符串中是否包含子字符串 str ，如果指定 beg（开始） 和 end（结束） 范围，则检查是否包含在指定范围内，该方法与 python find()方法一样，只不过如果str不在 string中会报一个异常。

<!-- more -->
``` 
>>> string="hello word"
# 返回字符串所在的位置
>>> string.index("o")
4
# 如果查找一个不存在的字符串那么就会报错
>>> string.index("a")
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ValueError: substring not found
```

* isalnum(self):

法检测字符串是否由字母和数字组成，如果string至少有一个字符并且所有字符都是字母或数字则返回True,否则返回False

```
>>> string="hes2323"
# 如果存在数字或字母就返回`True`，否则返回`False`
>>> string.isalnum()
True
# 中间有空格返回的就是False了
>>> string="hello word"
>>> string.isalnum()
False
```

* isalpha(self):

检测字符串是否只由字母组成。

```
# 如果全部都是字母就返回`True`
>>> string="helloword"
>>> string.isalpha()
True
# 否则就返回False
>>> string="hes2323"
>>> string.isalpha()
False
```

* isdigit(self):
检测字符串是否只由数字组成

```
# 如果变量里面都是数字就返回`True`，否则就返回`False`
>>> string="hes2323"
>>> string.isdigit()
False
>>> string="2323"
>>> string.isdigit()
True
```

* islower(self):

```
# 如果变量内容全部都是小写字母就返回`True`，否则就返回`False`
>>> string="hesasdasd"
>>> string.islower()
True
>>> string="HelloWord"
>>> string.islower()
False
```

* istitle(self):
检测字符串中所有的单词拼写首字母是否为大写，且其他字母为小写。

```
# 如果变量的内容首字母是大写并且其他字母为小写，那么就返回`True`，否则会返回`False`
>>> string="Hello Word"
>>> string.istitle()
True
>>> string="Hello word"
>>> string.istitle()
False
```

* isupper(self):

检测字符串中所有的字母是否都为大写。


```
# 如果变量值中所有的字母都是大写就返回`True`，否则就返回`False`
>>> string="hello word"
>>> string.isupper()
False
>>> string="HELLO WORD"
>>> string.isupper()
True
```

* join(self, iterable):

将序列中的元素以指定的字符连接生成一个新的字符串。

```
>>> string=("a","b","c")
>>> '-'.join(string)
'a-b-c'
```

* lower(self):

转换字符串中所有大写字符为小写。

```
# 把变量里的大写全部转换成小写
>>> string="Hello WORD"
>>> string.lower()
'hello word'
```


* lstrip(self, chars=None):

```
>>> string="hello word"
>>> string.lstrip("hello ")
'word'
chars –> 指定截取的字符
法用于截掉字符串左边的空格或指定字符
```


* replace(self, old, new, count=None):


`old` –> 将被替换的子字符串
`new` –> 新字符串，用于替换old子字符串
`count `–> 可选字符串, 替换不超过count次
把字符串中的 old(旧字符串)替换成new(新字符串)，如果指定第三个参数max，则替换不超过max次


```
string="www.ss.me"
# 把就字符串`www.`换成新字符串`https://`
>>> string.replace("www.","https://")
'https://www.xxx.me'
# 就字符串`w`换成新字符串`a`只替换`2`次
>>> string.replace("w","a",2)
'aaw.ansssss.me'
```


* rfind(self, sub, start=None, end=None):
查找的字符串
beg –> 开始查找的位置，默认为0
end –> 结束查找位置，默认为字符串的长度
返回字符串最后一次出现的位置，如果没有匹配项则返回-1。


```
string="hello word"
# rfind其实就是反向查找
>>> string.rfind("o")
7
# 指定查找的范围
>>> string.rfind("o",0,6)
4
```

* rsplit(self, sep=None, maxsplit=None):

`str--> 分隔符，默认为空格 *num*` –> 分割次数
从右到左通过指定分隔符对字符串进行切片,如果参数num有指定值，则仅分隔num个子字符串

```
string="www.msbgn.cn"
>>> string.rsplit(".",1)
['www.msbgn', 'cn']
>>> string.rsplit(".",2)
['www', 'msbgn', 'cn']
```


* split(self, sep=None, maxsplit=None):

str--> 分隔符，默认为空格 *num*` –> 分割次数
从左到右通过指定分隔符对字符串进行切片,如果参数num有指定值，则仅分隔num个子字符串

```
string="www.msbgn.cn"
# 指定切一次，以`.`来分割
>>> string.split(".",1)
['www', 'msbgn.cn']
# 指定切二次，以`.`来分割
>>> string.split(".",2)
['www', 'msbgn', 'cn']
```


* splitlines(self, keepends=False):

num –> 分割行的次数
按照行分隔，返回一个包含各行作为元素的列表，如果num指定则仅切片num个行.

```
# 定义一个有换行的变量，`\n`可以划行
>>> string="www\msbgn\cn"
# 输出内容
>>> print(string)
www
msbgn
cn
# 把有行的转换成一个列表
>>> string.splitlines(1)
['www\n', 'msbgn\n', 'cn']
```


* strip(self, chars=None):

chars –> 移除字符串头尾指定的字符
移除字符串头尾指定的字符（默认为空格）

```
>>> string=" www.msbgn.cn"
>>> string
' www.msbgn.cn'
# 删除空格
>>> string.strip()
'www.msbgn.cn'
>>> string="_www.msbgn.cn_"
# 指定要把左右两边的"_"删除掉
>>> string.strip("_")
'www.msbgn.cn'
```

* swapcase(self):

用于对字符串的大小写字母进行转换，大写变小写，小写变大写

```
>>> string="hello WORD"
>>> string.swapcase()
'HELLO word'
```


* translate(self, table, deletechars=None):

able –> 翻译表，翻译表是通过maketrans方法转换而来
deletechars –> 字符串中要过滤的字符列表

根据参数table给出的表(包含 256 个字符)转换字符串的字符, 要过滤掉的字符放到 del 参数中。

* upper(self):

将字符串中的小写字母转为大写字母


```
string="hello word"
>>> string.upper()
'HELLO WORD'
```


##  列表(list)


定义列表的两种方法

* 第一种

```
name_list = ['Python', 'PHP', 'JAVA']
```
第二种

```	
name_list ＝ list(['Python', 'PHP', 'JAVA'])
```

列表所具备的方法


* append(self, p_object):


在列表末尾添加新的对象


```
name_list = ['zhou','jack','old gog']
name_list.append('sds')
>>> name_list
['zhou', 'jack', 'old gog', 'sds']
```


* count(self, value):

bj –> 列表中统计的对象
统计某个元素在列表中出现的次数

```
name_list.count('jack')
2
```


* insert(self, index, p_object):


index –> 对象obj需要插入的索引位置
obj –> 要出入列表中的对象
将指定对象插入列表

```
name_list.insert(2,'110')
>>> name_list
['zhou', 'jack', '110', 'old gog', 'sds']
```


remove(self, value):


`value` –> 列表中要移除的对象
移除列表中某个值的第一个匹配项

```
name_list.remove('old gog')
>>> name_list
['zhou', 'jack', '110', 'sds']
```


* extend(self, iterable):


seq –> 元素列表
用于在列表末尾一次性追加另一个序列中的多个值

```
name_list = ['Python', 'PHP', 'Python']
>>> name_OS = ['Windows', 'Linux', 'Unix']
>>> name_list
['Python', 'PHP', 'Python']
>>> name_OS
['Windows', 'Linux', 'Unix']
# 把列表`name_OS`中的内容添加到`name_list`的尾部
>>> name_list.extend(name_OS)
# 输出的结果
>>> name_list
['Python', 'PHP', 'Python', 'Windows', 'Linux', 'Unix']
```


pop(self, index=None):

index –> 可选参数，要移除列表元素的位置
移除列表中的一个元素，并且返回该元素的值

```
>>> name_list = ['Python', 'PHP', 'JAVA']
# 删除位置1上面的内容，并且返回删除的字符串
>>> name_list.pop(1)
'PHP'
>>> name_list
['Python', 'JAVA']
```


sort(self, cmp=None, key=None, reverse=False):

对原列表进行排序，如果指定参数，则使用比较函数指定的比较函数

```
>>> name_list = ['Python', 'PHP', 'JAVA']
>>> name_list
['Python', 'PHP', 'JAVA']
>>> name_list.sort()
>>> name_list
['JAVA', 'PHP', 'Python']
```

* 列表求奇偶以及练习

隔断截取，求奇偶


```
name_list[1::2]
['jack', 2, 4]
>>> name_list[1::3]
['jack', 3]
>>> a = range(100)
>>> a
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, 52, 53, 54, 55, 56, 57, 58, 59, 60, 61, 62, 63, 64, 65, 66, 67, 68, 69, 70, 71, 72, 73, 74, 75, 76, 77, 78, 79, 80, 81, 82, 83, 84, 85, 86, 87, 88, 89, 90, 91, 92, 93, 94, 95, 96, 97, 98, 99]
>>> a[::2]
[0, 2, 4, 6, 8, 10, 12, 14, 16, 18, 20, 22, 24, 26, 28, 30, 32, 34, 36, 38, 40, 42, 44, 46, 48, 50, 52, 54, 56, 58, 60, 62, 64, 66, 68, 70, 72, 74, 76, 78, 80, 82, 84, 86, 88, 90, 92, 94, 96, 98]
>>> a[1::2]
[1, 3, 5, 7, 9, 11, 13, 15, 17, 19, 21, 23, 25, 27, 29, 31, 33, 35, 37, 39, 41, 43, 45, 47, 49, 51, 53, 55, 57, 59, 61, 63, 65, 67, 69, 71, 73, 75, 77, 79, 81, 83, 85, 87, 89, 91, 93, 95, 97, 99]
```

* 练习 求列表中2出现的次数的，分别对应的位置

```
name_list = [2,'zhou','jack',3,'old gog',1,2,3,4,5,6,2,8,99,3,2,666,2]
a1 = 0
for i in range(name_list.count(2)):
        nexts =name_list[a1:]
        a2 =  nexts.index(2)+1
        print 'find:',a1 + nexts.index(2)
        a1+=a2
```

结束
