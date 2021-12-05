---
title: 使用 StreamNative 控制台处理连接器 
id: connector-console
category: user-guides
---

# 使用 StreamNative 控制台处理连接器 

本文介绍如何通过 StreamNative 控制台来处理连接器。 

当前 StreamNative 控制台支持以下连接器： 

* AMQP1_0 [source](https://hub.streamnative.io/connectors/amqp-1-0-source/v2.7.1.1) 和 [sink](https://hub.streamnative.io/connectors/amqp-1-0-sink/v2.7.1.1) 连接器
* AWS SQS [source](https://hub.streamnative.io/connectors/sqs-source/2.7.0) 和 [sink](https://hub.streamnative.io/connectors/sqs-sink/2.7.0) 连接器
* [Cloud Storage [sink](https://hub.streamnative.io/connectors/cloud-storage-sink/2.5.1) 连接器]([https://hub.streamnative.io/connectors/cloud-storage-sink/2.5.1](https://hub.streamnative.io/connectors/cloud-storage-sink/2.5.1))
* Kinesis source 和 [sink](https://hub.streamnative.io/connectors/kinesis-sink/2.5.1) 连接器 [source](https://hub.streamnative.io/connectors/kinesis-source/2.5.1) 和 [sink]([https://hub.streamnative.io/connectors/kinesis-sink/2.5.1](https://hub.streamnative.io/connectors/kinesis-sink/2.5.1))
* 数据发生器的 source 和 sink 连接器（仅用于内部测试）。

>**注**
>
> 目前，Cloud Storage sink 连接器仅适用于 AWS 平台。 

# 新建连接器 

本节介绍如何新建 source 连接器和 sink 连接器。 

## 新建 source 连接器

按照如下步骤操作新建 source 连接器。 

1. 从左侧导航窗格中，点击 **连接器** > **Source**。

2. 点击想要新建的 source 连接器类型的图标，进入 source 连接器的配置页面。

   ![](../../image/create-source.png)

3. 如下表所列，配置有关 source 连接器的选项。

<table>
  <tr>
   <td>
字段
   </td>
   <td>描述
   </td>
  </tr>
  <tr>
   <td>Source 类别
   </td>
   <td>（只读）Source 连接器的类型。
   </td>
  </tr>
  <tr>
   <td>Source 名
   </td>
   <td>为 source 连接器输入一个名称。它是一串字符，支持小写字符、数字字符和特殊字符连字符（-）
   </td>
  </tr>
  <tr>
   <td>租户
   </td>
   <td>为 source 连接器选择租户。
   </td>
  </tr>
  <tr>
   <td>命名空间
   </td>
   <td>为 source 连接器选择命名空间。 
   </td>
  </tr>
  <tr>
   <td>输出主题
   </td>
   <td>选择接收信息的 Pulsar 主题。 
   </td>
  </tr>
  <tr>
   <td>复制数
   </td>
   <td>设置运行 source 连接器的实例数量。 
   </td>
  </tr>
  <tr>
   <td>启用自动调节大小
   </td>
   <td>启用或禁用自动缩放功能。如果启用，则应该设置运行 source 连接器的实例的最大数量。实例的最大数量必须大于 "Replicas" 的值，并且等于或小于 "10"。
   </td>
  </tr>
  <tr>
   <td>Config
   </td>
   <td>（可选）配置 source 连接器。有关所支持的 source 连接器的配置的详细描述，参阅[StreamNative Hub](<a href="https://hub.streamnative.io/">https://hub.streamnative.io/</a>)。如果你不对 source 连接器进行配置，则连接器将按照默认配置创建。 
   </td>
  </tr>
</table>


4. 点击**新建**。

## 新建 sink 连接器

按照如下步骤操作新建 sink 连接器。

1. 从左侧导航窗格中，点击 **连接器** > **Sink**。

2. 点击想要新建的 sink 连接器类型的图标，进入 sink 连接器的配置页面。

   ![](../../image/create-sink.png)

3. 如下表所列，配置有关 sink 连接器的选项。

<table>
  <tr>
   <td>
字段
   </td>
   <td>描述
   </td>
  </tr>
  <tr>
   <td>Sink 类别
   </td>
   <td>（只读）Sink 连接器的类型。
   </td>
  </tr>
  <tr>
   <td>Sink 名
   </td>
   <td>输入 sink 连接器的名称。  
   </td>
  </tr>
  <tr>
   <td>租户
   </td>
   <td>为 sink 连接器选择租户。
   </td>
  </tr>
  <tr>
   <td>命名空间
   </td>
   <td>为 sink 连接器选择命名空间。 
   </td>
  </tr>
  <tr>
   <td>输入主题
   </td>
   <td>点击<strong>添加</strong>，添加更多输出消息的 Pulsar 主题。
   </td>
  </tr>
  <tr>
   <td>复制数
   </td>
   <td>设置运行 sink 连接器的实例数量。
   </td>
  </tr>
  <tr>
   <td>启用自动调节大小
   </td>
   <td>启用或禁用自动缩放功能。如果启用，则应该设置运行 sink 连接器的实例的最大数量。实例的最大数量必须大于 "Replicas" 的值，并且等于或小于 "10"。
   </td>
  </tr>
  <tr>
   <td>Config
   </td>
   <td>（可选）配置 sink 连接器。有关所支持的 sink 连接器的配置的详细描述，参阅[StreamNative Hub](<a href="https://hub.streamnative.io/">https://hub.streamnative.io/</a>)。如果你不对 sink 连接器进行配置，则连接器将按照默认配置创建。
   </td>
  </tr>
</table>


4. 点击**新建**。

# 更新连接器 

新建连接器后，你可以对它的一些字段进行更新。本节介绍如何对 source 和 sink 连接器进行更新。 

## 更新 source 连接器

按照如下步骤更新 source 连接器。 

1. 从左侧导航窗格中，点击 **连接器** > **Source**。

2. 点击目标 source 连接器，可以查看 source 连接器相关信息。 

3. Update the following items about the source connector. 

    * Tenant: configure the target tenant for the source connector.
    * Namespace: configure the target namespace for the source connector.
    * Output Topic: update the Pulsar topic into which the messages are ingested.
    * Replicas: update the number of instances for running the source connector.
    * Auto-scaling: when auto-scaling is enabled, update the maximum number of the instances for running the source connector.
    * Config: update configurations about the source connector.
4. Click **Update**.

## Update sink connectors

To update a sink connector, follow these steps.

1. From the left navigation pane, click **Connector** > **Sink**. 
2. Click the target sink connector and you can check information about the sink connector.
3. Update the following items about the sink connector.
    * Tenant: configure the target tenant for the sink connector.
    * Namespace: configure the target namespace for the sink connector.
    * Input Topic: 
        * Click **Add** to add one or more Pulsar topics for the sink connector. 
        * Click **Delete** to delete one or more Pulsar topics for the sink connector.
    * Replicas: update the number of instances for running the sink connector.
    * Auto-scaling: when auto-scaling is enabled, update the maximum number of the instances for running the sink connector.
    * Config: update configurations about the sink connector.
4. Click **Update**.

# Clone connectors

If you want to create a new connector without modifying too many configurations, you can clone an existing connector and then update some of the configurations based on your requirements.

## Clone source connectors

To clone a source connector, follow these steps.

1. From the left navigation pane, click **Connector** > **Source**. 
2. Click the target source connector.
3. Scroll down the page and then click **Clone**. A page is displayed, where you can configure the source connector based on your requirements.

   ![](../../image/clone-source.png)

<table>
  <tr>
   <td>
Field
   </td>
   <td>Description
   </td>
  </tr>
  <tr>
   <td>Source Name
   </td>
   <td>Enter a name for the source connector. It is a string of characters, supporting lowercase characters, numeric characters, and the special character hyphen (-)
   </td>
  </tr>
  <tr>
   <td>Tenant
   </td>
   <td>Select the tenant for the source connector.
   </td>
  </tr>
  <tr>
   <td>Namespace
   </td>
   <td>Select the namespace for the source connector.
   </td>
  </tr>
  <tr>
   <td>Output Topic
   </td>
   <td>Select the Pulsar topic into which the messages are ingested.
   </td>
  </tr>
  <tr>
   <td>Replicas
   </td>
   <td>Set the number of instances for running the source connector.
   </td>
  </tr>
  <tr>
   <td>Enable auto scaling
   </td>
   <td>Enable or disable auto-scaling. If enabled, you should set the maximum number of the instances for running the source connector. The maximum number of the instances must be greater than the value of the `Replicas` and equal to or smaller than `10`.
   </td>
  </tr>
  <tr>
   <td>Config
   </td>
   <td>(Optional) Configure the source connector. For detailed descriptions about configurations of the supported source connectors, see [StreamNative Hub](<a href="https://hub.streamnative.io/">https://hub.streamnative.io/</a>). If you do not configure the source connector, the connector is created with default configurations. 
   </td>
  </tr>
</table>

4. Click **Create**.

## Clone sink connectors

To clone a sink connector, follow these steps.

1. From the left navigation pane, click **Connector** > **Sink**. 
2. Click the target sink connector.
3. Scroll down the page and then click **Clone**. A page is displayed, where you can configure the connector based on your requirements.

   ![](../../image/clone-sink.png)


<table>
  <tr>
   <td>Field
   </td>
   <td>Description
   </td>
  </tr>
  <tr>
   <td>Sink Name
   </td>
   <td>Enter a name for the sink connector.
   </td>
  </tr>
  <tr>
   <td>Tenant
   </td>
   <td>Select the tenant for the source connector.
   </td>
  </tr>
  <tr>
   <td>Namespace
   </td>
   <td>Select the namespace for the source connector.
   </td>
  </tr>
  <tr>
   <td>Input Topic
   </td>
   <td>
<ul>

<li>Click <strong>Add</strong> to add one or more Pulsar topics for the sink connector.

<li>Click <strong>Delete</strong> to delete one or more Pulsar topics for the sink connector.
</li>
</ul>
   </td>
  </tr>
  <tr>
   <td>Replicas
   </td>
   <td>Configure the number of instances for running the sink connector.
   </td>
  </tr>
  <tr>
   <td>Enable auto scaling
   </td>
   <td>Enable or disable auto-scaling. If enabled, you should set the maximum number of the instances for running the sink connector. The maximum number of the instances must be greater than the value of the `Replicas` and equal to or smaller than `10`.
   </td>
  </tr>
  <tr>
   <td>Config
   </td>
   <td>(Optional) Configure the sink connector. For detailed descriptions about configurations of the supported sink connectors, see [StreamNative Hub](<a href="https://hub.streamnative.io/">https://hub.streamnative.io/</a>). If you do not configure the source connector, the connector is created with default configurations. 
   </td>
  </tr>
</table>

4. Click **Create**.

# Delete connectors

This section describes how to delete source and sink connectors.

## Delete source connectors

To delete a source connector, follow these steps.

1. From the left navigation pane, click **Connector** > **Source**. 
2. Click the target source connector and then scroll down the page.
3. Click **Delete Source**. 
  
   ![](../../image/delete-source.png)

4. Enter the source connector name and then click **Confirm**.

## Delete sink connectors

To delete a sink connector, follow these steps.

1. From the left navigation pane, click **Connector** > **Sink**. 
2. Click the target sink connector and then scroll down the page. 
3. Click **Delete Sink**. A dialog box is displayed.

   ![](../../image/delete-sink.png)

4. Enter the sink connector name and then click **Confirm**.

