*设计初衷
1,解决传统数据库在高频率的写sql情境下的无力
2,对于数据量过于庞大、或者查询复杂的情况下，减小传统数据查询速度的高能耗

![[img/Pasted image 20231229171549.png]]

参考笔记
[Redis7笔记(完结)-CSDN博客](https://blog.csdn.net/m0_55993923/article/details/129718974)

## 启动方式

查询redis状态
[root@localhost redis-7.0.12]# ps -ef |grep redis
root      99242      1  0 14:34 ?        00:00:00 redis-server 0.0.0.0:6379
root      99325  99202  0 14:36 pts/0    00:00:00 grep --color=auto redis



### 默认启动/前台启动

任意位置，在命令行输入 redis-server
缺点：在启动窗口内无法执行其他操作，只能打开新的窗口进行操作


### 指定配置操作/后台启动

修改redis.conf文件


***** 一定要提前备份 ：[root@localhost redis-7.0.12]# cd  redis.conf redis.conf.bck

命令行输入:[root@localhost redis-7.0.12]# redis-server redis.conf 



### 开机自启




## 客户端登录

链接服务端


![[img/Pasted image 20240101143904.png]]
-p 指定连接的端口
-h 连接的ip，默认为127.0.0.0 本机环回地址
-a 访问密码


测试是否成功
![[img/Pasted image 20240101144109.png]]

## 数据类型


![[img/Pasted image 20240101153306.png]]



https://redis.io/docs/data-types/

![[img/Pasted image 20240101153955.png]]
![[img/Pasted image 20240101154009.png]]





# redis事务：
![image.png](http://obimage.wenzhuo4657.cn/20240509134601.png)

注意：如果事务为提交，还处于入队中，入队了语法错误的雨具，则该事务被标记，不能被执行。



![image.png](http://obimage.wenzhuo4657.cn/20240509140158.png)
也可以使用 no watch 取消监控



# java连接
## jedis
![image.png](http://obimage.wenzhuo4657.cn/20240509151505.png)



注入ioc实例：
```
@Configuration  
@ConfigurationProperties(prefix = "spring.redis")  
public class JedisConfig {  
  
private String database;  
private String host;  
private String port;  
  
@Value("${spring.redis.jedis.pool.max-active}")  
private int maxActive;  
@Value("${spring.redis.jedis.pool.max-idle}")  
private int maxIdle;  
  
@Value("${spring.redis.jedis.pool.min-idle}")  
private int minIdle;  
  
  
@Bean  
public JedisPool jedisPool(){  
JedisPoolConfig poolConfig =new JedisPoolConfig();  
poolConfig.setMaxTotal(maxActive);  
poolConfig.setMinIdle(minIdle);  
poolConfig.setMaxIdle(maxIdle);  
JedisPool jedisPool=new JedisPool(poolConfig);  
return jedisPool;  
  
}  
  
  
public String getDatabase() {  
return database;  
}  
  
public void setDatabase(String database) {  
this.database = database;  
}  
  
public String getHost() {  
return host;  
}  
  
public void setHost(String host) {  
this.host = host;  
}  
  
public String getPort() {  
return port;  
}  
  
public void setPort(String port) {  
this.port = port;  
}  
  
public int getMaxActive() {  
return maxActive;  
}  
  
public void setMaxActive(int maxActive) {  
this.maxActive = maxActive;  
}  
  
public int getMaxIdle() {  
return maxIdle;  
}  
  
public void setMaxIdle(int maxIdle) {  
this.maxIdle = maxIdle;  
}  
  
public int getMinIdle() {  
return minIdle;  
}  
  
public void setMinIdle(int minIdle) {  
this.minIdle = minIdle;  
}  
}
```

配置文件
```
spring:  
redis:  
database: 0  
host: 127.0.0.1  
port: 6379  
jedis:  
pool:  
max-active: 8 #最大连接数  
max-idle: 8 # 最大空闲数  
min-idle: 1 #最小空闲数
```

## spring-Data-redis
![image.png](http://obimage.wenzhuo4657.cn/20240509151650.png)


maven引入依赖

```
<dependency>  
<groupId>org.springframework.boot</groupId>  
<artifactId>spring-boot-starter-data-redis</artifactId>  
</dependency>  
<dependency>  
<groupId>org.apache.commons</groupId>  
<artifactId>commons-pool2</artifactId>  
</dependency>//jedis连接池底层也是他
```


![image.png](http://obimage.wenzhuo4657.cn/20240509152944.png)

提供两种客户端连接
RedsiTempliate<String,Value>：使用json处理value
StringRedsiTempliate<String,String>:k,v均使用string，但需要手动序列化和反序列化，

![image.png](http://obimage.wenzhuo4657.cn/20240509165457.png)

结合上面两种配置：使用了redisTemplate达到了StringRedsiTempliate的效果
![image.png](http://obimage.wenzhuo4657.cn/20240509165912.png)


# redis持久化
redis7提供了rdf+aop模式，在同一个实例中同时使用

## RDB（默认方式）
以指定时间间隔执行数据集的时间快照


配置：save  900 1（表示每900秒如果有一个key变化，则创建一个snapshots快照,）
注意：如果意外宕机，可能会丢失最后一次快照
rdb保存文件：dump.rdb

触发
- 1，达到设定频次
- 2，服务停止时（默认）

![image.png](http://obimage.wenzhuo4657.cn/20240509172534.png)



![image.png](http://obimage.wenzhuo4657.cn/20240509172514.png)

注意：fork过程如果主进程出现了需要修改的操作，不会干扰到子进程，也不能直接修改子进程锁定的内存区域，而是单独处理。其实就是不从修改数据的角度，而是从指令执行的角度，必须等待上一指令的完成再来操作下一指令。


rdb问题
（1）两次rdb之间有丢失数据的风险
（2）fork非常耗时
## AOF
持久性记录服务器收到的*写命令*，在重启时执行


默认关闭，需要修改redis.conf开启
```
appendonly yes #是否开启AOF 默认no
appendfilename "appendonly.aof"# AOF文件名
"appendon1ydir" #Redis7新增,用于储存所有AOF文件的目录名称,redis6之间都是只有一个aof文件
```

记录频率修改，同样修改redis.confw文件
```
appendfsync always #表示每执行一次写命令，立即记录到A0F文件 性能最差ī
appendfsync everysec #写命令执行完先放入A0F缓冲区，然后表示每个1秒钟将缓冲区数据写入A0F文件，默认(最多丢失1秒内的数
据)
appendfsync no #写命令执行完放入AOF缓冲区，由操作系统决定何时将缓冲区内容写会磁盘
```

### redis7
#### AOFRW(redis7.0之后提出)
移除冗余的写命令，转化为等效的写命令

例如：
原本打算写入 set aa 10 ,set aa 20 ,set aa 30,最后只写入set aa 30

触发条件：
1，自动：
```
-- redis.conf中默认配置，可修改
auto-aof-rewrite-percentage 100  -- 增加百分比，表示一倍
auto-aof-rewrite-min-size 64mb  --  重写后需满足的文件大小，默认64mb
```
2,手动
`bgrewriteaof`

原理：
要点
1、重写操作不执行在原文件中，而是一个临时文件中
2、如果重写失败，即不符合设置文件大小，原文件仍然保存
3.重写期间新的写命令会同时追加进*fork的临时文件*和**原文件**

![image.png](http://obimage.wenzhuo4657.cn/20240509175951.png)




#### aof文件拆分为以下三种类型

| fileType | des                                                                                             | numbers |
| -------- | ----------------------------------------------------------------------------------------------- | ------- |
| BASE     | 表示基础的AOF，它一般由子进程通过重写产生                                                                          | 1       |
| INCR     | 表示增量AOF，它一般会在AOFRW开始创建，                                                                         | >=1     |
| HISTORY  | 表示历史的AOF，它由BASE和INCR AOF变化而来，每次**AOFRW**成功完成时，本次AOFRW之前对应的BASE和INCR都变成HISTORY，此类型AOF会被Redis自动删除 |         |



## AOF+RDB（redis4之后）
注意：
1，会优先加载AOF
2，重启时默认只加载aof文件，只有当aof文件不存在时，会找rdb文件


开启方式：设置redis.conf
`aof-use-rdb-premble yes #开启混合模式 默认为no`



# redis发布、订阅模式
![image.png](http://obimage.wenzhuo4657.cn/20240509181221.png)



# 主从复制


作用
- 数据冗余:主从复制实现了数据的热备份，是持久化的一种数据几余方式
- 故障恢复:主节点一旦出现问题，可以由从节点提供服务，避免出现程序不可用的情况，实现快速故障恢复
- 负载均衡:在主从复制的基础之上，配合读写分离，主节点提供写服务，由从节点提供读服务，分担服务器负载，尤其是在读多写少场景下，可以大大提高Redis并发量
- 高可用(集群)基石:主从复制是集群和哨兵模式的基础。


# 关于进程
对于通信，redis使用io多路复用处理客户端、集群节点。
对于保存文件，redis会fork一个子进程用于保存文件到磁盘。



## io多路复用
[blog.51cto.com/u\_16213566/7626006](https://blog.51cto.com/u_16213566/7626006)
