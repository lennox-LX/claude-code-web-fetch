# Claude Code 通用网页抓取增强方案

> 一行命令解决 Claude Code 内置 WebFetch/WebSearch 被网站拦截的问题。
> 适用于使用 DeepSeek、通义千问、Kimi 等第三方 API 的 Claude Code 用户。

---

## 📌 能解决什么问题

Claude Code 内置的 `WebFetch` 和 `WebSearch` 在以下场景经常失效：

| 问题 | 表现 | 原因 |
|------|------|------|
| **网站拦截** | 返回 403 / Cloudflare 验证页面 | B站、CSDN、知乎等网站的 WAF 拦截 AI 请求 |
| **第三方 API 不兼容** | WebSearch 直接报错 | DeepSeek/通义千问等 API 不支持原生的 web_search 工具 |
| **中文内容搜索不到** | 搜索结果全是英文 | 内置搜索引擎对中文技术社区覆盖不足 |
| **TLS 指纹被识别** | 每次都被判定为机器人 | Go/Python HTTP 库的 TLS 指纹与真实浏览器不同 |

**本方案通过 4 个 MCP 服务分层协作，将网页抓取成功率提升到接近 100%。**

---

## 🏗️ 架构

```
┌──────────────────────────────────────────────────┐
│                    你的提问                        │
└──────────────────────┬───────────────────────────┘
                       │
         ┌─────────────▼─────────────┐
         │   cc-web-mcp (搜索层)      │
         │   DuckDuckGo → Bing 搜索   │
         │   DeepSeek 环境自动接管     │
         └─────────────┬─────────────┘
                       │
         ┌─────────────▼─────────────┐
         │   kosyak-fetch (抓取层)    │  ← Cloudflare 自动绕过
         │   通用网页 → Markdown      │
         └─────────────┬─────────────┘
                       │ 失败时
         ┌─────────────▼─────────────┐
         │   koon-mcp (强对抗层)      │  ← TLS 指纹模拟
         │   浏览器指纹绕过反爬        │
         └───────────────────────────┘

B站内容 ──→ bilibili-mcp (专用通道，27个工具)
```

### 各层职责

| 层级 | MCP 服务 | 解决什么 |
|------|---------|---------|
| **搜索层** | cc-web-mcp | DeepSeek 下 WebSearch 失效，中文内容搜索 |
| **抓取层** | kosyak-fetch-mcp | 90% 网站可直接抓取，Cloudflare 自动绕过 |
| **强对抗层** | koon-mcp | 剩余 10% 的顽固站点，TLS 指纹级模拟 |
| **专用通道** | bilibili-mcp | B站结构化数据（视频信息/字幕/评论） |

---

## 🚀 安装指南

### 前置要求

| 依赖 | 最低版本 | 检查命令 |
|------|---------|---------|
| Node.js | ≥ 18 | `node --version` |
| Python | ≥ 3.10 | `python --version` |
| Claude Code | 最新版 | 已安装 VS Code 插件或 CLI |

### 步骤 1：克隆仓库

```bash
git clone https://github.com/YOUR_USERNAME/claude-code-web-fetch.git
cd claude-code-web-fetch
```

### 步骤 2：配置 Claude Code 用户级设置

编辑 `~/.claude/settings.json`（Windows: `C:\Users\你的用户名\.claude\settings.json`），参考以下模板：

```json
{
  "env": {
    "ANTHROPIC_BASE_URL": "https://api.deepseek.com/anthropic",
    "ANTHROPIC_AUTH_TOKEN": "你的DeepSeek_API_Key",
    "ANTHROPIC_MODEL": "deepseek-v4-pro[1m]",
    "ANTHROPIC_DEFAULT_HAIKU_MODEL": "deepseek-v4-flash",
    "ANTHROPIC_DEFAULT_SONNET_MODEL": "deepseek-v4-flash",
    "ANTHROPIC_DEFAULT_OPUS_MODEL": "deepseek-v4-pro[1m]"
  },
  "effortLevel": "high",
  "enableAllProjectMcpServers": true,
  "permissions": {
    "allow": [
      "mcp__cc-web__research_brief",
      "mcp__cc-web__web_search",
      "mcp__cc-web__fetch_url",
      "mcp__kosyak-fetch__*",
      "mcp__bilibili__*"
    ]
  }
}
```

> ⚠️ **安全提示:** 不要将包含真实 API Key 的 `settings.json` 提交到 Git。本仓库的 `.gitignore` 已配置排除。

