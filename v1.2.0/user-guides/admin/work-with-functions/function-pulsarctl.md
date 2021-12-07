---
title: 使用 pulsarctl CLI 工具处理 Pulsar Function
id: function-pulsarctl
category: user-guides
---

为 Pulsar 集群创建 Pulsar Function 后，可以根据需要对其进行更新或删除这个 Function。本文介绍如何使用 pulsarctl CLI 工具更新和删除 Pulsar Function。有关 pulsarctl CLI 工具支持的操作的完整列表，请参阅 [pulsarctl 命令参考](https://docs.streamnative.io/pulsarctl/v2.7.0.7/)。

# 新建 Pulsar Function

新建集群后，可以使用 `pulsarctl functions create` 命令将 Pulsar Function 部署到集群中。
如下示例显示如何通过指定 JAR 包来为 `public` 租户和 `default` 命名空间新建 Function。 

**输入**

```
pulsarctl functions create
--tenant public
--namespace default
--name function1
--inputs test-input-topic
--output persistent://public/default/test-output-topic
--classname org.apache.pulsar.functions.api.examples.ExclamationFunction
--jar /examples/api-examples.jar

```

**输出**

```
Created function1 successfully
```

此表列出了可用于创建 Pulsar function 的所有字段。

此表列出了可用于创建 Pulsar function 的所有字段。

字段 | 描述 | 默认值 
---|---|---
auto-ack | 框架是否自动确认消息。                                       | true 
classname | Pulsar Function 的类名。 |
CPU | 每个 function 实例需要分配的内核中的 CPU 数（仅适用于 docker 运行时）。 |
custom-runtime-options | 对选项进行编码以自定义运行时的字符串。 |
custom-schema-inputs | 输入主题到 Schema 类名的映射（为 JSON 字符串）。 |
custom-serde-inputs | 输入主题到 SerDe 类名称的映射（为 JSON 字符串）。 |
dead-letter-topic | 所有没有成功处理的消息被发送到这个主题。Python 函数不支持这一参数。 |
disk | 每个 function 实例需要分配的磁盘（以字节为单位）（仅适用于 docker 运行时）。 |
fqfn | Function 的完全限定函数名称（FQFN）。 |
function-config-file | 指定 Pulsar Function 配置的 YAML 配置文件的路径。 |
go | Function 的主要 Go 可执行二进制文件的路径（如果该 function 是用 Go 编写的）。 |
inputs | Pulsar Function 的输入主题（多个主题可以表示为逗号分隔的列表）。 |
jar | Function 的 jar 文件的路径（如果 function 是用 Java 编写的）。同时还支持 URL 路径 [http/https/file（文件协议名，假定文件已经存在于工作主机上）]，worker 可以从中下载包。 |
log-topic | 生成 Pulsar Function 日志的主题。                            |
max-message-retries | 放弃前尝试处理消息的次数。                                   |
name | Pulsar Function 的名称。 |
namespace | Pulsar Function 的命名空间。 |
output | Pulsar Function 的输出主题（如未指定，则不写入输出）。       |
output-serde-classname | 用于 function 输出消息的 SerDe 类。                          |
parallelism | Pulsar Function 的并行度（即运行 function 的实例数）。 |
processing-guarantees | 应用于 function 的处理保证（传递语义）。可用值：[ATLEAST_ONCE、ATMOST_ONCE、EFFECTIVELY_ONCE]。 | ATLEAST_ONCE
py | 函数的主 Python 文件/Python Wheel 文件的路径（如 function 为使用 Python 编写）。 |
ram | 每个 function 实例需要分配的 ram 字节数（仅适用于进程/docker 运行时）。 |
retain-ordering | Function 按顺序消费和处理消息。                              |
schema-type | 用于 function 输出消息的内置 schema 类型或自定义 schema 类名。 | <empty string>
sliding-interval-count | 窗口滑动后的消息数。                                         |
sliding-interval-duration-ms | 窗口滑动的持续时间。                                         |
subs-name | 用于用户想要为输入主题消费者指定特定的订阅名称时，表示 Pulsar 源订阅名称。 |
tenant | Pulsar Function 的租户。 |
timeout-ms | 消息超时（以毫秒为单位）。 |
topics-pattern | 要从与模式匹配的命名空间下的主题列表中消费的主题模式。`--input` 和 `--topics-Pattern` 是互斥的。 <br/>在 `--customSerdeInputs` 中为模式添加 SerDe 类名（仅支持 Java 函数）。 |
user-config | 用户定义的 config key/值。 |
window-length-count | 每个窗口的消息数。 |
window-length-duration-ms | 窗口的持续时间（以毫秒为单位）。                             |

# 更新 Pulsar Function

如果要更新 Pulsar Function 的配置选项时，可以使用 `pulsarctl functions update` 命令来删除它。

* 下面的例子介绍如何为 `public` 租户下的 `default` 命名空间更新 `function1` 的输出主题。

    **输入**

    ```
    pulsarctl functions update
    --tenant public
    --namespace default
    --name function1
    --output test-output-topic
    ```

    **输出**

    ```
    Updated function1 successfully
    ```

* 下面的例子介绍如何为 `public` 租户下的 `default` 命名空间更新 `function1` 的日志主题。

    **输入**

    ```
    pulsarctl functions update
    --log-topic persistent://public/default/test-log-topic
    // other function parameters
    ```

    **输出**

    ```
    Updated function1 successfully
    ```

# 删除 Pulsar Function

如果想要删除 Function，可以使用  `pulsarctl functions delete` 命令来删除。下面的例子介绍如何从 `public` 租户下 `default` 命名空间删除 `function1`。

**输入**

```
pulsarctl functions delete
--tenant public
--namespace default
--name function1
```

**输出**

```
Deleted successfully
```
