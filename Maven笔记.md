# Maven学习笔记

# 基础篇

## Maven简介

### Maven是什么？

- Maven的本质是一个项目管理工具，将项目开发和管理过程中

### Maven能干什么

​		不使用maven的时候，我们需要哪个jar包，都需要添加到WEB-INF/lib，每新建一个项目就需要复制这些
​		从而造成工作区中存在大量重复的文件，让我们的工程显得很臃肿。
​		而使用 Maven 后每个 jar 包本身只在本地仓库中保存一份，需要 jar 包的工程只需要以坐标的方式
简单的引用一下就可以了。不仅极大的节约了存储空间，让项目更轻巧，更避免了重复文件太多而造成
的混乱。

### 什么是POM？

- POM：Project Object Model

### 坐标

#### 概念

Maven中的坐标用于描述仓库中资源的位置，即具体的目录结构。

#### 作用

使用唯一的标识，唯一性的定位资源位置，通过该标识能够将资源的识别与下载工作交给机器来完成。即告诉机器我需要的依赖的具体位置，机器负责进行下载安装。

#### 组成

- 组织ID：GroupID

> 定义当前Maven项目隶属于的组织名称（通常为域名的反写，com.codermast）

- 项目ID：ArtifactID

> 定义当前Maven项目名称（通常是模块的名称，如Log4j，Jackson等）

- 版本号：Version

> 定义当前项目的版本号

- 打包方式：Packaging（非必选）

> 定义项目打包的方式，可分为Jar包的War包

## 仓库

仓库就是存放各类Jar包的地址，分为两种类型，本地仓库和远程仓库

- 本地仓库

  > 本地仓库就是部署在本地计算机的一个jar包仓库，一般存储自身项目开发所需要的jar包和自己开发好的jar包

- 远程仓库

  - 中央仓库

    > 中央仓库就是全世界通用公开的仓库，我们所需要的所有的公开的Jar包，都能在这里找到，并且下载。

  - 私服

    > 一般为我们自己搭建的一个仅供自己使用的一个Jar包远程仓库，一个局域仓库，可以存放一些不公开的jar包。

### 修改本地仓库地址

在Maven安装目录下，有个conf文件夹，在这个文件夹内有个settings.xml文件，这个xml文件就是我们Maven的配置文件，所有的全局配置都在这个文件内进行配置。

找到 这一行配置，这里表示的是默认情况下的本地仓库地址

```xml
 <localRepository>/path/to/local/repo</localRepository>
```

我们在其该行下的正文处增加一行，如下的代码

```xml
<localRepository>G:\maven\repository</localRepository>
```

该标签内的内容即为我们设置的本地仓库地址，这里配置完后Maven不会将原来的仓库删除，只是在下一次需要该依赖的时候，再下载到这个仓库。

### 增加镜像地址

默认情况下我们下载依赖是直接去Maven官方的中央仓库去下载的，因为其服务器在国外，访问速度会比较的慢，我们这里可以使用国内的一些镜像源。

在配置的时候，仍然是在settings.xml文件内进行配置，找到<mirrors>标签，所有的镜像信息都在这个父标签内。

```xml
<mirror>
	<id>aliyunmaven</id>
	<mirrorOf>central</mirrorOf>
	<name>阿里云公共仓库</name>
	<url>https://maven.aliyun.com/repository/central</url>
</mirror>
```

- mirror：代表的是这是一个镜像的配置
- id：代表镜像的id
- mirrorOf：代表镜像的仓库名称
- name：代表镜像名称
- url：代表仓库的地址

### 项目构建命令

> Maven构建命令使用mvn开头，空格间隔后加上需要的功能，一次可以执行多个命令，使用空格间隔。

```shell
mvn compile			# 编译
mvn clean			# 清理
mvn test 			# 测试
mvn package			# 打包
mvn install 		# 安装到本地仓库
```

## 目录结构

```java
- src
	- main                   ：程序功能代码
		- Java               ：java代码
		- resource           ：资源代码、配置代码
	- test                   ：测试代码
		- Java               ：单元测试java代码
		- resource           ：资源代码、配置代码
	- pom.xml                ：项目对象模型
```

## 创建项目

- 手动创建

> 根据上面的目录结构自己手动创建对应的文件夹，创建对应的工程pom文件，即可完成构建。

- 命令行创建

```shell
mvn archetype:generate
-DgroupId=com.companyname.bank 
-DartifactId=consumerBanking 
-DarchetypeArtifactId=maven-archetype-quickstart 
-DinteractiveMode=false
```

该命令使用了maven-archetype-quickstart插件创建项目。

- 使用IDE进行创建（最常用，也最好用）

一般为IDEA或者Eclips等一些集成环境创建即可。

## 依赖管理

### 依赖传递

#### 依赖具有传递性

- 直接依赖：在项目中通过依赖配置项直接建立的依赖关系，即为直接依赖

- 间接依赖：没有直接通过以来配置项建立依赖关系，但是依赖中又依赖了别的没有直接依赖的依赖，此时为间接依赖。



举个简单的例子，在依赖C中使用了依赖A和依赖B，在程序中使用了依赖C，此时程序中虽然没有直接配置依赖A和B，只配置了依赖C，但是由于依赖C中使用了依赖A和依赖B，所以程序中也可以使用依赖A和B。这就是依赖传递。

### 依赖传递冲突问题

- 路径优先：当依赖中出现相同的资源时，层级越深，优先级越低，层级越浅，优先级越高
- 声明优先：当资源在相同层级被依赖时，配置顺序靠前的覆盖配置顺序靠后的。
- 特殊优先：当同级配置了相同的资源的不同版本，后配置的覆盖先配置的。

 ### 可选依赖

可选依赖指对外隐藏当前所依赖的资源不透明：仅不能被外界查看，但实际上是存在的。

