---
schema_version: 1
type: source
research_direction: LLM效率与适配
research_type: 应用型
title: GaLore：基于梯度低秩投影的内存高效训练
english_title: "GaLore: Memory-Efficient LLM Training by Gradient Low-Rank Projection"
authors: [Jiawei Zhao, Zhenyu Zhang, Beidi Chen, Zhangyang Wang, Anima Anandkumar, Yuandong Tian]
venue: ICML 2024
publication_date: 2024-07
doi: ""
official_url: https://proceedings.mlr.press/v235/zhao24s.html
code_url: ""
status: unread
publication_status: 正式发表
domain: [机器学习系统, 大语言模型]
parent: "[[README]]"
related: ["[[2023-QLoRA-量化大模型高效微调]]", "[[2024-DoRA-权重分解低秩适配]]", "[[2026-FineTuningLimitedData-小数据微调综述]]"]
sources: [https://proceedings.mlr.press/v235/zhao24s.html]
created: 2026-08-18
updated: 2026-08-31
category: [低秩梯度, 训练显存, 全参数学习]
relevance_to_DIULENS: 中
difficulty: 困难
review_status: 扩展阅读
reading_status: 未读
priority: P3
progress: 0
read_minutes: 0
presentation_ready: false
demo_status: 未计划
next_action: 理解梯度投影与LoRA训练参数的区别
tags: [方法/模型训练, 方法/低秩优化]
---

# GaLore：基于梯度低秩投影的内存高效训练

> [!abstract] 一句话结论
> GaLore 对优化梯度做低秩投影，在保留全参数学习形式的同时减少优化器状态显存。

## 为什么值得读

它与 LoRA/DoRA 不同：不是冻结基座后加适配器，而是尝试更省内存地更新完整模型参数。

## 来源事实

- 论文报告优化器状态内存最高减少 65.5%。
- 实验包括 LLaMA 1B、7B 预训练以及 RoBERTa 下游微调。
- 论文发表于 ICML 2024。

## 我的理解

对当前安全课题，GaLore 的工程成本通常高于 QLoRA，更适合需要持续预训练或全参数领域适配时考虑。

## 局限性

原论文重点是训练内存，不是小样本安全任务效果；全参数更新也更容易产生灾难性遗忘和部署成本。

## 待验证

- [ ] 确认目标硬件和代码模型是否有成熟实现。
- [ ] 与 QLoRA 比较总训练显存、时间和部署方式，而不只比较参数量。

## 下一步

将 GaLore 标记为后续路线，首轮实验优先 QLoRA。
