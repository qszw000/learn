# Servlet

Servlet会在服务器启动或第一次请求该Servlet的时候开始生命周期，在服务器结束的时候结束生命周期。无论请求多少次Servlet，最多只有一个Servlet实例。多个客户端并发请求Servlet时，服务器会启动多个线程分别执行该Servlet的service方法

### 概述

##### 工作流程

- Tomcat将浏览器提交的请求封装成HttpServletRequest对象，同时将输出流封装成HttpServletResponse对象
- Tomcat把request、response作为参数，调用Servlet的相应方法，例如doGet(request, response)等
- Servlet中主要处理业务逻辑

##### 接口

实现javax.servlet.Servlet接口类，规定了特定的方法来处理特定的请求。开发者只需要实现Servlet的相关方法，用户访问Web程序的时候，Tomcat会调用这些方法完成业务处理

##### Java Web目录结构

| 文件夹            | 描述                                                         |
| ----------------- | :----------------------------------------------------------- |
| /                 | Web应用根目录                                                |
| /WEB-INF/         | Tomcat会隐藏该文件夹下的所有文件及文件夹，保护它们不能通过浏览器直接访问 |
| /WEB-INF/web.xml  | 主要的配置文件                                               |
| /WEB-INF/classes/ | 类文件，包括Servlet类                                        |
| /WEB-INF/lib/     | jar文件位置                                                  |

### 配置

#### servlet

```
<servlet>
  <servlet-name>[唯一的名称]</servlet-name>
  <servlet-class>[包含包名的路径]</servlet-class>
  <init-param>
    <param-name>[配置名称]</param-name>
    <param-value>[配置值]</param-value>
  </init-param>
  <load-on-startup>1[0:请求时加载；1:启动时加载]</load-on-startup>
</servlet>
复制代码
```

#### 访问路径配置 servlet-mapping

```
<servlet-mapping>
  <servlet-name>[同servlet配置中的名称对应]</servlet-name>
  <url-pattern>[访问路径]</url-pattern>
</servlet-mapping>
复制代码
```

#### 上下文参数 context-param

全局共享，能够被所有的servlet读取

```
<context-param>
  <param-name>[配置名称]</param-name>
  <param-value>[配置值]</param-value>
</context-param>
复制代码
```

在Servlet类中，通过getServletConfig.getServletContext获取ServletContext，然后通过其方法获取相应上下文参数

### Servlet生命周期

Servlet运行在Servlet容器中，其生命周期由容器来管理。Servlet的生命周期通过Servlet接口中的init()、service()和destory()方法来表示。 

- **Init()**

  在Servlet实例化之后，Servlet容器会调用init()方法，来初始化该对象，主要是为了让Servlet对象在处理客户端请求前可以完成一些初始化的工作，例如：建立数据库连接、获取配置信息等。对于每一个Servlet实例，init()方法只能被调用一次。init()方法有一个类型为ServletConfig的参数，Servlet容器通过这个参数向Servlet传递配置信息。Servlet使用Servlet对象从Web应用程序的配置信息中获取以***名-值对***形式提供的***初始化参数***。另外，在Servlet中，还可以通过ServletConfig对行啊获取描述Servlet运行环境的ServletContext对象，使用该对象，Servlet可以和它的Servlet容器进行通信。

- **service()**

  容器调用service()方法来处理客户端请求。要注意，在Servlet方法被容器调用之前，必须确保init()方法正确完成。容器会构造一个表示客户端请求信息的请求对象（类型为ServletRequest）和一个用于对客户端进行响应的响应对象（类型为ServletResponse）作为参数传递给service()方法。在service方法中，Servlet对象通过ServletRequest对象得到客户端的相关信息和请求信息，在对请求进行处理后，调用ServletResponse对象的方法设置响应信息。

- **destory()**

  当容器检测到一个Servlet对象应该从服务中被移除的时候，容器会调用该对象的destory()方法，以便让Servlet对象可以释放它所使用的资源，同时保存数据到持久存储设备中，例如将内存中的数据保存到数据库中，关闭数据库连接等。当需要释放内存或者容器关闭时，容器就会调用Servlet对象的destory()方法。在Servlet容器调用destory()方法之前，如果还有其他线程正在service()方法中执行，容器将会等待这些线程执行完毕或等待服务器设定的超时时间到达。一旦Servlet对象的destory()方法被调用，容器不会再把其他的请求发送给该对象。如果需要该Servlet再次为客户端服务，容器将会重新产生一个Servlet对象来处理客户端请求。在destory()方法调用之后，容器会释放这个Servlet对象，在随后的时间内，该对象会被Java垃圾收集器所回收。

