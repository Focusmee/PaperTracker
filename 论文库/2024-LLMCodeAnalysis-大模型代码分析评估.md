---
schema_version: 1
type: source
research_direction: LLM程序分析与软件安全
research_type: 评测型
title: 大语言模型代码分析能力系统评估
english_title: "Large Language Models for Code Analysis: Do LLMs Really Do Their Job?"
authors: [Chongzhou Fang, Ning Miao, Shaurya Srivastav, Jialin Liu, Ruoyu Zhang, Ruijie Fang, Asmita, Ryan Tsang, Najmeh Nazari, Han Wang, Houman Homayoun]
venue: 33rd USENIX Security Symposium
publication_date: 2024-08-16
doi: ""
official_url: https://www.usenix.org/conference/usenixsecurity24/presentation/fang
code_url: ""
status: unread
publication_status: 正式发表
domain: [软件安全, 程序分析, 大语言模型]
parent: "[[README]]"
related: ["[[2024-LLM4Decompile-大模型反编译]]", "[[2025-LLMxCPG-CPG引导漏洞检测]]", "[[2026-BINREX-智能体静态二进制分析]]"]
sources: [https://www.usenix.org/conference/usenixsecurity24/presentation/fang]
created: 2026-08-18
updated: 2026-08-31
category: [能力评估, 混淆代码, 代码分析]
relevance_to_DIULENS: 强
difficulty: 进阶
review_status: 基线阅读
reading_status: 未读
priority: P1
progress: 0
read_minutes: 0
presentation_ready: false
demo_status: 未计划
next_action: 阅读任务定义、混淆设置和失败案例
tags: [研究/LLM程序分析, 方法/评测]
---

# 大语言模型代码分析能力系统评估

> [!abstract] 一句话结论
> 论文系统检查 LLM 在多种代码分析任务、混淆代码和真实案例中的能力，结论是 LLM 有辅助价值，但不能脱离工具和验证独立承担程序分析。

## 为什么值得读

它适合作为任何新系统的“LLM-only”基线和失败模式来源，避免只报告模型成功案例。

## 来源事实

- 研究重点包括 LLM 代码理解、程序分析任务以及代码混淆带来的影响。
- 论文给出了真实场景案例，同时强调模型能力存在明显限制。
- 论文发表于 USENIX Security 2024，页码 829–846。

## 我的理解

后续 LLMxCPG、IRIS、BINREX 的共同演进正是把 LLM 从“直接回答者”改造成“受程序结构、工具输出和验证器约束的语义模块”。

## 局限性

商业模型快速迭代会使绝对能力数字过时，但任务设计、混淆鲁棒性和工具验证结论仍可复用。

## 待验证

- [ ] 提取论文使用的全部代码分析任务和 prompt 模板。
- [ ] 用当前开源代码模型复现实验中的一个混淆对照组。

## 下一步

建立“纯 LLM—LLM+切片—LLM+工具—LLM+工具+验证”四级基线。
