# 1 Spring

> `Spring`是一个开源的`Java`应用程序框架，用于构建企业级应用程序，提供了全面的编程和配置模型，可以简化Java开发过程，使开发人员能够更轻松地构建可靠、高效和可扩展的应用程序

## 1.1 Spring核心模块

| 核心模块                  | 说明                                                                                                   |
|:---------------------:|:----------------------------------------------------------------------------------------------------:|
| 容器                    | 容器提供Spring框架的基本功能，Spring以bean的方式组织和管理Java应用中的各个组件及其关系                                                |
| Context 应用上下文         | 向Spring框架提供上下文信息。Spring上下文包括企业服务，如JNDI、EJB、电子邮件、国际化、校验和调度功能                                          |
| AOP 面向切面编程            | Spring的AOP是基于动态代理实现的，实现的方式有两种分别是Schema和AspectJ这两种方式                                                  |
| Spring Dao JDBC和Dao模块 | JDBC、DAO的抽象层提供了有意义的异常层次结构，可用该结构来管理异常处理，和不同数据库供应商所抛出的错误信息。异常层次结构简化了错误处理，并且极大的降低了需要编写的代码数量，比如打开和关闭链接   |
| ORM 对象实体映射            | Spring框架插入了若干个ORM框架，从而提供了ORM对象的关系工具，其中包括了Hibernate、JDO和 IBatis SQL Map等，所有这些都遵从Spring的通用事物和DAO异常层次结构 |
| Spring Web            | Web上下文模块建立在应用程序上下文模块之上，为基于web的应用程序提供了上下文。所以Spring框架支持与Struts集成，web模块还简化了处理多部分请求以及将请求参数绑定到域对象的工作      |

## 1.3 Spring 配置方式

### 1.3.1 基于 xml 配置

> bean 所需的依赖项和服务在 XML 配置文件中指定

```xml
<bean id="studentbean" class="org.edureka.firstSpring.StudentBean">
 <property name="name" value="Edureka"></property>
</bean>
```

### 1.3.2 基于注解配置

> 在相关的类，方法或字段声明上使用注解，将 bean 配置为组件类本身，而不是使用 XML 来描述 bean 装配。默认情况下，Spring 容器中未打开注解装配，需要在使用它之前在 Spring 配置文件中启用它

```java
@Configuration
public class StudentConfig {
    @Bean
    public StudentBean myStudent() {
        return new StudentBean();
    }
}
```

## 1.4 Spring加载流程

1. **创建Spring容器**
   Spring应用程序的加载流程通常从创建Spring容器开始。Spring容器是整个应用程序的核心，负责管理和协调各个组件的生命周期和依赖关系。Spring提供了不同类型的容器，如`ApplicationContext`和`BeanFactory`，可以根据需要选择适合的容器类型

2. **加载配置文件**
   Spring容器加载配置文件，解析配置信息，并创建相应的对象定义和依赖关系。配置文件可以使用XML、注解或Java配置类等方式进行定义

3. **创建和管理Bean对象**
   Spring容器根据配置信息创建和管理Bean对象。Bean是Spring中的基本组件，代表应用程序中的各种对象。Spring通过依赖注入和控制反转等机制，管理Bean对象的创建、初始化、依赖注入和销毁等过程

4. **解析依赖关系**
   Spring容器解析配置文件中定义的Bean之间的依赖关系。它会根据依赖注入的方式，将依赖的Bean注入到相应的目标Bean中。依赖注入可以通过构造函数注入、Setter方法注入或字段注入等方式进行

5. **初始化Bean**
   Spring容器会调用Bean的初始化方法、应用Bean的后置处理器（如AOP代理、事务管理等）以及执行其他必要的初始化操作对Bean进行初始化

6. **使用应用程序**
   一旦所有的Bean都被初始化和装配完成，Spring容器就准备好供应用程序使用了。开发人员可以从容器中获取所需的Bean，并使用它们来完成业务逻辑

## 1.5 Spring 应用程序组件

### 1.5.1 Bean

> Bean是Spring中的基本组件，代表应用程序中的各种对象。Bean可以是POJO（Plain Old Java Object）、服务、数据访问对象、控制器等。Spring容器负责创建、管理和协调Bean的生命周期和依赖关系

### 1.5.2 BeanFactory / ApplicationContext

> **BeanFactory**：BeanFactory是Spring的基础容器，负责创建、管理和协调Bean。它提供了基本的IOC（控制反转）和DI（依赖注入）功能。BeanFactory在需要时延迟加载Bean，以提高性能和资源利用率。
> 
> ApplicationContext是Spring的核心容器，负责加载、管理和协调应用程序中的Bean。它是BeanFactory的子接口，提供了更多的功能，如国际化支持、事件发布、AOP（面向切面编程）等。ApplicationContext可以通过XML配置文件、注解或Java配置类进行配置。

### 配置文件

> Spring应用程序使用配置文件来定义组件、依赖关系和其他配置信息。配置文件可以使用XML、注解或Java配置类等方式进行定义。配置文件中包含了Bean的定义、依赖关系、属性值等信息，Spring容器会根据配置文件创建和管理相应的Bean

### 依赖注入（Dependency Injection）

> 依赖注入是Spring的核心特性之一，它通过容器自动将依赖注入到Bean中，减少了组件之间的耦合性。依赖注入可以通过构造函数注入、Setter方法注入或字段注入等方式进行

### AOP（Aspect-Oriented Programming）

> AOP是一种编程范式，用于将横切关注点（如日志记录、事务管理）与核心业务逻辑分离。Spring提供了AOP支持，可以通过配置或注解方式将切面逻辑织入到应用程序中的特定方法或类中

### 事务管理

> Spring提供了事务管理的支持，可以对数据库操作、方法调用等进行事务管理。通过声明式事务管理或编程式事务管理，Spring可以管理事务的开始、提交、回滚等操作，确保数据的一致性和完整性

### 注解支持

> Spring支持使用注解来简化配置和开发过程。通过使用注解，可以将配置信息直接添加到代码中，减少了繁琐的XML配置。常见的注解包括@Component、@Autowired、@Service、@Controller等

# 2 Spring Bean