![](C:\Users\hsy\Desktop\learn\spring img\QQ截图20200920225200.png)

#### Servlet注解

- @PostContructor 
  - 在构造函数执行之后，init函数执行之前被调用
- @PreDestroy 
  - 在destroy方法之后，Servlet被彻底卸载之前被调用
- 注解会影响服务器的启动速度
  - 服务器在启动时会遍历Web应用的WEB-INF/classes下的所有class文件与WEB-INF/lib下的jar文件以检查哪些类使用了注解

### 线程安全性

##### 隐患原因

 Servlet规范中定义，默认情况下（Servlet不是在分布式的环境中部署），Servlet容器对声明的每一个Servlet，只创建一个实例。如果有多个客户端请求同时访问这个Servlet，Servlet容器如何处理多个请求呢？答案是采用多线程，Servlet容器维护一个线程池来服务请求。当容器接收到一个访问Servlet的请求，调度者线程从线程池中选取一个工作线程，将请求传递给该线程，然后由这个线程执行Servlet的service()方法。 

由于Servlet只会有一个实例，多个用户同时请求同一个Servlet时，Tomcat会派生出多条线程执行Servlet的代码，因此Servlet有线程不安全的隐患。

# Filter

Filter对用户请求进行预处理，接着将请求交给Servlet进行处理并生成响应，最后Filter再对服务器响应进行后处理。Filter是可以复用的代码片段，常用来转换HTTP请求、响应和头信息。Filter不像Servlet，它不能产生响应，而是只修改对某一资源的请求或者响应。

 一个 Servlet 请求可以经由多个Filter进行过滤，最终由Servlet处理并响应客户端。这里配置两个过滤器示例： 

### Filter生命周期

web.xml 中声明的每个 filter 在每个虚拟机中仅仅只有一个实例。

1. 加载和实例化
	Web 容器启动时，即会根据 web.xml 中声明的 filter 顺序依次实例化这些 filter。
3. 初始化
Web 容器调用 init(FilterConfig) 来初始化过滤器。容器在调用该方法时，向过滤器传递 FilterConfig 对象，FilterConfig 的用法和 ServletConfig 类似。利用 FilterConfig 对象可以得到 ServletContext 对象，以及在 web.xml 中配置的过滤器的初始化参数。在这个方法中，可以抛出 ServletException 异常，通知容器该过滤器不能正常工作。此时的 Web 容器启动失败，整个应用程序不能够被访问。实例化和初始化的操作只会在容器启动时执行，而且只会执行一次。
5. doFilter
	doFilter 方法类似于 Servlet 接口的 service 方法。当客户端请求目标资源的时候，容器会筛选出符合 filter-mapping 中的 url-pattern 的 filter，并按照声明 filter-mapping 的顺序依次调用这些 filter 的 doFilter 方法。在这个链式调用过程中，可以调用 chain.doFilter(ServletRequest, ServletResponse) 将请求传给下一个过滤器(或目标资源)，也可以直接向客户端返回响应信息，或者利用 RequestDispatcher 的 forward 和 include 方法，以及 HttpServletResponse 的 sendRedirect 方法将请求转向到其它资源。需要注意的是，这个方法的请求和响应参数的类型是 ServletRequest 和 ServletResponse，也就是说，过滤器的使用并不依赖于具体的协议。
7. 销毁
	Web 容器调用 destroy 方法指示过滤器的生命周期结束。在这个方法中，可以释放过滤器使用的资源。

# Listener

Listener可以监听web服务器中某一个事件操作，并触发注册的回调函数。通俗的语言就是在 application，session，request三个对象创建/消亡或者增删改属性时，自动执行代码的功能组件。 

### Listener生命周期

Listener在当web容器启动的时候，去读取每个web应用的web.xml配置文件，当配置文件中配有filter和listener时，web容器实例化listener，listener是当某个事件发生时，调用它特定方法，如HttpSessionListener,当创建一个session时会调用它的sessionCreated()方法，当servlet容器关闭或者重新加载web应用时lister对象被销毁。

