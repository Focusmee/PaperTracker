---
schema_version: 1
type: source
research_direction: LLM效率与适配
research_type: 应用型
title: LongLLMLingua：长上下文场景的提示压缩
english_title: "LongLLMLingua: Accelerating and Enhancing LLMs in Long Context Scenarios via Prompt Compression"
authors: [Huiqiang Jiang, Qianhui Wu, Xufang Luo, Dongsheng Li, Chin-Yew Lin, Yuqing Yang, Lili Qiu]
venue: ACL 2024 Main Conference
publication_date: 2024-08
doi: 10.18653/v1/2024.acl-long.91
official_url: https://aclanthology.org/2024.acl-long.91/
code_url: https://github.com/microsoft/LLMLingua
status: unread
publication_status: 正式发表
domain: [自然语言处理, 大语言模型]
parent: "[[README]]"
related: ["[[2025-PromptCompressionSurvey-提示词压缩综述]]", "[[2026-BRIEFPro-通用上下文压缩]]", "[[2025-LLMxCPG-CPG引导漏洞检测]]"]
sources: [https://aclanthology.org/2024.acl-long.91/, https://github.com/microsoft/LLMLingua]
created: 2026-08-18
updated: 2026-08-31
category: [长上下文, 问题感知压缩, 位置偏差]
relevance_to_DIULENS: 强
difficulty: 进阶
review_status: 方法阅读
reading_status: 未读
priority: P1
progress: 0
read_minutes: 0
presentation_ready: false
demo_status: 未计划
next_action: 阅读粗到细压缩与文档重排策略
tags: [方法/上下文压缩, 方法/长上下文]
---

# LongLLMLingua：长上下文场景的提示压缩

> [!abstract] 一句话结论
> LongLLMLingua 使用问题感知的粗到细压缩、动态压缩率和文档重排，提高关键证据密度并缓解“关键信息在中间被忽略”的位置偏差。

## 为什么值得读

它是无需改变目标 LLM 的长上下文压缩基线，适合与代码结构切片进行对照。

## 来源事实

- 在 NaturalQuestions 上，论文报告约 4 倍减少 token 的同时，GPT-3.5-Turbo 性能最高提升 21.4%。
- 在 LooGLE 实验中报告最高 94.0% 成本降低。
- 对约 10K token 提示进行 2–6 倍压缩时，端到端延迟加速为 1.4–2.6 倍。

## 我的理解

方法的“问题感知”适合安全查询，但对代码不应只依据语言模型困惑度，还应保护 source、sink、条件分支和跨函数依赖。

## 局限性

摘要中的高收益来自特定数据集与模型；压缩器自身也消耗计算，且删掉的证据不可恢复。

## 待验证

- [ ] 在同一代码样本上比较压缩前后污点路径保留率。
- [ ] 记录压缩器时间，避免只报告目标模型 token 节省。

## 下一步

选一段跨函数隐私数据流，做原文、固定截断、LongLLMLingua、CPG切片四组输入。
