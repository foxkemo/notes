我帮你把 **Redis Hash（哈希）常用命令**系统整理一遍，直接对应你后端开发使用场景 👇

---

# 一、Hash 是什么？

👉 类似 Java：

```java
Map<String, String>
```

👉 Redis 结构：

```text
key → field → value
```

---

## 示例

```bash
user:1 → {name: "Alice", age: "20"}
```

---

# 二、基本操作命令（核心🔥）

---

## 1️⃣ 设置字段

```bash
HSET key field value
```

---

### 示例

```bash
HSET user:1 name Alice
HSET user:1 age 20
```

---

## 2️⃣ 获取字段

```bash
HGET key field
```

---

### 示例

```bash
HGET user:1 name
```

---

## 3️⃣ 设置多个字段

```bash
HMSET key field value field value
```

⚠️ 新版本推荐：

```bash
HSET key field1 value1 field2 value2
```

---

---

## 4️⃣ 获取多个字段

```bash
HMGET key field1 field2
```

---

---

## 5️⃣ 获取所有字段和值

```bash
HGETALL key
```

---

👉 返回：

```text
name Alice age 20
```

---

# 三、字段判断 & 删除

---

## 1️⃣ 判断字段是否存在

```bash
HEXISTS key field
```

---

## 2️⃣ 删除字段

```bash
HDEL key field
```

---

# 四、计数操作（很重要🔥）

---

## 1️⃣ 自增（整数）

```bash
HINCRBY key field increment
```

---

### 示例

```bash
HINCRBY user:1 age 1
```

---

## 2️⃣ 自增（浮点）

```bash
HINCRBYFLOAT key field 0.5
```

---

# 五、获取结构信息

---

## 1️⃣ 字段数量

```bash
HLEN key
```

---

## 2️⃣ 所有字段

```bash
HKEYS key
```

---

## 3️⃣ 所有值

```bash
HVALS key
```

---

# 六、实际使用场景（你后端常用🔥）

---

## 1️⃣ 存对象（最常见）

```bash
HSET user:1 name Alice age 20 email xxx
```

👉 替代 JSON（简单场景）

---

---

## 2️⃣ 用户信息缓存

```bash
key: user:1001
field: name age score
```

---

---

## 3️⃣ 计数统计

```bash
HINCRBY article:1 viewCount 1
```

---

---

## 4️⃣ 商品库存（简单版）

```bash
HINCRBY product:1 stock -1
```

---

# 七、Hash vs String（重要）

| 对比 | String | Hash            |
| -- | ------ | --------------- |
| 结构 | 单值     | key-field-value |
| 更新 | 整体更新   | 局部更新            |
| 适合 | 简单值    | 对象              |

---

# 八、过期时间注意点（很重要❗）

👉 **Hash 只能对 key 设置过期，不能对 field 设置**

---

## 示例

```bash
EXPIRE user:1 60
```

👉 整个 hash 过期

---

❌ 不支持：

```bash
user:1.name 单独过期 ❌
```

---

# 九、Java 中使用（你重点）

```java
stringRedisTemplate.opsForHash().put("user:1", "name", "Alice");
```

---

## 获取

```java
Object name = stringRedisTemplate.opsForHash().get("user:1", "name");
```

---

## 自增

```java
stringRedisTemplate.opsForHash().increment("user:1", "age", 1);
```

---

# 十、总结（重点记住）

👉 核心命令：

```bash
HSET / HGET / HGETALL
HDEL / HEXISTS
HINCRBY
```

---

# 十一、一句话总结

👉 **Redis Hash = 适合存“对象结构”的 key-value 集合，支持字段级操作，但过期只能作用于整个 key**

---

# 十二、给你一个后端建议（很实用🔥）

👉 存对象时：

* 小对象 → Hash ✔
* 复杂对象 → JSON（String）✔

---

如果你想继续深入（你现在阶段很适合🔥）：

我可以帮你讲：

* 🔥 Hash vs JSON 存储性能对比
* 🔥 Redis 内存结构（ziplist / hashtable）
* 🔥 如何设计 Redis Key（大厂规范）

直接说 👍
