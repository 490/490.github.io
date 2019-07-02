---
title: Spring基础知识
date: 2019-03-09 21:39:31
tags: Spring
---
# Spring IoC、AOP 的理解以及实现的原理

## Spring IoC

*   Spring IoC 实现原理：反射创建实例。
*   IoC 容器的加戴过程：`XML -> 读取 -> Resource -> 解析 -> BeanDefinition -> 注册 -> BeanFactory`

<!--more-->
IoC(Inversion of Control,控制翻转) 是Spring 中一个非常非常重要的概念，它不是什么技术，而是一种解耦的设计思想。它的主要目的是借助于“第三方”(Spring 中的 IOC 容器) 实现具有依赖关系的对象之间的解耦(IOC容器管理对象，你只管使用即可)，从而降低代码之间的耦合度。IOC 是一个原则，而不是一个模式，以下模式（但不限于）实现了IoC原则。

Spring IOC 容器就像是一个工厂一样，当我们需要创建一个对象的时候，只需要配置好配置文件/注解即可，完全不用考虑对象是如何被创建出来的。 IOC 容器负责创建对象，将对象连接在一起，配置这些对象，并从创建中处理这些对象的整个生命周期，直到它们被完全销毁。

在实际项目中一个 Service 类如果有几百甚至上千个类作为它的底层，我们需要实例化这个 Service，你可能要每次都要搞清这个 Service 所有底层类的构造函数，这可能会把人逼疯。如果利用 IOC 的话，你只需要配置好，然后在需要的地方引用就行了，这大大增加了项目的可维护性且降低了开发难度。关于Spring IOC 的理解，推荐看这一下知乎的一个回答：https://www.zhihu.com/question/23277575/answer/169698662 ，非常不错。

**控制翻转怎么理解呢?** 举个例子："对象a 依赖了对象 b，当对象 a 需要使用 对象 b的时候必须自己去创建。但是当系统引入了 IOC 容器后， 对象a 和对象 b 之前就失去了直接的联系。这个时候，当对象 a 需要使用 对象 b的时候， 我们可以指定 IOC 容器去创建一个对象b注入到对象 a 中"。 对象 a 获得依赖对象 b 的过程,由主动行为变为了被动行为，控制权翻转，这就是控制反转名字的由来。

**DI(Dependecy Inject,依赖注入)是实现控制反转的一种设计模式，依赖注入就是将实例变量传入到一个对象中去。**

## Spring AOP

*   Spring AOP 实现原理：动态代理
    *   JDK 的动态代理：如果目标对象的实现类实现了接口，Spring AOP 将会采用 JDK 动态代理来生成 AOP 代理类。
    *   CGLib 动态代理：如果目标对象的实现类没有实现接口，Spring AOP 将会采用 CGLIB 来生成 AOP 代理类。
    *   动态代理与 CGLib 实现的区别

# Spring Boot 和 Spring 的区别

*   Spring Boot 是基于 Spring 的一套快速开发整合包；
*   内嵌了如 Tomcat，Jetty 和 Undertow 这样的容器，也就是说可以直接跑起来，用不着再做部署工作了；
*   无需再像 Spring 那样搞一堆繁琐的 xml 文件的配置；

# ApplicationContext 和 BeanFactory 的区别

*   加载 Bean 的时机不同
    *   BeanFactroy 采用的是延迟加载形式来注入Bean 的，即只有在使用到某个 Bean 时（调用getBean()），才对该 Bean 进行加载实例化，这样，我们就不能发现一些存在的 Spring 的配置问题。
    *   ApplicationContext 是在容器启动时，一次性创建了所有的 Bean。这样，在容器启动时，我们就可以发现 Spring 中存在的配置错误。

# Spring Bean 的作用域：

*   singleton：在 Spring 的 IoC 容器中只存在一个对象实例，这个实例会被保存到缓存中，并且对该 bean 的所有后续请求和引用都将返回该缓存中的对象实例。
*   prototype：每次对该 bean 的请求都会创建一个新的实例。
*   request：每次 http 请求将会有各自的 bean 实例。
*   session：在一个 http session 中，一个 bean 定义对应一个 bean 实例。
*   globalSession：在一个全局的 http session 中，一个 bean 定义对应一个 bean 实例。


# Spring Bean 生命周期

