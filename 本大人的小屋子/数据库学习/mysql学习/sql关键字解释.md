# union 和**UNION ALL**

union：只包含唯一的结果行。如果两个 `SELECT` 语句中有相同的行，`UNION` 只会保留一个实例。

**UNION ALL**：保留所有结果行，包括重复的行
[SQL常见面试题总结（1） | JavaGuide](https://javaguide.cn/database/sql/sql-questions-01.html#组合查询)

```
SELECT column_name(s) FROM table1
UNION ALL
SELECT column_name(s) FROM table2;
```

**两者必须返回相同列**
*合并多个查询的结果集仅仅是简单的拼接*


## IFNULL(expression, default_value)
一个sql函数



# DISTINCT
对结果集进行去重
`select DISTINCT salary  from Employee `

注意：这种方式是针对于行进行去重，只不过是当前行只有一列看起来像是去重某一列，


拓展：
`SELECT DISTINCT COUNT( column_name)`  去重的是结果集中的虚拟表

`SELECT COUNT(DISTINCT column_name)` 去重的是行` colunm_bame` 这一行，这是DISTINCT的特点。


但是对于conunt来说
`SELECT DISTINCT column1, column2, ..., columnN, COUNT(*)` 计数多行不重复。

`SELECT COUNT(DISTINCT column_name)` 计数单列不重复。




# null相关

当使用数字和null进行判断时，其结果为null，


# 聚合
## group by

GROUP BY 的作用是将一组数据按照指定的列（或表达式）进行分组，并将每组数据“浓缩”为一行。

如果列在 GROUP BY 中，可以直接使用。
如果列不在 GROUP BY 中，则必须使用聚合函数（如 SUM()、COUNT() 等），否则会触发 ONLY_FULL_GROUP_BY 错误


例：

```
1,分组依据为：column1, column2唯一，
SELECT column1, column2, SUM(column3)
FROM table_name
GROUP BY column1, column2;

2，分组依据：order_id唯一
SELECT order_id, SUM(quantity) AS total_quantity
FROM orderitems
GROUP BY order_id;
```

实际上这有点像索引不允许冲突，结合sql执行结果以虚拟表返回，我们可以认为在这个虚拟表上增加了一个索引特征。

## 聚合函数

SELECT SUM(quantity) FROM orderitems;

聚合函数可以单独对某列使用，但是同样，如果存在非聚合列，则会报错`ONLY_FULL_GROUP_BY`
`SELECT order_num, SUM(quantity) FROM orderitems;`

个人理解：聚合函数从设计逻辑上是和group by共同工作的，他们一起完成了聚合某些行这一目标，而报错`ONLY_FULL_GROUP_BY`则是针对于聚合列和非聚合列同时查询的错误。

两者单独使用的场景无非是单独的列进行了约束，从而避免触发`ONLY_FULL_GROUP_BY`


- 如果不存在group by ,聚合函数直接对结果集操作
- 如果存在group by, 执行：group by>聚合 >having过滤。（分组时，聚合函数对每个组单独进执行）




# count(* ), count(1), count(列名）
count(* )  == conunt (1),而count(列名)则是对非null值进行统计



# mysql索引描述

查询表结构：
```
desc 表名；
```

- PRI：值唯一，不能为空。
- **UNI**：值唯一，允许空值，但不允许重复的非空值。
- **MUL**：允许列中的值重复。

具体的键类型：
PRI： PRIMARY KEY、主键的联合索引的全部列
UNI：UNIQUE KEY
MUL：（唯一索引、普通索引）联合索引的第一列、外键、普通索引


**上述描述针对的是键的特征：是否允许null、是否允许重复，而并非具体的键类型**


尝试为某个列添加外键之后再添加唯一索引，可以清除的看到索引描述的变化。
```
CREATE TABLE my_class (
id INT PRIMARY KEY , -- 班级编号
`name` VARCHAR(32) NOT NULL DEFAULT '');
CREATE TABLE my_stu (
id INT PRIMARY KEY , -- 学生编号
`name` VARCHAR(32) NOT NULL DEFAULT '', class_id INT, -- 学生所在班级的编号
-- 下面指定外键关系
FOREIGN KEY (class_id) REFERENCES my_class(id))
```

此时索引描述为
![](img/Pasted%20image%2020250210112921.png)

```
ALTER TABLE my_stu ADD  UNIQUE KEY(class_id)
```

![](img/Pasted%20image%2020250210113217.png)


**上述示例中，对同一个字段设置了不同索引，对应到文件中则是不同索引文件，这在不同数据库中可能有不同的要求，例如mysql允许相同类型，但是oracle则不允许，具体应用中要额外的注意这一点。**

[技术分享 | MySQL 可以对相同字段创建不同索引？](https://opensource.actionsky.com/技术分享-mysql-可以对相同字段创建不同索引？/)





