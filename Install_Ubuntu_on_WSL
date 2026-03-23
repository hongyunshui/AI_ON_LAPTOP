**安装 Ubuntu 在 WSL 中**，

* 目标
* 安装步骤
* 遇到的问题
* 解决方法
* 最终结果

这是一份 **WSL + Ubuntu 部署记录**。

---

# 一、目标

在 **Windows 系统**中部署一个 **完整的 Linux 环境（Ubuntu 22.04）**，用于：

* 运行 Linux 软件
* 部署 Docker
* 运行 AI / DevOps / 开发环境

实现架构：

```text
Windows
   ↓
WSL2
   ↓
Ubuntu 22.04
```

---

# 二、Ubuntu 安装步骤

## 1 开启 WSL 功能

在 **PowerShell（管理员）**执行：

```powershell
wsl --install
```

作用：

* 启用 Windows Subsystem for Linux
* 启用 Hyper-V 虚拟化组件
* 安装默认 Linux 发行版

---

## 2 设置 WSL2 为默认版本

执行：

```powershell
wsl --set-default-version 2
```

原因：

WSL 有两个版本：

| 版本   | 特点          |
| ---- | ----------- |
| WSL1 | 兼容层         |
| WSL2 | 完整 Linux 内核 |

WSL2 性能更好，因此设为默认。

---

## 3 查看当前 WSL 状态

执行：

```powershell
wsl --status
```

结果：

```text
默认分发: Ubuntu-22.04
默认版本: 2
```

说明：

系统默认 Linux 发行版是 **Ubuntu 22.04**。

---

## 4 查看安装的 Linux 系统

执行：

```powershell
wsl -l -v
```

结果：

```text
NAME            STATE     VERSION
Ubuntu-22.04    Stopped   2
```

说明：

* Ubuntu 已安装
* 使用 WSL2

---

## 5 启动 Ubuntu

执行：

```powershell
wsl -d Ubuntu-22.04
```

第一次启动会出现初始化提示：

```text
Provisioning the new WSL instance Ubuntu-22.04
Create a default Unix user account:
```

系统要求创建 Linux 用户：

例如：

```text
username: hysclaw
password: ********
```

创建完成后进入 Ubuntu：

```text
hysclaw@WIN-xxxx:~$
```

说明 Ubuntu 安装成功。

---

# 三、安装过程中遇到的问题

在安装 Ubuntu 时，主要遇到 **两个现象**。

---

# 问题一

## WSL 代理提示

启动 Ubuntu 时系统提示：

```text
wsl: 检测到 localhost 代理配置，但未镜像到 WSL。
NAT 模式下的 WSL 不支持 localhost 代理
```

原因：

Windows 使用了代理软件：

**v2rayN**

代理端口：

| 类型    | 端口    |
| ----- | ----- |
| SOCKS | 10808 |
| HTTP  | 10809 |

但 WSL 默认 **不能直接使用 Windows localhost 代理**。

---

## 解决办法

有两种方式：

### 方法1：使用 Windows IP

WSL 访问：

```text
http://Windows_IP:10809
```

而不是：

```text
localhost:10809
```

---

### 方法2（最终采用）

启用 **WSL Mirrored 网络模式**

优点：

* WSL 与 Windows 同一 IP
* localhost 可以互通
* 代理可以直接使用

---

# 问题二

## WSL 网络 IP 变化

查询 Windows 网络：

```powershell
ipconfig
```

发现：

```text
vEthernet (WSL)
IPv4: 172.22.144.1
```

这是 WSL 默认 **NAT 网络模式**。

特点：

```
Windows IP: 172.22.144.1
WSL IP: 172.22.x.x
```

WSL 与 Windows 不同 IP。

---

## 解决方法

切换为 **Mirrored 网络模式**。

特点：

```
Windows IP = WSL IP
```

网络关系：

```
Internet
   │
Router
   │
Windows + WSL
```

优点：

* 网络一致
* localhost 通信
* Docker 更稳定

---

# 四、Ubuntu 系统验证

进入 Ubuntu 后执行：

```bash
lsb_release -a
```

输出：

```text
Distributor ID: Ubuntu
Description:    Ubuntu 22.04.5 LTS
Release:        22.04
Codename:       jammy
```

说明：

系统安装成功。

---

# 五、Ubuntu 系统结构

当前 Ubuntu 运行在：

```
Windows
   ↓
Hyper-V
   ↓
WSL2 虚拟机
   ↓
Ubuntu 22.04
```

特点：

| 项目   | 传统虚拟机 | WSL  |
| ---- | ----- | ---- |
| 启动时间 | 30s   | 1s   |
| 内存占用 | 高     | 低    |
| 磁盘   | 固定    | 动态   |
| 性能   | 中     | 接近原生 |

---

# 六、最终结果

Ubuntu 成功运行：

```text
hysclaw@WIN-D40SOJ4D0TF:~$
```

系统版本：

```
Ubuntu 22.04.5 LTS
```

WSL 状态：

```
WSL Version: 2
```

说明：

Linux 环境已经完全可用。

---

# 七、当前系统状态


```
Windows 11
   ↓
WSL2
   ↓
Ubuntu 22.04
```

随后在 Ubuntu 中又完成了：

```
Docker 29.3
```

所以现在整体环境已经变成：

```
Windows
   ↓
WSL2
   ↓
Ubuntu
   ↓
Docker
```

---

# 八、这一阶段完成的意义

这一阶段其实已经完成 **开发环境的基础设施**。

这套环境可以：

* 学习 Linux
* 部署 Docker
* 运行数据库
* 运行 AI 模型
* 做 DevOps 实验

基本等同于一台 **Linux服务器实验环境**。

---

