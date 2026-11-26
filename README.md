# Shadowrocket AI Rules

本版本基于 LingJingMaster 工程，适用于只有一台 VPS 的使用方式。国内服务直连，境外默认代理，AI 核心直接指定 `PROXY`。代理不可用时允许连接失败，重试仍须命中代理规则。

**唯一导入配置：[Personal.conf](Personal.conf)**。维护约束见 [AGENTS.md](AGENTS.md)。

个人维护仓库：[wangzy2u/Shadowrocket-AI-Rules](https://github.com/wangzy2u/Shadowrocket-AI-Rules)。仓库只保存规则与文档，节点凭据保存在客户端。本地备份、临时文件及私密材料不上传；参考上游更新时不恢复上游主配置或旧 Git 历史。

原有八个规则文件及许可证继续保留为上游参考，本文件集中说明个人配置的使用和维护。

个人配置没有 `include` 旧配置，远程规则只引用各自的规则文件。旧上游主配置已从根目录移除，本地备份不参与规则加载或发布。

## 导入与使用

1. 在 Shadowrocket 首页添加自己的 VPS，并手动选中它。节点密码、私钥、UUID 和订阅地址只保存在客户端。
2. 在 Shadowrocket 的“配置”中添加下方 Raw 地址，下载并设为当前配置；也可通过“文件”或 iCloud 导入本地 `Personal.conf`。先备份原来正在使用的配置。
3. 全局路由设为“配置”，重新连接隧道。不能使用“直连”模式验收本配置。
4. 检查远程规则集是否下载成功，再执行下方验收。

个人配置的 Raw 地址：

```text
https://raw.githubusercontent.com/wangzy2u/Shadowrocket-AI-Rules/main/Personal.conf
```

本文件没有 `update-url`，更新远程规则集不会用上游主配置覆盖个人设置。主配置从个人仓库或本地文件手动更新，不使用 LingJingMaster 原主配置的下载地址。

仓库或本地文件的变更不会自动修改客户端已经导入的副本，需要在客户端确认当前启用的是个人版，并在主配置更新后重新下载或导入。仓库内八个 `.list` 保留为上游参考，当前配置仍按各条远程引用读取对应上游，不会自动切换为仓库内副本。

## 代理边界

| 情况 | 配置的预期行为 |
| --- | --- |
| VPS 可用，隧道运行，全局路由为“配置” | AI 核心通过首页选择的 VPS |
| VPS 不可达，隧道仍运行 | AI 连接失败，应用重试仍匹配 `PROXY` |
| 国内服务已匹配明确直连域名 | 使用国内直连解析和连接路径 |
| VPS 不支持 UDP | 拒绝相关 UDP，不回退直连 |
| 隧道关闭、退出或被系统终止 | 本配置不能单独保证系统级断网保护，需另行验证客户端及系统能力 |

核心保护依赖域名和受控隧道。应用改用尚未收录的网关、硬编码 IP、另一台设备或远程容器时，要重新检查实际连接路径。不能仅凭页面打不开判断没有发起过直连。

## 服务策略

| 服务 | 默认处理 | 手动控制范围 |
| --- | --- | --- |
| ChatGPT、Claude、Claude Code、Gemini 核心及必要关联端点 | 本地规则直接 `PROXY` | 不设置 AI 出口组，不提供 DIRECT 选项 |
| 上游 AI 补充规则 | `PROXY` | 不经过普通服务组 |
| Google、YouTube、开发工具、微软、邮件、Telegram、券商 | `PROXY` | 各服务组只能选择 PROXY 或 REJECT |
| 汇丰香港、其他香港银行 | DIRECT | 可选 DIRECT、PROXY、REJECT；这是明确的境外直连例外 |
| Apple Intelligence | 本地核心规则直接 `PROXY` | 不受普通 Apple 组影响 |
| 普通 Apple 服务、苹果推送 | `PROXY` | 可选 PROXY 或 REJECT |
| `apple.com.cn`、`icloud.com.cn` | DIRECT | 国内名称例外，仍需验证实际业务依赖 |
| DeepSeek、豆包及常用国内服务 | DIRECT | 显式域名、Loyalsoldier 直连域名和中国大陆 IP 列表 |
| 国内 GEOIP | DIRECT | 位于专项规则之后 |
| 其他流量 | `FINAL,PROXY` | 使用首页唯一 VPS |

普通 Apple 服务由上游的全域直连改为默认代理，国内 Apple 名称明确直连。如果 iCloud 同步或推送体验不符合预期，先检查实际端点，再审核精确例外，不恢复整个 Apple 公网域名的旁路。

Google 的普通服务组设为 REJECT 后，Gemini API、网页及共享认证仍命中前面的本地 PROXY 规则。`accounts.google.com` 和 `oauth2.googleapis.com` 是共享认证域名，这也会影响其他 Google 登录连接。

汇丰规则中的 `mobile.eum-appdynamics.com` 为共享监控端点，域名分流无法按调用它的 App 区分，其他应用对该主机的访问也会使用汇丰策略。渣打的 `www.sc.com` 使用整主机策略，其其他地区页面也受影响；不依赖解密 HTTPS 后才能判断的 `/hk/` 路径。

Claude Code 的 API、认证、下载及内容桥接由相应域名覆盖；第三方 MCP、WebFetch 目标、Bedrock、Vertex 或自定义网关仍按其实际域名分流。要在运行 Claude Code 的实际设备验收，手机隧道不会自动接管另一台电脑或远程服务器。

## 规则优先级

1. localhost、局域网名称等必要本地访问。
2. 进入规则引擎的 IPv6、常见 STUN/TURN UDP 端口和 HTTPDNS 拦截。
3. 本地 AI 核心及必要关联端点，直接指定 PROXY。
4. 汇丰等必要业务的精确广告例外。
5. Johnshall 参考条目及 ACL4SSR 广告规则。
6. 国内 AI 等显式直连。
7. 可信上游 AI、邮件、视频、Google、开发工具、银行及 Apple 等分类，以及本地名称、私网地址。
8. Loyalsoldier 代理域名、直连域名和中国大陆 IP 分类，GEOIP CN 和 PROXY 兜底。

本地核心规则先于远程服务分类，避免关键域名因上游分类改变而进入国内直连。新核心端点应加在同一段，并检查前面不存在覆盖它的直连或旁路。

与原配置相比，移除了地区自动测速、海外 S3 附件直连、整个 `208.54.0.0/16` 强制美国出口、所有 TCP 5223 一概归为苹果推送、微信 HTTPDNS 前置放行和公网 Apple 系统 DNS 例外。不加载原 Mini 的整份 UnBan 或 ChinaCompanyIp：广告误杀例外及企业归属不等于地理上的国内直连。

2026-11-27 国内基础来源调整：撤下 blackmatrix7 的 China、China_Domain、Global、Global_Domain、BiliBili 和 Lan 六个远程引用。国内媒体由新的直连域名列表覆盖，本地名称使用同源 `private.txt` 及显式规则，私网 IPv4 另有本地 CIDR；不将 Fake IP 常用的 `198.18.0.0/15` 放入私网直连。

## DNS 与协议处理

| 用途 | 配置 |
| --- | --- |
| 主、备用 DNS | Cloudflare `1.1.1.1`、`1.0.0.1` DoH，均带 `#proxy` |
| 已匹配直连域名 | 阿里 `223.5.5.5`、`223.6.6.6` DoH |
| 节点域名引导解析 | 同一组阿里 DoH，仅用于建立代理前解析节点服务器名称 |
| 系统 DNS | 不作为通用查询或备用；仅对 `.local`、反向解析等本地名称保留 Host 例外 |
| 普通硬编码 DNS | `hijack-dns = :53` 接管进入隧道的 53 端口查询；不等同于拦截任意 DoH |

使用 IP 形式的 DoH 端点可以减少解析器主机名自身的引导查询。解析器 IP 不是伪造 STUN 地址。国内服务和 VPS 域名的国内解析是明确设计，不代表 AI 域名可以回退到本地解析。

未采用 `dns-fallback-system`：本机内置默认配置未列出此项，在应用可执行组件中也未检索到同名字符串，暂时无法确认它在目标版本被识别。这不是“不支持”的充分证明，但不将未确认的开关作为安全保障；主、备用 DNS 已明确设置为代理 DoH，且关闭直连域名使用系统 DNS。

`dns-direct-fallback-proxy=false` 控制直连解析失败后的代理处理，与 UDP 回退是两回事。目标版本的参数识别、DNS 优先级和失败路径仍须实测。

`bypass-system=false` 收窄系统旁路；`skip-proxy` 和 `tun-excluded-routes` 只保留必要本地、保留地址和网络兼容项。它们处于不同处理层级，隧道排除路由不会被后面的规则重新接管。`private-ip-answer=true` 为局域网兼容保留，应配合加密解析及异常私网应答检查。

IPv6 首版关闭，并添加 `IP-CIDR,::/0,REJECT,no-resolve` 拒绝进入规则引擎的 IPv6 目标。这不能证明未进入隧道的 IPv6 也被阻断；字面量 IPv6、IPv6 Only 网络及系统路由必须实测。

WebRTC 拦截 UDP 3478、5349、19302–19309，不按 CN 目标整体放行，不使用虚假 STUN 应答。这会影响部分国内语音、视频、实时通信；如需例外，要明确记录放开范围及对真实 IP 候选的影响。其他端口、TCP TURN 等方式不在这份端口表的完整覆盖范围内，浏览器还应验收非代理 UDP 行为。

`block-quic=all-proxy` 保留代理业务侧拦截；国内直连 UDP 没有全部禁用。VPS 若使用 Hysteria2、TUIC 等传输，需要分别验证节点连接和应用业务，不能把二者的 QUIC 行为混为一谈。

配置不包含 MITM、HTTPS 解密、URL 重写、脚本或伪造 STUN 参数。上游规则中的 USER-AGENT、HTTPS URL 路径条目不能在不解密时保证生效，不以它们作为 AI 保护依据。

## 广告与来源维护

- 基础广告源延续原 Mini 的 ACL4SSR `BanAD.list` 和 `BanProgramAD.list`，放在国内及普通服务放行之前。
- 参考 Johnshall 的 `sr_ad_only.conf`，在主配置里补充 Google Analytics、DoubleClick、字节广告域名的少量本地兜底。这些条目可能与远程广告源重叠，属于相同 REJECT 策略的有意兜底。
- 不引入整份 Johnshall 配置，不新增完整广告快照或自动生成器。广告过滤不保证消除同域或视频流内广告。
- 国内基础与通用代理分类改用 Loyalsoldier/surge-rules 的发布列表，具体来源见下节。银行、券商、Google、Apple 和 AI 补充继续引用 LingJingMaster 原始规则。
- HTTPDNS、YouTube、Telegram、开发工具、微软等部分专项仍引用 blackmatrix7，本次没有将全部专项来源迁移；不要将国内基础来源变更描述为整个工程已移除该项目。
- 部分 AI 依赖是共享基础设施，例如 `livekit.cloud`、Statsig 和 Datadog 精确端点。必要端点提前代理会放行其对应主机的其他调用；后续根据实际日志收窄，不代理整个云厂商根域名。

可信分支的远程更新能力保留，但分支内容会变化。新增来源或刷新后，检查下载结果、语法、广告误杀和关键服务；本地核心规则不能只存在于远程规则里。

### 国内基础数据的实际来源

| 配置引用 | 作用 | 生成链路 |
| --- | --- | --- |
| Loyalsoldier/surge-rules `proxy.txt` | 通用代理域名，先于直连名单 | `Loyalsoldier/v2ray-rules-dat` 的代理分类转换为 DOMAIN-SET |
| Loyalsoldier/surge-rules `direct.txt` | 国内及审核为可直连的域名白名单 | `Loyalsoldier/v2ray-rules-dat` 的直连分类转换为 DOMAIN-SET |
| Loyalsoldier/surge-rules `cncidr.txt` | 中国大陆 IPv4/IPv6 网络段 | `Loyalsoldier/geoip` 的 `text/cn.txt`；其中国大陆数据源为 `gaoyifan/china-operator-ip` |
| Loyalsoldier/surge-rules `private.txt` | 本地名称和路由器名称 | `Loyalsoldier/domain-list-custom` 的本地分类 |
| `GEOIP,CN,DIRECT` | 未命中以上规则的 CN 地址兜底 | 客户端当前选用的 GeoIP 数据库，需在客户端单独更新与验收 |

生成链路按 [实际工作流](https://github.com/Loyalsoldier/surge-rules/blob/master/.github/workflows/run.yml) 和 [GeoIP 数据来源说明](https://github.com/Loyalsoldier/geoip) 核对。工作流计划在台北时间每日 06:30 构建，当前工程仅订阅发布产物，不增加本地构建或格式转换系统。计划构建不等于每次都成功，也不证明所有底层数据同时更新，维护时需检查发布版本和内容。

`direct.txt` 是域名直连白名单，不是“每个域名的服务器都在中国大陆”的证明，其中也包含部分可直连的境外服务。AI 核心、广告和各专项策略先于它，通用代理名单也先于它；冲突时按个人配置优先级处理，不把整个云服务商或中国企业的全球网段当成国内。

Johnshall 的 `lazy.conf` 仍引用 blackmatrix7；其 `sr_cnip.conf` 的国内识别主要是 `GEOIP,CN,DIRECT`，没有可直接替代的独立国内域名/IP列表。因此本次按已确认的选择保留域名直连能力、使用持续构建的数据源，仅参考 Johnshall 的 GEOIP 兜底方式，不导入它的整份配置、MITM 或 Apple 旁路。

本次核对到的发布分支提交为 [93b4a873a62f](https://github.com/Loyalsoldier/surge-rules/commit/93b4a873a62f50ee6d51001bbb96e9ff21781e08)，提交时间为台北时间 2026-11-26 06:32:05。以下为检查时的数据，发布分支后续更新后数量和摘要会变化：

| 列表 | 有效条目 | SHA-256 前 12 位 |
| --- | ---: | --- |
| `direct.txt` | 110250 | `6fb9704786f2` |
| `proxy.txt` | 27288 | `49a49cde0b44` |
| `cncidr.txt` | 9665（IPv4 6221、IPv6 3444） | `c38344e4bb36` |
| `private.txt` | 132 | `44e02418cff1` |

四份新列表内部均未发现完全重复的有效条目。`private.txt` 包含 `.example`、`.test` 等保留后缀；检查境外兜底时应使用普通、未收录的域名，不能把这些本地保留后缀当作境外域名测试。

来源与核对入口（检查日期：2026-11-27）：

- [LingJingMaster 上游](https://github.com/LingJingMaster/Shadowrocket-Rules)：原配置及服务分类。
- [Loyalsoldier Surge 规则](https://github.com/Loyalsoldier/surge-rules)：直连、代理、本地域名及中国大陆 IP 发布列表。
- [Loyalsoldier GeoIP](https://github.com/Loyalsoldier/geoip)：中国大陆 IP 列表的生成来源。
- [blackmatrix7 Shadowrocket 规则](https://github.com/blackmatrix7/ios_rule_script/tree/master/rule/Shadowrocket)：暂时保留的 HTTPDNS 和部分代理侧专项，不再作为国内基础来源。
- [ACL4SSR 广告规则](https://github.com/ACL4SSR/ACL4SSR/tree/master/Clash)：沿用的基础广告源。
- [Johnshall 仅广告配置](https://raw.githubusercontent.com/Johnshall/Shadowrocket-ADBlock-Rules-Forever/release/sr_ad_only.conf)：本地广告兜底的参考来源。
- [Johnshall 国内外划分模板](https://raw.githubusercontent.com/Johnshall/Shadowrocket-ADBlock-Rules-Forever/build/factory/template/sr_cnip.txt)：GEOIP CN 与代理兜底方式参考。
- [LOWERTOP 配置参数示例](https://raw.githubusercontent.com/LOWERTOP/Shadowrocket/main/lazy_group.conf)：DNS、手动策略和 UDP 等参数的社区维护用例，不等同于完整官方规格。
- [53 端口 DNS 劫持用例](https://github.com/haritos90/shadowrocket-config-files)：`hijack-dns = :53` 的维护者配置用例，需在目标客户端确认。
- [Claude Code 官方网络要求](https://code.claude.com/docs/en/network-config)：API、认证、下载及关联主机。
- [Cloudflare DoH 文档](https://developers.cloudflare.com/1.1.1.1/encryption/dns-over-https/make-api-requests/)：解析接口说明。
- [Shadowrocket 官方版本记录](https://apps.apple.com/us/app/shadowrocket/id932747118)：平台和网络行为修复记录。

## 检查与实机验收

### 本地检查

本地检查用于确认配置结构、规则及引用，不代替 Shadowrocket 导入解析器。下载规则的时间、内容和解析器版本都可能影响实际行为。

- [x] 分流及 DNS 参数未因文件清理或文档合并而改变，旧上游主配置已备份，文档没有空白错误或敏感信息。
- [x] 参数、策略引用、CIDR、规则类型及重复条目检查。
- [x] 远程规则下载及内容类型检查。
- [x] AI、Gemini、DeepSeek、汇丰、HTTPDNS、广告、普通 Apple、局域网及兜底的代表性策略检查。
- [x] 远程分类不可用或普通服务组被设为 REJECT 时，本地 AI 核心仍直接命中 PROXY。

2026-11-27 国内来源替换后的检查结果：17 个 General 参数、11 个手动策略组、109 条主配置规则，其中包含 22 个远程引用。全部远程引用成功读取，检查到的类型为 DOMAIN、DOMAIN-SUFFIX、DOMAIN-KEYWORD、IP-CIDR、IP-CIDR6、IP-ASN、USER-AGENT、URL-REGEX；CIDR 均为规范网络地址。

408 项静态策略断言通过。其中 37 个核心域名分别在普通与 CN 兜底条件、远程分类存在与缺失、服务组保持默认与全部设为 REJECT 的组合下检查，均先命中本地 PROXY；另对这 37 个域名附加国内 IPv4 目标验证，仍为 PROXY。还覆盖了淘宝、京东、支付宝、哔哩哔哩、铁路、网易、知乎等国内域名、新 CN 地址列表、本地保留后缀、Fake IP 地址、银行、广告前置拦截、微信 HTTPDNS、IPv6 地址和 STUN 端口等场景。模拟不实现 USER-AGENT、HTTPS 路径、真实 GEOIP/ASN 数据库或 Shadowrocket 的全部匹配语义。

上一轮 4 个公开 DoH 端点均通过 TLS 校验和 DNS wireformat 响应检查；本次 DNS 参数未变，没有重复请求。该检查由验证进程访问公开端点，不表示 DNS 已经通过个人 VPS，也不能证明客户端没有其他回退路径。

在本机发现 Shadowrocket 2.2.93（3488），已只读核对应用自带 `default.conf` 中的系统旁路、DNS、IPv6、私网应答和 UDP 回退参数。带代理 DoH、直连及节点引导 DNS、全 53 端口劫持等配置另参考维护者用例。没有通过应用导入验证，不能因此认定所有新增参数均已被目标客户端识别；其他设备版本也需单独记录。

### 目标设备验收

尚未修改任何客户端设置或停止 VPS。以下各项需要在实际使用设备上完成，完成前不能将本版本标为已通过防泄露验收。

| 项目 | 方法与通过条件 | 状态 |
| --- | --- | --- |
| 版本与导入 | 记录平台、系统和 Shadowrocket 版本；确认参数、IPv6 CIDR、逻辑端口规则及全部远程规则被识别 | 待验收 |
| AI 分流 | 在实际设备操作 ChatGPT、Claude、Claude Code、Gemini；API、认证、内容及重试均命中 PROXY | 待验收 |
| VPS 故障 | 隧道保持开启，仅让测试连接无法到达 VPS；触发重试，检查连接记录及必要抓包，无 AI 直连尝试 | 待验收 |
| DNS | 正常、主解析失败、备用失败分别检查；AI 查询不转向系统或国内直连解析器，VPS 名称引导查询单独识别 | 待验收 |
| 国内与银行 | 测试微信、国内 AI、国内网站、汇丰登录和必要银行业务；确认 DIRECT 及受控例外 | 待验收 |
| WebRTC | 检查浏览器候选地址、真实公网 IP 和非代理 UDP，同时测试国内通话是否受影响 | 待验收 |
| IPv6 | 检查 AAAA、原生 IPv6 地址连接、系统路由和 IPv6 Only 网络；没有意外直连 AI 目标 | 待验收 |
| UDP / QUIC | 使用实际 VPS 协议检查不支持 UDP 时拒绝；代理业务 QUIC 被限制，节点仍能建立连接 | 待验收 |
| 网络变化 | Wi-Fi 与蜂窝切换、休眠恢复、客户端重启后，仍使用当前配置和唯一 VPS | 待验收 |
| 广告与 Apple | 广告规则生效，登录、支付、iCloud、照片同步及推送没有不可接受的误伤 | 待验收 |

不要只因 AI 页面报错就判定故障测试通过：本地网络即使发起直连也可能无法打开该服务。需要证明没有走错连接路径。

## 更新与回滚

本地回滚副本保存在被 Git 忽略的 `.backups/` 中，不参与规则加载或上传。仓库保存个人配置的版本历史；本地备份用于快速恢复客户端已验证的版本。

1. 保存客户端当前可用配置和策略选择；修改前另存 `Personal.conf`、`AGENTS.md`、`README.md` 的本地备份，并在备份名称中记录日期和时间。
2. 参考上游更新时先下载到临时文件，用文件差异审阅规则和参数变动，不把上游主配置覆盖到 `Personal.conf`。
3. 新增域名说明服务、用途、策略、来源及必要性；涉及共享端点或直连时说明影响范围。
4. 先复查本地规则，再更新客户端远程规则，重跑受影响的实机项目。
5. 出现问题时恢复事先保存的本地文件并重新导入客户端，保留脱敏后的匹配信息供排查。不要通过临时将 AI 改为 DIRECT 处理代理故障。

本阶段没有更改 VPS、系统防火墙或客户端设置，实机验收状态见上表；也不包含自动化构建及 Mihomo 支持。

## 来源与许可

项目基于 [LingJingMaster/Shadowrocket-Rules](https://github.com/LingJingMaster/Shadowrocket-Rules)，保留上游署名与 [MIT 许可证](LICENSE)。
