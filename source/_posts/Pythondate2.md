---
title:  Python 流程控制
thumbnail: https://www.msbgn.cn/msbgn/day2.png
categories: python
tags: [语言,python]
---
-------------------


###  条件语句
Python条件语句是通过一条或多条语句的执行结果（True或者False）来决定执行的代码块。
可以通过下图来简单了解条件语句的执行过程:

![ ](https://www.msbgn.cn/msbgn/day2.png)


<!-- more -->



Python程序语言指定任何非0和非空（null）值为true，0 或者 null为false。
Python 编程中 if 语句用于控制程序的执行，基本形式为：

```
if 判断条件：
    执行语句……
else：
    执行语句……
```

其中"判断条件"成立时（非零），则执行后面的语句，而执行内容可以多行，以缩进来区分表示同一范围。
else 为可选语句，当需要在条件不成立时执行内容则可以执行相关语句，具体例子如下：

场景一、用户登陆验证


```
user=raw_input("请输入你的用户名:")


if user=="zhou":
        print 'hello',user

else:
        print '登录失败！'
```




场景二、猜年龄游戏


在程序里设定好你的年龄，然后启动程序让用户猜测，用户输入后，根据他的输入提示用户输入的是否正确，如果错误，提示是猜大了还是小了



```
temp = 28

busd=int(raw_input("请输入一个数字"))

if busd ==temp:
    print '恭喜你%d猜对了！' %busd
elif busd>temp:
    print '你输入的%d数字太大了！' %busd

else:
    print '你输入的%d数字太小了！' %busd
```

-------------------
 


-------------------
###    Python for loop
Python提供了for循环和while循环（在Python中没有do..while循环）:


最简单的循环10次


```

for i in  range(10):
    print i,


0 1 2 3 4 5 6 7 8 9

```



需求一：还是上面的程序，但是遇到小于5的循环次数就不走了，直接跳入下一次循环


```
for i in  range(10):
    if i>=5:

        print i,
    else:
       continue
```


需求二：还是上面的程序，但是遇到大于5的循环次数就不走了，直接退出


```
for i in  range(10):
    if i<=5:

        print i,
    else:
       continue
```


###  while loop


有一种循环叫死循环，一经触发，就运行个天荒地老、海枯石烂。


```
count = 0
while True:
    print("alexzhou...",count)
count +=1
```


上面的代码循环100次就退出吧


```
count = 0
while True:
    print("alexzhou...",count)
    count +=1
    if count==100:
        print '循环该结束了大兄弟'
        break
```


回到上面for 循环的例子，如何实现让用户不断的猜年龄，但只给最多3次机会，再猜不对就退出程序。


```
temp = 28#年龄等于28岁
count =0

while count<3:
    age = int(raw_input("请输入的年龄："))




if age==temp:
        print '恭喜你%d猜对了！' %age
        break
    elif age<temp:

        print '你输入的年龄%d数字太小了！' %age

    else:
        print '你输入的年龄%d数字太大了！' %age
    count +=1
else:
    print '才那么多次傻逼啊'

```
