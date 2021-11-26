## 12、线程池(重点)

**线程池：3大方法、7大参数、4种拒绝策略**

> 池化技术

程序的运行，本质：占用系统的资源！ （优化资源的使用 => 池化技术）

线程池、连接池、内存池、对象池///… 创建、销毁。十分浪费资源

池化技术：事先准备好一些资源，有人要用，就来我这里拿，用完之后还给我。

> 线程池的好处:

- 1、降低系统资源的消耗
- 2、提高响应的速度
- 3、方便管理

**线程复用、可以控制最大并发数、管理线程**

#### 线程池：3大方法

> 线程池：3大方法

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210127130928587.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzU5MTk4MA==,size_16,color_FFFFFF,t_70)

示例代码：

```java
import java.util.concurrent.ExecutorService;
import java.util.List;
import java.util.concurrent.Executors;
public class Demo01 {
    public static void main(String[] args) {
        // Executors 工具类、3大方法
        // Executors.newSingleThreadExecutor();// 创建单个线程的线程池
        // Executors.newFixedThreadPool(5);// 创建一个固定大小的线程池
        // Executors.newCachedThreadPool();// 创建一个可伸缩的线程池
        // 单个线程的线程池
        ExecutorService threadPool =     
                                Executors.newSingleThreadExecutor();
        try {
            for (int i = 1; i < 100; i++) {
                // 使用了线程池之后，使用线程池来创建线程
                threadPool.execute(()->{
                    System.out.println(
                        Thread.currentThread().getName()+" ok");
                });
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            // 线程池用完，程序结束，关闭线程池
            threadPool.shutdown();
        }
    }
}
```

#### 线程池：7大参数

> 7大参数

源码分析：

```java
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService (
        new ThreadPoolExecutor(
            1, 
            1,
            0L, 
            TimeUnit.MILLISECONDS, 
            new LinkedBlockingQueue<Runnable>())); 
}
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(
        5, 
        5, 
        0L, 
        TimeUnit.MILLISECONDS, 
        new LinkedBlockingQueue<Runnable>()); 
}
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(
        0, 
        Integer.MAX_VALUE, 
        60L, 
        TimeUnit.SECONDS, 
        new SynchronousQueue<Runnable>()); 
}
// 本质ThreadPoolExecutor（） 
public ThreadPoolExecutor(int corePoolSize, // 核心线程池大小 
                          int maximumPoolSize, // 最大核心线程池大小 
                          long keepAliveTime, // 超时没有人调用就会释放 
                          TimeUnit unit, // 超时单位 
                          // 阻塞队列 
                          BlockingQueue<Runnable> workQueue, 
                          // 线程工厂：创建线程的，一般 不用动
                          ThreadFactory threadFactory,  
                          // 拒绝策略
                          RejectedExecutionHandler handle ) {
    if (corePoolSize < 0 
        || maximumPoolSize <= 0 
        || maximumPoolSize < corePoolSize 
        || keepAliveTime < 0) 
        throw new IllegalArgumentException(); 
    if (workQueue == null 
        || threadFactory == null 
        || handler == null) 
        throw new NullPointerException(); 
    this.acc = System.getSecurityManager() == null 
        ? null : AccessController.getContext(); 
    this.corePoolSize = corePoolSize; 
    this.maximumPoolSize = maximumPoolSize; 
    this.workQueue = workQueue; 
    this.keepAliveTime = unit.toNanos(keepAliveTime); 
    this.threadFactory = threadFactory; 
    this.handler = handler; 
}
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210127130942843.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzU5MTk4MA==,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210127130957541.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzU5MTk4MA==,size_16,color_FFFFFF,t_70)

> 手动创建一个线程池

因为实际开发中工具类**Executors** 不安全，所以需要手动创建线程池，自定义7个参数。

示例代码：

```java
package com.haust.pool;
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.*;
// Executors 工具类、3大方法
// Executors.newSingleThreadExecutor();// 创建一个单个线程的线程池
// Executors.newFixedThreadPool(5);// 创建一个固定大小的线程池
// Executors.newCachedThreadPool();// 创建一个可伸缩的线程池
/**
 * 四种拒绝策略：
 *
 * new ThreadPoolExecutor.AbortPolicy() 
 * 银行满了，还有人进来，不处理这个人的，抛出异常
 *
 * new ThreadPoolExecutor.CallerRunsPolicy() 
 * 哪来的去哪里！比如你爸爸 让你去通知妈妈洗衣服，妈妈拒绝，让你回去通知爸爸洗
 *
 * new ThreadPoolExecutor.DiscardPolicy() 
 * 队列满了，丢掉任务，不会抛出异常！
 *
 * new ThreadPoolExecutor.DiscardOldestPolicy() 
 * 队列满了，尝试去和最早的竞争，也不会抛出异常！
 */