> Bean是Spring中的基本组件，代表应用程序中的各种对象

## 2.1 Spring Bean 生命周期

1. **实例化（Instantiation）**：在这个阶段，Spring容器根据配置信息或注解创建Bean的实例。这可以通过构造函数实例化、工厂方法或通过依赖注入等方式完成
2. **属性赋值（Population）**：在实例化后，Spring容器将会为Bean的属性赋值。这可以通过依赖注入、属性注入或其他方式来完成
3. **初始化（Initialization）**：在属性赋值完成后，Spring容器将调用Bean的初始化方法。这可以通过实现`InitializingBean`接口的`afterPropertiesSet()`方法、使用`@PostConstruct`注解或在配置文件中指定的自定义初始化方法来实现
4. **使用（In Use）**：在初始化完成后，Bean可以被使用。它可以被其他Bean引用，也可以被容器管理的其他组件使用
5. **销毁（Destruction）**：当Bean不再被使用时，Spring容器会销毁它。这可以通过实现`DisposableBean`接口的`destroy()`方法、使用`@PreDestroy`注解或在配置文件中指定的自定义销毁方法来实现

## 2.2 Spring Bean 作用域

| 作用域                | 含义                                                                                                      |
|:------------------:|:-------------------------------------------------------------------------------------------------------:|
| 单例 Singleton       | 单例作用域是默认的作用域。在单例作用域下，Spring容器中只会创建一个Bean的实例，并在容器的整个生命周期内重用该实例。每次请求该Bean时，都会返回同一个实例                      |
| 原型 prototype       | 在默认情况下,spring容器在启动时不实例化prototype的Bean.此外,spring容器将prototype的Bean交给调用者后,就不再管理它的生命周期                      |
| 请求 request         | 每次HTTP请求都会创建一个新的Bean,HTTP请求处理完毕后,销毁这个Bean.该作用域仅适用于webApplicationContext环境                               |
| 会话 session         | 同一个HTTP session共享一个Bean,不同HTTP session使用不同的Bean,当HTTP Session结束后,实例才被销毁.该作用域仅适用于webApplicationContext环境 |
| 全局会话 globalSession | 同一个全局session共享一个Bean,一般用于portlet应用环境,该作用域仅适用于webApplicationContext环境                                    |

## 2.3 Spring Bean 实例化

**1、** 使用BeanFactory作为Spring Bean的工厂类，则所有的bean都是在第一次使用该Bean的时候实例化

**2、** 如果你使用 `ApplicationContext` 作为Spring Bean的工厂类

- 若作用域为 `singleton` 且 `lazy-init`为 `false`，则 `ApplicationContext` 启动的时候就实例化该Bean，并且将实例化的Bean放在一个map结构的缓存中，下次再使用该 Bean的时候，直接从缓存中取 
- 若作用域为 `singleton` 且 `lazy-init` 为 `true`，则该Bean的实例化是在第一次使用该Bean的时候进行实例化。
- 若作用域为 `prototype` 则该Bean的实例化是在第一次使用该Bean的时候进行实例化

## 2.4 内部 Bean

> 内部Bean是一种在其他Bean内部声明和使用的Bean

1. **声明方式**：内部Bean的声明是在外部Bean的属性或构造函数参数中，而不是在容器的顶层声明。它们的声明通常以`<property>`元素或构造函数参数的方式出现。

2. **作用域**：内部Bean的作用域是与包含它们的外部Bean相同的作用域。例如，如果外部Bean是单例作用域的，那么内部Bean也是单例作用域的。

3. **生命周期**：内部Bean的生命周期与外部Bean的生命周期紧密相关。当外部Bean被实例化时，内部Bean也会被实例化，当外部Bean被销毁时，内部Bean也会被销毁。

4. **可见性**：内部Bean对于外部Bean是私有的，外部Bean可以直接访问内部Bean的属性和方法，但外部Bean之外的组件无法直接访问内部Bean。

内部Bean通常用于以下情况：

- **封装性**：内部Bean可以用于封装和组织外部Bean的一部分功能或属性，从而提高代码的可读性和维护性。

- **局部性**：内部Bean通常只在外部Bean内部使用，不需要在其他地方引用或暴露。

- **依赖关系**：内部Bean可以作为外部Bean的依赖项，用于支持外部Bean的某些功能或实现特定的业务逻辑。

```xml
<bean id="outerBean" class="com.example.OuterBean">
    <property name="innerBean">
        <bean class="com.example.InnerBean">
            <!-- 内部Bean的属性配置 -->
        </bean>
    </property>
</bean>
```

## 自动装配

Spring框架提供了自动装配（Autowiring）的功能，它是一种方便的依赖注入的方式，可以自动将合适的Bean注入到需要它们的地方，而无需显式配置每个依赖关系。自动装配可以通过以下方式进行配置和使用：

1. **自动装配模式**：Spring框架支持多种自动装配模式，可以通过在Bean的定义上使用`@Autowired`注解或在XML配置文件中使用`autowire`属性来指定。常用的自动装配模式包括：
   
   - **按类型（By Type）**：根据依赖的类型自动装配。Spring会尝试找到一个与依赖类型匹配的Bean，并将其注入。如果找到多个匹配的Bean，将抛出异常。
   
   - **按名称（By Name）**：根据依赖的名称自动装配。Spring会尝试找到一个与依赖名称匹配的Bean，并将其注入。如果找不到匹配的Bean，将抛出异常。
   
   - **构造函数（Constructor）**：根据构造函数参数的类型或名称自动装配。Spring会尝试通过构造函数参数的类型或名称匹配相应的Bean，并将其注入。
   
   - **自动检测 (autodetect) ：** 该模式自动探测使用构造器自动装配或者 byType 自动装配。首先会尝试找合适的带参数的构造器，如果找到的话就是用构造器自动装配，如果在 bean 内部没有找到相应的构造器或者是无参构造器，容器就会自动选择 byTpe 的自动装配方式 。
   
   - **无（No）**：禁用自动装配。需要手动配置依赖关系，或使用`@Autowired`注解在属性或构造函数上显式指定。

