# 阿里云百炼 Coding Plan 模型通过 LiteLLM 接入 OpenWebUI 

---

## 1. 处理细节

### 1.1 架构设计
- **核心组件**：OpenWebUI（前端对话界面） → LiteLLM（模型网关） → 阿里云百炼 Coding Plan（后端模型） + 本地 Ollama
- **通信方式**：OpenWebUI 通过 OpenAI API 兼容格式调用 LiteLLM，LiteLLM 根据配置将请求转发至对应模型服务
- **持久化**：使用 PostgreSQL 容器存储 LiteLLM 的密钥、日志、模型配置等信息
- **网络**：所有容器加入同一个 Docker 网络（`ai-network`），通过容器名互相访问

### 1.2 关键配置步骤

#### （1）LiteLLM 配置文件 `config.yaml`
- 正确填写百炼 Coding Plan 的 **API URL**、**模型名**、**API Key**
  - URL：`https://coding.dashscope.aliyuncs.com/v1`（国内版）
  - 模型名：如 `qwen3-max-2026-01-23`（需与百炼控制台实际名称一致）
  - 密钥：`sk-sp-xxx` 格式的专属密钥
- 添加本地 Ollama 模型配置（通过 `host.docker.internal` 访问宿主机）
- 开启 `store_model_in_db: true`，允许 WebUI 动态管理模型

#### （2）`docker-compose.yml` 配置
- PostgreSQL 和 LiteLLM 两个服务
- PostgreSQL 数据卷挂载实现持久化
- LiteLLM 环境变量：`DATABASE_URL`、`LITELLM_MASTER_KEY`、`STORE_MODEL_IN_DB`
- 加入已存在的 Docker 网络：`networks: - ai-network`，并声明 `external: true`

#### （3）OpenWebUI 集成
- 在 OpenWebUI 的设置中添加新连接：
  - URL：`http://litellm-proxy:4000`（使用容器名）
  - 密钥：LiteLLM 中创建的 Virtual Key
- 连接成功后，OpenWebUI 的模型列表中会自动显示 LiteLLM 代理的所有模型

---

## 2. 遇到的问题及解决方案

| 问题 | 表现 | 原因 | 解决方案 |
|------|------|------|----------|
| **URL 错误** | 调用 `coding-intl.dashscope.aliyuncs.com` 返回 `invalid_access_token` | 混淆了国际版与国内版域名 | 改用 `coding.dashscope.aliyuncs.com/v1`（从成功的 curl 测试中确认） |
| **YAML 格式错误** | `config.yaml` 中写了多个模型，但 LiteLLM 只加载最后一个 | 每个模型配置缺少独立的 `-` 开头，导致被当作同一个列表项 | 修正 YAML 缩进，每个模型前加 `-` 和正确层级 |
| **数据库初始化** | 启动后担心需要手动初始化 | LiteLLM 容器启动时自动运行 Prisma 迁移 | 无需手动操作，只需确保 `DATABASE_URL` 正确；通过日志确认迁移成功 |
| **容器网络隔离** | OpenWebUI 容器无法访问 LiteLLM | 未加入同一 Docker 网络 | 在 compose 文件中为所有相关服务添加 `networks: - ai-network`，并声明该网络为 `external: true` |
| **Ollama 访问问题** | LiteLLM 容器内无法访问宿主机 Ollama | 容器网络隔离 | 使用 `http://host.docker.internal:11434` 作为 api_base（仅限 Docker for Mac/Windows） |

---

## 3. 最后结果

✅ **成功实现了 OpenWebUI 通过 LiteLLM 统一接入百炼 Coding Plan 模型和本地 Ollama 模型**。

- **LiteLLM 管理界面**（`http://localhost:4000/ui`）可正常登录，查看所有模型、密钥、调用日志
- **OpenWebUI 对话界面**（`http://localhost:3000`）中模型下拉列表包含：
  - `lite-cdp-qwen3-max-2026-01-23`（百炼）
  - `lite-cdp-qwen3.5-plus`（百炼）
  - `lite-cdp-glm-5`（百炼）
  - `lite-cdp-kimi-k2.5`（百炼）
  - `qwen-8b-local`（Ollama）
  - `llama3-local`（Ollama）
- **调用正常**：任意选择模型发送消息，能收到正确回复
- **数据持久化**：所有密钥、日志、模型配置都保存在 PostgreSQL 中，容器重启不丢失

---

## 4. 未来价值

本次架构带来的可扩展性和管理便利性：

| 价值点 | 说明 |
|--------|------|
| **统一接口** | OpenWebUI 只需对接 LiteLLM，无需关心后端模型来自何方，降低前端复杂度 |
| **多模型无缝切换** | 可在 LiteLLM 中动态添加/删除模型，OpenWebUI 中立即生效，无需修改前端代码 |
| **集中权限管理** | 通过 LiteLLM 的 Virtual Keys 功能，可为不同用户/团队分配不同的模型访问权限、设置预算和速率限制 |
| **成本可视化** | LiteLLM 自动记录每次调用的 token 消耗和费用，便于成本分析和优化 |
| **本地与云端混合部署** | 可同时接入本地开源模型（如 Ollama）和云端商业模型（如百炼），按需路由 |
| **可扩展性** | 未来接入其他模型（如 Azure OpenAI、Anthropic、Google Gemini 等）只需在 LiteLLM 添加配置，OpenWebUI 无需改动 |

这套方案非常适合团队内部统一管理多模型资源，也适合个人开发者体验和对比不同模型的效果，为后续 AI 应用开发提供了灵活、可控的基础设施。

---

**总结**：从最初遇到 URL 错误、YAML 格式问题，到最终实现 OpenWebUI 稳定调用，本次实践成功搭建了一个轻量、可扩展的多模型代理网关。后续如需增加新模型或调整访问策略，均可通过 LiteLLM 管理界面轻松完成。
