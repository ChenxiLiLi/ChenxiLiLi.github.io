#### 前言

​			**由于项目部署需要，我在阿里云云服务器上安装`MySQL`，并进行相应的配置。操作系统是`CentOS 8.064位 `，安装的`MySQL`版本为8.0.17，由于8.0版本较旧版本改动了些许内容，所以存在一些坑。**



#### 第一步：检查本机是否已安装了，如果有，需要先删除

使用rpm安装的mysql

```bash
1、rpm -qa | grep -i mysql 查看系统中的mysql包
2、rpm -e "包名" 删除所有的mysql包
3、chkconfig -list | grep -i mysql 查看mysql相关服务
4、chkconfig -del mysql 删除mysql相关服务
5、find / -name mysql 查找与mysql相关文件夹，使用rm -rf删除掉
```



#### 第二步：安装MySQL 8.0.17

1、下载安装包

```bash
[root@iZczg7jn8r9p6pZ ~]# rpm -ivh http://dev.mysql.com/get/mysql57-community-release-el7-8.noarch.rpm
```

2、安装mysql

```bas
[root@iZczg7jn8r9p6pZ ~]# yum install -y mysql-server
```

3、设置开机启动

```bash
[root@iZczg7jn8r9p6pZ ~]# systemctl enable mysqld.service
```

4、检查自启动设置结果

```bash
[root@iZczg7jn8r9p6pZ ~]# systemctl list-unit-files | grep mysqld
```

5、开启服务(两个命令都OK)

```bash
[root@iZczg7jn8r9p6pZ ~]# systemctl start mysqld.service/ service mysqld start
```

6、查看服务状态(两个命令都OK)

```bash
[root@iZczg7jn8r9p6pZ ~]# systemctl status mysqld.service/ service mysqld status
```



#### 第三步：配置MySQL

1、初始密码为空，需要自行设置

```bash
[root@iZczg7jn8r9p6pZ ~]# mysql -u root -p
Enter Password:(回车)
mysql> alter user 'root'@'localhost' identified by 'your password';
mysql> flush privileges;
```

2、配置远程访问（使用root用户登录mysql）

```bash
mysql> use mysql;
//创建用户用于远程访问：
mysql> create user 'xixi'@'%' identified by 'your password'; 
//修改加密规则：
mysql> alter user 'xixi'@'%' identified by 'your password' password expire never; 	
//更新用户密码：
mysql> alter user 'xixi'@'%' identified with mysql_native_password by 'your password';
//赋予权限
mysql> grant all on *.* to 'xixi'@'%' identified by "your password" with grant option;
//刷新设置
mysql> flush privileges;
//查看用户信息：
mysql> select user,host,plugin from user;
```

注意事项：配置完远程访问之后需要在阿里云服务器的安全组配置好3306端口映射。



#### 高版本修改密码需要注意的问题

**`MySQL 8.0`后使用alter来修改用户密码，可以通过上面查看用户信息的命令知道，在`MySQL 8.0`以后的加密方式为caching_sha2_password，所以如果在一开始就使用alter设置密码的话不会出现问题，但是如果使用`update`修改密码，之后使用alter修改密码会报`Operation ALTER USER failed for 'root'@'localhost'`，因为此时root用户的authentication_string字段已经被设置了值，此时需要先清空authentication_string字段，再进行密码修改操作。**

1、使用了update设置密码，会出现下面的情况

```bash
mysql> alter user 'root'@'localhost' identified with mysql_native_password by 'your password';
ERROR 1396 (HY000): Operation ALTER USER failed for 'root'@'localhost'
```

2、此时修改密码正确操作

```bash
mysql> update user set authentication_string='' where user='root';
mysql> alter user 'root'@'localhost' identified with mysql_native_password by 'your password';
mysql> flush privileges;
```



#### MySQL远程连接测试：使用Navicat

1、查看用户权限，如果host不为%，参照上面远程访问的配置

```bash
mysql> use mysql;
mysql> select user, host, plugin from user;
```

<img src="/images/mysql-connect1.PNG" style="zoom:110%;" />

2、查看MySQL Server是否监听了3306端口

```bash
[root@iZczg7jn8r9p6pZ ~]# netstat -tulpen
```

<img src="/images/mysql-connet2.PNG" style="zoom:100%;" />

3、没有监听的情况下，需要在my.cnf或mysql-server.cnf添加

```bash
bind-address=0.0.0.0
```

4、将MySQL服务加进防火墙

```bash
[root@iZczg7jn8r9p6pZ ~]# sudo firewall-cmd --zone=public --permanent --add-service=mysql
```

5、使用Navicat测试远程连接

![](/images/mysql-connect3.PNG)

*Thank you.*



