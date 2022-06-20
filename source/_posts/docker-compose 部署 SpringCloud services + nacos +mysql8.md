---
title: docker-compose 部署 SpringCloud services + nacos +mysql8 
date: 2022-06-20
tags: [docker, spring, springcloud, nacos, mysql8]
---

# docker-compose 一键部署 SpringCloud services + nacos +mysql8 的疑难杂症

###### about

- nacos-server 版本为 2.1.0 , 完全兼容 spring cloud alibaba 2021.0.1.0 版本对应的 nacos-client:v1.4.3
- 完整项目见: https://github.com/luoshieryi/springCloud-learn1

## 遇到的问题与解决方案

### 1. 抽象出的 feign-client 子模块没有 Application.java 类, 导致无法编译/引入

在该模块 pom.xml 的 spring-boot-maven-plugin 中添加如下配置:

- 没有第一条配置无法编译
- 没有第二条配置无法被其他包引入

```xml
<configuration>
    <layout>NONE</layout>  <!--让maven不打包可执行jar，不扫描项目的main函数-->
    <classifier>exec</classifier> <!--普通jar和可执行jar不同名，普通jar为xx.jar ， 可执行jar为 xx-exec.jar-->
</configuration>
```

### 2. dev 与 prod 环境时不同的 mysql, nacos地址

1. 使用 spring.profiles.active 参数来指定 dev 和 prod 环境
2. 在 Dockerfile 中运行jar时: ENTRYPOINT java -jar /app/app.jar --spring.profiles.active=prod

### 3. 同一个 docker-compose 中, 通过 hostname 访问 nacos 失败

- "Connection refused"
- 关联 issue: https://github.com/alibaba/nacos/issues/7298

显式手动设置 hostname : 在 docker-compose.yml 中添加如下配置:

```yaml
services:
  nacos:
    hostname: nacos
```

### 4. 在 mysql 中初始化数据库用户和数据库

- 映射sql文件到容器的 /docker-entrypoint-initdb.d/ 文件夹中, 会自动按名称顺序执行语句
- 可以在 environment 中添加用户( 官网 nacos.io 的示例docker部署采用)
  ```env
    MYSQL_USER=nacos
    MYSQL_PASSWORD=nacos
  ```

### 5. "currentServerAddr: http://localhost:8848, err : Connection refused"

- "java.net.ConnectException: [NACOS HTTP-POST] The maximum number of tolerable server reconnection errors has been reached"
- 关联 issue: https://github.com/alibaba/spring-cloud-alibaba/issues/1599

项目引入了 spring-cloud-starter-alibaba-nacos-config 依赖, 但是又没有使用它提供的动态配置功能

### 6. 关于MySQL 8 的 "Public Key Retrival" 错误

添加 allowPublicKeyRetrieval=true 到 jdbc 连接串中

- 详解: https://blog.csdn.net/qq_41287877/article/details/89818095

## 部署步骤与 docker 配置

### 部署

1. 使用 maven 打包父项目
2. 运行 `docker-compose up -d` 

### 配置源码

#### docker-compose.yml
```yaml
version: "3.8"

services:
  learn1-nacos:
    container_name: learn1-nacos
    hostname: learn1-nacos
    image: nacos/nacos-server:v2.1.0
    env_file:
      - ./docker/env/nacos.env
    volumes:
      - ./docker/nacos/logs/:/home/nacos/logs
#      - ./docker/nacos/custom.properties:/home/nacos/init.d/custom.properties
    ports:
      - "8848:8848"
      - "9848:9848"
      - "9555:9555"
    depends_on:
      - learn1-mysql
    restart: always

  learn1-mysql:
    container_name: learn1-mysql
    image: mysql:8.0.29
    env_file:
      - ./docker/env/mysql.env
    volumes:
      - ./docker/mysql/data/:/var/lib/mysql/
      - ./docker/mysql/initdb/:/docker-entrypoint-initdb.d/
    ports:
      - "3306:3306"
    cap_add:
      - SYS_NICE
    restart: always

  learn1-user-service:
    container_name: learn1-user-service
    build: ./user-service
    depends_on:
      - learn1-nacos
    restart: always

  learn1-order-service:
    container_name: learn1-order-service
    build: ./order-service
    depends_on:
      - learn1-nacos
    restart: always

  learn1-gateway:
    container_name: learn1-gateway
    build: ./gateway
    ports:
      - "10010:10010"
    depends_on:
      - learn1-nacos
    restart: always
```

#### .env

docker-compose.yml 中的 env_file 配置的文件

- nacos.env
    ```yaml
    PREFER_HOST_MODE=hostname
    MODE=standalone
    MYSQL_SERVICE_HOST=mysql
    MYSQL_SERVICE_DB_NAME=nacos
    MYSQL_SERVICE_PORT=3306
    MYSQL_SERVICE_USER=nacos
    MYSQL_SERVICE_PASSWORD=nacos
    MYSQL_SERVICE_DB_PARAM=characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useSSL=false
    ```
- mysql.env
  ```yaml
  MYSQL_ROOT_PASSWORD=root
    MYSQL_USER=nacos
    MYSQL_PASSWORD=nacos
  ```

#### Dockerfile

在需要部署的项目下创建 Dockerfile 文件

```yaml
FROM java:8-alpine
COPY ./target/app.jar /app/app.jar
ENTRYPOINT java -jar /app/app.jar --spring.profiles.active=prod
```

#### sql

- nacos.sql : 在官方提供的建表文件前添加数据库与用户的配置
  ```sql
  create database nacos character set utf8 collate utf8_general_ci;
  grant all on nacos.* to 'nacos'@'%';
  use nacos;
  ```
  - nacos 建表语句: https://github.com/alibaba/nacos/blob/master/distribution/conf/nacos-mysql.sql

- 其他项目 sql 见最上方的仓库地址