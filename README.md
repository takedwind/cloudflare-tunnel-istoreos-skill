# Cloudflare Tunnel on iStoreOS Configurator (iStoreOS 上的 Cloudflare Tunnel 配置器)

Simplify and automate the process of setting up **Cloudflare Tunnel (cloudflared)** on iStoreOS or OpenWRT routers. This package contains an AI agent skill that can handle zero-trust deployment from start to finish.

简化和自动化在 **iStoreOS** 或 **OpenWRT** 路由器上设置 **Cloudflare Tunnel (cloudflared)** 的过程。此包包含一个 AI 智能体（Agent）技能，可以全程处理零信任部署。

## What is this? (这是什么？)
Often, users deploy the `cloudflared` extension on OpenWRT/iStoreOS but struggle with the command-line setup, SSL validation issues, or UCI configuration. This skill (designed for AI agents like Antigravity/Cursor/Devin) allows the AI to automatically SSH into the router, configure the CF Tunnel token, establish the remote connection, and troubleshoot common issues like HTTP 503 errors.

用户经常在 OpenWRT/iStoreOS 上部署 `cloudflared` 扩展，但在命令行设置、SSL 验证问题或 UCI 配置方面遇到困难。此技能（专为 Antigravity/Cursor/Devin 等 AI Agent 设计）允许 AI 自动 SSH 连接到路由器，配置 CF Tunnel 令牌，建立远程连接，并排查常见的如 HTTP 503 错误等问题。

## Features (特点)
- **Automated Configuration (自动化配置)**: Applies your Cloudflare Tunnel token directly to the router's UCI system. (将 Cloudflare Tunnel 令牌直接应用到路由器的 UCI 系统。)
- **Service Management (服务管理)**: Automatically enables and restarts the `cloudflared` service. (自动启用并重启 `cloudflared` 服务。)
- **Clean up (清理)**: Can purge broken or old configurations before applying a new token. (在应用新令牌之前清除损坏或旧的配置。)
- **Troubleshooting (故障排除)**: Built-in instructions to help diagnose 503 errors caused by OpenWRT's local self-signed certificates (No TLS Verify). (内置指令可帮助诊断由 OpenWRT 本地自签名证书引起的 503 错误。)

## How to use (如何使用)

### For Agent Workflows (用于 Agent 工作流 - SKILL.md)
If you are using an AI agent (e.g., Antigravity), you can import the `SKILL.md` file into your agent's skill directory. Once imported, you can simply ask your agent:

如果你使用的是 AI Agent（例如 Antigravity），你可以将 `SKILL.md` 文件导入到 Agent 的技能目录中。导入后，你只需对 Agent 说：

> *"Install my cloudflare tunnel on my iStoreOS router at 192.168.1.1. My token is eyJh..."*
> *("把我的 Cloudflare Tunnel 安装在 192.168.1.1 的 iStoreOS 路由器上，我的 Token 是 eyJh...")*

The agent will read the `SKILL.md` instructions and execute the steps automatically. (Agent 会读取 `SKILL.md` 中的说明并自动执行这些步骤。)

### Manual Setup (手动设置)
If you prefer to configure this manually on your iStoreOS router via SSH:
如果您喜欢通过 SSH 在您的 iStoreOS 路由器上手动配置此设置：

1. SSH into your router (通过 SSH 连接到您的路由器): `ssh root@192.168.x.x`
2. Run the following commands, replacing `YOUR_TOKEN_HERE` with your actual Cloudflare Tunnel token (运行以下命令，将 `YOUR_TOKEN_HERE` 替换为实际的 CF Token):

```bash
uci set cloudflared.config.token='YOUR_TOKEN_HERE'
uci set cloudflared.config.enabled='1'
uci commit cloudflared
/etc/init.d/cloudflared restart
```

3. Check if the tunnel is running successfully (检查隧道是否成功运行):
```bash
tail -n 20 /var/log/cloudflared.log
```

## Troubleshooting 503 Errors (解决 503 错误)
If your tunnel shows as connected in the Cloudflare Dashboard, but your mapped local domains (e.g., `router.yourdomain.com`) return an **HTTP 503 Service Unavailable** error:

如果隧道在 Cloudflare 仪表板中显示为已连接，但您映射的本地网域（如 `router.yourdomain.com`）返回 HTTP 503 服务不可用错误：

1. Go to your Cloudflare Zero Trust Dashboard -> Networks -> Tunnels. (转到 Cloudflare Zero Trust Dashboard -> Networks -> Tunnels。)
2. Edit your tunnel and go to the **Public Hostname** tab. (编辑您的隧道，进入“Public Hostname”标签页。)
3. Edit the hostname mapping that is throwing the 503. (编辑抛出 503 的主机名映射。)
4. Expand **Additional application settings** -> **TLS**. (展开 **Additional application settings** -> **TLS**。)
5. Enable **No TLS Verify**. (启用 **No TLS Verify**。)
6. Save and wait 1-2 minutes. This bypasses Cloudflare's strict SSL check against your router's default self-signed certificate. (保存并等待 1-2 分钟。这会绕过 Cloudflare 对路由器默认自签名证书的严格 SSL 检查。)

## License (许可证)
MIT License
