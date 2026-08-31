---
schema_version: 1
type: topic
title: HScope中文译注与逐步精读
status: active
domain:
  - OpenHarmony
  - 静态污点分析
  - 论文精读
parent: "[[专题学习/四篇精读论文零基础路线/08 原文译读与伴读入口]]"
related:
  - "[[专题学习/四篇精读论文零基础路线/04 HScope零基础导读]]"
  - "[[论文库/2026-HScope-OpenHarmony细粒度隐私泄漏检测]]"
sources:
  - "[[HScope-英文原文-ISSTA-2026.pdf]]"
created: 2026-08-26
updated: 2026-08-26
tags:
  - 研究/移动隐私
  - 方法/静态分析
  - 学习/精读
---

# HScope 中文译注与逐步精读

> [!warning] 版本说明
> 本文依据作者公开的 22 页 ISSTA 2026 论文制作，是按原文章节和图表组织的非官方学习译注。为便于零基础理解，采用忠实译读与解释性压缩，不替代 [[HScope-英文原文-ISSTA-2026.pdf]]。

## 论文信息

- 英文题目：*Fine-Grained Privacy Leakage Detection in OpenHarmony Apps*
- 中文学习题目：OpenHarmony 应用中的细粒度隐私泄漏检测
- 作者：Aohan Mei、Guangliang Yang、Xinming Guo、Yi Wang、Fuan Gui、Min Yang
- 出处：ISSTA 2026
- 一句话：直接分析无类型 Ark 字节码，通过抽象解释、框架建模和污点传播恢复敏感数据从 Source 到 Sink 的可能路径。

## 贯穿案例

组件 C1 点击后读取 OAID 并写入状态 `message`；C1 创建组件 C2，把同一个状态和回调传给它；C2 点击按钮时通过[^11]回调把 `message` 传回 C1 的 `request`，最终进入 `http.post`。源和汇不在同一函数，也不在同一组件。



![[附件/论文截图/HScope-图1-跨组件泄漏案例.png]]

> [!note] 图1看图顺序
> C1 读取用户设备的 OAID，把它保存到 `message`；这个 `message` 又传给 C2；用户点击 C2 的按钮后，C2 通过回调函数把 OAID 传回 C1，最后 C1 用 `http.post()` 把它发送出去。
### `@Entry @Component`

```
@Entry
@Component
struct C1
```

先简单理解：

- `@Component`：这是一个 UI 组件
- `@Entry`：这是页面的入口组件

类似：

```
整个页面
└── C1
```

### `@State message`

```
@State message: string = 'no taint'
```

意思是：

> C1 有一个状态变量 `message`。

开始的时候：

```
message = "no taint"
```

也就是普通数据，还没有敏感信息。

`@State` 的关键点是：

> **这个变量发生变化以后，UI 和与它关联的状态也会更新。**

所以你可以把它想象成 C1 手里有一个盒子：

```
C1

message
┌────────────┐
│ "no taint" │
└────────────┘
```


### 关键点1：Source在哪？
看这里：

```
Text(this.message).onClick(() => {
    this.message = identifier.getOAID()
})
```

意思是：

> 用户点击这个 Text 后，执行里面的回调函数。

也就是：

```
用户点击 Text
       ↓
执行回调
       ↓
identifier.getOAID()
       ↓
得到 OAID
       ↓
保存到 this.message
```

`OAID` 可以理解为一个设备广告标识符，属于隐私相关数据。

因此论文把：

```
identifier.getOAID()
```

当成：

> **Source**

也就是敏感数据源头。

现在：

```
原来：

message = "no taint"

用户点击后：

message = OAID
```

于是 `message` 被“污染”了。

可以画成：

```
identifier.getOAID()
       ↑
     Source
       ↓
     OAID
       ↓
this.message
```

### 关键点 2：Sink 在哪里？

前面还有：

```
request(message) {
    http.post(message)
}
```

这里：

```
http.post(message)
```

就是 **Sink**。

因为它把数据发送到网络。

所以：

```
Source = identifier.getOAID()
             ↓
          敏感数据


Sink = http.post(...)
             ↓
         数据发送出去
```

问题就是：

> **OAID 最后有没有跑进 `http.post()`？**

污点分析就是要回答这个问题。




### `@Link message`

```
@Link message: string
```

这个 `message` 不是自己独立的一份数据。

你可以先把 `@Link` 理解成：

> **C2 的 message 和父组件 C1 的 message 绑定在一起。**

C1 创建 C2 的时候：

```
C2({
    callback: ...,
    message: this.message
})
```

所以关系大概是：

```
C1

this.message
    │
    │ 绑定
    ↓

C2

this.message
```

刚开始：

```
C1.message = "no taint"

       ↓ @Link

C2.message = "no taint"
```

后来用户点击 Text：

```
C1.message = OAID
```

因为存在状态关联，C2 里使用的 `message` 也对应这个状态。

所以可以理解：

```
identifier.getOAID()
       ↓
     OAID
       ↓
C1.message
       ↓
    @Link
       ↓
C2.message
```

这就是前面那句话中的：

> **共享状态 / 跨组件状态传播**


### C2的callback
C2 定义：

```
callback = (): void => {}
```

简单理解：

> C2 留了一个位置，用来装“别人传给我的函数”。

刚开始相当于：

```
callback = 一个什么也不做的函数
```

但 C1 创建 C2 的时候：

```
C2({
    callback: (msg) => { this.request(msg) },
    message: this.message
})
```

C1 把一个新函数传给 C2：

```
(msg) => {
    this.request(msg)
}
```

所以现在 C2 的：

```
callback
```

实际上就是：

```
(msg) => {
    C1.request(msg)
}
```

这就是你刚才问的：

> **函数作为值传递。**

不是传：

```
数字
字符串
对象
```

而是在传：

```
一个函数
```

可以理解成 C1 告诉 C2：

> “C2，以后你需要处理数据的时候，就调用我给你的这个函数。”


### C2 点击按钮之后发生什么？

C2：

```
Button().onClick(() => {
    this.callback(this.message)
})
```

大白话就是：

> 用户点击按钮以后，把 C2 当前的 `message` 交给 callback。

假设现在：

```
C2.message = OAID
```

那么：

```
this.callback(this.message)
```

实际上相当于：

```
callback(OAID)
```

可是刚才已经知道：

```
callback = (msg) => {
    C1.request(msg)
}
```

所以：

```
callback(OAID)
```

实际上就变成：

```
C1.request(OAID)
```

然后 `request()`：

```
request(message) {
    http.post(message)
}
```

于是：

```
C1.request(OAID)
```

变成：

```
http.post(OAID)
```

🎯 **Source 成功到达 Sink。**


## 把整个过程完整走一遍

这个例子最重要，你按照时间顺序看：

### 第一步：页面刚打开

```
C1.message
    ↓
"no taint"

    ↓ 传给 C2

C2.message
    ↓
"no taint"
```

同时 C1 给 C2 一个回调：

```
C2.callback
     ↓
(msg) => C1.request(msg)
```

---

### 第二步：用户点击 C1 的 Text

执行：

```
this.message = identifier.getOAID()
```

于是：

```
identifier.getOAID()
       ↓
      OAID
       ↓
C1.message
```

此时敏感数据出现。

---

### 第三步：OAID 传播到 C2

通过 `@Link`：

```
C1.message
    │
    │ OAID
    ↓
C2.message
```

