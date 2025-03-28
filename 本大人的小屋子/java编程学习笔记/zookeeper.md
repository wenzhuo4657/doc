# 概念
zookeeper是阿帕奇 hadoop项目下的一个子项目，该项目主要用于解决分布式的基础架构，而zookeeper是一个树形目录系统，用于解决使用分布式架构对节点管理不可避免会让系统更复杂的问题，具体运用于：统一配置管理，统一命名服务，[[分布式锁]]，集群管理。



服务端命令

- 启动：bin/zkServer.sh  start
- 查看状态： bin/zkServer.sh  status
- 停止： bin/zkServer.sh stop
- 重启： bin/zkServer.sh restart




## 数据模型

ZooKeeper 使用了一个树形结构的命名空间来表示其数据结构，将树中的每一个节点都称之为一个 Znode。
![image.png](http://obimage.wenzhuo4657.cn/20240612112427.png)



注意：
1，ZooKeeper 树中的每个节点都可以拥有子节点
2，每个 Znode 节点都存储了数据信息，同时也提供了对节点信息的监控等操作。


![image.png](http://obimage.wenzhuo4657.cn/20240612112651.png)



znode由三部分构成
- Stat：状态信息，用于存储该 Znode 的版本、权限、时间戳等信息；
- Data：实际存储的数据；
- Children：对子节点的信息描述；



# 命令操作


## ZookeeperClient

连接服务端，-server默认参数是这样，可以不加，如果连接远程需要添加
./zkCli.sh -server localhost:2181
客户端退出服务端连接
quit


节点的创建
-e :临时节点，存活周期为当前会话,退出会话自动删除
` create -e /zk-temp`
-s:顺序节点，自动追加数字表示区别
`create -s /zk-temp`
默认持久化，
`create /zk-permanent`


查看节点
`ls /` 当前节点，不包括子节点
`ls -s`(旧版本用`ls2 /`) 包括子节点，以及各种详细信息



## Zookeeper JAVA API
查看demo


###  watch事件监听
可用于实现发布订阅模式
![image.png](http://obimage.wenzhuo4657.cn/20240613175913.png)



## Curator


看demo

### [[分布式锁]]

![image.png](http://obimage.wenzhuo4657.cn/20240614092529.png)






