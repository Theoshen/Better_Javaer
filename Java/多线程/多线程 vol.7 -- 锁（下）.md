## 5、生产者和消费者问题

面试常考的问题：单例模式、排序算法、生产者和消费者、死锁

> 生产者和消费者问题 Synchronized 版

```java

/**
 * 线程之间的通信问题：生产者和消费者问题！  等待唤醒，通知唤醒
 * 线程交替执行  A   B 操作同一个变量   num = 0
 * A num+1
 * B num-1
 */
public class A {
    public static void main(String[] args) {
        Data data = new Data();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    data.increment();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "A").start();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    data.decrement();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "B").start();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    data.increment();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "C").start();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    data.decrement();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "D").start();
    }
}
// 判断等待，业务，通知
class Data {
     // 数字 资源类
    private int number = 0;
    //+1
    public synchronized void increment() throws InterruptedException {
        /*
        假设 number此时等于1，即已经被生产了产品
        如果这里用的是if判断，如果此时A,C两个生产者线程争夺increment()方法执行权
        假设A拿到执行权，经过判断number!=0成立，则A.wait()开始等待（wait()会释放锁），然后C试图去执行
        生产方法，但依然判断number!=0成立，则B.wait()开始等待（wait()会释放锁）
        碰巧这时候消费者线程线程B/D去消费了一个产品，使number=0然后，B/D消费完后调用this.notifyAll();
        这时候2个等待中的生产者线程继续生产产品，而此时number++ 执行了2次
        同理，重复上述过程，生产者线程继续wait()等待，消费者调用this.notifyAll();
        然后生产者继续超前生产，最终导致‘产能过剩’，即number大于1
        if(number != 0){
            // 等待
            this.wait();
        }*/
        while (number != 0) {
     // 注意这里不可以用if 否则会出现虚假唤醒问题，解决方法将if换成while
            // 等待
            this.wait();
        }
        number++;
        System.out.println(Thread.currentThread().getName() + "=>" + number);
        // 通知其他线程，我+1完毕了
        this.notifyAll();
    }
    //-1
    public synchronized void decrement() throws InterruptedException {
        while (number == 0) {
            // 等待
            this.wait();
        }
        number--;
        System.out.println(Thread.currentThread().getName() + "=>" + number);
        // 通知其他线程，我-1完毕了
        this.notifyAll();
    }
}
```

> 问题存在，A B C D 4 个线程！ 虚假唤醒

首先到CHM 官方文档 java.lang包下 找到Object ，然后找到wait()方法：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210127093713600.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzU5MTk4MA==,size_16,color_FFFFFF,t_70)

因此上述代码中必须使用**while**判断，而不能使用**if**

> JUC版的生产者和消费者问题

