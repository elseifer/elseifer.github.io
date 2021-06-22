# Spring Aop 的实现

Spring 的两个核心思想: Ioc（控制反转）和 AOP（面向切面编程）.Spring 是一个 AOP 框架, AOP 使得 Spring 更加轻便, 但 IoC 容器本身并不依赖 AOP, 可以在不需要时关闭 AOP<sup>[1]</sup>.

## AOP 概念

AOP(Aspect-Oriented Programming), 即 面向切面编程, 它与 OOP( Object-Oriented Programming, 面向对象编程) 相辅相成, 提供了与 OOP 不同的抽象软件结构的视角.
在 OOP 中, 我们以类(class)作为我们的基本单元, 而 AOP 中的基本单元是 Aspect(切面).

### 术语

#### Aspect(切面)
aspect 由 pointcut 和 advice 组成, 它既包含了横切逻辑的定义, 也包括了连接点的定义. Spring AOP就是负责实施切面的框架, 它将切面所定义的横切逻辑织入到切面所指定的连接点中.
AOP的工作重心在于如何将增强织入在目标对象的连接点上, 这里包含两个工作:

如何通过 pointcut 和 advice 定位到特定的 joinpoint 上
如何在 advice 中编写切面代码.
可以简单地认为, 使用 @Aspect 注解的类就是切面.

#### advice(增强)
由 aspect 添加到特定的 join point(即满足 point cut 规则的 join point) 的一段代码.
许多 AOP框架, 包括 Spring AOP, 会将 advice 模拟为一个拦截器(interceptor), 并且在 join point 上维护多个 advice, 进行层层拦截.
例如 HTTP 鉴权的实现, 我们可以为每个使用 RequestMapping 标注的方法织入 advice, 当 HTTP 请求到来时, 首先进入到 advice 代码中, 在这里我们可以分析这个 HTTP 请求是否有相应的权限, 如果有, 则执行 Controller, 如果没有, 则抛出异常. 这里的 advice 就扮演着鉴权拦截器的角色了.

#### 连接点(join point)
a point during the execution of a program, such as the execution of a method or the handling of an exception. In Spring AOP, a join point always represents a method execution.
程序运行中的一些时间点, 例如一个方法的执行, 或者是一个异常的处理.
在 Spring AOP 中, join point 总是方法的执行点, 即只有方法连接点.

#### 切点(point cut)
匹配 join point 的谓词(a predicate that matches join points).
Advice 是和特定的 point cut 关联的, 并且在 point cut 相匹配的 join point 中执行.
在 Spring 中, 所有的方法都可以认为是 joinpoint, 但是我们并不希望在所有的方法上都添加 Advice, 而 pointcut 的作用就是提供一组规则(使用 AspectJ pointcut expression language 来描述) 来匹配joinpoint, 给满足规则的 joinpoint 添加 Advice.

关于join point 和 point cut 的区别
在 Spring AOP 中, 所有的方法执行都是 join point. 而 point cut 是一个描述信息, 它修饰的是 join point, 通过 point cut, 我们就可以确定哪些 join point 可以被织入 Advice. 因此 join point 和 point cut 本质上就是两个不同纬度上的东西.
advice 是在 join point 上执行的, 而 point cut 规定了哪些 join point 可以执行哪些 advice

#### introduction
为一个类型添加额外的方法或字段. Spring AOP 允许我们为 目标对象 引入新的接口(和对应的实现). 例如我们可以使用 introduction 来为一个 bean 实现 IsModified 接口, 并以此来简化 caching 的实现.

#### 目标对象(Target)
织入 advice 的目标对象. 目标对象也被称为 advised object.
因为 Spring AOP 使用运行时代理的方式来实现 aspect, 因此 adviced object 总是一个代理对象(proxied object)
注意, adviced object 指的不是原来的类, 而是织入 advice 后所产生的代理类.

#### AOP proxy
一个类被 AOP 织入 advice, 就会产生一个结果类, 它是融合了原类和增强逻辑的代理类.
在 Spring AOP 中, 一个 AOP 代理是一个 JDK 动态代理对象或 CGLIB 代理对象.

#### 织入(Weaving)
将 aspect 和其他对象连接起来, 并创建 adviced object 的过程.
根据不同的实现技术, AOP织入有三种方式:

编译器织入, 这要求有特殊的Java编译器.
类装载期织入, 这需要有特殊的类装载器.
动态代理织入, 在运行期为目标类添加增强(Advice)生成子类的方式.
Spring 采用动态代理织入, 而AspectJ采用编译器织入和类装载期织入.

### advice 的类型
Spring AOP 包含以下几类的 advice：
- before advice: 在 join point 前被执行的 advice. 虽然 before advice 是在 join point 前被执行, 但是它并不能够阻止 join point 的执行, 除非发生了异常(即我们在 before advice 代码中, 不能人为地决定是否继续执行 join point 中的代码)
- after return advice: 在一个 join point 正常返回后执行的 advice
- after throwing advice: 当一个 join point 抛出异常后执行的 advice
- after(final) advice: 无论一个 join point 是正常退出还是发生了异常, 都会被执行的 advice.
- around advice, 在 join point 前和 joint point 退出后都执行的 advice. 这个是最常用的 advice.

## 关于 AOP Proxy
Spring AOP 默认使用标准的 JDK 动态代理(dynamic proxy)技术来实现 AOP 代理, 通过它, 我们可以为任意的接口实现代理.
如果需要为一个类实现代理, 那么可以使用 CGLIB 代理. 当一个业务逻辑对象没有实现接口时, 那么 Spring AOP 就默认使用 CGLIB 来作为 AOP 代理了. 即如果我们需要为一个方法织入 advice, 但是这个方法不是一个接口所提供的方法, 则此时 Spring AOP 会使用 CGLIB 来实现动态代理. 鉴于此, Spring AOP 建议基于接口编程, 对接口进行 AOP 而不是类.

| 是否有接口 |  代理类型 |   
| ----| ----|  
| 目标bean基于接口 | JDK 代理 |  
| 目标bean没有接口 | CGLIB 代理 | 

### 如何判断 Spring 用的哪种代理

org.springframework.aop.framework.ProxyFactoryBean 是 FactoryBean 的一个实现，它基于 Spring Bean 来构建 AOP 代理。

>FactoryBean implementation that builds an AOP proxy based on beans in Spring BeanFactory.



# Reference

1. [Spring-Framework Reference: Aop](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aop)

2. [Spring 中 JDK 动态代理与 CGLIB 动态代理](https://blog.csdn.net/wangzhihao1994/article/details/80913210)

3. [BeanFactory 和 FactoryBean 的区别](https://www.cnblogs.com/aspirant/p/9082858.html)

4. [AOP设计基本原理](https://blog.csdn.net/luanlouis/article/details/51095702)