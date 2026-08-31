---
schema_version: 1
type: source
research_direction: LLM程序分析与软件安全
research_type: 应用型
title: IRIS：LLM辅助的静态安全漏洞分析
english_title: "LLM-Assisted Static Analysis for Detecting Security Vulnerabilities"
authors: [Ziyang Li, Saikat Dutta, Mayur Naik]
venue: International Conference on Learning Representations
publication_date: 2025
doi: ""
official_url: https://openreview.net/forum?id=9LdJDU7E91
code_url: https://github.com/iris-sast/iris
local_pdf: "[[附件/论文原文/09-IRIS-英文原文-ICLR-2025公开版.pdf]]"
status: unread
publication_status: 正式发表
domain: [软件安全, 程序分析, 大语言模型]
parent: "[[README]]"
related: ["[[2025-LLMxCPG-CPG引导漏洞检测]]", "[[2026-BugAuditor-不一致防御代码审计]]", "[[2026-BINREX-智能体静态二进制分析]]"]
sources: [https://openreview.net/forum?id=9LdJDU7E91, https://github.com/iris-sast/iris]
created: 2026-08-18
updated: 2026-08-31
category: [神经符号方法, 静态污点分析, 漏洞检测]
relevance_to_DIULENS: 极强
difficulty: 困难
review_status: 精读候选
reading_status: 未读
priority: P1
progress: 0
read_minutes: 0
presentation_ready: false
demo_status: 未计划
next_action: 对照专题第06章阅读source、sink和CodeQL链路
tags: [研究/LLM程序分析, 方法/静态污点分析]
---

# IRIS：LLM辅助的静态安全漏洞分析

> [!abstract] 一句话结论
> IRIS 让 LLM 生成漏洞类别相关的 source/sink 规格并筛选候选路径，让 CodeQL 执行跨仓库、可追踪的静态污点分析。

## 为什么值得读

它是“LLM 提供语义规格、传统程序分析提供确定性路径证据”的代表性神经符号系统，也是当前专题第 06 章的正式来源卡。

## 来源事实

- 初始论文数据集包含 120 个经人工验证的真实 Java 漏洞。
- CodeQL 单独检测到 27 个，IRIS 与 GPT-4 组合检测到 55 个，并将平均误发现率降低 5 个百分点。
- 作者报告系统还发现 6 个此前未知漏洞。

## 我的理解

IRIS 的创新不在于让 LLM 取代 CodeQL，而在于补充静态分析长期依赖人工维护的 API 污点规格，并用 LLM 做上下文过滤。

## 局限性

结果主要来自 Java 与指定 CWE；LLM 生成错误规格时仍可能造成系统性漏报或误报，且 API 调用和框架版本变化会影响迁移。

## 待验证

- [ ] 核对正式版本与代码仓库中 CWE-Bench-Java 扩展后的样本数量差异。
- [ ] 复现一个最小第三方库 source/sink 规格生成案例。

## 下一步

下一步：继续整理 LLM 与 CodeQL 的责任边界，并把结论直接补充到本页。
