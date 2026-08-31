---
schema_version: 1
type: source
research_direction: LLM效率与适配
research_type: 综述型
title: 有限数据下的大语言模型微调综述与实践指南
english_title: "Fine-tuning Large Language Models with Limited Data: A Survey and Practical Guide"
authors: [Marton Szep, Daniel Rueckert, Rüdiger von Eisenhart-Rothe, Florian Hinterwimmer]
venue: Transactions of the Association for Computational Linguistics
publication_date: 2026
doi: 10.1162/tacl.a.627
official_url: https://aclanthology.org/2026.tacl-1.17/
code_url: ""
status: unread
publication_status: 正式发表
domain: [自然语言处理, 大语言模型]
parent: "[[README]]"
related: ["[[2023-QLoRA-量化大模型高效微调]]", "[[2024-DoRA-权重分解低秩适配]]", "[[2025-LoRAPro-低秩适配优化]]"]
sources: [https://aclanthology.org/2026.tacl-1.17/]
created: 2026-08-18
updated: 2026-08-31
category: [小数据微调, PEFT, 综述]
relevance_to_DIULENS: 强
difficulty: 进阶
review_status: 方法综述
reading_status: 未读
priority: P1
progress: 0
read_minutes: 0
presentation_ready: false
demo_status: 未计划
next_action: 阅读方法选择标准与灾难性遗忘章节
tags: [方法/模型微调, 综述/大语言模型]
---

# 有限数据下的大语言模型微调综述与实践指南

> [!abstract] 一句话结论
> 这篇 TACL 综述围绕小数据条件下的参数高效微调、领域适配、合成反馈、模型专门化和灾难性遗忘给出选择框架。

## 为什么值得读

安全数据常常规模小、标签昂贵且类别不平衡，这篇综述比直接套用某一种 LoRA 变体更适合作为方法入口。

## 来源事实

- 论文系统覆盖 PEFT、领域和跨语言适配、模型专门化及偏好对齐。
- 强调样本效率、计算效率、模型与数据规模以及灾难性遗忘之间的权衡。
- 论文发表于 TACL 2026，页码 341–377。

## 我的理解

是否微调应由错误模式决定：若问题来自缺少项目上下文，应优先检索与程序分析；若模型稳定地不认识某类 API 或输出格式，再考虑 PEFT。

## 局限性

综述提供通用原则，但不会替代特定代码模型、漏洞类型和数据切分上的实验。

## 待验证

- [ ] 提取适合少量高质量安全样本的决策树。
- [ ] 增加时间切分、跨项目切分和重复代码去污染要求。

## 下一步

用综述的选择标准判断当前课题应该使用 prompt、RAG、QLoRA 还是全参数微调。
