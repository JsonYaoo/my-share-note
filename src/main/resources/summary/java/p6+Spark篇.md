# **二十一、Spark 篇**

### 1.1. 什么是 Spark？

Spark，是一个基于内存的统一计算引擎，既可以做批处理（离线数据计算），也可以做流处理（实时数据计算），还可以实现 SQL 计算，其速度可以达到 MapReduce 的几十倍，甚至上百倍。

### 1.2. Spark 的特点？

1. Speed：基于内存计算，速度快。
2. Ease of Use：易用，内置函数多，使用方便，可以使用 Java、Scala、Python、R、SQL 编写应用程序。
3. Generality：通用，可以一站式处理地完成，批处理、流处理、SQL 处理、机器学习、图计算多个领域任务。
4. Runs Everywhere：不仅可以单独运行，还可以在很多框架运行，比如 Spark On Yarn、Mesos、K8S 等，以及可以读取很多其他数据源的数据，比如 HDFS、HBase、Hive、Cassandra 等。

![1662692042045](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1662692042045.png)

### 1.3. Spark 应用场景？

1. 低延时的，海量数据计算需求。
2. 低延时的，SQL交互查询需求。

### 1.4. Spark vs Hadoop？

|          | Spark                          | Hadoop                                             |
| -------- | ------------------------------ | -------------------------------------------------- |
| 定位     | 计算引擎                       | HDFS 分布式存储、Yarn 资源管理、MapReduce 计算引擎 |
| 计算模型 | 可以包含多种计算操作，非常灵活 | MapReduce 任务只能是 Map 和 Reduce 操作，不够灵活  |
| 处理速度 | 基于内存，速度快               | 基于磁盘，速度较慢                                 |

### 1.5. Spark vs MapReduce？

Spark 通过借鉴 MapReduce 发展而来，继承了其分布式并行计算的优点，并改进了 MapReduce 明显的缺陷，具体表现在以下几方面：

1. Spark 效率高：Spark 把中间计算结果存放在内存中，减少了迭代过程中的数据落地，能够实现数据高效共享，迭代运算效率高。而 Mapreduce 中的计算中间结果保存在磁盘上，影响了整体的运行速度。
2. Spark 容错性高：Spark 支持 DAG 有向无环图的分布式并行计算，通过引进 RDD 弹性分布式数据集的概念（分布在一组节点中的只读对象集合），如果数据集一部分数据丢失，可以根据血统来对进行重建，在 RDD 计算时，还可以通过 checkpoint 来实现容错（checkpoint 有两种方式，分别是 checkpiont data 和 logging the updates）。
   - 1）DAG：有向无环图，用于描述任务间的先后依赖关系，Spark 中 RDD 经过若干次 transform 操作，由于 transform 操作是 lazy 的，因此，当 RDD 进行 action 操作时，RDD 间的转换关系会被提交上去，得到 RDD 内部的依赖关系，进而根据依赖，划分出不同的 stage。
3. Spark 更加通用：MapReduce 只提供了 Map 和 Reduce 两种操作，Spark 则提供了 创建、转换、控制、行动 4 类 RDD 操作。

### 1.6. Spark 架构原理？

#### 1）节点角色

1. Driver：Driver 进程负责执行 Spark 程序，是 Spark 集群中的某一个节点，或者是提交 Spark 程序的客户端节点，具体是哪个由提交任务时的参数决定。
2. Master：集群主节点进程，负责集群资源的管理、分配、以及集群监控。
3. Worker：集群从节点进程，负责启动 Executor 进程来执行任务。
4. Executor：由 Worker 启动的进程，负责执行任务。
5. Task：由 Executor 启动的线程，是任务真正的执行者，负责执行实际数据的处理和计算。

#### 2）执行流程

