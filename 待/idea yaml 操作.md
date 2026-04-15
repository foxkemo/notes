这个问题很实用，尤其你现在在搞 **ShardingSphere + YAML 配置**，IDEA 的这些技巧能救命 👍

我分两块讲你要的👇

---

# 一、IDEA 中 YAML 调整缩进

## ✅ 1. 快捷键调整缩进

### 👉 向右缩进（增加层级）

```
Tab
```

### 👉 向左缩进（减少层级）

```
Shift + Tab
```

👉 适用于选中多行：

```yaml
rules:
- !SHARDING
tables:
```

选中后按 `Tab` → 自动变：

```yaml
rules:
  - !SHARDING
    tables:
```

---

## ✅ 2. 自动格式化（强烈推荐）

### 👉 快捷键：

- Mac：`⌘ + Option + L`
    
- Windows：`Ctrl + Alt + L`
    

👉 会自动帮你修正：

- 缩进
    
- 对齐
    
- 空格
    

⚠️ 前提：语法不能太离谱，否则不会完全修好

---

## ✅ 3. 结构折叠（看层级神器）

点击左边的小三角：

```yaml
dataSources:
  ds_0:  ▼
```

👉 可以快速看结构是否正确：

```text
dataSources
  ├─ ds_0
  ├─ ds_1
rules
  └─ SHARDING
```

---

## ✅ 4. 显示缩进辅助线（建议打开）

路径：

```
Settings → Editor → General → Appearance
```

✔ 勾选：

```
Show indent guides
```

👉 效果：一眼看出层级错没错

---

# 二、复制 YAML 键路径（你这个非常关键🔥）

👉 比如你想复制：

```yaml
rules:
  - tables:
      t_coupon_template:
```

👉 得到路径：

```text
rules[0].tables.t_coupon_template
```

---

## ✅ 方法一（最推荐）

### 👉 快捷键：

- Mac：`⌘ + Shift + A`
    
- Windows：`Ctrl + Shift + A`
    

输入：

```
Copy Reference
```

👉 会复制：

```text
rules[0].tables.t_coupon_template
```

---

## ✅ 方法二（右键）

右键 YAML key：

```
Copy Reference
```

---

## ✅ 方法三（JSON Path 形式）

有些插件支持：

```
Copy JSON Path
```

👉 结果类似：

```text
$.rules[0].tables.t_coupon_template
```

---

# 三、进阶技巧（强烈建议你掌握）

## ✅ 1. YAML 转 JSON（排错神器）

👉 IDEA 右键：

```
Convert YAML to JSON
```

👉 可以发现：

- List / Map 错误
    
- 层级问题
    

---

## ✅ 2. 对比结构（避免你刚才那个坑）

你刚才的问题本质是：

```yaml
dataSources:
  ds_0: ...
  rules: ❌
```

👉 用结构视图一眼看出：

```text
dataSources
  ├─ ds_0
  ├─ ds_1
  ├─ rules   ❌（错误）
```

---

## ✅ 3. 使用 Schema 校验（高级）

如果是 Spring YAML：

```yaml
spring:
  datasource:
```

👉 IDEA 会自动提示

但 ShardingSphere YAML：

👉 建议加官方 schema（可选）

---

# 四、你现在最应该养成的习惯（重点）

以后写 YAML：

---

## ✔ 第一步：写完立刻格式化

```
Ctrl + Alt + L
```

---

## ✔ 第二步：折叠结构看层级

👉 确认 `rules` 不在 `dataSources` 里

---

## ✔ 第三步：复制路径验证

👉 看是不是：

```text
dataSources.ds_0
rules[0].tables
```

---

# 🚀 总结一句话

👉 YAML 不难，难的是：

```text
缩进 = 语义
```

---

# 🎯 如果你想更进一步（非常建议）

我可以帮你把：

👉 **ShardingSphere YAML → Java 配置 → SQL 路由流程**

整个链路讲清楚（面试直接秒人）

或者你可以继续问：

- 👉 YAML vs properties 在 Spring 中的区别
    
- 👉 ShardingSphere 配置加载流程
    
- 👉 IDEA YAML 插件推荐
    

你现在这个阶段，已经在往**中高级后端工程师**走了 👍