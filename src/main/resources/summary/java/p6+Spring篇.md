# 十、Spring篇

### 1.1. 说说你对 Spring 的理解？

1. Spring 是一个开源框架，可以为了简化企业级 Java 开发过程，使得开发变得更加优雅和简洁。
2. Spring 是一个 **IoC** 控制反转和 **AOP** 面向切面编程的**容器框架**，包含并管理了对象的生命周期。

### 1.2. 说说 Spring 的优势有哪些？

1. Spring 通过 DI、AOP 和消除样板式代码，来简化企业级 Java 开发，同时由于低侵入式的设计，对业务代码的污染极低，以及其扩展性、开放性极高，使得并不强制应用完全依赖于 Spring，开发者可以自由选用 Spring 框架的部分或全部。
2. 其中， Spring 的 IoC 容器，降低了对象替换的复杂性，提高了组件之间的解耦。
3. Spring 的 AOP 则支持允许将一些通用任务如安全、事务、日志等进行集中式处理，从而提供了更好的复用。
4. Spring 的 ORM 和 DAO，则提供了与第三方持久层框架的的良好整合，并简化了底层的数据库访问。
5. 除 Spring 框架之外，还存在一个构建在核心框架之上的庞大生态圈，它将 Spring 扩展到不同的领域，比如 Web 服务、REST、移动开发、 NoSQL 以及 CloudFundry 这些。

### 1.3. Spring 是如何简化企业级 Java 开发的？

1. 基于 POJO 的轻量级和最小侵入性编程。
2. 通过依赖注入和面向接口，实现松耦合。
3. 基于切面和惯例，进行声明式编程。
4. 通过切面和模板，减少样板式代码。

### 1.4. 说说你对 IoC 的理解？

1. **IoC**，Inversion of Control，**控制反转**，是⼀种设计思想，指将原本在程序中⼿动创建对象的控制权，交由专门的容器来进行管理，比如 Spring，而 IoC 在其他语⾔中也有应⽤，并⾮ Spring 特有。 
2. **IoC 容器**，是 Spring ⽤来实现 IoC 的载体， 实际上就是个 Map（key，value）键值对，其中存放的是各种对象，将对象之间的相互依赖关系**交给 IoC 容器来管理**，并由 IoC 容器**完成对象的注⼊**。
   - **DI**：Dependancy Injection，依赖注入，站在容器的角度，将对象创建依赖的其他对象，注入到该对象中。
3. 也就是，**IoC 容器**像⼀个⼯⼚，当需要创建⼀个对象时，只需要配置好配置⽂件或者注解即可，不⽤考虑对象是如何被创建出来的，这样可以很⼤程度上简化应⽤的开发，把应⽤从复杂的依赖关系中解放出来。 

### 1.5. Spring IOC 容器的实现原理？

1. 先准备一个基本的容器对象，包含一些 map 结构的集合，用来方便后续过程中存储具体的对象。
2. 进行配置文件的读取工作，或者注解的解析工作，将需要创建的 bean 对象，都封装成 BeanDefinition 对象存储在容器中。
3. 容器将封装好了的 BeanDefinition 对象，通过反射的方式进行实例化，完成对象的实例化工作。
4. 进行对象属性的依赖注入工作，完成整个对象的创建，成为一个 bean 对象，存储在容器的 map 结构中。
5. 通过容器来获取对象，进行对象的获取和逻辑处理工作。
6. 提供销毁操作，当对象不用，或者容器关闭时，将无用的对象进行销毁。

### 1.6. Spring Bean 的生命周期？

#### 从生命周期长短来看

- **单例对象**： singleton，此时，对象的生命周期和容器相同。
- **多例对象**： prototype，此时，对象的生命周期有 3 个阶段：
  1. **出生**：使用对象时，Spring 才进行创建。
  2. **存活**：对象只要是在使用，那么就一直活着。
  3. **死亡**：当对象长时间不用，且没有其它对象引用时，会由 Java 垃圾回收机制回收。

#### 从构造过程的步骤来看

  ![1645326910721](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1645326910721.png)

1. **Bean 对象实例化**：通过反射的方式进行对象的创建，此时的创建只是在堆空间中申请内存空间，属性都是默认值。
2. **注入对象属性**：给对象中的属性进行依赖注入的工作。
3. **执行 Aware**：
   1. 如果这个 Bean 实现了 `BeanNameAware` 接口，那么就会调用它实现的 `setBeanName(String)` 方法，由 Spring 容器来传入 `beanName` 的值。
   2. 如果这个 Bean 实现了 `BeanClassLoaderAware` 接口，那么就会调用它实现 的`setBeanClassLoader(ClassLoader)`，由 Spring 容器来传入 `ClassLoader` 对象。
   3. 如果这个 Bean 实现了 `BeanFactoryAware` 接口，那么就会调用它实现 的`setBeanFactory(BeanFactory)`，由 Spring 容器来传入 `BeanFactory` 对象。
4. **BeanPostProcessor 前置处理**：如果这个 Bean 实现了 `BeanPostProcessor` 接口，那么就会调用它实现的 `postProcessBeforeInitialization()` 方法，对生成的 Bean 对象进行前置处理工作，比如经常用作对 Bean 内容的更改，由于这个是在 Bean 初始化结束时调用的方法，所以可以被应用于缓存技术。
5. **InitializatingBean、init-method 检查和执行**：
   1. 如果这个 Bean 实现了 InitializatingBean 接口，那么就会调用它实现的 `afterPropertiesSet()`，在属性设置后的进行一些处理工作。
   2. 如果这个 Bean 自定义并配置了 `init-method` 方法，那么就会调用其初始化方法。