### Listener分类

不同功能的Listener 需要实现不同的 Listener  接口，一个Listener也可以实现多个接口，这样就可以多种功能的监听器一起工作。常用监听器：

1）监听 Session、request、context 的创建于销毁，分别为  

HttpSessionLister、ServletContextListener、ServletRequestListener

2）监听对象属性变化，分别为：

HttpSessionAttributeLister、ServletContextAttributeListener、ServletRequestAttributeListener

# Interceptor

类似面向切面编程 中的 切面 和通知，我们通过动态代理对一个service()方法添加 通知 进行功能增强。比如说在方法执行前进行 初始化处理，在方法执行后进行 后置处理。拦截器 的思想和AOP类似，区别就是 拦截器 只能对Controller的HTTP请求进行拦截。

# Spring

### IoC

IoC（Inverse of Control:控制反转）是一种**设计思想**，就是 **将原本在程序中手动创建对象的控制权，交由Spring框架来管理。** IoC 在其他语言中也有应用，并非 Spring 特有。 **IoC 容器是 Spring 用来实现 IoC 的载体， IoC 容器实际上就是个Map（key，value）,Map 中存放的是各种对象。**

将对象之间的相互依赖关系交给 IoC 容器来管理，并由 IoC 容器完成对象的注入。这样可以很大程度上简化应用的开发，把应用从复杂的依赖关系中解放出来。 **IoC 容器就像是一个工厂一样，当我们需要创建一个对象的时候，只需要配置好配置文件/注解即可，完全不用考虑对象是如何被创建出来的。** 

下图spring Ioc的整体架构图中可以看出，Spring启动时读取bean配置信息，并在spring中生成一份相应bean配置注册表，然后会根据注册表实例化Bean，并且装配好bean 之间的依赖关系，为上层应用提供bean实例，其中bean的缓存池是通过hashmap实现的

