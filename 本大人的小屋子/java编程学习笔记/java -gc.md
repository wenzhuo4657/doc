
*GC（Garbage Collection，垃圾回收）主要针对的是程序运行时动态分配的内存，这部分内存通常是在堆（Heap）上分配的。因此GC回收器主要负责回收堆区中不再使用的对象或数据结构所占用的内存。
栈（Stack）上的内存通常是由程序自动管理的，当一个函数或者方法调用结束之后，其在栈上的局部变量就会自动释放，不需要GC来处理。*


# 引用类型区分

参考：[Java 四种引用类型（强引用、软引用、弱引用、虚引用） - 低吟不作语 - 博客园](https://www.cnblogs.com/Yee-Q/p/17833933.html)

ps：首先要区分一个概念，就是强引用等区分并非是使用对象的方式，而是垃圾回收机器gc对对象回收的不同机制，这意味着重点在于**gc是如何回收对象、如何定义这些类型**。


**引用类型的区分实际上是对多次gc回收的一种增强，希望jvm在不能回收到足够内存时可以舍弃掉一部分引用对象，而这种操作权限将由接口的形式交由程序员处理。**



## 强引用：
`Object obj=new Object();`

强引用即正常使用中的引用，这里就可以很明确的看出“引用”二字的含义。
对于java对象来说存在引用类型和运行类型，运行类型就是new Object(),引用类型则是变量定义类型，又或者是解析运行类型的方式。因为new Object()实际上是在java堆区创建数据，也就是gc回收器回收的内存，而对于栈区，则是《变量名-变量值》，变量值可分为基本数据和指向堆区的地址。

生命周期： 当一个对象被强引用变量引用时，除非超过了引用的作用域或者显示地将相应强引用赋值为 null，否则是不可能被垃圾回收器回收的


## 软引用：
```
import java.lang.ref.SoftReference
        Object o1 = new Object();
        SoftReference<Object> s1 = new SoftReference<Object>(o1);
```

生命周期：软引用用来描述一些有用但非必须的对象，此类对象只有在进行一次垃圾收集仍然没有足够内存时，才会在第二次垃圾收集时被回收

## 弱引用：
```
import java.lang.ref.WeakReference
		 Object o1 = new Object();
        WeakReference<Object> w1 = new WeakReference<Object>(o1);
```

生命周期：被弱引用关联的对象只能生存到下一次垃圾收集发生为止，当垃圾收集器开始工作，无论当前内存是否足够，都会回收掉只被弱引用关联的对象


## 虚引用：

虚引用，顾名思义，就是形同虚设，与其他几种引用都不太一样，一个对象是否有虚引用的存在，完全不会对其生存时间构成影响，也无法通过虚引用来取得一个对象实例。如果一个对象仅持有虚引用，那么它就和没有任何引用一样，在任何时候都可能被垃圾回收器回收。虚引用必须和引用队列（RefenenceQueue）联合使用。虚引用的主要作用是跟踪对象垃圾回收的状态，仅仅是提供了一种确保对象被 finalize 以后，收到一个系统通知或者后续添加进一步的处理。





# 垃圾回收机制(当前gc的回收机制)
[深入理解 JVM 的垃圾回收机制 | 二哥的Java进阶之路](https://javabetter.cn/jvm/gc.html#%E5%BC%95%E7%94%A8%E8%AE%A1%E6%95%B0%E7%AE%97%E6%B3%95)

## 引用计数

问题：无法解决嵌套引用
当对象内部存在引用，也就是成员变量的引用时，同样必须手动赋值为null,但是这gc的初衷不符，我们期望程序可以自主意识到成员变量的引用也应该被减掉。


```
public class ReferenceCountingGC {

    public Object instance;  // 对象属性，用于存储对另一个 ReferenceCountingGC 对象的引用

    public ReferenceCountingGC(String name) {
        // 构造方法
    }

    public static void testGC() {
        // 创建两个 ReferenceCountingGC 对象
        ReferenceCountingGC a = new ReferenceCountingGC("沉默王二");
        ReferenceCountingGC b = new ReferenceCountingGC("沉默王三");

        // 使 a 和 b 相互引用
        a.instance = b;
        b.instance = a;

        // 将 a 和 b 设置为 null
        a = null;
        b = null;

        // 这个位置是垃圾回收的触发点
    }
}
```

## 可达性分析算法（分代gc）



> [!NOTE] Title
> 具体到分两代的分代式GC来说，如果第0代叫做young gen，第1代叫做old gen，那么如果有minor GC / young GC只收集young gen里的垃圾，则young gen属于“收集部分”，而old gen属于“非收集部分”，那么从old gen指向young gen的引用就必须作为minor GC / young GC的GC roots的一部分


某大佬的回复，这里理解为gc roots 是根据新生代来判断老年代是否需要回收，这里需要理解几个概念。
新生代：新创建的对象，在jvm内存区域中由Eden区和Survivor区。**而新生代也作为gc频繁的区域**
老年代：存放长期存活的对象，垃圾回收较少但耗时较长。
gc roots的对象：当前系统当前时间活跃的对象

Minor GC：
只清理新生代（Eden区和Survivor区），速度快，频率高。

gcroots判断时，如果遇到老年代直接将其判断为存活，不会深层次遍历


Major GC / Full GC：
清理整个堆内存（新生代和老年代），速度慢，频率低。

gcroots判断时，难以理解的点在于老年代的引用如果无法被追溯该怎么办？
新生代升级老年代本身就要经历多次gc,重点该过程中的引用是否消失，该引用同样也是作为老年代引用存在的价值，将其作为老年代本身不是为了数据安全问题，而是认为其脱离了死亡率高的新生代。
且需要理解的是，**gcroots和新生代对象并无强关联**。

gcroots对象来源：
1，栈帧方法的局部变量、方法参数（这一部分才作为死亡率较高的新生代）
2，运行时常量池中的常量（String 或 Class 类型）
3，类静态变量
4，本地方法栈中 JNI 的引用



### 生存还是死亡？：代价高昂，不建议使用

gc roots引用链条判定失败时，并不会立即回收对象，具体来说，需要被gc roots标记两次。
第一次被标记后，会进行一次筛选，条件为：**是否有必要执行finalize()方法**

注意：对于finalize()的执行，虚拟机只承诺运行，但不承诺运行结束。因此该方法可能执行中就被中断销毁对象。

![[Pasted image 20241230173010.png]]



执行finalize方法并不意味着被拯救，重点在于finalize的内容是否能够该对象逃逸gc roots判断，例如将当前对象的引用赋值给静态变量。
```
public class finalize {  
static finalize finalize;  
  
@Override  
protected void finalize() throws Throwable {  
super.finalize();  
finalize=this;  
System.out.println("执行");  
}  
  
public static void main(String[] args) throws InterruptedException {  
cn.wenzhuo4657.IncreaseSecret.finalize finalize1 = new finalize();  
finalize1=null;  
System.gc();  
Thread.sleep(500);  //等待执行
System.out.println(cn.wenzhuo4657.IncreaseSecret.finalize.finalize);  
cn.wenzhuo4657.IncreaseSecret.finalize.finalize =null;  
System.gc();  
Thread.sleep(500);  
System.out.println(cn.wenzhuo4657.IncreaseSecret.finalize.finalize);  
}  
}
```



## 分代收集理论：基于实际情况的经验法则
- 1，弱分代假说：绝大多数对象都是朝生熄灭
- 2，强分代假说：熬过多次垃圾收集过程的对象就越难以消亡。
- 3，跨代引用假说：假设存在跨代引用，由于弱分代总是于强分代存在引用不会被回收，那么随着多次gc最终结果会使弱分代晋升为强分代。



