---
schema_version: 1
type: source
research_direction: LLM程序分析与软件安全
research_type: 应用型
title: APPATCH：自适应提示驱动的真实软件漏洞修复
english_title: "APPATCH: Automated Adaptive Prompting Large Language Models for Real-World Software Vulnerability Patching"
authors: [Yu Nong, Haoran Yang, Long Cheng, Hongxin Hu, Haipeng Cai]
venue: 34th USENIX Security Symposium
publication_date: 2025-08-15
doi: ""
official_url: https://www.usenix.org/conference/usenixsecurity25/presentation/nong
code_url: ""
status: unread
publication_status: 正式发表
domain: [软件安全, 大语言模型]
parent: "[[README]]"
related: ["[[2025-SAN2PATCH-树式推理漏洞修复]]", "[[2025-GoodPrompt-自然语言提示词质量]]", "[[2025-OptimizedPrompts-优化提示词机制]]"]
sources: [https://www.usenix.org/conference/usenixsecurity25/presentation/nong]
created: 2026-08-18
updated: 2026-08-31
category: [自适应提示, 漏洞语义, 自动修复]
relevance_to_DIULENS: 强
difficulty: 困难
review_status: 精读候选
reading_status: 未读
priority: P1
progress: 0
read_minutes: 0
presentation_ready: false
demo_status: 未计划
next_action: 阅读漏洞语义推理与自适应提示选择逻辑
tags: [研究/漏洞修复, 方法/提示词]
---

# APPATCH：自适应提示驱动的真实软件漏洞修复

> [!abstract] 一句话结论
> APPATCH 先推理漏洞行为语义，再按中间结果自适应选择提示步骤，在不微调模型且没有测试输入或 exploit 证据的条件下生成补丁。

## 为什么值得读

它把提示词设计嵌入安全任务工作流，适合回答“什么情况下应继续追问、补充哪种上下文”，而不是只给一个固定模板。

## 来源事实

- 评测包含 97 个零日漏洞和 20 个已有漏洞。
- 论文报告相对最佳基线最高提升 28.33% F1 和 182.26% recall。
- 系统强调漏洞语义推理和自适应 prompting，不依赖训练或微调。

## 我的理解

可将其思想迁移到检测：先识别候选风险类型，再有条件地请求调用者、被调用者、数据流或运行证据，按不确定性分配 token。

## 局限性

缺少测试或 exploit 时，补丁正确性验证仍是难点；提示策略可能依赖特定模型行为。

## 待验证

- [ ] 核对零日样本的时间切分和训练数据污染控制。
- [ ] 提取每个提示阶段的输入、输出与停止条件。

## 下一步

用一个缓冲区溢出例子复述自适应提示状态机。
