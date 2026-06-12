
在 Spring Web 开发中，**过滤器 (Filter)** 和 **拦截器 (Interceptor)** 是处理请求拦截、预处理和后处理的两种核心机制。

虽然它们的功能相似，但在**层级、生命周期、以及可控性**上有很大区别。

---

### 1. 过滤器 (Filter)

过滤器属于 **Servlet 容器**（如 Tomcat）的范畴，不属于 Spring MVC 框架本身。它在 `DispatcherServlet` 处理请求之前或之后执行。

*   **作用范围**：基于 URL 模式拦截。
*   **常见用途**：设置字符编码、跨域配置 (CORS)、敏感词过滤、全局日志记录、压缩响应。

#### 实现步骤：
1. 实现 `jakarta.servlet.Filter` 接口。
2. 使用 `@Component` 或 `FilterRegistrationBean` 注册。

```java
@Component
public class MyFilter implements Filter {
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) 
            throws IOException, ServletException {
        System.out.println("Filter: 请求进入");
        // 继续执行后续代码（其它过滤器或 Servlet）
        chain.doFilter(request, response);
        System.out.println("Filter: 响应返回");
    }
}
```

---

### 2. 拦截器 (Interceptor)

拦截器是 **Spring MVC** 框架的一部分。它在 `DispatcherServlet` 调用控制器（Controller）的前后执行。

*   **作用范围**：针对 Spring 的 Handler（控制器方法）。
*   **优点**：可以访问 Spring 上下文、Bean 注入，甚至能获取到即将执行的 Controller 方法信息。
*   **常见用途**：权限检查（权限校验）、登录验证、读取请求头、性能监控（统计方法耗时）。

#### 实现步骤：
1. 实现 `HandlerInterceptor` 接口。
2. 在配置类中注册并指定拦截路径。

**实现类：**
```java
public class MyInterceptor implements HandlerInterceptor {
    // 在控制器执行前调用
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("Interceptor: preHandle (进入控制器前)");
        return true; // 返回 true 则继续，返回 false 则中断请求
    }

    // 在控制器执行后，视图渲染前调用
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) {
        System.out.println("Interceptor: postHandle (控制器执行完)");
    }

    // 在整个请求结束后（视图渲染完）调用，常用于清理资源
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {
        System.out.println("Interceptor: afterCompletion (请求完全结束)");
    }
}
```

**配置类：**
```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new MyInterceptor())
                .addPathPatterns("/api/**")  // 拦截哪些路径
                .excludePathPatterns("/api/login"); // 排除哪些路径
    }
}
```

---

### 3. 过滤器 VS 拦截器：核心区别

| 特性 | 过滤器 (Filter) | 拦截器 (Interceptor) |
| :--- | :--- | :--- |
| **所属规范** | Servlet 容器 (如 Tomcat) | Spring MVC 框架 |
| **层级** | 在 DispatcherServlet 外部 | 在 DispatcherServlet 内部 |
| **依赖性** | 不依赖 Spring 容器 | 依赖 Spring 容器 |
| **拦截目标** | 几乎所有请求 (甚至包括静态资源) | 仅针对 Spring 处理的 Controller 请求 |
| **参数访问** | 只能访问 Request/Response | 能访问 Request/Response，还能访问 **Handler (Controller 方法)** |
| **异常处理** | 无法直接利用 Spring 的 `@ExceptionHandler` | 可以配合 Spring 的异常处理机制 |

---

### 4. 执行顺序图

请求流向如下：

1.  **Request**
2.  **Filter 1**
3.  **Filter 2**
4.  **DispatcherServlet**
5.  **Interceptor (preHandle)**
6.  **Controller (Method)**
7.  **Interceptor (postHandle)**
8.  **View Rendering (视图渲染)**
9.  **Interceptor (afterCompletion)**
10. **Filter (返回方向)**
11. **Response**

---

### 5. 什么时候用哪一个？

*   **用 Filter：**
    *   当需要修改 `HttpServletRequest` 的内容（如包装 Request 以实现可重复读取输入流）。
    *   通用的、与业务逻辑无关的操作（如：设置 Header、编码、跨域、Web 防火墙）。
    *   在请求进入 Spring 框架之前就需要拦截的情况。

*   **用 Interceptor：**
    *   需要根据具体的业务逻辑（Controller 上的注解、方法名）来判断权限。
    *   需要注入 Spring 里的 Bean 来进行数据库查询。
    *   需要对 Controller 返回的 `ModelAndView` 或数据进行处理。

**总结建议**：在 Spring Boot 开发中，除非是底层的 Servlet 级操作，**优先选择 Interceptor**，因为它更贴近 Spring 的生态，使用起来更灵活。