于是 C2 现在也能拿到 OAID。

---

### 第四步：用户点击 C2 的 Button

执行：

```
this.callback(this.message)
```

相当于：

```
callback(OAID)
```

---

### 第五步：callback 回到 C1

因为 callback 是 C1 当初传进来的：

```
(msg) => {
    this.request(msg)
}
```

所以：

```
callback(OAID)

↓ 等价于

C1.request(OAID)
```

---

### 第六步：到达 Sink

```
request(OAID) {
    http.post(OAID)
}
```

最终：

```
OAID
 ↓
http.post()
```

完成了一条隐私数据泄漏路径。


## 结论：
所以如果一个很笨的分析器只看 C2：
```
Button().onClick(() => {
    this.callback(this.message)
})
```

它可能看到：

```
callback 是谁？

不知道。
```

于是调用图可能断在：

```
C2.onClick
   ↓
callback
   ↓
？？？
```

那么它就不会知道后面其实还有：

```
C1.request()
↓
http.post()
```

另一方面，如果它只分析单个函数：

### 看读取 OAID 的函数

```
onClick(() => {
    this.message = identifier.getOAID()
})
```

发现：

```
Source ✅
Sink ❌
```

### 看 `request()`

```
request(message) {
    http.post(message)
}
```

发现：

```
Source ❌
Sink ✅
```

### 看 C2 的 onClick

```
onClick(() => {
    callback(message)
})
```

发现：

```
Source ❌
Sink ❌
```

于是一个过于简单的分析器可能错误认为：

> 没有 Source → Sink 路径。

但你现在站在整个程序角度看，明明存在：

```
getOAID
  ↓
message
  ↓
C1
  ↓
C2
  ↓
callback
  ↓
C1.request
  ↓
http.post
```

这就是论文说的 **跨函数 + 回调 + 跨组件 + 状态传播**。


还有一个小细节：图里 C2 第 24 行注释写的是：

```
// Call sink API
```

但严格来说，**第 25 行的 `this.callback(...)` 本身不是最终 Sink**。

真正的 Sink 是 C1 第 5 行：

```
http.post(message)
```

C2 第 25 行更准确地说是在：

> **通过 callback 间接触发 Sink。**

这也是为什么这个例子特别适合解释“间接调用”。

总的来说：
C1 从 Source `getOAID()` 获取敏感数据并存入 `@State message`，该状态通过 `@Link` 传播给 C2；C2 点击事件把 `message` 作为参数传入由 C1 提供的 callback，callback 再调用 C1.request()，最终数据进入 Sink `http.post()`，形成一条跨组件、跨回调的污点传播路径。

---

## 第一单元：摘要与引言——为什么 Android 工具不能直接搬来

### 原文摘要译读

OpenHarmony 快速普及后，隐私滥用与数据泄漏成为重要问题。Ark 字节码缺少静态类型且行为灵活，使现有分析方法难以精确解析间接调用和复杂组件通信。论文提出 HScope：直接分析 OpenHarmony App 字节码，通过抽象解释模拟程序行为，恢复间接调用与跨组件通信，并执行细粒度隐私数据流检测。

作者在 300 个真实 OpenHarmony App 上评估系统，报告 39 个此前未知、经人工确认的隐私问题，涉及 27 个 App。结果表明，HScope 能为不断扩张的 OpenHarmony 生态提供可扩展的静态隐私检测基础。

### 引言译读

[^2]OpenHarmony 使用 ArkTS/JavaScript 风格开发并编译为 Ark 字节码。与 Android 的 Java/Dalvik 生态不同，[^1][^4]字节码层可能丢失类型，[^3]函数可作为值传递，[^5]闭包、[^6]回调和[^7]共享状态十分普遍。因此，[^8]先建立固定调用图再做污点传播容易漏掉真实调用；只[^9]在单函数内找 [^10]Source 和 Sink 更会漏掉跨组件路径。


关于字节码丢失类型
（以 Java / Android 为例，一段程序从你写出来，到真正被 CPU 执行，中间会经过好几“层”。

最直观的是这条链：

```
源代码层
Java / Kotlin
    ↓ 编译

字节码层
.class / DEX
    ↓ JVM / ART 解释或编译

机器码 / 汇编层
x86 / ARM 指令
    ↓

CPU 硬件执行层
```

- Ark 字节码类型丢失的核心原因 ArkTS 本身是兼容 JavaScript 的静态强类型语言，为了保证 JS 的动态性兼容，在编译为 Ark 字节码的过程中，部分动态类型信息会被擦除，导致字节码层丢失类型信息。
    

- 与 Android Dalvik 生态的差异 Android 的 Java 是纯静态强类型语言，编译为 Dalvik 字节码时会保留完整的类型签名，字节码层可以直接进行类型校验，因此不会出现类型丢失的问题。
    

- 类型丢失的影响与优化方向 类型丢失会降低字节码层的类型安全性，同时也会影响性能优化的效果，目前 Ark 编译器正在逐步优化类型信息的保留策略，在兼容 JS 动态性的同时尽可能保留类型信息，提升运行时的性能和安全性。）



最关键的对比图：

```
简单情况：

Source
  ↓
函数 A
  ↓
函数 B
  ↓
Sink

调用关系比较容易提前确定
```

而文章描述的 ArkTS 情况更像：

```
                  ┌→ callback A ─────┐
Source → 函数对象 ├→ callback B      │
                  └→ callback C      │
                                     ↓
                                  闭包状态
                                     ↓
                                  共享状态
                                     ↓
                                  另一组件
                                     ↓
                                   Sink
```

所以真正难点不是“找一个 Source 和一个 Sink”，而是：

> **判断 Source 产生的数据，经过动态函数调用、回调、闭包、共享变量和组件通信之后，最终有没有真的到达 Sink。**
### 三个新术语

- **Ark Bytecode（Ark 字节码）**：OpenHarmony App 编译后的低层指令。
- **Indirect Call（间接调用）**：被调用函数由变量或回调决定，代码中没有固定目标名称。
- **Inter-component Communication（ICC，组件间通信）**：页面或组件通过回调、路由或共享状态传递数据和控制。

### 批注：论文的核心困难

HScope 不是“再写一个 Source/Sink 搜索器”。**真正贡献是让无类型、回调密集、组件化的程序重新拥有足够的语义关系，然后污点分析才有机会看见完整路径。**

### 最少记忆清单

- OpenHarmony 的难点来自无类型字节码、间接调用和 ICC。
- 先恢复程序语义，再做污点分析。
- 静态报告的是可能路径，不是某次运行已经泄漏。

### 主动回忆题

1. 为什么只看到 `getOAID` 和 `http.post` 还不够？
因为getOAID只发现Source没有Sink，而http.post只发现Sink没发现Source
2. 回调会怎样破坏预先固定的调用图？
**回调破坏“预先固定调用图”**：因为在代码里看到 `callback()` 时，你可能还不知道这个 `callback` 实际指向哪个函数；只有追踪函数值是怎么传递过来的，才能补上真正的调用边。
3. HScope 与动态抓包的结论边界有什么不同？
**HScope ≠ 动态抓包**：HScope回答的是“代码中是否存在一条可能把敏感数据从 Source 送到 Sink 的可行路径”；动态抓包回答的是“这一次真实运行过程中，有没有实际产生并发送这样的网络数据”。
### 进入下一单元检查点

