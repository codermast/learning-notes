

# ![知识星球推广](images/%E7%9F%A5%E8%AF%86%E6%98%9F%E7%90%83%E6%8E%A8%E5%B9%BF-9345260.jpeg)![公众号-爱编程的mast](images/%E5%85%AC%E4%BC%97%E5%8F%B7-%E7%88%B1%E7%BC%96%E7%A8%8B%E7%9A%84mast-9345257.png)Redis学习笔记

## 1. 认识Redis

### 1.1 什么是NoSQL？

SQL是关系型数据库，NoSQL是非关系型数据库

> 下面是几种非关系型数据库及其优缺点和应用场景。

| **分类**              | **Examples举例**                                      | 典型应用场景                                                 | 数据模型                                        | 优点                                                         | 缺点                                                         |
| --------------------- | ----------------------------------------------------- | ------------------------------------------------------------ | ----------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **键值（key-value）** | Tokyo Cabinet/Tyrant， Redis， Voldemort， Oracle BDB | 内容缓存，主要用于处理大量数据的高访问负载，也用于一些日志系统等等。 | Key 指向 Value 的键值对，通常用hash table来实现 | 查找速度快                                                   | 数据无结构化，通常只被当作字符串或者二进制数据               |
| **列存储数据库**      | Cassandra， HBase， Riak                              | 分布式的文件系统                                             | 以列簇式存储，将同一列数据存在一起              | 查找速度快，可扩展性强，更容易进行分布式扩展                 | 功能相对局限                                                 |
| **文档型数据库**      | CouchDB， MongoDb                                     | Web应用（与Key-Value类似，Value是结构化的，不同的是数据库能够了解Value的内容） | Key-Value对应的键值对，Value为结构化数据        | 数据结构要求不严格，表结构可变，不需要像关系型数据库一样需要预先定义表结构 | 查询性能不高，而且缺乏统一的查询语法。                       |
| **图形(Graph)数据库** | Neo4J， InfoGrid， Infinite Graph                     | 社交网络，推荐系统等。专注于构建关系图谱                     | 图结构                                          | 利用图结构相关算法。比如最短路径寻址，N度关系查找等          | 很多时候需要对整个图做计算才能得出需要的信息，而且这种结构不太好做分布式的集群方案。 |

### 1.2 什么是Redis？

Redis诞生于2009年全称是Remote Dictionary Server，远程词典服务器，是一个基于内存的键值型NoSQL数据库

**特征：**

- 键值（key-value）型，value支持多种不同数据结构，功能丰富
- 单线程，每个命令具备原子性
- 低延迟，速度快（基于内存、IO多路复用、良好的编码）
- 支持数据持久化
- 支持主从集群、分片集群
- 支持多语言客户端

## 2. 安装Redis

### 2.1 安装Redis依赖

> 在安装Redis之前，我们首先得安装Redis所需的依赖，因为Redis是使用C语言进行开发的，因此应该先安装gcc依赖。

```shell
yum install -y gcc tcl
```

### 2.2 上传安装包并解压缩

> 这里可以直接使用ftp从本地环境上传下载的Redis安装包，也可以直接去官网下载。这里我直接是使用的ftp上传下载好的redis压缩包。

比如这里我将压缩包上传到了`/usr/local/bin`目录下。你也可以放到其他目录。

![redis压缩包上传](images/redis%E5%8E%8B%E7%BC%A9%E5%8C%85%E4%B8%8A%E4%BC%A0.png)

解压缩安装包

```shell
tar -xzf redis-6.2.7.tar.gz
```

![redis安装包解压缩](images/redis%E5%AE%89%E8%A3%85%E5%8C%85%E8%A7%A3%E5%8E%8B%E7%BC%A9.png)

> 解压缩完成以后，就会有一个redis目录，==注意==：这里的redis目录的内容都还是没有进行编译的源代码，还无法使用，在使用之前，我们必须先进行编译，编译完以后才能够使用。

2.3 redis编译

先执行测试编译，看是否能编译通过。这里必须进入到redis解压缩的目录中执行。不能在目录外执行。

```shell
make test
```

这里又可能会有报错，报错信息如下：

```shell
[root@localhost redis-6.2.7]# make test
cd src && make test
make[1]: 进入目录“/usr/local/bin/redis-6.2.7/src”
    CC adlist.o
In file included from adlist.c:34:
zmalloc.h:50:10: 致命错误：jemalloc/jemalloc.h：没有那个文件或目录
   50 | #include <jemalloc/jemalloc.h>
      |          ^~~~~~~~~~~~~~~~~~~~~
编译中断。
make[1]: *** [Makefile:376：adlist.o] 错误 1
make[1]: 离开目录“/usr/local/bin/redis-6.2.7/src”
make: *** [Makefile:6：test] 错误 2
```

