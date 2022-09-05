# **十九、HBase 篇**

### 1.1. HBase 是什么？

HBase，Hadoop dataBase，是一个高可靠、高性能、可伸缩的、支持海量数据增删改查的、key-value 类型的 NoSQL 数据库。

1. 高可靠：指的是 HBase集群支持多个主节点、多个从节点，集群可靠性较高，由于 HBase 的数据存储在 HBase 上，数据可靠性也较高。
2. 高性能：指的是 HBase 可以存储上亿或者十亿级别的数据，并可以支持毫秒级查询。
3. 面向列：指的是 HBase 数据的存储方式，是按照列存储的。
4. 可伸缩：指的是 HBase 集群可以方便地添加、或者删除一个节点。
5. 支持海量数据增删改查：
   - 1）由于 Hive 不支持行记录级别的数据更新、删除等操作，即 OLTP 事务操作，而 HBase 的出现则可以解决这个问题。
   - 2）由于 HDFS 本身不支持行级修改、删除，而 HBase 又是基于 HDFS 存储数据的，但却可以支持修改、删除，这是因为 HBase 封装了一个中间层，在中间层针对删除了的数据，会做一个特殊标记，这样在查询时，就可以过滤掉已经标记为删除状态的数据，从而实现逻辑删除。 

### 1.2. 什么是列式存储？

1. 行式存储：
   - 1）指的是，数据存储时按行存储（比如 MySQL、Oracle），一行中的数据在存储介质中，以连续存储的形式存在。
   - 2）查询时，无论 `select 字段 from ...` 查询几个字段，都会一次性地查询出所有列，然后过滤掉不需要的列才返回，所以不适合查询少量字段的情况。
2. 列式存储：
   - 1）指的是，数据存储时按列存储，一行中的数据在存储介质中，以连续存储的形式存在。
   - 2）查询时，不必把所有列都查询出来，只需在特定列做 I/O 即可，效率节省 90%。
   - 3）存储时，列式数据库在每列上，还有专门的列压缩算法进一步提高数据库性能，这是行式数据库不具备的。

![1662120149473](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1662120149473.png)

### 1.3. HBase 应用场景？

| 场景                   | 解释                                                         |
| ---------------------- | ------------------------------------------------------------ |
| 半结构、非结构化的数据 | 对于数据结构字段不确定、杂乱无章、很难按一个概念进行抽取时，适合使用 HBase 进行存储，比如文章 tag 标签，会不断地增加、删除 |
| 记录非常稀疏的数据     | HBase 中 null 值的列不会被存储，既节省空间、又提高了读性能，而 MySQL 中 null 值占 1 个字节 |
| 多版本数据             | HBase 中列值支持多版本记录，而 MySQL 要想有版本支持则只能多存几行 |
| 超大数据量             | 大数据量时，MySQL 需要分库、分表、读写分离等操作，而 HBase 天然支持扩展 |

### 1.4. HBase 优缺点？

| 优点                                             | 缺点                                                         |
| ------------------------------------------------ | ------------------------------------------------------------ |
| 适合版半结构化、非结构化的数据                   | 不支持 SQL 数据分析，由于属于 NoSQL 数据库，所以不支持 SQL 语法 |
| 适合记录稀疏的数据                               | 不擅长多条件查询，在 HBase 中根据多列组合查询，相当于全表扫描，效率不高 |
| 支持多版本                                       | 不适合大范围扫描查询，大范围扫描效率不高，数据量过大时容易导致扫描超时、失败 |
| 支持海量数据存储                                 |                                                              |
| 支持动态列，可随时动态增减列数，而无需修改表结构 |                                                              |
| 高可靠                                           |                                                              |
| 高性能                                           |                                                              |

### 1.5. HBase 逻辑存储模型？

HBase，是三维有序存储的，即 `rowKey`，`column key(columnFamily + column)`、和 `timestamp` ，先按`rowKey` 升序排序，`rowkey` 相同的，则再按 `column key` 升序排序，而 `rowkey`、`column key` 都相同的，则按 `timestamp` 降序排序。

![1662095793429](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1662095793429.png)

| 名词         | 解释                                                         | 类比 MySQL 中的              |
| ------------ | ------------------------------------------------------------ | ---------------------------- |
| namespace    | 命名空间                                                     | database                     |
| table        | 表                                                           | table                        |
| row          | 行                                                           | row                          |
| rowKey       | 行键，每行数据必须存在，可以依据 r owkey 实现亿级别的毫秒级查询 | 主键，主键不是必须显式声明的 |
| columnFamily | 列族，一系列的集合，创建表时必须要定义                       | -                            |
| column       | 列，创建表时，列不能提前定义，添加数据时再动态指定列         | 列                           |
| timestamp    | 时间戳，64 位整型，插入数据时默认带有时间戳，无需建表时指定，可自动也可手动赋值，不同版本的数据会按照时间戳倒序排序，即最新的数据会排在前面 | -                            |
| dataType     | 字节数组，HBase 只有一种数据类型，任何数据存储时，都会统一转换为字节数组 | int、varchar、date 等        |

### 1.6. HBase 安装部署？

HBase 依赖 Zookeeper，所以先安装 ZK。

#### 1）安装 Zookeeper

##### 1、上传、解压

```shell
tar -zxvf apache-zookeeper-3.5.8-bin.tar.gz 
```

##### 2、修改配置文件

```shell
[root@bigdata01 conf]# pwd
/data/soft/apache-zookeeper-3.5.8-bin/conf
[root@bigdata01 conf]# ll
总用量 16
-rw-r--r--. 1 root root  535 5月   4 2020 configuration.xsl
-rw-r--r--. 1 root root 2712 5月   4 2020 log4j.properties
-rw-r--r--. 1 root root  922 5月   4 2020 zoo_sample.cfg

[root@bigdata01 soft]# cd apache-zookeeper-3.5.8-bin/conf/
[root@bigdata01 conf]# mv zoo_sample.cfg  zoo.cfg
[root@bigdata01 conf]# vi zoo.cfg
dataDir=/data/soft/apache-zookeeper-3.5.8-bin/data
server.0=bigdata01:2888:3888
server.1=bigdata02:2888:3888
server.2=bigdata03:2888:3888
```

##### 3、创建 myid 文件

```shell
[root@bigdata01 conf]#cd /data/soft/apache-zookeeper-3.5.8-bin
[root@bigdata01 apache-zookeeper-3.5.8-bin]# mkdir data
[root@bigdata01 apache-zookeeper-3.5.8-bin]# cd data
[root@bigdata01 data]# echo 0 > myid 
```

##### 4、拷贝配置到其他节点

```shell
[root@bigdata01 soft]# scp -rq apache-zookeeper-3.5.8-bin bigdata02:/data/soft/
[root@bigdata01 soft]# scp -rq apache-zookeeper-3.5.8-bin bigdata03:/data/soft/

[root@bigdata02 data]# pwd
/data/soft/apache-zookeeper-3.5.8-bin/data
[root@bigdata02 data]# vim myid
1

[root@bigdata02 data]# pwd
/data/soft/apache-zookeeper-3.5.8-bin/data
[root@bigdata02 data]# vim myid
2
```

##### 5、启动 Zookeeper 集群

```shell
[root@bigdata01 bin]# zkServer.sh start
ZooKeeper JMX enabled by default
Using config: /data/soft/apache-zookeeper-3.5.8-bin/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED

[root@bigdata02 bin]# zkServer.sh start
ZooKeeper JMX enabled by default
Using config: /data/soft/apache-zookeeper-3.5.8-bin/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED

[root@bigdata03 bin]# zkServer.sh start
ZooKeeper JMX enabled by default
Using config: /data/soft/apache-zookeeper-3.5.8-bin/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
```

