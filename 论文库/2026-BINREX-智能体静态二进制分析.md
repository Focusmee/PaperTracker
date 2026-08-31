---
schema_version: 1
type: source
research_direction: LLM程序分析与软件安全
research_type: 应用型
title: BINREX：任务自适应的智能体静态二进制分析
english_title: "Towards Generality: Task-Adaptive Binary Analysis via Semantic Retrieval and Verifiable Reasoning"
authors: [Yuzhe Liu, Zhijie Liu, Zhengmin Yu, Shu Wang, Ling Jiang, Sen Nie, Shi Wu, Zhanyong Tang, Yuan Zhang]
venue: 35th USENIX Security Symposium
publication_date: 2026-08-14
official_url: https://www.usenix.org/conference/usenixsecurity26/presentation/liu-yuzhe
pdf_url: https://www.usenix.org/system/files/usenixsecurity26-liu-yuzhe.pdf
code_url: ""
local_pdf: "[[附件/论文原文/11-BINREX-英文原文-USENIX-Security-2026.pdf]]"
status: unread
publication_status: 正式发表
domain: [二进制分析, 程序分析, 大语言模型]
parent: "[[README]]"
related: ["[[论文库/2024-LLM4Decompile-大模型反编译]]", "[[论文库/2026-PAGENT-程序分析引导PoC生成智能体]]"]
sources: [https://www.usenix.org/conference/usenixsecurity26/presentation/liu-yuzhe, https://www.usenix.org/system/files/usenixsecurity26-liu-yuzhe.pdf]
created: 2026-08-16
updated: 2026-08-31
category: [LLM Agent, 静态二进制分析, 可验证推理]
relevance_to_DIULENS: 强
difficulty: 困难
review_status: 精读候选
reading_status: 未读
priority: P1
progress: 0
read_minutes: 0
started_date:
last_read:
finished_date:
rating:
tags: [方法/静态分析, 方法/LLM-Agent]
presentation_ready: false
demo_status: 未计划
next_action: 阅读图1、图2和第3节，写出LLM与IDA各自负责什么
tags: [USENIX-Security-2026, LLM-Agent, static-analysis]
---

# BINREX：任务自适应的智能体静态二进制分析

> [!abstract] 一句话结论
> BINREX 不让 LLM 直接看反编译代码猜答案，而用 5700 万函数语义检索帮助定位、分层规划拆任务，再生成并执行 IDAPython，以可检查的地址、调用图和数据流证据约束结论。

## 为什么值得读

它回答“LLM 在程序分析中负责什么”：LLM 做意图理解、规划和工具调用；IDA/脚本产生确定事实；验证器检查证据。Agent→Tool→Evidence→Repair 可迁移到 DIULENS Demo。

## 研究问题、输入与输出

- 问题：无符号二进制中能否处理“找后门”等开放任务，而不限固定 CWE？
- 输入：IDA Pro 可加载、已解包的 stripped binary 和自然语言意图。
- 输出：结构化报告、地址、CFG 路径、交叉引用和数据流制品。
- 假设：LLM 遵循指令但会出错；VM 混淆、打包和必须运行的任务不在范围。

## 工作链路与分工

1. 从 57M 开源函数为 sub_xxx 检索名称/摘要提示。
2. 将“找认证后门”拆为定位认证、找魔数、提取绕过路径等子任务。
3. 为每个子任务定义完成条件和证据。
4. LLM 生成结构化执行表示与 IDAPython。
5. IDA 确定性执行，产生地址、XRef、CFG 和数据流。
6. 验证器检查谓词，失败则诊断、重写、重跑。
7. 汇总带证据报告。

- LLM：规划、代码合成、失败修复。
- 静态工具：确定性执行。
- 知识库：57M 函数向量；分析策略、工具模式、安全行为签名。
- Oracle：局部验证谓词和 BinREval Ground Truth。
- 动态分析：非核心，运行依赖任务明确范围外。

## 零基础术语

- stripped binary：删除函数名/调试符号，check_password 可能只剩 sub_40F20。
- IDAPython：在 IDA 中查询函数、字符串、调用关系的脚本。
- 语义检索：取回可能的名称/摘要，只是导航提示，不是漏洞证据。
- 验证谓词：如“必须给出魔数地址及到成功返回的 CFG 路径”。

## 实验与关键数字

- BinREval：72项任务、7类COTS安全场景。
- 成功率 83.3%；Codex 27.8%；Codex+IDA 43.1%。
- 总分析时间 87.9 小时降到 36.1 小时。
- 工业部署识别 395 个此前未知恶意软件样本，平均效率相对专家提升 96×。
- 2026-04价格：GPT-5平均299K tokens、约$4.50/任务。
- 双编码器预训练约8×A100一周；57M索引约27GB。

> [!warning] 正确解读
> 83.3% 是作者72项任务的成功率，不是任意漏洞准确率；395个样本也不能推广到所有恶意软件。

## 精读顺序

图1总览 → 图2语义检索 → §2.5–2.7边界 → §3计划/执行/验证 → 实验总表和消融图10 → §6局限。

## 局限与复现

受 IDA 反编译质量限制；依赖商业 IDA；打包/VM 混淆需外部处理；竞态、时序和环境触发不适用。正式页面暂未给代码仓库；论文称公开 BinREval artifact，复现前需继续核验。

## 论文关系

- [[2025-IRIS-LLM辅助静态漏洞分析]]：LLM补Source/Sink、CodeQL跑数据流；BINREX是多步Agent。
- [[GPTDroid]]：前者工具是Appium，本文工具是IDA。
- [[DIULENS]]：可把找到CMP、点击拒绝、确认Hook参数定义为带谓词的子任务。

## 三分钟讲解

符号剥离后只有sub_xxx，LLM盲读易幻觉；57M检索提供语义地图；LLM生成IDAPython、IDA执行、验证失败就修复；83.3%对比27.8%/43.1%说明工具和Oracle比单纯Prompt关键；移动分析中Appium/Hook/扫描器应负责事实。

## 最小 Demo

使用带硬编码后门的小型C程序，预先导出调用图/字符串表；Agent输出JSON子任务；Python查询XRef；验证器要求地址和路径同时存在。比较“LLM直接答、LLM+工具、LLM+工具+验证”。

## 思考题

1. 语义检索会否误导Agent？
2. 验证谓词由谁设计，Oracle不完整怎么办？
3. 可检查证据不能消除哪些语义误判？
4. Android APK中IDA应替换成哪些工具？
5. 是否还应报告误报、漏报和人工复核时间？
6. 知识库污染或API过期会产生什么系统误差？

## 下一步

写“模块—输入—输出—谁验证”四列表，确保能解释 LLM 不是漏洞 Oracle。

## 正式来源

- [USENIX页面](https://www.usenix.org/conference/usenixsecurity26/presentation/liu-yuzhe)
- [正式PDF](https://www.usenix.org/system/files/usenixsecurity26-liu-yuzhe.pdf)
