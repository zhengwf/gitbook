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
 ```
 [root@hadoop003 ~]# mysql -uroot -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 3
Server version: 5.1.73 Source distribution

Copyright (c) 2000, 2013, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> user mysql;
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'user mysql' at line 1
mysql> use mysql ;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123456' WITH GRANT OPTION;
Query OK, 0 rows affected (0.00 sec)

mysql> flush privileges
    -> ;
Query OK, 0 rows affected (0.00 sec)

mysql> select user ,host from user;
+------+-----------+
| user | host      |
+------+-----------+
| root | %         |
| root | 127.0.0.1 |
|      | hadoop003 |
| root | hadoop003 |
|      | localhost |
| root | localhost |
+------+-----------+
6 rows in set (0.00 sec)

mysql> 
 ```
