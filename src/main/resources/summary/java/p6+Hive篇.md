# **十八、Hive 篇**

### 1.1. 数据库 vs 数据仓库？

1. 数据库：传统的关系型数据库，主要应用在基本的事务处理，例如银行交易，支持增删改查。
2. 数据仓库：概念要比数据库要大，可以理解为包含的关系，主要做一些复杂的分析操作，侧重决策支持，数据规模相对大得多，只支持查询，不支持修改和删除。

因此，数据库和数据仓库的区别，本质上就是 OLTP 和 OLTP 的区别。

|          | OLTP                                   | OLAP                                 |
| -------- | -------------------------------------- | ------------------------------------ |
| 定位     | 联机事务处理，用于日常的事务处理       | 联机分析处理，用于复杂分析、决策支持 |
| 强调指标 | 内存命中率、SQL 变量绑定、SQL 并发执行 | SQL 执行时长、磁盘 I/O、带宽吞吐量   |
| 数据     | 当前的、最新的、细节的                 | 历史的、聚集的、统一的               |
| 读写方式 | 每次读写数十条记录                     | 每次读上百万条记录                   |
| 工作单位 | 简单的事务                             | 复杂的查询                           |
| DB大小   | 100 MB-GB                              | 100 GB-TB                            |

### 1.2. 什么是 Hive？

1. Hive，是建立在 Hadoop 上的数据仓库，提供了一系列的工具进行 ETL（数据提取、转化、加载），定义了简单的类 SQL 查询语言 HQL，直接查询 Hadoop 中的数据。
2. Hive 底层其实是，将 SQL 转译成 MapReduce 任务，然后再在 Hadoop 上执行该任务。
3. Hive 的数据存储基于 Hadoop#HDFS，没有专门的数据存储格式，默认使用 TextFile，还支持 SequenceFile、RCFile 等。

|                  | Hive                                            | MySQL        |
| ---------------- | ----------------------------------------------- | ------------ |
| **数据存储位置** | 元数据存储在 Derby / MySQL，业务数据存储在 HDFS | 本地磁盘     |
| **数据格式**     | 用户定义                                        | 系统决定     |
| **数据更新**     | 不支持                                          | 支持         |
| **索引**         | 有，但较弱                                      | 有，经常使用 |
| **执行器**       | MapReduce                                       | Executor     |
| **延迟**         | 高                                              | 低           |
| **可扩展性**     | 高                                              | 低           |
| **数据规模**     | 大                                              | 小           |

### 1.3. Hive 系统架构？

1. 用户接口：CLI 支持 shell 操作、JDBC/ODBC 支持程序连接、WEBUI 支持浏览器操作。
2. 元数据存储 ：
   - 1）Metastore，是 Hive 元数据的集中存放地，目前只支持数据库 derby、mysql。
   - 2）元数据，包括表名字、表列、表分区及其属性、表数据所在目录等。
   - 3）Metastore 默认使用内嵌的 Derby 数据库，作为存储引擎，缺点是在同一个目录下，只能打开一个会话，即同一时间只能一个人操作，且切换新目录操作时，之前的元数据会丢失。
   - 4）因此，推荐使用 Mysql 数据库，作为 Metastore 外置的存储引擎。
3. Driver：编译器、优化器、执行器，用于完成 Hive 词法分析 => 语法分析 => 编译 => 优化 => 生成查询计划 => 把查询计划存储到 HDFS 里 => 被 MapReduce 执行。
4. Hadoop：HDFS 存储、MapReduce 计算，Hive 默认使用第一代大数据计算引擎 MapReduce。
   - 特列：`select * from table`，由于可以直接查询 HDFS 中的文件数据，所以不会产生 MapReduce 任务，如果有 where 条件时，则会产生 MapReduce 任务。

![1661590101943](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1661590101943.png)

### 1.4. 大数据计算引擎？

1. 第一代：Map Reduce，基于磁盘实现。
2. 第二代：Tez，源于 MapReduce，核心是将 MapReduce 拆分，再灵活组合。
3. 第三代：Spark，基于内存计算，中间结果不写入磁盘，效率比 MapReduce 高 100 倍。
4. 第四代：Flink，基于实时计算。

### 1.5. Hive 安装部署？

#### 1）上传、解压

```shell
tar -zxvf apache-hive-3.1.2-bin.tar.gz
```

#### 2）修改配置文件名称

```shell
[root@bigdata04 conf]# pwd
/data/soft/apache-hive-3.1.2-bin/conf
[root@bigdata04 conf]# ll
总用量 332
-rw-r--r--. 1 root root   1596 8月  23 2019 beeline-log4j2.properties.template
-rw-r--r--. 1 root root 300482 8月  23 2019 hive-default.xml.template
-rw-r--r--. 1 root root   2365 8月  23 2019 hive-env.sh.template
-rw-r--r--. 1 root root   2274 8月  23 2019 hive-exec-log4j2.properties.template
-rw-r--r--. 1 root root   3086 8月  23 2019 hive-log4j2.properties.template
-rw-r--r--. 1 root root   2060 8月  23 2019 ivysettings.xml
-rw-r--r--. 1 root root   3558 8月  23 2019 llap-cli-log4j2.properties.template
-rw-r--r--. 1 root root   7163 8月  23 2019 llap-daemon-log4j2.properties.template
-rw-r--r--. 1 root root   2662 8月  23 2019 parquet-logging.properties

[root@bigdata04 conf]# mv hive-env.sh.template hive-env.sh
[root@bigdata04 conf]# mv hive-default.xml.template hive-site.xml
```

#### 3）修改 hive-env.sh

```shell
# 在Hive的hive-env.sh配置文件末尾，增加下面几行
export JAVA_HOME=/data/soft/jdk1.8
export HIVE_HOME=/data/soft/apache-hive-3.1.2-bin
export HADOOP_HOME=/data/soft/hadoop-3.2.0
```

#### 4）修改 hive-site.xml

这里连的是 MySQL 5.6，驱动包用的是 5.1.6 版本，所以不是 `com.mysql.cj.jdbc.Driver` 。

```xml
<property>
    <name>javax.jdo.option.ConnectionURL</name>
    <value>jdbc:mysql://192.168.56.1:3306/hive?serverTimezone=Asia/Shanghai</value>
</property>
<property>
    <name>javax.jdo.option.ConnectionDriverName</name>
    <value>com.mysql.jdbc.Driver</value>
</property>
<property>
    <name>javax.jdo.option.ConnectionUserName</name>
    <value>root</value>
</property>
<property>
    <name>javax.jdo.option.ConnectionPassword</name>
    <value>admin</value>
</property>
<property>
    <name>hive.querylog.location</name>
    <value>/data/hive_repo/querylog</value>
</property>
<property>
    <name>hive.exec.local.scratchdir</name>
    <value>/data/hive_repo/scratchdir</value>
</property>
<property>
    <name>hive.downloaded.resources.dir</name>
    <value>/data/hive_repo/resources</value>
</property>

<!-- hive-3.1.2版本初始化元数据时，发现hive-site.xml#3215行#描述信息报错，这里直接删掉即可 -->
```

#### 5）修改日志配置

```shell
[root@bigdata04 conf]# pwd
/data/soft/apache-hive-3.1.2-bin/conf
[root@bigdata04 conf]# mv hive-log4j2.properties.template hive-log4j2.properties
[root@bigdata04 conf]# vim hive-log4j2.properties 
...
# list of properties
property.hive.log.level = WARN
property.hive.log.dir = /data/hive_repo/log
...

[root@bigdata04 conf]# mv hive-exec-log4j2.properties.template hive-exec-log4j2.properties
[root@bigdata04 conf]# vim hive-exec-log4j2.properties
...
# list of properties
property.hive.log.level = WARN
property.hive.log.dir = /data/hive_repo/log
...
```

#### 6）修改 Hadoop 集群配置

```shell
[root@bigdata01 hadoop]# pwd
/data/soft/hadoop-3.2.0/etc/hadoop
[root@bigdata01 hadoop]# vim core-site.xml
    <property>
        <name>hadoop.proxyuser.root.hosts</name>
        <value>*</value>
    </property>
    <property>
        <name>hadoop.proxyuser.root.groups</name>
        <value>*</value>
    </property>

# 修改后，同步到集群中的其他节点
[root@bigdata01 hadoop]# scp -rq core-site.xml bigdata02:/data/soft/hadoop-3.2.0/etc/hadoop/
[root@bigdata01 hadoop]# scp -rq core-site.xml bigdata03:/data/soft/hadoop-3.2.0/etc/hadoop/
[root@bigdata01 hadoop]# 

# 重启Hadoop集群
[root@bigdata01 hadoop]# stop-all.sh
...
[root@bigdata01 hadoop]# start-all.sh 
...
```

#### 7）创建 MySQL#hive 数据库

```shell
create database hive;
```

#### 8）初始化 Metastore 元数据

在 mysql#hive 数据库中，显示对应的元数据表，即代表 hive 安装成功~

```shell
[root@bigdata04 bin]# pwd
/data/soft/apache-hive-3.1.2-bin/bin
[root@bigdata04 bin]# schematool -dbType mysql -initSchema
...
Initialization script completed
schemaTool completed
```

#### 9）操作使用 Hive

##### 1、hive

旧版方式

```shell
[root@bigdata04 bin]# pwd
/data/soft/apache-hive-3.1.2-bin/bin

# 进入hive命令行
[root@bigdata04 bin]# hive
...
Hive Session ID = a2c8850b-e9de-4a78-88b6-90bb2433f473

# 查询所有表
hive> 
hive> show tables;
OK
Time taken: 0.146 seconds

# 创建t1表
hive> create table t1(id int, name string);
OK
Time taken: 0.491 seconds
hive> show tables;
OK
t1
Time taken: 0.045 seconds, Fetched: 1 row(s)

# 插入一条数据到t1表，底层实际上是一个MapReduce任务
hive> insert into t1(id, name) values(1, "zs");
Query ID = root_20220827190427_406e4c0b-2ddc-44c3-97f3-4a18ad6ad14c
...

# 查询t1表
hive> select * from t1;
OK
1       zs
Time taken: 0.193 seconds, Fetched: 1 row(s)

# 删除t1表
hive> drop table t1;
OK
Time taken: 0.188 seconds
hive> show tables;
OK
Time taken: 0.031 seconds
hive> quit;
```

##### 2、beeline

新版方式

```shell
# 先启动hive2服务端：
[root@bigdata04 bin]# pwd
/data/soft/apache-hive-3.1.2-bin/bin
[root@bigdata04 bin]# hiveserver2 
...
Hive Session ID = 8f9d3cc8-c6ed-471e-afcf-7291a94f7c59
Hive Session ID = e7d9ec87-69c8-465e-bb48-50ae99ba21e8
Hive Session ID = c32ac3b6-1c28-430c-8abf-6da568e65f88
Hive Session ID = 7c86db5c-6ac1-49fd-94f2-8266e8aa997a

# 再开启另一个窗口，以root身份登入命令行：
[root@bigdata04 bin]# beeline -u jdbc:hive2://localhost:10000 -n root
...
Connecting to jdbc:hive2://localhost:10000
Connected to: Apache Hive (version 3.1.2)
Driver: Hive JDBC (version 3.1.2)
Transaction isolation: TRANSACTION_REPEATABLE_READ
Beeline version 3.1.2 by Apache Hive

# 查询所有表
0: jdbc:hive2://localhost:10000> show tables;
...
+-----------+
| tab_name  |
+-----------+
+-----------+
No rows selected (0.116 seconds)

# 创建t1表
0: jdbc:hive2://localhost:10000> create table t1(id int, name string);
...
No rows affected (0.092 seconds)

# 插入一条数据到t1表，底层实际上是一个MapReduce任务
0: jdbc:hive2://localhost:10000> insert into t1(id, name) values(1, "zs");
...
No rows affected (21.758 seconds)

# 查询t1表
0: jdbc:hive2://localhost:10000> select * from t1;
...
+--------+----------+
| t1.id  | t1.name  |
+--------+----------+
| 1      | zs       |
+--------+----------+
1 row selected (0.237 seconds)
```