- [x] 能用图1画出五节点数据流。
- [x] 能解释无类型字节码为什么增加分析难度。(没有可靠类型信息时，分析器很难知道“这个变量到底是什么对象、这个调用到底会调到哪个函数”，于是调用图和数据流都会变得更难确定。)
- [x] 不把静态可达说成真实发生。

---

## 第二单元：第2节——OpenHarmony 编程模型与真实案例

### 2.1 编程模型译读

OpenHarmony 应用由 Ability 和 UI 组件组成。ArkUI 采用声明式界面；事件回调、装饰器状态和路由跳转决定运行时控制流。状态变量可能在父子组件之间通过链接共享，函数也可以作为回调传递。对静态分析而言，数据关系常藏在框架约定中，而不是显式普通赋值。

### 2.2 真实案例译读

论文用两个组件说明传统分析为什么失败。C1 的 `message` 最初无污点；用户点击文本后，OAID 被写入该字段。C1 构造 C2 时把 `message` 与匿名回调一起传入；C2 的按钮点击处理器调用回调。回调闭包最终调用 C1 的 `request`，把共享字段送入 `http.post`。

若分析器不知道 `@State` 与 `@Link` 指向同一抽象对象，污点会在组件边界消失；若不知道 `this.callback` 的实际目标是父组件传入的匿名函数，调用图也会断开。

### 三个新术语

- **State Alias（状态别名）**：两个字段名在运行时可能指向同一个状态对象。
- **Callback（回调）**：把函数作为值交给另一组件，之后由对方触发。
- **Closure（闭包）**：函数携带创建位置的变量环境，使回调能够访问父作用域对象。

### 最少记忆清单

- 共享状态决定数据边，回调决定控制边。（共享状态告诉你“数据到了谁手里”，回调告诉你“接下来执行谁”。）
- 两类边缺一条，Source-to-Sink 路径都会断。
- 框架语义必须显式建模，不能只靠通用语法。（框架有很多“隐藏语义”，比如）
```
遇到 @Link：
→ 建状态关联 / alias

遇到 onClick：
→ 建事件回调控制边

遇到组件参数：
→ 建父子组件数据传播

遇到生命周期函数：
→ 补框架隐式调用
```
### 通用语法模型只认识

```
变量
赋值
函数
调用
对象
```
### OpenHarmony 框架模型还要认识

```
@State
@Link
@Component
@Builder
onClick
生命周期
组件通信
事件回调
状态刷新
……
```

### 主动回忆题

1. `@Link message` 为什么不是普通复制？
它们可能指向同一个抽象存储位置，而不是简单复制
2. 回调目标不明会影响调用图还是污点标签，还是两者都影响？
直接影响调用图，间接影响污点传播，因此最终两者都会受影响。
3. 如果 C2 在发送前清空 message，流敏感分析应怎样处理？
清空message这里通常叫：
> **kill taint / 杀死污点。**
> 流敏感指的是对
### 进入下一单元检查点

- [x] 能分别指出数据边和控制边。
```
identifier.getOAID()       Source
        │
        ▼
     C1.message
        │
      @Link
        │
        ▼
     C2.message
        │
        ▼
 callback(message)
        │
        ▼
    C1.request()
        │
        ▼
   http.post()             Sink
```
	数据边：OAID → C1.message → C2.message → 参数 msg
	控制边：C2.onClick → callback → C1.request → http.post
- [x] 能解释共享状态为何需要别名分析。
共享状态需要别名分析，是因为两个看起来不同的程序变量可能代表同一个底层状态。
- [x] 能说出传统单函数分析的漏报位置。

---

## 第三单元：第3.1—3.3节——从字节码到可分析语义

![[附件/论文截图/HScope-图2-系统工作流.png]]

> [!note] 图2看图顺序
> 三段即可：Bytecode Frontend 把包变成 IR；Semantic Analysis Engine 边解释边建立调用与 ICC 关系；Privacy Detection 在这些关系上跑污点规则并生成报告。
> 意思是：HScope 拿到一堆机器比较容易懂、人很难懂的 Ark 字节码，先把它整理成自己容易分析的格式，然后“假装执行”程序，在这个过程中搞清楚数据可能去哪、函数可能调谁，最后再检查隐私数据有没有从 Source 流到 Sink。

OpenHarmony 安装包
      ↓
① 把字节码翻译成人更容易分析的 IR
      ↓
② 不真正运行 App，而是在脑子里“模拟所有可能情况”
      ↓
③ 一边模拟，一边发现“谁可能调用谁”
      ↓
④ 再追踪敏感数据有没有流到危险位置
      ↓
生成隐私问题报告

### 3.1 字节码前端与 IR 译读

HScope 输入 OpenHarmony 应用包，使用 ArkCompiler 的 `ark_disasm` 解码 Ark 字节码。前端解析指令并做结构化反编译，把跳转恢复为条件分支等结构，再转换成轻量 IR。它还恢复变量名、类属性与词法环境，使闭包中的槽位能重新对应到变量。

IR = 给静态分析器看的简化语言。
恢复变量名、类属性与词法环境：搞明白闭包偷偷带走的那些变量到底是谁。
### 3.2 抽象域译读

[^12]真实程序对象无限多，静态分析不能枚举每次运行。HScope 使用抽象内存对象表示“某代码位置可能创建的一组对象”，[^13]并把函数、类、对象、闭包和属性的可能值记录在抽象状态中。值集合保守合并：宁可包含多种可能，也不能随意漏掉一种。




### 3.3 抽象语义译读

对赋值、属性读写、对象创建、条件和[^15]函数调用，[^14]论文定义状态转移规则。执行到调用点时，系统读取调用变量当前可能指向的函数对象，[^16]动态补充调用图，而不是要求开始前已有完整调用图。返回时再把被调函数结果合并回调用者状态。

### 三个新术语

- **IR（Intermediate Representation，中间表示）**：统一的赋值、属性读写和调用等分析指令。
- **Abstract Object（抽象对象）**：代表一组可能运行时对象的有限符号。
- **On-the-fly Call Graph（按需调用图）**：分析到调用点时根据当前状态解析目标并加边。

[^17]### 批注：抽象解释不是模拟器

它不追求还原某个具体 OAID 或某次点击，而是计算所有允许状态的安全近似。[^18]代价是可能把不可能同时出现的值合并，形成误报。

### 最少记忆清单

- 字节码先变成 IR，IR 再被抽象解释。
- 抽象对象用有限状态覆盖许多真实对象。
- 调用图在分析过程中逐步形成。

### 主动回忆题

1. 为什么恢复词法环境对闭包很重要？
闭包是一个函数，不仅带着“自己要执行的代码”，还顺手把它创建时周围用到的变量一起记住了。比如一个函数被调用的时候传参进去，结果这个被调用的函数一直记得这个变量
2. 抽象值合并为什么带来误报却有助于避免漏报？
f可能是 A 也可能是 B，而HScope：f = {A,B}，好处： 真正运行的是 A 还是 B，都不容易漏掉。坏处：可能把一些现实中不能同时发生的情况也组合起来，于是产生不存在的路径。
3. 固定调用图与按需调用图有什么差别？
按需调用图：分析执行到调用点时，根据当前变量可能保存的函数，再决定“谁调用谁”。
### 进入下一单元检查点

- [x] 能按“包—反汇编—IR—抽象状态—调用图”复述流程。
- [x] 能解释一个抽象对象代表什么。
- [x] 首次阅读可以略过公式，但能说出每类公式在更新什么状态。