2. **@Autowired注解**：`@Autowired`注解是Spring提供的用于自动装配的核心注解。它可以用于字段、属性的setter方法、构造函数等地方。当Spring遇到被`@Autowired`注解标记的依赖时，会自动寻找匹配的Bean并进行注入。

3. **@Qualifier注解**：`@Qualifier`注解可以与`@Autowired`一起使用，用于解决自动装配时存在多个匹配Bean的问题。通过在`@Qualifier`注解中指定Bean的名称，可以精确指定要注入的Bean。

4. **@Primary注解**：`@Primary`注解可以与`@Autowired`一起使用，用于标记优先注入的Bean。当存在多个匹配的Bean时，Spring会优先选择被`@Primary`注解标记的Bean进行注入。

### 自动装配的局限性

1. **多个匹配的Bean**：当存在多个匹配的Bean时，自动装配可能会导致歧义性。如果无法确定要注入哪个Bean，Spring将抛出异常。这可以通过使用`@Qualifier`注解或`@Primary`注解来解决，明确指定要注入的Bean

2. **复杂依赖关系**：自动装配可能无法满足复杂的依赖关系。例如，如果需要根据不同的条件选择不同的Bean进行注入，或者需要动态地创建依赖关系，自动装配可能无法满足这些要求。在这种情况下，可能需要使用显式的配置来解决依赖关系

3. **装配的可读性**：自动装配可能会使代码的可读性下降，因为依赖关系不再显式地在代码中体现出来。读取代码时，可能不太容易确定哪些Bean被注入到了哪些地方。这可以通过在代码中添加适当的注释和文档来缓解

4. **测试困难**：自动装配可能会增加单元测试的复杂性。在测试中，需要确保正确地注入了所需的Bean，并且不会发生意外的自动装配。这可能需要使用模拟对象或手动注入依赖来进行测试

5. **框架限制**：自动装配是Spring框架的一项功能，因此它受到框架本身的限制。某些特定的场景和需求可能无法通过自动装配来满足，需要借助其他手段或自定义解决方案

### 启动注解装配

默认情况下，Spring 容器中未打开注解装配。因此，要使用基于注解装配，我们必须通过配置<context：annotation-config /> 元素在 Spring 配置文件中启用它

# 3 控制反转 （Inversion of Control，IoC）

> IoC是Spring框架的核心概念之一，是一种设计原则和实现模式，旨在降低代码的耦合度、提高可维护性和可测试性。在传统的编程模型中，对象的创建和依赖关系的管理通常由开发者手动完成。这意味着对象之间的关系紧密耦合，难以灵活地进行修改和扩展。而IoC通过将对象的创建和依赖关系的管理转移到框架中，实现了控制的反转。在Spring中，IoC由容器（Container）来实现。容器负责创建和管理应用程序中的对象，以及它们之间的依赖关系。开发者只需要配置好对象的定义和它们之间的关系，容器就会负责实例化对象并自动解决它们的依赖关系。

### 3.1 IoC 优势

> 1. 低耦合度 ：传统的编程模型中，对象之间的依赖关系通常是硬编码在代码中的，这导致对象之间紧密耦合，难以修改和扩展。而通过IoC，依赖关系由容器来管理，对象之间的耦合度大大降低。这使得代码更加灵活、可维护和可扩展
> 
> 2. 可测试性 ：传统的编程模型中对象之间的依赖关系紧密耦合，很难对单个对象进行独立的测试。通过IoC可以实现对单个对象的独立测试，更方便地编写单元测试和集成测试，提高代码的可测试性
> 
> 3. 促进代码的重用和模块化 ：IoC通过将对象的创建和依赖关系的管理集中在容器中，使得这些对象可以在不同的环境中被重复使用。开发者只需要通过配置来定义对象和它们之间的关系，而不需要关注对象的创建和销毁过程。这促进了代码的重用和模块化，提高了开发效率
> 
> 4. 简化对象生命周期管理 ：在传统的编程模型中，对象的创建、初始化和销毁通常由开发者手动管理。而通过IoC容器，对象的生命周期可以由容器自动管理。容器可以负责对象的创建、初始化和销毁，并确保对象的生命周期得到正确管理，避免资源泄漏和内存泄漏等问题

## 3.2 IoC容器分类

### 3.2.1 BeanFactory

> BeanFactory是IoC容器的最底层实现，提供了基本的IoC功能，包括对象的创建、初始化和依赖注入。BeanFactory容器是<mark>延迟初始化</mark>的，即在需要使用对象时才进行实例化，适用于资源受限的环境或者需要手动控制对象的生命周期的场景。

#### 3.2.1.1 BeanFactory 实现

> - XmlBeanFactory是最基本的BeanFactory实现，通过读取XML配置文件来创建BeanFactory容器。它会解析XML配置文件中的Bean定义，根据配置信息创建和管理对象

```java
Resource resource = new ClassPathResource("applicationContext.xml");
BeanFactory beanFactory = new XmlBeanFactory(resource);
```

> - DefaultListableBeanFactory是BeanFactory的默认实现，它可以通过多种方式配置和注册Bean定义。可以通过XML配置文件、注解配置、编程方式等来定义和注册Bean

```java
DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();
// 注册Bean定义
beanFactory.registerBeanDefinition("myBean", new GenericBeanDefinition());
```

#### 3.2.1.2 BeanFactory延迟实例化的优缺点

> - 节省资源：延迟实例化可以避免在容器初始化阶段创建所有Bean对象，从而节省了系统资源。特别是对于大型系统或包含大量Bean对象的应用程序，延迟实例化可以减少启动时间和内存消耗
> 
> - 提高性能：延迟实例化可以避免不必要的对象创建和初始化过程，从而提高了系统的性能。只有在需要使用某个Bean对象时才进行实例化，可以减少不必要的开销
> 
> - 解决循环依赖：延迟实例化可以解决循环依赖的问题。当存在循环依赖时，如果所有Bean对象都在容器初始化阶段创建，可能会导致无法解决的循环依赖情况。而延迟实例化可以通过在获取Bean对象时进行依赖注入，从而避免了循环依赖的问题

