
你这个问题本质是 **Java 函数式接口 + Lambda + 方法引用** 的核心内容（面试高频🔥）。我给你从“能用 → 会用 → 理解底层”一步讲清楚。

---

# 一、Consumer / Supplier 是什么？

它们都在：
👉 `java.util.function` 包

---

## 1️⃣ Consumer（消费型接口）

👉 **有输入，没有返回值**

```java
@FunctionalInterface
public interface Consumer<T> {
    void accept(T t);
}
```

---

### ✔ 示例

```java
Consumer<String> consumer = s -> System.out.println(s);
consumer.accept("hello");
```

👉 等价于：

```java
System.out.println("hello");
```

---

### ✔ 方法引用写法（重点）

```java
Consumer<String> consumer = System.out::println;
consumer.accept("hello");
```

👉 `System.out::println` 就是方法引用

---

## 2️⃣ Supplier（供给型接口）

👉 **没有输入，有返回值**

```java
@FunctionalInterface
public interface Supplier<T> {
    T get();
}
```

---

### ✔ 示例

```java
Supplier<String> supplier = () -> "hello";
System.out.println(supplier.get());
```

---

### ✔ 方法引用

```java
Supplier<Double> supplier = Math::random;
System.out.println(supplier.get());
```

---

# 二、方法引用（Method Reference）

👉 本质：**Lambda 的简化写法**

---

## 1️⃣ 基本语法

```java
类名::方法名
对象::方法名
```

---

# 三、方法引用的 4 种形式（重点🔥）

---

## 1️⃣ 对象::实例方法

```java
Consumer<String> c = System.out::println;
```

👉 等价：

```java
s -> System.out.println(s)
```

---

## 2️⃣ 类::静态方法

```java
Supplier<Double> s = Math::random;
```

👉 等价：

```java
() -> Math.random()
```

---

## 3️⃣ 类::实例方法（最容易懵❗）

```java
Function<String, Integer> f = String::length;
```

👉 等价：

```java
s -> s.length()
```

📌 规则：

> Lambda 参数 = 调用对象

---

## 4️⃣ 类::构造方法

```java
Supplier<User> s = User::new;
```

👉 等价：

```java
() -> new User()
```

---

# 四、Consumer + Supplier + 方法引用实战

---

## 示例 1：打印集合

```java
List<String> list = Arrays.asList("a", "b", "c");

list.forEach(System.out::println);
```

👉 `forEach` 需要 Consumer

---

## 示例 2：生成对象

```java
Supplier<User> supplier = User::new;
User user = supplier.get();
```

---

## 示例 3：结合 Optional（你之前问的）

```java
Optional.of("hello")
        .ifPresent(System.out::println);
```

👉 `ifPresent` 需要 Consumer

---

# 五、常见函数式接口对比（面试）

| 接口            | 参数 | 返回值     |
| ------------- | -- | ------- |
| Consumer<T>   | 1个 | 无       |
| Supplier<T>   | 无  | 1个      |
| Function<T,R> | 1个 | 1个      |
| Predicate<T>  | 1个 | boolean |

---

# 六、什么时候可以用方法引用？（核心规则）

👉 **当 Lambda 只是“调用一个方法”时，可以替换**

---

### ✔ 可以替换

```java
s -> System.out.println(s)
```

➡

```java
System.out::println
```

---

### ❌ 不可以

```java
s -> s + "abc"
```

👉 因为不是直接调用方法

---

# 七、你必须理解的本质（面试加分）

👉 方法引用不是新东西，本质还是 Lambda

编译后：

```java
System.out::println
```

👉 会变成类似：

```java
s -> System.out.println(s)
```

---

# 八、总结一句话

👉

* **Consumer = 吃数据（无返回）**
* **Supplier = 生产数据（无参数）**
* **方法引用 = Lambda 的语法糖**

---

# 九、面试经典问题

你可以准备这几个👇

1. Consumer 和 Supplier 区别
2. 方法引用有哪几种
3. 方法引用和 Lambda 的关系
4. `String::length` 为什么可以这样写（重点）
5. Optional + Consumer 的使用

---

如果你想，我可以给你出一套 **Lambda + 方法引用 + Stream 的面试题（带答案）**，这一块在 Java 后端面试里非常常见。