---

## 第四单元：第3.4—3.7节——上下文、框架与污点传播

第 3.1—3.3 解决的是“代码到底在干什么、谁可能调用谁”；第 3.4—3.7 解决的是“同一个函数在不同地方调用时要不要分开看，以及敏感数据到底怎么一路追到 Sink”。

### 3.4 上下文敏感引擎译读

同一函数从不同调用点进入时，参数来源可能完全不同。HScope 使用有[^20]限调用点上下文区分这些执行，避免把无关数据流混在一起。引擎还维护[^21]摘要以减少重复分析，并在状态变化时继续[^22]迭代直到稳定。

[^19]上下文敏感 = 同一个函数的不同调用来源，尽可能分开分析。

### 3.5 污点分析译读

[^23]系统把 Source API 返回值标上数据类别，例如[^24] OAID、位置或联系人。标签沿赋值、属性、参数、返回值与容器传播。若标签到达网络、日志、短信等 Sink，就产生候选路径。对 Map/Record 等复合结构，论文使用[^25]汇总近似，这有利于覆盖但可能污染兄弟字段。

### 3.6 隐私泄漏检测译读

[^26]HScope 使用可配置 JSON 策略定义 Source、Sink 和污点类别，并包含 [^27]OpenHarmony 框架模型：Ability 生命周期、Router 导航、事件回调以及共享 UI 状态。路径报告不仅说“命中”，还应保留调用和传播位置，供人工复核。

### 3.7 实现译读

原型以 Python 为主，并配合 tree-sitter 语法和 Ark 工具链。论文版本后来集成进 Lian，可通过指定 `abc` 语言启用。实现规模说明系统远不只是几条正则表达式。

### 三个新术语

- **Context-sensitive（上下文敏感）**：区分函数从不同调用位置进入的情况。
- **Flow-sensitive（流敏感）**：保留语句先后顺序对状态的影响。
- **Taint Summary（污点摘要）**：复用函数已经计算过的数据流效果。

### 最少记忆清单

- 污点标签只有建立在正确调用与 ICC 关系上才有意义。
- Source/Sink 策略和框架模型都是知识盲区来源。
- 上下文与流敏感提高精度，也增加时间和内存。

### 主动回忆题

1. 为什么同一工具函数需要区分两个调用点？
因为同一个函数，被不同地方调用时，传进去的数据可能完全不同。
2. 复合对象汇总近似会怎样产生误报？
3. 若策略漏掉一个新隐私 API，会造成误报还是漏报？

### 进入下一单元检查点

- [x] 能逐边解释图1的污点传播。
污点传播是在问“敏感数据怎么走”；  
流敏感是在问“先后顺序是什么”；  
上下文敏感是在问“这个函数是从哪里被调用进来的
- [ ] 能说出 HScope 的三个框架建模对象。
- [ ] 能区分通用程序语义与平台专用语义。

---

## 第五单元：第4节——实验怎样证明有效

### 数据与开销译读

作者构建 50 个用例、覆盖 11 类模式的 OH-Bench 微基准，并收集 300 个开源 OpenHarmony App。由于官方应用市场包存在加密和反嗅探限制，真实数据集并非直接抓取全部闭源商店 App。实验机为 i7-14700、32 GB RAM、Ubuntu 24.04；平均每个 App 分析 206 秒，平均内存 1,523 MB。

### 微基准结果译读

HScope 在 OH-Bench 上 precision 与 recall 均为 87.5%，F1 为 0.88。基线 ArkAnalyzer precision 为 81.8%、recall 为 22.5%、F1 为 0.35。主要提升来自对间接调用、共享状态与框架通信的处理。

### 真实 App 结果译读

HScope 报告 48 条候选泄漏路径，经人工检查确认 39 条可行，涉及 27 个 App。这里“确认”是研究者根据代码路径判断，不应把 39/48 直接推广为所有环境下的通用准确率。

### 消融与对比译读

论文分别移除调用上下文和框架建模，观察召回与误报变化。结果说明间接调用解析和框架专用别名关系是关键贡献；仅有通用污点传播不足以恢复真实 OpenHarmony 路径。

### 三个新术语

- **Micro-benchmark（微基准）**：为某种语言或框架特性专门设计的小测试用例。
- **F1-score（F1值）**：precision 与 recall 的调和平均。
- **Candidate Path（候选路径）**：工具报告、仍需人工判断可行性和风险含义的路径。

### 最少记忆清单

- OH-Bench 与 300 个真实 App 回答不同问题。
- 48 是工具候选，39 是人工确认路径。
- 平均开销适合离线分析，不表示手机端实时执行。

### 主动回忆题

1. 为什么微基准高分不能替代真实 App 测试？
2. ArkAnalyzer recall 低说明可能漏掉哪类关系？
真实存在的泄漏路径里，它有不少没找出来。
3. 39 条确认路径是否等于 39 个法律违规？

### 进入下一单元检查点

- [ ] 能解释 50、300、48、39、27 五个数字。
- [ ] 能说出实验机器和平均开销。
- [ ] 能解释消融实验怎样支持框架建模贡献。

---

## 第六单元：第5—7节——相关工作、局限与结论

### 相关工作译读

论文把自己放在三条研究线上：JavaScript/动态语言静态分析、Android 与移动隐私污点分析、OpenHarmony 程序分析。HScope 的差异是直接面向 Ark 字节码，并同时处理无类型对象、间接回调和 OpenHarmony ICC。

### 局限译读

反射、字符串动态导入和运行时生成属性仍可能让目标不可解析。Map/Record 汇总近似可能产生误报。有限上下文深度无法完全覆盖非常深或高度多态的回调。Source/Sink 策略和框架模型也需要持续维护。

### 结论译读

HScope 说明 OpenHarmony 隐私分析不能只复制 Android 工具。只有恢复字节码语义、调用关系和组件数据关系，污点分析才能给出更完整的路径。真实 App 测量显示该方法具有实际价值，但路径仍需人工和动态证据进一步确认。

### 三个新术语

- **Reflection（反射）**：运行时根据字符串等信息决定类型或方法。
- **Soundness（健全性目标）**：尽量不漏掉真实行为的分析追求；现实工具常需折中。
- **False Positive（假阳性）**：工具报告但实际不可行或不构成目标风险的候选。

### 最少记忆清单

- HScope 的优势与盲区都来自静态近似。
- 框架升级会要求模型和策略持续更新。
- 最可靠结论来自静态路径、动态事件和人工复核的组合。

### 主动回忆题

1. 反射为什么会同时影响调用图和污点传播？
2. 怎样用 Appium/Hook 验证 HScope 的候选路径？
3. HScope 可怎样为 DIULENS 的 GUI 探索提供目标？

### 完成检查点

- [ ] 能用三分钟讲清“字节码—IR—抽象解释—调用图—ICC—污点”。
- [ ] 能沿图1讲清每条数据边和控制边。
- [ ] 能解释实验数字的样本与分母。
- [ ] 能说出至少三项局限。
- [ ] 能明确静态路径能证明什么、不能证明什么。

## 课堂用一句话

HScope 的关键不是把 Source 和 Sink 列出来，而是先恢复 OpenHarmony 无类型字节码中的对象、回调和组件关系，再让污点标签沿这些可验证关系传播。

