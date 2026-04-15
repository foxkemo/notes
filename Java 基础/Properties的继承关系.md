在 Java 中，`Properties` 和 `Map` 之间有着紧密的继承关系，但在用途和对不存在键（Key）的处理机制上有一些细微且重要的区别。

---

### 1. 关系与区别

#### 继承关系
`Properties` 类继承自 `Hashtable`，而 `Hashtable` 实现了 `Map<Object, Object>` 接口。
> `Properties` $\to$ `Hashtable` $\to$ `Map`

#### 主要区别

| 特性 | Map (如 HashMap) | Properties |
| :--- | :--- | :--- |
| **用途** | 通用的键值对存储。 | 主要用于读取/存储 `.properties` 配置文件。 |
| **泛型约束** | 键值可以是任何对象（如 `Map<String, Integer>`）。 | 虽然继承自 `Hashtable`，但设计初衷是**键和值都应为 String**。 |
| **持久化** | 无内置持久化方法（需手动处理）。 | 提供 `load()` 和 `store()` 直接操作文件。 |
| **线程安全** | `HashMap` 不安全，`ConcurrentHashMap` 安全。 | 它是**线程安全**的（因为它继承自 `Hashtable`）。 |
| **空值允许** | `HashMap` 允许一个 null 键和多个 null 值。 | **不允许** null 键或 null 值（会抛出 `NullPointerException`）。 |

---

### 2. 获取不存在的键（Key）的处理

这是两者在使用中最明显的行为差异点。

#### A. 在 Map 中处理
如果你使用 `Map<String, String> map = new HashMap<>();`

1.  **`get(key)`**: 如果键不存在，返回 `null`。
    ```java
    String value = map.get("missingKey"); // 返回 null
    ```
2.  **`getOrDefault(key, defaultValue)`** (Java 8+): 如果键不存在，返回你指定的默认值。
    ```java
    String value = map.getOrDefault("missingKey", "DefaultValue"); // 返回 "DefaultValue"
    ```

#### B. 在 Properties 中处理
`Properties` 提供了专门的方法来处理字符串：

1.  **`getProperty(String key)`**: 如果键不存在，返回 `null`。
    ```java
    Properties prop = new Properties();
    String val = prop.getProperty("db.password"); // 如果没有该配置，返回 null
    ```
2.  **`getProperty(String key, String defaultValue)`**: 如果键不存在，返回指定的默认值。这是处理配置缺失的标准做法。
    ```java
    String port = prop.getProperty("db.port", "3306"); // 找不到则返回 "3306"
    ```
3.  **注意：** 尽管你可以通过继承关系调用 `get(key)`，但不建议在 `Properties` 上使用它，因为 `get` 返回的是 `Object` 类型，而 `getProperty` 返回的是 `String` 类型，更符合配置文件的语境。

---

### 3. Properties 的特有机制：默认属性列表

`Properties` 有一个特殊的构造函数，允许你传入另一个 `Properties` 对象作为“二级默认值”。

```java
Properties defaultProps = new Properties();
defaultProps.setProperty("timeout", "5000");

// 将 defaultProps 作为基础
Properties userProps = new Properties(defaultProps);

// 查找过程：先查 userProps，查不到再去 defaultProps 查
System.out.println(userProps.getProperty("timeout")); // 输出 5000
```

---

### 总结与最佳实践

1.  **如果你在做业务逻辑开发**：使用 `Map`（通常是 `HashMap` 或 `ConcurrentHashMap`），因为它支持泛型，效率更高，更现代。
2.  **如果你在读取配置文件**：使用 `Properties`。
3.  **处理不存在的键**：
    *   在 `Map` 中，优先使用 `getOrDefault()` 以避免繁琐的 `null` 检查。
    *   在 `Properties` 中，始终使用 `getProperty(key, defaultValue)` 来确保你的程序在缺少某个配置项时仍有回退方案。
4.  **警告**：不要向 `Properties` 中 `put` 非 String 类型的对象（尽管编译能过），否则在使用 `store` 保存文件或 `getProperty` 时会抛出类型转换异常。