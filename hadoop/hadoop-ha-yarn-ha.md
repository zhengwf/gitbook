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