## [Maven](https://maven.apache.org/)
Apache Maven is a software project management and comprehension tool. Based on the concept of a project object model (POM), Maven can manage a project's build, reporting and documentation from a central piece of information.

### [Download Maven](https://archive.apache.org/dist/maven/maven-3/)
#### [hdss7-200]
```buildoutcfg
[root@hdss7-200 ~]# cd /opt/src/
[root@hdss7-200 src]# wget https://archive.apache.org/dist/maven/maven-3/3.6.2/binaries/apache-maven-3.6.2-bin.tar.gz

[root@hdss7-200 src]# ls -l
total 540128
-rw-r--r-- 1 root root   9142315 Sep  3  2019 apache-maven-3.6.2-bin.tar.gz
```
#### Java version
```buildoutcfg
# shell in Jenkins
root@jenkins-56cfb9c479-fqksc:/# java -version
openjdk version "1.8.0_242"
OpenJDK Runtime Environment (build 1.8.0_242-b08)
OpenJDK 64-Bit Server VM (build 25.242-b08, mixed mode)
```
```buildoutcfg
[root@hdss7-200 src]# tar xvf apache-maven-3.6.2-bin.tar.gz -C /data/xsj-storage/jenkins_home/

[root@hdss7-200 ~]# cd /data/xsj-storage/jenkins_home/
[root@hdss7-200 jenkins_home]# mv apache-maven-3.6.2 maven-3.6.2-8u242
[root@hdss7-200 jenkins_home]# ls -l maven-3.6.2-8u242/
total 28
drwxr-xr-x 2 root    root       97 Feb 20 01:19 bin
drwxr-xr-x 2 root    root       42 Feb 20 01:19 boot
drwxrwxr-x 3 xilongj xilongj    63 Aug 27  2019 conf
drwxrwxr-x 4 xilongj xilongj  4096 Feb 20 01:19 lib
-rw-rw-r-- 1 xilongj xilongj 12846 Aug 27  2019 LICENSE
-rw-rw-r-- 1 xilongj xilongj   182 Aug 27  2019 NOTICE
-rw-rw-r-- 1 xilongj xilongj  2533 Aug 27  2019 README.txt
```
#### Setup aliyun
```buildoutcfg
[root@hdss7-200 maven-3.6.2-8u242]# cat -n conf/settings.xml
   152      <mirror>
   153        <id>nexus-aliyun</id>
   154        <mirrorOf>*</mirrorOf>
   155        <name>Nexus aliyun</name>
   156        <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
   157      </mirror>
```
#### Define Java_Home
```buildoutcfg
[root@hdss7-200 ~]# grep "JAVA_HOME" /data/xsj-storage/jenkins_home/maven-3.6.2-8u242/bin/mvn
#   JAVA_HOME       Must point at your Java Development Kit installation.
```
```buildoutcfg
# test lab
[root@hdss7-200 ~]# docker images | grep jre
openjdk                            8-jre           608a4320629a   5 weeks ago     268MB
stanleyws/jre8                     8u112           fa3a085d6ef1   3 years ago     363MB
java                               openjdk-8-jre   e44d62cf8862   4 years ago     311MB

[root@hdss7-200 ~]# docker inspect --format='{{index .Created}}'  openjdk:8-jre
2021-01-12T11:00:59.623297734Z
[root@hdss7-200 ~]# docker inspect --format='{{index .Created}}'  java:openjdk-8-jre
2017-01-17T00:53:21.516522468Z
[root@hdss7-200 ~]# docker inspect --format='{{index .Created}}'  stanleyws/jre8:8u112
2017-05-10T02:20:55.902723144Z
```
```buildoutcfg
[root@hdss7-200 ~]# docker images | grep jre
stanleyws/jre8                     8u112           fa3a085d6ef1   3 years ago     363MB

[root@hdss7-200 ~]# docker tag fa3a085d6ef1 harbor.od.com/public/jre:8u112
[root@hdss7-200 ~]# docker push harbor.od.com/public/jre:8u112
[root@hdss7-200 ~]# docker images | grep jre
stanleyws/jre8                     8u112           fa3a085d6ef1   3 years ago     363MB
harbor.od.com/public/jre           8u112           fa3a085d6ef1   3 years ago     363MB
```
```buildoutcfg
[root@hdss7-200 ~]# cd /data/dockerfile/
[root@hdss7-200 dockerfile]# mkdir jre8
[root@hdss7-200 dockerfile]# cd jre8/
```
```buildoutcfg
# dockerfile
[root@hdss7-200 jre8]# cat Dockerfile
FROM harbor.od.com/public/jre:8u112
RUN /bin/cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime &&\
    echo 'Asia/Shanghai' >/etc/timezone
ADD config.yml /opt/prom/config.yml
ADD jmx_javaagent-0.3.1.jar /opt/prom/
WORKDIR /opt/project_dir
ADD entrypoint.sh /entrypoint.sh
CMD ["/entrypoint.sh"]
```
```buildoutcfg
# config.yaml
[root@hdss7-200 jre8]# cat config.yml
---
rules:
  - pattern: '.*'
```
```buildoutcfg
# entrypoint.sh
[root@hdss7-200 jre8]# cat entrypoint.sh
#!/bin/sh
M_OPTS="-Duser.timezone=Asia/Shanghai -javaagent:/opt/prom/jmx_javaagent-0.3.1.jar=$(hostname -i):${M_PORT:-"12346"}:/opt/prom/config.yml"
C_OPTS=${C_OPTS}
JAR_BALL=${JAR_BALL}
exec java -jar ${M_OPTS} ${C_OPTS} ${JAR_BALL}
[root@hdss7-200 jre8]# chmod +x entrypoint.sh
```

