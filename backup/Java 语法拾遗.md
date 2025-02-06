# String的不变性
相关题目：
```java
String a = "hello, ";
String b = "world!";
String c = "hello, world!";
if(a + b == c) System.out.println("yes");//无输出
if("hello, " + "world!" == c) System.out.println("yes");//有输出
String a1 = new String("hello, ");
if (a1 == a ) System.out.println("yes");//无输出
StringBuilder sb1 = new StringBulider("nihao, ");
String s1 = sb1.toString();
StringBuilder sb2 = sb1;
sb1.append("Java!");
String s2 = sb1.toString();
if(s1 == s2) System.out.println("yes");//无输出
if(sb1 == sb2) System.out.println("yes");//有输出

String str1 = new StringBuilder("计算机").append("软件").toString();
System.out.println(str1.intern() == str1);//true
String str2 = new StringBuilder("ja").append("va").toString();
System.out.println(str2.intern() == str2);//false
```
## 原因
String本身是一个final修饰的类，同时其内容保存在value变量中，其属性为final修饰的私有成员变量。

## 不可变的好处：
**1.便于实现字符串池**

在Java中，由于会大量的使用String常量，如果每一次声明一个String都创建一个String对象，那将会造成极大的空间资源的浪费。Java提出了String pool的概念，在堆中开辟一块存储空间String pool，当初始化一个String变量时，如果该字符串已经存在了，就不会去创建一个新的字符串变量，而是会返回已经存在了的字符串的引用。

**2.使多线程安全**

由于字符串对象的内容无法被修改，这意味着多个线程可以安全地访问相同的字符串实例，而无需额外的同步机制来保护数据的一致性。这种特性带来以下优势：

首先，不可变字符串消除了竞态条件（race condition）的可能性。多线程环境下，如果多个线程同时对可变对象进行读写操作，可能会导致数据不一致或意外的结果。但是，由于字符串不可变，多个线程可以自由地读取相同的字符串实例，不会出现线程之间的数据竞争问题。

其次，不可变性使得字符串在多线程环境下可以安全地共享。在Java中，字符串常量池允许相同的字符串常量在内存中只存在一份实例。这意味着多个线程可以引用并共享相同的字符串对象，而无需担心其中一个线程会修改它，从而保证了线程安全性。

**3.加快字符串处理速度**

不可变性还为字符串的缓存和优化提供了机会。由于字符串的值在创建后不会改变，可以在创建时计算并缓存其哈希码、长度等属性，以提高性能和效率。

由于String是不可变的，保证了hashCode的唯一性，于是在创建对象时其hashCode就可以放心的缓存了，不需要重新计算。这也就是一般将String作为Map的Key的原因，处理速度要快过其它的键对象。所以HashMap中的键往往都使用String。

# synchronized方法块
synchronized方法块的优点在于，能够更加精确的控制锁的范围，同时支持非this锁的特性，也使得不同方法的异步执行成为可能，提升了程序的并行性。