##### 6、验证 Zookeeper 集群

```shell
[root@bigdata01 bin]# jps
3810 Jps
3719 QuorumPeerMain

[root@bigdata02 bin]# jps
2723 QuorumPeerMain
2776 Jps

[root@bigdata03 bin]# jps
2764 Jps
2718 QuorumPeerMain

[root@bigdata01 bin]# zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /data/soft/apache-zookeeper-3.5.8-bin/bin/../conf/zoo.cfg
Client port found: 2181. Client address: localhost.
Mode: follower

[root@bigdata02 bin]# zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /data/soft/apache-zookeeper-3.5.8-bin/bin/../conf/zoo.cfg
Client port found: 2181. Client address: localhost.
Mode: leader

[root@bigdata03 bin]# zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /data/soft/apache-zookeeper-3.5.8-bin/bin/../conf/zoo.cfg
Client port found: 2181. Client address: localhost.
Mode: follower
```

##### 7、停止 Zookeeper 集群

```shell
[root@bigdata01 bin]# zkServer.sh stop
ZooKeeper JMX enabled by default
Using config: /data/soft/apache-zookeeper-3.5.8-bin/bin/../conf/zoo.cfg
Stopping zookeeper ... STOPPED

[root@bigdata02 bin]# zkServer.sh stop
ZooKeeper JMX enabled by default
Using config: /data/soft/apache-zookeeper-3.5.8-bin/bin/../conf/zoo.cfg
Stopping zookeeper ... STOPPED

[root@bigdata03 bin]# zkServer.sh stop
ZooKeeper JMX enabled by default
Using config: /data/soft/apache-zookeeper-3.5.8-bin/bin/../conf/zoo.cfg
Stopping zookeeper ... STOPPED
```

#### 2）安装 HBase

HBase 也分为伪分布集群（即多个进程都落在同一台机器上），以及分布式集群，这里采用分布式集群的部署。

##### 1、上传、解压

```shell
tar -zxvf hbase-2.2.7-bin.tar.gz
```

##### 2、设置脚本的环境变量

```shell
[root@bigdata01 hbase-2.2.7]# pwd
/data/soft/hbase-2.2.7
[root@bigdata01 conf]# ll
总用量 44
-rw-r--r--. 1 root root 1811 1月  22 2020 hadoop-metrics2-hbase.properties
-rw-r--r--. 1 root root 4284 1月  22 2020 hbase-env.cmd
-rw-r--r--. 1 root root 7536 1月  22 2020 hbase-env.sh
-rw-r--r--. 1 root root 2257 1月  22 2020 hbase-policy.xml
-rw-r--r--. 1 root root 2301 1月  22 2020 hbase-site.xml
-rw-r--r--. 1 root root 1169 1月  22 2020 log4j-hbtop.properties
-rw-r--r--. 1 root root 4977 1月  22 2020 log4j.properties
-rw-r--r--. 1 root root   10 1月  22 2020 regionservers

# 行末追加配置
[root@bigdata01 conf]# cp hbase-env.sh hbase-env.sh_bak_20220902
[root@bigdata01 conf]# vim hbase-env.sh
export JAVA_HOME=/data/soft/jdk1.8
export HADOOP_HOME=/data/soft/hadoop-3.2.0
export HBASE_MANAGES_ZK=false
export HBASE_LOG_DIR=/data/hbase/log
```

##### 3、修改配置文件

`hbase-site.xml`

```xml
	<!-- 修改原有配置 -->
    <property>
        <name>hbase.cluster.distributed</name>
        <value>true</value>
    </property>
    <property>
        <name>hbase.tmp.dir</name>
        <value>/data/hbase/tmp</value>
    </property>
    <property>
      
	<!-- 追加新的配置 -->
	<!-- 设置HBase表数据，也就是HBase数据在hdfs上的存储根目录 -->
    <property>
        <name>hbase.rootdir</name>
        <value>hdfs://bigdata01:9000/hbase</value>
    </property>
	<!-- zookeeper集群的URL配置，多个host中间用逗号隔开 -->
    <property>
        <name>hbase.zookeeper.quorum</name>
        <value>bigdata01,bigdata02,bigdata03</value>
    </property>
    <!-- HBase在zookeeper上数据的根目录znode节点 -->
    <property>
        <name>zookeeper.znode.parent</name>
        <value>/hbase</value>
    </property>
    <!-- 设置zookeeper通信端口，不配置也可以，zookeeper默认就是2181 -->
    <property>
        <name>hbase.zookeeper.property.clientPort</name>
        <value>2181</value>
    </property>
```

##### 4、配置从点节点主机名

```shell
[root@bigdata01 conf]# vim regionservers 
bigdata02
bigdata03
```

##### 5、拷贝配置到其他节点

```shell
[root@bigdata01 soft]# scp -rq hbase-2.2.7 bigdata02:/data/soft/
[root@bigdata01 soft]# scp -rq hbase-2.2.7 bigdata03:/data/soft/
```

##### 6、检查 Hadoop、ZK 集群是否正常

```shell
[root@bigdata01 soft]# jps
2451 ResourceManager
3940 Jps
3719 QuorumPeerMain
1928 NameNode
2203 SecondaryNameNode

[root@bigdata02 bin]# jps
2723 QuorumPeerMain
1784 DataNode
1898 NodeManager
2907 Jps

[root@bigdata03 soft]# jps
1784 DataNode
1898 NodeManager
2718 QuorumPeerMain
2895 Jps
```

##### 7、启动 HBase 集群

```shell
[root@bigdata01 bin]# pwd
/data/soft/hbase-2.2.7/bin
[root@bigdata01 bin]# start-hbase.sh 
```

##### 8、验证 HBase 集群

http://bigdata01:16010/master-status

```shell
[root@bigdata01 bin]# jps
2451 ResourceManager
4148 HMaster
3719 QuorumPeerMain
1928 NameNode
2203 SecondaryNameNode
4444 Jps

[root@bigdata02 bin]# jps
2723 QuorumPeerMain
3235 Jps
1784 DataNode
1898 NodeManager
2972 HRegionServer

[root@bigdata03 soft]# jps
2962 HRegionServer
1784 DataNode
1898 NodeManager
3212 Jps
2718 QuorumPeerMain
```

##### 9、停止 HBase 集群

```shell
[root@bigdata01 bin]# stop-hbase.sh 
stopping hbase..........
...
```

### 1.7. HBase 操作使用？

#### 1）Shell 命令

进行 shell 命令行

```shell
[root@bigdata01 bin]# pwd
/data/soft/hbase-2.2.7/bin
[root@bigdata01 bin]# hbase shell
...             
hbase(main):001:0> 
```

##### ① 基础命令

###### 1、查询集群状态

```shell
hbase(main):001:0> status
1 active master, 0 backup masters, 2 servers, 0 dead, 1.0000 average load
Took 0.6544 seconds  
```

###### 2、查看 HBase 版本

```shell
hbase(main):002:0> version
2.2.7, r0fc18a9056e5eb3a80fdbde916865607946c5195, 2021年 04月 11日 星期日 19:24:57 CST
Took 0.0002 seconds  
```

###### 3、查看当前用户

```shell
hbase(main):003:0> whoami
root (auth:SIMPLE)
    groups: root
Took 0.0077 seconds
```

###### 4、查看所有命令空间

namespace 相当于 MySQL 中的 database，hbase 存放系统表，，default 存放用户表

```shell
hbase(main):051:0> list_namespace
NAMESPACE                           
default                           
hbase                            
2 row(s)
Took 0.2818 seconds
```

