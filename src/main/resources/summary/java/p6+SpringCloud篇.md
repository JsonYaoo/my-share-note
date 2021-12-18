# 八、SpringCloud篇

### 1.1. 如何权衡微服务的利弊？

#### 优点

![1638067092432](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1638067092432.png)

##### 1、快速响应变更

1. 得益于微服务架构遵循了单一职责的架构理念，每块微服务都划进了自身的势力范围，再也不像单体应用那样改一发而动全身，天然适用于小步快跑的开发节奏。
2. 同时，每块微服务都可以独立部署，这样就可以不受限于单体应用的发布节奏与发布窗口等条件，特别适合互联网公司糙快猛的开发模式。

##### 2、独立拓展

微服务与微服务间有非常清晰的边界，且不过度受制于技术栈，可以在当前微服务的技术栈上，有更多的话语权，能够在保持整体技术栈相对统一情况下，给到各个微服务最大的技术自由度。

##### 3、精粒度业务控制

比如说，可以把某段特定的降级熔断逻辑，放在某个特定的微服务上，然后为其指定合理的调用策略和重试策略，还可以更精粒度的局部限流，提高利用率。

##### 4、面向业务和领域模型

1. 使用微服务架构，不再依赖于底层的数据模型，而是依赖于业务和领域模型，并不会暴露给底层数据模型给其他微服务，而只是暴露业务接口和业务对象，在对底层数据模型进行更改时，不会影响到其他业务方。
2. 同时基于业务模型也非常易于抽象，比如抽象到中台等。

#### 缺点

![1638067123418](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1638067123418.png)

##### 1、部署结构复杂

微服务架构模块众多，每个微服务部署、中间件千差万别，并且天生就引入了一堆额外的组件，这样就会使得部署结构变得复杂，提高了运维的成本。

##### 2、依赖平台支撑

1. 微服务架构也考验着公司的整体技术实力，非常依赖于平台的支撑，比如注册中心、配置中心、调用链路分析、服务网关等。
2. 另外，在把单体架构拆分成微服务架构时，也需要投入额外的研发成本。

##### 3、分布式问题

1. 在微服务架构下会存在一致性问题，比如串联调用了多个上下游的微服务，来共同完成一个事务，这时就需要在强一致性、弱一致性和最终一致性方案间做出选择。
2. 如果采用了最终一致性方案，还需要考虑如果做好异常补偿。

##### 4、拆分的水平

1. 微服务的拆分强依赖于当前业务，如果架构师对业务理解不到位，导致微服务拆分的粒度过粗或者过细，使得微服务的优越性大打折扣，甚至王者变青铜。 

#### 考虑要点

##### 1、从业务角度考虑

微服务拆分，不能为了微服务而微服务，需要结合自身业务，围绕业务进行拆分，而在拆之前可以从一下两个业务角度去思考：

| 思考角度     | 含义                                                         |
| ------------ | ------------------------------------------------------------ |
| 业务规模     | 比如两个极端，在业务发展得非常大时拆，和在一开始就使用微服务，此时，微服务拆分则需要考虑，在这两个极端之间，评估所要支出的成本和所能带来的收益，**决定着微服务的拆还是不拆**。 |
| 理解业务领域 | 深入理解业务架构、业务流、未来的业务规划等，**决定着如何去拆微服务**。 |

##### 2、从技术角度考虑

微服务架构落地需要亲身躬行，靠实践经验的积累才能做好，在改造过程中，需要去攻克的技术难点有：

| 技术难点   | 含义                                                         |
| ---------- | ------------------------------------------------------------ |
| 服务治理   | 服务与服务间的调用，是微服务架构第一个需要解决的问题，而**服务治理**指的是，在集群中虚机的日常上下线、扩缩容情况下，动态探知各个服务节点的状态变化，了解哪些节点可以正常提供服务、哪些节点已下线等信息。 |
| 负载均衡   | **负载均衡**指的是，面对茫茫多的服务器，如何根据实际情况，把海量用户请求分发到不同的机器，同时考虑各机器性能强弱、各机房带宽大小、网络响应快慢等实际条件。 |
| 服务调用   | 从 Java 代码发起调用，需要构造 Header 和 Body，非常麻烦，需要解决服务通信的代码编写问题，使得调用一个服务就像调用本地接口一样方便 |
| 服务容错   | 在高并发场景下，有的服务会承担较大的访问请求，可能导致响应时间过慢，甚至超时，如果调用方经常发起重试，那么势必会进一步增加被调用应用的压力，导致一个恶性循环，而解决方案就是**降级和熔断**这两种服务容错技术。 |
| 配置管理   | **服务配置管理**，可以使得随时动态调整应用配置，而无需重启机器。 |
| 服务网关   | **服务网关**，可以在微服务架构下，把用户请求转发到每个不同的服务器上。 |
| 调用链追踪 | **调用链追踪**，可以从前到后展示整个微服务调用链的全景数据。 |
| 消息驱动   | 消息组件不仅可以用于**削峰填谷**，还可以用于微服务间的**系统解耦**。 |
| 服务限流   | 再厉害的系统也有性能瓶颈，而限流则可以通过在源头处削减系统压力，是最经济高效的稳定手段，而微服务后台的服务节点数量庞大，单机版的限流远不能解决问题，因此，需要在集群范围内引入**分布式限流**。 |

### 1.2. 微服务架构设计模式？

#### 1、聚合器模式 | 少见

![1638003414651](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1638003414651.png)

1. 聚合器调用多个服务，从而实现系统所需的功能，每个服务都有自己的缓存和数据库。
2. 它可以是一个简单的 WEB 页面，把检索到的数据进行处理后再展示。
3. 它也可以是一个更高层次的组合服务，对检索到的数据增加业务逻辑后，进一步发布成一个新的微服务（**DRY 原则**），也有自己的缓存和数据库，可以沿 X轴（负载均衡、读写分离） 和 Z轴（分库分表） 独立扩展。
   - **DRY 原则**：
     1. Don't Repeat Yourself，不做重复的事，指在一个设计里，对于任何东西，都应该有且只有一个表示，其它的地方都应该引用这一处。
     2. 这样需要改动的时候，只需调整这一处，所有的地方就都变更过来了。
     3. 而在降低可管理单元复杂度的一个基本策略就是，将他们拆解成更小的单元。

#### 2、代理模式 | 网关

![1638003975019](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1638003975019.png)

1. 类似聚合器模式，但是客户端并不聚合数据，而是会根据业务需求的差别，调用不同的微服务。
2. 代理模式可以仅仅是委派请求，也可以进行数据转换工作。

#### 3、链式模式 | 少见

![1638004098621](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1638004098621.png)

1. ServiceA 接收到请求后，会与 ServiceB 进行通信，类似地，ServiceB 也会同 ServiceC 进行通信。
2. 所有服务都使用同步消息传递，在整个链式调用完成之前，客户端会一直阻塞，因此，服务调用链不宜过长，以免客户端长时间等待，是一个**同步串行处理**过程，无法进行并行处理。

#### 4、分支模式 | 常见

![1638004209220](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1638004209220.png)

1. 允许同时调用两个微服务链，效率更高，适合于并行调用处理。

#### 5、数据共享模式 | 过渡

![1638004289487](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1638004289487.png)

1. 自治是微服务的设计原则之一，就是说微服务是全栈式服务，但在重构现有的单体应用时，SQL 数据库反规范化拆分，可能会导致数据重复和不一致。
2. 因此，在单体应用到微服务架构的**过渡阶段**，可以使用这种设计模式，在这种情况下，部分微服务可能会共享缓存和数据库存储，不过，也只有在两个服务之间，存在**强耦合关系**时才可以。
3. 但这对于基于微服务的新建应用程序而言，是一种反模式，在正常的情况下，不推荐这么设计。

#### 6、异步消息模式 | 常见

![1638005387692](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1638005387692.png)

1. RestFul 设计模式非常流行，但它是同步的，会造成阻塞。
2. 因此，部分基于微服务的架构，可能会选择使用消息队列，来代替 RESTFul 请求和响应，以提升吞吐量。

### 1.3. 微服务拆分原则？

1. **单一职责原则**：每个服务独立出去，有界限地工作，只需关注自身的业务，做到高内聚。
2. **服务自治原则**：每个服务做到独立开发、独立测试、独立构建、独立部署、独立运行，与其他服务解耦。
3. **轻量级通信原则**：每个服务间的调用是轻量级的，并且能够跨平台、跨语言，比如采用 Restful、消息队列等进行通信。
4. **粒度进化原则**：每个服务的粒度没有统一的标准，需要结合具体的业务问题，并且随着业务发展而进化，不要进行过度设计。

### 1.4. 微服务拆分经验方案？

微服务拆分没有一个绝对正确的方案，服务拆分的粒度完全要根据业务场景来规划，而随着业务的发展，原先的架构方案也需要做调整，其常见的拆分方案有：

#### 按压力模型拆分

压力模型，简单来说就是用户访问量，需要识别出某些**超高并发量**的业务，尽可能把这部分业务独立拆出来，其业务场景分类有：

##### 1、高频高并发场景

比如商品详情页，它是一个高频场景，同时也是一个高并发场景。

- **建议**：通常建议将高频高并发场景隔离出来，单独作为一个微服务模块，典型的就是商品详情页的后台服务。

##### 2、低频突发流量场景

比如秒杀，虽然它并不是高频场景，但是会产生突发流量。

- **建议**：对于低频突发流量场景，条件允许也可以剥离出来，但如果必须和其他业务包在一个微服务下，那一定要做好**流控**措施（比如削峰），同时还要考虑异常情况的补偿机制。

##### 3、低频流量场景

多为后台运营团队的服务接口，比如商品图文编辑、添加新的优惠计算规则、上架新商品等，即发生的频率比较低，而且不会造成很高的并发量。

- **建议**：对于低频流量场景，根据业务模型拆分就好。

#### 按业务模型拆分

业务模型拆分的维度有很多，在实际项目中应该综合各个不同维度做考量，其分类有：

##### 1、主链路拆分

主链路是正面战场，必须力保主链路不失守，比如电商中的，商品搜索 => 商品详情页 => 购物车模块 => 订单结算 => 支付业务等，其拆分的目的有：

- **异常容错**：可为主链路建立层次化的多级降级策略，以及合理的熔断策略。

- **调配资源**：主链路通常来讲都是高频场景，自然需要更多的计算资源，最主要的体现就是集群里分配的虚机数量多，因此，把主链路服务单独隔离出来，有利于根据需要指定不同的资源计划。
- **服务隔离**：主链路是主打输出的 C 位，把主链路与其他打辅助的业务隔离开来，可以避免边缘服务的异常情况影响到主链路。

##### 2、领域模型拆分

所谓领域模型，其实就是一套各司其职的服务集合，在做微服务规划时，需要确保各个领域之间有清晰的界限，比如商品服务和订单服务等。

##### 3、用户群体划分

1. 对每个不同的用户群体来说，即便是相同的业务领域，也有该群体独有的业务场景，比如运营、采购、客服、买家、卖家等。
2. 因此，用户群体相当于一个二级域，建立先根据主链路和领域模型一级域的拆分，再结合具体的业务分析，看是否需要在用户领域方向上做更细粒度的拆分。

##### 4、前后台业务分离

在实际项目中，通常会将前台业务和后台业务做一个隔离，符合高频业务（前台）和低频业务（后台）的隔离策略，比如手淘 APP 前台和后台商品管理系统等。

### 1.5. 什么是 SpringCloud？

#### 概念

- SpringCloud 是一系列**框架的有序集合**，利用 SpringBoot 的开发便利性，巧妙地简化了分布式系统基础设施的开发，如**服务注册与发现、配置中心、智能路由、消息总线、负载均衡、断路器、数据监控**等，可以用 SpringBoot 的开发风格做到一键启动和部署。
- SpringCloud 并没有重复制造轮子，只是将各家公司开发的比较成熟、经得起实际考验的服务框架组合起来，通过SpringBoot 风格进行再封装，屏蔽掉了复杂的配置和实现原理，最终给开发者留出了一套简单易懂、易部署、易维护的**分布式系统开发工具包**。

|          | SpringCloud                                                  | SpringBoot                                         |
| -------- | ------------------------------------------------------------ | -------------------------------------------------- |
| 目标     | 关注全局的微服务协调整理治理框架，把 SpringBoot 开发的一个个微服务整合并管理起来，为各个微服务之间提供：配置管理、服务发现、断路器、路由、微代理、事件总线、全局锁、决策竞选、分布式会话等集成服务 | 专注于快速方便地开发单个微服务                     |
| 依赖关系 | SpringCloud 离不开SpringBoot ，属于依赖的关系                | 可以离开SpringCloud 独立使用开发项目，没有依赖关系 |

#### 整体架构

![1638089272529](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1638089272529.png)

- **Netflix**：奈非，一家大名鼎鼎的互联网传媒公司，经过自身业务漫长的微服务改造过程，沉淀出了一系列优秀的微服务组件，再经过 Pivotal（主导 Cloud Foundry）一系列封装后构成了初代的 SpringCloud。
- **Alibaba**：阿里巴巴，凭借 996 + 鸡血文化加持 + 国内互联网行业特有的糙快猛精神，在开源软件上不断开疆拓土，贡献了 SpringCloud Alibaba 组件库。
- **Spring Open Source**：Spring，独家挂牌开源，即**原配**组件。