##### 3、直接运行脚本

hive 会先进入命令行，然后执行完脚本，输出结果后再自动退出。

- 另外，使用 beeline 也一样。

```shell
[root@bigdata04 bin]# pwd
/data/soft/apache-hive-3.1.2-bin/bin
[root@bigdata04 bin]# hive -e "select * from t1"
...
Hive Session ID = a57dd4a5-f44b-4e53-9da1-30c9a485c327
OK
1       zs
Time taken: 2.32 seconds, Fetched: 1 row(s)
[root@bigdata04 bin]# 
```

##### 4、JDBC 方式连接

需要先启动 hive2 服务端，本质上相当于 beeline 方式远程连接。

```xml
<dependency>
    <groupId>org.apache.hive</groupId>
    <artifactId>hive-jdbc</artifactId>
    <version>3.1.2</version>
</dependency>
```

```java
/**
 * 测试JDBC连接Hive
 *
 * @author yaocs2
 * @since 2022-08-27
 */
public class HiveJdbcDemo {

    public static void main(String[] args) throws SQLException {
        // hive无需密码登录
        String jdbcUrl = "jdbc:hive2://192.168.56.111:10000";
        Connection conn = DriverManager.getConnection(jdbcUrl, "root", "*");

        // 获取Statement
        Statement stmt = conn.createStatement();

        // 执行sql
        String sql = "select * from t1";
        ResultSet res = stmt.executeQuery(sql);

        // 迭代结果
        while (res.next()) {
            // 1	zs
            System.out.println(res.getInt("id") + "\t" + res.getString("name"));
        }
    }
}
```

#### 10）hive 变量设置

set 命令设置，当前会话有效；`~/.hiverc` 配置，当前机器当前用户有效；`conf/hive-site.xml` 配置，永久有效。

```shell
# 显示当前db名称
hive> set hive.cli.print.current.db=true;
hive (default)> 

# 显示查询列头
hive (default)> set hive.cli.print.header=true;
hive (default)> select * from t1;
OK
t1.id   t1.name
1       zs
Time taken: 2.033 seconds, Fetched: 1 row(s)
```

#### 11）查看 hive 历史命令

```shell
[root@bigdata04 conf]# tail -3 ~/.hivehistory 
set hive.cli.print.current.db=true;
set hive.cli.print.header=true;
select * from t1;
```

### 1.6. Hive 使用与操作？

#### 1）数据库操作

##### 1、查看所有数据库

- 数据存储位置：HDFS#/user/hive/warehouse 文件夹里。

- 元数据位置：MySQL#hive#dbs 表中。

```shell
hive (default)> show databases;
OK
database_name
default
Time taken: 0.395 seconds, Fetched: 1 row(s)
```

##### 2、选择数据库

```shell
hive (default)> use default;
OK
Time taken: 0.04 seconds
```

##### 3、创建数据库

```shell
# 默认存储在HDFS#/user/hive/warehouse/mydb1.db/文件夹里
hive (default)> create database mydb1;
OK
Time taken: 0.13 seconds

# 指定存储在HDFS#/user/hive/mydb2/文件夹里
hive (default)> create database mydb2 location '/user/hive/mydb2';
OK
Time taken: 0.034 seconds
```

##### 4、删除数据库

```shell
hive (default)> drop database mydb1;
OK
Time taken: 0.27 seconds

# 但无法删除default数据库
hive (default)> drop database default;
FAILED: Execution Error, return code 1 from org.apache.hadoop.hive.ql.exec.DDLTask. MetaException(message:Can not drop default database in catalog hive)
```

#### 2）表操作

##### 1、创建表

```shell
hive (default)> create table t2(id int);
OK
Time taken: 1.142 seconds
```

##### 2、查看所有表

```shell
hive (default)> show tables;
OK
tab_name
t1
t2
Time taken: 0.3 seconds, Fetched: 2 row(s)
```

##### 3、查看表结构

```shell
hive (default)> desc t2;
OK
col_name        data_type       comment
id                      int                                         
Time taken: 0.079 seconds, Fetched: 1 row(s)
```

##### 4、查看建表语句

```shell
hive (default)> show create table t2;
OK
createtab_stmt
CREATE TABLE `t2`(
  `id` int)
ROW FORMAT SERDE 
  'org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe' 
STORED AS INPUTFORMAT 
  'org.apache.hadoop.mapred.TextInputFormat' 
OUTPUTFORMAT 
  'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION
  'hdfs://bigdata01:9000/user/hive/warehouse/t2'
TBLPROPERTIES (
  'bucketing_version'='2', 
  'transient_lastDdlTime'='1661603278')
Time taken: 0.113 seconds, Fetched: 13 row(s)
```

##### 5、修改表名

```shell
hive (default)> alter table t2 rename to t2_bak;
OK
Time taken: 0.156 seconds
```

##### 6、插入数据 | 仅用于测试

```shell
# 插入一条数据到t2_bak表，底层实际上是一个MapReduce任务
hive (default)> insert into t2_bak(id) values(0);
Query ID = root_20220827203415_687e6e5d-00b0-4c42-880f-51221c39e345
...

# 查询t2_bak表
hive (default)> select * from t2_bak;
OK
t2_bak.id
0
Time taken: 0.143 seconds, Fetched: 1 row(s)
```

##### 7、加载数据

```shell
# hive方式加载
hive (default)> load data local inpath '/data/hivedata/t2.data' into table t2_bak;
Loading data to table default.t2_bak
OK
Time taken: 1.325 seconds

# 等价于，直接添加到HDFS对应目录
[root@bigdata04 hivedata]# hdfs dfs -put t2.data /user/hive/warehouse/t2_bak/t2.data

# 查询t2_bak表
hive (default)> select * from t2_bak;
OK
t2_bak.id
0
1
2
3
4
5
Time taken: 1.147 seconds, Fetched: 6 row(s)
```

##### 8、增加表字段

```shell
# 增加name列字段
hive (default)> alter table t2_bak add columns(name string);
OK
Time taken: 0.21 seconds
hive (default)> desc;

# 查询表结构
hive (default)> desc t2_bak;
OK
col_name        data_type       comment
id                      int                                         
name                    string                                      
Time taken: 0.051 seconds, Fetched: 2 row(s)

# 查询旧数据
hive (default)> select * from t2_bak;
OK
t2_bak.id       t2_bak.name
0       NULL
1       NULL
2       NULL
3       NULL
4       NULL
5       NULL
Time taken: 0.146 seconds, Fetched: 6 row(s)
```

##### 9、解决字段注释、表注释乱码

```shell
# 建表t2
hive (default)> create table t2(
              >   age int comment '年龄'
              > ) comment '测试';
OK

# 查看t2建表语句，发现乱码
hive (default)> show create table t2;
OK
createtab_stmt
CREATE TABLE `t2`(
  `age` int COMMENT '??')
COMMENT '??'
ROW FORMAT SERDE 
  'org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe' 
STORED AS INPUTFORMAT 
  'org.apache.hadoop.mapred.TextInputFormat' 
OUTPUTFORMAT 
  'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION
  'hdfs://bigdata01:9000/user/hive/warehouse/t2'
TBLPROPERTIES (
  'bucketing_version'='2', 
  'transient_lastDdlTime'='1661604478')
Time taken: 0.087 seconds, Fetched: 14 row(s)

# 修改mysql#hive中的元数据字符集
alter table COLUMNS_V2 modify column COMMENT varchar(256) character set utf8;
alter table TABLE_PARAMS modify column PARAM_VALUE varchar(4000) character set utf8;
# 如果还创建了分区的话，则还要再执行以下两条命令
alter table PARTITION_PARAMS modify column PARAM_VALUE varchar(4000) character set utf8;
alter table PARTITION_KEYS modify column PKEY_COMMENT varchar(4000) character set utf8;

# 重建t2表
hive (default)> drop table t2;
OK
Time taken: 0.335 seconds 
hive (default)> create table t2(
              >   age int comment '年龄'
              > ) comment '测试';
OK
Time taken: 0.552 seconds

# 重新查看建表语句，发现乱码解决
hive (default)> show create table t2;
OK
createtab_stmt
CREATE TABLE `t2`(
  `age` int COMMENT '年龄')
COMMENT '测试'
ROW FORMAT SERDE 
  'org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe' 
STORED AS INPUTFORMAT 
  'org.apache.hadoop.mapred.TextInputFormat' 
OUTPUTFORMAT 
  'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION
  'hdfs://bigdata01:9000/user/hive/warehouse/t2'
TBLPROPERTIES (
  'bucketing_version'='2', 
  'transient_lastDdlTime'='1661604932')
Time taken: 0.146 seconds, Fetched: 14 row(s)

# 查看表结构，发现乱码也解决了
hive (default)> desc t2;
OK
col_name        data_type       comment
age                     int                     年龄                  
Time taken: 0.062 seconds, Fetched: 1 row(s)
```

##### 10、删除表

```shell
hive (default)> drop table t2;
OK
Time taken: 0.335 seconds
```

##### 11、指定行、列分隔符

默认行分隔符为 `\n`，默认列分割符为 `\001` （键盘输入 `ctrl + v` + `ctrl + a`）

```shell
# 查看要加载的数据
[root@bigdata04 hivedata]# more t3.data 
1       张三    2020-01-01      true
2       李四    2020-02-01      false
3       王五    2020-03-01      0

# 指定行、列分隔符
hive (default)> create table t3(
              >   id int comment 'ID',
              >   stu_name string comment 'name',
              >   stu_birthday date comment 'birthday',
              >   online boolean comment 'is online'
              > ) row format delimited 
              > fields terminated by '\t' 
              > lines terminated by '\n';
OK
Time taken: 0.173 seconds

# 加载数据
hive (default)> load data local inpath '/data/hivedata/t3.data' into table t3;
Loading data to table default.t3
OK
Time taken: 0.267 seconds

# 查看t3表：最后一行最后一列，由于原数据是0，而字段为boolean类型，hive在导入时不做校验，在查询时校验到不匹配，则返回null
hive (default)> select * from t3;
OK
t3.id   t3.stu_name     t3.stu_birthday t3.online
1       张三    2020-01-01      true
2       李四    2020-02-01      false
3       王五    2020-03-01      NULL
Time taken: 0.267 seconds, Fetched: 3 row(s)
```

#### 3）数据类型

| 分类         | 类型          | 格式                               |
| ------------ | ------------- | ---------------------------------- |
| 基本数据类型 | TINYINT       |                                    |
|              | SMALLINT      |                                    |
|              | INT / INTEGER |                                    |
|              | BIGINT        |                                    |
|              | FLOAT         |                                    |
|              | DOUBLE        |                                    |
|              | DECIMAL       |                                    |
|              | TIMESTAMP     |                                    |
|              | DATE          |                                    |
|              | STRING        |                                    |
|              | VARCHAR       |                                    |
|              | CHAR          |                                    |
|              | BOOLEAN       |                                    |
| 复合数据类型 | ARRAY         | ARRAY `<data_tpye>`                |
|              | MAP           | MAP `<primitive_type, data_type>`  |
|              | STRUCT        | STRUCT `<col_name:data_type, ...>` |

##### 1、array 类型

```sh
# 查看原始数据
[root@bigdata04 hivedata]# more stu.data 
1       zhangsan        swing,sing,coding
2       lisi    music,football

# 创建表（含array字段）
hive (default)> create table stu(
              >   id int,
              >   name string,
              >   favors array<string>
              > ) row format delimited 
              > fields terminated by '\t'
              > collection items terminated by ','
              > lines terminated by '\n';
OK
Time taken: 1.706 seconds

# 加载数据
hive (default)> load data local inpath '/data/hivedata/stu.data' into table stu;
Loading data to table default.stu
OK
Time taken: 1.962 seconds

# 查询结果
hive (default)> select * from stu;
OK
stu.id  stu.name        stu.favors
1       zhangsan        ["swing","sing","coding"]
2       lisi    ["music","football"]
Time taken: 3.393 seconds, Fetched: 2 row(s)
hive (default)> select id, name, favors[0] from stu;
OK
id      name    _c2
1       zhangsan        swing
2       lisi    music
Time taken: 1.266 seconds, Fetched: 2 row(s)
hive (default)> select id, name, favors[3] from stu;
OK
id      name    _c2
1       zhangsan        NULL
2       lisi    NULL
```

