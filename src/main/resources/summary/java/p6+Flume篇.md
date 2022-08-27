# **十七、Flume 篇**

### 1.1. 什么是 Flume？

Flume，是一个高可用的、高可靠的、分布式的、海量日志采集、聚合和传输系统，是目前大数据针对数据采集方面、最采用的一个框架，无需写一行代码，只需做简单的配置，启动后即可实现自动采集数据。

![1661258997203](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1661258997203.png)

|              | Flume                                                        | Logstash                                                     | FileBeat                                                     |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **语言**     | Java                                                         | Java                                                         | Go                                                           |
| **特点**     | 分布式、多路输入多路输出、自定义数据流向、支持拦截器、选择器和处理器、插件扩展性较强 | 分布式、多路输入多路输出、支持实时数据分析                   | 仅单机模式、不支持多路输入多路输出、数据处理分析能力弱、插件扩展性较强 |
| **性能**     | 内存消耗大，CPU消耗较大，较稳定                              | 内存消耗大，CPU消耗大，较稳定                                | 内存消耗小，CPU消耗小，十分稳定                              |
| **应用场景** | 更注重于数据的传输，适合对于数据路由要求较强的场景，对数据传输要求高的场景 | 更注重于数据的预处理，适合对于数据预处理要求高的场景，搭配 ELK 组件使用有很大的优势 | 更注重于轻量级，适合轻量级的日志收集场景，可搭配 Logstash 使用 |

### 1.2. Flume 具有哪些特性？

1. Flume，是一个简单的、灵活的、基于流的数据流结构。
2. Flume，具有负载均衡和故障转移机制。
3. Flume，是一个简单的、可扩展的数据模型，即 Source、Channel、Sink。

### 1.3. Flume 典型应用场景？

#### 1）多路输出

![1661259203258](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1661259203258.png)

#### 2）多路输入

![1661259224763](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1661259224763.png)

### 1.4. Flume 三大核心组件？

#### 1）Source

Source，数据源，负责从外界采集各种类型的数据，将数据传递给 Channel。

| 类型                      | 作用                                                         |
| ------------------------- | ------------------------------------------------------------ |
| Exec Source               | 监控文件，注意：tail -F 监控的是文件名，tail -f 监控的是文件描述符本身（不随文件名改变而转移） |
| NetCat TCP / UDP Source   | 采集指定 TCP / UDP 端口的数据                                |
| Spooling Directory Source | 采集文件夹新增的文件                                         |
| Kafka Source              | 从 Kafka 消息队列中采集数据                                  |
| Avro Source               | 能够对数据以 avro 格式反序列化，从而提升性能                 |

#### 2）Channel

Channel，临时存储数据的管道，负责接收 Source 发过来的数据，做临时存储。

| 类型                     | 作用                                                         |
| ------------------------ | ------------------------------------------------------------ |
| Memory Channel           | 使用内存作为数据的存储，缺点是可能会丢数据、以及内存不够用的情况 |
| File Channel             | 使用文件夹作为数据的存储，缺点是性能来得要比内存的要慢些     |
| Spillable Memory Channel | 使用内存和文件作为数据的存储，即先存到内存，如果内存中数据达到阈值后，再 flush 到文件中，虽然解决了内存不够用的问题，但还是存储丢失数据的风险 |

#### 3）Sink

Sink，目的地，负责从 Channel 中读取数据、并存储到指定的目的地，其中，Channel 数据是直到目的地后才会被删除，以支持当 Sink 写失败后，实现自动重写，保证数据不丢失。

| 类型        | 作用                                                         |
| ----------- | ------------------------------------------------------------ |
| Logger Sink | 将数据作为日志处理                                           |
| HDFS Sink   | 将数据传输到 HDFS 中保存                                     |
| Kafka Sink  | 将数据发送到 Kafka 消息队列中                                |
| Avro Sink   | 能够对数据以 avro 格式序列化，从而提升性能，默认攒够 100 条才发送数据 |

### 1.5. Flume 安装部署？

#### 1）上传、解压

```shell
[root@bigdata04 soft]# tar -zxvf apache-flume-1.9.0-bin.tar.gz 
```

#### 2）修改启动脚本

```shell
[root@bigdata04 conf]# pwd
/data/soft/apache-flume-1.9.0-bin/conf
[root@bigdata04 conf]# mv flume-env.sh.template flume-env.sh
```

