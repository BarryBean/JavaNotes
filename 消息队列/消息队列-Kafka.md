# 1. Kafka简介
答：Kafka是一种**分布式**的基于**发布/订阅**的消息系统。
## 1.1 特性
支持消息的持久化；支持批量读写消息；支持消息分区；支持多副本。
## 1.2 高性能原理
答：Kafka实现高性能的原因是：
- **顺序读写磁盘**：Kafka将消息保存在磁盘中，顺序进行读写，磁盘的顺序读写速度超过内存的随机读写。而且消息消费后不会删除。(删除针对的是过期消息)
- **页缓存**：把磁盘中的数据缓存到内存中，以减少对磁盘IO的操作。
- **零拷贝**：将数据从磁盘文件复制到网卡设备中，不经过从缓存到应用程序，再从应用程序到socket，提高性能。

总结：
1. 针对**写入**操作，Kafka**先在页缓存中写** = 写内存，**再将数据刷入磁盘顺序读写**；
2. 针对**消费**操作，Kafka将**缓存数据直接发送到网卡传输给消费者**，跳过了两次拷贝数据的步骤，socket缓存中拷贝一个描述符。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200216161241840.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0NzYxMDEy,size_16,color_FFFFFF,t_70)



# 2. 发布订阅模型
答：如下所述：
- 生产者(Producer)：生产消息，将消息推送到topic分区中；
- 消费者(Consumer)：消费消息，从topic中拉取；
- 主题(Topic)：消息存储的集合，**Kafka根据topic对消息归类**，每条消息都必须指定topic；
- 分区(Partition)：**每个topic可以划分多个分区**，相当于水平拓展，每个消息在分区中都有offset做编号排序；
- Broker：**一个server就是一个Broker**，用来接收生产者发来的消息并存入磁盘和服务消费者拉取分区消息；
- 集群(Cluster)：多个Broker组成一个Cluster集群，每个集群选举一个Broker称为集群控制器(Cluster Controller)，管理集群。
- 消费者组(Consumer Group)：每个消费者都属于一个消费者组，每条消息只能发送给一个消费者；
- 副本(Replica)：对消息进行冗余备份，**每个分区有一个Leader副本和多个Follewer副本**，副本中的内容都是一样的。
- 日志压缩和保留：日志**压缩就是定时进行相同key的合并**，日志**保留定时删除老消息**。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200216161610153.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0NzYxMDEy,size_16,color_FFFFFF,t_70)




# 3. 文件存储机制
## 3.1 存储在文件系统中
答：Kafka消息存储在文件系统上。消息以topic分类，topic是逻辑上的概念，物理上存储在Partition上。partition的命名规则是：<topic_name>-<partition_id>。
## 3.2 底层原理
答：**一个partition又能分为若干大小相等的segment**，segment**由索引和数据两部分组成**，相关文件为.index(index文件采用稀疏索引)和.log。**Segment是Kafka文件存储的最小单位**。命名规则是：上一个segment最后一条消息的offset值。
>索引文件和数据文件对应关系：索引文件<3,497>，表示数据文件中第3个message，物理偏移地址为497。
## 3.3 为什么不能减小分区+只能增长分区
答：总结如下：
- 删除的分区消息难处理，直接删除造成数据丢失；
- 如果插入现有分区的尾部，一些带时间戳的消息会有影响；
- 如果消息量大，复制到其它分区也会很耗费资源；

解决方案：创建一个分区数比较小的topic，将现有topic中的消息按照一定的逻辑复制过去。






# 4. 副本机制
## 4.1 同步复制和异步复制
答：分布式存储中对于冗余备份有两种方式。
 - **同步复制**：当**所有的Follower副本都复制完成**，**消息**才被认为**提交完成**。缺点是：其中一个故障，消息就无法提交，高延迟。
 - **异步复制**：当**Leader副本接收到消息时**，就认为**消息提交完成**。Follower副本异步从Leader副本同步消息。缺点是：不能保证同步速度和Leader宕机，Follower还未同步，导致消息丢失。




## 4.2 ISR可用副本集合
答：Kafka引入ISR(可用副本集合)，表示**当前可用的且消息量和Leader差不多的副本集合**。要求是副本最后一条信息的offset必须要与Leader最后一条信息的offset的差值在阈值以内。
 - 优点：Follwer副本**延迟过高就被踢**出集合，Leader**副本宕机**优先将ISR中的Follower副本**就选**为**新**Leader。
 - HW(HighWatermark)：高水位，当消费者拉取消息时只能拉取到HW之前的消息。
 - LEO(Log End Offset)：顾名思义，标记最后一条信息的offset。



