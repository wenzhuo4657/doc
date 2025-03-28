# 基本概念

[谷歌技术"三宝" - 储巍 - 博客园 (cnblogs.com)](https://www.cnblogs.com/javhu/archive/2013/03/25/cyue_hadoop_google.html)

数据库推荐：
[OceanBase](https://www.modb.pro/wiki/34)
[openGauss](https://www.modb.pro/wiki/601 "openGauss-百科")
[PostgreSQL](https://www.modb.pro/tag/PostgreSQL)

数据仓库：主要是汇总（关系数据库）
数据湖：既有明细也有汇总（结构化数据库，非关系数据）
湖仓一体：

[一文读懂数据仓库、数据湖、湖仓一体 - MasonZhang - 博客园 (cnblogs.com)](https://www.cnblogs.com/miketwais/articles/data_lakehouse.html)



- 存储模式
- 架构模式
- 设计模式



[分布式 (javalearn.cn)](https://www.javalearn.cn/#/doc/%E5%88%86%E5%B8%83%E5%BC%8F/%E9%9D%A2%E8%AF%95%E9%A2%98)

TRDB：ACID(强存储模式)
NOSQL:BASE（弱存储模式，例如reids默认情况下开启RDB日志，通过指定时间快照进行持久化，相对较弱可能会丢失很多数据。）


|         |          |
| ------- | -------- |
| 横向/水平扩展 | 基于多服务其扩展 |
| 纵向/垂直扩展 | 从服务器功能扩充 |




# nosql的存储模式

主要四大类
- 键值存储模式
- 文档存储模式
- 列族存储模式:

参考:[OLTP、OLAP列数据库、列族数据库的区别 - 昕友软件开发 - 博客园 (cnblogs.com)](https://www.cnblogs.com/starcrm/p/13822100.html)
OLTP：基于行存储的关系数据库，写入速度极快，用于数据记录修改场景
OLAP：基于列存储，查询速度极快，用于海量数据分析
		1，数据类型：与OLTP作区分，一列数据的类型相同
		2，索引：对于关系型数据库来说，索引是高效查询的手段，但会耗费空间，而列族数据本身就是索引
		3，读取和写入数据：行数据库操作数据的单位是行，列存储将一行记录拆分成单列保存，因此插入次数多耗时长，但替换而来的是数据读取速度块，不必像行数据库按行，而是集合的一段或者全部，不存在冗余性问题。

拓展： 在openGauss中，可以通过声明控制是否进行列族插入。
- 图存储模式




