---
title: 安装配置mysql
date: 2016-12-15 10:05:16
tags:
    - mysql
categories:
    - note
---

# 0 安装mysql

```shell
sudo yum install -y mysql-server mysql mysql-devel
service mysqld start
```

过程比较简单，可以直接mysq server启动了，如果需要修改mysql数据的存放目录，可以修改my.conf文件里相应的配置。
```shell
# 查看my.conf位置
mysql --help | grep my.cnf
vi /your/path/my.conf
```

修改datadir等相应的内容，重启server

# 账号设置

mysql数据库安装完以后只会有一个root管理员账号，但是此时的root账号还并没有为其设置密码，需要设置。

```shell
/usr/bin/mysqladmin -u root password 'new-password'　　// 为root账号设置密码
```
## 添加账号并授权

添加一个test账号，并授权
```shell
CREATE USER test IDENTIFIED BY PASSWORD('password');
GRANT ALL PRIVILEGES ON *.* TO 'test'@'%';
FLUSH PRIVILEGES;
```
## 忘记root密码

root密码是管理整个mysql数据库的入口，所以忘记时需要找回。
```shell
#杀掉进程 
killall -TERM mysqld。
#启动 MySQL
bin/safe_mysqld --skip-grant-tables &
```

这样就可以不需要密码进行mysql

```shell
>use mysql
>update user set password=password("new_pass") where user="root";
>flush privileges;
```


