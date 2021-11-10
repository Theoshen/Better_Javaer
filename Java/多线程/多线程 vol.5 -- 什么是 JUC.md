## 1、什么是 JUC

JUC就是 `java.util` 下的工具包、包、分类等。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210120130409745.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzU5MTk4MA==,size_16,color_FFFFFF,t_70)

> 普通的线程代码：

- **Thread**
- **Runnable** 没有返回值、效率相比入 Callable 相对较低！
- **Callable** 有返回值！

## 2、线程和进程

> 线程、进程，如果不能使用一句话说出来的技术，不扎实！

- **进程**：一个程序，QQ.exe Music.exe 程序的集合；
- 一个进程往往可以包含多个线程，至少包含一个！
- Java默认有2个线程？ mian、GC
- **线程**：开了一个进程 Typora，写字，自动保存（线程负责的）
- 对于Java而言提供了：`Thread、Runnable、Callable`操作线程。

**Java** **真的可以开启线程吗？** 

​	答案是：开不了！

```java
public synchronized void start() {
    /**
     * This method is not invoked for the main method thread 
     * or "system" group threads created/set up by the VM. Any new 
     * functionality added to this method in the future may have to 
     * also be added to the VM.A zero status value corresponds to 
     * state "NEW".
     */
    if (threadStatus != 0)
        throw new IllegalThreadStateException();
    /* 
     * Notify the group that this thread is about to be started
     * so that it can be added to the group's list of threads
     * and the group's unstarted count can be decremented. 
     */
    group.add(this);
    boolean started = false;
    try {
        start0();
        started = true;
    } finally {
        try {
            if (!started) {
                group.threadStartFailed(this);
            }
        } catch (Throwable ignore) {
            /* do nothing. If start0 threw a Throwable then
              it will be passed up the call stack */
        }
    }
}
// 本地方法，底层操作的是C++ ，Java 无法直接操作硬件
private native void start0();
```

> 并发、并行

并发编程：并发、并行

**并发**（多线程操作同一个资源）

- 一核CPU，模拟出来多条线程，快速交替。

**并行**（多个人一起行走）

- 多核CPU ，多个线程可以同时执行； eg: 线程池！

```java
public class Test1 {
    public static void main(String[] args) {
      // 获取cpu的核数 
     // CPU 密集型，IO密集型 
        System.out.println(Runtime.getRuntime().availableProcessors());
     // 如果电脑是8核，则结果输出8
     } 
}
```

并发编程的本质：**充分利用CPU的资源**

线程有几个状态（6个）

```java
public enum State {
    /**
     * Thread state for a thread which has not yet started.
     * 线程新生状态
     */
    NEW,
    /**
     * Thread state for a runnable thread.  A thread in the runnable
     * state is executing in the Java virtual machine but it may
     * be waiting for other resources from the operating system
     * such as processor.
     * 线程运行中
     */
    RUNNABLE,
    /**
     * Thread state for a thread blocked waiting for a monitor lock.
     * A thread in the blocked state is waiting for a monitor lock
     * to enter a synchronized block/method or
     * reenter a synchronized block/method after calling
     * {@link Object#wait() Object.wait}.
     * 线程阻塞状态
     */
    BLOCKED,
    /**
     * Thread state for a waiting thread.
     * A thread is in the waiting state due to calling one of the
     * following methods:
     * <ul>
     *   <li>{@link Object#wait() Object.wait} with no timeout</li>
     *   <li>{@link #join() Thread.join} with no timeout</li>
     *   <li>{@link LockSupport#park() LockSupport.park}</li>
     * </ul>
     *
     * <p>A thread in the waiting state is waiting for another thread to
     * perform a particular action.
     *
     * For example, a thread that has called <tt>Object.wait()</tt>
     * on an object is waiting for another thread to call
     * <tt>Object.notify()</tt> or <tt>Object.notifyAll()</tt> on
     * that object. A thread that has called <tt>Thread.join()</tt>
     * is waiting for a specified thread to terminate.
     * 线程等待状态，死等
     */
    WAITING,
    /**
     * Thread state for a waiting thread with a specified waiting time.
     * A thread is in the timed waiting state due to calling one of
     * the following methods with a specified positive waiting time:
     * <ul>
     *   <li>{@link #sleep Thread.sleep}</li>
     *   <li>{@link Object#wait(long) Object.wait} with timeout</li>
     *   <li>{@link #join(long) Thread.join} with timeout</li>
     *   <li>{@link LockSupport#parkNanos LockSupport.parkNanos}</li>
     *   <li>{@link LockSupport#parkUntil LockSupport.parkUntil}</li>
     * </ul>
     * 线程超时等待状态，超过一定时间就不再等
     */
    TIMED_WAITING,
    /**
     * Thread state for a terminated thread.
     * The thread has completed execution.
     * 线程终止状态，代表线程执行完毕
     */
    TERMINATED;
}
```

> wait/sleep 区别

**1、二者来自不同的类**

- wait => Object
- sleep => Thread

**2、关于锁的释放**

- wait 会释放锁
- sleep 睡觉了，抱着锁睡觉，不会释放！

**3、使用的范围是不同的**

- **wait 必须在同步代码块中使用**
- sleep 可以再任何地方睡眠