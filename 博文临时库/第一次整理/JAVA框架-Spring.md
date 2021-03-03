首先想要指明的是，Spring并不是JavaEE的全部，也更不是Java的全部。鄙人对于把学会Spring看成高手或者对于不会Spring而嗤之以鼻的行为都很是不满意。

Spring和javaEE，Spring和Servlet和Tomcat，Spring和SpringBoot和SpringCloud和SpringMVC等众多组件的关系，是为困惑一点。有时候需要了解到了一个图景才能进一步的有信息的学习，所以这个全局图景甚为重要。

---

为什么使用Spring

- IOC和AOP
- 其他核心功能如：事务管理、包装了JDBC异常等
- 方便集成其他框架
- 降低JavaEE API的使用难度

尽管如此，还是有它的缺点：

- Spring依赖反射，反射影响性能

# JavaEE & Spring

JavaEE是一组建立在JavaSE之上的**标准**，目的在于解决一系列问题，JavaEE不仅仅是指一种标准平台，它更多的表达着一种软件架构和设计思想。

JavaEE有着众多的标准（如下列表），这些标准也体现了面向接口编程的设计思想，日常经常使用的就是这样的接口。

- Servlet：定义了如何处理Web请求
- Java Server Faces：定义了如何编写Web界面
- JAX-RS：定义了如何编写RESTFul的接口
- EJB：定义了如何编写“企业Bean”
- JPA：定义了如何编写ORM和数据存取
- JTA：定义了如何编写事务相关的代码
- JMS：定义了如何编写消息队列程序
- CDI：定义了如何编写依赖注入
- JAX：定义了如何编写XML程序
- JAX-WS: 定义了如何编写基于XML的网络服务，即SOAP

Spring在早期是为了解决一系列JavaEE的不便于使用的问题，提供更多的便利。现在Spring有着众多的组件，这些组件大量使用或实现了JavaEE标准。JavaEE和Spring并不对立。

# Tomcat & Servlet & Spring

Tomcat是一个web服务器和Servlet容器的集合体，对于Tomcat的讲解在下面小节中，首先来看一下Servlet。

Servlet是一个Java接口，如上所说的JavaEE的一个接口。Servlet不负责处理网络协议，http等网络相关事项；也并不擅长往浏览器输出HTML页面。

- Servlet并不擅长往浏览器输出HTML页面，所以出现了JSP。
- 实现Servlet的类并不是就能处理请求，Servlet并不会监听端口，不会和远端交流。请求是通过Servlet容器到达的Servlet实现，这样的容器形如Tomcat、Jetty、Jboss等。Web服务器，
  - servlet理论上可以响应任何类型请求，而不只是HTTP请求，但大多数Servlet只用于HTTP协议的响应请求（HttpServlet类）。但是Servlet也可以来处理自己的应用层协议，自我实现，用以其他用处。（形如jetty）
- 任何Spring Web的entry point，都是servlet


Servlet更加的偏向于业务逻辑（尽管很讨厌业务这个词），或者说处理请求逻辑。Servlet（或者说实现Servlet的类）负责干的事情是：

- 实现类初始化逻辑
- 实现类销毁逻辑
- 实现类接收到网络请求时逻辑

**一个简单的流程：**

当客户端第一次请求Servlet的时候，tomcat会根据web.xml配置文件实例化servlet。当又有一个客户端访问该servlet的时候，不会再实例化该servlet，也就是多个线程在使用这个实例。

- 多线程问题在下面再说

HTTP请求到了Tomcat后，Tomcat通过字符串解析，把各个请求头（Header），请求地址（URL），请求参数（QueryString）都封装进了Request对象中。然后把这个对象和一个空的Response传到对应的Servlet。

Servlet可以通过Request的一系列方法来获得Http初始参数即取得客户端传过来的数据，而传入的Response是一个几乎没有数据的对象，servlet对数据进行处理后将结果写入response对象，然后Tomcat将处理结果response对象返回客户端。

- HttpServlet是常用的类，具有处理get、post等的方法；它继承自GenericServlet，而GenericServlet实现了Servlet接口。
- 开发servlet有三种方法，一种是实现servlet接口，一种是继承GernericServlet 还有一种是继承HttpServlet。其实是对应不同场景的方法。

## Tomcat

Tomcat 的本质其实就是一个 WEB 服务器 + 一个 Servlet 容器，那么它必然需要处理**网络连接的管理与 Servlet 的管理**，因此，Tomcat 设计了两个核心组件来实现这两个功能，分别是**连接器**和**容器**，连接器用来处理外部网络连接，容器用来处理内部 Servlet

