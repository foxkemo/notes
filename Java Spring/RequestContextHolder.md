理解 Spring Web 与原生 Java Web（Servlet API）的关系，就像是理解“**精装修房**”与“**毛坯房/原材料**”的关系。Spring Web 并不是取代了 Java Web，而是建立在 Java Web 之上的一种高度封装。

以下是两者的深度对比与联系：

---

### 1. 核心组件对比

| 功能维度 | 原生 Java Web (Servlet API) | Spring Web (Spring MVC) |
| :--- | :--- | :--- |
| **入口点** | `HttpServlet` (每个 Servlet 处理特定 URL) | `DispatcherServlet` (前端控制器，统一入口) |
| **配置方式** | `web.xml` 或 `@WebServlet` 注解 | `@Controller` / `@RequestMapping` 注解 |
| **请求参数** | 手动调用 `request.getParameter()` 并转换类型 | **自动数据绑定**：直接映射到方法参数或 POJO |
| **响应处理** | 手动 `response.getWriter().write()` | **自动序列化**：返回对象即自动转 JSON (通过 HttpMessageConverter) |
| **生命周期管理** | 由 Servlet 容器（Tomcat/Jetty）管理 | 由 Spring IoC 容器管理（作为 Bean） |
| **视图跳转** | `request.getRequestDispatcher(...).forward()` | 返回 String (视图名) 或 `ModelAndView` |

---

### 2. 关系图解：Spring 是如何“套壳”的

Spring Web 的核心是 **`DispatcherServlet`**。从本质上讲，`DispatcherServlet` 就是一个标准的 `HttpServlet`。

1.  **原生层**：Tomcat 接收到 HTTP 请求，解析成原生的 `HttpServletRequest`。
2.  **Spring 入口**：Tomcat 根据配置，将请求转交给 Spring 的 `DispatcherServlet`。
3.  **Spring 内部转发**：`DispatcherServlet` 并不直接写业务，而是查询 **HandlerMapping**，找到对应的 `@Controller` 方法。
4.  **业务层**：执行你写的代码（也就是你在项目里写的那些 Controller）。

---

### 3. 关键类映射关系

当你使用 Spring Web 时，你其实是在间接或直接地操作原生 Java Web 类：

#### (1) 请求与响应
*   **原生**：`javax.servlet.http.HttpServletRequest`
*   **Spring 封装**：`ServletWebRequest` 或通过 `RequestContextHolder` 获取。
*   **联系**：Spring 的 `ServletRequestAttributes` 内部就持有一个原生的 `HttpServletRequest` 引用。

#### (2) 过滤器与拦截器
*   **原生：Filter**
    *   属于 Servlet 规范。
    *   在 `DispatcherServlet` 执行**之前**触发。
    *   无法直接注入 Spring 的 Bean（除非通过特殊的代理）。
*   **Spring：Interceptor (拦截器)**
    *   属于 Spring 框架。
    *   在 `DispatcherServlet` 内部，进入 Controller **前后**触发。
    *   可以完美使用 Spring 的依赖注入（如 `@Autowired`）。

#### (3) 上下文
*   **原生：ServletContext**
    *   整个 Web 应用的全局上下文。
*   **Spring：WebApplicationContext**
    *   Spring 的 Bean 容器，它持有一个 `ServletContext` 的引用。

---

### 4. 为什么要用 Spring Web 而不是原生 Servlet？

举个简单的例子：**获取用户注册表单数据**。

*   **原生 Servlet 做法：**
    ```java
    public void doPost(HttpServletRequest request, HttpServletResponse response) {
        String name = request.getParameter("name");
        int age = Integer.parseInt(request.getParameter("age")); // 还要手动处理异常
        User user = new User(name, age);
        // 手动写回 JSON...
    }
    ```

*   **Spring Web 做法：**
    ```java
    @PostMapping("/register")
    public Result register(User user) { // Spring 自动填充属性
        return Result.ok(user); // Spring 自动转 JSON
    }
    ```
    Spring 帮你处理了：类型转换、异常捕获、参数校验（Validation）、内容协商（JSON/XML 切换）。

---

### 5. 你的代码中是如何体现的？

回到你之前的 `NoDuplicateSubmitAspect` 代码：

```java
ServletRequestAttributes sra = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
String path = sra.getRequest().getServletPath();
```

这里展现了完美的协作：
1.  **`RequestContextHolder`**：这是 Spring 的类。
2.  **`ServletRequestAttributes`**：这是 Spring 提供的适配器。
3.  **`.getRequest()`**：它返回的是原生的 `HttpServletRequest`（Java Web 标准）。
4.  **`.getServletPath()`**：这是原生 Servlet API 定义的方法。

### 总结
**Java Web 是地基，Spring Web 是地基之上的摩天大楼。** 
在 Spring Boot 时代，虽然我们几乎不再手写 Servlet，但 Spring 依然在底层默默地将每一个 `@RequestMapping` 翻译成对原生 Servlet API 的调用。理解了原生 Java Web，你才能明白 Spring 为什么要设计 `RequestContextHolder` 这些工具类。