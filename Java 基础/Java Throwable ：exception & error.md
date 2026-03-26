这个是 Java 异常体系的**核心三层结构**，你必须彻底搞懂（面试高频 + 实战必用）。

---

# 一、整体结构（先建立全局观）

```text
Throwable
├── Error        （严重错误，JVM级别）
└── Exception    （程序异常，可以处理）
     ├── RuntimeException（运行时异常）
     └── 其他异常（编译时异常）
```

👉 一句话总结：

👉 **Throwable = 所有异常的父类**

---

# 二、Throwable（根类）

## 1️⃣ 本质

👉 所有异常和错误的父类

```java
public class Throwable
```

---

## 2️⃣ 提供的核心能力

```java
e.getMessage();     // 错误信息
e.printStackTrace();// 打印堆栈
```

---

## 3️⃣ 两大子类

- Exception（可处理）
    
- Error（不可恢复）
    

---

# 三、Error（严重错误）

## 1️⃣ 特点

👉 JVM 层面的错误  
👉 一般**不能处理，也不应该处理**

---

## 2️⃣ 常见例子

```java
StackOverflowError
OutOfMemoryError
```

---

## 3️⃣ 举例说明

```java
public class Test {
    public static void main(String[] args) {
        main(args); // 无限递归
    }
}
```

👉 结果：

```text
StackOverflowError
```

---

## 4️⃣ 是否要 catch？

❌ 不建议

```java
catch (Error e) {} // 基本没意义
```

👉 因为程序已经“崩了”

---

# 四、Exception（异常）

## 1️⃣ 特点

👉 程序运行中出现的问题  
👉 **可以处理**

---

## 2️⃣ 分类（重点）

---

## 🔹1️⃣ 运行时异常（RuntimeException）

👉 不强制处理（编译不检查）

```java
NullPointerException
ArrayIndexOutOfBoundsException
ArithmeticException
```

---

### 示例

```java
String s = null;
s.length(); // NPE
```

---

👉 特点：

- 编译通过
    
- 运行时报错
    

---

## 🔹2️⃣ 编译时异常（Checked Exception）

👉 **必须处理**

```java
IOException
SQLException
```

---

### 示例

```java
FileInputStream fis = new FileInputStream("a.txt");
```

👉 编译报错：

```text
必须捕获或声明抛出
```

---

### 两种解决方案

#### ✔ try-catch

```java
try {
    ...
} catch (IOException e) {
}
```

#### ✔ throws

```java
public void test() throws IOException {
}
```

---

# 五、三者对比总结（非常重要）

|类型|是否必须处理|是否可恢复|示例|
|---|---|---|---|
|Throwable|-|-|所有异常|
|Error|❌|❌|OOM|
|Exception|✅|✅|IO异常|

---

# 六、为什么要这样设计？（本质）

👉 Java 把问题分成两类：

---

## 1️⃣ 程序员问题（RuntimeException）

👉 比如：

- 空指针
    
- 数组越界
    

👉 特点：

❌ 不强制你处理  
👉 因为是你代码写错了

---

## 2️⃣ 外部问题（Checked Exception）

👉 比如：

- 文件不存在
    
- 网络异常
    

👉 特点：

✔ 强制处理  
👉 因为不可控

---

# 七、开发中最佳实践（重点）

## 1️⃣ 不要乱 catch Exception

❌ 错误写法：

```java
catch (Exception e) {}
```

👉 会吞异常

---

## 2️⃣ 优先处理具体异常

```java
catch (IOException e) {}
```

---

## 3️⃣ RuntimeException 一般不 catch

👉 直接修代码

---

## 4️⃣ 自定义异常

```java
public class MyException extends RuntimeException {
}
```

👉 用于业务异常

---

# 八、面试高频问题

---

## ❓1：Exception 和 Error 区别？

👉 答：

- Exception 可处理
    
- Error JVM级错误，不可恢复
    

---

## ❓2：RuntimeException 和普通 Exception 区别？

👉 答：

- RuntimeException 不强制处理
    
- 其他 Exception 必须处理
    

---

## ❓3：Throwable 为什么存在？

👉 答：

> 统一所有异常体系，提供公共方法（堆栈、信息）

---

# 九、你要形成的认知模型（很重要）

```text
Throwable
   ↓
系统问题 → Error（别管）
   ↓
程序问题 → Exception（要处理）
       ↓
    可预期 → Checked（必须处理）
    程序bug → Runtime（修代码）
```

---

# 十、一句话总结

👉 **Error 是 JVM 崩溃级问题**  
👉 **Exception 是程序可处理问题**  
👉 **Throwable 是它们的父类**

---

如果你想进阶（非常建议）👇

我可以带你搞清楚：

👉 Spring 里为什么大量用 RuntimeException  
👉 事务回滚为什么只对 RuntimeException 生效  
👉 全局异常处理（@ControllerAdvice）底层原理

这些都是你做后端必须掌握的“进阶关键点”。