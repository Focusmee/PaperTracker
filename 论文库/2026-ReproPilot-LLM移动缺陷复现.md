---
schema_version: 1
type: source
research_direction: 移动隐私与软件供应链
research_type: 应用型
title: ReproPilot：全局状态重排与LLM轨迹探索的移动缺陷复现
english_title: "Mobile Bug Reproduction via Global State Reprioritization and LLM-Guided Trajectory Exploration"
authors: [Dingbang Wang, Sidong Feng, William G. J. Halfond, Tingting Yu]
venue: "41st IEEE/ACM International Conference on Automated Software Engineering (ASE 2026)"
publication_date:
doi: "10.1145/3832783.3837495"
official_url: "https://conf.researchr.org/details/ase-2026/ase-2026-research-track/199/Mobile-Bug-Reproduction-via-Global-State-Reprioritization-and-LLM-Guided-Trajectory-E"
code_url: ""
publication_status: 官方已接收（DOI已分配）
status: unread
domain: [移动GUI测试, LLM智能体, 缺陷复现]
parent: "[[README]]"
related:
  - "[[论文库/2026-ScenGen-场景引导移动GUI测试]]"
  - "[[论文库/2026-MATE-移动智能体策略感知安全审计]]"
sources:
  - "https://conf.researchr.org/details/ase-2026/ase-2026-research-track/199/Mobile-Bug-Reproduction-via-Global-State-Reprioritization-and-LLM-Guided-Trajectory-E"
  - "https://conf.researchr.org/track/ase-2026/ase-2026-not-in-person-presentations"
created: 2026-08-29
updated: 2026-08-31
tags: [方法/LLM-Agent, 方法/动态分析, 方法/移动GUI测试]
reading_status: 未读
priority: P2
progress: 0
read_minutes: 0
started_date:
last_read:
finished_date:
rating:
presentation_ready: false
demo_status: 未计划
next_action: 用25分钟把ReproPilot的“长程LLM指导—启发式信号—全局状态重排—轨迹探索”画成闭环，并与ScenGen逐步决策对照
---

# ReproPilot：全局状态重排与LLM轨迹探索的移动缺陷复现

> [!abstract] 一句话结论
> ReproPilot 不在每个页面只执行一次 LLM 建议，而让模型一次生成可复用的长程指导，再与启发式信号结合，对所有已发现 GUI 状态进行全局重排，从更有希望的状态继续探索，以降低不稳定性和 token 成本。

## 为什么与当前研究相关

### 来源事实

ASE 2026 官方详情页已核验题目、作者和接收状态；官方 Not-in-Person 页面进一步公开 DOI、摘要和关键数字。论文面向 Android 崩溃报告的自动复现，系统名为 ReproPilot。

### 我的理解

它可以直接帮助改进 DIULENS/ScenGen 风格 Demo 的路径规划：LLM 不必每走一步都重新理解整个界面，而可给出若干可复用的长程方向；探索器负责在全局状态图中选择下一处继续测试。它解决的是移动缺陷复现，不直接判断隐私合规。

### 与 DIULENS / PICOSCAN 的关系

- 与 DIULENS：可用于更稳定地找到深层 CMP/撤回入口，但最终 DIU 规则仍需截图、信号和时间事件。
- 与 ScenGen：ScenGen强调场景驱动的观察、决策、执行、监督与记录；ReproPilot强调跨状态的全局优先级和一次查询产生多条可复用指导。
- 与 PICOSCAN：ReproPilot不解析SDK调用或隐私API，不能替代供应链静态分析。

## 研究问题、输入与输出

- 问题：怎样减少 LLM 单步输出的不稳定性和频繁查询成本，同时保持移动崩溃复现效果？
- 输入：Android 崩溃报告、当前与历史 GUI 状态、LLM 长程指导和启发式信号。
- 输出：触发报告缺陷的操作轨迹，以及复现成功/失败记录。
- 威胁边界：这是软件测试与缺陷复现系统，不是 APK 漏洞扫描器或合规裁判。

## 方法工作链路与分工

