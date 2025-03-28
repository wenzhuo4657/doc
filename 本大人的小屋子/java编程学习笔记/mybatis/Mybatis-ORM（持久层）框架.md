

********不要乱用标签<>,其底层会识别，md


持久层框架：


![[mybatis/img/20231104_1.png]]




官网链接：
[MyBatis中文网](https://mybatis.net.cn/)


# 获取方法参数


### 前情提示：

假如我们已经完成如下示例,此时如果findAll需要传入一个参数或者多个参数，我们该如何在配置文件中获取这个参数并使用呢？

```
<?xml version="1.0" encoding="UTF-8" ?>  
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >  
<mapper namespace="org.example.Dao.UserDao">  
<select id="findAll" resultType="org.example.entity.User">  
select *from asd  
</select>  
</mapper>
```



### demo ：

#### 单参数：
##### 语法：#{ 方法参数名 }


###### 基本类型 or 包装类
```
-- 接口文件
public interface UserDao {    
User findbyid(Integer id);  
}

-- xml文件
<select id="findbyid" resultType="org.example.entity.User">  
select *from asd where id=#{id}  
</select>
```

###### POJO对象：指多种属性

```
例如：
public class User {  
private String name;  
private int id;  
private int nn;
}
```


```

使用：

--接口文件
User find(User user);


-- xml文件
<select id="find" resultType="org.example.entity.User">  
select *from asd where id=#{id} and name=#{name} and nn=#{nn}  
</select>
%% 直接使用对象的属性名即可 %%


```


###### map<k,v>


```

使用：

--接口文件
User find(map user);


-- xml文件
<select id="find" resultType="org.example.entity.User">  
select *from asd where id=#{id} and name=#{name} and nn=#{nn}  
</select>

%% 填入其中id name nn 为传入map集合的key %%

```

#### 多参数



 注解：@Param("重命名")

` User finds(@Param("id") Integer id,@Param("name") String name);`

对参数进行重命名后，其使用方法又绕回单参数的情况了。



注意：有默认写法，但这里不推荐
注意2：该注解单参数也可以使用，其功能仅代表可以给传参重命名，使其可以在xml文件中以另一种名字解析




## 核心类

### SqlSessionFactory 

   主要获取数据库连接

  `SqlSession session =factory.openSession(boolean is);`
   如果is == true,代表自动提交事务，
   如果is == false，表示不会自动提交事务
   且注意：无参构造器相当于is == false 的情况，不会自动提交事务
   

### SqlSession

提供执行sql命令的基本方法和事务操作的相关方法



# 增删改查


注意1：如果SqlSessionFactory没有开启自动提交事务，请手动提交事务，
`session.commit();`
注意2：--dao.xml文件有五种sql语句相关标签，分别是

```

<insert ----  
<delete ----  
<update ----  
<select ----
```


# [mybatis](https://mybatis.net.cn/) 配置

其中有多种配置文档，可自行查阅


### <mapper/>自动导入

目录结构:
<br>
![[mybatis/img/20231105_1.png]]
mybatis-config.xml文件配置

![[mybatis/img/20231105_2.png]]



# 动态sql

前面的代码里的dao实现中，只能进行简单一条sql语句的使用，
而动态sql就是指能够在dao层的实现中添加除去简单sql语句外的其他逻辑操作


### 本质：
使用  ***“简单的编程逻辑操作和拼接字符串作为底层”  的一系列标签而已




### <if  /text="条件判断语句"/>
示例：

```
<select id="find" resultType="org.example.entity.User">  
select *from asd where id=#{id} and name=#{name} 
<if test="nn!=null">  //注意，test内直接使用参数名即可，
and nn=#{username}  
</if>  
</select>
```

如果test == true ,标签内语句进行拼接
反之则不拼接


### <trim  />

#### 清除


##### prefixOverrider属性
：清除sql语句的前缀，剩余内容拼接


```
<select id="find" resultType="org.example.entity.User">  
select *from asd  
<trim  
prefixOverrides="where|and"> 
//此处使用|字符，where和and不能有空格，
//且条件内清除内容只会生效一个
//清除内容数量可以添加为多个，
where and  
</trim>  
</select>
```

##### suffixOverrides
:清除sql语句后缀，用法和prefixOverrider属性相同


```
<trim  
suffixOverrides="where|and">  
id=#{id} and  
</trim>
```


#### 添加:
标签体内有内容则添加没有，则不添加
***不同于清除属性可以清除多种，添加属性只能添加一种



suffix：添加前缀
prefix：添加后缀



```
<trim  
suffix="where">  
id=#{id}  
</trim>


<trim  
prefix="where">  
id=#{id}  
</trim>


```

#### 综合使用示例

![[mybatis/img/Pasted image 20231105151503.png]]




### <where />
动态的拼接where，并据情况删除前缀and|or
示例：

![[mybatis/img/Pasted image 20231105151822.png]]



###  < set/>

![[mybatis/img/Pasted image 20231105152752.png]]



###   < foreach />

传入一个对象，不确定里面有几个参数，该标签可以通过遍历的方式读取对象内所有属性，然后根据自定义的开头和结尾以及间隔符拼接到sql语句中
例如下图：
属性解释：open：语句开头  close:语句结尾 separator:遍历元素间隔符号  collection:遍历元素  item：将collection表示的对象依次复制给其
![[mybatis/img/Pasted image 20231106174730.png]]

实际执行sql语句
` select *from User where id in(id,id,id)` id数量动态显示



 ###  <choose />


 
![[mybatis/img/Pasted image 20231106180831.png]]
实际含义
![[mybatis/img/Pasted image 20231106180913.png]]
![[mybatis/img/Pasted image 20231106180844.png]]


### sql片段抽取
场景，在dao.xml编写sql语句时，可能会遇到重复的sql语句片段，为提高效率，我们通常这样做

![[mybatis/img/Pasted image 20231106182237.png]]


实际执行效果
` select id,username,age,address from user`




# ResultMap：
***解决表的列名和实体类属性名的映射的不相等、未注册问题
***自定义映射规则

![[mybatis/img/Pasted image 20231107181849.png]]


```
//dao.xml文件使用

<resultMap id="orderMap" type="org.example.entity.Order">  
<id column="id" property="id"></id>  
<result column="user_id" property="userid"></result>  
</resultMap>  
注意：当列名和要映射的属性名的相同时，可以不用谢，会自动映射，也就是说，只用写需要改变的
  
<select id="findAll" resultMap="orderMap">  
select id,time,price,remark,user_id from `order`  
</select>

```

在某些情况下，我们可能想要自动映射，例如多表查询
`autoMapping="false"`
```
<resultMap id="orderMap" type="org.example.entity.Order" autoMapping="false">  
<id column="id" property="id"></id>  
<result column="user_id" property="userid"></result>  
</resultMap>

```




继承属性;
`extends="映射规则id"`


具体使用
![[mybatis/img/Pasted image 20231107183017.png]]

等价于
![[mybatis/img/Pasted image 20231107183034.png]]



# 多表查询

## 关联查询
注意：以下知识点具有强烈的功能性，也就是说，设置出来专门为了解决一对一或者一对多的问题，请不要使用逻辑角度理解，更多的从解决问题的角度去理解这样设计的合理性
### 一对一


问题：如果实体类中含有一个另外一个实体类属性，
场景代码
```
public class Order {  
private Integer id;  
private Date time;  
private String price;  
private String remark;  
private String userid;  
private User user;

-----
}
```

要实现的sql语句
```
SELECT 
o.id,o.time,o.price,o.remark,u.id uid,u.username,u.age,u.address
FROM
`order` o,`user` u
WHERE o.user_id=u.id AND o.id=2;
```

根本问题：此时由于默认映射规则，无法将user实体类内部属性填充返回

#### 解决办法1：


使用:ResultMap自定义user实体类内部属性和相关列名的映射关系
这里不做演示

#### 解决办法2：
与方法1并无实际区别
仅仅是：
< association/>标签的使用，在标签内部去定义user实体类的映射规则

![[mybatis/img/Pasted image 20231107190914.png]]
### 一对多

本质为：
<collection />属性的使用，其代表着实体类中的是实体类中的实体类集合，其内部定义实体类集合中的实体类的映射规则
![[mybatis/img/Pasted image 20231107193009.png]]

![[mybatis/img/Pasted image 20231107192925.png]]




##  分步查询

本质：在一个查询内，内置自动调用另一个查询方法，接收参数并使用。






## 分页查询
待更


# Mybatis缓存问题

![[mybatis/img/Pasted image 20231107193846.png]]

缓存级别直观现象，不同缓存级别的连接执行相同的sql，其返回结果可能出现不同
![[mybatis/img/Pasted image 20231107194633.png]]


![[mybatis/img/Pasted image 20231107194543.png]]
