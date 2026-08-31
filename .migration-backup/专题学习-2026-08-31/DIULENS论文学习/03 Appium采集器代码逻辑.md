---
aliases: [Appium采集器代码导读]
tags: [论文阅读, Appium, UI自动化, DIULENS]
---

# Appium采集器代码导读

> [!info] 导航
> [[02 Android CMP实验App原理|上一章]] · [[DIULENS 学习总览|返回总览]] · [[04 四条DIU规则如何根据证据判断|下一章]]
	
## 1. Appium到底负责什么

Appium不是负责风险判断的AI。它提供四种基础能力：

1. 建立到模拟器的自动化会话。
2. 返回当前页面的XML结构。
3. 返回当前页面截图。
4. 根据resource-id或accessibility id点击元素。

本项目在这些基础能力之上实现CMP定向探索。

```text
Appium Driver
→ page_source / screenshot
→ parse_page_source
→ looks_like_cmp
→ rank_clickable_elements
→ click
→ 保存新页面并重复
```

## 2. ui.py：把平台XML变成算法输入

`UiElement`只保留跨页面分析需要的字段：class、text、resource-id、content-description、bounds、clickable和enabled。

`label` 优先使用无障碍描述，其次使用可见文本，最后使用resource-id尾部。原因是用户和LLM更容易理解语义标签，而resource-id更适合稳定定位。

### 页面指纹

`fingerprint` 对排序后的静态元素做SHA-256，故意排除坐标：

- 同一页面在不同分辨率下不应变成新页面。
- 元素顺序细微变化不应破坏去重。
- 文本或控件集合真正变化时应产生不同指纹。

指纹不是安全哈希用途，而是搜索算法的页面状态ID。

### CMP判断

仅出现“privacy”不能证明这是CMP，也可能是隐私政策。离线判断同时要求：

- 页面存在CMP语义，例如consent、privacy choices、vendors。
- 页面存在可执行选择，例如accept、reject、manage或allow。

这个保守条件对应论文指出的关键词方法误报问题，但仍只是教学近似；完整论文方案由LLM根据CMP指南判断。

### 点击排序

在CMP外部，Privacy choices、Consent、Privacy、Settings依次获得较高分。在CMP内部，accept、reject、save、close等可能结束CMP的操作扣20分，尽量最后点击。

这就是论文“延迟交互策略”的简化实现。

## 3. appium_collector.py：设备适配与流程编排

`AdbClient`负责：

- 安装APK。
- 清空旧日志。
- 强制停止旧进程。
- 通过Intent extra选择场景。
- 提取结构化Logcat证据。

所有命令使用参数列表调用，不拼接Shell字符串，避免空格、中文路径和注入问题。

`collect_with_appium` 的循环：

1. 获取XML。
2. 解析元素。
3. 生成页面指纹。
4. 判断是否为CMP。
5. 保存XML和PNG。
6. 对可点击元素排序。
7. 排除已经在该页面执行过的操作。
8. 点击分数最高的未访问元素。
9. 达到步数或无候选元素后结束。

如果路径已经离开CMP且仍有延迟操作没有探索，采集器最多重启App两次，再继续这些未访问操作。这是record-and-replay的受控简化：重启只服务于覆盖率，绝不能作为“用户可以重新进入CMP”的证据。

`visited_actions` 使用 `(页面指纹, 元素ID)`，因为同一个Settings按钮在不同页面语义不同，不能只按按钮文本全局去重。

CLI只向PowerShell返回纯ASCII的 `run_id`，再由脚本拼接项目路径。这是为了避免Windows旧代码页在捕获含中文的完整路径时把“研讨厅”损坏。

## 4. artifacts.py：为什么每一步都落盘

自动化实验如果只保存最终结论，就无法复查误报。每次运行因此保存：

- `manifest.json`：实验状态、页面和导航索引。
- `step-NNN.png`：人能检查的视觉证据。
- `step-NNN.xml`：算法实际看到的结构证据。
- `events.jsonl`：动态事件，每行独立JSON。
- `case.json`：统一分析输入。
- `report.json`：最终分析输出。

JSONL使日志即使最后一行写入失败，前面事件仍可读取；manifest每一步更新，使中断运行仍能诊断。

## 5. assembler.py：从采集格式到领域模型

Assembler是防腐层：Appium运行文件的格式可以变化，但分析器只认识 `CaseEvidence`。

它完成：

- 将毫秒时间转换成带UTC时区的datetime。
- 从供应商页面识别披露SDK。
- 将点击记录转为NavigationStep。
- 根据CMP重复出现或Privacy choices点击判断能否重新进入。
- 将截图路径转换为项目内相对路径。
- 保证真实案例的 `offline_report=None`，防止预填答案污染实验。

## 6. 第一次真实运行的解释

真实记录 `20260801T200426Z-early_access` 中：

- Appium保存6个页面快照。
- 检测出3个CMP页面状态。
- SDK在设备时间 `...72349` 读取Android ID。
- 首个CMP在 `...72371` 显示。
- 本地规则因此报告DIU-1=`YES`。
- SDK披露集合一致，且能经Settings→Privacy→Privacy choices重新进入CMP。

这条结论来自真实运行时序和UI路径，但数据访问事件目前仍由受控App自报告；加入Frida后才能称为外部动态观测。

## 7. 动手实验

1. 把 `max_steps` 改成2，观察证据为何不足以判断撤回路径。
2. 从 `_NAVIGATION_TERMS` 删除privacy，观察搜索路线变化。
3. 从页面指纹中加入bounds，改变模拟器分辨率并比较指纹。
4. 停止Appium服务后运行collect，检查失败运行的manifest。
5. 查看一次点击的reasons，逐项手算最终分数。

## 8. 自测题

1. 为什么截图和XML都要保留？
2. 为什么不能把所有包含privacy的页面都视为CMP？
3. 延迟点击Accept解决了什么覆盖率问题？
4. 页面指纹为什么不能包含时间戳？
5. Appium成功点击为什么仍不代表用户能够有效撤回同意？

参考答案：截图供人工语义复核，XML供程序定位；隐私政策会造成误报；Accept可能过早退出CMP；时间会让同页永不重复；还需验证保存后的consent signal和SDK实际行为。
