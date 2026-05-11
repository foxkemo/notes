
# 一、Java 启动结构

Java 启动：

```bash
java [JVM参数] 类名 [程序参数]
```

例如：

```bash
java -Xmx1g -Denv=prod Main aaa bbb
```

含义：

| 部分           | 作用              |
| ------------ | --------------- |
| `-Xmx1g`     | JVM 最大堆         |
| `-Denv=prod` | System Property |
| `Main`       | 主类              |
| `aaa bbb`    | main(args)      |

---

# 二、JVM 参数分类

Java 参数：

主要分：

```text
1. 标准参数
2. -X 参数
3. -XX 参数
4. -D 参数
```

---

# 三、标准 JVM 参数

标准参数：

```text
Java 官方规范定义
```

所有 JVM 基本都支持。

---

## 常见标准参数

|参数|作用|
|---|---|
|`-cp`|classpath|
|`-classpath`|类路径|
|`-jar`|运行 jar|
|`-version`|查看版本|
|`-showversion`|显示版本|
|`-ea`|开启 assert|
|`-da`|禁用 assert|

---

## 示例

### 1. classpath

```bash
java -cp lib/* Main
```

告诉 JVM：

```text
去哪里找 class
```

---

### 2. 运行 jar

```bash
java -jar app.jar
```

---

### 3. assert

```bash
java -ea Main
```

开启：

```java
assert x > 0;
```

---

# 四、-D 参数（System Properties）

这是：

```text
最重要的一类
```

格式：

```bash
-Dkey=value
```

例如：

```bash
java -Dname=kun Main
```

---

# 五、什么是 System Properties

Java 启动时：

JVM 内部维护：

```java
java.util.Properties
```

对象：

```java
System.getProperties()
```

本质：

```text
全局 key-value 配置
```

---

# 六、读取 System Properties

例如：

```java
System.getProperty("name");
```

启动：

```bash
java -Dname=kun Main
```

输出：

```text
kun
```

---

# 七、JVM 默认已有很多 System Properties

例如：

```java
System.getProperties().forEach(
    (k,v)-> System.out.println(k+"="+v)
);
```

会看到：

|key|示例|
|---|---|
|`java.version`|21|
|`os.name`|Mac OS X|
|`user.home`|/Users/xxx|
|`file.encoding`|UTF-8|

---

# 八、System Properties 的作用

很多框架：

通过：

```text
-D
```

读取配置。

例如：

---

## Spring

```bash
-Dspring.profiles.active=prod
```

---

## Maven

```bash
mvn -DskipTests package
```

---

## Logback

```bash
-Dlog.path=/data/logs
```

---

# 九、-X 参数

`-X`：

表示：

```text
非标准扩展 JVM 参数
```

不同 JVM：

可能不完全一样。

但 HotSpot 基本支持。

---

# 十、最常见 -X 参数

---

## 1. 堆大小

### 初始堆

```bash
-Xms512m
```

### 最大堆

```bash
-Xmx2g
```

---

## 2. 栈大小

```bash
-Xss1m
```

每线程：

```text
1MB 栈
```

---

## 3. 输出 JVM 参数

```bash
-XshowSettings
```

---

## 4. 禁止 JIT

```bash
-Xint
```

纯解释执行。

---

## 5. 强制 JIT

```bash
-Xcomp
```

尽量全部编译。

---

# 十一、-XX 参数

这是：

```text
HotSpot 高级内部参数
```

最强大。

---

# 十二、-XX 参数格式

三种：

|格式|示例|
|---|---|
|布尔开关|`-XX:+UseG1GC`|
|禁用开关|`-XX:-UseG1GC`|
|数值参数|`-XX:MaxGCPauseMillis=200`|

---

# 十三、经典 -XX 参数

---

## GC 选择

### G1

```bash
-XX:+UseG1GC
```

---

### ZGC

```bash
-XX:+UseZGC
```

---

### Shenandoah

```bash
-XX:+UseShenandoahGC
```

---

# 十四、GC 调优

---

## 最大停顿时间

```bash
-XX:MaxGCPauseMillis=200
```

---

## 新生代比例

```bash
-XX:NewRatio=2
```

---

## Survivor 比例

```bash
-XX:SurvivorRatio=8
```

---

# 十五、OOM 导出

---

## Dump 文件

```bash
-XX:+HeapDumpOnOutOfMemoryError
```

---

## Dump 路径

```bash
-XX:HeapDumpPath=/dump
```

---

# 十六、GC 日志

JDK8：

```bash
-XX:+PrintGCDetails
```

JDK9+：

```bash
-Xlog:gc*
```

---

# 十七、查看所有 JVM 参数

---

## 查看标准参数

```bash
java --help
```

---

## 查看 -X

```bash
java -X
```

---

## 查看 -XX

```bash
java -XX:+PrintFlagsFinal
```

