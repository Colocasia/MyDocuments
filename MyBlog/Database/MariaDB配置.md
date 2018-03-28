
[TOC]

# **MariaDB 安装和配置** #

好记性不如烂笔头啊，还是记录一下!

----------

## **<font color=#191970 size=5>1.添加 MariaDB yum 仓库</font> ** ##

首先在CentOS中添加MariaDB的YUM配置文件MariaDB.repo文件。

``` shell
vi /etc/yum.repos.d/MariaDB.repo
```

添加如下配置信息：

``` shell
# MariaDB 10.3 CentOS repository list - created 2018-03-10 06:28 UTC
# http://downloads.mariadb.org/mariadb/repositories/
[mariadb]
name=MariaDB
baseurl=http://yum.mariadb.org/10.3/centos7-amd64
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1
```

## **<font color=#191970 size=5>2.在 CentOS 7 中安装 MariaDB</font> ** ##

当 MariaDB 仓库地址添加好后，你可以通过下面的一行命令轻松安装 MariaDB。

``` shell
sudo yum install MariaDB-server MariaDB-client
```

MariaDB 包安装完毕后，立即启动数据库服务守护进程，并可以通过下面的操作设置，在操作系统重启后自动启动服务。

``` shell
systemctl start mariadb
systemctl enable mariadb
systemctl status mariadb
```

## **<font color=#191970 size=5>3.在 CentOS 7 中对 MariaDB 进行安全配置</font> ** ##

现在可以通过以下操作进行安全配置：设置 MariaDB 的 root 账户密码，禁用 root 远程登录，删除测试数据库以及测试帐号，最后需要使用下面的命令重新加载权限。

``` shell
mysql_secure_installation
```

Enter current password for root (enter for none):<–初次运行直接回车

设置密码
Set root password? [Y/n] <– 是否设置root用户密码，输入y并回车或直接回车<br>
New password: <– 设置root用户的密码<br>
Re-enter new password: <– 再输入一次你设置的密码<br>

其他配置
Remove anonymous users? [Y/n] <– 是否删除匿名用户，回车<br>
Disallow root login remotely? [Y/n] <–是否禁止root远程登录,回车,<br>
Remove test database and access to it? [Y/n] <– 是否删除test数据库，回车<br>
Reload privilege tables now? [Y/n] <– 是否重新加载权限表，回车<br>

``` shell
mysql -V
mysqld --print-defaults
mysql -u root -p
```

完成。

## **<font color=#191970 size=5>3.在 CentOS 7 中配置 MariaDB 的字符集</font> ** ##

编辑文件/etc/my.cnf

``` shell
vi /etc/my.cnf
```

在[mysqld]标签下添加

``` shell
[mysqld]
init_connect='SET collation_connection = utf8_general_ci'
init_connect='SET NAMES utf8'
character_set_server=utf8
collation_server=utf8_general_ci
```

在[mysqld_safe]标签下添加

``` shell
[mysqld_safe]
init_connect='SET collation_connection = utf8_general_ci'
init_connect='SET NAMES utf8'
character_set_server=utf8
collation_server=utf8_general_ci
```

在[mysqld_safe]标签下添加

``` shell
[mysqld_safe]
init_connect='SET collation_connection = utf8_general_ci'
init_connect='SET NAMES utf8'
character_set_server=utf8
collation_server=utf8_general_ci
```

在[mysql]中添加

``` shell
[mysql]
default-character-set=utf8
```

在[mysql.server]中添加

``` shell
[mysql.server]
default-character-set=utf8
```

在[client]中添加

``` shell
[client]
default-character-set=utf8
```

全部配置完成，重启mariadb

``` shell
systemctl restart mariadb
```

之后进入MariaDB查看字符集

``` shell
mysql> show variables like "%character%";show variables like "%collation%";
```


显示为
<br><br>
+--------------------------+----------------------------+<br>
| Variable_name            | Value                      |<br>
+--------------------------+----------------------------+<br>
| character_set_client    | utf8                      |<br>
| character_set_connection | utf8                      |<br>
| character_set_database  | utf8                      |<br>
| character_set_filesystem | binary                    |<br>
| character_set_results    | utf8                      |<br>
| character_set_server    | utf8                      |<br>
| character_set_system    | utf8                      |<br>
| character_sets_dir      | /usr/share/mysql/charsets/ |<br>
+--------------------------+----------------------------+<br>
8 rows in set (0.00 sec)<br>
<br>
+----------------------+-----------------+<br>
| Variable_name        | Value          |<br>
+----------------------+-----------------+<br>
| collation_connection | utf8_unicode_ci |<br>
| collation_database  | utf8_unicode_ci |<br>
| collation_server    | utf8_unicode_ci |<br>
+----------------------+-----------------+<br>
3 rows in set (0.00 sec)<br>

字符集配置完成。

