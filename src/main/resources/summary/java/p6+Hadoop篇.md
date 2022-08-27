# **十六、Hadoop 篇**

### 1.1. 什么是大数据？

大数据，big data，或称巨量资料，指的是基于规模巨大的资料，使得可以在合理时间内，通过撷取、管理、处理、整理成为更有价值的信息，典型的例子有，今日头条的个性推荐、百度地图的拥塞推断、《买披萨的故事》等。

### 1.2. 大数据产生的背景？

- 信息技术的进步。
- 云计算的兴起。
- 数据越来越被重视，数据资源化的趋势。

### 1.3. 大数据的 4V 特征？

1. Volume 量大：存储量、计算量大。
2. Variety 多样：来源多、格式多。
3. Velocity 快速：数据增长速度快、处理速度要求快。
4. Value 价值：价值密度低、和数据总量成反比。

### 1.4. 什么是 Hadoop？

Hadoop，是一个由 Apache 开发的分布式系统基础架构，充分利用集群的威力进行分布式存储和计算，用户可以在不了解分布式底层细节的情况下，开发分布式程序。

目前，Hadoop 已经演变为了大数据的代名词，形成了一套完善的大数据生态系统，也出现了很多发行版：

1. Apache Hadoop：官方版本，开源，也是现在学习的主要版本，缺点是技术支持比较慢。

2. Cloudera Hadoop：

   - 1）简称 CDH，是一个商业版本，对官方版本做了一些优化，提供收费的技术支持，以及界面操作，方便集群运维和管理。
   - 2）目前在企业中使用比较多，一些基本功能不收费，高级功能才收费。

   ![1660988504269](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1660988504269.png)

3. HortonWorks：

   - 1）简称 HDP，开源，提供界面操作，方便运维管理，一般互联网公司偏向于使用这个。
   - 2）目前 HDP 已经被 CDH 收购了，都属同一家公司的产品。

   ![1660988731987](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1660988731987.png)

### 1.5. Hadoop 三大核心组件？

1. HDFS：负责海量数据的分布式存储。
2. MapReduce：是一个计算模型，负责海量数据的分布式计算。
3. YARN：负责集群资源的管理和调度。

### 1.6. Hadoop 集群架构？

| 部署方式     | 部署详情            |
| ------------ | ------------------- |
| 伪分布式集群 | 使用一台 Linux 机器 |
| 分布式集群   | 使用多台 Linux 机器 |

#### 1）伪分布式集群

![1660391039404](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1660391039404.png)

**安装部署步骤**：

##### 1、设置主机名称

```shell
# 临时设置，立即生效
[root@bigdata01 ~]# hostname bigdata01
# 永久设置，重启生效
[root@bigdata01 ~]# vi /etc/hostname 
bigdata01

# 设置 dfs
[root@bigdata01 ~]# vi /etc/hosts 
192.168.182.100 bigdata01
```

##### 2、关闭防火墙

```shell
[root@bigdata01 ~]# systemctl stop firewalld
[root@bigdata01 ~]# systemctl disable firewalld
```

##### 3、设置免密码登录

```shell
# 在~/.ssh目录下生成对应的公钥和私钥文件
[root@bigdata01 ~]# ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:I8J8RDun4bklmx9T45SRsKAu7FvP2HqtriYUqUqF1q4 root@bigdata01
The key's randomart image is:
+---[RSA 2048]----+
|      o .        |
|     o o o .     |
|  o.. = o o      |
| +o* o *   o     |
|..=.= B S =      |
|.o.o o B = .     |
|o.o . +.o .      |
|.E.o.=...o       |
|  .o+=*..        |
+----[SHA256]-----+
[root@bigdata01 ~]# ll ~/.ssh/
total 12
-rw-------. 1 root root 1679 Apr  7 16:39 id_rsa
-rw-r--r--. 1 root root  396 Apr  7 16:39 id_rsa.pub

# 把公钥拷贝到需要免密码登录的机器上面
[root@bigdata01 ~]# cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys

# 免密码登录
[root@bigdata01 ~]# ssh bigdata01
Last login: Tue Apr  7 15:05:55 2020 from 192.168.182.1
```

##### 4、安装 JDK

```shell
# 上传、解压
[root@bigdata01 soft]# tar -zxvf jdk-8u202-linux-x64.tar.gz

# 重命名
[root@bigdata01 soft]# mv jdk1.8.0_202 jdk1.8

# 配置环境变量
[root@bigdata01 soft]# vi /etc/profile
.....
export JAVA_HOME=/data/soft/jdk1.8
export PATH=.:$JAVA_HOME/bin:$PATH

# 验证环境
[root@bigdata01 soft]# source /etc/profile
[root@bigdata01 soft]# java -version          
java version "1.8.0_202"
Java(TM) SE Runtime Environment (build 1.8.0_202-b08)
Java HotSpot(TM) 64-Bit Server VM (build 25.202-b08, mixed mode)
```

##### 5、解压 Hadoop

```shell
# 上传、解压
[root@bigdata01 soft]# tar -zxvf hadoop-3.2.0.tar.gz

# 设置环境变量
[root@bigdata01 hadoop-3.2.0]# vi /etc/profile
.......
export JAVA_HOME=/data/soft/jdk1.8
export HADOOP_HOME=/data/soft/hadoop-3.2.0
export PATH=.:$JAVA_HOME/bin:$HADOOP_HOME/sbin:$HADOOP_HOME/bin:$PATH
[root@bigdata01 hadoop-3.2.0]# source /etc/profile
```

##### 6、修改配置文件

```shell
[root@bigdata01 hadoop-3.2.0]# cd etc/hadoop/
[root@bigdata01 hadoop]# 

# 1、修改hadoop-env.sh文件，增加环境变量信息，添加到hadoop-env.sh文件末尾即可
[root@bigdata01 hadoop]# vi hadoop-env.sh
.......
export JAVA_HOME=/data/soft/jdk1.8
export HADOOP_LOG_DIR=/data/hadoop_repo/logs/hadoop

# 2、修改 core-site.xml 文件
[root@bigdata01 hadoop]# vi core-site.xml
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://bigdata01:9000</value>
    </property>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/data/hadoop_repo</value>
   </property>
</configuration>

# 3、修改hdfs-site.xml文件，把hdfs中文件副本的数量设置为1，因为现在伪分布集群只有一个节点
[root@bigdata01 hadoop]# vi hdfs-site.xml
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
</configuration>

# 4、修改mapred-site.xml，设置mapreduce使用的资源调度框架
[root@bigdata01 hadoop]# vi mapred-site.xml
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>

# 5、修改yarn-site.xml，设置yarn上支持运行的服务和环境变量白名单
[root@bigdata01 hadoop]# vi yarn-site.xml
<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.nodemanager.env-whitelist</name>
 <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value>
    </property>
</configuration>

# 6、修改workers，设置集群中从节点的主机名信息，在这里就一台集群，所以就填写bigdata01即可
[root@bigdata01 hadoop]# vi workers
bigdata01
```

##### 7、格式化 HDFS

```shell
[root@bigdata01 hadoop]# cd /data/soft/hadoop-3.2.0
[root@bigdata01 hadoop-3.2.0]# bin/hdfs namenode -format
WARNING: /data/hadoop_repo/logs/hadoop does not exist. Creating.
2020-04-07 17:45:22,086 INFO namenode.NameNode: STARTUP_MSG: 
...
2020-04-07 17:45:24,689 INFO common.Storage: Storage directory /data/hadoop_repo/dfs/name has been successfully formatted.
...
```

##### 8、修改启动脚本

```shell
# 修改sbin目录下的start-dfs.sh，stop-dfs.sh这两个脚本文件，在文件前面增加如下内容
[root@bigdata01 hadoop-3.2.0]# cd sbin/
[root@bigdata01 sbin]# vi start-dfs.sh
HDFS_DATANODE_USER=root
HDFS_DATANODE_SECURE_USER=hdfs
HDFS_NAMENODE_USER=root
HDFS_SECONDARYNAMENODE_USER=root
[root@bigdata01 sbin]# vi stop-dfs.sh
HDFS_DATANODE_USER=root
HDFS_DATANODE_SECURE_USER=hdfs
HDFS_NAMENODE_USER=root
HDFS_SECONDARYNAMENODE_USER=root

# 修改sbin目录下的start-yarn.sh，stop-yarn.sh这两个脚本文件，在文件前面增加如下内容
[root@bigdata01 sbin]# vi start-yarn.sh
YARN_RESOURCEMANAGER_USER=root
HADOOP_SECURE_DN_USER=yarn
YARN_NODEMANAGER_USER=root
[root@bigdata01 sbin]# vi stop-yarn.sh
YARN_RESOURCEMANAGER_USER=root
HADOOP_SECURE_DN_USER=yarn
YARN_NODEMANAGER_USER=root
```

##### 9、启动集群

```shell
[root@bigdata01 sbin]# cd /data/soft/hadoop-3.2.0
[root@bigdata01 hadoop-3.2.0]# sbin/start-all.sh 
Starting namenodes on [bigdata01]
Last login: Tue Apr  7 16:45:28 CST 2020 from fe80::c8a8:4edb:db7b:af53%ens33 on pts/1
Starting datanodes
Last login: Tue Apr  7 17:59:21 CST 2020 on pts/0
Starting secondary namenodes [bigdata01]
Last login: Tue Apr  7 17:59:23 CST 2020 on pts/0
Starting resourcemanager
Last login: Tue Apr  7 17:59:30 CST 2020 on pts/0
Starting nodemanagers
Last login: Tue Apr  7 17:59:37 CST 2020 on pts/0

# 验证环境，其中，由于 MapReduce 是一个计算框架，所以不会在图中展示，只在运行时其相关进程才会创建。
[root@bigdata01 hadoop-3.2.0]# jps
3267 NameNode
3859 ResourceManager
3397 DataNode
3623 SecondaryNameNode
3996 NodeManager
4319 Jps

# web界面
# HDFS webui界面：http://192.168.182.100:9870
# YARN webui界面：http://192.168.182.100:8088
```

##### 10、启动 History Server

```shell
[root@bigdata01 hadoop-3.2.0]# mapred --daemon start historyserver
```

##### 11、停止集群

```shell
[root@bigdata01 hadoop-3.2.0]# sbin/stop-all.sh 
Stopping namenodes on [bigdata01]
Last login: Tue Apr  7 17:59:40 CST 2020 on pts/0
Stopping datanodes
Last login: Tue Apr  7 18:06:09 CST 2020 on pts/0
Stopping secondary namenodes [bigdata01]
Last login: Tue Apr  7 18:06:10 CST 2020 on pts/0
Stopping nodemanagers
Last login: Tue Apr  7 18:06:13 CST 2020 on pts/0
Stopping resourcemanager
Last login: Tue Apr  7 18:06:16 CST 2020 on pts/0
```

#### 2）分布式集群

![1660394757523](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1660394757523.png)

##### 1、设置主机名称

每台机器都要

```shell
# 临时设置，立即生效
[root@bigdata01 ~]# hostname bigdata01
# 永久设置，重启生效
[root@bigdata01 ~]# vi /etc/hostname 
bigdata01

# 设置 dfs
[root@bigdata02 ~]# vi /etc/hosts
192.168.182.100 bigdata01
192.168.182.101 bigdata02
192.168.182.102 bigdata03
```

##### 2、关闭防火墙

每台机器都要

```shell
[root@bigdata01 ~]# systemctl stop firewalld
[root@bigdata01 ~]# systemctl disable firewalld
```

##### 3、设置免密码登录

每台机器都要

