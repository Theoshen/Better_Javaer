# 《To Be a Better Javaer》-- Java 基础篇  vol.2：面向对象

Java是面向对象的高级编程语言，面向对象的特征如下：

- 面向对象具有**抽象、封装、继承、多态**等特性；
- 面向对象可以将复杂的业务逻辑简单化，增强代码复用性；
- 面向对象是一种常见的思想，比较符合人们的思考习惯。

## 面向过程和面向对象是什么？

1. 面向过程是一种让程序的功能按照先后顺序执行的编程思想，每一个功能都是一步一步实现。
2. 面向对象是将功能、事务高度抽象化的编程思想，把问题拆分成很多步骤，每一步都要进行抽象，从而形成对象。程序通过对不同对象的调用，以此来解决问题。

### 二者的区别

1. 面向过程是自顶向下的设计模式，要考虑到每一个模块应该要分解成那些子模块，每一个子模块还要分解成更小的子模块，直到把模块分解为一个一个的函数（或者方法）。面向过程的最小程序单元是函数，每一个函数负责完成一个功能，用于接收输入的数据，函数进行处理，输出结果。
2. 面向对象最小的程序单元是类，当软件系统庞大起来的时候，易于维护

## 对象 & 类

在大多面向对象的语言都使用 `class` 定义类，类就是对一系列对象的抽象，类好比是快递的外包装，类里面的内容就表示这个对象的定义；例如：

```java
class Javaer{
		
}
```

这样，就声明了一个名字为 `Javaer` 的类，在 Java 中，可以通过 new 来创建这个对象：

```java
Javaer javaer = new Javaer();
```

在 Java 中，类的命名要遵守***驼峰命名法***，

> 骆驼式命名法（Camel-Case）又称驼峰式命名法，是电脑程式编写时的一套命名规则（惯例）。正如它的名称 CamelCase 所表示的那样，是指混合使用大小写字母来构成变量和函数的名字。程序员们为了自己的代码能更容易的在同行之间交流，所以多采取统一的可读性比较好的命名方式。

### 创建对象

在使用 Java 过程中，都是和对象打交道，Java 中的一切都可以看作是对象，但尽管如此，我们操作对象，却是对对象的**引用**:

```java
Door key;
```

如上代码中创建的只是引用，并非一个对象；如果要想正确的创建一个对象并且不在编译的过程中出错，那么就需要在创建对象引用时同时把一个对象赋给它。

```java
Door key = new Door();
```

在 Java 中，一旦创建了一个引用，就希望它能与一个新的对象进行关联，通常使用 `new` 操作符来实现这一目的。

### 属性和方法

对于一个类，最基本的要素就是要有**属性**和**方法**；

属性又被称为字段：

```java
Class Door {
	int a;
	Key key;
}
```

方法就是函数，方法的基本组成包括 **方法名称、参数、返回值和方法体**

```java
public int get(){
	return 1;
}
```

在上面的示例中，`get` 是方法名称，`()` 内是方法接收的参数、`return ` 后面的是方法的返回值（如果是 void 方法，则不需要返回值）,`{}` 里的是方法体。

#### 构造方法

Java 中有一种特殊的**构造方法**，又被叫做构造器或者构造函数。构造方法就是在创建对象的时候保证每个对象都被初始化。构造方法只在创建对象的时候调用一次，它没有参数类型和返回值，它的名称要和类名保持一致，并且构造方法可以有多个。

```java
Class Door{
	int number;
	String color;
	
	public Door(){}
	public Door(int number){}
	public Door(String color){}
	public Door(int number,String color){}
}
```

上述代码中定义了一个 `Door` 类，但是却没有参数类型和返回值，而且有多个同名的 Door() 方法，每个方法的参数列表也不同，其实这是面向对象特征--**多态**的体现，后文会介绍。定义好构造方法后，我们就能创建一个 `Door` 对象了。

```java
class creatDoor{
	 public static void main(String[] args) {
         Door door1 = new Door();
         Door door2 = new Door(1);
         Door door3 = new Door("yellow");
         Door door4 = new Door(1,"yellow");
     }
}
```

如果类中没有定义任何构造方法，那么 JVM 会为你自动生成一个构造方法，如下

```java
class Door {
  
  int number;
  String color;
  
}

class createDoor {

    public static void main(String[] args) {
        Door door1 = new Door();
    }
}
```

默认的构造方法也被称为默认构造器或者无参构造器。

这里需要注意一点的是，即使 JVM 会为你默认添加一个无参的构造器，但是如果你手动定义了任何一个构造方法，***JVM 就不再为你提供默认的构造器，你必须手动指定，否则会出现编译错误。***

#### 方法重载

重载在 Java 中是一个很重要的概念，它是类名的不同表现形式，上文提到的构造函数，是重载的一种，另外一种重载就是方法的重载。

