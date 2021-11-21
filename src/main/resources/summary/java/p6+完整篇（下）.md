# 七、Dubbo篇

### 1.1. 什么是集群、分布式、SOA、微服务架构？

| 架构   | 概念                                                         | 本质                                                         |
| ------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 集群   | 不同服务器部署**同一套应用**服务对外提供访问，实现服务的负载均衡，或者主从等，指同一种组件的多个实例，形成逻辑上的整体，其中的单个节点就可以提供完整服务。 | 节点的物理形态                                               |
| 分布式 | 服务的**不同模块**部署在不同的服务器上，其中的单个节点并不能提供完整服务，需要多个节点间通过交换信息协调提供服务 | 节点间的工作方式                                             |
| SOA    | Service Oriented Architecture，**面向服务的架构**，一种设计方法，其中包含多个服务，服务之间通过相互依赖最终提供一系列的功能，一个服务通常以独立的形式存在于某个操作系统进程中，各个服务之间通过网络进行调用 | ESB **中心化实现**，各服务通过 ESB 进行交互，解决异构系统之间的连通性，通过协议转换、消息解析、消息路由把服务提供者的数据传送到服务消费者；但由于所有服务都依赖于 ESB，最终会导致其承压过重，成为整体系统的一个瓶颈 |
| 微服务 | 微服务强调业务需要彻底的**组件化和服务化**，使原有的单个业务系统被拆分成多个可以独立开发、设计、运行的小应用，而这些小应用之间通过服务完成交换和集成 | SOA 架构的一种变体，其中**去中心化实现**是在 SOA 上做的升华  |

### 1.2. 详细介绍 RPC？

#### 背景概念

- **背景**：部署在 A 服务器的一个应用，想要调用 B 服务器上应用提供的方法，由于两个程序不在同一个内存空间，所以不能直接发起调用，需要通过网络来表达调用的语义和传达调用的数据，而 RPC 正是用来解决这种调用过程。
- **概念**：RPC，Remote Procedure Call，远程过程调用，是⼀种**技术思想**，⽽⾮⼀种规范或协议，是一种通过网络，使得程序能够像访问本地系统资源一样，去访问远端系统资源。

#### 架构组件

一个基本的 RPC 架构里面应该至少包含以下 4 个组件：

- **Client**：客户端，即服务调用方，也叫服务消费者。
- **Client Stub**：客户端存根，存放服务端地址信息，将客户端的请求参数打包成网络消息，再通过网络传输发送给服务端。
- **Server Stub**：服务端存根，接收客户端发送过来的请求消息，然后进行解包，再调用本地服务进行处理。
- **Server**：服务端，服务的真正提供者。

![1636547867858](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1636547867858.png)

#### 实现基础

1. **高效的网络通信**：比如一般选择 Netty 作为网络通信框架。
   - **NIO 通信**：出于并发性能的考虑，传统阻塞式 IO并不合适，因此需要非阻塞的 NIO，比如可以选择 Netty 或者 MINA 来解决 NIO 数据传输的问题。
2. **高效的序列化框架**：比如 Kryo、FastJson 和 Protobuf 序列化框架。
   - **序列化和反序列化**：在网络中，所有数据都将会被转化为**字节**进行传送，所以为了能够使参数能在网络中进行传输，需要对这些参数进行序列化和反序列化操作。
     - **序列化**：把对象转换为字节序列的过程称为**对象的序列化**，也就是编码的过程。
     - **反序列化**：把字节序列恢复为对象的过程称为**对象的反序列化**，也就是解码的过程。
3. **可靠的寻址方式**：指服务发现，比如可以使用 Zookeeper 来注册服务等。
   - **服务注册中心**：比如 Redis、Zookeeper、Consul 、Etcd 等，一般使用 ZooKeeper 来提供服务注册与发现功能，以及解决注册中心单点故障和分布式部署的问题。
4. **会话和状态保持**：如果是带会话（状态）的 RPC 调用，还需要有会话和状态保持的功能。
   - **长连接**。
5. **接口动态代理**：生成客户端存根 `Client Stub` 和服务端存根 `Server Stub` 时，需要用到动态代理技术，比如JDK 动态代理、CGLib 动态代理、Javassist 字节码生成。

![1636299221043](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1636299221043.png)

#### 调用过程

1. **客户端调用**：服务消费方通过调用本地服务的方式调用需要消费的服务。
2. **将方法入参等信息序列化**：客户端存根接收到调用请求后，负责将方法、入参等信息序列化，组装成能够进行网络传输的消息体。
3. **通过网络发送消息**：客户端存根找到远程的服务地址，并且将消息通过网络发送给服务端。
4. **发序列化操作**：服务端存根收到消息后进行反序列化解码操作。
5. **调用本地服务**：服务端存根根据解码得到的结果，调用本地的服务进行相关处理。
6. **服务端处理业务逻辑**：服务端本地服务执行具体业务逻辑。
7. **返回处理结果**：服务端把业务处理结果返回给服务端存根。
8. **返回消**息：服务端存根将返回结果重新序列化，打包成消息，并通过网络发送至消费方。
9. **反序列化操作**：客户端存根接收到消息，并进行反序列化解码操作。
10. **返回调用结果**：客户端存根返回解码得到的结果给服务消费方，服务消费方得到最终结果，完成一次 RPC 调用。

![1636298416654](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1636298416654.png)

#### 实现目标

RPC 框架的实现目标是，把以上调用过程的第 2~10 步，调用、编码/解码的过程给封装起来，让开发人员使用时，在感觉上就像调用本地服务一样地调用远程服务。

#### 常⻅框架

RPC 常见框架实现有：阿里 Dubbo & HSF、当当 Dubbox、京东 JSF、Google GRPC、Facebook 的 Thrift、Twitter Finagle、Spring Cloud 等。

- **Dubbo**：阿⾥集团开源的⼀个极为出名的 RPC 框架，底层使用 Netty 框架，基于 TCP 协议传输的，配合 Hession 序列化完成 RPC 通信，在很多互联⽹公司和企业应⽤中⼴泛使⽤，其协议和序列化框架都可以插拔是极其鲜明的特⾊。
- **Dubbox**：当当网基于 Dubbo 做的一个扩展项目，比如加了服务可 Restful 调用，更新了开源组件等。
- **GRPC**：Google 开源 RPC 框架，基于 HTTP 2.0 协议，底层使⽤ Netty 框架，⽀持常⻅的众多编程语⾔。
- **Thrift**：Facebook 开源 RPC 框架，主要是⼀个跨语⾔的服务开发框架，⽤户只要在其之上进⾏⼆次开发就⾏，应⽤对于底层的 RPC 通讯等都是透明的，不过这个对于⽤户来说需要学习特定领域语⾔这个特性，还是有⼀定成本的。
- **Spring Cloud**：基于 Http 协议 REST 接口调用进行远程过程通信，相对来说 TCP 实现的  RPC 请求会有更大的报文，占的带宽也会更多，但是 REST 相比 RPC 更为灵活，服务提供方和调用方的依赖只依靠一纸契约，不存在代码级别的强依赖。
  - Dubbo 是 SOA 时代的产物，以服务治理作为目标，关注点主要在于服务调用、流量分发、流量监控以及服务熔断。
  - Spring Cloud 诞生于微服务架构时代，依托了 Spring、Spring Boot，考虑的是微服务治理的方方面面，目的是打造一个生态。

#### RPC 与 HTTP 的区别

在大型网站内部子系统和接口都非常多的情况下，RPC 对比 Http 接口的优势有：

1. **长链接**：不必每次通信都要像 Http 那样去 3 次握手，减少了网络开销。
2. **服务治理**：RPC 框架一般都有注册中心，也有丰富的监控管理，来发布、下线接口、动态扩展等，这些对调用方来说是无感知、统一化的操作。

|              | RPC                                                          | HTTP                                                         |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 概念         | 远程过程调用，是⼀种**技术思想**，⽽⾮⼀种规范或协议         | 超长文本传输协议，是一种网络传输协议                         |
| 传输协议     | 基于 TCP 协议（网络 4 层），也可以基于 HTTP 协议             | 基于 HTTP 协议（网络 7 层）                                  |
| 传输效率     | 传输效率高，使⽤⾃定义的 TCP 协议，请求报⽂体积更⼩；或者使⽤ HTTP2 协议，也可以很好的减少报⽂的体积 | 如果是 HTTP1.1 协议，请求中会包含很多⽆⽤的内容；如果是 HTTP2.0 协议，进行简单封装可以作为⼀个 RPC 来使⽤，但对比标准 RPC 框架缺少了服务治理 |
| 性能消耗     | 可以基于 thrift 协议，实现⾼效的⼆进制传输                   | ⼤部分通过 json 来实现，字节⼤⼩和序列化都⽐ thrift 要更消耗性能 |
| 负载均衡     | RPC 框架基本都⾃带了负载均衡策略                             | 需要配置Nginx、HAProxy来实现                                 |
| 服务故障转移 | 能做到⾃动通知，不影响上游                                   | 需要事先通知，以修改下游的 Nginx、HAProxy 配置               |
| 总结         | 主要⽤于公司内部的服务调⽤，性能消耗低、传输效率⾼、服务治理⽅便 | 主要⽤于对外的异构环境，浏览器接⼝调⽤、APP接⼝调⽤、第三⽅接⼝调⽤等 |

### 1.3. 为什么要用 Dubbo？

####  单一应用架构

- **背景**：当⽹站流量很⼩时，只需⼀个应⽤，将所有功能都部署在⼀起，以减少部署节点和成本。
- **关键点**：此时，⽤于简化增删改查⼯作量的数据访问框架 **ORM** 是关键。

![1636254754534](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1636254754534.png)

#### 垂直应用架构

- **背景**：当访问量逐渐增⼤，单⼀应⽤增加机器带来的加速度越来越⼩，提升效率的⽅法之⼀是将应⽤拆成
  互不相⼲的⼏个应⽤，以提升效率。
- **关键点**：此时，⽤于加速前端⻚⾯开发的Web框架 **MVC** 是关键。

![1636254796925](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1636254796925.png)

#### 分布式服务架构

- **背景**：当垂直应⽤越来越多，应⽤之间交互不可避免，将核⼼业务抽取出来，作为独⽴的服务，逐渐形成
  稳定的服务中⼼，使前端应⽤能更快速的响应多变的市场需求。
- **关键点**：此时，⽤于提⾼业务复⽤及整合的分布式服务框架 **RPC** 是关键。

![1636254855881](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1636254855881.png)

#### 流动计算架构（SOA）

- **背景**：随着服务化的进⼀步发展，服务越来越多，服务之间的调⽤和依赖关系也越来越复杂，诞⽣了⾯向
  服务的架构体系（SOA），也因此衍⽣出了⼀系列相应的技术，如对服务提供、服务调⽤、连接处理、通信协议、序列化⽅式、服务发现、服务路由、⽇志输出等⾏为进⾏封装的**服务治理框架**，比如 Dubbo。

![1636254950341](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1636254950341.png)

### 1.4. 什么是 Dubbo？

Apache Dubbo，是一款高性能、轻量级的开源服务框架，提供了六大核心能力：面向接口代理的高性能 RPC 调用、智能负载均衡、服务自动注册与发现、高度可扩展能力、运行期流量调度、可视化的服务治理与运维。

#### 1、面向接口代理的高性能RPC调用

提供高性能的、基于代理的远程调用能力，服务以接口为粒度，为开发者屏蔽远程调用的底层细节。

- 默认使用 **Dubbo 协议**，基于 TCP 协议实现，有着报文小占用带宽小、基于网络 4 层通信效率高的优点。

#### 2、智能负载均衡

内置多种负载均衡策略，可以智能感知下游节点的健康状况，显著减少了调用延迟，提高了系统的吞吐量。

- **负载均衡**：当服务越来越多时，F5 硬件负载均衡器的单点压⼒也越来越⼤，通过在消费⽅获取服务提供⽅地址列表，实现软负载均衡和集群容错。

#### 3、服务自动注册与发现

支持多种注册中心服务，能够做到服务实例上下线的实时感知。

- **地址维护**：当服务越来越多时，服务 URL 配置管理变得⾮常困难，这时候需要⼀个服务注册中⼼，来动态注册和发现服务，使服务的位置透明。

#### 4、高度可扩展能力

遵循微内核+插件的设计原则，Microkernel 只负责组装 Plugin，其所有核心能力如 Protocol、Transport、Serialization 都被设计为扩展点，能够平等对待内置实现和第三方实现。

- **微内核**：Micro kernel，是提供操作系统核心功能的内核精简版本，设计成在很小的内存空间内增加移植性，提供模块化设计，以使用户安装不同的接口，如 DOS、Workplace OS、Workplace UNIX 等。

#### 5、运行期流量调度

内置条件、脚本等路由策略，通过配置不同的路由规则，轻松实现灰度发布、同机房优先等功能。

- 比如服务限流、服务降级、蓝绿发布、金丝雀发布、权重路由、同区域优先等。

#### 6、可视化的服务治理与运维

提供丰富服务治理、运维工具，可以随时查询服务元数据、服务健康状态以及调用统计，实时下发路由策略、调整配置参数。

1. 当调⽤量越来越⼤，服务容量问题就会暴露出来，服务需要多少机器⽀撑？什么时候该加机器？
2. 为了解决这些问题，第⼀步，要将服务当前每天的调⽤量、响应时间都统计出来，作为容量规划的参考指标。
3. 其次，进行动态调整权重，在线上将某台机器的权重⼀直加⼤，并在加⼤的过程中记录其响应时间的变化，直到响应时间到达阈值时，记录此时的访问量，再以此访问量乘以机器数反推总容量。

### 1.5. 什么是蓝绿发布、金丝雀发布、灰度发布、滚动发布？

| 发布类型   | 概念                                                         |
| ---------- | ------------------------------------------------------------ |
| 蓝绿发布   | 在线上老版本继续运行的前提下，直接发布新版本，然后进行测试；当新版本测试通过后，再将流量切到新版本，最后将老版本同时也升级到新版本 |
| 金丝雀发布 | 在线上版本可用的情况下，同时发布一个新版本应用作为**金丝雀**；测试该新版本的性能和表现，在保障整体系统稳定的前提下，可以尽早发现问题并及时做出调整 |
| 灰度发布   | 只升级部分服务，让一部分用户继续用老版本，一部分用户开始用新版本；如果用户对新版本没什么意见，再逐步扩大范围，最后将所有用户都迁移到新版本上面来 |
| 滚动发布   | 每次只升级一个或多个服务，升级完成后加入生产环境，不断执行这个过程，直到集群中的全部旧版本都升级到新版本为止 |

### 1.6. 什么是 Dubbo 核心组件？

![1636292050935](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1636292050935.png)

#### 节点角色

| 节点      | 角色                                 |
| --------- | ------------------------------------ |
| Container | 服务运行容器                         |
| Provider  | 暴露服务的服务提供方                 |
| Registry  | 服务注册与发现的注册中心             |
| Consumer  | 调用远程服务的服务消费方             |
| Monitor   | 统计服务调用次数和调用时间的监控中心 |

#### 调用关系

1. 服务容器负责启动、加载、运⾏服务提供者。

2. 服务提供者在启动时，向注册中⼼注册⾃⼰提供的服务。
3. 服务消费者在启动时，向注册中⼼订阅⾃⼰所需的服务。
4. 注册中⼼返回服务提供者地址列表给消费者，如果有变更，注册中⼼将会基于⻓连接推送变更数据给消费者。
5. 服务消费者，从提供者地址列表中，基于软负载均衡算法，选⼀台服务提供者进⾏调⽤，如果调⽤失败，再选另⼀台调⽤。
6. 服务消费者和提供者，在内存中累计调⽤次数和调⽤时间，定时每分钟发送⼀次统计数据到监控中⼼。

#### 架构优点

Dubbo 架构具有以下⼏个优点，分别是连通性、健壮性、伸缩性、以及面向未来架构的升级性。

##### 1、连通性

注册中⼼，服务提供者，服务消费者三者之间均为⻓连接，监控中⼼除外，而注册中⼼和监控中⼼都是可选的，服务消费者可以直连服务提供者。

- **服务提供者**：向注册中⼼注册其提供的服务，并汇报调⽤时间到监控中⼼，此时间不包含⽹络开销。
- **注册中⼼**：
  1. 负责服务地址的注册与查找，相当于⽬录服务，服务提供者和消费者只在启动时与注册中⼼交互，不转发客户端请求，压⼒较⼩。
  2. 通过⻓连接感知服务提供者的存在，当服务提供者宕机时，注册中⼼会⽴即推送事件通知消费者。
- **服务消费者**：向注册中⼼获取服务提供者地址列表，并根据负载算法直接调⽤提供者，同时汇报调⽤时间到监控中⼼，此时间包含⽹络开销。
- **监控中⼼**：负责统计各服务调⽤次数，调⽤时间等，统计先在内存汇总后每分钟⼀次发送到监控中⼼服务器，并以报表展示。

##### 2、健壮性

- 任意⼀台**注册中⼼**宕掉后，将会⾃动切换到另⼀台，全部宕掉后，服务提供者和服务消费者仍能通过本地缓存进行通讯，但不能注册新服务。
- ⽆任意⼀台**服务提供者**宕掉后，不会影响使⽤，全部宕掉后，服务消费者将⽆法使⽤，并⽆限次重连等待，直到服务提供者恢复。
- **监控中⼼**宕掉后，不影响使⽤，只是丢失部分采样数据而已。

##### 3、伸缩性

- **注册中心集群**：注册中⼼是对等集群，可动态增加机器部署实例，所有客户端将会⾃动发现新的注册中⼼。
- **服务提供者集群**：服务提供者⽆状态，可动态增加机器部署实例，注册中⼼将推送新的服务提供者信息给消费者。

##### 4、面向未来架构的升级性

当服务集群规模进⼀步扩⼤，带动 IT 治理结构进⼀步升级，需要实现动态部署，进⾏流动计算，而基于现有分布式服务架构，那时将不会带来阻⼒。下图是未来可能的⼀种架构：

![1636293350069](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1636293350069.png)

###### 节点角色

| 节点           | 角色                                           |
| -------------- | ---------------------------------------------- |
| **Admin**      | 统一管理控制台                                 |
| **Scheduler**  | 调度中心，可以基于访问压力，自动增减服务提供者 |
| **Deployer**   | 自动部署服务的本地代理                         |
| **Repository** | 版本仓库，用于存储服务应用发布所有代码包       |
| Container      | 服务运行容器                                   |
| Provider       | 暴露服务的服务提供方                           |
| Registry       | 服务注册与发现的注册中心                       |
| Consumer       | 调用远程服务的服务消费方                       |
| Monitor        | 统计服务调用次数和调用时间的监控中心           |

###### 调用关系

1. 统一管理控制台，不仅可以发布服务提供者的代码包到版本仓库，还可以通过大盘获取路由、流量统计等信息。
2. 调度中心会基于监控中心聚合过来的信息，进行动态增减服务提供者。
3. 当需要动态增加服务提供者时，会拉起新的服务运行容器。
4. 服务容器会先到版本仓库，拉取对应的代码包，然后负责启动、加载、运⾏服务提供者。
5. 服务提供者在启动时，向注册中⼼注册⾃⼰提供的服务。
6. 服务消费者在启动时，向注册中⼼订阅⾃⼰所需的服务。
7. 注册中⼼返回服务提供者地址列表给消费者，如果有变更，注册中⼼将会基于⻓连接推送变更数据给消费者。
8. 服务消费者，从提供者地址列表中，基于软负载均衡算法，选⼀台服务提供者进⾏调⽤，如果调⽤失败，再选另⼀台调⽤。
9. 服务消费者和提供者，在内存中累计调⽤次数和调⽤时间，定时每分钟发送⼀次统计数据到监控中⼼。

### 1.7. Dubbo 基本使用方式？

#### 原生 POM 依赖

```xml
<!-- Dubbo -->
<dependency>
    <groupId>org.apache.dubbo</groupId>
    <artifactId>dubbo</artifactId>
    <version>2.7.3</version>
</dependency>

<!-- 注册中心 -->
<dependency>
    <groupId>org.apache.dubbo</groupId>
    <artifactId>dubbo-registry-zookeeper</artifactId>
    <version>2.7.3</version>
</dependency>

<!-- 数据上报监控中心 -->
<dependency>
    <groupId>org.apache.dubbo</groupId>
    <artifactId>dubbo-metadata-report-zookeeper</artifactId>
    <version>2.7.3</version>
</dependency>
```

#### Provider | HelloWorld

##### XML 配置环境 & 接口

```xml
<beans "...">
    <!-- 应用配置 -->
    <dubbo:application name="demo-xml-provider" />
    <!-- 注册中心配置 -->                                         
    <dubbo:registry id="registry" address="zookeeper://127.0.0.1:2181"/>
    <!-- 协议配置 -->                                                         
    <dubbo:protocol name="dubbo" port="20880"/>
    <!-- 接口配置 -->
    <dubbo:service interface="com.end.dubbo.api.service.OrderService" ref="orderService"/>
   	<!-- SpringBean注入 -->
    <bean id="orderService" class="com.end.dubbo.service.OrderServiceImpl"/>
    <!-- 监控中心元数据上报配置 -->
    <dubbo:metadata-report address="zookeeper://127.0.0.1:2181"/>
</beans>
```

##### 实现类

```java
// 服务提供方
public class DubboXMLProvider {
    public static void main(String[] args) throws Exception {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("dubbo-provider.xml");
        context.start();

        System.out.println("xml provider 启动ing");
        System.in.read();
    }
}

// 订单服务
public class OrderServiceImpl implements OrderService {
    @Override
    public OrderDTO getOrder(String orderNo) {
        System.out.println("Provider OrderService getOrder start, orderNo:" + orderNo);
        OrderDTO orderDTO = new OrderDTO(1L, "no1", "订单1");
        System.out.println("Provider OrderService getOrder end, orderNo:" + orderNo + ", orderDTO:" + orderDTO.toString());
        return orderDTO;
    }
}
```

#### Consumer | HelloWorld

##### XML 配置环境 & 引用

```xml
<beans "...">
    <!-- SpringBean扫描 -->
    <context:component-scan base-package="com.end.dubbo.user.impl" />
	<!-- 应用配置 -->
    <dubbo:application name="demo-xml-consumer" />
	<!-- 注册中心配置 -->
    <dubbo:registry address="zookeeper://127.0.0.1:2181"/>
	<!-- 接口配置 -->
    <dubbo:reference id="orderService" interface="com.end.dubbo.api.service.OrderService" check="false"/>
</beans>
```

##### 调用类

```java
// user服务：服务消费方
public interface UserService {
    String getOrderInfo(String orderNo);
}
@Service
public class UserServiceImpl implements UserService {
    @Autowired
    private OrderService orderService;

    @Override
    public String getOrderInfo(String orderNo) {
        System.out.println("Consumer UserService getOrderInfo start, orderNo:" + orderNo);

        // 远程服务调用
        OrderDTO orderDTO = orderService.getOrder(orderNo);
        System.out.println("Consumer orderService.getOrderInfo end, orderNO:" + orderNo + ",orderDTO:" + orderDTO);
        return orderDTO.toString();
    }
}
```

#### SpringBoot POM 依赖

```xml
<!-- Dubbo、Spring整合 -->
<dependency>
    <groupId>org.apache.dubbo</groupId>
    <artifactId>dubbo-spring-boot-starter</artifactId>
    <version>2.7.3</version>
</dependency>

<!-- 注册中心 -->
<dependency>
    <groupId>org.apache.dubbo</groupId>
    <artifactId>dubbo-registry-zookeeper</artifactId>
    <version>2.7.3</version>
</dependency>

<!-- 数据上报监控中心 -->
<dependency>
    <groupId>org.apache.dubbo</groupId>
    <artifactId>dubbo-metadata-report-zookeeper</artifactId>
    <version>2.7.3</version>
</dependency>
```

#### SpringBoot Provider | HelloWorld

##### YML 配置环境

当然也可以用 XML 进行统一配置。

```yaml
dubbo:
  # 应用配置
  application:
    name: dubbo-annotation-provider
  # 注册中心配置
  registry:
    address: zookeeper://127.0.0.1:2181
  # 协议配置
  protocol:
    name: dubbo
    port: 20882
  # 数据上报监控中心
  metadata-report:
    address: zookeeper://127.0.0.1:2181
```

##### 注解配置接口

```java
// 订单服务
@org.apache.dubbo.config.annotation.Service(version = "1.0.0", group = "end1", methods = { 
    @Method(name = "getOrder", timeout = 1) 
})
public class OrderServiceImpl implements OrderService {
    @Override
    public OrderDTO getOrder(String orderNo) {
        System.out.println("Provider OrderService getOrder 2 start, orderNo:" + orderNo);
        OrderDTO orderDTO = new OrderDTO(1L, "no1", "订单11");
        System.out.println("Provider OrderService getOrder 2 end, orderNo:" + orderNo + ", orderDTO:" + orderDTO.toString());
        return orderDTO;
    }
}

// 服务提供方，启用Dubbo
@SpringBootApplication
@EnableDubbo
public class DubboAnnotationConsumerApplication {
    public static void main(String[] args) {
        SpringApplication.run(DubboAnnotationConsumerApplication.class, args);
    }
}
```

#### SpringBoot Consumer | HelloWorld

##### YML 配置环境

```yaml
dubbo:
  # 应用配置
  application:
    name: dubbo-annotation-consumer
  # 注册中心配置
  registry:
    address: zookeeper://127.0.0.1:2181
  # 协议配置
  protocol:
    name: dubbo
    port: 20882
  # 数据上报监控中心
  metadata-report:
    address: zookeeper://127.0.0.1:2181
```

##### 注解配置引用

```java
// user服务
@Service
public class UserServiceImpl implements UserService {
    @org.apache.dubbo.config.annotation.Reference(
        group = "end1", version = "1.0.0", timeout = 2000, loadbalance = "roundrobin", cluster = ""
    )
    private OrderService orderService;

    @Override
    public String getOrderInfo(String orderNo) {
        System.out.println("annotation Consumer UserService getOrderInfo start, orderNo:" + orderNo);

        OrderDTO orderDTO = orderService.getOrder(orderNo);
        System.out.println("annotation Consumer orderService.getOrderInfo end, orderNO:" + orderNo + ",orderDTO:" + orderDTO);
        return orderDTO.toString();
    }
}

// 服务消费方，启用Dubbo
@SpringBootApplication
@EnableDubbo
public class DubboAnnotationConsumerApplication {
    public static void main(String[] args) {
        SpringApplication.run(DubboAnnotationConsumerApplication.class, args);
    }
}
```

#### SpringBoot | Java Bean 配置1

```java
// 服务提供方Java Bean 配置1：注入实现类
@Configuration
@DubboComponentScan(basePackages = "com.end.dubbo.service.impl")
public class DubboDemoOtherProviderAPIConfig {
    @Bean
    public ServiceConfig<OrderService> orderServiceConfig(OrderService orderService){
        // 应用配置
        ApplicationConfig applicationConfig = new ApplicationConfig();
        applicationConfig.setName("dubbo-demo-other-provider-api");

        // 注册中心配置
        RegistryConfig registryConfig = new RegistryConfig();
        registryConfig.setProtocol("zookeeper");
        registryConfig.setAddress("127.0.0.1:2181");

        // 协议配置
        ProtocolConfig protocolConfig = new ProtocolConfig();
        protocolConfig.setName("dubbo");
        protocolConfig.setPort(20881);

        // 元数据上报配置
        MetadataReportConfig metadataReportConfig = new MetadataReportConfig();
        metadataReportConfig.setAddress("zookeeper://127.0.0.1:2181");

        // 接口方法配置
        MethodConfig methodConfig = new MethodConfig();
        methodConfig.setName("getOrder");
        methodConfig.setTimeout(1);
        
        // 接口配置
        ServiceConfig<OrderService> serviceConfig = new ServiceConfig<>();
        serviceConfig.setInterface(OrderService.class);
        serviceConfig.setRef(orderService);
        serviceConfig.setVersion("1.0.0");
        serviceConfig.setGroup("end4");
        serviceConfig.setApplication(applicationConfig);
        serviceConfig.setRegistry(registryConfig);
        serviceConfig.setProtocol(protocolConfig);
        serviceConfig.setMetadataReportConfig(metadataReportConfig);
		serviceConfig.setMethods(Lists.newArrayList(methodConfig));
        
        // 服务暴露
        serviceConfig.export();
        return serviceConfig;
    }
}
```

#### SpringBoot | Java Bean 配置2

