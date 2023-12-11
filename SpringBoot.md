# SpringBoot

用来简化spring应用的初始搭建以及开发过程 使用特定的方式来进行配置（properties或yml文件）

创建独立的spring引用程序 main方法运行

嵌入的Tomcat 无需部署war文件

简化maven配置

自动配置spring添加对应功能starter自动化配置

> spring boot来简化spring应用开发，约定大于配置，去繁从简，just run就能创建一个独立的，产品级别的应用

### Springboot 优点

- 减少开发，测试时间和努力。
- 使用`JavaConfig`有助于避免使用XML。
- 避免大量的`Maven`导入和各种版本冲突。
- 提供意见发展方法。
- 通过提供默认值快速开始开发。
- 没有单独的Web服务器需要。这意味着你不再需要启动Tomcat，Glassfish或其他任何东西。
- 需要更少的配置 因为没有web.xml文件。只需添加用@ Configuration注释的类，然后添加用@Bean注释的方法，Spring将自动加载对象并像以前一样对其进行管理。您甚至可以将@Autowired添加到bean方法中，以使Spring自动装入需要的依赖关系中。
- 基于环境的配置 使用这些属性，您可以将您正在使用的环境传递到应用程序：-Dspring.profiles.active = {enviornment}。在加载主应用程序属性文件后，Spring将在（application{environment} .properties）中加载后续的应用程序属性文件。

### Spring Boot 目录结构

```java
 +- javastack
     +- MyApplication.java
     |
     +- customer
     |   +- Customer.java
     |   +- CustomerController.java
     |   +- CustomerService.java
     |   +- CustomerRepository.java
     |
     +- order
         +- Order.java
         +- OrderController.java
         +- OrderService.java
         +- OrderRepository.java
```

这个目录结构是主流及推荐的做法，而在主入口类上加上 `@SpringBootApplication` 注解来开启 Spring Boot 的各项能力，如自动配置、组件扫描等。

### Spring Boot 常用注解

启动类上面的注解是@SpringBootApplication，它也是 Spring Boot 的核心注解，主要组合包含了以下 3 个注解：

**@SpringBootConfiguration：** 组合了 @Configuration 注解，实现配置文件的功能。

**@EnableAutoConfiguration：** 打开自动配置的功能，也可以关闭某个自动配置的选项，如关闭数据源自动配置功能：@SpringBootApplication(exclude = { DataSourceAutoConfiguration.class })。

**@ComponentScan** Spring组件扫描。

### 如何在自定义端口上运行 Spring Boot应用程序?

在 `application.properties`中指定端口`serverport=8090`。

### Spring Boot 初始化环境变量流程

**1、** 调用`prepareEnvironment`方法去设置环境变量

**2、** 接下来有三个方法`getOrCreateEnvironment`，`configureEnvironment`，`environmentPrepared`

**3、** `getOrCreateEnvironment`去初始化系统环境变量

**4、** `configureEnvironment`去初始化命令行参数

**5、** `environmentPrepared`当广播到来的时候调用`onApplicationEnvironmentPreparedEvent`方法去使用`postProcessEnvironment`方法`load yml`和`properties变量`

### Spring Boot 扫描流程

**1、** 调用run方法中的`refreshContext`方法

**2、** 用AbstractApplicationContext中的`refresh`方法

**3、** 委托给`invokeBeanFactoryPostProcessors`去处理调用链

**4、** 其中一个方法`postProcessBeanDefinitionRegistry会`去调用`processConfigBeanDefinitions`解析`beandefinitions`

**5、** 在`processConfigBeanDefinitions`中有一个`parse`方法，其中有`componentScanParser.parse`的方法，这个方法会扫描当前路径下所有`Component`组件

### Spring Boot 配置加载顺序

使用 Spring Boot 会涉及到各种各样的配置，如开发、测试、线上就至少 3 套配置信息了。Spring Boot 可以轻松的帮助我们使用相同的代码就能使开发、测试、线上环境使用不同的配置。

