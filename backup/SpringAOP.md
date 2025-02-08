# AOP实现

使用动态代理技术没在运行期间，对目标对象的方法进行增强，代理对象同名方法内可以执行原有逻辑的同时嵌入执行其他增强逻辑或其它对象的方法。

## 术语

目标对象 被增强的方法所在的独享

代理对象 对目标对象进行增强后的对象

连接点 目标对象中可以被曾庆的方法

切入点 目标对象中实际被增强的方法

通知\增强 增强部分的代码逻辑

切面 增强和切入点的组合

织入 将同志和切入点组合动态组合的过程

# 步骤

1. 导入AOP相关坐标

2. 准备目标类、增强类并配置给Spring管理

3. 配置切点表达式（哪些方法被增强）

4. 配置织入（哪些方法为增强方法，增强执行顺序是什么）
```xml
    <aop:config>
    <!--切点表达式的配置-->
        <aop:pointcut id="myPointcut" expression="excution([访问修饰符] 返回类型 全限定名.方法(参数))">
    <!--
    返回值类型、某一级报名、类名、方法名可以用*表示任意
    包名与类名之间使用单点表示该包下的类，使用双电..表示该包及其子包下的类
    参数列表可以使用两个点，表示任意参数（包括无参）
    -->
    <!--切面配置-->
        <aop:aspect ref="通知类">
            <aop:before method="方法名" pointcut-ref="myPointcut"><!--或者pointcut="切点表达式"-->
        </aop:respect>
    </aop:config>
    //任意报下任意类的任意方法
    excution(* *..*.*(..))
```
一共有五种通知
```
    <aop:before/> 目标之前
    <aop:after-returning/>目标之后，方法如果异常就不执行
    <aop:around/>目标方法执行前后执行，方法异常时，环绕后方法不再执行
    对于环绕方法其返回类型需为Object类，并且需要有一个参数，类型为ProceedingJoinPoint，该参数需要调用proceed方法执行
    <aop:after-throwing/>目标方法抛出异常时执行，异常通知中可以捕获异常，但需要在配置时指出异常对象的变量名，即<aop:aftrt-throwing throwing="参数名">
    <aop:after>不管目标方法是否有异常，最终都会执行
```
任何通知都可以使用JointPoint参数，可以获取当前目标对象和切点表达式

还可以通过实现Advice接口的类对应的子接口，在xml中使用aop:advidsor配置，可以实现一行配置所有通知类中的所有调用。

advisor只能配置一种固定的通知和切点表达式

## 原理

aop解析器向容器当中注入了一个Bean后处理器，通过此后处理器的after方法为对应的目标对象生成对应的动态代理

### AOP底层两种代理生成方式

**JdkDynamicProxy:** 使用条件，目标类有接口，基于接口动态生成实现类的代理对象，只要目标类有接口默认就采取此种方式

**CglibAopProxy：** 目标类无接口且不能使用final修饰，是基于被代理对象动态生成子对象为代理对象，目标类无接口时，默认使用该方式，可以在config标签中使用 proxy-target-class="true"强制调用。该方法的原理是生成目标类的子类作为代理对象。