| 应用功能       | Netfilx                           | Alibaba                  | Spring                   | 其他厂商                                                  |
| -------------- | --------------------------------- | ------------------------ | ------------------------ | --------------------------------------------------------- |
| **服务治理**   | **Eureka**                        | Nacos、Dubbo             | -                        | Google Consul                                             |
| **负载均衡**   | **Ribbon**                        | -                        | springcloud-loadbalancer | -                                                         |
| **服务调用**   | Fegin/**Open Feign**              | -                        | -                        | -                                                         |
| **服务容错**   | **Hystrix** + Turbine + Dashboard | **Sentinel** + Dashboard | -                        | -                                                         |
| **配置管理**   | Archaius                          | Alibaba Cloud ACM、Nacos | **Config**               | 携程 Apollo                                               |
| **服务网关**   | Zuul                              | -                        | **Gateway**              | -                                                         |
| **调用链追踪** | -                                 | -                        | **Sleuth**               | Elasticsearch B.V. ELK、Twitter Zipkin、Apache Skywalking |
| **消息驱动**   | -                                 | RocketMQ                 | **Stream**               | Apache Kafka、VMware RabbitMQ                             |
| **服务限流**   | -                                 | Sentinel + Dashboard     | **Gateway**              | -                                                         |
| 分布式任务调度 | -                                 | Alibaba Cloud SchedulerX | springcloud-task         | 当当网 Elastic-Job                                        |
| 分布式事务     | -                                 | Seata                    | -                        | -                                                         |

#### 优点

1. **系统耦合度低**：不会影响其他模块的开发。
2. **减轻团队的成本**：可以并行开发，不用关注其他人怎么开发，先关注自己的开发。
3. **配置比较简单**：基本用注解就能实现，不用使用过多的配置文件。
4. **微服务跨平台**：可以用任何一种语言开发。
5. **独立部署**：每个微服务可以有自己的独立的数据库也有用公共的数据库。
6. **前后端分离**：直接写后端的代码，不用关注前端怎么开发，直接写自己的后端代码即可，然后暴露接口，通过组件进行服务通信。

#### 缺点

1. **部署比较麻烦**：给运维工程师带来一定的麻烦。
2. **数据管理比较麻烦**：因为微服务可以每个微服务使用一个数据库。
3. **系统集成测试比较麻烦**：一个功能可能涉及多个微服务。
4. **性能监控比较麻烦**：最好开发一个大屏监控系统。

#### SpringCloud VS Dubbo

|              | SpringCloud                          | Dubbo                        |
| ------------ | ------------------------------------ | ---------------------------- |
| 框架定位     | 微服务解决方案，打造微服务生态       | 分布式服务治理框架           |
| 服务通信方式 | Rest API（轻量、灵活、支持 Swagger） | RPC 远程调用（高效、但耦合） |
| 注册中心     | Eureka、Nacos                        | Zookeeper                    |
| 优点         | 使用方便                             | 性能好                       |

### 1.6. 什么是服务治理？

#### 背景

1. 在传统的系统部署中，服务运行在一个固定的已知的IP和端口上，如果一个服务需要调用另一个服务，那么可以通过地址直接调用。
2. 但是，在虚拟化或者容器化的环境中，服务实例的启动和销毁是很频繁的，那么服务地址也是在动态变化的，因此，就产生了服务治理的概念。

#### 概念

![1638094365495](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1638094365495.png)

服务治理就是，在微服务架构中，提供各微服务实例快速上下线、保持各服务正常通信能力的方案总称，可以实现各微服务实例的**自动化注册与服务发现**。

1. **高可用性**：在服务治理麾下的所有微服务节点，不论是被闪电击中，还是被挖掘机铲掉了电源，即使战至最后一个存活节点，服务治理框架也要保证服务的可用性。
2. **分布式调用**：
   1. 微服务节点通常散落在不同的网络环境中，要求服务治理框架具备即使在复杂网络环境下，也能准确获知服务节点的网络地址的能力。
   2. 作为服务消费者，也可以借助服务治理框架的精准制导能力，向服务节点发起请求。
3. **生命周期管理**：服务治理贯穿每个微服务的生命周期，包括从服务上线，持续运行，到服务下线。
4. **健康度检查**：服务治理框架要精准识别"不健康"的微服务节点，然后将其从服务列表中剔除。

#### 实现方案

1. **服务注册**：服务提供方自报家门。
2. **服务发现**：服务消费方拉取注册数据。
3. **心跳检测、服务续约、服务剔除**：一套由服务提供方和注册中心配合完成的去伪存真的过程。
4. **服务下线**：服务提供方发起主动下线。

#### 技术选型

|                  | Eureka   | Consul    | Nacos                        | Zookeeper               | Etcd           |
| ---------------- | -------- | --------- | ---------------------------- | ----------------------- | -------------- |
| SpringCloud 集成 | 支持     | 支持      | 支持                         | 支持                    | 支持           |
| 性能             | 快       | 较慢      | 快                           | 较慢                    | 较慢           |
| 一致性           | 弱一致   | raft      | raft                         | paxos                   | raft           |
| CAP              | AP       | CP        | AP、CP                       | CP                      | CP             |
| 网络协议         | http     | http、dns | http、dns、udp               | 客户端                  | http、grpc     |
| KV 存储服务      | 不支持   | 支持      | 支持                         | 支持                    | 支持           |
| 本质             | 服务     |           | Jar 包                       | 进程                    | Service        |
| 特点             | 节点平等 |           | AP/CP切换、注册/配置中心通用 | Leader 选举、Watch 监听 | 云原生         |
| 用途             | 注册中心 |           | 注册中心和配置中心           | 分布式协调              | 云原生注册中心 |

### 1.7. 详细介绍 Eureka？

#### 背景 

1. 在传统应用组件间调用，是通过接口规范约束来实现的，从而实现不同模块间良好协作。
2. 但是在被拆分成微服务后，每个微服务实例的网络地址和数量都可能**动态变化**，导致使用原来硬编码地址的方式极不方便，因此，需要一个中心化的组件来进行服务的登记和管理。

#### 概念

Eureak，是 SpringCloud 服务治理的一种具体解决方案，是 Netflix 开源微服务框架一系列项目中的一个，Spring Cloud 对其进行了 SpringBoot 二次封装，形成了 Spring Cloud Netflix 子项目。

- **Eureka Server**：
  1. 一个公共注册中心服务，为 Eureka Client 提供服务注册和服务发现的功能，维护已注册到自身的Eureka Client 的相关信息。
  2. 同时，提供接口给 Eureka Client 获取注册表中其他服务的信息，使得动态变化的Eureka Client 能够进行服务间的相互调用，实现服务治理。
- **Eureka Client**：
  1. **作为服务提供者**，可以将自己的服务信息，通过 Restful 的方式注册到 Eureka Server上，并在正常范围内维护自己信息一致性，方便其他服务发现自己。
  2. **作为服务消费者**：可以通过 Eureka Server 获取到自己依赖的其他服务信息，完成服务调用，并且内置了负载均衡器，用来进行基本的负载均衡。

=> 因此，Eureka 是**服务注册和服务发现**的基础组件，屏蔽了 Eureka Server 和 Eureka Client 的交互细节，使得开发者能够不再重点关注服务治理，从而把精力放在业务上。

#### 架构原理

![1638234417008](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1638234417008.png)

**注册中心本质**，是存储每个服务客户端的注册信息，EurekaClient 从 EurekaServer 同步获取服务注册列表，通过一定的规则选择一个服务进行调用。

- **服务注册中心**：提供服务注册和发现的功能。每个Eureka Client向Eureka Server注册自己的信息，也可以通过Eureka Server获取到其他服务的信息达到发现和调用其他服务的目的。
- **服务提供者**：是一个 Eureka client，向 Eureka Server 注册和更新自己的信息，同时可以从 Eureka Server 注册表中获取到其他服务的信息。
  1. **服务注册**：服务提供者向 Eureka Server，注册自身的元数据以供服务发现。
  2. **服务续约**：通过发送心跳到 Eureka Server，维持和更新注册表中服务实例元数据的**有效性**，如果在一定时长内，Eureka Server 没有收到 Eureka Client的心跳信息，则默认认为它已下线，会把该服务实例信息从注册表中删除。
  3. **服务下线**：服务提供者在关闭时，主动向 Eureka Server 注销服务实例元数据，此时，该服务实例数据将会从 Eureka Server 注册表中删除。
- **服务消费者**：也是一个 Eureka client，通过从 Eureka Server 注册表中获取到其他服务的信息，找到所需要的服务，然后发起远程调用。
  1. **服务发现**：获取注册表信息，Eureka Client 向 Eureka Server 请求注册表信息，以供发起远程调用。

#### 使用方式

##### Eureka Server

###### POM 依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

###### 配置文件

```properties
spring.application.name=eureka-server
server.port=20000

# 注册中心自己不需要服务注册和服务拉取(默认需要拉取)
eureka.instance.hostname=localhost
eureka.client.register-with-eureka=false
eureka.client.fetch-registry=false

# 强制关闭注册中心服务自保功能(关闭后服务自保的自动开关不起作用)
eureka.server.enable-self-preservation=false
# 注册中心每隔多久触发一次服务剔除(服务端定时任务执行服务剔除操作)
eureka.server.eviction-interval-timer-in-ms=10000
```

###### 启动类

```java
// Eureka Server测试启动类
@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {
    public static void main(String[] args) {
        new SpringApplicationBuilder(EurekaServerApplication.class)
                .web(WebApplicationType.SERVLET)
                .run(args);
    }
}
```

##### Eureka Client

###### POM 依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

###### 配置文件

```properties
spring.application.name=eureka-client
server.port=30000

# Eureka注册中心地址: defaultZone是serviceUrl中的一个map属性, /eureka/为默认写法(可以更改)
#eureka.client.serviceUrl.defaultZone=http://localhost:20000/eureka/

# 测试Eureka集群+互备: 只配置单注册中心
#eureka.client.serviceUrl.defaultZone=http://localhost:20011/eureka/
# 测试Eureka集群+互备: 只配置双注册中心
eureka.client.serviceUrl.defaultZone=http://peer1:20000/eureka/,http://peer2:40000/eureka/

# 高可用服务改造(服务提供者): 客户端每个X秒钟, 向注册中心发送一条续约指令(服务续约)
eureka.instance.lease-renewal-interval-in-seconds=5
# 高可用服务改造(服务提供者): 如果X秒内, 注册中心依然没有收到客户端的续约请求, 则判定客户端服务过期(客户端声明服务剔除周期判定时间)
eureka.instance.lease-expiration-duration-in-seconds=10
```

###### 启动类

```java
// Eureka Client测试启动类(服务提供者，而消费者也类似)
@SpringBootApplication
@EnableDiscoveryClient
public class EurekaClientApplication {
    public static void main(String[] args) {
        new SpringApplicationBuilder(EurekaClientApplication.class)
                .web(WebApplicationType.SERVLET)
                .run(args);
    }
}
```

##### Portal UI

![1638107108492](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1638107108492.png)

![1638107129321](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1638107129321.png)

![1638108556248](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1638108556248.png)

| 模块                    | 属性                              | 释义                                                         |
| ----------------------- | --------------------------------- | ------------------------------------------------------------ |
| System Status           | Environment                       | 当前环境，默认值为 test，一般不用修改，为默认值就好          |
|                         | Data Center                       | 当前数据中心，默认值为 default，一般不用修改，为默认值就好   |
|                         | Current time                      | 当前系统时间，并不会主动刷新，只当浏览器刷新页面时才刷新     |
|                         | Uptime                            | 指 Eureka Server 从启动到至今所经历的时间，比如 01：30 表示已经运行了 1 小时 30 分钟 |
|                         | Lease expiration enabled          | 是否启用租约过期，跟服务续约、服务剔除、服务自保都有一定关系 |
|                         | Renews threshold                  | 每分钟最少的续约数，比如 3 表示每分钟至少收到 3 个续约请求   |
|                         | Renews（last min）                | 最近一分钟的续约数（不包括目前的一分钟）                     |
| EMERGENCY红字           | 情况一                            | 表示由于最近一分钟续约数小于每分钟最少的续约数，从而已进入自保状态 |
|                         | 情况二                            | 自我保护机关已被强制关闭                                     |
|                         | 情况三                            | 出现多数节点与 Eureka 续约出现问题                           |
| DS Replicas             | Application                       | 服务的应用名称，没指定时默认为 UNKOWN                        |
|                         | AMIs                              | 心跳检测机制                                                 |
|                         | Availability Zones                | （1）表示可用分区数                                          |
|                         | Status                            | 服务实例可用状态，（1）表示可用实例数                        |
| General Info            | total-avail-memory                | 当前 Eureka 实例可用内存                                     |
|                         | environment                       | 当前环境，默认值为 test，一般不用修改，为默认值就好          |
|                         | num-of-cpus                       | 当前 CPU 核数                                                |
|                         | current-memory-usage              | 当前已使用内存，44% 表示当前机器已使用 44% 的内存            |
|                         | server-uptime                     | 指 Eureka Server 从启动到至今所经历的时间，比如 01：30 表示已经运行了 1 小时 30 分钟 |
|                         | registered-replicas               | 其他已注册的 Eureka 集群副本                                 |
|                         | unavailable-replicas              | 其他不可用的 Eureka 集群副本                                 |
|                         | available-replicas                | 其他可用的 Eureka 集群副本                                   |
| Instance Info           | ipAddr                            | 当前机器的 IP 地址                                           |
|                         | status                            | 当前服务的状态，UP 表示已上线                                |
| LAST 1000 SINCE STARTUP | Last 1000 cancelled leases        | 过去最近 1000 个被取消的实例租约                             |
|                         | Last 1000 newly registered leases | 过去最近 1000 个新注册的实例租约                             |

##### 常用 Rest 接口

| 用途                     | URL                                                          |
| ------------------------ | ------------------------------------------------------------ |
| 查看所有服务的注册列表   | GET http://localhost:1001/eureka/apps                        |
| 查看某一个服务的注册列表 | GET http://localhost:1001/eureka/apps/SERVICE-NAME           |
| 服务下线                 | PUT http://localhost:1001/eureka/apps/SERVICE-NAME/INSTANCE-NAME/status?value=OUT_OF_SERVICE |
| 服务下线后恢复           | PUT http://localhost:1001/eureka/apps/SERVICE-NAME/INSTANCE-NAME/status?value=UP |
| 服务剔除                 | DELETE http://localhost:1001/eureka/apps/SERVICE-NAME/INSTANCE-NAME |

#### 注册中心启动原理 | Eureka Server

##### 1、导入配置类

```java
// 1、@EnableEurekaServer
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(EurekaServerMarkerConfiguration.class)
public @interface EnableEurekaServer {

}

@Configuration
public class EurekaServerMarkerConfiguration {

    // 2、构造Marker实例，以标识注册中心允许启动
	@Bean
	public Marker eurekaServerMarkerBean() {
		return new Marker();
	}

	class Marker {

	}
}
```

##### 2、Spring SPI 配置

```properties
# ..\spring-cloud-netflix-eureka-server-2.1.1.RELEASE.jar\META-INF\spring.factories
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  org.springframework.cloud.netflix.eureka.server.EurekaServerAutoConfiguration
```

##### 3、初始化 Bean 配置类

```java
// 1、标识当前类为配置类
@Configuration
// 2、注入Eureka Server启动核心类
@Import(EurekaServerInitializerConfiguration.class)
// 3、标识为true，才进行初始化
@ConditionalOnBean(EurekaServerMarkerConfiguration.Marker.class)
// 4、使没有@Component的配置类生效
@EnableConfigurationProperties({ EurekaDashboardProperties.class,
		InstanceRegistryProperties.class })
// 5、注入properties配置，属性需从Environment中获取
@PropertySource("classpath:/eureka/server.properties")
public class EurekaServerAutoConfiguration extends WebMvcConfigurerAdapter {

    ....
	// 6、注入Eureka服务端相关配置
	@Autowired
	private EurekaServerConfig eurekaServerConfig;
	// 7、注入Eureka客户端相关配置
	@Autowired
	private EurekaClientConfig eurekaClientConfig;
    
	// 8、初始化集群注册表——PeerAwareInstanceRegistry
	@Bean
	public PeerAwareInstanceRegistry peerAwareInstanceRegistry(ServerCodecs serverCodecs) {
		...
	}
    
	// 9、初始化集群节点集合——PeerEurekaNodes
	@Bean
	@ConditionalOnMissingBean
	public PeerEurekaNodes peerEurekaNodes(PeerAwareInstanceRegistry registry, ServerCodecs serverCodecs,
			ReplicationClientAdditionalFilters replicationClientAdditionalFilters) {
		...
	}
    
	// 10、Eureka服务端上下文——EurekaServerContext
	@Bean
	@ConditionalOnMissingBean
	public EurekaServerContext eurekaServerContext(ServerCodecs serverCodecs, PeerAwareInstanceRegistry registry, PeerEurekaNodes peerEurekaNodes) {
		...
	}
    
	// 11、 Eureka服务端启动类——EurekaServerBootstrap
	@Bean
	public EurekaServerBootstrap eurekaServerBootstrap(PeerAwareInstanceRegistry registry,EurekaServerContext serverContext) {
		...
	}
    
	// 12. jersey过滤器——FilterRegistrationBean
	@Bean
    public FilterRegistrationBean<?> jerseyFilterRegistration(javax.ws.rs.core.Application eurekaJerseyApp) {
        ...
    }
}
```

##### 4、LifecycleProcessor#onRefresh 回调

```java
package org.springframework.context.support;

public abstract class AbstractApplicationContext extends DefaultResourceLoader
implements ConfigurableApplicationContext {
	// 1、SpringBoot/Spring启动时调用
	@Override
    public void refresh() throws BeansException, IllegalStateException {
    	...
    	// 2、最后一步，完成最后的上下文刷新
    	finishRefresh();
    	...
    }
    
    protected void finishRefresh() {
   		...
    	// 3、上下文刷新的通知，比如用于自动启动组件
    	getLifecycleProcessor().onRefresh();
    	..
    }
}

public class DefaultLifecycleProcessor implements LifecycleProcessor, BeanFactoryAware {
	// 4、使用默认的生命周期后置处理器
	@Override
	public void onRefresh() {
		startBeans(true);
		this.running = true;
	}
	
    private void startBeans(boolean autoStartupOnly) {
    	...
    	// 5、启动每一个生命周期方法
    	phases.get(key).start();
    	...
    }
    
    // 维护一组应该启动的Lifecycle bean的Helper类
    private class LifecycleGroup {
        public void start() {
        	...
        	// 6、启动每一个生命周期方法
			doStart(this.lifecycleBeans, member.name, this.autoStartupOnly);
			...
        }
    }
    
    private void doStart(Map<String, ? extends Lifecycle> lifecycleBeans, String beanName, boolean autoStartupOnly) {
   		...
   		// 7、启动生命周期方法
    	bean.start();
    	...
    }
}

// 8、由于实现了SmartLifecycle接口，因此SpringBoot启动时会回调start方法
public class EurekaServerInitializerConfiguration
    implements ServletContextAware, SmartLifecycle, Ordered {
    
	@Override
    public void start() {
         // 9、异步启动任务
		new Thread(new Runnable() {
			@Override
			public void run() {
                // 10、初始化Eureka Server上下文
                eurekaServerBootstrap.contextInitialized(
                    EurekaServerInitializerConfiguration.this.servletContext);
                log.info("Started Eureka Server");
                ...// 发布完成事件等等
			}
		}).start();
    }
}
```

##### 5、初始化 Eureka Server 上下文

```java
public class EurekaServerBootstrap {
	public void contextInitialized(ServletContext context) {
        ...
		// 1、初始化Eureka环境
        initEurekaEnvironment();
        // 2、初始化Eureka上下文
        initEurekaServerContext();
        ...
	}
    
    protected void initEurekaServerContext() throws Exception {
        ...
		log.info("Initialized server context");

		// 3、向其他Eureka Server同步租约实例，如果已经注册了的，则在当前Eureka Server注册一次
		int registryCount = this.registry.syncUp();
        // 4、初始化启动变量，以及启动服务剔除定时任务
		this.registry.openForTraffic(this.applicationInfoManager, registryCount);
		// 5、注册所有监视统计信息
		EurekaMonitors.registerAllStats();
    }
}

public class InstanceRegistry extends PeerAwareInstanceRegistryImpl
implements ApplicationContextAware {
	@Override
	public void openForTraffic(ApplicationInfoManager applicationInfoManager, int count) {
		// 4.1. 初始化启动变量，以及启动服务剔除定时任务
		super.openForTraffic(applicationInfoManager,
				count == 0 ? this.defaultOpenForTrafficCount : count);
	}
}

public class PeerAwareInstanceRegistryImpl extends AbstractInstanceRegistry implements PeerAwareInstanceRegistry {
    @Override
    public void openForTraffic(ApplicationInfoManager applicationInfoManager, int count) {
    	// 4.2. 设置期望的客户端数量，这里为1
    	this.expectedNumberOfClientsSendingRenews = count;
    	// 4.3. 设置最小续租阈值数量 = (int) 1 * (60/30) * 0.85 = 1，用于开启自保开关
    	updateRenewsPerMinThreshold();
    	// 4.4. 设置系统启动时间
    	this.startupTime = System.currentTimeMillis();
    	...
    	// 4.5. 设置UP实例状态
    	applicationInfoManager.setInstanceStatus(InstanceStatus.UP);
    	// 4.6. 设置完毕，上下文后置初始化处理
    	super.postInit();
    }
}

public abstract class AbstractInstanceRegistry implements InstanceRegistry {
    protected void postInit() {
   		// 4.7. 设置服务剔除任务
        evictionTaskRef.set(new EvictionTask());
        // 4.8. 默认延迟60s后开始执行任务，且每60s执行一次
        evictionTimer.schedule(evictionTaskRef.get(),
                serverConfig.getEvictionIntervalTimerInMs(),
                serverConfig.getEvictionIntervalTimerInMs());
    }
}
```

#### 客户端启动原理 | Eureka Client

##### 1、导入配置类

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
// 2. 导入配置类EnableDiscoveryClientImportSelector
@Import(EnableDiscoveryClientImportSelector.class)
public @interface EnableDiscoveryClient {
    // 1. 默认为true，所以不设置@EnableDiscoveryClient，服务也能完成注册和发现
    boolean autoRegister() default true;
}

// 3. ImportSelector接口的返回值会递归进行解析，把解析到的类全名按照@Configuration进行处理
@Order(Ordered.LOWEST_PRECEDENCE - 100)
public class EnableDiscoveryClientImportSelector
    extends SpringFactoryImportSelector<EnableDiscoveryClient> {
	@Override
    public String[] selectImports(AnnotationMetadata metadata) {
        ...
		if (autoRegister) {
			List<String> importsList = new ArrayList<>(Arrays.asList(imports));
            // 4. 指定导入AutoServiceRegistrationConfiguration
			importsList.add(
					"org.springframework.cloud.client.serviceregistry.AutoServiceRegistrationConfiguration");
			imports = importsList.toArray(new String[0]);
		}
        ...
    }
}

@Configuration
// 5. 读取spring.cloud.service-registry.auto-registration文件，并允许注入属性
@EnableConfigurationProperties(AutoServiceRegistrationProperties.class)
// 6. 默认设置spring.cloud.service-registry.auto-registration.enabled开关为true
@ConditionalOnProperty(value = "spring.cloud.service-registry.auto-registration.enabled", matchIfMissing = true)
public class AutoServiceRegistrationConfiguration {

}
```

##### 2、Spring SPI 配置

```properties
# .../spring-cloud-netflix-eureka-client/2.1.1.RELEASE/spring-cloud-netflix-eureka-client-2.1.1.RELEASE.jar!/META-INF/spring.factories
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
# Eureka client自动配置类，负责client中关键beans的配置和初始化
org.springframework.cloud.netflix.eureka.EurekaClientAutoConfiguration,\
# Ribbon负载均衡自动配置类
org.springframework.cloud.netflix.ribbon.eureka.RibbonEurekaAutoConfiguration,\
# 服务注册和健康检查器自动配置类
org.springframework.cloud.netflix.eureka.EurekaDiscoveryClientConfiguration
```

##### 3、服务注册 & 健康检查自动装配

```java
@Configuration
@EnableConfigurationProperties
@ConditionalOnClass(EurekaClientConfig.class)
// 1、默认设置eureka.client.enabl=true
@ConditionalOnProperty(value = "eureka.client.enabled", matchIfMissing = true)
@ConditionalOnDiscoveryEnabled
public class EurekaDiscoveryClientConfiguration {
    
    class Marker {

	}
    
    // 2、注入服务启动标识
	@Bean
	public Marker eurekaDiscoverClientMarker() {
		return new Marker();
	}
    
    // 3、默认关闭健康检查
    @Configuration
	@ConditionalOnProperty(value = "eureka.client.healthcheck.enabled", matchIfMissing = false)
    protected static class EurekaHealthCheckHandlerConfiguration {
        ...
    }
    
    // 4、注册Spring上下文刷新完毕监听器
    @Configuration
	@ConditionalOnClass(RefreshScopeRefreshedEvent.class)
	protected static class EurekaClientConfigurationRefresher
        implements ApplicationListener<RefreshScopeRefreshedEvent> {
        
		@Autowired(required = false)
		private EurekaClient eurekaClient;

		@Autowired(required = false)
		private EurekaAutoServiceRegistration autoRegistration;

		public void onApplicationEvent(RefreshScopeRefreshedEvent event) {
			...
            // 5、确保在Spring上下文刷新事件后不为空，否则将重新注册
			if (autoRegistration != null) {
				this.autoRegistration.stop();
				this.autoRegistration.start();
			}
		}
    }
}
```

##### 3、Eureka Client 自动装配

```java
@Configuration
@EnableConfigurationProperties
@ConditionalOnClass(EurekaClientConfig.class)
@Import(DiscoveryClientOptionalArgsConfiguration.class)
// 1、需要有服务启动标识，默认有
@ConditionalOnBean(EurekaDiscoveryClientConfiguration.Marker.class)
// 2、默认设置eureka.client.enabl=true
@ConditionalOnProperty(value = "eureka.client.enabled", matchIfMissing = true)
// 3、默认设置spring.cloud.discovery.enabled=true
@ConditionalOnDiscoveryEnabled
// 4、该类注入在下面这些类注入之前
@AutoConfigureBefore({ NoopDiscoveryClientAutoConfiguration.class,
		CommonsClientAutoConfiguration.class, ServiceRegistryAutoConfiguration.class })
// 5、该类注入在下面这些类注入之后
@AutoConfigureAfter(name = {
		"org.springframework.cloud.autoconfigure.RefreshAutoConfiguration",
"org.springframework.cloud.netflix.eureka.EurekaDiscoveryClientConfiguration",	"org.springframework.cloud.client.serviceregistry.AutoServiceRegistrationAutoConfiguration" })
public class EurekaClientAutoConfiguration {
    
    // 6、注入EurekaClient配置类
	@Bean
	@ConditionalOnMissingBean(value = EurekaClientConfig.class, search = SearchStrategy.CURRENT)
    public EurekaClientConfigBean eurekaClientConfigBean(ConfigurableEnvironment env) {
        ...
    }
    
    // 7、注入EurekaClient配置管理类
	@Bean
	@ConditionalOnMissingBean
	public ManagementMetadataProvider serviceManagementMetadataProvider() {
		return new DefaultManagementMetadataProvider();
	}
    
    // 8、读取eureka.instance.hostname、eureka.instance.prefer-ip-address、eureka.instance.ip-address、server.servlet.context-path、server.port等配置，注入EurekaInstanceConfigBean instance实例
    @Bean
	@ConditionalOnMissingBean(value = EurekaInstanceConfig.class, search = SearchStrategy.CURRENT)
	public EurekaInstanceConfigBean eurekaInstanceConfigBean(InetUtils inetUtils,
                                                             ManagementMetadataProvider managementMetadataProvider) {
        ...
    }
    
    // 9、注入DiscoveryClient，用于包装EurekaClient
    @Bean
	public DiscoveryClient discoveryClient(EurekaClient client,
			EurekaClientConfig clientConfig) {
		return new EurekaDiscoveryClient(client, clientConfig);
	}
    
    // 10、注入EurekaServiceRegistry
    @Bean
	public EurekaServiceRegistry eurekaServiceRegistry() {
		return new EurekaServiceRegistry();
	}
    
    // 11、注入EurekaAutoServiceRegistration，对应上面所说的《确保在Spring上下文刷新事件后不为空，否则将重新注册》
    @Bean
	@ConditionalOnBean(AutoServiceRegistrationProperties.class)
	@ConditionalOnProperty(value = "spring.cloud.service-registry.auto-registration.enabled", matchIfMissing = true)
	public EurekaAutoServiceRegistration eurekaAutoServiceRegistration(
			ApplicationContext context, EurekaServiceRegistry registry,
			EurekaRegistration registration) {
		return new EurekaAutoServiceRegistration(context, registry, registration);
	}
    
    // #重点#
    @Configuration
	@ConditionalOnMissingRefreshScope
    protected static class EurekaClientConfiguration {
        ...
        // 12、注入EurekaClient核心类，且在销毁时调用EurekaClient#shutdown，进行服务下线
		@Bean(destroyMethod = "shutdown")
		@ConditionalOnMissingBean(value = EurekaClient.class, search = SearchStrategy.CURRENT)
		public EurekaClient eurekaClient(ApplicationInfoManager manager,
				EurekaClientConfig config) {
			return new CloudEurekaClient(manager, config, this.optionalArgs,
					this.context);
		}
        
        // 13、注入app管理器
		@Bean
		@ConditionalOnMissingBean(value = ApplicationInfoManager.class, search = SearchStrategy.CURRENT)
		public ApplicationInfoManager eurekaApplicationInfoManager(
				EurekaInstanceConfig config) {
			InstanceInfo instanceInfo = new InstanceInfoFactory().create(config);
			return new ApplicationInfoManager(config, instanceInfo);
		}
        
        // 14、注入服务注册实例
		@Bean
		@ConditionalOnBean(AutoServiceRegistrationProperties.class)
		@ConditionalOnProperty(value = "spring.cloud.service-registry.auto-registration.enabled", matchIfMissing = true)
		public EurekaRegistration eurekaRegistration(EurekaClient eurekaClient,
				CloudEurekaInstanceConfig instanceConfig,
				ApplicationInfoManager applicationInfoManager,
				@Autowired(required = false) ObjectProvider<HealthCheckHandler> healthCheckHandler) {
			return EurekaRegistration.builder(instanceConfig).with(applicationInfoManager)
					.with(eurekaClient).with(healthCheckHandler).build();
		}
    }
    ...// 15、注入一些支持配置刷新的EurekaClient相关类、以及健康检查指标类等等
}
```

##### 4、注入 EurekaClient 核心类

```java
// 0. 在EurekaClientAutoConfiguration进行了自动注入，这里介绍如何创建EurekaClient核心类
public class CloudEurekaClient extends DiscoveryClient {
	public CloudEurekaClient(ApplicationInfoManager applicationInfoManager,
			EurekaClientConfig config, AbstractDiscoveryClientOptionalArgs<?> args,
			ApplicationEventPublisher publisher) {
        // 1、调用父类DiscoveryClient构造器
		super(applicationInfoManager, config, args);
		...
	}
}

@Singleton
public class DiscoveryClient implements EurekaClient {
    public DiscoveryClient(ApplicationInfoManager applicationInfoManager, final EurekaClientConfig config, AbstractDiscoveryClientOptionalArgs args) {
        // 2、调用其他构造器
        this(applicationInfoManager, config, args, new Provider<BackupRegistry>() {
            private volatile BackupRegistry backupRegistryInstance;

            @Override
            public synchronized BackupRegistry get() {
                ...
            }
        });
    }
    
    // 3、注入构造器参数实例ApplicationInfoManager、EurekaClientConfig config和AbstractDiscoveryClientOptionalArgs
    @Inject
    DiscoveryClient(ApplicationInfoManager applicationInfoManager, EurekaClientConfig config, AbstractDiscoveryClientOptionalArgs args,
                    Provider<BackupRegistry> backupRegistryProvider) {
		...// 4、初始化一些配置参数
        try {
            // 5、构建2c、无边界线程、无边界容量的后台延迟线程池DiscoveryClient-n，用于定时执行
            scheduler = Executors.newScheduledThreadPool(2,
                    new ThreadFactoryBuilder()
                            .setNameFormat("DiscoveryClient-%d")
                            .setDaemon(true)
                            .build());
            // 6、构建1c、2max、0空闲、0容量的后台线程池DiscoveryClient-HeartbeatExecutor-n，用于执行心跳检测
            heartbeatExecutor = new ThreadPoolExecutor(
                    1, clientConfig.getHeartbeatExecutorThreadPoolSize(), 0, TimeUnit.SECONDS,
                    new SynchronousQueue<Runnable>(),
                    new ThreadFactoryBuilder()
                            .setNameFormat("DiscoveryClient-HeartbeatExecutor-%d")
                            .setDaemon(true)
                            .build()
            );
            
			// 7、构建1c、2max、0空闲、0容量的后台线程池DiscoveryClient-CacheRefreshExecutor-n，用于执行缓存刷新
            cacheRefreshExecutor = new ThreadPoolExecutor(
                    1, clientConfig.getCacheRefreshExecutorThreadPoolSize(), 0, TimeUnit.SECONDS,
                    new SynchronousQueue<Runnable>(),
                    new ThreadFactoryBuilder()
                            .setNameFormat("DiscoveryClient-CacheRefreshExecutor-%d")
                            .setDaemon(true)
                            .build()
            );

            ...
        } catch (Throwable e) {
            throw new RuntimeException("Failed to initialize DiscoveryClient!", e);
        }
        
        // 8、根据fetch-registry配置参数，增量拉取拉取注册表信息
        if (clientConfig.shouldFetchRegistry() && !fetchRegistry(false)) {
            // 8.1. 由于拉取没发生异常，所以不会返回false，这里也就不会进来了
            fetchRegistryFromBackup();
        }
        
        ...// 先初始化一些注册前的信息
        
        // 9、根据register-with-eureka配置参数向服务端注册，这里由于客户端初始化无需强制执行初始注册，所以也不会进行先注册
        if (clientConfig.shouldRegisterWithEureka() && clientConfig.shouldEnforceRegistrationAtInit()) {
            try {
                if (!register() ) {
                    throw new IllegalStateException("Registration error at startup. Invalid server response.");
                }
            } catch (Throwable th) {
                logger.error("Registration error at startup: {}", th.getMessage());
                throw new IllegalStateException(th);
            }
        }
        
        // 10、最后，初始化调度任务（例如集群解析器、心跳、instanceInfo 复制器、获取注册表）
        initScheduledTasks();
        
        ...// 注册到监控中心、注册DiscoveryClient实例 & 配置实例到管理器中、设置initTimestampMs、打印日志等操作
    }
    
    // 初始化调度任务（例如集群解析器、心跳、instanceInfo 复制器、获取注册表）
    private void initScheduledTasks() {
        // 10.1、默认需要拉取注册表信息
        if (clientConfig.shouldFetchRegistry()) {
            ...
            // 10.2、启动超时时间30s，最大超时300s，延迟30s执行，交由cacheRefreshExecutor线程池处理的，以指定的时间间，隔获取注册表信息任务（服务发现）
            scheduler.schedule(
                    new TimedSupervisorTask(
                            "cacheRefresh",
                            scheduler,
                            cacheRefreshExecutor,
                            registryFetchIntervalSeconds,// 30s，超时时间
                            TimeUnit.SECONDS,
                            expBackOffBound,// 10s，30*10，用于计算最大超时时间
                            // 以指定的时间间，隔获取注册表信息任务
                            new CacheRefreshThread()
                    ),
                    // 30s，延迟执行时间
                    registryFetchIntervalSeconds, TimeUnit.SECONDS);
        }
        
        // 10.3、默认需要注册服务到Eureka
        if (clientConfig.shouldRegisterWithEureka()) {
            ...

            // 10.4、非默认参数，启动续租超时时间5s，最大超时50s，延迟5s执行，交由heartbeatExecutor线程池处理的，在给定的时间间隔内，更新租约的心跳任务（心跳检测）
            scheduler.schedule(
                    new TimedSupervisorTask(
                            "heartbeat",
                            scheduler,
                            heartbeatExecutor,
                            renewalIntervalInSecs,// 5s，续租超时时间
                            TimeUnit.SECONDS,
                            expBackOffBound,// 10s，5*10，用于计算最大超时时间
                        	// 在给定的时间间隔内，更新租约的心跳任务
                            new HeartbeatThread()
                    ),
               		 // 5s，延迟执行时间
                    renewalIntervalInSecs, TimeUnit.SECONDS);

            // 10.5、构建更新和复制本地实例信息到远程服务器的任务，副本同步周期30s
            instanceInfoReplicator = new InstanceInfoReplicator(
                    this,
                    instanceInfo,
                    clientConfig.getInstanceInfoReplicationIntervalSeconds(),// 30s
                    2);

            // 10.6、构造服务实例监听器
            statusChangeListener = new ApplicationInfoManager.StatusChangeListener() {
                ...// getId()
                    
                @Override
                public void notify(StatusChangeEvent statusChangeEvent) {
                    ...// 打印日志
                    // 10.7、在应用状态发生变化时，刷新服务实例信息，在服务实例信息发生改变时，向server注册 => 底层调用instanceInfoReplicator#run方法
                    instanceInfoReplicator.onDemandUpdate();
                }
            };
            
            // 10.8、默认设置监听器
            if (clientConfig.shouldOnDemandUpdateStatusChange()) {
       applicationInfoManager.registerStatusChangeListener(statusChangeListener);
            }

            // 10.9、启动服务实例更新任务，默认第一次延迟40s运行，并设置isInstanceInfoDirty=true和lastDirtyTimestamp，表示第一次需要注册
            instanceInfoReplicator.start(
               clientConfig.getInitialInstanceInfoReplicationIntervalSeconds()// 40s
            );
        } else {
            logger.info("Not registering with Eureka server per configuration");
        }
    }
    
    // 10.10、启动服务实例更新任务，默认第一次延迟40s执行，并设置isInstanceInfoDirty=true和lastDirtyTimestamp脏时间戳（后面用于服务续约），表示第一次需要注册
    public void start(int initialDelayMs) {
        if (started.compareAndSet(false, true)) {
            instanceInfo.setIsDirty();
            Future next = scheduler.schedule(this, initialDelayMs, TimeUnit.SECONDS);
            scheduledPeriodicRef.set(next);
        }
    }
}
```

#### 服务注册原理

##### 发起请求 | Eureka Client

###### 1、第一次调用 InstanceInfoReplicator#run

| 配置                                  | 默认 | 说明                               |
| ------------------------------------- | ---- | ---------------------------------- |
| eureka.registration.enabled           | true | Eureka Client 是否开启服务注册     |
| eureka.appinfo.replicate.interval     | 30s  | 服务实例副本同步周期               |
| eureka.appinfo.initial.replicate.time | 40s  | 服务实例服务副本同步的初始延迟时间 |

```java
// 1、在构造EurekaClient时，调用了instanceInfoReplicator#start，来启动服务实例更新任务，默认第一次延迟40s运行，并设置isInstanceInfoDirty=true和lastDirtyTimestamp（后面用于服务续约），表示第一次需要注册
class InstanceInfoReplicator implements Runnable {
    public void run() {
        try {
            // 2、刷新当前本地instanceInfo，在观察到更改的有效刷新后，instanceInfo#isDirty会被设置为true，并且更新lastDirtyTimestamp脏时间戳（用于后面的服务续约）
            discoveryClient.refreshInstanceInfo();

            // 3、获取lastDirtyTimestamp，如果存在，则需要当前服务重新发起注册
            Long dirtyTimestamp = instanceInfo.isDirtyWithTime();
            if (dirtyTimestamp != null) {
                // 4、重新发起注册，第一次调用InstanceInfoReplicator#run则必定发起注册
                discoveryClient.register();
                // 5、重新注册成功，则清空isDirty，设置isInstanceInfoDirty=false
                instanceInfo.unsetIsDirty(dirtyTimestamp);
            }
        } catch (Throwable t) {
            logger.warn("There was a problem with the instance info replicator", t);
        } finally {
            // 6、最后再延迟30s后重新执行该任务
            Future next = scheduler.schedule(this, replicationIntervalSeconds, TimeUnit.SECONDS);
            scheduledPeriodicRef.set(next);
        }
    }
}
```

###### 2、调用 DiscoveryClient#register

```java
@Singleton
public class DiscoveryClient implements EurekaClient {
    // 4.1、通过进行适当的 REST 调用来注册 eureka 服务
    boolean register() throws Throwable {
        logger.info(PREFIX + "{}: registering service...", appPathIdentifier);
        EurekaHttpResponse<Void> httpResponse;
        try {
            // 4.2、调用EurekaHttpClient#register进行服务注册
            httpResponse = eurekaTransport.registrationClient.register(instanceInfo);
        } catch (Exception e) {
            logger.warn(..., e);
            throw e;
        }
        if (logger.isInfoEnabled()) {
            logger.info(...);
        }
        // 4.24、响应成功，状态码等于NO_CONTENT(204, "No Content"),代表服务注册成功
        return httpResponse.getStatusCode() == Status.NO_CONTENT.getStatusCode();
    }
}

public abstract class EurekaHttpClientDecorator implements EurekaHttpClient {
    
    // 4.4、多态调用SessionedEurekaHttpClient#execute
    // 4.8、多态调用RetryableEurekaHttpClient#execute
    // 4.13、多态调用RedirectingEurekaHttpClient#execute
    // 4.17、多态调用MetricsCollectingEurekaHttpClient#execute
    protected abstract <R> EurekaHttpResponse<R> execute(RequestExecutor<R> requestExecutor);
     
    @Override
    public EurekaHttpResponse<Void> register(final InstanceInfo info) {
        // 4.3、调用模板execute方法
        // 4.7、多态调用前又会先调用抽象父类的register方法，然后又会去调用模板execute方法
        // 4.12、多态调用前又会先调用抽象父类的register方法，然后又会去调用模板execute方法
        // 4.16、多态调用前又会先调用抽象父类的register方法，然后又会去调用模板execute方法
        return execute(new RequestExecutor<Void>() {
            @Override
            public EurekaHttpResponse<Void> execute(EurekaHttpClient delegate) {
                // 4.6、多态调用RetryableEurekaHttpClient#execute
                // 4.11、多态调用RedirectingEurekaHttpClient#execute
                // 4.15、多态调用MetricsCollectingEurekaHttpClient#execute
                return delegate.register(info);
            }

            @Override
            public RequestType getRequestType() {
                return RequestType.Register;
            }
        });
    }
}
```

###### 3、多态调用 SessionedEurekaHttpClient#execute

```java
public class SessionedEurekaHttpClient extends EurekaHttpClientDecorator {
    @Override
    protected <R> EurekaHttpResponse<R> execute(RequestExecutor<R> requestExecutor) {
        long now = System.currentTimeMillis();
        long delay = now - lastReconnectTimeStamp;
        
        // 4.4、更新session时间currentSessionDurationMs
        if (delay >= currentSessionDurationMs) {
            logger.debug("Ending a session and starting anew");
            lastReconnectTimeStamp = now;
            currentSessionDurationMs = randomizeSessionDuration(sessionDurationMs);
            TransportUtils.shutdown(eurekaHttpClientRef.getAndSet(null));
        }

        ...
        
        // 4.5、调用装饰者requestExecutor#execute
        // 4.23、响应成功，则直接返回结果
        return requestExecutor.execute(eurekaHttpClient);
    }
}
```

###### 4、多态调用 RetryableEurekaHttpClient#execute

```java
public class RetryableEurekaHttpClient extends EurekaHttpClientDecorator {
    @Override
    protected <R> EurekaHttpResponse<R> execute(RequestExecutor<R> requestExecutor) {
        ...
        // 4.9、默认最大重试3次
        for (int retry = 0; retry < numberOfRetries; retry++) {
            ...
            try {
                // 4.10、调用装饰者requestExecutor#execute
                EurekaHttpResponse<R> response = requestExecutor.execute(currentHttpClient);
                // 4.22、响应成功，设置当前客户端为调用成功的客户端，然后返回响应结果
                if (serverStatusEvaluator.accept(response.getStatusCode(), requestExecutor.getRequestType())) {
                    delegate.set(currentHttpClient);
                    if (retry > 0) {
                        logger.info("Request execution succeeded on retry #{}", retry);
                    }
                    return response;
                }
                logger.warn(...);
            } catch (Exception e) {
                logger.warn(...);
            }

            // 如果超过了最大重试次数，仍没调用成功，则会来到这里，则设置调用成功的客户端为null，代表没有调用成功过
            delegate.compareAndSet(currentHttpClient, null);
            // 然后把当前currentEndpoint加入失败列表
            if (currentEndpoint != null) {
                quarantineSet.add(currentEndpoint);
            }
        }
        throw new TransportException("Retry limit reached; giving up on completing the request");
    }
}
```

5、多态调用 RedirectingEurekaHttpClient#execute

```java
public class RedirectingEurekaHttpClient extends EurekaHttpClientDecorator {
    @Override
    protected <R> EurekaHttpResponse<R> execute(RequestExecutor<R> requestExecutor) {
        ...
        // 4.14、底层调用装饰者requestExecutor#execute
     	EurekaHttpResponse<R> response = executeOnNewServer(requestExecutor, currentEurekaClientRef);
        ...
        // 4.21、响应成功，无需重定向
        return response;
        ...
    }
}
```

###### 6、多态调用 MetricsCollectingEurekaHttpClient#execute

```java
public class MetricsCollectingEurekaHttpClient extends EurekaHttpClientDecorator {
    @Override
    protected <R> EurekaHttpResponse<R> execute(RequestExecutor<R> requestExecutor) {
        ...
        // 4.18、最后调用AbstractJerseyEurekaHttpClient#register方法
        EurekaHttpResponse<R> httpResponse = requestExecutor.execute(delegate);
        // 4.20、响应成功，记录调用结果
        requestMetrics.countersByStatus.get(mappedStatus(httpResponse)).increment();
        return httpResponse;
    }
}
```

###### 7、最后调用 AbstractJerseyEurekaHttpClient#register

```java
public abstract class AbstractJerseyEurekaHttpClient implements EurekaHttpClient {
    @Override
    public EurekaHttpResponse<Void> register(InstanceInfo info) {
        // urlPath => apps/EUREKA-CLIENT
        String urlPath = "apps/" + info.getAppName();
        ClientResponse response = null;
        try {
            Builder resourceBuilder = jerseyClient.resource(serviceUrl).path(urlPath).getRequestBuilder();
            addExtraHeaders(resourceBuilder);
            
            // 4.19、正式发起post请求进行服务注册 => 这里返回NO_CONTENT(204, "No Content"),代表服务注册成功（此时在Eureka Portal中已经能看到该服务实例了）
            response = resourceBuilder
                    .header("Accept-Encoding", "gzip")
                    .type(MediaType.APPLICATION_JSON_TYPE)
                    .accept(MediaType.APPLICATION_JSON)
                    .post(ClientResponse.class, info);
            return anEurekaHttpResponse(response.getStatus()).headers(headersOf(response)).build();
        } finally {
            if (logger.isDebugEnabled()) {
                logger.debug(...);
            }
            if (response != null) {
                response.close();
            }
        }
    }
}
```

##### 响应请求 | Eureka Server

###### 1、请求分发到 addInstance 接口

```java
// jersey server resource，相当于MVC中的Controller
@Produces({"application/xml", "application/json"})
public class ApplicationResource {
    // 1、服务注册接口，相当于MVC中的RequestMapping
    @POST
    @Consumes({"application/json", "application/xml"})
    public Response addInstance(InstanceInfo info,
                                @HeaderParam(PeerEurekaNode.HEADER_REPLICATION) String isReplication) {
        ...// 一大堆校验
        // 2、调用注册方法，然后返回204给Eureka Client
        registry.register(info, "true".equals(isReplication));
        return Response.status(204).build();
    }
}
```

###### 2、调用 register，开始注册

```java
public class InstanceRegistry extends PeerAwareInstanceRegistryImpl
implements ApplicationContextAware {
	@Override
	public void register(final InstanceInfo info, final boolean isReplication) {
		...// 获取租约过期时间（默认为90s），打印日志等
		// 3、注册有关InstanceInfo信息，并复制将此信息发送给所有对等的Eureka节点，但如果这是来自其他副本节点的，那么不会再复制 => 这里是服务注册，所以isReplication为false
		super.register(info, isReplication);
	}
}

@Singleton
public class PeerAwareInstanceRegistryImpl extends AbstractInstanceRegistry implements PeerAwareInstanceRegistry {
    @Override
    public void register(final InstanceInfo info, final boolean isReplication) {
    	...// 获取租约过期时间（默认为90s）
        // 4、注册具有给定持续时间的新实例
        super.register(info, leaseDuration, isReplication);
        replicateToPeers(Action.Register, info.getAppName(), info.getId(), info, null, isReplication);
    }
}
```

###### 3、执行注册 & 写入服务实例

```java
public abstract class AbstractInstanceRegistry implements InstanceRegistry {
	// 服务实例ID instance-id="host：appnNme:port"
	// Eureka Server服务列表双重Map缓存=>{"appName"：{instance-id:InstanceInfo}}
    private final ConcurrentHashMap<String, Map<String, Lease<InstanceInfo>>> registry = new ConcurrentHashMap<String, Map<String, Lease<InstanceInfo>>>();
            
	// 注册具有给定持续时间的新实例
    public void register(InstanceInfo registrant, int leaseDuration, boolean isReplication) {
        try {
        	// 5、获取读锁
            read.lock();
            
            // 6、如果为第一次注册，则创建appName：空Map（服务实例的状态变化时，Server可能会收到多次，来自同一个实例的注册请求）
            Map<String, Lease<InstanceInfo>> gMap = registry.get(registrant.getAppName());
            REGISTER.increment(isReplication);
            if (gMap == null) {
                final ConcurrentHashMap<String, Lease<InstanceInfo>> gNewMap = new ConcurrentHashMap<String, Lease<InstanceInfo>>();
                gMap = registry.putIfAbsent(registrant.getAppName(), gNewMap);
                if (gMap == null) {
                    gMap = gNewMap;
                }
            }
            
            // 如果已经有租约，则保留最后一个脏时间戳而不覆盖它
            Lease<InstanceInfo> existingLease = gMap.get(registrant.getId());
            if (existingLease != null && (existingLease.getHolder() != null)) {
            	// 7、如果双重Map缓存中的lastDirtyTimestamp比新传过来的lastDirtyTimestamp还要大/新，说明注册请求的时间落后，则继续使用缓存而不发生替换
                ...
            } else {
                // 8、租约不存在，因此是新注册
                synchronized (lock) {
                    if (this.expectedNumberOfClientsSendingRenews > 0) {
                        // 9、由于客户端要注册，增加客户端发送更新的数量
                        this.expectedNumberOfClientsSendingRenews = this.expectedNumberOfClientsSendingRenews + 1;
                        // 10、更新每分钟最少收到租约阈值=实例数 * （60/续租周期）* 0.85
                        updateRenewsPerMinThreshold();
                    }
                }
                logger.debug(...);
            }
            
            // 9、如果租约不存在，或者租约是最新的租约，则设置或者更新到双重Map缓存中
            Lease<InstanceInfo> lease = new Lease<InstanceInfo>(registrant, leaseDuration);
            if (existingLease != null) {
                lease.setServiceUpTimestamp(existingLease.getServiceUpTimestamp());
            }
            gMap.put(registrant.getId(), lease);
            
            // 10、加入最新注册队列中
            synchronized (recentRegisteredQueue) {
                recentRegisteredQueue.add(new Pair<Long, String>(
                        System.currentTimeMillis(),
                        registrant.getAppName() + "(" + registrant.getId() + ")"));
            }
           
			...// 更新租约等状态
			// 11、更新租约最后更新时间
            registrant.setLastUpdatedTimestamp();
            // 12、清空对应服务列表的读写缓存，在下次只读缓存与读写缓存同步时，会触发一次读写缓存的加载，由于该服务已重新加入服务列表，所以只读缓存也重新加入它，不过有一定的延迟（默认30s同步一次）
            invalidateCache(registrant.getAppName(), registrant.getVIPAddress(), registrant.getSecureVipAddress());
            logger.info(...);
        } finally {
        	// 13、释放读锁
            read.unlock();
        }
    }
}
```

#### 服务发现原理

##### 概念

服务发现分为两种实现方式，分别是客户端和服务端的服务发现，而 Eureka 正是客户端实现的服务发现。

###### 客户端服务发现

![1639056166351](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1639056166351.png)

- **优点**：客户端知道所有可用服务的实际网络地址，可以非常方便的实现负载均衡功能。
- **缺点**：语言耦合性很强，针对不同的语言，每个服务的客户端都得实现一套服务发现的功能。

###### 服务端服务发现

![1639056226782](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1639056226782.png)

- **优点**：服务发现逻辑对客户端是透明的，客户端只管向 LOAD BALANCER 负载均衡器发送请求即可。
- **缺点**：需要关心 LOAD BALANCER 负载均衡器组件的高可用。

##### 发起请求 | Eureka Client

###### 1、调用服务列表拉取1 | 构建 DiscoveryClient 时

```java
@Singleton
public class CloudEurekaClient extends DiscoveryClient {
    @Inject
    DiscoveryClient(ApplicationInfoManager applicationInfoManager, EurekaClientConfig config, AbstractDiscoveryClientOptionalArgs args,
                    Provider<BackupRegistry> backupRegistryProvider) {
        ...
        // 1、根据fetch-registry配置参数，增量拉取拉取注册表信息
        if (clientConfig.shouldFetchRegistry() && !fetchRegistry(false)) {
            // 2、 由于拉取没发生异常，所以不会返回false，这里也就不会进来了
            fetchRegistryFromBackup();
        }
        ...
    }
}
```

###### 2、调用服务列表拉取2 | 默认30s 定时拉取

| 配置                                               | 默认 | 说明                                |
| -------------------------------------------------- | ---- | ----------------------------------- |
| eureka.shouldFetchRegistry                         | true | Eureka Client 是否需要拉取服务列表  |
| eureka.client.refresh.interval                     | 30s  | 服务列表拉取周期                    |
| eureka.client.cacheRefresh.exponentialBackOffBound | 10倍 | 10 * 30，表示最大的服务拉取周期乘数 |

```java
@Singleton
public class DiscoveryClient implements EurekaClient {
    // 1、启动超时时间30s，最大超时300s，延迟30s执行，交由cacheRefreshExecutor线程池处理的，以指定的时间间，隔获取注册表信息任务（服务发现）
    class CacheRefreshThread implements Runnable {
        public void run() {
            refreshRegistry();
        }
    }
    @VisibleForTesting
    void refreshRegistry() {
        ...
        // 2、如果currentRemoteRegions不等于latestRemoteRegions，说明region发生了变化，则remoteRegionsModified=true，代表进行全量拉取；否则，默认按增量拉取
    	boolean success = fetchRegistry(remoteRegionsModified);
        ...
    }
}
```

###### 3、服务列表拉取

```java
@Singleton
public class DiscoveryClient implements EurekaClient {
    // 1、获取注册表信息，除非在协调Eureka服务器和客户端注册表信息时出现问题，否则，此方法尝试仅在第一次获取后获取增量，forceFullRegistryFetch 强制获取完整的注册表
    private boolean fetchRegistry(boolean forceFullRegistryFetch) {
        Stopwatch tracer = FETCH_REGISTRY_TIMER.start();

        try {
            Applications applications = getApplications();

            if (// 2、是否禁用了增量拉取，默认为false
                clientConfig.shouldDisableDelta() 
                // 3、是否关注某个VIP地址，默认为否
                ||                 (!Strings.isNullOrEmpty(clientConfig.getRegistryRefreshSingleVipAddress()))
                	// 4、是否需要拉取全量服务列表，默认为false
                    || forceFullRegistryFetch
                	// 5、apps是否为null，默认不为null
                    || (applications == null)
                	// 6、apps是否为空，注册时的拉取确实为空，定时拉取的一般不为空
                    || (applications.getRegisteredApplications().size() == 0)
                	// 7、客户端应用程序没有支持增量的最新库，默认支持
                    || (applications.getVersion() == -1))
            {
                ...// 日志打印
                // 8、全量拉取服务列表
                getAndStoreFullRegistry();
            } else {
                // 9、增量拉取服务列表
                getAndUpdateDelta(applications);
            }
            ...
        } catch (Throwable e) {
            logger.error(...);
            // 10、服务列表拉取异常，则返回false
            return false;
        } finally {
            if (tracer != null) {
                tracer.stop();
            }
        }

        // 11、更新实例远程状态前，通知缓存刷新
        onCacheRefreshed();

        // 12、根据缓存中保存的刷新数据，更新远程状态
        updateInstanceRemoteStatus();

        // 13、注册表已成功获取，因此返回 true
        return true;
    }
}
```

###### 4、全量拉取服务列表

```java
@Singleton
public class DiscoveryClient implements EurekaClient {
    // 全量拉取注册表信息
    private void getAndStoreFullRegistry() throws Throwable {
        ...     
        EurekaHttpResponse<Applications> httpResponse = 
            // 1、先判断是否对某个VIP地址感兴趣，这里没有配置
            clientConfig.getRegistryRefreshSingleVipAddress() == null?
            // 2、则拉取Regions的注册表，但这里没有任何已注册的信息
            eurekaTransport.queryClient.getApplications(remoteRegionsRef.get()): eurekaTransport.queryClient.getVip(clientConfig.getRegistryRefreshSingleVipAddress(), remoteRegionsRef.get());
        ...
        // 6、设置为本地服务列表
        localRegionApps.set(this.filterAndShuffle(apps));
        ...
    }   
}

public abstract class AbstractJerseyEurekaHttpClient implements EurekaHttpClient {
    // 4、最后实际调用AbstractJerseyEurekaHttpClient#getApplications方法，中间经历的代理跟注册的类似：
    // 1) 多态调用SessionedEurekaHttpClient#execute
    // 2) 多态调用RetryableEurekaHttpClient#execute
    // 3) 多态调用RedirectingEurekaHttpClient#execute
    // 4) 多态调用MetricsCollectingEurekaHttpClient#execute
    @Override
    public EurekaHttpResponse<Applications> getApplications(String... regions) {
        return getApplicationsInternal("apps/", regions);
    }
    
    private EurekaHttpResponse<Applications> getApplicationsInternal(String urlPath, String[] regions) {
		...
        try {
            // serviceUrl=http://localhost:20000/eureka/，urlPath=apps/
            WebResource webResource = jerseyClient.resource(serviceUrl).path(urlPath);
            ...
            // 5、真正发起请求 => 响应 Client response status: 200
            response = requestBuilder.accept(MediaType.APPLICATION_JSON_TYPE).get(ClientResponse.class);
			...
        } finally {
            ...
        }
    }
}
```

###### 5、增量拉取服务列表

```java
@Singleton
public class DiscoveryClient implements EurekaClient {
    private void getAndUpdateDelta(Applications applications) throws Throwable {
        ...
        // 1、增量拉取Regions的注册表
        EurekaHttpResponse<Applications> httpResponse = eurekaTransport.queryClient.getDelta(remoteRegionsRef.get());
        ...// 5、如果拉取结果为null，还会全量去拉取一遍
        // 6、根据增量更新本地服务列表
        updateDelta(delta);
        ...
    }
}

public abstract class AbstractJerseyEurekaHttpClient implements EurekaHttpClient {
    // 2、最后实际调用AbstractJerseyEurekaHttpClient#getDelta方法，中间经历的代理跟注册的类似：
    // 1) 多态调用SessionedEurekaHttpClient#execute
    // 2) 多态调用RetryableEurekaHttpClient#execute
    // 3) 多态调用RedirectingEurekaHttpClient#execute
    // 4) 多态调用MetricsCollectingEurekaHttpClient#execute
    @Override
    public EurekaHttpResponse<Applications> getDelta(String... regions) {
        return getApplicationsInternal("apps/delta", regions);
    }
    
    private EurekaHttpResponse<Applications> getApplicationsInternal(String urlPath, String[] regions) {
		...
        try {
            // serviceUrl=http://localhost:20000/eureka/，urlPath=apps/delta
            WebResource webResource = jerseyClient.resource(serviceUrl).path(urlPath);
            ...
            // 4、真正发起请求 => 响应 Client response status: 200
            response = requestBuilder.accept(MediaType.APPLICATION_JSON_TYPE).get(ClientResponse.class);
			...
        } finally {
            ...
        }
    }
}
```

##### 响应请求 | Eureka Server

###### 1、全量更新接口

```java
@Path("/{version}/apps")
@Produces({"application/xml", "application/json"})
public class ApplicationsResource {
    @GET
    public Response getContainers(@PathParam("version") String version,
                                  @HeaderParam(HEADER_ACCEPT) String acceptHeader,
                                  @HeaderParam(HEADER_ACCEPT_ENCODING) String acceptEncoding,
                                  @HeaderParam(EurekaAccept.HTTP_X_EUREKA_ACCEPT) String eurekaAccept,
                                  @Context UriInfo uriInfo,
                                  @Nullable @QueryParam("regions") String regionsStr) {
        ...
        Response response;
        if (acceptEncoding != null && acceptEncoding.contains(HEADER_GZIP_VALUE)) {
            // 1、从缓存中获取gzip部分（见两级缓存原理），eg：entity_name=ALL_APPS，hash_key=ApplicationALL_APPSJSONV2full
            response = Response.ok(responseCache.getGZIP(cacheKey))
                    .header(HEADER_CONTENT_ENCODING, HEADER_GZIP_VALUE)
                    .header(HEADER_CONTENT_TYPE, returnMediaType)
                    .build();
        } else {
            // 2、否则直接从缓存中获取
            response = Response.ok(responseCache.get(cacheKey))
                    .build();
        }
        return response;
    }
}
```

###### 2、增量拉取接口

```java
@Path("/{version}/apps")
@Produces({"application/xml", "application/json"})
public class ApplicationsResource {
    @Path("delta")
    @GET
    public Response getContainerDifferential(
            @PathParam("version") String version,
            @HeaderParam(HEADER_ACCEPT) String acceptHeader,
            @HeaderParam(HEADER_ACCEPT_ENCODING) String acceptEncoding,
            @HeaderParam(EurekaAccept.HTTP_X_EUREKA_ACCEPT) String eurekaAccept,
        @Context UriInfo uriInfo, @Nullable @QueryParam("regions") String regionsStr) {
        if (acceptEncoding != null
                && acceptEncoding.contains(HEADER_GZIP_VALUE)) {
            // 1、从缓存中获取gzip部分（见两级缓存原理），eg：entity_name=ALL_APPS_DELTA，hash_key=ApplicationALL_APPS_DELTAJSONV2full
            return Response.ok(responseCache.getGZIP(cacheKey))
                    .header(HEADER_CONTENT_ENCODING, HEADER_GZIP_VALUE)
                    .header(HEADER_CONTENT_TYPE, returnMediaType)
                    .build();
        } 
        // 2、否则直接从缓存中获取
        else {
            return Response.ok(responseCache.get(cacheKey))
                    .build();
        }
    }
}
```

#### 服务列表缓存原理

![1638705242665](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1638705242665.png)

##### 两级缓存机制 | Eureka Server

###### 概念

1. 服务注册到注册中⼼后，服务实例信息存储在 `AbstractInstanceRegistry#registry` 中，即内存中。
2. 但 Eureka 为了提⾼响应速度，在内部做了优化，加⼊了两层的缓存结构，将 Client 需要的实例信息，直接缓存起来，获取时直接从缓存中拿数据，然后响应给 Client。 
   1. 第⼀层缓存是 `ResponseCacheImpl#readOnlyCacheMap`，采⽤ ConcurrentHashMap 来存储数据，定时与 ` ResponseCacheImpl#readWriteCacheMap` 进⾏数据同步，默认同步时间为 30s ⼀次。
   2. 第⼆层缓存是 `ResponseCacheImpl#readWriteCacheMap`，采⽤ Guava 来实现缓存，缓存过期时间默认为 180s，当服务下线、过期、注册、状态变更等操作，都会清除该缓存中的数据。
   3. 如果两级缓存都无法查询，则会触发 Guava 缓存的加载 `CacheLoader#load`，从存储层  `AbstractInstanceRegistry#registry`  拉取数据到二级缓存中，然后再返回给 Client。

###### 优点

两级缓存机制，提⾼了 Eureka Server 的响应速度，避免了同时读写 `registery` 内存造成的并发冲突问题。

###### 缺点

两级缓存机制，会导致数据⼀致性很薄弱，造成 Eureka Client 获取不到**最新**的服务实例信息（**AP 模型**），⽆法快速发现**新的服务和已下线的服务**。

1. 新服务上线后，服务消费者不能**立即访问**到刚上线的新服务，需要过⼀段时间后才能访问。
2. 或者服务下线后，服务还是会被调⽤到，⼀段时候后才彻底停⽌服务，访问前期会导致**频繁的报错**。

###### 解决方案

- **针对只读缓存**：
  1. 缩短更新时间 `eureka.server.responseCacheUpdateIntervalMs`，让服务发现变得更加及时。
  2. 或者直接关闭 `eureka.server.useReadOnlyResponseCache=false`。
- **针对读写缓存**：
  1. 缩短过期时间 `eureka.server.responseCacheAutoExpirationInSeconds`。
  2. 缩短服务剔除周期 `eureka.server.evictionIntervalTimerInMs`，让服务下线后，能够及时把其从注册表中清除。

###### 相关配置

| 配置                                               | 默认  | 说明                                                         |
| -------------------------------------------------- | ----- | ------------------------------------------------------------ |
| eureka.server.useReadOnlyResponseCache             | true  | true 代表 Eureka Client 会从 `readOnlyCacheMap` 更新数据，而 false 则代表跳过 `readOnlyCacheMap`，直接从`readWriteCacheMap` 中更新 |
| eureka.server.responseCacheUpdateIntervalMs        | 30000 | `readWriteCacheMap` 更新至`readOnlyCacheMap` 的时间周期      |
| eureka.server.responseCacheAutoExpirationInSeconds | 180   | `readWriteCacheMap` Guava 缓存的过期时间                     |
| eureka.server.evictionIntervalTimerInMs            | 60000 | 清理未续约节点的服务剔除周期                                 |

###### 源码分析 - registry

服务注册时添加，服务剔除时删除。

```java
public abstract class AbstractInstanceRegistry implements InstanceRegistry {
	// 服务实例ID instance-id="host：appnNme:port"
	// Eureka Server服务列表双重Map缓存=>{"appName"：{instance-id:InstanceInfo}}
    private final ConcurrentHashMap<String, Map<String, Lease<InstanceInfo>>> registry = new ConcurrentHashMap<String, Map<String, Lease<InstanceInfo>>>();
            
	// 1、服务注册，注册具有给定持续时间的新实例
    public void register(InstanceInfo registrant, int leaseDuration, boolean isReplication) {
        try {
            // 如果为第一次注册，则创建appName：空Map（服务实例的状态变化时，Server可能会收到多次，来自同一个实例的注册请求）
            Map<String, Lease<InstanceInfo>> gMap = registry.get(registrant.getAppName());
            REGISTER.increment(isReplication);
            if (gMap == null) {
                final ConcurrentHashMap<String, Lease<InstanceInfo>> gNewMap = new ConcurrentHashMap<String, Lease<InstanceInfo>>();
                gMap = registry.putIfAbsent(registrant.getAppName(), gNewMap);
                if (gMap == null) {
                    gMap = gNewMap;
                }
            }
        }
    }
    
    // 2、服务剔除：剔除已过期的服务实例
    protected boolean internalCancel(String appName, String id, boolean isReplication) {
        Map<String, Lease<InstanceInfo>> gMap = registry.get(appName);
        Lease<InstanceInfo> leaseToCancel = null;
        if (gMap != null) {
            leaseToCancel = gMap.remove(id);
        }
    }
}
```

###### 源码分析 - readWriteCacheMap

自动装配时构造，一级缓存找不到时加载，从 `registry` 或者 `recentlyChangedQueue` 中加载。

```java
...
// 1、自动装配时构造
public class EurekaServerAutoConfiguration extends WebMvcConfigurerAdapter {
    ....
	// Eureka服务端上下文——EurekaServerContext
	@Bean
	@ConditionalOnMissingBean
	public EurekaServerContext eurekaServerContext(ServerCodecs serverCodecs, PeerAwareInstanceRegistry registry, PeerEurekaNodes peerEurekaNodes) {
		return new DefaultEurekaServerContext(this.eurekaServerConfig, serverCodecs,
				registry, peerEurekaNodes, this.applicationInfoManager);
	}
    ...
}

@Singleton
public class DefaultEurekaServerContext implements EurekaServerContext {
    // 2、自动装配时构造
    @PostConstruct
    @Override
    public void initialize() {
        ...
        registry.init(peerEurekaNodes);
		...
    }
}

@Singleton
public class PeerAwareInstanceRegistryImpl extends AbstractInstanceRegistry implements PeerAwareInstanceRegistry {
    
    // 2、自动装配时构造
    @Override
    public void init(PeerEurekaNodes peerEurekaNodes) throws Exception {
		...
        initializedResponseCache();
        ...
    }
}

public abstract class AbstractInstanceRegistry implements InstanceRegistry {
    // 3、自动装配时构造
    @Override
    public synchronized void initializedResponseCache() {
        if (responseCache == null) {
            responseCache = new ResponseCacheImpl(serverConfig, serverCodecs, this);
        }
    }
}

public class ResponseCacheImpl implements ResponseCache {
    ...
    // 读写缓存
    private final LoadingCache<Key, Value> readWriteCacheMap;
    
    // 4、自动装配时构造
    ResponseCacheImpl(EurekaServerConfig serverConfig, ServerCodecs serverCodecs, AbstractInstanceRegistry registry) {
        ...
        this.readWriteCacheMap = 
            CacheBuilder.newBuilder()
            .initialCapacity(serverConfig.getInitialCapacityOfResponseCache())
            .expireAfterWrite(serverConfig.getResponseCacheAutoExpirationInSeconds(), TimeUnit.SECONDS)
            .removalListener(new RemovalListener<Key, Value>() {...})
            .build(new CacheLoader<Key, Value>() {
                // 5、一级缓存找不到时加载
                @Override
                public Value load(Key key) throws Exception {
                    if (key.hasRegions()) {
                        Key cloneWithNoRegions = key.cloneWithoutRegions();
                        regionSpecificKeys.put(cloneWithNoRegions, key);
                    }
                    // 6、从registry/recentlyChangedQueue中加载
                    Value value = generatePayload(key);
                    return value;
                }
            });
        ...
    }
    
    private Value generatePayload(Key key) {
        ...
        if (ALL_APPS.equals(key.getName())) {
            ...
            // 7、全量拉取时，从registry中加载
            payload = getPayLoad(key, registry.getApplications());
        } else if (ALL_APPS_DELTA.equals(key.getName())) {
            ...
            // 8、全量拉取时，从recentlyChangedQueue中加载
            payload = getPayLoad(key, registry.getApplicationDeltas());
        } else {
            ...
        }
        ...
    }
}
```

###### 源码分析 - readOnlyCacheMap

自动装配时构造，定时同步读写缓存，服务发现时更新（有延迟）。

```java
public class ResponseCacheImpl implements ResponseCache {
    ...
    // 1、只读缓存，实例变量，自动装配时构造
    private final ConcurrentMap<Key, Value> readOnlyCacheMap = new ConcurrentHashMap<Key, Value>();
    
    ResponseCacheImpl(EurekaServerConfig serverConfig, ServerCodecs serverCodecs, AbstractInstanceRegistry registry) {
        ...
        if (shouldUseReadOnlyResponseCache) {
            // 2、定时同步读写缓存，默认延后30秒执行，执行周期为30s
            timer.schedule(getCacheUpdateTask(),
                    new Date(((System.currentTimeMillis() / responseCacheUpdateIntervalMs) * responseCacheUpdateIntervalMs)
                            + responseCacheUpdateIntervalMs),
                    responseCacheUpdateIntervalMs);
        }
        ...
    }
    
    private TimerTask getCacheUpdateTask() {
        return new TimerTask() {
            @Override
            public void run() {
                ...
                for (Key key : readOnlyCacheMap.keySet()) {
                    ...
                    try {
                        CurrentRequestVersion.set(key.getVersion());
                        Value cacheValue = readWriteCacheMap.get(key);
                        Value currentCacheValue = readOnlyCacheMap.get(key);
                        // 3、定时同步读写缓存
                        if (cacheValue != currentCacheValue) {
                            readOnlyCacheMap.put(key, cacheValue);
                        }
                    } catch (Throwable th) {
                        ...
                    }
                }
            }
        };
    }
    
    // 4、服务发现时更新（有延迟）：获取有关应用程序的压缩信息
    public byte[] getGZIP(Key key) {
        // 5、是否读取只读缓存，默认开启
        Value payload = getValue(key, shouldUseReadOnlyResponseCache);
        if (payload == null) {
            return null;
        }
        // 9、只返回gzip部分
        return payload.getGzipped();
    }
    
    @VisibleForTesting
    Value getValue(final Key key, boolean useReadOnlyCache) {
        Value payload = null;
        try {
            // 6、如果启动用了读取只读缓存，则先从只读缓存中获取（ConcurrentHashMap）
            if (useReadOnlyCache) {
                final Value currentPayload = readOnlyCacheMap.get(key);
                if (currentPayload != null) {
                    payload = currentPayload;
                } 
                // 7、如果只读缓存中获取不到，则还会从读写缓存中获取（com.google.common.cache.LoadingCache）
                else {
                    payload = readWriteCacheMap.get(key);
                    readOnlyCacheMap.put(key, payload);
                }
            }
            // 8、否则直接从读写缓存中获取（com.google.common.cache.LoadingCache）
            else {
                payload = readWriteCacheMap.get(key);
            }
        } catch (Throwable t) {
            ...
        }
        return payload;
    }
}
```

##### 本地内存缓存 | Eureka Client

###### 全量更新

```java
@Singleton
public class DiscoveryClient implements EurekaClient {
    
    // 本地服务列表内存缓存
    private final AtomicReference<Applications> localRegionApps = new AtomicReference<Applications>();
    
    // 1、全量更新
	private void getAndStoreFullRegistry() throws Throwable {
        ...// 2、服务列表全量拉取
        if (apps == null) {
            ...
        } 
        // 3、CAS更新服务列表版本
        else if (fetchRegistryGeneration.compareAndSet(currentUpdateGeneration, currentUpdateGeneration + 1)) {
            // 4、去除非UP服务列表，然后设置到本地内存缓存
            localRegionApps.set(this.filterAndShuffle(apps));
            ...
        } else {
            ...
        }
    }
}  
```

###### 增量更新

```java
@Singleton
public class DiscoveryClient implements EurekaClient {
    
    // 本地服务列表内存缓存
    private final AtomicReference<Applications> localRegionApps = new AtomicReference<Applications>();
    
    // 5、增量更新
    private void getAndUpdateDelta(Applications applications) throws Throwable {
        ...
        // 6、服务列表增量拉取
        if (delta == null) {
            ...
            // 7、如果结果为null，则还要重新发起全量拉取
            getAndStoreFullRegistry();
        } 
        // 8、CAS更新服务列表版本
        else if (fetchRegistryGeneration.compareAndSet(currentUpdateGeneration, currentUpdateGeneration + 1)) {
            ...
            if (fetchRegistryUpdateLock.tryLock()) {
                try {
                    // 9、去除非UP服务列表
                    updateDelta(delta);
                    reconcileHashCode = getReconcileHashCode(applications);
                } finally {
                    fetchRegistryUpdateLock.unlock();
                }
            } else {
                ...
            }
            // 10、服务列表的hashcode不同，则还要更新服务列表
            if (!reconcileHashCode.equals(delta.getAppsHashCode()) || clientConfig.shouldLogDeltaDiff()) {
                reconcileAndLogDifference(delta, reconcileHashCode);
            }
        } else {
            ...
        }
    }
    
    private void reconcileAndLogDifference(Applications delta, String reconcileHashCode) throws Throwable {
        // ... 11、服务列表的hashcode不同，则再全量拉取一次，用于更新服务列表
        // 12、CAS更新服务列表版本
        if (fetchRegistryGeneration.compareAndSet(currentUpdateGeneration, currentUpdateGeneration + 1)) {
            // 13、设置到本地内存缓存
            localRegionApps.set(this.filterAndShuffle(serverApps));
            ..
        } else {
            ...
        }
    }
}
```

##### 服务调用缓存 | Ribbon

LoadBalancerClient 每选择一次不同的服务，则 RibbonClientConfiguration 就会构造一次，触发 updateAction#doUpdate 的服务调用缓存更新，详情见《服务调用原理 - 6、服务调用缓存实现》。

| 配置                                        | 默认 | 说明                                                       |
| ------------------------------------------- | ---- | ---------------------------------------------------------- |
| xxxService.ribbon.serverListRefreshInterval | 30s  | Ribbon 服务调用缓存，与 Eureka Client 服务列表缓存同步周期 |

```java
public class PollingServerListUpdater implements ServerListUpdater {
    @Override
    public synchronized void start(final UpdateAction updateAction) {
        if (isActive.compareAndSet(false, true)) {
            final Runnable wrapperRunnable = new Runnable() {
                @Override
                public void run() {
                    // 2、每次执行updateAction#doUpdate
                   	updateAction.doUpdate();
                }
            };
			
            // 1、延迟10s，默认周期为30s定时执行任务
            scheduledFuture = getRefreshExecutor().scheduleWithFixedDelay(
                    wrapperRunnable,
                    initialDelayMs,
                    refreshIntervalMs,
                    TimeUnit.MILLISECONDS
            );
        } else {
            logger.info(...);
        }
    }
}
```

#### 服务调用原理 | Eureka Client

##### 1、Eureka 依赖 Ribbon 

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
</dependency>
<dependency>
    <groupId>com.netflix.ribbon</groupId>
    <artifactId>ribbon-eureka</artifactId>
</dependency>
```

##### 2、Spring SPI 配置

```properties
# .../spring-cloud-netflix-eureka-client/2.1.1.RELEASE/spring-cloud-netflix-eureka-client-2.1.1.RELEASE.jar!/META-INF/spring.factories
# Ribbon负载均衡自动配置类
org.springframework.cloud.netflix.ribbon.eureka.RibbonEurekaAutoConfiguration,\
```

##### 3、客户端负载均衡自动装配

```java
@Configuration
@EnableConfigurationProperties
@ConditionalOnRibbonAndEurekaEnabled
// 1、客户端负载均衡，自动装配
@AutoConfigureAfter(RibbonAutoConfiguration.class)
// Eureka客户端负载均衡预处理器，自动装配
@RibbonClients(defaultConfiguration = EurekaRibbonClientConfiguration.class)
public class RibbonEurekaAutoConfiguration {

}

@Configuration
@Conditional(RibbonAutoConfiguration.RibbonClassesConditions.class)
@RibbonClients
// 3、在Eureka Client初始化之后，才注入当前类
@AutoConfigureAfter(name = "org.springframework.cloud.netflix.eureka.EurekaClientAutoConfiguration")
// 4、loadbalancer自动装配
@AutoConfigureBefore({ 
    LoadBalancerAutoConfiguration.class,
	AsyncLoadBalancerAutoConfiguration.class 
 })
// 5、读取Ribbon配置
@EnableConfigurationProperties({ RibbonEagerLoadProperties.class,
		ServerIntrospectorProperties.class })
public class RibbonAutoConfiguration {
    ...  
	@Bean
	public SpringClientFactory springClientFactory() {
        // 6、配置负载均衡客户端的上下文工厂
		SpringClientFactory factory = new SpringClientFactory();
		factory.setConfigurations(this.configurations);
		return factory;
	}

	@Bean
	@ConditionalOnMissingBean(LoadBalancerClient.class)
	public LoadBalancerClient loadBalancerClient() {
        // 9、构造LoadBalancerClient实例RibbonLoadBalancerClient
		return new RibbonLoadBalancerClient(springClientFactory());
	}
    ...
}

// 创建客户端、负载均衡器和客户端配置实例的工厂，为每个客户端名称创建一个Spring ApplicationContext，并从那里提取需要的bean
public class SpringClientFactory extends NamedContextFactory<RibbonClientSpecification> {
    ...
    // 7、调用父类构造器，RibbonClientConfiguration在每个负载均衡客户端Bean实例构造时注入
	public SpringClientFactory() {
		super(RibbonClientConfiguration.class, NAMESPACE, "ribbon.client.name");
	}
    ...
}

// 创建一组子上下文，允许一组规范定义每个子上下文中的 bean，从spring-cloud-netflix FeignClientFactory和SpringClientFactory移植，实现DisposableBean，代表工厂用于构造一次性Bean
public abstract class NamedContextFactory<C extends NamedContextFactory.Specification>
implements DisposableBean, ApplicationContextAware {
	// 8、上下文工厂构造器，RibbonClientConfiguration在每个负载均衡客户端Bean实例构造时注入
	public NamedContextFactory(Class<?> defaultConfigType, String propertySourceName,
			String propertyName) {
		this.defaultConfigType = defaultConfigType;
		this.propertySourceName = propertySourceName;
		this.propertyName = propertyName;
	}
}
```

##### 4、Controller 注入 LoadBalancerClient

```java
// 1、Eureka Consumer测试前端控制类
@RestController
@Slf4j
public class ConsumerController {
    @Autowired
    private LoadBalancerClient loadBalancerClient;
    @Autowired
    private RestTemplate restTemplate;
    
    /**
     * 测试消费Post服务
     * @return
     */
    @PostMapping("/hello")
    public Friend helloPost(){
        // 2、使用loadBalancerClient，指定服务名，选择一个具体的服务实例
        ServiceInstance instance = loadBalancerClient.choose("eureka-client");
        if(instance == null){
            return null;
        }
        
        // 3、根据服务名拉取服务信息
        String targetUrl = String.format("http://%s:%s/sayHi", instance.getHost(), instance.getPort());
        log.info("url is {}", targetUrl);

        Friend friendParam = new Friend();
        friendParam.setName("Eureka Consumer");

        // 4、根据IP+端口发起远程调用
        return restTemplate.postForObject(targetUrl, friendParam, Friend.class);
    }
}
```

##### 5、LoadBalancerClient 服务实例选择原理

```java
public class RibbonLoadBalancerClient implements LoadBalancerClient {
    // 1、指定服务名，选择一个具体的服务实例，serviceId="eureka-client"
	@Override
	public ServiceInstance choose(String serviceId) {
		return choose(serviceId, null);
	}
    