#### [Download jxm_javaagent](https://repo1.maven.org/maven2/io/prometheus/jmx/jmx_prometheus_javaagent/)
```buildoutcfg
[root@hdss7-200 jre8]# wget https://repo1.maven.org/maven2/io/prometheus/jmx/jmx_prometheus_javaagent/0.3.1/jmx_prometheus_javaagent-0.3.1.jar -O jmx_javaagent-0.3.1.jar
--2021-02-20 12:34:31--  https://repo1.maven.org/maven2/io/prometheus/jmx/jmx_prometheus_javaagent/0.3.1/jmx_prometheus_javaagent-0.3.1.jar
Resolving repo1.maven.org (repo1.maven.org)... 151.101.24.209
Connecting to repo1.maven.org (repo1.maven.org)|151.101.24.209|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 367417 (359K) [application/java-archive]
Saving to: ‘jmx_javaagent-0.3.1.jar’

100%[=======================================================================================================================>] 367,417      104KB/s   in 3.4s

2021-02-20 12:34:35 (104 KB/s) - ‘jmx_javaagent-0.3.1.jar’ saved [367417/367417]
```
#### Create project base
```buildoutcfg
# http://harbor.od.com
# Project Name: base
# Access Level: Public
```
```buildoutcfg
[root@hdss7-200 jre8]# ls -l /data/dockerfile/jre8/
total 372
-rw-r--r-- 1 root root     29 Feb 20 12:39 config.yml
-rw-r--r-- 1 root root    297 Feb 20 12:31 Dockerfile
-rwxr-xr-x 1 root root    234 Feb 20 12:40 entrypoint.sh
-rw-r--r-- 1 root root 367417 May 10  2018 jmx_javaagent-0.3.1.jar
```
#### Build log
```buildoutcfg
[root@hdss7-200 jre8]# docker build . -t harbor.od.com/base/jre8:8u112
Sending build context to Docker daemon  372.2kB
Step 1/7 : FROM harbor.od.com/public/jre:8u112
 ---> fa3a085d6ef1
Step 2/7 : RUN /bin/cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime &&    echo 'Asia/Shanghai' >/etc/timezone
 ---> Using cache
 ---> bebb84c9a3e1
Step 3/7 : ADD config.yml /opt/prom/config.yml
 ---> f36b4ad99373
Step 4/7 : ADD jmx_javaagent-0.3.1.jar /opt/prom/
 ---> 4a8427282111
Step 5/7 : WORKDIR /opt/project_dir
 ---> Running in 44ce72c69096
Removing intermediate container 44ce72c69096
 ---> 200af2e2dc44
Step 6/7 : ADD entrypoint.sh /entrypoint.sh
 ---> af25740e651f
Step 7/7 : CMD ["/entrypoint.sh"]
 ---> Running in 2b9f2b22807d
Removing intermediate container 2b9f2b22807d
 ---> c59cb4d34024
Successfully built c59cb4d34024
Successfully tagged harbor.od.com/base/jre8:8u112
```
```buildoutcfg
[root@hdss7-200 ~]# docker push harbor.od.com/base/jre8:8u112
The push refers to repository [harbor.od.com/base/jre8]
6a5ba443b93e: Pushed
8b6fcd6af9ce: Pushed
e4da095e72a9: Pushed
df0b2f6172f0: Pushed
2d85bd2a1bb6: Pushed
0690f10a63a5: Mounted from public/jre
c843b2cf4e12: Mounted from public/jre
fddd8887b725: Mounted from public/jre
42052a19230c: Mounted from public/jre
8d4d1ab5ff74: Mounted from public/jre
8u112: digest: sha256:931b6ef050c72a4eb3c5c1176cc363c4bc2f4e03730cb5fb2cd7d7c1679ede87 size: 2405
```
```buildoutcfg
[root@hdss7-200 jre8]# docker images | grep jre
harbor.od.com/base/jre8            8u112           c59cb4d34024   About a minute ago   363MB
stanleyws/jre8                     8u112           fa3a085d6ef1   3 years ago          363MB
harbor.od.com/public/jre           8u112           fa3a085d6ef1   3 years ago          363MB
```
***