6. **BeanPostProcessor 后置处理**：如果这个 Bean 实现了 `BeanPostProcessor` 接口，那么就会调用 `postProcessAfterInitialization()`，对生成的 Bean 对象进行后置处理工作，比如打印日志等。
7. **注册回调方法**：如果对象实现了 `DestructionXXX` 相关的接口，那么就会调用它实现的回调接口，方便以后对象的销毁操作。
8. **Bean 使用**：通过 Spring 容器，来获取 Bean 对象并进行使用。
9. **执行销毁方法**：当Bean不再需要时，会进入清理阶段：
   1. 如果这个 Bean 实现了 `DisposableBean` 接口，自定义并配置了 `destroy-method` 方法，那么就会调用它实现的 `destroy()` 方法，进行对象的销毁工作。

### 1.7. Spring Bean 的作用域？

| 类型           | 作用域                                                       |
| -------------- | ------------------------------------------------------------ |
| **singleton**  | **单例对象，默认值的作用域**                                 |
| **prototype**  | **每次获取都会创建⼀个新的 Bean 实例**                       |
| request        | 每⼀次 HTTP 请求都会产⽣⼀个新的 Bean 实例，该 Bean 仅在当前 HTTP request 内有效 |
| session        | 仅用于 HTTP Session，同一个 Session 共享一个 Bean 实例，不同 Session 使用不同的实例 |
| global-session | 仅用于HTTP Session， 将对象存入到 web 项目集群的 session 域中，所有 Session 共享一个Bean 实例，但若不存在集群，则 global session 相当于 session |

- 默认作用域是 **singleton**，多个线程访问同一个 bean 时，会存在**线程不安全**问题，其**保障线程安全方法**有：
  1. 在 Bean 对象中，尽量避免不去定义可变的成员变量：有时不太现实。
  2. 在类中定义⼀个 `ThreadLocal` 成员变量，将需要的可变成员变量，保存在该 `ThreadLocal` 中：
     - 这样每个线程中都有一个自己的 ThreadLocalMap 对象，则可以把自己线程的对象保持到其中，各管各的，线程可以正确的访问到自己的对象。
     - 将一个**共用的 ThreadLocal 静态实例作为 key**，将不同对象的引用，保存到不同线程的ThreadLocalMap 中，然后在线程执行的各处，通过这个静态`ThreadLocal#get()` 方法取得自己线程保存的那个对象，避免了将这个对象作为**参数传递**的麻烦。

### 1.8. BeanFactory 和 ApplicationContext 有什么区别？

- **相同点**：
  1. **IoC 容器**：两者都是 Spring 的 IoC 容器，都可以用 XML 来配置属性，都支持属性的自动注入。
  2. **接口**：两者都是接口，其中 ApplicationContext 继承于 ListableBeanFactory，ListableBeanFactory 又继承于 BeanFactory，都提供了一种 `getBean(name)`的方式，来获取 Bean 对象。

- **不同点**：
  1. **Bean 提早实例化**：调用 BeanFactory#getBean()  时会对 Bean 进行实例化，而 Application#getBean() 被调用时，并不会再对 Bean 进行实例化，而是早就在容器启动时实例化完毕。
  2. **自动注入方便**：如果使用BeanFactory 来实现自动注入，则需要注册 AutoWiredBeanPostProcessor，而如果使用 ApplicationContext，则可以直接使用 XML 进行 Bean 配置。
  3. **支持事件发布**：BeanFactory不支持事件发布，而 ApplicationContext 能够把事件发布到注册了监听器的Bean 中。
  4. **支持国际化**：BeanFactory不支持国际化，即 i18n，而 ApplicationContext 提供了对它的支持。

- **总结**：
  1. 简而言之，BeanFactory 提供基本的 IoC 和 DI 的功能，而 ApplicationContext 则还提供了高级的功能。
  2. 所以，BeanFactory 只在测试和非生产环境使用，ApplicationContext 则提供更强大的功能来应对生产环境，应该优于 BeanFactory。

### 1.9. Bean Factory 与 Factory Bean 有什么区别？

- **相同点**：都是用来创建 Bean对象的。
- **不同点**：使用 BeanFactory 创建对象时，必须要遵循**严格的生命周期流程**，这太复杂了，如果想要简单地自定义某个对象的创建，同时创建完成的对象还想交给 Spring 来管理的话，那么就可以实现 FactoryBean 接口，其中包含以下三个方法：
  1. **boolean isSingleton()**：是否为单例对象。
  2. **Class<?> getObjectType()**：返回对象的类型。
  3. **Object getObject()**：自定义对象的创建过程，比如 new、反射、动态代理，例子有 FeignClient 的动态代理实现。

### 2.0. BeanFactoryPostProcessor 与 BeanPostProcessor 有什么区别？

- **相同点**：都是接口，都是 Spring 的扩展点。
- **不同点**：BeanFactoryPostProcessor 指的是 Bean 工厂后置处理器，在 Bean 工厂准备完毕后，用于改写 BeanDefinition 的，而 BeanPostProcessor 指的是 Bean 初始化前/后置处理器，是在 Bean 初始化前和初始化后进行调用处理的，分别对应 `postProcessBeforeInitialization` 和 `postProcessAfterInitialization`。

### 2.1. Spring 启动流程原理？

![1645341206075](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1645341206075.png)