```java
// 服务提供方Java Bean 配置2：注入实现类
@Configuration
@DubboComponentScan(basePackages = "com.end.dubbo.service.impl")
public class DubboDemoOtherProviderConfig {
    // 应用配置
    @Bean
    public ApplicationConfig applicationConfig(){
        ApplicationConfig applicationConfig = new ApplicationConfig();
        applicationConfig.setName("dubbo-demo-other-provider-api");
        return applicationConfig;
    }

    // 注册中心配置
    @Bean
    public RegistryConfig registryConfig(){
        RegistryConfig registryConfig = new RegistryConfig();
        registryConfig.setProtocol("zookeeper");
        registryConfig.setAddress("127.0.0.1:2181");
        return registryConfig;
    }

    // 协议配置
    @Bean
    public ProtocolConfig protocolConfig(){
        ProtocolConfig protocolConfig = new ProtocolConfig();
        protocolConfig.setName("dubbo");
        protocolConfig.setPort(20881);
        return protocolConfig;
    }

    // 元数据上报配置
    @Bean
    public MetadataReportConfig metadataReportConfig(){
        MetadataReportConfig metadataReportConfig = new MetadataReportConfig();
        metadataReportConfig.setAddress("zookeeper://127.0.0.1:2181");
        return metadataReportConfig;
    }

    // 服务配置
    @Bean
    public ServiceConfig<OrderService> orderServiceConfig(OrderService orderService){
        ServiceConfig<OrderService> serviceConfig = new ServiceConfig<>();
        serviceConfig.setInterface(OrderService.class);
        serviceConfig.setRef(orderService);
        serviceConfig.setVersion("1.0.0");
        serviceConfig.setGroup("end3");

        MethodConfig methodConfig = new MethodConfig();
        methodConfig.setName("getOrder");
        methodConfig.setTimeout(1);
        serviceConfig.setMethods(Lists.newArrayList(methodConfig));

        return serviceConfig;
    }
}
```

### 1.8. Dubbo 常用高级配置？

#### 开发与测试

##### 启动时检查

1. `check="true"` 默认会在应用启动时，检查依赖的服务是否可用，不可用则会抛出异常，阻止 Spring 初始化完成，以便上线前，能及早发现问题。
2. 可以通过 `check="false"` 关闭检查，比如测试时，有些服务不关心，或者出现了循环依赖，必须有一方先启动时。
3. 如果 Spring 容器是懒加载的，或者通过 API 编程延迟引用服务，请设置 `check=false`，否则由于拿到 null 引用，代表服务临时不可用时，会抛出异常，而如果 `check="false"`，当服务恢复时，能自动连上，总是会返回引用，而不会报错。

| 标签            | 属性  | 作用                                           |
| --------------- | ----- | ---------------------------------------------- |
| dubbo:reference | check | 关闭**某个服务**的启动时检查，没有提供者时报错 |
| dubbo:consumer  | check | 关闭**所有服务**的启动时检查，没有提供者时报错 |
| dubbo:registry  | check | 关闭**注册中心**启动时检查，注册订阅失败时报错 |

##### 点对点直连

点对点直连方式，可以以服务接口为单位，忽略注册中心的提供者列表，另外，A 接口配置点对点，并不影响 B 接口从注册中心获取列表。

- 在**开发及测试**环境下，经常需要绕过注册中心，只测试指定服务提供者，这时候可能需要点对点直连。
- 为了避免复杂化线上环境，不要在线上使用这个功能，只应在**测试阶段**使用。

| 标签            | 属性 | 作用                                                  |
| --------------- | ---- | ----------------------------------------------------- |
| dubbo:reference | url  | 配置 url 指向提供者，绕过注册中心，多个地址用分号隔开 |

##### 多版本

- 当一个接口实现，出现不兼容升级时，可以用版本号过渡，版本号不同的服务相互间不引用。
- 升级时，可以按照以下的步骤进行版本迁移：
  1. 在低压力时间段，先升级一半提供者为新版本。
  2. 再将所有消费者升级为新版本。
  3. 然后将剩下的一半提供者升级为新版本。

| 标签            | 属性    | 作用                         |
| --------------- | ------- | ---------------------------- |
| dubbo:service   | version | 提供服务的版本号，比如 1.0.0 |
| dubbo:reference | version | 消费服务的版本号，比如 1.0.0 |

##### 服务分组

当一个接口有多种实现时，可以用 group 区分，区分服务接口的不同实现。

| 标签            | 属性  | 作用           |
| --------------- | ----- | -------------- |
| dubbo:service   | group | 提供服务的分组 |
| dubbo:reference | group | 消费服务的分组 |

##### 泛化调用

Dubbo 支持 Json 字符串参数的泛化调用，即直接传递字符串来完成一次调用，无需具体的 JAR 包或接口。

- **优点**：扩展性、跨语⾔、轻量级。
- **使用场景**：⼀般使⽤在⽹关类项⽬中，在平常业务开发中基本不会使⽤。

###### API 方式

```java
// API方式 引用远程服务
// 该实例很重要，里面封装了所有与注册中心及服务提供方连接，请缓存
ReferenceConfig<GenericService> reference = new ReferenceConfig<GenericService>();
// 弱类型接口名
reference.setInterface("com.end.dubbo.api.service.OrderService");
// 声明为泛化接口
reference.setGeneric(true);
// 用com.alibaba.dubbo.rpc.service.GenericService可以替代所有接口引用
GenericService genericService = reference.get();
Object result = genericService.$invoke("getOrder", new String[] { "java.lang.String" }, new Object[]{ "ccc" });
```

###### Spring 注入方式

| 标签            | 属性    | 作用                                                      |
| --------------- | ------- | --------------------------------------------------------- |
| dubbo:reference | generic | 默认为 false，代表不启用泛化调用，true 则代表启用泛化调用 |

```java
// <dubbo:reference id="orderService"  // interface="com.end.dubbo.api.service.OrderService" generic="true"/>

// 通过spring
ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("dubbo-generic-consumer.xml");
GenericService orderService = (GenericService) context.getBean("orderService");
```

#### 服务高可用

##### 服务超时

| 标签            | 属性    | 作用                                                  |
| --------------- | ------- | ----------------------------------------------------- |
| dubbo:service   | timeout | 远程服务调用的超时时间，默认为 1000ms                 |
| dubbo:reference | timeout | 服务方法调用的超时时间，默认为 dubbo:consumer#timeout |
| dubbo:consumer  | timeout | 服务方法调用的超时时间，默认为 1000ms                 |

##### 重试次数

Dubbo 服务在尝试调用一次之后，如出现非业务异常，比如服务突然不可用、超时等，默认会进行额外的最多 2 次的重试。

| 标签           | 属性    | 作用              |
| -------------- | ------- | ----------------- |
| dubbo:consumer | retries | 默认额外重试 2 次 |

##### 集群容错

在集群**调用失败**时，Dubbo 提供 6 种集群容错方案，缺省为 `failover` 失败自动切换，可通过 SPI 自行扩展。

| 标签            | 属性    | 作用                   |
| --------------- | ------- | ---------------------- |
| dubbo:service   | cluster | 服务提供方配置集群模式 |
| dubbo:service   | weight  | 服务权重               |
| dubbo:reference | cluster | 消费方配置集群模式     |

##### 负载均衡

在集群负载均衡时，Dubbo 提供了  4 种负载均衡策略，缺省为 `random` 随机调用，可通过 SPI 自行扩展。

| 标签                         | 属性        | 作用           |
| ---------------------------- | ----------- | -------------- |
| dubbo:service                | loadbalance | 服务端服务级别 |
| dubbo:service#dubbo:method   | loadbalance | 服务端方法级别 |
| dubbo:reference              | loadbalance | 客户端服务级别 |
| dubbo:reference#dubbo:method | loadbalance | 客户端方法级别 |

##### 服务降级

服务降级，可以通过服务降级功能临时屏蔽某个出错的非关键服务，并定义降级后的返回策略。

```java
RegistryFactory registryFactory = ExtensionLoader.getExtensionLoader(RegistryFactory.class).getAdaptiveExtension();

Registry registry = registryFactory.getRegistry(URL.valueOf("zookeeper://10.20.153.10:2181"));

// 向注册中心写入动态配置覆盖规则
registry.register(URL.valueOf("override://0.0.0.0/com.foo.BarService?category=configurators&dynamic=false&application=foo&mock=force:return+null"));
```

| 降级策略               | 作用                                                         | 适用场景                                 |
| ---------------------- | ------------------------------------------------------------ | ---------------------------------------- |
| mock=force:return+null | 表示消费方对该服务的方法调用，都直接强制返回 null 值，不发起远程调用 | 用于屏蔽不重要服务不可用时对调用方的影响 |
| mock=fail:return+null  | 表示消费方对该服务的方法调用在失败后，才返回 null 值，而不抛异常 | 用来容忍不重要服务不稳定时对调用方的影响 |

##### 服务限流

| 标签            | 属性     | 作用                                                         |
| --------------- | -------- | ------------------------------------------------------------ |
| dubbo:service   | executes | 服务提供者的，每个服务的，每个方法的，最大可**并行**执行请求数 |
| dubbo:reference | actives  | 每个服务消费者的，每个服务的，每个方法的，最大**并发**调用数 |

#### 服务高性能

##### 请求派发策略

Dubbo 中的线程模型，可以通过不同的**请求派发策略**，和不同的**线程池类型**的组合，来应对不同的场景:

1. 如果事件处理的逻辑能**迅速完成**，并且不会发起新的 IO 请求，比如只是在内存中记个标识，则直接在 IO 线程上处理更快，因为减少了线程池调度。
2. 但如果事件处理**逻辑较慢**，或者需要发起新的 IO 请求，比如需要查询数据库，则必须派发到线程池，否则 IO 线程阻塞，将导致不能接收其它请求。
3. 如果用 IO 线程处理事件，又在事件处理过程中**发起新的 IO 请求**，比如在连接事件中发起登录请求，会报**“可能引发死锁”异常**，但不会真死锁。

| 标签           | 属性       | 作用         |
| -------------- | ---------- | ------------ |
| dubbo:protocol | dispatcher | 请求派发策略 |

##### 线程池类型

| 标签           | 属性       | 作用       |
| -------------- | ---------- | ---------- |
| dubbo:protocol | threadpool | 线程池类型 |
| dubbo:protocol | threads    | 最大线程数 |

##### 服务协议

###### 配置方式

| 标签          | 属性     | 作用                                                         |
| ------------- | -------- | ------------------------------------------------------------ |
| dubbo:service | protocol | 使用指定的协议暴露服务，在多协议时使用，值为 dubbo:protocol 的引用，并用逗号分隔 |

###### 协议汇总

| 协议       | 说明                                                         | 适用场景                                                     |
| ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Dubbo      | 单一长连接和 NIO 异步通讯，基于 TCP 协议，默认采用 Hessian 序列化，是 Dubbo 推荐使用的协议 | 适合大并发、小数据量的服务调用，消费者远大于提供者的场景     |
| RMI        | JDK 标准的 RMI 协议实现，采用 Java 标准序列化机制，传输参数和返回参数对象都需要实现 Serializable 接口，阻塞式短连接，基于 TCP 协议，数据包大小混合，可支持传输文件 | 多个短连接同步传输，适用常规的远程服务调用以及 RMI 互操作，但在依赖低版本的 Common-Collections 包时，Java 序列化存在安全漏洞 |
| WebService | 基于 HTTP#WebService 协议，需要集成 CXF 实现，提供与原生 WebService 的互操作 | 多个短连接同步传输，适用于系统集成和跨语言调用               |
| HTTP       | 基于 Http 协议，使用 Spring#HttpInvoke 实现                  | 适用于多个短连接、数据包大小混合、提供者个数多于消费者、需要给应用程序和浏览器 JS 调用的场景 |
| Hessian    | 集成 Hessian 服务，基于 HTTP 协议，采用 Servlet 暴露服务，Dubbo 内嵌 Jetty 作为服务器时的默认实现，提供与 Hession 服务互操作，采用 Hessian 序列化 | 多个短连接同步传输，适用于传入参数较大、提供者大于消费者、提供者压力较大的场景，比如传输文件 |
| Memcache   | 基于 Memcache实现的 RPC 协议                                 | -                                                            |
| Redis      | 基于 Redis 实现的RPC协议                                     | -                                                            |

###### 性能压测

1. 1k string 场景，压测得到的平均响应时间为：dubbo 2 > dubbo 1 > hessian > rmi > http。 

   ![1636884067979](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1636884067979.png)

2. 200k string 场景，压测得到的平均响应时间为：rmi > http > hessian > dubbo 2 > dubbo 1。 

   ![1636884139136](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1636884139136.png)

3. 因此，可见，小数据包传输时，Dubbo 2/Dubbo 1 性能最高，RMI/Http 性能最低；大数据包传输时，Http/RMI 性能最高，Dubbo 2/Dubbo 1 性能最低。

#### 配置总结

##### 配置方式汇总

XML 配置、注解配置、属性⽂件 properties/yaml 配置、-D JVM 参数配置、代码编码⽅式配置、配置中心配置。

##### XML 标签汇总

| 标签                   | 用途         | 解释                                                         |
| ---------------------- | ------------ | ------------------------------------------------------------ |
| 方法参数配置：         |              |                                                              |
| `<dubbo:argument/>`    | 参数配置     | 用于指定方法参数配置                                         |
| `<dubbo:method/>`      | 方法配置     | 用于 ServiceConfig 和 ReferenceConfig 指定方法级的配置信息   |
|                        |              |                                                              |
| 服务提供方配置：       |              |                                                              |
| `<dubbo:service/>`     | 服务配置     | 用于暴露一个服务，定义服务的元信息，一个服务可以用多个协议暴露，一个服务也可以注册到多个注册中心 |
| `<dubbo:provider/>`    | 提供方配置   | 当 ProtocolConfig 和 ServiceConfig 某属性没有配置时，采用此缺省值，可选 |
| `<dubbo:protocol/>`    | 协议配置     | 用于配置提供服务的协议信息，协议由提供方指定，消费方被动接受 |
|                        |              |                                                              |
| 服务消费方配置：       |              |                                                              |
| `<dubbo:reference/>`   | 引用配置     | 用于创建一个远程服务代理，一个引用可以指向多个注册中心       |
| `<dubbo:consumer/>`    | 消费方配置   | 当 ReferenceConfig 某属性没有配置时，采用此缺省值，可选      |
|                        |              |                                                              |
| 其他应用级配置：       |              |                                                              |
| `<dubbo:application/>` | 应用配置     | 用于配置当前应用信息，不管该应用是提供者还是消费者           |
| `<dubbo:registry/>`    | 注册中心配置 | 用于配置连接注册中心相关信息                                 |
| `<dubbo:monitor/>`     | 监控中心配置 | 用于配置连接监控中心相关信息，可选                           |
| `<dubbo:module/>`      | 模块配置     | 用于配置当前模块信息，可选                                   |

##### 配置方式覆盖策略

精确优先（比如方法大于全局等）、就近优先（比如 JVM 大于配置文件、消费者大于提供者等）。

1. **配置中心/-D JVM参数 优先**：这样可以使用户在部署和启动时，进行参数重写，比如在启动时需改变协议的端口。
2. **XML/注解/代码配置 次之**：如果在 XML 中有配置，则 dubbo.properties 中的相应配置项无效。
3. **properties/yml 最后**：相当于缺省值，只有 XML 没有配置时，dubbo.properties 的相应配置项才会生效，通常用于共享公共配置，比如应用名。

![1636881626159](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1636881626159.png)

##### XML  标签覆盖顺序

精确优先（比如方法大于全局等）、就近优先（比如 JVM 大于配置文件、消费者大于提供者等）。

1. **⽅法级优先**：当级别⼀样时，消费⽅优先，提供⽅次之。
   - 下面与这类似，提供方配置就像是消费方的缺省值，只有在无消费方配置时才会生效，这样可以留机会给消费方做配置改写。
   - 而全局配置就像是接口级的缺省值，接口级配置就像是方法级的缺省值，这样可以留机会到接口、方法层面做配置改写。
2. **接⼝级次之**：级别⼀样时，消费⽅优先，提供⽅次之。
3. **全局配置再次之**：级别⼀样时，消费⽅优先，提供⽅次之。

![1636882183556](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1636882183556.png)

### 1.9. 详细介绍 Dubbo 架构设计？

#### 整体设计

- 图中左边淡蓝背景的为**服务消费方**使用的接口，右边淡绿色背景的为**服务提供方**使用的接口，位于中轴线上的为双方都用到的接口。
- 图中从下至上分为十层，各层均为单向依赖，右边的黑色箭头代表层之间的**依赖关系**，每一层都可以剥离上层被复用，其中，Service 和 Config 层为 API，其它各层均为 SPI。
- 图中绿色小块的为**扩展接口**，蓝色小块为**实现类**，图中只显示用于关联各层的实现类。
- 图中蓝色虚线为**初始化过程**，即启动时组装链，红色实线为**方法调用过程**，即运行时调时链，紫色三角箭头为**继承**，可以把子类看作父类的同一个节点，线上的文字为调用的方法。

![1636607155860](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1636607155860.png)

#### 各层说明

1. **Service**：业务接⼝层，与业务逻辑相关，是**用户**根据 Provider 和 Consumer 的业务，设计对应的接⼝ `Interface` 和对应的实现 `Implement` 。
2. **config**：配置层，对外配置接口，可以直接初始化配置类，也可以通过 Spring 解析配置生成配置类。
3. **proxy**：服务代理层，服务接口透明代理，生成客户端和服务器端存根，封装了所有接口的透明化代理，再暴露给用户使用时，消费方使用 Proxy 将 `Invoker` 转成接口，提供方将接口实现转成 `Invoker`，所以，去掉 Proxy 层，RPC 是可以 Run 的，只是不那么透明，不那么看起来像调本地服务一样调远程服务而已。
4. **registry**：注册中心层，封装服务地址的注册与发现，Registry 实际上不算一层，而是一个独立的节点，只是为了全局概览，用层的方式画在了一起。
5. **cluster**：路由层，封装多个提供者的路由及负载均衡，并桥接注册中心，该层是一个外围概念，即将多个 `Invoker` 伪装成一个 `Invoker`，使得用户只要关注 Protocol 层 `Invoker` 即可，而加上 Cluster 或者去掉 Cluster 对其它层都不会造成影响，另外，只有一个提供者时，是不需要 Cluster 的。
6. **monitor**：监控层，记录 RPC 调用次数和调用时间的监控，Monitor 实际上不算一层，而是一个独立的节点，只是为了全局概览，用层的方式画在了一起。
7. **protocol**：远程调用层，核心层，封装 RPC 调用，只要有该层的 `Invoker` + `Exporter` 就可以完成非透明的 RPC 调用。
8. **exchange**：信息交换层，封装请求响应模式、同步转异步，在 transport 层上封装了 Request-Response 的语义，由于 Remoting 实现是 Dubbo 协议，如果选择了 RMI 协议，则整个 Remoting 都不会用上。
9. **transport**：网络传输层，负责消息传输，是对 Mina、Netty、Grizzly 的抽象，也可以扩展为 UDP 传输，由于 Remoting 实现是 Dubbo 协议，如果选择了 RMI 协议，则整个 Remoting 都不会用上。
10. **serialize**：数据序列化层，提供一些相关工具，默认序列化使用 Hessian2Serialization，另外还有 FastJsonSerialization、JavaSerialization、ProtostuffSerialization 等序列化方式。

#### 模块分包

![1636636045364](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1636636045364.png)

- **dubbo-common**：公共逻辑模块，包括 Util 类和通用模型。
  - `serialize` 层放在 common 模块中，以便更大程度复用。
- **dubbo-remoting**：远程通讯模块，Dubbo 协议的实现，如果 RPC 用 RMI协议，则不需要使用此包。
  - `transport` 层和 `exchange` 层都放在 remoting 模块中，作为 rpc 调用的通讯基础，由于 Remoting 实现是 Dubbo 协议，如果选择了 RMI 协议，则整个 Remoting 都不会用上。
- **dubbo-rpc**：远程调用模块，抽象各种协议，以及动态代理，只包含一对一的调用，不关心集群的管理。
  - `protocol` 层和 `proxy` 层都放在该模块中，这两层是 rpc 的核心，在不需要集群即只有一个提供者时，只使用这两层即完成 rpc 调用。
- **dubbo-cluster**：集群模块，将多个服务提供方伪装为一个提供方，包括：负载均衡、容错、路由等，集群的地址列表可以是静态配置的，也可以是由注册中心下发的。
  - `cluster` 层，属于一个外围概念，即加上 Cluster 或者去掉 Cluster 对其它层都不会造成影响，另外，只有一个提供者时，是不需要 Cluster 的。
- **dubbo-registry**：注册中心模块，基于注册中心下发地址的集群方式，以及对各种注册中心的抽象。
  - `Registry` 实际上不算一层，而是一个独立的节点，只是为了全局概览，用层的方式画在了一起。
- **dubbo-monitor**：监控模块，统计服务调用次数、调用时间和调用链跟踪。
  - `Monitor` 实际上不算一层，而是一个独立的节点，只是为了全局概览，用层的方式画在了一起。
- **dubbo-config**：配置模块，Dubbo 对外的 API，隐藏 Dubbo 所有细节，用户可通过 Config 使用 Dubbo。
  - `config` 配置层，对外配置接口，可以直接初始化配置类，也可以通过 Spring 解析配置生成配置类。
- **dubbo-container**：容器模块，是一个可独立启动的容器，以简单的 Main 加载 Spring 启动，因为服务通常不需要 Tomcat/JBoss 等 Web 容器的特性，因此没必要用 Web 容器去加载服务。
  - container 是服务容器，用于部署运行服务，所以没有在分层架构中画出。

#### 架构原理

![1636876443919](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1636876443919.png)

- Provider 启动时，把所有接口注册到注册中心，并且订阅动态配置 configurators，以及后台启动定时器，定时发送统计数据给 Monitor。
- Consumer 启动时，会订阅 providers、configurators、routers，在这些订阅内容发生变化时，ZK 会推送相关订阅的信息，并且与 Provider 建立长连接，方便以后数据通信，以及后台启动定时器，定时发送统计数据给 Monitor。

### 2.0. Dubbo 源码术语汇总？

| 术语          | 作用                                                         | 分类                                                         |
| ------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| @Adaptive     | 动态扩展一个类或者一个方法，1）注解在类上，代表类本身就是一个装饰类，只经过人工编码，不会生成代理类；2）注解在方法上，代表会自动生成和编译一个动态类，比如 Protocol$Adaptive，该类使用代码模板技术，通过 javassist 动态代理，生成装饰方法，该方法添加了根据URL参数获取 SPI 扩展类的逻辑 | 两种注解方式，注解在类上和注解在方法上                       |
| ObjectFactory | 为 Dubbo IOC 提供所有对象                                    | 1）ExtensionLoader 的ObjectFactory 为 null；2）其余的在 Spring 环境下为SpringExtensionFactory |
| proxyFatory   | 获取一个接口的代理类，有两个方，getInvoker()：针对Server端，用于把实现类包装成一个Invoker javasist代理对象，getProxy()：针对Client端，用于创建服务引用接口的代理对象 | JavassistProxyFactory、JdkProxyFactory                       |
| Wrapper       | 包装一个接口或者类，以实现通过wrapper复制或者定制方法调用    | 多种 Wrapper 结尾的扩展类                                    |
| Protocol      | 服务域，是 Invoker 暴露和引⽤的主功能⼊⼝，负责 Invoker 的⽣命周期管理，有两个方法：1）export：针对Server端，暴露远程服务，把实现类代理Invoker通过协议、打开Netty服务器暴露外部；2）refer：针对客户端，？？？ | InjvmProtocol、RegistryProtocol、DubboProtocol、MockProtocolRedisProtocol、ThriftProtocol、MemcachedProtocol 等 |
| Invoker       | 实体域，是 Dubbo 的核⼼模型，一个可执行对象，能够根据方法的名称、参数得到相应的结果 | 多种实现类，但主要包括3种类型：1）本地执行类型的Invoker；2）远程通信类型的Invoker；3）包含多个远程通信Invoker聚合成的集群类型Invoker |
| Invocation    | 会话域，持有调⽤过程中的变量，⽐如⽅法名、参数等             | RpcInvocation                                                |
| exporter      | 维护Invoker的生命周期，可在exporters缓存map中获取各种包装了Invoker的exporters | InjvmExporter、DubboExporter、DestroyableExporter            |
| exchanger     | 信息交换层，封装请求响应模式，比如利用Netty NIO特性，把同步操作转换为I/O监听的异步操作 | HeaderExchanger                                              |
| transporter   | 网络传输层，用来抽象Netty和Mina的统一接口                    | netty.NettyTransporter、netty4.NettyTransporter、MinaTransporter、GrizzlyTransporter |
| ZK 结点       | 持久化结点：一旦被创建，除非主动删除掉，否则一直存储在 ZK 中；临时结点：与客户端会话绑定，一旦客户端会话失效，该客户端所创建的所有临时结点都会被删除，进而触发监听器事件的回调 | CreateMode.PERSISTENT（持久化结点）、CreateMode.EPHEMERAL（非持久化结点） |
| Directory     | 目录服务，装载各种Invoker，为提供Cluster查找的数据           | StaticDirectory，静态目录服务，Invoker是固定的，用于最外层的固定封装；RegistryDirectory，注册目录服务，Invoker来源于ZK，由于实现了NotifyListener接口，在ZK 通知服务信息发生变更时，会回调notify方法，然后刷新对应的Invoker集合 |

### 2.1. 什么是 Dubbo SPI？

#### API 与 SPI 的区别？

![1636461572100](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1636461572100.png)

|          | API                                             | SPI                                                     |
| -------- | ----------------------------------------------- | ------------------------------------------------------- |
| 概念     | Application Programming Interface，应用程序接口 | Service Provider Interface，服务提供接口                |
| 接口实现 | 服务提供方，提供接口的不同实现                  | 客户消费方，不同消费方对同一接口，会提供不同的实现      |
| 接口调用 | 客户消费方，根据不同接口选择不同的实现          | 服务提供方，提供统一的接口                              |
| 应用场景 | 开发框架等提供的 API 支持                       | 数据库驱动、日志框架、Dubbo 扩展点、SpringBoot 自动装配 |

#### JDK SPI

##### 概念

一个简单的 JDK 服务提供者加载工具 `ServiceLoader`：

1. 通过在资源目录 `META-INF/services` 中，放置以 Provider 接口名作为配置文件名称，来标识 SPI 扩展点。
2. 其中，重复的扩展项将会被忽略。
3. 但是，`ServiceLoader` 非线程安全，不适用于多个并发线程。

##### 例子

###### 1、统一接口

```java
package jdk.spi;

// 动物接口
public interface Animal {
    void say();
}
```

###### 2、不同实现类

```java
// 猫实现类
public class Cat implements Animal {
    @Override
    public void say() {
        System.out.println("cat saying");
    }
}

// 狗实现类
public class Dog implements Animal {

    @Override
    public void say() {
        System.out.println("dog saying");
    }
}
```

###### 3、SPI 扩展配置

```properties
#./resources/META-INF/services/jdk.spi.Animal
jdk.impl.Dog
jdk.impl.Cat
```

###### 4、测试方法

```java
public class SpiTest {
    public static void main(String[] args) {
        ServiceLoader<Animal> serviceLoader = ServiceLoader.load(Animal.class);
        // dog saying，cat saying
        serviceLoader.forEach(Animal::say);
    }
}
```

##### 原理

1. 在 `ServiceLoader#load` ⽅法中，⾸先，会获取上下⽂类加载器，然后构造⼀个 `ServiceLoader`，而在 `ServiceLoader` 中有⼀个懒加载器，懒加载器会通过 `BufferedReader` 从 `META-INF/services` 路径下找到**对应的接⼝名**的全路径名⽂件，也就是SPI 扩展配置的⽂件。
2. 然后，通过⽂件的类解析器，读取⽂件中的内容。
3. 接着，再通过类加载器加载类的全路径，把所扩展的类加载到内存中。