这种情况下，我们需要执行下面这条指令：

```shell
make MALLOC=libc
```

![redis编译报错解决](images/redis%E7%BC%96%E8%AF%91%E6%8A%A5%E9%94%99%E8%A7%A3%E5%86%B3.png)

执行完以后，在执行测试编译指令即可。能够编译通过后，在进行编译操作。

```shell
make && make install
```

编译完成后，默认的安装目录是在`/usr/local/bin`，返回上级目录即可查看。

![redis编译成功](images/redis%E7%BC%96%E8%AF%91%E6%88%90%E5%8A%9F.png)

该目录以及默认配置到环境变量，因此可以在任意目录下运行这些命令。其中：

- redis-cli：是redis提供的命令行客户端
- redis-server：是redis的服务端启动脚本
- redis-sentinel：是redis的哨兵启动脚本

### 2.3 使用yum安装redis

> 除了使用安装包编译安装，还可以使用yum指令直接安装redis，这种方式安装成功以后，直接就可以运行。

```shell
yum install -y redis
```

![安装redis](images/%E5%AE%89%E8%A3%85redis.png)

## 3. 启动Redis

> Redis启动的方式有多种，这里就讲解一下最常见的三种方式：默认启动、配置文件启动、开机自启动

### 3.1 默认启动

```shell
redis-server
```

输入该指令即可在命令终端前台启动redis，

![redis前台启动](images/redis%E5%89%8D%E5%8F%B0%E5%90%AF%E5%8A%A8.png)

### 3.2 配置文件启动

> 这种方式启动Redis可以设置很多我们自己配置的信息，通过redis.conf文件进行启动。
>
> 使用安装包安装的在安装目录下。
>
> 使用yum指令安装的redis，其配置文件redis.conf保存在==/etc/redis== 目录下，我们通过vim进行编辑即可。

顾名思义，通过修改默认的配置文件，从而达到我们想要的效果或者功能。我们使用修改好的redis.conf配置文件进行启动的命令为

```shell
redis-server redis.conf
```

将配置文件的信息附在redis-server服务的后面，这里需要写的是redis.conf地址，我这里就在同一目录下所以可以直接写文件名。

因为我们配置了进程守护，所以启动以后我们并不能看到redis的启动界面。我们可以使用`redis-cli`客户端进行连接，也可以使用`ps -ef ｜ grep redis`指令查看进程是否启动。

![redis守护进程-密码连接](images/redis%E5%AE%88%E6%8A%A4%E8%BF%9B%E7%A8%8B-%E5%AF%86%E7%A0%81%E8%BF%9E%E6%8E%A5.png)

这里简单的介绍一下几个最常见的配置信息：

- Bind:连接地址

```shell
# 监听的地址，默认是127.0.0.1，会导致只能在本地访问。修改为0.0.0.0则可以在任意IP访问，生产环境不要设置为0.0.0.0
bind 0.0.0.0
```

- Daemonize：守护进程

```shell
# 守护进程，修改为yes后即可后台运行
daemonize yes
```

- Requirepass：密码

```shell
# 密码，设置后访问Redis必须输入密码
requirepass codermast # 这里的 codermast 就是登陆的密码
```

- Timeout：超时时间

```shell
# 当客户端闲置多长时间后关闭连接，如果指定为0，表示关闭该功能
timeout 300
```

- Port：监听的端口

```shell
# 指定Redis监听端口，默认端口为6379，作者在自己的一篇博文中解释了为什么选用6379作为默认端口，因为6379在手机按键上MERZ对应的号码，而MERZ取自意大利歌女Alessia Merz的名字
port 6379
```

- Loglevel：日志级别

```shell
# 指定日志记录级别，Redis总共支持四个级别：debug、verbose、notice、warning，默认为verbose
loglevel verbose
```

- Database：数据库数量

```shell
# 设置数据库的数量，默认数据库为0，可以使用SELECT <dbid>命令在连接上指定数据库id
databases 16
```

> 对与更多的redis.conf配置文件的内容，可以查看官方文档。这里参考的是菜鸟教程的文档：https://www.runoob.com/redis/redis-conf.html

### 3.3 开机自启动

1. 新建一个系统服务文件

```shell
vi /etc/systemd/system/redis.service
```

文件内容如下