![img](https://gitee.com/cbaj/blogImag/raw/master/img/20200804221959.png)

### bean的构建过程

spring.xml中保存了Bean的描述配置，BeanFactory读取这些配置然后生成bean，这是我们对ioc原理的一般理解，深入思考还会有更多问题产生

- 哪个java对象承载了配置信息里的内容
- 这些承载对象是读取配置文件并装载的
- 这些装载对象又保存在哪里

#### BeanDefinition（Bean定义）

ioc 实现中 我们在xml 中描述的Bean信息最后 都将保存至BeanDefinition （定义）对象中，其中xml bean 与BeanDefinition 程一对一的关系。xml  bean中设置的属性最后都会体现在BeanDefinition中

![img](https://gitee.com/cbaj/blogImag/raw/master/img/image.png!thumbnail.png)

由此可见，xml  bean中设置的属性最后都会体现在BeanDefinition中。如:

| **XML-bean**    | **BeanDefinition**                       |
| --------------- | ---------------------------------------- |
| class           | beanClassName                            |
| scope           | scope                                    |
| lazy-init       | lazyInit                                 |
| constructor-arg | ConstructorArgument                      |
| property        | MutablePropertyValues                    |
| factory-method  | factoryMethodName                        |
| destroy-method  | AbstractBeanDefinition.destroyMethodName |
| init-method     | AbstractBeanDefinition.initMethodName    |
| autowire        | AbstractBeanDefinition.autowireMode      |
| id              |                                          |
| name            |                                          |

#### BeanDefinition 属性结构

![img](https://gitee.com/cbaj/blogImag/raw/master/img/20200805101002.png)

#### BeanDefinitionRegistry（Bean注册器）

在上表中我们并没有看到 xml bean 中的 id  和name属性没有体现在定义中，原因是ID 其作为当前Bean的存储key注册到了BeanDefinitionRegistry 注册器中。name 作为别名key 注册到了 AliasRegistry 注册中心。其最后都是指向其对应的BeanDefinition。

 **BeanDefinitionRegistry属性结构**

![img](https://gitee.com/cbaj/blogImag/raw/master/img/20200805101338.png)

**BeanDefinitionReader（Bean定义读取）**

现在我们了解 BeanDefinition 中存储了Xml Bean信息，而BeanDefinitionRegister 基于ID和name 保存了Bean的定义。接下要学习的是从xml Bean到BeanDefinition 然后在注册至BeanDefinitionRegister 整个过程。

![img](https://gitee.com/cbaj/blogImag/raw/master/img/20200805102556.png)

上图中可以看出Bean的定义是由BeanDefinitionReader 从xml 中读取配置并构建出 BeanDefinitionReader,然后在基于别名注册到BeanDefinitionRegister中。

BeanDefinitionReader结构

![img](https://gitee.com/cbaj/blogImag/raw/master/img/20200805102838.png)

- int loadBeanDefinitions(Resource var1)

  基于资源加载beanDefinition并注册到注册器

- int loadBeanDefinitions(String var1)

   基于资源路径加载beanDefinition并注册大屏注册器

- BeanDefinitionRegistry getRegistry()

  获取注册器

- ResourceLoader getResourceLoader()

  获取资源装载器

### Beanfactory(bean 工厂)

 有了Bean的定义就相当于有了产品的配方，接下来就是要把这个配方送到工厂进行生产了。在ioc当中Bean的构建是由BeanFactory 负责的。其结构如下：

![img](https://gitee.com/cbaj/blogImag/raw/master/img/20200805110216.png)

**方法说明：**

- getBean(String) 
  - 基于ID或name 获取一个Bean
- T getBean(Class requiredType) 
  - 基于Bean的类别获取一个Bean（如果出现多个该类的实例，将会报错。但可以指定 primary=“true” 调整优先级来解决该错误 ）
- Object getBean(String name, Object... args)
  - 基于名称获取一个Bean，并覆盖默认的构造参数
- boolean isTypeMatch(String name, Class<?> typeToMatch)
  - 指定Bean与指定Class 是否匹配

**Bean创建时序图：**

![img](https://gitee.com/cbaj/blogImag/raw/master/img/bean创建的时序图.png)

从调用过程可以总结出以下几点：

1. 调用BeanFactory.getBean() 会触发Bean的实例化。
2. DefaultSingletonBeanRegistry 中缓存了单例Bean
3. Bean的创建与初始化是由AbstractAutowireCapableBeanFactory 完成的。

### Bean的作用域

1. singleton：单例模式，Spring IoC容器中只会存在一个共享的Bean
2. prototype：原型模式，每次通过Spring容器获取对象时，容器会创建一个新的Bean实例
3. request：在一次Http请求中，容器会返回该Bean的同一实例
4. session：在一次Http Session中，容器会返回该Bean的同一实例
5. global-session：在一个全局的Http Session中，容器会返回该Bean的同一实例，仅在使用portlet context时有效

### Bean的生命周期

1. 创建阶段
   1. Bean容器找到配置文件中Spring Bean的定义
   2. Bean容器利用Java Reflection API创建一个Bean实例
   3. Bean容器按照Spring上下文对实例化的Bean进行配置
   4. 如果Bean实现了BeanNameAware接口，调用setBeanName方法，传入Bean的名字
   5. 如果Bean实现了其他的*Awre接口，调用相应的方法
   6. 如果Bean关联了BeanPostProcessor接口，将会调用postProcessBeforeInitialization方法进行初始化预处理
   7. 如果Bean在Spring配置文件中配置了init-method属性会自动调用其配置的初始化方法
   8. 如果Bean关联了BeanPostProcessor接口，将会调用postProcessAfterInitialization方法
2. 销毁阶段
   1. 如果Bean实现了DisposableBean接口，会调用destroy方法
   2. 如果Bean在Spring配置文件中配置了destroy-method属性，会自动调用其配置的销毁方法

### AOP

AOP：Aspect oriented programming 面向切面编程，AOP 是 OOP（面向对象编程）的一种延续。 

AOP(Aspect-Oriented Programming:面向切面编程)能够将那些与业务无关，却为业务模块所共同调用的逻辑或责任（例如事务处理、日志管理、权限控制等）封装起来，便于减少系统的重复代码**，**降低模块间的耦合度，并有利于未来的可拓展性和可维护性。

Spring AOP就是基于动态代理的，如果要代理的对象，实现了某个接口，那么Spring AOP会使用JDK Proxy，去创建代理对象，而对于没有实现接口的对象，就无法使用 JDK Proxy 去进行代理了，这时候Spring AOP会使用Cglib ，这时候Spring AOP会使用 Cglib*生成一个被代理对象的子类来作为代理