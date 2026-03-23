# 基于WSL2的Ubuntu的Docker安装

1️⃣ 安装目标
2️⃣ 安装流程
3️⃣ 遇到的问题
4️⃣ 解决方法
5️⃣ 最终结果

---

# 一、安装目标

在 **WSL2 的 Ubuntu 22.04 系统中安装 Docker Engine**，用于：

* 运行容器服务
* 部署开发环境
* 运行 AI / DevOps / 数据库服务

最终目标架构：

```
Windows
   ↓
WSL2
   ↓
Ubuntu 22.04
   ↓
Docker Engine
   ↓
Containers
```

---

# 二、Docker 安装流程

Docker 官方安装流程分为 **4个步骤**。

---

# 1 安装依赖工具

首先更新软件包索引：

```bash
sudo apt update
```

安装必要工具：

```bash
sudo apt install -y ca-certificates curl gnupg
```

作用：

| 软件              | 用途        |
| --------------- | --------- |
| ca-certificates | HTTPS证书验证 |
| curl            | 下载远程文件    |
| gnupg           | 验证软件签名    |

---

# 2 添加 Docker 官方 GPG 密钥

创建 key 目录：

```bash
sudo mkdir -p /etc/apt/keyrings
```

下载 Docker GPG Key：

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
| sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

作用：

```
验证 Docker 软件包的安全性
```

---

# 3 添加 Docker 软件仓库

执行：

```bash
echo \
"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
https://download.docker.com/linux/ubuntu jammy stable" \
| sudo tee /etc/apt/sources.list.d/docker.list
```

说明：

| 参数         | 作用             |
| ---------- | -------------- |
| arch=amd64 | CPU架构          |
| jammy      | Ubuntu22.04 代号 |
| stable     | 稳定版本           |

---

# 4 更新软件源

执行：

```bash
sudo apt update
```

如果成功会看到：

```
download.docker.com
```

说明 Docker 源已经成功加入。

---

# 5 安装 Docker

执行：

```bash
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

安装组件：

| 组件                    | 作用            |
| --------------------- | ------------- |
| docker-ce             | Docker Engine |
| docker-ce-cli         | Docker 命令     |
| containerd            | 容器运行时         |
| docker-buildx-plugin  | 构建工具          |
| docker-compose-plugin | 容器编排          |

下载大小：

```
97MB
```

安装完成后系统自动启动 Docker。

---

# 三、安装过程中遇到的问题

在安装 Docker 时主要遇到了 **一个关键问题**。

---

# 问题：Docker 软件包无法找到

执行安装命令：

```bash
sudo apt install docker-ce
```

系统提示：

```
Package docker-ce is not available
```

并且出现：

```
Unable to locate package docker-ce-cli
Unable to locate package containerd.io
```

---

# 问题原因

Docker 官方仓库 **没有添加到 apt 软件源**。

查看目录：

```bash
ls /etc/apt/sources.list.d/
```

结果：

```
空目录
```

说明：

```
Docker 软件源没有成功配置
```

因此 apt 无法找到 Docker 软件包。

---

# 四、解决方法

重新添加 Docker 官方仓库。

---

## 1 创建 key 目录

```bash
sudo mkdir -p /etc/apt/keyrings
```

---

## 2 下载 Docker GPG key

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
| sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

---

## 3 添加 Docker 软件源

```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
https://download.docker.com/linux/ubuntu jammy stable" \
| sudo tee /etc/apt/sources.list.d/docker.list
```

---

## 4 更新 apt

```bash
sudo apt update
```

成功出现：

```
Get: https://download.docker.com/linux/ubuntu jammy InRelease
```

说明仓库成功添加。

---

## 5 重新安装 Docker

执行：

```bash
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

这次安装成功。

---

# 五、Docker 服务验证

安装完成后检查 Docker 状态：

```bash
systemctl status docker
```

系统输出：

```
Active: active (running)
```

说明：

```
Docker 服务已经启动
```

Docker 进程：

```
dockerd
```

---

# 六、Docker 版本验证

执行：

```bash
docker --version
```

输出：

```
Docker version 29.3.0
```

说明 Docker 已成功安装。

---

# 七、docker compose 验证

执行：

```bash
docker compose ps
```

系统提示：

```
no configuration file provided
```

说明：

```
docker compose 已安装
但当前目录没有 compose 配置文件
```

属于正常现象。

---

# 八、最终安装结果

Docker 安装成功。

当前系统环境：

```
Windows
   ↓
WSL2
   ↓
Ubuntu 22.04
   ↓
Docker Engine 29.3
```

Docker 服务状态：

```
running
```

Docker CLI：

```
docker
docker compose
docker buildx
```

全部可用。

---

# 九、当前系统能力

现在你的电脑已经具备 **容器化环境**。

可以运行：

| 类型     | 示例                 |
| ------ | ------------------ |
| 数据库    | MySQL / PostgreSQL |
| 缓存     | Redis              |
| Web服务  | Nginx              |
| AI模型   | Ollama             |
| DevOps | Gitlab / Jenkins   |

例如运行测试容器：

```bash
docker run hello-world
```

如果成功会输出：

```
Hello from Docker!
```

说明 Docker 完全正常。

---


