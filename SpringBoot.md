# Spring Boot



## 常用注解 

@SpringBootApplication

Spring Boot应用标注在某个类上说明这个类是

Spring Boot 的主配置类，Spring Boot就应该运行这个类的main方法来启动Spring Boot应用。该注解由多个注解组成：

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
public @interface SpringBootApplication {...
```

@SpringBootConfiguration：SpringBoot的配置类：标注在某个注解上表示这是一个SpringBoot的配置类；

​	配置类：代替配置文件

@EnableAutoConfiguration：开启自动配置

@AutoConfigurationPackage：

@ConfigurationProperties:默认从全局配置文件中获取值，将配置文件中的每一个属性的值映射到这个组件中

```yaml
server:
  port: 8081

person:
  lastName: snow
  age: 18
  isboss: false
  birth: 2001/12/13
  maps: {k1: v1,k2: v2}
  lists:
    -zhangsan
    - lisi
  dog:
    name: hach
    age: 5
    
```

```java
@ConfigurationProperties(prefix = "person")
/**该注解将配置文件中配置的每一个属性值，都映射到这个组件中(必须是容器中的组件)*/
@Component
public class Person{
    ...
} 

```

```xml
<!--导入配置文件处理器-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
</dependency>
```

@value("") 相当于<bean class.. >中的value

@PropertySource：加载指定的配置文件

配置文件位置及优先级由高到低排序如下，高优先级会覆盖低优先级：

- file: ./config/
- file: ./
- classpath:/config
- classpath:/

SpringBoot加载jar包外的配置文件(外部配置文件的优先级大于)

```java
yourproj-1.0.0-SNAPSHOT.jar
application.properties

java -jar yourproj-1.0.0-SNAPSHOT.jar
```

优先加载顺序：

- 优先加载带{profile}的(application-profile.yml/properties)
- 相同格式优先加载jar包外的

[Application.yml所有配置属性参照](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/appendix-application-properties.html#common-application-properties)

自动配置原理：

1. SpringBoot启动的时候加载主配置类，开启自动配置功能(@EnableAutoConfiguration)
   - @EnableAutoConfiguration 的作用
     - 利用EnableAutoConfigurationImportSelector给容器中导入一些组件



## 自动配置原理

[配置文件能配置的属性参照](https://docs.spring.io/spring-boot/docs/2.4.0-SNAPSHOT/reference/html/appendix-application-properties.html#common-application-properties)

自动配置类是Spring Boot的精髓

1. springBoot启动会加载大量的自动配置类
2. 要看一下需要的功能有没有在SpringBoot默认写好的自动配置类
3. 再来看这个自动配置类配置了哪些组件（如果已有，则不需要再配置）
4. 给容器中自动配置类添加组件的时候，会出properties类中获取某些属性，我们就可以在配置文件中指定这些属性的值





使用`@SpringBootApplication`s是一个组合注解，主要由**`@EnableAutoConfiguration`**和`@SpringBootConfiguration`构成



`@EnableAutoConfiguration`中又包含

- `@AutoConfigurationPackage`
- `@Import({AutoConfigurationImportSelector.class})`
  - 利用`AutoConfigurationImportSelector`类给容器中导入一些组件，该类有一个方法

`AutoConfigurationImportSelector`类会给容器中导入一些组件，该类有一个方法：

```java
public String[] selectImports(AnnotationMetadata annotationMetadata)
```

会使用到

```java
List<String> configurations = this.getCandidateConfigurations(annotationMetadata, attributes); //获取候选的配置
```

其中又有

```java
List<String> configurations = SpringFactoriesLoader.loadFactoryNames(this.getSpringFactoriesLoaderFactoryClass(), this.getBeanClassLoader()); 
//扫描所有所有jar包类路径下的 META-INF/spring.factories，把扫描到的这些文件的内容包装成properties对象，从此对象中获取到EnableAutoConfiguration.class类（类名）对应的值，然后把他们添加在容器中
```



总结：将(springboot-autoconfigura.jar)类路径下META-INF/spring.factory里面配置的所有EnableAutoConfiguration的值加入到了容器中；容器中最终会有如下类，每一给类都是一个组件，用它们来做自动配置。

```properties
# Initializers
org.springframework.context.ApplicationContextInitializer=\
org.springframework.boot.autoconfigure.SharedMetadataReaderFactoryContextInitializer,\
org.springframework.boot.autoconfigure.logging.ConditionEvaluationReportLoggingListener

# Application Listeners
org.springframework.context.ApplicationListener=\
org.springframework.boot.autoconfigure.BackgroundPreinitializer

# Auto Configuration Import Listeners
org.springframework.boot.autoconfigure.AutoConfigurationImportListener=\
org.springframework.boot.autoconfigure.condition.ConditionEvaluationReportAutoConfigurationImportListener

# Auto Configuration Import Filters
org.springframework.boot.autoconfigure.AutoConfigurationImportFilter=\
org.springframework.boot.autoconfigure.condition.OnBeanCondition,\
org.springframework.boot.autoconfigure.condition.OnClassCondition,\
org.springframework.boot.autoconfigure.condition.OnWebApplicationCondition

# Auto Configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
...
```



每一个自动配置类进行自动配置功能。以`HttpEncodingConfiguration`为例解释自动配置原理

```java
//表示这是一个配置类
@Configuration(proxyBeanMethods = false) 
//启用指定类的ConfigurationProperties功能，（该类见下文）将配置文件中对应的值和HttpProperties绑定，并将其加入IOC
@EnableConfigurationProperties({HttpProperties.class})
//spring底层的@Conditional注解，如果满足配置文件，整个配置类里面的配置才会生效
@ConditionalOnWebApplication(type = Type.SERVLET)
@ConditionalOnClass({CharacterEncodingFilter.class})
//判断配置文件中是否存在spring.http.encoding.enabled配置，matchIfMissing缺少该配置也可以加载。
@ConditionalOnProperty(prefix = "spring.http.encoding",
    value = {"enabled"},matchIfMissing = true)
public class HttpEncodingAutoConfiguration {
    //已经和SpringBoot的配置文件映射了
        private final Encoding properties;
	//只有一个有参构造函数的情况下，参数的值会从容器中取
    public HttpEncodingAutoConfiguration(HttpProperties properties) {
        this.properties = properties.getEncoding();
    }