#### 3）配置三大组件

```shell
# agent的名称为a1

# 指定source、channel、sink组件的名称
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# 配置source组件
a1.sources.r1.type = netcat
a1.sources.r1.bind = 0.0.0.0
a1.sources.r1.port = 44444

# 配置channel组件
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

# 配置sink组件
a1.sinks.k1.type = logger

# 把三组件连接起来
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```

#### 4）启动 Flume

```shell
[root@bigdata04 bin]# pwd
/data/soft/apache-flume-1.9.0-bin/bin

# 前台执行
[root@bigdata04 bin]# flume-ng agent --name a1 --conf conf --conf-file ../conf/example.conf -Dflume.root.logger=INFO,console
Info: Including Hadoop libraries found via (/data/soft/hadoop-3.2.0/bin/hadoop) for HDFS access
...
2022-08-23 22:05:53,416 INFO instrumentation.MonitoredCounterGroup: Component type: CHANNEL, name: c1 started
2022-08-23 22:05:53,762 INFO node.Application: Starting Sink k1
2022-08-23 22:05:53,763 INFO node.Application: Starting Source r1
2022-08-23 22:05:53,768 INFO source.NetcatSource: Source starting
2022-08-23 22:05:53,802 INFO source.NetcatSource: Created serverSocket:sun.nio.ch.ServerSocketChannelImpl[/127.0.0.1:44444]

# 后台执行
[root@bigdata04 bin]# nohup flume-ng agent --name a1 --conf conf --conf-file ../conf/example.conf -Dflume.root.logger=INFO,console &
[1] 2649
[root@bigdata04 bin]# nohup: 忽略输入并把输出追加到"nohup.out"

# 验证是否启动成功
[root@bigdata04 bin]# jps -m
2649 Application --name a1 --conf-file ../conf/example.conf
2764 Jps -m
```

#### 5）验证 Flume

```shell
# 安装telnet
[root@bigdata04 ~]# yum install -y telnet

# 连接、发送数据
[root@bigdata04 ~]# telnet localhost 44444
Trying ::1...
telnet: connect to address ::1: Connection refused
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
hello world!
OK

# Flume上即时输出验证数据
2022-08-23 22:08:31,412 INFO sink.LoggerSink: Event: { headers:{} body: 68 65 6C 6C 6F 20 77 6F 72 6C 64 21 0D          hello world!. }
```

### 1.6. Flume Event？

1. Event，是 Flume 传输数据的基本单位，也是 Flume 事务的基本单位，在文本文件中，通常一行记录就是一个 Event。
2. Event 里有 header 和 body，header 其实就是一个 `Map<String, String>`。
3. 其中，在 Source 中增加的 header `<key, value>`，到了 Channel 和 Sink 中也可以获取到。

### 1.7. Flume 事务？

#### 1）事务分类

1. Put 事务：Source 和 Channel 之间。
2. Take 事务：Channel 和 Sink 之间。

![1661587005090](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1661587005090.png)

#### 2）事务作用

1. 无论是 Put 或 Take 事务，都是为了将数据打包成原子操作，确保数据的完整性。
2. 当 Put 事务发现 Channel 空间不足以放下一批次 Event 时，开启回滚操作，让 Source 重新读取这批数据
3. 当 Sink 往其他数据源写出数据的时候，如果遇到网络连接失败等问题，也开启回滚操作，把这批数据还给 Channel，让 Sink 重新 take 这批数据。

### 1.8. Flume 高级组件？

#### 1）Source Interceptors

Source 拦截器，按先后顺序，依次对采集到的数据进行处理。

| 类型                           | 作用                                                         |
| ------------------------------ | ------------------------------------------------------------ |
| Timestamp Interceptor          | 可以往 header 中添加时间戳属性                               |
| Host Interceptor               | 可以往 header 中添加 host 属性                               |
| Search and Replace Interceptor | 根据查询规则，查询 Event#body 的原始数据，然后根据替换规则进行替换，即会修改原始 body 的值 |
| Static Interceptor             | 可以往 header 中添加一些固定的 key 和 value                  |
| Regex Extractor Interceptor    | 正则匹配 Event#body 的原始数据，然后生成 key 和 value，再将它们添加到 Header 中 |

示例：

