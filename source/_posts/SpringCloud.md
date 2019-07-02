---
title: SpringCloud
date: 2019-06-04 16:47:36
tags: Spring
---

微服务
https://blog.lqdev.cn/categories/SpringCloud/
<!--more-->
# 微服务架构的设计原则

拆分足够微、轻量级通信、领域驱动原则

# 如何设计

服务拆分、服务注册、服务发现、服务消费

统一入口、配置管理、熔断机制、自动扩展


把各种service拆开，每个注册eureka。有调用的设置feign

# eureka
https://blog.lqdev.cn/2018/09/06/SpringCloud/chapter-two/

默认情况下，如果`Eureka Server`在一定时间内没有接收到某个微服务实例的心跳，`Eureka Server`将会注销该实例（默认90秒）。但是当网络分区故障发生时，微服务与Eureka Server之间无法正常通信，这就可能变得非常危险了，因为微服务本身是健康的，此时本不应该注销这个微服务。

`Eureka Server`通过“自我保护模式”来解决这个问题，当`Eureka Server`节点在短时间内丢失过多客户端时（可能发生了网络分区故障），那么这个节点就会进入自我保护模式。一旦进入该模式，`Eureka Server`就会保护服务注册表中的信息，不再删除服务注册表中的数据（也就是不会注销任何微服务）。当网络故障恢复后，该`Eureka Server`节点会自动退出自我保护模式。

**自我保护模式是一种对网络异常的安全保护措施。使用自我保护模式，而让Eureka集群更加的健壮、稳定。**

开发阶段可以通过配置：`eureka.server.enable-self-preservation=false`关闭自我保护模式。


## @EnableDiscoveryClient
在启动类上，注册到eureka server
```
spring.application.name: micro-weather-config-client

eureka.client.serviceUrl.defaultZone: http://localhost:8761/eureka/

spring.cloud.config.profile=dev
spring.cloud.config.uri=http://localhost:8888/
```

## @EnableEurekaServer
注册服务中心

```
server.port: 8761

eureka.instance.hostname: localhost
eureka.client.registerWithEureka: false
eureka.client.fetchRegistry: false
eureka.client.serviceUrl.defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/

eureka.server.enable-self-preservation=false
```










# 微服务的消费者

## httpclient
## ribbon

`Spring Cloud Ribbon`是一个基于Http和TCP的客服端负载均衡工具，它是基于`Netflix Ribbon`实现的。与`Eureka`配合使用时，`Ribbon`可自动从`Eureka Server (注册中心)`获取服务提供者地址列表，并基于`负载均衡`算法，通过在客户端中配置`ribbonServerList`来设置服务端列表去轮询访问以达到均衡负载的作用。

**创建一个工程：`spring-cloud-eureka-consumer-ribbon`**
(其实这个工程和`spring-cloud-eureka-consumer`是差不多的，只是有些许不同。)

0.加入pom依赖
```
<!-- 客户端依赖 -->
 <dependency>
 <groupId>org.springframework.cloud</groupId>
 <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
 </dependency>
 <dependency>
 <groupId>org.springframework.boot</groupId>
 <artifactId>spring-boot-starter-web</artifactId>
 </dependency>
```

1.配置文件修改，添加注册中心等相关信息。
```
spring.application.name=eureka-consumer-ribbon
server.port=8018

#指定注册中心地址
eureka.client.service-url.defaultZone=http://127.0.0.1:1000/eureka
# 启用ip配置 这样在注册中心列表中看见的是以ip+端口呈现的
eureka.instance.prefer-ip-address=true
# 实例名称  最后呈现地址：ip:2000
eureka.instance.instance-id=${spring.cloud.client.ip-address}:${server.port}
```
 
2.编写启动类，加入`@EnableDiscoveryClient`，同时申明一个`RestTemplate`，**这里和原先不同，就在于加入了`@LoadBalanced`注解进行修饰`RestTemplate`类，稍后会大致讲解下是如何进行实现的。**

```
@SpringBootApplication
@EnableDiscoveryClient
@Slf4j
public class EurekaConsumerRibbonApplication 
{
 public static void main(String[] args) throws Exception 
 {
     SpringApplication.run(EurekaConsumerRibbonApplication.class, args);
     log.info("spring-cloud-eureka-consumer-ribbon启动!");
 }
 //添加 @LoadBalanced 使其具备了使用LoadBalancerClient 进行负载均衡的能力
 @Bean
 @LoadBalanced
 public RestTemplate restTemplage() 
 {
     return new RestTemplate();
 }
}
```

