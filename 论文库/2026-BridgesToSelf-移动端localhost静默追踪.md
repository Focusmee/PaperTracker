---
schema_version: 1
type: source
research_direction: 移动隐私与软件供应链
research_type: 探索型
title: Bridges to Self：移动端 localhost 静默 Web-to-App 追踪
english_title: "Bridges to Self: Silent Web-to-App Tracking on Mobile via Localhost"
authors: [Tim Vlummens, Aniketh Girish, Nipuna Weerasekara, Frederik Zuiderveen Borgesius, Gunes Acar, Narseo Vallina-Rodriguez]
venue: 35th USENIX Security Symposium
publication_date: 2026-08-14
official_url: https://www.usenix.org/conference/usenixsecurity26/presentation/vlummens
pdf_url: https://www.usenix.org/system/files/usenixsecurity26-vlummens.pdf
code_url: https://localmess.github.io/
local_pdf: "[[附件/论文原文/05-Bridges-to-Self-英文原文-USENIX-Security-2026.pdf]]"
status: unread
publication_status: 正式发表
domain: [移动应用隐私, 跨上下文追踪, Android]
parent: "[[README]]"
related: ["[[论文库/2026-DIULENS-移动CMP隐私风险]]", "[[论文库/2026-PlainTextPlainRisks-AndroidWebView明文风险]]"]
sources: [https://www.usenix.org/conference/usenixsecurity26/presentation/vlummens, https://www.usenix.org/system/files/usenixsecurity26-vlummens.pdf, https://localmess.github.io/]
created: 2026-08-16
updated: 2026-08-31
category: [移动应用隐私, 跨上下文追踪, 静态与动态分析]
relevance_to_DIULENS: 强
difficulty: 进阶
review_status: 精读候选
reading_status: 未读
priority: P1
progress: 0
read_minutes: 0
started_date:
last_read:
finished_date:
rating:
tags: [研究/移动隐私, 方法/动态分析]
presentation_ready: false
demo_status: 未计划
next_action: 用20分钟阅读图1、威胁模型和第4节方法并画出证据链
tags: [USENIX-Security-2026, Android, 隐私合规, tracking]
---

# Bridges to Self：移动端 localhost 静默 Web-to-App 追踪

> [!abstract] 一句话结论
> Meta 与 Yandex 曾让网页追踪脚本通过手机 localhost 与原生 App 通信，把网页 Cookie 和长期原生身份关联；该行为可在接受 Cookie 同意前启动，并绕过无痕模式、清 Cookie、重置广告 ID、VPN 与工作/个人资料隔离。

## 为什么值得读

[[DIULENS]] 检查 CMP 选择是否正确传给移动 SDK；本文证明即使 App 内配置看似正确，数据仍可能走“网站脚本→浏览器→localhost→原生 App/SDK→云端”。它启发新规则：拒绝前是否已产生跨上下文标识关联？

## 研究问题、威胁模型、输入与输出

- 问题：网页脚本能否绕过 Web/App 隔离，关联网页标识与长期移动身份？是否真实部署、是否在同意前发生？
- 攻击方：同时控制高覆盖网页脚本和广泛安装的 App/SDK，并让 App 监听 localhost。
- 输入：Top-100K 网站、5,000 个热门 Android App、真实浏览器和移动设备。
- 输出：website–app 配对、localhost 通信、ID 关联、同意前行为和防御评估。

## 系统工作链路

1. 网页爬虫记录 HTTP(S)、WebSocket、WebRTC、Cookie 与 JS API。
2. 筛出 localhost、127.0.0.0/8、::1 和相关端口。
3. 静态扫描 App 的本地端口、嵌入服务器、字符串和 SDK。
4. 真机运行；Frida 追踪 Java/Native、加密前字段与 socket 写入。
5. tcpdump、mitmproxy、netstat 记录请求、响应和端口。
6. 按端口、协议、Cookie、标识和时间关联网页/App证据。
7. 对比接受 Cookie 与不操作同意框两种实验。

### 分工

- 静态分析：定位和解释候选；受混淆、动态加载影响。
- 动态分析：主要事实证据；受路径、地域、时序和服务器控制影响。
- 知识规则：localhost 地址、Web API、哈希与 SDK 包名。
- LLM/Agent：不是本文核心。
- 人工验证：确认追踪语义、厂商、同意状态和政策含义。

## 零基础术语

- localhost：设备自己的地址，如 127.0.0.1；网页请求可能打到同一手机监听端口的 App。
- Cookie：网页身份标识；App 若返还长期账号 ID，可在清 Cookie 后重新识别。
- Web-to-App bridge：网页与原生 App 经本机网络交换标识。
- 确定性追踪：直接共享 ID 建立一一关联，而非概率猜测。

## 实验与关键数字

- USENIX Security 2026 正式论文，Distinguished Paper Award。
- CrUX Top-100K 网站，爬取成功访问约 92%–96%。
- 5,000 个美欧热门 Android App；单 App 自动执行 10 分钟。
- 确认 Meta Pixel/Yandex Metrica 部署及同意前 bridging。
- 披露促成缓解；Meta/Yandex 于 2025-06-03 停止该机制。

> [!warning] 正确解读
> localhost 也可用于调试和合法 IPC，不能仅凭字符串判恶意，必须结合动态标识流和语义证据。

## 精读顺序

摘要、图1 → §3威胁模型 → §4方法（重点§4.3）→ 同意实验与测量 → 防御和披露。

## 局限与复现

静态会误报/漏报；动态只能覆盖执行路径；完整复现需真机池、Frida、代理和网站爬虫。机制披露后已停止，历史结果不代表今天仍在发生。

## 论文关系

- [[DIULENS]]：从 CMP→SDK 扩展到 Web→App。
- [[PICOSCAN]]：都组合静态与动态；本文以动态为主证据。
- [[2026-HealthHazard-HealthConnect隐私风险]]：都对照平台承诺和真实数据流。

## 三分钟讲解

沙箱本应隔离网页/App；网页请求 127.0.0.1，App 监听并交换身份；介绍 Top-100K、5,000 App、Frida 与抓包；强调同意前追踪；提出 DIULENS 新规则“拒绝前不得建立跨上下文标识关联”。

## 最小 Demo

自写 Android App 监听 localhost；本地测试网页发送随机 Web ID；App 返回模拟 App ID。记录“时间—Web ID—端口—App ID—同意状态”。比较同意后连接与同意前连接，只使用合成标识。

## 思考题

1. 如何区分合法 localhost 和追踪？
2. 动态测试10分钟没发现能否证明安全？
3. 如何把 consent=false 与 socket 时间线关联？
4. 标识哈希后如何证明同一身份？
5. DIULENS 是否应加入跨上下文传播和同意前副作用？
6. mDNS/WebRTC 旁路说明知识库应如何版本化？

## 下一步

读图1和§4，并解释为什么静态与动态单独都不够。

## 正式来源

- [USENIX页面](https://www.usenix.org/conference/usenixsecurity26/presentation/vlummens)
- [正式PDF](https://www.usenix.org/system/files/usenixsecurity26-vlummens.pdf)
- [作者项目页](https://localmess.github.io/)
