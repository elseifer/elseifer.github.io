# Configuration 注解
主要面向 Spring Boot 介绍。

## Configuration 注解的作用

从 Spring3.0 开始，`@Configuration` 可用于定义配置类，可替换 xml 配置文件，配置类中声明了一个或多个被 @Bean 注解的方法，Spring 容器会处理这些方法以构建 bean 定义和处理 bean 的请求。  
配置类会被 AnnotationConfigApplicationContext 或 AnnotationConfigWebApplicationContext （web 环境）处理<sup>[1]</sup>，SpringBoot Web 环境下由 AnnotationConfigServletWebServerApplicationContext 处理配置类。

### 什么是 Spring 配置类

Spring 配置类用来注册 bean，它效果与 xml 文件是一样的，只是创建 Spring IoC 容器的方式不同：
```java
//通过xml文件创建springIOC容器
ApplicationContext applicationContext = new ClassPathXmlApplicationContext("/spring-beans.xml");
//通过配置类创建springIOC容器
ApplicationContext applicationContext = new AnnotationConfigApplicationContext(BeanConfig.class);
```

如果一个类中有以下任意注解之一就属于配置类：  
- 类上有 `@Configuration` 注解  
- 类上有 `@Compontent` 注解  
- 类上有 `@CompontentScan` 注解  
- 类上有 `@Import` 注解  
- 类上有 `@ImportResource` 注解  
- 类中有 `@Bean` 标注的方法

判断一个类是否为一个配置类，可以看一下 `org.springframework.context.annotation.ConfigurationClassUtils#isConfigurationCandidate` 方法，Spring 中处理这个过程会循环进行，直到完成所有配置类的解析及所有bean的注册。 

`@Configuration` 的 Java Doc 知道 `@Configuration` 由 `@Component` 注解，因此被 `@Configuration` 注解的类会被自动检测（auto-detection）处理。

