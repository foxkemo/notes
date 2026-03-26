在 Spring Framework 里，“读取配置文件并注入”是非常核心的能力（你做后端开发基本天天用到）。我给你系统梳理一下：**常见方式 + 使用场景 + 示例**。

---

# ✅ 一、最常用方式总览

| 方式       | 注解/类                              | 适用场景     |
| -------- | --------------------------------- | -------- |
| 单个值注入    | `@Value`                          | 简单配置     |
| 批量绑定     | `@ConfigurationProperties`        | 配置较多、结构化 |
| 环境变量读取   | `Environment`                     | 动态获取     |
| yml 映射对象 | `@ConfigurationProperties` + Bean | 推荐做法     |

---

# ✅ 二、@Value（最简单，最常用）

## 📌 配置文件（application.yml）

```yaml
server:
  port: 8080

user:
  name: admin
```

## 📌 注入方式

```java
@Value("${user.name}")
private String username;
```

---

## 🎯 使用场景

* 少量配置
* 临时读取
* 快速开发

👉 例如：

* 读取端口
* 读取开关（feature flag）
* 读取 token

---

# ✅ 三、@ConfigurationProperties（强烈推荐）

👉 更适合你现在做后端项目

## 📌 配置文件

```yaml
datasource:
  url: jdbc:mysql://localhost:3306/test
  username: root
  password: 123456
```

## 📌 创建配置类

```java
@Component
@ConfigurationProperties(prefix = "datasource")
public class DataSourceConfig {

    private String url;
    private String username;
    private String password;

    // getter / setter
}
```

---

## 🎯 使用场景

👉 **企业开发主流方式**

* 数据库配置
* Redis 配置
* JWT 配置
* 第三方 API 配置

---

## ✅ 优点

✔ 类型安全
✔ 自动映射
✔ 支持复杂结构（List / Map）

---

# ✅ 四、Environment（动态读取）

```java
@Autowired
private Environment env;

public void test() {
    String port = env.getProperty("server.port");
}
```

---

## 🎯 使用场景

* 动态判断配置
* 条件逻辑（if 判断）
* 非 Bean 场景

---

# ✅ 五、读取复杂结构（List / Map）

## 📌 YAML

```yaml
app:
  servers:
    - 192.168.1.1
    - 192.168.1.2
```

## 📌 Java

```java
@ConfigurationProperties(prefix = "app")
@Component
public class AppConfig {
    private List<String> servers;
}
```

---

# ✅ 六、结合 @Bean 使用（高级）

```java
@Configuration
public class Config {

    @Value("${user.name}")
    private String name;

    @Bean
    public User user() {
        return new User(name);
    }
}
```

---

# ✅ 七、典型真实开发场景（重点🔥）

## 1️⃣ 登录鉴权（你刚问的 Apifox 就对应这个）

```yaml
jwt:
  secret: abc123
  expire: 3600
```

👉 注入：

```java
@ConfigurationProperties(prefix = "jwt")
public class JwtConfig {
    private String secret;
    private long expire;
}
```

---

## 2️⃣ 短链接项目（你之前写的）

```yaml
short-url:
  domain: https://xx.com
```

👉 用于生成短链：

```java
String shortUrl = domain + "/" + code;
```

---

## 3️⃣ Redis 缓存

```yaml
spring:
  redis:
    host: localhost
    port: 6379
```

👉 Spring Boot 自动注入

---

## 4️⃣ 开关控制（灰度发布）

```yaml
feature:
  enableCache: true
```

```java
@Value("${feature.enableCache}")
private boolean enableCache;
```

---

## 5️⃣ 多环境配置

* application-dev.yml
* application-prod.yml

👉 Spring 自动切换

---

# 🚀 总结一句话

👉 **推荐优先级：**

```
少量配置 → @Value
结构化配置 → @ConfigurationProperties（主流）
动态读取 → Environment
```

---

# ⚠️ 常见坑（你一定会遇到）

1. ❌ 忘记加 `@Component`
2. ❌ prefix 写错
3. ❌ 没有 getter/setter
4. ❌ yml 层级不对

---

如果你愿意，我可以帮你做一套**完整 Spring Boot 配置体系设计（JWT + Redis + 短链接 + 多环境）**，直接能写进你课程论文里那种级别。
