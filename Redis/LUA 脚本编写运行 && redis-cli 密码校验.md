在 Redis 中使用 Lua 脚本可以保证操作的**原子性**（Atomic），减少网络开销，并允许你在服务端执行复杂的逻辑。

以下是关于 Lua 脚本编写、运行以及通过 `redis-cli` 进行密码校验的详细指南。

---

## 1. 编写 Lua 脚本

Redis 中的 Lua 脚本主要通过两个全局变量与数据库交互：

- `redis.call()`：执行命令，遇到错误时直接抛出异常。
    
- `redis.pcall()`：执行命令，遇到错误时捕获异常并返回错误表格。
    

## 脚本示例 (`script.lua`)

假设我们要实现一个简单的“限流”逻辑：如果 Key 不存在则设为 1 并设置过期时间；如果存在则自增。

Lua

```
-- 获取参数
local key = KEYS[1]
local expire_time = ARGV[1]

-- 执行逻辑
local current = redis.call('GET', key)

if current == false then
    redis.call('SET', key, 1)
    redis.call('EXPIRE', key, expire_time)
    return 1
else
    return redis.call('INCR', key)
end
```

---

## 2. 运行 Lua 脚本

运行脚本主要有两种方式：直接执行脚本文件或使用 `EVAL` 命令。

## A. 使用 `redis-cli` 执行脚本文件（推荐）

这是最常用的方式，尤其是脚本较长时。

Bash

```
redis-cli --eval /path/to/script.lua key1 key2 , arg1 arg2
```

> **注意：** 这里的 `,` 两边必须有空格。逗号前面是 `KEYS` 参数，后面是 `ARGV` 参数。

## B. 在交互模式下使用 `EVAL`

代码段

```
EVAL "return redis.call('SET', KEYS[1], ARGV[1])" 1 mykey myvalue
```

- `1` 表示后面紧跟的 Key 的数量。
    

---

## 3. redis-cli 密码校验

如果你的 Redis 开启了密码保护（`requirepass`），可以通过以下几种方式校验：

## 方式一：连接时通过 `-a` 参数（最直接）

Bash

```
redis-cli -h 127.0.0.1 -p 6379 -a "你的密码"
```

_警告：在终端明文输入密码可能会记录在 history 中，不建议在生产环境自动化脚本中使用。_

## 方式二：连接后使用 `AUTH` 命令

Bash

```
$ redis-cli
127.0.0.1:6379> AUTH "你的密码"
OK
```

## 方式三：结合 Lua 脚本执行

如果你要运行脚本且需要密码：

Bash

```
redis-cli -a "你的密码" --eval script.lua mykey , 10
```

---

## 4. Lua 脚本在 Redis 中的工作流

## 核心特性

1. **原子性**：Redis 保证脚本在执行期间不会被其他命令插入。
    
2. **缓存**：可以使用 `SCRIPT LOAD` 将脚本存入 Redis，之后通过 `EVALSHA` 调用，减少传输负担。
    
3. **脚本限制**：脚本中严禁使用非确定性函数（如随机数、当前时间），除非你是通过参数传进去的，否则可能导致主从同步不一致。
    

---

**您是想针对特定的业务逻辑（如分布式锁或排行榜）编写脚本吗？我可以为您提供更具体的代码模板。**