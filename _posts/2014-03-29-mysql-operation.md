---
layout: post
title: MySQL Operation
---

## MySQL Setup

#### 安装 from source(.tar.gz)

    # 添加mysql组和用户
    shell> groupadd mysql
    shell> useradd -g mysql mysql

    # build from source
    shell> tar zxvf mysql-VERSION.tar.gz
    shell> cd mysql-VERSION
    shell> ./configure --prefix=/usr/local/mysql
    shell> make
    shell> make install

    # Postinstallation setup
    shell> cd /usr/local/mysql
    shell> chown -R mysql .
    shell> chgrp -R mysql .
    shell> bin/mysql_install_db --user=mysql
    shell> chown -R root .
    shell> chown -R mysql var

#### 启动 from your own my.cnf

    shell> cd /your/mysql/data/path/
    shell> su root
    shell> mysql_install_db --defaults-file=/your/own/cnf/path/my.cnf --user=mysql
    shell> mysqld_safe --defaults-file=/your/own/cnf/path/my.cnf --user=mysql &
    shell> mysqladmin --defaults-file=/your/own/cnf/path/my.cnf --user=mysql -u root password 'new-password'
    # delete the password configuration command history from shell
    # show PASSWORD-CONF-COMMAND-HIS-NUM
    shell> history | less
    shell> history -d PASSWORD-CONF-COMMAND-HIS-NUM
    # exit the root shell
    shell> exit

#### 设置 for security
    
Connect to the mysql server from command client:
    
    # delete the test database
    mysql> drop database test;
    # LAN connection limitation
    mysql> GRANT ALL PRIVILEGES ON *.* TO 'root'@'192.168.1.%' INDENTIFIED BY 'password' WITH GRANT OPTION;
    mysql> use mysql;
    # delete all users whose password is null
    mysql> delete from user where Password=''