public class Demo01 {
    public static void main(String[] args) {
        // 自定义线程池！工作 ThreadPoolExecutor
        ExecutorService threadPool = new ThreadPoolExecutor(
                2,// int corePoolSize, 核心线程池大小(候客区窗口2个)
                5,// int maximumPoolSize, 最大核心线程池大小(总共5个窗口) 
                3,// long keepAliveTime, 超时3秒没有人调用就会释，放关闭窗口 
                TimeUnit.SECONDS,// TimeUnit unit, 超时单位 秒 
                new LinkedBlockingDeque<>(3),// 阻塞队列(候客区最多3人)
                Executors.defaultThreadFactory(),// 默认线程工厂
                // 4种拒绝策略之一：
                // 队列满了，尝试去和 最早的竞争，也不会抛出异常！
                new ThreadPoolExecutor.DiscardOldestPolicy());  
        //队列满了，尝试去和最早的竞争，也不会抛出异常！
        try {
            // 最大承载：Deque + max
            // 超过 RejectedExecutionException
            for (int i = 1; i <= 9; i++) {
                // 使用了线程池之后，使用线程池来创建线程
                threadPool.execute(()->{
                    System.out.println(
                        Thread.currentThread().getName()+" ok");
                });
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            // 线程池用完，程序结束，关闭线程池
            threadPool.shutdown();
        }
    }
}
```

#### 线程池：4种拒绝策略

> 4种拒绝策略

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210127131011428.png)

```java
/**
 * 四种拒绝策略：
 *
 * new ThreadPoolExecutor.AbortPolicy() 
 * 银行满了，还有人进来，不处理这个人的，抛出异常
 *
 * new ThreadPoolExecutor.CallerRunsPolicy() 
 * 哪来的去哪里！比如你爸爸 让你去通知妈妈洗衣服，妈妈拒绝，让你回去通知爸爸洗
 *
 * new ThreadPoolExecutor.DiscardPolicy() 
 * 队列满了，丢掉任务，不会抛出异常！
 *
 * new ThreadPoolExecutor.DiscardOldestPolicy() 
 * 队列满了，尝试去和最早的竞争，也不会抛出异常！
 */
```

> 小结和拓展

池的最大容量如何去设置！

了解：**IO密集型，CPU密集型：（调优）**

直接上代码：

```java
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.*;
public class Demo01 {
    public static void main(String[] args) {
        // 自定义线程池！工作 ThreadPoolExecutor
        // 最大线程到底该如何定义
        // 1、CPU 密集型，几核，就是几，可以保持CPu的效率最高！ 
        // 2、IO 密集型 > 判断你程序中十分耗IO的线程， 
        // 比如程序 15个大型任务 io十分占用资源！
        // IO密集型参数(最大线程数)就设置为大于15即可，一般选择两倍
        // 获取CPU的核数
        System.out.println(
            Runtime.getRuntime().availableProcessors());// 8核
        ExecutorService threadPool = new ThreadPoolExecutor(
                2,// int corePoolSize, 核心线程池大小
                // int maximumPoolSize, 最大核心线程池大小 8核电脑就是8
                Runtime.getRuntime().availableProcessors(),
                3,// long keepAliveTime, 超时3秒没有人调用就会释放
                TimeUnit.SECONDS,// TimeUnit unit, 超时单位 秒 
                new LinkedBlockingDeque<>(3),// 阻塞队列(候客区最多3人)
                Executors.defaultThreadFactory(),// 默认线程工厂
                // 4种拒绝策略之一：
                // 队列满了，尝试去和 最早的竞争，也不会抛出异常！
                new ThreadPoolExecutor.DiscardOldestPolicy());  
        //队列满了，尝试去和最早的竞争，也不会抛出异常！
        try {
            // 最大承载：Deque + max
            // 超过 RejectedExecutionException
            for (int i = 1; i <= 9; i++) {
                // 使用了线程池之后，使用线程池来创建线程
                threadPool.execute(()->{
                    System.out.println(
                        Thread.currentThread().getName()+" ok");
                });
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            // 线程池用完，程序结束，关闭线程池
            threadPool.shutdown();
        }
    }
}
```