![Image](https://github.com/user-attachments/assets/c1b5971b-3eb0-4061-ad56-dcaf9761f4ae)

## 使用注解配置AOP

需要开启AOP自动代理`<aop:aspectj-autoproxy/>`

在增强类上使用@Componet与@Aspect两个注解，并在对应的增强函数上配置对应的切点表达式

切点表达式抽取，可以通过定义一个空函数，在其上使用@Pointcut注解，将切点表达式写入value中，引入时，只需要写空函数所对应类名.空函数，即可实现切点表达式的引用

如果使用全注解开发，则需要在配置类上加一个@EnableAspectjAutoProxy即可。

### 原理

**用xml配置注解：** aop解析器向容器当中注入了一个Bean后处理器，通过此后处理器的after方法为对应的目标对象生成对应的动态代理，和纯xml的入口不同

**纯注解：** 使用@Import注解，加载一个实现了注册接口的类，注入了一个Bean后处理器，通过此后处理器的after方法为对应的目标对象生成对应的动态代理

![Image](https://github.com/user-attachments/assets/c14b156e-7c61-41c8-9f48-ed99dbb6ff36)

# 基于AOP的声明式事务控制

Spring在众多持久层框架技术的事务管理基础上，提供了同意控制事务的接口

## 编程式事务控制

编程控制，有些复杂

## 声明式事务控制

配置控制，而且解耦

平台事务管理器 PlatformTransactionManager 一个接口标准，实现类都具备事务提交、回滚和获得事务对象的功能（功能）

事务定义 TransactionDefinion 封装事物的隔离级别、传播行为、过期时间等属性信息。（定义）

事务状态 TransactionStatus 存储当前事务的状态信息，如果十五是否提交、是否会滚、是否有回滚点等。（信息）

功能+定义=信息
```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xmlns:context="http://www.springframework.org/schema/context"
           xmlns:aop="http://www.springframework.org/schema/aop"
           xmlns:tx="http://www.springframework.org/schema/tx"
           xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd
           http://www.springframework.org/schema/context
           http://www.springframework.org/schema/context/spring-context.xsd
           http://www.springframework.org/schema/aop
           http://www.springframework.org/schema/aop/spring-aop.xsd
           http://www.springframework.org/schema/tx
           http://www.springframework.org/schema/tx/spring-tx.xsd">
    
        <context:component-scan base-package="org.example"></context:component-scan>
        <context:property-placeholder location="classpath:jdbc.properties"></context:property-placeholder>
    
        <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
            <property name="driverClassName" value="${jdbc.driver}"></property>
            <property name="url" value="${jdbc.url}"></property>
            <property name="username" value="${jdbc.username}"></property>
            <property name="password" value="${jdbc.password}"></property>
        </bean>
    
        <bean class="org.mybatis.spring.SqlSessionFactoryBean">
            <property name="dataSource" ref="dataSource"></property>
        </bean>
    
        <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
            <property name="basePackage" value="org.example.mapper"></property>
        </bean>
    <!--配置平台事务管理器，不同的持久层框架使用不同的平台事务管理器-->
        <bean id = "transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
            <property name="dataSource" ref="dataSource"></property>
        </bean>
    <!--配置spring提供好的增强-->
        <tx:advice id="txAdvice" transaction-manager="transactionManager">
            <tx:attributes>
                <!--
                进一步配置不同的方法事务名称
                name：方法名称，*代表通配符， add*代表所有以add开头的方法
                isolation：事务隔离级别，解决事务并发问题（脏读，不可重复读，幻读）
                timeout：超时时间，默认-1，无超时时间，单位为秒
                read-only：连接是否只读，查询操作可设为true
                propagation：事务的传播行为，解决业务调用业务发生的问题（两个操作都有事务应该使用谁的事务）（事务嵌套问题）
                -->
                <tx:method name="*"/>
            </tx:attributes>
        </tx:advice>
        <aop:config>
            <!--此处配置总体AOP增强范围-->
            <aop:pointcut id="txPointcut" expression="execution(* org.example.service.impl.*.*(..))"/>
            <!--引入一个transactionInterceptor，实现了MethodInterceptor接口，相当于一个环绕通知-->
            <aop:advisor advice-ref="txAdvice" pointcut-ref="txPointcut"></aop:advisor>
        </aop:config>
    </beans>
```
本质上是引入了一个transactionInterceptor，实现了MethodInterceptor接口，相当于一个环绕通知。

使用注解配置，就近原则，配置函数上，只有该函数具有事务，配置到类上，类中所有方法共用一套配置。注解为`@Transaction` 需要使用`<tx:anotation-driven transcation-manager="transactionManager"/>` 标签配置，相当于`@EnableTransactionManagement` 标签。