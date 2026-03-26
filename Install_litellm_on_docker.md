# 基于 **Docker Compose**的 LiteLLM 部署
---

## 1. 部署准备

### 1.1 创建目录结构
```bash
mkdir -p /home/hysclaw/ai/litellm/data   # 存放 PostgreSQL 数据
touch /home/hysclaw/ai/litellm/config.yaml   # 配置文件
```

### 1.2 编写 `docker-compose.yml`
文件内容如下（与你实际使用的完全一致）：

```yaml
services:
  postgres:
    image: postgres:16-alpine
    container_name: litellm-postgres
    environment:
      POSTGRES_USER: litellm
      POSTGRES_PASSWORD: litellm123
      POSTGRES_DB: litellm
    volumes:
      - /home/hysclaw/ai/litellm/data:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    restart: unless-stopped
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U litellm"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - ai-network

  litellm:
    image: ghcr.io/berriai/litellm:main-v1.80.8-stable.1
    platform: linux/amd64
    container_name: litellm-proxy
    ports:
      - "4000:4000"
    volumes:
      - /home/hysclaw/ai/litellm/config.yaml:/app/config.yaml
    environment:
      DATABASE_URL: "postgresql://litellm:litellm123@postgres:5432/litellm"
      LITELLM_MASTER_KEY: "sk-1234"
    command:
      - "--config=/app/config.yaml"
      - "--port=4000"
      - "--detailed_debug"
    depends_on:
      postgres:
        condition: service_healthy
    restart: unless-stopped
    networks:
      - ai-network

networks:
  ai-network:
    external: true
```

### 1.3 准备 `config.yaml` 配置文件
根据你的需求，在 `config.yaml` 中预先定义模型。例如：

```yaml
model_list:
  - model_name: qwen3:8b
    litellm_params:
      model: ollama/qwen3:8b
      api_base: http://host.docker.internal:11434
  - model_name: gpt-4o
    litellm_params:
      model: gpt-4o
      api_key: sk-xxx   # 你的 OpenAI API Key
```

> **说明**：`host.docker.internal` 用于容器内访问宿主机的服务（如本地 Ollama）。如果 Ollama 监听在 `0.0.0.0:11434`，此地址即可连通。

### 1.4 创建外部 Docker 网络
由于 `docker-compose.yml` 中声明了 `ai-network` 为外部网络，需提前创建：
```bash
docker network create ai-network
```

---

## 2. 启动服务

```bash
cd /home/hysclaw/ai/litellm
docker compose up -d
```

### 2.1 验证容器状态
```bash
docker compose ps
docker logs litellm-proxy   # 查看 LiteLLM 日志
docker logs litellm-postgres # 查看 PostgreSQL 日志
```

正常启动后，两个容器均应处于 `Up` 状态，LiteLLM 日志中应显示 `Server started on port 4000`。

---

## 3. 访问与配置

### 3.1 登录管理界面
浏览器打开 `http://localhost:4000`  
- 用户名：`admin`  
- 密码：`sk-1234`（即环境变量 `LITELLM_MASTER_KEY`）

### 3.2 管理模型与密钥
- 模型已在 `config.yaml` 中定义，登录后可在 **Models + Endpoints** 页面查看和管理。  
- 如需新增模型，可通过管理界面添加，或在 `config.yaml` 中添加后重启容器（`docker compose restart litellm`）。  
- 在 **Virtual Keys** 页面生成虚拟密钥，用于后续应用调用。

---

## 4. 集成 Open WebUI

在 Open WebUI 的设置中，配置 OpenAI 兼容接口：
- **API 地址**：`http://<宿主机IP>:4000/v1`  
  （若 Open WebUI 也运行在 Docker 中，可使用 `http://host.docker.internal:4000/v1`）  
- **API 密钥**：LiteLLM 生成的虚拟密钥  
- 保存后，Open WebUI 将自动显示 LiteLLM 中可用的模型，可直接选择对话。

---

## 5. 遇到的典型问题及解决

| 问题 | 解决方案 |
|------|----------|
| **PostgreSQL 健康检查一直失败** | 检查 PostgreSQL 容器日志，确保数据库初始化完成；若长时间不健康，可临时移除 `healthcheck` 或手动 `docker compose restart postgres`。 |
| **LiteLLM 无法连接 PostgreSQL** | 确认 `DATABASE_URL` 中的用户名、密码、数据库名与 PostgreSQL 环境变量一致，且两容器在同一网络 `ai-network` 中。 |
| **配置文件未生效** | 检查挂载路径是否正确，容器内路径 `/app/config.yaml` 必须存在；若修改配置后未生效，重启 LiteLLM 容器。 |
| **无法访问本地 Ollama** | 容器内需使用 `host.docker.internal` 替代 `localhost`；若仍不通，确认 Ollama 监听 `0.0.0.0` 并已启动。 |
| **Open WebUI 调用返回 401** | 核对 API Key 是否正确，虚拟密钥是否已绑定可用模型。 |

---

## 6. 部署结果

- **数据库持久化**：PostgreSQL 数据保存在宿主机 `/home/hysclaw/ai/litellm/data`，容器重启后数据不丢失。  
- **LiteLLM 服务稳定运行**：管理界面可正常访问，模型列表、虚拟密钥、调用日志等功能完整可用。  
- **Open WebUI 集成成功**：可通过 LiteLLM 调用本地 Ollama 模型及云端 OpenAI 模型，对话流畅。  
- **日志与监控**：LiteLLM 记录每次调用的 spend、token 等，便于成本分析和故障排查。

---

## 7. 未来作用

- **统一 API 网关**：所有应用只需对接一个 OpenAI 兼容接口，底层模型可灵活替换。  
- **权限与预算控制**：通过虚拟密钥实现多租户、速率限制、预算配额。  
- **成本追踪**：自动统计各用户/团队的用量与费用。  
- **高可用扩展基础**：当前方案已具备持久化能力，后续可增加 Redis 提升并发性能，或水平扩展多个 LiteLLM 实例。

---
