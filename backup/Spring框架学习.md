## 三个思想

### IoC

控制反转，Bean的创建权力反转给第三方

### DI

依赖注入，Bean之间的关系由第三方负责设置
```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!--xmlns意思是命名空间，默认是spring空间，xsi是核心命名空间，xsi:schemaLocation配置的是命名空间以及对应解析方法-->
        <bean id="userService" class="org.example.service.impl.UserServiceImpl">
            <property name="userDAO" ref="userDao"></property><!--name指定了注入方法，ref指定了注入方法参数-->
        </bean>
        <bean id="userDao" class="org.example.dao.impl.UserDaoImpl"></bean>
        <bean id="simpledatefromatter" class="spdf的全限定类名">
            <constructor-arg name="形参变量名" value="格式字符串"></constructor-arg>
        </bean>
        <bean id="Date" class="Date的全限定类名" factory-bean="simpledatefromatter" factory-method="parse">
            <constructor-arg name="形参变量名" value="格式字符串"></constructor-arg>
        </bean>
    </beans>
```
### AOP

面向切面编程，利用proxy对功能进行横向抽取。

## BeanFactory与ApplicationContext

BeanFactory是Spring的早期接口，Bean工厂；ApplicationContext是高级接口，是Spring容器

ApplicationContext在BeanFactory上扩展了功能 ，并封装了BeanFactory的底层方法

BeanFactory的主要功能是创建Bean，ApplicatonContext不仅继承了BeanFactory，而且内部还维护者BeanFactory的引用，ApplicationContext与BeanFactory既有继承又有融合的关系。

BeanFactory在getbean时创建Bean，ApplicationContext则在容器创建时就对bean进行初始化。

## Spring应用配置

