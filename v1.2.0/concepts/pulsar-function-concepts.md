---
title: Pulsar Functions
id: pulsar-function-concepts
category: concepts

---

**Pulsar Function* 是轻量级计算机处理程序，具有如下特点： 

- 从一个或多个 Pulsar 主题中消费消息；
- 将用户提供的处理逻辑应用于每条消息；
- 将运行结果发布到另一个主题。

Pulsar Function 是 Pulsar 消息传递系统的计算基础。Pulsar Functions 让你能够轻松创建复杂处理逻辑，而无需部署单独的类似系统（如 [Apache Storm](http://storm.apache.org/)、 [Apache Heron](https://heron.incubator.apache.org/)，或 [Apache Flink](https://flink.apache.org/)）。

Pulsar Function 可以被描述为 [Lambda](https://aws.amazon.com/lambda/) 风格的函数，专门用于将 Pulsar 作为消息总线。

## 编程模式

Pulsar Function 提供了广泛的功能，其核心编程模型很简单。Function 从一个或多个**输入主题**接收消息。收到消息后，函数会完成以下任务。

- 对输入信息应用处理逻辑，并将输出信息写入 Pulsar 中的**输出主题**。 

- 将日志写入 **日志主题**，主要用于调试目的。 

![Pulsar Functions Programing Model](../../image/pulsar-functions-overview.png)

## 处理保证

Pulsar Function 提供了三种不同的信息传递语义，可将其应用于任何函数。 

| 传递语义             | 描述                                                         |
| -------------------- | ------------------------------------------------------------ |
| **At-most-once**     | 消息发送给 function 最多处理一次。因此消息可能不被处理。     |
| **At-least-once**    | 发送给 function 的消息被不止一次处理。因此消息可能被重复处理。 |
| **Effectively-once** | 发送给 function 的消息只被处理一次，并产生一个与之相关的输出。 |

## 支持语言

目前支持  Java、Python 和 Go 编写 Pulsar Function。详情参考[函数实例](https://github.com/streamnative/function-mesh/tree/master/config/samples)。
