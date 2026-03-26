Java 基础命令可以按**开发流程**来理解：编写 → 编译 → 运行 → 调试 → 打包。下面帮你系统梳理一套最常用的 👇

---

# 一、编译 & 运行（最核心）

这是你写 Java 程序必须掌握的。

### 1. 编译 `.java` → `.class`

```bash
javac HelloWorld.java
```

👉 生成：`HelloWorld.class`（字节码文件）

---

### 2. 运行程序

```bash
java HelloWorld
```

⚠️ 注意：

* **不要加 `.class` 后缀**
* 必须有 `main` 方法

---

# 二、查看 / 调试类文件

### 1. 反编译 class（查看字节码）

```bash
javap HelloWorld
```

更详细：

```bash
javap -c HelloWorld
```

👉 用于：

* 看 JVM 指令
* 分析底层执行（你做算法/底层优化会用到）

---

# 三、打包（Jar）

### 1. 打成 jar 包

```bash
jar cvf app.jar HelloWorld.class
```

参数解释：

* `c`：create
* `v`：verbose（显示过程）
* `f`：file

---

### 2. 运行 jar

```bash
java -jar app.jar
```

⚠️ 前提：
需要 `MANIFEST.MF` 指定入口类

---

# 四、查看 Java 环境

### 1. 查看版本

```bash
java -version
```

### 2. 查看编译器版本

```bash
javac -version
```

### 3. 查看环境变量

```bash
echo $JAVA_HOME
```

---

# 五、JShell（交互式 Java）

类似 Python REPL，很适合测试代码：

```bash
jshell
```

进入后可以直接写：

```java
System.out.println("Hello");
```

---

# 六、常见 JVM 参数（进阶）

运行时加参数：

```bash
java -Xms256m -Xmx512m HelloWorld
```

👉 含义：

* `-Xms`：初始堆内存
* `-Xmx`：最大堆内存

---

# 七、项目开发常用（你后端开发一定会用）

### 1. 编译指定目录

```bash
javac -d . HelloWorld.java
```

👉 自动生成包目录结构

---

### 2. 指定 classpath

```bash
java -cp . HelloWorld
```

或：

```bash
java -classpath . HelloWorld
```

---

# 八、总结（核心记住这 5 个）

你至少要熟练这几个：

```bash
javac xxx.java   # 编译
java xxx         # 运行
jar cvf xxx.jar  # 打包
javap -c xxx     # 看字节码
jshell           # 测试代码
```

---

# 给你的建议（结合你在学后端）

你现在在学 Spring / Java 后端，这些命令的实际作用是：

* `javac / java` 👉 理解 JVM 执行机制（面试常问）
* `jar` 👉 Spring Boot 本质就是 fat jar
* `classpath` 👉 依赖加载核心（非常重要）
* `javap` 👉 深入理解底层（进阶必备）

---

