# 一、进程和线程

- **进程（Process）：** 进程是执行程序的一次执行过程，它是一个动态的概念。是系统资源分配的单位

- **线程（Thread）：** 一个进程中可以包含若干个线程，一个进程中至少有一个线程，不然没有存在的意义。线程是CPU调度和执行的单位。

- **注意：** 很多多线程是模拟出来的，真正的多线程是指有多个CPU，即多核，如服务器。如果是模拟出来的多线程，即在一个CPU的情况下，在同一个时间点，CPU只能执行一个代码，因为切换的很快，所以就有同时执行的错觉。

- **普通方法调用和多线程**
  ![请添加图片描述](https://img-blog.csdnimg.cn/327d3981a5534baaa9fdd9853c0f5eea.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTYyNTM0OA==,size_16,color_FFFFFF,t_70)

- **核心概念：**![请添加图片描述](https://img-blog.csdnimg.cn/f87f91b105934d9c9d842fdf228d5422.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTYyNTM0OA==,size_16,color_FFFFFF,t_70)

  # 二、线程的创建

- 三种创建方式：

  

  ## 继承Thread类

  1. 自定义线程类继承Thread类
  2. 重写run()方法，编写线程执行体
  3. 创建线程对象，调用start()方法启动线程

```
//创建线程方式一：继承Thread类，重写run()方法，调用start开启线程//总结：注意，线程开启不一定立即执行，由CPU调度执行public class TestThread1 extends Thread{    @Override    public void run(){        for (int i = 0; i < 100; i++) {            System.out.println("我在看代码--"+i);        }    }    public static void main(String[] args) {        //main线程，主线程        //创建一个线程对象        TestThread1 testThread1 = new TestThread1();        //调用start()方法开启线程        testThread1.start();        for (int i = 0; i < 1000; i++) {            System.out.println("我在学习线程--"+i);        }    }}
```

## 实现Runnable

		1. 定义一个类实现Runnable接口
		1. 实现run()方法，编写线程执行体

创建线程对象，调用start()方法启动线程

> 推荐使用Runnable对象，因为Java单继承的局限性

```java
//创建线程方式2：实现runnble接口，重写run()方法，执行线程需要丢入runnable接口类，调用start()方法
public class TestThread3 implements Runnable{    
@Override    
public void run(){ 
for (int i = 0; i < 100; i++) {
	System.out.println("我在看代码--"+i);       
	}    
}    

public static void main(String[] args) { 
	//创建runnbale接口的实现类对象        
	TestThread3 testThread3 = new TestThread3();  
	//创建线程对象，通过线程对象来开启线程代理      
	new Thread(testThread3).start();     
	for (int i = 0; i < 1000; i++) {      
		System.out.println("我在学习线程--"+i);   
		}  
	}
}
```

# 三、小结

## 	继承Thread类

1. 子类继承Thread类具备多线程能力

2. 启动线程：子类对象.start()

3. 不建议使用：避免OOP单继承局限性

   ## 实现Runnable接口

4. 实现接口Runnable具有多线程能力

5. 启动线程：传入目标对象+Thread对象.start()

6. **推荐使用：避免单继承局限性，灵活方便，方便同一个对象被多个线程使用**