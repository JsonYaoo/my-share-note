# 二、JVM篇 

### 1.1. JDK、JRE、JVM的区别？

![1625884268478](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1625884268478.png)

从图中可以看出，**JDK包含了JRE，而JRE又包含了JVM**。

- **JDK**，Java Development Kit，是Java的软件开发工具包（SDK），包含JRE和Java工具。
- **JRE**，Java Runtime Environment，是Java的运行时环境，大部分都是C和C++语言编写的，可以在其上运行、测试应用程序的Java平台，包括JVM和Java核心类库。
- **JVM**，Java Virtual Machine，Java虚拟机，是一种用于计算设备的规范，是一个虚构出来的计算机，是通过在实际的计算机上仿真模拟各种计算机功能来实现的，屏蔽了与具体平台相关的信息，使得Java语言编译程序只需要在Java虚拟机上运行的字节码，就可以不加修改地在多种平台上运行。

### 1.2.  JVM整体架构？

![1625963806257](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1625963806257.png)

- JVM包含**2个子系统和2个组件**：2个子系统分别为Class Loader（类加载子系统）、Execution Engine（执行引擎）；2个组件分为别Runtime data area（运行时数据区）、Native Interface（本地接口）。
  - **Class loader（类加载子系统）**：根据给定的全限定名类名（如java.lang.Object）来装载class文件到Runtime data area中的Method Area（方法区）。
  - **Runtime data area（运行时数据区域）**：这就是我们常说的JVM的内存。
  - **Execution engine（执行引擎）**：执行class文件中的指令。
  - **Native Interface（本地接口）**：与native libraries交互，是其它编程语言交互的接口。
- 架构整体流程：

1. 通过编译器把 Java 代码转换成字节码，**类加载器（ClassLoader）**再把字节码加载到内存中，将其放在**运行时数据区（Runtime data area）**的方法区内。
2. 而字节码文件只是 JVM 的一套指令集规范，并不能直接交给底层操作系统去执行，因此需要特定的命令解析器**执行引擎（Execution Engine）**，将字节码翻译成底层系统指令，再交由 CPU 去执行。
3. 而这个过程中需要调用其他语言的**本地接口（Native Interface）**来实现整个程序的功能。

### 1.3. 详细介绍类加载机制？

![1625975271139](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1625975271139.png)

程序主动使用某个类时，如果这个类还未被加载到内存中，则JVM会通过**加载、链接、初始化**3个步骤来对该类进行初始化。如果没有意外，JVM将会连续完成3个步骤，所以有时也把这个3个步骤统称为**类加载或类初始化**。

1. **加载**：指的是将类的**class文件（二进制数据）**读入到内存，并转换成**方法区中的运行时数据结构**。同时在堆中生成一个代表这个类的**java.lang.Class对象**，该对象封装了类在方法区中的数据结构，并且向用户提供了访问方法区数据结构的接口，即Java反射的接口。
   - 加载过程需要**类加载器**参与。类加载器，可以从不同来源加载类的二进制数据，比如：本地Class文件、Jar包Class文件、网络Class文件等等。
   - Java类加载器由JVM提供，是所有程序运行的基础，JVM提供的这些类加载器通常被称为系统类加载器。
   - 除此之外，开发者可以通过继承ClassLoader基类来创建自己的类加载器。
   - Java的类加载是动态的，不会一次性将所有类全部加载后再运行，而是保证程序运行的基础类完全加载到JVM中。至于其他类，则**在需要的时候才加载**（为了节省内存开销）。
     - **隐式加载**：程序在运行过程中，当碰到通过new 方式生成对象时，将会隐式调用类装载器，加载对应的类到JVM中。
     - **显式加载**：通过class.forname（）反射方法，显式加载需要的类。
2. **链接**：该阶段负责把**类的二进制数据合并到JRE中**，可分为如下3个阶段：
   - **验证**：验证Class文件是否符合规范，是否能被当前的虚拟机加载处理，确保加载的类没有安全方面的问题。
     - 文件验证：是否以0xCAFEBABE开头、版本号是否合理等。
     - 元数据验证：是否有父类、是否继承了final类、非抽象类是否实现了所有抽象方法等。
     - 字节码验证：运行检查、栈数据类型和操作码的操作参数是否吻合（不能大于栈空间）、跳转指令是否指向合理的位置。
     - 符号引用验证：常量池中描述的类是否存在、访问的方法或字段是否存在且有足够的权限。
     - 可使用**-Xverify:none**关闭验证：比如提高IDEA的启动速度。
   - **准备**：为类的静态变量（static）分配内存，并初始化为初始值（0或null）。而对于静态常量（final static修饰）会直接被赋值为用户定义的值。
   - **解析**：将Class常量池（Constant Pool）的符号引用转换为直接引用。
   - 实际上，JVM不一定完全按照类加载机制顺序执行，比如解析操作有可能会发生在初始化操作之后。
3. **初始化**：类初始化是类加载的最后一步，真正执行Java代码，主要工作是为静态变量（static）赋值为用户定义的值。初始化完毕类就可以被使用了。
   - 执行< clinit >方法，clinit方法由编译器自动收集类里面的**所有静态变量的赋值动作及静态语句**合并而成，也叫**类构造器方法**。
     - 初始化的顺序和源文件中的顺序一致。
     - 子类的< clinit >被调用前，会先调用父类的< clinit >。
     - JVM会保证clinit方法的线程安全性。
   - 即执行顺序为：JVMTest5静态块 -> super静态块 -> Sub静态块 -> Super构造块 -> Super构造方法 -> Sub构造块 -> Sub构造方法。
     - 类初始化后，如果是实例化一个新对象，还会调用< init >方法，与< clinit >类似，< init >方法可以看作是**对象构造方法**，是由编译器自动收集类中所有实例变量的赋值动作、实例代码块和构造函数合并而成的。
     - 如果是对实例变量直接赋值或者使用实例代码块赋值，那么编译器会将这些代码合并到实例构造函数中去，并且它们还会被放在对父类构造函数的调用语句之后（因为Java要求构造函数的第一条语句必须是父类构造函数的调用语句)，自身构造函数的代码之前去执行。
     - 因此，类构造器和对象构造器的初始化过程为：**父类的类构造器 -> 子类的类构造器 -> 父类成员变量的赋值和实例代码块 -> 父类的构造函数 -> 子类成员变量的赋值和实例代码块 -> 子类的构造函数。**

```java
// JVMTest5不用被实例化，所以不会调用JVMTest5的构造块和构造方法
public class JVMTest5 {
    static {
        System.out.println("JVMTest5静态块");
    }

    {
        System.out.println("JVMTest5构造块");
    }

    public JVMTest5() {
        System.out.println("JVMTest5构造方法");
    }

    public static void main(String[] args) {
        new Sub();
    }
}

class Super {
    static {
        System.out.println("Super静态块");
    }

    public Super() {
        System.out.println("Super构造方法");
    }

    {
        System.out.println("Super构造块");
    }
}

class Sub extends Super {
    static {
        System.out.println("Sub静态块");
    }

    public Sub() {
        System.out.println("Sub构造方法");
    }

    {
        System.out.println("Sub构造块");
    }
}
```

### 1.4. 什么是类加载器？类加载器有哪些？

![1625986988156](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1625986988156.png)

**类加载器**，是能够实现通过类的全限定名，获取该类的二进制字节流的**代码块**。JVM提供了3种类加载器，启动类加载器、扩展类加载器、系统类加载器（也叫应用类加载器），以及用户自定义的类加载器（其父类为应用类加载器）。

- **启动类加载器**：Bootstrap ClassLoader，该类加载使用C++实现，其引用为null，无法被java程序直接引用。用于加载Java核心类库，即负责把**/lib**目录下或者**-Xbootclasspath**参数指定路径下的Jar包加载到内存中。
  - 注意，JVM是按照文件名识别加载Jar包，如rt.jar，如果文件名不被虚拟机识别，即使把Jar包丢到lib目录下也是没有作用的。
  - 处于安全考虑，启动类加载器只能加载包名为java、javax、sun等开头的类。
- **扩展类加载器**：ExtClassLoader，由Java实现，父类加载器为启动类加载器（持有为null的parent引用，并不是真正的继承关系）。用于加载 Java 的扩展库，即负责把**/lib/ext**目录下或者**-Djava.ext.dir**参数指定路径下的类库加载到内存中。
  - 开发者可以直接使用标准的扩展类加载器。
- **系统类加载器**：AppClassLoader，也叫应用类加载器，由Java实现，父类加载器为ExtClassLoader（持有ExtClassLoader的parent引用，并不是真正的继承关系）。用于加载一般的Java 应用类，即负责把**java -classpath**或者**-D java.class.path**指定路径下的类库加载到内存中。
  - 一般情况下，系统类加载器是程序中默认的类加载器。
  - 开发者可以直接使用应用类加载器，可以通过**ClassLoader.getSystemClassLoader（）**来获取。
- **用户自定义的类加载器**：用户可以通过继承 java.lang.ClassLoader类的方式，来自定义自己的加载器。
  - 应用场景：
    - 加密编译后的class字节码 ->  自定义ClassLoader -> 加载该class时解密字节码。
    - 自定义ClassLoader，加载时从非标准来源加载字节码：比如数据库、网络上。

### 1.5. 什么是双亲委派机制？

#### 概念

加载器之间存在着"父子关系"（区别于Java里的继承），子加载器保存着父加载器的引用。

1. 当一个类加载器需要加载一个目标类时，先会去缓存中查找，如果找到，则解析或者返回。
2. 如果缓存中找不到，则委托给父加载器加载，父加载器会在自己的加载路径中搜索目标类，如果找到，则解析或者返回。
3. 如果找不到，才会交还子加载器加载目标类，查找逻辑交由子加载器实现。

#### 实现原理

- **java.lang.ClassLoader**：扩展类、系统类以及自定义的加载器都继承这个类，需要实现findClass方法。
- **loadClass（String，boolean）**：类加载方法，子类在查询缓存中没有加载该Class后，会调用该方法，走双亲委派机制去查找。
- **findClass（String）**：加载器自身去加载Class的方法，交由子类去实现。比如子类URLClassLoader（ExtClassLoader和AppClassLoader的父类），根据URL找到对应的Class文件后，会调用**defineClass（String，Resource）**方法生成Class对象。
- **resolveClass（Class<?>）**： 底层调用native方法，解析生成出来的Class对象，将Class常量池（Constant Pool）的符号引用转换为直接引用，且为类变量（静态变量/实例变量[在该对象实例化时]）分配内存并设置初始值。
- **defineClass（String，Resource）**：在Java堆区生成Class对象。

```java
// java.lang.ClassLoader#loadClass：扩展类、系统类以及自定义的加载器都继承这个类，需要实现findClass方法
public abstract class ClassLoader {
    
    // 类加载方法，子类在查询缓存中没有加载该Class后，会调用该方法，走双亲委派机制去查找
    protected Class<?> loadClass(String name, boolean resolve) throws ClassNotFoundException {
        synchronized (getClassLoadingLock(name)) {
            // 当一个类加载器需要加载一个目标类时，先会去缓存中查找，如果找到，则解析或者返回
            Class<?> c = findLoadedClass(name);// native方法
            if (c == null) {
                long t0 = System.nanoTime();
                try {
                    // 如果缓存中找不到，则委托给父加载器加载，父加载器会在自己的加载路径中搜索目标类，如果找到则解析或者返回
                    if (parent != null) {
                        c = parent.loadClass(name, false);
                    } 
                    // 交由启动类加载器加载
                    else {
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                    // ClassNotFoundException thrown if class not found
                    // from the non-null parent class loader
                }

                if (c == null) {
                    // If still not found, then invoke findClass in order
                    // to find the class.
                    long t1 = System.nanoTime();
                    c = findClass(name);

                    // this is the defining class loader; record the stats
                    sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                    sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                    sun.misc.PerfCounter.getFindClasses().increment();
                }
            }

            // 底层调用native方法，解析生成出来的Class对象，将Class常量池（Constant Pool）的符号引用转换为直接引用，且为类变量（静态变量/实例变量[在该对象实例化时]）分配内存并设置初始值
            if (resolve) {
                resolveClass(c);
            }

            // 返回class
            return c;
        }
    }
    protected final Class<?> findLoadedClass(String name) {
        if (!checkName(name))
            return null;
        return findLoadedClass0(name);
    }
    private native final Class<?> findLoadedClass0(String name);
    
    // 底层调用native方法，解析生成出来的Class对象，将Class常量池（Constant Pool）的符号引用转换为直接引用，且为类变量（静态变量/实例变量[在该对象实例化时]）分配内存并设置初始值
    protected final void resolveClass(Class<?> c) {
        resolveClass0(c);
    }
    private native void resolveClass0(Class<?> c);

    // 加载器自身去加载Class的方法，交由子类去实现。比如子类URLClassLoader（ExtClassLoader和AppClassLoader的父类），根据URL找到对应的Class文件后，会调用defineClass(String，Resource)方法生成Class对象
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        throw new ClassNotFoundException(name);
    }
}
```

#### 双亲委派模型好处

- 此机制保证了Java核心类库被优先加载，避免了用户编写的类动态替换Java核心类的错误，使得Java程序能够稳定运⾏。
- 同时也避免了类的重复加载，使用双亲委派模型，JVM能够根据**类的完整类名+ClassLoader实例对象**来区分不同的类。如果不使⽤双亲委派模型，⽽是每个类加载器⾃⼰加载的话，会出现⼀些问题。⽐如编写⼀个称为 java.lang.Object 类的话，在程序运⾏的时候，系统会有多个不同的Object 类，此时会出现Object类的选择问题。

#### 双亲委派模型局限

1. SPI接口，Service Provider Interface，允许第三方为其提供实现，如JDBC、JNDI等 。
2. SPI接口属于Java核心类库，由启动类加载器加载（rt.jar），而SPI的第三方代码则是作为Java应用所依赖的Jar中（Classpath下）。
3. 其中SPI接口中的代码经常需要加载具体的第三方实现类，并调用其相关方法，此时由于双亲委派模型的存在，启动类加载器无法直接加载SPI实现类，也无法反向委托给系统类加载器加载，从而让JDK SPI机制产生了问题。

#### 打破双亲委派模型

如果不想打破双亲委派模型，则只需要重写findClass方法即可；如果想打破双亲委派模型，则需要重写整个loadClass方法。

##### 线程上下文类加载器

- **背景**：由于双亲委派模型存在SPI局限，需要一种特殊的类加载器来加载第三方类库，此时线程上下文加载器是个很好的选择。
- **线程上线文类加载器**：是从JDK 1.2开始引入的，可以通过java.lang.Thread#getContextClassLoader（）和setContextClassLoader（ClassLoader）方法来获取和设置线程的上下文类加载器。如果没有手动设置，则线程将会继承父线程的上下文类加载器，默认为系统类加载器，即在线程中运行的代码可以通过此类来加载Classpath下的类和资源。
- **打破双亲委派**：从图中可以看到，启动类加载器委派线程上下文加载器，把jdbc.jar中的实现类加载到内存中以便SPI相关类使用，因此打破了双亲委派模型，使得Java类加载更加灵活。

![1625996283343](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1625996283343.png)

- **实现原理**：
  - contextClassLoader是在ClassLoader.SystemClassLoaderAction#run（）方法进行了赋值，其中构造了**java.system.class.loader**加载器实例（实际上为AppClassLoader？），并持有当前加载器的parent作为其父加载器。
  - 接着，ClassLoader.SystemClassLoaderAction#run（）方法通过Thread#setContextClassLoader（ClassLoader）设置到当前线程的实例变量中，从而使得当前Thread实例持有contextClassLoader的引用。
  - 最后，java.sql.DriverManager在调用sevice.loader（Driver.class）时，jdbc.jar中在META-INF/sevice/java.sql.Driver配置的**com.mysql.cj.jdbc.Driver**，就会在java.util.ServiceLoader#load（Class）方法调用Thread.currentThread().getContextClassLoader()时进行类加载，从而达到SPI的目的。
  - 可以看出虽然java.util.ServiceLoader是rt.jat包的核心类库，由启动类加载器加载，但通过Thread.currentThread().getContextClassLoader()确实加载到了第三方包下的com.mysql.cj.jdbc.Driver，因此线程上下文类加载器可以打破双亲委派模型。

```java
// java.sql.DriverManager，启动类加载器加载
public class DriverManager {
 	static {
        loadInitialDrivers();
        println("JDBC DriverManager initialized");
    }
    
    private static void loadInitialDrivers() {
        AccessController.doPrivileged(new PrivilegedAction<Void>() {
            public Void run() {
                // SPI加载
                ServiceLoader<Driver> loadedDrivers = ServiceLoader.load(Driver.class);
                Iterator<Driver> driversIterator = loadedDrivers.iterator();
            }
            ...
        }
    }      
}    


// java.util.ServiceLoader，启动类加载器加载
public final class ServiceLoader<S>implements Iterable<S> {  
    // SPI加载: 获取当前线程上下文类加载器进行SPI加载，从而打破了双亲委派模型
    public static <S> ServiceLoader<S> load(Class<S> service) {
        ClassLoader cl = Thread.currentThread().getContextClassLoader();
        return ServiceLoader.load(service, cl);
    }
    ...
}
```

##### Tomcat类加载机制

- **背景**：
  - a. 一个web容器可能要部署两个或者多个应用程序，不同的应用程序可能会依赖同一个第三方类库的不同版本，因此要保证每一个应用程序的类库都是**独立、相互隔离**的。
  - b. 同一个web容器中的**相同类库的相同版本**可以共享，否则会有**重复的类被加载进JVM**。
  - c. **web容器也有自己的类库**，不能和应用程序的类库混淆，基于安全考虑，需要相互隔离。
  - d. Jsp文件也是要编译成class文件的，web容器需要支持在**Jsp**文件修改后，可以实现**HostSwap（热替换）**的功能。

![1626065847576](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1626065847576.png)

- **类加载器逻辑关系**：

  Common、Catalina、Shared类加载器（本质上是URLClassLoader）分别加载/common/、/server/、/shared/路径下的Class，但在Tomcat6后已经统一合并到了/lib目录下了。

  - **CommonClassLoader**：Tomcat最基本的类加载器，加载对应路径中的Class，可**以被Tomcat容器本身以及各个webapp访问**。
  - **CatalinaClassLoader**：Tomcat容器私有的类加载器，加载对应路径中的class，**对于webapp不可见**。
  - **SharedClassLoader**：各个webapp共享的类加载器，加载对应路径中的class，**对于所有webapp可见，但对于Tomcat容器不可见**。
  - **WebappClassLoader**：各个webapp私有的类加载器，每一个webapp对应一个WebAppClassLoader实例，加载路径中的class，**只对当前webapp可见**。
  - **JasperClassLoader**：
    - 每一个Jsp文件对应一个JasperClassLoader实例，**加载范围仅仅是这个Jsp文件所编译出来的那一个Class文件**。
    - JasperClassLoad出现的目的就是为了被丢弃，当Web容器检测到Jsp文件被修改时，会替换掉目前的JasperClassLoader实例，并通过重新建立一个新的JasperClassLoader实例来实现JSP文件的HostSwap（热替换）功能。

![1626066067882](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1626066067882.png)

![1626090586657](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1626090586657.png)

- **类加载流程**：WebappClassLoaderBase#loadClass(String, boolean)流程：

1. 先查找Tomcat缓存，如果找得到，则返回Tomcat缓存中的Class对象。
2. 如果Tomcat缓存中找不到，则查找JVM缓存，如果找得到，则返回JVM缓存中的Class对象。
3. 如果JVM缓存中也找不到，则用扩展类加载器来加载（**重点！这里并没有首先使用系统类加载器，而是直接使用了扩展类加载器来加载，也就是打破了系统类加载器的双亲委派机制**），根据双亲委派机制，扩展类加载器会委派启动类加载器来加载Class，从而保证了JRE核心类库不会被重复加载。
4. 如果指定了delegateLoad（需要先委托父类加载），则**先调用父类加载器加载**（share -> common -> app -> ext -> bootstrap，这里是为了保持顺序加载机制），如果找不到**才调用本地的findClass（String）**搜索本地存储库（WEB-INF/classes -> WEB-INF/lib），找到则返回，找不到则抛出ClassNotFoundException异常。
5. 如果没有指定delegateLoad（需要先委托父类加载），则**先调用调用本地的findClass（String）**搜索本地存储库（WEB-INF/classes -> WEB-INF/lib），如果找不到**才调用父类加载器加载**（share -> common -> app -> ext -> bootstrap，这里是为了保底机制），找到则返回，找不到则抛出ClassNotFoundException异常。

![1626090700749](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1626090700749.png)

- **总结**：可以看到，Tomcat#WebappClassLoaderBase的类加载机制是**打破了双亲委派模型**的：
  - **ext -> bootstarp模型**：保证了JRE核心类库不会被重复加载，满足了背景b加载JVM共同类库的需求。
  - **ext -> webapp模型**：实现了每个web应用只加载自己的类库（WEB-INF/classes -> WEB-INF/lib），从而实现了应用间的类库隔离，满足了背景a的需求。
  - **webapp -> share -> common模型**：实现了所有web应用之间、web与Tomcat之间，能够加载相同的类库，避免指定的类库不会被重复加载，满足了背景b加载其他共同类库的需求。
  - **（不确定）catalina -> 父类加载器模型**：实现了只加载Tomcat容器自身的类库，对于webapp是看不到的（可在config/catalina.properties的server.loader中配置jar和class的路径），满足了背景c的需求。
  - **（不确定）Jsp -> webapp -> 父类加载器模型**：通过在jsp修改后卸载再生成新的Jsp类加载器，重新加载新生成的Jsp class，从而实现Jsp的HostSwap（热替换），满足了背景d的需求。

### 1.6. JVM运行时数据区？

![1626181528728](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1626181528728.png)

JVM在执行 Java 程序的过程中会把它所管理的内存区域划分为若干个不同的数据区域。

这些区域都有各自的用途，以及创建和销毁的时间，有些区域随着JVM进程的启动而存在（**线程共享**），有些区域则是依赖线程的启动和结束而建立和销毁（**非线程共享**）。

JVM所管理的内存被划分为如下几个区域：

