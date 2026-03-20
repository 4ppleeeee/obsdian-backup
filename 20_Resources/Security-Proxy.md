# OpenClaw 跨机器反代安全方案 (2026-03-11)

## 场景描述
用户 OpenClaw Gateway 部署在 A 机器（192.168.31.82:22025），反向代理服务器（如 Nginx）部署在 B 机器。用户希望在外网通过 HTTPS 安全访问，同时在内网保留直连的便利性。

## 方案设计：分级白名单 + 自动授权

### 1. 基础设施层：IP 网段过滤
不要只允许反代服务器 IP，而是允许整个家庭局域网网段访问 Gateway 端口。
- **操作**：`sudo ufw allow from 192.168.31.0/24 to any port 22025`
- **效果**：内网所有设备可直连，外网必须通过反代。

### 2. 应用层：受信任代理配置
在 `openclaw.json` 中明确反代服务器的内网 IP。
- **配置**：`"trustedProxies": ["反代服务器IP"]`
- **效果**：消除日志中的 "Proxy headers detected" 警告，确保 OpenClaw 能穿透代理识别真实的客户端真实 IP。

### 3. 安全性与便利性平衡：Control UI 优化
- **开启 `allowInsecureAuth`**: 允许内网 HTTP 环境下的登录。
- **禁用 `dangerouslyDisableDeviceAuth`**: 恢复设备授权机制。
- **效果**：新设备首次登录（无论内网外网）必须通过 Agent 绑定的频道（如飞书）进行二次确认，有效防止密码泄露风险。

### 4. 社交频道安全加固 (Feishu specific)
针对具备系统级权限 (exec, fs) 的 Agent，必须收紧准入策略。
- **移除通配符**: 将 `allowFrom` 从 `["*", "ou_xxx"]` 改为 `["ou_xxx"]`，拒绝非 Owner 的私聊。
- **群组白名单**: 将 `groupPolicy` 改为 `allowlist`，并配合 `groupAllowlist`。
- **效果**: 防止 Agent 在被拉入陌生群组后，被他人通过 Prompt Injection 恶意调用服务器资源。
