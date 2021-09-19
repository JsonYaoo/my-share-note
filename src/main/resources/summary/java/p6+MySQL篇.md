# 四、MySQL篇

### 1.1. 为什么使用数据库？

使用数据库保存数据，**既能保证数据永久保存，又能兼顾查询效率**。

| 数据保存位置 | 内存中           | 文件中                                                      | 数据库中                                                     |
| ------------ | ---------------- | ----------------------------------------------------------- | ------------------------------------------------------------ |
| 优点         | 存取速度快       | 数据永久保存                                                | 1）数据永久保存；2）使用SQL语句，查询方便效率高；3）管理数据方便 |
| 缺点         | 数据不能永久保存 | 1）速度比内存操作慢；2）需要频繁的IO操作；3）查询数据不方便 | -                                                            |

### 1.2. 数据库OLTP与OLAP的区别？

数据处理大致可以分成两大类：OLTP和OLAP。

- **OLTP**：
  - Online Transaction Processing，**联机事务处理系统**，表示事务性非常高的系统，一般都是高可用的在线系统，以小的事务以及小的查询为主，评估其系统的时候，一般看其每秒执行的Transaction以及Execute SQL的数量。
  - 在OLTP系统中，单个数据库每秒处理的Transaction往往超过几百个，或者是几千个，Select 语句的执行量每秒几千甚至几万个，典型系统有电子商务系统、银行、证券等。
  - OLTP 系统，是一个数据块变化非常频繁、SQL 语句提交非常频繁的系统，因此出现瓶颈的地方就是CPU与磁盘子系统。
    - 对于数据块来说，应尽可能让数据块保存在内存当中；对于SQL来说，尽可能使用变量绑定技术来达到SQL 重用，减少物理I/O 和重复的SQL 解析，从而极大的改善数据库的性能。 
- **OLAP**：
  - Online Analytical Processing，**联机分析处理系统**，也叫DSS决策支持系统，就是大家所说的数据仓库。
  - 在OLAP系统中，语句的执行量不是考核标准，因为一条语句的执行时间可能会非常长，读取的数据也非常多，所以，考核的标准往往是磁盘子系统的吞吐量（带宽），如能达到多少MB/s的流量。
    - 磁盘子系统的吞吐量往往取决于磁盘的个数，应尽量采用个数比较多的磁盘以及比较大的带宽，如4Gb的光纤接口。

|          | OLTP                                 | OLAP                             |
| -------- | ------------------------------------ | -------------------------------- |
| 定位     | 基本日常的事务处理                   | 复杂分析、决策支持               |
| 强调指标 | 内存命中率、SQL变量绑定、SQL并发执行 | SQL执行时长、磁盘I/O、带宽吞吐量 |
| 数据     | 当前的、最新的、细节的               | 历史的、聚集的、统一的           |
| 读写方式 | 每次读写数十条记录                   | 每次读上百万条记录               |
| 工作单位 | 简单的事务                           | 复杂的查询                       |
| DB大小   | 100MB-GB                             | 100GB-TB                         |

### 1.3. 数据库产品对比？

| 产品           | 优点                                                        | 缺点                                             | 备份                                      | 高可用                                           | 适用场景                                   |
| -------------- | ----------------------------------------------------------- | ------------------------------------------------ | ----------------------------------------- | ------------------------------------------------ | ------------------------------------------ |
| 关系型数据库   |                                                             |                                                  |                                           |                                                  |                                            |
| MySQL          | 支持事务、开源免费                                          | 数据量大时，需要进行水平拆分                     | mysqldump逻辑备份，xtrabackup工具物理备份 | 主从复制、读写分离、分片集群                     | 中小型LAMP                                 |
| Oracle         | 支持事务、支持大字段、性能最高                              | 管理维护麻烦、价格昂贵                           | exp逻辑备份，RMAN工具物理备份             | 双机热备、Oracle Dataguard主从、Oracle RAC集群   | 大部分事业单位                             |
| SQL Server     | 方便易用                                                    | 只能运行在windows平台、性能不够稳定              | -                                         | -                                                | windows平台OLTP                            |
| PostgreSQL     | 支持事务、稳定性强、性能高                                  | 扩容麻烦、MVCC并发版本需定期清理                 | COPY命令逻辑备份，pgdump物理备份          | 主从复制、Slony-I第三方组件做数据同步、集群有bug | 地理位置信息处理                           |
| SQLite         | 自给自足、无需服务器、开源免费                              | 只能本地嵌入，无法远程访问                       | -                                         | -                                                | 嵌入式设备、本地应用程序                   |
| DB2            | 并行性高、适合海量数据                                      | -                                                | -                                         | -                                                | 数据仓库、数据挖掘                         |
| 非关系型数据库 |                                                             |                                                  |                                           |                                                  |                                            |
| Redis          | 单线程、支持K-V以及多种数据结构存储、支持持久化，适合热数据 | 容量受内存限制，不便于海量数据读写，不适合冷数据 | RDB、AOF                                  | 主从、哨兵、集群                                 | 缓存、最新回复、点赞数、共同好友、排行榜   |
| MemCache       | 多线程、性能高、速度快，用于减轻数据库负载                  | 不支持持久化、只能存储K-V数据                    | 不支持                                    | 集群没有同步复制机制                             | 前端缓存、用户信息、好友信息、文章信息     |
| MongoDB        | 文档结存存储、海量数据性能优越                              | 不支持事务、占用空间大、无法关联查询             | mongoexport逻辑备份、mongodump物理备份    | 主从复制、1.6 ReplicaSets复制集故障时自动切换    | Json文件、日志分析、敏捷开发、地理位置信息 |

### 1.4. MySQL与Oracle的使用区别？

#### MySQL

- MySQL是一个**关系型数据库管理系统**，由瑞典MySQL AB 公司开发，属于 Oracle 旗下产品，是最流行的关系型数据库管理系统之一，在 WEB 应用方面，MySQL是最好的关系数据库管理系统之一。
  - **关系数据库**：指将数据保存在不同的表中，而不是将所有数据放在一个大仓库内，这样就增加了速度并提高了灵活性。

- MySQL所使用的SQL语言，是用于访问数据库的最常用标准化语言，采用了双授权政策，分为社区版和商业版，由于其**体积小、速度快、总体拥有成本低**，尤其是**开放源码**这一特点，一般中小型网站的开发都选择 MySQL 作为网站数据库。
  - **SQL**：Structured Query Language，结构化查询语言，是一种数据库查询语言，用于**存取数据、查询、更新和管理**关系数据库系统。

#### Oracle

Oracle Database，又名Oracle RDBMS，简称Oracle，是甲骨文公司的一款关系数据库管理系统，系统可移植性好、使用方便、功能强，适用于各类大、中、小微机环境，是一种**高效率的、可靠性好的、适应高吞吐量**的数据库方案。

| 区别         | Oracle                                                       | MySQL                                                        |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 开源免费与否 | 收费                                                         | **开源免费**                                                 |
| 超长文本处理 | 使用CLOB类型                                                 | 使用text和longtext类型                                       |
| 日期字段查询 | select * from tb1 where dt>to_date('2020-09-13 12:15:01', 'yyyy-MM-dd hi24:mi:ss'); | select * from tb1 where dt>'2020-09-13 12:15:01';            |
| 分页实现     | select * from (select t.*,rownum num from tb1 t where rownum<=100 ) where num>50; | select username from tb1 limit 50, 100;                      |
| group by     | 不允许返回group by外的其他字段                               | 可以任意返回一个字段值                                       |
| 修改字段类型 | 空字段直接改，不允许更改字段名称，改类型必须保证正确         | alter table tb1 change column f1_old f1_new int(11) comment 'xxx'; |
| 表字段注释   | 只能在创建字段之后指定                                       | 可以在建表时指定，也可以在建表完成后修改                     |
| 字段移位     | 建表后不能移位                                               | 建表之后可以修改字段顺序                                     |
| 创建索引     | 只能在建表完成后添加                                         | 可以在建表时添加，也可以在建表完成后添加                     |
| 查询建表语句 | select  dbms_metadata.get_ddl(table, 'tb1') from dual;       | show create table tb1;                                       |
| 查询执行计划 | explain plan for select + select * from table(dbms_xplan.display()); | explain select                                               |

### 1.5. MySQL数据类型？

- **整数类型**：TINYINT、SMALLINT、MEDIUMINT、INT、BIGINT，分别表示1字节、2字节、3字节、4字节、8字节整数。
  - 任何整数类型都可以加上UNSIGNED属性，表示数据是无符号的，即非负整数。
  - 整数类型可以被指定长度，但在大多数场景是没有意义的，不会限制值的合法范围，只会影响显示字符的个数，而且需要和UNSIGNED ZEROFILL属性配合使用才有意义。
    - 比如，如果用户插入的数据为12的话，那么数据库实际存储数据为00012。
- **实数类型**：FLOAT、DOUBLE、DECIMAL。
  - **DECIMAL**：可以用于存储比BIGINT还大的整型，能存储精确的小数。
  - **FLOAT和DOUBLE**：有取值范围，并支持使用标准的浮点进行近似计算。
    -  计算时，FLOAT和DOUBLE相比DECIMAL效率更高一些，DECIMAL可以理解成是用字符串进行处理。
- **字符串类型**：VARCHAR、CHAR、TEXT、BLOB。
  - **尽量避免使用TEXT/BLOB类型**，查询时会使用临时表，导致严重的性能开销。
  - 使用时要注意只分配需要的空间，更长的列排序时会消耗更多内存。
  - **varchar与char**：对于经常变更的数据来说，CHAR比VARCHAR更好，因为CHAR**不容易产生碎片**；对于非常短的列，CHAR比VARCHAR在**存储空间上更有效率**。
    - **varchar**：使用额外1或2个字节存储字符串长度。列长度小于255字节时，使用1字节表示，否则使用2字节表示，当存储的内容超出设置的长度时，内容会被截断。
      - 用于存储可变长字符串，它比定长类型更节省空间。
    - **char**：字段定长，根据定义的字符串长度分配足够的空间，会根据需要使用空格进行填充，当存储的内容超出设置的长度时，内容会被截断。
      - 适合存储很短的字符串，或者所有值都接近同一个长度。
- **日期类型**：timestamp、datetime。
  - **尽量使用timestamp**，空间效率高于datetime。
  - 如果需要存储微妙，可以使用bigint存储。

### 1.6. MySQL存储引擎？

存储引擎，Storage engine，是**MySQL的一套文件系统实现**，使用各种不同的技术将数据存储在文件或内存中，通过使用不同的存储机制、索引技巧、锁定水平，来提供不同的能力，以使得能够获得额外的速度或者功能，从而改善应用的整体功能。

- **Innodb**：提供对事务的支持，还提供了行、表锁以及外键的约束，索引与数据同一文件。
  - 适合更新和删除操作频率高，要保证数据的完整性的、并发量高的场景，比如OA自动化办公系统。如果没有特别的需求，使用默认的InnoDB即可。
- **MyIASM**：不提供事务的支持，不支持行锁和外键，索引与数据在不同的文件。
  - 适合以读写插入为主的应用程序，比如博客系统、新闻门户网站。
- **MEMORY**：所有的数据都在内存中，数据的处理速度快，但是安全性不高。

### 1.7. MySQL锁分类？

当数据库发生**并发事务**时，需要**锁机制**来保证访问的次序，以满足事务的**隔离性**，最终保证**数据的一致性**。

#### 悲观锁和乐观锁

从**事务进入临界区前是否锁住同步资源的角度**来分：

- **悲观锁**：先锁再用。
  - 就是悲观思想，事务每次进入临界区操作数据的时候都认为别的事务会修改，所以事务每次在读写数据时都会**上锁**，锁住同步资源，这样其他事务需要读写这个数据时就会**阻塞**，一直等到拿到锁。
  - 适用于**写多读少**的场景，遇到**高并发写时性能高**，比如MySQL的**排他锁**。
- **乐观锁**：用时检查。
  - 是一种乐观思想，事务每次去拿数据的时候都认为别的事务不会修改，所以**不会上锁**，但是在更新的时候会判断一下在此期间别人有没有去更新这个数据，采取在写时先读出当前版本号，然后加锁操作（比较跟上一次的版本号，如果一样就更新），如果失败就要重复**读-比较-写**的操作。
  - 适用于**读多写少**的场景，遇到**高并发写时性能低**，MySQL的乐观锁可以通过**Version+SQL判断**实现。

#### 排他锁和共享锁

按照**锁定资源的方式**来分：

- **共享锁**：Shared lock，简称S锁，允许持有锁读取行的事务，即读锁，加锁时将自己和子节点全加S锁，父节点直到表头全加IS锁。
  - **兼容性**：读写互斥，加了S锁的记录，允许其他事务再加S锁，不允许其他事务再加X锁。
  - **加锁方式**：select … lock in share mode。
- **排他锁**：Exclusive lock，简称X锁，允许持有锁修改行的事务，即写锁， 加锁时将自己和子节点全加X锁，父节点直到表头全加IX锁 。
  - **兼容性**：写读互斥，写写互斥，加了X锁的记录，不允许其他事务再加S锁或者X锁。
  - **加锁方式**：select … for update。

#### 表锁和行锁

按照**锁定资源的粒度**来分：

##### 表锁

Mysql中锁定粒度最大的一种锁，对当前操作的整张表加锁，实现简单 ，资源消耗也比较少，加锁快，不会出现死锁 。其锁定粒度最大，触发锁冲突的概率最高，并发度最低，**MyISAM和 InnoDB引擎都支持表级锁**。

###### 意向锁

Intention Locks，表明某个事务正在某些行持有了锁，或该事务准备去持有锁，其存在是为了**协调行锁和表锁**的关系，支持多粒度的锁（表锁与行锁）并存。

- **意向共享锁**：Intention shared lock，IS，事务有意向对表中的某些行加S锁，在请求S锁前，要先获得IS锁。
- **意向排他锁**：Intention exclusive lock，IX，事务有意向对表中的某些行加X锁，在请求X锁前，要先获得IX锁。
- **意向锁举例**：事务A修改user表的记录r，会给记录r上一把行级的排他锁（X），同时会给user表上一把意向排他锁（IX），这时事务B要给user表上一个表级的排他锁就会被阻塞，因此意向锁通过这种**锁叠加的方式**实现了行锁和表锁共存，并且满足了事务隔离性的要求。
- **为什么意向锁是表级锁**：当需要加一个排他锁时，需要根据意向锁去判断表中**有没有数据行被锁定**，此时如果意向锁是行锁，则需要遍历每一行数据去确认，性能低下；而如果意向锁是表锁，则只需要**判断一次**即可知道有没数据行被锁定，**性能较高**。
- **意向锁兼容性**：指当事务A对某个数据范围（行或表）上了“某锁”后，另一个事务B是否能在这个数据范围上“某锁”，可见，意向锁之间、读锁与意向锁之间互相兼容，写锁与其他锁之间都不兼容。

| 互斥性       | 共享锁（S） | 排它锁（X） | 意向共享锁IS | 意向排他锁IX |
| ------------ | ----------- | ----------- | ------------ | ------------ |
| 共享锁（S）  | ✅           | ❌           | ✅            | ❌            |
| 排它锁（X）  | ❌           | ❌           | ❌            | ❌            |
| 意向共享锁IS | ✅           | ❌           | ✅            | ✅            |
| 意向排他锁IX | ❌           | ❌           | ✅            | ✅            |

###### 自增锁

AUTO-INC Locks，是一种特殊的表级锁，发生涉及**AUTO_INCREMENT**列的事务性插入操作时产生。

##### 行锁

Mysql中锁定粒度最小的一种锁，只针对当前操作的行进行加锁，能大大减少数据库操作的冲突，其加锁粒度最小，并发度高，但加锁的开销也最大，加锁慢，会出现死锁。**InnoDB默认使用行锁**，按**锁算法**的角度来分，InnoDB支持的行锁包括如下几种：

###### 记录锁

Record Lock，对**索引项**加锁，锁定**符合条件的行**，其他事务不能修改和删除加锁项。

- 如果该表上没有任何索引，那么InnoDB会在后台创建一个隐藏的聚蔟主键索引，那么锁住的就是这个隐藏的**聚蔟主键索引**。

###### 间隙锁

Gap Lock，对索引项之间的间隙加锁，锁定**记录的范围，不包含索引项本身**，其他事务不能在锁范围内插入数据。

- 比如在 1、2、3中，间隙锁的可能值有 (∞, 1)，(1, 2)，(2, ∞)。
- 可用于**防止幻读**，保证索引间的不会被插入数据。

###### 临建锁

Next-key Lock，锁定索引项本身和索引范围，**左开右闭区间**，即Record Lock+Gap Lock的结合，可解决幻读问题。

- 默认情况下，InnoDB select … for update使用Next-Key Lock来锁定记录。但当查询的索引含有唯一属性的时候，Next-Key Lock会进行优化，**降级为Record Lock**，即仅锁住索引本身，不是范围。
- 另外，Next-Key Lock在不同的场景中会退化：

| 场景                                        | 退化后的锁类型                   |
| ------------------------------------------- | -------------------------------- |
| 使用unique index精确匹配（=），且记录存在   | Record Lock                      |
| 使用unique index精确匹配（=），但记录不存在 | Gap Lock                         |
| 使用unique index范围匹配（< 或 >）          | Record Lock + Gap Lock，左开右闭 |

##### 页锁

MySQL中锁定粒度介于行级锁和表级锁中间的一种锁，表级锁速度快，但冲突多；行级冲突少，但速度慢，因此取了折衷的页级，**一次锁定相邻的一组记录**，其开销和加锁时间界于表锁和行锁之间、会出现死锁、锁定粒度界于表锁和行锁之间、并发度一般。

##### 锁的选择

比如执行，update test set name=“hello” where name=“world”，精确匹配时：

1. 如果更新条件没有走索引，则会进行全表扫描，扫表时会阻止其他任何的更新操作，**上升为表锁**。
2. 如果更新条件为索引字段，但是为非唯一索引，则会**使用Next-Key Lock**，保证在符合条件的记录上加上排他锁，锁定当前非唯一索引以及其对应的主键索引的值，同时，还要保证锁定的区间不能插入新的数据。
3. 如果更新条件为唯一索引，则会使用**Record Lock**。

#### MySQL死锁排查

死锁是指两个或多个事务在同一资源上相互占用，并请求锁定对方的资源，从而导致恶性循环的现象。

常见的解决死锁的方法

1. InnoDB目前处理死锁的方法是，将持有最少行级排他锁事务进行**回滚**，这是相对比较简单的死锁回滚算法。
2. **相同的访问顺序**，如果不同程序会并发存取多个表，尽量约定以相同的顺序访问表，可以大大降低死锁机会。
3. 在同一个事务中，尽可能做到**一次锁定所需要的所有资源**，减少死锁产生概率；
4. 对于非常容易产生死锁的业务部分，可以尝试使用升级锁定颗粒度，通过**表级锁**定来减少死锁产生的概率；

```mysql
show engine innodb status\G
-- TABLE LOCK：表锁
-- lock mode IX：意向锁
-- RECORD LOCK：行锁
-- index GEN_CLUST_INDEX：聚蔟索引，InnoDB加锁是给索引加的锁
-- lock_mode X：写锁，间隙锁
-- lock_mode X locks rec but not gap：行锁，非间隙锁
-- lock_mode X locks rec but not gap waiting：行锁等待
-- LATEST DETECTED DEADLOCK：最近探测到的死锁，排查到发生死锁
```

### 1.8. MVCC多版本并发控制？

#### 概念

MVCC，Multi-Version Concurrency Control，即**多版本并发控制**，实现对数据库的并发访问，实现读写冲突时的无锁并发控制。

- 可以在并发读写数据库时，做到在**读操作时不用阻塞写操作，写操作也不用阻塞读操作**，提高了数据库并发读写的性能。
- 同时还可以解决**脏读、不可重复读、幻读**等事务隔离问题，但不能解决更新丢失问题。
- MVCC手段只适用于Msyql **RC读已提交和 RR可重复读** 的事务隔离级别，RU读未提交由于存在脏读，即能读到未提交事务的数据行，所以不适用MVCC。

#### 当前读和快照读

- **当前读**：非MVCC实现，读取的是记录的最新版本，读取时要保证其他并发事务不能修改当前记录，会对读取的记录进行**加锁**。当前读就是悲观锁的具体功能实现
  - **举例**：select lock in share mode（共享锁）, select for update（拍他锁），update、insert、delete（排他锁）、串行化事务隔离级别。

- **快照读**：MySQL实现MVCC理想模型的中一个具体的**非阻塞读功能**，避免了加锁的操作，可以降低开销、提高并发性能，但快照读可能读到的不一定是数据的最新版本，而可能是之前的**历史版本**。
  - **前提**：快照读的前提是，事务使用**非串行化**的隔离级别，因为串行化级别下的快照读会**退化成当前读**。
  - **举例**：不加锁的select。

#### MySQL实现MVCC

![1630673126935](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1630673126935.png)

##### 版本链

InnoDB中，每次修改版本都会在版本链中记录，通过**undo log + roll_pointer**实现：

- **trx_id**：当前版本的事务ID，用来存储的每次对某条记录进行修改的时候的**事务ID**。当每个事务开启时，都会被分配一个ID，而这个ID是递增的，因此越新的事务其ID值越大。
- **roll_pointer**：回滚指针，由于每次对记录修改时，都会把老版本写入到undo日志中，使用回滚指针来指向这条记录**上一个版本**的位置，通过它来获得上一个版本的记录信息。
  - 注意，插入操作的undo日志没有roll_pointer，因为它没有老版本。

##### undo log

undo log是用于记录旧版本的链表，链首为最新的旧记录，链尾为最早的旧记录，主要分为两种：

- **insert undo log**：
  - 代表事务在insert新记录时产生的undo log，只在事务回滚时需要，并且在**事务提交后可以被立即丢弃**。
- **update undo log**：
  - 对MVCC其实质性的帮助，事务在进行update或delete时产生的undo log，不仅在事务回滚时需要，**在快照读时也需要**，所以不能随便删除。
  - 只有在快速读或事务回滚不涉及该日志时，对应的日志才会被purge线程（InnoDB专门的记录清理线程）统一清除。

##### Read View

![1630674963257](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1630674963257.png)

Read View，是事务进行快照读操作时产生的**读视图**，是数据库当前的一个快照，记录了系统当前**活跃事务ID**，用于做**可见性判断**。

- **活跃事务ID**：即还没有commit的事务ID。

- **可见性判断**：当某个事务执行快照读时，会对该记录创建一个Read View读视图， 并把它作为条件，来**判断当前事务能够看到哪个版本的数据**，其中可能是当前版本的数据，也有可能是该行记录的undo log里面的某个版本的老数据。

  1. **trx_id == creator_trx_id**：可以访问这个版本，这个版本的事务ID等于当前事务ID（创建Read View的事务ID），即**自己能够读到自己的版本**。
     - creator_trx_id用来标记当前生成Read View的事务，使得能够让该事务读取到自己未提交的版本。
  2. **trx_id < min_trx_id**：可以访问这个版本，这个版本的事务ID小于最小活跃事务ID，说明这个版本**已经提交过了**，对于当前做可见性判断的事务来说，**是可以看见的**。
     - min_trx_id用来标记最早未提交的事务A，对比版本记录中的事务B，从而判断出B是否已经提交。
  3. **trx_id > max_trx_id**：不可以访问这个版本，这个版本的事务ID大于下一个事务ID，说明这个版本记录是在该Read View生成之后产生的，已经超出了版本链范围，而快照读只能读取版本链中的版本，因此该版本对于当前做可见性判断的事务来说，**是不应该看见的**。
     - max_trx_id用来标记下一个将要生成的事务A，对比版本记录中的事务B，从而判断出B是否超出版本链范围。
  4. **min_trx_id <= trx_id <= max_trx_id**：
     - 如果这个版本的事务ID为m_ids中的某个值，则不可以访问这个版本的，因为m_ids都是活跃的、还没提交的事务，说明该版本记录还没有提交，对于当前做可见性判断的事务来说，**是不应该看见的**。
     - 如果这个版本的事务ID不为m_ids中的某个值，则可以访问这个版本，因为没在m_ids里，又小于等于max_trx_id，说明该版本记录已提交了，对于当前做可见性判断的事务来说，**是可以看见的**。

  => 因此，如果可见性判断到要读取的版本记录为**自己创建的或者已经提交的**，则说明对于当前做可见性判断的事务来说，**该版本记录是可以看见的**。

### 1.9. 事务四大特性ACID？

事务，Transaction，是指**访问、更新数据库数据一个程序执行单元**，是逻辑上的一组操作，要么都执行，要么都不执行，其执行的结果必须使数据库从一种一致性状态变到另一种一致性状态。

> ACID 则是衡量事务的四个特性，按照严格的 SQL 标准，只有同时满足 ACID 特性的才算是事务，但在各大数据库厂商的实现中，真正能严格满足 ACID 事务的少之又少。
>
> 比如 MySQL NDB Cluster 事务不满足持久性和隔离性，InnoDB RR 不满足严格的隔离性，Oracle RC 也不满足严格的隔离性等等。
>
> 所以，与其说 ACID 是事务必须满足的条件，不如说它们是衡量事务的四个维度而已。

- **原⼦性**： Atomicity， 事务是最小的执行单位，不允许分割，整个事务中所有的操作，要么全部提交成功，要么全部失败回滚，强调的是**事务操作原子不可分割**。
  - **undo log**：回滚日志，属于逻辑日志，记录的是 sql 执行相关的信息，是 MySQL 中事务**原子性**和**隔离性**实现的基础。
  - **实现原理**：在 MySQL InnoDB 中，当事务对数据库进行修改时，InnoDB 会生成对应的 undo log。如果事务执行失败或调用了 rollback，导致事务需要回滚时，就可以根据 undo log 的内容做与之前**相反**的工作，把数据回滚到修改之前的样子，以实现事务原子性操作。
    1. 对于每个 insert，回滚时会执行 delete。
    2. 对于每个 delete，回滚时会执行 insert。
    3. 对于每个 update，回滚时会执行一个相反的 update，把数据改回去。

- **一致性**：Consistency， 指事务执行结束后，数据库的完整性约束没有被破坏，都是合法的数据状态，强调的是**数据状态事务前后的一致**。

  - **数据库的完整性约束包括但不限于**：
    - **实体完整性**：比如，行的主键存在且唯一。
    - **列完整性**：比如，字段的类型、大小、长度都要符合要求。
    - **外键约束**：比如，主键所在的表是主表，外键所在的表是从表。
    - **用户自定义完整性**：比如，转账前后，两个账户余额的和应该保持不变。
  - **实现原理**：可以说，**一致性是事务追求的最终目标**，原子性、持久性和隔离性都是为了保证数据库状态的一致性。其中，实现一致性的措施包括：
    - **数据库层面的保障**：原子性、持久性和隔离性的保证，以及一些其他约束，比如不允许向整型列插入字符串值，或者字符串长度不能超过列的限制等等。
    - **应用层面的保障**：比如如果转账操作只扣除转账者的余额，而没有增加接收者的余额，无论数据库实现的多么完美，也无法保证状态的一致。

- **隔离性**： Isolation，并发访问数据库时，⼀个⽤户的事务不被其他事务所⼲扰，各并发事务之间数据库是独⽴的，一个事务所做的修改在最终提交之前，对其他事务是不可见的，强调的是**事务间的数据操作互不影响**。

  - **实现原理**：
    - **写写操作**：使用**锁机制**来保证隔离性。
    - **写读操作**：
      - **快照读**：使用**MVCC**来保证隔离性。
      - **当前读**：使用**锁机制**保证隔离性。

