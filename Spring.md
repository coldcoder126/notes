# Spring 

Spring的配置

## 通过xml文件配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:jdbc="http://www.springframework.org/schema/jdbc"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/context
                           http://www.springframework.org/schema/context/spring-context.xsd
                           http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/tx
                           http://www.springframework.org/schema/tx/spring-tx.xsd
                           http://www.springframework.org/schema/aop
                           http://www.springframework.org/schema/aop/spring-aop.xsd">

<!--开启组件扫描
	base-package:此包下所有的类都会扫描
 	annotation-config="true" :是否激活属性注入注解
	user-default-filter="false" :是否使用默认的过滤器，默认值true，设置为false之后可自定义过滤规则
	 <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Component"/> ：排除要扫描的注解，user-default-filter="true"
	context:include 只扫描配置的注解，user-default-filter="false"

-->
    <context:component-scan base-package="cn.coldcoder" annotation-config="true"/>
    
    
    <!--开启AspectJ生成代理对象：扫描并为有@AspectJ的类生成代理对象-->
    <aop:aspectj-autoproxy/>
    
    <!--创建事务管理器-->
    <bean id="txManager" class="org.springframework.jdbc.data.source.DataSourceTransactionManager">
    <!--注入数据源-->
        <property name="dataSource" ref="数据源bean"/>
    </bean>
    
    <!--开启事务注解-->
    <tx:annotation-driven transaction-manager="txManager"/>
    
    <!--使用xml方式配置通知-->
    <tx:advice id="txadvice">
        <!--配置事务参数-->
    	<tx:attributes>
            <!--指定在哪些方法是添加事务-->
        	<tx:method name="accountMoney" proagetion="REQUIRED"/>
        </tx:attributes>
    </tx:advice>

    <!--导入mybatis配置文件-->
    <import resource="applicationContext-datasource.xml"/>


</beans>
```

## 通过配置类实现

```java
@Configuration
@ComponentScan(basePackages = "cn.coldcoer")	//配置自动扫描
//@Import({ConfA.class,ConfB.class})	//导入其他的配置类
@EnableAspectJAutoProxy(proxyTargetClass = true)  //配置AOP
@EnableTransactionManager	//开启事务
public class SpringConfig {
    //创建数据库连接池
    @Bean
    public DruidDataSource getDruidDataSource(){
    DruidDataSource dataSource = new DruidDataSource();
    dataSource.setDriverClassName("com.mysql.jdbc.Driver");
    dataSource.setUrl("jdbc:mysql://user_db");
    dataSource.setUsername("xxx");
    dataSource.setPassword("xxxxxx");
    return dataSource;}
    
    //创建事务管理器
    @Bean
    public DataSourceTransactionManager getDSTxManager(DataSource dataSource){
        DataSourceTransactionManager txManager = new DataSourceTransactionManager();
        txManager.setDataSource(dataSource);
        return txManager;
    }
    
}


```





BeanFactory与ApplicationContext

 [`BeanFactory`](https://docs.spring.io/spring-framework/docs/5.1.3.BUILD-SNAPSHOT/javadoc-api/org/springframework/beans/factory/BeanFactory.html) 接口提供了一种更先进的配置机制来管理任意类型的对象. [`ApplicationContext`](https://docs.spring.io/spring-framework/docs/5.1.3.BUILD-SNAPSHOT/javadoc-api/org/springframework/context/ApplicationContext.html) 是`BeanFactory`的子接口。

`BeanFactory`提供了配置框架的基本功能，`ApplicationContext`添加了更多特定于企业的功能。，`ApplicationContext`完全扩展了`BeanFactory`的功能。





Bean

使用id或name属性标识。

@Configuration 和@Bean

`@Configuration`是一个类级别的注解,表明该类将作为bean定义的元数据配置.使用`@Configuration`注解类时，这个类的目的就是作为bean定义的地方。

 `@Bean`注解扮演的角色与`<beans/>`元素相同。开发者可以在任意的Spring `@Component`中使用`@Bean`注解方法 ，但大多数情况下，`@Bean`是配合`@Configuration`使用的。只需使用`@Bean`注解方法即可。使用此方法，将会在`ApplicationContext`内注册一个bean，bean的类型是方法的返回值类型。默认情况下， bean的id默认就是方法的名字。

```java
@Configuration
public class AppConfig {

