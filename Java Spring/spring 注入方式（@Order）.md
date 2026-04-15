

`@Primary` 和 `@Order` 看起来都像“优先级控制”，但它们**完全不是一类东西**，作用范围也不同。

---

# 一、核心结论（先记住）

|注解|作用对象|作用|
|---|---|---|
|`@Primary`|Bean 注入选择|**多个候选 Bean 时默认选谁**|
|`@Order`|Bean / List / AOP / 执行顺序|**多个 Bean 的排列顺序**|

👉 一句话：

- `@Primary` = “谁是默认用的”
    
- `@Order` = “谁先谁后”
    

---

# 二、@Primary（解决“选谁”问题）

## 1. 场景：多个 Bean 冲突

```java
public interface PayService {}
```

```java
@Service
@Primary
class AliPayService implements PayService {}

@Service
class WechatPayService implements PayService {}
```

```java
@Autowired
private PayService payService;
```

👉 结果：注入 `AliPayService`

---

## 2. 特点

- 只影响 **单个注入点选择**
    
- 只在 **类型冲突时生效**
    
- 优先级低于 `@Qualifier`
    

---

# 三、@Order（解决“顺序问题”）

## 1. 场景：List 注入

```java
public interface Handler {}
```

```java
@Component
@Order(2)
class BHandler implements Handler {}

@Component
@Order(1)
class AHandler implements Handler {}
```

```java
@Autowired
private List<Handler> handlers;
```

👉 注入顺序：

```text
AHandler → BHandler
```

---

## 2. @Order 的本质

👉 控制 Bean 在集合中的排序

---

## 3. 常见使用场景

### ✔ 责任链模式

```java
@Order(1)
@Order(2)
@Order(3)
```

---

### ✔ Spring AOP

```java
@Order(1)
@Aspect
class LogAspect {}

@Order(2)
@Aspect
class SecurityAspect {}
```

👉 数字越小 → 越先执行

---

# 四、关键区别（重点）

## 1. 决策维度不同

|对比|@Primary|@Order|
|---|---|---|
|解决问题|选哪个 Bean|排序|
|作用对象|单个注入|多个 Bean 集合|
|是否唯一选择|是|否|

---

## 2. 使用方式不同

### @Primary（单选）

```java
@Autowired
PayService service; // 只注入一个
```

---

### @Order（多选）

```java
@Autowired
List<PayService> services; // 全部注入 + 排序
```

---

# 五、优先级关系（非常重要）

## 当 @Primary + @Order 同时存在：

```java
@Primary
@Order(10)
class A {}

@Order(1)
class B {}
```

### 👉 注入结果：

#### 1. 单对象注入：

```java
@Autowired
PayService service;
```

👉 A（因为 Primary）

---

#### 2. List 注入：

```java
@Autowired
List<PayService> list;
```

👉 顺序：

```
B → A   （按 Order）
```

---

# 六、总结（面试必背）

```text
@Primary：解决“默认用谁”
@Order：解决“谁先执行”
```

---

# 七、一句话记忆

👉 Primary 是“选人”  
👉 Order 是“排队”

---

# 八、扩展（很常考）

## Q：@Order 能决定 @Autowired 单 Bean 注入吗？

❌ 不能

👉 它只影响集合顺序，不影响“单 Bean 选择”

---

## Q：@Primary 能控制顺序吗？

❌ 不能

---

# 如果你想深入，我可以继续讲：

- Spring AOP 多切面执行顺序源码
    
- @Order 在 BeanPostProcessor 中的作用
    
- @Primary vs @Qualifier vs byName 完整决策流程图（面试杀手级）
    

只要你说 👍


