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
    shell> apt-get install -y gcc g++ cmake make ncurses-dev
    shell> cmake . # 若出错试试将前次编译的残余文件CMakeCache.txt删除
    shell> make
    shell> make install
    # End of source-build specific instructions
    # Postinstallation setup
    shell> cd /usr/local/mysql
    shell> chown -R mysql .
    shell> chgrp -R mysql .
    shell> scripts/mysql_install_db --user=mysql
    shell> chown -R root .
    shell> chown -R mysql data

#### 启动 from your own my.cnf

    shell> sudo mkdir /your/mysql/data/path/
    shell> sudo chown mysql:mysql /your/mysql/data/path/
    shell> cd /your/mysql/data/path/
    shell> su root
    # 把my.cnf也放到和数据一起
    # 确定conf文件中的各项配置是否正确
    shell> cd /data/path
    shell> mkdir -p $tmpdir && chown -R mysql:mysql $tmpdir # $tmpdir 在my.cnf设置
    shell> /usr/local/mysql/scripts/mysql_install_db --defaults-file=./my.cnf --user=mysql --basedir=/usr/local/mysql
    shell> mysqld_safe --defaults-file=./my.cnf --user=mysql &
    shell> mysqladmin --defaults-file=./my.cnf --user=mysql -u root password 'new-password'
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
    mysql> GRANT ALL PRIVILEGES ON *.* TO 'root'@'192.168.1.%' IDENTIFIED BY 'password' WITH GRANT OPTION;
    mysql> use mysql;
    # delete all users whose password is null
    mysql> delete from user where Password=''

