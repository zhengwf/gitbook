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
- 重置root密码

  ```
   # 停止服务： service mysqld stop
   # 跳过授权表启动： mysqld_safe --skip-grant-tables &
   # 登录mysql ： mysql -uroot 
   # 执行mysql命令：
   mysql> use mysql;
   Reading table information for completion of table and column names
   You can turn off this feature to get a quicker startup with -A
   
   Database changed
   mysql> update user set password=PASSWORD('123456') where user="root"
       -> ;
   Query OK, 3 rows affected (0.00 sec)
   Rows matched: 3  Changed: 3  Warnings: 0
   
   mysql> flush privileges;
   Query OK, 0 rows affected (0.00 sec)
   
   mysql> quit
   # kill 掉mysql 进程，重新正常启动
  ```
- 设置远程登录权限
 
  在mysql数据库中执行：GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '你的密码' WITH GRANT OPTION;