### Setup pipeline on Jenkins
* New Item or Create a Job
  * Enter an item name: dubbo-demo
  * Select Pipeline then click ok

* General
  * check Discard old builds
    * Days to keep builds: 3
    * Max # of builds to keep: 30
  * check This project is parameterized
    * String Parameter [1]
      * Name: app_name
      * Description: Project Name, ex. dubbo-demo-service
      * check Trim the string
    
    * String Parameter [2]
      * Name: image_name
      * Description: Docker image name, ex. app/dubbo-demo-service;   
                     Create private project on harbor.
      * check Trim the string
    
    * String Parameter [3]
      * Name: git_repo
      * Description: Git central warehouse address， ex. https://gitee.com/xilongj/dubbo-demo-service.git
      * check Trim the string
    
    * String Parameter [4]
      * Name: git_ver
      * Description: Project branch or version number
      * check Trim the string 
    
    * String Parameter [5]
      * Name: add_tag
      * Description: Part of docker image tag, date stamp, ex. 210220_1621
      * check Trim the string
    
    * String Parameter [6]
      * Name: mvn_dir
      * Default value: ./  
      * Description: Project compile directory, default value is root
      * check Trim the string
    
    * String Parameter [7]
      * Name: target_dir
      * Default value: ./target
      * Description: Directory where the jar/war package generated after compiling the project.
      * check Trim the string
    
    * String Parameter [8]
      * Name: mvn_cmd
      * Default value: mvn clean package -Dmaven.test.skip=true
      * Description: Execute commands used for compilation, ex. mvn clean package -Dmaven.test.skip=true
    
    * Choice Parameter [9]
      * Name: base_image
      * Choices: 
        * base/jre7:7u80
        * base/jre8:8u112
      * Description: Docker base image for project
    
    * Choice Parameter [10]
      * Name: maven
      * Choices: 
        * 3.6.1-8u232
        * 3.6.2-8u242
        * 3.2.5-7u045
        * 2.2.1-6u025
      * Description: Maven version for executing compilation

