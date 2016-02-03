title: ToughMySQL，为ToughRADIUS提供数据库双机热备支持
date: 2016-02-04 02:39:38
tags: [mysql,docker]
---

ToughMySQL是一个基于Docker技术的MySQL应用，一开始它就是为了ToughRADIUS提供一个简单可靠易用的数据库服务。

现在 ToughMySQL发布了第一个正式的版本 v0.0.1

## 功能特性：

- 实现MySQL Docker容器部署。
- 提供针对不同服务器配置环境的优化配置。
- 提供一键脚本快速安装。
- 提供备份脚本，支持7天以上备份自动删除。
- 提供主从，互为主备的快速配置。

## 快速指南

tmshell是一个自动化安装和管理脚本，通过这个脚本，提供了很多有用的管理功能

    $ wget https://github.com/talkincode/toughmysql/raw/master/tmshell -O /usr/local/bin/tmshell
    $ chmod +x /usr/local/bin/tmshell
    $ tmshell install

直接输入 tmshell 可以看到支持的指令操作

    usage: tmshell [OPTIONS] instance
    
        docker_setup                install docker, docker-compose
        pull                        mysql docker images pull
        install                     install default mysql instance
        remove                      uninstall mysql instance
        config                      mysql instance config edit
        status                      mysql instance status
        restart                     mysql instance restart
        stop                        mysql instance stop
        logs                        mysql instance logs
        showmaster                  mysql instance show master status
        showslave                   mysql instance show slave status
        upmaster                    mysql instance update master sync config
        backup                      mysql instance backup database
        dsh                         mysql instance bash term
    
    All other options are passed to the tmshell program.

### 完整的安装过程

安装过程是一个交互式的过程，根据实际情况修改具体参数，注意如果打算以热备模式部署，需要输入server id

    [root@i-jahnm3dt ~]# tmshell install
    mysql user [raduser]:
    mysql user password [radpwd]:
    mysql database [radiusd]:
    mysql root password [none]:
    mysql replication password [replication]:
    mysql port [3306]:
    mysql server id [1,2...](default none): 1
    mysql max memary [512M,1G,4G](default none):
    
    ToughMySQL instance config:
    
    instance name: mysql
    mysql_user: raduser
    mysql_password: radpwd
    mysql_database: radiusd
    mysql_root_password:
    mysql_repl_password: replication
    mysql_port: 3306
    serverid: 1
    mysql_max_mem:
    
    
    database:
        container_name: db_mysql
        image: "index.alauda.cn/toughstruct/mysql"
        privileged: true
        ports:
            -"3306:3306"
        ulimits:
            nproc: 65535
            nofile:
                soft: 20000
                hard: 40000
            environment:
                - SERVERID=1
                - MYSQL_MAX_MEM=
                - MYSQL_USER=raduser
                - MYSQL_PASSWORD=radpwd
                - MYSQL_DATABASE=radiusd
                - MYSQL_ROOT_PASSWORD=
                - MYSQL_REPL_PASSWORD=replication
        restart: always
        volumes:
            /home/toughrun/mysql/dbmysql:/var/lib/mysql
            /home/toughrun/mysql/backup:/var/backup
    
    Creating db_mysql
      Name          Command         State           Ports
    ----------------------------------------------------------
    db_mysql   /usr/local/bin/run   Up      0.0.0.0:3306->3306/tcp
    

在另一台服务器上按同样的步骤部署mysql实例，注意server id必须不一样

> 具体过程略

查看每一个实例的 master status 

服务器1：

    [root@i-jahnm3dt ~]# tmshell showmaster
    *************************** 1. row ***************************
    File: mysql-bin.000002
    Position: 107
    Binlog_Do_DB:
    Binlog_Ignore_DB: test,mysql
    hosts:
    172.17.0.1
    10.61.105.29

服务器2：

    [root@i-jahnm3et ~]# tmshell showmaster
    *************************** 1. row ***************************
    File: mysql-bin.000002
    Position: 107
    Binlog_Do_DB:
    Binlog_Ignore_DB: test,mysql
    hosts:
    172.17.0.1
    10.60.128.105

在服务器1上执行：

    [root@i-jahnm3dt ~]# tmshell upmaster
    MASTER_HOST: 10.60.128.105
    MASTER_PORT (3306): 
    MASTER_REPL_PASSWORD (replication):
    MASTER_LOG_FILE: mysql-bin.000002
    MASTER_LOG_POS: 107

在服务器2上执行：

    [root@i-jahnm3et ~]# tmshell upmaster
    MASTER_HOST: 10.61.105.29
    MASTER_PORT (3306): 
    MASTER_REPL_PASSWORD (replication):
    MASTER_LOG_FILE: mysql-bin.000002
    MASTER_LOG_POS: 107

最后检查每个服务器上的slave状态：

    root@i-jahnm3dt ~]# tmshell showslave
    *************************** 1. row ***************************
      Slave_IO_State: Waiting for master to send event
      Master_Host: 10.60.128.105
      Master_User: repl
      Master_Port: 3306
      Connect_Retry: 60
      Master_Log_File: mysql-bin.000002
      Read_Master_Log_Pos: 107
      Relay_Log_File: mysqld-relay-bin.000002
      Relay_Log_Pos: 253
      Relay_Master_Log_File: mysql-bin.000002
      Slave_IO_Running: Yes
      Slave_SQL_Running: Yes

如果以下两项显示为：

      Slave_IO_Running: Yes
      Slave_SQL_Running: Yes

说明我们的mysql互为主备的配置成功了。现在我们的两个MySQL节点都可以提供服务了，并且实时热备。

