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

```bash
#! /bin/bash

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
sed -i '/SELINUX/s/enforcing/disabled/' /etc/selinux/config
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
  echo "hadoop"|passwd --stdin hadoop
fi
```

# 3. ssh免密登录





