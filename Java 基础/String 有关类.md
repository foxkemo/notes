你说的 “StringBuilder Appender” 我理解你是想问 **Java 中 StringBuilder 的 append 系列方法**，或者是类似的拼接逻辑。我们可以从底层机制和用法两个角度来看。

---

## 1️⃣ `StringBuilder` 基本概念

- `StringBuilder` 是 **可变字符序列**（mutable sequence of characters），位于 `java.lang` 包。
    
- 它的特点：
    
    1. **线程不安全**（比 `StringBuffer` 快）
        
    2. 内部维护一个 **字符数组 `char[] value`**
        
    3. **长度可动态扩展**，append 操作不会产生新的对象
        

---

## 2️⃣ `append` 方法

`StringBuilder` 提供了多种重载的 `append` 方法，几乎可以拼接所有基本类型和对象：

```java
StringBuilder sb = new StringBuilder();

// 拼接字符串
sb.append("Hello");

// 拼接整数
sb.append(123);

// 拼接字符
sb.append('A');

// 拼接布尔值
sb.append(true);

// 拼接对象（调用 obj.toString()）
sb.append(new Date());

// 支持链式调用
sb.append(" World").append("!").append(2026);
```

- **返回值**：`StringBuilder` 本身 → 支持链式调用
    
- **底层机制**：
    
    1. 判断内部数组是否够长，如果不够 → 扩容
        
    2. 调用 `System.arraycopy` 或循环拷贝原来的字符
        
    3. 将新字符追加到数组末尾
        
- **扩容策略**：
    
    ```text
    newCapacity = oldCapacity * 2 + 2
    ```
    
    - 这个是经典的扩容公式，比字符串拼接（String + String）效率高很多，因为不产生中间对象。
        

---

## 3️⃣ StringBuilder vs StringBuffer vs String

|类型|线程安全|可变|使用场景|
|---|---|---|---|
|StringBuilder|❌|✅|单线程拼接字符序列|
|StringBuffer|✅|✅|多线程安全字符序列拼接|
|String|❌|❌|不可变，每次拼接都会创建新对象|

- 经典示例：
    

```java
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 1000; i++) {
    sb.append(i);
}
String result = sb.toString(); // 转回 String
```

- 对比直接 `String += i`，`StringBuilder` **性能高很多**。
    

---

## 4️⃣ `append` 内部源码示意

```java
public StringBuilder append(String str) {
    if (str == null) str = "null";
    int len = str.length();
    ensureCapacityInternal(count + len);
    str.getChars(0, len, value, count);
    count += len;
    return this;
}
```

- `ensureCapacityInternal` → 扩容检查
    
- `getChars` → 拷贝字符到内部数组
    
- `count` → 当前字符长度
    
- **返回 this** → 链式调用
    

---

💡 **总结**：

1. `StringBuilder` 是可变字符序列，高效拼接字符串。
    
2. `append` 方法可以接收各种类型，支持链式调用。
    
3. 内部通过数组扩容实现高性能，不像 `String +` 每次都生成新对象。
    
4. 多线程场景用 `StringBuffer`，单线程用 `StringBuilder`。
    

---

如果你愿意，我可以画一个 **StringBuilder append 的底层字符数组扩容示意图**，直观展示它是如何在数组上追加字符的。

你希望我画吗？