---
schema_version: 1
type: system
title: Paper Reading 与 Codebase Onboarding 使用手册
status: active
domain: [知识管理]
parent: "[[README]]"
related: []
sources: ["https://learn.chatgpt.com/docs/build-skills"]
created: 2026-08-31
updated: 2026-08-31
tags: []
---

# Paper Reading 与 Codebase Onboarding 使用手册

使用方式很简单：在消息开头输入 `$`，选择 skill，然后给出材料、输出格式和要求。因为已关闭隐式调用，所以必须手动写 [$paper-reading](D:\\Projects\\研讨厅\\.agents\\skills\\paper-reading\\SKILL.md) 或 [$codebase-onboarding](D:\\Projects\\研讨厅\\.agents\\skills\\codebase-onboarding\\SKILL.md)。[OpenAI 官方说明](https://learn.chatgpt.com/docs/build-skills)

## 1. 阅读论文：[$paper-reading](D:\\Projects\\研讨厅\\.agents\\skills\\paper-reading\\SKILL.md)

先上传 PDF，或者提供本地路径/论文网址。Obsidian 建议明确指定 `Markdown`，否则 skill 会询问你选择 Markdown 还是 HTML。

可以直接复制：

```
$paper-reading

请精读论文：
D:\论文\example.pdf

输出格式：Markdown
输出位置：<我的 Obsidian 论文笔记目录>

请重点完成：
1. 判断研究方向，以及属于应用型、探索型、理论型、综述型还是系统型研究
2. 说明论文解决的问题、研究背景和前人工作的不足
3. 用一句话概括核心贡献
4. 抽象整体方法流程和各个关键模块
5. 对每个模块说明输入、处理过程、输出、训练目标和模块关系
6. 提取论文中真正承载方法或实验结论的原始图片
7. 说明主要实验、基线、消融实验和失败案例
8. 区分作者声明、论文证据、代码证据和你的推断
9. 所有重要结论标注页码、章节、图号或表号
10. 输出成一篇可以直接在 Obsidian 阅读的论文主卡
```

如果论文有官方代码，可以补一句：

```
同时只读检查论文的官方代码仓库，用代码验证模块接口、数据形状、损失函数和推理流程；不要安装依赖或运行训练。
```

需要独立的可视化阅读报告时，将格式改成：

```
输出格式：HTML
```

HTML 会包含导航、公式渲染、架构图和可放大的论文图片；Markdown 更适合长期存入 Obsidian。

需要注意：这个 skill 本质上偏向“完整精读”，即使要求简短输出，也会尽量读完整篇论文。快速判断一篇论文值不值得读，可以先不调用 skill，直接让我分析标题、摘要和结论。

## 2. 阅读开源项目：[$codebase-onboarding](D:\\Projects\\研讨厅\\.agents\\skills\\codebase-onboarding\\SKILL.md)

最好先把仓库放在 `D:\Projects\研讨厅` 或其子目录下，因为这是项目级安装范围。

只读分析、不改仓库：

```
$codebase-onboarding

请阅读这个开源项目：
D:\Projects\研讨厅\开源项目\项目名称

只进行只读分析，不创建或修改 CLAUDE.md，不安装依赖，不运行项目。

请输出：
1. 项目解决什么问题
2. 技术栈和整体架构
3. 关键目录及各自职责
4. 程序的主要入口
5. 一条完整的数据流或请求处理流程
6. 核心模块之间的调用关系
7. 配置、模型、数据集和训练代码分别在哪里
8. 如果我要深入阅读，推荐的文件阅读顺序
9. 项目的设计亮点、局限和需要警惕的复杂部分

输出为 Markdown 项目阅读笔记。
```

如果确实希望它生成项目开发说明，可以使用：

```
$codebase-onboarding

分析 D:\Projects\研讨厅\开源项目\项目名称，
生成完整 Onboarding Guide，并创建或更新项目根目录的 CLAUDE.md。
```

第二种模式会修改项目里的 `CLAUDE.md`，所以只想学习代码时建议使用前面的“只读分析”版本。

## 3. 论文和代码结合阅读

推荐分两步执行：

```
$paper-reading
精读这篇论文，输出 Markdown，重点抽象方法模块、输入输出和关键图片：
<论文 PDF>
```

完成后再执行：

```
$codebase-onboarding
结合刚才的论文笔记，只读分析对应官方代码仓库：
<代码仓库路径>

重点建立“论文模块 → 实际代码文件/类/函数”的对应关系，并指出：
- 论文中描述但代码中未找到的部分
- 代码实现与论文描述不一致的部分
- 训练、推理、数据处理和配置入口
- 建议的源码阅读顺序

不要修改仓库文件。
```

如果输入 `$` 后看不到这两个名称，重启一次 Codex；项目级 skill 通常会自动发现。
