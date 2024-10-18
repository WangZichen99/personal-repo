# String

## String为什么不可变？

1. String类用final关键字修饰，无法被继承，所以子类不会修改
2. String数据存储在char[]中，也被final所修饰，所以无法修改

这样做的好处？

1. 防止被篡改，保证安全性
2. 第一次计算字符串的哈希值后缓存，再次使用哈希值时可以直接使用缓存
3. 实现字符串常量池，相同的字符串的变量指向的是同一个字符串对象

## 字符串常量池

`String s = new String("Hello World!");`

当使用new关键字创建字符串对象时，会先检查字符串常量池中是否有字符串对象，如果没有，在常量池中创建一个对象，然后在堆上再创建一个对象，将变量指向堆上的对象。

`String s = "Hello World!";`

如果使用双引号直接赋值，会检查常量池中是否有字符串对象，如果没有，在常量池中创建，返回常量池中对象的地址，如果已经有对象，那么直接返回常量池中已有对象的地址。

### 字符串常量池的位置

在Java7之前，字符串常量池位于永久代中，永久代是堆内存的一部分，大小固定，用于存储类信息、方法信息、常量池信息等静态的数据。这就导致了一个问题，由于永久代大小固定且占用堆内存的一部分空间，所以当永久代空间不够用的时候就会出现OOM错误。

![[img-20240820125307006.png]]

为了解决这个问题，从Java7开始，将常量池放入堆内存中。

![[img-20240820125248928.png]]

Java8之后永久代取消，取而代之的是元空间，它存储的数据与永久代一样，不同的是不再占用JVM的堆内存，而是使用计算机的内存空间。并且元空间的大小可变，这就使得元空间更加灵活，不会再导致OOM错误，并且可以避免堆内存的碎片化问题。元空间的垃圾回收与堆内存的垃圾回收分离，避免因为元空间的数据而导致堆内存的gc。

![[img-20240820125306044.png]]

## String.intern()

intern方法的作用是将字符串对象放入字符串常量池中。注意：由于不同版本字符串常量池的实现不同，所以intern方法的具体操作也不同。在java7之前，执行该方法后会在常量池中创建一个新的字符串对象。Java7将常量池放入堆内存中，所以在执行该方法后需要先检查堆内存中是否已经存在字符串对象，若存在，则直接在常量池中保存对象的引用，若不存在，则在常量池中创建一个新的字符串对象。这样避免了堆内存与常量池对象的重复。

```java
String s1 = new String("二哥三妹");
String s2 = s1.intern();
System.out.println(s1 == s2); //false
```

对于例1，先在常量池中创建字符串对象，然后在堆内存中创建字符串对象，然后intern方法会判断常量池中是否存在字符串对象，此时是存在的，所以返回常量池中的字符串对象，所以s1是堆上的对象，s2是常量池中的对象。

![[img-20240820125306197.png]]

```java
String s1 = new String("二哥") + new String("三妹");
String s2 = s1.intern();
System.out.println(s1 == s2); //true
```

对于例2，在常量池和堆上创建“二哥”和“三妹”两个字符串对象后，对于两个字符串对象相加，会创建一个StringBuilder对象将两个字符串拼接形成一个新的字符串对象“二哥三妹”存放在堆中。执行intern方法后由于堆中已经存在了字符串对象，所以在常量池中会保存堆中对象的引用。所以s1和s2是相同的引用。

![[img-20240820125306373.png]]

## StringBuilder和StringBuffer

StringBuffer添加了synchronized关键字来保证线程安全，StringBuilder没有添加，除此之外两者几乎没有其他差别，在实际开发过程中使用StringBuilder的频率也要远高于StringBuffer。

StringBuilder底层使用char[]数组进行存储，在初始化时，数组容量为16。

appnd方法中需要先判断是否需要扩容，扩容策略为新容量 = 旧容量的2倍 + 2，然后使用Arrays.copyOf将原字符数组拷贝到新数组中。

## equals()和==

在比较字符串时，使用equals方法比较字符串的内容是否相等，==比较两个对象的地址值是否相等。equals方法是Object类的方法，所有Object的子类都可以重写自己的equals方法，String类就重写了equals方法来比较两个字符串的内容是否相等。

## +号操作符的本质

在使用+号拼接字符串时，编译器会进行优化，创建一个StringBuilder对象并使用append方法来拼接字符串，需要注意的是在循环中不要使用+号来拼接字符串，这会导致创建大量的StringBuilder对象，并且可能频繁触发GC，造成资源浪费和性能降低。

## 正则表达式

