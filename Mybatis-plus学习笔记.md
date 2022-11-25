# Mybatis-plus学习笔记

## 一、快速入门

### 1.简介

[MyBatis-Plus](https://github.com/baomidou/mybatis-plus)（简称 MP）是一个 [MyBatis](https://www.mybatis.org/mybatis-3/)的增强工具，在 MyBatis 的基础上只做增强不做改变，为简化开发、提高效率而生。

>  愿景

我们的愿景是成为 MyBatis 最好的搭档，就像魂斗罗中的 1P、2P，基友搭配，效率翻倍。

![img](https://static.codermast.com/picgo/image/relationship-with-mybatis.png)

### 2.特性

- **无侵入**：只做增强不做改变，引入它不会对现有工程产生影响，如丝般顺滑
- **损耗小**：启动即会自动注入基本 CURD，性能基本无损耗，直接面向对象操作
- **强大的 CRUD 操作**：内置通用 Mapper、通用 Service，仅仅通过少量配置即可实现单表大部分 CRUD 操作，更有强大的条件构造器，满足各类使用需求
- **支持 Lambda 形式调用**：通过 Lambda 表达式，方便的编写各类查询条件，无需再担心字段写错
- **支持主键自动生成**：支持多达 4 种主键策略（内含分布式唯一 ID 生成器 - Sequence），可自由配置，完美解决主键问题
- **支持 ActiveRecord 模式**：支持 ActiveRecord 形式调用，实体类只需继承 Model 类即可进行强大的 CRUD 操作
- **支持自定义全局通用操作**：支持全局通用方法注入（ Write once, use anywhere ）
- **内置代码生成器**：采用代码或者 Maven 插件可快速生成 Mapper 、 Model 、 Service 、 Controller 层代码，支持模板引擎，更有超多自定义配置等您来使用
- **内置分页插件**：基于 MyBatis 物理分页，开发者无需关心具体操作，配置好插件之后，写分页等同于普通 List 查询
- **分页插件支持多种数据库**：支持 MySQL、MariaDB、Oracle、DB2、H2、HSQL、SQLite、Postgre、SQLServer 等多种数据库
- **内置性能分析插件**：可输出 SQL 语句以及其执行时间，建议开发测试时启用该功能，能快速揪出慢查询
- **内置全局拦截插件**：提供全表 delete 、 update 操作智能分析阻断，也可自定义拦截规则，预防误操作

### 3.安装

#### 3.1 环境要求

全新的 `MyBatis-Plus` 3.0 版本基于 JDK8，提供了 `lambda` 形式的调用，所以安装集成 MP3.0 要求如下：

- JDK 8+
- Maven or Gradle

#### 3.2 安装

> 因为主要还是应用在Spring或者SpringBoot框架中，原生的开发很少，这里就只讲解这两种的安装，其他版本的安装，可以去mybatis-plus官方网站查阅官方文档。

##### 3.2.1 Springboot

- Maven

```xml
<!--引入mybatis-plus-boot-starter依赖-->
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>最新版本</version>	<!--这里将最新版本，更换成你想使用的版本号即可。-->
</dependency>
```

- Gradle

```groovy
compile group: 'com.baomidou', name: 'mybatis-plus-boot-starter', version: '最新版本'
```

> 这里将“最新版本”，更换成你想使用的版本号即可。

##### 3.2.2 Spring

- Maven

```xml
<!--引入mybatis-plus依赖-->
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus</artifactId>
    <version>最新版本</version>	<!--这里将最新版本，更换成你想使用的版本号即可。-->
</dependency>
```

- Gradle

```groovy
compile group: 'com.baomidou', name: 'mybatis-plus', version: '最新版本'
```

> 这里将“最新版本”，更换成你想使用的版本号即可。

> ==注意==：引入 `MyBatis-Plus` 之后请不要再次引入 `MyBatis` 以及 `MyBatis-Spring`，以避免因版本差异导致的问题。

### 4.配置

> 在跟随配置之前，您必须确保您正确的安装了对应版本的Mybatis-Plus，不会安装的读者，可以看上面的安装教程，跟随教程选择合适的版本，相信您一定能够成功。

#### 4.1 SpringBoot

- 配置MapperScan注解

```java
@SpringBootApplication
@MapperScan("com.baomidou.mybatisplus.samples.quickstart.mapper")
public class Application {
  
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

> 在SpringBoot项目的启动类中，配置MapperScan注解的。其目的是使项目在启动时能够扫描指定的包，去读取包下的mapper。

#### 4.2 Spring

- 配置 MapperScan

```xml
<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
    <property name="basePackage" value="com.baomidou.mybatisplus.samples.quickstart.mapper"/>
</bean>
```

- 调整 SqlSessionFactory 为 MyBatis-Plus 的 SqlSessionFactory

```xml
<bean id="sqlSessionFactory" class="com.baomidou.mybatisplus.extension.spring.MybatisSqlSessionFactoryBean">
    <property name="dataSource" ref="dataSource"/>
</bean>
```

这里的配置是需要写到Spring的applaction.xml中的，或者你自己定义的bean.xml文件中。只要使得Spring能够读取到即可。

### 5.注解

> 注解类包源码：👉 [mybatis-plus-annotation](https://gitee.com/baomidou/mybatis-plus/tree/3.0/mybatis-plus-annotation/src/main/java/com/baomidou/mybatisplus/annotation)

#### @TableName

- 描述：表名注解，标识实体类对应的表
- 使用位置：实体类

```java
@TableName("sys_user")
public class User {
    private Long id;
    private String name;
    private Integer age;
    private String email;
}
```

| 属性             | 类型     | 必须指定 | 默认值 | 描述                                                         |
| :--------------- | :------- | :------- | :----- | :----------------------------------------------------------- |
| value            | String   | 否       | ""     | 表名                                                         |
| schema           | String   | 否       | ""     | schema                                                       |
| keepGlobalPrefix | boolean  | 否       | false  | 是否保持使用全局的 tablePrefix 的值（当全局 tablePrefix 生效时） |
| resultMap        | String   | 否       | ""     | xml 中 resultMap 的 id（用于满足特定类型的实体类对象绑定）   |
| autoResultMap    | boolean  | 否       | false  | 是否自动构建 resultMap 并使用（如果设置 resultMap 则不会进行 resultMap 的自动构建与注入） |
| excludeProperty  | String[] | 否       | {}     | 需要排除的属性名 @since 3.3.1                                |

**关于 `autoResultMap` 的说明：**

MP 会自动构建一个 `resultMap` 并注入到 MyBatis 里（一般用不上），请注意以下内容：

因为 MP 底层是 MyBatis，所以 MP 只是帮您注入了常用 CRUD 到 MyBatis 里，注入之前是动态的（根据您的 Entity 字段以及注解变化而变化），但是注入之后是静态的（等于 XML 配置中的内容）。

而对于 `typeHandler` 属性，MyBatis 只支持写在 2 个地方:

1. 定义在 resultMap 里，作用于查询结果的封装
2. 定义在 `insert` 和 `update` 语句的 `#{property}` 中的 `property` 后面（例：`#{property,typehandler=xxx.xxx.xxx}`），并且只作用于当前 `设置值`

除了以上两种直接指定 `typeHandler` 的形式，MyBatis 有一个全局扫描自定义 `typeHandler` 包的配置，原理是根据您的 `property` 类型去找其对应的 `typeHandler` 并使用。

#### @TableId

- 描述：主键注解
- 使用位置：实体类主键字段

```java
@TableName("sys_user")
public class User {
    @TableId
    private Long id;
    private String name;
    private Integer age;
    private String email;
}
```

| 属性  | 类型   | 必须指定 | 默认值      | 描述         |
| :---- | :----- | :------- | :---------- | :----------- |
| value | String | 否       | ""          | 主键字段名   |
| type  | Enum   | 否       | IdType.NONE | 指定主键类型 |

**IdType**

| 值            | 描述                                                         |
| :------------ | :----------------------------------------------------------- |
| AUTO          | 数据库 ID 自增                                               |
| NONE          | 无状态，该类型为未设置主键类型（注解里等于跟随全局，全局里约等于 INPUT） |
| INPUT         | insert 前自行 set 主键值                                     |
| ASSIGN_ID     | 分配 ID(主键类型为 Number(Long 和 Integer)或 String)(since 3.3.0),使用接口`IdentifierGenerator`的方法`nextId`(默认实现类为`DefaultIdentifierGenerator`雪花算法) |
| ASSIGN_UUID   | 分配 UUID,主键类型为 String(since 3.3.0),使用接口`IdentifierGenerator`的方法`nextUUID`(默认 default 方法) |
| ID_WORKER     | 分布式全局唯一 ID 长整型类型(please use `ASSIGN_ID`)         |
| UUID          | 32 位 UUID 字符串(please use `ASSIGN_UUID`)                  |
| ID_WORKER_STR | 分布式全局唯一 ID 字符串类型(please use `ASSIGN_ID`)         |

#### @TableField

- 描述：字段注解（非主键）

```java
@TableName("sys_user")
public class User {
    @TableId
    private Long id;
    @TableField("nickname")
    private String name;
    private Integer age;
    private String email;
}
```

| 属性             | 类型                         | 必须指定 | 默认值                   | 描述                                                         |
| :--------------- | :--------------------------- | :------- | :----------------------- | :----------------------------------------------------------- |
| value            | String                       | 否       | ""                       | 数据库字段名                                                 |
| exist            | boolean                      | 否       | true                     | 是否为数据库表字段                                           |
| condition        | String                       | 否       | ""                       | 字段 `where` 实体查询比较条件，有值设置则按设置的值为准，没有则为默认全局的 `%s=#{%s}`，[参考(opens new window)](https://github.com/baomidou/mybatis-plus/blob/3.0/mybatis-plus-annotation/src/main/java/com/baomidou/mybatisplus/annotation/SqlCondition.java) |
| update           | String                       | 否       | ""                       | 字段 `update set` 部分注入，例如：当在version字段上注解`update="%s+1"` 表示更新时会 `set version=version+1` （该属性优先级高于 `el` 属性） |
| insertStrategy   | Enum                         | 否       | FieldStrategy.DEFAULT    | 举例：NOT_NULL `insert into table_a(<if test="columnProperty != null">column</if>) values (<if test="columnProperty != null">#{columnProperty}</if>)` |
| updateStrategy   | Enum                         | 否       | FieldStrategy.DEFAULT    | 举例：IGNORED `update table_a set column=#{columnProperty}`  |
| whereStrategy    | Enum                         | 否       | FieldStrategy.DEFAULT    | 举例：NOT_EMPTY `where <if test="columnProperty != null and columnProperty!=''">column=#{columnProperty}</if>` |
| fill             | Enum                         | 否       | FieldFill.DEFAULT        | 字段自动填充策略                                             |
| select           | boolean                      | 否       | true                     | 是否进行 select 查询                                         |
| keepGlobalFormat | boolean                      | 否       | false                    | 是否保持使用全局的 format 进行处理                           |
| jdbcType         | JdbcType                     | 否       | JdbcType.UNDEFINED       | JDBC 类型 (该默认值不代表会按照该值生效)                     |
| typeHandler      | Class<? extends TypeHandler> | 否       | UnknownTypeHandler.class | 类型处理器 (该默认值不代表会按照该值生效)                    |
| numericScale     | String                       | 否       | ""                       | 指定小数点后保留的位数                                       |

>  关于`jdbcType`和`typeHandler`以及`numericScale`的说明:
>
> `numericScale`只生效于 update 的 sql. `jdbcType`和`typeHandler`如果不配合`@TableName#autoResultMap = true`一起使用,也只生效于 update 的 sql. 对于`typeHandler`如果你的字段类型和 set 进去的类型为`equals`关系,则只需要让你的`typeHandler`让 Mybatis 加载到即可,不需要使用注解

**FieldStrategy**

| 值        | 描述                                                        |
| :-------- | :---------------------------------------------------------- |
| IGNORED   | 忽略判断                                                    |
| NOT_NULL  | 非 NULL 判断                                                |
| NOT_EMPTY | 非空判断(只对字符串类型字段,其他类型字段依然为非 NULL 判断) |
| DEFAULT   | 追随全局配置                                                |
| NEVER     | 不加入SQL                                                   |

**FieldFill**

| 值            | 描述                 |
| :------------ | :------------------- |
| DEFAULT       | 默认不处理           |
| INSERT        | 插入时填充字段       |
| UPDATE        | 更新时填充字段       |
| INSERT_UPDATE | 插入和更新时填充字段 |

#### @Version

- 描述：乐观锁注解、标记 `@Version` 在字段上

#### @EnumValue

- 描述：普通枚举类注解(注解在枚举字段上)

#### @TableLogic

- 描述：表字段逻辑处理注解（逻辑删除）

| 属性   | 类型   | 必须指定 | 默认值 | 描述         |
| :----- | :----- | :------- | :----- | :----------- |
| value  | String | 否       | ""     | 逻辑未删除值 |
| delval | String | 否       | ""     | 逻辑删除值   |

#### @SqlParser

> see [@InterceptorIgnore](https://baomidou.com/pages/223848/#InterceptorIgnore)

#### @KeySequence

- 描述：序列主键策略 `oracle`
- 属性：value、dbType

| 属性   | 类型   | 必须指定 | 默认值       | 描述                                                         |
| :----- | :----- | :------- | :----------- | :----------------------------------------------------------- |
| value  | String | 否       | ""           | 序列名                                                       |
| dbType | Enum   | 否       | DbType.OTHER | 数据库类型，未配置默认使用注入 IKeyGenerator 实现，多个实现必须指定 |

#### @InterceptorIgnore

- `value` 值为 `1` | `yes` | `on` 视为忽略，例如 `@InterceptorIgnore(tenantLine = "1")`
- `value` 值为 `0` | `false` | `off` | `空值不变` 视为正常执行。

> see [插件主体](https://baomidou.com/pages/2976a3/)

#### @OrderBy

- 描述：内置 SQL 默认指定排序，优先级低于 wrapper 条件查询

| 属性   | 类型    | 必须指定 | 默认值          | 描述           |
| :----- | :------ | :------- | :-------------- | :------------- |
| isDesc | boolean | 否       | true            | 是否倒序查询   |
| sort   | short   | 否       | Short.MAX_VALUE | 数字越小越靠前 |

### 6.快速入门

1. 初始化Springboot工程

![创建spring工程](https://static.codermast.com/picgo/image/%E5%88%9B%E5%BB%BAspring%E9%A1%B9%E7%9B%AE.png)

> 这里我选择的SpringBoot版本是2.3.7.RELEASE

![springboot版本选择](https://static.codermast.com/picgo/image/springboot%E7%89%88%E6%9C%AC%E9%80%89%E6%8B%A9.png)

2. 添加依赖



```xml
<dependencies>
  	<!--spring-boot-starter-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
    </dependency>
    <!--spring-boot-starter-test-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    <!--mybatis-plus-boot-starter-->
    <dependency>
        <groupId>com.baomidou</groupId>
        <artifactId>mybatis-plus-boot-starter</artifactId>
        <version>最新版本</version>	
    </dependency>
    <!--h2-->
    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
        <scope>runtime</scope>
    </dependency>
</dependencies>
```

3. 配置YML



4. 编写实体类



5. 编写Mapper接口



6. 准备数据库

   - 创建数据库表

     ```sql
     DROP TABLE IF EXISTS user;
     
     CREATE TABLE user
     (
         id BIGINT(20) NOT NULL COMMENT '主键ID',
         name VARCHAR(30) NULL DEFAULT NULL COMMENT '姓名',
         age INT(11) NULL DEFAULT NULL COMMENT '年龄',
         email VARCHAR(50) NULL DEFAULT NULL COMMENT '邮箱',
         PRIMARY KEY (id)
     );
     ```

   - 插入数据

     ```sql
     DELETE FROM user;
     
     INSERT INTO user (id, name, age, email) VALUES
     (1, 'Jone', 18, 'test1@baomidou.com'),
     (2, 'Jack', 20, 'test2@baomidou.com'),
     (3, 'Tom', 28, 'test3@baomidou.com'),
     (4, 'Sandy', 21, 'test4@baomidou.com'),
     (5, 'Billie', 24, 'test5@baomidou.com');
     ```
   

>  执行完上面两步的sql以后，我们能得到一张 `User` 表，其表结构和数据项如下：

| id   | name   | age  | email              |
| ---- | ------ | ---- | ------------------ |
| 1    | Jone   | 18   | test1@baomidou.com |
| 2    | Jack   | 20   | test2@baomidou.com |
| 3    | Tom    | 28   | test3@baomidou.com |
| 4    | Sandy  | 21   | test4@baomidou.com |
| 5    | Billie | 24   | test5@baomidou.com |