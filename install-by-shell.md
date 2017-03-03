# 脚本安装jdk

* 解压安装包
* 配置环境变量

  ```bash
  #!/bin/bash
  #调度方式： sh jdk-install.sh jdk-7u55-linux-x64.tar.gz
  # 其他参数修改default-conf.sh
  basepath=$(cd `dirname $0`; pwd);
  source ${basepath}/default-config.sh
  JDK_PACKAGE=$1
  # 解压后jdk 的名字变了/(ㄒoㄒ)/~~
  PACKAGE_PATH=$(cd `dirname $JDK_PACKAGE`;pwd)
  /bin/tar -zxf $JDK_PACKAGE -C $PACKAGE_PATH
  PACKAGE=`ls -l ${PACKAGE_PATH} |grep jdk |grep -v tar.gz |grep ^d |awk '{print $NF}'`
  # 清理以前解压出来的jdk包
  rm -rf  ${PACKAGE_PATH}/jdk
  mv ${PACKAGE_PATH}/$PACKAGE ${PACKAGE_PATH}/jdk
  # 创建jdk安装目录
  if [ $BEH_HOME = '' ]
  then
    BEH_HOME=/opt/beh
  fi
  if [ ! -d $BEH_HOME/core ]
  then
    echo "create $BEH_HOME/core";
    mkdir -p $BEH_HOME/core
  else
    echo "$BEH_HOME/core has exist"
  fi
  if [ ! -d $BEH_HOME/conf ]
  then
    echo "create $BEH_HOME/conf";
    mkdir $BEH_HOME/conf
  else
    echo "$BEH_HOME/core has exist"
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
  mv -f ${PACKAGE_PATH}/jdk $BEH_HOME/core/
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