- **程序计数器**：Program Counter Register，非线程共享，JVM当前线程所执行的**字节码的行号指示器**，字节码解析器的工作是通过改变这个计数器的值，来选取下一条需要执行的字节码指令，分支、循环、跳转、异常处理、线程恢复等基础功能，都需要依赖这个计数器来完成。
- **虚拟机栈**：Java Virtual Machine Stacks，非线程共享，每个方法在执行的同时都会在Java 虚拟机栈中创建一个栈帧（Stack Frame），用于存储**局部变量表、操作数栈、动态链接和方法返回地址**；
- **本地方法栈**：Native Method Stack，非线程共享，与虚拟机栈的作用是一样，只不过虚拟机栈是服务Java方法的，而本地方法栈是为虚拟机调用Native方法服务的。
- **堆**：Java Heap，线程共享，在JVM启动时创建，是Java虚拟机中内存最大的一块，**专门用来保存对象，几乎所有对象以及数组的内存都在堆上分配**。
- **方法区**：Methed Area，别命Non-Heap（非堆），线程共享，是JVM规范中定义的一个逻辑概念，用于存储已被虚拟机加载的**类信息、常量、静态变量和即时编译后的代码**等数据，具体放在哪里，不同的实现可能会放在不同的地方。

### 1.7. 详细介绍程序计数器？

- **概念**：程序计数器，Program Counter Register，非线程共享，JVM当前线程所执行的字节码的行号指示器，字节码解析器的工作是通过改变这个计数器的值，来选取下一条需要执行的字节码指令，分支、循环、跳转、异常处理、线程恢复等基础功能，都需要依赖这个计数器来完成。

- **出现的原因**：由于JVM多线程是通过线程轮流切换，并分配处理器执行时间的方式来实现的，一个处理器只会执行一条线程中的指令。因此，为了线程切换后能恢复到正确的执行位置，需要**记住原线程的下一条指令的位置**，所以每条线程都要有一个独立存储、互不影响的程序计数器，称之为“**线程私有**”的内存。

- **例子**：

  1. 比如线程A在看直播。
  2. 突然，线程B来了一个视频电话，就会抢夺线程A的时间片，就会打断了线程A，线程A就会挂起。
  3. 然后，视频电话结束，如果没有线程计数器，此时线程A就不知道要干什么了；如果有线程计数器，此时线程A就可以想起来要去看直播了。

  => 线程是最小的执行单位，不具备“记忆”功能，只负责去干，这就需要**由程序计数器来为线程提供保护和恢复现场**的功能。

### 1.8. 详细介绍虚拟机栈？

![1626137840755](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1626137840755.png)

Java虚拟机栈**是线程私有的（非线程共享）**，每个线程都会有自己的虚拟机栈，其生命周期与线程生命周期一致。单位是**栈帧**，在每个方法执行的时候，都会创建一个栈帧，在被调用直至执行完毕的过程，对应一个栈帧在虚拟机栈中**从入栈到出栈**的过程。

每个栈帧都存放着**局部变量表、操作数栈、动态链接和方法返回地址**。

> 在JVM规范中，对此区域规定了两种异常状况：在固定情况下，如果线程请求的栈深度大于虚拟机所允许的最大栈深度，则会抛出**StackOverflowError**异常；在可动态扩展情况下，如果虚拟机栈无法申请到足够的内存，或者在创建新线程的时候没有足够的内存去创建对应的虚拟机栈，则会抛出**OutOfMemoryError**异常。

在Hotspot虚拟机中，**栈内存是不允许扩展的**，且不区分虚拟机栈和本地方法栈，统一使用-Xss设置栈的大小，但同样会抛StackOverflowError异常，以及OutOfMemoryError异常。在有些VM中是有区分开的，比如使用-Xss设置虚拟机栈大小，-Xoss设置本地方法栈大小。

- **局部变量表**：

  - 是一组变量值的存储空间，用于存放方法参数和方法内部定义的局部变量。
  - 存放了编译期可知的各种**基本数据类型**（boolean、byte、char、short、int、float、long、double，对包装类型在栈中保存地址、在堆中保存值）、**对象引用**（Reference类型，可能是一个指向对象起始地址的引用指针，也可能是一个代表对象的句柄或者其他与对象相关的位置）和**returnAddress类型**（指向下一条字节码指令的地址）。
  - 局部变量表所需的内存空间在编译期间完成分配，方法在运行之前，该局部变量表所需要的内存空间是固定的，运行期间不会发生改变。

- **操作数栈**：

  - 用于保存计算过程中的**中间结果**，同时作为计算过程中变量临时的存储空间。
  - 操作数栈在方法的执行过程中，根据字节码指令往操作数栈中写入数据或提取数据，即入栈和出栈操作。
  - 比如add（）方法执行过程中，其操作数栈与局部变量表的交互顺序为：15入栈（操作数栈写入数据） -> 15出栈（操作数栈提取数据到局部变量表） -> 1入栈（操作数栈写入数据） -> 1出栈（操作数栈提取数据到局部变量表） -> 15入栈（加载局部变量表变量15） -> 1入栈（加载局部变量表变量1） -> iadd（执行相加15 + 1指令） -> 16出栈（操作数栈提取结果到局部变量表）-> return（如果返回值为void，则当前栈帧出栈即可，如果带有返回值，则局部变量表中的结果16，还会入栈操作数栈中）。

  ```java
  public class Test {
      public void add() {
          int a = 15;
          int b = 1;
          int c = a + b;
      }
  }
  ```

- **动态链接**：

  - 每个栈帧都包含一个指向运行时常量池中该栈帧所属方法的引用，持有这个引用是为了支持方法调用过程中的动态链接（Dynamic Linking）。
  - Class 文件的常量池中存在大量的符号引用，字节码中的方法调用指令就以常量池中指向方法的符号引用作为参数，这些符号引用一部分会在类加载阶段或**第一次使用时转化为直接引用，这种转化成为静态解析**。另一部分将在**每一次运行期间转化为直接引用，这部分称为动态连接**。

- **方法返回地址**：returnAddress类型（**指向下一条字节码指令的地址**）:

  - 当一个方法开始执行后，只有两种方式可以退出这个方法。一种是执行引擎遇到**任意一个方法返回的字节码指令**，这时候可能会有返回值传递给上层方法的调用者，是否有返回值和返回值的类型将根据遇到何种方法返回指令来决定，这种退出方法的方式称为**正常完成出口**。
  - 另一种退出方式是，在方法执行过程中遇到了异常，并且这个异常没有在方法体内得到处理，无论是 Java 虚拟机内部产生的异常，还是代码中使用 athrow 字节码指令产生的异常，只要在本方法的异常表中没有搜索到匹配的异常处理器，就会导致方法退出，这种称为**异常完成出口**。一个方法使用异常完成出口的方式退出，是不会给上层调用者产生任何返回值的。
  - 无论采用何种退出方式，在方法退出后都需要返回到方法被调用的位置，程序才能继续执行，方法返回时可能需要在栈帧中保存一些信息，用来恢复它的上层方法的执行状态。一般来说，方法正常退出时，**调用者的PC计数器的值可以作为返回地址**，栈帧中很可能会保存这个计数器值。而方法异常退出时，返回地址是要通过异常处理器表来确定的，栈帧中一般不会保存这部分信息。
  - 方法退出的过程实际上就等同于把当前栈帧出栈，因此退出时可能执行的操作有：恢复上次方法的局部变量表和操作数栈，把返回值（如果有的话）压入调用者栈帧的操作数栈中，**调整PC计数器的值以指向方法调用指令后面的一条指令等**。

- 附加信息：虚拟机规范允许具体的虚拟机实现增加一些规范里没有描述的信息到栈帧中，例如与调试相关的信息，这部分信息完全取决于具体的虚拟机实现。实际开发中，一般会把动态连接、方法返回地址与其他附加信息全部归为一类，成为栈帧信息。

### 1.9. 详细介绍本地方法栈？

- 本地方法栈，Native Method Stack，非线程共享，与虚拟机栈的作用是一样，只不过虚拟机栈是服务Java方法的，而本地方法栈是为虚拟机调用Native方法服务的。
- Native方法是看不到的，必须要去oracle官网去下载才可以看的到，而且native关键字修饰的大部分源码都是C和C++的代码。
- 在Hotspot虚拟机中，**栈内存是不允许扩展的**，且不区分虚拟机栈和本地方法栈，统一使用-Xss设置栈的大小，因此同样会抛StackOverflowError异常，以及OutOfMemoryError异常。
  - 在有些VM中是有区分开的，比如使用-Xss设置虚拟机栈大小，-Xoss设置本地方法栈大小。

### 2.0. 详细介绍堆？

![1626349372632](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1626349372632.png)

- **堆**：Java Heap，线程共享，在JVM启动时创建，是Java虚拟机中内存最大的一块，**专门用来保存对象，几乎所有对象以及数组的内存都在堆上分配**。
  - 非栈上分配情况下，**创建的对象会存储在堆内存中，在栈上存储该对象的引用**（栈空间只包含方法基础数据类型的局部变量以及引用堆对象的引用变量）。
  - ClassA a = new ClassA（）；此时a叫实例，不能说a对象，**实例在栈上，对象在堆中**，操作实例实际上是通过实例指针间接操作对象，多个实例可以指向同一个对象。
  - 栈中的数据和堆中的数据销毁并不是同步的，方法一旦结束，**栈中的局部变量会立即销毁**；堆中的对象不一定会销毁，因为可能有其他变量也指向了该对象，直到没有变量指向该对象，才有可能会被垃圾回收。
  - 类的成员变量在对象中，每个对象都有自己成员变量的存储空间；而**类的方法只有一套**，存储在方法区中，被该类的所有对象共享，对象在使用方法的时候才会将方法压入栈中，而**方法在不使用时并不会占用内存**。
- 对象在堆中分配好以后，会在栈中保存一个4字节的实例（指向对象堆内存地址），用来定位对应的对象在堆中的位置，便于找到该对象。但在开启逃逸分析后，某些未逃逸的对象也可以通过标量替换的方式在**栈上分配**。
- 堆是垃圾回收（GC）的主要场所，从内存回收角度来看，可以分为**新生代和老年代，默认内存大小比值为1：2**，对于新生代又可以分为**Eden区（伊甸园）、Survivor区（存活区），默认内存大小比值默认为8：2**，而Survivor区又分为**Surviver0（From Survivor）和Survivor1（To Survivor），默认内存大小比值默认为1：1**。
- **TLAB**：Thread Local Allocation Buffer，线程私有分配缓存区，是一块**线程专用的内存分配区域**，JVM会为每个线程分配一块TLAB区域，**实质占用的是Eden区的空间（即分配独享、使用共享）**，用于给每个线程往自己的TLAB中分配小对象，这样可以避免堆分配对象时的线程冲突，从而提升分配对象的效率。
  - **优点 - 加速对象分配**：
    - 当多个线程同时在堆上分配对象时，由于堆是线程共享的，为了保证线程同步，JVM底层采用CAS + 失败重试的方式来做同步处理，如果多线程竞争非常激烈，那么此时在堆中分配对象性能是非常差的。因此，JVM设计了TLAB，来避免堆分配对象时的线程冲突，从而提升分配对象的效率。
  - **缺点 - 大对象无法分配**：TLAB空间比较小，所以大对象无法在TLAB分配，这时只能直接分配到线程共享的堆里面。
- 堆可以处于物理上不连续的内存空间中，可通过 -Xmx（最大堆内存）和 -Xms（初始堆内存） 来扩展空间大小。如果堆中没有内存可以完成对象分配，且堆也无法再扩展时，将会抛出OutOfMemoryError异常。

### 2.1. 详细介绍方法区？

![1626181213768](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1626181213768.png)

- **方法区**：Methed Area，别命Non-Heap（非堆），线程共享，是JVM规范中定义的一个逻辑概念，用于存储已被虚拟机加载的**类信息、常量、静态变量和即时编译后的代码**等数据，具体放在哪里，不同的实现可能会放在不同的地方。
  - **永久代**：是Hotspot虚拟机特有的概念（在别的JVM没有），是方法区的一种实现，主要存放类信息、常量等方法区内容。
    - 在JDK1.6 中，方法区中包含的数据，除了JIT编译生成的代码是存放在native memory的CodeCache区域，其他都存放在永久代。
    - 移除永久代的工作从JDK1.7就开始了，在JDK1.7中，存储在永久代的部分数据就已经转移到了Java Heap或者是Native Heap。但永久代仍存在于JDK1.7中，并没完全移除，比如**符号引用**（Symbols）转移到了native heap，**字面量**（interned strings，见字符串常量池）转移到了java heap，**类的静态变量**（class statics）转移到了java heap。
    - 在Java 8中，永久代被彻底移除，取而代之的是另一块与堆不相连的本地内存：元空间（Metaspace）中，此时‑XX：MaxPermSize 参数失去了意义，取而代之的是-XX：MaxMetaspaceSize。
  - **元空间**：JDK8后用于替代永久代，存储类的元数据信息，存放在本地内存中。
    - **元空间与永久代最大的区别**：元空间并不是在JVM虚拟机中 ，而是使用了本地内存，默认情况下，元空间的大小仅受本地内存限制，解决了永久代容易溢出的问题。
  - **元空间替代永久代的原因**：
    - 字符串存在永久代中，容易出现性能问题和内存溢出。
    - 类及方法的信息等比较难确定其大小，因此对于永久代的大小指定比较困难，太小容易出现永久代溢出，太大则会导致空间的浪费。
    - JRockit虚拟机方法区没有永久代的实现，Oracle需要将HotSpot与JRockit合二为一 ，剔除永久代。
    - （？）永久代会为GC带来不必要的复杂度，并且回收效率偏低。
- 在JDK8以后，元空间替代了永久代，使得方法区与堆存在交集，静态变量和字符串常量池存放在堆中，类信息和运行时常量池放在元空间中，而静态常量池是class文件里的常量池，未加载前并不占用内存。
  - **常量池 - 静态常量池**：也叫class文件常量池，即class文件中的常量池，占用class文件绝大部分空间。主要存放：
    - **字面量**：相当于Java语言层面常量的概念，如文本字符串、final修饰的变量。
    - **符号引用**：属于编译原理方面的概念，包括类和接口的全限定名、字段的名称和描述符、方法的名称和描述符。
  - **常量池 - 运行时常量池**：当class文件加载到内存后，JVM会将静态常量池中的内容存放到运行时常量池中，这就是常说的“常量池”，主要存放：
    - 编译期间产生的字面量、符号引用。
    - 注意，运行时常量池具有动态性，也就是并非只有通过class文件常量池才能进入，运行期间也可能将新的常量放入池中，比如调用String#intern（）方法。
  - **常量池 - 字符串常量池**：可以理解为运行时常量池中分出来的一部分，当类加载到内存时，**字符串**会存到字符串常量池里面，即在编译阶段把所有字符串放到一个常量池中。
    - String#intern（）方法，native方法，返回规范的字符串，equals判断常量池是否有存在的字符串，如果没有则会将实参字符串加入常量池。
    - 程序运行时，除非手动向常量池中添加常量，比如调用String#intern（）方法，否则JVM不会自动添加常量到常量池。
    - 至于程序启动时，哪些字符串或常量、变量会加入常量池，取决于本身的编译性质，如果本身是字面量则会加入常量池；如果是变量，由于地址不能确定，所以在不调用String#intern（）时是并不会加入常量池的。
    - JDK5以后，除了有字符串常量池，实际上还有数值型常量池，也就是Java中大部分基本类型的包装类都实现了常量池技术，比如Byte、Short、Integer、Long、Character、Boolean，而两种浮点数类型的包装类Float、Double并没有实现常量池技术。其中，只有Integer常量池缓存区间（-128~127），可通过-XX:AutoBoxCacheMax参数进行设置。
  - **常量池的好处**：
    - 常量池是为了避免频繁的创建和销毁对象而影响系统性能，实现了对象的共享。
    - 常量池可以节省内存空间：常量池中所有相同的字符串常量被合并，只占用一个空间。
    - 常量池可以节省运行时间：在比较字符串时，==比equals（）快，所以对于两个引用变量，只用==判断引用是否相等，就可以判断实际值是否相等了。
- 垃圾回收在方法区出现得比较少，这个区域回收的目的主要是针对**常量池的回收和类的卸载**。
- 方法区也是可以由内存不连续的内存区域组成，也是可扩展的，当方法区无法满足内存分配需求时，则会抛出OutOfMemoryError异常。

### 2.2. 堆和虚拟机栈的区别？

|              | 堆                                                           | 虚拟机栈                                                     |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 物理地址     | 堆的物理地址是不连续的，因此分配对象的速度较慢               | 虚拟机栈使用的是栈数据结构，物理地址是连续的，因此分配对象的速度较快 |
| 内存大小     | 由于堆是不连续的，所以分配到的内存是在运行期才确定的，因此大小不固定，一般堆大小远远大于虚拟机栈 | 虚拟机栈是连续的，所以分配到的内存大小在编译期就已经确定好了，因此大小是固定的 |
| 存放内容     | 堆存放的是对象的实例和数组，所以更关注的是数据的存储         | 虚拟机栈存放的是局部变量、操作数栈、动态链接和方法返回地址，所以更关注的是程序方法的执行 |
| 程序的可见性 | 堆是线程共享的                                               | 栈是线程私有的，只对于线程是可见                             |
| 生命周期     | 堆的生命周期与JVM的生命周期相等                              | 虚拟机栈的生命周期与所在线程的生命周期相等                   |

### 2.3. 详细介绍直接内存？

- **直接内存**：DirectBuffer，是一块由操作系统直接管理的内存，也叫**堆外内存**，并不是JVM运行时数据区的一部分，也不是JVM规范中定义的内存区域，是利用本地方法库直接在堆外申请的内存区域，这部分内存会被频繁使用，而且也可能会导致OOM错误的出现。

- 直接内存（堆外内存）的使用，**避免了在I/O操作时Java堆和Native堆中来回复制数据**，从而提高性能。

  - 在JVM层面，每当程序需要执行一个I/O操作时，都需要将数据先从**Java Heap**复制到**C Heap**中，才能够触发系统调用完成操作。
  - 其中，C Heap内存站在JVM角度来看，属于堆外内存，但是站在操作系统的角度来看，其实都属于进程的Heap，**操作系统并不知道JVM的存在**，所以认为都是普通的用户程序。因此，JVM在I/O时永远比使用Native方法多一次数据复制。
  - **为什么必须有这一次的数据复制呢**？
    - 这是因为JVM只是一个用户程序，本身并没有直接访问硬件的能力，所有的I/O操作都需要借助于系统调用来实现。在Linux系统中，与I/O相关的read（）和write（）系统调用，都需要传入一个指向在程序中分配的一片内存区域的起始地址指针，然后操作系统才会将数据填入到这片区域或者从这片区域中读出数据。
    - 如果直接使用JVM堆中对应byte[]类型的地址的话，则会有两个无法解决的问题：一是，**Java中的对象实际的内存布局跟C不一样**，不同的JVM可能有不同的实现，byte[]的首地址可能只是个对象头，并不是真实的数据；二是，**垃圾收集器的存在使得JVM会经常移动对象的位置**，这样同一个对象的真实内存地址随时都有可能发生变化，而虽然JVM知道对象地址变了，但是操作系统并不知道。
  - 因此，在适当的位置直接使用直接内存，可以避免数据从JVM Heap到C Heap的拷贝。

- **API上**：可以使用**Unsafe**类或者**ByteBuffer**类分配直接内存：

  - **Unsafe**：Unsafe.allocateMemory（size）：
    - Unsafe可用来直接访问系统内存资源并自主管理，在提升Java运行效率、增强Java语言底层操作能力方面起了很大的作用。
    - 可以认为，**Unsafe类是Java中留下的后门**，提供了一些底层的操作，比如直接访问内存、线程调度等。
    - Unsafe不属于Java标准，官方并不建议使用Unsafe，并且从JDK 9开始去Unsafe，然而目前业界有很多好用的类库大量用了Unsafe类，比如JUC atomic包下的类、Netty、Hadoop、Kafka等，所以了解一下还是有好处的。
    - 不同JDK版本中，Unsafe类有区别：在JDK 8中归属于sun.misc包下；在JDK 11中归属于sun.misc包下与jdk.internal.misc下（这个功能更强大）。

  ```java
  public class DirectMemoryTest1 {
      private static final int MB_1 = 1024 * 1024;
  
      public static void main(String[] args) throws IllegalAccessException, NoSuchFieldException {
          //通过反射获取Unsafe类并通过其分配直接内存
          Field unsafeField = Unsafe.class.getDeclaredFields()[0];
          unsafeField.setAccessible(true);
          Unsafe unsafe = (Unsafe) unsafeField.get(null);
  
          // 分配1M内存，并返回这块内存的起始地址
          long address = unsafe.allocateMemory(MB_1);
  
          // 向内存地址中设置对象
          unsafe.putByte(address, (byte) 1);
  
          // 从内存中获取对象
          byte aByte = unsafe.getByte(address);
          System.out.println(aByte);
  
          // 释放内存
          unsafe.freeMemory(address);
      }
  }
  ```

  - **ByteBuffer**：ByteBuffer.allocateDirect（size）：

  ```java
  public class DirectMemoryTest2 {
      private static final int ONE_MB = 1024 * 1024;
  
      public static void main(String[] args) {
          // 底层使用unsafe分配内存，unsafe.freeMemory(address)释放内存
          ByteBuffer buffer = ByteBuffer.allocateDirect(ONE_MB);
          
          // 相对写，向position的位置写入一个byte，并将postion+1，为下次读写作准备
          buffer.put("abcde".getBytes());
          buffer.put("fghij".getBytes());
  
          // 转换为读取模式
          buffer.flip();
  
          // 相对读，从position位置读取一个byte，并将position+1，为下次读写作准备
          // 读取第1个字节(a)
          System.out.println((char) buffer.get());
  
          // 读取第2个字节
          System.out.println((char) buffer.get());
  
          // 绝对读，读取byteBuffer底层的bytes中下标为index的byte，不改变position
          // 读取第3个字节
          System.out.println((char) buffer.get(2));
      }
  }
  ```

- **JVM参数上**：可以使用-XX：MaxDirectMemorySize控制，默认是0，表示不控制。

- **优点**：

  - **减少了垃圾回收的工作**：因为直接内存是由操作系统直接管理的内存，分配到直接内存的对象不受JVM管理，也就不用JVM对其进行垃圾回收了。
  - **I/O效率高**：由于I/O操作中，使用直接内存可以减少一次Java Heap与C Heap之间的内存拷贝，从而提高了性能。

