
## java.lang.System
[blog.51cto.com/u\_16099326/6813631](https://blog.51cto.com/u_16099326/6813631)


### arraycopy:数组拷贝

浅拷贝，只会复制引用。
```
public static native void arraycopy(Object src, int srcPos,  
Object dest, int destPos,  
int length);
```

对于Arrays.copyOf和Array.copyOfRange方法来说，该方法都是底层真正实现拷贝的方法，Arrrays只是做了数组范围等安全性检查。


###   currentTimeMillis：获取时间戳

^b9c485

时间戳：从 1970年1月1日 00:00:00 UTC（Unix 纪元） 到当前时间的毫秒数

```
long l = System.currentTimeMillis();  //获取时间戳
System.out.println(l);  
Date date=new Date(l);//Date内部也有构造器直接调用System.currentTimeMillis获取当前时间
Date date = new Date(currentTimeMillis);
SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
String formattedDate = sdf.format(date);//获取日期字符串
```

也就是说，对于程序来说，日期通常为时间戳或者是格式化的日期字符串表示。





### 读取系统属性和环境变量

- System.getProperty
- System.getenv




## 钩子函数
[blog.51cto.com/springlearn/5508492](https://blog.51cto.com/springlearn/5508492)


``` 
rocketmq源码当中的钩子函数，需要注意的是，
Runtime.getRuntime().addShutdownHook(new ShutdownHookThread(log, (Callable<Void>) () -> {  
controller.shutdown();  
return null;  
}));
```


addShutdownHook方法要求传入的Thread类型，也就是说它仅仅关注于start方法的启动。


