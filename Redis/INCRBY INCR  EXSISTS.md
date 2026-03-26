这三个命令都是 Redis 里**非常高频、面试必问、实战必用**的基础命令，我帮你按「作用 + 示例 + 注意点 + 场景」讲清楚 👇

---

# 一、INCR（自增 +1）

## ✅ 作用

👉 对 key 的值 **+1**

```bash
INCR key
```

---

## ✅ 示例

```bash
SET count 0
INCR count   → 1
INCR count   → 2
```

---

## ❗ 特点

* key 不存在 → **自动初始化为 0，再 +1**
* 必须是整数，否则报错

---

## ❌ 错误情况

```bash
SET count "abc"
INCR count
```

👉 报错（不是数字）

---

## 🎯 使用场景

* 访问量统计
* 点赞数
* ID 生成（简单场景）

---

# 二、INCRBY（指定步长自增）

## ✅ 作用

👉 按指定值增加

```bash
INCRBY key increment
```

---

## ✅ 示例

```bash
SET count 10
INCRBY count 5   → 15
INCRBY count -3  → 12
```

---

## ❗ 特点

* 支持负数（等价于 DECR）
* key 不存在 → 默认从 0 开始

---

## 🎯 使用场景

* 批量计数
* 积分系统
* 库存变化

---

# 三、EXISTS（判断 key 是否存在）

## ✅ 作用

👉 判断 key 是否存在

```bash
EXISTS key
```

---

## ✅ 示例

```bash
SET name Alice
EXISTS name   → 1
EXISTS age    → 0
```

---

## ✅ 多 key（Redis 3+）

```bash
EXISTS k1 k2 k3
```

👉 返回：

👉 **存在的 key 数量**

---

## 示例

```bash
SET k1 v1
SET k2 v2

EXISTS k1 k2 k3 → 2
```

---

# 四、三个命令对比

| 命令     | 作用   | 特点   |
| ------ | ---- | ---- |
| INCR   | +1   | 原子操作 |
| INCRBY | 指定增量 | 更灵活  |
| EXISTS | 判断存在 | 返回数量 |

---

# 五、组合使用（非常重要🔥）

## 👉 限流（简单版）

```bash
INCR api_count
EXPIRE api_count 60
```

---

## 👉 判断是否存在再操作

```bash
EXISTS key
```

👉 常用于：

* 防止重复创建
* 缓存判断

---

# 六、核心特性（面试点🔥）

👉 **INCR / INCRBY 是原子操作**

原因：

👉 Redis 单线程执行命令

---

# 七、常见坑

## ❗ 1. INCR + EXPIRE 非原子

```bash
INCR count
EXPIRE count 60
```

👉 ❌ 可能出现：

* INCR 成功
* EXPIRE 没执行（程序挂了）

---

## ✅ 解决方案

```bash
SET count 1 EX 60 NX
```

或：

👉 Lua 脚本

---

## ❗ 2. EXISTS ≠ 判断业务状态

👉 只判断 key 是否存在
👉 不代表值是否有效

---

# 八、一句话总结

👉

* **INCR / INCRBY：原子计数器**
* **EXISTS：判断 key 是否存在（返回数量）**

---

# 九、给你一个后端实战建议（很关键🔥）

👉 高并发场景：

* 计数 → 用 INCR ✔
* 限流 → INCR + 过期 ✔
* 锁 → 不要用 EXISTS，要用 `SET NX` ✔

---

如果你想继续深入（强烈推荐🔥）：

我可以帮你讲：

* 🔥 Redis 限流完整实现（滑动窗口）
* 🔥 分布式 ID 生成（雪花算法 vs INCR）
* 🔥 为什么 INCR 比数据库自增快（底层原理）

直接说 👍
