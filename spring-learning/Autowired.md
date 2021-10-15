
# Autowired 注解

`@Autowired` `@Value` 和 `@Lookup` 均是 `AutowiredAnnotationBeanPostProcessor` 负责处理的。

## Value 注解

@Value("#{datasource.url}")   SpEl表达式通常用来获取bean的属性，或者调用bean的某个方法，例如 datasource 的 url 属性

@Value("${test.sever.host}")  从配置文件读取值


## default-autowire
在 xml 头中设置 `default-autowire="byName"`



## Ref

https://howtodoinjava.com/spring-core/spring-beans-autowiring-concepts/


# 多实例

### Lookup 注解

@Lookup 注解告诉 spring，在每次调用被 @Lookup 注解的方法时，依据该方法的返回值类型，返回一个新的该类型的实例。它的定义如下：
```java
@Target({ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Lookup {
    String value() default "";
}
```
可以看到它只作用于方法放上，例如：
```java
@Component
public class SingleBean {
    public void print() {
        PrototypeBean bean = getBean(); 
        bean.say();
    }
    @Lookup
    public PrototypeBean getBean() {
        return null;
    }
}

@Component
@Scope("prototype")
public class PrototypeBean {
    public void say() {
        System.out.println("i am prototype bean");
    }
}
```


```java
<public|protected> [abstract] <return-type> theMethodName (no-arguments);
```


@Lookup 等效于 xml 中的 lookup-method 元素。

#### Scope 注解

[Bean scopes](https://docs.spring.io/spring-framework/docs/3.0.0.M3/reference/html/ch04s04.html?)  

`@Scope` 的 value 默认为`""`，即
`ConfigurableBeanFactory#SCOPE_SINGLETON` (singleton) ，也就是我们常说的单例，它一般有如下取值：
+ `ConfigurableBeanFactory#SCOPE_SINGLETON` 
+ `ConfigurableBeanFactory#SCOPE_PROTOTYPE` (prototype)
+ `WebApplicationContext#SCOPE_REQUEST`
+ `WebApplicationContext#SCOPE_SESSION`  

示例：

```java
    @Component
    @Scope("prototype)
    public class Bean1
```
```xml
<bean id="bean1" class="Bean" scope="prototype"/>  
```
两者是等效，Bean1 都是原型的，而非spring默认的单例的。

+ AnnotationScopeMetadataResolver