```java
public abstract class AbstractApplicationContext extends DefaultResourceLoader
implements ConfigurableApplicationContext {
	@Override
    public void refresh() throws BeansException, IllegalStateException { synchronized (this.startupShutdownMonitor) {
        // 1：刷新前的预处理，包括一些scanner缓存清空、标志位active激活、时间戳记录等 
        prepareRefresh();
        // 2: 获取DefaultListableBeanFactory
        ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
        // 3: 做一些BeanFactory的准备工作，包括设置classLoader、注册environment、systemProperties、systemEnvironment单例等
        prepareBeanFactory(beanFactory);
        try {
                // 4: 允许一些上下文子类在BeanFactory完成准备工作后，做一些后置的处理工作
                postProcessBeanFactory(beanFactory);
                // 5: 实例化并顺序执行BeanFactoryProcessor，以在实例化Bean之前，添加更多的BeanDefinition，其中核心的是ConfigurationClassPostProcessor#postProcessBeanDefinitionRegistry去扫描解析所有@Configuration下的@PropertySource、@ComponentScan、@Import、@ImportResource、@Bean，并加载成为BeanDefinitions
                invokeBeanFactoryPostProcessors(beanFactory);
                // 6: 实例化并注册所有的BeanPostProcessor，但没有执行，要在Bean初始化前/后再执行
                registerBeanPostProcessors(beanFactory);
                // 7: 实例化并注册MessageSource组件，用于做国际化功能、消息绑定、消息解析等
                initMessageSource();
                // 8: 实例化并注册事件多播器，用于观察者模式
                initApplicationEventMulticaster();
                // 9: 允许一些上下文子类，在容器刷新时自定义逻辑 
                onRefresh();
                // 10: 注册ApplicationListener
                registerListeners();
                // 11： 实例化所有剩下非懒加载的单例Bean，并完成它们的初始化和依赖注入，通过遍历所有beanNames，然后挨个判断走FactoryBean的流程，还是走BeanFactory的流程，其中主要步骤总结起来分为3步，分别是NewInstance实例化、Populate属性赋值、Initialization初始化
                finishBeanFactoryInitialization(beanFactory);
                // 12: 完成上下文的刷新，主要是调用LifecycleProcessor#onRefresh()方法
                finishRefresh();
            }
        ...
	}
}
```

### 2.2. Spring 是如何解决循环依赖的？

- **概念**：循环依赖，其实就是 Bean 的循环引用，也就是两个或者两个以上的 Bean 互相持有对方，最终形成依赖闭环，比如 A 依赖于B，B又依赖于A。

- **Spring中循环依赖场景有**: 

  1. prototype 原型 bean 的循环依赖。
  2. 构造器注入的循环依赖（构造器注入）。
  3. Field 属性注入的循环依赖（set注入）。

  => 其中，**构造器的循环依赖**问题无法解决，在解决属性循环依赖，也可以使用**懒加载**，而 Spring 采用的是**提前暴露对象**的方式来解决的。

- **懒加载 @Lazy 解决循环依赖问题**：

  1. Spring 启动时，会把所有的 bean 信息，包括 XML 和注解解析转化成 Spring 能够识别的 BeanDefinition 存到 HashMap 里，供后面初始化时使用。
  2. 然后对每个 BeanDefinition 进行处理，普通 Bean 初始化是在容器启动初始化阶段执行的，而被 `lazy-init=true` 修饰的 bean，则是在从容器里第一次进行 `context.getBean()` 时才会被触发，而此时其依赖的普通 Bean 早就被初始化完毕了，所以可以解决正常情况下的属性注入的循环依赖问题。 

- **三级缓存解决循环依赖问题**：

  ![1645350673009](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1645350673009.png)

  Class A 依赖（这里是 @Autowire） Class B，Class B 又依赖（这里是 @Autowire） Class A，造成循环依赖，Spring 的解决方式是**三级缓存**：

  1. Class A 首先被实例化（一个空壳），实例化后立马放入三级缓存中。
  2. 然后在 populateBean填充 Class A 实例时，回调 @AutoWired 的回调接口`AutowiredAnnotationBeanPostProcessor#postProcessProperties()`，根据 Class B name 调用 doGetBean 方法，获取 Class B 的实例。
  3. 然后在 Class B 也被实例化（一个空壳），实例化后也立马放入三级缓存中。
  4. 然后在 populateBean 填充 Class B 实例时，回调 @AutoWired 的回调接口`AutowiredAnnotationBeanPostProcessor#postProcessProperties()`，根据 Class A name 调用 doGetBean 方法，获取 Class A 的实例。
  5. 由于此时 A 已经在三级缓存中，所以取出 A 的 ObjectFactory 表达式并执行，获取到 Class A 实例的引用。
     - 其中要注意的是，这个表达式调用的 `getEarlyBeanReference(beanName, mbd, bean)` 方法，可以获取 AOP 代理的空壳引用，即该方法要么返回的是原对象，要么返回的是代理对象，如果返回的是代理对象，那么该代理对象 B'/A' 不会持有另外一个 A/B 的引用，而是**持有 B/A 原对象的引用**，再由原对象去持有 A/B 的引用，而 A/B 则持有代理对象的引用 B'/A'。
  6. 获取到 Class A 实例的引用，设置到二级缓存中，删除 A 三级缓存，并返回给 Class B 注入。
  7. 此时 Class B 已经解决了循环依赖 A 的问题，最后设置单例 Class B 到一级缓存中，删除 B 二三级缓存。
  8. 由于此时 B 已经在一级缓存中，所以取出 B 实例引用，返回给 Class A 注入。
  9. 此时 Class A 也解决了循环依赖 B 的问题，最后设置单例 Class A 到一级缓存中，删除 A 二三级缓存。

  => 最后，Class A、Class B 分别完成注入，也就是解决了循环依赖的问题。

