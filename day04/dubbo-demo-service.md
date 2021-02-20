```text
app_name: dubbo-demo-service
image_name: app/dubbo-demo-service
git_repo: https://gitee.com/xilongj/dubbo-demo-service.git
git_ver: master
add_tag: 210220_22:36
mvn_dir: ./
target_dir: ./dubbo-server/target
mvn_cmd: mvn clean package -Dmaven.test.skip=true
base_image: base/jre8:8u112
maven: 3.6.2-8u242
```
```text
Started by user admin
Running in Durability level: MAX_SURVIVABILITY
[Pipeline] Start of Pipeline
[Pipeline] node
Running on Jenkins in /var/jenkins_home/workspace/dubbo-demo
[Pipeline] {
[Pipeline] stage
[Pipeline] { (pull)
[Pipeline] sh
+ git clone https://gitee.com/xilongj/dubbo-demo-service.git dubbo-demo-service/2
Cloning into 'dubbo-demo-service/2'...
+ cd dubbo-demo-service/2
+ git checkout master
Already on 'master'
Your branch is up-to-date with 'origin/master'.
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (build)
[Pipeline] sh
+ cd dubbo-demo-service/2
+ /var/jenkins_home/maven-3.6.2-8u242/bin/mvn clean package -Dmaven.test.skip=true
[INFO] Scanning for projects...
[WARNING]
[WARNING] Some problems were encountered while building the effective model for com.od.dubbotest:dubbo-server:jar:0.0.1-SNAPSHOT
[WARNING] 'parent.relativePath' of POM com.od.dubbotest:dubbo-server:0.0.1-SNAPSHOT (/var/jenkins_home/workspace/dubbo-demo/dubbo-demo-service/2/dubbo-server/pom.xml) points at com.od.dubbotest:dubbo-demo-service instead of org.springframework.boot:spring-boot-starter-parent, please verify your project structure @ line 14, column 10
[WARNING]
[WARNING] It is highly recommended to fix these problems because they threaten the stability of your build.
[WARNING]
[WARNING] For this reason, future Maven versions might no longer support building such malformed projects.
[WARNING]
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Build Order:
[INFO]
[INFO] dubbotest-api                                                      [jar]
[INFO] dubbotest                                                          [jar]
[INFO] demo                                                               [pom]
[INFO]
[INFO] ---------------------< com.od.dubbotest:dubbo-api >---------------------
[INFO] Building dubbotest-api 0.0.1-SNAPSHOT                              [1/3]
[INFO] --------------------------------[ jar ]---------------------------------
[INFO]
[INFO] --- maven-clean-plugin:2.5:clean (default-clean) @ dubbo-api ---
[INFO]
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ dubbo-api ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] skip non existing resourceDirectory /var/jenkins_home/workspace/dubbo-demo/dubbo-demo-service/2/dubbo-api/src/main/resources
[INFO]
[INFO] --- maven-compiler-plugin:3.1:compile (default-compile) @ dubbo-api ---
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 1 source file to /var/jenkins_home/workspace/dubbo-demo/dubbo-demo-service/2/dubbo-api/target/classes
[INFO]
[INFO] --- maven-resources-plugin:2.6:testResources (default-testResources) @ dubbo-api ---
[INFO] Not copying test resources
[INFO]
[INFO] --- maven-compiler-plugin:3.1:testCompile (default-testCompile) @ dubbo-api ---
[INFO] Not compiling test sources
[INFO]
[INFO] --- maven-surefire-plugin:2.12.4:test (default-test) @ dubbo-api ---
[INFO] Tests are skipped.
[INFO]
[INFO] --- maven-jar-plugin:2.4:jar (default-jar) @ dubbo-api ---
[INFO] Building jar: /var/jenkins_home/workspace/dubbo-demo/dubbo-demo-service/2/dubbo-api/target/dubbo-api-0.0.1-SNAPSHOT.jar
[INFO]
[INFO] -------------------< com.od.dubbotest:dubbo-server >--------------------
[INFO] Building dubbotest 0.0.1-SNAPSHOT                                  [2/3]
[INFO] --------------------------------[ jar ]---------------------------------
[INFO]
[INFO] --- maven-clean-plugin:2.5:clean (default-clean) @ dubbo-server ---
[INFO]
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ dubbo-server ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] Copying 4 resources
[INFO]
[INFO] --- maven-compiler-plugin:3.1:compile (default-compile) @ dubbo-server ---
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 2 source files to /var/jenkins_home/workspace/dubbo-demo/dubbo-demo-service/2/dubbo-server/target/classes
[INFO]
[INFO] --- maven-resources-plugin:2.6:testResources (default-testResources) @ dubbo-server ---
[INFO] Not copying test resources
[INFO]
[INFO] --- maven-compiler-plugin:3.1:testCompile (default-testCompile) @ dubbo-server ---
[INFO] Not compiling test sources
[INFO]
[INFO] --- maven-surefire-plugin:2.18.1:test (default-test) @ dubbo-server ---
[INFO] Tests are skipped.
[INFO]
[INFO] --- maven-jar-plugin:2.5:jar (default-jar) @ dubbo-server ---
[INFO] Building jar: /var/jenkins_home/workspace/dubbo-demo/dubbo-demo-service/2/dubbo-server/target/dubbo-server.jar
[INFO]
[INFO] --- spring-boot-maven-plugin:1.3.1.RELEASE:repackage (default) @ dubbo-server ---
[INFO]
[INFO] ----------------< com.od.dubbotest:dubbo-demo-service >-----------------
[INFO] Building demo 0.0.1-SNAPSHOT                                       [3/3]
[INFO] --------------------------------[ pom ]---------------------------------
[INFO]
[INFO] --- maven-clean-plugin:2.5:clean (default-clean) @ dubbo-demo-service ---
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Summary for demo 0.0.1-SNAPSHOT:
[INFO]
[INFO] dubbotest-api ...................................... SUCCESS [  6.416 s]
[INFO] dubbotest .......................................... SUCCESS [  5.333 s]
[INFO] demo ............................................... SUCCESS [  0.004 s]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  12.463 s
[INFO] Finished at: 2021-02-20T22:27:45+08:00
[INFO] ------------------------------------------------------------------------
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (package)
[Pipeline] sh
+ cd dubbo-demo-service/2
+ cd ./dubbo-server/target
+ mkdir project_dir
+ mv dubbo-server.jar ./project_dir
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (image)
[Pipeline] writeFile
[Pipeline] sh
+ cd dubbo-demo-service/2
+ docker build -t harbor.od.com/app/dubbo-demo-service:master_210220_2227 .
Sending build context to Docker daemon  14.45MB

Step 1/2 : FROM harbor.od.com/base/jre8:8u112
 ---> c59cb4d34024
Step 2/2 : ADD ./dubbo-server/target/project_dir /opt/project_dir
 ---> 5091c75e8cc9
Successfully built 5091c75e8cc9
Successfully tagged harbor.od.com/app/dubbo-demo-service:master_210220_2227
+ docker push harbor.od.com/app/dubbo-demo-service:master_210220_2227
The push refers to repository [harbor.od.com/app/dubbo-demo-service]
f9008515975c: Preparing
6a5ba443b93e: Preparing
8b6fcd6af9ce: Preparing
e4da095e72a9: Preparing
df0b2f6172f0: Preparing
2d85bd2a1bb6: Preparing
0690f10a63a5: Preparing
c843b2cf4e12: Preparing
fddd8887b725: Preparing
42052a19230c: Preparing
8d4d1ab5ff74: Preparing
fddd8887b725: Waiting
0690f10a63a5: Waiting
c843b2cf4e12: Waiting
42052a19230c: Waiting
2d85bd2a1bb6: Waiting
8b6fcd6af9ce: Layer already exists
df0b2f6172f0: Layer already exists
6a5ba443b93e: Layer already exists
e4da095e72a9: Layer already exists
0690f10a63a5: Layer already exists
2d85bd2a1bb6: Layer already exists
c843b2cf4e12: Layer already exists
fddd8887b725: Layer already exists
42052a19230c: Layer already exists
8d4d1ab5ff74: Layer already exists
f9008515975c: Pushed
master_210220_2227: digest: sha256:3ae3ab7c2461f9ccf33bc2f50811862d1d144e3d0a4542b355b77ef0e69e88d5 size: 2617
[Pipeline] }
[Pipeline] // stage
[Pipeline] }
[Pipeline] // node
[Pipeline] End of Pipeline
Finished: SUCCESS
```