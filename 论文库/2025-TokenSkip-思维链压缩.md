---
schema_version: 1
type: source
research_direction: LLM效率与适配
research_type: 应用型
title: TokenSkip：可控的思维链压缩
english_title: "TokenSkip: Controllable Chain-of-Thought Compression in LLMs"
authors: [Heming Xia, Chak Tou Leong, Wenjie Wang, Yongqi Li, Wenjie Li]
venue: EMNLP 2025 Main Conference
publication_date: 2025-11
doi: 10.18653/v1/2025.emnlp-main.165
official_url: https://aclanthology.org/2025.emnlp-main.165/
code_url: ""
status: unread
publication_status: 正式发表
domain: [自然语言处理, 大语言模型]
parent: "[[README]]"
related: ["[[2025-PromptCompressionSurvey-提示词压缩综述]]", "[[2025-SAN2PATCH-树式推理漏洞修复]]"]
sources: [https://aclanthology.org/2025.emnlp-main.165/]
created: 2026-08-18
updated: 2026-08-31
category: [思维链, 输出token, 推理效率]
relevance_to_DIULENS: 中
difficulty: 困难
review_status: 方法阅读
reading_status: 未读
priority: P2
progress: 0
read_minutes: 0
presentation_ready: false
demo_status: 未计划
next_action: 阅读token重要性分析与压缩训练方法
tags: [方法/推理压缩, 方法/高效推理]
---

# TokenSkip：可控的思维链压缩

> [!abstract] 一句话结论
> TokenSkip 训练模型跳过思维链中较不重要的 token，从输出侧降低推理长度，而不是压缩输入上下文。

## 为什么值得读

它有助于区分“减少输入代码 token”和“减少模型推理 token”两类成本，尤其适合 Tree-of-Thought 等输出迅速膨胀的安全工作流。

## 来源事实

- 在 Qwen2.5-14B-Instruct 的 GSM8K 实验中，推理 token 从 313 降到 181，减少约 40%。
- 该实验性能下降小于 0.4%。
- 方法支持控制思维链压缩程度。

## 我的理解

安全分析需要证据可审计，不能只保留最终标签。理想压缩应删除冗余思考，但保留引用的函数、路径、条件和验证结果。

## 局限性

数学推理上的 token 重要性未必适用于代码漏洞推理；压缩过程还可能隐藏错误推理路径。

## 待验证

- [ ] 测试压缩后安全报告证据字段的完整率。
- [ ] 比较短推理与完整推理在误报解释上的差异。

## 下一步

为安全输出定义不可删除字段，再讨论哪些自然语言推理可以压缩。