1. Spark 代码由 Driver 进程负责执行，用 spark-submit 脚本提交任务时，Driver 进程就启动了。
2. Driver 启动之后，会做一些初始化操作，找到主节点的 Master 进程，注册 Spark 程序。
3. Master 收到 Spark 程序的注册申请后，会发送请求给从节点的 Worker 进程，进行资源的调度和分配。
4. Worker 收到请求之后，会给 Spark 程序启动 Executor 进程。
5. Executor 启动完成后，会向 Driver 进程进行反注册，以让 Driver 知道有哪些 Executor 在为他服务。
6. Driver 根据程序中对 RDD 的函数操作，提交对应的 Task ( 比如 `map，flatmap` 等）到 Executor 上进行计算。

![1663343591964](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1663343591964.png)

#### 3）工作原理

1. Spark 提交任务。
2. 读取数据源 HDFS 的数据，把数据加载到内存中，转换为多个 RDD（弹性分布式数据集）。
3. 多个 RDD 分布在不同的节点，每个节点并行处理，与 Mapreduce 不同的是，RDD 数据存储在内存中。
4. 调用函数，每个节点分别对自己的 RDD 数据，进行相应的函数处理。
5. 经过多次处理后，最终把结果保存在 HDFS 上，完成一次 Spark 任务的计算。

![1663342961333](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1663342961333.png)

### 1.7. Spark 安装部署？

#### 1）Spark Standlone 模式

需要额外独立部署、维护一个 Spark 集群，运维麻烦。

一主两从、JDK 环境、无需 Hadoop 环境

##### 1、上传、解压

```shell
tar -zxvf spark-2.4.3-bin-hadoop2.7.tgz
```

##### 2、修改 spark-env.sh

```shell
[root@bigdata01 spark-2.4.3-bin-hadoop2.7]# pwd
/data/soft/spark-2.4.3-bin-hadoop2.7

[root@bigdata01 conf]# mv spark-env.sh.template spark-env.sh
[root@bigdata01 conf]# vim spark-env.sh
# 行末添加下面配置
export JAVA_HOME=/data/soft/jdk1.8
export SPARk_MASTER_HOST=bigdata01
```

##### 3、修改 slaves

```shell
# 行末添加从节点主机名
bigdata02
bigdata03
```

##### 4、拷贝到集群其他节点

```shell
[root@bigdata01 soft]# scp -rq spark-2.4.3-bin-hadoop2.7 bigdata02:/data/soft/
[root@bigdata01 soft]# scp -rq spark-2.4.3-bin-hadoop2.7 bigdata03:/data/soft/
```

##### 5、启动集群

```shell
[root@bigdata01 sbin]# pwd
/data/soft/spark-2.4.3-bin-hadoop2.7/sbin
[root@bigdata01 sbin]# start-all.sh 
...
```

##### 6、验证集群

http://bigdata01:8080/

```she
[root@bigdata01 sbin]# jps
3161 Master
3241 Jps

[root@bigdata02 soft]# jps
3200 Jps
3149 Worker

[root@bigdata03 soft]# jps
3170 Worker
3224 Jps
```

##### 7、提交任务到集群

```shell
[root@bigdata01 bin]# pwd
/data/soft/spark-2.4.3-bin-hadoop2.7/bin

# 指定main class、master、jar包地址、执行次数
[root@bigdata01 bin]# spark-submit --class org.apache.spark.examples.SparkPi --master spark://bigdata01:7077 ../examples/jars/spark-examples_2.11-2.4.3.jar 2
```

##### 8、停止集群

```shell
[root@bigdata01 sbin]# pwd
/data/soft/spark-2.4.3-bin-hadoop2.7/sbin

[root@bigdata01 sbin]# stop-all.sh 
bigdata03: stopping org.apache.spark.deploy.worker.Worker
bigdata02: stopping org.apache.spark.deploy.worker.Worker
stopping org.apache.spark.deploy.master.Master
```

#### 2）Spark On Yarn 模式

共用 Hadoop 集群资源，推荐使用，可以减少 Spark 集群的运维成本。

JDK 环境、需要 Hadoop 环境、Spark 作为 Hadoop 的一个客户端，提交到 Hadoop 中运行即可，无需启动任何 Spark 进程

##### 1、上传、解压

```shell
tar -zxvf spark-2.4.3-bin-hadoop2.7.tgz 
```

##### 2、修改 spark-env.sh

```shell
[root@bigdata04 spark-2.4.3-bin-hadoop2.7]# pwd
/data/soft/spark-2.4.3-bin-hadoop2.7

[root@bigdata01 conf]# mv spark-env.sh.template spark-env.sh
[root@bigdata01 conf]# vim spark-env.sh
# 行末添加下面配置
export JAVA_HOME=/data/soft/jdk1.8
export HADOOP_CONF_DIR=/data/soft/hadoop-3.2.0/etc/hadoop
```

##### 3、提交 Spark 任务到 Hadoop 集群

```shell
# 指定main class、master+yarn集群模式、jar包地址、执行次数
spark-submit --class org.apache.spark.examples.SparkPi --master yarn --deploy-mode cluster ../examples/jars/spark-examples_2.11-2.4.3.jar 2
```

### 1.8. Spark 执行模式？

#### 1）本地执行

本地无需依赖其他环境时可用

##### 1、Scala

```scala
package com.bigdata.scala

import org.apache.spark.{SparkConf, SparkContext}

/**
  * 单词计数 Scala版
  *
  * @author yaocs2
  * @since 2022-09-17
  */
object WordCountLocalScala {

  def main(args: Array[String]): Unit = {
    // 创建sparkContext
    val conf = new SparkConf()
    conf.setAppName("WordCountLocalScala")
      .setMaster("local")
    val sparkContext = new SparkContext(conf)

    // 加载数据
    /**
      * hello you
      * hello me
      */
    val linesRDD = sparkContext.textFile("D:\\hello.txt")

    // 切分单词
    /**
      * hello
      * you
      * hello
      * me
      */
    val wordsRDD = linesRDD.flatMap(_.split(" "))

    // 转换为(word, 1)
    /**
      * (hello,1)
      * (you,1)
      * (hello,1)
      * (me,1)
      */
    val pairRDD = wordsRDD.map((_, 1))

    // 根据key分组统计
    /**
      * (hello,2)
      * (you,1)
      * (me,1)
      */
    val wordCountRDD = pairRDD.reduceByKey(_ + _)

    // 打印
    wordCountRDD.foreach(wordCount => println(wordCount._1 + ": " + wordCount._2))

    // 停止sparkContext
    sparkContext.stop()
  }
}
```

##### 2、Java

```java
package com.bigdata.java;

import org.apache.spark.SparkConf;
import org.apache.spark.api.java.JavaPairRDD;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.JavaSparkContext;
import org.apache.spark.api.java.function.FlatMapFunction;
import org.apache.spark.api.java.function.Function2;
import org.apache.spark.api.java.function.PairFunction;
import org.apache.spark.api.java.function.VoidFunction;
import scala.Tuple2;

import java.util.Arrays;
import java.util.Iterator;

/**
 * 单词计数 Java版
 *
 * @author yaocs2
 * @since 2022-09-17
 */
public class WordCountLocalJava {

    public static void main(String[] args) {
        // 创建sparkContext
        SparkConf conf = new SparkConf();
        conf.setAppName("WordCountLocalJava")
                .setMaster("local");
        JavaSparkContext javaSparkContext = new JavaSparkContext(conf);

        // 加载数据
        /**
         * hello you
         * hello me
         */
        JavaRDD<String> linesRDD = javaSparkContext.textFile("D:\\hello.txt");

        // 切分单词
        /**
         * hello
         * you
         * hello
         * me
         */
        JavaRDD<String> wordsRDD = linesRDD.flatMap(new FlatMapFunction<String, String>() {
            public Iterator<String> call(String line) throws Exception {
                return Arrays.asList(line.split(" ")).iterator();
            }
        });

        // 转换为(word, 1)
        /**
         * (hello,1)
         * (you,1)
         * (hello,1)
         * (me,1)
         */
        JavaPairRDD<String, Integer> pairRDD = wordsRDD.mapToPair(new PairFunction<String, String, Integer>() {
            public Tuple2<String, Integer> call(String word) throws Exception {
                return new Tuple2<String, Integer>(word, 1);
            }
        });

        // 根据key分组统计
        /**
         * (hello,2)
         * (you,1)
         * (me,1)
         */
        JavaPairRDD<String, Integer> wordCountRDD = pairRDD.reduceByKey(new Function2<Integer, Integer, Integer>() {
            public Integer call(Integer i1, Integer i2) throws Exception {
                return i1 + i2;
            }
        });

        // 打印
        wordCountRDD.foreach(new VoidFunction<Tuple2<String, Integer>>() {
            public void call(Tuple2<String, Integer> tup) throws Exception {
                System.out.println(tup._1 + ": " + tup._2);
            }
        });

        // 停止sparkContext
        javaSparkContext.stop();
    }
}
```

#### 2）提交执行 | 常用

##### 1、Scala

```scala
package com.bigdata.scala

import org.apache.spark.{SparkConf, SparkContext}

/**
  * 单词计数 Scala版
  *
  * @author yaocs2
  * @since 2022-09-17
  */
object WordCountSubmitScala {

  def main(args: Array[String]): Unit = {
    // 创建sparkContext
    val conf = new SparkConf()
    conf.setAppName("WordCountSubmitScala")
    val sparkContext = new SparkContext(conf)

    // 加载数据
    /**
      * hello you
      * hello me
      */
    var path = "D:\\hello.txt"
    if (args.length == 1) {
      path = args(0)
    }
    val linesRDD = sparkContext.textFile(path)

    // 切分单词
    /**
      * hello
      * you
      * hello
      * me
      */
    val wordsRDD = linesRDD.flatMap(_.split(" "))

    // 转换为(word, 1)
    /**
      * (hello,1)
      * (you,1)
      * (hello,1)
      * (me,1)
      */
    val pairRDD = wordsRDD.map((_, 1))

    // 根据key分组统计
    /**
      * (hello,2)
      * (you,1)
      * (me,1)
      */
    val wordCountRDD = pairRDD.reduceByKey(_ + _)

    // 打印
    wordCountRDD.foreach(wordCount => println(wordCount._1 + ": " + wordCount._2))

    // 停止sparkContext
    sparkContext.stop()
  }
}
```

##### 2、Java

```java
package com.bigdata.java;

import org.apache.spark.SparkConf;
import org.apache.spark.api.java.JavaPairRDD;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.JavaSparkContext;
import org.apache.spark.api.java.function.FlatMapFunction;
import org.apache.spark.api.java.function.Function2;
import org.apache.spark.api.java.function.PairFunction;
import org.apache.spark.api.java.function.VoidFunction;
import scala.Tuple2;

import java.util.Arrays;
import java.util.Iterator;

/**
 * 单词计数 Java版
 *
 * @author yaocs2
 * @since 2022-09-17
 */
public class WordCountSubmitJava {

    public static void main(String[] args) {
        // 创建sparkContext
        SparkConf conf = new SparkConf();
        conf.setAppName("WordCountSubmitJava");
        JavaSparkContext javaSparkContext = new JavaSparkContext(conf);

        // 加载数据
        /**
         * hello you
         * hello me
         */
        String path = "D:\\hello.txt";
        if (args.length == 1) {
            path = args[0];
        }
        JavaRDD<String> linesRDD = javaSparkContext.textFile(path);

        // 切分单词
        /**
         * hello
         * you
         * hello
         * me
         */
        JavaRDD<String> wordsRDD = linesRDD.flatMap(new FlatMapFunction<String, String>() {
            public Iterator<String> call(String line) throws Exception {
                return Arrays.asList(line.split(" ")).iterator();
            }
        });

        // 转换为(word, 1)
        /**
         * (hello,1)
         * (you,1)
         * (hello,1)
         * (me,1)
         */
        JavaPairRDD<String, Integer> pairRDD = wordsRDD.mapToPair(new PairFunction<String, String, Integer>() {
            public Tuple2<String, Integer> call(String word) throws Exception {
                return new Tuple2<String, Integer>(word, 1);
            }
        });

        // 根据key分组统计
        /**
         * (hello,2)
         * (you,1)
         * (me,1)
         */
        JavaPairRDD<String, Integer> wordCountRDD = pairRDD.reduceByKey(new Function2<Integer, Integer, Integer>() {
            public Integer call(Integer i1, Integer i2) throws Exception {
                return i1 + i2;
            }
        });

        // 打印
        wordCountRDD.foreach(new VoidFunction<Tuple2<String, Integer>>() {
            public void call(Tuple2<String, Integer> tup) throws Exception {
                System.out.println(tup._1 + ": " + tup._2);
            }
        });

        // 停止sparkContext
        javaSparkContext.stop();
    }
}
```

##### 3、提交任务到集群

```shell
spark-submit \
--class com.bigdata.scala.WordCountSubmitScala \
--master yarn \
--deploy-mode client \
--executor-memory 1G \
--num-executors 1 \
bigdata-spark-1.0-SNAPSHOT-jar-with-dependencies.jar \
hdfs://bigdata01:9000/hello.txt
```

#### 3）spark-shell 调试执行

进入本地启动的一个集群，但可读取 hdfs 以及本地的数据，常用于集群调试

##### 1、指定 Lzo 依赖

```shell
[root@bigdata04 conf]# pwd
/data/soft/spark-2.4.3-bin-hadoop2.7/conf

[root@bigdata04 conf]# vim spark-defaults.conf
# 末尾添加下面的配置
spark.jars=/data/soft/hadoop-3.2.0/share/hadoop/common/hadoop-lzo-0.4.21-SNAPSHOT.jar
```

##### 2、spark-shell local

```shell
[root@bigdata04 conf]# spark-shell 
2022-09-17 17:56:09,529 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Setting default log level to "WARN".
To adjust logging level use sc.setLogLevel(newLevel). For SparkR, use setLogLevel(newLevel).
Spark context Web UI available at http://bigdata04:4040
Spark context available as 'sc' (master = local[*], app id = local-1663408576166).
Spark session available as 'spark'.
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /___/ .__/\_,_/_/ /_/\_\   version 2.4.3
      /_/
         
Using Scala version 2.11.12 (Java HotSpot(TM) 64-Bit Server VM, Java 1.8.0_202)
Type in expressions to have them evaluated.
Type :help for more information.

# 读取hdfs://bigdata01:9000/hello.txt中的内容
scala> val linesRDD = sc.textFile("/hello.txt")
linesRDD: org.apache.spark.rdd.RDD[String] = /hello.txt MapPartitionsRDD[1] at textFile at <console>:24

scala> val wordsRDD = linesRDD.flatMap(_.split(" "))
wordsRDD: org.apache.spark.rdd.RDD[String] = MapPartitionsRDD[2] at flatMap at <console>:25

scala> val pairRDD = wordsRDD.map((_, 1))
pairRDD: org.apache.spark.rdd.RDD[(String, Int)] = MapPartitionsRDD[3] at map at <console>:25

scala> pairRDD.foreach(wordCount => println(wordCount._1 + ": " + wordCount._2))
hello: 1
you: 1
hello: 1
me: 1
```

##### 3、spark-shell on yarn

```shell
[root@bigdata04 sparkjars]# spark-shell --master yarn --deploy-mode client
2022-09-17 18:02:32,681 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Setting default log level to "WARN".
To adjust logging level use sc.setLogLevel(newLevel). For SparkR, use setLogLevel(newLevel).
2022-09-17 18:02:40,757 WARN yarn.Client: Neither spark.yarn.jars nor spark.yarn.archive is set, falling back to uploading libraries under SPARK_HOME.
Spark context Web UI available at http://bigdata04:4040
Spark context available as 'sc' (master = yarn, app id = application_1663401953433_0003).
Spark session available as 'spark'.
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /___/ .__/\_,_/_/ /_/\_\   version 2.4.3
      /_/
         
Using Scala version 2.11.12 (Java HotSpot(TM) 64-Bit Server VM, Java 1.8.0_202)
Type in expressions to have them evaluated.
Type :help for more information.

# 读取hdfs://bigdata01:9000/hello.txt中的内容
scala> val linesRDD = sc.textFile("/hello.txt")
linesRDD: org.apache.spark.rdd.RDD[String] = /hello.txt MapPartitionsRDD[1] at textFile at <console>:24

scala> val wordsRDD = linesRDD.flatMap(_.split(" "))
wordsRDD: org.apache.spark.rdd.RDD[String] = MapPartitionsRDD[2] at flatMap at <console>:25

scala> val pairRDD = wordsRDD.map((_, 1))
pairRDD: org.apache.spark.rdd.RDD[(String, Int)] = MapPartitionsRDD[3] at map at <console>:25

# 打印到Yarn控制台，而不是当前spark-shell窗口
scala> pairRDD.foreach(wordCount => println(wordCount._1 + ": " + wordCount._2))
```

##### 4、启动 spark history server

```shell 
[root@bigdata04 conf]# pwd
/data/soft/spark-2.4.3-bin-hadoop2.7/conf
[root@bigdata04 conf]# vim spark-defaults.conf

[root@bigdata04 conf]# vim spark-env.sh 
# 末尾添加下面的配置
spark.eventLog.enabled=true
spark.eventLog.compress=true
spark.eventLog.dir=hdfs://bigdata01:9000/tmp/logs/root/logs
spark.history.fs.logDirectory=hdfs://bigdata01:9000/tmp/logs/root/logs
spark.yarn.historyServer.address=http://bigdata04:18080

[root@bigdata04 conf]# vim spark-env.sh
# 末尾添加下面的配置
export SPARK_HISTORY_OPTS="-Dspark.history.ui.port=18080 -Dspark.history.fs.logDirectory=hdfs://bigdata01:9000/tmp/logs/root/logs"

# 启动 spark history server
[root@bigdata04 sbin]# pwd
/data/soft/spark-2.4.3-bin-hadoop2.7/sbin
[root@bigdata04 sbin]# start-history-server.sh 
starting org.apache.spark.deploy.history.HistoryServer, logging to /data/soft/spark-2.4.3-bin-hadoop2.7/logs/spark-root-org.apache.spark.deploy.history.HistoryServer-1-bigdata04.out
[root@bigdata04 sbin]# jps
3688 Jps
3647 HistoryServer

# 重启hadoop、去yarn管理界面查看
```

### 1.9. 什么是 RDD？

RDD，Resillient Distributed DataSets，弹性分布式数据集，是 Spark 抽象出的一个分布式内存概念，是一个只读的对象数据集，是 Spark 编程的核心，可通过文件（包括本地的、HDFS 的）创建，也可以在程序中使用集合来生成。

1. 弹性：RDD 数据，默认情况存放在内存中，在内存资源不足时，Spark 还会自动将 RDD 数据写入到磁盘中。
2. 分布式：RDD 可以被分区，每个分区分布在集群不同节点上，从而让 RDD 中的数据可以被并行操作。
3. 容错：当一个节点失败或者宕机，Spark 可以自动根据分区规则，重新计算当前分区的 RDD 数据。

Spark 提供了 创建、转换、控制、行动 4 类 RDD 操作：

1. 创建操作：用于创建 RDD，有两种方式，一种是通过集合创建，一种是通过文件创建（本地 / HDFS 文件）。
2. 转换操作：将 RDD 通过一定的操作转换成新的 RDD，转换操作是 lazy 的，只是定义了一个新的 RDD，并没有立即执行，比如：`map、filter、flatmap、sample、groupbykey、reducebykey、union、join、cogroup、mapvalues、sort、partitionby` 等。
3. 控制操作：进行 RDD 持久化，可以将 RDD 按不同的存储策略，缓存在磁盘或内存中，比如 cache 接口实现，默认就是将 RDD 缓存在内存中。
4. 行动操作：能够触发 Spark 运行的操作，分为两类，一类的操作结果是变成 Scala 集合或变量，另一类是将 RDD 保存到外部文件系统或数据库中，比如：`collect、reduce、lookup、save` 等。

#### 1）创建操作

##### 1、通过集合创建

一般用于自己 mock 数据进行测试

###### ① Scala

```scala
package com.bigdata.scala

import org.apache.spark.{SparkConf, SparkContext}

/**
  * 通过数据集创建RDD
  *
  * @author yaocs2
  * @since 2022-09-17
  */
object CreateRddByArrayScala {

  def main(args: Array[String]): Unit = {
    // 创建sparkContext
    val conf = new SparkConf()
    conf.setAppName("CreateRddByArrayScala")
      .setMaster("local")
    val sc = new SparkContext(conf)

    // 创建集合：非spark代码, 在Driver进程上执行
    val arr = Array(1, 2, 3, 4, 5)

    // 指定3个分区, 默认为2：spark代码, 在Worker进程上执行
    // Worker会分散地发送数据给指定的分区, 形成分布式数据集RDD
    val rdd = sc.parallelize(arr, 3)

    // 对集合数据求和：spark代码, 在Worker进程上执行
    val sum = rdd.reduce(_ + _)

    // 打印结果15：非spark代码, 在Driver进程上执行
    println(sum)

    // 停止sparkContext
    sc.stop()
  }
}
```

###### ② Java

```java
package com.bigdata.java;

import org.apache.spark.SparkConf;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.JavaSparkContext;
import org.apache.spark.api.java.function.Function2;

import java.util.Arrays;
import java.util.List;

/**
 * 通过数据集创建RDD
 *
 * @author yaocs2
 * @since 2022-09-17
 */
public class CreateRddByArrayJava {

    public static void main(String[] args) {
        // 创建sparkContext
        SparkConf conf = new SparkConf();
        conf.setAppName("CreateRddByArrayJava")
                .setMaster("local");
        JavaSparkContext sc = new JavaSparkContext(conf);

        // 创建集合：非spark代码, 在Driver进程上执行
        List<Integer> arr = Arrays.asList(1, 2, 3, 4, 5);

        // 指定3个分区, 默认为2：spark代码, 在Worker进程上执行
        // Worker会分散地发送数据给指定的分区, 形成分布式数据集RDD
        JavaRDD<Integer> rdd = sc.parallelize(arr);

        // 对集合数据求和：spark代码, 在Worker进程上执行
        Integer sum = rdd.reduce(new Function2<Integer, Integer, Integer>() {
            @Override
            public Integer call(Integer i1, Integer i2) throws Exception {
                return i1 + i2;
            }
        });

        // 打印结果15：非spark代码, 在Driver进程上执行
        System.out.println(sum);

        // 停止sparkContext
        sc.stop();
    }
}
```

##### 2、通过文件创建

1. 通过 SparkContext `textFile()` 方法，可以使用本地 / HDFS 文件来创建 RDD，此时 RDD 的每个元素，其实就是文件中每一行数据。
2. `textFile()` 方法支持针对目录、压缩文件以及通配符来创建 RDD。
3. Spark 默认会为 HDFS 文件中，的每一个 block 创建一个分区，也可以通过 `textFile()` 方法的第二个参数来指定，但注意，此时设置的分区数量，只能比 block 数量多，不能比 block 数量少，否则不会生效。

###### ① Scala

```scala
package com.bigdata.scala

import org.apache.spark.{SparkConf, SparkContext}

/**
  * 通过文件创建RDD
  *
  * @author yaocs2
  * @since 2022-09-17
  */
object CreateRddByFileScala {

  def main(args: Array[String]): Unit = {
    // 创建sparkContext
    val conf = new SparkConf()
    conf.setAppName("CreateRddByFileScala")
      .setMaster("local")
    val sc = new SparkContext(conf)

    // 读取文件
//    var path = "D:\\hello.txt"// 本地文件
    var path = "hdfs://bigdata01:9000/hello.txt"// HDFS文件
    val rdd = sc.textFile(path);

    // 获取每一行数据的长度
    val rowLenRdd = rdd.map(_.length)

    // 统计文件所有行数据的总长度
    val sumLen = rowLenRdd.reduce(_ + _)

    // 打印结果
    println(sumLen)

    // 停止sparkContext
    sc.stop()
  }
}
```

###### ② Java

```java
package com.bigdata.java;

import org.apache.spark.SparkConf;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.JavaSparkContext;
import org.apache.spark.api.java.function.Function;
import org.apache.spark.api.java.function.Function2;

/**
 * 通过文件创建RDD
 *
 * @author yaocs2
 * @since 2022-09-17
 */
public class CreateRddByFileJava {

    public static void main(String[] args) {
        // 创建sparkContext
        SparkConf conf = new SparkConf();
        conf.setAppName("CreateRddByFileJava")
                .setMaster("local");
        JavaSparkContext sc = new JavaSparkContext(conf);

        // 读取文件
        String path = "D:\\hello.txt";// 本地文件
//        String path = "hdfs://bigdata01:9000/hello.txt";// HDFS文件
        JavaRDD<String> rdd = sc.textFile(path);

        // 获取每一行数据的长度
        // JavaRDD<Integer> rowLenRdd = rdd.map((Function<String, Integer>) String::length);
        JavaRDD<Integer> rowLenRdd = rdd.map(new Function<String, Integer>() {
            @Override
            public Integer call(String line) throws Exception {
                return line.length();
            }
        });

        // 统计文件所有行数据的总长度
        // Integer sumLen = rowLenRdd.reduce((Function2<Integer, Integer, Integer>) (i1, i2) -> i1 + i2);
        Integer sumLen = rowLenRdd.reduce(new Function2<Integer, Integer, Integer>() {
            @Override
            public Integer call(Integer i1, Integer i2) throws Exception {
                return i1 + i2;
            }
        });

        // 打印结果
        System.out.println(sumLen);

        // 停止sparkContext
        sc.stop();
    }
}
```

#### 2）转换操作 | Transformation 算子

1. Transformation 和 Action 操作，称之为算子。
2. Transformation 特性：lazy，执行 Transformation 操作，并不会立马执行，而是等到出现 Action 算子时，才会真正地执行，以优化底层的执行效率，避免产生过多的中间结果。
3. Action 特性：执行 Action 操作时，才会触发一个 Spark 任务的真正执行，从而触发这个 Action 之前所有的Transformation 操作的执行。

```scala
package com.bigdata.scala

import org.apache.spark.{SparkConf, SparkContext}

/**
  * 单词计数 Scala版
  *
  * @author yaocs2
  * @since 2022-09-17
  */
object WordCountLocalScala {

  def main(args: Array[String]): Unit = {
    // 创建sparkContext
    val conf = new SparkConf()
    conf.setAppName("WordCountLocalScala")
      .setMaster("local")
    val sparkContext = new SparkContext(conf)

    // 加载数据：转换操作, 数据不会被立马加载到内存中, 只是一个逻辑上的RDD
    /**
      * hello you
      * hello me
      */
    val linesRDD = sparkContext.textFile("D:\\hello.txt")

    // 切分单词：转换操作, 数据不会被立马加载到内存中, 只是一个逻辑上的RDD
    /**
      * hello
      * you
      * hello
      * me
      */
    val wordsRDD = linesRDD.flatMap(_.split(" "))

    // 转换为(word, 1)：转换操作, 数据不会被立马加载到内存中, 只是一个逻辑上的RDD
    /**
      * (hello,1)
      * (you,1)
      * (hello,1)
      * (me,1)
      */
    val pairRDD = wordsRDD.map((_, 1))

    // 根据key分组统计：转换操作, 数据不会被立马加载到内存中, 只是一个逻辑上的RDD
    /**
      * (hello,2)
      * (you,1)
      * (me,1)
      */
    val wordCountRDD = pairRDD.reduceByKey(_ + _)

    // 打印：行动操作, 会触发前面所有的转换操作一起执行
    // 只有到任务执行到这里时, 才会真正地执行计算, 在这里如果没有这行代码, 前面的转换操作是不会执行的
    wordCountRDD.foreach(wordCount => println(wordCount._1 + ": " + wordCount._2))

    // 停止sparkContext
    sparkContext.stop()
  }
}
```

| 算子        | 作用                                                        |
| ----------- | ----------------------------------------------------------- |
| map         | 对 RDD 中每个元素进行处理，一进一出                         |
| flatmap     | 与 map 类似，但每个元素可以返回一个、或者多个元素，一进多出 |
| filter      | 对 RDD 中每个元素进行判断，返回 true 则代表保留，否则会过滤 |
| groupByKey  | 根据 key 进行分组，每个 key 对应一个 `Iterable<value>`      |
| reduceByKey | 对每个相同 key 对应的 value 进行 reduce 操作                |
| sortByKey   | 对每个相同 key 对应的 value 进行排序操作，属于全局排序      |
| join        | 对两个包含 `<key,value>` 的 RDD 进行 join 操作              |
| distinct    | 对 RDD 元素进行全局去重                                     |

##### ① Scala

```scala
package com.bigdata.scala

import org.apache.spark.{SparkConf, SparkContext}

/**
  * Transformation 算子
  *
  * @author yaocs2
  * @since 2022-09-17
  */
object TransformationOpScala {

  def main(args: Array[String]): Unit = {
    // 创建sparkContext
    val sc = getSparkContext("TransformationOpScala")

    // map：对集合中每一个元素*2
    //    mapOp(sc)

    // filter：过滤出集合中的偶数
    //    filterOp(sc)

    // flatMap：将行拆分为单词
    //    flatMapOp(sc)

    // groupByKey：对每个大区的主播进行分组
    //    groupByKeyOp(sc)
    //    groupByKeyOp2(sc)

    // reduceByKey：统计每个大区的主播数量
    //    reduceByKeyOp(sc)

    // sortByKey：对主播的音浪收入排序
    //    sortByKeyOp(sc)
    //    sortByOp(sc)

    // join：打印每个主播大区信息和音浪收入
    //    joinOp(sc)

    // distinct：统计当天开播的大区信息
    //    distinctOp(sc)

    // 停止sparkContext
    sc.stop()
  }

  /**
    * distinct：统计当天开播的大区信息
    * CN
    * US
    * IN
    *
    * @param sc
    */
  private def distinctOp(sc: SparkContext): Unit = {
    // 主播ID、主播所在大区
    val dataRDD = sc.parallelize(Array((150001, "US"), (150002, "CN"), (150003, "CN"), (150004, "IN")))
    dataRDD.map(_._2).distinct().foreach(println(_))
  }

  /**
    * join：打印每个主播大区信息和音浪收入
    * (150001,(US,400))
    * (150002,(CN,200))
    * (150003,(CN,300))
    * (150004,(IN,100))
    *
    * @param sc
    */
  private def joinOp(sc: SparkContext): Unit = {
    // 主播ID、主播所在大区
    val dataRDD1 = sc.parallelize(Array((150001, "US"), (150002, "CN"), (150003, "CN"), (150004, "IN")))

    // 主播ID、主播的音浪收入
    val dataRDD2 = sc.parallelize(Array((150001, 400), (150002, 200), (150003, 300), (150004, 100)))

    // 根据key进行join操作
    dataRDD1.join(dataRDD2).foreach(println(_))
  }

  /**
    * sortBy：对主播的音浪收入排序(指定某一列进行排序)
    * (150004,100)
    * (150002,200)
    * (150003,300)
    * (150001,400)
    *
    * @param sc
    */
  private def sortByOp(sc: SparkContext): Unit = {
    // 主播ID、主播的音浪收入
    val dataRDD = sc.parallelize(Array((150001, 400), (150002, 200), (150003, 300), (150004, 100)))

    // sortBy时, 可以直接指定音浪收入作为key, 无需在第一位, 默认为true, 代表正序, 可以指定为false, 代表逆序
    dataRDD.sortBy(_._2).foreach(println(_))
  }

  /**
    * sortByKey：对主播的音浪收入排序
    * (100,150004)
    * (200,150002)
    * (300,150003)
    * (400,150001)
    *
    * @param sc
    */
  private def sortByKeyOp(sc: SparkContext): Unit = {
    // 主播ID、主播的音浪收入
    val dataRDD = sc.parallelize(Array((150001, 400), (150002, 200), (150003, 300), (150004, 100)))

    // sortByKey时, 音浪收入作为key, 必须在第一位, 默认为true, 代表正序, 可以指定为false, 代表逆序
    dataRDD.map(tup => (tup._2, tup._1)).sortByKey().foreach(println(_))
  }

  /**
    * reduceByKey：统计每个大区的主播数量
    * (CN,2)
    * (US,1)
    * (IN,1)
    *
    * @param sc
    */
  private def reduceByKeyOp(sc: SparkContext): Unit = {
    // 主播ID、主播所在大区
    val dataRDD = sc.parallelize(Array((150001, "US"), (150002, "CN"), (150003, "CN"), (150004, "IN")))
    dataRDD.map(tup => (tup._2, 1)).reduceByKey(_ + _).foreach(println(_))
  }

  /**
    * groupByKey：对每个大区的主播进行分组(tup超过2列数据时)
    * area: CN, <150002, female> <150003, male>
    * area: US, <150001, male>
    * area: IN, <150004, male>
    *
    * @param sc
    */
  private def groupByKeyOp2(sc: SparkContext): Unit = {
    // 主播ID、主播所在大区、性别
    val dataRDD = sc.parallelize(Array((150001, "US", "male"), (150002, "CN", "female"), (150003, "CN", "male"), (150004, "IN", "male")))

    // groupByKey需要tup2类型, 且key必须在第一位
    dataRDD.map(tup => (tup._2, (tup._1, tup._3))).groupByKey().foreach(tup => {
      // 大区
      val area = tup._1
      print("area: " + area + ", ")

      // 迭代同一个大区的所有主播ID
      for ((uid, sex) <- tup._2) {
        print("<" + uid + ", " + sex + "> ")
      }
      println()
    })
  }

  /**
    * groupByKey：对每个大区的主播进行分组
    * area: CN, 150002 150003
    * area: US, 150001
    * area: IN, 150004
    *
    * @param sc
    */
  private def groupByKeyOp(sc: SparkContext): Unit = {
    // 主播ID、主播所在大区
    val dataRDD = sc.parallelize(Array((150001, "US"), (150002, "CN"), (150003, "CN"), (150004, "IN")))

    // groupByKey需要tup2类型, 且key必须在第一位
    dataRDD.map(tup => (tup._2, tup._1)).groupByKey().foreach(tup => {
      // 大区
      val area = tup._1
      print("area: " + area + ", ")

      // 迭代同一个大区的所有主播ID
      for (uid <- tup._2) {
        print(uid + " ")
      }
      println()
    })
  }

  /**
    * flatMap：将行拆分为单词
    * good
    * good
    * study
    * day
    * day
    * up
    *
    * @param sc
    */
  private def flatMapOp(sc: SparkContext): Unit = {
    val dataRDD = sc.parallelize(Array("good good study", "day day up"))
    dataRDD.flatMap(_.split(" ")).foreach(println(_))
  }

  /**
    * filter：过滤出集合中的偶数
    * 2
    * 4
    *
    * @param sc
    */
  private def filterOp(sc: SparkContext): Unit = {
    val dataRDD = sc.parallelize(Array(1, 2, 3, 4, 5))
    dataRDD.filter(_ % 2 == 0).foreach(println(_))
  }

  /**
    * map：对集合中每一个元素*2
    * 2
    * 4
    * 6
    * 8
    * 10
    *
    * @param sc
    */
  private def mapOp(sc: SparkContext): Unit = {
    val dataRDD = sc.parallelize(Array(1, 2, 3, 4, 5))
    dataRDD.map(_ * 2).foreach(println(_))
  }

  /**
    * 创建sparkContext
    *
    * @return
    */
  private def getSparkContext(appName: String): SparkContext = {
    val conf = new SparkConf()
    conf.setAppName(appName)
      .setMaster("local")
    new SparkContext(conf)
  }
}
```

##### ② Java

```java
package com.bigdata.java;

import org.apache.spark.SparkConf;
import org.apache.spark.api.java.JavaPairRDD;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.JavaSparkContext;
import org.apache.spark.api.java.function.*;
import scala.Tuple2;
import scala.Tuple3;

import java.util.Arrays;
import java.util.Iterator;

/**
 * Transformation 算子
 *
 * @author yaocs2
 * @since 2022-09-17
 */
public class TransformationOpJava {

    public static void main(String[] args) {
        // 创建sparkContext
        JavaSparkContext sc = getSparkContext("TransformationOpJava");

        // map：对集合中每一个元素*2
//        mapOp(sc);

        // filter：过滤出集合中的偶数
//        filterOp(sc);

        // flatMap：将行拆分为单词
//        flatMapOp(sc);

        // groupByKey：对每个大区的主播进行分组
//        groupByKeyOp(sc);
//        groupByKeyOp2(sc);

        // reduceByKey：统计每个大区的主播数量
//        reduceByKeyOp(sc);

        // sortByKey：对主播的音浪收入排序
//        sortByKeyOp(sc);
//        sortByOp(sc);

        // join：打印每个主播大区信息和音浪收入
//        joinOp(sc);

        // distinct：统计当天开播的大区信息
//        distinctOp(sc);

        // 停止sparkContext
        sc.stop();
    }

    /**
     * distinct：统计当天开播的大区信息
     * CN
     * US
     * IN
     *
     * @param sc
     */
    private static void distinctOp(JavaSparkContext sc) {
        // 主播ID、主播所在大区
        Tuple2<Integer, String> t1 = new Tuple2<>(150001, "US");
        Tuple2<Integer, String> t2 = new Tuple2<>(150002, "CN");
        Tuple2<Integer, String> t3 = new Tuple2<>(150003, "CN");
        Tuple2<Integer, String> t4 = new Tuple2<>(150004, "IN");
        JavaRDD<Tuple2<Integer, String>> dataRDD = sc.parallelize(Arrays.asList(t1, t2, t3, t4));

        dataRDD.map(new Function<Tuple2<Integer, String>, String>() {
            @Override
            public String call(Tuple2<Integer, String> tup) throws Exception {
                return tup._2;
            }
        }).distinct().foreach(new VoidFunction<String>() {
            @Override
            public void call(String s) throws Exception {
                System.out.println(s);
            }
        });
    }

    /**
     * join：打印每个主播大区信息和音浪收入
     * (150001,(US,400))
     * (150002,(CN,200))
     * (150003,(CN,300))
     * (150004,(IN,100))
     *
     * @param sc
     */
    private static void joinOp(JavaSparkContext sc) {
        // 主播ID、主播所在大区
        Tuple2<Integer, String> t1 = new Tuple2<>(150001, "US");
        Tuple2<Integer, String> t2 = new Tuple2<>(150002, "CN");
        Tuple2<Integer, String> t3 = new Tuple2<>(150003, "CN");
        Tuple2<Integer, String> t4 = new Tuple2<>(150004, "IN");
        JavaRDD<Tuple2<Integer, String>> dataRDD1 = sc.parallelize(Arrays.asList(t1, t2, t3, t4));

        // 主播ID、主播的音浪收入
        Tuple2<Integer, Integer> t5 = new Tuple2<>(150001, 400);
        Tuple2<Integer, Integer> t6 = new Tuple2<>(150002, 200);
        Tuple2<Integer, Integer> t7 = new Tuple2<>(150003, 300);
        Tuple2<Integer, Integer> t8 = new Tuple2<>(150004, 100);
        JavaRDD<Tuple2<Integer, Integer>> dataRDD2 = sc.parallelize(Arrays.asList(t5, t6, t7, t8));

        // 先转换为JavaPairRDD
        JavaPairRDD<Integer, String> dataPairRDD1 = dataRDD1.mapToPair(new PairFunction<Tuple2<Integer, String>, Integer, String>() {
            @Override
            public Tuple2<Integer, String> call(Tuple2<Integer, String> tup) throws Exception {
                return new Tuple2<>(tup._1, tup._2);
            }
        });
        JavaPairRDD<Integer, Integer> dataPairRDD2 = dataRDD2.mapToPair(new PairFunction<Tuple2<Integer, Integer>, Integer, Integer>() {
            @Override
            public Tuple2<Integer, Integer> call(Tuple2<Integer, Integer> tup) throws Exception {
                return new Tuple2<>(tup._1, tup._2);
            }
        });

        // 根据key进行join操作
        dataPairRDD1.join(dataPairRDD2).foreach(new VoidFunction<Tuple2<Integer, Tuple2<String, Integer>>>() {
            @Override
            public void call(Tuple2<Integer, Tuple2<String, Integer>> tup) throws Exception {
                System.out.println(tup);
            }
        });
    }

    /**
     * sortBy：对主播的音浪收入排序(指定某一列进行排序)
     * (150004,100)
     * (150002,200)
     * (150003,300)
     * (150001,400)
     *
     * @param sc
     */
    private static void sortByOp(JavaSparkContext sc) {
        // 主播ID、主播的音浪收入
        Tuple2<Integer, Integer> t1 = new Tuple2<>(150001, 400);
        Tuple2<Integer, Integer> t2 = new Tuple2<>(150002, 200);
        Tuple2<Integer, Integer> t3 = new Tuple2<>(150003, 300);
        Tuple2<Integer, Integer> t4 = new Tuple2<>(150004, 100);
        JavaRDD<Tuple2<Integer, Integer>> dataRDD = sc.parallelize(Arrays.asList(t1, t2, t3, t4));

        // sortBy时, 可以直接指定音浪收入作为key, 无需在第一位, 默认为true, 代表正序, 可以指定为false, 代表逆序
        dataRDD.sortBy(new Function<Tuple2<Integer, Integer>, Integer>() {
            @Override
            public Integer call(Tuple2<Integer, Integer> tup) throws Exception {
                return tup._2;
            }
        }, true, 1).foreach(new VoidFunction<Tuple2<Integer, Integer>>() {
            @Override
            public void call(Tuple2<Integer, Integer> tup) throws Exception {
                System.out.println(tup);
            }
        });
    }

    /**
     * sortByKey：对主播的音浪收入排序
     * (100,150004)
     * (200,150002)
     * (300,150003)
     * (400,150001)
     *
     * @param sc
     */
    private static void sortByKeyOp(JavaSparkContext sc) {
        // 主播ID、主播的音浪收入
        Tuple2<Integer, Integer> t1 = new Tuple2<>(150001, 400);
        Tuple2<Integer, Integer> t2 = new Tuple2<>(150002, 200);
        Tuple2<Integer, Integer> t3 = new Tuple2<>(150003, 300);
        Tuple2<Integer, Integer> t4 = new Tuple2<>(150004, 100);
        JavaRDD<Tuple2<Integer, Integer>> dataRDD = sc.parallelize(Arrays.asList(t1, t2, t3, t4));

        // sortByKey时, 音浪收入作为key, 必须在第一位, 默认为true, 代表正序, 可以指定为false, 代表逆序
        // 底层是先通过RangePartitioner按照范围分区抽样，再对每个分区的数据进行排序，排完序后，分区内有序、分区间也有序，从而实现全局排序
        // 比如32G如何排序1T数据，也可以使用这种思想：先按范围分片、再排序、最后分片内有序、分片间也有序，从而实现全局排序
        dataRDD.mapToPair(new PairFunction<Tuple2<Integer, Integer>, Integer, Integer>() {
            @Override
            public Tuple2<Integer, Integer> call(Tuple2<Integer, Integer> tup) throws Exception {
                return new Tuple2<>(tup._2, tup._1);
            }
        }).sortByKey().foreach(new VoidFunction<Tuple2<Integer, Integer>>() {
            @Override
            public void call(Tuple2<Integer, Integer> tup) throws Exception {
                System.out.println(tup);
            }
        });
    }

    /**
     * reduceByKey：统计每个大区的主播数量
     * (CN,2)
     * (US,1)
     * (IN,1)
     *
     * @param sc
     */
    private static void reduceByKeyOp(JavaSparkContext sc) {
        // 主播ID、主播所在大区
        Tuple2<Integer, String> t1 = new Tuple2<>(150001, "US");
        Tuple2<Integer, String> t2 = new Tuple2<>(150002, "CN");
        Tuple2<Integer, String> t3 = new Tuple2<>(150003, "CN");
        Tuple2<Integer, String> t4 = new Tuple2<>(150004, "IN");
        JavaRDD<Tuple2<Integer, String>> dataRDD = sc.parallelize(Arrays.asList(t1, t2, t3, t4));

        dataRDD.mapToPair(new PairFunction<Tuple2<Integer, String>, String, Integer>() {
            @Override
            public Tuple2<String, Integer> call(Tuple2<Integer, String> tup) throws Exception {
                return new Tuple2<>(tup._2, 1);
            }
        }).reduceByKey(new Function2<Integer, Integer, Integer>() {
            @Override
            public Integer call(Integer i1, Integer i2) throws Exception {
                return i1 + i2;
            }
        }).foreach(new VoidFunction<Tuple2<String, Integer>>() {
            @Override
            public void call(Tuple2<String, Integer> tup) throws Exception {
                System.out.println(tup);
            }
        });
    }

    /**
     * groupByKey：对每个大区的主播进行分组(tup超过2列数据时)
     * area: CN, <150002, female> <150003, male>
     * area: US, <150001, male>
     * area: IN, <150004, male>
     *
     * @param sc
     */
    private static void groupByKeyOp2(JavaSparkContext sc) {
        // 主播ID、主播所在大区、性别
        Tuple3<Integer, String, String> t1 = new Tuple3<>(150001, "US", "male");
        Tuple3<Integer, String, String> t2 = new Tuple3<>(150002, "CN", "female");
        Tuple3<Integer, String, String> t3 = new Tuple3<>(150003, "CN", "male");
        Tuple3<Integer, String, String> t4 = new Tuple3<>(150004, "IN", "male");
        JavaRDD<Tuple3<Integer, String, String>> dataRDD = sc.parallelize(Arrays.asList(t1, t2, t3, t4));

        dataRDD.mapToPair(new PairFunction<Tuple3<Integer, String, String>, String, Tuple2<Integer, String>>() {
            @Override
            public Tuple2<String, Tuple2<Integer, String>> call(Tuple3<Integer, String, String> tup) throws Exception {
                return new Tuple2<>(tup._2(), new Tuple2<>(tup._1(), tup._3()));
            }
        }).groupByKey().foreach(new VoidFunction<Tuple2<String, Iterable<Tuple2<Integer, String>>>>() {
            @Override
            public void call(Tuple2<String, Iterable<Tuple2<Integer, String>>> tup) throws Exception {
                // 大区
                String area = tup._1;
                System.out.print("area: " + area + ", ");

                // 迭代同一个大区的所有主播ID
                for (Tuple2<Integer, String> uidSex : tup._2) {
                    System.out.print("<" + uidSex._1 + ", " + uidSex._2 + "> ");
                }
                System.out.println();
            }
        });
    }

    /**
     * groupByKey：对每个大区的主播进行分组
     * area: CN, 150002 150003
     * area: US, 150001
     * area: IN, 150004
     *
     * @param sc
     */
    private static void groupByKeyOp(JavaSparkContext sc) {
        // 主播ID、主播所在大区
        Tuple2<Integer, String> t1 = new Tuple2<>(150001, "US");
        Tuple2<Integer, String> t2 = new Tuple2<>(150002, "CN");
        Tuple2<Integer, String> t3 = new Tuple2<>(150003, "CN");
        Tuple2<Integer, String> t4 = new Tuple2<>(150004, "IN");
        JavaRDD<Tuple2<Integer, String>> dataRDD = sc.parallelize(Arrays.asList(t1, t2, t3, t4));

        // groupByKey需要tup2类型, 且key必须在第一位
        dataRDD.mapToPair(new PairFunction<Tuple2<Integer, String>, String, Integer>() {
            @Override
            public Tuple2<String, Integer> call(Tuple2<Integer, String> tup) throws Exception {
                return new Tuple2<>(tup._2, tup._1);
            }
        }).groupByKey().foreach(new VoidFunction<Tuple2<String, Iterable<Integer>>>() {
            @Override
            public void call(Tuple2<String, Iterable<Integer>> tup) throws Exception {
                // 大区
                String area = tup._1;
                System.out.print("area: " + area + ", ");

                // 迭代同一个大区的所有主播ID
                for (Integer uid : tup._2) {
                    System.out.print(uid + " ");
                }
                System.out.println();
            }
        });
    }

    /**
     * flatMap：将行拆分为单词
     * good
     * good
     * study
     * day
     * day
     * up
     *
     * @param sc
     */
    private static void flatMapOp(JavaSparkContext sc) {
        JavaRDD<String> dataRDD = sc.parallelize(Arrays.asList("good good study", "day day up"));
        dataRDD.flatMap(new FlatMapFunction<String, String>() {
            @Override
            public Iterator<String> call(String s) throws Exception {
                return Arrays.asList(s.split(" ")).iterator();
            }
        }).foreach(new VoidFunction<String>() {
            @Override
            public void call(String s) throws Exception {
                System.out.println(s);
            }
        });
    }

    /**
     * filter：过滤出集合中的偶数
     * 2
     * 4
     *
     * @param sc
     */
    private static void filterOp(JavaSparkContext sc) {
        JavaRDD<Integer> dataRDD = sc.parallelize(Arrays.asList(1, 2, 3, 4, 5));
        dataRDD.filter(new Function<Integer, Boolean>() {
            @Override
            public Boolean call(Integer i) throws Exception {
                return i % 2 == 0;
            }
        }).foreach(new VoidFunction<Integer>() {
            @Override
            public void call(Integer i) throws Exception {
                System.out.println(i);
            }
        });
    }

    /**
     * map：对集合中每一个元素*2
     * 2
     * 4
     * 6
     * 8
     * 10
     *
     * @param sc
     */
    private static void mapOp(JavaSparkContext sc) {
        JavaRDD<Integer> dataRDD = sc.parallelize(Arrays.asList(1, 2, 3, 4, 5));
        dataRDD.map(new Function<Integer, Integer>() {
            @Override
            public Integer call(Integer i) throws Exception {
                return i * 2;
            }
        }).foreach(new VoidFunction<Integer>() {
            @Override
            public void call(Integer i) throws Exception {
                System.out.println(i);
            }
        });
    }

    /**
     * 创建sparkContext
     *
     * @return
     */
    private static JavaSparkContext getSparkContext(String appName) {
        SparkConf conf = new SparkConf();
        conf.setAppName(appName)
                .setMaster("local");
        return new JavaSparkContext(conf);
    }
}
```

#### 3）行动操作 | Action 算子

| 算子           | 作用                                                         |
| -------------- | ------------------------------------------------------------ |
| reduce         | 将 RDD 中所有元素进行聚合操作（两两聚合）                    |
| collect        | 将 RDD 中所有元素发送到 Driver                               |
| take(n)        | 获取 RDD 中的前 n 个元素                                     |
| count          | 获取 RDD 中的元素总数                                        |
| saveAsTextFile | 将 RDD 中的元素保存到 HDFS 文件中，会对每个元素调用 `toString` 方法，产生和 MapReduce 任务执行结果一样的目录结构 |
| countByKey     | 对每个 key 对应的值进行 count 计数                           |
| foreach        | 遍历 RDD 中的每个元素                                        |

##### ① Scala

```scala
package com.bigdata.scala

import org.apache.spark.{SparkConf, SparkContext}

/**
  * Action 算子
  *
  * @author yaocs2
  * @since 2022-09-17
  */
object ActionOpScala {

  def main(args: Array[String]): Unit = {
    // 创建sparkContext
    val sc = getSparkContext("ActionOpScala")

    // reduce：聚合计算
    //    reduceOp(sc)

    // collect：获取元素集合
    //    collectOp(sc)

    // take(n)：获取前N个元素
    //    takeOp(sc)

    // count：获取元素总数
    //    countOp(sc)

    // saveAsTextFile：保存至文件
    //    saveAsTextFileOp(sc)

    // countByKey：根据Key进行元素计数
    //    countByKeyOp(sc)

    // foreach：迭代遍历元素
    foreachOp(sc)

    // 停止sparkContext
    sc.stop()
  }

  /**
    * foreach：迭代遍历元素
    * 1
    * 2
    * 3
    * 4
    * 5
    *
    * @param sc
    */
  def foreachOp(sc: SparkContext): Unit = {
    val dataRDD = sc.parallelize(Array(1, 2, 3, 4, 5))
    dataRDD.foreach(println(_))
  }

  /**
    * countByKey：根据Key进行元素计数
    * B: 1
    * A: 2
    * C: 1
    *
    * @param sc
    */
  def countByKeyOp(sc: SparkContext): Unit = {
    val dataRDD = sc.parallelize(Array(("A", 1001), ("B", 1002), ("A", 1003), ("C", 1004)))
    val res = dataRDD.countByKey()
    for ((k, v) <- res) {
      println(k + ": " + v)
    }
  }

  /**
    * saveAsTextFile：保存至文件
    *
    * @param sc
    */
  def saveAsTextFileOp(sc: SparkContext): Unit = {
    val dataRDD = sc.parallelize(Array(1, 2, 3, 4, 5))

    // 只能保存到HDFS中, 会产生和 MapReduce 任务执行结果一样的目录结构
    dataRDD.saveAsTextFile("hdfs://bigdata01:9000/array_out1")
  }

  /**
    * count：获取元素总数
    * 5
    *
    * @param sc
    */
  def countOp(sc: SparkContext): Unit = {
    val dataRDD = sc.parallelize(Array(1, 2, 3, 4, 5))
    val count = dataRDD.count()
    println(count)
  }

  /**
    * 获取前N个元素
    * 1
    * 2
    *
    * @param sc
    */
  def takeOp(sc: SparkContext): Unit = {
    val dataRDD = sc.parallelize(Array(1, 2, 3, 4, 5))
    dataRDD.take(2).foreach(println(_))
  }

  /**
    * collect：获取元素集合
    * 1
    * 2
    * 3
    * 4
    * 5
    *
    * @param sc
    */
  def collectOp(sc: SparkContext): Unit = {
    val dataRDD = sc.parallelize(Array(1, 2, 3, 4, 5))

    // 注意RDD数据量过大, 可能会导致Driver存不下, 建议使用take(n)
    dataRDD.collect().foreach(println(_))
  }

  /**
    * reduce：聚合计算
    * 15
    *
    * @param sc
    */
  def reduceOp(sc: SparkContext): Unit = {
    val dataRDD = sc.parallelize(Array(1, 2, 3, 4, 5))
    val sum = dataRDD.reduce(_ + _)
    println(sum)
  }

  /**
    * 创建sparkContext
    *
    * @return
    */
  private def getSparkContext(appName: String): SparkContext = {
    val conf = new SparkConf()
    conf.setAppName(appName)
      .setMaster("local")
    new SparkContext(conf)
  }
}
```

##### ②  Java

```java
package com.bigdata.java;

import org.apache.spark.SparkConf;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.JavaSparkContext;
import org.apache.spark.api.java.function.Function2;
import org.apache.spark.api.java.function.PairFunction;
import org.apache.spark.api.java.function.VoidFunction;
import scala.Tuple2;

import java.util.Arrays;
import java.util.Map;

/**
 * Action 算子
 *
 * @author yaocs2
 * @since 2022-09-17
 */
public class ActionOpJava {

    public static void main(String[] args) {
        // 创建sparkContext
        JavaSparkContext sc = getSparkContext("ActionOpJava");

        // reduce：聚合计算
//        reduceOp(sc);

        // collect：获取元素集合
//        collectOp(sc);

        // take(n)：获取前N个元素
//        takeOp(sc);

        // count：获取元素总数
//        countOp(sc);

        // saveAsTextFile：保存至文件
//        saveAsTextFileOp(sc);

        // countByKey：根据Key进行元素计数
//        countByKeyOp(sc);

        // foreach：迭代遍历元素
        foreachOp(sc);

        // 停止sparkContext
        sc.stop();
    }

    /**
     * foreach：迭代遍历元素
     * 1
     * 2
     * 3
     * 4
     * 5
     *
     * @param sc
     */
    private static void foreachOp(JavaSparkContext sc) {
        JavaRDD<Integer> dataRDD = sc.parallelize(Arrays.asList(1, 2, 3, 4, 5));
        dataRDD.foreach(new VoidFunction<Integer>() {
            @Override
            public void call(Integer i) throws Exception {
                System.out.println(i);
            }
        });
    }

    /**
     * countByKey：根据Key进行元素计数
     * B: 1
     * A: 2
     * C: 1
     *
     * @param sc
     */
    private static void countByKeyOp(JavaSparkContext sc) {
        Tuple2<String, Integer> t1 = new Tuple2<>("A", 1001);
        Tuple2<String, Integer> t2 = new Tuple2<>("B", 1002);
        Tuple2<String, Integer> t3 = new Tuple2<>("A", 1003);
        Tuple2<String, Integer> t4 = new Tuple2<>("C", 1004);

        JavaRDD<Tuple2<String, Integer>> dataRDD = sc.parallelize(Arrays.asList(t1, t2, t3, t4));
        Map<String, Long> res = dataRDD.mapToPair(new PairFunction<Tuple2<String, Integer>, String, Integer>() {
            @Override
            public Tuple2<String, Integer> call(Tuple2<String, Integer> tup) throws Exception {
                return new Tuple2<>(tup._1, tup._2);
            }
        }).countByKey();

        for (Map.Entry<String, Long> entry : res.entrySet()) {
            System.out.println(entry.getKey() + ": " + entry.getValue());
        }
    }

    /**
     * saveAsTextFile：保存至文件
     *
     * @param sc
     */
    private static void saveAsTextFileOp(JavaSparkContext sc) {
        JavaRDD<Integer> dataRDD = sc.parallelize(Arrays.asList(1, 2, 3, 4, 5));
        dataRDD.saveAsTextFile("hdfs://bigdata01:9000/array_out2");
    }

    /**
     * count：获取元素总数
     * 5
     *
     * @param sc
     */
    private static void countOp(JavaSparkContext sc) {
        JavaRDD<Integer> dataRDD = sc.parallelize(Arrays.asList(1, 2, 3, 4, 5));
        System.out.println(dataRDD.count());
    }

    /**
     * take(n)：获取前N个元素
     * 1
     * 2
     *
     * @param sc
     */
    private static void takeOp(JavaSparkContext sc) {
        JavaRDD<Integer> dataRDD = sc.parallelize(Arrays.asList(1, 2, 3, 4, 5));
        for (Integer i : dataRDD.take(2)) {
            System.out.println(i);
        }
    }

    /**
     * collect：获取元素集合
     * 1
     * 2
     * 3
     * 4
     * 5
     *
     * @param sc
     */
    private static void collectOp(JavaSparkContext sc) {
        JavaRDD<Integer> dataRDD = sc.parallelize(Arrays.asList(1, 2, 3, 4, 5));
        for (Integer i : dataRDD.collect()) {
            System.out.println(i);
        }
    }

    /**
     * reduce：聚合计算
     * 15
     *
     * @param sc
     */
    private static void reduceOp(JavaSparkContext sc) {
        JavaRDD<Integer> dataRDD = sc.parallelize(Arrays.asList(1, 2, 3, 4, 5));
        Integer sum = dataRDD.reduce(new Function2<Integer, Integer, Integer>() {
            @Override
            public Integer call(Integer i1, Integer i2) throws Exception {
                return i1 + i2;
            }
        });
        System.out.println(sum);
    }

    /**
     * 创建sparkContext
     *
     * @return
     */
    private static JavaSparkContext getSparkContext(String appName) {
        SparkConf conf = new SparkConf();
        conf.setAppName(appName)
                .setMaster("local");
        return new JavaSparkContext(conf);
    }
}
```

#### 4）控制操作 | RDD 持久化

1. RDD 持久化，当对 RDD 执行持久化操作时，每个节点会将自己操作的 RDD#partition 持久化到内存中，并在之后重复操作时，直接使用内存缓存的 RDD。而如果没有做持久化，RDD 中的数据使用后，内存中不会一直保存，在下一次使用 Action 算子操作 RDD 时，需要重新计算，消耗 CPU 和计算时间。
2. 持久化一个 RDD，可以调用 `cache()` 或者 `persist()` 方法，根据指定的持久化策略，将 RDD 缓存到每一个节点中，且 Spark 持久化机制支持自动容错，即如果持久化的 RDD 的任何 partition 数据丢失了，那么 Spark 自动通过原 RDD 使用 Transformation 算子，重新计算该 partition 的数据。
3. `cache()` 是 `persist()` 的简化方式，`cache()` 底层实际调用的是 `persist()` 的无参版本，`persist()` 无参版本调用 `persist(..)` 的有参版本。而如果需要从内存中清除掉，可以使用 `unpersist()` 方法。

| 策略                | 作用                                                         |
| ------------------- | ------------------------------------------------------------ |
| MEMORY_ONLY         | 默认策略，以非序列化的方式持久化在 JVM 内存中，如果内存不能完全存储，那些没有持久化的 partition，则会在下一次使用时被重新计算 |
| MEMORY_AND_DISK     | 优先存储在内存钟，当内存不够时，则会持久化剩余的 partition 到磁盘中 |
| MEMORY_ONLY_SER     | 同 MEMORY_ONLY，但是会使用 java 序列化方式，将 java 对象序列化后再存储，以减少内存开销，但是使用时需要进行反序列化，增加 CPU 开销 |
| MEMORY_AND_DISK_SER | 同 MEMORY_AND_DISK，但是会使用 java 序列化方式，将 java 对象序列化后再存储，以减少内存开销，但是使用时需要进行反序列化，增加 CPU 开销 |
| DISK_ONLY           | 以非序列化的方式，完全存储在磁盘上                           |
| MEMORY_ONLY_2       | 会额外将 RDD 多复制一份，保存到其他节点的内存中，实现副本冗余，优点是数据丢失时，无需重新计算，容错性高，缺点是占的空间是原来的两倍 |
| MEMORY_AND_DISK_2   | 会额外将 RDD 多复制一份，保存到其他节点的磁盘中，实现副本冗余，优点是数据丢失时，无需重新计算，容错性高，缺点是占的空间是原来的两倍 |

如何选择 RDD 持久化策略？

Spark 提供了多种持久化级别，主要是为了在 CPU 和内存消耗之间进行取舍，建议：

1. 优先选择 MEMORY_ONLY，因为是纯内存，速度最快，没有序列化，无需消耗 CPU 进行反序列化操作。
2. 在内存不够多时，可以选择 MEMORY_ONLY_SER，将数据进行序列化存储，纯内存还是非常快，只是要消耗 CPU 进行反序列化。
3. 如果需要数据快速失败恢复，那就选择 MEMORY_ONLY_2，进行副本冗余。
4. 对于磁盘策略，能不使用磁盘就不使用，因为这样可能还不如重新计算来得快。

##### ① Scala

```scala
package com.bigdata.scala

import org.apache.spark.{SparkConf, SparkContext}

/**
  * 持久化RDD
  *
  * @author yaocs2
  * @since 2022-09-18
  */
object PersistRddScala {

  def main(args: Array[String]): Unit = {
    // 创建sparkContext
    val sc = getSparkContext("PersistRddScala")

    // 不开启cache: 第一次耗时：8453, 第二次耗时：8018
    // 开启cache: 第一次耗时：24282, 第二次耗时：5078
    val dataRDD = sc.textFile("D:\\words.dat").cache()

    var startTime = System.currentTimeMillis()
    var count = dataRDD.count()
    println(count)
    var endTime = System.currentTimeMillis()
    println("第一次耗时：" + (endTime - startTime))

    startTime = System.currentTimeMillis()
    count = dataRDD.count()
    println(count)
    endTime = System.currentTimeMillis()
    println("第二次耗时：" + (endTime - startTime))

    // 停止sparkContext
    sc.stop()
  }

  /**
    * 创建sparkContext
    *
    * @return
    */
  private def getSparkContext(appName: String): SparkContext = {
    val conf = new SparkConf()
    conf.setAppName(appName)
      .setMaster("local")
    new SparkContext(conf)
  }
}
```

##### ② Java

```java
package com.bigdata.java;

import org.apache.spark.SparkConf;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.JavaSparkContext;

/**
 * 持久化RDD
 *
 * @author yaocs2
 * @since 2022-09-18
 */
public class PersistRddJava {

    public static void main(String[] args) {
        // 创建sparkContext
        JavaSparkContext sc = getSparkContext("PersistRddJava");

        // 不开启cache: 第一次耗时：8140, 第二次耗时：7603
        // 开启cache: 第一次耗时：25464, 第二次耗时：4826
        JavaRDD<String> dataRDD = sc.textFile("D:\\words.dat");
//                .cache();

        long startTime = System.currentTimeMillis();
        long count = dataRDD.count();
        System.out.println(count);
        long endTime = System.currentTimeMillis();
        System.out.println("第一次耗时：" + (endTime - startTime));

        startTime = System.currentTimeMillis();
        count = dataRDD.count();
        System.out.println(count);
        endTime = System.currentTimeMillis();
        System.out.println("第二次耗时：" + (endTime - startTime));

        // 停止sparkContext
        sc.stop();
    }

    /**
     * 创建sparkContext
     *
     * @return
     */
    private static JavaSparkContext getSparkContext(String appName) {
        SparkConf conf = new SparkConf();
        conf.setAppName(appName)
                .setMaster("local");
        return new JavaSparkContext(conf);
    }
}
```

### 2.0. Spark 综合案例？

#### 1）取 Top3

##### ① Scala

```scala
package com.bigdata.scala

import com.alibaba.fastjson.JSON
import org.apache.spark.{SparkConf, SparkContext}

/**
  * TopN主播统计
  *
  * 1：首先获取两份数据中的核心字段，使用fastjson包解析数据
  * 主播开播记录(video_info.log):主播ID：uid，直播间ID：vid，大区：area
  * (vid,(uid,area))
  * 用户送礼记录(gift_record.log)：直播间ID：vid，金币数量：gold
  * (vid,gold)
  *
  * 这样的话可以把这两份数据关联到一块就能获取到大区、主播id、金币这些信息了，使用直播间vid进行关联
  *
  * 2：对用户送礼记录数据进行聚合，对相同vid的数据求和
  * 因为用户可能在一次直播中给主播送多次礼物
  * (vid,gold_sum)
  *
  * 3：把这两份数据join到一块，vid作为join的key
  * (vid,((uid,area),gold_sum))
  *
  * 4：使用map迭代join之后的数据，最后获取到uid、area、gold_sum字段
  * 由于一个主播一天可能会开播多次，后面需要基于uid和area再做一次聚合，所以把数据转换成这种格式
  *
  * uid和area是一一对应的，一个人只能属于大区
  * ((uid,area),gold_sum)
  *
  * 5：使用reduceByKey算子对数据进行聚合
  * ((uid,area),gold_sum_all)
  *
  * 6：接下来对需要使用groupByKey对数据进行分组，所以先使用map进行转换
  * 因为我们要分区统计TopN，所以要根据大区分组
  * map：(area,(uid,gold_sum_all))
  * groupByKey: area,<(uid,gold_sum_all),(uid,gold_sum_all),(uid,gold_sum_all)>
  *
  * 7：使用map迭代每个分组内的数据，按照金币数量倒序排序，取前N个，最终输出area,topN
  * 这个topN其实就是把前几名主播的id还有金币数量拼接成一个字符串
  * (area,topN)
  *
  * 8：使用foreach将结果打印到控制台，多个字段使用制表符分割
  * area	topN
  *
  * @author yaocs2
  * @since 2022-09-18
  */
object AreaTopNSampleScala {

  def main(args: Array[String]): Unit = {
    val sc = getSparkContext("AreaTopNSampleScala")

    // 1：首先获取两份数据中的核心字段，使用fastjson包解析数据
    val videoInfoRDD = sc.textFile("D:\\Users\\yaocs2\\data\\srcProjects\\bigdata-spark\\src\\main\\resources\\video_info.log")
    val giftRecordRDD = sc.textFile("D:\\Users\\yaocs2\\data\\srcProjects\\bigdata-spark\\src\\main\\resources\\gift_record.log")

    // {"uid":"8407173251001","vid":"14943445328940001","area":"US","status":"1","start_time":"1494344544","end_time":"1494344570","watch_num":101,"share_num":"21","type":"video_info"}
    // (vid, (uid, area))
    val videoInfoFieldRDD = videoInfoRDD.map(line => {
      val jsonObj = JSON.parseObject(line)
      val uid = jsonObj.getString("uid")
      val vid = jsonObj.getString("vid")
      val area = jsonObj.getString("area")
      (vid, (uid, area))
    })

    // {"uid":"7201232141001","vid":"14943445328940001","good_id":"223","gold":"10","timestamp":1494344574,"type":"gift_record"}
    // (vid, gold)
    val giftRecordFieldRDD = giftRecordRDD.map(line => {
      val jsonObj = JSON.parseObject(line)
      val vid = jsonObj.getString("vid")
      val gold = jsonObj.getInteger("gold")
      (vid, gold)
    })

    // 2：对用户送礼记录数据进行聚合，对相同vid的数据求和
    // (vid,gold_sum)
    val giftRecordFieldAggRDD = giftRecordFieldRDD.reduceByKey(_ + _)

    // 3：把这两份数据join到一块，vid作为join的key
    // (vid,((uid,area),gold_sum))
    val joinRDD = videoInfoFieldRDD.join(giftRecordFieldAggRDD)

    // 4：使用map迭代join之后的数据，最后获取到uid、area、gold_sum字段
    // ((uid,area),gold_sum)
    val joinMapRDD = joinRDD.map(tup => {
      // joinRDD: (vid,((uid,area),gold_sum))
      val uid = tup._2._1._1
      val area = tup._2._1._2
      val goldSum = tup._2._2
      ((uid, area), goldSum)
    })

    // 5：使用reduceByKey算子对数据进行聚合
    // ((uid,area),gold_sum_all)
    val uidAggRDD = joinMapRDD.reduceByKey(_ + _)

    // 6：接下来对需要使用groupByKey对数据进行分组，所以先使用map进行转换
    // map：(area,(uid,gold_sum_all))
    // groupByKey: area,<(uid,gold_sum_all),(uid,gold_sum_all),(uid,gold_sum_all)>
    val areaGroupRDD = uidAggRDD.map(tup => {
      // uidAggRDD: ((uid,area),gold_sum_all)
      val area = tup._1._2
      val uid = tup._1._1
      val gold_sum_all = tup._2
      (area, (uid, gold_sum_all))
    }).groupByKey()

    // 7：使用map迭代每个分组内的数据，按照金币数量倒序排序，取前N个，最终输出area,topN
    // (area,topN)
    val top3RDD = areaGroupRDD.map(tup => {
      // areaGroupRDD: area,<(uid,gold_sum_all),(uid,gold_sum_all),(uid,gold_sum_all)>
      val area = tup._1

      // 按金币排序, 取3个, 代表Top3
      val top3 = tup._2.toList
        .sortBy(_._2)
        .reverse
        .take(3)
        // uid: gold_sum_all
        .map(uidGoldTup => uidGoldTup._1 + ": " + uidGoldTup._2)
        // uid: gold_sum_all, uid: gold_sum_all, uid: gold_sum_all
        .mkString(",")

      // 封装结果
      (area, top3)
    })

    // 8：使用foreach将结果打印到控制台，多个字段使用制表符分割
    // area	topN
    /**
      * CN	8407173251008: 120,8407173251003: 60,8407173251014: 50
      * ID	8407173251005: 160,8407173251010: 140,8407173251002: 70
      * US	8407173251015: 180,8407173251012: 70,8407173251001: 60
      */
    top3RDD.foreach(tup => println(tup._1 + "\t" + tup._2))

    // 停止sparkContext
    sc.stop()
  }

  /**
    * 创建sparkContext
    *
    * @return
    */
  private def getSparkContext(appName: String): SparkContext = {
    val conf = new SparkConf()
    conf.setAppName(appName)
      .setMaster("local")
    new SparkContext(conf)
  }
}
```

##### ② Java

```java
package com.bigdata.java;

import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONObject;
import com.google.common.collect.Lists;
import org.apache.spark.SparkConf;
import org.apache.spark.api.java.JavaPairRDD;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.JavaSparkContext;
import org.apache.spark.api.java.function.Function2;
import org.apache.spark.api.java.function.PairFunction;
import org.apache.spark.api.java.function.VoidFunction;
import scala.Tuple2;

import java.util.ArrayList;

/**
 * TopN主播统计
 * <p>
 * 1：首先获取两份数据中的核心字段，使用fastjson包解析数据
 * 主播开播记录(video_info.log):主播ID：uid，直播间ID：vid，大区：area
 * (vid,(uid,area))
 * 用户送礼记录(gift_record.log)：直播间ID：vid，金币数量：gold
 * (vid,gold)
 * <p>
 * 这样的话可以把这两份数据关联到一块就能获取到大区、主播id、金币这些信息了，使用直播间vid进行关联
 * <p>
 * 2：对用户送礼记录数据进行聚合，对相同vid的数据求和
 * 因为用户可能在一次直播中给主播送多次礼物
 * (vid,gold_sum)
 * <p>
 * 3：把这两份数据join到一块，vid作为join的key
 * (vid,((uid,area),gold_sum))
 * <p>
 * 4：使用map迭代join之后的数据，最后获取到uid、area、gold_sum字段
 * 由于一个主播一天可能会开播多次，后面需要基于uid和area再做一次聚合，所以把数据转换成这种格式
 * <p>
 * uid和area是一一对应的，一个人只能属于大区
 * ((uid,area),gold_sum)
 * <p>
 * 5：使用reduceByKey算子对数据进行聚合
 * ((uid,area),gold_sum_all)
 * <p>
 * 6：接下来对需要使用groupByKey对数据进行分组，所以先使用map进行转换
 * 因为我们要分区统计TopN，所以要根据大区分组
 * map：(area,(uid,gold_sum_all))
 * groupByKey: area,<(uid,gold_sum_all),(uid,gold_sum_all),(uid,gold_sum_all)>
 * <p>
 * 7：使用map迭代每个分组内的数据，按照金币数量倒序排序，取前N个，最终输出area,topN
 * 这个topN其实就是把前几名主播的id还有金币数量拼接成一个字符串
 * (area,topN)
 * <p>
 * 8：使用foreach将结果打印到控制台，多个字段使用制表符分割
 * area	topN
 *
 * @author yaocs2
 * @since 2022-09-18
 */
public class AreaTopNSampleJava {

    public static void main(String[] args) {
        // 创建sparkContext
        JavaSparkContext sc = getSparkContext("AreaTopNSampleJava");

        // 1：首先获取两份数据中的核心字段，使用fastjson包解析数据
        JavaRDD<String> videoInfoRDD = sc.textFile("D:\\Users\\yaocs2\\data\\srcProjects\\bigdata-spark\\src\\main\\resources\\video_info.log");
        JavaRDD<String> giftRecordRDD = sc.textFile("D:\\Users\\yaocs2\\data\\srcProjects\\bigdata-spark\\src\\main\\resources\\gift_record.log");

        // {"uid":"8407173251001","vid":"14943445328940001","area":"US","status":"1","start_time":"1494344544","end_time":"1494344570","watch_num":101,"share_num":"21","type":"video_info"}
        // (vid, (uid, area))
        JavaPairRDD<String, Tuple2<String, String>> videoInfoFieldRDD = videoInfoRDD.mapToPair(new PairFunction<String, String, Tuple2<String, String>>() {
            @Override
            public Tuple2<String, Tuple2<String, String>> call(String line) throws Exception {
                JSONObject jsonObj = JSON.parseObject(line);
                String uid = jsonObj.getString("uid");
                String vid = jsonObj.getString("vid");
                String area = jsonObj.getString("area");
                return new Tuple2<>(vid, new Tuple2<>(uid, area));
            }
        });

        // {"uid":"7201232141001","vid":"14943445328940001","good_id":"223","gold":"10","timestamp":1494344574,"type":"gift_record"}
        // (vid, gold)
        JavaPairRDD<String, Integer> giftRecordFieldRDD = giftRecordRDD.mapToPair(new PairFunction<String, String, Integer>() {
            @Override
            public Tuple2<String, Integer> call(String line) throws Exception {
                JSONObject jsonObj = JSON.parseObject(line);
                String vid = jsonObj.getString("vid");
                Integer gold = jsonObj.getInteger("gold");
                return new Tuple2<>(vid, gold);
            }
        });

        // 2：对用户送礼记录数据进行聚合，对相同vid的数据求和
        // (vid,gold_sum)
        JavaPairRDD<String, Integer> giftRecordFieldAggRDD = giftRecordFieldRDD.reduceByKey(new Function2<Integer, Integer, Integer>() {
            @Override
            public Integer call(Integer i1, Integer i2) throws Exception {
                return i1 + i2;
            }
        });

        // 3：把这两份数据join到一块，vid作为join的key
        // (vid,((uid,area),gold_sum))
        // Java中, join提取变量会报错：Incompatible equality constraint: Integer and T2
//        JavaPairRDD<String, Tuple2<Tuple2<String, String>, Integer>> joinRDD = videoInfoFieldRDD.join(giftRecordFieldAggRDD);

        // 所以join+map二合一了
        // 4：使用map迭代join之后的数据，最后获取到uid、area、gold_sum字段
        // ((uid,area),gold_sum)
        JavaPairRDD<Tuple2<String, String>, Integer> joinMapRDD = videoInfoFieldRDD.join(giftRecordFieldAggRDD)
                .mapToPair(new PairFunction<Tuple2<String, Tuple2<Tuple2<String, String>, Integer>>, Tuple2<String, String>, Integer>() {
                    @Override
                    public Tuple2<Tuple2<String, String>, Integer> call(Tuple2<String, Tuple2<Tuple2<String, String>, Integer>> tup) throws Exception {
                        // joinRDD: (vid,((uid,area),gold_sum))
                        String uid = tup._2._1._1;
                        String area = tup._2._1._2;
                        Integer goldSum = tup._2._2;
                        return new Tuple2<>(new Tuple2<>(uid, area), goldSum);
                    }
                });

        // 5：使用reduceByKey算子对数据进行聚合
        // ((uid,area),gold_sum_all)
        JavaPairRDD<Tuple2<String, String>, Integer> uidAggRDD = joinMapRDD.reduceByKey(new Function2<Integer, Integer, Integer>() {
            @Override
            public Integer call(Integer i1, Integer i2) throws Exception {
                return i1 + i2;
            }
        });

        // 6：接下来对需要使用groupByKey对数据进行分组，所以先使用map进行转换
        // map：(area,(uid,gold_sum_all))
        // groupByKey: area,<(uid,gold_sum_all),(uid,gold_sum_all),(uid,gold_sum_all)>
        JavaPairRDD<String, Iterable<Tuple2<String, Integer>>> areaGroupRDD = uidAggRDD.mapToPair(new PairFunction<Tuple2<Tuple2<String, String>, Integer>, String, Tuple2<String, Integer>>() {
            @Override
            public Tuple2<String, Tuple2<String, Integer>> call(Tuple2<Tuple2<String, String>, Integer> tup) throws Exception {
                // uidAggRDD: ((uid,area),gold_sum_all)
                String area = tup._1._2;
                String uid = tup._1._1;
                Integer gold_sum_all = tup._2;
                return new Tuple2<>(area, new Tuple2<>(uid, gold_sum_all));
            }
        }).groupByKey();

        // 7：使用map迭代每个分组内的数据，按照金币数量倒序排序，取前N个，最终输出area,topN
        // (area,topN)
        JavaPairRDD<String, String> top3RDD = areaGroupRDD.mapToPair(new PairFunction<Tuple2<String, Iterable<Tuple2<String, Integer>>>, String, String>() {
            @Override
            public Tuple2<String, String> call(Tuple2<String, Iterable<Tuple2<String, Integer>>> tup) throws Exception {
                // areaGroupRDD: area,<(uid,gold_sum_all),(uid,gold_sum_all),(uid,gold_sum_all)>
                String area = tup._1;

                // 按金币排序, 取3个, 代表Top3
                ArrayList<Tuple2<String, Integer>> tupleList = Lists.newArrayList(tup._2);
                tupleList.sort((o1, o2) -> o2._2 - o1._2);

                // uid: gold_sum_all, uid: gold_sum_all, uid: gold_sum_all
                StringBuilder sb = new StringBuilder();
                for (int i = 0; i < tupleList.size(); i++) {
                    // top3
                    if (i < 3) {
                        Tuple2<String, Integer> uidGoldTup = tupleList.get(i);
                        if (i != 0) {
                            sb.append(",");
                        }
                        sb.append(uidGoldTup._1).append(": ").append(uidGoldTup._2);
                    }
                }
                String top3 = sb.toString();

                // 封装结果
                return new Tuple2<>(area, top3);
            }
        });

        // 8：使用foreach将结果打印到控制台，多个字段使用制表符分割
        // area	topN
        /**
         * CN	8407173251008: 120,8407173251003: 60,8407173251014: 50
         * ID	8407173251005: 160,8407173251010: 140,8407173251007: 70
         * US	8407173251015: 180,8407173251012: 70,8407173251001: 60
         */
        top3RDD.foreach(new VoidFunction<Tuple2<String, String>>() {
                            @Override
                            public void call(Tuple2<String, String> tup) throws Exception {
                                System.out.println(tup._1 + "\t" + tup._2);
                            }
                        }
        );

        // 停止sparkContext
        sc.stop();
    }

    /**
     * 创建sparkContext
     *
     * @return
     */
    private static JavaSparkContext getSparkContext(String appName) {
        SparkConf conf = new SparkConf();
        conf.setAppName(appName)
                .setMaster("local");
        return new JavaSparkContext(conf);
    }
}
```

### 2.1. Spark 高级特性？

#### 1）共享变量

1. 默认情况下，一个算子函数中使用到了某个外部的变量，这个变量的值会被拷贝到每个 Task 中，此时每个Task 只能操作自己的那份变量数据。
2. Spark 提供了两种共享变量，一种是 Broadcast Variable 广播变量，另一种是 Accumulator 累加变量。

##### 1、广播变量

广播变量，Broadcast Variable 会将使用到的变量，为每个节点拷贝一份，而非每个 Task，同一个节点的 Task 共享同一份变量数据（只可读），减少变量拷贝的次数，减少网络传输以及内存占用。

![1663479603807](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1663479603807.png)

###### ① Scala

```scala
package com.bigdata.scala

import org.apache.spark.{SparkConf, SparkContext}

/**
  * 广播变量
  *
  * @author yaocs2
  * @since 2022-09-18
  */
object BroadcastOpScala {

  def main(args: Array[String]): Unit = {
    // 创建sparkContext
    val sc = getSparkContext("BroadcastOpScala")

    // 构造RDD
    val dataRDD = sc.parallelize(Array(1, 2, 3, 4, 5))

    // 非广播变量
    /**
      * 2
      * 4
      * 6
      * 8
      * 10
      */
    val factor = 2
    //    dataRDD.map(_ * factor)

    // 广播变量(只可读)
    /**
      * 2
      * 4
      * 6
      * 8
      * 10
      */
    val broadcastFactor = sc.broadcast(factor)
    dataRDD.map(_ * broadcastFactor.value).foreach(println(_))

    // 停止sparkContext
    sc.stop()
  }

  /**
    * 创建sparkContext
    *
    * @return
    */
  private def getSparkContext(appName: String): SparkContext = {
    val conf = new SparkConf()
    conf.setAppName(appName)
      .setMaster("local")
    new SparkContext(conf)
  }
}
```

###### ②  Java

```java
package com.bigdata.java;

import org.apache.spark.SparkConf;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.JavaSparkContext;
import org.apache.spark.api.java.function.Function;
import org.apache.spark.api.java.function.VoidFunction;
import org.apache.spark.broadcast.Broadcast;

import java.util.Arrays;

/**
 * 广播变量
 *
 * @author yaocs2
 * @since 2022-09-18
 */
public class BroadcastOpJava {

    public static void main(String[] args) {
        // 创建sparkContext
        JavaSparkContext sc = getSparkContext("BroadcastOpJava");

        // 构造RDD
        JavaRDD<Integer> dataRDD = sc.parallelize(Arrays.asList(1, 2, 3, 4, 5));

        // 非广播变量
        /**
         * 2
         * 4
         * 6
         * 8
         * 10
         */
        int factor = 2;
//        dataRDD.map((Function<Integer, Integer>) i -> i * 2);

        // 广播变量(只可读)
        /**
         * 2
         * 4
         * 6
         * 8
         * 10
         */
        Broadcast<Integer> broadcastFactor = sc.broadcast(factor);
        dataRDD.map((Function<Integer, Integer>) i -> i * broadcastFactor.value())
                .foreach((VoidFunction<Integer>) i -> System.out.println(i));

        // 停止sparkContext
        sc.stop();
    }

    /**
     * 创建sparkContext
     *
     * @return
     */
    private static JavaSparkContext getSparkContext(String appName) {
        SparkConf conf = new SparkConf();
        conf.setAppName(appName)
                .setMaster("local");
        return new JavaSparkContext(conf);
    }
}
```

##### 2、累加变量

累加变量，Accumulator，用于多个节点对一个变量进行共享性的累加操作。

###### ① Scala

```scala
package com.bigdata.scala

import org.apache.spark.{SparkConf, SparkContext}

/**
  * 累加变量
  *
  * @author yaocs2
  * @since 2022-09-18
  */
object AccumulatorOpScala {

  def main(args: Array[String]): Unit = {
    // 创建sparkContext
    val sc = getSparkContext("AccumulatorOpScala")

    // 构造RDD
    val dataRDD = sc.parallelize(Array(1, 2, 3, 4, 5))

    // 非Accumulator变量
    // 打印0：说明Driver端定义的非RDD数据, 只是作为变量拷贝到各个task中, 不能完成全局的累加操作
    //    var total = 0
    //    dataRDD.foreach(num => total += num)
    //    println(total)

    // Accumulator变量：可以全局累加
    /**
      * Task: 1
      * Task: 3
      * Task: 6
      * Task: 10
      * Task: 15
      * Driver: 15
      */
    val sumAccumulator = sc.longAccumulator
    dataRDD.foreach(num => {
      sumAccumulator.add(num)
      println("Task: " + sumAccumulator.value)
    })
    println("Driver: " + sumAccumulator.value)

    // 停止sparkContext
    sc.stop()
  }

  /**
    * 创建sparkContext
    *
    * @return
    */
  private def getSparkContext(appName: String): SparkContext = {
    val conf = new SparkConf()
    conf.setAppName(appName)
      .setMaster("local")
    new SparkContext(conf)
  }
}
```

###### ② Java

```java
package com.bigdata.java;

import org.apache.spark.SparkConf;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.JavaSparkContext;
import org.apache.spark.api.java.function.VoidFunction;
import org.apache.spark.util.LongAccumulator;

import java.util.Arrays;

/**
 * 累加变量
 *
 * @author yaocs2
 * @since 2022-09-18
 */
public class AccumulatorOpJava {

    public static void main(String[] args) {
        // 创建sparkContext
        JavaSparkContext sc = getSparkContext("AccumulatorOpJava");

        // 构造RDD
        JavaRDD<Integer> dataRDD = sc.parallelize(Arrays.asList(1, 2, 3, 4, 5));

        // 非Accumulator变量
        // 打印0：说明Driver端定义的非RDD数据, 只是作为变量拷贝到各个task中, 不能完成全局的累加操作
//        final long[] total = {0};
//        dataRDD.foreach(new VoidFunction<Integer>() {
//            @Override
//            public void call(Integer i) throws Exception {
//                total[0] += i;
//            }
//        });
//        System.out.println(total[0]);

        // Accumulator变量：可以全局累加
        /**
         * Task: 1
         * Task: 3
         * Task: 6
         * Task: 10
         * Task: 15
         * Driver: 15
         */
        LongAccumulator sumAccumulator = sc.sc().longAccumulator();
        dataRDD.foreach(new VoidFunction<Integer>() {
            @Override
            public void call(Integer i) throws Exception {
                sumAccumulator.add(i);
                System.out.println("Task: " + sumAccumulator.value());
            }
        });
        System.out.println("Driver: " + sumAccumulator.value());

        // 停止sparkContext
        sc.stop();
    }

    /**
     * 创建sparkContext
     *
     * @return
     */
    private static JavaSparkContext getSparkContext(String appName) {
        SparkConf conf = new SparkConf();
        conf.setAppName(appName)
                .setMaster("local");
        return new JavaSparkContext(conf);
    }
}
```

#### 2）窄依赖 & 宽依赖

1. 窄依赖：父 RDD 的每个分区只被子 RDD 的一个分区所使用，即每一个子 partition 仅仅依赖于父 RDD 中的一个 partition，不会产生 shuffle 操作，比如 map、filter。
2. 宽依赖：父 RDD 的每个分区可能被子 RDD 的多个分区使用，此时父 RDD 和子 RDD 的 partition 之间是多对多的关系，会产生 shuffle 操作，比如 groupBy，reduceBy。

![1663485997054](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1663485997054.png)

#### 3）Stage

1. Spark Job，是 action 算子触发的，遇到一个 action 算子就会启动一个 Job。
2. Spark Job 会被划分为多个 Stage，Stage 的划分是看是否产生了 shuffle（即宽依赖），遇到一个 shuffle 操作，就会被划分为前后两个 Stage，注意，是从后往前、遇到 shuffle（宽依赖）就切割出一个 stage。
3. 每一个 Stage 由一组并行的 Task 组成，Stage 会将一批 Task，用 TaskSet 封装起来，提交给 TaskScheduler，让 TaskScheduler 进行分配，最后发送给 Executor 执行。

![1663486632395](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1663486632395.png)

#### 4）Spark Job 提交模式

1. standalone 模式：

   ```shell
   spark-submit --master spark://bigdata01:7077
   ```

2. yarn client 模式：

   - 优点：client 模式会将提交任务的机器作为 Driver 节点，此时如果有日志输出，则可以直接在控制台查看，测试比较方便。
   - 缺点：Driver节点和 yarn 集群会存在大量的通信，会消耗带宽以及可能会出现 Driver 内存溢出等问题，生产不使用这种方式。

   ```shell
   spark-submit --master yarn --deploy-mode client
   ```

3. yarn cluster 模式：【生产推荐此种模式】

   - 优点：cluster 模式会将集群中某一个节点作为 Driver 节点，此时与 yarn 集群都是内网通信，且配置较高，不会出现内存溢出等问题。
   - 缺点：查看日志需要去集群中查看。

   ```shell
   spark-submit --master yarn --deploy-mode cluster
   ```

![1663514416738](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1663514416738.png)

#### 5）Shuffle

1. MapReduce 中的 shuffle，是连接 Map 和 Reduce 阶段的桥梁。
2. Map阶段，通过 shuffle 读取数据，并且把数据输出到对应的 Reduce 里面。
3. Reduce 阶段，则负责从 Map 端拉取数据进行计算，在整个 shuffle 阶段，伴随着大量磁盘和网络 I/O，其性能的高低，也直接决定了整个 MapReduce 程序的性能高低。
4. 而 Spark 也有自己的 shuffle 过程，在 `groupByKey，sortByKey，countByKey，join` 等情况下会发生 shuffle。

##### 1、未优化的 Hash Based Shuffle

1. ShuffleMapTask：负责拉取前一个RDD的数据。
2. ResultTask：负责将拉取的数据，按照算子规则汇总起来，比如：groupByKey，此时 ResultTask 的数量等于数据中 Key 的数量。
3. ShuffleMapTask 会为每一个 ResultTask，创建一个 Bucket 缓存、以及 ShuffleBlockFile（SBF），此时产生的文件数量 = mapTask * resultTask。
4. 缺点：
   - 1）ShuffleMapTask 会将数据全部写入 Bucket 后，才会写入 ShuffleBlockFile，此时，如果 map 中数据量特别大，就会导致内存溢出。
   - 2）文件数量多。

![1663514886687](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1663514886687.png)

##### 2、优化后的 hash based shuffle

优化内容：

1. 避免了内存溢出：Bucket 可以设置阈值，达到阈值后，逐步开始写入到 ShuffleBlockFile 文件中，避免了内存溢出问题，但同时也带来了磁盘 I/O 的消耗。
2. 减少了文件数量：引入 Consolidation 文件合并机制， 同一 Executor 下的 ShuffleMapTask 共享一份 Bucket 缓存、以及 ShuffleBlockFile（SBF），此时文件数量 = Executor 节点数 * ResultTask，但如果 ResultTask比较多时，还是会产生很多的小文件。

![1663515067497](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1663515067497.png)

##### 3、Sort-Based Shuffle

是目前 Spark 默认使用的 Shuffle 机制，其优化内容有：

1. 减少了文件数量：每一个 shuffleMapTask 只产生一个 data 文件、以及一个索引文件，文件数量 = mapTask *  2，与 ResultTask 无关。
2. 避免了内存溢出：在内存不够用时，会将内存中的数据溢写到磁盘中，在结束时，将溢写的文件联合内存中的数据一起进行归并，减少内存使用量。

![1663515403672](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1663515403672.png)

#### 6）RDD Checkpoint

针对 Spark job，如果担心某些关键的、在后面会反复使用的 RDD，因为节点故障导致丢失，则可以针对 RDD 启动 checkpoint 机制，从而实现容错和高可用。

##### 1、执行步骤

1. SparkContext 设置 checkpoint 目录，用于存放 checkpoint 数据，对 RDD 调用 checkpoint 方法，然后它就会被 RDDCheckpointData 对象进行管理，此时这个 RDD 的 checkpoint 状态会被设置为 `initialized`。
2. 待 RDD 所在的 job 运行结束后，会调用 job 中最后一个 RDD 的 doCheckpoint 方法，该方法沿着 RDD 的血缘关系，向上查找被 checkpoint 方法标记过的 RDD，并将其 checkpoint 状态设置为 `CheckpointInprocess`。
3. 启动一个单独的 job，将血缘关系中标记为 `CheckpointInprocess` 的 RDD，执行 checkpoint 操作，将数据写入 checkpoint 目录。
4. 数据写入 checkpoint 目录后，会将 RDD 状态变为 `checkpointed`，并且还会改变 RDD 的血缘关系，即会清除掉 RDD 所有依赖的 RDD，最后还会设置其父 RDD 为新创建的 checkpointRDD。

##### 2、Checkpoint vs Persist

|          | RDD Checkpoint                               | RDD 持久化                                                   |
| -------- | -------------------------------------------- | ------------------------------------------------------------ |
| 血缘关系 | 会清除对父 RDD 的依赖，改变了 RDD 的血缘关系 | 不会改变血缘关系                                             |
| 丢失数据 | 将 RDD 保存在 HDFS 中，数据几乎不可能丢失    | 一般选择持久化到内存中，可能会发生数据丢失，如果只持久化到本地磁盘中，数据也还是存在丢失的风险 |

=> 建议：可以对需要 CheckPoint 的 RDD，先 `persist(StorageLevel.DISK_ONLY)` 持久化到磁盘中，后续 CheckPoint 时就无需重新计算数据了，可以读取磁盘上的缓存，直接发送到 HDFS 中，而如果不设置，则还需要重新计算一遍 RDD。

##### 3、代码示例

###### ① Scala

```scala
package com.bigdata.scala

import org.apache.spark.storage.StorageLevel
import org.apache.spark.{SparkConf, SparkContext}

/**
  * RDD Checkpoint
  *
  * @author yaocs2
  * @since 2022-09-19
  */
object CheckpointOpScala {

  def main(args: Array[String]): Unit = {
    // 创建sparkContext
    val sc = getSparkContext("CheckpointOpScala")

    // 设置Checkpoint目录
    sc.setCheckpointDir("hdfs://bigdata01:9000/ckp001")

    // 先持久化RDD数据
    val dataRDD = sc.textFile("D:\\hello.txt").persist(StorageLevel.DISK_ONLY)

    // RDD Checkpoint
    dataRDD.checkpoint()

    // wordCount
    dataRDD.flatMap(_.split(" "))
      .map((_, 1))
      .reduceByKey(_ + _)
      .saveAsTextFile("hdfs://bigdata01:9000/checkpointOpScala")

    // 停止sparkContext
    sc.stop()
  }

  /**
    * 创建sparkContext
    *
    * @return
    */
  private def getSparkContext(appName: String): SparkContext = {
    val conf = new SparkConf()
    conf.setAppName(appName)
      .setMaster("local")
    new SparkContext(conf)
  }
}
```

###### ② Java

```java
package com.bigdata.java;

import org.apache.spark.SparkConf;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.JavaSparkContext;
import org.apache.spark.api.java.function.FlatMapFunction;
import org.apache.spark.api.java.function.Function2;
import org.apache.spark.api.java.function.PairFunction;
import org.apache.spark.storage.StorageLevel;
import scala.Tuple2;

import java.util.Arrays;
import java.util.Iterator;

/**
 * RDD Checkpoint
 *
 * @author yaocs2
 * @since 2022-09-19
 */
public class CheckpointOpJava {

    public static void main(String[] args) {
        // 创建sparkContext
        JavaSparkContext sc = getSparkContext("CheckpointOpJava");

        // 设置Checkpoint目录
        sc.setCheckpointDir("hdfs://bigdata01:9000/ckp002");

        // 读取RDD数据
        JavaRDD<String> dataRDD = sc.textFile("D:\\hello.txt").persist(StorageLevel.DISK_ONLY());

        // RDD Checkpoint
        dataRDD.checkpoint();

        // wordCount
        dataRDD.flatMap(new FlatMapFunction<String, String>() {
            @Override
            public Iterator<String> call(String line) throws Exception {
                return Arrays.asList(line.split(" ")).iterator();
            }
        }).mapToPair(new PairFunction<String, String, Integer>() {
            @Override
            public Tuple2<String, Integer> call(String word) throws Exception {
                return new Tuple2<>(word, 1);
            }
        }).reduceByKey(new Function2<Integer, Integer, Integer>() {
            @Override
            public Integer call(Integer i1, Integer i2) throws Exception {
                return i1 + i2;
            }
        }).saveAsTextFile("hdfs://bigdata01:9000/CheckpointOpJava");

        // 停止sparkContext
        sc.stop();
    }

    /**
     * 创建sparkContext
     *
     * @return
     */
    private static JavaSparkContext getSparkContext(String appName) {
        SparkConf conf = new SparkConf();
        conf.setAppName(appName)
                .setMaster("local");
        return new JavaSparkContext(conf);
    }
}
```

### 2.2. Spark 性能优化？

#### 1）内存调优

1. 一个计算任务的执行，主要依赖于 CPU、内存、带宽。
2. 而 Spark 是一个基于内存的计算引擎，实际工作中，计算任务的性能瓶颈一般出在内存上，所以，优化主要是针对内存进行调优。
3. 每个 java 对象，都有一个对象头，占用 16 个字节，主要包含一些对象的元信息，如果一个对象很小（比如 只有一个 int 字段），那么它的对象头比对象自身的内存还要大。
4. java 的 string 对象的对象头，会比它内部的原始数据多出 40 个字节，因为它内部使用 char 数组，来保存内部的字符序列，并且保存数组长度之类的信息。
5. java 中的集合类型，比如 HashMap 内部使用的是链表数据结构，链表中的每一个数据，都使用了 Entry 对象来包装，Entry 不光有对象头，还有指向下一个 entry 的指针，通常占用 8 个字节。
6. 所以，当我们把原始数据加载至内存中时，占用内存比实际数据要大。

=> 判断 Spark RDD 内存占用的方法：调用 `RDD.cache()` 方法，然后去 Spark 界面上，查看实际使用 Storage 是多少，然后调整 Spark Job 占用的内存。

#### 2）Kryo 序列化调优

1. 如果在算子中使用到了 Driver#java 对象，该对象必须实现序列化，因为数据需要序列化后，再从 Driver 拷贝至 Worker 节点。
2. 对于可以序列化的对象，可以直接实现 Java#Serialable 接口，对于不可以序列化的对象，比如数据库连接之类的，则需要把这段逻辑放置在算子内部实现，在 Task 进程中构造。
3. Spark 提供两种序列化机制：一种是 Java 原生的序列化机制、一种是 Kryo 高性能序列化框架。
   - 1）Java 序列化：性能不高，序列化速度相对比较慢，且序列化后的数据比较大，比较占用内存空间。
   - 2）Kryo序列化：速度快，序列化后的数据更小（比 Java 的差不多小 10 倍左右），但缺点是，对某一些类就算实现了 Serialable 接口，也无法 Kryo 序列化，且使用麻烦，自定义的类想要 Kryo 序列化，必须先在 SparkConf 进行注册，以节省更多的内存（因为 Kryo 无需维护类的全类名）。
4. 注意，如果 Kryo 内部缓存不够存放大对象，则可以在 SparkConf 调大 `spark.kryoserializer.buffer.mb` 参数，默认值为 2，单位为 MB。
5. 什么情况下适合使用 Kryo 序列化？一般自定义对象，且对象内包含集合，集合中数据还比较多时，如果使用Java 序列化会比较慢，且序列化后对象比较大，所以此时使用 Kryo 序列化可以解决这些问题。

##### ① Scala

```scala
package com.bigdata.scala

import org.apache.spark.storage.StorageLevel
import org.apache.spark.{SparkConf, SparkContext}

/**
  * Kyro高性能序列化框架
  *
  * @author yaocs2
  * @since 2022-09-19
  */
object KryoSerializationScala {

  def main(args: Array[String]): Unit = {
    // 创建sparkContext
    val conf = new SparkConf()
    conf.setAppName("KryoSerializationScala")
      .setMaster("local")
      // 默认使用的是JDK序列化框架 => 163 byte
      // 指定使用Kryo序列化 => 31 byte
      .set("spark.serializer", "org.apache.spark.serializer.KryoSerializer")
      // 注册外部对象
      .registerKryoClasses(Array(classOf[Person]))
    val sc = new SparkContext(conf)

    val dataRDD = sc.parallelize(Array("hello you", "hello me"))
    val wordsRDD = dataRDD.flatMap(_.split(" "))
    val personRDD = wordsRDD.map(word => Person(word, 18)).persist(StorageLevel.MEMORY_ONLY_SER)
    personRDD.foreach(println(_))

    // 查看Spark#Web界面
    while (true) {
      ;
    }
  }

  case class Person(name: String, age: Int) extends Serializable
}
```

##### ② Java

```java
package com.bigdata.java;

import org.apache.spark.SparkConf;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.JavaSparkContext;
import org.apache.spark.api.java.function.FlatMapFunction;
import org.apache.spark.api.java.function.Function;
import org.apache.spark.api.java.function.VoidFunction;
import org.apache.spark.storage.StorageLevel;

import java.io.Serializable;
import java.util.Arrays;
import java.util.Iterator;

/**
 * Kyro高性能序列化框架
 *
 * @author yaocs2
 * @since 2022-09-19
 */
public class KryoSerializationJava {

    public static void main(String[] args) {
        SparkConf conf = new SparkConf();
        conf.setAppName("KryoSerializationJava")
                .setMaster("local")
                // 默认使用的是JDK序列化框架 => 163 byte
                // 指定使用Kryo序列化 => 31 byte
                .set("spark.serializer", "org.apache.spark.serializer.KryoSerializer")
                // 注册外部对象
                .set("spark.kryo.classesToRegister", "com.bigdata.java.Person");
        JavaSparkContext sc = new JavaSparkContext(conf);

        JavaRDD<String> dataRDD = sc.parallelize(Arrays.asList("Hello you", "Hello me"));
        JavaRDD<String> wordsRDD = dataRDD.flatMap(new FlatMapFunction<String, String>() {
            @Override
            public Iterator<String> call(String line) throws Exception {
                return Arrays.asList(line.split(" ")).iterator();
            }
        });
        JavaRDD<Person> personRDD = wordsRDD.map(new Function<String, Person>() {
            @Override
            public Person call(String word) throws Exception {
                return new Person(word, 18);
            }
        }).persist(StorageLevel.MEMORY_ONLY_SER());
        personRDD.foreach(new VoidFunction<Person>() {
            @Override
            public void call(Person person) throws Exception {
                System.out.println(person);
            }
        });

        while (true) {
            // do nothing
        }
    }
}

class Person implements Serializable {
    private String name;
    private int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
```

#### 3）持久化 & checkpoint 调优

1. 针对多次被 transformation 或者 action 操作的 RDD，可以进行持久化操作，避免对一个 RDD 反复计算，也可以再进一步优化，使用序列化的持久化级别（比如开启 Kryo 序列化）。
2. 同时，为了保证 RDD 持久化数据，在可能丢失的情况下，还能实现高可靠，则需要对 RDD 执行 checkpoint 操作。

#### 4）JVM 调优

1. 如果 JVM 内存设置得不合理，则会导致大部分时间都消耗在垃圾回收上。
2. 默认情况下，Spark 使用每个 Executor 60% 的内存空间来缓存 RDD，只有 40% 的内存空间来存放算子执行期间创建的对象，如果执行期间算子创建的对象过大，大于这 40% 的内存空间，则会触发 JVM 的垃圾回收。
3. 如果垃圾回收频繁发生，就需要对这个比例进行调优，通过参数 `spark.storage.memoryFraction` 来修改比例，默认值为 0.6，即代表 Executor 60% 的内存空间来缓存 RDD。
4. 如果 Eden 区空间不足，最直接的解决方式是，`--executor memory` 增加 executor 的内存，这样年轻代和老年代的大小都提高了。

#### 5）并行度调优

##### 1、Task 调优

![1663553820953](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1663553820953.png)

1. 默认并行度，Spark 以文件作为输入源时，会自动设置 RDD的并行度，比如 HDFS，就会依据文件大小，给每一个 block 创建一个 partition，而对于 reduceByKey 等会发生 shuffle 操作的算子，则会使用 父 RDD 最大的并行度作为自己的并行度。
2. 也可通过 `spark.default.parallelism` 指定，这样对于所有算子操作，都只会创建这个数目的 task 线程 ，来处理对应的 RDD 数据。

```shell
# 指定任务名称
--name mySparkJobName
# 指定入口类
--class com.imooc.scala.xxxxx
# 指定集群地址，on yarn模式指定yarn
--master yarn
# client代表yarn-client，cluster代表yarn-cluster
--deploy-mode cluster
# executor进程的内存大小，实际工作中设置2~4G即可
--executor-memory 1G
# 分配多少个executor进程(可分布在不同节点)
--num-executors 2
# 一个executor进程分配多少个cpu core
--executor-cores 2
# driver进程分配多少cpu core，默认为1即可
--driver-cores 1
# driver进程的内存，如果需要使用类似于collect之类的action算子向driver端拉取数据，则可以设置大些
--driver-memory 1G
# 设置job依赖的第三方jar包，建议使用hdfs路径，即不会导致job过大，也可以统一管理依赖jar包的版本
--jars fastjson.jar,abc.jar
# 可以动态指定一些spark任务的参数，指定多个参数可以通过多个--conf来指定，或者在一个--conf后面的双引号中指定多个，多个参数之间用空格隔开即可
--conf "spark.default.parallelism=10"
```

##### 2、Executor 调优

下面两种设置，最终都只会向集群申请 2 个 CPU core，可以并行运行 2 个 Task，但区别有：

1. 多 Executor 模式：

   - 1）每个 Executor 只分配了 1 个 CPU core，每个 Executor JVM 进程只运行 1 个 Task 线程。
   - 2）两个 Executor 可以分配到不同的节点上运行，会使得广播变量为每个节点都拷贝一份。

   ```shell
   --num-executors 2
   --executor-cores 1
   ```

2. 多 core 模式：

   - 1）一个 Executor 分配了 2 个 CPU core，可以在一个 Executor JVM 进程运行多个 Task 线程，复用了进程资源。
   - 2）由于只有一个 Executor 节点，所以广播变量只需要拷贝一份。

   ```shell
   --num-executors 1
   --executor-cores 2
   ```

##### 3、小结

1. 要根据集群情况，来设定Executor 数量。
2. 每一个 Executor 节点设置 2~4 G 内存，分配 2~4 CPU core，因为为 Executor 设置多 CPU core，可以在一个 Executor JVM 上运行多个 Task 线程，复用一定的进程资源，但设置过多又会使得 Task 线程过多，导致内存不够用。
3. Task 线程并行度 = Executor * CPU core * 2~3 ，因为每个 CPU 分配 2~3 个 Task 线程，可以充分利用 CPU 资源，发挥最大价值。 

#### 6）数据本地化

1. 对于 Spark Job 来说，如果数据和计算代码在一起，性能会非常高，而如果将数据和计算代码分开，那么计算节点就需要到数据节点，去取回数据计算，非常损耗性能。而移动代码的开销就比移动数据的小得多，所以 Spark 也正是基于不同的数据本地化级别，来构建 Task 调度算法。

2. 数据本地化，指的是数据离计算代码有多近，存在以下级别：

   | 数据本地化级别 | 作用                                                         |
   | -------------- | ------------------------------------------------------------ |
   | PROCESS_LOCAL  | 进程本地化，性能最好，数据和计算代码在同一个 JVM 进程中      |
   | NODE_LOCAL     | 节点本地化，数据和计算代码在同一个节点，但是不在一个 JVM 进程中，所以数据需要跨进程通信 |
   | NO_PREF        | 中间件本地化，数据从哪里来性能都一样，对于 Task 而言没有区别，比如从数据库中获取数据 |
   | RACK_LOCAL     | 机架本地化，数据和计算代码在同一个机架，数据需要通过网络在节点之间进行传输 |
   | ANY            | 无本地化，数据可能在任意地方，比如其它网络环境、或者其它机架上，性能最差 |

3. Spark 会倾向使用最好的本地化级别去调度 Task，而如果当时没有空闲的 Executor，可以设置一个等待时间，等到某个节点空闲出 Executor，再将 Task 分配过去，但如果超过了等待时间，那么 Spark 会将 Task 分配到其他任意一个空闲 Executor 上执行。

   | 参数                        | 作用                                                         |
   | --------------------------- | ------------------------------------------------------------ |
   | spark.locality.wait         | 所有级别通用，等待指定的时间，默认值为 3 秒                  |
   | spark.locality.wait.process | 默认本地化级别，等待指定的时间，看能否达到数据和计算代码在同一个 JVM 进程中执行 |
   | spark.locality.wait.node    | 等待指定的时间，看能否达到数据和计算代码在同一个节点上执行   |
   | spark.locality.wait.rack    | 等待指定的时间，看能否达到数据和计算代码在同一个机架上执行   |


#### 7）map vs mapPartitions

1. map：一次处理一条数据。
   - 1）优点：不易发生内存溢出，因为每次一条，内存不足时会触发GC。
   - 2）缺点：耗时，建议不要在这里获取数据库连接等操作，因为每一次插入数据，都需要重新获取数据库连接，性能较低。
2. mapPartitions：一次处理一个分区数据。
   - 1）优点：性能较好，常用于初始化链接之类操作。
   - 2）缺点：相对于 map，mapPartitions 更容易发生内存溢出。

##### ① Scala

```scala
package com.bigdata.scala

import org.apache.spark.{SparkConf, SparkContext}

import scala.collection.mutable.ArrayBuffer

/**
  * MapPartitions
  *
  * @author yaocs2
  * @since 2022-09-19
  */
object MapPartitionsOpScala {

  def main(args: Array[String]): Unit = {
    // 创建sparkContext
    val sc = getSparkContext("MapPartitionsOpScala")

    // 设置2个分区
    val dataRDD = sc.parallelize(Array(1, 2, 3, 4, 5), 2)

    /**
      * ====================
      * ====================
      *
      * ====================
      * ====================
      * * ====================
      * sum: 30
      */
    //    val sum = dataRDD.map(item => {
    //      println("====================")
    //      item * 2
    //    }).reduce(_ + _)
    //    println("sum: " + sum)

    /**
      * * ====================
      *
      * * ====================
      *
      * sum: 30
      */
    // Spark代码
    val sum = dataRDD.mapPartitions(it => {
      println("====================")
      val result = new ArrayBuffer[Int]()
        
      // Scala代码
      it.foreach(item => {
        result.+=(item * 2)
      })
      result.iterator
    }).reduce(_ + _)
    println("sum: " + sum)

    // 停止sparkContext
    sc.stop()
  }

  /**
    * 创建sparkContext
    *
    * @return
    */
  private def getSparkContext(appName: String): SparkContext = {
    val conf = new SparkConf()
    conf.setAppName(appName)
      .setMaster("local")
    new SparkContext(conf)
  }
}
```

##### ② Java

```java
package com.bigdata.java;

import org.apache.spark.SparkConf;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.JavaSparkContext;
import org.apache.spark.api.java.function.FlatMapFunction;
import org.apache.spark.api.java.function.Function2;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.Iterator;
import java.util.List;

/**
 * MapPartitions
 *
 * @author yaocs2
 * @since 2022-09-19
 */
public class MapPartitionsOpJava {

    public static void main(String[] args) {
        // 创建sparkContext
        JavaSparkContext sc = getSparkContext("MapPartitionsOpJava");

        // 设置2个分区
        JavaRDD<Integer> dataRDD = sc.parallelize(Arrays.asList(1, 2, 3, 4, 5), 2);

        /**
         * ====================
         *
         * ====================
         *
         * sum: 30
         */
        // Spark代码
        Integer sum = dataRDD.mapPartitions(new FlatMapFunction<Iterator<Integer>, Integer>() {
            @Override
            public Iterator<Integer> call(Iterator<Integer> it) throws Exception {
                // Java代码
                System.out.println("====================");
                List<Integer> list = new ArrayList<>();
                while (it.hasNext()) {
                    list.add(it.next() * 2);
                }
                return list.iterator();
            }
        }).reduce(new Function2<Integer, Integer, Integer>() {
            @Override
            public Integer call(Integer i1, Integer i2) throws Exception {
                return i1 + i2;
            }
        });
        System.out.println("sum: " + sum);

        // 停止sparkContext
        sc.stop();
    }

    /**
     * 创建sparkContext
     *
     * @return
     */
    private static JavaSparkContext getSparkContext(String appName) {
        SparkConf conf = new SparkConf();
        conf.setAppName(appName)
                .setMaster("local");
        return new JavaSparkContext(conf);
    }
}
```

#### 8）foreach vs foreachPartition

与 map vs mapPartitions 不同的是，map、mapPartitions 是 Transformation 算子，foreach、foreachPartition 是 Action 算子。

1. foreach：一次处理一条数据。
2. foreachPartition：一次处理一个分区数据，可以用于往数据库写入数据，性能也不会很差。

##### ① Scala

```scala
package com.bigdata.scala

import org.apache.spark.{SparkConf, SparkContext}

import scala.collection.mutable.ArrayBuffer

/**
  * ForeachPartitions
  *
  * @author yaocs2
  * @since 2022-09-19
  */
object ForeachPartitionsOpScala {

  def main(args: Array[String]): Unit = {
    // 创建sparkContext
    val sc = getSparkContext("MapPartitionsOpScala")

    // 设置2个分区
    val dataRDD = sc.parallelize(Array(1, 2, 3, 4, 5), 2)

    /**
      * * ====================
      * 1
      * 2
      *
      * * ====================
      * 3
      * 4
      * 5
      */
    // Spark代码
    val sum = dataRDD.foreachPartition(it => {
      println("====================")

      // Scala代码
      it.foreach(item => {
        println(item)
      })
    })

    // 停止sparkContext
    sc.stop()
  }

  /**
    * 创建sparkContext
    *
    * @return
    */
  private def getSparkContext(appName: String): SparkContext = {
    val conf = new SparkConf()
    conf.setAppName(appName)
      .setMaster("local")
    new SparkContext(conf)
  }
}
```

##### ② Java

```java
package com.bigdata.java;

import org.apache.spark.SparkConf;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.JavaSparkContext;
import org.apache.spark.api.java.function.VoidFunction;

import java.util.Arrays;
import java.util.Iterator;

/**
 * ForeachPartitions
 *
 * @author yaocs2
 * @since 2022-09-19
 */
public class ForeachPartitionsOpJava {

    public static void main(String[] args) {
        // 创建sparkContext
        JavaSparkContext sc = getSparkContext("MapPartitionsOpJava");

        // 设置2个分区
        JavaRDD<Integer> dataRDD = sc.parallelize(Arrays.asList(1, 2, 3, 4, 5), 2);

        /**
         * ====================
         * 1
         * 2
         *
         * ====================
         * 3
         * 4
         * 5
         *
         */
        // Spark代码
        dataRDD.foreachPartition(new VoidFunction<Iterator<Integer>>() {
            @Override
            public void call(Iterator<Integer> it) throws Exception {
                // Java代码
                System.out.println("====================");
                while (it.hasNext()) {
                    System.out.println(it.next());
                }
            }
        });

        // 停止sparkContext
        sc.stop();
    }

    /**
     * 创建sparkContext
     *
     * @return
     */
    private static JavaSparkContext getSparkContext(String appName) {
        SparkConf conf = new SparkConf();
        conf.setAppName(appName)
                .setMaster("local");
        return new JavaSparkContext(conf);
    }
}
```

#### 9）repartition

repartition，可以对 RDD 进行重分区，调整 Task 并行度，会产生 shuffle，从而实现数据重新分发，解决 RDD 中数据倾斜的问题。

##### ① Scala

```scala
package com.bigdata.scala

import org.apache.spark.{SparkConf, SparkContext}

/**
  * Repartition
  *
  * @author yaocs2
  * @since 2022-09-19
  */
object RepartitionOpScala {

  def main(args: Array[String]): Unit = {
    // 创建sparkContext
    val sc = getSparkContext("RepartitionOpScala")

    // 设置2个分区
    val dataRDD = sc.parallelize(Array(1, 2, 3, 4, 5), 2)

    /**
      * * ==============
      *   4
      * * ==============
      *   1
      *   5
      * * ==============
      *   2
      *   3
      */
    // Repartition重分区, 会产生Shuffle
    dataRDD.repartition(3)
      .foreachPartition(it => {
        println("==============")
        for (elem <- it) {
          println(elem)
        }
      })

    // 停止sparkContext
    sc.stop()
  }

  /**
    * 创建sparkContext
    *
    * @return
    */
  private def getSparkContext(appName: String): SparkContext = {
    val conf = new SparkConf()
    conf.setAppName(appName)
      .setMaster("local")
    new SparkContext(conf)
  }
}
```

##### ② Java

```java
package com.bigdata.java;

import org.apache.spark.SparkConf;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.JavaSparkContext;
import org.apache.spark.api.java.function.VoidFunction;

import java.util.Arrays;
import java.util.Iterator;

/**
 * Repartition
 *
 * @author yaocs2
 * @since 2022-09-19
 */
public class RepartitionOpJava {

    public static void main(String[] args) {
        // 创建sparkContext
        JavaSparkContext sc = getSparkContext("MapPartitionsOpJava");

        // 设置2个分区
        JavaRDD<Integer> dataRDD = sc.parallelize(Arrays.asList(1, 2, 3, 4, 5), 2);

        /**
         ** ====================
         *  4
         ** ====================
         *  1
         *  5
         ** ====================
         *  2
         *  3
         */
        // Repartition重分区, 会产生Shuffle
        dataRDD.repartition(3).foreachPartition(new VoidFunction<Iterator<Integer>>() {
            @Override
            public void call(Iterator<Integer> it) throws Exception {
                System.out.println("==============");
                while (it.hasNext()) {
                    System.out.println(it.next());
                }
            }
        });

        // 停止sparkContext
        sc.stop();
    }

    /**
     * 创建sparkContext
     *
     * @return
     */
    private static JavaSparkContext getSparkContext(String appName) {
        SparkConf conf = new SparkConf();
        conf.setAppName(appName)
                .setMaster("local");
        return new JavaSparkContext(conf);
    }
}
```

#### 10）reduceByKey vs groupByKey

```scala
// 结果都是一致的
val counts = wordCountRDD.reduceByKey(_ + _)
val counts = wordCountRDD.groupByKey().map(wc => (wc.1, wc.2.sum))
```

1. reduceByKey：会提前进行局部聚合，然后再进行 shuffle 操作，由于提前聚合减少了 shuffle 的传输数据量，所以性能更高。
2. groupByKey：不会提前聚合，直接对原数据进行 shuffle 操作，shuffle 完了才进行聚合，由于 shuffle 过程中传输的数据量较大，所以性能也低些。

![1663598733282](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1663598733282.png)

### 2.3. Spark SQL？

1. Hive on spark：实质上是把底层的 MapReduce 引擎，替换成 Spark 引擎。
2. 而 Spark SQL：是 Spark 自己实现的一套 sql 处理引擎，是 spark 的一个模块，主要用于进行结构化数据的处理。
3. 其最核心的结构为：DataFrame = RDD（数据） + Schema（表结构），它和关系型数据库中的表非常类似，DataFrame 可以通过很多来源进行构建，包括结构化的一些数据文件、hive 表、外部的一些关系型数据库、RDD 等，在 Spark 2.0 进行统一后，和 Spark 1.6#dataset 同属一个概念，即 dateframe = dataSet[Row]。

#### 1）DataFrame 算子操作

##### ① Scala

```scala
package com.bigdata.scala.sql

import org.apache.spark.SparkConf
import org.apache.spark.sql.SparkSession

/**
  * Spark SQL DataFrame算子操作
  *
  * @author yaocs2
  * @since 2022-09-19
  */
object DataFrameOpScala {

  def main(args: Array[String]): Unit = {
    // 获取sparkSession
    val conf = new SparkConf()
      .setMaster("local")
    val sparkSession = SparkSession.builder()
      .appName("SqlDemoScala")
      .config(conf)
      .getOrCreate()

    // 读取json文件, 获取DataFrame
    val stuDataFrame = sparkSession.read.json("D:\\Users\\yaocs2\\data\\srcProjects\\bigdata-spark\\src\\main\\resources\\student.json")

    // 查询表结构
    /** root
      * |-- age: long (nullable = true)
      * |-- name: string (nullable = true)
      * |-- sex: string (nullable = true)
      */
    //    stuDataFrame.printSchema()

    // 显示n条数据, 默认显示全部
    /**
      * +---+------+------+
      * |age|name| sex|
      * +---+----+----+
      * | 19|jack|male|
      * | 18| tom|male|
      * +---+----+----+
      * only showing top 2 rows
      */
    //    stuDataFrame.show(2)

    // 查询指定字段
    /**
      * +------+---+
      * +------+---+
      * |  jack| 19|
      * |   tom| 18|
      * |jessic| 27|
      * |  hehe| 18|
      * |  haha| 15|
      * +------+---+
      */
    //    stuDataFrame.select("name", "age").show()

    // 查询指定字段, 但有对字段值进行修改(需要添加隐式转换)
    /**
      * +------+---------+
      * |  name|(age + 1)|
      * +------+---------+
      * |  jack|       20|
      * |   tom|       19|
      * |jessic|       28|
      * |  hehe|       19|
      * |  haha|       16|
      * +------+---------+
      */
    //    import sparkSession.implicits._
    //    stuDataFrame.select($"name", $"age" + 1).show()

    // 根据条件查询指定的字段(需要添加隐式转换)
    /**
      * +---+------+------+
      * |age|  name|   sex|
      * +---+------+------+
      * | 19|  jack|  male|
      * | 27|jessic|female|
      * +---+------+------+
      */
    //    import sparkSession.implicits._
    //    stuDataFrame.filter($"age" > 18).show()

    // where查询(底层调的就是filter, 需要添加隐式转换)
    /**
      * +---+------+------+
      * |age|  name|   sex|
      * +---+------+------+
      * | 19|  jack|  male|
      * | 27|jessic|female|
      * +---+------+------+
      */
    //    import sparkSession.implicits._
    //    stuDataFrame.where($"age" > 18).show()

    // 对数据进行分组求和
    /**
      * +---+-----+
      * |age|count|
      * +---+-----+
      * | 19|    1|
      * | 27|    1|
      * | 18|    2|
      * | 15|    1|
      * +---+-----+
      */
    //    stuDataFrame.groupBy("age").count().show()

    // 关闭sparkSession
    sparkSession.close()
  }
}
```

##### ② Java

```java
package com.bigdata.java.sql;

import org.apache.spark.SparkConf;
import org.apache.spark.sql.Dataset;
import org.apache.spark.sql.Row;
import org.apache.spark.sql.SparkSession;

/**
 * Spark SQL DataFrame算子操作
 *
 * @author yaocs2
 * @since 2022-09-19
 */
public class DataFrameOpJava {

    public static void main(String[] args) {
        SparkConf conf = new SparkConf();
        conf.setMaster("local");

        // 获取sparkSession
        SparkSession sparkSession = SparkSession.builder()
                .appName("SqlDemoJava")
                .config(conf)
                .getOrCreate();

        // 读取json文件, 获取DataFrame
        Dataset<Row> stuDataFrame = sparkSession.read().json("D:\\Users\\yaocs2\\data\\srcProjects\\bigdata-spark\\src\\main\\resources\\student.json");

        // 查询表结构
        /**root
         * |-- age: long (nullable = true)
         * |-- name: string (nullable = true)
         * |-- sex: string (nullable = true)
         */
//        stuDataFrame.printSchema();

        // 显示n条数据, 默认显示全部
        /**
         * +---+------+------+
         * |age|name| sex|
         * +---+----+----+
         * | 19|jack|male|
         * | 18| tom|male|
         * +---+----+----+
         * only showing top 2 rows
         */
//    stuDataFrame.show(2);

        // 查询指定字段
        /**
         * +------+---+
         * +------+---+
         * |  jack| 19|
         * |   tom| 18|
         * |jessic| 27|
         * |  hehe| 18|
         * |  haha| 15|
         * +------+---+
         */
//    stuDataFrame.select("name", "age").show();

        // 查询指定字段, 但有对字段值进行修改
        /**
         * +------+---------+
         * |  name|(age + 1)|
         * +------+---------+
         * |  jack|       20|
         * |   tom|       19|
         * |jessic|       28|
         * |  hehe|       19|
         * |  haha|       16|
         * +------+---------+
         */
//        stuDataFrame.select(
//                org.apache.spark.sql.functions.col("name"),
//                org.apache.spark.sql.functions.col("age").plus(1)
//        ).show();

        // 根据条件查询指定的字段(需要添加隐式转换)
        /**
         * +---+------+------+
         * |age|  name|   sex|
         * +---+------+------+
         * | 19|  jack|  male|
         * | 27|jessic|female|
         * +---+------+------+
         */
//        stuDataFrame.filter(org.apache.spark.sql.functions.col("age").gt(18)).show();

        // where查询(底层调的就是filter, 需要添加隐式转换)
        /**
         * +---+------+------+
         * |age|  name|   sex|
         * +---+------+------+
         * | 19|  jack|  male|
         * | 27|jessic|female|
         * +---+------+------+
         */
        stuDataFrame.where(org.apache.spark.sql.functions.col("age").gt(18)).show();

        // 对数据进行分组求和
        /**
         * +---+-----+
         * |age|count|
         * +---+-----+
         * | 19|    1|
         * | 27|    1|
         * | 18|    2|
         * | 15|    1|
         * +---+-----+
         */
//        stuDataFrame.groupBy("age").count().show();

        // 关闭sparkSession
        sparkSession.close();
    }
}
```

#### 2）DataFrame SQL 操作

##### ① Scala

```scala
package com.bigdata.scala.sql

import org.apache.spark.SparkConf
import org.apache.spark.sql.SparkSession

/**
  * Spark SQL DataFrame SQL操作
  *
  * @author yaocs2
  * @since 2022-09-19
  */
object DataFrameSqlScala {

  def main(args: Array[String]): Unit = {
    // 获取sparkSession
    val conf = new SparkConf()
      .setMaster("local")
    val sparkSession = SparkSession.builder()
      .appName("SqlDemoScala")
      .config(conf)
      .getOrCreate()

    // 读取json文件, 获取DataFrame
    val stuDataFrame = sparkSession.read.json("D:\\Users\\yaocs2\\data\\srcProjects\\bigdata-spark\\src\\main\\resources\\student.json")

    // 先将DataFrame注册为一个临时表
    stuDataFrame.createOrReplaceTempView("student")

    // 使用sql查询student临时表
    /**
      * +---+---+
      * |age|num|
      * +---+---+
      * | 19|  1|
      * | 27|  1|
      * | 18|  2|
      * | 15|  1|
      * +---+---+
      */
    sparkSession.sql(
      """
        | select
        |   age, count(*) as num
        | from student
        | group by age
      """.stripMargin)
        .show()

    // 关闭sparkSession
    sparkSession.close()
  }
}
```

##### ② Java

```java
package com.bigdata.java.sql;

import org.apache.spark.SparkConf;
import org.apache.spark.sql.Dataset;
import org.apache.spark.sql.Row;
import org.apache.spark.sql.SparkSession;

/**
 * Spark SQL DataFrame SQL操作
 *
 * @author yaocs2
 * @since 2022-09-19
 */
public class DataFrameSqlJava {

    public static void main(String[] args) {
        SparkConf conf = new SparkConf();
        conf.setMaster("local");

        // 获取sparkSession
        SparkSession sparkSession = SparkSession.builder()
                .appName("SqlDemoJava")
                .config(conf)
                .getOrCreate();

        // 读取json文件, 获取DataFrame
        Dataset<Row> stuDataFrame = sparkSession.read().json("D:\\Users\\yaocs2\\data\\srcProjects\\bigdata-spark\\src\\main\\resources\\student.json");

        // 先将DataFrame注册为一个临时表
        stuDataFrame.createOrReplaceTempView("student");

        // 使用sql查询student临时表
        /**
         * +---+---+
         * |age|num|
         * +---+---+
         * | 19|  1|
         * | 27|  1|
         * | 18|  2|
         * | 15|  1|
         * +---+---+
         */
        sparkSession.sql("select age, count(*) as num from student group by age").show();

        // 关闭sparkSession
        sparkSession.close();
    }
}
```

#### 3）RDD 互转 DataFrame

1. 实际工作中，往往是先将数据加载进来，然后做一些清洗转换，后续需要进行一定的统计，此时虽然可以使用transformation 算子进行统计分析，但是需要写代码且不太方便。
2. 所以，可以将 RDD 转换为 DataFrame，直接写 SQL 分析即可。
3. Spark SQL 支持两种方式，将 RDD 转换为 DataFrame，分别是反射方式和编程方式。

##### 1、反射方式

1. Scala 具有隐式转换的特性，可以将 Spark SQL 的 Scala 接口，自动把包含了 case class 的 RDD，转换为 DataFrame。
2. 但这种方式的前提条件是，需要提前直到 RDD 中的字段，然后定义好 case class。

###### ① Scala

```scala
package com.bigdata.scala.sql

import org.apache.spark.SparkConf
import org.apache.spark.sql.SparkSession

/**
  * RDD转DataFrame：通过反射方式
  *
  * @author yaocs2
  * @since 2022-09-19
  */
object Rdd2DataFrameByReflectScala {

  def main(args: Array[String]): Unit = {
    // 获取sparkSession
    val conf = new SparkConf()
      .setMaster("local")
    val sparkSession = SparkSession.builder()
      .appName("SqlDemoScala")
      .config(conf)
      .getOrCreate()

    // 获取sparkContext
    val sc = sparkSession.sparkContext

    // 创建RDD
    val dataRDD = sc.parallelize(Array(("jack", 18), ("tom", 20), ("jessic", 30)))

    // 通过case class反射, 将RDD转DataFrame(需要隐式转换)
    import sparkSession.implicits._
    val stuDF = dataRDD.map(tup => Student(tup._1, tup._2)).toDF()

    // 创建student临时表
    stuDF.createOrReplaceTempView("student")

    // 查询student表
    val resDF = sparkSession.sql("select name, age from student where age > 18")

    // DataFrame转RDD
    val resRDD = resDF.rdd

    // 从row中取出数据, 然后封装成student对象
    /**
      * Student(tom,20)
      * Student(jessic,30)
      */
    resRDD.map(row => Student(row.getAs[String]("name"), row.getAs[Int]("age")))
      // 收集好所有的Worker#Student, 再在Driver端遍历
      .collect()
      .foreach(println(_))
    // 等价于
    //    resRDD.map(row => Student(row(0).toString, row(1).toString.toInt))
    //      .foreach(println(_))

    // 关闭sparkSession
    sparkSession.close()
  }
}

// student case class
case class Student(name: String, age: Int)
```

###### ② Java

```java
package com.bigdata.java.sql;

import org.apache.spark.SparkConf;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.JavaSparkContext;
import org.apache.spark.api.java.function.Function;
import org.apache.spark.sql.Dataset;
import org.apache.spark.sql.Row;
import org.apache.spark.sql.SparkSession;
import scala.Tuple2;

import java.util.Arrays;
import java.util.List;

/**
 * RDD转DataFrame：通过反射方式
 *
 * @author yaocs2
 * @since 2022-09-19
 */
public class Rdd2DataFrameByReflectJava {

    public static void main(String[] args) {
        SparkConf conf = new SparkConf();
        conf.setMaster("local");

        // 获取sparkSession
        SparkSession sparkSession = SparkSession.builder()
                .appName("SqlDemoJava")
                .config(conf)
                .getOrCreate();

        // 获取sparkContext
        JavaSparkContext sc = JavaSparkContext.fromSparkContext(sparkSession.sparkContext());

        // 创建RDD
        Tuple2<String, Integer> t1 = new Tuple2<>("jack", 18);
        Tuple2<String, Integer> t2 = new Tuple2<>("tom", 20);
        Tuple2<String, Integer> t3 = new Tuple2<>("jessic", 30);
        JavaRDD<Tuple2<String, Integer>> dataRDD = sc.parallelize(Arrays.asList(t1, t2, t3));

        // 通过case class反射, 将RDD转DataFrame
        JavaRDD<Student> stuRDD = dataRDD.map(new Function<Tuple2<String, Integer>, Student>() {
            @Override
            public Student call(Tuple2<String, Integer> tup) throws Exception {
                return new Student(tup._1, tup._2);
            }
        });

        // Student必须声明为public, 且实现Serializable接口
        Dataset<Row> stuDF = sparkSession.createDataFrame(stuRDD, Student.class);

        // 创建student临时表
        stuDF.createOrReplaceTempView("student");

        // 查询student表
        Dataset<Row> resDF = sparkSession.sql("select name, age from student where age > 18");

        // DataFrame转RDD
        JavaRDD<Row> resRDD = resDF.javaRDD();

        // 从row中取出数据, 然后封装成student对象
        /**
         * Student(tom,20)
         * Student(jessic,30)
         */
        List<Student> studentList = resRDD.map(new Function<Row, Student>() {
            @Override
            public Student call(Row row) throws Exception {
//                return new Student(row.getAs("name"), Integer.parseInt(row.getAs("age")));

                // 等价于
                return new Student(row.getString(0), row.getInt(1));
            }
        }).collect();

        // 收集好所有的Worker#Student, 再在Driver端遍历
        /**
         * Student{name='tom', age=20}
         * Student{name='jessic', age=30}
         */
        for (Student student : studentList) {
            System.out.println(student.toString());
        }

        // 关闭sparkSession
        sparkSession.close();
    }
}

/**
 * Spark SQL Case Class
 *
 * @author yaocs2
 * @since 2022-09-21
 */
public class Student implements Serializable {

    private String name;
    private int age;

    public Student(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "Student{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
```

##### 2、编程方式

当 case class 中的字段，无法提前定义时，就只能通过编程的方式，动态指定元数据了。

###### ① Scala

```scala
package com.bigdata.scala.sql

import org.apache.spark.SparkConf
import org.apache.spark.sql.types.{IntegerType, StringType, StructField, StructType}
import org.apache.spark.sql.{Row, SparkSession}

/**
  * RDD转DataFrame：通过编程方式
  *
  * @author yaocs2
  * @since 2022-09-19
  */
object Rdd2DataFrameByProgramScala {

  def main(args: Array[String]): Unit = {
    // 获取sparkSession
    val conf = new SparkConf()
      .setMaster("local")
    val sparkSession = SparkSession.builder()
      .appName("SqlDemoScala")
      .config(conf)
      .getOrCreate()

    // 获取sparkContext
    val sc = sparkSession.sparkContext

    // 创建RDD
    val dataRDD = sc.parallelize(Array(("jack", 18), ("tom", 20), ("jessic", 30)))

    // 组装rowRDD
    val rowRDD = dataRDD.map(tup => Row(tup._1, tup._2))

    // 指定表结构
    val schema = StructType(Array(
      // 支持动态构建
      StructField("name", StringType, true),
      StructField("age", IntegerType, true)
    ))

    // 组装DataFrame
    val stuDF = sparkSession.createDataFrame(rowRDD, schema)

    // 创建student临时表
    stuDF.createOrReplaceTempView("student")

    // 查询student表
    val resDF = sparkSession.sql("select name, age from student where age > 18")

    // DataFrame转RDD
    val resRDD = resDF.rdd

    // 从row中取出数据, 然后封装成student对象
    /**
      * Student(tom,20)
      * Student(jessic,30)
      */
    resRDD.map(row => Student(row.getAs[String]("name"), row.getAs[Int]("age")))
      // 收集好所有的Worker#Student, 再在Driver端遍历
      .collect()
      .foreach(println(_))
    // 等价于
    //    resRDD.map(row => Student(row(0).toString, row(1).toString.toInt))
    //      .foreach(println(_))

    // 关闭sparkSession
    sparkSession.close()
  }
}
```

###### ② Java

```java
package com.bigdata.java.sql;

import org.apache.spark.SparkConf;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.JavaSparkContext;
import org.apache.spark.api.java.function.Function;
import org.apache.spark.sql.Dataset;
import org.apache.spark.sql.Row;
import org.apache.spark.sql.RowFactory;
import org.apache.spark.sql.SparkSession;
import org.apache.spark.sql.types.DataTypes;
import org.apache.spark.sql.types.StructField;
import org.apache.spark.sql.types.StructType;
import scala.Tuple2;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

/**
 * RDD转DataFrame：通过编程方式
 *
 * @author yaocs2
 * @since 2022-09-19
 */
public class Rdd2DataFrameByProgramJava {

    public static void main(String[] args) {
        SparkConf conf = new SparkConf();
        conf.setMaster("local");

        // 获取sparkSession
        SparkSession sparkSession = SparkSession.builder()
                .appName("SqlDemoJava")
                .config(conf)
                .getOrCreate();

        // 获取sparkContext
        JavaSparkContext sc = JavaSparkContext.fromSparkContext(sparkSession.sparkContext());

        // 创建RDD
        Tuple2<String, Integer> t1 = new Tuple2<>("jack", 18);
        Tuple2<String, Integer> t2 = new Tuple2<>("tom", 20);
        Tuple2<String, Integer> t3 = new Tuple2<>("jessic", 30);
        JavaRDD<Tuple2<String, Integer>> dataRDD = sc.parallelize(Arrays.asList(t1, t2, t3));

        // 通过case class反射, 将RDD转DataFrame
        JavaRDD<Row> rowRDD = dataRDD.map(new Function<Tuple2<String, Integer>, Row>() {
            @Override
            public Row call(Tuple2<String, Integer> tup) throws Exception {
                return RowFactory.create(tup._1, tup._2);
            }
        });

        // 指定表结构(支持动态构建)
        List<StructField> structFieldList = new ArrayList<>();
        structFieldList.add(DataTypes.createStructField("name", DataTypes.StringType, true));
        structFieldList.add(DataTypes.createStructField("age", DataTypes.IntegerType, true));
        StructType schema = DataTypes.createStructType(structFieldList);

        // Student必须声明为public, 且实现Serializable接口
        Dataset<Row> stuDF = sparkSession.createDataFrame(rowRDD, schema);

        // 创建student临时表
        stuDF.createOrReplaceTempView("student");

        // 查询student表
        Dataset<Row> resDF = sparkSession.sql("select name, age from student where age > 18");

        // DataFrame转RDD
        JavaRDD<Row> resRDD = resDF.javaRDD();

        // 从row中取出数据, 然后封装成student对象
        /**
         * Student(tom,20)
         * Student(jessic,30)
         */
        List<Student> studentList = resRDD.map(new Function<Row, Student>() {
            @Override
            public Student call(Row row) throws Exception {
//                return new Student(row.getAs("name"), Integer.parseInt(row.getAs("age")));

                // 等价于
                return new Student(row.getString(0), row.getInt(1));
            }
        }).collect();

        // 收集好所有的Worker#Student, 再在Driver端遍历
        /**
         * Student{name='tom', age=20}
         * Student{name='jessic', age=30}
         */
        for (Student student : studentList) {
            System.out.println(student.toString());
        }

        // 关闭sparkSession
        sparkSession.close();
    }
}
```

#### 4）DataFrame load & save 操作

1. load 操作，主要用于加载数据源数据，创建 DataFrame。
2. save 操作，主要用于将 DataFrame 中的数据，保存到其他数据源中。
3. 这两种操作，如果不指定 format，则默认操作是 `parquet 列式存储格式` ，也可以配置其他格式：`json、parquet、jdbc、orc、csv、text`。

| SaveMode               | 作用                                                         |
| ---------------------- | ------------------------------------------------------------ |
| SaveMode.ErrorIfExists | 默认，如果目标位置已经存在数据，则抛出一个异常               |
| SaveMode.Append        | 如果目标位置已经存在数据，则将数据追加进去                   |
| SaveMode.OverWrite     | 如果目标位置已经存在数据，则将已经存在的数据删除，用新数据进行覆盖 |
| SaveMode.Ignore        | 如果目标位置已经存在数据，则忽略，不做任何操作               |

##### ① Scala

```scala
package com.bigdata.scala.sql

import org.apache.spark.SparkConf
import org.apache.spark.sql.SparkSession

/**
  * DataFrame load & save 操作
  *
  * @author yaocs2
  * @since 2022-09-21
  */
object LoadAndSaveOpScala {

  def main(args: Array[String]): Unit = {
    // 获取sparkSession
    val conf = new SparkConf()
      .setMaster("local")
    val sparkSession = SparkSession.builder()
      .appName("SqlDemoScala")
      .config(conf)
      .getOrCreate()

    // load读取json文件, 获取DataFrame
    val stuDF = sparkSession.read.format("json").load("D:\\Users\\yaocs2\\data\\srcProjects\\bigdata-spark\\src\\main\\resources\\student.json")

    // save写数据到文件
    stuDF.select("name", "age")
      .write
      .format("csv")
      // 默认不支持本地, 要对接HDFS
      // jack,19
      // tom,18
      // jessic,27
      // hehe,18
      // haha,15
      .mode(SaveMode.Append)
      .save("hdfs://bigdata01:9000/out-save001")

    // 关闭sparkSession
    sparkSession.close()
  }
}
```

##### ② Java

```java
package com.bigdata.java.sql;

import org.apache.spark.SparkConf;
import org.apache.spark.sql.Dataset;
import org.apache.spark.sql.Row;
import org.apache.spark.sql.SparkSession;

/**
 * DataFrame load & save 操作
 *
 * @author yaocs2
 * @since 2022-09-21
 */
public class LoadAndSaveOpJava {

    public static void main(String[] args) {
        SparkConf conf = new SparkConf();
        conf.setMaster("local");

        // 获取sparkSession
        SparkSession sparkSession = SparkSession.builder()
                .appName("SqlDemoJava")
                .config(conf)
                .getOrCreate();

        // load读取json文件, 获取DataFrame
        Dataset<Row> stuDF = sparkSession.read().format("json").load("D:\\Users\\yaocs2\\data\\srcProjects\\bigdata-spark\\src\\main\\resources\\student.json");

        // save写数据到文件
        stuDF.select("name", "age")
                .write()
                .format("csv")
                // 默认不支持本地, 要对接HDFS
                // jack,19
                // tom,18
                // jessic,27
                // hehe,18
                // haha,15
            	.mode(SaveMode.Append)
                .save("hdfs://bigdata01:9000/out-save002");

        // 关闭sparkSession
        sparkSession.close();
    }
}
```

#### 5）内置函数

| 分类          | 函数                                                         |
| ------------- | ------------------------------------------------------------ |
| 聚合函数      | avg, count, counDistinct, first, last, max, mean, min, sum, sumDistinct |
| 集合函数      | array_contains, explode, size                                |
| 日期/时间函数 | datediff, data_add, date_sub. add_months, last_day, next_day, months_between |
| 数学函数      | abs, ceil, floor（向上取整）, round（向下取整）              |
| 混合函数      | if, isnull, md5, not, rand, when                             |
| 字符串函数    | concat, get_json_objext                                      |
| 窗口函数      | denseRank, rank, rowNumber                                   |

##### 1、UDF 自定义函数

```scala
package com.imooc.scala

import org.apache.spark.sql.SparkSession
import org.apache.spark.SparkConf
import org.apache.spark.sql.functions.udf

object UserDefinedFunction {
  def main(args: Array[String]): Unit = {
    // 配置SparkSession
    val conf = new SparkConf().setMaster("local")
    val sparkSession = SparkSession.builder()
      .appName("UserDefinedFunction").config(conf).getOrCreate()

    val userDf = sparkSession.read.format("csv").load("D:\\bigdata\\spark\\user.data")

    // 将userDf注册为一个临时表
    userDf.createOrReplaceTempView("user")

    // 生成user defined function
    val up_first = udf((str:String) => str.substring(0,1).toUpperCase() + str.substring(1))
    sparkSession.udf.register("up_first", up_first)

    // 使用SparkSQL查询临时表中的数据
    sparkSession.sql("select up_first(name), age from user").show()

    // 停止sparkSession
    sparkSession.stop()
  }
}
```

### 2.4. Spark SQL 综合案例？

#### 1）取 Top3

```sql
-- 区域 | UID:金币,UID:金币
select
    t4.area, concat_ws(",", collect_list(t4.it)) as top_n
from (
    -- 区域 | UID:金币
    select
        t3.area, concat(t3.uid, ":", t3.gold_sum) as it
    from (
        -- UID | 区域 | 金币 | 排名
        select
            t2.uid, t2.area, t2.gold_sum,
            row_number() over(partition by area order by gold_sum desc) as idx
        from (
            -- UID | 区域 | 对于所有视频，每个主播收到的所有金币
            select
                t1.uid, first(t1.area) as area, cast(sum(t1.gold_sum) as int) as gold_sum
            from (
                -- UID | 区域 | 对于每个视频，每个主播收到的所有金币
                select
                    vi.uid, vi.area, gr.gold_sum
                from 
                    video_info as vi
                join (
                    select
                        vid, sum(gold) as gold_sum
                    from 
                        gift_record
                    group by vid
                ) as gr
                on vi.vid = gr.vid
            ) as t1 group by t1.uid
        ) as t2
    ) as t3 where t3.idx <= 3
) as t4
group by t4.area
```

### 2.5. Spark SQL vs Hive？

|          | Spark                                                        | Hive                                                         |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 原理     | Hive 出现目的是为了少写 MR 代码，减少重复劳动而出现的，本质是一个翻译器，将 SQL 语言翻译成一连串的 MR 任务，再提交到 Hadoop 上执行，所以实际上 Hive 是不干活的 | Spark SQL 是基于 Spark RDD 实现的，不仅是一个翻译器，还是个内存型的 MapReduce |
| 应用场景 | Hive 基于MapReduce，适合大量数据的离线分析                   | SparkSQL 基于内存的 MapReduce，适合做实时数据分析，不太适合大量数据的分析 |

### 2.6. Spark SQL 集成 Hive？

Spark SQL 集成 Hive，指的是在 Spark SQL 中操作 Hive 表，底层走的不再是 MapReduce 引擎，而是读取 Hive 中的元数据，以及存储在 HDFS 的数据，通过 Spark 引擎进行计算，从而提高计算效率，以及省去了在 Spark SQL 中建立临时表的操作。

#### 1）安装部署

```shell
[root@bigdata04 conf]# pwd
/data/soft/spark-3.2.1-bin-hadoop3.2/conf

# 新增hive-site.xml配置
[root@bigdata04 conf]# vim hive-site.xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
  <!-- 指定Hive默认在HDFS中的数据存储目录 -->
  <property>
    <name>hive.metastore.warehouse.dir</name>
    <value>/user/hive/warehouse</value>
    <description>location of default database for the warehouse</description>
  </property>
  <!-- 指定Hive的MetaStore（MySQL的URL信息） -->
  <property>
    <name>javax.jdo.option.ConnectionURL</name>
    <value>jdbc:mysql://192.168.56.1:3306/hive?serverTimezone=Asia/Shanghai</value>
  </property> 
  <!-- 指定MySQL的驱动类名(5.6驱动) -->  
  <property>
    <name>javax.jdo.option.ConnectionDriverName</name>
    <value>com.mysql.jdbc.Driver</value>
  </property>
  <!-- 指定MySQL的用户名 -->
  <property>
    <name>javax.jdo.option.ConnectionUserName</name>
    <value>root</value>
  </property>
  <!-- 指定MySQL的密码 -->
  <property>
    <name>javax.jdo.option.ConnectionPassword</name>
    <value>root</value>
  </property>
</configuration>

# 拷贝mysql驱动包
[root@bigdata04 conf]# cp /data/soft/apache-hive-3.1.2-bin/lib/mysql-connector-java-5.1.41.jar ../jars/
```

#### 2）命令行模式

```shell
[root@bigdata04 conf]# spark-sql-3 --master yarn
hive.cli.print.current.db       true
hive.cli.print.header   true
Spark master: yarn, Application Id: application_1663816112760_0007
spark-sql> 

spark-sql> insert into t1(id, name) values (123456, "zs");
Time taken: 4.518 seconds

spark-sql> select * from t1;
1       zs
123456  zs
Time taken: 1.141 seconds, Fetched 2 row(s)
```

#### 3）代码模式

```xml
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-hive_2.12</artifactId>
            <version>3.2.1</version>
            <!--<scope>provided</scope>-->
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.41</version>
            <!--<scope>provided</scope>-->
        </dependency>
```

```scala
package com.bigdata.scala.sql

import org.apache.spark.SparkConf
import org.apache.spark.sql.SparkSession

/**
  * Spark 操作 Hive
  *
  * @author yaocs2
  * @since 2022-09-22
  */
object SparkSqlReadHiveScala {

  def main(args: Array[String]): Unit = {
    // 获取sparkSession
    val conf = new SparkConf()
    conf.setMaster("local")
    val sparkSession = SparkSession.builder()
      .appName("SparkSqlReadHiveScala")
      .config(conf)
      // 开启对Hive的支持
      .enableHiveSupport()
      .getOrCreate()

    // 执行sql查询
    /**
      * +------+----+
      * |    id|name|
      * +------+----+
      * |     1|  zs|
      * |123456|  zs|
      * +------+----+
      */
    sparkSession.sql("select * from t1").show()

    // 停止sparkSession
    sparkSession.stop()
  }
}
```

#### 4）写数据到 Hive 表

##### 1、insertInto()

```scala
package com.bigdata.scala.sql

import org.apache.spark.SparkConf
import org.apache.spark.sql.SparkSession

/**
  * Spark 写数据到 Hive 表
  *
  * @author yaocs2
  * @since 2022-09-22
  */
object SparkInsertIntoHiveScala {

  def main(args: Array[String]): Unit = {
    System.setProperty("hadoop.home.dir","D:\\MyData\\yaocs2\\tools\\hadoop-3.2.0")

    // 获取sparkSession
    val conf = new SparkConf()
    conf.setMaster("local")
    val sparkSession = SparkSession.builder()
      .appName("SparkSqlReadHiveScala")
      .config(conf)
      // 开启对Hive的支持
      .enableHiveSupport()
      .getOrCreate()

    /**
      * 官方建议提前在Hive中创建表，在SparkSQL中直接使用
      * 注意：通过SparkSQL创建Hive表的时候，如果想要指定存储格式等参数(默认是TextFile)，则必须要使用using hive。
      * 这样指定是无效的：create table t1(id int) OPTIONS(fileFormat 'parquet')
      * 这样才是有效的：create table t1(id int) using hive OPTIONS(fileFormat 'parquet')
      * create table t1(id int) 等于 create table t1(id int) using hive OPTIONS(fileFormat 'textfile')
      *
      * fileFormat支持6种参数：'sequencefile', 'rcfile', 'orc', 'parquet', 'textfile' and 'avro
      */
    sparkSession.sql(
      """
        |create table if not exists student_score_bak(
        | id int,
        | name string,
        | sub string,
        | score int
        |)using hive
        | OPTIONS(
        |   fileFormat 'textfile',
        |   fieldDelim ','
        |   )
        |""".stripMargin)

    // 执行sql查询
    val resDF = sparkSession.sql("select * from student_score")

    // insert into
    /**
      * spark-sql> select * from student_score_bak;
      * 1       zs1     chinese 80
      * 2       zs1     math    90
      * 3       zs1     english 89
      * 4       zs2     chinese 60
      * 5       zs2     math    75
      * 6       zs2     english 80
      * 7       zs3     chinese 79
      * 8       zs3     math    83
      * 9       zs3     english 72
      * 10      zs4     chinese 90
      * 11      zs4     math    76
      * 12      zs4     english 80
      * 13      zs5     chinese 98
      * 14      zs5     math    80
      * 15      zs5     english 70
      * Time taken: 0.313 seconds, Fetched 15 row(s)
      */
    resDF.write
      //指定数据写入格式append：追加。overwrite：覆盖。
      .mode("overwrite")
      //这里需要指定数据格式：parquet, orc, avro(需要添加外部依赖), json, csv, text。不指定的话默认是parquet格式。
      //针对insertInto而言，因为它是向一个已存在的表中写入数据，所以format参数和option参数会被忽略。
      //.format("text")
      .insertInto("student_score_bak")

    // 停止sparkSession
    sparkSession.stop()
  }
}
```

##### 2、saveAsTable()

```scala
package com.bigdata.scala.sql

import org.apache.spark.SparkConf
import org.apache.spark.sql.SparkSession

/**
  * Spark 写数据到 Hive 表
  *
  * @author yaocs2
  * @since 2022-09-22
  */
object SparkSaveAsTableHiveScala {

  def main(args: Array[String]): Unit = {
    System.setProperty("hadoop.home.dir", "D:\\MyData\\yaocs2\\tools\\hadoop-3.2.0")

    // 获取sparkSession
    val conf = new SparkConf()
    conf.setMaster("local")
    val sparkSession = SparkSession.builder()
      .appName("SparkSaveAsTableHiveScala")
      .config(conf)
      // 开启对Hive的支持
      .config("spark.sql.warehouse.dir", "hdfs://bigdata01:9000/user/hive/warehouse")
      .enableHiveSupport()
      .getOrCreate()

    // 查询数据
    import sparkSession.sql
    val resDf = sql("select * from student_score")

    //2：saveAsTable()
    /**
      * 分为两种情况，表不存在和表存在
      * 1：表不存在，则会根据DataFrame中的Schema自动创建目标表并写入数据
      * 2：表存在
      *  2.1：如果mode=append，当DataFrame中的Schema和表中的Schema相同(字段顺序可以不同)，则执行追加操作。
      * 当DataFrame中的Schema和表中的Schema不相同，则报错。
      *  2.2：如果mode=overwrite，当DataFrame中的Schema和表中的Schema相同(字段顺序可以不同)，则直接覆盖。
      * 当DataFrame中的Schema和表中的Schema不相同，则会删除之前的表，然后按照DataFrame中的Schema重新创建表并写入数据。
      */

    //表不存在
    /**
      * hive (default)> select * from student_score_bak;
      * OK
      * student_score_bak.id    student_score_bak.name  student_score_bak.sub   student_score_bak.score
      * 1       zs1     chinese 80
      * 2       zs1     math    90
      * 3       zs1     english 89
      * 4       zs2     chinese 60
      * 5       zs2     math    75
      * 6       zs2     english 80
      * 7       zs3     chinese 79
      * 8       zs3     math    83
      * 9       zs3     english 72
      * 10      zs4     chinese 90
      * 11      zs4     math    76
      * 12      zs4     english 80
      * 13      zs5     chinese 98
      * 14      zs5     math    80
      * 15      zs5     english 70
      * Time taken: 0.169 seconds, Fetched: 15 row(s)
      */
    //    resDf.write
    //      //指定数据写入格式append：追加。overwrite：覆盖。
    //      .mode("overwrite")
    //      //这里需要指定数据格式：parquet, orc, avro(需要添加外部依赖), json, csv, text。不指定的话默认是parquet格式。
    //      //注意：text数据格式在这里不支持int数据类型
    //      //针对普通文本文件数据格式(json、csv)，默认创建的Hive表示SequenceFile格式的，无法读取生成的普通文件，不要使用这种格式。
    //      //parquet、orc数据格式可以正常使用
    //      .format("parquet")
    //      .saveAsTable("student_score_bak")

    //表存在
    /**
      * hive (default)> select * from student_score_bak;
      * OK
      * student_score_bak.id    student_score_bak.name  student_score_bak.sub   student_score_bak.score
      * 1       zs1     chinese 80
      * 2       zs1     math    90
      * 3       zs1     english 89
      * 4       zs2     chinese 60
      * 5       zs2     math    75
      * 6       zs2     english 80
      * 7       zs3     chinese 79
      * 8       zs3     math    83
      * 9       zs3     english 72
      * 10      zs4     chinese 90
      * 11      zs4     math    76
      * 12      zs4     english 80
      * 13      zs5     chinese 98
      * 14      zs5     math    80
      * 15      zs5     english 70
      * Time taken: 0.207 seconds, Fetched: 15 row(s)
      */
    sql(
      """
        |create table if not exists student_score_bak(
        | id int,
        | name string,
        | sub string,
        | score int
        |)using hive
        | OPTIONS(
        |   fileFormat 'parquet'
        |   )
        |""".stripMargin)

    resDf.write
      //指定数据写入格式append：追加。overwrite：覆盖。
      .mode("overwrite")
      //这里需要指定数据格式：parquet, orc, avro(需要添加外部依赖), json, csv, text。不指定的话默认是parquet格式。
      //注意：text数据格式在这里不支持int数据类型
      //针对已存在的表，当mode为append时，这里必须指定为hive。
      //针对已存在的表，当mode为overwrite时：
      //  这里如果指定为hive，则会生成默认数据存储格式(TextFile)的Hive表
      //  这里如果指定为普通文本文件数据格式(json、csv)，默认创建的Hive表是SequenceFile格式的，无法读取生成的普通文件，不需要使用这种方式
      //parquet、orc数据格式可以正常使用
      .format("parquet")
      .saveAsTable("student_score_bak")

    /**
      * 所以, 无论是表存不存在, 统一用以下格式就好：
      * resDf.write
      * .mode("overwrite")
      * .format("parquet")
      * .saveAsTable("student_score_bak")
      */
    // 停止sparkSession
    sparkSession.stop()
  }
}
```

##### 3、Spark SQL | 常用

```scala
package com.bigdata.scala.sql

import org.apache.spark.SparkConf
import org.apache.spark.sql.SparkSession

/**
  * Spark 写数据到 Hive 表
  *
  * @author yaocs2
  * @since 2022-09-22
  */
object SparkSqlWriteHiveScala {

  def main(args: Array[String]): Unit = {
    System.setProperty("hadoop.home.dir", "D:\\MyData\\yaocs2\\tools\\hadoop-3.2.0")

    // 获取sparkSession
    val conf = new SparkConf()
    conf.setMaster("local")
    val sparkSession = SparkSession.builder()
      .appName("SparkSaveAsTableHiveScala")
      .config(conf)
      // 开启对Hive的支持
      .config("spark.sql.warehouse.dir", "hdfs://bigdata01:9000/user/hive/warehouse")
      .enableHiveSupport()
      .getOrCreate()

    //3：SparkSQL语句
    /**
      * 分为两种情况：表不存在和表存在
      * 1：表不存在，使用create table as select
      * 2：表存在，使用insert into select
      */
    //表不存在
    /**
      * hive (default)> select * from student_score_bak;
      * OK
      * student_score_bak.id    student_score_bak.name  student_score_bak.sub   student_score_bak.score
      * 1       zs1     chinese 80
      * 2       zs1     math    90
      * 3       zs1     english 89
      * 4       zs2     chinese 60
      * 5       zs2     math    75
      * 6       zs2     english 80
      * 7       zs3     chinese 79
      * 8       zs3     math    83
      * 9       zs3     english 72
      * 10      zs4     chinese 90
      * 11      zs4     math    76
      * 12      zs4     english 80
      * 13      zs5     chinese 98
      * 14      zs5     math    80
      * 15      zs5     english 70
      * Time taken: 0.155 seconds, Fetched: 15 row(s)
      */
    import sparkSession.sql
    //    sql(
    //      """
    //        |create table student_score_bak
    //        |as
    //        |select * from student_score
    //        |""".stripMargin)

    //表存在
    /**
      * hive (default)> select * from student_score_bak;
      * OK
      * student_score_bak.id    student_score_bak.name  student_score_bak.sub   student_score_bak.score
      * 1       zs1     chinese 80
      * 2       zs1     math    90
      * 3       zs1     english 89
      * 4       zs2     chinese 60
      * 5       zs2     math    75
      * 6       zs2     english 80
      * 7       zs3     chinese 79
      * 8       zs3     math    83
      * 9       zs3     english 72
      * 10      zs4     chinese 90
      * 11      zs4     math    76
      * 12      zs4     english 80
      * 13      zs5     chinese 98
      * 14      zs5     math    80
      * 15      zs5     english 70
      * Time taken: 0.185 seconds, Fetched: 15 row(s)
      */
    sql(
      """
        |create table if not exists student_score_bak(
        | id int,
        | name string,
        | sub string,
        | score int
        |)using hive
        | OPTIONS(
        |   fileFormat 'textfile',
        |   fieldDelim ','
        |   )
        |""".stripMargin)

    sql(
      """
        |insert into student_score_bak
        |select * from student_score
        |""".stripMargin)

    // 停止sparkSession
    sparkSession.stop()
  }
}
```

### 2.7. Spark 3.0？

Spark 3.0.0 是 3.x 系列的第一个正式版本，于 2020-06-10 正式发布，Spark 3.0 性能大约是 Spark 2.4 的 2 倍，RDD 计算没多大变化，重点是对 Spark SQL 进行了优化。

| Spark 3.x 新特性                          | 解释                                                         |
| ----------------------------------------- | ------------------------------------------------------------ |
| Adaptive Query Execution，简称 AQE        | 自适应查询执行                                               |
| Dynamic Partition Pruning，简称 DPP       | 动态分区裁剪                                                 |
| Accelerator-aware Scheduling              | 加速器感知调度，支持自动感知、调度集群中的 GPU 资源          |
| Catalog plugin API                        | Catalog 插件 API，支持修改外部表的结构                       |
| 支持 Hadoop 3.x  / Java 11 / Scala 2.12   | Spark 3.x 建议使用 Hadoop 3.x、JDK 8、Scala 2.12             |
| Better ANSI SQL compatibility             | 更好的 ANSI SQL 兼容性，用于解决 Spark SQL 和 PostgreSQL 之间的差异 |
| Redesigned pandas UDF API with type hints | PySpark#Pandas API 重大改进                                  |
| Structured Streaming UI                   | 用于流计算的新 UI                                            |

#### 1）安装部署

```shell
# 行末添加下面配置
export JAVA_HOME=/data/soft/jdk1.8
export HADOOP_CONF_DIR=/data/soft/hadoop-3.2.0/etc/hadooptar -zxvf spark-3.2.1-bin-hadoop3.2.tgz

[root@bigdata04 conf]# pwd
/data/soft/spark-3.2.1-bin-hadoop3.2/conf
[root@bigdata04 conf]# mv spark-env.sh.template spark-env.sh
[root@bigdata04 conf]# vim spark-env.sh 
# 行末添加下面配置
export JAVA_HOME=/data/soft/jdk1.8
export HADOOP_CONF_DIR=/data/soft/hadoop-3.2.0/etc/hadoop

# 兼容spark2.x与spark3.x
[root@bigdata04 sparkjars]# mv /data/soft/spark-3.2.1-bin-hadoop3.2/bin/spark-submit /data/soft/spark-3.2.1-bin-hadoop3.2/bin/spark-submit-3

# 兼容spark2.x与spark3.x
[root@bigdata04 bin]# vim spark-submit-3
export SPARK_HOME=/data/soft/spark-3.2.1-bin-hadoop3.2

# 兼容spark2.x与spark3.x
[root@bigdata04 sparkjars]# vim /etc/profile
...
export SPARK3_HOME=/data/soft/spark-3.2.1-bin-hadoop3.2
export PATH=.:$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$HIVE_HOME/bin:$SPARK_HOME/bin:$SPARK3_HOME/bin:$PATH
[root@bigdata04 sparkjars]# source /etc/profile

# 解决lzo依赖问题
[root@bigdata04 conf]# pwd
/data/soft/spark-3.2.1-bin-hadoop3.2/conf
[root@bigdata04 conf]# mv spark-defaults.conf.template spark-defaults.conf
[root@bigdata04 conf]# vim spark-defaults.conf
spark.jars=/data/soft/hadoop-3.2.0/share/hadoop/common/hadoop-lzo-0.4.21-SNAPSHOT.jar

# 配置spark3的histroy-server
[root@bigdata04 conf]# vim spark-defaults.conf
spark.eventLog.enabled=true
spark.eventLog.compress=true
spark.eventLog.dir=hdfs://bigdata01:9000/tmp/logs/root/logs
spark.history.fs.logDirectory=hdfs://bigdata01:9000/tmp/logs/root/logs
spark.yarn.historyServer.address=http://bigdata04:18080

# 启动spark2的histroy-server（与spark3共用，但spark3提交任务会读取spark3的配置）
[root@bigdata04 sparkjars]# /data/soft/spark-2.4.3-bin-hadoop2.7/sbin/start-history-server.sh 
starting org.apache.spark.deploy.history.HistoryServer, logging to /data/soft/spark-2.4.3-bin-hadoop2.7/logs/spark-root-org.apache.spark.deploy.history.HistoryServer-1-bigdata04.out
```

#### 2）提交 Spark 3.0 代码

 ```shell
# 可与spark2.x任务，提交到同一个集群执行
spark-submit-3 \
--class com.bigdata.scala.rdd.WordCountScala \
--master yarn \
--deploy-mode cluster \
--executor-memory 1G \
--num-executors 1 \
bigdata_spark3-1.0-SNAPSHOT-jar-with-dependencies.jar \
hdfs://bigdata01:9000/hello.txt
 ```

#### 3）AQE

AQE，Adaptive Query Execution，自适应查询执行，是对 Spark 执行计划的优化，可以基于任务运行时统计的数据指标，动态修改 Spark 的执行计划，主要实现了 3 个功能：

##### 1、自适应调整 Shuffle 分区数量

未合并前：

![1663771350974](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1663771350974.png)

自动合并后：

![1663771335718](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1663771335718.png)

| 核心参数                                        | 默认值 | 作用                                             |
| ----------------------------------------------- | ------ | ------------------------------------------------ |
| spark.sql.adaptive.enabled                      | true   | 是否开启 AQE 机制                                |
| spark.sql.adaptive.coalescePartitions.enabled   | true   | 是否开启 AQE 中的自适应调整 Shuffle 分区数量机制 |
| spark.sql.adaptive.advisoryPartitionSizeInBytes | 64 MB  | 建议的 Shuffle 分区数量大小                      |

```scala
package com.bigdata.scala.sql

import org.apache.spark.SparkConf
import org.apache.spark.sql.SparkSession

/**
  * AQE自适应分区数量
  *
  * @author yaocs2
  * @since 2022-09-21
  */
object AQECoalescePartitionsScala {

  def main(args: Array[String]): Unit = {
    // 获取sparkSession
    val conf = new SparkConf()
    conf.setMaster("local")
    val sparkSession = SparkSession.builder()
      .appName("AQECoalescePartitionsScala")
      .config(conf)
      // 手工指定分区数量, 默认是200
      //      .config("spark.sql.shuffle.partitions", "10")
      // 关闭AQE机制
      //      .config("spark.sql.adaptive.enabled", "false")
      // 关闭AQE自适应分区数量机制
      //      .config("spark.sql.adaptive.coalescePartitions.enabled", "false")
      .getOrCreate()

    /**
      * false
      * false
      * 67108864b
      *
      * true
      * true
      * 67108864b
      */
    println("#####################################")
    println(sparkSession.conf.get("spark.sql.adaptive.enabled"))
    println(sparkSession.conf.get("spark.sql.adaptive.coalescePartitions.enabled"))
    println(sparkSession.conf.get("spark.sql.adaptive.advisoryPartitionSizeInBytes"))
    println("#####################################")

    // 读取数据
    val jsonDF = sparkSession.read.json("D:\\spark_json_1.dat")

    // 创建临时表
    jsonDF.createOrReplaceTempView("t1")

    // 查询t1表(多shuffle, 方便验证效果)
    /**
      * 1、关闭AQECoalescePartitions, 默认分区数量：200个分区, 每个分区393.3 KiB/28455条记录, 总花费2.5min
      * 2、关闭AQECoalescePartitions, 手工指定分区数量：10个分区, 每个分区7.6 MiB/567513条记录, 总花费1.6min
      * 3、开启AQECoalescePartitions, 自适应分区数量：2个分区, 每个分区61 MiB/4518332条记录, 总花费16s
      */
    val sql =
      """
        |select id, uid, lat, lnt, hots, title, status, topicId, end_time, watch_num, start_time, timestamp, area
        |from t1
        |group by id, uid, lat, lnt, hots, title, status, topicId, end_time, watch_num, start_time, timestamp, area
      """.stripMargin
    sparkSession.sql(sql).write.format("json").save("hdfs://bigdata01:9000/aqe/coalesce_partitions_" + System.currentTimeMillis())

    // 查询spark界面
    while (true) {
      ;
    }
  }
}
```

##### 2、动态调整 Join 策略

BroadcastHashJoin 性能最好，因为可以在 Map 阶段把表广播到其他节点，提前在 Map 阶段 Join 好结果，无需  Reduce Join，减少了 Shuffle 的数量。

开启 AQE 后，可以自动根据表数据量，自动调整到适应的 Join 策略：

![1663774592351](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1663774592351.png)

| 核心参数                                      | 默认值 | 作用                                                         |
| --------------------------------------------- | ------ | ------------------------------------------------------------ |
| spark.sql.adaptive.autoBroadcastJoinThreshold | none   | 设置允许广播的表的最大值，设置为 -1 表示禁用，如果未设置会参考 spark.sql.autoBroadcastJoinThreshold 的值，默认为 10 MB |

```scala
package com.bigdata.scala.sql

import org.apache.spark.SparkConf
import org.apache.spark.sql.SparkSession

/**
  * AQE动态调整Join策略
  *
  * @author yaocs2
  * @since 2022-09-21
  */
object AQEJoinScala {

  def main(args: Array[String]): Unit = {
    // 获取sparkSession
    val conf = new SparkConf()
    conf.setMaster("local")
    val sparkSession = SparkSession.builder()
      .appName("AQEJoinScala")
      .config(conf)
      // 关闭AQE机制
      //      .config("spark.sql.adaptive.enabled", "false")
      .getOrCreate()

    // 读取数据
    val t1JsonDF = sparkSession.read.json("D:\\spark_json_t1.dat")
    val t2JsonDF = sparkSession.read.json("D:\\spark_json_t2.dat")

    // 创建临时表
    t1JsonDF.createOrReplaceTempView("t1")
    t2JsonDF.createOrReplaceTempView("t2")

    // 查询t1表
    /**
      * 1、关闭AQE动态调整Join策略：SortMergeJoin, 花费22s
      * 2、开启AQE动态调整Join策略: SortMergeJoin => BroadcastHashJoin, 花费9s
      */
    val sql =
      """
        |select
        | t1.id,
        | t2.name,
        | t1.score
        |from t1 join t2 on (t1.id = t2.id and t2.city like 'bj')
        |""".stripMargin
    sparkSession.sql(sql).write.format("json").save("hdfs://bigdata01:9000/aqe/join_" + System.currentTimeMillis())

    // 查询spark界面
    while (true) {
      ;
    }
  }
}
```

##### 3、动态优化倾斜 Join

自动检测倾斜严重的分区，然后把该分区进一步拆分，最后再关联拆分后的子分区：

![1663775706616](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1663775706616.png)

![1663775755416](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1663775755416.png)

| 核心参数                                                     | 默认值 | 作用                                                         |
| ------------------------------------------------------------ | ------ | ------------------------------------------------------------ |
| spark.sql.adaptive.skewJoin.enabled                          | true   | 是否开启 AQE 机制中的动态优化倾斜 Join 机制                  |
| spark.sql.adaptive.skewJoin.skewedPartitionFactor            | 5      | 分区数据倾斜的判断因子（必须满足倾斜因子和倾斜阈值），如果分区大小 > 倾斜因子 * 分区大小中位数，则认为该分区可能出现了倾斜 |
| spark.sql.adaptive.skewJoin.skewedPartitionThresholdInBytes（要大于 spark.sql.adaptive.advisoryPartitionSizeInBytes AQE 建议分区大小） | 256 MB | 分区数据倾斜的判断阈值（必须满足倾斜因子和倾斜阈值），如果分区大小 > 倾斜阈值，则认为该分区可能出现了倾斜 |

```scala
package com.bigdata.scala.sql

import org.apache.spark.SparkConf
import org.apache.spark.sql.SparkSession

/**
  * AQE动态优化倾斜Join
  *
  * @author yaocs2
  * @since 2022-09-21
  */
object AQESkewJoinScala {

  def main(args: Array[String]): Unit = {
    // 获取sparkSession
    val conf = new SparkConf()
    conf.setMaster("local")
    val sparkSession = SparkSession.builder()
      .appName("AQESkewJoinScala")
      .config(conf)
      // 关闭AQE机制
      //      .config("spark.sql.adaptive.enabled", "false")
      // 关闭AQE自适应分区数量机制(避免影响skewJoin测试)
      .config("spark.sql.adaptive.coalescePartitions.enabled", "false")
      // 关闭AQE动态优化倾斜Join机制
      //      .config("spark.sql.adaptive.skewJoin.enabled", "false")
      // AQE建议分区大小
      .config("spark.sql.adaptive.advisoryPartitionSizeInBytes", "64mb")
      // 倾斜因子
      .config("spark.sql.adaptive.skewJoin.skewedPartitionFactor", "5")
      // 倾斜阈值, 要大于spark.sql.adaptive.advisoryPartitionSizeInBytes
      //      .config("spark.sql.adaptive.skewJoin.skewedPartitionThresholdInBytes", "256mb")
      .config("spark.sql.adaptive.skewJoin.skewedPartitionThresholdInBytes", "100mb")
      .getOrCreate()

    // 读取数据
    val t1JsonDF = sparkSession.read.json("D:\\spark_json_skew_t1.dat")
    val t2JsonDF = sparkSession.read.json("D:\\spark_json_skew_t2.dat")

    // 创建临时表
    t1JsonDF.createOrReplaceTempView("t1")
    t2JsonDF.createOrReplaceTempView("t2")

    // 查询t1表
    /**
      * 1、关闭AQE动态优化倾斜Join：共200个分区, 最大的分区153.9 MiB/30140910条记录, 花费25s
      * 2、开启AQE动态优化倾斜Join(满足倾斜因子5*中位数0.719B, 但没满足倾斜阈值256MB): 共200个分区, 最大的分区153.9 MiB/30140910条记录, 花费24s
      * 3、开启AQE动态优化倾斜Join(满足倾斜因子5*中位数0.719B, 且没满足1倾斜阈值00MB): 共202个分区, 最大的分区61.8 MiB/12052718条记录, 花费24s
      */
    val sql =
      """
        |select
        | t1.id,
        | t2.name,
        | t1.score
        |from t1 join t2 on t1.id = t2.id
        |""".stripMargin
    sparkSession.sql(sql).write.format("json").save("hdfs://bigdata01:9000/aqe/skew_" + System.currentTimeMillis())

    // 查询spark界面
    while (true) {
      ;
    }
  }
}
```

#### 4）动态分区裁剪

DPP，Dynamic Partition Pruning，动态分区裁剪，是指在表进行 Join 操作时，当 on 后面的查询条件满足一定要求后，基于运行时推断出的信息，自动对表中的数据进行裁剪（即过滤），减少参与 Join 的数据量，以提升查询效率。

![1663778099244](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1663778099244.png)

触发条件：

1. 需要裁剪的表，必须是分区表（如上图的 t1 表），且分区字段必须在 Join#on 条件里面。
2. Join 类型必须是 Inner Join、Left Semi Join、Left Outer Join、或者 Right Outer Join。
3. 另一张表（如上图的 t2 表）里面，至少存在一个过滤条件。
4. 满足前面 3 个条件后，还需要对裁剪成本进行评价，划算的才会执行动态分区裁剪。

| 核心参数                                             | 默认值 | 作用                     |
| ---------------------------------------------------- | ------ | ------------------------ |
| spark.sql.optimizer.dynamic.PartitionPruning.enabled | true   | 是否开启动态分区裁剪功能 |

```scala
package com.bigdata.scala.sql

import org.apache.spark.SparkConf
import org.apache.spark.sql.SparkSession

/**
  * 动态分区裁剪
  *
  * @author yaocs2
  * @since 2022-09-21
  */
object DppScala {

  def main(args: Array[String]): Unit = {
    // 获取sparkSession
    val conf = new SparkConf()
    //    conf.setMaster("local")
    val sparkSession = SparkSession.builder()
      .appName("DppScala")
      .config(conf)
      // 关闭动态分区裁剪
      //      .config("spark.sql.optimizer.dynamic.PartitionPruning.enabled", "false")
      .getOrCreate()

    /**
      * 创建[0~999]的分区表t1
      */
    import sparkSession.implicits._
    sparkSession.range(1000)
      .select($"id", $"id".as("key"))
      .write
      .partitionBy("key")
      .mode("overwrite")
      .saveAsTable("t1")

    /**
      * 创建[0~9]的分区表t2
      */
    import sparkSession.implicits._
    sparkSession.range(10)
      .select($"id", $"id".as("key"))
      .write
      .partitionBy("key")
      .mode("overwrite")
      .saveAsTable("t2")

    // 查询t1、t2表
    /**
      * 1、关闭动态分区裁剪：t1表没做id<2的过滤, 32个task, 花费8s
      * 2、开启动态分区裁剪：t1表做了id<2的过滤, 2个task, 花费4s
      */
    val sql =
      """
        |select
        | t1.id,
        | t1.key
        |from t1 join t2 on (t1.key = t2.key and t2.id < 2)
      """.stripMargin
    sparkSession.sql(sql).write.format("json").save("hdfs://bigdata01:9000/aqe/dpp_" + System.currentTimeMillis())

    // 停止sparkSession
    sparkSession.stop()
  }
}
```



