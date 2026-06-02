# 开源项目致谢

本配置方案使用了以下开源项目，感谢所有作者的贡献。

---

## 核心 MCP 服务

### cc-web-mcp
- **仓库:** [JcDizzy/CC-Web-MCP](https://github.com/JcDizzy/CC-Web-MCP)
- **作者:** [JcDizzy](https://github.com/JcDizzy)
- **许可:** MIT
- **作用:** 面向 DeepSeek/Qwen/Kimi 等第三方模型后端的 WebSearch/WebFetch 回退 MCP

### kosyak-fetch-mcp
- **npm:** [kosyak-fetch-mcp](https://www.npmjs.com/package/kosyak-fetch-mcp)
- **作用:** 通用网页抓取，自动绕过 Cloudflare/Akamai 反爬，支持 Reddit/Medium/YouTube 等平台重写

### koon-mcp
- **npm:** [koon-mcp](https://www.npmjs.com/package/koon-mcp)
- **作用:** 基于 Rust + BoringSSL 的浏览器 TLS 指纹模拟，绕过强反爬检测

### bilibili-mcp
- **仓库:** [adoresever/bilibili-mcp](https://github.com/adoresever/bilibili-mcp)
- **作者:** [adoresever](https://github.com/adoresever)
- **作用:** B站专用 MCP，提供视频搜索、评论、字幕、弹幕等 27 个工具

---

## 相关参考

| 项目 | 仓库 | 说明 |
|------|------|------|
| cloakFetch | [Agents365-ai/cloakFetch](https://github.com/Agents365-ai/cloakFetch) | WebFetch Hook 回退方案（本方案未采用但设计上参考了其思路） |
| Claude-Local-Web-Capture | [danielrosehill/Claude-Local-Web-Capture-Plugin](https://github.com/danielrosehill/Claude-Local-Web-Capture-Plugin) | 四级抓取升级阶梯（本方案的分层架构参考） |
| web-access | [eze-is/web-access](https://github.com/eze-is/web-access) | 三层通道 + Chrome CDP 方案 |

---

## 致谢

本配置方案站在以上开源项目的肩膀上组合而成。如果你觉得本方案有用，请给上述项目点 Star——它们才是真正解决问题的代码。
