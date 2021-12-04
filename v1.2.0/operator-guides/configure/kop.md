---
title: 配置 KoP
id: kop
category: operator-guides
---

Kafka on Pulsar（KoP）通过在 Pulsar broker 上引入 Kafka 协议处理程序，将本地 Apache Kafka 协议支持带入 Apache Pulsar。通过向现有的 Pulsar 集群添加 KoP 协议处理程序，可以将现有的 Kafka 应用和服务迁移到 Pulsar 上，而不需要修改代码。

如果要在 Kubernetes 集群外启用 KoP 访问，则要安装 [Istio]((https://istio.io/latest/about/service-mesh/)) 并启用 TLS。

# 安装 Istio 以实现 KoP 访问 

[Istio ](https://istio.io/latest/about/service-mesh/)是一个开源的服务网，可以透明地分层分布到现有的分布式应用上。它为服务的安全、连接和监控提供了统一的、更有效的方法。 

> **注**
> 
> StreamNative Platform 为 Istio 服务暴露 9093 端口。 

1. 下载 Istio 并切换到目标目录。 

    ```
    curl -L https://istio.io/downloadIstio | sh -
    cd istio-*
    ```

2. 下载 Istio 配置文件并安装 Istio。 

    ```
    wget https://raw.githubusercontent.com/streamnative/examples/master/platform/istio.yaml
    bin/istioctl install -y -f ./istio.yaml
    ```

# 启用 KoP

按照如下步骤启用 KoP。

1. 为 KoP 启用 TLS。详情参见[在 KoP 上启用 TLS](/operator-guides/configure/security/network-encryption/tls-proxy.md#enable-tls-on-kop-with-manuallygenerated-certificates)。

2. 在 Pulsar 配置 YAML 文件中设置以下内容。

    ```
    # The domain name for accessing KoP outside from the Kuberbetes cluster.
    advertisedDomain: "kop.sn.dev"
    kop:
      enabled: true
    istio:
      gateway:
        selector:
          # set your istio gateway label here
          istio: "ingressgateway"
    ```

3. 应用新的配置。

    ```
    helm upgrade -f /path/to/your/file.yaml CLUSTER_NAME $PULSAR_CHART/
    ```
