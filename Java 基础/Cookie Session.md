在 Web 开发里，**Cookie 的“设置”和“返回”本质就是：服务器通过响应头下发，浏览器再通过请求头自动带回**。你可以把它理解成一个“自动来回传递的小键值对”。

我按你做后端开发（Spring/Java）的思路，给你讲清楚👇

---

# 一、Cookie 是怎么“设置”的（服务器 → 浏览器）

### ✅ 本质：通过 HTTP 响应头 `Set-Cookie`

```
Set-Cookie: username=kun; Path=/; Max-Age=3600
```

### 📌 Java（Servlet / Spring）写法

```java
Cookie cookie = new Cookie("username", "kun");
cookie.setMaxAge(3600); // 1小时
cookie.setPath("/");

response.addCookie(cookie);
```

---

# 二、Cookie 是怎么“返回”的（浏览器 → 服务器）

浏览器会自动在请求头里带上：

```
Cookie: username=kun
```

👉 注意：**这是浏览器自动做的，你不用手动处理**

---

### 📌 Java 获取 Cookie

```java
Cookie[] cookies = request.getCookies();

if (cookies != null) {
    for (Cookie c : cookies) {
        if ("username".equals(c.getName())) {
            System.out.println(c.getValue());
        }
    }
}
```

---

# 三、完整流程（面试高频）

```
1. 用户访问登录接口
2. 服务器验证成功
3. 服务器返回 Set-Cookie
4. 浏览器保存 Cookie
5. 后续请求自动带上 Cookie
6. 服务器通过 Cookie 识别用户
```

👉 这就是 **Session 登录机制的基础**

---

# 四、常见属性（很重要）

### 1️⃣ 过期时间

```java
cookie.setMaxAge(3600);
```

* 单位：秒
* 不设置 → 浏览器关闭就没了（会话 Cookie）

---

### 2️⃣ Path（作用路径）

```java
cookie.setPath("/");
```

* `/`：全站可用
* `/user`：只有 `/user` 下请求才带

---

### 3️⃣ Domain（跨子域）

```java
cookie.setDomain(".example.com");
```

👉 可以让：

```
a.example.com
b.example.com
```

共享 Cookie

---

### 4️⃣ HttpOnly（防 XSS）

```java
cookie.setHttpOnly(true);
```

👉 JS 无法读取（很重要）

---

### 5️⃣ Secure（HTTPS 才发送）

```java
cookie.setSecure(true);
```

---

# 五、Spring Boot 写法（更实战）

### ✅ 设置 Cookie

```java
@GetMapping("/set")
public String set(HttpServletResponse response) {
    Cookie cookie = new Cookie("token", "123456");
    cookie.setMaxAge(3600);
    cookie.setPath("/");
    response.addCookie(cookie);
    return "ok";
}
```

---

### ✅ 获取 Cookie

```java
@GetMapping("/get")
public String get(HttpServletRequest request) {
    for (Cookie cookie : request.getCookies()) {
        if ("token".equals(cookie.getName())) {
            return cookie.getValue();
        }
    }
    return "no cookie";
}
```

---

# 六、补充：前端也能设置 Cookie

```javascript
document.cookie = "username=kun; path=/; max-age=3600";
```

⚠️ 但：

* 不能设置 HttpOnly
* 安全性较低

---

# 七、Cookie vs Session（你后端必须会）

| 项目   | Cookie | Session               |
| ---- | ------ | --------------------- |
| 存储位置 | 浏览器    | 服务器                   |
| 安全性  | 低      | 高                     |
| 容量   | 小（4KB） | 大                     |
| 自动发送 | ✔      | 通过 Cookie（JSESSIONID） |

👉 实际：

* Cookie 存 SessionID
* Session 存用户数据

---

# 八、一句话总结（面试）

👉 **Cookie 是服务器通过 Set-Cookie 下发，浏览器自动保存并在后续请求中通过 Cookie 请求头携带的数据。**

---

如果你要，我可以帮你再进阶一层👇
👉 **JWT + Cookie + Session + Redis 登录体系（面试杀手级）**