- **缺点**：

  - **直接内存难以控制**：直接内存不受JVM管理，需要用户自己来释放内存，当发生内存溢出时排查问题可能会变得非常困难。

- **适用场景**：

  - **需要存储的数据大且生命周期长**。
  - **频繁的I/O操作**，比如并发网络通信。

### 2.4. JVM执行引擎？

- JVM核心的组件就是**执行引擎**，负责执行虚拟机的字节码，一般会先编译成机器码后执行。
- “虚拟机”是一个相对于“物理机”的概念，虚拟机的字节码是不能直接在物理机上运行的，需要执行引擎编译成机器码后才可在物理机上执行。

### 2.5. 编译器优化机制？

#### 字节码运行模式

- **解释执行**：由解释器一行一行翻译字节码执行。
  - 优势在于没有编译的等待时间，可以节省内存（不存放到CodeCache），但由于要一行一行去翻译性，所以能差一些。
- **编译执行**：把字节码编译成字节码，直接执行机器码。
  - 运行效率会高很多，一般比解释执行快一个数量级，但带来了额外的内存（CodeCache）和CPU的开销。
- 相关命令：

| JVM参数              | 显示值                             | 说明                                                  |
| -------------------- | ---------------------------------- | ----------------------------------------------------- |
| java -version        | mixed mode，表示混合模式           | 查看字节码运行模式                                    |
| java -Xint -version  | interpreted mode，表示解释执行模式 | 指定解释执行模式                                      |
| java -Xcomp -version | compiled mode，表示编译执行模式    | 指定JVM优先以编译模式运行，不能编译的再以解释模式运行 |
| java -Xmixed         | mixed mode，表示混合模式           | 指定以混合模式运行（默认）                            |

#### JIT即时编译器

- **背景**：

  - JVM一般开始会以解释器解释执行，当发现某个方法或者代码块的运行特别频繁，则会认为这些代码为**热点代码**。
  - 为了提高热点代码的执行效率，JVM会使用**即时编译器**，把这些热点代码编译成与本地平台相关的机器码，并进行**各层次的优化**。

- **概念**：Just In Time Compiler，JIT即时编译器，简称JIT编译器，在运行时JVM将会把热点代码编译成与本地平台相关的机器码，并进行各种层次的优化（比如锁粗化等），从而提高热点代码的执行效率。

  - **Hotspot - C1即时编译器**：也被称为Client  Compiler，是一个简单快速的编译器，主要关注局部性的优化，适用于执行时间较短或者对启动性能有要求的程序。比如GUI应用对界面启动速度就有一定的要求，此时适合用C1 编译器。
  - **Hotspot - C2 即时编译器**：也被称为Server Compiler，是为长期运行的服务器端应用程序做性能调优的编译器，适用于执行时间长或者对峰值性能有要求的程序。
  - **javac是前端编译**（也叫前期编译），负责把java代码编译成class字节码；而**JIT是后端编译**，负责把字节码编译成本地平台相关的机器码。

- **分层编译优化**：

  - level 0：解释执行。
  - level 1：简单的C1编译，使用C1编译器进行一些简单的优化，不开启Profiling（JVM的性能监控）。
  - level 2：受限的C1编译，仅执行**带方法调用次数**以及**循环回边执行次数**Pofiling的C1编译。
  - level 3：完全的C1编译，会执行带有所有Profiling的C1代码。
  - level 4：C2编译，使用C2编译器进行优化，该级别会启用一些编译耗时较长的优化，在一些情况下，会根据性能监控信息进行一些非常激进的性能优化。

  => 级别越高，应用启动越慢，优化的 开销越高，峰值性能也越高。

| JVM参数                                          | 默认值 | 说明                    |
| ------------------------------------------------ | ------ | ----------------------- |
| -XX：-TieredCompilation                          | ？     | 只开启C2（禁用123层）   |
| -XX：+TieredCompilation -XX：TieredStopAtLevel=1 | -      | 只开启C1（只开启0~1层） |

#### CodeCache

- **概念**：CodeCache，代码缓存区，是非堆区域，缓存的是JIT编译器编译后的代码（即机器码），以及部分JNI的机器码，不过JIT编译生成的机器码占主要部分。
  - 解释执行可以节省内存，不存放到CodeCache，立即执行。
  - 编译执行后的代码会存放在CodeCache里，虽然CodeCache在即将耗尽时会尝试回收，但满了后却会让JIT停止工作，此后已编译过的代码会继续以编译模式执行，还没有编译过的代码将会退化成以解释执行模式执行，从而出现系统运行变慢、响应时间增大的现象。

#### 热点代码

- **概念**：JVM一般开始会以解释器解释执行，当发现某个方法或者代码块的运行特别频繁，则会认为这些代码为**热点代码**。

- **探测方法**：

  - **基于采样的热点探测**：周期性检查各个线程的栈顶，经常出现在栈顶的则为热点方法。
  - **基于计数器的热点探测**：Hotspot使用的方法，思路是为每个方法或者代码块建立一个**计数器**，统计其执行的次数，如果超过某个阈值，则认为它是热点代码。

- **Hotspot内置计数器**：

  - **方法调用计数器**：Invocation Counter，用于统计方法被调用的次数（不是绝对次数，而是在一个相对的执行频率，即一段时间内方法被调用的次数），在不开启分层编译的情况下，默认C1阈值为1500次，C2为10000次。

  | JVM参数                  | 默认值 | 说明                                                     |
  | ------------------------ | ------ | -------------------------------------------------------- |
  | -XX：CompileThreshold=？ | ？     | 指定方法调用计数器阈值命令（开启分层编译后，此阈值失效） |

  ![1626357990780](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1626357990780.png)

  - **回边计数器**：

    - 回边，Back Edge，指定的是在字节码中遇到控制流向后跳转的指令。
    - 回边计数器，Back Edge Counter，用于统计一个方法中循环体代码执行的次数，在不开启分层编译的情况下，默认C1为13995次，C2为10700次。
    - 建立回边计数器的目的是为了触发OSR，OnStackReplacement编译，是一种在运行时替换正在运行函数或者方法的栈帧的技术，是一种用于提升benchmark跑分非常有效的技术。

    | JVM参数                         | 默认值 | 说明                                                 |
    | ------------------------------- | ------ | ---------------------------------------------------- |
    | -XX：OnStackReplacePercentage=? | ?      | 指定回边计数器阈值命令（开启分层编译后，此阈值失效） |

  ![1626358121407](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1626358121407.png)

#### 方法内联

- **概念**：把目标方法的代码复制到发起调用的方法之中，即方法内联，避免发生真实的方法调用，从而减少方法调用时压栈和出栈的操作，以减少内存消耗和操作的时间，提高系统性能。
  - 方法内联，本质上是空间换时间的方式，也就是即时编译器在编译期间把方法调用连接起来，从而减少入栈和出栈的开销。
- **内联条件**：
  - **方法体足够小**：
    - 热点方法，方法体小于325字节会尝试内联，可用-XX：FreqInlineSize命令修改阈值大小。
    - 非热点方法，方法体小于35字节会尝试内联，可用-XX：MaxInlineSize命令修改阈值大小。
  - **被调用的方法运行时的实现可以被唯一确定**：
    - static方法、private方法以及final方法，JIT可以唯一确定具体的实现代码，此时会尝试内联。
    - 而public的实例方法，指向的实现可能是自身、父类或者子类的代码，仅当JIT能够唯一确定其唯一实现时，才有可能完成内联。
- **内联带来的问题**：
  - 由于经过内联后的代码会变多，其增加的代码量取决于方法的调用次数和方法本身的大小，在一些极端情况下，内联可能会引起CodeCahce溢出，可能会导致JVM退化成解释执行模式。
    - CodeCahce：是热点代码的一个缓存区，即时编译器编译后的代码以及本地方法代码都会存放在这个区间内，空间大小比较有限（JDK 8中只有240M内存），比较容易出现CodeCahce溢出。

| JVM参数                             | 默认值 | 说明                                                         |
| ----------------------------------- | ------ | ------------------------------------------------------------ |
| -XX：+Printlnlining                 | -      | 打印内联详情，该参数需和-XX：+UnlockDiagnosticVMOption配合使用 |
| -XX：+UnlockDiagnosticVMOption      | -      | 打印JVM诊断相关的信息                                        |
| -XX：MaxlnlineSize=？               | 35     | 如果非热点方法的字节码超过该值（单位字节），则无法内联       |
| -XX：FreqlnlineSize=？              | 325    | 如果热点方法的字节码超过该值（单位字节），则无法内联         |
| -XX：lnlineSmallCode=？             | 1000   | 如果目标编译后生成的机器码大小大于该值（单位字节），则无法内联 |
| -XX：MaxlnlineLevel=？              | 9      | 内联方法的最大调用帧数（嵌套调用的最大内联深度）             |
| -XX：MaxTrivialSize=？              | 6      | 如果方法的字节码少于该值（单位字节），则直接内联             |
| -XX：MinlnlingThreshold=？          | 250    | 如果目标方法的调用次数低于该值，则不去内联                   |
| -XX：LiveNodeCountlnliningCutoff=？ | 40000  | 编译过程中最大活动节点（IR节点）的上限，仅对C2编译器有效     |
| -XX：lnliningFrequencyCount=？      | 100    | 如果方法的调用点（call site）的执行次数超过该值，则触发内联  |
| -XX：MaxRecursivelnlining Level=？  | 1      | 如果递归调用大于该值，则不去内联                             |
| -XX：+lnlineSynchronizedMethods     | 开启   | 是否允许同步方法的内联                                       |

#### 逃逸分析

- **概念**：分析变量能否逃出它的作用域。

- **4种逃逸场景**：

  - **全局变量赋值逃逸**：局部变量作用域放大到全局变量。

  ```java
  public static SomeClass someClass;
  
  // 全局变量赋值逃逸
  public void globalVariablePointerEscape() {
      someClass = new SomeClass();
  }
  ```

  - **方法返回值逃逸**：变量作用域随着方法返回而放大。

  ```java
  // someMethod(){
  //   SomeClass someClass = methodPointerEscape();// 方法返回值逃逸
  // }
  public SomeClass methodPointerEscape() {
      return new SomeClass();
  }
  ```

  - **实例引用逃逸**：变量作用域随着方法参数逃逸到其他作用域。

  ```java
  // 实例引用传递逃逸
  public void instancePassPointerEscape() {
      this.methodPointerEscape().printClassName(this);
  }
  
  ```

  - **线程逃逸**：类变量或者可以被其他线程中访问的实例变量，即共享变量，可以随着线程共享发生的逃逸。

- **逃逸状态标记**：JVM针对每个逃逸场景进行分析，分析后会给对象做一个逃逸状态标记。

  - **全局逃逸标记**：一个对象可能从**方法**或者**线程**中逃逸，即其他方法或者其他线程也可以访问这个对象。
    - 对象被作为方法的返回值。
    - 对象作为静态字段或者成员变量。
    - 如果某个类重写了析构函数finalize（）方法，则整个类的对象都会被标记为全局逃逸状态，并且一定会放到堆内存里面。
  - **参数逃逸状态**：一个对象被作为参数传递给一个方法，但在接收参数的方法之外无法访问该对象，且该对象对其他线程也是不可见的。
  - **无逃逸状态**：一个对象不会发生逃逸。

| JVM参数                    | 默认值       | 说明             |
| -------------------------- | ------------ | ---------------- |
| -XX：+DoEscapeAnalysis     | JDK8默认开启 | 是否开启逃逸分析 |
| -XX：+EliminateAllocations | JDK8默认开启 | 开启标量替换     |
| -XX：+EliminateLocks       | JDK8默认开启 | 是否开启锁消除   |

#### 逃逸分析优化 - 标量替换

标量替换指的是，在通过逃逸分析确定对象不会被外部访问，且对象可以进一步被分解后（聚合量），JVM不会创建该对象，而是创建其成员变量（标量）去代替。

- **标量**：不能被进一步分配的量，比如基础数据类型和对象的地址引用。
- **聚合量**：可以进一步分解的量，可以由标量聚合而成，比如字符串、自己定义变量。

```java
public void someTest() {
    // someTest没有逃逸时, 且可以进一步分解, 则可以进行标量替换
    SomeTest someTest = new SomeTest();
    someTest.age = 1;
    someTest.id = 1;

    // 开启标量替换之后, 上述代码会被优化成: 并不会创建SomeTest对象
    int age = 1;
    int id = 1;
}

```

#### 逃逸分析优化 - 栈上分配

栈上分配指的是，在通过逃逸分析确定对象不会被外部访问后，并且对象足够的小，那么JVM会直接在栈上分配对象，而其对象内存在出栈时会被回收，从而减少垃圾回收的压力。

#### 逃逸分析优化 - 锁消除

等到并发章节再写。

### 2.6. 详细介绍创建一个对象的步骤？

**步骤：类加载检查、类加载（加载、链接、初始化）、分配内存、初始化零值、设置对象头、执行init方法**

1. **类加载检查** ：当JVM遇到new指令时，⾸先去检查是否能在常量池中定位到这个类的符号引⽤，并且检查这个符号引⽤代表的类是否已被加载过、解析和初始化过。如果没有，那必须先执⾏相应的类加载过程。

2. **类加载 - 加载**：指的是将类的**class文件（二进制数据）**读入到内存，并转换成**方法区中的运行时数据结构**。同时在堆中生成一个代表这个类的**java.lang.Class对象**，该对象封装了类在方法区中的数据结构，并且向用户提供了访问方法区数据结构的接口，即Java反射的接口。

3. **类加载 - 链接**：该阶段负责把**类的二进制数据合并到JRE中**，可分为如下3个阶段：

   - **验证**：验证Class文件是否符合规范，是否能被当前的虚拟机加载处理，确保加载的类没有安全方面的问题。
   - **准备**：为类的静态变量（static）分配内存，并初始化为初始值（0或null）。而对于静态常量（final static修饰）会直接被赋值为用户定义的值。
   - **解析**：将Class常量池（Constant Pool）的符号引用转换为直接引用。

4. **类加载 - 初始化**：类初始化是类加载的最后一步，真正执行Java代码，主要工作是为静态变量（static）赋值为用户定义的值。初始化完毕类就可以被使用了。

   - 执行clinit 方法，clinit方法由编译器自动收集类里面的**所有静态变量的赋值动作及静态语句**合并而成，也叫**类构造器方法**。

5. **分配内存**：在确定对象需要创建后，接下来JVM将为对象分配内存，分配⽅式有 **“指针碰撞”** 和 **“空闲列表”** 两种，在分配内存的过程中，需要注意使用的是哪一种垃圾收集算法，因为垃圾收集算法的不同会导致内存块是否规整，从而影响到分配内存的方式是使用指针碰撞还是使用空闲列表。

   - 在进行内存分配的时候，如果使用的是指针碰撞方法，还需要注意并发情况下，内存的分配是否是线程安全的。一般使用**加同步块**的方式和**线程私有分配缓存区**这两种方式解决线程安全的问题。

6. **初始化零值**：对象内存分配完成后，JVM需要将分配到的内存空间都初始化为零值，这⼀步操作保证了对象的**实例字段**在Java代码中可以不赋初始值就直接使⽤，程序能访问到这些字段的数据类型所对应的零值。

7. **设置对象头**： 初始化零值完成之后，JVM要对对象进⾏必要的设置，比如这个对象是哪个类的实例、如何才能找到类的元数据信息、对象的哈希码、对象的GC分代年龄等信息，这些信息将存放在对象头中。另外，根据JVM当前运⾏状态的不同，比如是否启⽤偏向锁等，对象头会有不同的设置⽅式。

   - **对象头主要包括两部分**：用于存储对象自身的运行时数据（哈希码、GC 分代年龄、锁状态标志、线程持有的锁、偏向线程ID、偏向时间戳）以及类型指针（即对象指向该类元数据的指针，JVM通过这个指针来确定这个对象是哪个类的实例）。

   ![1626822381182](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1626822381182.png)

8. **执⾏init⽅法**： 从JVM的视⻆来看，⼀个新的对象已经产⽣了，但从Java程序的视⻆来看， init⽅法还没有执⾏，所有的字段都还为零。所以⼀般来说（除循环依赖），执⾏new指令之后会接着执⾏init⽅法，这样⼀个真正可⽤的对象才算产⽣出来。

   - 类初始化后，如果是实例化一个新对象，还会调用< init >方法，与< clinit >类似，< init >方法可以看作是**对象构造方法**，是由编译器自动收集类中所有实例变量的赋值动作、实例代码块和构造函数合并而成的。
   - 如果是对实例变量直接赋值或者使用实例代码块赋值，那么编译器会将这些代码合并到实例构造函数中去，并且它们还会被放在对父类构造函数的调用语句之后（因为Java要求构造函数的第一条语句必须是父类构造函数的调用语句)，自身构造函数的代码之前去执行。
   - 因此，类构造器和对象构造器的初始化过程为：**父类的类构造器 -> 子类的类构造器 -> 父类成员变量的赋值和实例代码块 -> 父类的构造函数 -> 子类成员变量的赋值和实例代码块 -> 子类的构造函数。**

### 2.7. 对象内存分配过程？

![1626823040622](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1626823040622.png)

1. 对象首先尝试栈上分配，如果栈上分配成功，则直接在栈上分配对象。
   - **栈上分配**指的是，在通过**逃逸分析**确定对象不会被外部访问后，并且**对象足够的小**，那么JVM会直接在栈上分配对象，而其对象内存在出栈时会被回收，从而减少垃圾回收的压力。
2. 如果不能在栈上分配，且**对象也足够的小**，则尝试TLAB分配，如果TLAB分配成功，则直接在TLAB分配对象。
   - **TLAB**：Thread Local Allocation Buffer，线程私有分配缓存区，是一块**线程专用的内存分配区域**，JVM会为每个线程分配一块TLAB区域，**实质占用的是Eden区的空间（即分配独享、使用共享）**，用于给每个线程往自己的TLAB中分配**小对象**，这样可以避免堆分配对象时的线程冲突，从而提升分配对象的效率。
3. 如果也不能在TLAB分配（大部分对象），则对象在创建时会优先存放到**Eden区**，当Eden区满时会触发Minor GC，即JVM会将Eden区存活的对象拷贝到**Survivor区（From Survivor/To Survivor）**里。而在下次Minor GC时，JVM又会将存活的对象拷贝到To Survivor/From Survivor区里，下下一次再周而复始。对象每经历一次垃圾回收后，如果仍然存活，则**该对象年龄+1**，当对象年龄达到阈值（默认15），则会晋升到**老年代**。
4. 然而，**新建的对象不一定直接分配到Eden区**：如果对象非常大，而新生代空间又不足，则会将该对象直接放到老年代去担保，主要是为了避免分配到采用复制算法的新生代，在大对象存活时内存拷贝带来的大量消耗。
5. 同时还要注意的是，**由于JVM有动态年龄判定机制，对象不一定要达到年龄才能进入老年代**：
   - **动态年龄**：如果Survivor区中相同年龄对象的大小总和，超过了Survivor区空间大小的一半时，则会晋升大于等于该年龄的对象到老年代。

### 2.8. Java垃圾回收机制？

- **背景**：
  - 在Java中，程序员是**不需要显式去释放一个对象的内存**的，而是由虚拟机自行执行。
  - 在JVM中，有一个垃圾回收线程，它是低优先级的，在正常情况下是不会执行的，只有在虚拟机空闲或者当前堆内存不足时，才会触发执行，扫面那些没有被任何引用的对象，并将它们添加到要回收的集合中，进行回收。
- **使用场景原则**：
  - **内存要求**：内存不够，则需要想办法提高对象的回收率，以多回收一些对象，从而腾出更多的内存。
  - **CPU要求**：CPU不够，则需要降低垃圾回收频率，让CPU多去执行业务，而不是垃圾回收。
- **垃圾回收的区域**：虚拟机栈、本地方法栈和程序计数器是线程独享的，是随着线程的创建而创建的，随着线程的销毁而销毁的， 是不需要考虑垃圾回收的；而堆和方法区是线程共享的，需要关注垃圾回收。
  - **堆**：是垃圾回收的主要区域，用于回收创建的对象。
  - **方法区**：用于回收废弃的常量以及不需要的类。
- **回收时机**：由对象存活算法决定。

### 2.9. 对象存活算法？

#### 引用计数法

通过对象的引用计数器，来判断该对象是否被引用，比如有对象引用就+1，其引用失效就-1，当为0时，则代表该对象没有被引用。

- **优缺点**：实现简单，判断效率高；但无法解决对象循环引用的问题，**目前Java并不使用该算法**。

![1626436324696](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1626436324696.png)

#### 可达性分析

以**根对象（GC Roots）**作为起点向下搜索，走过的路径被称为**引用链**（Reference Chain），如果某个对象到根对象之间没有引用链相连，则认为该对象是不可达的，是可以被回收的。**Java使用的是该存活算法**。

- **根对象**包括：
  - 虚拟机栈（栈帧中的局部变量表）中Reference对象所引用的对象。
  - 方法区中类的静态属性（static）Reference对象所引用的对象。
  - 方法区中常量（final）Reference对象所引用的对象。
  - 本地方法栈中JNI（即Native方法）Reference对象所引用的对象。

![1626436445717](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1626436445717.png)

- **可达性分析完整流程**：注意，一个对象即使不可达，也不一定会被回收，还要继续判断有无必要执行析构函数**finalize（）**方法，如果方法里面重新建立了与根对象之间的引用链，则不会去回收，否则还是会被回收。
  - **两次标记过程**：
    - 第一次标记不在“关系网”中的对象。
    - 第二次先判断该对象有没有实现finalize（）方法，如果没有实现，则直接判断该对象可回收；如果实现了，则会先放在一个队列中，并由JVM建立的一个低优先级的线程去执行它，随后会进行第二次的小规模标记，而在这次被标记的对象就会真正地被回收了。
  - **使用建议**：
    - 避免使用finalize（）方法，操作不当可能会导致问题。
    - finalize（）方法优先级低，什么时候会被调用也无法确定，因为什么时候发生GC是不确定的。
    - 建议使用try...catch...finally来代替finalzie（）方法。

![1626437028531](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1626437028531.png)

### 3.0. JVM垃圾回收算法

分为**基础垃圾回收算法**（标记清除算法、标记整理算法和复制算法）和**综合垃圾回收算法**（分代搜集算法和增量算法）。

#### 基础 - 标记清除算法

