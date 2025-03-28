
参考文章：[MySQL三大日志(binlog,redolog,undolog)详解 - 掘金 (juejin.cn)](https://juejin.cn/post/7090530790156533773)

# binlog二进制日志
*先把日志写到binlog cache，事务提交的时候再把binlog cache写到binlog文件中*

作用：
1，`binlog`会记录所有涉及更新数据的逻辑规则，并且按顺序写。
例如（数据库的**数据备份、主备、主主、主从**）


逻辑修改
`Query: UPDATE t SET col1 = 'new_value' WHERE id = 1;`





# redo log重做日志
*redo log在事务执行过程中可以不断写入*

物理修改
`Page ID: 5, Offset: 100, Old Value: 'old_value', New Value: 'new_value'`

InnoDB独有，当msyql实例挂掉或者宕机了，会使用该日志恢复数据，

需要注意的是刷盘到redolog物理文件的时机，
1，`Innodb`存储引擎有一个后台线程，每隔`1`秒，就会把会`redo log buffer`中的内容写入到文件系统缓存`page cache`，然后调用`fsync`刷盘。
2，提供了刷盘策略（和1无关，该参数设置是独立的），`innodb_flush_log_at_trx_commit`参数
- 0：设置为0的时候，每次提交事务时不刷盘。
- 1：设置为1的时候，每次提交事务时刷盘。
- 2：设置为2的时候，每次提交事务时都只把`redo log buffer`写入`page cache`。


![image.png](http://obimage.wenzhuo4657.cn/20240511153830.png)


参考文章：[一文看懂 | 什么是页缓存（Page Cache）_pagecache-CSDN博客](https://blog.csdn.net/goTsHgo/article/details/122256991)

pagecache（linux系统概念）:
文件一般存放在硬盘（机械硬盘或固态硬盘）中，CPU 并不能直接访问硬盘中的数据，而是需要先将硬盘中的数据读入到内存中，然后才能被 CPU 访问。
为了提高读写文件速度，且读写内存比读写硬盘的速度快很多,Linux 内核使用 `页缓存（Page Cache）` 机制来对文件中的数据进行缓存




注意这里为什么不直接将数据页（pagecache）直接刷盘呢？是因为刷盘时可能数据页没有写满，有大量空白，所以我们使用redolog进行记录，


# 两阶段提交（redo log 和binlog）
两阶段值指的是redo log日志，binlog只是起到印证redolog是否出错的作用

![image.png](http://obimage.wenzhuo4657.cn/20240511160347.png)

由于redolog和binlog写入时机的不同，在某些时候可以相互印证，例如，
1，redolog处于prepare阶段，但是在binlog中有对应的事务id,这里数据库就不会回滚事务，
2.redolog处于prepare阶段,但在写入binlog时异常，也就是说没有对应事务id,这里数据库会回滚事务。

注意:如果redolog有commit，那就直接执行提交。

# 慢查询日志
MySQL的慢查询日志可以记录**执行时间超过阈值**的SQL查询语句

参考文章[你写的每条SQL都是全表扫描吗 - 掘金 (juejin.cn)](https://juejin.cn/post/7367163252935786559#heading-0)



# undo log(回滚日志)
作用:在`MySQL`中恢复机制是通过`undo log`（回滚日志）实现的,在发生异常时，对已经执行的操作进行回滚，保证事务的原子性。

且注意： *回滚日志会先于数据持久化到磁盘上*



# 错误日志




# 通用查询日志
用户所有操作都会记录到通用查询日志中
