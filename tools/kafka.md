# Kafka基础概念
## 消息和批次	
1. 消息是数据单元
2. 批次是一组消息。消息可以分批次写入kafka

## 模式（schema）	消息的schema
要注意消息格式的一致性。升级之后消息的格式可能会有所改变，如果向后兼容是个问题

## 主题	
kafka按主题来区分不同的消息类型
## 分区	
1. 消息可以存储在不同的分区。
2. 同一个分区的消息是先入先出FIFO。不同分区的顺序无法保证
3. 分区在不同服务器，可以提升性能
4. 每个分区，最多属于一个consumer

## 流 stream	
把一个主题的数据看做一个流
## 生产者	
生产者创建消息
## 消费者	
消费者读取消息
## 偏移量 offset	
1. 偏移量是By topic, consumer group, partition
2. offset被消费者提交，并且被保存在一个特殊的主题中_consumer_offset
3. 如果提交的偏移量小于当前，那么这些message会被重新消费
4. 如果提交的偏移量大于当前，那么这些message不会被消费
5. 如果设置为自动提交偏移量，那么会在每隔auto.commit.interval.ms提交一次偏移量。这些自动提交的偏移量代表的message，可能被处理了，也可能没有被处理
6. 偏移量也可以被async commit，要注意async commit的偏移量，可能大的偏移量要早于小的，那么就要注意不要重复提交小的偏移量
7. 如果一直是某个consumer消费，那么偏移量提交与否都没有关系；但是consumer一旦crash，或者发生再均衡，那么偏移量就为新的consumer作为起始点

## Broker	
• 一个独立的kafka服务器称为broker
• broker可以处理数千个partition，每秒百万级的数据
• broker也是集群控制器
• 一个分区从属于broker，一个分区也可以分给多个broker，就会发生分区复制
• 如果一个broker失效，其它broker会接管

## 保留消息	
• 保留一定时间内的消息。如7天
• 保留一定大小的消息。如1G

## 多集群	
1. Kafka本身不能跨集群复制
2. 不过可以使用mirror make来在多集群间复制

## 多个消费者	
• 多个消费者可以读取队列，彼此之间不影响
• 也可以把多个消费者设定到一个群组，共享消息流

# kafka的适用场景
1. 活动跟踪
2. 传递消息
3. 收集度量指标和活动日志
4. 流处理

# kafka部署配置
1. Broker.id在集群中必须是唯一的
2. num.recovery.threads.per.data.dir
kafka服务器启动，关闭，重启时的线程数，默认是1，可以开启更多
3. auto.create.topics.enable设为false，这样kafka就不会在消息写入时创建主题，防止非预期的行为

# topic的配置
1. num.partitions
	• 新创建的主题包含多少个分区
	• 分区的个数，取决于网络带宽，以及预期每秒要写入的消息大小。比如网络带宽允许的情况下，每个分区每秒50M，预期写入1G，那么就需要大约20个分区
2. log.retention.hours/log.retention.minutes/log.retention.ms
数据保留时间。这三个值都是一样的。如果三个都设置，那么kafka使用最小单位的那个
3. log.retention.bytes
数据保留大小。数据保留时间、大小，满足任意一个，就会开始删除
4. log.segment.bytes
	• 日志片段的大小。
	• 当日志片段满了以后，kafka会打开一个新的日志片段。满了的日志片段会关闭等待过期。
	• 只有关闭了的日志片段，才会过期；没有关闭，就一直不会过期
	• 日志片段越小，在使用时间戳获取偏移量就会越准确
5. log.segment.ms
日志片段多久后过期。默认没有值
启用这个值，需要考虑多个分区的日志片段在同一时间关闭，对磁盘性能影响较大
6. message.max.bytes
消息最大size
如果这个值设置的过大，会影响块的大小，并且影响性能
如果设置的小呢？

# 硬件的选择
1. 磁盘吞吐量
2. 磁盘容量
3. 内存
4. 网络
5. CPU

# broker

| Name       | 解释                                |
| ---------- | ----------------------------------- |
| 数量的选择 | topic的容量     topic的吞吐量       |
| 加入集群   | 只需要有相同的zookeeper.connect参数 |