	public ServiceInstance choose(String serviceId, Object hint) {
        // 3、根据服务名选择一个具体的服务实例
		Server server = getServer(
            // 2、根据服务名选择一个具体的负载均衡器
            getLoadBalancer(serviceId), hint
        );
		if (server == null) {
			return null;
		}
        // 4、构造RibbonServer返回
		return new RibbonServer(serviceId, server, isSecure(server, serviceId),
				serverIntrospector(serviceId).getMetadata(server));
	}
    
	protected ILoadBalancer getLoadBalancer(String serviceId) {
        // 2.1、使用SpringClientFactory负载均衡客户端的上下文工厂，构造一个负载均衡器Bean实例
		return this.clientFactory.getLoadBalancer(serviceId);
	}

    protected Server getServer(ILoadBalancer loadBalancer, Object hint) {
		if (loadBalancer == null) {
			return null;
		}
		// 3.1、选择默认的服务实例
		return loadBalancer.chooseServer(hint != null ? hint : "default");
	}
}

public class ZoneAwareLoadBalancer<T extends Server> extends DynamicServerListLoadBalancer<T> {
    @Override
    public Server chooseServer(Object key) {
     if (!ENABLED.get() || getLoadBalancerStats().getAvailableZones().size() <= 1) {
            logger.debug(...);
         	// 3.2、调用父类方法，选择默认的服务实例
            return super.chooseServer(key);
            ...
        }
    }
}

