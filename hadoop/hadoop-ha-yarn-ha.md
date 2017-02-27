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
<
?xml version="1.0" encoding="UTF-8"?>
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