# 脚本安装jdk

* 解压安装包
* 配置环境变量

  ```bash
  #!/bin/bash
  #调度方式： sh jdk-install.sh jdk-7u55-linux-x64.tar.gz
  # 其他参数修改default-conf.sh
  basepath=$(cd `dirname $0`; pwd);
  #/bin/sh ${basepath}/default-config.sh
  BEH_HOME=/opt/beh
  JDK_PACKAGE=$1
  # 解压后jdk 的名字变了/(ㄒoㄒ)/~~
  /bin/tar -zvxf ${basepath}/$JDK_PACKAGE
  PACKAGE=`ls -l ${basepath} |grep jdk |grep -v tar.gz |grep ^d |awk '{print $NF}'`
  # 清理以前解压出来的jdk包
  rm -rf  ${basepath}/jdk
  mv ${basepath}/$PACKAGE ${basepath}/jdk
  # 创建jdk安装目录
  if [ ! -d $BEH_HOME/core ]
  then
    mkdir -p $BEH_HOME/core
  fi
  if [ ! -d $BEH_HOME/conf ]
  then
    mkdir $BEH_HOME/conf
  fi
  if [ ! -f $BEH_HOME/conf/beh_env ]
  then
    echo "" > $BEH_HOME/conf/beh_env
  fi
  # 如果有以前遗留的jdk包，删除
  if [ -d $BEH_HOME/core/jdk  ]
  then
    rm -rf $BEH_HOME/core/jdk
  fi
  mv -f ${basepath}/jdk $BEH_HOME/core/
  chown -R hadoop:hadoop $BEH_HOME/core/
  if [ $(grep -c $BEH_HOME/conf/beh_env /etc/profile) -lt 1 ]
  then
    echo "source $BEH_HOME/conf/beh_env" >> /etc/profile
  fi
  source /etc/profile
  if [ $(grep -c JAVA_HOME=$BEH_HOME/core/jdk $BEH_HOME/conf/beh_env) -lt 1 ]
  then
    echo "export JAVA_HOME=$BEH_HOME/core/jdk" >> $BEH_HOME/conf/beh_env
    echo 'PATH=$PATH:$JAVA_HOME/bin' >> $BEH_HOME/conf/beh_env
  fi

  source $BEH_HOME/conf/beh_env

  ```



