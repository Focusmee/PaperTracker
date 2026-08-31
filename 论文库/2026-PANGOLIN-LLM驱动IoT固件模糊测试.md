---
schema_version: 1
type: source
research_direction: LLM程序分析与软件安全
research_type: 应用型
title: PANGOLIN：LLM驱动的多语言IoT固件模糊测试
english_title: "PANGOLIN: Fuzzing Multilingual IoT Firmware with LLM-Driven Code Analysis"
authors: [Zhipeng Jia, Xiaokang Yin, Shuitao Gan, Chao Zhang, Hangtian Liu, Jiangan Ji, Enzhou Song, Ruijie Cai, Jinglei Tan, Shengli Liu]
venue: 35th USENIX Security Symposium
publication_date: 2026-08-14
doi: ""
official_url: https://www.usenix.org/conference/usenixsecurity26/presentation/jia-zhipeng
code_url: ""
local_pdf: "[[附件/论文原文/12-PANGOLIN-英文原文-USENIX-Security-2026.pdf]]"
status: unread
publication_status: 正式发表
domain: [软件安全, 物联网安全, 大语言模型]
parent: "[[README]]"
related: ["[[2026-BINREX-智能体静态二进制分析]]", "[[2024-PentestGPT-自动化渗透测试]]"]
sources: [https://www.usenix.org/conference/usenixsecurity26/presentation/jia-zhipeng]
created: 2026-08-18
updated: 2026-08-31
category: [LLM程序分析, 固件分析, 模糊测试]
relevance_to_DIULENS: 强
difficulty: 困难
review_status: 精读候选
reading_status: 未读
priority: P1
progress: 0
read_minutes: 0
presentation_ready: false
demo_status: 未计划
next_action: 阅读系统图并整理LLM、静态分析和fuzzer的分工
tags: [研究/LLM程序分析, 方法/模糊测试]
---

# PANGOLIN：LLM驱动的多语言IoT固件模糊测试

> [!abstract] 一句话结论
> PANGOLIN 用 LLM 分析多语言 IoT 固件中的 API 分发和跨语言参数语义，再让反馈驱动的模糊测试负责产生可验证漏洞证据。

## 为什么值得读

它把逆向代码理解、隐藏接口恢复、LLM Agent 和真实漏洞发现连接成完整链路，是从“LLM 看代码”走向“LLM 帮助安全工具探索代码”的代表工作。

## 来源事实

- 系统首先识别隐藏接口，再跨 C、Python、Lua 等语言生成参数规格，并依据响应反馈修正规格。
- 作者在 12 台、8 个厂商的 IoT 设备上发现 68 个此前未知漏洞，其中 31 个获得漏洞编号。
- 论文报告其发现数量为基线 LABRADOR 的 2.96 倍。

## 我的理解

真正有价值的部分不是让 LLM 直接判定漏洞，而是让它补齐传统 fuzzer 最缺的接口语义和结构化输入知识；漏洞成立仍由程序执行、覆盖率和崩溃证据确认。

## 局限性

结果依赖固件可解包程度、反编译质量、接口可触达性和具体设备环境，不能直接推广到任意 IoT 固件。

## 待验证

- [ ] 核对代码与数据集的公开地址及复现实验硬件要求。
- [ ] 精读参数规格纠错模块，确认其 token 成本和失败模式。

## 下一步

与 BINREX 对照画出“语义恢复—任务规划—工具执行—证据验证”四阶段表。