非常多：

```text
几百上千项
```

---

# 十八、JVM 参数影响哪些区域

|参数|影响|
|---|---|
|`-Xmx`|Java Heap|
|`-Xss`|Thread Stack|
|`-XX:MetaspaceSize`|Metaspace|
|`-XX:MaxDirectMemorySize`|DirectBuffer|
|`-XX:+UseG1GC`|GC 算法|

---

# 十九、一个完整例子

例如：

```bash
java \
-Xms2g \
-Xmx2g \
-Xss1m \
-XX:+UseG1GC \
-XX:MaxGCPauseMillis=200 \
-XX:+HeapDumpOnOutOfMemoryError \
-Dspring.profiles.active=prod \
-jar app.jar \
--server.port=8080
```

---

## 含义

|参数|含义|
|---|---|
|`-Xms2g`|初始堆|
|`-Xmx2g`|最大堆|
|`-Xss1m`|线程栈|
|`UseG1GC`|G1 GC|
|`MaxGCPauseMillis`|最大停顿|
|`HeapDumpOnOOM`|OOM dump|
|`-Dspring...`|Spring profile|
|`--server.port`|Spring Boot 参数|

---

# 二十、最核心理解

一句话：

```text
-D 是 Java 全局配置(System Properties)
-X 是 JVM 扩展参数
-XX 是 HotSpot 高级底层调优参数
```

而：

```text
--xxx
```

通常：

```text
是应用程序自己的命令行参数
```

例如 Spring Boot。




在 Spring Boot（[Spring Boot](https://spring.io/projects/spring-boot?utm_source=chatgpt.com)）里，配置来源很多，但它们会被统一放进 **Environment**，然后按优先级覆盖。

你问的核心是：

```text
YAML 配置 vs 系统参数 vs 自定义参数 的优先级
```

本质就是：**谁覆盖谁**

---

# 一、Spring Boot 配置来源（从低到高）

Spring Boot 的标准优先级大致如下：

```text
1. application.yml / application.properties（最低）
2. @PropertySource / 自定义 properties 文件
3. 环境变量（OS）
4. JVM System Properties（-D）
5. 命令行参数（--xxx）（最高）
```

---

# 二、重点三类对比

你关心的三种：

## 1️⃣ YAML 配置（最低优先级）

```yaml
server:
  port: 8080
```

来源：

```text
application.yml / properties
```

特点：

- 默认配置
    
- 最常见
    
- 优先级最低
    

---

## 2️⃣ 系统参数（-D）

```bash
java -Dserver.port=9090 -jar app.jar
```

来源：

```text
JVM System Properties
```

读取方式：

```java
System.getProperty("server.port")
```

Spring 会自动加入 Environment。

---

## 3️⃣ 命令行参数（--）

```bash
java -jar app.jar --server.port=10010
```

来源：

```text
Spring Boot CommandLineArgs
```

---

# 三、最终优先级（核心结论）

### Spring Boot 实际优先级：

```text
命令行参数（--xxx）   ← 最高
JVM 参数（-Dxxx）
系统环境变量
application.yml
默认值
```

---

# 四、覆盖示例（非常重要）

假设：

## application.yml

```yaml
server:
  port: 8080
```

---

## JVM 参数

```bash
-Dserver.port=9090
```

---

## 命令行参数

```bash
--server.port=10010
```

---

## 最终结果

```text
10010 生效
```

---

# 五、为什么这样设计？

Spring Boot 设计原则：

```text
越“临时”的配置，优先级越高
```

---

|类型|特性|
|---|---|
|YAML|固定配置（代码级）|
|-D|启动级配置|
|--|运行时覆盖|

---

# 六、配置系统本质（很关键）

Spring Boot 内部统一成：

```text
Environment
```

里面有多个：

```text
PropertySource（链式）
```

加载顺序类似：

```text
CommandLinePropertySource
SystemPropertySource
SystemEnvironmentPropertySource
application.yml
```

---

# 七、验证方法

你可以打印：

```java
@Autowired
private Environment env;

System.out.println(env.getProperty("server.port"));
```

---

# 八、一个容易混淆点

## ❗ YAML 不会覆盖 -D

例如：

```bash
java -Dserver.port=9090 -jar app.jar
```

即使 YAML 写：

```yaml
server:
  port: 8080
```

最终：

```text
9090 生效
```

---

# 九、总结（最核心一句）

```text
Spring Boot 配置优先级：-- 命令行 > -D 系统属性 > YAML 文件
```

---

# 十、如果你要再深入

我可以帮你把这一整套讲透：

- Spring Environment 初始化流程
    
- PropertySource 加载源码
    
- 为什么 CommandLineArguments 优先级最高
    
- 配置绑定 @ConfigurationProperties 底层机制
    

这些是 Spring Boot 配置系统的“本质逻辑”。