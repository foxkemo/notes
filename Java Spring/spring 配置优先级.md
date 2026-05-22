在 Spring / Spring Boot 中，spring.profiles.active **可以指定多个 Profile**，多个值之间用逗号（,）分隔。

例如：spring.profiles.active=dev,mysql,redis。此时，Spring 会同时激活这三个环境的配置。如果有配置冲突，**后面（右侧）的会覆盖前面（左侧）的**。

关于你提到的通过虚拟机参数（-D）**和**普通参数（命令行参数）来指定，它们的语法、底层原理以及优先级有很大不同。

## 1. 虚拟机参数（-D）指定

这是 Java VM 的系统属性（System Properties）。它必须紧跟在 java 命令之后，在 -jar 之前。

Bash

```
java -Dspring.profiles.active=dev,mysql -jar app.jar
```

- **原理：** Spring 启动时会通过 System.getProperty("spring.profiles.active") 读取。
    
- **适用场景：** 常用于 Tomcat 等外部容器部署，或者在控制台通过环境变量统一管理 JVM 行为。
    

## 2. 普通参数（命令行参数）指定

这是 Spring Boot 提供的非标准程序参数（Program Arguments）。它必须放在 -jar app.jar **之后**。

Bash

```
java -jar app.jar --spring.profiles.active=prod,redis
```

- **语法：** 必须使用双横杠 -- 开头。
    
- **原理：** 传入 main(String[] args) 方法，Spring Boot 的 CommandLineArgsPropertySource 会解析并加载它。
    

## 3. 两者的优先级（谁说了算？）

如果在一行命令中同时使用了这两种方式，**普通参数（命令行参数）会覆盖虚拟机参数（-D）**。

### 优先级排序（从高到低）：

**1.最高优先级：普通参数（命令行参数）：**--spring.profiles.active=...。

放在 -jar 后面的参数。即使前面设置了 -D，也会被这里完全覆盖。

**2.次高优先级：虚拟机参数（JVM 系统属性）：**-Dspring.profiles.active=...。

放在 java 和 -jar 之间的系统变量。

**3.第三优先级：操作系统环境变量：**export SPRING_PROFILES_ACTIVE=...。

操作系统级的环境变量。

**4.最低优先级：配置文件：**application.properties / yml。

代码里写死的 spring.profiles.active 默认值。

## 4. 混合使用的特殊行为

> **注意：** 这里的覆盖是**整体替换**，而不是合并。

如果你执行了以下命令：

Bash

```
java -Dspring.profiles.active=mysql -jar app.jar --spring.profiles.active=dev
```

最终**只会激活 dev**，而 mysql 会被彻底忽略，并不会合并成 dev,mysql。

### 避坑小贴士 💡

- 在写启动脚本（如 Dockerfile 或 Shell 脚本）时，建议**统一使用一种方式**。由于 -- 命令行参数优先级最高且最直观，目前在微服务、容器化部署中更为常用。
    
- 多个 Profile 激活时，后定义的会覆盖先定义的。例如 --spring.profiles.active=base,dev，如果 application-base.yml 和 application-dev.yml 里有同名配置项，以 dev 的为准。