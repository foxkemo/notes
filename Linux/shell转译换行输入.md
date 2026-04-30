在 Shell 中，處理換行符號時，引號的類型決定了它是被當作「指令」還是「純文字」。

## 1. 單引號 `'\n'` vs 雙引號 `"\n"`

- 單引號 `'\n'`：  
    這是「所見即所得」。Shell 會把它當成兩個字符：一個反斜槓 `\` 和一個字母 `n`。它不會產生換行效果。
- 雙引號 `"\n"`：  
    在大多數 Shell（如 Bash）的 `echo` 命令中，它依然會被當作純文字。  
    但是，如果你配合 `echo -e` 或使用特定的語法（如 `$'\n'`），它就會變成真正的換行。

正確產生換行的方法：

```bash
# 1. 使用 echo -e (常用)
echo -e "第一行\n第二行"

# 2. 使用 ANSI-C 引用 (最可靠)
line="第一行"$'\n'"第二行"
echo "$line"
```

---

## 2. 美觀情況下的「換行輸入」（代碼折行）

當你的命令太長（例如 `docker run` 有十幾個參數），為了美觀，我們使用 反斜槓 `\` 來折行。

- 規則： `\` 必須是該行的最後一個字符（後面不能有空格）。
- 美觀示例：

```bash
docker run -d \
  --name rmqnamesrv \
  -p 9876:9876 \
  -v /opt/logs:/root/logs \
  apache/rocketmq:4.9.4 \
  sh mqnamesrv
```

_這讓命令像列表一樣整齊，易於閱讀和修改。_

---

## 3. 多行字串的優雅寫法 (Here Document)

如果你要將一大段文字（包含換行）寫入文件，使用 `cat <<EOF` 是最美觀的：

```bash
cat <<EOF > config.txt
server.port=8080
namesrvAddr=127.0.0.1:9876
# 這裡直接換行，不需要寫 \n
debug=true
EOF
```

---

## 4. 總結與建議

1. 純顯示換行：用 `echo -e "line1\nline2"`。
2. 代碼太長要折行：用 `\`（末尾不能帶空格）。
3. 變量賦值多行：
    
    - `VAR="第一行`
    - `第二行"` （直接敲回車，雙引號會保留內部的換行）。
    

特別提醒： 在 Dockerfile 的 `RUN` 指令中，強烈建議使用 `\` 將多個命令連起來（配合 `&&`），例如：

```dockerfile
RUN apt-get update && \
    apt-get install -y vim && \
    rm -rf /var/lib/apt/lists/*
```

這既美觀，又能減少鏡像層數。

你是在編寫 NameServer 的啟動腳本，還是要在 Dockerfile 裡排版長命令？