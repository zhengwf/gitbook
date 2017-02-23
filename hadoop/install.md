# 环境

### 硬件

```
3台虚拟机，8G内存，500G硬盘，4核
```

### 软件

```
操作系统： centos6.5  
hadoop ： hadoop-2.6.0-cdh5.7.5  
jdk： 1.7.0_55
```

# 搭建模式

1. 单机模式
2. 伪分布模式
3. 完全分布式
4. ha 模式
5. faderation 模式
6. faderation + viewfs 模式

# 搭建大致步骤

1. 环境准备（免密，主机列表，关闭防火墙，句柄修改等）    
2. 安装jdk  
3. 解压安装包，修改配置，分发  
4. 格式化zk，namenode  
5. 启动服务

   # 单机模式

   ### 单机搭建步骤

   * 创建hadoop用户，对hadoop用户做本地免密
   * 安装java
   * 安装hadoop

   #### 创建hadoop用户，并做免密 ，修改host列表

   ```
   #添加hadoop用户
   [root@hadoop001 ~]# useradd hadoop 
   [root@hadoop001 ~]# passwd hadoop
   更改用户 hadoop 的密码 。
   新的 密码：
   无效的密码： 它基于字典单词
   无效的密码： 过于简单
   重新输入新的 密码：
   passwd： 所有的身份验证令牌已经成功更新。
   [root@hadoop001 ~]# su - hadoop 
   #ssh 本地免密
   [hadoop@hadoop001 ~]$ ssh-keygen -t rsa
   Generating public/private rsa key pair.
   Enter file in which to save the key (/home/hadoop/.ssh/id_rsa):       
   Created directory '/home/hadoop/.ssh'.
   Enter passphrase (empty for no passphrase): 
   Enter same passphrase again: 
   Your identification has been saved in /home/hadoop/.ssh/id_rsa.
   Your public key has been saved in /home/hadoop/.ssh/id_rsa.pub.
   The key fingerprint is:
   cd:13:42:32:94:7c:35:ec:5c:27:ef:ad:01:e9:ee:21 hadoop@hadoop001
   The key's randomart image is:
   +--[ RSA 2048]----+
   |     o+..oo      |
   |      o+. ..o .  |
   |       ..o.. =   |
   |         +o.o .  |
   |        S +. o . |
   |           .. o .|
   |          E..  o |
   |           ....  |
   |           ..    |
   +-----------------+
   [hadoop@hadoop001 ~]$ cat .ssh/id_rsa.pub >> .ssh/authorized_keys
   [hadoop@hadoop001 ~]$ chmod 700 ~/.ssh/
   [hadoop@hadoop001 ~]$ chmod 600 ~/.ssh/authorized_keys 
   ```

   添加主机列表 编辑/etc/host ，添加本机ip 和主机名对应关系

#### 安装java

```
   #解压tar包
   [hadoop@hadoop001 ~]$ tar -zvxf jdk-7u55-linux-x64.tar.gz 
   [hadoop@hadoop001 ~]$ mv jdk1.7.0_55/ jdk
   [hadoop@hadoop001 ~]$ mv jdk /opt/beh/core/
   #配置环境变量，在/etc/profile中添加
   export JAVA_HOME=/opt/beh/core/jdk
   PATH=$PATH:$JAVA_HOME/bin
   #执行重新加载环境变量 
   source /etc/profile
```

#### 安装hadoop

```
    # 配置环境变量
    export HADOOP_HOME=/opt/beh/core/hadoop
    PATH=$PATH:$JAVA_HOME/bin:$HADOOP_HOME/bin
    # 执行加载环境变量命令
    source /etc/profile
    # 执行hadoop命令  
    [hadoop@hadoop001 ~]$ hadoop fs -ls /
    # 能查看到本地文件系统 ，执行wordcount
     [hadoop@hadoop001 ~]$ hadoop jar /opt/beh/core/hadoop/share/hadoop/mapreduce2/hadoop-mapreduce-examples-2.6.0-cdh5.7.5.jar wordcount /home/hadoop/NOTICE.txt /home/hadoop/output   
```

到这里hadoop单机安装完成，单机版默认使用本地的文件系统

# 伪分布式安装

### 安装步骤

* ssh本地免密，修改host列表
* 修改配置
* 格式化hdfs，启动
* 配置yarn，启动yarn

  #### ssh免密，和修改主机列表

  参考单机安装

  #### 修改配置文件

  修改配置文件: $HADOOP\_HOME/etc/hadoop/core-site.xml ,添加配置

  ```
  <configuration>
   <property>
       <name>fs.defaultFS</name>
       <value>hdfs://localhost:9000</value>
   </property>
  </configuration>
  ```

  修改配置文件: $HADOOP\_HOME/etc/hadoop/hdfs-site.xml ,添加配置

  ```
  <configuration>
   <property>
       <name>dfs.replication</name>
       <value>1</value>
   </property>
  </configuration>
  ```

  #### 格式化hdfs，启动hdfs

