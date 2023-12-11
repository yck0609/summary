# 1 SpringMVC

> Spring 配备构建Web 应用的全功能MVC框架。Spring可以很便捷地和其他MVC框架集成，如Struts，Spring 的MVC框架用控制反转把业务对象和控制逻辑清晰地隔离。它也允许以声明的方式把请求参数和业务对象绑定
> 
> Spring Web MVC 框架提供 模型-视图-控制器 架构和随时可用的组件，用于开发灵活且松散耦合的 Web 应用程序。MVC 模式有助于分离应用程序的不同方面，如输入逻辑，业务逻辑和 UI 逻辑，同时在所有这些元素之间提供松散耦合

### 什么是Spring MVC框架的控制器？

控制器提供一个访问应用程序的行为，此行为通常通过服务接口实现。控制器解析用户输入并将其转换为一个由视图呈现给用户的模型。Spring用一个非常抽象的方式实现了一个控制层，允许用户创建多种用途的控制器

## DispatcherServlet初始化过程

## DispatcherServlet 工作流程

**1、** 向服务器发送 HTTP 请求，请求被前端控制器 DispatcherServlet 捕获。

**2、** DispatcherServlet 根据 -servlet.xml 中的配置对请求的 URL 进行解析，得到请求资源标识符（URI）。然后根据该 URI，调用 HandlerMapping 获得该 Handler 配置的所有相关的对象（包括 Handler 对象以及 Handler 对象对应的拦截器），最后以HandlerExecutionChain 对象的形式返回。

**3、** DispatcherServlet 根据获得的Handler，选择一个合适的 HandlerAdapter。（附注：如果成功获得HandlerAdapter后，此时将开始执行拦截器的 preHandler(…)方法）。

**4、** 提取Request中的模型数据，填充Handler入参，开始执行Handler（Controller)。在填充Handler的入参过程中，根据你的配置，Spring 将帮你做一些额外的工作：

- HttpMessageConveter：将请求消息（如 Json、xml 等数据）转换成一个对象，将对象转换为指定的响应信息。
- 数据转换：对请求消息进行数据转换。如`String`转换成`Integer`、`Double`等。
- 数据根式化：对请求消息进行数据格式化。如将字符串转换成格式化数字或格式化日期等。
- 数据验证：验证数据的有效性（长度、格式等），验证结果存储到`BindingResult`或`Error`中。

**5、** Handler(Controller)执行完成后，向 DispatcherServlet 返回一个 ModelAndView 对象；

**6、** 根据返回的ModelAndView，选择一个适合的 ViewResolver（必须是已经注册到 Spring 容器中的ViewResolver)返回给DispatcherServlet。

**7、** ViewResolver 结合Model和View，来渲染视图。

**8、** 视图负责将渲染结果返回给客户端。

### WebApplicationContext

WebApplicationContext 是 ApplicationContext 的扩展。它具有 Web 应用程序所需的一些额外功能。它与普通的 ApplicationContext 在解析主题和决定与哪个 servlet 关联的能力方面有所不同。

### Spring DAO 有什么用？

Spring DAO 使得 JDBC，Hibernate 或 JDO 这样的数据访问技术更容易以一种统一的方式工作。这使得用户容易在持久性技术之间切换。它还允许您在编写代码时，无需考虑捕获每种技术不同的异常。

### Spring MVC执行流程和原理  SpringMVC流程：

![](https://gitee.com/gsjqwyl/images_repo/raw/master/2021-3-11/20210328185541.png)

**1、** 用户发送出请求到前端控制器DispatcherServlet。

**2、** DispatcherServlet收到请求调用HandlerMapping（处理器映射器）。

**3、** HandlerMapping找到具体的处理器(可查找xml配置或注解配置)，生成处理器对象及处理器拦截器(如果有)，再一起返回给DispatcherServlet。

**4、** DispatcherServlet调用HandlerAdapter（处理器适配器）。

**5、** HandlerAdapter经过适配调用具体的处理器（Handler/Controller）。

**6、** Controller执行完成返回ModelAndView对象。

**7、 **HandlerAdapter将Controller执行结果ModelAndView返回给DispatcherServlet。

**8、** DispatcherServlet将ModelAndView传给ViewReslover（视图解析器）。

**9、** ViewReslover解析后返回具体View（视图）。

**10、** DispatcherServlet根据View进行渲染视图（即将模型数据填充至视图中）。

**11、** DispatcherServlet响应用户。

### spring常用注解

**@Controller注解**

是在Spring的org.springframework.stereotype包下，org.springframework.stereotype.Controller注解类型用于指示Spring类的实例是一个控制器。使用@Controller注解的类不需要继承特定的父类或者实现特定的接口，相对之前的版本实现Controller接口变的更加简单。而Controller接口的实现类只能处理一个单一的请求动作，而@Controller注解注解的控制器可以同时支持处理多个请求动作，使程序开发变的更加灵活。 @Controller用户标记一个类，使用它标记的类就是一个Spring MVC Controller对象，即：一个控制器类。Spring使用扫描机制查找应用程序中所有基于注解的控制器类，分发处理器会扫描使用了该注解的方法，并检测该方法是否使用了@RequestMapping注解，而使用@RequestMapping注解的方法才是真正处理请求的处理器。为了保证Spring能找到控制器，我们需要完成两件事：

**@RequestParam注解**
下面来说org.springframework.web.bind.annotation包下的第三个注解，即：@RequestParam注解，该注解类型用于将指定的请求参数赋值给方法中的形参。那么@RequestParam注解有什么属性呢？它有4种属性，下面将逐一介绍这四种属性：

**1、**name属性该属性的类型是String类型，它可以指定请求头绑定的名称；

**2、**value属性该属性的类型是String类型，它可以设置是name属性的别名；

**3、**required属性该属性的类型是boolean类型，它可以设置指定参数是否必须绑定；

**4、**defalutValue属性该属性的类型是String类型，它可以设置如果没有传递参数可以使用默认值。

**@PathVaribale注解**

下面来说org.springframework.web.bind.annotation包下的第四个注解，即：@PathVaribale注解，该注解类型可以非常方便的获得请求url中的动态参数。@PathVaribale注解只支持一个属性value，类型String，表示绑定的名称，如果省略则默认绑定同名参数。

### SpringMVC与Spring的父子容器关系

### Springmvc controller方法中为什么不能定义局部变量？

因为controller是默认单例模式，高并发下全局变量会出现线程安全问题

现这种问题如何解决呢？

**1、** 既然是全局变量惹的祸，那就将全局变量都编程局部变量，通过方法参数来传递。

**2、** jdk提供了java.lang.ThreadLocal,它为多线程并发提供了新思路。

**3、** 使用@Scope("session")，会话级别

```java
    @Controller  
    //把这个bean 的范围设置成session，表示这bean是会话级别的，  
    @Scope("session")  
    public class XxxController{  
        private List<String> list ;  

      //@PostConstruct当bean加载完之后，就会执行init方法，并且将list实例化；  
        @PostConstruct  
        public void init(){  
            list = new ArrayList<String>();  
        }  
    }
```

**4、** 将控制器的作用域从单例改为原型，即在spring配置文件Controller中声明 scope="prototype"，每次都创建新的controller