    @Bean
    public MyService myService() {
        return new MyServiceImpl();
    }
}
```



```java
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
    MyService myService = ctx.getBean(MyService.class);
    myService.doStuff();
}
```

#IoC

控制反转，把对象创建和对象之间的调用过程，交给Spring进行管理

IOC控制反转 :通过反射实现

IoC又称为依赖注入（DI：Dependency Injection），它解决了一个最主要的问题：将组件的创建+配置与组件的使用相分离，并且，由IoC容器负责管理组件的生命周期。

IoC容器要负责实例化所有的组件，因此，有必要告诉容器如何创建组件，以及各组件的依赖关系。

**创建组件(Bean)的方法**

1. 通过XML文件配置

   ```xml
   <bean id="userService" class="com.itranswarp.learnjava.service.UserService">
           <property name="mailService" ref="mailService" />
       </bean>
   ```

2. 通过注解

   1. 先使用`@Component`、`@Service`、`@`、
   2. 配置自动扫描
   3. 在需要的地方使用`@Autowired`注入

3. 通过

**取出组件(Bean)的方法**

1. 通过`ApplicationContext`

   ```java
   ApplicationContext context = new ClassPathXmlApplicationContext("application.xml");
   UserService userService = context.getBean(UserService.class);
   ```

2. 通过`BeanFactory`

   ```java
   BeanFactory factory = new XmlBeanFactory(new ClassPathResource("application.xml"));
   MailService mailService = factory.getBean(MailService.class);
   ```

   二者区别：

   > `BeanFactory`的实现是按需创建，即第一次获取Bean时才创建这个Bean，而`ApplicationContext`会一次性创建所有的Bean。实际上，`ApplicationContext`接口是从`BeanFactory`接口继承而来的，并且，`ApplicationContext`提供了一些额外的功能，包括国际化支持、事件和通知机制等。通常情况下，我们总是使用`ApplicationContext`，很少会考虑使用`BeanFactory`。




**底层原理**

IOC容器的底层就是工厂对象，Spring提供IOC容器实现通过两个接口：

1. `BeanFactory`：提供了配置框架和基本的功能，使用的时候才创建对象
2. `ApplicationContext`：`BeanFactory`的子接口，添加了更多企业级具体的功能，加载配置文件时就会创建对象(推荐使用)

BeanDefinition、BeanDefinitionReader、BeanFactory

AOP：动态代理实现

xml解析、工厂模式、反射

#AOP

面向切面编程

[理解概念](https://www.cnblogs.com/hongwz/p/5764917.html)

增强/通知：在方法执行前或执行后

连接点：类里面哪些方法可以被增强，这些方法称为连接点

切入点：实际被增强的方法，称为切入点

切面：把通知应用到切入点的过程 

通知（增强）：实际增强的逻辑部分。分为前置通知、后置通知、环绕通知、异常通知、最终通知

- Aspect：切面，即一个横跨多个核心逻辑的功能，或者称之为系统关注点；
- Joinpoint：连接点，即定义在应用程序流程的何处插入切面的执行；
- Pointcut：切入点，即一组连接点的集合；
- Advice：增强，指特定连接点上执行的动作；
- Introduction：引介，指为一个已有的Java对象动态地增加新的接口；
- Weaving：织入，指将切面整合到程序的执行流程中；
- Interceptor：拦截器，是一种实现增强的方式；
- Target Object：目标对象，即真正执行业务的核心逻辑对象；
- AOP Proxy：AOP代理，是客户端持有的增强后的对象引用。

##AOP的实现

Spring框架一般都是基于AspectJ实现AOP操作

AspectJ：它不是Spring组成部分，是一个独立的AOP框架，一般把AspectJ和Spring框架一起使用，进行AOP操作

1. 引入AOP相关依赖

2. 使用切入点表达式：让程序知道对哪个类中的哪个方法进行增强

   1. 语法结构：`execution([权限修饰符](返回类型)[类全路径][方法名称][参数列表])`

      > `execution(* com.coldcoder.dao.userDao.add(..))`对userDao类中的add方法进行增强
      >
      > `execution(* com.coldcoder.*.*(..))`对com.coldcoder包中的所有类的所有方法进行增强

3. 编写被增强的类A，和增强的类B

###通过注解实现

1. 在Spring配置中开启注解扫描

2. 在被增强类A和增强类B上添加注解`@Component`

3. 在增强类B上面添加注解`@AspectJ`

4. 在Spring配置文件中开启生成代理对象（见配置）

5. 配置不同类型的通知：在增强的类B中，作为通知的方法上添加通知类型，并使用切入点表达式配置。示例：

   1. 作为前置通知：`@Before("execution(* com.coldcoder.*.*.(..))")`

   2. 后置通知`@After`、异常通知`@AfterThrowing`、返回通知

   3. 环绕通知`@Around` 标注的方法中有一个参数

      ```java
      @Around(value="execute...")
      public void around(ProceedingJionPoint pjp){
          //before
          pjp.proceed(); //调用增强的方法
          //after
      }
      ```

   4. 通知顺序：`@Around(before)`->`@Before`->Method->`@Around(after)`->`@After`->`@AfterReturning`

   5. `@After`和`@AfterReturning`的区别：

      @After有没有异常都会执行

      @AfterReturning有异常不会执行

6. 多个增强类对同一个方法增强，通过`@Order(i)`设置优先级，i值越小优先级越高

**抽取相同切入点**

使用`@PointCut`可以抽取相同的切入点，其它通知使用的时候可以直接引用此方法。示例：

```java
@PointCut(value="execution...")
public void pointCut(){...}
@Before(value="pointCut()")
public void before(){...}
```

### 通过XML实现

1. 在spring配置文件中创建A和B两个类对象

   ```xml
   <bean id="a" class="..."></bean>
   <bean id="b" class="..."></bean>
   
   <aop:config>
   	<aop:pointcut id="point" expression="execution..."/>  <!--配置切入点-->
       <aop:aspect ref="b">  <!--配置切面-->
           <aop:before method="methodName" pointcut-ref="point"/> <!--配置通知-->
       </aop:aspect>
   </aop:config>
   ```

   

2. 在spring配置文件中配置切入点

**底层原理**



AOP使用动态代理实现，（了解代理模式）代理方法调用实际方法，可以在执行实际方法之前或之后添加其他方法。可以实现不修改源码而增加新功能。

1. 有接口的情况下，通过JDK动态代理

   创建接口的实现类代理对象，调用真实对象的方法并增强

2. 没有接口的情况下， 通过CGLIB动态代理

   创建当前类的子类代理对象，在其中调用当前父类方法并增强

# Bean

##Bean管理

Bean管理指的是两个操作：

1. Spring创建对象
2. Spring注入属性
   1. 使用setter方法
   2. 使用有参构造注入

###基于XML的方式

```xml
<!--1.使用setter()方法注入；Book类中有name属性以及对应Setter方法-->
<bean id="Book" class="全限定名">
	<property name="name" value="book-value" />
    <property name="" ref="other_bean_id"/>  <!--注入外部bean->
