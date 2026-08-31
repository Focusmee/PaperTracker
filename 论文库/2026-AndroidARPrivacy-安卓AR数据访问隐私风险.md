---
schema_version: 1
type: source
research_direction: 移动隐私与软件供应链
research_type: 探索型
title: Android AR应用数据访问实践与用户隐私风险
english_title: "An Empirical Study of Data Access Practice in Android AR Apps to Understand User Privacy Risks"
authors: [Sabbir Hussain Meraj, Long Trac, Xiaoyin Wang, Xusheng Xiao, Wei Wang]
venue: 41st IEEE/ACM International Conference on Automated Software Engineering (ASE 2026)
publication_date: 2026-10-12
doi: ""
official_url: https://conf.researchr.org/track/ase-2026/ase-2026-research-track
code_url: https://zenodo.org/records/21189157
local_pdf: "[[附件/论文原文/03-Android-AR-英文原文-ASE-2026.pdf]]"
publication_status: 官方已接收
status: unread
domain: [移动应用隐私, Android程序分析, AR应用]
parent: "[[README]]"
related: ["[[论文库/2026-PINFINDER-SDK隐私上下文一致性]]", "[[论文库/2026-DIULENS-移动CMP隐私风险]]", "[[论文库/2026-PlainTextPlainRisks-AndroidWebView明文风险]]"]
sources: [https://conf.researchr.org/track/ase-2026/ase-2026-research-track, https://zenodo.org/records/21189157/files/paper.pdf, https://zenodo.org/records/21189157]
created: 2026-08-17
updated: 2026-08-31
tags: [研究/移动隐私, 方法/静态分析, 方法/动态分析]
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
next_action: 用25分钟阅读图3、图6和表4、表5，画出“APK到隐私候选再到Frida验证”的五步链路
---

# Android AR应用数据访问实践与用户隐私风险

> [!abstract] 一句话结论
> 作者针对 ARCore 与 Unity ARFoundation 应用构建了 AR 专用静态分析流程，用调用图定位人脸、图像、骨骼和完整相机帧等敏感访问，再把代码行为与应用披露进行比对，并用 Frida 对部分候选做运行时确认。

## 为什么与当前研究相关

这篇论文同时覆盖“SDK 提供了什么宽泛能力”“App 实际调用了什么”“应用声明是否说明该访问”和“静态候选能否在运行时出现”。它能补充当前以 CMP 和 Consent Signal 为中心的学习主线：即使用户同意使用相机，App 仍可能通过过宽 API 获得超出功能所需的人脸、图像或完整环境数据。

### 与 DIULENS / PICOSCAN 的关系

- 与 [[论文库/2026-DIULENS-移动CMP隐私风险]]：DIULENS 检查同意界面与同意前后行为；本文不分析 CMP，而是检查同意之后的数据访问是否被披露、是否满足最小权限原则。
- 与 PICOSCAN：两者都从 SDK 类/API 语义出发定位风险。PICOSCAN 关注宿主、Wrapper 与隐私配置 API 的责任传播；本文关注 ARCore/ARFoundation 的敏感数据类、宽泛 API 与调用路径。
- 可迁移点：把 DIULENS Demo 中的“已知 CMP/SDK 入口知识”扩展为版本化的“敏感类、API、事件回调、预期数据类型”清单，再用动态验证降低纯静态分析的误报风险。

## 研究问题、输入与输出

- RQ1：AR App 在多大程度上访问了未明确披露的敏感数据？
- RQ2：AR App 在多大程度上通过完整相机帧或通用 Trackable API 违反最小权限原则？
- RQ3：可以怎样改进 AR SDK、应用商店审核和代码审计？
- 输入：179 个 Android AR APK、应用描述、隐私政策、Google Play Data Safety、ARCore/ARFoundation 的敏感类和 API 知识。
- 输出：敏感数据访问路径、未披露访问候选、最小权限违规候选，以及部分候选的运行时确认日志。

## 系统完整工作链路与分工

```text
APK + 商店披露材料
  ↓ 识别 ARCore / ARFoundation
Unity元数据恢复（仅ARFoundation）
  ↓
Androguard反编译Java / Ghidra反编译IL2CPP本地代码
  ↓
生成调用图
  ↓ 从已知AR敏感类/API出发反向+正向遍历
抽取AR专用调用路径
  ↓
代码与路径检查：敏感访问 ↔ 披露关键词；宽泛API ↔ 最小权限
  ↓
Frida Hook 已标记方法 → 运行时日志确认
```

| 部分 | 具体职责 | 证据边界 |
|---|---|---|
| 静态分析 | SDK识别、反编译、调用图、AR专用路径抽取、宽泛API定位 | 能覆盖潜在路径，但不能证明路径真实执行 |
| 动态分析 | Frida拦截指定方法，在 Pixel 8a / Android 16 上记录候选访问 | 只能确认测试条件下被触发的访问；未触发不等于不存在 |
| LLM / Agent | 原论文系统流程没有使用 LLM 或 Agent | 后续可用于解释路径或引导触发，但不能替代调用证据 |
| 知识库 | AR敏感类/API、ARFoundation事件、披露关键词和宽泛访问模式 | 本质是小型领域知识表，不只是SDK名称清单 |
| 人工验证 | 处理模糊披露语句、判断功能是否确实需要宽泛数据、解释候选 | 论文未提供可计算完整 recall 的全量 Ground Truth |

## 零基础术语与例子

- **ARCore**：Google 的 Android AR SDK。例如家具摆放 App 用它识别“地面”这个平面。
- **ARFoundation**：Unity 的跨平台 AR 抽象层；Android 版本通常最终调用 ARCore，但 App 逻辑会被编译成本地代码。
- **Trackable**：SDK 持续跟踪的真实世界对象，如平面、人脸、二维码图像或人体骨架。
- **Anchor**：把虚拟物体固定到真实对象上的位置关系。例如把虚拟椅子固定在识别出的地面上。
- **调用图**：以函数为节点、函数调用为边的图。若 `onDrawFrame → getAllTrackables → AugmentedFace`，分析器可沿图定位人脸访问。
- **最小权限原则（PoLP）**：程序只应取得完成任务所需的最少数据。只需识别平面的家具 App 不应读取完整相机帧。

## 推荐阅读顺序

1. 摘要与图1：先理解“AR功能需要相机”不等于“可以读取所有环境信息”。
2. 表1、图3：区分 ARCore 的主动拉取 API 与 ARFoundation 的事件回调。
3. 图6与 §3.3：精读完整分析链路，这是最适合复现和课堂讲解的部分。
4. 表4、表5：区分静态候选数量和动态确认数量。
5. 代码片段1–4：看完整相机帧和通用 Trackable API 为什么过宽。
6. §6、§7：理解 SDK 重设计、自动披露审核及静态/动态分析局限。

## 已核验的数据集、指标与关键数字

- 数据集为 179 个 Android AR App：54 个 ARCore App、125 个 ARFoundation App。
- 静态分析发现 58 个未披露敏感访问实例，其中 23 个在测试环境中经 Frida 确认。
- 静态分析发现 82 个最小权限违规候选，其中 40 个经动态执行确认。
- 754 条 AR 相关路径中有 317 条跨 App 复用：ARCore 为 236/620，ARFoundation 为 81/134。
- 论文是实证测量研究，没有与通用 Android 隐私检测器进行同任务精度基线比较；动态确认是候选的下界验证，不是完整 Ground Truth。

> [!warning] 正确解读
> “未被 Frida 确认”不能直接当作假阳性，因为某些访问需要特定人脸、图像标记或运行状态才能触发；反过来，静态路径存在也不能证明真实用户运行时一定执行。

## 局限与复现条件

- 反射、动态类加载、混淆和本地代码动态分派会影响静态结果。
- ARFoundation 经 IL2CPP 编译成本地代码，恢复标识符和调用关系比普通 Java APK 更困难。
- 披露核验主要依赖关键词，再由作者处理模糊表述，仍可能漏掉同义表达或语境差异。
- 动态验证只覆盖被测试条件触发的路径；论文明确把完整动态分析留作后续工作。
- 仅覆盖 ARCore 与 ARFoundation；Vuforia、EasyAR 和原生 iOS ARKit 不在主数据集内。
- 最小复现需要 APK、Androguard、Il2CppDumper、Ghidra、自定义调用图脚本、Frida、可插桩 Android 设备，以及论文公开的领域规则和样本材料。

## 三分钟课堂提纲

1. 问题：AR App 即使获得相机权限，也可能读取未披露或超出功能需要的物理世界数据。
2. 方法：用 AR SDK 类/API 构建领域入口，反编译 APK、生成调用图并抽取 AR 专用路径。
3. 判定：一条线检查“代码访问与披露是否一致”，另一条线检查“是否通过过宽 API 违反最小权限”。
4. 验证：静态分析给潜在上界，Frida 运行时日志给测试条件下的可观察下界。
5. 结果：179 个 App 中发现 58 个未披露访问实例和 82 个最小权限违规候选，说明 SDK API 设计本身也影响隐私风险。

## 最小 Demo

优先使用作者 Zenodo 材料中的 2-App 演示子集，不从零收集真实商店 APK：

1. 展示一份 AR 敏感 API 清单，例如 `acquireCameraImage()` 和 `getAllTrackables()`。
2. 对样本输出“调用路径 + 对应披露关键词 + 规则判定”。
3. 选择一个已标记方法，用 Frida 日志展示它在一次受控运行中是否被触发。
4. 课堂上明确区分：静态候选、动态确认、人工判断、论文 Ground Truth。

第一版 Demo 只复现一条调用路径和一条 Frida 事件即可；不要把“检测到 API”直接表述为法律违规。

## 主动回忆问题

1. 为什么相机权限已经获得，App 仍可能违反最小权限原则？
2. 为什么 ARCore 和 ARFoundation 需要不同的反编译与调用图生成流程？
3. 静态分析和动态分析在本文分别近似提供什么“上界”和“下界”？
4. 披露关键词表为什么是知识库的一部分，但不能直接当作 Ground Truth？
5. 如果把 DIULENS 接到这条链路中，CMP 的同意事件应放在哪一步，仍缺少哪些数据流证据？

## 下一步行动

用 25 分钟阅读图3、图6和表4、表5，在纸上画出五步链路，并给每一步标注“自动工具 / 人工 / 静态证据 / 动态证据”。

## 一手来源

- [ASE 2026 官方接收列表](https://conf.researchr.org/track/ase-2026/ase-2026-research-track)
- [论文 PDF（作者公开复现记录）](https://zenodo.org/records/21189157/files/paper.pdf)
- [Zenodo 复现材料与 2-App 演示子集](https://zenodo.org/records/21189157)