public class BaseLoadBalancer extends AbstractLoadBalancer implements
    PrimeConnections.PrimeConnectionListener, IClientConfigAware {
    public Server chooseServer(Object key) {
        ...
        // 3.3、以循环方式，从过滤列表中返回服务实例
        return rule.choose(key);
        ...
    }
    
    // 3.5、返回只可读的服务调用缓存（缓存实现见第6点）
    @Override
    public List<Server> getAllServers() {
        return Collections.unmodifiableList(allServerList);
    }
}

public abstract class PredicateBasedRule extends ClientConfigEnabledRoundRobinRule {
    @Override
    public Server choose(Object key) {
        ILoadBalancer lb = getLoadBalancer();
        // 3.6、根据服务名过滤服务列表
        Optional<Server> server = getPredicate().chooseRoundRobinAfterFiltering(
            //3.4、使用ZoneAwareLoadBalancer获取所有服务列表（服务调用缓存）
            lb.getAllServers(), key
        );
        if (server.isPresent()) {
            // 3.7、返回最后选择的服务实例
            return server.get();
        } else {
            return null;
        }       
    }
}

public class SpringClientFactory extends NamedContextFactory<RibbonClientSpecification> {
    
    // 2.2、构造一个负载均衡器ILoadBalancer Bean实例
	public ILoadBalancer getLoadBalancer(String name) {
		return getInstance(name, ILoadBalancer.class);
	}
    
