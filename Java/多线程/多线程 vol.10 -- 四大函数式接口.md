## 12、四大函数式接口（必需掌握）

新时代的程序员：lambda表达式、链式编程、函数式接口、Stream流式计算

> 函数式接口： 只有一个方法的接口

```java
@FunctionalInterface 
public interface Runnable {
    public abstract void run(); 
}
// 泛型、枚举、反射 
// lambda表达式、链式编程、函数式接口、Stream流式计算 
// 超级多FunctionalInterface 
// 简化编程模型，在新版本的框架底层大量应用！ 
// foreach(消费者类的函数式接口)
```

#### Function 函数式接口

> Function函数式接口

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210130191001358.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzU5MTk4MA==,size_16,color_FFFFFF,t_70)

```java
import java.util.function.Function;
/**
 * Function 函数型接口, 有一个输入参数，有一个输出参数
 * 只要是 函数型接口 可以 用 lambda表达式简化
 */
public class Demo01 {
    public static void main(String[] args) {
       /*Function<String,String> function = new 
                                        Function<String,String>() {
            @Override
            public String apply(String str) {
                return str;
            }
        };*/
        // lambda 表达式简化：
        Function<String,String> function = str->{
    return str;};
        System.out.println(function.apply("asd"));
    }
}
```

#### Predicate 断定型接口

> 断定型接口：有一个输入参数，返回值只能是 布尔值！

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210130191013684.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzU5MTk4MA==,size_16,color_FFFFFF,t_70)

```java
import java.util.function.Predicate;
/*
 * 断定型接口：有一个输入参数，返回值只能是 布尔值！
 */
public class Demo02 {
    public static void main(String[] args) {
        // 判断字符串是否为空
        /*Predicate<String> predicate = new Predicate<String>(){
            @Override
            public boolean test(String str) {
                return str.isEmpty();//true或false
            }
        };*/
        Predicate<String> predicate = 
                            (str)->{
    return str.isEmpty(); };
        System.out.println(predicate.test(""));//true
    }
}
```

#### Consumer 消费型接口

> Consumer 消费型接口

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210130191025678.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzU5MTk4MA==,size_16,color_FFFFFF,t_70)

```java
import java.util.function.Consumer;
/**
 * Consumer 消费型接口: 只有输入，没有返回值
 */
public class Demo03 {
    public static void main(String[] args) {
        /*Consumer<String> consumer = new Consumer<String>() {
            @Override
            public void accept(String str) {
                System.out.println(str);
            }
        };*/
        Consumer<String> consumer = 
                                (str)->{
    System.out.println(str);};
        consumer.accept("sdadasd");
    }
}
```

#### Supplier 供给型接口

> Supplier 供给型接口

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210130191038388.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzU5MTk4MA==,size_16,color_FFFFFF,t_70)

```java
import java.util.function.Supplier;
/**
 * Supplier 供给型接口 没有参数，只有返回值
 */
public class Demo04 {
    public static void main(String[] args) {
        /*Supplier supplier = new Supplier<Integer>() {
            @Override
            public Integer get() {
                System.out.println("get()");
                return 1024;
            }
        };*/
        Supplier supplier = ()->{
     return 1024; };
        System.out.println(supplier.get());
    }
}
```

## 13、Stream 流式计算

> 什么是Stream流式计算

大数据：存储 + 计算

集合、MySQL 本质就是存储东西的；

计算都应该交给流来操作！

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210130191049934.png)

```java
package com.haust.stream;
import java.util.Arrays;
import java.util.List;
/**
 * 题目要求：一分钟内完成此题，只能用一行代码实现！
 * 现在有5个用户！筛选：
 * 1、ID 必须是偶数
 * 2、年龄必须大于23岁
 * 3、用户名转为大写字母
 * 4、用户名字母倒着排序
 * 5、只输出一个用户！
 */
public class Test {
    public static void main(String[] args) {
        User u1 = new User(1,"a",21);
        User u2 = new User(2,"b",22);
        User u3 = new User(3,"c",23);
        User u4 = new User(4,"d",24);
        User u5 = new User(6,"e",25);
        // 集合就是存储
        List<User> list = Arrays.asList(u1, u2, u3, u4, u5);
        // 计算交给Stream流
        // lambda表达式、链式编程、函数式接口、Stream流式计算
        list.stream()
                .filter(u->{
    return u.getId()%2==0;})// ID 必须是偶数
                .filter(u->{
    return u.getAge()>23;})// 年龄必须大于23岁
                // 用户名转为大写字母
                .map(u->{
    return u.getName().toUpperCase();})
                // 用户名字母倒着排序
                .sorted((uu1,uu2)->{
    return uu2.compareTo(uu1);})
                .limit(1)// 只输出一个用户！
                .forEach(System.out::println);
    }
}
```