```
#格式化namenode
[hadoop@hadoop001 hadoop]$ hdfs namenode -format
17/02/23 16:47:57 INFO namenode.NameNode: STARTUP_MSG: 
/************************************************************
STARTUP_MSG: Starting NameNode
STARTUP_MSG:   host = hadoop001/172.16.31.206
STARTUP_MSG:   args = [-format]
STARTUP_MSG:   version = 2.6.0-cdh5.7.5
STARTUP_MSG:   classpath = /opt/beh/core/hadoop/etc/hadoop:/opt/beh/core/hadoop/share/hadoop/common/lib/jaxb-impl-2.2.3-1.jar:/opt/beh/core/hadoop/share/hadoop/common/lib/xz-1.0.jar:/opt/beh/core/hadoop/share/hadoop/common/lib/hadoop-annotations-2.6.0-cdh5.7.5.jar:/opt/beh/core/hadoop/share/hadoop/common/lib/asm-3.2.jar:/opt/beh/core/hadoop/share/hadoop/common/lib/jsp-api-2.1.jar:/opt/beh/core/hadoop/share/hadoop/common/lib/java-xmlbuilder-0.4.jar:/opt/beh/core/hadoop/share/hadoop/common/lib/api-asn1-api-1.0.0-M20.jar:/opt/beh/core/hadoop/share/hadoop/common/lib/api-util-1.0.0-M20.jar:/opt/beh/core/hadoop/share/hadoop/common/lib/commons-cli-1.2.jar:/opt/beh/core/hadoop/share/hadoop/common/lib/jackson-mapper-asl-1.8.8.jar:/opt/beh/core/hadoop/share/hadoop/common/lib/xmlenc-0.52.jar:/opt/beh/core/hadoop/share/hadoop/common/lib/guava-11.0.2.jar:/opt/beh/core/hadoop/share/hadoop/common/lib/commons-lang-2.6.jar:/opt/beh/core/hadoop/share/hadoop/common/lib/commons-net-3.1.jar:/opt/beh/core/hadoop/share/hadoop/common/lib/httpclient-4.2.5.jar:/opt/beh/core/hadoop/share/hadoop/common/lib/jersey-json-1.9.jar:/opt/beh/core/hadoop/share/hadoop/common/lib/commons-el-1.0.jar:/opt/beh/core/hadoop/share/hadoop/common/lib/jaxb-api-2.2.2.jar:/opt/beh/core/hadoop/share/hadoop/common/lib/commons-httpclient-3.1.jar:/opt/beh/core/hadoop/share/hadoop/common/lib/jsr305-3.0.0.jar:/opt/beh/core/hadoop/share/hadoop/common/lib/slf4j-log4j12-1.7.5.jar:/opt/beh/core/hadoop/share/hadoop/common/lib/netty-3.6.2.Final.jar:/opt/beh/core/hadoop/share/hadoop/common/lib/jetty-util-6.1.26.cloudera.4.jar:/opt/beh/core/hadoop/share/hadoop/common/lib/protobuf-java-2.5.0.jar:/opt/beh/core/hadoop/share/hadoop/common/lib/jackson-xc-1.8.8.jar:/opt/beh/core/hadoop/share/hadoop/common/lib/junit-4.11.jar:/opt/beh/core/hadoop/share/hadoop/common/lib/hamcrest-core-1.3.jar:/opt/beh/core/hadoop/share/hadoop/common/lib/commons-io-2.4.jar:/opt/beh/core/hadoop/share/hadoop/common/lib/httpcore-4.2.5.jar:/opt/beh/core/hadoop/share/hadoop/common/lib/jasper-runtime-5.5.23.jar:/opt/beh/core/hadoop/share/hadoop/common/lib/avro-1.7.6-cdh5.7.5.jar:/opt/beh/core/hadoop/share/hadoop/common/lib/jackson-core-asl-1.8.8.jar:/opt/beh/core/hadoop/share/hadoop/common/lib/commons-math3-3.1.1.jar:/opt/beh/core/hadoop/share/hadoop/common/lib/snappy-java-1.0.4.1.jar:/opt/beh/core/hadoop/share/hadoop/common/lib/slf4j-api-1.7.5.jar:/opt/beh/core/hadoop/share/hadoop/common/lib/commons-codec-1.4.jar:/opt/beh/core/hadoop/share/hadoop/common/lib/jersey-core-1.9.jar:/opt/beh/core/hadoop/share/hadoop/common/lib/curator-framework-2.7.1.jar:/opt/beh/core/hadoop/share/hadoop/common/lib/apacheds-i18n-2.0.0-M15.jar:/opt/beh/core/hadoop/share/hadoop/common/lib/jettison-1.1.jar:/opt/beh/core/hadoop/share/hadoop/common/lib/commons-digester-1.8.jar:/opt/beh/core/hadoop/share/hadoop/common/lib/commons-logging-1.1.3.jar:/opt/beh/core/hadoop/share/hadoop/common/lib/htrace-core4-4.0.1-incubating.jar:/opt/beh/core/hadoop/share/hadoop/common/lib/curator-client-2.7.1.jar:/opt/beh/core/hadoop/share/hadoop/common/lib/curator-recipes-2.7.1.jar:/opt/beh/core/hadoop/share/hadoop/common/lib/jasper-compiler-5.5.23.jar:/opt/beh/core/hadoop/share/hadoop/common/lib/jets3t-0.9.0.jar:/opt/beh/core/hadoop/share/hadoop/common/lib/paranamer-2.3.jar:/opt/beh/core/hadoop/share/hadoop/common/lib/commons-configuration-1.6.jar:/opt/beh/core/hadoop/share/hadoop/common/lib/activation-1.1.jar:/opt/beh/core/hadoop/share/hadoop/common/lib/stax-api-1.0-2.jar:/opt/beh/core/hadoop/share/hadoop/common/lib/jackson-jaxrs-1.8.8.jar:/opt/beh/core/hadoop/share/hadoop/common/lib/servlet-api-2.5.jar:/opt/beh/core/hadoop/share/hadoop/common/lib/apacheds-kerberos-codec-2.0.0-M15.jar:/opt/beh/core/hadoop/share/hadoop/common/lib/commons-compress-1.4.1.jar:/opt/beh/core/hadoop/share/hadoop/common/lib/jsch-0.1.42.jar:/opt/beh/core/hadoop/share/hadoop/common/lib/jetty-6.1.26.cloudera.4.jar:/opt/beh/core/hadoop/share/hadoop/common/lib/mockito-all-1.8.5.jar:/opt/beh/core/hadoop/share/hadoop/common/lib/log4j-1.2.17.jar:/opt/beh/core/hadoop/share/hadoop/common/lib/commons-beanutils-1.7.0.jar:/opt/beh/core/hadoop/share/hadoop/common/lib/zookeeper-3.4.5-cdh5.7.5.jar:/opt/beh/core/hadoop/share/hadoop/common/lib/gson-2.2.4.jar:/opt/beh/core/hadoop/share/hadoop/common/lib/jersey-server-1.9.jar:/opt/beh/core/hadoop/share/hadoop/common/lib/commons-beanutils-core-1.8.0.jar:/opt/beh/core/hadoop/share/hadoop/common/lib/hadoop-auth-2.6.0-cdh5.7.5.jar:/opt/beh/core/hadoop/share/hadoop/common/lib/commons-collections-3.2.2.jar:/opt/beh/core/hadoop/share/hadoop/common/lib/logredactor-1.0.3.jar:/opt/beh/core/hadoop/share/hadoop/common/hadoop-common-2.6.0-cdh5.7.5-tests.jar:/opt/beh/core/hadoop/share/hadoop/common/hadoop-common-2.6.0-cdh5.7.5.jar:/opt/beh/core/hadoop/share/hadoop/common/hadoop-nfs-2.6.0-cdh5.7.5.jar:/opt/beh/core/hadoop/share/hadoop/hdfs:/opt/beh/core/hadoop/share/hadoop/hdfs/lib/commons-daemon-1.0.13.jar:/opt/beh/core/hadoop/share/hadoop/hdfs/lib/asm-3.2.jar:/opt/beh/core/hadoop/share/hadoop/hdfs/lib/jsp-api-2.1.jar:/opt/beh/core/hadoop/share/hadoop/hdfs/lib/commons-cli-1.2.jar:/opt/beh/core/hadoop/share/hadoop/hdfs/lib/jackson-mapper-asl-1.8.8.jar:/opt/beh/core/hadoop/share/hadoop/hdfs/lib/xmlenc-0.52.jar:/opt/beh/core/hadoop/share/hadoop/hdfs/lib/guava-11.0.2.jar:/opt/beh/core/hadoop/share/hadoop/hdfs/lib/commons-lang-2.6.jar:/opt/beh/core/hadoop/share/hadoop/hdfs/lib/xml-apis-1.3.04.jar:/opt/beh/core/hadoop/share/hadoop/hdfs/lib/leveldbjni-all-1.8.jar:/opt/beh/core/hadoop/share/hadoop/hdfs/lib/commons-el-1.0.jar:/opt/beh/core/hadoop/share/hadoop/hdfs/lib/jsr305-3.0.0.jar:/opt/beh/core/hadoop/share/hadoop/hdfs/lib/netty-3.6.2.Final.jar:/opt/beh/core/hadoop/share/hadoop/hdfs/lib/jetty-util-6.1.26.cloudera.4.jar:/opt/beh/core/hadoop/share/hadoop/hdfs/lib/protobuf-java-2.5.0.jar:/opt/beh/core/hadoop/share/hadoop/hdfs/lib/commons-io-2.4.jar:/opt/beh/core/hadoop/share/hadoop/hdfs/lib/jasper-runtime-5.5.23.jar:/opt/beh/core/hadoop/share/hadoop/hdfs/lib/jackson-core-asl-1.8.8.jar:/opt/beh/core/hadoop/share/hadoop/hdfs/lib/commons-codec-1.4.jar:/opt/beh/core/hadoop/share/hadoop/hdfs/lib/jersey-core-1.9.jar:/opt/beh/core/hadoop/share/hadoop/hdfs/lib/commons-logging-1.1.3.jar:/opt/beh/core/hadoop/share/hadoop/hdfs/lib/htrace-core4-4.0.1-incubating.jar:/opt/beh/core/hadoop/share/hadoop/hdfs/lib/servlet-api-2.5.jar:/opt/beh/core/hadoop/share/hadoop/hdfs/lib/xercesImpl-2.9.1.jar:/opt/beh/core/hadoop/share/hadoop/hdfs/lib/jetty-6.1.26.cloudera.4.jar:/opt/beh/core/hadoop/share/hadoop/hdfs/lib/log4j-1.2.17.jar:/opt/beh/core/hadoop/share/hadoop/hdfs/lib/jersey-server-1.9.jar:/opt/beh/core/hadoop/share/hadoop/hdfs/hadoop-hdfs-2.6.0-cdh5.7.5.jar:/opt/beh/core/hadoop/share/hadoop/hdfs/hadoop-hdfs-2.6.0-cdh5.7.5-tests.jar:/opt/beh/core/hadoop/share/hadoop/hdfs/hadoop-hdfs-nfs-2.6.0-cdh5.7.5.jar:/opt/beh/core/hadoop/share/hadoop/yarn/lib/jaxb-impl-2.2.3-1.jar:/opt/beh/core/hadoop/share/hadoop/yarn/lib/xz-1.0.jar:/opt/beh/core/hadoop/share/hadoop/yarn/lib/asm-3.2.jar:/opt/beh/core/hadoop/share/hadoop/yarn/lib/commons-cli-1.2.jar:/opt/beh/core/hadoop/share/hadoop/yarn/lib/jackson-mapper-asl-1.8.8.jar:/opt/beh/core/hadoop/share/hadoop/yarn/lib/guava-11.0.2.jar:/opt/beh/core/hadoop/share/hadoop/yarn/lib/commons-lang-2.6.jar:/opt/beh/core/hadoop/share/hadoop/yarn/lib/jersey-guice-1.9.jar:/opt/beh/core/hadoop/share/hadoop/yarn/lib/jersey-client-1.9.jar:/opt/beh/core/hadoop/share/hadoop/yarn/lib/guice-3.0.jar:/opt/beh/core/hadoop/share/hadoop/yarn/lib/leveldbjni-all-1.8.jar:/opt/beh/core/hadoop/share/hadoop/yarn/lib/jersey-json-1.9.jar:/opt/beh/core/hadoop/share/hadoop/yarn/lib/aopalliance-1.0.jar:/opt/beh/core/hadoop/share/hadoop/yarn/lib/jaxb-api-2.2.2.jar:/opt/beh/core/hadoop/share/hadoop/yarn/lib/commons-httpclient-3.1.jar:/opt/beh/core/hadoop/share/hadoop/yarn/lib/jsr305-3.0.0.jar:/opt/beh/core/hadoop/share/hadoop/yarn/lib/guice-servlet-3.0.jar:/opt/beh/core/hadoop/share/hadoop/yarn/lib/jetty-util-6.1.26.cloudera.4.jar:/opt/beh/core/hadoop/share/hadoop/yarn/lib/protobuf-java-2.5.0.jar:/opt/beh/core/hadoop/share/hadoop/yarn/lib/jackson-xc-1.8.8.jar:/opt/beh/core/hadoop/share/hadoop/yarn/lib/javax.inject-1.jar:/opt/beh/core/hadoop/share/hadoop/yarn/lib/commons-io-2.4.jar:/opt/beh/core/hadoop/share/hadoop/yarn/lib/jackson-core-asl-1.8.8.jar:/opt/beh/core/hadoop/share/hadoop/yarn/lib/commons-codec-1.4.jar:/opt/beh/core/hadoop/share/hadoop/yarn/lib/jersey-core-1.9.jar:/opt/beh/core/hadoop/share/hadoop/yarn/lib/jettison-1.1.jar:/opt/beh/core/hadoop/share/hadoop/yarn/lib/commons-logging-1.1.3.jar:/opt/beh/core/hadoop/share/hadoop/yarn/lib/activation-1.1.jar:/opt/beh/core/hadoop/share/hadoop/yarn/lib/stax-api-1.0-2.jar:/opt/beh/core/hadoop/share/hadoop/yarn/lib/jackson-jaxrs-1.8.8.jar:/opt/beh/core/hadoop/share/hadoop/yarn/lib/servlet-api-2.5.jar:/opt/beh/core/hadoop/share/hadoop/yarn/lib/jline-2.11.jar:/opt/beh/core/hadoop/share/hadoop/yarn/lib/commons-compress-1.4.1.jar:/opt/beh/core/hadoop/share/hadoop/yarn/lib/jetty-6.1.26.cloudera.4.jar:/opt/beh/core/hadoop/share/hadoop/yarn/lib/log4j-1.2.17.jar:/opt/beh/core/hadoop/share/hadoop/yarn/lib/zookeeper-3.4.5-cdh5.7.5.jar:/opt/beh/core/hadoop/share/hadoop/yarn/lib/jersey-server-1.9.jar:/opt/beh/core/hadoop/share/hadoop/yarn/lib/commons-collections-3.2.2.jar:/opt/beh/core/hadoop/share/hadoop/yarn/hadoop-yarn-server-nodemanager-2.6.0-cdh5.7.5.jar:/opt/beh/core/hadoop/share/hadoop/yarn/hadoop-yarn-server-web-proxy-2.6.0-cdh5.7.5.jar:/opt/beh/core/hadoop/share/hadoop/yarn/hadoop-yarn-common-2.6.0-cdh5.7.5.jar:/opt/beh/core/hadoop/share/hadoop/yarn/hadoop-yarn-registry-2.6.0-cdh5.7.5.jar:/opt/beh/core/hadoop/share/hadoop/yarn/hadoop-yarn-server-applicationhistoryservice-2.6.0-cdh5.7.5.jar:/opt/beh/core/hadoop/share/hadoop/yarn/hadoop-yarn-applications-distributedshell-2.6.0-cdh5.7.5.jar:/opt/beh/core/hadoop/share/hadoop/yarn/hadoop-yarn-applications-unmanaged-am-launcher-2.6.0-cdh5.7.5.jar:/opt/beh/core/hadoop/share/hadoop/yarn/hadoop-yarn-server-resourcemanager-2.6.0-cdh5.7.5.jar:/opt/beh/core/hadoop/share/hadoop/yarn/hadoop-yarn-server-tests-2.6.0-cdh5.7.5.jar:/opt/beh/core/hadoop/share/hadoop/yarn/hadoop-yarn-server-common-2.6.0-cdh5.7.5.jar:/opt/beh/core/hadoop/share/hadoop/yarn/hadoop-yarn-client-2.6.0-cdh5.7.5.jar:/opt/beh/core/hadoop/share/hadoop/yarn/hadoop-yarn-api-2.6.0-cdh5.7.5.jar:/opt/beh/core/hadoop/share/hadoop/mapreduce/lib/xz-1.0.jar:/opt/beh/core/hadoop/share/hadoop/mapreduce/lib/hadoop-annotations-2.6.0-cdh5.7.5.jar:/opt/beh/core/hadoop/share/hadoop/mapreduce/lib/asm-3.2.jar:/opt/beh/core/hadoop/share/hadoop/mapreduce/lib/jackson-mapper-asl-1.8.8.jar:/opt/beh/core/hadoop/share/hadoop/mapreduce/lib/jersey-guice-1.9.jar:/opt/beh/core/hadoop/share/hadoop/mapreduce/lib/guice-3.0.jar:/opt/beh/core/hadoop/share/hadoop/mapreduce/lib/leveldbjni-all-1.8.jar:/opt/beh/core/hadoop/share/hadoop/mapreduce/lib/aopalliance-1.0.jar:/opt/beh/core/hadoop/share/hadoop/mapreduce/lib/guice-servlet-3.0.jar:/opt/beh/core/hadoop/share/hadoop/mapreduce/lib/netty-3.6.2.Final.jar:/opt/beh/core/hadoop/share/hadoop/mapreduce/lib/protobuf-java-2.5.0.jar:/opt/beh/core/hadoop/share/hadoop/mapreduce/lib/javax.inject-1.jar:/opt/beh/core/hadoop/share/hadoop/mapreduce/lib/junit-4.11.jar:/opt/beh/core/hadoop/share/hadoop/mapreduce/lib/hamcrest-core-1.3.jar:/opt/beh/core/hadoop/share/hadoop/mapreduce/lib/commons-io-2.4.jar:/opt/beh/core/hadoop/share/hadoop/mapreduce/lib/avro-1.7.6-cdh5.7.5.jar:/opt/beh/core/hadoop/share/hadoop/mapreduce/lib/jackson-core-asl-1.8.8.jar:/opt/beh/core/hadoop/share/hadoop/mapreduce/lib/snappy-java-1.0.4.1.jar:/opt/beh/core/hadoop/share/hadoop/mapreduce/lib/jersey-core-1.9.jar:/opt/beh/core/hadoop/share/hadoop/mapreduce/lib/paranamer-2.3.jar:/opt/beh/core/hadoop/share/hadoop/mapreduce/lib/commons-compress-1.4.1.jar:/opt/beh/core/hadoop/share/hadoop/mapreduce/lib/log4j-1.2.17.jar:/opt/beh/core/hadoop/share/hadoop/mapreduce/lib/jersey-server-1.9.jar:/opt/beh/core/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-client-common-2.6.0-cdh5.7.5.jar:/opt/beh/core/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-client-shuffle-2.6.0-cdh5.7.5.jar:/opt/beh/core/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-client-hs-2.6.0-cdh5.7.5.jar:/opt/beh/core/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-client-jobclient-2.6.0-cdh5.7.5.jar:/opt/beh/core/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-client-core-2.6.0-cdh5.7.5.jar:/opt/beh/core/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.6.0-cdh5.7.5.jar:/opt/beh/core/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-client-app-2.6.0-cdh5.7.5.jar:/opt/beh/core/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-client-jobclient-2.6.0-cdh5.7.5-tests.jar:/opt/beh/core/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-client-hs-plugins-2.6.0-cdh5.7.5.jar:/opt/beh/core/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-client-nativetask-2.6.0-cdh5.7.5.jar:/opt/beh/core/hadoop/contrib/capacity-scheduler/*.jar
STARTUP_MSG:   build = http://github.com/cloudera/hadoop -r e8ff3af7cda597c024c980707fbb02fce16dc79a; compiled by 'jenkins' on 2016-11-02T19:01Z
STARTUP_MSG:   java = 1.7.0_55
************************************************************/
17/02/23 16:47:57 INFO namenode.NameNode: registered UNIX signal handlers for [TERM, HUP, INT]
17/02/23 16:47:57 INFO namenode.NameNode: createNameNode [-format]
17/02/23 16:47:57 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Formatting using clusterid: CID-509edbef-226f-4a5b-a86d-d4ce6d4e6a7e
17/02/23 16:47:58 INFO namenode.FSEditLog: Edit logging is async:false
17/02/23 16:47:58 INFO namenode.FSNamesystem: No KeyProvider found.
17/02/23 16:47:58 INFO namenode.FSNamesystem: fsLock is fair:true
17/02/23 16:47:58 INFO blockmanagement.DatanodeManager: dfs.block.invalidate.limit=1000
17/02/23 16:47:58 INFO blockmanagement.DatanodeManager: dfs.namenode.datanode.registration.ip-hostname-check=true
17/02/23 16:47:58 INFO blockmanagement.BlockManager: dfs.namenode.startup.delay.block.deletion.sec is set to 000:00:00:00.000
17/02/23 16:47:58 INFO blockmanagement.BlockManager: The block deletion will start around 2017 二月 23 16:47:58
17/02/23 16:47:58 INFO util.GSet: Computing capacity for map BlocksMap
17/02/23 16:47:58 INFO util.GSet: VM type       = 64-bit
17/02/23 16:47:58 INFO util.GSet: 2.0% max memory 889 MB = 17.8 MB
17/02/23 16:47:58 INFO util.GSet: capacity      = 2^21 = 2097152 entries
17/02/23 16:47:58 INFO blockmanagement.BlockManager: dfs.block.access.token.enable=false
17/02/23 16:47:58 INFO blockmanagement.BlockManager: defaultReplication         = 1
17/02/23 16:47:58 INFO blockmanagement.BlockManager: maxReplication             = 512
17/02/23 16:47:58 INFO blockmanagement.BlockManager: minReplication             = 1
17/02/23 16:47:58 INFO blockmanagement.BlockManager: maxReplicationStreams      = 2
17/02/23 16:47:58 INFO blockmanagement.BlockManager: replicationRecheckInterval = 3000
17/02/23 16:47:58 INFO blockmanagement.BlockManager: encryptDataTransfer        = false
17/02/23 16:47:58 INFO blockmanagement.BlockManager: maxNumBlocksToLog          = 1000
17/02/23 16:47:58 INFO namenode.FSNamesystem: fsOwner             = hadoop (auth:SIMPLE)
17/02/23 16:47:58 INFO namenode.FSNamesystem: supergroup          = supergroup
17/02/23 16:47:58 INFO namenode.FSNamesystem: isPermissionEnabled = true
17/02/23 16:47:58 INFO namenode.FSNamesystem: HA Enabled: false
17/02/23 16:47:58 INFO namenode.FSNamesystem: Append Enabled: true
17/02/23 16:47:58 INFO util.GSet: Computing capacity for map INodeMap
17/02/23 16:47:58 INFO util.GSet: VM type       = 64-bit
17/02/23 16:47:58 INFO util.GSet: 1.0% max memory 889 MB = 8.9 MB
17/02/23 16:47:58 INFO util.GSet: capacity      = 2^20 = 1048576 entries
17/02/23 16:47:58 INFO namenode.FSDirectory: POSIX ACL inheritance enabled? false
17/02/23 16:47:58 INFO namenode.NameNode: Caching file names occuring more than 10 times
17/02/23 16:47:58 INFO util.GSet: Computing capacity for map cachedBlocks
17/02/23 16:47:58 INFO util.GSet: VM type       = 64-bit
17/02/23 16:47:58 INFO util.GSet: 0.25% max memory 889 MB = 2.2 MB
17/02/23 16:47:58 INFO util.GSet: capacity      = 2^18 = 262144 entries
17/02/23 16:47:58 INFO namenode.FSNamesystem: dfs.namenode.safemode.threshold-pct = 0.9990000128746033
17/02/23 16:47:58 INFO namenode.FSNamesystem: dfs.namenode.safemode.min.datanodes = 0
17/02/23 16:47:58 INFO namenode.FSNamesystem: dfs.namenode.safemode.extension     = 30000
17/02/23 16:47:58 INFO metrics.TopMetrics: NNTop conf: dfs.namenode.top.window.num.buckets = 10
17/02/23 16:47:58 INFO metrics.TopMetrics: NNTop conf: dfs.namenode.top.num.users = 10
17/02/23 16:47:58 INFO metrics.TopMetrics: NNTop conf: dfs.namenode.top.windows.minutes = 1,5,25
17/02/23 16:47:58 INFO namenode.FSNamesystem: Retry cache on namenode is enabled
17/02/23 16:47:58 INFO namenode.FSNamesystem: Retry cache will use 0.03 of total heap and retry cache entry expiry time is 600000 millis
17/02/23 16:47:58 INFO util.GSet: Computing capacity for map NameNodeRetryCache
17/02/23 16:47:58 INFO util.GSet: VM type       = 64-bit
17/02/23 16:47:58 INFO util.GSet: 0.029999999329447746% max memory 889 MB = 273.1 KB
17/02/23 16:47:58 INFO util.GSet: capacity      = 2^15 = 32768 entries
17/02/23 16:47:58 INFO namenode.FSNamesystem: ACLs enabled? false
17/02/23 16:47:58 INFO namenode.FSNamesystem: XAttrs enabled? true
17/02/23 16:47:58 INFO namenode.FSNamesystem: Maximum size of an xattr: 16384
17/02/23 16:47:58 INFO namenode.FSImage: Allocated new BlockPoolId: BP-814865150-172.16.31.206-1487839678501
17/02/23 16:47:58 INFO common.Storage: Storage directory /tmp/hadoop-hadoop/dfs/name has been successfully formatted.
17/02/23 16:47:58 INFO namenode.NNStorageRetentionManager: Going to retain 1 images with txid >= 0
17/02/23 16:47:58 INFO util.ExitUtil: Exiting with status 0
17/02/23 16:47:58 INFO namenode.NameNode: SHUTDOWN_MSG: 
/************************************************************
SHUTDOWN_MSG: Shutting down NameNode at hadoop001/172.16.31.206
************************************************************/
# 启动hdfs
[hadoop@hadoop001 hadoop]$ ./sbin/start-dfs.sh 
17/02/23 16:50:48 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Starting namenodes on [localhost]
localhost: Error: JAVA_HOME is not set and could not be found.
localhost: Error: JAVA_HOME is not set and could not be found.
#错误原因： 忘记配置 $HADOOP_HOME/etc/hadoop/hadoop-env.sh文件中JAVA_HOME
# hadoop启动
[hadoop@hadoop001 hadoop]$ start-dfs.sh 
17/02/23 16:57:05 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Starting namenodes on [localhost]
localhost: starting namenode, logging to /opt/beh/core/hadoop/logs/hadoop-hadoop-namenode-hadoop001.out
localhost: starting datanode, logging to /opt/beh/core/hadoop/logs/hadoop-hadoop-datanode-hadoop001.out
Starting secondary namenodes [0.0.0.0]
0.0.0.0: starting secondarynamenode, logging to /opt/beh/core/hadoop/logs/hadoop-hadoop-secondarynamenode-hadoop001.out
17/02/23 16:57:23 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
```

