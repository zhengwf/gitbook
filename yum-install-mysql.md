# 安装
```
查看安装包：yum list |grep mysql
安装命令： yum install -y mysql-server mysql mysql-devel 
查看版本： rpm -qi mysql-server
```
# 配置
- 启动mysql

    service mysqld start
- 配置开机启动
```    
 [root@hadoop003 ~]# chkconfig mysqld on
 [root@hadoop003 ~]# chkconfig --list |grep mysql
  mysqld         	0:关闭	1:关闭	2:启用	3:启用	4:启用	5:启用	6:关闭
```
- 设置root密码
  
   mysqladmin -u root password '123456'
- 重置密码

