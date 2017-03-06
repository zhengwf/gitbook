# 1. 单机安装

使用root用户做前置工作，后期用hadoop用户。需要把安装脚本复制到机器上。

* 复制安装脚本，和安装包到机器的root用户下
  * 这个步骤可以在统一调度那块做，这个脚本可以不提供
* 执行setEnv-by-root.sh
* 执行nopasswd.sh
* 安装jdk 
  * 假设已经安装完成，在依赖关系分析时安装
* 解压安装包，配置参数
* 启动hadoop

```
#!/bin/bash
#------------------
# 需要参数： hadoop安装包
# 安装单机版hadoop
#
# 需要在root用户下执行
#
# 安装步骤：
# 1.设置环境变量 --- 不需要
# 2. hadoop 用户本地免密
# 3. 在hadoop用户下解压安装包，
HADOOP_TAR=$1
basepath=$(cd `dirname $0`;cd ../; pwd);
. $basepath/conf/default-config.sh
#修改环境变量
#/bin/bash $basepath/shell/setEnv-by-root.sh
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
chown -R hadoop:hadoop $BEH_HOME

echo "intsll success"

```