> - 延迟加载带来的延迟：延迟实例化可能会导致在首次访问Bean对象时出现一定的延迟，因为需要进行对象的创建和初始化。这对于某些对响应时间要求较高的应用场景可能会有一定的影响
> 
> - 难以控制对象的生命周期：延迟实例化使得对象的创建和初始化时机由容器控制，开发者可能难以精确地控制对象的生命周期。如果需要在对象创建之前或之后执行某些特定的操作，可能需要额外的配置或处理

### 3.2.2 ApplicationContext

> ApplicationContext是BeanFactory容器的扩展，在启动时就实例化和初始化所有的对象，并提供了更丰富的功能，如国际化支持、事件发布、面向切面编程等。ApplicationContext容器适用于大多数应用场景，并且提供了更方便的配置和管理方式

#### 3.2.2.1 ApplicationContext实现

> 1. **ClassPathXmlApplicationContext**：通过读取类路径下的XML配置文件来创建ApplicationContext容器。它会根据XML配置文件中定义的Bean信息来实例化和管理对象
> 
> 2. **FileSystemXmlApplicationContext**：通过读取文件系统中的XML配置文件来创建ApplicationContext容器。相比于ClassPathXmlApplicationContext，它可以指定XML配置文件的绝对路径
> 
> 3. **AnnotationConfigApplicationContext**：通过扫描指定的Java类或包中的注解配置来创建ApplicationContext容器。它不需要使用XML配置文件，而是通过注解来定义和配置Bean
> 
> 4. **WebApplicationContext**：WebApplicationContext是ApplicationContext容器的Web应用程序扩展，用于Web应用程序中，提供了与Web相关的功能和特性，如处理Web请求、管理Web会话、处理Web事件等
> 
> 5. **XmlWebApplicationContext**：用于Web应用程序的ApplicationContext容器实现。它通过读取Web应用程序的WEB-INF目录下的XML配置文件来创建容器，并与Spring MVC框架进行无缝集成
> 
> 6. **AnnotationConfigWebApplicationContext**：用于Web应用程序的基于注解的ApplicationContext容器实现。它通过扫描指定的Java类或包中的注解配置来创建容器，并与Spring MVC框架进行无缝集成

### 3.2.3 Bean Factory 与 ApplicationContext 的区别

> 1. **初始化时机**：BeanFactory是延迟初始化的容器，即在需要使用对象时才进行实例化。这意味着在启动时，BeanFactory不会主动创建和初始化所有的对象，而是在第一次访问对象时才进行创建。相比之下，ApplicationContext在启动时就实例化和初始化所有的对象，提前完成对象的创建和依赖注入
> 
> 2. **功能特点**：ApplicationContext是BeanFactory的扩展，提供了更多的功能和特性。ApplicationContext支持国际化消息源、事件发布和监听、AOP等高级特性。它还可以与Spring MVC框架无缝集成，提供更方便的Web开发支持。相比之下，BeanFactory提供的功能相对较少，主要关注对象的创建、初始化和依赖注入
> 
> 3. **配置方式**：BeanFactory通常使用XML配置文件来定义和配置对象，通过读取XML配置文件来创建和管理对象。而ApplicationContext不仅支持XML配置方式，还支持基于注解的配置和基于Java的配置方式。ApplicationContext可以通过扫描注解或者Java类来创建和管理对象，避免了繁琐的XML配置
> 
> 4. **性能**：由于ApplicationContext在启动时就实例化和初始化所有的对象，因此在性能上相对于BeanFactory有一定的优势。ApplicationContext在启动阶段会预加载和缓存对象，提高了对象的访问速度。而BeanFactory是延迟初始化的，每次访问对象时都需要进行实例化，相对而言性能较低

## 3.3 IoC实现机制

> Spring的IoC容器主要通过以下两种方式来实现控制反转：
> 
> 1. 依赖注入：通过依赖注入，容器会在对象创建时自动将其所依赖的其他对象注入到它们的属性、构造函数或者方法中。这样，对象之间的依赖关系由容器来管理，而不是由对象自己去创建或查找依赖对象
> 2. 配置元数据：开发者可以使用配置元数据（如XML配置文件、注解或Java配置类）来描述对象的定义和它们之间的关系。容器会根据这些配置元数据来实例化对象，并自动解决它们的依赖关系

- 定义Bean：首先，需要在容器中定义要管理的Bean对象。可以通过XML配置文件、注解或Java配置类等方式进行Bean的定义。
- 创建容器：接下来，需要创建一个IoC容器，负责管理Bean的生命周期和依赖关系。
- 解析配置：容器会解析配置文件或扫描注解，读取Bean的定义信息，包括Bean的类型、依赖关系、初始化方法、销毁方法等。
- 创建Bean实例：容器根据Bean的定义信息，使用合适的实例化策略（如构造函数实例化、工厂方法实例化等）创建Bean的实例。
- 处理依赖注入：容器会自动处理Bean之间的依赖关系。它会检查Bean的定义信息，查找并注入所依赖的其他Bean对象。
- 完成初始化：容器完成Bean的实例化和依赖注入后，调用Bean的初始化方法进行一些额外的初始化操作。
- 提供Bean：容器将创建和管理的Bean对象提供给应用程序使用。

# 4 依赖注入 （Dependency Injection，DI）

> 依赖注入是Spring框架的核心特性之一，它是一种设计模式，用于解耦组件之间的依赖关系。在DI中，对象的依赖关系由容器负责管理和注入，而不是由对象自身负责获取依赖对象

### 依赖注入的方式

1. **构造函数注入**：通过构造函数将依赖对象作为参数传递给目标对象。在目标对象的构造函数中声明依赖对象的参数，并由Spring容器负责解析和注入依赖对象
   
   ```java
   public class MyController {
   private MyService myService;
   public MyController(MyService myService) {
   this.myService = myService;
   }
   // ...
   }
   ```

2. **Setter方法注入**：通过Setter方法设置依赖对象。在目标对象中定义Setter方法，并使用`@Autowired`注解标记需要注入的依赖对象
   
   ```java
   public class MyController {
   private MyService myService;
   @Autowired
   public void setMyService(MyService myService) {
   this.myService = myService;
   }
   // ...
   }
   ```

