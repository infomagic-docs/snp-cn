---
title: 在 AWS 上部署 StreamNative 平台
id: deploy-snp-aws
category: operator-guides
---

亚马逊 Elastic Kubernetes Service（EKS）是 AWS 的服务，用于部署、管理和扩展 Kubernetes 的容器化应用。本文介绍了如何使用 [Terraform](https://www.terraform.io/) 在亚马逊 EKS 集群上部署 StreamNative Platform。

# 前提条件

本教程假定用户已经基本了解 Kubernetes 和 `kubectl`，但假定实现未进行任何部署。

此外，教程假定用户熟悉日常的 Terraform 计划/应用工作流程。

按照本教程在 EKS 集群上部署 StreamNative Platform 之前，需要执行以下操作：

- 安装 [Terraform](https://learn.hashicorp.com/tutorials/terraform/install-cli?in=terraform/aws-get-started)。
- 安装 [AWS CLI version 2](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html)。
- 安装 [kubectl](https://kubernetes.io/docs/tasks/tools/#kubectl)，v1.16 或更高级版本。
- 安装 [Helm](https://helm.sh/docs/intro/install/) 3.0 或更高级版本。

# 在 AWS 上安装 StreamNative Platform 

本节介绍了如何使用 Terraform 在亚马逊 EKS 集群上部署 StreamNative Platform。

1. 用预先定义好的 StreamNative Platform 定义一个 EKS 集群配置文件。 

1. [此处](https://github.com/streamnative/terraform-aws-cloud/blob/master/examples/root-example/main.tf)给出用于配置 EKS 集群的 `main.tf` 文件的示例。 

2. 获取 AWS 凭证。

    Terraform 使用 AWS 提供商与 AWS 支持的许多资源进行交互。在使用 AWS 提供商之前，必须用适当的认证凭证对其进行配置。相关详细信息，请参阅[这里](https://registry.terraform.io/providers/hashicorp/aws/latest/docs#authentication)。

3. 部署一个具有预定义 StreamNative Platform 的 EKS 集群。

   1. 在包含你自己版本的 `main.tf` 文件的目录中初始化 Terraform 模块。 

		```
		terraform init
		```

	2. 检查已规划的行动。

		```
		terraform plan
		```
	3. 应用配置。

		```
		terraform apply
		```
   
		你的终端输出应该显示计划正在运行以及将创建哪些资源。用 `yes` 确认应用。这个过程应该需要大约 15-20 分钟。 

4. 验证 StreamNative Platform 是否部署成功。

   - 验证 BookKeeper Controller Manager、Pulsar Controller Manage、ZooKeeper Controller Manage 和 Vault Controller Manage 是否正常运行。

       ```
       kubectl --kubeconfig /path/to/sn-platform-cluster-config get po -n sn-system
       ```

       **输出**
   
       ```
       NAME                                                            READY   STATUS    RESTARTS   AGE
       prometheus-operator-987955d64-4wq67                             1/1     Running   0          3d5h
       pulsar-operator-bookkeeper-controller-manager-d5d7b4668-q256l   1/1     Running   0          30h
       pulsar-operator-pulsar-controller-manager-8f7fd4569-6jh2p       1/1     Running   0          30h
       pulsar-operator-zookeeper-controller-manager-6cff55bfb7-bngtf   1/1     Running   0          30h
       vault-operator-5655985464-hjlmv                                 1/1     Running   0          3d5h
       ```
   
   - 验证 cert-manager 和 ExternalDNS 是否正常运行。
   
       ```
       kubectl --kubeconfig ~/.kube/sn-ibex-test-config get po -n kube-system
       ```
   
       **输出**
       
       ```
       NAME                                                        READY   STATUS    RESTARTS   AGE
       cert-manager-7b95979bb-7vgth                                1/1     Running   0          2d1h
       cert-manager-cainjector-69d885bf55-fczz8                    1/1     Running   0          2d1h
       cert-manager-webhook-7d8f545f9b-p5jpg                       1/1     Running   0          2d1h
       external-dns-795b8cf977-v8h6t                               1/1     Running   0          23h
       ```
   
   基于上述输出，可以了解到所有组件均运行良好，表明 StreamNative Platform 已经安装成功。

# 部署 Pulsar 集群 

按照如下步骤来部署 Pulsar 集群。

1. 添加 Pulsar Chart 目录、Pulsar 集群名称和Kubernetes命名空间的环境变量。 

    ```
    # Define your pulsar cluster name
    export RELEASE_NAME=PULSAR_CLUSTER
    # Define the k8s namespace to install the pulsar cluster
    export NAMESPACE=KUBERNETES_NAMESPACE
    ```

2. 定义一个 Pulsar 集群配置文件。

    [这里](https://github.com/streamnative/examples/blob/master/platform/values_cluster.yaml)是一个 YAML 文件的示例，用于配置 StreamNative Platform 上的 Pulsar 集群。

3. 使用 `helm install` 命令来部署一个 Pulsar 集群。 

    ```
    helm install \
    --namespace $NAMESPACE \
    $RELEASE_NAME \
    --repo https://charts.streamnative.io sn-platform \
    --values /path/to/pulsar/file.yaml \
    --set initialize=true
    --kubeconfig=/path/to/sn-platform-cluster-config
    ```

    本表列出了用于部署 Pulsar 集群的选项。 
    | 选项 | 描述 |
    | --- | --- |
    | `namespace` | Kubernetes 命名空间。 |
    | `repo` | StreamNative Platform 的 Pulsar 图表的 URL。 |
    | `values` | 用于配置 Pulsar 集群的 YAML 文件的 URL。     |
    | `kubeconfig` | 用于配置访问 EKS 集群的文件的 URL。 |

    > **注**
    >
    > 如果这是你第一次安装 Helm 图表，则必须将初始化值覆盖为 `true`（设置 `--set initialize=true` 选项）。

# 清理资源 

当你不再需要 StreamNative Platform 和 Pulsar 集群，记得要销毁所创建的任何资源。执行 `terraform destroy` 命令，并在终端上用 `yes` 确认。 

```
terraform destroy
```

> **注**
>
> 如果一些资源（如认证 ConfigMap 和命名空间）不能被删除，则可以使用 `terraform state rm` 命令从 `terraform.tfstate` 文件中将它们删除，然后再使用 `terraform destroy `命令删除所有资源。
