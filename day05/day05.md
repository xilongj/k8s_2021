## [Apollo](https://github.com/ctripcorp/apollo/)

* Apollo（阿波罗）是携程框架部门研发的分布式配置中心，能够集中化管理应用不同环境、不同集群的配置，配置修改后能够实时推送到应用端，并且具备规范的权限、流程治理等特性，适用于微服务配置管理场景。
* 服务端基于Spring Boot和Spring Cloud开发，打包后可以直接运行，不需要额外安装Tomcat等应用容器。
* Java客户端不依赖任何框架，能够运行于所有Java运行时环境，同时对Spring/Spring Boot环境也有较好的支持。

### Install MySQL (5.6+)
#### [hdss7-11]
```text
# add repo
[root@hdss7-11 ~]# rpm --import https://mirrors.ustc.edu.cn/mariadb/yum/RPM-GPG-KEY-MariaDB
[root@hdss7-11 ~]# cat /etc/yum.repos.d/MariaDB.repo
[mariadb]
name = MariaDB
baseurl = https://mirrors.ustc.edu.cn/mariadb/yum/10.1/centos7-amd64/
gpgkey=https://mirrors.ustc.edu.cn/mariadb/yum/RPM-GPG-KEY-MariaDB
gpgcheck=1

[root@hdss7-11 ~]# yum list mariadb-server --show-duplicates
```
```text
[root@hdss7-11 ~]# yum -y install mariadb-server

[root@hdss7-11 ~]# rpm -qa MariaDB-server
MariaDB-server-10.1.48-1.el7.centos.x86_64
```
```text
[root@hdss7-11 ~]# cat /etc/my.cnf.d/server.cnf
[mysqld]
character_set_server = utf8mb4
collation_server = utf8mb4_general_ci
init_connect = "SET NAMES 'utf8mb4'"

[root@hdss7-11 ~]# cat /etc/my.cnf.d/mysql-clients.cnf
[mysql]
default-character-set = utf8mb4
```
```text
[root@hdss7-11 ~]# systemctl start mariadb

[root@hdss7-11 ~]# netstat -lntup | grep 3306
tcp6       0      0 :::3306                 :::*                    LISTEN      27443/mysqld

[root@hdss7-11 ~]# systemctl enable mariadb.service
Created symlink from /etc/systemd/system/mysql.service to /usr/lib/systemd/system/mariadb.service.
Created symlink from /etc/systemd/system/mysqld.service to /usr/lib/systemd/system/mariadb.service.
Created symlink from /etc/systemd/system/multi-user.target.wants/mariadb.service to /usr/lib/systemd/system/mariadb.service.
```
### Set up password
```text
[root@hdss7-11 ~]# mysqladmin -u root password
New password: (123456)
Confirm new password: (123456)
```
```text
# check character set
[root@hdss7-11 ~]# mysql -u root -p
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 5
Server version: 10.1.48-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> \s
--------------
mysql  Ver 15.1 Distrib 10.1.48-MariaDB, for Linux (x86_64) using readline 5.1

Connection id:          5
Current database:
Current user:           root@localhost
SSL:                    Not in use
Current pager:          stdout
Using outfile:          ''
Using delimiter:        ;
Server:                 MariaDB
Server version:         10.1.48-MariaDB MariaDB Server
Protocol version:       10
Connection:             Localhost via UNIX socket
Server characterset:    utf8mb4
Db     characterset:    utf8mb4
Client characterset:    utf8mb4
Conn.  characterset:    utf8mb4
UNIX socket:            /var/lib/mysql/mysql.sock
Uptime:                 28 min 42 sec

Threads: 1  Questions: 11  Slow queries: 0  Opens: 17  Flush tables: 1  Open tables: 11  Queries per second avg: 0.006
--------------

MariaDB [(none)]>
```
```text
# drop test DB (optional)
MariaDB [(none)]> drop database test;
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
+--------------------+
3 rows in set (0.00 sec)

MariaDB [(none)]> exit
Bye
```
### DB Script
```text
URL = https://raw.githubusercontent.com/ctripcorp/apollo/1.5.1/scripts/db/migration/configdb/V1.0.0__initialization.sql
[root@hdss7-11 ~]# wget URL -O apolloconfigdb.sql
```

