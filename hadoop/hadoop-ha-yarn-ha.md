# 前提准备
查看上一节文档，完全分布式安装部分
# 大致步骤
- 修改hdfs配置文件
- 配置zookeeper
- 启动hadoop
- 配置yarn ha
- 启动yarn

# 修改hdfs 配置

#### 1.修改core-site.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<!--
Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License. See accompanying LICENSE file.
-->

<!-- Put site-specific property overrides in this file. -->

<configuration>
  <property>
      <name>dfs.defaultFS</name>
      <value>hdfs://beh</value>
  </property>
  <property>
      <name>io.file.buffer.size</name>
      <value>131072</value>
  </property>
  <property>
      <name>hadoop.tmp.dir</name>
      <value>/opt/beh/data/hadoop/tmp</value>
  </property>
  <property>
    <name>ha.zookeeper.quorum</name>
    <value>hadoop001:2181,hadoop002:2181,hadoop003:2181</value>
  </property>
</configuration>

```
hadoop.tmp.dir ： hadoop文件系统依赖的基本配置，很多配置路径都依赖它，它的默认位置在/tmp/{$user}下面。linux 重启后/tmp目录会被情空，需要修改位置
ha.zookeeper.quorum： 配置zookeeper链接地址，主要用在hadoop ha切换时选举active的namenode。
#### 2.修改hdfs-site.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<!--
Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License. See accompanying LICENSE file.
-->

<!-- Put site-specific property overrides in this file. -->

<configuration>
  <property>
    <name>dfs.nameservices</name>
    <value>beh</value>
  </property>
  <property>
    <name>dfs.ha.namenodes.beh</name>
    <value>nn1,nn2</value>
  </property>
  <property>
    <name>dfs.namenode.rpc-address.beh.nn1</name>
    <value>hadoop001:8020</value>
  </property>
  <property>
    <name>dfs.namenode.rpc-address.beh.nn2</name>
    <value>hadoop002:8020</value>
  </property>
  <property>
    <name>dfs.namenode.http-address.beh.nn1</name>
    <value>hadoop001:50070</value>
  </property>
  <property>
    <name>dfs.namenode.http-address.beh.nn2</name>
    <value>hadoop002:50070</value>
  </property>
  <property>
    <name>dfs.namenode.shared.edits.dir</name>
    <value>qjournal://hadoop001:8485;hadoop002:8485;hadoop003:8485/beh</value>
  </property>
  <property>
    <name>dfs.client.failover.proxy.provider.beh</name>
    <value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
  </property>
  <property>
    <name>dfs.ha.fencing.methods</name>
    <value>sshfence</value>
  </property>
  <property>
    <name>dfs.ha.fencing.ssh.private-key-files</name>
    <value>/home/hadoop/.ssh/id_rsa</value>
  </property>
  <property>
   <name>dfs.ha.automatic-failover.enabled</name>
   <value>true</value>
 </property>
  <property>
    <name>dfs.replication</name>
    <value>1</value>
  </property>
  <property>
      <name>dfs.namenode.name.dir</name>
      <value>/opt/beh/data/hadoop/namenode</value>
  </property>
  <property>
    <name>dfs.blocksize</name>
    <value>268435456</value>
  </property>
  <property>
      <name>dfs.namenode.handler.count</name>
      <value>100</value>
  </property>
  <property>
    <name>dfs.datanode.data.dir</name>
    <value>/opt/beh/data/hadoop/datanode</value>
  </property>
</configuration>

```
|配置项|值|说明|
|--|--|--|
|dfs.nameservices| beh |nameservices 的逻辑名，也是namespace的名称|
|dfs.ha.namenodes.[nameservice ID] | nn1,nn2 |nameservice中namenode的标识符|
|dfs.namenode.rpc-address.[nameservice ID].[name node ID]|hadoop001:8020|namenode监听的rpc地址|
|dfs.namenode.shared.edits.dir |qjournal://hadoop001:8485;hadoop002:8485;hadoop003:8485/beh|namenode共享数据的存储位置，这里配置的是journalnode 中共享|
|dfs.client.failover.proxy.provider.[nameservice ID]|org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider|配置hdfs客户端判断哪个namenode active|
|dfs.ha.fencing.methods |sshfence|配置隔离失败的namenode的方法：sshfence-ssh方式到失败的namenode，kill掉进程。shell-执行脚本来隔离namenode|
|dfs.ha.automatic-failover.enabled|true | 设置自动切换namenode
#### 3.配置zookeeper
##### 1.下载安装包，并解压，放在/opt/beh/core/zookeeper
##### 2.配置zookeeper- zoo.cfg
```
tickTime=2000  
dataDir=/opt/beh/data/zookeeper  
clientPort=2181  
initLimit=5  
syncLimit=2  
server.1=hadoop001:2888:3888  
server.2=hadoop002:2888:3888  
server.3=hadoop003:2888:3888 
```
##### 3.分发安装包
```
scp -r /opt/beh/core/zookeeper hadoop002:/opt/beh/core
scp -r /opt/beh/core/zookeeper hadoop003:/opt/beh/core
```
##### 4.修改/opt/beh/data/zookeeper/myid 
```
ssh hadoop001 "mkdir /opt/beh/data/zookeeper ; echo '1' >/opt/beh/data/zookeeper/myid ";
ssh hadoop002 "mkdir /opt/beh/data/zookeeper ; echo '2' >/opt/beh/data/zookeeper/myid  ";
ssh hadoop003 "mkdir /opt/beh/data/zookeeper ; echo '3' >/opt/beh/data/zookeeper/myid  ";

```
##### 5.配置环境变量（非必要，为了使用方便）
##### 6.启动zookeeper
/opt/beh/data/zookeeper/zkServer.sh start

