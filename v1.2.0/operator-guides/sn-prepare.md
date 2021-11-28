---
title: 准备 Kubernetes 集群
id: sn-prepare
category: operator-guides
---

本文档描述了为部署 StreamNative Platform 准备 Kubernetes 集群所需的任务。执行这些任务必须具有适当的 Kubernetes 集群级别权限。

# 为 StreamNative Platform 创建 Kubernetes 命名空间 

1. 创建 Kubernetes 命名空间以将 StreamNative Platform 部署到：

    ```
    kubectl create namespace KUBERNETES_NAMESPACE 
    ```

2. （可选）将新命名空间设置为当前命名空间。

    ```
    kubectl config set-context --current --namespace=KUBERNETES_NAMESPACE
    ```

将新命名空间设置为当前命名空间后，无需在后续的 `kubectl` 命令中添加 `--namespace` 标志，因为这些命令会假定当前的命名空间。