# 操作系统调优
1. 避免发生虚拟内存的交换
2. 磁盘IO性能
3. 网络带宽

# Kafka transaction
1. 对于producer来说，要注明trasactional.id，每个producer一个
2. 对于consumer来说，要注明read commited
	• 只会消费producer已经提交的message，未提交的message不会被消费。
	• consumer成功消费message，提交offset+1；如果offset+1未被提交，下一次poll时，还会拿到相同的消息；如果被提交，那么下次就拿到下一个消息。
	• 如果consumer crash，offset+1未被提交，那么再均衡的新消费者，会重新消费该消息

# 消息递送类型
默认情况下，消息递送是at most once和at least once。

1. At most once  0/1：设置消息enable.idempotence，保证消息唯一。设置acks为0，不保证message成功到达
2. At least once  1/N：设置acks为all即可
3. Exactly once  1
	1. enable.idempotence：kafka会去掉重复的数据
	2. Transactional.id对于每个producer来说都必须是唯一的
	3. 还有一些其它的配置，详细看ProducerConfig代码
	4. 代码 https://www.baeldung.com/kafka-exactly-once

# Producer的配置项

| Acks                                                      | 0 不管是否存储成功；1   只保证发送到一个lead ；   all保证发送到lead和所有followers |
| --------------------------------------------------------- | ------------------------------------------------------------ |
| enable.idempotence                                        | 默认false。发送给kafka的消息，都被分配了product id和message sequence number，如果两者有重复，那么就会被拒绝。 |
| Transactional.id                                          | 每个producer都有唯一的transaction.id                         |
| Batch.size                                                | 生产者一次性发送到kafka的message   size in total             |
| compression.type                                          | 压缩的类型                                                   |
| linger.ms                                                 | 如果batch size在多少毫秒内为达到，那么就继续发送             |
| Buffer.memory                                             | 用于生产者缓冲发送到服务器的数据。如果生产者不能发送message到kafka，那么就在本地缓存。超过缓存的大小，就报错。 |
| Retires                                                   | 重试次数                                                     |
| Client.id                                                 | 任意字符串来标志消息的来源                                   |
| max.in.flight.requests.per.connection                     | 在生产者收到服务器的相应之前，可以发送多少个消息。设为1可以保证消息的顺序执行，不建议 |
| timeout.ms、request.timeout.ms和metadata.fetch.timeout.ms |                                                              |
| max.block.ms                                              |                                                              |
| receive.buffer.bytes和send.buffer.bytes                   | tcp的缓冲区，如果延迟较高，那么设置大一些                    |

# 消费者

## 消费者如何分配分区？	
	1. 如果消费者数量小于分区，那么分区就均分给消费者
	2. 如果消费者数量等于分区，那么每个消费者对应一个分区
	3. 如果消费者数量大于分区，那么多余的消费者就会被闲置
	4. 如果消费者离开，那么该消费者消费的分区就会被其它分区消费
## 再均衡
	1. 再均衡：分区的所有权从一个消费者转移到另外一个消费者
	2. 再均衡期间，消费者无法消费消息，服务在一段时间内不可用
## 心跳
	1. 消费者对称为群组协调器的broker发送心跳，如果停止心跳的时间足够长，那么就认为消费者不可用，触发再均衡
	2. 消费者的心跳和消息轮询是独立的
## 消费者如何获取消息
	以轮询的方式获取消息
## 线程
	一个消费者只能是一个线程，可以起多个线程，对应对个消费者

# consumer的配置项

