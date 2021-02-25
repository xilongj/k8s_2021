## [Apollo](https://github.com/ctripcorp/apollo/)

Apollo（阿波罗）是携程框架部门研发的分布式配置中心，能够集中化管理应用不同环境、不同集群的配置，配置修改后能够实时推送到应用端，并且具备规范的权限、流程治理等特性，适用于微服务配置管理场景。
服务端基于Spring Boot和Spring Cloud开发，打包后可以直接运行，不需要额外安装Tomcat等应用容器。
Java客户端不依赖任何框架，能够运行于所有Java运行时环境，同时对Spring/Spring Boot环境也有较好的支持。

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
### set up password
```text
[root@hdss7-11 ~]# mysqladmin -u root password
New password: (123456)
Confirm new password: (123456)
```
