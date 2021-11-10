## 线程协作（生产者消费者模式）

应用场景∶生产者和消费者问题

- 假设仓库中只能存放一件产品,生产者将生产出来的产品放入仓库,消费者将仓库中产品取走消费.
- 如果仓库中没有产品﹐则生产者将产品放入仓库，否则停止生产并等待，直到仓库中的产品被消费者取走为止.
- 如果仓库中放有产品﹐则消费者可以将产品取走消费，否则停止消费并等待，直到仓库中再次放入产品为止。

![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/07/28/kuangstudyccc6a899-33f1-45d5-b0b7-75633102c4e6.png)

## 分析

- **这是一个线程同步问题，生产者和消费者共享同一个资源，并且生产者和消费者之间相互依赖,互为条件．**

1. 对于生产者﹐没有生产产品之前，要通知消费者等待﹒而生产了产品之后﹐又需要马上通知消费者消费
2. 对于消费者﹐在消费之后,要通知生产者已经结束消费﹐需要生产新的产品以供消费.
3. 在生产者消费者问题中,仅有synchronized是不够的

- synchronized可阻止并发更新同一个共享资源，实现了同步
- synchronized不能用来实现不同线程之间的消息传递(通信)

## Java提供了几个方法解决线程之间的通信问题

![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/07/28/kuangstudy56c70553-33d5-40c6-ac50-4eb8364b9163.png)

## 解决方法1

并发协作模型“生产者l消费者模式”—->管程法

> 生产者:负责生产数据的模块(可能是方法﹐对象﹐线程,进程);
> 消费者:负责处理数据的模块(可能是方法﹐对象﹐线程﹐进程);
> 缓冲区:消费者不能直接使用生产者的数据﹐他们之间有个“缓冲区”

**生产者将生产好的数据放入缓冲区,消费者从缓冲区拿出数据**

例子：

```java
//测试：生产者消费者模型-->利用缓冲区解决：管程法
//生产者，消费者，产品，缓冲区
public class TestPC {
    public static void main(String[] args) {
        SynContainer container = new SynContainer();
        new Productor(container).start();
        new Consumer(container).start();
    }
}
//生产者
class Productor extends Thread{
    SynContainer container;
    public Productor(SynContainer container){
        this.container = container;
    }
    //生产
    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            container.push(new Chicken(i));
            System.out.println("生产了"+i+"只鸡");
        }
    }
}
//消费者
class Consumer extends Thread{
    SynContainer container;
    public Consumer(SynContainer container){
        this.container = container;
    }
    //消费
    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            System.out.println("消费了--->"+container.pop().id+"只鸡");
        }
    }
}
//产品
class Chicken{
    int id;// 产品编号
    public Chicken(int id){
        this.id = id;
    }
}
//缓冲区
class SynContainer{
    //需要一个容器大小
    Chicken[] chickens = new Chicken[10];
    //容器计算器
    int count = 0;
    //生产者放入产品
    public synchronized void push(Chicken chicken){
        //如果容器满了，就需要等待消费产品
        if(count==chickens.length){
            //生产者等待
            try {
                this.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        //如果没有满，我们就需要丢入产品
        chickens[count] = chicken;
        count++;
        //可以通知消费者消费了
        this.notifyAll();
    }
    //消费者消费产品
    public synchronized Chicken pop(){
        //判断能否消费
        if (count == 0){
            //等待生产者生产，消费者等待
            try {
                this.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        //如果可以消费
        count--;
        Chicken chicken = chickens[count];
        //吃完了，通知生产者生产
        this.notifyAll();
        return chicken;
    }
}
```

## 解决方法2

并发协作模型“生产者/消费者模式”—->信号灯法

```java
//测试消费者和生产者问题2：信号灯法
public class TestPC2 {
    public static void main(String[] args) {
        TV tv = new TV();
        new Player(tv).start();
        new Watcher(tv).start();
    }
}
//生产者-->演员
class Player extends Thread{
    TV tv;
    public Player(TV tv){
        this.tv = tv;
    }
    @Override
    public void run() {
        for (int i = 0; i < 20; i++) {
            if (i%2==0){
                this.tv.play("快乐大本营播放中");
            }else{
                this.tv.play("抖音，记录有钱人的美好生活");
            }
        }
    }
}
//消费者-->观众
class Watcher extends Thread{
    TV tv;
    public Watcher(TV tv){
        this.tv = tv;
    }
    @Override
    public void run() {
        for (int i = 0; i < 20; i++) {
            tv.watch();
        }
    }
}
//产品-->节目
class TV{
    // 演员表演，观众等待 T
    // 观众观看，演员等待 F
    String voice; // 表演的节目
    boolean flag = true;
    // 表演
    public synchronized void play(String voice){
        if (!flag){
            try {
                this.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        System.out.println("演员表演了："+voice);
        // 通知观众观看
        this.notifyAll();// 通知唤醒
        this.voice = voice;
        this.flag = !this.flag;//flag如果是真就取假，相同是假就取真
    }
    // 观看
    public synchronized void watch(){
        if (flag){
            try {
                this.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        System.out.println("观众观看了："+voice);
        // 通知演员表演
        this.notifyAll();
        this.flag = !this.flag;
    }
}
```

## Java解决线程之间通信问题的几个方法

| 方法名              | 作用                                                         |
| :------------------ | :----------------------------------------------------------- |
| wait()              | 表示线程一直等待，直到其他线程通知，与sleep不同，会释放锁    |
| wait(long tiimeout) | 指定等待的毫秒数                                             |
| notify()            | 唤醒一个处于等待状态的线程                                   |
| notifyAll()         | 唤醒同一个对象上所有调用wait()方法的线程，优先级别高的线程优先调度 |

- **注意：均是Object类的方法，都只能在同步方法或者同步代码块中使用，否则会抛出异常IIIegaIMonitorStateException**

