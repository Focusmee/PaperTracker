---
schema_version: 1
type: source
research_direction: 移动隐私与软件供应链
research_type: 评测型
title: 解锁秘密：评估LLM检测Android应用秘密的能力
english_title: "Secrets Unlocked: Evaluating LLMs for Secrets Detection in Android Apps"
authors: [Marco Alecci, Jordan Samhi, Tegawendé F. Bissyandé, Jacques Klein]
venue: ACM SIGSOFT International Symposium on Software Testing and Analysis (ISSTA 2026)
publication_date: 2026-10-03
doi: ""
official_url: https://conf.researchr.org/details/issta-2026/issta-2026-research-papers/49/Secrets-Unlocked-Evaluating-LLMs-for-Secrets-Detection-in-Android-Apps
code_url: ""
publication_status: 官方已接收（待正式出版）
status: unread
domain: [Android程序分析, LLM安全分析, 移动应用安全]
parent: "[[README]]"
related: ["[[论文库/2026-PlainTextPlainRisks-AndroidWebView明文风险]]", "[[论文库/2026-PINFINDER-SDK隐私上下文一致性]]"]
sources: [https://conf.researchr.org/details/issta-2026/issta-2026-research-papers/49/Secrets-Unlocked-Evaluating-LLMs-for-Secrets-Detection-in-Android-Apps, https://jordansamhi.com/static/files/papers/secretloc.pdf, https://jacquesklein2302.github.io/publications/]
created: 2026-08-20
updated: 2026-08-31
tags: [方法/静态分析, 方法/LLM-Agent, 研究/移动隐私]
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
next_action: 用25分钟阅读图1、3.1节和表4，画出XML模块与代码模块的四阶段流水线
---

# 解锁秘密：评估LLM检测Android应用秘密的能力

> [!abstract] 一句话结论
> 论文证明通用、未经专项微调的 LLM 可以结合 Android 资源和字节码上下文，大规模发现传统正则或固定 API 规则漏掉的硬编码凭据；但 LLM 输出仍需正则、有效性测试或人工抽样验证。

## 为什么与当前研究相关

这篇论文同时命中“移动 App 静态分析”“LLM 参与程序分析”和“大规模自动测试”。它不是 CMP 合规论文，也不分析 Consent Signal；它最值得迁移的是工程结构：先用确定性工具把 APK 压缩成少量高价值文本，再让 LLM 做语义识别，最后用独立证据验证结果。

### 与 DIULENS / PICOSCAN 的关系

- 与 DIULENS：DIULENS 用动态 GUI 探索取得同意路径和事件时序；本文完全是静态流水线，不操作界面。可学习其“粗筛 → 上下文复核 → 外部验证”设计来约束 LLM。
- 与 PICOSCAN：PICOSCAN 依赖预定义 SDK/API 隐私签名；本文刻意测试 LLM 能否在没有预定义秘密格式或 API 签名时发现新类别。两者分别代表“规则可解释性”和“开放类别泛化”。
- 对当前 Demo 的迁移：可把 Appium 截图、控件文本和事件日志先做确定性裁剪，再只让 LLM解释少量候选；最终规则结论仍由结构化事件计算，不由 LLM直接定案。

## 研究问题、输入与输出

- RQ1–RQ3：通用 LLM 相对正则、LeakScope 等既有方法能发现多少秘密，标签是否准确，模型选择影响多大。
- RQ4–RQ5：在 50,000 个新收集 Google Play APK 上能发现哪些秘密，其中多少仍然有效。
- 输入：APK、`strings.xml`、Soot 从字节码恢复的 Jimple 类与字符串常量。
- 输出：候选字符串、秘密类型/服务商标签，以及经正则、人工抽样和有效性工具验证的证据集合。
- 威胁模型：具备 APK 的现实攻击者，以低成本自动收集嵌入应用的 API Key、Token、私钥或密码。

## 系统工作链路

```text
APK
 ├─ ApkTool → strings.xml
 │             ├─ A1：LLM批量找候选
 │             └─ A2：候选 + 完整XML上下文 → 标签或NOT_SECRET
 └─ Soot → Jimple类 + 字符串常量
               ├─ B1：LLM批量找候选
               └─ B2：候选 + 所属完整类 → 标签或NOT_SECRET
                              ↓
             正则确认 + 人工抽样 + 凭据有效性测试
```

| 部分 | 承担的工作 | 不能证明什么 |
|---|---|---|
| 静态工具 | ApkTool 解析资源；Soot 将字节码转为 Jimple 并提取字符串及所属类 | 不能发现运行时拼接、复杂加密或未覆盖文件中的秘密 |
| LLM | 第一阶段低成本召回候选；第二阶段利用 XML/类上下文复核并标注类型 | 标签不是漏洞事实，可能幻觉、漏报或把公开标识符当秘密 |
| 动态分析 | 主检测链路没有运行 App 或做动态污点分析 | 因而不能观察秘密是否在运行时发送或到达某个网络端点 |
| 知识库 | 没有维护 DIULENS 式 SDK 规则库；上下文主要来自当前 App 的 XML 和 Jimple 类 | 模型自身知识无法替代版本化、可审计的规则事实 |
| 人工与工具验证 | 人工抽样判断语义；正则确认结构；TruffleHog 等检查凭据是否仍有效 | 抽样和已知正则不能构成全量 Ground Truth |

## 零基础术语

- **硬编码秘密**：开发者把凭据直接写进 App。例如 `OPENAI_API_KEY="sk-..."` 随 APK 一起发布，反编译者就可能读取。
- **ApkTool**：解包 Android APK 的工具。这里主要取出保留标签语义的 `strings.xml`。
- **Soot**：Java/Android 静态分析框架。它把 Dalvik 字节码转换成更适合分析的中间表示。
- **Jimple**：Soot 的三地址中间表示。例如复杂 Java 语句会被拆成若干简单赋值和调用，便于提取字符串及代码上下文。
- **正则表达式（Regex）**：按固定字符结构匹配文本。Google API Key 有典型前缀时很好用，但新服务或非标准密码可能没有固定格式。
- **Ground Truth**：被可靠标注的真实答案集合。本文没有全量真值，因此“LLM 多发现了候选”不等于每个候选都是真秘密。

## 推荐阅读顺序

1. 摘要与贡献：先看研究对象和攻击者视角。
2. §2.2：理解正则、静态分析和传统机器学习为什么依赖先验知识。
3. 图1与 §3.1：精读 A1、A2、B1、B2，这是最容易复现和讲解的核心。
4. 图2、表3与图4：比较既有方法和不同模型。
5. 表4与 RQ5：区分“LLM候选”“结构有效”“仍可使用”三层证据。
6. §6：重点看动态字符串、混淆、文件覆盖、训练数据泄漏和真值缺失。

## 已核验的数据集、基线、指标与关键数字

- 基准集：既有工作的 5,135 个 Android App；对比正则工具与 LeakScope，并参考既有确认结果。
- 新数据集：2025 年 8–10 月从 Google Play 收集 50,000 个 App；857 个因 Soot 或 ApkTool 错误未完成分析，实际成功分析 49,143 个。
- 基准结果：gpt-4o-mini 重新发现 93% 的已知秘密，并至少确认 4,361 个额外有效秘密，作者报告相对既有工具增加 195%。
- 新数据集结果：LLM 在 17,590/49,143 个成功分析 App 中给出 27,561 个候选；正则确认其中 18,908 个，人工随机检查 96 个候选时确认 74 个（77%，95%置信水平、10%误差界）。
- 模型对比：gpt-4o-mini 找回 93% 已知秘密；最佳开源对照 gpt-oss-20b 为 25%。论文报告单 App 的 gpt-4o-mini 分析成本低于 0.01 美元。
- 有效性：确认集合中有 1,802 个凭据在发现时仍有效；170 名开发者确认问题并更新应用。

> [!warning] 正确解读
> 27,561 是 LLM 候选数，不是 27,561 个已证实漏洞；18,908 主要是正则结构确认，1,802 才涉及仍有效凭据。不同证据层不能混为一个数字。

## 局限与复现条件

- 只覆盖 `strings.xml` 和代码字符串常量，不处理运行时拼接、复杂加密、Manifest、JSON 等其他文件。
- Soot 或 ApkTool 失败会使 App 退出分析；混淆也可能改变可获得的上下文。
- 商业模型训练集不可审计，5,135-App 基准可能存在训练数据泄漏；新鲜 50,000-App 数据只能降低而不能完全排除该风险。
- 没有全量秘密 Ground Truth，绝对 recall 无法计算；正则和人工样本只是互补验证。
- 大规模复现需要合法 APK 来源、模型预算、秘密处置流程和负责任披露机制；不得把真实凭据放进课堂 Demo 或提交给第三方模型。

## 三分钟课堂提纲

1. 问题：固定正则和 API 规则只能找“事先知道长什么样”的秘密。
2. 方法：从 XML 与字节码取字符串，先批量粗筛，再带完整上下文逐个复核。
3. 验证：LLM 只生成候选；正则、人工抽样和有效性测试提供不同强度的证据。
4. 结果：在 49,143 个成功分析的新 App 中，17,590 个 App 被检测到候选，18,908 个候选经正则确认。
5. 启示：LLM 擅长开放类别语义识别，但必须被确定性程序分析和独立验证包住。

## 最小 Demo

只使用合成 Android 项目，绝不放真实凭据：

1. 在 `strings.xml` 放一个格式明显的假 API Key、一个普通 ID 和一个语义型假密码。
2. 在 Java/Kotlin 类中放一个分段字符串和一个普通常量；用 ApkTool/Soot 或简化脚本抽取文本及所属文件。
3. 第一轮只给字符串列表，让 LLM返回候选；第二轮给候选及 XML/类上下文，允许输出 `NOT_SECRET`。
4. 用自定义正则和人工标签生成对照表：`候选 → LLM标签 → 规则确认 → 人工真值`。
5. 课堂展示一例 LLM发现正则遗漏、再展示一例 LLM误报，说明为什么需要证据分层。

## 主动回忆问题

1. 为什么不直接把整个 APK 全部交给 LLM？
2. A1/B1 与 A2/B2 的上下文和目标分别有什么不同？
3. 本文为什么不是动态程序分析？
4. “正则确认”“人工确认”“仍有效”分别能支持多强的结论？
5. 没有全量 Ground Truth 时，93% recall 可以对哪些对象成立，不能推广到什么范围？
6. 如果迁移到 DIULENS Demo，哪些字段必须由程序规则计算，哪些解释可交给 LLM？

## 下一步行动

用 25 分钟阅读图1、§3.1 和表4，画出 A1/A2/B1/B2，并在每个箭头旁写清输入、输出、上下文和验证证据。

## 一手来源

- [ISSTA 2026 官方论文详情页](https://conf.researchr.org/details/issta-2026/issta-2026-research-papers/49/Secrets-Unlocked-Evaluating-LLMs-for-Secrets-Detection-in-Android-Apps)
- [作者公开论文 PDF](https://jordansamhi.com/static/files/papers/secretloc.pdf)
- [作者团队出版物页](https://jacquesklein2302.github.io/publications/)