- **优缺点**：实现简单；但存在内存碎片，影响对象的内存分配速度，在极端情况下需要遍历整个内存链表。
- **算法流程**：

1. 通过可达性分析，标记需要回收的对象。
2. 再清理掉要回收的对象。

![1626437545225](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1626437545225.png)

#### 基础 - 标记整理算法

也叫标记压缩算法。

- **优缺点**：无内存碎片；但由于整理需要计算和时间整理对象到一端，存在CPU和时间的开销。
- **算法流程**：

1. 通过可达性分析，标记需要回收的对象。
2. 然后把所有存活对象压缩到内存的一端。
3. 再清理掉边界外的所有空间。

![1626437785184](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1626437785184.png)

#### 基础 - 复制算法

- **优缺点**：性能好（无需标记所有对象，只需找出存活的并移动即可），无内存碎片；但内存利用率低，最多才达到50%。
- **算法流程**：

1. 把内存分为两块，每次只使用其中一块。
2. 通过可达性分析，将存活的对象复制到另一块未使用的内存中，然后清除掉正在使用的那块内存中的所有对象。
3. 最后交换两块内存块的角色，等待下次回收重复执行上述操作。

![1626437874675](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1626437874675.png)

#### 综合 - 分代收集算法

- **概念**：

  - 各种商业虚拟机堆内存的垃圾收集基本上都采用了分代收集算法。
  - 根据对象的存活周期，把内存分为多个区域，**不同区域使用不同的回收算法**来回收对象，以提升整体性能。
  - 堆是垃圾回收的主要区域，其内存可以划分为以下区域：

  ![1626349372632](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1626349372632.png)

- **垃圾回收类型**：

  - **新生代回收**：Minor GC或者Young GC
  - **老年代回收**：Major GC，执行Major GC往往伴随一次Minor GC，所以**Major GC  ≈ Full GC**。
  - **清理整个堆**：Full GC = Major GC + Minor GC。

- **对象内存分配过程**：

  - **典型模型**：
    - **回收新生代使用复制算法**，因此新生代需要有两块内存（Eden区和Survivor区，8：2），Survivor区也有两块内存（From Survivor和To Survivor，1：1）。
      1. 对象在创建时会优先存放到**Eden区**，当Eden区满时会触发Minor GC，即JVM会将Eden区存活的对象拷贝到**Survivor区（From Survivor/To Survivor）**里。
      2. 而在下次Minor GC时，JVM又会将存活的对象拷贝到To Survivor/From Survivor区里，下下一次再周而复始。
      3. 对象每经历一次垃圾回收后，如果仍然存活，则**该对象年龄+1**，当对象年龄达到阈值（默认15），则会晋升到**老年代**。
    - **回收老年代使用标记清除或者标记整理算法**，老年代：新生代，默认内存大小比值为2：1。
  - **新建的对象不一定直接分配到Eden区**：
    - 如果对象大于-XX：PretenureSizeThreshold（默认为0，表示所有对象都优先在Eden区分配），则会直接分配到老年代。
    - 如果对象非常大，而新生代空间又不足，则会将该对象直接放到老年代去担保，主要是为了避免分配到采用复制算法的新生代，在大对象存活时内存拷贝带来的大量消耗。
  - **对象不一定要达到年龄才能进入老年代**：
    - **动态年龄**：如果Survivor区中相同年龄对象的大小总和，超过了Survivor区空间大小的一半时，则会晋升大于等于该年龄的对象到老年代。

- **触发垃圾回收的条件**：

  - **新生代Minor GC**：Eden区空间不足时。
  - **老年代/Full GC**：
    - **老年代空间不足**：没有足够空间，或者内存碎片过多导致没有足够的连续空间去分配对象。
    - **元空间不足**：方法区的元空间不足也会触发Full GC。
    - **显示调用System.gc（）**：该方法的作用是建议垃圾回收器执行垃圾回收，会触发Full GC，可以使用-XX：+DisableExplicitGC参数忽略System.gc（）的调用。

- **分代收集算法的好处**：

  - **更有效的清除不再需要的对象**：对于生命周期比较短的对象，在新生代就会被回收掉了。
  - **提升了垃圾回收的效率**：如果不做分代处理，每次回收需要扫描整个堆的对象，而分代回收则需要扫描新生代或者老年代就可以了。

- **分代收集算法的调优原则**：

  - **合理设置Survivor区的大小，避免内存浪费**：因为Survivor区的内存利用率不高，如果设置得过大，则会导致内存浪费严重。
  - **让GC尽量发生在Minor GC级别，尽量减少Full GC的发生**。

| JVM参数                        | 默认值 | 说明                                                         |
| ------------------------------ | ------ | ------------------------------------------------------------ |
| -XX：+NewRatio=？              | 2      | 老年代：新生代的内存大小比值                                 |
| -XX：SurvivorRatio=？          | 8      | Eden区：Survivor区的内存大小比值                             |
| -XX：PretenureSizeThreshold=？ | 0      | 分配到老年代的对象大小阈值，为0表示不做限制，所有对象都优先在Eden区分配 |
| -Xms                           | -      | 最小堆内存                                                   |
| -Xmx                           | -      | 最大堆内存                                                   |
| -Xmn                           | -      | 新生代大小                                                   |
| -XX：+DisableExplicitGC        | 开启   | 忽略掉System.gc（）的调用                                    |
| -XX：NewSize=？                | -      | 新生代初始内存大小                                           |
| -XX：MaxNewSize=？             | -      | 新生代最大内存                                               |

#### 综合 - 增量算法

每次只收集一小片区域内存的垃圾，从而减少系统的停顿时间，见G1收集器的实现。

### 3.1. JVM垃圾收集器？

#### 相关概念

- **垃圾回收算法**：为实现垃圾回收提供理论支持。
- **垃圾收集器**：利用垃圾回收算法，实现垃圾回收的实践落地。
- **Stop The World**：**简写为STW，也叫全局停顿**，处于该状态时，Java代码将停止运行，而native代码可以继续运行，但无法与JVM进行交互。
  - **原因**：多半由于垃圾回收导致，也有可能由Dump线程、Dump堆、死锁检查等操作导致。
  - **危害**：服务会停止，没有响应；STW时间过长，可能会导致主从发生切换，影响生产环境。
- **并行收集**：指多个垃圾收集线程同时并行工作，但在收集过程中，用户线程处于等待状态。
- **并发收集**：指用户线程与垃圾收集线程同时工作。
- **应用吞吐量**：指的是CPU用于运行业务代码的时间，与CPU总消耗时间的比值。
  - **计算公式**：应用吞吐量 = 运行业务代码时间 / （运行用户代码时间 + 垃圾收集时间）* 100%，垃圾收集时间越长，应用吞吐量越小。

 ![1626495749202](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1626495749202.png)

#### 新生代 - Serial收集器

- **背景**：最基本的、发展历史最悠久的收集器。
- **垃圾收集算法**：复制算法。
- **特点**：
  - **单线程、简单、相对高效**：由于是单线程实现的，不存在与其他线程的交互开销，可以专心做垃圾回收。
  - **收集过程全程Stop The World**。
- **适用场景**：
  - **用于客户端程序**：如果应用以java -client -jar方式启动时，默认使用的就是Serial收集器。
  - **用于单核机器上**：常见于一些嵌入式低性能的机器上运行。
- **执行过程**：
  - **Safepoint**：当发生GC时，用户线程必须全部停下来，才可以进行垃圾回收，这个状态可以认为JVM 是安全的（safe），整个堆的状态是稳定的。如果在GC前，有线程迟迟进入不了safepoint状态，那么整个 JVM都在等待这个线程，从而造成了GC整体时间变长。

![1626496514434](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1626496514434.png)

#### 新生代 - ParNew收集器

- **背景**：Serial收集器的多线程版本，除了使用多线程不同以外，其他都和Serial收集器一样，包括JVM参数、Stop The World别的表现和垃圾收集算法。
- **垃圾收集算法**：复制算法。
- **特点**：
  - **多线程、收集过程全程Stop The World**。
  - 可使用**-XX：ParallelGCThreads**设置垃圾收集的线程数，一般设置为CPU核心数就可以了。
- **适用场景**：主要用来和CMS收集器配合使用。
- **执行过程**：

![1626505507768](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1626505507768.png)

#### 新生代 - Parallel  Scavenge收集器

- **背景**：也叫吞吐量优先收集器，也是并行的多线程收集器（多线程的方式与ParNew收集器类似）。
- **垃圾收集算法**：复制算法。
- **特点**：
  - **可以达到一个可控制的吞吐量**：
    - -XX：MaxGCPauseMillis，设置阈值后，JVM将**尽力控制**最大的垃圾收集停顿时间为该阈值。
    - -XX：GCTimeRatio，设置吞吐量的大小，取值0~100，设置后JVM将花费不超过1 + / （1+n）的时间用于垃圾收集。
  - **自适应GC策略**：可用-XX：+UseAdptiveSizePolicy启用，启用后无需手动设置-Xmn、-XX：SurvivorRatio等参数，虚拟机会根据系统的运行状况收集性能监控信息，动态地调整这些参数，从而达到最优的停顿时间以及吞吐量。因此Parallel  Scavenge收集器存在着一定的智能性。
- **适用场景**：比较注重吞吐量的场景。
- **执行过程**：

![1626505950316](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1626505950316.png)

#### 老年代 - Serial Old收集器

- **背景**：也叫串行老年代收集器，可以认为是Serial收集器的老年代版本。
- **垃圾回收算法**：标记整理算法。
- **特点**：除了算法采用标记整理算法与Serial收集器不同之外，其他都是一样的。
- **适用场景**：
  - 可以和Serial、ParNew、Parallel Scavenge三个新生代收集器配合使用。
  - CMS收集器在出现故障时，会使用Serial Old收集器作为备用。
- **执行过程**：

![1626506526709](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1626506526709.png)

#### 老年代 - Parallel Old收集器

- **背景**：可以认为是Parallel Scavenge的老年代版本。
- **垃圾回收算法**：标记整理算法。
- **特点**：只能和Parallel Scavenge新生代收集器使用。
- **适用场景**：关注吞吐量的场景。
- **执行过程**：

![1626506871826](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1626506871826.png)

#### 老年代 - CMS收集器

- **背景**：CMS，Concurrent Mark Sweep，并发标记清除，是一个并发收集器，可以与用户线程同时工作。
- **垃圾回收算法**：标记清除算法。Serial Old与Parallel Scavenge采用的是标记整理算法。
- **特点**：
  - **优点**：
    - **Stop The World时间比较短，大多过程都是并发执行的**：只有1. 初始标记和5. 重新标记阶段存在Stop The World，其他阶段都是并发执行的）。
  - **缺点**：
    - **CPU资源比较敏感，并发执行的阶段会导致应用吞吐量的降低**：由于垃圾收集线程也需要占用一定的CPU资源，与业务线程一起去争抢CPU时间片，导致影响业务线程的执行效率，降低应用吞吐量。
    - **无法处理浮动垃圾**：由于并发清除阶段用户线程仍在并发执行，其可能会产生新的垃圾，这部分垃圾称为浮动垃圾，而CMS无法在本次GC清理掉这些浮动垃圾，需要留到下次GC才能清理掉。
    - **不能等到老年代几乎满了才开始收集**：因为用户线程并发执行，必须为老年代预留足够的内存给用户线程使用。如果CMS执行期间预留的内存不能满足用户程序的需要，则会出现一次Concurrent Mode  Failure异常，这将会导致JVM**改用备用的Serial Old收集器**去收集老年代的垃圾，从而导致Stop The World时间加长。
      - 可使用CMSInitiatingOccupancyFraction，设置老年代占比达到多少后（默认68%），就会触发CMS垃圾收集。
    - **存在内存碎片（最令人诟病的地方）**：标记清除算法会导致内存碎片的产生。
      - 可使用UseCMSCompactAtFullCollection，在完成Full GC后是否要进行内存碎片的整理（默认打开）。
      - 也可使用CMSFullGCsBeforeCompaction，在进行几次Full GC后就进行一次内存碎片的整理（默认为0）。
  - **其他**：对于CMS收集器，Major GC和Full GC并不约等于，因为CMS是作用在老年代的垃圾回收，这里讲的Major GC并不是之前讲的Full GC。
- **适用场景**：
  - **希望系统停顿时间短，响应速度快的场景**：比如各种服务端应用场景。
- **执行过程**：

1. **初始标记**： 
   - initial  mark，标记根对象（GC Roots）能直接关联到的对象，因此能够标记到的对象会比较少。
   - 存在Stop The World，不过由于标记的对象比较少，所以STW的时间也是比较短的。
2. **并发标记**：
   - concurrent mark，找出所有根对象（GC Roots）能够关联到的对象。
   - 垃圾收集线程和用户线程并发执行，没有Stop The World。
3. **并发预清理**：
   - concurrent-preclean，**不一定会执行的阶段**，可用-XX：-CMSPrecleaningEnabled，关闭并发预清理阶段，默认是打开的。
   - 重新标记那些在并发标记阶段，引用被更新了的对象（比如新晋升到老年代的对象），从而减少后面重新标记阶段的工作量。
   - 垃圾收集线程和用户线程并发执行，没有Stop The World。
4. **并发可中止的预清理阶段**：
   - concurrent-abortable-preclean，**不一定会执行的阶段**，使用该阶段的前提条件是：当Eden区使用量大于CMSScheduleRemarkEdenSizeThreshold的阈值（默认2M）时，才会执行该阶段。
   - 与并发预清理阶段所工作的事情是一样的。
   - 垃圾收集线程和用户线程并发执行，没有Stop The World。
   - 该阶段的主要作用是：允许用户能够控制预清理阶段的结束时机。
     - 比如，扫描多长时间：可用CMSMaxAbortablePrecleanTime进行设置，默认为5秒。
     - 再如，当Eden区使用占比达到多大阈值就结束本阶段：可用CMSScheduleRemarkEdenPenetration进行设置，默认为50%。
5. **重新标记**：
   - remark，修正并发标记期间，由于用户线程并发运行，导致标记发生变动的那些对象的标记。
     - 比如在并发标记期间，错误地把已经死亡了的对象，标记为了存活，会导致部分垃圾不被回收。
     - 再如把存活的对象错误地标记成为了死亡，可能会导致用户程序之后无法继续执行。
   - 存在Stop The World，一般来说（经验之谈），重新标记所花费的时间会比初始标记阶段的要长一些，但会比并发标记阶段的段一些。
6. **并发清理**：
   - concurrent sweep，或者叫**并发清除**，会基于标记结果，清除掉要前面标记出来需要清除的垃圾（会存在内存碎片）。
   - 垃圾收集线程和用户线程并发执行，没有Stop The World。
   - 为什么是并发清除，而不是并发整理？
     - 由于本阶段是并发执行的，如果还要整理对象的话，则还需要移动对象的位置。
     - 试想一下如果既要回收垃圾，又要整理移动对象的位置，还要与用户线程并发执行，保证业务程序没有问题，这实现起来会变得非常困难，还容易出错。
     - 而采用并发清除就变得容易了许多，因此这里是并发清除而不是并发整理。
7. **并发重置**：
   - concurrent reset，清理本次CMS GC的上下文信息，为下一次GC做准备。
   - 垃圾收集线程和用户线程并发执行，没有Stop The World。

![1626507231972](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1626507231972.png)

#### 新生代&老年代 - G1收集器

- **背景**：Garbge First，是一款面向服务器端应用的垃圾收集器，既可以用在新生代，又可以用在老年代，即整个堆内存，会优先处理那些垃圾多的Region（First是价值优先的意思）。

- **垃圾回收算法**：复制算法。

- **革命性变化**：

  - **堆内存布局上的变化**：G1将整个堆划分成了若干个大小相等的区域，每个区域叫一个Region。
    - Region的大小可通过-XX：G1HeapRegionSize来指定，取值范围为1M~32M，必须为2的N次幂。
    - 在G1收集器里，同一代的对象可能是不连续的：一共分为4类Region，分别为Eden Region（伊甸园）、Survivor Region（存活区）、Old Region（老年代）、 Humongous Region（用于存储大对象，即超过Region大小一半的对象，而特大对象会分配到连续的Humongous Region里面）。

  ![1626511732234](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1626511732234.png)

  - **设计思想上的变化**：**化整为零，分而治之**，本质上是应用了**增量算法**的思想。
    - 将堆内存按照Region分成多个块。
    - 然后会去跟踪每个Region里面的垃圾堆积的价值大小（即回收一个Region能够获取到多大的剩余空间）。
    - 最后构建一个优先列表，根据允许的收集时间，**优先回收价值高的Region**（回收后能够得到大的空间），以获得更高的垃圾收集效率。

- **特点**：

  - 可以作用在整个堆，既可以作用在新生代，又可以作用在老年代。
  - 垃圾回收时的停顿时间是可控的。
    - 可用MaxGCPauseMillis=？去控制。
  - 回收Region使用的是复制算法，无内存碎片的问题。

- **适用场景**：

  - 占用内存较大的应用（比如6G以上）。
  - 用于替换CMS垃圾收集器。
    - 对于JDK 8，G1和CMS的性能差异并不大，都可以使用。（经验之谈）如果内存<=6G，建议使用CMS；如果内存>6G，可以考虑使用G1。
    - 对于> JDK 8，则使用G1，因为CMS从JDK 9就已经被废弃了。

- **垃圾收集机制**：

  - **Young GC**：过程上与之前的Minor GC差不多（**复制算法**），只不过回收的单位是Region。

    - 所有Eden Region都满了时，会触发Young GC。
    - 所有Eden Region里面存活的对象，都会转移到Survivor Region里面去。
    - 而原先在Survivor Region中存活的对象，则会转移到新的Survivor Region中，或者晋升到Old Region中。
    - 其中，回收后空闲的Region会被放入空闲的列表中，等待下次被使用。

  - **Mixed GC**：最能体现G1的设计思想，与CMS有类似之处，但也有许多差异，比如使用的是复制算法。

    - 老年代大小占整个堆的百分比达到一定阈值时，则会触发Mixed GC。
      - 可用-XX：InitiatingHeapOccupancyPercent指定，默认为45%。
    - Mixed GC会回收所有Young Region，同时回收**部分**Old Region，回收那些根据收集时间与回收价值而选择的Old Region。
    - **执行过程**：除2. 并发标记是并发执行，其他阶段都是需要Stop The World的，但由于每次只回收部分Region，所以**Stop The World的时间是可控的**。

    1. **初始标记**：
       - Initial Marking，与CMS的初始标记类似，都是标记根对象（GC Roots）能直接关联到的对象。
       - 存在Stop The World，不过由于标记的对象比较少，所以STW的时间也是比较短的。
    2. **并发标记**：
       - Concurrent Marking，与CMS的并发标记类似，用于找出所有根对象（GC Roots）能够关联到的对象。
       - 垃圾收集线程和用户线程并发执行，没有Stop The World。
    3. **最终标记**：
       - Final Marking，与CMS的重新标记类似，用于修正并发标记期间，由于用户线程并发运行，导致标记发生变动的那些对象的标记。
         - 比如在并发标记期间，错误地把已经死亡了的对象，标记为了存活，会导致部分垃圾不被回收。
         - 再如把存活的对象错误地标记成为了死亡，可能会导致用户程序之后无法继续执行。
       - 存在Stop The World。
    4. **筛选回收**：
       - Live Data Counting and Evaluation，会对各个Region的回收价值和成本进行排序，根据用户所期望的停顿时间（MaxGCPauseMillis）来制定回收计划，并选择一些Region进行回收。
       - **回收过程：复制算法，无内存碎片**。
         - 选择一系列Region构成一个回收集。
         - 接着把决定要回收的Region中的存活对象复制空的 Region中。
         - 最后删除掉需要回收的Region。
       - 存在Stop The World。

  ![1626513840821](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1626513840821.png)

  - **Full  GC**：
    - 当G1在复制对象时发现内存不够，或者无法分配足够内存（比如特大对象没有足够的连续Humongous Region可分配）时，则会触发Full GC。
    - 一旦触发Full GC，在Full GC模式下使用的是Serial Old模式的垃圾回收，将会出现长时间的Stop The World。

- **G1调优原则**：

  - 尽量减少Full GC的发生，尽量只停留在Young GC或者Mixed GC的模式上进行垃圾回收。
  - **减少Full GC的思路**？
    - **增加预留的内存**：可用过加大-XX：G1ReserveRercent来实现，默认为堆的10%。
    - **更早地回收垃圾，可降低老年代大小占整个堆的百分比的阈值，提早触发Mixed GC**：可通过减少-XX：InitiatingHeapOccupancyPercent来实现，默认为45%。
    - **增加并发阶段使用的线程数**：可增大-XX：ConcGCThreads，这样就可以有更多的垃圾回收线程去工作，但会降低业务应用的吞吐量。

#### 其他垃圾收集器

Shenandoah、ZGC、Epsilon，JDK 14处于实验状态，不建议在生产环境中使用。

#### 如何选择垃圾收集器？

不能纸上谈兵，要根据实际情况选择。

- 应用系统所关注的最主要的矛盾点？
  - 响应快、吞吐量高：Parallel Scaveng。
  - Web应用，低延迟：CMS或者G1。
  - 桌面端应用，启动慢：Serial & -Xverify：none参数。
- 应用系统的基础设施？
  - 单核：Serial。
  - Windows + JDK11：应用不了ZGC，需要升级到JDK14才支持。
- 应用系统的JDK的版本？
  - JDK6用不了G1。
  - Oracle JDK用不了Shenandoah。

#### 垃圾收集器相关JVM参数

详细见《JVM参数选型-高级选型-常用高级垃圾收集选项》一栏。

### 3.2. JVM性能调优工具？

详细见《JVM调优工具集锦》：基于JDK 11编写。

#### JDK内置 - 监控类工具

##### jps

Java Virtual Machine Process Status Tool，实验性工具，用于查看所有Java进程，比ps -ef  | grep java方便。

```java
eg：
jps -q：只查看进程号。
jps -m：查看传递给main方法的参数。
jps -l：查看启动类的全限定名。
jps -v：查看JVM启动时的参数。

```

##### jstat

JVM Statistics Monitoring Tool，实验性工具，⽤于监控JVM的各种运⾏状态息，包括**内存状态**和**垃圾回收**。

