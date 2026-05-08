Java 中“文件与路径”相关的类主要分为两套体系：

- 早期 IO（`java.io`）
    
- NIO.2（`java.nio.file`，Java 7 引入，更推荐）
    

---

# 一、传统 IO：`java.io`

## 1. File 类（最核心）

```java
import java.io.File;
```

用于表示：

- 文件
    
- 文件夹（目录）
    
- 路径
    

但它本身不负责真正读写文件内容。

---

## 常见创建

```java
File file = new File("a.txt");

File dir = new File("test");

File abs = new File("/Users/test/a.txt");
```

---

## 常用方法

## 判断

```java
file.exists();      // 是否存在
file.isFile();      // 是否是文件
file.isDirectory(); // 是否是目录
```

---

## 创建

```java
file.createNewFile(); // 创建文件

dir.mkdir();          // 创建单层目录

dir.mkdirs();         // 创建多层目录
```

---

## 删除

```java
file.delete();
```

---

## 获取信息

```java
file.getName();       // 文件名

file.getPath();       // 路径

file.getAbsolutePath(); // 绝对路径

file.length();        // 文件大小（字节）
```

---

## 遍历目录

```java
File dir = new File(".");

File[] files = dir.listFiles();

for (File f : files) {
    System.out.println(f.getName());
}
```

---

# 二、NIO.2（推荐）

Java 7 后更现代的文件 API。

包：

```java
java.nio.file
```

核心：

- `Path`
    
- `Paths`
    
- `Files`
    

---

# 1. Path 接口

表示路径。

比 `File` 更现代。

---

## 创建 Path

```java
import java.nio.file.Path;
import java.nio.file.Paths;

Path path = Paths.get("a.txt");
```

或者：

```java
Path path = Path.of("a.txt");
```

（Java 11 后常用）

---

## 常用方法

```java
path.getFileName();

path.getParent();

path.toAbsolutePath();

path.normalize();
```

---

# 2. Files 类（重点）

真正进行文件操作。

```java
import java.nio.file.Files;
```

这是一个工具类。

---

# 文件判断

```java
Files.exists(path);

Files.isDirectory(path);

Files.isRegularFile(path);
```

---

# 创建文件/目录

```java
Files.createFile(path);

Files.createDirectory(path);

Files.createDirectories(path);
```

---

# 删除

```java
Files.delete(path);

Files.deleteIfExists(path);
```

---

# 读取文件

## 一次性读取全部

```java
String s = Files.readString(path);
```

---

## 读取所有行

```java
List<String> lines = Files.readAllLines(path);
```

---

# 写入文件

```java
Files.writeString(path, "hello");
```

---

# 复制文件

```java
Files.copy(source, target);
```

---

# 移动文件

```java
Files.move(source, target);
```

---

# 遍历目录

```java
Files.list(path)
```

例如：

```java
Files.list(Path.of("."))
     .forEach(System.out::println);
```

---

# 三、路径相关概念

# 1. 相对路径

```java
"a.txt"
```

相对于：

- 当前工作目录
    
- 通常是项目根目录
    

---

# 2. 绝对路径

Mac/Linux：

```java
/Users/test/a.txt
```

Windows：

```java
C:\\test\\a.txt
```

---

# 3. 路径分隔符

Java 自动适配系统：

```java
File.separator
```

Windows：

```text
\
```

Linux/macOS：

```text
/
```

---

# 四、字符流与字节流（与文件常一起出现）

---

# 字节流

处理：

- 图片
    
- 视频
    
- 二进制
    

类：

```java
FileInputStream
FileOutputStream
```

---

# 字符流

处理文本。

类：

```java
FileReader
FileWriter
BufferedReader
BufferedWriter
```

---

# 五、现代推荐写法

现在开发中通常：

- 路径：`Path`
    
- 文件操作：`Files`
    
- 文本读取：`Files.readString`
    
- 大文件：`BufferedReader`
    

较少直接用：

```java
new File(...)
```

---

# 六、经典对比

|功能|旧 IO|新 NIO|
|---|---|---|
|表示路径|File|Path|
|文件操作|File 方法|Files 工具类|
|遍历目录|listFiles|Files.list|
|异常处理|较弱|更完整|
|API 风格|面向对象|工具函数式|

---

# 七、示例：读取一个文本文件

```java
import java.nio.file.Files;
import java.nio.file.Path;

public class Main {
    public static void main(String[] args) throws Exception {

        Path path = Path.of("test.txt");

        String text = Files.readString(path);

        System.out.println(text);
    }
}
```

---

# 八、常见面试/学习重点

重点建议掌握：

## 基础

- File
    
- Path
    
- Files
    

---

## 文件读写

- FileInputStream
    
- BufferedReader
    
- Files.readString
    

---

## 目录操作

- 创建目录
    
- 遍历目录
    
- 递归删除
    

---

## 路径

- 相对路径
    
- 绝对路径
    
- normalize
    
- resolve
    

例如：

```java
Path p = Path.of("a").resolve("b");
```

结果：

```text
a/b
```

---

# 九、一个非常重要的区别

## File

是：

```text
旧 IO
```

设计较老。

---

## Path + Files

是：

```text
现代 Java 文件 API
```

Spring Boot、Maven、现代框架里更常见。