Tomcat没有完全实现JavaEE规范。13种核心技术，Tomcat只实现了两个：Servlet和JSP。而其他服务器比如JBoss、Weblogic等的都是完全支持的。所以人们往往更愿意叫Tomcat为<u>*轻量级的服务器*</u>，也有叫它Servlet/JSP容器的

- Tomcat服务器 = Web服务器 + Servlet/JSP容器（Web容器）

每当有一个 http 请求到来，web 容器会从线程池中取出一个空闲的线程，然后在线程中调用对应的 servlet 的 service 方法。servlet 是单例的并且 servlet 是由 web 容器来实例化的。即“Web 容器会为每个 http 请求启动一个线程去调用相应的 servlet”

- <u>*todo：connector中有专门的线程池来管理？*</u>

定位 Servlet 的流程图：

<img src="E:\_data\博文临时库\博文中的图片\Spring-tomcat-locateServlet.png" style="zoom:50%;" />

- Engine：表示一个虚拟主机的引擎，一个 Tomcat Server 只有一个 引擎，连接器所有的请求都交给引擎处理，而引擎则会交给相应的虚拟主机去处理请求；
- Host：表示虚拟主机**，**一个容器可以有多个虚拟主机，每个主机都有对应的域名，在 Tomcat 中，一个 webapps 就代表一个虚拟主机，当然webapps 可以配置多个；
- Context：表示一个应用容器，一个虚拟主机可以拥有多个应用，webapps 中每个目录都代表一个 Context，每个应用可以配置多个 Servlet。

<u>*todo：tomcat的各种配置；maxThreads 连接数限制、安全配置、堆内存大小；目录结构下的文件、针对各个组件的例如connector、engine等的配置；*</u>



## Dispatcher Servlet

(tomcat处理请求的过程、和spring boot怎么交互)

Spring Web应用程序将在其配置中定义了一个Spring Dispatcher Servlet，apache tomcat容器将初始化此Servlet，调度程序Servlet进而初始化应用程序上下文。Tomcat容器和Spring IOC容器之间没有直接交互

DispatcherServlet的工作是获取传入的URI，并找到处理程序（通常是*Controller*类上的方法）和视图（通常是JSP）的正确组合（找到处理方法和视图的组合），它如何完成此操作因配置和Spring版本而异。也没有理由最终结果必须是网页。

在Spring MVC中，所有传入请求都通过这个servlet。DispatcherServlet可以引用不同的映射器，

Dispatcher Servlet 流程：

- 所有传入请求都通过单个servlet。调用**映射器**来找到正确组合

- 确定目标控制器后，`DispatcherServlet`向其发送请求。控制器根据请求执行一些工作（或将其委派给其他对象），然后返回到`DispatcherServlet`Model和View名称。

- 视图的名称只是一个逻辑名称。然后使用此逻辑名搜索实际的View（以避免与控制器和特定View耦合）。然后`DispatcherServlet`引用**ViewResolver**并将View的逻辑名称映射到View的特定实现。

- 当`DispatcherServlet`确定要显示结果的视图时，它将作为响应呈现。

  最后，`DispatcherServlet`将`Response`对象返回给客户端。

<img src="E:\_data\博文临时库\博文中的图片\Spring-DispatcherServlet.png" style="zoom:50%;" />

（来源：spring in action 4）

参考：