##### 2、map 类型

```shell
# 查看原始数据
[root@bigdata04 hivedata]# more stu2.data 
1       zhangsan        chinese:80,math:90,english:100
2       lisi    chinese:89,english:70,math:88

# 创建表（含map字段）
hive (default)> create table stu2(
              >   id int,
              >   name string,
              >   soores map<string, int>
              > ) row format delimited 
              > fields terminated by '\t'
              > collection items terminated by ','
              > map keys terminated by ':'
              > lines terminated by '\n';
OK
Time taken: 0.915 seconds

# 加载数据
hive (default)> load data local inpath '/data/hivedata/stu2.data' into table stu2;
Loading data to table default.stu2
OK
Time taken: 0.581 seconds

# 查询结果
hive (default)> select * from stu2;
OK
stu2.id stu2.name       stu2.soores
1       zhangsan        {"chinese":80,"math":90,"english":100}
2       lisi    {"chinese":89,"english":70,"math":88}
Time taken: 1.608 seconds, Fetched: 2 row(s)
hive (default)> select id, name, soores['chinese'], soores['math'] from stu2;
OK
id      name    _c2     _c3
1       zhangsan        80      90
2       lisi    89      88
Time taken: 0.382 seconds, Fetched: 2 row(s)
```

##### 3、struct 类型

```shell
# 查看原始数据
[root@bigdata04 hivedata]# more stu3.data 
1       zhangsan        bj,sh
2       lisi    gz,sz

# 创建表（含struct字段）
hive (default)> create table stu3(
              >   id int,
              >   name string,
              >   address struct<home_addr:string, office_addr:string>
              > ) row format delimited 
              > fields terminated by '\t'
              > collection items terminated by ','
              > map keys terminated by ':'
              > lines terminated by '\n';
OK
Time taken: 1.087 seconds

# 加载数据："，"分隔
hive (default)> load data local inpath '/data/hivedata/stu3.data' into table stu3;
Loading data to table default.stu3
OK
Time taken: 0.51 seconds

# 查询结果
hive (default)> select * from stu3;
OK
stu3.id stu3.name       stu3.address
1       zhangsan        {"home_addr":"bj","office_addr":"sh"}
2       lisi    {"home_addr":"gz","office_addr":"sz"}
Time taken: 1.163 seconds, Fetched: 2 row(s)
hive (default)> select id, name, address.home_addr from stu3;
OK
id      name    home_addr
1       zhangsan        bj
2       lisi    gz
Time taken: 0.262 seconds, Fetched: 2 row(s)

# 查看原始数据
[root@bigdata04 hivedata]# more student_more_scores.data 
zs      语文:81,数学:80
ls      语文:90

# 上传到HDFS
[root@bigdata04 hivedata]# hdfs dfs -put student_more_scores.data /data/student_more_scores/student_more_scores.data

# 创建外部表：先用"，"分隔元素，每个元素再按照':'划分键值对
# 注意，亲测！这里是有array时，用map keys分隔才有效，其他时候strcut用map无效，而是用collection items分隔
hive (default)> create external table student_more_scores(
              >   name string,
              >   score array<struct<subject:string,score:int>>
              > )row format delimited
              > fields terminated by '\t'
              > collection items terminated by ','
              > map keys terminated by ':'
              > location '/data/student_more_scores';
OK
Time taken: 0.057 seconds

# 查询结果：
hive (default)> select * from student_more_scores;
OK
student_more_scores.name        student_more_scores.score
zs      [{"subject":"语文","score":81},{"subject":"数学","score":80}]
ls      [{"subject":"语文","score":90}]
Time taken: 0.142 seconds, Fetched: 2 row(s)
```

#### 4）表类型

##### 1、内部表

1. 内部表，是 Hive 中默认的表类型，表数据默认存储在 `warehouse` 目录下。
2. 在 `load` 加载数据时，实际数据会被移动到 `warehouse` 中。
3. 在删除表时，表中的数据和元数据，都会被删除。

```shell
# 查看原始数据
[root@bigdata04 hivedata]# more student.data 
1       zhangsan        english,sing,swing      chinese:80,math:90,english:100  bj,sh
2       lisi    games,coding    chinese:89,english:70,math:88   gz,sz

# 创建内部表
hive (default)> create table student(
              >   id int,
              >   name string,
              >   favors array<string>,
              >   scores map<string, int>, 
              >   address struct<home_addr:string, office_addr:string>
              > ) row format delimited 
              > fields terminated by '\t'
              > collection items terminated by ','
              > map keys terminated by ':'
              > lines terminated by '\n';
OK
Time taken: 0.819 seconds

# 加载数据
hive (default)> load data local inpath '/data/hivedata/student.data' into table student;
Loading data to table default.student
OK
Time taken: 0.459 seconds

# 查询结果
hive (default)> select * from student;
OK
student.id      student.name    student.favors  student.scores  student.address
1       zhangsan        ["english","sing","swing"]      {"chinese":80,"math":90,"english":100}  {"home_addr":"bj","office_addr":"sh"}
2       lisi    ["games","coding"]      {"chinese":89,"english":70,"math":88}   {"home_addr":"gz","office_addr":"sz"}
Time taken: 1.116 seconds, Fetched: 2 row(s)

# 查看hdfs#warehouse目录
[root@bigdata04 hivedata]# hdfs dfs -ls /user/hive/warehouse
Found 8 items
/user/hive/warehouse/student
...
```

##### 2、外部表

1. 外部表，指的是在建表语句中，包含 `External` 的表。
2. 在加载数据时，实际数据不会移动到 `warehouse` 目录，只是与外部数据建立一个链接而已。
3. 在删除表时，只会删除表的元数据，不会删除实际数据，即仅仅删除表和数据之间的链接。

```shell
# 查看原始数据
[root@bigdata04 hivedata]#  more external_table.data 
a
b
c
d
e

# 创建外部表
hive (default)> create external table external_table(
              >   key string
              > ) location '/data/external';
OK
Time taken: 0.65 seconds

# 加载数据
hive (default)> load data local inpath '/data/hivedata/external_table.data' into table external_table;
Loading data to table default.external_table
OK
Time taken: 0.731 seconds

# 查询结果
hive (default)> select * from external_table;
OK
external_table.key
a
b
c
d
e

# 查看hdfs#/data/external目录
[root@bigdata04 hivedata]# hdfs dfs -ls /data/external
Found 1 items
-rw-r--r--   2 root supergroup         15 2022-08-29 05:27 /data/external/external_table.data

# 测试删除外部表，发现表被删了，但hdfs中的数据还在
hive (default)> drop table external_table;
OK
Time taken: 0.284 seconds
hive (default)> show tables;
OK
tab_name
stu
stu2
stu3
student
t1
t2
t2_bak
t3
[root@bigdata04 hivedata]# hdfs dfs -ls /data/external
Found 1 items
-rw-r--r--   2 root supergroup         15 2022-08-29 05:27 /data/external/external_table.data

# 内部表转外部表
hive (default)> alter table student set tblproperties('external'='true');
OK
Time taken: 0.134 seconds

# 外部表转内部表
hive (default)> alter table student set tblproperties('external'='false');
OK
Time taken: 0.105 seconds
```

##### 3、分区表

1. 分区，可以通过指定一个或者多个字段，作为分区字段，然后把不同分类的数据，放到不同的目录中。
2. 分区表的意义在于优化查询，查询时尽量利用分区字段，如果不使用分区字段，就会进行全表扫描。
3. 最典型的场景是，把天作为分区字段，查询时指定具体的日期。

###### 1）内部分区表

```shell
# 查看原始数据
[root@bigdata04 hivedata]# more partition_1.data 
1       zhangsan
2       lisi

# 创建内部分区表（只有一个分区字段）
hive (default)> create table partition_1(
              >   id int,
              >   name string
              > ) 
              > partitioned by (dt string)
              > row format delimited 
              > fields terminated by '\t'
              > lines terminated by '\n';
OK
Time taken: 0.06 seconds

# 查看分区表结构
hive (default)> desc partition_1;
OK
col_name        data_type       comment
id                      int                                         
name                    string                                      
dt                      string                                                 
# Partition Information          
# col_name              data_type               comment             
dt                      string                                      
Time taken: 0.087 seconds, Fetched: 7 row(s)

# 加载分区1的数据
load data local inpath '/data/hivedata/partition_1.data' into table partition_1 partition(dt='20200101');

# 查看在hdfs中，分区表分区1的目录
[root@bigdata04 hivedata]# hdfs dfs -ls /user/hive/warehouse/partition_1
Found 1 items
drwxr-xr-x   - root supergroup          0 2022-08-29 05:46 /user/hive/warehouse/partition_1/dt=20200101

# 单独添加分区2
hive (default)> alter table partition_1 add partition(dt='20220102');
OK
Time taken: 0.218 seconds

# 查看在hdfs中，分区表所有分区的目录
[root@bigdata04 hivedata]# hdfs dfs -ls /user/hive/warehouse/partition_1
Found 2 items
drwxr-xr-x   - root supergroup          0 2022-08-29 05:46 /user/hive/warehouse/partition_1/dt=20200101
drwxr-xr-x   - root supergroup          0 2022-08-29 05:49 /user/hive/warehouse/partition_1/dt=20220102

# 直接查询表partition_1有多少分区
hive (default)> show partitions partition_1;
OK
partition
dt=20200101
dt=20220102
Time taken: 0.079 seconds, Fetched: 2 row(s)

# 直接删除某个分区，可发现hdfs中的整个分区目录被删除了
hive (default)> alter table partition_1 drop partition(dt='20220102');
Dropped the partition dt=20220102
OK
Time taken: 0.332 seconds
hive (default)> show partitions partition_1;
OK
partition
dt=20200101
Time taken: 0.072 seconds, Fetched: 1 row(s)
[root@bigdata04 hivedata]# hdfs dfs -ls /user/hive/warehouse/partition_1
Found 1 items
drwxr-xr-x   - root supergroup          0 2022-08-29 05:46 /user/hive/warehouse/partition_1/dt=20200101

# 查看原始数据
[root@bigdata04 hivedata]# more partition_2.data 
1       zhangsan
2       lisi
3       wangwu

# 创建内部分区表（含多个分区字段）
hive (default)> create table partition_2(
              >   id int,
              >   name string
              > ) 
              > partitioned by (year int, school string)
              > row format delimited 
              > fields terminated by '\t'
              > lines terminated by '\n';
OK
Time taken: 0.061 seconds

# 查看分区表结构
hive (default)> desc partition_2;
OK
col_name        data_type       comment
id                      int                                         
name                    string                                      
year                    int                                         
school                  string                                                   
# Partition Information          
# col_name              data_type               comment             
year                    int                                         
school                  string                                      
Time taken: 0.175 seconds, Fetched: 9 row(s)

# 加载数据
hive (default)> load data local inpath '/data/hivedata/partition_2.data' into table partition_2 partition(year=2020, school='xk');
Loading data to table default.partition_2 partition (year=2020, school=xk)
OK
Time taken: 0.512 seconds
hive (default)> load data local inpath '/data/hivedata/partition_2.data' into table partition_2 partition(year=2020, school='english');
Loading data to table default.partition_2 partition (year=2020, school=english)
OK
Time taken: 0.343 seconds
hive (default)> load data local inpath '/data/hivedata/partition_2.data' into table partition_2 partition(year=2019, school='english');
Loading data to table default.partition_2 partition (year=2019, school=english)
OK
Time taken: 0.392 seconds
hive (default)> load data local inpath '/data/hivedata/partition_2.data' into table partition_2 partition(year=2019, school='xk');
Loading data to table default.partition_2 partition (year=2019, school=xk)
OK
Time taken: 0.381 seconds

# 查看在hdfs中，分区表所有分区的目录
[root@bigdata04 hivedata]# hdfs dfs -ls /user/hive/warehouse/partition_2
Found 2 items
drwxr-xr-x   - root supergroup          0 2022-08-29 05:55 /user/hive/warehouse/partition_2/year=2019
drwxr-xr-x   - root supergroup          0 2022-08-29 05:55 /user/hive/warehouse/partition_2/year=2020
[root@bigdata04 hivedata]# hdfs dfs -ls /user/hive/warehouse/partition_2/year=2019
Found 2 items
drwxr-xr-x   - root supergroup          0 2022-08-29 05:55 /user/hive/warehouse/partition_2/year=2019/school=english
drwxr-xr-x   - root supergroup          0 2022-08-29 05:55 /user/hive/warehouse/partition_2/year=2019/school=xk

# 查询分区表
hive (default)> select * from partition_2;
OK
partition_2.id  partition_2.name        partition_2.year        partition_2.school
1       zhangsan        2019    english
2       lisi    2019    english
3       wangwu  2019    english
1       zhangsan        2019    xk
2       lisi    2019    xk
3       wangwu  2019    xk
1       zhangsan        2020    english
2       lisi    2020    english
3       wangwu  2020    english
1       zhangsan        2020    xk
2       lisi    2020    xk
3       wangwu  2020    xk
Time taken: 0.16 seconds, Fetched: 12 row(s)
hive (default)> select * from partition_2 where year=2019;
OK
partition_2.id  partition_2.name        partition_2.year        partition_2.school
1       zhangsan        2019    english
2       lisi    2019    english
3       wangwu  2019    english
1       zhangsan        2019    xk
2       lisi    2019    xk
3       wangwu  2019    xk
Time taken: 0.355 seconds, Fetched: 6 row(s)
hive (default)> select * from partition_2 where year=2019 and school='xk';
OK
partition_2.id  partition_2.name        partition_2.year        partition_2.school
1       zhangsan        2019    xk
2       lisi    2019    xk
3       wangwu  2019    xk
```