**在 Spring Boot 里面，可以使用以下几种方式来加载配置。本章内容基于 Spring Boot 2.0 进行详解。**

1、properties文件；

2、YAML文件；

3、系统环境变量；

4、命令行参数；

等等……

**配置属性加载的顺序如下：**

```java
1、开发者工具 `Devtools` 全局配置参数；

2、单元测试上的 `@TestPropertySource` 注解指定的参数；

3、单元测试上的 `@SpringBootTest` 注解指定的参数；

4、命令行指定的参数，如 `java -jar springboot.jar --name="Java技术栈"`；

5、命令行中的 `SPRING_APPLICATION_JSONJSON` 指定参数, 如 `java -Dspring.application.json='{"name":"Java技术栈"}' -jar springboot.jar`

6、`ServletConfig` 初始化参数；

7、`ServletContext` 初始化参数；

8、JNDI参数（如 `java:comp/env/spring.application.json`）；

9、Java系统参数（来源：`System.getProperties()`）；

10、操作系统环境变量参数；

11、`RandomValuePropertySource` 随机数，仅匹配：`ramdom.*`；

12、JAR包外面的配置文件参数（`application-{profile}.properties（YAML）`）

13、JAR包里面的配置文件参数（`application-{profile}.properties（YAML）`）

14、JAR包外面的配置文件参数（`application.properties（YAML）`）

15、JAR包里面的配置文件参数（`application.properties（YAML）`）

16、`@Configuration`配置文件上 `@PropertySource` 注解加载的参数；

17、默认参数（通过 `SpringApplication.setDefaultProperties` 指定）；
```

**数字小的优先级越高，即数字小的会覆盖数字大的参数值**

### Spring Boot 自动配置原理

只要使用了`@EnableAutoConfiguration`注解就能实现自动配置。

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration  
```

`@Import(AutoConfigurationImportSelector.class)`这个是自动配置的关键，它完成了自动配置的主要逻辑。 下面是代码的主要片段

```java
@Override
public String[] selectImports(AnnotationMetadata annotationMetadata) {
    if (!isEnabled(annotationMetadata)) {
        return NO_IMPORTS;
    }
   // 获取配置的元数据
   AutoConfigurationMetadata autoConfigurationMetadata = AutoConfigurationMetadataLoader
            .loadMetadata(this.beanClassLoader);
   // 这个方法包含了加载的主要逻辑，它能找到所有自动注入的类
   AutoConfigurationEntry autoConfigurationEntry = getAutoConfigurationEntry(
            autoConfigurationMetadata, annotationMetadata);
   return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
}
```

下面来看`getAutoConfigurationEntry`的主要逻辑

```java
protected AutoConfigurationEntry getAutoConfigurationEntry(
        AutoConfigurationMetadata autoConfigurationMetadata,
        AnnotationMetadata annotationMetadata) {
    // ....
    // 获取候选配置类
    List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);
    // ... 过滤、去重、排除一些配置类
    return new AutoConfigurationEntry(configurations, exclusions);
}
```

> `SpringFactoriesLoader`是spring提供的一个扩展机制，它能加载模块下的`META-INF/sring.factories`文件，这个Properties格式的文件中的key是接口、注解、或抽象类的全名，value是以逗号“,"分隔的实现类。SpringFactoriesLoader能将相应的实现类注入Spirng容器中。

查看一下spring-boot-autoconfigure模块下的`META-INF/spring.factories`文件

```java
...
# Auto Configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
...
```

就可以看出来它加载哪些自动配置类；通过这个机制我们可以自己写自动配置类并且在模块下的`META-INF/spring.factories`文件中写入

```java
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.yuan.interview.springboot.XXXAutoConfiguration
```

这样直接引入我们的模块就会加载`org.yuan.interview.springboot.XXXAutoConfiguration`配置类了。

### 如何禁用特定自动配置类

**1、**使用`@EnableAutoConfiguration`的`exclude`属性。

```java
@EnableAutoConfiguration(exclude={DataSourceAutoConfiguration.class})
```

**2、**如果类不在类路径上，可以使用`@EnableAutoConfiguration`的`excludeName`属性

```java
@EnableAutoConfiguration(excludeName={Foo.class})
```

**3、**可以使用配置spring.autoconfigure.exclude属性来控制要排除的自动配置类列表。 在application.properties配置文件中配置属性，并且可以使用逗号分隔添加多个类。

### Spring Boot 监视器

Spring boot actuator是spring启动框架中的重要功能之一。Spring boot监视器可帮助您访问生产环境中正在运行的应用程序的当前状态。

有几个指标必须在生产环境中进行检查和监控。即使一些外部应用程序可能正在使用这些服务来向相关人员触发警报消息。监视器模块公开了一组可直接作为HTTP URL访问的REST端点来检查状态。

### Spring Batch

`Spring Boot Batch`提供可重用的函数，这些函数在处理大量记录时非常重要；包括日志/跟踪，事务管理，作业处理统计信息，作业重新启动，跳过和资源管理。它还提供了更先进的技术服务和功能，通过优化和分区技术，可以实现极高批量和高性能批处理作业。简单以及复杂的大批量批处理作业可以高度可扩展的方式利用框架处理重要大量的信息。

### Spring Boot 多环境配置

提供多套配置文件，如：

```xml
applcation.properties