[^1]: 你看到的“**字节码丢失类型**”如果是在 Java / Android / 静态分析 / 反编译场景里，通常不是说“字节码整个坏了”，而是说：
	
	> **从字节码里已经无法完整恢复源代码中原本的类型信息。**
	
	最典型的原因就是：**Java 编译成字节码以后，有些只服务于源代码阶段的信息会被擦掉、简化，或者根本不会保留下来。**
	
	比如你写：
	
	```java
	List<String> names = new ArrayList<>();
	List<Integer> ages = new ArrayList<>();
	```
	
	在 Java 源码阶段，编译器知道：
	
	```text
	names 是 List<String>
	ages  是 List<Integer>
	```
	
	但是 Java 的泛型采用的是**类型擦除 Type Erasure**。
	
	编译后的字节码在很多地方，本质上只把它们看成：
	
	```text
	List
	List
	```
	
	也就是：
	
	```text
	List<String>
	        ↓ 编译
	List
	
	List<Integer>
	        ↓ 编译
	List
	```
	
	所以我们就可以说：
	
	**`String` / `Integer` 这一部分类型信息在运行时字节码层面被“擦除”了。**
	
	---
	
	再举一个更容易理解的例子。
	
	源码：
	
	```java
	public String getName() {
	    return "Tom";
	}
	```
	
	字节码里方法描述符会明确保存：
	
	```text
	()Ljava/lang/String;
	```
	
	所以 `String` 这个返回值类型**没有丢失**。
	
	但如果是：
	
	```java
	public List<String> getNames() {
	    return names;
	}
	```
	
	JVM 真正执行这个方法时，更核心的类型描述基本是：
	
	```text
	()Ljava/util/List;
	```
	
	也就是说 JVM 真正关心的是：
	
	```text
	返回一个 List
	```
	
	而不是：
	
	```text
	返回一个 List<String>
	```
	
	虽然现代 `.class` 文件里可能额外通过 `Signature` 属性保存部分泛型信息：
	
	```text
	Ljava/util/List<Ljava/lang/String;>;
	```
	
	但这个东西属于**额外元数据**，并不是 JVM 执行字节码必须依赖的信息。
	
	因此如果经过：
	
	- 混淆
	    
	- 优化
	    
	- dex 转换
	    
	- 某些编译流程
	    
	- 元数据删除
	    
	
	这些 `Signature` 信息被去掉以后，反编译器可能只能恢复：
	
	```java
	List
	```
	
	而恢复不了：
	
	```java
	List<String>
	```
	
	这就是一种典型的**类型信息丢失**。
	
	---
	
	### 在 Android / APK 分析里更常见
	
	你现在如果是在做 APK、SDK 或 JADX 分析，那么“类型丢失”通常还可能指另外一种情况。
	
	例如 JADX 反编译出来：
	
	```java
	Object obj = ...
	```
	
	但实际上源码可能原本是：
	
	```java
	UserInfo userInfo = ...
	```
	
	为什么最后变成 `Object`？
	
	因为 DEX / 字节码分析过程中，JADX可能没有足够的信息推断：
	
	```text
	这个寄存器到底是什么具体类型？
	```
	
	例如原始代码：
	
	```java
	User user = getUser();
	user.getName();
	```
	
	经过优化以后可能变成类似：
	
	```text
	v0 = invoke getUser()
	invoke v0.getName()
	```
	
	静态分析器需要根据：
	
	```text
	方法返回值
	调用关系
	字段类型
	cast
	控制流
	```
	
	推断 `v0` 是什么。
	
	如果部分信息缺失，最终可能只能推成：
	
	```text
	java.lang.Object
	```
	
	甚至显示：
	
	```java
	Object
	```
	
	或者 JADX 出现类似：
	
	```text
	/* JADX WARN: Type inference failed */
	```
	
	这也经常被称作：
	
	**类型推断失败 / 类型信息丢失。**
	
	---
	
	你可以把整个过程想成：
	
	```text
	Java 源代码
	↓
	信息非常丰富
	
	User user = new User();
	List<String> names;
	变量名、泛型、注释、源码结构……
	↓
	javac / kotlinc
	↓
	.class 字节码
	↓
	部分源码信息已经丢失
	↓
	D8 / R8
	↓
	DEX
	↓
	可能又经过优化 + 混淆
	↓
	更多信息丢失
	↓
	JADX
	↓
	尝试“反推”回 Java
	```
	
	关键点在于：
	
	> **反编译不是解压源码，而是在根据字节码猜源码。**
	
	因此只要编译的时候某些信息没有进入字节码，后面就**不可能 100% 恢复**。
	
	比如原代码：
	
	```java
	User currentUser = getUser();
	```
	
	如果变量名没保存，反编译可能得到：
	
	```java
	User user = getUser();
	```
	
	甚至：
	
	```java
	Object obj = getUser();
	```
	
	这里其实发生了两种不同的信息损失：
	
	```text
	currentUser
	↓
	变量名信息丢失
	↓
	user / obj
	
	User
	↓
	类型信息不足
	↓
	Object
	```
	
	---
	
	### 但要特别区分三件事
	
	以后你在字节码分析中看到“类型丢失”，最好先判断是哪一种：
	
	|情况|示例|原因|
	|---|---|---|
	|泛型类型丢失|`List<String>` → `List`|Type Erasure|
	|具体类型无法推断|`User` → `Object`|字节码/DEX类型信息不足|
	|类本身找不到|`com.xxx.User` 无法解析|依赖缺失 / Phantom Class|
	
	第三种在 **Soot、FlowDroid、WALA、静态程序分析**里尤其常见。
	
	比如你的程序引用：
	
	```java
	com.google.firebase.analytics.FirebaseAnalytics
	```
	
	但是你分析时没有把 Firebase SDK 的 jar / dex 提供给分析工具。
	
	于是分析器知道：
	
	```text
	这里应该存在一个 FirebaseAnalytics
	```
	
	但它找不到这个类的真正定义。
	
	这种通常叫：
	
	```text
	Missing Class
	Missing Type
	Unresolved Type
	Phantom Class
	```
	
	这和“编译过程中类型擦除”其实是**两种完全不同的问题**。
	
	如果你是在你现在做的 **Android SDK / JADX / Soot / FlowDroid / 字节码分析**里看到“字节码丢失类型”这个词，把出现它的那段原文、报错或者截图发给我，我可以直接结合你的场景告诉你它具体属于上面哪一种。

[^2]: 在 OpenHarmony 的 ArkTS/JavaScript 程序里，程序真正运行时“哪个函数会调用哪个函数”，往往比传统 Java 程序更灵活、更难提前完全确定。  
	所以如果静态分析一开始就把调用图固定死，再沿着这个图传播污点，可能漏掉实际运行时存在的调用路径。

[^3]: ## 2. “函数可作为值传递”是什么意思？
	
	这是最关键的一点。
	
	普通初学 Java 时你可能习惯：
	
	```
	a.doSomething();
	```
	
	看到这一行，很容易知道：
	
	```
	这里调用了 doSomething()
	```
	
	但 JS / ArkTS 可以直接把**函数本身当成一个变量**。
	
	例如：
	
	```
	function sendData(data) {
	    console.info(data)
	}
	
	let f = sendData
	
	f("hello")
	```
	
	注意：
	
	```
	f
	```
	
	不是普通数字或字符串。
	
	它里面装的是一个**函数**。
	
	所以：
	
	```
	f("hello")
	```
	
	真正调用的是：
	
	```
	sendData("hello")
	```
	
	---
	
	更麻烦的是：
	
	```
	let f
	
	if (condition) {
	    f = sendToServer
	} else {
	    f = saveToFile
	}
	
	f(data)
	```
	
	静态分析器看到最后：
	
	```
	f(data)
	```
	
	问题来了：
	
	> `f` 到底是谁？
	
	可能是：
	
	```
	sendToServer
	```
	
	也可能是：
	
	```
	saveToFile
	```
	
	于是调用图不能简单写成：
	
	```
	当前函数 → 某一个确定函数
	```
	
	而可能是：
	
	```
	当前函数
	  ├─→ sendToServer
	  └─→ saveToFile
	```
	
	这就是为什么这类语言的调用图构建更麻烦。