###### 5、创建命令空间

```shell
hbase(main):052:0> create_namespace 'n1'
Took 0.2729 seconds 

hbase(main):053:0> list_namespace
NAMESPACE                    
default                             
hbase                             
n1                           
3 row(s)
Took 0.0435 seconds    
```

###### 6、查看 n1 命令空间下的所有表

```shell
hbase(main):056:0> list_namespace_tables 'n1'
TABLE                             
t1                           
1 row(s)
Took 0.0170 seconds                                         
=> ["t1"]
```

##### ② DDL 命令

###### 1、创建表

```shell
# 创建student表，后面的全是列族：info、level，默认存放在default命令空间中
hbase(main):004:0> create 'student', 'info', 'level'
Created table student
Took 2.3748 seconds      
=> Hbase::Table - student

# 在n1命令空间创建t1表
hbase(main):054:0> create 'n1:t1', 'info', 'level'
Created table n1:t1
Took 1.2319 seconds                      
=> Hbase::Table - n1:t1

# 为t3#cf1列族，设置ttl过期时间
hbase(main):002:0> create 't3', { NAME => 'cf1', TTL => '18000' }
Created table t3
Took 1.5321 seconds
=> Hbase::Table - t3

# 为t4#cf1列族，设置最大版本个数
hbase(main):004:0> create 't4', { NAME => 'cf1', VERSIONS => 3 }
Created table t4
Took 1.2669 seconds                  
=> Hbase::Table - t4

# 为t5#cf1列族，设置SNAPPY压缩
hbase(main):006:0> create 't5', { NAME => 'cf1', COMPRESSION => 'SNAPPY' }
Created table t5
Took 1.2958 seconds              
=> Hbase::Table - t5

# 为t6#cf1列族，设置数据块大小（默认65536）
# 随机查询时，数据块设置得越小，加载到内存的索引越大，查询性能越高
# 而顺序查询时，数据块设置得越大，加载到内存的数据越多，磁盘I/O越少，查询性能越高
hbase(main):008:0> create 't6', { NAME => 'cf1', BLOCKSIZE => '65537' }
Created table t6
Took 1.2577 seconds                
=> Hbase::Table - t6

# 为t7#cf1列族，关闭数据块缓存（默认开启），对于那些查询很少的列族，关闭缓存可以节省内存
hbase(main):011:0> create 't7', { NAME => 'cf1', BLOCKCACHE => 'false' }
Created table t7
Took 2.2431 seconds                         
=> Hbase::Table - t7

# 为t8#cf1列族，设置列级布隆过滤器（默认为行级ROW）
# 行级：只检查rowkey一定不存在
# 列级：同时检查rowkey和列标识符一定不存在
hbase(main):012:0> create 't8', { NAME => 'cf1', BLOOMFILTER => 'ROWCOL' }
Created table t8
Took 0.7472 seconds                 
=> Hbase::Table - t8
```

###### 2、查看所有表

```shell
hbase(main):005:0> list
TABLE                       
student               
1 row(s)
Took 0.0308 seconds                      
=> ["student"]

# 查看存在n1:t1表
hbase(main):055:0> list 
TABLE                       
student                             
t2                         
n1:t1                             
3 row(s)
Took 0.0079 seconds                             
=> ["student", "t2", "n1:t1"]
```

###### 3、禁用表

用于在 Region 分裂过程中禁用，保证数据安全性

```shell
hbase(main):006:0> disable 'student'
Took 0.8546 seconds 

hbase(main):007:0> list
TABLE                       
student                    
1 row(s)
```

###### 4、验证表是否被禁用

```shell
hbase(main):008:0> is_disabled 'student'
true                
Took 0.0338 seconds                               
=> 1
```

###### 5、启用表

```shell
hbase(main):009:0> enable 'student'
Took 0.7617 seconds 

hbase(main):010:0> list
TABLE                         
student                          
1 row(s)
Took 0.0304 seconds                        
=> ["student"]
```

###### 6、验证表是否已启用

```shell
hbase(main):011:0> is_enabled 'student'
true                             
Took 0.0103 seconds                            
=> true
```

###### 7、查看表详细信息

```shell
hbase(main):012:0> desc 'student'
Table student is ENABLED                  
student                 
COLUMN FAMILIES DESCRIPTION                            
{NAME => 'info', VERSIONS => '1', NEW_VERSION_BEHAVIOR => 'false', KEEP_DELETED_CELLS => 'FALSE', DATA_BLOCK_ENCODING => 'NONE', TTL => 'FOREVER', MIN_VERSIONS => '0', REPLICATION_SCOPE => '0', BLOOMFILTER 
=> 'ROW', IN_MEMORY => 'false', COMPRESSION => 'NONE', BLOCKCACHE => 'true', BLOCKSIZE => '65536'}  

{NAME => 'level', VERSIONS => '1', NEW_VERSION_BEHAVIOR => 'false', KEEP_DELETED_CELLS => 'FALSE', DATA_BLOCK_ENCODING => 'NONE', TTL => 'FOREVER', MIN_VERSIONS => '0', REPLICATION_SCOPE => '0', BLOOMFILTER
 => 'ROW', IN_MEMORY => 'false', COMPRESSION => 'NONE', BLOCKCACHE => 'true', BLOCKSIZE => '65536'} 
 
2 row(s)
Quota is disabled
Took 0.0470 seconds
```

###### 8、修改表结构

