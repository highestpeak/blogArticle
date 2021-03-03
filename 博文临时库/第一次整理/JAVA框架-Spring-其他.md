# Spring 事务

Spring DAO 使得 JDBC，Hibernate 或 JDO 这样的数据访问技术更容易以一种统一的方式工作。这使得用户容易在持久性技术之间切换。它还允许您在编写代码时，无需考虑捕获每种技术不同的异常。

- Spring JDBC资源管理和错误处理的代价都会被减轻，使用提供的模板`JdbcTemplate`

- Spring支持两种类型的事务管理

  Spring事务的本质其实就是数据库对事务的支持，没有数据库的事务支持，spring是无法提供事务功能的。

  **编程式事务管理**：这意味你通过编程的方式管理事务，给你带来极大的灵活性，但是难维护。

  **声明式事务管理**：这意味着你可以将业务代码和事务管理分离，你只需用注解和XML配置来管理事务。

- 事务传播：所谓的事务传播就是一个没有被@Transactional修饰的方法A，调用了一个被@Transactional修饰的方法B，那么这个没有被修饰的方法A是如何处理这个B的事务行为的，需要事务还是不需要事务？和其他含有B的事务如何处理关系？这就是七种事务传播行为的作用

  methodA事务方法调用methodB事务方法时，methodB是继续在调用者methodA的事务中运行呢，还是为自己开启一个新事务运行，

  分为外围方法开启事务和外围方法没有开启事务几种类型

  https://juejin.im/entry/6844903566205779982

  ``` java
  public void methodA(){ 
      methodB(); 
      //doSomething 
  } 
  
  @Transactional(Propagation=XXX) 
  public void methodB(){ 
      //doSomething 
  }
  ```

- 事务隔离：即为数据库的事务隔离级别（ACID中的隔离性的实现的隔离级别）

- @Transactional 只能被应用到public方法上, 对于其它非public的方法,如果标记了@Transactional也不会报错,但方法没有事务功能.

  - 这个注解可以设置事务传播行为、隔离级别、超时回滚等
  - 实现机制：基于AOP代理实现

- 要注意自调用问题，其实就是AOP的自调用问题

| 事务传播行为类型          | 说明                                                         |
| ------------------------- | ------------------------------------------------------------ |
| PROPAGATION_REQUIRED      | 如果当前没有事务，就新建一个事务，如果已经存在一个事务中，加入到这个事务中。这是最常见的选择。 |
| PROPAGATION_SUPPORTS      | 支持当前事务，如果当前没有事务，就以非事务方式执行。         |
| PROPAGATION_MANDATORY     | 使用当前的事务，如果当前没有事务，就抛出异常。               |
| PROPAGATION_REQUIRES_NEW  | 新建事务，如果当前存在事务，把当前事务挂起。                 |
| PROPAGATION_NOT_SUPPORTED | 以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。   |
| PROPAGATION_NEVER         | 以非事务方式执行，如果当前存在事务，则抛出异常。             |
| PROPAGATION_NESTED        | 如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则执行与PROPAGATION_REQUIRED类似的操作。 |

# 常见注解

（todo：各类注解，❌spring在自定义注解上做了哪些优化）

配置注解：（注解配置代替xml配置：）

- @Configuration  标记类可以当做一个bean的定义

- @Bean 表示此方法要返回一个对象，并作为一个bean注册

- @Component 将 java 类标记为 bean

- @Controller  Web MVC 控制器

- @Service 不会对 @Component 注解提供任何其他行为

- @Repository 说人话就是有一个数据持久层的额外处理，处理了数据持久层的异常

  注解是具有类似用途和功能的 @Component 注解的特化，使未经检查的异常有资格转换为 `Spring DataAccessException`

自动装配如其意，`@Autowired`，实际上感觉这也是IOC的一部分，xml配置中共有5种自动装配（装配的来源依据），具体的可以查文档：

- `@Autowired`可用于构造函数、成员变量、Setter方法
- `@Autowired`默认是按照类型装配注入 `@Resource`默认是按照名称来装配注入

@Autowired 可以更准确地控制应该在何处以及如何进行自动装配。此注解用于在 setter 方法，构造函数，具有任意名称或多个参数的属性或方法上自动装配 bean。默认情况下，它是类型驱动的注入。

当您创建多个相同类型的 bean 并希望仅使用属性装配其中一个 bean 时，您可以使用@Qualifier 注解和 @Autowired 通过指定应该装配哪个确切的 bean 来消除歧义。