```shell
[Unit]
Description=redis-server
After=network.target

[Service]
Type=forking
ExecStart=/usr/local/bin/redis-server /usr/local/src/redis-6.2.6/redis.conf
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

2. 重载系统服务

```shell
systemctl daemon-reload
```

> 现在就可以使用系统指令来操作redis了

```shell
# 启动
systemctl start redis
# 停止
systemctl stop redis
# 重启
systemctl restart redis
# 查看状态
systemctl status redis
```

执行下面这行指令，即可使得redis开机自启动。

```shell
systemctl enable redis
```

## 4.客户端

redis的客户端也有很多种，按照形式简单的分为一下三类：

- 命令行客户端
- 图形化客户端
- 编程语言客户端

下面对这三种客户端进行学习。

### 4.1 命令行客户端

完成redis的安装以后，自带了一个命令行客户端：redis-cli，使用方式为：

```shell
redis-cli [options] [commonds]
```

其中常见的options有：

- `-h 127.0.0.1`：指定要连接的redis节点的IP地址，默认是127.0.0.1
- `-p 6379`：指定要连接的redis节点的端口，默认是6379
- `-a codermast`：指定redis的访问密码

其中的commonds就是Redis的操作命令，例如：

- `ping`：与redis服务端做心跳测试，服务端正常会返回`pong`
- AUTH：在连接时未验证密码，在连接后进行密码的校验

不指定commond时，会进入`redis-cli`的交互控制台：

![redis-cli连接](images/redis-cli%E8%BF%9E%E6%8E%A5.png)

### 4.2 图形化客户端

图形化客户端，顾名思义就是通过可视化的软件，来对redis数据库进行操作。

图形化界面的客户端种类很多，可以选择自己喜欢的一款就可以，这里我选择的是AnotherRedisDesktopManager这款，支持Mac、Linux、Windows系统，更重要的是还免费。

Github地址：https://github.com/qishibo/AnotherRedisDesktopManager/releases/tag/v1.5.9

> 对于不会访问github的用户，可以直接在国内的gitee中下载安装，一般来说两者都没有什么差别，唯一的区别就在于github上的更新比较及时。

Gitee地址：https://gitee.com/qishibo/AnotherRedisDesktopManager/releases/tag/v1.5.8

![redis图形化客户端-another redis desktop manager](images/redis%E5%9B%BE%E5%BD%A2%E5%8C%96%E5%AE%A2%E6%88%B7%E7%AB%AF-another%20redis%20desktop%20manager.png)

### 4.3 编程语言客户端

> 编程语言客户端是有很多种，并不局限于某种语言，对于任何语言只要编写了redis的接口，理论上都是可以使用redis的。这里我就用我最熟悉也是目前应用最广泛的Java语言来讲解。

Java的客户端也不止一种，凡是能够实现功能的都可以。你自己也可以编写一套属于你的redis客户端。这里罗列几个使用较多的：

- Jedis
- SpringDataRedis

下面就这两种进行详细的解释

#### 4.3.1 Jedis

> Jedis是开源的，这里附上地址：https://github.com/redis/jedis

##### 1.Jedis快速入门

- Maven工程中引入Jedis依赖

```xml
<!--引入Jedis依赖-->
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>4.2.0</version>
</dependency>
```

- 建立与redis的连接

```java
private Jedis jedis;

@BeforeEach //被该注解修饰的方法每次执行其他方法前自动执行
void setUp(){
    // 1. 获取连接
    jedis = new Jedis("10.211.55.5",6379);	
  	// 这里的ip地址改成你自己redis服务器的地址，如果是本地就写127.0.0.1，因为我是在虚拟机上，就写的虚拟机地址。
    // 2. 设置密码
    jedis.auth("codermast");
    // 3. 选择库（默认是下标为0的库）默认是有16个库，编号为0-15，任选一个即可。
    jedis.select(0);
}
```

- 操作redis

```java
@Test
public void testString(){
    // 1.往redis中存放一条String类型的数据并获取返回结果
    String result = jedis.set("url", "https://www.codermast.com/");
    System.out.println("result = " + result);

    // 2.从redis中获取一条数据
    String url = jedis.get("url");
    System.out.println("url = " + url);
}
```

- 释放资源

```java
@AfterEach
void tearDown(){
    // 如果不为空，即为没有释放，进行释放即可。
    if( jidis != null){
      jedis.close();
    }
}
```

##### 2.Jedis连接池

> Jedis本身是线程不安全的，因为redis是单线程的，多个jedis对象操作redis，必然会造成数据混乱。并且频繁的创建和销毁连接会有性能损耗，因此推荐大家使用Jedis连接池代替Jedis的直连方式

```java
public class JedisConnectionFactory {
    private static final JedisPool jedisPool;

