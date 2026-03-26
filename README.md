# ThinkPad 本地 AI 环境完整部署


✅ **WSL2 Ubuntu**
✅ **Docker + GPU支持**
✅ **Ollama**
✅ **OpenWebUI**
✅ **LiteLLM**
✅ **部署模型（如 百炼Coding Plan）**
✅ **统一目录结构**
✅ **容器编排与持久化**

# 本文价值：

📌 私人备份文档
📌 技术博客/笔记
📌 Markdown部署文档
📌 自动化脚本模板

---

# 📌 一、总体架构（部署结构图）

```
Windows 11
  └── WSL2 (Ubuntu 22.04)
       ├── Docker Engine
       │     ├── ollama (大模型推理)
       │     ├── openwebui (Web 前端)
       │     └── litellm (模型管理代理)
       ├── 数据持久化目录
       │     ├── ollama/
       │     ├── openwebui/
       │     └── litellm/
       └── GPU (NVIDIA A500)
```

---

# 📌 二、依赖环境

## 1️⃣ Windows → 启用 WSL2

PowerShell（管理员）：

```powershell
wsl --install
wsl --set-default-version 2
wsl --install -d Ubuntu-22.04
```

---

# 📌 三、Ubuntu 环境准备

进入 Ubuntu：

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl ca-certificates gnupg lsb-release
```

---

# 📌 四、安装 Docker

### 添加 Docker 官方源：

```bash
sudo install -m 0755 -d /etc/apt/keyrings

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

echo \
"deb [arch=$(dpkg --print-architecture) \
signed-by=/etc/apt/keyrings/docker.gpg] \
https://download.docker.com/linux/ubuntu \
$(lsb_release -cs) stable" | \
sudo tee /etc/apt/sources.list.d/docker.list
```

### 安装 Docker：

```bash
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

授权当前用户使用 Docker：

```bash
sudo usermod -aG docker $USER
```

重启 shell。

测试：

```bash
docker run hello-world
```

---

# 📌 五、配置 GPU 支持 (NVIDIA Container Runtime)

```bash
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | \
sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit.gpg

curl -sL https://nvidia.github.io/libnvidia-container/stable/ubuntu22.04/libnvidia-container.list | \
sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit.gpg] https://#g' | \
sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list

sudo apt update
sudo apt install -y nvidia-container-toolkit
sudo systemctl restart docker
```

测试：

```bash
docker run --rm --gpus all nvidia/cuda:12.2.0-base-ubuntu22.04 nvidia-smi
```

---

# 📌 六、统一目录规划

为了方便管理与数据持久化：

```
~/ai/
  ├── apps/
  │     ├── ollama/
  │     │     └── docker-compose.yml
  │     ├── openwebui/
  │     │     └── docker-compose.yml
  │     └── litellm/
  │           └── docker-compose.yml
  └── data/
        ├── ollama/
        ├── openwebui/
        └── litellm/
```

这样便于：

📌 数据分离
📌 容器可随时重建
📌 容器之间共享配置

---

# 📌 七、Ollama 部署

### 1️⃣ Docker Compose

文件：`~/ai/apps/ollama/docker-compose.yml`

```yaml
services:
  ollama:
    image: ollama/ollama
    container_name: ollama
    restart: always
    ports:
      - "11434:11434"
    volumes:
      - ~/ai/data/ollama:/root/.ollama
    deploy:
      resources:
        reservations:
          devices:
            - capabilities: [gpu]
    runtime: nvidia
```

启动：

```bash
cd ~/ai/apps/ollama
docker compose up -d
```

---

# 📌 八、OpenWebUI 部署

参考你自己的仓库：

📌 Install_OpenwebUI_on_Docker.md

### 1️⃣ 目录结构：

```
~/ai/data/openwebui/
```

### 2️⃣ docker-compose.yml 大致内容（示意）

```yaml
version: "3.8"
services:
  openwebui:
    image: openwebui/openwebui:latest
    container_name: openwebui
    ports:
      - "3000:3000"
    volumes:
      - ~/ai/data/openwebui:/root/.openwebui
```

启动：

```bash
cd ~/ai/apps/openwebui
docker compose up -d
```

---

# 📌 九、LiteLLM 部署

参考：

📌 Install_litellm_on_docker.md

### 1️⃣ 目录结构

```
~/ai/data/litellm
```

### 2️⃣ docker-compose.yml（示意）

```yaml
version: "3.8"
services:
  litellm:
    image: litellm/litellm:latest
    container_name: litellm
    ports:
      - "6789:6789"
    volumes:
      - ~/ai/data/litellm:/root/.litellm
```

启动：

```bash
cd ~/ai/apps/litellm
docker compose up -d
```

---

# 📌 十、模型管理（通过 Ollama + LiteLLM）

已经成功接入了：

📌 **百炼 Coding Plan 模型**

加载方式常见：

```bash
docker exec -it ollama ollama pull coding-plan
```

或者：

```bash
docker exec -it litellm litellm pull coding-plan
```

运行测试：

```bash
docker exec -it ollama ollama run coding-plan
```

---

# 📌 十一、容器运行状态确认

查看全部：

```bash
docker ps
```

---

# 📌 十二、端口 & 访问说明

| 服务         | 默认端口  | 说明      |
| ---------- | ----- | ------- |
| Ollama API | 11434 | HTTP 接口 |
| OpenWebUI  | 3000  | 浏览器 UI  |
| LiteLLM    | 6789  | 轻量模型服务  |

访问示例：

```
http://localhost:3000
http://localhost:3000/?model=coding-plan
```

---

# 📌 十三、GPU 使用监控

```bash
watch -n1 nvidia-smi
```

确认容器是否正确调用显卡：

✔ `ollama`
✔ `openwebui`
✔ `litellm`

---

# 📌 十四、数据持久化 & 安全

所有数据都存储在：

```
~/ai/data/
```

可备份：

```bash
tar czf ai-data-backup.tar.gz ~/ai/data
```

---

# 📌 十五、自动化部署脚本（升级版）

你之前的脚本可以升级为包含：

✨ ollama
✨ openwebui
✨ litellm
✨ GPU检测
✨ 模型自动下载

例如：

```bash
#!/usr/bin/env bash

# 1. 安装 Docker
# 2. 配置 NVIDIA Runtime
# 3. 生成 3 个 docker-compose.yml
# 4. 启动服务
# 5. 下载模型
```


---

# 📌 十六、可选高级优化

✔ 使用 **Traefik / Nginx Reverse Proxy** 做统一入口
✔ 使用 **Docker Network** 实现容器互联
✔ 使用 **TLS & 认证** 做安全访问
✔ 使用 **Systemd Unit** 实现开机自动启动
✔ 集成 **OpenClaw / Web UI 扩展**

---

# 📌 十七、总结

已经搭建了：

🎯 全栈本地AI服务平台
🎯 GPU加速推理
🎯 多引擎模型管理
🎯 Web UI 可视化访问
🎯 可持久化与可备份环境

这是一个 **完整的个人AI实验室**！

---