```java
public class Door {

    int number;
    String color;

    public Door(){}
    public Door(int number){}
    
    public int getDoor(int number){
        return 1;
    }
    
    public String getDoor(String color){
        return "color";
    }

}
```

如上所示，有 Door 的构造函数的重载，也有 getDoor 方法的重载。

如果有几个相同的方法名字，Java 如何知道你调用的是哪个方法呢？

**每个重载的方法都有独一无二的参数列表**。其中包括参数的类型、顺序、参数数量等，满足一种一个因素就构成了重载的必要条件：

- 方法名称必须相同。
- 参数列表必须不同（个数不同、或类型不同、参数类型排列顺序不同等）。
- 方法的返回类型可以相同也可以不相同。
- 仅仅返回类型不同不足以成为方法的重载。
- 重载是发生在编译时的，因为编译器可以根据参数的类型来选择使用哪个方法。

#### 方法的重写

方法重写的描述是对`子类和父类`之间的。

而重载是发生同一个类中的。

```java
class Food {
 
	public void eat(){
    System.out.printl('eat food');
  }
}

class Fruit extends Food{
  
  @Override
  public void eat(){
    System.out.printl('eat fruit');
  }
}
```

上述代码中，就含有重写的范例，子类 Fruit 中的方法和父类 Food 的方法相同

所以重写的标准是：

- 重写的方法必须要和父类保持一致，包括**返回值类型,方法名,参数列表** 也都一样。
- 重写的方法可以使用 `@Override` 注解来标识
- 子类中重写方法的访问权限不能低于父类中方法的访问权限。

### 初始化

我们在创建一个对象，使用 new 创建对象的时候，实际上是调用了这个对象无参数的构造方法进行的初始化。这个无参数的构造函数可以隐藏，由 JVM 自动添加。也就是说，构造函数能够确保类的初始化。

```java
class Door{
  public Door(){}
}
```

#### 成员初始化

成员初始化有两种形式

1. 编译器默认指定的字段初始化，基本数据类型的初始化	

|  类型   |  初始值  |
| :-----: | :------: |
| boolean |  false   |
|  char   |  /u0000  |
|  byte   | (byte)0  |
|  short  | (short)0 |
|   int   |    0     |
|  long   |    0L    |
|  float  |   0.0f   |
| double  |   0.0d   |

其他数据类型，如 String ，其初始值默认为 null 

2. 指定数值的初始化

```java
int a = 11;
```

#### 构造器初始化

构造器可以用来对某些方法和某些动作进行初始化，从而确定初始值：

```java
public class Number{
  int n;
  public Number(){
    n = 11;
  }
}
```

利用构造函数，能够把 n 的值初始化为 11。

#### 初始化顺序

- 静态属性：static 开头定义的属性
- 静态方法块： static {} 包起来的代码块
- 普通属性： 非 static 定义的属性
- 普通方法块： {} 包起来的代码块
- 构造函数： 类名相同的方法
- 方法： 普通方法

要验证初始化的顺序，最好的方法就是写代码实践得到

```Java
public class TheOrder {
        // 静态属性
        private static String staticField = getStaticField();
        // 静态方法块
        static {
            System.out.println(staticField);
            System.out.println("静态方法块初始化");
        }
        // 普通属性
        private String field = getField();
        // 普通方法块
        {
            System.out.println(field);
        }
        // 构造函数
        public TheOrder() {
            System.out.println("构造函数初始化");
        }

        public static String getStaticField() {
            String statiFiled = "Static Field Initial";
            return statiFiled;
        }

        public static String getField() {
            String filed = "Field Initial";
            return filed;
        }
        // 主函数
        public static void main(String[] argc) {
            new TheOrder();
        }
}

```

最终输出结果为

```java
Static Field Initial
静态方法块初始化
Field Initial
构造函数初始化
```

那么初始化顺序就是：

1. 静态属性初始化 
2. 静态方法块初始化 
3. 普通属性初始化 
4. 普通方法块初始化 
5. 构造函数初始化

### this 和 super

`this` 和 `super` 都是 Java 中的关键字

#### this 

`this` 这个关键字只能用在方法的方法体内。当一个对象创建后，JVM 就会给这个对象分配引用自己的指针，这个指针的名字就叫做 `this` 。

`this` 表示的当前的对象，可以用来调用方法、属性以及对象本身。