到这里hadoop的hdfs功能基本能用了从上边的日志可以看出，格式化时hadoop的一些配置信息。默认启动的服务有：namenode，datanode，和secondary namenode

```
[hadoop@hadoop001 hadoop]$ ps -ef |grep namenode
hadoop    9895     1  1 16:57 ?        00:00:08 /opt/beh/core/jdk/bin/java -Dproc_namenode -Xmx1000m -Djava.net.preferIPv4Stack=true -Dhadoop.log.dir=/opt/beh/core/hadoop/logs -Dhadoop.log.file=hadoop.log -Dhadoop.home.dir=/opt/beh/core/hadoop -Dhadoop.id.str=hadoop -Dhadoop.root.logger=INFO,console -Djava.library.path=/opt/beh/core/hadoop/lib/native -Dhadoop.policy.file=hadoop-policy.xml -Djava.net.preferIPv4Stack=true -Djava.net.preferIPv4Stack=true -Djava.net.preferIPv4Stack=true -Dhadoop.log.dir=/opt/beh/core/hadoop/logs -Dhadoop.log.file=hadoop-hadoop-namenode-hadoop001.log -Dhadoop.home.dir=/opt/beh/core/hadoop -Dhadoop.id.str=hadoop -Dhadoop.root.logger=INFO,RFA -Djava.library.path=/opt/beh/core/hadoop/lib/native -Dhadoop.policy.file=hadoop-policy.xml -Djava.net.preferIPv4Stack=true -Dhadoop.security.logger=INFO,RFAS -Dhdfs.audit.logger=INFO,NullAppender -Dhadoop.security.logger=INFO,RFAS -Dhdfs.audit.logger=INFO,NullAppender -Dhadoop.security.logger=INFO,RFAS -Dhdfs.audit.logger=INFO,NullAppender -Dhadoop.security.logger=INFO,RFAS org.apache.hadoop.hdfs.server.namenode.NameNode
```

