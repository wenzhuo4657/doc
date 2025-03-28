
参考文章：[JAVA并发编程——线程池原理、使用和参数设置详解 - 个人文章 - SegmentFault 思否](https://segmentfault.com/a/1190000040061529)






传统的new Thread()在实际中是不会直接使用的，它耗费资源不受限制、且过多线程会造成死锁等严重错误，并且不如线程池有更多的功能，例如**定时执行、定期执行、线程中断**等等




# Executor：JAVA自带的创建线程池的工具类



 创建方式
（1）ExecutorService threadPool = Executors.newFixedThreadPool(5);//一池固定数线程
（2）ExecutorService threadPoo2 = Executors.newCachedThreadPool();//一池多线程
（3）ExecutorService threadPoo3 = Executors.newSingleThreadExecutor();//一池一线程
其中还有许多重载方式

**固定数线程池最大和常驻线程数都是传入的参数**  
而cache线程池的最大线程数是**Integer.MAX_VALUE**  
单线程池底层使用了**SynchronousQueue** : 不存储元素的阻塞队列，即单个元素的队列。  
多线程池的底层使用**了LinkedBlockingQueue** : 由链表组成的有界（**但是默认值为Integer.MAX_VALUE阻塞队列**）

但还是不够灵活，实际工作中我们通常使用的是自定义线程池的方式

```
  public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              RejectedExecutionHandler handler) {
        this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
             Executors.defaultThreadFactory(), handler);
    }
```

参数详解

| param               | des                                      |
| ------------------- | ---------------------------------------- |
| **corePoolSize**    | 线程池中**常驻核心线程数**                          |
| **maximumPoolSize** | 线程池能够容纳的同时执行的**最大线程数**，此值必须大于等于1         |
| **keppAliveTime**   | **多余**的空闲线程的**存活时间**                     |
| **unit**            | 存活时间**单位**                               |
| **workQueue**       | 任务队列，**被提交但是尚未执行的任务**                    |
| **threadFactory**   | 表示生成线程池中工作线程的线程工厂，**用于创建线程，一般默认即可**      |
| handler             | 拒绝策略，表示**队列满了并且工作线程大于等于线程池的最大线程数，如何处理。** |



handler  ：只有当核心线程、阻塞队列都满了才会调用，常见的有
**AbortPolicy**(默认)：直接抛出RejectExcutionException异常阻止线程运行。  
**CallerRunsPolicy**:将多出来的任务**退还给调用者**，从而降低流量。

**DiscardOldestPolicy**:**抛弃等待队列中等待最久的任务**，然后把当前任务加入等待队列。

**DiscardPolicy**:**直接丢弃任务**。




**如何合理设置线程池参数**  
如何配置线程池的参数，其实我们最需要配置的参数只有一个：  
corePoolSize 线程池中常驻核心线程数

当我们的线程池使用场景是在**CPU密集型（进程绝大部份任务依靠cpu的计算能力完成）**，那么为了防止上下文的切换造成的开销，我们一般设置**最大线程数等于CPU核数**（核数获取方法:Runtime.getRuntime().availableProcessors();）

当我们的线程池使用场景是在**IO密集型****(绝大部分任务就是在读入，输出数据)**,那么cpu不是非常非常繁忙，大部分在等待的情况下，**最大线程可以是CPU核数的两倍。**、




使用实例：
```
     ThreadPoolExecutor threadPoolExecutor
                = new ThreadPoolExecutor(Runtime.getRuntime().availableProcessors(),//常驻核心线程数
                5,//最大核心线程数
                1L,//多余的空闲线程的存活时间
                TimeUnit.SECONDS,//多余空闲线程存活时间单位
                new ArrayBlockingQueue<Runnable>(10),//阻塞队列
                Executors.defaultThreadFactory(),//生成线程池中工作线程的线程工厂
                new ThreadPoolExecutor.AbortPolicy());//拒绝策略

        //模拟多个用户来办理业务，每一个用户就是来自外部的请求线程
        try {
            for (int i = 1; i < 10000; i++) {
                threadPoolExecutor.submit(() -> {
                    System.out.println(Thread.currentThread().getName() + "正在办理业务");
                });
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            threadPoolExecutor.shutdown();
        }
    }
```


# ScheduledThreadPoolExecutor
ScheduledThreadPoolExecutor继承自 ThreadPoolExecutor，为任务**提供延迟或周期执行**。
