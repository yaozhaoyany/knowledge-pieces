Kafka的topic是什么

把 Kafka 想象成一个超大的邮局系统：

Topic = 分类信箱
Kafka 集群（邮局）
├── Topic: orders        ← 所有订单相关的消息投到这个信箱
├── Topic: payments      ← 所有支付相关的消息投到这个信箱
├── Topic: notifications ← 所有通知相关的消息投到这个信箱
└── Topic: user-events   ← 所有用户行为日志投到这个信箱
Producer（寄信人） 指定往哪个 Topic 发消息，Consumer（收信人） 指定订阅哪个 Topic 来消费消息。彼此之间完全解耦——寄信人不需要知道谁会来取信。

Topic 内部结构：分区 (Partition)
每个 Topic 又被切成多个 Partition（分区），这是 Kafka 高吞吐量的秘密武器：

Topic: orders（3 个分区）
┌─────────────────────┐
│ Partition 0          │ → [消息0] [消息3] [消息6] [消息9]  ...
├─────────────────────┤
│ Partition 1          │ → [消息1] [消息4] [消息7] [消息10] ...
├─────────────────────┤
│ Partition 2          │ → [消息2] [消息5] [消息8] [消息11] ...
└─────────────────────┘
每个 Partition 是一个有序的、不可变的消息队列（只能追加，不能修改已有消息）。

为什么要分区？
并行消费！ 如果 Topic 只有 1 个分区，那只能有 1 个 Consumer 去消费它（因为每条消息在同一个 Consumer Group 里只能被消费一次）。但如果有 3 个分区，就可以安排 3 个 Consumer 各盯一个分区，吞吐量直接翻 3 倍：

Partition 0 ──→ Consumer 实例 A
Partition 1 ──→ Consumer 实例 B
Partition 2 ──→ Consumer 实例 C
这就是我们在 

topic.yaml
 里配了 partitions: 3 的原因——为后续 Day 4 写 order-consumer 时做多副本并行消费做好准备。

跟传统消息队列（RabbitMQ）的关键区别
特性	Kafka Topic	RabbitMQ Queue
消费后是否删除	❌ 不删除，保留到过期时间	✅ 消费后立即删除
能否重复消费	✅ 随时回放历史消息（Consumer 调整 offset 即可）	❌ 消费了就没了
设计定位	事件日志流（类似 Git 的 commit log）	任务分发队列
这就是为什么 Kafka 设置了 retention.ms: 604800000（7 天保留）——消息不会消费完就消失，它就像一条永不删除的时间线，任何 Consumer 都可以回到过去的任意时间点重新消费。