- [What is Dispatcher Servlet in Spring?](https://stackoverflow.com/questions/2769467/what-is-dispatcher-servlet-in-spring)



## Servlet线程安全

之前我们说：当客户端第一次请求Servlet的时候,tomcat会根据web.xml配置文件实例化servlet。当又有一个客户端访问该servlet的时候，不会再实例化该servlet，也就是多个线程在使用这个实例。那么在多个线程中使用不是就会出现线程安全问题么？事实上确实如此。

- Servlet容器默认是采用**`单实例多线程(这是造成线程安全的主因)方式处理多个请求的`**，这种默认以多线程方式执行的设计可大大降低对系统的资源需求，提高系统的并发量及响应时间。

Servlet不是线程安全的。每一个Servlet对象在Tomcat容器中只有一个实例对象，即单例模式。但如果多个HTTP请求请求的是同一个Servlet，那么这两个HTTP请求对应的线程将并发调用Servlet的service()方法。

- SingleThreadModel 接口：

  web 容器能为每个请求创建一个Servlet的实例吗？当然是可以的，只要Servlet实现SingleThreadModel接口，该接口为每次请求创建一个servlet实例。singlethreadmodel并不能解决所有的线程安全问题。例如，即使用实现SingleThreadModel 接口的 servlet，会话属性和静态变量仍然可以同时通过多线程的多个请求访问。

  而且这个接口也有很多问题：

  - 每个Servlet 创建多个对象实例
  - 如果并发高，每个servlet同时只能支持20线程的并发访问。挂起超过20个的线程。

- 属性的线程安全：ServletContext、HttpSession是线程安全的；ServletRequest是非线程安全的。

实现线程安全的方法：

- 避免使用实例变量

- 避免使用非线程安全的集合

- 加锁：

  在多个Servlet中对某个外部对象(例如文件)的修改是务必加锁（Synchronized，或者ReentrantLock），即互斥访问。

  使用synchronized 关键字，但这也会大大地影响系统的性能。

- 无状态设计

  Servlet本身是无状态的，**`一个无状态的Servlet是绝对线程安全的，无状态对象设计也是解决线程安全问题的一种有效手段`**。servlet是否线程安全是由它的实现来决定的，如果它内部的属性或方法会被多个线程改变，它就是线程不安全的，反之，就是线程安全的。

# Spring & 它的组件

Spring最重要的是他的IOC和AOP

Spring有很多不同的组成部分，划分成了很多组件，如：Spring Boot、Spring Cloud、Spring MVC

## Spring Boot、Spring Cloud、Spring MVC

- Spring Boot：build anything。

  Anything包含很多，其中就包含Spring Cloud和Spring Cloud Data Flow。

  - 可用于快速设置应用程序，提供开箱即用的配置以构建Spring支持的应用程序。Spring在其下集成了各种不同的模块，例如*spring-core*，*spring-data*， *spring-web*（包括Spring MVC）等等。

    使用此工具，您可以告诉Spring要使用的数量，并且可以快速设置它们（以后可以自行更改）

  即Spring Boot是基于Spring的可用于*生产的*项目初始化程序，只是一个自动配置工具（打包为框架）。

- Spring Cloud：Coordinate Anything

  Built directly on Spring Boot's innovative approach to enterprise Java

- Spring MVC：一个完整的面向HTTP的MVC框架，由Spring框架管理并基于Servlet

  它相当于JavaEE堆栈中的JSF。其中最流行的元素是带有注释的类`@Controller`，在其中实现可以使用不同的HTTP请求访问的方法。它等效`@RestController`于实现基于REST的API

  即Spring MVC是要在Web应用程序中使用的框架

- Spring boot是一个实用程序，可用于快速设置应用程序，提供开箱即用的配置以构建Spring支持的应用程序。如您所知，Spring在其保护伞下集成了各种不同的模块，例如*spring-core*，*spring-data*， *spring-web*（顺便说一下，包括Spring MVC）等等。使用此工具，您可以告诉Spring要使用的数量，并且可以快速设置它们（以后您可以自行更改）。

*Spring Boot*和*Spring MVC*不具有可比性或互斥。如果要使用Spring进行Web应用程序开发，则无论如何都要使用*Spring MVC*。然后，您的问题就变成是否使用*Spring Boot*。对于开发常见的Spring应用程序或开始学习Spring，我建议使用*Spring Boot*。它极大地简化了工作，并被广泛采用。

- 创建一个web'项目则：创建一个Spring Boot项目并在其中选择*Web*作为模块。在这种情况下：Spring boot = Spring MVC + Auto Configuration(Don't need to write spring.xml file for configurations) + Server(You can have embedded Tomcat, Netty, Jetty server).

---

Spring Framework扩展点及其应用：

日常开发中几乎用不到这些『屠龙技』，但是当阅读Java中间件源码的时候经常会见到它们

<img src="E:\_data\博文临时库\博文中的图片\Spring中的扩展点.jpg" style="zoom:25%;" />

## 一些小问题

<u>*todo：这些内容没有合适的连接起来，也就是上下文不连贯，暂时这么写的*</u>

**设计模式：**

- 工厂模式：BeanFactory就是简单工厂模式的体现，用来创建对象的实例；
- 单例模式：Bean默认为单例模式
- 代理模式：Spring的AOP功能用到了JDK的动态代理和CGLIB字节码生成技术；
-  ？模板方法：用来解决代码重复的问题。比如：RestTemplate, JmsTemplate, JpaTemplate 
-  ？观察者模式：定义对象键一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都会得到通知被制动更新，如Spring中listener的实现–ApplicationListener