```shell
# 在~/.ssh目录下生成对应的公钥和私钥文件
[root@bigdata01 ~]# ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:I8J8RDun4bklmx9T45SRsKAu7FvP2HqtriYUqUqF1q4 root@bigdata01
The key's randomart image is:
+---[RSA 2048]----+
|      o .        |
|     o o o .     |
|  o.. = o o      |
| +o* o *   o     |
|..=.= B S =      |
|.o.o o B = .     |
|o.o . +.o .      |
|.E.o.=...o       |
|  .o+=*..        |
+----[SHA256]-----+
[root@bigdata01 ~]# ll ~/.ssh/
total 12
-rw-------. 1 root root 1679 Apr  7 16:39 id_rsa
-rw-r--r--. 1 root root  396 Apr  7 16:39 id_rsa.pub

# 把公钥拷贝到需要免密码登录的机器上面
[root@bigdata01 ~]# cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys

# 免密码登录
[root@bigdata01 ~]# ssh bigdata01
Last login: Tue Apr  7 15:05:55 2020 from 192.168.182.1
```

##### 4、同步时钟

每台机器都要

```shell
# 安装
[root@bigdata01 ~]# yum install -y ntpdate

# 同步
[root@bigdata01 ~]# ntpdate -u ntp.sjtu.edu.cn
-bash: ntpdate: command not found

# 定时同步
[root@bigdata01 ~]# vi /etc/crontab
* * * * * root /usr/sbin/ntpdate -u ntp.sjtu.edu.cn
```

##### 5、安装 JDK

每台机器都要

```shell
# 上传、解压
[root@bigdata01 soft]# tar -zxvf jdk-8u202-linux-x64.tar.gz

# 重命名
[root@bigdata01 soft]# mv jdk1.8.0_202 jdk1.8

# 配置环境变量
[root@bigdata01 soft]# vi /etc/profile
.....
export JAVA_HOME=/data/soft/jdk1.8
export PATH=.:$JAVA_HOME/bin:$PATH

# 验证环境
[root@bigdata01 soft]# source /etc/profile
[root@bigdata01 soft]# java -version          
java version "1.8.0_202"
Java(TM) SE Runtime Environment (build 1.8.0_202-b08)
Java HotSpot(TM) 64-Bit Server VM (build 25.202-b08, mixed mode)
```

##### 6、解压 Hadoop

只修改一台，修改完成后，scp 到其他节点即可

```shell
# 上传、解压
[root@bigdata01 soft]# tar -zxvf hadoop-3.2.0.tar.gz

# 设置环境变量
[root@bigdata01 hadoop-3.2.0]# vi /etc/profile
.......
export JAVA_HOME=/data/soft/jdk1.8
export HADOOP_HOME=/data/soft/hadoop-3.2.0
export PATH=.:$JAVA_HOME/bin:$HADOOP_HOME/sbin:$HADOOP_HOME/bin:$PATH
[root@bigdata01 hadoop-3.2.0]# source /etc/profile
```

##### 7、修改配置文件

只修改一台，修改完成后，scp 到其他节点即可

```shell
# 进入目录
[root@bigdata01 soft]# cd hadoop-3.2.0/etc/hadoop/
[root@bigdata01 hadoop]# 

# 1、修改hadoop-env.sh文件，在文件末尾增加环境变量信息
[root@bigdata01 hadoop]# vi hadoop-env.sh 
export JAVA_HOME=/data/soft/jdk1.8
export HADOOP_LOG_DIR=/data/hadoop_repo/logs/hadoop

# 2、修改core-site.xml文件，注意fs.defaultFS属性中的主机名需要和主节点的主机名保持一致
[root@bigdata01 hadoop]# vi core-site.xml
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://bigdata01:9000</value>
    </property>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/data/hadoop_repo</value>
   </property>
</configuration>

# 3、修改hdfs-site.xml文件，把hdfs中文件副本的数量设置为2，最多为2，因为现在集群中有两个从节点，还有secondaryNamenode进程所在的节点信息
[root@bigdata01 hadoop]# vi hdfs-site.xml 
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>2</value>
    </property>
    <property>
        <name>dfs.namenode.secondary.http-address</name>
        <value>bigdata01:50090</value>
    </property>
</configuration>

# 4、修改mapred-site.xml，设置mapreduce使用的资源调度框架
[root@bigdata01 hadoop]# vi mapred-site.xml
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>

# 5、修改yarn-site.xml，设置yarn上支持运行的服务和环境变量白名单
[root@bigdata01 hadoop]# vi yarn-site.xml
<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.nodemanager.env-whitelist</name>
        <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value>
    </property>
	<property>
		<name>yarn.resourcemanager.hostname</name>
		<value>bigdata01</value>
	</property>
</configuration>

# 6、修改workers文件，增加所有从节点的主机名，一个一行
[root@bigdata01 hadoop]# vi workers
bigdata02
bigdata03
```

##### 8、修改启动脚本

只修改一台，修改完成后，scp 到其他节点即可

```shell
# 1、修改start-dfs.sh，stop-dfs.sh这两个脚本文件，在文件前面增加如下内容
[root@bigdata01 hadoop]# cd /data/soft/hadoop-3.2.0/sbin
[root@bigdata01 sbin]# vi start-dfs.sh
HDFS_DATANODE_USER=root
HDFS_DATANODE_SECURE_USER=hdfs
HDFS_NAMENODE_USER=root
HDFS_SECONDARYNAMENODE_USER=root

[root@bigdata01 sbin]# vi stop-dfs.sh
HDFS_DATANODE_USER=root
HDFS_DATANODE_SECURE_USER=hdfs
HDFS_NAMENODE_USER=root
HDFS_SECONDARYNAMENODE_USER=root

# 2、修改start-yarn.sh，stop-yarn.sh这两个脚本文件，在文件前面增加如下内容
[root@bigdata01 sbin]# vi start-yarn.sh
YARN_RESOURCEMANAGER_USER=root
HADOOP_SECURE_DN_USER=yarn
YARN_NODEMANAGER_USER=root

[root@bigdata01 sbin]# vi stop-yarn.sh
YARN_RESOURCEMANAGER_USER=root
HADOOP_SECURE_DN_USER=yarn
YARN_NODEMANAGER_USER=root
```

##### 9、同步 Hadoop 到其他节点

```shell
[root@bigdata01 sbin]# cd /data/soft/
[root@bigdata01 soft]# scp -rq hadoop-3.2.0 bigdata02:/data/soft/
[root@bigdata01 soft]# scp -rq hadoop-3.2.0 bigdata03:/data/soft/
```

##### 10、主节点格式化 HDFS

```shell
[root@bigdata01 soft]# cd /data/soft/hadoop-3.2.0
[root@bigdata01 hadoop-3.2.0]# bin/hdfs namenode -format
...
common.Storage: Storage directory /data/hadoop_repo/dfs/name has been successfully formatted.
...
```

##### 11、主节点启动集群

```shell
[root@bigdata01 hadoop-3.2.0]# sbin/start-all.sh 
Starting namenodes on [bigdata01]
Last login: Tue Apr  7 21:03:21 CST 2020 from 192.168.182.1 on pts/2
Starting datanodes
Last login: Tue Apr  7 22:15:51 CST 2020 on pts/1
bigdata02: WARNING: /data/hadoop_repo/logs/hadoop does not exist. Creating.
bigdata03: WARNING: /data/hadoop_repo/logs/hadoop does not exist. Creating.
Starting secondary namenodes [bigdata01]
Last login: Tue Apr  7 22:15:53 CST 2020 on pts/1
Starting resourcemanager
Last login: Tue Apr  7 22:15:58 CST 2020 on pts/1
Starting nodemanagers
Last login: Tue Apr  7 22:16:04 CST 2020 on pts/1

# 验证
[root@bigdata01 hadoop-3.2.0]# jps
6128 NameNode
6621 ResourceManager
6382 SecondaryNameNode

[root@bigdata02 ~]# jps
2385 NodeManager
2276 DataNode

[root@bigdata03 ~]# jps
2326 NodeManager
2217 DataNode

# web界面
# HDFS webui界面：http://192.168.182.100:9870
# YARN webui界面：http://192.168.182.100:8088
```

##### 12、启动 History Server

每个节点都要

```shell
[root@bigdata01 hadoop-3.2.0]# mapred --daemon start historyserver
```

##### 13、停止集群

```shell
[root@bigdata01 hadoop-3.2.0]# sbin/stop-all.sh 
Stopping namenodes on [bigdata01]
Last login: Tue Apr  7 22:21:16 CST 2020 on pts/1
Stopping datanodes
Last login: Tue Apr  7 22:22:42 CST 2020 on pts/1
Stopping secondary namenodes [bigdata01]
Last login: Tue Apr  7 22:22:44 CST 2020 on pts/1
Stopping nodemanagers
Last login: Tue Apr  7 22:22:46 CST 2020 on pts/1
Stopping resourcemanager
Last login: Tue Apr  7 22:22:50 CST 2020 on pts/1
```

#### 3）客户端节点

Hadoop 客户端节点，允许在业务机器上，操作 Hadoop 集群，避免直接把 Hadoop 集群的节点，暴露给开发人员，造成不安全的问题发生。

**安装步骤**：把集群的 Hadoop 包发送到该节点即可，无需启动，只需要使用 bin 目录下的命令而已。

![1660397090440](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1660397090440.png)

### 1.7. Hadoop 安装部署？

### 1.8. 什么是 HDFS？

HDFS，Hadoop Distributed File System，是一种允许通过网络，在多台主机上分享文件的文件系统，可以让多台机器上的多个用户分享文件和存储空间，即共享的分布式文件系统。

- HDFS 只适合大文件，不适合存储小文件。

![1660467170051](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1660467170051.png)

### 1.9 HDFS Shell 操作？

####  1）命令格式

```
bin/hdfs dfs -XXX     hdfs://bigdata01:9000
			(具体命令) (core-site.xml#fs.defaultFS)
```

#### 2）基本命令 - 查询指定路径

```shell
# 全路径
[root@bigdata01 hadoop-3.2.0]# hdfs dfs -ls hdfs://bigdata01:9000/
[root@bigdata01 hadoop-3.2.0]# 

# 配置了HADOOP_HOME和core-site.xml时, 可简写
[root@bigdata01 hadoop-3.2.0]# hdfs dfs -ls /
[root@bigdata01 hadoop-3.2.0]# 

# 递归查询多级目录
[root@bigdata01 hadoop-3.2.0]# hdfs dfs -ls -R /
-rw-r--r--   2 root supergroup       1361 2022-08-14 16:05 /README.txt
drwxr-xr-x   - root supergroup          0 2022-08-14 16:16 /abc
drwxr-xr-x   - root supergroup          0 2022-08-14 16:16 /abc/xyz
drwxr-xr-x   - root supergroup          0 2022-08-14 16:13 /test
```

#### 3）基本命令 - 上传文件

```shell
[root@bigdata01 hadoop-3.2.0]# hdfs dfs -put README.txt /
put: `/README.txt': File exists
[root@bigdata01 hadoop-3.2.0]#
```

#### 4）基本命令 - 查看文件内容

```shell
[root@bigdata01 hadoop-3.2.0]# hdfs dfs -cat /README.txt
For the latest information about Hadoop, please visit our website at:
...
```

#### 5）基本命令 - 下载文件

```shell
[root@bigdata01 hadoop-3.2.0]# hdfs dfs -get /README.txt README.txt_bak_20220814
[root@bigdata01 hadoop-3.2.0]# ll
总用量 188
drwxr-xr-x. 2 1001 1002    203 1月   8 2019 bin
drwxr-xr-x. 3 1001 1002     20 1月   8 2019 etc
drwxr-xr-x. 2 1001 1002    106 1月   8 2019 include
drwxr-xr-x. 3 1001 1002     20 1月   8 2019 lib
drwxr-xr-x. 4 1001 1002   4096 1月   8 2019 libexec
-rw-rw-r--. 1 1001 1002 150569 10月 19 2018 LICENSE.txt
-rw-rw-r--. 1 1001 1002  22125 10月 19 2018 NOTICE.txt
-rw-rw-r--. 1 1001 1002   1361 10月 19 2018 README.txt
-rw-r--r--. 1 root root   1361 8月  14 16:12 README.txt_bak_20220814
drwxr-xr-x. 3 1001 1002   4096 8月  13 21:12 sbin
drwxr-xr-x. 4 1001 1002     31 1月   8 2019 share
```

#### 6）基本命令 - 创建文件夹

```shell
[root@bigdata01 hadoop-3.2.0]# hdfs dfs -mkdir /test
[root@bigdata01 hadoop-3.2.0]# hdfs dfs -ls /
Found 2 items
-rw-r--r--   2 root supergroup       1361 2022-08-14 16:05 hdfs://bigdata01:9000/README.txt
drwxr-xr-x   - root supergroup          0 2022-08-14 16:13 hdfs://bigdata01:9000/test

