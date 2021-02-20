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
#### Setup Aliyun Source
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
#### Create a job
```buildoutcfg
# click New Item or Create a Job
# Enter an item name: dubbo-demo 
# select Pipeline then click OK
# General --> check Discard old builds --> Days to keep builds: 3 --> Max # of builds to keep: 30
# check This project is parameterized 
    --> select String Parameter --> Name: app_name --> Description: Project Name, ex. dubbo-demo-service --> check Trim the string
    --> select String Parameter --> Name: image_name --> Description: Docker Image Name, ex. app/dubbo-demo-service --> check Trim the string 
    --> 
```
* New Item or Create a Job
  * Enter an item name: dubbo-demo
  * Select Pipeline then click ok

* General
  * check Discard old builds
    * Days to keep builds: 3
    * Max # of builds to keep: 30
  * check This project is parameterized
    * String Parameter
      * Name: 
      * Description: 
      * check Trim the string
      
  * check This project is parameterized
    * String Parameter
      * Name: 
      * Description: 
      * check Trim the string
      
  * check This project is parameterized
    * String Parameter
      * Name: 
      * Description: 
      * check Trim the string

  * check This project is parameterized
    * String Parameter
      * Name: 
      * Description: 
      * check Trim the string 
        
  * check This project is parameterized
    * String Parameter
      * Name: 
      * Description: 
      * check Trim the string
    
  * check This project is parameterized
    * String Parameter
      * Name: 
      * Description: 
      * check Trim the string
    
  * check This project is parameterized
    * String Parameter
      * Name: 
      * Description: 
      * check Trim the string
    
  * check This project is parameterized
    * String Parameter
      * Name: 
      * Description: 
      * check Trim the string