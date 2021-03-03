学习Spring首先学习IOC和AOP，但这其实不是属于Spring的内容，他们是一种设计理念，这个其实不应该在Spring的章节里讲解，Spring的章节里应该只讲解它特有的，和通用的不一样的，一些特点和技巧等。

- <u>*todo： remove： 把IOC和AOP移动到单独的介绍设计理念文章中去*</u>
- 有时候我们学习一个东西，例如IOC，是不应当依赖于具体的实现去学习的（可能也应当），例如我去学习Spring的IOC实现。我认为的正确的做法是，先去了解IOC是什么，怎么用，是一种什么想法什么模式什么思想，然后来看Spring的IOC实现有什么功能、怎么用。
- IOC和AOP只是编程范式，并没有规定说，实现范式的代码，要用什么方式去实现

1. Spring做的就是通过xml或注解来注入到合适的地方。[Spring就是一个依赖注入框架](https://stackoverflow.com/questions/1061717/what-exactly-is-spring-framework-for)，（之后的种种只是一套完整的解决方案，但是Spring最初的、最根本的就是依赖注入，说是最根本，其实感觉也不是很复杂）

2. spring来负责控制对象的生命周期和对象间的关系，类的创建、销毁都由 spring来控制，控制对象生存周期的不再是引用它的对象，而是spring。

# 控制反转 IOC

IOC(Inversion of Control) 即译为：控制反转。（对于个人而言，这个词语多少有些让人摸不着头脑，但是作为约定俗称的词，这么说就这么说吧）

- [要理解控制反转，理解依赖倒置原则也是很必要的，从依赖倒置原则切入来看控制反转](https://www.zhihu.com/question/23277575)

- 控制反转的设计目的是为了**分离关注点**，分离调用方和依赖，从而提高可读性以及代码重用性。

- 一些控制反转和依赖注入的解释（或许有助于理解）：

  1. **类A原本接受各种参数来构造一个对象B，现在只接受一个参数——已经实例化的对象。**

     也就是说类A对对象B的『依赖』是注入进来的，而A和B的构造方式解耦了。构造B这个『控制』操作也交给了第三方，也就是控制反转

  2. A依赖b，但A不控制B的创建和销毁，仅使用B，那么B的控制权交给A之外处理，这叫控制反转（IOC），而A要依赖B，必然要使用B的instance，那么就会采用依赖注入/或 `Service Locator`

控制反转可以说是一种思想、一种设计、一种编程方法、你也可以说是一种模式，不管怎么样，他其实很简单。下面举一个很简单的例子来说明一下：

``` java
public interface MovieFinder {
    List findAll();
}
class MovieLister{
  private MovieFinder finder;
    
  public MovieLister() {
      finder = new ColonDelimitedMovieFinder("movies1.txt");
  }
    
  /*
  * 这个方法是干什么的不重要，我们只关注这里面使用了一个实例域 finder
  **/
  public Movie[] moviesDirectedBy(String arg) {
      List allMovies = finder.findAll();
      for (Iterator it = allMovies.iterator(); it.hasNext();) {
          Movie movie = (Movie) it.next();
          if (!movie.getDirector().equals(arg)) it.remove();
      }
      return (Movie[]) allMovies.toArray(new Movie[allMovies.size()]);
  }
}
```

在这个类里面我们使用 `ColonDelimitedMovieFinder` 来赋值给 `finder`。现在让我们考虑一下变化（变化就会引来问题，问题就需要解决）。简单的**变化**：

1. `ColonDelimitedMovieFinder` 读取文件的方式现在是读取逗号分割的文件，现在我们要读取冒号分割的文件，也就是需要新的实现方式（固然这可以通过检测文件的分隔符做到，但在某些情况下可能需要添加新的 `MovieFinder` 实现方式）
2. 更复杂的，我们需要读取在SQL数据库、XML文件，或者通过网络获取，或者另一种格式的文件，或者我们需要组合好几种方式来读取分布式的文件

考虑到这些变化，就不能简单的在 `MovieLister` 中简单new一个`ColonDelimitedMovieFinder`，而是应该很自然的把需要的 `MovieFinder` 传入 `MovieLister`，即：

``` java
public MovieLister(MovieFinder finder) {
      this.finder = finder;
}
```

其实这就是控制反转，**不在由自己创建实例，而是由外部传入**。这也是控制反转的一种实现方式：**依赖注入**。（这个方式是依赖注入的构造器传入的方式）

请注意，控制反转不只有依赖注入一种实现方式，另一种方式是 `Service Locator`。

---

控制反转也有它的代价：调试困难

## 两种方式

所有实现IOC的组件或者框架，我们叫它IOC Container即IOC容器。

### 依赖注入 DI

DI(Dependency Injection) 即 译为依赖注入。

从我们采取的行为来看，依赖注入更符合我们这个做法的命名，毕竟控制反转实在让人有点摸不到头脑。

- 控制反转 是一个设计思想，一个面向对象的原则。而依赖注入是一种控制反转的实现方式，实现方式还有 `Service Locator`

依赖注入有以下几种：

1. 构造器注入
2. set函数注入
3. 接口注入

> 构造器注入和set函数注入

只是java代码的方式实现

``` java
class ColonMovieFinder{
  public ColonMovieFinder(String filename) {
      this.filename = filename;
  }
}
class MovieLister{
  public MovieLister(MovieFinder finder) {
      this.finder = finder;       
  }
}

/*
* 配置
* 这种方式的写法其实同时可以支持 构造器注入 和 set注入
**/
private MutablePicoContainer configureContainer() {
    MutablePicoContainer pico = new DefaultPicoContainer();
    Parameter[] finderParams =  {new ConstantParameter("movies1.txt")};
    pico.registerComponentImplementation(MovieFinder.class, ColonMovieFinder.class, finderParams);
    pico.registerComponentImplementation(MovieLister.class);
    return pico;
}

/*
* 使用
**/
public void testWithPico() {
    MutablePicoContainer pico = configureContainer();
    MovieLister lister = (MovieLister) pico.getComponentInstance(MovieLister.class);
    Movie[] movies = lister.moviesDirectedBy("Sergio Leone");
    assertEquals("Once Upon a Time in the West", movies[0].getTitle());
}
```

通过xml的方式实现

``` java
class MovieLister{
    private MovieFinder finder;
    public void setFinder(MovieFinder finder) {
        this.finder = finder;
    }
}
class ColonMovieFinder{
    public void setFilename(String filename) {
        this.filename = filename;
    }
}
```

``` xml
<beans>
    <bean id="MovieLister" class="spring.MovieLister">
        <property name="finder">
            <ref local="MovieFinder"/>
        </property>
    </bean>
    <bean id="MovieFinder" class="spring.ColonMovieFinder">
        <property name="filename">
            <value>movies1.txt</value>
        </property>
    </bean>
</beans>
```

``` java
public void testWithSpring() throws Exception {
    ApplicationContext ctx = new FileSystemXmlApplicationContext("spring.xml");
    MovieLister lister = (MovieLister) ctx.getBean("MovieLister");
    Movie[] movies = lister.moviesDirectedBy("Sergio Leone");
    assertEquals("Once Upon a Time in the West", movies[0].getTitle());
}
```

> 接口注入

``` java
/**
* 注意！下面的代码可能一个类的代码会分开写，这样写是为了便于了解代码编写过程和思考过程的，实际操作时需要把同一个类的代码合并
*/

public interface InjectFinder {
    void injectFinder(MovieFinder finder);
}
class MovieLister implements InjectFinder{
  public void injectFinder(MovieFinder finder) {
      this.finder = finder;
  }  
}

public interface InjectFinderFilename {
    void injectFilename (String filename);
}
class ColonMovieFinder implements MovieFinder, InjectFinderFilename{
  public void injectFilename(String filename) {
      this.filename = filename;
  }
}

/**
* 在某个地方注册容器
**/
class Tester{
    private void configureContainer() {
        container = new Container();
        registerComponents();
        registerInjectors();
        container.start();
	}
    private void registerComponents() {
        container.registerComponent("MovieLister", MovieLister.class);
        container.registerComponent("MovieFinder", ColonMovieFinder.class);
  	}
    private void registerInjectors() {
        container.registerInjector(InjectFinder.class, container.lookup("MovieFinder"));
        container.registerInjector(InjectFinderFilename.class, new FinderFilenameInjector());
  	}
}

/**
* Injector的inject方法 来决定注入哪个target 什么值
*/
public interface Injector {
  public void inject(Object target);
}

public static class FinderFilenameInjector implements Injector {
    public void inject(Object target) {
      ((InjectFinderFilename)target).injectFilename("movies1.txt");      
    }
}

class ColonMovieFinder implements Injector{
  public void inject(Object target) {
    ((InjectFinder)target).injectFinder(this);        
  }
}

class Tester{
  /*
  * 使用
  **/
  public void testIface() {
    configureContainer();
    MovieLister lister = (MovieLister)container.lookup("MovieLister");
    Movie[] movies = lister.moviesDirectedBy("Sergio Leone");
    assertEquals("Once Upon a Time in the West", movies[0].getTitle());
  } 
}  
```

> 注解注入



> 接口注入的优劣

接口注入需要做很多工作，可能必须要编写很多接口来使得事情得到解决，如果只需要一小组接口，这么做还不错。但是组装组件和依赖项需要很多工作，这就是为什么当前的轻量级容器中使用了setter和构造函数注入的原因。

> 选择构造器注入还是set注入

这是面向对象编程的一个更普遍的问题：应该在构造函数中还是使用setter填充字段。

todo: 在这个 https://martinfowler.com/articles/injection.html#ConstructorVersusSetterInjection 这里可以看到一些说法，当然也需要查一下Google或者StackOverflow。

### Service Locator

(也有称呼这个是：依赖查找)

从参考的文章上来说，`Service Locator` 有两种写法，一种是静态的，一种是动态的，

静态的

``` java
class MovieLister{
  MovieFinderLocator locator = ServiceLocator.locator();
  MovieFinder finder = locator.movieFinder();
}

public interface MovieFinderLocator {
    public MovieFinder movieFinder();
}

class ServiceLocator implements MovieFinderLocator{
  private static ServiceLocator soleInstance;
  public static void load(ServiceLocator arg) {
      soleInstance = arg;
  }
  public ServiceLocator(MovieFinder movieFinder) {
      this.movieFinder = movieFinder;
  }
  public static ServiceLocator locator() {
     return soleInstance;
  }
    
  private MovieFinder movieFinder;   
  public static MovieFinder movieFinder() {
      return soleInstance.movieFinder;
  }
}

class Tester{
    private void configure() {
        ServiceLocator.load(new ServiceLocator(new ColonMovieFinder("movies1.txt")));
    }
    
    /*
    * 使用
    **/
    public void testSimple() {
      configure();
      MovieLister lister = new MovieLister();
      Movie[] movies = lister.moviesDirectedBy("Sergio Leone");
      assertEquals("Once Upon a Time in the West", movies[0].getTitle());
    }
}

```

动态的

动态的可以运行时选择，提供获取和加载服务的通用方法

``` java
class ServiceLocator{
  private static ServiceLocator soleInstance;
  public static void load(ServiceLocator arg) {
      soleInstance = arg;
  }
  private Map services = new HashMap();
  public static Object getService(String key){
      return soleInstance.services.get(key);
  }
  public void loadService (String key, Object service) {
      services.put(key, service);
  }
}
class MovieLister{
  MovieFinder finder = (MovieFinder) ServiceLocator.getService("MovieFinder"); 
}

class Tester{
  private void configure() {
      ServiceLocator locator = new ServiceLocator();
      locator.loadService("MovieFinder", new ColonMovieFinder("movies1.txt"));
      ServiceLocator.load(locator);
  }
}


```

无论如何，上面的代码只是万千写法之中的其一，并不是唯一的（但注意，个人觉得静态和动态的划分是很正确的）

### 选择哪一个

首先明确这样两点：

1. 两种做法 提供了 对开头很简单的例子中缺少的 基本解耦
2. 两种做法的区别在于如何将实现提供给应用程序类，即**如何提供实现**

再来看其他的区别：

1. `Service Locator`  的做法中，每个 `locator` 的使用者都对其具有依赖性

   所以选择二者哪一个也取决于，是否可以书写这样的依赖关系，如果不可以那只能选择 依赖注入

2. 由于没有组件到注入器的依赖性，使用 依赖注入 ，在配置之后就没办法从注入器获取更多服务

3. 使用简易性的问题：

   - 使用依赖注入，需要关注组件中的构造器或者set函数，以及config中的配置。一旦config编写好，之后就只需要注意组件中的构造器或者set函数。（当然也需要在config中定义依赖关系）（也许，你知道我在说什么）
   - 使用 `Service Locator` 需要依赖一个 `locator` 并找到可以获取目标实例的方法（当`locator` 中含有很多方法时，这样做并不是很优雅）。这样需要注意在 locator 中添加一个获取方法，并在组件中调用正确的方法。

   这很大程度上由实际情况来决定，如果 程序中很多类使用同一个 `service` （这里是 `MovieFinder`），那么使用 `service locator` 并没有什么不好的地方，反而好像 依赖注入 显得不能让人信服

4. 主动性的问题，在 依赖注入 中组件只能被动接收注入，而在`Service Locator` 中组件可以主动请求。

   依赖查找更加主动，在需要的时候通过调用框架提供的方法来获取对象，获取时需要提供相关的配置文件路径、key等信息来确定获取对象的状态

# 面向切面 AOP

面向对象的特点是继承、多态和封装。而封装就要求将功能分散到不同的对象中去。这样做的好处是降低了代码的复杂程度，使类可重用。但是实践发现，在分散代码的同时，也增加了代码的重复性，比如很多方法都需要日志记录、权限校验。

在这种情况下，AOP应运而生。在运行时，动态地将代码切入到类的指定方法、指定位置上的编程思想就是面向切面的编程。通过应用AOP可以使代码更加简洁，松耦合，并专注于业务逻辑。AOP实质上是更加的“关注点分离”。

- 面向方面的编程提供了一种很好的方式来实现横切**关注点**，例如日志记录，安全性，性能跟踪等标准代码（注意<u>*不会用该标准代码阻塞方法代码*</u>）。这些**跨领域**的概念是必须在许多地方应用的逻辑，但实际上与业务逻辑没有任何关系。

在面向切面编程的思想里面，把功能分为核心业务功能，和周边功能。

- **所谓的核心业务**，比如登陆，增加数据，删除数据都叫核心业务
- **所谓的周边功能**，比如性能统计，日志，事务管理等等

周边功能在面向切面编程AOP思想里，即被定义为切面。在面向切面编程AOP的思想里面，核心业务功能和切面功能分别独立进行开发，然后把切面功能和核心业务功能 "编织" 在一起，这就叫AOP。

AOP和OOP：AOP其实只是OOP的补充而已。OOP从横向上区分出一个个的类来，而AOP则从纵向上向对象中加入特定的代码。

通过应用AOP，将获得几个好处：

1. 每个关注点的逻辑都放在一个地方，而不是分散在整个代码库中。
2. 类更干净，因为它们仅包含针对其主要关注点（或核心功能）的代码，而次要关注点已移至各个方面。
3. 提高程序代码的模块化程度



？？？？？？？？？？？？SSSSSSSSSSSSSSSSSS

面向切面的程序设计将代码逻辑切分为不同的模块（即**关注点**（Concern），一段特定的逻辑功能)。

与切面相关的编程概念还包括[元对象协议](https://zh.wikipedia.org/w/index.php?title=元对象协议&action=edit&redlink=1)、主题（Subject）、[混入](https://zh.wikipedia.org/w/index.php?title=混入&action=edit&redlink=1)（Mixin）和委托（Delegate）

需要特别指出的是，AOP对于解决特定问题，例如事务管理非常有用，这是因为分散在各处的事务代码几乎是完全相同的，并且它们需要的参数（JDBC的Connection）也是固定的。另一些特定问题，如日志，就不那么容易实现，因为日志虽然简单，但打印日志的时候，经常需要捕获局部变量，如果使用AOP实现日志，我们只能输出固定格式的日志，因此，使用AOP时，必须适合特定的场景。

---

**举一个例子：**

假设您有一个带有许多“ set ...（）”方法的图形类。在每种设置方法之后，图形的数据都会更改，因此图形也会更改，因此需要在屏幕上更新图形。假设要重新绘制图形，则必须调用“ Display.update（）”。经典方法是通过添加*更多代码*来解决此问题。在每个set方法的最后，您编写

```
void set...(...) {
    :
    :
    Display.update();
}
```

如果这样的方法有成百上千个（事实上超过十个可能就会让人崩溃），

AOP无需添加大量代码即可解决此问题，而是添加一个方面：

```
after() : set() {
   Display.update();
}
```

另外，您只需要一个切入点：

```
pointcut set() : execution(* set*(*) ) && this(MyGraphicsClass) && within(com.company.*);
```

有人认为，此示例还显示了AOP的一大缺点。AOP实际上正在做许多程序员认为是“ **[反模式](http://en.wikipedia.org/wiki/Anti-pattern)** ”的事情。确切的模式称为“ [远距离动作](http://en.wikipedia.org/wiki/Action_at_a_distance_(computer_science)) ”。

## 几个术语

切点（Pointcut）、建议（Advice）、切面（Aspect）、织入（Weaving）

- 切点（Pointcut）
  在哪些类，哪些方法上切入（where）。是通知（Advice）所要织入（Weaving）的具体位置。

  - 连接点（Join point）：连接点是在应用执行过程中能够插入切面（Aspect）的一个点。这些点可以是调用方法时、甚至修改一个字段时。

  - 连接点和切点的区别：

    连接点是一个虚拟的概念，可以理解为所有满足切点扫描条件的所有的时机。

    具体举个例子：比如开车经过一条高速公路，这条高速公路上有很多个出口（连接点），但是我们不会每个出口都会出去，只会选择我们需要的那个出口（切点）开出去。

    每个应用有多个位置适合织入通知，这些位置都是连接点。但是只有我们选择的那个具体的位置才是切点。

- 建议（Advice）（我不知道为什么都叫`通知`，但是我更喜欢`建议`这个翻译）

  在方法执行的什么时机（**when:**方法前/方法后/方法前后）做什么（**what:**增强的功能），侧重于做什么（感觉when好像不是很重要）

  - 这些时机可以是（方法前后/环绕、方法成功返回后、异常）
  - “方法后”和“方法成功返回后”：“方法成功返回后”是方法返回结果之后执行（因此该通知方法在方法抛出异常时，不能执行），而“方法后”是方法成功不成功，执行完后都会执行

- 切面（Aspect）
  切面 = 切点Pointcut + 建议Advice

- 织入（Weaving）
  把切面加入到对象，并创建出代理对象的过程。即应用AOP的时候就叫织入，织入是一个过程，在上面高速路的例子中开车进入目标路口的过程。

## 实现方式

实现的关键在于代理模式。代理主要分为静态代理和动态代理，静态代理在编译时期加入，动态代理是在运行时相当于多了一个执行步骤。

动态代理一般在运行时织入，这样的效率就没有再编译器织入的高，但是编译器织入可能也会丧失一些灵活性。

具体的做法在下面的实现介绍

# Spring 实现

这部分讲解Spring的IOC和AOP的实现

## IOC实现

（ `org.springframework.beans` and `org.springframework.context` packages 是Spring IOC容器的基础包）

Spring的IOC实现是工厂模式加反射机制，Spring的IOC实现与BeanFactory、ApplicationContext息息相关，要明白Spring的IOC还需要理解他的Bean的概念。当我们明白他的实现方式之后，就需要了解他的Bean的声明周期。

**首先看BeanFactory、ApplicationContext：**

`BeanFactory` 接口提供了管理对象的配置机制（IOC容器），`ApplicationContext` 继承和扩展了 `BeanFactory` 接口。

- BeanFactory：是Spring里面**最底层的接口**，包含了各种Bean的定义，读取bean配置文档，管理bean的加载、实例化，控制bean的生命周期，维护bean之间的依赖关系

- ApplicationContext：是BeanFactory的派生，除了提供BeanFactory所具有的功能外，还提供了更完整的框架功能（其他功能不在这里介绍）

- BeanFactory 可以称为 “低级容器”，ApplicationContext 可以称为 “高级容器”

- 在IOC容器方面它们的区别是：
  - BeanFactory 是懒加载/延迟加载而ApplicationContext是即时加载；
    - 延迟加载形式来注入Bean：即只有在使用到某个Bean时(调用getBean())，才对该Bean进行加载实例化。
    - ApplicationContext，它是在容器启动时，一次性创建了所有的Bean。但是这样占用内存空间较大
    - BeanFactory的这种方式不能在第一时间发现一些存在的Spring的配置问题，指不定什么时候错了
    - 这样的加载方式也是我们的选择区别
  - BeanFactory 是使用语法显式提供资源对象而ApplicationContext是自己创建和管理资源对象；
  - BeanFactory 不支持基于依赖的注解而ApplicationContext支持基于依赖的注解
  
  （ApplicationContext还提供了额外的功能，用以支持IOC、AOP以及其他Spring服务等）
  
- BeanFactory是轻量级的，但是如果您要真正使用Spring，那么不妨使用ApplicationContext：如果您不使用它的高级功能，则涉及的开销很少，但是它们仍然可用如果/何时使用它们。

- 如果您需要在ApplicationContext额外的功能，则应使用ApplicationContext。

**然后看Bean：**

构成应用程序主干并由Spring IOC容器管理的对象称为bean。Bean是由Spring IOC容器实例化，组装和以其他方式管理的对象。Bean及其之间的依赖关系反映在容器使用的配置元数据中。配置元数据以XML、Java代码或Java注解表示。

**IOC初始化流程和Bean的构建过程：**

IOC容器实现他的工作需要这么两个大的阶段：收集和注册Bean，分析和组装Bean。（即构建和绑定）。如果再考虑IOC容器本身，那还要加上IOC容器本身的启动阶段。

- 先找到资源配置文件，然后使用BeanDefinationReader对配置文件进行解析得到容器支持的BeanDefination结构，最后把解析完成的所有的BeanDefination放入容器的一个map容器中便于容器对bean进行统一的管理
  - 资源配置文件包括Resource定位，即xml、config类、注解等
- 对于依赖关系的处理，即组装Bean又叫Spring装配、Bean装配。装配方式有自动装配，自动装配有不同的模式：
  - **no** - 这是默认设置，表示没有自动装配。应使用显式 bean 引用进行装配。
  - **byName** - 它根据 bean 的名称注入对象依赖项。它匹配并装配其属性与 XML 文件中由相同名称定义的 bean。
  - **byType** - 它根据类型注入对象依赖项。如果属性的类型与 XML 文件中的一个 bean 名称匹配，则匹配并装配属性。
  - **构造函数** - 它通过调用类的构造函数来注入依赖项。它有大量的参数。（可以基于index、type、name）
  - **autodetect** - 首先容器尝试通过构造函数使用 autowire 装配，如果不能，则尝试通过 byType 自动装配。

---

### Bean

Bean是被IOC容器管理着的，他的初始化、依赖关系、声明周期、初始数据等都是由IOC容器管理。

Bean的元数据可以被程序员指定，有多种不同的指定方式：XML、JavaConfig、注解。Bean的依赖关系的设置也有多种方式：Set方法、构造器、静态工厂、实例工厂。

Bean有着自己的作用域：

（使用 prototype 作用域需要慎重的思考，因为频繁创建和销毁 bean 会带来很大的性能开销）

- 基本的两个作用域：
  - **Singleton** - 每个 Spring IoC 容器仅有一个单实例。
    - 单例bean不是线程安全的，但是如果设计的bean是一个**无状态的**，那他就是安全的，只要涉及到数据等的操作就很可能是有状态的。
  - **Prototype** - 每次请求都会产生一个新的实例。
- Web场景下的：
  - **Request** - 每一次 HTTP 请求都会产生一个新的实例，并且该 bean 仅在当前 HTTP 请求内有效。
  - **Session** - 每一次 HTTP 请求都会产生一个新的 bean，同时该 bean 仅在当前 HTTP session 内有效。
  - **Global-session**

（<u>*todo:这几种Bean的作用域实现方式是：每当调用IOC容器获取Bean的时候，会判断当前的作用域，然后返回正确的Bean。这些Bean是在IOC容器初始化的时候或其装载的时候就已经放到了正确的作用域？*</u>）

Bean的生命周期：

（bean实例化-->填充属性-->Spring调用bean选择实现的一些列接口-->xxx--->容器关闭-->destroy）

（<u>*todo：有两个重要的bean 生命周期方法，第一个是setup ， 它是在容器加载bean的时候被调用。第二个方法是 teardown 它是在容器卸载类的时候被调用。？？？？*</u>）

![](E:\_data\博文临时库\博文中的图片\Spring-Bean初始化.png)

1. Spring 容器根据配置中的 bean 定义中实例化 bean。
2. Spring 使用依赖注入填充所有属性，如 bean 中所定义的配置。
3. 如果 bean 实现 BeanNameAware 接口，则工厂通过传递 bean 的 ID 来调用 setBeanName()。
4. 如果 bean 实现 BeanFactoryAware 接口，工厂通过传递自身的实例来调用 setBeanFactory()。
5. 如果存在与 bean 关联的任何 BeanPostProcessors，则调用 preProcessBeforeInitialization() 方法。
6. 如果为 bean 指定了 init 方法（`` 的 init-method 属性），那么将调用它。
7. 最后，如果存在与 bean 关联的任何 BeanPostProcessors，则将调用 postProcessAfterInitialization() 方法。
8. 如果 bean 实现 DisposableBean 接口，当 spring 容器关闭时，会调用 destory()。
9. 如果为 bean 指定了 destroy 方法（`` 的 destroy-method 属性），那么将调用它。

---

**POJO和Bean：**

- POJO是比Bean更纯净的简单类或接口。POJO严格地遵守简单对象的概念，而一些Bean中往往会封装一些简单逻辑。

- POJO主要用于数据的临时传递，它只能装载数据， 作为数据存储的载体，而不具有业务逻辑处理的能力。

  Bean虽然数据的获取与POJO一样，但是Bean当中可以有其它的方法。

**其他：**

- 内部Bean：

  在Spring框架中，每当一个bean仅用于一个特定属性时，建议将其声明为一个内部bean。内部bean在setter注入“ property”和构造函数注入“ constructor-arg”中均受支持。像内部类一样，是在另一个类的范围内定义的类。类似地，内部bean是在另一个bean范围内定义的bean。

- 多个Bean：

  @Autowired是根据类型进行自动装配的。如果当spring上下文中存在不止一个A类型的bean时，就会抛出BeanCreationException异常。

  @Quilifier可以解决，即指定名称或者id什么的

  （其实spring的@autowired已经非常智能了，会先根据type找，如果找到多个，在根据名字找，但是如果名字没有找到就会报错，找到了就用这个bean）
  
- Bean的循环依赖：

  “A的某个field或者setter依赖了B的实例对象，同时B的某个field或者setter依赖了A的实例对象”

  将当前正要创建的bean 记录在缓存中 Spring 容器将每一个正在创建的bean 标识符放在一个“当前创建bean 池”中， bean 标识符在创建过程中将一直保持在这个池中，因此如果在创建bean 过程中发现自己已经在“当前 创建bean 池” 里时，将抛出BeanCurrentlylnCreationException 异常表示循环依赖；而对于创建 完毕的bean 将从“ 当前创建bean 池”中清除掉。

  - 构造器的循环依赖：这种依赖spring是处理不了的，直 接抛出BeanCurrentlylnCreationException异常。 

  - 非单例循环依赖：无法处理。

  - **单例模式下的setter循环依赖**：通过“三级缓存”处理循环依赖。 

    Spring为了解决单例的循环依赖问题，使用了三级缓存。在创建bean的时候，会首先从singletonObjects中获取这个bean

    - singletonFactories ： 进入实例化阶段的单例对象工厂的cache （三级缓存）
    - earlySingletonObjects ：完成实例化但是尚未初始化的，提前公开的单例对象的Cache （二级缓存）

    - singletonObjects：完成初始化的单例对象的cache（一级缓存）

    使用这样的三级缓存解决这个问题的步骤：

    1. A首先完成了初始化的第一步，并且将自己提前公开到singletonFactories中
    2. A进行初始化的第二步，发现自己依赖对象B，但发现B还没有被create，因此开始创建B
    3. B在初始化第一步的时候发现自己依赖了对象A，尝试一级缓存singletonObjects没有找到，尝试二级缓存earlySingletonObjects也没有找到，，尝试三级缓存singletonFactories找到了A（虽然A还没有初始化完全），B拿到A对象后顺利完成了初始化阶段
    4. 此时返回A中，A此时能拿到B的对象顺利完成自己的初始化阶段。最终A也完成了初始化，进去了一级缓存singletonObjects中
    
  - 所以回答这个问题：A依赖于B B可以依赖于A吗？就需要考虑情况：构造器循环依赖还是set循环依赖，非单例还是单例
  
- beanFactory和factoryBean有什么区别：

  BeanFactory是个Factory，也就是IOC容器或对象工厂，FactoryBean是个Bean。对FactoryBean而言，这个Bean不是简单的Bean，而是一个能生产或者修饰对象生成的工厂Bean,它的实现与设计模式中的工厂模式和修饰器模式类似。Spring有很多基于它实现的类，例如ProxyFactoryBean自己实现一个FactoryBean，功能：用来代理一个对象，对该对象的所有方法做一个拦截，在调用前后都输出一行LOG

  即一个Bean A如果实现了FactoryBean接口，那么A就变成了一个工厂，根据A的名称获取到的实际上是工厂调用getObject()返回的对象，而不是A本身，如果要获取工厂A自身的实例，那么需要在名称前面加上'&'符号。

  ，bean 无须自己实现工厂模式，Spring 容器担任了工厂的 角色；但少数情况下，容器中的 bean 本身就是工厂，作用是产生其他 bean 实例。由工厂 bean 产生的其他 bean 实例，不再由 Spring 容器产生，因此与普通 bean 的配置不同，不再需要提供 class 元素。

  以当CustomerFactoryBean被扫描进Spring容器时，实际上它向容器中注册了两个bean，一个是CustomerFactoryBean类的单例对象；另外一个就是getObject()方法返回的对象，

  factoryBean是一个扩展点，这在一篇引用文章里可以看到（见下方引用），他讲述了mybatis使用factoryBean的例子

  - getObject('name')返回工厂中的实例
  - getObject('&name')返回工厂本身的实例

- 

## AOP实现

AOP基本概念介绍的部分说过：AOP很多基于代理模式实现，而代理分为静态代理和动态代理。

在Java平台上，对于AOP的织入，有3种方式：

1. 编译期：在编译时，由编译器把切面调用编译进字节码，这种方式需要定义新的关键字并扩展编译器，AspectJ就扩展了Java编译器，使用关键字aspect来实现织入；
2. 类加载器：在目标类被装载到JVM时，通过一个特殊的类加载器，对目标类的字节码重新“增强”；
3. 运行期：目标对象和切面都是普通Java类，通过JVM的动态代理功能或者第三方库实现运行期动态织入。

对于Spring来说，Spring可以用自己的方案来动态代理实现AOP，也可以用AspectJ来静态代理在编译期、类加载期织入实现AOP。

- Spring AOP旨在在Spring IoC上提供一个简单的AOP实现，以解决程序员面临的最常见问题。它<u>*不打算用作完整的AOP解决方案 -只能应用于由Spring容器管理的bean*</u>。

- AspectJ是原始的AOP技术，旨在提供完整的AOP解决方案。它比Spring AOP更强大，但也大大复杂。还值得注意的是，AspectJ可以应用于所有域对象

  - AspectJ在运行时不执行任何操作，因为类是直接用方面编译的。

    因此，与Spring AOP不同，它不需要任何设计模式。为了将这些方面编织到代码中，它引入了称为AspectJ编译器（ajc）的编译器，通过它我们可以编译程序，在编译阶段生成AOP代理类，因此也称为编译时增强，他会在编译阶段将AspectJ(切面)织入到Java字节码中，运行的时候就是增强之后的AOP对象。然后通过提供一个小的（<100K）运行时库来运行它

    AspectJ使用上更为复杂，但是性能要比Spring实现要高的多（快8-15倍）

- Spring缺少对字段连接点、构造器连接点。这是因为Spring是基于动态代理，（但是感觉是可以实现的，代理么，代理类里多几个字段不就得了。）

Spring自己实现的动态代理的AOP有两种情况：

- JDK动态代理– 如果要代理的对象，实现了某个接口，那么Spring AOP会使用JDK Proxy，去创建代理对象；
- CGLIB代理–对于没有实现接口的对象，Spring AOP会使用Cglib，生成一个被代理对象的子类，来作为代理
  - 如果某个类被标记为final，则无法使用CGLIB做动态代理

![](E:\_data\博文临时库\博文中的图片\Spring-AOP实现.png)

### 不能自调用

Spring的基于代理模式的AOP实现是不能自调用的，即下面这种情况：

``` java
@Service 
public class SomeService { 
    private Logger logger = LoggerFactory.getLogger(getClass()); 
    public void hello(String someParam) { 
        logger.info("---SomeService: hello invoked, param: {}---", someParam); 
        test(); 
    } 
    @MyMonitor // 这里是一个切点
    public void test() { 
        logger.info("---SomeService: test invoked---"); 
    } 
}
```

在这个例子中调用hello，是不能触发test的切面的方法的，因为基于代理实现的AOP切面是作用在代理对象上，而不是原对象上。再hello调用test方法时，相当于执行this.test，而此时的this指的是原对象，不是代理对象。

类似的例子还有：

``` java
public class UserService {
    private UserService self; 
    public void setSelf(UserService self) { this.self = self; } 
    @Transactional 
    public boolean addUser(String userName, String password) { 
        try { 
            // call DAO layer and adds to database. 
        } catch (Throwable e) { 
            TransactionAspectSupport.currentTransactionStatus() 
                .setRollbackOnly(); 
        } 
    } 
    public boolean addUsers(List<User> users) { 
        for (User user : users) { 
            self.addUser(user.getUserName, user.getPassword); 
        } 
    } 
}
```

在这个例子中，@Transactional就是使用AOP实现的，所以这样写不能触发AOP的逻辑。

**解决的办法：**

有两种解决办法：

- 注册bean到自己
- 使用AopContext
- 使用AspectJ

``` java
// 第一种方法
@Service 
public class SomeService { 
    private Logger logger = LoggerFactory.getLogger(getClass()); 
    
    @Autowired
	private SomeService self;// 注意这里
    
    public void hello(String someParam) { 
        logger.info("---SomeService: hello invoked, param: {}---", someParam); 
        self.test(); // 注意这里
    } 
    @MyMonitor // 这里是一个切点
    public void test() { 
        logger.info("---SomeService: test invoked---"); 
    } 
}

// ----------------------------------------------------------------
// 第二种方法
@Service 
public class SomeService { 
    ...;
    public void hello(String someParam) { 
        logger.info("---SomeService: hello invoked, param: {}---", someParam); 
        ((SomeService)AopContext.currentProxy()).test(); // 注意这里
    }     
    ...;
}

// ----------------------------------------------------------------
// 第三种方法
// AspectJ不介绍了就

```

# 参考

> 1. https://stackoverflow.com/questions/242177/what-is-aspect-oriented-programming
>
> 2. https://stackoverflow.com/questions/232884/aspect-oriented-programming-vs-object-oriented-programming
>
> 3. [面向切面的程序设计](https://zh.wikipedia.org/wiki/%E9%9D%A2%E5%90%91%E5%88%87%E9%9D%A2%E7%9A%84%E7%A8%8B%E5%BA%8F%E8%AE%BE%E8%AE%A1)
>
> 4. https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html
>
>    在上述链接的文档中，第一部分的IOC容器介绍了Spring的IOC容器的各种用法。包括但不限于：三种DI实现方式、不同DI实现方式如何注入（包括参数顺序、名称、赋值、别名、外部xml引入等等）、**[还用重要的 `Bean Scopes` Bean作用域的讲解](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#beans-factory-scopes)**、基于注解的如何自定义注解、Bean的声明周期回调、**[声明扫描哪里的类](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#beans-classpath-scanning)**
>
>    其实主要的就是 **DI实现和使用、Bean的定义和处理**
>
> 5. https://segmentfault.com/a/1190000008379179
>
> 6. [Inversion of Control Containers and the Dependency Injection pattern](https://martinfowler.com/articles/injection.html)
>
> 7. https://segmentfault.com/a/1190000015221968
>
> 8. https://juejin.im/post/6844903954615107597
>
> 9. https://stackoverflow.com/questions/243385/beanfactory-vs-applicationcontext