    @Bean	//给容器中添加一个组件，这个组件的也需要从properties中取内容
    @ConditionalOnMissingBean
    public CharacterEncodingFilter characterEncodingFilter() {
        CharacterEncodingFilter filter = new OrderedCharacterEncodingFilter();
        filter.setEncoding(this.properties.getCharset().name());
        filter.setForceRequestEncoding(this.properties.shouldForce(org.springframework.boot.autoconfigure.http.HttpProperties.Encoding.Type.REQUEST));
        filter.setForceResponseEncoding(this.properties.shouldForce(org.springframework.boot.autoconfigure.http.HttpProperties.Encoding.Type.RESPONSE));
        return filter;
    }
    ...
}
```



所有配置文件中能配置的属性都是在xxxProperties类中封装，配置文件能配置什么就可以参照某个功能对应的这个属性类

```java
//从配置文件中获取指定的值和bean 的属性进行绑定
@ConfigurationProperties(prefix = "spring.http")
public class HttpProperties {...}
```





总结：

`@SpringBootApplication`包含`@EnableAutoConfiguration`，后者又包含`@Import({AutoConfigurationImportSelector.class})`，导入的`AutoConfigurationImportSelector.class`类会去加载类路径下META-INF/spring.factory里面配置的所有配置项，每一个配置项都对应着一个类。

##过滤器、监听器、拦截器

Spring的拦截器与Servlet的Filter有相似之处，二者都是AOP编程思想的体现，都能实现权限检查、日志记录等，不同的是：

- 使用范围不同，Filter是Servlet规范规定的，只能用于Web程序中，而拦截器既可以用于Web程序，又可以用于Application、Swing程序中
- 规范不同：Filter实在Servlet规范中定义的，是Servlet容器支持的，而拦截器是在Spring容器内的，是Spring框架支持的。
- 使用的资源不同：拦截器归Spring管理，配置在S陪你过文件中，因此能使用Spring里的任何资源、对象，通过IOC注入到拦截器即可，而Fliter不能
- 深度不同：Filter只在Servlet前后起作用，而拦截器则能够深入到方法前后异常抛出前后灯，因此拦截器的使用具有更大的弹性。所以在Spring架构的程序中，要优先使用拦截器。