---
title: Pulsar简介
date: 2018-12-02 15:23:57
tags: [Pulsar]
categories: [Pulsar]
---

下一世代的MQ-Pulsar简述<!--more-->

### 概述

Pulsar 最初由 yahoo 开发，是一个多租户、高可用，服务间的消息系统。现在由 Apache Software Foundation 管理。

 Pulsar 的主要特性如下；

     Pular 通过跨集群的消息无缝 geo-replication，让 Pular 实例原生支持多集群
* 很低的发布和端到端延迟
* 无缝的可扩展性，乃至上百万个 topic
* 简洁的 client API，支持 Java、Python、C++
* 多种 topic 的订阅模式（独占、共享、灾备）
* 使用 Apache BookKeeper，确保传递的消息被持久存储
* serverless 轻量级计算框架 Pulsar Function，提供流原生数据处理。
* serverless connector 框架 Pulsar IO。它构建于 Pulsar Function 之上，使得从 Apache Pular 移入和移出数据更为简单
* 分层存储把过期的数据从热活存储区转移到冷 / 长期存储区（比如 S3 和 GCS）

### 消息概念

Pulsar基于 pub-sub（即发布-订阅模式），在此模式中，生产者将消息发布到主题，然后消费者可以订阅这些主题，处理传入的消息，并在处理完成时发送确认。

创建订阅后，即使消费者断开链接，Pulsar也会保留所有消息，仅当消费者确认已经成功处理后，才会丢弃保留的消息。

#### 消息

消息是Pulsar的基础单元，他们是producer向topic发布的内容，consumer从topic消费的内容，（并且会在消息处理时进行确认）