>@Configuration is meta-annotated with @Component, therefore @Configuration classes are candidates for component scanning (typically using Spring XML's <context:component-scan/> element) and therefore may also take advantage of @Autowired/@Inject like any regular @Component.


### 配置类处理器
ConfigurationClassPostProcessor 负责解析和处理被 `@Configuration` 注解的类（更具体的讲是 postProcessBeanDefinitionRegistry 方法）。

> BeanFactoryPostProcessor used for bootstrapping processing of `@Configuration` classes.  
Registered by default when using `<context:annotation-config/>` or `<context:component-scan/>`. Otherwise, may be declared manually as with any other BeanFactoryPostProcessor.  
This post processor is priority-ordered as it is important that any Bean methods declared in `@Configuration` classes have their corresponding bean definitions registered before any other BeanFactoryPostProcessor executes.

这里先埋下一个知识点： ConfigurationClassPostProcessor 需要配置 `<context:annotation-config/>` 、`<context:component-scan/>`，或者手动声明 `<bean class='org.springframework.context.annotation.ConfigurationClassPostProcessor'/>`。

#### 解析配置类的过程
ConfigurationClassPostProcessor 首先会判断在 ApplicationContext 中的 bean 是否被 `@Configuration` 注解标记，然后使用 ConfigurationClassParser 来解析 `@Configuration`。

ConfigurationClassPostProcessor 解析配置类的大致流程：
  1. 使用 `ConfigurationClassUtils.checkConfigurationClassCandidate` 检查 BeanDefinition 是否有 `@Configuration` 注解标记
  2. 对配置类进行排序
  3. 使用 ConfigurationClassParser 解析 `@Configuration` 注解的信息
  4. 使用 ConfigurationClassBeanDefinitionReader 解析 BeanDefinition

其中 `ConfigurationClassUtils.checkConfigurationClassCandidate` 的判断逻辑与下列代码类似，`AnnotatedElementUtils.isAnnotated` 递归遍历所有注解：
```java
//MyApplication 是启动类
AnnotatedElementUtils.isAnnotated(MyApplication.class, Configuration.class.getName());
```
#### Spring何时处理配置类 

ConfigurationClassPostProcessor 又由谁来调用呢？  

ApplicationContext 在 `org.springframework.context.support.AbstractApplicationContext#invokeBeanFactoryPostProcessors` 中触发 `ConfigurationClassPostProcessor#postProcessBeanDefinitionRegistry` 的调用。

具体是哪个 ApplicationContext 是由 `org.springframework.boot.SpringApplication#createApplicationContext` 判断并创建，这里以 SpringBoot 为例：

```java
protected ConfigurableApplicationContext createApplicationContext() {
  Class<?> contextClass = this.applicationContextClass;
  if (contextClass == null) {
    try {
      switch (this.webApplicationType) {
      case SERVLET:
        contextClass = Class.forName(DEFAULT_SERVLET_WEB_CONTEXT_CLASS);
        break;
      case REACTIVE:
        contextClass = Class.forName(DEFAULT_REACTIVE_WEB_CONTEXT_CLASS);
        break;
      default:
        contextClass = Class.forName(DEFAULT_CONTEXT_CLASS);
      }
    }
    catch (ClassNotFoundException ex) {
      throw new IllegalStateException(
          "Unable create a default ApplicationContext, "
              + "please specify an ApplicationContextClass",
          ex);
    }
  }
  return (ConfigurableApplicationContext) BeanUtils.instantiateClass(contextClass);
}
```

### SpringBootApplication 注解

在SpringBoot 中额外提供了一些配置类相关的注解，例如 SpringBoot 应用的入口启动类一般是如下形式：

```java 
@SpringBootApplication
public class MyApplication {
  SpringApplication.run(MyApplication.class, args);
}
```

在 SpringBoot 设计中 `SpringBootConfiguration` 来代替 `Configuration`，在使用 `SpringBootApplication` 时会自动带入 `SpringBootConfiguration` 注解

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Configuration
public @interface SpringBootConfiguration {}
```

`@SpringBootApplication` 由如下几个注解构成：
```mermaid
graph TD
  A(SpringBootApplication)-->B1(SpringBootConfiguration)
  A-->B2(EnableAutoConfiguration)
  A-->B3(ComponentScan)
  A-->B4(Inherited)
  B1-->C1(Configuration)
  C1-->C2(Component)
```

即一个 SpringBootApplication 注解具备 Configuration、ComponentScan、EnableAutoConfiguration 三者的功能：注册Bean、组件自动扫描、自动配置。

## ComponentScan 注解

这里填下前文的一个知识点：ConfigurationClassPostProcessor 是需要配置的，例如在 xml 中添加 `<context:component-scan/>` 元素，是不是仅能通过 xml 配置呢? 答案自然是否定的。

这里引用下 `@ComponentScan` 的注释：
>Configures component scanning directives for use with @Configuration classes. Provides support parallel with Spring XML's <context:component-scan> element.  
Note that the `<context:component-scan>` element has an annotation-config attribute; however, this annotation does not. This is because in almost all cases when using @ComponentScan, default annotation config processing (e.g. processing @Autowired and friends) is assumed. Furthermore, when using AnnotationConfigApplicationContext, annotation config processors are always registered, meaning that any attempt to disable them at the @ComponentScan level would be ignored.

`@ComponentScan` 注解可以代替 `<context:component-scan/>` 配置，`@ComponentScan` 同样会注册 Annotation Config 相关的处理器。

我们结合前文中提及 `@SpringBootApplication` 同时被 `@SpringBootConfiguration` 和 `@ComponentScan` 两个注解所标记，这就可以理解在使用 SpringBoot 的过程中，我们通常无需关心 ConfigurationClassPostProcessor 的配置了。

## EnableAutoConfiguration 注解
Spring Boot 提供了 `@EnableAutoConfiguration` 用于激活自动配置（auto-configuration），简单理解 `@Configuration` 标识了 Spring 配置类，`@ComponentScan` 告诉 Spring 扫描哪些配置类，而 `@EnableAutoConfiguration` 则告诉 Spring 可以使用或者不使用哪些配置类。
我理解这三者分别表述了：配置类是什么？有哪些？用哪些？

### 再谈 SpringBootApplication 注解

```java
@SpringBootApplication(scanBasePackages={}, 
  exclude = {DataSourceAutoConfiguration.class, DataSourceTransactionManagerAutoConfiguration.class, HibernateJpaAutoConfiguration.class},)
public class MyApplication {}
```
- scanBasePackages 属性对应 ComponentScan 的 basePackages 属性；
- exclude 属性对应 EnableAutoConfiguration 的 exclude 属性；

简单的实现了 ComponentScan、EnableAutoConfiguration 的组件扫描和自动配置的能力。

## 扫描配置类的其他方式

不论是 XML 或者配置类均是 ConfigurationClassPostProcessor 等处理的，配置类的扫描是按照 `<context:component-scan/>` 和 `@ComponentScan` 的 basePackages 属性值来取得扫描路径，从而扫描出 classpath 下所有需要解析的配置类。

如果需要被 Ioc 容器管理的 Bean 的路径不在 SpringBoot 的包扫描路径下怎么办呢，即如何加载第三方的 Bean 呢？

### 方式1：spring.factories

一种类似 spi 方式来引入配置类：spring.factories，该文件位于 classpatch:META-INF 目录中，里面的内容如下：

```properties
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  elseifer.example.service.MyAutoConfiguraion

org.springframework.context.ApplicationContextInitializer=\
  org.springframework.boot.context.ContextIdApplicationContextInitializer
``` 
它不单单可以指定配置类，还可以指定其他 Spring 提供的扩展类，例如 ApplicationContextInitializer 等等。

### 方式2：@Import

在 SpringBoot Application 启动类（或者 Spring 配置类）上使用 @Import 注解，它的等价于 `<import/>`

```java
@Import({MyConfig.class})
@SpringBootApplication
public class MyApplication {...}
```

额外提及下 `@ImportResource`，同 `@Import` 的使用方式，用于加载 xml 配置，提供类似 xml 中的 </import> 标签。

```java
@ImportResource({"classpath*:test/META-INF/example/*.xml"})
@SpringBootApplication
public class MyApplication {...}
```

## 排除配置类
除了使用 EnableAutoConfiguration 和 SpringBootApplication 的 exclude 属性进行排除配置类，还可以通过配置文件指定参数的形式：

properties 文件指定
```properties
spring.autoconfigure.exclude=org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration
```

yml 文件指定：
```yml
spring:     
  autoconfigure:
    exclude:
      - org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration
```

## End
到此，我们介绍了配置类、配置类的注解及其原理等内容，简单回顾：
1. 配置类可以代替 xml 来注册 Bean，使用 `@Configuration` 等注解声明配置类；
    - `@Configuation` 等价于 `<Beans></Beans>`
    - `@Bean` 等价于 `<Bean></Bean>`
    - `@ComponentScan` 等价于 `<context:component-scan/>`
2. 配置类由 `ConfigurationClassPostProcessor` 负责解析；
3. 使用 `@ComponentScan` 时 SpringBoot 将默认注册 `ConfigurationClassPostProcessor`；
4. 第三方或者非默认路径的配置类需要引入时，可以使用 `spring.factories`、`@Import`；

## REF
1. [Spring Framework Reference](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html)  
2. [componentscan&enableautoconfiguration](https://www.baeldung.com/spring-componentscan-vs-enableautoconfiguration)  
3. [Aggregating @Configuration classes with @Import](https://docs.spring.io/spring-javaconfig/docs/1.0.0.M4/reference/html/ch04s03.html)