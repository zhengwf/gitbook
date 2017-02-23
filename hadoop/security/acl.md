### 概述
  hdfs基于可移植模型实现了权限管理模块，每个文件和目录都有相关联的归属用户和组。

权限 | 文件 | 目录
---|---|---
r | 需要上层目录通过r-x权限，文件本身有r权限才能读取| 查看目录内，需要r-x权限才能查看目录结构|
w | 只需要w权限就可覆盖，删除文件| 只有w权限无法在目录中上传文件，创建目录，删除目录，需要加x权限|
x | -- | 操作子目录,只有x权限无法查看子目录
在hadoop 0.22，提供了两种身份认证  
 ##### 1.simple  
     基于主机操作系统的用户认证，在Unix-like系统中，用户名等于`whoami`  
 ##### 2.kerberos
     认证基于kerberos的资格，具体后边介绍
### 用户组
 用户组获取主要根据hadoop.security.group.mapping property配置，主要的实现有：
类 | 作用 | 默认
---|---|---
org.apache.hadoop.security.JniBasedUnixGroupsMappingWithFallback | 当本地JNI可用时，获取用户组节列表 | 是
 org.apache.hadoop.security.ShellBasedUnixGroupsMapping | 根据bash -c group 或net group 获取用户组 | 否
 org.apache.hadoop.security.LdapGroupsMapping | 从LDAP中获取用户组 | 否
### 实现
每次文件或目录操作都传递完整的路径名给name node，每一个操作都会对此路径做权限检查。客户框架会隐式地将用户身份和与name node的连接关联起来，从而减少改变现有客户端API的需求。经常会有这种情况，当对一个文件的某一操作成功后，之后同样的操作却会失败，这是因为文件或路径上的某些目录已经不复存在了。比如，客户端首先开始读一个文件，它向name node发出一个请求以获取文件第一个数据块的位置。但接下去的获取其他数据块的第二个请求可能会失败。另一方面，删除一个文件并不会撤销客户端已经获得的对文件数据块的访问权限。而权限管理能使得客户端对一个文件的访问许可在两次请求之间被收回。重复一下，权限的改变并不会撤销当前客户端对文件数据块的访问许可。

map-reduce框架通过传递字符串来指派用户身份，没有做其他特别的安全方面的考虑。文件或目录的所有者和组属性是以字符串的形式保存，而不是像传统的Unix方式转换为用户和组的数字ID。

这个发行版本的权限管理特性并不需要改变data node的任何行为。Data node上的数据块上并没有任何Hadoop所有者或权限等关联属性。
### 超级用户
超级用户即运行name node进程的用户。宽泛的讲，如果你启动了name node，你就是超级用户。超级用户干任何事情，因为超级用户能够通过所有的权限检查。没有永久记号保留谁过去是超级用户；当name node开始运行时，进程自动判断谁现在是超级用户。HDFS的超级用户不一定非得是name node主机上的超级用户，也不需要所有的集群的超级用户都是一个。同样的，在个人工作站上运行HDFS的实验者，不需任何配置就已方便的成为了他的部署实例的超级用户。

另外，管理员可以用配置参数指定一组特定的用户，如果做了设定，这个组的成员也会是超级用户。

### Web服务器
Web服务器的身份是一个可配置参数。Name node并没有真实用户的概念，但是Web服务器表现地就像它具有管理员选定的用户的身份（用户名和组）一样。除非这个选定的身份是超级用户，否则会有名字空间中的一部分对Web服务器来说不可见。
### acl权限
hdfs支持acl权限控制，主要针对非拥有者用户或用户组进行设置，开启设置：dfs.namenode.acls.enabled = true ，数据主要存储在namenode的namespace中。
    查看权限命令：Hadoop fs -getfacl /testacl ,修改权限的命令：hadoop fs -setfacl -m user:zwf:rwx，
    
