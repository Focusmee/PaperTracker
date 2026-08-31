---
aliases: [DIULENS Python代码链路]
tags: [论文阅读, Python, 程序分析, DIULENS]
---

# 当前Python Demo代码导读

> [!info] 导航
> [[00 学习地图|上一章]] · [[DIULENS 学习总览|返回总览]] · [[02 Android CMP实验App原理|下一章]]

## 1. 一次分析请求发生了什么

以 `early-access` 为例：

```text
用户在app.py选择案例
→ loader.load_cases读取JSON
→ CaseEvidence进行结构和语义校验
→ prepare_evidence计算可确定的事实
→ analyze_case选择在线或离线Provider
→ AnalysisReport再次校验模型输出
→ app.py把证据、规则和局限渲染出来
```

最重要的设计原则是：事实提取和风险解释分开。

- `Adjust在10:00:01访问Android ID` 是原始事件。
- `该事件早于10:00:05首次CMP显示` 是机械事实。
- `因此存在DIU-1风险` 是规则判断。
- `可能仍需确认事件归因和适用法律` 是结论边界。

## 2. models.py：项目的数据合同

### EventType

限制事件只能属于：

- `data_access`：某SDK访问某种数据。
- `cmp_displayed`：某个CMP页面出现。
- `consent_action`：用户进行同意相关操作。

如果所有模块都随意使用字符串，就会出现 `data-access`、`access_data` 等不兼容写法。枚举把这种错误提前变成校验失败。

### EvidenceEvent

它使用 `model_validator` 检查不同事件需要不同字段：

- 数据访问必须有 `sdk` 和 `data_type`。
- CMP显示必须有 `screen`。
- 同意操作必须有 `action`。

这属于“让非法状态无法悄悄进入分析器”。

### CaseEvidence

它是一宗案例的统一证据容器。目前包含截图、实际SDK、披露SDK、运行事件、导航路径、控件能力和离线报告。后续Appium不会改写分析器，而是负责生成同样的数据合同。

### AnalysisReport

最终报告必须恰好包含DIU-1到DIU-4。在线LLM即使返回合法JSON，如果缺少一条规则，也会被Pydantic拒绝并回退。

## 3. loader.py：只负责边界输入

Loader完成三件事：

1. 找到案例JSON。
2. 解析JSON。
3. 交给Pydantic校验。

它不判断风险。这使得以后输入可以来自文件、Appium运行目录或数据库，而规则层保持不变。

## 4. rules.py：机械事实而非最终裁决

`prepare_evidence` 当前计算四组事实：

1. 找到第一次CMP显示时间。
2. 找到此前发生的数据访问。
3. 对实际SDK集合和披露集合做差集。
4. 汇总撤回障碍与标签实际效果不一致。

集合差值的含义：

```text
actual - disclosed = App实际包含但CMP没告诉用户的SDK
disclosed - actual = CMP声称存在但静态证据没发现的SDK
```

为什么不让这里直接决定全部结果？DIU-3和DIU-4常涉及语义。例如“OK”可能实际上保存修改，仅凭关键词会误报。因此机械层提供事实，解释层负责结合上下文。

## 5. providers.py：把分析方法做成可替换策略

`AnalysisProvider` 是共同接口：输入案例和预处理证据，输出报告。

- `OfflineProvider` 返回人工审核过的合成基线，保证课堂无网可运行。
- `OpenAICompatibleProvider` 将结构化证据与截图发送给多模态模型。

在线路径的防护：

1. 密钥来自环境变量，不写进源码。
2. 请求设置超时且不无限重试。
3. 优先要求严格JSON Schema。
4. 再用Pydantic验证语义结构。
5. 任一步失败都由上层回退并显示原因。

离线基线的局限：它是预填答案，不是算法从原始数据重新推导的结论。因此它适合教学与界面演示，不能用于声称真实检测准确率。

## 6. analyzer.py：用例编排器

`analyze_case` 不包含具体规则或API细节，它只组织流程：

```python
prepared = prepare_evidence(case)
if mode == "offline":
    return offline.analyze(...)
try:
    return online.analyze(...)
except AnalysisProviderError:
    return offline.analyze(...)
```

这种编排方式使Streamlit不必知道API如何调用，也使单元测试能够直接测试分析逻辑。

## 7. app.py：展示层

页面负责：选择案例、选择模式、触发分析、保存本次会话结果、渲染截图和规则卡片。页面不应自己计算SDK差集或比较时间，否则相同规则无法被CLI和测试复用。

## 8. 动手实验

### 实验A：制造输入错误

复制一个案例，删除 `data_access` 事件的 `sdk` 字段，再运行：

```powershell
.\.venv\Scripts\python.exe -m pytest -q --basetemp .pytest-tmp
```

观察错误来自模型边界，而不是运行到规则层才崩溃。

### 实验B：理解SDK集合差

在合规案例的 `actual_sdks` 中增加 `ExampleTracker`，但不加入 `disclosed_sdks`。查看机械预处理中的 `missing_disclosures`。

### 实验C：理解回退

不配置密钥，选择在线模式。确认报告标记为离线回退，并显示原因。然后回答：回退结果来自哪里，是否代表模型真的分析了截图？

## 9. 自测题与答案

1. 为什么先校验再分析？——防止缺字段或类型错误被误认为没有风险。
2. `prepare_evidence` 为什么不直接调用LLM？——机械事实可重复、可测试，语义判断才需要模型。
3. Appium数据应接入哪个位置？——生成或适配为 `CaseEvidence`，而不是把设备代码塞进页面。
4. `UNKNOWN` 和 `NO` 有什么区别？——UNKNOWN是证据不足；NO是现有证据支持未发现该风险。
5. 为什么截图不能单独判断DIU-1？——DIU-1需要数据访问与CMP出现的时间关系。