```shell
# agent的名称为a1

# 指定source、channel、sink组件的名称
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# 配置source组件
a1.sources.r1.type = exec
a1.sources.r1.command = tail -F /data/log/moreType.log

# 配置Source拦截器(顺序执行)
a1.sources.r1.interceptors = i1 i2 i3 i4
a1.sources.r1.interceptors.i1.type = search_replace
a1.sources.r1.interceptors.i1.searchPattern = "type":"video_info"
a1.sources.r1.interceptors.i1.replaceString = "type":"videoInfo"

a1.sources.r1.interceptors.i2.type = search_replace
a1.sources.r1.interceptors.i2.searchPattern = "type":"user_info"
a1.sources.r1.interceptors.i2.replaceString = "type":"userInfo"

a1.sources.r1.interceptors.i3.type = search_replace
a1.sources.r1.interceptors.i3.searchPattern = "type":"gift_record"
a1.sources.r1.interceptors.i3.replaceString = "type":"giftRecord"

a1.sources.r1.interceptors.i4.type = regex_extractor
a1.sources.r1.interceptors.i4.regex = "type":"(\\w+)"
a1.sources.r1.interceptors.i4.serializers = s1
a1.sources.r1.interceptors.i4.serializers.s1.name = logType

# 配置channel组件
a1.channels.c1.type = file
a1.channels.c1.checkpointDir = /data/soft/apache-flume-1.9.0-bin/moreType/checkpoint
a1.channels.c1.dataDirs = /data/soft/apache-flume-1.9.0-bin/moreType/data

# 配置sink组件
a1.sinks.k1.type = hdfs
# 输入到HDFS的路径 => 输出结果eg：xx/moreType/20220826/giftRecord/data.1661526007233.log
a1.sinks.k1.hdfs.path = hdfs://192.168.56.106:9000/moreType/%Y%m%d/%{logType}
# 获取本地时间作为时间戳
a1.sinks.k1.hdfs.useLocalTimeStamp = true
# 文件前缀
a1.sinks.k1.hdfs.filePrefix = data
# 文件后缀
a1.sinks.k1.hdfs.fileSuffix = .log
# 原生数据
a1.sinks.k1.hdfs.fileType = DataStream
# 写入文本型
a1.sinks.k1.hdfs.writeFormat = Text
# 按时间切割文件：1h
a1.sinks.k1.hdfs.rollInterval = 3600
# 按大小切割文件：128 mb
a1.sinks.k1.hdfs.rollSize = 134217728
# 按条数切割：0表示不按条数切割
a1.sinks.k1.hdfs.rollCount = 0

# 把三组件连接起来
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```

#### 2）Channel Selectors

Channel 选择器，设置 Source 发往多个 Channel 的策略。

| 类型                          | 作用                                                         |
| ----------------------------- | ------------------------------------------------------------ |
| Replicating Channel Selector  | 默认，会将 Source 采集过来的 Event 发往所有 Channel          |
| Multiplexing Channel Selector | 会根据 Event 中 Header 里面的值，将 Event 发往不同的 Channel |

示例：

![1661580401358](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1661580401358.png)

```shell
# agent的名称为a1

# 指定source、channel、sink组件的名称
a1.sources = r1
a1.channels = c1 c2
a1.sinks = k1 k2

# 配置source组件
a1.sources.r1.type = netcat
a1.sources.r1.bind = 0.0.0.0
a1.sources.r1.port = 44444

# 配置source拦截器(顺序执行)
a1.sources.r1.interceptors = i1
a1.sources.r1.interceptors.i1.type = regex_extractor
a1.sources.r1.interceptors.i1.regex = "city":"(\\w+)"
a1.sources.r1.interceptors.i1.serializers = s1
a1.sources.r1.interceptors.i1.serializers.s1.name = city

# 配置Channel选择器
a1.sources.r1.selector.type = multiplexing
a1.sources.r1.selector.header = city
a1.sources.r1.selector.mapping.bj = c1
a1.sources.r1.selector.default = c2

# 配置channel组件
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

a1.channels.c2.type = memory
a1.channels.c2.capacity = 1000
a1.channels.c2.transactionCapacity = 100

# 配置sink组件
a1.sinks.k1.type = logger

a1.sinks.k2.type = hdfs
# 输入到HDFS的路径
a1.sinks.k2.hdfs.path = hdfs://192.168.56.106:9000/multiplexing
# 获取本地时间作为时间戳
a1.sinks.k2.hdfs.useLocalTimeStamp = true
# 文件前缀
a1.sinks.k2.hdfs.filePrefix = data
# 文件后缀
a1.sinks.k2.hdfs.fileSuffix = .log
# 原生数据
a1.sinks.k2.hdfs.fileType = DataStream
# 写入文本型
a1.sinks.k2.hdfs.writeFormat = Text
# 按时间切割文件：1h
a1.sinks.k2.hdfs.rollInterval = 3600
# 按大小切割文件：128 mb
a1.sinks.k2.hdfs.rollSize = 134217728
# 按条数切割：0表示不按条数切割
a1.sinks.k2.hdfs.rollCount = 0

# 把三组件连接起来
a1.sources.r1.channels = c1 c2
a1.sinks.k1.channel = c1
a1.sinks.k2.channel = c2
```