```
[hadoop@hadoop014 ~]$ hadoop fs -getfacl /testacl
# file: /testacl
# owner: hadoop
# group: hadoop
user::rwx
user:bb:rw-
user:zwf:rwx
group::rwx
mask::rwx
other::r-x
```
其中 mask 是acl所有权限的并集，如果自定了mask的权限，超过mask权限的部分将无效，但是如果再有改动mask回自动修改回并集的结果，==不建议修改mask==   
只能对目录设置default权限，这个设置会自动在创建子目录时集成，不会影响已经存在的目录。
```
[hadoop@hadoop014 ~]$ hadoop fs -setfacl -m  default:user:zwf:r-- /testacl
[hadoop@hadoop014 ~]$ hadoop fs -getfacl /testacl
# file: /testacl
# owner: hadoop
# group: hadoop
user::rwx
user:bb:rw-
user:zwf:rwx
group::rwx
mask::rwx
other::r-x
default:user::rwx
default:user:zwf:r--
default:group::rwx
default:mask::rwx
default:other::r-x

[hadoop@hadoop014 ~]$ hadoop fs -mkdir /testacl/t1
[hadoop@hadoop014 ~]$ hadoop fs -getfacl /testacl/t1
# file: /testacl/t1
# owner: hadoop
# group: hadoop
user::rwx
user:zwf:r--
group::rwx
mask::rwx
other::r-x
default:user::rwx
default:user:zwf:r--
default:group::rwx
default:mask::rwx
default:other::r-x

[hadoop@hadoop014 ~]$ hadoop fs -ls /testacl/
Found 6 items
-rw-rw-r--   3 zwf         hadoop          2 2017-01-09 14:38 /testacl/aa.txt
drwxrwxr-x+  - hadoop      hadoop          0 2017-01-09 14:58 /testacl/t1
drwxrwxr-x   - tianbaochao hadoop          0 2017-01-05 19:31 /testacl/test
drwxrwxr-x   - tianbaochao hadoop          0 2017-01-05 19:31 /testacl/test2
drwxrwxr-x   - tianbaochao hadoop          0 2017-01-05 19:32 /testacl/test3
drwxrwxr-x   - tianbaochao hadoop          0 2017-01-05 19:37 /testacl/test4
[hadoop@hadoop014 ~]$ hadoop fs -getfacl /testacl/test
# file: /testacl/test
# owner: tianbaochao
# group: hadoop
user::rwx
group::rwx
other::r-x
```
acl权限，比较流程：  
1. 比较是否时归属用户  
2. 比较是acl 用户名列表 ，用mask过滤后，测试权限
3. 比较归属组 ，mask过滤后，测试权限
4. 比较acl组，。。。
5. 如果用户组通过，但用户组权限不通过，则拒绝访问
6. 测试其他权限  
注：尽量减少acl权限控制，会增加namenode内存开销  
### 对应api接口
```
public void modifyAclEntries(Path path, List<AclEntry> aclSpec) throws IOException;
public void removeAclEntries(Path path, List<AclEntry> aclSpec) throws IOException;
public void public void removeDefaultAcl(Path path) throws IOException;
public void removeAcl(Path path) throws IOException;
public void setAcl(Path path, List<AclEntry> aclSpec) throws IOException;
public AclStatus getAclStatus(Path path) throws IOException;
```
 ### 配置
-  dfs.permissions.enabled = true  
If yes use the permissions system as described here. If no, permission checking is turned off, but all other behavior is unchanged. Switching from one parameter value to the other does not change the mode, owner or group of files or directories. Regardless of whether permissions are on or off, chmod, chgrp, chown and setfacl always check permissions. These functions are only useful in the permissions context, and so there is no backwards compatibility issue. Furthermore, this allows administrators to reliably set owners and permissions in advance of turning on regular permissions checking.

- dfs.web.ugi = webuser,webgroup  
The user name to be used by the web server. Setting this to the name of the super-user allows any web client to see everything. Changing this to an otherwise unused identity allows web clients to see only those things visible using "other" permissions. Additional groups may be added to the comma-separated list.

- dfs.permissions.superusergroup = supergroup  
The name of the group of super-users.

- fs.permissions.umask-mode = 0022  
The umask used when creating files and directories. For configuration files, the decimal value 18 may be used.

- dfs.cluster.administrators = ACL-for-admins  
The administrators for the cluster specified as an ACL. This controls who can access the default servlets, etc. in the HDFS.

- dfs.namenode.acls.enabled = true  
Set to true to enable support for HDFS ACLs (Access Control Lists). By default, ACLs are disabled. When ACLs are disabled, the NameNode rejects all attempts to set an ACL.