```text
崩溃报告 + 当前应用
→ LLM生成多条可复用长程指导
→ 探索器记录GUI状态与轨迹
→ 启发式信号评估复现进展
→ 全局规划器重排所有候选状态
→ 从更有希望的状态继续探索
→ 触发崩溃并保存复现轨迹
```

| 部分 | 当前可确认职责 | 证据边界 |
|---|---|---|
| 静态分析 | 官方摘要未报告为核心 | 不应推断其分析 APK 调用图 |
| 动态分析 | 实际探索 Android GUI 状态和轨迹，观察崩溃是否触发 | 只覆盖到达的状态 |
| LLM / Agent | 提供多样、可复用、长程的语义指导 | 指导可能错误，需与状态和启发式信号结合 |
| 知识/记忆 | 保存全局状态、轨迹和可重复利用的模型输出 | 数据结构与去重算法待正文核实 |
| 人工验证 | 真实崩溃报告提供任务目标；完整 Ground Truth 流程待核实 | 复现成功应能重放，而非只依赖模型陈述 |

## 零基础术语与例子

- **Bug Reproduction（缺陷复现）**：把文字报告转成能再次触发同一问题的操作序列。例如“打开设置→切换主题→返回”后 App 崩溃。
- **Global State Reprioritization（全局状态重排）**：不是只看当前页面，而是给所有访问过但仍可探索的页面重新排序。
- **Long-horizon Guidance（长程指导）**：一次模型调用给出多步或多个方向，之后可反复使用。例如“先登录，再进入设置，主题切换可能触发问题”。
- **Heuristic Signal（启发式信号）**：无需模型即可计算的线索，例如页面文字与崩溃报告关键词的相似度。

## 推荐优先阅读

1. 官方摘要：抓住“单步LLM建议”的两个问题和全局规划方案。
2. 正文公开后先看总览图、GUI状态表示和重排公式。
3. 再看 LLM 一次生成多少指导、何时重新查询、失败怎样回退。
4. 最后看74个报告的基线、重复运行设置和成本统计。

## 已核验的数据集、基线、指标和关键数字

- 在74个真实 Android 崩溃报告上评估。
- 官方摘要报告一致性为80.14%，有效性为86.49%。
- 相比所用 LLM 基线，复现时间降低2.32%–14.07%。
- token 消耗降低65.39%–83.96%。

> [!warning] 正确解读
> 一致性、有效性和成本是不同指标；86.49%不能写成漏洞检测准确率。时间和 token 的下降范围取决于具体对比基线，需回正文核对每组分母与重复次数。

## 局限和复现条件

### 来源事实

当前访问了 ASE 官方详情页、官方摘要和 DOI 信息，尚未实际打开正式正文、代码或数据。

### 待验证

- [ ] 正式出版日期、PDF、代码与74个崩溃报告数据集链接。
- [ ] 一致性与有效性的精确定义、重复运行次数和统计方法。
- [ ] 全局状态图、状态相似/去重、回退和崩溃 Oracle 的实现。
- [ ] 使用的模型、token 计价、设备、App版本和平均运行时。

## 主动回忆问题

1. 为什么每一步都只听一次 LLM 建议会不稳定？
2. 长程指导怎样减少 token，又可能引入什么过时风险？
3. 全局重排与普通深度优先/广度优先探索有什么区别？
4. 为什么触发崩溃比“模型认为已完成”更强？
5. 如何把崩溃 Oracle 替换成 DIULENS Demo 的 CMP 完成条件？

## 下一步

用25分钟画两条流程：`单步LLM→点击→再问LLM` 与 `一次长程指导→全局状态重排→多次探索`，分别标注模型调用次数、可验证状态和可能失败点。

## 一手来源

- [ASE 2026 官方论文详情页](https://conf.researchr.org/details/ase-2026/ase-2026-research-track/199/Mobile-Bug-Reproduction-via-Global-State-Reprioritization-and-LLM-Guided-Trajectory-E)
- [ASE 2026 官方摘要与DOI页](https://conf.researchr.org/track/ase-2026/ase-2026-not-in-person-presentations)
