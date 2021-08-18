# 一、Spring Boot 概述

## 1、Spring Boot 是什么

1）通过场景启动器、自动配置、嵌入式的Servlet容器等机制重新整合Spring技术栈，简化Spring应用开发的一个框架；是J2EE开发的一站式解决方案；

2）微服务：是一种架构风格（服务微化）；一个应用应该是一组小型服务；可以通过HTTP等方式进行互通；而单体应用：ALL IN ONE；微服务中每一个功能元素最终都是一个可独立替换和独立升级的软件单元；

3）为什么使用SpringBoot：传统的SSM等架构编写应用时需要很多的配置，而微服务中一个应用由很多独立部署的服务构成，细粒度的应用划分就会有更多的配置等繁琐的工作，显然SpringBoot更适合微服务架构应用。

## 2、Spring Boot HelloWorld

### 2.1、创建一个maven工程；

### 2.2、导入spring boot相关的依赖
```java
<parent>        
    <groupId>org.springframework.boot</groupId>       
    <artifactId>spring-boot-starter-parent</artifactId>        
    <version>1.5.9.RELEASE</version>    
</parent>    
<dependencies>        
    <dependency>            
        <groupId>org.springframework.boot</groupId>            
        <artifactId>spring-boot-starter-web</artifactId>        
    </dependency>    
</dependencies>
```
### 2.3、编写一个主程序；启动Spring Boot应用

```java
/*
 * @SpringBootApplication 来标注一个主程序类，说明这是一个Spring Boot应用
 */
@SpringBootApplication
public class HelloWorldMainApplication {
  public static void main(String[] args) {
     // Spring应用启动起来
     SpringApplication.run(HelloWorldMainApplication.class,args);
  }
}
```
### 2.4、编写相关的Controller、Service
```java
@Controller
public class HelloController {

  @ResponseBody
  @RequestMapping("/hello")
  public String hello(){
    return "Hello World!";
  }
}
```

### 2.5、运行主程序测试

### 2.6、简化部署
 ```java
<!-- 这个插件，可以将应用打包成一个可执行的jar包；-->
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
 ```
将这个应用打成jar包，直接使用java -jar的命令进行执行；

## 3、Hello World分析

### 3.1、POM文件

#### 1）父项目

```java
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.9.RELEASE</version>
</parent>
<parent>
    <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-dependencies</artifactId>
        <version>1.5.9.RELEASE</version>
    <relativePath>../../spring-boot-dependencies</relativePath>
</parent>
```

Spring Boot的版本仲裁中心；这里真正管理Spring Boot应用里面的所有依赖版本；以后导入依赖默认是不需要写版本；（没有在dependencies里面管理的依赖需要声明版本号）

#### 2）启动器


  org.springframework.boot
  spring-boot-starter-web


spring-boot-starter-web：spring-boot-web场景启动器；帮我们导入了web模块正常运行所依赖的组件；Spring Boot将所有的功能场景都抽取出来，做成一个个的starters（启动器），只需要在项目里面引入这些starter相关场景的所有依赖都会导入进来。要用什么功能就导入什么场景的启动器

 

### 3.2、主程序类，主入口类

/
  @SpringBootApplication 来标注一个主程序类，说明这是一个Spring Boot应用
 /
@SpringBootApplication
public class HelloWorldMainApplication {

  public static void main(String[] args) {

​    // Spring应用启动起来
​    SpringApplication.run(HelloWorldMainApplication.class,args);
  }
}