###### 2）外部分区表

```shell
# 查看原始数据
[root@bigdata04 hivedata]# more ex_par.data 
1       zhangsan
2       lisi
3       wangwu

# 创建外部分区表
hive (default)> create external table ex_par(
              >   id int,
              >   name string
              > ) 
              > partitioned by (dt string)
              > row format delimited 
              > fields terminated by '\t'
              > lines terminated by '\n'
              > location '/data/ex_par';
OK
Time taken: 0.643 seconds

# 加载数据
hive (default)> load data local inpath '/data/hivedata/ex_par.data' into table ex_par partition(dt='20200101');
Loading data to table default.ex_par partition (dt=20200101)
OK
Time taken: 1.039 seconds

# 查看在hdfs中，分区表所有分区的目录
[root@bigdata04 hivedata]# hdfs dfs -ls /data/ex_par
Found 1 items
drwxr-xr-x   - root supergroup          0 2022-08-29 06:10 /data/ex_par/dt=20200101

# 直接删除某个分区，可发现分区逻辑被删除了，但hdfs中的整个分区目录还在
hive (default)> alter table ex_par drop partition(dt='20200101');
Dropped the partition dt=20200101
OK
Time taken: 0.221 seconds
hive (default)> show partitions ex_par;
OK
partition
Time taken: 0.139 seconds
[root@bigdata04 hivedata]# hdfs dfs -ls /data/ex_par
Found 1 items
drwxr-xr-x   - root supergroup          0 2022-08-29 06:10 /data/ex_par/dt=20200101
hive (default)> select * from ex_par;
OK
ex_par.id       ex_par.name     ex_par.dt
Time taken: 0.178 seconds

# 重新补回被删除的分区
hive (default)> alter table ex_par add partition(dt='20200101') location '/data/ex_par/dt=20200101';
OK
Time taken: 0.133 seconds
hive (default)> select * from ex_par;
OK
ex_par.id       ex_par.name     ex_par.dt
1       zhangsan        20200101
2       lisi    20200101
3       wangwu  20200101
Time taken: 0.334 seconds, Fetched: 3 row(s)
```

##### 4、桶表

1. 桶表，可以对数据进行哈希取值，然后放到不同文件中存储，此时 hdfs 的表目录下的每个文件称为桶。
2. 可以用于数据抽样、提高  join 的查询效率。

```shell
# 查看原始数据
hive (default)> select * from b_source;
OK
b_source.id
1
2
3
4
5
6
7
8
9
10
11
12

# 创建桶表
hive (default)> create table bucket_tb(
              >   id int
              > ) 
              > clustered by (id) into 4 buckets;
OK
Time taken: 0.099 seconds

# 开启自动设置Reduce数量
hive (default)> set hive.enforce.bucketing=true;

# 插入数据到桶表，底层执行MapReduce：只能通过查询其他表导入，不能直接插入
hive (default)> insert into table bucket_tb select id from b_source where id is not null;
Query ID = root_20220829072508_84ef72f8-f9e4-40cd-b67a-8e208056e7cd
Total jobs = 2
Launching Job 1 out of 2
...

# 查询桶表
hive (default)> select * from bucket_tb;
OK
bucket_tb.id
12
8
4
9
5
1
10
6
2
11
7
3

# 查看hdfs实际目录
[root@bigdata04 ~]# hdfs dfs -ls /user/hive/warehouse/bucket_tb
Found 4 items
-rw-r--r--   2 root supergroup          7 2022-08-29 07:25 /user/hive/warehouse/bucket_tb/000000_0
-rw-r--r--   2 root supergroup          6 2022-08-29 07:25 /user/hive/warehouse/bucket_tb/000001_0
-rw-r--r--   2 root supergroup          7 2022-08-29 07:25 /user/hive/warehouse/bucket_tb/000002_0
-rw-r--r--   2 root supergroup          7 2022-08-29 07:25 /user/hive/warehouse/bucket_tb/000003_0

# 桶表抽样：y>=x，y：表示把桶表中的数据随机分为多少桶，x: 表示取出第几桶的数据
hive (default)> select * from bucket_tb tablesample(bucket 1 out of 4 on id);
OK
bucket_tb.id
10
6
2
7
Time taken: 0.153 seconds, Fetched: 4 row(s)
hive (default)> select * from bucket_tb tablesample(bucket 2 out of 4 on id);
OK
bucket_tb.id
8
4
9
1
11
Time taken: 0.091 seconds, Fetched: 5 row(s)
hive (default)> select * from bucket_tb tablesample(bucket 3 out of 4 on id);
OK
bucket_tb.id
3
Time taken: 0.083 seconds, Fetched: 1 row(s)
hive (default)> select * from bucket_tb tablesample(bucket 4 out of 4 on id);
OK
bucket_tb.id
12
5
Time taken: 0.09 seconds, Fetched: 2 row(s)
```

##### 5、视图

使用视图，可以降低查询的复杂度

```shell
# 查询所有表
hive (default)> show tables;
OK
tab_name
student

# 查看student表结构
hive (default)> desc student;
OK
col_name        data_type       comment
id                      int                                         
name                    string                                      
favors                  array<string>                               
scores                  map<string,int>                             
address                 struct<home_addr:string,office_addr:string> 

# 创建视图
hive (default)> create view v1 as select id, name from student;
OK
id      name
Time taken: 0.295 seconds

# 查询所有表
hive (default)> show tables;
OK
tab_name
student
v1

# 查看v1视图结构
hive (default)> desc v1;
OK
col_name        data_type       comment
id                      int                                         
name                    string    

# 查询v1视图
hive (default)> select * from v1;
OK
v1.id   v1.name
1       zhangsan
2       lisi

# hdfs上没有v1视图对应的文件
[root@bigdata04 ~]# hdfs dfs -ls /user/hive/warehouse
Found 1 items
drwxr-xr-x   - root supergroup          0 2022-08-29 05:11 /user/hive/warehouse/student

# 删除v1视图
hive (default)> drop view v1;
OK
hive (default)> show tables;
OK
tab_name
student
```

#### 5）函数操作

##### 1、查看所有函数

```shell
hive (default)> show functions;
...
~
Time taken: 0.479 seconds, Fetched: 289 row(s)
```

##### 2、查看函数的描述信息

```shell
hive (default)> desc function year;
OK
tab_name
year(param) - Returns the year component of the date/timestamp/interval
Time taken: 0.025 seconds, Fetched: 1 row(s)

hive (default)> desc function extended year;
OK
tab_name
year(param) - Returns the year component of the date/timestamp/interval
param can be one of:
1. A string in the format of 'yyyy-MM-dd HH:mm:ss' or 'yyyy-MM-dd'.
2. A date value
3. A timestamp value
4. A year-month interval valueExample:
   > SELECT year('2009-07-30') FROM src LIMIT 1;
  2009
Function class:org.apache.hadoop.hive.ql.udf.UDFYear
Function type:BUILTIN
Time taken: 0.009 seconds, Fetched: 10 row(s)
```

##### 3、TopN 实现