## 4.3 流程
 - producer向分区推送消息；
 - Leader副本将消息追加到log中，递增LEO；
 - Follower副本拉取消息并同步；
 - Follower副本更新本地log，递增LEO；
 - ISR集合中所有副本都完成同步，更新HW。





# 5. 生产者设计
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200216162405427.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0NzYxMDEy,size_16,color_FFFFFF,t_70)
1. 序列化消息+计算partition
根据key-value对消息进行序列化，然后计算partition。
2. 发送到batch
根据topic-partition获取对应的batchs，然后将消息append到batch中。
3. 唤醒Sender线程
如果batch满了则唤醒Sender线程。batch内消息有序。
4. Sender把消息有序发到 broker
确定tp relica leader 所在的broker和保证幂等性发送，引入了Producer ID（即PID）和Sequence Number。Producer端和Broker端都维护一个序号。
对于接收的每条消息，
> 消息序号=Broker维护的序号+1，Broker接受；
 消息序号>Broker维护的序号+1，说明中间有数据尚未写入，Broker拒绝该消息，抛出异常； 
 		消息序号<=Broker维护的序号，说明消息已被保存，即为重复消息，Broker直接丢弃该消息，Producer抛出DuplicateSequenceNumber Sender发送失败后会重试，这样可以保证每个消息都被发送到broker.
5. Sender处理broker发来的produce response。
broker处理完Sender的produce请求，就会发送produce response给Sender，此时producer将执行我们为send设置的回调函数。




# 6. 消费者设计
消费者时消费组的一部分，如果应用需要读取全量消息，则设置一个消费组；如果该消费能力不足，则在这个消费组里增加消费者。
## 6.1 重平衡
答：重平衡是指**重新分配分区**。新的消费者进入消费组时，从其他消费者那拿到分区；消费者离开消费组时，其所有的分区划分给其他。**重平衡期间所有消费者都不能消费消息**。
### 过程
消费者定期**发送心跳包**到作为组协调者(group coordinator)的 broker 来保持在消费组内存活。若超过一定时间没有发送心跳，组协调者认为该消费者已经宕机，触发重平衡。
注：高版本中可以设置一个消费者最长时间不消费但仍存活，避免活锁。
## 6.2 分区和消费模型
- 消费者在消费时能从不同分区获得消息，但无法重建topic内的顺序。因为**Kafka只能保证分区内消息有序，全局无法保证**。
- **分区只删除到期的消息，有效期内所有消费组都能消费**。分区会给每个Group保存一个offset，记录消费到的位置。

## 6.3 Kafka是pull模型
答：Kafka采用的是Producer向broker Push消息，Consumer从broker Pull消息。
### 原因：
- push模式的消息发送频率由broker决定，难以适应消费速率不同的消费者，容易造成网络拥塞；
- pull模式可以根据Consumer实际消费能力控制消息速率。

# 7. 消息传输的事务定义
答：通常有三种。
## 7.1 最多一次
答：At most once(最多消费一次)：**消息可能丢失，但绝不重复**。
- Producer生产消息时，写数据失败则直接跳过消息 => 消息丢失；
- Consumer消费消息时，提交offset，中途故障没有消费完，下次会直接从offset开始 => 故障点到offset间消息丢失
## 7.2 最少一次
答：At least once(最少消费一次)：**消息绝不丢失，但可能重复**。
- 若Producer没有收到成功ack，会重试写入 => 消息重复
## 7.3 恰好一次
答：Exactly once(恰好消费一次)：**消息有且仅有被传输一次和传输一次**。
- Producer生产消息时保证幂等性。**对于同一数据无论操作多少次，都只写入一次**。
注：幂等性指不论调用多少次，结果都是一样的。
- 原子性写操作。broker写入数据，保证原子性，要么成功要么失败，不重试。
# 8. 保证可靠性
答：最基础的有四点：
- **消息有序**。Producer先写入A再写入B，则Consumer就先读取A再读取B。
- **保证提交**。消息写入ISR集合所有副本时，才认为已提交。
- **数据不丢失**。消息提交后，只要有一个副本存活，数据就不会丢失。
- **读取已提交**。Consumer只能读取已提交的消息。
