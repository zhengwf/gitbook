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
在/etc/secuity/limited.conf中添加
```
hadoop		soft	nofile	10000
hadoop		hard	nofile	10000

```
在/etc/security/limits.conf中添加
```
hadoop		soft	nofile	10000
hadoop		hard	nofile	10000
```
#### 关闭防火墙
永久关闭，需要重启命令： chkconfig iptables off
临时关闭，重启后无效： service iptables stop
临时关闭selinux命令：setenforce 0
永久关闭selinux配置： 修改/etc/selinux/config文件，将SELINUX=enforcing改为SELINUX=disabled
查看防火墙状态命令： service iptables status
查看selinux状态命令：

