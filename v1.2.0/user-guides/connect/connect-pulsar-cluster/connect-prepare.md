---
title: 准备连接到 Pulsar 集群
id: connect-prepare
category: user-guides
---

连接到 Pulsar 集群前，需要获取目标 Pulsar 集群的 Web 服务 URL 和用于身份验证的 token。

* 获取 Web 服务 URL。

StreamNative Platform 不会向您的 Pulsar 集群公开任何外部 IP 地址。要获取 Web 服务 URL，必须请求 Kubernetes 集群管理员为你的 Pulsar 集群生成一个 Web 服务 URL。有关如何暴露 Web 服务 URL 的详细信息，参阅[此处](/operator-guides/configure/networking.md)。

* 获取服务帐户的 token。 

  1. 在 StreamNative 控制台的左侧导航窗格中，单击**服务账户**。 

  2. 点击 **Token** 列中的**复制**图标，将 token 复制到本地目录。 

      > **注意**
      >
      > 获取服务帐户的 token 前，需验证服务帐户是否被授权为租户和命名空间的超级用户或管理员。