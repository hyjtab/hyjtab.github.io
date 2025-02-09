mvc容器是Spring的子容器，子容器如果没有对象则在父容器中寻找，但不能反过来

## JavaWeb三大组件

![Image](https://github.com/user-attachments/assets/ab77d4a0-bd77-49c3-8fe6-1c275055acd4)

##Spring所做的

提供一个Listener，在其初始化方法中加载web配置中对应的配置文件，并能够正确返回ApplicationContext

## 传统web结构

![Image](https://github.com/user-attachments/assets/a6b7ed63-f17f-481b-a9c2-2bad50f84bd3)

问题有三：

1.每一个业务都对应一个servelet（拆分为Controller）

2.每一个Servelet都需要进行非常繁琐的操作（封装抽取功能）

3.Servlet获得Spring容器的组件只能通过客户端代码去获取，不能优雅的整合（直接注入）

最终思想：使用DispatcherServlet前端控制器完成共有行为，再加上所配置的Bean执行特有的行为，前端控制器可以完成：映射到特有业务Bean的能力，可以解析参数，封装实体类的共有功能，具备响应视图及响应其他数据的功能。

## SpringMVC简介

1.导入spring-mvc坐标

2.配置前端控制器

3.编写Controller配置映射路径，交给SpringMVC容器管理

### 原理概述

启动时就注册前端控制器，配置init参数，加载mvc文件，扫描并获取CotrollerBean，放到SpringMVC容器中，再根据信息进行任务分发。

 HandlerMapping:处理器映射器（匹配路径与bean）RequestMappingHandlerMapping

HandlerAdapter:处理器适配器（匹配bean与方法）RequestMappingHandlerAdapter

ViewResolver:视图解析器（解析视图）InternalResourceViewResolver

![Image](https://github.com/user-attachments/assets/59b898fa-9c75-498f-b4de-6c113129726f)

默认情况下，DispatcherServelet启动时就会调用initStratagies方法将默认配置DispatcherServlet.properties中多个映射器适配器和解析器注册到其自身List成员变量中，若Spring容器配置了HandlerMapping等，则不会加载默认的对应种类的多个映射器。

## 请求处理

@GetMapping,@PostMapping,@RequestMapping放在类上时，对整个类的所有方法起效。

@RequestParam("参数名") 形参这样可以实现类似自动注入的效果

当有多个同名参数时，形参如果是数组类型，Spring可以自动装入同名形参数组，但是如果是List或者map，则必须要加上注解@RequestParam来告诉spring不要创建Collection而是单纯装入。

还有一个参数required和defaultValue，前者要求参数不为空，后者设置默认值，注意当参数为基本类型时，无论如何设置均不能缺少。

如果想要获取所有请求体内容，则需要使用注解@RequestBody

自己在mvc配置文件中配置字符串转换器，Spring便可以自动转换请求体为转换器对象
```xml
    <bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMppingHandlerAdapter">
        <property name="mesasageConverters">
            <list>
                <bean class="实现了HttpessageConverter接口的工具类的全限定名">
                </bean>
            </list>
        </property>
    </bean>
```
非法异常一般===>无法解析

### Restful风格

（Representational State Transfer）

1. 用URI表示某个模块资源，资源名称为名词
  
2. 用情i去方式表示模块具体业务动作（GET表示查询，POST表示插入，PUT表示更新，DELETE表示删除（存在于URL地址中））
  
3. 用HTTP响应状态码表示结果，三部分状态码，状态信息，响应数据，code,message,data
  

如果需要实现http://localhost/user/100中获取100，则需要在请求处理注解的路径中用`{}` 包住想要解析的变量，并在对应的形参处使用@PathVariable("变量名")来确定一一对应关系。如果有多个`{}` 围住的，则可以多个@PathVariable("变量名")来读取。

## 文件上传

表单提交必须是POST

表单enctype属性必须是mutipart/form-data

文件上传表单必须要有name属性，且与服务端形参一致

形参类型必须为MultipartFile类型，要加注解@RequestBody

MVC默认关闭文件上传，必须引入文件上传解析器，并且名字是固定的，为“mutipartResolver”，并且引入Apache的commons-fileupload包

![Image](https://github.com/user-attachments/assets/19894606-a9b6-47e1-bff0-c2667c203673)

## 获得请求头信息

@RequestHeader("请求头名字")

@RequestHeader Map map===>获得所有请求头数据。

@CookieValue(value = "JESSIONID", default= "") String jessionid获得Cookie

获取request域对象

![Image](https://github.com/user-attachments/assets/c67896e7-4115-4efa-9b40-bfc23278afae)

## 请求静态资源

原本的Tomcat服务器，默认有一个DefaultServlet，负责处理其它Servlet无法访问的请求，其对应的urlParttern为/，即缺省拦截任何请求。使用了SpringMVC之后，其配置的urlParttern也为/，这就会覆盖掉原有的能够访问静态资源的DefaultServlet，从而丧失访问静态资源的能力。

三种解决方式

1. 修改default的servlet配置，使用更加精确的配置，而非/。
  
2. 在spring-mvc.xml中配置静态资源映射，匹配映射路径的请求。
  ```
    <mvc:resources mapping="/image/*" location="/img/"/>
```
3. 向容器内注册静态资源服务器`<mvc:defalut-servlet-handler/>`

第二、三种方式会自动提前注册一个SimpleUrlHandlerMapping，导致无法执行路径匹配。

### MVC注解驱动

可以自动配置默认HandlerMapping与HandlerAdapter以及字符串转换器。

![Image](https://github.com/user-attachments/assets/285e3153-f2fd-4627-a54c-b8ce7914387d)

### 响应

同步方式：请求资源转发（return 'forward: /index.jsp';），请求资源重定向（return 'redirect: /index.jsp';），响应模型数据return ModelAndView类型对象，直接回写数据给客户端(return string，并添加@ResponseBody注解)。

转发，客户请求一次，服务端响应一次，向资源请求一次，返回一次。重定向，客户请求两次，服务端响应两次，不同资源响应一次。

异步方式：在方法上添加注解@RequestBody，直接返回对象，SpringMVC会自动尝试将对象转为json。若一个类全部都返回json串，则可以将@Controller和@RequestBody合并，使用@RestController

### 拦截器

对Controller资源访问是进行拦截操作的技术，拦截后进行权限控制民工嗯那个增强都是可以的，类似于Filter，只不过Intercepter位于DispatcherServlet处理之后访问Controller之前，而Filter位于所有Servlet处理之前

实现HandlerInterceptor请求。有三个方法，preHandle，返回值决定是否能够访问到目标Bean。postHandle在目标执行后执行，afterCompletion，相当于final，总会执行，但是pre返回为false则不执行，可以处理异常对象。

配置拦截器

    <mvc:interceptors>
        <mvc:interceptor>
            <mvc:mapping path="/**"/>
            <bean class="拦截器"></bean>
        <mvc:interceptor>
    <mvc:interceptors>`

拦截器执行顺序取决于配置顺序，pre正向，post与after为逆序。

![Image](https://github.com/user-attachments/assets/44cd03df-8ba8-4314-acc1-91024dce1e98)

HandlerExcutionChain包含一个HandlerExcutionChain List和一个CotrollerBean，在其中会获得MapperHandler并按正序遍历pre，逆序遍历post和finally。

### SpringMVC配置文件

controller组件扫描
```
    #包扫描
    <context:component-scan base-package="controller所在基本包"/>
    #非自定义bean
    <mutipartResolver>
    #非bean
    <驱动注解>
    <拦截器配置>
    <静态资源处理器>
```
前两个解决方式同Spring

驱动注解与拦截器配置，在设置类上添加@EnableMVC注解，实现WebMvcConfigurer对应函数，开启默认配置，注册拦截器

![Image](https://github.com/user-attachments/assets/e1822da7-3d5f-440a-9bc4-2cce661520e3)

全注解开发，写一个实现ServletContainerInitializer接口的类，并在META-INF文件夹中创建service中的以ServletContainerInitializer接口全限定名为名的文件，其中内容为自己实现的类的全限定名。

自己的类需要配置@HandlerTypes({接口类名})，只扫描对应接口实现以及其子类，并可以利用ctx进行servelet的注册，本质上是java的SPI。

Spring实现了SpringServletContainer，自己实现一个其支持的扫描类的抽象类的子类，配置自己需要实现的方法即可。

![Image](https://github.com/user-attachments/assets/286617b2-31e9-4b2b-9632-3075773b30e9)

### 前端控制器原理

启动时进行初始化，做了两件事：

获得一个SpringMVC的ApplicationContext容器，注册SpringMVC的九大组件。

DispatcherServlet继承于HttpServletBean，在其init方法中，会将Spring容器注册为SpringMVC容器的父容器，即属性parent。

按注解配置时容器创建较早，其父类FrameworkServlet注册了一个Spring事件监听器，监听刷新操作发布的容器刷新完成这一事件，从而执行onFresh()方法，并执行initStrategies注册9大组件。

service方法被framework替换为同一方法，最后都指向DispatcherServlet的doDispatch方法。

最终执行intercertor的不是handlerAdapter而是在doDispatch流程中调用，ha只调用本身的方法。底层执行的是方法的反射。

### Spring异常处理机制

![Image](https://github.com/user-attachments/assets/3fde2a87-d9f8-45b6-8763-48c12ac87ddf)

简单异常处理器：simpleMappingExceptionResolver，在配置类中使用setDefaultErrorView方法注册兜底错误页面即可，也可以使用Properties经由setExceptionMappings传入异常处理器设置来配置。

自定义异常处理器：实现HandlerExceptionResolver接口，返回一个modelandview对象，如果想要返回Json，可以通过response.getWriter.write()来获取

使用注解标注异常处理器：类上使用@ControllerAdvice和函数上使用@ExceptionHandler(对应错误类.class)由默认情况下就会注册到SpringMVC中的异常解析器ExceptionalHandlerExceptionResolver解析的。这样就可以配置任意返回值的异常处理器了，注意，对于Json，仍然需要增加@ResponseBody，所以可以提升到类上。