#### 3）Sink Processors

Sink 后置处理器，设置 Sink 发送数据的策略。

| 类型                          | 作用                                                         |
| ----------------------------- | ------------------------------------------------------------ |
| Default Sink Processor        | 默认，无需配置 sink group，即一个 channel 连接一个 sink      |
| Load balancing Sink Processor | 负载均衡处理器，一个 Channel 连接多个 sink，这些 sink 同属一个 sink group，然后可以指定负载均衡算法，比如轮询、随机发送等，以减轻单个 sink 的压力 |
| Failover Sink Processor       | 故障转移处理器，一个 Channel 连接多个 sink，这些 sink 同属一个 sink group，会按照 sink 优先级进行处理，默认先给优先级高的 sink 处理，如果该 sink 出现故障，则再交给优先级低点的 sink 处理，从而使得尽可能有 sink 工作，保证数据不丢失 |

示例：

![1661581527723](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1661581527723.png)

```shell
# bigdata04配置：
# agent的名称为a1

# 指定source、channel、sink组件的名称
a1.sources = r1
a1.channels = c1
a1.sinks = k1 k2

# 配置source组件
a1.sources.r1.type = netcat
a1.sources.r1.bind = 0.0.0.0
a1.sources.r1.port = 44444

# 配置channel组件
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

# 配置sink组件
a1.sinks.k1.type = avro
a1.sinks.k1.hostname = 192.168.56.109
a1.sinks.k1.port = 41414
# 攒够多少条再发送
a1.sinks.k2.batch-size = 1

a1.sinks.k2.type = avro
a1.sinks.k2.hostname = 192.168.56.110
a1.sinks.k2.port = 41414
# 攒够多少条再发送
a1.sinks.k2.batch-size = 1

# 配置sink后置处理器
a1.sinkgroups = g1
a1.sinkgroups.g1.sinks = k1 k2
a1.sinkgroups.g1.processor.type = load_balance
# 故障记录、等待
a1.sinkgroups.g1.processor.backoff = true
# 轮询，还有random随机
a1.sinkgroups.g1.processor.selector = round_robin

# 把三组件连接起来
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
a1.sinks.k2.channel = c1

# bigdata02/03配置：
# agent的名称为a1

# 指定source、channel、sink组件的名称
a1.sources = r1
a1.channels = c1
a1.sinks = k1

# 配置source组件
a1.sources.r1.type = avro
a1.sources.r1.bind = 0.0.0.0
a1.sources.r1.port = 41414

# 配置channel组件
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

# 配置sink组件
a1.sinks.k1.type = hdfs
# 输入到HDFS的路径
a1.sinks.k1.hdfs.path = hdfs://192.168.56.106:9000/load_balance
# 获取本地时间作为时间戳
a1.sinks.k1.hdfs.useLocalTimeStamp = true
# 文件前缀
a1.sinks.k1.hdfs.filePrefix = data-bigdata02
# 文件后缀
a1.sinks.k1.hdfs.fileSuffix = .log
# 原生数据
a1.sinks.k1.hdfs.fileType = DataStream
# 写入文本型
a1.sinks.k1.hdfs.writeFormat = Text
# 按时间切割文件：1h
a1.sinks.k1.hdfs.rollInterval = 3600
# 按大小切割文件：128 mb
a1.sinks.k1.hdfs.rollSize = 134217728
# 按条数切割：0表示不按条数切割
a1.sinks.k1.hdfs.rollCount = 0

# 把三组件连接起来
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```

### 1.9. Flume 自定义组件？

