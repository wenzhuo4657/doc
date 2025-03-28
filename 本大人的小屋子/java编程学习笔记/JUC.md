
# 概念
参考：[Java 并发 - 理论基础 | Java 全栈知识体系](https://pdai.tech/md/java/thread/java-thread-x-theorty.html)


## 并发三要素：
可见性：一个线程对共享变量的修改，另外一个线程能够立刻看到。
而这种问题的本质是因为cache高速缓存，由于cpu和主内存的io速度有过大差距，因此中间还有一层cache缓存，线程对变量的操作并不会立即显示到主存当中。、

原子性：原子性操作。操作步骤不可分割



有序性：指令执行的顺序需要符合代码的语义。
在执行程序时为了提高性能，编译器和处理器常常会对指令做重排序，最终导致程序顺序不符号预期。（重排序视角为纯指令，无法预测翻译前代码的预期行为，进而可能会导致错误的优化。）
![[Pasted image 20241229193024.png]]



## java解决

绝佳： [「跬步千里」详解 Java 内存模型与原子性、可见性、有序性](https://link.juejin.cn/?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%2F2hKZ-iNhSkUiUmxeVPHXCw "https://mp.weixin.qq.com/s/2hKZ-iNhSkUiUmxeVPHXCw")

可见性解决：
可见性问题本质上是cache缓存和主内存之间的同步问题，对此**JMM提供按需禁用缓存和编译优化的方法。**

volatile：当一个共享变量被volatile修饰时，它会保证修改的值会立即被更新到主存，当有其他线程需要读取时，它会去内存中读取新值。
synchronized：解锁前会将修改共享变量的值写入主内存，隐式解决了可见性
final:毫无疑问的是，如果获取的是初始化完成的final,该值一定不可变，可见性问题也就没有发生。
[【面试问题】final 和可以保证可见性吗？-阿里云开发者社区](https://developer.aliyun.com/article/1432647)

原子性解决：

java代码的原子性错误。
```
cnt++;
---字节码指令
0: getstatic #2 // Field cnt:I  
3: iconst_1  
4: iadd  
5: putstatic #2 // Field cnt:I
```

对于cpu来说，java代码的代码并非是原子性，因此有点像是有序错误，但并非代码层面，而是字节码层面、机器代码层面的。

同步方法（synchronized：隐式加锁）、cas原子操作等


有序性解决：
**根本问题：重排序导致指令的执行顺序与代码语义的执行顺序出现歧义。**


 as-if-serial ：不管怎么重排序，**单线程环境**下程序的执行结果不能被改变。
在重排序时，CPU 和编译器都需要遵守 as-if-serial语义，具体实现上来说则是保证CPU 和编译器不会对存在**数据依赖关系的操作**做重排序。

然而在多线程多核心cpu场景下，不同 CPU 之间和不同线程之间的数据依赖性是不被 CPU 和编译器考虑的，因此会导致指令重排序的错误。问题本质上是无法预知数据依赖的关系：在线程a上观察线程b，线程b的操作都是无序的。

这里需要注意的是，重排序导致错误是代码语义上的执行不准确，具体来说，则是由于指令重排导致多线程下代码执行逻辑错误，而执行代码逻辑错误的根本原因又在与不同线程执行的代码具有关联性、数据依赖性，假设这些代码毫无关联，也就无所谓是否重排序。
JMM从这一点出发，将所有有关联的代码集中在一处，称其为同步方法，同步代码块，保证同一时间下只有一个线程会执行该代码，自然就避免了指令重排序的问题。

同步方法：synchronized（同步代码范围内会增加内存屏障）



## Happens-before

参考：[JMM 最最最核心的概念：Happens-before 原则Happens-before 是 JMM 最核心的概念。对应 - 掘金](https://juejin.cn/post/6960128601249284110)
Happens-before 提供**跨线程的内存可见性保证**


> [!NOTE] 《JSR-133：Java Memory Model and Thread Specification》对 Happens-before 关系的定义
> 1）如果一个操作 Happens-before 另一个操作，那么第一个操作的执行结果将对第二个操作可见，而且第一个操作的执行顺序排在第二个操作之前。
> 2）两个操作之间存在 Happens-before 关系，并不意味着 Java 平台的具体实现必须要按照 Happens-before 关系指定的顺序来执行。如果重排序之后的执行结果，与按 Happens-before 关系来执行的结果一致，那么这种重排序并不非法（也就是说，JMM 允许这种重排序）



## 并发线程

[Java 并发 - 线程基础 | Java 全栈知识体系](https://pdai.tech/md/java/thread/java-thread-x-thread-basic.html)

```
  
public class Main {  
public static class MyRunnable implements Runnable {  
int a;  
@Override  
public void run() {  
synchronized (this){  
System.out.println(a++);  
}  
}  
}  
public static class MyCallable implements Callable<String> {  
int a;  
@Override  
public String call() throws Exception {  
char x= (char) a++;  
Thread.sleep(1000);  
return String.valueOf(x);  
}  
}  
static int cnt=100;  
public static void main(String[] args) throws InterruptedException, ExecutionException {  
  
// Runnable接口  
// MyRunnable myRunnable = new MyRunnable();  
//  
// for (int i=0;i<cnt;i++){  
// Thread thread = new Thread(myRunnable);  
// thread.start();  
// }  
  
// Callable<String>  
// MyCallable myCallable=new MyCallable();  
// for (int i=0;i<cnt;i++){  
// FutureTask<String> ft;  
// ft = new FutureTask<>(myCallable);  
// Thread thread = new Thread(ft);  
// thread.start();  
// String s = ft.get();//阻塞方法  
// System.out.println(s);  
// }  
  
// Executors：线程池  
  
ThreadFactory daemonThreadFactory = r -> {  
Thread thread = new Thread(r);  
thread.setDaemon(true); // 设置为守护线程  
return thread;  
};  
MyRunnable myRunnable = new MyRunnable();  
ExecutorService executorService = Executors.newCachedThreadPool(daemonThreadFactory);//设置创建线程方式  
for (int i=0;i<cnt;i++){  
executorService.execute(myRunnable);  
executorService.shutdown();//拒绝新任务。  
  
}  
System.out.println(myRunnable);  
  
  
  
}  
}
```
## 锁


[Java并发 - Java中所有的锁 | Java 全栈知识体系](https://pdai.tech/md/java/thread/java-thread-x-lock-all.html)

可重入锁：
可重入锁的可重入性是通过 锁计数器 实现的：
- 当一个线程第一次获取锁时，JVM 会记录锁的持有者，并将计数器设置为 1。
- 如果同一个线程再次获取同一把锁，计数器会递增。
- **当线程退出同步代码块或方法时，计数器会递减**。
- 当计数器减到 0 时，锁才会被完全释放，其他线程才有机会获取锁。

需要注意的是，可重入的锁的释放是当计数器为0，而并非同步方法结束。

synchronized在字节码指令中被翻译为monitorenter和monitorexit。
```
1: getfield #3 // Field object:Ljava/lang/Object;  
4: dup  
5: astore_1  
6: monitorenter  
7: aload_1  
8: monitorexit  
9: goto 17  
12: astore_2  
13: aload_1  
14: monitorexit
```


# synchronized

## 重量级锁以及jvm优化

重量级锁：依赖于操作系统底层的Mutex Lock，这意味着使用该锁管理线程必定需要将线程升级为内核态线程，换言之，是操作系统内核在管理不同线程对共享资源的访问。同时，线程获取锁和释放锁的过程中也伴随着线程上下文的切换。

这意味着重量级锁不可避免地两个开销：
1，将普通线程升级为内核态线程，便于操作系统管理。
2，锁获取、释放过程中，需要唤醒、阻塞休眠线程。

此外，如果锁竞争激烈，则会导致大量的线程上下文和内核切换，进一步地增加系统额外开销。


对此jvm所做的优化大致分为两个方向
1，尽可能地减少、优化锁地范围，减少操作系统管理地次数
2，假设一种场景，锁竞争并不激烈，或者说等待锁的损耗不如线程上下文切换和升级内核态的损耗。尝试将线程持续尝试获取锁，也就是将**cpu占用等线程活跃的损耗**来替代**升级内核态、线程上文切换、唤醒休眠核心线程的损耗**。（需要注意的是，两者损耗究竟哪一方比较大，需要结合具体的场景来判断）
## 适应性自旋锁和轻量级锁

轻量级锁的原理：通过cas等操作保证锁在多线程场景的竞争，同时，假设锁的持有时间短，持续尝试获取锁。
适应性自旋锁：对轻量级锁的增强，如果尝试获取了**一定次数**（该次数会适应性改变，具体改变要看锁的机制）依旧没有获取到锁（多次尝试没有获取到实际上也违反了轻量级锁的初衷），就将本线程休眠。

## 锁消除
**锁消除是指虚拟机即时编译器再运行时，对一些代码上要求同步，但是被检测到不可能存在共享数据竞争的锁进行消除**



## 锁粗化
将多次获取、释放锁合成为一次大范围的获取锁和释放锁，jvm自动优化。

## 偏向锁
场景：在大多实际环境下，锁不仅不存在多线程竞争，而且总是由同一个线程多次获取，那么在同一个线程反复获取所释放锁中，其中并还没有锁的竞争，那么这样看上去，多次的获取锁和释放锁带来了很多不必要的性能开销和上下文切换。


当一个线程访问同步块并获取锁时，会在对象头和栈帧中的锁记录里存储锁偏向的线程ID，以后该线程在进入和退出同步块时不需要进行CAS操作来加锁和解锁。（**存储上一个获取锁的线程记录，换言之，通过缓存判断**）


损耗：偏向锁本身并不是锁，而是一种缓存上一个获取锁的进程的机制，这意味着该步骤是对单线程多次获取锁的优化，避免了获取锁、释放锁的损耗。但同时也对多线程竞争场景下的锁竞争带来了额外的损耗。


# volatile

内存屏障其实就是一个CPU指令，在硬件层面上来说可以扥为两种：Load Barrier 和 Store Barrier即读屏障和写屏障。主要有两个作用：

（1）阻止屏障两侧的指令重排序；

（2）强制把写缓冲区/高速缓存中的脏数据等写回主内存，让缓存中相应的数据失效。


```
private volatile int a;
public void update() {  
a = 1;  
}

====字节码=
0: aload_0  
1: iconst_1  
2: putfield #2 // Field a:I  
5: return

```


```
public void print(){  
if (a==1){  
  
}  
}
=====字节码=
0: aload_0  
1: getfield #2 // Field a:I  
4: iconst_1  
5: if_icmpne 8  
8: return
```


如果字段存在修饰符volatile，jvm会在执行getfield和putfiled指令前后添加插入内存屏障，禁止某些导致错误的指令重排序。同时在写入时，会禁用工作缓存。
![[Pasted image 20250102171717.png]]
![[Pasted image 20250102171728.png]]


具体来说，如new一个对象，分为3步骤
1，分配空间
2，初始化空间，（执行构造方法）
3，赋值引用

假如没有使用volatile修饰，执行顺序可能为132，这意味着可能会使用一个没有初始化的引用。
具体来说，这更像是代码的原子性问题，部分语句在编译后需要分步骤进行，仅从单个代码执行的角度来说没有问题，毕竟一定是可以执行完毕的，但是从代码语义上来说，某些重排序违反了代码语义。
```
singleton = new Singleton();
=========字节码===============
17: new #3 // class cn/wenzhuo4657/IncreaseSecret/Singleton  
20: dup  
21: invokespecial #4 // Method "<init>":()V  
24: putstatic #2 // Field singleton:Lcn/wenzhuo4657/IncreaseSecret/Singleton;  
27: aload_0
```




# CAS
CAS的全称为Compare-And-Swap，直译就是对比交换。是一条CPU的原子指令，其作用是让CPU先进行比较两个值是否相等，然后原子地更新某个位置的值，经过调查发现，其**实现方式是基于硬件平台的汇编指令**，就是说CAS是靠硬件实现的，JVM只是封装了汇编调用，那些AtomicInteger类便是使用了这些封装后的接口。  
简单解释：
**CAS操作需要输入两个数值，一个旧值(期望操作前的值)和一个新值，在操作期间先比较下在旧值有没有发生变化，如果没有发生变化，才交换成新值，发生了变化则不交换。**


## 问题
### ABA问题

CAS实际上基于一种假设，**如果期望的值等于内存中的值**，我们就认为该共享资源没有被获取或者说可以改变。

那么假若A-》B-》A,此时原始的汇编指令CAS则不足以应对这种变化，java的解决方式是追加版本号。例如 1A-》2B-》3A；
JDK的Atomic包里提供了一个类**AtomicStampedReference**来解决ABA问题。

### 循环时间长开销大

Java原子类（如AtomicInteger、AtomicReference等）会在cas操作失败时默认自旋，而这种自旋会带来额外的开销。

对此如果JVM能支持处理器提供的pause指令，那么效率会有一定的提升。（**HotSpot JVM（JDK 5及以上版本）在x86架构中已经支持PAUSE指令，并会自动优化自旋锁和CAS操作**。）

### 只能保证一个变量的原子操作

把多个共享变量合并成一个共享变量来操作，从Java 1.5开始，JDK提供了AtomicReference类来保证引用对象之间的原子性。


##  UnSafe类（了解）
Unsafe是位于sun.misc包下的一个类，主要提供一些用于执行低级别、不安全操作的方法，如直接访问系统内存资源、自主管理内存资源等，这些方法在提升Java运行效率、增强Java语言底层资源操作能力方面起到了很大的作用。但由于Unsafe类使Java语言拥有了类似C语言指针一样操作内存空间的能力，这无疑也增加了程序发生相关指针问题的风险。在程序中过度、不正确使用Unsafe类会使得程序出错的概率变大，使得Java这种安全的语言变得不再“安全”，因此对Unsafe的使用一定要慎重。
![[Pasted image 20250103123516.png]]

[JUC锁: LockSupport详解 | Java 全栈知识体系](https://pdai.tech/md/java/thread/java-thread-x-lock-LockSupport.html)


sun.misc.Unsafe类中的park和unpark函数
```
public native void park(boolean isAbsolute, long time);
public native void unpark(Thread thread);
```


park函数，阻塞线程，并且该线程在下列情况发生之前都会被阻塞: ① 调用unpark函数，释放该线程的许可。② 该线程被中断。③ 设置的时间到了。并且，当time为绝对时间时，isAbsolute为true，否则，isAbsolute为false。当time为0时，表示无限等待，直到unpark发生。

unpark函数，释放线程的许可，即激活调用park后阻塞的线程。这个函数不是安全的，调用这个函数时要确保线程依旧存活。

## AtomicStampedReference

```
public class AtomicStampedReference<V> {  
  
private static class Pair<T> {  
final T reference;  
final int stamp;  
private Pair(T reference, int stamp) {  
this.reference = reference;  
this.stamp = stamp;  
}  
static <T> Pair<T> of(T reference, int stamp) {  
return new Pair<T>(reference, stamp);  
}  
}  
  
private volatile Pair<V> pair;  
/**  
使用给定的初始值创建一个新 AtomicStampedReference 的。
形参:
initialRef – 初始参考 
initialStamp – 首字母印章
*/  
public AtomicStampedReference(V initialRef, int initialStamp) {  
pair = Pair.of(initialRef, initialStamp);  
}

......................

}
```

# JUC锁

## LockSupport
[JUC锁: LockSupport详解 | Java 全栈知识体系](https://pdai.tech/md/java/thread/java-thread-x-lock-LockSupport.html)


###  unpark、park 对比 notify、wait

notify详解：
notify并不会释放锁，只是**唤醒正在等待此对象的监视器的单个线程**，也就是说该方法是锁对象唤醒竞争线程的方法，而非持有锁的线程方法。

而park则是**出于线程调度目的禁用当前线程**，这意味着这是一个线程级别的方法，同时会释放锁。

本质上来讲，notify并不会干涉线程竞争锁的流程，只是唤醒了休眠的线程，换言之，该方法确保r如果有其他线程需要该资源，释放锁时会立即响应持有锁，而并非停留在wait方法所引起的休眠状态。



**实际上来说unpark、park方法与锁机制无关** 两个方法都是对于线程操作的方法，是非常具体的唤醒、休眠某线程的操作，且不会阻塞。

### 示例

1,notify唤醒失败：在该流程当中notify是锁对象myThread的方法 ，然而遗憾的是，在执行方法时，主线程并没有在休眠等待争抢锁，而是主动休眠Thread.sleep(3000)。

```
class MyThread extends Thread {
    public void run() {
        synchronized (this) {
            System.out.println("before notify");            
            notify();
            System.out.println("after notify");    
        }
    }
}

public class WaitAndNotifyDemo {
    public static void main(String[] args) throws InterruptedException {
        MyThread myThread = new MyThread();        
        myThread.start();
        // 主线程睡眠3s
        Thread.sleep(3000);
        synchronized (myThread) {
            try {        
                System.out.println("before wait");
                // 阻塞主线程
                myThread.wait();
                System.out.println("after wait");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }            
        }        
    }
}
```


2，唤醒休眠某线程
```
import java.util.concurrent.locks.LockSupport;

class MyThread extends Thread {
    private Object object;

    public MyThread(Object object) {
        this.object = object;
    }

    public void run() {
        System.out.println("before unpark");
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        // 获取blocker
        System.out.println("Blocker info " + LockSupport.getBlocker((Thread) object));
        // 释放许可
        LockSupport.unpark((Thread) object);
        // 休眠500ms，保证先执行park中的setBlocker(t, null);
        try {
            Thread.sleep(500);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        // 再次获取blocker
        System.out.println("Blocker info " + LockSupport.getBlocker((Thread) object));

        System.out.println("after unpark");
    }
}

public class test {
    public static void main(String[] args) {
        MyThread myThread = new MyThread(Thread.currentThread());
        myThread.start();
        System.out.println("before park");
        // 获取许可
        LockSupport.park("ParkAndUnparkDemo");
        System.out.println("after park");
    }
}
```



## 锁核心类AQS（AbstractQueuedSynchronizer）



AQS定义两种资源共享方式

Exclusive(独占)：只有一个线程能执行，如ReentrantLock。又可分为公平锁和非公平锁：
公平锁：按照线程在队列中的排队顺序，先到者先拿到锁
非公平锁：当线程要获取锁时，无视队列顺序直接去抢锁，谁抢到就是谁的
Share(共享)：多个线程可同时执行。

不同的自定义同步器争用共享资源的方式也不同。**自定义同步器在实现时只需要实现共享资源 state 的获取与释放方式即可**，至于具体线程等待队列的维护(如获取资源失败入队/唤醒出队等)，AQS已经在上层已经帮我们实现好了



### ReentrantLock实现
可重入锁、内部有非公平、公平锁实现。（没有自旋，获取失败会排队等待）

![[Pasted image 20250103163613.png]]


reentrantLock有三个内部类，继承实现关系如上。


sync的非公平获取锁实现
以下实现实际上是可重入锁的非公平实现，获取锁不会查看等待队列，尝试插队。

```
/**  
* Performs non-fair tryLock. tryAcquire is implemented in  
* subclasses, but both need nonfair try for trylock method.  
*/  
final boolean nonfairTryAcquire(int acquires) {  
final Thread current = Thread.currentThread();  
int c = getState();  
if (c == 0) {  
if (compareAndSetState(0, acquires)) {  
setExclusiveOwnerThread(current);  
return true;  
}  
}  
else if (current == getExclusiveOwnerThread()) {  
int nextc = c + acquires;  
if (nextc < 0) // overflow  
throw new Error("Maximum lock count exceeded");  
setState(nextc);  
return true;  
}  
return false;  
}
```


fairSync的获取公平锁实现
以下实现在遇到没有线程持有锁的情况时，会在查看当先线程是否是队列头部，因此是公平锁。
```
/**  
* Fair version of tryAcquire. Don't grant access unless  
* recursive call or no waiters or is first.  
*/  
protected final boolean tryAcquire(int acquires) {  
final Thread current = Thread.currentThread();  
int c = getState();  
if (c == 0) {  
if (!hasQueuedPredecessors() &&  
compareAndSetState(0, acquires)) {  
setExclusiveOwnerThread(current);  
return true;  
}  
}  
else if (current == getExclusiveOwnerThread()) {  
int nextc = c + acquires;  
if (nextc < 0)  
throw new Error("Maximum lock count exceeded");  
setState(nextc);  
return true;  
}  
return false;  
}
```


### ReentrantReadWriteLock

实质上该类是对ReadLock和WriteLock两种锁的封装，用于实现锁机制的sync、nofairSync、fairSync，还有具体应用锁机制的封装对象ReadLock、writeLock。实现的基本逻辑与ReentrantLock一致。

不同的地方在于对读锁和写锁的封装使用。构造器如下
```
==============================ReentrantReadWriteLock
public ReentrantReadWriteLock() {  
this(false);  
}  
  

public ReentrantReadWriteLock(boolean fair) {  
sync = fair ? new FairSync() : new NonfairSync();  
readerLock = new ReadLock(this);  
writerLock = new WriteLock(this);  
}
================================ReadLock
protected ReadLock(ReentrantReadWriteLock lock) {  
sync = lock.sync;  
}

===============================WriteLock
protected WriteLock(ReentrantReadWriteLock lock) {  
sync = lock.sync;  
}

```




# JUC集合

## ConcurrentHashMap
### put方法分析


查询jdk1.8源码可知，该方法时putVal的封装,
```
public V put(K key, V value) {  
return putVal(key, value, false);  
}
```

而`final V putVal(K key, V value, boolean onlyIfAbsent) `在源码注释中显示为put and putIfAbsent的实现，两者仅仅在于boolean参数，表示当有键不存时的处理。
```
final V putVal(K key, V value, boolean onlyIfAbsent) {  
if (key == null || value == null) throw new NullPointerException();

int hash = spread(key.hashCode());  //key的哈希值计算
int binCount = 0;  //计数，用于判断是否需要扩容

for (Node<K,V>[] tab = table;;) {  //无线循环，必须由循环内部主动退出循环，这意味着无需刻意设置组合的if分类，因为不止进入一次，换言之，该循环是判断是否存在不能修复的错误条件。

Node<K,V> f; int n, i, fh;  
if (tab == null || (n = tab.length) == 0)  //1，判断表是否初始化
tab = initTable();  //得益于无限循环，只要不移动指针，直接退出就可重新判断。
else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {  //2，读取节点( i = (n - 1) & hash)代表数组下标位置计算)，判断是否hash碰撞
if (casTabAt(tab, i, null,  
new Node<K,V>(hash, key, value, null)))  //cas添加节点，添加成功则退出循环
break; // no lock when adding to empty bin  
}  
else if ((fh = f.hash) == MOVED)  //3，。
tab = helpTransfer(tab, f);  
else {  
// 4，解决hash碰撞，添加节点
V oldVal = null;  
synchronized (f) {  
if (tabAt(tab, i) == f) {  
if (fh >= 0) {  //表示为一个链表，根据赋值也就是f.hash
binCount = 1;  
for (Node<K,V> e = f;; ++binCount) {  
K ek;  
if (e.hash == hash &&  
((ek = e.key) == key ||  
(ek != null && key.equals(ek)))) {  //找到hash碰撞的节点，
oldVal = e.val;  
if (!onlyIfAbsent)  
e.val = value;  
break;  
}  
Node<K,V> pred = e;  
if ((e = e.next) == null) {  //移动指针，并在移动到末尾时跳出循环，但是有问题的是，如果将hash碰撞的节点多次添加，则会创建多个的空值节点。
pred.next = new Node<K,V>(hash, key,  
value, null);  
break;  
}  
}  
}  
else if (f instanceof TreeBin) {  //红黑树的判断，
Node<K,V> p;  
binCount = 2;  
if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,  
value)) != null) {  
oldVal = p.val;  
if (!onlyIfAbsent)  
p.val = value;  
}  
}  
}  
}  
if (binCount != 0) {  
if (binCount >= TREEIFY_THRESHOLD)  
treeifyBin(tab, i); //扩容判断，1，数组扩容 2，红黑树扩容 
if (oldVal != null)  
return oldVal;  
break;  
}  
}  
}  
addCount(1L, binCount);  
return null;  
}
```

对比jdk17的源码，发现其中做了一个小优化，就是在判断返回值时，如果使用的时putifAbsent方法，在查询是否发生hash碰撞之后立即查询是否是相同的键，如果是则返回其值。

```
if (tab == null || (n = tab.length) == 0)  
.....
else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {  
.....
}  
else if ((fh = f.hash) == MOVED)  
....
else if (onlyIfAbsent // check first node without acquiring lock  
&& fh == hash  
&& ((fk = f.key) == key || (fk != null && key.equals(fk)))  
&& (fv = f.val) != null)  //优化判断
return fv;  
else {
.............
}
```

### get分析


```
public V get(Object key) {  
Node<K,V>[] tab; Node<K,V> e, p; int n, eh; K ek;  
int h = spread(key.hashCode());  
if ((tab = table) != null && (n = tab.length) > 0 &&  
(e = tabAt(tab, (n - 1) & h)) != null) {  
if ((eh = e.hash) == h) {  
if ((ek = e.key) == key || (ek != null && key.equals(ek)))  
return e.val;  
}  
else if (eh < 0)  
return (p = e.find(h, key)) != null ? p.val : null;  
while ((e = e.next) != null) {  
if (e.hash == h &&  
((ek = e.key) == key || (ek != null && key.equals(ek))))  
return e.val;  
}  
}  
return null;  
}
```


## CopyOnWriteArrayList

CopyOnWriteArrayList是ArrayList 的一个线程安全的变体，其中所有可变操作(add、set 等等)都是通过对底层数组进行一次新的拷贝来实现的。

并且该集合是cow模式的体现，即修改操作会**复制全部副本**进行修改，读取操作则直接读取原始数据，这样做的好处时，读取时不需要加锁，因为不会带来数据改变，重点在于修改操作的可见性和安全性，不可避免会将修改操作的损耗提高。


**该实现在修改时加锁，但并没有改变原数组引用，也就是说读取的数据可能不及时**

因此该集合适用于多读取，少写入的场景。



jdk1.8源码当中
```
public void add(int index, E element) {  
final ReentrantLock lock = this.lock;  
lock.lock();  
try {  
Object[] elements = getArray();  
int len = elements.length;  
if (index > len || index < 0)  
throw new IndexOutOfBoundsException("Index: "+index+  
", Size: "+len);  
Object[] newElements;  
int numMoved = len - index;  
if (numMoved == 0)  
newElements = Arrays.copyOf(elements, len + 1);  
else {  
newElements = new Object[len + 1];  
System.arraycopy(elements, 0, newElements, 0, index);  
System.arraycopy(elements, index, newElements, index + 1,  
numMoved);  
}  
newElements[index] = element;  
setArray(newElements);  
} finally {  
lock.unlock();  
}  
}
```


使用了ReentrantLock可重入锁的非公平锁所实现。并且注意添加元素实际上上的动作是copy到一个新的数组Arrays.copyOf(elements, len + 1)。


在jdk17当中使用synchronized(),锁则是普通final transient Object lock = new Object();
该优化的主要原因有两点
1，synchronized()锁状态升级，在轻量级锁状态中支持自旋，从功能上可以替代ReentrantLock。
2，从场景上来讲CopyOnWriteArrayList只有写操作使用锁，并且基于cow模型，应该是一个竞争少的场景，并且性能瓶颈于锁无关，而是数组复制，而ReentrantLock的初衷则是灵活转变，并且线程等待队列的存在要求时间短，而数据复制与这一场景违背。因此jdk17中选择使用了synchronized锁。




add源码
```
private E get(Object[] a, int index) {  
return (E) a[index];  
}
```



## ConcurrentLinkedQueue
**无锁的线程安全队列**

根据源码可知，head代码头节点，tail代表尾节点，同时二者的更新实际不同

```
public boolean offer(E e) {  
checkNotNull(e);  
final Node<E> newNode = new Node<E>(e);  
  
for (Node<E> t = tail, p = t;;) {  
Node<E> q = p.next;  
if (q == null) {  
// p is last node  
if (p.casNext(null, newNode)) {  
// Successful CAS is the linearization point  
// for e to become an element of this queue,  
// and for newNode to become "live".  
if (p != t) // hop two nodes at a time  
casTail(t, newNode); // Failure is OK.  
return true;  
}  
// Lost CAS race to another thread; re-read next  
}  
else if (p == q)  
// We have fallen off list. If tail is unchanged, it  
// will also be off-list, in which case we need to  
// jump to head, from which all live nodes are always  
// reachable. Else the new tail is a better bet.  
p = (t != (t = tail)) ? t : head;  
else  
// Check for tail updates after two hops.  
p = (p != t && t != (t = tail)) ? t : q;  
}  
}  
  
public E poll() {  
restartFromHead:  
for (;;) {  
for (Node<E> h = head, p = h, q;;) {  
E item = p.item;  
  
if (item != null && p.casItem(item, null)) {  
// Successful CAS is the linearization point  
// for item to be removed from this queue.  
if (p != h) // hop two nodes at a time  
updateHead(h, ((q = p.next) != null) ? q : p);  
return item;  
}  
else if ((q = p.next) == null) {  
updateHead(h, p);  
return null;  
}  
else if (p == q)  
continue restartFromHead;  
else  
p = q;  
}  
}  
}
```


对于offer和poll方法，他们被称作延迟更新的主要原因是，修改节点的操作已经可见，但是head和tail并没有**同时更新**,具体来说就是以下几个操作方法。

```
boolean casItem(E cmp, E val) { 
return UNSAFE.compareAndSwapObject(this, itemOffset, cmp, val);  
}
boolean casNext(Node<E> cmp, Node<E> val) {  
return UNSAFE.compareAndSwapObject(this, nextOffset, cmp, val);  
}
private boolean casTail(Node<E> cmp, Node<E> val) {  
return UNSAFE.compareAndSwapObject(this, tailOffset, cmp, val);  
}  
  
private boolean casHead(Node<E> cmp, Node<E> val) {  
return UNSAFE.compareAndSwapObject(this, headOffset, cmp, val);  
}
```


而上述四个更新节点的方法中除了cas即时更新值得注意意外，还有偏移量的使用，
```
itemOffset = UNSAFE.objectFieldOffset  
(k.getDeclaredField("item"));  
nextOffset = UNSAFE.objectFieldOffset  
(k.getDeclaredField("next"));
```


对于单纯的cas方法来说是可能失败的，具体来说：存在节点更新成功，而头尾节更新失败的情况，但是由于cas操作节点过程中会更新偏移量，所以偏移量是一定正确，因而可以保证其最终一致。



# 线程任务
线程任务（run/call方法）定义：

|            | 有无返回值 |
| ---------- | ----- |
| Runnable接口 | ✖️    |
| Callable接口 | ✔️    |
| 继承Thread   | ✖️    |


实现接口 VS 继承 Thread：实现接口会更好一些，因为:
Java 不支持多重继承，因此继承了 Thread 类就无法继承其它类，但是可以实现多个接口；
类可能只要求可执行就行，继承整个 Thread 类开销过大



线程启动，：
- Thread.start()
- 创建线程池，使用submit（有返回值）/execute（无返回值）


## 任务拆分
### fork/Join
Fork/Join框架是Java并发工具包中的一种可以将一个大任务拆分为很多小任务来异步执行的工具.

主要包含三个模块:
- 任务对象: ForkJoinTask 
- 执行Fork/Join任务的线程: ForkJoinWorkerThread
- 线程池: ForkJoinPool


1，传入的Runnable、Callable任务接口的对象最终都会被转换为ForkJoinTask任务对象进行执行。
**而对于ForkJoinTask接口，一般我们继承它的三个子类来完成业务：RecursiveTask 、RecursiveAction 或 CountedCompleter**

RecursiveTask ：可以执行递归，（实际上是支持fork提交子任务）
RecursiveAction：无返回值的 RecursiveTask
CountedCompleter：任务完成执行后会触发执行一个自定义的钩子函数

### CompletableFuture



1，`CompletableFuture<Object> objectCompletableFuture = new CompletableFuture<>();`
得到了一个可以通知的结果是否完成的载体，
```
CompletableFuture<Object> objectCompletableFuture = new CompletableFuture<>();  
objectCompletableFuture.complete(new Object());//提交结果  
objectCompletableFuture.isCancelled();//任务是否被取消  
objectCompletableFuture.isDone();//是否提交成功  
objectCompletableFuture.get();//阻塞获取结果
```

2,创建有结果的CompletableFuture，（大概是为了规范化提交任务结果）
```
CompletableFuture<String> stringCompletableFuture = CompletableFuture.completedFuture("123");  
stringCompletableFuture.get();
```


#### 封装计算逻辑


**在对象调用到这些方法时，内部会自动提交到内部的线程池执行**
```
static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier);
// 使用自定义线程池(推荐)
static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier, Executor executor);
static CompletableFuture<Void> runAsync(Runnable runnable);
// 使用自定义线程池(推荐)
static CompletableFuture<Void> runAsync(Runnable runnable, Executor executor);
```

#### 多任务编排
不同于join任务在任务逻辑中提交任务，而是使用**流式调用**的方式来使用上一个任务的结果或者定义任务之间的关系。

[CompletableFuture 详解 | JavaGuide](https://javaguide.cn/java/concurrent/completablefuture-intro.html#%E5%A4%84%E7%90%86%E5%BC%82%E6%AD%A5%E7%BB%93%E7%AE%97%E7%9A%84%E7%BB%93%E6%9E%9C)


```

thenCompose链接下一个任务的执行
CompletableFuture<String> future
        = CompletableFuture.supplyAsync(() -> "hello!")
        .thenCompose(s -> CompletableFuture.supplyAsync(() -> s + "world!"));

```


# AOS阻塞多个线程实现
## CountDownLatch

基于AQS提供计数器**阻塞某些线程**功能，

重要方法：
```
public final void acquireSharedInterruptibly(int arg)  
throws InterruptedException {  
if (Thread.interrupted())  
throw new InterruptedException();  
if (tryAcquireShared(arg) < 0)  //是否阻塞
doAcquireSharedInterruptibly(arg);  //加入等待队列等操作，
}
```


**acquireSharedInterruptibly方法是使用for (;;) 的方法实现的自旋。**



```
public final boolean releaseShared(int arg) {  
if (tryReleaseShared(arg)) {  //获取锁，
doReleaseShared();  //释放锁
return true;  
}  
return false;  
}
```

## CyclicBarrier
实现**阻塞一定数量的线程**后执行，换言之，当一定数量的线程完成任务后才能执行任务。
核心参数为：
- parties：阻塞数量
- barrierCommand：执行动作

核心方法：
```
private int dowait(boolean timed, long nanos)  
throws InterruptedException, BrokenBarrierException,  
TimeoutException {  
final ReentrantLock lock = this.lock;  
lock.lock();  
try {  
final Generation g = generation;  
  
if (g.broken)  
throw new BrokenBarrierException();  
  
if (Thread.interrupted()) {  
breakBarrier();  
throw new InterruptedException();  
}  
  ==============阻塞核心逻辑=======================================
int index = --count; 
if (index == 0) { 
boolean ranAction = false;  
try {  
final Runnable command = barrierCommand;  
if (command != null)  
command.run();  
ranAction = true;  
nextGeneration();  //重置阻塞参数。
return 0;  
} finally {  
if (!ranAction)  
breakBarrier();  
}  
}  
================================================================
  
// loop until tripped, broken, interrupted, or timed out  
for (;;) {  //自旋等待
try {  
if (!timed)  
trip.await();  
else if (nanos > 0L)  
nanos = trip.awaitNanos(nanos);  
} catch (InterruptedException ie) {  
if (g == generation && ! g.broken) {  
breakBarrier();  
throw ie;  
} else {  
// We're about to finish waiting even if we had not  
// been interrupted, so this interrupt is deemed to  
// "belong" to subsequent execution.  
Thread.currentThread().interrupt();  
}  
}  
  
if (g.broken)  
throw new BrokenBarrierException();  
  
if (g != generation)  
return index;  
  
if (timed && nanos <= 0L) {  
breakBarrier();  
throw new TimeoutException();  
}  
}  
} finally {  
lock.unlock();  
}  
}
```


```
private void nextGeneration() {  
// signal completion of last generation  
trip.signalAll();  //唤醒（自旋）等待的线程
// set up next generation  
count = parties;  
generation = new Generation();  
}
```



##  CyclicBarrier和CountDonwLatch对比
CountDownLatch减计数，CyclicBarrier加计数。
CountDownLatch是一次性的，CyclicBarrier可以重用。
CountDownLatch和CyclicBarrier都有让多个线程等待同步然后再开始下一步动作的意思，但是CountDownLatch的下一步的动作实施者是主线程，具有不可重复性；而CyclicBarrier的下一步动作实施者还是“其他线程”本身，具有往复多次实施动作的特点



## Semapho




## Exchanger

实现两个线程之间严格交换数据。
（暂不学习，原因：双线程场景太单调了，一般是多线程场景吧？）

多线程交换数据可以使用
1，队列（MQ、延迟和同步队列等）（队列本身实现了严格的安全交换）
2，CyclicBarrier屏障任务（任务本身作为传输数据的方法，严格交换也可以在这里定义）
3，加锁实现，（对交换数据的变量加锁，好像更简单也更轻量？）


# ThreadLocal
实现多线程变量，基于线程对不同的变量副本操作。
## ThreadLocal造成内存泄露的问题

ThreadLocal内部使用了ThreadLocalMap存储数据，而ThreadLocalMap并非继承map使用node节点作为<k,v>存储，而是使用自定义的类型`static class Entry extends WeakReference<ThreadLocal<?>>`，
```
static class Entry extends WeakReference<ThreadLocal<?>> {  
/** The value associated with this ThreadLocal. */  
Object value;  
  
Entry(ThreadLocal<?> k, Object v) {  
super(k);  //真正成为弱引用的是ThreadLocal对象。
value = v;  
}  
}
```


因此key有了两个引用。
1，弱引用：WeakReference<ThreadLocal< ? >>追溯到Reference类型。
（该弱引用是在ThreadLocal.set()中由于生成Entry对象产生的。）
```
Reference(T referent, ReferenceQueue<? super T> queue) {  
this.referent = referent;  //引用，在jvm中被视为弱引用，
this.queue = (queue == null) ? ReferenceQueue.NULL : queue;  
}
```

2.强引用：声明ThreadLocal变量，（这是必要存在的，如果不声明，程序就无法调用获得值）



> [!NOTE] WeakReference弱引用
> 生命周期：被弱引用关联的对象只能生存到下一次垃圾收集发生为止，当垃圾收集器开始工作，无论当前内存是否足够，都会回收掉只被弱引用关联的对象

因此内存泄漏的本质是无法清除数据的问题，由于ThreadLocal的外部强引用被gc时，线程的成员变量map无法自动检测删除对应的键值对。


最终解决方式: 将map当中的key变成弱引用封装在Entry中，当外部强引用被抹除后，最终会随着gc次数的增加最终被回收。

此时问题就变成如何清理<null,key>？对此ThreadLocal做出了以下举措。
1，get（）没有命中时可能会触发清理，

```
private Entry getEntry(ThreadLocal<?> key) {  
int i = key.threadLocalHashCode & (table.length - 1);  
Entry e = table[i];  
if (e != null && e.get() == key)  
return e;  
else  
return getEntryAfterMiss(key, i, e);  //遍历查找时会判断是否被回收，如果被回收则顺带清理。
}
```
2，调用set()方法时，采样清理、全量清理，扩容时还会继续检查。
3.调用remove()时，除了清理当前Entry，还会向后继续清理。

无论是哪种方式，实际执行清理的都是expungeStaleEntry()方法
```
private int expungeStaleEntry(int staleSlot) {  
Entry[] tab = table;  
int len = tab.length;  
  
// expunge entry at staleSlot  
tab[staleSlot].value = null;  
tab[staleSlot] = null;  
size--;  //清理当前已经查找到的
  
// Rehash until we encounter null  
Entry e;  
int i;  
for (i = nextIndex(staleSlot, len);  //向后查找是否存在被清理掉的
(e = tab[i]) != null;  
i = nextIndex(i, len)) {  
ThreadLocal<?> k = e.get();  
if (k == null) {  
e.value = null;  
tab[i] = null;  
size--;  
} else {  
int h = k.threadLocalHashCode & (len - 1);  
if (h != i) {  
tab[i] = null;  
  
// Unlike Knuth 6.4 Algorithm R, we must scan until  
// null because multiple entries could have been stale.  
while (tab[h] != null)  
h = nextIndex(h, len);  
tab[h] = e;  
}  
}  
}  
return i;  
}
```


**虽然 `ThreadLocalMap` 在 `get()`, `set()` 和 `remove()` 操作时会尝试清理 key 为 null 的 entry，但这种清理机制是被动的，并不完全可靠。**


## ThreadlocalMap和thradlocal的关系，如何实现多线程变量
ThreadLocal在在set、get等方法实际操作的是Threadlocalmap，
以set方法为例。
```
public void set(T value) {  
Thread t = Thread.currentThread();  
ThreadLocalMap map = getMap(t);  
if (map != null) {  
map.set(this, value);  
} else {  
createMap(t, value);  
}  
}
```

getMap中`return t.threadLocals;`类型为`ThreadLocal.ThreadLocalMap`这是一个线程级别的变量，这也是多线程变量副本的真相。
而map.set则是通过Threadlocal的hash值进行离散计算得到坐标，生成Entry并存储table数组中的。
`int i = key.threadLocalHashCode & (len-1);`

而这个Entry变量的类型是弱引用类型，
```
tab[i] = new Entry(key, value);//存储Entry

static class Entry extends WeakReference<ThreadLocal<?>> {  
/** The value associated with this ThreadLocal. */  
Object value;  
  
Entry(ThreadLocal<?> k, Object v) {  
super(k);  //封装弱引用
value = v;  //附带值
}  
}
```

总之在Entry中存储了key和value，只是将key作为弱引用。