[^4]: ## 1. “字节码层可能丢失类型”到底是什么意思？
	
	假设源码里有：
	
	```
	function processUser(user: User) {
	    ...
	}
	```
	
	源码层我们很清楚：
	
	```
	user 的类型 = User
	```
	
	但编译成 Ark 字节码以后，分析器拿到的未必还能完整看到：
	
	```
	这里一定是 User
	```
	
	有时只能知道：
	
	```
	这是一个对象
	```
	
	或者需要靠上下文重新推断。
	
	你可以理解成：
	
	```
	ArkTS 源码：
	
	User user
	
	↓ 编译
	
	Ark 字节码：
	
	某个寄存器 v3 里放了一个对象
	```
	
	如果类型信息不够完整，那么静态分析器看到：
	
	```
	v3.xxx()
	```
	
	就可能不知道：
	
	> 这个 `xxx()` 到底是哪个类的哪个方法？
	
	这就会影响**调用图**。

[^5]: # 3. “闭包”为什么又会增加难度？
	
	看这个：
	
	```
	function createSender(token: string) {
	
	    return function(data: string) {
	        send(token, data)
	    }
	}
	```
	
	然后：
	
	```
	let sender = createSender("abc123")
	
	sender("location")
	```
	
	这里 `sender` 不仅仅是一个函数。
	
	它还偷偷“记住”了：
	
	```
	token = "abc123"
	```
	
	这就是**闭包 Closure**。
	
	可以想象成：
	
	```
	sender
	  │
	  ├── 函数代码
	  │     send(token, data)
	  │
	  └── 捕获的环境
	        token = abc123
	```
	
	所以分析数据流的时候，不能只问：
	
	> 参数 data 从哪里来？
	
	还要问：
	
	> 这个函数捕获的 token 从哪里来的？

[^6]: ## 4. 回调为什么重要？
	
	OpenHarmony / ArkTS 中非常常见：
	
	```
	button.onClick(() => {
	    uploadLocation()
	})
	```
	
	代码表面顺序是：
	
	```
	注册回调
	↓
	程序继续运行
	```
	
	但 `uploadLocation()` 并不是现在调用。
	
	而是：
	
	```
	用户点击按钮
	↓
	系统触发回调
	↓
	uploadLocation()
	```
	
	于是调用关系不是简单：
	
	```
	A()
	↓
	B()
	↓
	C()
	```
	
	而是：
	
	```
	A()
	↓
	注册 callback
	↓
	A()结束
	
	       ……过了一会……
	
	系统事件
	↓
	callback()
	↓
	uploadLocation()
	```
	
	这叫：
	
	**event-driven / callback-driven execution**
	
	也就是事件驱动。

[^7]: # 5. “共享状态”又是什么意思？
	
	例如：
	
	```
	let globalData
	
	function readData() {
	    globalData = getDeviceId()
	}
	
	function upload() {
	    sendToServer(globalData)
	}
	```
	
	假设：
	
	```
	getDeviceId()
	```
	
	是 Source。
	
	```
	sendToServer()
	```
	
	是 Sink。
	
	真正的数据路径其实是：
	
	```
	getDeviceId()
	     │
	     ↓
	 globalData
	     │
	     ↓
	sendToServer()
	```
	
	但 Source 和 Sink 根本不在同一个函数：
	
	```
	readData()
	
	upload()
	```
	
	中间靠的是：
	
	```
	共享变量 globalData
	```

[^8]: # 6. 现在你就能理解“先建立固定调用图再做污点传播容易漏掉真实调用”了
	
	先说**调用图 Call Graph**是什么。
	
	假设：
	
	```
	function A() {
	    B()
	}
	
	function B() {
	    C()
	}
	```
	
	那么调用图就是：
	
	```
	A
	│
	↓
	B
	│
	↓
	C
	```
	
	这是非常固定的。
	
	于是污点分析可以很轻松：
	
	```
	Source
	 ↓
	A
	 ↓
	B
	 ↓
	C
	 ↓
	Sink
	```
	
	---
	
	但是 ArkTS 里可能是：
	
	```
	function A(callback) {
	    callback(data)
	}
	```
	
	谁调用？
	
	不知道。
	
	外面可能：
	
	```
	A(upload)
	```
	
	也可能：
	
	```
	A(save)
	```
	
	甚至：
	
	```
	A(condition ? upload : save)
	```
	
	于是你如果在最开始草率建立：
	
	```
	A → save
	```
	
	然后把这个调用图**固定不再修改**：
	
	```
	A
	↓
	save
	```
	
	那么真实运行时如果：
	
	```
	A
	↓
	upload
	```
	
	你的污点分析根本不会走到 `upload`。
	
	于是：
	
	```
	敏感数据
	 ↓
	A
	 ↓
	upload
	 ↓
	网络
	```
	
	就被漏掉了。
	
	这就是这句话：
	
	> **先建立固定调用图再做污点传播容易漏掉真实调用。**

[^9]: # 7. 最后一句：“只在单函数内找 Source 和 Sink 更会漏掉跨组件路径”
	
	这个其实最好理解。
	
	假设：
	
	```
	function getInfo() {
	    return getLocation()
	}
	```
	
	这里有 Source：
	
	```
	getLocation()
	```
	
	但是没有 Sink。
	
	另一个函数：
	
	```
	function upload(info) {
	    http.post(info)
	}
	```
	
	这里有 Sink：
	
	```
	http.post()
	```
	
	但是没有 Source。
	
	如果你的工具是：
	
	> 每个函数单独检查：有没有同时出现 Source 和 Sink？
	
	那么：
	
	```
	getInfo()
	
	Source ✅
	Sink   ❌
	
	upload()
	
	Source ❌
	Sink   ✅
	```
	
	于是工具得出：
	
	```
	没有隐私泄漏
	```
	
	但真实程序：
	
	```
	let info = getInfo()
	
	upload(info)
	```
	
	实际上：
	
	```
	getLocation()
	      ↓
	    info
	      ↓
	  upload(info)
	      ↓
	 http.post()
	```
	
	显然是：
	
	```
	Source → Sink
	```
	
	只是跨了函数。
	
	---
	
	## 再跨组件就更明显
	
	比如 OpenHarmony：
	
	```
	组件 A
	PageAbility / UIAbility
	
	读取设备 ID
	    ↓
	写入共享状态
	    ↓
	
	组件 B
	Service / Extension
	
	读取共享状态
	    ↓
	网络上传
	```
	
	真正路径：
	
	```
	Device ID
	   ↓
	组件 A
	   ↓
	共享对象 / Storage / Event
	   ↓
	组件 B
	   ↓
	HTTP
	```
	
	所以你不能只检查：
	
	```
	组件 A 内部
	```
	
	或者：
	
	```
	组件 B 内部
	```
	
	而要进行：
	
	**跨函数 + 跨组件的数据流分析。**

