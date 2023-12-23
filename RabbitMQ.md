# 1 RabbitMQ安装配置

## 1.1 安装

### 1.1.1 下载

下载链接 ：[Releases · rabbitmq/rabbitmq-server · GitHub](https://github.com/rabbitmq/rabbitmq-server/releases/)

### 1.1.2 安装

选择安装路径，使用默认安装配置

## 1.2 配置

配置教程 ：[windows下载与安装RabbitMQ](https://blog.csdn.net/gloamer/article/details/120130836)

### 1.2.1 配置环境变量

> 1. 打开 windows 【高级系统设置】
> 
> 2. 点击【环境变量】，在【系统变量】窗口下
> 
> 3. 新建变量，变量名为 【RABBITMQ_SERVER】，变量值为Erlang安装路径
> 
> 4. 双击 【path】变量，点击【新建】，变量值为 【%RABBITMQ_SERVER%\sbin】

### 1.2.2 激活可视化插件

> 1. 进入RabbitMQ安装位置下的sbin目录
> 
> 2. 在目录上输入cmd
> 
> 3. 在命令行窗口中执行命令【rabbitmq-plugins.bat enable rabbitmq_management】

## 1.3 使用

### 1.3.1 检查rabbitmq运行环境是否正常

> 命令行窗口输入 【rabbitmqctl status】，无报错信息则正常

### 1.3.2 使用可视化插件访问rabbitmq

> - 访问地址 ：[rabbitmq](http://localhost:15672)
> 
> - 账号 ：guest
> 
> - 密码 ：guest

# 2 基本概念

## 2.1 消息队列（Message Queue，MQ）

消息队列是一种通信模式，用于在应用程序之间传递消息。它通常被用于解耦发送者和接收者，以及处理高并发情况。基本上，消息队列允许应用程序在异步模式下进行通信，这意味着发送消息的应用程序不需要等待接收者立即处理消息。
消息队列系统通常包括以下几个关键组件：

1. **消息：** 传递的数据单元，可以是文本、JSON、XML 等格式的信息。
2. **生产者：** 负责发布消息到消息队列中的应用程序。
3. **消费者：** 从消息队列中接收消息的应用程序。
4. **消息队列：** 用于存储消息的中间件组件，确保消息在生产者和消费者之间的可靠传递。
   
   消息队列系统的优势包括解耦、异步通信、削峰填谷（平滑处理流量峰值）、消息持久化（确保消息不会丢失）以及实现分布式系统的能力。

### 2.1.1 MQ优点

**系统可用性降低**：以前只要担心系统的问题，现在还要考虑 MQ 挂掉的问题，MQ 挂掉，所关联的系统都会无法提供服务。

**系统复杂性变高**：要考虑消息丢失、消息重复消费等问题。

**一致性问题**：多个 MQ 消费系统，部分成功，部分失败，要考虑事务问题。

### 2.1.2 MQ缺点

### 2.1.3 常用MQ及其对比

**ActiveMQ**：支持万级的吞吐量，较成熟完善；官方更新迭代较少，社区的活跃度不是很高，有消息丢失的情况。

**RabbitMQ**：延时低，微妙级延时，社区活跃度高，bug 修复及时，而且提供了很友善的后台界面；用 Erlang 语言开发，只熟悉 Java 的无法阅读源码和自行修复 bug。

**RocketMQ**：阿里维护的消息中间件，可以达到十万级的吞吐量，支持分布式事务。

**Kafka**：分布式的中间件，最大优点是其吞吐量高，一般运用于大数据系统的实时运算和日志采集的场景，功能简单，可靠性高，扩展性高；缺点是可能导致重复消费。

### 2.1.4 MQ 使用场景

#### 2.1.4.1 应用解耦

当 A 系统生产关键数据，发送数据给多个其他系统消费，此时 A 系统和其他系统产生了严重的耦合，如果将 A 系统产生的数据放到 MQ 当中，其他系统去 MQ 获取消费数据，此时各系统独立运行只与 MQ 交互，添加新系统消费 A 系统的数据也不需要去修改 A 系统的代码，达到了解耦的效果。

#### 2.1.4.2 异步处理

互联网类企业对用户的直接操作，一般要求每个请求在 200ms 以内完成。对于一个系统调用多个系统，不使用 MQ 的情况下，它执行完返回的耗时是调用完所有系统所需时间的总和；使用 MQ 进行优化后，执行的耗时则是执行主系统的耗时加上发送数据到消息队列的耗时，大幅度提升系统性能和用户体验。

#### 2.1.4.3 流量削峰

MySQL 每秒最高并发请求在 2000 左右，用户访问量高峰期的时候涌入的大量请求，会将 MySQL 打死，然后系统就挂掉，但过了高峰期，请求量可能远低于 2000，这种情况去增加服务器就不值得，如果使用 MQ 的情况，将用户的请求全部放到 MQ 中，让系统去消费用户的请求，不要超过系统所能承受的最大请求数量，保证系统不会再高峰期挂掉，高峰期过后系统还是按照最大请求数量处理完请求。



## 2.2 生产者-消费者模式

生产者-消费者模式是一种经典的并发模式，用于解耦生产数据和消费数据的过程。这种模式通常用于解决生产者和消费者之间的处理速度不匹配的情况。
在这个模式中，生产者负责生成数据（或任务），而消费者则负责处理这些数据（或任务）。它们通过共享一个消息队列（或缓冲区）来实现通信，而不需要直接相互通信。
基本流程如下：

1. **生产者生成数据：** 生产者将数据放入共享的消息队列中，而不需要知道哪个消费者会处理这些数据。
2. **消费者获取数据：** 消费者从消息队列中获取数据，并进行处理。
   这种模式的优势在于能够平衡生产者和消费者之间的处理能力差异，以及允许系统进行水平扩展，从而更好地应对高并发情况。
   常见的应用包括异步任务处理、日志处理、事件驱动架构等。消息队列系统（如RabbitMQ、Kafka等）通常用于实现这种模式，以便管理数据的传输和处理。

# 3 RabbitMQ

### 有什么好处

**解耦**，系统A在代码中直接调用系统B和系统C的代码，如果将来D系统接入，系统A还需要修改代码，过于麻烦！

**异步**，将消息写入消息队列，非必要的业务逻辑以异步的方式运行，加快响应速度

**削峰**，并发量大的时候，所有的请求直接怼到数据库，造成数据库连接异常

### RabbitMQ特点?

**1、可靠性:** RabbitMQ使用一些机制来保证可靠性， 如持久化、传输确认及发布确认等。

**2、灵活的路由 :** 在消息进入队列之前，通过交换器来路由消息。对于典型的路由功能， RabbitMQ 己经提供了一些内置的交换器来实现。针对更复杂的路由功能，可以将多个 交换器绑定在一起， 也可以通过插件机制来实现自己的交换器。

**3、扩展性:** 多个RabbitMQ节点可以组成一个集群，也可以根据实际业务情况动态地扩展 集群中节点。

**4、高可用性 :** 队列可以在集群中的机器上设置镜像，使得在部分节点出现问题的情况下队 列仍然可用。

**5、多种协议:** RabbitMQ除了原生支持AMQP协议，还支持STOMP， MQTT等多种消息 中间件协议。

**6、多语言客户端 :**RabbitMQ 几乎支持所有常用语言，比如 Java、 Python、 Ruby、 PHP、 C#、 JavaScript 等。

**7、管理界面 :** RabbitMQ 提供了一个易用的用户界面，使得用户可以监控和管理消息、集 群中的节点等。

**8、令插件机制:** RabbitMQ 提供了许多插件 ， 以实现从多方面进行扩展，当然也可以编写自 己的插件。

### RocketMQ的消息堆积如何处理？

首先要找到是什么原因导致的消息堆积，是 Producer 太多了，Consumer 太少了导致的还是说其他情况，总之先定位问题。

然后看下消息消费速度是否正常，正常的话，可以通过上线更多 Consumer 临时解决消息堆积问题

### AMQP是什么?

RabbitMQ就是 AMQP 协议的 Erlang 的实现(当然 RabbitMQ 还支持 STOMP2、 MQTT3 等协议 ) AMQP 的模型架构 和 RabbitMQ 的模型架构是一样的，生产者将消息发送给交换器，交换器和队列绑定 。

RabbitMQ 中的交换器、交换器类型、队列、绑定、路由键等都是遵循的 AMQP 协议中相 应的概念。目前 RabbitMQ 最新版本默认支持的是 AMQP 0-9-1。

### AMQP协议3层？

**1、Module Layer:** 协议最高层，主要定义了一些客户端调用的命令，客户端可以用这些命令实现自己的业务逻辑。

**2、Session Layer:** 中间层，主要负责客户端命令发送给服务器，再将服务端应答返回客户端，提供可靠性同步机制和错误处理。

**3、TransportLayer:** 最底层，主要传输二进制数据流，提供帧的处理、信道服用、错误检测和数据表示等。

### rocketmq的工作流程是怎样的？

RocketMq的工作流程如下：

**1、首先启动NameServer**。NameServer启动后监听端口，等待Broker、Producer以及Consumer连上来

**2、启动Broker**。启动之后，会跟所有的NameServer建立并保持一个长连接，定时发送心跳包。心跳包中包含当前Broker信息(ip、port等)、Topic信息以及Borker与Topic的映射关系

**3、创建Topic**。创建时需要指定该Topic要存储在哪些Broker上，也可以在发送消息时自动创建Topic

**4、Producer发送消息**。启动时先跟NameServer集群中的其中一台建立长连接，并从NameServer中获取当前发送的Topic所在的Broker；然后从队列列表中轮询选择一个队列，与队列所在的Broker建立长连接，进行消息的发送

**5、Consumer消费消息**。跟其中一台NameServer建立长连接，获取当前订阅Topic存在哪些Broker上，然后直接跟Broker建立连接通道，进行消息的消费

### rocketmq如何保证高可用性？

1、集群化部署NameServer。Broker集群会将所有的broker基本信息、topic信息以及两者之间的映射关系，轮询存储在每个NameServer中（也就是说每个NameServer存储的信息完全一样）。因此，NameServer集群化，不会因为其中的一两台服务器挂掉，而影响整个架构的消息发送与接收；

2、集群化部署多broker。producer发送消息到broker的master，若当前的master挂掉，则会自动切换到其他的master

cosumer默认会访问broker的master节点获取消息，那么master节点挂了之后，该怎么办呢？它就会自动切换到同一个broker组的slave节点进行消费

那么你肯定会想到会有这样一个问题：consumer要是直接消费slave节点，那master在宕机前没有来得及把消息同步到slave节点，那这个时候，不就会出现消费者不就取不到消息的情况了？

这样，就引出了下一个措施，来保证消息的高可用性

3、设置同步复制

前面已经提到，消息发送到broker的master节点上，master需要将消息复制到slave节点上，rocketmq提供两种复制方式：同步复制和异步复制

异步复制，就是消息发送到master节点，只要master写成功，就直接向客户端返回成功，后续再异步写入slave节点

同步复制，就是等master和slave都成功写入内存之后，才会向客户端返回成功

那么，要保证高可用性，就需要将复制方式配置成同步复制，这样即使master节点挂了，slave上也有当前master的所有备份数据，那么不仅保证消费者消费到的消息是完整的，并且当master节点恢复之后，也容易恢复消息数据

在master的配置文件中直接配置brokerRole：SYNC_MASTER即可

### RocketMq如何负载均衡？

**1、**producer发送消息的负载均衡：默认会**轮询**向Topic的所有queue发送消息，以达到消息平均落到不同的queue上；而由于queue可以落在不同的broker上，就可以发到不同broker上（当然也可以指定发送到某个特定的queue上）

**2、**consumer订阅消息的负载均衡：假设有5个队列，两个消费者，则第一个消费者消费3个队列，第二个则消费2个队列，以达到平均消费的效果。而需要注意的是，当consumer的数量大于队列的数量的话，根据rocketMq的机制，多出来的队列不会去消费数据，因此建议consumer的数量小于或者等于queue的数量，避免不必要的浪费

### RocketMq的存储机制了解吗？

RocketMq采用文件系统进行消息的存储，相对于ActiveMq采用关系型数据库进行存储的方式就更直接，性能更高了

**RocketMq与Kafka在写消息与发送消息上，继续沿用了Kafka的这两个方面：顺序写和零拷贝**

**1、顺序写**

我们知道，操作系统每次从磁盘读写数据的时候，都需要找到数据在磁盘上的地址，再进行读写。而如果是机械硬盘，寻址需要的时间往往会比较长

而一般来说，如果把数据存储在内存上面，少了寻址的过程，性能会好很多；但Kafka 的数据存储在磁盘上面，依然性能很好，这是为什么呢？

这是因为，Kafka采用的是顺序写，直接追加数据到末尾。实际上，磁盘顺序写的性能极高，在磁盘个数一定，转数一定的情况下，基本和内存速度一致

因此，磁盘的顺序写这一机制，极大地保证了Kafka本身的性能

**2、零拷贝**

比如：读取文件，再用socket发送出去这一过程

```
buffer = File.read
Socket.send(buffer)
```

传统方式实现：
先读取、再发送，实际会经过以下四次复制

1、将磁盘文件，读取到操作系统内核缓冲区**Read Buffer**
2、将内核缓冲区的数据，复制到应用程序缓冲区**Application Buffer**
3、将应用程序缓冲区**Application Buffer**中的数据，复制到socket网络发送缓冲区
4、将**Socket buffer**的数据，复制到**网卡**，由网卡进行网络传输

![图片](https://mmbiz.qpic.cn/mmbiz_png/MnVGSgCV6PIK8jXpLaapkicbHNK1kUeCrHicCzLtySU8FOcDfVHBiabk7aaulnUbkLjOtic61dZjmdwdsfnOE7icoMQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

传统方式，读取磁盘文件并进行网络发送，经过的四次数据copy是非常繁琐的

重新思考传统IO方式，会注意到**在读取磁盘文件后，不需要做其他处理，直接用网络发送出去的这种场景下**，第二次和第三次数据的复制过程，不仅没有任何帮助，反而带来了巨大的开销。那么这里使用了**零拷贝**，也就是说，直接由内核缓冲区**Read Buffer**将数据复制到**网卡**，省去第二步和第三步的复制。

![图片](https://mmbiz.qpic.cn/mmbiz_png/MnVGSgCV6PIK8jXpLaapkicbHNK1kUeCrFUPV9ic9icMfxWPkopmIGImARxlx7DFW23uu93aQibahpic3WfllGicP28w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

那么采用零拷贝的方式发送消息，必定会大大减少读取的开销，使得RocketMq读取消息的性能有一个质的提升
此外，还需要再提一点，零拷贝技术采用了MappedByteBuffer内存映射技术，采用这种技术有一些限制，其中有一条就是传输的文件不能超过2G，这也就是为什么RocketMq的存储消息的文件CommitLog的大小规定为1G的原因

**小结：**RocketMq采用文件系统存储消息，并采用顺序写写入消息，使用零拷贝发送消息，极大得保证了RocketMq的性能

### RocketMq的存储结构是怎样的？

如图所示，消息生产者发送消息到broker，都是会按照顺序存储在CommitLog文件中，每个commitLog文件的大小为1G

![图片](https://mmbiz.qpic.cn/mmbiz_png/MnVGSgCV6PLEzXJc3dJXW4KoOEotpmgkbJAtb3fgohPUQrxrkUDrneStlibhRTR1xedPnrgOLCqCiafIRx0vUb3g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

CommitLog-存储所有的消息元数据，包括Topic、QueueId以及message

CosumerQueue-消费逻辑队列：存储消息在CommitLog的offset

IndexFile-索引文件：存储消息的key和时间戳等信息，使得RocketMq可以采用key和时间区间来查询消息

也就是说，rocketMq将消息均存储在CommitLog中，并分别提供了CosumerQueue和IndexFile两个索引，来快速检索消息

### RocketMq性能比较高的原因？

RocketMq采用文件系统存储消息，采用顺序写的方式写入消息，使用零拷贝发送消息，这三者的结合极大地保证了RocketMq的性能

### AMQP模型的几大组件？

- 交换器 (Exchange)：消息代理服务器中用于把消息路由到队列的组件。
- 队列 (Queue)：用来存储消息的数据结构，位于硬盘或内存中。
- 绑定 (Binding)：一套规则，告知交换器消息应该将消息投递给哪个队列。

### 生产者Producer?

消息生产者，就是投递消息的一方。

消息一般包含两个部分：消息体（payload)和标签(Label)。

### 消费者Consumer?

消费消息，也就是接收消息的一方。

消费者连接到RabbitMQ服务器，并订阅到队列上。消费消息时只消费消息体，丢弃标签。

### Broker服务节点？

Broker可以看做RabbitMQ的服务节点。一般请下一个Broker可以看做一个RabbitMQ服务器。

### Queue队列？

Queue:RabbitMQ的内部对象，用于存储消息。多个消费者可以订阅同一队列，这时队列中的消息会被平摊（轮询）给多个消费者进行处理。

### Exchange交换器？

Exchange:生产者将消息发送到交换器，有交换器将消息路由到一个或者多个队列中。当路由不到时，或返回给生产者或直接丢弃。

### RoutingKey路由键？

生产者将消息发送给交换器的时候，会指定一个RoutingKey,用来指定这个消息的路由规则，这个RoutingKey需要与交换器类型和绑定键(BindingKey)联合使用才能最终生效。

### Binding绑定？

通过绑定将交换器和队列关联起来，一般会指定一个BindingKey,这样RabbitMq就知道如何正确路由消息到队列了。

### 交换器4种类型？

主要有以下4种。

**fanout:** 把所有发送到该交换器的消息路由到所有与该交换器绑定的队列中。

**direct:**把消息路由到BindingKey和RoutingKey完全匹配的队列中。

**topic:** 匹配规则：

​    RoutingKey 为一个 点号'.': 分隔的字符串。比如: java.xiaoka.show

​    BindingKey和RoutingKey一样也是点号“.“分隔的字符串。

​    BindingKey可使用 * 和 # 用于做模糊匹配，*匹配一个单词，#匹配多个或者0个

**headers:**不依赖路由键匹配规则路由消息。是根据发送消息内容中的headers属性进行匹配。性能差，基本用不到。

### 如何保证消息不被重复消费?

消息被重复消费，就是消费方多次接受到了同一条消息。根本原因就是，第一次消费完之后，消费方给 MQ 确认已消费的反馈，MQ 没有成功接受。比如网络原因、MQ 重启等。

所以 MQ 是无法保证消息不被重复消费的，只能业务系统层面考虑。

不被重复消费的问题，就被转化为消息消费的幂等性的问题。幂等性就是指一次和多次请求的结果一致，多次请求不会产生副作用。

保证消息消费的幂等性可以考虑下面的方式：

- 给消息生成全局 id，消费成功过的消息可以直接丢弃
- 消息中保存业务数据的主键字段，结合业务系统需求场景进行处理，避免多次插入、是否可以根据主键多次更新而并不影响结果等

### 消息重复消费如何解决？

影响消息正常发送和消费的**「重要原因是网络的不确定性。」**

- **「引起重复消费的原因」**

- - **「ACK」** 正常情况下在consumer真正消费完消息后应该发送ack，通知broker该消息已正常消费，从queue中剔除
    
    当ack因为网络原因无法发送到broker，broker会认为词条消息没有被消费，此后会开启消息重投机制把消息再次投递到consumer
  
  - **「消费模式」** 在CLUSTERING模式下，消息在broker中会保证相同group的consumer消费一次，但是针对不同group的consumer会推送多次

- **「解决方案」**

- - **「数据库表」**
    
    处理消息前，使用消息主键在表中带有约束的字段中insert
  
  - **「Map」**
    
    单机时可以使用map ConcurrentHashMap -> putIfAbsent  guava cache
  
  - **「Redis」**
    
    分布式锁搞起来。

### 如何让 RocketMQ 保证消息的顺序消费？

首先多个 queue 只能保证单个 queue 里的顺序，queue 是典型的 FIFO，天然顺序。多个 queue 同时消费是无法绝对保证消息的有序性的。所以总结如下：

同一 topic，同一个 QUEUE，发消息的时候一个线程去发送消息，消费的时候 一个线程去消费一个 queue 里的消息。

### 怎么保证消息发到同一个queue？

Rocket MQ给我们提供了MessageQueueSelector接口，可以自己重写里面的接口，实现自己的算法，举个最简单的例子：判断`i % 2 == 0`，那就都放到queue1里，否则放到queue2里。

```java
for (int i = 0; i < 5; i++) {
    Message message = new Message("orderTopic", ("hello!" + i).getBytes());
    producer.send(
        // 要发的那条消息
        message,
        // queue 选择器 ，向 topic中的哪个queue去写消息
        new MessageQueueSelector() {
            // 手动 选择一个queue
            @Override
            public MessageQueue select(
                // 当前topic 里面包含的所有queue
                List<MessageQueue> mqs,
                // 具体要发的那条消息
                Message msg,
                // 对应到 send（） 里的 args，也就是2000前面的那个0
                Object arg) {
                // 向固定的一个queue里写消息，比如这里就是向第一个queue里写消息
                if (Integer.parseInt(arg.toString()) % 2 == 0) {
                    return mqs.get(0);
                } else {
                    return mqs.get(1);
                }
            }
        },
        // 自定义参数：0
        // 2000代表2000毫秒超时时间
        i, 2000);
}
```

### 如何保证消息不丢失？

**生产者丢失消息**：如网络传输中丢失消息、MQ 发生异常未成功接收消息等情况。**解决办法**：主流的 MQ 都有确认或事务机制，可以保证生产者将消息送达到 MQ。如 RabbitMQ 就有事务模式和 confirm 模式。

**MQ 丢失消息**：MQ 成功接收消息内部处理出错、宕机等情况。**解决办法**：开启 MQ 的持久化配置。

**消费者丢失消息**：采用消息自动确认模式，消费者取到消息未处理挂掉了。**解决办法**：改为手动确认模式，消费者成功消费消息再确认。

### 如何保证消息的顺序性？

生产者保证消息入队的顺序。

MQ 本身是一种先进先出的数据接口，将同一类消息，发到同一个 queue 中，保证出队是有序的。

避免多消费者并发消费同一个 queue 中的消息。

### 消息大量积压怎么解决？

消息的积压来自于两方面：要么发送快了，要么消费变慢了。

单位时间发送的消息增多，比如赶上大促或者抢购，短时间内不太可能优化消费端的代码来提升消费性能，唯一的办法是通过扩容消费端的实例数来提升总体的消费能力。严重影响 QM 甚至整个系统时，可以考虑临时启用多个消费者，并发接受消息，持久化之后再单独处理，或者直接丢弃消息，回头让生产者重新生产。

如果短时间内没有服务器资源扩容，没办法的办法是将系统降级，通过关闭某些不重要的业务，减少发送的数据量，最低限度让系统还能正常运转，服务重要业务。

监控发现，产生和消费消息的速度没什么变化，出现消息积压的情况，检查是有消费失败反复消费的情况。

监控发现，消费消息的速度变慢，检查消费实例，日志中是否有大量消费错误、消费线程是否死锁、是否卡在某些资源上。

### 如何保证MQ的高可用?

**ActiveMQ：**

Master-Slave 部署方式主从热备，方式包括通过共享存储目录来实现(shared filesystem Master-Slave)、通过共享数据库来实现(shared database Master-Slave)、5.9版本后新特性使用 ZooKeeper 协调选择 master(Replicated LevelDB Store)。

Broker-Cluster 部署方式进行负载均衡。

**RabbitMQ：**

单机模式与普通集群模式无法满足高可用，镜像集群模式指定多个节点复制 queue 中的消息做到高可用，但消息之间的同步网络性能开销较大。

**RocketMQ：**

有多 master 多 slave 异步复制模式和多 master 多 slave 同步双写模式支持集群部署模式。

Producer 随机选择 NameServer 集群中的其中一个节点建立长连接，定期从 NameServer 获取 Topic 路由信息，并向提供 Topic 服务的 Broker Master 建立长连接，且定时向 Master 发送心跳，只能将消息发送到 Broker master。

Consumer 同时与提供 Topic 服务的 Master、Slave 建立长连接，从 Master、Slave 订阅消息都可以，订阅规则由 Broker 配置决定。

**Kafka：**

由多个 broker 组成，每个 broker 是一个节点；topic 可以划分为多个 partition，每个 partition 可以存在于不同的 broker 上，每个 partition 存放一部分数据，这样每个 topic 的数据就分散存放在多个机器上的。

replica 副本机制保证每个 partition 的数据同步到其他节点，形成多 replica 副本；所有 replica 副本会选举一个 leader 与 Producer、Consumer 交互，其他 replica 就是 follower；写入消息 leader 会把数据同步到所有 follower，从 leader 读取消息。

每个 partition 的所有 replica 分布在不同的机器上。某个 broker 宕机，它上面的 partition 在其他节点有副本，如果有 partition 的 leader，会进行重新选举 leader。

### RocketMQ Broker中的消息被消费后会立即删除吗？

**「不会」**，每条消息都会持久化到CommitLog中，每个Consumer连接到Broker后会维持消费进度信息，当有消息消费后只是当前Consumer的消费进度（CommitLog的offset）更新了。

4.6版本默认48小时后会删除不再使用的CommitLog文件

- 检查这个文件最后访问时间
- 判断是否大于过期时间
- 指定时间删除，默认凌晨4点

源码如下：

```java
/**
 * {@link org.apache.rocketmq.store.DefaultMessageStore.CleanCommitLogService#isTimeToDelete()}
 */
private boolean isTimeToDelete() {
    // when = "04";
    String when = DefaultMessageStore.this.getMessageStoreConfig().getDeleteWhen();
    // 是04点，就返回true
    if (UtilAll.isItTimeToDo(when)) {
        return true;
    }
 // 不是04点，返回false
    return false;
}
/**
 * {@link org.apache.rocketmq.store.DefaultMessageStore.CleanCommitLogService#deleteExpiredFiles()}
 */
private void deleteExpiredFiles() {
    // isTimeToDelete()这个方法是判断是不是凌晨四点，是的话就执行删除逻辑。
    if (isTimeToDelete()) {
        // 默认是72，但是broker配置文件默认改成了48，所以新版本都是48。
        long fileReservedTime = 48 * 60 * 60 * 1000;
        deleteCount = DefaultMessageStore.this.commitLog.deleteExpiredFile(72 * 60 * 60 * 1000, xx, xx, xx);
    }
}                                                                     
/**
 * {@link org.apache.rocketmq.store.CommitLog#deleteExpiredFile()}
 */
public int deleteExpiredFile(xxx) {
    // 这个方法的主逻辑就是遍历查找最后更改时间+过期时间，小于当前系统时间的话就删了（也就是小于48小时）。
    return this.mappedFileQueue.deleteExpiredFileByTime(72 * 60 * 60 * 1000, xx, xx, xx);
}
```

### RocketMQ消费模式有几种？

消费模型由 Consumer 决定，消费维度为 Topic。

- 集群消费

- - 一条消息只会被同Group中的一个Consumer消费
  - 多个Group同时消费一个Topic时，每个Group都会有一个Consumer消费到数据

- 广播消费 消息将对一 个 Consumer Group 下的各个 Consumer 实例都消费一遍。即即使这些 Consumer 属于同一个 Consumer Group ，消息也会被 Consumer Group 中的每个 Consumer 都消费一次。

### 生产者消息运转？

**1、**Producer先连接到Broker,建立连接Connection,开启一个信道(Channel)。

**2、**Producer声明一个交换器并设置好相关属性。

**3、**Producer声明一个队列并设置好相关属性。

**4、**Producer通过路由键将交换器和队列绑定起来。

**5、**Producer发送消息到Broker,其中包含路由键、交换器等信息。

**6、**相应的交换器根据接收到的路由键查找匹配的队列。

**7、**如果找到，将消息存入对应的队列，如果没有找到，会根据生产者的配置丢弃或者退回给生产者。

**8、**关闭信道。

**9、**管理连接。

### RocketMQ消息是push还是pull？

RocketMQ没有真正意义的push，都是pull，虽然有push类，但实际底层实现采用的是**「长轮询机制」**，即拉取方式

> broker端属性 longPollingEnable 标记是否开启长轮询。默认开启

源码如下：

```java
// {@link org.apache.rocketmq.client.impl.consumer.DefaultMQPushConsumerImpl#pullMessage()}
// 看到没，这是一只披着羊皮的狼，名字叫PushConsumerImpl，实际干的确是pull的活。
// 拉取消息，结果放到pullCallback里
this.pullAPIWrapper.pullKernelImpl(pullCallback);
```

### 当消费负载均衡consumer和queue不对等的时候会发生什么？

Consumer 和 queue 会优先平均分配，如果 Consumer 少于 queue 的个数，则会存在部分 Consumer 消费多个 queue 的情况，如果 Consumer 等于 queue 的个数，那就是一个 Consumer 消费一个 queue，如果 Consumer 个数大于 queue 的个数，那么会有部分 Consumer 空余出来，白白的浪费了。

### 为什么要主动拉取消息而不使用事件监听方式？

事件驱动方式是建立好长连接，由事件（发送数据）的方式来实时推送。

如果broker主动推送消息的话有可能push速度快，消费速度慢的情况，那么就会造成消息在 Consumer 端堆积过多，同时又不能被其他 Consumer 消费的情况。而 pull 的方式可以根据当前自身情况来 pull，不会造成过多的压力而造成瓶颈。所以采取了 pull 的方式。

### 消费者接收消息过程？

**1、**Producer先连接到Broker,建立连接Connection,开启一个信道(Channel)。

**2、**向Broker请求消费响应的队列中消息，可能会设置响应的回调函数。

**3、**等待Broker回应并投递相应队列中的消息，接收消息。

**4、**消费者确认收到的消息,ack。

**5、**RabbitMq从队列中删除已经确定的消息。

**6、**关闭信道。

**7、**关闭连接。

### Broker如何处理拉取请求的？

Consumer首次请求Broker Broker中是否有符合条件的消息

- 有

- - 响应 Consumer
  - 等待下次 Consumer 的请求

- 没有

- - DefaultMessageStore#ReputMessageService#run方法
  - PullRequestHoldService 来 Hold 连接，每个 5s 执行一次检查 pullRequestTable 有没有消息，有的话立即推送
  - 每隔 1ms 检查 commitLog 中是否有新消息，有的话写入到 pullRequestTable
  - 当有新消息的时候返回请求
  - 挂起 consumer 的请求，即不断开连接，也不返回数据
  - 使用 consumer 的 offset，

### 交换器无法根据自身类型和路由键找到符合条件队列时，有哪些处理？

**mandatory ：**true 返回消息给生产者。

**mandatory:** false 直接丢弃。

### 死信队列？

DLX，全称为 Dead-Letter-Exchange，死信交换器，死信邮箱。当消息在一个队列中变成死信 (dead message) 之后，它能被重新被发送到另一个交换器中，这个交换器就是 DLX，绑定 DLX 的队列就称之为死信队列。

### 导致的死信的几种原因？

- 消息被拒（Basic.Reject /Basic.Nack) 且 requeue = false。
- 消息TTL过期。
- 队列满了，无法再添加。

### 延迟队列？

存储对应的延迟消息，指当消息被发送以后，并不想让消费者立刻拿到消息，而是等待特定时间后，消费者才能拿到这个消息进行消费。

### 优先级队列？

优先级高的队列会先被消费。

可以通过x-max-priority参数来实现。

当消费速度大于生产速度且Broker没有堆积的情况下，优先级显得没有意义。

### 事务机制？

RabbitMQ 客户端中与事务机制相关的方法有三个:

**1、**channel.txSelect  用于将当前的信道设置成事务模式。

**2、**channel . txCommit 用于提交事务 。

**3、**channel . txRollback 用于事务回滚,如果在事务提交执行之前由于 RabbitMQ 异常崩溃或者其他原因抛出异常,通过txRollback来回滚。

### 发送确认机制？

生产者把信道设置为confirm确认模式,设置后，所有再改信道发布的消息都会被指定一个唯一的ID，一旦消息被投递到所有匹配的队列之后，RabbitMQ就会发送一个确认（Basic.Ack)给生产者（包含消息的唯一ID)，这样生产者就知道消息到达对应的目的地了。

### 消费者获取消息的方式？

- 推
- 拉

### 消费者某些原因无法处理当前接受的消息如何来拒绝？

- channel .basicNack
- channel .basicReject

### 消息传输保证层级？

**At most once:**最多一次。消息可能会丢失，单不会重复传输。

**At least once：**最少一次。消息觉不会丢失，但可能会重复传输。

**Exactly once:**  恰好一次，每条消息肯定仅传输一次。

### vhost?

每一个RabbitMQ服务器都能创建虚拟的消息服务器，也叫虚拟主机(virtual host)，简称vhost。

默认为“/”。

### 集群中的节点类型？

**内存节点：**ram,将变更写入内存。

**磁盘节点：**disc,磁盘写入操作。

RabbitMQ要求最少有一个磁盘节点。

### 队列结构？

通常由以下两部分组成？

- rabbit_amqqueue_process :负责协议相关的消息处理，即接收生产者发布的消息、向消费者交付消息、处理消息的确认(包括生产端的 confirm 和消费端的 ack) 等。
- backing_queue:是消息存储的具体形式和引擎，并向 rabbit amqqueue process 提供相关的接口以供调用。

### RabbitMQ中消息可能有的几种状态?

**alpha:** 消息内容(包括消息体、属性和 headers) 和消息索引都存储在内存中 。

**beta:** 消息内容保存在磁盘中，消息索引保存在内存中。

**gamma:** 消息内容保存在磁盘中，消息索引在磁盘和内存中都有 。

**delta:** 消息内容和索引都在磁盘中 。

### **它有哪几种部署类型？分别有什么特点？**

RocketMQ有4种部署类型

**1、单Master**

单机模式, 即只有一个Broker, 如果Broker宕机了, 会导致RocketMQ服务不可用, 不推荐使用

**2、多Master模式**

组成一个集群, 集群每个节点都是Master节点, 配置简单, 性能也是最高, 某节点宕机重启不会影响RocketMQ服务

缺点：如果某个节点宕机了, 会导致该节点存在未被消费的消息在节点恢复之前不能被消费

**3、多Master多Slave模式，异步复制**

每个Master配置一个Slave, 多对Master-Slave, Master与Slave消息采用异步复制方式, 主从消息一致只会有毫秒级的延迟

优点是弥补了多Master模式（无slave）下节点宕机后在恢复前不可订阅的问题。在Master宕机后, 消费者还可以从Slave节点进行消费。采用异步模式复制，提升了一定的吞吐量。总结一句就是，采用

多Master多Slave模式，异步复制模式进行部署，系统将会有较低的延迟和较高的吞吐量

缺点就是如果Master宕机, 磁盘损坏的情况下, 如果没有及时将消息复制到Slave, 会导致有少量消息丢失

**4、多Master多Slave模式，同步双写**

与多Master多Slave模式，异步复制方式基本一致，唯一不同的是消息复制采用同步方式，只有master和slave都写成功以后，才会向客户端返回成功

优点：数据与服务都无单点，Master宕机情况下，消息无延迟，服务可用性与数据可用性都非常高

缺点就是会降低消息写入的效率，并影响系统的吞吐量

实际部署中，一般会根据业务场景的所需要的性能和消息可靠性等方面来选择后两种

# 3 常用命令

rabbitmqctl是用于与 RabbitMQ 节点进行交互的命令行工具。以下是一些常用的 rabbitmqctl`命令：

1. **列出节点信息：**
   
   ```bash
   rabbitmqctl status
   ```

2. **列出用户：**
   
   ```bash
   rabbitmqctl list_users
   ```

3. **添加用户：**
   
   ```bash
   rabbitmqctl add_user <username> <password>
   ```

4. **设置用户标签：**
   
   ```bash
   rabbitmqctl set_user_tags <username> <tag>
   ```

5. **给用户授予权限：**
   
   ```bash
   rabbitmqctl set_permissions -p <vhost> <username> ".*" ".*" ".*"
   ```

6. **列出虚拟主机：**
   
   ```bash
   rabbitmqctl list_vhosts
   ```

7. **添加虚拟主机：**
   
   ```bash
   rabbitmqctl add_vhost <vhost>
   ```

8. **设置策略：**
   
   ```bash
   rabbitmqctl set_policy <name> <pattern> <definition>
   ```

9. **停止应用程序：**
   
   ```bash
   rabbitmqctl stop_app
   ```

10. **重置节点：**
    
    ```bash
    rabbitmqctl reset
    ```
    
    这些命令可以帮助您管理 RabbitMQ 节点、用户、虚拟主机以及权限设置。