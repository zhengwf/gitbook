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

   #### 创建hadoop用户，并做免密 ，修改host列表{#16}

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
 - ssh本地免密，修改host列表
 - 修改配置
 - 格式化hdfs，启动
 - 配置yarn，启动yarn
 #### ssh免密，和修改主机列表
 [参考单机安装](#16)
 #### 修改配置文件
 
