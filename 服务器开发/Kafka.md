### 基本概念

#### broker

Kafka集群包含一个或多个服务器，这种服务器被称为broker。

#### Topic

每条发布到Kafka集群的消息都有一个类别，这个类别被称为Topic。Topic在逻辑上可以被认为是一个queue。

#### Partition

为了使得Kafka的吞吐率可以线性提高，物理上把Topic分成一个或多个Partition，每个Partition在物理上对应一个文件夹，该文件夹下存储这个Partition的所有消息和索引文件。若创建topic1和topic2两个topic，且分别有13个和19个分区，则**整个集群**上会相应会生成共32个文件夹。

#### Producer

负责发布消息到Kafka broker。Producer发送消息到broker时，会根据Paritition机制选择将其存储到哪一个Partition。如果Partition机制设置合理，所有消息可以均匀分布到不同的Partition里，这样就实现了负载均衡。如果一个Topic对应一个文件，那这个文件所在的机器I/O将会成为这个Topic的性能瓶颈，而有了Partition后，不同的消息可以**并行**写入不同broker的不同Partition里，极大的提高了吞吐率。

在发送一条消息时，可以指定这条消息的key，Producer根据这个key和Partition机制来判断应该将这条消息发送到哪个Parition。Paritition机制可以通过指定Producer的paritition这一参数来指定，该class必须实现kafka.producer.Partitioner接口。

#### Consumer

消息消费者，向Kafka broker读取消息的客户端。

#### Consumer Group

每个Consumer属于一个特定的Consumer Group（可为每个Consumer指定group name，若不指定group name则属于默认的group）。

使用Consumer high level API时，同一Topic的一条消息只能被同一个Consumer Group内的一个Consumer消费，但多个Consumer Group可同时消费这一消息。因此，如果需要实现广播，只要每个Consumer有一个独立的Group就可以了。要实现单播只要所有的Consumer在同一个Group里。

### 集群

一个典型的Kafka集群中包含若干Producer（可以是web前端产生的Page View，或者是服务器日志，系统CPU、Memory等），若干broker（Kafka支持水平扩展，一般broker数量越多，集群吞吐率越高），若干Consumer Group，以及一个Zookeeper集群。Kafka通过Zookeeper管理集群配置，选举leader，以及在Consumer Group发生变化时进行rebalance。Producer使用push模式将消息发布到broker，Consumer使用pull模式从broker订阅并消费消息。

### **Kafka delivery guarantee**
#### At most once 

消息可能会丢，但绝不会重复传输 

#### At least one 

消息绝不会丢，但可能会重复传输 

#### Exactly once 

每条消息肯定会被传输一次且仅传输一次（常用）
