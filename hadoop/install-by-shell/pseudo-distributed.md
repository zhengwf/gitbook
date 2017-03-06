# 伪分布式安装

### 依赖

```
需要提前安装： jdk
```

### 安装步骤

```
1.在root用户下设置环境：关闭防火墙，句柄修改，selinux等
2.解压安装包
3.复盖配置文件
4.格式化namenode和启动
```

### 脚本

    #!/bin/bash
    #------------------
    # 需要参数： hadoop安装包
    # 安装单机版伪分布hadoop
    #
    # 需要在root用户下执行
    #
    # 安装步骤：
    #1.在root用户下设置环境：关闭防火墙，句柄修改，selinux等
    #2.解压安装包
    #3.复盖配置文件
    #4.格式化namenode和启动
    HADOOP_TAR=$1
    basepath=$(cd `dirname $0`;cd ../; pwd);
    . $basepath/conf/default-config.sh
    #修改环境变量
    /bin/bash $basepath/shell/setEnv-by-root.sh
    if [ "$BEH_HOME" = '' ]
    then
      BEH_HOME=/opt/beh
    fi
    echo "create user hadoop "
    id hadoop >> /dev/null
    if [ $? -ne  0 ]
    then
      useradd -d /home/hadoop hadoop
      echo "$HADOOP_PASSWORD"|passwd --stdin hadoop
    fi
    #判断java 是否可用
    su - hadoop -c "java -version"
    if [ $? -ne - ]
    then
    echo "java is not enable for user hadoop";
    exit -1;
    fi
    #如果放在root 家目录下，是不可能有执行权限的，怎么处理？？？？？
    # 执行hadoop 用户本地 免密
    cp -r -u $basepath /tmp
    su - hadoop -c "/bin/bash /tmp/hadoop/shell/nopassword.sh"
    # 解压安装包
    /usr/bin/tar -zxf $HADOOP_TAR -C $BEH_HOME/core
    HADOOP_PACKAGE=`echo $HADOOP_TAR |awk -F'/' '{print $NF}'|sed "s/.tar.gz//g"`
    mv $BEH_HOME/core/$HADOOP_PACKAGE $BEH_HOME/core/hadoop
    # 配置环境变量
    if [ $(grep -c HADOOP_HOME=${BEH_HOME}/core/hadoop $BEH_HOME/conf/beh_env) -lt 1 ]
    then
      echo "export HADOOP_HOME=${BEH_HOME}/core/hadoop" >> $BEH_HOME/conf/beh_env
      echo 'PATH=$PATH:$HADOOP_HOME/bin' >> $BEH_HOME/conf/beh_env
    fi
    # 覆盖配置文件
    cp $basepath/conf/pseudo-conf/* $BEH_HOME/core/hadoop/etc/hadoop/
    sed -i 's|export\s*JAVA_HOME=${JAVA_HOME}|export JAVA_HOME='"${BEH_HOME}"'/core/jdk|g' $BEH_HOME/core/hadoop/etc/hadoop/hadoop-env.sh
    chown -R hadoop:hadoop $BEH_HOME
    # 格式化namenode，启动服务
    su - hadoop -c "echo Y|$BEH_HOME/core/hadoop/bin/hdfs namenode -format; 
    if [ $? = 0 ] 
    then 
    $BEH_HOME/core/hadoop/sbin/start-dfs.sh 
    $BEH_HOME/core/hadoop/sbin/start-yarn.sh
    else
       echo 'format namenode fail ,please chack the reason'
    fi; "
    echo "intsll success"




