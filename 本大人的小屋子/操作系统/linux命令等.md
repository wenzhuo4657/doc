开源协议：
- 木兰协议：
- GPL：
- LGPL：
- BSD



# shell：命令行解释器
分为命令行/cli(常见有 bash)和用户界面。

当用户在 Bash 中输入命令时，readline 库负责处理键盘输入，使得用户可以利用箭头键来导航命令历史，使用 Tab 键来进行命令或路径的自动补全等。此外，readline 还支持一些常用的编辑按键，如删除、插入等操作。


[Linux命令大全(手册) – 真正好用的Linux命令在线查询网站 (linuxcool.com)](https://www.linuxcool.com/)

https://man.niaoge.com/


ctrl+D:离开命令行界面
ctrl+C:中断当前程序
ctrl+Enter:换行，但不执行命令


查询帮助
- helo
- menu ：也可以写做 man 
- info



关机命令
shutdown


不安全的关机
- 重启：reboot
- 挂起：halt
- 强制关机和断电：poweroff


切换用户：
su 

# linux目录配置与FSH标准

[Linux目录配置与FHS标准 - 蚂蚁小哥 - 博客园 (cnblogs.com)](https://www.cnblogs.com/antLaddie/p/17613126.html)



# linux权限规则
[linuxwen文件权限](https://rgb-24bit.github.io/blog/2018/linux-file-permission.html)


权限掩码
umask:权限掩码是由3个八进制的数字所组成，将现有的存取权限减掉权限掩码后，即可获得建立文件时预设的权限啦。  
原文链接：[umask命令](https://www.linuxcool.com/umask)


修改权限：
- `chown` ：修改目录或文件的所有者
- `chgrp`
- chmod :设置权限时可以使用数字法，亦可使用字母表达式，  



特殊权限：
- SUID：
- SGID：
- SBIT:
https://cloud.tencent.com/developer/article/1722142

设置位置为权限掩码的第一位，




创建文件

mkdir:创建目录
rmdir:删除空目录
rm:删除非空目录
touch:创建空文件与修改时间戳  
cp:复制文件或目录
mv:移动文件和重命名（对同一个目录内的文件移动可视为重命名）

# systemd：
参考：
https://systemd.io/
https://0pointer.de/blog/projects/socket-activation.html

systemd是以 PID 1 运行的系统和服务管理器，许多linux都使用它。


基本用法： https://yanweixin.github.io/post/systemd-usage/





# ip相关

hostname：用于显示和设置系统的主机名称。环境变量HOSTNAME也保存了当前的主机名。


# 环境变量
[简单易懂解说「到底什么是环境变量？」| Code | Chihokyo BLOG | 旺财的博客](https://chihokyo.com/post/6/)
export导入临时环境变量

永久环境变量通过配置文件设置
Ubuntu:
/etc/profile>~/.bash_profile>~/.bashrc

环境变量的意义是将命令简写。对于linux系统来说，无论是shell脚本的执行，或者是可执行文件的执行，都无非是打开一个子shell又或者是直接在当前shell环境中执行，其本质上都可以通过路径**直接bash中执行**（这得益于bash，但bash本身就是操作系统的一部分，所以不去深究）
```
wenzhuo4657@4657001021:~$ which ls
/bin/ls
wenzhuo4657@4657001021:~$ /bin/ls
wenzhuo4657@4657001021:~$ ls
```
因此环境变量的机制实际上是将路径参数简写，便于bash解释执行。通常系统会顺序读取环境变量，将第一个找到的可执行文件执行，因此在加载时越早被读取优先级越高。

```

# PATH是一个环境变量，打印全部环境变量，可使用`printenv`或者`env`
wenzhuo4657@4657001021:~$ echo $PATH | awk -F: '{for(i=1;i<=NF;i++) print $i}'
/usr/local/sbin
/usr/local/bin
/usr/sbin
/usr/bin
/sbin
/bin
....
```



# 文件系统

[【Linux】理解Linux中的软链接与硬链接-CSDN博客](https://blog.csdn.net/2403_86785171/article/details/141906357)

centos07的文件系统是xfs


lnode:indoe号（文件数据的索引号：根据这个找到文件的数据区域data）和文件属性（例如ls -l所查询的信息）


对应关系：一个文件名对应一个lnode号,但是可以多个文件名对应同一个lnode号


硬链接：多个文件名可以指向同一个 inode号，这就是硬链接。每个硬链接都会增加 inode 的链接计数，**且共享相同的文件数据空间。**
- 共享存储块，指向相同的数据空间
- 不跨文件系统
- 不可以对目录进行硬链接
- 删除源文件后，硬链接不受影响
![[img/Pasted image 20240909113638.png]]

软链接/符号链接：软链接指向的是文件路径而不是实际的数据块，相当于引用，
- 路经形式存在，
- 删除源文件后，链接失效
- 可以跨文件系统
- 可以指向目录
![[img/Pasted image 20240909113954.png]]



**文件链接命令 
ln [-s] 源文件 链接文件++**

xfs文件系统分区

![[img/Pasted image 20240909105440.png]]


![[img/Pasted image 20240909105505.png]]





## 输入与输出
### stdin与 stdout以及stderr

stdin(标准输入: /dev/stdin) :  一般指键盘输入,   
stdout(标准输出:/dev/stdout):一般指标准输出
stderr(标准错误:/dev/stderr):一般指标准错误


### 重定向

输出重定向符号 `>`: 将**某些输出**重定向到某个文件中

![[img/Pasted image 20240913084348.png]]


输入重定向 `<`:
 < file
 <<!....!：!表示结束符号，该符号可以自定义，例如eof等 `<< eof  ......eof`,且注意eof作为结束符号不作为输入。

### xargs

参考： https://www.cnblogs.com/wangwust/p/10032275.html
xargs：将标准输入数据转换成命令行参数。
**xargs的默认命令是echo，空格是默认定界符**。这意味着通过管道传递给xargs的输入将会包含换行和空白，不过通过xargs的处理，换行和空白将被空格取代。

**xargs是构建单行命令的重要组件之一。**

参考： https://man.niaoge.com/xargs

1，定义一个测试文件，内有多行文本数据：

```
```cat test.txt

a b c d e f g
h i j k l m n
o p q
r s t
u v w x y z

```
多行输入单行输出：
```

cat test.txt | xargs

a b c d e f g h i j k l m n o p q r s t u v w x y z

```

2.使用管道符，将标准输入转换为后者命令的参数
`ls -R |xargs grep "root"`：递归检查文件内容



### awk
[awk命令\_Linux awk 命令用法详解：文本和数据进行处理的编程语言](https://man.niaoge.com/awk)
对文本和数据进行处理。
```
# 使用分割符 `:` 处理文本
echo $PATH | awk -F: '{for(i=1;i<=NF;i++) print $i}'  
```


## 文件查看

查看文件内容
- cat: 终端设备上显示文件内容,常搭配输出重定向符号`>` 使用

cat > filename :将标准输入重定向到文件filename中，使用 `ctrl+D`中断程序
cat > filename << eof: 依旧是标准输入，只是将eof作为结束符号，将输出重定向到filename文件中，（并非是从左到右执行，这里类似于汇编，在cpu执行的是整条指令翻译为的机器代码，而并非顺序执行一条命令）

![[img/Pasted image 20240913085737.png]]

- nl:显示文件内容及行号，使用nl命令会有类似于“cat -n 文件名”的效果，除此之外还可以对于显示的行号格式进行深度定制 。
- head：显示文件开头的内容，默认为前10行  
		-n参数： 正数表示显示行数，负数表示除了倒数行数不显示，剩余都显示
	head -n 5 file:显示前5行
	head -n -5 file:除了倒数5行不显示，剩余都显示。
- tail:查看文件尾部内容 ，默认为前10行  
- more：
- less：


管道 `|`
格式：cmd1 |cmd2|...
- 从左到有执行，将前者的输出作为后者的输入。


## 文件搜索
### grep

参考： https://www.cnblogs.com/wangwust/p/10032275.html

接收数据：标准输入、单个文件



如果把grep命令当作标准搜索命令，那么egrep则是扩展搜索命令，等价于grep -E命令，支持扩展的正则表达式。而fgrep则是快速搜索命令，等价于grep -F命令，不支持正则表达式，直接按照字符串内容进行匹配。

拓展： 


man grep查看手册得知
```
DESCRIPTION
       grep  searches  for  PATTERN  in  each  FILE.   A FILE of “-” stands for standard input.  If no FILE is given,
       recursive searches examine the working directory, and nonrecursive searches read standard input.  By  default,
       grep prints the matching lines.

       In  addition,  the  variant  programs  egrep,  fgrep  and rgrep are the same as grep -E, grep -F, and grep -r,
       respectively.  These variants are deprecated, but are provided for backward compatibility.

说明
       grep 搜索每个 FILE 中的 PATTERN。FILE 中的"-"代表标准输入。如果没有给出 FILE,递归搜索将检查工作目录，而非递归搜索将读取标准输入。默认情况下grep 打印匹配的行。
       此外，变体程序 egrep、fgrep 和 rgrep 与 grep -E、grep -F 和 grep -r、分别与 grep -E、grep -F 和 grep -r 相同。  这些变体已被弃用，但为了向后兼容而提供。

```

`FILE 中的"-"代表标准输入`,
![[Pasted image 20241103153559.png]]

`如果没有给出 FILE,递归搜索将检查工作目录，而非递归搜索将读取标准输入`
1:表示不输入文件，也不输入`-`，默认会从标准输入中读取。

![[Pasted image 20241103153540.png]]
2：使用-r参数，默认输入为标准输入
![[Pasted image 20241103154109.png]]

3：即便有-r递归参数，但如果指定了`-`为输入，则不会递归搜索。
![[Pasted image 20241103153854.png]]



因此grep接收参数优先级别 :只要指定了 file参数就不会失效。

	默认（`-`：标准输入）== 手动输入参数`-`  ==  grep -r "root" -  > grep -r "root" /path  


特殊：没有使用file参数，但使用了-r递归搜索时，将递归检查当前工作目录
grep -r "root" ==  ls -R |grep -r "root" 
（注意，并不会读取文件内容，而是将ls -R的标准输出作为后者的标准输入）

此外，如果想要将递归搜索文本文件的内容，而并非文件名，可以使用`xargs`命令进行转换，
`find . -name '*.py' |xargs grep test`

## 文件查找

### shell通配符
在 Linux 中，许多命令行工具都支持使用通配符来匹配文件名或其他模式。通常，这些通配符是在 shell 层面处理的，而不是由具体的命令来处理。
`*`:表示任意
`?`：表示任意一个
`[arg1,arg2..]`:表示范围




### 命令

####  which
查找二进制文件（可执行文件.exe）,也可以说是命令文件
 例如： `which ls` 
####  wherels
  显示命令及相关文件的路径位置信息  .
![[img/Pasted image 20240914104518.png]]

#### locate
基于数据文件（/var/lib/locatedb）进行的定点查找  文件或者目录


#### find 
根据给定的路径（将该路径作为根目录）和条件查找相关文件或目录 
命令格式：find [path] ....... [expression] 
其中[expression] 表示动作表达式
![[img/Pasted image 20240914115108.png]]

注意 command表示命令，实际使用应替换为ls等，
commadn是一种执行命令的方式，可以执行任何在路径环境变量 ($PATH) 中定义的命令。这包括Linux命令、shell内置命令以及其他可执行程序。
linux命令通常都是直接执行一个可执行文件。

（1）时间
（2）用户
（3）权限
（4）文件类型
![[img/Pasted image 20240914111952.png]]
![[img/Pasted image 20240914105340.png]]
默认路径是当前目录，且查找时将当前目录作为根目录。该参数的作用更像是做一个前缀的拼接，默认为`.`

![[img/Pasted image 20240914105827.png]]

mtime、等时间相关命令参数的正负数概念

![[img/Pasted image 20240914111545.png]]


find中根据时间相关查找的可选参数
![[img/Pasted image 20240914111808.png]]
![[img/Pasted image 20240914111821.png]]



# 编辑器

## vim


# 文件压缩和打包命令
压缩：将文件压缩，其结果只会是相同数量的文件，但存储大小变小了，
打包：将多个文件打包为一个文件,且比原大小要大。

主要压缩、打包工具：
- gzip：压缩
- -bzip2：压缩
- tar：打包
- xz：压缩


![[Pasted image 20240923105518.png]]


## gzip

格式:gzip[-cdtv#] files
Gzip称为GNUzip，采用Deflate数据压缩算法
默认压缩的文件扩展名为.gz
压缩后生成压缩文件，原文件会被自动删除。
压缩文件内容查看命令:zcat/zmore/zless
**且注意该命令只能读取一个文件参数，可以是通配符（由shell解释给gzip），也可以是具体的某个文件**

![[Pasted image 20240928092126.png]]
如上图所示，在gzip的执行过程中并没有将其作为一个压缩执行，而是分为多个，这里猜测shell将通配符翻译为了文件列表，并根据gzip的格式进行输入命令。


命令选项：
- -c,--stdout：将压缩的内容输出到标准输出，原文件不变。
- -d,--decompress：解压缩。
- -f,--force：强制覆盖旧文件。
- -l,--list：列出压缩包内储存的原始文件的信息(如，解压后的名字、压缩率等)
- -N，--name：压缩时保存原始文件的文件名和时间戳
- -r，--recursive：递归压缩
- -t，--test：测试压缩文件完整性
- -v，--verbose：冗余模式(即显示每一步的执行内容)
- -1、-2、…、-9：压缩率依次增大，默认为-6



## bzip2
![[Pasted image 20240923114544.png]]
用法:bzip2 [-cdkv#] 文件列表
- -d:解开压缩文件
- -c:压缩过程中的数据显示在屏幕上。
- -k:保留源文件
-  z:压缩的参数，默认值，可不加。
- -v:显示压缩比等信息。
- -#:压缩率是一个介于1~9的数值，数值越大压默认值为“6”，缩率越高，速度越慢。

## xz
![[Pasted image 20240923120440.png]]

## tar



![[Pasted image 20240923120422.png]]![[Pasted image 20240923120631.png]]
-r:添加文件到归档中
`tar -f ./tmp1.tar -r  ./q3
`



# 用户管理




![[Pasted image 20240927090541.png]]
UID：用户分类
![[Pasted image 20240927090757.png]]
GID：用户组分类

![[Pasted image 20240927090849.png]]

**当查看/etc/passwd文件的时候，发现所有用户信息中都包含一个x，这里x代表密码占位符，表示该用户的密码已经加密并存储在/etc/shadow文件中。**



![[Pasted image 20240927094548.png]]


![[Pasted image 20240929092928.png]]


## 切换用户

![[Pasted image 20240929085933.png]]

![[Pasted image 20240929090056.png]]
默认情况使用的是root用户。


配置sudo权限。
![[Pasted image 20240929090428.png]]
![[Pasted image 20240929090437.png]]


## 查看用户登录信息
id：显示用户的ID,以及所属用户组的ID
who: 显示当前登录系统的用户列表以及他们登录的时间和方式。
man:用于在Unix或Linux系统中查看命令的手册页。例如：`man who`、`man id`


# 用户组管理
![[Pasted image 20240929091438.png]]




# 磁盘管理


[挂载是指由操作系统使一个存储设备（诸如硬盘、CD-ROM或共享资源）上的电脑文件和目录可供用户通过计算机的文件系统访问的一个过程](https://zh.wikipedia.org/wiki/%E6%8C%82%E8%BD%BD)


一旦挂载完成，就可以像访问计算机内部硬盘上的任何其他文件夹一样来访问这个外部硬盘上的数据。挂载点只是一个目录，通过它来访问外部硬盘上的内容。



在虚拟机的磁盘管理步骤:
- (0)挂载磁盘
- (1)创建分区
		方式:MBR(MS-DOS)、GPT
		命令:fdisk、parted
- (2)创建分区文件系统
		文件系统:ext4、xfs等
		命令:mkfs -type xfs
- (3)挂载分区
		命令:mount   /dev/sda3 /home/dir
- (4)、
		命令:umount /dev/sdb2




df:查看磁盘的总容量、使用容量、剩余容量等
du:用来查看某个目录或文件所占空间大小，
Isblk:列出所有存储设备状态信息
blkid:列出文件系统及设备UUID
parted:磁盘分区情况


## MBR和GPT
参考：
https://www.cnblogs.com/tl542475736/p/8284628.html
https://www.cnblogs.com/baiting0317/p/3267786.html

硬盘命令
**hda一般是指IDE接口的硬盘**
**sda一般是指SATA接口的硬盘**

![[Pasted image 20241011085907.png]]

MBR分区: fdisk device-name
如: fdisk /dev/sdb
GPT分区:
gdisk device-name


## 格式化
![[Pasted image 20241011093259.png]]





# 进程管理
参考[linux进程相关命令 - xd\_xumaomao - 博客园](https://www.cnblogs.com/xumaomao/p/13054171.html)
pstree：查看进程树 ，（从全局角度查看，不具备列出单个的能力）
ps：当前系统的进程状态（从终端机视角看主机进程。）
例如：
a：显示现行终端机下的所有程序，包括其他用户的程序。
**u：以用户为主的格式来显示程序状况。**
x：显示所有程序，不以终端机来区分。
组合使用 ps aux



pgrep:根据进程名称查找进程id
![[Pasted image 20241129164310.png]]


kill:杀死进程

# 端口监听
netstat：(较新版本弃用)
![[Pasted image 20241129164823.png]]
![[Pasted image 20241129164834.png]]

lsof：查询端口信息
`lsof -i :80`
ss:替代netstat,性能更好
` ss -tln`

# 例题：
![[Pasted image 20240927092839.png]]

A:不正确
passwd中密码只是占位符，真正密码存在/etc/shadow中

B正确
https://cloud.tencent.com/developer/article/1722142

注意区分 /bin/passwd 和/etc/passwd
/bin/passwd具有强制位s权限。
s权限： 设置使文件在执行阶段具有文件所有者的权限，相当于临时拥有文件所有者的身份. 典型的文件是passwd. 如果一般用户执行该文件, 则在执行过程中, 该文件可以获得root权限, 从
而可以更改用户的密码。
![[Pasted image 20241027095320.png]]


/etc/passwd
![[Pasted image 20241027095200.png]]


注意：对于一般用户，可以通过sudo权限组获取管理员权限，

C：不正确
查询了解可以不设置密码

D：不正确

只有持有者和组成员具有读权限。

![[Pasted image 20241027095839.png]]