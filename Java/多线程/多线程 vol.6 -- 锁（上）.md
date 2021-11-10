## 3、Synchronized锁

> 传统 Synchronized锁

一个多线程卖票例子

```java
package com.haust.juc01;
/*
 * @Description: 卖票例子
 */
public class SaleTicketTDemo01 {
    /*
     * 线程就是一个单独的资源类，没有任何附属的操作！
     * 1、 属性、方法
     */
    public static void main(String[] args) {
        //并发：多个线程同时操作一个资源类，把资源类丢入线程
        Ticket ticket = new Ticket();
        // @FunctionalInterface 函数式接口，jdk1.8 lambada表达式
        new Thread(() -> {
            for (int i = 1; i < 50; i++) {
                ticket.sale();
            }
        }, "A").start();
        new Thread(() -> {
            for (int i = 1; i < 50; i++) {
                ticket.sale();
            }
        }, "B").start();
        new Thread(() -> {
            for (int i = 1; i < 50; i++) {
                ticket.sale();
            }
        }, "C").start();
    }
}
//资源类 OOP
class Ticket {
    //属性、方法
    private int number = 50;
    // 卖票的方式
    // synchronized 本质: 队列，锁
    public synchronized void sale() {
        if (number > 0) {
            System.out.println(Thread.currentThread().getName() + "卖出了" +
                    (50-(--number)) + "张票，剩余:" + number + "张票");
        }
    }
}
```

## 4、Lock锁(重点)

> Lock 接口

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210126212347234.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzU5MTk4MA==,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210126215632318.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzU5MTk4MA==,size_16,color_FFFFFF,t_70)

- **公平锁：十分公平，线程执行顺序按照先来后到顺序**
- **非公平锁：十分不公平：可以插队 （默认锁）**

将上面的卖票例子用lock锁 替换synchronized：

```java

/*
 * @Description: 卖票例子2
 */
public class SaleTicketTDemo02 {
    public static void main(String[] args) {
        //并发：多个线程同时操作一个资源类，把资源类丢入线程
        Ticket2 ticket = new Ticket2();
        // @FunctionalInterface 函数式接口，jdk1.8 lambada表达式
        new Thread(() -> {
            for (int i = 1; i < 50; i++) {
                ticket.sale();
            }
        }, "A").start();
        new Thread(() -> {
            for (int i = 1; i < 50; i++) {
                ticket.sale();
            }
        }, "B").start();
        new Thread(() -> {
            for (int i = 1; i < 50; i++) {
                ticket.sale();
            }
        }, "C").start();
    }
}
//Lock 3步骤
// 1. new ReentrantLock();
// 2. lock.lock()  加锁
// 3. lock.unlock() 解锁
class Ticket2 {
    //属性、方法
    private int number = 50;
    Lock lock = new ReentrantLock();
    // 卖票方式
    public void sale() {
        lock.lock();// 加锁
        try {
            // 业务代码
            if (number > 0) {
                System.out.println(Thread.currentThread().getName() + "卖出了" +
                        (50 - (--number)) + "张票，剩余:" + number + "张票");
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();// 解锁
        }
    }
}
```

> Synchronized 和 Lock 区别：

- 1、Synchronized 内置的Java关键字， Lock 是一个Java类
- 2、Synchronized 无法判断获取锁的状态，Lock 可以判断是否获取到了锁
- 3、Synchronized 会自动释放锁，lock 必须要手动释放锁！如果不释放锁，**死锁**
- 4、Synchronized 线程 1（获得锁，如果线程1阻塞）、线程2（等待，傻傻的等）；Lock锁就不一定会等待下去；
- 5、Synchronized **可重入锁，不可以中断的，非公平**；Lock ，**可重入锁，可以判断锁，非公平**（可以自己设置）；
- 6、Synchronized 适合锁少量的代码同步问题，Lock 适合锁大量的同步代码！

> 锁是什么，如何判断锁的是什么！