```java
package java.util;

public final class ServiceLoader<S> implements Iterable<S>
{
	private static final String PREFIX = "META-INF/services/";

    // 1、使用当前线程的{@linkplain java.lang.Thread＃getContextClassLoader上下文类加载器}，为给定的服务类型创建一个新的服务加载器
    public static <S> ServiceLoader<S> load(Class<S> service) {
        ClassLoader cl = Thread.currentThread().getContextClassLoader();
        return ServiceLoader.load(service, cl);
    }
    
    // 2、为给定的服务类型和类加载器创建一个新的服务加载器。
    public static <S> ServiceLoader<S> load(Class<S> service,
                                            ClassLoader loader)
    {
        return new ServiceLoader<>(service, loader);
    }
    
    // 3、重新加载 -> 没作并发同步，所以非线程安全
    private ServiceLoader(Class<S> svc, ClassLoader cl) {
        service = Objects.requireNonNull(svc, "Service interface cannot be null");
        loader = (cl == null) ? ClassLoader.getSystemClassLoader() : cl;
        acc = (System.getSecurityManager() != null) ? AccessController.getContext() : null;
        reload();
    }
    public void reload() {
        providers.clear();
        lookupIterator = new LazyIterator(service, loader);
    }
    
    // 4、SPI迭代器
    private class LazyIterator implements Iterator<S> {
        Class<S> service;
        ClassLoader loader;
        
        private LazyIterator(Class<S> service, ClassLoader loader) {
            this.service = service;
            this.loader = loader;
        }
        
        // 5、迭代器遍历时调用
        private S nextService() {
            if (!hasNextService())
                throw new NoSuchElementException();
            String cn = nextName;
            nextName = null;
            Class<?> c = null;
            try {
            	// 6、加载SPI扩展类
                c = Class.forName(cn, false, loader);
            } catch (ClassNotFoundException x) {
                fail(service, "Provider " + cn + " not found");
            }
            if (!service.isAssignableFrom(c)) {
                fail(service, "Provider " + cn  + " not a subtype");
            }
            try {
            	// 7、实例化SPI扩展类，实例化完成后并加入缓存
                S p = service.cast(c.newInstance());
                providers.put(cn, p);
                return p;
            } catch (Throwable x) {
                fail(service,
                     "Provider " + cn + " could not be instantiated",
                     x);
            }
            throw new Error();          // This cannot happen
        }
    }
}
```

#### Dubbo SPI

#####  例子 | 无 IOC

###### 1、统一接口

```java
package dubbo.spi;

// 人类接口 => 只有@SPI接口才会被SPI到
@org.apache.dubbo.common.extension.SPI
public interface Person {
    void say();
}
```

###### 2、不同实现类

```java
// 男人实现类
public class Man implements Person {
    @Override
    public void say() {
        System.out.println("man saying");
    }
}

// 女人实现类
public class Woman implements Person {
    @Override
    public void say() {
        System.out.println("woman saying");
    }
}
```

###### 3、SPI 扩展配置

```properties
# ./resources/META-INF/services/dubbo.spi.Person => 支持键值对形式配置
man=dubbo.impl.Man
woman=dubbo.impl.Woman
```

###### 4、测试方法

```java
public class SpiTest {
    public static void main(String[] args) {
        ExtensionLoader<Person> extensionLoader = ExtensionLoader.getExtensionLoader(Person.class);
        // 支持只加载某个键下的扩展类 => man saying
        Person man = extensionLoader.getExtension("man");
        man.say();
    }
}
```

##### 例子 | 有 IOC

###### 1、依赖 woman、cat

```java
public class Man implements Person {
    // 测试 Person IOC 注入
    private Person personDelegate;
    public void setWoman(Person personDelegate) {
        this.personDelegate = personDelegate;
    }
    
    // 测试 Animal IOC 注入
    private Animal animalDelegate;
    public void setCat(Animal animalDelegate) {
        this.animalDelegate = animalDelegate;
    }

    @Override
    public void say() {
        // 测试 Person IOC 注入 => woman saying
        personDelegate.say();
        // 测试 Animal IOC 注入 => cat saying
        animalDelegate.say();
    }
}
```

###### 2、注入 woman

$@Adaptive 动态代理类，代码生成模板格式：根据URL参数获取SPI扩展类。

![1636878858821](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1636878858821.png)

```java
// .resources/META-INF/services/dubbo.spi.Person
// woman=dubbo.impl.Woman
@SPI
public interface Person {
    @Adaptive
    void say();
}

// @Adaptive注解在类上：代表类本身就是一个装饰类，只经过人工编码，不会生成代理类
// @Adaptive注解在方法上：代表会自动生成和编译一个动态的$Adaptive类，比如Protocol$Adaptive，该类使用代码模板技术，通过javassist动态代理，生成装饰方法，该方法添加了根据URL参数获取SPI扩展类的逻辑
@Adaptive
public class Woman implements Person {
    // 这里的Woman类明显不是装饰类，只是用于测试而已，下面的Cat类也一样
    @Override
    public void say() {
        System.out.println("woman saying");
    }
}
```

###### 3、注入 cat

```java
// .resources/META-INF/services/dubbo.spi.Animal
// cat=jdk.impl.Cat

@SPI
public interface Animal {
    void say();
}

@Adaptive
public class Cat implements Animal {
    @Override
    public void say() {
        System.out.println("cat saying");
    }
}
```

###### 4、测试方法

```java
public class SpiTest {
    public static void main(String[] args) {
        ExtensionLoader<Person> personExtensionLoader = ExtensionLoader.getExtensionLoader(Person.class);
        Person man = personExtensionLoader.getExtension("man");
        man.say();// woman saying、cat saying
    }
}
```

##### 原理

![1636877812612](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1636877812612.png)

```java
package org.apache.dubbo.common.extension;

public class ExtensionLoader<T> {
    // 1、获取type对应的getExtensionLoader
    public static <T> ExtensionLoader<T> getExtensionLoader(Class<T> type) {
        ...// SPI注解校验
        ExtensionLoader<T> loader = (ExtensionLoader<T>) EXTENSION_LOADERS.get(type);
        if (loader == null) {
            // 2、创建type对应的getExtensionLoader
            EXTENSION_LOADERS.putIfAbsent(type, new ExtensionLoader<T>(type));
            loader = (ExtensionLoader<T>) EXTENSION_LOADERS.get(type);
        }
        return loader;
    }
    
    // 3、根据key获取SPI扩展类
    public T getExtension(String name) {
        return getExtension(name, true);
    }
    public T getExtension(String name, boolean wrap) {
        ...// 4、先从缓存中获取
        // 5、缓存获取不到，则用双重检查锁实例化 => 线程安全
        if (instance == null) {
            synchronized (holder) {
                instance = holder.get();
                if (instance == null) {
                    instance = createExtension(name, wrap);
                    holder.set(instance);
                }
            }
        }
    }

    // 6、实例化key对应的SPI扩展类
    private T createExtension(String name, boolean wrap) {
        // 11、根据key从key-Classes形式缓存中获取对应的class
        Class<?> clazz = getExtensionClasses().get(name);
        ... 
        // 12、利用Dubbo SPI IOC 注入接口实现类的实例 -> 接口上要有@SPI、注入的实现类上要有@Adaptive
        injectExtension(instance);
        
        if (wrap) {
            ...
            // 13. 通过wrapper包装类实现 Dubbo SPI 对 AOP 的支持，即将instance作为参数传递给wrapper，通过反射创建wrapper，再向wrapper实例中注入依赖，最后把包装后的实例作为instance返回
            instance = injectExtension((T) wrapperClass.getConstructor(type).newInstance(instance));
        }
        ...
        return instance;
    }

    // 12.1、Dobbo SPI IOC 注入核心方法
    private T injectExtension(T instance) {
        // 反射获取所有方法
        for (Method method : instance.getClass().getMethods()) {
            // 12.2、要有set方法
            if (!isSetter(method)) {
                continue;
            }
            
            // 12.3、可以注入
            if (method.getAnnotation(DisableInject.class) != null) {
                continue;
            }
            
            // 12.4、如果参数是基础类型就不注入, 因为这里注入指的是将加载的对象注入进来
            Class<?> pt = method.getParameterTypes()[0];
            if (ReflectUtils.isPrimitives(pt)) {
                continue;
            }
            
            // 12.5、获取要注入的属性名 -> 从@SPI扩展类中，获取对应接口和有@Adaptive的实现类
            String property = getSetterProperty(method);
            // ObjectFactory的作用是，为Dubbo IOC提供所有对象
            Object object = objectFactory.getExtension(pt, property);
            if (object != null) {
                // 12.6、反射调用set方法，完成注入
                method.invoke(instance, object);
            }
        }
    }
    
    private Map<String, Class<?>> getExtensionClasses() {
        ...
        // 7、加载SPI目录中所有的类，key-Classes形式放入缓存再返回
        classes = loadExtensionClasses();
        ...
    }
    
    // 8、SPI类加载核心方法
    private Map<String, Class<?>> loadExtensionClasses() {
        // 设置注解名称
        cacheDefaultExtensionName();
        
        // 9、加载读取配置文件：META-INF/services/、META-INF/dubbo/、META-INF/dubbo/internal/
         Map<String, Class<?>> extensionClasses = new HashMap<>();
        for (LoadingStrategy strategy : strategies) {
      		// 10、底层调用loadResource -> loadClass -> 加载对应Class -> extensionClasses
            loadDirectory(extensionClasses, strategy.directory(), type.getName(), strategy.preferExtensionClassLoader(), strategy.overridden(), strategy.excludedPackages
            // com.alibaba => org.apache
            loadDirectory(extensionClasses, strategy.directory(), type.getName().replace("org.apache", "com.alibaba"), strategy.preferExtensionClassLoader(), strategy.overridden(), strategy.excludedPackages());
        }
        
        // key-Classes形式
        return extensionClasses;
    }
}
```

#### JDK SPI 与 Dubbo SPI 的区别？

|                | JDK SPI                                                      | Dubbo SPI                                                    |
| -------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 按需加载       | 不能按需加载，只能一次性加载所有的扩展实现，有的扩展加载很耗时，如果后面没用上，会很浪费资源，所以若想只加载某个扩展类的实现，使用 JDK SPI 就不现实了 | 可以按需加载，只加载⾃⼰想要加载的扩展实现，通过 key-全限定类名 配置扩展文件，最后用 key 来获取指定的扩展类 |
| IOC和 AOP 支持 | 类之间的依赖关系无法解决，无法把⼀个实现类，注⼊到容器中，不支持 IOC 和 AOP | 对扩展点支持 IOC 和 AOP，一个扩展点可以直接 `setter` 注⼊其它扩展点，可以很好的⽀持第三⽅ IOC 容器，比如`Spring Bean`，另外通过 Wrapper 包装类实现 Dubbo SPI 对 AOP 的支持 |
| 线程安全       | `ServiceLoader` 非线程安全，会有线程安全的问题，即并发时扩展类可能会加载错误 | `ExtensionLoader` 线程安全，对扩展类进行实例化时，使用双重检查锁保证线程安全 |

### 2.2. Dubbo 与 Spring 融合原理？

![1636889489122](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1636889489122.png)

#### 1、Spring 解析 XML 回调

基于 dubbo.jar 内的 `META-INF/spring.handlers` 配置，Spring 在遇到 Dubbo XML 配置的名称空间 `http\://dubbo.apache.org/schema/dubbo` 时，会回调 `DubboNamespaceHandler` ，而 `http\://dubbo.apache.org/schema/dubbo/dubbo.xsd` 则定义了 Dubbo XML 的标签语法。

```properties
# ../dubbo-2.7.3.jar!/META-INF/spring.handlers
http\://dubbo.apache.org/schema/dubbo=org.apache.dubbo.config.spring.schema.DubboNamespaceHandler

# ../dubbo-2.7.3.jar!/META-INF/spring.schemas
http\://dubbo.apache.org/schema/dubbo/dubbo.xsd=META-INF/dubbo.xsd

# Dubbo XML配置头内容
<beans ... 
# DubboNamespaceHandler
http://dubbo.apache.org/schema/dubbo
# dubbo.xsd
http://dubbo.apache.org/schema/dubbo/dubbo.xsd">
</bean>
```

#### 2、 XML 标签解析成 Bean 

所有 dubbo 的标签，都统一用 `DubboBeanDefinitionParser` 进行解析，基于一对一属性映射，将 XML 标签解析为 Bean 对象。

```java
package org.apache.dubbo.config.spring.schema;

// Dubbo XML配置统一解析器
public class DubboNamespaceHandler extends NamespaceHandlerSupport {
    @Override
    public void init() {
        ...
        // 这里主要看ServiceBean和ReferenceBean，其他的Dubbo配置解析也类似
        registerBeanDefinitionParser("service", new DubboBeanDefinitionParser(ServiceBean.class, true));
        registerBeanDefinitionParser("reference", new DubboBeanDefinitionParser(ReferenceBean.class, false));
        ...
    }
}
```

#### 3、Spring 加载时机回调

1. 在 `ServiceConfig.export()` 或 `ReferenceConfig.get()` 初始化时，将 Bean 对象转换 URL 格式，所有 Bean 属性转成 Dubbo#URL 的参数。
2. 然后将 Dubbo#URL 传给 Protocol 扩展点，基于扩展点的 SPI 机制，根据 Dubbo#URL 的协议头，进行不同协议的服务暴露或引用。

```java
// ServiceBean实现了ApplicationListener，在ContextRefreshedEvent发布，即Spring上下文准备完毕时，会回调onApplicationEvent方法，拉起ServiceBean
public class ServiceBean<T> extends ServiceConfig<T> implements InitializingBean, DisposableBean, ApplicationContextAware, ApplicationListener<ContextRefreshedEvent>, BeanNameAware, ApplicationEventPublisherAware {
    ...
    @Override
    public void onApplicationEvent(ContextRefreshedEvent event) {
        if (!isExported() && !isUnexported()) {
            if (logger.isInfoEnabled()) {
                logger.info("The service ready on spring started. service: " + getInterface());
            }
            // 实现Provider的服务发布
            export();
        }
    }
    ...
}

// ReferenceBeans实现了FactoryBean，在注入Dubbo#ref接口接口时，Spring会回调getObject方法，拉起ReferenceBean
public class ReferenceBean<T> extends ReferenceConfig<T> implements FactoryBean, ApplicationContextAware, InitializingBean, DisposableBean {
	...
    @Override
    public Object getObject() {
        // 实现Consumer的服务引用
        return get();
    }
    ...
}
```

### 2.3. Dubbo 服务发布原理？

![1636896497908](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1636896497908.png)

1. `ServiceConfig` 类拿到对外提供实现类 ref。

2. 然后通过 `ProxyFactory#getInvoker（）` 为 ref 生成一个 `AbstractProxyInvoker` 实例，到这一步就完成具体服务到 `Invoker` 的转化。

3. 接下来就是 `Invoker` 转换到 `Exporter` 的过程，而**服务暴露的关键**就这个过程，即图中红色的部分，其转换分为两种类型：

   - **暴露本地服务**：指服务暴露和引用都在同一个 JVM 里，自己调用自己接口，没必要进行远程通信。
     1. `InjvmProtocol#export（）` 把 invoker 转换为 `InjvmExporter`，并存进 exporters 缓存中。
   - **暴露远程服务**：指服务暴露给远程客户端IP和端口号，以实现远程通信。
     1. `DubboProtocol#export（）`  把 invoker 转换为 `DubboExporter`，然后打开 Netty 服务器暴露服务，并存进 exporters 缓存中。
     2. `RegistryProtocol#register（）`，使用 Curator 客户端建立 ZK 连接，注册 provider 持久化结点、service 非持久化结点、configurators 非持久化结点，并设置监听器，当非持久化结点发生变更，则会回调监听器的 `notify（）`，修改 invoker 信息。

   ![1637067703250](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1637067703250.png)

#### 1、本地暴露服务

1. 构建实现类的动态代理对象，基于扩展点自适应机制，调用 `InjvmProtocol#export()` 方法获取 `InjvmExporter`，并存入 `exporters` 缓存中。

```java
// 1. ServiceBean实现了ApplicationListener，在ContextRefreshedEvent发布，即Spring上下文准备完毕时，会回调onApplicationEvent方法，拉起ServiceBean
public class ServiceBean<T> extends ServiceConfig<T> implements InitializingBean, DisposableBean, ApplicationContextAware, ApplicationListener<ContextRefreshedEvent>, BeanNameAware, ApplicationEventPublisherAware {
    ...
    @Override
    public void onApplicationEvent(ContextRefreshedEvent event) {
        if (!isExported() && !isUnexported()) {
            if (logger.isInfoEnabled()) {
                logger.info("The service ready on spring started. service: " + getInterface());
            }
            // 实现Provider的服务发布
            export();
        }
    }
    
    @Override
    public void export() {
        // 2. 实现Provider的服务发布
        super.export();
        publishExportEvent();
    }
}

// ServiceBean父类
public class ServiceConfig<T> extends AbstractServiceConfig {
    public synchronized void export() {
        ...
		doExport();
    }
    
    protected synchronized void doExport() {
        ...
        doExportUrls();
    }
    
    private void doExportUrls() {
        // 3. 获取dubbo.properties registry url配置
        List<URL> registryURLs = loadRegistries(true);
        
        // 4. 循环协议，说明一个服务支持多种协议
        for (ProtocolConfig protocolConfig : protocols) {
            String pathKey = URL.buildKey(getContextPath(protocolConfig).map(p -> p + "/" + path).orElse(path), group, version);
            ProviderModel providerModel = new ProviderModel(pathKey, ref, interfaceClass);
            ApplicationModel.initProviderModel(pathKey, providerModel);
            
            // 5. 根据协议配置，发布注册url
            doExportUrlsFor1Protocol(protocolConfig, registryURLs);
        }
    }
    
    private void doExportUrlsFor1Protocol(ProtocolConfig protocolConfig, List<URL> registryURLs) {
        ...
        // 6. 如果不是只暴露远程服务（一般不配），则暴露本地服务
        if (!SCOPE_REMOTE.equalsIgnoreCase(scope)) {
            exportLocal(url);
        }
        ... 
        // 7. 如果不是只暴露远程服务（一般不配），则暴露本地服务
        if (!SCOPE_LOCAL.equalsIgnoreCase(scope)) {
            ...// 放在暴露远程服务中展开
        }
        ...
    }
    
    private void exportLocal(URL url) {
        ...
        // 9. 通过InjvmProtocol#export获取InjvmExporter
        Exporter<?> exporter = protocol.export(
            // 8. 这里是Server端，获取interfaceClass的javasist动态代理Wrapper包装类Invoker
            PROXY_FACTORY.getInvoker(ref, (Class) interfaceClass, local));
        
        // 10. 最终目的：缓存每个服务的exporter到exporters中
        exporters.add(exporter);
        ...
    }
}

public class JavassistProxyFactory extends AbstractProxyFactory {
    @Override
    public <T> Invoker<T> getInvoker(T proxy, Class<T> type, URL url) {
        final Wrapper wrapper = Wrapper.getWrapper(proxy.getClass().getName().indexOf('$') < 0 ? proxy.getClass() : type);
        // 8.获取interfaceClass的javasist动态代理Wrapper包装类
        return new AbstractProxyInvoker<T>(proxy, type, url) {
            @Override
            protected Object doInvoke(T proxy, String methodName,
                                      Class<?>[] parameterTypes,
                                      Object[] arguments) throws Throwable {
                // 动态代理调用实现类方法
                return wrapper.invokeMethod(proxy, methodName, parameterTypes, arguments);
            }
        };
    }
}

// 9. 通过InjvmProtocol#export获取InjvmExporter
public class InjvmProtocol extends AbstractProtocol implements Protocol {
    @Override
    public <T> Exporter<T> export(Invoker<T> invoker) throws RpcException {
        return new InjvmExporter<T>(invoker, invoker.getUrl().getServiceKey(), exporterMap);
    }
}
```

#### 2、打开 Netty 服务器，远程暴露服务

![1636889534944](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1636889534944.png)

1. 基于扩展点自适应机制，通过提供者 URL 的 `dubbo://` 协议头识别，就会调用 `DubboProtocol` 的 `export()` 方法，打开 Netty 服务器，利用 Netty NIO 特性，把同步操作转换为 I/O 监听的异步操作，以打开本地 Dubbo 服务，最后把 `DubboExporter` 缓存到 `exportes` 中。

```java
// ServiceBean实现了ApplicationListener，在ContextRefreshedEvent发布，即Spring上下文准备完毕时，会回调onApplicationEvent方法，拉起ServiceBean
public class ServiceBean<T> extends ServiceConfig<T> implements InitializingBean, DisposableBean, ApplicationContextAware, ApplicationListener<ContextRefreshedEvent>, BeanNameAware, ApplicationEventPublisherAware {
   	private void doExportUrlsFor1Protocol(ProtocolConfig protocolConfig, List<URL> registryURLs) {
        ...
        // 1. 如果不是只暴露远程服务（一般不配），则暴露本地服务
        if (!SCOPE_REMOTE.equalsIgnoreCase(scope)) {
            exportLocal(url);
        }
        ... 
        // 2. 如果不是只暴露远程服务（一般不配），则暴露本地服务
        if (!SCOPE_LOCAL.equalsIgnoreCase(scope)) {
            // 3. 这里是Server端，获取interfaceClass的javasist动态代理Wrapper包装类Invoker
            Invoker<?> invoker = PROXY_FACTORY.getInvoker(ref, (Class) interfaceClass, registryURL.addParameterAndEncoded(EXPORT_KEY, url.toFullString()));
            DelegateProviderMetaDataInvoker wrapperInvoker = new DelegateProviderMetaDataInvoker(invoker, this);
            
            // 4. 通过RegistryProtocol#export暴露远程服务，获取对应的exporter
            Exporter<?> exporter = protocol.export(wrapperInvoker);
            
            // 18. 最终目的：缓存每个服务的exporter到exporters中
            exporters.add(exporter);
        }
        ...
    }
}    

public class RegistryProtocol implements Protocol {
    @Override
    public <T> Exporter<T> export(final Invoker<T> originInvoker) throws RpcException 	  {
       	// 5. 打开Netty服务器，利用Netty NIO特性，把同步操作转换为I/O监听的异步操作，以打开本地Dubbo服务，最后把DubboExporter缓存到exportes中
        final ExporterChangeableWrapper<T> exporter = doLocalExport(originInvoker, providerUrl);
        // ... 连接、订阅、监听ZK
    }
    
    private <T> ExporterChangeableWrapper<T> doLocalExport(final Invoker<T> originInvoker, URL providerUrl) {
        String key = getCacheKey(originInvoker);
        return (ExporterChangeableWrapper<T>) bounds.computeIfAbsent(key, s -> {
            Invoker<?> invokerDelegate = new InvokerDelegate<>(originInvoker, providerUrl);
            // 6. 通过ProtocolFilterWrapper过滤扩展链装饰->ProtocolListenerWrapper监听扩展链装饰->DubboProtocol#export，暴露本地Dubbo服务，打开Netty服务器
            return new ExporterChangeableWrapper<>((Exporter<T>) protocol.export(invokerDelegate), originInvoker);
        });
    }
}

public class ProtocolFilterWrapper implements Protocol {
    @Override
    public <T> Exporter<T> export(Invoker<T> invoker) throws RpcException {
        if (REGISTRY_PROTOCOL.equals(invoker.getUrl().getProtocol())) {
            return protocol.export(invoker);
        }
        // 7. ProtocolFilterWrapper过滤扩展链装饰（8个过滤器）
        return protocol.export(buildInvokerChain(invoker, SERVICE_FILTER_KEY, CommonConstants.PROVIDER));
    }
}

public class ProtocolListenerWrapper implements Protocol {
    @Override
    public <T> Exporter<T> export(Invoker<T> invoker) throws RpcException {
        if (REGISTRY_PROTOCOL.equals(invoker.getUrl().getProtocol())) {
            return protocol.export(invoker);
        }
        // 8. ProtocolListenerWrapper监听扩展链装饰
        return new ListenerExporterWrapper<T>(protocol.export(invoker),            Collections.unmodifiableList(ExtensionLoader.getExtensionLoader(ExporterListener.class) .getActivateExtension(invoker.getUrl(), EXPORTER_LISTENER_KEY)));
    }
}

public class DubboProtocol extends AbstractProtocol {
    @Override
    public <T> Exporter<T> export(Invoker<T> invoker) throws RpcException {
		...
        // 9. 组装（key=xxx.DemoService：20880，value=DubboExporter）到exporterMap
        DubboExporter<T> exporter = new DubboExporter<T>(invoker, key, exporterMap);
        exporterMap.put(key, exporter);
        ...
        // 10. 打开Netty服务器，暴露本地Dubbo服务
        openServer(url);
        ...
    }
    
    private void openServer(URL url) {
        ...
        serverMap.put(key, createServer(url));
        ...
    }
    
    private ExchangeServer createServer(URL url) {
        ...
        // 11. 调用Exchangers绑定端口，打开Netty服务器
        server = Exchangers.bind(url, requestHandler);
    }
}

public class Exchangers {
   public static ExchangeServer bind(URL url, ExchangeHandler handler) throws RemotingException {
        ...
        // 12. SPI获取HeaderExchanger来绑定端口
        return getExchanger(url).bind(url, handler);
    }
}

public class HeaderExchanger implements Exchanger {
    @Override
    public ExchangeServer bind(URL url, ExchangeHandler handler) throws RemotingException {
        return new HeaderExchangeServer(
             // 13. 调用Transporters绑定端口，打开Netty服务器
            Transporters.bind(url, new DecodeHandler(new HeaderExchangeHandler(handler))));
    }
}

public class Transporters {
    public static Server bind(URL url, ChannelHandler... handlers) throws RemotingException {
        ...
        // 14. SPI获取remoting.transport.netty4.NettyTransporter来绑定端口
        return getTransporter().bind(url, handler);
    }
}

public class NettyTransporter implements Transporter {
    @Override
    public Server bind(URL url, ChannelHandler listener) throws RemotingException {
        // 15. 打开Netty服务器
        return new NettyServer(url, listener);
    }
}

// NettyServer抽象父类
public abstract class AbstractServer extends AbstractEndpoint implements Server {
    public AbstractServer(URL url, ChannelHandler handler) throws RemotingException     {
        ...
        // 16. 父类模板模式多态调用NettyServer#doOpen
        doOpen();
    }
    
    protected abstract void doOpen() throws Throwable;
}

public class NettyServer extends AbstractServer implements Server {
    @Override
    protected void doOpen() throws Throwable {
        // 17. 利用Netty语法，打开Netty服务器，同时指定InternalDecoder解码器、InternalEncoder编码器、NettyServerHandler通道处理器
        ...// 省略Netty语法
    }
}
```

#### 3、建立连接并监听 ZK

1. `ServiceConfig` 解析出的 URL 的格式为: `registry://registry-host/org.apache.dubbo.registry.RegistryService?export=URL.encode("dubbo://service-host/com.foo.FooService?version=1.0.0")`。
2. 基于扩展点自适应机制，通过 URL 的 `registry://` 协议头识别，就会调用 `RegistryProtocol` 的 `export()` 方法，将 `export` 参数中的提供者 URL，注册到注册中心。

```java
public class RegistryProtocol implements Protocol {
    @Override
    public <T> Exporter<T> export(final Invoker<T> originInvoker) throws RpcException 	  {
       	// 0. 利用Netty NIO特性，打开Netty服务器，把同步操作转换为I/O监听的异步操作，以打开本地Dubbo服务，最后把DubboExporter缓存到exportes中
        final ExporterChangeableWrapper<T> exporter = doLocalExport(originInvoker, providerUrl);
        // 1. 获取注册中心 => 这里用的是ZK作为注册中心
        final Registry registry = getRegistry(originInvoker);
        ...
    }
    
    private Registry getRegistry(final Invoker<?> originInvoker) {
        URL registryUrl = getRegistryUrl(originInvoker);
        return registryFactory.getRegistry(registryUrl);
    }
}

public abstract class AbstractRegistryFactory implements RegistryFactory {
    @Override
    public Registry getRegistry(URL url) {
        ...
        // 2. 如果缓存中没有注册中心，则构建连接注册中心的客户端
        registry = createRegistry(url);
        ...
    }
    
    // 3. 父类模板模式多态调用RegistryFactory#createRegistry
    protected abstract Registry createRegistry(URL url);
}

public class ZookeeperRegistryFactory extends AbstractRegistryFactory {
    @Override
    public Registry createRegistry(URL url) {
        // 4. ZookeeperRegistryFactory子类构建ZK客户端
        return new ZookeeperRegistry(url, zookeeperTransporter);
    }
}

public class ZookeeperRegistry extends FailbackRegistry {
    public ZookeeperRegistry(URL url, ZookeeperTransporter zookeeperTransporter) {
        // 5. 调用父类加载、保存注册地址缓存文件
        super(url);
        
        // 7. 调用CuratorZookeeperTransporter#createZookeeperClient创建ZK客户端（Curator语法省略），并设置DISCONNECTED、CONNECTED、RECONNECTED事件的监听器
        zkClient = zookeeperTransporter.connect(url);
        
        // 8. 设置RECONNECTED事件的监听器 => 失败重连
        zkClient.addStateListener(state -> {
            if (state == StateListener.RECONNECTED) {
                try {
                    // 失败重连
                    recover();
                } catch (Exception e) {
                    logger.error(e.getMessage(), e);
                }
            }
        });
    }
}

public abstract class AbstractRegistry implements Registry {
    public AbstractRegistry(URL url) {
        ...
        // 6. 加载C：\Users\dubbo\.dubbo\dubbo-registry-192.168.48.117.cache缓存文件内容
        loadProperties();
        ...
    }
}
```

