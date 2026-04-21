在 Spring AOP 里，`Signature` 是整个 AOP 反射体系的“统一入口”，它的设计本质是：**用一个抽象接口描述“连接点（JoinPoint）对应的程序结构信息”**。

我帮你把 **Signature 体系完整拆开讲清楚（这是 AOP 源码理解关键）**👇

---

# 一、Signature 是什么？

```text
Signature = 连接点的“结构描述接口”
```

在 AOP 中：

👉 每个拦截点（JoinPoint）都对应一个“程序结构”

比如：

- 方法调用
    
- 构造器调用
    
- 字段访问（扩展）
    

---

# 二、Signature 的体系结构（核心）

```text
                Signature（顶层接口）
                     │
     ┌───────────────┼────────────────┐
     │               │                │
MethodSignature ConstructorSignature FieldSignature
```

---

# 三、各子类型作用

## 1️⃣ MethodSignature（最常用）

👉 对应 **方法执行 JoinPoint**

```java
MethodSignature signature = (MethodSignature) joinPoint.getSignature();
```

包含：

- Method
    
- 方法名
    
- 参数类型
    
- 返回值类型
    
- 参数名
    

👉 你现在用的就是这个

---

## 2️⃣ ConstructorSignature（构造器）

👉 对应 `new` 操作

```java
ConstructorSignature signature
```

可以获取：

- 构造器参数
    
- 构造器对象
    

👉 Spring 很少用，但 AOP 支持

---

## 3️⃣ FieldSignature（字段访问）

👉 对应字段读写（理论支持）

```java
FieldSignature signature
```

可以获取：

- Field 对象
    
- 字段类型
    
- 字段名
    

---

# 四、Signature 的核心设计思想

## ✔ 1. 统一模型

```text
不管你拦截什么：
方法 / 构造器 / 字段
→ 都叫 Signature
```

---

## ✔ 2. 分类型扩展

```text
Signature（统一接口）
   ↓
不同 JoinPoint → 不同子接口
```

---

## ✔ 3. 避免强绑定 Method

如果没有 Signature：

```text
AOP → 只能写死 Method
```

👉 Spring 就无法扩展

---

# 五、JoinPoint 和 Signature 的关系

```text
JoinPoint（连接点）
    ↓
getSignature()
    ↓
Signature（结构信息）
```

👉 JoinPoint 关注：

- 当前执行点
    

👉 Signature 关注：

- 这个执行点“是什么结构”
    

---

# 六、为什么要设计成接口体系？

这是 Spring AOP 很关键的设计思想：

## 1️⃣ 解耦执行与结构

```text
JoinPoint = 执行上下文
Signature = 静态结构信息
```

---

## 2️⃣ 支持扩展

未来可以加：

- MethodSignature
    
- ConstructorSignature
    
- FieldSignature
    

不用改 JoinPoint

---

## 3️⃣ 多代理模型统一

无论：

- JDK Proxy
    
- CGLIB
    

👉 都返回 Signature

---

# 七、你现在必须记住的核心结构

```text
JoinPoint
   ↓
Signature（统一入口）
   ↓
MethodSignature（最常用）
   ↓
Method（反射核心）
```

---

# 八、为什么你要强转 MethodSignature？

因为：

```text
Signature = 抽象信息（不够用）
MethodSignature = 方法级增强信息（够用）
```

---

# 九、实际开发中 99% 用的是 MethodSignature

原因：

- Spring AOP 主场景 = 方法拦截
    
- 构造器/字段几乎不用
    

---

# 十、一句话总结（重点）

👉 **Signature 是 AOP 的统一“结构描述体系”，MethodSignature 是方法场景的具体实现**

---

# 十一、帮你建立一个完整认知链（非常重要）

```text
Spring AOP 执行链：

JoinPoint（执行点）
   ↓
Signature（结构描述）
   ↓
MethodSignature（方法级）
   ↓
Method（反射执行）
   ↓
invoke()
```

---

# 十二、如果你要继续深入（建议）

你可以继续问我：

- Signature 为什么不是 Method 直接返回
    
- JoinPoint + ProceedingJoinPoint 区别
    
- Spring AOP 类型体系源码
    
- MethodSignatureImpl 内部结构
    
- AOP 执行链（ReflectiveMethodInvocation）
    

