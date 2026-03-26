这三个概念是 Java 并发编程和内存管理中的核心知识点。我们逐一深度拆解：

---

### 1. 什么是 OOM (Out Of Memory)？

**OOM** 全称 **`java.lang.OutOfMemoryError`**。简单来说，就是 **JVM（Java 虚拟机）由于没有足够的内存来分配给新对象，且垃圾回收器（GC）也无法释放更多空间时抛出的错误。**

在线程池场景下，最常见的 OOM 有两种情况：

1.  **堆溢出 (Heap Space)**：
    *   **原因**：如果你使用 `Executors.newFixedThreadPool`，它默认使用无界队列（`LinkedBlockingQueue`，容量为 $2^{31}-1$）。如果生产消息的速度远快于消费速度，数百万个任务会堆积在队列里，占满堆内存。
    *   **结果**：报错 `java.lang.OutOfMemoryError: Java heap space`。

2.  **线程溢出 (Unable to create native thread)**：
    *   **原因**：如果你使用 `Executors.newCachedThreadPool`，它的最大线程数是 `Integer.MAX_VALUE`。每来一个任务就开一个新线程，每个线程默认占 1MB 左右的内存。当线程开到几千个时，操作系统可能就不允许再开线程了。
    *   **结果**：报错 `java.lang.OutOfMemoryError: unable to create new native thread`。

---

### 2. Atomic（原子）变量的含义

`AtomicInteger`、`AtomicBoolean` 等属于 `java.util.concurrent.atomic` 包。其核心含义是：**保证单个变量的操作是“原子性”的，且线程安全。**

#### 为什么需要它？
普通的 `int i = 0; i++;` 看起来是一行代码，但在 CPU 底层其实分为三步：
1.  读取 `i` 的值。
2.  给 `i` 加 1。
3.  将新值写回内存。
如果在第一步和第三步之间，另一个线程也改了 `i`，那么结果就会出错（丢失更新）。

#### 原理：CAS (Compare And Swap)
Atomic 变量不使用重量级的 `synchronized` 锁，而是使用 **CAS** 这种无锁算法（靠硬件指令支持）：
*   它在修改时会对比：“我认为现在内存里的值应该是 5，如果是 5 我才改成 6”。
*   如果被别人改了，它就失败并重试（自旋）。这样效率比加锁高得多。

---

### 3. 为什么在 Lambda 表达式中使用 Atomic 变量？

这是初学者最常遇到的痛点。你在 Lambda 内部如果直接修改一个普通局部变量，编译器会报错：
> "Variable used in lambda expression should be final or effectively final"

#### 根本原因：变量捕获 (Variable Capture)
1.  **生命周期不一致**：Lambda 表达式（比如线程池里的任务）可能会在当前方法结束后很久才执行。此时方法里的局部变量（如 `int index`）早已在栈上销毁了。
2.  **副本机制**：为了让 Lambda 能访问这个变量，Java 会把这个变量的**快照（副本）**存一份到 Lambda 对象内部。
3.  **一致性要求**：如果 Java 允许你在 Lambda 内部修改这个变量，或者在外部修改它，就会导致“副本”和“本体”的值不统一，造成极其混乱的并发问题。

**因此，Java 强制要求：Lambda 引用的外部局部变量必须是 `final`（不可变）的。**

#### 解决方案：使用 AtomicInteger
虽然我们不能改变变量的**引用（指针）**，但我们可以改变引用对象内部的**状态**。

```java
// 错误写法：
int count = 0;
runnable -> {
    count++; // 编译报错！count 不是 final
}

// 正确写法：
AtomicInteger count = new AtomicInteger(0);
runnable -> {
    count.incrementAndGet(); // 编译通过！
}
```

*   **对 Lambda 来说**：`count` 这个变量名（引用）始终指向同一个 `AtomicInteger` 对象，它的“指向”没变，所以符合 `final` 的要求。
*   **对业务逻辑来说**：我们通过 `incrementAndGet()` 成功修改了对象内部的数值，实现了计数功能。
*   **对线程安全来说**：因为它是 `AtomicInteger`，多个线程同时创建线程并执行该 Lambda 时，计数器依然是准确的。

### 总结
*   **OOM** 是内存耗尽的灾难，手动配置线程池参数（如你代码中做的）是为了防止它。
*   **Atomic** 是为了不加锁也能安全地修改变量。
*   **Lambda 中使用 Atomic** 是为了绕过“局部变量必须为 final”的限制，并保证多线程下的计数准确。