可以自定义，包括：Source、Channel、Sink 核心组件，以及 Source Interceptors、Channel Selectors、Sink Processors 高级组件。

#### 1）自定义 Source Interceptors

模拟自定义实现 Search and Replace Interceptor

```java
import com.google.common.base.Preconditions;
import org.apache.commons.lang.StringUtils;
import org.apache.flume.Context;
import org.apache.flume.Event;
import org.apache.flume.interceptor.Interceptor;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.HashMap;
import java.util.List;
import java.util.regex.Matcher;
import java.util.regex.Pattern;

/**
 * 自定义拦截器
 
 * 将event的body中“type”: video_info转化为videoInfo
 * 参照Source拦截器 Search and Replace Interceptors
 */
public class MySearchAndReplaceInterceptor implements Interceptor {

    private static final Logger logger = LoggerFactory
            .getLogger(MySearchAndReplaceInterceptor.class);

    /**
     * 需要替换的字符串信息
     * 格式："key:value,key:value"
     */
    private final String search_replace;
    private String[] splits;
    private String[] key_value;
    private String key;
    private String value;
    private HashMap<String, String> hashMap = new HashMap<String, String>();
    private Pattern compile = Pattern.compile("\"type\":\"(\\w+)\"");
    private Matcher matcher;
    private String group;

    private MySearchAndReplaceInterceptor(String search_replace) {
        this.search_replace = search_replace;
    }

    /**
     * 初始化放在，最开始执行一次
     * 把配置的数据初始化到map中，方便后面调用
     */
    public void initialize() {
        try{
            if(StringUtils.isNotBlank(search_replace)){
                splits = search_replace.split(",");
                for (String key_value_pair:splits) {
                    key_value = key_value_pair.split(":");
                    key = key_value[0];
                    value = key_value[1];
                    hashMap.put(key,value);
                }
            }
        }catch (Exception e){
            logger.error("数据格式错误，初始化失败。"+search_replace,e.getCause());
        }

    }
    
    public void close() {

    }

    /**
     * 具体的处理逻辑
     
     * @param event
     * @return
     */
    public Event intercept(Event event) {
        try{
            String origBody = new String(event.getBody());
            matcher = compile.matcher(origBody);
            if(matcher.find()){
                group = matcher.group(1);
                if(StringUtils.isNotBlank(group)){
                    String newBody = origBody.replaceAll("\"type\":\""+group+"\"", "\"type\":\""+hashMap.get(group)+"\"");
                    event.setBody(newBody.getBytes());
                }
            }
        }catch (Exception e){
            logger.error("拦截器处理失败！",e.getCause());
        }
        return event;
    }

    public List<Event> intercept(List<Event> events) {
        for (Event event : events) {
            intercept(event);
        }
        return events;
    }

    public static class Builder implements Interceptor.Builder {
        private static final String SEARCH_REPLACE_KEY = "searchReplace";

        private String searchReplace;

        public void configure(Context context) {
            searchReplace = context.getString(SEARCH_REPLACE_KEY);
            Preconditions.checkArgument(!StringUtils.isEmpty(searchReplace),
                    "Must supply a valid search pattern " + SEARCH_REPLACE_KEY +
                            " (may not be empty)");
        }

        public Interceptor build() {
            Preconditions.checkNotNull(searchReplace,
                    "Regular expression searchReplace required");
            return new MySearchAndReplaceInterceptor(searchReplace);
        }

    }
}
```

```shell
# agent的名称是a1
# 指定source组件、channel组件和Sink组件的名称
a1.sources = r1
a1.channels = c1
a1.sinks = k1

# 配置source组件
a1.sources.r1.type = exec
a1.sources.r1.command = tail -F /data/log/moreType.log

# 配置拦截器 [多个拦截器按照顺序依次执行]
a1.sources.r1.interceptors = i1 i2
a1.sources.r1.interceptors.i1.type = com.imooc.flume.MySearchAndReplaceInterceptor$Builder
a1.sources.r1.interceptors.i1.searchReplace = gift_record:giftRecord,user_info:userInfo,video_info:videoInfo

a1.sources.r1.interceptors.i2.type = regex_extractor
a1.sources.r1.interceptors.i2.regex = "type":"(\\w+)"
a1.sources.r1.interceptors.i2.serializers = s1
a1.sources.r1.interceptors.i2.serializers.s1.name = logType

# 配置channel组件
a1.channels.c1.type = file
a1.channels.c1.checkpointDir = /data/soft/apache-flume-1.9.0-bin/data/moreType/checkpoint
a1.channels.c1.dataDirs = /data/soft/apache-flume-1.9.0-bin/data/moreType/data

# 配置sink组件
a1.sinks.k1.type = hdfs
a1.sinks.k1.hdfs.path = hdfs://192.168.132.100:9000/moreType/%Y%m%d/%{logType}
a1.sinks.k1.hdfs.filePrefix = data
a1.sinks.k1.hdfs.fileSuffix = .log
a1.sinks.k1.hdfs.fileType = DataStream
a1.sinks.k1.hdfs.writeFormat = Text
a1.sinks.k1.hdfs.rollInterval = 3600
a1.sinks.k1.hdfs.rollSize = 134217728
a1.sinks.k1.hdfs.rollCount = 0 
a1.sinks.k1.hdfs.useLocalTimeStamp = true

# 把组件连接起来
a1.sources.r1.channels = c1
```

