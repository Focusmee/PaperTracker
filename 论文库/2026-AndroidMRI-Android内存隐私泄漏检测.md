---
schema_version: 1
type: source
research_direction: 移动隐私与软件供应链
research_type: 应用型
title: ANDROID-MRI：Android应用内存敏感数据泄漏检测
english_title: "Ghosts in the Memory: Detecting Unintended Sensitive Data in Android Apps"
authors: [Seonghyeon Song, Taeyoung Kim, Woojoo Kim, Seojin Park, Sungjae Hwang, Hyoungshick Kim]
venue: "The ACM SIGSOFT International Symposium on Software Testing and Analysis (ISSTA 2026)"
publication_date:
doi: ""
official_url: "https://conf.researchr.org/details/issta-2026/issta-2026-research-papers/107/Ghosts-in-the-Memory-Detecting-Unintended-Sensitive-Data-in-Android-Apps"
code_url: ""
publication_status: 官方已接收
status: unread
domain: [移动应用隐私, Android动态分析, 内存数据流]
parent: "[[README]]"
related:
  - "[[论文库/2026-HealthHazard-HealthConnect隐私风险]]"
  - "[[论文库/2026-HScope-OpenHarmony细粒度隐私泄漏检测]]"
sources:
  - "https://conf.researchr.org/details/issta-2026/issta-2026-research-papers/107/Ghosts-in-the-Memory-Detecting-Unintended-Sensitive-Data-in-Android-Apps"
created: 2026-08-28
updated: 2026-08-31
tags: [研究/移动隐私, 方法/动态分析, 方法/程序分析]
reading_status: 未读
priority: P1
progress: 0
read_minutes: 0
started_date:
last_read:
finished_date:
rating:
presentation_ready: false
demo_status: 未计划
next_action: 用25分钟阅读官方摘要并画出“内核插桩—指令级传播—内存泄漏候选—厂商确认”四步证据链
---

# ANDROID-MRI：Android应用内存敏感数据泄漏检测

> [!abstract] 一句话结论
> ANDROID-MRI 把追踪逻辑放进 Android 内核，以指令粒度监控敏感数据在内存中的传播，从而绕过部分应用层反分析机制，并发现快照式工具容易漏掉的短暂或残留敏感数据。

## 为什么与当前研究相关

### 来源事实

ISSTA 2026 官方详情页已公开题目、作者和完整摘要。论文针对 Android App 在内存中意外保留共用账户数据、认证令牌和密钥的问题，提出操作系统级动态分析工具 ANDROID-MRI。

### 我的理解

它为当前 DIULENS/PICOSCAN 主线补充了一个不同证据层：界面轨迹和 SDK API 之外，敏感值还可能在进程内存中被不必要地保留或传播。它也展示了为什么普通 Frida Hook 不是动态分析的完整 Ground Truth。

### 与 DIULENS / PICOSCAN 的关系

- DIULENS 检查同意界面、操作路径和运行事件；ANDROID-MRI 检查内存中的敏感数据传播，两者观察层不同。
- PICOSCAN/HScope 风格静态分析给出可能路径；ANDROID-MRI 提供特定执行中的指令级运行证据。
- 可迁移点是保留“数据值—指令/地址—时间—调用上下文”的证据，而不是只记录一次 API 名称。

## 研究问题、输入与输出

- 研究问题：怎样在应用存在 RASP/反插桩、原始内存缺少高级语义、敏感值短暂出现的条件下检测内存泄漏？
- 输入：运行中的 Android App、敏感数据及其内存传播。
- 输出：敏感数据在内存中的传播与意外持久化上下文，以及待人工确认的问题。
- 威胁边界：官方摘要关注应用内存中的未预期暴露，不是网络外传检测，也不能自动等同于法律违规。

## 方法工作链路与分工

```text
Android App运行
→ 内核级插桩观察指令与内存
→ 跟踪敏感数据的传播和残留
→ 形成带上下文的候选披露
→ 厂商/人工确认与修复
```

| 部分 | 当前可确认职责 | 证据边界 |
|---|---|---|
| 静态分析 | 官方摘要未说明是核心组成 | 待正文核实是否使用静态预筛或符号恢复 |
| 动态分析 | Android 内核中的指令粒度传播监控 | 只覆盖实际执行；完整跟踪语义待正文核实 |
| LLM / Agent | 官方摘要未报告使用 | 不应擅自补入系统链路 |
| 知识库 | 摘要未公开具体数据类型/规则库 | 原始字节如何映射到高级敏感语义待核实 |
| 人工验证 | 厂商确认、补丁和漏洞奖励提供外部证据 | 厂商确认强于工具候选，但仍需区分问题类型 |

## 零基础术语与例子

- **RASP（Runtime Application Self-Protection，运行时应用自保护）**：App 在运行中检测 Frida 等插桩并阻止分析。例如金融 App 发现 Hook 后立即退出。
- **内存残留**：敏感值完成用途后仍留在内存。例：切换账号后，旧账号 PIN 仍能从进程内存读取。
- **指令粒度追踪**：记录具体机器指令怎样读写和传播数据，比偶尔截取内存快照更连续。
- **语义鸿沟**：内存里只有字节，分析器需要判断这些字节代表令牌、密钥还是普通缓存。

## 推荐优先阅读

1. 官方摘要：先理解三个挑战和内核级思路。
2. 正文公开后优先看总览图、威胁模型和敏感数据标记方式。
3. 再看 RASP 对比实验和17项披露的确认标准。
4. 最后看性能开销、内核版本、设备要求和伦理披露。

## 已核验的数据集、基线、指标和关键数字

- 评估50个安装量不少于100万的热门 App。
- 官方摘要报告 ANDROID-MRI 绕过97.6%的 RASP 保护，对比 Frida 为52.4%。
- 发现17项此前未知的内存披露，涉及资料 PIN、订阅内容和加密密钥等。
- 12家厂商确认问题，4家已修复，3家通过漏洞奖励计划确认。

> [!warning] 正确解读
> 97.6% 是论文特定 RASP 测试设置中的绕过比例，不是隐私泄漏检测准确率；17项是工具发现并进入厂商确认流程的问题，不代表50个 App 的普遍泄漏率。

## 局限和复现条件

### 来源事实

官方摘要只公开了核心挑战、方法和主要结果，尚未给出完整复现条件。

### 待验证

- [ ] 正文、DOI、代码和数据是否已经公开。
- [ ] 支持的 Android/内核、CPU架构、Root或刷机要求及运行开销。
- [ ] 敏感数据初始标记、传播规则、误报/漏报和 Ground Truth 建立方式。
- [ ] 97.6%与52.4%的测试样本、分母和RASP类别。

## 主动回忆问题

1. 为什么 Frida 看不到不等于内存中没有敏感数据？
2. 快照式扫描为什么容易漏掉短暂出现的值？
3. 内核级追踪如何绕过应用层 RASP，又会带来什么部署成本？
4. 17项内存披露为什么不能直接说成17项法律违规？
5. 怎样把 ANDROID-MRI 的证据接到 DIULENS 的同意时间线？

## 下一步

用25分钟把“RASP、语义鸿沟、短暂数据”三个挑战分别对应到 ANDROID-MRI 的设计选择；正文未打开前不补写系统细节。

## 一手来源

- [ISSTA 2026 官方论文详情页](https://conf.researchr.org/details/issta-2026/issta-2026-research-papers/107/Ghosts-in-the-Memory-Detecting-Unintended-Sensitive-Data-in-Android-Apps)
