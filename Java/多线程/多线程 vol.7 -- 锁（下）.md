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

