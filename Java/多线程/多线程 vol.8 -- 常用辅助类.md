## 9、常用的辅助类(必会)

### 9.1、CountDownLatch

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210127130629334.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzU5MTk4MA==,size_16,color_FFFFFF,t_70)

**减法计数器： 实现调用几次线程后 再触发某一个任务**

代码举例：

```java
import java.util.concurrent.CountDownLatch;
// 计数器
public class CountDownLatchDemo {
    public static void main(String[] args) throws InterruptedException {
        // 总数是6，必须要执行任务的时候，再使用！
        CountDownLatch countDownLatch = new CountDownLatch(6);
        for (int i = 1; i <=6 ; i++) {
            new Thread(()->{
                System.out.println(Thread.currentThread().getName()
                                                           +" Go out");
                countDownLatch.countDown(); // 数量-1
            },String.valueOf(i)).start();
        }
        countDownLatch.await(); // 等待计数器归零，然后再向下执行
        System.out.println("Close Door");
    }
}
```

原理：

`countDownLatch.countDown();` // 数量-1

`countDownLatch.await();` // 等待计数器归零，然后再向下执行

每次有线程调用 **countDown**() 数量-1，假设计数器变为0，**countDownLatch.await**() 就会被唤醒，继续执行！

### 9.2、CyclicBarrier

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210127130643629.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzU5MTk4MA==,size_16,color_FFFFFF,t_70)

**加法计数器**：**集齐7颗龙珠召唤神龙**

代码举例：

```java
import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;
public class  CyclicBarrierDemo {
    public static void main(String[] args) {
        /*
         * 集齐7颗龙珠召唤神龙
         */
        // 召唤龙珠的线程
        CyclicBarrier cyclicBarrier = new CyclicBarrier(7,()->{
            System.out.println("召唤神龙成功！");
        });
        for (int i = 1; i <=7 ; i++) {
            final int temp = i;
            // lambda能操作到 i 吗
            new Thread(()->{
                System.out.println(Thread.currentThread().getName()
                                                 +"收集"+temp+"个龙珠");
                try {
                    cyclicBarrier.await(); // 等待
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (BrokenBarrierException e) {
                    e.printStackTrace();
                }
            }).start();
        }
    }
}
```

### 9.3、Semaphore

Semaphore：信号量

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210127130706363.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzU5MTk4MA==,size_16,color_FFFFFF,t_70)

**限流/抢车位！6车—3个停车位置**

代码举例：

```java
import java.util.concurrent.Semaphore;
import java.util.concurrent.TimeUnit;
public class SemaphoreDemo {
    public static void main(String[] args) {
        // 线程数量：停车位! 限流！、
        // 如果已有3个线程执行（3个车位已满），则其他线程需要等待‘车位’释放后，才能执行！
        Semaphore semaphore = new Semaphore(3);
        for (int i = 1; i <=6 ; i++) {
            new Thread(()->{
                // acquire() 得到
                try {
                    semaphore.acquire();
                    System.out.println(Thread.currentThread()
                                               .getName()+"抢到车位");
                    TimeUnit.SECONDS.sleep(2);
                    System.out.println(Thread.currentThread()
                                               .getName()+"离开车位");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    semaphore.release(); // release() 释放 
                }
            },String.valueOf(i)).start();
        }
    }
}
```

只有三个车位，只有当某辆车离开车位，车位空出来后，下一辆车才能在此停放。

输出结果如图:

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210127130723988.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzU5MTk4MA==,size_16,color_FFFFFF,t_70)

**原理：**

`semaphore.acquire();` 获得，假设如果已经满了，等待，等待被释放为止！`semaphore.release();` 释放，会将当前的信号量释放 + 1，然后唤醒等待的线程！

作用： 多个共享资源互斥的使用！并发限流，控制最大的线程数！

## 10、读写锁 ReadWriteLock

> ReadWriteLock

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210127130738758.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzU5MTk4MA==,size_16,color_FFFFFF,t_70)

代码举例：

