直接给你结论，然后把几个容易混的点一次讲清。

---

# 一、`getAnnotation` 会不会“往父类找”？

## ✅ 分情况

### 1️⃣ **类上的注解（Class）**

```java
Child.class.getAnnotation(A.class);
```

👉 **可能会向父类找，但前提是：**

```java
@Inherited
@interface A {}
```

✔️ 有 `@Inherited` → 会沿着**继承链向上找父类**  
❌ 没有 → 只看当前类

---

### 2️⃣ **方法上的注解（Method）**

```java
method.getAnnotation(A.class);
```

👉 **绝对不会往父类找 ❌**

无论：

- 父类方法 ✔️
    
- 接口方法 ✔️
    
- override ✔️
    

👉 都不会自动继承

---

### 3️⃣ **字段 / 接口 / 参数**

👉 一律：

```text
❌ 不会继承
```

---

# 二、为什么方法不继承（本质）

Java 设计就是：

> 👉 **方法注解 = 绑定在“这个 Method 对象”上**

而不是“逻辑方法”

---

举个例子：

```java
class Parent {
    @A
    public void test() {}
}

class Child extends Parent {
    @Override
    public void test() {}
}
```

```java
Child.class.getMethod("test").getAnnotation(A.class);
```

👉 结果：

```text
null ❌
```

---

# 三、`getMethod` vs `getDeclaredMethod`（非常重要）

## ✅ 1. `getMethod`

```java
clazz.getMethod(name, params);
```

特点：

|行为|说明|
|---|---|
|✔️ 查 public 方法|包括继承的|
|✔️ 会往父类找|是|
|✔️ 会找接口方法|是|
|❌ 查不到 private|不行|

---

## ✅ 2. `getDeclaredMethod`

```java
clazz.getDeclaredMethod(name, params);
```

特点：

|行为|说明|
|---|---|
|✔️ 查本类所有方法|包括 private|
|❌ 不查父类|不会|
|❌ 不查接口|不会|

---

# 四、对比总结（记住这个表）

|方法|查当前类|查父类|查接口|private|
|---|---|---|---|---|
|getMethod|❌（只 public）|✔️|✔️|❌|
|getDeclaredMethod|✔️|❌|❌|✔️|

---

# 五、结合你 AOP 场景（关键）

你现在的问题其实是：

> 👉 “我要找的是哪个 Method？”

---

## ❌ 错误思路

```java
getDeclaredMethod → getAnnotation
```

👉 问题：

- 找不到父类 ❌
    
- 找不到接口 ❌
    
- 找不到 override 链 ❌
    

---

## ✅ 正确思路（Spring 做的）

```java
接口方法
   ↓
AopUtils.getMostSpecificMethod
   ↓
真实实现方法
   ↓
AnnotationUtils.findAnnotation
   ↓
沿继承链查找 ✔️
```

---

# 六、Spring 为什么“能找到父类方法注解”？

因为它不是用：

```java
getAnnotation ❌
```

而是：

```java
AnnotationUtils.findAnnotation ✔️
```

---

## 👉 它内部逻辑是：

```text
1. 当前方法找
2. 找不到 → 父类方法
3. 找不到 → 接口方法
4. 找不到 → 类级别
```

👉 相当于“手动实现继承”

---

# 七、一句话总结（帮你记住）

> 👉 **Java 反射不会帮你继承方法注解，Spring 会**

---

# 八、你现在应该这样写（最终版）

```java
MethodSignature signature = (MethodSignature) joinPoint.getSignature();

Method method = signature.getMethod();

Method targetMethod = AopUtils.getMostSpecificMethod(
        method,
        joinPoint.getTarget().getClass()
);

NoDuplicateSubmit annotation =
        AnnotationUtils.findAnnotation(targetMethod, NoDuplicateSubmit.class);
```

---

# 九、核心记忆口诀

```text
getAnnotation
    ↓
只看当前

getMethod
    ↓
看父类（但只 public）

getDeclaredMethod
    ↓
只看自己

Spring AnnotationUtils
    ↓
全都帮你找
```

---

如果你下一步想彻底打通这块，我可以帮你把：

👉 **Spring 注解解析底层（MergedAnnotations 机制）**

讲到源码级，这一块你现在这个阶段学非常合适。