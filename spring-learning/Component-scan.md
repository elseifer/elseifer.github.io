
# Spring 组件扫描
为了实现依赖项注入，Spring 创建了 application context（应用上下文）  
启动过程中，Spring 实例化对象并将其添加到应用上下文中，应用上下文中的对象称为 Spring beans 或 components  
Spring 解析 beans 之间的依赖关系，并将 beans 注入到其他 beans 的字段或构造函数中  
在类路径中搜索配置类的过程称为组件扫描<sup>[1]</sup>  

SpringBoot2 基于 Spring5 核心包，这里我们以 SpringBoot2 为例。

## @Component 注解
`@Component` 注解的类是 Spring 配置类，为了说明 Spring Boot 的组件扫描（component scan），先从 `@Component` 说起。 

以下是 `Component` 的注释中的内容：
>Indicates that an annotated class is a "component".
Such classes are considered as candidates for auto-detection
when using annotation-based configuration and classpath scanning.  
Other class-level annotations may be considered as identifying
a component as well, typically a special kind of component:
e.g. the {@link Repository @Repository} annotation or AspectJ's
{@link org.aspectj.lang.annotation.Aspect @Aspect} annotation.

主要表述了以下几点：
- 被 @Component 注解的类是 Spring 管理的组件；
- 当启用基于注解配置、类路径扫描时，被其注解的类会被自动检测的；
- 如果类被 @Repository、@Aspect 标记，也会被当做 spring 组件；

`@Component` 是通用的原型注解（stereotype annotation），其他原型注解，如 `@Repository`、`@Service` 和 `@Controller` 是 `@Component` 的特例；  

`@Component`、`Repository` 、`Service` 和 `Controller` 均是由 `org.springframework.context.annotation.ClassPathBeanDefinitionScanner` 处理。

[what-is-a-spring-stereotype](https://stackoverflow.com/questions/14756486/what-is-a-spring-stereotype)

## Bean 名称
工具类 AnnotationBeanNameGenerator 作为名字生成器，如果不指定 `@Component` 的 value 属性，默认 bean name 为短类名，详见 `java.beans.Introspector#decapitalize`，例如

`java.example.MyRepository` 的 bean 名称为 myRepository  
`java.example.DTMRepository` 的 bean 名称为 DTMRepository

![component-defaule-bean-name](images/component-defaule-bean-name.png)  

### 依赖注入的3种方式
- Constructor注入
- Setter注入（Field注入）
- 工厂方法注入

### bean 重复加载

如果需要允许 bean 重复加载，可以在 properties 中添加 `spring.main.allow-bean-definition-overriding=true` 配置

## Auto-detection

`@ComponentScan` 或者 `<context:component-scan/>` 会开启 Spring 基于类路径的自动扫描。

[Bean-classpath-scanning](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-classpath-scanning):

>The use of <context:component-scan> implicitly enables the functionality of <context:annotation-config>. There is usually no need to include the <context:annotation-config> element when using <context:component-scan>.

## Auto-configuration

[auto-configuration](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#using.auto-configuration):

>Spring Boot auto-configuration attempts to automatically configure your Spring application based on the jar dependencies that you have added.

`@EnableAutoConfiguration` 是 SpringBoot 提供，不在 Spring Framework 核心包中，如下是它的定义：
```Java 
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {

	String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

	Class<?>[] exclude() default {};
	
	String[] excludeName() default {};
}
```

其中 `@Import(AutoConfigurationImportSelector.class)` 很重要，通过 `AutoConfigurationImportSelector`，`@EnableAutoConfiguration` 把 SpringBoot 应用代码路径下所有符合条件的[配置类](./Configuration.md)加载到当前IoC容器中。

`AutoConfigurationImportSelector` 调用工具类 `SpringFactoriesLoader` 读取 `META-INF/spring.factories` 配置，告诉 spring 有哪些配置类需要加载。
调用 `AutoConfigurationMetadataLoader` 工具类读取 `META-INF/spring-autoconfigure-metadata.properties` 声明对应配置类需要自动装配的条件。

`exclude`、`excludeName` 排除指定的 auto-configuration 类（通过 `EnableAutoConfiguration` 或者 `/spring.factories`），注意不是配置类。
 
`spring.factories` 是一个 properties 配置文件，格式是 `Key=Value` 形式，Key 和 Value 均是 Java 的全限定类名，比如：`org.springframework.data.repository.core.support.RepositoryFactorySupport=org.springframework.data.jpa.repository.support.JpaRepositoryFactory`

# REF
1.[spring-component-scanning](https://reflectoring.io/spring-component-scanning/)