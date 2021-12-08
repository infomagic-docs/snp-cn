---
title: 使用 pulsar-client CLI 工具连接到 Pulsar 集群 
id: connect-pulsar-client
category: user-guides
---

`pulsar-client` 是一个 CLI 工具，用于生成和使用 pulsar 主题的消息。

如下示例说明如何使用 pulsar-client CLI 工具连接到 Pulsar 集群，并为 `test-topic` 主题生成 10 条消息。

    ```shell script
    ./bin/pulsar-client \
        --url SERVICE_URL \
        --auth-plugin org.apache.pulsar.client.impl.auth.AuthenticationToken \
        --auth-params token:AUTH_PARAMS \
        produce test-topic -m "test-message" -n 10
    ```

根据[准备连接到 Pulsar 集群](/user-guides/connect/connect-pulsar-cluster/connect-prepare.md)中的描述设置 `--url` 和 `--auth-params` 参数。
