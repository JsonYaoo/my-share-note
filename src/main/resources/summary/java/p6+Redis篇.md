# 五、Redis篇

### 1.1. 缓存原理与分类？

#### 概念

通过开辟一个新的数据交换缓冲区，来解决**原始数据获取代价太大**的问题，以让数据得到更快的访问。
- **狭义缓存**：缓存最初的含义，是指用于**加速 CPU 数据交换**的 RAM，即随机存取存储器，通常这种存储器使用更昂贵。
- **广义缓存**：其定义则更宽泛，任何用于**数据高速交换**的存储介质都可以是缓存，可以是硬件也可以是软件。

#### 原理

通过利用**时间局限性原理**，通过**空间换时间**来达到加速数据获取的目的。

1. **时间局限性原理**：被获取过一次的数据，在未来也可能会被多次引用。
   - 比如一条微博被一个人感兴趣并阅读后，它大概率还会被更多人阅读，当然如果变成热门微博后，会被数以百万/千万计算的更多用户查看。
2. **以空间换时间**：因为原始数据获取太慢，所以开辟了一块高速独立空间，提供高效访问，从而达到数据获取加速的目的。

#### 优势

1. **提升访问速度**：缓存存储了 DB 关键数据，数据存在缓存时，无需从 DB 获取，大大提升了访问速度。
   - 一般来讲，服务系统的全量原始数据存储在 DB 中（如 MySQL、HBase 等），所有数据的读写都可以通过 DB 操作来获取。
   - 但 DB 读写性能低、延迟高，如 MySQL 单实例的读写 QPS 通常只有千级别（线上可以到 **3000～6000**），读写平均耗时 **10～100ms** 级别，如果一个用户请求需要查 20 个不同的数据来聚合，仅仅 DB 请求就需要数百毫秒甚至数秒。
   - 而缓存读写性能高的特点，正好可以弥补 DB 的不足，比如 Memcached 的读写 QPS 可以达到 **10～100 万**级别，读写平均耗时在 **1ms** 以下，Redis 读写 QPS 也可以达到 **10万** 级别，结合并发访问技术，单个请求即便查上百条数据，也可以轻松应对。
2. **降低网络拥堵**：由于减少了频繁访问数据库的网络流量，降低了网络拥堵。
3. **减轻服务负载**：由于减少了解析和计算，调用方和存储服务的负载也可以大幅降低。
4. **增强可扩展性**：缓存的读写性能很高，预热快，在数据访问存在性能瓶颈或遇到突发流量，系统读写压力大增时，可以快速部署上线，同时在流量稳定后，也可以随时下线，从而使系统的可扩展性大大增强。

#### 代价

1. **增加了系统复杂度**：服务系统中引入缓存，会增加系统的复杂度。

2. **存在数据不一致**：由于一份数据同时存在缓存和 DB 中，甚至缓存内部也会有多个数据副本，多份数据就会存在一致性问题，同时，缓存体系本身也会存在可用性问题和分区的问题。

3. **缓存容量小**：只能存储部分访问频繁的热数据。

4. **部署成本比 DB 高**：由于缓存相比原始 DB 存储的成本更高，所以系统部署及运行的费用也会更高。

   - 由于缓存空间的成本较高，在实际设计架构中，还要考虑访问**读写延迟和成本**的权衡问题。

     - 系统的访问性能越高越好，访问延迟越低小越好，但要维持相同数据规模的存储及访问，性能越高延迟越小，成本也会越高，所以，在系统架构设计时，需要在系统性能和开发运行成本之间做取舍。
     - 比如相同成本的容量，SSD 硬盘容量会比内存大 10～30 倍以上，但读写延迟却高 50～100 倍。

     ![1631789780363](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1631789780363.png)

#### 分类

##### 按宿主层次分类

- **浏览器缓存**：指客户端把服务器请求或响应资源缓存在本地，以加速用户访问，提升单个用户的体验。
  - **浏览器本地内存缓存：**专题活动，一旦上线，在活动期间是不会随意变更的。
  - **浏览器本地磁盘缓存：**Logo缓存，大图片懒加载。
- **CDN缓存**：CDN，Content Delivery Network，即内容分发网络，而CDN缓存指的是，CDN边缘节点将数据缓存起来，使得用户能够**就近获取**所需内容，降低网络拥塞，提高用户访问响应速度和命中率。
- **反向代理缓存**：比如Nginx缓存，可以提升访问上游服务器的速度。
- **本地缓存**：指服务器把数据缓存到本地，性能非常高，但会占用堆内存，影响垃圾回收、影响系统性能，且由于由于没有被持久化，重启后必定会被穿透。
  - 为什么不使用服务器本地磁盘做缓存？是因为当系统处理大量磁盘 I/O 操作的时候，由于 CPU 和内存的速度远高于磁盘，可能导致 CPU 耗费太多时间，来等待磁盘返回处理的结果，对于这部分 CPU 在 I/O 上的开销，称为 **I/O Wait**。
- **分布式缓存**：跨服务器建立通用的缓存系统，可以减轻数据库压力、提升响应速度和分布式处理数据。
  - 比如 Redis 等，针对穿透的情况，可以继续做缓存分层，必须保证数据库不被压垮。

##### 按存储介质分类

- **内存型缓存**：将数据存储在内存，读写性能很高，但缓存系统重启或 Crash 后，内存数据会丢失。
- **可持久化型缓存**：将数据存储到硬盘中，在相同成本下，这种缓存的容量会比内存型缓存大 1 个数量级以上，而且数据会持久化落地，重启不丢失，但读写性能相对低 1～2 个数量级。
  - 比如 Memcached 就是典型的内存型缓存，而 Redis 则属于可持久化型缓存。

### 1.2. 为什么使用Redis？

Redis，是开源的，数据结构存储于内存中，被用来作为数据库，缓存和消息代理。

- 支持多种数据结构，例如字符串（string）、哈希（hash）、列表（list）、集合（set）、带范围查询的排序集合（zset）、位图（bitmap）、hyperloglog、带有半径查询和流的地理空间索引。
- 具有内置的复制、Lua脚本、LRU逐出、事务和不同级别的磁盘持久性。
- 通过Redis Sentinel和Redis Cluster自动分区提供高可用性。

#### 读写速度快（Redis为什么这么快？）

1. **完全基于内存操作**：数据存在内存中，机器访问内存的速度远远大于访问磁盘的速度。

2. **采用单线程架构**：避免了多线程不必要的上下文切换和竞争条件，不存在加锁释放锁操作，减少了因为锁竞争导致的性能消耗。

   - **线程上下文切换场景**：
      - **抢占式**：一般跟锁竞争有关，可以减少锁争用，来减少线程上下文切换。
      - **时间片轮转**：一般跟时间片有关，可以减少线程数，来减少线程上下文切换。

   - **Redis 6.0版本支持多线程**：仍然使用单线程处理命令，但是读写网络数据使和协议解析使用了多线程。

      - **背景**：从 Redis 自身角度来说，因为读写网络的 Read/Write 系统调用占用了 Redis 执行期间大部分 CPU 时间，瓶颈主要在于**网络的 IO 消耗**。

      - **目的**：支持多线程可以充分利用服务器 CPU 资源，分摊 Redis 同步 IO 读写负荷。

      - **使用**：默认关闭，可以在 `io-threads-do-reads` 配置中打开，建议线程数一定要小于CPU核数。

      - **效果**：不严谨测试结论为，对比单线程性能提升**翻倍**。

      - **实现机制**：IO 线程只负责读写 Socket 解析命令，不负责命令处理，而是把命令交给主线程执行。

        1. 主线程负责接收建立连接请求，获取 Socket 放入到全局等待读处理队列。
        2. 主线程处理完读事件之后，通过 RR（Round Robin）将这些连接分配给这些 IO 线程。
        3. 主线程阻塞，等待 IO 线程读取 Socket 完毕。
        4. 主线程通过单线程的方式，执行请求命令，请求数据读取并解析完成，但并不回写。
        5. 主线程阻塞，等待 IO 线程将数据回写 Socket 完毕。
        6. 解除绑定，清空等待队列。

        => 可以看到，Redis 6.0 多线程部分只是用来处理网络数据的读写和协议解析，执行命令仍然是单线程顺序执行，因此无需去考虑控制 Key、Lua、事务、LPUSH/LPOP 等等的并发线程安全问题。

        ![1632104148793](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1632104148793.png)

3. **使用I/O多路复用模型**：

   - **I/O交互方式**：
     - **BIO**：同步阻塞，早期传统的阻塞模型，一个连接对应一个线程，开销非常大。
     - **NIO**：同步非阻塞，在一个连接被创建时，Kernel会创建一个socket文件描述符fd，并提供非阻塞的select、poll、epoll多路复用器。
       - **select**：每次都会返回所有的fd，需要用户程序遍历全部的fd，然后找到有数据的fd进行读写，效率较低，最多只能监听1024个连接。
       - **poll**：实现和select基本一样，不过poll突破了1024个连接的监听限制。
       - **epoll**：epoll会返回有读写状态的fd，不再需要用户程序去全量遍历查找，epoll_create创建socket文件描述符fd，epoll_ctl注册事件，epoll_wait等待事件的fd。
     - **AIO**：异步非阻塞，无需一个线程去轮询所有I/O操作的状态改变，在I/O状态发生改变后，操作系统会通知对应的线程来处理。
   - **多路复用执行过程**：文件事件处理器是单线程的，采用多路复用的方式，来监听系统上多个socket，将socket上产生的事件压入队列中，由文件事件分派器从队列中取出一个socket，并根据事件类型发给相应的事件处理器。
     1. 客户端发起请求，向redis的server socket请求连接，这里命名为socket01。
     2. server socket产生一个**AE_READABLE事件**，IO多路复用程序监听到事件后，将这个socket01压入队列。
     3. 文件事件分派器从队列中取出socket01，交给连接应答处理器，连接应答处理器会将socket01的AE_READABLE事件与命令请求处理器相关联，然后压入队列中。
     4. 客户端执行set操作，此时命令请求处理器会从socket01读取key value，并在内存中完成key value的设置，接着在内存中完成设置后，会将socket01的**AE_WRITEABLE事件**与命令回复处理器相关联，然后压入队列中。
     5. 事件分派器拿到socket01后，交给命令回复处理器，由命令回复处理器向socket01写入本次操作的结果，比如OK，之后解除事件关联。

   ![1631793891157](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1631793891157.png)

4. **基于C语言开发**：直接跟操作系统交互，命令执行飞快。

5. **使用专门设计的数据结构**：对string、list、set、zset、hash都专门设计了数据结构，使得某些操作的性能有所提升，比如获取string的长度，使用sds的时间复杂度为O（1），而c字符串的为O（n）。

#### 数据结构丰富

Redis不仅仅支持简单的key-value类型的数据结构，同时还提供了list，set，zset，hash等数据结构。

#### 支持持久化

Redis提供了RDB和AOF两种持久化策略，能最大限度地保证Redis服务器宕机重启后数据不会丢失。

#### 支持高可用

Redis可以使用主从模式、哨兵模式以及集群模式，来保证服务器的高可用。

#### 客户端语言多

由于Redis受到社区和各大公司的广泛认可，所以客户端语言涵盖了所有的主流编程语言，比如Java，C，C++，PHP，NodeJS等等。

#### 竞品对比

| 产品               | 优点                                                        | 缺点                                                         | 持久化        | 高可用               | 使用场景                                 |
| ------------------ | ----------------------------------------------------------- | ------------------------------------------------------------ | ------------- | -------------------- | ---------------------------------------- |
| Redis              | 单线程、支持K-V以及多种数据结构存储、支持持久化，适合热数据 | 容量受内存限制，不便于海量数据读写，不适合冷数据             | RDB、AOF      | 主从、哨兵、集群     | 缓存、最新回复、点赞数、共同好友、排行榜 |
| Memcahe            | 多线程、性能高、速度快，用于减轻数据库负载                  | 不支持持久化、只能存储K-V数据                                | 不支持        | 集群没有同步复制机制 | 前端缓存、用户信息、好友信息、文章信息   |
| Tair               | 淘宝开源，多线程、支持K-V以及多种数据结构存储、支持持久化   | -                                                            | MDB、RDB、LDB | 集群可以灵活配置     | 适合作为大数据量缓存                     |
| EvCache            | Netflix基于Memcached 实现的缓存方案，性能很高               | -                                                            | 支持          | 支持                 | 适合对强一致性没有必须要求的场合         |
| Google Guava Cache | 本地缓存，性能非常高                                        | 会占用堆内存，影响垃圾回收、影响系统性能；每台JVM都有自己的本地缓存，没有分布式一致性可言 | -             | -                    | 本地缓存                                 |

##### Memcache

Memcache，是一个高性能的分布式内存对象缓存系统，通过在内存里维护一个统一的**巨大的hash表**，用来存储各种格式的数据，包括图像、视频、文件以及数据库检索的结果等，数据存储在内存中，然后直接从内存中读取，从而大大提高读取速度。

- **使用场景**：
  1. 如果有持久方面的需求，或对数据类型和处理有要求的，应该选择redis。 
  2. 如果简单的key-value存储的，应该选择memcached。	

| redis                                 | Memcached                    |
| ------------------------------------- | ---------------------------- |
| 内存高速数据库                        | 高性能、分布式内存缓存数据库 |
| 支持hash、list、set、zset、string结构 | 只支持key-value结构          |
| 将大部分数据放到内存                  | 全部数据放到内存中           |
| 支持持久化、主从复制备份              | 不支持数据持久化及数据备份   |
| 数据丢失可通过AOF恢复                 | 挂掉后，数据不可恢复         |
| 单线程（2~4万TPS）                    | 多线程（20-40万TPS）         |

##### Tair

Tair，Taobao Pair，是淘宝开发的分布式Key-Value存储引擎，默认支持基于内存和文件的两种存储方式，分别与缓存和持久化存储对应，既可以做缓存，也可以做数据源。

- **三种引擎切换**：
  - **MDB**：基于Memcache，，属于内存型产品，支持KV和类HashMap结构，性能最优，不支持持久化存储。
  - **RDB**：基于Redis，支持List、Set、Zset等复杂的数据结构，性能次之，可提供缓存以及持久化存储两种模式。
  - **LDB**： 基于Google LevelDB，属于持久化产品，支持KV和类Hashmap结构，性能稍低，但持久化可靠性最高。
- **痛点**：在Redis集群中，如果程序想借用缓存资源，则必须得指明redis服务器地址去获取，增加了程序维护的复杂度，因为Redis服务器很可能是频繁变动的。
  - **中心化管理**：Tair使用中心节点代理缓存集群，借用Tair资源的程序只需要跟该中心节点交互，无需频繁更改服务器地址。同时，Tair还有服务器配置的功能，使得在改集群配置文件时，不需要一个个机器地去修改。

##### EVCache

EVCache，Ephemeral Volatile Cache，短暂易失缓存，是一个Netflflix开源的，基于Memcached实现的分布式缓存，用以构建超大容量、高性能、低延时、跨区域全球可用的缓存数据层，适合对**强一致性没有必须要求**的场合。

- **特性**：分布式键值存储、AWS跨域复制存储数据、注册和自动发现新节点或者新服务。

##### Google Guava Cache

- **Google Guva**：是google开源的一个公共java库，类似于Apache Commons，它提供了集合，反射，缓存，科学计算，xml，io等一些工具类库，而cache只是其中的一个模块，使用Guva cache能够方便快速的构建本地缓存。
- **Google Guava Cache**：是一种非常优秀本地缓存解决方案，提供了基于容量，时间和引用的缓存回收方式。
  - 缓存核心类LocalCache里面的内部类Segment，与jdk1.7及以前的ConcurrentHashMap非常相似，都继承于ReetrantLock，还有六个队列，以实现丰富的本地缓存方案。
- **优点**：
  - 作用在LocalCache上，相对于IO操作速度快，性能非常高效。
  - 对于Redis等分布式缓存，它们受限于网络IO、吞吐率以及缓存的数据大小等原因，远水救不了近火，而DB + Redis + LocalCache可以实现高效存储、高效访问。
- **缺点**：
  - 会占用堆内存，影响垃圾回收、影响系统性能。
  - 对于缓存一致性方面，每台JVM都有自己的本地缓存，没有分布式一致性可言；而使用分布式缓存则更好一点，因为集群环境下的节点都使用同一份缓存。
- **使用场景**：
  - 对性能有非常高的要求时。
  - 缓存数据需要不经常变化，且占用内存不能太大时。
  - 整个集合都需要被访问时。
  - 数据允许不实时一致时。
- **使用方法**：

```java
// 5. 构造LocalCache
private final LoadingCache<String,List<SysDictItem>> vendorDictItemCache = CacheBuilder.newBuilder()
    // 1. 设置缓存容量
    .maximumSize(1000)
    // 2. 设置超时时间
    .expireAfterWrite(10, TimeUnit.SECONDS)
    // 3. 设置缓存Key失效监听器
    .removalListener((RemovalListener<String, List<SysDictItem>>) notification -> {
        LOGGER.warn(notification.getKey() + "缓存已失效, 失效原因为: " + notification.getCause().name());
    })
    // 4. 提供缓存加载器 -> 缓存不存在时，会去查找数据库来设置缓存
    .build(new CacheLoader<String, List<SysDictItem>>() {
        @Override
        public List<SysDictItem> load(String code) {
            return getDictItemsByCodeAndTypeInDB(code);
        }
    });

// 6. 获取缓存
sysDictItems = vendorDictItemCache.get(dictCode);
```

- **自己设计本地缓存痛点：**
  - **并发处理能力差**：针对并发可以使用CurrentHashMap，但缓存的其他功能需要自行实现。
  - **缓存处理**：需要根据一定的规则进行淘汰数据，比如LRU、LFU、FIFO等，缓存加载和刷新都需要手工实现。
  - **回调通知实现**：清除数据时，需要触发回调通知，同样需要自己实现。
- **使用Google Guava Cache的优势**：
  - **并发处理能力**：
    - Google  Guava Cache类似CurrentHashMap，是线程安全的，提供了设置并发级别的API，使得缓存支持并发的写入和读取，采用分段锁机制，以减小锁力度，提升并发能力。
  - **缓存过期和淘汰机制**：在Google  Guava Cache中，可以设置Key的过期时间，包括访问过期和创建过期，会在缓存容量达到指定大小时，采用LRU的方式，将不常使用的键值从缓存中删除。
  - **防止缓存击穿**： Google  Guava Cache可以在CacheLoader#load方法中加以控制，对同一个key，只让一个请求去读源数据并回填缓存，其他请求阻塞等待，相当于集成了数据源，方便用户使用。
  - **监控统计能力**：另外，还提供了缓存加载以及命中情况统计信息的API。

##### 总结

| 对比               | Redis优势                             |
| ------------------ | ------------------------------------- |
| Memcache           | Redis支持多种数据结构存储、支持持久化 |
| Tair               | 业务量不大时，Redis简单高效、实用性好 |
| EvCache            | Redis社区活跃、使用最多               |
| Google Guava Cache | Redis缓存统一存储，分布式一致性好点   |

### 1.3. Redis数据类型与底层实现？

Redis，是一个Key-Value型的内存数据库，它所有的key都是字符串，而value常见的数据类型有五种：**String、Hash、List、Set、Zset**。

![1631850908728](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1631850908728.png)

#### 基本类型API总结

http://redisdoc.com/、https://redis.io/topics/streams-intro

| String                    | Hash                  | List                     | Set                 | Zset                                    | Stream（5.0版本）        |
| ------------------------- | --------------------- | ------------------------ | ------------------- | --------------------------------------- | ------------------------ |
| SET、GET、GETSET          | HSET、HSETNX          | LPUSH、LPUSHHX、LPOP     | SADD                | ZADD                                    | XADD                     |
| SETNX、SETEX、PSETEX      | HGET、HGETALL         | RPUSH、RPUSHX、RPOP      | SISMEMBER           | ZINCRBY、ZSCORE                         | XLEN                     |
| APPEND                    | HEXSITS               | RPOPLPUSH                | SPOP、SRANDMEMBER   | ZCARD、ZCOUNT                           | XRANGE、XREVRANGE        |
| STRLEN                    | HDEL                  | LREM、LINSERT            | SREM、SMOVE         | ZRANGE、ZREVRANGE                       | XREAD、XREAD BLOCK       |
| SETRANGE、GETRANGE        | HLEN                  | LINDEX、LSET             | SCARD               | ZRANGEBYSCORE、ZREVRANGEBYSCORE         | XGROUP、XREADGROUP、XACK |
| INCR、INCRBY、INCRBYFLOAT | HSTRLEN               | LRANGE、LTRIM            | SMEMBERS            | ZRANK、ZREVRANK                         | XPENDING                 |
| DECR、DECBY               | HINCRBY、HINCRBYFLOAT | BLPOP、BRPOP、BRPOPLPUSH | SINTER、SINTERSTORE | ZREM、ZREMRANGEBYRANK、ZREMRANGEBYSCORE | XCLAIM、XAUTOCLAIM       |
| MSET、MSETNX、MGET        | HMSET、HMGET          |                          | SUNION、SUNIONSTORE | ZRANGEBYLEX、ZLEXCOUNT、ZREMRANGEBYLEX  | XINFO                    |
|                           | HKEYS、HVALS          |                          | SDIFF、SDIFFSTORE   | ZINTERSTORE、ZUNIONSTORE                | XTRIM、XDEL              |

#### 基本类型使用场景总结

| 类型              | 说明     | 使用场景                                       |
| ----------------- | -------- | ---------------------------------------------- |
| String            | 字符串   | 帖子、评论、热点数据、输入缓冲                 |
| Hash              | 哈希表   | 结构化数据，比如存储对象                       |
| List              | 列表     | 评论列表、商品列表、发布与订阅、慢查询、监视器 |
| Set               | 集合     | 交集、并集、差集，比如朋友关系                 |
| Zset              | 有序集合 | 去重后排序，适合排名场景                       |
| Stream（5.0版本） | 流       | 消息队列                                       |

#### Redis对象

```c
// Redis对象
typedef struct redisObjet {
    // 对象类型，取值范围有：REDIS_STRING、REDIS_HASH、REDIS_LIST、REDIS_SET、REDIS_ZSET
    unsigned type:4;
    // 对象编码，取值范围有：REDIS_ENCODING_INT、REDIS_ENCODING_EMBSTR、REDIS_ENCODING_RAW、REDIS_ENCODING_HT、REDIS_ENCODING_LINKEDLIST、REDIS_ENCODING_ZIPLIST、REDIS_ENCODING_INTSET、REDIS_ENCODING_SKIPLIST
    unsigned encoding:4;
    // 指向底层实现的数据结构的指针
    void *ptr;
    ...
}
```

使用Redis对象来表示key和Value，即每新建一个键值对，至少会创建有两个对象，而使用对象具有以下**好处**：

- **命令是否可执行判断**：在执行命令前，可以根据对象的类型来判断，一个对象是否可以执行该命令。
- **优化不同场景下的使用效率**：针对不同的使用场景，为对象设置不同的数据结构实现，从而优化对象的不同场景下的使用效率。
- **基于引用计数的内存回收**：可以基于引用计数的内存回收机制，自动释放对象所占用的内存。
- **对象内存共享**：可以让多个Key共享同一个对象，从而节约内存。
- **根据空转时长淘汰对象**：对象带有访问的时间记录信息，使用该信息可以进行优化空转时长较大的Key，从而进行删除。

#### Redis数据结构

Redis对象的**ptr指针**，指向对象底层实现数据结构，而这些数据结构由对象的**encoding属性**来决定，其**对应关系**为：

| RedisObject#encoding      | ptr指向的数据结构               | RedisObject#type                   |
| ------------------------- | ------------------------------- | ---------------------------------- |
| REDIS_ENCODING_INT        | redisObject + long型整数        | REDIS_STRING                       |
| REDIS_ENCODING_EMBSTR     | embstr（redisObject+优化的sds） | REDIS_STRING                       |
| REDIS_ENCODING_RAW        | raw（redisObject+sds）          | REDIS_STRING                       |
| REDIS_ENCODING_HT         | dict                            | REDIS_HASH、REDIS_SET              |
| REDIS_ENCODING_LINKEDLIST | linkedlist                      | REDIS_LIST                         |
| REDIS_ENCODING_ZIPLIST    | ziplist                         | REDIS_HASH、REDIS_LIST、REDIS_ZSET |
| REDIS_ENCODING_INTSET     | intset                          | REDIS_SET                          |
| REDIS_ENCODING_SKIPLIST   | skiplist+dict                   | REDIS_ZSET                         |

##### sds

```java
// 简单动态字符串
struct sdshdr{ 
  // 记录buf数组中已使用字节的数量，即实际使用的字节数 
  int len;
  // 记录buf数组中未使用字节的数量
  int free;
  // 字符数组，用于保存字符串
  char buf[];
}
```

![1631870815998](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1631870815998.png)

- **概念**：sds，simple dynamic String，**简单动态字符串**，是Redis自己实现的一个字符串数据结构，在Redis中所有场景中，出现的字符串基本都是由SDS来实现的，包括所有的Key、非数字值、字符串类型的值。

