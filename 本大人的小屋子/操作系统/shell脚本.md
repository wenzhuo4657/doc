
# 概述
[linux 的基本操作（编写shell 脚本） - 让编程成为一种习惯 - 博客园](https://www.cnblogs.com/beautiful-code/p/15763495.html)
定义：一系列命令的集合,
存放位置：一般将其放置到`/usr/local/sbin/`

执行命令： 
[sh命令\_Linux sh 命令用法详解：shell命令解释器](https://man.niaoge.com/sh)

```
1.sh
#! /bin/bash
## author: hwz
## 第一个脚本
date +"%Y-%m-%d"
echo "over"
```

![[Pasted image 20241104091726.png]]

参考: [Linux脚本开头#!/bin/bash和#!/bin/sh是什么意思以及区别 - EasonJim - 博客园](https://www.cnblogs.com/EasonJim/p/6850319.html)
#! /bin/bash  :表示脚本解释程序的路径，
ps：如果没有指定解释器，则使用系统默认的解析器，可使用 `echo $SHELL`打印环境变量。
![[Pasted image 20241104093140.png]]



# 执行shell脚本
[source、sh、bash、./对比命令\_Linux source、sh、bash、./对比 命令用法详解：](https://man.niaoge.com/source)


|        | 环境       | 是否需要执行权限 |
| ------ | -------- | -------- |
| source | 当前shell  | no       |
| sh     | subshell | no       |
| ./     | subshell | yes      |

subshell: 对父shell的拷贝。





# shell变量
## 定义和使用
```
d1=这是什么
echo "d1=$d1"
```

## 命令替换：**$()** 和 **反引号**
[shell中的 单引号' '、双引号 " " 和 反引号\` \` - 哑吧 - 博客园](https://www.cnblogs.com/zhangxl1016/articles/14807800.html)
**命令替换是指shell能够将一个命令的 标准输出 插在一个命令行中任何位置。**

```
wenzhuo4657@4657001021:~$ echo `ls`
1.sh 1.txt
wenzhuo4657@4657001021:~$ echo $(date)
Mon Nov 4 09:55:08 CST 2024
```



## read：
从标准输入中获取变量值,将变量保存在当前shell会话空间
```
wenzhuo4657@4657001021:~$ read x
45
wenzhuo4657@4657001021:~$ echo $x
45
```


## set
显示当前会话空间的变量


## 预设变量
参考： [Shell环境变量与特殊变量详解](https://www.cnblogs.com/liang-io/p/9825363.html "发布于 2018-10-23 17:36")
sh   filename.sh   arg1 arg2 （传参对应 变量为 0，1，2.....）
```
echo "$0 $1 $2"
```


```

- $0    获取当前执行Shell脚本的文件名字，如果执行脚本时候加了路径，那就包含脚本路径跟脚本名字一起输出
- $n    获取当前执行Shell交本的第n个参数，n=1..9，n>9，后面参数变量就需要用大括号，如：${10}，以空格分隔
- $#    获取当前执行Shell脚本接了多少个参数（总计）
- $*    获取当前执行Shell脚本所有传参的参数
- $@    获取当前执行Shell脚本所有传参的参数（$*和$@详解见例子）
```
## 变量类型




# 流程控制









