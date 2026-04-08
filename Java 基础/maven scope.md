好的，我们来详细讲解 **Maven 依赖的 `scope`（范围）机制**，尤其是 `compile`、`provided`、`runtime`，以及它们的底层工作原理和区别。

---

## 1️⃣ Maven 依赖范围（Scope）概念

Maven 的依赖 `scope` 决定了三个核心方面：

1. **编译时可用性**（classpath 是否可用）
    
2. **测试时可用性**（测试类是否可用）
    
3. **打包时是否包含在最终产物**（jar/war）
    

常用 scope 有：

|Scope|编译|运行|测试|打包|
|---|---|---|---|---|
|**compile**|✅|✅|✅|✅|
|**provided**|✅|❌|✅|❌|
|**runtime**|❌|✅|✅|✅|
|**test**|❌|❌|✅|❌|
|**system**|✅|✅|✅|✅|
|**import**|—|—|—|—|

> 默认 scope 是 `compile`。

---

## 2️⃣ 常用 scope 说明

### 2.1 `compile`

- **默认值**
    
- **编译、运行、测试时都可用**
    
- **会打包进最终 jar/war**
    
- 典型场景：业务代码依赖库
    

```xml
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>2.0.9</version>
    <scope>compile</scope> <!-- 可省略，默认就是 compile -->
</dependency>
```

### 2.2 `provided`

- **编译时可用**（IDE 编译正常）
    
- **运行时不包含在最终包**
    
- 典型场景：容器已经提供的依赖，例如 Servlet API
    

```xml
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>4.0.1</version>
    <scope>provided</scope>
</dependency>
```

- **底层机制**：
    
    1. Maven 会在 **编译 classpath** 加载 `provided` 依赖
        
    2. 打包阶段会 **排除该依赖**（不放入 jar/war）
        
    3. 运行时需要 **外部容器提供**（如 Tomcat, Jetty）
        

### 2.3 `runtime`

- **编译时不需要**
    
- **运行时需要**
    
- **会打包进最终产物**
    
- 典型场景：JDBC 驱动、日志实现
    

```xml
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.4.11</version>
    <scope>runtime</scope>
</dependency>
```

- **底层机制**：
    
    1. Maven **不把 runtime 依赖加入编译 classpath**
        
    2. **运行时 classpath** 会包含 runtime 依赖
        
    3. 打包到最终 jar/war
        

---

## 3️⃣ classpath 不同阶段表现

假设项目有如下依赖：

```text
slf4j-api (compile)
logback-classic (runtime)
servlet-api (provided)
```

|阶段|classpath 包含|
|---|---|
|编译|slf4j-api + servlet-api|
|测试|slf4j-api + servlet-api + logback-classic（test 默认包含 compile+runtime）|
|运行|slf4j-api + logback-classic + 容器提供 servlet-api|
|打包|slf4j-api + logback-classic（不包含 provided）|

---

## 4️⃣ 底层机制解析

### 4.1 Maven 编译时 classpath

- Maven 生成编译 classpath：
    
    ```
    compile + provided + (test scope if test compile)
    ```
    
- `provided` 依赖会出现在编译 classpath，但 Maven **不会打包**。
    

### 4.2 Maven 打包

- 打包插件（如 `maven-jar-plugin`、`maven-war-plugin`）会根据 scope **排除 provided**
    
- 运行时容器必须提供 provided 依赖
    

### 4.3 运行时 classpath

- Maven 自身不会管理运行时 classpath，**由容器或启动脚本决定**
    
- `runtime` scope 表示：不编译时需要，但运行时必须可用
    

---

## 5️⃣ 总结原则

|Scope|编译|打包|典型场景|
|---|---|---|---|
|compile|✅|✅|常规库、业务代码依赖|
|provided|✅|❌|容器提供（Servlet API, JDK 扩展）|
|runtime|❌|✅|运行时才需要（JDBC Driver, Logging Implementation）|

---

💡 **实践经验**：

- Spring Boot 项目：
    
    - `spring-boot-starter-*` → compile（默认）
        
    - Servlet API → provided（由内嵌 Tomcat 提供）
        
    - JDBC 驱动 → runtime
        
- **原则**：
    
    1. 编译必须用 → compile / provided
        
    2. 运行必须用 → compile / runtime
        
    3. 不打包 → provided
        

---

如果你愿意，我可以画一张 **compile / provided / runtime 在编译、测试、运行、打包阶段的 classpath 流程图**，直观理解它们底层机制。

你希望我画吗？