这题很好，已经进入**Java 后端 + 安全体系 + Web基础类**的核心了。我给你分三层讲：

---

# 一、`Cipher` 类详解（重点🔥）

## 👉 所在包

```java
javax.crypto.Cipher
```

---

## 👉 作用

```text
统一处理：加密 / 解密（对称 + 非对称）
```

👉 支持：

* AES（对称）
* RSA（非对称）

---

## 1️⃣ 获取实例（最关键）

```java
Cipher cipher = Cipher.getInstance("算法/模式/填充");
```

### 常见写法：

```java
"AES/CBC/PKCS5Padding"
"RSA/ECB/PKCS1Padding"
```

---

## 2️⃣ 初始化

```java
cipher.init(Cipher.ENCRYPT_MODE, key);
cipher.init(Cipher.DECRYPT_MODE, key);
```

---

## 3️⃣ 执行

```java
byte[] result = cipher.doFinal(data);
```

---

## 4️⃣ AES 示例（Spring 常用🔥）

```java
Cipher cipher = Cipher.getInstance("AES/CBC/PKCS5Padding");

SecretKeySpec key = new SecretKeySpec(keyBytes, "AES");
IvParameterSpec iv = new IvParameterSpec(ivBytes);

cipher.init(Cipher.ENCRYPT_MODE, key, iv);

byte[] encrypted = cipher.doFinal(data);
```

---

## 5️⃣ RSA 示例

```java
Cipher cipher = Cipher.getInstance("RSA/ECB/PKCS1Padding");

cipher.init(Cipher.ENCRYPT_MODE, publicKey);

byte[] encrypted = cipher.doFinal(data);
```

---

## ❗Cipher 常见坑

* ❌ 不能只写 `"AES"`（必须带模式）
* ❌ ECB 不安全
* ❌ RSA 不能加密大数据
* ✅ 必须配合 Base64 传输

---

# 二、Java 安全核心类（必须掌握🔥）

---

## 1️⃣ 哈希类

## 👉 `MessageDigest`

```java
java.security.MessageDigest
```

### 用途：

* MD5 / SHA

```java
MessageDigest md = MessageDigest.getInstance("SHA-256");
byte[] hash = md.digest(data);
```

---

## 2️⃣ 签名类

## 👉 `Signature`

```java
java.security.Signature
```

### 用途：

* RSA 签名（JWT、认证）

```java
Signature sign = Signature.getInstance("SHA256withRSA");
```

---

## 3️⃣ 密钥相关

### 👉 `KeyPairGenerator`

```java
java.security.KeyPairGenerator
```

生成公钥私钥

---

### 👉 `KeyFactory`

👉 用字符串恢复 key

---

### 👉 `SecretKeySpec`

👉 对称加密 key

---

### 👉 `IvParameterSpec`

👉 初始化向量（AES 必备）

---

# 三、编码类（常用🔥）

---

## 👉 Base64

```java
java.util.Base64
```

```java
Base64.getEncoder().encodeToString(data);
Base64.getDecoder().decode(str);
```

---

# 四、Spring Web 核心类（你必须熟🔥）

---

# 1️⃣ 请求相关

## 👉 `HttpServletRequest`

```java
javax.servlet.http.HttpServletRequest
```

👉 获取：

* 参数
* Header
* Token

---

## 👉 `ServletRequest`

👉 父接口

---

# 2️⃣ 响应相关

## 👉 `HttpServletResponse`

```java
javax.servlet.http.HttpServletResponse
```

👉 设置：

* Header
* Cookie
* 状态码

---

## 👉 `ServletResponse`

👉 父接口

---

# 3️⃣ Cookie

```java
javax.servlet.http.Cookie
```

---

# 4️⃣ Session

```java
HttpSession
```

---

# 五、SpringBoot 常用工具类（开发高频🔥）

---

## 1️⃣ JSON 处理

### 👉 Jackson

```java
com.fasterxml.jackson.databind.ObjectMapper
```

---

## 2️⃣ String 工具

### 👉 Spring

```java
org.springframework.util.StringUtils
```

---

## 3️⃣ Bean 工具

```java
org.springframework.beans.BeanUtils
```

👉 对象拷贝

---

## 4️⃣ 集合工具

```java
org.springframework.util.CollectionUtils
```

---

# 六、Spring 安全相关（重点🔥）

---

## 👉 密码加密

```java
org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder
```

```java
encoder.encode(password);
encoder.matches(raw, encoded);
```

---

## 👉 JWT（常用库）

* jjwt
* nimbus-jose-jwt

---

# 七、你要形成的体系（非常关键🔥）

```text
Java安全体系：

加密：
  Cipher（AES / RSA）

哈希：
  MessageDigest（SHA）

签名：
  Signature（RSA签名）

编码：
  Base64

Web：
  HttpServletRequest / Response

Spring：
  BCrypt / ObjectMapper / BeanUtils
```

---

# 八、面试标准回答（直接用🔥）

```text
在 Java 中：

1. Cipher 用于加密解密，支持 AES 和 RSA
2. MessageDigest 用于哈希，如 SHA-256
3. Signature 用于数字签名，如 RSA 签名
4. Base64 用于编码传输

在 Web 开发中：

1. HttpServletRequest / Response 处理请求响应
2. Spring 提供 BeanUtils、StringUtils 等工具类
3. 安全方面常用 BCrypt 进行密码加密

整体构成了 Java 后端的安全与 Web 基础体系
```

---

# 九、进阶建议（结合你当前学习🔥）

你现在在学 Spring + 后端，我建议你下一步：

👉 做一个完整项目：

```text
登录：
  bcrypt 存密码

认证：
  JWT（RSA签名）

接口：
  SpringBoot + 拦截器

安全：
  AES + RSA（可选）
```

---

如果你愿意，我可以帮你做一套👇

👉 **“Java 后端常用类速查表（面试版）”**
👉 或者 **SpringBoot 安全模块完整代码模板（登录 + JWT + 拦截器）**

这套就是中级后端的核心能力。