```java
public class Test{
  private int number;
  private String username;
  private String password;
  private int x = 100;
  
  public Test(int n){
    number = n;//这个可以写为this.number = n;
  }

  public Test(int i, String username, String password){
  //成员变量和参数同名，成员变量被屏蔽，用"this.成员变量"的方式访问成员变量.
    this.username = username;
    this.password = password;
  }

  //默认不带参数的构造方法
  public Test(){
    this(0, "未知", "空")；//通过this调用另外一个构造方法.  
  }

  public Test(String name){
    this(1, name, "空")；
  /*
   * 通过this调用另外一个构造方法. 虽然上面的两种构造方法都是编译通过的。但是并没有实际的意义。一般我们会在参数多的构造函数里面去用this调用参数少的构造函数（并且只能放在方法体里面的第一行）。
   * 示例里面的这种构造方法就相当于给了三个参数（其中两个参数已经定了，另一个参数在这个构造方法传入）。 
   */
  }

  public Test(int i, String username){
    this(i, username, null);//通过this调用另外一个构造方法.  
  }

}
```

`this` 使用方法总结

1. 通过 this 调用本类中另一个构造方法，用法是 this (参数列表)，这个仅仅在类的构造方法中可以用，并且只能放在类的构造方法的方法体的第一句。别的地方不能用。
2. 方法参数或者方法中的局部变量和成员变量同名的情况下，成员变量被屏蔽，此时要访问成员变量则需要用 “this.变量名”的方式来引用变量。但是，在没有同名的情况，可以直接用成员变量的名字，而不用 this ，用了也是正确的（看起来更加的直观）。
3. 在方法中，需要引用该方法所属类的当前对象的时候，直接用 this.

#### super

1. 在子类的构造方法中要调用父类的构造函数，用 “super(参数列表)”的方式调用，参数不是必须的（如果子类的构造方法没有某个参数，但是父类的构造函数里面有这个参数，那么在 super() 的参数列表里的相应位置，传入参数 null ）。
2. 如果子类重写了父类的某一个方法。（也就是子类和父类有相同的方法定义，但是有不同的方法体），此时，我们可以通过 "super.成员方法名"来调用父类里面的这个方法。

## 面向对象三大特性

### 封装

封装在 Java 中又称访问控制权限，访问控制权限其实最核心就是一点：只对需要的类可见。

Java 中成员的访问权限共有四种，分别是 **public、protected、default、private**，它们的可见性如下

|                 | 同一个类 | 同一个包 | 不同包的子类 | 不同包的非子类 |
| :-------------: | :------: | :------: | :----------: | :------------: |
|     public      |    √     |    √     |      √       |       √        |
|    protected    |    √     |    √     |      √       |                |
| 默认（default） |    √     |    √     |              |                |
|     private     |    √     |          |              |                |

### 继承

继承是所有 OOP 语言和 Java 语言不可缺少的组成部分。

继承是 Java 面对对象编程技术的一块基石，是面对对象的三大特征之一，也是实现软件复用的重要手段，继承可以理解为一个对象从另一个对象获取属性的过程。

当我们准备编写一个类时，发现某个类已有我们所需要的成员变量和方法，假如我们想复用这个类的成员变量和方法，即在所编写类中不用声明成员变量就相当于有了这个成员变量，不用定义方法就相当于有了这个方法，那么我们可以将编写的类声明为这个类的子类即继承。

> **源类，基类，超类**或者**父类**都是一个概念
>
> **导出类，继承类，子类**也都是同一个概念

继承中最常使用的两个关键字是 **extends** 和 **implements** 。

这两个关键字的使用决定了一个对象和另一个对象是否是 IS-A (是一个)关系。

通过使用这两个关键字，我们能实现一个对象获取另一个对象的属性。

所有 Java 的类均是由 java.lang.Object 类继承而来的，所以 Object 是所有类的祖先类，而除了 Object 外，所有类必须有一个父类。

#### 继承的语法

```java
class Father{
    private int i;
    protected int j;
    
    public void func(){
        
    }
}

class Son extend Father{
	public int k;
    
    public void func(){}
}
```

###### **注意：**如类声明语句中没有extends子句，则该类为java.lang包中的Object的子类。这就说明了java中的代码其实都有一个继承的关系，只不过是继承Object这个java中最根本的父类。

#### 继承的特点

1. 子类拥有父类非private的属性和方法，子类继承的父类方法和成员变量可以当作自己的方法和成员变量一样被子类的实例方法使用。
2. 子类可以有自己属性和方法，即子类可以在父类的基础上对父类进行扩展。
3. 子类可以用自己的方式实现父类的方法。（重写或者覆盖） 

### 多态

多态是同一个行为具有多个不同表现形式或形态的能力。多态就是同一个接口，使用不同的实例而执行不同操作。

> 多态性是面向对象编程的又一个重要特征，它是指在父类中定义的属性和方法被子类继承之后，可以具有不同的数据类型或表现出不同的行为，这使得同一个属性或方法在父类及其各个子类中具有不同的含义。
>     对面向对象来说，多态分为编译时多态和运行时多态。其中编译时多态是静态的，主要是指方法的重载，它是根据参数列表的不同来区分不同的方法。通过编译之后会变成两个不同的方法，在运行时谈不上多态。而运行时多态是动态的，它是通过动态绑定来实现的，也就是大家通常所说的多态性。 