- **特点**：

  - **空间预分配**：在进行修改之后，sds会对接下来可能需要的空间进行预分配，使用free属性来记录当前预分配了多少空间，来减少修改字符串带来的内存重分配次数，其分配策略如下：
    - 如果当前sds的长度小于1M，则分配等于len长的free空间。
    - 如果当前sds的长度大于1M，则分配1M的free空间。
  - **惰性释放内存**：为避免缩短字符串时的内存重分配操作，sds在数据减少时，并不会立刻释放空间，而是暂时留着，以备下次进行增长时使用。
    - 对于内存紧张的机器，sds也提供了对应的API，可以在需要的时候，来释放掉多余的未使用空间。
  - **SSD限制512M**：由于sds大小限制512M，所以Redis的Key以及字符串数据结构的值，最大大小也为 512M。

- **对比C字符串**：

  | SSD函数                | C字符串                                                      | SDS                                                  |
  | ---------------------- | ------------------------------------------------------------ | ---------------------------------------------------- |
  | 高性能获取字符串长度   | 由于需要遍历整个字符数组，所以获取字符串长度需要O（N）       | 由于记录了len，所以获取字符串长度只需要O（1）        |
  | 杜绝字符数组缓冲区溢出 | 字符数组空间不会自动扩展，容易造成缓冲区溢出                 | 会先检查free是否足够，来自动扩展空间，避免缓冲区溢出 |
  | 减少内存重分配次数     | 每次修改字符串长度，都需要内存重新分配                       | 空间预分配、惰性释放内存，但最坏情况下，同C字符串    |
  | 二进制安全             | 由于使用空间符'0'来判断一个字符串的结尾，所以只能保存纯文本，非二进制安全 | 二进制安全，可以保存任意格式的二进制数据             |
  | 部分兼容C库函数        | 可以无缝使用所有C库函数                                      | 只兼容部分的C库函数                                  |

##### int

![1631878723017](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1631878723017.png)

long类型的整数，占8个字节。

##### embstr

![1631878253168](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1631878253168.png)

embstr，embstr编码的简单动态字符串，是对sds的一个小优化，将redisObject对象头和sds对象连续存在一起，一旦两者整体大小大于64字节时，Redis则认为是一个大字符串（即字符串为44字节时，3.2版本前是39字节），然后将会将字符串转为raw进行存储。

- 当使用sds时，程序需要调用两次内存分配，redisObject和sds各自分配一块空间；而由于embstr需要的空间很少，可以采用**连续的空间保存**，只需要一次内存分配，将sds的值和字符串对象的值放在一块连续的内存空间上，由于内存是连续的，减少了很多内存碎片和指针内存的占用，进而节约了内存，提高短字符串的内存分配效率和空间利用率。
- 另外，embstr是只读的形式，Redis并未对其提供任何修改的方式，因此，embstr需要转换为RAW才能进行修改。

##### raw

![1631878637658](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1631878637658.png)

raw，简单动态字符串，以sds形式存储，主要为了解决长度计算和追加字符效率的问题。

##### dict

```c
// 字典，持有两个哈希表，只是对hashtable做了一层封装
typedef struct dict{
  // 类型特定函数，配合*private以实现字典多态
  dictType *type;
  // 私有数据，配合*type以实现字典多态
  void *private;
  // 哈希表
  dictht ht[2];
  // rehash索引，当当前的字典不在rehash时，值为-1
  int trehashidx;
}

// 哈希表，是字典条目的数组
typedef struct dictht{
  // 哈希表的数组
  dictEntry **table;
  // 哈希表的大小
  unsigned long size;
  // 哈希表的大小的掩码，用于计算索引值，总是等于 size-1
  unsigned long sizemasky;
  // 哈希表中已有的节点数量
  unsigned long used;
}

// 字典条目，持有Key和Value
typedef struct dictEntry{
  // 键
  void *key;
  // 值
  union {
    void *val;
    uint64_tu64;
    int64_ts64;
  } v;

  // 指向下一个节点的指针
  struct dictEntry *next;
} dictEntry;
```

![1631878774943](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1631878774943.png)

dict，字典，一种能够存储键值对的数据结构，在Redis中的字典实现中，它持有两张哈希表，一个为null，另一个存储实际键值对，在rehash时允许同时存在键值对。

- **hash算法**：在哈希表添加一个元素时，需要计算该键值的hash值，之后根据其hash值来定位被放入的槽，为了减少哈希冲突的发生，需要将key值打散的足够均匀，此时hash算法的选择尤其重要。Redis 选用了业内计算性能好的算法来实现hash过程：
  - **Redis 5.0 & 4.0版本**：siphash哈希算法，可以在输入的Key值很小的情况下，产生随机性比较好的输出。
  - **Redis 3.2、3.0 & 2.8版本**：Murmurhash2哈希算法，可以在输入值是有规律时，也能给出比较好的随机分布。
- **hash冲突**：hash算法计算结束之后，会根据当前哈希表的长度，来确定当前键值所在的index，而由于长度有限，迟早会产生两个键值要放到同一个位置的问题，即产生hash冲突。
  - **解决方案**：Redis的哈希表处理Hash冲突的方式，和Java中的HashMap一样，即链地址法，Hash表有两维，第一维度是个数组，第二维度是个链表，当发生了hash冲突的时，会将冲突的节点使用链表连接起来，放在同一个桶内。
    - **缺点**：由于第二维度是链表，如果hash冲突比较严重，导致单个链表过长，那么此时hash表的查询效率就会急速下降。
- **扩容与缩容**：当哈希表过于拥挤，查找效率就会下降，需要进行扩容操作；当哈希表过于稀疏，对内存就有点太浪费了，需要进行缩容操作。
  - **负载因子**：用于描述哈希表当前被填充的程度，计算公式是：`负载因子= 哈希表已保存的节点数量 / 哈希表的大小`，在Redis实现里，扩容缩容有三条规则：
    1. 当没有正在执行的`BGSAVE`和`BGREWRITEAOF`指令时，且**负载因子 >= 1**，才会对哈希表进行扩容。
    2. 当存在正在执行`BGSAVE`和`BGREWRITEAOF`指令时，且**负载因子 >= 5**，才会对哈希表进行扩容。
       - 这是因为在进行BGSAVE操作时，会存在**子进程**，Redis会尽量避免在存在子进程时进行扩容，以节省内存。
    3. 无论存不存`BGSAVE`和`BGREWRITEAOF`指令，只要**负载因子 < 0.1**时，会自动开始对哈希表进行缩容。
       - 而缩容过程中，由于申请的内存比较小，同时还会释放掉一些已经使用的内存，不会增大系统的压力，因此不需要在缩容时考虑是否正在进行`BGSAVE`和`BGREWRITEAOF`操作。
  - **扩容的目标数量**：第一个大于等于`ht[0].used * 2`的`2^n`，初始为4。
  - **缩容的目标数量**：第一个大于等于`ht[0].used`的`2^n`。
- **渐进式rehash**：在扩缩容期间，需要将当前哈希表中的所有节点，重新进行一次hash，即rehash。
  - **对比Java rehash**：
    - 在 Java的HashMap中，实现方式是，新建一个哈希表，一次性的将当前所有节点rehash完成，之后释放掉原有的 hash表，再持有新表。
    - 而Redis不是，由于rehash需要重新定位所有的元素，对数据量很大的字典执行此操作将比较耗时，这对于单线程的Redis说，是很难接受这种延时的，因此，Redis选择使用**一点一点搬的渐进式rehash**策略。
  - **渐进式rehash过程**：
    1. 如果当前数据在ht[0]中，则会首先为ht[1]分配足够的空间。
    2. 在字典中维护一个rehashindex变量，并更新rehashindex = 0，表示当前开始rehash操作。
    3. 在rehash期间，客户端每次对字典进行增删改查操作，在完成实际操作之后，redis都会对该字典进行一次rehash操作，将ht[0]在rehashindex对应位置上的值rehash到ht[1]上，然后把rehashindex 递增一位。
       - 为了让该字典在持有两个哈希表期间，仍然可以对外提供服务，对于添加操作，需要直接添加到 ht[1]上，让ht[0]的数量只会减少不会增加，保证rehash过程可以完结。
       - 而对于删除、修改以及查询操作，可以在ht[0]上进行，如果得不到结果，则会去ht[1]再执行一遍，保证rehash过程的推进。
    4. 随着字典不断被增删改查，原来的ht[0]上的数值会全部rehash完成，此时会将rehashindex置为-1，代表rehash结束。
    5. 而如果服务器很空闲，中间几小时没有被请求过，则Redis定时函数会加入帮助rehash的操作，从而加快rehash的过程。
  - **优点**：采用了分而治之的思想，将rehash操作，分散到每一个对该字典的操作上以及定时函数上，避免了集中式rehash带来的性能压力。
  - **缺点**：在 rehash 的时间内，需要同时持有两个哈希表，对服务器内存的占用稍大，如果此时服务器本来内存就不足时，突然进行的rehash，会使得Redis执行缓存淘汰策略，造成大量的Key被抛弃。

##### linkedlist

```c
// 双向链表
typedef struct list {
  // 表头结点
  listNode *head;
  // 表尾节点
  listNode *tail;
  // 链表所包含的节点数量
  unsigned long len;
  // 其他函数
  ...
} list;

// 双向链表结点
typedef struct listNode{
  // 前置节点
  struct listNode *prev;
  // 后置节点
  struct listNode *next;
  // 节点值
  void *value;
} listNode
```

![1631882244485](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1631882244485.png)

linkedlist，双向链表，由于经常需要使用，Redis自己实现了一个双向链表。

- **双向**：包含了头结点、尾结点，同时每个结点都有自己的前驱和后继，可以很方便地进行正向和反向的遍历。
- **无环**：头结点的prev指针和尾结点的next指针都指向null，是一个无环链表。
- **持有长度计数器**：len记录着当前链表的长度，获取的时间复杂度是O（1）。

##### ziplist

```c
// 压缩链表，底层是个结点数组
struct ziplist<T>{
    // 整个压缩列表占用字节数
    int32 zlbytes;
    // 最后一个节点到压缩列表起始位置的偏移量，可以用来快速定位到压缩列表中的最后一个元素
    int32 zltail_offset;
    // 压缩列表包含的元素个数
    int16 zllength;
    // 元素内容列表，用数组存储，内存上紧挨着
    T[] entries;
    // 压缩列表的结束标志位，值永远为0xFF.
    int8 zlend;
}

// 压缩链表结点
struct entry{
    // 前一个entry的长度，当其在254字节以内时，该值为1字节；否则为5字节
    int<var> prevlous_entry_length;
    // 编码方式，记录着结点content属性所保存数据的类型以及长度
    int<vat> encoding;
    // 内容，真正要保存的数据，类型和长度由encoding决定
    optional bute[] content;
}
```

![1631926865240](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1631926865240.png)

压缩列表，是由一系列连续内存块组成的顺序型数据结构，由于内存是连续的，减少了很多内存碎片和指针内存的占用，进而节约了内存。

- **prevlous_entry_length取值逻辑**：
  - 当前一个节点的长度小于254字节时，`previous_entry_length`长度为1字节。
  - 当前一个节点的长度大于等于254字节时，`previous_entry_length`长度为5字节。
- **遍历列表**：
  - **顺序遍历**：按顺序遍历`entries`数组。
  - **反向遍历**：首先拿到尾部节点的偏移量，找到最尾部的节点，然后调用`prevlous_entry_length`属性，就可以拿到前一个节点，接着不断向前遍历即可。
- **新增结点**：ziplist 是连续存储的数据结构，内存是没有冗余的（SDS中就有冗余空间）, 所以，每一次新增节点，都需要进行内存申请。
  - 如果当前ziplist所在的内存连续块够用，则将新节点添加即可。
  - 但如果申请到的是另外一块连续的内存空间，则需要将所有的内容拷贝到新的地址，当ziplist中存储的值太多，这样的内存拷贝将是一个很大的消耗。
  - 因此，Redis只在一些数据量小的场景下使用ziplist。
- **级联更新问题**：
  - **发生场景**：
    1. 假设有一个极端的场景，在这个ziplist内部，所有的节点的长度都是 253 字节，也就意味着所有节点的`prevlous_entry_length`属性都是一个字节。
    2. 此时，给压缩列表**最前端**插入一个大于 254 字节的节点，那么原来的第一个节点的`prevlous_entry_length`属性会从 1 个字节变成 5 个字节，这个节点的总长度也就来到了 257 字节，大于了254 字节，那么它的下一个节点（原来的第二个节点）的`prevlous_entry_length`属性也需要变成 5 个字节，这又会导致下一个节点的变化... 从而引起了连锁的变化，所有节点的`prevlous_entry_length`值都需要更新一遍。
    3. 同理，删除节点也有可能会造成级联更新的发生。
  - **缺点**：级联更新的时间复杂度很差，最多需要进行N次空间的重分配，每次空间的重分配最差也需要 O（N）, 所以，级联更新的时间复杂度最差是 O（N^2）。
    - 级联更新问题造成Redis性能压力的**概率极其低**：因为级联更新需要大范围连续的节点大小为250-253字节之间，出现的概率非常小，而且当只出现了3~5个结点的级联更新时，对Redis也不会造成性能压力。

##### quicklist（3.2版本）

```c
// 快速列表
struct quicklist{
    // 头结点
    quicklistNode* head;
    // 尾节点
    quicklistNode* tail;
    // 元素总数
    long count;
    // ziplist 节点的个数
    int nodes;
    // LZF 算法压缩深度
    int compressDepth;
}

// 快速列表结点
struct quicklistNode {
    // 快速列表前驱
    quicklistNode* prev;
    // 快速列表后继
    quicklistNode* next;
    // 指向压缩列表 ziplist
    ziplist* zi; 
    // ziplist 的字节总数
    int32 size;
    // ziplist 的元素总数
    int 16 count;
    // 存储形式，是原生的字节数组，还是 LZF 压缩存储
    // 为了进一步节约内存，quicklist 可使用 LZF 算法对 ziplist 进行压缩存储
    int2 encoding;
}
```

![1631929051582](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1631929051582.png)

quicklist，快速列表，是 Redis 3.2 列表的底层实现，是 zipList 和 linkedList 的混合体，它将 linkedList 按段切分，每一段使用 zipList 来紧凑存储，多个 zipList 之间使用双向指针串接起来。（在 Redis 3.2 之前，Redis 采用双向链表 linkedlist 和压缩列表 ziplist 实现）

- **linkedlist缺点**：
  - **指针内存浪费**：每个节点都有自己的前后指针，指针所占用的内存有点多，太浪费了。
  - **内存碎片多**：每个节点单独的进行内存分配，当节点过多，造成的内存碎片太多了，影响内存管理的效率。
- **quicklist优点**：将 linkedlist 和 ziplist 结合起来，通过前后指针，互相连接多个 ziplist，可以在一定程度上缓解 linkedlist 指针内存浪费和内存碎片多的问题，同时解决 ziplist 数据量太大导致的性能变差问题。
- **ziplist切割大小**：
  - ziplist 太小的话（比如为1个元素时），quicklist 退化成了普通的链表，起不到应有的作用；ziplist 太大的话（比如 quicklist 只用一个 ziplist），quicklist 退化成了 ziplist，性能太差。
  - 因此，quicklist 内部默认定义的单个 ziplist 的大小为 `8k 字节`，可以由参数`list-max-ziplist-size`来控制，其作用是，如果分配结点时，发现 ziplist 超过了这个大小，则会重新分配一个 ziplist。
- **压缩深度**：
  - 为了进一步节约内存，quicklist 可使用 LZF 算法对 ziplist 进行压缩存储，同时可以指定压缩深度，由`list-compress-depth`参数决定，默认的压缩深度为 0，表示所有的节点都不进行压缩。
    - 当压缩深度为 1 时：quicklist 两端的第一个 ziplist 不进行压缩。
    - 当压缩深度为 2 时：quicklist 两端的各自2个 ziplist 不进行压缩。
  - 之所以只压缩两端的结点，是因为需要支持列表快速的 push/pop 操作：
    - 如果对两端的 ziplist 压缩了，那么要从列表里面读取值时，必然需要先解压，从而导致性能变差。
    - 因此可以将两端即将被操作的节点不压缩，其他的选择压缩。

##### listpack（5.0版本）

```c
// 紧凑列表类似于ziplist压缩链表，以下是它结点的一些属性：

// 编码方式，记录着结点content属性所保存数据的类型以及长度
int<vat> encoding;
// 内容，真正要保存的数据，类型和长度由encoding决定
optional bute[] content;
// 结点自身长度
int<var> length;
```

![1631943890480](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1631943890480.png)

listpack，紧凑列表，是Redis 5.0 版本中新引入的一个数据结构，和 ziplist 极其相似，列表结点不再记录前一个结点的长度，而是记录自身的长度，从而解决ziplist的痛点问题。（在极小的概率下有可能发生级联更新，当连续规模较大的级联更新发生时，会对 Redis 的性能有比较大的影响）

- **优点**：
  - 不再需要 zltail_offset 属性也可以快速定位到最后一个节点，而是用`listpac总长度 - 最后一个结点的长度`。
  - 每个节点记录自己的长度，当本节点的值发生了改变，只需要更改自己的长度即可，不再需要更改别的节点的属性，彻底解决掉了级联更新的问题。
- **局限**：由于 ziplist 在 Reids 5.0版本前，内部使用得过于广泛，有一些兼容问题，listpack 替代 ziplist 需要一个逐步替换的过程，所以，在5.0版本中，listpack 只被 Stream 数据结构使用。

##### intset

```c
// 整数集合
typedef struct intset{
    // 编码方法，指定当前存储的是16位，还是32位，还是64位的整数数组
    int32 encoding;
    // 集合中的元素数量
    int32 length;
    // 保存元素的数组，具体是多少位整数的数组，取决encoding的值
    int<T> contents;
}
```

![1631932631768](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1631932631768.png)

intset，整数集合，用于保存整数值集合的抽象数据结构，可以保存16、32、64位的整数且保证不重复。

- **整数集合升级**：每当一个整数被添加到整数集合时，都需要先去判断 `这个整数 是否大于 当前编码方式所能容放的最大整数`，如果大于，则需要对当前的整数集合进行升级（16 -> 32 -> 64位），同时，将原来的所有整数转换成新的编码。
  - **好处**：
    - **节约内存**：用能容纳数字的最小编码进行存储，可以有效的节约内存。
    - **提升操作的灵活性**：整数集合封装了对三种整数之间的转换，使得不用考虑类型错误，可以不断的向整数集合内添加整数，提升了操作的灵活性。
  - **不能降级**：与升级相对应的，当大的数字被删除之后，整数集合不会进行降级。

##### skiplist

```c
// 跳表
typedef struct zskiplist{
    // 表头结点和尾节点
    struct zskiplistNode *header, *tail;
    // 表中节点的数量
    unsigned int length;
    // 表中层数最大的节点的层数
    int level;
} zskiplist;

// 跳表结点
typedef struct zskiplistNode{
    struct zskiplistLevel{
        // 前进指针
        struct zskiplistNode *forward;
        // 跨度
        unsigned int span;
    } level[];
    // 后退指针
    struct zskiplistNode *backward;
    // 分值
    double score;
    // 成员对象
    robj *obj;
} zskiplistNode;
```

![1631933408583](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1631933408583.png)

skiplsit，跳跃表，是一种有序的数据结构，它通过在每个节点中维持多个指向其他节点的指针，从而达到快速访问元素的目的，支持平均`O(log N)`、最坏`O(N)`的节点查找，大部分情况下查找效率可以和红黑树媲美，并且相比红黑树可以更方便地实现并发操作（例如Java中的`ConcurrentSkipListMap`）。

- **跳表结点**：
  - **forward**： 前进指针，可以在当前层，继续向右走。
  - **span**：跨度，可以累加查找路径中的所有跨度，计算出当前结点在跳跃表中的一个排名，比如zset提供查看排名的功能。
  - **backward**：后退指针，如果在向右走的太多了，可以用后退指针来向反向操作。
  - **score & obj**：用于保存当前结点的value值以及分值。
- **层级问题**：
  - **计算方式**：
    - 在 Java 的`ConcurrentSkipListMap`的实现中，索引每一次向上升级或者不升级，都是随机的，一个结点是否是一级索引的概率是 50%，是否是二级索引的概率是 25%...
    - 而在 Redis 中，每新添加一个结点，都会给结点随机一个索引层数，而且概率是 25%，之后将该结点的各层索引与左右的索引相链接，即结点为level 1的概率为 1 - 0.25 = 0.75，level 2的概率为 0.75 * 0.25...
  - 由于level + 1的概率都是25%，Redis跳跃表相对于Java中的跳跃表，结构更加扁平一些，因此：
    - **优点**：Redis索引的数量并不是完全的等同于节点数，额外的内存只占用了50%，可以**节省内存**。
    - **缺点**：Redis在查找的时候，可能需要在同级上**多查询几个索引**。
- **顺序问题**：Redis 除了按照score分值排序之外，还会按照value值的字符串字典顺序来排序。
- **排名问题**：可以累加查找路径中的所有跨度，计算出当前结点在跳跃表中的一个排名，比如zset提供查看排名的功能。

###### skiplist vs 平衡树 vs 哈希表

| 比较点         | skiplist                                                     | 平衡树                                                       | 哈希表                                         |
| -------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ---------------------------------------------- |
| 元素有序性     | 有序                                                         | 有序                                                         | 无序                                           |
| 单值查找       | O（logn）                                                    | O（logn）                                                    | 无哈希冲突时为O(1)                             |
| 范围查找       | 实现简单，只需要 O（logn）定位头尾结点，然后遍历链表即可，缓存局部性比平衡树的要好 | 实现困难，需要中序遍历 [ 范围最小的后继，范围最大的前驱 ]    | 只能做单值查找，不适合做范围查找               |
| 插入与删除操作 | 只需要修改相邻结点的指针，简单又快速，并发场景下需要使用volatile修饰指针，效果比平衡树的好 | 可能会引发子树的调整，逻辑复杂，并发场景下需要对根结点进行加锁 | 存在哈希冲突，并发场景下需要对桶头结点进行加锁 |
| 内存占用       | 每个结点平均包含`1/(1-p)`个指针，当p为25%概率时为**1.33**个指针，比平衡树少 | 每个结点平均包含左右子树，共2个指针                          | 无冲突时只有散列表数组                         |
| 缓存友好性     | 由于新结点插入的level是随机的，导致查找路径可能会发生变化，缓存友好性不如平衡树 | 插入新结点后，大部分老结点仍然处于原查找路径上               | 哈希冲突和扩容后，查找路径可能会发生变化       |
| 算法实现难度   | 比平衡树简单                                                 | 典型的有红黑树，实现麻烦                                     | 最简单，不过需要处理哈希冲突                   |

###### 为什么Redis使用skiplist？

- **范围查找效率高**：
  1. 对于 `ZRANGE` 和 `ZREVRANGE` 命令的范围查找，如果使用哈希表，则只能做单值查找，不适合做范围查找。
  2. 如果使用红黑树，需要中序遍历 [ 范围最小的后继，范围最大的前驱 ]，效率低且实现复杂。
  3. 而使用 skiplilst 只需要 O（logn）定位头尾结点，然后遍历链表即可，简单又高效，缓存局部性比红黑树的要好。
- **内存占用少**：Redis skiplist 索引的默认生成概率为 25%，即每个结点平均只包含1.33个指针，内存占用比红黑树的 2 个指针要少。
- **实现与调试容易**：使用 skiplist 比红黑树更容易实现与调试。

##### radix tree（5.0版本）

```c
// rax树
typedef struct rax {
    // rax树头节点
    raxNode *head;
    // 元素数量
    uint64_t numele;
    // rax树节点数量
    uint64_t numnodes;
} rax;

// rax树结点
typedef struct raxNode {
    // 表示这个节点是否包含key，0：没有，1:完整路径存储了key
    uint32_t iskey:1;
    // 是否有存储value值，比如存储元数据就只有key，没有value值，value值也是存储在data中
    uint32_t isnull:1;
    // 是否有前缀压缩，决定了data存储的数据结构，0:非压缩模式，1:压缩模式
    uint32_t iscompr:1;
    // 该节点存储的字符个数
    uint32_t size:29;
    // 存储子节点的信息
    unsigned char data[];
} raxNode;
```

![1631944112996](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1631944112996.png)

raxid tree，Rax树，即基数树，是一棵有序字典树 ，按照 key 的字典序排列，可以快速地定位、插入和删除操作。

#### 基本类型底层实现

Redis自定义了一个Object系统，其中包含5种Object，每种至少有2种不同的编码，其**对应关系**：

