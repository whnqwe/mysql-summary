## centos7安装mysql5.7操作步骤

#### 下载mysql的repo源

> wget http://repo.mysql.com/mysql57-community-release-el7-8.noarch.rpm

#### 安装源

>  rpm -ivh mysql57-community-release-el7-8.noarch.rpm

#### 安装数据库

>  yum install mysql-server

#### 启动数据库

>  systemctl start mysqld



#### 登录到mysql



1. 5.7版本默认对于root帐号有一个随机密码，可以通过 grep "password" /var/log/mysqld.log获得，root@localhost: 此处为随机密码
2. 运行mysql -uroot -p 回车
3. 粘贴随机密码



#### 操作

- 默认的随机密码是没办法直接对数据库做操作的，需要修改密码，然后，5.7版本用了validate_password密码加强插件，因此在修改密码的时候绝对不是 123456 能糊弄过去的。需要严格按照规范去设置密码



> 如果想让密码简单点也可以，降低安全策略， 登录到mysql客户端执行如下两条命令


> set global validate_password_length=1;


> set global validate_password_policy=0; 
>
> 这样就能设置简单的密码了，但是密码长度必须是大于等于4位



#### 赋权操作

> 默认情况下其他服务器的客户端不能直接访问mysql服务端，需要对ip授权

```mysql
mysql>GRANT ALL PRIVILEGES ON *.*  TO 'root'@'%' IDENTIFIED BY 'root' WITH GRANT OPTION;
mysql>flush privilege
```

```mysql
-- 给来自10.163.225.87的用户joe分配可对数据库vtdc所有表进行所有操作的权限，并设定口令为123
mysql>grant  all  privileges  on  vtdc.*  to  joe@'10.163.255.87'  identified  by  '123';
```






