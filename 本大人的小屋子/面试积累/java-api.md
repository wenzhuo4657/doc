# 面向对象的三大特征
封装：封装是指将对象的状态信息隐藏在对象内部，不允许外部对象直接访问对象的内部信息。
继承：继承是不同类的共有特征的一种优化措施，java是单继承机制，对于父类的方法和属性子类种都可以找到。
多态：多态是对java对象的一种解释机制，在编译阶段的限制，java对象所调用的属性和方法被限制为引用类型所拥有的，除此之外，还有重写机制，假如运行类型对父类方法有重写，则将其指向子类重写的方法实现。


# 设计模式的五大原则
- **单一职责原则**
- **开闭原则**
- **里氏替换原则**
- **依赖倒转原则**
- **迪米特原则**



# 为什么重写 equals() 时必须重写 hashCode() 方法？
因为两个相等的对象的 hashCode 值必须是相等。也就是说如果 equals 方法判断两个对象是相等的，那这两个对象的 hashCode 值也要相等。
具体来说hashcode我认为是作为一种快读判断两个对象是否相等的过滤方式，并且该方法返回的是int类型，判断较快，而equals是两个对象完全相等的判断，例如在hashmap等集合当中，都会有hash碰撞的概念，当hash值有冲突时会通过equals方法来判断两个对象是否真的相等。
# String、 StringBuffer、StringBuilder

创建String变量会优先从String常量池中，如果有则会将String内部的value数组指向，如果没有则在常量中创建再指向value数组，且注意，String的value是不可改变的，这是由于修饰符private和final，其没有封装访问的方法。


StringBuffer、StringBuilder都继承并实现了抽象类AbstractStringBuilder，创建变量不会影响到字符串常量池，但不同的是其value数组没有使用final、private修饰，且封装有例如apppend的方法可以改变其value数组的内容。
且由于其实现不同，StringBuffer线程安全，StringBuilder线程相对不安全，其效率均高于Stirng。


### StringBuffer线程安全，StringBuilder线程相对不安全？
具体到源码上则是StringBuffer的方法大多使用synchronized修饰，保证并发操作的安全性，而Stringbuilder则没有。
但是注意，这并不意味着Stringbuffer就一定能够比Stringbuilder慢，在jdk8或者之后的版本，虚拟机hotspot已经对锁升级等机制有了非常成熟的机制，甚至可以达到无锁的状态。
例如在jdk17当中的CopyOnWriteArrayList源码，其写操作已经更改使用了synchronized,而非自旋锁，这固然是基于CopyOnWriteArrayList的设计做出的优化，但同时也一定程度上说明synchronized已经非常成熟。



# 字符串拼接用“+” 还是 StringBuilder?
对于java程序来说`+`具有重载，当两个String相加，会被翻译为使用StringBuilder的append方法，两者的区别在于，重载的`+`不会复用对象，因此如果是两个字符串拼接没有区别，然而对于多个字符串特别是循环，自主创建一个StringBuilder比较好。

# finally 中的代码一定会执行吗？
不一定，假如程序提前终止则不会。
# 包装类型和基本数据类型的区别

存储区别：基本数据类型的局部变量存放于jvm虚拟机栈的局部变量表中，而成员变量作为对象的一部分共同存储与堆中，而包装类型作为对象只能存放与堆中。事实上这种区别得益于jvm的准确方内存管理，因为如果jvm如果不知道存放于内存中数据的类型也就无从分类管理。
是否支持泛型：包装类型可作为泛型，基本数据类型不可以，
超类以及各种增强方法：包装类型通过接口控制实现了各种增强方法。默认的最终超类为object,而基本数据类型仅仅拥有它内存区域当中的值。



# 包装类型的缓存机制
包装类型的缓存机制是指jvm提前为程序创建好一定范围内的包装对象便于程序调用，例如IntegerCache,在源码注释当中显示该类主要用于实现jls所规定的自动装箱，并在首次使用包装类时初始化，在实际使用中，Integer的valueof方法也同样使用到了该缓存。

而float、double两个浮点数的包装类型并没有实现缓存，个人理解为了其浮点数缓存的数量过大，性能损耗高于直接创建对象。



# 包装类型的自动装箱

包装类型的自动装箱和拆箱是指基本数据类型在被赋值给包装类型变量时自动装箱，包装类型数据在赋值给基本数据类型时自动拆箱。
在字节码文件的编译之后，自动装箱和自动拆箱被翻译为调用相应的包装类型方法，例如Integer.valueof和Integer.intvalue()的方法。
为了优化这两个方法，除了Float,Double两个浮点数的包装类型，其他包装类型都使用到了cache缓存来增强性能，减轻gc压力和频繁创建对象。



#  浮点数精度损失-使用BigDecimal
精度损失的问题从根本上来说是进制转换的问题，十进制数字无法精准的转换为二进制，在一定位数之后只能隔断。

解决这个问题的办法就是使用BigDecimal数据类型，从源码上来看，计算通过严格控制小数点位数和越界管理保证计算准确，保存数据时，则使用主要使用到两个成员变量，BigInteger intVal和int scale;

