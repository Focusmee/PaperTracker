---
schema_version: 1
type: source
research_direction: LLM效率与适配
research_type: 应用型
title: QLoRA：量化大语言模型的高效微调
english_title: "QLoRA: Efficient Finetuning of Quantized LLMs"
authors: [Tim Dettmers, Artidoro Pagnoni, Ari Holtzman, Luke Zettlemoyer]
venue: NeurIPS 2023 Main Conference
publication_date: 2023
doi: 10.52202/075280-0441
official_url: https://proceedings.neurips.cc/paper_files/paper/2023/hash/1feb87871436031bdc0f2beaa62a049b-Abstract-Conference.html
code_url: https://github.com/artidoro/qlora
status: unread
publication_status: 正式发表
domain: [机器学习系统, 大语言模型]
parent: "[[README]]"
related: ["[[2024-DoRA-权重分解低秩适配]]", "[[2025-LoRAPro-低秩适配优化]]", "[[2026-FineTuningLimitedData-小数据微调综述]]"]
sources: [https://proceedings.neurips.cc/paper_files/paper/2023/hash/1feb87871436031bdc0f2beaa62a049b-Abstract-Conference.html, https://github.com/artidoro/qlora]
created: 2026-08-18
updated: 2026-08-31
category: [量化, LoRA, 参数高效微调]
relevance_to_DIULENS: 强
difficulty: 进阶
review_status: 方法基线
reading_status: 未读
priority: P1
progress: 0
read_minutes: 0
presentation_ready: false
demo_status: 未计划
next_action: 阅读NF4、双重量化和分页优化器
tags: [方法/模型微调, 方法/量化]
---

# QLoRA：量化大语言模型的高效微调

> [!abstract] 一句话结论
> QLoRA 冻结 4-bit 量化基座模型，只训练低秩适配器，从而大幅降低显存需求并保持接近 16-bit 微调的任务表现。

## 为什么值得读

它仍是实验室预算下微调 7B–14B 开源代码模型的关键基线，适合训练 source/sink 识别、函数语义或候选排序模块。

## 来源事实

- 论文展示在单张 48GB GPU 上微调 65B 参数模型。
- 核心组件包括 4-bit NormalFloat、双重量化和分页优化器。
- 作者微调了超过 1,000 个模型并开源实现。

## 我的理解

QLoRA 降低的是训练显存门槛，不会解决错误标签、数据泄漏、跨项目泛化或漏洞证据不足。

## 局限性

论文主要评估通用指令任务；代码安全微调仍需单独验证长代码、类别不平衡和时序污染问题。

## 待验证

- [ ] 估算目标代码模型在本地 GPU 上的显存与训练时间。
- [ ] 设计按仓库和时间切分的数据集，避免同源函数泄漏。

## 下一步

先建立 zero-shot 与 few-shot 基线，再只微调一个窄任务模块。
