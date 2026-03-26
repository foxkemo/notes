在 Redisson 里，分布式锁最核心的就是 **`RLock` + `getLock()`**。你可以把它理解成：
👉 “基于 Redis 的可重入分布式互斥锁”。

我给你从**用法 → 原理 → 场景 → 常见坑**系统讲清楚（面试级🔥）

---

# ✅ 一、基本用法（getLock）

## 📌 获取锁对象

```java
RLock lock = redissonClient.getLock("myLock");
```

👉 `"myLock"` 就是锁的 key（存在 Redis 里）

---

## 📌 加锁（阻塞）

```java
lock.lock();   // 一直等，直到拿到锁
```

---

## 📌 释放锁

```java
lock.unlock();
```

---

## ✅ 推荐写法（必须这样写）

```java
RLock lock = redissonClient.getLock("myLock");

try {
    lock.lock();
    
    // 业务逻辑
    System.out.println("执行任务");

} finally {
    lock.unlock();
}
```

---

# ✅ 二、非阻塞加锁（高并发常用）

```java
if (lock.tryLock()) {
    try {
        // 执行业务
    } finally {
        lock.unlock();
    }
}
```

---

## 📌 带超时版本（强烈推荐🔥）

```java
boolean success = lock.tryLock(3, 10, TimeUnit.SECONDS);
```

👉 参数解释：

| 参数 | 含义            |
| -- | ------------- |
| 3  | 最多等 3 秒拿锁     |
| 10 | 锁自动释放时间（避免死锁） |

---

# ✅ 三、getLock 本质做了什么？

```java
RLock lock = redissonClient.getLock("order_lock");
```

👉 实际：

```text
Redis Key: order_lock
Value: 当前线程ID
```

---

## 🔒 特性

### ✔ 可重入锁（像 synchronized）

```java
lock.lock();
lock.lock();  // 同一线程可以再次获取
```

---

### ✔ 自动续期（看门狗机制🔥）

默认：

```text
锁时间 = 30秒
```

👉 如果业务没执行完：

➡ Redisson 自动帮你续期（每10秒续一次）

---

# ✅ 四、使用场景（你项目里会用到）

---

## 1️⃣ 防止重复下单

```java
RLock lock = redissonClient.getLock("order:" + userId);
```

---

## 2️⃣ 短链接生成（你之前写的）

👉 防止生成重复 code：

```java
RLock lock = redissonClient.getLock("short_url_lock");
```

---

## 3️⃣ Redis 缓存击穿保护

```java
RLock lock = redissonClient.getLock("cache_lock");
```

---

## 4️⃣ 定时任务（避免多实例重复执行）

```java
RLock lock = redissonClient.getLock("job_lock");
```

---

# ✅ 五、锁粒度设计（非常关键🔥）

### ❌ 错误（全局锁）

```java
getLock("lock")
```

👉 性能差

---

### ✔ 正确（细粒度）

```java
getLock("order:" + orderId)
getLock("user:" + userId)
```

👉 提高并发

---

# ✅ 六、常见坑（面试必问⚠️）

---

## ❌ 1. 忘记释放锁

```java
lock.lock();
// 没 unlock → 死锁
```

---

## ❌ 2. 业务异常导致锁没释放

👉 必须用 `try-finally`

---

## ❌ 3. 锁时间不够

```java
lock.lock(5, TimeUnit.SECONDS);
```

👉 业务执行超过 5 秒 → 锁提前释放 → 并发问题

---

## ❌ 4. 不用 tryLock

👉 高并发下容易阻塞线程

---

## ❌ 5. 锁 key 设计不合理

👉 导致：

* 串行化严重
* 性能下降

---

# ✅ 七、进阶：公平锁

```java
RLock lock = redissonClient.getFairLock("fairLock");
```

👉 按顺序获取锁（FIFO）

---

# ✅ 八、和 Java 锁对比

| 特性   | synchronized | RLock |
| ---- | ------------ | ----- |
| 作用范围 | 单机           | 分布式   |
| 可重入  | ✔            | ✔     |
| 自动释放 | JVM          | Redis |
| 超时控制 | ❌            | ✔     |

---

# 🚀 总结一句话

```text
getLock = 根据 key 在 Redis 上创建一个分布式可重入锁
```

---

# 💡 给你一个实战建议（很重要）

你现在做后端项目：

👉 优先用：

```java
tryLock(等待时间, 锁时间)
```

👉 锁 key 一定带业务 ID：

```java
"short_url:" + code
```

---

如果你愿意，我可以帮你写一套**“短链接系统 + Redisson 分布式锁完整实现（防重复生成 + 防并发写 DB）”**，直接能写进你项目里。
