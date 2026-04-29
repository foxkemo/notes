在 Linux 命令列處理中，`awk` 側重於「數據提取與計算」，而 `column` 側重於「視覺化排版」。兩者經常搭配使用，讓雜亂的輸出變得易讀。

以下是它們的具體區別與配合用法：

## 1. awk：強大的數據處理器

`awk` 是一個文本處理語言。它會將每一行拆分成多個欄位（預設以空格或制表符分隔），並允許你對這些欄位進行邏輯操作。

- 優勢：能挑選特定列、過濾行、進行數學計算。
- 缺點：直接輸出時，如果各列內容長度不一，對齊會顯得很亂。
- 範例：
    
    ```bash
    # 只提取第 1、3、5 欄位
    awk '{print $1, $3, $5}' file.txt
    ```
    

## 2. column：專業的排版工具

`column` 的唯一任務就是將輸入內容轉換為整齊的表格列。

- 優勢：自動計算每一列的最大寬度並對齊。
- 常用參數：`-t`（建立表格）。
- 範例：
    
    ```bash
    # 將不整齊的空格文本變成對齊的表格
    column -t file.txt
    ```
    

---

## 3. 強強聯手：先提取，後對齊

這是最常見的工作流：用 `awk` 挑出你需要的數據，再用 `column -t` 把它變得美觀。

情境：查看磁碟空間，但只想要「檔案系統」和「掛載點」這兩列

```bash
df -h | awk '{print $1, $6}' | column -t
```

- 沒有 `column`：輸出可能會因路徑長短不一而偏移。
- 加上 `column -t`：輸出會像 Excel 一樣垂直對齊，非常清晰。

## 4. 進階技巧：處理特殊分隔符

如果你的數據是用冒號 `:` 或逗號 `,` 分隔的（例如 `/etc/passwd`）：

- awk 使用 `-F` 指定輸入分隔符
- column 使用 `-s` 指定分隔符

```bash
# 提取用戶名和 Shell，並美化輸出
cat /etc/passwd | awk -F: '{print $1, $7}' | column -t
```

---

小總結：

- 如果你想「挑選或過濾」數據，用 `awk`。
- 如果你想「對齊或美化」顯示，用 `column`。

您目前是有特定的日誌檔案需要處理，還是想了解如何用這些工具進行條件過濾（例如只顯示大小超過 100MB 的行）？
在 Linux 中，如果你想使用 `awt` 命令来查看列（columns），可能是指你想要使用一些命令来格式化输出，尤其是在命令行中通过表格格式显示列数据。通常，`awk` 是用于列操作和数据处理的工具，而 `column` 是用于美化输出和对齐列的命令。

### 1. **`awk` 用法**

`awk` 是一个强大的文本处理工具，可以根据列来进行操作和格式化：

- 打印指定列：
    
    ```bash
    awk '{print $1, $2}' filename
    ```
    
    这将打印文件 `filename` 的第一列和第二列。
    
- 用指定分隔符分割并打印列：
    
    ```bash
    awk -F ':' '{print $1, $3}' /etc/passwd
    ```
    
    这会使用冒号 `:` 作为分隔符，并打印 `/etc/passwd` 文件的第一列和第三列。
    

### 2. **`column` 命令**

`column` 用于将文本格式化为对齐的列。例如，使用管道将输出传递给 `column` 来整齐显示数据。

#### 基本示例：

```bash
echo -e "name age city\nAlice 25 NewYork\nBob 30 LosAngeles" | column -t
```

输出：

```bash
name   age  city
Alice  25   NewYork
Bob    30   LosAngeles
```

#### 以特定字符为分隔符：

```bash
echo -e "name|age|city\nAlice|25|NewYork\nBob|30|LosAngeles" | column -s '|' -t
```

输出：

```bash
name  age  city
Alice 25   NewYork
Bob   30   LosAngeles
```

### 3. **`column` 和 `awk` 结合使用**

你可以将 `awk` 和 `column` 结合使用，首先通过 `awk` 提取你需要的列，然后通过 `column` 来格式化输出。

例如：

```bash
ps aux | awk '{print $1, $3, $11}' | column -t
```

这将显示进程的用户名、CPU 使用率和命令名，格式化成整齐的列。

---

### 4. **查看端口映射与 `column` 结合**

假设你想查看 Docker 容器的端口映射并格式化输出，可以这样做：

```bash
docker ps --format "table {{.ID}}\t{{.Names}}\t{{.Ports}}" | column -t
```

这会将 `docker ps` 输出的容器 ID、名称和端口映射格式化成整齐的表格。

---

### 总结：

- **`awk`** 用于列数据的提取和处理。
    
- **`column`** 用于格式化输出，使列数据对齐，适合用于将命令输出转换为易于阅读的表格格式。
    

你可以根据你的具体需求组合使用这两个命令。

