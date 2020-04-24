# 1. Spring概述
答：Spring是一个轻量级的IOC和AOP容器框架。目的是简化应用开发，让开发者注重业务开发。Spring依赖反射。

## 1.1 核心模块
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200209132802488.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0NzYxMDEy,size_16,color_FFFFFF,t_70)

5.x版本中Web模块的Portlet组件被废弃，增加了异步响应的WebFlux组件。
- Spring Core：核心功能。其他所有都依赖于该类库，提供IoC和DI功能。
- Spring Aspects：为与AspectsJ的集成提供支持
- Spring AOP：面向切面编程的实现
- Spring JDBC：Java数据库连接
- Spring JMS：Java消息服务
- Spring Web：为创建Web应用程序提供支持
- Spring Test：提供对JUnit测试的支持

## 1.2 优缺点
优点：
- 解耦，简化开发。把对象创建和依赖关系交给框架管理，减小了组件间的耦合性；
- AOP面向切面编程，复用性高；
- 声明式事务；
- 能集成其他主流框架。

缺点：
- 依赖反射，反射影响性能

## 1.3 用到的设计模式
- 工厂模式：BeanFactory用来创建对象的实例，简单工厂；
- 单例模式：Bean默认单例；
- 代理模式：AOP用到的JDK动态代理和字节码生成；
- 观察者模式：一对多，一发生改变，所有依赖都被动更新，比如listener。

## 1.4 提供事件
答：Spring提供5种事件。
- 上下文更新事件：refresh()
- 上下文开始事件：start()
- 上下文停止事件：stop()
- 上下文关闭事件：容器关闭时，管理的bean全部销毁
- 请求处理事件：web应用中http请求结束触发


# 2. IOC
答：IOC控制反转，是一种设计思想。将对象的控制权由程序本身转移给Spring框架管理。

优点有:
- 解耦
- 自动管理对象创建和关系维护，释放程序员
- 代码量大幅降低

IOC容器是实现IOC的载体，内部是一个Map，存放各种对象。


## 2.1 BeanFactory的IOC
答：最基础的IOC两个步骤。
- 加载配置文件，解析成BeanDefinition放进Map；
- 调用getBean()时，从Map拿出对象实例化，有依赖关系就递归getBean()。


## 2.2 IOC初始化过程
答：对Bean**资源定位、载入和注册**。
- **BeanDefinition定位**：资源定位，理解为IOC容器找数据，由ResourceLoader通过统一的Resource接口来完成定位。

- **BeanDefinition载入**：把Bean表示成BeanDefinition(容器的内部数据结构)。IOC容器内部维护着一个BeanDefinitionMap的数据结构。

- **BeanDefinition注册**：将BeanDefition保存到Map中的过程，通过BeanDefinitionRegistry接口来实现注册。


**总结：找数据 -> 把Bean转为BeanDefinition -> 把BeanDefinition保存到Map**


## 2.3 DI实现方式
答：三种方法。
- 接口注入：被废弃；
- 构造器注入：通过类的构造方法实现注入，每次修改都要创建实例；
- Setter方法注入：通过实例化bean后的setter方法实现注入，每次都会覆盖setter属性。

## 2.4 BeanFactory和FactoryBean的区别
答：区别为：
- BeanFactory：**Bean工厂**，是一个工厂，IOC的顶层接口，用来**实例化配置Bean**；
- FactoryBean：**工厂Bean**，是一个Bean，作用是**产生其他Bean实例**。

## 2.5 BeanFactory和ApplicationContext区别
答：区别为：
- 依赖关系：**BeanFactory是最底层的接口，ApplicationContext是Bean工厂派生**。
- 加载方式：**BeanFactory是延迟加载**，即用到才加载；**ApplicationContext是启动时一次创建所有**。
- 功能拓展：BeanFactory只负责加载和获取Bean；ApplicationContext拓展了更多的高级功能，如refresh，回调，事件发布等等。
- 注册方式：BeanFactory手动注册，ApplicationContext自动注册。

