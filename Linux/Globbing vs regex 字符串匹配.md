下面帮你把 **globbing（Shell通配符） vs regex（正则表达式）** 在 Bash 中的**语法 + 用法 + 对比**系统整理成一份“速查表级别总结”👇

---

# 🧠 一、核心区别一句话

|类型|作用|
|---|---|
|Globbing|匹配文件名（shell层）|
|Regex|匹配文本内容（工具层）|

---

# ⚙️ 二、Globbing（Shell 通配符语法）

👉 用于：`ls / rm / cp / mv`

## ✔ 基本语法

|符号|含义|
|---|---|
|`*`|任意多个字符|
|`?`|任意一个字符|
|`[abc]`|a 或 b 或 c|
|`[a-z]`|范围|
|`[!abc]`|不匹配 abc|

---

## ✔ 用法示例

### 1️⃣ 任意文件

```bash
ls *.txt
```

👉 匹配：

```
a.txt, b.txt
```

---

### 2️⃣ 单字符匹配

```bash
ls file?.log
```

👉 匹配：

```
file1.log
fileA.log
```

---

### 3️⃣ 字符集合

```bash
ls file[12].txt
```

👉 匹配：

```
file1.txt
file2.txt
```

---

### 4️⃣ 排除

```bash
ls file[!1].txt
```

👉 排除 file1.txt

---

# 🔍 三、Regex（正则表达式语法）

👉 用于：grep / sed / awk

---

## ✔ 基础语法

|符号|含义|
|---|---|
|`.`|任意字符|
|`*`|前一个字符 0+ 次|
|`+`|1+ 次（扩展）|
|`?`|0 或 1 次|
|`^`|行开头|
|`$`|行结尾|
|`[]`|字符集合|
|`|`|

---

## ✔ 用法示例

### 1️⃣ 任意内容

```bash
grep "a.*b" file.txt
```

👉 匹配：

```
axxxb
ab
a123b
```

---

### 2️⃣ 行开头

```bash
grep "^ERROR" log.txt
```

---

### 3️⃣ 行结尾

```bash
grep "done$" log.txt
```

---

### 4️⃣ 或关系

```bash
grep "error|timeout" log.txt
```

（需 egrep / grep -E）

---

# ⚠️ 四、核心区别（重点🔥）

## ❌ 1. 匹配对象不同

|类型|对象|
|---|---|
|glob|文件名|
|regex|文本内容|

---

## ❌ 2. `*` 含义不同

|类型|`*` 含义|
|---|---|
|glob|任意字符|
|regex|前一个字符重复|

---

## ❌ 3. `.txt` 含义不同

|类型|写法|
|---|---|
|glob|`*.txt`|
|regex|`.*\.txt`|

---

## ❌ 4. 执行阶段不同

|阶段|glob|regex|
|---|---|---|
|发生时机|命令执行前|命令执行中|

---

# 🧩 五、对照例子（非常重要🔥）

## ✔ 找 txt 文件

### glob（shell）

```bash
ls *.txt
```

### regex

```bash
grep ".*\.txt"
```

---

## ✔ 找 error 开头行

### regex

```bash
grep "^error"
```

👉 glob做不到

---

# 🧠 六、工具适用场景总结

## 🟢 glob（文件操作）

```bash
ls *.log
rm file?.txt
cp *.jpg /tmp
```

---

## 🔵 regex（文本分析）

```bash
grep "error.*timeout"
sed 's/foo/bar/g'
awk '{print $1}'
```

---

# ⚙️ 七、扩展正则（grep -E / egrep）

```bash
grep -E "error|timeout" log.txt
```

👉 支持：

- `|`
    
- `+`
    
- `?`
    

---

# 🚀 八、一句话总结

👉 **globbing 用于文件名匹配（简单规则），regex 用于文本匹配（复杂表达式），两者语法体系完全不同**

---

# 🔥 如果你想继续深入（推荐）

我可以帮你讲：

- 🔥 grep / sed / awk 三剑客完整体系
    
- 🔥 正则表达式 10 大必会模式
    
- 🔥 Linux 日志排障实战
    
- 🔥 find + regex 高级组合技巧
    

这些是后端工程师核心能力 🚀