[^10]: 在**污点分析（Taint Analysis）**里，`Source` 和 `Sink` 是最核心的两个词。
	
	你可以先记成一句话：
	
	> **Source = 敏感数据从哪里产生；Sink = 敏感数据最终被送到哪里。**
	
	比如：
	
	```
	let location = getLocation();
	http.post("https://example.com", location);
	```
	
	这里：
	
	```
	getLocation()
	```
	
	就是 **Source**，因为它产生了敏感数据“位置”。
	
	而：
	
	```
	http.post(...)
	```
	
	就是 **Sink**，因为数据通过它被发送到网络。
	
	所以整个污点路径是：
	
	```
	Source
	getLocation()
	    ↓
	location
	    ↓
	Sink
	http.post()
	```
	
	---
	
	### Source 具体是什么？
	
	Source 可以理解为“敏感信息入口”。
	
	Android / OpenHarmony 分析中常见的 Source 有：
	
	- 获取位置：GPS、经纬度
	- 获取设备标识符
	- 获取联系人
	- 获取手机号
	- 获取剪贴板
	- 获取相册、文件
	- 获取麦克风数据
	- 获取账号信息
	
	比如：
	
	```
	let id = getDeviceId();
	```
	
	如果 `getDeviceId()` 被规则定义为敏感 API，那么：
	
	```
	getDeviceId() = Source
	```
	
	此后 `id` 这个变量就可以被标记成“**污点数据**”。
	
	---
	
	### Sink 又是什么？
	
	Sink 可以理解为：
	
	> 数据到了这里，就可能“离开当前安全边界”或者执行敏感操作。
	
	最经典的 Sink 是网络发送：
	
	```
	http.post(url, data);
	```
	
	那么：
	
	```
	http.post = Sink
	```
	
	但 Sink 不一定只有网络。
	
	比如：
	
	- HTTP 上传
	- Socket 发送
	- 写文件
	- 写日志
	- 写数据库
	- IPC 跨进程发送
	- 发短信
	- 某些第三方 SDK 上传接口
	
	具体哪个 API 算 Sink，要看你的分析目标。
	
	---
	
	### 为什么叫“Source”和“Sink”？
	
	这两个词其实很形象。
	
	`Source` 本身就是：
	
	> 源头、水源
	
	`Sink` 可以理解成：
	
	> 汇集处、流向的终点
	
	所以你可以把敏感数据想象成一股水：
	
	```
	        Source
	       数据源头
	          ↓
	      ┌────────┐
	      │敏感数据│
	      └────────┘
	          ↓
	        变量 A
	          ↓
	        函数 B
	          ↓
	        对象 C
	          ↓
	         Sink
	       数据出口
	```
	
	污点分析本质上就在问：
	
	> **这股“敏感数据的水”，能不能从 Source 流到 Sink？**
	
	---
	
	例如下面这段：
	
	```
	let phone = getPhoneNumber();
	
	let userInfo = {
	    phoneNumber: phone
	};
	
	upload(userInfo);
	```
	
	假设：
	
	```
	getPhoneNumber() = Source
	upload() = Sink
	```
	
	分析工具就需要追：
	
	```
	getPhoneNumber()
	      ↓
	    phone
	      ↓
	userInfo.phoneNumber
	      ↓
	   userInfo
	      ↓
	   upload()
	```
	
	最终发现：
	
	```
	Source → Sink
	```
	
	那么就可能报告：
	
	> 手机号可能被上传到网络。
	
	---
	
	但要注意：
	
	**发现 Source → Sink 不等于一定存在安全漏洞。**
	
	比如：
	
	```
	let location = getLocation();
	
	if (userConsent) {
	    upload(location);
	}
	```
	
	静态污点分析可能找到：
	
	```
	getLocation()
	↓
	upload()
	```
	
	但程序实际上可能已经获得用户授权。
	
	所以污点分析告诉你的通常是：
	
	> **存在一条潜在的数据传播路径。**
	
	接下来还需要分析：
	
	- 有没有用户同意
	- 数据有没有脱敏
	- 是否真的执行
	- 上传给谁
	- 是否符合隐私政策
	
	---
	
	你还可以顺便记住第三个词：
	
	**Sanitizer（净化器）**。
	
	比如：
	
	```
	let id = getDeviceId();      // Source
	
	let hashId = hash(id);       // Sanitizer
	
	http.post(url, hashId);      // Sink
	```
	
	于是：
	
	```
	Source
	  ↓
	设备 ID
	  ↓
	hash()
	  ↓
	哈希后的 ID
	  ↓
	Sink
	```
	
	`hash()` 有时候会被视为一个 Sanitizer，也就是对敏感数据进行处理，降低其敏感性。不过是否真的算“净化”，取决于分析规则和威胁模型。
	
	所以三个词可以一起记：
	
	```
	Source
	数据从哪里来
	   ↓
	
	Propagation
	数据怎么传播
	   ↓
	
	Sanitizer
	中间有没有处理
	   ↓
	
	Sink
	数据最终去了哪里
	```
	
	如果你现在是在学污点分析，最重要的一句话就是：
	
	> **Source 是污点的起点，Sink 是我们关心的终点，污点分析就是判断 Source 的数据能否沿程序的数据流传播到 Sink。**

[^11]: “回调函数”这个名字也很好理解：
	
	> **回头再调用的函数。**
	
	不过更准确一点是：
	
	> **把一个函数传给另一个函数或系统，让对方在合适的时候调用它。**

[^12]: 把很多真实对象用一个“代表”来表示。
	例如：
	
	```
	代码第100行：
	
	new User()
	```
	
	分析器可能创建一个：
	
	```
	抽象对象 O100
	```
	
	它的含义不是：
	
	> 第100个 User。
	
	而是：
	
	> **所有可能由第100行 `new User()` 创建出来的 User，我统一用 O100 表示。**
	这就是：
	
	### Abstract Object
	
	用一个假想对象代表很多真实运行时对象。

[^13]: 假设当前代码：
	
	```
	let x = ...
	let f = ...
	let obj = ...
	```
	
	HScope需要记：
	
	```
	x 可能是什么？
	f 可能是什么函数？
	obj 可能是什么对象？
	某个属性可能是什么？
	```
	
	于是建立一张“当前情况记录表”：
	
	```
	x   → {OAID, 普通字符串}
	
	f   → {upload函数, save函数}
	
	obj → {对象O1, 对象O2}
	```
	
	这整张表就可以粗略理解为：
	
	> **抽象状态。**
	
	它不是说真实运行的时候：
	
	```
	f 同时等于 upload 和 save
	```
	
	而是说：
	
	> 我现在无法确定它是哪一个，所以把两个可能性都保存下来。

[^14]: 告诉 HScope：每看到一种代码，脑子里的那张“当前状态表”应该怎么改。

[^15]: 假设：
	
	```
	f(data)
	```
	
	以前你已经学过这个问题。
	
	HScope不会简单认为：
	
	```
	f → 某个固定函数
	```
	
	而是看当前抽象状态：
	
	```
	当前 f 可能是什么？
	```
	
	比如现在记录的是：
	
	```
	f → {upload, save}
	```
	
	那么执行到：
	
	```
	f(data)
	```
	
	的时候，才加入：
	
	```
	当前函数
	  ├──→ upload
	  └──→ save
	```
	
	这就是：
	
	> **On-the-fly Call Graph**
	
	按需建立调用图。