</bean>

<!--2.使用构造方法注入；Book类中有有参构造方法Book(String name)-->
<bean id="Book" class="全限定名">
	<constructor-arg name="name" value="book-value" />
</bean>

<!--属性值中包含特殊符号处理方法：
1. 把<>进行转义为&lt;和&gt;
2.将带特殊符号的内存写到CDATA-->
```

FactoryBean：定义的类型和返回的类型可以是不一样的。

普通Bean：配置文件中定义的类型就是返回的类型

**工厂Bean**

1. 创建一个类实现`FactoryBean`接口

```java
public MyBean implements FactoryBean{
    //定义返回Bean 类型
    @Override
    public Object getObject() throws Exception{ return null;}
    
    @Override
    public Class<?> getObjectType(){ return null;}
    //是否是单例
    @Override
    public boolean isSingleton(){ return false;}
    
}
```

**Bean的作用域**

在spring中可以设置创建bean实例是单实例(默认)还是多实例

可在bean标签中使用`scope`来设置是单实例还是多实例。scope常用取值

- singleton：默认，单实例。加载配置文件的时候就会完成创建
- protorype：多实例。使用的时候才创建。
- request：
- session：

**Bean的生命周期**

从Bean的创建到销毁的过程

1. 通过（默认使用）无参构造来创建bean实例
2. 为bean的属性设置值和堆其他bean的引用（调用setter方法）
3. 执行Bean后置处理器中的`postProcessBeforeInitialization`
4. 调用bean的初始化方法（需要进行配置）
5. 执行Bean后置处理器中的`postProcessAfterInitialization`
6. bean可以使用了
7. 当容器关闭的时候，调用bean的销毁方法（需要进行配置）

**Bean自动装配**

使用XML方式

```xml
<bean id="emp" class="..." autowired="byName/byType"/>
<!--byName:根据属性的名称注入，注入bean的id值和类属性名称一样
	byType:根据属性类型注入（相同类型的被注入bean不能有多个）-->
