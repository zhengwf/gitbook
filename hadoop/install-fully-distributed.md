#环境
查看单机安装部分
#环境准备
#### 修改主机列表
```
[root@hadoop001 ~]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
172.16.31.206	hadoop001
172.16.31.207	hadoop002
172.16.31.208	hadoop003
```
#### 修改主机名
编辑/etc/sysconfig/network文件，设置主机名
#### 修改句柄
在/etc/security/limits.conf中添加
```
hadoop		soft	nofile	10000
hadoop		hard	nofile	10000

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
生成本地公私钥：ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
对本地免密:
```
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
chmod -R 700 .ssh/
chmod 600 .ssh/authorized_keys
```
测试本地免密可用：ssh localhost
配置其他机器对本机免密：把其他机器上的id_rsa.pub文件复制到hadoop001上，追加到hadoop001上的authorized_keys文件中。然后分发hadoop001的authorized_keys文件