```java
import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReadWriteLock;
import java.util.concurrent.locks.ReentrantLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;
/**
 * 独占锁（写锁） 一次只能被一个线程占有
 * 共享锁（读锁） 多个线程可以同时占有
 * ReadWriteLock
 * 读-读  可以共存！
 * 读-写  不能共存！
 * 写-写  不能共存！
 */
public class ReadWriteLockDemo {
    public static void main(String[] args) {
        //MyCache myCache = new MyCache();
        MyCacheLock myCacheLock = new MyCacheLock();
        // 写入
        for (int i = 1; i <= 5 ; i++) {
            final int temp = i;
            new Thread(()->{
                myCacheLock.put(temp+"",temp+"");
            },String.valueOf(i)).start();
        }
        // 读取
        for (int i = 1; i <= 5 ; i++) {
            final int temp = i;
            new Thread(()->{
                myCacheLock.get(temp+"");
            },String.valueOf(i)).start();
        }
    }
}
/**
 * 自定义缓存
 * 加锁的
 */
class MyCacheLock{
    private volatile Map<String,Object> map = new HashMap<>();
    // 读写锁： 更加细粒度的控制
    private ReadWriteLock readWriteLock = new             
                                    ReentrantReadWriteLock();
    // private Lock lock = new ReentrantLock();
    // 存，写入的时候，只希望同时只有一个线程写
    public void put(String key,Object value){
        readWriteLock.writeLock().lock();
        try {
            System.out.println(Thread.currentThread().getName()
                                                       +"写入"+key);
            map.put(key,value);
            System.out.println(Thread.currentThread().getName()
                                                       +"写入OK");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            readWriteLock.writeLock().unlock();
        }
    }
    // 取，读，所有人都可以读！
    public void get(String key){
        readWriteLock.readLock().lock();
        try {
            System.out.println(Thread.currentThread().getName()
                                                       +"读取"+key);
            Object o = map.get(key);
            System.out.println(Thread.currentThread().getName()
                                                       +"读取OK");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            readWriteLock.readLock().unlock();
        }
    }
}
/**
 * 自定义缓存
 * 不加锁的
 */
class MyCache{
    private volatile Map<String,Object> map = new HashMap<>();
    // 存，写
    public void put(String key,Object value){
        System.out.println(Thread.currentThread().getName()
                                                       +"写入"+key);
        map.put(key,value);
        System.out.println(Thread.currentThread().getName()
                                                       +"写入OK");
    }
    // 取，读
    public void get(String key){
        System.out.println(Thread.currentThread().getName()
                                                       +"读取"+key);
        Object o = map.get(key);
        System.out.println(Thread.currentThread().getName()
                                                       +"读取OK");
    }
}
```

执行效果如图：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210127130754967.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzU5MTk4MA==,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210127130805518.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzU5MTk4MA==,size_16,color_FFFFFF,t_70)

## 11、阻塞队列

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210127130819260.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzU5MTk4MA==,size_16,color_FFFFFF,t_70)

**阻塞队列：**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210127130828879.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzU5MTk4MA==,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210127130843664.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzU5MTk4MA==,size_16,color_FFFFFF,t_70)

#### BlockingQueue

BlockingQueue 不是新的东西

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210127130855689.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzU5MTk4MA==,size_16,color_FFFFFF,t_70)

什么情况下我们会使用 阻塞队列?：多线程并发处理，线程池用的较多 ！

**学会使用队列**

添加、移除

**四组API**

```java
| 方式     | 抛出异常    | 有返回值，不抛出异常 | 阻塞 等待  | 超时等待     |
| ------ | ------- | ---------- | ------ | -------- |
| 添加     | add     | offer()    | put()  | offer(,) |
| 移除     | remove  | poll()     | take() | poll(,)  |
| 检测队首元素 | element | peek()     | -      | -        |
```

代码示例：