    static {
        //配置连接池
        JedisPoolConfig jedisPoolConfig = new JedisPoolConfig();
        jedisPoolConfig.setMaxTotal(8);	// 最大数量
        jedisPoolConfig.setMaxIdle(8);	// 最大空闲数量
        jedisPoolConfig.setMinIdle(0);	// 最小空闲数量
        jedisPoolConfig.setMaxWaitMillis(200);	// 获取连接的最大等待毫秒数
        //创建连接池对象
        jedisPool = new JedisPool(jedisPoolConfig,"10.211.55.5",6379,1000,"codermast");
    }

    public static Jedis getJedis(){
       return jedisPool.getResource();
    }
}
```

在需要使用jedis连接时，调用JedisConnectionFactory.getJedis()方法即可从连接池获得连接。效率比每次的直接连接时要高的。

#### 4.3.2 SpringDataRedis

##### 1.SpringDataRedis介绍

> SpringData是Spring中数据操作的模块，包含对各种数据库的集成，其中对Redis的集成模块就叫做`SpringDataRedis`
>
> 官网地址：https://spring.io/projects/spring-data-redis

- 提供了对不同Redis客户端的整合（`Lettuce`和`Jedis`）
- 提供了`RedisTemplate`统一API来操作Redis
- 支持Redis的发布订阅模型
- 支持Redis哨兵和Redis集群
- 支持基于Lettuce的响应式编程
- 支持基于JDK、JSON、字符串、Spring对象的数据序列化及反序列化
- 支持基于Redis的JDKCollection实现

> SpringDataRedis中提供了RedisTemplate工具类，其中封装了各种对Redis的操作。并且将不同数据类型的操作API封装到了不同的类型中：

|             API             |   返回值类型    |       说明        |
| :-------------------------: | :-------------: | :---------------: |
| redisTemplate.opsForValue() | ValueOperations |  操作String类型   |
| redisTemplate.opsForHash()  | HashOperations  |   操作Hash类型    |
| redisTemplate.opsForList()  | ListOperations  |   操作List类型    |
|  redisTemplate.opsForSet()  |  SetOperations  |    操作Set类型    |
| redisTemplate.opsForZSet()  | ZSetOperations  | 操作SortedSet类型 |
|        redisTemplate        |                 |    通用的命令     |

##### 2.SpringDataRedis快速入门

> `SpringBoot`已经提供了对`SpringDataRedis`的支持，使用非常简单

1. 新建SpringBoot项目

2. 引入所需要的依赖

```xml
<!--springdataredis依赖-->
<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<!--lombok依赖-->
<dependency>
		<groupId>org.projectlombok</groupId>
		<artifactId>lombok</artifactId>
		<optional>true</optional>
</dependency>
<!--连接池依赖-->
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-pool2</artifactId>
</dependency>
```

3. 编写配置文件

>  application.yml（也可以使用application.properties配置），这里对于不熟悉yml配置文件格式的读者来说，需要注意，每个关键字的冒号后需要加一个空格，这是yml文件格式，不加空格则不能够解析。对于初学者来说，这里出的问题很难用肉眼直接看到，也很难排查错误。一个小细节，多注意即可。

```yml
spring:
  redis:
    host: 10.211.55.5 #指定redis所在的host
    port: 6379  #指定redis的端口
    password: codermast  #设置redis密码
    lettuce:
      pool:
        max-active: 8 	#最大连接数
        max-idle: 8 		#最大空闲数
        min-idle: 0 		#最小空闲数
        max-wait: 200ms #连接等待时间
```

4. 编写测试类

```java
@SpringBootTest
class RedisDemoApplicationTests {

    @Autowired	// 这里springboot会自动注入
    private RedisTemplate redisTemplate;

    @Test
    void testString() {
        // 1.通过RedisTemplate获取操作String类型的ValueOperations对象
        ValueOperations ops = redisTemplate.opsForValue();
        // 2.插入一条数据
        ops.set("name","codermast");

        // 3.获取数据
        String name = (String) ops.get("name");
        System.out.println("name = " + name);
    }
}
```

> 执行完成以后，就能看到在命令行内输出了codermast，则证明成功！

#### 4.3.3 RedisSerializer序列化

RedisTemplate可以接收任意Object作为值写入Redis，只不过写入前会把Object序列化为字节形式，`默认是采用JDK序列化`，得到的结果是这样的

<img src="images/%E6%9C%AA%E5%BA%8F%E5%88%97%E5%8C%96%E5%AD%98%E5%85%A5redis.png" alt="未序列化存入redis" style="zoom:67%;" />

虽然在Java客户端内使用jdk进行序列化后是没有问题的，但是如果是多个环境需要共享同一个redis时，并不会有jdk系列化的规则，那么就会出现无法识别数据的问题，那么想要避免这个问题，我们就必须将真正的数据存入redis。则我们需要对其进行手动序列化配置。

**缺点：**

- 可读性差
- 内存占用较大

> **那么如何解决以上的问题呢？我们可以通过自定义RedisTemplate序列化的方式来解决。**

- **编写一个配置类`RedisConfig`**

```java
@Configuration
public class RedisConfig {

