---
schema_version: 1
type: source
research_direction: LLM效率与适配
research_type: 探索型
title: 什么构成高质量自然语言提示词
english_title: "What Makes a Good Natural Language Prompt?"
authors: [Do Xuan Long, Duy Dinh, Ngoc-Hai Nguyen, Kenji Kawaguchi, Nancy F. Chen, Shafiq Joty, Min-Yen Kan]
venue: ACL 2025 Main Conference
publication_date: 2025-07
doi: 10.18653/v1/2025.acl-long.292
official_url: https://aclanthology.org/2025.acl-long.292/
code_url: ""
status: unread
publication_status: 正式发表
domain: [自然语言处理, 大语言模型]
parent: "[[README]]"
related: ["[[2025-OptimizedPrompts-优化提示词机制]]", "[[2025-APPATCH-自适应提示漏洞修复]]"]
sources: [https://aclanthology.org/2025.acl-long.292/]
created: 2026-08-18
updated: 2026-08-31
category: [提示词设计, 元分析, 提示词评估]
relevance_to_DIULENS: 中强
difficulty: 进阶
review_status: 方法阅读
reading_status: 未读
priority: P2
progress: 0
read_minutes: 0
presentation_ready: false
demo_status: 未计划
next_action: 阅读六维二十一属性框架和研究空白
tags: [方法/提示词, 方法/评测]
---

# 什么构成高质量自然语言提示词

> [!abstract] 一句话结论
> 论文从 150 多篇研究中归纳出 6 个维度、21 种提示词属性，并指出不同属性的效果依赖模型和任务，叠加更多技巧并不必然更好。

## 为什么值得读

它提供了设计和报告安全分析 prompt 时可复用的属性框架，适合把“提示词经验”转化为可控制变量的实验。

## 来源事实

- 元分析覆盖 2022–2025 年 NLP/AI 会议论文及头部技术机构材料。
- 作者提出以属性和人为中心的提示词质量框架。
- 推理任务案例显示，增强单个属性常常比同时增强多个属性更有效。

## 我的理解

研究中应固定模型与输入代码，只改变清晰度、结构、示例、约束和输出格式等单一属性，才能知道性能变化来自哪里。

## 局限性

现有证据在模型、任务和属性之间分布不均，不能把某项建议当成跨模型通用规律。

## 待验证

- [ ] 提取与代码安全任务最相关的属性及其原始支持论文。
- [ ] 建立 prompt 属性—漏洞召回—token 成本三维实验表。

## 下一步

用同一段污点传播代码设计三个仅改变一个属性的 prompt。
