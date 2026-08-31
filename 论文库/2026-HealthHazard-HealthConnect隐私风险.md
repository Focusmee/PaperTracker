---
schema_version: 1
type: source
research_direction: 移动隐私与软件供应链
research_type: 探索型
title: Health Hazard：Android Health Connect 的隐私风险
english_title: "Health Hazard, Handle with Care: Investigating the Privacy Risks of Android's Health Connect"
authors: [Konstantinos Spyridakis, Ioannis Arkalakis, Michalis Diamantaris, Sotiris Ioannidis, Jason Polakis, Panagiotis Ilia]
venue: 35th USENIX Security Symposium
publication_date: 2026-08-14
official_url: https://www.usenix.org/conference/usenixsecurity26/presentation/spyridakis
pdf_url: https://www.usenix.org/system/files/usenixsecurity26-spyridakis.pdf
code_url: ""
local_pdf: "[[附件/论文原文/06-Health-Hazard-英文原文-USENIX-Security-2026.pdf]]"
status: unread
publication_status: 正式发表
domain: [移动应用隐私, Android, 隐私合规]
parent: "[[README]]"
related: ["[[论文库/2026-AndroidMRI-Android内存隐私泄漏检测]]", "[[论文库/2026-DIULENS-移动CMP隐私风险]]"]
sources: [https://www.usenix.org/conference/usenixsecurity26/presentation/spyridakis, https://www.usenix.org/system/files/usenixsecurity26-spyridakis.pdf]
created: 2026-08-16
updated: 2026-08-31
category: [Android隐私, 隐私合规, 动态分析]
relevance_to_DIULENS: 强
difficulty: 进阶
review_status: 泛读
reading_status: 未读
priority: P2
progress: 0
read_minutes: 0
started_date:
last_read:
finished_date:
rating:
tags: [研究/移动隐私, 方法/动态分析]
presentation_ready: false
demo_status: 未计划
next_action: 阅读图2、第3节动态分析管线和第8节局限
tags: [USENIX-Security-2026, Android, Health-Connect]
---

# Health Hazard：Android Health Connect 的隐私风险

> [!abstract] 一句话结论
> Health Connect 承诺本地存储、细粒度权限和用户控制，但对351个真实App的半自动动态分析发现，大量App的UI/披露与真实访问、分享和网络外传不一致。

## 为什么值得读

本文与 [[DIULENS]] 都研究“用户看到的控制和真实行为是否一致”。对象是 Health Connect 权限、UI、Data Safety、隐私政策与运行数据流，适合学习怎样把政策要求转成 API Hook、栈追踪、合成数据和网络证据。

## 研究问题、输入与输出

- 问题：Health Connect 架构和 App 使用是否真正实现透明、最小访问和用户控制？
- 风险主体：获授权健康 App、第一方后端和第三方组件；不要求突破权限。
- 输入：2,348,217 App预筛、1,068个被操作App、最终351个完整样本。
- 输出：Health API调用、调用栈、数据来源、网络外传和声明不一致。

## 工作链路与分工

1. Manifest健康权限筛选App。
2. Toy App写入39类可识别合成记录。
3. 人工操作目标App。
4. Frida Hook Health API，记录参数、对象和调用栈。
5. BurpSuite与反pinning记录网络流量。
6. 搜索原值、Base64、哈希和编码变换并人工确认。
7. 对照App UI、Data Safety和隐私政策。

静态分析负责Manifest初筛；动态分析提供API和网络事实；合成数据相当于示踪剂；人工负责定向操作和合规语义；LLM/Agent不是组件，但可改进人工GUI瓶颈。

## 零基础术语

- Health Connect：Android中央健康数据仓库。
- Priming：先注入已知合成值，便于追踪。
- Stack trace：显示谁一路调用到API，可区分第一方和第三方。
- 网络外传：数据离开设备；即使发到第一方后端，用户也失去设备内可见控制。

## 实验与关键数字

- 预处理2,348,217个App；人工操作1,068个；最终351个。
- 60.9%未遵守UI透明指南。
- 25.6%未报告健康记录收集或分享。
- 19.6%（69个）将数据发往网络，并可能与姓名、邮箱、广告ID、电话关联。
- Pixel 6/AOSP Android 14、Magisk、Frida、BurpSuite；网络泄漏均人工核验。

> [!warning] 结果口径
> 作者将任何Health Connect数据离设备都计为leak，这不自动等同违法；合规仍取决于同意、目的、声明、接收方和法域。

## 精读顺序

摘要和图2 → §3.1动态管线 → §4数据集 → §5.3网络外传 → §7讨论 → §8局限。

## 局限与复现

人工操作限制规模；未知加密流量会漏掉，结果是下界；排除付费、会员和外设App影响推广性。完整复现需真机、root、账号、代理和反pinning。论文称公开代码/数据，但正式页面尚无直接仓库链接。

## 论文关系

- [[DIULENS]]：都比较可见承诺和运行行为。
- [[PICOSCAN]]：前者偏大规模静态配置；本文以人工动态证据换语义。
- [[2026-BridgesToSelf-移动端localhost静默追踪]]：都组合多源证据。

## 三分钟讲解

Health Connect是健康数据中转库；Toy App写示踪数据，Frida记API，Burp抓网络，再对照政策；给出60.9%、25.6%、19.6%；Agent可提高GUI覆盖，但结论仍需Hook/网络证据。

## 最小 Demo

Producer写合成步数/心率，Reader读取并可选发送到本地测试服务。Hook记录时间、类型、来源包和栈；声明JSON写“只读步数、不上传”，规则引擎输出未声明读取/上传。

## 思考题

1. 读取权限是否等于同意数据离设备？
2. 任何离设备传输都称leak是否过严？
3. 合成值匹配会漏掉哪些变换？
4. Agent提高GUI覆盖后如何证明未漏路径？
5. Data Safety、政策和UI冲突时以谁为Oracle？
6. 能否把来源包名纳入DIULENS供应链规则？

## 下一步

读图2并复述 Priming→Probing→Correlation，再思考 Appium+LLM 如何替代部分人工操作而不直接判泄漏。

## 正式来源

- [USENIX页面](https://www.usenix.org/conference/usenixsecurity26/presentation/spyridakis)
- [正式PDF](https://www.usenix.org/system/files/usenixsecurity26-spyridakis.pdf)