3.编写测试类，进行服务调用。
```
/**
 * ribbon访问客户端示例
 * @author oKong
 *
 */
@RestController
@Slf4j
public class DemoController 
{
   @Autowired 
   RestTemplate restTemplate;
   @GetMapping("/hello") 
   public String hello(String name) 
   {
 //直接使用服务名进行访问
       log.info("请求参数name:{}", name);
       return restTemplate.getForObject("http://eureka-client/hello?name=" + name, String.class);
   } 
}
```

可以看见，可以直接注入`RestTemplate`，通过服务名直接调用.

4.启动应用，访问:(http://127.0.0.1:8018/hello?name=oKong) ,可以看见调用成功：

[![hello](http://qiniu.xds123.cn/18-9-20/20652450.jpg)](http://qiniu.xds123.cn/18-9-20/20652450.jpg "hello")
控制台输出：

[![控制台输出](http://qiniu.xds123.cn/18-9-20/75877117.jpg)](http://qiniu.xds123.cn/18-9-20/75877117.jpg "控制台输出")



## feign


哪个微服务A里面依赖了另一个微服务B，就把A改造成feign，这个客户端去调用B，这个客户端在Acontroller里像service一样用@Autowire注册。


一个类似于 Java HTTP 客户端的工具 Feign 来访问远程 HTTP 服务器；用于服务之间的调用

虽然说我们可以采用 RestTemplate、URLConnection、Netty、HttpClient都可以访问远端 HTTP 服务器，但是使用 Feign 来说，Feign 可以做到使用 HTTP 请求远程服务时就像调用本地的方法一样，让开发者完全感知不到这是在调用远端服务，感觉无非就是调用一个 API 方法一样；

当我们使用 Feign 的时候，SpringCloud 整合了 Ribbon 和 Eureka 去提供负载均衡；


- Feign的源码实现的过程如下：
 首先通过@EnableFeignCleints注解开启FeignCleint
 根据Feign的规则实现接口，并加@FeignCleint注解
 程序启动后，会进行包扫描，扫描所有的@ FeignCleint的注解的类，并将这些信息注入到ioc容器中。
 当接口的方法被调用，通过jdk的代理，来生成具体的RequesTemplate
 RequesTemplate在生成Request
 Request交给Client去处理，其中Client可以是HttpUrlConnection、HttpClient也可以是Okhttp
 最后Client被封装到LoadBalanceClient类，这个类结合类Ribbon做到了负载均衡。




### @EnableFeignClients

```
spring.application.name: msa-weather-collection-eureka-feign

eureka.client.serviceUrl.defaultZone: http://localhost:8761/eureka/

feign.client.config.feignName.connectTimeout: 5000
feign.client.config.feignName.readTimeout: 5000
```

### @FeignClient
value=要调用的服务名

```java
如下，可以调用服务名的cities接口
@FeignClient("msa-weather-city-eureka")
public interface CityClient {
	@GetMapping("/cities")
	String listCity();
}
```


```java
使用上面声明好的客户端、
@RestController
public class CityController {
	@Autowired
	private CityClient cityClient;
	@GetMapping("/cities")
	public String listCity() {
		// 通过Feign客户端来查找
		String body = cityClient.listCity();
		return body;
	}
}
```




# API网关


在微服务框架中，每个对外服务都是独立部署的，对外的api或者服务地址都不是不尽相同的。对于内部而言，很简单，通过注册中心自动感知即可。但我们大部分情况下，服务都是提供给外部系统进行调用的，不可能同享一个注册中心。同时一般上内部的微服务都是在内网的，和外界是不连通的。而且，就算我们每个微服务对外开放，对于调用者而言，调用不同的服务的地址或者参数也是不尽相同的，这样就会造成消费者客户端的复杂性，同时想想，可能微服务可能是不同的技术栈实现的，有的是`http`、`rpc`或者`websocket`等等，也会进一步加大客户端的调用难度。所以，**一般上都有会有个api网关，根据请求的url不同，路由到不同的服务上去，同时入口统一了，还能进行统一的身份鉴权、日志记录、分流等操作**。

## 客户端和服务端直连的弊端

*   客户端会对此请求不同的微服务，增加客户端复杂性
*   存在跨域请求时，需要进行额外处理
*   认证服务，每个服务需要独立认证
*   UI端和微服务耦合
* 
网关的优缺点

**优点**：

*   减少api请求次数
*   限流
*   缓存
*   统一认证
*   降低微服务的复杂度
*   支持混合通信协议(前端只和api通信，其他的由网关调用)
*   ……

**缺点**：

*   网关需高可用，可能产生单点故障
*   管理复杂


## Nginx
服务端负载均衡
## zuul


**Zuul的核心一系列的过滤器**：

- 身份认证与安全：识别每个资源的验证要求，并拒绝那些与要求不符的请求。
- 审查与监控：在边缘位置追踪有意义的数据和统计结果，从而带来精确的生产视图。
- 动态路由：动态地将请求路由到不同的后端集群。
- 压力测试：逐渐增加指向集群的流量，以了解性能。
- 负载分配：为每一种负载类型分配对应容量，并启用超出限定值的请求。
- 静态响应处理：在边缘位置直接建立部分相应，从而避免其转发到内部集群。



### @EnableZuulProxy
启动类。
```
spring.application.name: micro-weather-eureka-client-zuul

eureka.client.serviceUrl.defaultZone: http://localhost:8761/eureka/

zuul.routes.city.path: /city/**
zuul.routes.city.serviceId: msa-weather-city-eureka
```

把所有访问city的请求转发到msa-weather-city-eureka服务里去
这个服务就这么一个配置功能，别的为空。

### @FeignClient("msa-weather-eureka-client-zuul")
控制器类
另一个服务里
可以把一些client（service）合并成一个，用path路由来区分。如[@FeignClient](#feignclient) 这个cityclient，可以多个删掉合并
**改造前**
```java
@FeignClient("msa-weather-city-eureka")
public interface CityClient {	
	@GetMapping("/cities")
	List<City> listCity() throws Exception;
}
@FeignClient("msa-weather-data-eureka")
public interface WeatherDataClient {	
	@GetMapping("/weather/cityId/{cityId}")
	WeatherResponse getDataByCityId(@PathVariable("cityId") String cityId);
}
```

**改造后**
```java
@FeignClient("msa-weather-eureka-client-zuul")
public interface DataClient 
{
	/**
	 * 获取城市列表
	 * @return
	 * @throws Exception
	 */
	@GetMapping("/city/cities")
	List<City> listCity() throws Exception;	
	/**
	 * 根据城市ID查询天气数据
	 */
	@GetMapping("/data/weather/cityId/{cityId}")
	WeatherResponse getDataByCityId(@PathVariable("cityId") String cityId);
}

```


## kong




# 配置中心

## @EnableConfigServer

配置中心服务器

![](http://qiniu.xds123.cn/18-10-9/49986433.jpg)

*   远程Git仓库：存储配置文件。
*   ConfigServer：分布式配置管理中心，会于维护自己的git仓库信息。
*   本地Git仓库：在ConfigServer中，每次客户端请求获取配置信息时，都会从git仓库获取最新的配置到本地，然后本地读取并返回，远程无法获取时，使用本地仓库信息。

从上图可以看出，`Config Server`巧妙地通过`git clone`将配置信息存于本地，起到了缓存的作用，即使当`Git`服务端无法访问的时候，依然可以取`Config Server`中的缓存内容进行使用。

1. 启动类加入@EnableConfigServer注解，声明是`ConfigServer`。

```java
@SpringBootApplication
@EnableConfigServer
@Slf4j
public class SpringCloudConfigServerApplication 
{
   public static void main(String[] args) throws Exception 
   {
     SpringApplication.run(SpringCloudConfigServerApplication.class, args);
     log.info("spring-cloud-config-server启动!");
   }
}
```

2. 配置文件，添加git仓库相关信息。

建一个repo，里面放一个文件my-config-client-dev.properties


```java
spring.application.name=spring-cloud-config-server
server.port=5678

#配置文件git配置
spring.cloud.config.server.git.uri=https://github.com/xie19900123/spring-cloud-learning.git
# 搜索路径，即配置文件的目录，可配置多个，逗号分隔。默认为根目录。
spring.cloud.config.server.git.searchPaths=spring-cloud-config-repo
# git用户名和密码 针对私有仓库而言需要填写
spring.cloud.config.server.git.username=
spring.cloud.config.server.git.password=
```

3. 启动应用，访问[http://127.0.0.1:5678/my-config-client-dev.properties](http://127.0.0.1:5678/my-config-client-dev.properties) ，返回了配置文件的信息，说明已经读取到远程仓库信息了。

/{application}/{profile}[/{label}]
/{application}-{profile}.yml
/{label}/{application}-{profile}.yml
/{application}-{profile}.properties
/{label}/{application}-{profile}.properties


url会映射{application}-{profile}.properties对应的配置文件，
其中{label}对应Git上不同的分支，默认为master。我们可以尝试构造不同的url来访问不同的配置内容，
比如：
要访问`master`分支，`my-config-client`应用的dev环境

## Client端

创建一个客户端:`spring-cloud-confg-client`。当然也可以改造原来的应用了，只需加入相应pom文件和配置文件即可。
1. 创建启动类，就是一个正常的web应用。
```java
@SpringBootApplication
@Slf4j
public class SpringCloudConfigClientApplication 
{
   public static void main(String[] args) throws Exception 
   {
     SpringApplication.run(SpringCloudConfigClientApplication.class, args);
     log.info("spring-cloud-config-client启动!");
   }
}
```

2. 配置文件添加：`bootstrap.properties`和常规的`application.properties`。

`bootstrap.properties`

```
# 设置分支
spring.cloud.config.label=master
# 环境变量
spring.cloud.config.profile=dev
# 是否使用注册中心方式进行获取 后续会进行讲解
#spring.cloud.config.discovery.enabled=false
# 服务端地址 
# 在不使用注册中心模式下 直接填写实际地址
spring.cloud.config.uri=http://127.0.0.1:5678
# 注册中心应用id 下一章节会进行讲解
#spring.cloud.config.discovery.service-id=
```

`application.properties`

```
# 设置应用名称，需要和配置文件匹配
spring.application.name=my-config-client
server.port=5666
```

`spring-cloud-config`相关的属性**必须配置在`bootstrap.properties`中**，config部分内容才能被正确加载。因为config的相关配置会先于`application.properties`，而`bootstrap.properties`的加载也是先于`application.properties`。

3. 编写一个控制层，利用`@Value`进行参数测试。

```java
@RestController
public class DemoController 
{
   @Value("${config}")
   String config;
   @GetMapping("/")
   public String demo() 
   {
       return "返回的config参数值为:" + config;
   }
}
```



# 熔断

而在微服务调用中，自身异常可自行处理外，对于依赖的服务若发生错误，或者调用异常，或者调用时间过长等原因时，避免长时间等待，造成系统资源耗尽。
一般上都会通过`设置请求的超时时间`，如`http`请求中的`ConnectTimeout`和`ReadTimeout`；再或者就是使用`熔断器`模式，隔离问题服务，防止级联错误等。


熔断器，和现实生活中的`空气开关`作用很像。它可以实现**快速失败**，如果它在一段时间内侦测到许多类似的错误，**会强迫其以后的多个调用快速失败，不再访问远程服务器，从而防止应用程序不断地尝试执行可能会失败的操作**，使得应用程序继续执行而不用等待修正错误，或者浪费CPU时间去等到长时间的超时产生。**熔断器也可以使应用程序能够诊断错误是否已经修正，如果已经修正，应用程序会再次尝试调用操作。**

熔断器模式就像是那些容易导致错误的操作的一种代理。**这种代理能够记录最近调用发生错误的次数，然后决定使用允许操作继续，或者立即返回错误**。

 [](http://qiniu.xds123.cn/18-9-22/18901300.jpg "熔断器状态转换图")


可以看出，熔断器一共有三种状态，之间转换关系如下：

*   **关闭状态**
    当熔断器处于关闭状态时，请求是可以被放行的；
    当熔断器统计的失败次数触发开关时，转为打开状态。
*   **打开状态**
    当熔断器处于打开状态时，所有请求都是不被放行的，直接返回失败；
    只有在经过一个设定的时间窗口周期后，熔断器才会转换到半开状态
*   **半开状态**
    当熔断器处于半开状态时，当前只能有一个请求被放行；
    这个被放行的请求获得远端服务的响应后，假如是成功的，熔断器转换为关闭状态，否则转换到打开状态。


## Hystrix

Hystrix是有Netflix开源的一个延迟和容错库，用于隔离访问远程系统、服务或第三方库，防止级联失败，从而提升系统的可用性和容错性。

**Hystrix容错机制：**

*   **包裹请求**：使用HystrixCommand包裹对依赖的调用逻辑，每个命令在独立线程中执行，这是用到了设计模式“命令模式”。
*   **跳闸机制**：当某服务的错误率超过一定阈值时，Hystrix可以自动或手动跳闸，停止请求该服务一段时间。
*   **资源隔离**：Hystrix为每个依赖都维护了一个小型的线程池，如果该线程池已满，发往该依赖的请求就被立即拒绝，而不是排队等候，从而加速判定失败。
*   **监控**：Hystrix可以近乎实时的监控运行指标和配置的变化。如成功、失败、超时、被拒绝的请求等。
*   **回退机制**：当请求失败、超时、被拒绝，或当断路器打开时，执行回退逻辑。回退逻辑可自定义。
*   **自我修复**：断路器打开一段时间后，会自动进入半开状态，断路器打开、关闭、半开的逻辑转换。


## @EnableHystrix

启动类

## @HystrixCommand

```java
@RestController
@Slf4j
public class RibbonController 
{
	 @Autowired
	 RestTemplate restTemplate;
	 @GetMapping("/ribbon")
	 @HystrixCommand(fallbackMethod="fallback")
	 public String hello(String name) 
	 {
		 log.info("使用restTemplate调用服务，参数name:{}", name);
		 return restTemplate.getForObject("http://eureka-client/hello?name=" + name, String.class);
	 }
	 /**
	 * 发生熔断时调用的方法
	 * @param name
	 * @param throwable 发生异常时的异常信息
	 * @return
	 */
	 public String fallback(String name,Throwable throwable) 
	 {
		 log.error("熔断发生了：{}", throwable);
		 log.warn("restTemplate调用服务发生熔断，参数name:{}", name);
		 return "restTemplate调用服务发生熔断，参数name：" + name;
	 }
}
```


##  Feign整合Hystrix

如上小节说示例的，当我们方法很多时，要是分别编写一个`fallback`估计也是崩溃的，虽然可以使用一个通用的`fallback`，但未进行特殊设置下，也是无法知道具体是哪个方法发生熔断的。

而对于`Feign`，我们可以使用一种更加优雅的形式进行。我们可以指定`@FeignClient`注解的`fallback`属性，或者是`fallbackFactory`属性，后者可以获取异常信息的。


1. 启动类，加入`@EnableFeignClients`启用`Feign`.

```java
**
 * 熔断器示例
 * @author oKong
 *
 */
@SpringBootApplication
@EnableHystrix
@EnableDiscoveryClient
@EnableFeignClients
@Slf4j
public class HystrixApplication 
{
 public static void main(String[] args) throws Exception 
 {
   SpringApplication.run(HystrixApplication.class, args);
   log.info("sprign-cloud-hystrix启动!");
 }
 @Bean
 @LoadBalanced
 public RestTemplate restTemplat() 
 {
   return new RestTemplate();
 }
}
```

2. 创建一个服务接口类`IHelloClient.java`，同时定义`fallback`或者`fallbackFactory`属性值。注意：两者 同时设置时，优先调用`fallback`，`fallbackFactory`不进行调用了。

```java
@FeignClient(name="eureka-client",/*fallback=HelloClientFailImpl.class,*/ fallbackFactory = HelloClientFallbackFactory.class)
public interface IHelloClient 
{
 /**
 * 定义接口
 * @param name
 * @return
 */
   @RequestMapping(value="/hello", method=RequestMethod.GET)
   public String hello(@RequestParam("name") String name);
}
```

3.  创建`fallback`和`fallbackFactory`属性对应类。

```java
@Component("fallback")
@Slf4j
public class HelloClientFailImpl implements IHelloClient
{
 @Override
 public String hello(String name) 
 {
   log.error("restTemplate调用[hello]服务发生熔断，参数name:{}", name);
   return "restTemplate调用[hello]服务发生熔断，参数name:" + name;
 }
}
@Component
@Slf4j
public class HelloClientFallbackFactory implements FallbackFactory<IHelloClient>
{
   @Autowired
   @Qualifier("fallback")
   IHelloClient helloClient;
   @Override
   public IHelloClient create(Throwable cause) 
   {
     log.error("feign调用发生异常，触发熔断", cause);
     return helloClient;
   }  
}
```


可以知道，正常`fallback`就是一个接口的实现类，当发送异常时，会调用此接口实现类进行服务调用。而`FallbackFactory`是也是一个接口实现类，需要实现`feign.hystrix.FallbackFactory<T>`接口，在发生熔断时，调用`create`方法，同时返回被调用接口的实现类，以便进行fallback处理。

3.配置文件开启feign的熔断器功能。
```
feign.hystrix.enabled=true
```