application-dev.properties

application-test.properties

application-prod.properties
```

然后在applcation.properties文件中指定当前的环境spring.profiles.active=test,这时候读取的就是application-test.properties文件。

### Spring Boot 读取配置的方式

Spring Boot 可以通过

- @PropertySource
- @Value
- @Environment,
- @ConfigurationProperties

来绑定变量

### SpringBoot 实现热部署

这可以使用 DEV 工具来实现。

通过这种依赖关系，您可以节省任何更改，嵌入式tomcat 将重新启动。

Spring Boot 有一个开发工具（DevTools）模块，它有助于提高开发人员的生产力。

Java 开发人员面临的一个主要挑战是将文件更改自动部署到服务器并自动重启服务器。开发人员可以重新加载 Spring Boot 上的更改，而无需重新启动服务器。这将消除每次手动部署更改的需要。Spring Boot 在发布它的第一个版本时没有这个功能。

这是开发人员最需要的功能。DevTools 模块完全满足开发人员的需求。该模块将在生产环境中被禁用。它还提供 H2 数据库控制台以更好地测试应用程序。

### Spring Boot 日志框架

Spring Boot 支持 Java Util Logging, Log4j2, Lockback 作为日志框架，如果你使用 Starters 启动器，Spring Boot 将使用 Logback 作为默认日志框架

### Spring Boot Starters

Starters可以理解为启动器，它包含了一系列可以集成到应用里面的依赖包，你可以一站式集成 Spring 及其他技术，而不需要到处找示例代码和依赖包。如你想使用 Spring JPA 访问数据库，只要加入 spring-boot-starter-data-jpa 启动器依赖就能使用了。

Starters包含了许多项目中需要用到的依赖，它们能快速持续的运行，都是一系列得到支持的管理传递性依赖。

### spring-boot-starter-parent

我们都知道，新创建一个 Spring Boot 项目，默认都是有 parent 的，这个 parent 就是 spring-boot-starter-parent ，spring-boot-starter-parent 主要有如下作用：

定义了 Java 编译版本为 1.8 。
使用 UTF-8 格式编码。
继承自 spring-boot-dependencies，这个里边定义了依赖的版本，也正是因为继承了这个依赖，所以我们在写依赖时才不需要写版本号。
执行打包操作的配置。
自动化的资源过滤。
自动化的插件配置。
针对 application.properties 和 application.yml 的资源过滤，包括通过 profile 定义的不同环境的配置文件，例如 application-dev.properties 和 application-dev.yml。

### SpringBoot常用starter

**1、** `spring-boot-starter-web` (嵌入tomcat和web开发需要servlet与jsp支持)

**2、** `spring-boot-starter-data-jpa` (数据库支持)

**3、** `spring-boot-starter-data-redis` (redis数据库支持)

**4、** `spring-boot-starter-data-solr` (solr搜索应用框架支持)

**5、** `mybatis-spring-boot-starter` (第三方的mybatis集成starter)

### Spring Boot 的核心配置文件有哪几个？它们的区别是什么？

Spring Boot 的核心配置文件是 application 和 bootstrap 配置文件。

application 配置文件这个容易理解，主要用于 Spring Boot 项目的自动化配置。

bootstrap 配置文件有以下几个应用场景。

- 使用 Spring Cloud Config 配置中心时，这时需要在 bootstrap 配置文件中添加连接到配置中心的配置属性来加载外部配置中心的配置信息；
- 一些固定的不能被覆盖的属性；
- 一些加密/解密的场景；

### 运行 Spring Boot 有哪几种方式？

**1、**打包用命令或者放到容器中运行

**2、**用 Maven/ Gradle 插件运行

**3、**直接执行 main 方法运行

### Spring Boot 打成的 jar 和普通的 jar 有什么区别 ?

Spring Boot 项目最终打包成的 jar 是可执行 jar ，这种 jar 可以直接通过 java -jar xxx.jar 命令来运行，这种 jar 不可以作为普通的 jar 被其他项目依赖，即使依赖了也无法使用其中的类。

Spring Boot 的 jar 无法被其他项目依赖，主要还是他和普通 jar 的结构不同。普通的 jar 包，解压后直接就是包名，包里就是我们的代码，而 Spring Boot 打包成的可执行 jar 解压后，在 \BOOT-INF\classes 目录下才是我们的代码，因此无法被直接引用。如果非要引用，可以在 pom.xml 文件中增加配置，将 Spring Boot 项目打包成两个 jar ，一个可执行，一个可引用。

### 如何使用 Spring Boot 实现分页和排序？

使用 Spring Boot 实现分页非常简单。使用 Spring Data-JPA 可以实现将可分页的传递给存储库方法。

### Spring Boot 中如何实现定时任务 ?

定时任务也是一个常见的需求，Spring Boot 中对于定时任务的支持主要还是来自 Spring 框架。

在 Spring Boot 中使用定时任务主要有两种不同的方式，一个就是使用 Spring 中的 @Scheduled 注解，另一个则是使用第三方框架 Quartz。

使用 Spring 中的 @Scheduled 的方式主要通过 @Scheduled 注解来实现。

使用 Quartz ，则按照 Quartz 的方式，定义 Job 和 Trigger 即可。

### Spring Boot 还提供了其它的哪些 Starter Project Options？

Spring Boot 也提供了其它的启动器项目包括，包括用于开发特定类型应用程序的典型依赖项。

- spring-boot-starter-web-services - SOAP Web Services
- spring-boot-starter-web - Web 和 RESTful 应用程序
- spring-boot-starter-test - 单元测试和集成测试
- spring-boot-starter-jdbc - 传统的 JDBC
- spring-boot-starter-hateoas - 为服务添加 HATEOAS 功能
- spring-boot-starter-security - 使用 SpringSecurity 进行身份验证和授权
- spring-boot-starter-data-jpa - 带有 Hibeernate 的 Spring Data JPA
- spring-boot-starter-data-rest - 使用 Spring Data REST 公布简单的 REST 服务

### 为什么我们需要 spring-boot-maven-plugin?

spring-boot-maven-plugin 提供了一些像 jar 一样打包或者运行应用程序的命令。

**1、**spring-boot:run 运行你的 SpringBooty 应用程序。

**2、**spring-boot：repackage 重新打包你的 jar 包或者是 war 包使其可执行

**3、**spring-boot：start 和 spring-boot：stop 管理 Spring Boot 应用程序的生命周期（也可以说是为了集成测试）。

**4、**spring-boot:build-info 生成执行器可以使用的构造信息。

# **YAML**

YAML是一种人类可读的数据序列化语言。它通常用于配置文件。 与属性文件相比，如果我们想要在配置文件中添加复杂的属性，YAML文件就更加结构化，而且更少混淆。可以看出YAML具有分层配置数据。

## YAML 配置优势

YAML 现在可以算是非常流行的一种配置文件格式了，无论是前端还是后端，都可以见到 YAML 配置。那么 YAML 配置和传统的 properties 配置相比到底有哪些优势呢？

1. 配置有序，在一些特殊的场景下，配置有序很关键
2. 支持数组，数组中的元素可以是基本数据类型也可以是对象
3. 简洁

相比 properties 配置文件，YAML 还有一个缺点，就是不支持 @PropertySource 注解导入自定义的 YAML 配置。

### Spring Boot 是否可以使用 XML 配置 ?

Spring Boot 推荐使用 Java 配置而非 XML 配置，但是 Spring Boot 中也可以使用 XML 配置，通过 @ImportResource 注解可以引入一个 XML 配置。

### spring boot 核心配置文件是什么？bootstrap.properties 和 application.properties 有何区别 ?

单纯做 Spring Boot 开发，可能不太容易遇到 bootstrap.properties 配置文件，但是在结合 Spring Cloud 时，这个配置就会经常遇到了，特别是在需要加载一些远程配置文件的时侯。

**spring boot 核心的两个配置文件：**

- **bootstrap (. yml 或者 . properties)：** boostrap 由父 ApplicationContext 加载的，比 applicaton 优先加载，配置在应用程序上下文的引导阶段生效。一般来说我们在 Spring Cloud Config 或者 Nacos 中会用到它。且 boostrap 里面的属性不能被覆盖；
- **application (. yml 或者 . properties)：** 由ApplicatonContext 加载，用于 spring boot 项目的自动化配置。

### 什么是 Spring Profiles？

Spring Profiles 允许用户根据配置文件（dev，test，prod 等）来注册 bean。因此，当应用程序在开发中运行时，只有某些 bean 可以加载，而在 PRODUCTION中，某些其他 bean 可以加载。假设我们的要求是 Swagger 文档仅适用于 QA 环境，并且禁用所有其他文档。这可以使用配置文件来完成。Spring Boot 使得使用配置文件非常简单。

### 什么是JavaConfig？

Spring JavaConfig是Spring社区的产品，它提供了配置Spring IoC容器的纯Java方法。因此它有助于避免使用XML配置。使用JavaConfig的优点在于：

面向对象的配置。由于配置被定义为JavaConfig中的类，因此用户可以充分利用Java中的面向对象功能。一个配置类可以继承另一个，重写它的@Bean方法等。

减少或消除XML配置。基于依赖注入原则的外化配置的好处已被证明。但是，许多开发人员不希望在XML和Java之间来回切换。

JavaConfig为开发人员提供了一种纯Java方法来配置与XML配置概念相似的Spring容器。

从技术角度来讲，只使用JavaConfig配置类来配置容器是可行的，但实际上很多人认为将JavaConfig与XML混合匹配是理想的。

类型安全和重构友好。JavaConfig提供了一种类型安全的方法来配置Spring容器。由于Java 5.0对泛型的支持，现在可以按类型而不是按名称检索bean，不需要任何强制转换或基于字符串的查找。Spring JavaConfig是Spring社区的产品，它提供了配置Spring IoC容器的纯Java方法。因此它有助于避免使用XML配置。使用JavaConfig的优点在于：

面向对象的配置。由于配置被定义为JavaConfig中的类，因此用户可以充分利用Java中的面向对象功能。一个配置类可以继承另一个，重写它的@Bean方法等。

减少或消除XML配置。基于依赖注入原则的外化配置的好处已被证明。但是，许多开发人员不希望在XML和Java之间来回切换。

JavaConfig为开发人员提供了一种纯Java方法来配置与XML配置概念相似的Spring容器。

从技术角度来讲，只使用JavaConfig配置类来配置容器是可行的，但实际上很多人认为将JavaConfig与XML混合匹配是理想的。

类型安全和重构友好。JavaConfig提供了一种类型安全的方法来配置Spring容器。由于Java 5.0对泛型的支持，现在可以按类型而不是按名称检索bean，不需要任何强制转换或基于字符串的查找。