```java
命令格式：jstat -<option> [-t] [-h<lines>] <vmid> [<interval> [<count>]]
  -<option> ：指定参数，取值可⽤jstat -options查看
  -t        ：⽤来展示每次采样花费的时间
  <lines>   ：每抽样⼏次就列⼀个标题，默认0，显示数据第⼀⾏的列标题
  <interval>：抽样的周期，格式使⽤:<n>["ms"|"s"]，n是数字，ms/s是时间单位，默认是ms
  <count>   ：采样多少次停⽌，默认是一直打印

option参数解释：
-class    :：显示类加载器的统计信息
-gcutil    ：垃圾回收统计概述
-gc        ：垃圾回收堆的行为统计
-gcnew     ：新生代行为统计
-gcold     ：年老代和永生代行为统计
-gccapacity：各个垃圾回收代容量(young,old,perm)和他们相应的空间统计

```

#### JDK内置 - 故障排查类工具

##### jinfo

Java Configuration Info，实验性参数，主要⽤来查看以及调整JVM参数。

```java
命令格式：jinfo <option> <pid>

1）查看能力:
jinfo 11666：查看11666进程的Java System Properties、VM Flags、VM Arguments
jinfo -sysprops 11666          ：只查看11666进程的Java System Properties
jinfo -flags 11666             ：只查看11666进程的VM Flags
jinfo -flag 11666 Xmx          ：只查看11666进程的Xmx参数（最大堆内存大小）
jinfo -flag 11666 Xms          ：只查看11666进程的Xms参数（初始堆内存大小）
jinfo -flag 11666 Xmn          ：只查看11666进程的Xmn参数（新生代内存大小）
jinfo -flag 11666 MetaspaceSize：只查看11666进程的MetaspaceSize参数（元空间内存大小）

2）动态修改能力：不用重启JVM就可以生效，但能力比较有限
java -XX:+PrintFlagsInitial | grep manageable：只有显示出来的结果才能被动态修改。
开关类（打开/关闭）：jinfo -flag +HeapDumpAfterFullGC 11666
赋值类（更新为60） ：jinfo -flag MinHeapFreeRatio=60 11666

```

##### jmap

Java Memory Map，实验性工具，⽤来展示对象内存映射或者堆内存详细信息。

- **生成堆dump的8种方式**：
  - **jmap**：jmap -dump:live,format=b,file=mydump.hproft 11666：转储java的堆Dump文件为mydump.hprof。
  - **jcmd**：jcmd 11666 GC.heap_dump -all mydump.hprof：⽣成Java堆Dump⽂件（HPROF格式）。
  - **jhsdb jmap**：jhsdb jmap --binaryheap --dumpfile mydump.hprof --pid 11666：生成11666进程的堆dump文件。
  - **Visual VM**：Monitor界面的Heap Dump按钮Dump堆，相当于jmap dump命令。
  - **OOM异常自动后生成**：使用-XX：+HeapDumpOnOutOf  MemeoryError，使JVM在OOM异常出现后自动生成堆Dump文件。
  - **Ctrl + （Pause）Break生成**：使用-XX：+HeapDumpOnCtrBreak，开启后可使用Ctrl + （Pause）Break，让虚拟机生成堆Dump文件。
  - **kill -3命令**：在Linux操作系统下，发送kill -3 pid命令生成堆Dump文件。
  - **借助SpringBoot Actuator生成**：对于SpringBoot应⽤，可以使⽤SpringBoot Actuator提供的/actuator/heapdump来实现堆Dump的生成。

```java
命令格式：jmap [option] <pid>

option参数解释：
-heap             ：打印java heap摘要
-clstats          ：打印Java堆的类加载器统计信息
-finalizerinfo    ：打印等待finalization的对象的信息
-histo[:live]     ：打印Java堆的直⽅图。如果指定了live⼦选项，则仅统计活动对象
-dump:dump_options：生成java堆的dump文件。其中，dump_options的取值为：
              live：指定时，仅Dump活动对象；如果未指定，则转储堆中的所有对象
          format=b：以hprof格式Dump堆
     file=filename：将堆Dump到filename

eg:
jmap -dump:live,format=b,file=mydump.hproft 11666：转储java的堆dump文件为mydump.hprof

```

##### jstack

Stack Trace for Java，实验性工具，⽤于打印当前虚拟机的线程快照（线程快照也叫Thread Dump或者javacore⽂件，包含展示每个线程正在做什么、执行到了哪里等信息），常用于定位线程出现长时间卡顿的原因，比如死锁、死循环等。

- **生成线程dump的4种方式**：
  - **jstack**：jstack -l -e 11666：打印11666进程的所有线程以及持有锁的额外信息。
  - **jcmd**：jcmd 11666 Thread.print -l：打印11666进程所有线程以及线程持有锁的额外信息。
  - **jhsdb jstack**：jhsdb jstack --locks --mixed -pid 11666：打印11666进程的栈、本地方法栈以及持有锁的额外信息。
  - **VisualVM**：Threads界面的Thread Dump按钮Dump线程，相当于jstack。

```java
命令格式：jstack [-l][-e] <pid>
  
option参数解释：
-l：显示有关线程持有的锁的额外信息
-e：展示有关线程的额外信息（⽐如分配了多少内存、定义了多少个类等等）

eg：
jstack -l -e 11666：打印11666进程的所有线程以及持有锁的额外信息

```

##### jhat

JVM Heap Analysis Tool，实验性工具，⽤来分析jmap⽣成的堆Dump，能力比较弱，有非常多的替代品（比如VisualVM、Eclipse Memory Analyzer），且在JDK 11已被废弃（可在JDK 8中使用），对于学习不太重要了。

```java
命令格式：jhat [options] heap-dump-file

option参数解释：
-stack false | true   ：开启或关闭跟踪对象分配调⽤栈，默认true
-refs false | true    ：开启或关闭对对象引⽤的跟踪，默认true
-port port-number     ：指定jhat HTTP Server的端⼝，默认7000
-exclude exclude-file ：指定⼀个⽂件，该⽂件列出了应从可达对象查询中排除的数据成员
-baseline exclude-file：指定基线堆Dump⽂件。两个堆Dunmp中，对于⽐较两个不同的堆转储很有⽤
-debug intSets        ：指定该⼯具的debug级别。设置为0，则不会有debug输出。数值越⾼，⽇志越详细
-version              ：显示版本

```

##### jcmd

JVM Command，⽤于将诊断命令请求发送到正在运⾏的Java虚拟机，从JDK 7开始提供。

```java
命令格式：jcmd <pid | main class> <command ...|PerfCounter.print|-f file>

命令参数解释：
pid              ：接收诊断命令请求的进程ID
main class      :：接收诊断命令请求的main类的所有进程
command          ：command必须是⼀个有效的jcmd命令
PerfCounter.print：打印指定Java进程上可⽤的性能计数器
-f filename      ：从指定⽂件中读取命令并执⾏
-l               ：查看所有的JVM进程。jcmd不使⽤参数与jcmd -l效果相同

eg：
jcmd 11666 GC.heap_dump -all mydump.hprof：⽣成Java堆Dump⽂件（HPROF格式）
jcmd 11666 GC.run						 ：调⽤一次java.lang.System.gc()
jcmd 11666 Thread.print -l		         ：打印11666进程所有线程以及线程持有锁的额外信息

```

##### jhsdb

Java Hotspot Debugger，Hotspot进程调试器，可⽤于从崩溃的JVM附加到Java进程或核⼼转储，从JDK 9开始引入，JDK 9前使用sa-jdi.jar（jhsdb的原型）也是可以的。

```java
1）jhsdb clhsdb --pid 11666：进入11666进程的jhsdb的交互界面
eg：（交互界面下）
	flags          ：展示所有以-XX开头的JVM参数的值
	g1regiondetails：查看G1每个Region的起始指针、结束指针、是哪一个分代的信息

2）jhsdb hsdb --pid 11666：进入11666进程的图形化界面

3）jhsdb jinfo --flags --pid 11666：打印11666进程的VM标志

4）jhsdb jmap --binaryheap --dumpfile mydump.hprof --pid 11666：生成11666进程的堆dump文件

5）jhsdb jstack --locks --mixed -pid 11666：打印11666进程的栈、本地方法栈以及持有锁的额外信息

6）jhsdb jsnap --all -pid 11666：打印11666进程所有性能计数器的信息=jcmd的PerfCounter.print

```

#### JDK内置 - 可视化工具

##### jhsdb

jhsdb hsdb --pid 11666：进入11666进程的图形化界面，其菜单功能包括：

- Inspect Thread：这个线程的诊断信息，包含对象头和指向对象元数据的指针（Java类型的名字、继承关系、实现接⼝关系，字段信息、⽅法信息、运⾏时常量池的指针、内嵌的虚⽅法表（vtable）以及接⼝⽅法表（itable）等）。
- Stack Memory：这个线程栈的内存数据信息。
  - 第⼀列：内存地址（虚拟地址，⾮物理内存地址）。
  - 第⼆列：该地址上存储的数据，以字宽为单位。
  - 第三列是：对数据的注释，竖线表示范围，横线或斜线连接范围与注释⽂字。
- Show Java stack trace：这个线程的线程栈信息。
- Show Thread Information：这个线程的其他信息。
- Find Crashes：可找出这个线程崩溃的原因。
- Windows Console：可输⼊诊断命令，也就是jhsdb clhsdb命令交互页面。

![1626576342750](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1626576342750.png)

##### jconsole

- Java Monitoring and Management Console，是⼀款基于JMX（Java Manage-ment Extensions）的可视化监控、管理⼯具，主要是通过JMX的MBean（Managed Bean）对系统进⾏信息收集和参数动态调整。
  - JMX是⼀种开放性的技术，它既可以⽤在虚拟机本身的管理上，也可以⽤于运⾏在虚拟机之上的软件中。⽬前很多软件都⽀持基于JMX进⾏管理与监控。
- 执行jconsole命令打开界面，然后输入线程号即可。

![1626576867182](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1626576867182.png)

##### VisualVM

- 也叫JVisualVM，是⼀个All-in-One Java Troubleshooting Tool，即多合一的故障排查工具，从JDK 6开始提供，是⽬前最强⼤的监控及故障处理程序之⼀。
- JDK 8输入jvisualvm启动即可，JDK 9输入独自安装，其界面菜单功能包括：
  - Overview：展示应⽤的概要信息，相当于可视化的jps、jinfo。
  - Monitor：展示一些监控信息，包括CPU、内存、类、线程等曲线图，Perform GC按钮通知JVM执⾏垃圾回收，Heap Dump按钮Dump堆，相当于jmap dump命令。
  - Threads：展示查看线程状态，以及Thread Dump按钮Dump线程，相当于jstack。
  - Sampler：抽样器，可⽤于实时性能分析，支持CPU抽样以及内存抽样。
  - Profiler：性能分析，提供了程序运⾏期⽅法级的处理器执⾏时间分析及内存分析。
    - 执⾏该性能分析，会对程序运⾏性能有⽐较⼤的影响，⼀般不建议在⽣产环境使⽤这项功能，建议使用JMC来代替。
    - 开启类共享（⼀种共享类，从⽽提升加载速度、节省内存的技术）可能会导致执⾏Profiler的应⽤崩
      溃，建议在执⾏Profiler的应⽤上添加-Xshare:off，关闭掉类共享。
  - 分析堆dump文件：File -> Load -> 选择hprof -> 打开 -> 分析。
  - 其他插件：VisualVM还支持安装插件来扩展功能，比如Visual CC来实时分析垃圾回收的情况。

![1626577315983](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1626577315983.png)

##### JDK Mission Control

- 也叫Java Mission Control，简称JMC，是一款监控、定义线上问题以及性能调优的神器。
  - 它是⼀款商业授权⼯具（例如在JDK 8中），需要商业授权才能在⽣产环境中使⽤，现已开源，在JDK 11（哪怕是OpenJDK）中，任何⼈都可以使⽤JFR + JMC（需遵循 UPL协议 ）。
- JMC的两大功能：
  - 作为JMX控制台，监控虚拟机MBean提供的数据。
  - 可持续收集数据的JFR，并可作为JFR的可视化分析⼯具。
    - JFR：Java Flight Recorder，是⼀种⽤于收集有关运⾏中的Java应⽤的诊断信息和性能数据的⼯具。它⼏乎没有性能开销，因此，即使在负载很⼤的⽣产环境中也可以使⽤。
    - 主要用于性能分析、性能分析、⽀持与调试的场景。
- MBean服务器菜单功能介绍：
  - 概览：各种概要信息。
  - MBean浏览器：展示应⽤被JMX管理的Bean。
  - 触发器：配置触发规则，当规则满⾜时，就触发某个操作（在操作⼀栏配置）。
  - 系统：查看系统相关信息。
  - 内存：查看内存相关信息。
  - 线程：查看线程相关信息。
  - 诊断命令：可视化使⽤诊断命令，相当于可视化的jcmd。
- 飞行记录性菜单功能介绍：
  - 如果应用JDK版本 < JDK11，则启动项目时需要添加-XX:+UnlockCommercialFeatures -XX:+FlightRecorder。
  - ⾃动分析结果：JMC⾃动给出的优化提议。
  - Java应⽤程序：展示应⽤的各种执⾏情况。
  - JVM内部：展示JVM层⾯的执⾏情况。
  - 环境：展示操作系统层⾯的执⾏情况。
  - 事件：展示录制期间发⽣的事件。
- JMC优点：
  - JFR在⽣产环境中对吞吐量的影响⼀般不会⾼于1%。
  - JFR监控过程是可动态的，⽆需重启。
  - JFR监控过程对应⽤完全透明，⽆需修改应⽤的代码，也⽆需安装额外的插件或代理。
  - JFR提供的数据质量⾮常⾼，对监控、排查的参考价值更⼤。
- JMC缺点：
  - JFR并不完全向后兼容。⽐如，在JDK 11⾥⾯⽣成的JFR⽂件，⽤早期的JMC（例如JMC 5.5）⽆法打开。
  - JMC 7.0.1⽆法分析堆dump⽂件（hprof格式），但 官⽅Wiki 宣称⽀持分析堆dump⽂件。

![1626578206459](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1626578206459.png)

#### 第三方工具

##### Memory Analyzer Tool

- Memory Analyzer Tool，简称MAT，可以作为独⽴软件，也可作为Eclipse插件存在，是⼀个快
  速且功能丰富的Java堆内存分析器，可帮助您查找内存泄漏并减少内存消耗。

- MAT主要功能：

  - 找出内存泄漏的原因。
  - 找出重复引⽤的类和jar。
  - 分析集合的使⽤。
  - 分析类加载器。

- 相关概念：

  - **浅堆**：一个对象E自身所消耗的内存，根据堆转储格式，对象⼤⼩可能会被调整（例如，对⻬为8bit），从⽽更好地模拟VM的实际消耗量。⼀般来说，对象的浅堆是对象在堆中的⼤⼩，⽽同⼀对象的保留⼤⼩是在垃圾回收对象时将释放的堆内存量。
  - **X的保留集**：Retained set，当E被垃圾回收时，由GC删除的对象集E和G。同理，如果E没有被回收，那么该集合中的对象E和G都会“保留下来”。
  - **X的保留堆**：Retained heap，指的是对象E的保留集E和G的内存⼤⼩，即由于它的存活导致多⼤的内存没有被回收。
  - **前导对象集的保留集**：前导对象E不可达时，被释放的那些对象E和G，所以这里前导对象集E的保留集为E和G。

  ![1626581295844](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1626581295844.png)

  - **支配树**：MAT提供了对象图的⽀配树，通过将对象参考图转换为⽀配树，可以轻松地识别最⼤的保留内存块以及对象之间的依赖关系。⽀配树是在对象图的基础上建⽴的，在⽀配树中，每个节点都是其⼦节点的直接⽀配者。因此，基于⽀配树可以轻松看出对象之间的依赖关系。其具有以下属性：
    - 对象从属于X的⼦树（例如对象被X⽀配）就是X的Retained set。
      - **X⽀配Y**：如果对象图中从起始（或Root）节点到Y的每条路径都必须经过X，那么就说对象X⽀配对象Y。
      - **直接⽀配者**：某个对象路径最短的⽀配者。
      - **间接支配者**：一个对象X支配了该对象Y，但又不是Y的直接支配者，则称X为Y的间接支配者。
    - 如果X是Y的直接⽀配节点，那么⽀配X的节点也可以⽀配Y。
    - ⽀配树中的边并不直接对应于对象图中的对象引⽤。

  ![1626582280791](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1626582280791.png)

- MAT菜单功能介绍：

  - inspector：透视图，⽤于展示⼀个对象的详细信息，例如内存地址、加载器名称、包名、对象名称、对象所属的类的⽗类、对象所属的类的加载器对象、该对象的堆内存⼤⼩和保留⼤⼩，gc root信息。下半部分展示类的静态属性和值、对象的实例属性和值、对象所属的类的继承结构。
  - Heap Dump History：列出最近分析过的⽂件。
  - 功能选择栏：从左到右依次是：概览、类直⽅图、⽀配树、OQL查询、线程视图、报告相关、详细功能。其中概览就是上图的这个⻚⾯，其他则提供了⼀些更细致的分析能⼒。总的来说，功能上和VisualVM⼤同⼩异，但分析得更加细致。
  - 饼图：展示retained size对象占⽤⽐例。
  - Actions：常⽤的内存分析动作。
    - Histogram：列出内存中的对象，对象的个数及其⼤⼩。点击后⽣成的报表：
      - Class Name ： 类名称，java类名。
      - Objects ： 类的对象的数量，这个对象被创建了多少个。
      - Shallow Heap ：⼀个对象内存的消耗⼤⼩，不包含对其他对象的引⽤。
      - Retained Heap ：是shallow Heap的总和，也就是该对象被GC之后所能回收到内存的总和。
    - Dominator Tree：列出最⼤的那些对象，以及他们为什么存活。
    - Top Consumers：打印最昂贵的对象，以内和包分组。
    - Duplicate Classes：检测被多个classloader加载的类。
  - Reports：报表功能，包括：
    - Leak Suspects：⾃动分析内存泄漏的原因，并能直接定位到Class，找到可能导致内存泄露的代码⾏数。
    - Top Components：列出占⽤超过1%的组件的报告信息。
  - Step by Step：
    - Top Components：分析从属于指定包或者class loader的对象。

![1626582745483](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1626582745483.png)

##### JITWatch

- JITWatch是JIT编译器的⽇志分析器与可视化⼯具，可⽤来检查内联决策、热点⽅法、字节码以及汇编的各种细节，经常和HSDIS配合使⽤，实际中用的也不是特别多。
  - HSDIS是⼀个HotSpot虚拟机即时编译代码的反汇编插件，它包含在HotSpot虚拟机的源码当中，在OpenJDK的⽹站也可以找到单独的源码下载，但并没有提供编译后的程序。
- 安装HSDIS后，启动应用，添加以下参数，用于收集反汇编日志，执行完后将会⽣成⼀个  /Users/itmuch.com/logfile.log ⽂件，⾥⾯包括了各种类编辑以及汇编信息。
  - UnlockDiagnosticVMOptions：开启诊断信息。
  - PrintAssembly：输出反汇编内容。
  - Xcomp：以编译模式启动，这样，⽆执⾏⾜够次数来预热即可触发即时编译。
  - LogCompilation：打印编译相关信息。
  - LogFile：指定⽇志⽂件。
  - TraceClassLoading：是否跟踪类的加载。

```java
java -XX:+UnlockDiagnosticVMOptions -XX:+PrintAssembly -Xcomp -XX:+LogCompilation -XX:LogFile=/Users/itmuch.com
/logfile.log -XX:+TraceClassLoading -jar xxx.jar

```

- 最后使用JITWatch可视化阅读⽇志，使用以下命令启动JITWatch，选择反汇编日志，点击start即可可视化地分析了。

```java
mvn clean compile exec:java

```

![1626583188575](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1626583188575.png)

### 3.3. JVM参数选项？

笔记时间：2021-07-18。

#### 标准选项

- 用于**执行常见操作**（比如检查JRE版本、设置类路径、启动详细输出等），各种厂牌的虚拟机都会支持。
- **格式不统一**，以java -help的结果为准。
- **常见的标准选项有**：

| JVM参数                          | 作用                                                         |
| -------------------------------- | ------------------------------------------------------------ |
| -class-path \| -classpath \|- cp | 指定JVM类搜索路径，多个路径之间以分号隔开。如果指定了-classpath，则JVM就忽略系统变量CLASSPATH中指定的路径。如果-classpath和CLASSPATH都没有指定，则JVM会从当前路径寻找class |
| -server                          | 以server模式启动JVM，与client模式恰好相反。适合生产环境，适用于服务器。64位的JVM自动以server模式启动 |
| -client                          | 以client模式启动JVM，这种方式启动速度快，但运行时性能和内存管理效率不高，适合客户端程序或者开发调试。64位的JVM不支持client模式 |
| -Dproperty=value                 | 设置系统属性值。其中， property是属性名称，value是属性值，如果value有空格，则需要使用双引号，比如-Dfoo=“foo bar” |
| -javaagent：jarpath[=options]    | 加载指定的Java编程语言代理                                   |
| -verbose：class                  | 显示类加载相关的信息，当报找不到类或者类冲突时，可用此参数来诊断 |
| -verbose：gc                     | 显示垃圾收集事件的相关信息                                   |
| -verbose：jni                    | 显示本机方法和其他Java本机接口（JNI）的相关信息              |
| -version                         | 展示JDK版本                                                  |

#### 附加选项

- JDK 11文档中称为**额外参数**，JDK 8文档中称为**非标准参数**，是**Hotspot虚拟机的通用选项**，其他厂牌的JVM不一定会支持，并且未来可能会发生变化。
- 附加选项都以**-X开头**，具体以java -x的结果为准。
- **常见的附加选项有**：

