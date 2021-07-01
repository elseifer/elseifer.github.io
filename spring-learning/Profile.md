# Spring Profile 机制

## 默认加载顺序
SpringBoot启动会扫描以下位置的application.properties/yml文件作为spring boot的默认配置文件：

file:./config/
file:./
classpath:/config/
classpath:/
以上是按照优先级从高到低的顺序，所有位置的文件都会被加载，高优先级配置内容会覆盖低优先级配置的内容，并形成互补配置。我们也可以通过spring.config.location来改变默认配置。

file: 指当前项目根目录
classpath: 指当前项目的resources目录

## 外化配置 

### -Dspring.config.location 

指定外部配置文件时，需要此份配置文件需全量满足当前工程运行时所需，因为它不会去与 resources 目录下的配置文件去做 merge 操作。

### -Dspring.config.additional-location 
外化配置文件时，可以激活指定路径的配置文件，指定差异增量配置

spring.config.location > spring.profiles.active > spring.config.additional-location > 默认的 application.proerties。

其中通过 spring.profiles.active 和 spring.config.additional-location 指定的配置文件会与默认 application.proerties 合并以作为最终的配置，spring.config.location 则不会。

## 激活 profile 几种方法

- spring.profiles.active=dev 
在默认配置文件、-D、system.property 中指定，可以激活 application-dev.properties 配置。

- @ActiveProfiles("dev") 
在测试类上标记，可以激活 application-dev.properties 配置

- @PropertySource("classpath:application-test.properties")
在 Spring 配置类上标记，可以激活 application-test.properties 配置

- @TestPropertySource("classpath:application-test.properties")
只用于测试，在 Spring 配置类上标记，可以激活 application-test.properties 配置

## Profile 注解

- 注解普通 Bean，只有指定环境时，Bean 才能注册到容器中；
- 注解配置类，只有指定环境时，整个配置类中的所有配置才能生效；

例如 `@Profile("dev")`：在 dev 环境下组件才注册到容器中。