### 2.3. 说说你对 AOP 的理解？

1. **AOP**，Aspect-Oriented Programming，**⾯向切⾯编程**，为解耦而生，能够将那些与业务⽆关，却为业务模块所共同调⽤的逻辑，或者责任封装起来（比如事务管理、⽇志管理、权限控制等），以便于减少系统的重复代码，降低模块间的耦合度，有利于未来的可拓展性和可维护性。

2. AOP 有如下 7 个核心的**概念**：

   1. **切面**：Aspect，指对哪些方法进行拦截处理的**横切关注点**，可能会横切多个对象，比如 Spring 的事务管理， 在 Spring AOP 中，切面可以在普通类中以 `@Aspect` 注解来实现。
   2. **连接点**：Join point，指在程序执行过程中某个特定的点，比如某个方法调用的时间点，或者处理异常的时间点，在 Spring AOP 中，一个连接点代表一个**方法的执行**。
   3. **通知**：Advice，在切面的某个特定的连接点上**执行的动作**，通知有多种类型，包括 `around`， `before`， 和 `after` 等等。
   4. **切点**：Pointcut，**匹配连接点的断言**，通知会在满足这个切点的连接点上运行，比如 AOP 去执行某个特定名称的方法。
   5. **目标对象**：Target object，被一个或者多个切面所通知的对象，也被称作**被通知的业务对象**。
   6. **织入**：Weaving，把切面连接到目标对象上，并创建一个代理对象的过程。
   7. **AOP 代理**：AOP proxy，AOP 框架创建的对象，用来实现切面契约，包括通知方法执行等功能，在Spring 中，AOP代理可以是 JDK 动态代理，或者是 CGLIB 动态代理。

3. 比如日志管理的公共代码，可以抽象出一个**切面**，然后注入到**目标对象**中（具体的业务对象），通过**动态代理**，将对目标对象进行代理，在进行调用时，代理对象会根据通知类型，在对应的时间点，执行切面中增强的方法，从而实现日志统一管理，避免了代码的冗余。

4. **原理**：Spring AOP 是基于**动态代理**实现的，是 IoC 的一个扩展功能，是在 IoC 整个流程中新增的一个 BeanPostProcessor `AbstractAutoProxyCreator` 扩展点而已。

   1.  `AbstractAutoProxyCreator` 实现了 `postProcessAfterInitialization(bean,beanName)` 方法，底层调用动态代理过程。
   2. 如果要代理的对象实现了某个接⼝，那么 Spring AOP 会使⽤ JDK Proxy，去创建代理对象。
   3. ⽽对于没有实现接⼝的对象，就⽆法使⽤ JDK Proxy 去进⾏代理了，此时 Spring AOP 则会使⽤基于 asm框架字节流的 Cglib 动态代理 ，⽣成⼀个被代理对象的⼦类来作为代理。

5. **实现**：选讲，只是用于记忆而已。

   - **1、POM 依赖**：

     ```xml
     <dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter-aop</artifactId>
     </dependency>
     ```

   - **2、开启 AOP**：

     ```java
     // proxyTargetClass默认为false, 表示默认使用JDK 动态代理, 碰到接口或者设置为true, 则使用CGLIB动态代理
     @EnableAspectJAutoProxy(proxyTargetClass = true)
     @SpringBootApplication
     public class SpringbootDemoApplication {
         public static void main(String[] args) {
             SpringApplication.run(SpringbootDemoApplication.class, args);
         }
     }
     ```

   - **3、配置切面、切点、通知**：

     ```java
     @Aspect
     @Component
     public class AspectJConfig {
         @Pointcut("execution(* com.jsonyao.cs.controller.*.*(..))")
         private void pointcut() {
     
         }
     
         @Around("pointcut()")
         public Object around(ProceedingJoinPoint point) throws Throwable {
             long start = System.currentTimeMillis();
             Object res = point.proceed();
             long end = System.currentTimeMillis();
             System.err.println("执行结果: " + res + ", 消耗时间: " + (end - start));
             return res;
         }
     }
     ```

   - **4、测试切面**：

     ```java
     package com.jsonyao.cs.controller;
     
     @RestController
     @RequestMapping("/boot")
     public class UserController {
         @Autowired
         private UserSerivce userService;
     	
         // 打印了日志：执行结果: User{id=0, username='AOP', password='AOP'}, 消耗时间: 8
         @RequestMapping("/getUser/{id}")
         public String GetUser(@PathVariable Long id){
             return userService.findById(id).toString();
         }
     }
     ```

### 2.4. Spring 事务传播属性？

- **概念**：

| 属性                     | 含义                                                         |
| ------------------------ | ------------------------------------------------------------ |
| REQUIRED（默认属性）     | 如果存在一个事务，则支持**当前事务**，如果没有事务，则开启一个**新的事务** |
| MANDATORY                | 支持**当前事务**，如果当前没有事务，就**抛出异常**           |
| NEVER                    | 以**非事务**方式执行，如果当前存在事务，则**抛出异常**       |
| NOT_SUPPORTED            | 以**非事务**方式执行操作，如果当前存在事务，就把当前事务**挂起** |
| REQUIRES_NEW（相互独立） | **新建事务**，如果当前存在事务，把当前**事务挂起**，内外两个事务相互独立，互不影响，当外层事务失败时，并不会回滚内层事务所做的动作，而内层事务操作失败时，也不会引起外层事务的回滚 |
| SUPPORTS                 | 支持**当前事务**，如果当前没有事务，就以**非事务**方式执行   |
| NESTED（局部回滚）       | 支持**当前事务**，新增 Savepoint 点，与当前事务**同步提交或回滚**。嵌套事务一个非常重要的概念，就是内层事务依赖于外层事务，当外层事务失败时，会回滚内层事务所做的动作，而内层事务操作失败时，并不会引起外层事务的回滚 |