```



### 基于注解的方式

1. 引入AOP依赖
2. 先在要成为bean的类中使用`@Component`、`@Service`、`@Controller`、`@Repository`
1. @Bean标注在方法上(返回某个实例的方法)，等价于spring的xml配置文件中的< bean>，作用为：注册bean对象。
3. 在配置文件/配置类中开启组件扫描
4. 在需要的地方使用注解注入
   1. `@Autowired`：根据类型进行自动装配
   2. `@Qualifier`：根据名称进行注入，和`@AutoWired`一起使用。由于接口有多个实现，可以用此指定bean的名称。
   3. `@Resource`：根据类型或名称都可以进行注入。(是javax包中的不是Spring中的)
   4. `@value`：向属性注入普通数据类型的值。



Bean的生命周期

#Spring的事务

##基本概念

### 事务的特性（ACID）

- **原子性**（Atomicity）：事务是一个原子操作，由一系列动作组成。事务的原子性确保动作要么全部完成，要么完全不起作用。

- **一致性**（Consistency）：一致性指的是事务中所包含的操作不能违反数据库的一致性检查，数据在事务执行之前处于某个数据的一致性状态，那么在数据执行之后也要保持数据间的一致性状态。

  最常用的例子就是银行转账，A给B转账，转账之前和转账之后两个账户的总额是不变的。

- **隔离性**（Isolation）：事务的隔离性主要规定了各个事务之间相互影响的程度。隔离性主要面对数据的并发访问，当多个事务同时访问同一个数据时，不同的隔离级别决定了各个事务对数据访问的不同行为。一般事务的**隔离级别**有四种，从弱到强分别是

  - 读未提交 Read Uncommitted

    它允许另外一个事务可以看到这个事务未提交的数据。会产生脏读，不可重复读和幻读。

  - 读已提交 Read Committed

    保证一个事务修改的数据提交后才能被另外一个事务读取。可以避免脏读，可能会出现不可重复读和幻读

  - 可重复读 Repeatable Read

    保证了一个事务不能读取另一个事务未提交的数据，还保证了不可重复读。

  - 串行化 Serializable

    所有事务操作都必须依次进行，可以避免所有问题，是最安全的隔离级别，但效率低

- **持久性**（Durability）：一旦事务完成，事务的结果将被写到持久化存储器中。

## 事务操作

事务一般添加到Service层；底层使用AOP

Spring提供事务管理接口`PlantformTranscationManager`，代表事务管理器，这个接口针对不同的框架提供不同的实现类。 

以转账为例：一个Service中的转账方法，对数据库中两个用户的资金进行操作

```java
public void accountMoney(int money){
    userDao.reduce(user1);  //用户1减余额
    userDao.add(user2);		//用户2加余额
}
```

### 声明式事务管理

建立在AOP之上，本质是对方法前后进行拦截，在目标方法开始之前创建或加入一个事务，执行完目标方法之后更具执行的情况提交或者回滚。非侵入式，不足之处是声名式事务管理的粒度是方法级别。编程式可以到细化到代码块。

1. 在Spring配置文件中配置事务管理器（见配置文件）
2. 引入namespace，开启事务注解

#### 基于注解方式

在Service类（或其中的方法）上添加`@Transactional`注解

在`@Transactional`注解中可以配置事务相关参数

- propagation：事务传播行为，见下文
- isolation：事务隔离级别，见下文
- timeout：超时时间，默认为-1，设置时间以秒为单位
- readOnly：是否只读，默认为false，设为true后只能查询而不能修改
- rollbackFor：回滚，设置出现哪些异常会进行回滚
- noRollbackFor：不回滚，是指出现哪些异常不会回滚

####基于XML配置的方式

在



###编程式事务管理（不推荐）

##事务相关配置

### 事务的传播机制

事务的传播性一般在事务嵌套时候使用，比如在事务A里面调用了另外一个使用事务的方法，那么这俩个事务是各自作为独立的事务执行提交，还是内层的事务合并到外层的事务一块提交呢，这就是事务传播性要确定的问题。

常用的事务传播机制如下：

```java
/*
如果当前存在一个事务，则加入当前事务，如果不存在，则直接执行。
对于一些查询方法来说，PROPAGATION_SUPPORTS比较适合，如果当前方法被其他方法调用，而其他方法启动了一个事务，此行为可以保证当前方法加入当前事务，并洞察当前事务对数据资源所做的更新。其他传播方式则看不到更新，因为当前事务没有提交。
*/
PROPAGATION_SUPPORTS
    
