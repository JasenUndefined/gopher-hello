使用 Docker 安装 Kafka

## 概述

Docker 是软件行业中用于创建、打包和部署应用程序的最流行的容器引擎之一。以下主要记录使用 Docker 进行 Apache Kafka 设置。单节点 Kafka 代理设置将满足大部分本地开发需求，所以让我们从学习这个简单的安装开始。

## docker-compose.yml 配置

启动一个 Kafka 服务之前需要先启动一个 Zookeeper 服务。我们可以在 docker-compose.yml 文件中配置这个依赖，这将确保 Zookeeper 服务器总是在 Kafka 服务器之前启动并在它之后停止。

首先创建一个简单的 docker-compose.yml 文件，其中包含两个服务，即 Zookeeper 和 kafka：

```yaml
version: '3'
services:
  zookeeper:
    image: bitnami/zookeeper:latest
    environment:
      ZOO_PORT_NUMBER: 2181
      ZOO_TICK_TIME: 2000
    ports:
      - 2181:2181

  kafka:
    image: bitnami/kafka:latest
    depends_on:
      - zookeeper
    ports:
      - 9092:9092
    environment:
      KAFKA_ENABLE_KRAFT=yes
      KAFKA_CFG_PROCESS_ROLES=broker,controller
      KAFKA_CFG_CONTROLLER_LISTENER_NAMES=CONTROLLER
      KAFKA_CFG_LISTENERS=PLAINTEXT://:9092
      KAFKA_CFG_LISTENERS=PLAINTEXT://:9092,CONTROLLER://:9093
      KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP=CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT
      KAFKA_CFG_ADVERTISED_LISTENERS=PLAINTEXT://127.0.0.1:9092
      KAFKA_BROKER_ID=1
      KAFKA_CFG_CONTROLLER_QUORUM_VOTERS=1@127.0.0.1:9093
      KAFKA_CFG_ZOOKEEPER_CONNECT=zookeeper:2181
      ALLOW_PLAINTEXT_LISTENER=yes
```

在此设置中，我们的 Zookeeper 服务正在端口 2181 上监听 kafka 服务，该服务在同一容器设置中定义。 但是，对于在主机上运行的任何客户端，它将在端口 22181 上公开。 同样，kafka 服务通过端口 29092 暴露给主机应用程序，但它实际上是在由 KAFKA_ADVERTISED_LISTENERS 属性配置的容器环境中的端口 9092 上公布的。

## 相关配置说明：

Zookeeper 相关配置，仅列举部分配置，更多配置[查看](https://hub.docker.com/r/bitnami/zookeeper)：

- ZOO_PORT_NUMBER：Apache ZooKeeper 客户端端口。 默认值：2181
- ZOO_SERVER_ID：集成中服务器的 ID。 默认值：1
- ZOO_TICK_TIME：Apache ZooKeeper 用于心跳的基本时间单位（以毫秒为单位）。 默认值：2000
- ZOO_PRE_ALLOC_SIZE'：事务日志文件的块大小。 默认 65536
- ZOO_SNAPCOUNT：可以拍摄快照之前事务日志中记录的事务数（以及事务日志滚动）。 默认 100000
- ZOO_INIT_LIMIT：Apache ZooKeeper 用于限制 quorum 中的 Apache ZooKeeper 服务器必须连接到领导者的时间长度。 默认值：10

Kafka 相关配置，仅列举部分配置，更多配置[查看](https://hub.docker.com/r/bitnami/kafka)：

- ALLOW_PLAINTEXT_LISTENER：允许使用 PLAINTEXT 监听器。 默认值：否。
- KAFKA_INTER_BROKER_USER：Apache Kafka 代理间通信用户。 默认值：管理员。 默认值：user。
- KAFKA_INTER_BROKER_PASSWORD：Apache Kafka 代理间通信密码。 默认值：**bitnami**。
- KAFKA_CERTIFICATE_PASSWORD：证书的密码。 没有默认值。
- KAFKA_HEAP_OPTS：Apache Kafka 的 Java 堆大小。 默认值：-Xmx1024m -Xms1024m。
- KAFKA_ZOOKEEPER_PROTOCOL：Zookeeper 连接的身份验证协议。 允许的协议：PLAINTEXT、SASL、SSL 和 SASL_SSL。 默认值：明文。
- KAFKA_ZOOKEEPER_USER：用于 SASL 身份验证的 Apache Kafka Zookeeper 用户。 没有默认值。
- KAFKA_ZOOKEEPER_PASSWORD：用于 SASL 身份验证的 Apache Kafka Zookeeper 用户密码。 没有默认值。
- KAFKA_ZOOKEEPER_TLS_KEYSTORE_PASSWORD：Apache Kafka Zookeeper 密钥库文件密码和密钥密码。 没有默认值。
- KAFKA_ZOOKEEPER_TLS_TRUSTSTORE_PASSWORD：Apache Kafka Zookeeper 信任库文件密码。 没有默认值。

## 安装启动

接下来就是通过使用 docker-compose 命令启动容器来启动 Kafka 服务器：

```
docker-compose up -d
```

安装启动完之后就可以使用命令`nc`来验证两个服务是否都在成功启动，都在监听各自的端口。

```
$ nc -z localhost 2181
$ nc -z localhost 9092
```

此外，我们还可以在容器启动时检查详细日志并验证 Kafka 服务器是否已启动：

```
docker-compose logs kafka | grep -i started
```

这样，我们的 Kafka 设置就可以使用了。当然，我们也可以使用一些 kafka 可视化工具来新建 kafka服务的连接和管理。