- **实现**：@Transactional

  ```java
  @EnableTransactionManagement 
  @Transactional
  ```

- **原理**：

  ![1645359975955](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1645359975955.png)

  Spring 的事务是由 AOP 来实现的，首先按照 AOP 的整套流程来执行具体的操作逻辑，使用 `InfrastructureAdvisorAutoProxyCreator` （AbstractAutoProxyCreator 的子类）来生成具体的代理对象，然后通过一个 `TransactionInterceptor` 来实现，在调用 `JdkDynamicAopProxy#invoke(proxy, method, args)` 方法时，则代理到 `TransactionInterceptor`  实现的具体逻辑中。

  1. 先做准备工作，解析各个方法上事务相关的属性，根据具体的属性来判断是否开始新事务。
  2. 当需要开启事务时，则获取数据库连接，关闭自动提交功能，开启事务。
  3. 执行具体的业务逻辑。
  4. 在执行过程中发生异常，那么会通过 `completeTransactionAfterThrowing` 来完成事务的回滚操作，回滚的具体逻辑是通过 `doRollBack` 方法来实现的，实现的时候也是要先获取链接对象，再通过连接对象来回滚。
  5. 如果执行过程中，没有任何意外情况的发生，那么通过 `commitTransactionAfterReturning` 来完成事务的提交操作，提交的具体逻辑是通过 `doCommit` 方法来实现的，实现的时候也要获取链接，通过链接对象来提交。
  6. 当事务执行完毕之后，需要通过 `cleanupTransactionInfo` 来清除相关的事务信息。 

- **注意事项**：

  1. 事务函数中不要处理**耗时任务**，会导致长期占有数据库连接。
  2. 事务函数中不要处理**无关业务**，防止产生异常导致事务回滚。
  3. Spring 事务控制什么时候会**失效**？
     1. Bean 对象没有被 Spring 容器所管理。
     2. 调用方法的访问修饰符不是 public。
     3. 数据源没有配置事务管理器。
     4. 数据库不支持事务。
     5. 异常被捕获，所以没有回滚。

### 2.5. Spring 中的设计模式？

- **单例模式** : Bean 默认都是单例的。
- **⼯⼚模式** : 使用 BeanFactory、ApplicationContext 来创建 Bean 对象。
- **模板方法模式**：BeanFactoryPostProcessor#postProcessBeanFactory，AbstractApplicationContext#onRefresh。
- **代理模式** ：AOP。
- **观察者模式**：事件驱动模型。
- **责任链模式**：MVC Filter。
- **适配器模式**：MVC 适配器适配 Controller 。

### 2.6. 谈谈你对 SpringBoot 的理解？

- **背景**：SpringBoot 所解决了的问题。
  1. 搭建后端框架时需要涉及很多 **XML 配置文件**，增加了搭建难度和时间成本。
  2. 将项目编译成 war 包，部署到 Tomcat 中，项目**部署依赖 Tomcat**，这样非常不方便。
  3. **应用监控**需要做的比较简单，通过一个没有任何逻辑的接口，来判断应用的存活状态。
- **概念**：
  1. 使用 Spring Boot 通过简单的步骤，就可以创建一个 Spring 应用。
  2. Spring Boot 为 Spring 整合第三方框架提供了**开箱即用**的功能。
  3. Spring Boot 的核心思想是**约定大于配置**。
- **优点**：
  1. **自动装配**：Spring Boot 会根据某些规则对所有配置的 Bean 进行初始化，减少了很多重复性的工作。
     - 比如使用 MongoDB 时，只需加入 MongoDB 的 Starter 包，然后配置  的连接信息，就可以直接使用 MongoTemplate 自动装配来操作数据库了。简化了 Maven Jar 包的依赖，降低了烦琐配置的出错几率。
  2. **内嵌容器：**Spring Boot 应用程序可以不用部署到外部容器中，比如 Tomcat，应用程序可以直接通过 Maven 命令编译成可执行的 jar 包，通过 java-jar 命令启动即可，非常方便。
  3. **应用监控：**Spring Boot 中自带监控功能 Actuator，可以实现对程序内部运行情况进行监控，比如 Bean 加载情况、环境变量、日志信息、线程信息等，也可以自定义跟业务相关的监控，通过Actuator 的端点信息进行暴露。

### 2.7. SpringBoot 自动装配原理？

- **Starter 是什么**：
  1. starter 就是一个 jar 包，写一个 @Configuration 的配置类，把这些 Bean 的定义都包含在其中，然后在 Starter 包下的 `META-INF/spring.factories` 中写入该配置类，那么 SpringBoot 程序在启动时，就会按照约定来加载该配置类。
  2. 开发人员只需要将相应的 Starter 包依赖进应用中，然后进行相关的属性配置，就可以进行代码开发，而不需要再单独对 Bean 进行配置。
