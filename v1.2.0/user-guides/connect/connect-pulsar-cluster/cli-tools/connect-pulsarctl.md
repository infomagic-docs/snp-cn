---
title: 使用 pulsarctl CLI 工具连接到 Pulsar 集群
id: connect-pulsarctl
category: user-guides
---

`pulsarctl` 是一个 CLI 工具，用于管理 pulsar 集群。

如下示例展示了如何使用 `pulsarctl` CLI 工具连接到 Pulsar 集群，然后列出可用于该集群的租户。

```shell script
pulsarctl \
    --admin-service-url WEB_SERVICE_URL \
    --token AUTH_PARAMS \
    tenants list
```

根据[准备连接 Pulsar 集群](/user-guides/connect/connect-pulsar-cluster/connect-prepare.md)中的描述，设置 `--admin-service-url` 和 `--token` 参数。

关于如何使用 pulsarctl CLI 工具管理 Pulsar 组件的详细信息，请参见 [pulsarctl 命令参考](https://docs.streamnative.io/pulsarctl/v2.7.0.7/)。