@SpringBootApplication: Spring Boot应用标注在某个类上说明这个类是SpringBoot的主配置类，SpringBoot就应该运行这个类的main方法来启动SpringBoot应用；

 

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = {
   @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
   @Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {

@SpringBootConfiguration:Spring Boot的配置类；

 标注在某个类上，表示这是一个Spring Boot的配置类；

 @Configuration:配置类上来标注这个注解；

 配置类 ----- 配置文件；配置类也是容器中的一个组件；@Component

@EnableAutoConfiguration：开启自动配置功能；

@AutoConfigurationPackage
@Import(EnableAutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {

 @AutoConfigurationPackage：自动配置包

 @Import(AutoConfigurationPackages.Registrar.class)：

 Spring的底层注解@Import，给容器中导入一个组件；导入的组件由AutoConfigurationPackages.Registrar.class；

将主配置类（@SpringBootApplication标注的类）的所在包及下面所有子包里面的所有组件扫描到Spring容器；

 @Import(EnableAutoConfigurationImportSelector.class)；

 给容器中导入组件？

 EnableAutoConfigurationImportSelector：导入哪些组件的选择器；

 将所有需要导入的组件以全类名的方式返回；这些组件就会被添加到容器中；

 会给容器中导入非常多的自动配置类（xxxAutoConfiguration）；就是给容器中导入这个场景需要的所有组件，并配置好这些组件； ![img](file:///C:UsersADMINI~1AppDataLocalTempksohtmlwpsB8ED.tmp.jpg)

有了自动配置类，免去了我们手动编写配置注入功能组件等的工作；

SpringFactoriesLoader.loadFactoryNames(EnableAutoConfiguration.class,classLoader)；

Spring Boot在启动的时候从类路径下的META-INF/spring.factories中获取EnableAutoConfiguration指定的值，将这些值作为自动配置类导入到容器中，自动配置类就生效，帮我们进行自动配置工作；以前我们需要自己配置的东西，自动配置类都帮我们；

J2EE的整体整合解决方案和自动配置都在spring-boot-autoconfigure-2.0.3.RELEASE.jar；

 

#  二、配置文件

## 1、配置文件

SpringBoot使用一个全局的配置文件，配置文件名是固定的；

• application.properties

• application.yml

配置文件的作用：SpringBoot提供很多自动配置类，很多属性提供默认值，可以再配置文件中修改这些默认值；

 

## 2、配置文件值注入

配置文件application.yml

person:
  lastName: hello
  age: 18
  boss: false
  birth: 2019/09/09
  maps: {k1: v1,k2: 12}
  lists:
   - lisi
   - zhaoliu

    dog:
      name: 小狗
      age: 12

javaBean：

/
  将配置文件中配置的每一个属性的值，映射到这个组件中
  @ConfigurationProperties：告诉SpringBoot将本类中的所有属性和配置文件中相关的配置进行绑定；
    prefix = "person"：配置文件中哪个下面的所有属性进行一一映射

  只有这个组件是容器中的组件，才能容器提供的@ConfigurationProperties功能；

 /
@Component
@ConfigurationProperties(prefix = "person")
public class Person {

  private String lastName;
  private Integer age;
  private Boolean boss;
  private Date birth;

  private Map<String,Object> maps;
  private List<Object> lists;
  private Dog dog;



	org.springframework.boot
	spring-boot-configuration-processor
	true


#### 2.1、properties配置文件在idea中默认utf-8可能会乱码

![img](file:///C:UsersADMINI~1AppDataLocalTempksohtmlwpsB8EE.tmp.jpg) 

 

#### 2.2、@Value获取值和@ConfigurationProperties获取值比较

|                      | @ConfigurationProperties | @Value     |
| -------------------- | ------------------------ | ---------- |
| 功能                 | 批量注入配置文件中的属性 | 一个个指定 |
| 松散绑定（松散语法） | 支持                     | 不支持     |
| SpEL                 | 不支持                   | 支持       |
| JSR303数据校验       | 支持                     | 不支持     |
| 复杂类型封装         | 支持                     | 不支持     |

配置文件yml还是properties都能获取到值；

如果说只是在某个业务逻辑中需要获取一下配置文件中的某项值，使用@Value；

如果说专门编写了一个javaBean来和配置文件进行映射，就直接使用@ConfigurationProperties；

 

#### 2.3、配置文件注入值数据校验

@Component
@ConfigurationProperties(prefix = "person")
@Validated
public class Person {

  //lastName必须是邮箱格式
  @Email
  //@Value("${person.last-name}")
  private String lastName;

 

#### 2.4、@PropertySource&@ImportResource&@Bean

@PropertySource：加载指定的配置文件；

@PropertySource(value = {"classpath:person.properties"})
@Component
@ConfigurationProperties(prefix = "person")
//@Validated
public class Person {

 

@ImportResource：导入Spring的配置文件，让配置文件里面的内容生效；

Spring Boot里面没有Spring的配置文件，自己编写的配置文件也不能自动识别；想让Spring的配置文件生效，加载进来；@ImportResource标注在一个配置类上

@ImportResource(locations = {"classpath:beans.xml"})
导入Spring的配置文件让其生效

编写Spring的配置文件

 version="1.0" encoding="UTF-8"?>
 xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

   id="helloService" class="com.hello.springboot.service.HelloService">


SpringBoot推荐给容器中添加组件的方式；推荐使用全注解的方式

1、配置类@Configuration------>Spring配置文件

2、使用@Bean给容器中添加组件

/
  @Configuration：指明当前类是一个配置类；就是来替代之前的Spring配置文件

  在配置文件中用标签添加组件

 /
@Configuration
public class MyAppConfig {

  //将方法的返回值添加到容器中；容器中这个组件默认的id就是方法名
  @Bean
  public HelloService helloService02(){
    System.out.println("配置类@Bean给容器中添加组件了...");
    return new HelloService();
  }
}

## 3、配置文件占位符

### 3.1、随机数

${random.value}、${random.int}、${random.long}
${random.int(10)}、${random.int[1024,65536]}

 

### 3.2、占位符获取之前配置的值，如果没有可以是用:指定默认值

person.last-name=张三${random.uuid}
person.age=${random.int}
person.birth=2017/12/15
person.boss=false
person.maps.k1=v1
person.maps.k2=14
person.lists=a,b,c
person.dog.name=${person.hello:hello}_dog
person.dog.age=15

 

## 4、Profile

### 4.1、多Profile文件

我们在主配置文件编写的时候，文件名可以是 application-{profile}.properties/yml

默认使用application.properties的配置；

 

### 4.2、yml支持多文档块方式 server:  port: 8081 spring:  profiles:   active: prod --- server:  port: 8083 spring:  profiles: dev --- server:  port: 8084 spring:  profiles: prod #指定属于哪个环境

 

### 4.3、激活指定profile

 1、在application.properties配置文件中指定 spring.profiles.active=dev

 2、命令行参数：

 java -jar spring-boot-02-config-0.0.1-SNAPSHOT.jar --spring.profiles.active=dev；

 3、虚拟机参数；

 -Dspring.profiles.active=dev

 

## 5、配置文件加载位置

springboot 启动会扫描以下位置的application.properties或者application.yml文件作为Spring boot的默认配置文件

–file:./config/

–file:./

–classpath:/config/

–classpath:/

优先级由高到底，高优先级的配置会覆盖低优先级的配置；

SpringBoot会从这四个位置全部加载主配置文件；互补配置；

 

还可以通过spring.config.location来改变默认的配置文件位置

项目打包好以后，我们可以使用命令行参数的形式，启动项目的时候来指定配置文件的新位置；指定配置文件和默认加载的这些配置文件共同起作用形成互补配置；

java -jar demo.jar --spring.config.location=G:/application.properties

 

## 6、外部配置加载顺序

SpringBoot也可以从以下位置加载配置； 优先级从高到低；高优先级的配置覆盖低优先级的配置，所有的配置会形成互补配置

1.命令行参数:所有的配置都可以在命令行上进行指定

java -jar demo.jar --server.port=8080 --server.context-path=/abc

多个配置用空格分开； --配置项=值

2.来自java:comp/env的JNDI属性

3.Java系统属性（System.getProperties()）

4.操作系统环境变量

5.RandomValuePropertySource配置的random.属性值

由jar包外向jar包内进行寻找；优先加载带profile

6.jar包外部的application-{profile}.properties或application.yml(带spring.profile)配置文件

7.jar包内部的application-{profile}.properties或application.yml(带spring.profile)配置文件

再来加载不带profile

8.jar包外部的application.properties或application.yml(不带spring.profile)配置文件

9.jar包内部的application.properties或application.yml(不带spring.profile)配置文件

10.@Configuration注解类上的@PropertySource

11.通过SpringApplication.setDefaultProperties指定的默认属性

#  三、SpringBoot原理

几个重要的事件回调机制

配置在META-INF/spring.factories

ApplicationContextInitializer

SpringApplicationRunListener

只需要放在ioc容器中

ApplicationRunner

CommandLineRunner

 

## 1、启动流程：

### 1.1、创建SpringApplication对象

public static void main(String[] args) {
  SpringApplication.run(Demo2Application.class, args);
}

public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {

//保存主配置类

this.primarySources = new LinkedHashSet(Arrays.asList(primarySources));

//当前web应用类型“SERVLET”
this.webApplicationType = WebApplicationType.deduceFromClasspath();

//从类路径下找到META-INF/spring.factories配置的所有ApplicationContextInitializer；然后保存起来
this.setInitializers(this.getSpringFactoriesInstances(ApplicationContextInitializer.class));

//从类路径下找到ETA-INF/spring.factories配置的所有ApplicationListener
this.setListeners(this.getSpringFactoriesInstances(ApplicationListener.class));

//从多个配置类中找到有main方法的主配置类
this.mainApplicationClass = this.deduceMainApplicationClass();

![img](file:///C:UsersADMINI~1AppDataLocalTempksohtmlwpsB8FE.tmp.jpg) 

 

### 1.2、运行run方法

public ConfigurableApplicationContext run(String... args) {
  StopWatch stopWatch = new StopWatch();
  stopWatch.start();
  ConfigurableApplicationContext context = null;
  FailureAnalyzers analyzers = null;
  configureHeadlessProperty();

  //获取SpringApplicationRunListeners；从类路径下META-INF/spring.factories
  SpringApplicationRunListeners listeners = getRunListeners(args);
  //回调所有的获取SpringApplicationRunListener.starting()方法
  listeners.starting();
  try {
    //封装命令行参数
   ApplicationArguments applicationArguments = new DefaultApplicationArguments(
      args);
   //准备环境
   ConfigurableEnvironment environment = prepareEnvironment(listeners,
      applicationArguments);
    //创建环境完成后回调SpringApplicationRunListener.environmentPrepared()；表示环境准备完成
    
   Banner printedBanner = printBanner(environment);
    
    //创建ApplicationContext；决定创建web的ioc还是普通的ioc
   context = createApplicationContext();
    
   exceptionReporters = getSpringFactoriesInstances(SpringBootExceptionReporter.class,
   new Class[] { ConfigurableApplicationContext.class }, context);
    //准备上下文环境;将environment保存到ioc中；而且applyInitializers()；
    //applyInitializers()：回调之前保存的所有的ApplicationContextInitializer的initialize方法
    //回调所有的SpringApplicationRunListener的contextPrepared()；
    //
   prepareContext(context, environment, listeners, applicationArguments,
      printedBanner);
    //prepareContext运行完成以后回调所有的SpringApplicationRunListener的contextLoaded（）；
    
    //s刷新容器；ioc容器初始化（如果是web应用还会创建嵌入式的Tomcat）；
    //扫描，创建，加载所有组件的地方；（配置类，组件，自动配置）
   refreshContext(context);
    //从ioc容器中获取所有的ApplicationRunner和CommandLineRunner进行回调
    //ApplicationRunner先回调，CommandLineRunner再回调
   afterRefresh(context, applicationArguments);
    //所有的SpringApplicationRunListener回调started方法
   listeners.started(context);
   stopWatch.stop();
   if (this.logStartupInfo) {
     new StartupInfoLogger(this.mainApplicationClass)
        .logStarted(getApplicationLog(), stopWatch);
   }
    //整个SpringBoot应用启动完成以后返回启动的ioc容器；
   return context;
  }
  catch (Throwable ex) {
   handleRunFailure(context, listeners, analyzers, ex);
   throw new IllegalStateException(ex);
  }
}

## 2、事件监听机制

配置在META-INF/spring.factories

ApplicationContextInitializer

public class HelloApplicationContextInitializer implements ApplicationContextInitializer<ConfigurableApplicationContext> {
  @Override
  public void initialize(ConfigurableApplicationContext applicationContext) {
    System.out.println("ApplicationContextInitializer...initialize..."+applicationContext);
  }
}

SpringApplicationRunListener

public class HelloSpringApplicationRunListener implements SpringApplicationRunListener {

  //必须有的构造器
  public HelloSpringApplicationRunListener(SpringApplication application, String[] args){

  }

  @Override
  public void starting() {
    System.out.println("SpringApplicationRunListener...starting...");
  }

  @Override
  public void environmentPrepared(ConfigurableEnvironment environment) {
    Object o = environment.getSystemProperties().get("os.name");
    System.out.println("SpringApplicationRunListener...environmentPrepared.."+o);
  }

  @Override
  public void contextPrepared(ConfigurableApplicationContext context) {
    System.out.println("SpringApplicationRunListener...contextPrepared...");
  }

  @Override
  public void contextLoaded(ConfigurableApplicationContext context) {
    System.out.println("SpringApplicationRunListener...contextLoaded...");
  }

  @Override
  public void finished(ConfigurableApplicationContext context, Throwable exception) {
    System.out.println("SpringApplicationRunListener...finished...");
  }
}

配置（META-INF/spring.factories）

org.springframework.context.ApplicationContextInitializer=
com.hello.springboot.listener.HelloApplicationContextInitializer

org.springframework.boot.SpringApplicationRunListener=
com.hello.springboot.listener.HelloSpringApplicationRunListener

 

 

只需要放在ioc容器中

ApplicationRunner

@Component
public class HelloApplicationRunner implements ApplicationRunner {
  @Override
  public void run(ApplicationArguments args) throws Exception {
    System.out.println("ApplicationRunner...run....");
  }
}

 

CommandLineRunner

@Component
public class HelloCommandLineRunner implements CommandLineRunner {
  @Override
  public void run(String... args) throws Exception {
    System.out.println("CommandLineRunner...run..."+ Arrays.asList(args));
  }
}

 

## 3、自动配置原理

### 3.1、原理分析

配置文件到底能写什么？怎么写？自动配置原理；

1）、SpringBoot启动的时候加载主配置类，开启了自动配置功能 @EnableAutoConfiguration

2）、@EnableAutoConfiguration 作用：

• 利用EnableAutoConfigurationImportSelector给容器中导入一些组件？

• 可以查看selectImports()方法的内容；

• List configurations = getCandidateConfigurations(annotationMetadata, attributes);获取候选的配置

– SpringFactoriesLoader.loadFactoryNames()
扫描所有jar包类路径下  META-INF/spring.factories
把扫描到的这些文件的内容包装成properties对象
从properties中获取到EnableAutoConfiguration.class类（类名）对应的值，然后把他们添加在容器中

  

将 类路径下 META-INF/spring.factories 里面配置的所有EnableAutoConfiguration的值加入到了容器中；

# Auto Configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=
org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,
org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,
org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration,
org.springframework.boot.autoconfigure.batch.BatchAutoConfiguration,
org.springframework.boot.autoconfigure.cache.CacheAutoConfiguration,
org.springframework.boot.autoconfigure.cassandra.CassandraAutoConfiguration,
………………………………………………………………………………………………

每一个这样的 xxxAutoConfiguration类都是容器中的一个组件，都加入到容器中；用他们来做自动配置；

3）、每一个自动配置类进行自动配置功能；

4）、以HttpEncodingAutoConfiguration（Http编码自动配置）为例解释自动配置原理；

@Configuration  //表示这是一个配置类，以前编写的配置文件一样，也可以给容器中添加组件
@EnableConfigurationProperties(HttpEncodingProperties.class)  //启动指定类的ConfigurationProperties功能；将配置文件中对应的值和HttpEncodingProperties绑定起来；并把HttpEncodingProperties加入到ioc容器中

@ConditionalOnWebApplication //Spring底层@Conditional注解（Spring注解版），根据不同的条件，如果满足指定的条件，整个配置类里面的配置就会生效；   判断当前应用是否是web应用，如果是，当前配置类生效

@ConditionalOnClass(CharacterEncodingFilter.class)  //判断当前项目有没有这个类CharacterEncodingFilter；SpringMVC中进行乱码解决的过滤器；

@ConditionalOnProperty(prefix = "spring.http.encoding", value = "enabled", matchIfMissing = true)  //判断配置文件中是否存在某个配置  spring.http.encoding.enabled；如果不存在，判断也是成立的
//即使我们配置文件中不配置pring.http.encoding.enabled=true，也是默认生效的；
public class HttpEncodingAutoConfiguration {

 	//他已经和SpringBoot的配置文件映射了
 	private final HttpEncodingProperties properties;

  //只有一个有参构造器的情况下，参数的值就会从容器中拿
 	public HttpEncodingAutoConfiguration(HttpEncodingProperties properties) {
		this.properties = properties;
	}

  @Bean  //给容器中添加一个组件，这个组件的某些值需要从properties中获取
	@ConditionalOnMissingBean(CharacterEncodingFilter.class) //判断容器没有这个组件？
	public CharacterEncodingFilter characterEncodingFilter() {
		CharacterEncodingFilter filter = new OrderedCharacterEncodingFilter();
		filter.setEncoding(this.properties.getCharset().name());
		filter.setForceRequestEncoding(this.properties.shouldForce(Type.REQUEST));
		filter.setForceResponseEncoding(this.properties.shouldForce(Type.RESPONSE));
		return filter;
	}

根据当前不同的条件判断，决定这个配置类是否生效？

一但这个配置类生效；这个配置类就会给容器中添加各种组件；这些组件的属性是从对应的properties类中获取的，这些类里面的每一个属性又是和配置文件绑定的；

 

5）、所有在配置文件中能配置的属性都是在xxxxProperties类中封装者‘；配置文件能配置什么就可以参照某个功能对应的这个属性类

@ConfigurationProperties(prefix = "spring.http.encoding")  //从配置文件中获取指定的值和bean的属性进行绑定
public class HttpEncodingProperties {

  public static final Charset DEFAULT_CHARSET = Charset.forName("UTF-8");

 

总结：

 1）、SpringBoot启动会加载大量的自动配置类

 2）、实现是查看需要的功能有没有SpringBoot默认写好的自动配置类；

 3）、再来看这个自动配置类中到底配置了哪些组件；

 4）、给容器中自动配置类添加组件的时候，会从properties类中获取某些属性。我们就可以在配置文件中指定这些属性的值；

xxxxAutoConfigurartion：自动配置类；

给容器中添加组件xxxxProperties:封装配置文件中相关属性；

 

### 3.2、自动配置类生效条件

@Conditional派生注解（Spring注解版原生的@Conditional作用）

作用：必须是@Conditional指定的条件成立，才给容器中添加组件，配置配里面的所有内容才生效；

| @Conditional扩展注解            | 作用（判断是否满足当前指定条件）                 |
| ------------------------------- | ------------------------------------------------ |
| @ConditionalOnJava              | 系统的java版本是否符合要求                       |
| @ConditionalOnBean              | 容器中存在指定Bean；                             |
| @ConditionalOnMissingBean       | 容器中不存在指定Bean；                           |
| @ConditionalOnExpression        | 满足SpEL表达式指定                               |
| @ConditionalOnClass             | 系统中有指定的类                                 |
| @ConditionalOnMissingClass      | 系统中没有指定的类                               |
| @ConditionalOnSingleCandidate   | 容器中只有一个指定的Bean，或者这个Bean是首选Bean |
| @ConditionalOnProperty          | 系统中指定的属性是否有指定的值                   |
| @ConditionalOnResource          | 类路径下是否存在指定资源文件                     |
| @ConditionalOnWebApplication    | 当前是web环境                                    |
| @ConditionalOnNotWebApplication | 当前不是web环境                                  |
| @ConditionalOnJndi              | JNDI存在指定项                                   |

自动配置类必须在一定的条件下才能生效；

我们怎么知道哪些自动配置类生效；

我们可以通过启用 debug=true属性；来让控制台打印自动配置报告，这样我们就可以很方便的知道哪些自动配置类生效；

=
AUTO-CONFIGURATION REPORT
=


Positive matches:（自动配置类启用的）
-----------------

  DispatcherServletAutoConfiguration matched:
   - @ConditionalOnClass found required class 'org.springframework.web.servlet.DispatcherServlet'; @ConditionalOnMissingClass did not find unwanted class (OnClassCondition)
   - @ConditionalOnWebApplication (required) found StandardServletEnvironment (OnWebApplicationCondition)


Negative matches:（没有启动，没有匹配成功的自动配置类）
-----------------

  ActiveMQAutoConfiguration:
   Did not match:
     - @ConditionalOnClass did not find required classes 'javax.jms.ConnectionFactory', 'org.apache.activemq.ActiveMQConnectionFactory' (OnClassCondition)

  AopAutoConfiguration:
   Did not match:
     - @ConditionalOnClass did not find required classes 'org.aspectj.lang.annotation.Aspect', 'org.aspectj.lang.reflect.Advice' (OnClassCondition)

 

## 4、嵌入式Tomcat启动原理；

什么时候创建嵌入式的Servlet容器工厂？什么时候获取嵌入式的Servlet容器并启动Tomcat；

获取嵌入式的Servlet容器工厂：

1）、SpringBoot应用启动运行run方法

2）、refreshContext(context);SpringBoot刷新IOC容器【创建IOC容器对象，并初始化容器，创建容器中的每一个组件】；

3）、refresh(context);刷新刚才创建好的ioc容器；

public void refresh() throws BeansException, IllegalStateException {
        onRefresh();
  }

4）、onRefresh(); web的ServletWebServerApplicationContext

容器重写了onRefresh方法通过createWebServer()创建Tomcat实例

protected void onRefresh() {
  super.onRefresh();
  try {
   createWebServer();
  }
  catch (Throwable ex) {
   throw new ApplicationContextException("Unable to start web server", ex);
  }
}

5）、getWebServerFactory获取到TomcatServletWebServerFactory
private void createWebServer() {
  WebServer webServer = this.webServer;
  ServletContext servletContext = getServletContext();
  if (webServer == null && servletContext == null) {
   ServletWebServerFactory factory = getWebServerFactory();
   this.webServer = factory.getWebServer(getSelfInitializer());
  }
  else if (servletContext != null) {
   try {
     getSelfInitializer().onStartup(servletContext);
   }
   catch (ServletException ex) {
     throw new ApplicationContextException("Cannot initialize servlet context", ex);
   }
  }
  initPropertySources();
}

6）TomcatServletWebServerFactory生成Tomcat实例并配置属性

public WebServer getWebServer(ServletContextInitializer... initializers) {
  Tomcat tomcat = new Tomcat();
  File baseDir = (this.baseDirectory != null) ? this.baseDirectory : createTempDir("tomcat");
  tomcat.setBaseDir(baseDir.getAbsolutePath());
  Connector connector = new Connector(this.protocol);
  tomcat.getService().addConnector(connector);
  customizeConnector(connector);
  tomcat.setConnector(connector);
  tomcat.getHost().setAutoDeploy(false);
  configureEngine(tomcat.getEngine());
  for (Connector additionalConnector : this.additionalTomcatConnectors) {
   tomcat.getService().addConnector(additionalConnector);
  }
  prepareContext(tomcat.getHost(), initializers);
  return getTomcatWebServer(tomcat);
}

7）TomcatWebServer.initialize()中tomcat.start()

 

#  四、日志

## 1、日至框架概览

JUL、JCL、Jboss-logging、logback、log4j、log4j2、slf4j....

| 日志门面 （日志的抽象层）                                    | 日志实现                                                |
| ------------------------------------------------------------ | ------------------------------------------------------- |
| JCL（Jakarta Commons Logging） SLF4j（Simple Logging Facade for Java） jboss-logging | Log4j JUL（java.util.logging） Log4j2 Logback |

日志门面： SLF4J；

日志实现：Logback；

SpringBoot：Spring框架默认是用JCL；SpringBoot选用 SLF4j和logback；

 

## 2、SLF4j使用

### 2.1、如何在系统中使用SLF4j https://www.slf4j.org

以后开发的时候，日志记录方法的调用，不应该来直接调用日志的实现类，而是调用日志抽象层里面的方法；给系统里面导入slf4j的jar和 logback的实现jar

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class HelloWorld {
 public static void main(String[] args) {
  Logger logger = LoggerFactory.getLogger(HelloWorld.class);
  logger.info("Hello World");
 }
}

每一个日志的实现框架都有自己的配置文件。使用slf4j以后，配置文件还是做成日志实现框架自己本身的配置文件；

 

### 2.2、遗留问题

不同的框架采用不同的日志框架，如何统一进slf4j+logback： Spring（commons-logging）、Hibernate（jboss-logging）、MyBatis、……

如何让系统中所有的日志都统一到slf4j；

1、将系统中其他日志框架先排除出去；

2、用中间包来替换原有的日志框架；

3、导入slf4j其他的实现

 

## 3、SpringBoot日志关系


		org.springframework.boot
		spring-boot-starter


SpringBoot使用它来做日志功能；

​	
​		org.springframework.boot
​		spring-boot-starter-logging
​	

底层依赖关系

![img](file:///C:UsersADMINI~1AppDataLocalTempksohtmlwpsB90F.tmp.jpg) 

总结：

 1）、SpringBoot底层也是使用slf4j+logback的方式进行日志记录

 2）、SpringBoot把其他的日志都替换成了slf4j；

 3）、中间替换包

@SuppressWarnings("rawtypes")
public abstract class LogFactory {

  static String UNSUPPORTED_OPERATION_IN_JCL_OVER_SLF4J = "http://www.slf4j.org/codes.html#unsupported_operation_in_jcl_over_slf4j";

  static LogFactory logFactory = new SLF4JLogFactory();

![img](file:///C:UsersADMINI~1AppDataLocalTempksohtmlwpsB910.tmp.jpg) 

 4）、如果要引入其他框架？一定要把这个框架的默认日志依赖移除掉？

 Spring框架用的是commons-logging；

​	<dependency>		<groupId>org.springframework</groupId>		<artifactId>spring-core</artifactId>		<exclusions>			<exclusion>				<groupId>commons-logging</groupId>				<artifactId>commons-logging</artifactId>			</exclusion>		</exclusions>	</dependency>

SpringBoot能自动适配所有的日志，而且底层使用slf4j+logback的方式记录日志，引入其他框架的时候，只需要把这个框架依赖的日志框架排除掉即可；

## 4、日志使用；

### 4.1、默认配置

SpringBoot默认配置好了日志；

​	//记录器
​	Logger logger = LoggerFactory.getLogger(getClass());
​	@Test
​	public void contextLoads() {
​		//System.out.println();

​		//日志的级别；
​		//由低到高  trace
​		//可以调整输出的日志级别；日志就只会在这个级别以以后的高级别生效
​		logger.trace("这是trace日志...");
​		logger.debug("这是debug日志...");
​		//SpringBoot默认给我们使用的是info级别的，没有指定级别的就用SpringBoot默认规定的级别；root级别
​		logger.info("这是info日志...");
​		logger.warn("这是warn日志...");
​		logger.error("这是error日志...");


	}

 




SpringBoot修改日志的默认配置

logging.level.com.hello=trace

#logging.path=
# 不指定路径在当前项目下生成springboot.log日志
# 可以指定完整的路径；
#logging.file=G:/springboot.log

# 在当前磁盘的根路径下创建spring文件夹和里面的log文件夹；使用 spring.log 作为默认文件
logging.path=/spring/log

#  在控制台输出的日志的格式
logging.pattern.console=%d{yyyy-MM-dd} [%thread] %-5level %logger{50} - %msg%n
# 指定文件中日志输出的格式
logging.pattern.file=%d{yyyy-MM-dd} = [%thread] = %-5level = %logger{50}  %msg%n

| logging.file | logging.path | Example  | Description                        |
| ------------ | ------------ | -------- | ---------------------------------- |
| (none)       | (none)       |          | 只在控制台输出                     |
| 指定文件名   | (none)       | my.log   | 输出日志到my.log文件               |
| (none)       | 指定目录     | /var/log | 输出到指定目录的 spring.log 文件中 |

### 2、指定配置

给类路径下放上每个日志框架自己的配置文件即可；SpringBoot就不使用他默认配置的了

| Logging System          | Customization                                                |
| ----------------------- | ------------------------------------------------------------ |
| Logback                 | logback-spring.xml, logback-spring.groovy, logback.xml or logback.groovy |
| Log4j2                  | log4j2-spring.xml or log4j2.xml                              |
| JDK (Java Util Logging) | logging.properties                                           |

logback.xml：直接就被日志框架识别了；

logback-spring.xml：日志框架就不直接加载日志的配置项，由SpringBoot解析日志配置，可以使用SpringBoot的高级Profile功能

 name="staging">

 	可以指定某段配置只在某个环境下生效


如：

 name="stdout" class="ch.qos.logback.core.ConsoleAppender">
    
    日志输出格式：
    		%d表示日期时间，
    		%thread表示线程名，
    		%-5level：级别从左显示5个字符宽度
    		%logger{50} 表示logger名字最长50个字符，否则按照句点分割。 
    		%msg：日志消息，
    		%n是换行符
    -->
     class="ch.qos.logback.classic.PatternLayout">
       name="dev">
        %d{yyyy-MM-dd HH:mm:ss.SSS} ----> [%thread] ---> %-5level %logger{50} - %msg%n
      
       name="!dev">
        %d{yyyy-MM-dd HH:mm:ss.SSS}  [%thread]  %-5level %logger{50} - %msg%n


​    




如果使用logback.xml作为日志配置文件，还要使用profile功能，会有以下错误

 no applicable action for [springProfile]

#  五、Web开发

## 1、开发流程

1）、创建SpringBoot应用，选中我们需要的模块；

2）、SpringBoot已经默认将这些场景配置好了，只需要在配置文件中指定少量配置就可以运行起来

3）、自己编写业务代码；

自动配置原理：Web场景SpringBoot帮我们配置了什么？能不能修改？能修改哪些配置？能不能扩展？xxx

xxxxAutoConfiguration：帮我们给容器中自动配置组件；
xxxxProperties:配置类来封装配置文件的内容；

 

## 2、SpringMVC自动配置

### 2.1. Spring MVC auto-configuration

Spring Boot 自动配置好了SpringMVC

以下是SpringBoot对SpringMVC的默认配置:（WebMvcAutoConfiguration）

• ContentNegotiatingViewResolver and BeanNameViewResolver beans.

– 自动配置了ViewResolver（视图解析器：根据方法的返回值得到视图对象（View），视图对象决定如何渲染（转发？重定向？））

– ContentNegotiatingViewResolver：组合所有的视图解析器的；

– 如何定制：我们可以自己给容器中添加一个视图解析器；自动的将其组合进来；

• Support for serving static resources, including support for WebJars.静态资源文件夹路径,webjars

• RequestMappingHandlerAdapter：请求映射处理器

• 自动注册了 of Converter, GenericConverter, Formatter beans.

– Converter：转换器； public String hello(User user)：类型转换使用Converter

– Formatter 格式化器； 2019.09.10=Date；

​		@Bean
​		@ConditionalOnProperty(prefix = "spring.mvc", name = "date-format")//在文件中配置日期格式化的规则
​		public Formatter<Date> dateFormatter() {
​			return new DateFormatter(this.mvcProperties.getDateFormat());//日期格式化组件
​		}

 自己添加的格式化器转换器只需要放在容器中即可

• Support for HttpMessageConverters.

– HttpMessageConverter：SpringMVC用来转换Http请求和响应的；User---Json；

– HttpMessageConverters 是从容器中确定；获取所有的HttpMessageConverter；

  自己给容器中添加HttpMessageConverter，只需要将自己的组件注册容器中（@Bean,@Component）

 

### 2.2、扩展SpringMVC

   path="/hello" view-name="success"/>


       path="/hello"/>


​    


编写一个配置类（@Configuration），是WebMvcConfigurerAdapter类型；不能标注@EnableWebMvc;

既保留了所有的自动配置，也能用我们扩展的配置；

//使用WebMvcConfigurerAdapter可以来扩展SpringMVC的功能
@Configuration
public class MyMvcConfig extends WebMvcConfigurerAdapter {

  @Override
  public void addViewControllers(ViewControllerRegistry registry) {
    // super.addViewControllers(registry);
    //浏览器发送 /hello 请求来到 success
    registry.addViewController("/hello").setViewName("success");
  }
}

原理：

 1）、WebMvcAutoConfiguration是SpringMVC的自动配置类

 2）、在做其他自动配置时会导入；@Import(EnableWebMvcConfiguration.class)

  @Configuration
	public static class EnableWebMvcConfiguration extends DelegatingWebMvcConfiguration {
	}

 3）、容器中所有的WebMvcConfigurer都会一起起作用；

 4）、我们的配置类也会被调用；

 效果：SpringMVC的自动配置和我们的扩展配置都会起作用；

 

### 2.3、拦截器进行登陆检查

拦截器
/
  登陆检查，
 /
public class LoginHandlerInterceptor implements HandlerInterceptor {
  //目标方法执行之前
  @Override
  public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
    Object user = request.getSession().getAttribute("loginUser");
    if(user  null){
      //未登陆，返回登陆页面
      request.setAttribute("msg","没有权限请先登陆");
      request.getRequestDispatcher("/index.html").forward(request,response);
      return false;
    }else{
      //已登陆，放行请求
      return true;
    }

  }

  @Override
  public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {

  }

  @Override
  public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {

  }
}

注册拦截器

 //所有的WebMvcConfigurerAdapter组件都会一起起作用
  @Bean //将组件注册在容器
  public WebMvcConfigurerAdapter webMvcConfigurerAdapter(){
    WebMvcConfigurerAdapter adapter = new WebMvcConfigurerAdapter() {
      //注册拦截器
      @Override
      public void addInterceptors(InterceptorRegistry registry) {
        //super.addInterceptors(registry);
        //静态资源；  .css , .js
        //SpringBoot已经做好了静态资源映射
        registry.addInterceptor(new LoginHandlerInterceptor()).addPathPatterns("/")
            .excludePathPatterns("/index.html","/","/user/login");
      }
    };
    return adapter;
  }

 

### 2.4、全面接管SpringMVC；

SpringBoot对SpringMVC的自动配置不需要了，所有都是我们自己配置；所有的SpringMVC的自动配置都失效了

我们需要在配置类中添加@EnableWebMvc即可；

//使用WebMvcConfigurerAdapter可以来扩展SpringMVC的功能
@EnableWebMvc
@Configuration
public class MyMvcConfig extends WebMvcConfigurerAdapter {

  @Override
  public void addViewControllers(ViewControllerRegistry registry) {
    // super.addViewControllers(registry);
    //浏览器发送 /hello 请求来到 success
    registry.addViewController("/hello").setViewName("success");
  }
}

原理：为什么@EnableWebMvc自动配置就失效了；

1）@EnableWebMvc的核心

@Import(DelegatingWebMvcConfiguration.class)
public @interface EnableWebMvc {

2）、

@Configuration
public class DelegatingWebMvcConfiguration extends WebMvcConfigurationSupport {

3）、

@Configuration
@ConditionalOnWebApplication
@ConditionalOnClass({ Servlet.class, DispatcherServlet.class,
		WebMvcConfigurerAdapter.class })
//容器中没有这个组件的时候，这个自动配置类才生效
@ConditionalOnMissingBean(WebMvcConfigurationSupport.class)
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE + 10)
@AutoConfigureAfter({ DispatcherServletAutoConfiguration.class,
		ValidationAutoConfiguration.class })
public class WebMvcAutoConfiguration {

4）、@EnableWebMvc将WebMvcConfigurationSupport组件导入进来；

5）、导入的WebMvcConfigurationSupport只是SpringMVC最基本的功能；

 

## 3、修改SpringBoot的默认配置

模式：

 1）、SpringBoot在自动配置很多组件的时候，先看容器中有没有用户自己配置的（@Bean、@Component）如果有就用用户配置的，如果没有，才自动配置；如果有些组件可以有多个（ViewResolver）将用户配置的和自己默认的组合起来；

 2）、在SpringBoot中会有非常多的xxxConfigurer帮助我们进行扩展配置

 3）、在SpringBoot中会有很多的xxxCustomizer帮助我们进行定制配置

 

## 5、错误处理机制

### 5.1、SpringBoot默认的错误处理机制

 1）、浏览器，返回一个默认的错误页面

![img](file:///C:UsersADMINI~1AppDataLocalTempksohtmlwpsB911.tmp.jpg) 

 ErrorMvcAutoConfiguration；错误处理的自动配置；

 给容器中添加了以下组件

 1、DefaultErrorAttributes：
@Bean
@ConditionalOnMissingBean(value = ErrorAttributes.class, search = SearchStrategy.CURRENT)
public DefaultErrorAttributes errorAttributes() {
  return new DefaultErrorAttributes(this.serverProperties.getError().isIncludeException());
}

 2、BasicErrorController：处理默认/error请求

@Controller
@RequestMapping("${server.error.path:${error.path:/error}}")
public class BasicErrorController extends AbstractErrorController {

  @RequestMapping(produces = "text/html")//产生html类型的数据；浏览器发送的请求来到这个方法处理
	public ModelAndView errorHtml(HttpServletRequest request,
			HttpServletResponse response) {
		HttpStatus status = getStatus(request);
		Map<String, Object> model = Collections.unmodifiableMap(getErrorAttributes(
				request, isIncludeStackTrace(request, MediaType.TEXT_HTML)));
		response.setStatus(status.value());
    
    //去哪个页面作为错误页面；包含页面地址和页面内容
    	ModelAndView modelAndView = resolveErrorView(request, response, status, model);
    	return (modelAndView  null ? new ModelAndView("error", model) : modelAndView);
    }

​	@RequestMapping
​	@ResponseBody  //产生json数据，其他客户端来到这个方法处理；
​	public ResponseEntity<Map<String, Object>> error(HttpServletRequest request) {
​		Map<String, Object> body = getErrorAttributes(request,
​				isIncludeStackTrace(request, MediaType.ALL));
​		HttpStatus status = getStatus(request);
​		return new ResponseEntity<Map<String, Object>>(body, status);
​	}

 

 3、ErrorPageCustomizer：

​	@Value("${error.path:/error}")
​	private String path = "/error";  系统出现错误以后来到error请求进行处理；

 

 4、DefaultErrorViewResolver：

 步骤：

 一但系统出现4xx或者5xx之类的错误；ErrorPageCustomizer就会生效（定制错误的响应规则）；就会来到/error请求；就会被BasicErrorController处理；

 1）响应页面；去哪个页面是由DefaultErrorViewResolver解析得到的；

protected ModelAndView resolveErrorView(HttpServletRequest request,
   HttpServletResponse response, HttpStatus status, Map<String, Object> model) {
  //所有的ErrorViewResolver得到ModelAndView
  for (ErrorViewResolver resolver : this.errorViewResolvers) {
   ModelAndView modelAndView = resolver.resolveErrorView(request, status, model);
   if (modelAndView != null) {
     return modelAndView;
   }
  }
  return null;
}

### 5.2、自定义错误的json数据：

 1）、自定义异常处理&返回自定义json数据；

@ControllerAdvice
public class MyExceptionHandler {

  @ResponseBody
  @ExceptionHandler(UserNotExistException.class)
  public Map<String,Object> handleException(Exception e){
    Map<String,Object> map = new HashMap<>();
    map.put("code","user.notexist");
    map.put("message",e.getMessage());
    return map;
  }
}
//没有自适应效果...

 

 2）、转发到/error进行自适应响应效果处理

 @ExceptionHandler(UserNotExistException.class)
  public String handleException(Exception e, HttpServletRequest request){
    Map<String,Object> map = new HashMap<>();
    //传入我们自己的错误状态码  4xx 5xx，否则就不会进入定制错误页面的解析流程
    /
      Integer statusCode = (Integer) request
     .getAttribute("javax.servlet.error.status_code");
     /
    request.setAttribute("javax.servlet.error.status_code",500);
    map.put("code","user.notexist");
    map.put("message",e.getMessage());
    //转发到/error
    return "forward:/error";
  }

#### 3）、将自定义数据携带出去；

出现错误以后，会来到/error请求，会被BasicErrorController处理，响应出去可以获取的数据是由getErrorAttributes得到的（是AbstractErrorController（ErrorController）规定的方法）；

 1、完全来编写一个ErrorController的实现类【或者是编写AbstractErrorController的子类】，放在容器中；

 2、页面上能用的数据，或者是json返回能用的数据都是通过errorAttributes.getErrorAttributes得到；

 容器中DefaultErrorAttributes.getErrorAttributes()；默认进行数据处理的；

自定义ErrorAttributes

//给容器中加入我们自己定义的ErrorAttributes
@Component
public class MyErrorAttributes extends DefaultErrorAttributes {

  @Override
  public Map<String, Object> getErrorAttributes(RequestAttributes requestAttributes, boolean includeStackTrace) {
    Map<String, Object> map = super.getErrorAttributes(requestAttributes, includeStackTrace);
    map.put("company","hello");
    return map;
  }
}

通过自定义ErrorAttributes改变需要返回的内容，

 

## 6、配置嵌入式Servlet容器

SpringBoot默认使用Tomcat作为嵌入式的Servlet容器；

![img](file:///C:UsersADMINI~1AppDataLocalTempksohtmlwpsB922.tmp.jpg) 

 

### 1）、修改Servlet容器的相关配置；

1、修改和server有关的配置（ServerProperties）；

server.port=8081
server.context-path=/crud

server.tomcat.uri-encoding=UTF-8

//通用的Servlet容器设置
server.xxx
//Tomcat的设置
server.tomcat.xxx

### 2）、注册Servlet三大组件【Servlet、Filter、Listener】

由于SpringBoot默认是以jar包的方式启动嵌入式的Servlet容器来启动SpringBoot的web应用，没有web.xml文件。

注册三大组件用以下方式

ServletRegistrationBean

//注册三大组件
@Bean
public ServletRegistrationBean myServlet(){
  ServletRegistrationBean registrationBean = new ServletRegistrationBean(new MyServlet(),"/myServlet");
  return registrationBean;
}

FilterRegistrationBean

@Bean
public FilterRegistrationBean myFilter(){
  FilterRegistrationBean registrationBean = new FilterRegistrationBean();
  registrationBean.setFilter(new MyFilter());
  registrationBean.setUrlPatterns(Arrays.asList("/hello","/myServlet"));
  return registrationBean;
}

ServletListenerRegistrationBean

@Bean
public ServletListenerRegistrationBean myListener(){
  ServletListenerRegistrationBean<MyListener> registrationBean = new ServletListenerRegistrationBean<>(new MyListener());
  return registrationBean;
}

SpringBoot帮我们自动配置SpringMVC的时候，自动的注册SpringMVC的前端控制器；DIspatcherServlet；

DispatcherServletAutoConfiguration中：

@Bean(name = DEFAULT_DISPATCHER_SERVLET_REGISTRATION_BEAN_NAME)
@ConditionalOnBean(value = DispatcherServlet.class, name = DEFAULT_DISPATCHER_SERVLET_BEAN_NAME)
public ServletRegistrationBean dispatcherServletRegistration(
   DispatcherServlet dispatcherServlet) {
  ServletRegistrationBean registration = new ServletRegistrationBean(
     dispatcherServlet, this.serverProperties.getServletMapping());
  //默认拦截： /  所有请求；包静态资源，但是不拦截jsp请求；  /会拦截jsp
  //可以通过server.servletPath来修改SpringMVC前端控制器默认拦截的请求路径

  registration.setName(DEFAULT_DISPATCHER_SERVLET_BEAN_NAME);
  registration.setLoadOnStartup(
     this.webMvcProperties.getServlet().getLoadOnStartup());
  if (this.multipartConfig != null) {
   registration.setMultipartConfig(this.multipartConfig);
  }
  return registration;
}

#  六、SpringBoot与数据访问

## 1、JDBC

<dependency>	<groupId>org.springframework.boot</groupId>	<artifactId>spring-boot-starter-jdbc</artifactId></dependency><dependency>	<groupId>mysql</groupId>	<artifactId>mysql-connector-java</artifactId>	<scope>runtime</scope></dependency>

spring:
 datasource:
  username: root
  password: 123456
  url: jdbc:mysql://127.0.0.1:3306/jdbc
  driver-class-name: com.mysql.jdbc.Driver

 默认是用org.apache.tomcat.jdbc.pool.DataSource作为数据源；

 数据源的相关配置都在DataSourceProperties里面；

自动配置原理：

org.springframework.boot.autoconfigure.jdbc：

1、参考DataSourceConfiguration，根据配置创建数据源，默认使用Tomcat连接池；可以使用spring.datasource.type指定自定义的数据源类型；

2、SpringBoot默认可以支持；

org.apache.tomcat.jdbc.pool.DataSource、HikariDataSource、BasicDataSource、

3、自定义数据源类型

/
  Generic DataSource configuration.
 /
@ConditionalOnMissingBean(DataSource.class)
@ConditionalOnProperty(name = "spring.datasource.type")
static class Generic {

  @Bean
  public DataSource dataSource(DataSourceProperties properties) {
    //使用DataSourceBuilder创建数据源，利用反射创建响应type的数据源，并且绑定相关属性
   return properties.initializeDataSourceBuilder().build();
  }

}

4、DataSourceInitializer：ApplicationListener；

 作用：

 1）、runSchemaScripts();运行建表语句；

 2）、runDataScripts();运行插入数据的sql语句；

默认只需要将文件命名为：

schema-.sql、data-.sql
默认规则：schema.sql，schema-all.sql；
可以使用  
	schema:
   - classpath:department.sql
      指定位置

5、操作数据库：自动配置了JdbcTemplate操作数据库

## 2、整合Druid数据源

导入druid数据源
@Configuration
public class DruidConfig {

  @ConfigurationProperties(prefix = "spring.datasource")
  @Bean
  public DataSource druid(){
    return new DruidDataSource();
  }

  //配置Druid的监控
  //1、配置一个管理后台的Servlet
  @Bean
  public ServletRegistrationBean statViewServlet(){
    ServletRegistrationBean bean = new ServletRegistrationBean(new StatViewServlet(), "/druid/");
    Map<String,String> initParams = new HashMap<>();

​    initParams.put("loginUsername","admin");
​    initParams.put("loginPassword","123456");
​    initParams.put("allow","");//默认就是允许所有访问
​    initParams.put("deny","127.0.0.1");

​    bean.setInitParameters(initParams);
​    return bean;
  }


  //2、配置一个web监控的filter
  @Bean
  public FilterRegistrationBean webStatFilter(){
    FilterRegistrationBean bean = new FilterRegistrationBean();
    bean.setFilter(new WebStatFilter());

​    Map<String,String> initParams = new HashMap<>();
​    initParams.put("exclusions",".js,.css,/druid/");

​    bean.setInitParameters(initParams);

​    bean.setUrlPatterns(Arrays.asList("/"));

​    return bean;
  }
}

## 3、整合MyBatis

​		
​			org.mybatis.spring.boot
​			mybatis-spring-boot-starter
​			1.3.1
​		

![img](file:///C:UsersADMINI~1AppDataLocalTempksohtmlwpsB923.tmp.jpg) 

 

步骤：

 1）、配置数据源相关属性

 2）、给数据库建表

 3）、创建JavaBean

### 4）、注解版

//指定这是一个操作数据库的mapper
@Mapper
public interface DepartmentMapper {

  @Select("select  from department where id=#{id}")
  public Department getDeptById(Integer id);

  @Delete("delete from department where id=#{id}")
  public int deleteDeptById(Integer id);

  @Options(useGeneratedKeys = true,keyProperty = "id")
  @Insert("insert into department(departmentName) values(#{departmentName})")
  public int insertDept(Department department);

  @Update("update department set departmentName=#{departmentName} where id=#{id}")
  public int updateDept(Department department);
}

问题：

自定义MyBatis的配置规则；给容器中添加一个ConfigurationCustomizer；

@org.springframework.context.annotation.Configuration
public class MyBatisConfig {

  @Bean
  public ConfigurationCustomizer configurationCustomizer(){
    return new ConfigurationCustomizer(){

​      @Override
​      public void customize(Configuration configuration) {
​        configuration.setMapUnderscoreToCamelCase(true);
​      }
​    };
  }
}

 

使用MapperScan批量扫描所有的Mapper接口；
@MapperScan(value = "com.hello.springboot.mapper")
@SpringBootApplication
public class SpringBoot06DataMybatisApplication {

​	public static void main(String[] args) {
​		SpringApplication.run(SpringBoot06DataMybatisApplication.class, args);
​	}
}

### 5）、配置文件版

mybatis:
 config-location: classpath:mybatis/mybatis-config.xml 指定全局配置文件的位置
 mapper-locations: classpath:mybatis/mapper/.xml  指定sql映射文件的位置

 