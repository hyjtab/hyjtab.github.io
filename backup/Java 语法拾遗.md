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
```
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

属性和方法级别 (Property, Method): 注解 无论何时都会 被子类或子接口继承, 除非子类或子接口重写.