3. **字段注入**：通过直接注入依赖对象到目标对象的字段上。在目标对象中声明依赖对象的字段，并使用`@Autowired`注解标记需要注入的依赖对象
   
   ```java
   public class MyController {
   @Autowired
   private MyService myService;
   // ...
   }
   ```

4. **接口注入**：通过接口注入依赖对象。目标对象实现一个接口，在接口中定义需要注入的依赖对象的Setter方法，并由Spring容器负责调用Setter方法注入依赖对象。
   
   ```java
   public interface MyServiceAware {
   void setMyService(MyService myService);
   }
   public class MyController implements MyServiceAware {
   private MyService myService;
   @Override
   public void setMyService(MyService myService) {
   this.myService = myService;
   }
   // ...
   }
   ```

### 循环依赖问题

循环依赖问题是指在软件开发中，多个模块或组件之间存在相互依赖的环路关系，导致编译、构建或运行时出现问题。这种问题通常是设计或架构上的缺陷，应该尽量避免。

1. 编译错误：循环依赖可能导致编译错误，因为编译器无法确定模块之间的依赖关系和编译顺序。这会导致编译失败或无法正确生成可执行文件。
2. 运行时错误：如果循环依赖存在于运行时环境中，可能会导致程序崩溃、死锁或无法正常工作。由于循环依赖导致模块之间的相互依赖性，可能会出现无限循环或循环等待的情况。
3. 可维护性和扩展性问题：循环依赖会增加代码的复杂性，使得代码难以理解、调试和维护。同时，循环依赖也限制了系统的扩展性，因为修改一个模块可能会影响到其它所有相关的模块。
   为了解决循环依赖问题，可以考虑以下几个方法：
4. 重构代码：重新设计模块之间的依赖关系，尽量避免循环依赖。可以通过解耦模块、引入中间层或抽象接口等方式来改善设计。
5. 使用依赖注入：通过依赖注入容器来管理模块之间的依赖关系，避免显式的依赖声明。依赖注入可以帮助解耦和简化模块之间的依赖关系。
6. 引入事件驱动或消息传递：使用事件驱动或消息传递的方式来解耦模块之间的依赖关系。模块可以通过发布和订阅事件或消息来进行通信，而不直接依赖于其他模块。
7. 设计模式：使用适当的设计模式，如观察者模式、策略模式、工厂模式等，来降低模块之间的耦合度，减少循环依赖的可能性。

# Spring 注解

- **@Component**: 将 java 类标记为 bean。它是任何 Spring 管理组件的通用构造型。spring 的组件扫描机制现在可以将其拾取并将其拉入应用程序环境中,用于标识一个类为Spring容器的组件，通常与自动扫描配合使用
- @Controller - 用于 Spring MVC 项目中的控制器类
- **@Service**: 用于标识一个类为服务层组件，通常用于定义业务逻辑
- **@Repository**: 用于标识一个类为数据访问层组件，通常用于数据库操作
- **@RestController**: 结合@Controller和@ResponseBody，用于创建RESTful风格的控制器
- **@Bean**: 在配置类中使用，用于定义一个Bean对象，Spring容器会根据该注解创建Bean实例
- **@Autowired**: 用于自动装配依赖对象。可以用于字段、构造函数、Setter方法上，让Spring容器自动注入对应类型的Bean
- **@Qualifier**: 与@Autowired配合使用，用于指定具体的Bean名称进行注入，解决多个相同类型的Bean的歧义性
- 
- **@Value**: 用于注入属性值，可以注入基本类型、字符串、表达式等
- **@Configuration**: 用于标识一个类为配置类，相当于Spring的XML配置文件
- **@ComponentScan**: 用于指定要扫描的组件包，自动扫描并注册Bean到Spring容器
- **@RequestMapping**: 用于映射HTTP请求到控制器方法，指定URL路径和请求方法
- **@Transactional**: 用于标识事务的边界，确保方法在事务中执行
- **@Aspect**: 用于定义切面，结合其他注解如@Before、@After等实现切面逻辑
- @ResponseBody - 用于发送 Object 作为响应，通常用于发送 XML 或 JSON 数据作为响应
- @PathVariable - 用于将动态值从 URI 映射到处理程序方法参数
- @Required 应用于 bean 属性 setter 方法。此注解仅指示必须在配置时使用 bean 定义中的显式属性值或使用自动装配填充受影响的 bean 属性。如果尚未填充受影响的 bean 属性，则容器将抛出 BeanInitializationException。
- @Resource : 默认是根据name注入bean的，可以通过设置类型来实现通过类型来注入
- **@Scope**: 用于指定Bean的作用域，包括Singleton（单例，默认）、Prototype（原型）、Request、Session等

# 面向切面编程(Aspect-Oriented Programming，AOP)

即面向切面编程与 OOP( Object-Oriented Programming, 面向对象编程) 相辅相成, 提供了与 OOP 不同的抽象软件结构的视角.在 OOP 中, 我们以类(class)作为我们的基本单元, 而 AOP 中的基本单元是 Aspect(切面)

### AOP 实现方式

- 静态代理 - 指使用 AOP 框架提供的命令进行编译，从而在编译阶段就可生成 AOP 代理类，因此也称为编译时增强；
- 编译时编织（特殊编译器实现）
- 类加载时编织（特殊的类加载器实现）。
- 动态代理 - 在运行时在内存中“临时”生成 AOP 动态代理类，因此也被称为运行时增强。
- JDK 动态代理
- CGLIB

### Spring 代理

将 Advice 应用于目标对象后创建的对象称为代理。在客户端对象的情况下，目标对象和代理对象是相同的。

```
Advice + Target Object = Proxy 
```

### 静态代理与动态代理

**静态代理：** 代表：AspectJ
解析：就是AOP框架会在编译阶段生成AOP代理类，因此也称为编译时增强，
他会在编译阶段将AspectJ(切面)织入到Java字节码中

**动态代理：** 代表：Spring AOP
解析：就是说AOP框架不会去修改字节码，而是每次运行时在内存中临时为
方法生成一个AOP对象，这个AOP对象包含了目标对象的全部方法

