---
title: 使用 Kafka Java 客户端连接 Pulsar 集群
id: connect-kafka-java
category: user-guides
---

StreamNative Platform 使用 Kafka on Pulsar（KoP），引入原生 Kafka 协议支持 Pulsar broker。因此可以在不修改代码的情况下，直接将现有的 Kafka 应用程序和服务迁移到 Apache Pulsar。

本例展示如何使用 Kafka Java 客户端连接到 Pulsar 集群，然后向 Pulsar 集群消费和从 Pulsar 集群消费消息。

Currently, StreamNative Platform supports [Kafka Client v1.0.0 - v2.6.0](https://github.com/streamnative/kop/tree/master/integrations#readme).

# 前提条件

- 安装 Java 1.8.0 或更高版本。 
- 安装 Istio。详情参见[为 KoP 访问安装 Istio](/operator-guides/sn-plan.md#install-istio-for-kop-access).

# 步骤

1. 向命名空间上的 Admin 角色授予生产和消费权限。 

    ```bash
    pulsarctl namespaces grant-permission --role ROLE_NAME --actions produce,consume NAMESPACE_NAME
    ```

2. 配置 [KoP SSL 连接](/operator-guides/sn-plan.md#generate-ssl-key-and-certificates-for-KoP)并启用 KoP。

    KoP 与 Pulsar broker 一起启用。为了方便使用 KoP，KoP 默认开启。可以根据 KoP SSL 连接设置以下选项。

    ```
    broker:
    # set your own domain for accessing kop outside from the cluster
    advertisedDomain: ""
    kop:
        tls:
        # create a secret with keystore.jks and truststore.jks for kop tls security
        certSecretName: "kop-secret"
        # create a secret for keystore and truststore cert
        passwordSecretRef:
            key: password
            name: kop-keystore-password
    ```

3. 获取 Istio 网关服务的外部 IP 地址。

    ```
    kubectl get svc/istio-ingressgateway -n KUBERNETES_NAMESPACE
    ```

4. 安装 Kafka 客户端。

    1. 下载 [Kafka 客户端 2.2.0](https://kafka.apache.org/downloads)。本例中使用 Kafka 2.2.0，因为从 Kafka 2.1.0 开始，生产者的 `retries` 参数的默认配置从 `0` 更改为 `2147483647`。

        ```bash
        curl -O https://archive.apache.org/dist/kafka/2.2.0/kafka_2.11-2.2.0.tgz
        ```

    2. 解压下载的包。

        ```bash
        tar -xf kafka_2.11-2.2.0.tgz
        ```

5. 准备名为 `client-ssl.properties` 的文件。该文件包含以下信息。

    ```
    security.protocol=SASL_SSL
    ssl.truststore.location=client.truststore.jks
    ssl.truststore.password=client
    ssl.endpoint.identification.algorithm=
    sasl.mechanism=PLAIN
    sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required username="USER_NAME" password="token:[YOUR_TOKEN]";
    ```

6. 运行 Kafka 客户端。

   1. 进入 Kafka 客户端目录。

       ```bash
       cd kafka_2.11-2.2.0
       ```

   2. 启动 Kafka 生产者并从 Kafka 生产者发送消息。

       ```bash
       kafka-console-producer.sh --broker-list kop_service_url:9093 --topic TOPIC_NAME --producer.config client-ssl.properties
       > message-for-kafka-client
       ```

   3. 启动 Kafka 消费者。

       ```bash
       kafka-console-consumer.sh --bootstrap-server kop_service_url:9093 --topic TOPIC_NAME --consumer.config client-ssl.properties
       ```

       同时收到消息 `message-for-kafka-client`。