| RedisObject#type | RedisObject#encoding      | 值保存条件                                                   |
| ---------------- | ------------------------- | ------------------------------------------------------------ |
| 字符串类型       |                           |                                                              |
| REDIS_STRING     | REDIS_ENCODING_INT        | long范围内的整数值                                           |
| REDIS_STRING     | REDIS_ENCODING_EMBSTR     | <= 39字节字符串值（3.2版本），<= 44字节字符串值（4.0版本）   |
| REDIS_STRING     | REDIS_ENCODING_RAW        | > 39字节字符串值（3.2版本），> 44字节字符串值（4.0版本）     |
| 哈希类型         |                           |                                                              |
| REDIS_HASH       | REDIS_ENCODING_ZIPLIST    | 键和值的长度都 < 64字节，且键值对个数 < 512个时              |
| REDIS_HASH       | REDIS_ENCODING_HT         | 存在键和值的长度 >= 64字节，或者键值对个数 >= 512个时        |
| 列表类型         |                           |                                                              |
| REDIS_LIST       | REDIS_ENCODING_ZIPLIST    | Redis 3.2版本前，所有元素长度都 < 64字节，且列表元素个数 < 512个时 |
| REDIS_LIST       | REDIS_ENCODING_LINKEDLIST | Redis 3.2版本前，存在元素长度 >= 64字节，或者列表元素个数 >= 512个时 |
| REDIS_LIST       | REDIS_ENCODING_QUICKLIST  | Redis 3.2版本后，列表使用统一格式                            |
| 集合类型         |                           |                                                              |
| REDIS_SET        | REDIS_ENCODING_INTSET     | long范围内的整数值，且集合元素数量 < 512个时                 |
| REDIS_SET        | REDIS_ENCODING_HT         | 存在long范围内的整数值元素，或者集合元素数量 >= 512个时      |
| 有序集合类型     |                           |                                                              |
| REDIS_ZSET       | REDIS_ENCODING_ZIPLIST    | 所有元素长度都 < 64字节，且有序集合元素个数 < 128个时        |
| REDIS_ZSET       | REDIS_ENCODING_SKIPLIST   | 存在元素长度 >= 64字节，或者有序集合元素个数 >= 128个时      |

##### String

| 命令        | 使用                                                  | 作用                                                         |
| ----------- | ----------------------------------------------------- | ------------------------------------------------------------ |
| SET         | set key value [ex seconds] [px milliseconds] [nx\|xx] | 设置value，ex代表设置秒TTL，px代表设置毫秒TTL，nx代表只有键不存在时才设置，xx代表只有键存在时才设置 |
| GET         | get key                                               | 获取value                                                    |
| GETSET      | getset key value                                      | 设置value，并返回旧值                                        |
| SETNX       | setnx key value                                       | 如果key不存在，才新增key和value                              |
| SETEX       | setex key seconds value                               | 设置value，并原子设置TTL为seconds                            |
| PSETEX      | psetex key milliseconds value                         | 设置value，并原子设置TTL为milliseconds                       |
| APPEND      | append key value                                      | 在key值后面追加value                                         |
| STRLEN      | strlen key                                            | 返回key值的字符串长度                                        |
| SETRANGE    | setrange key offset value                             | value覆盖从offset开始的字符串                                |
| GETRANGE    | getrange key start end                                | 返回[start，end]部分的字符串                                 |
| INCR        | incr key                                              | value + 1                                                    |
| INCRBY      | incrby key increment                                  | value + increment                                            |
| INCRBYFLOAT | incrbyfloat key increment                             | value + increment                                            |
| DECR        | decr                                                  | value - 1                                                    |
| DECRBY      | decrby key decrement                                  | value - decrement                                            |
| MGET        | mget key[key...]                                      | 批量获取多个键的值                                           |
| MSET        | mset key value[key value...]                          | 批量设置多个键的值                                           |
| MSETNX      | msetnx key value[key value...]                        | 批量设置多个键的值，仅当所有键都不存在时，才会设置成功       |

###### int

- **值保存条件**：long范围内的整数值。
- **对应编码**：REDIS_ENCODING_INT。

![1631946982581](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1631946982581.png)

###### embstr

- **值保存条件**：<= 39字节字符串值（3.2版本），<= 44字节字符串值（4.0版本）。
- **对应编码**：REDIS_ENCODING_EMBSTR。

###### raw

- **值保存条件**：> 39字节字符串值（3.2版本），> 44字节字符串值（4.0版本）。
- **对应编码**：REDIS_ENCODING_RAW。

![1631947051873](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1631947051873.png)

##### Hash

| 命令         | 使用                                   | 作用                                                         |
| ------------ | -------------------------------------- | ------------------------------------------------------------ |
| HSET         | hset hash field value                  | 把hash#field设置为value                                      |
| HSETNX       | hsetnx hash field value                | 把hash#field设置为value，仅当hash#field不存在时，才会设置成功 |
| HGET         | hget hash field                        | 获取hash#field的value                                        |
| HGETALL      | hgetall hash                           | 获取hash中所有的field和value                                 |
| HEXSITS      | hexists hash field                     | 检查hash#field是否存在                                       |
| HDEL         | hdel hash field[field..]               | 删除hash中一个或者多个field                                  |
| HLEN         | hlen hash                              | 获取hash中field的数量                                        |
| HSTRLEN      | hstrlen hash field                     | 获取hash#field的value长度                                    |
| HINCRBY      | hincrby hash field increment           | 把hash#field设置为value+increment                            |
| HINCRBYFLOAT | hincrbyfloat hash field increment      | 把hash#field设置为value+increment                            |
| HMSET        | hmset hash field value[field value...] | 批量把hash#field设置为对应的value                            |
| HMGET        | hmget hash field[field...]             | 批量获取hash#field的value                                    |
| HKEYS        | hkeys hash                             | 获取hash中所有的field                                        |
| HVALS        | hvals key                              | 获取hash中所有的和value                                      |

###### ziplist

- **值保存条件**：键和值的长度都 < 64字节，且键值对个数 < 512个时。
- **对应编码**：REDIS_ENCODING_ZIPLIST。

![1631950146881](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1631950146881.png)

###### dict

- **值保存条件**：存在键和值的长度 >= 64字节，或者键值对个数 >= 512个时。
- **对应编码**：REDIS_ENCODING_HT。

![1631950298985](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1631950298985.png)

##### List

| 命令       | 使用                                  | 作用                                                         |
| ---------- | ------------------------------------- | ------------------------------------------------------------ |
| LPUSH      | lpush key value[value...]             | 把一个或者多个value从表头插入，当key不存在时，会先创建列表再插入元素 |
| LPUSHX     | lpushx key value                      | 把value从表头插入，当key不存在时，则什么也不做               |
| LPOP       | lpop key                              | 移除并返回表头元素                                           |
| RPUSH      | rpush key value[value...]             | 把一个或者多个value从表尾插入，当key不存在时，会先创建列表再插入元素 |
| RPUSHHX    | rpushx key value                      | 把value从表尾插入，当key不存在时，则什么也不做               |
| RPOP       | rpop key                              | 移除并返回表尾元素                                           |
| RPOPLPUSH  | rpoplpush source destination          | 弹出source表尾元素，插入到destination表头，并返回该元素      |
| LREM       | lrem key count value                  | 移除列表count个与value相等的元素，count=0代表移除所有相等的元素；count<0代表从表头开始查找；count>0代表从表尾开始查找 |
| LINSERT    | linsert key before\|after pivot value | 把value插入到pivot前面或者后面，如果没找到pivot，则什么也不做 |
| LINDEX     | lindex key index                      | 获取下标为index的元素（-1代表最后一个元素）                  |
| LSET       | lset key index value                  | 设置index元素为value（-1代表最后一个元素）                   |
| LRANGE     | lrange key start stop                 | 获取列表中[start，stop]区间内的元素                          |
| LTRIM      | ltrim key start stop                  | 剪裁并保留列表中[start，stop]区间内的元素                    |
| BLPOP      | blpop key [key...] timeout            | 阻塞式移除并返回表头元素                                     |
| BRPOP      | brpop key [key...] timeout            | 阻塞式移除并返回表尾元素                                     |
| BRPOPLPUSH | brpoplpush source destination timeout | 阻塞式弹出source表尾元素，插入到destination表头，并返回该元素 |

###### ziplist

- **值保存条件**：Redis 3.2版本前，所有元素长度都 < 64字节，且列表元素个数 < 512个时。
- **对应编码**：REDIS_ENCODING_ZIPLIST。

![1631952569099](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1631952569099.png)

###### linkedlist

- **值保存条件**：Redis 3.2版本前，存在元素长度 >= 64字节，或者列表元素个数 >= 512个时。
- **对应编码**：REDIS_ENCODING_LINKEDLIST。

![1631952812216](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1631952812216.png)

###### quicklist

- **值保存条件**：Redis 3.2版本后，列表使用统一格式。
- **对应编码**：REDIS_ENCODING_QUICKLIST。

![1631953074136](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1631953074136.png)

##### Set

| 命令        | 使用                                 | 作用                                                         |
| ----------- | ------------------------------------ | ------------------------------------------------------------ |
| SADD        | sadd key member [member...]          | 把一个或者多个member加入到集合中，当key不存在时，会先创建列表再添加元素 |
| SISMEMBER   | sismember key member                 | 判断member元素是否为集合的成员                               |
| SPOP        | spop key                             | 移除并返回集合中的一个随机元素                               |
| SRANDMEMBER | srandmember key [count]              | 返回集合中的一个随机元素，count>0，返回集合的子集数组，其元素不重复；count<0，返回含重复元素的数组 |
| SREM        | srem key member [member...]          | 移除集合中一个或者多个member元素，不存在的member元素会被忽略 |
| SMOVE       | smove source destination member      | 把member元素从source集合移动到destination集合中，如果member元素不存在，则什么也不做 |
| SCARD       | scard key                            | 获取集合中元素的数量                                         |
| SMEMBERS    | smembers key                         | 获取集合中所有的成员                                         |
| SINTER      | sinter key [key...]                  | 获取所有给定集合的交集                                       |
| SINTERSTORE | sinterstore destination key [key...] | 获取所有给定集合的交集，并覆盖到destination中                |
| SUNION      | sunion key [key...]                  | 获取所有给定集合的并集                                       |
| SUNIONSTORE | sunionstore destination key [key...] | 获取所有给定集合的并集，并覆盖到destination中                |
| SDIFF       | sdiff key [key...]                   | 获取所有给定集合的差集                                       |
| SDIFFSTORE  | sdiffstore destination key [key...]  | 获取所有给定集合的差集，并覆盖到destination中                |

###### intset

- **值保存条件**：long范围内的整数值，且集合元素数量 < 512个时。
- **对应编码**：REDIS_ENCODING_INTSET。

![1631954240528](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1631954240528.png)

###### dict

- **值保存条件**：存在long范围内的整数值元素，或者集合元素数量 >= 512个时。
- **对应编码**：REDIS_ENCODING_HT。

![1631954355626](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1631954355626.png)

##### Zset

