---
schema_version: 1
type: source
research_direction: LLM效率与适配
research_type: 应用型
title: BRIEF-Pro：面向多跳推理的通用上下文压缩
english_title: "BRIEF-Pro: Universal Context Compression with Short-to-Long Synthesis for Fast and Accurate Multi-Hop Reasoning"
authors: [Jia-Chen Gu, Junyi Zhang, Di Wu, Yuankai Li, Kai-Wei Chang, Nanyun Peng]
venue: Findings of ACL 2026
publication_date: 2026-07
doi: 10.18653/v1/2026.findings-acl.696
official_url: https://aclanthology.org/2026.findings-acl.696/
code_url: ""
status: unread
publication_status: 正式发表
domain: [自然语言处理, 大语言模型]
parent: "[[README]]"
related: ["[[2024-LongLLMLingua-长上下文提示压缩]]", "[[2025-PromptCompressionSurvey-提示词压缩综述]]"]
sources: [https://aclanthology.org/2026.findings-acl.696/]
created: 2026-08-18
updated: 2026-08-31
category: [上下文压缩, 多跳推理, RAG]
relevance_to_DIULENS: 中强
difficulty: 进阶
review_status: 方法阅读
reading_status: 未读
priority: P2
progress: 0
read_minutes: 0
presentation_ready: false
demo_status: 未计划
next_action: 阅读短到长训练和可控摘要长度设计
tags: [方法/上下文压缩, 方法/RAG]
---

# BRIEF-Pro：面向多跳推理的通用上下文压缩

> [!abstract] 一句话结论
> BRIEF-Pro 用短上下文种子训练可处理超过 10K 词上下文的摘要压缩器，并允许直接指定摘要句数。

## 为什么值得读

它代表“先抽象成紧凑证据摘要，再交给阅读模型”的路线，适合作为结构化程序切片之外的生成式压缩基线。

## 来源事实

- 评测覆盖四个开放域多跳问答数据集。
- 在 70B reader 上，论文报告 32 倍压缩相对 LongLLMLingua 的 9 倍压缩平均提高 4.67% QA 性能。
- 其计算开销被报告为 LongLLMLingua 的 23%。

## 我的理解

生成式摘要可能把代码事实改写错。用于安全审计时，摘要必须附带原始地址、函数和路径引用，才能被复核。

## 局限性

主要实验是自然语言问答而非代码；摘要忠实性不能仅用最终答案准确率代替。

## 待验证

- [ ] 检查压缩后实体、数值和否定条件的保真评测。
- [ ] 尝试让摘要输出引用原始代码行或图节点。

## 下一步

把“摘要长度”改成 token 预算控制变量，观察漏洞召回随预算变化的曲线。
