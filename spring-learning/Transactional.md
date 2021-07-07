# @Transactional 

使用注意：
1. 必须用在 public 方法上；
2. 被同一个类的方法调用时，不产生事物；
3. 注意在类、接口上使用的差异；
4. 配合 select for update 时注意 isolation = Isolation.REPEATABLE_READ 否则默认为 DEFAULT 会出现脏读、不可重复读和幻读

> Are class-based (CGLIB) proxies to be created? By default, standard Java interface-based proxies are created.  
Note: Class-based proxies require the @Transactional annotation to be defined on the concrete class. Annotations in interfaces will not work in that case (they will rather only work with interface-based proxies)!

在类上使用该注解时，需要配合如下：
`<tx:annotation-driven transaction-manager="transactionManager" proxy-target-class="true"/>`
默认情况下 proxy-target-class 为 false。