* Advanced Project Options
```buildoutcfg
pipeline {
  agent any 
    stages {
      stage('pull') { //get project code from repo 
        steps {
          sh "git clone ${params.git_repo} ${params.app_name}/${env.BUILD_NUMBER} && cd ${params.app_name}/${env.BUILD_NUMBER} && git checkout ${params.git_ver}"
        }
      }
      stage('build') { //exec mvn cmd
        steps {
          sh "cd ${params.app_name}/${env.BUILD_NUMBER}  && /var/jenkins_home/maven-${params.maven}/bin/${params.mvn_cmd}"
        }
      }
      stage('package') { //move jar file into project_dir
        steps {
          sh "cd ${params.app_name}/${env.BUILD_NUMBER} && cd ${params.target_dir} && mkdir project_dir && mv *.jar ./project_dir"
        }
      }
      stage('image') { //build image and push to registry
        steps {
          writeFile file: "${params.app_name}/${env.BUILD_NUMBER}/Dockerfile", text: """FROM harbor.od.com/${params.base_image}
ADD ${params.target_dir}/project_dir /opt/project_dir"""
          sh "cd  ${params.app_name}/${env.BUILD_NUMBER} && docker build -t harbor.od.com/${params.image_name}:${params.git_ver}_${params.add_tag} . && docker push harbor.od.com/${params.image_name}:${params.git_ver}_${params.add_tag}"
        }
      }
    }
}
```
#### Build with Parameters
```text
01. app_name: dubbo-demo-service
02. image_name: app/dubbo-demo-service
03. git_repo: https://gitee.com/xilongj/dubbo-demo-service.git
04. git_ver: master
05. add_tag: 210220_22:36
06. mvn_dir: ./
07. target_dir: ./dubbo-server/target
08. mvn_cmd: mvn clean package -Dmaven.test.skip=true
09. base_image: base/jre8:8u112
10. maven: 3.6.2-8u242
```
### Create deployment.yaml
#### [hdss7-200]
```text
[root@hdss7-200 ~]# mkdir -p /data/k8s-yaml/dubbo-demo-service
[root@hdss7-200 ~]# cd /data/k8s-yaml/dubbo-demo-service
```
```buildoutcfg
# deployment.yaml
[root@hdss7-200 dubbo-demo-service]# cat deployment.yaml
kind: Deployment
apiVersion: extensions/v1beta1
metadata:
  name: dubbo-demo-service
  namespace: app
  labels:
    name: dubbo-demo-service
spec:
  replicas: 1
  selector:
    matchLabels:
      name: dubbo-demo-service
  template:
    metadata:
      labels:
        app: dubbo-demo-service
        name: dubbo-demo-service
    spec:
      containers:
      - name: dubbo-demo-service
        image: harbor.od.com/app/dubbo-demo-service:master_210220_2227
        ports:
        - containerPort: 20880
          protocol: TCP
        env:
        - name: JAR_BALL
          value: dubbo-server.jar
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
#### [hdss7-21]
```buildoutcfg
[root@hdss7-21 ~]# kubectl create ns app
namespace/app created
[root@hdss7-21 ~]# kubectl create secret docker-registry harbor --docker-server=harbor.od.com --docker-username=admin --docker-password=xlnx@2021 -n app
secret/harbor created
```
#### [hdss7-11]
```buildoutcfg
[root@hdss7-11 ~]# cd /opt/zookeeper
[root@hdss7-11 zookeeper]# bin/zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /opt/zookeeper/bin/../conf/zoo.cfg
Mode: follower
```
#### Login ZooKeeper
```buildoutcfg
[root@hdss7-11 zookeeper]# bin/zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /opt/zookeeper/bin/../conf/zoo.cfg
Mode: follower
[root@hdss7-11 zookeeper]# bin/zkCli.sh -server localhost:2181
Connecting to localhost:2181
2021-02-20 15:03:09,083 [myid:] - INFO  [main:Environment@100] - Client environment:zookeeper.version=3.4.14-4c25d480e66aadd371de8bd2fd8da255ac140bcf, built on 03/06/2019 16:18 GMT
2021-02-20 15:03:09,087 [myid:] - INFO  [main:Environment@100] - Client environment:host.name=hdss7-11.host.com
2021-02-20 15:03:09,087 [myid:] - INFO  [main:Environment@100] - Client environment:java.version=1.8.0_281
2021-02-20 15:03:09,088 [myid:] - INFO  [main:Environment@100] - Client environment:java.vendor=Oracle Corporation
2021-02-20 15:03:09,088 [myid:] - INFO  [main:Environment@100] - Client environment:java.home=/usr/java/jdk1.8.0_281/jre
2021-02-20 15:03:09,088 [myid:] - INFO  [main:Environment@100] - Client environment:java.class.path=/opt/zookeeper/bin/../zookeeper-server/target/classes:/opt/zookeeper/bin/../build/classes:/opt/zookeeper/bin/../zookeeper-server/target/lib/*.jar:/opt/zookeeper/bin/../build/lib/*.jar:/opt/zookeeper/bin/../lib/slf4j-log4j12-1.7.25.jar:/opt/zookeeper/bin/../lib/slf4j-api-1.7.25.jar:/opt/zookeeper/bin/../lib/netty-3.10.6.Final.jar:/opt/zookeeper/bin/../lib/log4j-1.2.17.jar:/opt/zookeeper/bin/../lib/jline-0.9.94.jar:/opt/zookeeper/bin/../lib/audience-annotations-0.5.0.jar:/opt/zookeeper/bin/../zookeeper-3.4.14.jar:/opt/zookeeper/bin/../zookeeper-server/src/main/resources/lib/*.jar:/opt/zookeeper/bin/../conf::/usr/java/jdk/lib:/usr/java/jdk/lib/tools.jar
2021-02-20 15:03:09,089 [myid:] - INFO  [main:Environment@100] - Client environment:java.library.path=/usr/java/packages/lib/amd64:/usr/lib64:/lib64:/lib:/usr/lib
2021-02-20 15:03:09,089 [myid:] - INFO  [main:Environment@100] - Client environment:java.io.tmpdir=/tmp
2021-02-20 15:03:09,089 [myid:] - INFO  [main:Environment@100] - Client environment:java.compiler=<NA>
2021-02-20 15:03:09,089 [myid:] - INFO  [main:Environment@100] - Client environment:os.name=Linux
2021-02-20 15:03:09,089 [myid:] - INFO  [main:Environment@100] - Client environment:os.arch=amd64
2021-02-20 15:03:09,089 [myid:] - INFO  [main:Environment@100] - Client environment:os.version=3.10.0-1160.el7.x86_64
2021-02-20 15:03:09,089 [myid:] - INFO  [main:Environment@100] - Client environment:user.name=root
2021-02-20 15:03:09,089 [myid:] - INFO  [main:Environment@100] - Client environment:user.home=/root
2021-02-20 15:03:09,089 [myid:] - INFO  [main:Environment@100] - Client environment:user.dir=/opt/zookeeper-3.4.14
2021-02-20 15:03:09,090 [myid:] - INFO  [main:ZooKeeper@442] - Initiating client connection, connectString=localhost:2181 sessionTimeout=30000 watcher=org.apache.zookeeper.ZooKeeperMain$MyWatcher@25f38edc
Welcome to ZooKeeper!
2021-02-20 15:03:09,111 [myid:] - INFO  [main-SendThread(localhost:2181):ClientCnxn$SendThread@1025] - Opening socket connection to server localhost/0:0:0:0:0:0:0:1:2181. Will not attempt to authenticate using SASL (unknown error)
JLine support is enabled
2021-02-20 15:03:09,117 [myid:] - INFO  [main-SendThread(localhost:2181):ClientCnxn$SendThread@879] - Socket connection established to localhost/0:0:0:0:0:0:0:1:2181, initiating session
2021-02-20 15:03:09,125 [myid:] - INFO  [main-SendThread(localhost:2181):ClientCnxn$SendThread@1299] - Session establishment complete on server localhost/0:0:0:0:0:0:0:1:2181, sessionid = 0x100265e51e90001, negotiated timeout = 30000

WATCHER::

WatchedEvent state:SyncConnected type:None path:null
[zk: localhost:2181(CONNECTED) 0] ls /
[zookeeper]
```
#### [hdss7-22]
```buildoutcfg
[root@hdss7-22 ~]# kubectl apply -f http://k8s-yaml.od.com/dubbo-demo-service/deployment.yaml
deployment.extensions/dubbo-demo-service created

[root@hdss7-22 ~]# kubectl get pods -o wide -n app
NAME                                  READY   STATUS    RESTARTS   AGE   IP            NODE                NOMINATED NODE   READINESS GATES
dubbo-demo-service-6b596ff655-5lq9s   1/1     Running   0          13s   172.7.21.13   hdss7-21.host.com   <none>           <none>

[root@hdss7-22 ~]# kubectl describe pod dubbo-demo-service-6b596ff655-5lq9s -n app
```
#### Dashboard
```buildoutcfg
# Namespace: app
Dubbo server started
```
#### [hdss7-11]
```buildoutcfg
[zk: localhost:2181(CONNECTED) 0] ls /
[dubbo, zookeeper]
[zk: localhost:2181(CONNECTED) 1] ls /dubbo
[com.od.dubbotest.api.HelloService]
```
### [Dubbo Monitor](https://github.com/Jeromefromcn/dubbo-monitor)
```buildoutcfg
[root@hdss7-200 ~]# [root@hdss7-200 ~]# git clone https://github.com/Jeromefromcn/dubbo-monitor.git
[root@hdss7-200 ~]# mv dubbo-monitor /opt/src/
```
```buildoutcfg
[root@hdss7-200 ~]# cat /opt/src/dubbo-monitor/dubbo-monitor-simple/conf/dubbo_origin.properties
##
# Copyright 1999-2011 Alibaba Group.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
##
dubbo.container=log4j,spring,registry,jetty
dubbo.application.name=dubbo-monitor
dubbo.application.owner=HomeEdu
dubbo.registry.address=zookeeper://{ZOOKEEPER_ADDRESS}
dubbo.registry.address=zookeeper://zk1.od.com:2181?backup=zk2.od.com:2181,zk3.od.com:2181
dubbo.protocol.port=20880
dubbo.jetty.port=8080
dubbo.jetty.directory=/dubbo-monitor-simple/monitor
dubbo.charts.directory=/dubbo-monitor-simple/charts
dubbo.statistics.directory=/dubbo-monitor-simple/statistics
dubbo.log4j.file=logs/dubbo-monitor.log
dubbo.log4j.level=WARN
```
```buildoutcfg
[root@hdss7-200 dubbo-monitor]# sed -r -i -e '/^nohup/{p;:a;N;$!ba;d}'  ./dubbo-monitor-simple/bin/start.sh && sed  -r -i -e "s%^nohup(.*)%exec \1%"  ./dubbo-monitor-simple/bin/start.sh
[root@hdss7-200 dubbo-monitor]# cat dubbo-monitor-simple/bin/start.sh
#!/bin/bash
sed -e "s/{ZOOKEEPER_ADDRESS}/$ZOOKEEPER_ADDRESS/g" /dubbo-monitor-simple/conf/dubbo_origin.properties > /dubbo-monitor-simple/conf/dubbo.properties
cd `dirname $0`
BIN_DIR=`pwd`
cd ..
DEPLOY_DIR=`pwd`
CONF_DIR=$DEPLOY_DIR/conf

SERVER_NAME=`sed '/dubbo.application.name/!d;s/.*=//' conf/dubbo.properties | tr -d '\r'`
SERVER_PROTOCOL=`sed '/dubbo.protocol.name/!d;s/.*=//' conf/dubbo.properties | tr -d '\r'`
SERVER_PORT=`sed '/dubbo.protocol.port/!d;s/.*=//' conf/dubbo.properties | tr -d '\r'`
LOGS_FILE=`sed '/dubbo.log4j.file/!d;s/.*=//' conf/dubbo.properties | tr -d '\r'`

if [ -z "$SERVER_NAME" ]; then
    SERVER_NAME=`hostname`
fi

PIDS=`ps -f | grep java | grep "$CONF_DIR" |awk '{print $2}'`
if [ -n "$PIDS" ]; then
    echo "ERROR: The $SERVER_NAME already started!"
    echo "PID: $PIDS"
    exit 1
fi

if [ -n "$SERVER_PORT" ]; then
    SERVER_PORT_COUNT=`netstat -tln | grep $SERVER_PORT | wc -l`
    if [ $SERVER_PORT_COUNT -gt 0 ]; then
        echo "ERROR: The $SERVER_NAME port $SERVER_PORT already used!"
        exit 1
    fi
fi

LOGS_DIR=""
if [ -n "$LOGS_FILE" ]; then
    LOGS_DIR=`dirname $LOGS_FILE`
else
    LOGS_DIR=$DEPLOY_DIR/logs
fi
if [ ! -d $LOGS_DIR ]; then
    mkdir $LOGS_DIR
fi
STDOUT_FILE=$LOGS_DIR/stdout.log

LIB_DIR=$DEPLOY_DIR/lib
LIB_JARS=`ls $LIB_DIR|grep .jar|awk '{print "'$LIB_DIR'/"$0}'|tr "\n" ":"`

JAVA_OPTS=" -Djava.awt.headless=true -Djava.net.preferIPv4Stack=true "
JAVA_DEBUG_OPTS=""
if [ "$1" = "debug" ]; then
    JAVA_DEBUG_OPTS=" -Xdebug -Xnoagent -Djava.compiler=NONE -Xrunjdwp:transport=dt_socket,address=8000,server=y,suspend=n "
fi
JAVA_JMX_OPTS=""
if [ "$1" = "jmx" ]; then
    JAVA_JMX_OPTS=" -Dcom.sun.management.jmxremote.port=1099 -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.authenticate=false "
fi
JAVA_MEM_OPTS=""
BITS=`java -version 2>&1 | grep -i 64-bit`
if [ -n "$BITS" ]; then
    JAVA_MEM_OPTS=" -server -Xmx128m -Xms128m -Xmn32m -XX:PermSize=16m -Xss256k -XX:+DisableExplicitGC -XX:+UseConcMarkSweepGC -XX:+CMSParallelRemarkEnabled -XX:+UseCMSCompactAtFullCollection -XX:LargePageSizeInBytes=128m -XX:+UseFastAccessorMethods -XX:+UseCMSInitiatingOccupancyOnly -XX:CMSInitiatingOccupancyFraction=70 "
else
    JAVA_MEM_OPTS=" -server -Xms128m -Xmx128m -XX:PermSize=16m -XX:SurvivorRatio=2 -XX:+UseParallelGC "
fi

echo -e "Starting the $SERVER_NAME ...\c"
exec java $JAVA_OPTS $JAVA_MEM_OPTS $JAVA_DEBUG_OPTS $JAVA_JMX_OPTS -classpath $CONF_DIR:$LIB_JARS com.alibaba.dubbo.container.Main > $STDOUT_FILE 2>&1
```
```buildoutcfg
[root@hdss7-200 src]# ls -l
total 540128
-rw-r--r-- 1 root root   9142315 Sep  3  2019 apache-maven-3.6.2-bin.tar.gz
drwxr-xr-x 4 root root        81 Feb 21 15:31 dubbo-monitor
-rw-r--r-- 1 root root 543943276 Feb 11 13:12 harbor-offline-installer-v2.1.2.tgz

[root@hdss7-200 src]# cp -a dubbo-monitor/ /data/dockerfile/

[root@hdss7-200 src]# cd /data/dockerfile/
[root@hdss7-200 dockerfile]# ls -l
total 0
drwxr-xr-x 4 root root 81 Feb 21 15:31 dubbo-monitor
drwxr-xr-x 2 root root 78 Feb 19 23:05 jenkins
drwxr-xr-x 2 root root 94 Feb 20 12:50 jre8
```
```buildoutcfg
[root@hdss7-200 dubbo-monitor]# cat Dockerfile
FROM jeromefromcn/docker-alpine-java-bash
MAINTAINER Jerome Jiang
COPY dubbo-monitor-simple/ /dubbo-monitor-simple/
CMD /dubbo-monitor-simple/bin/start.sh
```
```buildoutcfg
[root@hdss7-200 dubbo-monitor]# docker build . -t harbor.od.com/infra/dubbo-monitor:latest
Sending build context to Docker daemon  50.02MB
Step 1/4 : FROM jeromefromcn/docker-alpine-java-bash
latest: Pulling from jeromefromcn/docker-alpine-java-bash
Image docker.io/jeromefromcn/docker-alpine-java-bash:latest uses outdated schema1 manifest format. Please upgrade to a schema2 image for better future compatibility. More information at https://docs.docker.com/registry/spec/deprecated-schema-v1/
420890c9e918: Pull complete
a3ed95caeb02: Pull complete
4a5cf8bc2931: Pull complete
6a17cae86292: Pull complete
4729ccfc7091: Pull complete
Digest: sha256:658f4a5a2f6dd06c4669f8f5baeb85ca823222cb938a15cfb7f6459c8cfe4f91
Status: Downloaded newer image for jeromefromcn/docker-alpine-java-bash:latest
 ---> 3114623bb27b
Step 2/4 : MAINTAINER Jerome Jiang
 ---> Running in 9756a2738cc9
Removing intermediate container 9756a2738cc9
 ---> 572d62c61a11
Step 3/4 : COPY dubbo-monitor-simple/ /dubbo-monitor-simple/
 ---> 2ef890e55020
Step 4/4 : CMD /dubbo-monitor-simple/bin/start.sh
 ---> Running in aeafff2d86f6
Removing intermediate container aeafff2d86f6
 ---> a413ff39101f
Successfully built a413ff39101f
Successfully tagged harbor.od.com/infra/dubbo-monitor:latest
```
```buildoutcfg
[root@hdss7-200 dubbo-monitor]# docker images | grep dubbo-monitor
harbor.od.com/infra/dubbo-monitor      latest          a413ff39101f   About a minute ago   139MB
[root@hdss7-200 dubbo-monitor]#
[root@hdss7-200 dubbo-monitor]#
[root@hdss7-200 dubbo-monitor]#
[root@hdss7-200 dubbo-monitor]# docker push harbor.od.com/infra/dubbo-monitor:latest
The push refers to repository [harbor.od.com/infra/dubbo-monitor]
177505d3b130: Pushed
6c05aa02bec9: Pushed
1bdff01a06a9: Pushed
5f70bf18a086: Mounted from public/pause
e271a1fb1dfc: Pushed
c56b7dabbc7a: Pushed
latest: digest: sha256:075d81653a455a701604ce039ca41173f5ce8dbdd50716df60d64ef29887529e size: 2400
```
#### [hdss7-200]
```buildoutcfg
[root@hdss7-200 ~]# mkdir -p /data/k8s-yaml/dubbo-monitor
```
```buildoutcfg
# deployment.yaml
[root@hdss7-200 dubbo-monitor]# cat deployment.yaml
kind: Deployment
apiVersion: extensions/v1beta1
metadata:
  name: dubbo-monitor
  namespace: infra
  labels:
    name: dubbo-monitor
spec:
  replicas: 1
  selector:
    matchLabels:
      name: dubbo-monitor
  template:
    metadata:
      labels:
        app: dubbo-monitor
        name: dubbo-monitor
    spec:
      containers:
      - name: dubbo-monitor
        image: harbor.od.com/infra/dubbo-monitor:latest
        ports:
        - containerPort: 8080
          protocol: TCP
        - containerPort: 20880
          protocol: TCP
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
[root@hdss7-200 dubbo-monitor]# cat svc.yaml
kind: Service
apiVersion: v1
metadata:
  name: dubbo-monitor
  namespace: infra
spec:
  ports:
  - protocol: TCP
    port: 8080
    targetPort: 8080
  selector:
    app: dubbo-monitor
  clusterIP: None
  type: ClusterIP
  sessionAffinity: None
```
```buildoutcfg
# ingress.yaml
[root@hdss7-200 dubbo-monitor]# cat ingress.yaml
kind: Ingress
apiVersion: extensions/v1beta1
metadata:
  name: dubbo-monitor
  namespace: infra
spec:
  rules:
  - host: dubbo-monitor.od.com
    http:
      paths:
      - path: /
        backend:
          serviceName: dubbo-monitor
          servicePort: 8080
```
#### [hdss7-22]
```buildoutcfg
[root@hdss7-22 ~]# kubectl apply -f http://k8s-yaml.od.com/dubbo-monitor/deployment.yaml

[root@hdss7-22 ~]# kubectl apply -f http://k8s-yaml.od.com/dubbo-monitor/svc.yaml

[root@hdss7-22 ~]# kubectl apply -f http://k8s-yaml.od.com/dubbo-monitor/ingress.yaml
```
```buildoutcfg
[root@hdss7-22 ~]# kubectl get pods -o wide -n infra
NAME                             READY   STATUS    RESTARTS   AGE     IP            NODE                NOMINATED NODE   READINESS GATES
dubbo-monitor-5bb45c8b97-5cvf7   1/1     Running   0          2m55s   172.7.22.17   hdss7-22.host.com   <none>           <none>
jenkins-56cfb9c479-fqksc         1/1     Running   0          2d4h    172.7.21.4    hdss7-21.host.com   <none>           <none>
```
### Update DNS
#### [hdss7-11]
```buildoutcfg
[root@hdss7-11 ~]# cat /var/named/od.com.zone
$ORIGIN od.com.
$TTL 600        ; 10 minutes
@       IN SOA dns.od.com. dnsadmin.od.com. (
                        2021021008      ; serial
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
```
```buildoutcfg
[root@hdss7-11 ~]# systemctl restart named
```
```buildoutcfg
[root@hdss7-11 ~]# dig -t A dubbo-monitor.od.com @10.4.7.11

; <<>> DiG 9.11.4-P2-RedHat-9.11.4-26.P2.el7_9.3 <<>> -t A dubbo-monitor.od.com @10.4.7.11
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 9139
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;dubbo-monitor.od.com.          IN      A

;; ANSWER SECTION:
dubbo-monitor.od.com.   60      IN      A       10.4.7.10

;; AUTHORITY SECTION:
od.com.                 600     IN      NS      dns.od.com.

;; ADDITIONAL SECTION:
dns.od.com.             60      IN      A       10.4.7.11

;; Query time: 0 msec
;; SERVER: 10.4.7.11#53(10.4.7.11)
;; WHEN: Sun Feb 21 18:13:00 CST 2021
;; MSG SIZE  rcvd: 99


[root@hdss7-11 ~]# dig -t A dubbo-monitor.od.com @10.4.7.11 +short
10.4.7.10
```
```buildoutcfg
# C:\Windows\System32\drivers\etc\hosts
10.4.7.10		dashboard.od.com  traefik.od.com  jenkins.od.com  dubbo-monitor.od.com
10.4.7.200		k8s-yaml.od.com   harbor.od.com
```
```buildoutcfg
Chrome: http://dubbo-monitor.od.com/index.html
```
### dubbo-demo-web
```buildoutcfg
01. app_name: dubbo-demo-consumer
02. image_name: app/dubbo-demo-consumer
03. git_repo: git@gitee.com:xilongj/dubbo-demo-web.git
04. git_ver: master
05. add_tag: 210222_20:36
06. mvn_dir: ./
07. target_dir: ./dubbo-client/target
08. mvn_cmd: mvn clean package -e -q -Dmaven.test.skip=true
09. base_image: base/jre8:8u112
10. maven: 3.6.2-8u242
```
