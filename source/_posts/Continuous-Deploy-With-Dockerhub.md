title: Continuous Deploy With Dockerhub
author: Sunkz
tags:
  - docker
  - webhook
  - ci/cd
  - ''
categories: []
date: 2019-09-21 18:50:00
---
## 准备工作

- Github
- Dockerhub
- Server (没有服务器的话,本机使用内网穿透也可以)

## 整体流程

![](https://tva1.sinaimg.cn/large/006y8mN6gy1g77fgk6h4zj31j60bimz2.jpg)

## 具体步骤 

### code

创建springboot工程提供一个接口, 并编写 dockerfile

```java
/**
 * hello 接口
 */
 @GetMapping("/hello")
 public String hello(){
     return "hello";
 }
```

```dockerfile
FROM openjdk:11

WORKDIR /workspace/app

COPY mvnw .
COPY .mvn .mvn
COPY pom.xml .
COPY src src

RUN ./mvnw package -DskipTests

ENTRYPOINT ["java","-jar","target/hello-0.0.1-SNAPSHOT.jar"]
```

### Github

创建hello repository

![](https://tva1.sinaimg.cn/large/006y8mN6gy1g77g4or0skj311w05saap.jpg)

### Dockerhub

创建hello repository并关联上Github的hello repository

<img src="https://tva1.sinaimg.cn/large/006y8mN6gy1g77g0bc9e7j31dk0u00vq.jpg" alt="" style="zoom:50%;" />

### hook

编写hook服务,监听 dockerhub hello repository变动

```java
/**
 * 在server运行上,当Dockerhub中hello repository有变动,将出触发该接口,该接口将会执行auto_deploy脚本
 */
@GetMapping("/hook")
public void hook(){
    Runtime.getRuntime().exec("/root/code/auto_deploy.sh");
}
```

```sh
#! /bin/bash

# 销毁旧的容器,从dockerhub拉取最新hello images并启动

docker stop hello
docker rm hello
docker pull sunkezheng/hello
docker run --name http -p8080:8080 -d sunkezheng/hello
```

在server上启动该hook服务

![](https://tva1.sinaimg.cn/large/006y8mN6gy1g77gm722goj317s02o0us.jpg)

### webhook

dockerhub hello repository添加webhook指向 server中的hook服务

<img src="https://tva1.sinaimg.cn/large/006y8mN6gy1g77gqaej82j319e0ms41h.jpg" style="zoom:67%;" />

## 触发流水线

至此整条流水线已经建立

![](https://tva1.sinaimg.cn/large/006y8mN6gy1g77gt03ryqj31j60bimz2.jpg)

将hello springboot工程提交到Github hello repository, 触发整条流水线.

![](https://tva1.sinaimg.cn/large/006y8mN6gy1g77h4x3ebnj31mu0i6gos.jpg)

curl 请求测试

![](https://tva1.sinaimg.cn/large/006y8mN6gy1g77hikuiufj31e403ut9m.jpg)

## 总结与展望

- Github Actions将支持CI/CD到自定的Server
- 使用Jenkins可以简化Hook部分
- 配合Nginx使用可实现 '零停机' 更新
- 第三方云厂商提供了大量模板式Serverless的DevOps解决方案