## 2.6 ApplicationContext
答：ApplicationContext是对Bean工厂的派生，增添加载多个配置文件，多个上下文等功能。
实现形式：
- FileSystemXmlApplicationContext：从XML文件中加载Bean的定义，XML Bean配置文件的全路径名必须提供给它的构造函数。
- ClassPathXmlApplicationContext：从XML文件中加载Bean的定义，在classpath里找Bean配置。
- WebXmlApplicationContext：加载XML文件，定义了一个WEB应用的所有Bean。




# 3. AOP
答：AOP面向切面编程，当需要**在某一方法之前或之后做一些额外的操作**时使用，如日志记录、权限判断、性能监视等。把对多个对象产生影响的公共行为封装为一个可重用的模块，就是切面。

## 3.1 Spring AOP和AspectJ AOP区别
答：总结为：

||Spring AOP | AspectJ AOP
---|---|---|---
代理类型| 动态代理 | 静态处理
增强方式|运行时增强 | 编译时增强 
实现方式|运行时生成AOP对象，在切点增强处理并回调 | 编译时将切面织入字节码


## 3.2 常用术语
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200209132917788.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0NzYxMDEy,size_16,color_FFFFFF,t_70)
1. Target（目标对象）：代理的目标对象，即要增强的类；
2. Joinpoint（连接点）：类里可以被增强的方法；
3. Pointcut（切入点）：对哪些连接点进行拦截；
4. Advice（通知/增强）：拦截到连接点后要做的事；
5. Weaving（织入）：把增强advice应用到目标target的过程；
6. Proxy（代理）：一个类被AOP织入增强后，就产生一个结果代理类；
7. Aspect（切面）：切入点pointcut和增强advice的结合。负责将逻辑编织到指定的连接点中。

简单总结：**target是类，连接点就是类中方法，切入点是要拓展功能的方法，增强就是指拓展功能的逻辑，切面就是把增强应用到具体方法上。**

## 3.3 两种实现方法
### 3.3.1 JDK动态代理
**针对实现了接口的类**。原理是运行期间创建一个接口的实现类完成对目标对象的代理。

步骤：
- 定义一个实现接口InvocationHandler的类
- 通过构造函数，注入被代理类
- 实现invoke(Object proxy, Method method, Object[] args)方法
- 在主函数中获得被代理类的类加载器
- 使用Proxy.newProxyInstance( )产生一个代理对象
- 通过代理对象调用各种方法

### 3.3.2 cglib代理
**针对普通类**代理，对实现接口无要求。原理是对指定类生成一个子类（extends继承），覆盖其中的方法。

步骤：
- 定义一个实现了MethodInterceptor接口的类
- 实现其 intercept()方法，在其中调用proxy.invokeSuper()
 

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200209133306947.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0NzYxMDEy,size_16,color_FFFFFF,t_70)
 
## 3.4 通知/增强类型
- 前置通知Before：在目标方法被调用前调用通知功能；
- 后置通知After：在目标方法完成后调用通知，无论方法执行成功与否；
- 返回通知After-returning ：在目标方法成功执行后调用通知；
- 异常通知After-throwing：在目标方法抛出异常后调用通知；
- 环绕通知Around：通知包裹了被通知的方法，在被通知的方法调用前和调用后执行自定义的行为。
 

# 4. Spring Beans
Spring Beans是形成Spring应用的Java对象，能被ioc初始化、装配和管理。
## 4.1 Spring Bean包含什么
答：包含ioc容器的**配置元数据**，如：如何创建、生命周期、依赖关系等。

Spring框架中可以通过XML、注解、java代码三种形式进行配置元数据。

## 4.2 Bean的作用域
答：Spring支持五种Bean的作用域。
- **singleton**： bean实例唯一，默认单例。
- **prototype**： 一个bean定义有多个实例。
- **request**： 每次HTTP请求都会创建一个bean，该bean仅在当前HTTP request内有效。
- **session**：每次HTTP请求都会创建一个 bean，该bean仅在当前HTTP session 内有效。
- **global-session**： 全局session作用域，仅仅在基于portlet的web应用中才有意义。

