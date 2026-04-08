结论先说清楚（很关键🔥）：

```text
大部分运行参数是在 create 阶段确定的
run = create + start
```

---

# 一、`docker run` 到底做了什么？

```bash
docker run -it -d nginx
```

👉 实际等价于：

```bash
docker create -it nginx
docker start container_id
```

---

# 二、那 `-i -t -d` 是在哪一步生效？

## 1️⃣ `-i` 和 `-t` —— ✔ 在 create 阶段

👉 这两个属于：

```text
容器“配置属性”（container config）
```

具体来说：

- 是否打开 STDIN
    
- 是否分配 TTY
    

👉 在 `create` 时就写进容器配置里了

---

### 举例

```bash
docker create -it ubuntu
```

👉 此时：

- 容器还没启动 ❗
    
- 但已经“具备交互能力”
    

---

## 2️⃣ `-d` —— ❗ 在 start / run 阶段

👉 `-d` 不属于容器本身配置

它只是：

```text
是否让当前命令“后台返回”
```

👉 也就是说：

- 不影响容器本身
    
- 只影响 CLI 行为
    

---

# 三、用一句话总结三者生命周期🔥

```text
-i / -t：属于容器配置（create 时确定）
-d     ：属于运行方式（start/run 时决定）
```

---

# 四、为什么要这样设计？（底层原因🔥）

---

## 1️⃣ `-i / -t` 必须在 create 阶段

因为：

```text
需要提前创建：
- TTY 设备
- stdin pipe
```

👉 这些是“资源分配”，必须在容器启动前完成

---

## 2️⃣ `-d` 不能在 create 阶段

因为：

```text
-d 只是 CLI 是否阻塞
```

👉 容器根本不知道你是前台还是后台运行

---

# 五、验证一下（你可以自己试🔥）

---

## 创建一个带 TTY 的容器

```bash
docker create -it ubuntu
```

---

## 启动

```bash
docker start container_id
```

👉 此时你会发现：

- 容器是有 tty 的
    
- 但你没 attach
    

---

## attach 进去

```bash
docker attach container_id
```

👉 能正常交互 ✔

---

## 如果没有 `-t`

```bash
docker create -i ubuntu
```

再 attach：

👉 体验很差（没有终端能力）

---

# 六、一个非常重要的理解（面试级🔥）

```text
create 阶段：定义容器“是什么”
start 阶段 ：决定容器“怎么跑”
```

---

# 七、再帮你梳理一张总表（强烈记住🔥）

|参数|阶段|本质|
|---|---|---|
|-i|create|打开 STDIN|
|-t|create|分配 TTY|
|-d|run/start|后台运行|

---

# 八、延伸一点（你这个阶段建议掌握🔥）

实际上 Docker 内部是：

```text
docker CLI
   ↓
containerd
   ↓
runc
   ↓
Linux namespace + cgroup + tty + pipe
```

👉 `-i -t` 会影响：

- `/dev/pts/*`
    
- pipe 管道
    
- 文件描述符（fd 0/1/2）
    

---

# 九、你可以这样记（最简单版本🔥）

```text
-i / -t = 创建容器时“焊死”的能力
-d      = 运行时“怎么看它”
```

---

如果你想再往下深入一层，我可以帮你讲👇

👉 **docker exec 为什么还能再开一个 tty（不是 create 的那个）**  
👉 **容器里 /dev/pts 是怎么来的（伪终端机制）**  
👉 **Java 程序为什么能直接输出到 docker logs（stdout 重定向）**

这些会把你从“会用 Docker”提升到“理解 Docker”。