---
title: docker app 등록
categories:
  - General
feature_text: |
  ## docker app 등록
---

#### Goal

> h2 데이터베이스 docker pull & run

```linux
$> docker pull thomseno/h2

$> docker run -d -p 9092:9092 -p 8082:8082 -v /home/zunee/ehr/h2-data:/h2-data --name=myH2Server thomseno/h2


브라우저에서 아래와 같이 실행

http://localhost:8082/login.do

```

> spring boot app docker images 생성 & run

```linux

$> pwd
/home/zunee/ehr/ustra-springjpa

// gradle build 
$> ./gradlew clean
$> ./gradlew build


// docker image 생성
$> sudo docker build -t app .
[sudo] password for zunee:
Sending build context to Docker daemon  59.78MB
Step 1/4 : FROM openjdk:8
 ---> 3bc5f7759e81
Step 2/4 : ARG JAR_FILE_PATH=build/libs/ustra-springjpa-0.0.1-SNAPSHOT.jar
 ---> Running in 00d892784a7a
Removing intermediate container 00d892784a7a
 ---> bf83eb4bb6d0
Step 3/4 : ADD ${JAR_FILE_PATH} app.jar
 ---> ad9224ed263a
Step 4/4 : ENTRYPOINT ["java","-jar", "app.jar"]
 ---> Running in 815b955ce572
Removing intermediate container 815b955ce572
 ---> 0941567057ba
Successfully built 0941567057ba
Successfully tagged app:latest

// docker image 생성확인
$> sudo docker images -a
REPOSITORY    TAG       IMAGE ID       CREATED          SIZE
<none>        <none>    ad9224ed263a   11 seconds ago   584MB
app           latest    0941567057ba   11 seconds ago   584MB
<none>        <none>    bf83eb4bb6d0   12 seconds ago   526MB
openjdk       8         3bc5f7759e81   7 days ago       526MB
thomseno/h2   latest    cde6a23eab08   2 months ago     88.4MB

// docker run
$> sudo docker run -p 8080:8080 --name app -t app

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.3.5.RELEASE)

2022-02-02 09:35:33.871  INFO 1 --- [           main] c.h.u.UstraSpringjpaApplication          : Starting UstraSpringjpaApplication on a29668a1fd7f with PID 1 (/app.jar started by root in /)


// docker ps 삭제

$> sudo docker stop app
app

$> sudo docker ps -a
CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS                     PORTS                                                                                  NAMES
b915c57cd7f1   app            "java -jar app.jar"      8 minutes ago   Exited (1) 8 minutes ago                                                                                          app
684386f204aa   thomseno/h2    "/bin/sh -c 'java $J…"   2 hours ago     Up 2 hours                 0.0.0.0:8082->8082/tcp, :::8082->8082/tcp, 0.0.0.0:9092->9092/tcp, :::9092->9092/tcp   myH2Server
85a5cc079bcd   68842797233d   "java -jar app.jar"      4 hours ago     Exited (1) 4 hours ago                                                                                            suspicious_feynman

$> sudo docker rm app
app

$> sudo docker ps -a
CONTAINER ID   IMAGE          COMMAND                  CREATED       STATUS                   PORTS                                                                                  NAMES
684386f204aa   thomseno/h2    "/bin/sh -c 'java $J…"   3 hours ago   Up 3 hours               0.0.0.0:8082->8082/tcp, :::8082->8082/tcp, 0.0.0.0:9092->9092/tcp, :::9092->9092/tcp   myH2Server
85a5cc079bcd   68842797233d   "java -jar app.jar"      4 hours ago   Exited (1) 4 hours ago


$> sudo docker images -a
REPOSITORY    TAG       IMAGE ID       CREATED          SIZE
app           latest    13ea556d971c   13 minutes ago   584MB
<none>        <none>    28ef36d76613   13 minutes ago   584MB
<none>        <none>    6860d01b7c93   13 minutes ago   526MB
openjdk       8         3bc5f7759e81   7 days ago       526MB
thomseno/h2   latest    cde6a23eab08   3 months ago     88.4MB

$> sudo docker rmi -f app
Untagged: app:latest
Deleted: sha256:13ea556d971cca496e5e2a7fbd663e351f22fe3e3bc842f6e84598d55eebac7f
Deleted: sha256:28ef36d76613b4d071de3e4e335d1525ddff8ac48ef9cd1268f9fb470dea1d99
Deleted: sha256:716046d3548fa79a58915e351cc4430cee412c69b437e244b8b09379d6851fd1
Deleted: sha256:6860d01b7c930f7366c1cb3dd0725d11f76f4fddb567260d5f9fcb3a4633246f

$> sudo docker images -a
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
openjdk       8         3bc5f7759e81   7 days ago     526MB
thomseno/h2   latest    cde6a23eab08   3 months ago   88.4MB

```

$> sudo docker run -it --restart=always --name "Ubuntu_Test" ubuntu:16.04 /bash
