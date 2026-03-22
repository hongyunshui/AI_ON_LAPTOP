# WSL 安装

## 安装环境
   CPU：AMD Ryzen 5 PRO 7640HS w/ Radeon 760M Graphics  (4.30 GHz)
   GPU:NVIDIA RTX A500
   RAM:16GB

---

## 1. 预期安装步骤（标准流程）

1. **以管理员身份打开 PowerShell**  
2. **执行安装命令**  
   ```powershell
   wsl --install
   ```
   - 自动启用 WSL 功能
   - 安装 WSL 2 内核
   - 安装默认的 Linux 发行版（Ubuntu）
3. **重启电脑**（如有提示）  
4. **验证安装**  
   ```powershell
   wsl -l -v
   ```
   正常输出应显示已安装的发行版及 WSL 版本。

---

## 2. 实际遇到的问题

- **问题 1**：执行 `wsl -l -v` 时未显示发行版列表，而是直接输出 `wsl.exe` 的帮助信息。  
  **原因**：WSL 核心功能尚未完全启用，系统未初始化 WSL 环境。

- **问题 2**：以管理员身份执行 `wsl --install` 后，出现 **“操作超时”** 错误，无法完成下载。  
  **原因**：网络连接不稳定或被限制，导致 PowerShell 无法从微软服务器下载所需组件及 Linux 发行版。

---

## 3. 解决办法（按推荐顺序）

### 方案一：更换网络 + 使用 `--web-download` 参数
- 切换至更稳定的网络（如手机热点），在管理员 PowerShell 中执行：
  ```powershell
  wsl --install --web-download
  ```
- 若仍有超时，可尝试修改 DNS 为 `114.114.114.114` 或 `8.8.8.8` 后重试。

### 方案二：手动启用 WSL 功能 + 通过 Microsoft Store 安装发行版（成功率最高）
1. **手动启用功能**  
   - 打开“控制面板” → “程序” → “启用或关闭 Windows 功能”  
   - 勾选：  
     - **适用于 Linux 的 Windows 子系统**  
     - **虚拟机平台**  
   - 点击“确定”，完成安装后**重启电脑**。  

2. **安装 Ubuntu**  
   - 打开 Microsoft Store  
   - 搜索 **Ubuntu**（推荐 22.04 LTS 版本）  
   - 点击“获取”或“安装”，等待下载完成。  

3. **设置 WSL 2**（可选，但建议）  
   - 启动安装好的 Ubuntu，完成首次配置（设置用户名、密码）  
   - 在 PowerShell 中执行：
     ```powershell
     wsl --set-default-version 2
     wsl --set-version Ubuntu 2
     ```

### 方案三：使用 winget 安装
- 以管理员身份运行 PowerShell，执行：
  ```powershell
  winget install Canonical.Ubuntu
  ```
- 安装完成后同样需启动并配置 Ubuntu。

### 方案四：离线安装包（终极方案）
- 从国内镜像站（如清华大学 TUNA）下载 Ubuntu WSL 的 `.appx` 离线包  
- 双击安装包，按提示完成安装  

---

## 4. 当前结果

一经成功安装
