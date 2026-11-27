# Shadowrocket AI Rules

一份面向 AI 服务访问的 Shadowrocket 分流配置，基于 [LingJingMaster/Shadowrocket-Rules](https://github.com/LingJingMaster/Shadowrocket-Rules) 维护。

- 国内服务直连，境外流量默认代理。
- ChatGPT、Claude / Claude Code、Gemini 核心请求优先匹配代理规则。
- 手动选择代理节点，不自动测速或切换出口。
- 使用加密 DNS，保留 HTTPDNS、广告及代理侧 QUIC 拦截。
- 保留香港银行、券商、开发工具和 Apple 等专项分流，不使用 MITM 解密。

## 快速开始

配置文件：[Shadowrocket-AI-Rules.conf](https://raw.githubusercontent.com/wangzy2u/Shadowrocket-AI-Rules/main/Shadowrocket-AI-Rules.conf)

```text
https://raw.githubusercontent.com/wangzy2u/Shadowrocket-AI-Rules/main/Shadowrocket-AI-Rules.conf
```

1. 复制上方链接，打开 Shadowrocket → **配置** → 右上角 **+** → 粘贴链接并下载。
2. 点击已下载的配置，将其设为当前使用的配置。
3. 在首页添加自己的节点或订阅，手动选择要使用的代理节点。
4. 将全局路由设为 **配置**，开启连接。
5. 确认远程规则集下载成功，再检查常用服务的连接记录。

或者扫描二维码获取配置：

<img src="assets/config-qrcode.png" width="200" height="200" alt="Shadowrocket AI Rules 配置下载二维码" />

仓库仅提供配置和规则，节点凭据由使用者保存在客户端。

## 分流规则

| 服务 | 默认策略 |
| --- | --- |
| 🤖 ChatGPT、Claude / Claude Code、Gemini / Gemini API | PROXY |
| 🐙 GitHub Copilot 接口与实时连接 | PROXY |
| 🧠 Apple Intelligence | PROXY |
| 🇨🇳 DeepSeek、豆包及国内服务 | DIRECT |
| 🔍 Google、YouTube | PROXY |
| 🐱 GitHub、GitLab、Atlassian | PROXY |
| 📲 Telegram | PROXY |
| Ⓜ️ 微软、邮件服务 | PROXY |
| 🏦 汇丰香港、其他香港银行 | DIRECT |
| 📈 香港及海外券商 | PROXY |
| 🍏 普通 Apple 服务、苹果推送 | PROXY |
| 🍎 `apple.com.cn`、`icloud.com.cn`、`icloud-content.com.cn` | DIRECT |
| 🏠 局域网名称及私网地址 | DIRECT |
| 🧱 HTTPDNS、广告及追踪域名 | REJECT |
| 🌍 通用代理域名 | PROXY |
| 🇨🇳 国内直连域名白名单、中国大陆 IP | DIRECT |
| 🐟 其他未匹配流量 | PROXY |

AI 核心规则先于广告、普通服务分类和国内直连规则；通用代理域名先于国内直连白名单。香港银行属于明确的境外直连例外。

## 默认策略

| 服务 | 默认策略 | 可选策略 |
| --- | --- | --- |
| AI 核心、GitHub Copilot 接口、Apple Intelligence、境外兜底 | PROXY | 固定规则 |
| Google、YouTube、开发工具、微软、邮件、Telegram、券商 | PROXY | PROXY、REJECT |
| 普通 Apple 服务、苹果推送 | PROXY | PROXY、REJECT |
| 汇丰香港、其他香港银行 | DIRECT | DIRECT、PROXY、REJECT |

`PROXY` 使用首页手动选中的节点。AI 核心直接指定 `PROXY`，代理故障时预期连接失败，重试继续匹配代理规则。

Gemini 的共享认证端点也优先代理，不受普通 Google 策略组影响。Claude Code 的第三方 MCP、Bedrock、Vertex 及自定义网关按实际目标域名分流。

WorkOS、Cloudflare 验证及 Apple 共享依赖采用精确主机规则，其他应用访问同一主机也会走代理。GitHub Copilot 的共享 GitHub 登录仍受开发工具策略控制；`npmjs.com`、`npmjs.org` 随 `GitHub.list` 归入开发工具策略（默认代理），通用云存储按各自规则处理。

## DNS 与隐私

| 项目 | 配置 |
| --- | --- |
| 主、备用 DNS | Cloudflare DoH：`1.1.1.1`、`1.0.0.1`，经代理查询 |
| 直连域名解析 | 阿里 DoH：`223.5.5.5`、`223.6.6.6` |
| 代理节点域名解析 | 阿里 DoH，用于建立代理连接前的引导解析 |
| 普通 DNS 接管 | 接管进入隧道的 53 端口查询 |
| IPv6 | 关闭，并拒绝进入规则引擎的 IPv6 连接 |
| UDP 回退 | 代理不支持 UDP 时拒绝，不回退直连 |
| QUIC | 拦截代理业务侧 QUIC |
| WebRTC | 拦截 UDP `3478`、`5349`、`19302–19309` |
| HTTPS | 不使用 MITM、证书解密或脚本重写 |

WebRTC 端口限制可能影响语音和视频通话，不能覆盖所有 STUN / TURN 通信方式。配置不使用伪造 STUN IP，也不对全部国内 UDP 流量进行封禁。

本配置在 Shadowrocket 隧道运行且全局路由为“配置”时生效，不提供隧道退出后的系统级断网保护。

## 规则集来源

| 来源 | 用途 |
| --- | --- |
| [本项目专项规则](https://github.com/wangzy2u/Shadowrocket-AI-Rules) | AI、Google、Apple、推送、邮件、银行及券商规则 |
| [LingJingMaster/Shadowrocket-Rules](https://github.com/LingJingMaster/Shadowrocket-Rules) | 专项规则的上游来源与基础配置参考 |
| [Loyalsoldier/surge-rules](https://github.com/Loyalsoldier/surge-rules) | 国内直连域名、通用代理域名、中国大陆 IP 和本地名称 |
| [blackmatrix7/ios_rule_script](https://github.com/blackmatrix7/ios_rule_script) | HTTPDNS、YouTube、Telegram、开发工具、微软和 Apple 等专项规则 |
| [ACL4SSR/ACL4SSR](https://github.com/ACL4SSR/ACL4SSR) | `BanAD.list`、`BanProgramAD.list` 广告过滤 |
| [Johnshall/Shadowrocket-ADBlock-Rules-Forever](https://github.com/Johnshall/Shadowrocket-ADBlock-Rules-Forever) | 本地广告条目及 GEOIP 兜底方式参考 |

国内基础使用 Loyalsoldier 的 `direct.txt`、`proxy.txt`、`cncidr.txt` 和 `private.txt`。`GEOIP,CN` 兜底使用客户端的 GeoIP 数据库。

根目录的八个 `.list` 文件由本项目维护，[Shadowrocket-AI-Rules.conf](Shadowrocket-AI-Rules.conf) 通过本仓库 `main` 分支的 Raw 地址加载。Johnshall 仅作为参考，不整份导入其配置。

[AI.list](AI.list) 与主配置的核心段共同维护，依据 [OpenAI 网络要求](https://help.openai.com/en/articles/9247338-network-recommendations-for-chatgpt-errors-on-web-and-apps)、[Claude Code 网络要求](https://code.claude.com/docs/en/network-config)、[Claude Desktop 网络要求](https://code.claude.com/docs/en/desktop#network-access-requirements)、[Gemini API 文档](https://ai.google.dev/gemini-api/docs/quickstart)及 [GitHub Copilot 网络清单](https://docs.github.com/en/copilot/reference/copilot-allowlist-reference)核查。规则采用明确后缀和精确主机，不使用 AI 品牌关键字或未经核实的历史 IP；已保留的 Sentry、Datadog 兼容条目不扩展为整域放行。

## 更新方式

- **专项规则**：通过 Shadowrocket 获取本仓库的 `.list` 更新；上游变更经审核后同步至本项目。
- **公共规则**：国内基础、广告及其他公共规则继续从 Loyalsoldier、ACL4SSR、blackmatrix7 获取更新。
- **主配置**：从本仓库重新下载 `Shadowrocket-AI-Rules.conf` 并启用；远程规则集更新不会覆盖主配置。
- **版本回退**：更新前保存客户端当前配置和策略选择，出现异常时重新导入已验证的版本。

## 验证状态与反馈

截至 2026-11-28，已完成配置结构、策略引用、规则优先级及远程规则读取的静态检查。Shadowrocket 导入、代理故障重试，以及 DNS、WebRTC、IPv6 和 UDP 的实际行为仍待实机验证。

故障测试应保持隧道开启，触发应用重试并检查连接记录，确认 AI 请求没有转为直连。

遇到问题可提交 [Issue](https://github.com/wangzy2u/Shadowrocket-AI-Rules/issues)，提供客户端版本、目标域名、匹配策略及复现步骤。提交日志前请移除节点密钥、订阅地址和其他敏感信息。项目维护规范见 [AGENTS.md](AGENTS.md)。

## 许可证

采用 [MIT License](LICENSE)，保留上游署名。远程规则遵循各自来源项目的许可证。
