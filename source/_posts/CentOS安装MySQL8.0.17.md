#### 前言

​			**由于项目部署需要，我在阿里云云服务器上安装`MySQL`，并进行相应的配置。操作系统是`CentOS`，安装的`MySQL`版本为8.0.17，由于8.0版本较旧版本改动了些许内容，所以存在一些坑。**



#### 第一步：检查本机是否已安装了，如果有，需要先删除

```bash
使用rpm安装的mysql
	1) rpm -qa | grep -i mysql 查看系统中的mysql包
    2) rpm -e "包名" 删除所有的mysql包
    3) chkconfig -list | grep -i mysql 查看mysql相关服务
    4) chkconfig -del mysql 删除mysql相关服务
    5) find / -name mysql 查找与mysql相关文件夹，使用rm -rf删除掉
```



#### 第二步：安装MySQL 8.0.17

```bash
1、下载安装包
	rpm -ivh http://dev.mysql.com/get/mysql57-community-release-el7-8.noarch.rpm
	
2、安装mysql
	yum install -y mysql-server
	
3、设置开机启动
	systemctl enable mysqld.service
	
4、检查自启动设置结果
	systemctl list-unit-files | grep mysqld
	
5、开启服务
    systemctl start mysqld.service/ service mysqld start
    
6、查看服务状态
	systemctl status mysqld.service/ service mysqld status
```



#### 第三步：配置MySQL

```bash
1、初始密码为空，需要自行设置
	1) 登录：mysql -u root -p + enter
	2) 设置密码：mysql> alter user 'root'@'localhost' identified by 'your password';
	3) 刷新设置：mysql> flush privileges;
	
2、配置远程访问（使用root用户登录mysql）
	1) 使用mysql: mysql> use mysql;
	2) 创建用户用于远程访问：mysql> create user 'xixi'@'%' identified by 'your password';
	3) 修改加密规则：mysql> alter user 'xixi'@'%' identified by 'your password' password expire never;
	4) 更新用户密码：mysql> alter user 'xixi'@'%' identified with mysql_native_password by 'your password';
	5) 赋予权限：grant all on *.* to 'xixi'@'%' identified by "your password" with grant option;
	5) 刷新设置：mysql> flush privileges;
	6) 查看用户信息：mysql> select user,host,plugin from user;
```



#### 最后一步：在阿里云服务器上配置好3306端口映射



#### 高版本修改密码需要注意的问题

			##### 		`MySQL 8.0`后使用alter来修改用户密码，可以通过上面查看用户信息的命令知道，在`MySQL 8.0`以后的加密方式为caching_sha2_password，所以如果在一开始就使用alter设置密码的话不会出现问题，但是如果使用`update`修改密码，之后使用alter修改密码会报`Operation ALTER USER failed for 'root'@'localhost'`，因为此时root用户的authentication_string字段已经被设置了值，此时需要先清空authentication_string字段，再进行密码修改操作。

```bash
//使用了update设置密码
mysql> alter user 'root'@'localhost' identified with mysql_native_password by 'your password';
ERROR 1396 (HY000): Operation ALTER USER failed for 'root'@'localhost'

//此时需要将authentication_string置空，然后才能修改密码
mysql> update user set authentication_string='' where user='root';
mysql> alter user 'root'@'localhost' identified with mysql_native_password by 'your password';
mysql> flush privileges;
```



*Thank you.*



