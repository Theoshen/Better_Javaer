## 18、彻底玩转单例模式

**饿汉式** **DCL懒汉式**，深究！

> 饿汉式

```java
// 饿汉式单例
public class Hungry {
    // 可能会浪费空间
    private byte[] data1 = new byte[1024*1024];
    private byte[] data2 = new byte[1024*1024];
    private byte[] data3 = new byte[1024*1024];
    private byte[] data4 = new byte[1024*1024];
    private Hungry(){
    }
    private final static Hungry HUNGRY = new Hungry();
    public static Hungry getInstance(){
        return HUNGRY;
    }
}
```

> DCL 懒汉式

```java
import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
// 懒汉式单例
// 道高一尺，魔高一丈！
public class LazyMan {
    private static boolean csp = false;// 标志位
    // 单例不安全，因为反射可以破坏单例，如下解决这个问题：
    private LazyMan(){
        synchronized (LazyMan.class){
            if (csp == false){
                csp = true;
            }else {
                throw new RuntimeException("不要试图使用反射破坏异常");
            }
        }
    }
    /**
     * 计算机指令执行顺序：
     * 1. 分配内存空间
     * 2、执行构造方法，初始化对象
     * 3、把这个对象指向这个空间
     *
     * 期望顺序是：123
     * 特殊情况下实际执行：132  ===>  此时 A 线程没有问题
     *                               若额外加一个 B 线程 
     *                               此时lazyMan还没有完成构造
     */
    // 原子性操作：避免指令重排
    private volatile static LazyMan lazyMan;
    // 双重检测锁模式的 懒汉式单例  DCL懒汉式
    public static LazyMan getInstance(){
        if (lazyMan==null){
            synchronized (LazyMan.class){
                if (lazyMan==null){
                    lazyMan = new LazyMan(); // 不是一个原子性操作
                }
            }
        }
        return lazyMan;
    }
    // 反射！
    public static void main(String[] args) throws Exception {
        //LazyMan instance = LazyMan.getInstance();
        Field qinjiang = LazyMan.class.getDeclaredField("csp");
        csp.setAccessible(true);
        Constructor<LazyMan> declaredConstructor = 
                        LazyMan.class.getDeclaredConstructor(null);
        declaredConstructor.setAccessible(true);
        LazyMan instance = declaredConstructor.newInstance();
        qinjiang.set(instance,false);
        LazyMan instance2 = declaredConstructor.newInstance();
        System.out.println(instance);
        System.out.println(instance2);
    }
}
```

> 静态内部类

```java
// 静态内部类
public class Holder {
    private Holder(){
    }
    public static Holder getInstace(){
        return InnerClass.HOLDER;
    }
    public static class InnerClass{
        private static final Holder HOLDER = new Holder();
    }
}
```

> 单例不安全，因为反射可以破坏单例

解决方式：

```java
private LazyMan(){
        synchronized (LazyMan.class){
            if (csp == false){
                csp = true;
            }else {
                throw new RuntimeException("不要试图使用反射破坏异常");
            }
        }
    }
```

> 枚举

```java
package com.haust.single;
import java.lang.reflect.Constructor;
import java.lang.reflect.InvocationTargetException;
// enum 是一个什么？ 本身也是一个Class类
public enum EnumSingle {
    INSTANCE;
    public EnumSingle getInstance(){
        return INSTANCE;
    }
}
class Test{
    public static void main(String[] args) throws NoSuchMethodException, IllegalAccessException, InvocationTargetException, InstantiationException {
        EnumSingle instance1 = EnumSingle.INSTANCE;
        Constructor<EnumSingle> declaredConstructor = EnumSingle.class.getDeclaredConstructor(String.class,int.class);
        declaredConstructor.setAccessible(true);
        EnumSingle instance2 = declaredConstructor.newInstance();
        // NoSuchMethodException: com.kuang.single.EnumSingle.<init>()
        System.out.println(instance1);
        System.out.println(instance2);
    }
}
```

