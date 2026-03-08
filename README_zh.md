# iStoreOS 上的 Cloudflare Tunnel 自动化配置器

[🇨🇳 简体中文](README_zh.md) | [🇺🇸 English](README.md)

简化和完全自动化在 **iStoreOS** 或 **OpenWRT** 路由器上设置 **Cloudflare Tunnel (cloudflared)** 的过程。此包包含一个 AI 智能体（Agent）技能，可以全程处理零信任部署，**包括自动下载和安装所需的软件包。**

## 自动化执行流程图

```mermaid
graph TD
    A[用户提供凭证和令牌] --> B[Agent 通过 SSH 连接路由器]
    B --> C{检测路由器架构}
    C -->|x86_64, aarch64, etc.| D[下载兼容的安装包]
    D --> E[自动安装插件]
    E --> F[清理旧配置和遗留文件]
    F --> G[将 Token 写入 UCI 数据库]
    G --> H[重启 Cloudflared 服务]
    H --> I[检查日志验证连接状态]
    I --> J[完成！隧道已激活]
```

## 这是什么？
用户经常希望在 OpenWRT/iStoreOS 上部署 `cloudflared` 扩展，但在查找正确的架构包 (`.ipk`)、命令行设置、SSL 验证问题或 UCI 配置方面遇到困难。此技能（专为 Antigravity/Cursor/Devin 等 AI Agent 设计）允许 AI 自动 SSH 连接到路由器，**检测架构，下载最新兼容的 `cloudflared.ipk` 并安装它**，配置 CF Tunnel 令牌，建立远程连接，并排查常见的如 HTTP 503 错误等问题。

## 配置文件详解

该技能会修改 OpenWRT 的核心配置系统 (UCI)。以下是交互的特定文件：

### 1. `/etc/config/cloudflared`
这是由 UCI 管理的 OpenWRT 主配置文件。当 Agent 运行 `uci set ...` 时，它会将用户的口令直接写入此处。

**生成的文件内容示例:**
```text
config cloudflared 'config'
        option enabled '1'              # 由 Agent 自动设置为开机自启
        option token 'eyJhIjoi...'      # 用户提供的 Token
        option config '/etc/cloudflared/config.yml'
        option origincert '/etc/cloudflared/cert.pem'
        option protocol 'http2'
        option loglevel 'info'
        option logfile '/var/log/cloudflared.log'
```

### 2. `/var/log/cloudflared.log`
Agent 会监控此日志文件，以验证隧道连接是否成功到达 Cloudflare 边缘网络（寻找 `Registered tunnel connection`）。

## 如何使用

### 用于 Agent 工作流 (`SKILL.md`)
如果你使用的是 AI Agent（例如 Antigravity），你可以将 `SKILL.md` 文件导入到 Agent 的技能目录中。导入后，你只需对 Agent 说：

> *"把我的 Cloudflare Tunnel 安装在 192.168.1.1 的 iStoreOS 路由器上，我的 Token 是 eyJh..."*

Agent 会读取 `SKILL.md` 中的说明，自动下载包、安装并执行全部配置步骤。

### 手动设置
如果您喜欢通过 SSH 在您的 iStoreOS 路由器上手动配置此设置：

1. 通过 SSH 连接到您的路由器: `ssh root@192.168.x.x`
2. 下载并安装 `cloudflared` `.ipk`:
   ```bash
   cd /tmp
   # 示例：下载适合 x86_64 架构的安装包
   wget https://your-repo/cloudflared_xxx.ipk
   opkg install cloudflared_xxx.ipk
   ```
3. 运行以下命令，将 `YOUR_TOKEN_HERE` 替换为实际的 CF Token：

```bash
uci set cloudflared.config.token='YOUR_TOKEN_HERE'
uci set cloudflared.config.enabled='1'
uci commit cloudflared
/etc/init.d/cloudflared enable
/etc/init.d/cloudflared restart
```

4. 检查隧道是否成功运行:
```bash
tail -n 20 /var/log/cloudflared.log
```

## 解决 503 错误
如果隧道在 Cloudflare 仪表板中显示为已连接，但您映射的本地网域（如 `router.yourdomain.com`）返回 **HTTP 503 Service Unavailable** 错误：

1. 转到 Cloudflare Zero Trust Dashboard -> Networks -> Tunnels。
2. 编辑您的隧道，进入“**Public Hostname**”标签页。
3. 编辑抛出 503 的主机名映射。
4. 展开 **Additional application settings** -> **TLS**。
5. 启用 **No TLS Verify**。
6. 保存并等待 1-2 分钟。这会绕过 Cloudflare 对路由器默认自签名证书的严格 SSL 检查。

## 许可证 (License)
MIT License
