
## 垃圾收集有哪些算法

从对象消亡的角度上看，有引用计数法和可达性分析两种算法。
而从整体的内存区域清理上，则是基于分代假说所建立的集中算法。
分代假说认为
1，young gen:绝大多数的对象都是朝生夕灭
2，old gen:熬过多次垃圾收集的对象，难以消亡
3,跨代引用是少数，或者说即使有跨代引用，在多次垃圾回收之后，他们大多一起变成old gen.



标记-清除：先标记，然后清除，
标记-复制：内存分区，现在做法为分成8:1:1的三份空间，一般用于新生代垃圾收集，即它认为该区域大部分对象都会消亡，只有小部分对象会存货，将其复制到1份空间中。此外，如果1份空间，不足，则将会启动担保区，即老年代区域。
标记-整理：该算法有点像标记清除，只是该算法会整理内存区域，减少内存碎片。该算法针对老年代，由于老年区域对象大多存活，所以整理移动的对象只是少部分。


需要注意的是，这些算法对于现代垃圾收集器仍是底层理论，但是分代假说已经被弱化了，变成使用最小内存单元region。

## gc方式分类，partial GC和full GC等
partial GC:部分收集
- 新生代收集（Minor GC/ Young GC）
- 老年代收集（Major GC/Old GC）
- 混合收集（Mixed GC）
full GC :收集整个java堆和方法区




## 虚拟机类加载机制

### 什么是类加载？何时类加载？类加载流程？

Java虚拟机把描述类的class文件加载到内存并解析成可以被使用的Java类型的过程称为虚拟机的类加载机制。

类加载的机制在《Java虚拟机规范》当中并没有强制规范，也就是说由虚拟机自行决定，只是规定了六种情况必须立即对类进行初始化（这之前的阶段也会按照顺序进行）。

（1）遇到new 、getstatic、putstatic、invokestatic这四条字节码指令时(具体到java代码中则是指，new对象、调用静态方法、静态字段的操作等)
（2）反射调用
（3）初始化子类时，如果发现其父类没有被初始化时，则先出发父类的初始化。
（4）虚拟机启动的主类。
等等


### 类与类加载器
类加载器：实现类加载阶段中“通过一个类的全限定类名来获取描述该类的二进制流”这个动作的代码被称为“类加载器”。

每一个类加载器，都拥有一个**独立的类名称空间**，而java语言中判断类型的instanceof底层是通过类加载器的链路查找判断，从实例检查来看，会比较全限定类型对应class对象，否则无法解释为何是返回false.

```
ClassLoader myLoader=new ClassLoader() {  
@Override  
public Class<?> loadClass(String name) throws ClassNotFoundException {  
try{  
String fileName = name.substring(name.lastIndexOf(".") + 1) + ".class";  
InputStream is = getClass().getResourceAsStream(fileName);  
if (is==null){  
return super.loadClass(name);  
}  
byte[] b = new byte[is.available()];  
is.read(b);  
return defineClass(name,b,0,b.length);  
  
} catch (IOException e) {  
throw new RuntimeException(e);  
}  
}  
};  
Class<?> aClass = myLoader.loadClass("cn.wenzhuo4657.IncreaseSecret.Threadpool");  
Object o = aClass.newInstance();  
System.out.println(o instanceof cn.wenzhuo4657.IncreaseSecret.Threadpool);
```


在该例子中使用自定义类加载器实例化目标对象，并通过全限定类名进行判断。
```
false

进程已结束,退出代码0
```

这里实际上提醒我们，在虚拟机当中，唯一标识类是存储在方法区的类型信息，也就是class对象，而全限定类名作为通过类加载器查找的方式。



### 双亲委派

使用组合的关系来复用父加载器，在代码中则表现为，如果父类可以处理，就交给父类处理。
```
ClassLoader myLoader=new ClassLoader() {  
@Override  
public Class<?> loadClass(String name) throws ClassNotFoundException {  
try{  
Class<?> loadedClass = findLoadedClass(name);  
if (loadedClass==null){  
if (getParent()!=null){  
loadedClass=getParent().loadClass(name);  
}  
if (loadedClass==null){  
loadedClass=findClass(name);  
}  
if (loadedClass!=null){  
return loadedClass;  
}  
}  
  
String fileName = name.substring(name.lastIndexOf(".") + 1) + ".class";  
InputStream is = getClass().getResourceAsStream(fileName);  
if (is==null){  
return super.loadClass(name);  
}  
byte[] b = new byte[is.available()];  
is.read(b);  
return defineClass(name,b,0,b.length);  
  
} catch (IOException e) {  
throw new RuntimeException(e);  
}  
}  
};  
Class<?> aClass = myLoader.loadClass("cn.wenzhuo4657.IncreaseSecret.Threadpool");  
Object o = aClass.newInstance();  
System.out.println(o instanceof cn.wenzhuo4657.IncreaseSecret.Threadpool);
```

