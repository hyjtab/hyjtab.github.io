# 类加载

## 类文件结构

javap 反编译
```
    ClassFile {
        u4     magic; //Class 文件的标志
        u2     minor_version;//Class 的小版本号
        u2     major_version;//Class 的大版本号
        u2     constant_pool_count;//常量池的数量cp_info constant_pool[constant_pool_count-1];//常量池
        u2      access_flags;//Class 的访问标记
        u2      this_class;//当前类
        u2      super_class;//父类
        u2      interfaces_count;//接口数量
        u2     interfaces[interfaces_count];//一个类可以实现多个接口
        u2     fields_count;//字段数量
        field_info  fields[fields_count];//一个类可以有多个字段
        u2      methods_count;//方法数量
        method_info methods[methods_count];//一个类可以有个多个方法
        u2     attributes_count;//此类的属性表中的属性数
        attribute_info attributes[attributes_count];//属性表集合
    }
```
## 方法调用
```
    invokevirtual 是最耗时的
    <cinit> ，此方法用于初始化类变量
    <init>，此方法用于初始化实例成员变量
    创建一个类实例的三条字节码指令：
    new
    dup
    invokespecial #3
```    
    调用静态方法时，使用类直接调用更加迅速，使用实例调用会产生两条不必要的虚拟机指令

## 多态

（实验时禁用指针压缩）

使用HSDB工具查看
```
    java -cp .lib/sa-jdi.jar sun.jvm.hotspot.HSDB
    (在安装目录执行)
```
虚方法表在链接时确立，记录了此类所有方法的实际入口地址

当执行invokevirtual时，先通过栈帧的引用找到对象，分析对象头，找到对象实际Class,随后找到Class的vtable，查找虚方法表得到方法具体地址，随后执行方法的字节码。

## 异常

异常表：[from,to) target(跳转位置)type(异常类型)

第一句将是一个astore指令，将异常对象存入局部变量表上。

finally会在catch监测范围外的每一个try或catch块后复制一段finally块内内容，return语句之前，并且会自动生成若干个异常表项，对应于每个try或catch中的内容，target位于最后，type为any。any异常对象与Exception对象不共享槽位。

自动生成的athrow字节码指令会扔出any对象。而在finally里面有return的话，会吞掉异常，仿佛没有发生任何异常一样。

finally中的返回不会进行暂存操作，而try和catch中的返回会进行暂存操作。

synchronized块保证加锁解锁正常，也同样采用了异常表和anyh对象，保证monitorexit能够在任何情况下都可以解锁成功。

## 泛型擦除

实际上code中的泛型信息在编译为字节码之后就丢失了，对于此部分可以说，泛型仅针对编译阶段。而局部变量类型表中，泛型信息不会被擦除，但局部变量的反射 无法通过反射得到。

## 语法糖

switch string会变成hashcode，并用equals进行比较，确认情况后，再增加一个switch，根据情况结果实现功能，而枚举类会生成一个合成类，实现是一个数组，并附上对应的整数值，，foreach循环会自动调用iterator结构，压制异常。

## 类加载阶段

#### 加载

内部采用C++的instanceKlass描述java类
```
    _java_mirror 是java的类镜像，把klass暴露给java使用
    _supper父类
    _fileds成员变量
    _methods方法
    _constants常量池
    _class_loader类加载器
    _vtable虚方法表
    _itable接口方法表
```
加载和链接可能是交替运行的。

#### 链接

1.验证：检查字节码格式

2.准备：为static变量准备

static变量在JDK7之前存储与instanceKlass末尾，从JDK7开始，存储于_java_mirror末尾

static变量分配空间和赋值是两个步骤，分配空间在准备阶段完成，复制在初始化阶段完成，final变量非引用类型在准备阶段即可完成赋值，其余类型变量均在初始化阶段完成赋值。

#### 解析

将常量池中的符号引用（无意义）解析为直接引用（实际地址）

#### 初始化
```
    <cinit>()v方法
```
类初始化是懒惰的，main方法所在的类，首次访问类的静态变量或静态方法时，子类初始化时，子类访问父类的静态变量是，Class.forName以及new时会初始化

然而访问类的static final不会触发初始化，类对象class不会触发初始化，创建该类的数组不会初始化，类加载器的loadClass方法不会初始化，Class .forName的参数2为false时不会初始化。

## 类加载器

启动类加载器jre/lib，扩展类加载器jre/lib/ext，应用程序加载器，用户自定义加载器

## 双亲委派模式

调用类加载器的loadClass方法时，查找类的规则，从应用开始逐级向上，到最顶时开始加载，最后回到最底层

## 线程上下文类加载器

jdbc的Driver接口类在启动路径下，然而其具体实现类在用户路径下，而启动启动类加载器加载类时，规定要求必须能够完成其对应实现类的加载，因此在他的启动类加载器中直接调用了应用程序加载器加载对应实现类完成类加载，打破了双亲委派。

**SPI**

约定如下，在jar包的META-INF/services包下，以接口全限定名为文件，文件内容是类名称。在实现了SPI机制的接口中，可以通过ServiceLoader对所有包进行扫描，获取所有对应的实现类，逐个加载。