/**-------------- 以下四个受外围事务影响---------------*/

/*
默认的事务传播行为，如果外围函数存在事务，则加入，如果不存在事务，用自己的事务。
*/
PROPAGATION_REQUIRED

/*
不管外围是否存在事务，都会使用自己的事务。如果外围存在事务，会将外围事务挂起。如果某个业务对象不想影响外层事务，可以选择PROPAGATION_REQUIRES_NEW。
*/
PROPAGATION_REQUIRES_NEW

/*
如果存在事务，则在当前事务的一个嵌套事务中执行。与PROPAGATION_REQUIRES_NEW类似，但与PROPAGATION_REQUIRES_NEW创建的事务与原有事务为同一档次，在新事务运行时原有事务被挂起；PROPAGATION_NESTED创建的事务寄生在外层事务，地位比外层事务低，嵌套事务执行时外层事务也处于活跃状态。
*/
PROPAGATION_NESTED

/*
PROPAGATION_MANDATORY强制要求当前存在一个事务，如果不存在，则抛出异常。
*/
PROPAGATION_MANDATORY


/**-------------- 以下两个不需要外围事务，或不受外围事务影响---------------*/

/*
不支持当前事务，而是在没有事务的情况下执行。如果当前存在事务，则将当前事务挂起，但要看PlatformTransactionManager的实现类是否支持事务的挂起。
即外围的事务管不了内围事务
*/
PROPAGATION_NOT_SUPPORTED

