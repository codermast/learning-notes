# Spring学习笔记



## 常见注解

### 定义Bean

- @Component

  > 标注一个普通的Spring Boot类。

- @Controller

  > 控制器，直接和前端客户交互

- @Service

  > 业务层，使用Service注解

- @Respository

  > 数据层，直接和数据进行交互，操作数据

- ComponentScan

	> 定义**扫描的路径**从中找出标识了**需要装配**的类自动装配到spring的bean容器中

### 设置依赖注入

- @Autowired

  > 可用于为类的树形、构造器、方法进行注值

- @Resource

  > 不属于Spring的注解，而是来自于JSR-250位于java.annotation包下，使用该annotation为目标bean指定协作者Bean

- @PostConstruct

  > 初始化Bean之后执行的方法。

- @PreDestroy

  > 销毁Bean之前执行的方法。

小结

- 相同点

  > @Resource的作用相当于@Autowired，均可标注在字段或属性的setter方法上

- 不同点

  > 1. 提供方不同
  >
  >    - @Autowired是Spring的注解，
  >    - @Resource是javax.annotation注解,需要JDK1.6以上
  >
  > 2. 注入方式不同
  >
  >    - @Autowired只按照Type 注入；
  >    - @Resource默认按Name自动注入，也提供按照Type 注入；
  >
  > 3. 属性
  >
  >    - @Autowired注解可用于为类的属性、构造器、方法进行注值。默认情况下，其依赖的对象必须存在（bean可用），如果需要改变这种默认方式，可以设置其required属性为false。
  >    - 还有一个比较重要的点就是，@Autowired注解默认按照类型装配，如果容器中包含多个同一类型的Bean，那么启动容器时会报找不到指定类型bean的异常，解决办法是结合**@Qualifier**注解进行限定，指定注入的bean名称。
  >
  >    - @Resource有两个中重要的属性：name和type。name属性指定byName，如果没有指定name属性，当注解标注在字段上，即默认取字段的名称作为bean名称寻找依赖对象，当注解标注在属性的setter方法上，即默认取属性名作为bean名称寻找依赖对象。
  >    - 需要注意的是，@Resource如果没有指定name属性，并且按照默认的名称仍然找不到依赖对象时， @Resource注解会回退到按类型装配。但一旦指定了name属性，就只能按名称装配了。
  >
  > 4. 灵活性
  >    - @Resource注解的使用性更为灵活，可指定名称，也可以指定类型 ；
  >    - @Autowired注解进行装配容易抛出异常，特别是装配的bean类型有多个的时候，而解决的办法是需要在增加@Qualifier进行限定。

### 配置第三方Bean

@Bean

### 作用范围

@Scope

### 生命周期

@PostConstructor

@PreDestroy

