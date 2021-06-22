
## Structuring Your Code

## Configuration Classes
Spring Boot favors Java-based configuration. Although it is possible to use SpringApplication with
XML sources, we generally recommend that your primary source be a single `@Configuration` class.
Usually the class that defines the main method is a good candidate as the primary `@Configuration`.

If you don’t want to use `@SpringBootApplication`, the @`EnableAutoConfiguration` and
`@ComponentScan` annotations that it imports defines that behaviour so you can also
use those instead.

### Importing Additional Configuration Classes  
You need not put all your `@Configuration` into a single class. The `@Import` annotation can be used to
import additional configuration classes. Alternatively, you can use `@ComponentScan` to automatically
pick up all Spring components, including @Configuration classes.


### Importing XML Configuration  
If you absolutely must use XML based configuration, we recommend that you still start with a
`@Configuration` class. You can then use an `@ImportResource` annotation to load XML configuration
files.


```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {...}
```

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Configuration
public @interface SpringBootConfiguration {}
```

## Auto-configuration  
Spring Boot auto-configuration attempts to automatically configure your Spring application based
on the jar dependencies that you have added.For example, if HSQLDB is on your classpath, and you
have not manually configured any database connection beans, then Spring Boot auto-configures an
in-memory database.

You need to opt-in to auto-configuration by adding the `@EnableAutoConfiguration` or
`@SpringBootApplication` annotations to one of your `@Configuration` classes.

You should only ever add one @SpringBootApplication or `@EnableAutoConfiguration`
annotation. We generally recommend that you add one or the other to your
primary `@Configuration` class only.

### Disabling Specific Auto-configuration Classes  
Auto-configuration is non-invasive. If you need to find out what auto-configuration is currently being applied, and why, start your
application with the `--debug` switch.  
If you find that specific auto-configuration classes that you do not want are being applied, you can
use the exclude attribute of `@SpringBootApplication` to disable them, as shown in the following
example:
```java 
@SpringBootApplication(exclude={DataSourceAutoConfiguration.class})
public class MyApplication {}
```

If the class is not on the classpath, you can use the excludeName attribute of the annotation and
specify the fully qualified name instead.If you prefer to use `@EnableAutoConfiguration` rather than
`@SpringBootApplication`, exclude and excludeName are also available. Finally, you can also control the
list of auto-configuration classes to exclude by using the `spring.autoconfigure.exclude` property.

```properties
spring.autoconfigure.exclude=org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration
```

## Spring Beans and Dependency Injection

We often find that using `@ComponentScan` (to find your beans) and using
`@Autowired` (to do constructor injection) works well.  

If you structure your code as suggested above (locating your application class in a root package),
you can add `@ComponentScan` without any arguments. All of your application components (
`@Component`, `@Service`, `@Repository`, `@Controller` etc.) are automatically registered as Spring Beans.

## Using the @SpringBootApplication Annotation

Many Spring Boot developers like their apps to use auto-configuration, component scan and be able
to define extra configuration on their "application class". A single `@SpringBootApplication`
annotation can be used to enable those three features, that is:
- `@EnableAutoConfiguration`: enable Spring Boot’s auto-configuration mechanism
- `@ComponentScan:` enable `@Component` scan on the package where the application is located 
- `@Configuration`: allow to register extra beans in the context or import additional configuration classes