### 步骤 3：安装 CC-Web-MCP（搜索层）

```bash
# 安装 uv（Python 包管理器）
pip install uv

# 一键初始化（自动配置 settings.json hooks + CLAUDE.md 路由）
uvx cc-web-mcp init --runner uvx
```

### 步骤 4：安装项目级 MCP（抓取层）

项目根目录的 `.mcp.json` 已包含配置，Claude Code 启动时会自动通过 `npx` 下载：

```bash
# 无需手动安装，启动 Claude Code 后自动完成
# 也可提前预热：
npx -y kosyak-fetch-mcp --version
npx -y koon-mcp --version
npx -y bilibili-mcp --version
```

### 步骤 5：重启 Claude Code

关闭并重新打开 Claude Code（或重启 VS Code），配置生效。

### 步骤 6：验证

在 Claude Code 中输入：

```
用 cc-web 搜索"Claude Code MCP 配置"，然后用 fetch 抓取第一个结果的完整内容
```

如果返回完整的网页内容而非 403 错误，说明配置成功。

---

## 📂 文件说明

```
claude-code-web-fetch/
├── .mcp.json              # 项目级 MCP 服务配置（抓取层）
├── .gitignore             # 保护 API Key 和会话数据
├── README.md              # 本文件
├── CREDITS.md             # 开源项目致谢
└── claude-code-settings.example.json  # 用户级配置模板（参考用）
```

---

## 🔧 如果你是其他用户（非本项目作者）

### 场景 A：你用的是 Claude 官方 API

你不需要 cc-web-mcp（搜索层），只需要抓取层。编辑 `.mcp.json`，删除 cc-web 相关配置，保留 kosyak-fetch、koon、bilibili。

然后编辑 `~/.claude/settings.json` 添加权限：

```json
{
  "enableAllProjectMcpServers": true,
  "permissions": {
    "allow": [
      "mcp__kosyak-fetch__*",
      "mcp__bilibili__*"
    ]
  }
}
```

### 场景 B：你用的是另一个第三方 API（通义千问 / Kimi / 智谱）

编辑 `~/.claude/settings.json` 的 `env` 部分，替换为你的 API 地址和 Key。其余步骤不变。

cc-web-mcp 的配置文件中需要更新 `allowed_model_patterns`（位于 `%APPDATA%\cc-web-mcp\config.json`）：

```json
{
  "allowed_model_patterns": ["qwen", "kimi", "zhipu"]
}
```

### 场景 C：你需要登录态抓取（如公司内网、付费网站）

本方案不覆盖登录态场景。推荐额外安装 [Wick](https://github.com/wickproject/wick)：

```bash
npm install -g wick-mcp
wick setup
```

---

## ❓ 常见问题

### Q: 提示 "MCP server not found"

A: 首次使用 MCP 时需要下载 npm 包，可能需要 30-60 秒。等待后重试。

### Q: kosyak-fetch 仍然被某网站拦截

A: koon-mcp 会自动作为回退。如果两者都失败，说明该网站使用了交互式验证码（如 reCAPTCHA），需要人工介入。

### Q: 搜索结果是英文的

A: 编辑 `%APPDATA%\cc-web-mcp\config.json`，确保 `search_providers` 中包含 `"bing_cn"`：

```json
"search_providers": ["duckduckgo", "bing", "bing_cn"]
```

### Q: 如何临时禁用某个 MCP？

A: 编辑 `.mcp.json`，删除对应条目，重启 Claude Code。

---

## 📊 效果对比

| 场景 | 配置前 | 配置后 |
|------|--------|--------|
| 搜索 Python 技术问题 | WebSearch 报错（第三方API） | ✅ cc-web 正常搜索 |
| 抓取 B站视频页面 | 403 / Cloudflare | ✅ bilibili-mcp 结构化数据 |
| 抓取 CSDN 博客 | 403 | ✅ kosyak-fetch 返回 Markdown |
| 抓取 Medium 文章 | 403 | ✅ kosyak-fetch 平台重写 |
| 抓取 Bloomberg 财经 | Cloudflare 拦截 | ✅ koon-mcp TLS 指纹绕过 |

---

## 📄 许可

本配置方案（`.mcp.json`、文档）采用 MIT 许可。

所使用的各 MCP 服务的许可请参见 [CREDITS.md](CREDITS.md) 及各项目仓库。