/*
永远不支持事务，如果存在事务，则抛出异常。
*/
PROPAGATION_NEVER
```

| 内部事务类型             | 外围方法是否开启事务 | 结果                                                         |
| ------------------------ | -------------------- | ------------------------------------------------------------ |
| Propagation.REQUIRED     | 是                   | 所有`Propagation.REQUIRED`修饰的内部方法和外围方法均属于同一事务，只要一个方法回滚，整个事务均回滚 |
| Propagation.REQUIRED     | 否                   | `Propagation.REQUIRED`修饰的内部方法会新开启自己的事务，且开启的事务相互独立，互不干扰。 |
| Propagation.REQUIRES_NEW | 是                   | `Propagation.REQUIRES_NEW`修饰的内部方法依然会单独开启独立事务，且与外部方法事务也独立，内部方法之间、内部方法和外部方法事务均相互独立，互不干扰 |
| Propagation.REQUIRES_NEW | 否                   | `Propagation.REQUIRES_NEW`修饰的内部方法会新开启自己的事务，且开启的事务相互独立，互不干扰 |
| Propagation.NESTED       | 是                   | `Propagation.NESTED`修饰的内部方法属于外部事务的子事务，外围主事务回滚，子事务一定回滚，而内部子事务可以单独回滚而不影响外围主事务和其他子事务 |
| Propagation.NESTED       | 否                   | `Propagation.NESTED`和`Propagation.REQUIRED`作用相同，修饰的内部方法都会新开启自己的事务，且开启的事务相互独立，互不干扰 |
|                          |                      |                                                              |
|                          |                      |                                                              |
|                          |                      |                                                              |
|                          |                      |                                                              |

###REQUIRED,REQUIRES_NEW,NESTED 异同

**NESTED 和 REQUIRED 修饰的内部方法都属于外围方法事务，如果外围方法抛出异常，这两种方法的事务都会被回滚。但是 REQUIRED 是加入外围方法事务，所以和外围事务同属于一个事务，一旦 REQUIRED 事务抛出异常被回滚，外围方法事务也将被回滚。而 NESTED 是外围方法的子事务，有单独的保存点，所以 NESTED 方法抛出异常被回滚，不会影响到外围方法的事务。**

**NESTED 和 REQUIRES_NEW 都可以做到内部方法事务回滚而不影响外围方法事务。但是因为 NESTED 是嵌套事务，所以外围方法回滚之后，作为外围方法事务的子事务也会被回滚。而 REQUIRES_NEW 是通过开启新的事务实现的，内部事务和外围事务是两个事务，外围事务回滚不会影响内部事务。**





### 事务的隔离级别

**脏读：**事务A正在访问数据，并对数据进行了修改，还未提交到数据库。此时事务B也访问了这个数据，事务B读到的数据就是脏数据

**不可重复读：**事务A正在第一次读数据，然后事务B过来修改了数据，事务A又读取了数据，此时事务A在一个事务内两次读到的数据时不一样的，因此成为是不可重复读。

**幻觉读：**指当事务不是独立执行时发生的一种现象，例如第一个事务对一个表中的数据进行了修改，这种修改涉及到表中的全部数据行。同时，第二个事务也修改这个表中的数据，这种修改是向表中插入一行新数据。那么，以后就会发生操作第一个事务的用户发现表中还有没有修改的数据行，就好象发生了幻觉一样。

通过设置事务隔离级别可以解决读问题

|                           | 脏读 | 不可重复读 | 幻读 |
| ------------------------- | ---- | ---------- | ---- |
| READ UNCOMMITTED 读未提交 | 有   | 有         | 有   |
| READ COMMITTED 读已提交   | 无   | 有         | 有   |
| REPEATABLE READ 可重复读  | 无   | 无         | 有   |
| SERIALIZABLE 串行化       | 无   | 无         | 无   |

# Spring5新特性

1. 支持`@Nullable`注解
   1. 该注解可以用在方法上面表示方法返回可以为空；用在属性值上面表示属性值可以为空；用在参数上面表示参数值可以为空。
2. 支持函数式风格