| 组成               | 用途                                                         |
| ------------------ | ------------------------------------------------------------ |
| value/data payload | 消息携带的数据，所有 pulsar 的消息携带原始 bytes，但是消息数据也需要遵循数据 shcema |
| Key                | 可以选择使用Key标记消息，这对于[topic压缩](http://pulsar.apache.org/docs/en/concepts-topic-compaction)等操作非常有用 |
| properties         | 可选 用户自定义的key/value的属性map                          |
| Producer Name      | 发布消息的Producer的名称（如果用户没有定义Producer的名称，那么会被自动赋予默认名称） |
| Sequence ID        | 在topic中，每个Pulsar消息属于一个有序的序列，消息的sequence Id是该消息在整个序列中次序或者说叫坐标 |
| Publish time       | 消息发布的时间戳(由Producer自动生成)                         |
| Event time         | 这个时间戳是可选的，为了记录当消息发生某个事件时候的事件，例如处理消息时，如果没有明确设置该项，则消息的事件事件为0 |

#### 生产者

Producer是关联到Topic，并把消息发布到Pulsar broker进行处理的程序

##### 发送模式

Pulsar的producer可以同步或者异步的发送消息

| 模式     | 描述                                                         |
| -------- | ------------------------------------------------------------ |
| 同步发送 | 发布者在发布消息后等待代理的broker的确认，如果没有收到确认，那么发布者会认为发送操作失败 |
| 异步发送 | Producer将会把消息访入blocking队列并立即返回，然后客户端类库将在后台把消息发送给代理，如果队列满了（队列最大大小可以配置），那么根据发布者传入的参数与配置，可能在调用Api时被阻止或者立即失败 |

##### 压缩

为了节省带宽，在传输过程中，producer发布的消息可以被压缩，目前pulsar支持两种压缩类型

- [LZ4](https://github.com/lz4/lz4)
- [ZLIB](https://zlib.net/)

##### 批处理

如果启用了批处理，那么发布者将在单个请求中累积并发送一批消息，批处理的大小取决于配置的最大消息数与最大发布延迟。（Kafka中也有）

#### 消费者

消费者是通过订阅关联的topic，然后接收消息并处理的程序

##### 接收模式

可以同步或者异步的从pulsar的broker中接收消息

| 模式     | 描述                                                         |
| -------- | ------------------------------------------------------------ |
| 同步接收 | 同步接收将会阻塞，直到消息可用                               |
| 异步接收 | 异步接收后会立即返回future值 --- [`CompletableFuture`](http://www.baeldung.com/java-completablefuture) ，一旦新消息可用，立即完成。 |

##### 确认

消费者成功处理了消息，需要发送确认给broker，以便让broker丢弃这条消息（否则broker将一直存储该消息）

可以逐条或累积的确认消息，当累积一起确认时候，消费者只需要确认收到的最后一条消息，所有之前（包括该消息），都不会再发送给消费者

累积确认不能与[共享订阅模式](http://pulsar.apache.org/docs/en/concepts-messaging/#subscription-modes)一起使用，因为共享模式涉及多个消费者可以访问相同的订阅。

##### 监听

客户端类库提供了对consumer的监听实现，例如Java客户端类库中提供了MessageListener  ，在该接口中，一旦接收到新的消息，received方法将被调用

#### Topic

和其他的消息队列一样，Pulsar的topic也是生产者到消费者之间传播消息的通道，Topic的命名是具有明确定义结构的url

```
{persistent|non-persistent}://tenant/namespace/topic
```

| Topic名称组成                  | 描述                                                         |
| ------------------------------ | ------------------------------------------------------------ |
| `persistent`/ `non-persistent` | 定义topic的类型，pulsar支持非持久和持久的topic， （默认是持久类型，如果你没有指明类型，topic 将会是持久类型）。持久 topic 的所有消息都会保存在硬盘上 （这意味着多块硬盘，除非是单机模式的 broker），反之，非持久 topic 的数据不会存储到硬盘上 |
| tenant                         | 实例中 topic 的租户。tenant 是 Pulsar 多租户的基本要素。可以被跨集群的传播。 |
| namespace                      | topic 的管理单元，相关 topic 组的管理机制。大多数的 topic 配置在 namespace 层面生效。 每个 tenant 可以有多个 namespace |
| topic                          | topic名称的最后组成部分，没有什么特殊的含义                  |

**不需要显式的创建 topic **

**你并不需要显式的创建 topic。如果客户端尝试从一个还不存在的 topic 写或者接受消息，pulsar 将会按在 topic 名称提供的 namnespace 下自动创建 topic。**

#### 命名空间（NameSpace）

命名空间是租户中的逻辑命名法。租户可以通过[管理 API](http://pulsar.apache.org/docs/en/admin-api-namespaces#create) 创建多个名称空间。例如，具有不同应用程序的租户可以为每个应用程序创建单独的命名空间。命名空间允许应用程序创建和管理主题层次结构。主题`my-tenant/app1`是应用程序命名空间`app1`的`my-tenant`。您可以在命名空间下创建任意数量的Topic。

#### 订阅模式

订阅是一种命名配置规则，用于确定如何把消息传递给消费者

Pulsar支持三种可用的订阅模式：exclusive（独占），shared（共享），failover （故障转移）

![](Pulsar简介\pulsar-subscription-modes.png)

##### Exclusive

在exclusive模式下，只允许一个consumer绑定到订阅上，如果多于一个消费者订阅该主题的话，消费者将会收到错误，上图中只有ConsumerA能接收到消息

**Exclusive为默认的订阅模式**

##### Shared

在Share或round robin模式下，多个消费者可以订阅同一个topic，消息通过round robin轮询机制分发给不同的消费者，并且每个消息仅且只会分发给一个消费者，当消费者断开链接时候，发送给他的并且违背确认的所有消息将会被重新安排分发给其他剩余存活的小消费者

**共享模式的局限性**

```
使用共享模式时需要注意：
* 共享模式下不能保证消息排序
* 在共享模式下不能使用累积确认
```



![](Pulsar简介\pulsar-shared-subscriptions.png)

##### 故障转移

Failover 模式中，多个 consumer 可以绑定到同一个 subscription。consumer 将会按字典顺序排序，第一个 consumer 被初始化为唯一接受消息的消费者。这个 consumer 被称为 master consumer。

当 master consumer 断开时，所有的消息（未被确认和后续进入的）将会被分发给队列中的下一个 consumer。

第一个图中，Consumer-C-1 是 master consumer，当 Consumer-C-1 断开连接时，由于 Consumer-C-2 在队列中下一个位置，那么它将会开始接收消息。

![](Pulsar简介\pulsar-failover-subscriptions.png)

#### 多主题订阅

当消费者订阅 Pulsar 主题时，默认情况下它订阅一个特定主题 。

从 Pulsar 版本 1.23.0 开始，Pulsar 消费者可以同时订阅多个主题。您可以通过两种方式定义主题列表： 

* 通过基础的正则表达式（regex），例如 persistent://public/default/finance-.*
* 通过明确定义的 topic 列表\

**通过正则订阅多主题时，所有的主题必须在同一个命名空间（namespace)**

订阅多个主题时，Pulsar 客户端将自动调用 Pulsar API 以发现与正则表达式模式 / 列表匹配的主题，然后订阅所有这些主题。如果当前不存在任何主题，则一旦创建主题，消费者将自动订阅它们。

**不能保证顺序性**

**当消费者订阅多个主题时，Pulsar 在单个主题上提供的所有顺序保证都不成立。如果您在使用 Pulsar 时涉及任何严格的顺序要求，我们强烈建议您不要使用此功能**

以下是 Java 的一些多主题订阅示例：

```
import java.util.regex.Pattern;

import org.apache.pulsar.client.api.Consumer;
import org.apache.pulsar.client.api.PulsarClient;

PulsarClient pulsarClient = // Instantiate Pulsar client object

// Subscribe to all topics in a namespace
Pattern allTopicsInNamespace = Pattern.compile("persistent://public/default/.*");
Consumer allTopicsConsumer = pulsarClient.subscribe(allTopicsInNamespace, "subscription-1");

// Subscribe to a subsets of topics in a namespace, based on regex
Pattern someTopicsInNamespace = Pattern.compile("persistent://public/default/foo.*");
Consumer someTopicsConsumer = pulsarClient.subscribe(someTopicsInNamespace, "subscription-1");
```

##### 分区Topic

通常一个 topic 仅被一个 broker 服务，这限制了 topic 最大吞吐量。分区 topic 是特殊的 topic 类型，他可以被多个 broker 处理，这让 topic 有更高的吞吐量。

 其实在背后，分区的 topic 通过 N 个内部 topic 实现，N 是分区的数量。当向分区的 topic 发送消息，每条消息被路由到其中一个 broker。Pulsar 自动处理跨 broker 的分区分布。



![](Pulsar简介\partitioning.png)

在这里，主题 **Topic1** 有五个分区（**P0** 到 **P4**），分为三个代理。因为分区比broker多，所以两个broker处理两个分区，而第三个只处理一个（同样，Pulsar 自动处理这个分区的分配）。

该主题的消息将广播给两个消费者。该[路由模式](http://pulsar.apache.org/docs/en/concepts-messaging/#routing-modes)决定了两者的broker来处理每个分区，而[订阅模式](http://pulsar.apache.org/docs/en/concepts-messaging/#subscription-modes)确定哪些消息去哪个消费者。

在大多数情况下，可以单独决定路由和订阅模式。通常，吞吐量问题应指导分区 / 路由决策，而订阅决策应由应用程序语义指导。

在订阅模式如何工作方面，分区主题和普通主题之间没有区别，因为分区仅确定生产者发布消息和消费者处理和确认消息之间发生的情况。

需要通过 [admin API](http://pulsar.apache.org/docs/en/admin-api-overview) 显式创建分区主题。创建主题时可以指定分区数。

#### 路由模式

当发布消息到分区 topic，你必须要指定路由模式。路由模式决定了每条消息被发布到的分区（其实是内部主题）。

 默认情况下有三种路由模式：

 

| 模式                     | 描述                                                         | 顺序保证           |
| ------------------------ | ------------------------------------------------------------ | ------------------ |
| Key hash                 | 如果 message 指定了 key，producer 将会把 key hash，然后把他分配给指定分区 | 同一个 key 下有序  |
| single default partition | 如果没有 key，每个生产者的消息将会被路由分发给专用的分区。初始时候随机选择 | 同一个生产者下有序 |
| round robin 分发         | 如果没有 key，所有的消息通过 round-robin 方式被路由到不同的分区，以达到最大的生产能力 | 无序               |

#### 非持久Topic

默认的，Pulsar 保存所有没有确认的消息到多个 BookKeeper 的 bookies 中（存储节点）。持久 topic 的消息数据可以在 broker 重启或者订阅者出问题的情况下存活下来。 

Pulsar 也提供了非持久 topic。非持久 topic 的消息不会被保存在硬盘上，只存活于内存中。当使用非持久 topic 分发时，杀掉 Pulsar 的 broker 或者关闭订阅者，意味着客户端可能会遭遇消息丢失。 

 非持久 topic 有如下格式的名称（注意名字中的 non-persistent）：

```
non-persistent://tenant/namespace/topic
```

非持久 topic 中，broker 会立即发布消息给所有连接的订阅者，而不会在 BookKeeper 中存储。如果有一个订阅者断开连接，broker 将无法重发这些瞬时消息。订阅者将永远也不能收到这些消息了。去掉持久化存储的步骤，在某些情况下，使得非持久 topic 的消息比持久 topic 稍微变快。但是同时，Pulsar 的一些核心优势也丧失掉了 

**非持久 topic，消息数据仅存活在内存。如果 broker 挂掉或者其他情况不能从内存取出，你的消息数据就可能会丢失。只有真的觉得你的使用场景符合，并且你可以忍受时，才可去使用非持久 topic。**

默认非持久 topic 在 broker 上是开启的，你可以通过 broker 的配置关闭。你可以通过使用 pulsar-admin-topics 接口管理非持久 topic。

##### 性能

非持久性消息传递通常比持久消息传递更快，因为broker不会持久保存消息，并且只要将消息传递给所有连接的订阅者，就立即将确认ACK消息发送回生产者。因此，非持久 topic 让 producer 有更低的发布延迟。 

 ##### 客户端api

producer 和 consumer 以连接持久 topic 同样的方式连接到非持久 topic。重要的区别是 topic 的名称必须以 non-persistent 开头。三种订阅模式 --exclusive，shared，failover 对于非持久 topic 都是支持的。 

非持久 topic 的 java consumer 

```
PulsarClient client = PulsarClient.create("pulsar://localhost:6650");
String npTopic = "non-persistent://public/default/my-topic";
String subscriptionName = "my-subscription-name";

Consumer consumer = client.subscribe(npTopic, subscriptionName);
```

以下是同一个非持久主题的 Java Producer示例：

```
Producer producer = client.createProducer(npTopic);
```

#### 消息存留和过期

Pulsar Broker的默认设置如下：

- 立即删除所有已经被 cunsumer 确认过的的消息
- 以消息 backlog 的形式，持久保存所有的未被确认消息

但是，Pulsar 有两个功能，可以覆盖此默认行为：

* 通过对消息的保存，让你可以保存concumer确认过的消息

* 消息过期机制，可以让你为尚未确认的消息设置消息存活时间（TTL）

  **消息保留与消息的TTL在Namespace级别中进行管理请参阅[消息保留和到期](http://pulsar.apache.org/docs/en/cookbooks-retention-expiry)**

  下图说明了这两种概念：

   ![](Pulsar简介\20181008112559290.png)

  ​	通过消息保留（如顶部所示），应用于命名空间中所有主题的保留策略表明，即使已经确认了某些消息，也会将这些消息持久地存储在 Pulsar 中。已删除保留策略未涵盖的已确认消息。如果没有保留策略，将删除*所有*已确认的消息。

  ​	随着消息到期，如下所示，一些消息被删除，即使它们尚未被确认，因为它们已根据应用于命名空间的 TTL 过期（例如，因为已应用了 5 分钟的 TTL 并且消息尚未确认，但已有 10 分钟）。

#### 消息去重

当消息被 Pulsar 持久化多于一次的时候，会发生数据重复。消息去重是 Pulsar 可选的特性，阻止不必要的消息重复，每条消息仅处理一次。

下图展示了开启和关闭消息去重的场景

![](Pulsar简介\message-deduplication.png)

在顶部显示的方案中禁用了消息重复数据删除。这里，生产者发布关于主题的消息 1; 消息到达 Pulsar的Broker并持久化到 BookKeeper。然后，生产者再次发送消息 1（在这种情况下由于一些重试逻辑），并且消息由broker接收并再次存储在 BookKeeper 中，这意味着发生了重复。

在底部的第二个场景中，生产者发布消息 1，该消息由broker接收并持久化，如第一个场景中那样。但是，当生产者再次尝试发布消息时，broker知道它已经接收消息 1，因此不会保留消息。

##### 生产者的幂等性

**消息去重的另外一种方法是确保每条消息仅被生产一次。这种方法通常被叫做生产者幂等**。这种方式的缺点是，把消息去重的工作推给了应用去做。在 Pulsar 中，这是被 broker 处理的，这意味着你不需要修改你的客户端代码，你只需要做一些管理上的变化（参考 [Managing message deduplication](http://pulsar.apache.org/docs/en/cookbooks-deduplication)） 

##### 去重与实际一次语义

消息去重，使 Pulsar 成为与流处理引擎（SPE）或者其他寻求 “实际一次” 处理语义的系统连接的完美消息系统。消息系统若不提供自动消息去重，则需要 SPE 或者其他系统保证去重。这意味着严格的消息顺序来自于让程序承担额外的去重工作。使用 Pulsar，严格的顺序保证不会带来任何应用层面的消耗。

**更深入的信息可以参考 [this post](https://streaml.io/blog/pulsar-effectively-once/) 及 [Streamlio blog](https://streaml.io/blog)**

[]: http://pulsar.apache.org/docs/en/concepts-messaging/	"pulsar官方wiki"
[]: https://blog.csdn.net/liyiming2017/article/details/82966031	"稀有气体CSDN博客"