| 命令             | 使用                                                         | 作用                                                         |
| ---------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| ZADD             | zadd key score member [[score member] [score member] ..]     | 把一个或者多个member及其score加入有序集合中，如果key不存在，则会先创建有序集合再添加元素 |
| ZINCRBY          | zincrby key increment member                                 | 把member#score值+increment                                   |
| ZSCORE           | zscore key member                                            | 获取member#score值                                           |
| ZCARD            | zcard key                                                    | 获取有序集合中元素的个数                                     |
| ZCOUNT           | zcount key min max                                           | 获取有序集合中score在[min，max]之间的元素个数                |
| ZRANGE           | zrange key start stop [withscores]                           | 获取有序集合[start，stop]区间的元素，返回的元素会按score从小到大进行排序，相等时则再按字典顺序进行排序 |
| ZREVRANGE        | zrevrange key start stop [withscores]                        | 获取有序集合[start，stop]区间的元素，返回的元素会按score从大到小进行排序，相等时则再按字典逆序进行排序 |
| ZRANGEBYSCORE    | zrangebyscore key min max [withscores] [limit offset count]  | 获取有序集合中score在[min，max]之间的元素个数，返回的元素会按score从小到大进行排序，相等时则再按字典顺序进行排序；min/max语句中，-inf表示最低score，+inf表示最高score，（表示开区间，否则表示闭区间 |
| ZREVRANGEBYSCORE | zrevrangebyscore key min max [withscores] [limit offset count] | 获取有序集合中score在[min，max]之间的元素个数，返回的元素会按score从大到小进行排序，相等时则再按字典逆序进行排序；min/max语句中，-inf表示最低score，+inf表示最高score，（表示开区间，否则表示闭区间 |
| ZRANK            | zrank key member                                             | 获取member在有序集合中从大到小的排名                         |
| ZREVRANK         | zrevrank key member                                          | 获取member在有序集合中从小到大的排名                         |
| ZREM             | zrem key member [member...]                                  | 移除一个或者多个member，如果member不存在，则会被忽略         |
| ZREMRANGEBYRANK  | zremrangebyrank key start stop                               | 移除有序集合[start，stop]区间的元素，返回的元素会按score从小到大进行排序，相等时则再按字典顺序进行排序 |
| ZREMRANGEBYSCORE | zremrangebyscore key min max                                 | 移除有序集合中score在[min，max]之间的元素个数，返回的元素会按score从小到大进行排序，相等时则再按字典顺序进行排序；min/max语句中，-inf表示最低score，+inf表示最高score，（表示开区间，否则表示闭区间 |
| ZRANGEBYLEX      | zrangebylex key min max [limit offset count]                 | 获取有序集合[start，stop]区间的元素，仅当有序集合中的元素分数都相同时才有效；min/max语句中，-表示负无限，+表示正无限，（表示开区间，[ 表示闭区间 |
| ZLEXCOUNT        | zlexcount key min max                                        | 获取有序集合中score在[min，max]之间的元素个数，仅当有序集合中的元素分数都相同时才有效；min/max语句中，-表示负无限，+表示正无限，（表示开区间，[ 表示闭区间 |
| ZREMRANGEBYLEX   | zremrangebylex key min max                                   | 移除有序集合中score在[min，max]之间的元素个数；min/max语句中，-表示负无限，+表示正无限，（表示开区间，[ 表示闭区间 |
| ZINTERSTORE      | zinterstore destination numkeys key [key...] [weights weight [weight...]] [aggregate sum | 计算一个或者多个有序集合的交集，再把结果集覆盖到destination中，默认使用sum；sum会统计所有相同成员的score之和，min会取相同成员的最小score，max会取相同成员的最大score，作为该成员结果score |
| ZUNIONSTORE      | zunionstore destination numkeys key [key...] [weights weight [weight...]] [aggregate sum\|min\|max] | 计算一个或者多个有序集合的并集，再把结果集覆盖到destination中，默认使用sum；sum会统计所有相同成员的score之和，min会取相同成员的最小score，max会取相同成员的最大score，作为该成员结果score |

###### ziplist

- **值保存条件**：所有元素长度都 < 64字节，且有序集合元素个数 < 128个时。
- **对应编码**：REDIS_ENCODING_ZIPLIST。

![1631954526032](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1631954526032.png)

###### skiplist+dict

- **值保存条件**：存在元素长度 >= 64字节，或者有序集合元素个数 >= 128个时。
- **对应编码**：REDIS_ENCODING_SKIPLIST。

![1631955510770](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1631955510770.png)

##### Stream（5.0版本）

（非网上资料，自己理解可能有差错）

| 命令        | 使用                                                   | 作用                                                         |
| ----------- | ------------------------------------------------------ | ------------------------------------------------------------ |
| XADD        | xadd stream 0-1 field value                            | 添加field-value消息到stream中，0-1为指定的消息id，可以使用*，代表使用redis自增消息ID |
| XLEN        | xlen stream                                            | 获取stream中的消息个数                                       |
| XRANGE      | xrange stream min max                                  | 获取消息ID在[min，max]区间的消息，返回的消息按ID顺序排列；min/max语句中，-表示最x小的消息ID，+表示最大的消息ID |
| XREVRANGE   | xrevrange stream min max                               | 获取消息ID在[min，max]区间的消息，返回的消息按ID逆序排列；min/max语句中，-表示最x小的消息ID，+表示最大的消息ID |
| XREAD       | xread count n streams stream                           | 非阻塞式从stream获取n个消息                                  |
| XREAD BLOCK | xread block count n STREAMS stream                     | 阻塞式从stream获取n个消息                                    |
| XGROUP      | xgroup create stream group $                           | 为stream创建消费组group                                      |
| XREADGROUP  | xreadgroup GROUP group cousumer COUNT n STREAMS stream | consumer消费者从group消费组读取stream消费消息                |
| XACK        | xack stream group id                                   | 确认stream队列group消费组中序号为id的消息                    |
| XPENDING    | xpending stream group min max count                    | 获取stream队列group消费组中ID在[min，max]区间的消息总量      |
| XCLAIM      | xclaim stream group consumer id [id...]                | stream队列group消费组中consumer认领指定id消息的所有权        |
| XAUTOCLAIM  | xautoclaim stream group consumer id[id...]             | stream队列group消费组中consumer自动认领指定id消息的所有权    |
| XINFO       | xinfo STREAM stream                                    | 获取stream状态以及所有消费组的信息                           |
| XTRIM       | xtrim stream MAXLEN\|MINID n                           | MAXLEN，表示剪裁stream并最大保存10个消息；MINID，表示剪裁stream并只保留大于等于ID的消息 |
| XDEL        | xdel stream id                                         | 删除stream队列中指定id的消息                                 |

Redis Stream，消息流类型，底层主要使用了紧凑列表（listpack）和基数树（Rax树）。

- listpack，表示一个字符串列表的序列化，用于存储stream的消息内容。
- raxid tree，Rax树，即基数树，是一棵有序字典树 ，按照 key 的字典序排列，可以快速地定位、插入和删除操作。

![1631955983886](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1631955983886.png)

### 1.4. Redis Bitmap与布隆过滤器？

#### BitMap

| 命令     | 使用                                                         | 作用                                                         |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| SETBIT   | set key offset value                                         | 设置offset位置为value（0或1），如果offset不存在，则字符串会自动扩展，空白位置以0填充；如果key不存在，则会先创建一个字符串再设置；时间复杂度为O（1），就跟操作数组一样 |
| GETBIT   | getbit key offset                                            | 获取offset位置的value（0或1），如果offset或者key不存在，则会返回0；时间复杂度为O（1），就跟操作数组一样 |
| BITCOUNT | bitcount key [start] [end]                                   | 计算[start，end]区间内为1的数量，默认计算整个字符串          |
| BITPOS   | bitpos key bit [start] [end]                                 | 获取[start，end]区间内value为bit的位置，默认比较整个字符串   |
| BITOP    | bitop operation destkey key [key...]                         | 对一个或者多个bitmap key进行位操作，最后把结果保存到destkey上；operation为AND代表与，OR代表或，NOT代表非，XOR代表异或 |
| BITFIELD | bitfield key get type offset] [set type offset value] [incrby type offset increment] [overflow wrap\|sat\|fail] | 把整个bitmap看作是一个二进制位数组，可以对指定偏移量的bit进行其他命令操作 |

![1632039188474](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1632039188474.png)

Bitmap，位图，基本思想是，用一个bit位来标记某个元素对应的value，而key即是该元素。

##### 优点

由于采用了bit为单位来存储数据，因此，可以**大大节省存储空间**。比如，1G大约只可以存储1.34亿个int数值，但可以存储10.74亿个bit。

##### 使用场景

主要用于**检索大数据量关键字的状态**，比如大数据量的排序、查找、去重，以及登录表等实际场景。

- **大数据量计数排序**：把序列中所有的元素，一个一个等于自身数值的offset中，然后遍历一次bitmap即可拿出所有顺序的序列；如果序列中有重复的元素，则bitmap中元素需要更多bit来存储，以记录该元素一共出现多少次，在后续顺序遍历中，对于这种元素需要重复取多次。
  - **优点**：非比较排序算法，运算效率高；占用内存少。
- **大数据量快速去重**：使用2 bit 来表示元素的3种状态，00代表不存在，01代表出现1次，11代表出现了多次，接着把每个元素放入等于自身数值的offset中（2bit一个单位），最后遍历出每单位为01的数字即可。
- **大数据量快速查找**：把每个元素的2进制值对应存进bitmap中，最后根据bitmap长度对元素长度取余即可，比如int类型的数值，占4个字节，则应该对32取余 n % 32。
- **登录表**：用于记录用户每天的登录情况，可以一个用户一个bitmap，然后当前有登录则在对应天数offset上记为1，否则记为0，最后遍历bitmap，即可拿到该用户每天的登录情况了。

#### 布隆过滤器

Bloom filter，布隆过滤器，基础数据结构是一个bitmap 位图，可以用来判断集合中一个元**素一定不存在**，或者**可能存在**。

##### 优点

运行快速、内存占用小。

##### 缺点

- 对于元素的存在结果有误判，并且随着系统的不断运行，误判率会越来越高，此时可以**定期重建**布隆过滤器。
- 元素一旦存进布隆过滤器中，删除则十分困难，因为可能会删掉其他元素的 bit 结果，此时可以使用额外的删除标记变量进行**逻辑删除**。

##### 实现原理

1. 当一个元素被加入集合时，通过K个散列函数该元素映射成位图中的K个点，同时把这些点都置为1。
2. 在检索时，只要看看这些点是不是都为1，就可以知道该元素是否在集合中。
3. 如果这些点有任何一个为0，则该元素一定不在。
4. 如果这些点都为1，则该元素很可能在，因为散列函数存在哈希冲突，所以布隆过滤器存在对结果的误判。

![1632042655580](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1632042655580.png)

##### 使用方式

| 实现方式      | 存储位置 | 优点                                   | 缺点                                                         |
| ------------- | -------- | -------------------------------------- | ------------------------------------------------------------ |
| Guava 实现    | JVM      | 可以减轻 Redis 内存与 I/O 的压力       | 应用有状态，水平复制麻烦，布隆过滤器重启即失效，也不支持大数据量的存储；本地缓存无分布式一致性可言，所以无法应用于分布式场景。 |
| Redisson 实现 | Redis    | 支持分布式场景、可以继续保持无状态应用 | 大量的布隆过滤器查询请求，会增加 Redis 的 I/O 压力，同时布隆过滤器还占用一定的 Redis 内存。 |

###### Guava实现

```java
// 构造布隆过滤器
BloomFilter<String> bloomFilter = BloomFilter.create(Funnels.stringFunnel(Charsets.UTF_8),100000,0.01);
// 将号码10086插入到布隆过滤器中
bloomFilter.put("10086");
// 使用布隆过滤器进行判断
System.out.println(bloomFilter.mightContain("123456"));
System.out.println(bloomFilter.mightContain("10086"));
```

###### Redisson实现

```java
// 构造Redisson
RedissonClient redisson = Redisson.create(config);
// 构造布隆过滤器
RBloomFilter<String> bloomFilter = redisson.getBloomFilter("phoneList");
// 初始化布隆过滤器：预计元素为100000000L,误差率为3%
bloomFilter.tryInit(100000000L,0.03);
// 将号码10086插入到布隆过滤器中
bloomFilter.add("10086");
// 使用布隆过滤器进行判断
System.out.println(bloomFilter.contains("123456"));// false
System.out.println(bloomFilter.mightContain("10086"));// true
```

##### 使用场景

主要应用于大规模数据下，**不需要精确过滤**的场景，如检查垃圾邮件地址，爬虫URL地址去重，以及解决**缓存穿透**等问题：

###### 白名单 | 解决缓存穿透、开放转载权限

由于布隆过滤器的存在，可以拦截大部分非白名单内的key，基本解决**缓存穿透**问题。

- **执行过程**：

  1. 服务器启动时，会先把所有key加载到布隆过滤器中，然后客户端请求服务器，会先根据key查询布隆过滤器。
  2. 如果布隆过滤器查询结果为true，代表key可能存在白名单中，则查询Redis。
  3. 如果Redis查询结果不为null，说明该key存在缓存中，则直接返回缓存数据，不需要查询数据库了。
  4. 如果Redis查询结果为null，说明该key不存在缓存中，则继续查询数据库。
  5. 如果数据库查询结果不为null，说明该key是合法，且布隆过滤器没有误判，则把数据查询结果存进Redis中，更新缓存后再返回数据。
  6. 如果数据库查询结果为null，说明这个key是个非法的key（根据业务而定），此时按理应该把key从布隆过滤器删掉，但布隆过滤器存在误判，且删除困难，所以这里对其不做处理（业务允许时），最后返回null。

  ![1632043975276](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1632043975276.png)

- **注意点**：服务器启动时，需要把所有key都存到布隆过滤器里，不然所有请求都会返回空数据。

- **局限**：布隆过滤器存在误判，会有少量请求穿透到数据库中。

  - **解决方案**：误判的几率很小，问题不大无需处理。

######  黑名单 | 视频不重复推送

由于布隆过滤器的存在，可以拦截大部分黑名单内的key，而初次不能拦截，因为需要初始化。

- **执行过程**：与白名单类似，但不同的地方在于：

  1. 服务器启动后，布隆过滤器一开始为空，没有任何的key，需要判断到黑名单才会填充进来，而白名单的是必须在服务器启动前全部key都设置完成。
  2. 如果布隆过滤器判断到key不存在时，才允许查询Redis和数据库，而白名单的是存在才允许查询。
  3. 如果数据库查询结果为null，说明这个key是个非法的key（根据业务而定），这时需要填充回布隆过滤器中，待下次布隆过滤器判断时使用，而白名单的是不做任何处理。

  ![1632044944582](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1632044944582.png)

- **局限**：

  - 黑名单要很全，不然一开始会存在大量的缓存穿透请求。
    - **解决方案**：在判断为非法key时，加入布隆过滤器的黑名单中。
  - 布隆过滤器存在误判，会有少量正常请求被意外拦截掉，返回空的数据。
    - **解决方案**：业务允许则无需处理，比如视频推送。

### 1.5. Redis事务？

#### 概念

- Redis事务，本质上是一组命令的集合，可以一次执行多个命令，在事务执行过程，会按照顺序串行化执行队列中的命令，其他客户端提交的命令请求不会被插入到事务执行命令序列中。
- 总结来说就是，Redis事务是**一次性、顺序性、排他性地执行**一个队列中的一系列命令，整个操作是一个原子操作，事务中的命令要么全部被执行，要么全部都不执行。

#### 事务特性

- **隔离性**：
  - Redis的事务总是具有ACID中的**一致性和隔离性**。
  - 事务是一个单独的隔离操作，事务期间所有命令都会序列化、按顺序地执行，并且在执行的过程中，不会被其他客户端发送来的命令请求所打断。
- **一致性**：Redis的事务总是具有ACID中的**一致性和隔离性**。
- **持久性**：当服务器运行在AOF持久化模式下，并且 `appendfsync` 选项的值为 `always` 时，事务也具有持久性。
  - 如果Redis服务器因为某些原因被管理员杀死，或者遇上某种硬件故障，那么可能只有部分事务命令会被成功写入到磁盘中，这种情况下，Redis在重新启动时发现 AOF 文件出了这样的问题，那么它会退出，并汇报一个错误。
  - 可以使用 `redis-check-aof` 程序可以修复这一问题：它会移除 AOF 文件中不完整事务的信息，确保服务器可以顺利启动。
- **不保证原子性**：虽然Redis事务操作是一个原子操作，但 EXEC执行过程中，如果有一条命令执行失败，其后的命令仍然会被执行，不会进行回滚，因此不保证事务的原子性。

事务命令

Redis事务功能是通过 `MULTI`、`EXEC`、`DISCARD` 和 `WATCH` 四个原语实现的。

- **MULTI**：用于开启一个事务，它总是返回OK，在 `MULTI` 执行之后，客户端可以继续向服务器发送任意多条命令，这些命令不会立即被执行，而是被放到一个队列中，当 `EXEC`命令被调用时，所有队列中的命令才会被执行。
- **EXEC**：执行所有事务块内的命令，返回事务块内所有命令的返回值，按命令执行的先后顺序排列；当操作被打断时，返回空值 `nil` 。
- **WATCH** ：是一个乐观锁，可以为 Redis 事务提供 check-and-set （CAS）行为，监控一个或多个键，一旦其中有一个键被修改（或删除），之后的事务就不会执行，监控会一直持续到 `EXEC` | `DISCARD` | `UNWATCH` 命令。
- **DISCARD**：调用该命令，客户端可以清空事务队列，并放弃执行事务，且客户端会从事务状态中退出。
- **UNWATCH**：命令可以取消 `WATCH` 命令对所有key的监控。

| 命令    | 使用               | 作用                                                         |
| ------- | ------------------ | ------------------------------------------------------------ |
| MULTI   | multi              | 开启事务，标记一个事务的开始                                 |
| EXEC    | exec               | 执行事务，一次执行事务内的所有命令，如果事务被打断，则返回nil，否则按顺序返回命令执行结果 |
| DISCARD | discard            | 取消事务，放弃执行事务块内的所有命令                         |
| WATCH   | watch key [key...] | 监视一个或者多个key，如果事务执行前，这些key被其他命令改动，那么事务将会被打断 |
| UNWATCH | unwatch            | 取消watch命令对key的监视，如果该事务执行了exec或者discard命令，则无需unwatch了 |

#### 事务中的错误

- **命令入队前出错**：事务在执行 `EXEC` 命令前，入队的命令可能会出错，比如命令可能会产生语法错误等，或者其他更严重的错误，比如内存不足等。
  - **Redis应对措施**：
    1. 在 Redis 2.6.5 以前， 客户端的做法是，检查命令入队所得的返回值：如果命令入队时返回`QUEUED` ，那么入队成功；否则，就是入队失败。如果有命令在入队时失败，那么大部分客户端都会**停止并取消**这个事务。
    2. 从 Redis 2.6.5 开始，服务器会对命令入队失败的情况进行记录，并在客户端调用 `EXEC` 命令时，**拒绝执行**并自动放弃这个事务。
- **命令入队后出错**：命令可能在 EXEC 调用之后出错，比如事务中的命令可能处理了错误类型的键：列表命令用在了字符串键上面等等。
  - **Redis应对措施**：在事务中某些命令在 `EXEC` 执行时产生了错误，Redis并不会对它们进行特别处理，而是让其他命令**继续执行**，因而无法保证事务的原子性。

#### 为什么Redis事务不支持回滚？

- **描述**：当事务执行过程中，遇到入队后出错的命令，Redis 并不会进行回滚，而是继续执行其他命令。
- **原因**：
  - **从实用性的角度来说**：入队后出错的命令都是由编程错误造成的，而这些错误应该在开发的过程中被发现，而不应该出现在生产环境中。
  - **从回滚功能的角度来说**：回滚并不能解决编程错误带来的问题，比如程序员本来想通过 `INCR key` 命令将键的值加上 `1` ， 却不小心调用了两次加上了 `2`，或者对错误类型的键执行了`INCR key`，回滚是没有办法处理这些情况的。
  - 因此，鉴于没有任何机制能避免程序员**自己造成的错误**， 并且这类错误通常不会在生产环境中出现，所以，Redis 选择了更简单、更快速的无回滚方式来处理事务。
- **优点**：由于无需对回滚进行支持，所以 Redis 内部可以保持简单且快速。

### 1.6. Redis key过期机制？

- **永久 key**：在没有指定过期时间的情况下创建，这种 key 将永远存在，除非以明确的方式将其删除，比如使用 `DEL` 命令。

- **可过期 key**：`EXPIRE` 命令，通过增加一些额外的内存成本，为 key 指定一个过期时间，当一个 key 设置了过期时间后，Redis 将确保在指定的时间段过去后，删除这个 key。

- Redis 同时使用 2 种 **key 过期机制**：

  - **被动方式**：当客户端尝试访问某个 key 时，如果发现该 key 已超时，则该 key 才会被动地过期与删除。

    - **对内存不友好**：虽然存在 key 超时后，如果永远没有被访问，那么它就不会被删除，永远地占用着服务器的内存。

  - **主动方式**：Redis 默认每秒 10 次（可在 `hz 10` 中配置），进行以下随机测试：

    1. 在可过期 key 集合中，随机测试 20 个 key。
    2. 删除所有发现已过期的 key。
    3. 如果超过 25% 的 key 都已过期，则继续以上循环。

    => 这意味着在任何时刻，过期 key 所能占用的最大内存 = 每秒新写入 key 所占用的内存之和 / 4，即过期 key 最多只会占用 1/4 的内存。

### 1.7. Redis内存淘汰策略？

- **内存淘汰机制**：使用 Redis，可以方便地在客户端添加新数据时，自动淘汰旧数据。其中，LRU 是 Redis 支持的淘汰方法之一，从 Redis 4.0 版本开始，引入了新的 LFU 淘汰方法。

  - **LRU**：Least Recently Used，最近最少使用淘汰算法，用于淘汰**最长时间没有被访问**的旧数据。
  - **LFU**：Least Frequently Used，最不经常使用淘汰算法，用于淘汰**在一段时间内访问次数最少**的旧数据。

- **内存淘汰策略**：指当达到指定的 `maxmemory` （默认为 0，代表没有限制）内存量时，可以在不同的行为中进行选择需要淘汰的旧数据。

  - 当 `maxmemory` 达到限制时，发生的内存淘汰策略是根据 `maxmemory-policy` 配置来指定的。

  | 策略                     | 作用对象   | 客户端请求发现内存不足时                                     | 适用场景                                            |
  | ------------------------ | ---------- | ------------------------------------------------------------ | --------------------------------------------------- |
  | noeviction               | 全局 key   | 会返回错误                                                   | 常量字典，不能淘汰任何 key 时                       |
  | allkeys-lru              | 全局 key   | 会先尝试删除 LRU key                                         | 热点缓存，需要淘汰非热点 key 时                     |
  | volatile-lru             | 可过期 key | 会先尝试删除 LRU key，但仅限于可过期 key，如果没有任何可过期 key，则会返回错误 | 热点缓存，需要淘汰非热点 key，又需要保护永久 key 时 |
  | allkeys-random           | 全局 key   | 会随机淘汰 key                                               | 需要以相同概率去淘汰 key 时                         |
  | volatile-random          | 可过期 key | 会随机淘汰 key，但仅限于可过期 key，如果没有任何可过期 key，则会返回错误 | 需要以相同概率去淘汰 key ，又需要保护永久 key 时    |
  | volatile-ttl             | 可过期 key | 会先尝试删除剩余 TTL 最短的 key，但仅限于可过期 key，如果没有任何可过期 key，则会返回错误 | 需要根据过期时间去淘汰 key 时                       |
  | allkeys-lfu（4.0 版本）  | 全局  key  | 会先尝试删除 LFU key                                         | 需要淘汰访问次数少的 key 时                         |
  | volatile-lfu（4.0 版本） | 可过期 key | 会先尝试删除 LFU key，但仅限于可过期 key，如果没有任何可过期 key，则会返回错误 | 需要淘汰访问次数少的 key，又需要保护永久 key 时     |

### 1.8. Redis分布式锁？

当发生高并发访问量激增时，虽然在系统会通过限流、异步、排队等方式优化，但整体的并发还是平时的数倍以上，为了避免并发问题，**防止库存超卖**，给用户提供一个良好的购物体验，这些系统中都会用到锁的机制。

- **背景**：

  - 对于单进程的并发场景，可以使用编程语言及相应的类库提供的锁，如 Java 中的 synchronized 语法以及 ReentrantLock 类等，来避免并发问题。

  ![1632658813782](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1632658813782.png)

  - 而如果在分布式场景中，实现**不同客户端**的线程对代码和资源的同步访问，保证在多线程下处理共享数据的安全性，就需要用到分布式锁技术。

  ![1632658861088](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1632658861088.png)

- **概念**：分布式锁是指，控制分布式系统或者不同系统之间，**共同访问共享资源**的一种锁实现，在不同的系统或者同一个系统的不同主机之间共享了某个资源时，可以用来互斥地防止彼此干扰保证一致性。

- **特性**：

  - **互斥性**：互斥是锁的基本特征，同一时刻锁只能被一个线程持有，执行临界区操作。
  - **超时释放**：通过超时释放，可以避免死锁，防止不必要的线程等待和资源浪费。
  - **可重入性**：一个线程在持有锁的情况下，可以对其再次请求加锁。
  - **高性能和高可用**：加锁和释放锁的过程性能开销要尽可能的低，同时也要保证高可用，防止分布式锁意外失效。

- **实现方式**：

  - **通过数据库方式实现**：基于乐观锁和唯一索引实现。
  - **基于分布式缓存实现**： 基于 Memcached、Redis 单机、Redisson 和 Redis RedLock 实现。
  - **基于分布式一致性算法实现**：基于ZooKeeper、Chubby（google闭源实现）和 Etcd 实现。

#### 基于数据库实现 | 负担大

##### 基于乐观锁实现

- **原理**：根据版本号，来判断更新之前有没有其他线程更新过，如果被更新过，则获取锁失败。

##### 基于唯一索引实现

- **原理**：数据库建立唯一索引，当想要获得锁时，向数据库中插入一条记录，释放锁时则删除这条记录。

- **存在的问题**：
  1. 锁没有失效时间，解锁失败会导致死锁，此时该唯一索引所有 insert 都会返回失败，其他线程无法再获取到锁。
  2. 只能是非阻塞锁，insert 失败直接就报错了，无法进入队列进行重试。
  3. 不可重入，同一线程在没有释放锁之前无法再获取到锁。

#### 基于分布式缓存实现 | 锁失效

##### 基于 Memcached 实现

- **原理**：利用 Memcached 的 `add` 命令，由于该命令原子性操作，只有在 key 不存在的情况下，才能 add 成功，也就意味着同一时刻只有一个线程获得锁。

##### 基于 Redis 单机实现

###### 1、使用 setnx 指令

- **原理**：

  ```shell
  setnx key value
  	do do something
  del key
  ```

- **存在的问题**：do something 有问题，锁一直不会释放，因此需要增加锁的过期时间。

###### 2、使用 setnx+expire 指令

- **原理**：

  ```shell
  setnx key value
  expire key 10
  	do do something
  del key
  ```

- **存在的问题**：`setnx` 与 `expire` 不是一个原子操作，如果在执行 `setnx` 和 `expire`  之间发生异常，`setnx` 执行成功，但 `expire` 没有执行，则会导致这把锁在长期存在，导致其他进程无法正常获取锁。

###### 3、使用 set 扩展指令

- **原理**：

  ```shell
  set key value NX EX 10
  	do something
  del key
  ```

- **存在的问题**：如果 do something 耗时过长，会出现锁被提前释放，甚至被别的进程误删。

- **解决方案**：

  1. do something 部分不要做过长时间的处理，但还是可能存在 STW 的停顿风险。
  2. 释放锁时，需要验证锁的持有者是否是自己。

###### 4、释放前判断值是否改变

- **原理**：

  ```shell
  set key random_value nx ex 10 # 加锁
  	do something
  if random_value == key.value  # 判断 value
  	del key 			      # 删除 key
  ```

- **存在的问题**：判断 value 和删除 key 两个操作不能保证原子性。

- **解决方案**：需要使用 Lua 脚本进行处理，因为 Lua 脚本可以保证连续多个指令的原子性执行。

###### 5、使用 lua 脚本

- **原理**：

  ```lua
  if redis.call("get", KEY[1]) === ARGV[1] then
      return redis.call("del", KEY[1])
  else 
      return 0
  end
  ```

- **存在的问题**：还是会出现，由于时钟漂移 或者 do something 耗时过长，导致的锁被提前释放，甚至被别的进程误删的问题。

- **解决方案**：锁租期续约。

##### 基于 Redisson 实现

Redisson，是一个在 Redis 的基础上实现的 Java 驻内存数据网格，是一个分布式、可扩展的 Java 数据结构。

- **原理**：
  1. 让获得锁的线程开启一个定时器的守护线程，每 expireTime/3 执行一次，去检查该线程的锁是否存在。
  2. 如果存在，则对锁的过期时间重新设置为 expireTime，即利用守护线程对锁进行**续约**，防止锁由于过期提前释放。
  3. 对于Redis 多机环境的分布式锁，Redisson 也提供了 Red Lock的算法实现。

##### 基于 Redis RedLock 实现

- **背景**：基于 Redis 单机实现的分布式锁，加锁时只作用在一个 Redis 节点上，即使通过了 Sentinel 保证了高可用，但由于 Redis 的复制是异步的，如果在 Master 节点获取到锁后，在未完成数据同步的情况下发生故障转移，此时其他客户端上的线程依然可以获取到锁，因此会丧失锁的安全性。
- **原理**：
  1. 获取当前 Unix 时间 `t1`，以毫秒为单位。
  2. 按顺序依次尝试从 5 个实例使用相同的 key 和具有唯一性的 value（例如 UUID）来获取锁，当向 Redis 请求获取锁时，除了设置锁自动失效时间 `expire`，客户端还应该设置响应超时时间 `timeout`，且这个超时时间 < 锁的失效时间 `expire` 。
     1. 比如锁自动失效时间 `expire` 为10秒，则超时时间 `timeout` 应该在5-50毫秒之间。
     2. 这样可以避免服务器端 Redis 已经挂掉的情况下，客户端还在一直等待响应结果，使得在服务器端没有在规定时间内响应时，客户端可以尽快尝试去另外一个 Redis 实例请求获取锁。
  3. 客户端使用当前时间 `t3` 减去开始获取锁时间 `t1`，就得到获取锁花费的总时间 `T`，当且仅当从大多数（N/2+1，这里是3个节点）的 Redis 节点都取到锁，并且获取锁花费的总时间 `T` < 锁失效时间 `expire`时，锁才算获取成功。
  4. 如果取到了锁，key 的真正有效时间 `real_expire` 等于锁失效时间 `expire  `减去锁花费的总时间 `T`。
  5. 如果因为某些原因，获取锁失败（没有在至少N/2+1个 Redis 实例取到锁，或者取锁时间已经超过了锁失效时间 `expire），客户端应该在所有的 Redis 实例上使用 Redis Lua 脚本进行解锁。
     1. 原因是可能存在某个节点加锁成功后，**返回客户端时**的响应包丢失了，即客户端向服务器通信是正常的，但反方向却是有问题的。
     2. 虽然对客户端而言，由于响应超时导致加锁失败，但是对 Redis节点而言，`SET` 指令执行成功，意味着加锁成功。
     3. 因此，释放锁的时候，客户端也应该对当时获取锁失败的那些 Redis 节点同样发起请求。
     4. 除此之外，为了避免 Redis 节点发生崩溃重启后造成锁丢失，从而影响锁的安全性，Redis 官方还提出了**延时重启**的概念，即一个节点崩溃后不要立即重启，而是等待一段时间后再进行重启，这段时间应该大于锁失效时间 `expire  `。
- **局限**：
  - **性能过重**：使用 RedLock 维护那么多的Redis实例，提升了系统的维护成本。
  - **仍然不安全**：RedLock 严重依赖系统时钟，如果 Master 系统时间发生错误，会导致它持有的锁提前过期然后被释放，因此，RedLock 还是不能保证锁的安全性。

##### Redis 分布式锁总结

- **存在的问题**：
  - **客户端长时间阻塞导致锁失效问题**：比如执行业务时间过长，或者发生了 STW。
    - **解决方案**：锁租期续约，在锁有效时间内，异步启动另外一个线程去检查的问题，判断这个 key 是否超时，如果锁超时时间快到期且逻辑未执行完，则延长锁超时时间。
  - **服务器时钟漂移问题导致同时加锁**：由于 Redis 的过期时间是依赖系统时钟的，如果时钟漂移过大时，理论上是可能出现的会影响到过期时间的计算。
    - **解决方案**：根据时间来自动释放的分布式锁，都没办解决这个问题。
  - **单点实例故障，锁未及时同步导致锁丢失**：
    1. 虽然使用 RedLock 算法，可以通过多节点来防止 Redis 的单点故障，但效果一般，且仍然无法防止在**主从切换**导致的两个客户端同时持有锁的发生。
    2. 但在大部分情况下，主从切换持续时间极短，RedLock 会在切换的瞬间获取到节点的锁，虽然也是有问题发生的可能，但这已经是极低的概率了，无法避免。
    3. 因此，Redis 分布式锁适合**幂等性**事务，如果一定要保证安全，应该使用 Zookeeper、ETCD 或者 DB，但是，锁的性能会急剧下降。

#### 基于分布式一致性算法实现 | 强一致

##### 基于 Zookeeper 实现

Zookeeper，是一个为分布式应用提供一致性服务的软件，它内部是一个分层的文件系统目录树结构，规定统一个目录下**只能有一个唯一文件名**。

- **概念**：
  - **数据模型**：
    - **永久结点**：结点创建后，不会因为会话失效而消失。
    - **临时结点**：与永久结点相反，如果客户端连接失效，则立即删除结点。
    - **顺序结点**：与上述两个结点特性类似，如果指定创建这类结点时，ZK 会自动在结点名后加一个数字后缀，并且是有序的。
  - **监视器**：watcher，当创建一个节点时，可以注册一个该结点的监视器，当节点状态发生改变时，watcher 被触发时，ZooKeeper 会向客户端发送且仅发送一条通知（因为 watcher 只能被触发一次）。
- **原理**：
  1. 创建一个锁目录 lock。
  2. 希望获得锁的线程 A 就在 lock 目录下，创建**临时顺序结点**。
  3. 先获取锁目录下所有的子结点，再获取比自己小的结点，如果不存在，则说明当前线程的顺序号最小，则 线程 A 获得锁。
  4. 接着，线程 B 获取所有结点，判断自己不是最小的结点，此时发现存在有更小的线程 A 的结点，则设置监听器 watcher，只监听比自己次小的结点 A（为了防止发生**羊群效应**）。
     - **分布式中的羊群效应**：也称惊群效应，指在分布式系统中，比如Zokeeper集群中，当某一结点 A 被大量 Client 进行Watch 时，当结点 A 发生变化时，可能只会对某一个 Client 有影响，但是由于所有 Client 都对该结点进行了 watch，其他没有影响的 Client 也会受到通知，因此，对于产生了这种**不必要的通知**就是分布式中的羊群效应。
  5. 当线程 A 处理完业务，会删除结点 A，释放掉，然后线程 B 监听到变更事件，判断到自己是最小的结点，成功获得锁。

##### 基于 Chubby 实现

- **原理**：Google 公司实现的**粗粒度**分布式锁服务，有点类似于 ZooKeeper，但也存在很多差异，通过 sequencer 机制解决了**请求延迟**造成的锁失效的问题。

##### 基于 Etcd 实现

- **概念**：
  - **Lease 机制**：租约机制，Etcd 可以为存储的 KV 对设置租约，当租约到期，KV 将失效删除，同时支持续约，即 KeepAlive。
  - **Revision 机制**：
    1. 每个 key 带有一个 Revision 属性值，Etcd 每进行一次事务对应的全局 Revision 值都会加 1。
    2. 因此，每个 key 对应的 Revision 属性值都是全局唯一的，通过比较 Revision 的大小就可以知道进行写操作的顺序。
    3. 在实现分布式锁时，多个程序同时抢锁，根据 **Revision 值大小**依次获得锁，可以避免**羊群效应**，实现公平锁。
  - **Prefix 机制**：前缀机制，也称目录机制，可以根据前缀（目录），来获取该目录下所有的 key 及对应的属性（包括 key, value 以及 revision 等）。
  - **Watch 机制**：监听机制，Watch 机制支持 Watch 某个固定的 key，也支持 Watch 一个前缀（目录），当被 Watch 的 key 或目录发生变化，客户端将收到通知。
- **原理**：
  1. **准备 key**：客户端连接 Etcd，以 `/lock/mylock` 为前缀创建**全局唯一**的 key。
     - 假设第一个客户端对应的 key="/lock/mylock/UUID1"，第二个为 key="/lock/mylock/UUID2"，客户端分别为自己的 key 创建租约 Lease，租约的长度根据业务耗时确定，假设为 15s；
  2. **写入 key**：进行 put 操作，将步骤 1 中创建的 key 绑定租约写入 Etcd，根据 Etcd 的 Revision 机制，假设两个客户端 put 操作返回的 Revision 分别为 1、2，客户端需记录 **Revision** 用以接下来判断自己是否获得锁。
  3. **获取锁**：
     1. 客户端以前缀 `/lock/mylock` 读取 keyValue 列表，其中 keyValue 中会带有 key 对应的 Revision。
     2. 接着判断自己 key 的 Revision 是否为当前列表中**最小**的，如果是则认为获得锁。
     3. 否则，需要监听列表中前一个 Revision 比自己小的 key 的**删除事件**，一旦监听到删除事件，或者因租约失效而删除的事件，则自己获得锁。
  4. **执行业务**：获得锁后，操作共享资源，执行业务代码。
  5. **心跳续约**：
     1. 当一个客户端持有锁期间，其它客户端只能等待，为了避免等待期间租约失效，持有锁的客户端需创建一个定时任务，作为**心跳**进行续约。
     2. 如果持有锁期间客户端崩溃，心跳停止，key 将因**租约到期**而被删除，从而锁释放，**避免死锁**。
  6. **释放锁**：完成业务流程后，删除对应的key释放锁。

#### 分布式锁总结

- **数据库锁**：
  - **优点**：直接使用数据库，使用简单。
  - **缺点**：分布式系统大多数瓶颈都在数据库，使用数据库锁会增加数据库负担。
- **分布式缓存锁**：
  - **优点**：性能高，实现起来较为方便，在**允许偶发的锁失效**情况，不影响系统正常使用，建议采用分布式缓存锁。
  - **缺点**：通过锁超时机制不是十分可靠，当线程获得锁后，处理时间过长导致锁超时，就失去了锁的作用。
    - 当业务必须要数据的强一致性，不允许重复获得锁时，比如金融场景的重复下单与重复转账场景下请不要使用分布式缓存锁，此时可以使用 CP 模型实现，比如Zookeeper 和 Etcd。
- **分布式一致性算法锁**：
  - **优点**：不依靠超时时间释放锁，可靠性高，当系统要求高可靠性时，建议采用分布式一致性算法锁。
  - **缺点**：性能比不上分布式缓存锁，因为无论是 Zookeeper 还是 Etcd，都需要频繁的创建和删除结点。

|            | Redis    | Zookeeper                       | Etcd                            |
| ---------- | -------- | ------------------------------- | ------------------------------- |
| 一致性算法 | 弱一致性 | Paxos（ZAB）                    | Raft                            |
| CAP        | AP       | CP                              | CP                              |
| 高可用     | 主从集群 | n+1（保证奇数，过半存活即可用） | n+1（保证奇数，过半存活即可用） |
| 实现       | setnx    | createNode                      | restfulAPI                      |

### 1.9. Redis持久化？

持久化，就是把内存中的数据，持久化到本地磁盘中，防止服务器宕机了内存数据丢失。

- Redis 提供两种持久化机制 **RDB（默认）** 和 **AOF**，Redis 4.0 以后采用混合持久化，用 AOF 来**保证数据不丢失**，作为数据恢复的第一选择，用 RDB 来做不同程度的**冷备**。

#### RDB

RDB，Redis DataBase，是Redis**默认**的持久化方式，Redis 会按照一定的时间间隔，将内存的数据以**时间点快照、二进制**的形式保存到磁盘中。

##### 特点

- 只会产生一个数据文件，为dump.rdb。
- 可以通过配置文件中的 `save` 参数，来定义快照生成周期。

##### 生成方式

- **`SAVE` 命令**：会阻塞当前 Redis 主线程，直到持久化完成。线上应该禁止使用，要使用也需要先明确时间点，再关机维护。
- **`BGSAVE` 命令**：
  1. Redis 调用 `fork（） `一个子进程，同时拥有父进程和子进程。
  2. 父进程继续处理其他命令，子进程负责持久化过程，将数据集写入到一个临时 RDB 文件中。
     - **写时复制**：Copy On Write，COW，指的是，为了节约物理内存，操作系统在调用 `fork()` 生成子进程时，子进程与父进程会**共享同一内存区域**，只有当其中父子中任意一个进程进行写操作时，操作系统才会为其另外分配内存页面。其中，子进程与父进程是两个独立的进程，在父进程结束后，子进程并不会结束，而是会被 `init` 进程托管。**原理如下**：
       1. 当进程 A 使用系统调用 `fork()` 创建一个子进程 B 时，由于子进程 B 实际上是父进程A的一个拷贝，因此会拥有与父进程相同的物理页面。
       2. 为了节约内存和加快创建速度的目标，`fork()` 函数会让子进程 B 以**只读方式**共享父进程A的物理页面，同时，也将父进程 A 对这些物理页面的访问权限设置成**只读**。
       3. 这样，当父进程 A 或子进程 B 任何一方对这些已共享的物理页面执行写操作时，都会产生页面出错异常 `page_fault int14` 中断，此时 CPU 会执行系统提供的异常处理函数`do_wp_page()` 来解决这个异常。
       4. `do_wp_page()` 会对这块导致写入异常中断的物理页面进行**取消共享操作**，为写进程复制一新的物理页面，使父进程 A 和子进程 B 各自拥有一块内容相同的物理页面。
       5. 最后，在从异常处理函数中返回后，CPU会重新执行刚才导致异常的写入操作指令，使陷入异常的进程继续执行下去。
     - **Redis 与写时复制**：根据 Copy On Write 技术可知， `fork()` 操作并不会导致生成 RDB 时内存的暴涨。
       1. 因为只有父进程发生写操作修改内存数据时，才会真正去给子进程分配内存空间，并只是复制父进程中被修改过的内存页中的数据，并不是全部的内存数据。
       2. 对于那些没有写操作的数据，子进程会共享父进程同一段内存。
       3. 因此，`fork()` 操作是不会导致内存暴涨的。
  3. 当子进程完成对新 RDB 文件的写入时，Redis 用新 RDB 文件替换原来的 RDB 文件，并删除旧的 RDB 文件。

##### 生成周期配置

save参数。

```shell
# 900秒（15分钟）内数据集至少有1个改动
save 900 1
# 300秒（5分钟）内数据集至少有10个改动
save 300 10
# 60秒（1分钟）内数据集至少有10000个改动
save 60 10000
```

##### 其他参数

```shell
# save、bgsave期间，如果出错，是否停止继续生成RDB
# yes：停止
# no：不停止，但可能会造成数据不一致
stop-writes-on-bgsave-error

# 是否压缩RDB文件
# yes：开启
# no：关闭，可以节省CPU消耗，但可以减小RDB大小
rdbcompression

# 是否校验RDB文件
# yes：使用CRC 64算法对RDB数据进行校验，但会有10%左右的性能损耗
# no：不校验
rdbchecksum
```

##### 优点

- **适合备份**：RDB 保存了 Redis 在某个时间点上的数据集，非常适合用于备份。
- **适合灾备**：RDB 只有一个文件，并且内容都非常紧凑，可以远程传输到别的数据中心，灾备简单，非常适用于灾难恢复。
- **影响 Redis 性能小**：RDB 可以最大化减少对 Redis 性能的影响，在保存 RDB 文件时，父进程唯一要做的就是 `fork` 出一个子进程，然后交由子进程处理接下来所有的保存工作，父进程无须执行任何磁盘 I/O 操作，从而可以保证主进程继续处理命令，RDB 时只存在毫秒级不响应请求。
- **恢复速度快**：RDB 相对 AOF 来说，在恢复大数据集时，速度比 AOF 的恢复速度要快。

##### 缺点

**数据安全性低**，RDB 是间隔一段时间进行持久化，如果间隔前进，Redis 发生故障，则会发生数据丢失。

#### AOF

AOF，Append Only File，记录 Redis 服务器执行的所有**写操作命令**，并在服务器启动时，通过重新执行这些命令来还原数据集。

##### 特点

- AOF 文件中的命令，会全部以 Redis 协议的格式来保存，新命令会被**追加**到文件的末尾。 
- 可以在后台对 AOF 文件进行**重写**，使得 AOF 文件的体积，不会超出保存数据集状态所需的实际大小。

##### 写入与同步参数

```shell
# AOF默认关闭，yes可以开启
appendonly no
# AOF的文件名
appendfilename "appendonly.aof"

#appendfsync always
appendfsync everysec
#appendfsync no
```

- **appendfsync always**：命令写入 aof_buf 后，直接调用系统的 `fsync` 操作同步到 AOF 文件中，真正的把指令写入了磁盘中。
  - **aof_buf**：AOF 缓冲区，打开 AOF 开关后，每次执行完一个写命令后，都会把写命令以 Redis 协议的格式保存到 aof_buf 缓冲区中。
  - **特点**：由于 AOF 每次都会同步落盘，所以优点是**数据不会丢失**，缺点是**效率低**。
- **appendfsync everysec**：命令写入aof_buf 后，调用系统的 `write` 操作，把 aof_buf 缓冲区的内容写入到 AOF 文件中， 然后在`write` 操作完成后返回；而`fsync` 同步 AOF 文件的操作，将由专门的线程每秒调用一次。
  - **特点**：对 always 和 no 方案，在**数据安全性**和**性能**上做了折衷。
- **appendfsync no**：命令写入 aof_buf 后，调用系统的 `write` 操作，把 aof_buf 缓冲区的内容写入到 AOF 文件中，然后在`write` 操作完成后返回；不对 AOF 文件做 `fsync` 同步操作，交由操作系统负责。
  - **操作系统同步条件**：缓冲区被填满，或者，超过了同步周期限制，通常同步周期限制最长为30秒。
  - **特点**：与 always方案相反，是另一个极端，这种方案由于 Redis 不保证 AOF 的落盘，所以保证不了数据安全性。

![1632132586785](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1632132586785.png)

##### 备份流程

1. 处理客户端**写命令**请求，处理完毕后，响应客户端请求。
2. 接着处理时间事件，Redis 将 `serverCron` 作为时间事件运行，定期自动运行一次，比如尝试进行 AOF 或者 RDB 持久化等操作。
3. 然后判断是否打开 `appendonly`，如果为是，则把**写命令**写入 aof_buf 缓冲区，并根据 `appendfsync` 策略同步到磁盘中；如果为否，则结束 AOF 备份流程。

![1632135630382](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1632135630382.png)

##### 重写流程

```shell
# AOF重写机制的触发参数：
# 当前AOF文件的大小是上次AOF大小的100%，且文件体积达到64m时，则触发AOF重写
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
```

1. 当前 AOF 文件的大小是上次 AOF 文件大小的100%，且文件体积达到 64m时，则触发 AOF 重写；或者显示调用 `BGREWRITEAOF` 命令，也会触发 AOF 重写。
2. 此时，Redis 执行 `fork()` ，同时拥有父进程和子进程。
3. 子进程开辟 AOF 重写缓冲区，并开始执行 AOF 重写，根据 `fork()`写时复制机制，子进程只能共享 `fork()`时的内存数据，此时子进程会根据内存快照，按照**命令合并规则**把重写后的命令写入到新 AOF 文件。
4. 父进程继续处理其他命令，对于所有新执行的写入命令，父进程一边把这些改动，追加到旧 AOF 文件的末尾，并根据 `appendfsync` 策略同步到磁盘中，一边把它们累积到重写缓冲区中，这样即使在重写的中途发生了停机，旧 AOF 文件也还是安全的。
5. 当子进程完成重写工作时，会给父进程发送一个信号，父进程在接收到信号之后，阻塞服务器进程，拒绝所有命令，同时把重写缓冲区中的所有数据，追加到新 AOF 文件的末尾。
6. 最后原子地用新 AOF 文件替换旧 AOF 文件，再恢复服务器进程，之后所有命令都会追加到新 AOF 文件的末尾，完成一次 AOF 的重写。

![1632138509700](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1632138509700.png)

##### 还原流程

1. Redis 服务启动时，会先创建 Fack Client 伪客户端，该客户端不会发送网络请求，只是用来读取 Redis 配置文件。
2. 接着判断 AOF 文件中是否存有数据，如果没有，则结束 AOF 还原流程。
3. 如果有，则分析、读取、执行 AOF 文件中的每一条指令，直到 AOF 中的指令全部被执行完毕后，则结束 AOF 还原流程。
4. Redis 还可以同时使用 AOF 和 RDB 持久化，在这种情况下，会优先使用 AOF 文件来还原数据集， 因为 AOF 文件保存的数据集，通常比 RDB 文件所保存的数据集更完整。

![1632138046711](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1632138046711.png)

##### 优点

- **持久化实时性好**：AOF 可以根据不同的 fsync 策略来持久化，默认为 `always`，表示每秒钟 fsync 一次，Redis 仍然可以保持良好的性能，就算发生故障停机，最多也只是丢失一秒钟的数据。
- **顺序写，写入效率高**：AOF 文件是一个只进行追加操作的日志文件， 对 AOF 文件的写入不需要进行 seek，顺序写即可，所以写入效率高。
- **AOF重写**：Redis 可以在 AOF 文件体积变得过大时，自动地在后台对 AOF 进行重写，重写后的新 AOF 文件包含了恢复当前数据集所需的**最小命令集合**。 
- **简单易懂**：AOF 文件写入的命令，会以 Redis 协议的格式进行保存， 其内容非常容易被人读懂， 对文件进行分析会比较轻松。 

##### 缺点

- **体积较大，恢复速度慢**：对于相同的数据集来说，AOF 文件的体积通常要大于 RDB 文件的体积，且恢复速度也会比 RDB 的慢。
- **可能会影响 Redis 性能**：针对不同的同步机制，AOF 会比 RDB 慢，由于 AOF 每秒都会备份做日志写操作，这样相对比 RDB 来说，性能就略低，比如每秒备份 fsync，如果客户端每秒的写入都做一次 fsync 备份的话，那么 Redis 的性能就会下降。

#### RDB & AOF如何选择？

|                        | RDB                                           | AOF                                                  |
| ---------------------- | --------------------------------------------- | ---------------------------------------------------- |
| 文件内容               | 全量的二进制数据                              | 按Redis 协议格式保存的写命令                         |
| 文件体积               | 较小                                          | 较大，同时提供 AOF 重写                              |
| 数据恢复速度           | 快                                            | 慢                                                   |
| 对Redis 性能的影响程度 | 子进程备份时间间隔较远，对 Redis 性能影响较小 | 如果子进程每秒都做 AOF 同步，则对 Redis 性能影响较大 |
| 适用场景               | 按照时间间隔备份，适合冷备                    | 实时性好，适合热备                                   |

##### 仅使用 RDB

如果可以承受一段时间内的数据丢失， 那么可以只使用 RDB 持久化。

##### 仅使用 AOF

**不推荐**，因为定时生成 RDB 快照非常便于进行数据库备份，并且 RDB 恢复数据集的速度也要比 AOF 恢复的速度要快，除此之外，使用 RDB 还可以避免之前 AOF 程序的 bug 。

- AOF 在过去曾经发生过这样的 bug： 因为个别命令的原因，导致 AOF 文件在重新载入时，无法将数据集恢复成保存时的原样，比如阻塞命令 `BRPOPLPUSH` 。
- 虽然这种 bug 在 AOF 文件中并不常见， 但是对比来说， RDB 几乎是不可能出现这种 bug 的。

##### 同时使用 AOF 和 RDB

如果对实时性数据比较关心，则应该同时使用两种持久化功能。

1. 使用 RDB 和 AOF 结合一起做持久化，**RDB 做冷备**，可以在不同时期对不同版本做恢复， **AOF 做热备**，保证数据仅仅只有1秒的损失。
2. 当 AOF 破损不可用时，那么再用 RDB 恢复，这样就做到了两者的相互结合，也就是说 Redis 恢复会先加载 AOF，当 AOF 有问题时再加载 RDB，从而达到**冷热备份**的目的。
   - **冷备份**：一般发生在数据库已经正常关闭的情况下，当正常关闭时，会提供给我们一个完整的数据库，但冷备份的数据往往不够实时。
   - **热备份**：是在数据库运行的情况下，采用archivelog mode（归档日志模式）方式备份数据库的方法，其备份的数据具有很好的实时性。
   - **冷热备份**：在发生问题时，如果同时存在一个冷备份和不久的热备份文件，那么就可以利用这些资料来恢复更多的信息。

### 2.0. Redis数据分区？

![1632400451957](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1632400451957.png)

#### 分区概念

在Redis中，数据分区（分片），是一种在多个 Redis 实例之间拆分数据的技术，将全部 Key 数据根据分区规则分成多个子集，并存储到 Redis 实例中。

#### 分区优点

- **带宽与算力的提升**：单机 Redis 的网络 I/O 能力和计算资源都是有限的，把请求分散到多台机器，可以充分利用多台机器的计算能力与网络带宽，有助于提高 Redis 总体的服务能力。
- **内存的横向扩展**：即使 Redis 的服务能力能够满足应用需求，但是随着存储数据的增加，单台机器受限于机器本身的内存，把数据分散到多台机器上存储，使得 Redis 内存可以横向扩展。

#### 分区缺点

- **不支持跨实例的命令与事务**：涉及多个 key 的操作通常不会被支持，比如不能直接使用交集指令来对两个集合求交集，因为这些 key 可能被存储到不同的 Redis 实例中，同理，操作多个key时也不能使用Redis事务。
- **大 Key 数据集无法再分片**：由于是基于 key 进行数据分区的，因此无法使用单个大 key 对数据集进行分区，比如一个很大的排序集或列表，只能作为一个 key 存储进一个 Redis 实例中，而不能对其再分片。
- **备份管理要复杂得多**：数据分区后，如果需要对数据进行备份，则必须从不同的 Redis 实例同时收集 RDB 和 AOF 文件。
- **扩缩容时可能需要对数据再平衡**：数据分区后，在集群运行时增加或者删除 Redis 节点，对于散列分区方式，需要对数据进行再平衡，使用预分片可以较好地解决这个问题。
  - **预分片**：可以在刚开始时就开启多个 Redis 实例，比如 32 或 64 个实例来作为我们的工作集群，当一台物理机器内存不够时，可以把其中一些实例移动到第二台存储更大的物理机上，这样就可以保证在集群Redis 实例数不变的情况下，又达到了扩充机器内存的目的。

#### 分区方案

##### 范围分区

- **特点**：将分片规则对象范围映射到 Redis 实例。
  - 比如，从 ID 0到 ID 10000的用户将进入实例 R0，而从 ID 10001到 ID 20000的用户将进入实例 R1 等。
- **优点**：
  - **简单有效**。
  - **分片可顺序访问**：使用分片字段进行范围查找时，连续分片可快速定位分片进行快速查询，有效避免跨分片查询的问题。
- **缺点**：
  - **数据分散度容易倾斜**：容易出现某分片过热的问题，即切分后热点数据可能都落在同一分片上，成为系统性能的瓶颈。
  - **需要管理映射表**：每种分片规则对象都需要管理一个将范围映射到 Redis 实例的表，对比其他分区方案效率低得多，因此，Redis 中的范围分区通常是不可取的。

##### 哈希分区

- **特点**：采用 hash 取模的切分方式。
  1. 获取 Key 并使用散列函数（比如 `crc32` 散列函数）将其转换为数字值。
  2. 对这个数字值使用模运算，将它转换成一个 0 到 3 之间的数字，映射到 4个 Redis 实例的其中一个。
- **优点**：**数据分散度高**：不容易出现热点和并发访问的瓶颈。
- **缺点**：**分片无法顺序访问**：容易面临跨分片查询的复杂问题。

###### 节点取余 | 数据迁移率大

![1632448521969](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1632448521969.png)

- **概念**：hash（key）% node。
- **优点**：简单。
- **缺点**：数据迁移率大，当节点数量变化时，数据节点的映射关系需要重新计算，会导致数据的重新迁移。
  - **翻倍扩容**：扩容时通常采用翻倍扩容，可以避免数据映射被全部打乱，导致全量迁移的情况。
  - **一致性哈希**：可以减小影响的范围。
- **适用场景**：常用于数据库的分库分表规则，**不推荐**用于 Redis 数据分区。
  - 一般采用预分区的方式，提前根据数据量规划好分区数，比如提前划分为 512 或 1024 张表，保证可支撑未来一段时间的数据容量，再根据负载情况将表迁移到其他数据库中。

###### 一致性哈希 | 数据不均匀

![1632447059227](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1632447059227.png)

- **概念**：hash（key） + 顺时针优化取余。
  1. 一致性 Hash 可以很好的解决稳定问题，可以将所有的存储节点排列在**首尾相接**的 Hash 环上。
  2. 每个key 在计算 Hash 后，会**顺时针**找到遇到的第一组存储节点来存放。
  3. 而当有节点加入或退出时，仅影响该节点在 Hash 环上顺时针相邻的后续节点，将数据从该节点接收或者给予。
- **优点**：加减节点只影响 Hash 环中顺时针方向的相邻节点，对其他节点无影响。
- **局限**：
  - **数据不均匀**：在节点太少时，容易出现节点分布不均匀，从而导致数据倾斜。
  - **数据未命中**：增加节点时，会造成 Hash 环中部分数据无法命中，需要进行手动处理。
  - **数据有迁移**：虽然只影响邻近节点，但仍然有数据迁移。
  - **需翻倍扩容**：在加减节点时，需要增加一倍或者减少一半，才可以保证数据和负载的均衡。
- **适用场景**：当使用少量节点时，节点变化将大范围影响 Hash 环中的数据映射，且容易出现数据倾斜，因此，一致性哈希只适合**数据节点较多**的分布式方案。

###### 虚拟槽分区 | 数据可均匀分配

![1632450278219](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1632450278219.png)

- **概念**：使用分散度良好的哈希函数，把所有数据映射到一个**固定范围**的整数集合中，每一个节点负责维护一部分槽以及该槽所映射的数据。
  - 把这个范围的整数定义为**槽**（slot），其范围一般远远大于节点数，比如 Redis Cluster 槽范围是 0 ~ 16383。
- **优点**：
  - **容易扩缩容**：因为解耦了数据和节点之间的关系，降低了节点扩缩容的难度。
    1. 如果增加一个节点 6，就需要从节点 1 ~ 5 获得部分槽分配到节点 6 上。
    2. 如果想移除节点 1，需要将节点 1 中的槽移到节点 2 ~ 5 上，然后将没有任何槽的节点 1 从集群中移除即可。
  - **数据可均匀分配**：由于槽位范围固定，选用合适的算法，可以将数据均匀分配。
- **适用场景**：比如 Redis Cluster。

#### 分区实现

##### 客户端分区

- **概念**：key 在 Redis 客户端就决定了要被存储在哪个 Redis 实例中。
- **实现**：Redis Cluster（客户端分区与查询路由分区的混合体）、Redis-rb、Predis 和 Jedis 等。

![1632451244550](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1632451244550.png)

##### 代理辅助分区

- **概念**：
  - 客户端将请求发送到能够使用 Redis 协议的代理，而不是直接将请求发送到正确的 Redis 实例。
  - 该代理将确保根据配置的分区模式，把请求转发到正确的 Redis 实例，并将回复发送回客户端。
- **实现**：
  - **Twemproxy**：Twitter 开源，轻量级，是客户端和 Redis 实例之间的中间层，支持在多个 Redis 实例之间自动分区，无法平滑地扩缩容，性能一般，在 Redis Cluster出现后便不再维护。
  - **Codis**：豌豆荚开源，支持水平拓展，运维平台完善，性能较 Twemproxy 快，在国内使用的较多，在 Redis Cluster出现后便不再维护。

![1632451320936](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1632451320936.png)

##### 查询路由分区

- **概念**：把查询发送到随机 Redis 实例，该实例会确保该查询转发到正确的 Redis 节点。
- **实现**：Redis Cluster（客户端分区与查询路由分区的混合体）。

![1632451413068](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1632451413068.png)

### 2.1. Redis高可用架构？

#### 主从模式 | 无高可用 & 简单

![1632299571516](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1632299571516.png)

##### 概念

Redis 支持简单易用的**主从复制**功能， 使得从服务器成为主服务器的精确复制品。

##### 特点

（>= Redis 2.8）

- 一个 Master 可以有多个 Slave，并且 Slave 也可以有自己的 Slave ， 多级 Slave 之间可以构成一个**图状结构**。
- 复制功能默认使用**异步复制**， Slave 会以每秒 1 次的频率向 Master 报告复制流的处理进度。
- 复制功能**不会阻塞 Master**，即使有一个或多个 Slave 正在进行初次同步， Master 也可以继续处理命令请求。
- 复制功能也**不会阻塞 Slave**，只要启用了 `slave-serve-stale-data` 设置，即使 Slave 正在进行初次同步， 也可以使用旧版本的数据集来处理命令查询，不过在 Slave 删除旧版本数据集并载入新版本数据集的那段时间内， 连接请求会被阻塞。
- 复制功能只是单纯地进行**数据冗余**， 可以通过 `slave-read-only` 配置**读写分离**，让多个 Slave 来处理只读请求来提升扩展性，比说繁重的 `SORT` 命令可以交给 Slave 去运行。
- **禁止**《关闭 Master 持久化 + 自动拉起 Master 服务》：强烈建议打开 Master 的持久化，否则可能会因为延迟等问题，叠加自动拉起 Master 服务的原因，造成数据的丢失：
  1. 比如 Master 节点 A 关闭了持久化，Slave 节点 B 和节点 C 从节点 A 进行复制数据。
  2. 此时，节点 A 崩溃，然后服务被自动拉起，重启了节点 A，由于节点 A 的持久化被关闭了，所以 A 重启之后没有任何数据。
  3. 接着，节点 B 和节点 C 将从节点 A 复制数据，但是 A 的数据是空的，于是 B 和 C就把自身保存的数据副本全部都删除掉。
  4. 这种情况下，即便使用 Sentinel 来实现Redis 高可用也是非常危险的， 因为 Master 可能拉起得非常快，以至于 Sentinel 在配置的心跳时间间隔内，还没有检测到 Master 已被重启，然后 Slave 还是会执行上面数据丢失的流程。

##### 配置参数

```shell
# Slave参数
# 配置主从复制Master的IP+端口
slaveof <masterip> <masterport>
# 如果Master通过requirepass配置密码，则Slave也需要进行相应的配置
masterauth <master-password>
# 默认允许，Slave初次同步未完成时，继续使用旧数据来响应客户端，配置no会阻塞初次同步期间的所有请求
slave-serve-stale-data yes
# 默认开启读写分离，Slave只能读取数据，不能写入数据
slave-read-only yes

# Master参数
# 默认关闭无磁盘化复制，Master磁盘不生成RDB文件，直接通过网络同步给Slave
repl-diskless-sync no
# 默认为3和10，如果至少有3个从服务器，并且这3服务器的延迟值都少于10秒，Master才会执行客户端请求的写操作
# min-slaves-to-write 3
# min-slaves-max-lag 10
```

##### 原理

（>= Redis 2.8）

1. 当建立一个 Slave 时， Slave 会向 Master 发送一个 `PSYNC master_run_id offset` 命令。
2. 如果 Slave 是首次连接，由于 Master 中不存在该 Slave 的复制偏移量，所以会触发一次**完整重同步**操作， 此时，Master 开始执行 `BGSAVE`， 并在保存操作执行期间， 将所有新执行的写入命令都保存到一个缓冲区里面。
   - **部分重同步**：
     1. Master 会为被发送的复制流创建一个缓冲区 `in-memory backlog`， 并且 Master 和所有 Slave 之间都记录一个复制偏移量 `replication offset` 和一个主服务器ID `master run id`，当出现网络连接断开时， Slave 重新连接后会向 Master 请求继续执行原来的复制进程。
     2. 如果 Slave 记录的主服务器ID `master run id` 和当前要连接的主服务器ID `master run id` 相同， 并且 Slave 记录的偏移量 `replication offset` 所指定的数据仍然保存在 Master 的复制流缓冲区 `in-memory backlog` 里面， 那么 Master 会向 Slave 发送断线时缺失的那部分数据， 然后复制工作可以继续执行。
     3. 否则，Slave 就要执行**完整重同步**操作。
3. 当 Master  `BGSAVE` 执行完毕后， Master 将执行保存操作所得的 .rdb 文件发送给 Slave， Slave 接收这个 .rdb 文件， 并将文件中的数据载入到内存中。
4. 之后，Master 会以 Redis 命令协议的格式， 将写命令缓冲区中积累的所有内容都发送给 Slave，Slave 则实时同步这些数据。
5. 如果主从复制期间 Slave 断开连接，在自动重连后，会使用 `PSYNC master_run_id offset` 命令来进行同步，Master 会以**增量复制**的形式，向 Slave 发送断线时缺失的那部分数据， 然后复制工作可以继续执行。
   - Slave 会在连接断开后进行自动重连：
     - 在 Redis 2.8 版本之前， 断线之后重连的 Slave 总要执行一次**完整重同步**操作。
     - 从 Redis 2.8 版本开始， Slave 可以根据 Master 的情况来选择执行**完整重同步**还是**部分重同步**。
       1. 如果此时 Master 是 Redis 2.8 之前的版本，那么 Slave 使用 `SYNC` 命令来进行同步。
       2. 如果此时 Master 是 Redis 2.8 或以上版本，那么 Slave 使用 `PSYNC master_run_id offset` 命令来进行同步。

##### 优点

- **部署简单**，仅使用两个节点即可构成主从模式。
- 可以通过**读写分离**，来避免读和写同时不可用。

##### 缺点

- 一旦 Master 节点出现故障，主从节点就**无法自动切换**，直接导致 SLA 服务等级下降。
  - **解决方案**：添加哨兵监控。
- 所有的 Slave 节点数据的复制和同步都由 Master 节点来处理，会造成 Master节点压力过大。
  - **解决方案**：可以使用**主从从结构**，通过引入从从同步，以减少主从同步的次数。

##### 适用场景

一般适合业务**发展初期**，并发量低，运维成本低的情况。

#### 哨兵模式 | 高可用 & 多读

![1632320226337](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1632320226337.png)

##### 概念

Sentinel，哨兵，是 Redis 高可用的解决方案，可以监视一个或者多个 Redis Master 服务，以及这些 Master 服务的所有从服务，当某个 Master 服务宕机后，会把这个 Master 下的某个从服务升级为 Master，从而替代已宕机的 Master 继续工作。

##### 特点

- Redis Sentinel 用于管理多个 Redis 服务器实例， 它会执行以下**三个任务**：
  1. **监控**：Monitoring，Sentinel 会不断地检查 Master 和 Slave 是否运作正常。
  2. **提醒**：Notification，当被监控的某个 Redis 服务器出现问题时， Sentinel 可以通过 API 向管理员或者其他应用程序发送通知。
  3. **自动故障迁移**：Automatic failover。
     - 当一个 Master 不能正常工作时， Sentinel 会开始一次自动故障迁移操作， 它会将失效 Master 的其中一个 Slave 升级为新的 Master， 并且让失效 Master 的其他 Slave 改为复制新的 Master。
     - 当客户端试图连接失效的 Master 时， 集群也会向客户端返回新 Master 的地址， 使得集群可以使用新 Master 代替失效 Master。
- Redis Sentinel 是一个分布式系统， 可以在一个架构中运行多个 Sentinel 进程， 这些进程会使用**流言协议**来接收关于 Master 是否下线的信息， 并使用**投票协议**来决定是否执行自动故障迁移， 以及选择哪个 Slave 作为新的 Master。
  - **流言协议**：gossip protocols，是一种计算机对计算机的沟通协议，是一种复制**没有强一致性**需求状态的方式，即使在通讯失败或消息丢失的情形下，更新也可以在期望的时间内传播，通常用来解决其它方式难以解决的分布式问题，比如底层的网络结构非常复杂，或者流言协议是最有效的解决方案等。
  - **投票协议**：agreement protocols，比如 Raft。
- 虽然 Redis Sentinel 释出为一个单独的可执行文件 `redis-sentinel`，但实际上只是一个运行在**特殊模式**下的 Redis 服务器， 可以在启动一个普通 Redis 服务器时通过给定 `--sentinel` 选项来启动 Redis Sentinel 。

##### 配置参数

```shell
# 配置监控 127.0.0.1：6379 的 mymaster 服务器，并且客观下线需要取得2个(quorum)哨兵的同意
# 但是无论该值配置了多少，都需要多数哨兵的选举后，才能发起一次自动故障转移
sentinel monitor mymaster 127.0.0.1 6379 2
# 配置Master服务器密码（如果Master有配置的话）
sentinel auth-pass <master-name> <password>
# 配置判定mymaster主观下线的毫秒数
sentinel down-after-milliseconds mymaster 60000
# 配置主从切换的超时时间，需要自动故障转移时，如果当前哨兵没有去执行，那么在超过这个时间后，会由其他的哨兵来进行处理
sentinel failover-timeout mymaster 180000
# 配置在执行故障转移时，同步新mymaster的最大Slave并行数，如果全部Slave一起对新Master进行同步，由于在Slave载入RDB时会阻塞客户端请求，所以可能会造成所有Slave在短时间内全部不可用的情况出现
sentinel parallel-syncs mymaster 1
```

##### 运行命令

```shell
# 启动命令
# 运行纯Sentinel服务器
redis-sentinel /path/to/sentinel.conf
# 在Redis Server上运行哨兵
redis-server /path/to/sentinel.conf --sentinel

# 运维命令
# 查看imooc-master下的master节点信息
sentinel master imooc-master
# 查看imooc-master下的slaves节点信息
sentinel slaves imooc-master
# 查看imooc-master下的哨兵节点信息
sentinel sentinels imooc-master
```

##### 自动发现原理

###### Sentinel 发现

一个 Sentinel 可以与其他多个 Sentinel 进行连接， 各个 Sentinel 之间可以互相检查对方的可用性， 并进行信息交换，通过**发布与订阅**功能，向频道 `__sentinel__:hello` 发送信息，来自动发现正在监视相同 Master 的其他 Sentinel ，无须为运行的每个 Sentinel 分别设置其他 Sentinel 的地址。

1. **发布**：每个 Sentinel 会以每两秒一次的频率， 通过发布与订阅功能， 向被它监视的所有 Master 和 Slave 的 `__sentinel__:hello` 频道发送一条信息， 信息中包含了 Sentinel 的 IP、端口和运行 ID `runid`。
   - Sentinel 发送的信息中还包括完整的 Master 当前配置。 如果一个 Sentinel 包含的 Master 配置比另一个 Sentinel 发送的配置要旧， 那么这个 Sentinel 会立即升级到新配置上。
2. **订阅**：每个 Sentinel 都订阅了被它监视的所有 Master 和 Slave 的 `__sentinel__:hello` 频道， 查找之前未出现过的 Sentinel，当一个 Sentinel 发现一个新的 Sentinel 时， 它会将新的 Sentinel 添加到一个列表中， 这个列表保存了已知的、正在监视同一个 Master 的所有其他 Sentinel 。
   - 在将一个新 Sentinel 添加到监视 Master 的列表上面之前， Sentinel 会先检查列表中是否已经包含了和要添加的 Sentinel 拥有相同运行 ID 或者相同地址（IP+端口）的 Sentinel ， 如果是，则 Sentinel 会先移除列表中已有的那些拥有相同运行 ID 或者相同地址的 Sentinel ， 然后再添加新 Sentinel 。

###### Slave 发现

与此类似，也不必在配置文件中，手动列出 Master 属下的所有 Slave， 因为 Sentinel 可以通过询问 Master 来获得所有 Slave 的信息。

- 在一般情况下， 每个 Sentinel 会以**每10秒1次**的频率向它已知的所有 Master 和 Slave 发送 `INFO [section]` 命令。 
- 当一个 Master 被 Sentinel 标记为客观下线时， Sentinel 向下线 Master 的所有 Slave 发送 `INFO [section]`命令的频率会从每10秒1次改为**每秒1次**。

##### 故障判定原理

###### Sentinel 定期任务

1. 每个 Sentinel 以**每秒1次**的频率向它所知的 Master、Slave 以及其他 Sentinel 实例发送一个 `PING` 命令。
2. 如果一个实例距离最后一次有效回复 `PING` 命令的时间超过 `down-after-milliseconds` 选项所指定的值， 那么这个实例会被 Sentinel 标记为**主观下线**。 
3. 如果一个 Master 被标记为主观下线， 那么正在监视这个 Master 的**所有 Sentinel** 要以每秒1次的频率确认Master 是否的确进入了主观下线状态。
   - 当 Master 重新向 Sentinel 的 `PING` 命令返回有效回复时，Master 的主观下线状态就会**被移除**。
4. 如果一个 Master 被标记为主观下线， 并且有足够数量（>= `quorum` ）的 Sentinel 在指定的时间范围内同意这一判断， 那么这个主服务器被标记为**客观下线**。
   - 当没有足够数量（>= `quorum` ）的 Sentinel 同意主服务器已经下线， Master 的客观下线状态就会**被移除**。

###### 主观下线

主观下线，Subjectively Down， 简称 **SDOWN**，指的是单个 Sentinel 实例对服务器做出的下线判断，如果一个服务器（Master、Slave 和其他 Sentinel）没有在 `master-down-after-milliseconds` 选项所指定的时间内， 对向它发送 `PING` 命令的 Sentinel 一直返回**无效回复**， 那么 Sentinel 就会将这个服务器标记为**主观下线**。

- 服务器对 PING 命令的**有效回复**，可以是以下三种回复的其中一种：
  - 返回 `+PONG` 。
  - 返回 `-LOADING` 错误。
  - 返回 `-MASTERDOWN` 错误。
- 如果服务器返回除以上三种回复之外的其他回复， 又或者在指定时间内**没有回复** `PING` 命令， 那么 Sentinel 认为服务器返回的**回复无效**。
- 只有一个 Sentinel 将服务器标记为**主观下线**，并不一定会引起服务器的自动故障迁移，只有在足够数量（>= `quorum` ）的 Sentinel 都将一个服务器标记为主观下线之后， 服务器才会被标记为**客观下线**， 这时**自动故障迁移**才会执行。

###### 客观下线

客观下线，Objectively Down， 简称 **ODOWN**，指的是多个 Sentinel 实例在对同一个服务器做出 **SDOWN** 判断， 并且通过  `SENTINEL is-master-down-by-addr` 命令互相交流之后， 得出的服务器下线判断。

- 一个 Sentinel 可以通过向另一个 Sentinel 发送 `SENTINEL is-master-down-by-addr` 命令来询问对方是否认为给定的服务器已下线。
- 从主观下线状态切换到**客观下线**状态，并没有使用严格的法定人数算法， 而是使用了**流言协议**：
  - 如果 Sentinel 在给定的时间范围内， 从其他 Sentinel 那里接收到了足够数量（>= `quorum` ）的 Master 下线报告， 那么 Sentinel 就会将 Master 的状态从主观下线改变为**客观下线**。 
  - 如果之后其他 Sentinel 不再报告 Master 已下线， 那么客观下线状态就会**被移除**。
- 客观下线条件只适用于 Master，对于任何其他类型的 Redis 实例， Sentinel 在将它们判断为下线前不需要进行协商， 所以  Slave 或者其他 Sentinel 永远不会达到客观下线的条件。
- 只要一个 Sentinel 发现某个 Master 进入了客观下线状态， 这个 Sentinel 就可能会被其他 Sentinel 推选出， 并对失效的 Master 执行自动故障迁移操作。

##### 自动故障转移

###### Leader Sentinel 选举

当一个 Master 被判定为**客观下线**后，监视这个 Master 的所有Sentinel会通过 `Raft` 选举算法，选出一个Leader Sentinel 去执行故障转移 `failover` 操作。

- **选举一致性保证**：
  1. 使用 Raft 算法来选举 Leader Sentinel，可以确保在一个给定的 `epoch`（纪元）里， **只有一个** Leader 产生，因为更高的 `epoch` 配置总是优于较低的 `epoch` 配置 ， 每个 Sentinel 都会主动使用更新的 `epoch` 来代替自己的配置。
  2. 当出现网络分区时， 一个 Sentinel 可能会包含了较旧的配置， 而当这个 Sentinel 接到其他 Sentinel 发来的版本更新的配置时， Sentinel 就会对自己的配置进行更新。
  3. 如果想要在出现网络分区时仍然保持一致性，可以使用 `min-slaves-to-write` 选项， 让 Master 在连接的 Slave 实例数**少于给定数量时，停止执行写操作**，并且在每个运行 Redis Master或 Slave 的机器上运行 Redis Sentinel 进程。

###### Raft 算法

http://thesecretlivesofdata.com/raft/#home、https://raft.github.io

![1632381533618](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1632381533618.png)

Raft，是一种共识算法，旨在使其易于理解，在容错和性能上与Paxos媲美，不同之处在于它被分解为相对独立的子问题（Leader Election 和 Log Replation），并且清晰明了地解决了分布式系统实际所需的主要部分。

- **Leader Election**：领导者选举，这部分主要就是通过用随机 `election timeout` 的方式，使得能够很快地选出 Leader，并且通过 `heartbeat（AppendEntries RPC，心跳续约）` 周期性地通知 Follower，以维持 Leader 的状态。这大大简化了 Leader 选举的算法，加快了选举速度。**选举流程**为：
  1. 每个 Follower 或者 Candidate 都有一个 election timer，用来计算随机 `election timeout` 时间。
  2. 当 Follower 没有收到 Leader 的 `heartbeat（AppendEntries RPC，心跳续约）` 或者 Candidate 的 `RequestVote RPC（选举发起）`时，则等待 election timer 超时。
  3. 一旦 Follower 超时，则会立刻变为 Candidate 身份，令 `currentTerm++` 发起新一轮 election term 的选举，它会先给自己投票，并重置自己的 election timer，然后广播 `RequestVote RPC（选举发起）` 给所有 server。
  4. 这些 server 如果收到某个 Candidate 的 `RequestVote RPC（选举发起）` 时，会经过一系列规则来判断是否能够进行投票。
  5. 如果在 election timer 超时前，该 Candidate 能收到超过半数 server 的投票，则会立刻变为 Leader，并发送 `heartbeat`，以维持 Leader 身份。
- **Log Replation**：日志复制（自己写的，可能理解会有出入，在分布式章节再补充完整）。
  1. 在客户端请求 Leader 更改集群数据时，Leader 会在数据更改后，先把同步日志发送到它所有的 Follower。
  2. 当超过半数的 Follow 的同步日志写入成功后，Leader 才会对该数据的更改进行提交，并响应客户端。
  3. 最后，Leader 再把提交结果同步到它所有的 Follower，通知他们 Leader 数据的提交。

###### Master 选择

当选举出 Leader Sentinel 后，Leader Sentinel 会根据以下规则，在失效 Master 属下的 Slave 当中，选择出新的 Master：

1. 先淘汰主观下线 | 已断线 | 最后一次回复 `PING` 命令时间大于5秒钟 | 与失效 Master 断开连接时长超过 10 倍主观判断时长 `down-after-milliseconds` 的 Slave 节点。
2. 选择配置 `slave-priority` 最高的 Slave 节点，如果没有，则继续选择。
3. 选择复制偏移量 `replication offset` 最大的 Slave 节点，复制偏移量越大，说明数据复制的越完整。
4. 如果复制偏移量不可用，或者 Slave 的复制偏移量相同， 则选择运行 ID `run_id` 最小的 Slave 节点，`run_id` 越小，说明重启次数越少。

###### 故障转移流程

一次故障转移操作由以下步骤组成：

1. 某个 Sentinel 发现 Master 已经进入**客观下线**状态。
2. 该  Sentinel 对当前 `epoch` （纪元）进行自增， 并尝试在这个 `epoch` 中当选 Leader Sentinel 。
3. 如果 Leader Sentinel 当选失败， 则在设定的故障迁移超时时间 `failover-timeout` 的两倍之后， 会继续重新尝试当选。
4. 如果 Leader Sentinel 当选成功， 则根据Master选择规则，选出一个 Slave，向其发送 `SLAVEOF NO ONE` 命令，让它转变为 Master。
5. 然后， Leader Sentinel 通过**发布与订阅**功能， 将更新后的配置传播给所有其他 Sentinel，让其他 Sentinel 对它们自己的配置进行更新。
   - Sentinel 的状态会被持久化在 Sentinel 的配置文件中，每当 Sentinel 接收到一个新的配置， 或者当Leader Sentinel 为 Master 创建一个新的配置时， 这个配置会与 `epoch` 一起被保存到自己的磁盘里，意味着停止和重启 Sentinel 进程都是安全的。
6. 接着， Leader Sentinel 会向已下线 Master 的其他 Slave 发送 `SLAVEOF host port` 命令， 让它们去复制新的 Master。
   - 当Redis 实例被重新配置，无论是被设置成 Master，或者Slave，或者其他 Master 的 Slave，Sentinel 都会向这个被重新配置的实例发送一个 `CONFIG REWRITE` 命令， 从而确保这些配置会被这个实例持久化在它自己的硬盘里。
7. 最后，当所有 Slave 都已经开始复制新的 Master 时， Leader Sentinel 则结束这次故障迁移操作。

##### 优点

监控、提醒、自动故障转移，从而实现 Redis 的高可用。

##### 缺点

如果写请求较多，当集群 Slave 节点数量多了后，Master 节点同步数据的压力会非常大。

##### 适用场景

适合**读远多于写**的业务场景，比如在秒杀系统中，用来缓存活动信息。

#### 集群模式 | 高可用 & 多写

![1632464692288](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1632464692288.png)

##### 概念

Redis Cluster，于 3.0 版本推出，是一个分布式容错的高可用实现。

##### 特点

- 采用**哈希虚拟槽**的数据分区方案，把 key 分布到各个 Master 节点上，每个 Master 后跟若干个 Slave 做主从切换。

- 采用**客户端查询分区+服务端查询路由**的分区实现，客户端可以连接任意 Master 节点，集群内部会按照不同的 key，把请求转发到不同的 Master 节点。

- 可使用的功能只是普通单机 Redis 所有功能的一个**子集**：

  - 不支持同时使用多个键的 Redis 命令， 因为执行这些命令需要在多个 Redis 节点之间移动数据， 并且在高负载的情况下， 这些命令将降低 Redis 集群的性能， 并导致不可预测的行为。
  - 不支持多数据库功能， 只能使用默认的 `0` 号数据库， 并且不能使用 `SELECT index` 命令。

- Redis Cluster 的设计目标主要为高性能、高可用和高扩展，抛弃了一部分数据一致性，即 **AP**：

  - **数据一致性**：由于 Redis 主从复制使用的是异步复制，在某些情况下，如果 Master 宕机但未同步至 Slave，可能会导致丢失写入，可通过 `WAIT` 命令实现，使丢失写入的可能性大大降低。

    - **`wait` 命令不能保证 Redis 强一致性**：如果写操作已经被传送给一个或多个 Slave节点，当 Master 发生故障，虽然使用 `wait` 命令可以**极大概率**（不保证100%）地提升一个受到写命令的 Slave 节点为Master，但由于其他原因，比如 `slave-priority`， Sentinel 还是有可能去提升其他未同步写命令的 Slave 节点，造成同步写操作的丢失。

    |          | WAIT                                                         |
    | -------- | ------------------------------------------------------------ |
    | 出现版本 | Redis 3.0                                                    |
    | 命令格式 | wait numslaves timeout                                       |
    | 含义     | 阻塞当前客户端 timeout 毫秒，为 0 则代表永远阻塞，直到所有以前的写命令都成功传输并被指定数目的 Slave 确认，命令再返回；或者如果发生 timeout 超时，即使指定数目的 Slave 还没有确认，命令也会返回。 |
    | 返回值   | 返回已处理至该偏移量的 Slave 个数                            |

  - **高可用**：当集群中一部分节点故障后，集群整体依然能响应客户端读写请求。

    - **自动故障转移**：节点间定时互 `Ping` ，当超过一半 Master 判定某节点失败，则标记为 FAIL，然后向集群广播节点下线的消息，如果下线节点是带有哈希槽的 Master，则会在它的 Slave 中挑选出一个来进行主从替换。

  - **高性能**：操作某个 key 时，不会先找到节点再处理，而是直接重定向到目标 Redis 实例，相较代理分片少了 proxy 的连接损耗。

    - 但是在进行 multiple key 操作时，由于 keys 可能会散列到不同的 Redis 实例上，造成操作的非原子性，加之 Redis Cluster 本身就不支持 multiple key 操作，此时可以使用 **hash tags** 哈希标签， 将 keys 散列到同一个 slot 上，以满足进行 multiple key 的需求。
      - **hash tags**：哈希标签，使用 {} 确保两个 key 都在散列到同一个哈希槽，比如 key1 {user1000}.following 和 key2 {user1000}.followers，由于使用了哈希标签，Redis Cluster 在散列时，只会使用 { } 里的字符串进行计算，所以 key1 和 key2 将会落到同一个哈希槽中。

  - **高拓展**：不存在中心节点或者代理节点，同时最大支持线性拓展 1000 个节点，把新节点加入集群后，可以通过命令平均分配已有节点的哈希槽。

##### 配置参数

```shell
# 开启集群模式
cluster-enabled yes
# 每一个节点都需要有这么一个配置文件，3主3从则一共需要6份，用于存储集群模式下的集群状态等信息，由Redis自己维护，而这些信息会相互告知其他所有节点。如果要重新创建集群，只需要把这个文件删除掉就行。
cluster-config-file nodes-201.conf
# 节点超时时限，如果发生超时则会被认定为PFAIL，如果超过半数其他Master认定为PFAIL，则会被集群认定为FIAL，然后会进行主从切换
cluster-node-timeout 5000
# 开启AOF
appendonly yes
```

##### 运行命令

```shell
# Redis3.x旧版集群构建方式，需要使用redis-trib.rb来构建集群，最新版使用C语言来构建了，这个要注意
# ./redis-trib.rb create --replicas 1 127.0.0.1:7000 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005

# 新版Redis集群构建方式
# 创建集群，主节点和从节点比例为1，1-3为主，4-6为从，1和4，2和5，3和6分别对应为主从关系，这也是最经典用的最多的集群模式
redis-cli --cluster create ip1:port1 ip2:port2 ip3:port3 ip4:port4 ip5:port5 ip6:port6 --cluster-replicas 1

# 集群客户端
redis-cli -c -p 7000

# 检查集群信息
redis-cli --cluster check 127.0.0.1:6379
```

##### 集群节点

###### 节点属性

```shell
# 获取集群所有节点的描述信息
$ redis-cli --cluster nodes
# 结果：节点ID，IP，端口号，节点标志，最后PING发送时间，最后PONG接收时间，连接状态，哈希槽范围
d1861060fe6a534d42d8a19aeb36600e18785e04 :0 myself - 0 1318428930 connected 0-1364
```

- **节点ID**：
  1. 每个节点在集群中都有一个独一无二的 ID，是一个十六进制表示的 160 位随机数，用于标识集群中的每个节点，一个节点可以改变它的 IP 和端口号， 但重启的话不会改变它的节点 ID 。
  2. 节点ID，在节点**第一次启动**时由 `/dev/urandom` 生成，然后保存到配置文件中， 只要这个配置文件不被删除，该节点就会一直沿用这个 ID。 
  3. 集群可以自动识别出 IP + 端口号的变化， 并把这一信息通过 `Gossip` 协议广播让其他节点知道。

- 如果该节点是从节点的话，那么它会记录主节点的节点 ID 。 如果这是一个主节点的话，那么主节点 ID 这一栏的值为 `0000000` 。

###### 节点任务

- **保存数据**：持有键值对数据。
- **状态映射记录**：记录集群的状态，包括键到正确节点的映射。
- **自动发现与选举**：自动发现其他节点，识别工作不正常的节点，并在有需要时，在从节点中选举出新的主节点。

###### 节点通信

集群中的每个节点都会与其他节点建立起**集群连接**， 该连接是一个 TCP 连接， 使用二进制 `Gossip` 协议进行通讯，用于：

- **节点发现**：传播关于集群的信息，以此来发现新的节点。
- **心跳检测**：向其他节点发送 `PING` 数据包，以此来检查目标节点是否正常运作。
  - **集群内互通**：集群节点总是应答来自集群连接端口的连接请求。
  - **非集群内只允许PING**：可以对接收到的 `PING` 数据包进行回复， 即使这个 `PING` 数据包来自不可信的节点，但除了 `PING` 之外， 集群节点会拒绝其他所有非来自集群节点的数据包。
- **事件传播**：在特定事件发生时，发送集群信息。

###### 节点自动发现

如果将一个新节点添加到一个集群中， 只要管理员使用 `CLUSTER MEET` 命令显式地指定了可信关系， 集群就可以自动发现其他节点，最终该节点会与集群中已有的其他所有节点连接起来。

- **握手规则**：
  - A 节点刚加入集群中，B 为集群中的某个节点，此时如果管理员显式地向 A 发送 `CLUSTER MEET ip port` 命令， 那么 A 会向 B 发送 `MEET` 信息， 强制让 B 节点承认 A 属于集群中的一份子。
  - A、B为集群中的某个节点，A 刚承认 C 为集群中的一份子，此时如果 A 向 B 传播第三者 C 的信息， 那么 B 也会把 C 识别为集群中的一份子， 并尝试连接 C 。

- **作用**：可以防止不同的 Redis 集群由于 IP 地址的变更或者其他网络事件的发生，而产生意料之外的问题，从而使得集群更具健壮性。当节点的网络连接断开时， 它会主动连接其他已知的节点。

##### 分区实现原理

###### 虚拟哈希槽模型

Redis 集群的键空间被分割为 `16384` 个槽（slot）， 集群的最大节点数量也是 `16384` 个，但推荐的最大节点数量为 1000 个左右，每个主节点都负责处理 `16384` 个哈希槽的其中一部分，其键的映射算法为：

```shell
# CRC16 算法可以很好地将各种不同类型的键，平稳地分布到 16384 个槽里面
HASH_SLOT = CRC16(key) mod 16384
```

###### 为什么哈希槽数为16384个？

`CRC16` 算法产生的 hash 值有 16 bit，即可产生 2 ^ 16 = 65536 个值，换句话说，值是分布在 0 ~ 65535 之间，那 Redis 在做 `mod` 运算的时候，为什么不 `mod` 65536，而是选择 `mod` 16384 呢？

1. **槽位数不宜过大**：
   1. 如果槽位为 65536，那么节点发送 `PING/PONG` 心跳包消息头所占用的空间会达到 8k （65536 / 8），过于庞大，传输时浪费带宽。
   2. 而使用 16384 个槽位，心跳包消息头所占用的空间仅为 2k（16384 / 8），大小还能接受。
2. **槽位数不宜过小**：
   1. Redis Master 节点数量基本不可能大于 1000 个，因为集群节点越多，心跳数据包消息体携带的数据就越多，就越可能导致网络拥堵。
   2. 同时，Redis Master 的哈希槽配置，是通过一张 bitmap 的形式来保存的，其填充率为 slots / N，N为节点数目，在 N 最大为 1000 的情况下，如果槽位数越小，bitmap 填充率就越小，导致 bitmap 在传输过程中的压缩率就越高，越消耗 CPU 资源。
3. 因此，16384 个插槽可以确保在最大 1000 个 Master 的情况下，仍然有足够的插槽，使得哈希槽配置作为原始 bitmap 进行传播，是综合了心跳包大小、网络带宽、压缩率等方面考虑的结果，16384 个插槽处于**合适的范围**内，能够满足业务需求并且更有优势。

###### 请求路由原理

1. **服务端路由**：集群节点不能代理客户端的命令请求， 客户端应该在节点返回 `-MOVED` 或者 `-ASK` 转向错误时， 自行将命令请求转发至其他节点。
   - 当节点需要让一个客户端**长期地**将针对某个槽的命令请求发送至另一个节点时， 节点会向客户端返回 `-MOVED` 转向。
   - 当节点需要让客户端仅仅在**下一个命令**请求中转向至另一个节点时， 节点会向客户端返回 `-ASK` 转向。
2. **客户端路由**：如果客户端可以将键和节点之间的映射信息保存起来， 可以有效地减少可能出现的转向次数， 籍此提升命令执行的效率。

###### MOVED 转向

1. 一个 Redis 客户端可以向集群中的任意节点（包括从节点）发送 `GET key` 命令请求。

2. 集群节点会对命令请求进行分析， 如果该命令是集群可以执行的命令， 那么节点会查找这个命令所要处理的键所在的哈希槽。

3. 如果要查找的哈希槽正好就处于当前节点中，则接收到的命令由当前节点负责处理。

4. 如果所查找的哈希槽不处于当前节点中，则当前节点会先查看自身内部所保存的哈希槽到节点 ID 的**映射记录**，然后向客户端回复一个 `-MOVED` 转向错误。

   ```shell
   GET x
   # GET命令收到一个 -MOVED 转向错误
   # x真正所在的目标哈希槽，目标节点IP，目标节点端口号
   -MOVED 3999 127.0.0.1:6381
   ```

5. 客户端收到 `-MOVED` 转向错误后，根据目标节点 IP 与端口号，会再向目标节点重新发送一次 `GET key`命令请求。

   - 为了让客户端的转向操作**尽可能地简单**， 节点在 `-MOVED` 错误中直接返回了目标节点的 IP 和端口号， 而不是目标节点的 ID 。

6. 如果客户端在重新发送 `GET key` 命令时，集群刚好又更改了 key 的 slot 配置， 此时客户端请求目标哈希槽，会再次收到 `-MOVED` 转向错误， 需要再次向新的目标节点重新发送一次 `GET key`命令请求。

7. 客户端会循环以上操作，直到该命令请求成功，然后记录下该成功请求的哈希槽的目标节点信息，在下次执行相同 key 命令时，加快正确节点的寻找速度。

   - 当集群处于稳定状态时， 所有客户端最终都会保存有一个哈希槽至节点的映射记录，使得集群非常高效，此后客户端可以直接向正确的节点发送命令请求， 而无须转向、代理或者请求其他可能发生单点故障的 Redis 实例。

###### 集群重分片

Redis 集群支持在集群运行的过程中添加或者移除节点，无论是添加还是删除，都需要将哈希槽从一个节点移动到另一个节点，如果添加一个新节点到集群， 则需要将其他已存在节点的槽移动到一个空白的新节点里面；如果从集群中移除一个节点， 则需要将被移除节点的所有槽移动到集群的其他节点上面去。

- **使用 Cluster 子命令**：管理集群节点的哈希槽转换表。

| Cluster子命令                              | 作用                                                         |
| ------------------------------------------ | ------------------------------------------------------------ |
| CLUSTER ADDSLOTS slot1 [slot2] ... [slotN] | 用于添加并指派节点哈希槽，当哈希槽被添加并指派之后， 节点会把这一信息通过 Gossip 协议传播到整个集群，通常在新创建集群时， 作为一种快速地将各个槽指派给各个节点的手段来使用 |
| CLUSTER DELSLOTS slot1 [slot2] ... [slotN] | 用于删除节点哈希槽，当哈希槽被删除之后， 节点会把这一信息通过 Gossip 协议传播到整个集群 |
| CLUSTER SETSLOT slot NODE node             | 可以将指定的哈希槽指派给节点 node                            |
| CLUSTER SETSLOT slot MIGRATING node        | 用于将给定节点 node 中的哈希槽迁移出节点 node。当一个哈希槽被设置为 `MIGRATING` 状态时， 只有当哈希槽的键仍然存在于节点时，该节点才会继续处理关于这个槽的命令请求；如果哈希槽的键不存在于节点， 那么该节点会向客户端返回一个 `-ASK` 转向错误， 以告知客户端要把命令请求发送到 哈希槽迁移后的目标节点 |
| CLUSTER SETSLOT slot IMPORTING node        | 用于将给定哈希槽导入到节点 node 中。当一个槽被设置为 `IMPORTING` 状态时，节点仅在接收到 `ASKING` 命令之后， 才会接受关于这个槽的命令请求；如果客户端没有向节点发送 `ASKING` 命令， 那么节点会使用 `-MOVED` 转向错误，把命令请求转向至真正负责处理这个槽的节点 |

- **使用 redis-trib 客户端**：Redis3.x旧版命令，`./redis-trib.rb reshard <集群中任意结点的IP+端口号>`，不推荐，会阻塞客户端。
  1. 先执行 `CLUSTER GETKEYSINSLOT slot count` 命令，让节点返回 count 个 哈希槽中的键。
  2. 然后对于命令所返回的每个键， redis-trib 会向节点 A 发送一条 `MIGRATE host port key destination-db timeout [COPY] [REPLACE] ` 命令， 把所指定的键**原子地**从节点 A 移动到节点 B，并且在移动键期间，A 和 B两个节点都会处于**阻塞状态**，以免出现竞争条件。
     - `MIGRATE host port key destination-db timeout [COPY] [REPLACE]` ：
       1. 执行该命令的节点会连接到 target 节点， 并将序列化后的 key 数据发送给 target ， 一旦 target 返回 `OK` ， 节点就将自己的 key 从数据库中删除。
       2. 从一个外部客户端的视角来看， 在某个时间点上， 键 key 要么存在于节点 A ， 要么存在于节点 B ， 但不会同时存在于节点 A 和节点 B 。
       3. 由于 Redis 集群只使用 `0` 号数据库， 所以当该命令被用于执行集群操作时， target_database 的值总是 `0` 。
       4. 尽管该命令非常高效， 但对一个键非常多、并且数据量非常大的集群来说，该命令会占用大量的时间， 有可能会导致集群没办法适应那些对于响应时间有严格要求的应用程序。

###### ASK 转向

1. 节点 A 开始迁移 哈希槽8 的键到 节点 B，迁移过程中，客户端对 A 发起了请求。
2. 由于 A 的 哈希槽8 的键被迁移了一部分到 B，导致客户端在节点 A 中没找到某个键，A 节点会向客户端返回一个 `-ASK` 转向错误。
   - 这种转向仅仅会影响**一次命令**查询， 而不是让客户端每次都直接去查找节点 B，并且只针对 16384 个槽中的其中**一个槽**， 所以对集群造成的性能损耗是属于可接受范围的。
3. 客户端接收到 `-ASK` 转向错误后， 则将命令请求的发送对象，调整为转向所指定的节点 B。
4. 接着，客户端会先发送一个 `ASKING` 命令，然后再发送真正的命令请求，否则这个针对带有 `IMPORTING` 状态槽 8 的命令请求将会被节点 B 拒绝执行。
   - **先发送 `ASKING` 命令的好处**：如果客户端出现 Bug ， 过早地将槽8 映射到了节点 B 上面， 只要不先发送 `ASKING` 命令， 后面发送的命令请求就会被 B 返回 `-MOVED` 转向错误， 并将请求转回节点 A，从而保证了在槽8 转移期间，必须先请求节点 A。
5. 节点 B 接收到客户端 `ASKING` 命令后，会将为客户端设置一个一次性的标志， 使得客户端可以执行一次针对 `IMPORTING` 状态槽8 的命令请求。
6. 命令请求成功后，客户端不会去更新所记录的槽8 到节点的映射，此时槽8 仍然是映射到节点 A ， 而不是节点 B。
7. 接下来，在键转移期间，如果客户端继续发起 槽8 的请求，会重复以上操作。
8.  一旦节点 A 的 槽8 转移工作完成，当节点 A 再次收到针对槽8 的命令请求时，就会向客户端返回 `-MOVED` 转向， 把所有关于槽8 的命令请求**长期地**转向到节点 B中。
9. 至此，完成了一次 哈希槽**动态转移**的过程。

##### 高可用原理

###### 节点失效检测

简单来说就是，当一个节点 `PING` 不通另一个节点时，则会把它标记为 `PFAIL`；当一个节点要把另一个节点从 `PFAIL`标记为 `FAIL`时， 则必须得到大部分 Master 的同意才行。

1. 当一个节点 A 向另一个节点 B 发送 `PING` 命令， 但是 B 未能在 `cluster-node-timeout` 配置的节点超时时限内返回该 `PING` 命令的回复时， 那么 A 会将 B 标记为 `PFAIL` （possible failure，代表可能已失效）。
2. 然后下次在节点 A 对 C、D、E... 发送 `PING` 命令时， 都会随机地广播**3个**它所知道的节点的信息，其中一项说明了 B 节点是否已经被 A 标记为 `PFAIL` 或者 `FAIL` 。
3. 当节点 C、D、E... 接收到 A 发来的信息时，会记下那些被 A 标记为失效的节点 B，这操作称为**失效报告**。
4. 如果在Master F 将 B 标记为 `PFAIL` 时， 根据**最近接收到的**失效报告显式， 集群中的大部分其他 Master 都认为 B 已经进入了 `PFAIL` ， 那么 E 会将那个 B 的状态标记为 `FAIL`（failure，代表已失效） 。
5. 一旦 B 被标记为 `FAIL` ，关于它的已失效信息就会被广播到整个集群， 所有接收到这信息的节点都会将 B 标记为 `FAIL` 。
   - **Slave Fail**：当 B 为 Slave 时，在 B 重新上线时， 它的 `FAIL` 标记就会被移除。
     - 由于 Slave 不需要处理任何哈希槽，是否处于 `FAIL` 状态，只决定了它能否在有需要时被提升为 Master，所以保持 Slave 的 `FAIL` 状态是没有意义的。
   - **Master Fail**：当 B 为 Master 时，在 B 经过了 （**4 倍** `cluster-node-timeout` 配置的节点超时时限 +  **10 秒钟**）后，如果 Slave 的故障转移操作仍未完成，且 B 又发生了重新上线，那么 B 的 `FAIL` 标记会被移除，集群继续使用原来的 Master B。

###### 主从复制

为了在一部分节点下线，或者无法与集群的大多数节点进行通讯的情况下， 集群仍然可以正常运作， Redis 集群对节点使用了**主从复制**功能。

1. 集群中的每个节点都有 N 个复制品， 其中 1 个复制品为 Master， 而其余的 N-1 个复制品为 Slave。
2. 比如，在创建集群时，如果为 Master B 添加了 Slave B1 ， 那么当 Master B 下线时， 集群就会将 B1 设置为新的 Master， 并让它代替已下线的 Master B ， 继续处理原来的哈希槽， 这样集群就不会因为 Master B 的下线而无法正常运作了。
3. 但是，对于在这种场景下，如果节点 B 和 B1 都下线的话， 那么 Redis 集群还是会停止运作的。

###### 从节点选举

一旦某个 Master 进入 `FAIL` 状态， 如果这个 Master 有一个或多个 Slave 存在， 那么其中一个 Slave 会被升级为新的 Master， 而其他 Slave 则会开始对这个新的 Master 进行复制。

1. **新 Master 的选举条件**：
   1. 这个节点是已下线 Master 的 Slave。
   2. 已下线 Master 负责处理的槽数量为非空。
   3. Slave 的数据需要是可靠的， 即主从节点之间复制连接的断线时长，不能超过 （`cluster-node-timeout` 配置的节点超时时限 * `REDIS_CLUSTER_SLAVE_VALIDITY_MULT` 常量）得出的积。
2. 如果一个 Slave 满足了所有条件， 那么这个 Slave 将会向集群中的其他 Master **发送授权请求**， 询问它们， 是否允许自己升级为新的 Master。
3. 如果发送授权请求的 Slave 满足以下**授权要求**， 那么其他 Master 将向 Slave 返回 `FAILOVER_AUTH_GRANTED` 授权， 同意 Slave 的升级要求：
   1. 发送授权请求的是一个 Slave， 并且它所属的 Master 处于 `FAIL` 状态。
   2. 在已下线 Master 的所有 Slave 中， 这个 Slave 的节点 ID 在排序中是最小的。
   3. 这个 Slave 处于正常的运行状态，即没有被标记为 `FAIL` 状态， 也没有被标记为 `PFAIL` 状态。
4. 一旦某个 Slave 在给定的时限内得到大部分 Master 的 `FAILOVER_AUTH_GRANTED` 授权， 那么该 Slave 就会开始执行以下**故障转移**操作：显式地向所有节点广播一个 `PONG` 数据包， 加速其他节点识别这个节点的进度， 而不是等待定时的 `PING` / `PONG` 数据包。
   1. 告知其他节点， 自己现在是 Master了。
   2. 告知其他节点， 自己是一个已升级的 `PROMOTED` Slave。
      - 如果一个带有 `PROMOTED` 标识的 Master，由于某些原因又转变成回了 Slave，那么该节点将丢失它所带有的 `PROMOTED` 标识。
   3. 告知其他节点，自己接管了由那个已下线 Master 负责处理所有的哈希槽。
5. 所有其他节点都会根据新的 Master 对进行相应的**配置更新**：
   1. 所有被新的 Master 接管的槽会被更新。
   2. 已下线 Master 的所有 Slave 会察觉到 `PROMOTED` 标志， 并开始对新的 Master 进行复制。
   3. 如果已下线的 Master 重新回到上线状态， 那么它会察觉到 `PROMOTED` 标志， 并将自身调整为现任 Master 的 Slave。

###### 集群状态检测

1. 每当集群发生配置变化时（可能是哈希槽被更新，也可能是某个节点进入了 `FAIL` 状态），集群中的每个节点都会对它们自己所知道的节点进行扫描。
2. 配置扫描处理完毕后， 集群会进入以下两种状态的其中一种：

   - `OK` ： 集群可以正常工作，负责处理全部 `16384` 个槽的节点中， 没有一个节点被标记为 `FAIL` 状态。
   - `FAIL` ： 集群不能正常工作，当集群中有某个节点（该节点的所有主从）进入 `FAIL` 状态时， 集群不能处理任何命令请求， 对于每个命令请求， 集群节点都返回错误回复。
     - 说明即使集群中只有**一部分**哈希槽不能正常使用， 整个集群也会停止处理任何命令。
     - 不过节点从出现问题到被标记为 `FAIL` 状态的这段时间里， 集群仍然会**正常运作**， 所以集群在某些时候， 仍然有可能只能处理针对 `16384` 个槽的其中一个子集的命令请求。
3. 以下是集群进入 `FAIL` 状态的两种情况：
   - **出现至少有一个 哈希槽不可用时**：负责处理某个哈希槽的节点（该节点的所有主从）进入了 `FAIL` 状态。
   - **集群中的大部分 Master 进入了 `PFAIL` 状态时**：此时，集群可以在不请求大部分 Master 的意见下，快速地将某个节点判断为 `FAIL` 状态， 然后集群也会进入 `FAIL` 状态，从而让整个集群停止处理命令请求。

##### 数据弱一致性

Redis 集群**不保证数据的强一致性**，在特定条件下， Redis 集群可能会丢失已经被执行过的写命令，**原因如下**：

- **异步复制**：Master 对命令的复制工作发生在返回客户端命令回复之后， 因为如果每次处理命令请求都需要等待复制操作完成的话， 那么 Master 处理命令请求的速度将极大地降低，因此 Redis 在性能和一致性之间做出了权衡，在复制工作上采取了异步复制。
  1. 客户端向 Master B 发送一条写命令。
  2. Master B 执行写命令，并向客户端返回命令回复。
  3. Master B 将刚刚执行的写命令复制给它的从节点 B1 、 B2 和 B3。
  4. 如果写命令复制过程中，Master B 发生宕机的话，则会丢失这些被执行过的写命令。
- **集群内出现网络分区**：一个客户端与至少包括一个 Master 在内的少数实例被孤立。
  1. 举个例子，假设集群包含 A 、 B 、 C 、 A1 、 B1 、 C1 六个节点，其中 A 、B 、C 为 Master，而 A1 、B1 、C1 分别为三个 Master 的 Slave，另外还有一个客户端 Z1 。
  2. 假设集群中发生网络分区，那么集群可能会分裂为两方，大多数的一方包含节点 A 、C 、A1 、B1 和 C1，而少数的一方则包含节点 B 和客户端 Z1 。
  3. 在这网络分区期间，主节点 B 仍然会接受 Z1 发送的写命令。
  4. 如果网络分区出现的时间很短，那么集群是可以继续正常运行的。
  5. 如果网络分区出现的时间足够长，使得大多数一方将 Slave B1 设置为新的 Master，那么当网络分区恢复时，原来的 Master B 就会被大多数一方的 Master B1 替代，成为 B1 的 Slave，导致原先 Z1 发送给 Master B 的写命令发生丢失。
     - 不过，在网络分区期间， 客户端 Z1 可以向 Master B 发送写命令的最大时间会被 `cluster-node-timeout` 配置的节点超时时限所限制：
       1. 对于大多数一方来说， 如果一个 Master 未能在 `cluster-node-timeout` 内重新联系上集群， 那么集群会将这个 Master 视为下线， 并使用它的 Slave 来代替自己继续工作。
       2. 对于少数一方来说， 如果一个 Master 未能在 `cluster-node-timeout` 内重新联系上集群， 那么它将停止处理写命令， 并向客户端**报告错误**。

##### 优点

- **高可用**：当集群中的一部分节点失效或者无法进行通讯时， 集群仍然可以继续处理命令请求的能力。
- **高性能**：操作某个 key 时，不会先找到节点再处理，而是直接重定向到目标 Redis 实例，相较代理分片少了 proxy 的连接损耗。
- **高拓展**：不存在中心节点或者代理节点，同时最大支持线性拓展 1000 个节点，把新节点加入集群后，可以通过命令平均分配已有节点的哈希槽。

##### 缺点

- **只可使用普通单机 Redis 所有功能的一个子集**：不支持同时使用多个键的 Redis 命令，不支持多数据库功能， 只能使用默认的 `0` 号数据库。
- **数据弱一致性**：与其他高可用方案一样，Redis 集群也**不保证数据的强一致性**，在特定条件下， Redis 集群可能会丢失已经被执行过的写命令。
- **占用带宽**：虽然避免了 Master 单节点的问题，但集群内的数据同步、节点通信会占用一定的带宽。

##### 适用场景

只有在**写操作比较多**的情况下，集群模式才更有优势，相对于其他大多数情况，使用**哨兵模式**都能满足需求。

#### 高可用架构总结

##### AKF 拆分原则

![1632542719978](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1632542719978.png)

**AKF扩展立方体**，Scalability Cube，是在《The Art of Scalability》一书中被首次提出，旨在提供一个系统化的扩展思路。AKF 把系统扩展分为以下三个维度：

1. **X 轴**：代表无差别的克隆服务和数据，工作可以很均匀的分散在不同的服务实例上。
2. **Y 轴**：关注应用中职责的划分，比如数据类型、交易执行类型的划分。
3. **Z 轴**：关注服务和数据的优先级划分，如分地域划分。

=> X、Y、Z 轴的扩展并不是孤立的，可以同时对这 3 个维度进行扩展系统，分布式系统非常复杂，而 AKF 提供了一种自上而下的方法论，能够让我们针对不同场景下的性能瓶颈，以最低的成本去提升系统性能。

###### X 轴扩展 | 应用负载均衡

![1632542998175](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1632542998175.png)

- **特点**：
  1. 在应用层做 X 轴扩展，需要处理业务的应用进程属于**无状态服务**，用户数据全部放在了关系数据库中。
  2. 此时可以在应用进程前加 1 个负载均衡服务，通过部署更多的应用进程，来获得更大的系统容量。
- **优点**：
  - 成本最低，实施简单：在搭建好负载均衡后，只需要在新的物理机、虚拟机或者微服务上复制程序，就可以让新进程分担请求流量，而且不会影响事务的处理。
  - 解决了**应用程序的单点故障**，提高了系统的可用性。
- **缺点**：只能扩展无状态服务，当有状态的数据库出现性能瓶颈时，应用层的 X 轴扩展就无能为力了。
  1. 当请求用户频率越来越高，这时可以把单实例数据库扩展为主备多实例，在数据库层沿 X 轴**读写分离**提升性能。
  2. 当业务持续发展，数据库的 CPU、网络带宽、内存、磁盘 IO 等某个指标率先达到上限后，系统的吞吐量就达到了瓶颈，这时可以在数据层库层沿 Y 轴`垂直分表` 提升性能。
  3. 当用户数据量持续增长，关系数据库中的表就会达到百万、千万行数据，SQL 语句会越来越慢，这时可以沿着 Z 轴**分库分表**提升性能。
- **适用场景**：发展初期，业务复杂度低，需要增大系统容量时。

###### X 轴扩展 | 读写分离

![1632544601407](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1632544601407.png)

- **特点**：
  - 把数据库按应用程序的读写操作拆分后，用复制的方式把数据库单机架构扩展为主从架构，让主库支持读写两种 SQL，而从库只支持读 SQL。
  - 如果读库性能达到了瓶颈，还可以继续沿着 X 轴扩展多个从库，提升读 SQL 的性能。
- **优点**：
  - 实现简单，读写分离成本由中间件承担。
  - 实现了**数据库的故障转移**，进一步提高了系统的可用性。
- **缺点**：应用中编写代码的成本有所增加，且主从复制存在数据一致性问题。
- **适用场景**：读频率远大于写频率，影响到系统性能时。

###### Y 轴扩展 | 垂直分表

![1632544928615](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1632544928615.png)

- **特点**：当业务持续发展，数据库的 CPU、网络带宽、内存、磁盘 IO 等某个指标率先达到上限后，系统的吞吐量就达到了瓶颈，拆分系统功能，使得各组件的职责、分工更细，从而提升系统的效率。
- **优点**：
  - 每个后端的子应用更加聚焦于细分的功能，使得数据库规模会变小，更容易优化性能。
  - 实现了**数据库的故障隔离**，进一步提高了系统的可用性。
- **缺点**：拆分功能需要重构应用代码，成本对比较沿 X 轴的复制扩展要高得多。
- **适用场景**：业务复杂，代码耦合度高，非数据量导致的单库压力大时。

###### Z 轴扩展 | 分库分表

![1632547686194](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1632547686194.png)

- **背景**：当用户数量上亿后，无论怎样基于 Y 轴的功能去垂直拆分表字段，都无法使得关系数据库单个表的行数在千万级以下，这样表字段的 B 树索引非常庞大，难以完全放在内存中，最后大量的磁盘 IO 操作会拖慢 SQL 语句的执行。
- **特点**：沿 Z 轴拆分关系数据库，进行**分库分表**操作。
  - 比如已经含有上亿行数据的 User 用户信息表，可以分成 10 个库，每个库再分成 10 张表，利用固定的哈希函数，就可以把每个用户的数据映射到某个库的某张表中。
  - 这样，单张表的数据量就可以降低到 1 百万行左右，如果每个库部署在不同的服务器上，它们处理的数据量减少了很多，却可以独占服务器的硬件资源，性能自然就有了提升。

- **优点**：
  - 是关系数据库中解决数据增长压力的最有效办法，可以降低数据持续增长的压力，提升系统性能。
  - 还可以从 `用户维度` 拆分系统，基于用户的地理位置获得额外收益。
    - 比如充分利用 IDC 与用户间的网速差，选择不同速度的 IDC，为不同的用户，提供不同性能的服务。
    - 再如将不同的用户分组，免费用户组与付费用户组，在业务上分离用户群体后，然后有针对性地为不同组的用户提供不同水准的服务。
- **缺点**：
  - 导致跨表的查询语句复杂许多，而跨库的事务几乎难以实现，应用程序代码编码成本高。
    - **解决方案**：使用某些厂商提供相应的中间件层，可以降低 Z 轴扩展的代价，同时，分片采用按照 ER 规则进行分片。
  - 当系统需要扩容时，一旦路由规则发生变化，会带来很大的**数据迁移成本**。
    - **解决方案**：使用一致性 hash 算法可以较好的避免这个问题。
  - 跨分片的事务一致性难以保证，当更新内容同时分布在不同库中，不可避免地带来跨库事务问题。
    - **解决方案**：分布式事务和最终一致性。
- **场景**：单表数据过大，影响系统性能时。

##### Redis AKF 拆分

![1632542544732](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1632542544732.png)

###### X 轴扩展 | 主从复制

- **特点**：
  1. 按照主从设计，Master 负责读写， Slave 负责读。
  2. 再结合哨兵集群，在 Master 故障时，使用 Slave 进行切换，从而实现高可用。
- **优点**：
  1. 主从，解决了读并发压力大的问题。
  2. 哨兵，解决了单点故障问题。
- **缺点**：单机容量会有限制，并且会出现写并发压力大的问题。

###### Y 轴扩展 | 业务拆分

- **特点**：
  - 把 Redis 所有键，按照业务进行拆分，拆分到不同的 Redis 实例上。
  - 可以在 Y 轴的基础上，再进行 X 轴的主从复制的扩展，形成不同业务的 Redis 集群。
- **优点**：从分离不同业务数据的角度，暂时解决了单机容量受限，以及写并发压力大的问题。
- **缺点**：当某个业务的集群达到一定规模后，如果数据量过大，仍然会出现单机容量受限，以及写并发压力大的问题。

###### Z 轴扩展 | 数据分区

- **特点**：
  - 将全部 Key 数据根据分区规则分成多个子集，并存储到 Redis 实例中。
  - 可以在 Z 轴的基础上，叠加 X 轴的主从复制，集群内进行数据分片，比如 Redis Cluster。
  - 可以再叠加 Y 轴的业务拆分，把整个 Redis 系统划分成多个不同业务的、数据分片过的 Redis Cluster。
- **优点**：增加了整个集群的算力、带宽和内存，从根本上解决了单机容量受限，以及写并发压力大的问题。
- **缺点**：
  - 数据分区后，不支持跨实例的命令与事务。
  - 备份管理要复杂得多。
  - 扩缩容时可能需要对数据再平衡。

### 2.2. Redis经典问题？

#### 缓存击穿

- **概念**：缓存中某一个热点数据，在某一时刻**失效**了，导致大量**并发**请求压垮数据库，就像被击穿了一样。

  - 说白了就是，某个数据在数据库中有，但是在缓存中没有。

- **解决方案**：

  1. **保证缓存一直存在**：以缓存**失效**作为切入点，通过保证缓存一直存在，避免或者减少击穿的发生。

     - **优点**：由于缓存长时间存在，且只在更新时才需要加互斥锁，整体吞吐量高。
     - **缺点**：缓存长期存在内存中，Redis 需要设置内存淘汰策略，淘汰非热点数据。

     - **实现方式**：
       - **不设置过期时间**：设置 key 永远不过期，在修改数据库时，同时更新缓存。
       - **定时任务更新**：后台起一个定时任务，每隔一段时间，在 key 快要失效时，提前将 key 刷新为最新数据。
       - **获取前更新**：每次获取前，检查 key 剩余过期时间，如果发现快过期了，则更新该 key。
       - **Redis 分级缓存**：缓存两份 Redis 数据，第 1 份数据用于被请求命中，第 2 份数据作为备份，生存时间设置得较第 1 份的长一点，如果第 2 份数据被命令则说明第 1 份数据已过期，此时要去查询数据库，更新这两级缓存。

  2. **保证数据库安全**：以**并发**请求数据库作为切入点，虽然保证了缓存一直存在，但在 Redis 发生内存淘汰时，还是可能发生击穿的，此时还需要保证数据库的安全。实现方式有：

     - **分布式锁**：只有获取到互斥锁的线程才能访问数据库，然后在释放锁前重新设置缓存；而其他获取锁失败的线程在唤醒后，不再访问数据库，而是查询缓存。
       - **优点**：可以在缓存更新时，实现互斥访问数据库，保证不被压垮。
       - **缺点**：
         1. 分布式锁本身也存在缺点。
         2. 需要搭配缓存一直存在机制使用，保证吞吐量。
         3. 如果只使用互斥锁来防止击穿，在获取锁失败时，快速返回默认值，也可以保证吞吐量。
     - **分级缓存**：Ehcache、Guava 本地缓存 + Hystrix限流、降级、熔断，避免 MySQL 被打死。
       - **优点**：有多级缓存和限流熔断机制来兜底。
       - **缺点**：增加了系统架构复杂度；本地缓存没有分布式一致性可言。

  3. **缓存预热**：见缓存预热。

#### 缓存雪崩

- **概念**：指大面积的 key 同时**失效**，导致大量**并发**请求压垮数据库。
  - 同样也是，某个数据在数据库中有，但是在缓存中没有。
- **缓存击穿与缓存雪崩**：
  1. 缓存击穿指的是，压垮数据库的所有并发请求都是只查同一条数据。
  2. 而缓存雪崩指的是，大量 key 同时失效，压垮数据库的所有并发请求，等于这些失效 key 的并发请求之和。
  3. 因此，可以把缓存雪崩中这些失效 key，拆成一个一个 key，从而把缓存雪崩看做成一个一个缓存击穿的集合。
- **解决方案**：
  1. **分散过期时间**：以缓存**失效**作为切入点，通过把 key 的过期时间设置成随机的，防止同一时间大量 key 同时失效的发生。
     - **优点**：简单有效。
     - **缺点**：不适合实时性要求比较高的业务，比如游戏的每日两点更新、财报记录等，它们都是需要在一个固定的时间，保证数据刷新，并且不允许出现旧数据。
  2. **分散并发请求**：同缓存击穿。

#### 缓存穿透

- **概念**：指由于请求了数据库也不存在的数据，导致缓存一直没有数据，大量并发绕过了缓存，从而压垮了数据库。
- **缓存击穿与缓存穿透**：
  1. 缓存击穿时，数据库里有请求所需数的据。
  2. 缓存穿透时，数据库里并不存在请求所需的数据。
- **解决方案**：
  1. **使用空 key**：对于这种在缓存获取不到，在数据库也获取不到的数据，可以把 key-null 设置到缓存中，这样下次访问时就可以走上缓存了。
     - **优点**：可以用于防止系统被恶意攻击。
     - **缺点**：只适用于数据能为空的 key；如果空 key 数量多且不重复时，则会造成很多无用 key 的存在。
  2. **使用布隆过滤器**：将所有可能存在的数据哈希到一个足够大的 bitmap 中，一个一定不存在的数据会被这个 bitmap 拦截掉。
     - **优点**：运行快速、内存占用小。
     - **缺点**：
       1. 对于元素的存在结果有误判，并且随着系统的不断运行，误判率会越来越高，此时可以**定期重建**布隆过滤器。
       2. 元素一旦存进布隆过滤器中，删除则十分困难，因为可能会删掉其他元素的 bit 结果，此时可以使用额外的删除标记变量进行**逻辑删除**。
  3. **后端接口增加校验**：比如用户鉴权校验、ID 基础校验等（ID <= 0的请求则直接拦截掉）。
     - **优点**：属于网关请求过滤，能过滤掉恶意请求。

#### 缓存预热

- **概念**：指在请求被缓存前，提早把相关的数据加载到缓存系统中，避免**初次击穿**问题的发生。
- **实现方式**：
  - **启动时加载**：在系统上线时，把相关可预期（比如排行榜）等热点数据，直接加载到缓存中。
  - **页面手动刷新**：也可以写一个缓存刷新页面，手动操作热点数据上下线（比如广告推广）。
- **优点**：保证了系统一开始就有缓存存在，能够在一定程度上避免**初次击穿**的发生。
- **缺点**：只能保证提前加载能够确定的热点数据，对于运行时的热点数据，还是需要靠以上两种方案。

#### 缓存降级

- **概念**：当访问量剧增、服务出现问题（如响应时间慢、甚至不响应），导致非核心服务影响到了核心流程的性能时，可以根据一些关键数据进行**自动降级**，也可以配置开关实现**人工降级**，从而保证核心服务可用性，即使是可能损害非核心服务。
- **实现方式**：比如基于日志级别实现的方案：
  1. **一般级别**：比如有些服务偶尔因为网络抖动或者服务正在上线而超时，可以**自动降级**。
  2. **警告级别**：比如有些服务在一段时间内，成功率有所波动（比如在95~100%之间），可以**自动降级或人工降级**，并发送**告警**。
  3. **错误级别**：比如可用率低于90%，或者数据库连接池被打爆了，或者访问量突然猛增到系统能承受的最大阈值时，可以根据情况**自动降级或者人工降级**。
  4. **严重错误级别**：比如因为特殊原因数据错误了，此时需要**紧急人工降级**。
- **优点**：可以防止因非核心业务导致的 Redis 服务故障，从而保证核心服务的高可用，因此，对于**不重要的**缓存数据，可以采取缓存降级的策略。
- **缺点**：在进行降级之前，需要要对系统业务进行梳理，哪些可降级的，哪些是不可降级的（比如核心业务购物车、结算等）。

#### 缓存一致性

##### 一致性保证

- **概念**：在缓存机器的带宽被打满，或者机房网络出现波动时，**缓存更新失败**，新数据没有写入缓存，就会导致**缓存和DB**数据的不一致。
- **解决方案**：
  - **最终一致性保证**：
    1. Cache 更新失败后，可以进行重试，则将重试失败的 key 写入mq。
    2. 待缓存访问恢复后，将这些 key 从缓存**删除**。
    3. 然后 key 在再次被查询时，会重新从 DB 加载，从而保证数据的一致性。
    4. 另外，缓存 key 的过期时间可以适当地调短，让缓存数据及早过期，然后从 DB 重新加载，确保数据不一致的情况不会持续很长时间。
  - **强一致性保证**：读请求和写请求串行化，串到一个内存队列里去。
    - **缺点**：串行化之后，就会导致系统的吞吐量会大幅度的下降，正常情况下，需要多几倍的机器去支撑线上的一个请求。

##### 缓存模式

###### Cache-Aside

![1632829929514](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1632829929514.png)

- **概念**：Cache-Aside，旁路缓存，缓存系统不和数据库直接进行交互，而是由应用程序同时和缓存以及数据库打交道，可能是项目中最常见的一种模式，是一种控制逻辑都实现在应用程序中的模式。
- **原理**：
  - **读数据时**：
    1. 应用程序需要判断缓存中是否已经存在数据。
    2. 当缓存中已经存在数据，也就是缓存命中时，则直接从缓存中返回数据。
    3. 当缓存中不存在数据，也就是缓存未命中时，则先从数据库里读取数据，并且存入缓存，然后返回数据。
  - **写数据时**：
    - **策略一**：先更新数据库，再更新缓存。
      - **缺点**：存在线程安全问题，当线程 A 先于 B 写入数据库，却后于 B 才更新缓存时，会导致数据库和缓存的数据不一致。
      - **解决方案**：加锁保证写入数据库和更新缓存操作的原子性。
    - **策略二**：先更新数据库，再删除缓存中的数据，下次在读取缓存时，才从数据库中重新加载。
      - **缺点**：数据最终一致性，在重新加载缓存时，数据库刚好被更新，此时就读到了旧的缓存，相当于发生了**脏读**。
      - **适合场景**：业务允许暂时不一致的场景。
  - **优点**：和 Write-Through 模式相比，避免了任何数据都被写入缓存，导致**缓存频繁更新**的问题。
  - **缺点**：
    - 当发生缓存未命中时，则从请求到成功获取缓存的过程会比较慢，因为需要经过三个步骤，先查询缓存，再数据库读取，最后写入缓存。
    - 复杂的逻辑都在应用程序中，如果实现微服务，多个微服务中会有这些重复的逻辑代码。
  - **使用场景**：适用于**不支持 Read-Through/Write-Through** 的缓存系统。

###### Read-Through/Write-Through

- **概念**：这种模式中，应用程序将**缓存**作为主要的数据源，而数据库对于应用程序是透明的，更新数据库和从数据库的读取的任务，都交给缓存来代理了，因此对于应用程序来说，实现时可以简单很多。

- **原理**：

  - **Read-Through**：

    1. 缓存配置了一个**读模块**，它知道如何将数据库中的数据写入缓存。
    2. 在数据被请求的时候，如果未命中，则将数据从数据库载入缓存。

    ![1632831602956](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1632831602956.png)

  - **Write-Through**：

    1. 缓存配置了一个**写模块**，它知道如何将数据写入数据库。
    2. 当应用要写入数据时，缓存会先存储数据，并调用写模块将数据写入数据库。

    ![1632831718553](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1632831718553.png)

- **优点**：

  - 缓存不存在脏数据，相较于Cache aside而言更适合**缓存一致**的场景。
  - 屏蔽了底层数据库的操作，使得只需要操作缓存，应用程序逻辑相对简单。

- **缺点**：写多读少时，Write-Through 会非常浪费性能，因为数据可能更改了很多次，却没有被读取，白白地每次都写入缓存，造成写入延迟。

- **使用场景**：适用于**写入之后经常被读取**的应用。

###### Write-Behind

![1632832224155](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1632832224155.png)

- **概念**：又叫 Write-Back，和 Write-Through 写入的时机不同，Write-Back 将缓存作为可靠的数据源，每次都只写入缓存，而写入数据库则采用异步的方式，比如当数据要被移除出缓存时再存储到数据库，或者在一段时间后批量更新数据库。
- **优点**：
  - 写入和读取数据都非常的快，因为都是从缓存中直接读取和写入。
  - 对于数据库不可用的情况有一定的容忍度，即使数据库暂时不可用，系统也整体可用，在数据库恢复后，再将数据写入数据库。
- **缺点**：有数据丢失的风险，如果缓存挂掉而数据没有及时写到数据库时，则缓存中有些数据会永久的丢失。
- **使用场景**：读写效率都非常好，由于是异步存储到数据库，提升了写的效率，适用于**读写密集**的应用。
  - 比如微博对一些计数业务，一条 Feed 会被点赞 1万 次，如果每次都更新 DB，前后需要更新 1万 次 DB， 代价很大；但如果每次只更新缓存，再合并成一次请求 DB 直接加 1万，则是一个非常轻量的操作。

###### Write-Around

![1632832607368](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1632832607368.png)

- **概念**：和 Write-Through 不同，与 Write-Behind 相反，Write Aroud 更新时只写入数据库，不写入缓存。
- **优点**：写效率高，对比 Write-Through，如果数据写入后很少被读取，缓存也不会被没用到的数据占满。
- **缺点**：如果数据被写入多次，则可能导致缓存和数据库的数据不一致。
  - 可以结合 Read-Through 或者 Cache-Aside读 一起使用，只在缓存未命中的情况下写缓存，保证数据的最终一致性。
- **使用场景**：适合于**只写入一次而很少被读取**的应用。

#### Hot Key 问题

- **概念**：指在某个时刻，大量并发请求访问同一个 Key，导致 Redis 服务器压力过大甚至压垮。

  - 明星结婚、离婚、出轨这种特殊突发事件，比如奥运、春节这些重大活动或节日，还比如秒杀、双12、618 等线上促销活动，都很容易出现 Hot Key 的情况。

- **如何提前发现 Hot Key**？提前找到 Hot Key，可以先进行缓存预热，避免**初次击穿**问题的发生。

  1. **提前评估**：对于重要节假日、线上促销活动这些提前已知的事情，可以**提前评估**出可能的 Hot key 来。
  2. **实时分析**：而对于突发事件，无法提前评估，可以通过 Spark，对应流任务进行**实时分析**，及时发现新发布的 Hot Key。而对于之前已发出的事情，逐步发酵成为 Hot Key 的，则可以通过 Hadoop 对批处理任务离线计算，找出最近历史数据中的高频 Hot Key。

- **解决方案**：

  1. **分散 Hot Key**：这 n 个 key 分散存在多个缓存节点，然后 Client 端请求时，随机访问其中某个后缀的 Hot Key，这样就可以把 Hot Key 的请求打散，避免一个缓存节点过载。
  2. **限制逃逸流量**：和缓存击穿一样，通过加锁或者分散的方式，来限制逃逸流量，使得只有一个请求进行数据回源，并刷新本地缓存与 Redis 缓存，其他请求则等待后直接从缓存中的读取。
  3. **水平+垂直扩容**：缓存集群可以单节点进行主从复制和业务拆分（垂直分表）。
  4. **应用本地缓存**：利用应用内的本地缓存，但是需注意需要设置上限。
  5. **定时刷新**：延迟不敏感，定时刷新，实时感知用主动刷新。

  => 无论如何设计，最后都要写一个**兜底逻辑**，千万级流量说来就来。

#### Big Key 问题

- **概念**：指 key 存放的 value 非常大，由于 Redis 是单线程运行的，如果一次操作的 value 很大，则会对 Redis

  服务器长时间进入网络 I/O 拥塞，响应时间增大。

  - 比如互联网系统中需要保存用户最新1万个粉丝的业务，常常会出现 Big Key，因为一个用户个人信息缓存，包括基本资料、关系图谱计数、发 feed 统计等。
  - 再如，微博的 feed 内容缓存也很容易出现 Big Key，一般用户微博在 140 字以内，但很多用户也会发表 1千字甚至更长的微博内容，这些长微博也就成了 Big Key。

- **解决方案**：将 value 要存的大对象，分拆为多个小对象，然后通过 `multiGet` 来获取，这样可以把单次操作的压力，**平摊**到多个 Redis 实例中，降低对单个 Redis 的 I/O 影响。

### 2.3. Redis调优？

Redis 调优，可以根据缓存架构设计过程中常见的考量点入手：

https://blog.csdn.net/hualaoshuan/article/details/102638188

![1632893948796](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1632893948796.png)

#### 缓存读写方式

- **考量内容**：value 读写方式，是选择全部整体读写，还是只部分读写及变更。

  - 比如，用户粉丝数，很多普通用户的粉丝有几千到几万，而大 V 的粉丝更是高达几千万甚至过亿，因此，获取粉丝列表肯定不能采用整体读写的方式，**只能部分获取**。
  - 再如，在判断某用户是否关注了另外一个用户时，不需要拉取该用户的全部关注列表，可以直接在关注列表上进行**检查判断**，然后返回 True / False 或 0 / 1 的方式更为高效。

- **调优原则**：

  1. 避免使用 `keys`、`smembers` ，这些命令操作执行期间是阻塞其他客户端的，如果数据量大时，其阻塞的时间是不可接受的。

     - **解决方案**：生产上，可以使用增量式迭代命令 `scan`、`sscan`、`hscan`、`zscan` 命令来替代 `keys` 和 `smembers`，但增量迭代过程中 key 有可能会被修改，所以只能对被返回的元素提供有限的保证。

     | 命令  | 使用                                      | 作用                                                         |
     | ----- | ----------------------------------------- | ------------------------------------------------------------ |
     | SCAN  | SCAN cursor [MATCH pattern] [COUNT count] | 用于迭代当前数据库中的所有 key，返回的每个元素都是一个数据库键。cursor 使用 0，表示开始一次新的迭代，否则代表迭代时返回的游标，直到返回 0，则代表一次完整遍历；count 用于告知迭代命令需要返回的元素个数，默认为 10；pattern 与 `keys` 命令一样。 |
     | SSCAN | 用法同 `SCAN`                             | 用于迭代集合键中的元素，返回的每个元素都是一个集合成员。     |
     | HSCAN | 用法同 `SCAN`                             | 用于迭代哈希键中的键值对，返回的每个元素都是一个键值对，一个键值对由一个键和一个值组成。 |
     | ZSCAN | 用法同 `SCAN`                             | 用于迭代有序集合中的元素，返回的每个元素都是一个有序集合元素，一个有序集合元素由一个成员 member 和一个 score 组成。 |

  2. 批量添加多条数据时尽量选择 `pipline` ，减少发起多次客户端连接。

#### 缓存 KV Size

- **考量内容**：缓存键值对 key-value 所占用的内存。
- **调优原则**：
  1. 对于 key，尽量使用短的 key，有时可以不用刻意追求"见名知意"的目标，而导致拉大了 key 的长度。
  2. 对于 value，尽量也保持精简，比如性别可以用 0、1 表示。
  3. 对象实体可以尽量使用 `hash` 存储，因为如果使用 `string` 存储，每当要修改其中一项时，就需要把整个对象取回，效率低下，而使用 `hash` 可以很好地解决这个问题。
  4. 某些业务场景可以考虑使用 `bitmap` 来减少不必要的内存使用。

#### 缓存 Key 数量

1. 如果 key 数量不大，可以在缓存中存下**全量数据**，把缓存当 DB 存储来用，缓存读取 miss 时，则表明数据不存在，根本不需要再去 DB 查询。
2. 如果 key 数据巨大，则在缓存中尽可能只保留频繁访问的**热数据**，对于**冷数据**直接访问 DB。
3. 选择合适的内存回收策略，让 Redis 内存被填满了之后，尝试回收一部分key。

#### 缓存读写峰值

1. 对缓存数据的读写峰值，如果**小于 10 万**级别，简单分拆到独立 Redis 即可。
2. 而一旦数据的读写峰值**超过 10 万**甚至到达 100 万级的QPS，则可以同时使用 本地缓存 + Redis，甚至 Redis 缓存内部继续分层。

#### 缓存命中率

缓存的命中率，对整个服务体系的性能影响非常大，对于核心高并发访问的业务，需要预留**足够的容量**，减少内存淘汰的发生，确保核心业务缓存维持较高的命中率。

- 比如，微博中的 Feed Vector Cache，常年的命中率高达 99.5% 以上。为了持续保持缓存的命中率，缓存体系需要持续监控，及时进行故障处理或故障转移。

#### 缓存过期策略

1. 可以设置较短的过期时间，让冷 key 自动过期。
2. 也可以让 key 带上时间戳，同时设置较长的过期时间，比如 key_20191019，过了这个时间就可以把缓存删掉。

#### 缓存穿透时间

对于一些缓存穿透后，**加载时间特别长**或者需要**复杂计算**的数据，而且**访问量还比较大**的业务数据，要配置**更多容量**，维持更高的命中率，从而减少穿透到 DB 的概率，来确保整个系统的访问性能。

#### 缓存可运维性

需要考虑 Redis 的集群管理，如何进行一键扩缩容，如何进行 Redis 的升级和变更，如何快速发现并定位问题，如何持续监控报警，最好有一个完善的运维平台，将各种运维工具进行集成。

#### 缓存安全性

1. 一方面可以限制来源 IP，只允许内网访问。
2. 另一方面，对于一些**关键性指令**，需要增加**访问权限**，避免被攻击或误操作时，导致重大后果。