	@Override
	public <C> C getInstance(String name, Class<C> type) {
        // 2.3、构造一个负载均衡器ILoadBalancer Bean实例并返回
		C instance = super.getInstance(name, type);
		if (instance != null) {
			return instance;
		}
		IClientConfig config = getInstance(name, IClientConfig.class);
		return instantiateWithConfig(getContext(name), type, config);
	}
}

public abstract class NamedContextFactory<C extends NamedContextFactory.Specification>
implements DisposableBean, ApplicationContextAware {

	public <T> T getInstance(String name, Class<T> type) {
		// 2.4、根据服务名获取对应的上下文配置
		AnnotationConfigApplicationContext context = getContext(name);
		if (BeanFactoryUtils.beanNamesForTypeIncludingAncestors(context,
				type).length > 0) {
			// 2.7、Spring AbstractApplicationContext#getBean构造ILoadBalancer Bean实例ZoneAwareLoadBalancer
			return context.getBean(type);
		}
		return null;
	}
	
	protected AnnotationConfigApplicationContext getContext(String name) {
		if (!this.contexts.containsKey(name)) {
			synchronized (this.contexts) {
				if (!this.contexts.containsKey(name)) {
					// 2.5、第一次构造对应的上下文，然后服务名做key缓存起来
					this.contexts.put(name, createContext(name));
				}
			}
		}
		return this.contexts.get(name);
	}

	protected AnnotationConfigApplicationContext createContext(String name) {
		...
		// 2.6、底层调用Spring AbstractApplicationContext#refresh，触发RibbonClientConfiguration注入
		context.refresh();
		return context;
	}
}
```

##### 6、服务调用缓存实现

```java
// 1、RibbonClientConfiguration，每个服务名对应的服务均衡器构造时，都会重新refresh注入
@SuppressWarnings("deprecation")
@Configuration
@EnableConfigurationProperties
@Import({ HttpClientConfiguration.class, OkHttpRibbonConfiguration.class,
		RestClientRibbonConfiguration.class, HttpClientRibbonConfiguration.class })
public class RibbonClientConfiguration {
    ...
	@Bean
	@ConditionalOnMissingBean
	public ServerListUpdater ribbonServerListUpdater(IClientConfig config) {
        // 1、构造服务列表自动拉取的Updater
		return new PollingServerListUpdater(config);
	}
    
	@Bean
	@ConditionalOnMissingBean
	public ILoadBalancer ribbonLoadBalancer(IClientConfig config,
			ServerList<Server> serverList, ServerListFilter<Server> serverListFilter,
			IRule rule, IPing ping, ServerListUpdater serverListUpdater) {
		if (this.propertiesFactory.isSet(ILoadBalancer.class, name)) {
			return this.propertiesFactory.get(ILoadBalancer.class, config, name);
		}
        // 2、根据Updater构造负载均衡器实例ZoneAwareLoadBalancer
		return new ZoneAwareLoadBalancer<>(config, rule, ping, serverList,
				serverListFilter, serverListUpdater);
	}
	...
}

public class PollingServerListUpdater implements ServerListUpdater {
    // 1.1、调用父类构造器，设置延迟10s，默认周期为30s定时执行任务的参数
    this(LISTOFSERVERS_CACHE_UPDATE_DELAY, getRefreshIntervalMs(clientConfig));
}

public class PollingServerListUpdater implements ServerListUpdater {
     // 1.2、设置延迟10s，默认周期为30s定时执行任务的参数
    public PollingServerListUpdater(final long initialDelayMs, final long refreshIntervalMs) {
        this.initialDelayMs = initialDelayMs;
        this.refreshIntervalMs = refreshIntervalMs;
    }
}

public class ZoneAwareLoadBalancer<T extends Server> extends DynamicServerListLoadBalancer<T> {
    public ZoneAwareLoadBalancer(IClientConfig clientConfig, IRule rule, IPing ping, ServerList<T> serverList, ServerListFilter<T> filter, ServerListUpdater serverListUpdater) {
        // 2.1、调用父类构造器，启动调用服务缓存更新的定时任务
        super(clientConfig, rule, ping, serverList, filter, serverListUpdater);
    }
}

public class DynamicServerListLoadBalancer<T extends Server> extends BaseLoadBalancer {
     // DynamicServerListLoadBalancer构造器
     public DynamicServerListLoadBalancer(IClientConfig clientConfig, IRule rule, IPing ping, ServerList<T> serverList, ServerListFilter<T> filter, ServerListUpdater serverListUpdater) {
        ...
        // 2.2、启动调用服务缓存更新的定时任务
        restOfInit(clientConfig);
    }
    
    void restOfInit(IClientConfig clientConfig) {
        ...
        // 2.3、启动调用服务缓存更新的定时任务
        enableAndInitLearnNewServersFeature();
        
		// 2.8. 第一次不等定时执行，立即执行一次updateAction#doUpdate，同步EurekaClient服务实例缓存，到调用缓存中
        updateListOfServers();
        ...
    }

    public void enableAndInitLearnNewServersFeature() {
        LOGGER.info(...);
        // 2.4、启动调用服务缓存更新的定时任务
        serverListUpdater.start(updateAction);
    }
    
    // 2.7、定时任务成员变量
    protected final ServerListUpdater.UpdateAction updateAction = new ServerListUpdater.UpdateAction() {
        @Override
        public void doUpdate() {
            // 2.8. updateAction#doUpdate每次都同步EurekaClient服务实例缓存，到调用缓存中
            updateListOfServers();
        }
    };

    @VisibleForTesting
    public void updateListOfServers() {
        List<T> servers = new ArrayList<T>();
        if (serverListImpl != null) {
            // 2.9、拉取最新的EurekaClient服务实例缓存
            servers = serverListImpl.getUpdatedListOfServers();
            ...
        }
        // 2.14、更新服务调用缓存
        updateAllServerList(servers);
    }
    
    protected void updateAllServerList(List<T> ls) {
        if (serverListUpdateInProgress.compareAndSet(false, true)) {
            ...
            // 2.15、更新服务调用缓存
            setServersList(ls);
            ...
        }
    }
    
    @Override
    public void setServersList(List lsrv) {
        ...
        // 2.16、更新服务调用缓存
        super.setServersList(lsrv);
        ...
    }
}

public class BaseLoadBalancer extends AbstractLoadBalancer implements
    PrimeConnections.PrimeConnectionListener, IClientConfigAware {
    public void setServersList(List lsrv) {
        // 2.17、更新服务调用缓存
        ...
        allServerList = allServers;
        ...
    }
}

public class PollingServerListUpdater implements ServerListUpdater {
    @Override
    public synchronized void start(final UpdateAction updateAction) {
        if (isActive.compareAndSet(false, true)) {
            final Runnable wrapperRunnable = new Runnable() {
                @Override
                public void run() {
                    // 2.6、每次执行updateAction#doUpdate
                   	updateAction.doUpdate();
                }
            };
			
            // 2.5、延迟10s，默认周期为30s定时执行任务
            scheduledFuture = getRefreshExecutor().scheduleWithFixedDelay(
                    wrapperRunnable,
                    initialDelayMs,
                    refreshIntervalMs,
                    TimeUnit.MILLISECONDS
            );
        } else {
            logger.info(...);
        }
    }
}

public class DomainExtractingServerList implements ServerList<DiscoveryEnabledServer> {
	@Override
	public List<DiscoveryEnabledServer> getUpdatedListOfServers() {
		List<DiscoveryEnabledServer> servers = setZones(
            	// 2.10、拉取最新的EurekaClient服务实例缓存
				this.list.getUpdatedListOfServers());
		return servers;
	}
}

public class DiscoveryEnabledNIWSServerList extends AbstractServerList<DiscoveryEnabledServer>{
    @Override
    public List<DiscoveryEnabledServer> getUpdatedListOfServers(){
        // 2.11、拉取最新的EurekaClient服务实例缓存
        return obtainServersViaDiscovery();
    }
    
    private List<DiscoveryEnabledServer> obtainServersViaDiscovery() {
        ...
        // 2.12、获取EurekaClient中的服务实例缓存
        List<InstanceInfo> listOfInstanceInfo = eurekaClient.getInstancesByVipAddress(vipAddress, isSecure, targetRegion);
        ...
    }
}

@Singleton
public class DiscoveryClient implements EurekaClient {
    @Override
    public List<InstanceInfo> getInstancesByVipAddress(String vipAddress, boolean secure, @Nullable String region) {
        ...
        // 2.13、从原子变量中获取服务实例缓存
        applications = this.localRegionApps.get();
        ...
    }
}
```

#### 服务续约原理

#####  发起请求 | Eureka Client

###### 1、定时心跳检测 | 默认周期 30s

| 配置                                            | 默认 | 说明                                |
| ----------------------------------------------- | ---- | ----------------------------------- |
| eureka.instance.lease.renewalInterval           | 30s  | 服务实例续约周期                    |
| eureka.client.heartbeat.exponentialBackOffBound | 10倍 | 10 * 30，表示最大的服务续约周期乘数 |

```java
@Singleton
public class DiscoveryClient implements EurekaClient {
    // 1、非默认参数，启动续租超时时间5s，最大超时50s，延迟5s执行，交由heartbeatExecutor线程池处理的，在给定的时间间隔内，更新租约的心跳任务（心跳检测）
    private class HeartbeatThread implements Runnable {
        public void run() {
            if (renew()) {
                // 2、续约成功，则更新最后续约成功时间
                lastSuccessfulHeartbeatTimestamp = System.currentTimeMillis();
            }
        }
    }
}
```

###### 2、发送续约心跳包

```java
@Singleton
public class DiscoveryClient implements EurekaClient {
    boolean renew() {
        EurekaHttpResponse<InstanceInfo> httpResponse;
        try {
            // 1、发送续约心跳包
            httpResponse = eurekaTransport.registrationClient.sendHeartBeat(instanceInfo.getAppName(), instanceInfo.getId(), instanceInfo, null);
            logger.debug(...);
            // 2、如果响应结果为 NOT_FOUND(404, "Not Found")
            if (httpResponse.getStatusCode() == Status.NOT_FOUND.getStatusCode()) {
                REREGISTER_COUNTER.increment();
                logger.info(...);
                // 3、则设置isInstanceInfoDirty=true，刷新最后脏时间戳，并重新发起服务注册
                long timestamp = instanceInfo.setIsDirtyWithTime();
                boolean success = register();
                if (success) {
                    // 4、如果重新注册成功，则清除isInstanceInfoDirty（设置为false）
                    instanceInfo.unsetIsDirty(timestamp);
                }
                // 5、重新注册成功则返回true，否则返回false
                return success;
            }
            // 6、如果响应结果为 OK(200, "OK")，则直接返回 true
            return httpResponse.getStatusCode() == Status.OK.getStatusCode();
        } catch (Throwable e) {
            logger.error(...);
            // 7、处理异常，则也返回 false
            return false;
        }
    }  
}
```

###### 3、最后调用 AbstractJerseyEurekaHttpClient#sendHeartBeat

```java
public abstract class AbstractJerseyEurekaHttpClient implements EurekaHttpClient {
    // 最后实际调用AbstractJerseyEurekaHttpClient#getDelta方法，中间经历的代理跟注册的类似：
    // 1) 多态调用SessionedEurekaHttpClient#execute
    // 2) 多态调用RetryableEurekaHttpClient#execute
    // 3) 多态调用RedirectingEurekaHttpClient#execute
    // 4) 多态调用MetricsCollectingEurekaHttpClient#execute
    @Override
    public EurekaHttpResponse<InstanceInfo> sendHeartBeat(String appName, String id, InstanceInfo info, InstanceStatus overriddenStatus) {
        String urlPath = "apps/" + appName + '/' + id;
        ClientResponse response = null;
        try {
            // 1、serviceurl=http://localhost:20000/eureka/
            WebResource webResource = jerseyClient.resource(serviceUrl)
                	// 2、urlPath=apps/service-name/host:service-name:ip
                    .path(urlPath)
                	// 3、status=UP
                    .queryParam("status", info.getStatus().toString())
                	// 4、lastDirtyTimestamp=1638885398794
                    .queryParam("lastDirtyTimestamp", info.getLastDirtyTimestamp().toString());
            ...
            // 5、真正发起心跳包到Eureka Server
            response = requestBuilder.put(ClientResponse.class);
            ...
        } finally {
            ...
        }
    }
}
```

##### 响应请求 | Eureka Server

###### 1、续约心跳包接收接口

| 配置                                   | 默认 | 说明                             |
| -------------------------------------- | ---- | -------------------------------- |
| eureka.server.syncWhenTimestampDiffers | true | 检查是否在脏时间戳不同时同步实例 |

```java
@Produces({"application/xml", "application/json"})
public class InstanceResource {
    @PUT
    public Response renewLease(
            @HeaderParam(PeerEurekaNode.HEADER_REPLICATION) String isReplication,
            @QueryParam("overriddenstatus") String overriddenStatus,
            @QueryParam("status") String status,
        @QueryParam("lastDirtyTimestamp") String lastDirtyTimestamp) {
        // 1、执行服务续约
        boolean isSuccess = registry.renew(app.getName(), id, isFromReplicaNode);
        
        // 2、续约失败，返回404，代表客户端服务找不到，让客户端重新发起注册
        if (!isSuccess) {
            logger.warn("Not Found (Renew): {} - {}", app.getName(), id);
            return Response.status(Status.NOT_FOUND).build();
        }
        
        // 3、对比Eureka Server中的租约，检查是否需要根据脏时间戳进行同步，以及客户端实例是否可能已经更改了某些值
        Response response;
        if (lastDirtyTimestamp != null && serverConfig.shouldSyncWhenTimestampDiffers()) {
            // 4、租约验脏，如果需要在脏时间戳不同时，重新同步实例，则校验客户端传过来的脏时间戳
            response = this.validateDirtyTimestamp(Long.valueOf(lastDirtyTimestamp), isFromReplicaNode);
            // 5、如果是副本间同步，且租约状态发生变化，则覆盖租约实例
            if (response.getStatus() == Response.Status.NOT_FOUND.getStatusCode()
                    && (overriddenStatus != null)
                    && !(InstanceStatus.UNKNOWN.name().equals(overriddenStatus))
                    && isFromReplicaNode) {
                registry.storeOverriddenStatusIfRequired(app.getAppName(), id, InstanceStatus.valueOf(overriddenStatus));
            }
        } 
        // 6、如果无需重新同步，则直接返回ok
        else {
            response = Response.ok().build();
        }
        logger.debug(...);
        return response;
    }
}
```

###### 2、执行服务续约

```java
public class InstanceRegistry extends PeerAwareInstanceRegistryImpl
implements ApplicationContextAware {
	@Override
	public boolean renew(final String appName, final String serverId,
			boolean isReplication) {
		log(...);
		// 1、获取Eureka Server缓存中的所有租约
		List<Application> applications = getSortedApplications();
		// 2、根据服务名获取对应的租约
		for (Application input : applications) {
			if (input.getName().equals(appName)) {
				InstanceInfo instance = null;
				for (InstanceInfo info : input.getInstances()) {
					if (info.getId().equals(serverId)) {
						instance = info;
						break;
					}
				}
				publishEvent(new EurekaInstanceRenewedEvent(this, appName, serverId,
						instance, isReplication));
				break;
			}
		}
		// 3、调用父类进行续租
		return super.renew(appName, serverId, isReplication);
	}
}

@Singleton
public class PeerAwareInstanceRegistryImpl extends AbstractInstanceRegistry implements PeerAwareInstanceRegistry {
    public boolean renew(final String appName, final String id, final boolean isReplication) {
    	// 4、再调用父类进行续租
        if (super.renew(appName, id, isReplication)) {
        	// 7、如果续约成功，则还要将所有eureka操作复制到对等的其他eureka节点
            replicateToPeers(Action.Heartbeat, appName, id, null, null, isReplication);
            return true;
        }
        return false;
    }
}

public abstract class AbstractInstanceRegistry implements InstanceRegistry {
    public boolean renew(String appName, String id, boolean isReplication) {
    	...
    	// 5、租期续约
		leaseToRenew.renew();
		return true;
    }
}

public class Lease<T> {
    public void renew() {
    	// 6、租期续约，当前时间+客户端设置的租约过期时间（eureka.instance.lease-expiration-duration-in-seconds）
        lastUpdateTimestamp = System.currentTimeMillis() + duration;
    }
}
```

###### 3、租约验脏

```java
@Produces({"application/xml", "application/json"})
public class InstanceResource {
    private Response validateDirtyTimestamp(Long lastDirtyTimestamp,
                                            boolean isReplication) {
        // 1、获取Eureka Server缓存的服务租约
        InstanceInfo appInfo = registry.getInstanceByAppAndId(app.getName(), id, false);
        if (appInfo != null) {
            // 2、如果脏时间戳不一致
            if ((lastDirtyTimestamp != null) && (!lastDirtyTimestamp.equals(appInfo.getLastDirtyTimestamp()))) {
                Object[] args = {id, appInfo.getLastDirtyTimestamp(), lastDirtyTimestamp, isReplication};
				// 3、如果传过来的脏时间戳大，则认为Eureka Server中的租约落后，需要重新注册服务，则返回404给客户端，让其重新注册
                if (lastDirtyTimestamp > appInfo.getLastDirtyTimestamp()) {
                    logger.debug(...);
                    return Response.status(Status.NOT_FOUND).build();
                } 
                else if (appInfo.getLastDirtyTimestamp() > lastDirtyTimestamp) {
                    // 4、如果传过来的脏时间落后，且是副本间同步的话，则将注册表中的当前实例信息，发送给复制节点以与该节点同步
                    if (isReplication) {
                        logger.debug(...);
                        return Response.status(Status.CONFLICT).entity(appInfo).build();
                    } 
                    // 5、否则，返回ok，代表租约正常
                    else {
                        return Response.ok().build();
                    }
                }
            }
        }
        return Response.ok().build();
    } 
}
```

#### 服务剔除原理 | Eureka Server

##### 1、初始化 Eureka Server 上下文

| 配置                                    | 默认  | 说明                   |
| --------------------------------------- | ----- | ---------------------- |
| eureka.server.evictionIntervalTimerInMs | 60000 | 服务剔除任务的执行周期 |

```java
public class EurekaServerBootstrap {
	public void contextInitialized(ServletContext context) {
        ...
        // 1、注册中心启动时，初始化Eureka上下文，详情见《注册中心启动原理》
        initEurekaServerContext();
        ...
	}
    
    protected void initEurekaServerContext() throws Exception {
        ...
        // 2、初始化启动变量，以及启动服务剔除定时任务
		this.registry.openForTraffic(this.applicationInfoManager, registryCount);
    }
}

public class InstanceRegistry extends PeerAwareInstanceRegistryImpl
implements ApplicationContextAware {
	@Override
	public void openForTraffic(ApplicationInfoManager applicationInfoManager, int count) {
		// 3. 初始化启动变量，以及启动服务剔除定时任务
		super.openForTraffic(applicationInfoManager,
				count == 0 ? this.defaultOpenForTrafficCount : count);
	}
}

public class PeerAwareInstanceRegistryImpl extends AbstractInstanceRegistry implements PeerAwareInstanceRegistry {
    @Override
    public void openForTraffic(ApplicationInfoManager applicationInfoManager, int count) {
    	...
    	// 4.设置完毕，上下文后置初始化处理
    	super.postInit();
    }
}

public abstract class AbstractInstanceRegistry implements InstanceRegistry {
    protected void postInit() {
   		// 5. 设置服务剔除任务
        evictionTaskRef.set(new EvictionTask());
        // 6. 默认延迟60s后开始执行任务，且每60s执行一次
        evictionTimer.schedule(evictionTaskRef.get(),
                serverConfig.getEvictionIntervalTimerInMs(),
                serverConfig.getEvictionIntervalTimerInMs());
    }
}
```

##### 2、执行服务剔除

```java
public abstract class AbstractInstanceRegistry implements InstanceRegistry {
    
    class EvictionTask extends TimerTask {
        private final AtomicLong lastExecutionNanosRef = new AtomicLong(0l);

        @Override
        public void run() {
            try {
                // 1、计算补偿时间，定义为自上次迭代以来执行此任务的实际时间，与配置的执行时间量。 这对于时间变化（例如由于时钟偏差或 gc）导致实际驱逐任务执行晚于根据配置的周期所需的时间的情况很有用
                long compensationTimeMs = getCompensationTimeMs();
                logger.info(...);
                // 2、执行服务剔除
                evict(compensationTimeMs);
            } catch (Throwable e) {
                logger.error(...);
            }
        }
    }
    
    // additionalLeaseMs补偿时间，默认为0
    public void evict(long additionalLeaseMs) {
        logger.debug("Running the evict task");
        
        // 3、如果自我保护已经打开了（动态变化），则不进行服务剔除，直接返回
        if (!isLeaseExpirationEnabled()) {
            logger.debug("");
            return;
        }
        ...
        // 4、先收集所有过期的租约，判断租期是否已经过期，如果已经过期则加入过期列表
        if (lease.isExpired(additionalLeaseMs) && lease.getHolder() != null) {
            expiredLeases.add(lease);
        }
        ...
        // 6、然后随机顺序驱逐它们，对于大型驱逐集，如果我们不这样做，我们可能会在自我保护开始之前清除整个应用程序，而通过随机化它们，剔除后租约应该可以均匀分布在各个应用程序中
        int next = i + random.nextInt(expiredLeases.size() - i);
        Collections.swap(expiredLeases, i, next);
        Lease<InstanceInfo> lease = expiredLeases.get(i);
        String appName = lease.getHolder().getAppName();
        String id = lease.getHolder().getId();
        EXPIRED.increment();
        logger.warn("DS: Registry: expired lease for {}/{}", appName, id);
        // 7、取消服务租约
        internalCancel(appName, id, false);
    }
}

public class Lease<T> {
    public boolean isExpired(long additionalLeaseMs) {
        // 5、如果该租约经过过期了，或者最后心跳时间+最长续约时间+时间补偿，还大于了当前时间，则也认为该租约过期了
        return (evictionTimestamp > 0 || System.currentTimeMillis() > (lastUpdateTimestamp + duration + additionalLeaseMs));
    }
}

public class InstanceRegistry extends PeerAwareInstanceRegistryImpl
implements ApplicationContextAware {
	@Override
	protected boolean internalCancel(String appName, String id, boolean isReplication) 
		// 8、记录日志、发布租约取消事件
		handleCancelation(appName, id, isReplication);
		// 9、调用父类，取消服务租约
		return super.internalCancel(appName, id, isReplication);
	}
}

public abstract class AbstractInstanceRegistry implements InstanceRegistry {
    protected boolean internalCancel(String appName, String id, boolean isReplication) {
    	...
    	// 10、删除对应的服务列表
		leaseToCancel = gMap.remove(id);
		// 11、清空对应服务列表的读写缓存，在下次只读缓存与读写缓存同步时，会触发一次读写缓存的加载，由于该服务已从服务列表删除，所以只读缓存也会清空该服务列表，不过有一定的延迟（默认30s同步一次）
		invalidateCache(appName, vip, svip);
		...
    }
}
```

#### 服务自保原理 | Eureka Server

##### 1、服务剔除

```java
public abstract class AbstractInstanceRegistry implements InstanceRegistry {
    // 1、只有服务剔除才会触发服务自保，服务下线并不会有这判断
    public void evict(long additionalLeaseMs) {
       	...
        
        // 2、如果自我保护已经打开了（动态变化），则不进行服务剔除，直接返回
        if (!isLeaseExpirationEnabled()) {
            logger.debug("");
            return;
        }
        ...
    }
}
```

##### 2、服务自保判断

| 配置                                  | 默认 | 说明                         |
| ------------------------------------- | ---- | ---------------------------- |
| eureka.server.enableSelfPreservation  | true | 是否开启自我保护             |
| eureka.instance.lease.renewalInterval | 30s  | 服务实例续约周期             |
| eureka.server.renewalPercentThreshold | 0.85 | 服务实例续约到服务自保的阈值 |

```java
@Singleton
public class PeerAwareInstanceRegistryImpl extends AbstractInstanceRegistry implements PeerAwareInstanceRegistry {
    @Override
    public boolean isLeaseExpirationEnabled() {
        // 1、读取开关配置
        if (!isSelfPreservationModeEnabled()) {
            // 3、服务自保开关已被关闭，所以返回true，代表允许服务剔除，而不再判断自保条件
            return true;
        }
        // 4、如果服务自保开关已开启，则还需要判断自保条件，实例数量大于0，且上一分钟最少续约数 > 每分钟最少收到租约阈值（=实例数 * （60/续租周期）* 0.85）
        return numberOfRenewsPerMinThreshold > 0 && getNumOfRenewsInLastMin() > numberOfRenewsPerMinThreshold;
    }
    
