Redis 的 **List（列表）**本质就是一个**有序、可重复的双端链表（LinkedList）**，非常适合做：

* 队列（Queue）
* 栈（Stack）
* 消息队列（简单版 MQ）

我帮你从**核心命令 + 使用场景 + 常见坑**讲一遍👇

---

# 一、List 核心特点

* 有序（按插入顺序）
* 可重复
* 支持**头部 / 尾部操作（双端）**
* 底层：双向链表（Redis 7 后是 quicklist）

---

# 二、最核心命令（必须掌握）

## 1️⃣ 插入元素

### 👉 左插（头部）

```bash
LPUSH mylist a b c
```

结果：

```text
c b a
```

👉 类似“栈 push”

---

### 👉 右插（尾部）

```bash
RPUSH mylist a b c
```

结果：

```text
a b c
```

👉 类似“队列 enqueue”

---

## 🔥 面试重点

```text
LPUSH + LPOP   → 栈（后进先出）
LPUSH + RPOP   → 队列（先进先出）
```

---

## 2️⃣ 获取元素

### 👉 查看整个列表

```bash
LRANGE mylist 0 -1
```

✔ 常用写法：

| 写法    | 含义  |
| ----- | --- |
| 0 -1  | 全部  |
| 0 2   | 前3个 |
| -3 -1 | 后3个 |

---

## 3️⃣ 弹出元素

### 👉 左弹出

```bash
LPOP mylist
```

---

### 👉 右弹出

```bash
RPOP mylist
```

---

# 三、进阶命令

## 4️⃣ 获取长度

```bash
LLEN mylist
```

---

## 5️⃣ 获取指定位置元素

```bash
LINDEX mylist 0
```

---

## 6️⃣ 修改元素

```bash
LSET mylist 1 "newValue"
```

⚠️ 下标必须存在，否则报错

---

## 7️⃣ 删除元素（按值）

```bash
LREM mylist 1 "a"
```

参数解释：

```text
LREM key count value
```

| count | 含义   |
| ----- | ---- |
| >0    | 从左删  |
| <0    | 从右删  |
| =0    | 删除全部 |

---

## 8️⃣ 裁剪列表（非常重要）

```bash
LTRIM mylist 0 9
```

👉 只保留前 10 个元素

🔥 常用于：

> 限制列表大小（比如只保留最近100条）

---

# 四、阻塞操作（高级必问）

## 👉 阻塞弹出（模拟消息队列）

```bash
BLPOP mylist 10
```

含义：

* 如果 list 空
* **最多等待 10 秒**
* 有数据就立即返回

---

### 对应：

| 命令    | 含义    |
| ----- | ----- |
| BLPOP | 左阻塞弹出 |
| BRPOP | 右阻塞弹出 |

---

## 🔥 使用场景（面试高频）

👉 简单消息队列：

```text
生产者 → LPUSH
消费者 → BRPOP
```

✔ 优点：

* 简单
* 无需额外中间件

❗ 缺点：

* 不可靠（会丢数据）

---

# 五、典型使用模式

## 1️⃣ 栈（Stack）

```bash
LPUSH stack x
LPOP stack
```

---

## 2️⃣ 队列（Queue）

```bash
LPUSH queue x
RPOP queue
```

---

## 3️⃣ 消息队列（Blocking Queue）

```bash
LPUSH queue msg
BRPOP queue 0
```

---

## 4️⃣ 最新消息列表（类似微博）

```bash
LPUSH timeline msg
LTRIM timeline 0 99
```

👉 只保留最新 100 条

---

# 六、常见坑

## ⚠️ 1. LRANGE 是 O(n)

```bash
LRANGE mylist 0 -1
```

👉 大 list 会很慢

---

## ⚠️ 2. LSET 越界直接报错

---

## ⚠️ 3. List 不适合随机访问

👉 频繁 `LINDEX` 性能差

---

# 七、底层（加分点）

Redis List 实际结构：

```text
quicklist = linkedlist + ziplist
```

👉 优点：

* 节省内存
* 插入快

---

# 八、一句话总结

👉 **List = 双端队列，核心就是“左右进出”**

---

如果你想继续进阶（你做后端开发很关键），我可以帮你讲：

* List vs Stream（为什么现在 MQ 更推荐 Stream）
* Redis List 如何实现延迟队列
* 高并发下 List 会遇到的问题

这些都是面试加分点🔥
