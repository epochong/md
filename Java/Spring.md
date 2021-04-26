# Spring，Spring MVC，Spring Boot 三者比较

这三者专注的领域不同，解决的问题也不一样，Spring MVC和Spring Boot都属于Spring，Spring MVC 是基于Spring的一个 MVC 框架，而Spring Boot 是基于Spring的一套快速开发整合包

总的来说，Spring 就像一个大家族，有众多衍生产品例如 Boot，Security，JPA等等。但他们的基础都是Spring 的 IOC 和 AOP，IOC提供了依赖注入的容器，而AOP解决了面向切面的编程，然后在此两者的基础上实现了其他衍生产品的高级功能；

Spring MVC是基于 Servlet 的一个 MVC 框架，主要解决 WEB 开发的问题，因为 Spring 的配置非常复杂，各种xml，properties处理起来比较繁琐。

SpringBoot，于是为了简化开发者的使用，Spring社区创造性地推出了Spring Boot，它遵循约定优于配置，极大降低了Spring使用门槛，但又不失Spring原本灵活强大的功能。

# SpringBoot

## 主要特性

> 使配置（自动配置），部署（内嵌Servlet容器），编码（注解），监控变简单

- 遵循“习惯优于配置”的原则，使用Spring Boot只需要很少的配置，大部分的时候我们直接使用默认的配置； 
- 内置了嵌入式的Tomcat、Jetty等Servlet容器，降低了对环境的要求，应用可以不用打包成War格式，而是可以直接以Jar格式运行，java -jar；。
- 提供了多个可选择的”starter”以简化Maven的依赖管理（也支持Gradle），让您可以按需加载需要的功能模块。提供了starter POM, 能够非常方便的进行包管理, 很大程度上减少了jar hell或者dependency hell；
- 尽可能地进行自动配置，减少了用户需要动手写的各种冗余配置项，Spring Boot提倡无XML配置文件的理念，只需要自动配置和Java Config，使用Spring Boot生成的应用完全不会生成任何配置代码与XML配置文件。
- 提供了一整套的对应用状态的监控与管理的功能模块（通过引入spring-boot-starter-actuator），包括应用的线程信息、内存信息、应用是否处于健康状态等，为了满足更多的资源监控需求，Spring Cloud中的很多模块还对其进行了扩展。

## 核心功能

（1）独立运行的Spring项目

- Spring Boot可以以jar包的形式进行独立的运行，使用：java -jar xx.jar 就可以成功的运行项目
- 或者在应用项目的主程序中运行main函数即可；

（2）内嵌的Servlet容器

内嵌容器，使得我们可以执行运行项目的主程序main函数，是想项目的快速运行；

主程序代码SpringbootDemoApplication.java

```java
package com.ceshi.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SpringbootDemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringbootDemoApplication.class, args);
    }
}
```

（3）提供starter简化Manen配置

Spring Boot提供了一系列的starter pom用来简化我们的Maven依赖

可参考官网，https://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/reference/htmlsingle/#using-boot-starter

（4）自动配置Spring

Spring Boot会根据我们项目中类路径的jar包/类，为jar包的类进行自动配置Bean，这样一来就大大的简化了我们的配置。当然，这只是Spring考虑到的大多数的使用场景，在一些特殊情况，我们还需要自定义自动配置；

（5）应用监控

Spring Boot提供了基于http、ssh、telnet对运行时的项目进行监控

（6）无代码生成和XML配置

Spring Boot神奇的地方不是借助于代码生成来实现的，而是通过条件注解的方式来实现的，这也是Spring 4.x的新特性。

## Starter原理（依赖引入，自动配置）

创建SpringBoot应用，我们在使用Web开发时，选择的是spring-boot-starter-web。starter是一种服务，使用某个功能的开发者不需要关注各种依赖库的处理，不需要具体的配置信息，由Spring Boot自动通过classpath路径下的类发现并加载需要的Bean。

### 原理