#### 4、注册 Provider、创建 ZK 结点

```java
public class RegistryProtocol implements Protocol {
    @Override
    public <T> Exporter<T> export(final Invoker<T> originInvoker) throws RpcException 	  {
       	// 0.1 利用Netty NIO特性，打开Netty服务器，把同步操作转换为I/O监听的异步操作，以打开本地Dubbo服务，最后把DubboExporter缓存到exportes中
        final ExporterChangeableWrapper<T> exporter = doLocalExport(originInvoker, providerUrl);
        // 0.2 获取注册中心 => 这里用的是ZK作为注册中心
        final Registry registry = getRegistry(originInvoker);
        ...
        // 1. 注册Provider到ZK注册中心
        register(registryUrl, registeredProviderUrl);
    }
}

public abstract class FailbackRegistry extends AbstractRegistry {
    @Override
    public void register(URL url) {
        ...
        // 2. 注册Provider到ZK注册中心
        doRegister(url);
        ...
    }
    
    // 3. 父类模板模式多态调用ZookeeperRegistry#doRegister
    public abstract void doRegister(URL url);
}

public class ZookeeperRegistry extends FailbackRegistry {
    @Override
    public void doRegister(URL url) {
        // 4. Curator ZK客户端创建 /dubbo/xxx.DemoService/providers/... 前面的为持久化结点，后面的（...）为非持久化结点（Curator语法省略）
        zkClient.create(toUrlPath(url), url.getParameter(DYNAMIC_KEY, true));
    }
}
```

#### 5、订阅 ZK 配置信息

```java
public class RegistryProtocol implements Protocol {
    @Override
    public <T> Exporter<T> export(final Invoker<T> originInvoker) throws RpcException 	  {
       	// 0.1 利用Netty NIO特性，打开Netty服务器，把同步操作转换为I/O监听的异步操作，以打开本地Dubbo服务，最后把DubboExporter缓存到exportes中
        final ExporterChangeableWrapper<T> exporter = doLocalExport(originInvoker, providerUrl);
        // 0.2 获取注册中心 => 这里用的是ZK作为注册中心
        final Registry registry = getRegistry(originInvoker);
        ...
        // 0.3. 注册Provider到ZK注册中心
        register(registryUrl, registeredProviderUrl);
        ...
        // 1. 订阅ZK配置信息，当有变更时自动推送
        registry.subscribe(overrideSubscribeUrl, overrideSubscribeListener);
        ...
        return new DestroyableExporter<>(exporter);
    }
}

public abstract class FailbackRegistry extends AbstractRegistry {
    @Override
    public void subscribe(URL url, NotifyListener listener) {
        ...
        // 2. 向服务器端发送订阅请求
        doSubscribe(url, listener);
        ...  
    }
    
    @Override
    protected void notify(URL url, NotifyListener listener, List<URL> urls) {
        // 5. 通知、写入缓存文件
        doNotify(url, listener, urls);
    }
    
    protected void doNotify(URL url, NotifyListener listener, List<URL> urls) {
        // 6. 通知、写入缓存文件
        super.notify(url, listener, urls);
    }
}

public abstract class AbstractRegistry implements Registry {
    protected void notify(URL url, NotifyListener listener, List<URL> urls) {
        // 7. 刷新配置和Invoker列表
        listener.notify(categoryList);
        
        // 11. 线程池异步写入C：\Users\dubbo\.dubbo\dubbo-registry-192.168.48.117.cache缓存文件内容
        saveProperties(url);
    }
}

public class RegistryDirectory<T> extends AbstractDirectory<T> implements NotifyListener {
    @Override
    public synchronized void notify(List<URL> urls) {
        // 8. 刷新配置和Invoker列表
        refreshOverrideAndInvoker(providerURLs);
    }
    
    private void refreshOverrideAndInvoker(List<URL> urls) {
        // 9. 刷新configurators配置
        overrideDirectoryUrl();
        
        // 10. 刷新RegistryDirectory中的Invoker列表
        refreshInvoker(urls);
    }
    private void overrideDirectoryUrl() {
        ...
        doOverrideUrl(localDynamicConfigurators);
    }
    private void doOverrideUrl(List<Configurator> configurators) {
        if (CollectionUtils.isNotEmpty(configurators)) {
            for (Configurator configurator : configurators) {
                this.overrideDirectoryUrl = configurator.configure(overrideDirectoryUrl);
            }
        }
    }
    private void refreshInvoker(List<URL> invokerUrls) {
        Map<String, Invoker<T>> newUrlInvokerMap = toInvokers(invokerUrls);
        ...
        this.invokers = multiGroup ? toMergeInvokerList(newInvokers) : newInvokers;
        this.urlInvokerMap = newUrlInvokerMap;
        destroyUnusedInvokers(oldUrlInvokerMap, newUrlInvokerMap);
        ...
    }
}

public class ZookeeperRegistry extends FailbackRegistry {
    protected void doSubscribe(final URL url, final NotifyListener listener) {
        // 3. 创建configurators配置信息（非持久化结点）
        zkClient.create(path, false);
        
        // 4. 通知、写入缓存文件
        notify(url, listener, urls);
    }
}
```

### 2.4. Dubbo 服务引用原理？

![1636896718809](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1636896718809.png)

1. 首先 `ReferenceConfig#init（）` ，调用 `Protocol#refer（）`生成 `Invoker` 实例（图中的红色部分），这是**服务消费的关键**。
   - **本地服务引用**：如果为本地暴露服务，则 `InjvmProtocol` 生成 `serviceType` 本地执行的 `InjvmInvoker` 。
   - **直连服务引用**：如果为直连服务，则 `DubboProtocol` 创建 Netty 客户端，连接 url 服务，构建 `DubboInvoker` 。
   - **远程服务引用**：如果为非直连的远程服务引用，则 `RegistryProtocol` 注册、订阅 ZK，拉取相关 url 和 配置信息，当有发生变更时，则触发监听器的回调函数，实际调用 `DubboProtocol#refer()` 生成 `DubboInvoker` ，并加入集群，最后默认伪装返回一个 `FailoverClusterInvoker`。
2. 接下来是利用动态代理，把 `Invoker` 转换为客户端需要的接口实现类。

![1637149757299](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1637149757299.png)

#### 1、本地服务引用

```java
// ReferenceBeans实现了FactoryBean，在注入Dubbo#ref接口接口时，Spring会回调getObject方法，拉起ReferenceBean
public class ReferenceBean<T> extends ReferenceConfig<T> implements FactoryBean, ApplicationContextAware, InitializingBean, DisposableBean {
	...
    @Override
    public Object getObject() {
        // 0. 实现Consumer的服务引用
        return get();
    }
}

public class ReferenceConfig<T> extends AbstractReferenceConfig {
    public synchronized T get() {
        checkAndUpdateSubConfigs();

        if (destroyed) {
            throw new IllegalStateException("The invoker of ReferenceConfig(" + url + ") has already destroyed!");
        }
        if (ref == null) {
            // 1. 实现Consumer的服务引用
            init();
        }
        return ref;
    }
    
    private void init() {
        ...
        // 2. 创建服务引用的动态代理
        ref = createProxy(map);
        ...
    }
    
    private T createProxy(Map<String, String> map) {
        // 3. 如果为本地暴露服务，则生成本地执行的Invoker
    	if (shouldJvmRefer(map)) {
            URL url = new URL(LOCAL_PROTOCOL, LOCALHOST_VALUE, 0, interfaceClass.getName()).addParameters(map);
            invoker = REF_PROTOCOL.refer(interfaceClass, url);
            ...
        } else {
            ...
    	}
        
        // 5. 生成Invoker的动态代理
        return (T) PROXY_FACTORY.getProxy(invoker);
    }
}

public class InjvmProtocol extends AbstractProtocol implements Protocol {
    // 4. 生成本地执行的Invoker
    @Override
    public <T> Invoker<T> protocolBindingRefer(Class<T> serviceType, URL url) throws RpcException {
        return new InjvmInvoker<T>(serviceType, url, url.getServiceKey(), exporterMap);
    }
}
```

#### 2、直连服务引用

1. 在没有注册中心，直连提供者的情况下，`ReferenceConfig` 解析出的 URL 的格式为：`dubbo://service-host/com.foo.FooService?version=1.0.0` 。
2. 基于扩展点自适应机制，通过 URL 的 `dubbo://` 协议头识别，直接调用 `DubboProtocol` 的 `refer()` 方法，建立 Netty 客户端连接，构建 `DubboInvoker`，并加入 invokers 缓存。

```java
// ReferenceBeans实现了FactoryBean，在注入Dubbo#ref接口接口时，Spring会回调getObject方法，拉起ReferenceBean
public class ReferenceBean<T> extends ReferenceConfig<T> implements FactoryBean, ApplicationContextAware, InitializingBean, DisposableBean {

    private T createProxy(Map<String, String> map) {
        // 0. 如果为本地暴露服务，则生成本地执行的Invoker
    	if (shouldJvmRefer(map)) {
            URL url = new URL(LOCAL_PROTOCOL, LOCALHOST_VALUE, 0, interfaceClass.getName()).addParameters(map);
            invoker = REF_PROTOCOL.refer(interfaceClass, url);
            ...
        } else {
            for (URL url : urls) {
                // 1. 如果不是本地暴露服务，则生成远程服务引用的Invoker
                invokers.add(REF_PROTOCOL.refer(interfaceClass, url));
            }
            ...
    	}
        
        // 21. 生成Invoker的动态代理
        return (T) PROXY_FACTORY.getProxy(invoker);
    }
}

public abstract class AbstractProtocol implements Protocol {
    @Override
    public <T> Invoker<T> refer(Class<T> type, URL url) throws RpcException {
        // 2. 调用protocolBindingRefer
        return new AsyncToSyncInvoker<>(protocolBindingRefer(type, url));
    }
    
    // 3. 模板方法，多态调用DubboProtocol#protocolBindingRefer
    protected abstract <T> Invoker<T> protocolBindingRefer(Class<T> type, URL url) throws RpcException;
}

public class DubboProtocol extends AbstractProtocol {
    @Override
    public <T> Invoker<T> protocolBindingRefer(Class<T> serviceType, URL url) throws RpcException {
		...
        //  19. 生成serviceType的invoker
        DubboInvoker<T> invoker = new DubboInvoker<T>(serviceType, url, 
                                                      // 4. 根据url获取客户端连接
                                                      getClients(url), invokers);
        
        // 20. 加入invokers缓存
        invokers.add(invoker);
        return invoker;
    }
    
    private ExchangeClient[] getClients(URL url) {
        ...
        // 5. 初始化客户端连接
        clients[i] = initClient(url);
        ...
    }
    
    private ExchangeClient initClient(URL url) {
        // 6. 进入Exchanger层，连接url服务
        ExchangeClient client = Exchangers.connect(url, requestHandler);
    }
}

public class Exchangers {
    public static ExchangeClient connect(URL url, ExchangeHandler handler) throws RemotingException {
        ...
        // 7. 进入Exchanger层，连接url服务
        return getExchanger(url).connect(url, handler);
    }
}

public class HeaderExchanger implements Exchanger {
    @Override
    public ExchangeClient connect(URL url, ExchangeHandler handler) throws RemotingException {
        return new HeaderExchangeClient(
            // 8. 进入Transporter层，连接url服务
            Transporters.connect(url, new DecodeHandler(new HeaderExchangeHandler(handler))), true);
    }
}

public class Transporters {
    public static Client connect(URL url, ChannelHandler... handlers) throws RemotingException {
        // 9. 进入Transporter层，连接url服务
        return getTransporter().connect(url, handler);
    }
}

public class NettyTransporter implements Transporter {
    @Override
    public Client connect(URL url, ChannelHandler listener) throws RemotingException {
        // 10. 默认打开Netty客户端，连接url服务
        return new NettyClient(url, listener);
    }
}

public class NettyClient extends AbstractClient {
    public NettyClient(final URL url, final ChannelHandler handler) throws RemotingException {
        // 11. 默认打开Netty客户端，连接url服务
        super(url, wrapChannelHandler(url, handler));
    }
    
    @Override
    protected void doOpen() throws Throwable {
        // 14. 利用Netty语法，使用客户端连接url连接服务，同时指定InternalDecoder解码器、InternalEncoder编码器、NettyClientHandler通道处理器
        ...// 省略Netty语法
    }
    
    @Override
    protected void doConnect() throws Throwable {
        ...
        // 18. 利用Netty语法，发起url连接
        ChannelFuture future = bootstrap.connect(getConnectAddress());
        ...// 省略Netty语法
    }
}

public abstract class AbstractClient extends AbstractEndpoint implements Client {
    public AbstractClient(URL url, ChannelHandler handler) throws RemotingException {
        ...
        // 12. 打开Netty客户端
        doOpen();
        ...
        // 15. 发起url连接
        connect();
        ...
    }
    
    // 13. 模板方法，多态调用子类NettyClient#doOpen
    protected abstract void doOpen() throws Throwable;
    
    protected void connect() throws RemotingException {
        ...
        // 16. 发起url连接
        doConnect();
        ...
    }
    
    // 17. 模板方法，多态调用子类NettyClient#doConnect
    protected abstract void doConnect() throws Throwable;
}
```

#### 3、远程服务引用

![1637068993216](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1637068993216.png)

1. 在有注册中心，通过注册中心发现提供者地址的情况下，`ReferenceConfig` 解析出的 URL 的格式为： `registry://registry-host/org.apache.dubbo.registry.RegistryService?refer=URL.encode("consumer://consumer-host/com.foo.FooService?version=1.0.0") `。
2. 基于扩展点自适应机制，通过 URL 的 `registry://` 协议头识别，就会调用 `RegistryProtocol` 的 `refer()` 方法，基于 `refer` 参数中的条件，查询提供者 URL，如： `dubbo://service-host/com.foo.FooService?version=1.0.0` 。
3. 当订阅内容发生更新时，触发监听器回调，里面调用 `protocol.refer()`，基于扩展点自适应机制，通过提供者 URL 的 `dubbo://` 协议头识别，就会调用 `DubboProtocol#refer()` 方法，得到提供者引用。
4. 然后 `RegistryProtocol` 将多个提供者引用，通过 `Cluster` 扩展点，伪装成单个提供者引用 `FailoverClusterInvoker` 返回。

```java
// ReferenceBeans实现了FactoryBean，在注入Dubbo#ref接口接口时，Spring会回调getObject方法，拉起ReferenceBean
public class ReferenceBean<T> extends ReferenceConfig<T> implements FactoryBean, ApplicationContextAware, InitializingBean, DisposableBean {

    private T createProxy(Map<String, String> map) {
        // 0. 如果为本地暴露服务，则生成本地执行的Invoker
    	if (shouldJvmRefer(map)) {
            URL url = new URL(LOCAL_PROTOCOL, LOCALHOST_VALUE, 0, interfaceClass.getName()).addParameters(map);
            invoker = REF_PROTOCOL.refer(interfaceClass, url);
            ...
        } else {
            for (URL url : urls) {
                // 1. 如果不是本地暴露服务，则生成远程服务引用的Invoker
                invokers.add(REF_PROTOCOL.refer(interfaceClass, url));
            }
            ...
    	}
        
        // 28. 生成Invoker的动态代理
        return (T) PROXY_FACTORY.getProxy(invoker);
    }
}

public class ProtocolFilterWrapper implements Protocol {
    @Override
    public <T> Invoker<T> refer(Class<T> type, URL url) throws RpcException {
        if (REGISTRY_PROTOCOL.equals(url.getProtocol())) {
            return protocol.refer(type, url);
        }
        // 1.1 ProtocolFilterWrapper过滤扩展链装饰（8个过滤器）
        return buildInvokerChain(protocol.refer(type, url), REFERENCE_FILTER_KEY, CommonConstants.CONSUMER);
    }
}

public class RegistryProtocol implements Protocol {
    public <T> Invoker<T> refer(Class<T> type, URL url) throws RpcException {
        ...
        // 2. 建立zk的连接，和服务端发布一样（省略代码）
        Registry registry = registryFactory.getRegistry(url);
        ...
        // 3. 引用远程服务
        return doRefer(cluster, registry, type, url);
    }
    
    private <T> Invoker<T> doRefer(Cluster cluster, Registry registry, Class<T> type, URL url) {
        // 4. 建立url Invoker目录
        RegistryDirectory<T> directory = new RegistryDirectory<T>(type, url);
        ...
        // 5. 和服务端发布一样，注册ZK，创建zk的节点
        registry.register(directory.getRegisteredConsumerUrl());
        ...
        // 9. 和服务端发布一样，订阅ZK节点：/dubbo/com.alibaba.dubbo.demo.DemoService/providers，/dubbo/com.alibaba.dubbo.demo.DemoService/configurators，/dubbo/com.alibaba.dubbo.demo.DemoService/routers，url发生变化，则刷新Invoker
        directory.subscribe(subscribeUrl.addParameter(CATEGORY_KEY,
                                                      PROVIDERS_CATEGORY + "," + CONFIGURATORS_CATEGORY + "," + ROUTERS_CATEGORY));
        
        // 25. 根据directory，加入集群路由
        Invoker invoker = cluster.join(directory);
        ...
        return invoker;
    }
    
    public void subscribe(URL url) {
        ...
        // 10. 订阅ZK节点
        registry.subscribe(url, this);
    }
}

public abstract class FailbackRegistry extends AbstractRegistry {
    @Override
    public void register(URL url) {
        // 6. 注册ZK，创建zk的节点
        doRegister(url);
    }
    
    // 7. 模板方法，多态调用ZookeeperRegistry#doRegister
    public abstract void doRegister(URL url);
    
    @Override
    public void subscribe(URL url, NotifyListener listener) {
        ...
        // 11. 订阅ZK节点
        doSubscribe(url, listener);
    }
    
    // 12. 模板方法，多态调用ZookeeperRegistry#doSubscribe
    public abstract void doSubscribe(URL url, NotifyListener listener);
    
    @Override
    protected void notify(URL url, NotifyListener listener, List<URL> urls) {
        ...
        // 14. 通知更新url配置
        doNotify(url, listener, urls);
        ...
    }
    
    protected void doNotify(URL url, NotifyListener listener, List<URL> urls) {
        ...
        // 15. 通知更新url配置
        super.notify(url, listener, urls);
        ...
    }
}

public class ZookeeperRegistry extends FailbackRegistry {
    @Override
    public void doRegister(URL url) {
        // 8. 创建ZK持久化：dubbo/com.alibaba.dubbo.demo.DemoService/consumers
        zkClient.create(toUrlPath(url), url.getParameter(DYNAMIC_KEY, true));
    }
    
    @Override
    public void doSubscribe(final URL url, final NotifyListener listener) {
        ...
        // 13. 通知更新url配置 
        notify(url, listener, urls);
    }
}

public abstract class AbstractRegistry implements Registry {
    protected void notify(URL url, NotifyListener listener, List<URL> urls) {
        ...
        // 19. 通知更新url配置，生成新的Invoker
        listener.notify(categoryList);
        // 29. 通过线程池，异步把服务端的注册url信息更新到C:\Users\bobo\.dubbo\dubbo-registry-192.168.48.117.cache
        saveProperties(url);
    }
}

public class RegistryDirectory<T> extends AbstractDirectory<T> implements NotifyListener {
    @Override
    public synchronized void notify(List<URL> urls) {
        ...
        // 16. 通知更新url配置，刷新Invoker列表
        refreshOverrideAndInvoker(providerURLs);
    }
    
    private void refreshOverrideAndInvoker(List<URL> urls) {
        // 17. 更新configurators配置（和服务端一样，省略代码）
        overrideDirectoryUrl();
        // 18. 刷新Invoker列表
        refreshInvoker(urls);
    }
    
    private void refreshInvoker(List<URL> invokerUrls) {
        // 19. 根据url生成新的Invoker
        Map<String, Invoker<T>> newUrlInvokerMap = toInvokers(invokerUrls);
        List<Invoker<T>> newInvokers = Collections.unmodifiableList(new ArrayList<>(newUrlInvokerMap.values()));
        routerChain.setInvokers(newInvokers);
        this.invokers = multiGroup ? toMergeInvokerList(newInvokers) : newInvokers;
        this.urlInvokerMap = newUrlInvokerMap;
    }
    
    private Map<String, Invoker<T>> toInvokers(List<URL> urls) {
        // 20. 调用父类AbstractProtocol#refer
        invoker = new InvokerDelegate<>(protocol.refer(serviceType, url), url, providerUrl);
        ...
    }
}

public abstract class AbstractProtocol implements Protocol {
    @Override
    public <T> Invoker<T> refer(Class<T> type, URL url) throws RpcException {
        // 21. 调用protocolBindingRefer
        return new AsyncToSyncInvoker<>(protocolBindingRefer(type, url));
    }
    
    // 22. 模板方法，多态调用DubboProtocol#protocolBindingRefer
    protected abstract <T> Invoker<T> protocolBindingRefer(Class<T> type, URL url) throws RpcException;
}

public class DubboProtocol extends AbstractProtocol {
    @Override
    public <T> Invoker<T> protocolBindingRefer(Class<T> serviceType, URL url) throws RpcException {
        // 23. 和服务端一样，根据接口class和url，生成DubboInvoker
        DubboInvoker<T> invoker = new DubboInvoker<T>(serviceType, url, getClients(url), invokers);
        // 24. 加入invokers缓存
        invokers.add(invoker);
        return invoker;
    }
}

public class MockClusterWrapper implements Cluster {
    @Override
    public <T> Invoker<T> join(Directory<T> directory) throws RpcException {
        return new MockClusterInvoker<T>(directory,
                // 26. MockCluster包装类
                this.cluster.join(directory));
    }
}

public class FailoverCluster implements Cluster {
    @Override
    public <T> Invoker<T> join(Directory<T> directory) throws RpcException {
        // 27. 默认构建失败重试集群Invoker（省略构造函数）
        return new FailoverClusterInvoker<T>(directory);
    }
}
```

#### 4、生成接口的动态代理类

```java
// ReferenceBeans实现了FactoryBean，在注入Dubbo#ref接口接口时，Spring会回调getObject方法，拉起ReferenceBean
public class ReferenceBean<T> extends ReferenceConfig<T> implements FactoryBean, ApplicationContextAware, InitializingBean, DisposableBean {

    private T createProxy(Map<String, String> map) {
        // 0. 如果为本地暴露服务，则生成本地执行的Invoker
    	if (shouldJvmRefer(map)) {
            URL url = new URL(LOCAL_PROTOCOL, LOCALHOST_VALUE, 0, interfaceClass.getName()).addParameters(map);
            invoker = REF_PROTOCOL.refer(interfaceClass, url);
            ...
        } else {
            for (URL url : urls) {
                // 1. 如果不是本地暴露服务，则生成远程服务引用的Invoker
                invokers.add(REF_PROTOCOL.refer(interfaceClass, url));
            }
            ...
    	}
        
        // 2. 生成Invoker的动态代理
        return (T) PROXY_FACTORY.getProxy(invoker);
    }
}

public class StubProxyFactoryWrapper implements ProxyFactory {
    @Override
    public <T> T getProxy(Invoker<T> invoker) throws RpcException {
        // 3. 调用父类AbstractProxyFactory#getProxy
        T proxy = proxyFactory.getProxy(invoker);
        ...
        return proxy;
    }
}

public abstract class AbstractProxyFactory implements ProxyFactory {
    @Override
    public <T> T getProxy(Invoker<T> invoker) throws RpcException {
        // 4. 调用非泛化方法生成代理对象
        return getProxy(invoker, false);
    }
    
    @Override
    public <T> T getProxy(Invoker<T> invoker, boolean generic) throws RpcException {
        ...
        // 5. 调用实现类#getProxy生成代理对象
        return getProxy(invoker, interfaces);
    }
    
    // 6. 模板方法，多态调用JavassistProxyFactory#getProxy生成代理对象
    public abstract <T> T getProxy(Invoker<T> invoker, Class<?>[] types);
}

public class JavassistProxyFactory extends AbstractProxyFactory {
    @Override
    public <T> T getProxy(Invoker<T> invoker, Class<?>[] interfaces) {
        // 7. 生成代理对象，织入InvokerInvocationHandler
        return (T) Proxy.getProxy(interfaces).newInstance(new InvokerInvocationHandler(invoker));
    }
}
```

### 2.5. Dubbo 服务调用原理？

![1636896757166](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1636896757166.png)

1. 消费方使用的接口动态代理实现类，就是图中服务消费端的 proxy，用户代码通过这个 proxy 调用其对应的 `Invoker`，而该 `Invoker` 实现了真正的远程服务调用。

2. 而提供方的实现类，则会被封装成为一个 `AbstractProxyInvoker` ，然后生成一个 `Exporter` ，当服务提供方收到一个请求后，则会找到对应的 `Exporter` 实例，并调用它所对应的 `AbstractProxyInvoker` 实例，从而真正调用了服务提供者的代码。

3. 整个**消费方调用提供方**的调用链架构如下：

   ![1637150417647](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1637150417647.png)

   1. **服务引用**：通过 Javassist 反向代理，代理调用 InvokerInvocationHandler#invoke 方法。
   2. 服务本地调用、**降级**、缓存。
   3. **集群容错与负载均衡**：非 mock Invoker 筛选，Invoker 目录查找，根据容错策略、负载均衡策略，挑选唯一的 Invoker。
   4. 服务**过滤链**、监听器、包装类 SPI 扩展。
   5. **服务协议**：根据协议，使用不同的 Invoker 调用不同的网络传输底层。
   6. **网络传输**：抽象 Netty、Mina 等统一接口，把消息序列化后发送到网络，传输给 Server 端。
   7. **消息接收与异步处理**：Server 端接收到消息后，经过反序列化后，交由线程池异步处理。
   8. **服务协议**：根据协议，选择不同的 Exporter 进行调用。
   9. 服务**过滤链**、监听器、包装类 SPI 扩展。
   10. **服务调用**：最后调用真正的接口实现类，得到方法执行结果。

4. 而**提供方响应消费方**的顺序为：

   1. **服务响应**：在得到方法执行结果后，通过网络传输底层，序列化后发送响应报文给客户端。
   2. **结果接收**：客户端接收到响应报文后，交由线程池异步处理程序，经过反序列化后塞回到 Future 对象中，完成一次服务调用。

#### 1、消费方发起服务请求