```shell
# 修改student#level列族，最多支持3个历史版本
hbase(main):013:0> alter 'student', {NAME => 'level', VERSIONS => '3'}
Updating all regions with the new schema...
1/1 regions updated.
Done.
Took 2.1733 seconds    

hbase(main):014:0> desc 'student'
Table student is ENABLED                          
student                 
COLUMN FAMILIES DESCRIPTION                              
{NAME => 'info', VERSIONS => '1', NEW_VERSION_BEHAVIOR => 'false', KEEP_DELETED_CELLS => 'FALSE', DATA_BLOCK_ENCODING => 'NONE', TTL => 'FOREVER', MIN_VERSIONS => '0', REPLICATION_SCOPE => '0', BLOOMFILTER 
=> 'ROW', IN_MEMORY => 'false', COMPRESSION => 'NONE', BLOCKCACHE => 'true', BLOCKSIZE => '65536'}                                                                                                            
{NAME => 'level', VERSIONS => '3', NEW_VERSION_BEHAVIOR => 'false', KEEP_DELETED_CELLS => 'FALSE', DATA_BLOCK_ENCODING => 'NONE', TTL => 'FOREVER', MIN_VERSIONS => '0', REPLICATION_SCOPE => '0', BLOOMFILTER
 => 'ROW', IN_MEMORY => 'false', COMPRESSION => 'NONE', BLOCKCACHE => 'true', BLOCKSIZE => '65536'}
 
2 row(s)
Quota is disabled
Took 0.0526 seconds 

# 增加student#about列族
hbase(main):015:0> alter 'student', 'about'
Updating all regions with the new schema...
1/1 regions updated.
Done.
Took 2.2224 seconds

hbase(main):016:0> desc 'student'
Table student is ENABLED                         
student           
COLUMN FAMILIES DESCRIPTION                               
{NAME => 'about', VERSIONS => '1', NEW_VERSION_BEHAVIOR => 'false', KEEP_DELETED_CELLS => 'FALSE', DATA_BLOCK_ENCODING => 'NONE', TTL => 'FOREVER', MIN_VERSIONS => '0', REPLICATION_SCOPE => '0', BLOOMFILTER
 => 'ROW', IN_MEMORY => 'false', COMPRESSION => 'NONE', BLOCKCACHE => 'true', BLOCKSIZE => '65536'}                                                                                                           
{NAME => 'info', VERSIONS => '1', NEW_VERSION_BEHAVIOR => 'false', KEEP_DELETED_CELLS => 'FALSE', DATA_BLOCK_ENCODING => 'NONE', TTL => 'FOREVER', MIN_VERSIONS => '0', REPLICATION_SCOPE => '0', BLOOMFILTER 
=> 'ROW', IN_MEMORY => 'false', COMPRESSION => 'NONE', BLOCKCACHE => 'true', BLOCKSIZE => '65536'}                                                                                                            
{NAME => 'level', VERSIONS => '3', NEW_VERSION_BEHAVIOR => 'false', KEEP_DELETED_CELLS => 'FALSE', DATA_BLOCK_ENCODING => 'NONE', TTL => 'FOREVER', MIN_VERSIONS => '0', REPLICATION_SCOPE => '0', BLOOMFILTER
 => 'ROW', IN_MEMORY => 'false', COMPRESSION => 'NONE', BLOCKCACHE => 'true', BLOCKSIZE => '65536'}                                                                                                           
3 row(s)
Quota is disabled
Took 0.0520 seconds 

# 删除student#about列族，但每张表最少要保留一个列族
hbase(main):017:0> alter 'student', { NAME => 'about', METHOD => 'delete' }
Updating all regions with the new schema...
1/1 regions updated.
Done.
Took 2.2309 seconds

hbase(main):018:0> desc 'student'
Table student is ENABLED                           
student          
COLUMN FAMILIES DESCRIPTION                             
{NAME => 'info', VERSIONS => '1', NEW_VERSION_BEHAVIOR => 'false', KEEP_DELETED_CELLS => 'FALSE', DATA_BLOCK_ENCODING => 'NONE', TTL => 'FOREVER', MIN_VERSIONS => '0', REPLICATION_SCOPE => '0', BLOOMFILTER 
=> 'ROW', IN_MEMORY => 'false', COMPRESSION => 'NONE', BLOCKCACHE => 'true', BLOCKSIZE => '65536'}                                                                                                            
{NAME => 'level', VERSIONS => '3', NEW_VERSION_BEHAVIOR => 'false', KEEP_DELETED_CELLS => 'FALSE', DATA_BLOCK_ENCODING => 'NONE', TTL => 'FOREVER', MIN_VERSIONS => '0', REPLICATION_SCOPE => '0', BLOOMFILTER
 => 'ROW', IN_MEMORY => 'false', COMPRESSION => 'NONE', BLOCKCACHE => 'true', BLOCKSIZE => '65536'}                                                                                                           
2 row(s)
Quota is disabled
Took 0.0772 seconds 
```

###### 9、验证表是否存在

```shell
hbase(main):019:0> exists 'student'
Table student does exist                               
Took 0.0079 seconds                
=> true
```

###### 10、删除表

```shell
# 建立t1表
hbase(main):021:0> create 't1', 'info'
Created table t1
Took 0.7281 seconds                               
=> Hbase::Table - t1

# 直接删除t1表：报错！需要先禁用表
hbase(main):022:0> drop 't1'
ERROR: Table t1 is enabled. Disable it first.

For usage try 'help "drop"'

Took 0.0147 seconds

# 禁用t1表
hbase(main):023:0> disable 't1'
Took 0.7486 seconds
# 再删除t1表：删除成功
hbase(main):024:0> drop 't1'
Took 0.2370 seconds    
```

###### 11、清空表数据

```shell
# 建立t2表
hbase(main):028:0> create 't2', 'info'
Created table t2
Took 1.2314 seconds                              
=> Hbase::Table - t2

# 清空t2表的数据
hbase(main):029:0> truncate 't2'
Truncating 't2' table (it may take a while):
Disabling table...
Truncating table...
Took 1.4857 seconds

# t2表是先被禁用->清空数据->再重新启用
hbase(main):030:0> is_enabled 't2'
true                            
Took 0.0123 seconds                         
=> true

# t2表还是存在的
hbase(main):031:0> list
TABLE                       
student                              
t2                            
2 row(s)
Took 0.0062 seconds                                    
=> ["student", "t2"]
```

##### ③ CURD 命令

###### 1、添加数据

```shell
# 添加jack、tom两条数据，格式为：put 表，rowkey，列族:列，值
hbase(main):032:0> put 'student', 'jack', 'info:sex', 'man'
Took 0.1497 seconds
hbase(main):033:0> put 'student', 'jack', 'info:age', '22'
Took 0.0054 seconds                 
hbase(main):034:0> put 'student', 'jack', 'level:class', 'A'
Took 0.0115 seconds              
hbase(main):035:0> put 'student', 'tom', 'level:sex', 'woman'
Took 0.0070 seconds              
hbase(main):036:0> put 'student', 'tom', 'info:age', '20'
Took 0.0205 seconds              
hbase(main):037:0> put 'student', 'tom', 'level:class', 'B'
```

###### 2、修改数据

```shell
# 更新（覆盖）表数据
hbase(main):038:0> put 'student', 'tom', 'level:class', 'C'
Took 0.0080 seconds
```

###### 3、查看数据

```shell
# 查看所有rowkey#所有列族信息：格式，get 表，rowkey
hbase(main):039:0> get 'student', 'jack'
COLUMN                                               CELL                           
 info:age                                            timestamp=1662205384206, value=22
 info:sex                                            timestamp=1662205352567, value=man                                   
 level:class                                         timestamp=1662205403542, value=A                  
1 row(s)
Took 0.0420 seconds  

hbase(main):040:0> get 'student', 'tom'
COLUMN                                               CELL                            
 info:age                                            timestamp=1662205429127, value=20 
 level:class                                         timestamp=1662205529529, value=C 
 level:sex                                           timestamp=1662205421347, value=woman                                                                                                                     
1 row(s)
Took 0.0187 seconds  

# 单独查看所有rowkey#info列族信息：格式，get 表，rowkey，列族
hbase(main):042:0> get 'student', 'jack', 'info'
COLUMN                                               CELL                             
 info:age                                            timestamp=1662205384206, value=22 
 info:sex                                            timestamp=1662205352567, value=man                                                                                                                       
1 row(s)
Took 0.0061 seconds 

# 单独查看所有rowkey#info列族#age列信息：格式，get 表，rowkey，列族，列
hbase(main):043:0> get 'student', 'jack', 'info:age'
COLUMN                                               CELL                          
 info:age                                            timestamp=1662205384206, value=22                   
1 row(s)
Took 0.0162 seconds 
```

###### 4、查看表数据量

```shell
hbase(main):044:0> count 'student'
2 row(s)
Took 0.0682 seconds                         
=> 2
```

###### 5、全表扫描

```shell
# 全表扫描
hbase(main):045:0> scan 'student'
ROW                                                  COLUMN+CELL
 jack                                                column=info:age, timestamp=1662205384206, value=22             
 jack                                                column=info:sex, timestamp=1662205352567, value=man                 
 jack                                                column=level:class, timestamp=1662205403542, value=A              
 tom                                                 column=info:age, timestamp=1662205429127, value=20                 
 tom                                                 column=level:class, timestamp=1662205529529, value=C              
 tom                                                 column=level:sex, timestamp=1662205421347, value=woman                                                                                                   
2 row(s)
Took 0.0143 seconds
```

###### 6、删除数据

