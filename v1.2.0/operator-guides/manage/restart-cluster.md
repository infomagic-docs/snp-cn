---
title: 重启 Pulsar 集群
id: restart-cluster
category: operator-guides
---

可能会遇到需要重启 Pulsar 集群的情况。例如在 Kubernetes Secret 中应用证书更改或应用凭证更改。

按照如下步骤重启 Pulsar 集群。

1. 找到要重启的 Pulsar 集群对应的 StatefulSet 名称。

    ```
    kubectl get statefulset --namespace KUBERNETES_NAMESPACE
    ```

2. 使用上一步获取的 StatefulSet 名称重启 Pulsar 集群。 

    ```
    kubectl rollout restart statefulset/STATEFULSET_NAME --namespace KUBERNETES_NAMESPACE
    ```