![Image](https://github.com/user-attachments/assets/0d98d4a2-87ab-46a8-a25f-15daff47f8f4)

name

id没有name时默认是类全限定名，有name时默认是第一个别名，用于创建指定的bean对象，alias标签可以单独起别名

### scope

singgleton默认，在容器创建时就对bean进行初始化，置于单例池中。prototype则会在getbean时创建Bean，也不会放在Spring容器单例池。

### destroy-method

必须显式销毁容器才能够执行，不显式销毁，虽然不执行destroy-method，但是bean对象也会销毁。

### init-method

在执行构造方法后调用，除此方法外，还可以实现InitializingBean接口中的afterPropertiesSet方法，来完成Bean的初始化操作，此种方式的调用在init-method之前，属性设置之后。

### autowire

自动在bean中装配，

1. byname方式匹配的是实现类set方法set之后的名字，其首字母小写对应的bean对象id。
  
2. byType方式匹配的是实现类set方法set之后的名字，其首字母小写对应的bean对象类型。
  

## Bean的实例化

有参构造使用constructor-arg标签。constructor-arg标签用于指明产生bean对象方法的参数。

### 静态工厂实例化

写一个工厂对象，对象包含一个静态方法，在配置时指定factory-method，即可在容器创建时创建工厂所实现的bean实例。

1. 可以在bean创建之前进行一些其它的业务逻辑操作
  
2. 可以引入不知名构造函数的对象的初始化。
  

### 实例工厂实例化

需要配置工厂对象，再配置新的bean，指定factory-bean和factory-method，以获得对性的bean对象。

### 实现FactoryBean规范延迟实例化

实现FactoryBean接口，在容器创建时会将工厂类放置于单例池中，在getbean时才调用getObject方法放置于factoryBeanObjectCache的缓存中，再次get则会从缓存中取得。

## 注入序列化数据

注入调用的是setter方法，如果需要注入一个序列数据则需要在property标签中写子标签`<list><value>xxx</value></list>`或者`<list><bean class>xxx</value></list>`或者`<list><ref>xxx</ref></list>`

如果是map，则`<list><entry key[-ref]="" value[-ref]=""></entry></list>`,ref是否需要取决于键值对是否为引用类型。

如果是properties，则是`<props><prop key="">xxx</prop></props>`

## bean实例化的基本流程

![Image](https://github.com/user-attachments/assets/b9fc4a54-27b0-47e7-b187-8643eb9fa4c4)

## Spring的后处理器

Bean工厂后处理器-BeanFactoryPostProcessor

这是一个接口规范，在beanDefinitionMap填充完毕之后调用，其参数是一个ConfigurableListablebeanFactory类型参数，可以通过该参数对指定的Bean进行修改。

该参数其实经过过泛型，本质是其子类对象，可以通过其子类对象的方法对Map进行修改。当然更加推荐的方式是实现后处理器的子接口BeanDefinitionRegistryPostProcessor

注意这些后处理器都需要在xml中进行配置。

Bean后处理器-BeanPostProcessor

创建Bean之后，缓存到sigletonObjects单例池之前调用，需要实现的是BeanPostProcesser接口，其内部有两个default方法。具体而言是在Bean的实例化之后，InitializingBean接口和init-method方法之前。

## Bean生命周期

从Bean实例化之后，到Bean成为一个完整对象放入单例池中的整个过程。可以大体分为三个阶段

### Bean的实例化阶段

创建对象，根据xml进行ifelse判断

### Bean的初始化阶段

进行属性填充，执行Aware接口方法，BeanPostProcessor，InitialzingBean接口方法，Init-method方法等

### Bean的完成阶段

存到单例池singletonObjects中

### 属性填充

在Beandefinition中会封装xml中配置的bean信息，包括属性填充信息propertyValues中。注入单项引用属性时，从容器中getBean获取后通过set方法反射设置进去，如果容器中没有，则先创建被注入对象的Bean实例后，再进行注入操作。

## 三级缓存

注入双向对象引用属性时，涉及了循环引用问题，利用三级缓存来解决。

在DefaultListableBeanFactory上的四级父类DefaultSingleton提供三个map
```
    sigletonObjects//最终成品的Map private final
    earlySingletonObjects//创建的半成品，早期单例池，已经被别人引用
    singletonFactories<String,ObjectFactory<T>>//单例Bean的工厂池，对象未被引用，bean还未创建完毕,三级缓存，存储的是对应的ObjectFactory对象，getObject时返回早期单例
```
当被别人引用时，对象从三级缓存演变到二级缓存中

## Aware接口

实现框架提供的Aware接口，让框架自动注入底层功能API对象，常用的有ServletContextAware、BeanFactoryAware、BeanNameAware、ApplicationContextAware这四种

![Image](https://github.com/user-attachments/assets/bad085dd-a756-42d2-ac35-a2935a3b067b)

从xml到beanDefinitionMap需要借助xmlReader

## 基于xml的Spring应用

Spring支持el表达式，需要使用如下标签配置获得对应的properties变量

    <context:property-placeholder location="classpath:"/>
    ${}就可以引用

### 不需要自定义命名空间

    导入MyBatis整合的Spring的相关坐标
    编写Mapper和Mapper.xml
    配置SqlSessionFactoryBean和MapperScannerConfigurer（前者向容器提供SqlSessionFactory，后者将Mapper对象存储到Spring容器中）（注意第一个需要配置数据源属性（数据库连接池），第二个需要配置一个）
    编写测试代码

原理：

SqlSessionFactoryBean类在执行InitializingBean接口方法时已经创造了SqlSessionFactory，在getObject时，直接返回提前创建的SqlSessionFactory对象

MapperScannerConfigurer类实现了BeanFactory的后处理器，其中ClassPathMapperScanner会修改自动注入状态，注入SqlSessionFactory对象，扫描所有接口，然后将该接口包装成一个FactoryBean，然后在getObject方法中调用Mybatis的动态代理，以此获得对象

### 需要自定义命名空间

第三方需要自定义命名空间，并且能够被Spring框架解析，需要在包的META_INF下创建一个spring.hadlers，其中包含命名空间以及对应的NameSpaceHandler解析器，每个解析器偶需要实现NamespaceHandler接口，其中有init和parse方法，先执行init方法注册多个标签的解析器，再执行parse方法进行解析。若有多个标签，每一个标签都要实现BeanDefinitionParser的接口，从而Spring就能够在调用parse方法解析时，调用对应的标签Parser解析器的解析方法，在这个最终的解析方法中可能会有两种操作，一个是直接向BeanDefinitionMap中注册bean对象，另一个是注册BeanPostProcessor从而对其他bean执行操作。

该文件夹下还要有一个spring.schemas文件，该文件需要配置网址类型的xsd实际映射到的jar包内的本地约束xsd文件路径。约束文件能够指定可配置标签类型。
```
    //parser方法
    BeanDefinition beanDefinition = new RootBeanDefinition();
    beanDfinition.setBeanClassName("class全限定名")
    parserContext.getRegistry().registerBeanDefinition("name",beanDefinition);
    return beanDefinitio;
```
## 基于注解的Spring配置
```
bean-->@Component(value="id")
```
同样需要在applicationContext.xml中配置
```
    <contex:component-scan base-package="扫描基本包"/>
```
value为空时，默认为类名首字母小写。
```
    @scope("sibngleton")
    @Lasy(true)
    @PostConstruct--->加在函数上
    @PreDestroy--->加在函数上
```
@Component的语义化衍生注解
```
    @Repository
    @Service
    @Controller
```
### 依赖注入注解
```
    @Value()          使用在字段或者方法上，诸如普通数据
    @Autowired        使用在字段或方法上，根据类型注入引用数据，如果有多个同一类型的bean，尝试根据名字与变量名进行二次匹配，用在方法上时，根据形参的类型和名字进行匹配，如果形参是Colection，则可以装配对应泛型类型所有的实例。在@Bean方法的形参上可以省略Autowired注解。
    @Qualifier()      使用在属性字段或者方法上，结合@Autowired，根据名称注入，在@Bean方法的形参上可以省略Autowired注解。
    @Resource(name="")使用在字段或者方法上，根据类型或名称进行注入
    方法都是指set方法
```
### 非自定义Bean的注解开发
```
    @Bean("beanname")
    public type foo(@Value("${something}")ele x){
        type t = xxx.getfoo(x);
        return t;
    }
    Bean注解的默认名字为方法名，其所在的类必须要加上一个Component注解。
```
### Bean配置类的开发
```
    @Configuration//标注当前类是一个配置类+@Component
    @ComponentScan("basepackage名字")替代组件扫描配置
    @PropertySource("配置文件路径")替代标签配置扫描
    @Import(类名.class)替代import标签
    Class Config(){}
    
    
    //然后在主方法中使用这句话加载核心配置类
    ApplicationContext ApplicationContext = new AnnotationConfigApplicationContext(Config.class);
```
### 其他标签
```
    @Primary()提升注册优先级
    @ProFile()标注当前Bean从属于哪个环境
    使用命令行参数或者System.setProperty("spring.profile.active","环境名")
```
### xml配置注解组件扫描原理

还未执行BeanFactoryProcessor时，就通过ComponentScanBeanDefinitionPaser，利用ClassPathBeanDefinitionScanner扫描器的doScan方法扫描并利用配置的conetxt自定义命名空间的解析器的注册函数注册到BeanDefinitionMap中了。

### 配置类注解组件扫描原理

首先注册多个Spring内部bean，再通过其中的bean对象ConfigurationClassPostProcessor，随后在BeanDefinitionRegistryPostProcesso中利用ClassPathBeanDefinitionScanner扫描器的doScan方法扫描各个bean，并，在BeanPostProcessor中进行注入操作

### 利用注解调用Mybatis

首先配置DataSource类，再配置SqlSessionFactoryBean，返回一个可以调用mybatis的动态代理的BeanFactory，最后在配置类上用注解@MapperScan("基本包")

原理上，注解方式比xml方式会多一步，在注册后处理器调用时注册MapperScannnerConfigurer，也就是xml方式的入口

### 补充：@Import可以导入三种类

普通的配置类

实现ImportSelector接口的类，接口对应的方法需要返回一个需要被注册到Spring容器中的**所有Bean的全限定名**的String数组。函数参数能够通过getAnnotationAttributes获取到使用import导入此接口实现类的母类上的其它的注解元信息的Map

实现ImportBeanDefinitionRegistrar接口的类，此函数用来直接注册BeanDefinition。