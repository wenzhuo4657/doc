[Maven build之pom.xml文件中的Build配置_maven build作用-CSDN博客](https://blog.csdn.net/u010010606/article/details/79727438?spm=1001.2014.3001.5506)


##  `<filtering>`
 
```
<resources>  
<resource>  
<directory>src/main/resource</directory>  
<!-- maven的模版变量可以在其资源目录下使用，打包时自动替换，实现动态替换配置文件  
具体模版变量来源有：properties、profiles、环境变量、系统变量、等 -->  
<filtering>true</filtering>  
</resource>  
<resource>  
<directory>src/assembly/resource</directory>  
<filtering>false</filtering>  
</resource>  
</resources>
```


## `<profiles>`   环境变量配置

```  <profiles>
        <profile>
            <id>dev</id>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
            <properties>
                <java_jvm>-Xms1G -Xmx1G -server  -XX:MaxPermSize=256M -Xss256K -Dspring.profiles.active=test -XX:+DisableExplicitGC -XX:+UseG1GC  -XX:LargePageSizeInBytes=128m -XX:+UseFastAccessorMethods -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/export/Logs/xfg-frame-archetype-lite-boot -Xloggc:/export/Logs/xfg-frame-archetype-lite-boot/gc-xfg-frame-archetype-lite-boot.log -XX:+PrintGCDetails -XX:+PrintGCDateStamps</java_jvm>
                <profileActive>dev</profileActive>
            </properties>
        </profile>
        <profile>
            <id>test</id>
            <properties>
                <java_jvm>-Xms1G -Xmx1G -server  -XX:MaxPermSize=256M -Xss256K -Dspring.profiles.active=test -XX:+DisableExplicitGC -XX:+UseG1GC  -XX:LargePageSizeInBytes=128m -XX:+UseFastAccessorMethods -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/export/Logs/xfg-frame-archetype-lite-boot -Xloggc:/export/Logs/xfg-frame-archetype-lite-boot/gc-xfg-frame-archetype-lite-boot.log -XX:+PrintGCDetails -XX:+PrintGCDateStamps</java_jvm>
                <profileActive>test</profileActive>
            </properties>
        </profile>
        <profile>
            <id>prod</id>
            <properties>
                <java_jvm>-Xms6G -Xmx6G -server  -XX:MaxPermSize=256M -Xss256K -Dspring.profiles.active=release -XX:+DisableExplicitGC -XX:+UseG1GC  -XX:LargePageSizeInBytes=128m -XX:+UseFastAccessorMethods -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/export/Logs/fq-mall-activity-app -Xloggc:/export/Logs/xfg-frame-archetype-lite-boot/gc-xfg-frame-archetype-lite-boot.log -XX:+PrintGCDetails -XX:+PrintGCDateStamps</java_jvm>
                <profileActive>prod</profileActive>
            </properties>
        </profile>
    </profiles>
```


## maven插件

[Maven-插件使用与配置什么是插件 插件可以理解为一组任务的集合，用户可以通过命令行直接运行指定插件的任务，也可以将插 - 掘金](https://juejin.cn/post/7055598592999817253)


maven插件本身是一个maven项目，可以作为依赖引入，他之所是插件是因为他实现了maven官方提供的api（maven-plugin-api）。

官方：maven-${prefix}-plugin
第三方：${prefix}-maven-plugin

官方可用插件：[Available Plugins – Maven](https://maven.apache.org/plugins/index.html)
非官方可用插件：[MojoHaus · GitHub](https://github.com/mojohaus)


*idea启动项目会指定classpasth路径，向其中添加需要加载所有class文件（多模块下则是各个目录下的target）,但是maven打包并不会这样，他是从本地仓库中寻找jar包，这一点非常重要，当你idea运行成功，但是jar运行失败时，一定要将每个模块重新install.!!!*


### spring-boot-maven-plugin
[Maven Plugin :: Spring Boot](https://docs.spring.io/spring-boot/maven-plugin/index.html)

中文文档[打包可执行存档文件 (Packaging Executable Archives) | Spring Boot3.4.0中文文档|Spring官方文档|SpringBoot 教程|Spring中文网](https://www.spring-doc.cn/spring-boot/3.4.0/maven-plugin_packaging.html)


[Spring Boot 打包成的 jar 和普通jar有什么区别\_spring boot 打成的 jar 和普通的 jar 有什么区别-CSDN博客](https://blog.csdn.net/weixin_45760137/article/details/118725697)

jar包分为可执行jar包和不可执行jar包。
可执行jar:  
- 将依赖引入jar包当中
- META-INF/MANIFEST.MF文件中存在启动类等配置
- 使用命令`java -jar` 执行

不可执行jar:
- 内部只有项目的class文件
-  META-INF/MANIFEST.MF文件中不存在启动类等配置
- 不能使用`java -jar` 执行。


java -jar执行逻辑： JVM会根据MANIFEST.MF文件中的Main-Class属性找到程序的入口点（即主类），然后开始执行该类的main方法。
（*这也就是说，实际上依赖是否存在并不是java- jar的启动的条件，或者说，它只有在当jvm开始加载class文件时，寻找对应的import不存在时才会暴露出来。*）



#### spring-boot: repackage

attach： install时，maven仓库中时可执行jar包还是不可执行jar包。 默认true
classifier:  instal时将可执行jar包取别名存入，同一位置（普通jar包依旧存入maven仓库）

attach的意义在于你要归档什么jar包的类型，而classifier的意义时你要引入什么类型的jar，因为无论如何maven仓库只能引入一个版本的jar，或者说插件无法更改maven的引入机制，它只能修改或增加文件。


```
<project>
	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
				<executions>
					<execution>
						<id>repackage</id>
						<goals>
							<goal>repackage</goal>
						</goals>
						<configuration>
							<classifier>exec</classifier>
						</configuration>
					</execution>
				</executions>
			</plugin>
		</plugins>
	</build>
</project>
```

这样子配置之后install在仓库中两者都存在
![](img/Pasted%20image%2020250405180246.png)

但是注意，版本号没变，maven依旧只能引入mcp-server-stdio-1.0-SNAPSHOT，所以classifier在不改变原有普通jar机制的情况下，重新打包并归档可执行jar包的一种机制。

这种配置并没有抢占普通的jar包的存储，而是作为附加一同添加，只是改了名字。而attach则是与普通的jar包争抢存档的机会。

