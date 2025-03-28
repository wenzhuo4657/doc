![[img/Pasted image 20231128142855.png]]


![[img/Pasted image 20231129123604.png]]


基本使用：

```
Properties properties =new Properties();  
properties.load(new FileInputStream("D:\学习\编译\idea-project\reflection\src\main\resources\re.properties"));  
String classfullpath=properties.get("classfullpath").toString();  
String methodName=properties.get("method").toString();  
  
  
  
Class cls=Class.forName(classfullpath);  
Object o=cls.newInstance();  
System.out.println("o运行类型；"+o.getClass());  
Method method= cls.getMethod(methodName);  
System.out.println("======================");  
method.invoke(o);  
  
Field field=cls.getField("age");  
System.out.println("age:"+field.get(o));  
Constructor constructor= cls.getConstructor();  
System.out.println("无参构造器： "+constructor);  
System.out.println("有参构造器： "+cls.getConstructor(int.class));
//且注意，在此处我们得出构造器不允许出现参数类型相同，这与其参数名称和访问级别无关，即便是为了反射的合理性也要这样设计
```


配置文件
re.properties
```
classfullpath=org.cat  
method=hi
```



### 优化：关闭访问检查
![[img/Pasted image 20231129132143.png]]
`constructor.setAccessible(true);`
意味着访问修饰符失效，

### 类加载


每个类只会加载一次，通过调用classloadcer.loadClass(）完成，且注意无论是传统调用还是反射调用都会触发。
![[img/Pasted image 20231129133746.png]]


#### 相关；
类加载，
  
JVM（Java虚拟机）在类加载的过程中是按需加载的，而不是一次性加载所有类。当程序首次引用一个类时，JVM会尝试加载该类。类加载是懒加载的一部分，这意味着只有在需要使用类时才会加载它，而不是在程序启动时一次性加载所有类。（此处需要结合静态加载和动态加载进行理解）


懒加载：
懒加载（Lazy Loading）是一种延迟加载的策略，即在需要使用某个资源时再进行加载，而不是在程序启动或初始化阶段一次性加载所有资源。这种加载方式的主要目的是优化性能、减少资源消耗，以及提高程序的响应速度。

在懒加载的情况下，资源只有在首次被请求时才会被加载到内存中。这对于大型应用程序或包含大量资源的系统来说尤为重要，因为不是所有的资源都在程序的初始阶段就被使用到。通过延迟加载，可以避免不必要的资源占用，提高系统的启动速度，并在程序执行过程中根据需要加载必要的资源。

懒加载常见的应用场景包括：

1. **类加载：** 如前面提到的Java虚拟机中的类加载机制，类在首次被引用时才会被加载，而不是一开始就加载所有类。

2. **图像加载：** 在图形应用中，如果一个页面包含大量的图像资源，可以选择在用户滚动到可见区域时再加载这些图像，而不是一开始就全部加载。

3. **数据库查询：** 在数据库中，可以延迟加载关联对象的数据，只有在访问这些关联对象时才执行实际的查询。

4. **资源加载：** 在游戏开发中，可以延迟加载游戏关卡中的资源，只有在玩家到达某个关卡时才加载该关卡所需的资源。

懒加载的优势在于节省资源、提高响应速度，并允许系统更加灵活地根据需求进行资源管理。


### 获取class的六种方式
![[img/Pasted image 20231129140712.png]]
![[img/Pasted image 20231129140858.png]]
![[img/Pasted image 20231129140940.png]]


### 静态加载和动态加载


`
`Dog dog=new Dog();  `
传统创建对象即为静态加载，在编译阶段就会进行加载，


```
Class cls=Class.forName(classfullpath);  
Object o=cls.newInstance();  
```
反射调用为动态加载，在运行时进行加载，


加载方式的不同意味着，如果不存在上述类的定义，静态加载将在程序的编译阶段报错，而动态加载则会在程序执行过程中报错，如果程序没有执行到相关字段就不会报错


![[img/Pasted image 20231129160713.png]]

![[img/Pasted image 20231129160647.png]]

![[img/Pasted image 20231129162110.png]]


![[img/Pasted image 20231129162712.png]]




## 反射的含义
**基于方法区中的 Class 信息来获取类的元数据，据此操作堆区的数据**，并且可以根据需要突破访问限制。

主要类：
java.lang.class # 获取运行时类对象，也就是运行类型的类对象
java.lang.reflect.Method
java.lang.reflect.Field
java.lang.reflect.Constructor





ps:
生成代理类
JDK代理：java.lang.reflect.Proxy（实现代理对象指定接口，并继承Proxy）
CGLIB代理：net.sf.cglib.proxy.* （生成子类）

首先要明确编写的任何代码在编译阶段、类加载阶段都不会执行，对于动态代理代理生成需要执行程序代理，它们确实会生成字节码文件，这是必不可少的，因为读取堆区数据需要Class对象，但是无论是通过何种方式生成代理，其字节码文件都是在运行阶段产生并保存在JVM内存模型的方法区当中。




## getDeclaredField(name)和getSuperclass()

在java当中，继承机制会使子类中存在父类字段，但反射中getDeclardField()方法只能查找当前类中的字段，不包括父类字段。


而getSuperclass()方法则会返回当前类的超类（超类：指当前类的直接父类，此外object是所有类的**最终**超类）