![请添加图片描述](https://img-blog.csdnimg.cn/20210127093741185.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzU5MTk4MA==,size_16,color_FFFFFF,t_70)

官方文档中通过Lock 找到 Condition

![请添加图片描述](https://img-blog.csdnimg.cn/20210127093741192.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzU5MTk4MA==,size_16,color_FFFFFF,t_70)

点入Condition 查看

![请添加图片描述](https://img-blog.csdnimg.cn/20210127101412718.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzU5MTk4MA==,size_16,color_FFFFFF,t_70)

![请添加图片描述](https://img-blog.csdnimg.cn/20210127101412673.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzU5MTk4MA==,size_16,color_FFFFFF,t_70)

代码实现：

```java

/**
 * 线程之间的通信问题：生产者和消费者问题！  等待唤醒，通知唤醒
 * 线程交替执行  A   B 操作同一个变量   num = 0
 * A num+1
 * B num-1
 */
public class B {
    public static void main(String[] args) {
        Data2 data = new Data2();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    data.increment();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "A").start();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    data.decrement();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "B").start();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    data.increment();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "C").start();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    data.decrement();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "D").start();
    }
}
// 判断等待，业务，通知
class Data2 {
     // 数字 资源类
    private int number = 0;
    Lock lock = new ReentrantLock();
    Condition condition = lock.newCondition();
    //condition.await(); // 等待 
    //condition.signalAll(); // 唤醒全部
    //+1
    public  void increment() throws InterruptedException {
        lock.lock();
        try {
            // 业务代码
            while (number != 0) {
                // 等待
                condition.await();
            }
            number++;
            System.out.println(Thread.currentThread().getName() + "=>" + number);
            // 通知其他线程，我+1完毕了
            condition.signal();
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            lock.unlock();
        }
    }
    //-1
    public  void decrement() throws InterruptedException {
        lock.lock();
        try {
            while (number == 0) {
                // 等待
                condition.await();
            }
            number--;
            System.out.println(Thread.currentThread().getName() + "=>" + number);
            // 通知其他线程，我-1完毕了
            condition.signal();
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            lock.unlock();
        }
    }
}
```

**任何一个新的技术，绝对不是仅仅只是覆盖了原来的技术，是有其对旧技术的优势和补充！**

> Condition 精准的通知和唤醒线程

上述代码运行结果如图：

![请添加图片描述](https://img-blog.csdnimg.cn/20210127101516447.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzU5MTk4MA==,size_16,color_FFFFFF,t_70)

**问题：ABCD线程 抢占执行的顺序是随机的，如果想让ABCD线程有序执行，该如何改进代码？**

代码实现：

```java
/*
 * A 执行完调用B，B执行完调用C，C执行完调用A
 */
public class C {
    public static void main(String[] args) {
        Data3 data = new Data3();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                data.printA();
            }
        }, "A").start();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                data.printB();
            }
        }, "B").start();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                data.printC();
            }
        }, "C").start();
    }
}
class Data3 {
     // 资源类 Lock
    private Lock lock = new ReentrantLock();
    private Condition condition1 = lock.newCondition();
    private Condition condition2 = lock.newCondition();
    private Condition condition3 = lock.newCondition();
    private int number = 1; 
    // number=1 A执行  number=2 B执行 number=3 C执行
    public void printA() {
        lock.lock();
        try {
            // 业务，判断-> 执行-> 通知
            while (number != 1) {
                // A等待
                condition1.await();
            }
            System.out.println(Thread.currentThread().getName() + "=>AAAAAAA");
            // 唤醒，唤醒指定的人，B
            number = 2;
            condition2.signal();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
    public void printB() {
        lock.lock();
        try {
            // 业务，判断-> 执行-> 通知
            while (number != 2) {
                // B等待
                condition2.await();
            }
            System.out.println(Thread.currentThread().getName() + "=>BBBBBBBBB");
            // 唤醒，唤醒指定的人，c
            number = 3;
            condition3.signal();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
    public void printC() {
        lock.lock();
        try {
            // 业务，判断-> 执行-> 通知
            // 业务，判断-> 执行-> 通知
            while (number != 3) {
                // C等待
                condition3.await();
            }
            System.out.println(Thread.currentThread().getName() + "=>CCCCC ");
            // 唤醒，唤醒指定的人，A
            number = 1;
            condition1.signal();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}
```

测试结果：

![请添加图片描述](https://img-blog.csdnimg.cn/20210127101516440.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzU5MTk4MA==,size_16,color_FFFFFF,t_70)

## 6、8锁现象

前面提出一个问题：如何判断锁的是谁！知道什么是锁，锁到底锁的是谁！

**深刻理解我们的锁**

**synchronized 锁的对象是方法的调用者**

#### 代码举例1：

```java
import java.util.concurrent.TimeUnit;
/**
 * 8锁，就是关于锁的8个问题
 * 1、标准情况下，两个线程先打印 发短信还是 先打印 打电话？ 1/发短信  2/打电话
 * 1、sendSms延迟4秒，两个线程先打印 发短信还是 打电话？ 1/发短信  2/打电话
 */.
public class Test1 {
    public static void main(String[] args) {
        Phone phone = new Phone();
        // 锁的存在
        new Thread(()->{
            phone.sendSms();
        },"A").start();
        // 捕获
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        new Thread(()->{
            phone.call();
        },"B").start();
    }
}
class Phone{
    // synchronized 锁的对象是方法的调用者！、
    // 两个方法用的是同一个对象调用(同一个锁)，谁先拿到锁谁执行！
    public synchronized void sendSms(){
        try {
            TimeUnit.SECONDS.sleep(4);// 抱着锁睡眠
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("发短信");
    }
    public synchronized void call(){
        System.out.println("打电话");
    }
}
// 先执行 发短信，后执行打电话
```

**普通方法没有锁！不是同步方法，就不受锁的影响，正常执行**

#### 代码举例2：

```java
import java.util.concurrent.TimeUnit;
/**
 * 3、 增加了一个普通方法后！先执行发短信还是Hello？// 普通方法
 * 4、 两个对象，两个同步方法， 发短信还是 打电话？ // 打电话
 */
public class Test2  {
    public static void main(String[] args) {
        // 两个对象，两个调用者，两把锁！
        Phone2 phone1 = new Phone2();
        Phone2 phone2 = new Phone2();
        //锁的存在
        new Thread(()->{
            phone1.sendSms();
        },"A").start();
        // 捕获
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        new Thread(()->{
            phone2.call();
        },"B").start();
        new Thread(()->{
            phone2.hello();
        },"C").start();
    }
}
class Phone2{
    // synchronized 锁的对象是方法的调用者！
    public synchronized void sendSms(){
        try {
            TimeUnit.SECONDS.sleep(4);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("发短信");
    }
    public synchronized void call(){
        System.out.println("打电话");
    }
    // 这里没有锁！不是同步方法，不受锁的影响
    public void hello(){
        System.out.println("hello");
    }
}
// 先执行打电话，接着执行hello，最后执行发短信
```

**不同实例对象的Class类模板只有一个，static静态的同步方法，锁的是Class **

#### 代码举例3：

```java
import java.util.concurrent.TimeUnit;
/**
 * 5、增加两个静态的同步方法，只有一个对象，先打印 发短信？打电话？
 * 6、两个对象！增加两个静态的同步方法， 先打印 发短信？打电话？
 */
public class Test3  {
    public static void main(String[] args) {
        // 两个对象的Class类模板只有一个，static，锁的是Class
        Phone3 phone1 = new Phone3();
        Phone3 phone2 = new Phone3();
        //锁的存在
        new Thread(()->{
            phone1.sendSms();
        },"A").start();
        // 捕获
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        new Thread(()->{
            phone2.call();
        },"B").start();
    }
}
// Phone3唯一的一个 Class 对象
class Phone3{
    // synchronized 锁的对象是方法的调用者！
    // static 静态方法
    // 类一加载就有了！锁的是Class
    public static synchronized void sendSms(){
        try {
            TimeUnit.SECONDS.sleep(4);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("发短信");
    } 
    public static synchronized void call(){
        System.out.println("打电话");
    }
}
// 先执行发短信，后执行打电话
```

#### 代码举例4：

```java
import java.util.concurrent.TimeUnit;
/**
 * 7、1个静态的同步方法，1个普通的同步方法 ，一个对象，先打印 发短信？打电话？
 * 8、1个静态的同步方法，1个普通的同步方法 ，两个对象，先打印 发短信？打电话？
 */
public class Test4  {
    public static void main(String[] args) {
        // 两个对象的Class类模板只有一个，static，锁的是Class
        Phone4 phone1 = new Phone4();
        Phone4 phone2 = new Phone4();
        //锁的存在
        new Thread(()->{
            phone1.sendSms();
        },"A").start();
        // 捕获
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        new Thread(()->{
            phone2.call();
        },"B").start();
    }
}
// Phone3唯一的一个 Class 对象
class Phone4{
    // 静态的同步方法 锁的是 Class 类模板
    public static synchronized void sendSms(){
        try {
            TimeUnit.SECONDS.sleep(4);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("发短信");
    }
    // 普通的同步方法  锁的调用者(对象),二者锁的对象不同,所以不需要等待
    public synchronized void call(){
        System.out.println("打电话");
    }
}
// 7/8 两种情况下，都是先执行打电话,后执行发短信,因为二者锁的对象不同,
// 静态同步方法锁的是Class类模板,普通同步方法锁的是实例化的对象,
// 所以不用等待前者解锁后 后者才能执行,而是两者并行执行,因为发短信休眠4s
// 所以打电话先执行。
```

> 小结

- new this 具体的一个手机
- static Class 唯一的一个模板



## 7、集合类不安全

> List 不安全

**List、ArrayList 等在并发多线程条件下，不能实现数据共享，多个线程同时调用一个list对象时候就会出现并发修改异常ConcurrentModificationException **。

代码举例：

```
package com.haust.unsafe;import java.util.*;import java.util.concurrent.CopyOnWriteArrayList;// java.util.ConcurrentModificationException 并发修改异常！public class ListTest {    public static void main(String[] args) {        // 并发下 ArrayList 不安全的吗，Synchronized；        /*         * 解决方案；         * 方案1、List<String> list = new Vector<>();         * 方案2、List<String> list =         * Collections.synchronizedList(new ArrayList<>());         * 方案3、List<String> list = new CopyOnWriteArrayList<>()；         */       /* CopyOnWrite 写入时复制  COW  计算机程序设计领域的一种优化策略；        * 多个线程调用的时候，list，读取的时候，固定的，写入（覆盖）        * 在写入的时候避免覆盖，造成数据问题！        * 读写分离        * CopyOnWriteArrayList  比 Vector Nb 在哪里？        */            List<String> list = new CopyOnWriteArrayList<>();        for (int i = 1; i <= 10; i++) {            new Thread(()->{                list.add(UUID.randomUUID().toString().substring(0,5));                System.out.println(list);            },String.valueOf(i)).start();        }    }}
```

[CopyOnWriteArrayList源码分析参考](https://csp1999.blog.csdn.net/article/details/112862436)

> Set 不安全

**Set、Hash 等在并发多线程条件下，不能实现数据共享，多个线程同时调用一个set对象时候就会出现并发修改异常ConcurrentModificationException **。

代码举例：

```
package com.haust.unsafe;import java.util.HashSet;import java.util.Set;import java.util.UUID;/** * 同理可证 ： ConcurrentModificationException 并发修改异常 * 1、Set<String> set =  *                     Collections.synchronizedSet(new HashSet<>()); * 2、 */public class SetTest {    public static void main(String[] args) {        //Set<String> set = new HashSet<>();//不安全        // Set<String> set = Collections.synchronizedSet(new HashSet<>());//安全        Set<String> set = new CopyOnWriteArraySet<>();//安全        for (int i = 1; i <=30 ; i++) {           new Thread(()->{               set.add(UUID.randomUUID().toString().substring(0,5));               System.out.println(set);           },String.valueOf(i)).start();        }    }}
```

**扩展：hashSet 底层是什么？**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210127130107360.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzU5MTk4MA==,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210127130131617.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzU5MTk4MA==,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210127130147132.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzU5MTk4MA==,size_16,color_FFFFFF,t_70)

**可以看出 HashSet 的底层就是一个HashMap**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210127130207128.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzU5MTk4MA==,size_16,color_FFFFFF,t_70)

[HashMap源码分析参考](https://blog.csdn.net/weixin_43591980/article/details/109442223)

> Map 不安全

回顾Map基本操作：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210127130221562.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzU5MTk4MA==,size_16,color_FFFFFF,t_70)

代码举例：

```java
import java.util.Map;
import java.util.UUID;
import java.util.concurrent.ConcurrentHashMap;
// ConcurrentModificationException
public class  MapTest {
    public static void main(String[] args) {
        // map 是这样用的吗？ 不是，工作中不用 HashMap
        // 默认等价于什么？  new HashMap<>(16,0.75);
        // Map<String, String> map = new HashMap<>();
        // 扩展：研究ConcurrentHashMap的原理
        Map<String, String> map = new ConcurrentHashMap<>();
        for (int i = 1; i <=30; i++) {
            new Thread(()->{
                map.put(Thread.currentThread().getName(),
                       UUID.randomUUID().toString().substring(0,5));
                System.out.println(map);
            },String.valueOf(i)).start();
        }
    }
}
```

## 8、Callable (简单)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210127130507798.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzU5MTk4MA==,size_16,color_FFFFFF,t_70)

**Callable 和 Runable 对比：**

> 举例：比如**Callable** 是你自己，你想通过你的女朋友 **Runable **认识她的闺蜜 **Thread**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210127130525919.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzU5MTk4MA==,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210127130613347.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzU5MTk4MA==,size_16,color_FFFFFF,t_70)

- **Callable** 是 java.util 包下 concurrent 下的接口，有返回值，可以抛出被检查的异常
- **Runable** 是 java.lang 包下的接口，没有返回值，不可以抛出被检查的异常
- 二者调用的方法不同，**run**()/ **call**()

同样的 **Lock** 和 **Synchronized** 二者的区别，前者是java.util 下的接口 后者是 java.lang 下的关键字。

**代码举例**

```java
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.FutureTask;
/**
 * 1、探究原理
 * 2、觉自己会用
 */
public class CallableTest {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        // new Thread(new Runnable()).start();// 启动Runnable
        // new Thread(new FutureTask<V>()).start();
        // new Thread(new FutureTask<V>( Callable )).start();
        new Thread().start(); // 怎么启动Callable？
        // new 一个MyThread实例
        MyThread thread = new MyThread();
        // MyThread实例放入FutureTask
        FutureTask futureTask = new FutureTask(thread); // 适配类
        new Thread(futureTask,"A").start();
        new Thread(futureTask,"B").start(); // call()方法结果会被缓存，提高效率，因此只打印1个call
        // 这个get 方法可能会产生阻塞！把他放到最后
        Integer o = (Integer) futureTask.get(); 
        // 或者使用异步通信来处理！
        System.out.println(o);// 1024
    }
}
class MyThread implements Callable<Integer> {
    @Override
    public Integer call() {
        System.out.println("call()"); // A,B两个线程会打印几个call？（1个）
        // 耗时的操作
        return 1024;
    }
}
//class MyThread implements Runnable {
//
//    @Override
//    public void run() {
//        System.out.println("run()"); // 会打印几个run
//    }
//}
```

**细节：**

1、有缓存

2、结果可能需要等待，会阻塞！