![image](http://490.github.io/images/20190327_193609.png)

*   Spring 对 Bean 进行实例化。

    *   相当于程序中的 `new Xxx()`。
*   Spring 将值和 Bean 的引用注入进 Bean 对应的属性中。

*   如果 Bean 实现了 BeanNameAware 接口，Spring 将 Bean 的 ID 传递给 setBeanName() 方法。

    *   实现 BeanNameAware 接口主要是为了通过 Bean 的引用来获得 Bean 的 ID，不过一般很少用到 Bean 的 ID。
*   如果 Bean 实现了 BeanFactoryAware 接口，Spring 将调用 setBeanFactory(BeanFactory bf) 方法并把 BeanFactory 容器实例作为参数传入。

    *   实现 BeanFactoryAware 主要目的是为了获取 Spring 容器，如 Bean 通过 Spring 容器发布事件。
*   如果 Bean 实现了 ApplicationContextAware 接口，Spring 容器将调用 setApplicationContext(ApplicationContext ctx) 方法，把当前应用上下文作为参数传入。

    *   作用与 BeanFactory 类似，都是为了获取 Spring 容器。 **不同的是**：Spring 容器在调用 setApplicationContext 方法时会把它自己作为参数传入，而调用 setBeanFactory 方法前需要程序员自己指定（注入）setBeanDactory 里的 BeanFactory 参数。
*   如果 Bean 实现了 BeanPostProcessor 接口，Spring 将调用它们的 postProcessorBeforeInitialization 方法。

    *   作用是在 Bean 实例创建成功后对进行增强处理，如对 Bean 进行修改或增加某个功能。
*   如果 Bean 实现了 InitializingBean 接口，Spring 将调用它们的 afterPropertiesSet 方法

    *   作用与在配置文件中对 Bean 使用 init-method 声明初始化的作用一样，都是在 Bean 的全部属性设置成功后执行的初始化方法。
*   如果 Bean 实现了 BeanPostProcessor 接口，Spring 将调用它们的 postProcessorAfterInitialization 方法。

    *   作用与 postProcessorBeforeInitialization 一样，只不过 postProcessorBeforeInitialization 是在 Bean 初始化前执行，这个在 Bean 初始化后执行。
*   Bean 就准备就绪了，如果这个 Bean 是 singleton 的，就把它保存到容器的缓存中，如果是 prototype 的，就交给调用者。

*   如果 Bean 实现了 DisposableBean 接口，Spring 将调用它的 destroy 方法

    *   作用与在配置文件中对 Bean 使用 destory-method 属性一样，都是在 Bean 实例销毁前执行的方法。


# Spring 事务

## Spring 事务中的隔离级别

*   `TransactionDefinition.ISOLATION_DEFAULT`：使用后端数据库默认的隔离级别，Mysql 默认采用的 REPEATABLE_READ 隔离级别 Oracle 默认采用的 READ_COMMITTED 隔离级别。
*   `TransactionDefinition.ISOLATION_READ_UNCOMMITTED`：最低的隔离级别，允许读取尚未提交的数据变更，可能会导致脏读、幻读或不可重复读。
*   `TransactionDefinition.ISOLATION_READ_COMMITTED`：允许读取并发事务已经提交的数据，可以阻止脏读，但是幻读或不可重复读仍有可能发生。
*   `TransactionDefinition.ISOLATION_REPEATABLE_READ`：对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，可以阻止脏读和不可重复读，但幻读仍有可能发生。
*   `TransactionDefinition.ISOLATION_SERIALIZABLE`：最高的隔离级别，完全服从 ACID 的隔离级别。所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，该级别可以防止脏读、不可重复读以及幻读。但是这将严重影响程序的性能。通常情况下也不会用到该级别。

## Spring 事务中的事务传播行为

### 支持当前事务的情况

*   `TransactionDefinition.PROPAGATION_REQUIRED`：如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。
*   `TransactionDefinition.PROPAGATION_SUPPORTS`：如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。
*   `TransactionDefinition.PROPAGATION_MANDATORY`：如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。（mandatory：强制性）

### 不支持当前事务的情况

*   `TransactionDefinition.PROPAGATION_REQUIRES_NEW`：创建一个新的事务，如果当前存在事务，则把当前事务挂起。
*   `TransactionDefinition.PROPAGATION_NOT_SUPPORTED`：以非事务方式运行，如果当前存在事务，则把当前事务挂起。
*   `TransactionDefinition.PROPAGATION_NEVER`：以非事务方式运行，如果当前存在事务，则抛出异常。

### 其他情况

*   `TransactionDefinition.PROPAGATION_NESTED`：如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务，则该取值等价于 TransactionDefinition.PROPAGATION_REQUIRED。

## 事务怎么配置（基于 Aspectj AOP 配置事务）


# Spring MVC

## Spring MVC 原理

![image](http://490.github.io/images/20190327_193715.png)

1.  客户端将请求发送到 DispatchServlet；
2.  DispatchServlet 通过调用 HandleMapping，根据 URL 找到对应的处理器，并返回给 DispatchServlet；
3.  DispatchServlet 通过 HandleAdapter 调用 Handle；
4.  Handle 运行完会返回 ModelAndView 给 DispatchServlet；
5.  DispatchServlet 将 ModelAndView 交给 ViewResolver 解析，解析将得到具体的 View；
6.  将 Model 填充进 View 中，将渲染结果返回客户端。

## 描述从 Tomcat 开始到 Spring MVC 返回到前端显示的整个流程

在没有 Spring MVC 前，Tomcat 处理一个 HTTP 请求的具体处理流程是这样的：

*   Web 客户向 Servlet 容器（Tomcat）发出 HTTP 请求；
*   Servlet 容器分析客户的请求信息
*   Servlet 容器创建一个 HttpRequest 对象和一个 HttpResponse 对象，并将客户请求的信息封装到 HttpRequest 对象中；
*   Servlet 容器调用 HttpServlet 对象的 service 方法，把 HttpRequest 对象与 HttpResponse 对象作为参数传给 HttpServlet对象；
*   HttpServlet 调用 HttpRequest 对象的有关方法，获取 HTTP 请求信息；
*   HttpServlet 调用 HttpResponse 对象的有关方法，生成响应数据；
*   Servlet 容器把 HttpServlet 的响应结果传给 Web 客户；

我们有多少服务，就写多少个 Servlet，不过有了 Spring MVC 后，服务器里就剩一个 DispatchServlet 了，所有的 HTTP 请求都会被映射到这个 Servlet 上，请求进入到 DispatchServlet 中后，就算进入到了框架之中了，由 DispatchServlet 统一的分配 HTTP 请求到各个 Controller 中进行处理，流程详见Spring MVC 原理。



# Spring 中的设计模式

*   简单工厂
*   工厂方法
*   单例模式
*   适配器模式
*   装饰者模式
*   代理模式
*   观察者模式
*   策略模式
*   模板方法

## 工厂设计模式

Spring使用工厂模式可以通过 `BeanFactory` 或 `ApplicationContext` 创建 bean 对象。

**两者对比：**

*   `BeanFactory` ：延迟注入(使用到某个 bean 的时候才会注入),相比于`BeanFactory`来说会占用更少的内存，程序启动速度更快。
*   `ApplicationContext` ：容器启动的时候，不管你用没用到，一次性创建所有 bean 。`BeanFactory` 仅提供了最基本的依赖注入支持，`ApplicationContext` 扩展了 `BeanFactory` ,除了有`BeanFactory`的功能还有额外更多功能，所以一般开发人员使用`ApplicationContext`会更多。

ApplicationContext的三个实现类：

1.  `ClassPathXmlApplication`：把上下文文件当成类路径资源。
2.  `FileSystemXmlApplication`：从文件系统中的 XML 文件载入上下文定义信息。
3.  `XmlWebApplicationContext`：从Web系统中的XML文件载入上下文定义信息。

## 单例设计模式

在我们的系统中，有一些对象其实我们只需要一个，比如说：线程池、缓存、对话框、注册表、日志对象、充当打印机、显卡等设备驱动程序的对象。事实上，这一类对象只能有一个实例，如果制造出多个实例就可能会导致一些问题的产生，比如：程序的行为异常、资源使用过量、或者不一致性的结果。

**使用单例模式的好处:**

*   对于频繁使用的对象，可以省略创建对象所花费的时间，这对于那些重量级对象而言，是非常可观的一笔系统开销；
*   由于 new 操作的次数减少，因而对系统内存的使用频率也会降低，这将减轻 GC 压力，缩短 GC 停顿时间。

**Spring 中 bean 的默认作用域就是 singleton(单例)的。**

**Spring 实现单例的方式：**

*   xml:<bean id="userService" class="top.snailclimb.UserService" scope="singleton"/>
*   注解：`@Scope(value = "singleton")`

Spring 通过 `ConcurrentHashMap` 实现单例注册表的特殊方式实现单例模式。Spring 实现单例的核心代码如下：

```java
// 通过 ConcurrentHashMap（线程安全） 实现单例注册表
private final Map<String, Object> singletonObjects = new ConcurrentHashMap<String, Object>(64);
public Object getSingleton(String beanName, ObjectFactory<?> singletonFactory) 
{
        Assert.notNull(beanName, "'beanName' must not be null");
        synchronized (this.singletonObjects) 
        {
            // 检查缓存中是否存在实例  
            Object singletonObject = this.singletonObjects.get(beanName);
            if (singletonObject == null) 
            {
                //...省略了很多代码
                try 
                {
                    singletonObject = singletonFactory.getObject();
                }
                //...省略了很多代码
                // 如果实例对象在不存在，我们注册到单例注册表中。
                addSingleton(beanName, singletonObject);
            }
            return (singletonObject != NULL_OBJECT ? singletonObject : null);
        }
    }
    //将对象添加到单例注册表
    protected void addSingleton(String beanName, Object singletonObject) 
    {
            synchronized (this.singletonObjects)
             {
                this.singletonObjects.put(beanName, (singletonObject != null ? singletonObject : NULL_OBJECT));
            }
    }
}
```

## 代理设计模式

### 代理模式在 AOP 中的应用

AOP(Aspect-Oriented Programming:面向切面编程)能够将那些与业务无关，却为业务模块所共同调用的逻辑或责任（例如事务处理、日志管理、权限控制等）封装起来，便于减少系统的重复代码，降低模块间的耦合度，并有利于未来的可拓展性和可维护性。

**Spring AOP 就是基于动态代理的**，如果要代理的对象，实现了某个接口，那么Spring AOP会使用**JDK Proxy**，去创建代理对象，而对于没有实现接口的对象，就无法使用 JDK Proxy 去进行代理了，这时候Spring AOP会使用**Cglib** ，这时候Spring AOP会使用 **Cglib** 生成一个被代理对象的子类来作为代理，如下图所示：


![](https://mmbiz.qpic.cn/mmbiz_jpg/iaIdQfEric9TwBuibJ4N5OTyAvJibFj8b7zhPn9m0PpOfqr7exaXPcBL5qJJDiaFibZxVprZjicTJxialjXKjicQrxnOcEA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

当然你也可以使用 AspectJ ,Spring AOP 已经集成了AspectJ ，AspectJ 应该算的上是 Java 生态系统中最完整的 AOP 框架了。

使用 AOP 之后我们可以把一些通用功能抽象出来，在需要用到的地方直接使用即可，这样大大简化了代码量。我们需要增加新功能时也方便，这样也提高了系统扩展性。日志功能、事务管理等等场景都用到了 AOP 。

### Spring AOP 和 AspectJ AOP 有什么区别?

**Spring AOP 属于运行时增强，而 AspectJ 是编译时增强。** Spring AOP 基于代理(Proxying)，而 AspectJ 基于字节码操作(Bytecode Manipulation)。

Spring AOP 已经集成了 AspectJ ，AspectJ 应该算的上是 Java 生态系统中最完整的 AOP 框架了。AspectJ 相比于 Spring AOP 功能更加强大，但是 Spring AOP 相对来说更简单，

如果我们的切面比较少，那么两者性能差异不大。但是，当切面太多的话，最好选择 AspectJ ，它比Spring AOP 快很多。

## 观察者模式

观察者模式是一种对象行为型模式。它表示的是一种对象与对象之间具有依赖关系，当一个对象发生改变的时候，这个对象所依赖的对象也会做出反应。Spring 事件驱动模型就是观察者模式很经典的一个应用。Spring 事件驱动模型非常有用，在很多场景都可以解耦我们的代码。比如我们每次添加商品的时候都需要重新更新商品索引，这个时候就可以利用观察者模式来解决这个问题。

### Spring 事件驱动模型中的三种角色

#### 事件角色

`ApplicationEvent` (`org.springframework.context`包下)充当事件的角色,这是一个抽象类，它继承了`java.util.EventObject`并实现了 `java.io.Serializable`接口。

Spring 中默认存在以下事件，他们都是对 `ApplicationContextEvent` 的实现(继承自`ApplicationContextEvent`)：

*   `ContextStartedEvent`：`ApplicationContext` 启动后触发的事件;
*   `ContextStoppedEvent`：`ApplicationContext` 停止后触发的事件;
*   `ContextRefreshedEvent`：`ApplicationContext` 初始化或刷新完成后触发的事件;
*   `ContextClosedEvent`：`ApplicationContext` 关闭后触发的事件。


#### 事件监听者角色

`ApplicationListener` 充当了事件监听者角色，它是一个接口，里面只定义了一个 `onApplicationEvent（）`方法来处理`ApplicationEvent`。`ApplicationListener`接口类源码如下，可以看出接口定义看出接口中的事件只要实现了 `ApplicationEvent`就可以了。所以，在 Spring中我们只要实现 `ApplicationListener` 接口实现 `onApplicationEvent()` 方法即可完成监听事件

```java
package org.springframework.context;
import java.util.EventListener;
@FunctionalInterface
public interface ApplicationListener<E extends ApplicationEvent> extends EventListener 
{
    void onApplicationEvent(E var1);
}
```

#### 事件发布者角色

`ApplicationEventPublisher` 充当了事件的发布者，它也是一个接口。

```
@FunctionalInterfacepublic interface ApplicationEventPublisher 
{    default void publishEvent(ApplicationEvent event) 
    {        
         this.publishEvent((Object)event);    
    }    
    void publishEvent(Object var1);
}
```

`ApplicationEventPublisher` 接口的`publishEvent（）`这个方法在`AbstractApplicationContext`类中被实现，阅读这个方法的实现，你会发现实际上事件真正是通过`ApplicationEventMulticaster`来广播出去的。具体内容过多，就不在这里分析了，后面可能会单独写一篇文章提到。

### Spring 的事件流程总结

1.  定义一个事件: 实现一个继承自 `ApplicationEvent`，并且写相应的构造函数；
2.  定义一个事件监听者：实现 `ApplicationListener` 接口，重写 `onApplicationEvent()` 方法；
3.  使用事件发布者发布消息: 可以通过 `ApplicationEventPublisher` 的 `publishEvent()` 方法发布消息。


```java
// 定义一个事件,继承自ApplicationEvent并且写相应的构造函数
public class DemoEvent extends ApplicationEvent
{
    private static final long serialVersionUID = 1L;
    private String message;
    public DemoEvent(Object source,String message)
    {
        super(source);
        this.message = message;
    }
    public String getMessage() 
    {
         return message;
    }
// 定义一个事件监听者,实现ApplicationListener接口，重写 onApplicationEvent() 方法；
@Component
public class DemoListener implements ApplicationListener<DemoEvent>
{
    //使用onApplicationEvent接收消息
    @Override
    public void onApplicationEvent(DemoEvent event) 
    {
        String msg = event.getMessage();
        System.out.println("接收到的信息是："+msg);
    }
}
// 发布事件，可以通过ApplicationEventPublisher  的 publishEvent() 方法发布消息。
@Component
public class DemoPublisher 
{
    @Autowired
    ApplicationContext applicationContext;
    public void publish(String message)
    {
        //发布事件
        applicationContext.publishEvent(new DemoEvent(this, message));
    }
}
```


## 适配器模式

适配器模式(Adapter Pattern) 将一个接口转换成客户希望的另一个接口，适配器模式使接口不兼容的那些类可以一起工作，其别名为包装器(Wrapper)。

### spring AOP中的适配器模式

我们知道 Spring AOP 的实现是基于代理模式，但是 Spring AOP 的增强或通知(Advice)使用到了适配器模式，与之相关的接口是`AdvisorAdapter` 。Advice 常用的类型有：`BeforeAdvice`（目标方法调用前,前置通知）、`AfterAdvice`（目标方法调用后,后置通知）、`AfterReturningAdvice`(目标方法执行结束后，return之前)等等。每个类型Advice（通知）都有对应的拦截器:`MethodBeforeAdviceInterceptor`、`AfterReturningAdviceAdapter`、`AfterReturningAdviceInterceptor`。Spring预定义的通知要通过对应的适配器，适配成 `MethodInterceptor`接口(方法拦截器)类型的对象（如：`MethodBeforeAdviceInterceptor` 负责适配 `MethodBeforeAdvice`）。

### spring MVC中的适配器模式

在Spring MVC中，`DispatcherServlet` 根据请求信息调用 `HandlerMapping`，解析请求对应的 `Handler`。解析到对应的 `Handler`（也就是我们平常说的 `Controller` 控制器）后，开始由`HandlerAdapter` 适配器处理。`HandlerAdapter` 作为期望接口，具体的适配器实现类用于对目标类进行适配，`Controller` 作为需要适配的类。

**为什么要在 Spring MVC 中使用适配器模式？** Spring MVC 中的 `Controller` 种类众多，不同类型的 `Controller` 通过不同的方法来对请求进行处理。如果不利用适配器模式的话，`DispatcherServlet` 直接获取对应类型的 `Controller`，需要的自行来判断，像下面这段代码一样：

```
if(mappedHandler.getHandler() instanceof MultiActionController)
{     ((MultiActionController)mappedHandler.getHandler()).xxx  
}
else if(mappedHandler.getHandler() instanceof XXX)
{      ...  }
else if(...)
{     ...  }  
```

假如我们再增加一个 `Controller`类型就要在上面代码中再加入一行 判断语句，这种形式就使得程序难以维护，也违反了设计模式中的开闭原则 – 对扩展开放，对修改关闭。

## 装饰者模式

装饰者模式可以动态地给对象添加一些额外的属性或行为。相比于使用继承，装饰者模式更加灵活。简单点儿说就是当我们需要修改原有的功能，但我们又不愿直接去修改原有的代码时，设计一个Decorator套在原有代码外面。其实在 JDK 中就有很多地方用到了装饰者模式，比如 `InputStream`家族，`InputStream` 类下有 `FileInputStream` (读取文件)、`BufferedInputStream` (增加缓存,使读取文件速度大大提升)等子类都在不修改`InputStream` 代码的情况下扩展了它的功能。
![](https://mmbiz.qpic.cn/mmbiz_png/iaIdQfEric9TwBuibJ4N5OTyAvJibFj8b7zh3apBemUibszLADBA4wIW10sYZcPXiaBrSaJUTPjVY5EY03Pu65eeDdNA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)


Spring 中配置 DataSource 的时候，DataSource 可能是不同的数据库和数据源。我们能否根据客户的需求在少修改原有类的代码下动态切换不同的数据源？这个时候就要用到装饰者模式(这一点我自己还没太理解具体原理)。Spring 中用到的包装器模式在类名上含有 `Wrapper`或者 `Decorator`。这些类基本上都是动态地给一个对象添加一些额外的职责

## 总结

Spring 框架中用到了哪些设计模式：

*   **工厂设计模式** : Spring使用工厂模式通过 `BeanFactory`、`ApplicationContext` 创建 bean 对象。
*   **代理设计模式** : Spring AOP 功能的实现。
*   **单例设计模式** : Spring 中的 Bean 默认都是单例的。
*   **模板方法模式** : Spring 中 `jdbcTemplate`、`hibernateTemplate` 等以 Template 结尾的对数据库操作的类，它们就使用到了模板模式。
*   **包装器设计模式** : 我们的项目需要连接多个数据库，而且不同的客户在每次访问中根据需要会去访问不同的数据库。这种模式让我们可以根据客户的需求能够动态切换不同的数据源。
*   **观察者模式:** Spring 事件驱动模型就是观察者模式很经典的一个应用。
*   **适配器模式** :Spring AOP 的增强或通知(Advice)使用到了适配器模式、spring MVC 中也是用到了适配器模式适配`Controller`。

*   ……
[javaguide公众号](https://mp.weixin.qq.com/s?__biz=Mzg2OTA0Njk0OA==&mid=2247485303&idx=1&sn=9e4626a1e3f001f9b0d84a6fa0cff04a&chksm=cea248bcf9d5c1aaf48b67cc52bac74eb29d6037848d6cf213b0e5466f2d1fda970db700ba41&scene=0&xtrack=1&key=5d2b73313be61eb1c0068be1cdc521e96f90502d857b37b3622d7c2d09ac076594f25f23787ead7b5f15b6a9cd462b80cc9a1d29594129009adec7efe959a497dea66101d3e979f5249a8de7b684ab69&ascene=1&uin=MjA2MDMxMjkyMQ%3D%3D&devicetype=Windows+10&version=62060739&lang=zh_CN&pass_ticket=2z37pBUCd7Q96bYDF9Rg88qSfg4FrpiJC%2BFaSvk21jAn75bdwPk8RLIlOAIVdH4A)






#  Spring依赖注入的3种方式

常用的注入方式主要有三种：构造方法注入，setter注入，基于注解的注入。

## 通过构造方法注入bb

通过构造方法注入，就相当于给构造方法的参数传值。Bean必须提供带参数的构造函数

set注入的缺点是无法清晰表达哪些属性是必须的，哪些是可选的，构造注入的优势是通过构造强制依赖关系，不可能实例化不 完全的或无法使用的bean。


```java
MemberBean定义四个变量，

private String name;
private Double salary;
private Dept dept;
private String sex;

加上构造方法和toString方法：方便测试

Dept：
private String dname;
private String deptno;
```

```
第一种方法：根据索引赋值，索引都是以0开头的：

 <bean id="memberBean" class="www.csdn.spring01.constructor.MemberBean">
         <constructor-arg index="0" value="刘晓刚" />
         <constructor-arg index="1" value="3500" />
         <constructor-arg index="2" ref="dept"/>
         <constructor-arg index="3" value="男" />

第二种方法是根据所属类型传值
这种方法基本上不怎么适用，因为一个类里可以有好几个相同基本类型的变量，很容易就混淆值传给哪一个参数了所以做好不要使用这种方法：

         <constructor-arg type="java.lang.String" value="刘晓刚" 
         <constructor-arg type="java.lang.Double" value="3500" />
         <constructor-arg type="www.csdn.spring01.constructor.Dept" ref="dept"/>
         <constructor-arg type="java.lang.String" value="男" /> 

第三种方法：根据参数的名字传值：（推荐用法）
在这几种方法里我感觉这种方法是最实用的，他是根据名字来传值的，所以基本上只要名字对了，这个值就可以获取到  

         <constructor-arg name="name" value="刘晓刚" />
         <constructor-arg name="salary" value="3500" />
         <constructor-arg name="dept" ref="dept"/>
         <constructor-arg name="sex" value="男" />

第四种方法：直接传值
直接给参数赋值，这种方法也是根据顺序排的，所以一旦调换位置的话，就会出现bug，这种方法已经很原始了

         <constructor-arg  value="刘晓刚" />
         <constructor-arg  value="3500" />
         <constructor-arg  ref="dept"/>
         <constructor-arg  value="男" />
    </bean>

<bean id="dept" class="www.csdn.spring01.constructor.Dept" >
  <property name="dname" value="北航"/>
  <property name="deptno" value="00001"/>
</bean>
```


# 构造函数注入的 Spring IoC 

## IoC 概述

IoC (Inversion of Control)，即控制反转，或者称其为依赖注入更好理解一些，不过这个翻译还是比较晦涩难懂的，所以我们举一个吃西红柿炒鸡蛋的例子来说明一下
<!--more-->
之前我们要吃西红柿炒鸡蛋，是要自己做的，我们得先上下厨房查菜谱，去超市买西红柿（new Tomato），买鸡蛋（new Egg），然后把它俩放搅和到一块，我们才能有西红柿炒鸡蛋吃。就像这样：

![image](http://490.github.io/images/20190424_215419.png)

可以看到，我们要 5 行代码才能得到 twe 对象（一盘西红柿炒鸡蛋），而且想吃一回，就给重复一次这 5 行代码，真的是相当的麻烦，可是这只是做法相当简单的西红柿炒鸡蛋啊，这要是想吃个圣诞节烤鸡就不要想着自己做了，即使能找到齐全的菜谱，正常人应该也懒得自己做，所以我们选择冲向饭店，也就是 Spring 容器：

![image](http://490.github.io/images/20190424_215427.png)

现在，我们把 `菜谱.xml` 给饭店了，只要 Spring 容器初始化好了（饭店开门了），我们就对服务员（ac 对象）说：“给我来一盘西红柿炒鸡蛋”，然后服务员就会给我们端来一盘西红柿炒鸡蛋（ac.getBean("TomatoWithEgg")），只要想吃就向服务员要就行，是不是非常的方便呀。

而且这个饭店还是高度定制化的，你想吃啥就把 `菜谱.xml` 给它，它就会丝毫不差的照着菜谱给你做你想要的菜（构造你想要的对象），如果你想改造一下菜的做法，只消改一下给饭店的 `菜谱.xml` 就行，然后 Spring 容器就会照着新的 `菜谱.xml` 给你造对象。这相当于将应用程序的配置和依赖性规范与实际的应用程序代码分开，也就是说，我们现在要是想往西红柿炒鸡蛋里面放土豆块，只消在 `菜谱.xml` 中的原料表中加上土豆块，然后给 TomatoWithEgg 的属性中加上土豆块就可以了，不用去代码中搜索所有用到 TomatoWithEgg 对象的地方，再给它加上土豆块属性了。

也就是说，可以把 IoC 模式看做工厂模式的升华，IoC 就是一个大工厂，这个大工厂要生成的对象在 XML 中给出定义，然后就可以利用 Java 反射，根据 XML 中给出的类名生成相应的对象。这种做法有比工厂模式高级在哪里呢？以前的工厂模式都是在工厂方法中把对象的生成代码写死，而 Spring 容器则是由 XML 文件来定义类与类之间的关系，将工厂和对象生成两者独立开来。简单来说， **Spring IoC 就是我们把类与类之间的依赖关系在 XML 配置文件中写好，就不再自己 new 对象了，之后我们要用哪个对象就向 Spring 要，Spring 会帮我们按照我们定义的依赖关系给我们创建好相应的对象。**



## 技术准备

要实现一个简单的 IoC 框架，我们需要使用到如下技术：

- 解析 XML 文件（dom4j & XPath）
- 反射和内省（BeanUtils）

我们先在这里简单的写一下我们会用到的地方，至于详细的介绍可以看：

- [010-XML.md](./doc/010-XML.md)
- [011-反射与内省.md](./doc/011-反射与内省.md)

首先，我们要导包：

```xml
<!-- 解析 XML 要用到的包 -->
<dependency>
    <groupId>dom4j</groupId>
    <artifactId>dom4j</artifactId>
    <version>1.6.1</version>
</dependency>

<dependency>
    <groupId>jaxen</groupId>
    <artifactId>jaxen</artifactId>
    <version>1.1.6</version>
</dependency>

<!-- 通过字符串类名创建对应类的对象并且为对象注入属性要用到的包 -->
<dependency>
    <groupId>commons-beanutils</groupId>
    <artifactId>commons-beanutils</artifactId>
    <version>1.9.3</version>
</dependency>
```

### dom4j 的简单使用

```java
// 获取xml配置文件的输入流
InputStream xmlConfigFile = ConfigManager.class.getResourceAsStream(path);
// 创建xml文件解析器
SAXReader reader = new SAXReader();
try {
    // 得到xml文件的Document对象
    Document doc = reader.read(xmlConfigFile);
    // 取出所有<bean>标签
    List<Element> beans = doc.selectNodes("//bean");
    // 遍历<bean>标签
    for (Element bElement : beans) {
        // 取出name属性值
        String bName = bElement.attributeValue("name");
        // 取出bean下的<property>标签
        List<Element> properties = bElement.elements("property");
    }
} catch (DocumentException e) {}
```

### BeanUtils 的简单使用

```java
User user = new User();
BeanUtils.setProperty(user, "username", "admin");
BeanUtils.setProperty(user, "password", "admin123");
```



## 实现流程

整个实现流程分为以下几步：

- 读取配置文件，将配置文件信息加载进容器（dom4j）；
- 根据配置文件初始化容器，创建容器需要创建的 Bean 对象（BeanUtils）；
- 通过 `getBean(String beanName)` 方法获取我们想要的 Bean 对象。

### 解析配置文件

我们需要做的一切都是要基于配置文件的，所有先来看一下配置文件的格式：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans>
    <bean name="A" class="org.simplespring.model.A">
        <property name="strData" value="A的strData"/>
        <property name="intData" value="123"/>
    </bean>

    <bean name="B" class="org.simplespring.model.B">
        <property name="refData" ref="A"/>
    </bean>

    <bean name="C" class="org.simplespring.model.C" scope="prototype">
        <property name="strData" value="C的strData"/>
    </bean>
    
    <bean name="D" class="org.simplespring.model.D">
        <constructor-arg index="0" type="java.lang.String" value="D的name属性值"/>
        <constructor-arg index="1" type="java.lang.String" value="D的value属性值"/>
        <constructor-arg index="2" type="org.simplespring.model.A" ref="A"/>
    </bean>
</beans>
```

我们配置了 3 个类：A、B、C，其中 A 和 B 是 singleton 的，也就是说是单例的，只在容器初始化时创建一次，每次调用 getBean 返回的都是同一个对象；C 是多例的，即每次调用 getBean 都返回一个新创建的对象。

我们首先需要考虑的事情是，我们解析出来的 xml 文件的内容应该放在哪里呢？我们采用的解决方案是：创建 3 个 Model 类：`Bean` 、 `Property` 和 `ConstructorArg`，专门放解析出来的 `<bean>` 、 `<property>` 和 `<constructor-arg>` 标签。然后我们把解析出来的 Bean 类放到一个 `Map<String, Bean>` 中，其中 key 是我们配置的 bean 标签的 name，value 就是对应的 Bean 对象。这部分的实现在 `org.simplespring.config.parse.ConfigManager.java` 中：

- **方法：** `public static Map<String, Bean> getConfig(String path)`
- **流程：**
  - 创建解析器：SAXReader
  - 加载 XML 文件
  - 定义 xpath 表达式：`"//bean"`，取出所有 Bean
  - 对 Bean 元素进行遍历，并通过 `setBean` 方法进行解析
    - 将 `<bean>` 标签的 name & class & scope 属性封装到 Bean 对象中
    - 调用 `setBeanConstructorArg(Element bElement, Bean bean)` 方法，解析 `<constructor-arg>` 标签
    - 调用 `setBeanProperties(Element bElement, Bean bean)` 方法，解析 `<property>` 标签
    - 将 Bean 对象封装到 Map res 中（这个 Map 就是我们的返回结果）
  - 返回 Map res

`<property>` 标签的解析较为简单，而 `<constructor-arg>` 标签的解析却不是那么的容易，因为如果像 `<property>` 标签一样，在 Bean 中仅仅通过一个 `List<Property>` 来存储的话，我们之后将很难通过配置文件给出的配置信息来确定到底应该执行哪一个构造函数来创建对象。因此，为了方便后面构造函数匹配的实现，我们在 Bean 中通过以下结构来存储 `<constructor-arg>` 标签的信息：

```java
/** 用来存储通过索引定位的构造函数参数 */
private final Map<Integer, ConstructorArg> indexConstructorArgs = new HashMap<>();

/** 用来存储不索引定位的构造函数参数 */
private final List<ConstructorArg> genericConstructorArgs = new ArrayList<>();
```

因此，在解析 `<constructor-arg>` 标签时，优先解析 index 属性，将解析结果放入 Bean 的 `Map<Integer, ConstructorArg> indexConstructorArgs` 集合中，如果该标签没有 index 属性，则将解析结果放入 Bean 的 `List<ConstructorArg> genericConstructorArgs` 集合中。

具体实现详见：[ConfigManager.java](./src/main/java/org/simplespring/config/parse/ConfigManager.java).

### 根据配置文件解析结果初始化容器

可以成功的解析配置文件之后，我们就可以进行容器的初始化了，我们先构造一个 `BeanFactory` 接口，容器的实现类都实现于这个接口，这个接口中有一个 `getBean(String beanName)` 方法，用于从容器中获取我们在配置文件中配置的对象。

```java
public interface BeanFactory {
    public Object getBean(String beanName);
}
```

然后，我们写一个这个接口的实现类：`org.simplespring.main.impl.ClassPathXmlApplicationContext`，来真正的进行容器的初始化。

首先我们要在 ClassPathXmlApplicationContext 的构造函数中调用我们上一小节写的 `ConfigManager.getConfig(path)` 加载配置文件进来，然后根据配置文件的配置内容，创建相应的对象，并放入一个 `Map<String, Object>` 中，其中 key 是我们在配置文件中给该对象配置的 name，value 是容器创建的对象，也是我们之后要通过 getBean 从容器中取的东西。

也就是说，ClassPathXmlApplicationContext 的初始化过程中，我们要完成：

- 读取配置文件中需要初始化的 Bean 信息： `ConfigManager.getConfig()`
- 遍历配置对象 Map，初始化所有不是 prototype 的 Bean
	- 通过方法 `Object object = createBean(beanInfo);` 完成
	- 将初始化好的 Bean 放入 `Map<String, Object> beanMap` 中

接下来就是实现关键的 createBean 方法了：

- **方法：** `private Object createBean(Bean beanInfo)`
- **流程：**
  - 判断容器中是否已经存在该实例，如果存在，直接返回即可
  - 获取要创建的 Bean 的 Class，调用 `newObject(Bean beanInfo, Class beanClass)` 方法创建对象
  - 获取 Bean 需要的属性对象，将其注入到Bean中
    - value 属性注入：`prop.getValue() != null`
    	- `BeanUtils.setProperty(object, property.getName(), property.getValue())`
    - ref 属性注入：`prop.getRef() != null`
    	- 先来判断一下要加载的 ref 类是否已经创建并放入容器中了
    		- 不存在：递归调用 createBean 方法
    		- 存在：直接从容器中取出并注入
  - 返回创建好的 object

在创建对象时，我们调用了 `newObject(Bean beanInfo, Class beanClass)` 方法，在没有配置 constructor-arg 时，Spring 容器会选择使用无参构造函数创建对象，及直接调用 `beanClass.newInstance()` 方法完成对象的创建。但是当配置文件中配置了 constructor-arg 时，容器就需要通过配置文件中 `<constructor-arg>` 标签的 type 和 index 属性的配置来选择合适的构造函数创建实例了，具体的匹配方法将会在 ConstructorResolver.matchConstructor 方法的实现流程中进行详细说明，我们首先来看一下 newObject 方法的实现，以下是 newObject 方法的详细实现流程：

- **方法：** `public Object newObject(Bean beanInfo, Class beanClass)`
- **流程：**
	- 如果 beanInfo 中没有 constructor-arg 的配置信息，直接调用 `beanClass.newInstance()` 创建对象返回
	- 调用 `ConstructorResolver.matchConstructor(Bean beanInfo, Class beanClass, BeanFactory beanFactory)` 方法搜索匹配的构造函数及其参数列表
	- 如果能搜索到，则通过反射调用构造函数创建对象，如果无法搜索到则抛出异常

createBean 和 newObject 方法的具体实现详见：[ClassPathXmlApplicationContext.java](./src/main/java/org/simplespring/main/impl/ClassPathXmlApplicationContext.java).

`ConstructorResolver.matchConstructor(Bean beanInfo, Class beanClass, BeanFactory beanFactory)` 方法的详细实现流程：

- **方法：** `public static ArgumentsHolder matchConstructor(Bean beanInfo, Class beanClass, BeanFactory beanFactory)`
- **流程：**
	- 从 beanClass 中取出该类所有的构造函数，并对构造函数们按照 public 在前，参数个数多的在前的顺序进行排序，以方便后面的匹配操作
	- 遍历所有的构造函数，并将每个构造函数的参数数组与配置文件中配置的参数进行匹配
		- 首先按照构造函数的入参个数进行匹配
		- 匹配到入参个数相同的构造函数时，根据 beanInfo 中读取的构造函数参数配置信息，生成对应于当前构造函数的参数列表，生成的过程对参数的类型进行匹配，只有每个位置的参数都符合配置文件中配置的构造函数才能作为匹配结果返回，在遍历参数列表过程中：
			- 先获取配置文件中配置在该 index 下的参数
			- 如果当前位置未设置参数，就按照类型搜索参数
			- 如果按照类型也搜索不到就获取 `beanInfo.genericConstructorArgs` 中第一个没有使用的参数作为该位置的参数
			- 获得当前位置的参数后，对类型匹配判断，如果类型与当前正在匹配的构造函数相应位置的入参相同，则匹配成功，可以继续匹配下一个参数，如果类型匹配不成功，则当前构造函数匹配失败，跳过该构造函数，继续匹配下一个构造函数

ConstructorResolver 类的具体实现详见：[ConstructorResolver.java](./src/main/java/org/simplespring/support/ConstructorResolver.java).

通过以上复杂的构造函数匹配流程，我们也可以理解为什么一般建议使用 set/get 方法的方式注入属性了，因为构造函数注入属性的方式除了存在循环依赖的问题，在容器初始化的时候，由于需要匹配合适的构造函数，会增加容器的初始化时间。

### getBean 返回所需对象

getBean 方法的实现十分简单，流程如下：

- **方法：** `public Object getBean(String beanName)`
- **流程：**
  - 从 Map configs 中取出该 bean 的配置，判断 scope 属性
    - scope == "prototype"：调用 createBean 新创建一个对象返回
    - scope != "prototype"：直接从 beanMap 中取出对象返回

### 测试

我们在 `ClassPathXmlApplicationContextTest` 进行了简单的测试：

```java
String path = "/applicationContext.xml";
BeanFactory factory = new ClassPathXmlApplicationContext(path);
A a1 = (A) factory.getBean("A");
B b1 = (B) factory.getBean("B");
C c1 = (C) factory.getBean("C");

A a2 = (A) factory.getBean("A");
C c2 = (C) factory.getBean("C");

D d1 = (D) factory.getBean("D");

System.out.println(a1);
System.out.println(b1);
System.out.println(c1);
System.out.println(a2);
System.out.println(c2);
System.out.println(d1);
```

输出结果：

```
创建A对象一次
创建B对象一次
创建C对象一次
创建C对象一次
A{strData='A的strData', intData=123}
B{refData=A{strData='A的strData', intData=123}}
C{strData='C的strData'}
A{strData='A的strData', intData=123}
C{strData='C的strData'}
D{nameAttribute='D的name属性值', valueAttribute='D的value属性值', refAttribute=A{strData='A的strData', intData=123}, doubleAttribute=0.0}
```

我们创建了 2 个 A 和 C 对象，一个 B 对象，通过观察输出结果可以发现，A 对象只会被创建一次，而 C 对象被创建了两次（根据构造函数的执行次数判断的，ABC 每个类的构造函数都会 print "创建X对象一次"），并且我们在配置文件中配置的属性也被注入进了相应的对象，A 对象中的 int 型和 String 型数据都被很好的注入了。至此，简单的 Spring IoC 框架实现完毕。

# 注解注入

之前tiny-spring已经实现了通过xml配置类的方式自动装配和依赖注入，现在要给tiny-spring框架加入自动扫描包下的类，再执行自动装配和依赖注入。

流程步骤可以分为：

- 类加载器获取包路径；
- 扫描并加载路径下的类集合；
- 将扫描到的类集合解析成BeanDefinition对象集合；
- 交给自动装配和依赖注入；

步骤1~2可以归结为获取指定包下面的类集合，然后再解析，最后自动装配和注入。

# 统一异常处理 

 @ ExceptionHandler

# DAO条件查询

```java
public Page<Lable> pageSearch(int page, int size, Lable lable) 
{
        Pageable pageable = PageRequest.of(page - 1, size);
        return lableDao.findAll(new Specification<Lable>() //匿名内部类？还是叫什么
        {
            List<Predicate> list = new ArrayList<>();
            @Override
            public Predicate toPredicate(Root<Lable> root, CriteriaQuery<?> criteriaQuery, CriteriaBuilder criteriaBuilder) 
            {
                if (lable.getLabelname() != null && !"".equals(lable.getLabelname())) 
                {
                    Predicate predicate = criteriaBuilder.like(root.get("lablename").as(String.class), "%" + lable.getLabelname() + "%");
                    list.add(predicate);
                }
                if (lable.getFans() != null && !"".equals(lable.getFans())) 
                {
                    Predicate predicate = criteriaBuilder.equal(root.get("fans").as(String.class), "='" + lable.getFans());
                    list.add(predicate);
                }
                Predicate[] arr = new Predicate[list.size()];//必须获取长度才能初始化
                list.toArray(arr);
                return criteriaBuilder.and(arr);
            }
        }, pageable);
    }
```



# 跨域
**同源策略**[same origin policy]是浏览器的一个安全功能，不同源的客户端脚本在没有明确授权的情况下，不能读写对方资源。 同源策略是浏览器安全的基石。

### 什么是源

**源**[origin]就是协议、域名和端口号。例如：http://www.baidu.com:80这个URL。

### 什么是同源

若地址里面的协议、域名和端口号均相同则属于同源。

### 是否是同源的判断

例如判断下面的`URL`是否与 http://www.a.com/test/index.html 同源

*   http://www.a.com/dir/page.html 同源
*   http://www.child.a.com/test/index.html 不同源，域名不相同
*   https://www.a.com/test/index.html 不同源，协议不相同
*   http://www.a.com:8080/test/index.html 不同源，端口号不相同

### 哪些操作不受同源策略限制

1.  页面中的链接，重定向以及表单提交是不会受到同源策略限制的；
2.  跨域资源的引入是可以的。但是`JS`不能读写加载的内容。如嵌入到页面中的`<script src="..."></script>`，`<img>`，`<link>`，`<iframe>`等。

### 跨域

受前面所讲的浏览器同源策略的影响，不是同源的脚本不能操作其他源下面的对象。想要操作另一个源下的对象就需要跨域。 在同源策略的限制下，_非同源_的网站之间不能发送 `AJAX` 请求。

### 如何跨域

*   降域

    可以通过设置 `document.damain='a.com'`，浏览器就会认为它们都是同一个源。想要实现以上任意两个页面之间的通信，两个页面必须都设置`documen.damain='a.com'`。

*   `JSONP`跨域

*   `CORS` 跨域



## Spring Boot 配置 CORS

### 1、使用`@CrossOrigin` 注解实现

`#`如果想要对某一接口配置 `CORS`，可以在方法上添加 `@CrossOrigin` 注解 ：

```
@CrossOrigin(origins = {"http://localhost:9000", "null"})
@RequestMapping(value = "/test", method = RequestMethod.GET)
public String greetings() {
    return "{\"project\":\"just a test\"}";
}
```

`#`如果想对一系列接口添加 CORS 配置，可以在类上添加注解，对该类声明所有接口都有效：

```
@CrossOrigin(origins = {"http://localhost:9000", "null"})
@RestController
@SpringBootApplication
public class SpringBootCorsTestApplication {

}
```

`#`如果想添加全局配置，则需要添加一个配置类 ：

```
@Configuration
public class WebMvcConfig extends WebMvcConfigurerAdapter {

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                .allowedOrigins("*")
                .allowedMethods("POST", "GET", "PUT", "OPTIONS", "DELETE")
                .maxAge(3600)
                .allowCredentials(true);
    }
}
```

另外，还可以通过添加 Filter 的方式，配置 CORS 规则，并手动指定对哪些接口有效。

```
@Bean
public FilterRegistrationBean corsFilter() {
    UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
    CorsConfiguration config = new CorsConfiguration();
    config.setAllowCredentials(true);   config.addAllowedOrigin("http://localhost:9000");
    config.addAllowedOrigin("null");
    config.addAllowedHeader("*");
    config.addAllowedMethod("*");
    source.registerCorsConfiguration("/**", config); // CORS 配置对所有接口都有效
    FilterRegistrationBean bean = newFilterRegistrationBean(new CorsFilter(source));
    bean.setOrder(0);
    return bean;
}
```