

#### 1，版本锁定

```
<parent>  
<groupId>org.springframework.boot</groupId>  
<artifactId>spring-boot-starter-parent</artifactId>  
<version>2.5.1</version>  
</parent>
pom.xml语言中添加这个依赖，其内部规定了一系列包的版本
```

使用中注意，已有的包不需要写版本号，即`<version>*.*.*.*</version>`,但如果是版本锁定中没有的，则需要自定义版本号
例：
```
<dependencies>  
<dependency>  
<groupId>org.springframework.boot</groupId>  
<artifactId>spring-boot-starter-web</artifactId>  
</dependency>  
<dependency>  
<groupId>org.mybatis.spring.boot</groupId>  
<artifactId>mybatis-spring-boot-starter</artifactId>  
<version>2.1.4</version>  
</dependency>  
</dependencies>
```


#### 2，约定大于配置
1，,如springmvc中设定扫描`controller`包一样，boot中默认了controller包作为mvc的扫描包，意味这路由等相关设定必须在该包中
2，yml的文件名默认为`application.yml`
3，测试类的包结构和springboot启动类的包结构一样，或者指定启动类@SpringBootTest(classes = HelloApplication.class)
4，在resourse目录下设置
![[img/Pasted image 20231206193220.png]]

表示静态资源根目录





