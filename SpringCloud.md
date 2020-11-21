# 微服务

**微服务要解决的四大问题：**

- 客户端如何访问多个服务
- 服务间如何通信
- 多个服务如何管理
- 服务崩溃怎么办

**CAP原则**

一个分布式系统中，[一致性](https://baike.baidu.com/item/一致性/9840083)（Consistency）、[可用性](https://baike.baidu.com/item/可用性/109628)（Availability）、分区容错性（Partition tolerance）。CAP 原则指的是，这三个要素最多只能同时实现两点，不可能三者兼顾。

# Spring cloud和Spring boot之间的依赖关系

- 去Spring cloud官网最后查看
- 网址输入`start.spring.io/actuator/info`，其中的Json串中有详细信息

Spring Cloud 小版本分为:

- SNAPSHOT： 快照版本，随时可能修改

- M： MileStone，M1表示第1个里程碑版本，一般同时标注PRE，表示预览版版。

- SR： Service Release，SR1表示第1个正式版本，一般同时标注GA：(GenerallyAvailable),表示稳定版本。

## Spring Cloud Alibaba

### 组件

- **Sentinel：** 把流量作为切入点，从流量控制、熔断降级、系统负载保护等多个维度保护服务的稳定性。
- **Nacos：** 一个更易于构建云原生应用的动态服务发现、配置管理和服务管理平台。
- **RocketMQ：** 一款开源的分布式消息系统，基于高可用分布式集群技术，提供低延时的、高可靠的消息发布与订阅服务。
- **Dubbo：** Apache Dubbo™ 是一款高性能 Java RPC 框架。
- **Seata：** 阿里巴巴开源产品，一个易于使用的高性能微服务分布式事务解决方案。
- **Alibaba Cloud ACM：** 一款在分布式架构环境中对应用配置进行集中管理和推送的应用配置中心产品。
- **Alibaba Cloud OSS：** 阿里云对象存储服务（Object Storage Service，简称 OSS），是阿里云提供的海量、安全、低成本、高可靠的云存储服务。您可以在任何应用、任何时间、任何地点存储和访问任意类型的数据。
- **Alibaba Cloud SchedulerX：** 阿里中间件团队开发的一款分布式任务调度产品，提供秒级、精准、高可靠、高可用的定时（基于 Cron 表达式）任务调度服务。
- **Alibaba Cloud SMS：** 覆盖全球的短信服务，友好、高效、智能的互联化通讯能力，帮助企业迅速搭建客户触达通道。

### Eureka是一个服务注册中心

```yaml  
spring:
  application:
    name: hello-spring-cloud-eureka      //服务的名称

server:
  port: 8761      //使用的端口

eureka:
  instance:
    hostname: localhost      //主机位置
    lease-expiration-duration-in-seconds=90		//（服务续约，定义服务失效时间，默认为90秒）
    lease-renewal-interval-in-seconds=30		//（定义服务续约的间隔时间，默认为30秒。）
  client:
    registerWithEureka: false      //是否向Eureka注册
    fetchRegistry: false      //是否去检索其他的服务
    serviceUrl:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
  server
    enable-self-preservation=false		//（确保注册中心中不可用的实例被及时的剔除）
```
注册中心之间也互相注册为服务，当服务提供者发送请求到一个服务注册中心时，它会将该请求转发给集群中相连的其他注册中心，从而实现注册中心之间的服务同步，通过服务同步，两个服务提供者的服务信息我们就可以通过任意一台注册中心来获取到。服务提供者会维护一个心跳来不停的告诉Eureka Server：“我还在运行”，以防止Eureka Server将该服务实例从服务列表中剔除，这个动作称之为服务续约。超时未续约的服务将会被剔除。

```@EnableDiscoveryClient```注解，表示该应用是一个Eureka客户端应用，这样该应用就自动具备了发现服务的能力  

----

### 负载均衡
- 服务端负载均衡
  - 硬件负载均衡：服务器节点之间安装专业的负载均衡设备
  - 软件负载均衡：服务器上安装有负载均衡功能的软件，如Nginx
- 客户端负载均衡
>Ribbon和Eureka一起使用时，Ribbon会从Eureka注册中心去获取服务端列表，然后进行轮询访问以到达负载均衡的作用  
```@LoadBalanced```注解，表示开启客户端负载均衡
Spring Cloud中采取的默认负载均衡策略就是轮询
-----

Feign是一个声明式的伪 Http 客户端，Feign 支持可插拔的编码器和解码器。Feign 默认集成了 Ribbon，并和 Eureka 结合，默认实现了负载均衡的效果
Ribbon是一个基于HTTP和TCP的客户端负载均衡器，当我们将Ribbon和Eureka一起使用时，Ribbon会从Eureka注册中心去获取服务端列表，然后进行轮询访问以到达负载均衡的作用，服务端是否在线这些问题则交由Eureka去维护。

#### RestTemplate

RestTemplate是Spring提供的用于访问Rest服务的客户端，简化了与http服务的通信，并满足RestFul原则,能够大大提高客户端的编写效率，可替换HttpClient。  
首先要理解HttpEntity  

该类的入口主要是根据HTTP的六个方法制定  
HTTP method | RestTemplate method
-|-  
DELETE | delete
GET | getForObject/getForEntity
POST | postForLocation/postForObject
PUT | put
HEAD | headForHeaders
OPTIONS | optionsForAllow
通用 | exchange/execute  

getForObject()多包含了将HTTP转成POJO的功能;  
getForObject()没有处理response的能力,因为它获取的数据省略了很多response的信息。
```java  
restTemplate.getForObject(URL,返回类型)
restTemplate.postForEntity(URL,要传的参数,返回类型)
```
RestTemplate默认使用HttpMessageConverter实例将HTTP消息转换成POJO或者从POJO转换成HTTP消息

-----
hystrix 熔断器

```@EnableCircuitBreaker```在入口类上通过此注解开启断路器功能
在service方法注解中写熔断方法
@HystrixCommand(fallbackMethod = "error")

-----

feign
```@EnableFeignClients``` 表示开启Spring Cloud Feign的支持功能



# Spring Cloud Alibaba

元数据：描述数据的数据

# 尚硅谷Spring cloud

## 建工程

`new project -> Maven -> maven-archety-site `

粘贴pom.xml文件

新建Moudule

每个模块：

1. 建module
2. 改pom
3. 写yml
4. 主启动
5. 业务类

## 写业务

1. 建表
2. 建立实体类
3. 建立统一的返回接口
4. 建立Dao
5. 写mapper.xml映射
6. 其他和SSM类似

## 使用Eureka

1. 引入pom文件

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

2. 启动类中使用`@EnableEurekaServer`注解
3. application.yml

```yaml
eureka:
  instance:
    hostname: eureka7001.com #eureka服务端的实例名称
  client:
    # false表示不向注册中心注册自己
    register-with-eureka: false
    # false表示自己端就是注册中心,我的职责就是维护服务实例,并不需要检索服务
    fetch-registry: false
    service-url:
      # 设置与Eureka Server交互的地址查询服务和注册服务都需要依赖这个地址
      # 单机 defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
      # 相互注册
      #defaultZone: http://eureka7002.com:7002/eureka/
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
```

## 其他服务向Erueka注册

1. 添加pom依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

2. 在启动类中使用`@EnableEurekaClient` 
3. 在application.yml中配置向Eureka注册

```yaml
eureka:
  client:
    #表示是否将自己注册进EurekaServer默认为true
    register-with-eureka: true
    #是否从EurekaServer抓取已有的注册消息，默认为true，单节点无所谓，集群必须设置为true才能配合ribbon使用负载均衡
    fetch-registry: true
    service-url:
      #集群版
      #defaultZone: http://localhost:7001/eureka/,http://eureka7002.com:7002/eureka/
      #单机版
      defaultZone: http://localhost:7001/eureka/
```

## Eureka集群

每个Eureka Server都要向除自身以外的其它Server注册，此行为在application.yml中反映如下：

```yaml 
server:
  port: 7001

eureka:
  instance:
    hostname: eureka7001.com #eureka服务端的实例名称
  client:
    # false表示不向注册中心注册自己
    register-with-eureka: false
    # false表示自己端就是注册中心,我的职责就是维护服务实例,并不需要检索服务
    fetch-registry: false
    service-url:
      # 设置与Eureka Server交互的地址查询服务和注册服务都需要依赖这个地址
      # 相互注册，多个可以以逗号分隔
      defaultZone: http://eureka7002.com:7002/eureka/
```

##通过Eureka访问服务

1. 将服务提供者集群都注册到Eureka
2. 在服务消费者中将要调用的服务URL写为`http://PROVIDER-SERVICE` provider-service 和服务提供者在Eureka中注册的名字对应。
3. 在服务消费者的`ApplicationContextConfig.java`中使用`@LoadBalanced`注解来赋予`RestTemplate`负载均衡的能力。默认使用轮询的负载均衡机制

```yaml
eureka 
  instance:
    instance-id: payment8001
    prefer-ip-address: true #访问路径可以显示ip
```



## Eureka自我保护机制

如果在Eureka首页看到这段提示，则说明Eureka进入了保护模式

>EMERGENCY EUREKA MAY BE INCORRECTLY CLAMING INSTANCES ARE UP WHEN THEY'RE NOT...

默认情况下，如果Eureka Sever在一定时间(默认90s)内没有接收到某个微服务实例的心跳，Eureka会注销该实例

一旦进入保护模式，某一时刻由于网络拥堵等原因，某个微服务不可用了，Eureka 不会立刻清理，依旧会对该微服务信息进行保存

**怎么禁止自我保护**

`eureka.server.enable-self-preservation: false` 该选项默认为true

`eureka.server.eviction-interval-timer-in-ms: 2000` 设置断开连接时间，单位毫秒

```yaml
eureka
  instance:
    #Eureka客户端向服务端发送心跳的实际间隔，单位为秒（默认为30秒）
    lease-renewal-interval-in-seconds: 1
    #Eureka服务端收到最后一次心跳后等待时间上线，单位为秒（默认为90秒） 超时将剔除服务
    lease-expiration-duration-in-seconds: 2
```



## Consul

Consul 是一套分布式**服务发现**和**配置管理**系统，具有服务发现、健康监测、KV存储、多数据中心、可视化web页面。

Consul 服务端是一个单独部署的软件，不需要在微服务中使用代码写出来。

和Eureka类似，将服务消费者和服务提供者注册到Consul中，需要在pom中引入如下依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-consul-discovery</artifactId>
</dependency>
```

application.yml 

```yaml 
server:
  port: 8006
spring:
  application:
    name: consul-provider-payment

  cloud:
    consul:
      host: 47.92.2.46
      port: 8500
      discovery:
        service-name: ${spring.application.name}

```

## Ribbon

Ribbon主要功能是提供**客户端**软件**负载均衡**算法和**服务调用**

Ribbon需要集成在服务消费者中。

在新版中Ribbon已经集成在Eureka client中，只要引入了以下依赖，就会引入Ribbon

```xml 
<dependency>
     <groupId>org.springframework.cloud</groupId>
     <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

负载均衡的两种实现方式：

- 集中式LB(服务端)
  - Nginx：客户端所有请求都会交给Nginx，Nginx直接转发请求
- 进程内LB(本地)
  - Ribbon： 本地负载均衡，调用微服务接口时，会在注册中心获取注册信息服务列表后缓存到JVM本地，Ribbon再根据规则去选择连接这些机器

### Ribbon 中的负载均衡算法

Ribbon中有一个核心组件IRule，带有7种负载均衡规则，默认方式为轮询。

修改默认负载均衡算法：

1. 自定义一个配置类（可以在主启动类所在的包的上一级建一个包）

> 该配置类不能放在@ComponentScan所扫描的当前包下以及子包下，否则配置类会被所有Ribbon客户端所共享

2. 在包中新建自己的规则类

```java
@Configuration
public class PersonalRule {
    @Bean
    public IRule myRule(){
        return new RandomRule();
    }
}
```

3. 在主启动上添加`@RibbonClient` 

```j
@RibbonClient(name = "CLOUD-PAyMENT-SERVICE", configuration = PersonalRule.class)
```

## OpenFeign

Feign是一个声名式的web service客户端，使用Feign能让编写Web Service客户端更加简单，它的使用方法是定义一个服务接口然后在上面添加注解。

Feign集成了Ribbon

Feign用在消费端

###OpenFeign的使用

1. 引入新依赖

```xml
<dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

1. 主启动类中添加`@EnableFeignClients`
2. 新建业务接口，并新增注解`@FeignClient`和`@Component`

```java
@Component
@FeignClient(value = "CLOUD-PAYMENT-SERVICE") //要调用的在Eureka注册的服务
public interface PaymentFeignService {
    @GetMapping(value = "/payment/get/{id}")
   public CommonResult<Payment> getpaymentById(@PathVariable("id") Long id);
}
```

3. 在Controller中调用业务接口中的方法，以此来调用服务提供者

```java
@RestController
public class OrderFeignController {
    @Autowired
    private PaymentFeignService paymentFeignService;

    @GetMapping("/consumer/payment/get/{id}")
    public CommonResult<Payment> getPaymentById(@PathVariable("id") Long id){
        return paymentFeignService.getpaymentById(id);  //通过接口的调用来调用服务提供者
    }
}
```

### OpenFeign超时控制

Feign客户端默认只等待1秒钟，收不到服务提供者的返回将会直接报错

设置Feign客户端超时控制，在application.yml中通过ribbon设置

```yaml
ribbon: 
  ReadTimeOut: 5000   #指的是建立连接所用的时间，适用于网络状况正常的情况
  ConnectTimeOut: 5000	#指的是建立连接后从服务器读取到可用资源所用的时间
```

### Feign日志级别

在application.yml中配置

```yaml
logging:
  level:
    com.coldcoder.springcloud.service.PaymentFeignService: debug 
    #Feign日志以什么级别监控哪个接口
```

## Hystrix

Hystrix是一个用处理分布式系统的延迟和容错的开源库，它能保证在一个依赖出问题的情况下，不会导致整体服务失败，避免联级故障。

重要概念：

- 服务降级(fallback)：向调用方返回一个符合预期的、可处理的备选响应，而不是长时间的等待或者抛出异常
  - 导致服务降级的原因：异常、超时、服务熔断触发熔断降级
- 服务熔断(break)：
- 服务限流(flowlimit)：避免突发流量访问过多，有序访问

Hystrix在消费者端和提供者端都可以使用

### 服务降级

主要要解决以下三种问题：

1. 提供者超时，消费者不能一直卡死等待，必须有服务降级
2. 提供者宕机，消费者必须有服务降级
3. 调用者自身故障，自己处理降级

### Hystrix使用

#### 第一种方法

1. 引入新依赖

```xml
<dependency>
     <groupId>org.springframework.cloud</groupId>
     <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```

2. 在要进行服务降级保护的方法上使用`HystrixCommand(fallback="MethodName")` 注解

```java
@DefaultProperties(defalutFallback = "defalut_fallbackMethod") 
public class myController{
@HystrixCommand(fallbackMethod = "fallbackMethod",commandProperties = {
        @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds",value = "3000")  //超时设置，该线程3秒钟以内为正常，否则出错
    })		//其他Exception也会进行服务降级
public String mainMethod1(int id) {...}

@HystrixCommand  //未配置fallback使用设置的默认fallback方法
public String mainMethod2(int id){...}
    
public String fallbackMethod(int id){...}

public String defalut_fallbackMethod(){...}
}

```

####第二种方法(推荐)

配合Feign使用

1. 在application.yml中开启配置

```yaml 
feign:
  hystrix:
    enabled: true
```

2. 在主启动类中使用`@EnableFeignClients`和`@EnableHystrix` 注解
3. 定义Feign调用接口，并在`@FeignClient`注解中定义fallback方法

```java
@FeignClient(value = "MY-SERVICE", fallback = FallbackService.class)
public interface FeignService {
    ...
}
```

4. 新建统一fallback处理类并实现Feign调用接口    

```java
@Component
public class FallbackService implements FeignService {

    @Override
    ...
}
```

###服务熔断

熔断机制是应对雪崩效应的一种微服务链路保护机制，当检测出链路某个微服务不可用时，会进行降级进而熔断该节点的微服务调用，快速返回错误的相应信息。当检测到该节点微服务调用相应正常后，恢复调用链路。

```java
    // 服务熔断
    @HystrixCommand(fallbackMethod = "paymentCircuitBreaker_fallback",commandProperties = {
            @HystrixProperty(name = "circuitBreaker.enabled", value = "true"),              //是否开启断路器
            @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold", value = "10"),    //请求数达到后才计算
            @HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds", value = "10000"), //休眠时间窗
            @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage", value = "60"),  //错误率达到%多少跳闸
    })
```

## Gateway 

Gateway 是下Spring生态系统之上构建的API网关服务，基于Spring 5,Spring Boot 2和Project Reactor等技术

Gateway旨在提供一种简单而有效地方式来对API进行路由，以及提供一些强大的过滤器功能，如熔断、限流

、重试等

三大核心概念：

- Route（路由）
- Predicate（断言）
- Filter（过滤）

### 注册中心和Gateway配合使用

默认情况下Gateway会根据注册中心注册的服务列表，以注册中心上微服务名为路径创建动态路由进行转发，从而实现动态路由的功能

1. 创建
2. 修改aplication.yml

```yaml
spring:
  application:
    name: cloud-gateway
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true # 开启从注册中心动态创建路由的功能，利用微服务名称进行路由
      routes:
        - id: payment_route # 路由的id,没有规定规则但要求唯一,建议配合服务名
          #匹配后提供服务的路由地址,以lb://开头表示去服务注册中心找
          uri: lb://cloud-payment-service
          predicates:
            - Path=/payment/get/** # 断言，路径相匹配的进行路由
            #此处可自定义多种过滤条件
eureka:
  instance:
    hostname: cloud-gateway-service
  client:
    fetch-registry: true
    register-with-eureka: true
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/
```

### 使用Filter

- GatewayFilter：看官网有模板
- GlobalFilter：看官网有模板
- 自定义Filter

### Predicates和Filter的区别

Predicates(断言)：是路由转发的判断条件，当满足这些条件之后才会转发

filter(过滤)：转发之后进行过滤。分为pre类型过滤器和post类型过滤器，pre类型过滤器作用于访问路由前，可以进行鉴权，限流，日志输出，请求头的修改，post类型过滤器作用访问路由后，可以进行响应头协议的修改。



## Config 配置中心

SpringCloud Config 为微服务架构中的微服务提供集中化的外部配置支持，配置服务器为各个不同微服务应用的所有环境提供了一个中心化的外部配置。它可以

- 集中管理配置文件，可抽出公共配置
- 分环境部署，如dev/test/prod/beta
- 运行期间动态调整配置，服务会向配置中心统一拉取配置自己的信息
- 配置发生改变，无需重启

### Config Server配置中心的使用

1. 在github或其他代码托管平台建立一个仓库，存放配置文件。例如github下的config仓库
2. 新建一个服务，修改application.yml (此处使用https方法，使用ssh方法配置也可)

```yaml
server:
  port: 3344

spring:
  application:
    name: cloud-config-center
  cloud:
    config:
      server:
        git:
          uri: https://github.com/coldcoder126/config.git  
          #uri: Github上的git仓库的http地址
          search-paths: respo            
          ##搜索目录.这个目录指的是github上的目录
          username: 1348*****@qq.com
          password: ********
      ##读取分支
      label: master
eureka:
  client:
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/
```

访问资源要按照以下格式之一

>```
>/{application}/{profile}[/{label}]
>/{application}-{profile}.yml
>/{label}/{application}-{profile}.yml
>```

###Config Client的使用

1. 使用`bootstrap.yml`代替`application.yml` ，因为前者优先级更高

```yaml
server:
  port: 3355

spring:
  application:
    name: config-client
  cloud:
    config:  #Config客户端配置
      label: master #分支名称
      name: config #配置文件名称
      profile: dev #读取后缀名称 以上3个综合就是 master分支上 config-dev.yml
      uri: http://localhost:3344 #配置中心

eureka:
  client:
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/
```

###解决配置动态刷新的问题

1. pom文件引入actuator监控

```yaml
<dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

2. 修改yml，暴露监控端口

```yaml
management:
  endpoints:
    web:
      exposure:
        include: "*"
```

3. 在业务类Controller上添加`@RefreshScope` 
4. 修改配置之后，手动发送更新post到需要升级配置的服务`curl -X POST "http://localhost:3355/actuator/refresh"`

spring cloud bus 配合Spring Cloud Config可做到自动刷新

## Spring Cloud Bus消息总线

**什么是总线**

在微服务架构的系统中，通常会使用轻量级的消息代理来构建一个公用的消息主题，并让系统中所有的微服务实例都连接上来。由于该主题中产生的消息会被所有实例监听和消费，所以称它为消息总线。在总线上的各个实例都可以方便地广播一些需要让其他连接在该主题上的实例都知道的消息。

**基本原理**

ConfigClient实例都监听MQ中同一个topic(默认为springCloudBus)。当一个服务刷新数据的时候，它会把这个信息放入Topic中，这样其它监听同一Topic的服务就能得到通知，然后去更新自身的配置。

Bus支持两种消息代理：RabbitMQ和Kafka

**设计思想**

利用消息总线触发一个服务端ConfigServer的bus/refresh端点，而刷新所有客户端的配置

### SpringCloud Bus 动态刷新全局广播

1. 配置中心Server中添加依赖

```xml
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-bus-amqp</artifactId>
        </dependency>
```

2. application.yml中增加rabbitmq相关配置

```yaml 
spring: 
  rabbitmq:
    host: 47.92.2.46
    port: 5672
    username: guest
    password: guest
    
#rabbit相关配置，暴露bus刷新配置的端点
management:
  endpoints:
    web:
      exposure:
        include: 'bus-refresh'
```

3. ConfigClient客户端增加总线支持

```xml
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-bus-amqp</artifactId>
        </dependency>
```

4. ConfigClient配置文件

```yaml
spring: 
  rabbitmq:
    host: 47.92.2.46
    port: 8672
    username: guest
    password: guest
```

5. 在ConfigClient端增加`@RefreshScope` 注解
6. 向ConfigClient Server 发送更新POST请求`curl -X POST "http://localhost:3344/actuator/bus-refresh"`

###SpringCloud Bus动态刷新定点通知

指定具体某一部分实例生效而不是全部

`http://localhost:3344/actuator/bus-refresh/application-name:3355`

## SpringCloud Stream消息驱动

**是什么？**

官方定义Spring Cloud Stream是一个构建消息驱动微服务的框架

**解决了什么问题？**

屏蔽底层消息中间件的差异，降低切换成本，统一消息的编程模型

##Spring Cloud Sleuth分布式请求链路跟踪

提供了一套完整的服务跟踪解决方案，并兼容zipkin

到`https://dl.bintray.com/openzipkin/maven/io/zipkin/java/zipkin-server/`下载jar包并使用`java -jar `运行。

**zipkin和sleuth的关系**



1. 引入依赖

```xml
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-zipkin</artifactId>
        </dependency>
```

2. 配置

```yaml
spring: 
  zipkin:
    base-url: http://localhost:9411
  sleuth:
    sampler:
      probability: 1 #采样率值介于0到1之间，1代表全部采集
```

#Spring Cloud Alibaba

## Nacos 服务注册和配置中心

Nacos是一个注册中心和配置中心的组合，是一个更易于构建云原生应用的动态服务发现、配置管理和服务管理平台。

Nacos = Eureka +Config + Bus

Nacos支持AP和CP的切换

### Provider使用Nacos做服务注册中心

1. 到nacos的github官网下载Nacos-Server并解压运行，访问链接为`http://hostname:8848/nacos`
2. 在父pom中引入依赖

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-alibaba-dependencies</artifactId>
            <version>2.1.0.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

3. 新建项目并增加依赖

```xml
<dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
```

4. 修改配置文件

```yaml
server:
  port: 9001
spring:
  application:
    name: nacos-payment-provider
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848

management:
  endpoints:
    web:
      exposure:
        include: '*'
```

5. 主启动类使用`@SpringBootApplication` 和 `@EnableDiscoveryClient`

###Consumer使用Nacos做服务注册中心

其他和Provider配置相同

1. 增加配置类，使用RestTemplate

```java
@Configuration
public class ApplicationContextConfig {
    @LoadBalanced
    @Bean
    public RestTemplate getRestTemplate(){
        return new RestTemplate();
    }
}
```

2. 在配置文件中配置`service-url`，使代码和配置分离

3. 调用

```java
@RestController
public class OrderNacosController {
    @Resource
    private RestTemplate restTemplate;
    
    @Value("${service-url.nacos-user-service}")
    private String serverURL;

    @GetMapping("/consumer/payment/nacos/{id}")
    public String paymentInfo(@PathVariable("id") int id){
        return restTemplate.getForObject(serverURL+"/payment/nacos/"+id,String.class);
    }
}
```

###Nacos做配置中心

####基础配置

1. 引入依赖

```xml
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
```

2. 修改配置文件

```
server:
  port: 3377

spring:
  application:
    name: nacos-config-client
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848 #服务注册中心地址
      config:
        server-addr: localhost:8848 #Nacos作为配置中心地址
        file-extension: yml #指定yml格式的配置
  profiles:
    active: dev
```

3. 在`Nacos->配置管理->配置列表`中新建一个配置文件，其中`Data ID`的完整格式如下：(该处与配置文件对应)

```yaml
${spring.application.name}-${spring.profile.active}.${file-extension}
#     nacos-config-client - dev . yml
```

4. 不要忘记在`controller`中加`@RefreshScope` 注解

####分类配置

Nacos访问使用`NameSpace + Group + Data ID ` 类似于java中的包名和类名

最外层的namespace是可以用于区分部署环境的，Group和Data ID逻辑上区分两个目标对象

`NameSpace` 主要用于实现隔离，不同的namespace之间是隔离的，默认为`public `

`Group`可以把不同的微服务划分到一个group里，默认为`DEGAULT_ROUP`

`Service` 就是微服务，一个Service可以包含多个`Cluster`（集群）

如果不使用默认的namespace和group，则需要在配置文件中配置

```yaml
spring: 
  cloud: 
    nacos: 
      config: 
        group: YOUR_GROUP
        namespace: NAMESPACE_ID
```

###Nacos集群

默认Nacos使用嵌入式数据库实现数据的存储。所以，如果启动多个默认配置下的Nacos节点，数据存储是存在一致性问题的。为了解决这个问题，Nacos采用了集中式存储的方式来支持集群化部署，目前只支持MySQL

> Nginx -> 多个 Nacos -> 共用一个MySQL(集群)

**使用方法：**

1. 在MySQL中新建数据库`nacos_config`，并执行该[sql脚本](https://github.com/alibaba/nacos/blob/master/distribution/conf/nacos-mysql.sql)

2. 在`nacos/conf/application.properties`中新增如下配置信息

```properties
spring.datasource.platform=mysql

db.num=1
db.url.0=jdbc:mysql://yourhost:3306/nacos_config?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true
db.user=root
db.password=123456
```

3. Linux服务器上nacos的集群配置`nacos/conf/cluster.conf`，其中内容为各个集群的`ip:port` ,端口为8848可以不写。通过此配置文件让nacos知道这些是一个集群。

4. 配置Nginx转发到多台服务器的Nacos
5. 配置微服务，注册地址为Nginx，Nginx再将其转发到Nacos

#### 解决Nacos默认不支持mysql 8.0+ 版本的问题

> 在nacos安装目录下新建`plugins/mysql` 目录，将`mysql-connector-java-8.xx.jar`包放入其中



## Sentinel

Hystrix 的阿里升级版

Sentinel 以流量为切入点，从流量控制、熔断降级、系统负载保护等多个维度保护服务的稳定性。

Sentinel只对访问控制的规则进行监控管理，而无法处理运行时异常

Sentinel 分为两个部分:

- 核心库（Java 客户端）不依赖任何框架/库，能够运行于所有 Java 运行时环境，同时对 Dubbo / Spring Cloud 等框架也有较好的支持。
- 控制台（Dashboard）基于 Spring Boot 开发，打包后可以直接运行，不需要额外的 Tomcat 等应用容器。

### 使用

1. 下载`sentinel-dashboard.jar`然后`java -jar`运行即可，注意会占用8080端口

2. 新建模块，引入以下依赖

```xml
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba.csp</groupId>
            <artifactId>sentinel-datasource-nacos</artifactId>
            <version>1.7.1</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
        </dependency>
```

3. 配置文件中修改配置

```yaml
spring: 
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848
    sentinel:
      transport:
        dashboard: localhost:8080  #配置sentinel地址
```

4. 启动nacos， sentinel和服务，进入sentinel查看服务监控

   > 服务需被访问过才会出现在sentinel监控界面

### 图形化界面操作

#### 基本概念

QPS：每秒查询率（Queries-per-second）

RT：响应时间

####流控模式

- 直接（默认）：API达到限流条件时，直接限流
- 关联：当关联的资源达到阈值时，就限流自己。（例如：当支付接口压力过大，可以限制订单接口）
- 链路：只记录指定链路上的流量

#### 流控效果

- 快速失败（默认）：超出阈值直接失败调用默认失败页面
- WarmUp：预热，阈值除以coldFactor(默认值为3)，经过预热时长后才会达到阈值，而不是瞬间达到峰值
- 排队等待：匀速排队，让请求以均匀的速度通过，阈值必须设置为QPS否则无效

#### 熔断降级

*Sentinel的熔断没有半开状态*

Sentinel熔断降级会在调用链路中某个资源出现不稳定状态时，对这个资源的调用进行限制，让请求快速失败，避免影响到其他资源而出现级联错误。

**降级策略**

- RT：需满足1) 1s内持续进入5个请求，2)平均响应时间都超过设置的阈值（RT，单位:ms），那么在接下来的时间窗口中，对这个方法的调用都会自动熔断
- 异常比例：需满足1)资源的每秒请求量大于5个，2）每秒异常占通过量的比值超过阈值(单位：%)；那么在接下来的时间窗口中，对这个方法的调用都会自动熔断
- 异常数：当资源近一分钟的异常数目超过阈值之后，会进行熔断。*异常数是按分钟统计的*

#### 热点限流

热点：即经常访问的数据，对某一个资源数据的访问限制

**使用**

资源名：`@SentinelResource(value = "唯一资源名")`

参数索引：针对该资源名所在方法的第几个参数，从零开始

单机阈值：QPS的阈值

参数例外项：在上面规则的基础上添加例外条件

- 参数类型：
- 参数值： 
- 限流阈值：

使用`@SentinelResource` 注解

```java
@SentinelResource(value="唯一资源名",blockHandler="兜底方法名")
```

####系统规则

系统保护规则是从应用级别的入口流量进行控制，入口流量指的是进入应用的流量。系统规则支持以下的模式：

- Load自适应：按系统的资源使用率等
- RT：
- CPU使用率
- 

阈值类型：

#### 处理代码和业务代码分离

1. 业务代码

```java
	@GetMapping("/myservice")
    @SentinelResource(value = "myservice",
                      blockHandlerClass = CustomerHandler.class, //规则错误处理的类
                      blockHandler = "handlerException",
                      fallbackClass = myFallback.class,
                      fallback = "fallbackMethod") //规则错误处理的方法
    public CommonResult myService(){
        return new CommonResult(200,"successful service");
    }
```

2. 定义全局的规则异常(BlockException)处理类和处理方法

```java
public class CustomerHandler {
    public static CommonResult handlerException(BlockException exception){ //要有BlockException参数
        return new CommonResult(444,"自定义global返回处理");
    }
}
```

3. 定义运行时异常处理类和处理方法

```java
public class myFallback {
    public static String fallbackMethod(){
        return "fallback method";
    }
}
```

### Sentinel整合ribbon+OpenFeign+fallback

1. 在原有的依赖基础上再引入OpenFeign

```xml
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
```

2. 修改配置文件

```yaml
feign:
  sentinel:
    enable: true  #激活Sentinel对Feign的支持
```

3. 主启动类加上`@EnableFeignClient`
4. 

### 规则持久化

服务重启Sentinel规则会消失。

####将限流配置规则持久化到Nacos保存

1. 添加依赖

```xml
        <dependency>
            <groupId>com.alibaba.csp</groupId>
            <artifactId>sentinel-datasource-nacos</artifactId>
        </dependency>
```

2. 配置文件

```yaml
spring:
  cloud:
    sentinel:
      datasource: 
        ds1:
          nacos:  
            server-addr: localhost:8848
            dataId: ${spring.application.name}
            groupId: DEFAULT_GROUP
            data-type: json
            rule-type: flow
```

3. 在Nacos中配置以json格式配置sentinel对应的规则文件

```json
[{
    "resource": "/testA",     资源名称
    "limitApp": "default",    来源应用
    "grade": 1,               阈值类型：0线程数，1QPS
    "count": 1,				  单击阈值
    "strategy": 0,			  流控模式：0直接，1关联，2链路
    "controlBehavior": 0, 	  流控效果：0快速失败，1WarmUp，2排队
    "clusterMode": false	  是否集群
}]
```

4. 只要服务上线，规则就会存在

## Seata处理分布式事务

在分布式之后，一次业务操作需要跨多个数据源或需要跨多个系统进行远程调用，就会产生分布式问题。

seata用来保证全局数据的一致性

**Seata术语**

**TC - 事务协调者**

维护全局和分支事务的状态，驱动全局事务提交或回滚。

**TM - 事务管理器**

定义全局事务的范围：开始全局事务、提交或回滚全局事务。

**RM - 资源管理器**

管理分支事务处理的资源，与TC交谈以注册分支事务和报告分支事务的状态，并驱动分支事务提交或回滚。







- [ ] feign 的fallback和sentinel的fallback





















---



###热部署Devtools

**电脑配置高了再用**

1. 添加依赖`spring-boot-devtools`到项目中

2. 添加一个插件`spring-boot-maven-plugin` 到父类总工程里 

   1. ```xm
          <build>
              <plugins>
                  <plugin>
                      <groupId>org.springframework.boot</groupId>
                      <artifactId>spring-boot-maven-plugin</artifactId>
                      <configuration>
                          <fork>true</fork>
                          <addResources>true</addResources>
                      </configuration>
                  </plugin>
              </plugins>
          </build>
     ```

3. 开启自动编译选项

###一个服务调用另一个服务使用RestTemplate

RestTemplate是Spring封装好的Http请求工具，服务消费者可以使用它来调用服务提供者

### 重构：将重复的代码提取到commons模块中





搜索 尚硅谷周阳

# Spring Security oAuth2

##概念

###什么是 oAuth

oAuth 协议为用户资源的授权提供了一个安全的、开放而又简易的标准。与以往的授权方式不同之处是 oAuth 的授权不会使第三方触及到用户的帐号信息（如用户名与密码），即**第三方无需使用用户的用户名与密码就可以申请获得该用户资源的授权**，因此 oAuth 是安全的。

###什么是 Spring Security

Spring Security 是一个安全框架，前身是 Acegi Security，能够为 Spring 企业应用系统提供声明式的安全访问控制。Spring Security 基于 Servlet 过滤器、IoC 和 AOP，为 Web 请求和方法调用提供身份确认和授权处理，避免了代码耦合，减少了大量重复代码工作。

### RBAC模型

RBAC（Role-Based Access Control，基于角色的访问控制），就是用户通过角色与权限进行关联。简单地说，一个用户拥有若干角色，每一个角色拥有若干权限。这样，就构造成“用户-角色-权限”的授权模型。在这种模型中，用户与角色之间，角色与权限之间，一般是多对多的关系。由此引申出必要的5张表

### Access Toke 和 Refresh Token

`Access Token` ：访问资源的token，用户带着这个token可以访问对应权限的资源

但是Access token用在客户端，用户请求带着此令牌可能会被泄漏，造成安全隐患。因此Access Token设计的过期时间较短，而一旦失效，用户需要重新登录才能再次获取Access Token，比较麻烦。于是出现了Refresh Token

`Refresh Token`：用来刷新Access Token的token，当访问令牌过期时，客户端使用刷新令牌来交换访问令牌。这允许客户端继续拥有有效的访问令牌，而无需与用户进一步交互。

### OAuth 2.0的授权码模式

以A网站申请B网站的授权为例

**第一步：**A网站提供一个链接，用户点击后，跳转到B网站提供的页面。链接内容格式如下所示：

```http
https://b.com/oauth/authorize?
  response_type=code&				
  client_id=CLIENT_ID&
  redirect_uri=CALLBACK_URL&
  scope=read
```

其中`response_type`参数表示要求返回授权码（`code`），`client_id`参数让 B 知道是谁在请求，`redirect_uri`参数是 B 接受或拒绝请求后的跳转网址，`scope`参数表示要求的授权范围

**第二步：**在这个页面中用户需要填写账号密码登录B网站，登录后会询问用户是否需要授权访问自己的信息，若用户同意，则重定向到`redirect_uri`，并返回给客户端一个授权码`code`，返回内容如下：

```http
https://a.com/callback?code=AUTHORIZATION_CODE
```

**第三步：**A客户端拿到`code`后，带着`code`向A服务器请求A的信息，此时A服务器获取到`code`，由A服务器向B服务器请求token令牌，请求格式如下

```http
https://b.com/oauth/token?
 client_id=CLIENT_ID&
 client_secret=CLIENT_SECRET&
 grant_type=authorization_code&
 code=AUTHORIZATION_CODE&
 redirect_uri=CALLBACK_URL
```

其中`client_id`参数和`client_secret`参数用来让 B 确认 A 的身份（`client_secret`参数是保密的，因此只能在后端发请求），`grant_type`参数的值是`AUTHORIZATION_CODE`，表示采用的授权方式是授权码，`code`参数是上一步拿到的授权码，`redirect_uri`参数是令牌颁发后的回调网址

**第四步：**B 网站收到请求以后，就会颁发令牌。具体做法是向`redirect_uri`指定的网址，发送一段 JSON 数据

```json
{    
  "access_token":"ACCESS_TOKEN",
  "token_type":"bearer",
  "expires_in":2592000,
  "refresh_token":"REFRESH_TOKEN",
  "scope":"read",
  "uid":100101,
  "info":{...}
}
```

## 使用Spring Security

1. 新建模块`oauth-server`，并引入依赖。该模块主要负责鉴权（即登录）

```xml
    <dependencies>
        <!-- Spring Boot -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
        </dependency>

        <!-- Spring Security -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-oauth2</artifactId>
        </dependency>

    </dependencies>
```

认证授权服务登录，查出用户的权限后，

```java
@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true,securedEnabled = true, jsr250Enabled = true)
public class WebSecurityConfiguration extends WebSecurityConfigurerAdapter {

    @Bean
    public BCryptPasswordEncoder passwordEncoder(){
        return new BCryptPasswordEncoder();
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userDetailsService());
    }
}
```





```java
@Service
public class UserDetailServiceImpl implements UserDetailsService {
    @Autowired
    private TbUserService tbUserService;
    @Autowired
    private TbPermissionService permissionService;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        tbUser tbUser = tbUserService.getByUsername(username);
        List<GrantedAuthority> grantedAuthorits = Lists.newArrayList();
        if(tbUser != null){
            List<TbPermission> permissions = permissionService.selectByUserId(tbUser.getId());  //登录成功则查出权限列表
            permissions.forEach(tbPermission -> {
                GrantedAuthority authority = new SimpleGrantedAuthority(tbPermission.getEnname);
                grantedAuthorits.add(authority);    //权限集合放入list
            });
        }
        return new User(tbUser.getName(),tbUser.getPassword(),grantedAuthorits);    //返回的User是security中的User
    }
}
```



资源服务和oauth服务对接，在需要授权才能访问的服务中创建全局方法拦截器，用户带着`access_token`来访问资源服务，资源服务通过此token判断用户是否有相应权限。

创建

```java
@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true,securedEnabled = true, jsr250Enabled = true)
public class ResourceServerConfiguration extends ResourceServerConfigurerAdapter {
    @Override
    public void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .antMatchers("/").hasAuthority("SystemContent")
                .antMatchers("/view/**").hasAuthority("SystemView");
    }
}
```

application.yml中增加如下必要模块

```yaml
security:
  oauth2:
    client:
      client-id: client		#该资源服务的id
      client-secret: secret	
      access-token-uri: http://localhost:8080/oauth/token	#请求token的地址
      user-authorization-uri: http://localhost:8080/oauth/authorize	#对用户授权的地址
    resource:
      token-info-uri: http://localhost:8080/oauth/check_token 	#检查token是否有效的地址
```



## 关于Authentication认证的一些概念

**SecurityContextHolder： **  Spring Security存储被认证用户细节的地方

**SecurityContext：** 从`SecurityContextHolder`获取并包含当前已验证用户的`Authentication`

**Grantication：**授予主体身份验证（即角色、作用域等）的权限

**Authentication：** 可以是`AuthenticationManager`的输入，来提供 用户已提供的用于验证的凭据或`SecurityContext`中的当前用户的凭据。

**ProviderManager：**AuthenticationManager的最常见实现。

**AuthenticationProvider：**由ProviderManager用于执行特定类型的身份验证。