| JVM参数                  | 默认值                                                       | 作用                                                         |
| ------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| -Xcomp                   | 默认情况下，client模式下会解释执行10000次（< JDK 11），server模式下会解释执行10000次，并收集信息，此后才可能编译运行。 | 在第一次调用时强制编译方法。指定该选项将禁用解释方法调用。此外，还可使用-XX：CompileThreshold选项更改在编译之前解释执行方法的调用次数 |
| -Xint                    | -                                                            | 以解释模式运行                                               |
| -Xmixed                  | -                                                            | 热点方法以编译模式运行，其他方法以解释模式运行               |
| -Xloggc：option          | -                                                            | 将GC事件的相关信息记录到文件中                               |
| -Xnoclassgc              | -                                                            | 禁用类的垃圾收集。使用该参数可节省一些GC时间，缩短应用程序运行期间的停顿。但一旦使用该参数，那么应用程序中的类对象就会始终被视为活动对象，从而导致这块内存被永久占用。如果使用不当，将会导致内存溢出 |
| -Xshare：mode            | auto：尽可能使用CDS，是32位的Hotspot JVM的默认值；on：开启CDS。如果CDS无法被开启，将会打印错误信息；off：不适用CDS | 设置类数据共享模式（class data sharing，即CDS）。注意，此选项只应用于测试目的，并且可能由于操作系统使用地址空间布局随机化而导致间歇性故障，不应在生产环境中使用 |
| -XshowSettings：category | all：默认值，展示所有设置；locale：展示语言环境相关的设置；properties：展示系统属性相关的设置；vm：展示JVM的设置；system：展示Linux主机系统或者容器的配置 | 展示设置                                                     |
| -Xmn                     | -                                                            | 设置年轻代的初始值以及最大值，以字节为单位，也可在size后追加字母k或者K表示千字节，m或者M表示兆字节，g或者G表示千兆字节，例如-Xmn256m。此外，还可用-XX：NewSize设置年轻代初始大小，-XX：MaxNewSize设置年轻代最大大小 |
| -Xms                     | -                                                            | 设置堆内存的初始大小，以字节为单位。此值必须是1024的倍数且大于1MB。如果未设置此选项，则将堆内存初始大小设置为老年代和年轻代分配的大小之和。设置格式同-Xmn，例如-Xms6144K |
| -Xmx                     | -                                                            | 设置堆内存的最大大小，以字节为单位。此值必须是1024的倍数且大于2MB。等效于-XX：MaxHeapSize |
| -Xss                     | 默认值取决于平台，64位Linux：1024KB；64位 MacOS：1024KB；64位Oracle Solaris：1024KB；Windows：默认值取决于虚拟内存 | 设置线程栈大小，以字节为单位                                 |

#### 高级选项

- 高级选项是**为开发人员提供的选项**，用于调整Java **HotSpot虚拟机**操作的特定区域（这些区域通常具有特定的系统要求，并且可能需要对系统配置参数的特权访问），其他厂牌的JVM不一定会支持，并且未来可能会发生变化。
- 高级选项都以**-XX开头**，可以使用以下方法查看所支持的选项：

```java
1）解锁参数并打印：java -XX:+UnlockExperimentalVMOptions -XX:+UnlockDiagnosticVMOptions -XX:+PrintFlagsInitial

2）jhsdb clhsdb --pid 11666，进入交互页面后使用flags查看

```

- **使用格式**：
  - **boolean类型**：格式为-XX：（+/-），+表示将选项设置为true，-表示将选项设置为false。
  - **非boolean类型**：格式为-XX：选项=值。
- **常见的高级选项有**：

##### 常用高级运行时选项

用于控制HotSpot VM的运行时的各种行为。

| JVM参数                               | 默认值或者取值方式 | 作用                                                         |
| ------------------------------------- | ------------------ | ------------------------------------------------------------ |
| -XX：ActiveProcessorCount=n           | -                  | JVM使用多少个CPU核心，去计算用于执行垃圾收集或者ForkJoinPool线程池的大小 |
| -XX：InitiatingHeapOccupancyPercent=n | 45                 | 老年代大小到达该阈值，会触发G1 Mixed GC                      |
| -XX：LargePageSizeInBytes=n           | -                  | 设置用于Java堆的大页面尺寸，以字节为单位，其数值必须是2的幂次，也可在size后追加字母k或者K表示千字节，m或者M表示兆字节，g或者G表示千兆字节 |
| -XX：MaxDirectMemorySize=n            | -                  | 设置java.nio包的直接缓存区分配的最大总大小，以字节为单位，也可在size后追加字母k或者K表示千字节，m或者M表示兆字节，g或者G表示千兆字节 |
| -XX：MaxGCPauseMillis=n               | 200ms              | 期望的最大停顿时间                                           |
| -XX：OnError=string                   | -                  | 发生错误的时候做某事，string是一个或者多个命令，多个命令使用分号分隔，如果字符串包含空格，则必须将其用引号引起来。比如“gcore %p；dbx - %p”，表示当发生错误时，使用gcore命令创建核心dump文件 |
| -XX：OnOutOfMemoryError=string        | -                  | 当发生OOM异常时做模式，配置格式同-XX：OnError                |
| -XX：ParallelGCThreads=n              | -                  | 设置GC并行阶段的线程数                                       |
| -XX：+PrintCommandLineFlags           | 关闭               | 打印命令行标记                                               |
| -XX：ThreadStackSize=size             | -                  | 设置线程栈的大小，和-Xss等价                                 |
| -XX：-UseBiasedLocking                | -                  | 禁用偏向锁                                                   |
| -XX：-UseCompressedOops               | 启用               | 禁用压缩指针。此选项仅适用于64位JVM，当Java堆大小小于32GB时，将使用压缩指针。启用此选项后，对象引用将表示为32位偏移量，而非64位指针，这通常会在运行Java堆大小小于32GB的应用程序时提高性能。当Java堆大小大于32GB时，可使用-XX：ObjectAlignmentInBytes选项 |
| -XX：GCLogFileSize=n                  | 512KB              | 处理大型日志文件                                             |
| -XX：+UseLargePages                   | 关闭               | 启用大页面内存的使用                                         |
| -XX： VMOptionsFile=filename          | -                  | 允许用户在文件中指定VM选项。比如java -XX：VMOptionsFile=/var/my_vm_options_HelloWorld |
|                                       |                    |                                                              |
| 元空间参数                            |                    |                                                              |
| -XX：MetaspaceSize                    | 20.8MB             | 元空间的初始值，元空间占用达到该值就会触发垃圾回收，进行类的卸载，同时收集器会自动调整该值。如果能够释放空间，则会自动降低该值（减少空间浪费）；如果释放空间很少，则在不超过-XX：MaxMetaspaceSize的情况下，适当提高该值（保证有足够空间） |
| -XX：MaxMetaspaceSize                 | 受限于本地内存大小 | 元空间大小的最大值                                           |
| -XX：MinMetaspaceFreeRatio            | 40%                | 垃圾收集后，计算当前元空间的空闲百分比，如果小于该值，则增加元空间的大小（保证有足够空间） |
| -XX：MaxMetaspaceFreeRatio            | 70%                | 垃圾收集后，计算当前元空间的空闲百分比，如果大于该值，则减少元空间的大小（减少空间浪费） |
| -XX：MinMetaspaceExpansion            | 332.8KB            | 元空间增长时的最小幅度                                       |
| -XX：MaxMetaspaceExpansion            | 5.2MB              | 元空间增长时的最大幅度                                       |
|                                       |                    |                                                              |
| 直接内存参数                          |                    |                                                              |
| -XX：MaxDirectMemorySize              | -                  | 设置最大直接内存大小，对Unsafe不起作用，但对ByteBuffer有效   |
|                                       |                    |                                                              |
| TLAB参数（不建议修改）                |                    |                                                              |
| -XX：+UseTLAB                         | 开启               | 是否启用线程私有分配缓存区（Thread-Local Allocation Buffer） |
| -XX：MinTLABSize                      | 2048B              | 最小TLAB大小，单位字节                                       |
| -XX：+ResizeTLAB                      | 是                 | 是否动态调整TLAB的大小                                       |
| -XX：TLABRefillWasteFraction          | 64                 | 由于TLAB空间比较小，因此很容易装满。比如TLAB 100KB，已使用80KB，当需要再分配一个30KB的对象时，就无法分配到这个TLAB了。这时虚拟机会有两种选择，第一，废弃当前TLAB，这样就会浪费20KB的空间；第二，保留当前的TLAB并将这30KB的对象直接分配在堆上，这样将来有小于20KB的对象时，仍可以使用这块空间。实际上，JVM内部维护了一个叫做refill_waste的值，当请求对象大于refill_waste时，会在堆中分配；若小于该值，则会废弃当前TLAB，新建TLAB分配对象。可以用TLABRefillWasteFraction来调整该阈值，表示TLAB中允许产生这种浪费的比例，默认为64，即允许使用1/64的TLAB空间作为refill_waste。默认情况下，TLAB和refill_waste都会在运行时不断地调整，使系统的运行状态达到最优。如果想要禁用自动调整TLAB的大小，可以使用-XX：-ResizeTLAB禁用ResizeTLAB，并使用-XX：TLABSize手工指定一个TLAB的大小 |
| -XX：+TLABStats                       | 是                 | 是否提供详细的TLAB的统计信息                                 |
| -XX：TLABSize                         | 0                  | 设置TLAB的初始大小，如果设置为0，JVM会自动设置TLAB的初始大小 |
| -XX：TLABWasteTargetPercent           | 1                  | 允许TLAB占用Eden空间的百分比                                 |

##### 常用高级JIT编译器选项

用于控制HotSpot VM如果执行的JIT编译。

| JVM参数                                       | 默认值                                                   | 作用                                                         |
| --------------------------------------------- | -------------------------------------------------------- | ------------------------------------------------------------ |
| -XX：+BackgroundCompilation                   | 开启                                                     | 启用JIT后台编译                                              |
| -XX：CompileCommand=command，method[，option] | -                                                        | 在指定方法上执行指定command。command可选项为：break：在调试JVM时设置一个断点，以便在指定方法编译开始时停止； compileonly：排除所有未指定的所有方法；dontinline：防止内联指定方法；exclude：排除指定的方法；help：打印-XX： CompileCommand选项的帮助信息；inline：尝试内联指定的方法；log：排除指定方法以外的所有方法的编译日志记录（用-XX：+Log Compilation打印编译日志），默认情况下将对所有编译方法执行日志记录；option：将JIT编译选项传递给指定的方法，以代替最后一个参数（option）。编译选项设置在方法名称之后，且可指定多个编译选项，以逗号或者空格分隔；print：在编译指定的方法后打印生成的汇编代码；quiet：不打印编译命令，默认情况下，将显示使用-XX：CompileCommand选项指定的命令 |
| -XX：+DoEscapeAnalysis                        | 开启                                                     | 启用逃逸分析                                                 |
| -XX：+Inline                                  | 开启                                                     | 启用方法内敛                                                 |
| -XX：InlineSmallCode=n                        | 1000B                                                    | 指定内联的以编译的方法的最大代码大小，以字节为单位           |
| -XX：+LogCompliation                          | 关闭                                                     | 将编译活动记录到当前工作目录的hotspot.log文件中，也可用+XX：LogFile选项来指定其他日志文件路径和名称。该选项必须与-XX：+ UnlockDiagnosticVMOptions配合使用 |
| -XX：MaxInlineSize=n                          | -                                                        | 设置要内联的方法的最大字节码大小，以字节为单位               |
| -XX：+PrintAssembly                           | 关闭                                                     | 打印字节码和本地方法的汇编代码，需要HSDIS的支持。该选项必须与-XX：+ UnlockDiagnosticVMOptions配合使用 |
| -XX：+PrintCompaction                         | 关闭                                                     | 打印哪些方法被内联。该选项必须与-XX：+ UnlockDiagnosticVMOptions配合使用 |
|                                               |                                                          |                                                              |
| CodeCache参数                                 |                                                          |                                                              |
| -XX：ReservedCodeCacheSize=n                  | 不同版本不同，JDK 8 + 64位以及JDK 11 + 64位都是240MB     | 设置 JIT编译的代码的最大代码缓存大小，以字节为单位，最大不超过2GB，否则会产生错误。该配置不应小于-XX： InitialCodeCacheSize的值，以java -XX：PrintFlagsFianl \| grep ReservedCodeCacheSize的结果为准 |
| -XX：InitialCodeCacheSize=n                   | 不同的操作系统，以及不同的编译器的值也会不同，一般为48MB | 设置代码缓存区的初始大小，以java -XX：PrintFlagsFianl \| grep InitialCodeCacheSize的结果为准 |
| -XX：-PrintCodeCache                          | 关闭                                                     | 在JVM停止时打印代码缓存的使用情况                            |
| -XX：-PrintCodeCacheOnCompilation             | 关闭                                                     | 每当方法被编译后，就打印一下代码缓存区的使用情况             |
| -XX：+UseCodeCacheFlushing                    | 打开                                                     | 代码缓存区即将耗尽时，尝试回收一些早期编译但又很久没有被调用的方法 |
| -XX：-SegementedCodeCache                     | 关闭，表示使用整体的代码缓存区                           | 是否使用分段的代码缓存区                                     |

##### 常用高级可服务性选项

用于控制系统信息收集与调试支持。

| JVM参数                                  | 默认值                                | 作用                                                         |
| ---------------------------------------- | ------------------------------------- | ------------------------------------------------------------ |
| -XX：HeapDumpPath=path                   | -                                     | 指定堆Dump的文件路径，经常和-XX：+HeapDumpOnOutOf MemoryError选项配合使用 |
| -XX：LogFile=path                        | 在当前工作目录中创建，名为hotspot.log | 指定日志文件的路径，经常和-XX：+LogCompilation配合使用       |
| -XX：+Unlock ExperimentalVMOptions       | 关闭                                  | 用于解锁JVM实验性参数                                        |
| -XX：+UnlockDiagnosticVMOptions          | 关闭                                  | 用于解锁JVM诊断性参数                                        |
|                                          |                                       |                                                              |
| JDK 8日志参数                            |                                       |                                                              |
| -XX：+PrintFlagsInitial                  | 关闭                                  | 打印支持的高级选项，并展示默认值                             |
| -XX：+PrintGC                            | 关闭                                  | 输出GC日志                                                   |
| -XX：+PrintGCDetails                     | 关闭                                  | 打印GC详情                                                   |
| -XX：+PrintGCCause                       | 打开                                  | 是否在GC日志中打印造成GC的原因                               |
| -XX：+PrintGCID                          | 关闭                                  | 打印垃圾GC的唯一标识                                         |
| -XX：+PrintGCDateStamps                  | 关闭                                  | 以日期的格式输出GC的时间戳，如2013-05-04T21：53：59.234+0800 |
| -XX：+PrintGCTimeStamps                  | 关闭                                  | 以基准时间的格式，打印GC时间戳                               |
| -XX：+PrintHeapAtGC                      | 关闭                                  | 在GC前后打印堆信息                                           |
| -XX：+PrintHeapAtGCExtended              | 关闭                                  | 在开启PrintHeapAtGC的前提下，额外打印更多堆的相关信息        |
| -XX：+PrintGCApplicationStoppedTime      | 关闭                                  | 打印垃圾回收期间程序暂定的时间                               |
| -XX：+PrintGCApplicationConcurrentTime   | 关闭                                  | 打印每次垃圾回收前，程序未中断的执行时间，可与PrintGCApplicationStoppedTime配合使用 |
| -XX：+PrintClassHistogramAfterFullGC     | 关闭                                  | Full GC之后打印堆的直方图                                    |
| -XX：+PrintClassHistogramBeforeFullGC    | 关闭                                  | Full GC之前打印堆的直方图                                    |
| -XX：+PrintReferenceGC                   | 关闭                                  | 打印处理引用对象的时间消耗，需开启PrintGCDetails才有效       |
| -XX：+PrintTLAB                          | 关闭                                  | 查看TLAB空间的使用情况                                       |
| -XX：-UseGCLogFileRotation               | 关闭                                  | 轮换文件，日志文件达到一定大小后，就创建一个新的日志文件。需指定-Xloggc：时才有效 |
| -XX：GCLogFileSize                       | 8KB                                   | 设置单个日志文件的大小，需开启UseGCLogFileRotation才有效     |
| -XX：NumberOfGCLogFiles                  | 0，表示保留所有日志                   | 日志轮换时，保留几个日志文件                                 |
| -Xloggc：path                            | -                                     | 指定GC日志文件路径                                           |
| -XX：+PrintAdaptiveSizePolicy            | 关闭                                  | 某些GC收集器有自适应策略，自适应调整策略会动态调整Eden、Survivor、老年代的大小。使用该标记，可打印自适应调节策略的相关信息 |
| -XX：+PrintTenuringDistribution          | 关闭                                  | 查看每次Minor GC后新的存活周期的阈值                         |
| -XX：G1PrintRegionLivenessInfo           | -                                     | 标记阶段结束后打印所有Region的存活情况，需开启-XX：+UnlockDiagnosticVMOptions后才能使用 |
| -XX：+G1PrintHeapRegions                 | -                                     | 打印堆的区域上的分配和释放的信息，需开启-XX：+UnlockDiagnosticVMOptions后才能使用 |
| -XX：+PrintStringDeduplicationStatistics | -                                     | JDK 8u20开始，使用G1垃圾收集器，可支持-XX：+UseStringDeduplication开启字符串去重。可用-XX：+PrintStringDeduplicationStatistics打印字符串去重的统计信息 |
|                                          |                                       |                                                              |
| JDK 11统一日志管理（只剩下两个）         |                                       |                                                              |
| -XX：+PrintGC                            | 关闭                                  | 输出GC日志                                                   |
| -XX：+PrintGCDetails                     | 关闭                                  | 打印GC详情                                                   |

##### 常用高级垃圾收集选项

用于控制HotSpot VM如何执行垃圾收集

| 收集器                | 参数以及默认值                             | 备注                                                         |
| --------------------- | ------------------------------------------ | ------------------------------------------------------------ |
| Serial                | -XX：+UseSerialGC                          | 虚拟机在Client模式下的默认值，开启后，使用Serial + Serial Old的组合 |
| ParNew                | -XX：+UseParNewGC                          | 开启后，使用ParNew + Serial Old的组合                        |
|                       | -XX：ParallelGCThreads=n                   | 设置垃圾收集器在并行阶段使用的垃圾收集线程数，当逻辑处理器小于8时，n的值与逻辑处理器数量相同；如果逻辑处理器数量大于8时，则n的值大约为逻辑处理器数量的5/8，大多数情况下是这样，除了较大的SPARC系统，其中n的值约为逻辑处理器的5/16 |
| Parallel Scavenge     | -XX：+UseParallelGC                        | 虚拟机在Server模式下的默认值，开启后，使用Parallel Scavenge + Serial Old的组合 |
|                       | -XX：MaxGCPauseMillis=n                    | 收集器尽可能保证单次内存回收停顿的时间不超过这个值，但是并不保证不超过该值，只是尽可能 |
|                       | -XX：GCTimeRatio=n                         | 设置吞吐量的大小，取值范围为0~100，假设GCTimeRatio的值为n，那么系统将花费不超过1 / （1 + n）的时间用于垃圾收集 |
|                       | -XX：+UseAdaptiveSizePolicy                | 开启后，无需人工指定新生代的大小（-Xmn）、Eden和Survivor的比例（-XX：SurvivorRatio）以及晋升老年代对象的年龄（-XX：PretenureSizeThreshold）等参数，收集器会根据当前系统的运行情况自动调整 |
| Serial Old            | 无                                         | Serial Old是Serial的老年代版本，主要用于Client模式下的老年代收集，同时也是CMS在发生Concurrent Mode Failure时的后备方案 |
| Parallel Old          | -XX：+UseParallelOldGC                     | 开启后，使用Parallel Scavenge + Parallel Old的组合。Parallel Old是Parallel Scavenge的老年代版本，在注重吞吐量和CPU资源敏感的场合，可以优先考虑这个组合 |
| CMS                   | -XX：+UseConcMarkSweepGC                   | 开启后，使用ParNew + CMS的组合，Serial Old收集器将作为CMS收集器出现Concurrent Mode Failure失败后的后备收集器使用 |
|                       | -XX：CMSInitiatingOccupancyFraction=68     | CMS收集器在老年代空间被使用多少后触发垃圾收集，默认68%       |
|                       | -XX：+UseCMSCompactAtFullCollection        | 在完成垃圾收集后是否要进行一次内存碎片整理，默认开启         |
|                       | -XX：CMSFullGCsBeforeCompaction=0          | 在进行若干次Full GC后就进行一次内存碎片整理，默认为0         |
|                       | -XX：+UseCMSInitiatingOccupancyOnly        | 允许使用占用值作为启动CMS收集器的唯一标准，一般和CMSFullGCsBeforeCompaction配合使用。如果开启，那么当CMSFullGCsBeforeCompaction达到阈值就开始GC，如果关闭，那么JVM仅在第一次使用CMSFullGCsBeforeCompaction的值，后续则自动调整，默认关闭 |
|                       | -XX：+CMSParallelRemarkEnabled             | 重新标记阶段会并行执行，使用此参数可降低标记停顿，默认打开（仅适用于CMS + ParNew的GC） |
|                       | -XX：+CMSScavengeBeforeRemark              | 开启或关闭在CMS重新标记阶段之前的清除Young GC的尝试。新生代里一部分对象会作为GC Roots，让CMS在重新标记之前，做一次Young GC，而YGC能够回收掉新生代里大多数对象，这样就可以减少GC Roots的开销。因此，打开此开关，可在一定程度上降低CMS重新标记阶段的扫描时间，当然，开启此开关后，Young GC也会消耗一些时间。PS：开启此开关并不保证在标记阶段前一定会进行清除操作，生产环境建议开启，默认关闭 |
| CMS-Precleaning       | -XX：+CMSPrecleaningEnabled                | 是否启用并发预清理，默认开启                                 |
| CMS-AbortablePreclean | -XX：CMSScheduleRemarkEdenSizeThreshold=2M | 如果Eden区的内存使用超过该值，才可能进入并发可中止的预清理阶段 |
| CMS-AbortablePreclean | -XX：+CMSMaxAbortablePrecleanTime=5000     | 并发可终止的预清理阶段持续的最大时间                         |
| CMS                   | -XX：+CMSClassUnloadingEnabled             | 使用CMS时，是否启用类卸载，默认开启                          |
|                       | -XX：+ExplicitGCInvokesConcurrent          | 显示调用System.gc（）会触发Full GC，会有Stop The World，开启此参数后，可让System.gc（）触发的垃圾回收变成一次普通的CMS GC |
| G1                    | -XX：+UseG1GC                              | 使用G1收集器                                                 |
|                       | -XX：G1HeapRegionSize=n                    | 设置每个 Region的大小，该值必须为2的幂次，范围为1M到32M，如果不指定G1会根据堆的大小自动决定 |
|                       | -XX：MaxGCPauseMillis=200                  | 设置最大停顿时间，默认值为200毫秒                            |
|                       | -XX：G1NewSizePercent=5                    | 设置年轻代占整个堆的最小百分比，默认值为5，这是个实验参数，需要-XX：+UnlockExperimentalVMOptions解锁实验参数后，才能使用该参数 |
|                       | -XX：G1MaxNewSizePercent=60                | 设置年轻代占整个堆的最大百分比，默认值为60，这是个实验参数，需要-XX：+UnlockExperimentalVMOptions解锁实验参数后，才能使用该参数 |
|                       | -XX：ParallelGCThreads=n                   | 设置垃圾收集器在并行阶段使用的垃圾收集线程数，当逻辑处理器小于8时，n的值与逻辑处理器数量相同；如果逻辑处理器数量大于8时，则n的值大约为逻辑处理器数量的5/8，大多数情况下是这样，除了较大的SPARC系统，其中n的值约为逻辑处理器的5/16 |
|                       | -XX：ConcGCThreads=n                       | 设置垃圾收集器并发阶段使用的线程数量，设置n大约为ParallelGCThreads的1/4 |
|                       | -XX：InitiatingHeapOccupancyPercent=45     | 老年代大小达到该阈值，则会触发Mixed GC，默认值为45           |
|                       | -XX：G1MixedGCLiveThresholdPercent=85      | Region中的对象，活跃度低于该阈值，才可能被包含在Mixed GC收集周期中，默认值为85，这是个实验参数，需要-XX：+UnlockExperimentalVMOptions解锁实验参数后，才能使用该参数 |
|                       | -XX：G1HeapWastePercent=5                  | 设置浪费的堆内存百分比，当可回收百分比小于浪费百分比时，JVM就不会启动Mixed GC，从而避免昂贵的GC开销。此参数相当于用来设置允许垃圾对象占用内存的最大百分比。 |
|                       | -XX：G1MixedGCCountTarget=8                | 设置在标记周期完成之后，最多执行多少个Mixed GC，默认值为8，启动多个Mixed GC可以缩短老年代的收集时间 |
|                       | -XX：G1OldCSetRegionThresholdPercent=10    | 设置在一次Mixed GC中被收集的老年代的比例上限，默认值为Java堆的10%，这是个实验参数，需要-XX：+UnlockExperimentalVMOptions解锁实验参数后，才能使用该参数 |
|                       | -XX：G1ReservePercent=10                   | 设置预留空闲内存百分比，虚拟机会保证Java堆有这么多空间可用，从而防止对象晋升时无空间可用而失败，默认值为Java堆的10% |
|                       | -XX：G1PrintHeapRegions                    | 输出Region被分配和回收的信息，默认为false                    |
|                       | -XX：G1PrintRegionLivenessInfo             | 在清理阶段的并发标记环节，输出堆中的所有Regions的活跃度信息，默认为fasle |
| Shenandoah            | -XX：+UseShenandoahGC                      | 使用Shenandoah收集器，这是个实验参数，需要-XX：+UnlockExperimentalVMOptions解锁实验参数后，才能使用该参数 |
| ZGC                   | -XX：+UseZGC                               | 使用ZGC收集器，这是个实验参数，需要-XX：+UnlockExperimentalVMOptions解锁实验参数后，才能使用该参数 |
| Epsilon               | -XX：+UseEpsilonGC                         | 使用Epsilon收集器，这是个实验参数，需要-XX：+UnlockExperimentalVMOptions解锁实验参数后，才能使用该参数 |

