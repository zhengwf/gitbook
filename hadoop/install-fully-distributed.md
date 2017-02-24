# 环境

查看单机安装部分

# 环境准备

#### 修改主机列表

```
[root@hadoop001 ~]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
172.16.31.206    hadoop001
172.16.31.207    hadoop002
172.16.31.208    hadoop003
```

#### 修改主机名

编辑/etc/sysconfig/network文件，设置主机名

#### 修改句柄

在/etc/security/limits.conf中添加

```
hadoop        soft    nofile    10000
hadoop        hard    nofile    10000
```

在/etc/security/limits.d/90-nproc.conf 中添加

```
hadoop     soft    nproc     unlimited
hadoop     hard    nproc     unlimited
```

#### 关闭防火墙

永久关闭，需要重启命令： chkconfig iptables off  
临时关闭，重启后无效： service iptables stop  
临时关闭selinux命令：setenforce 0  
永久关闭selinux配置： 修改/etc/selinux/config文件，将SELINUX=enforcing改为SELINUX=disabled  
查看防火墙状态命令： service iptables status  
查看selinux状态命令：getenforce 或 sestatus -v

#### 在其他节点上重复以上操作，或者同步配置

#### 创建hadoop用户，并且对hadoop做免密

创建用户命令：useradd -m hadoop -p hadoop  
切换用户到hadoop ： su - hadoop  
生成本地公私钥：ssh-keygen -t rsa -P '' -f ~/.ssh/id\_rsa  
对本地免密:

```
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
chmod -R 700 .ssh/
chmod 600 .ssh/authorized_keys
```

测试本地免密可用：ssh localhost  
配置其他机器对本机免密：把其他机器上的id\_rsa.pub文件复制到hadoop001上，追加到hadoop001上的authorized\_keys文件中。然后分发hadoop001的authorized\_keys文件  
常见问题：~/.ssh文件夹，和~/.ssh/authorized\_keys文件权限，如果还有问题检查  
一下服务器的ssh配置文件/etc/ssh/sshd\_config

```
RSAAuthentication yes        # 启用 RSA 认证，默认为yes
PubkeyAuthentication yes     # 启用公钥认证，默认为yes
```

# 配置hadoop文件

| 服务 | 文件 | 配置项 | 值 | 说明 |
| --- | --- | --- | --- | --- |
| 整体 | core-site.xml | fs.defaultFS | hdfs://hadoop001:9000 | hdfs访问url |
| 整体 | core-site.xml | io.file.buffer.size | 131072 | 设置读写SequenceFiles文件是bufferd大小\(128k\) |
| namenode | hdfs-site.xml | dfs.namenode.name.dir | /opt/beh/data/hadoop/namenode | 设置namenode存储namespace和 transactions logs 的本地存储目录。如果是以逗号分隔的一组路径，就会保存namenode数据多个副本 |
| namenode | hdfs-site.xml | dfs.blocksize | 268435456 | hdfs block块大小（256M） |
| namenode | hdfs-site.xml | dfs.namenode.handler.count | 100 | namenode 线程持有datanode链接的rpc数 |
| datanode | hdfs-site.xml | dfs.datanode.data.dir | /opt/beh/data/hadoop/datanode | datanode存储数据位置，可以是以逗号分隔的多个路径 |

core-site.xml

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
        <name>fs.defaultFS</name>
        <value>hdfs://hadoop001:9000</value>
    </property>
    <property>
            <name>io.file.buffer.size</name>
        <value>131072</value>
    </property>
</configuration>
```

hdfs-site.xml

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

##### 配置slave文件

添加要启动datanode的主机名

# 格式化namenode，启动hdfs

格式化namenode ： hdfs namenode -format  
启动namenode： hadoop-deamon.sh start namenode  
启动datanode： hadoop-deamons.sh start datanode

# 配置yarn



# 启动yarn



