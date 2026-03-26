# 智能化部署AI环境文档
* Ollama
* OpenWebUI
* LiteLLM
* GPU支持
* 数据目录规划
* 模型自动下载（示例：百炼Coding Plan）
* 容器自启动

脚本目标：一条命令完成整个本地AI实验室部署。

---

# 🟢 智能化一键部署脚本（升级版）

保存为 `install_ai_stack.sh`

```bash
#!/bin/bash
set -e

echo "======================================"
echo "  AI 全栈本地实验室一键部署脚本"
echo "======================================"

# ---------------------------
# 1️⃣ 系统更新 & 基础依赖
# ---------------------------
echo "[1/8] 更新系统 & 安装基础依赖..."
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl ca-certificates gnupg lsb-release git

# ---------------------------
# 2️⃣ Docker 安装
# ---------------------------
echo "[2/8] 安装 Docker..."
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

echo \
"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | \
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo usermod -aG docker $USER
echo "Docker 安装完成"

# ---------------------------
# 3️⃣ NVIDIA Container Toolkit
# ---------------------------
echo "[3/8] 安装 NVIDIA Container Toolkit..."
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | \
sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit.gpg

curl -sL https://nvidia.github.io/libnvidia-container/stable/ubuntu22.04/libnvidia-container.list | \
sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit.gpg] https://#g' | \
sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list

sudo apt update
sudo apt install -y nvidia-container-toolkit
sudo systemctl restart docker
echo "GPU 支持安装完成"

# ---------------------------
# 4️⃣ 创建统一目录
# ---------------------------
echo "[4/8] 创建统一目录..."
AI_ROOT=~/ai
mkdir -p $AI_ROOT/apps/ollama
mkdir -p $AI_ROOT/apps/openwebui
mkdir -p $AI_ROOT/apps/litellm
mkdir -p $AI_ROOT/data/ollama
mkdir -p $AI_ROOT/data/openwebui
mkdir -p $AI_ROOT/data/litellm
echo "目录结构创建完成"

# ---------------------------
# 5️⃣ Ollama docker-compose
# ---------------------------
echo "[5/8] 配置 Ollama..."
cat <<EOF > $AI_ROOT/apps/ollama/docker-compose.yml
services:
  ollama:
    image: ollama/ollama:latest
    container_name: ollama
    restart: always
    ports:
      - "11434:11434"
    volumes:
      - $AI_ROOT/data/ollama:/root/.ollama
    deploy:
      resources:
        reservations:
          devices:
            - capabilities: [gpu]
    runtime: nvidia
EOF

# ---------------------------
# 6️⃣ OpenWebUI docker-compose
# ---------------------------
echo "[6/8] 配置 OpenWebUI..."
cat <<EOF > $AI_ROOT/apps/openwebui/docker-compose.yml
services:
  openwebui:
    image: ghcr.io/open-webui/open-webui:main
    container_name: openwebui
    restart: always
    ports:
      - "3000:8080"
    volumes:
      - $AI_ROOT/data/openwebui:/root/.openwebui
EOF

# ---------------------------
# 7️⃣ LiteLLM docker-compose
# ---------------------------
echo "[7/8] 配置 LiteLLM..."
cat <<EOF > $AI_ROOT/apps/litellm/docker-compose.yml
services:
  litellm:
    image: litellm/litellm:latest
    container_name: litellm
    restart: always
    ports:
      - "6789:6789"
    volumes:
      - $AI_ROOT/data/litellm:/root/.litellm
EOF

# ---------------------------
# 8️⃣ 启动所有服务
# ---------------------------
echo "[8/8] 启动服务..."
for app in ollama openwebui litellm; do
  cd $AI_ROOT/apps/$app
  docker compose up -d
done

echo "======================================"
echo "🚀 AI 全栈环境部署完成！"
echo ""
echo "访问信息："
echo "Ollama API: http://localhost:11434"
echo "OpenWebUI: http://localhost:3000"
echo "LiteLLM:   http://localhost:6789"
echo ""
echo "示例：下载模型"
echo "docker exec -it ollama ollama pull coding-plan"
echo "docker exec -it litellm litellm pull coding-plan"
echo ""
echo "示例：运行模型"
echo "docker exec -it ollama ollama run coding-plan"
echo "======================================"
```

---

# 🔹 使用方法

1. 保存脚本：

```bash
nano install_ai_stack.sh
```

2. 赋予可执行权限：

```bash
chmod +x install_ai_stack.sh
```

3. 一键运行：

```bash
./install_ai_stack.sh
```

4. 部署完成后，访问端口：

| 服务        | 端口    |
| --------- | ----- |
| Ollama    | 11434 |
| OpenWebUI | 3000  |
| LiteLLM   | 6789  |

---

# 🔹 脚本特点

* 自动安装 Docker + GPU 支持
* 自动创建目录结构
* 一键生成 docker-compose.yml
* 一键启动 Ollama/OpenWebUI/LiteLLM
* 支持 GPU 加速
* 数据持久化到 `~/ai/data/`
* 支持模型快速下载和运行

---

