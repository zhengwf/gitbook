# 部署大致步骤

* 配置环境

  ```
       包含修改句柄，关闭防火墙，关闭selinux，配置host列表，ssh免密
  ```

* 下载解压安装

  ```
       设想： 方法1.从svn上下载。 方法2. 检查本地是否有，如果有，不需要下载。分发安装包后台进行，并且多线程。对大数量节点的集群分发过程会消耗大量时间
  ```

* 修改配置文件

  ```
       根据环境主机分配，修改配置
  ```

* 分发配置文件

* 格式化namenode

  ```
       格式化namenode 需要注意失败时处理办法，和结果获取。重新格式化之前，清理历史数据，特别是datanode的数据目录
  ```

* 启动服务

* 测试服务

  ```
       测试服务能够正常运行，如果可以，给出标准的测试性能报告。
  ```

  # 脚本安排

  单独脚本

* 脚本免密

  * 使用所有用户
  * 不需要重复输入密码
  * 如果有错误能够正常提示

* 修改句柄，防火墙，selinux ，host，创建用户

* hadoop 安装脚本 包含调度其他脚本。

  # 依赖

  需要提前安装java，zookeeper（ha 模式下需要），ssh 服务

  # 使用root 设置环境变量

  调用时需要传递本机主机名，需要在每台机器上执行这个脚本，在整体的调度脚本调用。

脚本内容：

* 关闭防火墙
* 关闭selinux
* 修改句柄
* 创建hadoop用户
* 修改host列表

```bash
#! /bin/bash
# 调用前配置conf/default-config.sh
# 需要在每台机器上执行这个脚本，调用时需要传递当前主机名
HOSTNAME=$1
basepath=$(cd `dirname $0`;cd ../; pwd);
/bin/sh ${basepath}/conf/default-config.sh
OS="undefined"
DIST="undefined"
REV="undefined"
function linuxRelease() {
  OS=`uname -s`
  REV=`uname -r`
  MACH=`uname -m`

  if [ "${OS}" = "SunOS" ] ; then
      OS=Solaris
      ARCH=`uname -p`
      OSSTR="${OS} ${REV}(${ARCH} `uname -v`)"
  elif [ "${OS}" = "AIX" ] ; then
      OSSTR="${OS} `oslevel` (`oslevel -r`)"
  elif [ "${OS}" = "Linux" ] ; then
      KERNEL=`uname -r`
      if [ -f /etc/redhat-release ] ; then
          DIST=`cat /etc/redhat-release | sed s/\ .*// `
          PSUEDONAME=`cat /etc/redhat-release | sed s/.*\(// | sed s/\)//`
          REV=`cat /etc/redhat-release | sed s/.*release\ // | sed s/\ .*//`
      elif [ -f /etc/SUSE-release ] ; then
          DIST=`cat /etc/SUSE-release | tr "\n" ' '| sed s/VERSION.*//`
          REV=`cat /etc/SUSE-release | tr "\n" ' ' | sed s/.*=\ //`
      elif [ -f /etc/mandrake-release ] ; then
          DIST='Mandrake'
          PSUEDONAME=`cat /etc/mandrake-release | sed s/.*\(// | sed s/\)//`
          REV=`cat /etc/mandrake-release | sed s/.*release\ // | sed s/\ .*//`
      elif [ -f /etc/debian_version ] ; then
          DIST="Debian `cat /etc/debian_version`"
          REV=""
      fi

      if [ -f /etc/UnitedLinux-release ] ; then
          DIST="${DIST}[`cat /etc/UnitedLinux-release | tr "\n" ' ' | sed s/VERSION.*//`]"
      fi

      OSSTR="${OS} ${DIST} ${REV}(${PSUEDONAME} ${KERNEL} ${MACH})"
  fi

  echo  "you system is：${OSSTR}"
}

linuxRelease;

if [ $DIST -eq 'CentOS' &&  $(REV:0:1) = 6 ]
then
  #关闭防火墙 -centos6
  /sbin/chkconfig iptables off
  /sbin/service iptables stop
elif [ $DIST -eq 'CentOS' &&  $(REV:0:1) = 7 ]
then
  #关闭防火墙 -centos7
  /usr/bin/systemctl stop firewalld.service
  /usr/bin/systemctl disable firewalld.service
else
  echo "--------------------------------------------"
  echo "this shell not support ${OS} ${DIST} ${REV}"
  echo "please close then fairwall by yourself";
  echo "--------------------------------------------"
fi
# 关闭selinux
echo "close selinux"
/usr/sbin/setenforce 0
/bin/sed -i '/SELINUX/s/enforcing/disabled/' /etc/selinux/config
# 修改句柄
if [ $(grep -c hadoop  /etc/security/limits.conf)  -lt  2 ]
then
  /bin/sed -i '$i hadoop   soft    nofile    131072'  /etc/security/limits.conf
  /bin/sed -i '$i hadoop   hard    nofile    131072'  /etc/security/limits.conf
else
  /bin/sed -i "s/hadoop\s*soft\s*.*/hadoop   soft    nofile    131072/g" /etc/security/limits.conf
  /bin/sed -i "s/hadoop\s*hard\s*.*/hadoop   hard    nofile    131072/g" /etc/security/limits.conf
