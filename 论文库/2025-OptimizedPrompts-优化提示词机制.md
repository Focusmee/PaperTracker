---
schema_version: 1
type: source
research_direction: LLM效率与适配
research_type: 探索型
title: 优化提示词在语言模型中的机制分析
english_title: "Demystifying Optimized Prompts in Language Models"
authors: [Rimon Melamed, Lucas Hurley McCabe, H. Howie Huang]
venue: EMNLP 2025 Main Conference
publication_date: 2025-11
doi: 10.18653/v1/2025.emnlp-main.147
official_url: https://aclanthology.org/2025.emnlp-main.147/
code_url: ""
status: unread
publication_status: 正式发表
domain: [自然语言处理, 大语言模型]
parent: "[[README]]"
related: ["[[2025-GoodPrompt-自然语言提示词质量]]", "[[2025-APPATCH-自适应提示漏洞修复]]"]
sources: [https://aclanthology.org/2025.emnlp-main.147/]
created: 2026-08-18
updated: 2026-08-31
category: [自动提示优化, 可解释性, 模型内部机制]
relevance_to_DIULENS: 中
difficulty: 困难
review_status: 方法阅读
reading_status: 未读
priority: P2
progress: 0
read_minutes: 0
presentation_ready: false
demo_status: 未计划
next_action: 阅读优化prompt的token组成与激活分析
tags: [方法/提示词, 研究/模型机制]
---

# 优化提示词在语言模型中的机制分析

> [!abstract] 一句话结论
> 论文发现机器优化出的提示词常包含稀有标点和名词 token，并能通过模型内部少量激活维度与自然语言提示明显区分。

## 为什么值得读

自动提示优化可能获得高分却难以解释；安全分析若使用这类 prompt，需要额外检查跨模型迁移、鲁棒性和是否意外触发异常行为。

## 来源事实

- 研究比较机器优化提示与自然语言提示的 token 组成和模型内部表示。
- 优化提示在多个指令模型家族中表现出相似的表示形成路径。
- 论文发表于 EMNLP 2025 Main。

## 我的理解

面向安全任务不能只以 benchmark 分数选择 prompt，还应把可读性、可审计性、迁移性和对输入扰动的稳定性纳入目标函数。

## 局限性

内部激活差异说明模型处理方式不同，但不直接给出优化 prompt 在真实安全任务上可靠的因果解释。

## 待验证

- [ ] 检查论文覆盖的模型家族和任务是否包含代码模型。
- [ ] 比较自然语言 prompt 与自动优化 prompt 在代码混淆下的稳定性。

## 下一步

把“性能、token、可解释性、鲁棒性”列为提示词优化的四个评价轴。
