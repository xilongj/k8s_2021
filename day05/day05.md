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
MariaDB [ApolloConfigDB]> grant INSERT,DELETE,UPDATE,SELECT on ApolloConfigDB.* to "apolloconfig"@"10.4.7.%" identified by "123456";
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
                    Value: http://config.od.com/eureka
                  Comment: Eureka服务Url，多个service以英文逗号分隔
                IsDeleted:
     DataChange_CreatedBy: default
   DataChange_CreatedTime: 2021-02-26 00:18:49
DataChange_LastModifiedBy:
      DataChange_LastTime: 2021-02-26 00:32:13
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
### Apollo Config Service
#### [hdss7-200]
```text
[root@hdss7-200 src]# wget https://github.com/ctripcorp/apollo/releases/download/v1.5.1/apollo-adminservice-1.5.1-github.zip
[root@hdss7-200 src]# wget https://github.com/ctripcorp/apollo/releases/download/v1.5.1/apollo-configservice-1.5.1-github.zip
[root@hdss7-200 src]# wget https://github.com/ctripcorp/apollo/releases/download/v1.5.1/apollo-portal-1.5.1-github.zip
```
```text
[root@hdss7-200 src]# mkdir -p /data/dockerfile/apollo-configservice
[root@hdss7-200 src]# unzip -o apollo-configservice-1.5.1-github.zip -d /data/dockerfile/apollo-configservice/
```
```text
[root@hdss7-200 src]# cd /data/dockerfile/apollo-configservice/
[root@hdss7-200 apollo-configservice]# mv apollo-configservice-1.5.1-sources.jar /tmp/
[root@hdss7-200 apollo-configservice]# ls -l
total 4
-rwxr-xr-x 1 root root 61991736 Nov  9  2019 apollo-configservice-1.5.1.jar
-rw-r--r-- 1 root root 57 Apr 20  2017 apollo-configservice.conf
drwxr-xr-x 2 root root 65 Feb 27 08:40 config
drwxr-xr-x 2 root root 43 Oct  1  2019 scripts
```
#### Update DNS
```text
[root@hdss7-11 ~]# cat /var/named/od.com.zone
$ORIGIN od.com.
$TTL 600        ; 10 minutes
@       IN SOA dns.od.com. dnsadmin.od.com. (
                        2021021011      ; serial
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
mysql           A       10.4.7.11
[root@hdss7-11 ~]# systemctl restart named

[root@hdss7-11 ~]# dig -t A mysql.od.com @10.4.7.11 +short
10.4.7.11
```
```text
[root@hdss7-200 apollo-configservice]# cd config/
[root@hdss7-200 config]# cat application-github.properties
# DataSource
spring.datasource.url = jdbc:mysql://mysql.od.com:3306/ApolloConfigDB?characterEncoding=utf8
spring.datasource.username = apolloconfig
spring.datasource.password = 123456


#apollo.eureka.server.enabled=true
#apollo.eureka.client.enabled=true
[root@hdss7-200 config]# pwd
/data/dockerfile/apollo-configservice/config
```
```text
[root@hdss7-200 scripts]# cat startup.sh
#!/bin/bash
SERVICE_NAME=apollo-configservice
## Adjust log dir if necessary
LOG_DIR=/opt/logs/apollo-config-server
## Adjust server port if necessary
SERVER_PORT=8080
APOLLO_CONFIG_SERVICE_NAME=$(hostname -i)
SERVER_URL="http://${APOLLO_CONFIG_SERVICE_NAME}:${SERVER_PORT}"

## Adjust memory settings if necessary
export JAVA_OPTS="-Xms128m -Xmx128m -Xss256k -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=384m -XX:NewSize=256m -XX:MaxNewSize=256m -XX:SurvivorRatio=8"

## Only uncomment the following when you are using server jvm
#export JAVA_OPTS="$JAVA_OPTS -server -XX:-ReduceInitialCardMarks"

########### The following is the same for configservice, adminservice, portal ###########
export JAVA_OPTS="$JAVA_OPTS -XX:ParallelGCThreads=4 -XX:MaxTenuringThreshold=9 -XX:+DisableExplicitGC -XX:+ScavengeBeforeFullGC -XX:SoftRefLRUPolicyMSPerMB=0 -XX:+ExplicitGCInvokesConcurrent -XX:+PrintGCDetails -XX:+HeapDumpOnOutOfMemoryError -XX:-OmitStackTraceInFastThrow -Duser.timezone=Asia/Shanghai -Dclient.encoding.override=UTF-8 -Dfile.encoding=UTF-8 -Djava.security.egd=file:/dev/./urandom"
export JAVA_OPTS="$JAVA_OPTS -Dserver.port=$SERVER_PORT -Dlogging.file=$LOG_DIR/$SERVICE_NAME.log -XX:HeapDumpPath=$LOG_DIR/HeapDumpOnOutOfMemoryError/"

# Find Java
if [[ -n "$JAVA_HOME" ]] && [[ -x "$JAVA_HOME/bin/java" ]]; then
    javaexe="$JAVA_HOME/bin/java"
elif type -p java > /dev/null 2>&1; then
    javaexe=$(type -p java)
elif [[ -x "/usr/bin/java" ]];  then
    javaexe="/usr/bin/java"
else
    echo "Unable to find Java"
    exit 1
fi

if [[ "$javaexe" ]]; then
    version=$("$javaexe" -version 2>&1 | awk -F '"' '/version/ {print $2}')
    version=$(echo "$version" | awk -F. '{printf("%03d%03d",$1,$2);}')
    # now version is of format 009003 (9.3.x)
    if [ $version -ge 011000 ]; then
        JAVA_OPTS="$JAVA_OPTS -Xlog:gc*:$LOG_DIR/gc.log:time,level,tags -Xlog:safepoint -Xlog:gc+heap=trace"
    elif [ $version -ge 010000 ]; then
        JAVA_OPTS="$JAVA_OPTS -Xlog:gc*:$LOG_DIR/gc.log:time,level,tags -Xlog:safepoint -Xlog:gc+heap=trace"
    elif [ $version -ge 009000 ]; then
        JAVA_OPTS="$JAVA_OPTS -Xlog:gc*:$LOG_DIR/gc.log:time,level,tags -Xlog:safepoint -Xlog:gc+heap=trace"
    else
        JAVA_OPTS="$JAVA_OPTS -XX:+UseParNewGC"
        JAVA_OPTS="$JAVA_OPTS -Xloggc:$LOG_DIR/gc.log -XX:+PrintGCDetails"
        JAVA_OPTS="$JAVA_OPTS -XX:+UseConcMarkSweepGC -XX:+UseCMSCompactAtFullCollection -XX:+UseCMSInitiatingOccupancyOnly -XX:CMSInitiatingOccupancyFraction=60 -XX:+CMSClassUnloadingEnabled -XX:+CMSParallelRemarkEnabled -XX:CMSFullGCsBeforeCompaction=9 -XX:+CMSClassUnloadingEnabled  -XX:+PrintGCDateStamps -XX:+PrintGCApplicationConcurrentTime -XX:+PrintHeapAtGC -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=5 -XX:GCLogFileSize=5M"
    fi
fi

printf "$(date) ==== Starting ==== \n"

cd `dirname $0`/..
chmod 755 $SERVICE_NAME".jar"
./$SERVICE_NAME".jar" start

rc=$?;

if [[ $rc != 0 ]];
then
    echo "$(date) Failed to start $SERVICE_NAME.jar, return code: $rc"
    exit $rc;
fi

tail -f /dev/null
```
```text
https://github.com/ctripcorp/apollo/blob/v1.5.1/scripts/apollo-on-kubernetes/apollo-config-server/scripts/startup-kubernetes.sh
```
```text
[root@hdss7-200 apollo-configservice]# pwd
/data/dockerfile/apollo-configservice
[root@hdss7-200 apollo-configservice]# cat Dockerfile
FROM stanleyws/jre8:8u112

ENV VERSION 1.5.1

RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime &&\
    echo "Asia/Shanghai" > /etc/timezone

ADD apollo-configservice-${VERSION}.jar /apollo-configservice/apollo-configservice.jar
ADD config/ /apollo-configservice/config
ADD scripts/ /apollo-configservice/scripts

CMD ["/apollo-configservice/scripts/startup.sh"]
```
#### Build image
```text
[root@hdss7-200 apollo-configservice]# docker build . -t harbor.od.com/infra/apollo-configservice:v1.5.1
Sending build context to Docker daemon     62MB
Step 1/7 : FROM stanleyws/jre8:8u112
 ---> fa3a085d6ef1
Step 2/7 : ENV VERSION 1.5.1
 ---> Running in b132229f568b
Removing intermediate container b132229f568b
 ---> d4a281ac5a60
Step 3/7 : RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime &&    echo "Asia/Shanghai" > /etc/timezone
 ---> Running in 397b33be0862
Removing intermediate container 397b33be0862
 ---> 73938aba034b
Step 4/7 : ADD apollo-configservice-${VERSION}.jar /apollo-configservice/apollo-configservice.jar
 ---> 1db45bb21cad
Step 5/7 : ADD config/ /apollo-configservice/config
 ---> c878f5d01ab9
Step 6/7 : ADD scripts/ /apollo-configservice/scripts
 ---> 208583f39c15
Step 7/7 : CMD ["/apollo-configservice/scripts/startup.sh"]
 ---> Running in 327d60dd5489
Removing intermediate container 327d60dd5489
 ---> cd5099acae8f
Successfully built cd5099acae8f
Successfully tagged harbor.od.com/infra/apollo-configservice:v1.5.1

[root@hdss7-200 apollo-configservice]# docker push harbor.od.com/infra/apollo-configservice:v1.5.1
```
```text
[root@hdss7-200 ~]# mkdir -p /data/k8s-yaml/apollo-configservice

[root@hdss7-200 ~]# cd /data/k8s-yaml/apollo-configservice
```
```text
# deployment.yaml
[root@hdss7-200 apollo-configservice]# cat deployment.yaml
kind: Deployment
apiVersion: extensions/v1beta1
metadata:
  name: apollo-configservice
  namespace: infra
  labels:
    name: apollo-configservice
spec:
  replicas: 1
  selector:
    matchLabels:
      name: apollo-configservice
  template:
    metadata:
      labels:
        app: apollo-configservice
        name: apollo-configservice
    spec:
      volumes:
      - name: configmap-volume
        configMap:
          name: apollo-configservice-cm
      containers:
      - name: apollo-configservice
        image: harbor.od.com/infra/apollo-configservice:v1.5.1
        ports:
        - containerPort: 8080
          protocol: TCP
        volumeMounts:
        - name: configmap-volume
          mountPath: /apollo-configservice/config
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
        imagePullPolicy: IfNotPresent
      imagePullSecrets:
      - name: harbor
      restartPolicy: Always
      terminationGracePeriodSeconds: 30
      securityContext:
        runAsUser: 0
      schedulerName: default-scheduler
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
  revisionHistoryLimit: 7
  progressDeadlineSeconds: 600
```
```text
# cm.yaml
[root@hdss7-200 apollo-configservice]# cat cm.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: apollo-configservice-cm
  namespace: infra
data:
  application-github.properties: |
    # DataSource
    spring.datasource.url = jdbc:mysql://mysql.od.com:3306/ApolloConfigDB?characterEncoding=utf8
    spring.datasource.username = apolloconfig
    spring.datasource.password = 123456
    eureka.service.url = http://config.od.com/eureka
  app.properties: |
    appId=100003171
```
```text
# svc.yaml
[root@hdss7-200 apollo-configservice]# cat svc.yaml
kind: Service
apiVersion: v1
metadata:
  name: apollo-configservice
  namespace: infra
spec:
  ports:
  - protocol: TCP
    port: 8080
    targetPort: 8080
  selector:
    app: apollo-configservice
  clusterIP: None
  type: ClusterIP
  sessionAffinity: None
```
```text
# ingress.yaml
[root@hdss7-200 apollo-configservice]# cat ingress.yaml
kind: Ingress
apiVersion: extensions/v1beta1
metadata:
  name: apollo-configservice
  namespace: infra
spec:
  rules:
  - host: config.od.com
    http:
      paths:
      - path: /
        backend:
          serviceName: apollo-configservice
          servicePort: 8080
```
#### [hdss7-22]
```text
[root@hdss7-22 ~]# kubectl apply -f http://k8s-yaml.od.com/apollo-configservice/cm.yaml
configmap/apollo-configservice-cm created
[root@hdss7-22 ~]# kubectl apply -f http://k8s-yaml.od.com/apollo-configservice/deployment.yaml
deployment.extensions/apollo-configservice created
[root@hdss7-22 ~]# kubectl apply -f http://k8s-yaml.od.com/apollo-configservice/svc.yaml
service/apollo-configservice created
[root@hdss7-22 ~]# kubectl apply -f http://k8s-yaml.od.com/apollo-configservice/ingress.yaml
ingress.extensions/apollo-configservice created
```
```text
# update C:\Windows\System32\drivers\etc\hosts
10.4.7.10		dashboard.od.com  traefik.od.com  jenkins.od.com  dubbo-monitor.od.com  config.od.com
10.4.7.200		k8s-yaml.od.com   harbor.od.com
Chrome: config.od.com
```
### Apollo admin service
#### [hdss7-200]
```text
[root@hdss7-200 src]# wget https://github.com/ctripcorp/apollo/releases/download/v1.5.1/apollo-adminservice-1.5.1-github.zip
```
```text
[root@hdss7-200 ~]# cd /opt/src/
[root@hdss7-200 src]# mkdir -p /data/dockerfile/apollo-adminservice
[root@hdss7-200 src]# unzip -o apollo-adminservice-1.5.1-github.zip -d /data/dockerfile/apollo-adminservice/
```
```buildoutcfg
[root@hdss7-200 src]# cd /data/dockerfile/apollo-adminservice/
[root@hdss7-200 apollo-adminservice]# ls -l
total 57024
-rwxr-xr-x 1 root root 58358738 Nov  9  2019 apollo-adminservice-1.5.1.jar
-rwxr-xr-x 1 root root    25991 Nov  9  2019 apollo-adminservice-1.5.1-sources.jar
-rw-r--r-- 1 root root       57 Apr 20  2017 apollo-adminservice.conf
drwxr-xr-x 2 root root       65 Mar  1 23:31 config
drwxr-xr-x 2 root root       43 Oct  1  2019 scripts
[root@hdss7-200 apollo-adminservice]# mv apollo-adminservice-1.5.1-sources.jar /tmp/
```
```buildoutcfg
[root@hdss7-200 apollo-adminservice]# cd scripts/
[root@hdss7-200 scripts]# mv shutdown.sh /tmp/
```
```buildoutcfg
# startup.sh
[root@hdss7-200 scripts]# cat startup.sh
#!/bin/bash
SERVICE_NAME=apollo-adminservice
## Adjust log dir if necessary
LOG_DIR=/opt/logs/apollo-adminservice
## Adjust server port if necessary
SERVER_PORT=8080
APOLLO_ADMIN_SERVICE_NAME=$(hostname -i)
# SERVER_URL="http://localhost:${SERVER_PORT}"
SERVER_URL="http://${APOLLO_ADMIN_SERVICE_NAME}:${SERVER_PORT}"

## Adjust memory settings if necessary
#export JAVA_OPTS="-Xms2560m -Xmx2560m -Xss256k -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=384m -XX:NewSize=1536m -XX:MaxNewSize=1536m -XX:SurvivorRatio=8"

## Only uncomment the following when you are using server jvm
#export JAVA_OPTS="$JAVA_OPTS -server -XX:-ReduceInitialCardMarks"

########### The following is the same for configservice, adminservice, portal ###########
export JAVA_OPTS="$JAVA_OPTS -XX:ParallelGCThreads=4 -XX:MaxTenuringThreshold=9 -XX:+DisableExplicitGC -XX:+ScavengeBeforeFullGC -XX:SoftRefLRUPolicyMSPerMB=0 -XX:+ExplicitGCInvokesConcurrent -XX:+PrintGCDetails -XX:+HeapDumpOnOutOfMemoryError -XX:-OmitStackTraceInFastThrow -Duser.timezone=Asia/Shanghai -Dclient.encoding.override=UTF-8 -Dfile.encoding=UTF-8 -Djava.security.egd=file:/dev/./urandom"
export JAVA_OPTS="$JAVA_OPTS -Dserver.port=$SERVER_PORT -Dlogging.file=$LOG_DIR/$SERVICE_NAME.log -XX:HeapDumpPath=$LOG_DIR/HeapDumpOnOutOfMemoryError/"

# Find Java
if [[ -n "$JAVA_HOME" ]] && [[ -x "$JAVA_HOME/bin/java" ]]; then
    javaexe="$JAVA_HOME/bin/java"
elif type -p java > /dev/null 2>&1; then
    javaexe=$(type -p java)
elif [[ -x "/usr/bin/java" ]];  then
    javaexe="/usr/bin/java"
else
    echo "Unable to find Java"
    exit 1
fi

if [[ "$javaexe" ]]; then
    version=$("$javaexe" -version 2>&1 | awk -F '"' '/version/ {print $2}')
    version=$(echo "$version" | awk -F. '{printf("%03d%03d",$1,$2);}')
    # now version is of format 009003 (9.3.x)
    if [ $version -ge 011000 ]; then
        JAVA_OPTS="$JAVA_OPTS -Xlog:gc*:$LOG_DIR/gc.log:time,level,tags -Xlog:safepoint -Xlog:gc+heap=trace"
    elif [ $version -ge 010000 ]; then
        JAVA_OPTS="$JAVA_OPTS -Xlog:gc*:$LOG_DIR/gc.log:time,level,tags -Xlog:safepoint -Xlog:gc+heap=trace"
    elif [ $version -ge 009000 ]; then
        JAVA_OPTS="$JAVA_OPTS -Xlog:gc*:$LOG_DIR/gc.log:time,level,tags -Xlog:safepoint -Xlog:gc+heap=trace"
    else
        JAVA_OPTS="$JAVA_OPTS -XX:+UseParNewGC"
        JAVA_OPTS="$JAVA_OPTS -Xloggc:$LOG_DIR/gc.log -XX:+PrintGCDetails"
        JAVA_OPTS="$JAVA_OPTS -XX:+UseConcMarkSweepGC -XX:+UseCMSCompactAtFullCollection -XX:+UseCMSInitiatingOccupancyOnly -XX:CMSInitiatingOccupancyFraction=60 -XX:+CMSClassUnloadingEnabled -XX:+CMSParallelRemarkEnabled -XX:CMSFullGCsBeforeCompaction=9 -XX:+CMSClassUnloadingEnabled  -XX:+PrintGCDateStamps -XX:+PrintGCApplicationConcurrentTime -XX:+PrintHeapAtGC -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=5 -XX:GCLogFileSize=5M"
    fi
fi

printf "$(date) ==== Starting ==== \n"

cd `dirname $0`/..
chmod 755 $SERVICE_NAME".jar"
./$SERVICE_NAME".jar" start

rc=$?;

if [[ $rc != 0 ]];
then
    echo "$(date) Failed to start $SERVICE_NAME.jar, return code: $rc"
    exit $rc;
fi

tail -f /dev/null
```
```buildoutcfg
# Dockerfile
[root@hdss7-200 ~]# cat /data/dockerfile/apollo-adminservice/Dockerfile
FROM stanleyws/jre8:8u112

ENV VERSION 1.5.1

RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime &&\
    echo "Asia/Shanghai" > /etc/timezone

ADD apollo-adminservice-${VERSION}.jar /apollo-adminservice/apollo-adminservice.jar
ADD config/ /apollo-adminservice/config
ADD scripts/ /apollo-adminservice/scripts

CMD ["/apollo-adminservice/scripts/startup.sh"]
```
```buildoutcfg
# docker build
[root@hdss7-200 apollo-adminservice]# ls -l
total 57000
-rwxr-xr-x 1 root root 58358738 Nov  9  2019 apollo-adminservice-1.5.1.jar
-rw-r--r-- 1 root root       57 Apr 20  2017 apollo-adminservice.conf
drwxr-xr-x 2 root root       65 Mar  1 23:31 config
-rw-r--r-- 1 root root      367 Mar  1 23:40 Dockerfile
drwxr-xr-x 2 root root       24 Mar  1 23:37 scripts
[root@hdss7-200 apollo-adminservice]# docker build . -t harbor.od.com/infra/apollo-adminservice:v1.5.1
Sending build context to Docker daemon  58.37MB
Step 1/7 : FROM stanleyws/jre8:8u112
 ---> fa3a085d6ef1
Step 2/7 : ENV VERSION 1.5.1
 ---> Using cache
 ---> e9b6b9ea4eae
Step 3/7 : RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime &&    echo "Asia/Shanghai" > /etc/timezone
 ---> Using cache
 ---> 68387c1aa0f1
Step 4/7 : ADD apollo-adminservice-${VERSION}.jar /apollo-adminservice/apollo-adminservice.jar
 ---> d89863e77754
Step 5/7 : ADD config/ /apollo-adminservice/config
 ---> d95cc949a9b2
Step 6/7 : ADD scripts/ /apollo-adminservice/scripts
 ---> 69cde9b5d21a
Step 7/7 : CMD ["/apollo-adminservice/scripts/startup.sh"]
 ---> Running in 63609ea1a4e2
Removing intermediate container 63609ea1a4e2
 ---> 3ddfc104a8ec
Successfully built 3ddfc104a8ec
Successfully tagged harbor.od.com/infra/apollo-adminservice:v1.5.1
```
```buildoutcfg
# docker push
[root@hdss7-200 apollo-adminservice]# docker push harbor.od.com/infra/apollo-adminservice:v1.5.1
The push refers to repository [harbor.od.com/infra/apollo-adminservice]
da824f9d4950: Pushed
eb3583aaba45: Pushed
552900b22ce7: Pushed
dfc28ad7718f: Mounted from infra/apollo-configservice
0690f10a63a5: Mounted from infra/apollo-configservice
c843b2cf4e12: Mounted from infra/apollo-configservice
fddd8887b725: Mounted from infra/apollo-configservice
42052a19230c: Mounted from infra/apollo-configservice
8d4d1ab5ff74: Mounted from infra/apollo-configservice
v1.5.1: digest: sha256:05ad0b29b2798b6b5b9b05c229288f9b906a47edb0f5a26641e5aa7e02a264d9 size: 2201
```
```buildoutcfg
# cm.yaml
[root@hdss7-200 ~]# mkdir -p /data/k8s-yaml/apollo-adminservice
[root@hdss7-200 ~]# cd !$
cd /data/k8s-yaml/apollo-adminservice

[root@hdss7-200 apollo-adminservice]# cat cm.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: apollo-adminservice-cm
  namespace: infra
data:
  application-github.properties: |
    # DataSource
    spring.datasource.url = jdbc:mysql://mysql.od.com:3306/ApolloConfigDB?characterEncoding=utf8
    spring.datasource.username = apolloconfig
    spring.datasource.password = 123456
    eureka.service.url = http://config.od.com/eureka
  app.properties: |
    appId=100003172
```
```buildoutcfg
# deployment.yaml
[root@hdss7-200 apollo-adminservice]# cat deployment.yaml
kind: Deployment
apiVersion: extensions/v1beta1
metadata:
  name: apollo-adminservice
  namespace: infra
  labels:
    name: apollo-adminservice
spec:
  replicas: 1
  selector:
    matchLabels:
      name: apollo-adminservice
  template:
    metadata:
      labels:
        app: apollo-adminservice
        name: apollo-adminservice
    spec:
      volumes:
      - name: configmap-volume
        configMap:
          name: apollo-adminservice-cm
      containers:
      - name: apollo-adminservice
        image: harbor.od.com/infra/apollo-adminservice:v1.5.1
        ports:
        - containerPort: 8080
          protocol: TCP
        volumeMounts:
        - name: configmap-volume
          mountPath: /apollo-adminservice/config
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
        imagePullPolicy: IfNotPresent
      imagePullSecrets:
      - name: harbor
      restartPolicy: Always
      terminationGracePeriodSeconds: 30
      securityContext:
        runAsUser: 0
      schedulerName: default-scheduler
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
  revisionHistoryLimit: 7
  progressDeadlineSeconds: 600
```
#### [hdss7-22]
```buildoutcfg
[root@hdss7-22 ~]# kubectl apply -f http://k8s-yaml.od.com/apollo-adminservice/cm.yaml
configmap/apollo-adminservice-cm created
[root@hdss7-22 ~]# kubectl apply -f http://k8s-yaml.od.com/apollo-adminservice/deployment.yaml
deployment.extensions/apollo-adminservice created

[root@hdss7-22 ~]# kubectl get pods -o wide -n infra
NAME                                   READY   STATUS    RESTARTS   AGE   IP            NODE                NOMINATED NODE   READINESS GATES
apollo-adminservice-5cccf97c64-qdlwz   1/1     Running   0          16s   172.7.21.12   hdss7-21.host.com   <none>           <none>
apollo-configservice-5f6555448-6xbwj   1/1     Running   0          24h   172.7.21.10   hdss7-21.host.com   <none>           <none>
dubbo-monitor-6676dd74cc-r559b         1/1     Running   0          24h   172.7.22.20   hdss7-22.host.com   <none>           <none>
jenkins-56cfb9c479-dfh6w               1/1     Running   0          24h   172.7.21.17   hdss7-21.host.com   <none>           <none>
```
```buildoutcfg
[root@hdss7-22 ~]# kubectl get pods -o wide -n infra
NAME                                   READY   STATUS    RESTARTS   AGE   IP            NODE                NOMINATED NODE   READINESS GATES
apollo-adminservice-5cccf97c64-k8fhx   1/1     Running   0          64s   172.7.22.17   hdss7-22.host.com   <none>           <none>
apollo-configservice-5f6555448-6xbwj   1/1     Running   0          24h   172.7.21.10   hdss7-21.host.com   <none>           <none>
dubbo-monitor-6676dd74cc-r559b         1/1     Running   0          24h   172.7.22.20   hdss7-22.host.com   <none>           <none>
jenkins-56cfb9c479-dfh6w               1/1     Running   0          24h   172.7.21.17   hdss7-21.host.com   <none>           <none>
# hdss7-11
MariaDB [(none)]> show processlist;
+-----+--------------+-----------------+----------------+---------+------+-------+------------------+----------+
| Id  | User         | Host            | db             | Command | Time | State | Info             | Progress |
+-----+--------------+-----------------+----------------+---------+------+-------+------------------+----------+
| 548 | apolloconfig | 10.4.7.21:33148 | ApolloConfigDB | Sleep   |    0 |       | NULL             |    0.000 |
| 549 | apolloconfig | 10.4.7.21:33246 | ApolloConfigDB | Sleep   |    0 |       | NULL             |    0.000 |
| 550 | apolloconfig | 10.4.7.21:33270 | ApolloConfigDB | Sleep   |    0 |       | NULL             |    0.000 |
| 551 | apolloconfig | 10.4.7.21:33296 | ApolloConfigDB | Sleep   | 1471 |       | NULL             |    0.000 |
| 552 | apolloconfig | 10.4.7.21:33306 | ApolloConfigDB | Sleep   | 1461 |       | NULL             |    0.000 |
| 553 | apolloconfig | 10.4.7.21:33312 | ApolloConfigDB | Sleep   | 1459 |       | NULL             |    0.000 |
| 554 | apolloconfig | 10.4.7.21:33374 | ApolloConfigDB | Sleep   | 1397 |       | NULL             |    0.000 |
| 555 | apolloconfig | 10.4.7.21:33952 | ApolloConfigDB | Sleep   |  821 |       | NULL             |    0.000 |
| 556 | apolloconfig | 10.4.7.21:33966 | ApolloConfigDB | Sleep   |  809 |       | NULL             |    0.000 |
| 557 | apolloconfig | 10.4.7.21:34016 | ApolloConfigDB | Sleep   |  762 |       | NULL             |    0.000 |
| 568 | apolloconfig | 10.4.7.22:58742 | ApolloConfigDB | Sleep   |   10 |       | NULL             |    0.000 |
| 569 | apolloconfig | 10.4.7.22:58748 | ApolloConfigDB | Sleep   |   24 |       | NULL             |    0.000 |
| 570 | apolloconfig | 10.4.7.22:58750 | ApolloConfigDB | Sleep   |   24 |       | NULL             |    0.000 |
| 571 | apolloconfig | 10.4.7.22:58752 | ApolloConfigDB | Sleep   |   24 |       | NULL             |    0.000 |
| 572 | apolloconfig | 10.4.7.22:58754 | ApolloConfigDB | Sleep   |   24 |       | NULL             |    0.000 |
| 573 | apolloconfig | 10.4.7.22:58756 | ApolloConfigDB | Sleep   |   24 |       | NULL             |    0.000 |
| 574 | apolloconfig | 10.4.7.22:58758 | ApolloConfigDB | Sleep   |   24 |       | NULL             |    0.000 |
| 575 | apolloconfig | 10.4.7.22:58760 | ApolloConfigDB | Sleep   |   24 |       | NULL             |    0.000 |
| 576 | apolloconfig | 10.4.7.22:58762 | ApolloConfigDB | Sleep   |   24 |       | NULL             |    0.000 |
| 577 | apolloconfig | 10.4.7.22:58764 | ApolloConfigDB | Sleep   |   24 |       | NULL             |    0.000 |
| 578 | root         | localhost       | NULL           | Query   |    0 | init  | show processlist |    0.000 |
+-----+--------------+-----------------+----------------+---------+------+-------+------------------+----------+
21 rows in set (0.00 sec)
```
```buildoutcfg
# health check
[root@hdss7-22 ~]# curl 172.7.22.17:8080/info
{"git":{"commit":{"time":{"seconds":1573275854,"nanos":0},"id":"c9eae54"},"branch":"1.5.1"}}
```
### Apollo Portal
#### [hdss7-200]
```buildoutcfg
# download apollo portal
[root@hdss7-200 src]# wget https://github.com/ctripcorp/apollo/releases/download/v1.5.1/apollo-portal-1.5.1-github.zip
```
```buildoutcfg
[root@hdss7-200 ~]# cd /opt/src/
[root@hdss7-200 src]# mkdir -p /data/dockerfile/apollo-portal

[root@hdss7-200 src]# unzip -o apollo-portal-1.5.1-github.zip -d !$

[root@hdss7-200 src]# ls -l /data/dockerfile/apollo-portal/
total 42512
-rwxr-xr-x 1 root root 42342196 Nov  9  2019 apollo-portal-1.5.1.jar
-rwxr-xr-x 1 root root  1183429 Nov  9  2019 apollo-portal-1.5.1-sources.jar
-rw-r--r-- 1 root root       57 Apr 20  2017 apollo-portal.conf
drwxr-xr-x 2 root root       94 Mar  2 00:16 config
drwxr-xr-x 2 root root       43 Oct  1  2019 scripts
```
```buildoutcfg
[root@hdss7-200 src]# cd /data/dockerfile/apollo-portal/
[root@hdss7-200 apollo-portal]# mv apollo-portal-1.5.1-sources.jar /tmp/
[root@hdss7-200 apollo-portal]# mv apollo-portal.conf /tmp/
[root@hdss7-200 apollo-portal]# mv scripts/shutdown.sh /tmp/
```
### Create Portal DB
#### [hdss7-11]
```buildoutcfg
URL= https://github.com/ctripcorp/apollo/tree/1.5.1/scripts/db/migration/portaldb
[root@hdss7-11 ~]# wget URL -O apolloportal.sql
```
```buildoutcfg
[root@hdss7-11 ~]# ls -l
total 40
-rw-------. 1 root root  1633 Feb 10 11:43 anaconda-ks.cfg
-rw-r--r--  1 root root 20194 Feb 26 00:11 apolloconfig.sql
-rw-r--r--  1 root root 16205 Mar  2 00:26 apolloportal.sql
[root@hdss7-11 ~]# mysql -uroot -p
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 660
Server version: 10.1.48-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> source ./apolloportal.sql
Query OK, 0 rows affected (0.00 sec)
```
```buildoutcfg
MariaDB [(none)]> grant INSERT,DELETE,UPDATE,SELECT on ApolloPortalDB.* to "apolloportal"@"10.4.7.%" identified by "123456";
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> show grants for "apolloportal"@"10.4.7.%";
+--------------------------------------------------------------------------------------------------------------------+
| Grants for apolloportal@10.4.7.%                                                                                   |
+--------------------------------------------------------------------------------------------------------------------+
| GRANT USAGE ON *.* TO 'apolloportal'@'10.4.7.%' IDENTIFIED BY PASSWORD '*6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9' |
| GRANT SELECT, INSERT, UPDATE, DELETE ON `ApolloPortalDB`.* TO 'apolloportal'@'10.4.7.%'                            |
+--------------------------------------------------------------------------------------------------------------------+
2 rows in set (0.00 sec)
```
```buildoutcfg
MariaDB [(none)]> select user,host from mysql.user;
+--------------+-------------------+
| user         | host              |
+--------------+-------------------+
| apolloconfig | 10.4.7.%          |
| apolloportal | 10.4.7.%          |
| root         | 127.0.0.1         |
| root         | ::1               |
|              | hdss7-11.host.com |
| root         | hdss7-11.host.com |
|              | localhost         |
| root         | localhost         |
+--------------+-------------------+
8 rows in set (0.00 sec)
```
```buildoutcfg
# update table
MariaDB [ApolloPortalDB]> update ServerConfig set Value='[{"orgId":"dts01","orgName":"Azure Cloud"},{"orgId":"dts02","orgName":"AWS Cloud"},{"orgId":"dts03","orgName":"Ali Cloud"}]' where id=2;
Query OK, 0 rows affected (0.00 sec)
Rows matched: 1  Changed: 0  Warnings: 0

MariaDB [ApolloPortalDB]> select * from ServerConfig \G
*************************** 1. row ***************************
                       Id: 1
                      Key: apollo.portal.envs
                    Value: dev
                  Comment: 可支持的环境列表
                IsDeleted:
     DataChange_CreatedBy: default
   DataChange_CreatedTime: 2021-03-02 00:30:48
DataChange_LastModifiedBy:
      DataChange_LastTime: 2021-03-02 00:30:48
*************************** 2. row ***************************
                       Id: 2
                      Key: organizations
                    Value: [{"orgId":"dts01","orgName":"Azure Cloud"},{"orgId":"dts02","orgName":"AWS Cloud"},{"orgId":"dts03","orgName":"Ali Cloud"}]
                  Comment: 部门列表
                IsDeleted:
     DataChange_CreatedBy: default
   DataChange_CreatedTime: 2021-03-02 00:30:48
DataChange_LastModifiedBy:
      DataChange_LastTime: 2021-03-02 00:55:15
*************************** 3. row ***************************
                       Id: 3
                      Key: superAdmin
                    Value: apollo
                  Comment: Portal超级管理员
                IsDeleted:
     DataChange_CreatedBy: default
   DataChange_CreatedTime: 2021-03-02 00:30:48
DataChange_LastModifiedBy:
      DataChange_LastTime: 2021-03-02 00:30:48
*************************** 4. row ***************************
                       Id: 4
                      Key: api.readTimeout
                    Value: 10000
                  Comment: http接口read timeout
                IsDeleted:
     DataChange_CreatedBy: default
   DataChange_CreatedTime: 2021-03-02 00:30:48
DataChange_LastModifiedBy:
      DataChange_LastTime: 2021-03-02 00:30:48
*************************** 5. row ***************************
                       Id: 5
                      Key: consumer.token.salt
                    Value: someSalt
                  Comment: consumer token salt
                IsDeleted:
     DataChange_CreatedBy: default
   DataChange_CreatedTime: 2021-03-02 00:30:48
DataChange_LastModifiedBy:
      DataChange_LastTime: 2021-03-02 00:30:48
*************************** 6. row ***************************
                       Id: 6
                      Key: admin.createPrivateNamespace.switch
                    Value: true
                  Comment: 是否允许项目管理员创建私有namespace
                IsDeleted:
     DataChange_CreatedBy: default
   DataChange_CreatedTime: 2021-03-02 00:30:48
DataChange_LastModifiedBy:
      DataChange_LastTime: 2021-03-02 00:30:48
*************************** 7. row ***************************
                       Id: 7
                      Key: configView.memberOnly.envs
                    Value: pro
                  Comment: 只对项目成员显示配置信息的环境列表，多个env以英文逗号分隔
                IsDeleted:
     DataChange_CreatedBy: default
   DataChange_CreatedTime: 2021-03-02 00:30:48
DataChange_LastModifiedBy:
      DataChange_LastTime: 2021-03-02 00:30:48
7 rows in set (0.00 sec)
```
```buildoutcfg
# startup.sh
[root@hdss7-200 scripts]# pwd
/data/dockerfile/apollo-portal/scripts
[root@hdss7-200 scripts]# cat startup.sh
#!/bin/bash
SERVICE_NAME=apollo-portal
## Adjust log dir if necessary
LOG_DIR=/opt/logs/apollo-portal-server
## Adjust server port if necessary
SERVER_PORT=8080
APOLLO_PORTAL_SERVICE_NAME=$(hostname -i)
# SERVER_URL="http://localhost:$SERVER_PORT"
SERVER_URL="http://${APOLLO_PORTAL_SERVICE_NAME}:${SERVER_PORT}"

## Adjust memory settings if necessary
#export JAVA_OPTS="-Xms2560m -Xmx2560m -Xss256k -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=384m -XX:NewSize=1536m -XX:MaxNewSize=1536m -XX:SurvivorRatio=8"

## Only uncomment the following when you are using server jvm
#export JAVA_OPTS="$JAVA_OPTS -server -XX:-ReduceInitialCardMarks"

########### The following is the same for configservice, adminservice, portal ###########
export JAVA_OPTS="$JAVA_OPTS -XX:ParallelGCThreads=4 -XX:MaxTenuringThreshold=9 -XX:+DisableExplicitGC -XX:+ScavengeBeforeFullGC -XX:SoftRefLRUPolicyMSPerMB=0 -XX:+ExplicitGCInvokesConcurrent -XX:+PrintGCDetails -XX:+HeapDumpOnOutOfMemoryError -XX:-OmitStackTraceInFastThrow -Duser.timezone=Asia/Shanghai -Dclient.encoding.override=UTF-8 -Dfile.encoding=UTF-8 -Djava.security.egd=file:/dev/./urandom"
export JAVA_OPTS="$JAVA_OPTS -Dserver.port=$SERVER_PORT -Dlogging.file=$LOG_DIR/$SERVICE_NAME.log -XX:HeapDumpPath=$LOG_DIR/HeapDumpOnOutOfMemoryError/"

# Find Java
if [[ -n "$JAVA_HOME" ]] && [[ -x "$JAVA_HOME/bin/java" ]]; then
    javaexe="$JAVA_HOME/bin/java"
elif type -p java > /dev/null 2>&1; then
    javaexe=$(type -p java)
elif [[ -x "/usr/bin/java" ]];  then
    javaexe="/usr/bin/java"
else
    echo "Unable to find Java"
    exit 1
fi

if [[ "$javaexe" ]]; then
    version=$("$javaexe" -version 2>&1 | awk -F '"' '/version/ {print $2}')
    version=$(echo "$version" | awk -F. '{printf("%03d%03d",$1,$2);}')
    # now version is of format 009003 (9.3.x)
    if [ $version -ge 011000 ]; then
        JAVA_OPTS="$JAVA_OPTS -Xlog:gc*:$LOG_DIR/gc.log:time,level,tags -Xlog:safepoint -Xlog:gc+heap=trace"
    elif [ $version -ge 010000 ]; then
        JAVA_OPTS="$JAVA_OPTS -Xlog:gc*:$LOG_DIR/gc.log:time,level,tags -Xlog:safepoint -Xlog:gc+heap=trace"
    elif [ $version -ge 009000 ]; then
        JAVA_OPTS="$JAVA_OPTS -Xlog:gc*:$LOG_DIR/gc.log:time,level,tags -Xlog:safepoint -Xlog:gc+heap=trace"
    else
        JAVA_OPTS="$JAVA_OPTS -XX:+UseParNewGC"
        JAVA_OPTS="$JAVA_OPTS -Xloggc:$LOG_DIR/gc.log -XX:+PrintGCDetails"
        JAVA_OPTS="$JAVA_OPTS -XX:+UseConcMarkSweepGC -XX:+UseCMSCompactAtFullCollection -XX:+UseCMSInitiatingOccupancyOnly -XX:CMSInitiatingOccupancyFraction=60 -XX:+CMSClassUnloadingEnabled -XX:+CMSParallelRemarkEnabled -XX:CMSFullGCsBeforeCompaction=9 -XX:+CMSClassUnloadingEnabled  -XX:+PrintGCDateStamps -XX:+PrintGCApplicationConcurrentTime -XX:+PrintHeapAtGC -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=5 -XX:GCLogFileSize=5M"
    fi
fi

printf "$(date) ==== Starting ==== \n"

cd `dirname $0`/..
chmod 755 $SERVICE_NAME".jar"
./$SERVICE_NAME".jar" start

rc=$?;

if [[ $rc != 0 ]];
then
    echo "$(date) Failed to start $SERVICE_NAME.jar, return code: $rc"
    exit $rc;
fi

tail -f /dev/null
```
```buildoutcfg
# Dockerfile
[root@hdss7-200 apollo-portal]# cat Dockerfile
FROM stanleyws/jre8:8u112

ENV VERSION 1.5.1

RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime &&\
    echo "Asia/Shanghai" > /etc/timezone

ADD apollo-portal-${VERSION}.jar /apollo-portal/apollo-portal.jar
ADD config/ /apollo-portal/config
ADD scripts/ /apollo-portal/scripts

CMD ["/apollo-portal/scripts/startup.sh"]
```
```buildoutcfg
[root@hdss7-200 apollo-portal]# docker build . -t harbor.od.com/infra/apollo-portal:v1.5.1
Sending build context to Docker daemon  42.35MB
Step 1/7 : FROM stanleyws/jre8:8u112
 ---> fa3a085d6ef1
Step 2/7 : ENV VERSION 1.5.1
 ---> Using cache
 ---> e9b6b9ea4eae
Step 3/7 : RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime &&    echo "Asia/Shanghai" > /etc/timezone
 ---> Using cache
 ---> 68387c1aa0f1
Step 4/7 : ADD apollo-portal-${VERSION}.jar /apollo-portal/apollo-portal.jar
 ---> 297a0caf753f
Step 5/7 : ADD config/ /apollo-portal/config
 ---> e45f2f2cde8b
Step 6/7 : ADD scripts/ /apollo-portal/scripts
 ---> db92463368a9
Step 7/7 : CMD ["/apollo-portal/scripts/startup.sh"]
 ---> Running in 6deac31011b6
Removing intermediate container 6deac31011b6
 ---> 4abfcc37c107
Successfully built 4abfcc37c107
Successfully tagged harbor.od.com/infra/apollo-portal:v1.5.1
```
```buildoutcfg
[root@hdss7-200 apollo-portal]# docker push harbor.od.com/infra/apollo-portal:v1.5.1
The push refers to repository [harbor.od.com/infra/apollo-portal]
0be6e1135b51: Pushed
4a9f3af0cf28: Pushed
3f6f6ac5c214: Pushed
dfc28ad7718f: Mounted from infra/apollo-adminservice
0690f10a63a5: Mounted from infra/apollo-adminservice
c843b2cf4e12: Mounted from infra/apollo-adminservice
fddd8887b725: Mounted from infra/apollo-adminservice
42052a19230c: Mounted from infra/apollo-adminservice
8d4d1ab5ff74: Mounted from infra/apollo-adminservice
v1.5.1: digest: sha256:e2f192050bf791727dbad1dfe59e65cc8d0f90cbc94dca443ea690a33cceafc7 size: 2201
```
```buildoutcfg
# cm.yaml
[root@hdss7-200 ~]# mkdir -p /data/k8s-yaml/apollo-portal
[root@hdss7-200 ~]# cd /data/k8s-yaml/apollo-portal
[root@hdss7-200 apollo-portal]# cat cm.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: apollo-portal-cm
  namespace: infra
data:
  application-github.properties: |
    # DataSource
    spring.datasource.url = jdbc:mysql://mysql.od.com:3306/ApolloPortalDB?characterEncoding=utf8
    spring.datasource.username = apolloportal
    spring.datasource.password = 123456
  app.properties: |
    appId=100003173
  apollo-env.properties: |
    dev.meta=http://config.od.com
```
```buildoutcfg
# deployment.yaml
[root@hdss7-200 apollo-portal]# cat deployment.yaml
kind: Deployment
apiVersion: extensions/v1beta1
metadata:
  name: apollo-portal
  namespace: infra
  labels:
    name: apollo-portal
spec:
  replicas: 1
  selector:
    matchLabels:
      name: apollo-portal
  template:
    metadata:
      labels:
        app: apollo-portal
        name: apollo-portal
    spec:
      volumes:
      - name: configmap-volume
        configMap:
          name: apollo-portal-cm
      containers:
      - name: apollo-portal
        image: harbor.od.com/infra/apollo-portal:v1.5.1
        ports:
        - containerPort: 8080
          protocol: TCP
        volumeMounts:
        - name: configmap-volume
          mountPath: /apollo-portal/config
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
        imagePullPolicy: IfNotPresent
      imagePullSecrets:
      - name: harbor
      restartPolicy: Always
      terminationGracePeriodSeconds: 30
      securityContext:
        runAsUser: 0
      schedulerName: default-scheduler
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
  revisionHistoryLimit: 7
  progressDeadlineSeconds: 600
```
```buildoutcfg
# svc.yaml
[root@hdss7-200 apollo-portal]# cat svc.yaml
kind: Service
apiVersion: v1
metadata:
  name: apollo-portal
  namespace: infra
spec:
  ports:
  - protocol: TCP
    port: 8080
    targetPort: 8080
  selector:
    app: apollo-portal
  clusterIP: None
  type: ClusterIP
  sessionAffinity: None
```
```buildoutcfg
# ingress.yaml
[root@hdss7-200 apollo-portal]# cat ingress.yaml
kind: Ingress
apiVersion: extensions/v1beta1
metadata:
  name: apollo-portal
  namespace: infra
spec:
  rules:
  - host: portal.od.com
    http:
      paths:
      - path: /
        backend:
          serviceName: apollo-portal
          servicePort: 8080
```
### Update DNS
#### [hdss7-11]
```buildoutcfg
[root@hdss7-11 ~]# cat /var/named/od.com.zone
$ORIGIN od.com.
$TTL 600        ; 10 minutes
@       IN SOA dns.od.com. dnsadmin.od.com. (
                        2021021012      ; serial
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
mysql           A       10.4.7.11
portal          A       10.4.7.10
[root@hdss7-11 ~]# systemctl restart named

[root@hdss7-11 ~]# dig -t A portal.od.com @10.4.7.11 +short
10.4.7.10
```
#### [hdss7-22]
```buildoutcfg
[root@hdss7-22 ~]# kubectl apply -f http://k8s-yaml.od.com/apollo-portal/cm.yaml
configmap/apollo-portal-cm created
[root@hdss7-22 ~]# kubectl apply -f http://k8s-yaml.od.com/apollo-portal/deployment.yaml
deployment.extensions/apollo-portal created
[root@hdss7-22 ~]# kubectl apply -f http://k8s-yaml.od.com/apollo-portal/svc.yaml
service/apollo-portal created
[root@hdss7-22 ~]# kubectl apply -f http://k8s-yaml.od.com/apollo-portal/ingress.yaml
ingress.extensions/apollo-portal created

[root@hdss7-22 ~]# kubectl get pods -o wide -n infra
NAME                                   READY   STATUS    RESTARTS   AGE   IP            NODE                NOMINATED NODE   READINESS GATES
apollo-adminservice-5cccf97c64-k8fhx   1/1     Running   0          73m   172.7.22.17   hdss7-22.host.com   <none>           <none>
apollo-configservice-5f6555448-6xbwj   1/1     Running   0          25h   172.7.21.10   hdss7-21.host.com   <none>           <none>
apollo-portal-57bc86966d-2pftd         1/1     Running   0          15s   172.7.21.12   hdss7-21.host.com   <none>           <none>
dubbo-monitor-6676dd74cc-r559b         1/1     Running   0          25h   172.7.22.20   hdss7-22.host.com   <none>           <none>
jenkins-56cfb9c479-dfh6w               1/1     Running   0          25h   172.7.21.17   hdss7-21.host.com   <none>           <none>
[root@hdss7-22 ~]#
```
```buildoutcfg
C:\Windows\System32\drivers\etc\
10.4.7.10		dashboard.od.com  traefik.od.com  jenkins.od.com  dubbo-monitor.od.com  config.od.com  portal.od.com
10.4.7.200		k8s-yaml.od.com   harbor.od.com
Chrome: portal.od.com
username: apollo
password: admin
Admin Tools --> User Management --> reset password
```