    @Override
    public boolean isSelfPreservationModeEnabled() {
        // 2、eureka.server.enableSelfPreservation，默认为true，代表开启服务自保
        return serverConfig.shouldEnableSelfPreservation();
    }
}

public abstract class AbstractInstanceRegistry implements InstanceRegistry {
    protected void updateRenewsPerMinThreshold() {
        // 4.1. 每分钟最少收到租约阈值=实例数 * （60/续租周期）* 0.85
        this.numberOfRenewsPerMinThreshold = (int) (this.expectedNumberOfClientsSendingRenews
                * (60.0 / serverConfig.getExpectedClientRenewalIntervalSeconds())
                * serverConfig.getRenewalPercentThreshold());
    }
}
```

#### 服务下线原理

##### 发起请求 | Eureka Client

###### 1、Eureka Client 自动装配

```java
public class EurekaClientAutoConfiguration {
    ...
    @Configuration
	@ConditionalOnMissingRefreshScope
    protected static class EurekaClientConfiguration {
        ...
        // 1、注入EurekaClient核心类，且在销毁时调用EurekaClient#shutdown，进行服务下线
		@Bean(destroyMethod = "shutdown")
		@ConditionalOnMissingBean(value = EurekaClient.class, search = SearchStrategy.CURRENT)
		public EurekaClient eurekaClient(ApplicationInfoManager manager,
				EurekaClientConfig config) {
			return new CloudEurekaClient(manager, config, this.optionalArgs,
					this.context);
		}
    }
    ...
}
```

###### 2、服务手动下线

```java
@SpringBootApplication
@EnableDiscoveryClient
public class EurekaClientApplication {

    public static void main(String[] args) {
        new SpringApplicationBuilder(EurekaClientApplication.class)
                .web(WebApplicationType.SERVLET)
                .run(args);
        LockSupport.parkNanos(TimeUnit.SECONDS.toNanos(61));
        // 1、服务手动下线
        DiscoveryManager.getInstance().shutdownComponent();
    }
}

@Deprecated
public class DiscoveryManager {
    public void shutdownComponent() {
        if (discoveryClient != null) {
            try {
                // 2、服务手动下线
                discoveryClient.shutdown();
                discoveryClient = null;
            } catch (Throwable th) {
                logger.error("Error in shutting down client", th);
            }
        }
    }
}

@Singleton
public class DiscoveryClient implements EurekaClient {
    @PreDestroy
    @Override
    public synchronized void shutdown() {
        if (isShutdown.compareAndSet(false, true)) {
            logger.info("Shutting down DiscoveryClient ...");

            // 3、取消监听服务状态变更事件
            if (statusChangeListener != null && applicationInfoManager != null) {
applicationInfoManager.unregisterStatusChangeListener(statusChangeListener.getId());
            }

            // 4、销毁instanceInfoReplicator（本地刷新）、heartbeatExecutor（服务续约）、cacheRefreshExecutor（服务发现）、scheduler（DiscoveryClient定时器）
            cancelScheduledTasks();

            if (applicationInfoManager != null
                    && clientConfig.shouldRegisterWithEureka()
                    && clientConfig.shouldUnregisterOnShutdown()) {
                // 5、标记服务状态为DOWN
                applicationInfoManager.setInstanceStatus(InstanceStatus.DOWN);
                
                // 6、发起取消注册请求
                unregister();
            }

            // 7、销毁各种Client
            if (eurekaTransport != null) {
                eurekaTransport.shutdown();
            }

            // 8、销毁心跳失效、服务租约失效监视器
            heartbeatStalenessMonitor.shutdown();
            registryStalenessMonitor.shutdown();

            logger.info("Completed shut down of DiscoveryClient");
        }
    }
}
```

###### 3、发起取消注册请求

```java
@Singleton
public class DiscoveryClient implements EurekaClient {
    void unregister() {
        ...
        // 1、发起取消注册请求
        EurekaHttpResponse<Void> httpResponse = eurekaTransport.registrationClient.cancel(instanceInfo.getAppName(), instanceInfo.getId());
        ...
    }
}

public abstract class AbstractJerseyEurekaHttpClient implements EurekaHttpClient {
// 2、最后实际调用AbstractJerseyEurekaHttpClient#cancel方法，中间经历的代理跟注册的类似：
// 1) 多态调用SessionedEurekaHttpClient#execute
// 2) 多态调用RetryableEurekaHttpClient#execute
// 3) 多态调用RedirectingEurekaHttpClient#execute
// 4) 多态调用MetricsCollectingEurekaHttpClient#execute
    @Override
    public EurekaHttpResponse<Void> cancel(String appName, String id) {
        // 3、urlPath=apps/service-name/host:service-name:port
        String urlPath = "apps/" + appName + '/' + id;
        ...
        try {
            ...
            // 4、真正发起取消注册的请求
            response = resourceBuilder.delete(ClientResponse.class);
            ...
        } finally {
            ...
        }
    }
}
```

##### 响应请求 | Eureka Server

###### 1、取消注册接口

```java
@Produces({"application/xml", "application/json"})
public class InstanceResource {
    @DELETE
    public Response cancelLease(
            @HeaderParam(PeerEurekaNode.HEADER_REPLICATION) String isReplication) {
        try {
            // 1、取消服务注册
            boolean isSuccess = registry.cancel(app.getName(), id,
                "true".equals(isReplication));

            // 2、取消成功，则返回200，客户端打印状态
            if (isSuccess) {
                logger.debug("Found (Cancel): {} - {}", app.getName(), id);
                return Response.ok().build();
            } 
            // 3、取消失败，则返回404，客户端打印状态
            else {
                logger.info("Not Found (Cancel): {} - {}", app.getName(), id);
                return Response.status(Status.NOT_FOUND).build();
            }
        } 
        // 4、发生异常，则返回500，客户端打印状态
        catch (Throwable e) {
            logger.error("Error (cancel): {} - {}", app.getName(), id, e);
            return Response.serverError().build();
        }
    }
}
```

###### 2、取消服务注册

```java
public class InstanceRegistry extends PeerAwareInstanceRegistryImpl
implements ApplicationContextAware {
	@Override
	public boolean cancel(String appName, String serverId, boolean isReplication) {
		// 1、打印日志、发布服务实例取消事件
		handleCancelation(appName, serverId, isReplication);
		// 2、调用父类方法，取消服务注册
		return super.cancel(appName, serverId, isReplication);
	}

	@Override
	protected boolean internalCancel(String appName, String id, boolean isReplication) {
		// 5、打印日志，发布服务租约取消事件
		handleCancelation(appName, id, isReplication);
		// 6、调用父类方法，取消服务租约（与服务剔除调用的是同一个方法）
		return super.internalCancel(appName, id, isReplication);
	}
}

@Singleton
public class PeerAwareInstanceRegistryImpl extends AbstractInstanceRegistry implements PeerAwareInstanceRegistry {
    @Override
    public boolean cancel(final String appName, final String id,
                          final boolean isReplication) {
        // 3、调用父类方法，取消服务注册
        if (super.cancel(appName, id, isReplication)) {
        	// 9、租约取消成功，则将所有eureka操作，复制到对等的其他eureka节点
            replicateToPeers(Action.Cancel, appName, id, null, null, isReplication);
            // 10、减少每分钟最少续租的阈值
            synchronized (lock) {
                if (this.expectedNumberOfClientsSendingRenews > 0) {
                    this.expectedNumberOfClientsSendingRenews = this.expectedNumberOfClientsSendingRenews - 1;
                    updateRenewsPerMinThreshold();
                }
            }
            // 11、返回true，代表取消注册成功
            return true;
        }
        // 12、返回false，代表取消注册失败
        return false;
    }
}

public abstract class AbstractInstanceRegistry implements InstanceRegistry {
    @Override
    public boolean cancel(String appName, String id, boolean isReplication) {
    	// 4、调用子类实现，取消服务租约
        return internalCancel(appName, id, isReplication);
    }
    
    // 取消服务租约，与服务剔除调用的是同一个方法
    protected boolean internalCancel(String appName, String id, boolean isReplication) {
    	...
    	// 7、删除对应的服务列表
		leaseToCancel = gMap.remove(id);
		// 8、清空对应服务列表的读写缓存，在下次只读缓存与读写缓存同步时，会触发一次读写缓存的加载，由于该服务已从服务列表删除，所以只读缓存也会清空该服务列表，不过有一定的延迟（默认30s同步一次）
		invalidateCache(appName, vip, svip);
		...
    }
}
```

#### 集群同步原理 | Eureka Server

##### 1、服务注册后同步

```java
@Singleton
public class PeerAwareInstanceRegistryImpl extends AbstractInstanceRegistry implements PeerAwareInstanceRegistry {
    @Override
    public void register(final InstanceInfo info, final boolean isReplication) {
        ...
        // 1、服务注册完毕
        super.register(info, leaseDuration, isReplication);
        // 2、集群服务实例同步
        replicateToPeers(Action.Register, info.getAppName(), info.getId(), info, null, isReplication);
    }
}
```

##### 2、服务续约后同步

```java
@Singleton
public class PeerAwareInstanceRegistryImpl extends AbstractInstanceRegistry implements PeerAwareInstanceRegistry {
    public boolean renew(final String appName, final String id, final boolean isReplication) {
        // 1、服务续约
        if (super.renew(appName, id, isReplication)) {
            // 2、如果服务续约成功，则进行集群服务实例同步
            replicateToPeers(Action.Heartbeat, appName, id, null, null, isReplication);
            return true;
        }
        return false;
    }
}
```

##### 3、服务取消后同步

```java
@Singleton
public class PeerAwareInstanceRegistryImpl extends AbstractInstanceRegistry implements PeerAwareInstanceRegistry {
    @Override
    public boolean cancel(final String appName, final String id,
                          final boolean isReplication) {
        // 1、取消服务注册
        if (super.cancel(appName, id, isReplication)) {
            // 2、如果取消成功，则进行集群服务实例同步
            replicateToPeers(Action.Cancel, appName, id, null, null, isReplication);
            ...
            return true;
        }
        return false;
    }
}
```

##### 4、集群同步请求处理 & 发送

```java
// 1、将所有当前Eureka操作，复制到对等的其他Eureka节点
private void replicateToPeers(Action action, String appName, String id,
                              InstanceInfo info /* optional */,
                              InstanceStatus newStatus /* optional */, boolean isReplication) {
    Stopwatch tracer = action.getTimer().start();
    try {
        // 2、期望副本数量加1
        if (isReplication) {
            numberOfReplicationsLastMin.increment();
        }
        // 3、如果该请求已经是一个集群同步请求，则直接返回即可，代表不再发起集群同步请求
        if (peerEurekaNodes == Collections.EMPTY_LIST || isReplication) {
            return;
        }
        // 4、否则需要发起集群同步请求
        for (final PeerEurekaNode node : peerEurekaNodes.getPeerEurekaNodes()) {
            // 5、如果url代表此主机，则直接跳过，不能发送同步请求到当前副本自己
            if (peerEurekaNodes.isThisMyUrl(node.getServiceUrl())) {
                continue;
            }
            // 6、将所有实例的更改操作，复制到对等的其他Eureka节点
            replicateInstanceActionsToPeers(action, appName, id, info, newStatus, node);
        }
    } finally {
        tracer.stop();
    }
}

// 将所有实例的更改操作，复制到对等的其他Eureka节点
private void replicateInstanceActionsToPeers(Action action, String appName,
String id, InstanceInfo info, InstanceStatus newStatus,PeerEurekaNode node) {
    try {
        InstanceInfo infoFromRegistry = null;
        CurrentRequestVersion.set(Version.V2);
        switch (action) {
            case Cancel:
                // 7、如果为服务取消后同步，则调用最终调用AbstractJerseyEurekaHttpClient#cancel发送取消注册请求（同步客户端请求类似，但isReplication=true）
                node.cancel(appName, id);
                break;
            case Heartbeat:
                 // 8、如果为服务续约后同步，则调用最终调用AbstractJerseyEurekaHttpClient#sendHeartBeat发送服务续约请求（同步客户端请求类似，但isReplication=true）
                InstanceStatus overriddenStatus = overriddenInstanceStatusMap.get(id);
                infoFromRegistry = getInstanceByAppAndId(appName, id, false);
                node.heartbeat(appName, id, infoFromRegistry, overriddenStatus, false);
                break;
            case Register:
                // 9、如果为服务注册后同步，则调用最终调用AbstractJerseyEurekaHttpClient#register发送服务注册请求（同步客户端请求类似，但isReplication=true）
                node.register(info);
                break;
            case StatusUpdate:
                // 10、手工接口调用操作，略
                infoFromRegistry = getInstanceByAppAndId(appName, id, false);
                node.statusUpdate(appName, id, newStatus, infoFromRegistry);
                break;
            case DeleteStatusOverride:
                // 11、手工接口调用操作，略
                infoFromRegistry = getInstanceByAppAndId(appName, id, false);
                node.deleteStatusOverride(appName, id, infoFromRegistry);
                break;
        }
    } catch (Throwable t) {
        logger.error(...);
    }
}
```

### 1.8. 详细介绍 Ribbon？

#### 背景

![1639058178382](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1639058178382.png)

1. **负载均衡**，在系统架构中是非常重要的一个不得不做的事情，因为这是系统高可用、网络压力缓解和处理能力扩容的重要手段之一，比如有了负载均衡后，系统最大容量就可以近似于**单机容量 * 集群机器数**了。
2. 负载均衡，按**实现手段**分，可以分为硬件负载均衡和软件负载均衡：
   - **硬件负载均衡**：通过在服务器节点间，安装专门的设备来实现负载均衡。
   - **软件负载均衡**：只需要安装一些模块或者软件，即可完成请求分发等负载均衡工作，
3. 负载均衡，按**实现位置**分，可以分为客户端负载均衡和服务端负载均衡：但大型应用通常是客户端+服务端负载均衡搭配使用的。
   - **客户端负载均衡**：所有客户端节点，都维护着自己要访问的服务列表清单（来自注册中心），并且通过定时或者订阅等方式，维护服务列表清单的健康性。

     ![1639192959329](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1639192959329.png)

   - **服务端负载均衡**：需要一个中间节点来实现请求的转发，比如 Nginx，此时需要考虑该节点的高可用。

     ![1639193020487](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1639193020487.png)

| 负载均衡 | 原理                                                         |
| -------- | ------------------------------------------------------------ |
| Ribbon   | 客户端负载均衡，从注册中心定时读取服务列表，根据策略挑选出目标服务器进行访问 |
| Dubbo    | 客户端负载均衡，拉取 + 订阅 ZK 上的服务列表，根据策略挑选出目标服务器进行访问 |
| Nginx    | 反向代理实现，通过拦截客户端请求，根据策略和 upstream 配置进行转发 |

#### 概念

Ribbon，前身 Netflix Ribbon，通过封装后，成为 Spring Cloud 中一个基于 HTTP 实现的客户端均衡组件，主要功能是提供**客户端的负载均衡算法**，简单来说，就是 Ribbon 基于某种规则（如简单轮询，随即连接等），自动让服务去连接这些机器。

| Ribbon 组件接口   | 默认实现                       | 作用                                                         |
| ----------------- | ------------------------------ | ------------------------------------------------------------ |
| ILoadBalancer     | ZoneAwareLoadBalancer          | 负载均衡器，定义了一系列的操作接口，比如选择服务实例         |
| IRule             | ZoneAvoidanceRule              | 负载均衡策略，内置了很多算法，用于为服务实例的选择提供策略   |
| ServerList        | ConfigurationBasedServerList   | Ribbon 服务列表，负责服务实例的获取和存储，可以从配置文件中读取，也可以从注册中心获取 |
| ServerListFilter  | ZonePreferenceServerListFilter | Ribbon 服务列表过滤器，过滤指定的或者动态获得的服务实例      |
| ServerListUpdater | PollingServerListUpdater       | Ribbon 服务列表更新器，负责更新 Ribbon 缓存的服务实例        |
| IPing             | DummyPing                      | 服务实例检查器，负责检查服务实例是否存活                     |
| IClientConfig     | DefaultClientConfigImpl        | Ribbon 客户端配置类，用于初始化 Ribbon 客户端和 LoadBalancer |

#### 架构原理

![1639281718549](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1639281718549.png)



1. Ribbon 本质上使用 BaseLoadBalancer 存储 ServerList，IPing 机制进行服务实例的健康检查，ServrListFilter 进行服务实例过滤，ServrListUpdater 进行服务列表的更新。
2. 在客户端使用 ILoadBalancer 发起服务调用时，会根据 IRule 负载均衡策略，到 BaseLoadBalancer#ServerList 进行服务实例的筛选， 然后才发起对应的服务调用，从而实现负载均衡。

#### 使用方式

##### POM 依赖

```xml
<!-- spring-cloud-starter-netflix-eureka-client默认自带Ribbon依赖 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
</dependency>
```

##### 负载均衡调用方式

###### 直接使用 LoadBalancerClient

```java
@RestController
@Slf4j
public class ConsumerController {
    @Autowired
    private LoadBalancerClient loadBalancerClient;
    
    @PostMapping("/hello")
    public Friend helloPost(){
        // 1、使用LoadBalancerClient，根据服务名获取服务实例信息
        ServiceInstance instance = loadBalancerClient.choose("eureka-client");
        if(instance == null){
            return null;
        }

        // 2、获取服务地址进行参数组装
        String targetUrl = String.format("http://%s:%s/sayHi", instance.getHost(), instance.getPort());
        log.info("url is {}", targetUrl);

        Friend friendParam = new Friend();
        friendParam.setName("Eureka Consumer");

        // 3、最后使用原生RestTemplate进行服务调用
        return restTemplate.postForObject(targetUrl, friendParam, Friend.class);
    }
}
```

###### RestTemplate + @LoadBalanced

```java
// 1、RestTemplate配置类
@Configuration
public class RestTemplateConfig {
	// 2、配置@LoadBalanced注解
    @Bean("restTemplate")
    @LoadBalanced
    public RestTemplate restTemplate(){
        return new RestTemplate();
    }
}

@RestController
public class RibbonConsumerController {
    // 3、注入配置好的RestTemplate
    @Autowired
    private RestTemplate restTemplate;

    @GetMapping("/sayHi")
    public String sayHi() {
        // 4、使用配置了@LoadBalanced注解的RestTemplate，直接发起服务调用，Ribbon底层会choose对应的服务实例进行调用
        return restTemplate.getForObject("http://eureka-client/sayHi", String.class);
    }
}
```

###### Feign + Ribbon

```java
@SpringBootApplication
@EnableDiscoveryClient
// 1、开启Feign
@EnableFeignClients
public class FeignConsumerApplication {
    public static void main(String[] args) {
        new SpringApplicationBuilder(FeignConsumerApplication.class)
                .web(WebApplicationType.SERVLET)
                .run(args);
    }
}

// 2、配置Feign要调用的服务
@FeignClient("eureka-client")
public interface IService {
	// 3、远程调用eureka-client服务提供者的方法
    @GetMapping("/sayHi")
    String sayHi();
}

@RestController
public class FeignConsumerController {
    // 4、注入对应的Fegin动态代理类
    @Resource
    private IService iService;

    @GetMapping("/sayHi")
    public String sayHi(){
        // 5、发起Feign代理请求，默认集成Ribbon来实现负载均衡
        return iService.sayHi();
    }
}
```

##### 负载均衡策略配置

###### 配置文件配置

```properties
# 配置文件配置，指定Ribbon某个服务的负载均衡策略(先加载, 优先级低)(=> xml->yml->yaml>properties)
eureka-client.ribbon.NFLoadBalancerRuleClassName=com.netflix.loadbalancer.RoundRobinRule
```

###### Java Bean 注入

```java
// Ribbon配置类
@Configuration
public class RibbonConfiguration {
    // 1、指定Ribbon全局的负载均衡策略
    @Bean("defaultLBStrategy")
    public IRule defaultLBStrategy(){
        // 2. RandomRule: 随机访问服务结点
        return new RandomRule();
    }
}
```

###### @RibbonClient 注解配置

```java
@Configuration
// 注解方式配置，指定Ribbon某个服务的负载均衡策略(后加载, 优先级高)
@RibbonClient(name = "eureka-client", configuration = RandomRule.class)
public class RibbonConfiguration {

}
```

#### 负载均衡调用原理

##### LoadBalancerClient 原理

见《详细介绍 Eureka - 服务调用原理》

##### RestTemplate + @LoadBalanced 原理

###### 1、@LoadBalanced 注解打标

```java
@Target({ ElementType.FIELD, ElementType.PARAMETER, ElementType.METHOD })
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
// 0、@LoadBalanced继承于@Qualifier
@Qualifier
public @interface LoadBalanced {

}

// 1、（非源码）RestTemplate配置类
@Configuration
public class RestTemplateConfig {
	// 2、配置@LoadBalanced注解，表示打标一个Bean，并且打标的Bean名称等于@Bean配置的restTemplate
    @Bean("restTemplate")
    @LoadBalanced
    public RestTemplate restTemplate(){
        return new RestTemplate();
    }
}
```

###### 2、Spring SPI 配置

```properties
# ./spring-cloud-commons/2.1.1.RELEASE/spring-cloud-commons-2.1.1.RELEASE.jar!/META-INF/spring.factories
# AutoConfiguration
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
# ...
org.springframework.cloud.client.loadbalancer.LoadBalancerAutoConfiguration,\
# ...
```

###### 3、LoadBalancer  自动装配

```java
@Configuration
@ConditionalOnClass(RestTemplate.class)
@ConditionalOnBean(LoadBalancerClient.class)
@EnableConfigurationProperties(LoadBalancerRetryProperties.class)
public class LoadBalancerAutoConfiguration {
    
	@Configuration
	@ConditionalOnMissingClass("org.springframework.retry.support.RetryTemplate")
    static class LoadBalancerInterceptorConfig {
        // 1、注入LoadBalancerInterceptor，依赖之前RibbonAutoConfiguration注入的LoadBalancerClient和LoadBalancerRequestFactory
		@Bean
		public LoadBalancerInterceptor ribbonInterceptor(
				LoadBalancerClient loadBalancerClient,
				LoadBalancerRequestFactory requestFactory) {
			return new LoadBalancerInterceptor(loadBalancerClient, requestFactory);
		}
        
        // 2、注入RestTemplateCustomizer，依赖上面注入的LoadBalancerInterceptor
		@Bean
		@ConditionalOnMissingBean
		public RestTemplateCustomizer restTemplateCustomizer(
				final LoadBalancerInterceptor loadBalancerInterceptor) {
			return restTemplate -> {
				List<ClientHttpRequestInterceptor> list = new ArrayList<>(
						restTemplate.getInterceptors());
				list.add(loadBalancerInterceptor);
				restTemplate.setInterceptors(list);
			};
		}
    }
    
    // 3、注入List<RestTemplate>，依赖上面@LoadBalanced注解打标的RestTemplate
	@LoadBalanced
	@Autowired(required = false)
	private List<RestTemplate> restTemplates = Collections.emptyList();
    
	@Bean
	public SmartInitializingSingleton loadBalancedRestTemplateInitializerDeprecated(
			final ObjectProvider<List<RestTemplateCustomizer>> restTemplateCustomizers) {
		return () -> restTemplateCustomizers.ifAvailable(customizers -> {
			for (RestTemplate restTemplate : LoadBalancerAutoConfiguration.this.restTemplates) {
				for (RestTemplateCustomizer customizer : customizers) {
                    // 4、使用上面注入的LoadBalancerInterceptor，装饰RestTemplate
					customizer.customize(restTemplate);
				}
			}
		});
	}
    ...
}
```

###### 4、RestTemplate 发起服务调用

```java
@RestController
public class RibbonConsumerController {
    @Autowired
    private RestTemplate restTemplate;

