好，这里把 **Pointcut（切点）怎么用**讲清楚，直接围绕 Spring AOP 实战来。

---

# 一、Pointcut 是干嘛的？

👉 Pointcut

一句话：

> **用来“选方法”的规则**

---

# 二、最常见用法（必须会）

## 1️⃣ 定义切点

```java
@Pointcut("execution(* com.example.service.*.*(..))")
public void serviceMethods() {}
```

👉 含义：

```text
com.example.service 包下
所有类
所有方法
任意参数
```

---

## 2️⃣ 使用切点

```java
@Before("serviceMethods()")
public void before() {
    System.out.println("前置增强");
}
```

---

# 三、execution 表达式（核心）

格式：

```text
execution(返回值 包名.类名.方法名(参数))
```

---

## 常见写法

### 1️⃣ 匹配所有方法

```java
execution(* com.xxx.service.*.*(..))
```

---

### 2️⃣ 匹配某个方法

```java
execution(* com.xxx.service.UserService.save(..))
```

---

### 3️⃣ 匹配某个类

```java
execution(* com.xxx.service.UserService.*(..))
```

---

### 4️⃣ 匹配参数类型

```java
execution(* com.xxx.service.*.*(String))
```

---

# 四、通配符说明（必须掌握）

|符号|含义|
|---|---|
|`*`|任意|
|`..`|任意层级 / 任意参数|
|`()`|方法参数|

---

## 举例

```text
*        → 任意返回值
..       → 任意参数 / 子包
```

---

# 五、完整示例（推荐你背）

```java
@Aspect
@Component
public class LogAspect {

    @Pointcut("execution(* com.example.service.*.*(..))")
    public void pointcut(){}

    @Around("pointcut()")
    public Object around(ProceedingJoinPoint pjp) throws Throwable {
        System.out.println("前");

        Object result = pjp.proceed();

        System.out.println("后");
        return result;
    }
}
```

---

# 六、进阶用法（常用但很多人不会）

## 1️⃣ 按注解切

```java
@Pointcut("@annotation(com.xxx.Log)")
```

👉 拦截带 `@Log` 注解的方法

---

## 2️⃣ 按类注解切

```java
@Pointcut("@within(org.springframework.stereotype.Service)")
```

---

## 3️⃣ 组合表达式

```java
@Pointcut("execution(* com.xxx.service.*.*(..)) && @annotation(com.xxx.Log)")
```

---

# 七、和 JoinPoint 的关系

👉 JoinPoint

```text
JoinPoint = 所有方法
Pointcut  = 筛选方法
```

---

# 八、常见坑（很重要）

## ❌ 写错包路径 → 不生效

```text
com.xxx.service
```

---

## ❌ 方法必须是 public

👉 Spring AOP 默认：

- 只代理 public 方法
    

---

## ❌ 自调用失效

```java
this.method()
```

👉 不走代理 → AOP 不生效

---

# 九、面试总结（直接用）

> Pointcut 用于定义 AOP 的拦截规则，通常通过 execution 表达式指定匹配的方法范围，再结合 Advice 在这些方法执行前后织入增强逻辑。

---

# 十、给你一个进阶建议（你现在很适合）

你下一步可以重点搞：

👉 execution 表达式高级写法（参数、泛型匹配）  
👉 @annotation + 反射获取注解参数  
👉 自定义注解 + AOP（企业级常用）

---

如果你想，我可以带你写一个：

👉 **“防重复提交 + AOP + Redis” 实战切面（你之前问过那个场景）**

这个是直接能用到项目里的。