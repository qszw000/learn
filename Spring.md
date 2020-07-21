1. IOC:

   IOC（Inverse of Control:控制反转）是一种设计思想，就是 将原本在程序中手动创建对象的控制权，交由Spring框架来管理。*IoC 在其他语言中也有应用，并非 Spring 特有。 IoC 容器是 Spring 用来实现 IoC 的载体， IoC 容器实际上就是个Map（key，value）,Map 中存放的是各种对象。

   将对象之间的相互依赖关系交给 IoC 容器来管理，并由 IoC 容器完成对象的注入。这样可以很大程度上简化应用的开发，把应用从复杂的依赖关系中解放出来。 IoC 容器就像是一个工厂一样，当我们需要创建一个对象的时候，只需要配置好配置文件/注解即可，完全不用考虑对象是如何被创建出来的。

2. AOP

   AOP(Aspect-Oriented Programming:面向切面编程)能够将那些与业务无关，却为业务模块所共同调用的逻辑或责任（例如事务处理、日志管理、权限控制等）封装起来，便于减少系统的重复代码**，**降低模块间的耦合度，并有利于未来的可拓展性和可维护性。

   Spring AOP就是基于动态代理的，如果要代理的对象，实现了某个接口，那么Spring AOP会使用JDK Proxy，去创建代理对象，而对于没有实现接口的对象，就无法使用 JDK Proxy 去进行代理了，这时候Spring AOP会使用Cglib ，这时候Spring AOP会使用 Cglib*生成一个被代理对象的子类来作为代理

3. Spring中Bean的作用域

   1. singleton：单例模式，Spring IoC容器中只会存在一个共享的Bean
   2. prototype：原型模式，每次通过Spring容器获取对象时，容器会创建一个新的Bean实例
   3. request：在一次Http请求中，容器会返回该Bean的同一实例
   4. session：在一次Http Session中，容器会返回该Bean的同一实例
   5. global-session：在一个全局的Http Session中，容器会返回该Bean的同一实例，仅在使用portlet context时有效

4. Spring Bean生命周期

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

5. Spring依赖注入的四种方式

   1. 构造器注入
   2. setter方法注入
   3. 静态工厂注入
   4. 实例工厂

6. Spring五种自动装配

   1. no：不进行自动装配，通过显示设置ref属性来进行装配
   2. byName：通过参数名自动装配
   3. byType：通过参数类型自动装配
   4. constructor：通过构造器参数进行装配
   5. autodetect：先参数constructor来自动装配，如果不行则使用byType方式

7. SpringMVC的运行流程

   1. 客户端发送请求，直接请求到DispatcherServlet
   2. DispatcherServlet根据请求信息调用HandlerMapping，解析请求对应的Handler
   3. 解析到对应的Handler后，HandlerAdapter会根据Handler来调用直至的处理器开始处理请求，处理相应的业务逻辑
   4. 处理器处理完业务后，会返回一个ModelAndView对象
   5. ViewResolver会根据逻辑View查找实际的View
   6. DispatcherServler把返回的Model传给View，渲染视图
   7. 把View返回给请求者

8. Spring

   支持当前事务的情况：

   - TransactionDefinition.PROPAGATION_REQUIRED：如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。
   - TransactionDefinition.PROPAGATION_SUPPORTS：如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。
   - TransactionDefinition.PROPAGATION_MANDATORY：如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。（mandatory：强制性）

   不支持当前事务的情况：

   - TransactionDefinition.PROPAGATION_REQUIRES_NEW：创建一个新的事务，如果当前存在事务，则把当前事务挂起。
   - TransactionDefinition.PROPAGATION_NOT_SUPPORTED：以非事务方式运行，如果当前存在事务，则把当前事务挂起。
   - TransactionDefinition.PROPAGATION_NEVER：以非事务方式运行，如果当前存在事务，则抛出异常。

   其他情况：

   - TransactionDefinition.PROPAGATION_NESTED：如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务，则该取值等价于TransactionDefinition.PROPAGATION_REQUIRED。

9. SpringBoot

   SpringBoot是一个框架，一种全新的编程规范，他的产生简化了框架的使用，所谓简化是指简化了Spring众多框架中所需的大量且繁琐的配置文件，所以 SpringBoot是一个服务于框架的框架，服务范围是简化配置文件。 