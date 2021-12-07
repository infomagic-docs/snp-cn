---
title: 关于命名空间的操作
id: work-with-namespaces
category: user-guides
---

Pulsar 中，租户内将命名空间作为管理单元的命名法。本文介绍如何使用 pulsarctl CLI 工具或 StreamNative 控制台为集群创建和管理命名空间。

# 使用 pulsarctl CLI 工具操作命名空间

可以使用 pulsarctl CLI 工具创建和管理命名空间。有关命名空间支持操作的完整列表，参阅 [pulsarctl 命令参考](https://docs.streamnative.io/pulsarctl/v2.7.0.7/#-em-update-em--32)。

> **注**
> 
> 本节使用名为 `example-tenant` 的租户举例。关于如何创建租户的详细内容，参见[关于租户的操作](/user-guides/admin/work-with-tenants.md)。

## 新建命名空间

创建和授权租户后，就可以创建和管理命名空间和主题。 

如下示例介绍如何为 `example-tenant` 创建一个名为 `example-ns` 的命名空间。 

**输入**

```
pulsarctl namespaces create example-tenant/example-ns
```

**输出**

```
Created example-tenant/example-ns successfully
```

## 清空命名空间 backlog

Pulsar 将所有未确认的消息存储在 backlog 中，直到它们被处理和确认。可以清除特定命名空间的 backlog，以释放更多命名空间的存储配额。 

如下示例介绍如何清除 `example-tenant/example-ns`的所有主题的 backlog。 

**输入**

```
pulsarctl namespaces clear-backlog example-tenant/example-ns
```

**输出**

```
Are you sure you want to clear the backlog? (Y or N)

y

Successfully clear backlog for all topics of the example-tenant/example-ns
```

## 卸载命名空间 

如下示例显示如何从当前服务代理卸载 `example-tenant/example-ns`。 

**输入**

```
pulsarctl namespaces unload example-tenant/example-ns
```

**输出**

```
Unload namespace example-tenant/example-ns successfully
```

## 删除命名空间 

> **注**
> 
> 只有在没有资源和命名空间关联时，方能将命名空间删除。

如下示例介绍如何删除 `example-tenant/example-ns`。

**输入**

```
pulsarctl namespaces delete example-tenant/example-ns
```

**输出**

```
Deleted example-tenant/example-ns successfully
```

# 使用 StreamNative 控制台操作命名空间

本节介绍如何使用 StreamNative 控制台创建和管理命名空间。使用 StreamNative 控制台创建和管理命名空间之前，首先需要创建一个租户。详细信息参见[关于租户的操作](/user-guides/admin/work-with-tenants.md)。

## 新建命名空间

按照如下步骤新建命名空间。 

1. 从左侧导航窗格中，单击**命名空间**。 

2. 点击**新建命名空间**。出现一个对话框。

   ![](../../image/create-namespace.png)

3. 输入命名空间的名称，然后点击**确认**。命名空间名称为最多 40 个字符的字符串，支持小写字母 (a-z)、数字字符 (0-9) 和特殊字符连字符 (-)。

## 查看命名空间统计信息 

按照如下步骤操作，查看命名空间的统计信息。

1. 从左侧导航窗格中，单击**命名空间**。

2. 选择**概览**选项卡。

3. 分别从下拉列表中选择目标租户和命名空间。下表列出了有关命名空间的统计信息。

   ![](../../image/check-ns-1.png)

    <table>
    <tr>
    <td>
    项目
    </td>
    <td>描述
    </td>
    </tr>
    <tr>
    <td>In Rate
    </td>
    <td>命名空间的入口速率。 
    </td>
    </tr>
    <tr>
    <td>Out Rate
    </td>
    <td>命名空间的出口速率。
    </td>
    </tr>
    <tr>
    <td>In Throughput 
    </td>
    <td>命名空间的入口通量。
    </td>
    </tr>
    <tr>
    <td>Out Throughput
    </td>
    <td>命名空间的出口通量。
    </td>
    </tr>
    </table>

4. 选择**主题**选项卡，检查可用于命名空间的主题数量和有关主题的统计信息。此外还可以创建新主题并更新特定主题。

## 卸载命名空间 bundle

为了方便分配，命名空间被划分成一个 bundle 列表，每个 bundle 包含命名空间整体哈希范围的一部分。默认情况下，每个命名空间支持四个 bundle。

按照以下步骤操作卸载命名空间的 bundle。

1. From the left navigation pane, click **Namespaces**.

2. Select the **OVERVIEW** tab.

3. Select the target tenant and namespace from the drop-down lists respectively.

4. At the **Bundle** section, select the target cluster from the **Cluster** drop-down list, and then click **Unload All** to unload all namespace bundles from the cluster.

5. At the **Bundle** section, click **Unload** next to a specific bundle to unload the bundle for the namespace.

## Split namespace bundles

Since the load for topics in a bundle might change over time, one bundle can be split in two bundles by brokers. Then, the new smaller bundles are reassigned to different brokers. By default, the newly split bundles are immediately offloaded to other brokers to facilitate the traffic distribution.

To split bundles for a namespace, follow these steps.

1. From the left navigation pane, click **Namespaces**.

2. Select the **OVERVIEW** tab.

3. Select the target tenant and namespace from the drop-down lists respectively.

4. At the **Bundle** section, click **Split** next to a specific bundle to split the bundle for the namespace.

## Clear namespace backlog

You can clear backlog for bundles of a namespace.

1. From the left navigation pane, click **Namespaces**.

2. Select the **OVERVIEW** tab.

3. Select the target tenant and namespace from the drop-down lists respectively.

4. At the **Bundle** section, select the target cluster from the **Cluster** drop-down list, and then click **Clean All Backlog** to clear the backlog for all topics of the namespace.

5. At the **Bundle** section, click **Clear Backlog** next to a specific bundle to clear the backlog for all topics of the namespace bundle.

## Configure namespace policies

The policies set on a namespace apply to all the topics created in that namespace.

To configure policies for a namespace, follow these steps.

1. From the left navigation pane, click **Namespaces**.

2. Select the **POLICY** tab.

3. Select the target tenant and namespace from the drop-down lists respectively.

4. Configure policies for the namespace, as listed in the following table.

   ![](../../image/configure-ns-policy.png)

    <table>
    <tr>
    <td>
    Field
    </td>
    <td>Description
    </td>
    </tr>
    <tr>
    <td>Clusters
    </td>
    <td>Select a replication cluster. Messages of the topics in this namespace are asynchronically replicated between the configured replication clusters. In this release, the replication cluster is the same with the cluster you created because there is only one cluster available for an instance.
    </td>
    </tr>
    <tr>
    <td>Authorization
    </td>
    <td>Grant/Revoke permissions to other client roles in this namespace.
    <ul>

    <li>consume: grant/revoke the consuming action. 

    <li>produce: grant/revoke the producing action. 

    <li>functions: grant/revoke the Pulsar functions action.

    <p>
    Click <strong>Add Role</strong> to set one or more service accounts as the administrator of the namespace.
    <p>
    Click <strong>Delete Role</strong> to remove a specific service account from the administrator list of the namespace.
    </li>
    </ul>
    </td>
    </tr>
    <tr>
    <td>Subscription Authentication
    </td>
    <td>Configure the subscription authentication mode to limit the naming convention for subscriptions when authentication is enabled. 
    <ul>

    <li>None: Pulsar clients can use any allowed subscription name to connect to the cluster. 

    <li>Prefix: Pulsar clients can only use subscription names that are prefixed with their role names to subscribe messages. 

    <p>
    By default, it is set to <em>None</em>.
    </li>
    </ul>
    </td>
    </tr>
    <tr>
    <td>Storage
    </td>
    <td>Configure the storage policy for the namespace.

    **Replication Factor**: configure the storage replication settings. <ul><li> Ensemble size: configure the number of bookies to be used when creating a ledger. <li> Write Quorum Size: configure the number of replicas to be stored for each message. <li> Ack Quorum Size: configure the number of responses to wait before claiming a message is guaranteed to be stored. </ul>
   **Mark-Delete Rate**: configure the number of mark-deletion calls that are allowed for each second. When the value is set to 0, the rate limiter is disabled. By default, the value is set to 0. </p>
   **Encryption Required**: enable/disable message encryption for the namespace. <ul><li> Enabled: enable message encryption for the namespace. <li> Disabled: disable message encryption for the namespace. </ul>
   **Deduplication**: enable/disable deduplication for the namespace. <ul><li> Enabled: enable deduplication for the namespace. <li> Disabled: disable deduplication for the namespace.</ul>
    </td>
    </tr>
    <tr>
    <td>Backlog
    </td>
    <td>Configure the backlog policy for the namespace. 
   
    **Backlog Quota Size**: configure the maximum backlog quota (in bytes) allowed for the namespace. The backlog quota size must be smaller than 1000000000 bytes. By default, it is set to -1073741824 bytes.</p> **Backlog Retention Policy**: configure the backlog retention policy. <ul><li> consumer_backlog_eviction: the broker starts discarding backlog messages. <li>producer_exception: the broker disconnects with the client by giving an exception. <li>producer_request_hold: the broker holds but does not persist producer request payload.</ul>
    </td>
    </tr>
    <tr>
    <td>Schema
    </td>
    <td>Configure the schema policy for the namespace. 
    <ul>

    <li>AutoUpdate Strategy: configure the automatic update strategy of the schema for the namespace. 

    <li>Schema Validation Enforced: enable/disable schema validation for producers without schema. If it is disabled, producers without schema are disallowed to produce messages to topics with schema.
    </li>
    </ul>
    </td>
    </tr>
    <tr>
    <td>Cleanup Policy
    </td>
    <td>Configure the cleanup policy for the namespace. 
    <ul>

    <li>Message TTL (seconds): configure the message TTL in seconds. If messages are not consumed by any consumer of a subscription, they are marked as "consumed" after the configured TTL period expires. 

    <li>Retention Size (bytes): configure the retention size. This field is only applied to the messages that are acknowledged by all subscriptions. The retention size must be smaller than 1000 MB. By default, it is set to 0. 

    <li>Retention Period (minutes): configure the retention period. This field is only applied to the messages that are acknowledged by all subscriptions. By default, it is set to 0. 

    <li>Compaction Threshold (bytes): configure the threshold to compact topics. The compaction is automatically triggered when the threshold is reached. By default, it is set to 0. 

    <li>Offload Threshold (bytes): configure the threshold to offload messages. Messages are automatically offloaded to the tiered storage when the threshold is reached. By default, it is set to -1. 

    <p>
    Offload Deletion Lag (milliseconds): configure the time (in milliseconds) to wait before deleting a ledger offloaded from the BookKeeper. When the value is set to a negative value, offloading deletion is disabled.
    </li>
    </ul>
    </td>
    </tr>
    <tr>
    <td>Throttling
    </td>
    <td>Configure the throttling policy for the namespace. 
    <ul>

    <li>Max Producers Per Topic: configure the maximum number of producers allowed for each topic.

    <li>Max Consumers Per Topic: configure the maximum number of consumers allowed for each topic. 

    <li>Max Consumers Per Subscription: configure the maximum number of consumers allowed for each subscription.
    </li>
    </ul>
    </td>
    </tr>
    <tr>
    <td rowspan="3" >Dispatch Rate
    </td>
    <td>Dispatch Rate Per Topic: configure the dispatch rate based on the topic. 
    <ul>

    <li>Throughput (bytes/second): configure the dispatch rate of bytes in a topic. 

    <li>Rate (messages/second): configure the dispatch rate of messages in a topic. Time Period (seconds): configure the dispatch rate period. 
    </li>
    </ul>
    </td>
    </tr>
    <tr>
    <td>Dispatch Rate Per Subscription: configure the dispatch rate based on the subscription mode. 
    <ul>

    <li>Throughput (bytes/second): configure the dispatch rate of bytes for a subscription.

    <li>Rate (messages/second): configure the dispatch rate of messages for a subscription.

    <li>Time Period (seconds): configure the dispatch rate period. 
    </li>
    </ul>
    </td>
    </tr>
    <tr>
    <td>Dispatch Rate Per Consumer: configure the rate at which a consumer subscribes to a topic. 
    <ul>

    <li>Throughput (bytes/second): configure the throughput for a topic that a consumer subscribes to. 

    <li>Rate (messages/second): configure the rate for a topic that a consumer subscribes to.
    </li>
    </ul>
    </td>
    </tr>
    </table>

## Delete namespaces

To delete a namespace, follow these steps.

1. From the left navigation pane, click **Namespaces**.

2. Select the **POLICY** tab.

3. Select the target tenant and namespace from the drop-down lists respectively.

4. Scroll down the page and then click **Delete** **Namespace**. A dialog box is displayed, asking “_Are you sure you want to delete this?_”

   ![](../../image/delete-ns.png)

5. Enter the namespace name and then click **Confirm**.