[https://github.com/cdoco/learn-regex-zh](https://github.com/cdoco/learn-regex-zh)

# 面向对象

## Object类

![[img-20240820125306604.png]]

### hashCode

用于返回对象的哈希码，相等的对象的哈希码必须相等，所以在重写equals方法后必须重写hashCode方法。

### equals

用于比较两个对象的地址是否相等，如果两个对象相等的逻辑不是地址相等，需要重写该方法

### clone

返回对象的一个副本（浅拷贝），对象需要实现Cloneable接口

### toString

默认返回“类名@哈希码”

### wait/notify

调用wait方法会导致当前线程进入等待，直到其他线程调用此对象的notify方法或notifyAll方法。

notify方法会唤醒在此对象监视器上等待的单个线程，如果有多个线程等待，选择一个线程唤醒。

notifyAll方法将此对象监视器上的所有线程唤醒。

wait方法的重载方法可以设定等待时间，如果timeout毫秒内没有被唤醒，则自动唤醒。

### getClass

用于获取对象的类信息

### finalize

不推荐使用，Java9已经移除

## Java变量

### 局部变量

在方法体内声明的变量为局部变量，该变量只能在方法内使用

```java
public static void main(String[] args) {
    int a = 10;
    int b = 10;
    int c = a + b;
    System.out.println(c);
}
```

局部变量的注意事项：

- 局部变量生命在方法、构造方法或语句块中
- 在方法内被执行时创建，方法执行完后被销毁
- 访问修饰符（private、public...）不能用于局部变量
- 局部变量创建在栈上
- 局部变量没有默认值，必须进行初始化后才能使用（与是否为基本数据类型无关）

### 成员变量

在类内部方法体外声明的变量为成员变量

```java
public class InstanceVariable {
    int data = 88;
    public static void main(String[] args) {
        InstanceVariable iv = new InstanceVariable();
        System.out.println(iv.data); // 88
    }
}
```

成员变量的注意事项：

- 成员变量声明在类中，但在方法和语句块之外
- 当一个对象被实例化后，成员变量的值就被确定
- 成员变量在对象创建的时候创建，在对象销毁的时候销毁
- 成员变量可以声明在使用前或使用后
- 访问修饰符可以修饰成员变量
- 成员变量有默认值，数值型变量默认值为0，布尔型变量默认值为false，引用类型变量默认值为null
- 对象存储在堆内存中，成员变量存储在对象中

### 静态变量

静态变量以static关键字声明，在类中，方法和语句块之外

```java
public class StaticVariable {
    static int data = 99;
    public static void main(String[] args) {
        System.out.println(StaticVariable.data); // 99
    }
}
```

静态变量的注意事项：

- 静态变量在类中用static修饰，必须在方法和语句块之外
- 静态变量属于类，无论有多少个对象，静态变量都只有一份
- 静态变量一般被声明为常量使用
- 静态变量存储在静态存储区
- 静态变量在程序开始时创建，在程序结束时销毁
- 静态变量默认值与成员变量一致（数值型变量默认值为0，布尔型变量默认值为false，引用类型变量默认值为null）

### 常量

final关键字修饰的变量为常量，一旦初始化就不可更改

```java
public class FinalVariable {
    final String CHEN = "沉";
    static final String MO = "默";
    public static void main(String[] args) {
        FinalVariable fv = new FinalVariable();
        System.out.println(fv.CHEN);
        System.out.println(MO);
    }
}
```

## Java可变参数

可变参数实际上是一个数组，数组的大小就是可变参数的个数，然后将数组作为参数，调用可变参数的方法，这就是为什么可以使用数组作为实参传递的根本原因。

```java
public static void main(String[] args) {
    print(new String[]{"沉"});
    print(new String[]{"沉", "默"});
    print(new String[]{"沉", "默", "王"});
    print(new String[]{"沉", "默", "王", "二"});
}

public static void print(String... strs) {
    for (String s : strs)
        System.out.print(s);
    System.out.println();
}
```

在String类的format方法中就是用了可变参数。同样的，在打印日志时也是用了可变参数。

```java
System.out.println(String.format("年纪是: %d", 18));
System.out.println(String.format("年纪是: %d 名字是: %s", 18, "沉默王二"));

protected Logger logger = LoggerFactory.getLogger(getClass());
logger.debug("名字是{}", mem.getName());
logger.debug("名字是{}，年纪是{}", mem.getName(), mem.getAge());
```

## Java构造方法

### 创建规则

- 构造方法名必须与类名保持一致。
- 构造方法没有返回值，void也不行。如果定义了与类名相同的void方法，被认为是普通方法而不是构造方法。
- 构造方法不能是抽象的（abstract），因为抽象方法没有方法体，无法创建对象实例。
- 构造方法不能是最终的（final），因为构造方法不能被继承，所以使用final没有意义。
- 构造方法不能是静态的（static），因为构造方法用于初始化一个对象，静态方法无法访问非静态成员变量，所以构造方法不能是静态的。
- 同步的（synchronized），因为构造方法中不存在共享资源的安全访问，所以synchronized没有必要。
- 可以使用访问权限修饰符来修饰构造方法。

## Java访问权限修饰符

- public：方法可以被所有类访问
- protected：方法可以被同一包中的类或不同包中的子类访问
- private：方法只能在类中访问
- default：如果方法没有被前面三个修饰，那么默认只能被同一包中的类访问

对于类来说，只可以使用public或默认访问权限修饰

```java
public class Wanger{}

class Wanger{}
```

对于方法和变量来说，所有访问修饰符都可以使用

## Java代码块

```java
public class Car {
    Car() {
        System.out.println("构造方法");
    }

    {
        System.out.println("代码初始化块");
    }

    public static void main(String[] args) {
        new Car();
    }
}
//代码初始化块
//构造方法
```

构造方法在执行时会将代码块放在构造方法的其他代码之前执行，如下图所示

![[img-20240820125308054.png]]

也就是说每次创建对象都会执行一次代码初始化块。

如果子类继承了父类，那么在创建子类对象时的执行顺序是父类构造方法->子类代码块->子类构造方法

```java
public class Father {
    static {
        System.out.println("Father静态代码块");
    }
    {
        System.out.println("Father代码块");
    }
    Father() {
        System.out.println("Father构造方法");
    }
}

public class Child extends Father{
    static {
        System.out.println("Child静态代码块");
    }
    {
        System.out.println("Child代码块");
    }
    Child() {
        System.out.println("Child构造方法");
    }

    public static void main(String[] args) {
        Child child = new Child();
    }
}

/*
Father静态代码块
Child静态代码块
Father代码块
Father构造方法
Child代码块
Child构造方法
*/
```

## Java抽象类

- 如果一个类定义了一个或多个抽象方法，那么这个类必须为抽象类。抽象类中可以包含非抽象方法。
- 抽象类最好以Abstract或Base开头进行命名
- 抽象类可以被继承，其子类必须实现父类的抽象方法
- 抽象类可以实现接口

### 抽象类的应用场景

1. 当一个通用的功能需要被多个子类复用的时候，可以将方法写在抽象类中。

```java
abstract class AbstractPlayer {
    public void sleep() {
        System.out.println("运动员也要休息而不是挑战极限");
    }
}

class BasketballPlayer extends AbstractPlayer {
}

class FootballPlayer extends AbstractPlayer {
}

BasketballPlayer basketballPlayer = new BasketballPlayer();
basketballPlayer.sleep();
FootballPlayer footballPlayer = new FootballPlayer();
footballPlayer.sleep();
```

2. 当一个功能需要被子类各自进行不同的实现的时候，可以在抽象类中声明一个抽象方法。

```java
abstract class AbstractPlayer {
    abstract void play();
}

class BasketballPlayer extends AbstractPlayer {
    @Override
    void play() {
        System.out.println("我是张伯伦，我篮球场上得过 100 分，");
    }
}

class FootballPlayer extends AbstractPlayer {
    @Override
    void play() {
        System.out.println("我是C罗，我能接住任意高度的头球");
    }
}
```

## Java接口

- 接口中定义的变量在编译后会自动加上public static final，也就是说接口中定义的都是常量。
- 接口中可以定义private（Java9开始支持）、default（Java8开始支持）、static（Java8开始支持）方法
- 接口中的方法不能被protected、final所修饰，因为方法必须被实现。
- 未被private、default、static修饰的方法会默认加上public abstract声明为公共抽象方法，接口上会默认添加abstract。

![[img-20240820125307769.png]]

### 接口的作用

1. 使得实现类具有接口的功能。比如实现了Cloneable接口的类具有拷贝功能
2. 通过接口可以实现多继承。如果有两个类同时继承了一个父类并重写了父类的方法，那么当一个类继承这两个子类的时候，无法判断调用哪个子类的方法，这就是菱形问题。![[img-20240820125307403.png]]
3. 实现多态。多态就是父类引用指向子类对象，多态可以通过继承实现，也可以通过接口实现。

```java
public interface Shape {
    String name();
}

public class Circle implements Shape {
    @Override
    public String name() {
        return "圆";
    }
}

public class Square implements Shape {
    @Override
    public String name() {
        return "正方形";
    }
}

public class Test() {
    public static void main(String[] args)
    {
        List<Shape> shapes = new ArrayList<>();
        Shape circleShape = new Circle();
        Shape squareShape = new Square();
        
        shapes.add(circleShape);
        shapes.add(squareShape);
        
        for (Shape shape : shapes) {
            System.out.println(shape.name());
        }
    }
}

```

### 接口的三种模式

#### 策略模式

```java
interface Coach {
    // 方法：防守
    void defend();
}

// 何塞·穆里尼奥
class Hesai implements Coach {
    @Override
    public void defend() {
        System.out.println("防守赢得冠军");
    }
}

// 德普·瓜迪奥拉
class Guatu implements Coach {
    @Override
    public void defend() {
        System.out.println("进攻就是最好的防守");
    }
}

public class Demo {
    // 接口作为参数
    public static void defend(Coach coach) {
        coach.defend();
    }
    
    public static void main(String[] args) {
        // 为同一个方法传递不同的对象
        defend(new Hesai());
        defend(new Guatu());
    }
}
```

Demo类中声明了静态方法defend，并且以接口作为参数，这样当传入不同的接口实现类的对象时，就能完成不同的行为，这被称为策略模式。

#### 适配器模式

```java
interface Coach {
    void defend();
    void attack();
}

// 抽象类实现接口，并置空方法
abstract class AdapterCoach implements Coach {
    public void defend() {};
    public void attack() {};
}

// 新类继承适配器
class Hesai extends AdapterCoach {
    public void defend() {
        System.out.println("防守赢得冠军");
    }
}

public class Demo {
    public static void main(String[] args) {
        Coach coach = new Hesai();
        coach.defend();
    }
}
```

在接口中定义了两个方法defend和attack方法，如果不想在实现类中实现接口的全部方法，那么可以在接口与实现类中加一层适配器类，适配器类实现接口的所有方法，但是方法体为空，最终的实现类去继承适配器类，并且实现想要实现的方法。

#### 工厂模式

```java
package com.wzc.test;

public interface Coach {
    // 教练的接口
    void teach();
}

interface CoachFactory {
    Coach getCoach();
}

class SwimmingCoach implements Coach {
    @Override
    public void teach() {
        System.out.println("教授游泳技能");
    }
}

class SwimmingCoachFactory implements CoachFactory {
    @Override
    public Coach getCoach() {
        return new SwimmingCoach();
    }
}

class FootballCoach implements Coach {
    @Override
    public void teach() {
        System.out.println("教授足球技能");
    }
}

class FootballCoachFactory implements CoachFactory {
    @Override
    public Coach getCoach() {
        return new FootballCoach();
    }
}

class BasketballCoach implements Coach {
    @Override
    public void teach() {
        System.out.println("教授篮球技能");
    }
}

class BasketballCoachFactory implements CoachFactory {
    @Override
    public Coach getCoach() {
        return new BasketballCoach();
    }
}

class Main {
    public static Coach getCoach(CoachFactory factory) {
        return factory.getCoach();
    }
    
    public static void main(String[] args) {
        Coach swimmingCoach = getCoach(new SwimmingCoachFactory());
        swimmingCoach.teach(); // 输出：教授游泳技能

        Coach footballCoach = getCoach(new FootballCoachFactory());
        footballCoach.teach(); // 输出：教授足球技能

        Coach basketballCoach = getCoach(new BasketballCoachFactory());
        basketballCoach.teach(); // 输出：教授篮球技能
    }
}
```

### 抽象类和接口的区别

1. 可以理解为接口是从抽象类中抽象出来的，对于普通方法来说，抽象类可以包含非抽象方法，但接口中的普通方法默认都是抽象的。
2. 接口中的成员变量隐式的声明为static final，但抽象类不是。
3. 抽象类中可以包含静态代码块，但接口中不行。
4. 一个类可以实现多个接口，但只能继承一个抽象类。
5. 抽象类是对事物的抽象，包含事物的属性和行为，如果一个类属于抽象类所表示的事物就应该继承这个抽象类，抽象类表示的是“是不是”，比如苹果是水果，苹果就继承了水果。而接口则是对行为的抽象，只有当一个事物具有这个接口所定义的行为时才需要实现这个接口，表示的是“有没有”，比如鸟具有飞的行为，鸟就实现了飞这个接口。

## 继承封装多态

### 实现多继承的几种方法：

1. 内部类可以继承一个与外部类无关的类，保证了内部类的独立性，可以实现多继承的效果
2. 多层继承，C继承B，B继承A，那么就相当于C同时继承了B和A![[img-20240820125307956.png]]
3. 实现接口

### 子类与父类的一些关系：

1. 继承后子类方法的访问权限修饰符不能比父类的访问权限修饰符的作用域更小。子类的权限访问只能更加宽松。
2. 子类中抛出的异常只能是父类的异常或父类异常的子异常。
3. 子类可以继承但原则上无法重写父类中static修饰的方法。因为static方法是属于类的方法，如果子类重写了父类的static方法，虽然编译器不会报错，但是子类重写的静态方法是属于子类的，而不是父类。**重写的目的在于根据对象类型不同而表现出多态，但是静态方法无法表现出多态，并且静态方法不需要创建对象调用，所以重写静态方法是没有意义的。**
4. 子类可以继承但无法重写父类中final修饰的方法。
5. 父类中abstract修饰的方法必须被子类重写。

### 子类与父类之间的转型：

#### 向上转型

父类引用指向子类对象时就会自动向上转型。例如`
`List<String> list = new ArrayList<>();`

父类引用指向子类对象后，只能使用父类声明的方法，如果子类重写了父类的方法会执行子类的重写方法。

![[img-20240820125306746.png]]

#### 向下转型

向下转型为强制转型，并且只有父类引用对象为子类对象时才会转型成功。向下转型后可以调用子类特有的方法。

![[img-20240820125307157.png]]

### 子父类初始化顺序

1. 父类静态成员变量和静态代码块
2. 子类静态成员变量和静态代码块
3. 父类普通成员变量和代码块，父类的构造方法
4. 子类的普通成员变量和代码块，子类的构造方法

### 多态

多态就是同一个类的对象在不同的情况下表现出不同的行为和状态。

多态的条件是子类继承并重写父类的方法和父类引用指向子类对象。当父类引用指向某个子类对象的时候执行的是这个子类的重写方法，当父类引用指向另一个子类对象的时候执行的是另一个子类的重写方法。所以多态指的是父类的方法是多态的。

```java
//子类继承父类
public class Wangxiaoer extends Wanger {
    public void write() { // 子类重写父类方法
        System.out.println("记住仇恨，表明我们要奋发图强的心智");
    }

    public static void main(String[] args) {
        // 父类引用指向子类对象
        Wanger[] wangers = { new Wanger(), new Wangxiaoer() };

        for (Wanger wanger : wangers) {
            // 对象是王二的时候输出：勿忘国耻
            // 对象是王小二的时候输出：记住仇恨，表明我们要奋发图强的心智
            wanger.write();
        }
    }
}

class Wanger {
    public void write() {
        System.out.println("勿忘国耻");
    }
}
```

多态与构造方法

当执行子类构造方法时会先执行父类构造方法，父类构造方法中由于子类重写了write方法，所以执行的是子类的重写方法，但是父类并不知道子类的age是多少，所以为int的初始化值0，然后才执行子类的构造方法。

```java
public class Wangxiaosan extends Wangsan {
    private int age = 3;
    public Wangxiaosan(int age) {
        this.age = age;
        System.out.println("王小三的年龄：" + this.age);
    }

    public void write() { // 子类覆盖父类方法
        System.out.println("我小三上幼儿园的年龄是：" + this.age);
    }

    public static void main(String[] args) {
        new Wangxiaosan(4);
//      上幼儿园之前
//      我小三上幼儿园的年龄是：0
//      上幼儿园之后
//      王小三的年龄：4
    }
}

class Wangsan {
    Wangsan () {
        System.out.println("上幼儿园之前");
        write();
        System.out.println("上幼儿园之后");
    }
    public void write() {
        System.out.println("老子上幼儿园的年龄是3岁半");
    }
}
```

## static关键字

静态方法中不能使用非静态变量和方法

## Java不可变对象

在Java中，常见的不可变对象有String、Integer等包装类型。

一个不可变类具有几个特点：

1. 类是final的，不可被继承
2. 所有成员变量都是final的，不可被修改
3. 不提供任何set方法修改成员变量
4. 所有方法返回的都是一个新的对象
5. 如果不可变类中包含了其他可变类，那么应该返回可变类对象的副本，而不是对象本身，这样外部就无法对可变类对象进行修改

## 重写和重载

### 重载

方法名相同，参数个数或参数类型不同的方法称为方法重载，在此基础上可以改变方法返回值和权限访问修饰符。

但是，不能只改变方法的返回值

![[img-20240820125307867.png]]

main方法也可以重载，但是不会被识别为main方法

![[img-20240820125306880.png]]

当重载方法参数与调用方法参数不能完全对应时，会进行隐式转换

![[img-20240820125307271.png]]

```java
public class OverloadingTypePromotion {
    void sum(int a, long b) {
        System.out.println(a + b);
    }

    void sum(int a, int b, int c) {
        System.out.println(a + b + c);
    }

    public static void main(String args[]) {
        OverloadingTypePromotion obj = new OverloadingTypePromotion();
        obj.sum(20, 20);
        obj.sum(20, 20, 20);
    }
}
```

### 重写

重写规则：

1. 只能重写继承过来的方法，父类的private方法不能被重写
2. final和static方法不能被重写，**重写的目的在于根据对象类型不同而表现出多态，但是静态方法无法表现出多态，并且静态方法不需要创建对象调用，所以重写静态方法是没有意义的。**
3. 重写方法必须具有相同的返回类型、方法名和参数列表
4. 重写的方法不能使用更严格的权限修饰符
5. 重写的方法不能抛出更高级别的异常
6. 可以在子类中通过super关键字调用父类中被重写的方法
7. 构造方法不能被重写
8. synchronized和strictfp对于重写没有任何影响

## Java注解

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD)
public @interface JsonField {
    public String value() default "";
}
```

使用@interface来声明一个注解，注解中包含一个value参数，默认为空字符串

@Retention(RetentionPolicy.RUNTIME)表示注解的声明周期是运行时有效

@Target(ElementType.FIELD)表示该注解是作用在字段上的

## Java枚举

```java
public enum PlayerType {
    TENNIS,
    FOOTBALL,
    BASKETBALL
}

//反编译后的枚举类
public final class PlayerType extends Enum
{

    public static PlayerType[] values()
    {
        return (PlayerType[])$VALUES.clone();
    }

    public static PlayerType valueOf(String name)
    {
        return (PlayerType)Enum.valueOf(com/cmower/baeldung/enum1/PlayerType, name);
    }

    private PlayerType(String s, int i)
    {
        super(s, i);
    }

    public static final PlayerType TENNIS;
    public static final PlayerType FOOTBALL;
    public static final PlayerType BASKETBALL;
    private static final PlayerType $VALUES[];

    static 
    {
        TENNIS = new PlayerType("TENNIS", 0);
        FOOTBALL = new PlayerType("FOOTBALL", 1);
        BASKETBALL = new PlayerType("BASKETBALL", 2);
        $VALUES = (new PlayerType[] {
            TENNIS, FOOTBALL, BASKETBALL
        });
    }
}
```

1. 枚举是一个final类无法被继承，并且在JVM中只有一个常量对象
2. 枚举继承了Enum类
3. 枚举中声明的所有字段都是static final修饰的
4. 枚举的values方法返回的是枚举数组的副本
5. 在静态代码块中进行枚举的初始化

# 集合框架

![[img-20240820125308162.png]]

## List

有序，可重复，可以存储null值

- 有序无序是指存储的数据在底层数组中的顺序不是按照添加顺序排列，而是hashcode决定
- 可否重复是根据equals方法判断，所以需要同时重写hashcode和equals方法

### ArrayList

1. 底层用Object数组实现，并且实现了RandomAccess接口，可以使用下标直接访问元素
2. 在不需要扩容的情况下，尾部插入和删除操作时间复杂度都是O(1)。需要扩容时尾部插入需要先扩容再插入，扩容的时间复杂度为O(n)
3. 首部插入和删除的时间复杂度为O(n)，因为插入时需要将所有元素向后移，删除时需要将所有元素向前移动
4. 在其他位置插入或删除的时间复杂度为O(n)，因为平均需要移动(n / 2)个元素
5. 非线程安全

#### 源码分析

1. 在创建一个ArrayList对象后，执行构造方法，如果没有指定对象的初始大小，则初始化一个空数组，如果指定大小，那么初始化对应大小的数组。
2. 添加元素时，判断是否为初始空数组，如果为空数组，那么第一次扩容大小为10，如果不是空数组，根据索引和数组容量大小来判断是否需要扩容。
3. 扩容机制：
	1. 空数组第一次扩容时，扩容大小为10
	2. 扩容原容量的1.5倍为新容量
	3. 检查新容量大小是否够用，如果不够，直接扩容到需要的容量
	4. 检查新容量是否超过数组最大容量(Integer.MAX_VALUE - 8)，如果超过，再检查需要的容量是否超过数组最大容量，如果超过扩容到Integer.MAX_VALUE，如果没超过，扩容到数组最大容量

### LinkedList

1. 底层用双向链表实现，意味着在存储空间中是不连续的，没有实现RandomAccess接口，无法通过下标访问元素，因为链表需要通过遍历后继节点找到元素
2. 因为LinkedList中存储了头部和尾部节点信息，所以首部插入删除和尾部插入删除时间复杂度都是O(1)
3. 其他位置插入和删除都需要移动到指定节点进行操作，所以时间复杂度为O(n)
4. 因为链表额外存储了前驱节点和后继节点，所以会占用更多的空间。实际开发中一般不会用到LinkedList，几乎可以用ArrayList代替，性能会更好
5. 非线程安全

![[img-20240820125307503.png]]

#### 源码分析

索引查找：根据索引判断元素属于链表的前半部分还是后半部分，如果在前半部分，则从first节点开始查找，如果在后半部分，则从last节点开始查找。充分利用了双向队列的特点

```java
// 获取链表指定位置的元素
public E get(int index) {
  // 下标越界检查，如果越界就抛异常
  checkElementIndex(index);
  // 返回链表中对应下标的元素
  return node(index).item;
}

// 返回指定下标的非空节点
Node<E> node(int index) {
    // 断言下标未越界
    // assert isElementIndex(index);
    // 如果index小于size的二分之一  从前开始查找（向后查找）  反之向前查找
    if (index < (size >> 1)) {
        Node<E> x = first;
        // 遍历，循环向后查找，直至 i == index
        for (int i = 0; i < index; i++)
            x = x.next;
        return x;
    } else {
        Node<E> x = last;
        for (int i = size - 1; i > index; i--)
            x = x.prev;
        return x;
    }
}
```

删除元素unlink方法：首先获取删除节点的前驱节点和后继节点，如果前驱节点为空，代表该节点为第一个节点，将fist指向删除节点的后继节点，如果不为空，将前驱节点的后继节点指向删除节点的后继节点，将删除节点的前驱节点置空，方便垃圾回收。如果后继节点为空，代表该节点为最后一个节点，将last指向前驱节点，如果不为空，将后继节点的前驱节点指向删除节点的前驱节点，将删除节点的后继节点置空，方便垃圾回收

```java
E unlink(Node<E> x) {
    // 断言 x 不为 null
    // assert x != null;
    // 获取当前节点（也就是待删除节点）的元素
    final E element = x.item;
    // 获取当前节点的下一个节点
    final Node<E> next = x.next;
    // 获取当前节点的前一个节点
    final Node<E> prev = x.prev;

    // 如果前一个节点为空，则说明当前节点是头节点
    if (prev == null) {
        // 直接让链表头指向当前节点的下一个节点
        first = next;
    } else { // 如果前一个节点不为空
        // 将前一个节点的 next 指针指向当前节点的下一个节点
        prev.next = next;
        // 将当前节点的 prev 指针置为 null，，方便 GC 回收
        x.prev = null;
    }

    // 如果下一个节点为空，则说明当前节点是尾节点
    if (next == null) {
        // 直接让链表尾指向当前节点的前一个节点
        last = prev;
    } else { // 如果下一个节点不为空
        // 将下一个节点的 prev 指针指向当前节点的前一个节点
        next.prev = prev;
        // 将当前节点的 next 指针置为 null，方便 GC 回收
        x.next = null;
    }

    // 将当前节点元素置为 null，方便 GC 回收
    x.item = null;
    size--;
    modCount++;
    return element;
}
```


## Set

### HashSet

### LinkedHashSet

### TreeSet

## Queue和Deque

Queue为单端队列，在一端插入，另一端删除

Deque为双端队列，两端都可以插入和删除

### LinkedList和ArrayDeque的区别

相同点：

- 两者都实现了Deque接口，可以看作是双向队列，两端都可以插入和删除元素

不同点：

- ArrayDeque底层使用数组和首尾指针实现，LinkedList使用链表实现
- ArrayDeque底层通过索引直接访问元素，LinkedList不可以
- ArrayDeque只存储数据，LinkedList还需存储节点信息，占用空间大
- ArrayDeque不能存储null值，LinkedList可以

### PriorityQueue

出队的顺序按照优先级由高到低顺序

## Map

键不可重复，值可以重复

### HashMap

- 无序
- 键和值都可以为null
- 底层用数组+链表+红黑树实现
- 非线程安全

#### 源码分析

在Java8之前，HashMap采用数组+链表的形式实现，从Java8开始采用数组+链表+红黑树的方式实现。初始数组大小为16。当添加key，value时，计算key的hash值来确定放到数组中的哪个位置。如果数组中的某个位置已经有entry（key，value），则将新的entry尾插到已有的entry的后面组成一个链表，当链表长度超过8时就会将链表转换为红黑树来增加查询效率。

**数组**
![[img-20240820125307675.png]]
**Java7实现**

![[img-20240820183019173.png]]

**Java8实现**

![[img-20240820183037407.png]]
##### 计算hash的方法

```java
static final int hash(Object key) {
  int h;
  // key.hashCode()：返回散列值也就是hashcode
  // ^：按位异或
  // >>>:无符号右移，忽略符号位，空位都以0补齐
  return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

如果key为null，则hash为0，也就是放在数组的第一个。将key的hashCode和hashCode右移16位做异或操作。因为hashcode是一个4字节int类型，所以右移16位刚好是取的32位hashcode的前一半，也就是高位16位，然后与原来的hashcode做异或操作，因为右移后的空位补0，所以就相当于是高位与低位做异或操作，**这样做的好处是是的计算出来的hash结果比较均匀，减少hash碰撞的发生。**

hash方法的改变使得在扩容后，entry在数组中的位置要么与扩容前保持不变，要么等于扩容前的位置 + 扩容前的数组长度。**这就解决了Java7在扩容后entry在数组中的位置改变的问题。**

##### 数组下标映射

在取值和存值时涉及到根据hash值映射数组下标，来看hashMap是怎么操作的：

```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict) {
    // 数组
    HashMap.Node<K,V>[] tab;
    // 元素
    HashMap.Node<K,V> p;

    // n 为数组的长度 i 为下标
    int n, i;
    // 数组为空的时候
    if ((tab = table) == null || (n = tab.length) == 0)
        // 第一次扩容后的数组长度
        n = (tab = resize()).length;
    // 计算节点的插入位置，如果该位置为空，则新建一个节点插入
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
}