```shell
# 查看原始数据
[root@bigdata04 hivedata]# more student_score.data 
1       zs1     chinese 80
2       zs1     math    90
3       zs1     english 89
4       zs2     chinese 60
5       zs2     math    75
6       zs2     english 80
7       zs3     chinese 79
8       zs3     math    83
9       zs3     english 72
10      zs4     chinese 90
11      zs4     math    76
12      zs4     english 80
13      zs5     chinese 98
14      zs5     math    80
15      zs5     english 70

# 建立外部表
hive (default)> create external table student_score(
              >   id int,
              >   name string,
              >   sub string,
              > score int
              > ) row format delimited 
              > fields terminated by '\t'
              > lines terminated by '\n'
              > location '/data/student_score';
OK
Time taken: 0.258 seconds

# 加载数据
[root@bigdata04 hivedata]# hdfs dfs -put student_score.data /data/student_score

# 查询结果
hive (default)> select * from student_score;
OK
student_score.id        student_score.name      student_score.sub       student_score.score
1       zs1     chinese 80
2       zs1     math    90
3       zs1     english 89
4       zs2     chinese 60
5       zs2     math    75
6       zs2     english 80
7       zs3     chinese 79
8       zs3     math    83
9       zs3     english 72
10      zs4     chinese 90
11      zs4     math    76
12      zs4     english 80
13      zs5     chinese 98
14      zs5     math    80
15      zs5     english 70
Time taken: 1.107 seconds, Fetched: 15 row(s)

# 对全局数据做编号：底层依赖MapReduce任务
hive (default)> select *, row_number() over() from student_score;
Query ID = root_20220830193501_5c924e7b-cf97-4341-a4b3-e1303f878fb4
Total jobs = 1
Launching Job 1 out of 1
...
Total MapReduce CPU Time Spent: 1 seconds 850 msec
OK
student_score.id        student_score.name      student_score.sub       student_score.score     row_number_window_0
15      zs5     english 70      1
14      zs5     math    80      2
13      zs5     chinese 98      3
12      zs4     english 80      4
11      zs4     math    76      5
10      zs4     chinese 90      6
9       zs3     english 72      7
8       zs3     math    83      8
7       zs3     chinese 79      9
6       zs2     english 80      10
5       zs2     math    75      11
4       zs2     chinese 60      12
3       zs1     english 89      13
2       zs1     math    90      14
1       zs1     chinese 80      15
Time taken: 22.83 seconds, Fetched: 15 row(s)

# TopN实现：先按sub分组，然后按score倒序排序，再顺序编号
hive (default)> select *, row_number() over(partition by sub order by score desc) from student_score;
Query ID = root_20220830193827_6632878e-e857-40b2-8e0d-eb5ba9dc34ef
Total jobs = 1
Launching Job 1 out of 1
...
Total MapReduce CPU Time Spent: 1 seconds 900 msec
OK
student_score.id        student_score.name      student_score.sub       student_score.score     row_number_window_0
13      zs5     chinese 98      1
10      zs4     chinese 90      2
1       zs1     chinese 80      3
7       zs3     chinese 79      4
4       zs2     chinese 60      5
3       zs1     english 89      1
6       zs2     english 80      2
12      zs4     english 80      3
9       zs3     english 72      4
15      zs5     english 70      5
2       zs1     math    90      1
8       zs3     math    83      2
14      zs5     math    80      3
11      zs4     math    76      4
5       zs2     math    75      5
Time taken: 19.437 seconds, Fetched: 15 row(s)

# Top3实现：先按sub分组，然后按score倒序排序，再顺序编号
hive (default)> select t.* 
              > from (
              >  select *, row_number() over(partition by sub order by score desc) num from student_score
              > ) t where t.num <= 3;
...
2022-08-30 19:41:47,323 Stage-1 map = 0%,  reduce = 0%
2022-08-30 19:41:52,511 Stage-1 map = 100%,  reduce = 0%, Cumulative CPU 0.73 sec
2022-08-30 19:41:57,741 Stage-1 map = 100%,  reduce = 100%, Cumulative CPU 2.18 sec
...
Total MapReduce CPU Time Spent: 2 seconds 180 msec
OK
t.id    t.name  t.sub   t.score t.num
13      zs5     chinese 98      1
10      zs4     chinese 90      2
1       zs1     chinese 80      3
3       zs1     english 89      1
6       zs2     english 80      2
12      zs4     english 80      3
2       zs1     math    90      1
8       zs3     math    83      2
14      zs5     math    80      3
Time taken: 18.813 seconds, Fetched: 9 row(s)

# Rank3实现：先按sub分组，然后按score倒序排序，再顺序排名（相同分数的为同一名，人数累加）
hive (default)> select t.* 
              > from (
              >  select *, rank() over(partition by sub order by score desc) num from student_score
              > ) t where t.num <= 3;
...
Hadoop job information for Stage-1: number of mappers: 1; number of reducers: 1
2022-08-30 19:44:05,083 Stage-1 map = 0%,  reduce = 0%
2022-08-30 19:44:10,313 Stage-1 map = 100%,  reduce = 0%, Cumulative CPU 0.93 sec
2022-08-30 19:44:17,445 Stage-1 map = 100%,  reduce = 100%, Cumulative CPU 2.41 sec
...
Total MapReduce CPU Time Spent: 2 seconds 410 msec
OK
t.id    t.name  t.sub   t.score t.num
13      zs5     chinese 98      1
10      zs4     chinese 90      2
1       zs1     chinese 80      3
3       zs1     english 89      1
12      zs4     english 80      2
6       zs2     english 80      2
2       zs1     math    90      1
8       zs3     math    83      2
14      zs5     math    80      3
Time taken: 19.433 seconds, Fetched: 9 row(s)

# Rank3实现：先按sub分组，然后按score倒序排序，再顺序排名（相同分数的为同一名，人数不累加）
hive (default)> select t.* 
              > from (
              >  select *, dense_rank() over(partition by sub order by score desc) num from student_score
              > ) t where t.num <= 3;
...
Hadoop job information for Stage-1: number of mappers: 1; number of reducers: 1
2022-08-30 19:46:19,172 Stage-1 map = 0%,  reduce = 0%
2022-08-30 19:46:24,297 Stage-1 map = 100%,  reduce = 0%, Cumulative CPU 0.82 sec
2022-08-30 19:46:29,536 Stage-1 map = 100%,  reduce = 100%, Cumulative CPU 2.18 sec
...
Total MapReduce CPU Time Spent: 2 seconds 180 msec
OK
t.id    t.name  t.sub   t.score t.num
13      zs5     chinese 98      1
10      zs4     chinese 90      2
1       zs1     chinese 80      3
3       zs1     english 89      1
12      zs4     english 80      2
6       zs2     english 80      2
9       zs3     english 72      3
2       zs1     math    90      1
8       zs3     math    83      2
14      zs5     math    80      3
Time taken: 20.983 seconds, Fetched: 10 row(s)
```

##### 4、行转列实现

行转列，即把多行数据，转换为一列数据。

```shell
# 查询原始数据
[root@bigdata04 hivedata]# more student_favors.data 
zs      swing
zs      footbal
zs      sing
zs      codeing
zs      swing

# 建立外部表
hive (default)> create external table student_favors(
              >   name string,
              >   favor string
              > ) row format delimited 
              > fields terminated by '\t'
              > lines terminated by '\n'
              > location '/data/student_favors';
OK
Time taken: 0.43 seconds

# 加载数据
[root@bigdata04 hivedata]# hdfs dfs -put student_favors.data /data/student_favors

# 查询结果
hive (default)> select * from student_favors;
OK
student_favors.name     student_favors.favor
zs      swing
zs      footbal
zs      sing
zs      codeing
zs      swing
Time taken: 1.144 seconds, Fetched: 5 row(s)

# collect_list行转[数组]
hive (default)> select name, collect_list(favor) as favor_list from student_favors group by name;
...
Hadoop job information for Stage-1: number of mappers: 1; number of reducers: 1
2022-08-30 19:59:28,311 Stage-1 map = 0%,  reduce = 0%
2022-08-30 19:59:34,494 Stage-1 map = 100%,  reduce = 0%, Cumulative CPU 0.83 sec
2022-08-30 19:59:38,614 Stage-1 map = 100%,  reduce = 100%, Cumulative CPU 1.67 sec
MapReduce Total cumulative CPU time: 1 seconds 670 msec
Ended Job = job_1661858656416_0006
MapReduce Jobs Launched: 
Stage-Stage-1: Map: 1  Reduce: 1   Cumulative CPU: 1.67 sec   HDFS Read: 8802 HDFS Write: 135 SUCCESS
Total MapReduce CPU Time Spent: 1 seconds 670 msec
OK
name    favor_list
zs      ["swing","footbal","sing","codeing","swing"]
Time taken: 19.581 seconds, Fetched: 1 row(s)

# collect_list行转[数组]，[数组]转字符串
hive (default)> select name, concat_ws(',', collect_list(favor)) as favor_list from student_favors group by name;
...
Hadoop job information for Stage-1: number of mappers: 1; number of reducers: 1
2022-08-30 20:02:06,449 Stage-1 map = 0%,  reduce = 0%
2022-08-30 20:02:12,577 Stage-1 map = 100%,  reduce = 0%, Cumulative CPU 0.74 sec
2022-08-30 20:02:17,790 Stage-1 map = 100%,  reduce = 100%, Cumulative CPU 2.09 sec
MapReduce Total cumulative CPU time: 2 seconds 90 msec
Ended Job = job_1661858656416_0007
MapReduce Jobs Launched: 
Stage-Stage-1: Map: 1  Reduce: 1   Cumulative CPU: 2.09 sec   HDFS Read: 9278 HDFS Write: 135 SUCCESS
Total MapReduce CPU Time Spent: 2 seconds 90 msec
OK
name    favor_list
zs      swing,footbal,sing,codeing,swing
Time taken: 18.549 seconds, Fetched: 1 row(s)

# collect_set去重、并行转[数组]，[数组]转字符串
hive (default)> select name, concat_ws(',', collect_set(favor)) as favor_list from student_favors group by name;
...
Hadoop job information for Stage-1: number of mappers: 1; number of reducers: 1
2022-08-30 20:03:11,431 Stage-1 map = 0%,  reduce = 0%
2022-08-30 20:03:16,632 Stage-1 map = 100%,  reduce = 0%, Cumulative CPU 0.94 sec
2022-08-30 20:03:23,779 Stage-1 map = 100%,  reduce = 100%, Cumulative CPU 2.3 sec
MapReduce Total cumulative CPU time: 2 seconds 300 msec
Ended Job = job_1661858656416_0008
MapReduce Jobs Launched: 
Stage-Stage-1: Map: 1  Reduce: 1   Cumulative CPU: 2.3 sec   HDFS Read: 9274 HDFS Write: 129 SUCCESS
Total MapReduce CPU Time Spent: 2 seconds 300 msec
OK
name    favor_list
zs      swing,footbal,sing,codeing
Time taken: 20.598 seconds, Fetched: 1 row(s)
```

##### 5、列转行实现

列转行，即把一列数据，转换为多行数据。

```shell
# 查看原始数据
[root@bigdata04 hivedata]# more student_favors_2.data 
zs      swing,footbal,sing
ls      codeing,swing

# 建立外部表
hive (default)> create external table student_favors_2(
              >   name string,
              >   favorlist string
              > ) row format delimited 
              > fields terminated by '\t'
              > lines terminated by '\n'
              > location '/data/student_favors_2';
OK
Time taken: 0.088 seconds

# 加载数据
[root@bigdata04 hivedata]# hdfs dfs -put student_favors_2.data /data/student_favors_2

# 查询结果
hive (default)> select * from student_favors_2;
OK
student_favors_2.name   student_favors_2.favorlist
zs      swing,footbal,sing
ls      codeing,swing
Time taken: 0.23 seconds, Fetched: 2 row(s)

# 本地切割字符串, 无需MapReduce任务
hive (default)> select split(favorlist, ',') from student_favors_2;
OK
_c0
["swing","footbal","sing"]
["codeing","swing"]
Time taken: 0.193 seconds, Fetched: 2 row(s)

# explode列专行，无需MapReduce任务
hive (default)> select explode(split(favorlist, ',')) from student_favors_2;
OK
col
swing
footbal
sing
codeing
swing
Time taken: 0.126 seconds, Fetched: 5 row(s)

# lateral view对多行建立虚拟表，然后与student_favors_2关联
hive (default)> select name, favor_new from student_favors_2 
              > lateral view
              > explode(
              >   split(favorlist, ',')
              > ) table1 as favor_new;
OK
name    favor_new
zs      swing
zs      footbal
zs      sing
ls      codeing
ls      swing
Time taken: 0.069 seconds, Fetched: 5 row(s)
```

##### 6、排序实现

| 函数          | 作用                                                         |
| ------------- | ------------------------------------------------------------ |
| ORDER BY      | 全局排序，生成的 Reduce 任务只有一个                         |
| SORT BY       | 局部排序，多个 Reduce 任务无法保证全局有序，可以用 `set mapreduce.job.reduces=2` 来设置 Reduce 的个数 |
| DISTRIBUTE BY | 只分区不排序，用来控制 Map 到 Reduce 之间是如何划分的，一般和 sort by 结合起来使用，但 distribute by 一定要写在 sort by 之前 |
| CLUSTER BY    | 等价于 `distribute by + sort by`，但只支持升序，不支持降序   |