    @Bean
    public RedisTemplate<String,Object> redisTemplate(RedisConnectionFactory factory){
        // 1.创建RedisTemplate对象
        RedisTemplate<String ,Object> redisTemplate = new RedisTemplate<>();
        // 2.设置连接工厂
        redisTemplate.setConnectionFactory(factory);

        // 3.创建序列化对象
        StringRedisSerializer stringRedisSerializer = new StringRedisSerializer();
        GenericJackson2JsonRedisSerializer genericJackson2JsonRedisSerializer = new GenericJackson2JsonRedisSerializer();

        // 4.设置key和hashKey采用String的序列化方式
        redisTemplate.setKeySerializer(stringRedisSerializer);
        redisTemplate.setHashKeySerializer(stringRedisSerializer);

        // 5.设置value和hashValue采用json的序列化方式
        redisTemplate.setValueSerializer(genericJackson2JsonRedisSerializer);
        redisTemplate.setHashValueSerializer(genericJackson2JsonRedisSerializer);

        return redisTemplate;
    }
}
```

> ==注意==：这里我们配置完以后，直接运行是有可能会报错的，是因为我们的配置类中需要一个Jackson序列化的类，但是这个类我们没有，报错信息也会提示我们这个类未被定义，所以我们需要先引入jackson-databind这个依赖，再次运行就可以成功了。

- jackson-databind依赖

```xml
<!--jackson-databind依赖-->
<dependency>
			<groupId>com.fasterxml.jackson.core</groupId>
			<artifactId>jackson-databind</artifactId>
</dependency>
```

再次执行测试方法，则可以看到redis中的key和value已经恢复正常。

![序列化后存入redis](images/%E5%BA%8F%E5%88%97%E5%8C%96%E5%90%8E%E5%AD%98%E5%85%A5redis.png)

我们在上面已经配置好了value为对象的序列化，将对象转换成json字符串存储，在测试之前我们必须先准备一个对象类。

- 定义User类

```java
public class User {
    private int age;        // 年龄
    private String username;// 用户名
    private String password;// 密码
    private boolean sex;    // 性别

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public boolean isSex() {
        return sex;
    }

    public void setSex(boolean sex) {
        this.sex = sex;
    }

    @Override
    public String toString() {
        return "User{" +
                "age=" + age +
                ", username='" + username + '\'' +
                ", password='" + password + '\'' +
                ", sex=" + sex +
                '}';
    }
}
```

- 定义测试方法

```java
@Test
void testUser(){
    // 1.通过RedisTemplate获取操作String类型的ValueOperations对象
    ValueOperations ops = redisTemplate.opsForValue();

    // 2.准备一个user对象
    User user = new User();
    user.setAge(18);
    user.setUsername("codermast");
    user.setPassword("123456789");
    user.setSex(true);
    // 3.插入一条数据
    ops.set("user", user);

    // 3.获取数据
    Object user1 = ops.get("user");
    System.out.println("name = " + user1);
}
```

> 现在可以直接运行测试方法，得到的结果如下：

![redis序列化对象](images/redis%E5%BA%8F%E5%88%97%E5%8C%96%E5%AF%B9%E8%B1%A1.png)

从截图中我们可以看到，user对象已经被成功的序列化了，但是除了我们自己添加的一些属性以外，还有一个不是我们添加的属性值，"@class"这个键值对添加了我们User类的全限定名，也就是类的位置。他自动帮我们记录了这个值，可以用于我们反序列化这个json字符串。但是在存储的数据多了以后，这个信息会占用大量的存储空间，导致我们的存储效率大幅下降。有时候还会出现这个字符串的长度比我们真正想存储的数据的内容还多的情况。接下来我们需要对这些情况进行处理。

#### 4.3.4 StringRedisTemplate

>  为了节省内存空间，我们并不会使用JSON序列化器来处理value，而是统一使用String序列化器，要求只能存储String类型的key和value。当需要存储Java对象时，手动完成对象的序列化和反序列化。

> Spring默认提供了一个StringRedisTemplate类，它的key和value的序列化方式默认就是String方式。省去了我们自定义RedisTemplate的过程

- 我们可以直接编写一个测试类使用StringRedisTemplate来执行以下方法

```java
@SpringBootTest
class RedisStringTemplateTest {
		@Autowired
    private StringRedisTemplate stringRedisTemplate;
  
