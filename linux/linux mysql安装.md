1.在线安装

# [MySQL安装之ym安装](https://www.cnblogs.com/brianzhu/p/8575243.html)

 在CentOS7中默认安装有MariaDB，这个是MySQL的分支，但为了需要，还是要在系统中安装MySQL，而且安装完成之后可以直接覆盖掉MariaDB。

### 1. 下载并安装MySQL官方的 Yum Repository

```shell
[root@BrianZhu /]# wget -i -c http://dev.mysql.com/get/mysql57-community-release-el7-10.noarch.rpm
```

　　使用上面的命令就直接下载了安装用的Yum Repository，大概25KB的样子，然后就可以直接yum安装了。

```shell l
[root@BrianZhu /]# yum -y install mysql57-community-release-el7-10.noarch.rpm
```

　　下面就是使用yum安装MySQL了

```shell
[root@BrianZhu /]# yum -y install mysql-community-server

```

　　这步可能会花些时间，安装完成后就会覆盖掉之前的mariadb。

![img](https://images2018.cnblogs.com/blog/1072166/201803/1072166-20180315175306361-1976995930.png)

出现这样的提示表示安装成功

### 2. MySQL数据库设置

首先启动MySQL

```shell
[root@BrianZhu /]# systemctl start  mysqld.service
```

查看MySQL运行状态，运行状态如图：

```shell
[root@BrianZhu /]# systemctl status mysqld.service
```

![img](https://images2018.cnblogs.com/blog/1072166/201803/1072166-20180315175438838-730726112.png)

此时MySQL已经开始正常运行，不过要想进入MySQL还得先找出此时root用户的密码，通过如下命令可以在日志文件中找出密码：

```shell
[root@BrianZhu /]# grep "password" /var/log/mysqld.log
```

![img](https://images2018.cnblogs.com/blog/1072166/201803/1072166-20180315175527134-401244557.png)

上面标记的就是初始密码

 如下命令进入数据库：

```shell
[root@BrianZhu /]# mysql -uroot -p     # 回车后会提示输入密码
```

输入初始密码，此时不能做任何事情，因为MySQL默认必须修改密码之后才能操作数据库：

```shell
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'new password';
```

这里有个问题，新密码设置的时候如果设置的过于简单会报错：

![img](/Users/qishen/Documents/my-notebook/linux/linux mysql安装.assets/1072166-20180315175841014-874657230.png)

原因是因为MySQL有密码设置的规范，具体是与validate_password_policy的值有关：

![img](/Users/qishen/Documents/my-notebook/linux/linux mysql安装.assets/1072166-20180315175858792-1817095409.png)

MySQL完整的初始密码规则可以通过如下命令查看：

```
mysql> SHOW VARIABLES LIKE 'validate_password%';
+--------------------------------------+-------+
| Variable_name                        | Value |
+--------------------------------------+-------+
| validate_password_check_user_name    | OFF   |
| validate_password_dictionary_file    |       |
| validate_password_length             | 4     |
| validate_password_mixed_case_count   | 1     |
| validate_password_number_count       | 1     |
| validate_password_policy             | LOW   |
| validate_password_special_char_count | 1     |
+--------------------------------------+-------+
rows in set (0.01 sec)
```

 密码的长度是由validate_password_length决定的，而validate_password_length的计算公式是：

```
validate_password_length = validate_password_number_count + validate_password_special_char_count + (2 * validate_password_mixed_case_count)
```

　解决方法就是修改密码为规范复杂的密码：

```
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'z?guwrBhH7p>';
Query OK, 0 rows affected (0.00 sec)
 
mysql>
```

![img](/Users/qishen/Documents/my-notebook/linux/linux mysql安装.assets/1072166-20180315180147293-851658794.png)

这时候我们要把密码规则改一下，执行下面sql就可以了：

```
mysql> set global validate_password_policy=0;
Query OK, 0 rows affected (0.00 sec)
 
mysql> set global validate_password_length=1;
Query OK, 0 rows affected (0.00 sec)
 
mysql>
```

 设置之后就是我上面查出来的那几个值了，此时密码就可以设置的很简单，例如1234之类的。到此数据库的密码设置就完成了。

 但此时还有一个问题，就是因为安装了Yum Repository，以后每次yum操作都会自动更新，需要把这个卸载掉：

```
[root@BrianZhu ~]# yum -y remove mysql57-community-release-el7-10.noarch
```

　配置算是完成了

 

### 可视化工具的登录授权：(如果授权不成功，请查看防火墙)

操作完成上面的，现在还不能用可视化的客户端进行连接，需要我们进行授权：

```
grant all on *.* to root@'%' identified by '数据库密码';
```



## mysql赋予用户权限指令描述

```sql
grant 权限1,权限2,…权限n on 数据库名称.表名称 to 用户名@用户地址 identified by ‘连接口令’;
grant flush privileges;
```

权限1,权限2,…权限n代表select,insert,update,delete,create,drop,index,alter,grant,references,reload,shutdown,process,file等14个权限。

当权限1,权限2,…权限n被all privileges或者all代替，表示赋予用户全部权限。

当数据库名称.表名称被```*.*```代替，表示赋予用户操作服务器上所有数据库所有表的权限。

用户地址可以是localhost，也可以是ip地址、机器名字、域名。也可以用’%'表示从任何地址连接。

‘连接口令’不能为空，否则创建失败。

　　大功告成！！！





yum update 出现 Could not resolve host: repo.mysql.com; Unknown error解决办法

本地尝试下载rpm文件可以成功，服务器上异常，一般出现是因为DNS解析异常导致，换一个DNS服务即可解决

```shell
vi /etc/resolv.conf
```

增加如下服务器

```shell
nameserver 8.8.8.8
```

重启网络

```shell
systemctl restart network
```





# 离线安装模式

### **1.删除原有的mariadb**

**不然安装报错**

```
rpm -qa|grep mariadb
```

 ![img](/Users/qishen/Documents/my-notebook/linux/linux mysql安装.assets/591946-20180702140957497-2131683553.png)

```
rpm -e --nodeps mariadb-libs
```

### **2. 下载RPM安装包**

**在https://dev.mysql.com/downloads/mysql/选择为Red Hat Enterprise Linux 7 / Oracle Linux 7 ，把os的版本选择为all。** **直接下载mysql-5.7.21-1.el7.x86_64.rpm-bundle.tar，所有的rpm包都在里面，然后rpm命令安装。**

```
rpm -ivh mysql-community-common-5.7.21-1.el7.x86_64.rpm

rpm -ivh mysql-community-libs-5.7.21-1.el7.x86_64.rpm

rpm -ivh mysql-community-devel-5.7.21-1.el7.x86_64.rpm

rpm -ivh mysql-community-libs-compat-5.7.21-1.el7.x86_64.rpm

rpm -ivh mysql-community-client-5.7.21-1.el7.x86_64.rpm

rpm -ivh mysql-community-server-5.7.21-1.el7.x86_64.rpm
```

至此，mysql5.7所有文件安装完毕，接下来就是开启服务测试了

### **3. 启动mysql服务**

查看mysql服务是否启动

```
service mysqld status
```

启动服务：

```
systemctl start mysqld
```

### **4. 重置root密码**

MySQL5.7会在安装后为root用户生成一个随机密码，而不是像以往版本的空密码。 可以安全模式修改root登录密码或者用随机密码登录修改密码。下面用随机密码方式
MySQL为root用户生成的随机密码通过mysqld.log文件可以查找到：

 

```
grep 'temporary password' /var/log/mysqld.log
```

 

### **5. 修改root用户密码**

**(MySQL的密码策略比较复杂，过于简单的密码会被拒绝)**

```
1 mysql -u root -p
2 mysql> Enter password: （输入刚才查询到的随机密码）
3 mysql> SET PASSWORD FOR 'root'@'localhost'= "qaz-123";
4 mysql> exit
```

### **6. 用root新密码登录**

```
mysql -u root -pqaz-123
```

如果上面的方式不能修改可以使用下面安全模式修改root：
关闭服务，修改mysql配置文件:

```
1 systemctl stop mysqld.service
2 vi /etc/my.cnf
```

mysqld下面添加skip-grant-tables 保存退出启动服务。

```
systemctl start mysqld.service
mysql -u root #不用密码直接回车
use mysql
update user set authentication_string=password('qaz-123') where user='root' and host='localhost';
flush privileges;
exit;
vi /etc/my.cnf #把 skip-grant-tables 一句删除保存退出重启mysql服务
systemctl restart mysqld.service
```

 再次登录即可

```
mysql -uroot -pqaz123
```

如果进行操作出现下面的提示：

```
You must reset your password using ALTER USER statement before executing this statement.
```

就重新设置密码（mysql默认密码策略比较复杂，如果设置简单密码，需修改默认安全策略，可以参考另外一篇文章：[MYSQL57密码策略修改](https://www.cnblogs.com/mymelody/p/9253730.html)）

```
set password = password('qaz-123');
```

### **7.开放3306端口**

```
1 mysql>GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'qaz-123' WITH GRANT OPTION;
2 mysql>FLUSH PRIVILEGES;
3 mysql>exit;
```

开启防火墙mysql 3306端口的外部访问：

```shell
1 firewall-cmd --zone=public --add-port=3306/tcp --permanent
2 firewall-cmd --reload
```