```shell
hive (default)> select * from t2_bak;
OK
t2_bak.id       t2_bak.name
0       NULL
1       NULL
2       NULL
3       NULL
4       NULL
5       NULL
1       NULL
2       NULL
3       NULL
4       NULL
5       NULL
Time taken: 0.152 seconds, Fetched: 11 row(s)

# 强制设置reduce个数=2
hive (default)> set mapreduce.job.reduces=2;

# reduces对order by无效，产生全局排序的效果
hive (default)> select * from t2_bak order by id;
...
Hadoop job information for Stage-1: number of mappers: 1; number of reducers: 1
2022-08-30 20:19:30,335 Stage-1 map = 0%,  reduce = 0%
2022-08-30 20:19:36,459 Stage-1 map = 100%,  reduce = 0%, Cumulative CPU 0.87 sec
2022-08-30 20:19:41,543 Stage-1 map = 100%,  reduce = 100%, Cumulative CPU 2.07 sec
MapReduce Total cumulative CPU time: 2 seconds 70 msec
Ended Job = job_1661858656416_0011
MapReduce Jobs Launched: 
Stage-Stage-1: Map: 1  Reduce: 1   Cumulative CPU: 2.07 sec   HDFS Read: 10031 HDFS Write: 274 SUCCESS
Total MapReduce CPU Time Spent: 2 seconds 70 msec
OK
t2_bak.id       t2_bak.name
0       NULL
1       NULL
1       NULL
2       NULL
2       NULL
3       NULL
3       NULL
4       NULL
4       NULL
5       NULL
5       NULL
Time taken: 18.006 seconds, Fetched: 11 row(s)

# reduces对sort by有效，产生局部排序的效果
hive (default)> select * from t2_bak sort by id;
...
Hadoop job information for Stage-1: number of mappers: 1; number of reducers: 2
2022-08-30 20:18:00,272 Stage-1 map = 0%,  reduce = 0%
2022-08-30 20:18:04,417 Stage-1 map = 100%,  reduce = 0%, Cumulative CPU 0.73 sec
2022-08-30 20:18:10,636 Stage-1 map = 100%,  reduce = 50%, Cumulative CPU 1.87 sec
2022-08-30 20:18:11,658 Stage-1 map = 100%,  reduce = 100%, Cumulative CPU 3.01 sec
MapReduce Total cumulative CPU time: 3 seconds 10 msec
Ended Job = job_1661858656416_0010
MapReduce Jobs Launched: 
Stage-Stage-1: Map: 1  Reduce: 2   Cumulative CPU: 3.01 sec   HDFS Read: 14943 HDFS Write: 361 SUCCESS
Total MapReduce CPU Time Spent: 3 seconds 10 msec
OK
t2_bak.id       t2_bak.name
0       NULL
2       NULL
2       NULL
3       NULL
4       NULL
4       NULL
5       NULL
1       NULL
1       NULL
3       NULL
5       NULL
Time taken: 19.511 seconds, Fetched: 11 row(s)

# reduces对distribute by有效，产生分区的效果
hive (default)> select * from t2_bak distribute by id;
...
Hadoop job information for Stage-1: number of mappers: 1; number of reducers: 2
2022-08-30 20:21:47,534 Stage-1 map = 0%,  reduce = 0%
2022-08-30 20:21:52,622 Stage-1 map = 100%,  reduce = 0%, Cumulative CPU 0.66 sec
2022-08-30 20:21:58,842 Stage-1 map = 100%,  reduce = 100%, Cumulative CPU 2.95 sec
MapReduce Total cumulative CPU time: 2 seconds 950 msec
Ended Job = job_1661858656416_0012
MapReduce Jobs Launched: 
Stage-Stage-1: Map: 1  Reduce: 2   Cumulative CPU: 2.95 sec   HDFS Read: 14895 HDFS Write: 361 SUCCESS
Total MapReduce CPU Time Spent: 2 seconds 950 msec
OK
t2_bak.id       t2_bak.name
4       NULL
2       NULL
4       NULL
2       NULL
0       NULL
5       NULL
3       NULL
1       NULL
5       NULL
3       NULL
1       NULL
Time taken: 18.963 seconds, Fetched: 11 row(s)

# reduces对distribute by+sort by有效，产生分区+分区后排序的效果
hive (default)> select * from t2_bak distribute by id sort by id;
Hadoop job information for Stage-1: number of mappers: 1; number of reducers: 2
2022-08-30 20:22:57,436 Stage-1 map = 0%,  reduce = 0%
2022-08-30 20:23:02,661 Stage-1 map = 100%,  reduce = 0%, Cumulative CPU 0.95 sec
2022-08-30 20:23:08,893 Stage-1 map = 100%,  reduce = 50%, Cumulative CPU 2.04 sec
2022-08-30 20:23:09,907 Stage-1 map = 100%,  reduce = 100%, Cumulative CPU 3.01 sec
MapReduce Total cumulative CPU time: 3 seconds 10 msec
Ended Job = job_1661858656416_0013
MapReduce Jobs Launched: 
Stage-Stage-1: Map: 1  Reduce: 2   Cumulative CPU: 3.01 sec   HDFS Read: 15028 HDFS Write: 361 SUCCESS
Total MapReduce CPU Time Spent: 3 seconds 10 msec
OK
t2_bak.id       t2_bak.name
0       NULL
2       NULL
2       NULL
4       NULL
4       NULL
1       NULL
1       NULL
3       NULL
3       NULL
5       NULL
5       NULL
Time taken: 19.618 seconds, Fetched: 11 row(s)

# reduces对cluster by有效，产生分区+分区后排序的效果，等价于distribute by+sort by
hive (default)> select * from t2_bak cluster by id;
...
Hadoop job information for Stage-1: number of mappers: 1; number of reducers: 2
2022-08-30 20:25:13,075 Stage-1 map = 0%,  reduce = 0%
2022-08-30 20:25:18,298 Stage-1 map = 100%,  reduce = 0%, Cumulative CPU 0.87 sec
2022-08-30 20:25:23,511 Stage-1 map = 100%,  reduce = 50%, Cumulative CPU 1.89 sec
2022-08-30 20:25:24,525 Stage-1 map = 100%,  reduce = 100%, Cumulative CPU 2.94 sec
MapReduce Total cumulative CPU time: 2 seconds 940 msec
Ended Job = job_1661858656416_0014
MapReduce Jobs Launched: 
Stage-Stage-1: Map: 1  Reduce: 2   Cumulative CPU: 2.94 sec   HDFS Read: 15042 HDFS Write: 361 SUCCESS
Total MapReduce CPU Time Spent: 2 seconds 940 msec
OK
t2_bak.id       t2_bak.name
0       NULL
2       NULL
2       NULL
4       NULL
4       NULL
1       NULL
1       NULL
3       NULL
3       NULL
5       NULL
5       NULL
Time taken: 18.318 seconds, Fetched: 11 row(s)
```

##### 7、分组 & 去重实现

```shell
# distinct，去重，设置reduces无效，会将所有的id，都shuffle到同一个Reduce中执行，数据量大时，性能很差
hive (default)> select count(distinct id) from t2_bak;
...
Hadoop job information for Stage-1: number of mappers: 1; number of reducers: 1
2022-08-30 20:30:21,356 Stage-1 map = 0%,  reduce = 0%
2022-08-30 20:30:26,457 Stage-1 map = 100%,  reduce = 0%, Cumulative CPU 0.76 sec
2022-08-30 20:30:32,656 Stage-1 map = 100%,  reduce = 100%, Cumulative CPU 1.76 sec
MapReduce Total cumulative CPU time: 1 seconds 760 msec
Ended Job = job_1661858656416_0015
MapReduce Jobs Launched: 
Stage-Stage-1: Map: 1  Reduce: 1   Cumulative CPU: 1.76 sec   HDFS Read: 8535 HDFS Write: 101 SUCCESS
Total MapReduce CPU Time Spent: 1 seconds 760 msec
OK
_c0
6
Time taken: 19.038 seconds, Fetched: 1 row(s)

# group by，先分组，设置reduces有效，让相同的id，进到同一个Reduce任务，且多个Reduce任务并行计算，最终再count，数据量大时，效率高
hive (default)> select count(*) from (select id from t2_bak group by id) t;
...
Hadoop job information for Stage-2: number of mappers: 1; number of reducers: 1
2022-08-30 20:32:17,217 Stage-2 map = 0%,  reduce = 0%
2022-08-30 20:32:23,340 Stage-2 map = 100%,  reduce = 0%, Cumulative CPU 0.9 sec
2022-08-30 20:32:27,485 Stage-2 map = 100%,  reduce = 100%, Cumulative CPU 1.79 sec
MapReduce Total cumulative CPU time: 1 seconds 790 msec
Ended Job = job_1661858656416_0018
MapReduce Jobs Launched: 
Stage-Stage-1: Map: 1  Reduce: 2   Cumulative CPU: 2.68 sec   HDFS Read: 16390 HDFS Write: 228 SUCCESS
Stage-Stage-2: Map: 1  Reduce: 1   Cumulative CPU: 1.79 sec   HDFS Read: 7342 HDFS Write: 101 SUCCESS
Total MapReduce CPU Time Spent: 4 seconds 470 msec
OK
_c0
6
Time taken: 42.26 seconds, Fetched: 1 row(s)
```

##### 8、UDF 自定义函数实现

###### 1）引入依赖

```xml
<dependency>
    <groupId>org.apache.hive</groupId>
    <artifactId>hive-exec</artifactId>
    <version>3.1.2</version>
</dependency>
```

###### 2）继承 UDF 类

```java
package hive.udf;

import org.apache.hadoop.hive.ql.exec.UDF;
import org.apache.hadoop.io.Text;

public class UpFirst extends UDF {
    public Text evaluate(final Text s) {
        if (s == null) {
            return null;
        }
        String str = s.toString();
        if (str.length() == 0) {
            return new Text(str);
        }
        if (str.length() == 1) {
            str = str.substring(0, 1).toUpperCase();
        } else {
            str = str.substring(0, 1).toUpperCase() + str.substring(1);
        }
        return new Text(str);
    }
}
```

###### 3）打包、上传

```shell
hive> add jar /usr/local/hive-1.0-SNAPSHOT.jar;
Added [/usr/local/hive-1.0-SNAPSHOT.jar] to class path
Added resources: [/usr/local/hive-1.0-SNAPSHOT.jar]

hive> list jars;
/usr/local/hive-1.0-SNAPSHOT.jar 
```

###### 4）创建临时函数

```shell
hive> create temporary function up_first as 'hive.udf.UpFirst';
```

###### 5）使用自定义函数

```shell
hive> select up_first(name),age from my_users;
OK
Tom     18
Jack    20
Jessic  30
Time taken: 0.14 seconds, Fetched: 3 row(s)
```

### 1.7. Hive 性能优化？

#### 1）数据倾斜问题

```sql
SELECT
    a.key,
    SUM(a.cnt) AS cnt
FROM
(
    SELECT
        key,
        COUNT(*) AS cnt
    FROM
        tableName
    -- 先根据key分组，再将KEY001打散成50份
    GROUP BY
        key,
        CASE WHEN key = 'KEY001' THEN Hash(Random()) % 50 ELSE 0 END
)a
GROUP BY
    key
```

#### 2）数据压缩格式

TextFile 类型的数据存储格式，浪费存储空间，而想要发挥数据存储格式的最大性能，需要配合数据压缩格式一起使用。

1. 是否可切分：意思是，对于 TextFile 类型的文件，在 MapReduce#Map 阶段，能否产生多个 InputSplit，如果不可切分，则无论压缩文件有多大，都只会产生一个 Map 任务，大数据量时不能并行计算，性能较差。
2. 压缩比：越高代表压缩效果越好，产生压缩文件越小，如果追求存储空间，则重点关注压缩比。
3. 压缩速度：MapReduce 在压缩文件消耗的时间，会包含在整个 MapReduce 任务的总消耗时间中。
4. 解压速度：MapReduce 在使用压缩文件时，需要先对压缩文件进行解压，解压消耗的时间包含在整个 MapReduce 任务的总消耗时间中。

| 压缩格式    | Hadoop 自带包  | 文件扩展名 | 是否可切分         | 压缩比 | 压缩速度 | 解压速度 |
| ----------- | -------------- | ---------- | ------------------ | ------ | -------- | -------- |
| **DEFLATE** | zlib           | .deflate   | 否                 | 中     | 中       | 中       |
| **Gzip**    | deflate        | .gz        | 否                 | 中     | 中       | 中       |
| **Bzip2**   | bzip2          | .bz2       | 是                 | 高     | 低       | 低       |
| **Lz4**     | lz4            | .lz4       | 否                 | 低     | 高       | 高       |
| **Lzo**     | 无，需自行安装 | .lzo       | 是，但需要建立索引 | 低     | 高       | 高       |
| **Snappy**  | snappy         | .snappy    | 否                 | 低     | 高       | 高       |

同文件，不压缩、使用不同格式压缩后的大小对比：

![1661946594479](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1661946594479.png)

使用建议：

1. 优先使用包含压缩并支持切分的存储格式，比如 Sequence File、RCFile、ORCFile 等。

