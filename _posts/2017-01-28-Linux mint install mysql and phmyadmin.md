---
layout: post
title: Linux mint 安装  mysql 与 phpmyadmin
categories: mysql
tag:
- Linux
---

## mysql 

### 1. 安装mysql

sudo apt-get install mysql-server

### 2. 设置默认字符集

mysql 的配置文件位于 /etc/mysql/my.cnf


$ sudo cp /etc/mysql/my.cnf /etc/mysql/my.cnf.bak

$ sudo cp /etc/mysql/my.cnf 

```
[client]
default-character-set = utf8

[mysqld]
default-storage-engine = INNODB
character-set-server = utf8
collation-server = utf8_general_ci
```
登录 mysql 查看是否设置成功.

```
$ mysql -u root -p
Enter password: 
Welcome to the MySQL monitor...
...

mysql> show variables like '%char%';
+--------------------------+--------------------------------------------------------+
| Variable_name            | Value                                                  |
+--------------------------+--------------------------------------------------------+
| character_set_client     | utf8                                                   |
| character_set_connection | utf8                                                   |
| character_set_database   | utf8                                                   |
| character_set_filesystem | binary                                                 |
| character_set_results    | utf8                                                   |
| character_set_server     | utf8                                                   |
| character_set_system     | utf8                                                   |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+--------------------------------------------------------+
8 rows in set (0.00 sec)
```
# phpmyadmin

 安装 php apache2服务器 phpmyadmin

sudo apt-get install php5-cli
 
sudo apt-get install apache2
 
sudo apt-get install phpmyadmin

phpmyadmin 的网站目录位于/usr/share/phpmyadmin,
在 apache2 的默认目录 /var/www/html 下创建 phpmyadmin 的软连接

sudo ln -s /usr/share/phpmyadmin /var/www/html

重启 apache2 服务器
sudo service apache2 restart

打开 本地网址 [127.0.0.1/phpmyadmin](http://127.0.0.1/phpmyadmin),输入 mysql 账号密码就可以登录了.