```java
public class JavassistProxyFactory extends AbstractProxyFactory {
    @Override
    public <T> T getProxy(Invoker<T> invoker, Class<?>[] interfaces) {
        // 0. 生成代理对象，织入InvokerInvocationHandler
        return (T) Proxy.getProxy(interfaces).newInstance(new InvokerInvocationHandler(invoker));
    }
}

public class InvokerInvocationHandler implements InvocationHandler {
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        ...
        // 1. 动态代理执行
		return invoker.invoke(new RpcInvocation(method, args)).recreate();
    }
}

public class MockClusterInvoker<T> implements Invoker<T> {
    @Override
    public Result invoke(Invocation invocation) throws RpcException {
        ...
        // 2. 交由抽象父类执行
        result = this.invoker.invoke(invocation);
        ...
    }
}

public abstract class AbstractClusterInvoker<T> implements Invoker<T> {
    @Override
    public Result invoke(final Invocation invocation) throws RpcException {
        ...
        // 3. 进入目录查找，从this.methodInvokerMap里面查找一个Invoker
        List<Invoker<T>> invokers = list(invocation);
        ...
        
        // 11. 进入集群，交由集群路由执行
        return doInvoke(invocation, invokers, loadbalance);
    }
    
    protected List<Invoker<T>> list(Invocation invocation) throws RpcException {
        // 4. 从this.methodInvokerMap里面查找一个Invoker
        return directory.list(invocation);
    }
    
    // 12. 模板方法，交由FailoverClusterInvoker#doInvoke执行
    protected abstract Result doInvoke(Invocation invocation, List<Invoker<T>> invokers,LoadBalance loadbalance) throws RpcException;
    
    protected Invoker<T> select(LoadBalance loadbalance, Invocation invocation,
                                List<Invoker<T>> invokers, List<Invoker<T>> selected) throws RpcException {
        ...
		// 14. 进入负载均衡
        Invoker<T> invoker = doSelect(loadbalance, invocation, invokers, selected);
        ...
    }
    
    private Invoker<T> doSelect(LoadBalance loadbalance, Invocation invocation,
                                List<Invoker<T>> invokers, List<Invoker<T>> selected) throws RpcException {
        ...
        // 15. 交由AbstractLoadBalance决定哪个路由规则进行负载均衡
        Invoker<T> invoker = loadbalance.select(invokers, getUrl(), invocation);
        ...
    }
}

public abstract class AbstractDirectory<T> implements Directory<T> {
    @Override
    public List<Invoker<T>> list(Invocation invocation) throws RpcException {
        ...
        // 5. 从this.methodInvokerMap里面查找一个Invoker
        return doList(invocation);
    }
    
    // 6. 模板方法，多态调用子类RegistryDirectory#doList
    protected abstract List<Invoker<T>> doList(Invocation invocation) throws RpcException;
}

public class RegistryDirectory<T> extends AbstractDirectory<T> implements NotifyListener {
    @Override
    public List<Invoker<T>> doList(Invocation invocation) {
        ...
        // 7. 进入路由 
        List<Invoker<T>> invokers = routerChain.route(getConsumerUrl(), invocation);
        ...
    }
}

public class RouterChain<T> {
    public List<Invoker<T>> route(URL url, Invocation invocation) {
        List<Invoker<T>> finalInvokers = invokers;
        for (Router router : routers) {
            // 8. 进入路由 
            finalInvokers = router.route(finalInvokers, url, invocation);
        }
        return finalInvokers;
    }
}

public class MockInvokersSelector extends AbstractRouter {
    @Override
    public <T> List<Invoker<T>> route(final List<Invoker<T>> invokers,
                                      URL url, final Invocation invocation) throws RpcException {
        ...
		// 9. 获取正常路径的Invoker
        return getNormalInvokers(invokers);
    }
    
    private <T> List<Invoker<T>> getNormalInvokers(final List<Invoker<T>> invokers) {
        if (!hasMockProviders(invokers)) {
            return invokers;
        } else {
            List<Invoker<T>> sInvokers = new ArrayList<Invoker<T>>(invokers.size());
            for (Invoker<T> invoker : invokers) {
                // 10. 过滤掉非正常路径的Invoker，即mock协议路径的Invoker不需要
                if (!invoker.getUrl().getProtocol().equals(MOCK_PROTOCOL)) {
                    sInvokers.add(invoker);
                }
            }
            return sInvokers;
        }
    }
}

public class FailoverClusterInvoker<T> extends AbstractClusterInvoker<T> {
    @Override
    public Result doInvoke(Invocation invocation, final List<Invoker<T>> invokers, LoadBalance loadbalance) throws RpcException {
        ...
        // 13. 进入负载均衡
        Invoker<T> invoker = select(loadbalance, invocation, copyInvokers, invoked);
        ...
        
        // 20. 使用负载均衡得到的Invoker执行
        Result result = invoker.invoke(invocation);
        return result;
    }
}

public abstract class AbstractLoadBalance implements LoadBalance {
    @Override
    public <T> Invoker<T> select(List<Invoker<T>> invokers, URL url, Invocation invocation) {
        ...
        // 16. 交由AbstractLoadBalance决定哪个路由规则进行负载均衡
        return doSelect(invokers, url, invocation);
    }
    
    // 17. 模板方法，默认交由RandomLoadBalance进行负载均衡
	protected abstract <T> Invoker<T> doSelect(List<Invoker<T>> invokers, URL url, Invocation invocation);
}

public class RandomLoadBalance extends AbstractLoadBalance {
    @Override
    protected <T> Invoker<T> doSelect(List<Invoker<T>> invokers, URL url, Invocation invocation) {
        ...
        // 18. 根据随机权重得到Invoker索引
        int offset = ThreadLocalRandom.current().nextInt(totalWeight);
        ...
        // 19. 根据Invoker索引获取对应的Invoker
        return invokers.get(ThreadLocalRandom.current().nextInt(length));
    }
}

public class InvokerWrapper<T> implements Invoker<T> {
    @Override
    public Result invoke(Invocation invocation) throws RpcException {
        // 21. 负载均衡得到的Invoker执行前，被包装类拦截
        return invoker.invoke(invocation);
    }
}

public class ProtocolFilterWrapper implements Protocol {
    private static <T> Invoker<T> buildInvokerChain(final Invoker<T> invoker, String key, String group) {
        ...
        last = new Invoker<T>() {
            public Result invoke(Invocation invocation) throws RpcException {
                ...
                // 22. 负载均衡得到的Invoker执行前，被过滤器拦截，用于添加一系列过滤器链
                Result asyncResult = filter.invoke(last, invocation);
                return asyncResult;
            }
        }
        ...
    }
}

// ... 可扩展一大堆包装类和过滤器

public class ConsumerInvokerWrapper<T> implements Invoker {
    @Override
    public Result invoke(Invocation invocation) throws RpcException {
        // 23. 负载均衡得到的Invoker执行前，被包装类拦截
        return invoker.invoke(invocation);
    }
}

public class ListenerInvokerWrapper<T> implements Invoker<T> {
    @Override
    public Result invoke(Invocation invocation) throws RpcException {
        // 24. 负载均衡得到的Invoker执行前，被包装类拦截
        return invoker.invoke(invocation);
    }
}

public abstract class AbstractInvoker<T> implements Invoker<T> {
    @Override
    public Result invoke(Invocation inv) throws RpcException {
        ...
        // 25. 负载均衡得到的Invoker执行前，先执行父类方法
        return doInvoke(invocation);
    }
    
    // 26. 模板方法，多态调用DubboInvoker#doInvoke => DubboInvoker为多态实例的原因见：RegistryDirectory.refreshInvoker.toInvokers#protocol.refer
    protected abstract Result doInvoke(Invocation invocation) throws Throwable;
}

public class DubboInvoker<T> extends AbstractInvoker<T> {
    @Override
    protected Result doInvoke(final Invocation invocation) throws Throwable {
        ...
        // 27. 如果不为oneway单向传输，则向远程服务发起consumer#request请求
        CompletableFuture<Object> responseFuture = currentClient.request(inv, timeout);
        ...
    }
}

final class ReferenceCountExchangeClient implements ExchangeClient {
    @Override
    public CompletableFuture<Object> request(Object request, int timeout) throws RemotingException {
        // 28. 如果不为oneway单向传输，则向远程服务发起consumer#request请求
        return client.request(request, timeout);
    }
}

public class HeaderExchangeClient implements ExchangeClient {
    @Override
    public CompletableFuture<Object> request(Object request, int timeout) throws RemotingException {
        // 29. 如果不为oneway单向传输，则向远程服务发起consumer#request请求
        return channel.request(request, timeout);
    }
}

final class HeaderExchangeChannel implements ExchangeChannel {
    @Override
    public CompletableFuture<Object> request(Object request, int timeout) throws RemotingException {
        ...
        // 30. 如果不为oneway单向传输，则向远程服务发起consumer#request请求
        channel.send(req);
        ...
    }
}

public abstract class AbstractPeer implements Endpoint, ChannelHandler {
    @Override
    public void send(Object message) throws RemotingException {
        // 31. 如果不为oneway单向传输，则向远程服务发起consumer#request请求
        send(message, url.getParameter(Constants.SENT_KEY, false));
    }
}

final class NettyChannel extends AbstractChannel {
    @Override
    public void send(Object message, boolean sent) throws RemotingException {
        ...
        // 32. 最后使用Netty客户端channel，向远程服务发起consumer#request请求
        ChannelFuture future = channel.writeAndFlush(message);
        ...
    }
}
```

#### 2、提供方接收并响应

```java
public class NettyServer extends AbstractServer implements Server {
    @Override
    protected void doOpen() throws Throwable {
        // 0. 利用Netty语法，打开Netty服务器，同时指定InternalDecoder解码器、InternalEncoder编码器、NettyServerHandler通道处理器
        ...// 省略Netty语法
    }
}

public class NettyServerHandler extends ChannelDuplexHandler {
    // 1. Netty读事件
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        NettyChannel channel = NettyChannel.getOrAddChannel(ctx.channel(), url, handler);
        // 2. 当有请求数据时，则调用handler进行接收、处理
        handler.received(channel, msg);
    }
}

public abstract class AbstractPeer implements Endpoint, ChannelHandler {
    @Override
    public void received(Channel ch, Object msg) throws RemotingException {
        ...
        // 3. 当有请求数据时，则调用handler进行接收、处理
        handler.received(ch, msg);
    }
}

public class HeartbeatHandler extends AbstractChannelHandlerDelegate {
    @Override
    public void received(Channel channel, Object message) throws RemotingException {
        ...
        // 4. 当有请求数据时，则调用handler进行接收、处理
        handler.received(channel, message);
    }
}

public class AllChannelHandler extends WrappedChannelHandler {
    @Override
    public void received(Channel channel, Object message) throws RemotingException {
        ExecutorService executor = getPreferredExecutorService(message);
        // 4. 当有请求数据时，使用线程池异步接收、处理
        executor.execute(new ChannelEventRunnable(channel, handler, ChannelState.RECEIVED, message));
        ...
    }
}

public class ChannelEventRunnable implements Runnable {
    @Override
    public void run() {
        // 5. 数据接收事件，继续调用handler进行接收、处理
        if (state == ChannelState.RECEIVED) {
            handler.received(channel, message);
        } else {
           .... 
        }
    }
}

public class DecodeHandler extends AbstractChannelHandlerDelegate {
    @Override
    public void received(Channel channel, Object message) throws RemotingException {
        ...
        // 6. 继续调用handler进行接收、处理
        handler.received(channel, message);
    }
}

public class HeaderExchangeHandler implements ChannelHandlerDelegate {
    @Override
    public void received(Channel channel, Object message) throws RemotingException {
        ...
        // 7. 如果报文属于来回传输类型，则调用handleRequest接收、处理，并响应客户端
        if (request.isTwoWay()) {
            handleRequest(exchangeChannel, request);
        }
        // 而如果报文属于onoWay单程传输类型，则调用received只接收、处理，不会响应客户端
        else {
            handler.received(exchangeChannel, request.getData());
        }
        ...
    }
    
    void handleRequest(final ExchangeChannel channel, Request req) throws RemotingException {
        ...
        // 8. 调用reply处理方法，得到返回结果future
        CompletionStage<Object> future = handler.reply(channel, msg);
        future.whenComplete((appResult, t) -> {
            try {
                if (t == null) {
                    res.setStatus(Response.OK);
                    res.setResult(appResult);
                } else {
                    res.setStatus(Response.SERVICE_ERROR);
                    res.setErrorMessage(StringUtils.toString(t));
                }
                
                // 20. 最后把结果返回写回给客户端，发送流程类似于客户端请求流程（代码略）
                channel.send(res);
            } catch (RemotingException e) {
                logger.warn("Send result to consumer failed, channel is " + channel + ", msg is " + e);
            }
        });
        ...
    }
}

public class DubboProtocol extends AbstractProtocol {
    private ExchangeHandler requestHandler = new ExchangeHandlerAdapter() {
        @Override
        public CompletableFuture<Object> reply(ExchangeChannel channel, Object message) throws RemotingException {
            ...
			// 9. 先根据报文，提取对应的Invoker
            Invoker<?> invoker = getInvoker(channel, inv);
            ...
            // 13. 调用提取到的Invoker#invoker方法
            Result result = invoker.invoke(inv);
            
            // 19. 最后把结果返回到外层
            return result.completionFuture().thenApply(Function.identity());
        }
    }
    
    Invoker<?> getInvoker(Channel channel, Invocation inv) throws RemotingException  {
        ...
        // 10. 端口+url+版本号+组号，组装成servicekey
        String serviceKey = serviceKey(port, path, inv.getAttachments().get(VERSION_KEY), inv.getAttachments().get(GROUP_KEY)); 
        // 11. 根据servicekey从之前缓存起来的exports中，提取出exporter
        DubboExporter<?> exporter = (DubboExporter<?>) exporterMap.get(serviceKey);
        ...
        // 12. 然后返回exporter中的Invoker
        return exporter.getInvoker();
    }
}

public class ProtocolFilterWrapper implements Protocol {
    private static <T> Invoker<T> buildInvokerChain(final Invoker<T> invoker, String key, String group) {
        ...
        last = new Invoker<T>() {
            public Result invoke(Invocation invocation) throws RpcException {
                // 14. invoker方法调用前，被过滤器拦截
                Result asyncResult = filter.invoke(last, invocation);
                return asyncResult;
            }
        }
    }
}

public class EchoFilter implements Filter {
    @Override
    public Result invoke(Invoker<?> invoker, Invocation inv) throws RpcException {
        ...
        // 15. invoker方法调用前，被过滤器拦截
        return invoker.invoke(inv);
    }
}

// ... 可扩展一大堆包装类和过滤器

public abstract class AbstractProxyInvoker<T> implements Invoker<T> {
    @Override
    public Result invoke(Invocation invocation) throws RpcException {
        // 16. invoker方法调用前，过滤器、监听器、包装类执行后，还需要先执行父类的doInvoke方法
        Object value = doInvoke(proxy, invocation.getMethodName(), invocation.getParameterTypes(), invocation.getArguments());
        ...
    }
    
    // 17. 模板方法，多态调用AbstractProxyInvoker#doInvoke方法
    protected abstract Object doInvoke(T proxy, String methodName, Class<?>[] parameterTypes, Object[] arguments) throws Throwable;
}

public class JavassistProxyFactory extends AbstractProxyFactory {
    @Override
    public <T> Invoker<T> getInvoker(T proxy, Class<T> type, URL url) {
        final Wrapper wrapper = Wrapper.getWrapper(proxy.getClass().getName().indexOf('$') < 0 ? proxy.getClass() : type);
        return new AbstractProxyInvoker<T>(proxy, type, url) {
            @Override
            protected Object doInvoke(T proxy, String methodName,
                                      Class<?>[] parameterTypes,
                                      Object[] arguments) throws Throwable {
                // 18. 最后才调用回实际接口实现类的方法
                return wrapper.invokeMethod(proxy, methodName, parameterTypes, arguments);
            }
        };
    }
}
```

#### 3、消费方接收服务响应

```java
public class NettyClient extends AbstractClient {
    @Override
    protected void doOpen() throws Throwable {
        // 0. 利用Netty语法，使用客户端连接url连接服务，同时指定InternalDecoder解码器、InternalEncoder编码器、NettyClientHandler通道处理器
        ...// 省略Netty语法
    }
}

public class NettyClientHandler extends ChannelDuplexHandler {
    // 1. Netty读事件
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        NettyChannel channel = NettyChannel.getOrAddChannel(ctx.channel(), url, handler);
        // 2. 读事件触发后，调用handler进行接收、处理
        handler.received(channel, msg);
        ...
    }
}

public abstract class AbstractPeer implements Endpoint, ChannelHandler {
    @Override
    public void received(Channel ch, Object msg) throws RemotingException {
        ...
        // 3. 调用handler进行接收、处理
        handler.received(ch, msg);
    }
}

public class MultiMessageHandler extends AbstractChannelHandlerDelegate {
    @Override
    public void received(Channel channel, Object message) throws RemotingException {
        ...
        // 4. 调用handler进行接收、处理
        handler.received(channel, message);
    }
}

public class HeartbeatHandler extends AbstractChannelHandlerDelegate {
    @Override
    public void received(Channel channel, Object message) throws RemotingException {
        ...
        // 5. 调用handler进行接收、处理
        handler.received(channel, message);
    }
}

public class AllChannelHandler extends WrappedChannelHandler {
    @Override
    public void received(Channel channel, Object message) throws RemotingException {
        ExecutorService executor = getExecutorService();
        // 6. 利用线程池，异步调用handler进行接收、处理
        executor.execute(new ChannelEventRunnable(channel, handler, ChannelState.RECEIVED, message));
        ...
    }
}

public class ChannelEventRunnable implements Runnable {
    @Override
    public void run() {
        // 7. 如果为数据接收类型，则继续调用handler进行接收、处理
        if (state == ChannelState.RECEIVED) {
            handler.received(channel, message);
        } else {
            ...
        }
    }
}

public class HeaderExchangeHandler implements ChannelHandlerDelegate {
    @Override
    public void received(Channel channel, Object message) throws RemotingException {
        if (message instanceof Request) {
            ...
        } else {
            // 8. 如果为响应报文，则继续调用handleResponse进行接收、处理
            handleResponse(channel, (Response) message);
        } else {
            ...
        }
    }
    
    static void handleResponse(Channel channel, Response response) throws RemotingException {
        if (response != null && !response.isHeartbeat()) {
            // 9. 如果为响应报文，则继续调用received进行接收、处理
            DefaultFuture.received(channel, response);
        }
    }
}

public class DefaultFuture extends CompletableFuture<Object> {
    public static void received(Channel channel, Response response) {
        // 10. 继续调用received进行接收、处理
        received(channel, response, false);
    }
    
    public static void received(Channel channel, Response response, boolean timeout) {
        ...
        // 11. 继续调用received进行接收、处理
        future.doReceived(response);
        ...
    }
    
    private void doReceived(Response res) {
 		...
        if (res.getStatus() == Response.OK) {
            // 12. 如果报文响应OK=20，则取出响应数据，设置到之前返回的future中
            this.complete(res.getResult());
        } else {
            ...
        }
    }
}
```

### 2.6. Dubbo 协议编解码原理？

#### 1、协议格式

![1637153667359](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1637153667359.png)

Dubbo 协议，通过消息头（16 字节 = 4 + 8 + 4）+ 消息体（不限），来解决 TCP 拆包、粘包的问题。

| 头属性           | 长度   | 作用                                                         |
| ---------------- | ------ | ------------------------------------------------------------ |
| Magic            | 16 bit | 魔数，标识 Dubbo 协议                                        |
| Req/Res          | 1 bit  | 标识报文是请求（1）还是响应（0）                             |
| 2 Way            | 1 bit  | 双向或单向的标记 ，仅当 Req/Res 为 1（请求）时才有用，如果需要来自服务器的返回值，则会设置为 1 |
| Event            | 1 bit  | 标识事件消息与否，比如心跳事件，如果这是一个事件，则设置为 1，而请求是没有的 |
| Serialization ID | 5 bit  | 标识序列化类型，比如 fastjson 的值为 6                       |
| Status           | 8 bit  | 仅在 Req/Res 为 0（响应）时有用，标识响应状态：20 - OK，30 - CLIENT_TIMEOUT，31 - SERVER_TIMEOUT，40 - BAD_REQUEST，50 - BAD_RESPONSE，60 - SERVICE_NOT_FOUND，70 - SERVICE_ERROR，80 - SERVER_ERROR，90 - CLIENT_ERROR，100 - SERVER_THREADPOOL_EXHAUSTED_ERROR |
| Request ID       | 64 bit | 请求 ID，标识唯一的请求，long 类型数字                       |
| Data Length      | 32 bit | 请求体序列化后的长度，以字节为单位，int 类型数字             |
| Variable Part    | 不限   | 请求体，长度可变，每个部分是序列化后的字节数组，由序列化ID标识，最大长度为 Integer.MAX_VALUE |
#### 2、消费方编码请求

```java
public class NettyClient extends AbstractClient {
    @Override
    protected void doOpen() throws Throwable {
        // 0. 利用Netty语法，使用客户端连接url连接服务，同时指定InternalDecoder解码器、InternalEncoder编码器、NettyClientHandler通道处理器
        ...// 省略Netty语法
    }
}

final public class NettyCodecAdapter {
    private class InternalEncoder extends MessageToByteEncoder {
        @Override
        protected void encode(ChannelHandlerContext ctx, Object msg, ByteBuf out) throws Exception {
            ...
            // 1. 调用codec进行编码
            codec.encode(channel, buffer, msg);
        }
    }
}

public final class DubboCountCodec implements Codec2 {
    @Override
    public void encode(Channel channel, ChannelBuffer buffer, Object msg) throws IOException {
        // 2. 调用codec进行编码
        codec.encode(channel, buffer, msg);
    }
}

public class ExchangeCodec extends TelnetCodec {
    @Override
    public void encode(Channel channel, ChannelBuffer buffer, Object msg) throws IOException {
        // 3. 如果消息为请求类型，则对请求进行编码
        if (msg instanceof Request) {
            encodeRequest(channel, buffer, (Request) msg);
        } else if (msg instanceof Response) {
            ...
        } else {
            ...
        }
    }
    
    protected void encodeRequest(Channel channel, ChannelBuffer buffer, Request req) throws IOException {
        ...
        // 4. 初始化消息头=16字节长的字节数组
        byte[] header = new byte[HEADER_LENGTH];
        
        // 5. 2字节长的魔数
        Bytes.short2bytes(MAGIC, header);
        
        // 6. 1字节长的请求标识、来回标识、序列化类型
        header[2] = (byte) (FLAG_REQUEST | serialization.getContentTypeId());
        if (req.isTwoWay()) {
            header[2] |= FLAG_TWOWAY;
        }
        // 7. 消费请求报文，没有Event事件标识，也没有status状态标识
        if (req.isEvent()) {
            header[2] |= FLAG_EVENT;
        }
        
        // 8. 8字节长的请求唯一ID
        Bytes.long2bytes(req.getId(), header, 4);
        ...
        // 9. 4字节长的请求体序列化后的长度
        int len = bos.writtenBytes();
        checkPayload(channel, len);
        Bytes.int2bytes(len, header, 12);
        ...
        // 10. write header.
        buffer.writeBytes(header); 
        ...
    }
}
```

#### 3、提供方解码请求

```java
public class NettyServer extends AbstractServer implements Server {
    @Override
    protected void doOpen() throws Throwable {
        // 0. 利用Netty语法，打开Netty服务器，同时指定InternalDecoder解码器、InternalEncoder编码器、NettyServerHandler通道处理器
        ...// 省略Netty语法
    }
}

final public class NettyCodecAdapter {
    private class InternalDecoder extends ByteToMessageDecoder {
        @Override
        protected void decode(ChannelHandlerContext ctx, ByteBuf input, List<Object> out) throws Exception {
            ...
            // 1. 调用codec解码
            Object msg = codec.decode(channel, message);
            ...
        }
    }
}

public final class DubboCountCodec implements Codec2 {
    @Override
    public Object decode(Channel channel, ChannelBuffer buffer) throws IOException {
        ...
        // 2. 调用codec解码
        Object obj = codec.decode(channel, buffer);
        ...
    }
}

public class ExchangeCodec extends TelnetCodec {
    @Override
    public Object decode(Channel channel, ChannelBuffer buffer) throws IOException {
        int readable = buffer.readableBytes();
        
        // 3. 读取16字节长的消息头
        byte[] header = new byte[Math.min(readable, HEADER_LENGTH)];
        buffer.readBytes(header);
        
        // 4. 校验消息头，并解码消息体
        return decode(channel, buffer, readable, header);
    }
}
```

#### 4、提供方编码响应

```java
public class NettyServer extends AbstractServer implements Server {
    @Override
    protected void doOpen() throws Throwable {
        // 0. 利用Netty语法，打开Netty服务器，同时指定InternalDecoder解码器、InternalEncoder编码器、NettyServerHandler通道处理器
        ...// 省略Netty语法
    }
}

final public class NettyCodecAdapter {
    private class InternalEncoder extends MessageToByteEncoder {
        @Override
        protected void encode(ChannelHandlerContext ctx, Object msg, ByteBuf out) throws Exception {
            ...
            // 1. 调用codec编码
            codec.encode(channel, buffer, msg);
        }
    }
}

public final class DubboCountCodec implements Codec2 {
    @Override
    public void encode(Channel channel, ChannelBuffer buffer, Object msg) throws IOException {
        // 2. 调用codec编码
        codec.encode(channel, buffer, msg);
    }
}

public class ExchangeCodec extends TelnetCodec {
    @Override
    public void encode(Channel channel, ChannelBuffer buffer, Object msg) throws IOException {
        if (msg instanceof Request) {
            ...
        } 
        // 3. 如果消息为响应类型，则对响应进行编码
        else if (msg instanceof Response) {
            encodeResponse(channel, buffer, (Response) msg);
        } else {
            ...
        }
    }
    
    protected void encodeResponse(Channel channel, ChannelBuffer buffer, Response res) throws IOException {
        // 4. 初始化消息头=16字节长的字节数组
        byte[] header = new byte[HEADER_LENGTH];
        
        // 5. 2字节长的魔数
        Bytes.short2bytes(MAGIC, header);
        
        // 6. 1字节长的请求标识、来回标识、序列化类型、Event事件标识（响应报文才有）
        header[2] = serialization.getContentTypeId();
        if (res.isHeartbeat()) {
            header[2] |= FLAG_EVENT;
        }
        
        // 7. 1字节长的响应状态码(响应报文才有)
        byte status = res.getStatus();
        header[3] = status;
        
        // 8. 8字节长的请求唯一标识
        Bytes.long2bytes(res.getId(), header, 4);
        ...
        // 9. 4字节长的请求体序列化后的长度
        int len = bos.writtenBytes();
        checkPayload(channel, len);
        Bytes.int2bytes(len, header, 12);
        ...
        // 10. write header.
        buffer.writeBytes(header); 
        ...
    }
}
```

#### 5、消费方解码响应

```java
public class NettyClient extends AbstractClient {
    @Override
    protected void doOpen() throws Throwable {
        // 0. 利用Netty语法，使用客户端连接url连接服务，同时指定InternalDecoder解码器、InternalEncoder编码器、NettyClientHandler通道处理器
        ...// 省略Netty语法
    }
}

final public class NettyCodecAdapter {
    private class InternalDecoder extends ByteToMessageDecoder {
        @Override
        protected void decode(ChannelHandlerContext ctx, ByteBuf input, List<Object> out) throws Exception {
            ...
			// 1. 调用codec解码
            Object msg = codec.decode(channel, message);
            ...
        }
    }
}

public final class DubboCountCodec implements Codec2 {
    @Override
    public Object decode(Channel channel, ChannelBuffer buffer) throws IOException {
        ...
        // 2. 调用codec解码
        Object obj = codec.decode(channel, buffer);
        ...
    }
}

public class ExchangeCodec extends TelnetCodec {
    @Override
    public Object decode(Channel channel, ChannelBuffer buffer) throws IOException {
        int readable = buffer.readableBytes();
        
        // 3. 读取16字节长的消息头
        byte[] header = new byte[Math.min(readable, HEADER_LENGTH)];
        buffer.readBytes(header);
        
        // 4. 校验消息头，并解码消息体
        return decode(channel, buffer, readable, header);
    }
}
```

### 2.7. Dubbo 集群容错原理？

#### 集群容错策略

在集群**调用失败**时或者**发起调用前**，Dubbo 提供 6 种集群容错方案，缺省为 `failover` 失败自动切换，可通过 SPI 自行扩展。

| 集群模式          | 作用                                               | 适用场景                                                     |
| ----------------- | -------------------------------------------------- | ------------------------------------------------------------ |
| Failover Cluster  | 默认配置，失败自动切换，当出现失败，重试其它服务器 | 通常用于读操作，但重试会带来更长延迟，可通过 `retries="2"` 来设置额外重试次数（不含第一次） |
| Failfast Cluster  | 快速失败，只发起一次调用，失败则立即报错           | 通常用于非幂等性的写操作，比如新增记录                       |
| Failsafe Cluster  | 安全失败，出现异常时，会直接忽略                   | 通常用于写入审计日志等操作                                   |
| Failback Cluster  | 失败自动恢复，后台记录失败请求，然后定时重发       | 通常用于消息通知操作                                         |
| Forking Cluster   | 并行调用多个服务器，只要一个成功即返回             | 通常用于实时性要求较高的读操作，但需要浪费更多服务资源，可通过 `forks="2"` 来设置最大并行数 |
| Broadcast Cluster | 广播调用所有提供者，逐个调用，任意一台报错则报错   | 通常用于通知所有提供者更新缓存或者日志等本地资源信息，可以通过 `broadcast.fail.percent` 配置节点调用失败的比例，当达到这个比例后，将不再调用其他节点，而是直接抛出异常，取值范围为 0 ~ 100，默认情况下当全部调用失败后，才会抛出异常 |

#### 策略实现原理

```java
public abstract class AbstractClusterInvoker<T> implements Invoker<T> {
    @Override
    public Result invoke(final Invocation invocation) throws RpcException {
        ...
        // 0.1. 进入目录查找，从this.methodInvokerMap里面查找一个Invoker
        List<Invoker<T>> invokers = list(invocation);
        ...
        
        // 0.2. 进入集群，交由集群路由执行
        return doInvoke(invocation, invokers, loadbalance);
    }
    
    // 0.3. 模板方法，交由*ClusterInvoker#doInvoke执行
    protected abstract Result doInvoke(Invocation invocation, List<Invoker<T>> invokers,LoadBalance loadbalance) throws RpcException;
}
```

##### Failover | 失败重试