#### 2）自定义 Sink

自定义 Sink，实现 Flume -> MySQL

```java
import org.apache.flume.*;
import org.apache.flume.conf.Configurable;
import org.apache.flume.sink.AbstractSink;
import java.sql.*;
import java.text.SimpleDateFormat;

public class MysqlSink extends AbstractSink implements Configurable {
    
    private PreparedStatement stmt;
    private String mysqlUrl;
    private String username;
    private String password;
    private String tableName;
    private Connection conn;

    /**
     * 获取配置文件指定的参数
     *
     * @param context
     */
    @Override
    public void configure(Context context) {
        mysqlUrl = context.getString("mysql.url");
        username = context.getString("mysql.username");
        password = context.getString("mysql.password");
        tableName = context.getString("mysql.tableName");
    }

    @Override
    public Status process() throws EventDeliveryException {
        Status status = null;

        // Start transaction 获得Channel对象
        Channel ch = getChannel();
        Transaction txn = ch.getTransaction();
        txn.begin();
        try {
            // This try clause includes whatever Channel operations you want to do
            Event event = ch.take();
            if (event != null) {
                //获取body中的数据
                String content = new String(event.getBody(), "UTF-8");
                String[] contents = content.split(",");
                
                // 操作JDBC，存入Mysql
                SimpleDateFormat df = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
                Timestamp timestamp = new Timestamp(df.parse(contents[0]).getTime());
                stmt = conn.prepareStatement("insert into " + tableName + "(name,age,city,create_time) values (?,?,?,?)");
                stmt.setString(1, contents[1]);
                stmt.setInt(2,Integer.parseInt(contents[2]));
                stmt.setString(3, contents[3]);
                stmt.setTimestamp(4,timestamp);
                stmt.execute();
                stmt.close();
                status = Status.READY;
            } else {
                status = Status.BACKOFF;
            }

            txn.commit();
        } catch (Throwable t) {
            txn.rollback();
            t.getCause().printStackTrace();
            status = Status.BACKOFF;
        } finally {
            txn.close();
        }
        return status;
    }

    @Override
    public synchronized void start() {
        try {
            // 初始化数据库连接
            Class.forName("com.mysql.cj.jdbc.Driver");
            if (conn == null) {
                conn = DriverManager.getConnection(mysqlUrl, username, password);
                System.out.println("finish start");
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    @Override
    public synchronized void stop() {
        try {
            conn.close();
        } catch (SQLException e) {
            e.printStackTrace();
        }
        super.stop();
    }
}
```

```shell
# agent的名称是a1
# 指定source组件、channel组件和Sink组件的名称
a1.sources = r1
a1.channels = c1
a1.sinks = k1

# 配置source组件
a1.sources.r1.type = exec
a1.sources.r1.command = tail -F /data/log/user.log

# 配置channel组件
a1.channels.c1.type = file
a1.channels.c1.checkpointDir = /data/soft/apache-flume-1.9.0-bin/data/customMysqlSink/checkpoint
a1.channels.c1.dataDirs = /data/soft/apache-flume-1.9.0-bin/data/customMysqlSink/data

# 配置sink组件
a1.sinks.k1.type = com.imooc.flume.MysqlSink
a1.sinks.k1.mysql.url=jdbc:mysql://192.168.132.100:3306/flume
a1.sinks.k1.mysql.username=root
a1.sinks.k1.mysql.password=Password123456?
a1.sinks.k1.mysql.tableName=user

# 把组件连接起来
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```

