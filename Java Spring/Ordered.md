`Spring Ordered` 是 Spring 用来**控制同类组件执行顺序的接口/规范**，本质上就是：

👉 **给 Bean 排队用的优先级机制**

---

# 一、核心结论（先记这个）

👉 **Ordered = 控制 Spring 容器中 Bean 的执行顺序（越小越优先）**

---

# 二、基本定义

```java
public interface Ordered {
    int getOrder();
}
```

或者常量方式：

```java
public interface Ordered {
    int HIGHEST_PRECEDENCE = Integer.MIN_VALUE;
    int LOWEST_PRECEDENCE = Integer.MAX_VALUE;
}
```

---

# 三、规则（非常重要）

👉 **数值越小 → 优先级越高**

```text
-100  >  0  >  100
（先执行）    （后执行）
```

---

# 四、典型使用场景

Ordered 主要用于“同类型组件链”：

---

## 1️⃣ Filter 顺序（Servlet Filter）

```java
@Component
@Order(1)
public class LogFilter implements Filter {}
```

```java
@Component
@Order(2)
public class AuthFilter implements Filter {}
```

👉 执行顺序：

```text
LogFilter → AuthFilter
```

---

## 2️⃣ Interceptor 顺序

```java
registry.addInterceptor(new A()).order(1);
registry.addInterceptor(new B()).order(2);
```

---

## 3️⃣ AOP 切面顺序（非常关键）

```java
@Aspect
@Order(1)
class TxAspect {}
```

```java
@Aspect
@Order(2)
class LogAspect {}
```

👉 执行顺序：

```text
LogAspect → TxAspect → Method → TxAspect → LogAspect
```

（AOP 是双向嵌套）

---

## 4️⃣ Spring Security Filter Chain（重点）

```text
SecurityFilterChain = Ordered 控制
```

---

# 五、@Order vs Ordered 接口

## 1️⃣ @Order（更常用）

```java
@Order(1)
@Component
class A {}
```

---

## 2️⃣ Ordered 接口

```java
class A implements Ordered {
    public int getOrder() {
        return 1;
    }
}
```

---

## ✔ 推荐

👉 Spring 项目一般用：

> @Order（简单）  
> Ordered（框架/底层）

---

# 六、本质理解（非常重要）

👉 Ordered 控制的是：

## ✔ “同一类扩展点的执行顺序”

比如：

- Filter 链
    
- Interceptor 链
    
- AOP 切面链
    
- Event Listener
    
- BeanPostProcessor
    

---

# 七、和你前面学的链式结构关系

这是关键理解👇

|概念|本质|
|---|---|
|Ordered|排队规则|
|Filter Chain|执行结构|
|AOP|嵌套结构|
|Pipeline|数据流|

👉 Ordered 只是：

> **决定链的顺序，不决定链结构**

---

# 八、一个经典执行示例（面试必会）

## Filter + Order

```text
Order(1) LogFilter
Order(2) AuthFilter
Order(3) TraceFilter
```

执行：

```text
Log → Auth → Trace → Controller → Trace → Auth → Log
```

👉 注意：

- 顺序由 Ordered 控制
    
- 执行结构由 Filter 决定（双向）
    

---

# 九、一个容易混淆点（重点）

## ❌ Ordered ≠ 执行逻辑

Ordered 只做一件事：

👉 **排序，不负责执行方式**

---

# 十、终极一句话总结

👉 **Ordered = Spring 中的“优先级排序规则”，决定同类 Bean 的执行顺序（数值越小优先级越高）**

---

# 十一、如果你想继续深入（推荐）

我可以帮你讲一个非常重要的高级点：

👉 **Spring 是如何把 Ordered + List + 反射 → 自动组装成 FilterChain / AOP Chain 的**

这个就是 Spring 源码设计核心 👍