###### e.g.  彩色打印机和黑白打印机都是打印机，但是他们打印出来的东西颜色不一样。这就是多态

#### 多态实现的条件

1. **继承**：在多态中必须存在有继承关系的子类和父类。
2. **重写**：子类对父类中某些方法进行重新定义，在调用这些方法时就会调用子类的方法，如果没有重写的话在父类中就会调用。当子类对象调用重写的方法时，调用的是子类的方法，而不是父类中被重写的方法。要想调用父类中被重写的方法，则必须使用关键字 **super**。
3. **向上转型**：在多态中需要将子类的引用赋给父类对象，只有这样该引用才既能可以调用父类的方法，又能调用子类的方法。 

```java
public class Person {
	
	public void drink(){
		System.out.println("喝咖啡");
	}
	
	public void eat(){
		System.out.println("吃点东西");
	}
}

public class Man extends Person {
	public void drink(){
		System.out.println("喝冰美式");
	}
}

```

```java
public class Test {
	public static void main(String[] args) {
		Person p = new Man();
		p.drink();
		p.eat();
	}
}
```

```java
输出：
喝冰美式
吃点东西
```

#### 多态的优点

1. 消除类型之间的耦合关系
2. 可替换性
3. 可扩充性
4. 接口性
5. 灵活性
6. 简化性

## 接口和抽象类

### 接口

接口（英文：Interface），在 Java 编程语言中是一个抽象类型，是抽象方法的集合，接口通常以 `interface` 来声明。一个类通过继承接口的方式，从而来继承接口的抽象方法。

接口并不是类，编写接口的方式和类很相似，但是它们属于不同的概念。类描述对象的属性和方法。接口则包含类要实现的方法。除非实现接口的类是抽象类，否则该类要定义接口中的所有方法。

接口无法被实例化，但是可以被实现。一个实现接口的类，必须实现接口内所描述的所有方法，否则就必须声明为抽象类。

#### 接口和类相似点：

- 一个接口可以有多个方法。
- 接口文件保存在.java结尾的文件中，文件名使用接口名。
- 接口的字节码文件保存在.class结尾的文件中。
- 接口相应的字节码文件必须在与包名称相匹配的目录结构中。

#### 接口和类的区别：

- 接口不能用于实例化对象。
- 接口没有构造方法。
- 接口中所有的方法必须是抽象方法。
- 接口不能包含成员变量，除了static和final变量。
- 接口不是被类继承了，而是要被类实现。
- 接口支持多重继承。

#### 接口的声明

```java
[可见度] interface 接口名称 [extends 其他类名]{
	// 声明变量
	// 抽象方法
}
```

范例：

```java
public interface TestInterface{
    public void eat();
    public void drink();
}
```

#### 接口的实现

类实现接口的时候，类要实现接口中所有的方法。否则，类必须声明为抽象的类。

范例：

```java
public class Test implements TestInterface{
	public void eat(){
		System.out.println("吃东西");
	}
	
	public void drink(){
		System.out,println("喝水");
	}
	
	public static void main(String args[]){
		Test t = new Test();
		t.eat();
		t.drink();
	}
}
```

在实现接口的时候，要注意：

- 一个类可以同时实现多个接口。
- 一个类只能继承一个类，但是能实现多个接口。
- 一个接口能继承另一个接口，这和类之间的继承比较相似。

### 抽象类

> **一个类中没有包含足够的信息来描绘一个具体的对象，这样的类就是抽象类。**
>
> 抽象类除了不能实例化对象之外，类的其它功能依然存在，成员变量、成员方法和构造方法的访问方式和普通类一样。
>
> 由于抽象类不能实例化对象，所以抽象类必须被继承，才能被使用。也是因为这个原因，通常在设计阶段决定要不要设计抽象类。
>
> 父类包含了子类集合的常见的方法，但是由于父类本身是抽象的，所以不能使用这些方法。

在 Java 中用 abstract class 来定义抽象类。

```java
public interface Test {

    void ColorTest();

}

abstract class WhiteDog implements Dog{

    public void ColorTest(){
        System.out.println("Color is white");
    }

    abstract void MMiniTest();
}
```

在抽象类中，具有如下特征

- 如果一个类中有抽象方法，那么这个类一定是抽象类，也就是说，使用关键字 `abstract` 修饰的方法一定是抽象方法，具有抽象方法的类一定是抽象类。实现类方法中只有方法具体的实现。
- 抽象类中不一定只有抽象方法，抽象类中也可以有具体的方法，你可以自己去选择是否实现这些方法。
- 抽象类中的约束不像接口那么严格，你可以在抽象类中定义 **构造方法、抽象方法、普通属性、方法、静态属性和静态方法**
- 抽象类和接口一样不能被实例化，实例化只能实例化`具体的类`