通过ps 查看启动namenode时使用到的一些命令行参数，通过日志可以看到默认的web访问地址，但是如果远程连接好像访问不到

```
2017-02-23 16:51:44,297 INFO org.apache.hadoop.hdfs.DFSUtil: Starting Web-server for hdfs at: http://0.0.0.0:50070
2017-02-23 16:51:44,350 INFO org.mortbay.log: Logging to org.slf4j.impl.Log4jLoggerAdapter(org.mortbay.log) via org.mortbay.log.Slf4jLog
2017-02-23 16:51:44,358 INFO org.apache.hadoop.security.authentication.server.AuthenticationFilter: Unable to initialize FileSignerSecretProvider, falling back to use random secrets.
2017-02-23 16:51:44,366 INFO org.apache.hadoop.http.HttpRequestLog: Http request log for http.requests.namenode is not defined
2017-02-23 16:51:44,379 INFO org.apache.hadoop.http.HttpServer2: Added global filter 'safety' (class=org.apache.hadoop.http.HttpServer2$QuotingInputFilter)
2017-02-23 16:51:44,385 INFO org.apache.hadoop.http.HttpServer2: Added filter static_user_filter (class=org.apache.hadoop.http.lib.StaticUserWebFilter$StaticUserFilter) to context hdfs
2017-02-23 16:51:44,385 INFO org.apache.hadoop.http.HttpServer2: Added filter static_user_filter (class=org.apache.hadoop.http.lib.StaticUserWebFilter$StaticUserFilter) to context logs
2017-02-23 16:51:44,385 INFO org.apache.hadoop.http.HttpServer2: Added filter static_user_filter (class=org.apache.hadoop.http.lib.StaticUserWebFilter$StaticUserFilter) to context static
2017-02-23 16:51:44,417 INFO org.apache.hadoop.http.HttpServer2: Added filter 'org.apache.hadoop.hdfs.web.AuthFilter' (class=org.apache.hadoop.hdfs.web.AuthFilter)
2017-02-23 16:51:44,419 INFO org.apache.hadoop.http.HttpServer2: addJerseyResourcePackage: packageName=org.apache.hadoop.hdfs.server.namenode.web.resources;org.apache.hadoop.hdfs.web.resources, pathSpec=/webhdfs/v1/*

```



