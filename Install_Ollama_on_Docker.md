# 在WSL-->Ubuntu-->Docker的环境中部署Ollama
---

# 一、部署目标

在 **WSL Ubuntu + Docker 环境**中部署 **Ollama**，实现：

1. 在本地运行大模型推理服务
2. **优先使用 GPU 推理**
3. 提供统一 API 接口
4. 为后续部署 **OpenClaw** 等 AI Agent 提供模型服务

最终目标架构：

```
AI Agent / 应用
       │
       ▼
   Ollama API
       │
       ▼
    GPU推理
```

---

# 二、系统环境

部署环境如下：

| 项目   | 配置                         |
| ---- | -------------------------- |
| 系统   | Windows + WSL Ubuntu       |
| 容器   | Docker                     |
| 编排   | Docker Compose             |
| GPU  | NVIDIA RTX A500 Laptop GPU |
| 显存   | 4GB                        |
| CUDA | 13.0                       |
| 驱动   | 581.60                     |

GPU检测：

```bash
nvidia-smi
```

结果：

```
NVIDIA RTX A500 Laptop GPU
Memory: 4096 MiB
CUDA Version: 13.0
```

说明：

```
GPU → WSL CUDA → 正常
```

---

# 三、Ollama 部署方案

采用 **Docker 部署方式**。

原因：

| 优势   | 说明            |
| ---- | ------------- |
| 环境隔离 | 不污染系统         |
| 便于升级 | 镜像更新即可        |
| 易扩展  | 可与其他 AI 服务集成  |
| 统一管理 | 通过 Compose 管理 |

---

# 四、部署流程

整个部署流程可以分为 **4个阶段**。

---

# 阶段1：Docker 环境准备

验证 Docker：

```bash
docker --version
```

输出：

```
Docker version 29.3.0
```

说明：

```
Docker安装成功
```

测试容器：

```bash
docker run hello-world
```

成功运行。

---

# 阶段2：配置 Docker 国内镜像源

问题：

从 Docker Hub 拉取镜像时出现：

```
i/o timeout
```

原因：

```
中国网络访问 docker.io 不稳定
```

解决：

编辑：

```
/etc/docker/daemon.json
```

配置镜像加速：

```json
{
    "registry-mirrors": [
        "https://docker.m.daocloud.io",
        "https://hub-mirror.c.163.com",
        "https://mirror.baidubce.com"
    ]
}
```

重启 Docker：

```bash
sudo service docker restart
```

效果：

```
docker pull 成功
```

---

# 阶段3：配置 Docker GPU Runtime

目标：

让容器访问 GPU。

需要组件：

**NVIDIA Container Toolkit**

---

## 遇到的问题

安装失败：

```bash
sudo apt install nvidia-container-toolkit
```

错误：

```
Unable to locate package
```

原因：

```
Ubuntu 默认软件源没有 NVIDIA Container Toolkit
```

---

## 解决方案

添加 NVIDIA 官方仓库。

导入密钥：

```bash
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey \
| sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg
```

添加仓库：

```bash
curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list \
| sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' \
| sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
```

更新：

```bash
sudo apt update
```

安装：

```bash
sudo apt install nvidia-container-toolkit
```

安装成功。

---

# 阶段4：配置 Docker GPU Runtime

修改：

```
/etc/docker/daemon.json
```

最终配置：

```json
{
    "registry-mirrors": [
        "https://docker.m.daocloud.io",
        "https://hub-mirror.c.163.com",
        "https://mirror.baidubce.com"
    ],
    "default-runtime": "nvidia",
    "runtimes": {
        "nvidia": {
            "args": [],
            "path": "nvidia-container-runtime"
        }
    }
}
```

作用：

| 配置               | 作用          |
| ---------------- | ----------- |
| registry-mirrors | Docker 镜像加速 |
| runtimes         | GPU runtime |
| default-runtime  | 默认支持 GPU    |

重启 Docker：

```bash
sudo service docker restart
```

---

# 五、部署 Ollama

创建目录：

```bash
mkdir ~/ollama
cd ~/ollama
```

创建：

```
docker-compose.yml
```

配置：

```yaml
version: "3"

services:

  ollama:
    image: ollama/ollama
    container_name: ollama
    restart: always
    ports:
      - "11434:11434"
    volumes:
      - ./ollama:/root/.ollama
    deploy:
      resources:
        reservations:
          devices:
            - capabilities: [gpu]
```

启动：

```bash
docker compose up -d
```

查看容器：

```bash
docker ps
```

---

# 六、验证 Ollama

测试 API：

```bash
curl http://localhost:11434
```

返回：

```
Ollama is running
```

说明服务正常。

---

# 七、模型测试

下载模型：

例如：

* Phi-3 Mini
* Qwen2

运行：

```bash
docker exec -it ollama ollama run phi3
```

同时查看 GPU：

```bash
nvidia-smi
```

如果看到：

```
ollama 进程
GPU Memory 使用
```

说明：

```
GPU 推理成功
```

---

# 八、部署结果

部署完成后系统结构：

```
Windows
│
└── WSL Ubuntu
       │
       ├── Docker
       │
       ├── NVIDIA Container Toolkit
       │
       └── Ollama
```

访问接口：

```
http://localhost:11434
```

功能：

| 功能       | 状态 |
| -------- | -- |
| Docker运行 | ✔  |
| GPU识别    | ✔  |
| GPU容器支持  | ✔  |
| Ollama运行 | ✔  |
| 模型推理     | ✔  |

---

# 九、性能建议

你的显卡：

**NVIDIA RTX A500 Laptop GPU**

显存：

```
4GB
```

推荐模型：

| 模型             | 推荐   |
| -------------- | ---- |
| Phi-3 Mini     | ⭐⭐⭐⭐ |
| Qwen2 7B       | ⭐⭐⭐  |
| DeepSeek Coder | ⭐⭐⭐  |

不推荐：

```
13B+
```

---

# 十、下一步规划

未来计划增加：

| 组件         | 作用       |
| ---------- | -------- |
| OpenClaw   | AI Agent |
| Open WebUI | 模型管理界面   |

未来架构：

```
Docker
│
├── Ollama
│
├── OpenClaw
│
└── OpenWebUI
```

---

