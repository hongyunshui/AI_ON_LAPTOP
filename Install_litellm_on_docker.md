## LiteLLM 部署总结（基于实际部署方案）

---

### 一、部署流程

#### 1. 环境准备
- 宿主机：Linux（已确认 Docker 和 Docker Compose 可用）
- 创建目录结构：
  ```bash
  mkdir -p /home/hysclaw/ai/litellm/data
  touch /home/hysclaw/ai/litellm/config.yaml
  ```

#### 2. 编写 docker-compose.yml
文件内容如下（与你实际使用的一致）：
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

#### 3. 准备配置文件 config.yaml
根据实际需要，在 `config.yaml` 中预先定义模型、预算等。例如：
```yaml
model_list:
  - model_name: qwen3:8b
    litellm_params:
      model: ollama/qwen3:8b
      api_base: http://host.docker.internal:11434
  - model_name: gpt-4o
    litellm_params:
      model: gpt-4o
      api_key: your-openai-key
```

#### 4. 创建外部网络
由于 compose 中使用了 `external: true` 的网络，需要提前创建：
```bash
docker network create ai-network
```

#### 5. 启动服务
```bash
cd /home/hysclaw/ai/litellm
docker compose up -d
```
等待镜像拉取和数据库初始化，使用以下命令检查状态：
```bash
docker compose ps
docker logs litellm-proxy
```

#### 6. 访问管理界面
浏览器打开 `http://localhost:4000`，使用 `admin` 和 `sk-1234` 登录。

#### 7. 配置模型与密钥
- 模型已在 `config.yaml` 中预定义，登录后可在界面中看到并管理。
- 生成虚拟密钥（Virtual Key），用于后续调用。

#### 8. 集成 Open WebUI
在 Open WebUI 中设置 OpenAI 兼容接口：
- **API 地址**：`http://<宿主机IP>:4000/v1`（如果 Open WebUI 也运行在 Docker 中，可能需要使用 `http://host.docker.internal:4000/v1`）
- **API 密钥**：生成的虚拟密钥
保存后即可在 Open WebUI 中选择 LiteLLM 中暴露的模型进行对话。

---

### 二、遇到的问题及解决方法

| 问题 | 现象 | 解决方案 |
|------|------|----------|
| **PostgreSQL 健康检查失败** | `litellm` 容器一直等待 `postgres` 就绪，日志显示 `database is not ready` | 检查 PostgreSQL 容器日志，确认数据库已正确初始化。最终健康检查通过，服务启动。 |
| **网络连接问题** | 容器无法通过 `postgres` 主机名解析 | 确保两个容器在同一个自定义网络 `ai-network` 中，且该网络已创建。 |
| **配置文件挂载不生效** | LiteLLM 启动时未读取 `config.yaml` 中的模型 | 检查挂载路径是否正确，容器内路径 `/app/config.yaml` 必须存在；同时确认 `command` 中包含 `--config=/app/config.yaml`。 |
| **本地 Ollama 无法访问** | 添加 Ollama 模型后调用失败，提示连接拒绝 | 在容器内，宿主机 `localhost` 不可达，应使用 `host.docker.internal`（Linux 下需添加 `--add-host=host.docker.internal:host-gateway` 或使用宿主机 IP）。本例中 `config.yaml` 已使用 `http://host.docker.internal:11434`，若仍不通，检查 Ollama 是否监听 `0.0.0.0`。 |
| **Open WebUI 调用返回 401** | 密钥无效 | 确认 Open WebUI 中填写的 API Key 与 LiteLLM 生成的虚拟密钥完全一致，且该密钥已绑定可用模型。 |

---

### 三、最终结果

- **数据库持久化**：PostgreSQL 容器将数据存储于宿主机 `/home/hysclaw/ai/litellm/data`，重启容器配置不丢失。
- **LiteLLM 服务稳定运行**：管理界面可访问，模型列表、虚拟密钥、调用日志均正常。
- **与 Open WebUI 成功集成**：在 Open WebUI 中可通过 LiteLLM 调用本地 Ollama 模型和云端 OpenAI 模型，对话流畅。
- **调用日志可查**：在 LiteLLM 管理界面的 `Spend Logs` 中可看到每次请求的详细信息，便于监控和成本核算。

---

### 四、未来作用与价值

1. **统一 API 网关**  
   所有上层应用（Open WebUI、其他客户端）只需对接一个 OpenAI 兼容接口，无需关心后端模型的具体部署位置和认证方式。

2. **多模型管理与切换**  
   通过 `config.yaml` 或管理界面动态添加/删除模型，实现模型的集中管理和无感切换。

3. **权限控制与预算管理**  
   可创建多个虚拟密钥，为不同用户/团队分配不同的模型访问权限、速率限制和预算配额，适合企业内部使用。

4. **使用量追踪与成本分析**  
   自动记录每次调用的 token 消耗和费用，便于成本分摊和用量审计。

5. **高可用扩展基础**  
   当前方案已使用 PostgreSQL 持久化，后续可增加 Redis 作为缓存，提升高并发下的性能，并支持水平扩展。

6. **模型访问安全**  
   通过虚拟密钥将真实 API Key 隐藏，避免在前端暴露敏感信息；可设置 IP 白名单、组织管理等增强安全。

---

通过本次部署，已成功构建了一个灵活、可扩展的 LLM 统一入口，为 AI 应用开发和模型管理打下了坚实基础。接下来优化方向：添加 Redis、配置告警、对接更多模型提供商。