[^16]: HScope的方式：
	
	```
	开始：
	A
	
	分析到 f()
	↓
	当前状态发现：
	f 可能是 B
	↓
	补：
	A → B
	
	继续分析 B
	↓
	又发现 callback 可能是 C
	↓
	补：
	B → C
	```
	
	所以调用图像一张：
	
	> **边走边画的地图。**

[^17]: 真正模拟器运行一次：
	
	```
	condition = true
	f = upload
	OAID = "123456"
	```
	
	它看到的是：
	
	> **这一轮真实发生了什么。**
	
	而抽象解释看到：
	
	```
	condition = {true,false}
	
	f = {upload,save}
	
	OAID = {某个敏感字符串}
	```
	
	它根本不关心真实 OAID 是：
	
	```
	123456789
	```
	
	还是：
	
	```
	987654321
	```
	
	它只关心：
	
	> **这里可能产生“一个 OAID 类型的敏感数据”。**
	
	所以：
	
	### 模拟器 / 真运行
	
	```
	这次到底发生什么？
	```
	
	### 抽象解释
	
	```
	所有可能发生的情况，大概有哪些？
	```
	
	这就是区别。

[^18]: 例如现实程序：
	
	```
	if (x > 0) {
	    f = upload
	}
	
	if (x < 0) {
	    data = OAID
	}
	```
	
	真实情况下：
	
	```
	x不可能同时 >0 和 <0
	```
	
	所以实际上可能不存在：
	
	```
	OAID → upload
	```
	
	但如果抽象分析把信息合并得比较粗：
	
	```
	f可能 = upload
	data可能 = OAID
	```
	
	就可能错误组合成：
	
	```
	OAID → upload
	```
	
	于是报告一个实际上走不通的路径。
	
	这就是：
	
	> **为了少漏报而保守合并，可能产生误报。**
	
	这句话以后你看静态分析论文会一直遇到：
	
	```
	更保守
	↓
	Recall通常更好
	↓
	但可能引入False Positive
	```

[^19]: ### 为什么这能减少误报？
	
	例如：
	
	```
	function process(x) {
	    upload(x)
	}
	```
	
	一个地方：
	
	```
	process("天气很好")
	```
	
	另一个地方：
	
	```
	process(getOAID())
	```
	
	如果全部合并：
	
	```
	process.x = {普通字符串, OAID}
	```
	
	分析器以后可能到处觉得：
	
	```
	process()
	→ 可能发送 OAID
	```
	
	上下文敏感则能分清：
	
	```
	调用点1：
	普通数据 → upload
	
	调用点2：
	OAID → upload
	```
	
	结论更准确。

[^20]: 如果程序：
	
	```
	A → B → C → D → E → ...
	```
	
	分析器如果把完整调用历史全部记下来：
	
	```
	A从哪里来
	B从哪里来
	C从哪里来
	D从哪里来
	……
	```
	
	信息可能越来越多，内存爆炸。
	
	所以通常只记最近有限几个调用点。
	
	例如只记最近 2 层：
	
	```
	当前正在分析 C
	
	上下文：
	A → B → C
	```
	
	可能只保存：
	
	```
	B → C
	```
	
	你暂时只需要理解：
	
	> **上下文越详细，通常越准确，但越耗时间和内存。**

[^21]: 把一个函数以前算过的“污点怎么进、怎么出”的结果记下来，下次直接复用。假设程序有 100 个地方调用：
	
	```
	sanitize(x)
	```
	
	如果每次都从头把 `sanitize()` 分析一遍，很浪费。
	
	所以可以提前记住：
	
	```
	sanitize 的作用：
	
	输入 x
	  ↓
	执行某些操作
	  ↓
	输出 y
	```
	
	这个“函数做了什么”的精简结果就是：
	
	> **Summary，摘要。**
	
	在污点分析里，可以记录类似：
	
	```
	如果参数1有污点
	    ↓
	返回值也有污点
	
	如果参数2有污点
	    ↓
	不会影响返回值
	```
	
	以后再调用这个函数，就不用重新完整跑一次。

[^22]: 假设：
	
	```
	A → B
	B → C
	C → A
	```
	
	出现循环调用。
	
	第一次分析 A，发现一些信息。
	
	分析 B 后，又得到新信息。
	
	这些新信息又可能改变 A。
	
	于是：
	
	```
	分析 A
	↓
	分析 B
	↓
	发现新信息
	↓
	重新更新 A
	↓
	又更新 B
	↓
	……
	```
	
	直到某一次：
	
	```
	再算一遍
	↓
	已经没有任何新东西
	```
	
	这就叫：
	
	> **达到稳定点 / fixed point。**
	
	人话：
	
	> **一直算，算到继续算也不会发现新情况为止。**

[^23]: 敏感数据被复制、装进对象、传给函数、从函数返回时，那个“这是 OAID”的标记也跟着走。

[^24]: 不是只标：
	
	```
	tainted / not tainted
	```
	
	还可以标成：
	
	```
	OAID
	Location
	Contacts
	PhoneNumber
	……
	```
	
	这样报告才能说：
	
	```
	OAID → network
	
	Location → log
	```
	
	而不是含糊地说：
	
	```
	“某种敏感数据泄漏了”
	```

[^25]: 只要 user 里面某个字段脏了 ↓ 整个 user 都当成可能脏，可以理解为：
	
	```
	真实情况：
	
	user
	├── name → clean
	└── id   → OAID
	```
	
	粗略分析：
	
	```
	user → OAID
	├── name → 也被认为可能 OAID ❌
	└── id   → OAID ✅
	```
	
	所以它：
	
	> **不容易漏掉，但可能误报。**

[^26]: 分析能力再强，如果你的 Source/Sink 知识库不完整，也照样会漏。HScope 本身并不是天生就知道：
	
	```
	getOAID()
	```
	
	是敏感 API。
	
	也不知道：
	
	```
	http.post()
	```
	
	是危险出口。
	
	需要有人提供规则。
	
	例如 JSON 策略概念上可以类似：
	
	```
	Source:
	getOAID → OAID
	getLocation → Location
	
	Sink:
	http.post → Network
	console.log → Log
	```
	
	于是 HScope 才知道：
	
	```
	看到 getOAID()
	↓
	贴 OAID 标签
	
	看到 http.post()
	↓
	检查参数有没有污点
	```

[^27]: 但 OpenHarmony 还有很多平台自己的特殊规则。
	
	例如：
	
	```
	@Link message
	```
	
	通用语言分析器未必知道：
	
	> 这是共享状态关系。
	
	还有：
	
	```
	Button().onClick(callback)
	```
	
	它可能只知道：
	
	> 把一个函数传进去了。
	
	但不知道：
	
	> 用户点击之后，OpenHarmony 框架会调用它。
	
	所以 HScope 需要额外告诉分析器这些“平台规则”。
	
	你这份材料列出的例子实际上至少包括：
	
	```
	Ability 生命周期
	Router 导航
	事件回调
	共享 UI 状态
	```
	
	例如生命周期：
	
	```
	系统
	 ↓
	onCreate()
	 ↓
	onForeground()
	```
	
	代码里可能根本没有：
	
	```
	onCreate()
	```
	
	被普通函数主动调用。
	
	而是：
	
	> **框架偷偷帮你调用。**
	
	所以不建模，控制图就缺边。