### Import script
```text
[root@hdss7-11 ~]# ls
anaconda-ks.cfg  apolloconfig.sql

[root@hdss7-11 ~]# mysql -u root -p < apolloconfig.sql
Enter password:
[root@hdss7-11 ~]# mysql -u root -p
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 10
Server version: 10.1.48-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| ApolloConfigDB     |
| information_schema |
| mysql              |
| performance_schema |
+--------------------+
4 rows in set (0.00 sec)
```
```text
MariaDB [(none)]> use ApolloConfigDB;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
MariaDB [ApolloConfigDB]> grant INSERT,DELETE,UPDATE,SELECT on ApollConfigDB.* to "apolloconfig"@"10.4.7.%" identified by "123456";
Query OK, 0 rows affected (0.01 sec)

MariaDB [ApolloConfigDB]> select user,host from mysql.user;
+--------------+-------------------+
| user         | host              |
+--------------+-------------------+
| apolloconfig | 10.4.7.%          |
| root         | 127.0.0.1         |
| root         | ::1               |
|              | hdss7-11.host.com |
| root         | hdss7-11.host.com |
|              | localhost         |
| root         | localhost         |
+--------------+-------------------+
7 rows in set (0.01 sec)
```
```text
MariaDB [ApolloConfigDB]> show tables;
+--------------------------+
| Tables_in_ApolloConfigDB |
+--------------------------+
| App                      |
| AppNamespace             |
| Audit                    |
| Cluster                  |
| Commit                   |
| GrayReleaseRule          |
| Instance                 |
| InstanceConfig           |
| Item                     |
| Namespace                |
| NamespaceLock            |
| Release                  |
| ReleaseHistory           |
| ReleaseMessage           |
| ServerConfig             |
+--------------------------+
15 rows in set (0.00 sec)

MariaDB [ApolloConfigDB]> select * from ServerConfig \G
*************************** 1. row ***************************
                       Id: 1
                      Key: eureka.service.url
                  Cluster: default
                    Value: http://localhost:8080/eureka/
                  Comment: Eureka服务Url，多个service以英文逗号分隔
                IsDeleted:
     DataChange_CreatedBy: default
   DataChange_CreatedTime: 2021-02-26 00:18:49
DataChange_LastModifiedBy:
      DataChange_LastTime: 2021-02-26 00:18:49
*************************** 2. row ***************************
                       Id: 2
                      Key: namespace.lock.switch
                  Cluster: default
                    Value: false
                  Comment: 一次发布只能有一个人修改开关
                IsDeleted:
     DataChange_CreatedBy: default
   DataChange_CreatedTime: 2021-02-26 00:18:49
DataChange_LastModifiedBy:
      DataChange_LastTime: 2021-02-26 00:18:49
*************************** 3. row ***************************
                       Id: 3
                      Key: item.key.length.limit
                  Cluster: default
                    Value: 128
                  Comment: item key 最大长度限制
                IsDeleted:
     DataChange_CreatedBy: default
   DataChange_CreatedTime: 2021-02-26 00:18:49
DataChange_LastModifiedBy:
      DataChange_LastTime: 2021-02-26 00:18:49
*************************** 4. row ***************************
                       Id: 4
                      Key: item.value.length.limit
                  Cluster: default
                    Value: 20000
                  Comment: item value最大长度限制
                IsDeleted:
     DataChange_CreatedBy: default
   DataChange_CreatedTime: 2021-02-26 00:18:49
DataChange_LastModifiedBy:
      DataChange_LastTime: 2021-02-26 00:18:49
*************************** 5. row ***************************
                       Id: 5
                      Key: config-service.cache.enabled
                  Cluster: default
                    Value: false
                  Comment: ConfigService是否开启缓存，开启后能提高性能，但是会增大内存消耗！
                IsDeleted:
     DataChange_CreatedBy: default
   DataChange_CreatedTime: 2021-02-26 00:18:49
DataChange_LastModifiedBy:
      DataChange_LastTime: 2021-02-26 00:18:49
5 rows in set (0.00 sec)

MariaDB [ApolloConfigDB]>
```
```text
MariaDB [ApolloConfigDB]> update ApolloConfigDB.ServerConfig set ServerConfig.Value="http://config.od.com/eureka" where ServerConfig.Key="eureka.service.url";
Query OK, 1 row affected (0.01 sec)
Rows matched: 1  Changed: 1  Warnings: 0
```
### Update DNS
#### [hdss7-11]
```text
[root@hdss7-11 ~]# cat /var/named/od.com.zone
$ORIGIN od.com.
$TTL 600        ; 10 minutes
@       IN SOA dns.od.com. dnsadmin.od.com. (
                        2021021010      ; serial
                        10800           ; refresh (3 hours)
                        900             ; retry (15 minutes)
                        604800          ; expire (1 week)
                        86400           ; minimum (1 day)
                        )
                        NS      dns.od.com.
$TTL 60 ; 1 minute
dns             A       10.4.7.11
harbor          A       10.4.7.200
k8s-yaml        A       10.4.7.200
traefik         A       10.4.7.10
dashboard       A       10.4.7.10
zk1             A       10.4.7.11
zk2             A       10.4.7.12
zk3             A       10.4.7.21
jenkins         A       10.4.7.10
dubbo-monitor   A       10.4.7.10
demo            A       10.4.7.10
config          A       10.4.7.10
```
```text
[root@hdss7-11 ~]# systemctl restart named

[root@hdss7-21 ~]# dig -t A config.od.com @192.168.0.2 +short
10.4.7.10
```
37:38