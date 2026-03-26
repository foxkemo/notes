不会报错 ❌

👉 **`SET key value NX` 在 key 已存在时，不会报错，而是“设置失败，返回 nil（或 null）”**

---

# 一、行为说明

```bash
SET key value NX
```

👉 含义：

* **NX = Not eXists**
* 只有当 key 不存在时才会设置

---

## ✅ 情况 1：key 不存在

```bash
SET name Alice NX
```

👉 返回：

```text
OK
```

👉 设置成功

---

## ❌ 情况 2：key 已存在

```bash
SET name Bob NX
```

👉 返回：

```text
(nil)
```

👉 设置失败（但**不会报错**）

---

# 二、为什么不会报错？

👉 Redis 设计原则：

* 命令失败 ≠ 异常
* 用返回值表示结果

---

👉 类似：

| 命令        | 失败行为   |
| --------- | ------ |
| GET 不存在   | 返回 nil |
| SET NX 失败 | 返回 nil |
| DEL 不存在   | 返回 0   |

---

# 三、Java 中的表现（你会用到🔥）

如果用 `StringRedisTemplate`：

```java
Boolean result = stringRedisTemplate.opsForValue()
    .setIfAbsent("key", "value");
```

---

## 返回值：

| 情况          | 返回    |
| ----------- | ----- |
| 成功          | true  |
| 失败（key 已存在） | false |

---

# 四、经典应用（分布式锁🔥）

```bash
SET lock_key uuid EX 30 NX
```

---

## 判断是否加锁成功：

* 返回 `OK` → 成功
* 返回 `nil` → 失败

---

# 五、容易踩坑点

## ❗ 不要这样写

```bash
SETNX key value
EXPIRE key 30
```

👉 ❌ 非原子操作（可能出问题）

---

## ✅ 正确写法

```bash
SET key value EX 30 NX
```

👉 原子操作 ✔

---

# 六、一句话总结

👉 **`SET NX` 在 key 已存在时不会报错，只是返回 nil 表示操作失败**

---

如果你想继续深入（很关键🔥）：

我可以帮你讲：

* 🔥 `SETNX` vs `SET NX` 区别
* 🔥 分布式锁误删问题（必须掌握）
* 🔥 Redisson 锁底层实现（你之前问过 RLock）

直接说 👍