```shell
# 删除student#info#age列最新版本的数据
hbase(main):046:0> delete 'student', 'jack', 'info:age'
Took 0.0117 seconds

# 查看删除结果：jack没有age列数据了
hbase(main):047:0> scan 'student'
ROW                                                  COLUMN+CELL
 jack                                                column=info:sex, timestamp=1662205352567, value=man
 jack                                                column=level:class, timestamp=1662205403542, value=A              
 tom                                                 column=info:age, timestamp=1662205429127, value=20                 
 tom                                                 column=level:class, timestamp=1662205529529, value=C             
 tom                                                 column=level:sex, timestamp=1662205421347, value=woman                                                                                                   
2 row(s)
Took 0.0077 seconds    

# 删除student#jack#info#age#最新2个版本的数据
hbase(main):048:0> delete 'student', 'jack', 'info:age', 2
Took 0.0402 seconds

# delete操作不能跨列族，如果需要删除整个rowkey的数据，可以使用deleteall
hbase(main):049:0> deleteall 'student', 'jack'
Took 0.0078 seconds
hbase(main):050:0> scan 'student'
ROW                                                  COLUMN+CELL
 tom                                                 column=info:age, timestamp=1662205429127, value=20                          
 tom                                                 column=level:class, timestamp=1662205529529, value=C                  
 tom                                                 column=level:sex, timestamp=1662205421347, value=woman                                                                                                   
1 row(s)
Took 0.0093 seconds  
```

#### 2）Java API

##### 1、获取 HBase 连接

```java
package com.bigdata.hbase;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.client.Connection;
import org.apache.hadoop.hbase.client.ConnectionFactory;

import java.io.IOException;

/**
 * 操作HBase
 *
 * @author yaocs2
 * @since 2022-09-03
 */
public class HBaseOp {

    public static void main(String[] args) throws IOException {
        // 获取HBase连接
        Configuration conf = HBaseConfiguration.create();
        conf.set("hbase.zookeeper.quorum", "bigdata01:2181,bigdata02:2181,bigdata03:2181");
        conf.set("hbase.rootdir", "hdfs://bigdata01:9000/hbase");
        Connection conn = ConnectionFactory.createConnection(conf);

        // 关闭HBase连接
        conn.close();
    }
}
```

##### 2、添加数据

```java
    /**
     * 添加数据
     *
     * @param conn
     * @throws IOException
     */
    private static void put(Connection conn) throws IOException {
        // 获取已经创建好了的student表
        Table table = conn.getTable(TableName.valueOf("student"));

        // 指定rowkey
        Put put = new Put(Bytes.toBytes("laowang"));

        // 指定列族、列、值
        put.addColumn(Bytes.toBytes("info"), Bytes.toBytes("age"), Bytes.toBytes("18"));
        put.addColumn(Bytes.toBytes("info"), Bytes.toBytes("sex"), Bytes.toBytes("man"));
        put.addColumn(Bytes.toBytes("level"), Bytes.toBytes("class"), Bytes.toBytes("A"));

        // 向表中添加数据
        table.put(put);

        // 关闭table连接
        table.close();
    }
```

##### 3、查询数据

```java
    /**
     * 查询数据
     * 
     * @param conn
     * @throws IOException
     */
    private static void get(Connection conn) throws IOException {
        // 获取已经创建好了的student表
        Table table = conn.getTable(TableName.valueOf("student"));

        // 指定rowkey
        Get get = new Get(Bytes.toBytes("laowang"));

        // 指定列族、列、值
        get.addColumn(Bytes.toBytes("info"), Bytes.toBytes("age"));
        get.addColumn(Bytes.toBytes("info"), Bytes.toBytes("sex"));

        // 查询数据
        Result result = table.get(get);
        byte[] age_bytes = result.getValue(Bytes.toBytes("info"), Bytes.toBytes("age"));
        byte[] sex_bytes = result.getValue(Bytes.toBytes("info"), Bytes.toBytes("sex"));
        System.out.println(String.format("age: %s, sex: %s", new String(age_bytes), new String(sex_bytes)));

        // 迭代所有列族、列、值
        List<Cell> cells = result.listCells();
        for (Cell cell : cells) {
            // 列族
            byte[] family_bytes = CellUtil.cloneFamily(cell);
            // 列
            byte[] column_bytes = CellUtil.cloneQualifier(cell);
            // 值
            byte[] value_bytes = CellUtil.cloneValue(cell);
            System.out.println(String.format("列族: %s, 列: %s, 值: %s", new String(family_bytes), new String(column_bytes), new String(value_bytes)));
        }

        // 关闭table连接
        table.close();
    }
```

##### 4、查询多版本数据

```java
    /**
     * 查询多版本数据
     *
     * alter 'student', { NAME => 'info', VERSIONS => 3 }
     *
     * put 'student', 'laowang', 'info:age', '18'
     * put 'student', 'laowang', 'info:age', '19'
     * put 'student', 'laowang', 'info:age', '20'
     */
    private static void getMoreVersions(Connection conn) throws IOException {
        // 获取已经创建好了的student表
        Table table = conn.getTable(TableName.valueOf("student"));

        // 指定rowkey
        Get get = new Get(Bytes.toBytes("laowang"));

        // 指定读取的版本个数
//        get.readVersions(2);// 读取2个版本数据
        get.readAllVersions();

        // 查询数据
        Result result = table.get(get);

        // 迭代所有列族、列、值
        List<Cell> cells = result.listCells();
        for (Cell cell : cells) {
            // 时间戳
            long timestamp = cell.getTimestamp();
            // 列族
            byte[] family_bytes = CellUtil.cloneFamily(cell);
            // 列
            byte[] column_bytes = CellUtil.cloneQualifier(cell);
            // 值
            byte[] value_bytes = CellUtil.cloneValue(cell);
            System.out.println(String.format("时间戳: %s, 列族: %s, 列: %s, 值: %s", timestamp, new String(family_bytes), new String(column_bytes), new String(value_bytes)));
        }

        // 关闭table连接
        table.close();
    }
```

##### 5、全表扫描

不过全表扫描性能不高

![1662305713846](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1662305713846.png)

```java
    /**
     * 全表扫描scan+filter
     *
     * @param conn
     */
    private static void scanFilter(Connection conn) throws IOException {
        // 获取已经创建好了的student表
        Table table = conn.getTable(TableName.valueOf("student"));

        // 全表扫描
        Scan scan = new Scan();

        // 指定列族查询区间：左闭右开
        scan.withStartRow(Bytes.toBytes("info"));
        scan.withStopRow(Bytes.toBytes("sex"));

        // 指定filter
        RowFilter rowFilter = new RowFilter(CompareOperator.LESS_OR_EQUAL, new BinaryComparator(Bytes.toBytes("laowang")));
        scan.setFilter(rowFilter);

        // 获取查询结果
        ResultScanner scanner = table.getScanner(scan);
        for (Result result : scanner) {
            byte[] rowKey_bytes = result.getRow();

            // 迭代所有列族、列、值
            List<Cell> cells = result.listCells();
            for (Cell cell : cells) {
                // 列族
                byte[] family_bytes = CellUtil.cloneFamily(cell);
                // 列
                byte[] column_bytes = CellUtil.cloneQualifier(cell);
                // 值
                byte[] value_bytes = CellUtil.cloneValue(cell);
                System.out.println(String.format("rowkey: %s, 列族: %s, 列: %s, 值: %s", new String(rowKey_bytes), new String(family_bytes), new String(column_bytes), new String(value_bytes)));
            }

            System.out.println("=================================================");
        }

        // 关闭table连接
        table.close();
    }
```

##### 6、删除数据

