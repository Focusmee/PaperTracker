---
schema_version: 1
type: source
research_direction: LLM效率与适配
research_type: 应用型
title: SnapKV：长上下文大模型的KV缓存压缩
english_title: "SnapKV: LLM Knows What You are Looking for Before Generation"
authors: [Yuhong Li, Yingbing Huang, Bowen Yang, Bharat Venkitesh, Acyr Locatelli, Hanchen Ye, Tianle Cai, Patrick Lewis, Deming Chen]
venue: NeurIPS 2024 Main Conference
publication_date: 2024
doi: 10.52202/079017-0722
official_url: https://proceedings.neurips.cc/paper_files/paper/2024/hash/28ab418242603e0f7323e54185d19bde-Abstract-Conference.html
code_url: ""
status: unread
publication_status: 正式发表
domain: [机器学习系统, 大语言模型]
parent: "[[README]]"
related: ["[[2024-LongLLMLingua-长上下文提示压缩]]", "[[2025-PromptCompressionSurvey-提示词压缩综述]]"]
sources: [https://proceedings.neurips.cc/paper_files/paper/2024/hash/28ab418242603e0f7323e54185d19bde-Abstract-Conference.html]
created: 2026-08-18
updated: 2026-08-31
category: [KV Cache, 推理显存, 长上下文]
relevance_to_DIULENS: 中
difficulty: 困难
review_status: 扩展阅读
reading_status: 未读
priority: P3
progress: 0
read_minutes: 0
presentation_ready: false
demo_status: 未计划
next_action: 先明确KV Cache压缩与计费token压缩的区别
tags: [方法/KV缓存, 方法/高效推理]
---

# SnapKV：长上下文大模型的KV缓存压缩

> [!abstract] 一句话结论
> SnapKV 从提示末端观察窗口估计各注意力头关注的位置，再聚类保留重要 KV 项，以更少显存处理长上下文。

## 为什么值得读

它解决的是推理显存和速度，而不是直接减少 API 输入 token；这一区分对实验成本表述非常重要。

## 来源事实

- 对 16K token 输入，论文报告生成速度提高 3.6 倍、显存效率提高 8.2 倍。
- 在 16 个长序列数据集上保持与完整缓存相近的表现。
- 作者报告单张 A100-80GB 可处理最高 380K 上下文，并在 Needle-in-a-Haystack 测试中仅有轻微精度下降。

## 我的理解

SnapKV 可用于本地部署长代码模型，但不会自动降低传给 API 的 token 数，也不能替代代码级检索和切片。

## 局限性

效果依赖模型架构、实现和任务；极长上下文可运行不等于模型能可靠理解全部信息。

## 待验证

- [ ] 核对代码仓库和当前支持的模型架构。
- [ ] 在代码检索任务上测量被压缩 KV 对跨函数依赖的影响。

## 下一步

在研究设计中把输入压缩、输出压缩、KV Cache 压缩分别计量。
