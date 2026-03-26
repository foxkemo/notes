你这个问题其实指的是：  
👉 **线程在使用 `synchronized` / `ReentrantLock` 时，线程状态的三种转换**  
👉 以及它们在“阻塞 / 等待 / 运行”之间是怎么切换的

我帮你用**统一模型 + 两种锁对比**讲清楚👇

---

# 一、线程三种核心状态（先统一概念）

👉 Java 并发里，最关键的三种状态：

```text
RUNNABLE  （运行/就绪）
BLOCKED   （等锁）
WAITING   （主动等待）
```

---

# 二、synchronized 的三种状态转换（重点🔥）

---

## 1️⃣ 获取锁失败 → BLOCKED

```java
synchronized (obj) {
    // 临界区
}
```

---

👉 如果锁被占用：

```text
RUNNABLE → BLOCKED
```

👉 原因：

- JVM 让线程进入 Monitor 的 EntryList
    
- OS 层面阻塞
    

---

---

## 2️⃣ 获取锁成功 → RUNNABLE

```text
BLOCKED → RUNNABLE
```

👉 锁释放后：

- JVM 唤醒线程
    
- 进入可运行状态
    

---

---

## 3️⃣ 调用 wait() → WAITING

```java
synchronized (obj) {
    obj.wait();
}
```

---

👉 状态变化：

```text
RUNNABLE → WAITING
```

👉 同时：

- 释放锁（关键）
    

---

---

## 4️⃣ 被唤醒 → BLOCKED → RUNNABLE

```java
obj.notify();
```

---

👉 流程：

```text
WAITING → BLOCKED → RUNNABLE
```

👉 原因：

- 被唤醒后还要重新抢锁
    

---

---

## 🔥 synchronized 状态流转总结

```text
RUNNABLE
   ↓（抢锁失败）
BLOCKED
   ↓（抢到锁）
RUNNABLE
   ↓（wait）
WAITING
   ↓（notify）
BLOCKED
   ↓（获取锁）
RUNNABLE
```

---

# 三、ReentrantLock 的三种状态转换（核心区别🔥）

---

## 1️⃣ 获取锁失败 → WAITING（不是 BLOCKED）

```java
lock.lock();
```

---

👉 如果失败：

```text
RUNNABLE → WAITING
```

👉 原因：

- AQS 调用：
    

```java
LockSupport.park();
```

---

👉 注意：

> ❗ 这里不是 BLOCKED

---

---

## 2️⃣ 获取锁成功 → RUNNABLE

```text
WAITING → RUNNABLE
```

👉 通过：

```java
LockSupport.unpark();
```

---

---

## 3️⃣ Condition.await() → WAITING

```java
condition.await();
```

---

👉 状态：

```text
RUNNABLE → WAITING
```

👉 同时：

- 释放锁
    

---

---

## 4️⃣ signal() 唤醒 → WAITING → RUNNABLE

```java
condition.signal();
```

---

👉 流程：

```text
WAITING → WAITING（入AQS队列） → RUNNABLE
```

👉 注意：

- 不会变成 BLOCKED
    

---

---

## 🔥 ReentrantLock 状态流转总结

```text
RUNNABLE
   ↓（获取锁失败）
WAITING（park）
   ↓（unpark）
RUNNABLE
   ↓（await）
WAITING
   ↓（signal）
WAITING（队列中）
   ↓（获取锁）
RUNNABLE
```

---

# 四、核心区别总结（非常重要🔥）

---

## 1️⃣ 阻塞方式不同

||synchronized|ReentrantLock|
|---|---|---|
|等锁失败|BLOCKED|WAITING|
|实现|OS阻塞|park/unpark|

---

---

## 2️⃣ 唤醒机制

||synchronized|Lock|
|---|---|---|
|唤醒方式|notify|unpark|
|是否精准|❌ 随机|✅ 精准|

---

---

## 3️⃣ 状态转换路径

---

### 🔒 synchronized

```text
WAITING → BLOCKED → RUNNABLE
```

---

### 🔒 ReentrantLock

```text
WAITING → RUNNABLE
```

👉 少了一次 BLOCKED

---

# 五、一个本质理解（高级🔥）

---

👉 synchronized：

> **依赖 OS 的阻塞/唤醒机制**

---

👉 ReentrantLock：

> **依赖 JVM 的 park/unpark（用户态控制）**

---

👉 所以：

- synchronized → 重量级（涉及内核）
    
- Lock → 更轻量
    

---

# 六、面试总结（直接背🔥）

---

> synchronized 获取锁失败进入 BLOCKED 状态，wait 进入 WAITING，被唤醒后需重新竞争锁；  
> ReentrantLock 获取锁失败进入 WAITING（park），通过 unpark 唤醒，避免了 BLOCKED 状态，调度更灵活。

---

# 七、再帮你升维一句（非常重要）

---

> **synchronized = “阻塞式锁”**  
> **ReentrantLock = “挂起式锁（park）”**

---

# 八、如果你再往下走（强烈建议🔥）

你已经接近并发核心了，下一步建议：

- AQS 源码（state + CLH队列）
    
- LockSupport 原理（park/unpark）
    
- JVM 线程状态（6种完整状态）
    

---

如果你愿意，我可以帮你：  
👉 画一个 **完整线程状态转换图（面试神器）**  
👉 或带你逐行拆 **ReentrantLock源码执行流程（超清晰🔥）**