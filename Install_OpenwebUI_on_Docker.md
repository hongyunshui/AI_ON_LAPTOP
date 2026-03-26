#`OpenWebUI` 的部署流程**。

当前机器环境（Win11 + WSL2 + Docker + Ollama）

---

# 一、部署目标

在本地 AI 环境中部署 **Open WebUI**，作为统一的 AI 交互界面，用来：

* 调用 **Ollama** 本地模型
* 接入云端模型（阿里百炼 Coding Plan）
* 未来接入 AI Agent（OpenClaw）

最终访问方式：

```
http://localhost:3000
```

---

# 二、部署环境

当前运行环境：

```
Windows 11
   │
WSL2
   │
Ubuntu 22.04
   │
Docker
   │
OpenWebUI 容器
```

AI 服务结构：

```
ai-network
 ├─ ollama
 └─ openwebui
```

---

# 三、目录规划

根据整体 AI 项目架构，OpenWebUI 目录规划如下：

```
~/ai
 ├─ apps
 │   └─ openwebui
 │        └─ docker-compose.yml
 │
 └─ openwebui
      └─ data
```

说明：

| 目录                    | 用途            |
| --------------------- | ------------- |
| `~/ai/apps/openwebui` | 存放 compose 文件 |
| `~/ai/openwebui/data` | 存放持久化数据       |

---

# 四、创建 Docker 网络

为了让多个 AI 服务互相通信，需要创建统一网络：

```bash
docker network create ai-network
```

作用：

```
ai-network
 ├─ ollama
 ├─ openwebui
 └─ (未来 openclaw)
```

这样容器之间可以通过 **容器名通信**。

例如：

```
http://ollama:11434
```

---

# 五、编写 docker-compose.yml

文件位置：

```
~/ai/apps/openwebui/docker-compose.yml
```

内容：

```yaml
services:

  openwebui:
    image: ghcr.io/open-webui/open-webui:main
    container_name: openwebui

    ports:
      - "3000:8080"

    environment:
      - OLLAMA_BASE_URL=http://ollama:11434

    volumes:
      - /home/hysclaw/ai/openwebui/data:/app/backend/data

    networks:
      - ai-network

    restart: unless-stopped


networks:
  ai-network:
    external: true
```

---

# 六、配置说明

## 1 端口映射

```
3000:8080
```

含义：

| 主机   | 容器              |
| ---- | --------------- |
| 3000 | OpenWebUI Web端口 |

访问地址：

```
http://localhost:3000
```

---

## 2 Ollama连接

```
OLLAMA_BASE_URL=http://ollama:11434
```

原因：

OpenWebUI 与 Ollama 在 **同一个 Docker 网络** 中。

Docker 会自动解析：

```
ollama → 容器IP
```

---

## 3 数据持久化

```
/home/hysclaw/ai/openwebui/data
```

映射到：

```
/app/backend/data
```

保存内容：

* 用户账户
* 对话记录
* API配置
* 模型连接
* UI设置

这样删除容器也不会丢数据。

---

## 4 网络配置

```
external: true
```

表示：

使用已经存在的网络：

```
ai-network
```

而不是由 compose 自动创建。

---

# 七、启动 OpenWebUI

进入目录：

```bash
cd ~/ai/apps/openwebui
```

启动容器：

```bash
docker compose up -d
```

查看运行状态：

```bash
docker ps
```

正常状态：

```
openwebui   Up
```

---

# 八、访问 OpenWebUI

浏览器打开：

```
http://localhost:3000
```

首次使用需要：

```
创建管理员账号
```

之后即可进入 WebUI。

---

# 九、验证 Ollama 连接

在 OpenWebUI 中：

```
Settings
→ Models
```

如果成功连接 Ollama，会自动看到：

```
qwen
llama
mistral
```

等本地模型。

---

# 十、接入云模型

在 OpenWebUI 中：

```
Settings
→ Connections
→ Add Connection
```

配置阿里百炼 Coding Plan：

Base URL

```
https://coding.dashscope.aliyuncs.com/v1
```

API Key

```
sk-sp-xxxx
```

Provider

```
OpenAI
```

接口类型：

```
Chat Completions
```

---

# 十一、部署结果

OpenWebUI 成功实现：

### 本地模型调用

```
OpenWebUI
   │
   └── Ollama
```

---

### 云模型调用

```
OpenWebUI
   │
   └── Alibaba Coding Plan
```

---

# 十二、最终 AI 架构

当前环境：

```
Docker
 │
ai-network
 │
 ├─ ollama
 │
 └─ openwebui
      │
      ├─ 本地模型
      │    └─ Ollama
      │
      └─ 云模型
           └─ Coding Plan
```

未来扩展：

```
ai-network
 ├─ ollama
 ├─ openwebui
 └─ openclaw
```

---

✅ **总结：**

OpenWebUI 是通过 **Docker Compose + 外部 Docker 网络 + Ollama API** 的方式部署，实现了 **本地模型 + 云模型统一管理的 AI 交互平台**。

---