# 递归创建文件目录
[root@bigdata01 hadoop-3.2.0]# hdfs dfs -mkdir -p /abc/xyz
[root@bigdata01 hadoop-3.2.0]# hdfs dfs -ls /
Found 3 items
-rw-r--r--   2 root supergroup       1361 2022-08-14 16:05 /README.txt
drwxr-xr-x   - root supergroup          0 2022-08-14 16:16 /abc
drwxr-xr-x   - root supergroup          0 2022-08-14 16:13 /test
[root@bigdata01 hadoop-3.2.0]# hdfs dfs -ls /abc
Found 1 items
drwxr-xr-x   - root supergroup          0 2022-08-14 16:16 /abc/xyz
```

#### 7）基本命令 - 删除文件/文件夹

```shell
# 删除文件
[root@bigdata01 hadoop-3.2.0]# hdfs dfs -rm /README.txt
Deleted /README.txt
[root@bigdata01 hadoop-3.2.0]# hdfs dfs -ls /
Found 2 items
drwxr-xr-x   - root supergroup          0 2022-08-14 16:16 /abc
drwxr-xr-x   - root supergroup          0 2022-08-14 16:13 /test

# 删除文件夹
[root@bigdata01 hadoop-3.2.0]# hdfs dfs -rm -R /test
Deleted /test
[root@bigdata01 hadoop-3.2.0]# hdfs dfs -ls /
Found 1 items
drwxr-xr-x   - root supergroup          0 2022-08-14 16:16 /abc
```

#### 8）高级命令 - 修改权限

```shell
[root@bigdata01 hadoop-3.2.0]# hdfs dfs -ls -R /
-rw-r--r--   2 root supergroup     150569 2022-08-14 16:22 /LICENSE.txt
-rw-r--r--   2 root supergroup      22125 2022-08-14 16:22 /NOTICE.txt
-rw-r--r--   2 root supergroup       1361 2022-08-14 16:22 /README.txt
drwxr-xr-x   - root supergroup          0 2022-08-14 16:28 /abc
drwxr-xr-x   - root supergroup          0 2022-08-14 16:28 /abc/xyz

[root@bigdata01 hadoop-3.2.0]# hdfs dfs -chmod 777 /README.txt
[root@bigdata01 hadoop-3.2.0]# hdfs dfs -chmod 777 /abc
[root@bigdata01 hadoop-3.2.0]# hdfs dfs -ls -R /
-rw-r--r--   2 root supergroup     150569 2022-08-14 16:22 /LICENSE.txt
-rw-r--r--   2 root supergroup      22125 2022-08-14 16:22 /NOTICE.txt
-rwxrwxrwx   2 root supergroup       1361 2022-08-14 16:22 /README.txt
drwxrwxrwx   - root supergroup          0 2022-08-14 16:28 /abc
drwxr-xr-x   - root supergroup          0 2022-08-14 16:28 /abc/xyz
```

#### 9）高级命令 - 查询文件/文件夹大小

```shell
[root@bigdata01 hadoop-3.2.0]# hdfs dfs -ls -R /
-rw-r--r--   2 root supergroup     150569 2022-08-14 16:22 /LICENSE.txt
-rw-r--r--   2 root supergroup      22125 2022-08-14 16:22 /NOTICE.txt
-rwxrwxrwx   2 root supergroup       1361 2022-08-14 16:22 /README.txt
drwxrwxrwx   - root supergroup          0 2022-08-14 16:28 /abc
drwxr-xr-x   - root supergroup          0 2022-08-14 16:28 /abc/xyz

# 查询文件/文件夹大小，单位为字节，第1列表示原始大小，第2列表示所有副本大小之和（这里副本因子配置了2）
[root@bigdata01 hadoop-3.2.0]# hdfs dfs -du /
150569  301138  /LICENSE.txt
22125   44250   /NOTICE.txt
1361    2722    /README.txt
0       0       /abc

# 便于识别地查询文件/文件夹大小，第1列表示原始大小，第2列表示所有副本大小之和（这里副本因子配置了2）
[root@bigdata01 hadoop-3.2.0]# hdfs dfs -du -h /
147.0 K  294.1 K  /LICENSE.txt
21.6 K   43.2 K   /NOTICE.txt
1.3 K    2.7 K    /README.txt
0        0        /abc

# 汇总查询整个目录的大小，第1列表示原始大小，第2列表示所有副本大小之和（这里副本因子配置了2）
[root@bigdata01 hadoop-3.2.0]# hdfs dfs -du -s /
174055  348110  /
```

#### 10）高级命令 - 回收站相关

开启回收站功能：设置文件删除后，还能存活的生命周期，超过这个周期，则会被永久删除。

```xml
<!-- hadoop-xxx/etc/hadoop/core-site.xml -->
<property>
    <name>fs.trash.interval</name>
    <value>1440</value>