```java
package com.kuang.bq;
import java.util.Collection;
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.TimeUnit;
public class Test {
    public static void main(String[] args) throws InterruptedException {
        test4();
    }
    /**
     * 1. 无返回值，抛出异常的方式
     */
    public static void test1(){
        // 队列的大小
        ArrayBlockingQueue blockingQueue = 
                                    new ArrayBlockingQueue<>(3);
        System.out.println(blockingQueue.add("a"));// true
        System.out.println(blockingQueue.add("b"));// true
        System.out.println(blockingQueue.add("c"));// true
        // System.out.println(blockingQueue.add("d"));
        // IllegalStateException: Queue full 抛出异常---队列已满！
        System.out.println("===========================");
        System.out.println(blockingQueue.element());//
        // 查看队首元素是谁
        System.out.println(blockingQueue.remove());//
        System.out.println(blockingQueue.remove());//
        System.out.println(blockingQueue.remove());//
        // System.out.println(blockingQueue.remove());
        // java.util.NoSuchElementException 抛出异常---队列已为空！
    }
    /**
     * 2. 有返回值，不抛出异常的方式
     */
    public static void test2(){
        // 队列的大小
        ArrayBlockingQueue blockingQueue = 
                                    new ArrayBlockingQueue<>(3);
        System.out.println(blockingQueue.offer("a"));
        System.out.println(blockingQueue.offer("b"));
        System.out.println(blockingQueue.offer("c"));
        System.out.println(blockingQueue.peek());
        // System.out.println(blockingQueue.offer("d")); 
        // false 不抛出异常！
        System.out.println("===========================");
        System.out.println(blockingQueue.poll());
        System.out.println(blockingQueue.poll());
        System.out.println(blockingQueue.poll());
        System.out.println(blockingQueue.poll()); 
        // null  不抛出异常！
    }
    /**
     * 3. 等待，阻塞（一直阻塞）
     */
    public static void test3() throws InterruptedException {
        // 队列的大小
        ArrayBlockingQueue blockingQueue = 
                                    new ArrayBlockingQueue<>(3);
        // 一直阻塞
        blockingQueue.put("a");
        blockingQueue.put("b");
        blockingQueue.put("c");
        // blockingQueue.put("d"); // 队列没有位置了，一直阻塞等待
        System.out.println(blockingQueue.take());
        System.out.println(blockingQueue.take());
        System.out.println(blockingQueue.take());
        System.out.println(blockingQueue.take()); 
        // 没有这个元素，一直阻塞等待
    }
    /**
     * 4. 等待，阻塞（等待超时）
     */
    public static void test4() throws InterruptedException {
        // 队列的大小
        ArrayBlockingQueue blockingQueue = 
                                    new ArrayBlockingQueue<>(3);
        blockingQueue.offer("a");
        blockingQueue.offer("b");
        blockingQueue.offer("c");
        // blockingQueue.offer("d",2,TimeUnit.SECONDS); 
        // 等待超过2秒就退出
        System.out.println("===============");
        System.out.println(blockingQueue.poll());
        System.out.println(blockingQueue.poll());
        System.out.println(blockingQueue.poll());
        blockingQueue.poll(2,TimeUnit.SECONDS); // 等待超过2秒就退出
    }
}
```

#### SynchronousQueue

> SynchronousQueue 同步队列

**没有容量，进去一个元素，必须等待取出来之后，才能再往里面放一个元素！**

put、take

代码举例：

```java
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.SynchronousQueue;
import java.util.concurrent.TimeUnit;
/**
 * 同步队列:
 * 和其他的BlockingQueue 不一样， SynchronousQueue 不存储元素
 * put了一个元素，必须从里面先take取出来，否则不能在put进去值！
 */
public class SynchronousQueueDemo {
    public static void main(String[] args) {
        BlockingQueue<String> blockingQueue = 
                                new SynchronousQueue<>(); // 同步队列
        new Thread(()->{
            try {
                System.out.println(Thread.currentThread().getName()
                                                           +" put 1");
                // put进入一个元素
                blockingQueue.put("1");
                System.out.println(Thread.currentThread().getName()
                                                           +" put 2");
                blockingQueue.put("2");
                System.out.println(Thread.currentThread().getName()
                                                           +" put 3");
                blockingQueue.put("3");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        },"T1").start();
        new Thread(()->{
            try {
                // 睡眠3s取出一个元素
                TimeUnit.SECONDS.sleep(3);
                System.out.println(Thread.currentThread().getName()
                                           +"=>"+blockingQueue.take());
                TimeUnit.SECONDS.sleep(3);
                System.out.println(Thread.currentThread().getName()
                                           +"=>"+blockingQueue.take());
                TimeUnit.SECONDS.sleep(3);
                System.out.println(Thread.currentThread().getName()
                                           +"=>"+blockingQueue.take());
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        },"T2").start();
    }
}
```

执行结果如图所示：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210127130914527.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzU5MTk4MA==,size_16,color_FFFFFF,t_70)