- **持久性**：Durability， ⼀个事务被提交之后，它对数据库中数据的改变是持久的，即使数据库发⽣故障，也不应该对其有任何影响，强调的是**事务后的数据会永久保存**。

  - **Buffer Pool**：为减少每次读写数据的磁盘I/O，InnoDB 提供了 Buffer Pool 缓存，其中包含了磁盘中部分数据页的映射，作为访问数据库的缓冲。当从数据库读取数据时，InnoDB 会先从 Buffer Pool 中读取，如果 Buffer Pool 中没有，则从磁盘读取后放入 Buffer Pool；当向数据库写入数据时，同样 InnoDB 也会先写入 Buffer Pool，Buffer Pool 中修改的数据会定期刷新到磁盘中，这一过程称为**刷脏**。

    - **优点**：大大提高了读写数据的效率。
    - **缺点**：如果 MySQL 宕机，而此时 Buffer Pool 中修改的数据还没有刷新到磁盘，就会导致数据的丢失，**无法保证事务的持久性**。

  - **实现原理**：InnoDB 引入 redo log 来解决 Buffer Pool 的问题， 以保证事务的持久性。

    1. 当数据修改时，修改 Buffer Pool 中的数据前，会先在把这次操作**写入** redo log 缓冲区。

    2. 写入 redo log缓冲区后，根据innodb_flush_log_at_trx_commit属性来决定**刷盘机制**：

       - **0**：表示当事务提交时，不会把 redo log 缓冲区的日志**同步**到磁盘 redo log 文件中，而是等待线程每秒的刷新，不能完全保证全部写入成功。

       - **1**：表示当事务提交时，把 redo log 缓存区的日志**同步**到磁盘 redo log 文件中，且会保证全部写入成功。
       - **2**：表示当事务提交时，异步把 redo log 缓存区的日志**同步**到磁盘 redo log 文件中，不能完全保证全部写入成功。

    3. 如果 MySQL 宕机，重启时可以**读取** redo log 中的数据，对数据库进行**恢复**。

  - **redo log**：重做日志，属于物理日志，用于保证事务的持久性。

    - **WAL**：Write-ahead logging，预写式日志，redo log 采用 WAL ，所有修改先**写入**日志，再更新到 Buffer Pool，保证了数据不会因 MySQL 宕机而丢失，从而满足了持久性要求。

    - **写入 redo log 性能高于 Buffer Pool 刷脏**：因此，写入 redo log 是可以用在 Buffer Pool 刷脏之前来保证事务持久性的。

      | 写入 redo log 缓冲区                 | Buffer Pool刷脏                                              |
      | ------------------------------------ | ------------------------------------------------------------ |
      | 是追加操作，属于顺序 I/O             | 每次修改的数据位置都是随机的，属于随机 I/O                   |
      | 只需要修改的部分，大大减少了无效 I/O | 以数据页 Page 为单位（MySQL默认为16 KB），一个 Page上只出现一个小修改，需要将整页写入磁盘，存在大量的无效 I/O |

    - **redo log vs binlog**：

      | 不同点   | redo log                         | bin log                      |
      | -------- | -------------------------------- | ---------------------------- |
      | 日志名称 | 重做日志                         | 二进制日志                   |
      | 日志作用 | 用于崩溃恢复，保证事务的持久性   | 用于时间点恢复，保证主从复制 |
      | 存储内容 | 属于物理日志，内容基于磁盘数据页 | 逻辑日志，内容为一条条SQL    |
      | 实现层面 | InnoDB存储引擎实现               | MySQL服务器层实现            |
      | 写入时机 | 事务提交时、每秒一次             | 事务提交时写入               |

### 2.0. 事务并发问题？

- **脏读**：Drity Read，指两个事务并发执行，事务A已更新某一份数据，事务B读取同一份数据，出于某种原因，事务A回滚了更新的操作，导致事务B读取的数据不正确。
  - **产生原因**：事务A读取了事务B中**未提交的数据**。
  - **特点**：违背了隔离性。
- **不可重复读**：Non-repeatable read，指两个事务并发执行，事务A前后两次查询的结果不一样（**行内容发生了变更**）。
  - **产生原因**：事务A前后两次查询有间隔，期间内，被事务B**修改并提交了事务**。
  - **特点**：相比脏读的区别是，不可重复读是读取另一事务提交的数据。这种现象是正常的，是由于事务的隔离级造成的，但是在在某些特别的情况下也是不允许的。 
- **幻读**：Phantom Read，指两个事务并发执行，事务A前后两次查询的结果不一样（**出现了幻行，导致行数量发生了变更**）。
  - **产生原因**：事务A前后两次查询有间隔，期间内，被事务B**新增数据并提交了事务**，比如事务A查询了几行数据，而事务B并发插入了新的几行数据，事务A在接下来的查询中，会发现有几行数据是它先前所没有的。
  - **特点**：和不可重复读一样，都是读取了另外一个事务的数据，不同的是不可重复读查询的是同一条数据，而幻读则是针对批量的数据，或者说**不可重复读是A读取了B的更新数据，幻读是A读取了B的新增数据**。

### 2.1. 事务隔离级别？

| 事务隔离级别 | 存在的事务并发问题     |
| ------------ | ---------------------- |
| 读未提交     | 脏读、幻读、不可重复读 |
| 读已提交     | 幻读、不可重复读       |
| 可重复读     | 幻读                   |
| 串行化       | 没有事务并发问题       |

- **读未提交**：READ UNCOMMITTED，是最低的事务隔离级别，事务可以读取到其他事务未提交的数据，可能会导致脏读、不可重复读和幻读。

  - **当前读**：读取数据**不需要加共享锁**，这样就不会与修改的数据上的排他锁冲突了。

- **读已提交**：RC，READ COMMITTED，也叫不可重复读，事务从开始直到提交之前，所做的任何修改对其他事务都是不可见的，是大多数据库的默认事务隔离级别（**比如Oracle**），可以阻⽌脏读，但还是可能会导致不可重复读和幻读。

  - **当前读**：读操作**需要加共享锁**，但是**在语句执行完以后释放共享锁**。
  - **快照读**：MVCC
    - **Read View以select为单位**，每次select都会生成一个Read View，事务根据这个Read View做可见性判断，只读取那些期间自己创建的（未提交）以及已经提交了的版本记录，即只读别人提交过的，不能读别人未提交的，做到了读已提交的隔离性。
    - **不可重复读的原因**：由于Read View以select为单位，每次select都会生成一个Read View，两者的可见性判断的结果可能不一样，能看到的版本也就不一样，从而可能导致事务前后两次的**查询别人版本的结果**不一致，产生不可重复读。

- **可重复读**：RR，REPEATABLE READ，事务对同⼀字段的**多次读取的结果都是⼀致的**，除⾮数据是被事务本身所修改，是**MySQL的默认事务隔离级别**，可以阻⽌脏读和不可重复读，但还是可能会导致幻读。

  - **当前读**：读操作**需要加共享锁**，但是**在事务提交之前并不释放共享锁**，也就是必须等待事务执行完毕以后，才释放共享锁。
    - **解决幻读的方式**：
      - **使用间隙锁**：MySQL默认开启间隙锁，因此在MySQL中RR可重复读隔离级别下，是没有事务并发问题的。
      - **使用MVCC快照读**：快照读基于Read View来实现，每个事务对应一个Read View，解决了幻读的问题。
      - **升级到串行化隔离级别**：把隔离级别设置成SERIALIZABLE，但这样所有事务都只能顺序执行，自然不会因为并发有什么影响了，但是性能会下降许多，实际中很少用到。
  - **快照读**：MVCC
    - **Read View以事务为单位**，每个事务只会生成一个Read View，事务根据这个Read View做可见性判断，只读取那些期间自己创建的（未提交）以及已经提交了的版本记录，即只读别人提交过的，不能读别人未提交的，做到了读已提交的隔离性。
    - **解决不可重复读、幻读的问题**：由于Read View以事务为单位，每个事务只会生成一个Read View，事务前后的快照读只对应同一份Read View，可见性判断一致，能看到的版本一致，因此整个事务过程中**查询别人版本的结果**一致，避免了不可重复读、幻读的发生，因此在MySQL中RR可重复读隔离级别下，是没有事务并发问题的。

  |        | RC                                   | RR                         |
  | ------ | ------------------------------------ | -------------------------- |
  | 实现   | 多条查询语句会创建多个不同的ReadView | 仅需要一个版本的ReadView   |
  | 粒度   | 语句级读一致性                       | 事务级读一致性             |
  | 准确性 | 每次语句执行时间点的数据             | 第一条语句执行时间点的数据 |

- **串行化**：SERIALIZABLE，是最高的隔离级别，通过强制事务串行执行，所有的事务依次逐个执⾏，事务之间完全不存在互相⼲扰，解决了幻读的问题，即阻止了所有的事务并发问题，包括脏读、不可重复读和幻读。

  - **当前读**：**锁定整个范围的键**，并一直持有锁，直到事务完成，可能导致大量的超时和锁争用的问题，实际中很少使用。

### 2.2. MySQL慢SQL与慢查询日志？

#### 慢SQL

- **危害**：
  - **从数据库角度看**：
    - 每个SQL执行都需要消耗一定I/O资源，SQL执行的快慢，决定资源被占用时间的长短。
    - 假设总资源是100，如果有一条慢SQL占用了30的资源共计1分钟，那么在这1分钟时间内，其他SQL能够分配的资源总量就是70，如此循环，当资源分配完的时候，会**导致所有新的执行SQL进行排队等待**。
  - **从应用的角度看**：SQL执行时间长，意味着应用需要等待，导致用户的体验较差。
- **治理原则**：
  - **优先治理写库的慢SQL**：目前数据库基本上都是读写分离架构，读在从库上执行，写在主库上执行，而由于从库的数据都是从主库上复制过去的，如果主库等待较多，则会**加大与从库的复制时延**。
  - **执行次数多的慢SQL优先治理**：如果有一类SQL高并发集中访问某一张表，应当优先治理。

#### 慢查询日志

慢查询日志，是MySQL内置的一项功能，可以记录执行超过指定时间的SQL语句，即**记录慢SQL**。

- **参数设置方式**：

  - **修改my.cof配置文件**：

    ```sh
    [mysqld]
    # ...
    log_output = 'FILE,TABLE';
    show_query_log = ON
    
    # 表示0.001s，即默认值为1ms，表示任何SQL都会记录起来，对于实际业务没任何意义
    long_query_time = 0.001
    ```

  - **set global设置全局变量**：

    ```sql
    set global log_output = 'FILE,TABLE';
    set global slow_query_log = 'ON';
    
    -- 表示0.001s，即默认值为1ms，表示任何SQL都会记录起来，对于实际业务没任何意义
    set global long_query_time = 0.001;
    ```

- **慢查询日志分析**：

  - **log_output = TABLE**：select * from `mysql`.slow_log。
  - **log_output = FILE**：使用show variables like '%slow_query_log_file%'获取日志路径，使用mysqldumpslow分析日志。

  ![1630805035402](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1630805035402.png)

  ```mysql
  -- mysqldumpslow：
  -- -s：排序方式，默认为at
  --		1) al：平均时间
  --		2）ar：平均返回记录
  --		3）at：平均查询时间
  --		4） c：访问计数
  --		5） l：锁定时间
  --		6） r：返回记录
  --		7） t：查询时间
  -- -r：将-s的排序倒序
  -- -t：top n的意思，展示最前面的几条
  -- -g：正则匹配，只有符合正则的行才会展示
  -- -a：展示原始SQL
  
  -- eg: 得到返回记录集中最多的10条SQL
  mysqldumpslow -s -r -t 10 /var/lib/mysql/894503c23e0-slow.log
  
  -- eg: 得到按照查询时间排序，并且带有left join的10条SQL
  mysqldumpslow -s t -t 10 -g "left join" /var/lib/mysql/894503c23e0-slow.log
  ```

| 慢查询日志参数                         | 作用                                                         | 默认值    |
| -------------------------------------- | ------------------------------------------------------------ | --------- |
| log_output                             | 日志输出的类型，可以设置多种格式，比如FILE，TABLE；默认为FILE，表示文件；设置成TABLE，则将日志记录到mysql.slow_log中； | FILE      |
| long_query_time                        | 执行时间超过指定阈值，才记录到慢查询日志，单位为秒，可使用小数表示小于秒的时间 | 10        |
| log_queries_not_using_indexs           | 是否要将未使用索引的SQL，都记录到慢查询日志中，此配置会无视long_query_time的配置，生产配置建议关闭，开发环境建议开启 | OFF       |
| log_throttle_queries_not_using_indexes | 和log_queries_not_using_indexs配置使用，如果log_queries_not_using_indexs打开，则该参数将限制每分钟写入的、未使用索引的SQL数量 | 0         |
| min_examined_row_limit                 | 扫描行数至少达到指定阈值，才记录到慢查询日志                 | 0         |
| log_show_admin_statements              | 是否要记录管理语句，默认关闭，管理语句包括ALTER、ANALYZE、CHECK、CREATE、DROP、OPTIMIZE、REPAIR | OFF       |
| slow_query_log_file                    | 指定慢查询日志的文件路径                                     | /var/路径 |
| log_slow_slave_statements              | 该参数在从库上设置，决定是否记录在复制过程中超过long_query_time的SQL，如果binlog格式是row，则该参数无效 | OFF       |
| log_show_extra                         | 当log_output=FILE时，是否要记录额外信息（>= MySQL 8.0.14开始提供），对log_output=TABLE的结果无影响 | OFF       |

### 2.3. 详细介绍MySQL EXPLAIN？

- 使用EXPLAIN关键字可以**模拟优化器执行SQL语句**，分析查询语句或是结构的性能瓶颈。

- 在select语句之前增加explain关键字，MySQL会在查询上设置一个标记，执行查询会返回执行计划的信息，而不是执行SQL。

#### 总结

基于MySQL 8.0编写，理论上支持MySQL 5.0以及更高版本。

| Explain结果字段   | json名称      | 含义                         |
| ----------------- | ------------- | ---------------------------- |
| id                | select_id     | 该语句的唯一标识             |
| select_type       | 无            | 查询类型                     |
| table             | table_name    | 表名                         |
| partitions        | partitions    | 匹配的分区                   |
| **type**          | access_tpye   | 联接类型                     |
| **possible_keys** | possible_keys | 可能的索引选择               |
| **key**           | key           | 实际选择的索引               |
| **key_len**       | key_length    | 索引的长度                   |
| ref               | ref           | 索引的哪一列被引用了         |
| **rows**          | rows          | 估计要扫描的行               |
| **filtered**      | filtered      | 表示符合查询条件的数据百分比 |
| **Extra**         | 没有          | 附件信息                     |

#### id

该语句的唯一标识：

- 如果结果包含多个id值，则**数字越大，越先执行**。
- 而相同id的，则**从上往下依次执行**。

#### select type

查询类型，有以下几种取值：

| 查询类型             | 作用                                                         |
| -------------------- | ------------------------------------------------------------ |
| SIMPLE               | 简单查询（未使用UNION或者子查询）                            |
| PRIMARY              | 最外层的查询                                                 |
| UNION                | 在UNION中的第二个和随后的SELECT被标记为UNION，如果UNION被FROM子句中的子查询包含，则它的第一个SELECT会被标记为DERIVED（派生表）。 |
| DEPENDENT UNION      | UNION中的第二个或后面的查询，依赖了外面的查询                |
| UNION RESULT         | UNION的结果                                                  |
| SUBQUERY             | 子查询中的第一个SELECT                                       |
| DEPENDENT SUBQUERY   | 子查询中的第一个SELECT，依赖了外面的查询                     |
| DERIVED              | 用来表示包含在FROM子句的子查询中的SELECT，MySQL会递归执行并将结果放入到一个临时表中，内部称其为Derived table（派生表），因为该临时表是从子查询派生出来的 |
| DEPENDEDNT DERIVED   | 派生表，依赖了其他的表                                       |
| MATERIALIZED         | 物化子查询                                                   |
| UNCACHEABLE SUBQUERY | 子查询，结果无法缓存，必须针对外部查询的每一行重新评估       |
| UNCACHEABLE UNION    | UNION属于UNCACHEABLE SUBQUERY的第二个或后面的查询            |

#### table

表示当前这一行正在访问哪张表，如果SQL定义了表名，则展示表的别名。

#### partitions

当前查询匹配记录的分区，对于未分区的表，返回null。

#### type

**重点**，连接类型，有如下几种取值，性能从好到坏排序：

| 连接类型        | 含义                                                         | 备注                                                         |
| --------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| system          | 该表只有一行，相当于系统表                                   | system是const的特例                                          |
| const           | 针对主键或者唯一索引的等值查询，最多返回一行数据             | 查询速度非常快                                               |
| eq_ref          | 当使用了索引的全部组成部分，并且索引是**主键或者非空唯一索引**才会发生 | 性能仅次于system和const                                      |
| ref             | 当满足索引的最左前缀规则，或者索引**不是主键也不是唯一索引**时才会发生 | 如果索引只匹配到少量的行，则性能也是不错的                   |
| fulltext        | 全文索引                                                     | 使用MyISAM存储引擎才有                                       |
| ref_or_null     | 该类型类似于ref，但MySQL会额外搜索哪些行包含NULL，           | SELECT * FROM ref_table WHERE key_col = expr OR key_col IS NULL； |
| index_merge     | 该类型表示使用了索引合并优化，表示一个查询里面用到了多个索引 | -                                                            |
| unique_subquery | 该类型和eq_ref类型，但使用了**IN查询**，且**子查询是主键或者唯一索引** | value IN (SLECT id FROM single_talbe WHERE expr)             |
| index_subquery  | 和unique_subquery类型，只是**子查询使用的是非唯一索引**      | value IN (SELECT key_col FROM single_table WHERE other_expr) |
| range           | 范围扫描，表示检索了指定范围的行，主要用于有限制的索引扫描   | 常见的有，BETWEEN、>、>=、<、<=、IS NULL、<=>、LIKE、IN      |
| index           | 全索引扫描，和ALL类型，只不过index是全盘扫描了索引的数据     | 当查询仅使用索引中的一部分列时，会使用该类型，有两种触发场景：1）覆盖索引，比ALL快，此时只扫描索引数，Extra列为Using Index；2）全表扫描，同ALL，此时会回表查询数据，Extra列不会出现Using Index； |
| ALL             | 全表扫描                                                     | 性能最差                                                     |

#### possible_keys

展示当前查询可以使用哪些索引，由于这一列的数据是在SQL优化过程早期创建的，因此，有些索引可能对于SQL后续优化过程是没用到的。

#### key

表示MySQL实际选择的索引。

#### key_len

索引使用的字节数，由于存储格式，当字段允许为NULL时，key_len比不允许为NULL的大1个字节，计算公式为：

