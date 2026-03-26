我帮你把 **Redis 字符串（String）最常用命令 + 过期时间 + 自增操作**系统梳理一遍，直接对应你后端开发常用场景 👇

---

# 一、字符串基本命令（String）

## 1️⃣ 设置值

```bash
SET key value
```

例：

```bash
SET name Alice
```

---

## 2️⃣ 获取值

```bash
GET key
```

---

## 3️⃣ 设置多个

```bash
MSET k1 v1 k2 v2
```

---

## 4️⃣ 获取多个

```bash
MGET k1 k2
```

---

## 5️⃣ 仅当不存在时设置（分布式锁常用🔥）

```bash
SET key value NX
```

---

## 6️⃣ 仅当存在时设置

```bash
SET key value XX
```

---

# 二、设置过期时间（重点🔥）

Redis 支持两种方式：

---

## 1️⃣ 设置时直接带过期时间（推荐）

```bash
SET key value EX 60
```

👉 60 秒过期

---

### 毫秒版本：

```bash
SET key value PX 1000
```

---

### 示例（分布式锁经典写法🔥）

```bash
SET lock_key uuid EX 30 NX
```

👉 含义：

* 不存在才设置
* 30 秒自动过期

---

---

## 2️⃣ 单独设置过期时间

```bash
EXPIRE key seconds
```

---

### 示例

```bash
SET name Alice
EXPIRE name 60
```

---

---

## 3️⃣ 查看剩余时间

```bash
TTL key
```

返回：

* `>0`：剩余秒数
* `-1`：没有过期时间
* `-2`：key 不存在

---

---

## 4️⃣ 取消过期时间

```bash
PERSIST key
```

---

# 三、自增命令（重点🔥）

👉 Redis 的 String 可以当“计数器”

---

## 1️⃣ 自增（+1）

```bash
INCR key
```

---

### 示例

```bash
SET count 0
INCR count   → 1
INCR count   → 2
```

---

## 2️⃣ 自减（-1）

```bash
DECR key
```

---

## 3️⃣ 指定步长自增

```bash
INCRBY key 10
```

---

## 4️⃣ 浮点自增

```bash
INCRBYFLOAT key 0.5
```

---

# 四、组合使用（非常重要🔥）

## 👉 带过期的计数器（限流经典）

```bash
INCR visit_count
EXPIRE visit_count 60
```

👉 统计 1 分钟访问量

---

⚠️ 问题：

👉 不是原子操作 ❗

---

## ✅ 正确写法（推荐）

```bash
SET visit_count 1 EX 60 NX
```

或：

👉 用 Lua / 或业务控制

---

# 五、实际应用场景（你后端会用）

---

## 1️⃣ 分布式锁

```bash
SET lock_key uuid EX 30 NX
```

---

## 2️⃣ 访问计数

```bash
INCR page_view
```

---

## 3️⃣ 限流

```bash
INCR api_count
EXPIRE api_count 60
```

---

## 4️⃣ 验证码

```bash
SET code 123456 EX 300
```

---

# 六、总结表（重点记住）

| 类型    | 命令               |
| ----- | ---------------- |
| 设置    | SET              |
| 获取    | GET              |
| 过期    | EX / PX / EXPIRE |
| 查看TTL | TTL              |
| 自增    | INCR             |
| 指定增量  | INCRBY           |
| 浮点    | INCRBYFLOAT      |

---

# 七、一句话总结

👉
**Redis String = 最基础结构 + 支持过期 + 支持原子自增，非常适合做缓存、计数器、分布式锁**

---

# 八、给你一个面试级补充（很关键🔥）

👉 Redis 自增为什么安全？

👉 因为：

👉 **单线程 + 命令原子性**

---

如果你想继续深入（强烈建议🔥）：

我可以帮你讲：

* 🔥 分布式锁完整实现（防误删 + 续期）
* 🔥 Redis 限流算法（滑动窗口 / 漏桶）
* 🔥 INCR + EXPIRE 为什么不安全（以及解决方案）

直接说 👍