```xml
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
    <optional>true</optional>
</dependency>
```

> 这里使用的<optional>标签能够实现，将该项目下的所有依赖不对外公开，即外界无法看到该项目使用了什么依赖。

- true：隐藏，即不公开
- false(默认值)：不隐藏，即公开

### 排除依赖

排除依赖值主动断开依赖的资源，被排除的资源无需指定版本：直接排除该依赖，所以看不到。

```xml
<dependency>
    <exclusions>
        <exclusion>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

### 依赖范围

依赖的jar默认情况可以在任何地方使用，可以通过scope标签设定其作用范围

作用范围

- 主程序范围内有效：（main文件夹范围内）
- 测试程序范围有效：（text文件夹范围内）
- 是否参与打包：（package指令范围内）

| scope         | 主代码 | 测试代码 | 打包 | 范例        |
| ------------- | ------ | -------- | ---- | ----------- |
| compile(默认) | Y      | Y        | Y    | log4j       |
| test          |        | Y        |      | junit       |
| provided      | Y      | Y        |      | servlet-api |
| runtime       |        |          | Y    | jdbc        |

依赖范围在依赖传递时会取交集。

如直接依赖为compile，间接依赖为test，则最后的范围为test，两个都为test或者provite则不会传递，只有能进行打包的才可以进行传递。

### 生命周期

项目构建是有生命周期的：

```mermaid
graph LR
	构建阶段 --> 测试构建--> 测试阶段 --> 打包阶段 --> 安装阶段;
```

```mermaid
graph LR
	compile --> test-compile --> test --> package --> install
```

项目构建是有流程的，必须按照上面的流程进行完成。

例如，测试阶段，就需要先进行构建，没有进行构建，是没有办法进行测试的。

### 三个生命周期的具体过程

#### 1. clean 生命周期

clean 生命周期的目的是清理项目，它包括以下三个阶段。

- pre-clean：执行清理前需要完成的工作。
- clean：清理上一次构建过程中生成的文件，比如编译后的 class 文件等。
- post-clean：执行清理后需要完成的工作。

#### 2. default 生命周期

default 生命周期定义了构建项目时所需要的执行步骤，它是所有生命周期中最核心部分，包含的阶段如下表所述，比较常用的阶段用粗体标记。

| 名称                    | 说明                                                         |
| ----------------------- | ------------------------------------------------------------ |
| validate                | 验证项目结构是否正常，必要的配置文件是否存在                 |
| initialize              | 做构建前的初始化操作，比如初始化参数、创建必要的目录等       |
| generate-sources        | 产生在编译过程中需要的源代码                                 |
| process-sources         | 处理源代码，比如过滤值                                       |
| generate-resources      | 产生主代码中的资源在 classpath 中的包                        |
| process-resources       | 将资源文件复制到 classpath 的对应包中                        |
| compile                 | 编译项目中的源代码                                           |
| process-classes         | 产生编译过程中生成的文件                                     |
| generate-test-sources   | 产生编译过程中测试相关的代码                                 |
| process-test-sources    | 处理测试代码                                                 |
| generate-test-resources | 产生测试中资源在 classpath 中的包                            |
| process-test-resources  | 将测试资源复制到 classpath 中                                |
| test-compile            | 编译测试代码                                                 |
| process-test-classes    | 产生编译测试代码过程的文件                                   |
| test                    | 运行测试案例                                                 |
| prepare-package         | 处理打包前需要初始化的准备工作                               |
| package                 | 将编译后的 class 和资源打包成压缩文件，比如 rar              |
| pre-integration-test    | 做好集成测试前的准备工作，比如集成环境的参数设置             |
| integration-test        | 集成测试                                                     |
| post-integration-test   | 完成集成测试后的收尾工作，比如清理集成环境的值               |
| verify                  | 检测测试后的包是否完好                                       |
| install                 | 将打包的组件以构件的形式，安装到本地依赖仓库中，以便共享给本地的其他项目 |
| deploy                  | 运行集成和发布环境，将测试后的最终包以构件的方式发布到远程仓库中，方便所有程序员共享 |

#### 3. site 生命周期

site 生命周期的目的是建立和发布项目站点。Maven 可以基于 pom 所描述的信息自动生成项目的站点，同时还可以根据需要生成相关的报告文档集成在站点中，方便团队交流和发布项目信息。site 生命周期包括如下阶段。

- pre-site：执行生成站点前的准备工作。
- site：生成站点文档。
- post-site：执行生成站点后需要收尾的工作。
- site-deploy：将生成的站点发布到服务器上。

实现这些生命周期需要通过对应的一些插件来进行实现的。

#### 调用生命周期阶段

- mvn clean

> 用 clean 生命周期的 clean 阶段，实际执行的是 clean 生命周期中的 pre-clean 和 clean 阶段

- mvn test

> 该命令调用 default 生命周期中的 test 阶段。实际执行的阶段包括 validate、initialize、generate-sources…compile…test-compile、process-test-classes、test，也就是把 default 生命周期中从开始到 test 的所有阶段都执行完了，而且是按顺序执行。

- mvn clean install

> 该命令调用 clean 生命周期的 clean 阶段和 default 生命周期的 install 阶段。
>
> 实际执行的是 clean 生命周期中的 pre-clean、clean 两个阶段和 default 生命周期中从开始的 validate 到 install 的所有阶段。

- mvn clean deploy site-deploy

> 该命令调用 clean 生命周期中的 pre-clean、clean 阶段，default 生命周期中从 validate 到 deploy 的所有阶段，以及 site 生命周期中的 pre-site、site、post-site 和 site-deploy 阶段。

最后的结果是把该项目编译、测试、打包好发布到远程仓库，同时还将生成好的站点发布到站点服务器。