- **varchar（10）+ 允许为NULL**：2（varchar变长字段） + 10 \* ( Character Set）+ 1（允许为NULL）。
  - **varchar变长字段**：使用额外1或2个字节存储字符串长度。列长度小于255字节时，使用**1字节**表示，否则使用**2字节**表示，当存储的内容超出设置的长度时，内容会被截断。
  - **Character Set**：UTF8 = 3字节, GBK = 2字节, LATIN = 1字节。
  - **允许为NULL** = 1字节。
- **varchar（10）+ 不允许为NULL**：2（varchar变长字段） + 10 \* ( Character Set）。
- **char（10）+ 允许为NULL**：10 \* ( Character Set） + 1（允许为NULL）。
- **char（10）+ 不允许为NULL**：10 \* ( Character Set） 。

#### ref

表示索引的哪一列被引用了，将哪个字段或者常量，和key列所使用的字段进行比较。

- 如果ref是一个函数，则使用的值是函数的结果。
  - 要想查看是哪个函数，可在Explain语句之后紧跟一个SHOW WARNING语句。

#### rows

MySQL估算会扫描的行数，数值越小，性能越高。

#### filtered

表示符合查询条件的数据百分比，最大100，用rows * filtered可获得和下一张表连接的行数。

#### Extra

展示有关本次查询的附件信息，取值如下：

| 附件信息                                                     | 含义                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| Child of 'table' pushed join@1                               | 此值只会在NDB CLUSTER下出现                                  |
| const row not found                                          | 查询的表是空的                                               |
| Deleting all rows                                            | 对于DELETE语句，某些引擎（比如MyISAM）支持以一种简单而快速的方式删除所有数据，如果使用了这种优化，则会显示此值 |
| Distinct                                                     | 查找Distinct值，当找到第一个匹配的行后，将停止为当前行组合搜索更多的行 |
| FirstMatch(table_name)                                       | 当前使用了半连接FirstMatch策略                               |
| Full Scan on NULL key                                        | 子查询中的一种优化方式，在无法通过索引访问NULL值时会使用     |
| Impossible HAVING                                            | HAVING子句始终为false，不会命中任何行                        |
| Impossible WHERE                                             | WHERE子句始终为false，不会命中任何行                         |
| Impossible WHERE noticed after reading const tables          | MySQL已经读取了所有const（或者system）表，并发现WHERE子句始终为false |
| LooseScan（m...n）                                           | 当前使用了半连接LooseScan策略                                |
| No  matching min/max row                                     | MIN、MAX语句中，没有任何能满足WHERE条件的行                  |
| No matching row in const table                               | 对于关联查询，存在一个空表，或者没有行能够满足唯一索引条件   |
| No matching rows after partition pruning                     | 对于DELETE或者UPDATE语句，优化器在partition pruning（分区修剪）之后，找不到要DELETE或者UPDATE的内容 |
| No tables used                                               | 当此查询没有FROM子句或者拥有FROM DUAL子句时出现              |
| Not exists                                                   | MySQL能对LEFT JOIN优化，在找到符合LEFT JOIN的行后，不会为上一行组合中检查此表中的更多行，只查找一次 |
| Plan isn't ready yet                                         | 使用了EXPLAIN FOR CONNECTION，当优化器尚未完成为在指定连接中执行的语句创建执行计划时，就会出现此值 |
| Range checked  for each record（index map：N）               | MySQL没有找到合适的索引去使用，但是去检查是否可以使用range或者index_merge来检索行时，会出现此提示 |
| Recursive                                                    | 出现了递归查询                                               |
| Rematerialize                                                | 用得很少                                                     |
| Scanned N databases                                          | 表示在处理INFOMATION_SCHEMA表的查询时，扫描了几个目录，N的取值可以是0、1或者all |
| Select tables optimized away                                 | 优化器确定最多返回1行时会出现此提示，一般在用某些聚合函数访问存在索引的某个字段时，优化器会通过索引直接一次定位到所需要的数据行完成整个查询时，会展示该值 |
| Skip_open_table，Open_firm_only，Open_full_table             | 表示适用于INFORMATION_SCHEMA表查询的文件打开优化，Skip_open_table表示无需打开表文件，信息已经通过扫描数据字典获得；Open_firm_only表示仅需要读取数据字典以获取表信息；Open_full_table表示未优化的信息查找，表信息必须从数据字典以及表文件中获取 |
| Start temporary，End temporary                               | 表示临时表使用Duplicate Weedout策略                          |
| unique row not found                                         | 对于形如select ... from tab_name的查询，但没有行能够满足唯一索引或者主键查询的条件时出现 |
| **Using filesort**                                           | 当Query中包含ORDER BY操作，而且无法利用索引完成排序操作时，MySQL Query Optimizer不得不选择相应的排序算法来实现。在数据较少时，从内存排序，否则从磁盘排序。 其中，Explain不会显式地告诉客户端用哪种排序。 |
| **Using Index**                                              | 仅使用索引树中的检索列信息，不必进行其他查找以读取实际行，当查询仅使用属于单个索引的列时，会使用此策略 |
| **Using index condition**                                    | 使用索引下推时出现，表示先按条件过滤索引，过滤完索引后，找到符合索引条件的数据行，随后用WHERE子句中的其他非索引条件，去过滤这些数据行。 |
| **Using index for group-by**                                 | 数据访问和Using Index一样，所需数据只需要读取索引，当Query中使用GROUP BY或者DISTINCT子句时，如果所有分组字段也在索引中，该信息就会出现 |
| Using index for skip scan                                    | 表示使用了Skip Scan                                          |
| Using join buffer（Block Nested Loop），Using join Buffer（Batched Key  Access） | 使用Block Nested Loop或者Batched Key  Access算法来提高join的性能 |
| Using MRR                                                    | 使用了Muti-Range Read优化策略                                |
| Using sort_union（..），Using union（..），Using intersect（..） | 这些提示索引扫描如何合并为index_merge连接类型                |
| **Using temporary**                                          | 为了解决该查询，MySQL创建了一个临时表来保存结果，如果查询包含不同列的GROUP BY和ORDER BY子句，通常会发生这种情况。 |
| **Using Where**                                              | 如果不是读取表的所有数据，或者不仅仅通过索引就可以获取所有需要的数据时，则会出现该值 |
| Using where with pushed condiction                           | 仅用于NDB                                                    |
| Zero limit                                                   | 该查询有一个limit 0子句，不能选择任何行                      |

### 2.4. MySQL SQL性能分析？

除了使用EXPLAIN分析模拟优化器执行SQL语句外，还可以深入SQL内部来分析性能瓶颈，包括三种形式：

#### SHOW PROFILE

SHOW PROFILES，是MySQL的一个性能分析命令，可以跟踪SQL各种资源消耗情况，但官方文档声明SHOW PROFILES已被废弃，建议使用Performance Schema作为替代品。

- **使用步骤如下**：

  ```sql
  -- 1. 查看是否支持SHOW PROFILES功能，yes表示支持
  select @@have_profiling;
  
  -- 2. 查看是否启用了SHOW PROFILES功能，0表示未启动，1表示已启动
  select @@profiling;
  
  -- 3. 开启SHOW PROFILES功能
  set profiling = 1;
  
  -- 4. 使用SHOW PROFILES功能，默认展示15条，可通过set profiling_history_size来设置
  SHOW PROFILES;
  
  -- 5. 关闭SHOW PROFILES功能
  set profiling = 0;
  ```

  ![1630814806087](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1630814806087.png)

- 默认情况下，SHOW PROFILES只展示Status和Duration两列，如果想展示更多信息，还可以**指定type**。

| type             | 含义                                         |
| ---------------- | -------------------------------------------- |
| ALL              | 显示所有信息                                 |
| BLOCK IO         | 显示阻塞I/O次数                              |
| CONTEXT SWITCHES | 显示自愿及非自愿的上下文切换次数             |
| CPU              | 显示用户与系统CPU使用时间                    |
| IPC              | 显示消息发送与接收的次数                     |
| MEMORY           | 显示内存相关的开销                           |
| PAGE FAULTS      | 显示页错误相关的开销                         |
| SOURCE           | 列出相应操作对应的函数名及其在源码中的行位置 |
| SWAPS            | 显示swap交换次数                             |

#### INFORMATION_SCHEMA.PROFILING

SHOW PROFILES本质上读的就是INFORMATION_SCHEMA.PROFILING表，因此除了使用SHOW PROFILES做性能分析，也可以直接查询INFORMATION_SCHEMA.PROFILING，除非设置set profiling = 1，否则该表不会有任何数据。 

```sql
SHOW PROFILE FOR QUERY 2;

-- 等价于
SELECT STATE, FORMAT(DURATION, 6) AS DURATION
FROM INFORMATION_SCHEMA.PROFILING
WHERE QUERY_ID = 2
ORDER BY SEQ;
```

| INFORMATION_SCHEMA.PROFILING字段          | 含义                                         |
| ----------------------------------------- | -------------------------------------------- |
| QUERY_ID                                  | SQL语句的唯一标识                            |
| SEQ                                       | 一个序号，展示具有相同QUERY_ID值的行显示顺序 |
| STATE                                     | 分析状态                                     |
| DURATION                                  | 在这个状态下持续了多久时间（秒）             |
| CPU_USER，CPU_SYSTEM                      | 用户和系统CPU使用情况（秒）                  |
| CONTEXT_VOLUNTARY，CONTEXT_INVOLUNTARY    | 发生了多少次自愿和非自愿的上下文切换         |
| BLOCK_OPS_IN，BLOCK_OPS_OUT               | 块输入和输出操作的数量                       |
| PAGE_FAULTS_MAJOR，PAGE_FAULTS_MINOR      | 主要和次要的页错误信息                       |
| SWAPS                                     | 发生了多少次SWAP                             |
| SOURCE_FUNCTION，SOURCE_FILE，SOURCE_LINE | 当前状态是在源码的哪里执行的                 |

#### PERFOMANCE_SCHEMA

PERFORMANCE_SCHEMA是MySQL建议的性能分析方式，未来SHOW PROFILES、INFORMATION_SCHEMA.PROFILING都会被废弃。

```sql
-- 1. 查看是否开启性能监控，默认是开启的（>= MySQL 5.6）
mysql> SELECT * FROM performance_schema.setup_actors;
+------+------+------+---------+---------+
| HOST | USER | ROLE | ENABLED | HISTORY |
+------+------+------+---------+---------+
| %    | %    | %    | YES     | YES     |
+------+------+------+---------+---------+

-- 2. 开启相关监控
mysql> UPDATE performance_schema.setup_instruments
       SET ENABLED = 'YES', TIMED = 'YES'
       WHERE NAME LIKE '%statement/%';

mysql> UPDATE performance_schema.setup_instruments
       SET ENABLED = 'YES', TIMED = 'YES'
       WHERE NAME LIKE '%stage/%';
       
mysql> UPDATE performance_schema.setup_consumers
       SET ENABLED = 'YES'
       WHERE NAME LIKE '%events_statements_%';

mysql> UPDATE performance_schema.setup_consumers
       SET ENABLED = 'YES'
       WHERE NAME LIKE '%events_stages_%';

-- 3. 执行业务SQL后，获取EVENT_ID
mysql> SELECT EVENT_ID, TRUNCATE(TIMER_WAIT/1000000000000,6) as Duration, SQL_TEXT
       FROM performance_schema.events_statements_history_long WHERE SQL_TEXT like '%10001%';
+----------+----------+--------------------------------------------------------+
| event_id | duration | sql_text                                               |
+----------+----------+--------------------------------------------------------+
|       31 | 0.028310 | SELECT * FROM employees.employees WHERE emp_no = 10001 |
+----------+----------+--------------------------------------------------------+

-- 4. 根据EVENT_ID获取SQL性能分析信息
mysql> SELECT event_name AS Stage, TRUNCATE(TIMER_WAIT/1000000000000,6) AS Duration
       FROM performance_schema.events_stages_history_long WHERE NESTING_EVENT_ID=31;
+--------------------------------+----------+
| Stage                          | Duration |
+--------------------------------+----------+
| stage/sql/starting             | 0.000080 |
| stage/sql/checking permissions | 0.000005 |
| stage/sql/Opening tables       | 0.027759 |
| stage/sql/init                 | 0.000052 |
| stage/sql/System lock          | 0.000009 |
| stage/sql/optimizing           | 0.000006 |
| stage/sql/statistics           | 0.000082 |
| stage/sql/preparing            | 0.000008 |
| stage/sql/executing            | 0.000000 |
| stage/sql/Sending data         | 0.000017 |
| stage/sql/end                  | 0.000001 |
| stage/sql/query end            | 0.000004 |
| stage/sql/closing tables       | 0.000006 |
| stage/sql/freeing items        | 0.000272 |
| stage/sql/cleaning up          | 0.000001 |
+--------------------------------+----------+
```

### 2.5. MySQL OPTIMIZER_TRACE优化器跟踪？

OPTIMIZER_TRACE是MySQL 5.6引入的一项跟踪功能，可以跟踪优化器做出的各种决策，比如访问表的方法、各种开销的计算、各种的转换等，并将跟踪结果记录到INFORMATION_SCHEMA.OPTIMIZER_TRACE表中。

- 默认关闭，在开启后可分析SELECT、INSERT、REPLACE、UPDATE、EXPLAIN、SET、DECLARE、IF、RETURN、CALL。

- **使用步骤**：

  ```sql
  -- 1. 开启OPTIMIZER_TRACE
  SET OPTIMIZER_TRACE="enabled=on",END_MARKERS_IN_JSON=on;
  SET optimizer_trace_offset=-30, optimizer_trace_limit=30;
  
  -- 2. 执行业务SQL
  select *
  from salaries
  where from_date = '1986-06-26' and to_date = '1987-06-26';
  
  -- 3. 查看跟踪信息
  SELECT * FROM INFORMATION_SCHEMA.OPTIMIZER_TRACE limit 30;
  
  -- 4. 关闭OPTIMIZER_TRACE
  SET optimizer_trace="enabled=off";
  ```

- **OPTIMIZER_TRACE字段含义**：

  | 字段                              | 含义                                                         |
  | --------------------------------- | ------------------------------------------------------------ |
  | QUERY                             | 查询的SQL语句                                                |
  | TRACE                             | QUERY字段对应语句的跟踪信息                                  |
  | MISSING_BYTES_BEYOND_MAX_MEM_SIZE | 跟踪信息过长时，被截断的跟踪信息字节数                       |
  | INSUFFICIENT_PRIVILEGES           | 执行跟踪语句的用户是否有查看对象的权限，当不具有权限时，该列信息为1且TRACE字段为空，一般在调用带有SQL SECURITY DEFINER的视图，或者存储过程的情况下，会出现此问题 |

- **TRACE字段内容**：

  ```java
  # 跟踪结果展示
  TRACE: {
  	"steps": [{
  			# 1. 准备阶段的执行过程
  			"join_preparation": {
  				"select#": 1,
  				"steps": [{
  					"expanded_query": "/* select#1 */ select `salaries`.`emp_no` AS `emp_no`,`salaries`.`salary` AS `salary`,`salaries`.`from_date` AS `from_date`,`salaries`.`to_date` AS `to_date` from `salaries` where ((`salaries`.`from_date` = '1986-06-26') and (`salaries`.`to_date` = '1987-06-26'))"
  				}] /* steps */
  			} /* join_preparation */
  		},
  		{
  			# 2. 优化阶段的执行过程，是分析OPTIMIZER_TRACE的重点
  			"join_optimization": {
  				"select#": 1,
  				"steps": [{
  						# 2.1. 条件处理，主要对WHERE条件进行优化处理
  						"condition_processing": {
  							# 优化的对象类型，比如WHERE 或者 HAVING
  							"condition": "WHERE",
  							# 优化前的原始语句
  							"original_condition": "((`salaries`.`from_date` = '1986-06-26') and (`salaries`.`to_date` = '1987-06-26'))",
  							# 主要包括三步，分别是quality_propagation（）、constant_propagation（）、trivial_condition_removal（）
  							"steps": [
  							    # 等值条件句转换
  							    {
  									# 转换类型句
  									"transformation": "equality_propagation",
  									# 转换之后的结果输出
  									"resulting_condition": "(multiple equal('1986-06-26', `salaries`.`from_date`) and multiple equal('1987-06-26', `salaries`.`to_date`))"
  								},
  								# 常量条件句转换
  								{
  									# 转换类型句
  									"transformation": "constant_propagation",
  									# 转换之后的结果输出
  									"resulting_condition": "(multiple equal('1986-06-26', `salaries`.`from_date`) and multiple equal('1987-06-26', `salaries`.`to_date`))"
  								},
  								# 无效条件移除的转换
  								{
  									# 转换类型句
  									"transformation": "trivial_condition_removal",
  									# 转换之后的结果输出
  									"resulting_condition": "(multiple equal(DATE'1986-06-26', `salaries`.`from_date`) and multiple equal(DATE'1987-06-26', `salaries`.`to_date`))"
  								}
  							] /* steps */
  						} /* condition_processing */
  					},
  					{
  						# 2.2. 用于替换虚拟生成列
  						"substitute_generated_columns": {} /* substitute_generated_columns */
  					},
  					{
  						# 2.3. 分析表之间的依赖关系
  						"table_dependencies": [{
  							# 涉及的表名，如果有别名，也会展示出来
  							"table": "`salaries`",
  							# 行是否可能为NULL，这里指JOIN操作之后，这张表里的数据是不是可能为NULL。如果语句中使用了LEFT JOIN，则后一张表row_may_be_null会显示为true
  							"row_may_be_null": false,
  							# 表的映射编号，从0开始递增
  							"map_bit": 0,
  							# 依赖的映射表，当使用STRAIGHT JOIN强行控制连接顺序或者LEFT JOIN/RIGHT JOIN有顺序差别时，会在depends_on_map_bits中展示前置表的map_bit值
  							"depends_on_map_bits": [] /* depends_on_map_bits */
  						}] /* table_dependencies */
  					},
  					{
  						# 2.4. 列出所有可用的ref类型的索引，如果使用了组合索引的多个部分，则会在ref_optimizer_key_uses下列出多个元素，每个元素中会列出ref使用的索引及对应值
  						"ref_optimizer_key_uses": [{
  								"table": "`salaries`",
  								"field": "from_date",
  								"equals": "DATE'1986-06-26'",
  								"null_rejecting": false
  							},
  							{
  								"table": "`salaries`",
  								"field": "to_date",
  								"equals": "DATE'1987-06-26'",
  								"null_rejecting": false
  							}
  						] /* ref_optimizer_key_uses */
  					},
  					{
  						# 2.5. 估算需要扫描的记录数
  						"rows_estimation": [{
  							# 表名
  							"table": "`salaries`",
  							"range_analysis": {
  								# 如果全表扫描的话，需要扫描多少行，以及需要的代价
  								"table_scan": {
  									"rows": 2838216,
  									"cost": 286799
  								} /* table_scan */ ,
  								# 列出表中所有的索引，并分析其是否可用。如果不可用的话，会列出不可用的原因是什么，如果可用会列出索引中可用的字段
  								"potential_range_indexes": [{
  										"index": "PRIMARY",
  										"usable": false,
  										"cause": "not_applicable"
  									},
  									{
  										"index": "salaries_from_date_to_date_index",
  										"usable": true,
  										"key_parts": [
  											"from_date",
  											"to_date",
  											"emp_no"
  										] /* key_parts */
  									}
  								] /* potential_range_indexes */ ,
  								# 如果有下推的条件，则带条件考虑范围查询
  								"setup_range_conditions": [] /* setup_range_conditions */ ,
  								# 当使用了GROUP BY或者DISTINCT时，是否有合适的索引可用。当未使用GROUP BY或者DISTINCT时，会显示chosen=false，cause=not_group_by_or_distinct；如果使用了GROUP或者DISTINCT，但为多表查询时，则会显示chosen=false，cause=not_single_table。其他情况下会尝试分析索引（potential_group_range_indexes）并计算对应的扫描行数及其所需代价
  								"group_index_range": {
  									"chosen": false,
  									"cause": "not_group_by_or_distinct"
  								} /* group_index_range */ ,
  								# 是否使用了skip scan，skip scan是MySQL 8.0的新特性
  								"skip_scan_range": {
  									"potential_skip_scan_indexes": [{
  										"index": "salaries_from_date_to_date_index",
  										"usable": false,
  										"cause": "query_references_nonkey_column"
  									}] /* potential_skip_scan_indexes */
  								} /* skip_scan_range */ ,
  								# 分析各个索引的使用成本
  								"analyzing_range_alternatives": {
  									# range扫描分析
  									"range_scan_alternatives": [{
  										# 索引名
  										"index": "salaries_from_date_to_date_index",
  										# range扫描的条件范围
  										"ranges": [
  											"0xda840f <= from_date <= 0xda840f AND 0xda860f <= to_date <= 0xda860f"
  										] /* ranges */ ,
  										# 是否使用了index dive，该值会被参数eq_range_index_dive_limit变量值影响
  										"index_dives_for_eq_ranges": true,
  										# 该range扫描的结果集是否根据主键值进行排序
  										"rowid_ordered": true,
  										# 是否使用了MRR
  										"using_mrr": false,
  										# 表示是否使用了覆盖索引
  										"index_only": false,
  										# 扫描的行数
  										"rows": 86,
  										# 索引的使用成本
  										"cost": 50.909,
  										# 表示是否使用了索引
  										"chosen": true
  									}] /* range_scan_alternatives */ ,
  									# 分析是否使用了索引合并(index merge)，如果未使用，会在cause中展示原因；如果使用了索引合并，会在该部分展示索引合并的代价
  									"analyzing_roworder_intersect": {
  										"usable": false,
  										"cause": "too_few_roworder_scans"
  									} /* analyzing_roworder_intersect */
  								} /* analyzing_range_alternatives */ ,
  								# 在前一个步骤中分析了各类索引使用的方法及代价，得出了一定的中间结果之后，在summary阶段汇总前一阶段的中间结果确认最后的方案
  								"chosen_range_access_summary": {
  									# range扫描最终选择的执行计划
  									"range_access_plan": {
  										# 展示执行计划的type，如果使用了索引合并，则会展示index_roworder_intersect
  										"type": "range_scan",
  										# 索引名
  										"index": "salaries_from_date_to_date_index",
  										# 扫描的行数
  										"rows": 86,
  										# range扫描的条件范围
  										"ranges": [
  											"0xda840f <= from_date <= 0xda840f AND 0xda860f <= to_date <= 0xda860f"
  										] /* ranges */
  									} /* range_access_plan */ ,
  									# 该执行计划的扫描行数
  									"rows_for_plan": 86,
  									# 该执行计划的执行代价
  									"cost_for_plan": 50.909,
  									# 是否选择该执行计划
  									"chosen": true
  								} /* chosen_range_access_summary */
  							} /* range_analysis */
  						}] /* rows_estimation */
  					},
  					{
  						# 2.6. 负责对比各可行计划的开销，并选择相对最优的执行计划
  						"considered_execution_plans": [{
  							# 当前计划的前置执行计划
  							"plan_prefix": [] /* plan_prefix */ ,
  							# 涉及的表名，如果有别名，也会展示出来
  							"table": "`salaries`",
  							# 通过对比considered_access_paths，选择一个最优的访问路径
  							"best_access_path": {
  								# 当前考虑的访问路径
  								"considered_access_paths": [{
  										# 使用索引的方式
  										"access_type": "ref",
  										# 索引
  										"index": "salaries_from_date_to_date_index",
  										# 行数
  										"rows": 86,
  										# 开销
  										"cost": 50.412,
  										# 是否选用这种执行路径
  										"chosen": true
  									},
  									{
  										"access_type": "range",
  										"range_details": {
  											"used_index": "salaries_from_date_to_date_index"
  										} /* range_details */ ,
  										"chosen": false,
  										"cause": "heuristic_index_cheaper"
  									}
  								] /* considered_access_paths */
  							} /* best_access_path */ ,
  							# 类似于explain的filtered列，是一个估算值
  							"condition_filtering_pct": 100,
  							# 执行计划最终的扫描行数，由considered_access_paths.rows * condition_filtering_pct计算获得
  							"rows_for_plan": 86,
  							# 执行计划的代价，由considered_access_paths.cost相加获得
  							"cost_for_plan": 50.412,
  							# 是否选择了该执行计划
  							"chosen": true
  						}] /* considered_execution_plans */
  					},
  					{
  						# 2.7. 基于considered_execution_plans中选择的执行计划，改造原有的where条件，并针对表增加适当的附加条件，以便于单表数据的筛选，主要是为了便于索引条件下推（ICP），但ICP是否开启并不影响这部分内容的构造
  						"attaching_conditions_to_tables": {
  							# 原始的条件语句
  							"original_condition": "((`salaries`.`to_date` = DATE'1987-06-26') and (`salaries`.`from_date` = DATE'1986-06-26'))",
  							# 使用启发式算法计算已使用的索引，如果已使用的索引的访问类型是ref，则计算用range能否使用组合索引中更多的列，如果可以，则用range的方式替换ref
  							"attached_conditions_computation": [] /* attached_conditions_computation */ ,
  							# 附加之后的情况汇总
  							"attached_conditions_summary": [{
  								# 表名
  								"table": "`salaries`",
  								# 附加的条件或原语句中能直接下推给单表筛选的条件
  								"attached": "((`salaries`.`to_date` = DATE'1987-06-26') and (`salaries`.`from_date` = DATE'1986-06-26'))"
  							}] /* attached_conditions_summary */
  						} /* attaching_conditions_to_tables */
  					},
  					{
  						# 2.8. 最终的、经过优化后的表条件
  						"finalizing_table_conditions": [{
  							"table": "`salaries`",
  							"original_table_condition": "((`salaries`.`to_date` = DATE'1987-06-26') and (`salaries`.`from_date` = DATE'1986-06-26'))",
  							"final_table_condition   ": null
  						}] /* finalizing_table_conditions */
  					},
  					{
  						# 2.9. 改善执行计划
  						"refine_plan": [{
  							"table": "`salaries`"
  						}] /* refine_plan */
  					}
  				] /* steps */
  			} /* join_optimization */
  		},
  		{
  			# 3. 执行阶段的执行过程
  			"join_execution": {
  				"select#": 1,
  				"steps": [] /* steps */
  			} /* join_execution */
  		}
  	] /* steps */
  }
  ```

### 2.6. MySQL数据库诊断命令？

MySQL数据库诊断命令，可以帮助了解数据库的运行情况。

| 命令                 | 作用                                                         |
| -------------------- | ------------------------------------------------------------ |
| SHOW PROCESSLIST     | 查看当前正在运行的线程，如果执行此命令的用户拥有PROCESS权限，则可看到所有线程；否则只能看到自己的线程。等价于select * from information_schema.PROCESSLIST; |
| SHOW STATUS          | 查看服务器相关信息                                           |
| SHOW VARIABLES       | 查看MySQL的变量                                              |
| SHOW TABLE           | 查看表以及视图的状态                                         |
| SHOW INDEX           | 查看索引相关信息                                             |
| SHOW ENGINE          | 查看有关存储引擎的相关信息                                   |
| SHOW MASTER STATUS   | 查看有关master binlog文件的相关信息                          |
| SHOW SLAVE STATUS    | 查看slave线程的相关信息                                      |
| SHOW PROCEDURE       | 查看存储过程相关信息                                         |
| SHOW FUNCTION STATUS | 查看函数相关信息                                             |
| SHOW TRIGGERS        | 查看触发器相关信息                                           |
| SHOW WARNINGS        | 查看error、waring、note级别的诊断信息                        |
| SHOW ERRORS          | 查看error级别的诊断信息，和show warnings类似                 |
| SHOW BINARY LOGS     | 查看服务器上所有的binary log                                 |
| SHOW BINLOG EVENTS   | 查看binary log中的事件                                       |
| SHOW RELAYLOG EVENTS | 查看复制从库的relay log事件相关信息                          |

### 2.7. MySQL索引分类？

- 索引，是按照特定的数据结构，把数据表中的数据放在索引文件中，以便于快速查找。
- 索引存在于磁盘中，会占据物理空间。

| 存储内容   | InnoDB           | MyISAM             |
| ---------- | ---------------- | ------------------ |
| 表结构文件 | .frm             | .frm               |
| 表数据文件 | .idb（聚蔟索引） | .myd               |
| 索引文件   | .idb（聚蔟索引） | .myi（非聚蔟索引） |

#### 按数据结构角度分

##### B-Tree索引

###### BST

- **特点**：
  - Binary Sort Tree，二叉树查找树，左边的结点比右边的结点小，右边的结点比左边的结点大。
  - 查找效率取决于树的高度。
- **查找流程**：
  1. 从根结点出发比较要查找的关键字key。
  2. 如果根结点关键字等于key，则返回根结点。
  3. 如果根结点关键字比key小，则继续查找左子树。
  4. 如果根结点关键字比key大，则继续查找右子树。
- **缺点**：有可能会退化成链表，查询的时间复杂度也从O(logn)退化成O(n)。

![1630832978045](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1630832978045.png)

###### AVL

- **特点**：
  - Self-balancing binary search tree，平衡二叉搜索树，每个结点的左子树和右子树的高度差不能超过1。
  - 对于n个结点，树的高度是logn，查询的时间复杂度为O(logn)。
- **缺点**：结点过多时，数的高度也会越来越大，最终导致查询效率较低。

###### B-Tree

- **特点**：
  - Balance Tree，平衡多路搜索树，根结点的子结点个数为 2 <= x <= m，m是树的阶。
    - 假设m=3，则根结点可以有2~3个孩子。
  - 中间结点的子结点个数为m/2 <= y <= m。
    - 假设m=3，中间结点至少有2个孩子，最多3个孩子。
  - 每个中间结点包含n个关键字，n为子结点个数-1，且按升序排序。
    - 如果中间结点有3个子结点，则中间结点里面会有**2个关键字，且按升序排序**。
  - Pi（i=1，...，n+1）为指向子树根结点的指针，其中P[1]指向关键字小于Key[1]的子树，P[i]指向关键字属于（Key[i-1]，Key[i]）的子树，P[n+1]指向关键字大于Key[n]的子树。
    - 如果中间结点有3个子结点，则中间结点里面会有3个指针，2个关键字。
    - P1、P2、P3为指向子树根结点的指针，P1指向关键字小于Key1的树，P2指向Key1~Key2之间的子树，P3指向大于Key2的树。
- **优点**：可以有效地降低树的高度，树的阶m越大，树的高度就越低，查询次数就越少，性能就越高。
  - 在大规模数据存储的时候，红黑树往往出现由于**树的深度过大**，而造成磁盘IO读写过于频繁，进而导致效率低下的情况，此时，只要通过某种较好的树结构减少树的结构尽量减少树的高度，而B树与B+树每个结点可以有多个关键字，由于多路子树的存在，可以大大降低降低树的高度。
- **缺点**：范围查询时，需要多次从头遍历树，性能较低。

![1630833491913](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1630833491913.png)

###### B+Tree

- **特点**：
  - B+Tree中，有n个子结点的结点，会含有n个关键字。
    - 而B-Tree中的是n-1个关键字。
  - B+Tree中，所有的叶子结点中包含了全部的关键字信息（会冗余父结点的关键字），且叶子结点按照关键字大小， 自小而大地顺序链接，构成一个有序链表。
    - 而B-Tree中的叶子结点不会包括全部的关键字。
  - B+Tree中，非叶子结点仅用于存放索引（即关键字），不保存数据记录（即data），记录都存放在叶子结点中。
    - 而B-Tree中的非叶子结点既保存索引，也保存数据记录。
- **优点**：
  - B+树是B树的升级版，B+树只有叶节点存放数据，其余节点用来索引，其索引节点可以全部加入内存，增加查询效率，叶子节点可以做双向链表，从而**提高范围查找的效率，增加的索引的范围**。
  - B+树的磁盘读写代价低，更少的查询次数，查询效率更加稳定，有利于对数据库的扫描。

![1630834501421](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1630834501421.png)

- **磁盘预读原理**：将**一个节点的大小设为等于一个页**，这样每个节点只需要一次I/O就可以完全载入。为了达到这个目的，在实际实现B-Tree还需要使用如下技巧：
  - 每次新建节点时，直接申请一个页的空间，这样就保证**一个节点物理上也存储在一个页里**，加之计算机存储分配都是按页对齐的，就实现了一个node只需一次I/O。
- **页存储**：
  - 自mysql5.7后，提供了一个设定page大小的参数innodb_page_size，默认值是16K，可以通过来改变page的大小来**间接改变m阶**，来改变B+树的m的大小。
  - 比如要存20G大小的数据，那么page=16K和page=4K，树的高度是不一样的，也就是说，树的高度是根据要存下的数据是多少来决定的。

![1630915960707](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1630915960707.png)

###### B+Tree VS B-Tree

| 场景     | B+Tree                                                    | B-Tree                             |
| -------- | --------------------------------------------------------- | ---------------------------------- |
| 等值查询 | 中间结点存储的索引多，树整体更加矮胖，磁盘I/O次数比较稳定 | 中间结点会存放数据，查询效率不稳定 |
| 范围查询 | 只需要遍历叶子结点的有序链表即可，性能较高                | 需要多次从根结点开始遍历，性能较低 |

###### InnoDB VS MyISAM

|                                      | InnoDB                                 | MyISAM                                           |
| ------------------------------------ | -------------------------------------- | ------------------------------------------------ |
| 索引数据结构                         | B+Tree                                 | B+Tree                                           |
| 主键索引                             | 叶子结点存储主键及数据记录（聚蔟索引） | 叶子结点存储的是指向数据记录的指针（非聚蔟索引） |
| 非主键索引（即二级索引或者辅助索引） | 叶子结点存储索引以及主键（非聚蔟索引） | 叶子结点存储的是指向数据记录的指针（非聚蔟索引） |

##### Hash索引

- **特点**：
  - 基于哈希表来实现，用索引列的值来计算hashCode，根据hashCode组成一个哈希表，同时在哈希表中存储了指向每个数据行的物理位置。
  - 由于使用哈希算法，因此时间复杂度为O（1），访问速度非常快，而当产生哈希冲突时，需要遍历指针数组或者指针链表，性能会下降一些，因此使用哈希索引需要尽量避免哈希冲突的发生。
- **缺点**：
  - 由于一个值只能对应一个hashCode，且根据哈希算法进行散列，哈希后的值不再具有比较意义，因此hash索引**不支持范围查找和排序，只支持等值匹配**，因此Hash索引只适合特殊场景下才使用。

![1630836502073](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1630836502073.png)

###### InnoDB VS MyISAM

| InnoDB                                                       | MyISAM                 |
| ------------------------------------------------------------ | ---------------------- |
| 1）自适应的hash索引，不支持显示创建Hash索引，当InnoDB发现某个索引值使用非常频繁，它会在B+Tree的基础上再建立一个哈希索引。2）可以使用show variables like 'innodb_adaptive_hash_index'查看开关是否打开。3）以及使用set global innodb_adaptive_hash_index = ‘OFF’，来关闭该功能（默认是打开的） | 支持显示地创建Hash索引 |

##### 空间索引

- **特点**：
  - 基于R-Tree构建，也叫R-Tree索引，用来存储GIS数据（地图数据）。
  - 在早期只有MyISAM引擎支持空间索引，在MySQL 5.7开始，InnoDB也开始支持空间索引了。

##### 全文索引

- **特点**：
  - 用于适应全文搜索的需求。
  - 在MySQL 5.7之前，全文索引不支持中文，经常搭配Sphinx的使用，而从5.7开始MySQL内置了ngram，支持中文。
  - 但是目前一般使用搜索引擎来解决全文搜索的需求，比如ES。

#### 按功能逻辑角度分

##### 普通索引

普通索引，是基础的索引，没有任何约束，主要用于提高查询效率。

```sql
CREATE INDEX index_name ON table(column(length))
```

##### 唯一索引

- 唯一索引，是在普通索引的基础上，增加了数据唯一性的约束，其索引列的值必须唯一，且允许为NULL值。
- 如果一个为唯一索引同时还是组合索引，那么表示列值组合必须唯一。
- 在一张数据表里可以有多个唯一索引。

```sql
CREATE UNIQUE INDEX indexName ON table(column(length))
```

##### 主键索引

主键索引，是一种特殊的唯一索引，其索引列不允许有NULL值，并且一张表最多只有一个主键索引。

##### 组合索引

组合索引，指多个字段上创建的索引，使用组合索引时需要遵循**最左前缀原则**。

```sql
CREATE index index_name ON table (column1, column2);
```

##### 全文索引

全文索引，用来检索文本中的关键字，用得很少，一般应对这种需求用ES或者Solr之类的全文搜索引擎比较好。

```sql
CREATE FULLTEXT INDEX ...
```

#### 按物理存储角度分

![1630845876331](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1630845876331.png)

##### 聚蔟索引

聚蔟索引，**叶子结点就是数据结点**，将表数据和主键一起存储，其数据的物理存放顺序与索引顺序是一致的，即只要索引是相邻的，那么对应的数据一定也是相邻地存放在磁盘上，而由于无法同时把数据行同时存放在两个不同的地方，因此一张表只有一个聚蔟索引。

- InnoDB的主键索引使用的是聚蔟索引，如果创建的表没有主键，InnoDB则会隐式定义一个主键来作为聚蔟索引。
- **聚蔟索引的二级索引**：叶子结点不会保存引用行数据，而是保存行的主键值，然后根据主键值获取到数据，比如InnoDB的普通索引、联合索引。
- **优点**：
  - 查找效率理论上比非聚蔟索引的要高，因为数据就存放在索引上，且数据存储顺序就是索引的顺序。
  - 范围查询方便，因为只需要遍历索引的叶子结点即可。
- **缺点**：
  - 插入、修改、删除操作的性能比非聚蔟索引的要低，因为非聚蔟索引在更新时，数据行在表中的位置不会发生变化，而聚蔟索引的数据行需要重新移动位置。
  - **插入速度严重依赖于插入顺序**，因为按照主键顺序插入数据是InnoDB表中速度最快的插入方式，如果不是按照主键顺序插入数据，那么在插入完成后，最好使用**optimize table**命令重新组织一下表，因此，对于聚集索引，一般都会定义一个自增的ID列为主键。
  - **更新主键的代价很高**，因为会导致InnoDB移动被更新的行。
  - 另外，插入新行或者更新主键导致数据行被移动时，还有可能面临**页分裂**的问题，当行的主键值被要求必须插入行数据到某个已满的页中，InnoDB会将页分裂为两个页面来容纳这行，产生一次页分裂操作，这会**导致表占用更多的磁盘空间。**
  - 而由于页分裂导致数据存储不连续，或者本身行数据就存储得比较稀疏时，聚蔟索引可能导致**全表扫描变得更慢**。

##### 非聚蔟索引

非聚蔟索引，**叶子结点不存储数据**，而是指向对应数据块的指针，表数据和索引分开存储，查询时先找到索引，再根据索引找到对应的数据行。

- MyISAM的主键索引使用的是非聚蔟索引。

##### 聚蔟索引与自增主键？

1. **随机主键给聚蔟索引的坏处**：

   - 如果主键不是自增ID（比如UUID），会使得聚簇索引的插入变得随机，使得数据没有任何聚集特性。
   - 性能上，由于插入的随机性产生乱序写入，为了给新行分配空间，InnoDB不得不频繁做页分裂操作，导致需要移动大量数据，不断地调整数据的物理地址和分页，**影响写入的性能**。
   - 而由于**页分裂**导致数据存储不连续，或者本身行数据就存储得比较稀疏时，聚蔟索引可能导致**全表扫描变得更慢**。

   => 因此随机主键在插入完成后，最好使用**optimize table**命令，来重建表并优化页的填充。

2. **自增主键对聚蔟索引的好处**：对于聚蔟索引，叶子结点就是数据结点，将表数据和主键一起存储，其数据的物理存放顺序与索引顺序是一致的，即只要索引是相邻的，那么对应的数据一定也是相邻地存放在磁盘上。

   - 对于这种数据结构，如果按主键自增的顺序写入数据，InnoDB只需要一页一页地写，**索引结构相对紧凑，磁盘碎片少，效率也高**。

3. **自增主键对聚蔟索引的坏处**：对于高并发场景下，如果在InnoDB中按主键顺序插入，**可能会造成明显的锁争用**。这是因为：

   - **主键的上界（左边）会成为热点**：由于所有插入都发生在这里，所以并发插入可能会导致**间隙锁竞争**。
   - **另一个热点是自增锁机制**：如果遇到这个问题，则可能需要考虑重新设计表或者应用，比如应用层面生成单调递增的主键ID，插表不使用auto_increment机制。

### 2.8. MySQL创建索引的原则？

**创建索引的原则**：目的是让索引过滤更多的行，**快速定位记录，或者利用索引的有序性**。

- **where条件、分组、排序、去重、联表、唯一的字段**，一般建议创建索引。
- **更新多、查询少、表数据少、列重复数据多、不是where频繁使用的字段**，一般不建议创建索引。

#### 创建索引

```sql
CREATE [UNIQUE | FULLTEXT]  INDEX 索引名 ON 表名(字段名) [USING 索引方法]；
-- eg: CREATE index index_name ON table (column1, column2);

-- 说明：
-- UNIQUE: 可选。表示索引为唯一性索引。
-- FULLTEXT: 可选。表示索引为全文索引。
-- INDEX: 用于指定字段为索引，两者选择其中之一就可以了，作用是一样的。
-- 索引名: 可选。给创建的索引取一个新名称。
-- 字段名: 指定索引对应的字段的名称，该字段必须是前面定义好的字段。
-- 注： 索引方法默认使用B+TREE。
```

#### 哪些场景建议创建索引

1. **select语句中，频繁作为where条件的字段**，建议创建索引，可以快速定位行记录。
2. **update和delete语句中，where条件的字段**，建议创建索引，可以快速定位行记录。
3. **需要分组、排序的字段**，建议创建索引，可以利用索引的有序性，避免文件内排序和建立临时表。
4. **distinct所使用的字段**，建议创建索引，可以快速定位行记录。
5. **字段的值有唯一性约束**，可以创建主键或者唯一索引。
6. **对于多表查询，联接字段应创建索引**，且**类型务必保持一致**，因为如果不一致可能会出现**类型的隐式转换**，导致索引失效。

#### 哪些场景不建议创建索引

1. **where子句里用不到的字段**，不建议创建索引，索引作用是用来定位记录，如果查询条件里不需要这个字段，那么这个字段也不需要创建索引。
2. **表的记录非常少时**，不建议创建索引，因为这种情况下（比如100条数据），建立索引的作用并不大，没有必要创建索引。
3. **列里有大量重复数据时**，不建议创建索引，因为此时索引的选择性低，建立索引作用不大。
   - **唯一索引比普通索引效率高的原因**：索引选择性越高，查询的效率就越好，因为可以在查找时过滤掉更多的行，如果索引列重复的值过多，索引的选择性就低，查询能过滤掉的行就少，此时效率自然就不高。
4. **频繁更新的字段，想要建立要考虑索引维护的开销**，如果该字段查询得少，则没必要为它创建索引。

#### 索引性能评估公式

多数情况下，可以通过计算磁盘的搜索次数来估算查询性能。

- **对于比较小的表**：可以在一次磁盘搜索中找到，因为索引可能已经被缓存了。
- **对于更大的表**：可以使用B-Tree索引来进行估算，估算公式为：

```mysql
磁盘I/O次数 = log(row_count) / log(
          	index_block_length / 3 * 2 / (index_length + data_pointer_length)
           ) + 1
          
-- 其中，index_block_length=1024字节，data_pointer_length=4字节，则公式等于
磁盘I/O次数 = log(row_count) / log(
          	1024 / 3 * 2 / (index_length + 4)
           ) + 1 
           = log（row_count） / [ log(683 / (index_length + 4) ) ]
           = log（row_count） / [log683 - log(index_length + 4)]

-- => 可见，当row_count越大，index_length越大，内存中存放的索引就越少，需要的磁盘I/O次数就越多，性能就越差。
```

### 2.9. MySQL索引失效与解决方案？

#### 索引列不独立

- **独立**：是指列不能是**表达式**的一部分，也不能是**函数**的参数。
- **解决方案**：
  - 事先计算好结果，再传到SQL，避免在where条件等号左侧做表达式和函数运算。
  - 或者用等价的SQL去替代。

```sql
-- 示例1：索引字段不独立(索引字段进行了表达式计算)
explain
select *
from employees
where emp_no + 1 = 10003;
-- 解决方案：事先计算好表达式的值，再传过来，避免在SQLwhere条件 = 的左侧做计算

-- 示例2：索引字段不独立(索引字段是函数的参数)
explain
select *
from employees
where SUBSTRING(first_name, 1, 3) = 'Geo';
-- 解决方案：预先计算好结果，再传过来，在where条件的左侧，不要使用函数；或者使用等价的SQL去实现
explain
select *
from employees
where first_name like 'Geo%';
```

#### 使用了左模糊

- **解决方案**：尽量避免使用左模糊，如果避免不了，可以考虑使用搜索引擎去解决。

```sql
-- 示例3：使用了左模糊
explain
select *
from employees
where first_name like '%Geo%';
-- 解决方案：尽量避免使用左模糊，如果避免不了，可以考虑使用搜索引擎去解决
explain
select *
from employees
where first_name like 'Geo%';
```

#### 使用OR查询的部分字段没有索引

- **解决方案**：额外添加索引，使得or的两侧字段都走索引。
  - 添加后，explain#type会变成index_merge，表示索引合并（mysql的内部优化机制），合并了or两侧字段的索引。

```sql
-- 示例4：使用OR查询的部分字段没有索引
explain
select *
from employees
where first_name = 'Georgi'
   or last_name = 'Georgi';
-- 解决方案：分别为first_name以及last_name字段创建索引
```

#### 字符串条件未使用''引起来

- **解决方案**：实质上发生了类型的隐式转换，因此需要规范地编写SQL。

```sql
-- 示例5：字符串条件未使用''引起来
explain
select *
from dept_emp
where dept_no = 3;
-- 解决方案：规范地编写SQL
explain
select *
from dept_emp
where dept_no = '3';
```

#### 不符合最左前缀原则的查询

- **解决方案**：调整索引的顺序，使其满足最左前缀原则。

```sql
-- 示例6：不符合最左前缀原则的查询
-- 存在index(last_name, first_name)
explain select *
        from employees
        where first_name = 'Facello';
-- 解决方案：调整索引的顺序，变成index(first_name,last_name)/index(first_name)
```

#### 索引字段建议添加NOT NULL约束

- **原因**：**单列索引无法储null值，复合索引无法储全为null的值**，因此查询时，如果采用is null条件，则不能利用到索引，只能全表扫描。
- **解决方案**：索引字段设置成NOT NULL，甚至可以把所有字段都设置成NOT NULL并为字段设置默认值。

```sql
-- 示例7：索引字段建议添加NOT NULL约束
-- 单列索引无法储null值，复合索引无法储全为null的值
-- 查询时，采用is null条件时，不能利用到索引，只能全表扫描
-- MySQL官方建议尽量把字段定义为NOT NULL：https://dev.mysql.com/doc/refman/8.0/en/data-size.html
explain
select *
from `foodie-shop-dev`.users
where mobile is null;
-- 解决方案：把索引字段设置成NOT NULL，甚至可以把所有字段都设置成NOT NULL并为字段设置默认值
```

#### 隐式转换导致索引失效

- **解决方案**：在创建表的时候尽量规范一点，比如统一用int，或者bigint。

```sql
-- 示例8：隐式转换导致索引失效
-- 目前没这样的表，演示不了，同学们可以试试把de.emp_no的字段类型改成varchar
select emp.*, d.dept_name
from employees emp
         left join dept_emp de
                   on emp.emp_no = de.emp_no
         left join departments d
                   on de.dept_no = d.dept_no
where de.emp_no = '100001';
-- 解决方案：在创建表的时候尽量规范一点，比如统一用int，或者bigint
```

### 3.0. MySQL最左前缀原则与原理？

- **概念**：最左前缀原则，指的是索引按照**最左优先**的方式匹配索引，主要使用在**联合索引**中，其索引的数据结构在InnoDB中是B+Tree，它会按照第一个关键字、第二个关键字...**顺序进行索引排列**。
- **原理**：如果查询条件遵循最左前缀原则，那么MySQL会：
  1. 先根据第一个关键字查找联合索引树。
  2. 定位到索引树的叶子结点后，找出所有满足第一个关键字的叶子结点。
  3. 第一个关键字选择的叶子结点确定后，如果这些叶子结点对于**第二个关键字是存在且顺序**的，则MySQL会使用二分查找的方式查找这些叶子结点，接着该索引继续匹配下一个关键字。
  4. 如果这些叶子结点对于第二个关键字**不存在或者乱序**，此时则会在关键字上出现**索引失效**，导致后面的条件无法继续走上索引，而是根据之前选择的叶子结点上的**主键回表查找**。
- **联合索引失效场景**：这些场景都是因为匹配到某个关键字的叶子结点时，**不满足存在以及顺序性**。
  - 如果不是从索引的最左列开始查找，则无法使用索引。
  - 不能在中间跳过索引中的某个列，这样的查询只能使用到索引的前几列。
  - 如果查询中有某个列的范围查询，则该列右边的所有列都无法使用索引优化查找。

![1630924286192](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1630924286192.png)

### 3.1. MySQL索引调优？

#### 长字段调优

- **背景**：

  - 索引字段类型范围过长，比如varchar(300)，会导致占用的空间大。
  - 而根据性能公式`磁盘I/O次数 = log（row_count） / [log683 - log(index_length + 4)]`可知，索引长度越大，一次磁盘I/O读取到的索引就越少，磁盘I/O次数就越多，性能就越差。

- **解决方案**：

  - **使用Hash算法减少B-Tree索引长度**：根据原索引字段建立一个新的字段，使用Hash算法计算原字段的Hash值，并在其上建立B-Tree索引，因此本质上依然是B-Tree索引，是"伪Hash索引"，并不是真正的哈希索引。另外还需要注意：

    - **Hash后的长度应该比较小**：SHA1/MD5是不合适的，因为它们Hash出来的值也是比较长的。
    - **应尽量避免hash冲突**：就目前来说，流行的Hash算法有CRC32（）和FNV64（）。

    ```sql
    -- Hash索引使用案例：
    -- 1) 如果直接使用B-Tree索引存储URL，由于URL一般都比较长，存储的内容就会很大。
    -- 2) 此时可以在表中新建一个列url_crc，把crc32(url)计算后的hashCode存到这个列，并在url_crc列上建立一个Hash索引，这样只需要很小的索引就可以为超长的列建立索引，性能会提高很多。
    -- 3) 但要注意，因为有可能会有哈希冲突，所以还要对url进行等值比较。
    ```

  - **使用前缀索引**：alter table employees add key (first_name(n))，其中n应调试到**最佳的索引选择性**，以提高索引的效益。

    - **索引选择性 = 不重复的索引值 / 数据表的总记录数**，结果越大，表示选择性越高，性能也越好。
    - 另外，翻转字符串存储 + 前缀索引，可以实现**后缀索引**。
    - **优点**：能节省空间，提高索引性能；优化对上层应用透明，落地成本小。
    - **缺点**：无法做order by和group by；也无法使用覆盖索引。

#### 单列索引调优

如果使用两个**单列索引**作为查询条件，则会导致MySQL合并索引，对两索引列匹配的结果求交集，导致产生额外的开销。

- 如果出现索引合并，往往说明存在的索引不够合理。
- 调优的目的是为了解决性能问题，如果SQL暂时没有性能问题，可以先放一边暂时不管。
- 必要时，可以将两列索引组合成为一个**组合索引**，可以节省额外的开销，但使用时需要注意**最左前缀原则**。

#### 覆盖索引调优

覆盖索引指的是，对于索引X，**SELECT的字段直接出现在从索引树上**，而无需回表数据里获取，换句话说就是，当SELECT的字段被使用的索引字段覆盖时，这种用法就是覆盖索引。

- **特点**：使用覆盖索引时，Explain#Extra结果会展示Using Index。

- **原理**：
  1. 如果索引是非主键索引，则索引树上只有对应数据行的主键，此后还需要根据主键，然后在主键索引树上定位到叶子结点后，才能返回对应的数据行（**回表查询**）。
     - **回表查询**，就是对于二级索引，需要先定位到主键值，然后再定位行记录，其性能比覆盖索引扫一遍索引树的低。
  2. 如果该索引满足覆盖索引，则在索引树上就找到需要的数据行的字段，此时直接返回即可，无需回表查询。

- **优点**：可以直接在索引树上获取到想要的数据，而无需回表查询，减少了回表的开销，提升性能。
  - 因此，编写SQL时，**尽量只返回想要的字段**，一来可以使用上覆盖索引，二来可以减少网络传输的开销，从而提升SQL的性能。

#### 重复索引调优

重复索引指的是，在相同的列上，按照相同的顺序创建的索引。

- 由于索引在增删改时是有开销的，所以**尽量避免重复索引**，如果发现则应该删除。

```sql
-- 重复索引
create table test_table
(
    id int not null primary key auto_increment,
    a  int not null,
    b  int not null,
    UNIQUE (id),
    INDEX (id)
) ENGINE = InnoDB;
-- 发生了重复索引，改进方案： 删掉唯一索引和普通索引
```

#### 冗余索引调优

冗余索引指的是，如果已经存在索引index（A，B），又创建了index（A），那么index（A）就是index（A，B）的冗余索引。

- 冗余索引针对的是联合索引，不是Hash索引和其他索引。
- 但冗余索引也有特例，特别是与**where + order by / group by主键**时。
  - 这种情况下，是利用了复合索引来定位叶子主键，然后order by / group by利用了主键索引的排序特性，来避免文件内排序。
  - 如果误删某个冗余索引，可能会出现问题。

```sql
-- 冗余索引: index(a)是index(a, b)的冗余索引
-- 冗余索引特例:
explain
select *
from salaries
where from_date = '1986-06-26'
order by emp_no;

-- 创建from_date索引: salaries_from_date_index
-- index(from_date): type=ref, extra=null，使用了索引
-- index(from_date) 某种意义上来说就相当于index(from_date, emp_no) => 因为emp_no是主键索引, 所以order by子句可以使用索引

-- 而index(from_date, to_date): type=ref, extra=Using filesort，order by子句无法使用索引
-- index(from_date, to_date)某种意义上来说就相当于index(from_date, to_date, emp_no) 
-- => 该复合索引跳过了to_date, 导致from_date定位到叶子主键时, 顺序是乱序的, 对于后面的order by就没办法利用上索引了, 所以出现了文件内排序Using filesort
-- 因此, 这种特例下: 在有了index(from_date, to_date)复合索引, 为了保证order by能走索引, 还需要创建冗余索引index(from_date)
```

#### 未使用的索引调优

未使用的索引指的是，某个索引根本未被使用。

- 这种索引就是累赘，应当删除。

### 3.2. MySQL索引条件下推优化？

- **概念**：索引条件下推， Index Condition PushDown，ICP，是针对MySQL**使用索引从表中检索行**情况的优化，其目标是**减少全行读取的次数，从而减少 I/O 操作**。

  - 在没有 ICP 的情况下，存储引擎遍历索引，以定位基表中的行，并将它们返回给 MySQL 服务器，该服务器评估`WHERE`行的条件。
    - 步骤⑥从存储引擎返回查找到的**多条元组**给MySQL Server，MySQL Server在⑦得到较多的元组。
    - 步骤⑦到⑧，MySQL Server依据WHERE子句条件进行过滤，得到满足条件的元组。
    - 因此，在无ICP方式下，是在MySQL Server层得到较多元组，然后才过滤，最终得到的是少量的、符合条件的元组。

  ![1630976765074](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1630976765074.png)

  - 启用ICP后，如果`WHERE`可以仅使用索引中的列来评估部分条件，则MySQL服务器会推送这部分**`WHERE`条件下降到存储引擎**。然后，存储引擎使用索引条目评估推送的索引条件，并且仅当满足该条件时才从表中读取行。
    - 步骤③，不仅要在索引行进行索引读取，还要在该阶段依据MySQL Server下推过来的条件进行**条件判断**，不满足条件的则不去读取表中的数据，直接在索引树上进行下一个索引项的判断，直到有满足条件的，才进行步骤④。
    - 步骤⑥从存储引擎返回查找到的**少量元组**给MySQL Server，MySQL Server在⑦得到少量的元组。
    - 因此，在对比无ICP的方式，ICP方式返回给MySQL Server层的是**少量的、符合条件的元组**。

  ![1630976784073](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1630976784073.png)

- **特点**：当使用Explain进行分析时，如果使用了索引条件下推，Extra会显示**Using index condition**。

- **作用**：索引条件下推优化ICP，可以减少**存储引擎访问基表的次数**和**MySQL服务器访问存储引擎的次数**。

- **适用条件**：

  - **当需要全表扫描时**，比如range、ref、eq_ref、ref_or_null，适用于InnoDB引擎和MyISAM引擎的查询。
  - 对于InnoDB引擎，**只适用于二级索引**，因为其聚簇索引会将整行数据读到InnoDB缓冲区中，此时数据已经在内存中了，存储已经不再需要去I/O读取了，使得ICP减少I/O的目的失去意义，因此**ICP不适合聚蔟索引**。
  - 不能下推引用**子查询、存储函数、触发器函数**的条件。

```sql
-- 关闭索引下推条件优化
SET optimizer_switch = 'index_condition_pushdown=off';

-- 开启索引下推条件优化
SET optimizer_switch = 'index_condition_pushdown=on';
```

### 3.3. MySQL SQL生命周期？

![1630983427049](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1630983427049.png)

① 通过客户端/服务器通信协议与MySQL建立连接，并查询是否有权限。

② MySQL8.0之前需要先查看是否开启缓存，如果开启了Query Cache且命中完全相同的SQL语句，则将查询结果直接返回给客户端。

③ 由解析器进行语法语义解析，并生成解析树，如查询是select、表名tb_student、条件是id='1'等。

④ 由查询优化器生成执行计划，根据索引看看是否可以优化。

⑤ 由执行引擎执行SQL语句对应的执行计划，并根据存储引擎类型得到查询结果，如果开启了Query Cache，则还要把结果缓存起来，否则直接返回。

### 3.4. MySQL SQL执行顺序？

1. **FROM**：将数据从硬盘加载到数据缓冲区，方便对接下来的数据进行操作。
2. **WHERE**：从基表或视图中选择满足条件的元组。
3. **JOIN**：比如right left右连接，此时从右边表中读取某个元组，并且找到该元组在左边表中对应的元组或元组集。
4. **ON**：join on实现多表连接查询，**推荐该种方式进行多表查询，不使用子查询**。
5. **GROUP BY**：对满足条件的元组进行分组，一般与聚合函数一起使用。
6. **HAVING**：在以上元组的基础上进行筛选，选出符合条件的元组，一般与GROUP BY进行连用。
7. **SELECT**：从查询到得的所有元组中，获取需要展示的列。
8. **DISTINCT**：对查询得到的所有元组进行去重。
9. **UNION**：将多个查询结果合并，默认去掉重复的记录。
10. **ORDER BY**：对以上的元组结果进行相应的排序。
11. **LIMIT 1**：显示输出一条元组数据记录。

### 3.5. MySQL SQL语句调优？

#### JOIN优化 

##### 用法

![1631013588400](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1631013588400.png)

从左到右，从上到下：

1. **A LEFT JOIN B**：左连接，返回A的全集。
2. **A RIGHT JOIN B**：右连接，返回B的全集。
3. **A INNER JOIN B**：内连接，返回A和B的交集。
4. **A LEFT JOIN B WHERE B.Key IS NULL**：返回A的独有集合。
5. **A RIGHT JOIN B WHERE A.Key IS NULL**：返回B的独有集合。
6. **A FULL OUTER JOIN B**：全连接，返回A和B的并集。
7. **A FULL OUTER JOIN B WHERE A.Key IS NULL OR B.Key IS NULL**：返回A和B独有集合的并集。
8. 另外，还有一种**A CROSS JOIN B**：笛卡尔连接，返回A和B的笛卡尔集，结果行数为 A *  B，如果有ON连接条件，则等于内连接。

##### 原理

联接操作的本质就是，把各个联接表中的记录都取出来，依次匹配的组合会加入结果集并返回给用户。

- 如果没有任何限制条件的话，多表联接起来产生的笛卡尔积可能是非常巨大的。比方说3个100行记录的表联接起来产生的笛卡尔积就有100×100×100=1000000行数据！

- 所以在联接的时候过滤掉特定记录组合是有必要的，在联接查询中的过滤条件可以分成两种，下面以一个JOIN查询为例：

  - **涉及单表的条件**：WHERE条件也可以称为搜索过滤条件，比如t1.m1 > 1是只针对t1表的过滤条件，t2.n2 < ‘d’是只针对t2表的过滤条件。
  - **涉及两表的条件**：比如t1.m1 = t2.m2、t1.n1 > t2.n2等。

  ```sql
  SELECT * FROM t1, t2 WHERE t1.m1 > 1 AND t1.m1 = t2.m2 AND t2.n2 < 'd';
  
  -- 查询结果
  +------+------+------+------+
  | m1   | n1   | m2   | n2   |
  +------+------+------+------+
  |    2 | b    |    2 | b    |
  |    3 | c    |    3 | c    |
  +------+------+------+------+
  ```

  - **联接过程**：
    1. 首先确定第一个需要查询的表，这个表称之为**驱动表**，这里使用t1作为驱动表，那么就需要到t1表中找满足t1.m1 > 1的记录，由于这里没有给t1字段添加索引，所以查询t1表的访问方法为all，也就是采用全表扫描的方式执行单表查询。
    2. 从驱动表t1产生的结果集中的每一条记录，分别需要到t2表中，查找符合过滤条件的记录。由于是根据t1表中的记录去找t2表中的记录，所以t2表也可以被称之为**被驱动表**。比如上一步骤从驱动表中得到了2条记录，所以需要查询2次t2表，此时涉及两个表的列的过滤条件t1.m1 = t2.m2就派上用场了。
    3. 当t1.m1 = 2时，过滤条件t1.m1 = t2.m2就相当于t2.m2 = 2，所以此时t2表相当于有了t1.m1 = 2、t2.n2 < ‘d’这两个过滤条件，然后到t2表中执行**单表查询**。
    4. 当t1.m1 = 3时，过滤条件t1.m1 = t2.m2就相当于t2.m2 = 3，所以此时t2表相当于有了t1.m1 = 3、t2.n2 < ‘d’这两个过滤条件，然后到t2表中执行**单表查询**。

  ![1631149759571](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1631149759571.png)

  - **结论**：
    - 因此，这个两表联接查询共需要查询1次t1表，2次t2表，如果把t1.m1 > 1这个条件去掉，那么从t1表中查出的记录就有3条，就需要查询3次t2表了。
    - 也就是说在两表联接查询中，**驱动表只需要访问一次，被驱动表可能被访问多次**，这种方式在MySQL中有一个专有名词，叫**Nested-Loops Join**（嵌套循环联接）。

##### 算法

###### Nested-Loops Join

- **概念**：
  - 联接算法是，MySQL数据库用于处理联接的物理策略，当联接的表上有索引时，Nested-Loops Join是非常高效的算法。
  - 根据B+树的特性，其联接的时间复杂度为O（logn * logn），若没有索引，则可视为最坏的情况，时间复杂度为O(N²)。
  - MySQL数据库根据不同的使用场合，支持两种Nested-Loops Join算法实现，一种是**Simple Nested-Loops Join（SNLJ）**算法，另一种是**Block Nested-Loops Join（BNLJ）**算法。
- **算法思想**：
  1. **Join阶段**：
     - Join阶段是指，驱动表row scan过滤后的结果集中的每一条记录，需要分别到被驱动表中匹配关联条件的过程。
  2. **Fetch阶段**：
     - Fetch阶段是指，被驱动表根据过滤条件以及匹配的关联条件，进行回表查询数据的过程。
     - 如果关联条件的列是二级索引时，需要再访问主键索引才能得到表中数据（回表），其中MyISAM由于其二级索引存放的是指向记录的指针，所以回表速度要快点；而InnoDB是索引组织表，需要再次通过主键查找才能定位数据。
     - 然而，Fetch阶段也不是必须存在的，如果是覆盖索引联接，那么可以直接得到数据，无需回表，也就没有Fetch这个阶段。

![1631151245915](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1631151245915.png)

- **性能评估**：评判一个Join算法是否优劣，需要**Join开销成本**是否比较小、**I/O访问方式**是顺序还是随机等。

  | 开销统计                      | 含义                                            |
  | ----------------------------- | ----------------------------------------------- |
  | 驱动表的扫描次数（O）         | 通常都是1，即Join时扫描一次驱动表的数据即可     |
  | 被驱动表的扫描次数（I）       | 使用不同的Join算法，该值可能会不同              |
  | 联接过程读取的总记录数（R）   | 使用不同的Join算法，该值可能会不同              |
  | Join时的比较次数（M）         | 使用不同的Join算法，该值可能会不同              |
  | 被驱动表回表读取的记录数（F） | 如果Fetch为非覆盖索引，被驱动表可能需要回表查询 |

###### Simple Nested-Loops Join

Simple Nested-Loops Join，SNLJ，**简单直接的嵌套循环联接**，即驱动表中的**每一条记录**与被驱动表中的记录都需要进行比较判断，对于两表联接来说，驱动表只会被访问一遍，但被驱动表却要被访问到好多遍，具体访问几遍取决于对驱动表执行单表查询后的结果集中的记录条数。

- **执行过程**：

  ```sql
  For each row r in R do                         -- 扫描R表（驱动表）
      For each row s in S do                     -- 扫描S表（被驱动表）,全表扫描
          If r and s satisfy the join condition  -- 如果r和s满足join条件
              Then output the tuple <r, s>       -- 返回结果集
  ```

  ![1631152538991](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1631152538991.png)

- **优点**：**算法最简单**。

- **缺点**：由于联接过程需要多次全表扫描被驱动表，所以**性能开销最大**。

  - 联接过程读取的总记录数的成本，和Join时的比较次数的成本都是SN*RN，也就是笛卡儿积。假设外表内表都是1万条记录，那么其读取的记录数量和Join的比较次数都需要上亿，因此实际上MySQL并不会使用到SNLJ算法。

- **性能评估**：记关联过程中，驱动表读取的记录数为RN，被驱动表读取的记录数为SN，则其开销统计为：

  | 开销统计                      | SNLJ         |
  | ----------------------------- | ------------ |
  | 驱动表的扫描次数（O）         | 1            |
  | 被驱动表的扫描次数（I）       | RN           |
  | 联接过程读取的总记录数（R）   | RN + RN * SN |
  | Join时的比较次数（M）         | RN * SN      |
  | 被驱动表回表读取的记录数（F） | 0            |

###### Index Nested-Loops Join

Index Nested-Loops Join，INLJ，**基于索引的嵌套循环联接**，为了降低SNLJ联接过程中多次全表扫描被驱动表的成本开销，可以在被驱动表中建立索引，减少被驱动表读取的记录数，其中MySQL中使用较多的是这种算法。

- **执行过程**：

  ```sql
  For each row r in R do                     -- 扫描R表
      lookup s in S index                    -- 查询S表的索引（固定3~4次IO，B+树高度）
          If find s == r                     -- 如果r匹配了索引s
              Then output the tuple <r, s>   -- 返回结果集
  ```

  ![1631157411686](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1631157411686.png)

- **优点**：由于内表上有索引，所以Join比较时不再需要一条条记录进行比较，而可以通过索引来减少比较次数，从而加快查询速度。

  - 由于驱动表中每条记录会通过被驱动表的索引进行**二分查找匹配**，所以每次被驱动表的比较次数为**索引树的高度**。
  - 而一般B+树的高度为3~4层，也就是说匹配一次的I/O消耗也就3~4次，所以索引查询的成本是比较固定的，因此MySQL优化器会倾向于**使用记录数少的表作为驱动表**。

- **缺点**：

  - 如果被驱动表进行Join时，关联列使用的是**主键索引**，在大数据量的情况下，由于主键索引查找的开销非常小，且此时的访问模式也是比较顺序的，INLJ的效率也是相当不错的。
  - 如果被驱动表进行Join时，关联列使用的是**非覆盖的二级索引**，在大数据量的情况下，由于访问的是非覆盖的二级索引，需要通过主键索引进行回表查询，此时会产生**大量的随机I/O**，对比顺序I/O性能低下。

- **性能评估**：记关联过程中，驱动表读取的记录数为RN，被驱动表读取的记录数为SN，则其开销统计为：

  | 开销统计                      | SNLJ         | INLJ                         |
  | ----------------------------- | ------------ | ---------------------------- |
  | 驱动表的扫描次数（O）         | 1            | 1                            |
  | 被驱动表的扫描次数（I）       | RN           | RN                           |
  | 联接过程读取的总记录数（R）   | RN + RN * SN | RN + IndexMatches            |
  | Join时的比较次数（M）         | RN * SN      | RN * IndexHeight             |
  | 被驱动表回表读取的记录数（F） | 0            | IndexMatches（非覆盖索引时） |

###### Block Nested-Loops Join

Block Nested-Loops Join，BNLJ，**基于块的嵌套循环联接**，为了减少SNLJ联接过程中访问被驱动表的次数，使用一次性把多条驱动表中的记录去和被驱动表做匹配，可以大大减少重复从磁盘上加载被驱动表的代价。

- **Join Buffer使用原则**：
  - 每次联接使用一个Join Buffer，因此多表的联接可以使用多个Join Buffer。
  - Join Buffer在联接发生之前进行分配，在SQL语句执行完后进行释放。
  - Join Buffer只存储要进行查询操作的相关列数据，而不是整行的记录。
  - Join Buffer可被用于被驱动表的关联列为**ALL、index、和range**类型的联接查询。
  - 系统变量**Join_buffer_size**决定了Join Buffer的大小。
    - 当MySQL的Join有使用到Block Nested-Loop Join，调大后可以避免多次的内表扫描，从而提高性能；如果是Index Nested-Loop Join使用索引进行Join，那么调大这个变量则毫无意义。
    - Join_buffer_size默认值是256K，显然对于稍复杂的SQL是不够用的。
    - 建议在会话级别进行设置，但如果设置不好，则会容易导致因无法分配内存而宕机的问题。

- **执行过程**：

  ```sql
  For each tuple r in R do                             -- 扫描外表R
      store used columns as p from R in Join Buffer    -- 将部分或者全部R的记录保存到Join Buffer中，记为p
      For each tuple s in S do                         -- 扫描内表S
          If p and s satisfy the join condition        -- p与s满足join条件
              Then output the tuple                    -- 返回为结果集
  ```

  ![1631159874694](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1631159874694.png)

- **优点**：
  - BNLJ算法较SNLJ算法的改进就在于，可以**减少被驱动表的扫描次数**，甚至可以和Hash Join算法一样，仅需扫描内表一次。
  - 可以使用增大**Join Buffer**（联接缓冲），或者只需要把驱动表关心的列放到查询列表，以支持一次性匹配更多的驱动表记录，来减少循环读取被驱动表的次数，从而提升Join的性能。

- **缺点**：仍然有可能会多次全表扫描被驱动表，占用磁盘 IO 资源。

- **性能评估**：记关联过程中，驱动表读取的记录数为RN，被驱动表读取的记录数为SN，则其开销统计为：

  | 开销统计                      | SNLJ         | INLJ                         | BNLJ                                      |
  | ----------------------------- | ------------ | ---------------------------- | ----------------------------------------- |
  | 驱动表的扫描次数（O）         | 1            | 1                            | 1                                         |
  | 被驱动表的扫描次数（I）       | RN           | RN                           | RN * used_col_size / join_buffer_size + 1 |
  | 联接过程读取的总记录数（R）   | RN + RN * SN | RN + IndexMatches            | RN + SN * I                               |
  | Join时的比较次数（M）         | RN * SN      | RN * IndexHeight             | RN * SN                                   |
  | 被驱动表回表读取的记录数（F） | 0            | IndexMatches（非覆盖索引时） | 0                                         |

- **算法使用标记**：在BNLJ算法使用后，在SQL Explain#Extra中会提示 Using join buffer (Block Nested Loop)。

  - 在有索引的情况下，MySQL会尝试去使用Index Nested-Loop Join算法。
  - 但在Join列没有索引的情况下，MySQL不会去使用最简单的Simple Nested-Loop Join算法，而是使用Block Nested-Loop Join算法。

###### Batched Key Access Join

Batched Key Access Join，BKA，**批量键访问联接**，为了优化INLJ存在的大量随机I/O操作，在MySQL 5.6开始支持BKA算法，它是通过常见的空间换时间方式，将随机I/O转换为顺序I/O，以此来极大地提升Join的性能。

- **MRR**：Multi Range Read，**多范围读取**，MySQL 5.6的新特性，是BKA的重要支柱，目的是**为了减少磁盘的随机访问**。

  1. InnoDB由于索引组织表的特性，如果查询是使用非覆盖二级索引，则需要回表读取数据做后续处理，虽然二级索引是有序的，但对应的主键索引很可能是无效的，因此该程会随机的回表，并伴随着大量的随机I/O。

  2. 而MRR的优化在于，并不是每次都会通过二级索引直接回表读取记录，而是在范围扫描（Range Access）中，MySQL将扫描到的数据存入由**read_rnd_buffer_size**变量定义的内存大小中，默认256K。

  3. 然后对其按照**Primary Key（RowID）**排序，然后使用排序好的数据进行**顺序回表**，由于InnoDB中叶子节点数据，是按照PRIMARY KEY（ROWID）进行顺序排列的，所以可以认为，如果按照主键的递增顺序查询的话，对磁盘的读比较接近顺序读，是能够提升读性能的，为SQL查询语句带来极大的性能提升。

  4. MRR能够提升性能的核心在于，这条查询语句在索引上做的是一个**范围查询**，可以得到足够多的主键id。这样通过排序以后，再去主键索引查数据，才能体现出**顺序性**的优势，所以MRR优化可用于被驱动表关联列为**range，ref，eq_ref**类型的查询。

  5. **开启mrr的参数**：optimizer_switch#mrr和optimizer_switch#mrr_cost_based选项。
     - **optimizer_switch#mrr选项**：表示是否开启MRR优化，默认为on。
     - **optimizer_switch#mrr_cost_based**：表示通过基于成本的算法，来确定是否需要开启MRR特性，默认为off。然而，在MySQL当前版本中，基于成本的算法过于保守，导致大部分情况下优化器都不会选择MRR特性。因为如果强制开启MRR，由于MRR需要排序，在某些SQL语句下，假如排序的时间超过直接扫描的时间，那么性能可能会变差。

     ```sql
     set optimizer_switch='mrr=on,mrr_cost_based=off';
     ```

     

  ![1631178784100](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1631178784100.png)

- **执行过程**：

  1. 将驱动表中相关的列放入Join Buffer中。
  2. 批量地将Key（索引键值）发送到MRR接口。
  3. MRR接口通过收到的Key，**根据其对应的主键ID进行排序**，然后再进行数据的读取操作。
  4. 最后返回结果集给客户端。

  ![1631176759460](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1631176759460.png)

- **优点**：

  - 与INLJ不同的地方在于，BKA不是从驱动表一行一行地取出 Join条件值，再到被驱动表去做Join，而是从驱动表里**一次性拿出多行存到join_buffer**，然后再一起传给被驱动表。
  - 而这个join_buffer，在BNLJ算法里的作用是暂存驱动表的数据，但在NLJ算法里并没有用，而此时刚好可以**复用join_buffer**到BKA算法中。
  - 如果扫描被驱动表的是主键，那么被驱动表中的记录访问都是比较有序的；如果扫描被驱动表的是非覆盖二级索引，那么对于被驱动表中记录的访问可能就是非常离散的。
  - 因此，对于非覆盖二级索引的join联接，BKA算法会调用MRR接口，通过**对索引对应的主键ID进行排序**，使得该索引能够以**顺序的I/O**读取数据，而不是以随机I/O读取，从而提高Join的执行效率。

- **缺点**：

  - 由于BKA算法本质上是通过MRR接口，将非覆盖二级索引对于记录的访问，转化为根据主键ID排序的较为有序地获取记录，所以要想通过BKA算法来提高性能，需要确保联接列为被驱动表的**非覆盖二级索引**。

  - **BKA参数**：optimizer_switch#batched_key_access选项。

    - 由于BKA使用MRR，因此MRR标志也必须打开，但目前MRR的成本估算过于悲观，必须关闭mrr_cost_based才能使用BKA。

    ```sql
    SET optimizer_switch='mrr=on,mrr_cost_based=off,batched_key_access=on';
    ```

- **性能评估**：记关联过程中，驱动表读取的记录数为RN，被驱动表读取的记录数为SN，则其开销统计为：

  | 开销统计                      | SNLJ         | INLJ/BKA                     | BNLJ                                      |
  | ----------------------------- | ------------ | ---------------------------- | ----------------------------------------- |
  | 驱动表的扫描次数（O）         | 1            | 1                            | 1                                         |
  | 被驱动表的扫描次数（I）       | RN           | RN                           | RN * used_col_size / join_buffer_size + 1 |
  | 联接过程读取的总记录数（R）   | RN + RN * SN | RN + IndexMatches            | RN + SN * I                               |
  | Join时的比较次数（M）         | RN * SN      | RN * IndexHeight             | RN * SN                                   |
  | 被驱动表回表读取的记录数（F） | 0            | IndexMatches（非覆盖索引时） | 0                                         |

- **算法使用标记**：EXPLAIN#Extra值包含Using join buffer（Batched Key Access），且类型值为**ref或eq_ref**时，表示使用了BKA。

###### Classic Hash Join

Classic Hash Join，CHJ，Hash Join不需要任何索引，而是在Join Buffer中**创建散列表**，然后被驱动表通过哈希算法进行查找，使得能够在BNLJ算法的基础上，**进一步减少了被驱动表的比较次数**，从而提升JOIN的查询性能。

- **执行过程**：
  1. build阶段：CHJ的第一个阶段，先将驱动表中的数据放入Join Buffer中，然后根据键值产生一张散列表。
  2. probe阶段：CHJ的第二个阶段，随后读取被驱动表中的一条记录，对其应用散列函数，将其和散列表中的数据进行比较。

![1631179295488](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1631179295488.png)

- **优点**：

  - 如果Join Buffer能够缓存所有驱动表的查询列，那么驱动表和被驱动表的扫描次数都将只有1次，并且比较的次数也只是被驱动表的记录数（假设哈希算法冲突为0时）；反之则需要扫描多次内部表。因此，为了使Classic Hash Join更有效果，应该规划好Join Buffer的大小。

  - 要使用Classic Hash Join算法，需要将join_cache_level设置为大于等于4的值，并显示地打开优化器的选项，设置过程如下：

    ```sql
    set join_cache_join=4;
    set optimizer_switch='join_cache_hashed=on';
    ```

- **缺点**：

  - Classic Hash Join算法虽好，但是仅能用于**等值联接**，对于非等值联接的JOIN查询，就会显得无能为力了。
  - **创建散列表也是费时的工作**，不过一旦建立完成后，就能大幅提升JOIN的速度，因此，在通常情况下，大表之间的JOIN，Hash Join算法会比较有优势；小表通过索引查询，利用BKA Join就已经能很好的完成查询。
  - 此外，Hash Join需要在MySQL 8.0.18才会有支持，业界中使用该版本的公司还是比较少的，所以在生产应用得可能并不多。

- **性能评估**：记关联过程中，驱动表读取的记录数为RN，被驱动表读取的记录数为SN，则其开销统计为：

  | 开销统计                      | SNLJ         | INLJ/BKA                     | BNLJ                                      | CHJ                                       |
  | ----------------------------- | ------------ | ---------------------------- | ----------------------------------------- | ----------------------------------------- |
  | 驱动表的扫描次数（O）         | 1            | 1                            | 1                                         | 1                                         |
  | 被驱动表的扫描次数（I）       | RN           | RN                           | RN * used_col_size / join_buffer_size + 1 | RN * used_col_size / join_buffer_size + 1 |
  | 联接过程读取的总记录数（R）   | RN + RN * SN | RN + IndexMatches            | RN + SN * I                               | RN + SN * I                               |
  | Join时的比较次数（M）         | RN * SN      | RN * IndexHeight             | RN * SN                                   | SN / I                                    |
  | 被驱动表回表读取的记录数（F） | 0            | IndexMatches（非覆盖索引时） | 0                                         | 0                                         |

- **算法使用标记**：如果将Hash Join应用于Simple Nested-Loops Join中，则Explain#Extra列会显示BNLH；如果将Hash Join应用于Batched Key Access Join中，则Explain#Extra列会显示BKAH。

##### 优化方案

根据JOIN原理和每种JOIN的算法分析可得出，JOIN联接查询的成本开销占大头的是，**驱动表记录数 * 单次访问被驱动表的成本**，因此优化重点应该为这两个部分：

- **尽量减少驱动表的记录数**。
  - **小表驱动大表**：一般无需人工考虑，MySQL优化器会自动选择最优的执行方式，另外可以使用STRAIGHT_JOIN可以指定左右表顺序，使其不被优化器优化。
  - **尽量使用过滤条件先减少驱动表的记录数**：如果有where条件，应当要能够使用索引，并尽可能地减少驱动表的数据量。
- **对被驱动表的访问成本尽可能降低**。
  - **尽量在被驱动表的联接列上建立索引**：主键、唯一索引最优，其次是非唯一的二级索引，这样就可以使用eq_ref或ref的索引匹配类型来访问被驱动表，从而降低访问被驱动表的成本。
    - 注意，join字段的类型需要保持一致，以**避免索引失效**。
  - **合理设置join_buffer大小**：如果被驱动表的join字段用不了索引，且内存较为充足，可以考虑把join_buffer设置得大一些，以**减少被驱动表的扫描次数**，提高查询性能。
  - **尽量减少联接过程读取的总记录数**：经验之谈，如果能控制扫描驱动表和被驱动表的行数在**百万条以内**，效率还是可以接受的。
- **参与Join的表不要太多**：
  - 对于NLJ的算法实现，参与Join的驱动表越多，**循环嵌套也就越多**，导致访问被驱动表的次数也越多，性能自然就不会太高。
  - 再者，参与Join的表过多，会导致SQL变得复杂和臃肿，**不利于后期调优和维护**。
  - 在阿里变成规约中建议，**参与Join的表不能超过3张**，如果业务确实需要关联这么多表，可以抽取到应用层去实现，避免在SQL中Join过多的表。

#### GROUP BY优化

##### 执行过程分析

1. 如果group by和select字段都建立了索引，但不做任何优化，则会先扫描emp_no索引树上的每一个索引键。
2. 确定好emp_no后，再去扫描salary索引树的上每一个索引键，然后得出当前emp_no最小的salary。
3. 最后遍历每个emp_no的salary，得出最小的salary并返回。
4. 其中，如果group by和select字段没建立索引，则扫描的不是索引树，而是全表扫描。

```sql
/*
 * 分析这条SQL如何执行：
 * [emp_no, salary] 组合索引
 * [10001,50000]
 * [10001,51000]
 * ...
 * [10002,30000]
 * [10002,32000]
 * ...
 * 1. 先扫描emp_no = 10001的数据，并计算出最小的salary是多少，[10001,50000]
 * 2. 扫描emp_no = 10002，并计算出最小的salary是多少，[10002,30000]
 * 3. 遍历出每个员工的最小薪资，并返回
 */
 -- 查询每个员工拿到过的最少的工资是多少[比惨大会]
-- CREATE INDEX salaries_emp_no_salary_index ON salaries (emp_no, salary);
explain
select emp_no, min(salary)
from salaries
group by emp_no
```

##### 扫描模式

###### Loose Index Scan（松散索引扫描）

松散索引扫描，**无需扫描所有满足条件的索引键**，即可返回结果。

- **执行过程**：

  1. 在确定好emp_no后，由于此时salary索引是有序的，所以第一个salary正式当前emp_no最小的salary，因此，会直接返回第一个salary，跳过后面的salary索引树扫描。
  2. 接着继续遍历下一个emp_no，同样也是取第一个salary...
  3. 最后遍历每个emp_no的salary，得出最小的salary并返回。

- **特点**：

  - **性能最好**：由于索引树是有序的，获取MIN（）时，只需返回第一个索引即可，无需扫描所有满足条件的索引键，**性能最好**。
  - **标志**：Explain#Extra会显示Using index for group-by。

- **使用条件**：

  1. **查询作用在单张表上**：
     - 因为如果作用在多张表上，可能需要扫描整个索引树。
  2. **GROUP BY关键字都要符合最左前缀原则**：
     - 因为如果不符合最左前缀原则，会导致复合索引的索引失效，此时可能会全表扫描。
  3. **聚合函数只支持MIN（）和MAX（）**：
     - 由于其他聚合函数需要扫描所有索引键，因此只支持MIN（）和MAX（）。
     - 且如果MIN（）和MAX（）同时在一条SQL中使用，则必须作用在同一个字段，才能走松散索引扫描。
     - 且MIN（）和MAX（）的字段必须也符合最左前缀原则，保证索引不失效。
     - 另外，当查询中不存在GROUP BY和DISTINCT语句时，AVG（DISTINCT 单个参数）、SUM（DISTINCT 单个参数）、COUNT（DISTINCT 多个参数） 也可以使用松散索引扫描。
  4. **SELECT字段必须为GROUP BY关键字或者常量**：
     - 因为如果SELECT字段为非GROUP BY关键字、非常量，可能会导致回表查询（最左前缀索引等值查询除外），不符合松散索引扫描原则。
  5. **索引不能为前缀索引**：
     - 因为使用前缀索引无法让GROUP BY正常工作。

- **适用场景举例**：假设index（c1，c2，c3）作用在t1（c1，c2，c3，c4）表上。

  ```sql
  -- 符合最左前缀原则
  SELECT c1，c2 FROM t1 GROUP BY c1，c2；
  
  -- 等价与GROUP BY c1，c2，只不过DISTINCT取的是分组后的一条数据
  SELECT DISTINCT c1，c2 FROM t1；
  
  -- 聚合函数 + DISTINCT + 非GROUP BY和DISTINCT条件时，也可以走松散索引扫描
  SELECT COUNT(DISTINCT c1)，SUM(DISTINCT) FROM t1；
  SELECT COUNT(DISTINCT c1，c2)，COUNT(DISTINCT c2，c1) FROM t1；
  
  -- 符合最左前缀原则 + MIN（）
  SELECT c1，MIN（c2） FROM t1 GROUP BY c1；
  
  -- 范围查询 + 符合最左前缀原则
  SELECT c1，c2 FROM t1 WHERE c1 < const GROUP BY c1，c2；
  
  -- 范围查询 + 符合最左前缀原则
  SELECT c2 FROM t1 WHERE c1 < const GROUP BY c1，c2；
  
  -- 范围查询 + 符合最左前缀原则 + MIN（）& MAX（）作用在同一个字段上
  SELECT MAX（c3），MIN（c3），c1，c2 FROM t1 WHERE c2 > const GROUP BY c1，c2；
  
  -- 符合最左前缀原则 + 最左前缀索引等值查询
  SELECT c1，c2 FROM t1 WHERE c3 = const GROUP BY c1，c2；
  ```

- **不适用场景举例**：假设index（c1，c2，c3）作用在t1（c1，c2，c3，c4）表上。

  ```sql
  -- 聚合函数不是MIN（）或MAX（）
  SELECT c1，SUM（c2） FROM t1 GROUP BY c1；
  
  -- 不符合最左前缀原则
  SELECT c1，c2 FROM t1 GROUP BY c2，c3；
  
  -- SELECT查询了非GROUP BY关键字、非常量，且c3最左前缀索引没作等值查询
  SELECT c1，c3 FROM t1 GROUP BY c1，c2；
  -- => 改成下面语句，则可以走松散索引扫描
  -- 符合最左前缀原则 + 最左前缀索引等值查询
  -- SELECT c1，c2 FROM t1 WHERE c3 = const GROUP BY c1，c2；
  ```

###### Tight index Scan（紧凑索引扫描）

紧凑索引扫描，**需要扫描所有满足条件的索引键**，才可返回结果，如果一条SQL无法使用松散索引扫描，则会尝试使用紧凑索引扫描。

- **特点**：

  - **性能次好**：由于需要扫描整个索引树，因此，性能会比松散索引扫描差一些，不过还是可以接受的。
  - **标志**：Explain#Extra会显示Using index，表示覆盖索引，扫描了整个索引树。

- **适用场景举例**：

  ```sql
  -- 紧凑索引扫描，虽然emp_no和salary有索引，但使用了SUM聚合函数，需要遍历整个salary索引树
  -- CREATE INDEX salaries_emp_no_salary_index ON salaries (emp_no, salary);
  explain
  select emp_no, sum(salary)
  from salaries
  group by emp_no;
  ```

###### Temporary table（临时表）

当紧凑索引扫描也无法使用的话，MySQL会读取需要的数据，**创建一个临时表**，用临时表实现GROUP BY操作。

- **特点**：
  - **性能最差**：因为走了全表扫描以及创建了一个临时表。
  - **标志**：Explain#Extra会显示Using temporary，表示使用了临时表。

- **场景举例**：

  ```sql
  -- 出现临时表，hire_date没有索引
  explain
  select max(hire_date)
  from employees
  group by hire_date;
  ```

###### 扫描模式总结

| 扫描模式                         | 使用规则                             | Extra标识                | 性能 |
| -------------------------------- | ------------------------------------ | ------------------------ | ---- |
| Loose Index Scan（松散索引扫描） | 优先尝试使用                         | Using index for group-by | 最好 |
| Tight index Scan（紧凑索引扫描） | 使用不了松散索引扫描时，才会尝试使用 | Using index              | 次好 |
| Temporary table（临时表）        | 使用不了紧凑索引扫描时，才会使用     | Using temporary          | 最差 |

##### 优化方案

如果GROUP BY使用了临时表，则要想办法**走上索引**，即用上松散索引扫描，或者紧凑索引扫描，尽量避免临时表。

#### DISTINCT优化

- **原理**：DSITINCT本质上是在GROUP BY操作之后，**每组只取1条数据**。
- **优化方案**：和GROUP BY思路一样，如果使用了临时表，则要想办法**走上索引**，即用上松散索引扫描，或者紧凑索引扫描，尽量避免临时表。

#### ORDER BY优化

##### 索引使用规律

###### 全表扫描 VS 索引排序

|          | 场景                                | 标志                                   |
| -------- | ----------------------------------- | -------------------------------------- |
| 全表扫描 | 当MySQL优化器发现全表扫描开销更低时 | 此时Explain#Extra会显示Using File Sort |
| 索引排序 | 当MySQL优化器发现走索引开销更低时   | 此时Explain#Extra没有Using File Sort   |

###### 单列索引+范围查询

- **排序情况**：可以使用索引避免排序。
- **原理**：单列索引first_name是有序的，即使做了范围查询，也是可以利用索引避免排序的。

```sql
explain
select *
from employees
where first_name < 'Bader'
order by first_name;
```

###### 联合索引+等值查询

- **排序情况**：可以使用索引避免排序。

- **原理**：对于联合索引（first_name，last_name），first_name等值结果中的last_name是有序的，因此可以利用索引的有序性。

```sql
explain
select *
from employees
where first_name = 'Bader'
order by last_name;
```

###### 联合索引+范围查询

- **排序情况**：无法利用索引避免排序。
- **原理**：对于联合索引（first_name，last_name），first_name非等值结果集中，last_name是无序的，因此无法利用索引避免排序。

```sql
explain
select *
from employees
where first_name < 'Bader'
order by first_name;
```

###### 升降序不一致

- **排序情况**：无法利用索引避免排序。
- **原理**：对于联合索引（first_name，last_name），first_name等值结果中的last_name只是顺序的，如果再对last_name做反向排序，则利用不了last_name的有序性，因此无法利用索引避免排序。

```sql
explain
select *
from employees
order by first_name desc, last_name asc
limit 10;
```

###### 排序字段存在多个索引中

- **排序情况**：无法利用索引避免排序。
- **原理**：
  1. 虽然first_name和emp_no分别是普通索引和主键索引，它们都是有序的。
  2. 但如果先对first_name做排序再对主键做排序，由于普通索引是非聚蔟索引，会先去找主键，然后把找到的主键集做排序。
  3. 而在这种情况下，是不能保证这些主键集的有序性的，因此无法利用索引避免排序。

```sql
-- emp_no为主键
explain
select *
from employees
order by first_name, emp_no
limit 10;
```

##### 排序模式

Using File Sort，共有三种排序模式：

###### rowid排序（常规排序）

- **执行过程**：
  1. 从表中获取满足WHERE条件的记录。
  2. 对于每条记录，将记录的主键及排序键（id，order_column）取出来，放入sort buffer（排序缓存，由**sort_buffer_size**控制）。
  3. 如果sort  buffer能存放所有满足条件的（id，order_column），则直接在sort buffer内存中进行排序；否则，会进行多次sort buffer排序，并在每次在sort buffer满后，把排序后的结果写到**临时文件**中。
     - 这里sort buffer排序算法用的是，**快速排序**算法，O（n * logn）。
  4. 如果sort buffer排序后，产生了临时文件，则还需要对其使用**归并排序算法**，来保证记录是有序的。
  5. 循环执行上述过程，直到所有满足条件的路基全部参与排序。
  6. 扫描排好序的（id，order_column），并**使用id**去获得SELECT语句中其他需要返回的字段。
  7. 返回结果集。
- **特点**：
  - **可能会产生临时文件**：需要看sort buffer能否存放WHERE里面的所有（id，order_column），如果不满足，则会产生临时文件。
  - **一次排序需要两次I/O**：
    1. 第一次I/O发生在第2步，将（id，order_column）读取出来放入sort buffer中，且如果sort buffer满后，还会写出到临时文件中
    2. 第二次I/O发生在第6步，当（id，order_column）排序好后，需要根据主键ID获取其他字段，由于其结果是按照order_column进行排序的，只能保证order_column是顺序的，并不能保证主键ID的有序性，因此会存在随机I/O的问题。
       - **解决方案**：MySQL内部针对这种情况做了优化，在使用主键ID去获取数据之前，先对主键ID排好序并放入一个缓存里面，其大小由**read_md_buffer_size**控制（即MRR接口），默认256K，接着再去获取记录，从而把随机I/O转换为顺序I/O。

###### additional_fields排序（全字段排序）

- **执行过程**：
  1. 与rowid排序（常规排序）执行过程类似，不过排序的字段不仅仅只有（id，order_column），而是该SQL中**所有需要的字段都**会放入到sort buffer中参与排序。
  2. 由于sort buffer已经包含了查询需要的所有字段，因此，在sort buffer中排序完成后，即可直接返回。
- **优点**：排序完成后即可直接返回，无需两次I/O，从而获得了性能的提升。
- **缺点**：
  - 由于所有字段都参与排序，所以一行数据占用的空间会比rowid排序的多。
  - 如果sort buffer设置的比较小（默认256K），由于排序的字段多，在排序时会容易占满sort buffer，从而导致产生临时文件，而发生了写临时文件+归并排序，性能下降。
- **使用场景**：当order by中出现的**字段总长度小于max_length_for_sort_data**时，MySQL会使用全字段排序；否则，MySQL会使用rowid排序。

###### packed_additional_fields排序（打包字段排序）

- **执行过程**：
  1. 与全字段排序的工作原理一样，也是将所有需要的字段都放入到sort buffer中参与排序。
  2. 不同的地方在于，打包字段排序，会将**字段紧密地排列**在一起，而不是使用固定长度空间（字段总长度）。
     - 比如VARCHAR（255）"yes"字段，使用全字段排序会占用255字节的sort buffer，而使用打包字段排序则只占用2+3字节的sort buffer。
- **优点**：由于字段会紧密排列，一个sort buffer可以容纳下更多的字节，从而减少写临时文件的发生，减少性能下降的概率。

###### 排序与OPTIMIZER_TRACE

```json
    {
      # 3. 执行阶段的执行过程
      "join_execution": {
        "select#": 1,
        "steps": [
          {
            # 排序后特有的内容
            "sorting_table": "employees",
            "filesort_information": [
              {
                "direction": "asc",
                "expression": "`employees`.`last_name`"
              }
            ] /* filesort_information */,
            "filesort_priority_queue_optimization": {
              "limit": 502,
              "chosen": true
            } /* filesort_priority_queue_optimization */,
            "filesort_execution": [
            ] /* filesort_execution */,
            # 重点关注 filesort_summary
            "filesort_summary": {
              # 可用内存, 其实就是sort_buffer_size => 默认256k
              "memory_available": 262144,
              "key_size": 264,
              "row_size": 401,
              "max_rows_per_buffer": 503,
              "num_rows_estimate": 45208,
              # 本次排序一共参与排序的行数
              "num_rows_found": 22287,
              # 本次排序产生了几个临时文件, 0则代表是完全基于内存排序
              "num_initial_chunks_spilled_to_disk": 0,
              "peak_memory_used": 205727,
              "sort_algorithm": "std::sort",
              "unpacked_addon_fields": "using_priority_queue",
              # 使用的排序模式: 这里全字段排序 => rowid、additional_fields、packed_additional_fields
              "sort_mode": "<varlen_sort_key, additional_fields>"
            } /* filesort_summary */
          }
        ] /* steps */
      } /* join_execution */
    }
```

###### 排序模式总结

| 变量                     | 作用                                                         |
| ------------------------ | ------------------------------------------------------------ |
| sort_buffer_size         | 指定sort_buffer的大小                                        |
| read_rnd_buffer_size     | 默认256K，MRR接口主键ID排序后的缓存区大小                    |
| max_sort_length          | 指定排序时，排序字段最多取多少字节                           |
| max_length_for_sort_data | 当order by中出现的**字段总长度小于max_length_for_sort_data**时，MySQL会使用全字段排序；否则，MySQL会使用rowid排序。 |

##### 优化方案

- **使用索引**：最好的做法是，利用索引的有序性，让MySQL跳过filesort排序过程，以避免排序。
- **优化filesort**：如果发生了filesort，并且没有办法避免时，则需要想办法优化filesort。
   - **调大sort_buffer_size**：设置较大的sort buffer，可以减少甚至避免临时文件的产生，从而减少归并操作的次数，或者避免归并操作的发生。
     - 当OPTIMIZER_TRACE#filesort_summary#**num_initial_chunks_spilled_to_disk**过大时，说明产生的临时文件过多，归并次数也就越多，此时可以调大sort buffer，以减少临时文件的产生，提升性能。
   - **调大read_rnd_buffer_size**：设置较大的MRR排序缓存区，可以接收更多来自MRR接口排序好的主键ID，从而让一次顺序I/O返回更多的结果。
   - **调小max_sort_length**：如果排序字段超长，减少取值的字段长度，可以减少sort buffer的占用，减少甚至避免临时文件的产生，从而减少归并操作的次数，或者避免归并操作的发生。
   - **设置合理的max_length_for_sort_data**：一般不建议随意调整。
     - 如果设置得过大，MySQL则会认为有足够的缓存容纳全部字段，使得各种排序SQL都走上全字段排序，从而导致**大量的内存占用**，当还发生写临时文件时，又会**占用大量的硬盘**。
     - 如果设置得太小，MySQL则会认为没有足够的缓存容纳全部字段，使得各种排序SQL都走上rowid排序，由于rowid排序需要走上两次I/O，并且会有随机I/O的发生，所以导致**各种排序SQL性能都比较低下**。

#### LIMIT优化

##### 用法

limit offset, size：

- **offset**：返回结果第一行的偏移量，即想要跳过多少行。
- **size**：指定返回多少条记录。

##### 优化需求

当offset过大时，会导致MySQL先扫描offset条行数据，从而可能使MySQL走向**全表扫描**。

```sql
-- 查询第30001页的时候，花费174ms
explain
select *
from employees
limit 300000,10;
```

##### 优化方案

###### 使用覆盖索引

可在只返回索引字段的场合下，让全表扫描ALL提升到**全索引树扫描Index**。

- **缺点**：只能返回单个字段。

```sql
-- 方案1：覆盖索引 (108ms)
explain
select emp_no
from employees
limit 300000,10;
```

###### 使用覆盖索引+Join

这种方式会先走一次方案1，不过由于方案一为全索引数扫描Index，满足了BNLJ的使用场景，因此可以使用覆盖索引 + Join的方式，来获取深度分页的**全字段查询结果**。

- **分析**：此时联接算法为**BNLJ**，由于t比较小，MySQL会使用t为驱动表，然后把t的查询结果存进join_buffer，接着e表一次性将join_buffer的结果进行匹配，由于联接字段是e的覆盖索引，可以在索引树上直接返回，不会产生大量的随机I/O，因此这种优化方式还是可以接受的。

```sql
-- 方案2：覆盖索引+join(109ms)
select *
from employees e
         inner join
     (select emp_no from employees limit 300000,10) t
     using (emp_no);
```

###### 覆盖索引+子查询

这种方式也是先需要走一次方案1，先查询达到深度分页最小的emp_no，再利用索引的有序性，往后再找10条记录。

- **分析**：利用了索引的有序性。

```sql
-- 方案3：覆盖索引+子查询（126ms）
select *
from employees
where emp_no >=
      (select emp_no from employees limit 300000,1)
limit 10;
```

###### 范围查询

- **优点**：扫描的行数永远只有10行，性能次优。
- **缺点**：需要先获取上一页最大的emp_no。

```sql
select *
from employees
where emp_no > #{last_max_emp_no}
limit 10;
```

###### 起始主键值 + 结束主键值

- **优点**：扫描的行数永远只有10行，且只在主键索引上查询，性能最优。
- **缺点**：需要先获取上一页最后一行的主键值。

```sql
-- 方案5：如果能获得起始主键值 & 结束主键值
select *
from employees
where emp_no between 20000 and 20010;
```

###### 禁止传入过大的页码

从业务的角度解决深度分页问题，比如百度搜索最多只显示76页的结果。

- **缺点**：需要业务的妥协。

#### COUNT优化

##### MySQL#count（？）区别

如果没有特殊要求，一般建议使用count（*）就好。

| 形式              | 性能上的区别                                                 | 业务上的区别                                   |
| ----------------- | ------------------------------------------------------------ | ---------------------------------------------- |
| count（业务字段） | 如果该字段存在索引，则会走上索引；否则只能全表扫描           | 只会对该字段进行统计，会排除字段值为NULL的行   |
| count（*）        | 与count（1）没有区别，会优先选择最小字段长度的非主键索引，其次才是主键；InnDB >= 8.0.13，对于无条件count（*）做了优化 | 与count（1）没有区别，不会排除字段值为NULL的行 |
| count（1）        | 与count（*）没有区别，会优先选择最小字段长度的非主键索引，其次才是主键 | 与count（*）没有区别，不会排除字段值为NULL的行 |

##### MySQL#count（*）索引选择规律

1. 当没有非主键索引时，会使用主键索引。eg：PRIMARY => ken_len: 4。
2. 如果存在非主键索引的话，会使用非主键索引。eg：user_test_count_email_index => ken_len: 243。
3. 如果存在多个非主键索引，会使用一个最小的非主键索引。eg：user_test_count_birthday_index => ken_len: 4。

##### MySQL#count（*）索引选择原理

1. InnoDB的非主键索引叶子结点存储的是索引 + 主键，而主键索引存储的是主键+表数据。
2. 在MySQL#count时，如果使用非主键索引，可以比使用主键索引，在一页里面容纳更多的关键字，节省索引树扫描次数，从而提升性能。
3. 同理，在MySQL#count时，如果使用key_len小的非主键索引，可以比使用key_len大的非主键索引，在一页里面容纳更多的关键字，节省索引树扫描次数，从而提升性能。

##### 优化方案

目的是减少count查询的性能开销。

###### 升级MySQL版本

InnDB >= 8.0.13，对于无条件count（*）做了优化，性能提升比较大。

- **缺点**：实际项目用的很少，一般不会升级MySQL版本。

###### 更换MyISAM引擎

把数据库引擎换成MyISAM，由于MyISAM引擎的表数据不存在索引树上，在遇到没有条件的count查询会直接返回结果。

- **缺点**：实际项目用的很少，一般不会修改数据库引擎。

###### 创建更小的非主键索引

- **缺点**：提升并不大。

###### 建立汇总表

分别记录每张业务表的表名以及表数量，当业务表发生变化时，更新汇总表，也可以使用触发器去维护汇总表。

- **优点**：结果比较准确、用法比较灵活。
- **缺点**：增加了维护的成本。

###### 使用sql_calc_found_rows

在做完limit语句后，紧跟found_rows()语句获取分页时sql_calc_found_rows统计的结果。

- **缺点**：需要分页；mysql 8.0.17已经废弃这种用法，且未来会被删除。
- **注意点**：需要在MYSQL终端执行，IDEA无法正常返回结果。

```sql
-- 在做完本条查询之后，自动地去执行COUNT
select sql_calc_found_rows * from salaries limit 0,10;
select found_rows() as salary_count;
```

###### 缓存+定时更新

使用定时器定时统计结果到缓存中。

- **优点**：性能比较高；结果比较准确，有误差但是比较小，除非在缓存更新的期间，新增或者删除了大量数据，这时无差才会比较大。
- **缺点**：引入了额外的组件，增加了架构的复杂度。

###### information_schema.tables

- **优点**：不操作业务表，不论业务表有多少数据，都可以迅速地返回结果。
- **缺点**：是估算值，并不是准确值。

```sql
-- 方案6：information_schema.tables
select *
from `information_schema`.TABLES
where TABLE_SCHEMA = 'employees' and TABLE_NAME = 'salaries';
```

###### show table status

优缺点同information_schema.tables。

- **优点**：不操作业务表，不论业务表有多少数据，都可以迅速地返回结果。
- **缺点**：是估算值，并不是准确值。

```sql
show table status where Name = 'salaries';
```

###### explain

优缺点同information_schema.tables。

- **优点**：不操作业务表，不论业务表有多少数据，都可以迅速地返回结果。
- **缺点**：是估算值，并不是准确值。

```sql
explain select * from salaries;
```

###### 反向查询

如果反向查询能够减少扫描的行数的话，可以考虑进行反向查询。

- **缺点**：针对的只是**特殊场合**下的范围统计查询。

```sql
select count(*) from salaries where emp_no > 10010;
-- 等价于 => 逆向查询
select count(*) - (select count(*) from salaries where emp_no <= 10010) from salaries;
```

#### 表结构优化

##### 数据库三范式

###### 第一范式（1NF）

- **概念**：
  - 字段具有**原子性**，即数据库表的每一个字段都是不可分割的原子数据项，不能是集合、数组、记录等非原子数据项。
  - 当实体中的某个属性有多个值时，必须拆分为不同的属性。
- **好处**：在统计比如address字段时，如果满足1NF的话，更加容易统计出省、市和具体地址的个数，粒度更细，字段具有原子性。
- **解决方案**：如果需要让表符合1NF，可以通过**拆字段**的方式来实现。

![1631344696874](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1631344696874.png)

###### 第二范式（2NF）

- **概念**：在满足1NF的基础上，要求表中每一行数据具有**唯一性**，并且非主键字段完全依赖主键字段。
- **好处**：消除了部分依赖，每个非主键属性必须完全依赖于主键。
- **解决方案**：如果需要让表符合2NF，可以通过**垂直拆表**的方式来实现。

![1631345460201](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1631345460201.png)

![1631345039416](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1631345039416.png)

###### 第三范式（3NF）

- **概念**：在满足2NF的基础上，表中字段不能存在传递依赖。
- **好处**：消除了传递依赖。
- **解决方案**：如果需要让表符合3NF，可以通过**垂直拆表**的方式来实现。

![1631345416769](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1631345416769.png)

![1631345387480](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1631345387480.png)

###### 反模式设计

- **概念**：出于性能的考虑，可以打破三范式的约束，将某些字段冗余起来，减少表之间的关联以提升查询效率。
- **好处**：灵活、不需要联表、性能较好。

![1631346021073](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1631346021073.png)

##### 表设计原则

1. **字段应少而精**，建议20个以内（经验之谈） ，超过后可以拆分出去。
   - 把常用的字段放到一起。
   - 把不常用的字段独立出去。
   - 大字段（TEXT/BLOB/CLOB等）独立出去。
2. **尽量使用小型字段**。
   - 比如ip可以用int类型存储，而不是varchar类型，这样可以节省空间和提高性能。
3. **避免使用允许为NULL的字段**。
   - MySQL官方建议把字段设置为非NULL，因为允许为NULL的字段很难做查询优化，且索引需要额外的空间。
4. **合理平衡范式和冗余**。
   - 一般情况下都是要遵守三范式的，但必要时可以进行反模式设计，以提高查询效率。
5. 如果表数据量非常大，可以考虑**分库分表**。
6. 其他的量化建议：
   - 单表不超过40列。
   - 单表索引不超过5个。
   - 单表不超过500w数据。
   - 单库不超过200张表。

#### SQL语句优化建议总结

##### 避免使用子查询

```sql
-- 子查询在MySQL5.5版本里，内部执行计划器是先查外表再匹配内表，而不是先查内表t2，所以，当外表的数据很大时，查询速度会非常慢。
SELECT * FROM t1 WHERE id (SELECT id FROM t2 WHERE name='hechunyang');

-- 而在MariaDB10/MySQL5.6版本里，采用了Join关联方式对其进行了优化，该SQL会自动转换为：
-- SELECT t1.* FROM t1 JOIN t2 ON t1.id = t2.id；
-- 而JOIN语句会被MySQL优化器使用对应的Join算法优化，此时，相对于原本的子查询来说，有了一定的性能提升

-- 但还要注意的是：这种优化只针对SELECT有效，对UPDATE/DELETE的子查询是无效的，因此，生产环境应避免使用子查询！
```

##### 避免函数索引

```sql
-- 低效查询，MySQL不像Oracle那样支持函数索引，即使d字段有索引，也会直接全表扫描
SELECT * FROM t WHERE YEAR(d) >= 2016;

-- 高效查询，MySQL索引项不适用函数，避免索引失效
SELECT * FROM t WHERE d >= '2016-01-01'；
```

##### 使用IN来替换OR

```sql
-- 低效查询，回表3次
SELECT * FROM t WHERE LOC_ID = 10 OR LOC_ID = 20 OR LOC_ID = 30;

-- 高效查询，MRR接口优化
SELECT * FROM t WHERE LOC_IN IN (10,20,30);
```

##### LIKE双百分号无法使用到索引

```sql
-- 低效查询，索引失效
SELECT * FROM t WHERE name LIKE '%de%';

-- 高效查询，索引不失效
SELECT * FROM t WHERE name LIKE 'de%';
```

##### 使用LIMIT读取适当的记录

```sql
-- 低效查询，查询所有记录
SELECT * FROM t WHERE 1;

-- 低效查询，只查询10条记录
SELECT * FROM t WHERE 1 LIMIT 10;
```

##### 避免数据类型不一致

```sql
-- 低效查询，索引失效
SELECT * FROM t WHERE id = '19';

-- 高效查询，索引不失效
SELECT * FROM t WHERE id = 19;
```

##### 减少GROUP BY多余的排序

```sql
-- 低效查询，GROUP BY会对goods_id进行排序，因为只是想看统计结果而已，没必要排序
SELECT goods_id,count(*) FROM t GROUP BY goods_id;

-- 高效查询，使用ORDER BY NULL来禁止GROUP BY排序，避免排序结果的消耗
SELECT goods_id,count(*) FROM t GROUP BY goods_id ORDER BY NULL;
```

##### 禁止不必要的ORDER BY排序

```sql
-- 低效查询，因为只是想看统计结果而已，没必要排序
SELECT count(1) FROM user u LEFT JOIN user_info i ON u.id = i.user_id WHERE 1 = 1 
ORDER BY u.create_time DESC;

-- 高效查询，禁止不必要的ORDER BY排序
SELECT count(1) FROM user u LEFT JOIN user_info i ON u.id = i.user_id;
```

### 3.6. MySQL主从复制？

#### 概念

- **原理**：把主节点的binlog日志，复制到从节点上执行一遍，从而达到主从数据一致的状态。

- **过程**：

  1. 主节点**binlog线程**，在每个事务更新数据完成之前，将该操作记录串行地写入到自己的binlog文件中。
     - **bin log**：主库的二进制日志。
  2. 在start slave开启主从同步后，从节点**I/O线程**，负责从master上拉取binlog内容，放在自己的relay log（中继日志）中，如果从节点读取的进度已经跟上了master，则会进入睡眠状态，并等待master产生新的事件。
     - **relay log**：从库的中继日志。
  3. 从节点另外开启一个**SQL线程**，把relay log中的语句，在自身机器上执行一遍，从而达到主从数据一致的状态。

  ![1631410904850](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1631410904850.png)

- **优点**：

  - **数据分布**：可以作为**备用数据库**，避免单点故障。
  - **负载均衡**：可以做读写分离，一个写库，一个或多个读库，在不同的服务器上，充分发挥服务器和数据库的性能，但要保证**数据的一致性**。

- **开启主从配置**：

  ```sql
  -- 1. 修改主节点的/etc/my.cnf
  log-bin=imooc_mysql
  server-id=1
  
  -- 2. 编辑从节点的/etc/my.cnf
  server-id=2
  
  -- 3. 主节点创建备份账号
  mysql> CREATE USER 'repl'@'%' IDENTIFIED BY 'password'; 
  mysql> GRANT REPLICATION SLAVE ON *.* TO 'repl'@'%';
  
  -- 4. 主节点表加读锁，阻止写入
  mysql> FLUSH TABLES WITH READ LOCK;
  
  -- 5. 主节点查看bin-log状态
  mysql > SHOW MASTER STATUS;
  
  -- 6. 主节点dump所有数据，用于初始化从节点
  mysqldump --all-databases --master-data > dbdump.db -uroot -p
  
  -- 7. 主节点解锁，允许写入
  mysql> UNLOCK TABLES;
  
  -- 8. 从节点导入主节点dump出来的数据
  mysql < aa.db -uroot -p
  
  -- 9. 从节点配置主从连接信息
  mysql> CHANGE MASTER TO
      -- 主节点地址
  	-> MASTER_HOST='master_host_name',
  	-- 主节点端口
  	-> MASTER_PORT=port_num
      -- 备份账户用户名
  	-> MASTER_USER='replication_user_name', 
  	-- 备份账户密码
  	-> MASTER_PASSWORD='replication_password', 	
      -- bin-log文件名
  	-> MASTER_LOG_FILE='recorded_log_file_name',
      -- bin-log位置
      -> MASTER_LOG_POS=recorded_log_position;
  
  -- 10. 从节点开启主从同步
  mysql> START SLAVE;
  
  -- 11. 主节点查看从节点状态
  show slave status;
  ```

#### bin log日志格式

- **三种日志格式**：statement、row、mixed。

  - **statement**：基于SQL语句的复制模式，每一条会修改数据的SQL都会记录在binlog中。
    - **优点**：不需要记录每一行的变化，减少了binlog日志量，**节约IO**，提高性能。
    - **缺点**：
      - 由于记录的只是执行语句，为了这些语句能在slave上正确运行，因此，还必须记录每条语句在执行的时候的**一些相关信息**，以保证所有语句能让slave执行出**在master端执行时相同的结果**。
      - 像一些特定函数功能，slave可与master上要保持一致，则会有很多相关问题，比如sleep()， last_insert_id()，user-defined functions(udf)等，都可能会出现问题，**导致数据不一致**。
  - **row**：基于行的复制模式，不记录sql语句上下文相关信息，仅保存哪条记录被修改。
    - **优点**：**更能保证主从库数据的一致性**，因为row level的日志内容会非常清楚的记录下每一行数据修改的细节，而且不会出现某些特定情况下的存储过程、function、trigger的调用以及触发无法等无法被正确复制的问题。
    - **缺点**：所有的执行的语句当记录到日志中的时候，都将以每行记录的修改来记录，这样可能会产生**大量日志内容**，从而导致**从库延迟变大**。
  - **mixed**：基于SQL语句和行的混合复制模式，根据语句来选用是statement还是row模式。
    - 一般的语句修改使用statment格式保存binlog。
    - 而对于一些函数，statement无法完成主从复制的操作时，则采用row格式保存binlog。

- **应用场景**：

  - Mysql默认是使用statement日志格式，推荐使用mixed。
  - 而对于一些特殊使用，可以考虑使用row，比如自己通过binlog日志来同步数据的修改，使用row会节省很多相关操作。

- **配置方式**：

  ```sql
  -- 修改主节点的/etc/my.cnf
  -- binlog日志名称
  log_bin = mysql-bin.log
  -- binlog日志格式
  binlog_format = MIXED
  -- binlog过期时间
  expire_logs_days = 7
  -- binlog文件大小
  max_binlog_size = 100m
  ```

#### 主从复制模式

- **异步复制**：Asynchronous replication，MySQL默认的复制即是异步，主库在执行完客户端提交的事务后会**立即将结果返回**给给客户端，并不关心从库是否已经接收并处理。
  - **缺点**：主如果crash掉了，此时主上已经提交的事务可能并没有传到从上，如果此时强行将从提升为主，可能导致新主上的数据不完整，从而导致数据的不一致。
- **全同步复制**：Fully synchronous replication，指当主库执行完一个事务，**所有的从库**都执行了该事务才返回给客户端。
  - **缺点**：由于需要等待所有从库执行完该事务才能返回，所以全同步复制的**性能必然会收到严重的影响**。
- **半同步复制**：Semisynchronous replication，介于异步复制和全同步复制之间，主库在执行完客户端提交的事务后不是立刻返回给客户端，而是等待**至少一个**从库接收到并写到relay log中才返回给客户端。
  - **优点**：相对于异步复制，半同步复制提高了数据的安全性，同时它也造成了一定程度的延迟，这个延迟最少是一个TCP/IP往返的时间，所以，半同步复制最好在**低延时**的网络中使用。

#### 数据一致性问题

- **原因**：
  1. **人为**：人为地在从库写入，导致从库与主库的数据不一致。
  2. **bin log格式**：使用了statement的bin log格式，某些特定函数功能在复制过程中出现问题，导致数据的不一致。
  3. **异常**：主从复制过程中，主库异常宕机了，数据还没及时同步到从库，主从切换导致的数据不一致。
  4. **延时**：主从复制有延时，如果在这个延时期间，应用程序读取从库，可能读到与主库不一致的数据。

- **解决方案**：
  1. **从库只读**：设置从库为只读模式。
  2. **row/mixed**：使用row或者mixed的复制模式。
  3. **非异步复制**：使用全同步复制，或者MySQL 5.7半同步复制。
  4. **缓存写key法**：在缓存中记录哪些行发生过写的操作，来路由到底读主库，还是读从库。
  5. **定期校验**：引入定期的主从数据校验，保证数据一致性。

- **数据强一致架构方案**：

##### SAN共享存储

![1631430708683](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1631430708683.png)

- **特点**：使用共享存储，MySQL服务器能够正常挂载文件系统并操作，如果主库发生宕机，备库可以挂载相同的文件系统，保证主库和备库使用相同的数据。
- **优点**：保证数据的强一致性，不会因为MySQL的逻辑错误发生数据不一致的情况。
- **缺点**：价格昂贵，且需要考虑共享存储的高可用。

##### DRBD磁盘复制

![1631430728718](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1631430728718.png)

- **特点**：当本地主机出现问题，远程主机上还保留着一份相同的数据，可以继续使用，保证了数据的安全。
  - DRBD是一种**基于软件、基于网络**的块复制存储解决方案，是linux内核模块实现的快级别的同步复制技术，可以与SAN达到相同的共享存储效果，主要用于对服务器之间的磁盘、分区、逻辑卷等进行数据镜像。当用户将数据写入本地磁盘时，还会将数据发送到网络中另一台主机的磁盘上，这样的本地主机(主节点)与远程主机(备节点)的数据就可以保证实时同步。
- **优点**：保证数据的强一致性，且相比于SAN储存网络，价格低廉。
- **缺点**：对io性能影响较大，且从库不提供读操作，造成服务器资源浪费。

##### 分布式协议方案 - MySQL Cluster

分布式协议可以很好解决**数据一致性**问题，比如Paxos、Raft、2PC算法等等，一系列成熟的产品如PhxSQL、MariaDB Galera Cluster、Percona XtraDB Cluster等越来越多的被大规模使用。

![1631430373534](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1631430373534.png)

- **特点**：MySQL Cluster是官方集群的部署方案，通过使用**NDB存储引擎**实时备份冗余数据，实现数据库的高可用性和数据一致性。
- **优点**：保证数据的强一致性，且全部使用官方组件，不依赖于第三方软件。
- **缺点**：配置较复杂，需要使用NDB储存引擎，与MySQL常规引擎存在一定差异，且**国内使用的较少**。

##### 分布式协议方案 - Galera

![1631430780963](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1631430780963.png)

- **特点**：基于Galera的MySQL高可用集群，是多主数据同步的MySQL集群解决方案，使用简单，没有单点故障，可用性高。
- **优点**：
  - 多主写入，无延迟复制，能保证数据强一致性。
  - 有自动故障转移功能，能够自动添加、剔除节点。
  - 有成熟的社区，有互联网公司在大规模的使用。
- **缺点**：需要为原生MySQL节点打wsrep补丁，且只支持InnoDB储存引擎。

##### 分布式协议方案 - PAXOS

![1631430935520](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1631430935520.png)

- **特点**：Paxos算法解决的问题是一个分布式系统如何就某个值（决议）达成一致，被认为是同类算法中**最有效**的，与MySQL相结合可以实现在分布式的MySQL数据的强一致性。
- **优点**：
  - 多主写入，无延迟复制，能保证数据强一致性；
  - 有自动故障转移功能，能够自动添加、剔除节点。
  - 也有成熟理论基础。
- **缺点**：只支持InnoDB储存引擎。

#### 高可用架构

在考虑MySQL数据库的高可用架构时，主要考虑以下方面：

- **可用性**：如果数据库发生了宕机或者意外中断等故障，能尽快恢复数据库的可用性，尽可能的减少停机时间，保证业务不会因为数据库的故障而中断。
- **数据一致性**：
  - 用作备份、只读副本等功能的非主节点的数据应该和主节点的数据实时或者最终保持一致。
  - 当业务发生数据库切换时，切换前后的数据库内容应当一致，不会因为数据缺失或者数据不一致而影响业务。

常见的MySQL高可用方案有：

##### 双机高可用

![1631430437735](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1631430437735.png)

- **特点**：
  1. A作为主库，负责读和写，B库作为备用库。
  2. A库故障后，B升级为主库，负责读写，A库作为备用库。
- **开发说明**：
  - 数据源配置中的数据库IP地址，采用虚拟的VIP地址，VIP由两台数据库机器上的keepalive配置，并互相检测心跳，当其中一台故障后，VIP自动漂移到另外一台正常的库上。
  - 数据库的主备配置、故障排除和数据补全，需要DBA和运维人员来维护，而程序代码或配置并不需要修改。
- **优点**：
  - 双节点，需求资源少，部署简单。
  - 架构比较简单，使用原生**半同步复制**作为数据同步的依据。
  - 保证可用性，一个机器故障了可以自动切换，没有主机宕机后的选主问题，直接切换即可。
- **缺点**：
  - 需要额外考虑haproxy、keepalived的高可用机制。
  - 完全依赖于**半同步复制**，如果半同步复制退化为异步复制，**数据一致性无法得到保证**。
    - 半同步复制机制是可靠的，如果半同步复制一直是生效的，那么便可以认为数据是一致的。
    - 但是如果由于**网络波动**等一些客观原因，导致半同步复制发生超时而切换为异步复制，那么这时便不能保证数据的一致性。
    - 所以尽可能的保证半同步复制，便可提高数据的一致性。
  - 只有一个库在工作，读写并未分离，并发有限制，且无故障时浪费服务器资源。
- **适用场景**：读和写都不高的场景（单表数据低于**500万**），双机高可用。

##### 一主一从+读写分离

![1631430468863](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1631430468863.png)

- **特点**： 
  1. A作为主库，负责写，B库作为从库，负责读。
  2. A库故障后，B负责读写。
  3. A库恢复后，B作为主库，负责写，A作为从库，负责读。
- **开发说明**：
  - 数据库的主主配置、故障排除和数据补全，依然需要DBA和运维人员来维护。
  - 而程序代码需要借助数据库中间件Mycat来实现，配置Mycat数据源，并实现对Mycat数据源的数据操作，数据库A和数据库B应该互为主从。
- **优点**：读写分离，并发有了很大的提升。
- **缺点**：
  - 完全依赖于**半同步复制**，如果半同步复制退化为异步复制，**数据一致性无法得到保证**。
  - 需要额外考虑Mycat的高可用机制，常规的解决方案是引入haproxy和keepalive对mycat做集群。
- **适合场景**：读和写都不是非常高的场景（单表数据低于**1000万**），但比方案一并发要高很多。

##### 一主多从+读写分离

![1631430498326](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1631430498326.png)

- **特点**： 
  1. A作为主库，负责写，BCD库作为从库，负责读。
  2. A库故障后，选举B为主库，负责写，CD负责读。
  3. A库恢复后，B仍然作为主库，负责写，CD负责读，A加入从库，负责读。
- **开发说明**：
  - 主库A故障后，Mycat会自动把从B提升为写库，而C、D从库，则可以通过**MHA**等工具，自动修改其主库为B，进而实现自动切换的目地。
    - **MHA Manager**：可以单独部署在一台独立的机器上**管理多个MHA master-slave集群**，也可以部署在一台MHA slave节点上。
    - **MHA Node**：运行在每台MySQL服务器上，MHA Manager会定时探测集群中的master节点，当master出现故障时，它可以自动将**最新数据的slave**提升为新的master，然后将所有其他的slave重新指向新的master。整个故障转移过程对应用程序完全透明。
- **优点**：
  - 相比于双节点的MySQL复制，三节点/多节点的MySQL发生不可用的概率更低。
  - 由于配置了多个读节点，读并发的能力有了质的提高，理论上来说，读节点可以多个，可以负载很高级别的读并发。
  - 可扩展性较好，可以根据需要扩展MySQL的节点数量和结构。
- **缺点**：
  - 至少需要三节点，相对于双节点需要更多的资源，还有可能因为网络分区发生脑裂现象。
  - 数据一致性仍然靠原生半同步复制保证，仍然存在**数据不一致**的风险。
  - 需要额外考虑配置MHA集群保证MHA的高可用。
- **适合场景**：适合写并发不大，但是读并发大的很的场景。

##### MariaDB Galera Cluster

![1631431756775](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1631431756775.png)

- **特点**：
  - 多个数据库，在负载均衡作用下，可同时进行写入和读取操作。
  - 各个库之间以Galera Replication的方法进行数据同步，即每个库理论上来说，数据是**完全一致**的。
- **开发说明**：
  - 应用程序做数据库读写时，只需要修改数据库读写IP为keepalive的虚拟节点即可。
  - 数据库配置方面相对比较复杂，需要引入haproxy、keepalive、Galaera等各种插件和配置。
- **优点**：
  - 多主写入，无延迟复制，能保证数据强一致性，可以在任意节点上进行读。
  - 有自动故障转移功能，能够自动添加、剔除节点。
- **缺点**：
  - 只支持InnoDB储存引擎，在处理事务时，会运行一个协调认证程序来保证事务的全局一致性，若该事务长时间运行，就会锁死节点中所有的相关表，导致插入卡住。
  - 整个集群的写入吞吐量是由最弱的节点限制，如果有一个节点变得缓慢，那么整个集群将是缓慢的，所以为了稳定的高性能要求，需要保证所有的节点使用统一的硬件。
  - Mysql数据库5.7.6及之后的版本才支持此种方案。
- **适合场景**：适合读写并发较大，但数据量不是非常大的场景。

##### 数据库分片+一主多从集群

![1631432218392](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1631432218392.png)

- **特点**：
  - 采用Mycat进行分片存储，可以解决写负载均衡和**数据量过大**问题。
  - 每个分片配置多个读从库，可以减少单个库的读压力。
- **开发说明**： 配置和维护量都比较大，需要配置Haproxy、keepalive和mycat集群，每个分片上又需要配置一主多从的集群。
- **优点**：可以解决高并发高数据量的问题。
- **缺点**： 
  - 配置和维护都比较麻烦，需要的软硬件设备资源大。
  - 每个分片至少需要三节点，相对于双节点需要更多的资源，还有可能因为网络分区发生脑裂现象。
  - 数据一致性仍然靠原生半同步复制保证，仍然存在**数据不一致**的风险。
- **适用场景**：读写并发都很大，并且数据量非常大的场景。

##### 高可用架构总结

- 一些对数据**实时性要求不高**的业务场景，可以考虑使用读写分离。
  - 经验之谈，如果网络延迟在5ms以内，此时做读写分离是没有问题的，数据几乎是实时同步到读库，根本感觉不到延迟。
- 但是对数据**实时性要求比较高**的业务场景，比如订单支付状态场景，则不建议采用读写分离的方案，或者也可以在写程序时指定去写库读取数据。

### 3.7. MySQL分库分表？

- **背景**：
  - 关系型数据库本身比较容易成为系统瓶颈，单机存储容量、连接数、处理能力都有限。当单表的数据量达到**1000W或100G**以后，由于查询维度较多，即使添加从库、优化索引，很多操作性下降严重。
  - 此时就要考虑对其进行切分了，切分的目的就在于**减少数据库的负担，缩短查询时间**。
- **概念**：
  - 数据库分库分表的核心内容是**数据切分**，以及切分后对**数据的定位和整合**。
    - **分库**：可以根据业务场景和地域分库，每个库并发量不超过2000。
    - **分表**：比如根据用户ID进行分表，每个表控制在300万数据。
  - **数据切分**：Sharding，指将数据分散存储到多个数据库中，使得单一数据库中的**数据量变小**，通过扩充主机的数量缓解单一数据库的性能问题，从而达到提升数据库操作性能的目的。数据切分可以分为两种方式：**垂直切分**和**水平切分**。

#### 垂直切分

垂直切分，分为**垂直分库以及垂直分表**，由于垂直分表切分后，提升的只是表级性能，仍然存在单库并发瓶颈问题，所以，垂直切分一般指的是垂直分库。

##### 垂直分库

垂直分库，指根据业务耦合性，将关联度低的不同表，存储在不同的数据库中。从按照业务进行独立划分的角度来看，其做法与大系统拆分为多个小系统类似；从单独使用一个数据库的角度来看，其做法与微服务治理的做法类似。

- **优点**：
  - 解决业务系统层面的耦合，业务清晰。
  - 与微服务的治理类似，能对不同业务的数据进行分级管理、维护、监控、扩展等。
  - 高并发场景下，垂直切分一定程度的提升IO、数据库连接数、单机硬件资源的瓶颈。
- **缺点**：
  - 部分表无法join，只能通过接口聚合方式解决，提升了开发的复杂度。
  - 分布式事务处理复杂。
  - 依然存在单表数据量过大的问题，此时需要水平切分。

![1631512512432](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1631512512432.png)

##### 垂直分表

垂直分表，指基于数据库中的列进行切分，如果某个表字段较多，此时可以新建一张扩展表，将不经常用
的或者字段长度较大的字段，拆分到扩展表中。

- **优点**：
  - 在字段很多的情况下（比如一个大表有100多个字段），通过"大表拆小表"，更**便于开发与维护**，也能**避免MySQL跨页问题**。
    - MySQL底层是通过数据页存储的，一条记录占用空间过大会导致跨页，造成额外的性能开销。
  - 另外，数据库以行为单位将数据加载到内存中，这样表中字段长度较短且访问频率较高，内存能加载更多的数据，命中率更高，**减少了磁盘IO**，从而提升了数据库性能。
- **缺点**：
  - 对于应用层来说增加了开发成本，比如查询所有数据时，需要所有的表做Join操作。
  - 依然存在单库并发瓶颈问题，此时需要垂直分库。

![1631512742958](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1631512742958.png)

#### 水平切分

- **背景**：当一个应用难以再细粒度的垂直切分，或切分后数据量行数仍然十分巨大，存在单库读写、存储性能瓶颈时，就需要进行水平切分了。
- **概念**：水平切分是指，根据表内数据内在的逻辑关系，将同一个表按不同的条件分散到多个数据库或多个表中，每个表中只包含一部分数据，从而使得单个表的数据量变小，达到分布式的效果。水平切分可以分为**库内分表**和**异库分表**。

![1631513461176](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1631513461176.png)

##### 库内分表

库内分表，是指当数据库中的某张表由于数据库过大时，可以对该表按行进行拆分，由于拆分数据后的表数据量回到了正常水平，所以在一定程度上能够解决单表的性能问题。

- **特点**：
  - 虽然解决了单一表数据量过大的问题，但是并没有将表分布到不同机器的库上。
  - 而对于减轻MySQL数据库的压力来说，帮助不是很大，因为查询请求还是竞争同一个物理机的CPU、内存和网络IO，因此，水平切分最好还是通过**异库分表**来解决。

##### 异库分表

水平切分进行异库分表后，同一张表会出现在多个数据库/表中，每个库/表的内容不同，其中典型的数据分片规则为**根据数值范围**和**根据数值取模**。

- **优点**：
  - 不存在单库数据量过大、高并发的性能瓶颈，提升了系统稳定性和负载能力。
  - 应用端改造较小，不需要拆分业务模块。
- **缺点**：
  - 跨库的join关联查询性能较差。
  - 跨分片的事务一致性难以保证。
  - 数据多次扩展难度和维护量极大。

###### 根据数值范围切分

- **概念**：可以按照时间区间，或者按ID区间来切分。
  - 比如，按日期将不同月甚至是日的数据分散到不同的库中；将userId为1~9999的记录分到第一个库，10000~20000的分到第二个库，以此类推。
  - 另外，某些系统中使用的**冷热数据分离**，其原理是将一些使用较少的历史数据迁移到其他库中，业务功能上只提供热点数据的查询，也是类似的实践。
- **优点**：
  - 单表大小可控。
  - **天然便于水平扩展**，后期如果想对整个分片集群扩容时，只需要添加节点即可，无需对其他分片的数据进行迁移。
  - 使用分片字段进行范围查找时，连续分片可快速定位分片进行快速查询，**有效避免跨分片查询的问题**。
- **缺点**：容易出现**某分片过热**的问题，即切分后热点数据可能都落在同一分片上，成为系统性能的瓶颈。
  - 比如按时间字段分片时，有些分片存储了最近时间段内的数据，可能会被频繁的读写，为热点数据；而有些分片存储的历史数据，则很少被查询。

![1631514616880](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1631514616880.png)

###### 根据数值取模切分

- **概念**：一般采用hash取模mod的切分方式。
  - 比如，将Customer表根据cusno字段切分到4个库中，余数为0的放到第一个库，余数为1的放到第二个库，以此类推。这样同一个用户的数据会分散到同一个库中，如果查询条件带有cusno字段，则可明确定位到相应库去查询。
- **优点**：分片相对比较均匀，不容易出现热点和并发访问的瓶颈。

- **缺点**：
  - 后期分片集群扩容时，需要迁移旧的数据。
    - 此时，使用**一致性hash算法**可以较好的避免这个问题。
  - 容易面临跨分片查询的复杂问题。
    - 比如上例中，如果频繁用到的查询条件中不带cusno时，将会导致无法定位数据库，从而需要同时向4个库发起查询，再在内存中合并数据，取最小集返回给应用，分库反而成为拖累。

![1631514925416](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1631514925416.png)

#### 分库分表问题

##### 事务一致性问题

- **描述**：
  1. 数据切分前，一个事务处于一个数据库实例中，提交和回滚都能保持原子性。
  2. 但数据切分后，原本同一个事务的代码逻辑，可能需要被拆分成请求多个数据库实例，导致一个事务割裂成多个事务，从而失去了本来单个事务的原子性，出现事务一致性问题。
- **解决方案**：分布式事务和最终一致性。

###### 分布式事务

- **背景**：
  - 当更新内容同时分布在不同库中，不可避免地带来跨库事务问题。
  - 跨分片事务也是分布式事务，没有简单的方案，一般可使用**XA协议**和**两阶段提交**处理。
- **优点**：分布式事务能最大限度保证了数据库操作的原子性。
- **缺点**：
  - 但在提交事务时需要协调多个节点，延后了提交事务的时间点，延长了事务的执行时间，导致事务在访问共享资源时发生冲突或死锁的概率增高。
  - 随着数据库节点的增多，这种趋势会越来越严重，从而成为系统在数据库层面上水平扩展的枷锁。

###### 最终一致性

- **背景**：
  - 对于那些性能要求很高，但对一致性要求不高的系统，往往不苛求系统的实时一致性，只要
    在允许的时间段内达到最终一致性即可，可采用**事务补偿**的方式。
- **特点**：与事务在执行中发生错误后立即回滚的方式不同，事务补偿是一种**事后检查补救**的措施。
  - 事务补偿需要结合业务系统来考虑，一些常见的实现方法有：对数据进行对账检查，基于日志进行对比，定期同标准数据来源进行同步等等。

##### 跨节点关联查询Join问题

- **描述**：
  1. 数据切分前，系统中很多列表和详情页所需的数据可以通过SQL Join来完成。
  2. 而数据切分后，数据可能分布在不同的节点上，此时Join带来的问题就比较麻烦了，考虑到性能，应尽量避免使用Join查询。
- **解决方案**：全局表、字段冗余、数据组装以及ER分片。

###### 全局表

- 全局表，也可看做是**数据字典表**，指系统中所有模块都可能依赖的一些表，为了避免跨库join查询，可以将这类表在每个数据库中都保存一份。
  - 由于这些数据通常很少会进行修改，所以也不担心一致性的问题。

###### 字段冗余

- 字段冗余，指一种典型的反范式设计，利用空间换时间，为了性能而避免join查询。
  - 比如，订单表保存userId时候，也将userName冗余保存一份，这样查询订单详情时就不需要再去查询"买家user表"了。
- **缺点**：
  - 适用场景也有限，比较适用于依赖字段比较少的情况。
  - 而且，冗余字段的数据一致性也较难保证，就像上面订单表的例子，买家修改了userName后，是否需要在历史订单中同步更新呢？因此，这种方案要结合实际业务场景进行考虑。

###### 数据组装

数据组装，指在系统层面，分两次查询，第一次查询的结果集中找出关联数据id，然后根据id发起第二次
请求得到关联数据，最后将获得到的数据进行字段拼装。

###### ER分片

- ER分片，指在关系型数据库中，如果可以先确定表之间的关联关系（ER关系），则可以将那些存在关联关系的表记录，按照ER关系存放在同一个分片上，这样能较好的避免跨分片join问题。
  - 在1:1或1:n的情况下，通常按照**主表的主键ID**切分。
  - 如下图，Data Node1上面的order订单表与orderdetail订单详情表，就可以通过orderId
    进行局部的关联查询了，而无需跨分片join了。同理，Data Node2上也一样。

![1631518097468](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1631518097468.png)

##### 跨节点分页、排序、函数问题

- **描述**：

  - 跨节点多库进行查询时，会出现limit分页、order by排序等问题。
  - 比如，分页时需要按照指定字段进行排序：当排序字段就是分片字段时，通过分片规则就比较容易定位到指定的分片。当排序字段非分片字段时，就变得比较复杂了，此时需要先在不同的分片节点中将数据进行排序并返回，然后将不同分片返回的结果集进行汇总和再次排序，最终返回给用户。
    - 如果只是取第一页的数据，对性能影响还不是很大。
    - 但是如果取得页数很大，情况则变得复杂很多，因为各分片节点中的数据可能是随机的，为了排序的准确性，需要将所有节点的前N页数据都排序好做合并，最后再进行整体的排序，这样的操作时很耗费CPU和内存资源的，所以，页数越大，系统的性能也会越差。
  - 另外，在使用Max、Min、Sum、Count之类的函数进行计算的时候，也需要先在每个分片上执行
    相应的函数，然后将各个分片的结果集进行汇总和再次计算，最终将结果返回。

  ![1631520513150](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1631520513150.png)

- **解决方案**：全局表、缓存统计、应用内统计等。

##### 全局主键避重问题

- **描述**：
  - 在分库分表环境中，由于表中数据同时存在不同数据库中，主键值平时使用的自增长将无用武之地，因为某个分区数据库自生成的ID无法保证全局唯一。
  - 因此，需要单独设计全局主键，以避免跨库主键重复问题。
- **解决方案**：UUID、MyISAM ID表、高可用ID服务器、Snowflake分布式自增ID算法。

###### UUID

UUID标准形式包含32个16进制数字，分为5段，形式为8­4­4­4­12的36个字符，例如：550e8400­e29b­41d4­a716­446655440000。

- **优点**：UUID主键，是最简单的方案，且本地生成，性能高，没有网络耗时。
- **缺点**：
  - 由于UUID非常长，会占用大量的存储空间。
  - UUID作为主键，建立索引和基于索引进行查询时都会存在性能问题，在InnoDB下，UUID的无序性会引起数据位置频繁变动，导致分页。

###### MyISAM ID表

```sql
-- 使用MyISAM存储引擎建立ID表
CREATE TABLE `sequence` (  
  ìd` bigint(20) unsigned NOT NULL auto_increment,  
  `stub` char(1) NOT NULL default '', 
  PRIMARY KEY  (ìd`),  
  UNIQUE KEY `stub` (`stub`)  
) ENGINE=MyISAM;

-- 先删除再获取自增ID
REPLACE INTO sequence (stub) VALUES ('a');  
SELECT LAST_INSERT_ID();
```

- **概念**：
  - stub字段（存根）设置为**唯一索引**，同一stub值在sequence表中只有一条记录，可以同时为多张表生成全局ID。
  - 使用MyISAM存储引擎而不是 InnoDB，以获取更高的性能，因为MyISAM使用的是**表级锁**，对表的读写是串行的，所以不用担心在并发时两次读取同一个ID值。
  - 使用REPLACE INTO + SELECT获取自增ID，但必须保证两操作在同一事务内，其中REPLACE INTO会先删除旧数据再生成新数据，从而实现主键自增。
- **优点**：简单。
- **缺点**：
  - 存在单点问题，强依赖DB，当DB异常时，整个系统都不可用。
  - 虽然配置主从可以增加可用性，但当主库挂了，主从切换时，数据一致性在特殊情况下难以保证。
  - 另外，整个系统性能瓶颈限制在单台MySQL的读写性能上。

###### 高可用ID服务器

![1631524708834](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1631524708834.png)

- **背景**：flickr团队使用的一种主键生成策略，与上面的sequence表方案类似，但更好的解决了单点和性能瓶颈的问题。

- **思想**：

  - 建立2个以上的全局ID生成的服务器，每个服务器上只部署一个数据库，每个库有一张sequence表用于记录当前全局ID。
  - 表中ID增长的步长相同，等于库的数量，**起始值依次错开**，这样能将ID的生成散列到各个数据库上。
    - 比如第一台为（1，3，5，7，...）以及第二台为（2，4，6，8，...），可见ID是错开的。

- **优点**：生成ID的压力能够均匀分布在两台机器上，同时提供了系统容错，当第一台出现了错误，可以自动切换到第二台机器上获取ID。

- **缺点**：

  - 系统添加机器水平扩展时，需要停止原本正在运行的ID服务器，以修改步长。
  - 每次获取ID都要读写一次DB，DB的压力还是很大，只能靠堆机器来提升性能。

- **优化方案  **：批量获取ID。

  - 使用批量的方式降低数据库的写压力，每次获取一段区间的ID号段，用完之后再去数据库获取，可以大大减轻数据库的压力。
    1. 比如，还是使用两台DB保证可用性，数据库中只存储当前的最大ID。
    2. ID生成服务每次批量拉取6个ID，先将max_id修改为5，当应用访问ID生成服务时，就不需要访问数据库，从号段缓存中依次派发0~5的ID。
    3. 当这些ID发完后，再将max_id修改为11，下次就能派发6~11的ID。
    4. 可见，数据库的压力降低为原来的1/6。
  - **缺点**：ID生成服务需要维护最大ID值，再下次生成ID时，需要告诉DB M1、DB M2各自的初始值。

  ![1631525249951](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1631525249951.png)

###### Snowflake分布式自增ID算法

Twitter的snowflake算法，解决了分布式系统生成全局ID的需求，可以生成64位的Long型数字。

- **概念**：1 + 41 + 10 + 12  = 64位。
  - 第一位未使用。
  - 接下来41位是毫秒级时间，41位的长度可以表示**69年**的时间。
  - 5位datacenterId，5位workerId，这10位的长度最多支持部署**1024个节点**。
  - 最后12位是毫秒内的计数，12位的计数顺序号支持每个节点每毫秒产生**4096个ID序列**。
- **优点**：
  - 毫秒数在高位，生成的ID整体上按时间趋势**递增**。
  - 不依赖第三方系统，稳定性和效率较高，理论上QPS约为409.6w/s（1000*2^12），并且整个分布式系统内不会产生ID碰撞。
  - 可根据自身业务灵活分配bit位。
- **缺点**：强依赖机器时钟，如果时钟回拨，则可能导致生成ID重复。

![1631525530303](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1631525530303.png)

###### 美团点评分布式ID生成系统 - Leaf

Leaf，在美团点评公司内部服务包含金融、支付交易、餐饮、外卖、酒店旅游、猫眼电影等众多业务线，其性能在4C8G的机器上QPS能压测到近5w/s，TP999 1ms，已经能够满足大部分的业务的需求。

![1631532240042](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1631532240042.png)

- **Leaf-segment ID服务器方案**：

  - **思想**：
    1. 利用proxy server批量获取，每次获取一个segment(step决定大小)号段的值。
    2. 用完之后再去数据库获取新的号段，可以大大的减轻数据库的压力。
  - **实现**：
    1. biz_tag用来区分业务，max_id表示该biz_tag目前所被分配的ID号段的最大值，step表示每次分配的号段长度。
       - 各个业务不同的发号需求用biz_tag字段来区分，每个biz-tag的ID获取相互隔离，互不影响。
       - 如果以后有性能需求需要对数据库扩容，不需要上述描述的复杂的扩容操作，只需要对biz_tag分库分表就行。
    2. 原来获取ID每次都需要写数据库，现在只需要把step设置得足够大，比如1000，那么只有当1000个号被消耗完了之后才会去重新读写一次数据库，此时读写数据库的频率从1减小到了1/step。
    3. 比如，test_tag在第一台Leaf机器上是1~1000的号段，当这个号段用完时，会去加载另一个长度为step=1000的号段。如果另外两台号段都没有更新，这个时候第一台机器新加载的号段就应该是3001~4000，同时，数据库对应的biz_tag这条数据的max_id会从3000被更新成4000。
  - **优点**：
    - Leaf服务可以很方便的线性扩展，性能完全能够支撑大多数业务场景。
    - ID号码是趋势递增的8byte的64位数字，满足上述数据库存储的主键要求。
    - 容灾性高，Leaf服务内部有号段缓存，即使DB宕机，短时间内Leaf仍能正常对外提供服务。
    - 可以自定义max_id的大小，非常方便业务从原有的ID方式上迁移过来。
  - **缺点**：
    - TP999数据波动大，当号段使用完之后还是会hang在更新数据库的I/O上，TP999数据会出现偶尔的尖刺。
    - DB宕机会造成整个系统不可用。
    - ID号码不够随机，能够泄露发号数量的信息，不太安全。

- **双buffer优化**：优化第一个缺点，当TP999号段使用完，线程取号时阻塞的问题。

  - **背景**：

    - Leaf 取号段的时机是在**号段消耗完**的时候进行的，也就意味着号段临界点的ID下发时间取决于下一次从DB取回号段的时间，并且在这期间进来的请求也会因为DB号段没有取回来，导致**线程阻塞**。
    - 如果请求DB的网络和DB的性能稳定，这种情况对系统的影响是不大的，但是假如取DB的时候网络发生抖动，或者DB发生慢查询就会导致整个系统的响应时间变慢。

  - **思想**：

    - 为了让DB取号段的过程能够做到无阻塞，不需要在DB取号段的时候阻塞请求线程，即当号段消费到**某个点时**就异步的把下一个号段加载到内存中，而不需要等到号段用尽的时候才去更新号段。

  - **实现**：

    1. 采用双buffer的方式，Leaf服务内部有两个号段缓存区segment。
    2. 当前号段已下发10%时，如果下一个号段未更新，则另启一个更新线程去更新下一个号段。
    3. 当前号段全部下发完后，如果下个号段准备好了则切换到下个号段为当前segment接着下发，循环往复。

    ![1631532711485](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1631532711485.png)

- **高可用容灾**：解决第二个缺点，DB宕机会造成整个系统不可用的问题。

  - **DB高可用方案**：
    - 采用一主两从的方式，同时分机房部署，Master和Slave之间采用**半同步复制**方式同步数据。同时使用公司DBProxy（原Atlas）数据库中间件做主从切换。
    - 当然，这种方案在一些情况会退化成异步模式，甚至在**非常极端**情况下仍然会造成数据不一致的情况，但是出现的概率非常小。
    - 如果系统要保证100%的数据强一致，可以选择使用**类Paxos算法**实现的强一致MySQL方案，但是运维成本和精力都会相应的增加，应该根据实际情况进行选型。
  - **应用高可用方案**：
    - Leaf服务分IDC部署，内部的服务化框架是“MTthrift RPC”。
    - 服务调用的时候，根据负载均衡算法会优先调用同机房的Leaf服务。
    - 如果该IDC内Leaf服务不可用时，会选择其他机房的Leaf服务。
    - 同时，服务治理平台OCTO还提供了针对服务的过载保护、一键截流、动态流量分配等对服务的保护措施。

  ![1631543929139](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1631543929139.png)

- **Leaf-snowflake方案**：优化第三个缺点，ID号码不够随机，能够泄露发号数量的信息，不太安全。

  ![1631544317384](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1631544317384.png)

  - Leaf-snowflake方案完全沿用snowflake方案的bit位设计，即是“1+41+10+12”的方式组装ID号。

    - 对于workerID的分配，当服务集群数量较小的情况下，完全可以手动配置；当Leaf服务规模较大，动手配置成本太高，此时使用Zookeeper持久顺序节点的特性自动对snowflake节点**配置wokerID**。

  - Leaf-snowflake是按照下面几个步骤启动的：

    1. 启动Leaf-snowflake服务，连接Zookeeper，在leaf_forever父节点下检查自己是否已经注册过（是否有该顺序子节点）。
    2. 如果有注册过，则直接取回自己的workerID（zk顺序节点生成的int类型ID号），启动服务。
    3. 如果没有注册过，就在该父节点下面创建一个持久顺序节点，创建成功后取回顺序号当做自己的workerID号，启动服务。
    4. 除了每次会去ZK拿数据以外，也会在本机文件系统上**缓存一个workerID文件**，当ZooKeeper出现问题，恰好机器出现问题需要重启时，能保证服务能够正常启动。这样做到了对Zookeeper的**弱依赖**。一定程度上提高了SLA。

    ![1631544564754](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1631544564754.png)

  - **解决snowflake时钟回退问题**：由于snowflake依赖时间，如果机器的时钟发生了回拨，那么就会有可能生成重复的ID号，需要解决时钟回退的问题。

    1. 服务启动时首先检查自己是否写过ZooKeeper leaf_forever节点，若写过，则用自身系统时间与leaf_forever/#{self}节点记录时间做比较。
    2. 若小于leaf_forever/​#{self}时间则认为机器时间发生了**大步长回拨**，服务启动失败并报警；若未写过，证明是新服务节点，直接创建持久节点leaf_forever/#{self}并写入自身系统时间。
    3. 接下来综合对比其余Leaf节点的系统时间来判断自身系统时间是否准确，具体做法是取leaf_temporary下的所有临时节点（所有运行中的Leaf-snowflake节点）的服务IP：Port。
    4. 然后通过RPC请求得到所有节点的系统时间，计算sum(time)/nodeSize。
    5. 若abs( 系统时间-sum(time)/nodeSize ) < 阈值，认为当前系统时间准确，正常启动服务，同时写临时节点leaf_temporary/#{self} 维持租约；否则认为本机系统时间发生大步长偏移，启动失败并报警。
    6. 其中，leaf_temporary临时结点会每隔一段时间(3s)上报自身系统时间，并写入到leaf_forever/#{self}。

##### 数据迁移、扩容问题

当业务高速发展，面临性能和存储的瓶颈时，才会考虑分片设计，此时就不可避免的需要考虑历史数据迁移的问题。

- 一般做法是先读出历史数据，然后按指定的分片规则再将数据写入到各个分片节点中。
  - 如果采用**数值范围分片**，只需要添加节点就可以进行扩容了，不需要对分片数据迁移。
  - 如果采用的是**数值取模分片**，则考虑后期的扩容问题就相对比较麻烦。
- 此外，还需要根据当前的数据量和QPS，以及业务发展的速度，进行容量规划，推算出大概需要多少分片（一般建议单个分片上的单表数据量不超过**1000W**）。

#### 分库分表原则

能不切分尽量不要切分，不到万不得已不用轻易使用分库分表这个大招，**避免过度设计和过早优化**。

- 因为并不是所有表都需要进行切分，主要还是看数据的增长速度，切分后会在某种程度上提升业务的复杂度，数据库除了承载数据的存储和查询外，协助业务更好的实现需求也是其重要工作之一。

- 分库分表之前，不要为分而分，先尽力去做力所能及的事情，例如：升级硬件、升级网络、读写分离、索引优化等等，只有当数据量达到单表的瓶颈时候，才考虑分库分表。

#### 分库分表时机

##### 影响正常运维和业务访问时

数据量过大，影响到正常运维和业务访问时，需要进行分库分表，原因有：

- **备份风险高**：对数据库备份，如果单表太大，备份时需要大量的磁盘IO和网络IO，比如1T的数据，网络传输占50MB时候，需要20000秒才能传输完毕，整个过程的风险都是比较高的。
- **DDL修改锁表时间长**：对一个很大的表进行DDL修改时，MySQL会锁住全表，这个时间会很长，这段时间业务不能访问此表，影响很大，在此操作过程中，都算为风险时间。此时，如果将数据表拆分，总量减少，则有助于降低这个风险。
- **访问压力高**：大表会经常访问与更新，就更有可能出现锁等待，如果将数据切分，用空间换时间，可以降低访问压力。

##### 业务快速发展，查询效益不高时

```sql
-- 项目初始阶段的user表
id                   bigint             #用户的ID 
name                 varchar            #用户的名字 
last_login_time      datetime           #最近登录时间 
personal_info        text               #私人信息 
.....                                   #其他信息字段
```

1. 在项目初始阶段，这种设计是满足简单的业务需求的，也方便快速迭代开发。
2. 而当业务快速发展时，用户量从10w激增到10亿，用户非常的活跃，每次登录会更新last_login_name字段，使得user表被不断update，压力很大，而其他字段：id, name, personal_info 是不变的或很少更新的，
3. 此时，在业务角度，就需要将last_login_time拆分出去，新建一个user_time表。
4. 而由于personal_info 属性是更新和查询频率较低的，并且text字段占据了太多的空间，此时，也要垂直拆分出 user_ext 表了。

##### 数据量快速增长，性能出现瓶颈时

- 随着业务的快速发展，单表中的数据量会持续增长，当性能接近瓶颈时，就需要考虑水平切分，做分库分表了。
  - 经验数值参考，每个表控制在300万数据，每个库并发不超过2000。
- 此时，一定要选择合适的切分规则，提前预估好数据容量。

##### 出于安全性和可用性考虑时

鸡蛋不要放在一个篮子里。

- 在业务层面上垂直切分，将不相关的业务的数据库分隔，因为每个业务的数据量、访问量都不同，不能因为一个业务把数据库搞挂而牵连到其他业务。
- 利用水平切分，当一个数据库出现问题时，不会影响到100%的用户，每个库只承担业务的一部分数据，这样整体的可用性就能提高。

#### 分库分表实现

##### MyCAT

- MyCAT，是一个开源的分布式数据库系统，是一个实现了MySQL协议的的Server，是基于**服务端代理模式**的分库分表实现。
- 前端用户可以把它看作是一个数据库代理，用 MySQL 客户端工具和命令行访问；而其后端可以用MySQL 原生（Native）协议与多个 MySQL 服务器通信。
- 可以用 JDBC 协议与大多数主流数据库服务器通信，其核心功能是分表分库，即将一个大表水平分割为 N 个小表，存储在后端 MySQL 服务器里或者其他数据库里。

```xml
<!-- mycat/conf/schema.xml -->
<schema>:   表示的是在mycat中的逻辑库配置，逻辑库名称为:TESTDB
<table>:    表示在mycat中的逻辑表配置，逻辑表名称为:user,映射到两个数据库节点dataNode中,切分规则为:rule1(在rule.xml配置)
<dataNode>: 表示数据库节点,这个节点不一定是单节点，可以配置成读写分离.
<dataHost>: 真实的数据库的地址配置
<heartbeat>:用户心跳检测
<writeHost>:写库的配置
    
<!-- mycat/conf/rule.xml -->
<property name="count">2</property>: 配置有拆分了多个库(表)，需要和前面配置中的dataNode个数一致，否则会出错.
    
<!-- 至于Java应用层配置无需做任何变化，只需要连接MyCAT的逻辑库和逻辑名就好 -->
```

##### Sharding-JDBC

- Sharding-JDBC，是一个开源的分布式关系型数据库中间件，是基于**客户端代理模式**的分库分表实现。
- 可以定位为轻量级的Java框架，通过Jar包提供服务；可以理解为增强版的JDBC驱动，完全兼容各种ORM框架。

![1631587631799](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1631587631799.png)

###### 配置实际数据源

```xml
<!-- 实际数据源1 -->
<bean id="ds0" class="com.zaxxer.hikari.HikariDataSource" destroy-method="close">
    <property name="driverClassName" value="com.mysql.cj.jdbc.Driver"/>
    <property name="jdbcUrl" value="jdbc:mysql://192.168.1.142/sharding_order?serverTimezone=Asia/Shanghai&amp;useSSL=false"/>
    <property name="username" value="imooc"/>
    <property name="password" value="Imooc@123456"/>
</bean>

<!-- 实际从数据源1.1 -->
<bean id="slave0" class="com.zaxxer.hikari.HikariDataSource" destroy-method="close">
    <property name="driverClassName" value="com.mysql.cj.jdbc.Driver"/>
    <property name="jdbcUrl" value="jdbc:mysql://192.168.1.141/sharding_order?serverTimezone=Asia/Shanghai&amp;useSSL=false"/>
    <property name="username" value="imooc"/>
    <property name="password" value="Imooc@123456"/>
</bean>

<!-- 实际数据源2 -->
<bean id="ms1" class="com.zaxxer.hikari.HikariDataSource" destroy-method="close">
    <property name="driverClassName" value="com.mysql.cj.jdbc.Driver"/>
    <property name="jdbcUrl" value="jdbc:mysql://192.168.1.143/shard_order?serverTimezone=Asia/Shanghai&amp;useSSL=false"/>
    <property name="username" value="imooc"/>
    <property name="password" value="Imooc@123456"/>
</bean>

<!-- 配置读写分离负载规则 -->
<master-slave:load-balance-algorithm id="msStrategy" type="RANDOM"/>
```

###### 配置实例分片规则

```xml
<!-- 配置Sharding JDBC数据源 以及 逻辑表与分片规则 -->
<sharding:data-source id="sharding-data-source">
    <sharding:sharding-rule data-source-names="ds0,slave0,ms1" default-data-source-name="ms0">
        <!-- 读写分离配置 -->
        <sharding:master-slave-rules>
            <sharding:master-slave-rule id="ms0" master-data-source-name="ds0" slave-data-source-names="slave0" strategy-ref="msStrategy"/>
        </sharding:master-slave-rules>

        <!-- 配置普通分片表 -->
        <sharding:table-rules>
            <sharding:table-rule logic-table="t_order" actual-data-nodes="ms$->{0..1}.t_order_$->{1..2}"
                                 database-strategy-ref="databaseStrategy" table-strategy-ref="tableStrategy" key-generator-ref="snowflake"/>

            <!-- 配置绑定表的分片表 -->
            <sharding:table-rule logic-table="t_order_item" actual-data-nodes="ms$->{0..1}.t_order_item_$->{1..2}"
                                 database-strategy-ref="databaseStrategy" table-strategy-ref="tableOrderItemStrategy" key-generator-ref="snowflake"/>
        </sharding:table-rules>

        <!-- 配置广播表(全局表) -->
        <sharding:broadcast-table-rules>
            <sharding:broadcast-table-rule table="area"/>
        </sharding:broadcast-table-rules>

        <!-- 配置绑定表(子表): 4.0.0-RC2版本会抛出空指针bug: 原因是源码中先创建了绑定表规则然后才是获取广播表 -->
        <sharding:binding-table-rules>
            <sharding:binding-table-rule logic-tables="t_order,t_order_item"/>
        </sharding:binding-table-rules>
    </sharding:sharding-rule>
</sharding:data-source>
```

###### 配置散列规则

```xml
<!-- 数据库分片规则: 根据user_id取模 -->
<sharding:inline-strategy id="databaseStrategy" sharding-column="user_id" algorithm-expression="ms$->{user_id % 2}"/>
```

###### 配置主键ID生成规则

```xml
<!-- 配置主键Key生成规则 -->
<!--<sharding:key-generator id="uuid" column="order_id" type="UUID"/>-->
<sharding:key-generator id="snowflake" column="order_id" type="SNOWFLAKE" props-ref="snow"/>
<bean:properties id="snow">
    <!-- DataCenterId + MachineId => 10bit -->
    <prop key="worker.id">678</prop>
    <!-- 最大容忍回调时间 -->
    <prop key="max.tolerate.time.difference.milliseconds">10</prop>
</bean:properties>
```

##### 总结

|      | MyCAT                                          | Sharding-JDBC                                            |
| ---- | ---------------------------------------------- | -------------------------------------------------------- |
| 区别 | 是服务端代理；不支持库内分表，只支持分异库分表 | 是客户端代理；既支持库内分表，也支持异库分表             |
| 优点 | 对于各个项目透明，应用层无需关心               | 不用部署，运维成本低；不需要服务代理层的二次转发，性能高 |
| 缺点 | 需要部署，运维成本高                           | 各个系统耦合Sharding-JDBC，给以后升级带来麻烦            |

#### 分库分表案例

比如，用户中心，是一个非常常见的业务，主要提供用户注册、登录、查询/修改等功能，其核心表为用户表。

```sql
-- 用户表
User(uid, login_name, passwd, sex, age, nickname)
```

1. 任何脱离业务的架构设计都是耍流氓，在进行分库分表前，需要对**业务场景**需求进行梳理：

   - **用户侧**：前台访问，访问量较大，需要保证高可用和高一致性。主要有两类需求：
     - **用户登录**：通过login_name/phone/email查询用户信息，1%请求属于这种类型。
     - **用户信息查询**：登录之后，通过uid来查询用户信息，99%请求属这种类型。
   - **运营侧**：后台访问，支持运营需求，按照年龄、性别、登陆时间、注册时间等进行分页的查询，是内部系统，访问量较低，对可用性、一致性的要求不高。

2. 水平切分方法，当数据量越来越大时，需要对数据库进行水平切分，切分方法有**根据数值范围**和**根据数值取模**：

   - **根据数值范围**：以主键uid为划分依据，按uid的范围将数据水平切分到多个数据库上。
     - **例如**：user­db1存储uid范围为0~1000w的数据，user­db2存储uid范围为1000w~2000w uid数据。
     - **优点**：扩容简单，如果容量不够，只要增加新db即可。
     - **缺点**：请求量不均匀，一般新注册的用户活跃度会比较高，所以新的user­ db2会比user­ db1负载高，导致**服务器利用率不平衡**。
   - **根据数值取模**：也是以主键uid为划分依据，按uid取模的值将数据水平切分到多个数据库上。
     - **例如**：user­ db1存储uid取模得1的数据，user­ db2存储uid取模得0的uid数据。
     - **优点**：数据量和请求量分布均均匀。
     - **不足**：扩容麻烦，当容量不够时，新增加db，需要rehash，同时需要考虑对数据进行平滑的迁移。

3. 水平切分后，对于按uid查询的需求能很好的满足，可以直接路由到具体数据库，但对于按**非uid**的查询，例如login_name，就不知道具体该访问哪个库了，此时需要**遍历所有库**，性能会降低很多。

   - **用户侧解决方案**：可以采用**建立非uid属性到uid的映射关系**的方案。

     - **非uid需求**：主要需求以单行查询为主，需要建立login_name/phone/email到uid的映射关系，可以解决这些字段的查询问题。

     - **映射关系**：

       1. 比如，login_name不能直接定位到数据库，可以建立**login_name→uid**的映射关系，用索引表或缓存来存储。
       2. 当访问login_name时，先通过映射表查询出login_name对应的uid，再通过uid定位到具体的库。
       3. 由于映射表只有两列，因此可以承载很多数据，当数据量过大时，还可以对映射表再做水平切分。同时，这类kv格式的索引结构，可以很好的使用cache来优化查询性能，而且映射关系不会频繁变更，缓存命中率会很高。

     - **分库基因优化**：上面的映射关系的方法需要额外存储映射表，按非uid字段查询时，还需要多一次数据库或cache的访问。

       ![1631583587908](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1631583587908.png)

       1. 假如通过uid分库，分为8个库，采用uid%8的方式进行路由，此时是由uid的最后3bit来决定这行User数据具体落到哪个库上，那么这3bit可以看为分**库基因**。
       2. 如果想要消除多余的存储和查询，可以通过**f函数取login_name的基因作为uid的分库基因**。
       3. 而生成uid可以参考分布式唯一ID的生成方案，来生成61big的全局唯一ID，再加上最后3位bit值=f(login_name)当做分库基金。
       4. 这样，当查询login_name时，只需计算f(login_name)%8的值，就可以定位到具体的库。
       5. 不过这样需要提前做好容量规划，预估未来几年的数据量需要分多少库，要预留一定bit的分库基因。

   - **运营侧解决方案**：可以采用**前台与后台分离**的方案。

     - **非uid需求**：很多批量分页且条件多样的查询，这类查询计算量大，返回数据量大，对数据库的性能消耗较高。此时，如果和用户侧公用同一批服务或数据库，可能因为后台的少量请求，占用大量数据库资源，而导致用户侧访问性能降低或超时。
     - **前台与后台分离**：
       1. 运营侧后台业务抽取**独立的service和db**，解决和前台业务系统的耦合。
       2. 由于运营侧对可用性、一致性的要求不高，可以不访问实时库，而是通过binlog异步同步数据到**运营库**进行访问。
       3. 另外，在数据量很大的情况下，还可以使用**ES搜索引擎或Hive**来满足后台复杂的查询方式。

#### 老数据迁移

双写（新老写库）不中断迁移：

1. 线上系统里所有写库的地方，增删改操作，除了对老库增删改，都加上对新库的增删改（**双写**）。
2. 系统部署以后，还需要跑程序**读老库数据写新库**，写的时候需要判断updateTime。
3. 循环执行，直至两个库的数据完全一致，最后重新部署分库分表的代码即可。

#### 系统性能评估与扩容

- **场景**：和家亲目前有1亿用户：场景 10万写并发，100万读并发，60亿数据量。
- **设计思路**：考虑极限情况，32库*32表~64个表，一共1000 ~ 2000张表。
  - 支持**3万**的写并发，配合MQ可以实现每秒**10万**的写入速度。
  - 读写分离**6万**读并发，配合分布式缓存可以实现每秒**100万**读并发。
  - 2000张表每张**300万**，可以最多写入**60亿**的数据。
  - 64张用户表，支撑**亿级**用户，后续最多也就扩容一次。
- **动态扩容步骤**：
  1. 推荐是32 库 * 32 表，对于我们公司来说，可能几年都够了。
  2. 配置路由的规则，uid % 32 = 库，uid / 32 % 32 = 表。
  3. 扩容的时候，申请增加更多的数据库服务器，呈倍数扩容。
  4. 由 DBA 负责将原先数据库服务器的库，迁移到新的数据库服务器上去。
  5. 修改一下配置，重新发布系统，上线，原先的路由规则变都不用变。
  6. 直接可以基于 n 倍的数据库服务器的资源，继续进行线上系统的提供服务。

### 3.8. 线上故障及优化？

#### 更新失败 | 主从同步延时

1. 以前线上确实处理过因为主从同步延时问题而导致的线上的 bug，属于小型的生产事故。
2. 是这个么场景，有个同学是这样写代码逻辑的：先插入一条数据，再把它查出来，然后更新这条数据。
3. 在生产环境高峰期，写并发达到了 2000/s，这个时候，**主从复制延时**大概是在小几十毫秒，此时，线上会发现，每天总有那么一些数据，我们期望更新一些重要的数据状态，但在高峰期时候却没更新。
4. 接着用户跟客服反馈，而客服就会反馈给我们。
5. 我们通过 MySQL 命令：`show slave status`，查看 `Seconds_Behind_Master` ，可以看到从库复制主库的数据落后了几 ms。一般来说，如果主从延迟较为严重，有以下解决方案：
   - **分库**：拆分为多个主库，每个主库的写并发就减少了几倍，主从延迟可以忽略不计。
   - **重写代码**：由于插入数据时立马查询可能查不到，如果确实需要立马要求就查询到，可以对这个查询**设置直连主库**或者**延迟查询**（主从复制延迟一般不会超过50ms）。

#### **应用崩溃 | 分库分表优化**

1. 我们有一个线上通行记录的表，由于数据量过大，进行了分库分表，当时分库分表初期经常产生一些问题。典型的就是通行记录查询中使用了深分页，通过一些工具如MAT、Jstack追踪到，是由于sharding-jdbc内部引用造成的。
2. 通行记录数据被存放在两个库中，如果没有提供**切分键**，查询语句就会被分发到所有的数据库中，比如查询语句是 limit 10、offset 1000，最终结果只需要返回 10 条记录，但是数据库中间件要完成这种计算，则需要 （1000+10）* 2 = 2020条记录来完成这个计算过程。
3. 如果 offset 的值过大，使用的内存就会暴涨，虽然sharding-jdbc使用归并算法进行了一些优化，但在实际场景中，深分页仍然引起了**内存和性能**问题。
4. 这种在中间节点进行**归并聚合**的操作，在分布式框架中非常常见，比如，在ElasticSearch中，就存在相似的数据获取逻辑，**不加限制的深分页**，同样会造成ES的内存问题。
5. **业界解决方案**：
   - **方法一 - 全局视野法**：
     - 将order by time offset X limit Y，改写成order by time offset 0 limit X + Y，直接返回X + Y条数据，然后服务层对得到的N *（X + Y）条数据后，N为数据库实例的数量，再在服务层进行内存排序，内存排序后再取偏移量X后的Y条记录。
     - **缺点**：随着翻页的进行，性能会越来越低。
   - **方法二 - 业务折衷法-禁止跳页查询**：
     - 用正常的方法取得第一页数据，并得到第一页记录的time_max，然后每次翻页，将order by time offset X limit Y，改写成order by time where time > #time_max limit Y以保证每次只返回一页数据。
     - **优点**：性能为常量。
     - **缺点**：由于每次记录上一页的time_max，所以不能跳页查询。
   - **方法三 - 业务折衷法-允许模糊数据**：将order by time offset X limit Y，改写成order by time offset X/N limit Y/N，N为数据库实例的数量，这样视所有数据库实例为一个整体，每个实例平均查询X/N偏移，Y/N数据量，汇总后可以达到X偏移量以及Y数据量。
     - **优点**：由于每个数据库实例平摊掉成本，所以查询性能较好。
     - **缺点**：由于每个数据库实例查询的都是平均偏移量和平均数据量，可能并不是真实想要的数据，所以需要业务允许这样模糊查询。

#### 查询异常 | SQL 调优

1. 分库分表前，有一段用用户名来查询某个用户的 SQL 语句：select * from user where name = "xxx" and community="other";
2. 为了达到动态拼接的效果，这句SQL语句被一位同事进行了如下修改select * from user where 1=1，他的本意是，当name 或者community 传入为空的时候，动态去掉这些查询条件，这种写法在 MyBaits 的配置文件中，也非常常见，在大多数情况下，这种写法是没有问题的，因为结果集合是可以控制的。
3. 但随着系统的运行，用户表的记录越来越多，当传入的 name 和 community 全部为空时，悲剧的事情发生了：数据库中的所有记录，都会被查询出来，载入到 JVM 的内存中。由于数据库记录实在太多，直接把内存给撑爆了。
4. 由于这种原因引起的内存溢出，发生的频率非常高，比如导入Excel文件时，通常的解决方式是**强行加入分页功能**，或者对一些**必填的参数进行校验**：
   - **Controller层优化**：
     - 现在很多项目都采用前后端分离架构，所以 Controller 层的方法，一般使用@ResponseBody 注解，把查询的结果，解析成 JSON 数据返回。这在数据集非常大的情况下，会**占用很多内存资源**。假如结果集在解析成 JSON 之前，占用的内存是 10MB，那么在解析过程中，有可能会使用20M或者更多的内存。
     - 因此，**保持结果集的精简**，是非常有必要的，这也是 DTO（Data Transfer Object）存在的必要，互联网环境不怕小结果集的高并发请求，却非常恐惧大结果集的耗时请求，这是其中一方面的原因。
   - **Service层优化**：
     - Service 层用于处理具体的业务，更加贴合业务的功能需求，一个 Service可能会被多个 Controller 层所使用，也可能会使用多个dao结构的查询结果进行计算、拼装。
     - 比如List< User > users = dao.getAllUser();会导致全询user表全部的数据，在数据量达到一定程度后，才会暴露问题。
   - **ORM层优化**：
     - 比如使用Mybatis时，有一个批量导入服务，在 MyBatis 执行**批量插入**的时候，竟然产生了内存溢出，按道理这种插入操作是不会引起额外内存占用的，最后通过源码追踪到了问题。
     - 这是因为 MyBatis 循环处理batch的时候，操作对象是数组，而我们在接口定义的时候，使用的是 List；当传入一个非常大的 List 时，它需要调用 List 的 toArray 方法将列表转换成数组（浅拷贝）；在最后的拼装阶段，又使用了**StringBuilder**来拼接最终的 SQL，所以实际使用的内存要比 List 多很多。
     - 事实证明，不论是插入操作还是查询动作，只要涉及的数据集非常大，就容易出现问题，由于项目中众多框架的引入，想要分析这些具体的内存占用，就变得非常困难。因此，**保持小批量操作和结果集的干净**，是一个非常好的习惯。