![SpringBoot Starter原理](http://www.51gjie.com/images/2019/d2yup1in.jyj.jpg)

利用starter实现自动化配置只需要两个条件——maven依赖、配置文件。引入maven实质上就是导入jar包，spring-boot启动的时候会找到starter jar包中的resources/META-INF/spring.factories文件，根据spring.factories文件中的配置，找到需要自动配置的类。

配置类上面包括以下注解

@Configuration 表明是一个配置文件，被注解的类将成为一个bean配置类

@ConditionalOnClass 当classpath下发现该类的情况下进行自动配置

@ConditionalOnBean 当classpath下发现该类的情况下进行自动配置

@EnableConfigurationProperties 使@ConfigurationProperties注解生效

@AutoConfigureAfter 完成自动配置后实例化这个bean

### 依赖引入

spring-boot-starter 核心Spring Boot starter，包括自动配置支持，日志和YAML

spring-boot-starter-actuator 生产准备的特性，用于帮我们监控和管理应用

spring-boot-starter-amqp 对”高级消息队列协议”的支持，通过spring-rabbit实现

spring-boot-starter-aop 对面向切面编程的支持，包括spring-aop和AspectJ

spring-boot-starter-batch 对Spring Batch的支持，包括HSQLDB数据库

spring-boot-starter-cloud-connectors 对Spring Cloud Connectors的支持，简化在云平台下（例如，Cloud Foundry 和Heroku）服务的连接

spring-boot-starter-data-elasticsearch 对Elasticsearch搜索和分析引擎的支持，包括spring-data-elasticsearch

spring-boot-starter-data-gemfire 对GemFire分布式数据存储的支持，包括spring-data-gemfire

spring-boot-starter-data-jpa 对”Java持久化API”的支持，包括spring-data-jpa，spring-orm和Hibernate

spring-boot-starter-data-mongodb 对MongoDB NOSQL数据库的支持，包括spring-data-mongodb

spring-boot-starter-data-rest 对通过REST暴露Spring Data仓库的支持，通过spring-data-rest-webmvc实现

spring-boot-starter-data-solr 对Apache Solr搜索平台的支持，包括spring-data-solr

spring-boot-starter-freemarker 对FreeMarker模板引擎的支持

spring-boot-starter-groovy-templates 对Groovy模板引擎的支持

spring-boot-starter-hateoas 对基于HATEOAS的RESTful服务的支持，通过spring-hateoas实现

spring-boot-starter-hornetq 对”Java消息服务API”的支持，通过HornetQ实现

spring-boot-starter-integration 对普通spring-integration模块的支持

spring-boot-starter-jdbc 对JDBC数据库的支持

spring-boot-starter-jersey 对Jersey RESTful Web服务框架的支持

spring-boot-starter-jta-atomikos 对JTA分布式事务的支持，通过Atomikos实现

spring-boot-starter-jta-bitronix 对JTA分布式事务的支持，通过Bitronix实现

spring-boot-starter-mail 对javax.mail的支持

spring-boot-starter-mobile 对spring-mobile的支持

spring-boot-starter-mustache 对Mustache模板引擎的支持

spring-boot-starter-redis 对REDIS键值数据存储的支持，包括spring-redis

spring-boot-starter-security 对spring-security的支持

spring-boot-starter-social-facebook 对spring-social-facebook的支持

spring-boot-starter-social-linkedin 对spring-social-linkedin的支持

spring-boot-starter-social-twitter 对spring-social-twitter的支持

spring-boot-starter-test 对常用测试依赖的支持，包括JUnit, Hamcrest和Mockito，还有spring-test模块

spring-boot-starter-thymeleaf 对Thymeleaf模板引擎的支持，包括和Spring的集成

spring-boot-starter-velocity 对Velocity模板引擎的支持

spring-boot-starter-web 对全栈web开发的支持， 包括Tomcat和spring-webmvc

spring-boot-starter-websocket 对WebSocket开发的支持

spring-boot-starter-ws 对Spring Web服务的支持

依赖spring-boot-starter-web，springboot会根据需要自动引入jar包。

## 自动配置

我们引入starter的依赖，会将自动配置的类的jar引入。@SpringBootApplication的注解中有一个是@EnableAutoConfiguration注解，这个注解有一个@Import({EnableAutoConfigurationImportSelector.class})，EnableAutoConfigurationImportSelector内部则是使用了SpringFactoriesLoader.loadFactoryNames方法进行扫描具有META-INF/spring.factories文件的jar包。而自动配置类的jar是有一个META-INF/spring.factories文件内容如下：

![img](http://www.51gjie.com/images/2019/hvjikj1l.syw.jpg)

应用在启动时就会加载spring-boot-autoconfigure的jar包下面的META-INF/spring.factories文件中定义的autoconfiguration类。将configuration类中定义的bean加入spring到容器中。就相当于加载之前我们自己配置组件的xml文件。而现在SpringBoot自己定义了一个默认的值，然后直接加载进入了Spring容器。

SpringBoot提供的自动配置依赖模块都以spring-boot-starter-为命名前缀，并且这些依赖都在org.springframework.boot下。 所有的spring-boot-starter都有约定俗成的默认配置，但允许调整这些配置调整默认的行为。

## 自动配置源码

不知道大家第一次搭SpringBoot环境的时候，有没有觉得非常简单。无须各种的配置文件，无须各种繁杂的pom坐标，一个main方法，就能run起来了。与其他框架整合也贼方便，使用`EnableXXXXX`注解就可以搞起来了！

所以今天来讲讲SpringBoot是如何实现自动配置的~

#### `@SpringBootApplication`

`@SpringBootApplication`等同于下面三个注解：

- `@SpringBootConfiguration`
- `@EnableAutoConfiguration`
- `@ComponentScan`

其中`@EnableAutoConfiguration`是关键(启用自动配置)，内部实际上就去加载`META-INF/spring.factories`文件的信息，然后筛选出以`EnableAutoConfiguration`为key的数据，加载到IOC容器中，实现自动配置功能！

![自动配置功能](https://segmentfault.com/img/remote/1460000018011548?w=1797&h=561)

在使用`main()`启动SpringBoot的时候，只有一个注解`@SpringBootApplication`

点击进去`@SpringBootApplication`注解中看看，可以发现有**三个注解**是比较重要的：

![SpringBootApplication注解详情](https://segmentfault.com/img/remote/1460000018011539?w=1292&h=304)

- `@SpringBootConfiguration`：我们点进去以后可以发现底层是**Configuration**注解，说白了就是支持**JavaConfig**的方式来进行配置(**使用Configuration配置类等同于XML文件**)。
- `@EnableAutoConfiguration`：开启**自动配置**功能(后文详解)
- `@ComponentScan`：这个注解，学过Spring的同学应该对它不会陌生，就是**扫描**注解，默认是扫描**当前类下**的package。将`@Controller/@Service/@Component/@Repository`等注解加载到IOC容器中。

所以，`Java3yApplication`类可以被我们当做是这样的：

```java
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan
public class Java3yApplication {
    public static void main(String[] args) {
        SpringApplication.run(Java3yApplication.class, args);
    }
}
```

#### @EnableAutoConfiguration

我们知道SpringBoot可以帮我们减少很多的配置，也肯定听过“约定大于配置”这么一句话，那SpringBoot是怎么做的呢？其实靠的就是`@EnableAutoConfiguration`注解。

简单来说，这个注解可以帮助我们**自动载入**应用程序所需要的所有**默认配置**。

介绍有一句说：

> if you have tomcat-embedded.jar on your classpath you are likely to want a TomcatServletWebServerFactory

如果你的类路径下有`tomcat-embedded.jar`包，那么你很可能就需要TomcatServletWebServerFactory

我们点进去看一下，发现有**两个**比较重要的注解：

![EnableAutoConfiguration注解详情](https://segmentfault.com/img/remote/1460000018011540?w=862&h=242)

- `@AutoConfigurationPackage`：自动配置包
- `@Import`：给IOC容器导入组件

1.2.1AutoConfigurationPackage

网上将这个`@AutoConfigurationPackage`注解解释成**自动配置包**，我们也看看`@AutoConfigurationPackage`里边有什么：

![AutoConfigurationPackage注解实现](https://segmentfault.com/img/remote/1460000018011541)

我们可以发现，依靠的还是`@Import`注解，再点进去查看，我们发现重要的就是以下的代码：

```java
@Override
public void registerBeanDefinitions(AnnotationMetadata metadata,
        BeanDefinitionRegistry registry) {
    register(registry, new PackageImport(metadata).getPackageName());
}
```

在**默认**的情况下就是将：主配置类(`@SpringBootApplication`)的所在包及其子包里边的组件扫描到Spring容器中。

- 看完这句话，会不会觉得，这不就是ComponentScan的功能吗？这俩不就重复了吗？

我开始也有这个疑问，直到我看到文档的这句话：

> it will be used when scanning for code @Entity classes.
> It is generally recommended that you place EnableAutoConfiguration (if you're
> not using @SpringBootApplication) in a root package so that all sub-packages
> and classes can be searched.

比如说，你用了Spring Data JPA，可能会在实体类上写`@Entity`注解。这个`@Entity`注解由`@AutoConfigurationPackage`扫描并加载，而我们平时开发用的`@Controller/@Service/@Component/@Repository`这些注解是由`ComponentScan`来扫描并加载的。

- 简单理解：这二者**扫描的对象是不一样**的。

1.2.2回到Import

我们回到`@Import(AutoConfigurationImportSelector.class)`这句代码上，再点进去`AutoConfigurationImportSelector.class`看看具体的实现是什么：

![得到很多配置信息](https://segmentfault.com/img/remote/1460000018011542?w=1687&h=652)

我们再进去看一下这些配置信息是从哪里来的(进去getCandidateConfigurations方法)：

![通过SpringFactoriesLoader来加载](https://segmentfault.com/img/remote/1460000018011543?w=1542&h=392)

这里包装了一层，我们看到的只是通过SpringFactoriesLoader来加载，还没看到关键信息，继续进去：

![跟踪实现](https://segmentfault.com/img/remote/1460000018011544)

简单梳理：

- `FACTORIES_RESOURCE_LOCATION`的值是`META-INF/spring.factories`
- Spring启动的时候会扫描所有jar路径下的`META-INF/spring.factories`，将其文件包装成Properties对象
- 从Properties对象获取到key值为`EnableAutoConfiguration`的数据，然后添加到容器里边。

![从Properties对象获取到EnableAutoConfiguration.class对应的值，然后添加到容器里边](https://segmentfault.com/img/remote/1460000018011545?w=1617&h=639)

最后我们会默认加载113个默认的配置类：

![img](https://segmentfault.com/img/remote/1460000018011546)

有兴趣的同学可以去翻一下这些文件以及配置类哦：

![加载的配置类和文件的信息](https://segmentfault.com/img/remote/1460000018011547?w=1911&h=874)

## 自定义配置

SpringBoot为我们提供了application.properties配置文件，让我们可以进行自定义配置，来对默认的配置进行修改，以适应具体的生产情况，当然还包括一些第三方的配置。几乎所有配置都可以写到application.peroperties文件中，这个文件会被SpringBoot自动加载，免去了我们手动加载的烦恼。但实际上，很多时候我们却会自定义配置文件，这些文件就需要我们进行手动加载，SpringBoot是不会自动识别这些文件的，下面就来仔细看看这些方面的内容。

### 属性源

为了使应用能适应不同的环境，SpringBoot支持外化配置。可以使用.properties文件、YAML文件、环境变量、命令行参数等方式。

SpringBoot能从多种属性源获得属性，包括以下几处：

1. 命令行参数
2. JVM系统属性
3. 操作系统环境变量
4. 随机生成的带random.*前缀的属性（在设置其他属性时，可以引用它们，比如${random.long})
5. 应用程序以外的application.properties或者application.yml文件
6. 打包在应用程序内的application.properites或者application.yml文件
7. 通过@propertySource标注的属性源
8. 默认属性

优先级由高到低，即在上面的列表中“命令行参数”的优先级最高，会覆盖下面其他属性源的相同配置。

这里只挑选了部分比较常见的属性源，详细信息可以参考官网教程：[24. Externalized Configuration](https://docs.spring.io/spring-boot/docs/2.0.1.BUILD-SNAPSHOT/reference/htmlsingle/#boot-features-external-config)

默认情况下SpringApplication将任何可选的命令行参数（以'--'开头，比如，--server.port=9000）转化为property，并将其添加到Spring Environment中。如上所述，命令行属性总是优先于其他属性源。

如果不想用这个特性可以用SpringApplication.setAddCommandLineProperties(false)来禁用。

### YAML

SpringBoot支持YAML格式的配置文件文件后缀为yml，是一种更易读写的通用的数据串行化格式。

在SpringBoot中.yml效果相当于.properties，通常情况下二者是可以互相替换的，比如下面2种配置文件在SpringBoot中是等效的：

application.properties

```
server.port=8020
server.address=127.0.0.1
```

application.yml

```
server:
  port: 8020
  address: 127.0.0.1
```

.yml文件在配置数据的时候具有面向对象的特性，更易阅读。

虽然.yml配置和.properties基本等效，但也有略微区别，.yml配置不能用@propertySource注解加载。~~~~

### Application属性文件

SpringBoot会从以下位置加载.properties或.yml配置文件：

1. 当前目录下的/config目录
2. 当前目录
3. classpath下的/config目录
4. classpath的根目录

优先级也是从高到低，高优先级的配置文件会覆盖低优先级的配置。

使用.properties或.yml配置文件可以对SpringBoot的自动配置进行覆盖，比如在默认的情况下http端口为8080，我们可以像上面的例子那样在修改为8020。

SpringBoot提供了上百个这样可以覆盖的配置，具体可以查阅官网或者查看autoconfiguration包的的META-INF/spring-configuration-metadata.json和META-INF/additional-spring-configuration-metadata.json文件（这2个文件用来给IDE做输入提示用，有一些简单的描述）。

### 类型安全的配置属性

有时候使用@Value("#{property}")注解注入配置会比较笨重，SpringBoot提供一个类型安全的方案，用强类型的Bean对象来替代属性。

@ConfigurationProperties用法如下

```
@ConfigurationProperties(prefix = "acme")
public class AcmeProperties {
    private boolean enabled;
    private final Security security = new Security();
    // 省略getter、setter

    public static class Security {
        private String username;
        private String password;
        private List<String> roles = new ArrayList<>(Collections.singleton("USER"));
        // 省略getter、setter
    }
}
```

@ConfigurationProperties的参数prefix表示前缀，AcmeProperties里的每个属性的名称便是具体的配置名。比如成员变量enabled绑定属性acme.enabled。

上面这个例子表示AcmeProperties对象分别绑定下列属性：

- acme.ebalbe：布尔类型
- acme.security.username：String类型
- acme.security.pasword：String类型
- acme.security.roles：类型是一个String集合

绑定配置之后还需要注册Spring上下文中，有3种方式：

1. 在java配置类中用@EnableConfigurationProperties注解激活

```
@Configuration
@EnableConfigurationProperties(AcmeProperties.class)
public class MyConfiguration {
}
```

2. 直接在AcmeProperties加@Component注解

```
@Component
@ConfigurationProperties(prefix="acme")
public class AcmeProperties {

    // ... see the preceding example
}
```

3. 在配置类中与@Bean注解组合

```
@Bean
public AnotherComponent anotherComponent() {
    ...
}
```

注册之后，可以在任意地方使用@Autowire注解注入使用。

### 松散的绑定(Relaxed binding)

Spring Boot使用一些宽松的规则用于绑定Environment属性到@ConfigurationProperties beans，所以Environment属性名和bean属性名不需要精确匹配。常见的有虚线匹配大写（比如，context-path绑定到contextPath）和将环境属性转为大写字母（比如，PORT绑定port）。

比如：

```
@ConfigurationProperties(prefix="acme.my-project.person")
public class OwnerProperties {
    private String firstName;
    // 省略getter、setter
}
```

可以绑定到：

| Property                            | Note                                |
| ----------------------------------- | ----------------------------------- |
| acme.my-project.person.firstName    | 标准的驼峰式命名                    |
| `acme.my-project.person.first-name` | 虚线表示，适用于.properties和.yml   |
| `acme.my-project.person.first_name` | 下划线表示，适用于.properties和.yml |
| `ACME_MYPROJECT_PERSON_FIRSTNAME`   | 大写形式，适用于环境变量            |

### @ConfigurationProperties vs @Value

| Feature                                                      | `@ConfigurationProperties` | `@Value` |
| ------------------------------------------------------------ | -------------------------- | -------- |
| [Relaxed binding](https://docs.spring.io/spring-boot/docs/2.0.1.BUILD-SNAPSHOT/reference/htmlsingle/#boot-features-external-config-relaxed-binding) | Yes                        | No       |
| [Meta-data support](https://docs.spring.io/spring-boot/docs/2.0.1.BUILD-SNAPSHOT/reference/htmlsingle/#configuration-metadata) | Yes                        | No       |
| `SpEL` evaluation                                            | No                         | Yes      |

## 通过类配置

前言
Spring3.0之前要使用Spring必须要有一个xml配置文件，而Spring3.0之后注解慢慢登上舞台，通过注解@Configuration和@Bean可以完全搞定。此时，注解和xml配置形成了相互协作与竞争的关系。随着Springboot的推广，注解的使用在Spring中大放光彩，xml的辉煌一去不返。通过注解，简化了配置，提升了编码效率。

Spring 3.0新增了另外两个实现类：AnnotationConfigApplicationContext 和 AnnotationConfigWebApplicationContext。它们是为注解而生，直接依赖于注解作为容器配置信息来源的IoC容器初始化类。AnnotationConfigWebApplicationContext是AnnotationConfigApplicationContext的web版本，其用法与后者相比几乎没有什么差别。

今天这篇文章带大家学习@Configuration和@Bean的使用，并通过具体的实例体验一下注解的方便快捷。如果你的项目中还未曾使用此类注解，说明你的技术栈已经在被淘汰的边缘。

基于Java类的配置选项
Spring 3.0引入了注解，配置文件的载体就从xml文件转换为了Java类，Java类就是一个普通的类，除了命名建议以“**Config”结尾方便识别外，Spring对其有一定的约定条件。

配置类不能是 final 类（没法动态代理）。
配置类必须是非本地的（即不能将配置类定义在其他类的方法内部，不能是private）。
配置类必须有一个无参构造函数。
基本使用方法
符合上述条件的类，就可以使用@Configuration来进行注解，表示这个类可以使用Spring IoC容器作为bean定义的来源。@Bean注解在该类的方法上，AnnotationConfigApplicationContext将配置类中标注了@Bean的方法的返回值识别为Spring Bean，并注册到容器中，归入IoC容器管理。

### @Configuration

的作用等价于XML配置中的标签，@Bean的作用等价于XML配置中的标签。下面代码完成了一个简单的示例。

```java
@Configuration
public class DataSourceConfig {
    @Bean
    public MysqlDataSource mysqlDataSource() {
      return new MysqlDataSource();
    }
	
    @Bean(name = "oracleDataSource")
    public OracleDataSource oracleDataSource() {
      return new OracleDataSource();
    }
}
```
Spring在解析该类时，会识别出标注@Bean的所有方法，执行并将方法的返回值(MysqlDataSource和OracleDataSource对象)注册到IoC容器中。默认情况下，方法名即为Bean的名字。与以上配置等价的XML配置如下：

```java
<bean id="mysqlDataSource" class="**.MysqlDataSource"/> 
<bean id="oracleDataSource" class="**.OracleDataSource"/>
@Configuration的属性
@Configuration的定义代码：

@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Configuration {
    String value() default "";
}
```

@Configuration注解本身定义时被@Component标注了，因此本质上来说@Configuration也是一个@Component，只不过我们在具体使用的过程中基本用不到它的实例化对象。

@Bean属性
@Bean注解的具体代码定义：

```java
@Target({ElementType.METHOD, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Bean {
    @AliasFor("name")
    String[] value() default {};

    @AliasFor("value")
    String[] name() default {};
    
    Autowire autowire() default Autowire.NO;
    
    String initMethod() default "";
    
    String destroyMethod() default "(inferred)";

}
```

可以看出@Bean具有以下属性：

name ：指定一个或者多个Bean的名字。等价于XML配置中的name属性，示例中的@Bean(name = “oracleDataSource”)。
initMethod：容器在初始化完Bean之后，会调用该属性指定的方法。等价于XML配置中的init-method属性。
destroyMethod：该属性与initMethod功能相似，在容器销毁Bean之前，会调用该属性指定的方法。等价于XML配置中的destroy-method属性。
autowire：指定Bean属性的自动装配策略，取值是Autowire类型的三个静态属性。Autowire.BY_NAME，Autowire.BY_TYPE，Autowire.NO。与XML配置中的autowire属性的取值相比，少了constructor，因为 constructor在这里已经没有意义了。
@Bean默认是单例模式，并且没有提供指定作用域的属性，可以通过@Scope来实现该功能。

```java
@Bean
@Scope("prototype")
public MysqlDataSource mysqlDataSource() {
	return new MysqlDataSource();
}
```

Spring扫描加载
当配置完Spring扫描指定包及其子包中的类时，会识别所有标记了@Component、@Controller、@Service、@Repository注解的类，由于@Configuration注解本身也用@Component标注了，Spring将能够识别出 @Configuration标注类。

实例
现在对上面的示例进行单元测试，其中MysqlDataSource和OracleDataSource类中分别提供了打印日志的方法：

```java
public void print(){
	System.out.println("I am MysqlDataSource!");
}
和
public void print(){
	System.out.println("I am OracleDataSource!");
}
```


指定单元测试的代码如下：

public class ConfigBeanTest {

```java
@Test
public static void testGetBean() {
	ApplicationContext ctx = new AnnotationConfigApplicationContext(DataSourceConfig.class);
	MysqlDataSource mysqlDataSource = ctx.getBean(MysqlDataSource.class);
	mysqlDataSource.print();
}
```
当执行单元测试，成功打印出：

I am MysqlDataSource!
实战技巧
如果我们在DataSourceConfig中再添加一个方法，这个方法用到了前面实例化的两个bean对象，那么该如何处理？常规的想法是在DataSourceConfig中添加如下代码，然后再直接使用属性：

@Autowired
private MysqlDataSource mysqlDataSource;

其实不必如此，直接调用方法mysqlDataSource()方法即可。比如，需要新增下面类的实例化：

```java
public class CommonDataSource {
    private MysqlDataSource mysqlDataSource;

    private OracleDataSource oracleDataSource;
    // 省略getter/setter方法
}
```

那么，我们只用在DataSourceConfig中添加如下代码即可：

```java
@Bean
public CommonDataSource commonDataSource() {
	CommonDataSource commonDataSource = new CommonDataSource();
	commonDataSource.setMysqlDataSource(mysqlDataSource());
	commonDataSource.setOracleDataSource(oracleDataSource());
	return commonDataSource;
}
```


注意：set方法内直接set的是mysqlDataSource()。

注意：set方法内直接set的是mysqlDataSource()。

小结
这节课我们讲解了Spring注解中@Configuration和@Bean使用方法，在Springboot中集成其他三方框架时，这种写法使用的越来越普遍。如果一时无法转换思维，可对照xml文件的配置进行逐一切换过来，比如xml要定义一个bean，那么用注解就是@Bean注解一个方法。如果方法里面的参数还有其他的依赖，那就采用上面介绍的实战技巧依次追踪set对应的参数，并将其先通过@Bean实例化。

### 查看当前生效的配置

SpringBoot默认提供了大量的自动配置，我们可以通过启动时添加--debug参数来查看当前的配置信息。

激活的配置如下：

```
Positive matches:
-----------------

   CodecsAutoConfiguration matched:
      - @ConditionalOnClass found required class 'org.springframework.http.codec.CodecConfigurer'; @ConditionalOnMissingClass did not find unwanted class (OnClassCondition)

   CodecsAutoConfiguration.JacksonCodecConfiguration matched:
      - @ConditionalOnClass found required class 'com.fasterxml.jackson.databind.ObjectMapper'; @ConditionalOnMissingClass did not find unwanted class (OnClassCondition)

...
```

未激活的配置如下：

```
Negative matches:
-----------------

   ActiveMQAutoConfiguration:
      Did not match:
         - @ConditionalOnClass did not find required classes 'javax.jms.ConnectionFactory', 'org.apache.activemq.ActiveMQConnectionFactory' (OnClassCondition)

   AopAutoConfiguration:
      Did not match:
         - @ConditionalOnClass did not find required classes 'org.aspectj.lang.annotation.Aspect', 'org.aspectj.lang.reflect.Advice', 'org.aspectj.weaver.AnnotatedElement' (OnClassCondition)
...
```

可以清楚的看到每个配置的条件，未激活的配置可以从这里直观的看到原因。

debug可以用-Ddebug或--debug来启用，也可以在.properties或.yml文件中配置debug的值为true。~~~~

###  

### 去除指定的自动配置

用@EnableAutoConfiguration注解的exclude参数去除指定的自动配置：

```
import org.springframework.boot.autoconfigure.*;
import org.springframework.boot.autoconfigure.jdbc.*;
import org.springframework.context.annotation.*;

@Configuration
@EnableAutoConfiguration(exclude={DataSourceAutoConfiguration.class})
public class MyConfiguration {
}
```

# 常用注解

### @Configuation

> 1. @Configuration不可以是final类型；
> 2. @Configuration不可以是匿名类；
> 3. 嵌套的configuration必须是静态类。

@Configuration可理解为用spring的时候xml里面的`<beans>`标签

从`Spring3.0`，`@Configuration`用于定义配置类，可替换`xml`配置文件，被注解的类内部包含有一个或多个被`@Bean`注解的方法，这些方法将会被`AnnotationConfigApplicationContext`或`AnnotationConfigWebApplicationContext`类进行扫描，并用于构建`bean`定义，初始化`Spring`容器。

```java
@Configuration
public class AppConfig {
    @Bean
    public MyBean myBean() {
        // instantiate, configure and return bean ...
    }
}

AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
ctx.register(AppConfig.class);//加载配置类
ctx.refresh();//刷新并创建容器
MyBean myBean = ctx.getBean(MyBean.class);
// use myBean ...
```

Spring Boot不是spring的加强版，所以@Configuration和@Bean同样可以用在普通的spring项目中，而不是Spring Boot特有的，只是在spring用的时候，注意加上扫包配置

<context:component-scan base-package="com.xxx.xxx" />，普通的spring项目好多注解都需要扫包，才有用，有时候自己注解用的挺6，但不起效果，就要注意这点。

Spring Boot则不需要，主要你保证你的启动Spring Boot main入口，在这些类的上层包就行
@Configuration和@Bean的Demo类

```java
@Configuration  
public class ExampleConfiguration {  
    @Value("com.mysql.jdbc.Driver")  
    private String driverClassName;  

    @Value("jdbc://xxxx.xx.xxx/xx")  
    private String driverUrl;  

    @Value("${root}")  
    private String driverUsername;  

    @Value("123456")  
    private String driverPassword;  

    @Bean(name = "dataSource")  
    public DataSource dataSource() {  
        BasicDataSource dataSource = new BasicDataSource();  
        dataSource.setDriverClassName(driverClassName);  
        dataSource.setUrl(driverUrl);  
        dataSource.setUsername(driverUsername);  
        dataSource.setPassword(driverPassword);  
        return dataSource;  
    }  

    @Bean  
    public PlatformTransactionManager transactionManager() {  
        return new DataSourceTransactionManager(dataSource());  
    }  
}
```


这样，在项目中

```java
@Autowired
private DataSource dataSource;
```

的时候，这个dataSource就是我们在ExampleConfiguration中配的DataSource

#### @Configuration配置spring并启动spring容器

`@Configuration`标注在类上，相当于把该类作为spring的xml配置文件中的，作用为：配置spring容器(应用上下文)

测试

```java
package config;

import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class TestConfig {
    public static void main(String[] args){
        // @Configuration注解的spring容器加载方式，用AnnotationConfigApplicationContext替换ClassPathXmlApplicationContext
        ApplicationContext context = new AnnotationConfigApplicationContext(TestConfiguration.class);

        // 如果加载spring-context.xml文件：
        // ApplicationContext context = new
        // ClassPathXmlApplicationContext("spring-context.xml");
    }
}
```

测试结果如下：

```java
四月 03, 2018 10:12:52 上午 org.springframework.context.annotation.AnnotationConfigApplicationContext prepareRefresh
信息: Refreshing org.springframework.context.annotation.AnnotationConfigApplicationContext@179d3b25: startup date [Tue Apr 03 10:12:52 CST 2018]; root of context hierarchy
TestConfiguration 容器启动初始化...
```

### @ConditionalOnProperty

这个注解能够控制某个configuration是否生效。具体操作是通过其两个属性name以及havingValue来实现的，其中name用来从application.properties中读取某个属性值，如果该值为空，则返回false;如果值不为空，则将该值与havingValue指定的值进行比较，如果一样则返回true;否则返回false。如果返回值为false，则该configuration不生效；为true则生效。

### @Conponent

`@Component`, `@Service`, `@Controller`, `@Repository`是spring注解，注解后可以被spring框架所扫描并注入到spring容器来进行管理
`@Component`是通用注解，其他三个注解是这个注解的拓展，并且具有了特定的功能
`@Repository`注解在持久层中，具有将数据库操作抛出的原生异常翻译转化为spring的持久层异常的功能。
`@Controller`层是spring-mvc的注解，具有将请求进行转发，重定向的功能。
`@Service`层是业务逻辑层注解，这个注解只是标注该类处于业务逻辑层。
用这些注解对应用进行分层之后，就能将请求处理，义务逻辑处理，数据库操作处理分离出来，为代码解耦，也方便了以后项目的维护和开发。

### @Bean

@Bean可理解为用spring的时候xml里面的`<bean>`标签

### @RestController 

@RestController是一个组合注解，写在类上面，是组合了@ResponseBody和@Controller，默认了类中所有的方法都包含ResponseBody注解的一种简写形式，表示Controller 里面的方法都以 json 格式输出，不用再写什么 jackjson 配置的了！

1)如果只是使用@RestController注解Controller，则Controller中的方法无法返回jsp页面，配置的视图解析器InternalResourceViewResolver则不起作用，返回的内容就是Return 里的内容（String/JSON）。
例如：本来应该到success.jsp页面的，则其显示success.

```java
@RequestMapping(value = "/test")
public String test(HttpServletRequest request, HttpServletResponse response){   				return "success";
}
```

2)如果使用@RestController注解Controller，需要返回到指定页面，则需要配置视图解析器InternalResourceViewResolver，可以利用ModelAndView返回试图。

```java
@RequestMapping(value = "/test")
public String test(HttpServletRequest request, HttpServletResponse response){   				return newModelAndView("success");
}
```

3)如果使用@Controller注解Controller，如果需要返回JSON，XML或自定义mediaType内容到页面，则需要在对应的方法上加上@ResponseBody注解。

```java
@ResponseBody
@RequestMapping(value = "/test")
public String test(HttpServletRequest request, HttpServletResponse response){   
    return "success";
}
```

### @Controller

以前在编写Controller方法的时候，需要开发者自定义一个Controller类实现Controller接口，实现handleRequest方法返回ModelAndView。并且需要在Spring配置文件中配置Handle，将某个接口与自定义Controller类做映射。

这么做有个复杂的地方在于，一个自定义的Controller类智能处理一个单一请求。而在采用@Contoller注解的方式，可以使接口的定义更加简单，将**@Controller标记在某个类上，配合@RequestMapping注解，可以在一个类中定义多个接口**，这样使用起来更加灵活。

**被@Controller标记的类实际上就是个SpringMVC Controller对象，它是一个控制器类，而@Contoller注解在org.springframework.stereotype包下。其中被@RequestMapping标记的方法会被分发处理器扫描识别，将不同的请求分发到对应的接口上。**

### @Repository

Spring 自 2.0 版本开始，陆续引入了一些注解用于简化 Spring 的开发。@Repository注解便属于最先引入的一批，它用于将数据访问层 (DAO 层 ) 的类标识为 Spring Bean。具体只需将该注解标注在 DAO类上即可。同时，为了让 Spring 能够扫描类路径中的类并识别出 @Repository 注解，需要在 XML 配置文件中启用Bean 的自动扫描功能，这可以通过`<context:component-scan/>`实现

为什么只能标注在 DAO 类上呢？

这是因为该注解的作用不只是将类识别为Bean，同时它还能将所标注的类中抛出的数据访问异常封装为 Spring 的数据访问异常类型。 Spring本身提供了一个丰富的并且是与具体的数据访问技术无关的数据访问异常结构，用于封装不同的持久层框架抛出的异常，使得异常独立于底层的框架。

### @RequestMapping 

是一个用来处理请求地址映射的注解，可用于类或方法上。用于类上，表示类中的所有响应请求的方法都是以该地址作为父路径；用于方法上，表示在类的父路径下追加方法上注解中的地址将会访问到该方法。

### @Responsebody 

注解表示该方法的返回的结果直接写入 HTTP 响应正文（ResponseBody）中，一般在异步获取数据时使用；
在使用 @RequestMapping 后，返回值通常解析为跳转路径，加上 @Responsebody 后返回结果不会被解析为跳转路径，而是直接写入HTTP 响应正文中。例如，异步获取 json 数据，加上 @Responsebody 注解后，就会直接返回 json 数据。

```java
@RequestMapping(value = "person/login", method = RequestMethod.GET)
@ResponseBody
public Person login(@RequestBody Person person) {   // 将请求中的 datas 写入 Person 对象中
    return person;    // 不会被解析为跳转路径，而是直接写入 HTTP 响应正文中
}
```

### @GetMapping

用于将HTTP get请求映射到特定处理程序的方法注解
是一个组合注解，是@RequestMapping(method = RequestMethod.GET)的缩写

### @PostMapping

用于将HTTP post请求映射到特定处理程序的方法注解
是一个组合注解，是@RequestMapping(method = RequestMethod.POST)的缩写。

在Spring 4.3以后，引入了@GetMapping、@PostMapping、@PutMapping、@DeleteMapping和@PatchMapping，一共5个注解。

**注：**

请求资源应该使用GET;
添加资源应该使用POST;
更新资源应该使用PUT;
删除资源应该使用DELETE.

### @Slf4j

- slf4j是一个日志标准，使用它可以完美的桥接到具体的日志框架，必要时可以简便的更换底层的日志框架，而不需要关心具体的日志框架的实现（slf4j-simple、logback等）。
- 每次写新的类，就需要重新写logger，麻烦，可以使用@Slf4j注解简化

### @PostConstruct

Spring的@PostConstruct注解在方法上，表示此方法是在Spring实例化该Bean之后马上执行此方法，之后才会去实例化其他Bean，并且一个Bean中@PostConstruct注解的方法可以有多个。

### @RequestParam

看下面一段代码：

```
http://localhost:8080/springmvc/hello/101?param1=10&param2=20
```

根据上面的这个URL，你可以用这样的方式来进行获取

```java
public String getDetails(
    @RequestParam(value="param1", required=true) String param1,
        @RequestParam(value="param2", required=false) String param2){
}
```

> @RequestParam 支持下面四种参数

- defaultValue 如果本次请求没有携带这个参数，或者参数为空，那么就会启用默认值
- name 绑定本次参数的名称，要跟URL上面的一样
- required 这个参数是不是必须的
- value 跟name一样的作用，是name属性的一个别名

### @ApiOperation

不是spring自带的注解

是swagger里的 com.wordnik.swagger.annotations.ApiOperation;

@ApiOperation和@ApiParam为添加的API相关注解，个参数说明如下： 

> @ApiOperation(value = “接口说明”, httpMethod = “接口请求方式”, response = “接口返回参数类型”, notes = “接口发布说明”；

> @ApiParam(required = “是否必须参数”, name = “参数名称”, value = “参数具体描述”

### @Async

@Async注解标记的方法，称为异步方法，它会在调用方的当前线程之外的独立的线程中执行，其实就相当于我们自己new Thread(()-> System.out.println("hello world !"))这样在另一个线程中去执行相应的业务逻辑。

####  @Async注解使用条件：

1. @Async注解一般用在类的方法上，如果用在类上，那么这个类所有的方法都是异步执行的；
2. 所使用的@Async注解方法的类对象应该是Spring容器管理的bean对象；
3. 调用异步方法类上需要配置上注解@EnableAsync

#### @EnableAsync

EnableAsync注解的意思是可以异步执行，就是开启多线程的意思。可以标注在方法、类上。

SpringBoot 提供了注解 `@EnableAsync` + `@Async` 实现方法的异步调用。使用方法超级简单，在启动类上加上 `@EnableAsync` 注解开启项目的异步调用功能，再在需异步调用的方法上加上注解 `@Async` 即可实现方法的异步调用。

### @scheduled注解作用

用来开启定时任务

#### cron：通过表达式来配置任务执行时间

**cron表达式详解**

表达式总共七个部分，含义为：“秒 分 时 日期 月 星期几 [年]”
其中年不是必填项，所以Cron表达式可能也只有六部分

![image-20200110105426450](/Users/wangchong/Library/Application Support/typora-user-images/image-20200110105426450.png)

#### 特殊字符

![image-20200110105508531](/Users/wangchong/Library/Application Support/typora-user-images/image-20200110105508531.png)

#### 示例

```ruby
  每隔5秒执行一次：*/5 * * * * ?
  每隔1分钟执行一次：0 */1 * * * ?
  每天23点执行一次：0 0 23 * * ?
  每天凌晨1点执行一次：0 0 1 * * ?
  每月1号凌晨1点执行一次：0 0 1 1 * ?
  每月最后一天23点执行一次：0 0 23 L * ?
  每周星期天凌晨1点实行一次：0 0 1 ? * L
  在26分、29分、33分执行一次：0 26,29,33 * * * ?
  每天的0点、13点、18点、21点都执行一次：0 0 0,13,18,21 * * ?
	0 0 10,14,16 * * ? 每天上午10点，下午2点，4点
	0 0/30 9-17 * * ?   朝九晚五工作时间内每半小时
	0 0 12 ? * WED 表示每个星期三中午12点
	“0 0 12 * * ?” 每天中午12点触发 
	“0 15 10 ? * *” 每天上午10:15触发
	“0 15 10 * * ?” 每天上午10:15触发
	“0 15 10 * * ? *” 每天上午10:15触发
	“0 15 10 * * ? 2005” 2005年的每天上午10:15触发
	“0 * 14 * * ?” 在每天下午2点到下午2:59期间的每1分钟触发
	“0 0/5 14 * * ?” 在每天下午2点到下午2:55期间的每5分钟触发
	“0 0/5 14,18 * * ?” 在每天下午2点到2:55期间和下午6点到6:55期间的每5分钟触发
	“0 0-5 14 * * ?” 在每天下午2点到下午2:05期间的每1分钟触发
	“0 10,44 14 ? 3 WED” 每年三月的星期三的下午2:10和2:44触发
	“0 15 10 ? * MON-FRI” 周一至周五的上午10:15触发
	“0 15 10 15 * ?” 每月15日上午10:15触发
	“0 15 10 L * ?” 每月最后一日的上午10:15触发
	“0 15 10 ? * 6L” 每月的最后一个星期五上午10:15触发
	“0 15 10 ? * 6L 2002-2005” 2002年至2005年的每月的最后一个星期五上午10:15触发
	“0 15 10 ? * 6#3” 每月的第三个星期五上午10:15触发 
	0 0 10,14,16 * * ? 每天上午10点，下午2点，4点 
	0 0/30 9-17 * * ? 朝九晚五工作时间内每半小时 
	0 0 12 ? * WED 表示每个星期三中午12点 
	"0 0 12 * * ?" 每天中午12点触发 
	"0 15 10 ? * *" 每天上午10:15触发 
	"0 15 10 * * ?" 每天上午10:15触发 
	"0 15 10 * * ? *" 每天上午10:15触发 
	"0 15 10 * * ? 2005" 2005年的每天上午10:15触发 
	"0 * 14 * * ?" 在每天下午2点到下午2:59期间的每1分钟触发 
	"0 0/5 14 * * ?" 在每天下午2点到下午2:55期间的每5分钟触发 
	"0 0/5 14,18 * * ?" 在每天下午2点到2:55期间和下午6点到6:55期间的每5分钟触发 
	"0 0-5 14 * * ?" 在每天下午2点到下午2:05期间的每1分钟触发 
	"0 10,44 14 ? 3 WED" 每年三月的星期三的下午2:10和2:44触发 
	"0 15 10 ? * MON-FRI" 周一至周五的上午10:15触发 
	"0 15 10 15 * ?" 每月15日上午10:15触发 
	"0 15 10 L * ?" 每月最后一日的上午10:15触发 
	"0 15 10 ? * 6L" 每月的最后一个星期五上午10:15触发 
	"0 15 10 ? * 6L 2002-2005" 2002年至2005年的每月的最后一个星期五上午10:15触发 
	"0 15 10 ? * 6#3" 每月的第三个星期五上午10:15触发
```

## @Autowired、@Resource

> 注入有4种模式，byName、byType、constructor、autodectect

功能上是一样的（DI注解），DI注解其实有两套：

- Spring官方的注解：@Autowired注解

- JavaEE的注解：@Resource注解

共同点：都需要配置DI注解解析器

```xml
<!-- DI注解解析器 -->
<context:annotation-config/>
```

### @Autowired

@Autowired为Spring提供的注解，需要导入包org.springframework.beans.factory.annotation.Autowired;只按照byType注入。

@Autowired注解是按照类型（byType）装配依赖对象，默认情况下它要求依赖对象必须存在，如果允许null值，可以设置它的required属性为false。如果我们想使用按照名称（byName）来装配，可以结合@Qualifier注解一起使用。(通过类型匹配找到多个candidate,在没有@Qualifier、@Primary注解的情况下，会使用对象名作为最后的fallback匹配)


规则如下

1. Spring先去容器中寻找对应类型的bean，若找不到一个bean，会抛出异常
2. 若找到一个类型的bean，则注入bean。若该类型的bean有多个，则扫描字段名进行名字匹配

特例：

```java
@Autowired
@Qualifier("userJdbcImps")
private UserRepository userRepository;
```

若有多个UserRepository 类型的bean，可以指定bean的名称，名称为userJdbcImps，装配到userRepository中

```java
@Autowired（required = false）
NewsService newsService;
```

若找不到bean时不会抛出异常

### @Resource

@Resource默认按照ByName自动注入，由J2EE提供，需要导入包javax.annotation.Resource。@Resource有两个重要的属性：name和type，而Spring将@Resource注解的name属性解析为bean的名字，而type属性则解析为bean的类型。所以，如果使用name属性，则使用byName的自动注入策略，而使用type属性时则使用byType自动注入策略。如果既不制定name也不制定type属性，这时将通过反射机制使用byName自动注入策略。

@Resource装配顺序：

1. 如果同时指定了name和type，则从Spring上下文中找到唯一匹配的bean进行装配，找不到则抛出异常。
2. 如果指定了name，则从上下文中查找名称（id）匹配的bean进行装配，找不到则抛出异常。
3. 如果指定了type，则从上下文中找到类似匹配的唯一bean进行装配，找不到或是找到多个，都会抛出异常。
4. 如果既没有指定name，又没有指定type，则自动按照byName方式进行装配；如果没有匹配，则回退为一个原始类型进行匹配，如果匹配则自动装配。

@Resource的作用相当于@Autowired，只不过@Autowired按照byType自动注入。

### @EnableCaching

Spring 3.1 引入了激动人心的基于注释（annotation）的缓存（cache）技术，它本质上不是一个具体的缓存实现方案（例如 EHCache 或者 OSCache），而是一个对缓存使用的抽象，通过在既有代码中添加少量它定义的各种 annotation，即能够达到缓存方法的返回对象的效果。

Spring 的缓存技术还具备相当的灵活性，不仅能够使用 SpEL（Spring Expression Language）来定义缓存的 key 和各种 condition，还提供开箱即用的缓存临时存储方案，也支持和主流的专业缓存例如 EHCache 集成。

```java
import org.springframework.cache.annotation.Cacheable;
 
public class Book {
	/**
	 * value : 缓存的名字  ,key ： 缓存map中的key
	 * @param id
	 * @return
	 */
    @Cacheable(value = { "sampleCache" },key="#id")
    public String getBook(int id) {
        System.out.println("Method executed..");
        if (id == 1) {
            return "Book 1";
        } else {
            return "Book 2";
        }
    }
}
```

@Cacheable(value=“sampleCache”)，这个注释的意思是，当调用这个方法的时候，会从一个名叫 sampleCache 的缓存(缓存本质是一个map)中查询key为id的值，如果不存在，则执行实际的方法（即查询数据库等服务逻辑），并将执行的结果存入缓存中，否则返回缓存中的对象。这里的缓存中的 key 就是参数 id，value 就是 返回的String 对象


#### cache配置类


```java
package com.learn.frame.spring.cache;

import java.util.Arrays;

import org.springframework.cache.CacheManager;
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.cache.concurrent.ConcurrentMapCache;
import org.springframework.cache.support.SimpleCacheManager;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
 /**
   * spring cache 缓存的使用
   */
   @Configuration
   @EnableCaching
   public class CachingConfig {
   @Bean
   public Book book() {
       return new Book();
   }

   @Bean
   public CacheManager cacheManager() {
       SimpleCacheManager cacheManager = new SimpleCacheManager();
       cacheManager.setCaches(Arrays.asList(new ConcurrentMapCache("sampleCache")));//注册名为sampleCache的缓存
       return cacheManager;
   }
public static void main(String[] args) {
         AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext(); 
        ctx.register(CachingConfig.class);
        ctx.refresh();
        Book book = ctx.getBean(Book.class);        
        // calling getBook method first time.
        System.out.println(book.getBook(1));
        // calling getBook method second time. This time, method will not
        // execute.
        System.out.println(book.getBook(1));
        // calling getBook method third time with different value.
        System.out.println(book.getBook(2));
        ctx.close();
}
}
```

@EnableCaching

注解是spring framework中的注解驱动的缓存管理功能。自spring版本3.1起加入了该注解。如果你使用了这个注解，那么你就不需要在XML文件中配置cache manager了，等价于 `<cache:annotation-driven/>` 。能够在服务类方法上标注@Cacheable

CacheManager ： 管理Cache对象

Cache ： 用合适的数据结构存储数据（上述例子使用ConcurrentMapCache map结构存储数据对象）

上面的java config和下面的xml配置文件是等效的

```xml
<beans>
     <cache:annotation-driven/>
     <bean id="book" class="Book"/>
     <bean id="cacheManager" class="org.springframework.cache.support.SimpleCacheManager">
         <property name="caches">
             <set>
                 <bean class="org.springframework.cache.concurrent.ConcurrentMapCacheFactoryBean">
                     <property name="name" value="sampleCache"/>
                 </bean>
             </set>
         </property>
     </bean>
 </beans>
```


定义了一个缓存管理器SimpleCacheManager 并设置名称为sampleCache的缓存，@Cachable的value属性要和此对应。

测试输出结果

> Method executed..
> Book 1
> Book 1
> Method executed..
> Book 2

## 校验

### @Valid+BindingResult进行controller参数校验

由于controller是调用的第一层，经常参数校验将在这里完成，常见有非空校验、类型校验等，常见写法为以下伪代码：

```java
public void round(Object a){

  if(a.getLogin() == null){
     return "手机号不能为空！";
   }
}
```

但是调用对象的位置会有很多，而且手机号都不能为空，那么我们会想到把校验方法抽出来，避免重复的代码。但有框架支持我们通过注解的方式进行参数校验。

先立个场景，为往动物园添加动物，动物对象如下，时间节点大概在3030年，我们认为动物可登陆动物专用的系统，所以有password即自己的登录密码。

```java
public class Animal {

    private String name;

    private Integer age;

    private String password;

    private Date birthDay;

}
```

调用创建动物的controller层如下，简洁明了，打印下信息后直接返回。

```java
@RestController
@RequestMapping("/animal")
public class AnimalController {
   @PostMapping
    public Animal createAnimal(@RequestBody Animal animal){
        logger.info(animal.toString());
        return animal;
    }
}
```

伪造Mvc调用的测试类。

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class TestAnimal {
    private final static Logger logger = LoggerFactory.getLogger(TestAnimal.class);
   @Autowired
    private WebApplicationContext wac;
    private MockMvc mockMvc;
    @Before
    public void initMock(){
        mockMvc = MockMvcBuilders.webAppContextSetup(wac).build();
    }
    @Test
    public void createAnimal() throws Exception {
        String content = "{\"name\":\"elephant\",\"password\":null,\"birthDay\":"+System.currentTimeMillis()+"}";
        String result = mockMvc.perform(MockMvcRequestBuilders.post("/animal")
                .content(content)
                .contentType(MediaType.APPLICATION_JSON_UTF8))
                .andExpect(MockMvcResultMatchers.status().isOk())
                .andReturn().getResponse().getContentAsString();
        logger.info(result);
    }
}
```

以上代码基于搭建的springboot项目，想搭建的同学可以参考姊妹搭建篇 https://blog.csdn.net/FU250/article/details/80208261

代码分析，日期格式的参数建议使用时间戳传递，以上birthDay传递 "2018-05-08 20:00:00"，将会抛出日期转换异常，感兴趣的同学可以试试。

由于密码很重要，现在要求密码为必填，操作如下，添加@NotBlank注解到password上：

```java
@NotBlank
private String password;
```

但光加校验注解是不起作用的，还需要在方法参数上添加@Valid注解，如下：

```java
@Valid @RequestBody Animal animal
```

此时执行测试方法，抛出异常，返回状态为400：

```java
java.lang.AssertionError: Status 
Expected :200
Actual   :400
 <Click to see difference>
	at org.springframework.test.util.AssertionErrors.fail(AssertionErrors.java:54)
	at org.springframework.test.util.AssertionErrors.assertEquals(AssertionErrors.java:81)
```

说明对password的非空校验已经生效了，直接抛出异常。如果不想抛出异常，想返回校验信息给前端，这个时候就需要用到BindingResult了，修改创建动物的方法，添加BindingResult参数：

```java
@PostMapping



    public Animal createAnimal(@Valid @RequestBody Animal animal, BindingResult bindingResult){
        if (bindingResult.hasErrors()){
            bindingResult.getAllErrors().forEach(o ->{
                FieldError error = (FieldError) o;
	              logger.info(error.getField() + ":" + error.getDefaultMessage());
            });
        }
        logger.info(animal.toString());
        return animal;
    }
```

此时，执行测试，可以看到日志中的错误信息：

```html
2018-05-09 00:59:37.453  INFO 14044 --- [           main] c.i.s.d.web.controller.AnimalController  : password:may not be empty
```

为了满足我们编码需要我们需要进行代码改造，1.不能直接返回animal。2.返回的提示信息得是用户可读懂的信息。

controller方法改造如下，通过Map对象传递请求成功后的信息或错误提示信息。

```java
		@PostMapping
    public Map<String,Object> createAnimal(@Valid @RequestBody Animal animal, BindingResult bindingResult){
        logger.info(animal.toString());
        Map<String,Object> result = new HashMap<>();
        if (bindingResult.hasErrors()){
            FieldError error = (FieldError) bindingResult.getAllErrors().get(0);
            result.put("code","400");//错误编码400
            result.put("message",error.getDefaultMessage());//错误信息
				    return result;
        }
        result.put("code","200");//成功编码200
        result.put("data",animal);//成功返回数据
        return result;
    }
```

返回的密码提示信息如下：

```java
@NotBlank(message = "密码不能为空！")



private String password;
```

执行测试方法，返回结果

```java
com.imooc.security.demo.TestAnimal       : {"code":"400","message":"密码不能为空！"}
```

最后贴一个，设置password值返回成功的信息

```java
com.imooc.security.demo.TestAnimal       : {"code":"200","data":{"name":"elephant","age":null,"password":"lalaland","birthDay":1525799768955}}
```


## Java

### StopWatch类

可以获得任务执行时间

```java
@Test
public void test1() throws InterruptedException {
        StopWatch stopWatch =new StopWatch();
        stopWatch.start();
        int sum=0;
        for(int i=0;i<100000;i++){
            sum+=i;
        }
        Thread.sleep(2000);
        stopWatch.stop();
        System.out.println("总计是："+sum);
        System.out.println("耗时："+stopWatch.getTotalTimeMillis()+"毫秒");
        System.out.println("耗时："+stopWatch.getTotalTimeSeconds()+"秒");
       //打印一份格式化好的汇总统计信息
        System.out.println(stopWatch.prettyPrint());
    }
```

> 总计是：704982704
> 耗时：2004毫秒
> 耗时：2.004秒

### 双冒号

运算就是 java 中的[方法引用]。 [方法引用]格式为：类名::方法名。

注意是方法名，方法名，方法名！重要的事情说三遍！后面没有括号“()”的。

为啥不要括号 ?

因为这样的是式子并不代表一定会调用这个方法。这种式子一般是用作Lambda表达式，Lambda有所谓懒加载嘛，不要括号就是说，看情况调用方法。

#### 使用范例

方法调用

```java
person -> person.getAge();
可以替换成
Person::getAge

x -> System.out.println(x)
可以替换成
System.out::println
out是一个PrintStream类的对象，println是该类的方法，依据x的类型来重载方法

创建对象

() -> new ArrayList<>();
可以替换为
ArrayList::new
```

### AtomicLong

#### **AtomicLong介绍**

AtomicLong是作用是对长整形进行原子操作。
在32位操作系统中，64位的long 和 double 变量由于会被JVM当作两个分离的32位来进行操作，所以不具有原子性。而使用AtomicLong能让long的操作保持原子型。

#### **AtomicLong的几个常用方法**

- 创建具有初始值 0 的新 AtomicLong

```java
AtomicLong atomicLong = new AtomicLong();
```

- 创建具有给定初始值的新 AtomicLong。        

```java
AtomicLong atomicLong1 = new AtomicLong(10);
```

- addAndGet()方法：以原子方式将给定值添加到当前值，先加上特定的值，再获取结果

```java
AtomicLong atomicLong = new AtomicLong(3);
atomicLong.addAndGet(5)
```

- getAndAdd()方法：先获取当前值再加上特定的值

```java
atomicLong5.getAndAdd(5);
```

- compareAndSet()方法：如果当前值 == 预期值，则以原子方式将该值设置为给定的更新值

```java
AtomicLong atomicLong3 = new AtomicLong(10);
atomicLong3.compareAndSet(10,15);
```

- decrementAndGet()方法：以原子方式将当前值减 1,先减去1再获取值

- getAndDecrement()方法：先获取当前值再减1
- getAndIncrement()方法：先获取当前值再加1
- incrementAndGet()方法：先加1再获取当前值
- getAndSet()方法：先获取当前值再设置新的值

```java
atomicLong.getAndSet(20);
```

### Stream.map()

如果需要将流中的元素映射到另一个流中，可以使用map 方法。方法签名：

```java
<R> Stream<R> map(Function<? super T, ? extends R> mapper);
```


该接口需要一个Function 函数式接口参数，可以将当前流中的T类型数据转换为另一种R类型的流。 

复习Function接口

java.util.stream.Function 函数式接口，其中唯一的抽象方法为：

```java
R apply(T t);
```


这可以将一种T类型转换成为R类型，而这种转换的动作，就称为“映射”。

基本使用

```java
import java.util.stream.Stream;
/*
    Stream流中的常用方法_map:用于类型转换
    如果需要将流中的元素映射到另一个流中，可以使用map方法.
    <R> Stream<R> map(Function<? super T, ? extends R> mapper);
    该接口需要一个Function函数式接口参数，可以将当前流中的T类型数据转换为另一种R类型的流。
    Function中的抽象方法:
        R apply(T t);
 */
public class StreamMap {
    public static void main(String[] args) {
        //获取一个String类型的Stream流
        Stream<String> stream = Stream.of("1", "2", "3", "4");
        //使用map方法,把字符串类型的整数,转换(映射)为Integer类型的整数
        Stream<Integer> stream2 = stream.map((String s)->{
            return Integer.parseInt(s);
        });
        //遍历Stream2流
        stream2.forEach(i-> System.out.println(i));
    }
}
```

### POJO（Plain Ordinary Java Object）简单的Java对象

实际就是普通JavaBeans，是为了避免和EJB混淆所创造的简称。
使用POJO名称是为了避免和EJB混淆起来, 而且简称比较直接. 其中有一些属性及其getter setter方法的类,没有业务逻辑，有时可以作为VO(value -object)或dto(Data Transform Object)来使用.当然,如果你有一个简单的运算属性也是可以的,但不允许有业务方法,也不能携带有connection之类的方法。

### POJO与javabean的区别

POJO 和JavaBean是我们常见的两个关键字，一般容易混淆，POJO全称是Plain Ordinary Java Object / Pure Old Java Object，中文可以翻译成：普通Java类，具有一部分getter/setter方法的那种类就可以称作POJO，但是JavaBean则比 POJO复杂很多， Java Bean 是可复用的组件，对 Java Bean 并没有严格的规范，理论上讲，任何一个 Java 类都可以是一个 Bean 。但通常情况下，由于 Java Bean 是被容器所创建（如 Tomcat) 的，所以 Java Bean 应具有一个无参的构造器，另外，通常 Java Bean 还要实现 Serializable 接口用于实现 Bean 的持久性。 Java Bean 是不能被跨进程访问的。

JavaBean是一种组件技术，就好像你做了一个扳子，而这个扳子会在很多地方被拿去用，这个扳子也提供多种功能(你可以拿这个扳子扳、锤、撬等等)，而这个扳子就是一个组件。

一般在web应用程序中建立一个数据库的映射对象时，我们只能称它为POJO。POJO(Plain Old Java Object)这个名字用来强调它是一个普通java对象，而不是一个特殊的对象，其主要用来指代那些没有遵从特定的Java对象模型、约定或框架（如EJB）的Java对象。理想地讲，一个POJO是一个不受任何限制的Java对象（除了Java语言规范）。JavaBean 是一种JAVA语言写成的可重用组件。

JavaBean符合一定规范编写的Java类，不是一种技术，而是一种规范。大家针对这种规范，总结了很多开发技巧、工具函数。符合这种规范的类，可以被其它的程序员或者框架使用。它的方法命名，构造及行为必须符合特定的约定

1、所有属性为private。

2、这个类必须有一个公共的缺省构造函数。即是提供无参数的构造器。

3、这个类的属性使用getter和setter来访问，其他方法遵从标准命名规范。

4、这个类应是可序列化的。实现serializable接口。

- 因为这些要求主要是靠约定而不是靠实现接口，所以许多开发者把JavaBean看作遵从特定命名约定的POJO。
- POJO其实是比javabean更纯净的简单类或接口。POJO严格地遵守简单对象的概念，而一些JavaBean中往往会封装一些简单逻辑。
- POJO主要用于数据的临时传递，它只能装载数据， 作为数据存储的载体，而不具有业务逻辑处理的能力。
- Javabean虽然数据的获取与POJO一样，但是javabean当中可以有其它的方法。
- entity（实体类）：一般的实体类对应一个数据表，其中的属性对应数据表中的字段。 

#### 各种对象

 - PO: POJO在持久层的体现，对POJO持久化后就成了PO。PO更多的是跟数据库设计层面相关，一般PO与数据表对应，一个PO就是对应数据表的一条记录。 
  - DAO: PO持久化到数据库是要进行相关的数据库操作的(CRUQ)，这些对数据库操作的方法会统一放到一个Java对象中，这就是DAO。

- BO: POJO在业务层的体现，对于业务操作来说，更多的是从业务上来包装对象，如一个User的BO，可能包括name, age, sex, privilege, group等，这些属性在数据库中可能会在多张表中，因为每一张表对应一个PO，而我们的BO需要这些PO组合起来(或说重新拼装)才能成为业务上的一个完整对象。

 - VO(Value Object/View Object): POJO在表现层的体现。 当我们处理完数据时，需要展现时，这时传递到表现层的POJO就成了VO。它就是为了展现数据时用的。

 - DTO(Data Transfer Object): POJO在系统间传递时。当我们需要在两个系统间传递数据时，一种方式就是将POJO序列化后传递，这个传递状态的POJO就是DTO。

 - EJB(Enterprise JavaBean): 我认为它是一组”功能”JavaBean的集合。上面说了JavaBean是实现了一种规范的Java对象。这里说EJB是一组JavaBean，的意思是这一组JavaBean组合起来实现了某个企业组的业务逻辑。这里的一组JavaBean不是乱组合的，它们要满足能实现某项业务功能的搭配。找个比方，对于一身穿着来说，包括一顶帽子，一件衣服，一条裤子，两只鞋,这穿着就是EJB.

# Spring MVC

Spring MVC是Spring的一部分，Spring 出来以后，大家觉得很好用，于是按照这种模式设计了一个 MVC框架（一些用Spring 解耦的组件），**主要用于开发WEB应用和网络接口，它是Spring的一个模块，通过Dispatcher Servlet, ModelAndView 和 View Resolver，让应用开发变得很容易**，一个典型的Spring MVC应用开发分为下面几步：
 首先通过配置文件声明Dispatcher Servlet：

```xml
    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>com.qgd.oms.web.common.mvc.OmsDispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>/WEB-INF/applicationContext.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
```

通过配置文件声明servlet详情，如MVC resource，data source，bean等



```xml
    <mvc:resources mapping="/css/**/*" location="/static/css/" cache-period="21600"/>
    <mvc:resources mapping="/js/**/*" location="/static/js/" cache-period="21600"/>
    <mvc:resources mapping="/views/**/*.html" location="/static/views/" cache-period="21600"/>
    <mvc:resources mapping="/fonts/**/*" location="/static/fonts/"/>
    <mvc:resources mapping="/ueditor/**/*" location="/static/js/lib/ueditor/"/>
    <mvc:resources mapping="/img/**/*" location="/static/img/"/>

    <bean id="dataSource" class="org.apache.commons.dbcp2.BasicDataSource" destroy-method="close">
        <property name="driverClassName" value="${jdbc.driver}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
        <property name="validationQuery" value="${jdbc.validationQuery}"/>
        <property name="maxTotal" value="10"/>
        <property name="minIdle" value="5"/>
        <property name="maxIdle" value="10"/>
        <property name="defaultAutoCommit" value="true"/>
        <property name="testWhileIdle" value="true"/>
        <property name="testOnBorrow" value="true"/>
        <property name="poolPreparedStatements" value="true"/>
        <property name="maxOpenPreparedStatements" value="50"/>
    </bean>

    <bean id="configService" class="com.qgd.oms.web.common.service.ConfigService">
        <property name="configStore">
            <bean class="com.qgd.oms.web.common.service.impl.DbConfigStore">
                <property name="dataSource" ref="dataSource"/>
                    <property name="taskScheduler" ref="defaultScheduler"/>
                <property name="refreshInterval" value="30000"/>
            </bean>
        </property>
    </bean>
```

若需添加其它功能，如security，则需添加对应配置：



```xml
    <http pattern="/css/**/*" security="none"/>
    <http pattern="/js/**/*" security="none"/>
    <http pattern="/views/**/*.html" security="none"/>
    <http pattern="/fonts/**/*" security="none"/>
    <http pattern="/ueditor/**/*" security="none"/>
    <http pattern="/img/**/*" security="none"/>

    <http use-expressions="true" entry-point-ref="omsAuthenticationEntryPoint">
        <logout logout-url="/omsmc/authentication/logout/*" success-handler-ref="omsLogoutSuccessHandler"></logout>
        <intercept-url pattern='/omsmc/authentication/login*' access="permitAll" />
        <intercept-url pattern='/ms/**/*' access="permitAll" />
        <intercept-url pattern='/**' access="authenticated" />
        <!--<security:form-login />-->
        <custom-filter ref="omsUsernamePasswordAuthenticationFilter" position="FORM_LOGIN_FILTER" />
        <remember-me services-ref="omsRememberMeServices" key="yfboms"/>
        <csrf disabled="true"/>
    </http>
```

增加业务代码，如controller，service，model等，最后生成war包，通过容器进行启动

# Spring

## 概览

### 什么是 Spring 框架？

Spring 框架是一个为 Java 应用程序的开发提供了综合、广泛的基础性支持的 Java 平台。Spring帮助开发者解决了开发中基础性的问题，使得开发人员可以专注于应用程序的开发。

Spring 框架本身亦是按照设计模式精心打造，这使得我们可以在开发环境中安心的集成 Spring 框架，不必担心 Spring 是如何在后台进行工作的。

Spring 框架至今已集成了 20 多个模块。这些模块主要被分如下图所示的核心容器、数据访问/集成,、Web、AOP（面向切面编程）、工具、消息和测试模块。

 它是一个一站式（full-stack全栈式）框架，提供了从表现层-springMVC 到 业务层-spring 再到 持久层-springdata 的一套完整的解决方案。我们在项目中可以只使用spring一个框架，它就可以提供表现层的mvc框架，持久层的Dao框架。它的两大核心IoC和AOP更是为我们程序解耦和代码简洁易维护提供了支持。

Spring是一个轻量级的IoC和AOP容器框架。是为Java应用程序提供基础性服务的一套框架，目的是用于简化企业应用程序的开发，它使得开发者只需要关心业务需求。常见的配置方式有三种：基于XML的配置、基于注解的配置、基于Java的配置。

### 主要模块？

主要由以下几个模块组成：

Spring Core：核心类库，提供IOC服务；

Spring Context：提供框架式的Bean访问方式，以及企业级功能（JNDI、定时任务等）；

Spring AOP：AOP服务；

Spring DAO：对JDBC的抽象，简化了数据访问异常的处理；

Spring ORM：对现有的ORM框架的支持；

Spring Web：提供了基本的面向Web的综合特性，例如多方文件上传；

Spring MVC：提供面向Web应用的Model-View-Controller实现。

#### Spring核心类

答：BeanFactory：产生一个新的实例，可以实现单例模式

BeanWrapper：提供统一的get及set方法

ApplicationContext:提供框架的实现，包括BeanFactory的所有功能

#### 优点

1、Dependency Injection(DI) 方法使得构造器和 JavaBean properties 文件中的依赖关系一目了然。
2、与 EJB 容器相比较，IOC 容器更加趋向于轻量级。这样一来 IOC 容器在有限的内存和 CPU资源的情况下进行应用程序的开发和发布就变得十分有利。
3、Spring 并没有闭门造车，Spring 利用了已有的技术比如 ORM 框架、logging 框架、J2EE、Quartz 和 JDK Timer，以及其他视图技术。
4、Spring 框架是按照模块的形式来组织的。由包和类的编号就可以看出其所属的模块，开发者仅仅需要选用他们需要的模块即可。
5、要测试一项用 Spring 开发的应用程序十分简单，因为测试相关的环境代码都已经囊括在框架中了。更加简单的是，利用 JavaBean 形式的 POJO 类，可以很方便的利用依赖注入来写入测试数据。
6、Spring 的 Web 框架亦是一个精心设计的 Web MVC 框架，为开发者们在 web 框架的选择上提供了一个除了主流框架比如 Struts、过度设计的、不流行 web 框架的以外的有力选项。
7、Spring 提供了一个便捷的事务管理接口，适用于小型的本地事务处理（比如在单 DB 的环境下）和复杂的共同事务处理（比如利用 JTA 的复杂 DB 环境）。

答：1.降低了组件之间的耦合性 ，实现了软件各层之间的解耦 
2.可以使用容易提供的众多服务，如事务管理，消息服务等 
3.容器提供单例模式支持 
4.容器提供了AOP技术，利用它很容易实现如权限拦截，运行期监控等功能 
5.容器提供了众多的辅助类，能加快应用的开发 
6.spring对于主流的应用框架提供了集成支持，如hibernate，JPA，Struts等 
7.spring属于低侵入式设计，代码的污染极低 
8.独立于各种应用服务器 
9.spring的DI机制降低了业务对象替换的复杂性 
10.Spring的高度开放性，并不强制应用完全依赖于Spring，开发者可以自由选择spring 的部分或全部 

（1）spring属于低侵入式设计，代码的污染极低；

（2）spring的DI机制将对象之间的依赖关系交由框架处理，减低组件的耦合性；

（3）Spring提供了AOP技术，支持将一些通用任务，如安全、事务、日志、权限等进行集中式管理，从而提供更好的复用。

（4）spring对于主流的应用框架提供了集成支持。

### 控制反转(IOC)

> 传统Java SE程序设计，我们直接在对象内部通过new进行创建对象，是程序主动去创建依赖对象；而IoC是有专门一个容器来创建这些对象，即由Ioc容器来控制对象的创建。而绑定的过程是通过“依赖注入”实现的。

**Ioc—Inversion of Control，即“控制反转”，不是什么技术，而是一种设计思想。**在Java开发中，**Ioc意味着将你设计好的对象交给容器控制，而不是传统的在你的对象内部直接控制。**

- **控制：**就是对象的创建、初始化、销毁。创建对象，原来是 new 一个，现在给 Spring 容器创建了；对象初始化，比如 A 依赖 B，原来是我们通过构造器或者 setter 方法赋值，现在给 Spring 容器自动注入了；销毁对象，原来是我们直接赋值 null 或者做一些销毁操作，现在给 Spring 容器管理生命周期负责销毁。IOC 解决了繁琐的对象生命周期的操作，解耦了我们的代码。
- **反转：**有反转就有正转，传统应用程序是由我们自己在对象中主动控制去直接获取依赖对象，也就是正转；而反转则是由容器来帮忙创建及注入依赖对象；为何是反转？**因为由容器帮我们查找及注入依赖对象，对象只是被动的接受依赖对象，所以是反转；哪些方面反转了？依赖对象的获取被反转了。**

#### IOC好处

首先来看一个实际开发中的典型应用场景，假设我们有一个基于MVC分层结构的应用，通过controller层对外提供接口，而通过service层提供具体的实现，在service层中有一个`WelcomeService`服务接口，一般情况下都是通过`WelcomeService service = new WelcomeServiceImpl();`创建实例并进行调用：

```java
public class WelcomeController {
    private WelcomeService service = new WelcomeServiceImpl();

    @RequestMapping("/welcome")
    public String welcome() {
    return service.retrieveWelcomeMessage();
    }
}
```

调用后发现一切正常，此时，功能提交，需要进行测试，而由于实际应用环境与测试环境有所区别，需要替换`WelcomeServiceImpl`为一个`MockWelcomeServiceImpl`，以方便测试，怎么办？没有其他办法，只有改代码：

```cpp
public class WelcomeController {
    private WelcomeService service = new MockWelcomeServiceImpl();

    ...
}
```

测试OK后再将代码改回去，这种方式太过于繁琐，且对代码的侵入性很强；
 下面看通过Spring的IOC如何实现，首先将`WelcomeService`交由Spring管理：

```csharp
<bean name="WelcomeService" class="XXX.XXX.XXX.service.impl.WelcomeServiceImpl"/>
```

然后在业务代码处通过Spring IOC拿到具体对象：

```kotlin
public class WelcomeController {
    @Autowired
    private WelcomeService service;

    @RequestMapping("/welcome")
    public String welcome() {
        return service.retrieveWelcomeMessage();
    }
}
```

测试的时候，只需要更改配置文件，将`WelcomeService`对应的实现改为`MockWelcomeServiceImpl`即可：



```csharp
<bean name="WelcomeService" class="XXX.XXX.XXX.service.impl.MockWelcomeServiceImpl"/>
```

这种方式对业务代码没有任何侵入，**它有效的实现松耦合**，大家都知道紧耦合的代码是业务发展的噩梦；同时，Spring IOC提供的远不止这些，如通过单例减少创建无用的对象，通过延迟加载优化初始化成本等

### 依赖注入（DI）

#### DI的好处

依赖注入降低了开发的成本、提高了代码的复用率、提高了软件的灵活性，并为系统搭建一个灵活、可扩展的平台。

- **谁依赖于谁：**当然是**应用程序依赖于IoC容器**；
- **为什么需要依赖：应用程序需要IoC容器来提供对象需要的外部资源**；
- **谁注入谁：**很明显是**IoC容器注入应用程序某个对象，应用程序依赖的对象**；
- **注入了什么：**就是**注入某个对象所需要的外部资源（包括对象、资源、常量数据）**。

#### 依赖注入三种方式

1. 构造器注入
2. Setter 方法注入
3. 接口注入

## AOP

OOP面向对象，允许开发者定义纵向的关系，但并适用于定义横向的关系，导致了大量代码的重复，而不利于各个模块的重用。

AOP，一般称为面向切面，作为面向对象的一种补充，用于将那些与业务无关，但却对多个对象产生影响的公共行为和逻辑，抽取并封装为一个可重用的模块，这个模块被命名为“切面”（Aspect），减少系统中的重复代码，降低了模块间的耦合度，同时提高了系统的可维护性。可用于权限认证、日志、事务处理。

使用场景:

Authentication 权限、Caching 缓存、Context passing 内容传递、Error handling 错误处理Lazy loading懒加载、Debugging调试、logging, tracing, profiling and monitoring 记录跟踪优化　校准、Performance optimization　性能优化、Persistence 持久化、Resource pooling　资源池、Synchronization　同步、Transactions 事务

#### 优点

1. 各个步骤之间的良好隔离性耦合性大大降低 

2. 源代码无关性，再扩展功能的同时不对源码进行修改操作 

#### 原理

AOP是面向切面编程，是通过动态代理的方式为程序添加统一功能，集中解决一些公共问题。

### 静态代理

AOP实现的关键在于 代理模式，AOP代理主要分为静态代理和动态代理。静态代理的代表为AspectJ；动态代理则以Spring AOP为代表。

（1）AspectJ是静态代理的增强，所谓静态代理，就是AOP框架会在编译阶段生成AOP代理类，因此也称为编译时增强，他会在编译阶段将AspectJ(切面)织入到Java字节码中，运行的时候就是增强之后的AOP对象。

### Spring AOP中的动态代理

> Spring AOP使用的动态代理，所谓的动态代理就是说AOP框架不会去修改字节码，而是每次运行时在内存中临时为方法生成一个AOP对象，这个AOP对象包含了目标对象的全部方法，并且在特定的切点做了增强处理，并回调原对象的方法。

#### JDK动态代理

JDK动态代理只提供接口的代理，不支持类的代理。核心InvocationHandler接口和Proxy类，InvocationHandler 通过invoke()方法反射来调用目标类中的代码，动态地将横切逻辑和业务编织在一起；接着，Proxy利用 InvocationHandler动态创建一个符合某一接口的的实例, 生成目标类的代理对象。

若目标对象实现了若干接口，spring使用JDK的java.lang.reflect.Proxy类代理。

   优点：因为有接口，所以使系统更加松耦合

   缺点：为每一个目标类创建接口

#### CGLIB动态代理

如果代理类没有实现 InvocationHandler 接口，那么Spring AOP会选择使用CGLIB来动态代理目标类。CGLIB（Code Generation Library），是一个代码生成的类库，可以在运行时动态的生成指定类的一个子类对象，并覆盖其中特定方法并添加增强代码，从而实现AOP。CGLIB是通过继承的方式做的动态代理，因此如果某个类被标记为final，那么它是无法使用CGLIB做动态代理的。

若目标对象没有实现任何接口，spring使用CGLIB库生成目标对象的子类。

   优点：因为代理类与目标类是继承关系，所以不需要有接口的存在。

   缺点：因为没有使用接口，所以系统的耦合性没有使用JDK的动态代理好。

#### 静态代理与动态代理区别

在于生成AOP代理对象的时机不同，相对来说AspectJ的静态代理方式具有更好的性能，但是AspectJ需要特定的编译器进行处理，而Spring AOP则无需特定的编译器处理。

> InvocationHandler 的 invoke(Object proxy,Method method,Object[] args)：proxy是最终生成的代理实例; method 是被代理目标实例的某个具体方法; args 是被代理目标实例某个方法的具体入参, 在方法反射调用时使用。 

### Spring 配置三种方式：

1. 基于 XML 的配置
2. 基于注解的配置
3. 基于 Java 的配置

#### 使用 XML 配置

在 Spring 框架中，依赖和服务需要在专门的配置文件来实现，我常用的 XML 格式的配置文件。
这些配置文件的格式通常用开头，然后一系列的 bean 定义和专门的应用配置选项组成。
SpringXML配置的主要目的时候是使所有的Spring组件都可以用xml文件的形式来进行配置。
这意味着不会出现其他的 Spring 配置类型（比如声明的方式或基于 Java Class 的配置方式）Spring 的 XML 配置方式是使用被 Spring 命名空间的所支持的一系列的 XML 标签来实现的。
Spring 有以下主要的命名空间：context、beans、jdbc、tx、aop、mvc 和 aso。

```xml
<beans>
    <bean name="viewResolver"
    		class="org.springframework.web.servlet.view.BeanNameViewResolver"/>
    <bean name="jsonTemplate"
    		class="org.springframework.web.servlet.view.json.MappingJackson2JsonView"/>
    <bean id="restTemplate" class="org.springframework.web.client.RestTemplate"/>
</beans>
```

#### 基于 Java 配置

Spring 对 Java 配置的支持是由@Configuration 注解和@Bean 注解来实现的。由@Bean 注解的方法将会实例化、配置和初始化一个新对象，这个对象将由 Spring 的 IOC 容器来管理。
@Bean 声明所起到的作用与 元素类似。被@Configuration 所注解的类则表示这个类的主要目的是作为 bean 定义的资源。被@Configuration 声明的类可以通过在同一个类的内部调用@bean 方法来设置嵌入 bean 的依赖关系。
最简单的@Configuration 声明类请参考下面的代码： 

```java
@Configuration
public class AppConfig{
    @Bean
    public MyService myService() {
      	return new MyServiceImpl();
    }
}
```

对于上面的@Beans 配置文件相同的 XML 配置文件如下：

```xml
<beans>
		<bean id="myService" class="com.gupaoedu.services.MyServiceImpl"/>
</beans>
```

上述配置方式的实例化方式如下：利用AnnotationConfigApplicationContext 类进行实例化

```java
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
    MyService myService = ctx.getBean(MyService.class);
    myService.doStuff();
}
```

要使用组件组建扫描，仅需用@Configuration 进行注解即可：

```java
@Configuration
@ComponentScan(basePackages = "com.gupaoedu")
public class AppConfig {
}
```

在上面的例子中，com.gupaoedu 包首先会被扫到，然后再容器内查找被@Component 声明的类，找到后将这些类按照 Sring bean 定义进行注册。如 果 你 要 在 你 的 web 应 用 开 发 中 选 用 上 述 的 配 置 的 方 式 的 话 ， 需 要 用AnnotationConfigWebApplicationContext 类来读取配置文件，可以用来配置 Spring 的Servlet 监听器 ContrextLoaderListener 或者 Spring MVC 的 DispatcherServlet。

```xml
<web-app>
<!-- Configure ContextLoaderListener to use AnnotationConfigWebApplicationContext
instead of the default XmlWebApplicationContext -->
<context-param>
<param-name>contextClass</param-name>
<param-value>
org.springframework.web.context.support.AnnotationConfigWebApplicationContext
</param-value>
</context-param>
<!-- Configuration locations must consist of one or more comma- or space-delimited
fully-qualified @Configuration classes. Fully-qualified packages may also be
specified for component-scanning -->
<context-param>
<param-name>contextConfigLocation</param-name>
<param-value>com.gupaoedu.AppConfig</param-value>
</context-param>
<!-- Bootstrap the root application context as usual using ContextLoaderListener -->
<listener>
<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
<!-- Declare a Spring MVC DispatcherServlet as usual -->
<servlet>
<servlet-name>dispatcher</servlet-name>
<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
<!-- Configure DispatcherServlet to use AnnotationConfigWebApplicationContext
instead of the default XmlWebApplicationContext -->
<init-param>
<param-name>contextClass</param-name>
<param-value>
org.springframework.web.context.support.AnnotationConfigWebApplicationContext
</param-value>
</init-param>
<!-- Again, config locations must consist of one or more comma- or space-delimited
and fully-qualified @Configuration classes -->
<init-param>
<param-name>contextConfigLocation</param-name>
<param-value>com.gupaoedu.web.MVCConfig</param-value>
</init-param>
</servlet>
<!-- map all requests for /web/* to the dispatcher servlet -->
<servlet-mapping>
<servlet-name>dispatcher</servlet-name>
<url-pattern>/web/*</url-pattern>
</servlet-mapping></web-app>
```

#### 怎样用注解的方式配置 Spring？

Spring 在 2.5 版本以后开始支持用注解的方式来配置依赖注入。可以用注解的方式来替代 XML方式的 bean 描述，可以将 bean 描述转移到组件类的内部，只需要在相关类上、方法上或者字段声明上使用注解即可。注解注入将会被容器在 XML 注入之前被处理，所以后者会覆盖掉前者对于同一个属性的处理结果。
注解装配在 Spring 中是默认关闭的。所以需要在 Spring 文件中配置一下才能使用基于注解的装配模式。如果你想要在你的应用程序中使用关于注解的方法的话，请参考如下的配置。

```xml
<beans>
<context:annotation-config/>
<!-- bean definitions go here -->
</beans>
```

在标签配置完成以后，就可以用注解的方式在 Spring 中向属性、方法和构造方法中自动装配变
量。
下面是几种比较重要的注解类型：
1.@Required：该注解应用于设值方法。
2.@Autowired：该注解应用于有值设值方法、非设值方法、构造方法和变量。
3.@Qualifier：该注解和@Autowired 注解搭配使用，用于消除特定 bean 自动装配的歧义。
4.JSR-250 Annotations ：Spring 支持 基于 JSR-250 注解 的以下 注解 ，@Resource、
@PostConstruct 和 @PreDestroy。

## Bean

###  Bean 加载原理

加载过程： 通过 `ResourceLoader`和其子类`DefaultResourceLoader`完成资源文件位置定位，实现从类路径，文件系统，url等方式定位功能，完成定位后得到`Resource`对象，再交给`BeanDefinitionReader`，它再委托给`BeanDefinitionParserDelegate`完成bean的解析并得到`BeanDefinition`对象，然后通过`registerBeanDefinition`方法进行注册，IOC容器内ibu维护了一个HashMap来保存该`BeanDefinition`对象，Spring中的`BeanDefinition`其实就是我们用的`JavaBean`。

### Bean 的生命周期？

Spring Bean的完整生命周期从创建Spring容器开始，直到最终Spring容器销毁Bean

#### 1. 实例化Bean

对于BeanFactory容器，当客户向容器请求一个尚未初始化的bean时，或初始化bean的时候需要注入另一个尚未初始化的依赖时，容器就会调用createBean进行实例化。 
对于ApplicationContext容器，当容器启动结束后，便实例化所有的bean。 
容器通过获取BeanDefinition对象中的信息进行实例化。并且这一步仅仅是简单的实例化，并未进行依赖注入。 
实例化对象被包装在BeanWrapper对象中，BeanWrapper提供了设置对象属性的接口，**从而避免了使用反射机制设置属性**。

#### 2. 设置对象属性（依赖注入）

实例化后的对象被封装在BeanWrapper对象中，并且此时对象仍然是一个原生的状态，并没有进行依赖注入。 
紧接着，Spring根据BeanDefinition中的信息进行依赖注入。 
并且通过BeanWrapper提供的设置属性的接口完成依赖注入。

#### 3. 注入Aware接口

紧接着，Spring会检测该对象是否实现了xxxAware接口，并将相关的xxxAware实例注入给bean。

#### 4. BeanPostProcessor

当经过上述几个步骤后，bean对象已经被正确构造，但如果你想要对象被使用前再进行一些自定义的处理，就可以通过BeanPostProcessor接口实现。 
该接口提供了两个函数：

- postProcessBeforeInitialzation( Object bean, String beanName ) 
  当前正在初始化的bean对象会被传递进来，我们就可以对这个bean作任何处理。 
  这个函数会先于InitialzationBean执行，因此称为前置处理。 
  所有Aware接口的注入就是在这一步完成的。
- postProcessAfterInitialzation( Object bean, String beanName ) 
  当前正在初始化的bean对象会被传递进来，我们就可以对这个bean作任何处理。 
  这个函数会在InitialzationBean完成后执行，因此称为后置处理。

#### 5. InitializingBean与init-method

当BeanPostProcessor的前置处理完成后就会进入本阶段。 
InitializingBean接口只有一个函数：

- afterPropertiesSet()

这一阶段也可以在bean正式构造完成前增加我们自定义的逻辑，但它与前置处理不同，由于该函数并不会把当前bean对象传进来，因此在这一步没办法处理对象本身，只能增加一些额外的逻辑。 
若要使用它，我们需要让bean实现该接口，并把要增加的逻辑写在该函数中。然后Spring会在前置处理完成后检测当前bean是否实现了该接口，并执行afterPropertiesSet函数。

当然，Spring为了降低对客户代码的侵入性，给bean的配置提供了init-method属性，该属性指定了在这一阶段需要执行的函数名。Spring便会在初始化阶段执行我们设置的函数。init-method本质上仍然使用了InitializingBean接口。

#### 6. DisposableBean和destroy-method

和init-method一样，通过给destroy-method指定函数，就可以在bean销毁前执行指定的逻辑。

![img](https://pic1.zhimg.com/80/v2-baaf7d50702f6d0935820b9415ff364c_1440w.jpg)

![img](https://images0.cnblogs.com/i/580631/201405/181453414212066.png)![img](https://images2018.cnblogs.com/blog/717817/201805/717817-20180522141553606-1691095215.png)

Spring Bean 的生命周期简单易懂。在一个 bean 实例被初始化时，需要执行一系列的初始化操作以达到可用的状态。同样的，当一个 bean 不在被调用时需要进行相关的析构操作，并从 bean容器中移除。
Spring bean factory 负责管理在 spring 容器中被创建的 bean 的生命周期。Bean 的生命周期由两组回调（call back）方法组成。
1.初始化之后调用的回调方法。
2.销毁之前调用的回调方法。
Spring 框架提供了以下四种方式来管理 bean 的生命周期事件：
1、InitializingBean 和 DisposableBean 回调接口
2、针对特殊行为的其他 Aware 接口
3、Bean 配置文件中的 Custom init()方法和 destroy()方法
4、@PostConstruct 和@PreDestroy 注解方式
使用 customInit()和 customDestroy()方法管理 bean 生命周期的代码样例如下：

```xml
<beans>
<bean id="demoBean" class="com.gupaoedu.task.DemoBean"
init-method="customInit" destroy-method="customDestroy">
</bean>
</beans>
```

### Bean的调用方式

答：有三种方式可以得到Bean并进行调用：
1、使用BeanWrapper

```java
HelloWorld hw=new HelloWorld();
BeanWrapper bw=new BeanWrapperImpl(hw);
bw.setPropertyvalue(”msg”,”HelloWorld”);
system.out.println(bw.getPropertyCalue(”msg”));
```

2、使用BeanFactory

```java
InputStream is=new FileInputStream(”config.xml”);
XmlBeanFactory factory=new XmlBeanFactory(is);
HelloWorld hw=(HelloWorld) factory.getBean(”HelloWorld”);
system.out.println(hw.getMsg());
```

3、使用ApplicationConttext

```java
ApplicationContext actx=new FleSystemXmlApplicationContext(”config.xml”);
HelloWorld hw=(HelloWorld) actx.getBean(”HelloWorld”);
System.out.println(hw.getMsg());
```

### Bean的注解

答：@Component@Controller@ Service@ Repository

### BeanFactory和FactoryBean的区别

####  BeanFactory

BeanFactory是IOC最基本的容器，负责生产和管理bean，它为其他具体的IOC容器提供了最基本的规范，例如DefaultListableBeanFactory，XmlBeanFactory,ApplicationContext 等具体的容器都是实现了BeanFactory，再在其基础之上附加了其他的功能。

BeanFactory，以Factory结尾，表示它是一个工厂类(接口)， **它负责生产和管理bean的一个工厂**。在Spring中，**BeanFactory是IOC容器的核心接口，它的职责包括：实例化、定位、配置应用程序中的对象及建立这些对象间的依赖。BeanFactory只是个接口，并不是IOC容器的具体实现，但是Spring容器给出了很多种实现，如 DefaultListableBeanFactory、XmlBeanFactory、ApplicationContext等，其中****XmlBeanFactory就是常用的一个，该实现将以XML方式描述组成应用的对象及对象间的依赖关系**。XmlBeanFactory类将持有此XML配置元数据，并用它来构建一个完全可配置的系统或应用。  

 都是附加了某种功能的实现。 它为其他具体的IOC容器提供了最基本的规范，例如DefaultListableBeanFactory,XmlBeanFactory,ApplicationContext 等具体的容器都是实现了BeanFactory，再在其基础之上附加了其他的功能。

**BeanFactory方法**

- boolean containsBean(String beanName) 判断工厂中是否包含给定名称的bean定义，若有则返回true
- Object getBean(String) 返回给定名称注册的bean实例。根据bean的配置情况，如果是singleton模式将返回一个共享实例，否则将返回一个新建的实例，如果没有找到指定bean,该方法可能会抛出异常
- Object getBean(String, Class) 返回以给定名称注册的bean实例，并转换为给定class类型
- Class getType(String name) 返回给定名称的bean的Class,如果没有找到指定的bean实例，则排除NoSuchBeanDefinitionException异常
- boolean isSingleton(String) 判断给定名称的bean定义是否为单例模式
- String[] getAliases(String name) 返回给定bean名称的所有别名 

 BeanFactory源码          

```java
package org.springframework.beans.factory;  
import org.springframework.beans.BeansException;  
public interface BeanFactory {  
    String FACTORY_BEAN_PREFIX = "&";  
    Object getBean(String name) throws BeansException;  
    <T> T getBean(String name, Class<T> requiredType) throws BeansException;  
    <T> T getBean(Class<T> requiredType) throws BeansException;  
    Object getBean(String name, Object... args) throws BeansException;  
    boolean containsBean(String name);  
    boolean isSingleton(String name) throws NoSuchBeanDefinitionException;  
    boolean isPrototype(String name) throws NoSuchBeanDefinitionException;  
    boolean isTypeMatch(String name, Class<?> targetType) throws NoSuchBeanDefinitionException;  
    Class<?> getType(String name) throws NoSuchBeanDefinitionException;  
    String[] getAliases(String name);  
}  
```

##### ApplicationContext

BeanFactory和ApplicationContext就是spring框架的两个IOC容器，现在一般使用ApplicationnContext，其不但包含了BeanFactory的作用，同时还进行更多的扩展。 

**BeanFacotry是spring中比较原始的Factory。如XMLBeanFactory就是一种典型的BeanFactory。**
原始的BeanFactory无法支持spring的许多插件，**如AOP功能、Web应用等**。ApplicationContext接口,它由BeanFactory接口派生而来， 

ApplicationContext包含BeanFactory的所有功能，通常建议比BeanFactory优先 

ApplicationContext以一种更向面向框架的方式工作以及对上下文进行分层和实现继承，ApplicationContext包还提供了以下的功能： 
• MessageSource, 提供国际化的消息访问 
• 资源访问，如URL和文件 
• 事件传播 
• 载入多个（有继承关系）上下文 ，使得每一个上下文都专注于一个特定的层次，比如应用的web层; 

在不使用spring框架之前，我们的service层中要使用dao层的对象，不得不在service层中new一个对象。存在的问题：层与层之间的依赖。

service层要用dao层对象需要配置到xml配置文件中，至于对象是怎么创建的，关系是怎么组合的都交给了spring框架去实现。 

```java
Resource resource = new FileSystemResource("beans.xml");
2 BeanFactory factory = new XmlBeanFactory(resource);
1 ClassPathResource resource = new ClassPathResource("beans.xml");
2 BeanFactory factory = new XmlBeanFactory(resource);
1 ApplicationContext context = new ClassPathXmlApplicationContext(new String[] {"applicationContext.xml", "applicationContext-part2.xml"});
3 BeanFactory factory = (BeanFactory) context;
```

 基本就是这些了，接着使用getBean(String beanName)方法就可以取得bean的实例；

#### FactoryBean

FactoryBean是一个接口，当在IOC容器中的Bean实现了FactoryBean后，通过getBean(String BeanName)获取到的Bean对象并不是FactoryBean的实现类对象，而是这个实现类中的getObject()方法返回的对象。要想获取FactoryBean的实现类，就要getBean(&BeanName)，在BeanName之前加上&。

**一般情况下，Spring通过反射机制利用的class属性指定实现类实例化Bean，在某些情况下，实例化Bean过程比较复杂，如果按照传统的方式，则需要在Spring中提供大量的配置信息。配置方式的灵活性是受限的，这时采用编码的方式可能会得到一个简单的方案。Spring为此提供了一个org.springframework.bean.factory.FactoryBean的工厂类接口，用户可以通过实现该接口定制实例化Bean的逻辑。FactoryBean接口对于Spring框架来说占用重要的地位，Spring自身就提供了70多个FactoryBean的实现**。它们隐藏了实例化一些复杂Bean的细节，给上层应用带来了便利。从Spring3.0开始，FactoryBean开始支持泛型，即接口声明改为FactoryBean<T>的形式

以Bean结尾，表示它是一个Bean，不同于普通Bean的是：它是实现了FactoryBean<T>接口的Bean，根据该Bean的ID从BeanFactory中获取的实际上是FactoryBean的getObject()返回的对象，而不是FactoryBean本身，如果要获取FactoryBean对象，请在id前面加一个&符号来获取。

　　**例如自己实现一个FactoryBean，功能：用来代理一个对象，对该对象的所有方法做一个拦截，在调用前后都输出一行LOG，模仿ProxyFactoryBean的功能**。 

 FactoryBean源码

```java
package org.springframework.beans.factory;  
public interface FactoryBean<T> {  
    T getObject() throws Exception;  
    Class<?> getObjectType();  
    boolean isSingleton();  
}
```

#### BeanFactory和FactoryBean的区别

BeanFactory和FactoryBean其实没有什么比较性的，只是两者的名称特别接近，所以有时候会拿出来比较一番，BeanFactory是提供了IOC容器最基本的形式，给具体的IOC容器的实现提供了规范，FactoryBean可以说为IOC容器中Bean的实现提供了更加灵活的方式，FactoryBean在IOC容器的基础上给Bean的实现加上了一个简单工厂模式和装饰模式，我们可以在getObject()方法中灵活配置。

BeanFactory是个Factory，也就是IOC容器或对象工厂，FactoryBean是个Bean。在Spring中，**所有的Bean都是由BeanFactory(也就是IOC容器)来进行管理的**。但对FactoryBean而言，**这个Bean不是简单的Bean，而是一个能生产或者修饰对象生成的工厂Bean,它的实现与设计模式中的工厂模式和修饰器模式类似** 

#### BeanFactory 和 ApplicationContext 

BeanFactory 可以理解为含有 bean 集合的工厂类。

- BeanFactory 包含了种 bean 的定义，以便在接收到客户端请求时将对应的 bean 实例化。
- 还能在实例化对象的时生成协作类之间的关系。此举将 bean 自身与 bean 客户端的配置中解放出来。

- BeanFactory 还包含了 bean 生命周期的控制，调用客户端的初始化方法（initialization methods）和销毁方法（destruction methods）。

ApplicationContext

- 从表面上看，application context 如同 bean factory 一样具有 bean 定义、bean 关联关系的设置，根据请求分发 bean 的功能。但 application context 在此基础上还提供了其他的功能。

1. 提供了支持国际化的文本消息
2. 统一的资源文件读取方式
3. 已在监听器中注册的 bean 的事件

三种较常见的 ApplicationContext 实现方式：
1、ClassPathXmlApplicationContext：从 classpath 的 XML 配置文件中读取上下文，并生成上下文定义。应用程序上下文从程序环境变量中取得。
ApplicationContext context = new ClassPathXmlApplicationContext(“application.xml”);
2、FileSystemXmlApplicationContext ：由文件系统中的 XML 配置文件读取上下文。ApplicationContext context = new FileSystemXmlApplicationContext(“application.xml”);
3、XmlWebApplicationContext：由 Web 应用的 XML 文件读取上下文。

### 作用域之间的区别？

Spring 容器中的 bean 可以分为 5 个范围。所有范围的名称都是自说明的，但是为了避免混淆，
还是让我们来解释一下：
1.singleton：这种 bean 范围是默认的，这种范围确保不管接受到多少个请求，每个容器中只有一个 bean 的实例，单例的模式由 bean factory 自身来维护。
2.prototype：原形范围与单例范围相反，为每一个 bean 请求提供一个实例。
3.request：在请求 bean 范围内会每一个来自客户端的网络请求创建一个实例，在请求完成以后，bean 会失效并被垃圾回收器回收。
4.Session：与请求范围类似，确保每个 session 中有一个 bean 的实例，在 session 过期后，bean 会随之失效。更多面试资料在群619881427免费获取（JVM/并发编程/分布式/微服务/等面试视频学习资料都可以免费获取）
5.global-session：global-session 和 Portlet 应用相关。当你的应用部署在 Portlet 容器中作时，它包含很多 portlet。如果你想要声明让所有的 portlet 共用全局的存储变量的话，那么这全局变量需要存储在 global-session 中。全局作用域与 Servlet 中的 session 作用域效果相同。

### Spring inner beans？

在 Spring 框架中，无论何时 bean 被使用时，当仅被调用了一个属性。一个明智的做法是将这个 bean 声明为内部 bean。内部 bean 可以用 setter 注入“属性”和构造方法注入“构造参数”的方式来实现。
比如，在我们的应用程序中，一个 Customer 类引用了一个 Person 类，我们的要做的是创建一个 Person 的实例，然后在 Customer 内部使用。

```java
public class Customer{
    private Person person;
    //Setters and Getters
}
public class Person{
    private String name;
    private String address;
    private int age;
    //Setters and Getters
}
```

内部 bean 的声明方式如下： 

```xml
<bean id="CustomerBean" class="com.howtodoinjava.common.Customer">
<property name="person">
		<!-- This is inner bean -->
    <bean class="com.howtodoinjava.common.Person">
        <property name="name" value="lokesh" />
        <property name="address" value="India" />
        <property name="age" value="34" />
    </bean>
</property>
</bean>
```

### 单例 Beans 是线程安全的么？

Spring 框架并没有对单例 bean 进行任何多线程的封装处理。关于单例 bean 的线程安全和并发问题需要开发者自行去搞定。但实际上，大部分的 Spring bean 并没有可变的状态(比如Serview类和DAO类)，所以在某种程度上说Spring的单例bean是线程安全的。如果你的bean有多种状态的话（比如 View Model 对象），就需要自行保证线程安全。
最浅显的解决办法就是将多态 bean 的作用域由“singleton”变更为“prototype”。

### 在 Bean 中注入一个 Java 集合？

Spring 提供了以下四种集合类的配置元素：
1、该标签用来装配可重复的 list 值。
2、该标签用来装配没有重复的 set 值。
3、该标签可用来注入键和值可以为任何类型的键值对。
4、该标签支持注入键和值都是字符串类型的键值对。
下面看一下具体的例子： 

```xml
<beans>
<!-- Definition for javaCollection -->
<bean id="javaCollection" class="com.gupaoedu.JavaCollection">
<!-- java.util.List -->
<property name="customList">
    <list>
      <value>INDIA</value>
      <value>Pakistan</value>
      <value>USA</value>
      <value>UK</value>
    </list>
</property>
  
<!-- java.util.Set -->
<property name="customSet">
    <set>
        <value>INDIA</value>
        <value>Pakistan</value>
        <value>USA</value>
        <value>UK</value>
    </set>
</property>
  
<!-- java.util.Map -->
<property name="customMap">
    <map>
        <entry key="1" value="INDIA"/>
        <entry key="2" value="Pakistan"/>
        <entry key="3" value="USA"/>
        <entry key="4" value="UK"/>
    </map>
</property>
  
<!-- java.util.Properties -->
<property name="customProperies">
    <props>
        <prop key="admin">admin@gupaoedu.com</prop>
        <prop key="support">support@gupaoedu.com</prop>
    </props>
</property>
</bean>
</beans>
```

### 向Bean 中注入 java.util.Properties？

第一种方法是使用如下面代码所示的 标签：

```xml
<bean id="adminUser" class="com.howtodoinjava.common.Customer">
<!-- java.util.Properties -->
<property name="emails">
    <props>
        <prop key="admin">admin@gupaoedu.com</prop>
        <prop key="support">support@gupaoedu.com</prop>
    </props>
</property>
</bean>
```

也可用”util:”命名空间来从 properties 文件中创建出一个 propertiesbean，然后利用 setter方法注入 bean 的引用。

### Spring Bean 的自动装配？

在 Spring 框架中，在配置文件中设定 bean 的依赖关系是一个很好的机制，Spring 容器还可以自动装配合作关系 bean 之间的关联关系。这意味着 Spring 可以通过向 Bean Factory 中注入的方式自动搞定 bean 之间的依赖关系。自动装配可以设置在每个 bean 上，也可以设定在特定的 bean 上。
下面的 XML 配置文件表明了如何根据名称将一个 bean 设置为自动装配：

```java
<bean id="employeeDAO" class="com.gupaoedu.EmployeeDAOImpl" autowire="byName" />
```

除了 bean 配置文件中提供的自动装配模式，还可以使用@Autowired 注解来自动装配指定的bean。在使用@Autowired 注解之前需要在按照如下的配置方式在 Spring 配置文件进行配置才可以使用。
<context:annotation-config />也可以通过在配置文件中配置 AutowiredAnnotationBeanPostProcessor 达到相同的效果。

```xml
<bean class
="org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor
"/>
```

 1配置好以后就可以使用@Autowired 来标注了。

```java
@Autowired
public EmployeeDAOImpl ( EmployeeManager manager ) {
		this.manager = manager;
}
```

#### 各种自动装配模式的区别？

在 Spring 框架中共有 5 种自动装配，让我们逐一分析。

1. no：这是 Spring 框架的默认设置，在该设置下自动装配是关闭的，开发者需要自行在 bean定义中用标签明确的设置依赖关系。
2. byName：该选项可以根据 bean 名称设置依赖关系。当向一个 bean 中自动装配一个属性时，容器将根据 bean 的名称自动在在配置文件中查询一个匹配的 bean。如果找到的话，就装配这个属性，如果没找到的话就报错。
3. byType：该选项可以根据 bean 类型设置依赖关系。当向一个 bean 中自动装配一个属性时，容器将根据 bean 的类型自动在在配置文件中查询一个匹配的 bean。如果找到的话，就装配这个属性，如果没找到的话就报错。
4. constructor：造器的自动装配和 byType 模式类似，但是仅仅适用于与有构造器相同参数的bean，如果在容器中没有找到与构造器参数类型一致的bean，那么将会抛出异常。
5. autodetect：该模式自动探测使用构造器自动装配或者 byType 自动装配。首先，首先会尝试找合适的带参数的构造器，如果找到的话就是用构造器自动装配，如果在 bean 内部没有找到相应的构造器或者是无参构造器，容器就会自动选择 byTpe 的自动装配方式。

#### 开启基于注解的自动装配？

要使用 @Autowired，需要注册AutowiredAnnotationBeanPostProcessor，可以有以下两种方式来实现：
1、引入配置文件中的下引入

```
<beans>



<context:annotation-config />



</beans>
```

2、在 bean 配置文件中直接引入 AutowiredAnnotationBeanPostProcessor

```
<beans>



<bean



class="org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProc



essor"/>



</beans> 
```

#### 自动装配有哪些局限性？

自动装配有如下局限性：
重写：你仍然需要使用 和< property>设置指明依赖，这意味着总要重写自动装配。
原生数据类型:你不能自动装配简单的属性，如原生类型、字符串和类。
模糊特性：自动装配总是没有自定义装配精确，因此，如果可能尽量使用自定义装配。 

### 构造方法注入和设值注入有什么区别？

请注意以下明显的区别：

1. 在设值注入方法支持大部分的依赖注入，如果我们仅需要注入 int、string 和 long 型的变量，我们不要用设值的方法注入。对于基本类型，如果我们没有注入的话，可以为基本类型设置默认值。在构造方法注入不支持大部分的依赖注入，因为在调用构造方法中必须传入正确的构造参数，否则的话为报错。

2. 设值注入不会重写构造方法的值。如果我们对同一个变量同时使用了构造方法注入又使用了设置方法注入的话，那么构造方法将不能覆盖由设值方法注入的值。很明显，因为构造方法尽在对象被创建时调用。

3. 在使用设值注入时有可能还不能保证某种依赖是否已经被注入，也就是说这时对象的依赖关系有可能是不完整的。而在另一种情况下，构造器注入则不允许生成依赖关系不完整的对象。

4. 在 设 值 注 入 时 如 果 对 象 A 和 对 象 B 互 相 依 赖 ， 在 创 建 对 象 A 时 Spring 会 抛 出ObjectCurrentlyInCreationException 异常，因为在 B 对象被创建之前 A 对象是不能被创建的，反之亦然。所以 Spring 用设值注入的方法解决了循环依赖的问题，因对象的设值方法是在对象被创建之前被调用的。更多面试资料在群619881427免费获取（JVM/并发编程/分布式/微服务/等面试疑难解答都可以群里免费获取）。 

### 有哪些不同类型的事件？

Spring 的 ApplicationContext 提供了支持事件和代码中监听器的功能。我们可以创建 bean 用来监听在 ApplicationContext 中发布的事件。ApplicationEvent 类和在 ApplicationContext 接口中处理的事件，如果一个 bean 实现了 ApplicationListener 接口，当一个 ApplicationEvent 被发布以后，bean 会自动被通知。

####  五种标准的事件

1.上下文更新事件（ContextRefreshedEvent）：该事件会在 ApplicationContext 被初始化或者更新时发布。也可以在调用 ConfigurableApplicationContext 接口中的 refresh()方法时被触发。


2.上下文开始事件（ContextStartedEvent）：当容器调用ConfigurableApplicationContext的 Start()方法开始/重新开始容器时触发该事件。


3.上下文停止事件（ContextStoppedEvent）：当容器调用ConfigurableApplicationContext的 Stop()方法停止容器时触发该事件。


4.上下文关闭事件（ContextClosedEvent）：当 ApplicationContext 被关闭时触发该事件。容器被关闭时，其管理的所有单例 Bean 都被销毁。


5.请求处理事件（RequestHandledEvent）：在 Web 应用中，当一个 http 请求（request）结束触发该事件。
除了上面介绍的事件以外，还可以通过扩展 ApplicationEvent 类来开发自定义的事件。

```java
public class CustomApplicationEvent extends ApplicationEvent{
    public CustomApplicationEvent ( Object source, final String msg ){
        super(source);
        System.out.println("Created a Custom event");
    }
}
```

为了监听这个事件，还需要创建一个监听器：

```java
public class CustomEventListener implements ApplicationListener <
CustomApplicationEvent >{
@Override
public void onApplicationEvent(CustomApplicationEvent applicationEvent) {
    //handle event
    }
}
```

之后通过 applicationContext 接口的 publishEvent()方法来发布自定义事件。

```java
CustomApplicationEvent customEvent = new CustomApplicationEvent(applicationContext,
“Test message”);
applicationContext.publishEvent(customEvent);
```

### FileSystemResource 和 ClassPathResource 

在 FileSystemResource 中需要给出 spring-config.xml 文件在你项目中的相对路径或者绝对路径。在 ClassPathResource 中 spring 会在 ClassPath 中自动搜寻配置文件，所以要把ClassPathResource 文件放在 ClassPath 下。
如果将 spring-config.xml 保存在了 src 文件夹下的话，只需给出配置文件的名称即可，因为 src文件夹是默认。
简而言之，ClassPathResource 在环境变量中读取配置文件FileSystemResource 在配置文件中读取配置文件。

## 设计模式

Spring 框架中使用到了大量的设计模式，下面列举了比较有代表性的：
1、代理模式—在 AOP 和 remoting 中被用的比较多，AOP 思想的底层实现技术，Spring 中采用 JDK Proxy 和 CgLib 类库。 
2、单例模式：在 spring 配置文件中定义的 bean 默认为单例模式。
3、模板模式：用来解决代码重复的问题。
比如. RestTemplate, JmsTemplate, JpaTemplate。
4、委派模式：Spring 提供了 DispatcherServlet 来对请求进行分发。
5、工厂模式：BeanFactory 用来创建对象的实例，贯穿于 BeanFactory / ApplicationContext
接口的核心理念。

### **第一种：简单工厂**

又叫做静态工厂方法（StaticFactory Method）模式，但不属于23种GOF设计模式之一。 
简单工厂模式的实质是由一个工厂类根据传入的参数，动态决定应该创建哪一个产品类。 **spring中的BeanFactory就是简单工厂模式的体现**，根据传入一个唯一的标识来获得bean对象，但是否是在传入参数后创建还是传入参数前创建这个要根据具体情况来定。如下配置，就是在 HelloItxxz 类中创建一个 itxxzBean。

```xml
<beans>
  <bean id="singletonBean" class="com.itxxz.HelloItxxz">
    <constructor-arg>
      <value>Hello! 这是singletonBean!value>
    </constructor-arg>
  </ bean>

  <bean id="itxxzBean" class="com.itxxz.HelloItxxz"
    singleton="false">
    <constructor-arg>
      <value>Hello! 这是itxxzBean! value>
    </constructor-arg>
  </bean>
</beans>
```

### 第二种：工厂方法（Factory Method）

通常由应用程序直接使用new创建新的对象，为了将对象的创建和使用相分离，采用工厂模式,即应用程序将对象的创建及初始化职责交给工厂对象。

一般情况下,应用程序有自己的工厂对象来创建bean.如果将应用程序自己的工厂对象交给Spring管理，那么Spring管理的就不是普通的bean,而是工厂Bean。

以工厂方法中的静态方法为例讲解一下：

```java
import java.util.Random;

public class StaticFactoryBean {
   public static Integer createRandom() {
      return new Integer(new Random().nextInt());
    }
}
```

建一个config.xm配置文件，将其纳入Spring容器来管理,需要通过factory-method指定静态方法名称

```java
<bean id="random"
class="example.chapter3.StaticFactoryBean" factory-method="createRandom" //createRandom方法必须是static的,才能找到 scope="prototype"
/>

测试:
public static void main(String[] args) {
    //调用getBean()时,返回随机数.如果没有指定factory-method,会返回StaticFactoryBean的实例,即返回工厂Bean的实例    
    XmlBeanFactory factory = new XmlBeanFactory(new ClassPathResource("config.xml"));    
    System.out.println("我是IT学习者创建的实例:"+factory.getBean("random").toString());
}
```

### 第三种：单例模式（Singleton）

保证一个类仅有一个实例，并提供一个访问它的全局访问点。 
spring中的单例模式完成了后半句话，即提供了全局的访问点BeanFactory。但没有从构造器级别去控制单例，这是因为spring管理的是是任意的java对象。 
核心提示点：Spring下默认的bean均为singleton，可以通过singleton=“true|false” 或者 scope=“？”来指定

### 第四种：适配器（Adapter）

在**Spring的Aop中**，使用的Advice（通知）来增强被代理类的功能。Spring实现这一AOP功能的原理就使用代理模式（1、JDK动态代理。2、CGLib字节码生成技术代理。）对类进行方法级别的切面增强，即，生成被代理类的代理类， 并在代理类的方法前，设置拦截器，通过执行拦截器重的内容增强了代理方法的功能，实现的面向切面编程。

### 第五种：包装器（Decorator）

spring中用到的包装器模式在类名上有两种表现：一种是类名中含有Wrapper，另一种是类名中含有Decorator。基本上都是动态地给一个对象添加一些额外的职责。 

在我们的项目中遇到这样一个问题：我们的项目需要连接多个数据库，而且不同的客户在每次访问中根据需要会去访问不同的数据库。我们以往在spring总是配置一个数据源，因而sessionFactory的dataSource属性总是指向这个数据源并且恒定不变，所有DAO在使用sessionFactory的时候都是通过这个数据源访问数据库。

但是现在，由于项目的需要，我们的DAO在访问sessionFactory的时候都不得不在多个数据源中不断切换，问题就出现了：如何让sessionFactory在执行数据持久化的时候，根据客户的需求能够动态切换不同的数据源？我们能不能在spring的框架下通过少量修改得到解决？是否有什么设计模式可以利用呢？ 
首先想到在**spring的applicationContext中配置所有的dataSource**。这些dataSource可能是各种不同类型的，比如不同的数据库：Oracle、SQL Server、MySQL等，也可能是不同的数据源：比如apache 提供的org.apache.commons.dbcp.BasicDataSource、spring提供的org.springframework.jndi.JndiObjectFactoryBean等。然后sessionFactory根据客户的每次请求，将dataSource属性设置成不同的数据源，以到达切换数据源的目的。

### 第六种：代理（Proxy）

为其他对象提供一种代理以控制对这个对象的访问。 从结构上来看和Decorator模式类似，但Proxy是控制，更像是一种对功能的限制，而Decorator是增加职责。 
**spring的Proxy模式在aop中有体现**，比如JdkDynamicAopProxy和Cglib2AopProxy。 

### 第七种：观察者（Observer）

定义对象间的一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都得到通知并被自动更新。
spring中Observer模式常用的地方**是listener的实现。如ApplicationListener。** 

### 第八种：策略（Strategy）

定义一系列的算法，把它们一个个封装起来，并且使它们可相互替换。本模式使得算法可独立于使用它的客户而变化。 
spring中在实例化对象的时候用到Strategy模式
**在SimpleInstantiationStrategy中有如下代码说明了策略模式的使用情况**： 
![img](https://images2017.cnblogs.com/blog/106546/201708/106546-20170820090830818-985726848.png)

### 第九种：模板方法（Template Method）

定义一个操作中的算法的骨架，而将一些步骤延迟到子类中。Template Method使得子类可以不改变一个算法的结构，可重定义该算法的某些特定步骤。
Template Method模式一般是需要继承的。

**spring中的JdbcTemplate**，在用这个类时并不想去继承这个类，因为这个类的方法太多

但是我们还是想用到JdbcTemplate已有的稳定的、公用的数据库连接，那么我们怎么办呢？

> 我们可以把变化的东西抽出来作为一个参数传入JdbcTemplate的方法中。

但是变化的东西是一段代码，而且这段代码会用到JdbcTemplate中的变量。怎么办？

那我们就用回调对象。在这个回调对象中定义一个操纵JdbcTemplate中变量的方法，我们去实现这个方法，就把变化的东西集中到这里了。然后我们再传入这个回调对象到JdbcTemplate，从而完成了调用。这可能是Template Method不需要继承的另一种实现方式吧。 

以下是一个具体的例子： 
JdbcTemplate中的execute方法 
![img](https://images2017.cnblogs.com/blog/106546/201708/106546-20170820090844475-1011274237.png)

JdbcTemplate执行execute方法 
![img](https://images2017.cnblogs.com/blog/106546/201708/106546-20170820090853693-1656858967.png)

## 基础

### Spring5 新特性

1、依赖 JDK 8+和 Java EE7+以上版本
2、首次采用反应式编程模型
3、支持使用注解进行编程
4、新增函数式编程
5、支持使用 REST 断点执行反应式编程
6、支持 HTTP 2.0
7、新增 Kotlin 和 Spring WebFlux
8、可使用 Lambda 表达式注册 Bean
9、Spring WebMVC 支持最新的 API
10、使用 JUnit5 执行条件和并发测试
11、使用 Spring WebFlux 执行集成测试
12、核心容器优化

### 管理事务有几种方式？

答：有两种方式：

1、编程式事务，在代码中硬编码。(不推荐使用)

2、声明式事务，在配置文件中配置（推荐使用）

声明式事务又分为两种：

a、基于XML的声明式事务

b、基于注解的声明式事务

### 自动装配的方式

答：1、 No：即不启用自动装配。

2、 byName：通过属性的名字的方式查找JavaBean依赖的对象并为其注入。比如说类Computer有个属性printer，指定其autowire属性为byName后，Spring IoC容器会在配置文件中查找id/name属性为printer的bean，然后使用Seter方法为其注入。

3、 byType：通过属性的类型查找JavaBean依赖的对象并为其注入。比如类Computer有个属性printer，类型为Printer，那么，指定其autowire属性为byType后，Spring IoC容器会查找Class属性为Printer的bean，使用Seter方法为其注入。

4、 constructor：通byType一样，也是通过类型查找依赖对象。与byType的区别在于它不是使用Seter方法注入，而是使用构造子注入。

5、 autodetect：在byType和constructor之间自动的选择注入方式。

6、 default：由上级标签·的default-autowire属性确定。

14.s**pringMVC的流程？**

答：1.用户发送请求至前端控制器DispatcherServlet

2.DispatcherServlet收到请求调用HandlerMapping处理器映射器。

3.处理器映射器根据请求url找到具体的处理器，生成处理器对象及处理器拦截器(如果有则生成)一并返回给DispatcherServlet。

4.DispatcherServlet通过HandlerAdapter处理器适配器调用处理器

5.执行处理器(Controller，也叫后端控制器)。

6.Controller执行完成返回ModelAndView

7.HandlerAdapter将controller执行结果ModelAndView返回给DispatcherServlet

8.DispatcherServlet将ModelAndView传给ViewReslover视图解析器

9.ViewReslover解析后返回具体View

10.DispatcherServlet对View进行渲染视图（即将模型数据填充至视图中）。

11.DispatcherServlet响应用户

![img](https://img-blog.csdnimg.cn/20181108085004643.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5NDcwNzMz,size_16,color_FFFFFF,t_70)

15.S**pringmvc的优点**

答：1.它是基于组件技术的.全部的应用对象,无论控制器和视图,还是业务对象之类的都是 java组件.并且和Spring提供的其他基础结构紧密集成.

2.不依赖于Servlet API(目标虽是如此,但是在实现的时候确实是依赖于Servlet的)

3. 可以任意使用各种视图技术,而不仅仅局限于JSP

4 . 支持各种请求资源的映射策略

5 .它应是易于扩展的

### BeanFactory和ApplicationContext有什么区别？

​    BeanFactory和ApplicationContext是Spring的两大核心接口，都可以当做Spring的容器。其中ApplicationContext是BeanFactory的子接口。

（1）BeanFactory：是Spring里面最底层的接口，包含了各种Bean的定义，读取bean配置文档，管理bean的加载、实例化，控制bean的生命周期，维护bean之间的依赖关系。ApplicationContext接口作为BeanFactory的派生，除了提供BeanFactory所具有的功能外，还提供了更完整的框架功能：

①继承MessageSource，因此支持国际化。

②统一的资源文件访问方式。

③提供在监听器中注册bean的事件。

④同时加载多个配置文件。

⑤载入多个（有继承关系）上下文 ，使得每一个上下文都专注于一个特定的层次，比如应用的web层。

（2）①BeanFactroy采用的是延迟加载形式来注入Bean的，即只有在使用到某个Bean时(调用getBean())，才对该Bean进行加载实例化。这样，我们就不能发现一些存在的Spring的配置问题。如果Bean的某一个属性没有注入，BeanFacotry加载后，直至第一次使用调用getBean方法才会抛出异常。

​    ②ApplicationContext，它是在容器启动时，一次性创建了所有的Bean。这样，在容器启动时，我们就可以发现Spring中存在的配置错误，这样有利于检查所依赖属性是否注入。 ApplicationContext启动后预载入所有的单实例Bean，通过预载入单实例bean ,确保当你需要的时候，你就不用等待，因为它们已经创建好了。

​    ③相对于基本的BeanFactory，ApplicationContext 唯一的不足是占用内存空间。当应用程序配置Bean较多时，程序启动较慢。

（3）BeanFactory通常以编程的方式被创建，ApplicationContext还能以声明的方式创建，如使用ContextLoader。

（4）BeanFactory和ApplicationContext都支持BeanPostProcessor、BeanFactoryPostProcessor的使用，但两者之间的区别是：BeanFactory需要手动注册，而ApplicationContext则是自动注册。

### 解释Spring Bean的生命周期

 首先说一下Servlet的生命周期：实例化，初始init，接收请求service，销毁destroy；

 Spring上下文中的Bean生命周期也类似，如下：

（1）实例化Bean：

对于BeanFactory容器，当客户向容器请求一个尚未初始化的bean时，或初始化bean的时候需要注入另一个尚未初始化的依赖时，容器就会调用createBean进行实例化。对于ApplicationContext容器，当容器启动结束后，通过获取BeanDefinition对象中的信息，实例化所有的bean。

（2）设置对象属性（依赖注入）：

实例化后的对象被封装在BeanWrapper对象中，紧接着，Spring根据BeanDefinition中的信息 以及 通过BeanWrapper提供的设置属性的接口完成依赖注入。

（3）处理Aware接口：

接着，Spring会检测该对象是否实现了xxxAware接口，并将相关的xxxAware实例注入给Bean：

①如果这个Bean已经实现了BeanNameAware接口，会调用它实现的setBeanName(String beanId)方法，此处传递的就是Spring配置文件中Bean的id值；

②如果这个Bean已经实现了BeanFactoryAware接口，会调用它实现的setBeanFactory()方法，传递的是Spring工厂自身。

③如果这个Bean已经实现了ApplicationContextAware接口，会调用setApplicationContext(ApplicationContext)方法，传入Spring上下文；

（4）BeanPostProcessor：

如果想对Bean进行一些自定义的处理，那么可以让Bean实现了BeanPostProcessor接口，那将会调用postProcessBeforeInitialization(Object obj, String s)方法。

（5）InitializingBean 与 init-method：

如果Bean在Spring配置文件中配置了 init-method 属性，则会自动调用其配置的初始化方法。

（6）如果这个Bean实现了BeanPostProcessor接口，将会调用postProcessAfterInitialization(Object obj, String s)方法；由于这个方法是在Bean初始化结束时调用的，所以可以被应用于内存或缓存技术；

> 以上几个步骤完成后，Bean就已经被正确创建了，之后就可以使用这个Bean了。

（7）DisposableBean：

当Bean不再需要时，会经过清理阶段，如果Bean实现了DisposableBean这个接口，会调用其实现的destroy()方法；

（8）destroy-method：

最后，如果这个Bean的Spring配置中配置了destroy-method属性，会自动调用其配置的销毁方法。

###  解释Spring支持的几种bean的作用域。

Spring容器中的bean可以分为5个范围：

（1）singleton：默认，每个容器中只有一个bean的实例，单例的模式由BeanFactory自身来维护。

（2）prototype：为每一个bean请求提供一个实例。

（3）request：为每一个网络请求创建一个实例，在请求完成以后，bean会失效并被垃圾回收器回收。

（4）session：与request范围类似，确保每个session中有一个bean的实例，在session过期后，bean会随之失效。

（5）global-session：全局作用域，global-session和Portlet应用相关。当你的应用部署在Portlet容器中工作时，它包含很多portlet。如果你想要声明让所有的portlet共用全局的存储变量的话，那么这全局变量需要存储在global-session中。全局作用域与Servlet中的session作用域效果相同。

### Spring框架中的单例Beans是线程安全的么？

​    Spring框架并没有对单例bean进行任何多线程的封装处理。关于单例bean的线程安全和并发问题需要开发者自行去搞定。但实际上，大部分的Spring bean并没有可变的状态(比如Serview类和DAO类)，所以在某种程度上说Spring的单例bean是线程安全的。如果你的bean有多种状态的话（比如 View Model 对象），就需要自行保证线程安全。最浅显的解决办法就是将多态bean的作用域由“singleton”变更为“prototype”。

### Spring如何处理线程并发问题？

在一般情况下，只有无状态的Bean才可以在多线程环境下共享，在Spring中，绝大部分Bean都可以声明为singleton作用域，因为Spring对一些Bean中非线程安全状态采用ThreadLocal进行处理，解决线程安全问题。

ThreadLocal和线程同步机制都是为了解决多线程中相同变量的访问冲突问题。同步机制采用了“时间换空间”的方式，仅提供一份变量，不同的线程在访问前需要获取锁，没获得锁的线程则需要排队。而ThreadLocal采用了“空间换时间”的方式。

ThreadLocal会为每一个线程提供一个独立的变量副本，从而隔离了多个线程对数据的访问冲突。因为每一个线程都拥有自己的变量副本，从而也就没有必要对该变量进行同步了。ThreadLocal提供了线程安全的共享对象，在编写多线程代码时，可以把不安全的变量封装进ThreadLocal。

### Spring基于xml注入bean的几种方式

（1）Set方法注入；

（2）构造器注入：①通过index设置参数的位置；②通过type设置参数类型；

（3）静态工厂注入；

（4）实例工厂；

详细内容可以阅读：https://blog.csdn.net/a745233700/article/details/89307518

### Spring的自动装配

在spring中，对象无需自己查找或创建与其关联的其他对象，由容器负责把需要相互协作的对象引用赋予各个对象，使用autowire来配置自动装载模式。

在Spring框架xml配置中共有5种自动装配：

（1）no：默认的方式是不进行自动装配的，通过手工设置ref属性来进行装配bean。

（2）byName：通过bean的名称进行自动装配，如果一个bean的 property 与另一bean 的name 相同，就进行自动装配。 

（3）byType：通过参数的数据类型进行自动装配。

（4）constructor：利用构造函数进行装配，并且构造函数的参数通过byType进行装配。

（5）autodetect：自动探测，如果有构造方法，通过 construct的方式自动装配，否则使用 byType的方式自动装配。

基于注解的方式：

使用@Autowired注解来自动装配指定的bean。在使用@Autowired注解之前需要在Spring配置文件进行配置，<context:annotation-config />。在启动spring IoC时，容器自动装载了一个AutowiredAnnotationBeanPostProcessor后置处理器，当容器扫描到@Autowied、@Resource或@Inject时，就会在IoC容器自动查找需要的bean，并装配给该对象的属性。在使用@Autowired时，首先在容器中查询对应类型的bean：

如果查询结果刚好为一个，就将该bean装配给@Autowired指定的数据；

如果查询的结果不止一个，那么@Autowired会根据名称来查找；

如果上述查找的结果为空，那么会抛出异常。解决方法时，使用required=false。

> @Autowired可用于：构造函数、成员变量、Setter方法
>
> 注：@Autowired和@Resource之间的区别
>
> (1) @Autowired默认是按照类型装配注入的，默认情况下它要求依赖对象必须存在（可以设置它required属性为false）。
>
> (2) @Resource默认是按照名称来装配注入的，只有当找不到与名称匹配的bean才会按照类型来装配注入。

### Spring 框架中都用到了哪些设计模式？

（1）工厂模式：BeanFactory就是简单工厂模式的体现，用来创建对象的实例；

（2）单例模式：Bean默认为单例模式。

（3）代理模式：Spring的AOP功能用到了JDK的动态代理和CGLIB字节码生成技术；

（4）模板方法：用来解决代码重复的问题。比如. RestTemplate, JmsTemplate, JpaTemplate。

（5）观察者模式：定义对象键一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都会得到通知被制动更新，如Spring中listener的实现--ApplicationListener。

### Spring事务的实现方式和实现原理

Spring事务的本质其实就是数据库对事务的支持，没有数据库的事务支持，spring是无法提供事务功能的。真正的数据库层的事务提交和回滚是通过binlog或者redo log实现的。

**（1）Spring事务的种类：**

spring支持编程式事务管理和声明式事务管理两种方式：

①编程式事务管理使用TransactionTemplate。

②声明式事务管理建立在AOP之上的。其本质是通过AOP功能，对方法前后进行拦截，将事务处理的功能编织到拦截的方法中，也就是在目标方法开始之前加入一个事务，在执行完目标方法之后根据执行情况提交或者回滚事务。

> 声明式事务最大的优点就是不需要在业务逻辑代码中掺杂事务管理的代码，只需在配置文件中做相关的事务规则声明或通过@Transactional注解的方式，便可以将事务规则应用到业务逻辑中。
>
> 声明式事务管理要优于编程式事务管理，这正是spring倡导的非侵入式的开发方式，使业务代码不受污染，只要加上注解就可以获得完全的事务支持。唯一不足地方是，最细粒度只能作用到方法级别，无法做到像编程式事务那样可以作用到代码块级别。

**（2）spring的事务传播行为：**

spring事务的传播行为说的是，当多个事务同时存在的时候，spring如何处理这些事务的行为。

> ① PROPAGATION_REQUIRED：如果当前没有事务，就创建一个新事务，如果当前存在事务，就加入该事务，该设置是最常用的设置。
>
> ② PROPAGATION_SUPPORTS：支持当前事务，如果当前存在事务，就加入该事务，如果当前不存在事务，就以非事务执行。‘
>
> ③ PROPAGATION_MANDATORY：支持当前事务，如果当前存在事务，就加入该事务，如果当前不存在事务，就抛出异常。
>
> ④ PROPAGATION_REQUIRES_NEW：创建新事务，无论当前存不存在事务，都创建新事务。
>
> ⑤ PROPAGATION_NOT_SUPPORTED：以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。
>
> ⑥ PROPAGATION_NEVER：以非事务方式执行，如果当前存在事务，则抛出异常。
>
> ⑦ PROPAGATION_NESTED：如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则按REQUIRED属性执行。

**（3）Spring中的隔离级别：**

> ① ISOLATION_DEFAULT：这是个 PlatfromTransactionManager 默认的隔离级别，使用数据库默认的事务隔离级别。
>
> ② ISOLATION_READ_UNCOMMITTED：读未提交，允许另外一个事务可以看到这个事务未提交的数据。
>
> ③ ISOLATION_READ_COMMITTED：读已提交，保证一个事务修改的数据提交后才能被另一事务读取，而且能看到该事务对已有记录的更新。
>
> ④ ISOLATION_REPEATABLE_READ：可重复读，保证一个事务修改的数据提交后才能被另一事务读取，但是不能看到该事务对已有记录的更新。
>
> ⑤ ISOLATION_SERIALIZABLE：一个事务在执行的过程中完全看不到其他事务对数据库所做的更新。

###  Spring框架中有哪些不同类型的事件？

Spring 提供了以下5种标准的事件：

（1）上下文更新事件（ContextRefreshedEvent）：在调用ConfigurableApplicationContext 接口中的refresh()方法时被触发。

（2）上下文开始事件（ContextStartedEvent）：当容器调用ConfigurableApplicationContext的Start()方法开始/重新开始容器时触发该事件。

（3）上下文停止事件（ContextStoppedEvent）：当容器调用ConfigurableApplicationContext的Stop()方法停止容器时触发该事件。

（4）上下文关闭事件（ContextClosedEvent）：当ApplicationContext被关闭时触发该事件。容器被关闭时，其管理的所有单例Bean都被销毁。

（5）请求处理事件（RequestHandledEvent）：在Web应用中，当一个http请求（request）结束触发该事件。

如果一个bean实现了ApplicationListener接口，当一个ApplicationEvent 被发布以后，bean会自动被通知。

### 解释一下Spring AOP里面的几个名词

（1）切面（Aspect）：被抽取的公共模块，可能会横切多个对象。 在Spring AOP中，切面可以使用通用类（基于模式的风格） 或者在普通类中以 @AspectJ 注解来实现。

（2）连接点（Join point）：指方法，在Spring AOP中，一个连接点 总是 代表一个方法的执行。 

（3）通知（Advice）：在切面的某个特定的连接点（Join point）上执行的动作。通知有各种类型，其中包括“around”、“before”和“after”等通知。许多AOP框架，包括Spring，都是以拦截器做通知模型， 并维护一个以连接点为中心的拦截器链。

（4）切入点（Pointcut）：切入点是指 我们要对哪些Join point进行拦截的定义。通过切入点表达式，指定拦截的方法，比如指定拦截add*、search*。

（5）引入（Introduction）：（也被称为内部类型声明（inter-type declaration））。声明额外的方法或者某个类型的字段。Spring允许引入新的接口（以及一个对应的实现）到任何被代理的对象。例如，你可以使用一个引入来使bean实现 IsModified 接口，以便简化缓存机制。

（6）目标对象（Target Object）： 被一个或者多个切面（aspect）所通知（advise）的对象。也有人把它叫做 被通知（adviced） 对象。 既然Spring AOP是通过运行时代理实现的，这个对象永远是一个 被代理（proxied） 对象。

（7）织入（Weaving）：指把增强应用到目标对象来创建新的代理对象的过程。Spring是在运行时完成织入。

切入点（pointcut）和连接点（join point）匹配的概念是AOP的关键，这使得AOP不同于其它仅仅提供拦截功能的旧技术。 切入点使得定位通知（advice）可独立于OO层次。 例如，一个提供声明式事务管理的around通知可以被应用到一组横跨多个对象中的方法上（例如服务层的所有业务操作）。

![img](https://img-blog.csdn.net/20180708154818891?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2E3NDUyMzM3MDA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

**15、Spring通知有哪些类型？**

https://blog.csdn.net/qq_32331073/article/details/80596084

（1）前置通知（Before advice）：在某连接点（join point）之前执行的通知，但这个通知不能阻止连接点前的执行（除非它抛出一个异常）。

（2）返回后通知（After returning advice）：在某连接点（join point）正常完成后执行的通知：例如，一个方法没有抛出任何异常，正常返回。 

（3）抛出异常后通知（After throwing advice）：在方法抛出异常退出时执行的通知。 

（4）后通知（After (finally) advice）：当某连接点退出的时候执行的通知（不论是正常返回还是异常退出）。 

（5）环绕通知（Around Advice）：包围一个连接点（join point）的通知，如方法调用。这是最强大的一种通知类型。 环绕通知可以在方法调用前后完成自定义的行为。它也会选择是否继续执行连接点或直接返回它们自己的返回值或抛出异常来结束执行。 环绕通知是最常用的一种通知类型。大部分基于拦截的AOP框架，例如Nanning和JBoss4，都只提供环绕通知。 

> 同一个aspect，不同advice的执行顺序：
>
> ①没有异常情况下的执行顺序：
>
> around before advice
> before advice
> target method 执行
> around after advice
> after advice
> afterReturning
>
> ②有异常情况下的执行顺序：
>
> around before advice
> before advice
> target method 执行
> around after advice
> after advice
> afterThrowing:异常发生
> java.lang.RuntimeException: 异常发生

[SpringMVC常见面试题总结（超详细回答）](https://blog.csdn.net/a745233700/article/details/80963758)

[Spring常见面试题总结（超详细回答）](https://blog.csdn.net/a745233700/article/details/80959716)

[Mybatis常见面试题总结](https://blog.csdn.net/a745233700/article/details/80977133)

# Maybatis

## Spring Data JPA 和MyBatis比较

现在Dao持久层的解决方案中，大部分是采用Spring Data JPA或MyBatis解决方案，并且传统企业多用前者，互联网企业多用后者。

Spring Data JPA 是Spring Data 在JPA（Java持久层规范）和ORM（对象关系映射）框架之间抽象封装层，它不直接代替ORM框架，默认低层使用的ORM框架是Hibernate，但使用它可以灵活的在各种ORM框架之间切换，并且减少ORM框架接入部分重复代码，进而简化代码。

MyBatis是一个持久层框架的，但它设计初衷与Hibernate等全自动、符合JPA规范的ORM框架不同，重点关注关系到对象的（R——》O），而后者不仅是关系到对象的映射，还有对象到关系的映射（O——》R），设计上希望通过面向对象的方式写SQL，可以更好的屏蔽不同数据库之间的差异，抽象程度更高。而前者MyBatis需要自己手动写SQL，更灵活，但受限于开发编写SQL代码水平，可能会出现不兼容不同数据库SQL的情况。

网上也有观点认为ORM是一种反模式，认为从关系数据库到面向对象不完全是一一对应的，强行的要ORM反而会让一些设计变得很奇怪。

总的来说，Spring Data JPA和MyBatis都是很不错且被广泛使用的持久层解决方案，具体用那个可以看团队成员对技术栈熟悉程度以及项目是否对数据持久层方面有特殊需求。相对来说Spring Data JPA/Hibernate用好的话会简单些，不过复杂查询及结果集的返回没有直接用MyBatis灵活方便