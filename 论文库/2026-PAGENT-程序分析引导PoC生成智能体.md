---
schema_version: 1
type: source
research_direction: LLM程序分析与软件安全
research_type: 应用型
title: PAGENT：程序分析引导的漏洞PoC生成智能体
english_title: "PAGENT: Program Analysis Guided LLM Agent for Proof-of-Concept Generation"
authors: [Achintya Desai, Md Shafiuzzaman, Wenbo Guo, Tevfik Bultan]
venue: "The ACM SIGSOFT International Symposium on Software Testing and Analysis (ISSTA 2026)"
publication_date:
doi: ""
official_url: "https://conf.researchr.org/details/issta-2026/issta-2026-research-papers/108/PAGENT-Program-Analysis-Guided-LLM-Agent-for-Proof-of-Concept-Generation"
code_url: ""
publication_status: 官方已接收
status: unread
domain: [LLM智能体, 程序分析, 漏洞验证]
parent: "[[README]]"
related:
  - "[[论文库/2026-BINREX-智能体静态二进制分析]]"
  - "[[论文库/2026-PANGOLIN-LLM驱动IoT固件模糊测试]]"
sources:
  - "https://conf.researchr.org/details/issta-2026/issta-2026-research-papers/108/PAGENT-Program-Analysis-Guided-LLM-Agent-for-Proof-of-Concept-Generation"
created: 2026-08-28
updated: 2026-08-31
tags: [方法/LLM-Agent, 方法/静态分析, 方法/动态分析]
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
next_action: 用25分钟把官方摘要拆成“静态指导—动态反馈—Agent生成—PoC验证”四格，并为每格写证据类型
---

# PAGENT：程序分析引导的漏洞PoC生成智能体

> [!abstract] 一句话结论
> PAGENT 不让 LLM Agent 盲猜触发漏洞的输入，而用轻量规则静态分析提供代码指导，再用 sanitizer、运行剖析和覆盖率提供动态反馈，迭代生成能实际触发候选漏洞的 PoC。

## 为什么与当前研究相关

### 来源事实

ISSTA 2026 官方详情页已公开题目、作者与摘要。论文研究已知源码和候选漏洞位置条件下，怎样自动生成可靠触发漏洞的 Proof of Concept（PoC）输入。

### 我的理解

它最适合补充当前 Agent Demo 的“验证闭环”：LLM 负责根据静态和动态提示修改输入，sanitizer与覆盖率提供确定性反馈。这个结构可迁移到合规测试，但论文自身不是移动隐私研究。

### 与 DIULENS / PICOSCAN 的关系

- 与 DIULENS：两者都应把模型动作接到确定性反馈；PAGENT看漏洞是否被输入触发，DIULENS看同意场景是否满足规则。
- 与 PICOSCAN：PICOSCAN把SDK隐私知识变成代码规则；PAGENT用规则静态分析为Agent缩小搜索范围。
- 与 BINREX/PANGOLIN：共同体现“LLM规划或生成，程序分析与执行工具提供证据”。

## 研究问题、输入与输出

- 问题：给定源码和候选漏洞位置，怎样以更高成功率自动生成可复现PoC？
- 输入：软件项目源码、候选漏洞代码位置，以及迭代过程中的静态/动态反馈。
- 输出：能触发候选漏洞的PoC输入，或未成功的分析轨迹。
- 边界：PoC生成以既定候选位置为起点，不等同于从零发现任意漏洞。

## 方法工作链路与分工

```text
源码 + 候选漏洞位置
→ 轻量规则静态分析提供结构指导
→ LLM Agent生成/修改PoC输入
→ sanitizer剖析 + 覆盖率提供动态反馈
→ Agent继续迭代
→ 实际触发并保存PoC
```

| 部分 | 当前可确认职责 | 证据边界 |
|---|---|---|
| 静态分析 | 轻量、规则化分析，为 Agent 提供指导 | 规则种类、精度和实现待正文核实 |
| 动态分析 | sanitizer剖析与覆盖信息反馈执行结果 | 只有真正运行到的路径产生反馈 |
| LLM / Agent | 根据两类指导生成和修正 PoC | 模型文本本身不是漏洞证据 |
| 知识库 | 官方摘要未报告独立知识库 | 不擅自推断其保存漏洞模板 |
| 人工验证 | 摘要未公开完整流程 | PoC稳定性、去重和漏洞真实性待正文核实 |

## 零基础术语与例子

- **PoC（Proof of Concept，概念验证）**：能稳定展示问题确实发生的最小输入。例如一段特定 JSON 让解析器越界。
- **Sanitizer（运行时检测器）**：程序执行时监控越界、释放后使用等错误，并给出崩溃位置。
- **Coverage（覆盖率）**：测试执行过哪些代码。例：输入距离漏洞分支更近时覆盖了新的基本块。
- **静态指导**：不运行程序，从代码结构提取到达目标可能需要的条件。

## 推荐优先阅读

1. 官方摘要：明确输入是“源码+候选位置”，不是完全未知程序。
2. 正文公开后先看系统总览图和Agent每轮收到的提示格式。
3. 再看静态规则、sanitizer反馈、覆盖率反馈的消融实验。
4. 最后核对基线、成功判定、时间预算和失败案例。

## 已核验的数据集、基线、指标和关键数字

- 官方摘要报告：PAGENT 在 PoC 生成任务上，相对此前表现最好的 Agent 方法提升132%。
- 摘要未公开样本数量、绝对成功率、漏洞类型和运行预算，均待正文核实。

> [!warning] 正确解读
> “提升132%”是相对提升，不等于成功率为132%，也不能在缺少绝对分母时判断实际成功了多少样本。

## 局限和复现条件

### 来源事实

当前只访问了官方详情页和摘要，未访问正文、代码或数据。

### 待验证

- [ ] DOI、正式出版日期、正文与代码链接。
- [ ] 数据集规模、漏洞类型、基线名称、绝对成功率和时间预算。
- [ ] 静态分析规则、动态反馈格式、Agent模型和重试停止条件。
- [ ] PoC是否在独立环境重放，以及人工 Ground Truth 如何建立。

## 主动回忆问题

1. 为什么给定候选漏洞位置仍不容易生成PoC？
2. 静态指导与覆盖率反馈分别解决什么问题？
3. sanitizer报告为什么比LLM自然语言解释更接近执行证据？
4. “提升132%”缺少哪些信息就不能完整解读？
5. 如何把这种闭环迁移到CMP场景测试而不混淆漏洞与合规？

## 下一步

用25分钟画出四格闭环，并为每一格标注“模型推断、静态事实、动态事实或待人工确认”。

## 一手来源

- [ISSTA 2026 官方论文详情页](https://conf.researchr.org/details/issta-2026/issta-2026-research-papers/108/PAGENT-Program-Analysis-Guided-LLM-Agent-for-Proof-of-Concept-Generation)
