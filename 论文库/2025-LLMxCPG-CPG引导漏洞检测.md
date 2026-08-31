---
schema_version: 1
type: source
research_direction: LLM程序分析与软件安全
research_type: 应用型
title: LLMxCPG：代码属性图引导的上下文感知漏洞检测
english_title: "LLMxCPG: Context-Aware Vulnerability Detection Through Code Property Graph-Guided Large Language Models"
authors: [Ahmed Lekssays, Hamza Mouhcine, Khang Tran, Ting Yu, Issa Khalil]
venue: 34th USENIX Security Symposium
publication_date: 2025-08-15
doi: ""
official_url: https://www.usenix.org/conference/usenixsecurity25/presentation/lekssays
code_url: ""
local_pdf: "[[附件/论文原文/08-LLMxCPG-英文原文-USENIX-Security-2025.pdf]]"
status: unread
publication_status: 正式发表
domain: [软件安全, 程序分析, 大语言模型]
parent: "[[README]]"
related: ["[[2025-IRIS-LLM辅助静态漏洞分析]]", "[[2024-LongLLMLingua-长上下文提示压缩]]", "[[2026-BugAuditor-不一致防御代码审计]]"]
sources: [https://www.usenix.org/conference/usenixsecurity25/presentation/lekssays]
created: 2026-08-18
updated: 2026-08-31
category: [代码属性图, 上下文裁剪, 漏洞检测]
relevance_to_DIULENS: 极强
difficulty: 困难
review_status: 精读候选
reading_status: 未读
priority: P1
progress: 0
read_minutes: 0
presentation_ready: false
demo_status: 未计划
next_action: 阅读CPG切片算法和跨函数实验
tags: [研究/LLM程序分析, 方法/静态分析, 方法/上下文压缩]
---

# LLMxCPG：代码属性图引导的上下文感知漏洞检测

> [!abstract] 一句话结论
> LLMxCPG 用代码属性图（Code Property Graph，CPG）构造漏洞相关切片，以更少代码上下文保留控制流和数据依赖，再交给 LLM 检测漏洞。

## 为什么值得读

它直接对应“如何节省 token 又不丢掉安全关键上下文”，比按自然语言重要性删除代码更符合程序语义。

## 来源事实

- CPG 切片使输入代码规模减少 67.84%–90.93%。
- 论文报告相对已有基线的 F1 提升为 15%–40%。
- 实验同时覆盖函数级与多函数代码，并测试了简单语法修改下的鲁棒性。

## 我的理解

这篇论文可作为“安全感知上下文压缩”的核心基线：压缩目标不应只是最短文本，而应最大化漏洞相关数据流、控制依赖和调用关系的保留率。

## 局限性

CPG 质量会受到语言前端、指针分析、动态分派和反编译结果影响；切片减少 token 不等于自动消除 LLM 误报。

## 待验证

- [ ] 核对开源实现、数据集划分与是否存在训练数据污染控制。
- [ ] 比较 CPG 切片与普通 RAG、固定窗口、LongLLMLingua 的成本—召回曲线。

## 下一步

提取其切片输入格式，设计一个移动隐私 source—sink 小型对照实验。
