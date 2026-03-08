---
name: Configure Cloudflare Tunnel on iStoreOS
description: A skill to automate the installation and configuration of Cloudflare Tunnel (cloudflared) on iStoreOS / OpenWRT routers. (自动化配置 iStoreOS/OpenWRT 上的 Cloudflare Tunnel 的技能)
---

# Configure Cloudflare Tunnel on iStoreOS (自动化 iStoreOS 的 Cloudflare Tunnel 配置)

This skill enables the agent to automate the setup of a Cloudflare Tunnel on an iStoreOS or OpenWRT router. It handles everything from verifying the installation to configuring the tunnel token and starting the service.

该技能（Skill）使 AI Agent 能够自动在 iStoreOS 或 OpenWRT 路由器上设置 Cloudflare Tunnel。它处理从验证安装到配置隧道令牌（Token）并启动服务的所有细节。

## Prerequisites (前提条件)
- **Router IP & Credentials (路由器 IP 和凭据):** The agent needs SSH access to the iStoreOS router (typically `root` at `192.168.x.x`). (Agent 需要 SSH 访问 iStoreOS 路由器，通常是 `192.168.x.x` 的 `root` 用户。)
- **Cloudflared Plugin (插件提供):** The `cloudflared` package must be installed on the router. (e.g., `cloudflared_xxx_x86_64.ipk`). (路由器上必须安装 `cloudflared` 软件包。)
- **Cloudflare Tunnel Token (隧道令牌):** A valid token from the Cloudflare Zero Trust dashboard. (来自 Cloudflare Zero Trust 仪表板的有效令牌。)

## Execution Steps (执行步骤)

When a user requests to configure Cloudflare Tunnel on their iStoreOS router, follow these steps:
当用户请求在其 iStoreOS 路由器上配置 Cloudflare Tunnel 时，请执行以下步骤：

1. **Connect via SSH (通过 SSH 连接):**
   - Use the `run_command` and `send_command_input` tools to SSH into the router using the provided IP, username, and password. (使用 `run_command` 和 `send_command_input` 工具，利用提供的 IP、用户名和密码 SSH 进入路由器。)
   - Example (示例): `ssh -o StrictHostKeyChecking=no root@ROUTER_IP`

2. **Verify Installation (验证安装):**
   - Run `opkg list-installed | grep cloudflared` to ensure the package is installed. (运行 `opkg list-installed | grep cloudflared` 以确保已安装该软件包。)

3. **Clean Up Previous Configurations (清理之前的配置):**
   ```sh
   /etc/init.d/cloudflared stop
   rm -rf /etc/config/cloudflared
   rm -rf /etc/cloudflared
   rm -f /var/log/cloudflared.log
   ```

4. **Apply New Configuration via UCI (通过 UCI 应用新配置):**
   - Configure the token and enable the service using UCI (Unified Configuration Interface): (使用 UCI 配置令牌并启用服务：)
   ```sh
   uci set cloudflared.config.token='YOUR_CF_TUNNEL_TOKEN'
   uci set cloudflared.config.enabled='1'
   uci commit cloudflared
   ```

5. **Start the Service (启动服务):**
   - Restart the service to apply the new configuration. (重启服务以应用新配置。)
   ```sh
   /etc/init.d/cloudflared restart
   /etc/init.d/cloudflared status
   ```

6. **Verify Logs (验证日志):**
   - Check the logs to ensure the tunnel successfully established a connection with Cloudflare's edge servers. (检查日志以确保隧道成功与 Cloudflare 的边缘服务器建立连接。)
   ```sh
   tail -n 20 /var/log/cloudflared.log
   ```
   - Look for elements like `Registered tunnel connection` and `Updated to new configuration`. (查找诸如“Registered tunnel connection”和“Updated to new configuration”之类的日志。)

## Troubleshooting (故障排除)
- **HTTP 503 Errors (503错误):** If mapped domains return a 503 error but local access is fine (HTTP 200), advise the user to check their Cloudflare Zero Trust dashboard: (如果映射的域名返回 503 错误但本地访问正常，请建议用户检查其 Cloudflare Zero Trust 仪表板：)
  - Go to the Public Hostname configuration -> **Additional application settings** -> **TLS**. (转到 Public Hostname 配置 -> Additional application settings -> TLS。)
  - Enable **No TLS Verify**. (启用 **No TLS Verify**。这经常发生，因为 OpenWRT 本地 Web 界面通常没有受信任的 SSL 证书。)