此时返回
```
已连接到目标 VM, 地址: ''127.0.0.1:60259'，传输: '套接字''
true
与目标 VM 断开连接, 地址为: ''127.0.0.1:60259'，传输: '套接字''

进程已结束,退出代码0
```

## tomcat如何打破双亲委派，隔离不同web应用

从jvm加载的机制来看，是通过重写loadClass方法，无论如何，他需要保证引用到相同jar包时，可以通过共同的父类加载器共享，但是对于类中servlet则必须进行隔离，这一点对于spring来说，已经通过@HandlesTypes解决，他通过标记感兴趣类型，一次性在onstarup方法中实例化，只需要保证在那个方法中解决即可.


在Servlet容器（如Tomcat、Jetty等）的上下文中，**Web应用的主要职责是提供Servlet**，而Servlet容器则负责管理这些Servlet的生命周期、请求处理、线程池、安全性等。这句话的核心意思是：**Web应用的角色是定义和提供Servlet，而Servlet容器则是运行和管理这些Servlet的环境**。


在Web应用容器（如Tomcat）中，每个Web应用通常有一个独立的类加载器（`WebAppClassLoader`），它负责加载该应用的类和资源。这种设计确保了不同Web应用之间的类隔离，避免了类冲突。

- **共享类**：Web应用容器通常会有一个共享的类加载器（如`CommonClassLoader`），用于加载容器本身和所有Web应用共享的类（如Servlet API、Spring框架等）。
    
- **隔离类**：每个Web应用的类加载器只会加载该应用独有的类和资源，确保不同应用之间的类不会互相干扰。
    

 Servlet容器中的类加载

在Servlet容器中，Servlet类的加载是由Web应用的类加载器负责的。Servlet容器会确保每个Web应用的Servlet类是由其独立的类加载器加载的，从而实现类隔离。

- **Servlet类的加载**：当Servlet容器启动时，它会通过Web应用的类加载器加载Servlet类。由于每个Web应用有自己的类加载器，因此不同Web应用中的同名Servlet类不会冲突。
    
- **共享库**：Servlet容器提供的共享库（如Servlet API）通常由容器的共享类加载器加载，所有Web应用都可以使用这些共享库。


tomcat使用了线程上下文类加载器来破坏双亲委派机制。线程上下文类加载器的原理是将一个类加载器保存在线程的私有数据中，与线程绑定，再需要的时候取出来使用
# Java对象创建概念，一定在堆中分配？内存分配的概念？

正常情况下，即我们所熟知的jvm分配逻辑，对象一定会在堆中分析，但是在某些特殊情况下，例如该对象只会在某个方法中使用，jvm的“逃逸分析”等优化技术会将对象分配到栈上。


# 内存
## jvm内存分配概念


以g1为例子，它使用分代收集的理论，在堆中建立了eden、Survivor和老年代区域，其中如果是基本变量直接分配的栈中，而对象则分配到堆中的eden，然后根据垃圾回收的次数来逐渐转移对象的区域。

## 内存溢出和内存泄漏

内存溢出：是指程序在申请内存时，没有足够的内存空间供其使用，出现out of memory。
内存溢出经典场景： 栈溢出、堆区内存不足，其中jdk1.8版本之后方法区使用元空间自动调整大小，导致不会发生内存溢出。
内存泄漏：是指程序在申请内存后，无法释放已申请的内存空间。



## 类加载器

### 类加载器

JVM 中内置了三个重要的 ClassLoader
- BootstrapClassLoader(启动类加载器)： 最顶层的加载类，由 C++实现，通常表示为 null，并且没有父级，主要用来加载 JDK 内部的核心类库（ %JAVA_HOME%/lib目录下的 rt.jar、resources.jar、charsets.jar等 jar 包和类）以及被 -Xbootclasspath参数指定的路径下的所有类
- ExtensionClassLoader(扩展类加载器)：主要负责加载 %JRE_HOME%/lib/ext 目录下的 jar 包和类以及被 java.ext.dirs 系统变量所指定的路径下的所有类。
- AppClassLoader(应用程序类加载器)：面向我们用户的加载器，负责加载当前应用 classpath 下的所有 jar 包和类。



springboot的类加载器
- Bootstrap ClassLoader：加载 JVM 核心类库（如 rt.jar）。

- Extension ClassLoader：加载 JRE 扩展目录（lib/ext）中的类。

- Application ClassLoader：加载应用的类路径（classpath）中的类。

- LaunchedURLClassLoader：Spring Boot 自定义的类加载器，用于加载可执行 JAR 中的类和依赖

springboot的jar结构
```
example.jar
├── META-INF
├── BOOT-INF
│   ├── classes       # 应用的类文件
│   └── lib           # 依赖的 JAR 文件
└── org.springframework.boot.loader
    └── JarLauncher   # Spring Boot 启动器
```


此处需要引申出的参数是classpath,该参数指定了当前程序所有需要加载的class文件，并非仅仅是指程序的class文件。