final Node<K,V> getNode(int hash, Object key) {
    // 获取当前的数组和长度，以及当前节点链表的第一个节点（根据索引直接从数组中找）
    Node<K,V>[] tab;
    Node<K,V> first, e;
    int n;
    K k;
    if ((tab = table) != null && (n = tab.length) > 0 &&
            (first = tab[(n - 1) & hash]) != null) {
        // 如果第一个节点就是要查找的节点，则直接返回
        if (first.hash == hash && ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        // 如果第一个节点不是要查找的节点，则遍历节点链表查找
        if ((e = first.next) != null) {
            do {
                if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    // 如果节点链表中没有找到对应的节点，则返回 null
    return null;
}
```

关键代码是`(n - 1) & hash`，n为数组长度

其实这是一个取模运算，当$b = 2^n$时，存在如下公式：$a \% b = a \& (b-1)$

a对应的是hash值，b对应的就是数组长度，所以HashMap的长度为$2^n$

因为b为偶数，那么b-1为奇数，最低位一定为1，这表明$a \& (b - 1)$的结果取决于a，也就是key的hash值，这样取模后就将hash值映射到了数组下标。

##### 重要参数

```java
// 默认的初始容量是16
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;
// 最大容量
static final int MAXIMUM_CAPACITY = 1 << 30;
// 默认的负载因子
static final float DEFAULT_LOAD_FACTOR = 0.75f;
// 当桶(bucket)上的结点数大于等于这个值时会转成红黑树
static final int TREEIFY_THRESHOLD = 8;
// 当桶(bucket)上的结点数小于等于这个值时红黑树转成链表
static final int UNTREEIFY_THRESHOLD = 6;
// 桶中结构转化为红黑树对应的table的最小容量
static final int MIN_TREEIFY_CAPACITY = 64;
```

- loadFactor负载因子：默认值为0.75。控制数组存放数据的疏密程度，loadFactor越接近1，那么数组存放的entry就越多，发生hash重复的情况就会增加，所以链表的长度就会增加，这就会导致查找数据的效率降低，越接近0，存放的entry就越少，那么数组就越稀疏，这就会导致数组的利用率降低。

- threshold阈值：$threshold = 数组容量capacity * 负载因子loadFactor$，当$size > threshold$时，就会发生扩容，也就是说当数组的大小达到其容量的75%就会发生扩容。由初始容量16和负载因子0.75可以得出，当数组大小达到$16 * 0.75 = 12$时就会发生扩容。

##### 头插和尾插（多线程操作导致死循环的问题）

Java7在扩容时，原数组的元素采用头插法插入到新数组中，在多线程环境下，当操作原数组中的链表元素时，就可能出现某个已经完成插入的节点的next节点重新被插入到新数组中，由于采用头插法，next节点的next会指向已经完成插入的节点，这就导致了链表的死循环。在Java8中由于要判断链表的长度是否需要转为红黑树，所以选择了尾插法，也就解决了死循环的问题（因为尾插法的next节点为空，不可能循环）

另外还有put和put并发可能会导致元素丢失，put和get并发可能会get到null值的问题，所以在多线程环境下应该使用concurrentHashMap

##### 构造函数和get、put方法

```java
public HashMap() {  
    this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted  
}  

//注意下面两种构造方法即使指定了初始化容量 initialCapacity ，也只是通过 tableSizeFor 将其扩容到与 initialCapacity 最接近的 2 的幂次方大小，然后暂时赋值给 threshold ，后续通过 resize 方法将 threshold 赋值给 newCap 进行 table 的初始化。
public HashMap(int initialCapacity) {  
    this(initialCapacity, DEFAULT_LOAD_FACTOR);  
}  

public HashMap(int initialCapacity, float loadFactor) {  
    if (initialCapacity < 0)  
        throw new IllegalArgumentException("Illegal initial capacity: " +  
                                           initialCapacity);  
    if (initialCapacity > MAXIMUM_CAPACITY)  
        initialCapacity = MAXIMUM_CAPACITY;  
    if (loadFactor <= 0 || Float.isNaN(loadFactor))  
        throw new IllegalArgumentException("Illegal load factor: " +  
                                           loadFactor);  
    this.loadFactor = loadFactor;  
    this.threshold = tableSizeFor(initialCapacity);  
}  

static final int tableSizeFor(int cap) {  
    int n = cap - 1;  
    n |= n >>> 1;  
    n |= n >>> 2;  
    n |= n >>> 4;  
    n |= n >>> 8;  
    n |= n >>> 16;  
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;  
}
  
public HashMap(Map<? extends K, ? extends V> m) {  
    this.loadFactor = DEFAULT_LOAD_FACTOR;  
    putMapEntries(m, false);  
}
```

```java
public V get(Object key) {  
    Node<K,V> e;  
    return (e = getNode(hash(key), key)) == null ? null : e.value;  
}  
  
final Node<K,V> getNode(int hash, Object key) {  
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;  
    if ((tab = table) != null && (n = tab.length) > 0 &&  
        (first = tab[(n - 1) & hash]) != null) {  
        //如果hash相等，key也相等，直接返回first
        if (first.hash == hash && // always check first node  
            ((k = first.key) == key || (key != null && key.equals(k))))  
            return first;  
		//如果是数节点调用getTreeNode，如果是链表遍历查找
        if ((e = first.next) != null) {  
            if (first instanceof TreeNode)  
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);  
            do {  
                if (e.hash == hash &&  
                    ((k = e.key) == key || (key != null && key.equals(k))))  
                    return e;  
            } while ((e = e.next) != null);  
        }  
    }  
    return null;  
}
```

```java
public V put(K key, V value) {  
    return putVal(hash(key), key, value, false, true);  
}  
  
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,  
               boolean evict) {  
    Node<K,V>[] tab; Node<K,V> p; int n, i;  
    //如果数组为null或者容量为0，先扩容
    if ((tab = table) == null || (n = tab.length) == 0)  
        n = (tab = resize()).length;  
	//如果元素位置为空，直接放入
    if ((p = tab[i = (n - 1) & hash]) == null)  
        tab[i] = newNode(hash, key, value, null);  
    else {  
        Node<K,V> e; K k;  
        //如果存在重复key，直接覆盖
        if (p.hash == hash &&  
            ((k = p.key) == key || (key != null && key.equals(k))))  
            e = p;  
		//如果是树节点，调用putTreeVal
        else if (p instanceof TreeNode)  
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);  
        //如果是链表
        else {  
            for (int binCount = 0; ; ++binCount) {  
                if ((e = p.next) == null) {  
	                //尾插放入新节点
                    p.next = newNode(hash, key, value, null);  
                    //如果链表长度超过8，执行treeifyBin方法
                    //这个方法会根据 HashMap 数组来决定是否转换为红黑树
                    //只有当数组容量大于等于64的时候才会转为红黑树，小于64进行扩容
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st  
                        treeifyBin(tab, hash);  
                    break;  
                }  
                //如果链表中存在key，直接跳出循环
                if (e.hash == hash &&  
                    ((k = e.key) == key || (key != null && key.equals(k))))  
                    break;  
				//用于遍历桶中的链表，与前面的e = p.next组合，可以遍历链表
                p = e;  
            }  
        }  
        //表示map中已经存在key
        //onlyIfAbsent传入为false，将新值覆盖旧值，然后返回旧值
        if (e != null) { // existing mapping for key  
            V oldValue = e.value;  
            if (!onlyIfAbsent || oldValue == null)  
                e.value = value;  
            afterNodeAccess(e);  
            return oldValue;  
        }  
    }  
    ++modCount; 
    //超过阈值再扩容 
    if (++size > threshold)  
        resize();  
    afterNodeInsertion(evict);  
    return null;  
}
```
##### 扩容机制

```java
//扩容方法
final Node<K,V>[] resize() {  
    Node<K,V>[] oldTab = table;  
    int oldCap = (oldTab == null) ? 0 : oldTab.length;  
    int oldThr = threshold;  
    int newCap, newThr = 0; 
    if (oldCap > 0) {  
	    //判断如果原数组的容量大于最大值，那么将阈值设置为`Integer.MAX_VALUE`，不进行扩容，直接返回原数组。
        if (oldCap >= MAXIMUM_CAPACITY) {  
            threshold = Integer.MAX_VALUE;  
            return oldTab;  
        }  
        //如果原数组扩大2倍后在最大容量和默认初始容量之间，设置新数组的阈值也扩大2倍。
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&  
                 oldCap >= DEFAULT_INITIAL_CAPACITY)  
            newThr = oldThr << 1; // double threshold  
    }  
    else if (oldThr > 0) // initial capacity was placed in threshold  
        //使用初始容量和负载因子构造函数初始化时会出现初始容量为0，但阈值大于0的情况
        //创建对象时初始化容量大小放在threshold中，此时只需要将其作为新的数组容量
        newCap = oldThr;  
    else {               // zero initial threshold signifies using defaults  
        //无参构造初始化时容量和阈值都为0，在这里计算容量和阈值
        newCap = DEFAULT_INITIAL_CAPACITY;  
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);  
    }  
    //计算新的阈值
    //创建时指定了初始化容量或者负载因子，在这里进行阈值初始化
    //或者扩容前的旧容量小于16，在这里计算新的resize上限
    if (newThr == 0) {  
        float ft = (float)newCap * loadFactor;  
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?  
                  (int)ft : Integer.MAX_VALUE);  
    }  
    threshold = newThr;  
    //创建一个扩容后的数组
    @SuppressWarnings({"rawtypes","unchecked"})  
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];  
    table = newTab;  
    if (oldTab != null) {  
        for (int j = 0; j < oldCap; ++j) {  //遍历旧数组
            Node<K,V> e;  
            if ((e = oldTab[j]) != null) {  //e为数组的节点元素
                oldTab[j] = null;  //将旧数组中该位置的元素置为 null，以便垃圾回收
                if (e.next == null)  //如果节点e没有链接其他节点，直接放入新数组
                    newTab[e.hash & (newCap - 1)] = e;  
                else if (e instanceof TreeNode)  
                    //将红黑树拆分成2棵子树，如果子树节点数小于等于 UNTREEIFY_THRESHOLD（默认为 6），则将子树转换为链表。 
                    //如果子树节点数大于 UNTREEIFY_THRESHOLD，则保持子树的树结构。
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);  
                else { // preserve order  如果节点e是链表
                    Node<K,V> loHead = null, loTail = null;  //低位链表头尾节点
                    Node<K,V> hiHead = null, hiTail = null;  //高位链表头尾节点
                    Node<K,V> next;  
                    do {  
                        next = e.next;  
                        //使用哈希&原数组容量来判断移动到扩容后的新数组时位置是否改变
                        //如果&运算结果为0则表示hash小于原数组容量，位置不会改变
                        //如果&运算结果大于0则表示hash大于原数组容量，位置变为原数组位置+原数组容量
                        //将不改变位置的节点尾插入低位链表
                        if ((e.hash & oldCap) == 0) {  
                            if (loTail == null)  
                                loHead = e;  
                            else  
                                loTail.next = e;  
                            loTail = e;  
                        }  
                        //将改变位置的节点尾插入高位链表
                        else {  
                            if (hiTail == null)  
                                hiHead = e;  
                            else  
                                hiTail.next = e;  
                            hiTail = e;  
                        }  
                    } while ((e = next) != null);  
                    if (loTail != null) {  
                        loTail.next = null;  //将低位链表的尾结点指向null，以便垃圾回收
                        newTab[j] = loHead;  //低位链表放入新数组中，位置不变
                    }  
                    if (hiTail != null) {  
                        hiTail.next = null;  //尾节点置空，以便垃圾回收
                        newTab[j + oldCap] = hiHead;  //高位链表放入新数组，位置改变
                    }  
                }  
            }  
        }  
    }  
    return newTab;  
}
```

**总结**
1. 根据原数组容量计算新数组容量和阈值（如果原数组在默认容量16和最大容量之间，新数组容量扩大2倍，新数组阈值也扩大2倍）
2. 遍历旧数组：
	1. 如果旧数组节点无next节点，直接根据新数组容量计算下标，放入新数组
	2. 如果节点为树节点，分成两个子树，如果子树节点数小于等于6，则将子树转为链表，如果大于6，则保持子树结构
	3. 如果节点为链表，根据（hash & oldCap）按照位置是否改变分成低位链表和高位链表，位置不变的节点尾插放入低位链表，位置改变的节点尾插放入高位链表，最后将低位链表放入新数组原下标位置，高位链表放入原下标+原容量的位置


### LinkedHashMap

- 有序，使用双向链表记录元素顺序
- 底层使用哈希表+链表实现

##### 源码分析

LinkedHashMap继承了HashMap，增加了双向链表用来记录元素的顺序，增加了有序性

```java
static class Entry<K,V> extends HashMap.Node<K,V> {
	//before和after就是前驱节点和后继节点
    Entry<K,V> before, after;
    Entry(int hash, K key, V value, Node<K,V> next) {
        super(hash, key, value, next);
    }
}
```

链表的头节点表示最早插入或访问的元素，尾部节点表示最近插入或访问的元素，换句话说双向链表是尾插，这和hashMap的链表插入方法一致。双向链表默认记录的是插入顺序，还支持记录访问顺序。访问顺序与插入顺序的不同在于记录了调用get方法和remove方法的顺序，可以通过构造函数指定开启访问顺序记录
```java
public LinkedHashMap(int initialCapacity,  
                     float loadFactor,  
                     boolean accessOrder) {  
    super(initialCapacity, loadFactor);  
    this.accessOrder = accessOrder;  //访问顺序
}
```

### TreeMap

- 按照键的顺序自动排序，可以自定义排序规则
- 底层使用红黑树实现