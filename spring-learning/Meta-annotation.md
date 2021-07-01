# 元注解

元注解（meta-annotation）的作用就是负责注解其他注解。

Java5.0 定义了 4 个标准的 meta-annotation 类型，用来提供对其它 annotation 类型作说明：
1. @Target,
2. @Retention,
3. @Documented,
4. @Inherited




## Spring中的注解

Spring 很多注解可以作为元注解使用，我们可以通过 Spring 提供的注解组合出新的复合注解（composed annotations）。

>Many of the annotations provided by Spring can be used as meta-annotations in your own code. A meta-annotation is an annotation that can be applied to another annotation. For example, the @Service annotation mentioned earlier is meta-annotated with @Component, as the following example shows:
```Java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component 
public @interface Service {
    // ...
}
```

>You can also combine meta-annotations to create “composed annotations”. For example, the @RestController annotation from Spring MVC is composed of @Controller and @ResponseBody.

>In addition, composed annotations can optionally redeclare attributes from meta-annotations to allow customization. This can be particularly useful when you want to only expose a subset of the meta-annotation’s attributes. For example, Spring’s @SessionScope annotation hardcodes the scope name to session but still allows customization of the proxyMode. The following listing shows the definition of the SessionScope annotation:
```Java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Scope(WebApplicationContext.SCOPE_SESSION)
public @interface SessionScope {

    /**
     * Alias for {@link Scope#proxyMode}.
     * Defaults to {@link ScopedProxyMode#TARGET_CLASS}.
     */
    @AliasFor(annotation = Scope.class)
    ScopedProxyMode proxyMode() default ScopedProxyMode.TARGET_CLASS;

}
```
>You can then use @SessionScope without declaring the proxyMode as follows:
```Java
@Service
@SessionScope
public class SessionScopedService {
    // ...
}
```
>You can also override the value for the proxyMode, as the following example shows:
```Java
@Service
@SessionScope(proxyMode = ScopedProxyMode.INTERFACES)
public class SessionScopedUserService implements UserService {
    // ...
}
```
For further details, see the [Spring Annotation Programming Model](https://github.com/spring-projects/spring-framework/wiki/Spring-Annotation-Programming-Model) wiki page.

# Ref
1. [Spring Framework Documentation](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#spring-core)