    @GetMapping("/sayHi")
    public String sayHi() {
        // 1、（非源码）使用被LoadBalancerInterceptor装饰过的RestTemplate，发起服务调用
        return restTemplate.getForObject("http://eureka-client/sayHi", String.class);
    }
}
```

###### 5、LoadBalancerInterceptor 拦截调用请求

```java
public class LoadBalancerInterceptor implements ClientHttpRequestInterceptor {
	@Override
	public ClientHttpResponse intercept(final HttpRequest request, final byte[] body,
			final ClientHttpRequestExecution execution) throws IOException {
		...
        // 1、使用注入的LoadBalancerClient执行服务调用
		return this.loadBalancer.execute(serviceName,
				this.requestFactory.createRequest(request, body, execution));
	}
}

public class RibbonLoadBalancerClient implements LoadBalancerClient {
	@Override
	public <T> T execute(String serviceId, LoadBalancerRequest<T> request)
			throws IOException {
        // 2、使用注入的LoadBalancerClient执行服务调用
		return execute(serviceId, request, null);
	}
    
	public <T> T execute(String serviceId, LoadBalancerRequest<T> request, Object hint) throws IOException {
        // 3、同Eureka章节服务调用中的，RibbonLoadBalancerClient#getLoadBalancer => 使用SpringClientFactory负载均衡客户端的上下文工厂，构造一个负载均衡器Bean实例
		ILoadBalancer loadBalancer = getLoadBalancer(serviceId);
        
        // 4、同Eureka章节服务调用中的，RibbonLoadBalancerClient#getServer => 根据服务名选择一个具体的服务实例
		Server server = getServer(loadBalancer, hint);
		if (server == null) {
			throw new IllegalStateException("No instances available for " + serviceId);
		}
        
        // 5、构造RibbonServer
		RibbonServer ribbonServer = new RibbonServer(serviceId, server,
				isSecure(server, serviceId),
				serverIntrospector(serviceId).getMetadata(server));

        // 6、执行远程调用
		return execute(serviceId, ribbonServer, request);
	}
    
	@Override
	public <T> T execute(String serviceId, ServiceInstance serviceInstance,
			LoadBalancerRequest<T> request) throws IOException {
		...
        // 7. 底层依赖于org.springframework.http.client.AsyncClientHttpRequestExecution#executeAsync方法，执行异步远程调用
        T returnVal = request.apply(serviceInstance);
        statsRecorder.recordStats(returnVal);
        return returnVal;
		...
	}
}
```

##### Feign + Ribbon 原理

写到 Feign 时再回来补充。

#### 负载均衡策略原理

##### 负载均衡策略

| 负载均衡策略              | 作用                                                         | 特点                                                         |
| ------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| RandomRule                | 随机选择 Server                                              | 随机策略，使用**自旋重试**方式获取，直到服务器列表为空，或者找到有效服务实例为止 |
| RoundRobinRule            | 按顺序选择 Server                                            | 轮训策略，Ribbon 默认负载均衡策略，**CAS 索引计数器 + 自旋重试**，最多重试 10 次 |
| RetryRule                 | 在一个配置的时间段内（默认为 500ms），如果 Server 选择不成功，则会一直重新尝试选择，直到时间结束或者选到为止 | 重试策略，默认使用装饰类 RoundRobinRule 进行负载均衡，接口实现要注意**幂等性** |
| WeightedResponseTimeRule  | 根据 Server 的响应时间分配权重，响应时间越长，权重越低，则被选中的概率就越低；相反，响应时间越短，权重越高，则被选中的高绿就越高 | 最小响应时间策略，适用于**耗时主要在接口**上的重量级接口（RT 敏感模型） |
| BestAvailableRule         | 逐个检查 Server，如果 Server 的断路器已打开（当前发生熔断），则忽略该 Server，即只会从断路器关闭中进行选取，然后再选择并发连接最低的 Server | 最低并发策略，适用于耗时主要在网络上的短响应时间接口（连接数敏感模型） |
| AvailabilityFilteringRule | 过滤掉断路器已打开（当前发生熔断），或者当前并发数超过阈值的 Server | 可用性过滤策略                                               |
| ZoneAvoidanceRule         | 剔除不可用区域中的所有 Server，和区域中的不可用 Server（默认区域为1），在过滤后的服务列表中轮训选择 Server | 区域可用性过滤策略                                           |

##### 策略实现原理

###### RandomRule | 随机

```java
public class RandomRule extends AbstractLoadBalancerRule {
    public Server choose(ILoadBalancer lb, Object key) {
        if (lb == null) {
            return null;
        }
        Server server = null;

        // 1、开始自旋
        while (server == null) {
            if (Thread.interrupted()) {
                return null;
            }
            List<Server> upList = lb.getReachableServers();
            List<Server> allList = lb.getAllServers();

            int serverCount = allList.size();
            if (serverCount == 0) {
                // 2、一台 Server 都没，则返回 null
                return null;
            }

            // 3、ThreadLocalRandom.current().nextInt(serverCount); => 从所有ServerCount中，返回一个索引
            int index = chooseRandomInt(serverCount);
            // 4、从在线的upList中，选择对应索引的Server（可能会越界！）
            server = upList.get(index);

            // 5、当前线程让步，继续自旋，直到服务器列表为空，或者找到为止
            if (server == null) {
                Thread.yield();
                continue;
            }

            // 6、如果Server存活（通过ServerListUpdater.UpdateAction#doUpdate通过Eureka Client 定时更新），则返回该Server
            if (server.isAlive()) {
                return (server);
            }

            // 7、当前线程让步，继续自旋，直到服务器列表为空，或者找到为止
            server = null;
            Thread.yield();
        }

        return server;
    }
}
```

###### RoundRobinRule | 轮训

```java
public class RoundRobinRule extends AbstractLoadBalancerRule {
    
    private AtomicInteger nextServerCyclicCounter;
    
    public RoundRobinRule() {
        // 1、初始化索引计数器 => 如果某一时刻，大片服务器同时重启，则可能会有大量请求负载到第0个服务实例，但由于每台机器最终启动完成的时间与接收的请求数量并不相同，所以也不会给负载的服务实例造成太大压力
        nextServerCyclicCounter = new AtomicInteger(0);
    }
    
    // 4、int next = (current + 1) % modulo 取模，得到目标索引
    private int incrementAndGetModulo(int modulo) {
        for (;;) {
            int current = nextServerCyclicCounter.get();
            int next = (current + 1) % modulo;
            if (nextServerCyclicCounter.compareAndSet(current, next))
                return next;
        }
    }
    
    public Server choose(ILoadBalancer lb, Object key) {
        if (lb == null) {
            log.warn("no load balancer");
            return null;
        }

        Server server = null;
        int count = 0;
        
        // 2、最多重试10次
        while (server == null && count++ < 10) {
            List<Server> reachableServers = lb.getReachableServers();
            List<Server> allServers = lb.getAllServers();
            int upCount = reachableServers.size();
            int serverCount = allServers.size();

            if ((upCount == 0) || (serverCount == 0)) {
                log.warn("No up servers available from load balancer: " + lb);
                return null;
            }

            // 3、int next = (current + 1) % modulo 取模，得到目标索引
            int nextServerIndex = incrementAndGetModulo(serverCount);
            server = allServers.get(nextServerIndex);

            // 4、服务实例为空，则线程让步，继续自旋
            if (server == null) {
                Thread.yield();
                continue;
            }

            // 5、服务实例可用，则返回即可
            if (server.isAlive() && (server.isReadyToServe())) {
                return (server);
            }

            // 6、继续自旋
            server = null;
        }

        // 7、最多重试10次
        if (count >= 10) {
            log.warn("No available alive servers after 10 tries from load balancer: "
                    + lb);
        }
        return server;
    }
    ...
}
```

###### RetryRule | 重试

```java
public class RetryRule extends AbstractLoadBalancerRule {
    
    long maxRetryMillis = 500;
    IRule subRule = new RoundRobinRule();
    
    ....
    
   	public Server choose(ILoadBalancer lb, Object key) {
		long requestTime = System.currentTimeMillis();
		long deadline = requestTime + maxRetryMillis;

		Server answer = null;

        // 1、默认使用装饰类RoundRobinRule，进行负载均衡
		answer = subRule.choose(key);

        // 2、如果装饰类获取不到有效服务实例，且当前时间小于最大重试时间，则继续使用装饰类重试
		if (((answer == null) || (!answer.isAlive()))
				&& (System.currentTimeMillis() < deadline)) {

            // 3、根据最大重试时间开启定时任务，到点则中断当前线程
			InterruptTask task = new InterruptTask(deadline
					- System.currentTimeMillis());

            // 4、开始自旋
			while (!Thread.interrupted()) {
                // 5、继续使用装饰类重试
				answer = subRule.choose(key);

                // 6、如果还是获取不到有效服务实例，且没超出最大重试时间，则线程让步，继续自旋
				if (((answer == null) || (!answer.isAlive()))
						&& (System.currentTimeMillis() < deadline)) {
					Thread.yield();
				} 
                // 7、否则结束自旋
                else {
					break;
				}
			}

            // 8、取消定时任务
			task.cancel();
		}

        // 9、返回目标服务实例，如果无效则返回null
		if ((answer == null) || (!answer.isAlive())) {
			return null;
		} else {
			return answer;
		}
	}
	...
}
```

###### WeightedResponseTimeRule | 最小响应时间

```java
public class WeightedResponseTimeRule extends RoundRobinRule {
    
    public static final int DEFAULT_TIMER_INTERVAL = 30 * 1000;
    private int serverWeightTaskTimerInterval = DEFAULT_TIMER_INTERVAL;
    
    private volatile List<Double> accumulatedWeights = new ArrayList<Double>();
    protected Timer serverWeightTimer = null;
    protected AtomicBoolean serverWeightAssignmentInProgress = new AtomicBoolean(false);

    @Override
    public void setLoadBalancer(ILoadBalancer lb) {
        ...
        // 1、设置抽象父类的LoadBalancer后，初始化权重
        initialize(lb);
    }
    
    void initialize(ILoadBalancer lb) {
        ...
        // 2、不延迟执行，默认定期30s执行一次DynamicServerWeightTask，来初始化权重
        serverWeightTimer.schedule(new DynamicServerWeightTask(), 0,
                serverWeightTaskTimerInterval);
        
        // 3、立即调用一次，来初始化权重
        ServerWeight sw = new ServerWeight();
        sw.maintainWeights();
    }
    
    class DynamicServerWeightTask extends TimerTask {
        public void run() {
            ServerWeight serverWeight = new ServerWeight();
            try {
                // 4、初始化权重
                serverWeight.maintainWeights();
            } catch (Exception e) {
                logger.error("Error running DynamicServerWeightTask for {}", name, e);
            }
        }
    }
    
    
    class ServerWeight {
        // 初始化权重
        public void maintainWeights() {
            ...
            
            // 5、CAS设置服务权重更新中的标记为true，代表更新中
            if (!serverWeightAssignmentInProgress.compareAndSet(false,  true))  {
                return; 
            }
            
            try {
                ...
                
                // 找到最大 95% 的响应时间
                // 6、遍历全部Server，累加平均响应时间（在RibbonLoadBalancerClient#execute方法中，执行完远程调用后，会调用一次statsRecorder.recordStats来记录统计信息）
                double totalResponseTime = 0;
                for (Server server : nlb.getAllServers()) {
                    ServerStats ss = stats.getSingleServerStat(server);
                    totalResponseTime += ss.getResponseTimeAvg();
                }

                // 7、使用总的平均响应时间 - 每个服务实例的平均响应时间，可以得到响应时间的权重 => 响应时间越长，服务实例越靠前，权重越小；响应时间越短，服务实例越靠后，权重越大
                Double weightSoFar = 0.0;
                List<Double> finalWeights = new ArrayList<Double>();
                for (Server server : nlb.getAllServers()) {
                    ServerStats ss = stats.getSingleServerStat(server);
                    double weight = totalResponseTime - ss.getResponseTimeAvg();
                    weightSoFar += weight;
                    finalWeights.add(weightSoFar);   
                }
                // 8、设置服务权重列表
                setWeights(finalWeights);
            } catch (Exception e) {
                logger.error(...);
            } finally 
                // 10、最后CAS设置服务权重更新中的标记为false，代表更新完毕
                serverWeightAssignmentInProgress.set(false);
            }

        }
    }
    
    // 9、设置服务权重列表
    void setWeights(List<Double> weights) {
        this.accumulatedWeights = weights;
    }

    @Override
    public Server choose(ILoadBalancer lb, Object key) {
        if (lb == null) {
            return null;
        }
        Server server = null;

        while (server == null) {
            // 11、获取当前时刻的服务权重列表（有可能下一刻就会被其他列表更新替换掉）
            List<Double> currentWeights = accumulatedWeights;
            if (Thread.interrupted()) {
                return null;
            }
            
            List<Server> allList = lb.getAllServers();
            int serverCount = allList.size();
            if (serverCount == 0) {
                return null;
            }
			int serverIndex = 0;
            
            // 12、列表中的最后一个，代表最大的服务权重
            double maxTotalWeight = currentWeights.size() == 0 ? 0 : currentWeights.get(currentWeights.size() - 1); 
            
            // 13、如果最大权重小于0.001，或者权重列表个数不等于服务列表个数，说明权重无效，则回退到父类RoundRobinRule规则，来获取服务实例
            if (maxTotalWeight < 0.001d || serverCount != currentWeights.size()) {
                // 14、以父类RoundRobinRule规则获取服务实例
                server =  super.choose(getLoadBalancer(), key);
                if(server == null) {
                    return server;
                }
            } 
            // 15、如果最大权重不小于0.001，且权重列表个数等于服务列表个数，说明权重有效，则进行权重筛选
            else {
                // 16、生成 [0，maxTotalWeight）之间的随机权重
                double randomWeight = random.nextDouble() * maxTotalWeight;
                
                // 17、获取权重列表中，第一个大于等于随机权重的服务实例，代表目标服务实例
                int n = 0;
                for (Double d : currentWeights) {
                    if (d >= randomWeight) {
                        serverIndex = n;
                        break;
                    } else {
                        n++;
                    }
                }

                server = allList.get(serverIndex);
            }

            // 18、如果实例为空，则线程让步，继续自旋
            if (server == null) {
                Thread.yield();
                continue;
            }

            // 19、如果实例存活，则返回即可
            if (server.isAlive()) {
                return (server);
            }

            server = null;
        }
        return server;
    }
}
```

###### BestAvailableRule | 最低并发

```java
public class BestAvailableRule extends ClientConfigEnabledRoundRobinRule {
    @Override
    public Server choose(Object key) {
        // 1、如果没有服务统计信息，则使用父类默认的RoundRobinRule规则，进行服务实例选取
        if (loadBalancerStats == null) {
            return super.choose(key);
        }
        
        // 2、如果有服务统计信息，则获取当前系统时间
        List<Server> serverList = getLoadBalancer().getAllServers();
        int minimalConcurrentConnections = Integer.MAX_VALUE;
        long currentTime = System.currentTimeMillis();
        Server chosen = null;
        
        // 3、遍历所有服务列表
        for (Server server: serverList) {
            ServerStats serverStats = loadBalancerStats.getSingleServerStat(server);
            // 4、根据当前系统时间，判断服务实例的断路器是否已经打开
            if (!serverStats.isCircuitBreakerTripped(currentTime)) {
                // 8、如果该服务实例断路器没有被打开，则获取他当前系统时间的并发请求数量
                int concurrentConnections = serverStats.getActiveRequestsCount(currentTime);
                // 9、比较获取最小并发请求数量的服务实例，作为目标服务实例
                if (concurrentConnections < minimalConcurrentConnections) {
                    minimalConcurrentConnections = concurrentConnections;
                    chosen = server;
                }
            }
        }
        // 10、如果目标服务实例无效，则使用父类默认的RoundRobinRule规则，进行服务实例选取
        if (chosen == null) {
            return super.choose(key);
        } 
        // 11、如果目标服务实例有效，则返回即可
        else {
            return chosen;
        }
    }
}

public class ServerStats {
    // 5、根据当前系统时间，判断服务实例的断路器是否已经打开
    public boolean isCircuitBreakerTripped(long currentTime) {
        // 6、获取当前服务断路器的超时时间
        long circuitBreakerTimeout = getCircuitBreakerTimeout();
        
        // 7、如果该超时时间无效，或者已经小于了当前系统时间，则认为断路器已关闭，服务实例有效
        if (circuitBreakerTimeout <= 0) {
            return false;
        }
        return circuitBreakerTimeout > currentTime;
    }
    
    private long getCircuitBreakerTimeout() {
        // 6.1、 获取断路器持续周期
        long blackOutPeriod = getCircuitBreakerBlackoutPeriod();
        if (blackOutPeriod <= 0) {
            return 0;
        }
        // 6.6、断路器持续周期 + 最后连接失败的时间 = 断路器超时时间
        return lastConnectionFailedTimestamp + blackOutPeriod;
    }
    
    private long getCircuitBreakerBlackoutPeriod() {
        int failureCount = successiveConnectionFailureCount.get();
        int threshold = connectionFailureThreshold.get();
        
        // 6.2、如果调用失败的数量，没有达到阈值niws.loadbalancer.service-name.connectionFailureCountThreshold（默认为3），则返回0，表示不开启断路器
        if (failureCount < threshold) {
            return 0;
        }
        
        // 6.3、否则调用失败数量 - 断路器阈值，得到差值（最大为16）
        int diff = (failureCount - threshold) > 16 ? 16 : (failureCount - threshold);
        
        // 6.4、然后计算断路器持续周期/s = 差值 * niws.loadbalancer.service-name-.circuitTripTimeoutFactorSeconds（默认为10），即默认最大为160s（2min40s）
        int blackOutSeconds = (1 << diff) * circuitTrippedTimeoutFactor.get();
        if (blackOutSeconds > maxCircuitTrippedTimeout.get()) {
            blackOutSeconds = maxCircuitTrippedTimeout.get();
        }
        
        // 6.5、最后s转换为ms，并返回
        return blackOutSeconds * 1000L;
    }
}
```

###### AvailabilityFilteringRule | 可用性过滤

```java
public class AvailabilityFilteringRule extends PredicateBasedRule {
    
    private AbstractServerPredicate predicate;
    
    @Override
    public void initWithNiwsConfig(IClientConfig clientConfig) {
        // 1、初始化可用性断言
    	predicate = CompositePredicate.withPredicate(new AvailabilityPredicate(this, clientConfig)).addFallbackPredicate(AbstractServerPredicate.alwaysTrue()).build();
    }
    
    @Override
    public Server choose(Object key) {
        int count = 0;
        // 4、调用父类默认的RoundRobinRule规则，进行服务实例的选择
        Server server = roundRobinRule.choose(key);
        // 5、最多选择再选择10次服务实例+判断可用性断言是否成立
        while (count++ <= 10) {
            // 6、判断当前服务实例是否符合可用性断言，如果是则返回即可，否则继续获取服务实例来判断
            if (predicate.apply(new PredicateKey(server))) {
                return server;
            }
            server = roundRobinRule.choose(key);
        }
        // 7、如果10次判断都没有符合断言的服务实例，则再使用RoundRobinRule来获取最后一次
        return super.choose(key);
    }
}

public class AvailabilityPredicate extends  AbstractServerPredicate {
   
    public AvailabilityPredicate(IRule rule, IClientConfig clientConfig) {
        super(rule, clientConfig);
        // 2、初始化可用性断言
        initDynamicProperty(clientConfig);
    }
    
    private void initDynamicProperty(IClientConfig clientConfig) {
        String id = "default";
        if (clientConfig != null) {
            id = clientConfig.getClientName();
            // 3、设置activeConnectionsLimit，默认为niws.loadbalancer.availabilityFilteringRule.activeConnectionsLimit=Integer.MAX_VALUE
            activeConnectionsLimit = new ChainedDynamicProperty.IntProperty(id + "." + clientConfig.getNameSpace() + ".ActiveConnectionsLimit", ACTIVE_CONNECTIONS_LIMIT); 
        }               
    }
    
    @Override
    public boolean apply(@Nullable PredicateKey input) {
        LoadBalancerStats stats = getLBStats();
        if (stats == null) {
            return true;
        }
        // 6.1、判断当前服务实例是否符合可用性断言，如果shouldSkipServer返回true，则代表需要跳过，不能作为目标服务实例；否则说明可以作为目标服务实例
        return !shouldSkipServer(stats.getSingleServerStat(input.getServer()));
    }
    
    private boolean shouldSkipServer(ServerStats stats) {
        // 6.2、先看断路器过滤是否已经打开niws.loadbalancer.availabilityFilteringRule.filterCircuitTripped（默认为true），如果判断到断路器开关已经打开，说明发生了熔断，则直接返回true；否则，如果并发数超过了activeConnectionsLimit（默认最大Integer.MAX_VALUE），也返回true => 认为不可用，需要跳过
        if ((CIRCUIT_BREAKER_FILTERING.get() && stats.isCircuitBreakerTripped()) 
                || stats.getActiveRequestsCount() >= activeConnectionsLimit.get()) {
            return true;
        }
        // 6.3、如果既不熔断，又没超过最大并发数，则返回false，代表可用，不需要跳过
        return false;
    }
}
```

###### ZoneAvoidanceRule | 区域可用性过滤

```java
public class ZoneAvoidanceRule extends PredicateBasedRule {
    
    private CompositePredicate compositePredicate;
    
    @Override
    public void initWithNiwsConfig(IClientConfig clientConfig) {
        // 1、初始化ZoneAvoidancePredicate断言，在区域<=1时恒为true
        ZoneAvoidancePredicate zonePredicate = new ZoneAvoidancePredicate(this, clientConfig);
        // 2、初始化可用性断言，同AvailabilityFilteringRule的可用性过滤
        AvailabilityPredicate availabilityPredicate = new AvailabilityPredicate(this, clientConfig);
        // 3、组合断言compositePredicate，链式判断zonePredicate和availabilityPredicate
        compositePredicate = createCompositePredicate(zonePredicate, availabilityPredicate);
    }
    
    // 7、获取ZoneAvoidanceRule的断言compositePredicate
    @Override
    public AbstractServerPredicate getPredicate() {
        return compositePredicate;
    }
}

public abstract class PredicateBasedRule extends ClientConfigEnabledRoundRobinRule {
 	// 4、ZoneAvoidanceRule使用父类PredicateBasedRule，进行服务实例的选择
    @Override
    public Server choose(Object key) {
        ILoadBalancer lb = getLoadBalancer();
        Optional<Server> server = 
            // 5、多态获取ZoneAvoidanceRule的断言
            getPredicate()
            // 8、从过滤后的服务列表中，获取目标服务实例
            .chooseRoundRobinAfterFiltering(lb.getAllServers(), key);
        if (server.isPresent()) {
            return server.get();
        } else {
            return null;
        }       
    }
    
    // 6、多态获取ZoneAvoidanceRule的断言
    public abstract AbstractServerPredicate getPredicate();
}

public abstract class AbstractServerPredicate implements Predicate<PredicateKey> {
    public Optional<Server> chooseRoundRobinAfterFiltering(List<Server> servers, Object loadBalancerKey) {
        // 9、根据断言，获取过滤后的服务列表
        List<Server> eligible = getEligibleServers(servers, loadBalancerKey);
        if (eligible.size() == 0) {
            return Optional.absent();
        }
        return Optional.of(eligible.get(incrementAndGetModulo(eligible.size())));
    }
    
    public List<Server> getEligibleServers(List<Server> servers, Object loadBalancerKey) {
        if (loadBalancerKey == null) {
            return ImmutableList.copyOf(Iterables.filter(servers, this.getServerOnlyPredicate()));            
        } else {
            List<Server> results = Lists.newArrayList();
            for (Server server: servers) {
                // 10、应用组合断言compositePredicate，链式判断zonePredicate（区域<=1时恒为true）和availabilityPredicate（无熔断，并发数不超阈值时为true）
                if (this.apply(new PredicateKey(loadBalancerKey, server))) {
                    results.add(server);
                }
            }
            return results;            
        }
    }
}
```

##### 自定义负载均衡算法 | 一致性 Hash

```java
// 自定义负载均衡算法，一致性Hash => 实现 IRule 接口，继承 AbstractLoadBalancerRule 抽象类
public class ConsistentHashRule extends AbstractLoadBalancerRule implements IRule {
    @Override
    public void initWithNiwsConfig(IClientConfig clientConfig) {

    }
    