2. 而对于 TextFile 的存储格式，则建议使用支持切分的压缩格式，比如 Bzip2 和 Lzo。

3. 而如果不想考虑切分的问题，则可以提前把一个大文件拆分成多个块，先对每个块进行单独压缩，再执行 MapReduce 任务。

4. 设置压缩格式的位置有两个：Map 输出时设置，以及 Reduce 输出时设置。

   - 1）如果 Map / Reduce 阶段产生的文件需要永久保存，无 Reduce / 下一个MapReduce#Map 阶段，则考虑能否支持切分，推荐使用高压缩比、占空间小、但耗 CPU 的 Bzip2，或者低压缩比、占空间大些、但节省些 CPU 的 Lzo。
   - 2）而如果需要 Map / Reduce 阶段产生的文件不需要永久保存，直接传递到 Reduce / 下一个MapReduce#Map 阶段使用，则考虑节省 CPU，推荐使用低压缩比、占空间大些、但节省些 CPU、且支持切分的 Lzo，或者消耗差不多、但不支持切分的 Snappy。

   ![1661946544624](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1661946544624.png)

##### 1、未压缩

```shell
# 原始未压缩，输出2.2GB
hadoop jar db_hadoop-1.0-SNAPSHOT-jar-with-dependencies.jar com.imooc.compress.MrDataCompress /words.dat /words_out/non_compress
```

##### 2、Deflate 压缩

```shell
# 使用Deflate压缩，输出346.19mb
hadoop jar db_hadoop-1.0-SNAPSHOT-jar-with-dependencies.jar com.imooc.compress.MrDataCompress -Dmapreduce.output.fileoutputformat.compress=true -Dmapreduce.output.fileoutputformat.compress.codec=org.apache.hadoop.io.compress.DeflateCodec /words.dat /words_out/compress_deflate
# 读取输出文件，再次wordCount，只产生1个Map任务，说明使用Deflate压缩不支持切分
hadoop jar db_hadoop-1.0-SNAPSHOT-jar-with-dependencies.jar com.imooc.compress.MrDataCompress -Dmapreduce.output.fileoutputformat.compress=true -Dmapreduce.output.fileoutputformat.compress.codec=org.apache.hadoop.io.compress.DeflateCodec /words_out/compress_deflate /words_out/compress_deflate_test
```

##### 3、Gzip 压缩

```shell
# 使用Gzip压缩，输出346.19mb
hadoop jar db_hadoop-1.0-SNAPSHOT-jar-with-dependencies.jar com.imooc.compress.MrDataCompress -Dmapreduce.output.fileoutputformat.compress=true -Dmapreduce.output.fileoutputformat.compress.codec=org.apache.hadoop.io.compress.GzipCodec /words.dat /words_out/compress_gzip
# 读取输出文件，再次wordCount，只产生1个Map任务，说明使用Gzip压缩不支持切分
hadoop jar db_hadoop-1.0-SNAPSHOT-jar-with-dependencies.jar com.imooc.compress.MrDataCompress -Dmapreduce.output.fileoutputformat.compress=true -Dmapreduce.output.fileoutputformat.compress.codec=org.apache.hadoop.io.compress.GzipCodec /words_out/compress_gzip /words_out/compress_gzip_test
```

##### 4、Bzip2 压缩

```shell
# 使用Bzip2压缩，输出203.15mb
hadoop jar db_hadoop-1.0-SNAPSHOT-jar-with-dependencies.jar com.imooc.compress.MrDataCompress -Dmapreduce.output.fileoutputformat.compress=true -Dmapreduce.output.fileoutputformat.compress.codec=org.apache.hadoop.io.compress.BZip2Codec /words.dat /words_out/compress_bzip2
# 读取输出文件，再次wordCount，产生了2个Map任务，说明使用Bzip2压缩支持切分
hadoop jar db_hadoop-1.0-SNAPSHOT-jar-with-dependencies.jar com.imooc.compress.MrDataCompress -Dmapreduce.output.fileoutputformat.compress=true -Dmapreduce.output.fileoutputformat.compress.codec=org.apache.hadoop.io.compress.BZip2Codec /words_out/compress_bzip2 /words_out/compress_bzip2_test
```

##### 5、Lz4 压缩

```shell
# 使用Lz4压缩，输出562.63mb
hadoop jar db_hadoop-1.0-SNAPSHOT-jar-with-dependencies.jar com.imooc.compress.MrDataCompress -Dmapreduce.output.fileoutputformat.compress=true -Dmapreduce.output.fileoutputformat.compress.codec=org.apache.hadoop.io.compress.Lz4Codec /words.dat /words_out/compress_lz4
# 读取输出文件，再次wordCount，只产生1个Map任务，说明使用Lz4压缩不支持切分
hadoop jar db_hadoop-1.0-SNAPSHOT-jar-with-dependencies.jar com.imooc.compress.MrDataCompress -Dmapreduce.output.fileoutputformat.compress=true -Dmapreduce.output.fileoutputformat.compress.codec=org.apache.hadoop.io.compress.Lz4Codec /words_out/compress_lz4 /words_out/compress_lz4_test
```

##### 6、Snappy 压缩

```shell
# 使用Snappy压缩，输出651.12mb
hadoop jar db_hadoop-1.0-SNAPSHOT-jar-with-dependencies.jar com.imooc.compress.MrDataCompress -Dmapreduce.output.fileoutputformat.compress=true -Dmapreduce.output.fileoutputformat.compress.codec=org.apache.hadoop.io.compress.SnappyCodec /words.dat /words_out/compress_snappy
# 读取输出文件，再次wordCount，只产生1个Map任务，说明使用Snappy压缩不支持切分
hadoop jar db_hadoop-1.0-SNAPSHOT-jar-with-dependencies.jar com.imooc.compress.MrDataCompress -Dmapreduce.output.fileoutputformat.compress=true -Dmapreduce.output.fileoutputformat.compress.codec=org.apache.hadoop.io.compress.SnappyCodec /words_out/compress_snappy /words_out/compress_snappy_test
```

##### 6、Lzo 压缩

```shell
# 添加Lzo依赖
[root@bigdata01 common]# pwd
/data/soft/hadoop-3.2.0/share/hadoop/common
[root@bigdata01 common]# mv ~/hadoop-lzo-0.4.21-SNAPSHOT.jar .
[root@bigdata01 common]# scp -rq hadoop-lzo-0.4.21-SNAPSHOT.jar bigdata02:/data/soft/hadoop-3.2.0/share/hadoop/common/
[root@bigdata01 common]# scp -rq hadoop-lzo-0.4.21-SNAPSHOT.jar bigdata03:/data/soft/hadoop-3.2.0/share/hadoop/common/

# 添加Lzo配置
[root@bigdata01 hadoop]# pwd
/data/soft/hadoop-3.2.0/etc/hadoop
[root@bigdata01 hadoop]# vim core-site.xml
<!-- 配置使用的各种压缩算法的编/解码器 -->
<property>
	<name>io.compression.codecs</name>
	<value>
		org.apache.hadoop.io.compress.GzipCodec,
		org.apache.hadoop.io.compress.DefaultCodec,
		org.apache.hadoop.io.compress.DeflateCodec,
		org.apache.hadoop.io.compress.BZip2Codec,
		org.apache.hadoop.io.compress.SnappyCodec,
		org.apache.hadoop.io.compress.Lz4Codec,
		com.hadoop.compression.lzo.LzoCodec,
		com.hadoop.compression.lzo.LzopCodec
	</value>
</property>
<!-- 配置Lzo编解码器相关参数 -->
<property>
	<name>io.compression.codec.lzo.class</name>
	<value>com.hadoop.compression.lzo.LzoCodec</value>
</property>
[root@bigdata01 hadoop]# scp -rq core-site.xml bigdata02:/data/soft/hadoop-3.2.0/etc/hadoop/
[root@bigdata01 hadoop]# scp -rq core-site.xml bigdata03:/data/soft/hadoop-3.2.0/etc/hadoop/

# 重启Hadoop
[root@bigdata01 hadoop]# start-all.sh
...

# 使用Lzo压缩（指定lzopCodeC才支持建立索引，lzo则不支持），输出612.96mb
hadoop jar db_hadoop-1.0-SNAPSHOT-jar-with-dependencies.jar 
com.imooc.compress.MrDataCompress -Dmapreduce.output.fileoutputformat.compress=true -Dmapreduce.output.fileoutputformat.compress.codec=com.hadoop.compression.lzo.LzopCodec /words.dat /words_out/compress_lzo
# 读取输出文件，再次wordCount，只产生1个Map任务，说明使用Lzo压缩未建立索引时不支持切分
# 由于Lzo是第三方包，所以在读取Lzo文件时，需要指定Inputformat，即使用Lzo对应的Inputformat
hadoop jar db_hadoop-1.0-SNAPSHOT-jar-with-dependencies.jar com.imooc.compress.MrDataCompress -Dmapreduce.job.inputformat.class=com.hadoop.mapreduce.LzoTextInputFormat -Dmapreduce.output.fileoutputformat.compress=true -Dmapreduce.output.fileoutputformat.compress.codec=com.hadoop.compression.lzo.LzopCodec /words_out/compress_lzo /words_out/compress_lzo_test
# 建立Lzo索引
hadoop jar /data/soft/hadoop-3.2.0/share/hadoop/common/hadoop-lzo-0.4.21-SNAPSHOT.jar com.hadoop.compression.lzo.DistributedLzoIndexer /words_out/compress_lzo
# 重新验证，读取输出文件，再次wordCount，只产生5个Map任务，说明使用Lzo压缩建立索引后不支持切分
hdfs dfs -rm -r -skipTrash /words_out/compress_lzo_test
hadoop jar db_hadoop-1.0-SNAPSHOT-jar-with-dependencies.jar com.imooc.compress.MrDataCompress -Dmapreduce.job.inputformat.class=com.hadoop.mapreduce.LzoTextInputFormat -Dmapreduce.output.fileoutputformat.compress=true -Dmapreduce.output.fileoutputformat.compress.codec=com.hadoop.compression.lzo.LzopCodec /words_out/compress_lzo /words_out/compress_lzo_test
```

#### 3）数据存储格式

| 存储格式     | 特点                                                         |
| ------------ | ------------------------------------------------------------ |
| **TextFile** | 原生 Hadoop 格式、Hive 的默认存储格式，基于行存储，磁盘存储开销大（无压缩，但支持压缩）、数据解析开销大（逐个字符判断是否为分隔符） |
| SequenceFile | 原生 Hadoop 格式，是一个 `<key, value>` 形式的二进制文件，基于行存储，使用方便，支持切分，支持 `NONE、RECORD、BLOCK` 级别的压缩 |
| RCFile       | 专门为 Hive 设计的格式，数据首先按行分组，每组按列存储，属于行列式存储，压缩快，支持切分，支持快速列存取，但在读取所有列时，性能没有 SequenceFile 高 |
| Avro         | 原生 Hadoop 格式（Flume 就有使用），基于行存储，以 JSON 格式存储，使其易于被任何程序读取和解释，被广泛用作序列化平台，提供了丰富的数据结构，且数据本身也紧凑高效 |
| **ORCFile**  | RCFile 的升级版，性能上有大幅提升，属于行列式存储            |
| **Parquet**  | 是一种新型的、与语言无关的、并且不和任何一种数据处理框架绑定的、列式存储结构，可以适用于大数据圈的任何项目，属于行列式存储 |

使用建议：

![1662033465295](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1662033465295.png)

![1662033560127](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1662033560127.png)

1. 追求存储空间，则优先考虑使用 ORCFile 进行存储。
2. 同时由于 ORCFile 本身支持切分，所以还要追求压缩、解压速度，则优先考虑使用 Snappy。
3. 而如果追求的是数据兼容性，则考虑使用 Parquet，因为 Parquet 可以适用于大数据圈的任何项目。

##### 1、TextFile