| Isolation   level                       | Default : read uncommited。可配置的是read commited           |
| --------------------------------------- | ------------------------------------------------------------ |
| fetch.min.bytes                         | 每次拿取的message最大size。如果不够，那么就等待              |
| fetch.max.wait.ms                       | 等待拿取message的最大等待时间。如果这个值和fetch.min.bytes同时设置，那么谁先满足条件，就会触消费者拿走message的操作 |
| max.partition.fetch.bytes               | 每个分区最多能给一个消费者返回的字节数。默认1MB。     如果有20个分区和5个消费者，那么每个消费者都对应4个分区，那么消费者最多能拿取4MB的message     这个值必须比max.message.size大，否则消费者可能无法获取消息。因为分区出来了1MB，max         message size只有512KB，那么消息就出不去了。。 |
| session.timeout.ms                      | 获取不到消费者发送的心跳的最长时间。超出这个时间还没有获取到来自消费者的心跳，那么就认为消费者已经死亡，触发再均衡     这个值不能小于heartbeat.interval.ms。否则心跳还没来得及发呢，就认为人家死了。。。 |
| groupId                                 | 消费者群组                                                   |
| auto.offset.reset                       | 默认是latest，可以改为earliest。当消费者群组的偏移量被删除，或者新加入消费者群组时，从哪里读取offset |
| enable.auto.commit                      | 默认false，consumer拿了message就自动commit。设为true后，auto.commit.interval.ms就无用了   在代码里来commit offset+1即可。因为当前只有一个消费者在消费这个分区，所以commit offset+1，不存在加错的问题 |
| partition.assignment.strategy           | 默认range，可选round   robin。最好是round robin，能够保证分区是被均等分配给消费者的 |
| max.poll.records                        | 最大获取的message数量                                        |
| receive.buffer.bytes和send.buffer.bytes | TCP缓冲区大小。如果延迟高，可以设置大一些                    |

# 分区
kafka使用自己实现的散列，决定message到哪个分区；
也可以指定message到哪个分区

# 序列化
要指定序列化的类型，发送和读取kafka的message都需要序列化和反序列化

# JVM配置
也可以配置垃圾回收G1 GC等

# Zookeeper
Kafka使用zookeeper来保存broker，topic和partition的信息
Kafka最新版本，偏移量是提交在broker上的；但是在老版本，是在zookeeper上的。
多个kafka集群可以共享一个zookeeper群组，但是最好不要与其他应用程序公用zookeeper

# Kafka与azure service bus的差异
kafka的性能卓越，但是消息的transaction和消费高级应用方面比较弱。

|                 | Azure Service Bus                                            | Kafka                                                        |
| --------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 消费端事务      | Azure service bus提供了更好的消费端事务机制。每个message都可以被单独提交事务 | kafka只能提交偏移量，而偏移量是大段的，意味着多个消息一起被提交。   CommitSync会阻塞，但是能保证所有的偏移量被提交了。   CommitAsync性能好，但是非常混乱，因为可能导致大的偏移量，早于小的偏移量被提交。此时一旦crash，那么message就丢失了。 |
| 重试            | 一个消息超时后可返回队列，可以按消息重试     可以renew消息，防止超时 | 一组消息超时后返回队列，按组重试     不能防止超时            |
| 消费端的个数    | 在最大连接数以内即可。应该是5000                             | 最多是partition相等                                          |
| 性能            | 一般                                                         | 每秒百万级                                                   |
| 多个queue的事务 | 支持                                                         | 不支持                                                       |

# 复制

| 解释     | kafka有多个topic，每个topic都有多个分区，每个分区都有多个副本。 |
| -------- | ------------------------------------------------------------ |
| Leader   | 每个分区都有个首领，所有的生产和消费消息，都是走的leader，从而保证数据的一致性 |
| Follower | 副本，不参与与生产者和消费者的任何操作。只是请求leader获取消息             如果副本在﻿replica.lag.time.max.ms的时间范围内，没有同步消息，就会被leader认为是不同步     只有同步的副本，才能在leader失效时，称为新的leader |
| 首选首领 |                                                              |

# Kafka时如何保证数据不丢失

Partition有replica，topic的partition的replica是分布在不同的broker上的。
Kafka有replica和acks来保证消息不丢失。当lead失效时，会从replica中选举出新的lead来提供服务；acks设置为all时，当lead和replica都写入成功，才返回成功给客户端。
当消费消息时，客户端可以自动提交或者手动在事务里提交offset，这样客户端一旦失效，就会由其它客户端接手继续从offset消费。