fi
# centos6 配置文件为：/etc/security/limits.d/90-nproc.conf
# centos7 为：/etc/security/limits.d/20-nproc.conf
if [ $(grep -c hadoop  /etc/security/limits.d/*-nproc.conf)  -lt  2 ]
then
  /bin/sed -i '$i hadoop   soft    nproc    unlimited'  /etc/security/limits.d/*-nproc.conf
  /bin/sed -i '$i hadoop   hard    nproc    unlimited'  /etc/security/limits.d/*-nproc.conf
else
  /bin/sed -i "s/hadoop\s*soft\s*.*/hadoop   soft    nproc    unlimited/g" /etc/security/limits.d/*-nproc.conf
  /bin/sed -i "s/hadoop\s*hard\s*.*/hadoop   hard    nproc    unlimited/g" /etc/security/limits.d/*-nproc.conf
fi

#创建hadoop用户
echo "create user hadoop for install "
id hadoop >> /dev/null
if [ $? -ne  0 ]
then
  useradd -d /home/hadoop hadoop
  echo "$HADOOP_PASSWORD"|passwd --stdin hadoop
fi

# 修改主机列表 注：主机名不能有下划线
while read line
do
  ip=`echo $line |awk -F' ' '{print $1}' `
  if [ $(grep -c "$ip" /etc/hosts)  -le 0 ]
  then
    echo $line >> /etc/hosts
  fi
done < $basepath/conf/hosts

# 修改主机名，先不做需要判断是那台主机，然后修改主机名
#/bin/sed -i "s/HOSTNAME=.*/HOSTNAME=${HOSTNAME}/g" /etc/sysconfig/network
# 不用重启就可生效
#echo ${HOSTNAME} >/proc/sys/kernel/hostname
```

# 3. ssh免密登录

查找了网上ssh免密的做法，大多数使用expect做ssh免密，但是前提需要安装expect ，在离线环境下，安装组件有点复杂。那么可以用户jsch 去做ssh免密吗。下边是尝试的过程。遇见问题： ssh 免密后know\_hosts文件怎么生成，里边存储了什么内容？ 网上都说存储了一个公钥，就没有进一步说明，存储了什么公钥，我可以在机器上找到吗？ 答案：可以，在cat /etc/ssh/ssh\_host\_ecdsa\_key.pub中，ecdsa 是加密方式。

    #！/bin/bash
    #echo y|ssh-keygen -t rsa -f ~/.ssh/id_rsa -N ''
    basepath=$(cd `dirname $0`;cd ../; pwd);
    /bin/sh ${basepath}/conf/default-config.sh
    if [ "$JAVA_HOME" = "" ]
    then
      JAVA_HOME=/opt/beh/core/jdk
    fi
    #先生成hadoop的公私钥，然后复制到 ~/.ssh/authorized_keys文件中，同时设置目录权限，检查/etc/ssh/sshd_config

    #设置/etc/ssh/sshd_config
    if [ -f /etc/ssh/sshd_config ]
    then
      /bin/sed -i '/RSAAuthentication/s/no/yes/' /etc/ssh/sshd_config
      /bin/sed -i '/PubkeyAuthentication/s/no/yes/' /etc/ssh/sshd_config
      #生成公私钥
      echo y|ssh-keygen -t rsa -f ~/.ssh/id_rsa -N ''
      #本地免密
      cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
      chmod -R 700 ~/.ssh
      chmod 600 ~/.ssh/authorized_keys
      # 把公钥写入known_hosts文件中
      HOSTNAME=`hostname`
      HOSTNAME_AND_IP=`grep $HOSTNAME /etc/hosts|awk -F' ' '{print $2","$1}'`
      if [ ! -f ~/.ssh/known_hosts ]
      then
          echo "" >> ~/.ssh/known_hosts
      fi
      cat /etc/ssh/ssh_host_*_key.pub | while read i
      do
        if [  $(grep -c "$i" ~/.ssh/known_hosts)  -le 0  ]
        then
          echo ${HOSTNAME_AND_IP}' '${i} >> ~/.ssh/known_hosts
          echo 'localhost '${i} >> ~/.ssh/known_hosts
        fi
      done
    else
      echo "请确认服务器上已安装ssh服务"
    fi

## 4. 单机安装

使用root用户做前置工作，后期用hadoop用户。需要把安装脚本复制到机器上。

* 复制安装脚本，和安装包到机器的root用户下
  * 这个步骤可以在统一调度那块做，这个脚本可以不提供
* 执行setEnv-by-root.sh
* 执行nopasswd.sh
* 安装jdk 
  * 假设已经安装完成，在依赖关系分析时安装
* 解压安装包，配置参数
* 启动hadoop

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
    #如果放在root 家目录下，是不可能有执行权限的，怎么处理？？？？？
    cp -r -u $basepath /tmp
    su - hadoop -c "/bin/bash /tmp/hadoop/shell/nopassword.sh"
    /usr/bin/tar -zxf $HADOOP_TAR -C $BEH_HOME/core
    HADOOP_PACKAGE=`echo $HADOOP_TAR |awk -F'/' '{print $NF}'|sed "s/.tar.gz//g"`
    mv $BEH_HOME/core/$HADOOP_PACKAGE $BEH_HOME/core/hadoop
    echo "export HADOOP_HOME=${BEH_HOME}/core/hadoop" >> $BEH_HOME/conf/beh_env
    echo 'PATH=$PATH:$HADOOP_HOME/bin' >> $BEH_HOME/conf/beh_env
    chown -R hadoop:hadoop $BEH_HOME

    echo "intsll success"




