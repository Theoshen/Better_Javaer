## 16、JMM

> 请你谈谈你对 Volatile 的理解

**Volatile** 是 Java 虚拟机提供**轻量级的同步机制**，类似于**synchronized** 但是没有其强大。

1、保证可见性

**2、不保证原子性**

3、防止指令重排

> 什么是JMM

JMM ： Java内存模型，不存在的东西，概念！约定！

**关于JMM的一些同步的约定：**

1、线程解锁前，必须把共享变量**立刻**刷回主存。

2、线程加锁前，必须读取主存中的最新值到工作内存中！

3、加锁和解锁是同一把锁。

线程 **工作内存** 、**主内存**

**8 种操作：**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210130191449596.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzU5MTk4MA==,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/2021013019150132.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzU5MTk4MA==,size_16,color_FFFFFF,t_70)

**内存交互操作有8种，虚拟机实现必须保证每一个操作都是原子的，不可在分的（对于double和long类型的变量来说，load、store、read和writ操作在某些平台上允许例外）**

- lock （锁定）：作用于主内存的变量，把一个变量标识为线程独占状态
- unlock （解锁）：作用于主内存的变量，它把一个处于锁定状态的变量释放出来，释放后的变量才可以被其他线程锁定
- read （读取）：作用于主内存变量，它把一个变量的值从主内存传输到线程的工作内存中，以便随后的load动作使用
- load （载入）：作用于工作内存的变量，它把read操作从主存中变量放入工作内存中
- use （使用）：作用于工作内存中的变量，它把工作内存中的变量传输给执行引擎，每当虚拟机遇到一个需要使用到变量的值，就会使用到这个指令
- assign （赋值）：作用于工作内存中的变量，它把一个从执行引擎中接受到的值放入工作内存的变量副本中
- store （存储）：作用于主内存中的变量，它把一个从工作内存中一个变量的值传送到主内存中，以便后续的write使用
- write （写入）：作用于主内存中的变量，它把store操作从工作内存中得到的变量的值放入主内存的变量中

**JMM 对这八种指令的使用，制定了如下规则：**

- 不允许read和load、store和write操作之一单独出现。即使用了read必须load，使用了store必须write
- 不允许线程丢弃他最近的assign操作，即工作变量的数据改变了之后，必须告知主存
- 不允许一个线程将没有assign的数据从工作内存同步回主内存
- 一个新的变量必须在主内存中诞生，不允许工作内存直接使用一个未被初始化的变量。就是怼变量实施use、store操作之前，必须经过assign和load操作
- 一个变量同一时间只有一个线程能对其进行lock。多次lock后，必须执行相同次数的unlock才能解锁
- 如果对一个变量进行lock操作，会清空所有工作内存中此变量的值，在执行引擎使用这个变量前，必须重新load或assign操作初始化变量的值
- 如果一个变量没有被lock，就不能对其进行unlock操作。也不能unlock一个被其他线程锁住的变量
- 对一个变量进行unlock操作之前，必须把此变量同步回主内存

**问题： 程序不知道主内存的值已经被修改过了**

![在这里插入图片描述](https://img-blog.csdnimg.cn/2021013019151213.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzU5MTk4MA==,size_16,color_FFFFFF,t_70)

## 17、Volatile

> 1、保证可见性

```java
import java.util.concurrent.TimeUnit;
public class JMMDemo {
    // 不加 volatile 程序就会死循环！
    // 加 volatile 可以保证可见性
    private volatile static int num = 0;
    public static void main(String[] args) {
     // main
        new Thread(()->{
     // 线程 1 对主内存的变化不知道的
            while (num==0){
            }
        }).start();
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        num = 1;
        System.out.println(num);
    }
}
```

> 不保证原子性

原子性 : 不可分割

线程A在执行任务的时候，不能被打扰的，也不能被分割。要么同时成功，要么同时失败。

```java
import java.util.concurrent.atomic.AtomicInteger;
// volatile 不保证原子性
public class VDemo02 {
    // volatile 不保证原子性
    // 原子类的 Integer
    private volatile static AtomicInteger num = new AtomicInteger();
    public static void add(){
        // num++; // 不是一个原子性操作
        num.getAndIncrement(); // AtomicInteger + 1 方法， CAS
    }
    public static void main(String[] args) {
        //理论上num结果应该为 2 万
        for (int i = 1; i <= 20; i++) {
            new Thread(()->{
                for (int j = 0; j < 1000 ; j++) {
                    add();
                }
            }).start();
        }
        // 判断只要剩下的线程不大于2个，就说明20个创建的线程已经执行结束
        while (Thread.activeCount()>2){
     // Java 默认有 main gc 2个线程
            Thread.yield();
        }
        System.out.println(Thread.currentThread().getName() 
                                                       + " " + num);
    }
}
```

**如果不加** **lock** **和** **synchronized** **，怎么样保证原子性**

![在这里插入图片描述](https://img-blog.csdnimg.cn/2021013019152570.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzU5MTk4MA==,size_16,color_FFFFFF,t_70)

使用原子类，解决原子性问题。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210130191536463.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzU5MTk4MA==,size_16,color_FFFFFF,t_70)

```java
// volatile 不保证原子性
 // 原子类的 Integer
 private volatile static AtomicInteger num = new AtomicInteger();
 public static void add(){
    // num++; // 不是一个原子性操作
    num.getAndIncrement(); // AtomicInteger + 1 方法， CAS
 }
```

这些类的底层都直接和操作系统挂钩！在内存中修改值！Unsafe类是一个很特殊的存在！

> 指令重排

什么是指令重排？：**我们写的程序，计算机并不是按照你写的那样去执行的。**

源代码 —> 编译器优化的重排 —> 指令并行也可能会重排 —> 内存系统也会重排 ——> 执行

**处理器在执行指令重排的时候，会考虑：数据之间的依赖性**

```java
int x = 1; // 1
int y = 2; // 2
x = x + 5; // 3
y = x * x; // 4
```

我们所期望的：1234 但是可能执行的时候会变成 2134 或者 1324

但是不可能是 4123！

前提：a b x y 这四个值默认都是 0：

可能造成影响得到不同的结果：

```java
| 线程A   | 线程B   |
| ----- | ----- |
| x = a | y = b |
| b =1  | a = 2 |
```

正常的结果：x = 0; y = 0; 但是可能由于指令重排出现以下结果：

```java
| 线程A   | 线程B   |
| ----- | ----- |
| b = 1 | a = 2 |
| x = a | y = b |
```

指令重排导致的诡异结果： x = 2; y = 1;

> 非计算机专业

**volatile** 可以避免指令重排：

内存屏障。CPU指令。作用：

1. 保证特定操作的执行顺序！
2. 可以保证某些变量的内存可见性 (利用这些特性**volatile** 实现了可见性)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210130191550199.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzU5MTk4MA==,size_16,color_FFFFFF,t_70)

**volatile 是可以保证可见性。不能保证原子性，由于内存屏障，可以保证避免指令重排的现象产生！**

**volatile 内存屏障在单例模式中使用的最多！**