</property>
```

```shell
# 删除文件 => 文件不再直接删除，而是进入到回收站中
[root@bigdata01 hadoop]# hdfs dfs -ls /
Found 8 items
-rw-r--r--   2 root supergroup     150569 2022-08-14 16:22 /LICENSE.txt
-rw-r--r--   2 root supergroup      22125 2022-08-14 16:22 /NOTICE.txt
-rwxrwxrwx   2 root supergroup       1361 2022-08-14 16:22 /README.txt
drwxrwxrwx   - root supergroup          0 2022-08-14 16:28 /abc
-rw-r--r--   2 root supergroup  345625475 2022-08-14 18:30 /hadoop-3.2.0.tar.gz
-rw-r--r--   2 root supergroup          0 2022-08-14 18:30 /hello.txt
-rw-r--r--   2 root supergroup          0 2022-08-14 16:48 /k.txt
-rw-r--r--   2 root supergroup          0 2022-08-14 16:48 /m.txt
[[A[root@bigdata01 hadoop]# hdfs dfs -rm /README.txt
2022-08-14 20:26:34,372 INFO fs.TrashPolicyDefault: Moved: 'hdfs://bigdata01:9000/README.txt' to trash at: hdfs://bigdata01:9000/user/root/.Trash/Current/README.txt
# 直接删除文件 => 跳过回收站
[root@bigdata01 hadoop]# hdfs dfs -rm -skipTrash /m.txt
Deleted /m.txt

# 查询回收站
[root@bigdata01 hadoop]# hdfs dfs -ls /user/root/.Trash
Found 2 items
drwx------   - root supergroup          0 2022-08-14 20:26 /user/root/.Trash/220814202900
drwx------   - root supergroup          0 2022-08-14 20:32 /user/root/.Trash/Current

# 清空回收站
[root@bigdata01 hadoop]# hdfs dfs -expunge
2022-08-14 20:32:39,202 INFO fs.TrashPolicyDefault: TrashPolicyDefault#deleteCheckpoint for trashRoot: hdfs://bigdata01:9000/user/root/.Trash
2022-08-14 20:32:39,202 INFO fs.TrashPolicyDefault: TrashPolicyDefault#deleteCheckpoint for trashRoot: hdfs://bigdata01:9000/user/root/.Trash
2022-08-14 20:32:39,220 INFO fs.TrashPolicyDefault: TrashPolicyDefault#createCheckpoint for trashRoot: hdfs://bigdata01:9000/user/root/.Trash
2022-08-14 20:32:39,228 INFO fs.TrashPolicyDefault: Created trash checkpoint: /user/root/.Trash/220814203239
[root@bigdata01 hadoop]# hdfs dfs -ls /user/root/.Trash
Found 2 items
drwx------   - root supergroup          0 2022-08-14 20:26 /user/root/.Trash/220814202900
drwx------   - root supergroup          0 2022-08-14 20:32 /user/root/.Trash/220814203239
```

#### 11）高级命令 - 查询文件头部内容

```shell
# 查询尾部内容
[root@bigdata01 hadoop-3.2.0]# hdfs dfs -head /README.txt

# head本身不支持动态查询
```

#### 12）高级命令 - 查询文件尾部内容

```shell
# 查询尾部内容
[root@bigdata01 hadoop-3.2.0]# hdfs dfs -tail /README.txt
...

# 动态查询尾部内容
[root@bigdata01 hadoop-3.2.0]# hdfs dfs -tail -f /README.txt
...
```

#### 13）高级命令 - 检查文件/文件夹是否存在

```shell
# -e，检查文件是否存在，如果存在，则返回0
[root@bigdata01 hadoop-3.2.0]# hdfs dfs -test -e /README.txt
[root@bigdata01 hadoop-3.2.0]# echo $?
0
[root@bigdata01 hadoop-3.2.0]# hdfs dfs -test -e /README123.txt
[root@bigdata01 hadoop-3.2.0]# echo $?
1

# -z，检查文件是否0字节，如果是，则返回0
[root@bigdata01 hadoop-3.2.0]# hdfs dfs -test -z /README.txt
[root@bigdata01 hadoop-3.2.0]# echo $?
1
[root@bigdata01 hadoop-3.2.0]# hdfs dfs -test -z /abc
[root@bigdata01 hadoop-3.2.0]# echo $?
0

# -d，如果路径是个目录，则返回0，否则返回1
[root@bigdata01 hadoop-3.2.0]# hdfs dfs -test -d /abc
[root@bigdata01 hadoop-3.2.0]# echo $?
0
[root@bigdata01 hadoop-3.2.0]# hdfs dfs -test -d /README.txt
[root@bigdata01 hadoop-3.2.0]# echo $?
1
```

#### 14）高级命令 - 创建空文件

```shell
[root@bigdata01 hadoop-3.2.0]# hdfs dfs -ls /
Found 4 items
-rw-r--r--   2 root supergroup     150569 2022-08-14 16:22 /LICENSE.txt
-rw-r--r--   2 root supergroup      22125 2022-08-14 16:22 /NOTICE.txt
-rwxrwxrwx   2 root supergroup       1361 2022-08-14 16:22 /README.txt
drwxrwxrwx   - root supergroup          0 2022-08-14 16:28 /abc

# touch创建空文件，可用于生成标记文件，比如_SUCCESS等
[[A[root@bigdata01 hadoop-3.2.0]# hdfs dfs -touch /k.txt
[root@bigdata01 hadoop-3.2.0]# hdfs dfs -ls /
Found 5 items
-rw-r--r--   2 root supergroup     150569 2022-08-14 16:22 /LICENSE.txt
-rw-r--r--   2 root supergroup      22125 2022-08-14 16:22 /NOTICE.txt
-rwxrwxrwx   2 root supergroup       1361 2022-08-14 16:22 /README.txt
drwxrwxrwx   - root supergroup          0 2022-08-14 16:28 /abc
-rw-r--r--   2 root supergroup          0 2022-08-14 16:48 /k.txt

# touchz创建空文件，等同于touch，可用于生成标记文件，比如_SUCCESS等
[root@bigdata01 hadoop-3.2.0]# hdfs dfs -touchz /m.txt
[root@bigdata01 hadoop-3.2.0]# hdfs dfs -ls /
Found 6 items
-rw-r--r--   2 root supergroup     150569 2022-08-14 16:22 /LICENSE.txt
-rw-r--r--   2 root supergroup      22125 2022-08-14 16:22 /NOTICE.txt
-rwxrwxrwx   2 root supergroup       1361 2022-08-14 16:22 /README.txt
drwxrwxrwx   - root supergroup          0 2022-08-14 16:28 /abc
-rw-r--r--   2 root supergroup          0 2022-08-14 16:48 /k.txt
-rw-r--r--   2 root supergroup          0 2022-08-14 16:48 /m.txt
```

#### 15）高级命令 - 查看文件内容（text）

```shell
# 效果类似于cat，但text还可以查看zip和TextRecordInputStream格式的数据，而cat只能查看普通文件
[root@bigdata01 hadoop-3.2.0]# hdfs dfs -text /README.txt
For the latest information about Hadoop, please visit our website at:
...
```

#### 16）高级命令 - 安全模式

集群刚启动时，HDFS 先会进入安全模式，此时无法执行写操作，当自检完毕后，则会自动退出安全模式。

```shell
# 查看当前安全模式状态
[root@bigdata01 hadoop]# hdfs dfsadmin -safemode get
Safe mode is OFF

# 刚重启时，删除文件，则删除失败
[root@bigdata02 ~]# hdfs dfs -rm -R /abc
2022-08-14 20:37:53,157 WARN fs.TrashPolicyDefault: Can't create trash directory: hdfs://bigdata01:9000/user/root/.Trash/Current
org.apache.hadoop.hdfs.server.namenode.SafeModeException: Cannot create directory /user/root/.Trash/Current. Name node is in safe mode.

# 手动强制退出安全模式
[root@bigdata01 hadoop]# hdfs dfsadmin -safemode leave
Safe mode is OFF
```

### 2.0. HDFS Java 客户端操作？

#### 1）POM 依赖

```xml
<dependency>
    <groupId>org.apache.hadoop</groupId>
    <artifactId>hadoop-client</artifactId>
    <version>3.2.0</version>
</dependency>
```

#### 2）获取配置

```java
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.*;
import org.apache.hadoop.io.IOUtils;

import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;

/**
 * JAVA代码操作HDFS：上传文件、下载文件、删除文件
 *
 * @author yaocs2
 * @since 2022-08-14
 */
public class HdfsOp {

    /**
     * org.apache.hadoop.security.AccessControlException: Permission denied:
     * 解决方案：
     *
     *     <property>
     *         <name>dfs.permissions.enabled</name>
     *         <value>false</value>
     *     </property>
     *
     * @param args
     * @throws IOException
     */
    public static void main(String[] args) throws IOException {
        // 获取配置
        Configuration conf = new Configuration();
        conf.set("fs.defaultFS", "hdfs://bigdata01:9000");// 即core-site.xml中配置的fs.defaultFS

        // 获取HDFS文件系统对象
        FileSystem fileSystem = FileSystem.get(conf);

        // 上传文件
//        put(fileSystem);

        // 下载文件
//        get(fileSystem);

        // 删除文件: true表示递归删除
//        delete(fileSystem);

        // 获取文件所有block块
//        list(fileSystem);
    }
}
```

#### 3）上传文件

```java
    /**
     * 上传文件
     *
     * @param fileSystem
     * @throws IOException
     */
    private static void put(FileSystem fileSystem) throws IOException {
        FileInputStream fis = new FileInputStream("D:\\Users\\yaocs2\\Desktop\\00 test.txt");
        FSDataOutputStream fos = fileSystem.create(new Path("/00 test.txt"));
        IOUtils.copyBytes(fis, fos, 1024, true);
    }
```

#### 4）下载文件

```java
    /**
     * 下载文件
     *
     * @param fileSystem
     * @throws IOException
     */
    private static void get(FileSystem fileSystem) throws IOException {
        FSDataInputStream fis = fileSystem.open(new Path("/README.txt"));
        FileOutputStream fos = new FileOutputStream("D:\\README.txt");
        IOUtils.copyBytes(fis, fos, 1024, true);
    }
```

#### 5）删除文件

```java
    /**
     * 删除文件
     *
     * @param fileSystem
     * @throws IOException
     */
    private static void delete(FileSystem fileSystem) throws IOException {
        if (fileSystem.delete(new Path("/00 test.txt"), true)) {
            System.out.println("删除成功!");
        } else {
            System.out.println("删除失败!");
        }
    }
```

#### 6）获取文件所有 block 块

```java
    /**
     * 获取文件所有block块
     *
     * path: hdfs://bigdata01:9000/hadoop-3.2.0.tar.gz
     * 分块数量：3
     * block_0_location: bigdata02	134217728
     * block_0_location: bigdata03	134217728
     * block_1_location: bigdata03	134217728
     * block_1_location: bigdata02	134217728
     * block_2_location: bigdata02	77190019
     * block_2_location: bigdata03	77190019
     * ==========
     *
     * @param fileSystem
     * @throws IOException
     */
    private static void list(FileSystem fileSystem) throws IOException {
        RemoteIterator<LocatedFileStatus> iterator = fileSystem.listFiles(new Path("/hadoop-3.2.0.tar.gz"), true);
        while (iterator.hasNext()) {
            LocatedFileStatus fileStatus = iterator.next();
            BlockLocation[] blockLocations = fileSystem.getFileBlockLocations(fileStatus, 0, fileStatus.getLen());

            int blockLen = blockLocations.length;
            System.out.println("path: " + fileStatus.getPath().toString());
            System.out.println("分块数量：" + blockLen);

            for (int i = 0; i < blockLen; i++) {
                String[] hosts = blockLocations[i].getHosts();
                long blkLength = blockLocations[i].getLength();
                for(int j = 0; j < hosts.length; j++) {
                    System.out.println("block_" + i + "_location: " + hosts[j] + "\t" + blkLength);
                }
            }

            System.out.println("==========");
        }
    }
```

### 2.1. HDFS 架构原理？

![1660475210252](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1660475210252.png)

HDFS 主从架构：

1. 主节点称为 NameNode，可以支持多个，相当于老板。
2. 从节点称为 DataNode，也支持多个，相当于工人。
3. 以及 SecondaryNameNode 进程，相当于老板的秘书。

#### 1）NameNode

NameNode 是整个文件系统的管理节点，主要维护整个文件系统的文件目录树、文件/目录的信息、每个文件对应的数据块列表，以及负责接收用户操作请求。

NameNode 维护了 2 份关系，这样 NameNode 就可以找到对应的关系了，即 File -> Block -> DataNode 了：

1. 第一份关系是，File 与 Block list 的关系，对应的关系信息存储在 fsimage 和 edits 文件中，这些文件中的元数据信息，会在 NameNode 启动时加载到内存中。
2. 第二份关系是，DataNode 与 Block 之间的关系，当 DataNode 启动时，会把 DataNode 节点上的 Block 和节点信息，上报给 NameNode。

但是，无论是大文件还是小文件，每一份文件的这些关系数据，都会占用 NameNode 150 字节的内存空间，所以 NameNode 不适合存储小文件。

![1660476267007](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1660476267007.png)

##### 1、配置路径

```xml
<!-- hadoop-hdfs-xxx.jar#hdfs-default.xml -->
<property>
  <name>dfs.namenode.name.dir</name>
  <value>file://${hadoop.tmp.dir}/dfs/name</value>
  <description>Determines where on the local filesystem the DFS name node
      should store the name table(fsimage).  If this is a comma-delimited list
      of directories then the name table is replicated in all of the
      directories, for redundancy. </description>
</property>

<!-- hadoop-xxx/etc/hadoop/core-site.xml -->
<property>
    <name>hadoop.tmp.dir</name>
    <value>/data/hadoop_repo</value>
</property>
```

##### 2、实际文件

```shell
[root@bigdata01 current]# pwd
/data/hadoop_repo/dfs/name/current

[root@bigdata01 current]# ll
总用量 6184
-rw-r--r--. 1 root root 1048576 8月  13 21:15 edits_0000000000000000001-0000000000000000001
-rw-r--r--. 1 root root 1048576 8月  13 21:20 edits_0000000000000000002-0000000000000000002
-rw-r--r--. 1 root root 1048576 8月  13 21:28 edits_0000000000000000003-0000000000000000003
-rw-r--r--. 1 root root 1048576 8月  13 21:33 edits_0000000000000000004-0000000000000000004
-rw-r--r--. 1 root root    2838 8月  14 16:48 edits_0000000000000000005-0000000000000000042
-rw-r--r--. 1 root root     288 8月  14 17:48 edits_0000000000000000043-0000000000000000046
-rw-r--r--. 1 root root 1048576 8月  14 17:48 edits_0000000000000000047-0000000000000000047
-rw-r--r--. 1 root root      42 8月  14 18:10 edits_0000000000000000048-0000000000000000049
# edits事务文件：记录正在上传的文件信息，以及每个block的上传状态，上传完毕后，则定期合并到fsimage中，包含操作动作、事务ID、块ID、临时路径地址等信息
-rw-r--r--. 1 root root    1643 8月  14 19:10 edits_0000000000000000050-0000000000000000072
-rw-r--r--. 1 root root 1048576 8月  14 19:10 edits_inprogress_0000000000000000073
# fsimage文件和Block之间的关系：包含事务ID、类型(文件/文件夹)、名称、副本数量、修改时间、访问时间、默认块大小、权限、块ID、实际块大小、存储策略等信息
-rw-r--r--. 1 root root     875 8月  14 18:10 fsimage_0000000000000000049
-rw-r--r--. 1 root root      62 8月  14 18:10 fsimage_0000000000000000049.md5
-rw-r--r--. 1 root root    1062 8月  14 19:10 fsimage_0000000000000000072
-rw-r--r--. 1 root root      62 8月  14 19:10 fsimage_0000000000000000072.md5
# seen_txid：记录最后一个事务ID，重启时会从头读取到这个事务ID的文件，读取失败则报错
-rw-r--r--. 1 root root       3 8月  14 19:10 seen_txid
# VERSION：集群版本信息
-rw-r--r--. 1 root root     217 8月  14 15:59 VERSION
```

#### 2）SecondaryNameNode

SecondaryNameNode，主要负责定期地把 NameNode#edits 文件中的内容，合并到 NameNode#fsimage 文件中，这个合并的操作称为 checkpoint，在合并时，会对 edits 中的内容进行转换，生成新的内容保存到 NameNode#fsimage 中。

- 不过要注意的是，在 NameNode 的 HA 高可用架构中，由于同时存在主、备 NameNode，所以文件合并操作会有 StandbyNameNode 备用节点负责，此时不会有 SecondaryNameNode 进程。

#### 3）DataNode

1. DataNode，负责提供真实的文件数据存储服务，它会按照固定的大小，顺序地对文件进行划分并编号，划分好地每一块称为 Block，默认 Block 大小是 128 MB，而如果一个文件小于一个数据块的大小，那么并不会占用整个数据块的存储空间，实际多大就占多大。
2. DataNode 有两个核心概念，一个是 Block，另外一个是 Replication。
3. Replication，多副本机制，默认副本数量为 3，用于保证系统数据的高可靠。

##### 1、Block 配置路径

```xml
<!-- hadoop-hdfs-xxx.jar#hdfs-default.xml -->
<property>
  <name>dfs.datanode.data.dir</name>
  <value>file://${hadoop.tmp.dir}/dfs/data</value>
  <description>Determines where on the local filesystem an DFS data node
  should store its blocks.  If this is a comma-delimited
  list of directories, then data will be stored in all named
  directories, typically on different devices. The directories should be tagged
  with corresponding storage types ([SSD]/[DISK]/[ARCHIVE]/[RAM_DISK]) for HDFS
  storage policies. The default storage type will be DISK if the directory does
  not have a storage type tagged explicitly. Directories that do not exist will
  be created if local filesystem permission allows.
  </description>
</property>

<!-- hadoop-xxx/etc/hadoop/core-site.xml -->
<property>
    <name>hadoop.tmp.dir</name>
    <value>/data/hadoop_repo</value>
</property>
```

##### 2、Block 实际文件

```shell
[root@bigdata02 subdir0]# pwd
/data/hadoop_repo/dfs/data/current/BP-562300349-192.168.56.106-1660396543488/current/finalized/subdir0/subdir0

[root@bigdata02 subdir0]# ll
总用量 340364
-rw-r--r--. 1 root root    150569 8月  14 16:22 blk_1073741826
-rw-r--r--. 1 root root      1187 8月  14 16:22 blk_1073741826_1002.meta
-rw-r--r--. 1 root root     22125 8月  14 16:22 blk_1073741827
-rw-r--r--. 1 root root       183 8月  14 16:22 blk_1073741827_1003.meta
-rw-r--r--. 1 root root      1361 8月  14 16:22 blk_1073741828
-rw-r--r--. 1 root root        19 8月  14 16:22 blk_1073741828_1004.meta
-rw-r--r--. 1 root root 134217728 8月  14 18:30 blk_1073741830
-rw-r--r--. 1 root root   1048583 8月  14 18:30 blk_1073741830_1006.meta
-rw-r--r--. 1 root root 134217728 8月  14 18:30 blk_1073741831
-rw-r--r--. 1 root root   1048583 8月  14 18:30 blk_1073741831_1007.meta
# 存储内容
-rw-r--r--. 1 root root  77190019 8月  14 18:30 blk_1073741832
# 校验文件
-rw-r--r--. 1 root root    603055 8月  14 18:30 blk_1073741832_1008.meta
```

##### 3、Replication 配置路径

```xml
<!-- hadoop-hdfs-xxx.jar#hdfs-default.xml -->
<property>
  <name>dfs.replication</name>
  <value>3</value>
  <description>Default block replication. 
  The actual number of replications can be specified when the file is created.
  The default is used if replication is not specified in create time.
  </description>
</property>

<!-- hadoop-xxx/etc/hadoop/hdfs-site.xml -->
<property>
    <name>dfs.replication</name>
    <value>2</value>
</property>
```

#### 4）HDFS 高可用架构

1. HDFS HA，HDFS 的高可用架构，指一个集群中存在多个 NameNode，只有一个 NameNode 是 Active 状态，表示主节点，其他的都是 Standby 状态，表示备用节点。
2. 主节点负责所有的客户端操作，备用节点负责同步主节点的状态信息，以提供快速故障恢复能力。
3. 因此，在使用 HA 时，不能启动 SecondaryNameNode，否则会报错。

![1660482228528](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1660482228528.png)

#### 5）HDFS 高扩展架构

Federation 高扩展架构，可以解决单一命名空间的内存不足问题，以支持集群的高扩展性、更高效的性能、以及良好的业务隔离性。

![1660482562065](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1660482562065.png)

### 2.2. HDFS 源码解析？

#### 1）读数据过程

![1660473872633](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1660473872633.png)

从 HDFS 中，读数据的过程是这样的： 

1. 客户端向 NameNode 请求下载指定文件， NameNode 获取文件的元数据信息（主要
  是 Block 的存放位置信息），然后返回给客户端。
2. 客户端根据返回的 Block 位置信息，请求对应的 DataNode 节点，获取数据。
3. DataNode 会向客户端传输数据，客户端会以 Packet 为单位接收，先在本地缓
  存，然后写入对应的文件。 

其中，要注意的是：

1. 如果在读数据时，DFSInputStream 和 DataNode 的通讯发生异常，就会尝试正
   在读的 Block 的排第二近的 DataNode，并且会记录哪个 DataNode 发生错误，剩余的 Blocks
   读时，就会直接跳过这些坏的 DataNode 。

1. 而 DFSInputStream 也会检查 Block 数据校验和，如果发现一个坏的 Block，就会先报告到 NameNode 节点，然后 DFSInputStream 在其他的 DataNode 上读该 Block 的镜像。 

因此，设计的原则就是：

1. 客户端直接连接 DataNode 来检索数据。
2. NameNode 来负责为每一个 Block 提供最优的 DataNode，且仅仅处理 Block Location 的请求，这些信息都加载在 NameNode 的内存中，所以，HDFS 通过 DataNode 可以承受大量客户端的并发访问。 

#### 2）写数据过程

详细见《https://www.processon.com/view/link/608107fef346fb66d2d6d782》

![1660474705407](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1660474705407.png)

向 HDFS 中，写数据的过程是这样的：
1. 客户端首先向 NameNode 发送请求上传文件， NameNode 会做一些基础检查，例
  如：目标文件是否已存在，父目录是否存在等。

2. NameNode 如果检查通过，则会返回该文件的所有 Block 块，以及对应存储的 DataNode 列表，如果检查没通过，则抛异常结束。

  ```java
  blk-001:bigdata01,bigdata02,bigdata03
  blk-002:bigdata01,bigdata02,bigdata03
  ...
  ```

3. 客户端收到 Block 块，以及对应存储的 DataNode 列表之后，就开始向对应的节点中上传数据。

4. 客户端首先上传第一个 Block 块：blk-001 ，它对应的 DataNode 列表是 bigdata01、bigdata02、bigdata03，所以客户端就把 blk-001 先上传到 bigdata01，bigdata01 再把 blk-001 上传到 bigdata02 ， bigdata02 再把 blk-001 上传到 bigdata03 。

5. 客户端接着再上传第二个 Block 块： blk-002 ，流程如上。

6. 当所有的 Block 块都传完以后，客户端会给 NameNode 返回一个状态信息，表示数据已经全部写入成功，或者失败。

7. NameNode 根据客户端返回的状态信息，来判断本次写入数据是成功还是失败，如果成功，则需要对应更新元数据信息。 

### 2.3. 什么是分布式计算？

1. 移动计算：由于传统的移动数据方案，需要大规模地将数据向网络中传输，非常耗时，所以可以反过来，将计算程序移动到数据所在地节点，以节约网络 I/O 带来的损耗。
2. 分布式计算的方案则是，通过局部节点进行移动计算，得出局部的结果，最终在把分散的局部结果汇总起来，得到最终的结果。

![1660648984985](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1660648984985.png)

### 2.4. 什么是 MapReduce？

1. MapReduce，是一种分布式计算模型，由 Google 提出，主要用于搜索领域，解决海量数据的计算问题。
2. MapReduce，由两个阶段组成：
   - Map：第一阶段，进行分布式移动计算。
   - Reduce：第二阶段，非必须，把分布在各个节点的局部结果，进行最终的全局汇总。

![1660651802579](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1660651802579.png)

### 2.5. MapReduce 执行原理？

![1660650097690](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1660650097690.png)

```
hello.txt

hello you
hello me
```

#### 1）Map 阶段

1. MapReduce 把输入文件/文件夹，划分为很多的 InputSplit，默认情况下，一个 Block 对应一个 InputSplit，大小为 128 MB。然后通过 `RecordReader` 类把每个 InputSplit 解析成一个个 `<k1, v1>`，默认情况下，每行数据会被解析成一个 `<k1, v1>`。

   ```
   读取：
   <0, hello you>
   <10, hello me>
   ```

2. 一个 InputSplit 对应一个 Map Task，MapReduce 调用 `Mapper` 类中的 `map(..)` 函数，把输入的 `<k1, v1>` 输出成 `<k2, v2>`，其中，这个 `map(..)` 函数正是程序员需要编写的业务逻辑。

   ```
   分割：
   <hello, 1>
   <you, 1>
   <hello, 1>
   <me, 1>
   ```

3. MapReduce 对 `map(..)` 函数输出的 `<k2, v2>` 进行分区，默认只有一个分区，不同分区中的 `<k2, v2>` 在以后的 Reduce 阶段由不同的 Reduce Task 处理。

   ```
   分区：
   <hello, 1>
   <you, 1>
   <hello, 1>
   <me, 1>
   ```

4. MapReduce 对每个分区中的数据，按照 k2 进行排序、分组（指将相同 k2 的 v2 划分为同一组）。

   ```
   先按字母顺序排序：
   <hello, 1>
   <hello, 1>
   <me, 1>
   <you, 1>
   
   再分组：
   <hello, {1,1}>
   <me, {1}>
   <you, {1}>
   ```

5. 【可选，默认不执行】MapReduce 执行 Combiner 规约操作，以支持局部的聚合。

6. MapReduce 把 Map Task 输出的 `<k2, v2>` 写入 Linux 的磁盘文件，至此，整个 Map 阶段执行结束。

   ```
   写入Linux磁盘文件：
   <hello, {1,1}>
   <me, {1}>
   <you, {1}>
   ```

#### 2）Reduce 阶段（非必须）

1. MapReduce 对多个 Map Task 的输出，按照不同的分区，通过网络 I/O Copy 到不同的 Reduce 节点，这个过程称作 Shuffle。

   ```
   Shuffle：
   <hello, {1,1}>
   <me, {1}>
   <you, {1}>
   ```

2. MapReduce 对 Reduce 节点接收到的、相同分区的 `<k2, v2>` 数据，进行合并、排序、分组。

   ```
   获取分区数据、合并、排序、分组，虽然步骤与Map相似，但Map阶段是局部的，Reduce阶段是全局的：
   <k2, {v2...}>
   <hello, {1,1}>
   <me, {1}>
   <you, {1}>
   ```

3. MapReduce 调用 `Reducer` 类中的 `reduce(..)` 方法，把输入的 `<k2, {v2..}>` 输出成 `<k3, v3>`，其中，一个 `<k2, {v2..}>`  会调用一次 `reduce(..)` 方法，而该方法也正是程序员需要编写的业务逻辑。

   ```
   reduce统计：
   <hello, 2>
   <me, 1>
   <you, 1>
   ```

4. MapReduce 把 Reduce 的输出结果，保存到 HDFS 中，至此，整个 Reduce 阶段执行结束。

   ```
   输出到HDFS：
   hello 2
   me 1
   you 1
   ```

#### 3）单文件单词计数

详细数据，见上述的分析过程

![1660652518960](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1660652518960.png)

#### 4）多文件单词计数

1、

![1660652659029](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1660652659029.png)

2、

![1660652694068](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1660652694068.png)

#### 5）WordCount | Hello world

##### 1、Mapper

```java
    /**
     * Map阶段
     *
     * @author yaocs2
     * @since 2022-08-16
     */
    public static class MyMapper extends Mapper<LongWritable, Text, Text, LongWritable> {

        /**
         * Map函数：<k1, v1> => <k2, v2>
         *
         * @param k1      每行数据的行首偏移量
         * @param v1      每行的数据内容
         * @param context
         * @throws IOException
         * @throws InterruptedException
         */
        @Override
        protected void map(LongWritable k1, Text v1, Context context) throws IOException, InterruptedException {
            // 切割字符串
            String[] words = v1.toString().split(" ");

            // <k1, v1> => <k2, v2>
            for (String word : words) {
                Text k2 = new Text(word);
                LongWritable v2 = new LongWritable(1L);
                context.write(k2, v2);
            }
        }
    }
```

##### 2、Reducer

```java
    /**
     * Reduce阶段
     *
     * @author yaocs2
     * @since 2022-08-16
     */
    public static class MyReducer extends Reducer<Text, LongWritable, Text, LongWritable> {

        /**
         * Reduce函数：<k2, {v2,..}> => <k3, v3>
         *
         * @param k2      单词的值
         * @param v2s     单词出现的所有次数
         * @param context
         * @throws IOException
         * @throws InterruptedException
         */
        @Override
        protected void reduce(Text k2, Iterable<LongWritable> v2s, Context context) throws IOException, InterruptedException {
            // 累加所有次数
            long sum = 0L;
            for (LongWritable v2 : v2s) {
                sum += v2.get();
            }

            // <k2, {v2,..}> => <k3, v3>
            Text k3 = k2;
            LongWritable v3 = new LongWritable(sum);
            context.write(k3, v3);
        }
    }
```

##### 3、Job

```java
/**
 * 单词计数：统计每个单词出现的总次数
 * <p>
 * hello you
 * hello me
 * <p>
 * =>
 * <p>
 * hello    2
 * me   1
 * you  1
 *
 * @author yaocs2
 * @since 2022-08-16
 */
public class WordCountJob {

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

        try {
            Configuration conf = new Configuration();
            Job job = Job.getInstance(conf);
            job.setJarByClass(WordCountJob.class);// 必须设置, 否则提交到集群后, Job会找不到这个WordCountJob类的

            // 输入、输出
            FileInputFormat.setInputPaths(job, new Path(fileInputPath));
            FileOutputFormat.setOutputPath(job, new Path(fileOutputPath));

            // Map
            job.setMapperClass(MyMapper.class);
            job.setMapOutputKeyClass(Text.class);
            job.setMapOutputValueClass(LongWritable.class);

            // Reduce
            job.setReducerClass(MyReducer.class);
            job.setOutputKeyClass(Text.class);
            job.setOutputValueClass(LongWritable.class);

            // 提交Job
            job.waitForCompletion(true);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

##### 4、提交执行

```shell
# 原始文件
[root@bigdata01 ~]# hdfs dfs -ls /test
Found 1 items
-rw-r--r--   2 root supergroup         19 2022-08-16 21:12 /test/hello.txt
[root@bigdata01 ~]# hdfs dfs -cat /test/hello.txt
hello you
hello me

# 提交程序到集群中执行
[root@bigdata01 ~]# hadoop jar bigdata-1.0-SNAPSHOT-jar-with-dependencies.jar com.jsonyao.mr.WordCountJob /test/hello.txt /out

# 查看执行结果
[root@bigdata01 ~]# hdfs dfs -ls /out
Found 2 items
-rw-r--r--   2 root supergroup          0 2022-08-16 21:14 /out/_SUCCESS
-rw-r--r--   2 root supergroup         19 2022-08-16 21:14 /out/part-r-00000
[root@bigdata01 ~]# hdfs dfs -cat /out/part-r-00000
hello   2
me      1
you     1

# 查看MR执行日志
# 1、http://bigdata01:8088/浏览器界面查看，不过需要启动historyServer
[root@bigdata01 ~]# mapred --daemon start historyserver
# 2、命令行查看
[root@bigdata01 ~]# yarn logs -applicationId application_1660793117360_0003

# 提前停止MR任务（已经提交到Hadoop集群后）
[root@bigdata02 ~]# yarn application -kill application_1660793117360_0005
2022-08-18 13:59:44,626 INFO client.RMProxy: Connecting to ResourceManager at bigdata01/192.168.56.106:8032
Killing application application_1660793117360_0005
2022-08-18 13:59:45,155 INFO impl.YarnClientImpl: Killed application application_1660793117360_0005
```

#### 6）Shuffle 机制

Shuffle，是 MapReduce 中网络拷贝的过程，具体地，是通过网络把数据从 Map 端，拷贝到 Reduce 端的一个过程，Map 中相同分区的数据会被拷贝到同一个 Reduce 节点。

![1660803289631](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1660803289631.png)

#### 7）序列化机制

![1660804403856](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1660804403856.png)

1. 影响 MapReduce 的性能有两点，一是磁盘 I/O，二是网络 I/O。
2. 磁盘由每个分布式节点负责，网络 I/O 由 Shuffle 负责。
3. 对于网络 I/O 和磁盘 I/O，优化出性能高的序列化机制， 则可以大大提升 MapReduce 整个过程的性能。

而 Java 中序列化机制存在以下不足：

1. 不精简，附件信息多，不太适合随机访问
2. 存储空间大，需要递归地输出每个层级的所有超类描述，直到不再有超类为止

因此，MapReduce 提供了自己实现的数据类型，其特点有：

1. 紧凑：高效使用存储空间
2. 快速：读写数据的额外开销小
3. 可扩展：可透明地读取老格式地数据
4. 可交户：支持多语言的交互操作

| Java 基本数据类型                   | Writable 实现类                  | 序列化大小（byte） |
| ----------------------------------- | -------------------------------- | ------------------ |
| 布尔型 boolean                      | BooleanWritable                  | 1                  |
| 字节型 byte                         | ByteWritable                     | 1                  |
| 整型 int                            | IntWritable                      | 4                  |
|                                     | VIntWritable                     | 1~5                |
| 浮点型 float                        | FloatWritable                    | 4                  |
| 长整型 long                         | LongWritable                     | 8                  |
|                                     | VLongWritable                    | 1~9                |
| 双精度浮点型 double                 | DoubleWritable                   | 8                  |
| 字符串类型 String（非基本数据类型） | Text （UTF，非 Writable 实现类） | -                  |
| -                                   | NullWritable.get()               | ?                  |

#### 8）InputFormat

![1660810386790](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1660810386790.png)

1. 在 Map 阶段，MapReduce 会把输入文件/文件夹，划分为很多的 InputSplit，默认情况下，一个 Block 对应一个 InputSplit，大小为 128 MB。

   - 1）其中，实现类 `FileInputFormat#getSplits()`方法，用于划分原始数据为 InputSplit，源码规定了只有当 `remaing > 128 * 1.1 = 140.8 mb` 时，才会再划分为一个 InputSplit。
   - 2）所以，一个 140 mb 的文件只会被划分为 1 个 InputSplit，即 1 个 Map Task。
   - 3）而 1000 个 100 kb 小文件，则会被划分为 1000 个 InputSplit，即 1000 个 Map Task。

2. 然后，通过 `RecordReader` 类把每个 InputSplit 解析成一个个 `<k1, v1>`，默认情况下，每行数据会被解析成一个 `<k1, v1>`。

   - 1）其中，`LineRecordReader` 每次读取一个 InputSplit 时，都会往下多读取一行数据，即读到了下一个 InputSplit 的数据。
   - 2）并且，如果要读取的 InputSplit 不是第一个 InputSplit，则会放弃第一行数据的读取，直接跳到第二行读取。
   - 3）这样，由于每次多读 1 行、非首个 InputSplit 放弃读取首行，所以从整体来看，即使一个文件被无规则地切分成了多个 InputSplit，`LineRecordReader` 都会无遗漏地、完整地读取出每一行文本，并组装成 `<k1, v1>`，只不过读每个 InputSplit 产生的文本内容大小不同罢了。

   ![1660811652312](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1660811652312.png)

3. 接着，一个 InputSplit 对应一个 Map Task，MapReduce 调用 `Mapper` 类中的 `map(..)` 函数，把输入的 `<k1, v1>` 输出成 `<k2, v2>`，其中，这个 `map(..)` 函数正是程序员需要编写的业务逻辑。

4. ...

#### 9）OutputFormat

![1660812260619](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1660812260619.png)

TextOutputFormat，输出 `<k1, v1>` 时，会读取 `"mapreduce.output.textoutputformat.separator"` 作为分隔符，如果没有配置，则默认以 `/t` 作为分隔符。

#### 10）性能优化

##### 1、小文件问题

小文件问题，指的是，Hadoop 的 HDFS 和 MapReduce，是针对大数据文件来设计的，在小文件的处理上，不但效率低下，还十分消耗内存资源。

因此，HDFS 提供了两种解决方案：

###### ① SequeceFile

1. SequeceFile，是 Hadoop 提供的一种二进制文件，这种二进制文件直接将 `<key, value>` 对序列化到文件中，所以不支持直接查看。
2. 一般情况下，小文件可以使用这种方式来进行合并，将文件名作为 key，文件内容作为 value，然后序列化到大文件中。
3. 但是，合并文件需要一个过程，且合并后的文件不方便查看，必须通过遍历的方式，查看对应文件名的小文件内容。

```java
/**
 * 小文件解决方案之 SequenceFile
 *
 * @author yaocs2
 * @since 2022-08-19
 */
public class SmallFileSeq {

    public static void main(String[] args) throws IOException {
        String hdfsFilePath = "/seqFile";
        // 合并inputDir下所有的小文件, 到HDFS#outputFile文件中
        write("D:\\Users\\yaocs2\\data\\myWorkspace\\imooc_bigdata\\bigdata_course_materials\\hadoop\\mapreduce+yarn\\smallFile", hdfsFilePath);

        // 读取HDFS中合并后的inputFile文件
        read(hdfsFilePath);
    }

    /**
     * 合并inputDir下所有的小文件, 到HDFS#outputFile文件中
     *
     * @param inputDir
     * @param outputFile
     * @throws IOException
     */
    private static void write(String inputDir, String outputFile) throws IOException {
        Configuration conf = new Configuration();
        conf.set("fs.defaultFS", "hdfs://bigdata01:9000");

        // 先删除HDFS中合并后的文件
        FileSystem fileSystem = FileSystem.get(conf);
        fileSystem.delete(new Path(outputFile), true);

        /**
         * 构造options数组：
         *
         * a. 输出路径
         * b. key的类型
         * c. value的类型
         */
        SequenceFile.Writer.Option[] options = new SequenceFile.Writer.Option[] {
                SequenceFile.Writer.file(new Path(outputFile)),
                SequenceFile.Writer.keyClass(Text.class),
                SequenceFile.Writer.valueClass(Text.class)
        };

        SequenceFile.Writer writer = SequenceFile.createWriter(conf, options);

        // 读取指定目录下的所有小文件
        File inputDirPath = new File(inputDir);
        if(inputDirPath.isDirectory()) {
            File[] files = inputDirPath.listFiles();
            for (File file : files) {
                String fileName = file.getName();
                String content = FileUtils.readFileToString(file, "UTF-8");

                // 把合并后的文件写到HDFS中
                Text key = new Text(fileName);
                Text value = new Text(content);
                writer.append(key, value);
            }
        }

        // 关闭流
        writer.close();
    }

    /**
     * 读取HDFS中合并后的inputFile文件
     *
     * @param inputFile
     * @throws IOException
     */
    private static void read(String inputFile) throws IOException {
        Configuration conf = new Configuration();
        conf.set("fs.defaultFS", "hdfs://bigdata01:9000");

        // 创建阅读器
        SequenceFile.Reader reader = new SequenceFile.Reader(conf, SequenceFile.Reader.file(new Path(inputFile)));

        // 创建缓冲区
        Text key = new Text();
        Text value = new Text();

        // 开始读取
        while (reader.next(key, value)) {
            System.out.print(String.format("文件名: %s, ", key.toString()));
            System.out.println(String.format("文件内容: %s", value.toString()));
        }

        // 关闭流
        reader.close();
    }
}

/**
输出结果：
文件名: file1.txt, 文件内容: hello you
文件名: file10.txt, 文件内容: hello you
文件名: file2.txt, 文件内容: hello you
文件名: file3.txt, 文件内容: hello you
文件名: file4.txt, 文件内容: hello you
文件名: file5.txt, 文件内容: hello you
文件名: file6.txt, 文件内容: hello you
文件名: file7.txt, 文件内容: hello you
文件名: file8.txt, 文件内容: hello you
文件名: file9.txt, 文件内容: hello you
 */

// MapReduce应用
//    /**
//     * 组装Job = Map + Reduce
//     *
//     * @param args
//     */
//    public static void main(String[] args) {
//        if (args.length < 2) {
//            System.exit(100);
//        }
//
//        String fileInputPath = args[0];
//        String fileOutputPath = args[1];
//
//        try {
//            Configuration conf = new Configuration();
//            Job job = Job.getInstance(conf);
//            job.setJarByClass(WordCountJobSmallFile.class);// 必须设置, 否则提交到集群后, Job会找不到这个WordCountJob类的
//
//            // 输入、输出
//            FileInputFormat.setInputPaths(job, new Path(fileInputPath));
//            FileOutputFormat.setOutputPath(job, new Path(fileOutputPath));
//
//            // 设置小文件合并优化处理类
//            job.setInputFormatClass(SequenceFileInputFormat.class);
//
//            // Map
//            job.setMapperClass(MyMapper.class);
//            job.setMapOutputKeyClass(Text.class);
//            job.setMapOutputValueClass(LongWritable.class);
//
//            // Reduce
//            job.setReducerClass(MyReducer.class);
//            job.setOutputKeyClass(Text.class);
//            job.setOutputValueClass(LongWritable.class);
//
//            // 提交Job
//            job.waitForCompletion(true);
//        } catch (Exception e) {
//            e.printStackTrace();
//        }
//    }
```

###### ② MapFile

1. MapFile，是排序后的 SequenceFile，由两部分组成，分别是 index 和 data。
2. index，是文件的数据索引，主要记录了每个 Record 的 key 值，以及该 Record 在文件中的偏移，
3. 在 MapFile 被访问时，索引文件会被加载到内存中，根据 key 进行随机访问时，就可以根据索引，迅速找到对应的偏移量，再通过偏移量，直接访问 data 文件中的 Record 数据了。
4. 但是，索引数据被加载，需要消耗一定的内存资源。

```java
/**
 * 小文件解决方案之 MapFile
 *
 * @author yaocs2
 * @since 2022-08-19
 */
public class SmallFileMap {

    public static void main(String[] args) throws IOException {
        String hdfsFilePath = "/mapFile";
        // 合并inputDir下所有的小文件, 到HDFS#outputFile文件中
        write("D:\\Users\\yaocs2\\data\\myWorkspace\\imooc_bigdata\\bigdata_course_materials\\hadoop\\mapreduce+yarn\\smallFile", hdfsFilePath);

        // 读取HDFS中合并后的inputFile文件
        read(hdfsFilePath);
    }

    /**
     * 合并inputDir下所有的小文件, 到HDFS#outputFile文件中
     *
     * @param inputDir
     * @param outputDir
     * @throws IOException
     */
    private static void write(String inputDir, String outputDir) throws IOException {
        Configuration conf = new Configuration();
        conf.set("fs.defaultFS", "hdfs://bigdata01:9000");

        // 先删除HDFS中合并后的文件
        FileSystem fileSystem = FileSystem.get(conf);
        fileSystem.delete(new Path(outputDir), true);

        /**
         * 构造options数组：
         *
         * a. key的类型
         * b. value的类型
         */
        SequenceFile.Writer.Option[] options = new SequenceFile.Writer.Option[]{
                MapFile.Writer.keyClass(Text.class),
                MapFile.Writer.valueClass(Text.class)
        };

        MapFile.Writer writer = new MapFile.Writer(conf, new Path(outputDir), options);

        // 读取指定目录下的所有小文件
        File inputDirPath = new File(inputDir);
        if (inputDirPath.isDirectory()) {
            File[] files = inputDirPath.listFiles();
            for (File file : files) {
                String fileName = file.getName();
                String content = FileUtils.readFileToString(file, "UTF-8");

                // 把合并后的文件写到HDFS中
                Text key = new Text(fileName);
                Text value = new Text(content);
                writer.append(key, value);
            }
        }

        // 关闭流
        writer.close();
    }

    /**
     * 读取HDFS中合并后的inputFile文件
     *
     * @param inputDir
     * @throws IOException
     */
    private static void read(String inputDir) throws IOException {
        Configuration conf = new Configuration();
        conf.set("fs.defaultFS", "hdfs://bigdata01:9000");

        // 创建阅读器
        MapFile.Reader reader = new MapFile.Reader(new Path(inputDir), conf);

        // 创建缓冲区
        Text key = new Text();
        Text value = new Text();

        // 开始读取
        while (reader.next(key, value)) {
            System.out.print(String.format("文件名: %s, ", key.toString()));
            System.out.println(String.format("文件内容: %s", value.toString()));
        }

        // 关闭流
        reader.close();
    }
}

/**
输出结果：
文件名: file1.txt, 文件内容: hello you
文件名: file10.txt, 文件内容: hello you
文件名: file2.txt, 文件内容: hello you
文件名: file3.txt, 文件内容: hello you
文件名: file4.txt, 文件内容: hello you
文件名: file5.txt, 文件内容: hello you
文件名: file6.txt, 文件内容: hello you
文件名: file7.txt, 文件内容: hello you
文件名: file8.txt, 文件内容: hello you
文件名: file9.txt, 文件内容: hello you
 */
```

##### 2、数据倾斜问题

数据倾斜问题，指的是，MapReduce 执行时，Reduce 节点大部分都已执行完毕，但是有一个或者几个 Reduce 节点运行很慢，导致整个程序处理时间变得很长，具体的表现为：Reduce 阶段一直卡着某个百分比不动...

对此，解决方案有 2 个：

###### ① 增加 Reduce 任务的个数

治标不治本，这对于数据倾斜特别严重的数据，可能起不到任何作用。

```java
/**
 * 数据倾斜-解决方案1：增加Reduce个数
 *
 * @author yaocs2
 * @since 2022-08-16
 */
public class WordCountJobSkewAddReduces {

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
        String reduceTaskNum = args[2];

        try {
            Configuration conf = new Configuration();
            Job job = Job.getInstance(conf);
            job.setJarByClass(WordCountJobSkewAddReduces.class);// 必须设置, 否则提交到集群后, Job会找不到这个WordCountJob类的

            // 输入、输出
            FileInputFormat.setInputPaths(job, new Path(fileInputPath));
            FileOutputFormat.setOutputPath(job, new Path(fileOutputPath));

            // Map
            job.setMapperClass(MyMapper.class);
            job.setMapOutputKeyClass(Text.class);
            job.setMapOutputValueClass(LongWritable.class);

            // Reduce
            job.setReducerClass(MyReducer.class);
            job.setOutputKeyClass(Text.class);
            job.setOutputValueClass(LongWritable.class);

            // 设置Reduce任务个数(默认为1)
            job.setNumReduceTasks(Integer.parseInt(reduceTaskNum));

            // 提交Job
            job.waitForCompletion(true);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
     * Map阶段
     *
     * @author yaocs2
     * @since 2022-08-16
     */
    public static class MyMapper extends Mapper<LongWritable, Text, Text, LongWritable> {

        private static final Logger logger = LoggerFactory.getLogger(MyMapper.class);

        /**
         * Map函数：<k1, v1> => <k2, v2>
         *
         * @param k1      每行数据的行首偏移量
         * @param v1      每行的数据内容
         * @param context
         * @throws IOException
         * @throws InterruptedException
         */
        @Override
        protected void map(LongWritable k1, Text v1, Context context) throws IOException, InterruptedException {
            logger.info(String.format("<k1, v1> = <%s, %s>", k1.get(), v1));

            // 切割字符串
            String[] words = v1.toString().split(" ");

            // <k1, v1> => <k2, v2>：只取每行的第一个单词
            Text k2 = new Text(words[0]);
            LongWritable v2 = new LongWritable(1L);
            context.write(k2, v2);
        }
    }

    /**
     * Reduce阶段
     *
     * @author yaocs2
     * @since 2022-08-16
     */
    public static class MyReducer extends Reducer<Text, LongWritable, Text, LongWritable> {

        private static final Logger logger = LoggerFactory.getLogger(MyReducer.class);

        /**
         * Reduce函数：<k2, {v2,..}> => <k3, v3>
         *
         * @param k2      单词的值
         * @param v2s     单词出现的所有次数
         * @param context
         * @throws IOException
         * @throws InterruptedException
         */
        @Override
        protected void reduce(Text k2, Iterable<LongWritable> v2s, Context context) throws IOException, InterruptedException {
            // 累加所有次数
            long sum = 0L;
            for (LongWritable v2 : v2s) {
                logger.info(String.format("<k2, v2> = <%s, %s>", k2.toString(), v2));
                sum += v2.get();

                // 模拟Reduce复杂计算所消耗的时间
                if(sum % 200 == 0) {
                    Thread.sleep(1);
                }
            }

            // <k2, {v2,..}> => <k3, v3>
            Text k3 = k2;
            LongWritable v3 = new LongWritable(sum);
            logger.info(String.format("<k1, v1> = <%s, %s>", k3.toString(), v3));
            context.write(k3, v3);
        }
    }
}

// 提交执行（1个Reduce任务）：hadoop jar bigdata-1.0-SNAPSHOT-jar-with-dependencies.jar com.jsonyao.mr.WordCountJobSkewAddReduces /hello_10000000.dat /out10000000 1
// yarn界面，分析执行过程：整体耗时3mins, 7sec，最大Reduce耗时1mins, 40sec

// 提交执行（10个Reduce任务）：hadoop jar bigdata-1.0-SNAPSHOT-jar-with-dependencies.jar com.jsonyao.mr.WordCountJobSkewAddReduces /hello_10000000.dat /out10000000 10
// yarn界面，分析执行过程：整体耗时3mins, 19sec，最大Reduce耗时2mins, 10sec

// 查看执行结果：
/**
[root@bigdata01 ~]# hdfs dfs -cat /out10000000/*
1       100000
10      100000
2       100000
3       100000
4       100000
5       9100000
6       100000
7       100000
8       100000
9       100000
*/

// 结论：对于key=5这种倾斜非常严重的数据，由于默认采用的HashPartitioner，是取key#hashCode，再%numReduceTasks的分区策略，导致key=5的数据，都进入到了同一个Reduce Task中，导致该Task成为了Reduce阶段的性能瓶颈，因此此时再单纯地增加Reduce任务个数，已经是负提升了~
```

###### ② 把倾斜的数据打散

抽样取出倾斜严重的数据，然后将其打散。

```java
/**
 * 数据倾斜-解决方案2：打散倾斜数据
 *
 * @author yaocs2
 * @since 2022-08-16
 */
public class WordCountJobSkewRandomKey {

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
        String reduceTaskNum = args[2];

        try {
            Configuration conf = new Configuration();
            Job job = Job.getInstance(conf);
            job.setJarByClass(WordCountJobSkewRandomKey.class);// 必须设置, 否则提交到集群后, Job会找不到这个WordCountJob类的

            // 输入、输出
            FileInputFormat.setInputPaths(job, new Path(fileInputPath));
            FileOutputFormat.setOutputPath(job, new Path(fileOutputPath));

            // Map
            job.setMapperClass(MyMapper.class);
            job.setMapOutputKeyClass(Text.class);
            job.setMapOutputValueClass(LongWritable.class);

            // Reduce
            job.setReducerClass(MyReducer.class);
            job.setOutputKeyClass(Text.class);
            job.setOutputValueClass(LongWritable.class);

            // 设置Reduce任务个数(默认为1)
            job.setNumReduceTasks(Integer.parseInt(reduceTaskNum));

            // 提交Job
            job.waitForCompletion(true);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
     * Map阶段
     *
     * @author yaocs2
     * @since 2022-08-16
     */
    public static class MyMapper extends Mapper<LongWritable, Text, Text, LongWritable> {

        private static final Logger logger = LoggerFactory.getLogger(MyMapper.class);

        /**
         * Map函数：<k1, v1> => <k2, v2>
         *
         * @param k1      每行数据的行首偏移量
         * @param v1      每行的数据内容
         * @param context
         * @throws IOException
         * @throws InterruptedException
         */
        @Override
        protected void map(LongWritable k1, Text v1, Context context) throws IOException, InterruptedException {
            logger.info(String.format("<k1, v1> = <%s, %s>", k1.get(), v1));

            // 切割字符串
            String[] words = v1.toString().split(" ");

            // 打散倾斜数据: 这里是假设, 已经抽样知道了5这个key倾斜最严重
            String key = words[0];
            if("5".equals(key)) {
                key = "5" + "_" + ThreadLocalRandom.current().nextInt(10);
            }

            // <k1, v1> => <k2, v2>：只取每行的第一个单词
            Text k2 = new Text(key);
            LongWritable v2 = new LongWritable(1L);
            context.write(k2, v2);
        }
    }

    /**
     * Reduce阶段
     *
     * @author yaocs2
     * @since 2022-08-16
     */
    public static class MyReducer extends Reducer<Text, LongWritable, Text, LongWritable> {

        private static final Logger logger = LoggerFactory.getLogger(MyReducer.class);

        /**
         * Reduce函数：<k2, {v2,..}> => <k3, v3>
         *
         * @param k2      单词的值
         * @param v2s     单词出现的所有次数
         * @param context
         * @throws IOException
         * @throws InterruptedException
         */
        @Override
        protected void reduce(Text k2, Iterable<LongWritable> v2s, Context context) throws IOException, InterruptedException {
            // 累加所有次数
            long sum = 0L;
            for (LongWritable v2 : v2s) {
                logger.info(String.format("<k2, v2> = <%s, %s>", k2.toString(), v2));
                sum += v2.get();

                // 模拟Reduce复杂计算所消耗的时间
                if(sum % 200 == 0) {
                    Thread.sleep(1);
                }
            }

            // <k2, {v2,..}> => <k3, v3>
            Text k3 = k2;
            LongWritable v3 = new LongWritable(sum);
            logger.info(String.format("<k1, v1> = <%s, %s>", k3.toString(), v3));
            context.write(k3, v3);
        }
    }
}

// 提交执行（10个Reduce任务）：hadoop jar bigdata-1.0-SNAPSHOT-jar-with-dependencies.jar com.jsonyao.mr.WordCountJobSkewRandomKey /hello_10000000.dat /out10000000 10
// yarn界面，分析执行过程：整体耗时1mins, 54sec，最大Reduce耗时53sec

// 查看执行结果：
/**
[root@bigdata01 ~]# hdfs dfs -cat /out10000000/*
1       100000
5_3     909907
2       100000
5_4     909942
3       100000
5_5     910130
4       100000
5_6     907714
5_7     908168
5_8     910539
6       100000
5_9     909887
7       100000
5_0     910867
8       100000
10      100000
5_1     910784
9       100000
5_2     912062
*/

// 结论：由于对key=5这种倾斜严重的数据，进行了打散+再哈希，导致了最终出现的结果不是想要的，此时需要再起一个MapReduce任务，对/out1000000里面的数据进行再次单词统计

/**
 * 数据倾斜-解决方案2：打散倾斜数据后, 再次聚合
 *
 * @author yaocs2
 * @since 2022-08-16
 */
public class WordCountJobSkewReduceAgain {

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
        String reduceTaskNum = args[2];

        try {
            Configuration conf = new Configuration();
            Job job = Job.getInstance(conf);
            job.setJarByClass(WordCountJobSkewReduceAgain.class);// 必须设置, 否则提交到集群后, Job会找不到这个WordCountJob类的

            // 输入、输出
            FileInputFormat.setInputPaths(job, new Path(fileInputPath));
            FileOutputFormat.setOutputPath(job, new Path(fileOutputPath));

            // Map
            job.setMapperClass(MyMapper.class);
            job.setMapOutputKeyClass(Text.class);
            job.setMapOutputValueClass(LongWritable.class);

            // Reduce
            job.setReducerClass(MyReducer.class);
            job.setOutputKeyClass(Text.class);
            job.setOutputValueClass(LongWritable.class);

            // 设置Reduce任务个数(默认为1)
            job.setNumReduceTasks(Integer.parseInt(reduceTaskNum));

            // 提交Job
            job.waitForCompletion(true);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
     * Map阶段
     *
     * @author yaocs2
     * @since 2022-08-16
     */
    public static class MyMapper extends Mapper<LongWritable, Text, Text, LongWritable> {

        private static final Logger logger = LoggerFactory.getLogger(MyMapper.class);

        /**
         * Map函数：<k1, v1> => <k2, v2>
         *
         * @param k1      每行数据的行首偏移量
         * @param v1      每行的数据内容
         * @param context
         * @throws IOException
         * @throws InterruptedException
         */
        @Override
        protected void map(LongWritable k1, Text v1, Context context) throws IOException, InterruptedException {
            logger.info(String.format("<k1, v1> = <%s, %s>", k1.get(), v1));

            // 切割字符串
            String[] words = v1.toString().split("\t");

            // 切割5_?
            String[] splits = words[0].split("_");
            String key = splits[0];
            String num = words[1];

            // <k1, v1> => <k2, v2>：只取每行的第一个单词
            Text k2 = new Text(key);
            LongWritable v2 = new LongWritable(Long.parseLong(num));
            context.write(k2, v2);
        }
    }

    /**
     * Reduce阶段
     *
     * @author yaocs2
     * @since 2022-08-16
     */
    public static class MyReducer extends Reducer<Text, LongWritable, Text, LongWritable> {

        private static final Logger logger = LoggerFactory.getLogger(MyReducer.class);

        /**
         * Reduce函数：<k2, {v2,..}> => <k3, v3>
         *
         * @param k2      单词的值
         * @param v2s     单词出现的所有次数
         * @param context
         * @throws IOException
         * @throws InterruptedException
         */
        @Override
        protected void reduce(Text k2, Iterable<LongWritable> v2s, Context context) throws IOException, InterruptedException {
            // 累加所有次数
            long sum = 0L;
            for (LongWritable v2 : v2s) {
                logger.info(String.format("<k2, v2> = <%s, %s>", k2.toString(), v2));
                sum += v2.get();

                // 模拟Reduce复杂计算所消耗的时间
                if(sum % 200 == 0) {
                    Thread.sleep(1);
                }
            }

            // <k2, {v2,..}> => <k3, v3>
            Text k3 = k2;
            LongWritable v3 = new LongWritable(sum);
            logger.info(String.format("<k1, v1> = <%s, %s>", k3.toString(), v3));
            context.write(k3, v3);
        }
    }
}

// 提交执行（10个Reduce任务）：hadoop jar bigdata-1.0-SNAPSHOT-jar-with-dependencies.jar com.jsonyao.mr.WordCountJobSkewReduceAgain /out10000000 /out10000000_reduce 1
// yarn界面，分析执行过程：整体耗时20sec，最大Reduce耗时2sec

// 查看执行结果：
/**
[root@bigdata01 ~]# hdfs dfs -cat /out10000000_reduce/*
1       100000
10      100000
2       100000
3       100000
4       100000
5       9100000
6       100000
7       100000
8       100000
9       100000
*/

// 结论：该方案先针对性打散+再次聚合，整体耗时=1mins, 54sec+20sec=2min14sec，小于方案1的3mins, 19sec，性能提升了接近33%，因此，对于数据倾斜严重的数据，需要执行方案2，即先针对性打散+再次聚合结果
```

### 2.6. 什么是 Yarn？

Yarn，是 Hadoop 2.0 以后剥离出来的，一个实现 Hadoop 集群资源共享组件，不仅仅支持 MapReduce，还支持 Spark、Flink 等计算引擎。

### 2.7. Yarn 架构原理？

#### 1）架构模型

![1660394757523](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1660394757523.png)

Yarn，主要负责集群集群资源的管理和调度，支持主从架构，其中主节点最多可以有 2 个，从节点可以有多个。

1. Resource Manager：位于主节点，负责集群资源的分配和管理，主要管理内存和 CPU 这两种资源类型。
2. Node Manager：位于从节点，负责当前机器的资源管理，并在启动时会向 Resource Manager 注册，其中注册信息就包含当前机器可分配的内存和 CPU 总量。

#### 2）资源调度器

![1660982921576](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1660982921576.png)

##### 1、FIFO Scheduler

先进先出，只有一个队列，所有任务需要按顺序排队。

##### 2、Capacity Scheduler

多队列，每一个队列都初始化分配了一定的资源，队内都是先进先出，是 Yarn 目前默认的调度策略。

- 比如：A 用于分配一般任务，B 用于分配优先级较高的任务，但在队内任务还是需要按顺序排序。

###### ① 队列配置

```xml
  <property>
    <name>yarn.scheduler.capacity.root.queues</name>
    <value>default,online,offline</value>
    <description>队列列表,多个队列之间使用逗号分割</description>
  </property>

  <property>
    <name>yarn.scheduler.capacity.root.default.capacity</name>
    <value>70</value>
    <description>default队列70%</description>
  </property>
  <property>
    <name>yarn.scheduler.capacity.root.online.capacity</name>
    <value>10</value>
    <description>online队列10%</description>
  </property>
  <property>
    <name>yarn.scheduler.capacity.root.offline.capacity</name>
    <value>20</value>
    <description>offline队列20%</description>
  </property>

  <property>
    <name>yarn.scheduler.capacity.root.default.maximum-capacity</name>
    <value>70</value>
    <description>Default队列可使用的资源上限.</description>
  </property>
  <property>
    <name>yarn.scheduler.capacity.root.online.maximum-capacity</name>
    <value>10</value>
    <description>online队列可使用的资源上限.</description>
  </property>
  <property>
    <name>yarn.scheduler.capacity.root.offline.maximum-capacity</name>
    <value>20</value>
    <description>offline队列可使用的资源上限.</description>
  </property>
```

###### ② 指定队列执行任务

```java
/**
 * 指定队列名称
 *
 * @author yaocs2
 * @since 2022-08-20
 */
public class WordCountJobQueue {

    /**
     * 组装Job = Map + Reduce
     *
     * @param args
     */
    public static void main(String[] args) {
        if (args.length < 2) {
            System.exit(100);
        }

        try {
            Configuration conf = new Configuration();

            // 解析命令行中, 通过-D传入的参数, 并添加到conf中
            String[] remainingArgs = new GenericOptionsParser(conf, args).getRemainingArgs();

            // 创建job
            Job job = Job.getInstance(conf);
            job.setJarByClass(WordCountJobQueue.class);// 必须设置, 否则提交到集群后, Job会找不到这个WordCountJob类的

            // 输入、输出
            FileInputFormat.setInputPaths(job, new Path(remainingArgs[0]));
            FileOutputFormat.setOutputPath(job, new Path(remainingArgs[1]));

            // Map
            job.setMapperClass(MyMapper.class);
            job.setMapOutputKeyClass(Text.class);
            job.setMapOutputValueClass(LongWritable.class);

            // Reduce
            job.setReducerClass(MyReducer.class);
            job.setOutputKeyClass(Text.class);
            job.setOutputValueClass(LongWritable.class);

            // 提交Job
            job.waitForCompletion(true);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
     * Map阶段
     *
     * @author yaocs2
     * @since 2022-08-16
     */
    public static class MyMapper extends Mapper<LongWritable, Text, Text, LongWritable> {

        private static final Logger logger = LoggerFactory.getLogger(MyMapper.class);

        /**
         * Map函数：<k1, v1> => <k2, v2>
         *
         * @param k1      每行数据的行首偏移量
         * @param v1      每行的数据内容
         * @param context
         * @throws IOException
         * @throws InterruptedException
         */
        @Override
        protected void map(LongWritable k1, Text v1, Context context) throws IOException, InterruptedException {
            logger.info(String.format("<k1, v1> = <%s, %s>", k1.get(), v1));

            // 切割字符串
            String[] words = v1.toString().split(" ");

            // <k1, v1> => <k2, v2>
            for (String word : words) {
                Text k2 = new Text(word);
                LongWritable v2 = new LongWritable(1L);
                context.write(k2, v2);
            }
        }
    }

    /**
     * Reduce阶段
     *
     * @author yaocs2
     * @since 2022-08-16
     */
    public static class MyReducer extends Reducer<Text, LongWritable, Text, LongWritable> {

        private static final Logger logger = LoggerFactory.getLogger(MyReducer.class);

        /**
         * Reduce函数：<k2, {v2,..}> => <k3, v3>
         *
         * @param k2      单词的值
         * @param v2s     单词出现的所有次数
         * @param context
         * @throws IOException
         * @throws InterruptedException
         */
        @Override
        protected void reduce(Text k2, Iterable<LongWritable> v2s, Context context) throws IOException, InterruptedException {
            // 累加所有次数
            long sum = 0L;
            for (LongWritable v2 : v2s) {
                logger.info(String.format("<k2, v2> = <%s, %s>", k2.toString(), v2));
                sum += v2.get();
            }

            // <k2, {v2,..}> => <k3, v3>
            Text k3 = k2;
            LongWritable v3 = new LongWritable(sum);
            logger.info(String.format("<k1, v1> = <%s, %s>", k3.toString(), v3));
            context.write(k3, v3);
        }
    }
}

// [root@bigdata01 hadoop]# hadoop jar bigdata-1.0-SNAPSHOT-jar-with-dependencies.jar com.jsonyao.mr.WordCountJobQueue -Dmapreduce.job.queuename=offline /test/hello.txt /outqueue

// [root@bigdata01 hadoop]# hadoop jar bigdata-1.0-SNAPSHOT-jar-with-dependencies.jar com.jsonyao.mr.WordCountJobQueue -Dmapreduce.job.queuename=online /test/hello.txt /outqueue2
```

##### 3、Fair Scheduler

多队列，闲时独享，忙时共享，即：

- 只有一个任务执行时，独享资源。
- 多任务执行时，则会从抽取一部分资源给到新任务。

