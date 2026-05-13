宿主機訪問 Docker 橋接子網（Bridge Network）的實現方式取決於你的操作系統。

## 1. Linux 宿主機 (原生支持)

在 Linux 上，Docker 會在宿主機創建一個虛擬網橋（通常是 `docker0`），宿主機可以直接透過路由訪問該子網下的容器 IP。 [1, 2]

- 直接訪問： 你可以直接在宿主機 `ping` 容器 IP（例如 `ping 172.17.0.2`）。
- 查看子網段： 使用命令 `docker network inspect bridge` 獲取子網範圍（Subnet）。
- 查看網關 IP： 通常宿主機在橋接網絡中的 IP（網關）是 `172.17.0.1`。 [3, 4, 5, 6, 7]

---

## 2. macOS / Windows 宿主機 (需特殊處理)

在 Mac 和 Windows 上，Docker 運行在虛擬機（VM）中，宿主機與容器子網之間存在網絡隔離，無法直接透過容器 IP 訪問。 [8, 9]

## 推薦解決方案：

- 端口映射 (Port Forwarding): 這是最標準的做法。啟動容器時使用 `-p` 參數將容器端口映射到宿主機端口（例如 `docker run -p 8080:80`），然後透過 `localhost:8080` 訪問。
- 使用輔助工具 (實現直接 IP 訪問):
    
    - macOS: 可以安裝 docker-mac-net-connect 或 docker-connector 來打通宿主機到 Docker VM 內部的路由。
    - Windows: 通常只能依賴端口映射，或使用 WSL2 內部的 IP 進行通信。 [9, 10, 11, 12]
    

---

## 3. 網絡配置檢查清單

如果您在 Linux 上仍無法訪問，請檢查以下設置：

|檢查項 [13, 14, 15, 16, 17]|說明|
|---|---|
|IP Forwarding|確保內核已開啟轉發：`sysctl net.ipv4.ip_forward` 應為 `1`。|
|iptables 規則|Docker 會自動配置防火牆規則。檢查是否被 `DOCKER-ISOLATION` 鏈截斷。|
|子網衝突|確保 Docker 子網（默認 `172.17.0.0/16`）與宿主機所在的物理局域網 IP 段不重疊。|

您目前的環境是 Linux 還是 macOS/Windows？

  

[1] [https://docs.docker.com](https://docs.docker.com/engine/network/drivers/bridge/)

[2] [https://www.qg.net](https://www.qg.net/doc/899.html)

[3] [https://indumathimanivannan.medium.com](https://indumathimanivannan.medium.com/docker-network-modes-explained-bridge-host-and-overlay-comparisons-d691857f9d30)

[4] [https://stackoverflow.com](https://stackoverflow.com/questions/39070547/how-to-expose-a-docker-network-to-the-host-machine)

[5] [https://romanglushach.medium.com](https://romanglushach.medium.com/navigating-the-docker-network-a-comprehensive-guide-52c251f674f6)

[6] [https://modelers.csdn.net](https://modelers.csdn.net/69a784b17bbde9200b9cfd5a.html)

[7] [https://medium.com](https://medium.com/@haroldfinch01/how-to-access-host-port-from-docker-container-c5045552ca42)

[8] [https://forums.docker.com](https://forums.docker.com/t/access-from-local-network/133079)

[9] [https://zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/653707214)

[10] [https://medium.com](https://medium.com/@tylerauerbeck/making-your-docker-network-reachable-in-osx-e68f998f8249)

[11] [https://www.reddit.com](https://www.reddit.com/r/docker/comments/1clcwxi/new_docker_user_how_do_i_access_bridge_network/)

[12] [https://dev.to](https://dev.to/caffinecoder54/docker-networking-deep-dive-understanding-bridge-host-and-overlay-networks-1kac)

[13] [https://superuser.com](https://superuser.com/questions/1353208/how-to-reach-a-docker-container-from-a-device-that-is-in-the-same-network-as-the)

[14] [https://stackoverflow.com](https://stackoverflow.com/questions/57610988/access-docker-container-in-different-subnet-bridge)

[15] [https://stackoverflow.com](https://stackoverflow.com/questions/57610988/access-docker-container-in-different-subnet-bridge)

[16] [https://developer.aliyun.com](https://developer.aliyun.com/article/689340)

[17] [https://oneuptime.com](https://oneuptime.com/blog/post/2026-02-08-how-to-fix-docker-bridge-network-subnet-conflicts-with-host/view)