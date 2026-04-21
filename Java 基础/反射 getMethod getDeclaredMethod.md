这两个方法是反射里最容易混的点之一，我直接帮你把**差异 + 使用场景 + 常见坑**一次讲清。

---

# 一、核心区别（一句话版）

> 👉 **`getMethod`：找“能调用的 public 方法”（包括继承的）**  
> 👉 **`getDeclaredMethod`：找“这个类自己声明的所有方法”（不管访问权限）**

---

# 二、详细对比（必须记住这个表）

|特性|`getMethod`|`getDeclaredMethod`|
|---|---|---|
|查找范围|当前类 + 父类 + 接口|只查当前类|
|访问修饰符|只支持 `public`|所有（private/protected/default/public）|
|是否查父类|✔️ 会|❌ 不会|
|是否查接口|✔️ 会|❌ 不会|
|能否拿 private|❌ 不行|✔️ 可以|
|常见用途|调用 API 方法|框架 / 反射底层|

---

# 三、举例说明（非常直观）

```java
class Parent {
    public void a() {}
    private void b() {}
}

class Child extends Parent {
    public void c() {}
    private void d() {}
}
```

---

## 1️⃣ `getMethod`

```java
Child.class.getMethod("a"); // ✔️ 父类 public
Child.class.getMethod("c"); // ✔️ 本类 public
Child.class.getMethod("b"); // ❌ private
Child.class.getMethod("d"); // ❌ private
```

---

## 2️⃣ `getDeclaredMethod`

```java
Child.class.getDeclaredMethod("c"); // ✔️
Child.class.getDeclaredMethod("d"); // ✔️ private
Child.class.getDeclaredMethod("a"); // ❌（父类）
```

---

# 四、在 AOP 场景里的区别（你重点要看这个）

你之前写的：

```java
getDeclaredMethod(...)
```

👉 问题：

- ❌ 找不到父类方法
    
- ❌ 找不到接口方法
    
- ❌ 很容易 `NoSuchMethodException`
    

---

## 更稳一点的：

```java
getMethod(...)
```

👉 优点：

- ✔️ 能找父类
    
- ✔️ 能找接口
    

👉 但问题：

- ❌ 拿不到 private
    
- ❌ 不解决 JDK Proxy “接口方法”问题
    

---

# 五、为什么你在 AOP 里都不应该直接用这两个？

因为你真正的问题不是：

> ❌ “我找不到 method”

而是：

> ✔️ “我拿到的是不是**正确的 method（实现类那个）**”

---

# 六、Spring 的正确做法（关键）

```java
MethodSignature signature = (MethodSignature) joinPoint.getSignature();

Method method = signature.getMethod(); // 可能是接口方法

Method targetMethod =
    AopUtils.getMostSpecificMethod(
        method,
        joinPoint.getTarget().getClass()
    );
```

---

👉 这个方法帮你解决：

- 接口 → 实现类 ✔️
    
- 父类 → 子类 ✔️
    
- 泛型 bridge method ✔️
    
- CGLIB / JDK proxy ✔️
    

---

# 七、注解获取再配合

```java
AnnotationUtils.findAnnotation(targetMethod, Xxx.class);
```

👉 自动帮你：

- 查父类 ✔️
    
- 查接口 ✔️
    
- 查组合注解 ✔️
    

---

# 八、使用建议（非常重要）

## ✔️ 普通反射

|场景|用哪个|
|---|---|
|调 public API|`getMethod`|
|操作 private 字段/方法|`getDeclaredMethod`|

---

## ✔️ Spring / AOP（你现在这个场景）

👉 **不要直接用这两个！**

直接用：

```java
AopUtils.getMostSpecificMethod + AnnotationUtils
```

---

# 九、一句话总结

> 👉 **`getMethod` 是“对外接口视角”，`getDeclaredMethod` 是“类内部视角”，但 AOP 需要“真实执行方法视角”**

---

如果你下一步想更深入，我可以帮你把这三层彻底打通：

```text
接口方法（JDK Proxy）
↓
桥接方法（泛型）
↓
真实方法（target class）
```

这是 Spring AOP 最核心的一条链路。