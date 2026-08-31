---
schema_version: 1
type: source
research_direction: LLM效率与适配
research_type: 应用型
title: LoRA-Pro：低秩适配器的优化改进
english_title: "LoRA-Pro: Are Low-Rank Adapters Properly Optimized?"
authors: [Zhengbo Wang, Jian Liang, Ran He, Zilei Wang, Tieniu Tan]
venue: ICLR 2025 Spotlight
publication_date: 2025
doi: ""
official_url: https://openreview.net/forum?id=gTwRMU3lJ5
code_url: ""
status: unread
publication_status: 正式发表
domain: [机器学习, 大语言模型]
parent: "[[README]]"
related: ["[[2023-QLoRA-量化大模型高效微调]]", "[[2024-DoRA-权重分解低秩适配]]", "[[2026-FineTuningLimitedData-小数据微调综述]]"]
sources: [https://openreview.net/forum?id=gTwRMU3lJ5]
created: 2026-08-18
updated: 2026-08-31
category: [LoRA, 梯度优化, 参数高效微调]
relevance_to_DIULENS: 中
difficulty: 困难
review_status: 扩展阅读
reading_status: 未读
priority: P2
progress: 0
read_minutes: 0
presentation_ready: false
demo_status: 未计划
next_action: 阅读LoRA与低秩全参数梯度的等价推导
tags: [方法/模型微调, 方法/低秩适配]
---

# LoRA-Pro：低秩适配器的优化改进

> [!abstract] 一句话结论
> LoRA-Pro 将 LoRA 优化解释为使用低秩梯度进行全参数更新，并据此调整两个低秩矩阵的梯度以缩小与全参数微调的差距。

## 为什么值得读

它提供了从优化器而非模块结构改进 LoRA 的路线，可作为高质量微调实验中的最新强基线之一。

## 来源事实

- 论文发表于 ICLR 2025，并获 Spotlight。
- 实验覆盖自然语言理解、对话、数学推理、代码生成和图像分类。
- 作者报告 LoRA-Pro 在多类任务上改善普通 LoRA，并缩小与全参数微调的差距。

## 我的理解

LoRA-Pro 适合研究“同样参数量下能否更好优化”，但不能替代安全数据构造和程序级评测。

## 局限性

论文的代码任务以生成为主，不代表在漏洞检测、隐私流分析或反编译上已有结论。

## 待验证

- [ ] 核对官方代码和 Hugging Face PEFT 支持状态。
- [ ] 在相同数据、rank、步数下与 QLoRA/DoRA 比较。

## 下一步

仅在普通 QLoRA 已显示有效后加入 LoRA-Pro 消融。