---

？Spring 提供了以下5种标准的事件：

1. 上下文更新事件（ContextRefreshedEvent）：在调用ConfigurableApplicationContext 接口中的refresh()方法时被触发。
2. 上下文开始事件（ContextStartedEvent）：当容器调用ConfigurableApplicationContext的Start()方法开始/重新开始容器时触发该事件。
3. 上下文停止事件（ContextStoppedEvent）：当容器调用ConfigurableApplicationContext的Stop()方法停止容器时触发该事件。
4. 上下文关闭事件（ContextClosedEvent）：当ApplicationContext被关闭时触发该事件。容器被关闭时，其管理的所有单例Bean都被销毁。
5. 请求处理事件（RequestHandledEvent）：在Web应用中，当一个http请求（request）结束触发该事件。如果一个bean实现了ApplicationListener接口，当一个ApplicationEvent 被发布以后，bean会自动被通知。

---

一个Controller调用两个Service，这两Service又都分别调用两个Dao，问其中用到了几个数据库连接池的连接？

- 要看有几个datasource数据源，如果是一个数据源那么感觉是2个，因为在dao里面，getConection的时候都是从连接池里拿到不同的连接
- 也要看有没有事务，如果这两个service，每个Service调用的两个dao都是在同一个线程同一个数据源并且开启了事务管理，则每个service都会使用一个Connection。
- 如果不在Spring事务管理下，则无论在不在同一个数据源在不在同一个线程，每次调用Dao执行sql，都会使用DataSourceUtils.doGetConnection获取一个连接，执行完之后释放。
- 该题目应该是用到2-4个数据库连接。

---

其他：

- Spring处理的线程安全是threadlocal或者无状态POJO类可以自己就是安全的

- spring在自定义注解上做了哪些优化

  - hahahah，这我就不知道了

- Spring的线程池

  ThreadPoolTaskExecutor线程是Spring的线程池，其底层是依据JDK线程池ThreadPoolExecutor来实现的

- spring的监听器、过滤器、拦截器、先后顺序

  （Java就爱搞这些名字听起来高大上，实际功能干不了什么的玩意）

   启动顺序：`监听器 >  过滤器 > 拦截器`

  <img src="E:\_data\博文临时库\博文中的图片\Spring的过滤器拦截器监听器.jpg" style="zoom: 50%;" />

  - Filter和Listener：依赖Servlet容器，基于函数回调实现。可以拦截所有请求，覆盖范围更广，但无法获取ioc容器中的bean
  - Interceptor：依赖spring框架，基于java反射和动态代理实现。只能拦截controller的请求，可以获取ioc容器中的bean。这是AOP的一种应用，不依赖于servlet容器
  - 拦截器只能对action请求起作用，而过滤器则可以对几乎所有的请求起作用
  - 拦截器可以访问action上下文、值栈里的对象，而过滤器不能
  - 在action的生命周期中，拦截器可以多次被调用，而过滤器只能在容器初始化时被调用一次

  使用原则：

  - 对整个流程清楚之后，然后就是各自的使用，在使用之前应该有一个使用规则，为什么这个说，因为有些功能比如判断用户是否登录，既可以用过滤器，也可以用拦截器，用哪一个才是合理的呢？那么如果有一个原则，使用起来就会更加合理。实际上这个原则是有的：

    把整个项目的流程比作一条河，那么监听器的作用就是能够听到河流里的所有声音，过滤器就是能够过滤出其中的鱼，而拦截器则是拦截其中的部分鱼，并且作标记。**所以当需要监听到项目中的一些信息，并且不需要对流程做更改时，用监听器；当需要过滤掉其中的部分信息，只留一部分时，就用过滤器；当需要对其流程进行更改，做相关的记录时用拦截器。下面是具体的使用案例**

  实现：

  - 过滤器实现：

    - 无路径无顺序@Component
    - 有路径无顺序@WebFilter+@ServletComponentScan
    - 有路径有顺序@Configuration

  - 监听器实现：
    - 同过滤器？

  - 拦截器实现：
    - 两个步骤：声明拦截器的类：通过实现HandlerInterceptor接口，实现preHandle、postHandle和afterCompletion方法。 通过配置类配置拦截器：通过实现WebMvcConfigurer接口，实现addInterceptors方法。

  https://zhuanlan.zhihu.com/p/69060111