在 Linux 和 Docker 的日常操作中，`cat`、`echo` 配合重定向符号（`>` 和 `>>`）是处理文件 IO 的基础工具。理解它们的区别，关键在于理解**数据流的源头**。

### 1. 核心符号：重定向的行为

不论前面是 `cat` 还是 `echo`，重定向符号的规则是统一的：

- **`>` (覆盖输出):** 先将目标文件清空（若不存在则创建），再写入数据。
    
- **`>>` (追加输出):** 不会清空文件，而是将数据挂在文件的末尾（EOF 之后）。
    

---

### 2. `echo`：常用于单行或简单变量写入

`echo` 是将你提供的**字符串**直接推送到标准输出（stdout）。

- **追加单行：**
    
    `echo "export JAVA_HOME=/usr/lib/jvm" >> ~/.bashrc`
    
    _常用于修改配置文件。_
    
- **配合变量：**
    
    `echo $VAR >> log.txt`
    

---

### 3. `cat`：常用于多行写入或合并文件

`cat` 的本意是“连接”（Concatenate），它从一个或多个源头读取内容。

#### 场景 A：合并文件

如果你想把 `file1.txt` 的内容追加到 `file2.txt`：

`cat file1.txt >> file2.txt`

#### 场景 B：多行追加（Here Document）

这是开发者最常用的技巧，特别是在编写 Dockerfile 或自动化脚本时，避免使用一堆 `echo`：

Bash

```
cat <<EOF >> config.yaml
server:
  port: 8080
  mode: production
EOF
```

- `<<EOF` 告诉系统：接下来的内容都是输入，直到遇到 `EOF` 为止。
    
- `>> config.yaml` 负责把这整块内容追加到文件末尾。
    

---

### 4. 进阶：Docker 中的“权限坑”

如果你在宿主机上尝试往一个只有 `root` 权限的 Docker 配置文件里追加内容，可能会遇到权限拒绝：

`sudo echo "test" >> /var/lib/docker/config.json` # **会报错！**

**原因：** `sudo` 只作用于 `echo`，而重定向符号 `>>` 是由当前 Shell 执行的，当前 Shell 没有 root 权限。

**解决方案（使用 `tee`）：**

`echo "test" | sudo tee -a /var/lib/docker/config.json > /dev/null`

- `tee -a` 相当于 `>>`（append）。
    
- 这样整个链路都拥有了必要的权限。
    

---

### 5. 对比总结

|**工具**|**优势场景**|**典型用法**|
|---|---|---|
|**`echo`**|写入简单的字符串、环境变量。|`echo "done" >> task.log`|
|**`cat`**|写入大段配置、合并多个文件。|`cat file1 file2 > bigfile`|
|**`tee`**|需要 `sudo` 权限写入时。|`echo "str"|

你是在为你的 **Java 后端项目** 编写自动化部署的 `entrypoint.sh` 脚本吗？如果是为了动态生成 `application.yml`，我建议使用 `cat <<EOF` 这种方式，它在可读性上比一连串的 `echo` 要好得多。需要我为你演示一个复杂的模板生成示例吗？