```java
public class FailoverClusterInvoker<T> extends AbstractClusterInvoker<T> {
    @Override
    public Result doInvoke(Invocation invocation, final List<Invoker<T>> invokers, LoadBalance loadbalance) throws RpcException {
        ...
        // 1. 获取retries参数，默认额外重试2次
        int len = getUrl().getMethodParameter(methodName, RETRIES_KEY, DEFAULT_RETRIES) + 1;

        // 2. 这里retries再加1，代表共尝试3次
        for (int i = 0; i < len; i++) {
            try {
                Result result = invoker.invoke(invocation);
                return result;
            } catch (RpcException e) {
                if (e.isBiz()) { // biz exception.
                    throw e;
                }
                le = e;
            } catch (Throwable e) {
                // 3. catch住异常，然后循环重试
                le = new RpcException(e.getMessage(), e);
            } finally {
                providers.add(invoker.getUrl().getAddress());
            }
        }
        
        // 4. 重试完毕后，如果还没返回结果，说明全部都失败了，则抛出异常
        throw new RpcException(...);
    }
}
```

##### Failfast | 快速失败

```java
public class FailfastClusterInvoker<T> extends AbstractClusterInvoker<T> {
    @Override
    public Result doInvoke(Invocation invocation, List<Invoker<T>> invokers, LoadBalance loadbalance) throws RpcException {
        ...
        try {
            return invoker.invoke(invocation);
        } catch (Throwable e) {
            if (e instanceof RpcException && ((RpcException) e).isBiz()) { // biz exception.
                throw (RpcException) e;
            }
            
            // 1. 发生异常直接抛出
            throw new RpcException(...);
        }
    }
}
```

##### Failsafe | 安全失败

```java
public class FailsafeClusterInvoker<T> extends AbstractClusterInvoker<T> {
    @Override
    public Result doInvoke(Invocation invocation, List<Invoker<T>> invokers, LoadBalance loadbalance) throws RpcException {
        try {
            ...
            return invoker.invoke(invocation);
        } catch (Throwable e) {
            // 1. 发生异常只打印到日志中，然后返回null
            logger.error("Failsafe ignore exception: " + e.getMessage(), e);
            return AsyncRpcResult.newDefaultAsyncResult(null, null, invocation); // ignore
        }
    }
}
```

##### Failback | 失败自动恢复

```java
public class FailbackClusterInvoker<T> extends AbstractClusterInvoker<T> {

    @Override
    protected Result doInvoke(Invocation invocation, List<Invoker<T>> invokers, LoadBalance loadbalance) throws RpcException {
        Invoker<T> invoker = null;
        try {
            ...
            return invoker.invoke(invocation);
        } catch (Throwable e) {
            // 1. 发生异常不仅打印到日志中
            logger.error(...);
            
            // 2. 还设置定时任务，重试执行，并返回null，后面的重试再无返回值
            addFailed(loadbalance, invocation, invokers, invoker);
            return AsyncRpcResult.newDefaultAsyncResult(null, null, invocation); // ignore
        }
    }
    
    private void addFailed(LoadBalance loadbalance, Invocation invocation, List<Invoker<T>> invokers, Invoker<T> lastInvoker) {
        // 3. 双重检查锁创建定时器
        if (failTimer == null) {
            synchronized (this) {
                if (failTimer == null) {
                    failTimer = new HashedWheelTimer(
                            new NamedThreadFactory("failback-cluster-timer", true),
                            1,
                            TimeUnit.SECONDS, 32, failbackTasks);
                }
            }
        }
        
        // 4. 创建重试任务，每5s执行一次
        RetryTimerTask retryTimerTask = new RetryTimerTask(loadbalance, invocation, invokers, lastInvoker, retries, RETRY_FAILED_PERIOD);
        try {
            failTimer.newTimeout(retryTimerTask, RETRY_FAILED_PERIOD, TimeUnit.SECONDS);
        } catch (Throwable e) {
            logger.error("Failback background works error,invocation->" + invocation + ", exception: " + e.getMessage());
        }
    }
    
    private class RetryTimerTask implements TimerTask {
        @Override
        public void run(Timeout timeout) {
            try {
                ...
                // 5. 发起重新执行，无返回值，会一直重试！
                retryInvoker.invoke(invocation);
            } catch (Throwable e) {
                logger.error(...);
                if ((++retryTimes) >= retries) {
                    logger.error(...);
                } else {
                    // 6. 超过了最大重试次数retries，默认共3次，则重新添加定时任务，继续重试
                    rePut(timeout);
                }
            }
        }
        
        private void rePut(Timeout timeout) {
            if (timeout == null) {
                return;
            }

            Timer timer = timeout.timer();
            if (timer.isStop() || timeout.isCancelled()) {
                return;
            }

            // 7. 超过了最大重试次数retries，默认共3次，则重新添加定时任务，继续重试
            timer.newTimeout(timeout.task(), tick, TimeUnit.SECONDS);
        }
    }
}
```

##### Forking | 并行调用

```java
public class ForkingClusterInvoker<T> extends AbstractClusterInvoker<T> {
    
    // 0. 并行调用的缓存线程池
    private final ExecutorService executor = Executors.newCachedThreadPool(
            new NamedInternalThreadFactory("forking-cluster-timer", true));
    
    @Override
    public Result doInvoke(final Invocation invocation, List<Invoker<T>> invokers, LoadBalance loadbalance) throws RpcException {
        final List<Invoker<T>> selected;
        
        // 1. 获取并行调用数量，默认为2
        final int forks = getUrl().getParameter(FORKS_KEY, DEFAULT_FORKS);
        
        // 2. 设置并行调用的Invoker列表
        selected = new ArrayList<>();
        for (int i = 0; i < forks; i++) {
            Invoker<T> invoker = select(loadbalance, invocation, invokers, selected);
            if (!selected.contains(invoker)) {.
                selected.add(invoker);
            }
        }
        
        // 3. 遍历并行调用的Invoker列表，交由线程池并行调用
        final BlockingQueue<Object> ref = new LinkedBlockingQueue<>();
        for (final Invoker<T> invoker : selected) {
            executor.execute(new Runnable() {
                @Override
                public void run() {
                    try {
                        Result result = invoker.invoke(invocation);
                        
                        // 4. 执行结果添加到LinkedBlockingQueue中
                        ref.offer(result);
                    } catch (Throwable e) {
                        int value = count.incrementAndGet();
                        if (value >= selected.size()) {
                            ref.offer(e);
                        }
                    }
                }
            });
        }
        
        try {
            // 5. 超时阻塞式从LinkedBlockingQueue中，取出第一个结果并返回，如果结果异常则抛出
            Object ret = ref.poll(timeout, TimeUnit.MILLISECONDS);
            if (ret instanceof Throwable) {
                Throwable e = (Throwable) ret;
                throw new RpcException(...);
            }
            return (Result) ret;
        } catch (InterruptedException e) {
            throw new RpcException(...);
        }
        
        ...
    }
}
```

##### Broadcast | 逐个广播式调用

```java
public class BroadcastClusterInvoker<T> extends AbstractClusterInvoker<T> {
    @Override
    public Result doInvoke(final Invocation invocation, List<Invoker<T>> invokers, LoadBalance loadbalance) throws RpcException {
        RpcException exception = null;
        Result result = null;
        
        // 1. 不做任何的负载均衡
        for (Invoker<T> invoker : invokers) {
            try {
                // 2. 每个Invoker都执行一遍
                result = invoker.invoke(invocation);
            } catch (RpcException e) {
                exception = e;
                logger.warn(e.getMessage(), e);
            } catch (Throwable e) {
                exception = new RpcException(e.getMessage(), e);
                logger.warn(e.getMessage(), e);
            }
        }
        
        // 2. 任何一台报错则报错
        if (exception != null) {
            throw exception;
        }
        
        // 3. 只返回最后一次执行成功的结果
        return result;
    }
}
```

### 2.8. Dubbo 负载均衡原理？

#### 负载均衡策略

在集群负载均衡时，Dubbo 提供了 5 种负载均衡策略，缺省为 `random` 随机调用，可通过 SPI 自行扩展。

1. ⼀个请求从客户端发起，⽐如查询订单列表，要选择服务器进⾏处理，但是集群环境提供了 5 个服务器，每个服务器都有处理这个请求的能⼒，此时，客户端就必须选择⼀个服务器来进⾏处理，说⽩了，负载均衡就是⼀个选择的问题。
2. 当请求多了，负载均衡的优点则得以体现：可以均衡各服务器的负载，避免单个服务器响应同⼀请求，而导致的服务器宕机、崩溃等问题。

| 负载均衡策略                 | 作用                                                         | 特点                                                         |
| ---------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Random LoadBalance           | 默认策略，随机，按权重设置随机概率                           | 在一个截面上碰撞的概率高，但调用量越大分布越**均匀**，而且按概率使用权重后也比较均匀，有利于动态调整提供者权重 |
| RoundRobin LoadBalance       | 轮训，按公约后的权重设置轮训比率                             | 存在慢的提供者**累积请求**的问题，比如：第二台机器很慢，但没挂，当请求调到第二台时就卡在那，久而久之，所有请求都卡在调到第二台上 |
| LeastActive LoadBalance      | 最少活跃调用数，相同活跃数的则随机，活跃数指并发调用的数量   | 使并发少的提供者收到**更多请求**，因为并发越少的提供者，压力更小；使并发多的提供者收到**更少请求**，因为并发越的多提供者，压力更大 |
| ShortestResponse LoadBalance | 最短响应负载，优先选择平均调用成功时长短、响应效率高，而相同响应时长的则随机选取 | 使快的提供者收到**更多请求**，因为越快的提供者，调用的平均成功时长越小；使慢的提供者收到**更少请求**，因为越慢的提供者，调用的平均成功时长越大 |
| ConsistentHash LoadBalance   | 一致性 Hash，相同参数的请求总是发到同一提供者，缺省用 160 份虚拟节点，可通过 `dubbo:parameter` 进行修改 | 当某一台提供者挂时，原本发往该提供者的请求，基于虚拟节点，平摊到其它提供者，**不会引起剧烈变动** |

#### 策略实现原理

```java
public abstract class AbstractClusterInvoker<T> implements Invoker<T> {
    
    // 0.1. 从具体的ClusterInvoker#doInvoke中回调
    protected Invoker<T> select(LoadBalance loadbalance, Invocation invocation,
                                List<Invoker<T>> invokers, List<Invoker<T>> selected) throws RpcException {
        ...
		// 0.2. 进入负载均衡
        Invoker<T> invoker = doSelect(loadbalance, invocation, invokers, selected);
        ...
    }
    
    private Invoker<T> doSelect(LoadBalance loadbalance, Invocation invocation,
                                List<Invoker<T>> invokers, List<Invoker<T>> selected) throws RpcException {
        ...
        // 0.3. 交由AbstractLoadBalance决定哪个路由规则进行负载均衡
        Invoker<T> invoker = loadbalance.select(invokers, getUrl(), invocation);
        ...
    }
}

public abstract class AbstractLoadBalance implements LoadBalance {
    @Override
    public <T> Invoker<T> select(List<Invoker<T>> invokers, URL url, Invocation invocation) {
        ...
        // 0.4. 交由AbstractLoadBalance决定哪个路由规则进行负载均衡
        return doSelect(invokers, url, invocation);
    }
    
    // 0.5. 模板方法，默认交由RandomLoadBalance进行负载均衡
	protected abstract <T> Invoker<T> doSelect(List<Invoker<T>> invokers, URL url, Invocation invocation);
    
    // 0.6. 根据会话状态获取路由权重
    int getWeight(Invoker<?> invoker, Invocation invocation) {
        int weight;
        URL url = invoker.getUrl();

        // 0.7. 当有多个注册地址时,根据注册地址key 获取对应服务权重
        if (REGISTRY_SERVICE_REFERENCE_PATH.equals(url.getServiceInterface())) {
            weight = url.getParameter(REGISTRY_KEY + "." + WEIGHT_KEY, DEFAULT_WEIGHT);
        } else {
            // 0.8. 从url获取权重，默认100
            weight = url.getMethodParameter(invocation.getMethodName(), WEIGHT_KEY, DEFAULT_WEIGHT);
            if (weight > 0) {
                // 以下过程主要是，判断机器的启动时间和预热时间的差值(默认为10min), 为什么要预热? 
                // 1、服务预热是一个优化手段，与此类似的还有 JVM 预热，主要目的是让服务启动后“低功率”运行一段时间，使其效率慢慢提升至最佳状态，避免由此引发的调用超时问题。

                // 0.9. 获取启动时间
                long timestamp = invoker.getUrl().getParameter(TIMESTAMP_KEY, 0L);
                if (timestamp > 0L) {
                    // 0.91. 获取运行时间
                    long uptime = System.currentTimeMillis() - timestamp;
                    if (uptime < 0) {
                        return 1;
                    }
                    // 0.92. 获取服务预热时间，默认为10分钟
                    int warmup = invoker.getUrl().getParameter(WARMUP_KEY, DEFAULT_WARMUP);
                    if (uptime > 0 && uptime < warmup) {
                        // 0.93. 重新计算服务权重
                        weight = calculateWarmupWeight((int)uptime, warmup, weight);
                    }
                }
            }
        }

        return Math.max(weight, 0);
    }
    
    // 0.94. 重新计算服务权重，当服务运行时长小于服务预热时间时，则会对服务进行降权，以避免让服务在启动之初就处于高负载状态
    static int calculateWarmupWeight(int uptime, int warmup, int weight) {
        // 0.95. 计算权重，下面代码逻辑上形似于 (uptime / warmup) * weight，可见，随着服务运行时间 uptime 增大，权重计算值 ww 会慢慢接近配置值 weight
        int ww = (int) ( uptime / ((float) warmup / weight));
        return ww < 1 ? 1 : (Math.min(ww, weight));
    }
}
```

##### Random | 随机

1. 遍历 invokers 列表，获取它们降权后的路由权重，并累加它们的权重，然后比较它们的权重值是否相等。
2. 如果它们权重不相等，则根据权重和，求出一个**随机数**，并计算看该数落在权重数组的哪个区间段，然后取该区间段权重最大的 invoker 索引，去 invokers 集合中获取返回。
3. 如果它们权重都相等，则随机返回一个 invoker 即可。

```java
public class RandomLoadBalance extends AbstractLoadBalance {
    @Override
    protected <T> Invoker<T> doSelect(List<Invoker<T>> invokers, URL url, Invocation invocation) {
        int length = invokers.size()
        boolean sameWeight = true;
        int[] weights = new int[length];
        int totalWeight = 0;
        
        // 1. 累加总权重
        for (int i = 0; i < length; i++) {
            // 2. 获取路由权重（包括预热降权）
            int weight = getWeight(invokers.get(i), invocation);
            totalWeight += weight;
            weights[i] = totalWeight;
            if (sameWeight && totalWeight != weight * (i + 1)) {
                sameWeight = false;
            }
        }
        
        // 3. 如果服务路由中，存在不一样的权重，根据权重和求出一个随机数,然后计算看该数落在哪个区间段
        if (totalWeight > 0 && !sameWeight) {
            int offset = ThreadLocalRandom.current().nextInt(totalWeight);
            for (int i = 0; i < length; i++) {
                if (offset < weights[i]) {
                    // 4. 只取区间中最大权重的一个invoker
                    return invokers.get(i);
                }
            }
        }
        
        // 5. 如果权重相同或总权重为0，则随机选一个，多线程产生随机数，然后取对应索引的invoker
        return invokers.get(ThreadLocalRandom.current().nextInt(length));
    }
}
```

##### RoundRobin | 轮训

1. 遍历 invokers 列表，累加它们的权重。
2. 如果它们权重不相等，则返回**权重最大**的一个 invoker。
3. 如果它们权重都相等，则返回第一个 invoker。

```java
public class RoundRobinLoadBalance extends AbstractLoadBalance {
    @Override
    protected <T> Invoker<T> doSelect(List<Invoker<T>> invokers, URL url, Invocation invocation) {
        ...
        // 1. 遍历所有invoker
        for (Invoker<T> invoker : invokers) {
            ...
            // 2. 计算胡最大的路由权重，并取其invoker
            if (cur > maxCurrent) {
                maxCurrent = cur;
                selectedInvoker = invoker;
                selectedWRR = weightedRoundRobin;
            }
            totalWeight += weight;
        }
        if (selectedInvoker != null) {
            selectedWRR.sel(totalWeight);
            
            // 3. 返回权重最大的路由
            return selectedInvoker;
        }
        
        // 4. 如果所有路由权重都相同，则取第一个invoker
        return invokers.get(0);
    }
}
```

##### LeastActive | 最不活跃调用数

1. 遍历 invokers 列表，寻找**活跃数最小**的 Invoker。
2. 如果有多个 Invoker 具有相同的最小活跃数，则记录下这些 Invoker 在 invokers 集合中的下标，并累加它们的权重，比较它们的权重值是否相等。
3. 如果只有一个 Invoker 具有最小的活跃数，此时直接返回该 Invoker 即可。
4. 如果有多个 Invoker 具有最小活跃数，且它们的权重不相等，此时处理方式和 RandomLoadBalance 一致。
5. 如果有多个 Invoker 具有最小活跃数，但它们的权重相等，此时随机返回一个即可。

```java
public class LeastActiveLoadBalance extends AbstractLoadBalance {
    @Override
    protected <T> Invoker<T> doSelect(List<Invoker<T>> invokers, URL url, Invocation invocation) {
        int length = invokers.size();
        // 1. 最小的活跃数
        int leastActive = -1;
        // 2.具有相同“最小活跃数”的服务者提供者（以下用 Invoker 代称）数量
        int leastCount = 0;
        // 3. leastIndexs 用于记录具有相同“最小活跃数”的 Invoker 在 invokers 列表中的下标信息
        int[] leastIndexes = new int[length];
        int[] weights = new int[length];
        int totalWeight = 0;
        // 4. 第一个最小活跃数的 Invoker 权重值，用于与其他具有相同最小活跃数的 Invoker 的权重进行对比， 以检测是否“所有具有相同最小活跃数的 Invoker 的权重”均相等
        int firstWeight = 0;
        boolean sameWeight = true;

        // 5. 遍历 invokers 列表
        for (int i = 0; i < length; i++) {
            Invoker<T> invoker = invokers.get(i);
            // 6. 获取 Invoker 对应的活跃数，记录方式见《限流原理》
            int active = RpcStatus.getStatus(invoker.getUrl(), invocation.getMethodName()).getActive();
            // 7. 获取降权后的服务权重
            int afterWarmup = getWeight(invoker, invocation);
            weights[i] = afterWarmup;
            // 8. 对比查找更小的活跃数，重新开始
            if (leastActive == -1 || active < leastActive) {
                // 9. 使用当前活跃数 active 更新最小活跃数 leastActive
                leastActive = active;
                // 10. 更新 leastCount 为 1
                leastCount = 1;
                // 11. 记录当前下标值到 leastIndexs 中
                leastIndexes[0] = i;
                totalWeight = afterWarmup;
                firstWeight = afterWarmup;
                sameWeight = true;
            } 
  			// 12. 当前 Invoker 的活跃数 active 与最小活跃数 leastActive 相同
            else if (active == leastActive) {
                // 13. 在 leastIndexs 中记录下当前 Invoker 在 invokers 集合中的下标
                leastIndexes[leastCount++] = i;
                // 14. 累加权重
                totalWeight += afterWarmup;
                // 15. 检测当前 Invoker 的权重与 firstWeight 是否相等，不相等则将 sameWeight 置为 false
                if (sameWeight && afterWarmup != firstWeight) {
                    sameWeight = false;
                }
            }
        }

        // 16. 当只有一个 Invoker 具有最小活跃数，此时直接返回该 Invoker 即可
        if (leastCount == 1) {
            return invokers.get(leastIndexes[0]);
        }

        // 17. 有多个 Invoker 具有相同的最小活跃数，但它们之间的权重不同
        if (!sameWeight && totalWeight > 0) {
            // 18. 则随机生成一个 [0, totalWeight) 之间的数字
            int offsetWeight = ThreadLocalRandom.current().nextInt(totalWeight);
            // 19. 循环让随机数减去具有最小活跃数的 Invoker 的权重值， 当 offset 小于等于0时，返回相应的 Invoker
            for (int i = 0; i < leastCount; i++) {
                int leastIndex = leastIndexes[i];
                // 20. 获取权重值，并让随机数减去权重值
                offsetWeight -= weights[leastIndex];
                if (offsetWeight < 0) {
                    return invokers.get(leastIndex);
                }
            }
        }

        // 21. 如果权重相同或权重为0时，随机返回一个 Invoker
        return invokers.get(leastIndexes[ThreadLocalRandom.current().nextInt(leastCount)]);
    }
}
```

##### ShortestResponse | 最短响应负载

1.  找到服务下，所有提供者中**响应时间最短**的⼀个或多个，并计算其权重和。
2. 如果响应时间最短的服务提供者只有⼀个，则直接返回给服务。
3. 如果响应时间最短的服务提供者⼤于1个，则分为以下 2 种情况：
   1. 如果所有服务权重值不同，则按 RandomLoadBalance（2） 过程，选出服务提供者。
   2. 如果所有服务权重相同，则随机返回⼀个。

```java
public class ShortestResponseLoadBalance extends AbstractLoadBalance {
    @Override
    protected <T> Invoker<T> doSelect(List<Invoker<T>> invokers, URL url, Invocation invocation) {
		...
        // 1. 遍历所有invoker
        for (int i = 0; i < length; i++) {
            // 2. 获取每个invoker的rpcStatus（记录着各种调用统计）
            Invoker<T> invoker = invokers.get(i);
            RpcStatus rpcStatus = RpcStatus.getStatus(invoker.getUrl(), invocation.getMethodName());
            // 3. 获取平均成功调用时长 = 成功调用的总时长 / 成功调用总次数，记录方式见《限流原理》
            long succeededAverageElapsed = rpcStatus.getSucceededAverageElapsed();
            // 4. 获取此时并发调用数（活跃数，指从请求到未响应结果的调用），记录方式见《限流原理》
            int active = rpcStatus.getActive();
            // 5. 计算总响应时长 = 平均成功调用时长 * 并发调用数
            long estimateResponse = succeededAverageElapsed * active;
            // 6. 获取路由权重（包括预热降权）
            int afterWarmup = getWeight(invoker, invocation);
            weights[i] = afterWarmup;
            // 7. 判断路由最短响应时长、路由权重都相同
            if (estimateResponse < shortestResponse) {
                shortestResponse = estimateResponse;
                shortestCount = 1;
                shortestIndexes[0] = i;
                totalWeight = afterWarmup;
                firstWeight = afterWarmup;
                sameWeight = true;
            } else if (estimateResponse == shortestResponse) {
                shortestIndexes[shortestCount++] = i;
                totalWeight += afterWarmup;
                if (sameWeight && i > 0 && afterWarmup != firstWeight) {
                    sameWeight = false;
                }
            }
        }
        
        // 8. 如果路由最短响应时长的路由只有1个，则只返回第一个invoker
        if (shortestCount == 1) {
            return invokers.get(shortestIndexes[0]);
        }
        // 9. 如果不存在相同最短响应时长、相同权重的路由
        if (!sameWeight && totalWeight > 0) {
            // 10. 则随机计算权重
            int offsetWeight = ThreadLocalRandom.current().nextInt(totalWeight);
            // 11. 然后取响应时长中，某个区间内最响应时长最小的一个invoker
            for (int i = 0; i < shortestCount; i++) {
                int shortestIndex = shortestIndexes[i];
                offsetWeight -= weights[shortestIndex];
                if (offsetWeight < 0) {
                    return invokers.get(shortestIndex);
                }
            }
        }
        
        // 12. 如果所有路由最短响应时长、路由权重都相同，则随机取一个Invoker
        return invokers.get(shortestIndexes[ThreadLocalRandom.current().nextInt(shortestCount)]);
    }
}
```

##### ConsistentHash | 一致性哈希

- **概念**：⼀致性 hash 算法，由麻省理⼯学院的 Karger 及其合作者于 1997 年提出的，算法提出之初是⽤于⼤规模缓存系统的**负载均衡**。

- **⼯作过程**：

  ![1637325363460](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1637325363460.png)

  1. ⾸先根据 ip 或者其他的信息为缓存节点⽣成⼀个 hash，并将这个 hash 投射到 [0, 232 - 1] 的圆环上。
  2. 当有查询或写⼊请求时，则为缓存项的 key ⽣成⼀个 hash 值，然后查找**第⼀个**⼤于或等于该 hash 值的缓存节点，并到这个节点中查询或写⼊缓存项。
  3. 如果那个节点挂了，则在下⼀次查询或写⼊缓存时，为缓存项查找**另⼀个**⼤于其 hash 值的缓存节点即
     可。
  4. ⼤致效果如图所示，每个缓存节点在圆环上占据⼀个位置，如果缓存项的 key 的 hash 值⼩于缓存节点 hash 值，则到该缓存节点中存储或读取缓存项。
  5. ⽐如绿⾊点对应的缓存项将会被存储到 cache-2 节点中，而如果 cache-3 挂了，则原本应该存到该节点中的缓存项，最终会存储到 cache-4 节点中。

- **一致性 hash 在 Dubbo 中的应用**：

  ![1637325455225](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1637325455225.png)

  1. 首先把一致性 hash 环的缓存节点替换成 Dubbo#Invoker，相同颜色的则代表同属于同一个服务。
  2. ⽐如 Invoker1-1，Invoker1-2，……, Invoker1-160，这样做的⽬的是通过**引⼊虚拟节点**，让 Invoker 在圆环上分散开来，通过虚拟节点均衡各个节点的请求量，**避免数据倾斜问题**。
     - 所谓**数据倾斜**是指，由于节点不够分散，导致⼤量请求落到了同⼀个节点上，⽽其他节点只会接收到了少量请求的情况。

```java
public class ConsistentHashLoadBalance extends AbstractLoadBalance {
    
    private final ConcurrentMap<String, ConsistentHashSelector<?>> selectors = new ConcurrentHashMap<String, ConsistentHashSelector<?>>();
    
    @Override
    protected <T> Invoker<T> doSelect(List<Invoker<T>> invokers, URL url, Invocation invocation) {
        String methodName = RpcUtils.getMethodName(invocation);
        String key = invokers.get(0).getUrl().getServiceKey() + "." + 
methodName;
 
        // 1. 获取 invokers 原始的 hashcode，
        int identityHashCode = System.identityHashCode(invokers);
        ConsistentHashSelector<T> selector = (ConsistentHashSelector<T>) 
selectors.get(key);
        // 2. 由于 invokers 是⼀个新的 List 对象，如果selector.identityHashCode != identityHashCode，则意味着服务提供者数量发⽣了变化，因为正常情况下，key-Invokers是保持不变的
        if (selector == null || selector.identityHashCode != 
identityHashCode) {
            // 3. 则创建新的ConsistentHashSelector
            selectors.put(key, new ConsistentHashSelector<T>(invokers, 
methodName, identityHashCode));
            
            // 4. 更新为刚创建的selector
            selector = (ConsistentHashSelector<T>) selectors.get(key);
        }
        
        // 18. 调⽤ ConsistentHashSelector 的 select ⽅法选择 Invoker
        return selector.select(invocation);
    }
}

private static final class ConsistentHashSelector<T> {
    // 5. 使⽤ TreeMap 存储 Invoker 虚拟节点
    private final TreeMap<Long, Invoker<T>> virtualInvokers;
    // 6. 虚拟节点数量，默认为160
    private final int replicaNumber;
	// 7. selector的hashCode，用来标志invokers是否发生变化
    private final int identityHashCode;
	// 8. 方法参数索引，用于对指定某个方法实参进行hash
    private final int[] argumentIndex;
    
    ConsistentHashSelector(List<Invoker<T>> invokers, String methodName, 
                           int identityHashCode) {
		this.virtualInvokers = new TreeMap<Long, Invoker<T>>();
        this.identityHashCode = identityHashCode;
        URL url = invokers.get(0).getUrl();
        this.replicaNumber = url.getMethodParameter(methodName, 
"hash.nodes", 160);
        
        // 9. 获取参与 hash 计算的参数下标值，默认对第1个参数进⾏ hash 运算，可通过hash.arguments配置多个比如"0,1,2"等等
        String[] index = 
Constants.COMMA_SPLIT_PATTERN.split(url.getMethodParameter(methodName, 
"hash.arguments", "0"));
        argumentIndex = new int[index.length];
        for (int i = 0; i < index.length; i++) {
            argumentIndex[i] = Integer.parseInt(index[i]);
        }
        
        // 10. 遍历Invokers，准备建立虚拟节点
        for (Invoker<T> invoker : invokers) {
            String address = invoker.getUrl().getAddress();
            for (int i = 0; i < replicaNumber / 4; i++) {
                // 11. 对 address + i 进⾏ md5 运算，得到⼀个⻓度为16的字节数组
                byte[] digest = md5(address + i);
                // 12. 对 digest 部分字节进⾏ 4 次 hash 运算，得到四个不同的 long 型正整数
                for (int h = 0; h < 4; h++) {
                    // 13. h = 0 时，取 digest 中下标为 0 ~ 3 的4个字节进⾏位运算
                    // 14. h = 1 时，取 digest 中下标为 4 ~ 7 的4个字节进⾏位运算
                    // 15. h = 2 时，取 digest 中下标为 8 ~ 11 的4个字节进⾏位运算
                    // 16. h = 3 时，取 digest 中下标为 12 ~ 15 的4个字节进⾏位运算
                    long m = hash(digest, h);
                    // 17. 将（计算好的落环 hash 值 -> invoker 的映射关系）存储到 virtualInvokers 中，使用 TreeMap 提供⾼效的查询操作
                    virtualInvokers.put(m, invoker);
                }
            }
        }
    }
    
    // 13、14、15、16、23、虚拟节点哈希落环规则
    private long hash(byte[] digest, int number) {
        return (((long) (digest[3 + number * 4] & 0xFF) << 24)
                | ((long) (digest[2 + number * 4] & 0xFF) << 16)
                | ((long) (digest[1 + number * 4] & 0xFF) << 8)
                | (digest[number * 4] & 0xFF))
            & 0xFFFFFFFFL;
    }
    
    // 19. 调⽤ ConsistentHashSelector 的 select ⽅法选择 Invoker
    public Invoker<T> select(Invocation invocation) {
        // 20. 将参数转为 key
        String key = toKey(invocation.getArguments());
        // 22. 生成JDK MD5消息摘要
        byte[] digest = Bytes.getMD5(key);
        // 24. 根据落环后的hash值，寻找合适的 Invoker
        return selectForKey(
            // 23. h = 0 时，取 digest 中下标为 0 ~ 3 的4个字节进⾏位运算
            hash(digest, 0));
    }
    
    private String toKey(Object[] args) {
        StringBuilder buf = new StringBuilder();
        // 21. 根据hash.arguments配置的hash参数索引，进行拼凑成key，因此，同服务的Dubbo一致性哈希的负载均衡逻辑，只受参数值影响，具有相同参数值的请求将会被分配到同一个服务提供者，且没有权重的概念
        for (int i : argumentIndex) {
            if (i >= 0 && i < args.length) {
                buf.append(args[i]);
            }
        }
        return buf.toString();
    }
    
    private Invoker<T> selectForKey(long hash) {
        // 25. 根据落环的hash值，到 TreeMap 中查找第⼀个⼤于或等于当前 hash 的 Invoker
        Map.Entry<Long, Invoker<T>> entry = virtualInvokers.ceilingEntry(hash);
        // 26. 如果没找到，说明落环的 hash 值，已经⼤于所有 Invoker 在圆环上最⼤的位置，此时需要将 TreeMap 的头节点赋值给 entry，表示获取第一个 Invoker
        if (entry == null) {
            entry = virtualInvokers.firstEntry();
        }
        // 27. 最后返回找到的 Invoker，完成Dubbo一致性hash的负载均衡
        return entry.getValue();
    }
}
```