## 4.3 单例bean的线程安全问题
答：单例bean不是线程安全的。因为当**多线程操作同一个对象**时，对非静态成员变量的写操作会存在线程安全问题。

理解为：有状态就有数据存储就有线程安全隐患，无状态就不存数据就安全。

解决方案：在类中**定义一个ThreadLocal在每个线程中保存可变成员变量**；将singleton变成prototype。

## 4.4 Bean的生命周期
![spring周期](3B6E30DE06E143DE8AA2735B41649199)

实例化 ——> 设置属性 ——> 检查Aware相关接口，有就setBeanName()、setBeanFactory()、setApplicationContext() ——> BeanPostProcessor的初始化前置处理 ——> 检查是否是初始化bean从而决定调用afterPropertiesSet() ——> 自定义初始化 ——> BeanPostProcessor的初始化后置处理 ——> 使用 ——> destroy() ——> 自定义销毁

## 4.5 自动装配
答：@Autowired注解自动装配指定bean，即容器自动处理bean间的依赖关系。

## 4.6 循环依赖问题
答：A创建过程中要用到B，但B也要用到A，这就是循环依赖。这种构造器依赖无法解决，只能抛出BeanCurrentlyInCreationException异常。

## 4.7 @Component和@Bean区别
||@Component|@Bean|
|--|--|--|
|作用对象|类|方法|
|装配方法|类路径扫描|在方法中定义产生bean|
|自定义性|弱|强|
 

# 5. Spring事务
Spring事务本质就是数据库的事务支持。
## 5.1 事务管理类型
答：Spring事务分为编程式事务管理和声明式事务管理。
- **编程式事务管理**：硬编码，使用Transaction Template实现；
- **声明式事务管理**：配置文件中配置，**基于AOP**，对方法前后进行拦截，将事务处理编织到增强方法中。分为基于XML和基于注解两种。

## 5.2 事务传播行为
支持当前事务：
- PROPAGATION_**REQUIRED**：如果当前没有事务，就创建一个新事务，如果当前存在事务，就加入该事务。默认设置。
**没有就创建，有就加入**
- PROPAGATION_SUPPORTS：如果当前存在事务，就加入该事务，如果当前不存在事务，就以非事务执行。
- PROPAGATION_MANDATORY：如果当前存在事务，就加入该事务，如果当前不存在事务，就抛出异常。

不支持当前事务：
- PROPAGATION_REQUIRES_NEW：创建新事务，如果当前存在事务，就挂起。
- PROPAGATION_NOT_SUPPORTED：以非事务方式执行，如果当前存在事务，就挂起。
- PROPAGATION_NEVER：以非事务方式执行，如果当前存在事务，则抛出异常。

其他情况：
- PROPAGATION_NESTED：如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则按REQUIRED属性执行。

## 5.3 事务隔离级别
答：和MySQL的事务隔离级别类似，但多了一个，**default(用数据库默认的隔离级别)、读未提交、读已提交、可重复读和串行。**

## 5.4 注解
1. @Transaction(rollbackFor=Exception.class)，此注解作用在类上，能让所有public方法都具有事务属性，执行失败就抛出异常并回滚。
2. rollbackFor能让事务在遇到非运行异常(编译异常，写代码时标红)时也回滚。

# 6. Spring MVC
## 6.1 Spring MVC的理解
答：MVC是一种设计模式，Spring MVC是一个轻量级的基于请求驱动的Web框架。一般把后端项目分为Service层（处理业务），Dao层（数据操作），Entity层（实体类），Controller层（控制层）。

## 6.2 Spring MVC工作原理
1. 浏览器发送请求，直接到DispatcherServlet；
2. DispatcherServlet接收，调用HandlerMapping解析对应的Handler(Controller)，传递给HandlerAdapter；
3. HandlerAdapter根据Handler处理对应业务逻辑；
4. 业务处理完成，返回ModelAndView对象；
5. ViewResolver查找实际的View；
6. DispaterServlet把Model传给View，进行渲染；
7. View视图返回给浏览器。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200209134900587.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0NzYxMDEy,size_16,color_FFFFFF,t_70)
