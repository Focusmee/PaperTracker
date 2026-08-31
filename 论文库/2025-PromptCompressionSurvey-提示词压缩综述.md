---
schema_version: 1
type: source
research_direction: LLM效率与适配
research_type: 综述型
title: 大语言模型提示词压缩综述
english_title: "Prompt Compression for Large Language Models: A Survey"
authors: [Zongqian Li, Yinhong Liu, Yixuan Su, Nigel Collier]
venue: NAACL 2025 Main Conference
publication_date: 2025-04
doi: 10.18653/v1/2025.naacl-long.368
official_url: https://aclanthology.org/2025.naacl-long.368/
code_url: ""
status: unread
publication_status: 正式发表
domain: [自然语言处理, 大语言模型]
parent: "[[README]]"
related: ["[[2024-LongLLMLingua-长上下文提示压缩]]", "[[2026-BRIEFPro-通用上下文压缩]]", "[[2025-LLMxCPG-CPG引导漏洞检测]]"]
sources: [https://aclanthology.org/2025.naacl-long.368/]
created: 2026-08-18
updated: 2026-08-31
category: [提示词压缩, 上下文管理, 综述]
relevance_to_DIULENS: 强
difficulty: 进阶
review_status: 方法综述
reading_status: 未读
priority: P1
progress: 0
read_minutes: 0
presentation_ready: false
demo_status: 未计划
next_action: 整理hard与soft compression方法表
tags: [方法/上下文压缩, 综述/大语言模型]
---

# 大语言模型提示词压缩综述

> [!abstract] 一句话结论
> 综述将提示压缩分为 hard prompt 和 soft prompt 两类，并从注意力、参数高效微调、多模态和合成语言等角度解释其机制与应用。

## 为什么值得读

这是进入 token 节省研究的首选总览，可帮助区分删除/摘要原文本与学习连续压缩表示两条路线。

## 来源事实

- 论文系统比较 hard compression 与 soft compression 技术。
- 讨论下游适配、压缩编码器、软硬混合方法和多模态方向。
- 论文发表于 NAACL 2025 Main。

## 我的理解

代码安全更看重依赖关系和证据保真，因此应在通用分类上增加“程序结构引导压缩”，并单独测量安全信息保留率。

## 局限性

综述覆盖面广但不能替代具体方法的任务级验证，尤其不能保证自然语言压缩器适合代码数据流。

## 待验证

- [ ] 从综述表格提取开源、无需训练且适合长代码的候选方法。
- [ ] 建立输入 token、输出 token、KV Cache 三类优化的术语边界。

## 下一步

先读分类图，再选择 LongLLMLingua、BRIEF-Pro 和 LLMxCPG 做三类代表方法。
