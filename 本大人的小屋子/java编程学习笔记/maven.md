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
