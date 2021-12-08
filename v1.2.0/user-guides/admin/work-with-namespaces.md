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
    <td>命名空间的入口吞吐量。
    </td>
    </tr>
    <tr>
    <td>Out Throughput
    </td>
    <td>命名空间的出口吞吐量。
    </td>
    </tr>
    </table>

4. 选择**主题**选项卡，检查可用于命名空间的主题数量和有关主题的统计信息。此外还可以创建新主题并更新特定主题。

## 卸载命名空间的 bundle

为了方便分配，命名空间被划分成一个 bundle 列表，每个 bundle 包含命名空间整体哈希范围的一部分。默认情况下，每个命名空间支持四个 bundle。

按照以下步骤操作卸载命名空间的 bundle。

1. 在左侧导航窗格中，点击**命名空间**。 
2. 选择**概览**选项卡。
3. 分别从下拉列表中选择目标租户和命名空间。
4. 在 **Bundle** 部分，从**集群**下拉列表中选择目标集群，然后点击**全部卸载（Unload All）**，即可从集群中卸载所有命名空间包。

5. 在 **Bundle** 部分，点击特定包旁边的**卸载（Unload）**以卸载命名空间的 bundle。 

## 拆分命名空间的 bundle 

由于 bundle 中主题的负载可能会随时间变化，因此 broker 可以将一个 bundle 拆分成两个 bundle。 然后新的较小的 bundle 被重新分配给不同的 broker。默认情况下，新拆分的 bundle 会立即卸载给其他的 broker 以促进流量分配。

按照如下步骤为拆分命名空间的 bundle。

1. 在左侧导航窗格中，点击**命名空间**。 
2. 选择**概览**选项卡。
3. 分别从下拉列表中选择目标租户和命名空间。
4. 在 **Bundle** 部分，点击特定包旁边的**拆分（Split）**以拆分命名空间的 bundle。

## 清空命名空间 backlog

用户可以清空命名空间中 bundle 的 backlog。 

1. 在左侧导航窗格中，点击**命名空间**。 
2. 选择**概览**选项卡。
3. 分别从下拉列表中选择目标租户和命名空间。
4. 在 **Bundle** 部分，从**集群**下拉列表中选择目标集群，点击**清空所有 Backlog（Clean All Backlog）**来清空命名空间中所有主题的 backlog。  
5. 在 **Bundle** 部分，点击特定包旁边的**清空 Backlog（Clear Backlog）**，清空命名空间 bundle 中的所有主题的 backlog。

## 配置命名空间策略 

在命名空间上设置的策略适用于在该命名空间中创建的所有主题。 

按照如下步骤操作来配置命名空间的策略。 

1. 在左侧导航窗格中，点击**命名空间**。 

2. 选择**策略**选项卡。

3. 分别从下拉列表中选择目标租户和命名空间。