synchronized方法的默认同步监视器为this，而synchronized静态方法为当前类的class对象。
# 抽象类
抽象类不一定要有抽象函数，但有抽象函数的类一定是抽象类。抽象类的非抽象子类必须实现抽象类的所有抽象方法。抽象类不能拥有自己的实例。
# 接口
接口中定义的所有变量或者方法，都会自动添加上 public 关键字，接口中定义的变量会在编译的时候自动加上 public static final 修饰符，没有使用 private、default 或者 static 关键字修饰的方法是隐式抽象的，静态方法无法由（实现了该接口的）类的对象调用，它只能通过接口名来调用。
# 枚举类
```java
class Season{
    private final String SEASONNAME;//季节的名称
    private final String SEASONDESC;//季节的描述
    private Season(String seasonName,String seasonDesc){
        this.SEASONNAME = seasonName;
        this.SEASONDESC = seasonDesc;
    }
    public static final Season SPRING = new Season("春天", "春暖花开");
    public static final Season SUMMER = new Season("夏天", "夏日炎炎");
    public static final Season AUTUMN = new Season("秋天", "秋高气爽");
    public static final Season WINTER = new Season("冬天", "白雪皑皑");

    @Override
    public String toString() {
        return "Season{" +
                "SEASONNAME='" + SEASONNAME + '\'' +
                ", SEASONDESC='" + SEASONDESC + '\'' +
                '}';
    }
}
class SeasonTest{
    public static void main(String[] args) {
        System.out.println(Season.AUTUMN);
    }
}
```
枚举类基本成员变量为private final，枚举类暴露的变量public static final,更加现代化的写法如下所示：
```java
public enum Week {
    MONDAY("星期一"),
    TUESDAY("星期二"),
    WEDNESDAY("星期三"),
    THURSDAY("星期四"),
    FRIDAY("星期五"),
    SATURDAY("星期六"),
    SUNDAY("星期日");

    private final String description;

    private Week(String description){
        this.description = description;
    }

    @Override
    public String toString() {
        return super.toString() +":"+ description;
    }
}
```
用枚举类实现单例模式：
```java
public enum Singleton {
     INSTANCE;
     public void businessMethod() {
          System.out.println("我是一个单例！");
     }
}
```
# 注解相关
有元注解和自定义注解
## 注解的继承
类级别 (Type): 注解 仅 在 类 Class 上且注解上含有 元注解 @Inherited 时, 才会被继承;（在 jdk 8 中, 接口Interface 无法继承任何Type类型注解)

属性和方法级别 (Property, Method): 注解 无论何时都会 被子类或子接口继承, 除非子类或子接口重写。

## 文件输入输出流的一般用法
```java
while ((len = fis.read(buffer)) != -1) {//
                fos.write(buffer, 0, len);
       }
```
## 序列化
自定义异常类需要声明serializeUID这个属性，并且其修饰符为stastic final long，作为唯一标识。

serialize接口作为表示接口，也需要这么做。

## hashmap
hashmap长度永远为2的幂次，方便计算index，hashmap判断一致性，首先比对hash2，再比对equals。
### hashmap扩容机制：
①.判断键值对数组table[i]是否为空或为null，否则执行resize()进行扩容；
②.根据键值key计算hash值得到插入的数组索引i，如果table[i]==null，直接新建节点添加，转向⑥，如果table[i]不为空，转向③；
③.判断table[i]的首个元素是否和key一样，如果相同直接覆盖value，否则转向④，这里的相同指的是hashCode以及equals；
④.判断table[i] 是否为treeNode，即table[i] 是否是红黑树，如果是红黑树，则直接在树中插入键值
对，否则转向⑤；
⑤.遍历table[i]，并记录遍历长度，如果遍历过程中发现key值相同的，则直接覆盖value，没有相同的key则在链表尾部插入结点，插入后判断该链表长度是否大等于8，大等于则考虑树化，如果数组的元素个数小于64，则只是将数组resize，大等于才树化该链表；
⑥.插入成功后，判断数组中的键值对数量size是否超过了阈值threshold，如果超过，进行扩容。
### 线程不安全
首先线程A需要阻塞在原hashmap扩容后移动的首个节点上，随后，线程B完成扩容操作，A恢复运行，继续尝试移动节点，此时他将把他认为的第一个节点，实际上的倒数第一个节点的next设为null，随后进行第二次移动，移动现在的倒数第二个节点，他认为的第二个节点，此时倒数第二个节点的next已经设为前面移动过的倒数第一个节点，随后进行移动，将他以为的第二个节点的next设为他以为的第一个节点，但实际上这步操作没有改变任何东西，其此步保存的原始next早就是他以为的第一个节点了，因此在进行一步循环，环就已经成了。

