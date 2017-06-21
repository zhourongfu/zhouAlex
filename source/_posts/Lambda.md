
---
title: Lambda表达式以及python的内置函数
thumbnail: https://www.msbgn.cn/msbgn/lambda.jpg
categories: python
tags: [语言,python]
---

-------------------


![ ](https://www.msbgn.cn/msbgn/lambda.jpg)


<!-- more -->


##### Lambda表达式

```
temp= lambda x,y:x+y

print temp(4,5)
temp= lambda x,y,z:x*y*z

print temp(4,5,2)

list(map(lambda x:x*2,range(10)))


```

`注意`: Lambda函数是匿名函数，一般结合`map`


-------------------
 


-------------------
#####    内置函数

```
a=[]
print dir() \\列出来的是key
print  vars() \\列出来的是key - value


['__builtins__', '__doc__', '__file__', '__name__', '__package__', 'a']
{'a': [], '__builtins__': <module '__builtin__' (built-in)>, '__file__': 'D:/distribute_project_py/shop.py', '__package__': None, '__name__': '__main__', '__doc__': '\nproduct = {\n\n    \'\xe8\x8b\xb9\xe6\x9e\x9c\xe6\x89\x8b\xe6\x9c\xba\':3000,\n    \'mackair\':8000,\n    \'\xe7\x89\x99\xe5\x88\xb7\':8,\n    \'\xe5\x85\x85\xe7\x94\xb5\xe7\xba\xbf\':80,\n    \'\xe9\x93\x85\xe7\xac\x94\':15\n\n}\nmoney =raw_input("\xe8\xaf\xb7\xe5\x85\x85\xe5\x80\xbc\xe9\x87\x91\xe9\xa2\x9d:")\nprint \'\xe5\x85\x85\xe5\x80\xbc\xe6\x88\x90\xe5\x8a\x9f\xe9\x87\x91\xe9\xa2\x9d\xe4\xb8\xba %s\' %money\nshop=[]\nprint "-"*12 +u"\xe5\x95\x86\xe5\x93\x81\xe5\x88\x97\xe8\xa1\xa8" +"-"*12\nfor i in product:\n\n    print \'\xe5\x90\x8d\xe7\xa7\xb0:%s---->\xe4\xbb\xb7\xe6\xa0\xbc\xef\xbc\x9a%d\'%(i,product[i])\nprint "-"*12 +u"\xe5\x95\x86\xe5\x93\x81\xe5\x88\x97\xe8\xa1\xa8" +"-"*12\nsheng_money=money\nwhile True:\n    buy=raw_input(u"\xe4\xbd\xa0\xe9\x9c\x80\xe8\xa6\x81\xe8\xb4\xad\xe7\x89\xa9\xe5\x90\x97\xef\xbc\x9fy/n")\n    if buy==\'y\':\n        salry=raw_input(u"\xe8\xbe\x93\xe5\x85\xa5\xe4\xbd\xa0\xe8\xa6\x81\xe8\xb4\xad\xe4\xb9\xb0\xe7\x9a\x84\xe5\x95\x86\xe5\x93\x81:")\n        if salry in product:\n            temp=int(sheng_money)-int(product[salry])\n            sheng_money=temp\n            if sheng_money>=0:\n                 print \'\xe6\x81\xad\xe5\x96\x9c\xe4\xbd\xa0\xe8\xb4\xad\xe4\xb9\xb0\xe6\x88\x90\xe5\x8a\x9f,\xe8\xb4\xad\xe4\xb9\xb0\xe7\x9a\x84\xe5\x95\x86\xe5\x93\x81\xef\xbc\x9a%s ,\xe5\x89\xa9\xe4\xbd\x99\xe9\x87\x91\xe9\xa2\x9d\xe4\xb8\xba\xef\xbc\x9a%d,\xe5\x95\x86\xe5\x93\x81\xe4\xbb\xb7\xe6\xa0\xbc\xe4\xb8\xba\xef\xbc\x9a%d\'  %(salry,sheng_money,product[salry])\n            else:\n                print \'\xe5\xbd\x93\xe5\x89\x8d\xe5\x89\xa9\xe4\xbd\x99\xe9\x87\x91\xe9\xa2\x9d\xef\xbc\x9a%d,\xe5\x95\x86\xe5\x93\x81\xe4\xbb\xb7\xe6\xa0\xbc\xef\xbc\x9a%d,\xe5\x89\xa9\xe4\xbd\x99\xe9\x87\x91\xe9\xa2\x9d\xe4\xb8\x8d\xe8\xb6\xb3\xe8\xaf\xb7\xe5\x85\x85\xe5\x80\xbc\xef\xbc\x81\' %(sheng_money,product[salry])\n                break\n        else:\n            print \'\xe6\xb2\xa1\xe6\x9c\x89\xe8\xaf\xa5\xe5\x95\x86\xe5\x93\x81%s\' %salry\n            salry=raw_input(u"\xe8\xaf\xb7\xe9\x87\x8d\xe6\x96\xb0\xe8\xbe\x93\xe5\x85\xa5:")\n    else:\n        print "\xe6\xac\xa2\xe8\xbf\x8e\xe4\xb8\x8b\xe6\xac\xa1\xe5\x85\x89\xe4\xb8\xb4\xef\xbc\x81"\n        break\n        '}
help(vars())
```

```
print  type(a)

<type 'list'>
```
查看一个变量的类型

导入模块

```
from  file import demo
reload(demo)//再次导入模块
```

id()，内存地址

```
t1 = 123
t2=999
print id(t1)
print id(t2)

6793536  //t1的内存地址
44305828  //t2的内存地址
```

Abs()，取绝对值

```
print  abs(-9)

9
```

Bool()，判断真假

```
print  bool(1)  //输出True
print  bool(0)   //输出False
print  bool(15)   //输出True
print  bool(-1)   //输出True
```

divmod(),数字处理函数,实用场景例如：分页

```
print  divmod(9,4)
print  divmod(9,3)

(2, 1)
(3, 0)
```

`cmp( x, y)`，比较大小，如果x>y，则返回1，负责返回-1

```
print cmp(4,8)
-1
print cmp(4,2)
1
```

`Max()`,返回参数的最大值

```
print max([11,33,55,22,9,77,38])

77
```

min(),返回参数的最小值

```
print min([11,33,55,22,9,77,38])

9
```

`sum(iterable[, start])`，用于计算可迭代对iterable的和

```
print sum([11,33,55,22,9,77,38])

245
```

`Pow(x,y)`,用于求参数x的多少次方y

```
print pow(2,10)

1024
```


`Len()`,返回字符串的长度，如果是中文，则返回的是字节的长度

```
print len('alexzhou')

8


print len('你好')
6

```


`all(iterable)`，判断可迭代的值里面如果有出现的值为0，则返回的值为false,否则返回True

```
print all([1,2,3,4,7])
print all([1,2,3,4,0,1,22])


True
False
```


`Any(iterable)`, 判断可迭代的值里面如果值都为0，则返回的值为`false`,否则返回`True`

```
print any([1,2,3,4,7])
print any([1,2,3,4,0,1,22])
print any([0,0,0,0,0])

True
True
False
```

`Chr()`,根据参数值`ascii`，返回对应的字符串


```
print  chr(65)
print  chr(44)
print  chr(66)
print  chr(67)
print  chr(68)


A
,
B
C
D
```

Ord(),根据参数值，返回对应的ascii的转换（实用于验证码数字字母转换）

```
print  ord('A')
print  ord('B')
print  ord('C')
```

`hex()`、bin()、`oct()`,八进制，16进制，2进制之间的转换

```
print hex(20)
print bin(20)
print oct(20)

0x14
0b10100
024
```



`Enumerate()`,添加索引序号，返回值可以根据自定义的序号返回参数

```
shop=['猕猴桃','香蕉','橙子']

for i in shop:
    print i


for team in enumerate(shop,10):
    print team[0],team[1]


猕猴桃
香蕉
橙子
10 猕猴桃
11 香蕉
12 橙子
```

`format`函数格式化字符串，{0}，{1}相当于占位符%s

```
s='我 叫 {0},{1}'

print s.format('Alexzhou','xx')
```


apply(),用于执行函数


```
def say(a,b):
  print a,b

apply(say,("hello","Alex"))
```


`map()`是 `Python` 内置的高阶函数，它接收一个函数 f 和一个 list，并通过把函数 f 依次作用在 list 的每个元素上，得到一个新的 list 并返回,常常于`lambda`一起使用

```
print  map(lambda x:(x*2)-1,range(1,50))
print  map(lambda x:x*2,range(51))

100以内的奇偶数
[1, 3, 5, 7, 9, 11, 13, 15, 17, 19, 21, 23, 25, 27, 29, 31, 33, 35, 37, 39, 41, 43, 45, 47, 49, 51, 53, 55, 57, 59, 61, 63, 65, 67, 69, 71, 73, 75, 77, 79, 81, 83, 85, 87, 89, 91, 93, 95, 97]
[0, 2, 4, 6, 8, 10, 12, 14, 16, 18, 20, 22, 24, 26, 28, 30, 32, 34, 36, 38, 40, 42, 44, 46, 48, 50, 52, 54, 56, 58, 60, 62, 64, 66, 68, 70, 72, 74, 76, 78, 80, 82, 84, 86, 88, 90, 92, 94, 96, 98, 100]
```


Filter(),用于过滤条件，过滤

```
shop=[11,22,33]
def foo(arg):
    if arg<22:
        return True
    else:
        return False
temp=filter(foo,shop)
print  temp

[11]
```


Reduce(),累加，累乘…，必须要接收两个参数


```
shop=[11,22,33]


print  reduce(lambda x,y:x+y,shop)
```


zip(),拼接列表的列组成一个新的序列

```
x=[1,2,4]
y=[4,5]
z=[4,5,6]
t =[8,9,10]
print zip(x,y,z,t)
```


`eval()`,字符串当做表达式执行

```
a= '9+9'
print  eval(a)
```

##### 反射
通过字符串的形式去导入模块，并以字符串的形式执行函数

```
temp = 'mysqlheerp'
mode = __import__(temp)
mode.cout()
```