生成AOP代理对象的时机不同，相对来说AspectJ性能更好，

但是AspectJ需要特定的编译器进行处理，而Spring AOP则无需特定的编译器处理

### 动态代理分类

**JDK动态代理**
**核心：** InvocationHandler接口和Proxy类
**解析：** JDK动态代理只提供接口的代理，不支持类的代理。
InvocationHandler 通过invoke()方法反射来调用目标类中的代码

**CGLIB动态代理**
**核心：** CGLIB（Code Generation Library），是一个代码生成的类库
**解析：** CGLIB是通过继承的方式做的动态代理，因此如果某个类被标记为final，
那么它是无法使用CGLIB做动态代理的

### AOP组成

**1、切面（Aspect）**

被抽取的公共模块，可能会横切多个对象。 在Spring AOP中，切面可以使用通用类（基于模式的风格） 或者在普通类中以 @AspectJ 注解来实现

**2、连接点（Join point）**

指方法，在Spring AOP中，一个连接点 总是 代表一个方法的执行

**3、通知（Advice）**

 在切面的某个特定的连接点（Join point）上执行的动作。通知有各种类型，其中包括“around”、“before”和“after”等通知。许多AOP框架，包括Spring，都是以拦截器做通知模型， 并维护一个以连接点为中心的拦截器链

**3、Advice Arguments** - 我们可以在 advice 方法中传递参数。我们可以在切入点中使用 args() 表达式来应用于与参数模式匹配的任何方法。如果我们使用它，那么我们需要在确定参数类型的 advice 方法中使用相同的名称。

**4、切入点（Pointcut）：** 切入点是指 我们要对哪些Join point进行拦截的定义。通过切入点表达式，指定拦截的方法，比如指定拦截add*、search*。

**5、引入（Introduction）：**（也被称为内部类型声明（inter-type declaration））。声明额外的方法或者某个类型的字段。Spring允许引入新的接口（以及一个对应的实现）到任何被代理的对象。例如，你可以使用一个引入来使bean实现 IsModified 接口，以便简化缓存机制。

**6、目标对象（Target Object）：** 被一个或者多个切面（aspect）所通知（advise）的对象。也有人把它叫做 被通知（adviced） 对象。 既然Spring AOP是通过运行时代理实现的，这个对象永远是一个 被代理（proxied） 对象。

**7、织入（Weaving）：** 指把增强应用到目标对象来创建新的代理对象的过程。Spring是在运行时完成织入。

### AOP实现原理

1、基于JDK的动态代理的原理

需要用到的几个关键成员 InvocationHandler （你想要通过动态代理生成的对象都必须实现这个接口） 真实的需要代理的对象（帮你代理的对象） Proxy对象（是JDK中java.lang.reflect包下的）

下面是具体如何动态利用这三个组件生成代理对象

- 首先你的真是要代理的对象必须要实现InvocationHandler 这个接口，并且覆盖这个接口的invoke(Object proxyObject, Method method, Object[] args)方法，这个Invoker中方法的参数的proxyObject就是你要代理的真实目标对象，方法调用会被转发到该类的invoke()方法， method是真实对象中调用方法的Method类，Object[] args是真实对象中调用方法的参数

- 然后通过Proxy类去调用newProxyInstance(classLoader, interfaces, handler)方法，classLoader是指真实代理对象的类加载器,interfaces是指真实代理对象需要实现的接口，还可以同时指定多个接口，handler方法调用的实际处理者（其实就是帮你代理的那个对象），代理对象的方法调用都会转发到这里，然后直接就能生成你想要的对象类了。

### Spring AOP and AspectJ AOP 有什么区别？

Spring AOP 基于动态代理方式实现；AspectJ 基于静态代理方式实现。

Spring AOP 仅支持方法级别的 PointCut；提供了完全的 AOP 支持，它还支持属性级别的 PointCut。

# 事务

Spring事务的本质其实就是数据库对事务的支持，没有数据库的事务支持，spring是无法提供事务功能的。真正的数据库层的事务提交和回滚是通过binlog或者redo log实现的。

### 事务的种类

spring支持编程式事务管理和声明式事务管理两种方式：

①编程式事务管理使用TransactionTemplate

②声明式事务管理建立在AOP之上的。其本质是通过AOP功能，对方法前后进行拦截，将事务处理的功能编织到拦截的方法中，也就是在目标方法开始之前加入一个事务，在执行完目标方法之后根据执行情况提交或者回滚事务

声明式事务最大的优点就是不需要在业务逻辑代码中掺杂事务管理的代码，只需在配置文件中做相关的事务规则声明或通过@Transactional注解的方式，便可以将事务规则应用到业务逻辑中。

声明式事务管理要优于编程式事务管理，这正是spring倡导的非侵入式的开发方式，使业务代码不受污染，只要加上注解就可以获得完全的事务支持。唯一不足地方是，最细粒度只能作用到方法级别，无法做到像编程式事务那样可以作用到代码块级别。

### 事务传播行为

事务的传播行为说的是，当多个事务同时存在的时候，spring如何处理这些事务的行为

① PROPAGATION_REQUIRED：如果当前没有事务，就创建一个新事务，如果当前存在事务，就加入该事务，该设置是最常用的设置。

② PROPAGATION_SUPPORTS：支持当前事务，如果当前存在事务，就加入该事务，如果当前不存在事务，就以非事务执行。‘

③ PROPAGATION_MANDATORY：支持当前事务，如果当前存在事务，就加入该事务，如果当前不存在事务，就抛出异常。

④ PROPAGATION_REQUIRES_NEW：创建新事务，无论当前存不存在事务，都创建新事务。

⑤ PROPAGATION_NOT_SUPPORTED：以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。

⑥ PROPAGATION_NEVER：以非事务方式执行，如果当前存在事务，则抛出异常。

⑦ PROPAGATION_NESTED：如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则按REQUIRED属性执行。

### 事务隔离级别

① ISOLATION_DEFAULT：这是个 PlatfromTransactionManager 默认的隔离级别，使用数据库默认的事务隔离级别

② ISOLATION_READ_UNCOMMITTED：读未提交，允许另外一个事务可以看到这个事务未提交的数据

