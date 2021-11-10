# 一、静态代理模式

```java
public class StaticProxy {
    public static void main(String[] args) {
        SantiagoBernabeu santiagoBernabeu = new SantiagoBernabeu(new You());
        santiagoBernabeu.championLeague();
    }

}

interface Match {
    /**
     * @Description 欧冠比赛方法，测试用
     * @createTime 13:56 2021/10/12
     * @version 1.0.0
     */
    void championLeague();
}

/**
 * @Description  真实角色，参加欧冠
 * @createTime 13:37 2021/10/12
 * @version 1.0.0
 */
class You implements Match {

    @Override
    public void championLeague() {
        System.out.println("比赛开始，冲");
    }
}

/** 
 * @Description 代理角色，提供场地
 * @createTime 13:41 2021/10/12
 * @version 1.0.0
 */
class SantiagoBernabeu implements Match {

    private Match target ;

    public SantiagoBernabeu(Match target) {
        this.target = target;
    }

    @Override
    public void championLeague() {
        before();
        this.target.championLeague();
        after();
    }

    private void after() {
        System.out.println("比赛之后，总结、吃饭");
    }

    private void before() {
        System.out.println("比赛之前，热身、训练");
    }
}

```



# 二、Lamda表达式

- 理解Functional Interface（函数式接口）是学习Java8 lambda表达式的关键所在
- **函数式接口的定义：**

1. 任何接口，如果只包含唯一一个抽象方法，那么它就是一个函数式接口

   ```java
   public interface Runnable{ public abstract void run();}
   ```

2. 对于函数式接口，我们可以通过lambda表达式来创建该接口的对象

- **推导Lambda表达式：**

1. 定义一个函数式接口
2. 实现类
3. 静态内部类
4. 局部内部类
5. 匿名内部类
6. 用lambda简化

```java
public class Testlambda {   
//3.静态内部类   
	static class Like2 implements ILike{       
		@Override       
		public void lambda() {   
			System.out.println("我是静态内部类");   
			}   
		}   

	public static void main(String[] args) {    
		ILike like = new Like();     
		like.lambda();   
		like = new Like2(); 
		like.lambda();  

//4.局部内部类     
class Like3 implements ILike{    
	@Override           
	public void lambda() {    
		System.out.println("我是局部内部类");     
		}    
	}   
	like = new Like3();  
	like.lambda();     
    
//5.匿名内部类，没有类的名称，必须借助接口或者父类    
	like = new Like() {     
	@Override     
	public void lambda() {   
		System.out.println("我是匿名内部类");      
		}     
	};     
	like.lambda();   
//6.用lambda简化   
	like = ()->{  
		System.out.println("我是lambda表达式");   
		};        
	like.lambda(); 
		}
	}

//1.定义一个函数式接口
interface ILike{ 
	void lambda();
	}
//2.实现类
class Like implements ILike{ 
	@Override   
	public void lambda() {   
	System.out.println("我喜欢Java");  
	}
}
```

- 直接用lambda表达式

  ```java
  public class Testlambda { 
  public static void main(String[] args) {  
  //1.lambda表示简化    
  Ilove love = (int a)->{   
  System.out.println("我爱Lambda"+a);   
  };     
  //简化1.参数类型  
  love = (a)->{       
    System.out.println("我爱Lambda"+a);   
  };   
  //简化2.简化括号   
  love = a->{ 
    System.out.println("我爱Lambda"+a);   
  };    
  //简化3.去掉花括号   
  love = a -> 
    System.out.println("我爱Lambda"+a);   
  love.love(520); 
  }}
  //1.定义一个Ilove接口
  interface Ilove{ 
  void love(int a);}
  ```

- **总结：**

1. 前提是接口为*函数式接口*（只包含一个抽象方法的接口）
2. lambda表达式只能有一行代码的情况下才能简化成为一行，如果有多行，那么就用代码块包裹。
3. 多个参数也可以去掉参数类型，要去掉就都去掉，必须加上括号

# 三、线程状态

## 五大状态

![在这里插入图片描述](https://img-blog.csdnimg.cn/75aae5c32b5546689bd9db93a862e328.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTYyNTM0OA==,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/56a8af8ecc4f4452863d003f8c16cdcf.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTYyNTM0OA==,size_16,color_FFFFFF,t_70)

## 停止线程

- 不推荐使用stop()、destroy()方法【已废弃】

- 建议使用一个标志位进行终止变量，当flag=false，则终止线程运行。

  ## 线程休眠

- sleep(时间)指定当前线程阻塞的毫秒数（1000毫秒等于1秒）；

- sleep存在异常InterruptedException；

- sleep时间达到后线程进入就绪状态；

- sleep可以模拟网络延时，倒计时等；

- 每一个对象都有一个锁，sleep不会释放锁；

**例子：模拟倒计时**

```java
public class TestSleep {
	public static void main(String[] args) {  
		try {           
		tenDown();      
		} catch (InterruptedException e) {  
		e.printStackTrace();       
		}    
	}    
	public static void tenDown() throws InterruptedException {  
		int num = 10;      
		while (true){       
			Thread.sleep(1000);    
			System.out.println(num--);    
			if(num<=0){   	
				break;      
				}      
			} 
		}
}
```

## 线程礼让

- 礼让线程，让当前正在执行的线程暂停，但不阻塞
- 将线程从运行状态转为就绪状态
- 让CPU重新调度，礼让不一定成功！看CPU心情

```java
public class TestYield {  
	public static void main(String[] args) {  
		MyYield myYield = new MyYield();   
		new Thread(myYield,"a").start();    
		new Thread(myYield,"b").start();   
	}
}

class MyYield implements Runnable{  
	@Override  
	public void run() {  
		System.out.println(Thread.currentThread().getName()+"线程开始执行"); 
		Thread.yield(); 
		System.out.println(Thread.currentThread().getName()+"线程停止执行");  
	}
}

```

```shell
a线程开始执行
b线程开始执行
b线程停止执行
a线程停止执行
```



## Join

- Join合并线程，待此线程执行完成后，再执行其他线程，其他线程阻塞（可以想象成插队）

- **例子：模拟插队**

  ```java
  public class TestJoin implements Runnable{ 
  	@Override 
    public void run() { 
  		for (int i = 0; i < 100; i++) {   
  			System.out.println("线程vip来了"+i);   
  		} 
  	}
    
  public static void main(String[] args) throws InterruptedException {  
  	//启动线程    
  	TestJoin testJoin = new TestJoin();   
  	Thread thread = new Thread(testJoin);   
  	thread.start();    
  	//主线程    
  	for (int i = 0; i < 300; i++) {    
  		if(i == 150){     
  			thread.join();
  			//插队    
  		}    
  		System.out.println("main"+i); 
  	}
  }
  }
  ```