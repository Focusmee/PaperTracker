---
schema_version: 1
type: source
research_direction: LLM程序分析与软件安全
research_type: 应用型
title: SAN2PATCH：基于Tree-of-Thought分析的漏洞自动修复
english_title: "Logs In, Patches Out: Automated Vulnerability Repair via Tree-of-Thought LLM Analysis"
authors: [Youngjoon Kim, Sunguk Shin, Hyoungshick Kim, Jiwon Yoon]
venue: 34th USENIX Security Symposium
publication_date: 2025-08-15
doi: ""
official_url: https://www.usenix.org/conference/usenixsecurity25/presentation/kim-youngjoon
code_url: ""
status: unread
publication_status: 正式发表
domain: [软件安全, 大语言模型]
parent: "[[README]]"
related: ["[[2025-APPATCH-自适应提示漏洞修复]]", "[[2024-PentestGPT-自动化渗透测试]]"]
sources: [https://www.usenix.org/conference/usenixsecurity25/presentation/kim-youngjoon]
created: 2026-08-18
updated: 2026-08-31
category: [Tree-of-Thought, 多阶段推理, 漏洞修复]
relevance_to_DIULENS: 中强
difficulty: 困难
review_status: 扩展精读
reading_status: 未读
priority: P2
progress: 0
read_minutes: 0
presentation_ready: false
demo_status: 未计划
next_action: 阅读四阶段推理和候选剪枝机制
tags: [研究/漏洞修复, 方法/提示词]
---

# SAN2PATCH：基于Tree-of-Thought分析的漏洞自动修复

> [!abstract] 一句话结论
> SAN2PATCH 将 sanitizer 日志和源码逐步转化为漏洞理解、故障定位、修复策略和补丁，并在各阶段生成、评分和剪枝多个推理候选。

## 为什么值得读

它展示如何为复杂代码安全任务设计分阶段提示与验证，而不是让模型一次完成全部推理。

## 来源事实

- 在 VulnLoc 上成功修复 79.5% 的漏洞；作者列出的 ExtractFix 和 VulnFix 基线分别为 43% 和 51%。
- 在包含 27 个较新漏洞的 SAN2VULN 上成功率为 63%。
- 对缓冲区溢出，论文报告 81.8% 修复成功率。

## 我的理解

Tree-of-Thought 会显著增加 token，应与候选多样性、修复率和验证成本一起报告；并非分支越多越好。

## 局限性

结果依赖 sanitizer 日志质量、测试覆盖和模型自评分可靠性；同一模型评分可能偏好自己生成的错误解释。

## 待验证

- [ ] 统计各阶段的实际 token、候选数和剪枝阈值。
- [ ] 检查补丁通过测试是否足以排除过拟合修复。

## 下一步

将四阶段流程改成漏洞检测版本，并标注每阶段所需的外部证据。