    @Test
    void testSaveUser() throws JsonProcessingException {
        // 1.创建一个Json序列化对象
        ObjectMapper objectMapper = new ObjectMapper();
        // 2.将要存入的对象通过Json序列化对象转换为字符串
        User user = new User();
        user.setAge(18);
        user.setUsername("codermast");
        user.setPassword("123456789");
        user.setSex(true);
        String userJson1 = objectMapper.writeValueAsString(user);
        // 3.通过StringRedisTemplate将数据存入redis
        stringRedisTemplate.opsForValue().set("user:100",userJson1);
        // 4.通过key取出value
        String userJson2 = stringRedisTemplate.opsForValue().get("user:100");
        // 5.由于取出的值是String类型的Json字符串，因此我们需要通过Json序列化对象来转换为java对象
        User user2 = objectMapper.readValue(userJson2, User.class);
        // 6.打印结果
        System.out.println("user = " + user2);
    }
}
```

![自己序列化的json](images/%E8%87%AA%E5%B7%B1%E5%BA%8F%E5%88%97%E5%8C%96%E7%9A%84json.png)

#### 4.3.5 总结

>  RedisTemplate的两种序列化实践方案，两种方案各有各的优缺点，可以根据实际情况选择使用。

方案一：

1. 自定义RedisTemplate
2. 修改RedisTemplate的序列化器为GenericJackson2JsonRedisSerializer

方案二：

1. 使用StringRedisTemplate
2. 写入Redis时，手动把对象序列化为JSON
3. 读取Redis时，手动把读取到的JSON反序列化为对象

## 5.Redis常见指令

> **通用指令是部分数据类型的，都可以使用的指令，常见的有如下表格所示**

|     指令      |                        描述                        |
| :-----------: | :------------------------------------------------: |
|     keys      | 查看符合模板的所有key，不建议在生产环境设备上使用  |
|      del      |                 删除一个指定的key                  |
|    esists     |                  判断key是否存在                   |
|    expire     | 给一个key设置有效期，有效期到期时该key会被自动删除 |
|      ttl      |              查看一个KEY的剩余有效期               |
|     quit      |                        退出                        |
|   shutdown    |                     关闭服务器                     |
| select [0-15] |                  选择指定的数据库                  |

**可以通过`help [command] `可以查看一个命令的具体用法！**例如：

```shell
help set
```

## 6.Redis的数据结构

redis的数据结构分为两种类型，基本类型和特殊类型。其中基本类型有5种，特殊类型有三种。

### 6.1基本类型

#### 6.1.1String类型

|    命令     |                             描述                             |
| :---------: | :----------------------------------------------------------: |
|     SET     |         添加或者修改已经存在的一个String类型的键值对         |
|     GET     |                 根据key获取String类型的value                 |
|    MSET     |                批量添加多个String类型的键值对                |
|    MGET     |             根据多个key获取多个String类型的value             |
|    INCR     |                     让一个整型的key自增1                     |
|   INCRBY    | 让一个整型的key自增并指定步长，例如：incrby num 2 让num值自增2 |
| INCRBYFLOAT |              让一个浮点类型的数字自增并指定步长              |
|    SETNX    | 添加一个String类型的键值对，前提是这个key不存在，否则不执行  |
|    SETEX    |          添加一个String类型的键值对，并且指定有效期          |

> redis的key本质上是没有层次结构的都直接存储在数据库的根下，但是实际我们存储时是根据业务的层级结构进行划分的。
>
> 虽然redis的key没有物理上的层次结构，但是允许有多个单词形成层级结构，多个单词之间使用“==:==”隔开，格式如下：

```java
项目名:业务名:类型:key
// 例如：codermast:articles:user:123
```

这个格式并非固定，也可以根据自己的需求来删除或添加词条，可以实现n级分层。

例如我们的项目名称叫 `codermast`，有`user`和`product`两种不同类型的数据，我们可以这样定义key：

- **user**相关的key：`codermast:user:1`
- **product**相关的：`codermast:product:1`

如果Value是一个Java对象，例如一个User对象，则可以将对象序列化为JSON字符串后存储

|         KEY         |                   VALUE                   |
| :-----------------: | :---------------------------------------: |
|  codermast:user:1   |    {“id”:1, “name”: “Jack”, “age”: 21}    |
| codermast:product:1 | {“id”:1, “name”: “小米11”, “price”: 4999} |



#### 6.1.2List类型

> Redis中的List类型与Java中的LinkedList类似，可以看做是一个双向链表结构。既可以支持正向检索和也可以支持反向检索。

特征也与`LinkedList`类似：

- 有序
- 元素可以重复
- 插入和删除快
- 查询速度一般

常用来存储一个有序数据，例如：朋友圈点赞列表，评论列表等.

> List的常见命令有

|          命令           |                             描述                             |
| :---------------------: | :----------------------------------------------------------: |
|   LPUSH key element …   |                 向列表左侧插入一个或多个元素                 |
|        LPOP key         |        移除并返回列表左侧的第一个元素，没有则返回nil         |
| **RPUSH key element …** |                 向列表右侧插入一个或多个元素                 |
|        RPOP key         |                移除并返回列表右侧的第一个元素                |
|   LRANGE key star end   |                 返回一段角标范围内的所有元素                 |
|      BLPOP和BRPOP       | 与LPOP和RPOP类似，只不过在没有元素时等待指定时间，而不是直接返回nil |

![双端队列](images/%E5%8F%8C%E7%AB%AF%E9%98%9F%E5%88%97.gif)

#### 6.1.3Hash类型

> Hash类型，也叫散列，其value是一个无序字典，类似于Java中的`HashMap`结构。

- Hash结构可以将对象中的每个字段独立存储，可以针对单个字段做CRUD

![hash类型的数据](images/hash%E7%B1%BB%E5%9E%8B%E7%9A%84%E6%95%B0%E6%8D%AE.png)

- Hash的常见命令有：

  |         命令         |                             描述                             |
  | :------------------: | :----------------------------------------------------------: |
  | HSET key field value |              添加或者修改hash类型key的field的值              |
  |    HGET key field    |                获取一个hash类型key的field的值                |
  |        HMSET         |       hmset 和 hset 效果相同 ，4.0之后hmset可以弃用了        |
  |        HMGET         |              批量获取多个hash类型key的field的值              |
  |       HGETALL        |         获取一个hash类型的key中的所有的field和value          |
  |        HKEYS         |             获取一个hash类型的key中的所有的field             |
  |        HVALS         |             获取一个hash类型的key中的所有的value             |
  |       HINCRBY        |           让一个hash类型key的字段值自增并指定步长            |
  |        HSETNX        | 添加一个hash类型的key的field值，前提是这个field不存在，否则不执行 |

#### 6.1.4Set类型

> Redis的Set结构与Java中的HashSet类似，可以看做是一个value为null的HashMap。因为也是一个hash表，因此具备与HashSet类似的特征

- 无序
- 元素不可重复
- 查找快
- 支持交集、并集、差集等功能

> Set的常见命令有

|         命令         |            描述             |
| :------------------: | :-------------------------: |
|  SADD key member …   |  向set中添加一个或多个元素  |
|  SREM key member …   |     移除set中的指定元素     |
|      SCARD key       |     返回set中元素的个数     |
| SISMEMBER key member | 判断一个元素是否存在于set中 |
|       SMEMBERS       |     获取set中的所有元素     |
|  SINTER key1 key2 …  |     求key1与key2的交集      |
|  SDIFF key1 key2 …   |     求key1与key2的差集      |
|  SUNION key1 key2 …  |     求key1和key2的并集      |

> **交集、差集、并集图示**

![交集、并集、差集](images/%E4%BA%A4%E9%9B%86%E3%80%81%E5%B9%B6%E9%9B%86%E3%80%81%E5%B7%AE%E9%9B%86.png)

#### 6.1.5Sortedset类型

> Redis的SortedSet是一个可排序的set集合，与Java中的TreeSet有些类似，但底层数据结构却差别很大。SortedSet中的每一个元素都带有一个score属性，可以基于score属性对元素排序，底层的实现是一个跳表（SkipList）加 hash表。

SortedSet具备下列特性：

- 可排序
- 元素不重复
- 查询速度快

因为SortedSet的可排序特性，经常被用来实现排行榜这样的功能。

> SortedSet的常见命令有

|             命令             |                             描述                             |
| :--------------------------: | :----------------------------------------------------------: |
|    ZADD key score member     | 添加一个或多个元素到sorted set ，如果已经存在则更新其score值 |
|       ZREM key member        |                删除sorted set中的一个指定元素                |
|      ZSCORE key member       |             获取sorted set中的指定元素的score值              |
|       ZRANK key member       |              获取sorted set 中的指定元素的排名               |
|          ZCARD key           |                  获取sorted set中的元素个数                  |
|      ZCOUNT key min max      |           统计score值在给定范围内的所有元素的个数            |
| ZINCRBY key increment member |    让sorted set中的指定元素自增，步长为指定的increment值     |
|      ZRANGE key min max      |          按照score排序后，获取指定排名范围内的元素           |
|  ZRANGEBYSCORE key min max   |          按照score排序后，获取指定score范围内的元素          |
|    ZDIFF、ZINTER、ZUNION     |                      求差集、交集、并集                      |

注意：所有的排名默认都是升序，如果要降序则在命令的Z后面添加`REV`即可

### 6.2特殊类型

#### 6.2.1GEO类型

GEO，Geographic，地理信息的缩写。该类型，就是元素的2维坐标，在地图上就是经纬度。redis基于该类型，提供了经纬度设置，查询，范围查询，距离查询，经纬度Hash等常见操作。

- geoadd：添加地理位置的坐标。
- geopos：获取地理位置的坐标。
- geodist：计算两个位置之间的距离。
- georadius：根据用户给定的经纬度坐标来获取指定范围内的地理位置集合。
- georadiusbymember：根据储存在位置集合里面的某个地点获取指定范围内的地理位置集合。
- geohash：返回一个或多个位置对象的 geohash 值。

| 命令                                                         | 描述                                                      |
| ------------------------------------------------------------ | --------------------------------------------------------- |
| GEOHASH key member [member ...]                              | 返回一个或多个位置元素的 Geohash 表示                     |
| GEOPOS key member [member ...]                               | 从key里返回所有给定位置元素的位置（经度和纬度）           |
| GEODIST key member1 member2 [m\|km\|ft\|mi]                  | 返回两个给定位置之间的距离                                |
| GEORADIUS key longitude latitude radius m\|km\|ft\|mi [WITHCOORD] [WITHDIST] [WITHHASH] [COUNT count] [ASC\|DESC] [STORE key] [STOREDIST key] | 以给定的经纬度为中心， 找出某一半径内的元素               |
| GEOADD key longitude latitude member [longitude latitude member ...] | 将指定的地理空间位置（纬度、经度、名称）添加到指定的key中 |
| GEORADIUSBYMEMBER key member radius m\|km\|ft\|mi [WITHCOORD] [WITHDIST] [WITHHASH] [COUNT count] [ASC\|DESC] [STORE key] [STOREDIST key] | 找出位于指定范围内的元素，中心点是由给定的位置元素决定    |

#### 6.2.2Bitmap类型

从本质上来说，bitmap不是一种数据类型，本质是字符串key-value，但是其可以对位进行操作。也可以将bitmap想象成一个只能存储0、1的整型数组，可以随时对任意一位进行运算。下标在bitmap中成为偏移量。

>  相关命令

| 命令                                   | 描述                                                         |
| -------------------------------------- | ------------------------------------------------------------ |
| setbit<key><offset><value>             | 设置Bitmaps中某个偏移量的值（0或1）(offset:偏移量从0开始)    |
| getbit<key><offset>                    | 获取Bitmaps中某个偏移量的值（偏移量不存在，也是返回0）       |
| bitcount<key>[start end]               | 统计**字符串**被设置为1的bit数。                             |
| bitop and(or/not/xor) <destkey> [key…] | bitop是一个复合操作， 它可以做多个Bitmaps的and（交集） 、 or（并集） 、 not（非） 、 xor（异或） 操作并将结果保存在destkey中。 |

#### 6.2.3Hyperloglog类型

Redis HyperLogLog 是用来做基数统计的算法，HyperLogLog 的优点是，在输入元素的数量或者体积非常非常大时，计算基数所需的空间总是固定 的、并且是很小的。

在 Redis 里面，每个 HyperLogLog 键只需要花费 12 KB 内存，就可以计算接近 2^64 个不同元素的基 数。这和计算基数时，元素越多耗费内存就越多的集合形成鲜明对比。

但是，因为 HyperLogLog 只会根据输入元素来计算基数，而不会储存输入元素本身，所以 HyperLogLog 不能像集合那样，返回输入的各个元素。

> 相关命令

|                    命令                     | 描述                                                         |
| :-----------------------------------------: | ------------------------------------------------------------ |
|     pfadd <key>< element> [element ...]     | 将所有元素添加到指定HyperLogLog数据结构中。如果执行命令后HLL估计的近似基数发生变化，则返回1，否则返回0。 |
|           pfcount<key> [key ...]            | 计算HLL的近似基数，可以计算多个HLL，比如用HLL存储每天的UV，计算一周的UV可以使用7天的UV合并计算即可 |
| pfmerge<destkey><sourcekey> [sourcekey ...] | 将一个或多个HLL合并后的结果存储在另一个HLL中，比如每月活跃用户可以使用每天的活跃用户来合并计算可得 |

![知识星球推广](images/%E7%9F%A5%E8%AF%86%E6%98%9F%E7%90%83%E6%8E%A8%E5%B9%BF.jpeg)![公众号-爱编程的mast](images/%E5%85%AC%E4%BC%97%E5%8F%B7-%E7%88%B1%E7%BC%96%E7%A8%8B%E7%9A%84mast.png)