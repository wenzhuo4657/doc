	所谓服务发现，本质上就是配置+读取配置的方式

# java服务发现

**根据给定接口从配置文件中获取实现类。**
1，配置文件的文件名是接口的全限定类名

参考： 
[Java SPI 与 Dubbo SPI - 废物大师兄 - 博客园](https://www.cnblogs.com/cjsblog/p/14346766.html)
[Java SPI与SpringBoot 自动配置 - 雷雷提 - 博客园](https://www.cnblogs.com/Alei777/p/16638430.html)

重要类java.util.ServiceLoader和java.util.Iterator
![[Pasted image 20241111185629.png]]

配置文件：resources目录下的META-INF/services/接口的全限定类名，
读取方式：通过迭代器进行迭代加载。
```
ServiceLoader<Car> load = ServiceLoader.load(Car.class);  
  
// 获取迭代器遍历  
Iterator<Car> iterator = load.iterator();  
while (iterator.hasNext()){  
Car registry = iterator.next();  
registry.run();  
}  
  
// forEach(直接调用迭代器方法)  
load.forEach(x->x.run());

```


注意：META-INF/services是二级目录，和META-INF.services进行区分


java SPI 不足之处：

- 不能按需加载。Java SPI在加载扩展点的时候，会一次性加载所有可用的扩展点，很多是不需要的，会浪费系统资源
- 获取某个实现类的方式不够灵活，只能通过 Iterator 形式获取，不能根据某个参数来获取对应的实现类
- 不支持AOP与IOC
- 如果扩展点加载失败，会导致调用方报错，导致追踪问题很困难

## 实例分析

### jdbc
JDK 通过 JDBC（Java Database Connectivity） 为关系型数据库提供了标准化的支持：
标准接口（需要外部实现）：
- java.sql.Driver：驱动程序的接口。
- java.sql.Connection：数据库连接的接口。
- java.sql.Statement、java.sql.PreparedStatement：执行 SQL 语句的接口。
- java.sql.ResultSet：查询结果的接口。
SPI 机制：
通过 java.util.ServiceLoader 动态加载驱动程序。
管理驱动：
- DriverManager 类（（jdk实现）：负责管理驱动程序、选择驱动程序以及创建数据库连接。
- DataSource类（需要外部实现）：

**总的来说，jdk通过接口定义了标准，而spi则提供了一种通过配置文件自动装配机制。**


以下均为mysql-connector-java-5.1.47.jar、mybatis-3.5.6.jar配置，尝试分析源码

配置文件名java.sql.Driver（同时作为接口）。
![[Pasted image 20250108131403.png]]


加载：
在DriverManager存在关键方法,并且使用了synchronized

```
public static synchronized void registerDriver(java.sql.Driver driver)  
throws SQLException {  
  
registerDriver(driver, null);  
}
```

以及关键类
```
class DriverInfo {  
  
final Driver driver;  
DriverAction da;  
DriverInfo(Driver driver, DriverAction action) {  
this.driver = driver;  
da = action;  
}  
  
@Override  
public boolean equals(Object other) {  
return (other instanceof DriverInfo)  
&& this.driver == ((DriverInfo) other).driver;  
}  
  
@Override  
public int hashCode() {  
return driver.hashCode();  
}  
  
@Override  
public String toString() {  
return ("driver[className=" + driver + "]");  
}  
  
DriverAction action() {  
return da;  
}  
}
```


追踪registerDriver方法，可发现动态加载是由外部实现，此外应有启动器之类代码区块作为自动连接。
```UnpooledDataSource.class

private synchronized void initializeDriver() throws SQLException {  
if (!registeredDrivers.containsKey(driver)) {  
Class<?> driverType;  
try {  
if (driverClassLoader != null) {  
driverType = Class.forName(driver, true, driverClassLoader);  //获取驱动加载的classd对象
} else {  
driverType = Resources.classForName(driver);  
}  

Driver driverInstance = (Driver) driverType.getDeclaredConstructor().newInstance();  
DriverManager.registerDriver(new DriverProxy(driverInstance));  //注册驱动
registeredDrivers.put(driver, driverInstance);  
} catch (Exception e) {  
throw new SQLException("Error setting driver on UnpooledDataSource. Cause: " + e);  
}  
}  
}
```


### jdbc的自动加载
JDBC 4.0+ 的自动加载机制是基于 SPI，而最初的jdbc并没有spi机制。

而从代码角度上考虑，则是静态代码块，当jvm会自动加载这一部分的代码

例如在com.mysql.jdbc.Driver中
```
public class Driver extends NonRegisteringDriver implements java.sql.Driver {  
public Driver() throws SQLException {  
}  
  
static {  
try {  
DriverManager.registerDriver(new Driver());  //注册，
} catch (SQLException var1) {  
throw new RuntimeException("Can't register driver!");  
}  
}  
}
```
2，在DriverManager中也同样存在静态代码块，但加载的配置为系统属性：`jdbc.drivers.`
```
static {  
loadInitialDrivers();  
println("JDBC DriverManager initialized");  
}

private static void loadInitialDrivers() {  
String drivers;  
try {  
drivers = AccessController.doPrivileged(new PrivilegedAction<String>() {  
public String run() {  
return System.getProperty("jdbc.drivers");  
}  
});  
} catch (Exception ex) {  
drivers = null;  
}
.............................................
}
```
2，DriverSource
```
static {  
Enumeration<Driver> drivers = DriverManager.getDrivers();  
while (drivers.hasMoreElements()) {  
Driver driver = drivers.nextElement();  
registeredDrivers.put(driver.getClass().getName(), driver);  
}  
}
```

# Dubbo服务发现

参考
[扩展点加载 | Apache Dubbo](https://cn.dubbo.apache.org/zh-cn/docsv2.7/dev/spi/)

Dubbo重新实现了一套功能更强的SPI机制, 支持了AOP与依赖注入，并且**利用缓存提高加载实现类的性能**，同时支持实现类的灵活获取。

```
<dependency>
      <groupId>org.apache.dubbo</groupId>
      <artifactId>dubbo</artifactId>
      <version>2.7.8</version>
</dependency>

核心类：org.apache.dubbo.common.extension.ExtensionLoader
```

配置：META-INF/dubbo/类的全限定类型
读取配置：键值对读取



## 重要注解

 
 @SPI注解，标注接口在dubbo服务发现中，作为默认实现
 @Activate(order = 2）在Wrapper类上添加@Activate注解，通过设置order进行排序,越小优先级越高。
 

## 扩展点加载


### 不适用包装类

```
ExtensionLoader<Car> extensionLoader = ExtensionLoader.getExtensionLoader(Car.class,false);  
Car car = extensionLoader.getExtension("honda");  
car.run();


META-INF/dubbo/org.example.Car
toyota=org.example.ToyotaCar  
honda=org.example.HondaCar
```


### 自动包装== aop

参考：
[Dubbo源码篇---Wrapper详解Dubbo源码篇---Wrapper详解 Wrapper wrapper为Dub - 掘金](https://juejin.cn/post/7046671549545316388)
[Dubbo-Adaptive实现原理 - 大魔王先生 - 博客园](https://www.cnblogs.com/wtzbk/p/16656309.html)

需要注意包装类支持多层包装本质上就是一个局部变量的引用，形如以下形式


```
@Activate(order = 2)  
public class CarWrapper1 implements Car{  
private final Car car;  
  
public CarWrapper1(Car car) {  
this.car = car;  
}  
  
@Override  
public void run() {  
System.out.print("CarWrapper1包含");  
car.run();  
}  
}

```


多个包装类时通过注解控制顺序。

### 自动装配 == ioc
加载扩展点时，扩展点实现类的成员如果为其它扩展点类型，`ExtensionLoader` 在会自动注入依赖的扩展点。`ExtensionLoader` 通过扫描扩展点实现类的所有 **setter 方法**来判定其成员。即 `ExtensionLoader` 会执行扩展点的拼装操作。

引出问题：当扩展点有多个实现时，应该注入哪一个？



### 自适应装配 和扩展点自动激活

两者在使用上都是关于注解@Adaptive，将其标注在方法上就是自动装配，将其标注在类上就是自动激活。

对应于Adaptive机制，Dubbo提供了一个注解@Adaptive，该注解可以用于接口的某个子类上，也可以用于接口方法上。
如果用在接口的子类上，则表示Adaptive机制的实现会按照该子类的方式进行自定义实现；
如果用在方法上，则表示Dubbo会为该接口自动生成一个子类，并且重写该方法，没有标注@Adaptive注解的方法将会默认抛出异常。
对于第一种Adaptive的使用方式，Dubbo里只有ExtensionFactory接口使用，
#### 自适应

`ExtensionLoader` 注入的依赖扩展点是一个 `Adaptive` 实例，直到**扩展点方法执行时**才决定调用是哪一个扩展点实现。

Dubbo 使用 **URL 对象**（包含了Key-Value）传递配置信息。

以上为官网原话，需要注意的时url对象和注入依赖的时机，这意味着具体的实现时必不可少的两点。一个是url对象需要作为方法参数进行传递，保证在需要获取依赖配置时能够及时获取；另一个时依赖注入的时机实际上决定了代码的实现，也是官网所提到的扩展点`Adaptive`实例。






自我理解，将注解的标注在方法上的所用时: 为方法生成子类也就是`Adaptive`实例，并重写对应方法，方法内部会使用标注在类上的配置信息依次查找实现，因此官网提到`直到扩展点方法执行时才决定调用是哪一个扩展点实现`,搜索文章发现相似，生成代码如下

```



public java.lang.String queryCountry(org.apache.dubbo.common.URL arg0) {         if (arg0 == null) throw new IllegalArgumentException("url == null");         org.apache.dubbo.common.URL url = arg0;         String extName = url.getParameter("person.service", "china");         if (extName == null)             throw new IllegalStateException("Failed to get extension (org.dubbo.spi.example.PersonService) name from url (" + url.toString() + ") use keys([person.service])");         org.dubbo.spi.example.PersonService extension = (org.dubbo.spi.example.PersonService) ExtensionLoader.getExtensionLoader(org.dubbo.spi.example.PersonService.class).getExtension(extName);       
return extension.queryCountry(arg0);    
}
```

注意代码最后一行`return extension.queryCountry(arg0); `,所以dubbo-spi所实现的依赖注入，实际上不如说是方法重写。


更正；url时k-v配置，而注解Adaptive是key配置，
```
@Adaptive({"PenRes"，“”})  
void run(URL url);
```

在执行这个方法前，也就是在包装类当中会一次查询是否存在响应的键值。
```
public String getParameter(String key, String defaultValue) {  
String value = getParameter(key);  
return StringUtils.isEmpty(value) ? defaultValue : value;  
}
```

然后在调用` ExtensionLoader.getExtensionLoader(org.dubbo.spi.example.PersonService.class).getExtension(extName);`加载对应的实现类，也就是说该注解定义了方法的具体执行者，而对于服务调用而言，自动装配的首要目的也正是调用对应的方法，而非获取对象。
因此对于自动装配的`直到扩展点方法执行时才决定调用是哪一个扩展点实现`是完全合理的。


### 自适应装配的自动装配

```
public void setWheelMaker(WheelMaker wheelMaker) {//自适应装配
        this.wheelMaker = wheelMaker;
    }
}
public Car makeCar(URL url) {//自动装配
        // ...
        Wheel wheel = wheelMaker.makeWheel(url);
        // ...
        return new RaceCar(wheel, ...);
    }
```

需要明确的是，自适应装配无法使用url参数，而url参数是针对于服务方法而言的，换句话说，自适应装配，提供默认的依赖服务，而url参数提供自适应的服务提供，实现扩展时加载。




#### 扩展点自动激活


如果用在接口的子类上，则表示Adaptive机制的实现会按照该子类的方式进行自定义实现；