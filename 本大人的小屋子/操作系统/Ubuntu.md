## 关于网卡配置

参考：
[Fetching Title#lgtt](https://blog.csdn.net/allway2/article/details/121949816)
在ubuntu18之后，采用 Netplan 来配置网络接口。Netplan 基于 YAML 的配置系统。


## 关于gpg密钥
[GPG入门教程 - 阮一峰的网络日志](https://www.ruanyifeng.com/blog/2013/07/gpg.html)（版本陈旧教程，和当前版本不同了）

Ubuntu系统上




## 关于apt


查看已安装软件版本
`apt show `

查看软件安装的位置
` dpkg -L  软件名称`

搜想相关软件包 
`apt search  软件包名称/相关单词`
例如： `apt search jdk `

查看软件版本
`apt-cache madison maven`

安装指定版本
`sudo apt-get install maven=3.6.0-1~18.04.1`
也可不指定，默认安装最新版本
`sudo apt-get install maven`