③ ISOLATION_READ_COMMITTED：读已提交，保证一个事务修改的数据提交后才能被另一事务读取，而且能看到该事务对已有记录的更新

④ ISOLATION_REPEATABLE_READ：可重复读，保证一个事务修改的数据提交后才能被另一事务读取，但是不能看到该事务对已有记录的更新

⑤ ISOLATION_SERIALIZABLE：一个事务在执行的过程中完全看不到其他事务对数据库所做的更新

### 管理事务

**1、PlatformTransactionManager** 事务管理器，主要用于平台相关事务的管理。主要包括三个方法：①、commit：事务提交。②、rollback：事务回滚。③、getTransaction：获取事务状态。

**2、TransacitonDefinition** 事务定义信息，用来定义事务相关属性，给事务管理器PlatformTransactionManager使用这个接口有下面四个主要方法：①、getIsolationLevel：获取隔离级别。②、getPropagationBehavior：获取传播行为。③、getTimeout获取超时时间。④、isReadOnly：是否只读（保存、更新、删除时属性变为false--可读写，查询时为true--只读）事务管理器能够根据这个返回值进行优化，这些事务的配置信息，都可以通过配置文件进行配置。

**3、TransationStatus** 事务具体运行状态，事务管理过程中，每个时间点事务的状态信息。例如：①、hasSavepoint()：返回这个事务内部是否包含一个保存点。②、isCompleted()：返回该事务是否已完成，也就是说，是否已经提交或回滚。③、isNewTransaction()：判断当前事务是否是一个新事务。

### Spring框架的事务管理优点

它为不同的事务API 如 JTA，JDBC，Hibernate，JPA 和JDO，提供一个不变的编程模式。 它为编程式事务管理提供了一套简单的API而不是一些复杂的事务API如它为编程式事务管理提供了一套简单的API而不是一些复杂的事务API如 它支持声明式事务管理。它支持声明式事务管理。 它和Spring各种数据访问抽象层很好得集成。它和Spring各种数据访问抽象层很好得集成

### Spring配置事务

**1、基于XML的事务配置。**

```java
<?xml version="1.0" encoding="UTF-8"?>
<!-- from the file 'context.xml' -->  
<beans xmlns="http://www.springframework.org/schema/beans"  
     xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
     xmlns:aop="http://www.springframework.org/schema/aop"  
     xmlns:tx="http://www.springframework.org/schema/tx"  
     xsi:schemaLocation="  
     http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-2.5.xsd  
     http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-2.5.xsd  
     http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-2.5.xsd">  

  <!-- 数据元信息 -->  
  <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">  
      <property name="driverClassName" value="oracle.jdbc.driver.OracleDriver"/>  
      <property name="url" value="jdbc:oracle:thin:@rj-t42:1521:elvis"/>  
      <property name="username" value="root"/>  
      <property name="password" value="root"/>  
  </bean>  

  <!-- 管理事务的类,指定我们用谁来管理我们的事务-->  
  <bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">  
      <property name="dataSource" ref="dataSource"/>  
  </bean>   

  <!-- 首先我们要把服务对象声明成一个bean  例如HelloService -->  
  <bean id="helloService" class="com.yintong.service.HelloService"/>  

  <!-- 然后是声明一个事物建议tx:advice,spring为我们提供了事物的封装，这个就是封装在了<tx:advice/>中 -->
  <!-- <tx:advice/>有一个transaction-manager属性，我们可以用它来指定我们的事物由谁来管理。
      默认：事务传播设置是 REQUIRED，隔离级别是DEFAULT -->
  <tx:advice id="txAdvice" transaction-manager="txManager">  
      <!-- 配置这个事务建议的属性 -->  
      <tx:attributes>  
        <!-- 指定所有get开头的方法执行在只读事务上下文中 -->  
        <tx:method name="get*" read-only="true"/>  
        <!-- 其余方法执行在默认的读写上下文中 -->  
        <tx:method name="*"/>  
      </tx:attributes>  
  </tx:advice>  

  <!-- 我们定义一个切面，它匹配FooService接口定义的所有操作 -->  
  <aop:config>  
     <!-- <aop:pointcut/>元素定义AspectJ的切面表示法，这里是表示com.yintong.service.helloService包下的任意方法。 -->
      <aop:pointcut id="helloServiceOperation" expression="execution(* com.yintong.service.helloService.*(..))"/>  
     <!-- 然后我们用一个通知器：<aop:advisor/>把这个切面和tx:advice绑定在一起，表示当这个切面：fooServiceOperation执行时tx:advice定义的通知逻辑将被执行 -->
       <aop:advisor advice-ref="txAdvice" pointcut-ref="helloServiceOperation"/>  
  </aop:config>  
</beans>  
```

**2、基于注解方式的事务配置。**

@Transactional：直接在Java源代码中声明事务的做法让事务声明和将受其影响的代码距离更近了，而且一般来说不会有不恰当的耦合的风险，因为，使用事务性的代码几乎总是被部署在事务环境中。

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"  
     xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
     xmlns:aop="http://www.springframework.org/schema/aop"  
     xmlns:tx="http://www.springframework.org/schema/tx"  
     xsi:schemaLocation="  
     http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-2.5.xsd  
     http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-2.5.xsd  
     http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-2.5.xsd">  

  <bean id="helloService" class="com.yintong.service.HelloService"/>  
  <bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">  
     <property name="dataSource" ref="dataSource"/>  
  </bean>
  <!-- 配置注解事务 -->  
  <tx:annotation-driven transaction-manager="txManager"/>  
