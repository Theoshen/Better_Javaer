## 19、深入理解CAS

> 什么是 CAS

```java
import java.util.concurrent.atomic.AtomicInteger;
public class CASDemo {
    // CAS compareAndSet : 比较并交换！ 
    public static void main(String[] args) {
        AtomicInteger atomicInteger = new AtomicInteger(2020); 
        // 期望、更新 
        // public final boolean compareAndSet
        //                                    (int expect, int update) 
        // 如果我期望的值达到了，那么就更新，否则，
        // 就不更新, CAS 是CPU的并发原语！ 
        System.out.println(atomicInteger.compareAndSet(2020, 2021)); 
        System.out.println(atomicInteger.get()); 
        atomicInteger.getAndIncrement() // 看底层如何实现 ++ 
        System.out.println(atomicInteger.compareAndSet(2020, 2021)); 
        System.out.println(atomicInteger.get()); 
    } 
}
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210130191605818.png)

> Unsafe 类

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210130191615435.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzU5MTk4MA==,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210130191629917.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzU5MTk4MA==,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/2021013019164533.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzU5MTk4MA==,size_16,color_FFFFFF,t_70)

CAS ： 比较当前工作内存中的值和主内存中的值，如果这个值是期望的，那么则执行操作！如果不是就

一直循环！

> CAS ： ABA 问题（狸猫换太子）

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210130191656733.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzU5MTk4MA==,size_16,color_FFFFFF,t_70)

```java
import java.util.concurrent.atomic.AtomicInteger; 
public class CASDemo {
    // CAS compareAndSet : 比较并交换！ 
    public static void main(String[] args) {
        AtomicInteger atomicInteger = new AtomicInteger(2020); 
        /*
         * 类似于我们平时写的SQL：乐观锁
         *
         * 如果某个线程在执行操作某个对象的时候，其他线程若操作了该对象，
         * 即使对象内容未发生变化，也需要告诉我。
         *
         * 期望、更新：
         * public final boolean compareAndSet(int 
         *                                    expect, int update) 
         * 如果我期望的值达到了，那么就更新，否则，就不更新, 
         *                                    CAS 是CPU的并发原语！ 
         */
        // ============== 捣乱的线程 ================== 
        System.out.println(atomicInteger.compareAndSet(2020, 2021)); 
        System.out.println(atomicInteger.get()); 
        System.out.println(atomicInteger.compareAndSet(2021, 2020)); 
        System.out.println(atomicInteger.get()); 
        // ============== 期望的线程 ================== 
        System.out.println(atomicInteger.compareAndSet(2020, 6666)); 
        System.out.println(atomicInteger.get()); 
    } 
}
```

输出结果如图：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210130191710471.png)

## 20、原子引用

> 解决ABA 问题，引入原子引用！ 对应的思想：乐观锁！

带版本号 的原子操作！

```java
import java.util.concurrent.TimeUnit; 
import java.util.concurrent.atomic.AtomicStampedReference; 
    public class CASDemo {
        /*
         * AtomicStampedReference 注意，
         * 如果泛型是一个包装类，注意对象的引用问题 
         * 正常在业务操作，这里面比较的都是一个个对象 
         */
        // 可以有一个初始对应的版本号 1
        static AtomicStampedReference<Integer> 
                        atomicStampedReference = 
                            new AtomicStampedReference<>(2020,1);
        // CAS compareAndSet : 比较并交换！ 
        public static void main(String[] args) {
            new Thread(()->{
                // 获得版本号
                int stamp = atomicStampedReference.getStamp(); 
                System.out.println("a1=>"+stamp); 
                try {
                    TimeUnit.SECONDS.sleep(2); 
                } catch (InterruptedException e) {
                    e.printStackTrace(); 
                }
                atomicStampedReference.compareAndSet(
                    2020, 
                    2022, 
                    atomicStampedReference.getStamp(), // 最新版本号
                    // 更新版本号
                    atomicStampedReference.getStamp() + 1); 
                      System.out.println("a2=>"
                                 +atomicStampedReference.getStamp()); 
                     System.out.println(
                        atomicStampedReference.compareAndSet(
                            2022, 
                            2020, 
                            atomicStampedReference.getStamp(), 
                            atomicStampedReference.getStamp() + 1)); 
                    System.out.println("a3=>"
                                 +atomicStampedReference.getStamp()); 
                },"a").start(); 
            // 乐观锁的原理相同！ 
            new Thread(()->{
                // 获得版本号 
                int stamp = atomicStampedReference.getStamp(); 
                System.out.println("b1=>"+stamp); 
                try {
                    TimeUnit.SECONDS.sleep(2); 
                } catch (InterruptedException e) {
                    e.printStackTrace(); 
                }
                System.out.println(
                    atomicStampedReference.compareAndSet(
                                    2020, 6666, stamp, stamp + 1)); 
                System.out.println("b2=>"
                +atomicStampedReference.getStamp()); 
            },"b").start();
        } 
}
```

结果如图：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210130191721474.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzU5MTk4MA==,size_16,color_FFFFFF,t_70)

**注意：**

**Integer 使用了对象缓存机制，默认范围是 -128 ~ 127 ，推荐使用静态工厂方法 valueOf 获取对象实例，而不是 new，因为 valueOf 使用缓存，而 new 一定会创建新的对象分配新的内存空间；**

下面是阿里巴巴开发手册的规范点：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210130191731534.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzU5MTk4MA==,size_16,color_FFFFFF,t_70)

## 21、各种锁的理解

### 1、公平锁、非公平锁

公平锁： 非常公平， 不能够插队，必须先来后到！

非公平锁：非常不公平，可以插队 （默认都是非公平）

```java
public ReentrantLock() {
    sync = new NonfairSync(); 
}
public ReentrantLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync(); 
}
```

### 2、可重入锁

可重入锁（递归锁）

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210130191741797.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzU5MTk4MA==,size_16,color_FFFFFF,t_70)

> Synchronized 版

```java
// Synchronized
public class Demo01 {
    public static void main(String[] args) {
        Phone phone = new Phone();
        new Thread(()->{
            phone.sms();
        },"A").start();
        new Thread(()->{
            phone.sms();
        },"B").start();
    }
}
class Phone{
    public synchronized void sms(){
        System.out.println(Thread.currentThread().getName() 
                                                           + "sms");
        call(); // 这里也有锁(sms锁 里面的call锁)
    }
    public synchronized void call(){
        System.out.println(Thread.currentThread().getName() 
                                                           + "call");
    }
}
```

> Lock 版

```java
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;
public class Demo02 {
    public static void main(String[] args) {
        Phone2 phone = new Phone2();
        new Thread(()->{
            phone.sms();
        },"A").start();
        new Thread(()->{
            phone.sms();
        },"B").start();
    }
}
class Phone2{
    Lock lock = new ReentrantLock();
    public void sms(){
        lock.lock(); 
        // 细节问题：lock.lock(); lock.unlock(); 
        // lock 锁必须配对，否则就会死在里面
        // 两个lock() 就需要两次解锁
        lock.lock();
        try {
            System.out.println(Thread.currentThread().getName() 
                                                           + "sms");
            call(); // 这里也有锁
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
            lock.unlock();
        }
    }
    public void call(){
        lock.lock();
        try {
            System.out.println(Thread.currentThread().getName() 
                                                           + "call");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}
```

### 3、自旋锁

spinlock

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210130191755119.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzU5MTk4MA==,size_16,color_FFFFFF,t_70)

我们来自定义一个锁测试：

```java
import java.util.concurrent.atomic.AtomicReference;
/**
 * 自旋锁
 */
public class SpinlockDemo {
    // int   0
    // Thread  null
    // 原子引用
    AtomicReference<Thread> atomicReference = 
                                            new AtomicReference<>();
    // 加锁
    public void myLock(){
        Thread thread = Thread.currentThread();
        System.out.println(Thread.currentThread().getName() 
                                                       + "==> mylock");
        // 自旋锁
        while (!atomicReference.compareAndSet(null,thread)){
        }
    }
    // 解锁
    // 加锁
    public void myUnLock(){
        Thread thread = Thread.currentThread();
        System.out.println(Thread.currentThread().getName()
                                                   + "==> myUnlock");
        atomicReference.compareAndSet(thread,null);// 解锁
    }
}
```

> 测试

```java
import java.util.concurrent.TimeUnit;
public class TestSpinLock {
    public static void main(String[] args) throws 
                                            InterruptedException {
//        ReentrantLock reentrantLock = new ReentrantLock();
//        reentrantLock.lock();
//        reentrantLock.unlock();
        // 底层使用的自旋锁CAS
        SpinlockDemo lock = new SpinlockDemo();// 定义锁
        new Thread(()-> {
            lock.myLock();// 加锁
            try {
                TimeUnit.SECONDS.sleep(5);
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                lock.myUnLock();// 解锁
            }
        },"T1").start();
        TimeUnit.SECONDS.sleep(1);
        new Thread(()-> {
            lock.myLock();
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                lock.myUnLock();
            }
        },"T2").start();
    }
}
```

结果如图：

![请添加图片描述](https://img-blog.csdnimg.cn/20210130191812975.png)

### 4、死锁

> 死锁是什么?

![请添加图片描述](https://img-blog.csdnimg.cn/20210130191813160.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzU5MTk4MA==,size_16,color_FFFFFF,t_70)