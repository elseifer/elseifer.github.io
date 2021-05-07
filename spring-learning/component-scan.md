
# component scan
为了实现依赖项注入，Spring 创建了一个应用上下文（ApplicationContext）。  
Spring 实例化对象并将其添加到应用上下文中，应用上下文中的对象称为 Spring bean 或组件。  
Spring 解析 Spring bean 之间的依赖关系，并将Spring bean注入到其他Spring bean的字段或构造函数中。  
在类路径中搜索配置类的过程称为组件扫描。


## @Component 

为了说明 Spring Boot 的组件扫描（component scan），先从 `@Component` 注解说起。 

以下是 `Component` 的注释中的内容：
>Indicates that an annotated class is a "component".
Such classes are considered as candidates for auto-detection
when using annotation-based configuration and classpath scanning.  
Other class-level annotations may be considered as identifying
a component as well, typically a special kind of component:
e.g. the {@link Repository @Repository} annotation or AspectJ's
{@link org.aspectj.lang.annotation.Aspect @Aspect} annotation.

主要表述了以下几点：
- 被 @Component 注解的类是 Spring 管理的组件；
- 当启用注解配置、类路径扫描时，被其注解的类会被自动检测的；
- @Component 是通用的原型注解（stereotype annotation），其他原型注解，如 @Repository、@Service 和 @Controller 是 @Component 的特例化；


@Component 解析

`@Component`、`Repository` 、`Service` 和 `Controller` 均是由 `org.springframework.context.annotation.ClassPathBeanDefinitionScanner` 处理。

### Bean 的名称
AnnotationBeanNameGenerator 名字生成器，如果不指定 Component 的 value 属性，默认 bean name 为短类名，详见 `java.beans.Introspector#decapitalize`，例如

java.example.MyRepository 的 bean 名称为 myRepository  
java.example.DTMRepository 的 bean 名称为 DTMRepository

![component-defaule-bean-name](images/component-defaule-bean-name.png)  

```xml
<context:annotation-config />
<context:component-scan />
```

在基于注解方式配置Spring时，Spring 配置文件 applicationContext.xml 中有 <context:annotation-config/> ，它的作用是隐式的向Spring容器注册
AutowiredAnnotationBeanPostProcessor,CommonAnnotationBeanPostProcessor,PersistenceAnnotationBeanPostProcessor,
RequiredAnnotationBeanPostProcessor 

`@ComponentScan` 

## 如何理解 stereotype 

[what-is-a-spring-stereotype](https://stackoverflow.com/questions/14756486/what-is-a-spring-stereotype)


# auto-detection

ComponentScan

```xml
<context:component-scan />
```


# auto-configuration


EnableAutoConfiguration


### bean 可以被重复加载么？


如果允许重复加载 bean ，可以在 spring properties 中添加 `spring.main.allow-bean-definition-overriding=true` 配置


## SpringBoot 的装配几种装配方式
 
基于 xml 的配置 
基于 注解 配置
基于 java 配置


# REF
1. [spring-component-scanning](https://reflectoring.io/spring-component-scanning/)