```java
    /**
     * 删除数据
     *
     * @param conn
     * @throws IOException
     */
    private static void delete(Connection conn) throws IOException {
        // 获取已经创建好了的student表
        Table table = conn.getTable(TableName.valueOf("student"));

        // 指定rowkey
        Delete delete = new Delete(Bytes.toBytes("laowang"));

        // 指定列族、列、值, 不指定时默认删除整个rowkey的数据
//        delete.addColumn(Bytes.toBytes("info"), Bytes.toBytes("age"));// 删除student#info#age列最新版本的数据

        // 删除数据
        table.delete(delete);

        // 关闭table连接
        table.close();
    }
```

##### 7、创建表

```java
    /**
     * 创建表
     * 
     * @param conn
     * @throws IOException
     */
    private static void createTable(Connection conn) throws IOException {
        // 构造列族描述信息
        ColumnFamilyDescriptor infoFamily = ColumnFamilyDescriptorBuilder
                .newBuilder(Bytes.toBytes("info"))
                .setMaxVersions(3)
                .build();
        ColumnFamilyDescriptor levelFamily = ColumnFamilyDescriptorBuilder
                .newBuilder(Bytes.toBytes("level"))
                .setMaxVersions(2)
                .build();
        List<ColumnFamilyDescriptor> familyList = new ArrayList<ColumnFamilyDescriptor>();
        familyList.add(infoFamily);
        familyList.add(levelFamily);

        // 构造表描述信息
        TableDescriptor tableDesc = TableDescriptorBuilder
                .newBuilder(TableName.valueOf("test"))
                .setColumnFamilies(familyList)
                .build();

        // 创建表
        Admin admin = conn.getAdmin();
        admin.createTable(tableDesc);

        // 关闭admin连接
        admin.close();
    }
```

##### 8、删除表

```java
    /**
     * 先禁用表, 再删除表
     * 
     * @param conn
     * @throws IOException
     */
    private static void deleteTable(Connection conn) throws IOException {
        Admin admin = conn.getAdmin();
        admin.disableTable(TableName.valueOf("test"));
        admin.deleteTable(TableName.valueOf("test"));
    }
```

##### 9、批量导入

###### 1）MapReduce + Put

```java
package com.bigdata.hbase;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.hbase.client.Put;
import org.apache.hadoop.hbase.mapreduce.TableMapReduceUtil;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;

import java.io.IOException;

/**
 * HBase批量导入之 MapReduce#put
 * <p>
 * 原始数据：
 * [root@bigdata01 ~]# more hbase_import.dat
 * a       c1      name    zs
 * a       c1      age     18
 * b       c1      name    ls
 * b       c1      age     29
 * c       c1      name    ww
 * c       c1      age     31
 * <p>
 * 创建batch1表：
 * hbase(main):013:0> create 'batch1', 'c1'
 * <p>
 * 提交MapReduce任务：
 * hadoop jar hbase-1.0-SNAPSHOT-jar-with-dependencies.jar com.bigdata.hbase.BatchImportMR hdfs://bigdata01:9000/hbase_import.dat batch1
 *
 * @author yaocs2
 * @since 2022-09-05
 */
public class BatchImportMR {

    /**
     * 组装Job = Map + Reduce
     *
     * @param args
     */
    public static void main(String[] args) {
        if (args.length < 2) {
            System.exit(100);
        }

        String fileInputPath = args[0];
        String outputTableName = args[1];

        try {
            Configuration conf = new Configuration();
            conf.set("hbase.zookeeper.quorum", "bigdata01:2181,bigdata02:2181,bigdata03:2181");
            conf.set("hbase.rootdir", "hdfs://bigdata01:9000/hbase");

            Job job = Job.getInstance(conf, "Batch Import HBase Table: " + outputTableName);
            job.setJarByClass(BatchImportMR.class);// 必须设置, 否则提交到集群后, Job会找不到这个类的

            // 输入
            FileInputFormat.setInputPaths(job, new Path(fileInputPath));

            // Map
            job.setMapperClass(BatchImportMapper.class);
            job.setMapOutputKeyClass(NullWritable.class);
            job.setMapOutputValueClass(Put.class);

            // Reduce
            job.setNumReduceTasks(0);
            
            // 输出为表
            TableMapReduceUtil.initTableReducerJob(outputTableName, null, job);

            // 提交Job
            job.waitForCompletion(true);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    private static class BatchImportMapper extends Mapper<LongWritable, Text, NullWritable, Put> {
        @Override
        protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
            String[] strs = value.toString().split("\t");
            if (strs.length == 4) {
                String rowKey = strs[0];
                String columnFamily = strs[1];
                String column = strs[2];
                String readValue = strs[3];

                // 指定rowkey
                Put put = new Put(rowKey.getBytes());

                // 指定列族、列、值
                put.addColumn(columnFamily.getBytes(), column.getBytes(), readValue.getBytes());

                // map写入
                context.write(NullWritable.get(), put);
            }
        }
    }
}
```

###### 2）MapReduce + HFile + Bulkload

通过 MapReduce 直接生成 HFile，再把 HFile 批量转移到表对应的 Region 中，省去了大量 HBase#RPC 的过程。

![1662308645799](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1662308645799.png)

```java
package com.bigdata.hbase;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.hbase.TableName;
import org.apache.hadoop.hbase.client.Connection;
import org.apache.hadoop.hbase.client.ConnectionFactory;
import org.apache.hadoop.hbase.client.Put;
import org.apache.hadoop.hbase.io.ImmutableBytesWritable;
import org.apache.hadoop.hbase.mapreduce.HFileOutputFormat2;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

import java.io.IOException;

/**
 * HBase批量导入之 MapReduce+HFile+Bulkload
 * <p>
 * 原始数据：
 * [root@bigdata01 ~]# more hbase_import.dat
 * a       c1      name    zs
 * a       c1      age     18
 * b       c1      name    ls
 * b       c1      age     29
 * c       c1      name    ww
 * c       c1      age     31
 * <p>
 * 创建batch1表：
 * hbase(main):013:0> create 'batch2', 'c1'
 * <p>
 * 提交MapReduce任务, 生成HFile：
 * hadoop jar hbase-1.0-SNAPSHOT-jar-with-dependencies.jar com.bigdata.hbase.BatchImportBulkLoad hdfs://bigdata01:9000/hbase_import.dat hdfs://bigdata01:9000/hbase_import_out batch2 *
 *
 * BulkLoad批量转移HFile：
 * hbase org.apache.hadoop.hbase.tool.BulkLoadHFilesTool hdfs://bigdata01:9000/hbase_import_out batch2
 *
 * @author yaocs2
 * @since 2022-09-05
 */
public class BatchImportBulkLoad {

    /**
     * 组装Job = Map + Reduce
     *
     * @param args
     */
    public static void main(String[] args) {
        if (args.length < 2) {
            System.exit(100);
        }

        String fileInputPath = args[0];
        String fileOutputPath = args[1];
        String outputTableName = args[2];

        try {
            Configuration conf = new Configuration();
            conf.set("hbase.zookeeper.quorum", "bigdata01:2181,bigdata02:2181,bigdata03:2181");
            conf.set("hbase.rootdir", "hdfs://bigdata01:9000/hbase");

            Job job = Job.getInstance(conf, "Batch Import HBase Table: " + fileOutputPath);
            job.setJarByClass(BatchImportBulkLoad.class);// 必须设置, 否则提交到集群后, Job会找不到这个类的

            // 输入
            FileInputFormat.setInputPaths(job, new Path(fileInputPath));

            // 输出
            FileSystem fileSystem = FileSystem.get(conf);
            Path output = new Path(fileOutputPath);
            if (fileSystem.exists(output)) {
                fileSystem.delete(output, true);
            }
            FileOutputFormat.setOutputPath(job, output);

            // Map
            job.setMapperClass(BatchImportMapper.class);
            job.setMapOutputKeyClass(ImmutableBytesWritable.class);
            job.setMapOutputValueClass(Put.class);

            // Reduce
            job.setNumReduceTasks(0);

            // 生成HFile
            Connection conn = ConnectionFactory.createConnection(conf);
            TableName tableName = TableName.valueOf(outputTableName);
            HFileOutputFormat2.configureIncrementalLoad(job, conn.getTable(tableName), conn.getRegionLocator(tableName));

            // 提交Job
            job.waitForCompletion(true);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    private static class BatchImportMapper extends Mapper<LongWritable, Text, ImmutableBytesWritable, Put> {
        @Override
        protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
            String[] strs = value.toString().split("\t");
            if (strs.length == 4) {
                String rowKey = strs[0];
                String columnFamily = strs[1];
                String column = strs[2];
                String readValue = strs[3];

                // key
                ImmutableBytesWritable rowKeyWritable = new ImmutableBytesWritable(rowKey.getBytes());

                // 指定rowkey
                Put put = new Put(rowKey.getBytes());

                // 指定列族、列、值
                put.addColumn(columnFamily.getBytes(), column.getBytes(), readValue.getBytes());

                // map写入
                context.write(rowKeyWritable, put);
            }
        }
    }
}
```

