
详解：https://mp.weixin.qq.com/s/SvhP3009BkomJbemKqBsvw
code：https://github.com/wenzhuo4657/demo.git




# 基础

DI:
通过注入方式分类：
属性注入
构造器注入：多个构造器时，可以根据注解@Autowired指定由那个构造器创建bean
通过管理bean的方式：
xml管理
注解管理


## xml管理bean

根据标签，有构造器或者set注入，两个标签可以在同个bean中使用
![image.png](http://obimage.wenzhuo4657.cn/20240527111157.png)


![](http://obimage.wenzhuo4657.cn/20240527111230.png)



获取bean:读取xml文件，然后根据id等锁定某个【配置
```
@Test
public void testDIByConstructor(){
    ApplicationContext ac = new ClassPathXmlApplicationContext("spring-di.xml");
    Student studentOne = ac.getBean("studentTwo", Student.class);
    System.out.println(studentOne);
}

```
## 注解管理

同时注意开启组件扫描，在springboot中有默认配置，spring需要手动配置
且注意，注解注入同样支持构造器注入和属性注入，


### @Autowired注入

单独使用@Autowired注解，**默认根据类型装配**。【默认是byType】

### @Resource注入
1，属于jdk扩展包，**高于JDK11或低于JDK8需要引入以下依赖。**
```
<dependency>
    <groupId>jakarta.annotation</groupId>
    <artifactId>jakarta.annotation-api</artifactId>
    <version>2.1.1</version>
</dependency>

```
2，**注解默认根据名称装配byName，未指定name时，使用属性名作为name。通过name找不到的话会自动启动通过类型byType装配。**





## 高级


## list、map
list和map注入都是根据关键字instanceof在bean中找到对应value，map的Key为bean的名称





##  检测创建，避免重复

```
@Bean("redisson02")   
@ConditionalOnMissingBean//默认条件判断为是否有相同bean存在  
public String redisson02() {  
    return "模拟的 Redis 客户端 02";  
}
```



## 根据某变量的值来控制是否注入

```

@Bean
@ConditionalOnProperty(name = "feature.flag.enable", havingValue = "true") 
		public SomeFeatureBean someFeatureBean() { 
			return new SomeFeatureBean();
	} 
```






# 相关注解
1，@Autowired
作用范围：
@Target({ElementType.CONSTRUCTOR, ElementType.METHOD, ElementType.PARAMETER, ElementType.FIELD, ElementType.ANNOTATION_TYPE})
spring会自动根据byType、ByName等类型匹配来找到对应的bean,
`required=false` 阻止找不到对应bean时的默认行为：抛出异常


2，@Qualifier
指定类型匹配的beanName，可搭配@Resouse、@Autowired来使用，用于指定某个具体的bean


3,**@Primary**
多个相同类型的实现，可以使用该注解来指定默认注入



4,@bean注解
将方法返回值注入容器，且支持指定beanName

拓展： @ConditionalOnMissingBean ：只有当容器中不存在特定bean时才会注入
@ConditionalOnProperty(value = "sdk.config.enabled", havingValue = "true", matchIfMissing = false)


注意：该注解条件判断的是特定的bean而不是某个实例，也就是说，beanName，beanType等条件，且注意其默认条件判断为是否有相同的bean存在

