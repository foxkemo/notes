Redis 的 **Stream（流）**是一个**比 List 更高级的消息队列结构**，从 Redis 5.0 引入，专门用来做：

> ✔ 消息队列（MQ）
> ✔ 日志流（类似 Kafka）
> ✔ 事件流处理

我帮你从**核心概念 → 命令 → 和 List 对比 → 面试点**一步讲清👇

---

# 一、Stream 是什么

👉 本质：

```text
有序 + 可持久化 + 可消费确认 的消息队列
```

和 List 最大区别：

👉 **消息不会被消费后删除（默认）**

---

# 二、核心结构（非常重要）

每条消息长这样：

```bash
XADD mystream * name zhang age 18
```

结构：

```text
ID         field-value
1670000000-0   name=zhang age=18
```

---

## 🔥 ID 解释

```text
时间戳-序号
```

例如：

```text
1670000000-0
1670000000-1
```

✔ 保证：

* 有序
* 唯一

---

# 三、核心命令

---

## 1️⃣ 添加消息（生产者）

```bash
XADD mystream * name zhang age 18
```

✔ `*` 表示自动生成 ID

---

## 2️⃣ 读取消息（简单模式）

```bash
XRANGE mystream - +
```

👉 从头到尾读取

---

```bash
XRANGE mystream 1670000000-0 1670000001-0
```

👉 按区间读

---

## 3️⃣ 阻塞读取（类似 MQ）

```bash
XREAD BLOCK 0 STREAMS mystream 0
```

✔ 含义：

* BLOCK 0：一直等
* 0：从头开始

---

# 四、消费者组（重点🔥）

👉 Stream 最核心能力！

---

## 4️⃣ 创建消费者组

```bash
XGROUP CREATE mystream mygroup 0
```

---

## 5️⃣ 消费消息

```bash
XREADGROUP GROUP mygroup c1 COUNT 1 STREAMS mystream >
```

参数解释：

| 参数      | 含义    |
| ------- | ----- |
| mygroup | 消费组   |
| c1      | 消费者   |
| >       | 只读新消息 |

---

## 6️⃣ 确认消息（ACK）

```bash
XACK mystream mygroup 1670000000-0
```

👉 告诉 Redis：

> 这条消息我处理完了 ✅

---

# 五、为什么比 List 强？

## 🔥 List 的问题

```text
LPOP 后数据就没了
```

👉 一旦消费者挂了：

❌ 数据丢失

---

## ✅ Stream 解决方案

* 消费后不删除
* 必须 ACK
* 支持“未确认消息重试”

---

# 六、Pending List（面试高频🔥）

👉 每个消费者组都有：

```text
Pending List（待确认消息）
```

查看：

```bash
XPENDING mystream mygroup
```

---

👉 如果消费者挂了：

可以：

```bash
XCLAIM
```

👉 把消息转给其他消费者

---

# 七、和 List 对比（重点总结）

| 特性    | List | Stream |
| ----- | ---- | ------ |
| 顺序    | ✔    | ✔      |
| 消费后删除 | ✔    | ❌      |
| 消费确认  | ❌    | ✔      |
| 消费者组  | ❌    | ✔      |
| 消息重试  | ❌    | ✔      |
| 适合场景  | 简单队列 | 专业 MQ  |

---

# 八、典型使用场景

## 1️⃣ 消息队列（推荐）

```text
生产者 → XADD
消费者 → XREADGROUP
处理 → XACK
```

---

## 2️⃣ 日志系统

类似：

* Kafka
* RocketMQ

---

## 3️⃣ 订单系统（防丢单）

👉 比如：

* 下单
* 支付
* 发货

---

# 九、常见坑

## ⚠️ 1. 不 ACK 会怎样？

👉 消息一直卡在 Pending

---

## ⚠️ 2. Stream 会无限增长

👉 需要裁剪：

```bash
XADD mystream MAXLEN 1000 * field value
```

---

## ⚠️ 3. ID 不能乱写

```bash
XADD mystream 1-0 ...
```

👉 可能报错（必须递增）

---

# 十、底层理解（加分）

Stream ≈

```text
Radix Tree（基数树） + 链表
```

👉 优点：

* 高效范围查询
* 按 ID 有序

---

# 十一、一句话总结

👉 **Stream = Redis 内置的可靠消息队列（带 ACK + 消费组）**

---

# 十二、面试总结一句话（建议背）

👉

> List 是简单队列，Stream 是带确认机制和消费者组的专业消息队列

---

如果你想继续深入（你做后端很关键🔥），我可以带你：

* Redis Stream vs Kafka（面试高频对比）
* 用 SpringBoot + Redis Stream 实现 MQ
* Stream 如何保证“至少一次 / 恰好一次”消费

这些都是中高级后端必问内容 💯
好，这里我给你一套 **Redis Stream 从创建 → 写入 → 创建消费者组 → 消费** 的完整实战示例（非常适合你现在做短链接统计/异步消费 👍）

---

# 一、核心概念先搞清（很重要）

