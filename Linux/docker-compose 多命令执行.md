在 `docker-compose.yml` 中，如果需要在一个容器启动时执行多条命令，最常用的方法是使用 Shell 拼接符号（如 `&&` 或 `;`）。

由于 YAML 的 `command` 字段只能接收一个字符串或一个列表，你需要通过调用 `/bin/sh -c` 或 `/bin/bash -c` 来解析这些指令。

## 常用写法

## 1. 字符串写法（推荐，最直观）

使用 `&&` 确保前一个命令成功后才执行下一个：

```yaml
services:
  app:
    image: node:18
    command: >
      sh -c "npm install && 
             npm run build && 
             npm start"
```

_注：`>` 是 YAML 的折行符号，可以将多行文本转为单行。_

## 2. 列表写法

```yaml
services:
  app:
    image: python:3.9
    command:
      - /bin/bash
      - -c
      - |
        pip install -r requirements.txt
        python migrate.py
        python app.py
```

_注：`|` 是 YAML 的保留换行符，适合书写较复杂的脚本。_

---

## 进阶建议：使用 Entrypoint 脚本

如果命令非常多或包含复杂的逻辑（如判断数据库是否就绪），建议将命令写在单独的 `.sh` 文件中：

1. 编写脚本 (`init.sh`):
    
    ```bash
    #!/bin/bash
    echo "正在初始化..."
    # 执行你的多条命令
    python manage.py db upgrade
    exec python main.py # 使用 exec 让主进程替换 shell 进程，方便接收停止信号
    ```
    
2. 在 Dockerfile 或 Compose 中挂载并执行:
    
    ```yaml
    services:
      app:
        image: my-app
        volumes:
          - ./init.sh:/init.sh
        entrypoint: ["/bin/bash", "/init.sh"]
    ```
    

注意事项：

- 前台运行：容器的最后一条命令必须是阻塞式的（如启动服务），否则容器执行完脚本后会立即退出。
- 权限：如果挂载脚本，确保该文件有执行权限 (`chmod +x init.sh`)。

您目前的场景是需要初始化数据库还是单纯顺序执行服务？