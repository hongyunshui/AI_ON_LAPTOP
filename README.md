# ThinkPad 本地 AI 环境完整部署


---

# 一、部署目标

在 **ThinkPad + Windows11** 环境中实现：

* Windows 11
* WSL2 运行 **Ubuntu 22.04**
* Ubuntu 内安装 **Docker**
* Docker 中运行 **Ollama**
* Ollama **调用 NVIDIA A500 GPU**
* 为后续 **OpenWebUI / OpenClaw / AI实验**提供模型服务

最终目标：

```
Windows11
   │
   └── WSL2 (Ubuntu 22.04)
           │
           └── Docker
                  │
                  └── Ollama (GPU加速)
```

---

# 二、WSL2 与 Ubuntu 安装

首先安装 **WSL2 + Ubuntu22.04**

### 1 安装WSL

PowerShell（管理员）

```powershell
wsl --install
```

设置WSL2

```powershell
wsl --set-default-version 2
```

---

### 2 安装 Ubuntu22.04

```powershell
wsl --install -d Ubuntu-22.04
```

安装完成后

创建 Linux 用户。

---

# 三、Docker 安装

进入 **Ubuntu**

### 1 更新系统

```bash
sudo apt update
sudo apt upgrade -y
```

---

### 2 安装依赖

```bash
sudo apt install -y ca-certificates curl gnupg
```

---

### 3 添加 Docker 官方仓库

```bash
sudo install -m 0755 -d /etc/apt/keyrings
```

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

```bash
echo \
"deb [arch=$(dpkg --print-architecture) \
signed-by=/etc/apt/keyrings/docker.gpg] \
https://download.docker.com/linux/ubuntu \
$(lsb_release -cs) stable" | \
sudo tee /etc/apt/sources.list.d/docker.list
```

---

### 4 安装 Docker

```bash
sudo apt update
```

```bash
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

---

### 5 允许普通用户运行docker

```bash
sudo usermod -aG docker $USER
```

重新登录。

测试：

```bash
docker run hello-world
```

---

# 四、配置 GPU 支持

为了让 **Docker 容器调用显卡**，安装 **NVIDIA Container Toolkit**

---

### 1 添加 NVIDIA 源

```bash
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | \
sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit.gpg
```

---

### 2 添加仓库

```bash
curl -s -L https://nvidia.github.io/libnvidia-container/stable/ubuntu22.04/libnvidia-container.list | \
sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit.gpg] https://#g' | \
sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
```

---

### 3 安装 toolkit

```bash
sudo apt update
```

```bash
sudo apt install -y nvidia-container-toolkit
```

---

### 4 重启 docker

```bash
sudo systemctl restart docker
```

---

### 5 测试 GPU

```bash
docker run --rm --gpus all nvidia/cuda:12.2.0-base-ubuntu22.04 nvidia-smi
```

如果看到 **A500 显卡信息**说明成功。

---

# 五、部署 Ollama

为了方便管理，你后来做了**目录规划**：

```
~/ai/
   ├── apps/
   │    └── ollama/
   │         └── docker-compose.yml
   │
   └── ollama/
        └── models
```

---

# 六、Ollama docker-compose

进入目录

```
~/ai/apps/ollama
```

创建

```
docker-compose.yml
```

内容：

```yaml
services:
  ollama:
    image: ollama/ollama
    container_name: ollama
    restart: always
    ports:
      - "11434:11434"

    volumes:
      - ~/ai/ollama:/root/.ollama

    deploy:
      resources:
        reservations:
          devices:
            - capabilities: [gpu]

    runtime: nvidia
```

---

# 七、启动 Ollama

```bash
sudo docker compose up -d
```

检查

```bash
docker ps
```

---

# 八、下载模型测试

例如：

### 下载模型

```bash
docker exec -it ollama ollama pull llama3
```

或

```bash
docker exec -it ollama ollama pull qwen:7b
```

---

### 运行模型

```bash
docker exec -it ollama ollama run llama3
```

---

# 九、GPU调用测试

查看 GPU 是否被使用

```bash
watch -n 1 nvidia-smi
```

如果看到

```
ollama
python
```

占用 GPU，说明成功。

---

# 十、数据持久化

你还做了一个重要优化：

**将ollama数据独立存储**

目录：

```
~/ai/ollama
```

容器内部：

```
/root/.ollama
```

好处：

* 删除容器 **模型不会丢**
* 方便迁移
* 方便备份

---

# 十一、遇到的问题

部署过程中遇到过几个问题：

### 1 Docker 镜像拉取慢

解决：

配置国内镜像

```
/etc/docker/daemon.json
```

例如：

```json
{
  "registry-mirrors": [
    "https://docker.m.daocloud.io",
    "https://hub-mirror.c.163.com"
  ]
}
```

然后

```
sudo systemctl restart docker
```

---

### 2 WSL网络问题

WSL默认：

* 与 Windows 共用网络
* 可以访问外网

后来你还讨论了：

* 全局代理
* WSL代理配置

---

### 3 数据目录规划问题

早期目录：

```
~/docker_ollama
```

后来优化为：

```
~/ai/apps/ollama
~/ai/ollama
```

并使用 **rsync 迁移数据**

---

# 十二、最终结果

最终成功实现：

✔ WSL2 Ubuntu
✔ Docker
✔ GPU支持
✔ Ollama容器
✔ 模型下载运行
✔ 数据持久化

系统结构：

```
Windows 11
   │
   └── WSL2 (Ubuntu 22.04)
         │
         └── Docker
               │
               └── Ollama
                     │
                     └── GPU (A500)
```

你现在已经可以：

* 跑 **Qwen**
* 跑 **Llama**
* 跑 **Mistral**
* 接入 **OpenWebUI**

---

# 十三、你这套架构的价值

已经搭出了一个**标准本地AI开发环境**：

能力包括：

* 本地大模型推理
* GPU推理测试
* Docker AI服务部署
* WebUI调用
* 后续 API服务


---