- **自定义 Starter**：
  1. 创建 Starter 项目，定义 Starter 需要的 Properties 配置类，比如数据库连接信息等。
  2. 然后编写自动配置类，自动配置类就是获取配置，根据配置来自动装配 Bean。
  3. 编写 `META-INF/spring.factories` 文件，以让 SpringBoot 在启动时加载自动配置类。
  4. 然后在项目中，引入自定义 Starter 的 Maven 依赖，增加对应的配置值后，即可直接使用。
- **原理**：
  1. 当启动 SpringBoot 应用时，会先创建 SpringApplication 的对象，在对象的构造方法中，会进行某些参数的初始化工作，最主要的是判断当前应用的类型（比如 Servlet） 以及 SPI 加载整个应用的 `spring.factories` 文件中的初始化器和监听器 Class 类。
  2. SpringApplication 对象创建完成之后，开始执行 `run()` 方法，来完成整个启动，启动过程中最主要的有两个方法，第一个叫做 `prepareContext()`，第二个叫做 `refreshContext()`，在这两个步骤中完整了自动装配的核心功能，而其他方法的处理逻辑包含了上下文对象的创建、Banner 的打印、异常报告期的准备等各个准备工作，方便后续来进行调用。
  3. 在 `prepareContext()` 方法中，主要完成了对上下文对象的初始化工作，包括属性值的设置，比如环境对象，在整个过程中，有一个非常重要的方法 `load()`，`load()` 主要完成一件事，那就是将**启动类**作为BeanDefinition 注册到 Registry 中，方便后续在进行 BeanFactoryPostProcessor 调用执行时，找到对应的主类来完成 `@SpringBootApplication` 和 `@EnableAutoConfiguration` 等注解的解析工作。
  4. 在 `refreshContext()` 方法中，会进行整个容器的刷新过程，会调用 Spring 中的启动流程，即`AbstractApplicationContext#refresh()`，有 13 个关键方法，来完成整个  Spring 应用的启动，其中会调用 `invokeBeanFactoryPostProcessor()` 方法，主要是对`ConfigurationClassPostProcessor` 的处理，会先调用实现 `BeanDefinitionRegistryPostProcessor` 接口的 `postProcessBeanDefinitionRegistry()` 方法，然后再调用自己实现的 `postProcessBeanFactory()` 方法，处理各种包括 @PropertySource、@ComponentScan、@Import、@ImportResource、@Bean 等注解。
  5. 其中，在解析 @Import 注解时，会有一个 `getImports()` 的方法，会从**启动类**开始递归解析注解，把所有包含 @Import 注解都收集到，然后在 `processImport()` 方法中，对 Import 导入的类进行分类，这里主要起识别作用的是 `ImportSelect` 的实现类  `AutoConfigurationImportSelect`，来调用 `selectImports()` 方法使用 SPI 方式，来获取并加载 `spring.factories` 中的 `EnableAutoConfiguration` 自动装配配置类的 Class。
  6. 接着，调用子类上下文 `ServletWebServerApplicationContext#onRefresh` 方法，来拉起嵌入式 Tomcat 服务器。
  7. 最后，还有一步关键步骤是 `finishBeanFactoryInitialization()` 方法，主要是实例化所有剩下非懒加载的单例 Bean，并完成它们的初始化和依赖注入，通过遍历所有 beanNames，然后挨个判断走FactoryBean 的流程，还是走 BeanFactory 的流程，其中主要步骤总结起来分为 3 步，分别是NewInstance 实例化、Populate 属性赋值 和 Initialization 初始化，从而完成自动配置配置 Bean 的自动注入。

### 2.8. Spring MVC 原理？

