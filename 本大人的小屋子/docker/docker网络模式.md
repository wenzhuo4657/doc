[docker(7)：docker容器的四种网络类型 - 光阴8023 - 博客园](https://www.cnblogs.com/wangxu01/articles/11316447.html)

# Bridge（默认模式）
在主机上创建一个名为docker0的虚拟网桥，然后将容器加入到该子网中。


![[Pasted image 20241125125412.png]]

![[Pasted image 20241125125944.png]]


注意：使用docker desktop时，不能再主机上看到docker 0网卡，且主机和容器互联也有问题。一般使用端口发布或者host网络模式。
[在 Docker Desktop 上探索网络功能 | Docker 文档](https://docs.docker.net.cn/desktop/networking/#i-want-to-connect-to-a-container-from-the-host)

# container

container网络类型：和其他容器共享网络IP

容器直接端口尽量不要冲突，如果冲突，先到先得。


# host
与主机共享Network Namespace



# 代理配置

[如何优雅的给 Docker 配置网络代理-腾讯云开发者社区-腾讯云](https://cloud.tencent.com/developer/article/1806455)