| 概念             | 说明             |
| -------------- | -------------- |
| Stream         | 类似“日志流 / MQ”   |
| stream key     | 流的名字（类似 topic） |
| entry          | 一条消息           |
| consumer group | 消费者组           |
| consumer       | 组里的消费者         |

---

# 二、1️⃣ 创建 Stream（用 XADD）

👉 Redis Stream 是 **自动创建的**（第一次 XADD 就创建）

---

## 示例：创建 stream + 写入数据

```bash
XADD short_link_stream * url "https://xxx.com" userId "1001"
```

👉 解释：

```text
short_link_stream  = stream key
*                  = 自动生成ID（时间戳）
url/userId         = 字段
```

---

## 再插入几条

```bash
XADD short_link_stream * url "https://a.com" userId "1002"
XADD short_link_stream * url "https://b.com" userId "1003"
```

---

# 三、2️⃣ 创建消费者组（XGROUP）

👉 创建消费者组：

```bash
XGROUP CREATE short_link_stream group1 0 MKSTREAM
```

---

## 参数解释：

```text
short_link_stream  = stream
group1             = 消费者组名
0                  = 从头开始消费
MKSTREAM           = 如果 stream 不存在就创建
```

---

👉 如果只想消费“新消息”：

```bash
XGROUP CREATE short_link_stream group1 $ MKSTREAM
```

```text
$ = 只消费之后的新数据
```

---

# 四、3️⃣ 消费数据（XREADGROUP）

---

## 示例：消费者消费

```bash
XREADGROUP GROUP group1 consumer1 COUNT 1 BLOCK 0 STREAMS short_link_stream >
```

---

## 参数解释：

```text
GROUP group1 consumer1  = 组名 + 消费者名
COUNT 1                 = 一次取1条
BLOCK 0                 = 阻塞（类似队列）
STREAMS key >           = 读取新消息
```

---

👉 `>` 的含义：

```text
只读“未分配过”的新消息
```

---

# 五、4️⃣ ACK 确认消费（必须！）

```bash
XACK short_link_stream group1 167xxxxxxx-0
```

---

👉 含义：

> 告诉 Redis：这条消息我处理完了

---

# 六、完整流程总结

```text
生产者：
    XADD → 写入 Stream

消费者：
    XGROUP CREATE → 建组
    XREADGROUP → 读消息
    XACK → 确认
```

---

# 七、Java（Spring / Jedis / Redisson）示例

---

## 1️⃣ 生产消息

```java
stringRedisTemplate.opsForStream().add(
    "short_link_stream",
    Map.of(
        "url", "https://xxx.com",
        "userId", "1001"
    )
);
```

---

## 2️⃣ 创建消费者组

```java
stringRedisTemplate.opsForStream()
    .createGroup("short_link_stream", "group1");
```

---

## 3️⃣ 消费消息（核心）

```java
List<MapRecord<String, Object, Object>> messages =
    stringRedisTemplate.opsForStream().read(
        Consumer.from("group1", "consumer1"),
        StreamReadOptions.empty().count(1).block(Duration.ofSeconds(2)),
        StreamOffset.create("short_link_stream", ReadOffset.lastConsumed())
    );
```

---

## 4️⃣ ACK

```java
stringRedisTemplate.opsForStream().acknowledge(
    "short_link_stream",
    "group1",
    record.getId()
);
```

---

# 八、结合你现在的线程池（重点🔥）

你之前写了线程池：

👉 可以这样消费：

```java
executor.execute(() -> {
    while (true) {
        List<MapRecord<String, Object, Object>> messages = read();

        for (var msg : messages) {
            process(msg);
            ack(msg);
        }
    }
});
```

---

👉 本质：

```text
线程 = 消费者
Stream = 队列
task = 处理一条消息
```

---

# 九、几个关键坑（必须知道）

---

## ❗1️⃣ 不 ACK 会怎样？

👉 消息会进入：

```text
PEL（Pending Entries List）
```

👉 后果：

* 不会再被新消费者读
* 会“卡住”

---

---

## ❗2️⃣ 怎么处理异常消息？

👉 用：

```bash
XPENDING
XCLAIM
```

---

---

## ❗3️⃣ BLOCK 0 会怎样？

```text
一直阻塞（推荐）
```

👉 类似：

```java
while(true) { 阻塞等待 }
```

---

# 十、总结一句话

> Redis Stream 使用 XADD 写入消息，通过 XGROUP 创建消费者组，使用 XREADGROUP 消费，并通过 XACK 确认，实现类似 MQ 的消息队列模型。

---

# 十一、如果你要面试（加分说法）

> Redis Stream 是基于日志结构的消息队列，支持消费者组、消息确认（ACK）、Pending List 等特性，相比 List 更适合实现可靠消息消费。

---

如果你想，我可以帮你继续深入👇

👉 Redis Stream vs Kafka vs RabbitMQ 对比
👉 PEL（Pending List）底层原理
👉 如何实现“消息重试 + 死信队列”

这部分是你做分布式系统的关键 🚀
