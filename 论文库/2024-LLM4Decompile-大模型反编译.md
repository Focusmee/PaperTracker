---
schema_version: 1
type: source
research_direction: LLM程序分析与软件安全
research_type: 应用型
title: LLM4Decompile：使用大语言模型反编译二进制代码
english_title: "LLM4Decompile: Decompiling Binary Code with Large Language Models"
authors: [Hanzhuo Tan, Qi Luo, Jing Li, Yuqun Zhang]
venue: EMNLP 2024 Main Conference
publication_date: 2024-11
doi: 10.18653/v1/2024.emnlp-main.203
official_url: https://aclanthology.org/2024.emnlp-main.203/
code_url: ""
local_pdf: "[[附件/论文原文/07-LLM4Decompile-英文原文-EMNLP-2024.pdf]]"
status: unread
publication_status: 正式发表
domain: [二进制分析, 大语言模型]
parent: "[[README]]"
related: ["[[2026-BINREX-智能体静态二进制分析]]", "[[2024-LLMCodeAnalysis-大模型代码分析评估]]", "[[2023-QLoRA-量化大模型高效微调]]"]
sources: [https://aclanthology.org/2024.emnlp-main.203/]
created: 2026-08-18
updated: 2026-08-31
category: [二进制反编译, 代码大模型, 领域微调]
relevance_to_DIULENS: 强
difficulty: 困难
review_status: 精读候选
reading_status: 未读
priority: P1
progress: 0
read_minutes: 0
presentation_ready: false
demo_status: 未计划
next_action: 阅读训练数据构造与End、Ref两类模型差异
tags: [研究/二进制分析, 方法/模型微调]
---

# LLM4Decompile：使用大语言模型反编译二进制代码

> [!abstract] 一句话结论
> LLM4Decompile 训练 1.3B–33B 参数模型直接将二进制恢复为高级代码，并用另一个系列改写 Ghidra 反编译结果。

## 为什么值得读

它提供了二进制—源码数据对构造、反编译模型微调和可执行性评测的完整起点，适合支撑“先恢复语义，再检测隐私风险”的研究路线。

## 来源事实

- 作者报告 LLM4Decompile-End 在 HumanEval 与 ExeBench 的可重新执行率上显著超过 GPT-4o 和 Ghidra。
- LLM4Decompile-Ref 对 Ghidra 输出进行微调改写，相对 End 模型进一步提升 16.2%。
- 论文发表于 EMNLP 2024 Main。

## 我的理解

“看起来像源码”不是充分指标。用于安全分析时还应评估调用关系、条件分支、数据流和漏洞语义是否被忠实恢复。

## 局限性

训练集编译器、优化级别和函数边界可能形成分布偏差；重新执行成功也不能证明反编译结果保留所有安全语义。

## 待验证

- [ ] 核对官方代码与模型地址、许可证和训练语料范围。
- [ ] 设计隐私 source/sink 保留率而非仅可执行率的评测。

## 下一步

比较 LLM4Decompile 的模型式反编译与 BINREX 的工具式任务分析。