</beans>
```

主要在类中定义事务注解@Transactional，如下：

```java
//@Transactional 注解可以声明在类上，也可以声明在方法上。在大多数情况下，方法上的事务会首先执行
@Transactional(readOnly = true)    
public class HelloService{  
    public Foo getFoo(String fooName) {  
    }  
    //@Transactional 注解的事务设置将优先于类级别注解的事务设置   propagation:可选的传播性设置
    @Transactional(readOnly = false, propagation = Propagation.REQUIRES_NEW)      
    public void updateFoo(Hel hel) {
    }
}  
```

# 通知

### 通知类型

**1、前置通知（Before advice）** 

在某连接点（join point）之前执行的通知，但这个通知不能阻止连接点前的执行（除非它抛出一个异常）

**2、返回后通知（After returning advice）** 

在某连接点（join point）正常完成后执行的通知：例如，一个方法没有抛出任何异常，正常返回

**3、抛出异常后通知（After throwing advice）** 

在方法抛出异常退出时执行的通知。

**4、后通知（After (finally) advice）** 

当某连接点退出的时候执行的通知（不论是正常返回还是异常退出）

**5、环绕通知（Around Advice）** 

包围一个连接点（join point）的通知，如方法调用。这是最强大的一种通知类型。 环绕通知可以在方法调用前后完成自定义的行为。它也会选择是否继续执行连接点或直接返回它们自己的返回值或抛出异常来结束执行。 环绕通知是最常用的一种通知类型。大部分基于拦截的AOP框架，例如Nanning和JBoss4，都只提供环绕通知

### spring aop 中 concern 和 cross-cutting concern 的不同之处

concern 是我们想要在应用程序的特定模块中定义的行为。它可以定义为我们想要实现的功能。

cross-cutting concern 是一个适用于整个应用的行为，这会影响整个应用程序。例如，日志记录，安全性和数据传输是应用程序几乎每个模块都需要关注的问题，因此它们是跨领域的问题。

## 事件

### 事件种类

**1、上下文更新事件（ContextRefreshedEvent）：** 在调用ConfigurableApplicationContext 接口中的refresh()方法时被触发。

**2、上下文开始事件（ContextStartedEvent）：** 当容器调用ConfigurableApplicationContext的Start()方法开始/重新开始容器时触发该事件。

**3、上下文停止事件（ContextStoppedEvent）：** 当容器调用ConfigurableApplicationContext的Stop()方法停止容器时触发该事件。

**4、上下文关闭事件（ContextClosedEvent）：** 当ApplicationContext被关闭时触发该事件。容器被关闭时，其管理的所有单例Bean都被销毁。

**5、请求处理事件（RequestHandledEvent）：** 在Web应用中，当一个http请求（request）结束触发该事件。

如果一个bean实现了ApplicationListener接口，当一个ApplicationEvent 被发布以后，bean会自动被通知。

而Servlet的filter是基于函数回调实现的过滤器，Filter主要是针对URL地址做一个编码的事情、过滤掉没用的参数、安全校验（比较泛的，比如登录不登录之类）

## 其他问题

### @Autowire和@Resource区别

| 对比项  | @Autowire       | @Resource                 |
| ---- | --------------- | ------------------------- |
| 注解来源 | Spring注解        | JDK注解(JSR-250标准注解，属于J2EE) |
| 装配方式 | 优先按类型           | 优先按名称                     |
| 属性   | required        | name、type                 |
| 作用范围 | 字段、setter方法、构造器 | 字段、setter方法               |

### 单例 Bean 的线程安全

        在Spring框架中，默认情况下，单例作用域（Singleton）的Bean是线程安全的。这是因为Spring容器在创建单例Bean时，会使用同步机制来保证在多线程环境下的安全性。
当多个线程同时请求获取同一个单例Bean时，Spring容器会确保只有一个线程能够执行Bean的初始化过程，而其他线程会等待初始化完成后直接获取已初始化的Bean实例。这样可以避免多个线程同时执行初始化逻辑，从而保证了单例Bean的线程安全性。
        需要注意的是，虽然单例Bean本身是线程安全的，但如果单例Bean内部包含了可变状态（例如实例变量），并且多个线程同时对该状态进行修改，那么仍然需要采取适当的线程安全措施，例如使用同步关键字或使用线程安全的数据结构来管理状态。此外，如果在 Spring 中使用原型作用域（Prototype）的Bean，那么每次获取该Bean时都会创建一个新的实例，因此需要自行确保在多线程环境下的线程安全性。总结起来，Spring框架的单例Bean默认是线程安全的，但如果单例Bean内部包含可变状态，仍需注意线程安全性，并采取相应的措施来保证线程安全。

### 线程并发问题

在一般情况下，只有无状态的Bean才可以在多线程环境下共享，在Spring中，绝大部分Bean都可以声明为singleton作用域，因为Spring对一些Bean中非线程安全状态采用ThreadLocal进行处理，解决线程安全问题。

ThreadLocal和线程同步机制都是为了解决多线程中相同变量的访问冲突问题。同步机制采用了“时间换空间”的方式，仅提供一份变量，不同的线程在访问前需要获取锁，没获得锁的线程则需要排队。而ThreadLocal采用了“空间换时间”的方式。

ThreadLocal会为每一个线程提供一个独立的变量副本，从而隔离了多个线程对数据的访问冲突。因为每一个线程都拥有自己的变量副本，从而也就没有必要对该变量进行同步了。ThreadLocal提供了线程安全的共享对象，在编写多线程代码时，可以把不安全的变量封装进ThreadLocal。

### 在 Spring中如何注入一个java集合？

Spring提供以下几种集合的配置元素：

| 元素        | 说明                           |
| --------- | ---------------------------- |
| **list**  | 类型用于注入一列值，允许有相同的值。           |
| **set**   | 类型用于注入一组值，不允许有相同的值。          |
| **map**   | 类型用于注入一组键值对，键和值都可以为任意类型。     |
| **props** | 类型用于注入一组键值对，键和值都只能为String类型。 |

```xml
<!-- 配置 student对象 -->
<bean class="com.dpb.javabean.Student">
    <property name="id" value="10"/>
    <property name="name" value="波波烤鸭"/>
    <!-- 对象注入 -->
    <property name="cat" ref="catId"></property>
    <!-- List集合注入 -->
    <property name="games">
        <list>
            <value>LOL</value>
            <value>DNF</value>
            <value>CS</value>
        </list>
    </property>
<property name="score">
    <map>
        <entry key="数学" value="99"/>
        <entry key="英语" value="78"/>
        <entry key="化学" value="84"/>
    </map>
</property>
<property name="props">
    <props>
        <prop key="userName">admin</prop>
        <prop key="password">123</prop>
    </props>
</property>
```