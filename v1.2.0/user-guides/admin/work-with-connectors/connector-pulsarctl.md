---
title: 使用 pulsarctl CLI 工具处理连接器 
id: connector-pulsarctl
category: user-guides
---

将消息系统与数据库和其他消息系统等外部系统一起使用时，才能最大程度地发挥消息系统的作用。使用 **Pulsar 连接器**，你可以轻松创建、部署和管理与外部系统交互的连接器。

本文介绍如何使用 pulsarctl CLI 工具新建、更新和删除连接器。有关 pulsarctl CLI 工具支持的操作的完整列表，参见[此处](https://docs.streamnative.io/pulsarctl/v2.7.0.7/)。

> **注**
>
> 在使用连接器之前，确保命名空间已被给予 `function`  权限。

# 新建连接器 

本节描述如何新建 source 和 sink 连接器。 

## 新建 source 连接器

Source 连接器将数据从外部系统抓取到 Pulsar 主题。 下面的例子说明如何使用 pulsarctl CLI 工具创建 source 连接器。

1. 使用 pulsarctl  CLI 工具，连接到目标集群。详细信息参见[此处](/user-guides/connect/connect-pulsar-cluster/cli-tools/connect-pulsarctl.md)。

2. 通过指定特定字段创建连接器。如需了解所有支持的字段，可参见[source and sink 配置支持选项](#supported-source-and-sink-configuration-options)。

    本例介绍如何新建一个名为 `activemq-source` 的 source 连接器。
    
    ```
    pulsarctl sources create
    --tenant public
    --namespace default
    --name activemq-source
    --destination-topic-name activemq-source-topic
    --classname org.apache.pulsar.io.activemq.ActiveMQSource
    --source-config-file activemq-source.yaml
    --parallelism 1
    # other source configurations
    ```

3. 验证该 source 连接器是否被成功创建。 

    **输入**
    
    ```
    pulsarctl source list
    --tenant public
    --namespace default
    ```
    
    **输出**
    
    ```
    +--------------------+
    | Source Name |
    +--------------------+
    | activemq-source |
    +--------------------+
    ```
    
    如输出所示，source 连接器是在 `public` 租户和 `default` 命名空间下创建的。 

## 新建 sink 连接器

Sink 连接器将 Pulsar 主题中的数据导出到外部系统。如下示例描述如何创建 ActiveMQ sink 连接器。

1. 使用 pulsarctl CLI 工具连接到目标集群。详细信息参见[此处](/user-guides/connect/connect-pulsar-cluster/cli-tools/connect-pulsarctl.md)。

2. 通过指定特定字段创建 sink 连接器。如需查看所有支持的字段，参见[source and sink 配置支持选项](#supported-source-and-sink-configuration-options)。 

    如下示例介绍如何新建一个名为 `activemq-sink` 的 sink 连接器。

    ```
    pulsarctl sink create
    --tenant public
    --namespace default
    --name (name of the sink connector)
    --inputs test-activemq
    --sink-config-file activemq-sink.yaml
    --parallelism 1
    # other sink configurations
    ```

3. 验证该 sink 连接器是否被成功创建。

    **输入**

    ```
    pulsarctl source list
    --tenant public
    --namespace default
    ```

    **输出**

    ```
    +--------------------+
    | Sink Name |
    +--------------------+
    | activemq-sink |
    +--------------------+
    ```

    如输出所示，sink 连接器是在 `public` 租户和 `default` 命名空间下创建的。

# 更新连接器 

如果想要修改 sink 或 sink 连接器的配置，或者更新 source 或 source 连接器，可以对连接器进行更新。

如下示例显示如何将 activemq-sink 连接器的并行度更新为 2。 

**输入**

```
pulsarctl sinks update \
--name activemq-sink \
--parallelism 2
```

**输出**

```
Updated successfully
```

# 删除连接器 

如果想要从租户或者命名空间中移除连接器，可以使用 pulsarctl CLI 工具来删除连接器。

如下示例显示如何删除 `activemq-sink` sink 连接器。

**输入**

```
pulsarctl sinks delete \
--tenant public \
--namespace default \
--name activemq-sink
```

**输出**

```
"Deleted successfully"
```

可以使用 `pulsarctl sinks get` 命令来验证接收器连接器是否已被成功删除。

**输入**

```
pulsarctl sinks get \
--tenant public \
--namespace default \
--name activemq-sink
```

**输出**

```
HTTP 404 Not Found

Reason: Sink activemq-sink doesn't exist
```
这样的结果就表明 sink 连接器已不存在。 

# Source 和 sink 配置支持选项  

本节介绍 source 和 sink 连接器支持的所有配置选项。 

## Source 配置选项 

下表列出新建 source 连接器的所有字段。 

|字段|描述|
|----|---|
| `-a`, `--archive` | Source 的 NAR 存档的路径。 <br/>同样支持 url-path（http/https/file [文件协议名，假定文件已经存在于工作主机上]），工作人员可以从该路径下载包。 |
| `--classname` | 对于 `archive` 是 file-url-path (file://) 的情况下，为 source 的类名。 |
| `--cpu` | 需要为每个 source 的实例分配的 CPU（以核为单位）（仅适用于 Docker 运行时）。 |
| `--deserialization-classname` | Source 的 SerDe 类名。 |
| `--destination-topic-name` | 作为数据发送对象的 Pulsar 主题。 |
| `--disk` | 需要为每个 source 实例分配的磁盘数（以字节为单位）（仅适用于 Docker 运行时）。 |
|`--name` | Source 的名称。 |
| `--namespace` | Source 的命名空间。 |
| ` --parallelism` | Source 的并行度因子，即运行 source 的实例的数量。 |
| `--processing-guarantees` | 应用于 source 的处理保证（也称为传递语义）。Source 连接器从外部系统接收消息并将消息写入 Pulsar 主题。`--processing-guarantees` 确保将消息写入 Pulsar 主题的处理保证。 <br/>可用的值有 ATLEAST_ONCE、ATMOST_ONCE、EFFECTIVELY_ONCE。 |
| `--ram` | 需要为每个 source 实例分配的 RAM（以字节为单位）（仅适用于进程和 Docker 运行时）。 |
| `-st`, `--schema-type` | Schema 的类型。<br>用于编码从 source 发出的消息的内置 schema（例如，AVRO 和 JSON）或自定义 schema 类名称。 |
| `--source-config` | Source config key/值。 |
| `--source-config-file` | 指定 source 配置的 YAML 配置文件的路径。 |
| `-t`, `--source-type` | Source 连接器的供应方。 |
| `--tenant` | Source 的租户。 |
|`--producer-config`| 自定义生产者配置项（为 JSON 字符串） |

## Sink 配置选项

下表列出新建 sink 连接器的所有字段。

|字段|描述|
|----|---|
| `-a`, `--archive` | Sink 的存档的路径。 <br/>同样支持 url-path（http/https/file [文件协议名，假定文件已经存在于工作主机上]），worker 可以从该路径下载包。 |
| `--auto-ack` | 框架是否会自动确认消息。 |
| `--classname` | 在 `archive` 为 file-url-path (file://) 的情况下，为 sink 的类名。 |
| `--cpu` | 需要为每个 sink 的实例分配的 CPU（以核为单位）（仅适用于 Docker 运行时）。 |
| `--custom-schema-inputs` | 输入主题到 schema 类别或类名称的映射（为 JSON 字符串）。 |
| `--custom-serde-inputs` | 输入主题到 SerDe 类名称的映射（为 JSON 字符串）。 |
| `--disk` | 需要为每个 sink 实例分配的磁盘（以字节为单位）（仅适用于 Docker 运行时）。 |
|`-i, --inputs` | Sink 的输入主题或主题（多个主题可以表示为以逗号分隔的列表）。 |
|`--name` | Sink 的名称。 |
| `--namespace` | Sink 的命名空间。 |
| ` --parallelism` | Sink 的并行度因子，即运行 sink 的实例的数量。                |
| `--processing-guarantees` | 应用于 sink 的处理保证（也称为传递语义）。`--processing-guarantees` 在 Pulsar 上的实现也依赖于 sink 的实现。<br/>可用的值有 ATLEAST_ONCE、ATMOST_ONCE、EFFECTIVELY_ONCE。 |
| `--ram` | 需要为每个 sink 实例分配的 RAM（以字节为单位）（仅适用于进程和 Docker 运行时）。 |
| `--retain-ordering` | Sink 按顺序消费和汇入信息。 |
| `--sink-config` | Sink config key/值。 |
| `--sink-config-file` | 指定 sink 配置的 YAML 配置文件的路径。 |
| `-t`, `--sink-type` | Sink 连接器的供应方。当前内置连接器的 `sink-type` 参数由 pulsar-io.yaml 文件中指定的 `name` 参数的设置决定。 |
| `--subs-name` | 用于用户想要为输入主题消费者指定特定的订阅名称时，表示 Pulsar 源订阅名称。 |
| `--tenant` | Sink 的租户。 |
| `--timeout-ms` | 消息超时（以毫秒为单位）。 |
| `--topics-pattern` | 要从与模式匹配的命名空间下的主题列表中消费的主题模式。 <br/>`--input` 和 `--topics-Pattern` 是互斥的。 <br/>在 `--customSerdeInputs` 中为模式添加 SerDe 类名（仅支持 Java 函数）。 |