### 2.9. Dubbo 线程派发模型？

#### 请求派发策略

| 请求派发策略 | 特点                                                         | 对应 Netty Boss-Worker 模型                                  |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| all          | 全部派发，**默认**的请求派发策略，所有消息都派发到线程池，包括请求，响应，连接事件，断开事件，心跳等 | Worker 线程接收到事件后，只会把事件提交到线程池，⾃⼰去处理其他事情 |
| direct       | 直接执行，所有消息都不派发到线程池，全部在 IO 线程上直接执行 | Worker 线程接收到事件后，只会由自己执⾏到底                  |
| message      | 派发请求和响应，只有请求响应消息派发到线程池，其它连接断开事件，心跳等消息，直接在 IO 线程上执行 | Worker 线程接收到事件后，只会把请求和响应消息派发到线程池，其他消息由自己执行到底 |
| execution    | 派发请求，只有请求消息派发到线程池，不含响应，响应和其它连接断开事件，心跳等消息，直接在 IO 线程上执行 | Worker 线程接收到事件后，只会把请求消息派发到线程池，其他消息由自己执行到底 |
| connection   | 在 IO 线程上执行连接与断开事件，将连接断开事件放入队列，有序逐个执行，其它消息派发到线程池 | Worker 线程接收到事件后，只会把连接和断开消息加入队列自己处理，其他消息则会派发到线程池 |

#### 策略实现原理

![1636383507769](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1636383507769.png)

1. 上图是 Dubbo Server 端的请求派发流程，在 Dispatcher 节点，接收到 Client 请求后，根据不同的策略，派发到 ThreadPool 业务线程池中。

```java
public class NettyTransporter implements Transporter {
    @Override
    public Server bind(URL url, ChannelHandler listener) throws RemotingException {
        // 0. 打开Netty服务器
        return new NettyServer(url, listener);
    }
}

public class NettyServer extends AbstractServer implements RemotingServer {
    // 1. 构造NettyServer
    public NettyServer(URL url, ChannelHandler handler) throws RemotingException {
        super(ExecutorUtil.setThreadName(url, SERVER_THREAD_POOL_NAME), 
              // 2. 使用请求派发策略包装 ChannelHandler
              ChannelHandlers.wrap(handler, url));
    }
}

public class ChannelHandlers {
    
    private static ChannelHandlers INSTANCE = new ChannelHandlers();

    protected ChannelHandlers() {
    }
    
    public static ChannelHandler wrap(ChannelHandler handler, URL url) {
        // 3. 获取ChannelHandlers单例，并根据请求派发策略包装ChannelHandler
        return ChannelHandlers.getInstance().wrapInternal(handler, url);
    }
    
    protected static ChannelHandlers getInstance() {
        return INSTANCE;
    }
    
    protected ChannelHandler wrapInternal(ChannelHandler handler, URL url) {
        return new MultiMessageHandler(new HeartbeatHandler(
             	// 4. SPI扩展请求派发策略
                ExtensionLoader.getExtensionLoader(Dispatcher.class)
                .getAdaptiveExtension()
                .dispatch(handler, url)));
    }
}
```

##### all | 全部派发

```java
public class AllDispatcher implements Dispatcher {
    @Override
    public ChannelHandler dispatch(ChannelHandler handler, URL url) {
        // 0. 构建AllChannelHandler对ChannelHandler事件进行派发
        return new AllChannelHandler(handler, url);
    }
}

public class AllChannelHandler extends WrappedChannelHandler {
    @Override
    public void connected(Channel channel) throws RemotingException {
        ExecutorService executor = getExecutorService();
        try {
            // 1. 连接事件，派发到业务线程池
            executor.execute(new ChannelEventRunnable(channel, handler, ChannelState.CONNECTED));
        } catch (Throwable t) {
            ...
        }
    }
    
    @Override
    public void disconnected(Channel channel) throws RemotingException {
        ExecutorService executor = getExecutorService();
        try {
            // 2. 断开连接事件，派发到业务线程池
            executor.execute(new ChannelEventRunnable(channel, handler, ChannelState.DISCONNECTED));
        } catch (Throwable t) {
            ...
        }
    }
    
    @Override
    public void received(Channel channel, Object message) throws RemotingException {
        ExecutorService executor = getPreferredExecutorService(message);
        try {
            // 3. 接收事件，派发到业务线程池
            executor.execute(new ChannelEventRunnable(channel, handler, ChannelState.RECEIVED, message));
        } catch (Throwable t) {
        	...
        }
    }
    
    @Override
    public void caught(Channel channel, Throwable exception) throws RemotingException {
        ExecutorService executor = getExecutorService();
        try {
            // 4. 异常事件，派发到业务线程池
            executor.execute(new ChannelEventRunnable(channel, handler, ChannelState.CAUGHT, exception));
        } catch (Throwable t) {
            ...
        }
    }
}
```

##### direct | 全不派发

```java
public class DirectDispatcher implements Dispatcher {
    @Override
    public ChannelHandler dispatch(ChannelHandler handler, URL url) {
        // 0. 构建DirectChannelHandler对ChannelHandler事件进行派发
        return new DirectChannelHandler(handler, url);
    }
}

public class DirectChannelHandler extends WrappedChannelHandler {
    @Override
    public void received(Channel channel, Object message) throws RemotingException {
        ExecutorService executor = getPreferredExecutorService(message);
        if (executor instanceof ThreadlessExecutor) {
            try {
                // 1. 接收事件，使用 ThreadlessExecutor，将回调直接委托给发起调用的线程。
                executor.execute(new ChannelEventRunnable(channel, handler, ChannelState.RECEIVED, message));
            } catch (Throwable t) {
                ...
            }
        } 
        // 2. 否则，判断后交由ChannelHandler IO线程去执行
        else {
            handler.received(channel, message);
        }
    }
}

public class WrappedChannelHandler implements ChannelHandlerDelegate {
    
    // 3. 连接事件，交由ChannelHandler IO线程去执行
    @Override
    public void connected(Channel channel) throws RemotingException {
        handler.connected(channel);
    }
    
    // 4. 断开连接事件，交由ChannelHandler IO线程去执行
    @Override
    public void disconnected(Channel channel) throws RemotingException {
        handler.disconnected(channel);
    }
    
    // 5. 接收事件，交由ChannelHandler IO线程去执行，对于DirectChannelHandler不会执行，因为它重写了该方法
    @Override
    public void received(Channel channel, Object message) throws RemotingException {
        handler.received(channel, message);
    }
    
    // 6. 发送事件，交由ChannelHandler IO线程去执行
    @Override
    public void sent(Channel channel, Object message) throws RemotingException {
        handler.sent(channel, message);
    }
    
    // 7. 异常事件，交由ChannelHandler IO线程去执行
    @Override
    public void caught(Channel channel, Throwable exception) throws RemotingException {
        handler.caught(channel, exception);
    }
}
```

##### message | 只派发请求和响应

```java
public class MessageOnlyDispatcher implements Dispatcher {
    @Override
    public ChannelHandler dispatch(ChannelHandler handler, URL url) {
        // 0. 构建MessageOnlyChannelHandler对ChannelHandler事件进行派发
        return new MessageOnlyChannelHandler(handler, url);
    }
}

public class MessageOnlyChannelHandler extends WrappedChannelHandler {
   @Override
    public void received(Channel channel, Object message) throws RemotingException {
        ExecutorService executor = getPreferredExecutorService(message);
        try {
            // 1. 接收事件，派发到业务线程池，其余地均交由ChannelHandler IO线程去执行
            executor.execute(new ChannelEventRunnable(channel, handler, ChannelState.RECEIVED, message));
        } catch (Throwable t) {
            ...
        }
    }
}
```

##### execution | 只派发请求

```java
public class ExecutionDispatcher implements Dispatcher {
    @Override
    public ChannelHandler dispatch(ChannelHandler handler, URL url) {
        // 0. 构建ExecutionChannelHandler对ChannelHandler事件进行派发
        return new ExecutionChannelHandler(handler, url);
    }
}

public class ExecutionChannelHandler extends WrappedChannelHandler {
    @Override
    public void received(Channel channel, Object message) throws RemotingException {
        ExecutorService executor = getPreferredExecutorService(message);

        // 1.只有请求接收事件，才会派发到业务线程池
        if (message instanceof Request) {
            try {
                executor.execute(new ChannelEventRunnable(channel, handler, ChannelState.RECEIVED, message));
            } catch (Throwable t) {
                ...
            }
        } 
        // 2. 其余接收事件，使用 ThreadlessExecutor，将回调直接委托给发起调用的线程
        else if (executor instanceof ThreadlessExecutor) {
            executor.execute(new ChannelEventRunnable(channel, handler, ChannelState.RECEIVED, message));
        } 
        // 3. 否则，判断后交由ChannelHandler IO线程去执行，与DirectChannelHandler类似
        else {
            handler.received(channel, message);
        }
    }
}
```

##### connection | 串行处理连接和断开事件

```java
public class ConnectionOrderedDispatcher implements Dispatcher {
    @Override
    public ChannelHandler dispatch(ChannelHandler handler, URL url) {
        // 0. 构建ConnectionOrderedChannelHandler对ChannelHandler事件进行派发
        return new ConnectionOrderedChannelHandler(handler, url);
    }
}

public class ConnectionOrderedChannelHandler extends WrappedChannelHandler {
    
    protected final ThreadPoolExecutor connectionExecutor;
    
    public ConnectionOrderedChannelHandler(ChannelHandler handler, URL url) {
        ...
        // 1. 构造一个只有1个线程的无界阻塞队列线程池
        connectionExecutor = new ThreadPoolExecutor(1, 1,
                0L, TimeUnit.MILLISECONDS,
                new LinkedBlockingQueue<Runnable>(url.getPositiveParameter(CONNECT_QUEUE_CAPACITY, Integer.MAX_VALUE)),
                new NamedThreadFactory(threadName, true),
                new AbortPolicyWithReport(threadName, url)
        );  
    }
    
    @Override
    public void connected(Channel channel) throws RemotingException {
        try {
            ...
            // 2. 连接事件，派发到队列中，串行执行
            connectionExecutor.execute(new ChannelEventRunnable(channel, handler, ChannelState.CONNECTED));
        } catch (Throwable t) {
            ...
        }
    }
    
    @Override
    public void disconnected(Channel channel) throws RemotingException {
        try {
            ...
            // 3. 断开连接事件，派发到队列中，串行执行
            connectionExecutor.execute(new ChannelEventRunnable(channel, handler, ChannelState.DISCONNECTED));
        } catch (Throwable t) {
            ...
        }
    }
    
    @Override
    public void received(Channel channel, Object message) throws RemotingException {
        ExecutorService executor = getPreferredExecutorService(message);
        try {
            // 4. 接收事件，派发到业务线程池执行
            executor.execute(new ChannelEventRunnable(channel, handler, ChannelState.RECEIVED, message));
        } catch (Throwable t) {
            ...
        }
    }
    
    @Override
    public void caught(Channel channel, Throwable exception) throws RemotingException {
        ExecutorService executor = getExecutorService();
        try {
            // 5. 异常事件，派发到业务线程池执行
            executor.execute(new ChannelEventRunnable(channel, handler, ChannelState.CAUGHT, exception));
        } catch (Throwable t) {
            ...
        }
    }
}
```

### 3.0. Dubbo 线程池类型？

#### 线程池类型

![1637247042637](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1637247042637.png)

以默认的 all 派发策略，以及 Netty4 同步调用为例：

1. 客户端主线程，发出⼀个请求后获得 future 实例，在执⾏ get（）时，进⾏阻塞等待。
2. 服务端使⽤ worker 线程（Netty 通信模型），接收到请求后，会将请求提交到 Dubbo#Server 业务线程池中进⾏处理。
3. Dubbo#Server 业务线程，处理完成之后，将相应结果返回给客户端的 worker 线程池（Netty 通信模型）。
4. 最后，客户端的 worker 线程，将响应结果提交到 Dubbo#Client 业务线程池进⾏处理。
5. Dubbo#Client 业务线程，把响应结果填充到之前的 future 实例中，然后唤醒等待的客户端主线程。
6. 最后客户端主线程获取到结果，然后返回给客户端，完成一次 RPC 调用。

| 线程池类型 | 特点                                                         |
| ---------- | ------------------------------------------------------------ |
| fixed      | 默认类型，固定大小线程池，启动时建立线程，不关闭，一直持有   |
| cached     | 缓存线程池，空闲一分钟自动删除，需要时重建                   |
| limited    | 可伸缩线程池，但池中的线程数只会增长不会收缩，只增长不收缩的目的是为了避免收缩时突然来了大流量引起的性能问题 |
| eager      | 正常类型的线程池，即优先创建 `Worker` 线程池，在任务数量大于 `corePoolSize` 、小于 `maximumPoolSize` 时，优先创建 `Worker` 来处理任务。当任务数量大于 `maximumPoolSize` 时，将任务放入阻塞队列中。阻塞队列充满时抛出 `RejectedExecutionException`，相比于`cached` 缓存线程池，在任务数量超过 `maximumPoolSize` 时，不会抛出异常，而是将任务放入阻塞队列 |

#### 线程池实现原理

```java
public class AllChannelHandler extends WrappedChannelHandler {
    @Override
    public void received(Channel channel, Object message) throws RemotingException {
        // 0.1. 获取线程池
        ExecutorService executor = getPreferredExecutorService(message);
        try {
            // 0.2. 接收事件，派发到业务线程池
            executor.execute(new ChannelEventRunnable(channel, handler, ChannelState.RECEIVED, message));
        } catch (Throwable t) {
        	...
        }
    }
}

public class WrappedChannelHandler implements ChannelHandlerDelegate {
    public ExecutorService getPreferredExecutorService(Object msg) {
        ...
        // 1. 获取Server端和Client端共享的业务线程池
        return getSharedExecutorService();
    }
    
    public ExecutorService getSharedExecutorService() {
        // 2. 获取线程池仓库DefaultExecutorRepository
        ExecutorRepository executorRepository =
  ExtensionLoader.getExtensionLoader(ExecutorRepository.class).getDefaultExtension();
        ExecutorService executor = executorRepository.getExecutor(url);
        if (executor == null) 
            // 3. SPI构造线程池
            executor = executorRepository.createExecutorIfAbsent(url);
        }
        return executor;
    }
}

public class DefaultExecutorRepository implements ExecutorRepository {
    public synchronized ExecutorService createExecutorIfAbsent(URL url) {
        ...
        // 4. SPI构造线程池
        executor = createExecutor(url);
        ...
    }
    
    private ExecutorService createExecutor(URL url) {
        // 5. SPI构造线程池，默认为fixed类型
        return (ExecutorService) ExtensionLoader.getExtensionLoader(ThreadPool.class).getAdaptiveExtension().getExecutor(url);
    }
}
```

##### fixed | 固定大小型

```java
public class FixedThreadPool implements ThreadPool {
    @Override
    public Executor getExecutor(URL url) {
        // 1. 读取threadname配置参数，默认为Dubbo
        String name = url.getParameter(THREAD_NAME_KEY, DEFAULT_THREAD_NAME);
        // 2. 读取threads配置参数，默认为200
        int threads = url.getParameter(THREADS_KEY, DEFAULT_THREADS);
        // 3. 读取queues配置参数，默认为0
        int queues = url.getParameter(QUEUES_KEY, DEFAULT_QUEUES);
        // 4. 默认构造200个固定线程、同步阻塞队列的线程池
        return new ThreadPoolExecutor(threads, threads, 0, TimeUnit.MILLISECONDS,
                queues == 0 ? new SynchronousQueue<Runnable>() :
                        (queues < 0 ? new LinkedBlockingQueue<Runnable>()
                                : new LinkedBlockingQueue<Runnable>(queues)),
                new NamedInternalThreadFactory(name, true), new AbortPolicyWithReport(name, url));
    }
}
```

##### cached | 缓存型

```java
public class CachedThreadPool implements ThreadPool {
    @Override
    public Executor getExecutor(URL url) {
        // 1. 读取threadname配置参数，默认为Dubbo
        String name = url.getParameter(THREAD_NAME_KEY, DEFAULT_THREAD_NAME);
        // 2. 读取corethreads配置参数，默认为0
        int cores = url.getParameter(CORE_THREADS_KEY, DEFAULT_CORE_THREADS);
        // 3. 读取threads配置参数，默认为Integer.MAX_VALUE，表示无界
        int threads = url.getParameter(THREADS_KEY, Integer.MAX_VALUE);
        // 4. 读取queues配置参数，默认为0
        int queues = url.getParameter(QUEUES_KEY, DEFAULT_QUEUES);
        // 5. 读取alive配置参数，默认为60 * 1000，表示60s最大空闲时间，即空闲时最大缓存时间为60s
        int alive = url.getParameter(ALIVE_KEY, DEFAULT_ALIVE);
        // 6. 默认构造核心数为0，最大线程数为Integer.MAX_VALUE，60s最大空闲时间的同步阻塞队列的线程池
        return new ThreadPoolExecutor(cores, threads, alive, TimeUnit.MILLISECONDS,
                queues == 0 ? new SynchronousQueue<Runnable>() :
                        (queues < 0 ? new LinkedBlockingQueue<Runnable>()
                                : new LinkedBlockingQueue<Runnable>(queues)),
                new NamedInternalThreadFactory(name, true), new AbortPolicyWithReport(name, url));
    }
}
```

##### limited | 可伸缩型（线程数只能增加不能减少）

```java
public class LimitedThreadPool implements ThreadPool {
    @Override
    public Executor getExecutor(URL url) {
        // 1. 读取threadname配置参数，默认为Dubbo
        String name = url.getParameter(THREAD_NAME_KEY, DEFAULT_THREAD_NAME);
        // 2. 读取corethreads配置参数，默认为0
        int cores = url.getParameter(CORE_THREADS_KEY, DEFAULT_CORE_THREADS);
        // 3. 读取threads配置参数，默认为200
        int threads = url.getParameter(THREADS_KEY, DEFAULT_THREADS);
        // 4. 读取queues配置参数，默认为0
        int queues = url.getParameter(QUEUES_KEY, DEFAULT_QUEUES);
        // 5. 默认构造核心数为0，最大线程数为200，用不过期的，线程数只能增加不能减少的线程池
        return new ThreadPoolExecutor(cores, threads, Long.MAX_VALUE, TimeUnit.MILLISECONDS,
                queues == 0 ? new SynchronousQueue<Runnable>() :
                        (queues < 0 ? new LinkedBlockingQueue<Runnable>()
                                : new LinkedBlockingQueue<Runnable>(queues)),
                new NamedInternalThreadFactory(name, true), new AbortPolicyWithReport(name, url));
    }
}
```

##### eager | 正常型

```java
public class EagerThreadPool implements ThreadPool {
    @Override
    public Executor getExecutor(URL url) {
        // 1. 读取threadname配置参数，默认为Dubbo
        String name = url.getParameter(THREAD_NAME_KEY, DEFAULT_THREAD_NAME);
        // 2. 读取corethreads配置参数，默认为0
        int cores = url.getParameter(CORE_THREADS_KEY, DEFAULT_CORE_THREADS);
        // 3. 读取threads配置参数，默认为Integer.MAX_VALUE，表示无界
        int threads = url.getParameter(THREADS_KEY, Integer.MAX_VALUE);
        // 4. 读取queues配置参数，默认为0
        int queues = url.getParameter(QUEUES_KEY, DEFAULT_QUEUES);
        // 5. 读取alive配置参数，默认为60 * 1000，表示60s最大空闲时间，即空闲时最大缓存时间为60s
        int alive = url.getParameter(ALIVE_KEY, DEFAULT_ALIVE);
        // 6. 默认构造1容量的TaskQueue
        TaskQueue<Runnable> taskQueue = new TaskQueue<Runnable>(queues <= 0 ? 1 : queues);
        // 8. 根据参数正常构造EagerThreadPoolExecutor
        EagerThreadPoolExecutor executor = new EagerThreadPoolExecutor(cores,
                threads,
                alive,
                TimeUnit.MILLISECONDS,
                taskQueue,
                new NamedInternalThreadFactory(name, true),
                new AbortPolicyWithReport(name, url));
        
        // 10. 最后还回写线程池实例到阻塞队列中（与正常线程池不同的地方，用于在添加任务时，获取线程池的线程数量，然后判断是否阻塞）
        taskQueue.setExecutor(executor);
        return executor;
    }
}

public class TaskQueue<R extends Runnable> extends LinkedBlockingQueue<Runnable> {
    public TaskQueue(int capacity) {
        // 7. TaskQueue等同于LinkedBlockingQueue
        super(capacity);
    }
}

public class EagerThreadPoolExecutor extends ThreadPoolExecutor {
    public EagerThreadPoolExecutor(int corePoolSize,
                                   int maximumPoolSize,
                                   long keepAliveTime,
                                   TimeUnit unit, TaskQueue<Runnable> workQueue,
                                   ThreadFactory threadFactory,
                                   RejectedExecutionHandler handler) {
        // 9. EagerThreadPoolExecutor等同于ThreadPoolExecutor
        super(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue, threadFactory, handler);
    }
}
```

### 3.1. Dubbo 限流原理？

```java
// 1. 使用ProtocolFilterWrapper#buildInvokerChain构造过滤器链，在服务端的export方法和消费端refer方法都会用到
public class ProtocolFilterWrapper implements Protocol {
    private static <T> Invoker<T> buildInvokerChain(final Invoker<T> invoker, String key, String group) {
        ...
        if (!filters.isEmpty()) {
            for (int i = filters.size() - 1; i >= 0; i--) {
                final Filter filter = filters.get(i);
                final Invoker<T> next = last;
                last = new Invoker<T>() {
                    ...
                    @Override
                    public Result invoke(Invocation invocation) throws RpcException {
                        Result asyncResult;
                        try {
                            // 2. 根据服务端和消费端，构造不同的过滤器
                            asyncResult = filter.invoke(next, invocation);
                        } catch (Exception e) {
                            ...
                        }
                        return asyncResult;
                    }
                    ...
                };
            }
        }
        ...
    }
}

// 3. 限流统计用到的核心类（服务端和消费端限流都会用到）
public class RpcStatus {
    // 4. 开始统计，如果方法活跃数，即并发调用数，等于了最大值则返回false，表示进行了限流
    public static boolean beginCount(URL url, String methodName, int max) {
        max = (max <= 0) ? Integer.MAX_VALUE : max;
        // 5、 取出应用级别、方法级别的统计信息
        RpcStatus appStatus = getStatus(url);
        RpcStatus methodStatus = getStatus(url, methodName);
        // 6. 如果方法活跃数，即并发调用数，等于了最大值则返回false，表示进行了限流
        if (methodStatus.active.get() == Integer.MAX_VALUE) {
            return false;
        }
        // 7. 否则进行自旋+1，如果方法活跃数，即并发调用数，等于了最大值则返回false，表示进行了限流
        for (int i; ; ) {
            i = methodStatus.active.get();
            if (i + 1 > max) {
                return false;
            }
            if (methodStatus.active.compareAndSet(i, i + 1)) {
                break;
            }
        }
        // 8. 最后不需要限流的，则应用级别的自旋+1，然后返回true
        appStatus.active.incrementAndGet();
        return true;
    }

    // 9. 结束统计，活跃数-1，以及累加调用总数、失败调用总数、成功和失败调用花费的时间
    public static void endCount(URL url, String methodName, long elapsed, boolean succeeded) {
        // 10. 结束应用级别单次调用的统计
        endCount(getStatus(url), elapsed, succeeded);
        // 11. 结束方法级别单词调用的统计
        endCount(getStatus(url, methodName), elapsed, succeeded);
    }

    private static void endCount(RpcStatus status, long elapsed, boolean succeeded) {
        // 12. 活跃数-1
        status.active.decrementAndGet();
        // 13. 调用总数+1
        status.total.incrementAndGet();
        // 14. 累加单次调用花费时间
        status.totalElapsed.addAndGet(elapsed);
        if (status.maxElapsed.get() < elapsed) {
            status.maxElapsed.set(elapsed);
        }
        // 15. 如果调用成功，则累加单次成功调用花费时间
        if (succeeded) {
            if (status.succeededMaxElapsed.get() < elapsed) {
                status.succeededMaxElapsed.set(elapsed);
            }
        } 
        // 16. 如果调用失败，则累加调用失败次数，以及累加单次失败调用花费的时间
        else {
            status.failed.incrementAndGet();
            status.failedElapsed.addAndGet(elapsed);
            if (status.failedMaxElapsed.get() < elapsed) {
                status.failedMaxElapsed.set(elapsed);
            }
        }
    }
}
```

#### 服务端限流

1. ExecuteLimitFilter 继承了 Filter.Listener，本质上也是一个监听器，提供 `onResponse()` 监听响应成功的方法，`onError` 监听响应异常的方法。
2. ExecuteLimitFilter 的 `invoke()` 先调⽤ `RpcStatus#beginCount()` 来判断是否可以通过，不通过则**抛出RpcException**。
3. 通过则记录开始执⾏的时间，则记录 `execute_limit_filter_start_time` 值，然后执⾏ `invoker.invoke()` 。
4. 执⾏结束时会回调 Listener 的`onResponse()` 或 `onError()` ，而它们都会调⽤ `RpcStatus#endCount()`，该⽅法会通过 `getElapsed()` ，取出 `execute_limit_filter_start_time` 值，以计算执⾏耗时。