##### 10、批量导出

###### 1）MapReduce + TableMapReduceUtil

导出格式自主可控

```java
package com.bigdata.hbase;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.hbase.Cell;
import org.apache.hadoop.hbase.client.Result;
import org.apache.hadoop.hbase.client.Scan;
import org.apache.hadoop.hbase.io.ImmutableBytesWritable;
import org.apache.hadoop.hbase.mapreduce.TableMapReduceUtil;
import org.apache.hadoop.hbase.mapreduce.TableMapper;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

import java.io.IOException;

/**
 * 批量导出之 TableMapReduceUtil
 * <p>
 * 提交MapReduce任务：
 * hadoop jar hbase-1.0-SNAPSHOT-jar-with-dependencies.jar com.bigdata.hbase.BatchExportTableMapReduceUtil batch1 hdfs://bigdata01:9000/batch1
 * <p>
 * 查看导出结果：
 * [root@bigdata01 ~]# hdfs dfs -cat /batch1/*
 * a       zs      18
 * b       ls      29
 * c       ww      31
 *
 * @author yaocs2
 * @since 2022-09-05
 */
public class BatchExportTableMapReduceUtil {

    /**
     * 组装Job = Map + Reduce
     *
     * @param args
     */
    public static void main(String[] args) {
        if (args.length < 2) {
            System.exit(100);
        }

        String inTableName = args[0];
        String fileOutputPath = args[1];

        try {
            Configuration conf = new Configuration();
            conf.set("hbase.zookeeper.quorum", "bigdata01:2181,bigdata02:2181,bigdata03:2181");
            conf.set("hbase.rootdir", "hdfs://bigdata01:9000/hbase");

            Job job = Job.getInstance(conf, "Batch Import HBase Table: " + fileOutputPath);
            job.setJarByClass(BatchExportTableMapReduceUtil.class);// 必须设置, 否则提交到集群后, Job会找不到这个类的

            // Map
            job.setMapperClass(BatchExportMapper.class);
            job.setMapOutputKeyClass(Text.class);
            job.setMapOutputValueClass(Text.class);

            // Reduce
            job.setNumReduceTasks(0);

            // 输入为表
            TableMapReduceUtil.initTableMapperJob(inTableName, new Scan(), BatchExportMapper.class, Text.class, Text.class, job);

            // 输出
            FileOutputFormat.setOutputPath(job, new Path(fileOutputPath));

            // 提交Job
            job.waitForCompletion(true);
        } catch (Exception e) {
            e.printStackTrace();
        }

    }

    private static class BatchExportMapper extends TableMapper<Text, Text> {
        @Override
        protected void map(ImmutableBytesWritable rowKey, Result result, Context context) throws IOException, InterruptedException {
            byte[] name_bytes = result.getValue("c1".getBytes(), "name".getBytes());
            byte[] age_bytes = result.getValue("c1".getBytes(), "age".getBytes());

            String v2 = new String(name_bytes) + "\t" + new String(age_bytes);
            context.write(new Text(rowKey.get()), new Text(v2));
        }
    }
}
```

###### 2）HBase Export 工具类

使用方便，不需要写代码，但格式固定不可变

```shell
[root@bigdata01 bin]# pwd
/data/soft/hbase-2.2.7/bin
[root@bigdata01 bin]# hbase org.apache.hadoop.hbase.mapreduce.Export batch2 hdfs://bigdata01:9000/batch2
```

### 1.8. HBase 架构原理？

![1662298321613](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1662298321613.png)

#### 1）Zookeeper

1. Zookeeper，为 HBase 集群提供协调服务，管理着 HMaster 和 HRegionServer 的状体，在节点宕机时会做出相应的通知。
2. HReginServer 启动时，会在 Zookeeper 的 `/hbase/rs` 下创建临时节点。
3. HMaster 启动时，会在 Zookeeper 的 `/hbase/master` 下创建临时节点，其他 Master 会监听该节点，在发现节点失效时，可以接管这个角色。

#### 2）HMaster

1. HMaster，是 HBase 集群的主节点，同时，HBase 还支持多个 HMaster 节点，从而实现 HA。
2. HMaster 负责管理 HRegionServer，实现负载均衡。
3. HMaster 负责管理和分配 Region，比如在 Region 分裂时，负责分配新的 Region、在 HRegionServer 退出时，负责迁移里面的 Region 到其他 HReginServer 上。
4. HMaster 还负责管理 namespace 和 table 的元数据（这些元数据实际存储在 HDFS 上）。
5. HMaster 还负责 HBase 的权限控制，即 ACL 。

#### 3）HRegionServer

1. HRegionServer，是 HBase 集群的从节点。
2. HRegionServer 负责管理数据，一般建议和 HDFS#datanode 部署在同一台机器上，这样可以充分利用数据本地化的特性，读取数据时只有磁盘 I/O，节省了网络 I/O。
3. HRegionServer，包含 1 个 Hlog 和多个 Region（至少保证有 1 个）。

![1662299480030](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1662299480030.png)

##### ① Hlog

1. Hlog，负责记录日志，针对 HRegion 所有的写操作，包括 `put、delete` 等操作，凡是对数据产生变化的操作，都会先记录到这个日志中，然后再把数据写入 HRegion。
2. HLog，预写式日志，一旦服务器崩溃，可以通过重放 HLog，恢复崩溃之前的数据，以保证数据安全性。

##### ② HRegion

![1662299274560](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1662299274560.png)

1. HRegion，简写 Region，负责数据实际数据的存储，在 HBase 中，一个表的数据，会按照行被横向划分为多个 Region，每个 Region，通过存储的 rowkey 的最小行键和最大行键来指定的，使用区间 `[startRwokey, endRowKey]` 表示。

2. 查找数据：前提是必须指定 rowKey，且 rowKey 在 HBase 中是有序存储的，这样可以根据 rowKey 与区域的 `[startRowKey，endRowKey]` 比较，获取到对应的 Region。

3. 写入数据：HRegion，由多个 Store 组成，每一个 Store 对应一个列族，写入数据时，先写 HLog，再把数据写入 MemStore（基于内存），当 MemStore 写满时，才将数据持久化至一个 StoreFile 文件中。

   - 1）StoreFile 底层对应是一个 Hfile 文件，这个 Hfile 文件最终会通过 dfs client 写入到 HDFS 中，每个 Region 中的每一个列族，在底层都是一个单独的文件。

     ![1662290703459](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1662290703459.png)

   - 2）因此，在设计时，可以考虑将经常读取的列，放置在同一个列族中，以提高读取效率。

