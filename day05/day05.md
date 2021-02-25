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
[root@hdss7-11 ~]# wget https://raw.githubusercontent.com/ctripcorp/apollo/1.5.1/scripts/db/migration/configdb/V1.0.0__initialization.sql -O apolloconfigdb.sql
```
<details>
  <summary>Apolloconfig.sql</summary>
  <pre><code> 
        [root@hdss7-11 ~]# cat apolloconfig.sql
        /*!40101 SET @OLD_CHARACTER_SET_CLIENT=@@CHARACTER_SET_CLIENT */;
        /*!40101 SET @OLD_CHARACTER_SET_RESULTS=@@CHARACTER_SET_RESULTS */;
        /*!40101 SET @OLD_COLLATION_CONNECTION=@@COLLATION_CONNECTION */;
        /*!40101 SET NAMES utf8 */;
        /*!40014 SET @OLD_FOREIGN_KEY_CHECKS=@@FOREIGN_KEY_CHECKS, FOREIGN_KEY_CHECKS=0 */;
        /*!40101 SET @OLD_SQL_MODE=@@SQL_MODE, SQL_MODE='NO_AUTO_VALUE_ON_ZERO' */;
        /*!40111 SET @OLD_SQL_NOTES=@@SQL_NOTES, SQL_NOTES=0 */;
        
        # Create Database
        # ------------------------------------------------------------
        CREATE DATABASE IF NOT EXISTS ApolloConfigDB DEFAULT CHARACTER SET = utf8mb4;
        
        Use ApolloConfigDB;
        
        # Dump of table app
        # ------------------------------------------------------------
        
        DROP TABLE IF EXISTS `App`;
        
        CREATE TABLE `App` (
          `Id` int(10) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键',
          `AppId` varchar(500) NOT NULL DEFAULT 'default' COMMENT 'AppID',
          `Name` varchar(500) NOT NULL DEFAULT 'default' COMMENT '应用名',
          `OrgId` varchar(32) NOT NULL DEFAULT 'default' COMMENT '部门Id',
          `OrgName` varchar(64) NOT NULL DEFAULT 'default' COMMENT '部门名字',
          `OwnerName` varchar(500) NOT NULL DEFAULT 'default' COMMENT 'ownerName',
          `OwnerEmail` varchar(500) NOT NULL DEFAULT 'default' COMMENT 'ownerEmail',
          `IsDeleted` bit(1) NOT NULL DEFAULT b'0' COMMENT '1: deleted, 0: normal',
          `DataChange_CreatedBy` varchar(32) NOT NULL DEFAULT 'default' COMMENT '创建人邮箱前缀',
          `DataChange_CreatedTime` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
          `DataChange_LastModifiedBy` varchar(32) DEFAULT '' COMMENT '最后修改人邮箱前缀',
          `DataChange_LastTime` timestamp NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '最后修改时间',
          PRIMARY KEY (`Id`),
          KEY `AppId` (`AppId`(191)),
          KEY `DataChange_LastTime` (`DataChange_LastTime`),
          KEY `IX_Name` (`Name`(191))
        ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='应用表';
        
        
        
        # Dump of table appnamespace
        # ------------------------------------------------------------
        
        DROP TABLE IF EXISTS `AppNamespace`;
        
        CREATE TABLE `AppNamespace` (
          `Id` int(10) unsigned NOT NULL AUTO_INCREMENT COMMENT '自增主键',
          `Name` varchar(32) NOT NULL DEFAULT '' COMMENT 'namespace名字，注意，需要全局唯一',
          `AppId` varchar(32) NOT NULL DEFAULT '' COMMENT 'app id',
          `Format` varchar(32) NOT NULL DEFAULT 'properties' COMMENT 'namespace的format类型',
          `IsPublic` bit(1) NOT NULL DEFAULT b'0' COMMENT 'namespace是否为公共',
          `Comment` varchar(64) NOT NULL DEFAULT '' COMMENT '注释',
          `IsDeleted` bit(1) NOT NULL DEFAULT b'0' COMMENT '1: deleted, 0: normal',
          `DataChange_CreatedBy` varchar(32) NOT NULL DEFAULT '' COMMENT '创建人邮箱前缀',
          `DataChange_CreatedTime` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
          `DataChange_LastModifiedBy` varchar(32) DEFAULT '' COMMENT '最后修改人邮箱前缀',
          `DataChange_LastTime` timestamp NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '最后修改时间',
          PRIMARY KEY (`Id`),
          KEY `IX_AppId` (`AppId`),
          KEY `Name_AppId` (`Name`,`AppId`),
          KEY `DataChange_LastTime` (`DataChange_LastTime`)
        ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='应用namespace定义';
        
        
        
        # Dump of table audit
        # ------------------------------------------------------------
        
        DROP TABLE IF EXISTS `Audit`;
        
        CREATE TABLE `Audit` (
          `Id` int(10) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键',
          `EntityName` varchar(50) NOT NULL DEFAULT 'default' COMMENT '表名',
          `EntityId` int(10) unsigned DEFAULT NULL COMMENT '记录ID',
          `OpName` varchar(50) NOT NULL DEFAULT 'default' COMMENT '操作类型',
          `Comment` varchar(500) DEFAULT NULL COMMENT '备注',
          `IsDeleted` bit(1) NOT NULL DEFAULT b'0' COMMENT '1: deleted, 0: normal',
          `DataChange_CreatedBy` varchar(32) NOT NULL DEFAULT 'default' COMMENT '创建人邮箱前缀',
          `DataChange_CreatedTime` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
          `DataChange_LastModifiedBy` varchar(32) DEFAULT '' COMMENT '最后修改人邮箱前缀',
          `DataChange_LastTime` timestamp NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '最后修改时间',
          PRIMARY KEY (`Id`),
          KEY `DataChange_LastTime` (`DataChange_LastTime`)
        ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='日志审计表';
        
        
        
        # Dump of table cluster
        # ------------------------------------------------------------
        
        DROP TABLE IF EXISTS `Cluster`;
        
        CREATE TABLE `Cluster` (
          `Id` int(10) unsigned NOT NULL AUTO_INCREMENT COMMENT '自增主键',
          `Name` varchar(32) NOT NULL DEFAULT '' COMMENT '集群名字',
          `AppId` varchar(32) NOT NULL DEFAULT '' COMMENT 'App id',
          `ParentClusterId` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '父cluster',
          `IsDeleted` bit(1) NOT NULL DEFAULT b'0' COMMENT '1: deleted, 0: normal',
          `DataChange_CreatedBy` varchar(32) NOT NULL DEFAULT '' COMMENT '创建人邮箱前缀',
          `DataChange_CreatedTime` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
          `DataChange_LastModifiedBy` varchar(32) DEFAULT '' COMMENT '最后修改人邮箱前缀',
          `DataChange_LastTime` timestamp NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '最后修改时间',
          PRIMARY KEY (`Id`),
          KEY `IX_AppId_Name` (`AppId`,`Name`),
          KEY `IX_ParentClusterId` (`ParentClusterId`),
          KEY `DataChange_LastTime` (`DataChange_LastTime`)
        ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='集群';
        
        
        
        # Dump of table commit
        # ------------------------------------------------------------
        
        DROP TABLE IF EXISTS `Commit`;
        
        CREATE TABLE `Commit` (
          `Id` int(10) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键',
          `ChangeSets` longtext NOT NULL COMMENT '修改变更集',
          `AppId` varchar(500) NOT NULL DEFAULT 'default' COMMENT 'AppID',
          `ClusterName` varchar(500) NOT NULL DEFAULT 'default' COMMENT 'ClusterName',
          `NamespaceName` varchar(500) NOT NULL DEFAULT 'default' COMMENT 'namespaceName',
          `Comment` varchar(500) DEFAULT NULL COMMENT '备注',
          `IsDeleted` bit(1) NOT NULL DEFAULT b'0' COMMENT '1: deleted, 0: normal',
          `DataChange_CreatedBy` varchar(32) NOT NULL DEFAULT 'default' COMMENT '创建人邮箱前缀',
          `DataChange_CreatedTime` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
          `DataChange_LastModifiedBy` varchar(32) DEFAULT '' COMMENT '最后修改人邮箱前缀',
          `DataChange_LastTime` timestamp NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '最后修改时间',
          PRIMARY KEY (`Id`),
          KEY `DataChange_LastTime` (`DataChange_LastTime`),
          KEY `AppId` (`AppId`(191)),
          KEY `ClusterName` (`ClusterName`(191)),
          KEY `NamespaceName` (`NamespaceName`(191))
        ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='commit 历史表';
        
        # Dump of table grayreleaserule
        # ------------------------------------------------------------
        
        DROP TABLE IF EXISTS `GrayReleaseRule`;
        
        CREATE TABLE `GrayReleaseRule` (
          `Id` int(11) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键',
          `AppId` varchar(32) NOT NULL DEFAULT 'default' COMMENT 'AppID',
          `ClusterName` varchar(32) NOT NULL DEFAULT 'default' COMMENT 'Cluster Name',
          `NamespaceName` varchar(32) NOT NULL DEFAULT 'default' COMMENT 'Namespace Name',
          `BranchName` varchar(32) NOT NULL DEFAULT 'default' COMMENT 'branch name',
          `Rules` varchar(16000) DEFAULT '[]' COMMENT '灰度规则',
          `ReleaseId` int(11) unsigned NOT NULL DEFAULT '0' COMMENT '灰度对应的release',
          `BranchStatus` tinyint(2) DEFAULT '1' COMMENT '灰度分支状态: 0:删除分支,1:正在使用的规则 2：全量发布',
          `IsDeleted` bit(1) NOT NULL DEFAULT b'0' COMMENT '1: deleted, 0: normal',
          `DataChange_CreatedBy` varchar(32) NOT NULL DEFAULT 'default' COMMENT '创建人邮箱前缀',
          `DataChange_CreatedTime` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
          `DataChange_LastModifiedBy` varchar(32) DEFAULT '' COMMENT '最后修改人邮箱前缀',
          `DataChange_LastTime` timestamp NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '最后修改时间',
          PRIMARY KEY (`Id`),
          KEY `DataChange_LastTime` (`DataChange_LastTime`),
          KEY `IX_Namespace` (`AppId`,`ClusterName`,`NamespaceName`)
        ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='灰度规则表';
        
        
        # Dump of table instance
        # ------------------------------------------------------------
        
        DROP TABLE IF EXISTS `Instance`;
        
        CREATE TABLE `Instance` (
          `Id` int(11) unsigned NOT NULL AUTO_INCREMENT COMMENT '自增Id',
          `AppId` varchar(32) NOT NULL DEFAULT 'default' COMMENT 'AppID',
          `ClusterName` varchar(32) NOT NULL DEFAULT 'default' COMMENT 'ClusterName',
          `DataCenter` varchar(64) NOT NULL DEFAULT 'default' COMMENT 'Data Center Name',
          `Ip` varchar(32) NOT NULL DEFAULT '' COMMENT 'instance ip',
          `DataChange_CreatedTime` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
          `DataChange_LastTime` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '最后修改时间',
          PRIMARY KEY (`Id`),
          UNIQUE KEY `IX_UNIQUE_KEY` (`AppId`,`ClusterName`,`Ip`,`DataCenter`),
          KEY `IX_IP` (`Ip`),
          KEY `IX_DataChange_LastTime` (`DataChange_LastTime`)
        ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='使用配置的应用实例';
        
        
        
        # Dump of table instanceconfig
        # ------------------------------------------------------------
        
        DROP TABLE IF EXISTS `InstanceConfig`;
        
        CREATE TABLE `InstanceConfig` (
          `Id` int(11) unsigned NOT NULL AUTO_INCREMENT COMMENT '自增Id',
          `InstanceId` int(11) unsigned DEFAULT NULL COMMENT 'Instance Id',
          `ConfigAppId` varchar(32) NOT NULL DEFAULT 'default' COMMENT 'Config App Id',
          `ConfigClusterName` varchar(32) NOT NULL DEFAULT 'default' COMMENT 'Config Cluster Name',
          `ConfigNamespaceName` varchar(32) NOT NULL DEFAULT 'default' COMMENT 'Config Namespace Name',
          `ReleaseKey` varchar(64) NOT NULL DEFAULT '' COMMENT '发布的Key',
          `ReleaseDeliveryTime` timestamp NULL DEFAULT NULL COMMENT '配置获取时间',
          `DataChange_CreatedTime` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
          `DataChange_LastTime` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '最后修改时间',
          PRIMARY KEY (`Id`),
          UNIQUE KEY `IX_UNIQUE_KEY` (`InstanceId`,`ConfigAppId`,`ConfigNamespaceName`),
          KEY `IX_ReleaseKey` (`ReleaseKey`),
          KEY `IX_DataChange_LastTime` (`DataChange_LastTime`),
          KEY `IX_Valid_Namespace` (`ConfigAppId`,`ConfigClusterName`,`ConfigNamespaceName`,`DataChange_LastTime`)
        ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='应用实例的配置信息';
        
        
        
        # Dump of table item
        # ------------------------------------------------------------
        
        DROP TABLE IF EXISTS `Item`;
        
        CREATE TABLE `Item` (
          `Id` int(10) unsigned NOT NULL AUTO_INCREMENT COMMENT '自增Id',
          `NamespaceId` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '集群NamespaceId',
          `Key` varchar(128) NOT NULL DEFAULT 'default' COMMENT '配置项Key',
          `Value` longtext NOT NULL COMMENT '配置项值',
          `Comment` varchar(1024) DEFAULT '' COMMENT '注释',
          `LineNum` int(10) unsigned DEFAULT '0' COMMENT '行号',
          `IsDeleted` bit(1) NOT NULL DEFAULT b'0' COMMENT '1: deleted, 0: normal',
          `DataChange_CreatedBy` varchar(32) NOT NULL DEFAULT 'default' COMMENT '创建人邮箱前缀',
          `DataChange_CreatedTime` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
          `DataChange_LastModifiedBy` varchar(32) DEFAULT '' COMMENT '最后修改人邮箱前缀',
          `DataChange_LastTime` timestamp NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '最后修改时间',
          PRIMARY KEY (`Id`),
          KEY `IX_GroupId` (`NamespaceId`),
          KEY `DataChange_LastTime` (`DataChange_LastTime`)
        ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='配置项目';
        
        
        
        # Dump of table namespace
        # ------------------------------------------------------------
        
        DROP TABLE IF EXISTS `Namespace`;
        
        CREATE TABLE `Namespace` (
          `Id` int(10) unsigned NOT NULL AUTO_INCREMENT COMMENT '自增主键',
          `AppId` varchar(500) NOT NULL DEFAULT 'default' COMMENT 'AppID',
          `ClusterName` varchar(500) NOT NULL DEFAULT 'default' COMMENT 'Cluster Name',
          `NamespaceName` varchar(500) NOT NULL DEFAULT 'default' COMMENT 'Namespace Name',
          `IsDeleted` bit(1) NOT NULL DEFAULT b'0' COMMENT '1: deleted, 0: normal',
          `DataChange_CreatedBy` varchar(32) NOT NULL DEFAULT 'default' COMMENT '创建人邮箱前缀',
          `DataChange_CreatedTime` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
          `DataChange_LastModifiedBy` varchar(32) DEFAULT '' COMMENT '最后修改人邮箱前缀',
          `DataChange_LastTime` timestamp NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '最后修改时间',
          PRIMARY KEY (`Id`),
          KEY `AppId_ClusterName_NamespaceName` (`AppId`(191),`ClusterName`(191),`NamespaceName`(191)),
          KEY `DataChange_LastTime` (`DataChange_LastTime`),
          KEY `IX_NamespaceName` (`NamespaceName`(191))
        ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='命名空间';
        
        
        
        # Dump of table namespacelock
        # ------------------------------------------------------------
        
        DROP TABLE IF EXISTS `NamespaceLock`;
        
        CREATE TABLE `NamespaceLock` (
          `Id` int(11) unsigned NOT NULL AUTO_INCREMENT COMMENT '自增id',
          `NamespaceId` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '集群NamespaceId',
          `DataChange_CreatedBy` varchar(32) NOT NULL DEFAULT 'default' COMMENT '创建人邮箱前缀',
          `DataChange_CreatedTime` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
          `DataChange_LastModifiedBy` varchar(32) DEFAULT 'default' COMMENT '最后修改人邮箱前缀',
          `DataChange_LastTime` timestamp NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '最后修改时间',
          `IsDeleted` bit(1) DEFAULT b'0' COMMENT '软删除',
          PRIMARY KEY (`Id`),
          UNIQUE KEY `IX_NamespaceId` (`NamespaceId`),
          KEY `DataChange_LastTime` (`DataChange_LastTime`)
        ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='namespace的编辑锁';
        
        
        
        # Dump of table release
        # ------------------------------------------------------------
        
        DROP TABLE IF EXISTS `Release`;
        
        CREATE TABLE `Release` (
          `Id` int(10) unsigned NOT NULL AUTO_INCREMENT COMMENT '自增主键',
          `ReleaseKey` varchar(64) NOT NULL DEFAULT '' COMMENT '发布的Key',
          `Name` varchar(64) NOT NULL DEFAULT 'default' COMMENT '发布名字',
          `Comment` varchar(256) DEFAULT NULL COMMENT '发布说明',
          `AppId` varchar(500) NOT NULL DEFAULT 'default' COMMENT 'AppID',
          `ClusterName` varchar(500) NOT NULL DEFAULT 'default' COMMENT 'ClusterName',
          `NamespaceName` varchar(500) NOT NULL DEFAULT 'default' COMMENT 'namespaceName',
          `Configurations` longtext NOT NULL COMMENT '发布配置',
          `IsAbandoned` bit(1) NOT NULL DEFAULT b'0' COMMENT '是否废弃',
          `IsDeleted` bit(1) NOT NULL DEFAULT b'0' COMMENT '1: deleted, 0: normal',
          `DataChange_CreatedBy` varchar(32) NOT NULL DEFAULT 'default' COMMENT '创建人邮箱前缀',
          `DataChange_CreatedTime` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
          `DataChange_LastModifiedBy` varchar(32) DEFAULT '' COMMENT '最后修改人邮箱前缀',
          `DataChange_LastTime` timestamp NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '最后修改时间',
          PRIMARY KEY (`Id`),
          KEY `AppId_ClusterName_GroupName` (`AppId`(191),`ClusterName`(191),`NamespaceName`(191)),
          KEY `DataChange_LastTime` (`DataChange_LastTime`),
          KEY `IX_ReleaseKey` (`ReleaseKey`)
        ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='发布';
        
        
        # Dump of table releasehistory
        # ------------------------------------------------------------
        
        DROP TABLE IF EXISTS `ReleaseHistory`;
        
        CREATE TABLE `ReleaseHistory` (
          `Id` int(11) unsigned NOT NULL AUTO_INCREMENT COMMENT '自增Id',
          `AppId` varchar(32) NOT NULL DEFAULT 'default' COMMENT 'AppID',
          `ClusterName` varchar(32) NOT NULL DEFAULT 'default' COMMENT 'ClusterName',
          `NamespaceName` varchar(32) NOT NULL DEFAULT 'default' COMMENT 'namespaceName',
          `BranchName` varchar(32) NOT NULL DEFAULT 'default' COMMENT '发布分支名',
          `ReleaseId` int(11) unsigned NOT NULL DEFAULT '0' COMMENT '关联的Release Id',
          `PreviousReleaseId` int(11) unsigned NOT NULL DEFAULT '0' COMMENT '前一次发布的ReleaseId',
          `Operation` tinyint(3) unsigned NOT NULL DEFAULT '0' COMMENT '发布类型，0: 普通发布，1: 回滚，2: 灰度发布，3: 灰度规则更新，4: 灰度合并回主分支发布，5: 主分支发布灰度自动发布，6: 主分支回滚灰度自动发布，7: 放弃灰度',
          `OperationContext` longtext NOT NULL COMMENT '发布上下文信息',
          `IsDeleted` bit(1) NOT NULL DEFAULT b'0' COMMENT '1: deleted, 0: normal',
          `DataChange_CreatedBy` varchar(32) NOT NULL DEFAULT 'default' COMMENT '创建人邮箱前缀',
          `DataChange_CreatedTime` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
          `DataChange_LastModifiedBy` varchar(32) DEFAULT '' COMMENT '最后修改人邮箱前缀',
          `DataChange_LastTime` timestamp NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '最后修改时间',
          PRIMARY KEY (`Id`),
          KEY `IX_Namespace` (`AppId`,`ClusterName`,`NamespaceName`,`BranchName`),
          KEY `IX_ReleaseId` (`ReleaseId`),
          KEY `IX_DataChange_LastTime` (`DataChange_LastTime`)
        ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='发布历史';
        
        
        # Dump of table releasemessage
        # ------------------------------------------------------------
        
        DROP TABLE IF EXISTS `ReleaseMessage`;
        
        CREATE TABLE `ReleaseMessage` (
          `Id` int(11) unsigned NOT NULL AUTO_INCREMENT COMMENT '自增主键',
          `Message` varchar(1024) NOT NULL DEFAULT '' COMMENT '发布的消息内容',
          `DataChange_LastTime` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '最后修改时间',
          PRIMARY KEY (`Id`),
          KEY `DataChange_LastTime` (`DataChange_LastTime`),
          KEY `IX_Message` (`Message`(191))
        ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='发布消息';
        
        
        
        # Dump of table serverconfig
        # ------------------------------------------------------------
        
        DROP TABLE IF EXISTS `ServerConfig`;
        
        CREATE TABLE `ServerConfig` (
          `Id` int(10) unsigned NOT NULL AUTO_INCREMENT COMMENT '自增Id',
          `Key` varchar(64) NOT NULL DEFAULT 'default' COMMENT '配置项Key',
          `Cluster` varchar(32) NOT NULL DEFAULT 'default' COMMENT '配置对应的集群，default为不针对特定的集群',
          `Value` varchar(2048) NOT NULL DEFAULT 'default' COMMENT '配置项值',
          `Comment` varchar(1024) DEFAULT '' COMMENT '注释',
          `IsDeleted` bit(1) NOT NULL DEFAULT b'0' COMMENT '1: deleted, 0: normal',
          `DataChange_CreatedBy` varchar(32) NOT NULL DEFAULT 'default' COMMENT '创建人邮箱前缀',
          `DataChange_CreatedTime` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
          `DataChange_LastModifiedBy` varchar(32) DEFAULT '' COMMENT '最后修改人邮箱前缀',
          `DataChange_LastTime` timestamp NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '最后修改时间',
          PRIMARY KEY (`Id`),
          KEY `IX_Key` (`Key`),
          KEY `DataChange_LastTime` (`DataChange_LastTime`)
        ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='配置服务自身配置';
        
        # Config
        # ------------------------------------------------------------
        INSERT INTO `ServerConfig` (`Key`, `Cluster`, `Value`, `Comment`)
        VALUES
            ('eureka.service.url', 'default', 'http://localhost:8080/eureka/', 'Eureka服务Url，多个service以英文逗号分隔'),
            ('namespace.lock.switch', 'default', 'false', '一次发布只能有一个人修改开关'),
            ('item.key.length.limit', 'default', '128', 'item key 最大长度限制'),
            ('item.value.length.limit', 'default', '20000', 'item value最大长度限制'),
            ('config-service.cache.enabled', 'default', 'false', 'ConfigService是否开启缓存，开启后能提高性能，但是会增大内存消耗！');
        
        /*!40111 SET SQL_NOTES=@OLD_SQL_NOTES */;
        /*!40101 SET SQL_MODE=@OLD_SQL_MODE */;
        /*!40014 SET FOREIGN_KEY_CHECKS=@OLD_FOREIGN_KEY_CHECKS */;
        /*!40101 SET CHARACTER_SET_CLIENT=@OLD_CHARACTER_SET_CLIENT */;
        /*!40101 SET CHARACTER_SET_RESULTS=@OLD_CHARACTER_SET_RESULTS */;
        /*!40101 SET COLLATION_CONNECTION=@OLD_COLLATION_CONNECTION */;
  </code></pre>
</details>
***

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