还有一种自环，数据丢失的情况，其常常发生于扩容后重新hash确认的index不同的时候，且线程A阻塞在该节点时，该节点发生自环，其后数据都丢失。(参考链接)[https://juejin.cn/post/6844904149616705543]

## 自增操作
```java
x=x++
```
x先进行iload，再进行llinc，再赋值，自身不会改变。

## final关键字
final关键字修饰类时，不能继承，修饰成员变量时，变量值不能改变

**tips:** 局部内部类与匿名内部类只能使用final关键字的外部参数。这么做一方面是为了避免生命周期不同，另一方面是为了避免内外部数据不一致。

若使用的外部变量不能在编译期间确定，jvm会默认将局部内部类与匿名内部类的默认构造方法上再增加该参数，通过构造器传参的方式来对拷贝进行初始化赋值。

此外，除静态内部类外，其他形式的内部类都依赖于外部类来进行访问，如果需要继承一个外部类，其不能拥有无参的构造函数，必须至少有一个外部类变量作为参数和访问指针，详情请查看[此篇文章](https://www.cnblogs.com/dolphin0520/p/3811445.html)

## 重写与遮蔽
```java
public class Parents {
    private static void _show(){
        System.out.println("父类");
    }
    public static void show(){
        _show();
    }
}

public class Son extends Parents{
    private static void _show(){
        System.out.println("子类");
    }
    public static void show(){
        _show();
    }

    public static void main(String[] args) {
        Parents parents = new Son();
        parents.show();
    }
}
```
有`static`和没有的结果是相反的，如果使用了`static`方法，则会调用`invokestatic`字节码指令，在编译时确定函数入口地址，编译时认为`parents`变量为`Parents`类型，而非`static`方法则会调用`invokevirtual`字节码指令，在运行时确定函数入口地址，运行时认为`parents`变量为`Son`类型，调用子类方法。
```java
public class Parents {
    public void _show(){
        System.out.println("父类");
    }
    public void show(Parents parents){
        parents._show();
    }
}
public class Son extends Parents{
    public void _show(){
        System.out.println("子类");
    }

    public static void main(String[] args) {
        Parents parents = new Son();
        parents.show(parents);
    }
}
```
`_show`方法是否为私有，也会影响输出结果，这里就用到了多态的特性。如果父类方法被子类重写，则可以通过子类对象调用，一旦父类`_show`方法声明为私有，`parents._show();`在编译阶段就会被编译为`invokespecial`从而无法进行多态查找。
```
 * 1、修饰符： >= 父类（非私有方法的情况下编译器报错）
 * 2、返回值： <= 父类
 * 3、方法名： 必须一致
 * 4、形式参数：必须一致 ， 传参时 <= 父类
 * 5、抛出异常：<= 父类
```

## Java动态代理
动态代理让我们能够不编写实现类就能够创建接口实例。其本质就是让jvm在运行期间动态创建class字节码并加载的过程。

其实现方式可以分为以下几步：
定义一个InvocationHandler实例，重写invoke方法

通过Proxy.newProxyInstance创建接口实例，需要三个参数，接口类的ClassLoader，接口数组，以及InvocationHandler实例。

将返回的Obj强转为接口。
```java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

public class Main {
    public static void main(String[] args) {
        InvocationHandler handler = new InvocationHandler() {
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                System.out.println(method);
                if (method.getName().equals("morning")) {
                    System.out.println("Good morning, " + args[0]);
                }
                return null;
            }
        };
        Hello hello = (Hello) Proxy.newProxyInstance(
            Hello.class.getClassLoader(), // 传入ClassLoader
            new Class[] { Hello.class }, // 传入要实现的接口
            handler); // 传入处理调用方法的InvocationHandler
        hello.morning("Bob");
    }
}

interface Hello {
    void morning(String name);
}
```

从 UserServiceProxy 的代码中我们可以发现：

UserServiceProxy 继承了 Proxy 类，并且实现了被代理的所有接口，以及equals、hashCode、toString等方法

由于 UserServiceProxy 继承了 Proxy 类，所以每个代理类都会关联一个 InvocationHandler 方法调用处理器
类和所有方法都被 public final 修饰，所以代理类只可被使用，不可以再被继承

每个方法都有一个 Method 对象来描述，Method 对象在static静态代码块中创建，以 m + 数字 的格式命名

调用方法的时候通过 super.h.invoke(this, m1, (Object[])null); 调用，其中的 super.h.invoke 实际上是在创建代理的时候传递给 Proxy.newProxyInstance 的 LogHandler 对象，它继承 InvocationHandler 类，负责实际的调用处理逻辑