### 2.0. Flume 优化？

1. 调整 Flume 进程大小，建议设置为 1G ~ 2G，太小会导致频繁 GC。

   ```shell
   /data/soft/apache-flume-1.9.0-bin/conf
   # Give Flume more memory and pre-allocate, enable remote monitoring via JMX
   # export JAVA_OPTS="-Xms100m -Xmx2000m -Dcom.sun.management.jmxremote"
   ```

   ```shell
   [root@bigdata02 ~]# jstat -gcutil 3476 1000
     S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT   
   100.00   0.00  73.51  40.37  96.02  94.85      6    0.010     0    0.000    0.010
   100.00   0.00  73.51  40.37  96.02  94.85      6    0.010     0    0.000    0.010
   100.00   0.00  73.51  40.37  96.02  94.85      6    0.010     0    0.000    0.010
   100.00   0.00  73.52  40.37  96.02  94.85      6    0.010     0    0.000    0.010
   100.00   0.00  73.52  40.37  96.02  94.85      6    0.010     0    0.000    0.010
   100.00   0.00  73.52  40.37  96.02  94.85      6    0.010     0    0.000    0.010
   # S0：survivor0使用比例
   # S1：survivor1使用比例
   # E：eden区使用比例
   # O：老年代使用比例
   # M：元数据区使用比例
   # CCS：压缩类使用比例
   # YGC：年轻代垃圾回收次数
   # YGCT：年轻代垃圾回收消耗时间
   # FGC：老年代垃圾回收次数
   # FGCT：老年代垃圾回收消耗时间
   # GCT：垃圾回收消耗总时间
   ```

2. 启动多个 Flume#Agent 进程时，可以复制多个 conf 目录，并修改 log4j.properties 中日志文件的名称，以区分不同的配置，和产生不同 Agent 进程的日志，同时将日志级别修改为 WARN，因为 INFO 产生的日志太多了。

   ```shell
   # 1、修改/data/soft/apache-flume-1.9.0-bin/conf-failover/log4j.properties
   #flume.root.logger=DEBUG,console
   flume.root.logger=WARN,LOGFILE
   flume.log.dir=./logs
   flume.log.file=flume-failover.log
   
   # 2、启动命令
   nohup flume-ng agent --name a1 --conf conf-failover --conf-file ../conf-failover/failover.conf &
   ```

### 2.1. Flume 进程监控？

Flume 是一个单进程程序，存在单点故障，所以需要一个监控机制，发现进程 Down 掉后进行重启。

这里通过 Shell 脚本，实现 Flume 进程监控和重启。

#### 1）创建监控列表

monlist.conf，设置要监控的列表，以及对应重启执行的脚本。

```shell
example=startExample.sh
```

#### 2）创建重启脚本

startExample.sh

```shell
#!/bin/bash
flume_path=/data/soft/apache-flume-1.9.0-bin
nohup ${flume_path}/bin/flume-ng agent --name a1 --conf ${flume_path}/conf --conf-file ${flume_path}/conf/example.conf &
```

#### 3）创建监控程序脚本

monlist.sh

```shell
#!/bin/bash
monlist=`cat monlist.conf`
echo "start check"
for item in ${monlist}
do
	# 设置字段分隔符, IFS是存储定界符的环境变量, 是shell环境中默认的定界字符串, 默认值为空串
	OLD_IFS=$IFS
	IFS="="
	
	# 把一行内容转成多列, 即数组形式
	arr=($item)
	
	# 还原IFS为旧的值
	IFS=$OLD_IFS
	
	# 获取等号左边内容
	name=${arr[0]}
	
	# 获取等号右边内容
	script=${arr[1]}
	
	echo "time is: "`date +"%Y-%m-%d %H:%M:%S"` " check " $name
		
	# 根据标识, 查看对应进程在不在
	if [ `jps -m | grep $name | wc -l` -eq 0 ]
	then
		# 进程已Down掉
		echo "time is: "`date +"%Y-%m-%d %H:%M:%S"` $name " is down"
		
		# 重启
		sh -x ./${script}
	fi
done
```

#### 4）定时调度监控程序

```shell
[root@bigdata01 ~]# vim /etc/crontab

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed
  *  *  *  *  * root       sh /data/shell/monlist.sh >> /data/logs/monlist.log
```