### 3.4. JVM线上故障排查？

#### 如何打印JVM日志？

- JDK 8垃圾收集日志打印参数：

打印GC明细、日期、系统相对时间戳、GC的原因、GC日志存储的位置。

```java
-Xms50m -Xmx50m -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintGCTimeStamps -XX:+PrintGCCause -Xloggc:/Users/itmuch.com/gclog.log
```

- JDK 11垃圾收集日志打印参数：

打印GC详情、GC日志存储的位置，使用-Xlog进行统一日志管理。

```java
-Xms50m -Xmx50m -Xlog:gc*=trace:file=/Users/itmuch.com/xloggc.log
```

- JDK 8运行时日志打印参数：

跟踪类加载的情况，以及偏向锁相关的日志。

```java
-XX:+TraceClassLoading -XX:+TraceBiasedLocking
```

- JDK 11运行时日志打印参数：

跟踪类加载的情况，以及偏向锁相关的日志，使用-Xlog进行统一日志管理。

```java
-Xlog:class+load=debug,biasedlocking=debug:file=/Users/itmuch.com/trace.log
```

#### 如果分析GC日志？

- **自动分析GC日志工具**：

  - **GCEasy（在线）**：<https://www.gceasy.io/>。

  ![1626740104774](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1626740104774.png)

  - **GC Viewer（老牌）**：<https://github.com/chewiebug/GCViewer>。

  ![1626740032908](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1626740032908.png)

  - **GCPlot（很久没维护了）**：<https://github.com/GCPlot/gcplot>。

- **手工分析，格式如下**：

##### Serial GC日志

除CMS、G1 GC日志与Serial GC（Serial + Serial Old）日志不太一样外，其他的格式都是类似的。

###### JDK 8 Serial GC日志

```java
# Serial GC收集器日志分析: -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+PrintGCDateStamps -XX:+UseSerialGC -Xmx50m -Xloggc:./gc_analysis.log

# JDK相关信息
Java HotSpot(TM) 64-Bit Server VM (25.91-b14) for windows-amd64 JRE (1.8.0_91-b14), built on Apr  1 2016 00:58:32 by "java_re" with MS VC++ 10.0 (VS2010)

# 内存相关信息
Memory: 4k page, physical 8266332k(2973848k free), swap 16532664k(9866032k free)

# 展示当前应用使用的JVM参数
CommandLine flags:
    -XX:-BytecodeVerificationLocal
    -XX:-BytecodeVerificationRemote
    -XX:InitialHeapSize=52428800
    -XX:+ManagementServer
    -XX:MaxHeapSize=52428800
    -XX:+PrintGC
    -XX:+PrintGCDateStamps
    -XX:+PrintGCDetails
    -XX:+PrintGCTimeStamps
    -XX:TieredStopAtLevel=1
    -XX:+UseCompressedClassPointers
    -XX:+UseCompressedOops
    -XX:-UseLargePagesIndividualAllocation
    -XX:+UseSerialGC

# 年轻代GC日志: 当前时间戳(PrintGCDateStamps):
    2021-04-15T20:36:08.848+0800:
# 相对时间戳(PrintGCTimeStamps):
    3.708:
# 造成GC的原因(PrintGCCause):...:
    [GC (Allocation Failure) 2021-04-15T20:36:08.849+0800: 3.709:
# 展示DefaultNew的回收前后的内存以及年轻代总内存大小(Serial DefNew)
    [DefNew: 13696K->1663K(15360K),
# GC (Allocation Failure) 到目前总共花费的时间
    0.0099516 secs]
# 展示回收前后整个堆内存以及堆内存总大小
    13696K->2410K(49536K),
# GC (Allocation Failure) 到目前总共花费的时间
    0.0111863 secs]
# 用户、系统、实际耗时
    [Times: user=0.00 sys=0.00, real=0.01 secs]

# FullGC日志: 当前时间戳(PrintGCDateStamps):
    2021-04-15T20:36:20.608+0800:
# 相对时间戳(PrintGCTimeStamps):
    15.466:
# 造成GC的原因(PrintGCCause):...:
    [Full GC (Metadata GC Threshold) 2021-04-15T20:36:20.608+0800: 15.466:
# 老年代回收之前和回收之后的内存大小以及老年代总内存大小
    [Tenured: 5749K->6890K(34176K),
# Full GC (Metadata GC Threshold) 到目前总共花费的时间
    0.0256217 secs]
# 展示回收前后整个堆内存以及堆内存总大小
    13665K->6890K(49536K),
# 元空间回收之前和回收之后的内存大小以及元空间总内存大小
    [Metaspace: 20533K->20533K(1067008K)],
# Full GC (Metadata GC Threshold) 到目前总共花费的时间
    0.0257088 secs]
# 用户、系统、实际耗时
    [Times: user=0.01 sys=0.00, real=0.02 secs]
```

###### JDK 11 Serial GC日志

```java
# JDK11 Serial GC收集器日志分析:

[0.104s][info][gc] Using Serial
# 内存概览: 堆内存地址、堆内存总大小、、压缩指针模式
[0.105s][info][gc,heap,coops] Heap address: 0x00000000fe200000, size: 30 MB, Compressed Oops mode: 32-bit
# 年轻代GC, 第1次回收为GC(0)
[1.846s][info][gc,start     ] GC(0) Pause Young (Allocation Failure)
# 年轻代回收前后的内存以及总内存大小
[1.862s][info][gc,heap      ] GC(0) DefNew: 8192K->1024K(9216K)
# 老年代回收前后的内存以及总内存大小
[1.862s][info][gc,heap      ] GC(0) Tenured: 0K->4482K(20480K)
# 元空间回收前后的内存以及总内存大小
[1.862s][info][gc,metaspace ] GC(0) Metaspace: 6131K->6131K(1056768K)
# 整个堆回收前后的内存以及总内存大小
[1.862s][info][gc           ] GC(0) Pause Young (Allocation Failure) 8M->5M(29M) 16.267ms
# 用户、系统、实际耗时
[1.862s][info][gc,cpu       ] GC(0) User=0.00s Sys=0.00s Real=0.02s
[4.813s][info][gc,start     ] GC(1) Pause Young (Allocation Failure)
[4.824s][info][gc,heap      ] GC(1) DefNew: 9216K->1024K(9216K)
[4.824s][info][gc,heap      ] GC(1) Tenured: 4482K->6236K(20480K)
[4.824s][info][gc,metaspace ] GC(1) Metaspace: 11473K->11473K(1060864K)
[4.824s][info][gc           ] GC(1) Pause Young (Allocation Failure) 13M->7M(29M) 11.722ms
```

##### CMS GC日志

使用ParNew + CMS的组合，Serial Old作为后备。

```java
# CMS GC收集器日志分析: -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+PrintGCDateStamps -XX:+UseConcMarkSweepGC -Xmx50m -Xloggc:./GC_CMS.log
# 结论: => 日志格式大体上与GC_Serial.log一致, 但增加了一些CMS的步骤描述

Java HotSpot(TM) 64-Bit Server VM (25.91-b14) for windows-amd64 JRE (1.8.0_91-b14), built on Apr  1 2016 00:58:32 by "java_re" with MS VC++ 10.0 (VS2010)
Memory: 4k page, physical 8266332k(3336640k free), swap 16532664k(9654484k free)
CommandLine flags: -XX:-BytecodeVerificationLocal -XX:-BytecodeVerificationRemote -XX:InitialHeapSize=52428800 -XX:+ManagementServer -XX:MaxHeapSize=52428800 -XX:MaxNewSize=17477632 -XX:MaxTenuringThreshold=6 -XX:OldPLABSize=16 -XX:+PrintGC -XX:+PrintGCDateStamps -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:TieredStopAtLevel=1 -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:+UseConcMarkSweepGC -XX:-UseLargePagesIndividualAllocation -XX:+UseParNewGC 
2021-04-15T21:04:10.842+0800: 2.650: [GC (Allocation Failure) 2021-04-15T21:04:10.842+0800: 2.650: [ParNew: 13696K->1664K(15360K), 0.0128738 secs] 13696K->2445K(49536K), 0.0131776 secs] [Times: user=0.00 sys=0.00, real=0.01 secs]

# 1、初始标记
2021-04-15T21:04:22.827+0800: 14.635: [GC (CMS Initial Mark) [1 CMS-initial-mark: 7035K(34176K)] 8820K(49536K), 0.0007531 secs] [Times: user=0.00 sys=0.00, real=0.00 secs]
# 2、并发标记
2021-04-15T21:04:22.827+0800: 14.636: [CMS-concurrent-mark-start]
2021-04-15T21:04:22.847+0800: 14.655: [CMS-concurrent-mark: 0.019/0.019 secs] [Times: user=0.06 sys=0.03, real=0.02 secs]
# 3、并发预清理 -> (4、并发可中止清理)
2021-04-15T21:04:22.847+0800: 14.655: [CMS-concurrent-preclean-start]
2021-04-15T21:04:22.848+0800: 14.656: [CMS-concurrent-preclean: 0.001/0.001 secs] [Times: user=0.00 sys=0.00, real=0.00 secs]
# 5、重新标记
2021-04-15T21:04:22.848+0800: 14.656: [GC (CMS Final Remark) [YG occupancy: 2776 K (15360 K)]2021-04-15T21:04:22.848+0800: 14.656: [Rescan (parallel) , 0.0009519 secs]2021-04-15T21:04:22.849+0800: 14.657: [weak refs processing, 0.0000363 secs]2021-04-15T21:04:22.849+0800: 14.657: [class unloading, 0.0032038 secs]2021-04-15T21:04:22.853+0800: 14.660: [scrub symbol table, 0.0038571 secs]2021-04-15T21:04:22.856+0800: 14.664: [scrub string table, 0.0003238 secs][1 CMS-remark: 7035K(34176K)] 9811K(49536K), 0.0087240 secs] [Times: user=0.00 sys=0.00, real=0.01 secs]
# 6、并发清除
2021-04-15T21:04:22.857+0800: 14.665: [CMS-concurrent-sweep-start]
2021-04-15T21:04:22.860+0800: 14.668: [CMS-concurrent-sweep: 0.003/0.003 secs] [Times: user=0.00 sys=0.00, real=0.00 secs]
# 7、并发重置
2021-04-15T21:04:22.860+0800: 14.668: [CMS-concurrent-reset-start]
2021-04-15T21:04:22.860+0800: 14.668: [CMS-concurrent-reset: 0.000/0.000 secs] [Times: user=0.00 sys=0.00, real=0.00 secs]

# 1、初始标记
2021-04-15T21:04:27.076+0800: 18.884: [GC (CMS Initial Mark) [1 CMS-initial-mark: 21029K(34176K)] 22587K(49536K), 0.0011465 secs] [Times: user=0.00 sys=0.00, real=0.00 secs]
# 2、并发标记
2021-04-15T21:04:27.077+0800: 18.885: [CMS-concurrent-mark-start]
2021-04-15T21:04:27.113+0800: 18.921: [CMS-concurrent-mark: 0.036/0.036 secs] [Times: user=0.08 sys=0.01, real=0.04 secs]
# 3、并发预清理
2021-04-15T21:04:27.114+0800: 18.922: [CMS-concurrent-preclean-start]
2021-04-15T21:04:27.115+0800: 18.923: [CMS-concurrent-preclean: 0.001/0.001 secs] [Times: user=0.00 sys=0.00, real=0.00 secs]
# 4、并发可中止清理
2021-04-15T21:04:27.115+0800: 18.923: [CMS-concurrent-abortable-preclean-start]
2021-04-15T21:04:27.282+0800: 19.090: [GC (Allocation Failure) 2021-04-15T21:04:27.282+0800: 19.090: [ParNew: 14979K->1443K(15360K), 0.0043541 secs] 36009K->22692K(49536K), 0.0044774 secs] [Times: user=0.06 sys=0.00, real=0.00 secs] 
2021-04-15T21:04:27.418+0800: 19.226: [CMS-concurrent-abortable-preclean: 0.050/0.303 secs] [Times: user=0.47 sys=0.00, real=0.30 secs]
# 5、重新标记
2021-04-15T21:04:27.418+0800: 19.226: [GC (CMS Final Remark) [YG occupancy: 8840 K (15360 K)]2021-04-15T21:04:27.418+0800: 19.226: [Rescan (parallel) , 0.0028890 secs]2021-04-15T21:04:27.421+0800: 19.229: [weak refs processing, 0.0000623 secs]2021-04-15T21:04:27.421+0800: 19.229: [class unloading, 0.0035473 secs]2021-04-15T21:04:27.425+0800: 19.233: [scrub symbol table, 0.0088034 secs]2021-04-15T21:04:27.434+0800: 19.242: [scrub string table, 0.0005316 secs][1 CMS-remark: 21248K(34176K)] 30088K(49536K), 0.0162688 secs] [Times: user=0.01 sys=0.00, real=0.02 secs]
# 6、并发清除
2021-04-15T21:04:27.435+0800: 19.243: [CMS-concurrent-sweep-start]
2021-04-15T21:04:27.447+0800: 19.255: [CMS-concurrent-sweep: 0.012/0.012 secs] [Times: user=0.03 sys=0.00, real=0.01 secs]
# 7、并发重置
2021-04-15T21:04:27.447+0800: 19.255: [CMS-concurrent-reset-start]
2021-04-15T21:04:27.447+0800: 19.255: [CMS-concurrent-reset: 0.000/0.000 secs] [Times: user=0.00 sys=0.00, real=0.00 secs]

```

##### G1 GC日志

与Serial GC、CMS GC格式差异非常大。

```java
# G1 GC收集器日志分析: -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+PrintGCDateStamps -XX:+UseG1GC -Xmx50m -Xloggc:./GC_G1.log

Java HotSpot(TM) 64-Bit Server VM (25.91-b14) for windows-amd64 JRE (1.8.0_91-b14), built on Apr  1 2016 00:58:32 by "java_re" with MS VC++ 10.0 (VS2010)
Memory: 4k page, physical 8266332k(3458332k free), swap 16532664k(9495268k free)
CommandLine flags: -XX:-BytecodeVerificationLocal -XX:-BytecodeVerificationRemote -XX:InitialHeapSize=52428800 -XX:+ManagementServer -XX:MaxHeapSize=52428800 -XX:+PrintGC -XX:+PrintGCDateStamps -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:TieredStopAtLevel=1 -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:+UseG1GC -XX:-UseLargePagesIndividualAllocation

# 年轻代G1 GC: 下面的缩进表示日志的子任务
2021-04-15T21:12:09.426+0800: 2.494: [GC pause (G1 Evacuation Pause) (young), 0.0052693 secs]
   # 并发任务解释
   [Parallel Time: 4.1 ms, GC Workers: 4]
      # GC统计
      [GC Worker Start (ms): Min: 2493.6, Avg: 2493.6, Max: 2493.6, Diff: 0.0]
      # GC扫描对象统计
      [Ext Root Scanning (ms): Min: 0.0, Avg: 0.7, Max: 1.1, Diff: 1.1, Sum: 2.7]
      # Update Remembered Sets(指保存到堆中的区域跟踪引用 -> 保存到Update Buffers更新缓存中)
      [Update RS (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
      # Update Buffers数量统计
         [Processed Buffers: Min: 0, Avg: 0.0, Max: 0, Diff: 0, Sum: 0]
      # Remembered Sets扫描统计
      [Scan RS (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
      # Code Root扫描统计
      [Code Root Scanning (ms): Min: 0.0, Avg: 0.1, Max: 0.2, Diff: 0.2, Sum: 0.3]
      # 拷贝存活对象统计
      [Object Copy (ms): Min: 0.0, Avg: 2.2, Max: 3.1, Diff: 3.1, Sum: 8.6]
      # 中断统计
      [Termination (ms): Min: 0.0, Avg: 0.2, Max: 0.2, Diff: 0.2, Sum: 0.6]
         # 尝试中断统计
         [Termination Attempts: Min: 1, Avg: 1.3, Max: 2, Diff: 1, Sum: 5]
      # GC线程其他工作统计
      [GC Worker Other (ms): Min: 0.0, Avg: 1.0, Max: 4.0, Diff: 3.9, Sum: 4.1]
      # GC线程总工作统计
      [GC Worker Total (ms): Min: 4.0, Avg: 4.1, Max: 4.1, Diff: 0.1, Sum: 16.3]
      # GC线程结束时间
      [GC Worker End (ms): Min: 2497.6, Avg: 2497.7, Max: 2497.7, Diff: 0.1]
   # 串行任务, 修复Code Root耗时统计
   [Code Root Fixup: 0.1 ms]
   # 串行任务, 清除Code Root耗时统计
   [Code Root Purge: 0.0 ms]
   # 清除Card Table中的Dirty Card耗时统计
   [Clear CT: 0.0 ms]
   # 其他任务
   [Other: 1.0 ms]
      # Collection Set选择区域耗时统计
      [Choose CSet: 0.0 ms]
      # 对象引用处理耗时统计
      [Ref Proc: 0.8 ms]
      # 引用队列ReferenceQueue耗时统计
      [Ref Enq: 0.0 ms]
      [Redirty Cards: 0.0 ms]
      # 处理超大对象耗时统计
      [Humongous Register: 0.0 ms]
      [Humongous Reclaim: 0.0 ms]
      # 释放Collection Set耗时统计
      [Free CSet: 0.0 ms]
      # 各区域内存变化统计
   [Eden: 14.0M(14.0M)->0.0B(18.0M) Survivors: 0.0B->2048.0K Heap: 14.0M(50.0M)->2555.5K(50.0M)]
 # 用户、系统、实际耗时
 [Times: user=0.00 sys=0.00, real=0.01 secs]

# 最重要, 体现了G1 GC的过程
# 并发回收日志: 1、初始标记(stop the world)
2021-04-15T21:12:11.327+0800: 4.394: [GC pause (Metadata GC Threshold) (young) (initial-mark), 0.0047654 secs]
   [Parallel Time: 4.3 ms, GC Workers: 4]
      [GC Worker Start (ms): Min: 4394.3, Avg: 4394.3, Max: 4394.3, Diff: 0.0]
      [Ext Root Scanning (ms): Min: 0.9, Avg: 1.0, Max: 1.1, Diff: 0.2, Sum: 4.1]
      [Update RS (ms): Min: 0.6, Avg: 0.7, Max: 0.8, Diff: 0.2, Sum: 2.7]
         [Processed Buffers: Min: 2, Avg: 3.8, Max: 7, Diff: 5, Sum: 15]
      [Scan RS (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
      [Code Root Scanning (ms): Min: 0.0, Avg: 0.0, Max: 0.1, Diff: 0.1, Sum: 0.1]
      [Object Copy (ms): Min: 2.4, Avg: 2.5, Max: 2.5, Diff: 0.1, Sum: 9.9]
      [Termination (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
         [Termination Attempts: Min: 1, Avg: 1.0, Max: 1, Diff: 0, Sum: 4]
      [GC Worker Other (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
      [GC Worker Total (ms): Min: 4.2, Avg: 4.2, Max: 4.2, Diff: 0.0, Sum: 16.8]
      [GC Worker End (ms): Min: 4398.5, Avg: 4398.5, Max: 4398.5, Diff: 0.0]
   [Code Root Fixup: 0.0 ms]
   [Code Root Purge: 0.0 ms]
   [Clear CT: 0.1 ms]
   [Other: 0.4 ms]
      [Choose CSet: 0.0 ms]
      [Ref Proc: 0.2 ms]
      [Ref Enq: 0.0 ms]
      [Redirty Cards: 0.1 ms]
      [Humongous Register: 0.0 ms]
      [Humongous Reclaim: 0.0 ms]
      [Free CSet: 0.0 ms]
   [Eden: 5120.0K(26.0M)->0.0B(26.0M) Survivors: 4096.0K->4096.0K Heap: 14.9M(50.0M)->11.3M(50.0M)]
 [Times: user=0.06 sys=0.00, real=0.01 secs]
# 开始扫描初始标记阶段Survivor区的Root Region
2021-04-15T21:12:11.332+0800: 4.399: [GC concurrent-root-region-scan-start]
2021-04-15T21:12:11.338+0800: 4.405: [GC concurrent-root-region-scan-end, 0.0055232 secs]
# 2、并发标记
2021-04-15T21:12:11.338+0800: 4.405: [GC concurrent-mark-start]
2021-04-15T21:12:11.352+0800: 4.419: [GC concurrent-mark-end, 0.0143048 secs]
# 3、最终标记(stop the world)
2021-04-15T21:12:11.355+0800: 4.422: [GC remark 2021-04-15T21:12:11.355+0800: 4.422: [Finalize Marking, 0.0001519 secs] 2021-04-15T21:12:11.355+0800: 4.422: [GC ref-proc, 0.0006391 secs] 2021-04-15T21:12:11.356+0800: 4.423: [Unloading, 0.0037653 secs], 0.0048311 secs]
 [Times: user=0.00 sys=0.00, real=0.00 secs]
# 4、筛选回收(stop the world)
2021-04-15T21:12:11.360+0800: 4.427: [GC cleanup 12M->12M(50M), 0.0005645 secs]
 [Times: user=0.00 sys=0.02, real=0.00 secs] 
```