@Required 应用于 bean 属性 setter 方法。











https://blog.csdn.net/nextyu/article/details/78669997



Autowired和Resource的区别

@Resource和@Autowired都是做bean的注入时使用，其实@Resource并不是Spring的注解，它的包是javax.annotation.Resource，需要导入，但是Spring支持该注解的注入。

两者都可以写在字段和setter方法上。两者如果都写在字段上，那么就不需要再写setter方法。



@Autowired为Spring提供的注解，需要导入包org.springframework.beans.factory.annotation.Autowired;只按照byType注入。，默认情况下它要求依赖对象必须存在，如果允许null值，可以设置它的required属性为false。如果我们想使用按照名称（byName）来装配，可以结合@Qualifier注解一起使用。



@Resource默认按照ByName自动注入，由J2EE提供，需要导入包javax.annotation.Resource。@Resource有两个重要的属性：name和type，而Spring将@Resource注解的name属性解析为bean的名字，而type属性则解析为bean的类型。所以，如果使用name属性，则使用byName的自动注入策略，而使用type属性时则使用byType自动注入策略。如果既不制定name也不制定type属性，这时将通过反射机制使用byName自动注入策略。



@Resource装配顺序：

①如果同时指定了name和type，则从Spring上下文中找到唯一匹配的bean进行装配，找不到则抛出异常。

②如果指定了name，则从上下文中查找名称（id）匹配的bean进行装配，找不到则抛出异常。

③如果指定了type，则从上下文中找到类似匹配的唯一bean进行装配，找不到或是找到多个，都会抛出异常。

④如果既没有指定name，又没有指定type，则自动按照byName方式进行装配；如果没有匹配，则回退为一个原始类型进行匹配，如果匹配则自动装配。

@Resource的作用相当于@Autowired，只不过@Autowired按照byType自动注入。



@Resource默认按名称装配，如果不到与名称匹配的bean，会按类型装配。

# 其他

ApplicationContext（除了IOC容器之外的功能）

- 继承MessageSource，因此支持国际化。
- 统一的资源文件访问方式。
- 提供在监听器中注册bean的事件。
- 同时加载多个配置文件。
- 载入多个（有继承关系）上下文 ，使得每一个上下文都专注于一个特定的层次，比如应用的web层

ApplicationContext 添加了 AOP 继承支持、消息资源处理（国际化支持）、事件发布、应用层上下文（例如web中的WebApplicationContext）



---



[todo：选择代码中还是配置文件中写依赖关系？](https://martinfowler.com/articles/injection.html#CodeOrConfigurationFiles)

[todo：将配置与使用分开](https://martinfowler.com/articles/injection.html#SeparatingConfigurationFromUse)

- ❌spring的线程池
- ❌一个Controller调用两个Service，这两Service又都分别调用两个Dao，问其中用到了几个数据库连接池的连接？我不知道这个问题考察的是哪个方面的知识？？？

重要的Spring模块

- Spring核心容器：这是Spring的基本模块，提供Spring容器（BeanFactory和ApplicationContext）
- AOP 面向切面编程：可以实现跨领域的关注？
- 控制反转容器：主要通过依赖项注入来完成应用程序组件的配置和Java对象的生命周期管理
- 数据访问：使用Java数据库连接（JDBC）和对象关系映射工具在Java平台上与关系数据库管理系统一起使用，并与NoSQL数据库一起使用
- 消息传递：消息侦听器对象的配置注册，以通过Java消息服务（JMS）从消息队列中透明地消耗消息，改进了通过标准JMS API发送消息的过程
- 事务管理：统一多个事务管理API并协调Java对象的事务
- 认证和授权：可配置的安全过程，通过Spring Security子项目（以前称为Acegi Security System for Spring）支持各种标准，协议，工具和实践。
- 约定优于配置：Spring Roo模块中提供了基于Spring的企业应用程序的快速应用程序开发解决方案
- 模型-视图-控制器：基于HTTP和Servlet的框架，为Web应用程序和RESTful（表示状态传输）Web服务的扩展和自定义提供了挂钩。
- 测试：用于编写单元测试和集成测试的支持类
- 远程访问框架：通过支持Java远程方法调用（RMI），CORBA（公共对象请求代理体系结构）和基于HTTP的协议（包括Web服务，SOAP）的网络上的Java对象的配置远程过程调用（RPC）样式的编组，SOAP（简单对象访问协议）
- 远程管理：通过Java管理扩展（JMX）进行本地或远程配置的Java对象的配置公开和管理