4. HRegion 分裂机制：

   - 1）HBase 对每个新创建的 Table，默认只会分配一个 Region，此时所有的读写请求，都会访问到同一个RegionServer 的同一个 Region 中，从而出现读写热点的问题。

   - 2）为达到负载均衡，当 Region 达到一定大小时，就会将一个 Region 分裂成两个新的子 Region，并对父 Region 进行清除处理。
   - 3）HMaster 会根据 Balance 策略，重新分配 Region 所属的 RegionServer，最大化发挥分布式系统的优点。

5. HRegion 分裂参数：

   - 1）ConstantSizeRegionSplitPolicy（0.94版本前）：一个Region 中超过 `hbase.hregion.max.filesize` 阈值之后才会触发切分，默认为 10G。由于该值是个固定值，无法兼容大小表的情况，设置得过大，小表可能就只存在一个 Region，设置得过小，大表则会分列出太多 Region，不方便管理。
   - 2）IncreasingToUpperBoundRegionSpliyPolicy（0.94 ~ 2.x 版本默认切分策略）：阈值不再是固定值，而是会不断地自动调整，公式为 `阈值 = min {当前 Region 个数的3次方 * flushsize * 2, hbase.hregion.max.filesize}`。

6. HRegion Balance 策略：分裂后的 Region 分配由 HMaster 负责，HMaster 进程会自动根据指定策略，挑选出一些 Region，并将这些 Region 分配到负载比较低的 RegionServer 上。目前支持两种 Region 分配策略：

   - 1）DefaultLoadBanancer：默认负载均衡器，直接看 Region 个数，可以保证每个 RegionServer 中的 Region 个数基本相等，但无法针对 Region 中的实际内容进行改变，比如：虽然两个 RegionServer 都是 10 个 Region，但其中一个 RegionServer 中 Region 内容比较多，另一个则只有几条，这样会导致后期读写请求还是会集中落在同一个 RegionServer 上。

   - 2）StochasticLoadBalancer：随机负载均衡器，是一种综合权衡 6 个因素的均衡策略，计算出的代价值越低，说明 Region 越均衡。

     ![1662303239650](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1662303239650.png)

###### HFile

1. HFile，是 HBase 架构中最小的结构，HBase 的底层数据都在 HFile 中，本质上，HFile 就是一个有着自己特殊格式的 HDFS 文件。

2. HFile 文件组成：

   ![1662301088025](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1662301088025.png)

   - 1）Data：数据块，保存表中的数据（ KEY-VALUE 形式）。
   - 2）Meta：可选，元数据块，存储用户自定义的一些 KEY-VALUE。
   - 3）File Info：定长，记录文件的元数据信息。
   - 4）Data Index：数据索引，记录每个数据块的起始索引。
   - 5）Meta Index：元数据索引，记录每个元数据块的起始索引。
   - 6）Trailer：定长，用于指向其他数据块的一个起始点。

3. HFile 合并机制：当 MemStore 写满时，就会持久化生成一个 StoreFile 文件，而 StoreFile 底层则对应一个 HFile 文件， 当 HFile 数量过多时，会降低读性能，此时则需要一个合并机制，将多个 HFile 合并成一个 HFile。

   - 1）minor：小合并，只做部分文件的合并操作，合并过程一般较快，I/O 相对较低。
   - 2）major：大合并，将 Region 下所有的 HFile 合并成一个文件，此时会真正忽略掉已经标记为删除的、ttl 过期的、版本超过限制的数据，期间会产生大量的 I/O 操作，对 HBase 读写性能产生较大影响，整个过程持续时间长且消耗大量系统资源，对相关业务会有较大影响，因此线上一般会关闭自动触发机制，改为在业务低峰期时手动触发。

#### 4）HBase Client

HBase Client 与 HBase 的通信流程：

1. Client 连接 Zookeeper，`get /hbase/meta-region-server` 查找节点信息，该节点保存了 Hbase#meta 表，meta 表存储了 HBase 中所有表的 ReginServer 节点信息。
2. 然后，Client 把此次查询 meta 表的信息，加载至本地缓存中。
3. 这样，Client 在需要时，则会从本地缓存中读入到内存中，从 meta 表获取 RegionServer 的地址以及端口信息，再通过 RPC 机制获取对应的表数据。

### 1.9. HBase 调优策略？

#### 1）预分区

- 1）新建表时默认只有一个 Region，且没有 startRowKey 和 endRowKey，数据都默认写入这一个 Region，存在写热点问题。
  - 发现热点问题：查看 Region Request 计数是否相对过高，不过 HBase 重启后该指标会清 0。
- 2）当数据量增大，又会导致 Region 分裂，虽然能解决写热点问题，但又会带来大量的 I/O，影响系统性能。
- 3）因此，可以在建表时，预先创建多个空的 Region，并确定每个 Region 的起始和终止 rowkey，只要 Region 足够均匀，那么就可以解决写热点问题，同时，后期 Region 分裂的频率也会降低。 

```shell
# 创建5个Region：0、1、2、3、4开头的rowkey
hbase(main):001:0> create 't20', 'c1', SPLITS => ['10', '20', '30', '40']
```

![1662379997983](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1662379997983.png)

#### 2）RowKey 设计原则

1. 长度原则：RowKey 越短越好，一般不要超过 16 个字节，这是因为 HFile 的底层以 KEY-VALUE 形式存储，RowKey 过长会占用更多的存储空间，其次，Rowkey 过长也会占用更多的 Memstore 空间，导致缓存数据减少，降低检索效率。
2. 散列原则：散列均匀，可以避免数据热点问题。
3. 唯一性原则：Rowkey 相当于 MySQL 的唯一索引，设计时必须要保证其唯一性，因为相同的 RowKey 会覆盖。

#### 3）列族的设计原则

把经常读取的字段，存储至一个列族中，不经常读取的字段放到另一个列族中，这样读取数据时，只需要读取一个列族文件即可，从而提高读取效率。

#### 4）调优相关参数

| 参数                                    | 默认值 | 作用                                                         |
| --------------------------------------- | ------ | ------------------------------------------------------------ |
| hbase.hregion.majorcompaction           | 7 天   | 大合并的间隔时间，可以设置为 0，表示禁止自动的大合并，在低峰期手动合并，减少对业务的影响 |
| hbase.hregion.max.filesize              | 10 GB  | Region 分裂的阈值，调大该值，并在低峰期手动分裂，减少对业务的影响 |
| hbase.hregionserver.handler.count       | 30     | 负责底层数据的发送，对于大范围的 Scan 或者 Put，配置过大容易导致 OOM，对于小 get / put /delete，适当调大可以提高效率 |
| hbase.hregion.memstore.flush.size       | 128 mb | 控制把 MemStore 数据持久化到 StoreFile 的阈值，适当调大该值，可以减少 MemStore 写文件的次数 |
| hbase.hregion.memstore.block.multiplier | 4      | 当 MemStore 大小超过 `hbase.hregion.memstore.flush.size *  hbase.hregion.memstore.block.multiplier` 时， HBase 会阻塞该 MemStore 的写操作，适当调大，可以避免写阻塞的发生，但过大会有 OOM 的风险 |
| hbase.hstore.compaction.min             | 3      | StoreFile 总数超过该阈值时，会触发 HFile 的合并机制，可以设置为 5~8，并定期手工合并，减少合并的次数 |

