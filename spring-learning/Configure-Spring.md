# 配置Spring的三种方式

## 基于注解配置

当我们需要使用 BeanPostProcessor 时，例如：

使用 @Autowired 注解，必须事先在Spring容器中声明AutowiredAnnotationBeanPostProcessor的Bean：
`<bean class="org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor"/>`

使用 @Required注解，就必须声明RequiredAnnotationBeanPostProcessor的Bean：
`<bean class="org.springframework.beans.factory.annotation.RequiredAnnotationBeanPostProcessor"/>`

类似地，使用 @Resource、@PostConstruct、@PreDestroy 等注解就必须声明 CommonAnnotationBeanPostProcessor ；使用 @PersistenceContext 注解，就必须声明 PersistenceAnnotationBeanPostProcessor 的 Bean。

在基于注解方式配置 Spring 时可以使用 `<context:annotation-config/>` 简化上面的 bean 声明，`<context:annotation-config/>` 的作用是隐式的向 Spring 容器注册<sup>[1]</sup>以下 5 个 BeanPostProcessor：
- ConfigurationClassPostProcessor
- AutowiredAnnotationBeanPostProcessor
- CommonAnnotationBeanPostProcessor
- PersistenceAnnotationBeanPostProcessor
- EventListenerMethodProcessor。

注：在 Spring5.1 之前 RequiredAnnotationBeanPostProcessor 也是 `<context:annotation-config/>` 负责注册，但该类在 Spring5.1 中已废弃。

注意：`<context:annotation-config/>` 不能跨不同的上下文<sup>[1]</sup>使用：
><context:annotation-config/> only looks for annotations on beans in the same application context in which it is defined. This means that, if you put <context:annotation-config/> in a WebApplicationContext for a DispatcherServlet, it only checks for @Autowired beans in your controllers, and not your services. See [The DispatcherServlet](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-servlet) for more information.

`AnnotationConfigApplicationContext`


## 基于Java配置


 
## 基于XML配置


# REF

1. [Annotation-based Container Configuration](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-annotation-config)
2. [Java-based Container Configuration](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-java)