```shell
# 验证压缩格式
# 创建TextFile普通表
create external table stu_textfile(
  id int,
  name string,
  city string
)row format delimited
fields terminated by ','
lines terminated by '\n'
location 'hdfs://bigdata01:9000/stu_textfile';

# 加载原始数据：占用2.09GB
hdfs dfs -put stu_textfile.dat /stu_textfile

# 创建TextFile+deflate压缩表
create external table stu_textfile_deflate_compress(
  id int,
  name string,
  city string
)row format delimited
fields terminated by ','
lines terminated by '\n'
location 'hdfs://bigdata01:9000/stu_textfile_deflate_compress';
# 指定TextFile+deflate压缩格式
# 1、最终输出是否开启压缩
set hive.exec.compress.output=true;
# 2、MapReduce阶段是否开启压缩
set mapreduce.output.fileoutputformat.compress=true;
# 3、以哪种压缩格式生成最终结果
set mapreduce.output.fileoutputformat.compress.codec=org.apache.hadoop.io.compress.DeflateCodec;
# 读取TextFile普通表，并导入数据到TextFile+deflate压缩表中：占用388.57mb
# 1、控制最终生成文件只有1个，以方便继续判断文件是否支持切分
set mapreduce.job.reduces=1;
# 2、group by控制产生Reduce阶段，不加则只有Map阶段，没有Reduce阶段
insert into stu_textfile_deflate_compress select id,name,city from stu_textfile group by id,name,city;
# 3、验证使用TextFile+deflate压缩是否可进行切分：只产生1个Map任务，说明不支持切分
set mapred.max.split.size=134217728;# 设置128mb进行切分，hive默认按244mb进行切分
select id,count(*) from stu_textfile_deflate_compress group by id;

# 创建TextFile+bzip2压缩表
create external table stu_textfile_bzip2_compress(
  id int,
  name string,
  city string
)row format delimited
fields terminated by ','
lines terminated by '\n'
location 'hdfs://bigdata01:9000/stu_textfile_bzip2_compress';
# 指定TextFile+bzip2压缩格式
set hive.exec.compress.output=true;
set mapreduce.output.fileoutputformat.compress=true;
set mapreduce.output.fileoutputformat.compress.codec=org.apache.hadoop.io.compress.BZip2Codec;
# 读取TextFile普通表，并导入数据到TextFile+bzip2压缩表中：占用189.98mb
set mapreduce.job.reduces=1;
insert into stu_textfile_bzip2_compress select id,name,city from stu_textfile group by id,name,city;
# 验证使用TextFile+bzip2压缩是否可进行切分：产生了2个Map任务，说明支持切分
set mapred.max.split.size=134217728;# 设置128mb进行切分，hive默认按244mb进行切分
select id,count(*) from stu_textfile_bzip2_compress group by id;
```

##### 2、SequenceFile

![1661953292237](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1661953292237.png)

![1661953310885](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1661953310885.png)

```shell
# 验证存储格式
# 创建SequenceFile普通表
create external table stu_seqfile_none_compress(
  id int,
  name string,
  city string
)row format delimited
fields terminated by ','
lines terminated by '\n'
stored as sequencefile
location 'hdfs://bigdata01:9000/stu_seqfile_none_compress';
# 读取TextFile普通表，并导入数据到SequenceFile普通表中：占用2.71GB（还变大了！）
# 生成的结果解析：文件名：，文件内容：1204492，zs204492，beijing204492
set mapreduce.job.reduces=1;
insert into stu_seqfile_none_compress select id,name,city from stu_textfile group by id,name,city;
# 验证使用SequenceFile+无压缩是否可进行切分：产生了2个Map任务，说明支持切分
select id,count(*) from stu_seqfile_none_compress group by id;

# 创建SequenceFile+deflate压缩表
create external table stu_seqfile_deflate_compress(
  id int,
  name string,
  city string
)row format delimited
fields terminated by ','
lines terminated by '\n'
stored as sequencefile
location 'hdfs://bigdata01:9000/stu_seqfile_deflate_compress';
# 指定SequenceFile+deflate压缩格式
set hive.exec.compress.output=true;
set mapreduce.output.fileoutputformat.compress=true;
set mapreduce.output.fileoutputformat.compress.codec=org.apache.hadoop.io.compress.DeflateCodec;
set io.seqfile.compression.type=BLOCK;
# 读取TextFile普通表，并导入数据到SequenceFile+deflate压缩表中：占用2.39GB
set mapreduce.job.reduces=1;
insert into stu_seqfile_deflate_compress select id,name,city from stu_textfile group by id,name,city;
# 验证使用SequenceFile+deflate压缩是否可进行切分：产生了10个Map任务，说明支持切分，由于deflate本身不支持切分，所以说明SequenceFile能否切分，与压缩格式无关，使用压缩格式只需要考虑压缩比就可以了，因为SequenceFile压缩只是对key-value的value进行压缩，并不会被压缩格式改变文件本身的特性
select id,count(*) from stu_seqfile_deflate_compress group by id;
```

##### 3、RCFile

![1661953595581](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1661953595581.png)

```shell
# 验证存储格式
# 创建rcfile普通表
create external table stu_rcfile_none_compress(
  id int,
  name string,
  city string
)row format delimited
fields terminated by ','
lines terminated by '\n'
stored as rcfile
location 'hdfs://bigdata01:9000/stu_rcfile_none_compress';
# 读取TextFile普通表，并导入数据到SequenceFile普通表中：占用1.65GB（变小了！）
set mapreduce.job.reduces=1;
insert into stu_rcfile_none_compress select id,name,city from stu_textfile group by id,name,city;
# 验证使用rcfile+无压缩是否可进行切分：产生了7个Map任务，说明支持切分
select id,count(*) from stu_rcfile_none_compress group by id;

# 创建rcfile+deflate压缩表
create external table stu_rcfile_deflate_compress(
  id int,
  name string,
  city string
)row format delimited
fields terminated by ','
lines terminated by '\n'
stored as rcfile
location 'hdfs://bigdata01:9000/stu_rcfile_deflate_compress';
# 指定rcfile+deflate压缩格式
set hive.exec.compress.output=true;
set mapreduce.output.fileoutputformat.compress=true;
set mapreduce.output.fileoutputformat.compress.codec=org.apache.hadoop.io.compress.DeflateCodec;
# 读取TextFile普通表，并导入数据到rcfile+deflate压缩表中：占用382.49mb
set mapreduce.job.reduces=1;
insert into stu_rcfile_deflate_compress select id,name,city from stu_textfile group by id,name,city;
# 验证使用rcfile+deflate压缩是否可进行切分：产生了2个Map任务，说明支持切分，同样rcfile能否切分，与压缩格式无关
select id,count(*) from stu_rcfile_deflate_compress group by id;
```

##### 4、ORCFile

![1662031516236](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1662031516236.png)

```shell
# 验证存储格式
# 创建orcfile普通表
create external table stu_orc_none_compress(
  id int,
  name string,
  city string
)row format delimited
fields terminated by ','
lines terminated by '\n'
stored as orc
location 'hdfs://bigdata01:9000/stu_orc_none_compress'
tblproperties("orc.compress"="NONE");
# 读取TextFile普通表，并导入数据到orcfile普通表中：占用1.38GB（变小了！）
set mapreduce.job.reduces=1;
insert into stu_orc_none_compress select id,name,city from stu_textfile group by id,name,city;
# 验证使用orcfile+无压缩是否可进行切分：产生了6个Map任务，说明支持切分
select id,count(*) from stu_orc_none_compress group by id;

# 创建orcfile+zlib(即deflate)压缩表
create external table stu_orc_zlib_compress(
  id int,
  name string,
  city string
)row format delimited
fields terminated by ','
lines terminated by '\n'
stored as orc
location 'hdfs://bigdata01:9000/stu_orc_zlib_compress'
tblproperties("orc.compress"="ZLIB");
# 读取TextFile普通表，并导入数据到orcfile+zlib(即deflate)压缩表中：占用269.1mb
set mapreduce.job.reduces=1;
insert into stu_orc_zlib_compress select id,name,city from stu_textfile group by id,name,city;
# 验证使用orcfile+zlib(即deflate)压缩是否可进行切分：产生了2个Map任务，说明支持切分，同样orcfile能否切分，与压缩格式无关
select id,count(*) from stu_orc_zlib_compress group by id;

# 创建orcfile+snappy压缩表
create external table stu_orc_snappy_compress(
  id int,
  name string,
  city string
)row format delimited
fields terminated by ','
lines terminated by '\n'
stored as orc
location 'hdfs://bigdata01:9000/stu_orc_snappy_compress'
tblproperties("orc.compress"="SNAPPY");
# 读取TextFile普通表，并导入数据到orcfile+snappy压缩表中：占用529.48mb
set mapreduce.job.reduces=1;
insert into stu_orc_snappy_compress select id,name,city from stu_textfile group by id,name,city;
# 验证使用orcfile+snappy压缩是否可进行切分：产生了3个Map任务，说明支持切分，同样orcfile能否切分，与压缩格式无关
select id,count(*) from stu_orc_snappy_compress group by id;

# 验证是否支持orcfile+BZIP2压缩
create external table stu_orc_bzip2_compress(
  id int,
  name string,
  city string
)row format delimited
fields terminated by ','
lines terminated by '\n'
stored as orc
location 'hdfs://bigdata01:9000/stu_orc_bzip2_compress'
tblproperties("orc.compress"="BZIP2");
# 读取TextFile普通表，并导入数据到orcfile+BZIP2压缩表中：发现报错了，说明orcfile不支持BZIP2
set mapreduce.job.reduces=1;
insert into stu_orc_bzip2_compress select id,name,city from stu_textfile group by id,name,city;
```

##### 5、Parquet

![1662032973843](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1662032973843.png)

```shell
# 验证存储格式
# 创建parquet普通表
create external table stu_parquet_none_compress(
  id int,
  name string,
  city string
)row format delimited
fields terminated by ','
lines terminated by '\n'
stored as parquet
location 'hdfs://bigdata01:9000/stu_parquet_none_compress'
tblproperties("parquet.compression"="uncompressed");
# 读取TextFile普通表，并导入数据到parquet普通表中：占用2.05GB（变小一点点）
set mapreduce.job.reduces=1;
insert into stu_parquet_none_compress select id,name,city from stu_textfile group by id,name,city;
# 验证使用parquet+无压缩是否可进行切分：产生了9个Map任务，说明支持切分，同样parquet能否切分，与压缩格式无关
select id,count(*) from stu_parquet_none_compress group by id;

# 创建parquet+gzip(即deflate)压缩表
create external table stu_parquet_gzip_compress(
  id int,
  name string,
  city string
)row format delimited
fields terminated by ','
lines terminated by '\n'
stored as parquet
location 'hdfs://bigdata01:9000/stu_parquet_gzip_compress'
tblproperties("parquet.compression"="gzip");
# 读取TextFile普通表，并导入数据到parquet+gzip(即deflate)压缩表中：占用358.61mb
set mapreduce.job.reduces=1;
insert into stu_parquet_gzip_compress select id,name,city from stu_textfile group by id,name,city;
# 验证使用parquet+gzip(即deflate)压缩表是否可进行切分：产生了2个Map任务，说明支持切分，同样parquet能否切分，与压缩格式无关
select id,count(*) from stu_parquet_gzip_compress group by id;


# 创建parquet+snappy压缩表
create external table stu_parquet_snappy_compress(
  id int,
  name string,
  city string
)row format delimited
fields terminated by ','
lines terminated by '\n'
stored as parquet
location 'hdfs://bigdata01:9000/stu_parquet_snappy_compress'
tblproperties("parquet.compression"="snappy");
# 读取TextFile普通表，并导入数据到parquet+snappy压缩表中：占用804.03mb
set mapreduce.job.reduces=1;
insert into stu_parquet_snappy_compress select id,name,city from stu_textfile group by id,name,city;
# 验证使用parquet+snappy压缩表是否可进行切分：产生了4个Map任务，说明支持切分，同样parquet能否切分，与压缩格式无关
select id,count(*) from stu_parquet_snappy_compress group by id;
```





