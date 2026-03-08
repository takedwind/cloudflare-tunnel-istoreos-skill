---
name: Cloudflare Tunnel on iStoreOS
description: Automates the download, installation, and UCI configuration of cloudflared (Cloudflare Tunnel) on iStoreOS or OpenWRT routers.
---

# Cloudflare Tunnel on iStoreOS Configurator

## Overview
This skill fully automates the deployment of an active Cloudflare Zero Trust Tunnel on an OpenWRT or iStoreOS router.

When a user requests to "install cloudflare tunnel" or "configure cloudflared" and provides a router IP, credentials, and their Cloudflare Tunnel token, you should execute the following steps exactly as described.

## Execution Steps

### Step 1: Connect to the Router
Establish an SSH connection to the router using the provided credentials.
*Note: Due to the nature of router deployments, you may need to accept the SSH fingerprint automatically (e.g., `-o StrictHostKeyChecking=no`).*

### Step 2: Determine Architecture
Run the following command to determine the router's architecture:
```bash
uname -m
```
This typically outputs `x86_64`, `aarch64`, etc.

### Step 3: Download and Install the `cloudflared` package
Based on the architecture detected in Step 2, download the appropriate `.ipk` package from the OpenWRT official releases repository to the `/tmp` directory.

Example for `x86_64`:
```bash
cd /tmp
wget https://downloads.openwrt.org/releases/24.10.5/packages/x86_64/packages/cloudflared_2025.5.0-r1_x86_64.ipk
opkg install cloudflared_2025.5.0-r1_x86_64.ipk
```
*(If the exact version 2025.5.0-r1 is not available or you are on a different architecture, search or infer the correct OpenWRT package URL for `cloudflared` before downloading).*

### Step 4: Stop and Clean Legacy Configurations
To ensure a clean slate, stop the service and remove old configurations:
```bash
/etc/init.d/cloudflared stop
rm -rf /etc/config/cloudflared
rm -rf /etc/cloudflared
rm -f /var/log/cloudflared.log
```

### Step 5: Initialize Clean Environment & Recreate Configuration Files
**CRITICAL:** Sometimes the `opkg` installer leaves the config template as `cloudflared-opkg` or fails to recreate the directories.
```bash
mkdir -p /etc/cloudflared
```

### Step 6: Configure the Token via UCI
Use the OpenWRT Unified Configuration Interface (UCI) to set the token and enable the service. Replace `$TOKEN` with the user's provided token.

```bash
uci set cloudflared.config.token='$TOKEN'
uci set cloudflared.config.enabled='1'
uci commit cloudflared
```

### Step 7: Enable and Restart the Service
```bash
/etc/init.d/cloudflared enable
/etc/init.d/cloudflared restart
```

### Step 8: Verify Connectivity
Wait a few seconds, then check the logs:
```bash
tail -n 20 /var/log/cloudflared.log
```
Look for a line containing `Registered tunnel connection`. If present, the tunnel is active.

---

## 适用于中文用户的说明 (Chinese Version)

### 步骤 1：连接到路由器
使用提供的凭证建立到路由器的 SSH 连接。(建议使用 `-o StrictHostKeyChecking=no`)

### 步骤 2：确定架构
运行 `uname -m` 确定路由器的架构类型（例如 `x86_64` 或 `aarch64`）。

### 步骤 3：下载并安装 `cloudflared`
根据检测到的架构，从 OpenWRT 官方源下载 `.ipk` 到 `/tmp` 并使用 `opkg` 安装。
```bash
cd /tmp
wget https://downloads.openwrt.org/releases/24.10.5/packages/x86_64/packages/cloudflared_2025.5.0-r1_x86_64.ipk
opkg install cloudflared_2025.5.0-r1_x86_64.ipk
```

### 步骤 4：停止并清理旧配置
```bash
/etc/init.d/cloudflared stop
rm -rf /etc/config/cloudflared
rm -rf /etc/cloudflared
rm -f /var/log/cloudflared.log
```

### 步骤 5：初始化纯净环境
**注意：** 安装包有时不会自动创建工作目录，必须手动创建，防止配置下发失败。
```bash
mkdir -p /etc/cloudflared
```

### 步骤 6：通过 UCI 写入 Token
```bash
uci set cloudflared.config.token='用户提供的TOKEN'
uci set cloudflared.config.enabled='1'
uci commit cloudflared
```

### 步骤 7：启用并重启服务
```bash
/etc/init.d/cloudflared enable
/etc/init.d/cloudflared restart
```

### 步骤 8：验证连接
使用 `tail -n 20 /var/log/cloudflared.log` 检查日志。需要看到 `Registered tunnel connection` 字样。