#### 如何定位CPU过高的地方？

- **定位问题代码的方法**：
  - 可以使用JMC MBean服务器实时查看：但需要应用开启JMX连接。
  - **也可以使用top + jstack配合查看**：

1. top查看当前CPU占用率最高的进程，拿到占用率最高的进程号36032：

   ```java
   top
   ```

2. top -Hp查看该进程线程的运行信息，拿到占用率最高的线程号36044：

   ```java
   top -Hp 36032
   ```

3. printf得到该线程号36044的16进制数8ccc，用于搜索dump文件：

   ```java
   printf %x 36044
   ```

4. jstack dump出进程36032所有的线程栈，得到文件1.txt：

   ```java
   jstack -l 36032 > 1.txt
   ```

5. cat搜索线程栈文件1.txt，找到16进制为8ccc（即线程号为36044），并往后再搜索30行的内容：

   ```java
   cat 1.txt | grep -A 30 8ccc
   ```

6. 定位该线程中的出问题代码，并分析原因：原来是有个for循环，导致CPU过高。

```java
@Override
public void run() {
    while (true) {
        double a = Math.random() * Math.random();
        System.out.println(a);
    }
}
```

- **CPU过高的场景与解决方案**？
  - **无限while循环**：
    - 尽量避免无限循环。
    - 也可以让循环执行得慢点，比如sleep（）或者yeild（）。
  - **频繁GC**：
    - 尽量降低GC频率。
  - **频繁创建新的对象**：
    - 合理使用单例，避免频繁创建对象。
  - **频繁的线程上下文切换**：
    - 降低切换的频率，不过需要结合业务进行业务改造，而改造的难度取决于业务的复杂度。
  - 序列化和反序列化：
    - 原因：大多都是由于使用了不合理的类库导致的，比如XStream反序列化大对象，改用ObjectInputStream来解决问题。
    - 解决方案：使用合理的API来实现，选择好用的序列化与反序列化的类库。
  - 正则表达式：
    - 原因：由于正则表达式使用NFA自动机的引擎，在进行字符串匹配时会发生回溯，一旦发生回溯可能会导致CPU过高的问题。
    - 解决方案：改写正则表达式，降低回溯的发生。

#### 如何解决内存溢出问题？

##### 堆内存溢出

- **相关概念**：**堆**：Java Heap，线程共享，在JVM启动时创建，是Java虚拟机中内存最大的一块，**专门用来保存对象，几乎所有对象以及数组的内存都在堆上分配**。
- **定位问题代码的方法**：

1. 指定OOM溢出后转储堆Dump：

   真实环境下，堆内存溢出很可能会导致进程直接挂掉，根本不会打印堆栈日志，所以需要JVM发生异常时自动转储出堆Dump文件，以便出现问题后能够进行分析。

   ```java
   --XX:+HeapDumpOnOutOfMemoryError
   
   ```

2. 在项目根目录，找到堆Dump文件，使用MAT打开堆Dump文件：

   ![1626605721295](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1626605721295.png)

3. 点击Leak Suspects，分析内存泄露：

   ![1626605971298](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1626605971298.png)

4. 查看问题对象Object[]的details，分析with incoming references，即问题对象被引用的情况：

   此外，如果Leak Suspects有堆栈信息，也可以从堆栈信息中分析问题所在。

   ![1626606234660](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1626606234660.png)

5. 原来是因为无限循环插入了一堆随机String对象：

   ```java
   public class HeapOOMTest {
       private List<String> oomList = new ArrayList<>();
   
       public static void main(String[] args) {
           HeapOOMTest oomTest = new HeapOOMTest();
           while (true) {
               oomTest.oomList.add(UUID.randomUUID().toString());
           }
       }
   }
   
   ```

- **堆内存溢出的场景**？
  - **内存泄露**：
    - 可以借助MAT或者VisualVM，去查看泄露对象到对象的引用链，分析这个泄露的对象是通过哪个路径跟哪个对象关联，从而导致没有被回收掉。最后找到内存泄露对象的创建位置，优化相关的代码。
  - **存在生命周期或者数据结构不合理的对象**：
    - 更换不合理对象的生命周期或者数据结构。
  - **机器的堆内存大小设置得过小**：
    - 根据实际情况适当增大-Xms、-Xmn的值。

##### 栈内存溢出

- **相关概念**：在Hotspot虚拟机中，**栈内存是不允许扩展的**，且不区分虚拟机栈和本地方法栈，统一使用-Xss设置栈的大小，但同样会抛StackOverflowError异常，以及OutOfMemoryError异常。在有些VM中是有区分开的，比如使用-Xss设置虚拟机栈大小，-Xoss设置本地方法栈大小。
- **定位的方法**：栈溢出后，抛出的错误会导致进程的挂掉，还是可以在日志中打印出堆栈日志，这时分析日志文件即可定位到问题代码了。
- **栈内存溢出的场景**？
  - **递归调用深度过大**：
    - 原因：每递归调用一次，就会创建一个栈帧压入栈中，在栈容量有限的情况下，当容纳不了足够多的栈帧时，则会抛出StackOverflowError异常。
    - 解决方法：优化问题代码。
  - **方法内创建过多的局部变量**：
    - 原因：局部变量存放在局部变量表中，当栈中容纳不了这么多变量时也会抛出StackOverflowError异常。
    - 解决方法：优化问题代码。
  - **创建了过多的线程**：
    - 原因：栈是线程独享的，每个线程会创建自己的栈，当创建新线程的时候没有足够的内存去创建对应的栈，则会抛出**OutOfMemoryError**异常。
    - 解决方法：优化问题代码。
- **如何保证创建足够多的线程**？
  - **减少-Xss配置**：由于栈是线程独享的，每个线程会创建自己的栈，对于相同的内存总量，减少每个栈的大小，就可以创建更多的栈，在OOM之前就可以创建更多的线程了。
  - **增大栈能分配的内存**：尽量减少除了栈以外的内存占用，增大栈内存占用。其中公式为：
    - 栈内存 = 机器总内存 - 操作系统内存 - 堆内存 - 方法区内存 - 程序计数器内存 - 直接内存。 
  - **尽量杀死其他应用程序**：这样可以为目标应用腾出更多的内存。
  - **增大操作系统对线程数量的限制**：
    - sysctl -w kernel.threads-max：增大Linux系统支持的最大线程数（表示物理内存决定的理论系统进程数上限，一般会很大）。
    - sysctl -w kernel.pid_max：增大Linux系统限制某用户下最多可以运行多少进程或线程数。
    - sysctl -w vm.max_map_count：增大限制一个进程可以拥有的VMA(虚拟内存区域)的数量。
      - 虚拟内存区域是一个连续的虚拟地址空间区域。在进程的生命周期中，每当程序尝试在内存中映射文件，链接到共享内存段，或者分配堆空间的时候，这些区域将被创建。
    - ulimit -u：增大用户最多可启动的进程数目。

##### 方法区溢出

- **相关概念**：

  - Methed Area，别命Non-Heap（非堆），线程共享，是JVM规范中定义的一个逻辑概念，用于存储已被虚拟机加载的**类信息、常量、静态变量和即时编译后的代码**等数据，具体放在哪里，不同的实现可能会放在不同的地方。
  - 在JDK8以后，元空间替代了永久代，使得方法区与堆存在交集，静态变量和字符串常量池存放在堆中，类信息和运行时常量池放在元空间中，而静态常量池是class文件里的常量池，未加载前并不占用内存。

- **定位的方法**：方法区溢出后，抛出的错误会导致进程的挂掉，还是可以在日志中打印出堆栈日志，这时分析日志文件即可定位到问题代码了。

- **方法区内存溢出分类**：

  不同的JDK版本，方法区存放的结构不同，所以相同的代码导致的内存溢出抛出的异常信息也可能不同。

  - **永久代溢出 & 堆内存溢出**：
    - 对于小于JDK 7的版本，采用的是**永久代**存储符号引用、字符串以及类的静态变量，所以小于JDK 7的版本，在遇到字符串过多的情况下，是否溢出取决于**永久代**的大小。
    - 由于JDK 7把符号引用（native heap）、字符串常量池以及类的静态变量移动到了**堆**中，所以大于等于JDK 7的版本，在遇到字符串过多的情况下，是否溢出取决于**堆的大小**。
  - **永久代溢出 & 元空间溢出**：
    - 对于小于JDK 8的版本，采用的是**永久代**存储类信息，所以小于JDK 8的版本，在遇到类加载过多的情况下，是否溢出取决于**永久代**的大小。
    - 由于JDK 8中使用了**元空间**替代永久代，采用运行时常量池存储已加载的类的元数据信息，所以大于等于JDK 8的版本，在遇到类加载过多的情况下，是否溢出取决于**元空间**的大小，而元空间是放在**本地内存**上的，只要**机器内存**足够大，理论上是很难发生溢出的。

- **方法区内存溢出的场景**？

  - **常量池对象太大**：
    - **原因**：比如字符串过多。
    - **解决方案**：根据JDK版本，为（字符串）常量池预留足够的空间。
      - < JDK 7：增大PermSize、MaxPermSize。
      - 》= JDK7：增大-Xms、-Xmx。
  - **加载的类过多**：
    - **原因**：
      - **动态代理的操作库生成了大量的动态类**：比如CXF、XSource、CGLIB等动态代理的框架，因为增强的类越多，就需要更多的空间来存储类的定义信息。
      - **JSP过多**：因为JSP是在第一次被访问的时候，才会被编译成Java类，在极端场景下，访问过多的JSP页面，可能会打满方法区导致内存溢出。
      - 脚本语言动态类加载：比如Grovy脚本出现的动态类加载导致的元空间溢出的问题。
    - **解决方案**：
      - < JDK 8：增大PermSize、MaxPermSize。
      - 》= JDK 8：可以留空元空间相关的配置（本地内存实现，不配置时JVM会动态去分配），或者设置合理的元空间大小。

##### 直接内存溢出

- **相关概念**：
  - **直接内存**：DirectBuffer，是一块由操作系统直接管理的内存，也叫**堆外内存**，并不是JVM运行时数据区的一部分，也不是JVM规范中定义的内存区域，但这部分内存会被频繁使用，而且也可能会导致OOM错误的出现。
- **定位的方法**：直接溢出后，抛出的错误会导致进程的挂掉，还是可以在日志中打印出堆栈日志，这时分析日志文件即可定位到问题代码了。
  - java.lang.OutOfMemoryError：Unsafe导致直接内存溢出报错没有小尾巴。
  - java.lang.OutOfMemoryError: Direct buffer memory：ByteBuffer直接内存溢出报错是有小尾巴的。
  - **（经验之谈）如果堆Dump文件看不出问题或者太小，可考虑是直接内存溢出的问题**。
- **直接内存溢出的场景**？
  - **内存泄露多导致的内存溢出**：
    - **原因**：分配对象到直接内存后，不使用时不释放内存，然后继续分配对象，时间久了就会出现溢出问题。
    - **解决方案**：设置最大直接内存大小-XX：MaxDirectMemorySize，**对Unsafe不起作用**，但对ByteBuffer有效（会先设置long maxMemory = VM.maxDirectMemory（））。

##### 代码缓存区溢出

- **概念**：CodeCache，代码缓存区，是非堆区域，缓存的是JIT编译器编译后的代码（即机器码），以及部分JNI的机器码，不过JIT编译生成的机器码占主要部分。
  - 解释执行可以节省内存，不存放到CodeCache，立即执行。
  - 编译执行后的代码会存放在CodeCache里，虽然CodeCache在即将耗尽时会尝试回收，但满了后却会让JIT停止工作，此后已编译过的代码会继续以编译模式执行，还没有编译过的代码将会退化成以解释执行模式执行，从而出现系统运行变慢、响应时间增大的现象。
- **定位的方法**：
  - 使用jconsole连接进程观察CodeCache。
  - 日志打印出VM warning：CodeCache is full. Compiler has been disabled信息。
  - 项目平常性能OK，但突然出现性能下降，业务又没有问题时，可排查是否由代码缓存区溢出所导致。
- **代码缓存区溢出的场景**：
  - **单体项目过于庞大但CodeCache又设置得过小**。
    - **解决方案**：
      - 可以对照jconsole设置合理的-XX：ReservedCodeCacheSize（代码缓存区的最大大小）。
      - 而对于微服务等代码量不多的小应用来说，240MB默认的CodeCache一般都是够用的了。

#### 项目越跑越慢如何定位与解决？

可能的场景有：

- **Stop The World时间过长、GC频繁**：

  - **定位方法**：检查GC日志。
  - 解决方案？：增加-XX：ParallelGCThreads并行收集的线程数、根据实际情况更换垃圾收集器。

- **项目依赖的资源导致变慢**：

  - **定位方法**：检查数据库、网络等资源是否被其他程序占用了很多。
  - 解决方案？：释放调用问题的资源。

- **CodeCache满了**：会导致JIT从编译执行退化成了解释执行。

  - **定位方法**：使用jconsole检查CodeCache大小。

- **线程争抢过于激烈**：会导致目标线程抢不到CPU片。

  - **定位方法**：使用VisualVM检查目标进程中的线程运行情况、分析ThreadDump（比如可以使用FastThread、PerfMa进行可视化分析）。
    - 结果发现：在循环中创建了一堆线程池，且使用完了又没关闭掉，导致线程池争抢激烈，消耗CPU资源严重。
    - 解决方案：使用完线程池后，在finally代码块把线程池关闭掉；同时根据业务规则命名线程（new ThreadFactoryBuilder().setNameFormat("my-thread-pool-%d")），方便以后定位问题。

- 服务器问题（了解就好）：操作系统问题、其他进程争抢资源。

  如果一个实例发生了问题，根据情况选择，要不要着急去重启。如果出现的CPU、内存飙高或者日志里出现了OOM异常**第一步是隔离**，第二步是**保留现场**，第三步才是**问题排查**。

  - **隔离**：就是把你机器从请求列表里摘除，比如把nginx相关的权重设成零。

  - **保留现场**：

    1. **系统当前网络连接**：使用 ss 命令而不是netstat的原因是：netstat 在网络连接非常多的情况下，执行非常缓慢。后续的处理，可通过查看各种网络连接状态的梳理，排查 TIME_WAIT或者CLOSE_WAIT，或者其他连接过高的问题，非常有用。

       ```shell
       ss -antp > $DUMP_DIR/ss.dump 2>&1
       ```

    2. **网络状态统计**：

       ```shell
       # 它能够按照各个协议进行统计输出，对把握当时整个网络状态，有非常大的作用。
       netstat -s > $DUMP_DIR/netstat-s.dump 2>&1
       
       # 在一些速度非常高的模块上，比如 Redis、Kafka，就经常发生跑满网卡的情况。表现形式就是网络通信非常缓慢。
       sar -n DEV 1 2 > $DUMP_DIR/sar-traffic.dump 2>&1
       ```

    3. **进程资源**：通过查看进程，能看到打开了哪些文件，可以以进程的维度来查看整个资源的使用情况，包括每条网络连接、每个打开的文件句柄。同时，也可以很容易的看到连接到了哪些服务器、使用了哪些资源。这个命令在资源非常多的情况下，输出稍慢，请耐心等待。

       ```shell
       lsof -p $PID > $DUMP_DIR/lsof-$PID.dump
       ```

    4. **CPU 资源**：主要用于输出当前系统的 CPU 和负载，便于事后排查。

       ```shell
       mpstat > $DUMP_DIR/mpstat.dump 2>&1
       vmstat 1 3 > $DUMP_DIR/vmstat.dump 2>&1
       sar -p ALL  > $DUMP_DIR/sar-cpu.dump  2>&1
       uptime > $DUMP_DIR/uptime.dump 2>&1
       ```

    5. **I/O 资源**：一般，以计算为主的服务节点，I/O 资源会比较正常，但有时也会发生问题，比如**日志输出过多，或者磁盘问题**等。此命令可以输出每块磁盘的基本性能信息，用来排查 I/O 问题。在第 8 课时介绍的 GC 日志分磁盘问题，就可以使用这个命令去发现。

       ```shell
       iostat -x > $DUMP_DIR/iostat.dump 2>&1
       ```

    6. **内存问题**：free 命令能够大体展现操作系统的内存概况，这是故障排查中一个非常重要的点，比如 SWAP 影响了 GC，SLAB 区挤占了 JVM 的内存。

       ```shell
       free -h > $DUMP_DIR/free.dump 2>&1
       ```

    7. **其他全局**：dmesg 是许多静悄悄死掉的服务留下的最后一点线索。当然，ps 作为执行频率最高的一个命令，由于内核的配置参数，会对系统和 JVM 产生影响，所以我们也输出了一份。

       ```shell
       ps -ef > $DUMP_DIR/ps.dump 2>&1
       dmesg > $DUMP_DIR/dmesg.dump 2>&1
       sysctl -a > $DUMP_DIR/sysctl.dump 2>&1
       ```

    8. **进程快照**：此命令将输出 Java 的基本进程信息，包括**环境变量和参数配置**，可以查看是否因为一些错误的配置造成了 JVM 问题。

       ```shell
       ${JDK_BIN}jinfo $PID > $DUMP_DIR/jinfo.dump 2>&1
       ```

    9. **dump堆信息**：jstat 将输出当前的 gc 信息。一般，基本能大体看出一个端倪，如果不能，可将借助 jmap 来进行分析。

       ```shell
       ${JDK_BIN}jstat -gcutil $PID > $DUMP_DIR/jstat-gcutil.dump 2>&1
       ${JDK_BIN}jstat -gccapacity $PID > $DUMP_DIR/jstat-gccapacity.dump 2>&1
       ```

    10. **堆信息**：jmap 将会得到当前 Java 进程的 dump 信息。如上所示，其实最有用的就是第 4 个命令，但是前面三个能够让你初步对系统概况进行大体判断。因为，第 4 个命令产生的文件，一般都非常的大。而且，需要下载下来，导入 MAT 这样的工具进行深入分析，才能获取结果。这是分析内存泄漏一个必经的过程。

        ```shell
        ${JDK_BIN}jmap $PID > $DUMP_DIR/jmap.dump 2>&1
        ${JDK_BIN}jmap -heap $PID > $DUMP_DIR/jmap-heap.dump 2>&1
        ${JDK_BIN}jmap -histo $PID > $DUMP_DIR/jmap-histo.dump 2>&1
        ${JDK_BIN}jmap -dump:format=b,file=$DUMP_DIR/heap.bin $PID > /dev/null  2>&1
        ```

    11. **JVM 执行栈**：

        ```shell
        # jstack 将会获取当时的执行栈。一般会多次取值，我们这里取一次即可。这些信息非常有用，能够还原 Java 进程中的线程情况。
        ${JDK_BIN}jstack $PID > $DUMP_DIR/jstack.dump 2>&1
        
        # 为了能够得到更加精细的信息，我们使用 top 命令，来获取进程中所有线程的 CPU 信息，这样，就可以看到资源到底耗费在什么地方了。
        top -Hp $PID -b -n 1 -c >  $DUMP_DIR/top-$PID.dump 2>&1
        ```

    12. **高级替补**：

        ```shell
        # 有时候，jstack 并不能够运行，有很多原因，比如 Java 进程几乎不响应了等之类的情况。我们会尝试向进程发送 kill -3 信号，这个信号将会打印 jstack 的 trace 信息到日志文件中，是 jstack 的一个替补方案。
        kill -3 $PID
        
        # 对于 jmap 无法执行的问题，也有替补，那就是 GDB 组件中的 gcore，将会生成一个 core 文件。我们可以使用如下的命令去生成 dump：
        gcore -o $DUMP_DIR/core $PID
        ${JDK_BIN}jhsdb jmap --exe ${JDK}java  --core $DUMP_DIR/core --binaryheap
        ```

    13. **内存泄漏的现象**：稍微提一下 jmap 命令，它在 9 版本里被干掉了，取而代之的是 jhsdb，你可以像下面的命令一样使用。一般内存溢出，表现形式就是 Old 区的占用持续上升，即使经过了多轮 GC 也没有明显改善。比如ThreadLocal里面的GC Roots，内存泄漏的根本就是，这些对象并没有切断和 GC Roots 的关系，可通过一些工具，能够看到它们的联系。

        ```shell
        jhsdb jmap  --heap --pid  37340
        jhsdb jmap  --pid  37288
        jhsdb jmap  --histo --pid  37340
        jhsdb jmap  --binaryheap --pid  37340
        ```

