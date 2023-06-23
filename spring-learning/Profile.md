# Spring Profile 机制

## 默认加载顺序
SpringBoot 启动会扫描以下位置的 application.properties/yml 文件（file 指当前项目的根目录，classpath 指当前项目的 resources 目录
）作为默认配置文件：
- file:./config/
- file:./
- classpath:/config/
- classpath:/

以上是按照优先级从高到低的顺序，所有位置的文件都会被加载，高优先级配置内容会覆盖低优先级配置的内容，并形成互补配置。

用户可以通过 `spring.config.location` 来改变这一默认行为。

## 外化配置 

常见的有 `spring.config.location` 和 `spring.config.additional-location`

### -Dspring.config.location

指定外部配置文件时，此份配置文件必须满足当前工程运行时所需全量配置，因为它不会去与 resources 目录下的配置文件去做 merge 操作。

### -Dspring.config.additional-location
外化配置文件时，可以激活指定路径的配置文件，指定差异增量配置

### 配置文件生效的优先级

优先级由高到低：
- `spring.config.location` 
- `spring.profiles.active`
- `spring.config.additional-location`
- `application-default.proerties`，仅本地环境生效 
- `application.proerties`

其中通过 `spring.profiles.active` 和 `spring.config.additional-location` 指定的配置文件会与 `application.proerties` 合并以作为最终的配置，`spring.config.location` 则不会。

## 激活 profile 几种方法

### spring.profiles.active
在默认配置文件、`-D`、`system.property` 中指定 `spring.profiles.active=dev`，可以激活 application-dev.properties 配置。

### @ActiveProfiles
由 spring-test 提供，
在测试类上标记 `@ActiveProfiles("dev")` 可以激活 application-dev.properties 配置。

### @PropertySource
在 Spring 配置类上标记 `@PropertySource("classpath:application-test.properties")` 可以激活 application-test.properties 配置。

### @TestPropertySource
由 spring-test 提供，只用于测试，在 Spring 配置类上标记 `@TestPropertySource("classpath:application-test.properties")` 可以激活 application-test.properties 配置。

## Profile 注解

- 注解普通 Bean，只有指定环境时，Bean 才能注册到容器中；
- 注解配置类，只有指定环境时，整个配置类中的所有配置才能生效；

例如 `@Profile("dev")`：在 dev 环境下组件才注册到容器中。

