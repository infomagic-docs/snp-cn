---
title: 调节 Pulsar 集群大小
id: scale-cluster
category: operator-guides
---

本文描述了如何根据工作负载需求调整 Pulsar 集群的大小。当需求很高时，可以添加节点来扩展集群。当需求降低时，可以缩减 Pulsar 集群的规模。

可以通过增加或减少 Pulsar 组件的数量来调整 Pulsar 集群的大小，包括 Pulsar brokers、BookKeeper、ZooKeeper 和 Pulsar Proxy。大多数情况下只需要添加或删除 Pulsar broker 和 BookKeeper 节点。

# 扩展 Pulsar 集群

需要进行如下几步的操作来向 Pulsar 集群中添加新的节点： 

1. 定义每个新节点的配置。
2. 为节点提供存储、网络和计算资源。 
3. 使用定义的配置和提供的资源更新 Pulsar 集群。 
4. 跨集群重新分配流量，以便新代理共享负载。 
应当能观察到整个集群的性能有所提高。

> **注**
> 
> 默认情况下，StreamNative Platform 支持自动负载均衡。

按照如下步骤来扩展 Pulsar 集群。

1. 通过在 Pulsar CRD 文件中将 `replicaCount` 选项设置为更大的值来增加节点数。 

    ```
    broker:
    component: broker
    replicaCount: 9 # In this example, scaling up from 5
    ```

2. 使用新设置更新 Pulsar 集群。

    ```
    helm upgrade -f PULSAR_CRD_FILE PULSAR_CLUSTER PULSAR_CHART/
    ```

# 缩小 Pulsar 集群

按照如下步骤来缩小 Pulsar 集群。

1. 通过在 Pulsar CRD 文件中将 `replicaCount` 选项设置为更小的值来减少节点数。 

    ```
    broker:
    component: broker
    replicaCount: 3 # In this example, scaling down from 5
    ```

2. 使用新设置更新 Pulsar 集群。

    ```
    helm upgrade -f PULSAR_CRD_FILE PULSAR_CLUSTER PULSAR_CHART/
    ```

3. 等待适当的时间，等待流量被复制到其他现有节点。等待时间会因集群大小和主题数量而不同。