参考：[BigDecimal 详解 | JavaGuide](https://javaguide.cn/java/basis/bigdecimal.html#%E5%8A%A0%E5%87%8F%E4%B9%98%E9%99%A4)


# BigInteger任意精度的整数类型
源码中采用int[]来保存数据，不具备小数位，但作为代价，其运算效率也会十分低下。

# 为什么成员变量有默认值，而局部变量没有

1，局部变量是gc频繁的内存区域，如果没有赋值会被快速回收，而且作为gc roots的选定对象，要求其值具有意义，便于jvm定位回收。
2，对于成员变量来说，则没有这种需求，具有默认值的意义是为了避免编译时误报未初始化错误。


# 为什么静态方法不能调用非静态成员
这和jvm的运行时数据区域的加载相关，静态方法会随着类加载的时候被加载进方法区，而成员变量不会。从根本上来讲，静态成员和非静态成员不能够在同一时间加载，因此java程序禁止了这种行为，而并非时静态成员和非静态成员不能够在同一时间存在。


# 重载和重写的区别
重载只是对方法名称相同的一组方法的称谓，方法参数、实现、返回值都可以不同且没有什么约束。
而重写是子类对父类已有的方法的自行实现，jvm当中的栈帧存在一个该栈帧所属方法的引用，该引用即表明该方法究竟是属于父类方法还是子类方法，然而无论如何，该引用最终会被翻译为指向方法的直接地址，而重写的检查也只是在编译阶段，保证生成出jvm能够正确识别的规范class文件。

因此可以说重载和重写没有关联，他们只是叫法上相似，且都是编译阶段的程序规范。



# java编译和解释共存
jvm的最初版本sun classic虚拟机就是纯解释器的虚拟机，程序在运行时逐句翻译class代码为机器代码执行，这也是最初的java语言所号称的一次编译、处处运行，同时也带来了性能瓶颈，程序执行中需要首先将class代码翻译为机器代码才能交由cpu执行，而当时的编译器只能使用外挂的形式完全替代jvm的解释器，但问题只不过从运行速度慢变为启动速度慢，jvm必须完全将java代码编译为机器代码才能够启动程序。随机技术的发展，如今jvm的流行版本hotsport使用的是即时编译jit和解释器同时使用，得益于hotsport的另一项热点侦测技术，帮助jvm找到高频的代码，将其编译为机器代码执行。


# java异常

java中异常的祖先类是java.lang包的Throwalbe,有两个重要子类，Exception,程序可以处理的异常，和erro，重大错误，
其Exception又分为checked exception和unchecked exception,前者表示受检查异常，可以程序编译时检查出来，如果不去处理就不能通过编译，后者为不受检查异常，即不做任何处理也可以通过编译，但是在程序运行时通常会被暴露出来导致程序运行出错。



# I/O 流为什么要分为字节流和字符流呢?
虽然两者均是通过字节传输，但是在jvm中字节流需要经过处理的才能与java程序对接，而且比较耗时，大概是基于这一点jvm提供了字符流，而且对于文件的编码等也做了封装，利于开发者使用。



# BIO、NIO 和 AIO 的区别？
bio、nio、aio解决的问题都是一个通信过程中的阻塞问题，对于bio而言，当用户想服务端发起一个通信，要求上传或者下载某个文件，这个线程会阻塞，无论网络状况如何、用户端和服务端是否由于其他原因不能进行正常的传输，bio都会将线程阻塞在该处，此时若是有其他用户也要进行通信，bio的做法是多开线程，对于每个用户都有一个对应的线程。nio对此做出了优化，提出了就绪操作集的概念，即双方在正式传输前告知对方自己的当前的状态，对于每个用户，服务端仅需一个用一个channel通道来表示双方的数据通道，使用selector复用器来根据不同用户就绪操作集的来决定当前执行哪些任务，对此netty做了更近一步的优化，然后就是aio,nio对于通信在线程处的阻塞做出的处理是同步处理，但是通过复用器来达到非阻塞的效果，而aio则是直接将线程阻塞处变为异步的方式，使用回调函数来通知程序通信完成。


# Java中的不可变类以及for循环的赋值机制
Java 中八个基本类型的包装类和 String 类都属于不可变类，且注意这里的不可变是指包装对象真实值属性不可变，而非引用值不可变。

例如：Integer中的`private final int value;`,String当中的`private final char value[];`。

示例证明：
```
ArrayList<Integer> list = new ArrayList<>();  
list.add(new Integer(12));  
list.add(new Integer(13));  
list.add(new Integer(14));  
for (Integer i:list){//对象进行引用值赋值  
i=i+10;//改变了引用，而非属性值。  
System.out.println(i);  
}  
int [] arr=new int[2];  
arr[0]=1;  
arr[1]=2;  
for (int i:arr){//基本是数据类型进行变量副本赋值  
i+=10;//由于是副本，所以修改不影响原变量  
System.out.println(i);  
}  
System.out.println("["+arr[0]+","+arr[1]+"]");  
System.out.println(list);
```

# 引用赋值时原子性的吗？

在java中引用赋值是原子操作，这意味着只会存在新值和旧值两种状态。
# 值传递和引用传递

值传递：只传递值，（无论是基本数据类型还是对象类型，本质上都是传递其值）
引用传递：传递变量的地址，而非变量的值的。


## 为什么java中只有值传递
1，引用传递违反了java语言的初衷，即将内存管理交给jvm，程序员只需要编写业务代码即可。
2，如果存在引用传递机制，则必定需要额外的安全机制，避免影响原变量，带来额外的开销。并且最终这些内存的管理依然要交给jvm进行管理。


## 什么是变量副本

变量副本是指将变量的值传递给一个新的变量，这个新的变量就可以被成为原变量的副本，具体来说，
基本数据类型变量的副本：副本值即基本变量本身，
引用数据类型变量的副本：副本值即引用变量的引用值。

这种变量副本的区别实际上也就是值传递的区别，是由于变量类型决定的。


# fail-safe和fail-fast
fail-safe:安全失败，指即便发生意外情况，也依旧可以正常运行，即使运行结果不正确。
fail-fast:快速失败，指发生意外情况时立即感知到，并且立即停止运行抛出错误。

fail-fast实现：
在for循环时，循环体被修改
```
List<Integer> list = new ArrayList<>();  
CountDownLatch countDownLatch = new CountDownLatch(2);  
  
// 添加元素  
for (int i = 0; i < 100; i++) {  
list.add(i);  
}  
  
Thread t1 = new Thread(() -> {  
// 迭代元素 (注意：Integer 是不可变的，这里的 i++ 不会修改 list 中的值)  
for (Integer i : list) {  
i++; // 这行代码实际上没有修改list中的元素  
}  
countDownLatch.countDown();  
});  
  
Thread t2 = new Thread(() -> {  
System.out.println("删除元素1");  
list.remove(Integer.valueOf(1)); // 使用 Integer.valueOf(1) 删除指定值的对象  
countDownLatch.countDown();  
});  
  
t1.start();  
t2.start();  
countDownLatch.await();
```


报错
```
Exception in thread "Thread-0" java.util.ConcurrentModificationException
	at java.util.ArrayList$Itr.checkForComodification(ArrayList.java:911)
	at java.util.ArrayList$Itr.next(ArrayList.java:861)
	at cn.wenzhuo4657.IncreaseSecret.Threadpool.lambda$main$0(Threadpool.java:28)
	at java.lang.Thread.run(Thread.java:750)
```

调用链路显示，底层使用ArrayLis$Itrt.next方法进行赋值，但是checkForComodification失败报错`java.util.ConcurrentModificationException`,

ArrayLis$Itrt: 这表示arraylist的内部类Itr，实现遍历器接口`Iterator<E>`,

```
final void checkForComodification() {  
if (modCount != expectedModCount)  
throw new ConcurrentModificationException();  
}
```

modCount：这是AbstractList的一个属性，表示集合修改次数。
expectedModCount:这是arraylist的内部类Itr的一个属性，表示期望的修改次数，也是arraylist实现快速失败的关键属性。

对于快速失败而言，要求在发生意外情况时立即停止运行并抛出错误，而expectedModCount和modCount正是这一实现。
本质上来说，实现了快速失败的方法是ArrayLis$Itrt.next。


fail-safle：使用CopyOnWriteArrayList替代arraylist。
省去过程，直接看CopyOnWriteArrayList$COWIterator.next（）。
```
public E next() {  
if (! hasNext())  
throw new NoSuchElementException();  
return (E) snapshot[cursor++];  
}
```

这里仅仅进行了简单的索引长度判断，因而并没有实现快速失败（在访问时发现意外情况，立即终止），转而实现：即使发生了意外情况，也要保证正常访问，即使访问结果不正确。

对CopyOnWriteArrayList来说，则是在add方法当中，修改操作setArray(newElements); 即仅仅改变引用，属于原子操作。
```
public boolean add(E e) {  
final ReentrantLock lock = this.lock;  
lock.lock();  
try {  
Object[] elements = getArray();  
int len = elements.length;  
Object[] newElements = Arrays.copyOf(elements, len + 1);  
newElements[len] = e;  
setArray(newElements);  
return true;  
} finally {  
lock.unlock();  
}  
}
```


转而查看访问者，对于ArrayLis$Itrt访问的是原始数据` Object[] elementData = ArrayList.this.elementData; ` 而 
CopyOnWriteArrayList$COWIterator.next访问的是快照snapshot[cursor++]，查看构造器可知，其虽然也是赋值，但是由于cow模式，不再原数组上修改，而是转而全量复制到一个新数组修改，因此，该快照并不会改变。
```
private COWIterator(Object[] elements, int initialCursor) {  
cursor = initialCursor;  
snapshot = elements;  
}
```

CopyOnWriteArrayList的fail-safe是基于cow模式的所实现的，它保证读取数组的安全性，但并不保证正确性，这也正符合安全失败的定义。

同时需要注意的该快照不会修改，但会随着遍历器被回收而导致没有强引用，最终会被gc回收。



# final和static关键字
final
类：表示类不能被继承
变量：表示变量值不能被改变
方法：表示该方法不允许被重写

static
变量：加载class文件时，会被加载到方法区，属于所有实例共享的部分
**静态方法**：类的 `static` 方法可以通过类名直接调用，不需要创建类的实例。静态方法不能访问非静态成员
**静态块**：用于在类加载时初始化静态变量

# 静态变量和静态常量
静态变量：static修饰
静态常量： static final修饰


静态常量：jdk8之前随着类加载放在方法区中，jdk8之后静态变量和常量池被迁移到堆中。
静态常量: 类加载阶段中的链接时，被加载进jvm常量池中


# jvm三大变量池
常量池：常量池位于字节码文件中
运行时常量池：
字符串常量池


#  Synchronized 和 Reentrantlock区别？

基本：[[整理] Java 锁的分类 - 哆啦梦乐园 - 博客园 (cnblogs.com)](https://www.cnblogs.com/feily/articles/14095877.html)

首先先说两者共同点，这两个锁都是可重入锁、非公平锁、排他锁、悲观锁。


两者最大的区别在于Synchronized是java原生语法，由jvm实现，而ReentrantLock它是JDK 1.5之后提供的API层面的互斥锁，需要lock()和unlock()方法配合try/finally语句块来完成


其次相比Synchronized，ReentrantLock类提供了一些高级功能，主要有以下3项：


        1.等待可中断，持有锁的线程长期不释放的时候，正在等待的线程可以选择放弃等待，这相当于Synchronized来说可以避免出现死锁的情况。通过lock.lockInterruptibly()来实现这个机制。

        2.公平锁，多个线程等待同一个锁时，必须按照申请锁的时间顺序获得锁，Synchronized锁非公平锁，ReentrantLock默认的构造函数是创建的非公平锁，可以通过参数true设为公平锁，但公平锁表现的性能不是很好。
         3.锁绑定多个条件，一个ReentrantLock对象可以同时绑定对个对象。ReenTrantLock提供了一个Condition（条件）类，用来实现分组唤醒需要唤醒的线程们，而不是像synchronized要么随机唤醒一个线程要么唤醒全部线程。


# ArrayList、LinkedList、LinkedHashMap

ArrrayList是列表结构继承list接口，内部使用数组维护，当容量不够时会进行扩容然后拷贝到新数组当中。

linkedList实现Deque接口，使用内部类Node存储队列，不支持并发
而LinkedHashMap继承了hashmap接口，存储数据用的继承了HashMap.Node的Entry，内部使用存储的首节点和末节点的引用，也正是基于这个结构实现了无序存储，有序遍历。

# Comparable 和 Comparator 的区别

传参区别：
Comparable：int compare(T o1, T o2);需要两个参数比较
Comparable：public int compareTo(T o);一个参数比较
应用场景：
Comparable：在源码注释中显示“一个比较函数，它对某些对象集合施加 总排序 。可以将比较器传递给 sort 方法”
Comparable：此接口对实现它的每个类的对象施加总 order。此排序称为类的自然 排序，类的 compareTo 方法称为其 自然比较方法。

比如对于TreeMap的构造函数会允许传入，比较函数，
```
public TreeMap(Comparator<? super K> comparator) {  
this.comparator = comparator;  
}
/**  
*  比较器用于保持此树形图中的顺序，如果它使用其键的自然顺序，则为 null
* @serial  
*/  
private final Comparator<? super K> comparator;
```

在Arrays.sort、以及各种集合的sort方法当中，大多数都会追踪到Arrays的legacyMergeSort方法当中
```
/** 将在将来的发行版中删除。. */  ps:jdk8、jdk17都没有删除，可能是忘记了？
private static <T> void legacyMergeSort(T[] a, Comparator<? super T> c) {  
T[] aux = a.clone();  
if (c==null)  
mergeSort(aux, a, 0, a.length, 0);  
else  
mergeSort(aux, a, 0, a.length, 0, c);  
}
```

而关于mergeSort的两个重载，在实现中显示，如果有比较器 Comparator存在则使用，`c.compare(src[mid-1], src[mid])`,没有则根据集合/数组元素自身的自然排序方法进行排序，`((Comparable)src[mid-1]).compareTo(src[mid])`,
也就是说，Comparable更多的是用于集合元素顺序的定义，而Comparable则是元素本身顺序定义的自然顺序，具体来说从比较本身的作用可能区别不是很大，更多的是一种规范作用的区别。


# 什么是虚拟线程

虚拟线程对应平台线程（操作系统的线程）。具体来说，即平台线程可以作为容器管理多个虚拟线程。
这样做会使虚拟线程相比于平台线程十分轻量，适合于一些不复杂的定时任务等，但不适合于计算密集的任务，而且集成方面和某些第三库的不是很好。


# 多线程

## 什么是线程和进程?线程与进程的关系,区别及优缺点？
1,线程是系统执行的最小单位，进程是系统资源分配的最小单位。线程必须依赖于进程执行，二者之间的区别在于单纯的进程并不能执行任务，需要在进程内部开启线程才能够执行任务/

线程：
优点：资源开销小、通信高效、充分利用多核cpu
缺点：线程安全问题（例如对共享资源的竞争）、稳定性差（线程的崩溃可能会导致进程的崩溃，进程内部的其他线程自然也就崩溃了）
进程：
优点：独立性高（进程之间互不影响，进而解决线程的安全性、稳定性问题）
缺点：资源开销大、通信困难

## 线程和进程的区别
进程中包含了多个线程，对于进程而言，他代表了计算机上某个与外部进行通信的实体，即应用进程，在jvm中，当我们启动了一个main函数，实际上就启动了一个jvm的进程，而main函数所在的线程被称为主进程。
任何一个线程都不可以脱离进程单独存在，对于jvm启动的线程而言，每个线程都必须包含程序技术器（存储将要执行的指令），虚拟机栈（描述方法执行的内存模型，局部变量、操作数、动态链接等），本地方法栈（本地方法，javaffi），


对于进程而言一定是相互独立，互不影响的，但是线程就不同了，不同线程之间可以产生交互，这同样也就是线程所属的虚拟机栈、程序计数器、本地方法栈是私有的原因，


## 方法区的移动
jdk8以前方法区是单独的，在启动时可以设置，但是在jdk8之后被移入了元空间之中，其大小可根据实际情况改变，更加灵活，



*局部变量存储在（虚拟机栈）栈中，而成员变量存储在堆中，静态变量存储在方法区中*




## 并发和并行的区别
并发是两个及以上的任务在同一时间段执行，总的来说是同一时间点正在处理的任务数量
而并行满足并发的前提下，强调，任务是在同一时间点开始的。



## 同步和异步的区别
同步：会阻塞等待程序调用完成
异步：直接跳过，通过回调函数通知程序执行完成，


## 多线程可能带来的问题
多线程解决的是并发编程问题，即单线程难以处理大量请求，但不存在某个程序会一直保持高并发，此外还有可能发生内存泄漏、死锁、线程不安全等问题。


## 什么是上下文切换？
线程在执行过程中有自己的运行条件和状态，简称上下文，而上下文切换即指占用cpu的线程从占用cpu的状态中退出，并切换至其他上下文。
常见情况：
1，主动让出cpu，例如主动调用sleep()，wait()
2，时间片用完，操作系统强行切换
3，调用了阻塞类型的代码，例如io通信等。
4，线程被杀死或者执行结束


## synchronized的关键字的使用
方法同步
```
public synchronized void method() { // 代码 }
```
代码块同步
```
public void method() { Object lock = new Object(); synchronized (lock) { // 需要同步的代码 } }
```

代码块同步需要指定一个锁，来保证其同步，只有争抢到这个锁才可以执行
## 什么是线程死锁?如何避免死锁?
不同线程在执行同步代码块或者同步方法时，对同一资源的争抢导致的僵持状态，例如逻，线程a占用了资源a,线程b占用了资源b，但是线程a需要占用资源b然后再释放资源a，而线程b正好相反，

避免死锁：
1，避免在占用某资源的情况下去请求另一资源，或者说一次性请求线程需要的所有资源，执行完成后释放
2，占用某些资源区请求其他资源时，如果时间过长，主动释放当前所拥有的资源来退步，使其他线程优先执行
3，定义一定的顺序来请求资源，从请求占用逻辑上避免。




## 既然Thread执行start底层又会调用run，为什么不直接调用run方法呢？
start方法的内部会对线程的创建做支持，且其内部有调用start0方法，其内部才调用了run方法，直接调用run方法，仅仅是一个普通的同步方法，不会新建线程。


## volatile 关键字
在java中可保证变量的可见性，但不保证原子性，且将变量声明为volatitle会告知jvm这个变量是共享、不稳定的，每次使用都要到主存中读取。


## synchronized关键字
synchronized翻译为中文是同步的意思，用于解决多线程对资源访问的同步性。




修饰方法，锁是当前实例对象，
修饰静态方法，锁是当前类的class对象
修饰代码块，可传入类或者对象表示锁，传入类是锁是对应的class对象。



## Threadlocal
线程级别变量，对于不同的线程都会有不同的绑定的值，也就是可以存储线程的私有变量，其实际的值存储在内部的Threadlocalmap中，其键是threadlocalmao中，v是实际的值，也就是说，key为弱引用，value为强引用


### Threadlocal内存泄漏？
Threadloaclmap为Threadlocal的静态内部类，其中key为弱引用，value为强引用，因此可能出现key被回收，value未被回收的情况，即出现key为null的value，
对此threadlocalmap的解决措施是，每次调用get、set、remove()等方法，都会检查并清理key为null的键值对



## JVM的线程是用户级别还是内核级别？
[Site Unreachable](https://www.zhihu.com/question/23096638/answer/29617153)

这个问题的实际上上并不准确，因为在jvm规范当中并没有规定，也就是说具体到任意虚拟机当中，可以是一对一、N对1、M对N，在javaSE常用的虚拟机Hotsport VM的较新版本当中都是默认使用一对一线程模型的。


## 如何创建线程？
创建线程一般是指man主线程中启动另一个线程，基本的思路为创建线程任务也就是实现run方法，可以通过继承Thread、实现Runnable和Callable接口，此外就是真正意义上的启动一个线程并指定线程任务，new Thread().start()方法启动线程。
而对于各种管理线程的池化技术，实际上底层创建线程也仍是基于new Thread().start(),只是做了各种封装便于管理。


## 线程的生命周期和状态

NEW: 初始状态，线程被创建出来但没有被调用 start() 。
RUNNABLE: 运行状态，线程被调用了 start()等待运行的状态。
BLOCKED：阻塞状态，需要等待锁释放。
WAITING：等待状态，表示该线程需要等待其他线程做出一些特定动作（通知或中断）。
TIME_WAITING：超时等待状态，可以在指定的时间后自行返回而不是像 WAITING 那样一直等待。
TERMINATED：终止状态，表示该线程已经运行完毕。



对于new状态，是new Thread对象被创建的默认状态，在源码注释中可以看到关键成员变量threadStatus
```
/*  
* 工具的 Java 线程状态，默认指示线程“尚未启动” 
*/  
private volatile int threadStatus;
public State getState() {  
// get current thread state  
return jdk.internal.misc.VM.toThreadState(threadStatus);  
}
```

进一步查看可发现，底层是通过按位与运算，并且new状态的代码正好可以和成员变量的默认值0相匹配来获取new状态。
```
private static final int JVMTI_THREAD_STATE_ALIVE = 0x0001;  
private static final int JVMTI_THREAD_STATE_TERMINATED = 0x0002;  
private static final int JVMTI_THREAD_STATE_RUNNABLE = 0x0004;  
private static final int JVMTI_THREAD_STATE_BLOCKED_ON_MONITOR_ENTER = 0x0400;  
private static final int JVMTI_THREAD_STATE_WAITING_INDEFINITELY = 0x0010;  
private static final int JVMTI_THREAD_STATE_WAITING_WITH_TIMEOUT = 0x0020;
public static Thread.State toThreadState(int threadStatus) {  
if ((threadStatus & JVMTI_THREAD_STATE_RUNNABLE) != 0) {  
return RUNNABLE;  
} else if ((threadStatus & JVMTI_THREAD_STATE_BLOCKED_ON_MONITOR_ENTER) != 0) {  
return BLOCKED;  
} else if ((threadStatus & JVMTI_THREAD_STATE_WAITING_INDEFINITELY) != 0) {  
return WAITING;  
} else if ((threadStatus & JVMTI_THREAD_STATE_WAITING_WITH_TIMEOUT) != 0) {  
return TIMED_WAITING;  
} else if ((threadStatus & JVMTI_THREAD_STATE_TERMINATED) != 0) {  
return TERMINATED;  
} else if ((threadStatus & JVMTI_THREAD_STATE_ALIVE) == 0) {  
return NEW;  
} else {  
return RUNNABLE;  
}  
}
```

总的来说，new状态是完成线程唯一的一个状态参数，在源码方法当中可以看到，如果threadStatus!=0就抛出错误。
总之Java的线程状态并非是操作系统的实际线程状态，但与操作系统的线程状态是有对应关系的。



##  如何理解线程安全和不安全？
线程安全和不安全是在多线程环境下对于同一份数据的访问是否能够保证其正确性和一致性的描述。

线程安全指的是在多线程环境下，对于同一份数据，不管有多少个线程同时访问，都能保证这份数据的正确性和一致性。
线程不安全则表示在多线程环境下，对于同一份数据，多个线程同时访问时可能会导致数据混乱、错误或者丢失。


## wait队列和Entry Set
![[Pasted image 20250118141451.png]]

![[Pasted image 20250118141503.png]]
# 浏览器


## 浏览器的CORS策略

浏览器为了保护用户数据的安全，实施了一项称为同源策略的安全机制。这项策略规定了脚本只能读取来自相同来源的文档或资源。当一个请求发起方和目标服务器不在同一个源（即协议、域名和端口三者都不同）时，就需要遵守CORS策略。

CORS是一种安全机制，用于确定跨源HTTP请求是否应该被允许。浏览器会在跨源请求中自动添加特定的HTTP头来请求服务器的许可。如果服务器响应中包含了正确的CORS头，则浏览器允许请求继续；否则，浏览器会阻止请求，并报告一个CORS错误。


要点
1，浏览器并不会阻止请求的发送，而是通过服务器的响应头决定是否访问服务返回的数据。
2，对于某些请求，浏览器会发送预检请求，提前判断服务器是否允许实际的请求。


# SPI

## SPI和API的区别
spi仅仅是作为一种调用实现的机制，而api则是完整的服务，比如一个服务提供方根据接口实现了他的服务，那么它对外发布的接口即是api服务，而服务使用方则会根据服务提供方的接口进行调用api服务，也就是spi调用。

## java_spi缺点
1，不能按需加载，需要遍历是所有实现类，
2，由于本质上是外部实现，所以存在无法规避的风险。
3，当有多个ServiceLoader.load加载时，可能会产生并发问题。
```
public static <S> ServiceLoader<S> load(Class<S> service) {  
ClassLoader cl = Thread.currentThread().getContextClassLoader();  
return ServiceLoader.load(service, cl);  
}
```




# 序列化和反序列化

## 什么是序列化和反序列化

序列化和反序列化的任务是完成一个系统对外的数据交换，也就是说在发送数据时将数据转换为外部期望的数据，接收时再将数据转换为系统内部期望的数据。

具体来说，则是需要协议在序列化的效率和反序列化的兼容性做权衡。
越高效的协议，通常也就意味着更小的数据体积，同时也对于反序列化提出了更大的困难。
而反序列化的兼容性、可读性越好，也就意味着数据本身的不能有歧义，这同样也为序列化带来了难度。
取极端的例子则是Protocol二进制协议和JSON文本协议。



## 协议
### jdk自带序列化

```

public class RpcRequest implements Serializable {
    private static final long serialVersionUID = 1905122041950251207L;
    。。。。。。。。。。。。。。
    
}
```

serialVersionUID的作用？
版本控制，所有该类的该字段都一致，如果在反序列化时发现不一致则会抛出错误`InvalidClassException`
并且在源码注释中要求该字段一定是 static、final 和 type long。

> [!NOTE] public interface Serializable
> 一个可序列化的类可以通过声明一个名为 "serialVersionUID" 的字段来显式声明自己的 serialVersionUID，该字段必须是 static、final 和 type long为：


serialVersionUID 不是被 static 变量修饰了吗？为什么还会被“序列化”？

static 修饰的变量是静态变量，属于类而非类的实例，本身是不会被序列化的。然而，serialVersionUID 是一个特例，serialVersionUID 的序列化做了特殊处理。当一个对象被序列化时，serialVersionUID 会被写入到序列化的二进制流中；在反序列化时，也会解析它并做一致性判断，以此来验证序列化对象的版本一致性。如果两者不匹配，反序列化过程将抛出 InvalidClassException，因为这通常意味着序列化的类的定义已经发生了更改，可能不再兼容。


# Java 代理模式详解

## 静态代理和动态代理

静态代理是单独为每一个方法编写代理方法，而动态代理则是通过反射调用编写一个代理方法。

从JVM 层面来说， 
静态代理在编译时就将接口、实现类、代理类这些都变成了一个个实际的 class 文件。
而动态代理是在运行时动态生成类字节码，并加载到 JVM 中的。


## jdk动态代理和cglib动态代理
JDK 动态代理只能代理实现了接口的类或者直接代理接口，而 CGLIB 可以代理未实现任何接口的类。 另外， CGLIB 动态代理是通过生成一个被代理类的子类来拦截被代理类的方法调用，因此不能代理声明为 final 类型的类和方法。


## cglib的核心方法intercept

```
return Enhancer.create(target.getClass(), new MethodInterceptor() {  
@Override  
public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {  
Object result=null;  
try {  
System.out.println("[===================]");  
result = method.invoke(target,args);  
System.out.println("[===================]");  
}catch (Exception E){  
System.out.println(E);  
}  
return result;  
}  
});
```



### Method和MethodProxy对比
intercept用于实现代理流程，而在方法参数中存在两个相似的参数Method和MethodProxy ，他们十分相似，都能够调用方法。



#### 1. **Method**
   - **来源**: `Method` 是 Java 反射 API 的一部分，属于 `java.lang.reflect` 包。
   - **作用**: `Method` 主要用于描述方法的信息，比如方法名、参数类型、返回类型、注解等。它可以通过反射机制来调用方法。
   - **调用方式**: 通过 `Method.invoke(Object obj, Object... args)` 来调用方法。这种方式是基于反射的，因此在性能上相对较低。
   - **适用场景**: 
     - 当你需要获取方法的元信息（如方法名、参数类型等）时，`Method` 是唯一的选择。
     - 当你需要在运行时动态调用某个方法，且无法通过其他方式（如直接调用或 `MethodProxy`）实现时，可以使用 `Method.invoke`。



#### 2. **MethodProxy**
   - **来源**: `MethodProxy` 是 CGLIB（Code Generation Library）库提供的工具，专门用于高效调用代理对象的方法。
   - **作用**: `MethodProxy` 主要用于在代理类中调用被代理对象的方法。它通过生成字节码来直接调用方法，避免了反射的开销。
   - **调用方式**: 通过 `MethodProxy.invokeSuper(Object obj, Object[] args)` 来调用被代理对象的原始方法。这种方式比反射调用更高效。
   - **适用场景**:
     - 当你使用 CGLIB 进行动态代理时，`MethodProxy` 是调用被代理对象方法的首选方式。
     - 当你需要高效调用方法时，`MethodProxy` 比 `Method.invoke` 更合适。


#### 3. **效率对比**
   - **Method**: 由于 `Method` 是基于反射的，每次调用方法时都需要进行方法查找和参数匹配，因此性能较低。
   - **MethodProxy**: `MethodProxy` 通过生成字节码来直接调用方法，避免了反射的开销，因此在性能上优于 `Method`。

#### 4. **功能对比**
   - **Method**: 主要用于获取方法的元信息（如方法名、参数类型、返回类型等），并且可以通过反射调用方法。
   - **MethodProxy**: 主要用于高效调用被代理对象的方法，但不提供获取方法元信息的功能。

#### 5. **总结**
   - 如果你需要获取方法的元信息（如方法名、参数类型等），或者需要在运行时动态调用某个方法，且无法通过其他方式实现，那么 `Method` 是合适的选择。
   - 如果你使用 CGLIB 进行动态代理，并且需要高效调用被代理对象的方法，那么 `MethodProxy` 是更好的选择。
## 动态代理载入


### jdk

核心方法
```
stu proxy= (cn.wenzhuo4657.jdk.stu) Proxy.newProxyInstance(classLoader,interfaces,jdk);
```


该核心方法追溯可以看到
```
Class<?> cl = getProxyClass0(loader, intfs);
```

注意，对于java数据类型来说找到class对象就意味
1，方法区中已经加载了该类型信息
2.可以通过反射得到该数据类型的实例。


尝试追溯该方法，发现了两个值得关注的方法
```
ProxyGenerator.generateProxyClass :生成代理类的字节码，返回一个字节数组

ClassLoader.defineClass：将字节码数组转换为class对象
```


```
byte[] proxyClassFile = ProxyGenerator.generateProxyClass(  
proxyName, interfaces, accessFlags);  
try {  
return defineClass0(loader, proxyName,  
proxyClassFile, 0, proxyClassFile.length);
```

### cglib

cglib载入则使用asm技术，流程类似jdk，依旧是生成字节码数组，然后使用类加载器装载到方法区。

不同
1，cglib可以选择继承的接口/类，而jdk只能代理接口方法
（这意味着cglib代理范围更灵活，jdk返回的代理实例只能将类型指向代理类的接口，而非代理类）
2，cglib默认使用当前线程的类加载器，而jdk动态代理使用方法参数中的类加载。
（springboot当中，jdk的默认类加载器是目标类的类加载哦。）
```
stuImpl stu=new stuImpl();  
ClassLoader classLoader = stu.getClass().getClassLoader(); //直接从目标类中获取。 
Class<?>[] interfaces = stu.getClass().getInterfaces();  
// 封装代理类的执行逻辑处理器  
Jdk jdk = new Jdk(stu);  
stu proxy= (cn.wenzhuo4657.jdk.stu) Proxy.newProxyInstance(classLoader,interfaces,jdk);  
proxy.goSchool();
```

# 集合
## 接口定义

顶级接口为`Collection<E>`和`Map<K,V >`。
![[Pasted image 20250108173012.png]]
![[Pasted image 20250108173029.png]]


list接口：定义有序集合,不考虑元素重复，常见实现ArrayList是数组结构实现，因此对于索引查找比较有优势
set接口：定义集合元素不能重复，常见实现hashset使用hashmap存储数据，因此元素无序。
Queue：定义队列实现，队列通常（但不一定）以 FIFO （先进先出） 方式对元素进行排序。
Map：键值存储，无序且不能重复。

## 为什么要使用集合？
集合作为存储数据的数据类型，与数据有一定的重复性，但同样也实现了额外的功能，例如在并发场景下的安全问题，又或者是应用场景下功能封装，比如对于TreeSet的排序问题，作为程序员只需编写比较器即可，复杂的算法由源码定义。总之使用集合并不是因为可以存储数据，而是其功能较为丰富，满足多种场景。


## 如何选用集合?




## LinkedList 为什么不能实现 RandomAccess 接口？
RandomAccess是一个标记接口，表示随机访问功能的支持，而linkedlist内部使用node节点所链接的双端队列存储数据，只能通过指针顺序或者逆序查找，因此不能实现。


## 什么是 BlockingQueue？
阻塞队列，其应用内场景为消费者-生产者，当队列内部没有元素时会阻塞消费者，当队列已满时会阻塞生产者

## HashMap 的长度为什么是 2 的幂次方

1，位运算效率更高：位运算(&)比取余运算(%)更高效。当长度为 2 的幂次方时，hash % length 等价于 hash & (length - 1)。
2，可以更好地保证哈希值的均匀分布：扩容之后，在旧数组元素 hash 值比较均匀的情况下，新数组元素也会被分配的比较均匀，最好的情况是会有一半在新数组的前半部分，一半在新数组后半部分。
3，扩容机制变得简单和高效：扩容后只需检查哈希值高位的变化来决定元素的新位置，要么位置不变（高位为 0），要么就是移动到新位置（高位为 1，原索引位置+原容量）。


## 红黑树是否会退化为链表？
当红黑树节点少于阈值时会退化为单链表，一般时5-8。

## concurrentHashmap1.7vs1.8

数据结构：1.7采用segments数组+hashtable存储数据，1.8采用Node数组+链表/红黑树结构。
并发瓶颈：1.7的直接引用segments对象，并且继承ReentrantLock作为锁机制，所以无论内部用于存储的数据的hashtable或者是其他结构，1.7的性能瓶颈一定是segments的数量，而1.8则通过cas操作和头部节点加锁避开这一瓶颈，虽然大部分情况下桶的数量和sgments是一一对应关系，但是在数据迁移、桶的结构转换时，他实现了更加细微的锁颗粒度。
数据读取：****


## 什么是哈希表

hash table是指通过k计算出v的存储位置的一种数据结构，在java集合中有hashmap、concurrenthashmap等集合就是使用hashtable，其中对于hashtable常见问题的哈希冲突，它们使用的是拉链法，即在冲突的坐标位置之后继续延长存储链表等结构，通过等值判断k得到v,这里也需要注意拉链法的原理实际上是hash一致，但key并不一致这一原则。


## 为什么arraylist检索快增删慢

arraylist内部使用数组存储数据，只需要使用坐标就可以实现随机读取。

而增删慢则是因为，他支持指定位置增删，并不总是在尾部进行，所以大部分时候操作都会伴随着其余元素的移动，
# 线程池


## 线程池队列过长，最大线程失效

这个问题的本质是由于线程池创建新线程的机制导致的，
**corePoolSize**：常驻线程池数
**maximumPoolSize**：最大线程数


workerCountOf < corePoolSize时，新提交的任务会创建线程执行
workerCountOf > corePoolSize时，会将任务添加至队列当中，等待常驻线程执行。只有当队列满了，而工作线程<最大线程数时，才会新建线程。


因此可能会存在这样一种场景，队列过长且任务耗时，然而线程池并没有创建线程处理，一直排队等待核心线程处理。

jdk1.8 ThreadPoolExecutor.class源码
```
int c = ctl.get();  
if (workerCountOf(c) < corePoolSize) {  
if (addWorker(command, true))  
return;  
c = ctl.get();  
}  
if (isRunning(c) && workQueue.offer(command)) {  
int recheck = ctl.get();  
if (! isRunning(recheck) && remove(command))  
reject(command);  
else if (workerCountOf(recheck) == 0)  
addWorker(null, false);  
}  
else if (!addWorker(command, false))  
reject(command);
```

1，addWorker(command, true)：添加任务
true:使用核心线程数
false:使用最大线程数
2， workQueue.offer(command)：添加任务到队列



## 核心线程会被回收吗？
默认情况下，Java线程不会被回收，但是可以通过设置allowCoreThreadTimeOut和keepAliveTime来控制，allowCoreThreadTimeOut用于表示控制是否启用超时回收核心线程，而keepAliveTime则是超时时间的设置，即正常的空闲线程超时时间。

## 线程池的拒绝策略

ThreadPoolExecutor.AbortPolicy：拒绝任务并抛出错误、
ThreadPoolExecutor.CallerRunsPolicy：调用提交任务的线程执行。这意味着将会降低提交线程的性能，并且提交任务执行是否成功将取决于提交线程，如果提交线程提前关闭，则会丢弃任务。
ThreadPoolExecutor.DiscardPolicy：拒绝任务，
ThreadPoolExecutor.DiscardOldestPolicy：丢弃队列头部的任务，即未处理且最早提交的任务。

## CallerRunsPolicy 拒绝策略有什么风险？如何解决？
该策略实际上是将提交线程作为担保，即如果线程池处理不了则剩余任务由提交线程处理，这样做的风险即是如果处理任务非常耗时则会严重损耗提交线程的性能，如果提交线程是主线程则会成为系统性能的瓶颈。


## 线程池中线程异常后，销毁还是复用？
使用execute()提交任务：抛出异常会导致执行线程终止，而线程池会重新创建一个线程替代。
使用submit()提交任务：抛出异常不会导致线程终止，而是被封装在方法的回调对象Future对象中当中。

## 如何给线程池命名？

可以通过线程池参数中的ThreadFactory，该参数用于在线程池中创建线程。
可以自行实现ThreadFactory接口，或者使用工具类中已实现的。

## 如何设计一个能够根据任务的优先级来执行的线程池？

使用优先级阻塞队列，例如PriorityBlockingQueue，内部再添加任务时会使用排序，如果传入Comparator对象则使用该参数排除，如果没有则使用自然排序，即添加元素实现Comparable接口。


## Callable 和 Future 有什么关系？

callable接口用于定义异步任务。而future用于定义如何操作callable接口定义的任务，并且在实现中将其融入Runnable的体系，因为new Thread以及各种封装所接收都是Runnable接口。
从功能上讲callable定义了一个有返回值的任务，而future则是完整的异步任务定义，包括如何获取返回值、任务出现错误如何处理等。


从接口实现角度来看，实现Future接口的FutureTask类更像是异步方法的定义，它同时实现了Runnable和future接口，并且实现了借助callable的call方法完成了异步任务。
```
public void run() {  
if (state != NEW ||  
!UNSAFE.compareAndSwapObject(this, runnerOffset,  
null, Thread.currentThread()))  
return;  
try {  
Callable<V> c = callable;  
if (c != null && state == NEW) {  
V result;  
boolean ran;  
try {  
result = c.call();  //callable仅仅定义了执行异步任务的内容
ran = true;  
} catch (Throwable ex) {  
result = null;  
ran = false;  
setException(ex);  
}  
if (ran)  
set(result);  
}  
} finally {  
// runner must be non-null until state is settled to  
// prevent concurrent calls to run()  
runner = null;  
// state must be re-read after nulling runner to prevent  
// leaked interrupts  
int s = state;  
if (s >= INTERRUPTING)  
handlePossibleCancellationInterrupt(s);  
}  
}
```

也就是说这两者之间的关系就是FutureTask类，而该类又实现了异步方法真正的接口RunnableFuture，而callable接口只是作为该类的参数。

# 锁

## 如何实现乐观锁？
乐观锁一般会使用版本号机制和 CAS 算法组合实现。


## 简述cas
cas是无锁编程中的原子操作，一般包含内存地址、预期值、新值三个操作，用于实现快速判断变量是否被修改并且替换。

## Java中的CAS如何实现
在 Java 中，实现 CAS（Compare-And-Swap, 比较并交换）操作的一个关键类是Unsafe。

Unsafe类位于sun.misc包下，是一个提供低级别、不安全操作的类。由于其强大的功能和潜在的危险性，它通常用于 JVM 内部或一些需要极高性能和底层访问的库中，而不推荐普通开发者在应用程序中使用



# JUC

## 什么是AQS
AQS实现阻塞-等待机制，将每条请求共享资源的线程封装成CLH锁队列的一个Node节点，插入就更新尾节点，出队列就更新头节点。
该抽象类内部还有ConditionObject，用于向Node节点内添加、删除节点。
总之AQS的两个内部类主要功能是实现了一个虚拟的双端队列，而AQS成员变量和成员方法则是定义了一些其余的功能，但是最主要的tryAcquire等独占锁和非独占锁的加锁、解锁则交由子类定义。

## 什么是Semaphore

是aqs的一种共享锁实现，默认构造 AQS 的 state 值为 permits，也就是令牌数量，只有拿到许可证的线程才能执行


## ReentrantLock是怎么保证线程安全的

通过cas操作对state状态值的预期判断实现锁的安全获取，并且他继承了aqs维护了一个 FIFO 的等待队列，用于管理获取锁失败的线程,在具体的实现代码当中使用自旋等待当前节点是队列头部之后获取锁，此外还支持可重入的性质，aqs会将获取的锁的线程记录，当其再次获取锁时会自动获取，并在state状态变量上执行自增操作。

# IO
## IO模型
同步阻塞 I/O（bio）、同步非阻塞 I/O(nio)、I/O 多路复用(reator模型)、信号驱动 I/O 和异步 I/O(aio)。


## NIO 零拷贝
[零拷贝](../java编程学习笔记/Rocket%20MQ：数据量小也可实现高吞吐.md#零拷贝)