# 启动hdfs
##### 1.格式化zk
```
[hadoop@hadoop001 hadoop]$ hdfs zkfc -formatZK
```
显示日志：ha.ActiveStandbyElector: Successfully created /hadoop-ha/beh in ZK.
表示格式化成功，主要在zookeeper上创建目录
##### 2.启动journalnode
在hadoop001，hadoop002，hadoop003上执行
```
hadoop-daemon.sh start journalnode
```
##### 3.格式化namenode，启动，启动zkfc
在hadoop001上
```
hdfs namenode -format
hadoop-daemon.sh start namenode
hadoop-daemon.sh start zkfc
```
在hadoop002上
```
hdfs namenode -bootstrapStandby
hadoop-daemon.sh start namenode
hadoop-daemon.sh start zkfc
```
##### 4.启动datanode
```
hadoop-daemons.sh start datanode
```
# yarn 配置
```
<?xml version="1.0" encoding="UTF-8"?>

<!-- Licensed under the Apache License, Version 2.0 (the "License"); you
	may not use this file except in compliance with the License. You may obtain
	a copy of the License at http://www.apache.org/licenses/LICENSE-2.0 Unless
	required by applicable law or agreed to in writing, software distributed
	under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES
	OR CONDITIONS OF ANY KIND, either express or implied. See the License for
	the specific language governing permissions and limitations under the License.
	See accompanying LICENSE file. -->
<configuration>
  <property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
  </property>
  <property>
   <name>yarn.resourcemanager.ha.enabled</name>
   <value>true</value>
 </property>
 <property>
   <name>yarn.resourcemanager.cluster-id</name>
   <value>beh</value>
 </property>
 <property>
   <name>yarn.resourcemanager.ha.rm-ids</name>
   <value>rm1,rm2</value>
 </property>
 <property>
   <name>yarn.resourcemanager.hostname.rm1</name>
   <value>hadoop001</value>
 </property>
 <property>
   <name>yarn.resourcemanager.hostname.rm2</name>
   <value>hadoop002</value>
 </property>
  <property>
    <name>yarn.resourcemanager.webapp.address.rm1</name>
    <value>hadoop001:23188</value>
  </property>
  <property>
    <name>yarn.resourcemanager.webapp.address.rm2</name>
    <value>hadoop002:23188</value>
  </property>
  <property>
    <name>yarn.resourcemanager.zk-address</name>
    <value>hadoop001:2181,hadoop002:2181,hadoop003:2181</value>
  </property>
  <property>
    <name>yarn.scheduler.minimum-allocation-mb</name>
    <value>1024</value>
  </property>
  <property>
    <name>yarn.scheduler.maximum-allocation-mb</name>
    <value>2048</value>
  </property>
  <property>
	   <name>yarn.resourcemanager.scheduler.class</name>
	    <value>org.apache.hadoop.yarn.server.resourcemanager.scheduler.capacity.CapacityScheduler</value>
  </property>
  <property>
    <name>yarn.nodemanager.resource.memory-mb</name>
    <value>4096</value>
  </property>
  <property>
    <name>yarn.nodemanager.vmem-pmem-ratio</name>
    <value>2.1</value>
  </property>
  <property>
    <name>yarn.nodemanager.local-dirs</name>
    <value>/opt/beh/data/nodemanager</value>
  </property>
  <property>
    <name>yarn.nodemanager.log-dirs</name>
    <value>/opt/beh/log/yarn</value>
  </property>
  <property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
  </property>
</configuration>

```
# 分发配置文件,启动yarn
```

```