- **自动装配原理**：

  - ../spring-boot-**autoconfigure**-2.2.2.RELEASE.jar!/META-INF/**spring.factories**：

    ```properties
    # Auto Configure
    org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
    ...,
    org.springframework.boot.autoconfigure.web.servlet.DispatcherServletAutoConfiguration,\
    ...
    ```

  - **DispatcherServletAutoConfiguration**：Springboot 中 的Spring MVC，主要是依靠DispatcherServletAutoConfiguration 中的两个内部类进行初始化，两者配合，最终完成初始化工作：

    ```java
    @AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE)
    @Configuration(proxyBeanMethods = false)
    @ConditionalOnWebApplication(type = Type.SERVLET)
    @ConditionalOnClass(DispatcherServlet.class)
    @AutoConfigureAfter(ServletWebServerFactoryAutoConfiguration.class)
    public class DispatcherServletAutoConfiguration {
        
        // 1、DispatcherServletConfiguration，负责生成dispatcherServlet
    	@Configuration(proxyBeanMethods = false)
    	@Conditional(DefaultDispatcherServletCondition.class)
    	@ConditionalOnClass(ServletRegistration.class)
    	@EnableConfigurationProperties({ HttpProperties.class, WebMvcProperties.class })
        protected static class DispatcherServletConfiguration {
            @Bean(name = DEFAULT_DISPATCHER_SERVLET_BEAN_NAME)
            public DispatcherServlet dispatcherServlet(HttpProperties httpProperties, WebMvcProperties webMvcProperties) {
                ...
            }
            ...
        }
        
        // 2、DispatcherServletRegistrationConfiguration，负责将dispatcherServlet注册到系统里面的servlet中，使其生效
        @Configuration(proxyBeanMethods = false)
    	@Conditional(DispatcherServletRegistrationCondition.class)
    	@ConditionalOnClass(ServletRegistration.class)
    	@EnableConfigurationProperties(WebMvcProperties.class)
    	@Import(DispatcherServletConfiguration.class)
        protected static class DispatcherServletRegistrationConfiguration {
            ...
        }
    }
    ```

- **请求访问原理**：

![1645420150788](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1645420150788.png)

总：当客户端发起请求时，被中心控制器拦截到请求，根据请求参数生成代理请求，找到对应的实际控制器，调用控制器去处理请求，创建数据模型，访问数据库填充好模型后，再把模型视图返回给适配器到中心控制器，然后由中心控制器去调用视图解析器，根据逻辑的视图获取实际的视图，再使用模型去渲染该视图后，最后再把视图返回给客户端，从而完成一次请求的处理。

分：

1. DispatcherServlet，表示中心控制器，在客户端发出请求后，经过 Web 容器（比如 Tomcat）后，会打到 DispatcherServlet 上，并由其来处理请求。
2. HandlerMapping，表示处理器映射，DispatcherServlet 收到请求后，会调用 HandlerMapping，HandlerMapping 会根据请求 url 去查找对应的 Handler，即一个 Handler Method 对象，指的是 url 对应 Controller 中的对应方法。
3. HandlerExecutionChain，表示处理器执行链，Handler 解析完 url  后，会返回一个处理器执行链给DispatcherServlet。
4. HandlerAdapter，表示处理器适配器，DispatcherServlet 会按照规则去匹配对应的 HandlerAdapter。
5. 再由对应的 Handler Method 去处理请求，其中就包括调用去执行我们编写的业务逻辑。
6. Controller 代理对象，则会去创建数据模型，访问数据库填充好模型后，再把 ModelAndView 返回给 HandlerAdapter。
   - 对于我们平常使用的 @ResponseBoday 的方法，返回的 ModelAndView 为空，也就是不会返回视图给客户端，而是经过 `RequestResponseBodyMethodProcessor`，把 JSON 串写入到 Response 的 Body 中。
7. HandlerAdapter 收到后，再将 ModelAndView  传递给 DispatcherServlet。
8. DispatcherServlet 收到后，则调用视图解析器 ViewResolver，来解析 ModelAndView。
9. ViewResolver 会解析逻辑视图名，根据逻辑的 View 找到实例的 View，并返回给 DispatcherServlet。
10. DispatcherServlet 收到后，则根据 ViewResolver 解析出的 View，调用对应的实际视图，结合 Model 进行渲染。
11. 最后，DispatcherServlet 再将渲染后的 View 作为结果，响应给客户端。

- **MVC 9 大组件**：
  1. **HandlerMapping**：处理器映射，RequestMappingHandlerMapping 可以根据 request#url，找到 **Handler Method** 对象，这指的是 url 对应 Controller 中的对应方法。
  2. **HandlerAdapter**：调用 Handler Method 的适配器，主要处理方法参数、相关注解、数据绑定、消息转换、返回值、调用视图解析器等工作。
  3. HandlerExceptionResolver：异常解析器，对异常进行处理。
  4. **ViewResolver**：视图解析器，用来将 String 类型的视图名和 Locale 解析为 View 类型的逻辑视图。
  5. RequestToViewNameTranslator：请求视图名获取器，有的 Handler Method 处理完后没有设置返回类型，比如是 void 返回值的方法，就需要从 request 中获取 viewName。
  6. LocaleResolver：多语言解析器，从 request 中解析出请求中的本地语言 Locale，Locale表示一个区域，比如 zh-cn，针对不同的区域的用户，显示不同的结果，即 i18n（SpringMVC 中有具体的拦截器`LocaleChangeInterceptor`）。
  7. ThemeResolver：主题解析器，主题解析，这种类似于我们手机更换主题，不同的 UI，css 等。
  8. MultipartResolver：多部件文件解析器，处理上传请求，把普通的request封装成MultipartHttpServletRequest。
  9. FlashMapManager：FlashMap 管理器，用于管理 FlashMap，FlashMap 可以用于在 redirect 重定向中传递参数。

### 2.9. Spring、Spring MVC、Spring Boot 的区别是什么？

1. Spring，是一个一站式的轻量级 Java 开发框架，核心是控制反转（IOC）和面向切面（AOP），针对于开发的
  WEB 层(Spring Mvc)、业务层(IoC)、持久层(JdbcTemplate)等都提供了多种配置解决方案。
2. Spring MVC，是 Spring 基础之上的一个 MVC 框架，主要处理 WEB 开发的路径映射和视图渲染，属于 Spring 框架中 WEB 层开发的一部分。
3. SpringBoot，由于 Spring 配置非常复杂，各种 XML、JavaConfig、Servlet 处理起来比较繁琐，为了简化开发者的使用，从而创造性地推出了 SpringBoot 脚手架，相对于 Spring MVC 框架来说，更专注于开发微服务后台接口，不开发前端视图，同时遵循默认优于配置，简化了插件配置流程，不需要配置 XML，大大简化了配置流程。

### 3.0. Spring 常用注解？

| 注解           | 包位置           | 作用                                                         | 用法举例                                                     |
| -------------- | ---------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| @Autowired     | spring-beans     | 属性注入，先按属性类型去找，再按属性名字去找、方法注入、方法参数注入、构造方法注入 | @Autowired + 属性、@Autowired + setxxx()、@Autowired + 构造方法、@Autowired + 方法参数，原理见 AutowiredAnnotationBeanPostProcessor |
| @Resource      | javax.annotation | 属性注入，先按属性名字去找，再按属性类型去找，如果指定名字则直接按名字去找 | @Resource、@Resource(name = "beanName")                      |
| @Value         | spring-beans     | 注入普通字符串、占位符替换、SpringEL                         | @Value（"abc"）、@Value（"${str}"）、@Value（"#{beanName}"） |
| @Lazy          | spring-context   | 懒加载式注入代理 Bean、解决构造函数循环依赖问题              | @Lazy + 类、@Lazy + 属性、@Lazy + 方法、@Lazy + 方法参数、@Lazy + 构造方法 |
| @Lookup        | spring-beans     | 找到对应的 Bean 作为返回值返回                               | @Lookup("beanName") + 方法                                   |
| @Bean          | spring-context   | Bean 注入                                                    | @Configuration + @Bean 配置 Bean、@Component + @Bean 普通 Bean |
| @Component     | spring-context   | Bean 注入                                                    | @Component + 类，延伸出 @Configuration、@Service、@Controller、@Repository |
| @Primary       | spring-context   | 标识某个 Bean 为主 Bean，优先注入                            | @Bean + @Primary、@Component + @Primary                      |
| @Configuration | spring-context   | 标识为配置 Bean                                              | @Configuration + @Bean，配置 Bean 不仅会被注入，还会被解析   |
| @ComponentScan | spring-context   | @Component Bean 扫描                                         | @Component("beanPkg") + includeFilters、excludeFilters、META-INF/spring-components 索引 |
| @Conditional   | spring-context   | 条件式注入                                                   | @Conditional + 类、@Conditional + 方法、实现 Condition#matches 接口方法，返回 true 则代表匹配 |
| @Import        | spring-context   | 批量导入 Bean 类从而注入 Bean                                | @Import（BeanClass） + ImportSelector#selectImports、或者 DeferredImportSelector#selectImports、或者 ImportBeanDefinitionRegistrar#registerBeanDefinitions |

### 3.1. Spring 注入一个 Bean 的方式？

1. **@Component（@Configuration、@Service、@Controller、@Repository）**：通过注入某个类成为 Bean。
2. **@Bean**：通过解析某个方法成为 Bean。
3. **@Import**：导入类或者 BeanDefinition 成为 Bean。
4. **@ImportResource**：导入一个 Spring.xml 文件，通过解析文件注册 Bean。
5. **BeanDefinitionRegistryPostProcessor**：通过 BeanDefinition 注册 Bean。
6. **FactoryBean、SmartFactoryBean**：将自己 New 的一个对象注册为 Bean。
7. **ApplicationContext#registerBean**：通过 ApplicationContext 实现 Supplier 接口，来提供一个对象成为 Bean。
8. **ApplicationContext#register**：通过 ApplicationContext 某个类注册成为 Bean。
9. **ApplicationContext#registerBeanDefinition**：通过 BeanDefinition 注册 Bean。

### 3.2. Spring 依赖注入方式？

1. **@Autowired**：属性注入，先按属性类型去找，再按属性名字去找、方法注入、方法参数注入、构造方法注入。
2. **@Resource**：属性注入，先按属性名字去找，再按属性类型去找，如果指定名字则直接按名字去找。
3. **@Value**：注入普通字符串、占位符替换、SpringEL。
4. **自定义注解**：实现 BeanPostProcessor#postProcessBeforeInitialization 方法，解析 Bean 上的自定义注解，完成属性注入。

### 3.3. ApplicationContext 获取方式？

1. **@Autowired**：属性注入，先按属性类型去找，再按属性名字去找、方法注入、方法参数注入、构造方法注入。
2. **实现自省接口**：实现 ApplicationContextAware#setApplicationContext 方法。

### 3.4. Spring AOP 使用方式？

1. **ProxyFactory**：代理对象工厂，封装了 JDK 和 CGLIB 动态代理方法，比如：

   ![1647155729524](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1647155729524.png)

2. **ProxyFactoryBean**：利用 FactoryBean 机制，将代理对象作为一个 Bean，比如：

   ![1647155822496](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1647155822496.png)

3. **BeanNameAutoProxyCreator**：指定某个 beanName，让 Spring 对其匹配的 Bean，进行批量 AOP，比如：

   ![1647155851621](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1647155851621.png)

4. **DefaultAdvisorAutoProxyCreator**：指定某个 Advisor，让 Spring 对其匹配的 Bean，进行批量 AOP。

   ![1647155943409](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1647155943409.png)

5. **@EnableAspectJAutoProxy**：开启支持 AspectJ。

### 3.5. Spring MVC 常用注解？

| 注解            | 包位置     | 作用                                           | 用法举例                                                     |
| --------------- | ---------- | ---------------------------------------------- | ------------------------------------------------------------ |
| @RequestMapping | spring-web | 映射Web请求                                    | @RequestMapping（"path"）+ 类、@RequestMapping（"path"）+ 方法 |
| @RequestBody    | spring-web | 读取请求体中的参数                             | @RequestBody + 方法参数                                      |
| @PathVariable   | spring-web | 读取请求路径中的参数                           | @PathVariable + 方法参数                                     |
| @ResponseBody   | spring-web | 把 json 返回值放在 response 内，而不是一个页面 | @ResponseBody + 类 = @RestController、@ResponseBody + 方法   |

### 3.6. Spring Boot 常用注解？

| 注解                   | 包位置                    | 作用                                                         | 用法举例                    |
| ---------------------- | ------------------------- | ------------------------------------------------------------ | --------------------------- |
| @SpringBootApplication | spring-boot-autoconfigure | @SpringBootConfiguration 表示为配置 Bean + @EnableAutoConfiguration 开启自动装配 `META-INF/spring.factories` + @ComponentScan 扫描注解 | @SpringBootApplication + 类 |