```java
// ServiceBean实现了ApplicationListener，在ContextRefreshedEvent发布，即Spring上下文准备完毕时，会回调onApplicationEvent方法，拉起ServiceBean
public class ServiceBean<T> extends ServiceConfig<T> implements InitializingBean, DisposableBean, ApplicationContextAware, ApplicationListener<ContextRefreshedEvent>, BeanNameAware, ApplicationEventPublisherAware {
   	private void doExportUrlsFor1Protocol(ProtocolConfig protocolConfig, List<URL> registryURLs) {
        ...
        // 0.1. 如果不是只暴露远程服务（一般不配），则暴露本地服务
        if (!SCOPE_REMOTE.equalsIgnoreCase(scope)) {
            exportLocal(url);
        }
        ... 
        // 0.2. 如果不是只暴露远程服务（一般不配），则暴露本地服务
        if (!SCOPE_LOCAL.equalsIgnoreCase(scope)) {
            // 0.3. 这里是Server端，获取interfaceClass的javasist动态代理Wrapper包装类Invoker
            Invoker<?> invoker = PROXY_FACTORY.getInvoker(ref, (Class) interfaceClass, registryURL.addParameterAndEncoded(EXPORT_KEY, url.toFullString()));
            DelegateProviderMetaDataInvoker wrapperInvoker = new DelegateProviderMetaDataInvoker(invoker, this);
            
            // 0.4. 通过RegistryProtocol#export暴露远程服务，获取对应的exporter
            Exporter<?> exporter = protocol.export(wrapperInvoker);
            ...
        }
        ...
    }
}    

public class RegistryProtocol implements Protocol {
    @Override
    public <T> Exporter<T> export(final Invoker<T> originInvoker) throws RpcException 	  {
       	// 0.5. 打开Netty服务器，利用Netty NIO特性，把同步操作转换为I/O监听的异步操作，以打开本地Dubbo服务，最后把DubboExporter缓存到exportes中
        final ExporterChangeableWrapper<T> exporter = doLocalExport(originInvoker, providerUrl);
        // ... 连接、订阅、监听ZK
    }
    
    private <T> ExporterChangeableWrapper<T> doLocalExport(final Invoker<T> originInvoker, URL providerUrl) {
        String key = getCacheKey(originInvoker);
        return (ExporterChangeableWrapper<T>) bounds.computeIfAbsent(key, s -> {
            Invoker<?> invokerDelegate = new InvokerDelegate<>(originInvoker, providerUrl);
            // 0.6. 通过ProtocolFilterWrapper过滤扩展链装饰->ProtocolListenerWrapper监听扩展链装饰->DubboProtocol#export，暴露本地Dubbo服务，打开Netty服务器
            return new ExporterChangeableWrapper<>((Exporter<T>) protocol.export(invokerDelegate), originInvoker);
        });
    }
}

public class ProtocolFilterWrapper implements Protocol {
    @Override
    public <T> Exporter<T> export(Invoker<T> invoker) throws RpcException {
        if (REGISTRY_PROTOCOL.equals(invoker.getUrl().getProtocol())) {
            return protocol.export(invoker);
        }
        // 0.7. ProtocolFilterWrapper过滤扩展链装饰（8个过滤器）
        return protocol.export(buildInvokerChain(invoker, SERVICE_FILTER_KEY, CommonConstants.PROVIDER));
    }
}

// 1. 服务端限流，则构造ExecuteLimitFilter
@Activate(group = CommonConstants.PROVIDER, value = EXECUTES_KEY)
public class ExecuteLimitFilter implements Filter, Filter.Listener {
    
    private static final String EXECUTE_LIMIT_FILTER_START_TIME = "execute_limit_filter_start_time";
    
    @Override
    public Result invoke(Invoker<?> invoker, Invocation invocation) throws RpcException {
        // 2. 获取invoker的url和响应的方法名称
        URL url = invoker.getUrl();
        String methodName = invocation.getMethodName();
        // 3. 获取参数配置的executes，默认为0，表示服务提供者的，每个服务的，每个方法的，最大可并行执行请求数
        int max = url.getMethodParameter(methodName, EXECUTES_KEY, 0);
        // 4. 开始统计，如果方法活跃数，即并发调用数，等于了最大值则返回false，表示进行了限流
        if (!RpcStatus.beginCount(url, methodName, max)) {
            // 4.1. 如果进行了限流，则抛出异常
            throw new RpcException(...)
        }
        // 5. 远程调用前，先记录当前时间execute_limit_filter_start_time
        invocation.put(EXECUTE_LIMIT_FILTER_START_TIME, System.currentTimeMillis());
        try {
            // 6. 进行本地实现类的动态代理类的方法调用
            return invoker.invoke(invocation);
        } catch (Throwable t) {
            if (t instanceof RuntimeException) {
                throw (RuntimeException) t;
            } else {
                throw new RpcException(...);
            }
        }
    }

    // 7. 根据记录的execute_limit_filter_start_time，算出本地方法调用总共花费的时间
    private long getElapsed(Invocation invocation) {
        Object beginTime = invocation.get(EXECUTE_LIMIT_FILTER_START_TIME);
        return beginTime != null ? System.currentTimeMillis() - (Long) beginTime : 0;
    }
                                   
    @Override
    public void onResponse(Result appResponse, Invoker<?> invoker, Invocation invocation) {
        // 8. 结束统计，活跃数-1，以及累加调用总数、失败调用总数、成功和失败调用花费的时间
        RpcStatus.endCount(invoker.getUrl(), invocation.getMethodName(), getElapsed(invocation), true);
    }

    @Override
    public void onError(Throwable t, Invoker<?> invoker, Invocation invocation) {
        ...
  		// 9. 结束统计，活跃数-1，以及累加调用总数、失败调用总数、成功和失败调用花费的时间
        RpcStatus.endCount(invoker.getUrl(), invocation.getMethodName(), getElapsed(invocation), false);
    }
}
```

#### 消费端限流

1. ActiveLimitFilter 继承了 Filter.Listener，本质上也是一个监听器，提供 `onResponse()` 监听响应成功的方法，`onError` 监听响应异常的方法。
2. ActiveLimitFilter 的 `invoke()` 先调⽤ `RpcStatus#beginCount()` 来判断是否可以通过，不通过则**阻塞当前线程**。
3. 通过则记录开始执⾏的时间，则记录 `activelimit_filter_start_time` 值，然后执⾏ `invoker.invoke()` 。
4. 执⾏结束时会回调 Listener 的`onResponse()` 或 `onError()` ，而它们都会调⽤ `RpcStatus#endCount()`，该⽅法会通过 `getElapsed()` ，取出 `activelimit_filter_start_time` 值，以计算执⾏耗时，并**唤醒单例rpcStatus阻塞的所有线程**。

```java
// ReferenceBeans实现了FactoryBean，在注入Dubbo#ref接口接口时，Spring会回调getObject方法，拉起ReferenceBean
public class ReferenceBean<T> extends ReferenceConfig<T> implements FactoryBean, ApplicationContextAware, InitializingBean, DisposableBean {

    private T createProxy(Map<String, String> map) {
        // 0. 如果为本地暴露服务，则生成本地执行的Invoker
    	if (shouldJvmRefer(map)) {
            URL url = new URL(LOCAL_PROTOCOL, LOCALHOST_VALUE, 0, interfaceClass.getName()).addParameters(map);
            invoker = REF_PROTOCOL.refer(interfaceClass, url);
            ...
        } else {
            for (URL url : urls) {
                // 0.1 如果不是本地暴露服务，则生成远程服务引用的Invoker
                invokers.add(REF_PROTOCOL.refer(interfaceClass, url));
            }
            ...
    	}
        ...
    }
}

public class ProtocolFilterWrapper implements Protocol {
    @Override
    public <T> Invoker<T> refer(Class<T> type, URL url) throws RpcException {
        if (REGISTRY_PROTOCOL.equals(url.getProtocol())) {
            return protocol.refer(type, url);
        }
        // 0.2 ProtocolFilterWrapper过滤扩展链装饰（8个过滤器）
        return buildInvokerChain(protocol.refer(type, url), REFERENCE_FILTER_KEY, CommonConstants.CONSUMER);
    }
}

// 1. 消费端限流，则构造ActiveLimitFilter
@Activate(group = CONSUMER, value = ACTIVES_KEY)
public class ActiveLimitFilter implements Filter, Filter.Listener {
    
    private static final String ACTIVELIMIT_FILTER_START_TIME = "activelimit_filter_start_time";
    
    @Override
    public Result invoke(Invoker<?> invoker, Invocation invocation) throws RpcException {
        // 2. 获取url和方法名称
        URL url = invoker.getUrl();
        String methodName = invocation.getMethodName();
        // 3. 获取参数配置的actives，默认为0，表示每个服务消费者的，每个服务的，每个方法的，最大并发调用数
        int max = invoker.getUrl().getMethodParameter(methodName, ACTIVES_KEY, 0);
        // 4. 从缓存中获取方法级别的单例rpcStatus（多个线程共享，用于synchronized）
        final RpcStatus rpcStatus = RpcStatus.getStatus(invoker.getUrl(), invocation.getMethodName());
        // 5. 开始统计，如果方法活跃数，即并发调用数，等于了最大值则返回false，表示进行了限流
        if (!RpcStatus.beginCount(url, methodName, max)) {
            // 6. 获取参数配置的timeout，默认为0，表示超时时间
            long timeout = invoker.getUrl().getMethodParameter(invocation.getMethodName(), TIMEOUT_KEY, 0);
            long start = System.currentTimeMillis();
            long remain = timeout;
            synchronized (rpcStatus) {
                while (!RpcStatus.beginCount(url, methodName, max)) {
                    try {
                        // 7. 如果进行了限流，则阻塞当前线程，直到别的线程调用响应或者异常后，才会被唤醒
                        rpcStatus.wait(remain);
                    } catch (InterruptedException e) {
                        // ignore
                    }
                    // 8. 线程被唤醒后，则计算剩余超时时间，如果为负数，则抛出异常，代表调用超时
                    long elapsed = System.currentTimeMillis() - start;
                    remain = timeout - elapsed;
                    if (remain <= 0) {
                        throw new RpcException(...);
                    }
                }
            }
        }
        // 9. 如果没被进行限流，或者仍没有超时，则先记录activelimit_filter_start_time
        invocation.put(ACTIVELIMIT_FILTER_START_TIME, System.currentTimeMillis());
		// 10. 进行远程调用
        return invoker.invoke(invocation);
    }

    // 11. 根据记录的activelimit_filter_start_time，算出远程调用总共花费的时间
    private long getElapsed(Invocation invocation) {
        Object beginTime = invocation.get(ACTIVELIMIT_FILTER_START_TIME);
        return beginTime != null ? System.currentTimeMillis() - (Long) beginTime : 0;
    }
    
    @Override
    public void onResponse(Result appResponse, Invoker<?> invoker, Invocation invocation) {
        ...
		// 12. 结束统计，活跃数-1，以及累加调用总数、失败调用总数、成功和失败调用花费的时间
        RpcStatus.endCount(url, methodName, getElapsed(invocation), true);
        // 13. 当前线程响应成功，则唤醒单例rpcStatus阻塞的所有线程
        notifyFinish(RpcStatus.getStatus(url, methodName), max);
    }

    @Override
    public void onError(Throwable t, Invoker<?> invoker, Invocation invocation) {
        ...
		// 15. 结束统计，活跃数-1，以及累加调用总数、失败调用总数、成功和失败调用花费的时间
        RpcStatus.endCount(url, methodName, getElapsed(invocation), false);
        // 16. 当前线程响应失败，则唤醒单例rpcStatus阻塞的所有线程 
        notifyFinish(RpcStatus.getStatus(url, methodName), max);
    }
    
    // 14. 唤醒单例rpcStatus阻塞的所有线程
    private void notifyFinish(final RpcStatus rpcStatus, int max) {
        if (max > 0) {
            synchronized (rpcStatus) {
                rpcStatus.notifyAll();
            }
        }
    }
}
```

### 3.2. Dubbo 降级原理？

#### 容错 | 失败处理

1. 当系统出现非业务异常（不可知、不可预测的异常），比如并发数太高导致的服务超时、网络异常等。
2. 为保证核心链路不受此异常影响，可对**该接口**进行降级处理，采用控制台动态配置 `mock=fail:return null`，失败时不对接口进行处理。

#### 屏蔽 | 强制屏蔽

1. 而对于大促、促销、双 11 等一些可预知、可预测的情况下，为保证核心链路不受非核心接口的影响，可以提前对**非核心接口**进行降级处理，采用控制台动态配置 `mock=force:return null` ，强制屏蔽接口。

```java
public class JavassistProxyFactory extends AbstractProxyFactory {
    @Override
    public <T> T getProxy(Invoker<T> invoker, Class<?>[] interfaces) {
        // 0. 生成代理对象，织入InvokerInvocationHandler
        return (T) Proxy.getProxy(interfaces).newInstance(new InvokerInvocationHandler(invoker));
    }
}

public class InvokerInvocationHandler implements InvocationHandler {
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        ...
        // 0.1 动态代理执行
		return invoker.invoke(new RpcInvocation(method, args)).recreate();
    }
}

public class MockClusterInvoker<T> implements Invoker<T> {
    @Override
    public Result invoke(Invocation invocation) throws RpcException {
        Result result = null;
        // 1. 获取参数配置的mock，默认为false
        String value = directory.getUrl().getMethodParameter(invocation.getMethodName(), MOCK_KEY, Boolean.FALSE.toString()).trim();
        // 2. 默认值不进行降级，则直接进行invoker调用
        if (value.length() == 0 || value.equalsIgnoreCase("false")) {
            result = this.invoker.invoke(invocation);
        } 
        else if (value.startsWith("force")) {
            ...
            // 3. 如果有值，且以force开头，说明为屏蔽，则调用doMockInvoke(invocation, null)
            result = doMockInvoke(invocation, null);
        } else {
            try {
                result = this.invoker.invoke(invocation);
            } catch (RpcException e) {
                ...
                // 4. 如果有值，但不以force开头，说明为容错，则异常时才调用doMockInvoke(invocation, e)，其中e是用来记录原始异常信息的，并没有其他作用
                result = doMockInvoke(invocation, e);
            }
        }
        
        // 9. 因此，对于屏蔽来说，则直接返回null；对于容错来说，则先远程调用，在调用异常时才进行降级处理，然后返回null
        return result;
    }
    
    private Result doMockInvoke(Invocation invocation, RpcException e) {
        ...
        try {
            // 5. 调用MockInvoker#invoker方法进行降级
            result = minvoker.invoke(invocation);
        } catch (RpcException me) {
            ...
        } catch (Throwable me) {
            ...
        }
        
        // 8. 降级后返回null
        return result;
    }
}

final public class MockInvoker<T> implements Invoker<T> {
    @Override
    public Result invoke(Invocation invocation) throws RpcException {
        ...
        // 6. 提取参数配置的mock内容（降级的为return null），如果为return开头
        if (mock.startsWith(RETURN_PREFIX)) {
            mock = mock.substring(RETURN_PREFIX.length()).trim();
            try {
                Type[] returnTypes = RpcUtils.getReturnTypes(invocation);
                Object value = parseMockValue(mock, returnTypes);
                /// 7. 则直接返回null
                return AsyncRpcResult.newDefaultAsyncResult(value, invocation);
            } catch (Exception ew) {
                throw new RpcException(...);
            }
        } else if (mock.startsWith(THROW_PREFIX)) {
            ...
        } else {
            ...
        }
    }
}
```

### 3.3. Dubbo 设计模式？

#### 1、责任链模式

1. 责任链模式，在 Dubbo 中发挥的作⽤举⾜轻重，就像是 Dubbo 框架的⻣架。
2. Dubbo 调⽤链组织是⽤责任链模式串连起来的，责任链中的每个节点实现 Filter 接⼝，然后由
   **ProtocolFilterWrapper**，将所有 Filter 串连起来。
3. 通过 Filter 扩展实现了 Dubbo 许多功能，⽐如监控、⽇志、缓存、安全、telnet 以及 RPC 本身都是。
4. 如果把 Dubbo ⽐作⼀列⽕⻋，那么责任链就像是⽕⻋的各⻋厢，每个⻋厢的功能不同，如果需要加⼊新的功能，增加⻋厢就可以了，⾮常容易扩展。

```java
// 1. 使用ProtocolFilterWrapper#buildInvokerChain构造过滤器链，在服务端的export方法和消费端refer方法都会用到
public class ProtocolFilterWrapper implements Protocol {
    private static <T> Invoker<T> buildInvokerChain(final Invoker<T> invoker, String key, String group) {
        ...
        if (!filters.isEmpty()) {
            for (int i = filters.size() - 1; i >= 0; i--) {
                final Filter filter = filters.get(i);
                final Invoker<T> next = last;
                last = new Invoker<T>() {
                    ...
                    @Override
                    public Result invoke(Invocation invocation) throws RpcException {
                        Result asyncResult;
                        try {
                            // 2. 根据服务端和消费端，构造不同的过滤器
                            asyncResult = filter.invoke(next, invocation);
                        } catch (Exception e) {
                            ...
                        }
                        return asyncResult;
                    }
                    ...
                };
            }
        }
        ...
    }
}
```

#### 2、观察者模式

1. Dubbo 使⽤观察者模式最典型的例⼦是 RegistryService。
2. 消费者在初始化的时候回调⽤ subscribe ⽅法，会注册⼀个观察者，如果观察者引⽤的服务地址列表发⽣改变，就会通过 NotifyListener 通知消费者。
3. 此外，Dubbo的 InvokerListener、ExporterListener 也实现了观察者模式，只要实现该接⼝，并注册，就可以通知到 consumer 端调⽤ refer（）、provider 端调⽤ export（） 。

```java
public class RegistryDirectory<T> extends AbstractDirectory<T> implements NotifyListener {
    @Override
    public synchronized void notify(List<URL> urls) {
        ...
        // 1. 通知更新url配置，刷新Invoker列表
        refreshOverrideAndInvoker(providerURLs);
    }
    
    private void refreshOverrideAndInvoker(List<URL> urls) {
        // 2. 更新configurators配置（和服务端一样，省略代码）
        overrideDirectoryUrl();
        // 3. 刷新Invoker列表
        refreshInvoker(urls);
    }
}
```

#### 3、模板方法模式

1. 模板方法模式，指把通用的方法写到抽象父类，然后交由不同子类去实现，从而在不改变算法结构的情况下，重新定义具体的实现。
2. Dubbo 中比如 AbstractProtocol#protocolBindingRefer、AbstractProxyFactory#getProxy、AbstractClusterInvoker#doInvoke、AbstractDirectory#doList、AbstractLoadBalance#doSelect、AbstractInvoker#doInvoke、AbstractProxyInvoker#doInvoke 等，都是使用了模板方法模式，再结合 SPI 动态扩展机制，更改配置策略，就可以实现不同的算法效果，比如负载均衡、集群容错等。

```java
public abstract class AbstractLoadBalance implements LoadBalance {
    @Override
    public <T> Invoker<T> select(List<Invoker<T>> invokers, URL url, Invocation invocation) {
        ...
        // 1. 交由AbstractLoadBalance决定哪个路由规则进行负载均衡
        return doSelect(invokers, url, invocation);
    }
    
    // 2. 模板方法，默认交由RandomLoadBalance进行负载均衡
	protected abstract <T> Invoker<T> doSelect(List<Invoker<T>> invokers, URL url, Invocation invocation);
}
```

#### 4、装饰者模式

1. Dubbo 中⼤量⽤到了修饰器模式。
2. ⽐如 ProtocolFilterWrapper 类是对 Protocol 类的修饰，在 export 和 refer ⽅法中，配合责任链模式，把Filter 组装成责任链，实现对 Protocol 功能的修饰。
3. 其他还有ProtocolListenerWrapper、 ListenerInvokerWrapper、InvokerWrapper等。
4. 不过，修饰器模式是⼀把双刃剑，⼀⽅⾯⽤它可以⽅便地扩展类的功能，⽽且对⽤户⽆感；但另⼀⽅⾯，过多地使⽤修饰器模式不利于理解，因为⼀个类可能经过层层修饰，最终的⾏为已经和原始⾏为偏离较⼤。

```java
private <T> ExporterChangeableWrapper<T> doLocalExport(final Invoker<T> originInvoker, URL providerUrl) {
    String key = getCacheKey(originInvoker);
    return (ExporterChangeableWrapper<T>) bounds.computeIfAbsent(key, s -> {
        Invoker<?> invokerDelegate = new InvokerDelegate<>(originInvoker, providerUrl);
        // 1. 通过ProtocolFilterWrapper过滤扩展链装饰->ProtocolListenerWrapper监听扩展链装饰->DubboProtocol#export，暴露本地Dubbo服务，打开Netty服务器
        return new ExporterChangeableWrapper<>((Exporter<T>) protocol.export(invokerDelegate), originInvoker);
    });
}
```

#### 5、代理模式

1. Dubbo 服务端，使用 Proxy 类来创建本地实现类的动态代理，对调用前做一定的包装，比如 Filter、Listener 等。
2. 而 Dubbo 消费端， 使⽤ Proxy 类来创建远程服务的动态代理，从而屏蔽⽹络通信的细节，使得⽤户在使⽤本地代理的时候，感觉和使⽤本地服务⼀样。

```java
public class JavassistProxyFactory extends AbstractProxyFactory {
    // Dubbo 服务端代理
    @Override
    public <T> Invoker<T> getInvoker(T proxy, Class<T> type, URL url) {
        final Wrapper wrapper = Wrapper.getWrapper(proxy.getClass().getName().indexOf('$') < 0 ? proxy.getClass() : type);
        // 1.获取interfaceClass的javasist动态代理Wrapper包装类
        return new AbstractProxyInvoker<T>(proxy, type, url) {
            @Override
            protected Object doInvoke(T proxy, String methodName,
                                      Class<?>[] parameterTypes,
                                      Object[] arguments) throws Throwable {
                // 2. 动态代理调用实现类方法
                return wrapper.invokeMethod(proxy, methodName, parameterTypes, arguments);
            }
        };
    }
    
    // Dubbo 消费端代理
    @Override
    public <T> T getProxy(Invoker<T> invoker, Class<?>[] interfaces) {
        // 3. 生成代理对象，织入InvokerInvocationHandler
        return (T) Proxy.getProxy(interfaces).newInstance(new InvokerInvocationHandler(invoker));
    }
}
```

#### 6、适配器模式

1. 为了让⽤户根据⾃⼰的需求选择⽇志组件，Dubbo ⾃定义了⾃⼰的 Logger 接⼝，并为常⻅的⽇志组件，比如 jcl、jdk、log4j、slf4j 提供了相应的适配器。
2. 同时利⽤简单⼯⼚模式提供⼀个 LoggerFactory，客户可以创建抽象的 Dubbo ⾃定义Logger，⽽⽆需关⼼实际使⽤的⽇志组件类型。
3. 在 LoggerFactory 初始化时，客户通过设置系统变量的⽅式，来选择⾃⼰所⽤的⽇志组件，提供了很⼤的灵活性。

```java
// 实现类有：JclLoggerAdapter、JdkLoggerAdapter、Log4j2LoggerAdapter、Log4jLoggerAdapter、Slf4jLoggerAdapter
@SPI
public interface LoggerAdapter {
    Logger getLogger(Class<?> key);
    Logger getLogger(String key);
    Level getLevel();
    void setLevel(Level level);
    File getFile();
    void setFile(File file);
}
```

#### 7、工厂方法模式

1. CacheFactory 采⽤⼯⼚⽅法模式实现，它定义 getCache ⽅法，然后定义⼀个 AbstractCacheFactory 抽象类实现 CacheFactory，并将实际创建 cache 的 createCache（）分离出来，并设置为抽象⽅法，这样具体 cache 的创建⼯作，就留给具体的⼦类去完成。

```java
@SPI("lru")
public interface CacheFactory {
    @Adaptive("cache")
    Cache getCache(URL url, Invocation invocation);
}

public abstract class AbstractCacheFactory implements CacheFactory {
    ...
    // 模板方法，交由实现类去实现：ExpiringCacheFactory、JCacheFactory、LfuCacheFactory、LruCacheFactory、ThreadLocalCacheFactory
    protected abstract Cache createCache(URL url);
}
```

#### 8、抽象工厂模式

> 工厂方法模式，针对的是多个产品系列结构（同一个抽象产品角色, 同一个产品族）。
> 抽象工厂模式，针对的是多个产品族结构（多个抽象产品角色,多个产品族），一个产品族内有多个产品系列（同一个抽象产品角色, 同一个产品）。

1. ProxyFactory 及其⼦类，是Dubbo 中使⽤抽象⼯⼚模式的典型例⼦。
2. ProxyFactory 提供 getProxy（） 和 Invoker（），getProxy（）需要传⼊⼀个 Invoker 对象，针对的是消费端获取接口的动态代理；⽽ getInvoker（）需要传⼊⼀个 proxy 对象，针对的是服务端用于获取本地实现类的代理 invoker，从而创建对应的 exporter。
3. AbstractProxyFactory 实现了 ProxyFactory 接⼝，作为具体实现类的抽象⽗类，提供通用的实现。
4. 然后还定义了 JavassistProxyFactory 和 JdkProxyFactory 两个实现类，分别⽤来⽣产基于 javassist 代理机制和 jdk 代理机制的 Proxy 和 Invoker。

```java
@SPI("javassist")
public interface ProxyFactory {
    @Adaptive({PROXY_KEY})
    <T> T getProxy(Invoker<T> invoker) throws RpcException;
    
    @Adaptive({PROXY_KEY})
    <T> T getProxy(Invoker<T> invoker, boolean generic) throws RpcException;
    
    @Adaptive({PROXY_KEY})
    <T> Invoker<T> getInvoker(T proxy, Class<T> type, URL url) throws RpcException;
}

public abstract class AbstractProxyFactory implements ProxyFactory {
    ...
    // 1. 模板方法，多态调用doInvoke方法，实现类：JavassistProxyFactory和JdkProxyFactory
    protected abstract Object doInvoke(T proxy, String methodName, Class<?>[] parameterTypes, Object[] arguments) throws Throwable;
}
```

#### 9、单例模式

1. 另外，在类创建方式方面，对于一些缓存变量，常常用单例模式实例化（线程安全+单例保证），比如 AbstractProxyFactory#INTERNAL_INTERFACES，ExtensionLoader#EXTENSION_LOADERS、EXTENSION_INSTANCES 等。

```java
public abstract class AbstractProxyFactory implements ProxyFactory {
    // 1. 饿汉式创建缓存单例
    private static final Class<?>[] INTERNAL_INTERFACES = new Class<?>[]{
            EchoService.class, Destroyable.class
    };
}

public class ExtensionLoader<T> {
    ...
    // 2. 饿汉式创建缓存单例
    private static final ConcurrentMap<Class<?>, ExtensionLoader<?>> EXTENSION_LOADERS = new ConcurrentHashMap<>();
    private static final ConcurrentMap<Class<?>, Object> EXTENSION_INSTANCES = new ConcurrentHashMap<>();
    ...
}
```

#### 10、建造者模式

1. 另外，在类创建方式方面，对于一些多参数变量，比如 org.apache.dubbo.common.URL 对象，则可以通过建造者模式构造出来（易于阅读、方便扩展）。

```java
public class RegistryProtocol implements Protocol {
    @Override
    public <T> Invoker<T> refer(Class<T> type, URL url) throws RpcException {
        // 1. 通过URLBuilder来创建URL对象
        url = URLBuilder.from(url)
                .setProtocol(url.getParameter(REGISTRY_KEY, DEFAULT_REGISTRY))
                .removeParameter(REGISTRY_KEY)
                .build();
        Registry registry = registryFactory.getRegistry(url);
        ...
    }
}
```

### 3.4. Dubbo 最佳实践？

#### 1、Provider 多配置接口参数

1. 作为服务提供⽅，自己⽐消费⽅更清楚服务的这些接口参数的取值，比如超时时间、重试次数、负载均衡策略等。
2. 由于配置覆盖策略存在，在 Provider 端配置后，Consumer 端不配置则会使⽤ Provider 端配置，即 Provider 配置可以作为 Consumer 的缺省值。
3. 而如果 Provider 不配置，Consumer 也不直接配置，则会使⽤ Consumer 端的全局设置，这对于 Provider 是不可控的，并且往往是不合理的。
4. 因此，Provider 端尽量多配置、完善这些接口参数，让 Provider 实现者⼀开始就思考 Provider 端的服
   务特点和服务质量等问题。

#### 2、Provider 合理配置性能参数

比如，**threads**（服务线程池⼤⼩）、**executes**（服务提供者并发请求的上限）。

#### 3、服务使用固定端口

使⽤固定端⼝来暴露服务，不要使⽤随机端⼝，这样在注册中⼼推送延迟的情况下，消费端仍然能够通过缓存列表，调⽤到原地址+原端口的服务，保证调⽤成功。

#### 4、推荐使用 XML 进行配置

XML 配置优先级高于 properties 和 yml，且标签的配置方式更加容易阅读和理解。

#### 5、应用配置负责人参数

配置 `dubbo:application#owner` 负责人参数，这些可以在运维平台上看到，以便于在发现问题时，找到对应服务负责⼈。