4. 如下表所列，为命名空间配置策略。

   ![](../../image/configure-ns-policy.png)

    <table>
    <tr>
    <td>
    字段
    </td>
    <td>描述
    </td>
    </tr>
    <tr>
    <td>集群
    </td>
    <td>选择复制集群。此命名空间中的主题消息在配置的复制集群之间异步复制。此版本中，复制集群与您创建的集群相同，因为一个实例只有一个集群可用。
    </td>
    </tr>
    <tr>
    <td>授权
    </td>
    <td>授予/撤消此命名空间中其他客户端角色的权限。 
    <ul>


    <li>消费：授予/撤销消费操作。 

    <li>生产：授予/撤销生产操作。 

    <li>functions：授予/撤销 Pulsar function操作。

    <p>
    点击<strong>添加角色</strong>将一个或多个服务账号添加为命名空间的管理员。
    <p>
    点击<strong>删除角色</strong>将某个特定的服务账号从命名空间的管理员列表中删除。
    </li>
    </ul>
    </td>
    </tr>
    <tr>
    <td>订阅认证
    </td>
    <td>当启用认证时，配置订阅认证模式以限制订阅的命名约定。 
    <ul>


    <li>None：Pulsar 客户端可以使用任何允许的订阅名称连接到集群。 

    <li>前缀：Pulsar 客户端只能使用以角色名称为前缀的订阅名称来订阅消息。

    <p>
    默认情况下设置为<em>None</em>。
    </li>
    </ul>
    </td>
    </tr>
    <tr>
    <td>储存
    </td>
    <td>配置命名空间的储存策略。 

    **副本因子**：配置存储复制设置。<ul><li> Ensemble size：配置创建 ledger 时要使用的 bookie 的数量。<li> Write Quorum Size：配置要为每条消息存储的副本数。<li> Ack Quorum Size：配置在声明消息保证被存储之前等待的响应数量。</ul>
   **Mark-Delete Rate**：配置每秒允许的 mark-deletion 的调用次数。当该值设置为 0 时，速率限制器被禁用。默认情况下该值设置为 0。 </p>
   **需要加密**：为命名空间启用/禁用消息加密。<ul><li> 启用：为命名空间启用消息加密。<li> 禁用：为命名空间禁用消息加密。</ul>
   **消息去重**：为命名空间启用/禁用消息去重。<ul><li>  启用：为命名空间启用消息去重。<li>  禁用：为命名空间禁用消息去重。</ul>
    </td>
    </tr>
    <tr>
    <td>Backlog
    </td>
    <td>为命名空间配置 backlog 策略。 

    **堆积量**：配置命名空间允许的最大堆积量（以字节为单位）。堆积量必须小于 1000000000 字节。默认情况下设置为 -1073741824 字节。</p> **Backlog 保留策略**：配置 backlog 的保留策略。<ul><li> consumer_backlog_eviction：broker 开始丢弃 backlog 消息。<li>producer_exception：broker 通过给出异常与客户端断开连接。<li>producer_request_hold：broker 持有但不保留生产者请求的有效载荷。</ul>
    </td>
    </tr>
    <tr>
    <td>Schema
    </td>
    <td>为命名空间配置 schema 策略。 

    <ul>

    <li>AutoUpdate 策略：为命名空间配置 schema 的自动更新策略。

    <li>Schema 有效性：为没有 schema 的生产者启用/禁用架构验证。如果禁用，则不允许没有 schema 的生产者向具有 schema 的主题生产消息。
    </li>
    </ul>
    </td>
    </tr>
    <tr>
    <td>Cleanup 策略
    </td>
    <td>为命名空间配置 cleanup 策略。 

    <ul>

    <li>消息 TTL（秒）：配置消息 TTL（秒）。在 TTL 周期内，如果没有任何消费者消费该消息，当 TTL 周期结束后，该消息被标记为“已消费”。

    <li>保留大小（字节）：配置保留大小。该字段仅适用于所有订阅确认的消息。保留大小必须小于 1000 MB。 默认情况下设置为 0。

    <li>保留周期（分钟）：配置保留周期。该字段仅适用于所有订阅确认的消息。默认情况下设置为 0。 

    <li>压缩阈值（字节）：配置压缩主题的阈值。当达到阈值时自动触发压缩。默认情况下设置为 0。 

    <li>卸载阈值（字节）：配置阈值以卸载消息。当达到阈值时，消息会自动卸载到分层存储。默认情况下设置为 -1。 

    <p>
    卸载删除延迟（毫秒）：配置删除从 BookKeeper 卸载 ledger 前等待的时间（以毫秒为单位）。当该值设置为负值时，禁用卸载删除。
    </li>
    </ul>
    </td>
    </tr>
    <tr>
    <td>限流
    </td>
    <td>配置命名空间的限流策略。  
    <ul>


    <li>最大生产者数量（基于主题）：配置每个主题支持的最大生产者数量。 

    <li>最大消费者数量（基于主题）：配置每个主题支持的最大消费者数量。 

    <li>最大消费者数量（基于订阅）：每个订阅支持的最大消费者的数量。
    </li>
    </ul>
    </td>
    </tr>
    <tr>
    <td rowspan="3" >派发速率
    </td>
    <td>派发速率（基于主题）：配置基于主题的派发速率。  

    <ul>

    <li>吞吐量（字节/秒）：配置主题中的派发速率（单位为字节）。

    <li>速率（消息/秒）：配置主题中消息的派发速率。时间周期（秒）：配置调度速率周期。
    </li>
    </ul>
    </td>
    </tr>
    <tr>
    <td>派发速率（基于订阅）：根据订阅模式配置派发速率。 

    <ul>

    <li>吞吐量（字节/秒）：配置订阅的派发速率（单位为字节）。

    <li>速率（消息/秒）：配置订阅消息的派发速率。 

    <li>时间周期（秒）：配置派发速率周期。 
    </li>
    </ul>
    </td>
    </tr>
    <tr>
    <td>派发速率（基于消费者）：配置消费者订阅主题的速率。 

    <ul>

    <li>吞吐量（字节/秒）：配置消费者订阅的主题的吞吐量。

    <li>速率（消息/秒）：配置消费者订阅的主题的速率。
    </li>
    </ul>
    </td>
    </tr>
    </table>

## 删除命名空间 

按照如下步骤操作删除命名空间。 

1. 在左侧导航窗格中，点击**命名空间**。 

2. 选择**策略**选项卡。

3. 分别从下拉列表中选择目标租户和命名空间。

4. 向下滚动页面，然后点击**删除命名空间**。出现对话框显示：“您是否确认希望删除？”

   ![](../../image/delete-ns.png)

5. 输入命名空间的名称，然后单击**确定**。