---

如果你想，我可以帮你画一张：

👉 **“Spring AOP JoinPoint + Signature + Method 调用全流程图（源码级）”**

这个图看懂，AOP 就彻底通了。
你这句注释其实已经踩到 Spring AOP 最核心的点了：**你拿到的 method 可能不是“真实实现类的方法”**。

我们把问题拆开讲清楚：**JoinPoint 到底是什么**，以及为什么会出现你说的“接口方法”。

---

# 一、JoinPoint 是什么？

在 Spring AOP 里：

👉 **JoinPoint = 程序执行过程中的“一个切入点”**

简单说就是：

```text
程序运行到某个方法时，被 AOP 拦截的那个点
```

---

## 举个最直观例子

```java
userService.test("Tom");
```

👉 当这个方法被 AOP 拦截时：

```text
JoinPoint = test("Tom") 这个执行瞬间
```

---

# 二、JoinPoint 里面有什么？

```text
JoinPoint
 ├── getTarget()   → 真实对象
 ├── getArgs()     → 参数
 ├── getSignature()→ 方法签名
 └── getThis()     → 代理对象
```

---

# 三、你真正踩坑的点（重点）

你写的：

```java
MethodSignature methodSignature = (MethodSignature) joinPoint.getSignature();
return methodSignature.getMethod().getAnnotation(NoDuplicateSubmit.class);
```

---

## ⚠️ 问题来了：

👉 `getMethod()` **不一定是实现类的方法**

---

# 四、为什么会这样？

取决于 **代理方式**

---

## 1️⃣ JDK 动态代理（你最容易遇到）

```text
代理对象 = 接口代理
```

👉 结果：

```text
methodSignature.getMethod()
→ 接口中的方法
```

---

## 2️⃣ CGLIB 代理

```text
代理对象 = 子类
```

👉 结果：

```text
methodSignature.getMethod()
→ 实现类方法
```

---

# 五、所以你那句注释是什么意思？

```java
/*
错误：可能拿到的是被代理类的接口的方法
*/
```

👉 翻译成人话：

```text
你拿到的 Method 可能不是实现类上的方法
所以注解可能读不到
```

---

# 六、JoinPoint 和 Method 的关系（关键）

```text
JoinPoint = 当前执行点（运行时信息）
   ↓
getSignature()
   ↓
MethodSignature（方法描述）
   ↓
getMethod()
   ↓
可能是：
   - 接口方法（JDK Proxy）
   - 实现类方法（CGLIB）
```

---

# 七、为什么 Spring 设计成这样？

因为 Spring AOP 必须兼容：

- JDK Proxy（接口代理）
    
- CGLIB（类代理）
    

所以：

👉 **JoinPoint 不绑定具体 Method 类型**

---

# 八、正确做法（你现在应该用的）

## ✔ 标准安全写法

```java
MethodSignature signature = (MethodSignature) joinPoint.getSignature();

// 1. 先拿接口方法
Method method = signature.getMethod();

// 2. 再去真实类找方法（关键）
Method targetMethod = joinPoint.getTarget()
        .getClass()
        .getMethod(method.getName(), method.getParameterTypes());

// 3. 从真实方法拿注解
return targetMethod.getAnnotation(NoDuplicateSubmit.class);
```

---

# 九、JoinPoint 一句话本质

👉 **JoinPoint = “程序执行到某个方法时的现场信息快照”**

---

# 十、你现在这个问题的本质

不是 JoinPoint 复杂，而是：

```text
代理对象 vs 真实对象 方法不一致
```

---

# 十一、帮你总结（非常重要）

👉 你这段注释的正确理解是：

```text
JoinPoint.getSignature().getMethod()
不一定是“真实业务方法”
所以不能直接拿注解
```

---

# 十二、如果你继续深入（建议）

你可以继续问我：

- JDK Proxy vs CGLIB 如何决定 JoinPoint.getMethod()
    
- Spring 如何统一 Method 解析（源码）
    
- ReflectiveMethodInvocation 是怎么调用的
    
- 为什么 MethodSignature 不直接返回 targetMethod
    

---

如果你想，我可以帮你画一张：

👉 **“JoinPoint → MethodSignature → Proxy → Target Method 完整调用链图”**

这个图一看你就彻底通 Spring AOP 了。