    @Override
    public Server choose(Object key) {
        // 1、获取请求标识，用于计算hash，这里用了服务路径+请求参数
        HttpServletRequest request = ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes()).getRequest();
        String uri = request.getServletPath() + "?" + request.getQueryString();
  		// 2、先获取全部服务实例，然后根据hash值，获取目标服务实例
        return route(uri.hashCode(), getLoadBalancer().getAllServers());
    }
    
    public Server route(int hashId, List<Server> addressList){
        if(CollectionUtils.isEmpty(addressList)){
            return null;
        }

        // 3、虚化Server结点, 假设这里每个Server拥有8个虚化结点 => 可以参考Dubbo 160个虚拟结点的hash环
        TreeMap<Long, Server> address = new TreeMap<>();
        addressList.stream().forEach(server -> {
            // 4、虚化Server结点到环上
            for(int i = 0; i < 8; i++){
                // 5、加入服务实例ID作为变数
                long hash = hash(server.getId() + i);
                address.put(hash, server);
            }
        });

        // 6、tailMap取比当前hash大的且离他最近的一个结点 => 顺时针方向: tailMap有序, 会拿到所有比他大的结点
        long hash = hash(String.valueOf(hashId));
        SortedMap<Long, Server> lastSevers = address.tailMap(hash);

        // 7、如果拿不到比他大的结点, 说明我们的hash到了末尾, 需要取最小的结点, 从而虚化成一个环
        if(lastSevers.isEmpty()){
            return address.firstEntry().getValue();
        }

        // 8、如果拿得到比他大的结点, 则取大集中的最小值
        return lastSevers.get(lastSevers.firstKey());
    }
    
    // 5.1、哈希函数
    public long hash(String key){
        MessageDigest md5 = null;
        try {
            md5 = MessageDigest.getInstance("MD5");
        } catch (NoSuchAlgorithmException e) {
            e.printStackTrace();
        }

        byte[] keyBytes = null;
        try {
            keyBytes = key.getBytes("UTF-8");
        } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
        }

        // 5.2、根据Key的字节数组生成16字节(128位)的MD5摘要
        md5.update(keyBytes);
        byte[] digest = md5.digest();

        // 5.3、计算long长度的hashCode => 这里做测试, 只取低4个字节整成long型数字作为hashCode, 其中高4个字节都取0, 这里与0xFF避免字节本身为1开头时移位带来的问题
        long hashCode = (digest[3] & 0xFF) << 24    // 高16位
                      | (digest[2] & 0xFF) << 16    // 高8位
                      | (digest[1] & 0xFF) << 8     // 低16位
                      | (digest[0] & 0xFF);         // 低8位

        // 5.4、只取低8字节作为long型返回, 一个long型数字占8个字节
        return hashCode & 0xFFFFFFFFL;
    }
}
```

### 1.9. 详细介绍 Feign？

#### 概念

Feign 是声明式的服务客户端，使得调用远程方法就像调用本地接口一样方便，只需把要调用的服务方法，定义成接口，然后直接调用就行，而无需手动构建 Http 请求再发起服务调用。

| @Feign Client 属性 | 默认值     | 作用                                                         |
| ------------------ | ---------- | ------------------------------------------------------------ |
| value              | ""         | 将要调用的服务名称，是 name 属性的别名                       |
| serviceId          | ""         | 已废弃，将要调用的服务id，作用和 name 属性相同               |
| name               | ""         | 将要调用的服务名称，是 name 属性的别名                       |
| url                | ""         | 全服务路径地址，或者 hostname                                |
| decode404          | false      | 响应状态码为 404 时，是否应该抛出 FeginException             |
| configuration      | {}         | 自定义当前 Feign Client 配置                                 |
| fallback           | void.class | 降级机制（需启用 @EnableHystrix），调用失败时走的一些回退方法，可以用来抛出异常或者给出默认返回的数据 |
| fallbackFactory    | void.class | 为当前 Feign Client接口定义一个降级工厂，用于生成降级类的实例 |
| path               | ""         | 自动给所有方法的 @RequestMapping 加上前缀，类似于 Controller 类上的 @RequestMapping |
| primary            | true       | 是否将当前 Feign Client 标记为第一个注入 Bean                |

#### 架构原理

![1639579104510](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1639579104510.png)

| 组件                     | 作用                                                         |
| ------------------------ | ------------------------------------------------------------ |
| InvocationHandlerFactory | 代理对象生成组件，采用 JDK 的动态代理方式生成代理对象，当调用 Feign 接口时，实际上是去调用远程的 HTTP API |
| Contract                 | 协议组件，解析 @Feign 配置的参数，比如请求类型是 GET 还是 POST，请求的 URI 是什么 |
| MethodHander             | 实际处理组件，当调用 Feign 接口时，实际上是交由该类，来进行远程调用的相关处理 |
| Encoder/Decoder          | 编码/解码组件，通过它们，可以将请求信息，采用指定的方式进行编/解码后再传输/接收 |
| Logger                   | 日志组件，负责记录 Feign 中的日志。其中，可以指定 Logger 的级别以及自定义日志的输出 |
| Client                   | 请求执行组件，负责 HTTP 请求的执行。其中，Feign 默认的 Client 是通过 JDK#HttpURLConnection 发起请求的，在每次发送请求时，都会创建新的 HttpURLConnection 连接，性能很差；因此，可以通过扩展该接口，使用比如 Apache HttpClient 等，基于连接池的高性能 HTTP 客户端来进行优化 |
| Retryer                  | 重试组件，负责重试请求调用，Feign 内置了重试器，当 HTTP 请求出现 IO 异常时，Feign 会限定一个最大重试次数，来进行重试操作 |
| RequestInterceptor       | 请求拦截器组件，可以为 Feign 添加多个拦截器，在请求执行前设置一些扩展的参数信息 |

#### 使用方式

##### POM 依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

##### 配置 Feign 接口

```java
@FeignClient(value = "feign-client")
public interface IService {
    @GetMapping("/sayHi")
    public String sayHi();
}
```

##### 使用 Feign 接口

```java
@SpringBootApplication
@EnableDiscoveryClient
// 1、开启FeignClient扫描
@EnableFeignClients
public class FeignConsumerApplication {
    public static void main(String[] args) {
        new SpringApplicationBuilder(FeignConsumerApplication.class)
                .web(WebApplicationType.SERVLET)
                .run(args);
    }
}

@RestController
public class FeignConsumerController {
    // 2、注入Feign接口代理类
    @Resource
    private IService iService;

    @GetMapping("/sayHi")
    public String sayHi(){
        // 3、发起Http调用
        return iService.sayHi();
    }
}
```

##### 超时重试配置

下面这些配置只是针对 feign-client 服务的配置，对于全局服务的配置参数名一样，不同的是没有服务名作为配置前缀，而是使用 `feign.client.config.default` 作为前缀。

```properties
# 只对服务消费者设置超时与重试策略:
# Ribbon指定服务的超时策略: 请求服务的连接超时时间: Http连接所花费的时间(ms)
feign-client.ribbon.ConnectTimeout=1000
# Ribbon指定服务的超时策略: 服务的业务处理超时时间(ms)
feign-client.ribbon.ReadTimeout=2000

# Ribbon指定服务的重试策略: 允许在所有Http Method都进行重试策略(Get/Post/Put/Delete...)
feign-client.ribbon.OkToRetryOnAllOperations=true
# Ribbon指定服务的重试策略: 每台机器最大的服务超时重试次数 => 因此, 每台机器的请求次数 = 1 + 2 = 3次
feign-client.ribbon.MaxAutoRetries=2
# Ribbon指定服务的重试策略: 服务重试超时时, 还可以再重试几台机器 => 因此, 一共可以请求机器数 = 1 + 2 = 3台, 如果只有1台机器, 那么会继续请求该台机器
feign-client.ribbon.MaxAutoRetriesNextServer=2
# 因此, 一次服务请求的最大超时次数 = (1000 + 2000) * (1 + 2) * (1 + 2) = 27000ms = 27s
```

##### 高性能客户端配置

```yml
feign:
  # 使用okhttp高性能客户端
  okhttp: true
  # 使用Apache httpclient高性能客户端
  # httpclient: true
```

##### 调用日志配置

```yaml
feign:
  client:
    config:
      # Feign全局配置
      default:
        # 记录请求和响应的标头、正文和元数据
        loggerLevel: full
```

##### 请求包压缩配置

```yaml
feign:
  compression:
    request:
      # 开启请求GZIP压缩
      enabled: true
      # 配置压缩数据⼤大⼩小的下限
      min-request-size: 2048
      # 配置支持压缩的MIME TYPE
      mime-types: text/xml,application/xml,application/json
    response:
      #开启响应GZIP压缩
      enabled: true
```

##### 拦截器扩展

```java
@Component
@Slf4j
public class FeignRequestInterceptor implements RequestInterceptor {
    @Override
    public void apply(RequestTemplate requestTemplate) {
        // 在Header设置Feign Client请求标志
        requestTemplate.header(ProjectConstant.FEIGN_CLIENT_REQUEST_FLAG, "true");
    }
}
```

##### 自定义包装类解码器

```java
// 1、自定义Feign解码器: 解决Feign Sever结果统一包装的问题
@Component
class FeignResultDecoder implements Decoder {
    @Autowired
    private ObjectMapper objectMapper;

    @Override
    public Object decode(Response response, Type type) throws IOException, DecodeException, FeignException {
        if (response.body() == null)
            throw new DecodeException(...);

        // 2、解析body, 得到Result包装类实例
        Result result = objectMapper.readValue(Util.toString(response.body().asReader(Util.UTF_8)), Result.class);
        if (ResultCode.ERROR_CODE.equals(result.getRetCode()))
            throw new DecodeException(...);

        // 3、重新解析包装类Result#data实例并返回
        return objectMapper.readValue(objectMapper.writeValueAsString(result.getData()), TypeFactory.defaultInstance().constructType(type));
    }
}
```

##### Hystrix 降级熔断配置

```java
// 1、直接配置降级熔断处理类IFallbackHandler
@FeignClient(value = "feign-client", fallback = IFallbackHandler.class)
public interface HystrixFallbackService extends IService {

}

// 2、配置降级工厂类HystrixClientFallbackFactory，用于生成降级熔断处理类
@FeignClient(value = "feign-client", fallback = HystrixClientFallbackFactory.class)
public interface HystrixFallbackService extends IService {

}

@Component
public class HystrixClientFallbackFactory implements FallbackFactory<HystrixFallbackService> {
    @Override
    public HystrixFallbackService create(Throwable cause) {
        // 2.1、手动实现一个HystrixFallbackService降级熔断处理类
        return new HystrixFallbackService(){
            ...
        }
    }
}
```

#### 动态代理原理

##### 1、开启 FeignClient 扫描

```java
@SpringBootApplication
@EnableDiscoveryClient
// 1、开启FeignClient扫描
@EnableFeignClients
public class FeignConsumerApplication {
    public static void main(String[] args) {
        new SpringApplicationBuilder(FeignConsumerApplication.class)
                .web(WebApplicationType.SERVLET)
                .run(args);
    }
}

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
// 2、导入FeignClientsRegistrar
@Import(FeignClientsRegistrar.class)
public @interface EnableFeignClients {
    ...
}


```

##### 2、扫描 @FeignClient 注解

```java
class FeignClientsRegistrar
    implements ImportBeanDefinitionRegistrar, ResourceLoaderAware, EnvironmentAware {
	@Override
	public void registerBeanDefinitions(AnnotationMetadata metadata,
			BeanDefinitionRegistry registry) {
        // 1、读取并注册@EnableFeignClients#defaultConfiguration配置
		registerDefaultConfiguration(metadata, registry);
        // 2、扫描并注册FeignClient
		registerFeignClients(metadata, registry);
	}
    
    public void registerFeignClients(AnnotationMetadata metadata,
                                     BeanDefinitionRegistry registry) {
        // 3、获取注解扫描器
		ClassPathScanningCandidateComponentProvider scanner = getScanner();
		scanner.setResourceLoader(this.resourceLoader);

		Set<String> basePackages;

        // 4、读取@EnableFeignClients#clients属性
		Map<String, Object> attrs = metadata
				.getAnnotationAttributes(EnableFeignClients.class.getName());
		final Class<?>[] clients = attrs == null ? null
				: (Class<?>[]) attrs.get("clients");
        
        // 5、如果@EnableFeignClients没配置clients属性，则配置@FeignClient注解扫描过滤器
        AnnotationTypeFilter annotationTypeFilter = new AnnotationTypeFilter(
            FeignClient.class);
		if (clients == null || clients.length == 0) {
			scanner.addIncludeFilter(annotationTypeFilter);
            // 5.1、从@EnableFeignClients配置的value、basePackages、basePackageClasses，以及springbootApplication启动类包中，选出要扫描的包
			basePackages = getBasePackages(metadata);
		}
        // 6、否则，@EnableFeignClients#clients属性所在的包名加入basePackages
		else {
			...
		}

        // 7、扫描basePackages下的注解
		for (String basePackage : basePackages) {
			Set<BeanDefinition> candidateComponents = scanner
					.findCandidateComponents(basePackage);
			for (BeanDefinition candidateComponent : candidateComponents) {
				if (candidateComponent instanceof AnnotatedBeanDefinition) {
					AnnotatedBeanDefinition beanDefinition = (AnnotatedBeanDefinition) candidateComponent;
					AnnotationMetadata annotationMetadata = beanDefinition.getMetadata();
					Assert.isTrue(...);

                    // 8、扫描basePackages下的@FeignClient组件
					Map<String, Object> attributes = annotationMetadata
							.getAnnotationAttributes(
									FeignClient.class.getCanonicalName());

                    // 9、获取@FeignClient#contextId、value、name、serviceId属性
					String name = getClientName(attributes);
                    
                    // 10、注册@FeignClient#configuration
					registerClientConfiguration(registry, name,
							attributes.get("configuration"));

                    // 11、注册 FeignClientFactoryBean
					registerFeignClient(registry, annotationMetadata, attributes);
				}
			}
		}
    }
}
```

##### 3、注册 FeignClientFactoryBean

```java
class FeignClientsRegistrar
    implements ImportBeanDefinitionRegistrar, ResourceLoaderAware, EnvironmentAware {
    
	private void registerFeignClient(BeanDefinitionRegistry registry,
			AnnotationMetadata annotationMetadata, Map<String, Object> attributes) {
        ...
        // 1、获取FeignClientFactoryBean#BeanDefinitionBuilder
        BeanDefinitionBuilder definition = BeanDefinitionBuilder
            .genericBeanDefinition(FeignClientFactoryBean.class);
		...
        // 2、构造FeignClientFactoryBean#BeanDefinition
		AbstractBeanDefinition beanDefinition = definition.getBeanDefinition();
        BeanDefinitionHolder holder = new BeanDefinitionHolder(beanDefinition, className, new String[] { alias });
        // 3、注册FeignClientFactoryBean
		BeanDefinitionReaderUtils.registerBeanDefinition(holder, registry);
	}
}
```

##### 4、注入 FeignClient

```java
class FeignClientFactoryBean
    implements FactoryBean<Object>, InitializingBean, ApplicationContextAware {
    // 1、@Autowire在注入时调用FactoryBean#getObject
	@Override
	public Object getObject() throws Exception {
		return getTarget();
	}

    <T> T getTarget() {
        // 2、获取SPI#FeignAutoConfiguration注入的FeignContext Bean实例
        FeignContext context = this.applicationContext.getBean(FeignContext.class);
        
        // 3、根据上下文构造Feign.Builder
        Feign.Builder builder = feign(context);
        
        // 4、@FeignClient#uri属性为空
		if (!StringUtils.hasText(this.url)) {
            // 5、则追加http+@FeignClient#name
			if (!this.name.startsWith("http")) {
				this.url = "http://" + this.name;
			}
			else {
				this.url = this.name;
			}
			this.url += cleanPath();
            
            // 6、对已注入的FeignClient进行代理
			return (T) loadBalance(builder, context,
					new HardCodedTarget<>(this.type, this.name, this.url));
		}
        // 4、否则使用uri对已注入的FeignClient进行代理
    }

    protected <T> T loadBalance(Feign.Builder builder, FeignContext context,
			HardCodedTarget<T> target) {
        // 7、获取已注入的LoadBalancerFeignClient实例 => DefaultFeignLoadBalancedConfiguration自动装配注入
		Client client = getOptional(context, Client.class);
		if (client != null) {
			builder.client(client);
            // 8、获取代理生产类HystrixTargeter => SPI#FeignAutoConfiguration注入
			Targeter targeter = get(context, Targeter.class);
            // 9、获取FeignClient目标代理
			return targeter.target(this, builder, context, target);
		}

		throw new IllegalStateException(...);
	}
    
    // 根据上下文构造Feign.Builder
	protected Feign.Builder feign(FeignContext context) {
        ...
        // 3.1、获取Feign.Builder实例 => FeignClientsConfiguration自动装配注入
		Feign.Builder builder = get(context, Feign.Builder.class)
            // 3.2、配置当前类作为日志记录者logger 
            .logger(logger)
            // 3.3、配置Encoder实例SpringDecoder => FeignClientsConfiguration自动装配注入，Missing Encoder Bean 时默认注入（可扩展修改）
            .encoder(get(context, Encoder.class))
            // 3.4、配置Decoder实例SpringEncoder => FeignClientsConfiguration自动装配注入，Missing Decoder Bean 时默认注入
            .decoder(get(context, Decoder.class))
       // 3.5. 配置Contract实例SpringMvcContract，用于限定 @FeignClient 的配置规则 => FeignClientsConfiguration自动装配注入，Missing Contract Bean 时默认注入（可扩展修改）
            .contract(get(context, Contract.class));
		// 3.6 配置Feign，比如RequestInterceptor
        configureFeign(context, builder);
        // 3.7. 返回Feign Builder
		return builder;
	}
}
```

##### 5、获取 FeignClient 目标代理

```java
class HystrixTargeter implements Targeter {

	@Override
	public <T> T target(FeignClientFactoryBean factory, Feign.Builder feign,
                        FeignContext context, Target.HardCodedTarget<T> target) {
        // 1、Feign.Builder实例 => FeignClientsConfiguration自动装配注入
		if (!(feign instanceof feign.hystrix.HystrixFeign.Builder)) {
            // 2、进入Feign代理逻辑
			return feign.target(target);
		}
        ...// 1、否则进入Hystrix熔断逻辑
    }
}

public abstract class Feign {
    ...
    public static class Builder {
        public <T> T target(Target<T> target) {
            // 3、构造ReflectiveFeign实例，其中包括设置SynchronousMethodHandler.Factory
      		return build().newInstance(target);
    	}
    }
    ...
}

public class ReflectiveFeign extends Feign { 
  	@Override
    public <T> T newInstance(Target<T> target) {
        // 4、为@FeignClient接口里的方法，使用SynchronousMethodHandler.Factory构建SynchronousMethodHandler实例
        Map<String, MethodHandler> nameToHandler = targetToHandlersByName.apply(target);
        ...
        // 4、为@FeignClient接口设置JDK动态代理处理类FeignInvocationHandler
        InvocationHandler handler = factory.create(target, methodToHandler);
        // 5、创建@FeignClient接口动态代理类
        T proxy = (T) Proxy.newProxyInstance(target.type().getClassLoader(),
        	new Class<?>[] {target.type()}, handler);
        ...
        return proxy;
    }
}
```

#### 服务调用原理

##### 1、FeignInvocationHandler#invoke

```java
public class ReflectiveFeign extends Feign {
    ...
    static class FeignInvocationHandler implements InvocationHandler {
        ...
        // 1、JDK动态代理
    	@Override
        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
           ...
           // 2、调用SynchronousMethodHandler#invoke
           return dispatch.get(method).invoke(args);
        }
    }
}
```

##### 2、SynchronousMethodHandler#invoke

```java
final class SynchronousMethodHandler implements MethodHandler {
    @Override
    public Object invoke(Object[] argv) throws Throwable {
        // 1、获取请求模板 => GET /sayHi HTTP/1.1 Binary data
        RequestTemplate template = buildTemplateFromArgs.create(argv);
        // 2、默认从不重试 => FeignClientsConfiguration#Retryer.NEVER_RETRY注入
        Retryer retryer = this.retryer.clone();
        // 3、开始自旋重试
        while (true) {
            try {
                // 4、执行请求和解码响应
                return executeAndDecode(template);
            } catch (RetryableException e) {
                try {
                    // 5、由于默认不重试，所以重试时会报错
                    retryer.continueOrPropagate(e);
                } catch (RetryableException th) {
                    ...
                }
                // 6、记录日志
                if (logLevel != Logger.Level.NONE) {
                    logger.logRetry(metadata.configKey(), logLevel);
                }
                continue;
            }
        }
    }
}
```

##### 3、SynchronousMethodHandler#executeAndDecode

```java
final class SynchronousMethodHandler implements MethodHandler {
    
    Object executeAndDecode(RequestTemplate template) throws Throwable {
        // 1、请求前拦截
        Request request = targetRequest(template);
		...
        // 2、调用LoadBalancerFeignClient（修饰了feign.Client#Default）发起请求
        response = client.execute(request, options);
        ...
        if (response.status() >= 200 && response.status() < 300) {
            if (void.class == metadata.returnType()) {
                return null;
            } else {
                // 3、响应200，解码（默认为SpringEncoder编码,SpringDecoder解码）
                Object result = decode(response);
                shouldClose = closeAfterDecode;
                return result;
            }
        } else if (decode404 && response.status() == 404 && void.class != metadata.returnType()) {
            // 4、响应404，解码（默认为SpringEncoder编码,SpringDecoder解码）
            Object result = decode(response);
            shouldClose = closeAfterDecode;
            return result;
        } else {
        // 5、ErrorDecoder异常解码，默认为ErrorDecoder.Default，用于抛出RetryableException
            throw errorDecoder.decode(metadata.configKey(), response);
        }
    }

    Request targetRequest(RequestTemplate template) {
        // 1.1、请求前拦截
        for (RequestInterceptor interceptor : requestInterceptors) {
            interceptor.apply(template);
        }
        // 1.2、构造feign.Request实例
        return target.apply(template);
    }
}
```

##### 4、LoadBalancerFeignClient#execute

```java
package org.springframework.cloud.openfeign.ribbon;

public class LoadBalancerFeignClient implements Client {
	@Override
    public Response execute(Request request, Request.Options options) throws IOException {
        ...// 1、组装uri、host和RibbonRequest实例
        // 2、类似于Ribbon调用SpringClientFactory，底层实质调用Spring AbstractApplicationContext#refresh，触发RibbonClientConfiguration注入，然后就有了Ribbon服务列表缓存和其他注入的实例
        IClientConfig requestConfig = getClientConfig(options, clientName);
        // 3、底层构造FeignLoadBalancer（默认修饰Ribbon#ZoneAwareLoadBalancer）
        return lbClient(clientName)
            // 4、调用AbstractLoadBalancerAwareClient#executeWithLoadBalancer
            .executeWithLoadBalancer(ribbonRequest, requestConfig).toResponse();
    }
}

public abstract class AbstractLoadBalancerAwareClient<S extends ClientRequest, T extends IResponse> extends LoadBalancerContext implements IClient<S, T>, IClientConfigAware {
    public T executeWithLoadBalancer(final S request, final IClientConfig requestConfig) throws ClientException {
        ...// RXJava语法略
        return Observable.just(
            // 5、调用FeignLoadBalancer#execute方法
            AbstractLoadBalancerAwareClient.this.execute(requestForServer, requestConfig));
    }
}
```

##### 5、FeignLoadBalancer#execute

```java
public class FeignLoadBalancer extends
    AbstractLoadBalancerAwareClient<FeignLoadBalancer.RibbonRequest, FeignLoadBalancer.RibbonResponse> {
	@Override
	public RibbonResponse execute(RibbonRequest request, IClientConfig configOverride) throws IOException {
		...
        // 1、最终调用feign.Client#execute方法
		Response response = request.client().execute(request.toRequest(), options);
		return new RibbonResponse(request.getUri(), response);
	}
}

public interface Client {
    @Override
    public Response execute(Request request, Options options) throws IOException {
      // 2、建立连接，发送请求
      HttpURLConnection connection = convertAndSend(request, options);
      // 3、把连接和请求转换为响应实例
      return convertResponse(connection, request);
    }
    
    HttpURLConnection convertAndSend(Request request, Options options) throws IOException {
        // 2.1、建立连接
        final HttpURLConnection connection =
            (HttpURLConnection) new URL(request.url()).openConnection();
        ...// 2.2、设置连接超时、读超时等属性
        // 2.3、如果存在请求体，则发送请求体
        if (request.body() != null) {
            ...
			out.write(request.body());
            ...
        }
        ...
        // 2.4、最后返回建立好的连接
        return connection;
    }
    
    Response convertResponse(HttpURLConnection connection, Request request) throws IOException {
        // 3.1、获取响应码
        int status = connection.getResponseCode();
        // 3.2、获取响应内容
        String reason = connection.getResponseMessage();
        // 3.3、构造返回响应实例
        return Response.builder()
            .status(status)
            .reason(reason)
            .headers(headers)
            .request(request)
            .body(stream, length)
            .build();
    }
}
```

