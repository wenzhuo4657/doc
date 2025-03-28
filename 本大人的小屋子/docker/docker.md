# docker 安装

- **`docker-ce`**：包含dockerd、docer-ce-cli和其他依赖（其中有部分containerd依赖，但不直接包含）
- **`docker-ce-cli`**：仅包含 Docker 客户端命令行工具，用于与远程 Docker 守护进程通信。
- **`containerd.io`**：独立的容器运行时，用于管理和运行容器。

这是最基本的三个安装，可以支持docker的基本功能。

通常安装时使用命令
`sudo apt install docker-ce docker-ce-cli containerd.io`
这是因为apt 默认会选择最新的稳定版本来安装,而docker-ce中的docker-ce-cli有时不是最新的。


此外还有docker-compose等其他支撑重要功能的包。可通过 `apt search docker`进行搜索


# docker desktop

配置文件地址
[Windows Docker Desktop 配置损坏无法启动还原配置_docker desktop 调整资源后无法重启-CSDN博客](https://blog.csdn.net/qq_40233736/article/details/121505890)
win11上： %userprofile%\.docker
![[Pasted image 20241102090325.png]]


一系列尝试后，
- daemon.json: ` 主机上的配置和集成的wsl实例配置有冲突关系。
		因此当我清空wsl实例中的`/etc/docker/daemon.json`,将配置保留在win11上的之后发现docker.service可以正常运行。
- dokcer.service:主机配置与wsl实例无关系，但似乎类似与集群，因为我win11上的docker一直在正常启动，我疑惑的是不知道为什么在wsl上无法连接win11上的docker,同时也对默认守护进程的监听端口/docker.sock发送过消息，可以正常发送
``` 
我的覆盖配置
[Service] 
ExecStart=  # 清空了默认的 ExecStart 指令
ExecStart=/usr/bin/dockerd -H fd:// -H tcp://127.0.0.1:2375  
```
因此我wsl实例中的docker连接的是本机的docker Service,事实上应该让其远程连接win11上的docker（docker的架构是cs架构，而并非集群）。

因此尝试删除docker其他组件，仅仅留下docker cli用于运行docker 客户端命令，此外使用`export DOCKER_HOST="unix:///var/run/docker.sock"`进行通信。

很不理解这里的目录，因此尝试重新安装一个实例，采用Ubuntu20LTS,没有进行docker安装，仅仅在docker desktop中开启集成，然后发现可以正常运行，查看环境变量并没有找到docker-host的配置，，这里只能认为docker-cli默认使用docker.sock，且发送消息可以正常接收返回。
![[Pasted image 20241102130448.png]]
![[Pasted image 20241102130522.png]]

因此，当docker desktop集成其他wsl实例时，
-  保存docker -cli命令的文件
		通过`apt list --installed`查看得知并没有安装包，但是可以通过`which docker`找到命令信息。
- 保存docker.sock等通信文件
		![[Pasted image 20241102131135.png]]

- 由于访问的是同一个守护进程，所以只要在desktop上登录，被集成wsl实例执行docker login不需要输入密码。
		![[Pasted image 20241102132148.png]]


# docker通信


## docker.sock文件
在我创建portainer被要求挂载数据卷 `/var/run/docker.sock`,如果不挂在就会报错无法连接守护进程，但提示信息中并没有显示为什么无法连接，因此好几个小时内我都在尝试启动守护进程，直到越来越跑骗，开始尝试启动tcp远程访问守护进程。

*portainer要求挂载docker.sock,实际上印证了在同一台机器任何地方、不同虚拟机之间，都可以通过该文件和docker Daemon进行通信。*

参考: https://docs.docker.net.cn/engine/

docker.sock:Unix域套接字文件，用于Docker客户端与Docker守护进程之间的本地通信。

参考: https://www.cnblogs.com/nufangrensheng/p/3569416.html
Unix域套接字文件：
**UNIX域套接字用于在同一台机器上运行的进程之间的通信**。虽然因特网域套接字可用于同一目的，但UNIX域套接字的效率更高。**UNIX域套接字仅仅复制数据**；它们并不执行协议处理，不需要添加或删除网络报头，无需计算检验和，不要产生顺序号，无需发送确认报文。





https://docs.docker.com/reference/cli/dockerd/#daemon-socket-option
docker deamon（守护进程）默认监听/var/run/docker.sock,除非明确禁止，否则一定会监听，该文件用于在同一台机器中docker通信。


![[Pasted image 20241101171522.png]]


# 守护进程
默认只允许本地访问，也就是docker.sock文件。

参考文档: https://docs.docker.net.cn/config/daemon/start/

##  启动方式：
sudo systemctl start docker（Ubuntu、 Debian中默认自动启动）
dockerd（属于命令行启动）




## 配置远程访问
https://docs.docker.com/reference/cli/dockerd/#daemon-socket-option

可以三种不同类型的 Socket监听 
- [Docker Engine API](https://docs.docker.com/engine/api/)`unix`请求
- `tcp`
- `fd`


配置文件 
https://docs.docker.com/engine/daemon/remote-access/#enable-remote-access
`/etc/docker/daemon.json`：守护进程的配置文件


`sudo systemctl edit docker.service` ：systemd服务单元配置，用于添加一个unit片段文件，以增加或覆盖默认设置。
![[Pasted image 20241101173258.png]]


#### tcp监听

使用tcp监听时，Docker 守护进程默认提供对 Docker 守护进程的未加密和未认证的直接访问。但是可以使用 [内置的 HTTPS 加密套接字](https://docs.docker.com/engine/security/protect-access/)或在其前面放置安全的 Web 代理来保护守护进程。




#### 非 常规
[Windows宿主机如何访问连接虚拟机中的Docker容器\_本地如何访问docker虚拟ip-CSDN博客](https://blog.csdn.net/NUT_0/article/details/131459920)

[blog.51cto.com/u\_16213431/8729845](https://blog.51cto.com/u_16213431/8729845)

# docker 架构
docker是一个C/S模式的架构，后端是一个松耦合架构，模块各司其职。


## dns服务器

[Docker的网络配置 4 内嵌的DNS server\_docker内部dns-CSDN博客](https://blog.csdn.net/m0_45406092/article/details/105662836)
地址： 127.0.0.11
可解析容器名称、主机名称，而这二者尽可能地统一配置，便于阅读。



---
Ubuntu18:安装了docker-credential-pass、还有config.json配置文件。