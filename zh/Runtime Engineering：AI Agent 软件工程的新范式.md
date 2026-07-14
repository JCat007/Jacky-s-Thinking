# Runtime Engineering：AI Agent 软件工程的新范式

## 前言 ｜AI 软件正在进入 Runtime 时代


过去三年，大语言模型以前所未有的速度重构了软件开发的底层逻辑。

行业发展初期，市场与开发者的核心焦点集中在模型本身：更大的参数规模、更强的通用推理能力、更长的上下文窗口，构成了大模型赛道的核心竞争维度。开发者深耕提示词优化、上下文调校，试图通过精细化的 Prompt 与 Context 设计，让模型输出更精准、更智能的结果。彼时行业形成了普遍共识：只要模型的智能足够强大，AI 就能持续攻克越来越复杂的业务问题。

但随着 AI 应用从被动应答的聊天机器人，迭代为可自主规划、自主执行、自主迭代的 AI Agent，一个全新的行业现实逐渐浮出水面：​真正限制 Agent 落地效果与生产稳定性的核心瓶颈，早已不是模型能力，而是 Runtime（运行时）。

越来越多的团队发现，同样一个模型，在不同的 Agent 系统中，表现可能截然不同。

优质的生产级 Agent 能够连续运行数小时，自主拆解复杂任务、完成数百次工具调用、自动捕获并修复执行异常、调度多个子 Agent 协同作业，闭环完成全流程软件开发、科研调研等复杂场景。而普通 Agent 往往会因一次 API 超时、一段格式异常的 JSON、一次工具调用失败，导致整体任务直接中断、全盘重启。

造成这种差距的核心变量，与模型本身无关，而是​模型的运行环境​。

AI 行业的核心竞争逻辑，已经完成一次关键迁移：​从模型竞争，走向 Runtime 竞争​。

大语言模型为 AI 赋予了核心智能，而 Runtime 为这份智能赋予了持续、稳定、可靠的落地能力。

回顾传统软件工程，绝大多数软件都是​确定性程序​。代码逻辑由开发者预先定义，输入与输出一一对应，控制流清晰可追溯、结果可精准预测。传统软件工程的核心工作，聚焦于代码架构设计、工程规范落地、依赖管理与系统运维，核心目标是保障预设逻辑的稳定执行。

而 AI Agent 软件彻底颠覆了这一范式。系统的核心决策不再完全依赖人工编写的固定代码，而是由具备概率性输出的大语言模型动态生成。开发者不再逐行定义程序的每一步执行逻辑，而是聚焦于目标定义、约束规则设计与运行环境搭建，交由模型在运行过程中自主完成规划、决策、执行与迭代修正。

这是软件工程发展史上的首次突破：软件系统拥有了强大但非确定性的“自主大脑”，也带来了前所未有的工程挑战：如何让本质上概率化、非确定性的智能系统，在真实生产环境中稳定、可靠、安全、持续地运行？

这已经不是 Prompt Engineering 能够回答的问题，也不仅仅是 Context Engineering 能够解决的问题，它需要一套全新的 Runtime Engineering（运行时工程）。

当前，OpenAI、Anthropic、Google、Microsoft、Cognition、Factory AI 等行业头部企业，均将核心工程资源倾斜至同一赛道：Agent Runtime。各家产品形态、技术实现虽有差异，但本质目标高度统一：围绕大模型搭建一套完整的 Runtime 基础设施，实现状态管理、工具调度、异常恢复、权限治理、结果校验、全链路观测与动态优化，保障 Agent 长期稳定运行。

在此背景下，Harness Engineering（驾驭工程）成为行业高频核心概念。需要明确的是，Harness 既不是模型、不是提示词，也不是上下文，而是一套全新的 AI 工程思想：​模型负责思考推理，Runtime 负责让思考落地、持续、稳定工作。

本文不止于解读 Harness Engineering 的技术细节，更致力于回答一个核心行业命题：为什么 AI Agent 的规模化落地，必然催生 Runtime Engineering 这一软件工程新范式？

读懂这一逻辑，便能厘清未来 Agent 架构的核心演进方向，也能理解行业共识：​未来 AI 软件的核心竞争力，不取决于是否拥有超大参数模型，而取决于是否拥有成熟、可靠、可迭代的 Runtime 基础设施​。

### 0.1 本文核心研究问题

当前行业对 Agent Runtime 存在大量认知混淆：有人将其等同于 Prompt 升级，有人将其等价于 Context，也有人将所有 Agent 基础设施统称为 Harness。为厘清概念边界、建立体系化认知，本文将围绕五大核心问题展开论述：

1. 为什么 AI Agent 必然迈入 Runtime 驱动的新时代？
2. Agent Runtime 的核心定义是什么？Harness Engineering 在 Runtime 体系中承担何种角色？
3. Prompt、Context、Harness、Runtime 四者的层级关系与核心边界是什么？
4. 生产级 Agent Runtime 必须具备哪些核心工程能力？
5. Runtime Engineering 为何会成为 AI 软件工程的主流新范式？

通过全文梳理，读者将系统性掌握 Harness Engineering 的核心逻辑，并从软件工程演进的宏观视角，理解 Runtime 成为 AI Agent 核心基础设施的底层必然性。

## 第一章 ｜ 什么是 Agent Runtime：让模型真正落地工作的核心层级

大模型普及初期，行业通用的 AI 应用架构极简且单一，完全适配聊天机器人的交互模式：

```
Application
      │
      ▼
     LLM
```

应用接收用户请求、传递给大模型，模型生成结果后直接返回用户，单次推理、单次应答，流程闭环且无后续链路。

基于这一简单架构，很多人自然地认为：Agent 只是在基础 LLM 之上，叠加了 Prompt 模板、工具调用与简易工作流的升级版应用。

然而，当 Agent 开始承担越来越复杂的任务之后，人们逐渐发现，这种理解已经无法解释现实中的 AI 系统。一个能够连续运行数小时的软件开发 Agent，一个能够自主浏览网页、分析数据、调用 API、管理多个子任务的 Research Agent，它们真正运行的过程，远比一次模型推理复杂得多。

事实上，在任何一个生产级 Agent 中，模型真正参与工作的时间，可能只占整个生命周期的一小部分。

绝大多数时间，系统都在执行另一类工作：任务状态持续维护、多轮工具调度、外部系统结果等待、上下文窗口动态管理、长期记忆读写、执行结果合法性校验、异常捕获与容错恢复、任务续跑与终止决策。这些工作，并不是模型完成的，它们共同构成了 AI Agent 的 Runtime 体系。

### 1.1 Runtime：计算机系统的经典核心概念

Runtime 并非 AI 时代的新生概念，而是贯穿现代计算机体系的核心基础设施。几乎所有主流软件、程序、服务，都依托专属 Runtime 运行：

```
Java 程序依托 JVM（Java 虚拟机）Runtime 运行；
JavaScript 依托浏览器、Node.js Runtime 运行；
Python 依托 CPython Runtime 运行；
容器化服务依托 Container Runtime 运行。
```

Runtime 的核心职责不包含具体业务逻辑，其唯一使命是：为程序提供稳定、持续、可管控的运行环境，保障程序生命周期完整可控。

通用 Runtime 的核心能力体系标准化且通用，涵盖：生命周期管理、内存资源调度、状态持久化维护、异常捕获与恢复、安全隔离、IO 管理、全链路可观测性。开发者编写的业务逻辑，本质是运行在 Runtime 基础设施之上的应用程序，Runtime 的稳定性直接决定业务程序的运行质量。

### 1.2 AI Agent，同样需要自己的 Runtime

Agent 的本质，其实也是一种程序。不同的是，它的"业务逻辑"并不是完全由开发者编写，而是部分交给了大语言模型。

这带来了一个新的问题。

传统程序的控制流（Control Flow）是确定性的。例如：

```
if success:
    next_step()
else:
    retry()
```

开发者能够精确知道程序接下来会做什么。而 Agent 不一样。其核心决策、执行路径、工具选择，均由大模型动态推理生成，比如：

```
下一步应该调用哪个工具？
是否继续搜索？
是否需要重新规划？
是否应该结束任务？
```


这些决策并不是硬编码，而是动态生成的。

于是，软件第一次出现了一种新的结构：控制流不再完全属于代码，而开始部分属于模型。这意味着，Runtime 必须承担更多责任。它不仅要管理程序，还要管理模型。

### 1.3 一个生产级 Agent，到底在运行什么

如果把一个 Agent 的生命周期拆开来看，你会发现，它其实一直在重复一个循环：

```
目标（Goal）
      │
      ▼
感知（Observe）
      │
      ▼
推理（Reason）
      │
      ▼
行动（Act）
      │
      ▼
验证（Verify）
      │
      ▼
恢复 / 学习（Recover / Learn）
      │
      └──────────────┐
                     ▼
                下一轮循环
```

其中，真正属于 LLM 的，只有"推理（Reason）"这一环。其他所有步骤，都依赖 Runtime。例如：

**1. Observe（感知）环节：** Runtime 负责收集上下文、检索长期记忆、获取工具状态、整理环境信息、构造最终 Prompt。LLM 并不知道这些信息来自哪里。

**2. Act（行动）环节：** 模型只能生成`调用 search()`，而真正执行`search()` 的是 Runtime，它负责参数校验、权限检查、工具路由、超时控制、重试、日志记录等。

**3. Verify（验证）环节：** 模型认为“任务完成了”，Runtime 仍然需要判断最终的完成结果，例如：

```
文件是否真的修改成功？
API 是否返回 200？
单元测试是否全部通过？
SQL 是否真正写入数据库？
```

没有验证环节，Agent 根本无法进入生产环境。

**4. Recover（恢复）环节：** 一旦出现执行失败、接口超时、格式异常等问题，Runtime 将自主决策是否重试、回滚、断点续跑、人工介入或任务终止。

### 1.4 Agent Runtime 核心定义

很多人第一次接触 Runtime 时，会误以为 Memory、Tool Calling、Prompt 就是 Runtime。事实上，这些都只是 Runtime 的一部分能力。Runtime 更大的职责是管理模型与真实世界之间的所有交互。例如：状态管理（State Management）、执行管理（Execution Management）、资源管理（Resource Management）、权限管理（Permission）、错误恢复（Recovery）、调度（Scheduling）、验证（Verification）、可观测性（Observability）。这些能力，本质上都属于 Runtime。

因此，我们可以把 Agent Runtime 定义为：围绕大语言模型构建的专属 Runtime 系统，全权管理模型与外部世界交互的全生命周期，涵盖状态管理、上下文调度、工具执行、异常恢复、安全治理、结果校验、全链路观测与资源治理，将无状态、概率性的大模型，转化为可稳定、可落地、可管控、可持续执行的生产级智能系统。

### 1.5 Runtime 与 Harness 的区别

很多文章把 Harness 理解为“除了模型之外的一切”，这种说法虽然容易传播，却并不准确。Harness 并不是 Runtime 的同义词。更准确地说：Harness 是 Runtime 的工程实现。

如果说 Runtime 回答的是“Agent 运行需要具备哪些核心能力”，是能力标准问题；那么 Harness Engineering 回答的是“如何通过工程手段落地这些能力”，是落地方法论问题。例如：

```
如何管理状态？
如何组织工具？
如何恢复失败？
如何验证输出？
如何保证安全？
如何观测整个系统？
```

在本文中，我们将采用如下关系：


```
Software Engineering
        │
        ▼
Agent Engineering
        │
        ▼
Runtime Engineering
        │
        ▼
Harness Engineering
```

其中，Software Engineering 关注如何设计、构建和维护一个完整的软件系统，通过编写确定性的业务逻辑来解决问题；Agent Engineering 关注如何构建能够自主理解目标、规划任务并调用工具完成工作的 Agent；Runtime Engineering 进一步关注 Agent 在真实环境中如何持续、可靠、安全地运行，包括状态管理、任务调度、错误恢复和执行治理；而 Harness Engineering 则是 Runtime Engineering 在工程实践中的具体实现方式，它通过状态管理、工具运行时、沙箱、验证、恢复、可观测性等基础设施，将 Runtime 的设计真正落地为一个生产级系统。

### 1.6 案例：直观理解 4 个概念

为了更直观地理解这 4 个概念，我们以一个 Research Agent 为例。

假设用户向 Agent 提出一个任务：“帮我调研 OpenAI、Anthropic 和 Google 在 Agent Runtime 方向的布局，并生成一份分析报告”。

虽然最终用户只看到了一次简单的对话，但在系统内部，不同层次的 Engineering 实际上承担着完全不同的职责。

#### （1）Software Engineering：构建整个产品

Software Engineering 关注的是整个软件系统，而不仅仅是 Agent 本身。例如，开发团队需要完成：

```
开发 Web 页面，让用户能够输入研究任务；
实现用户登录、权限管理和任务列表；
将生成的报告保存到数据库；
支持 PDF、Markdown 等格式导出；
提供 REST API，方便其他系统调用。
```

这些都属于传统的软件工程范畴，它们主要通过编写确定性的业务逻辑来完成。换句话说，Software Engineering 负责把整个产品搭建起来。

#### （2）Agent Engineering：让 Agent 学会完成研究任务

接下来，Agent Engineering 开始发挥作用。它关注的问题是：Agent 应该如何完成用户交给它的目标？

例如，当 Agent 收到任务后，它需要自主规划：

```
① 分析用户需求
        ↓
② 搜索 OpenAI 相关资料
        ↓
③ 搜索 Anthropic 相关资料
        ↓
④ 搜索 Google 相关资料
        ↓
⑤ 阅读并整理内容
        ↓
⑥ 对比三家公司技术路线
        ↓
⑦ 撰写最终研究报告
```

为了让 Agent 具备这种能力，开发者需要设计提示词（Prompt）、任务规划（Planning）、工具调用（Tool Calling）、记忆存储（Memory）、自省迭代（Reflection）等，这些都属于 Agent Engineering。

此时，开发者关注的是如何让 Agent 自主完成任务。

#### （3）Runtime Engineering：让 Agent 能够稳定运行

如果只是 Demo，到这里已经结束了。但真正投入生产环境后，新的问题才刚刚开始。例如：

```
搜索到一半，搜索
调研需要运行两个小时，中途服务器重启怎么办？
浏览器突然崩溃怎么办？
Token 超预算怎么办？
Agent 无限重复搜索同一篇文章怎么办？
最终报告是否真的引用了可靠来源？
```

这些问题都与 Prompt 无关，也不是 Agent 如何规划的问题，而是 Agent 如何持续、可靠、安全地运行。因此，Runtime Engineering 需要负责：

```
保存任务状态（State）
调度执行流程（Scheduler）
自动重试失败任务（Retry）
从中断位置恢复（Resume）
验证最终结果（Verifier）
控制权限和资源预算（Governance）
```

Runtime Engineering 更关心的是：如何保证 Agent 从开始执行到最终完成任务的整个生命周期都稳定可控。

#### （4）Harness Engineering：把 Runtime 真正落地

最后，Harness Engineering 登场。

Runtime Engineering 告诉我们：系统应该具备状态管理、恢复、验证等能力。

而 Harness Engineering 则回答另一个问题：这些能力具体如何实现？例如，在这个 Research Agent 中：

```
使用 Checkpoint 保存每一步调研进度；
使用 State Store 持久化任务状态；
使用 Browser Sandbox 隔离网页浏览环境；
使用 Retry Engine 自动重试失败的搜索请求；
使用 Citation Verifier 检查报告中的引用是否真实存在；
使用 Telemetry 记录每一次工具调用和模型推理过程，方便后续排查问题。
```

这些基础设施共同组成了 Agent 的 Harness。也就是说，Harness 并不会决定 Agent "研究什么"，而是负责让 Runtime 中设计的能力真正变成一个可以稳定运行的生产级系统。

对照上述，如果把整个过程浓缩成一句话，可以理解为：Software Engineering 负责把整个 AI 产品开发出来；Agent Engineering 负责让 Agent 学会完成研究任务；Runtime Engineering 负责让 Agent 在真实环境中持续、可靠地完成任务；而 Harness Engineering 则负责把这一切通过状态管理、调度、恢复、验证等基础设施真正落地。

从这个 Research Agent 的例子来看，四者之间并不是替代关系，而是层层递进、各司其职：Software Engineering 负责整个产品，Agent Engineering 负责智能，Runtime Engineering 负责运行，Harness Engineering 负责实现。

### 1.7 从"写代码"到"构建 Runtime"

软件工程师的工作正在发生改变。

过去，我们主要编写业务逻辑；今天，我们越来越多地构建运行环境。未来，一个优秀的 Agent 系统，其核心竞争力可能并不是拥有最强的模型，而是拥有最成熟的 Runtime。

模型决定智能，Runtime 决定智能能否真正落地，而 Harness Engineering 正是连接这两者的桥梁。

### 小结

在传统软件中，Runtime 往往隐藏在语言或操作系统之下，很少成为开发者关注的对象。

而在 AI Agent 时代，Runtime 从幕后走到了台前。因为模型本身并不知道如何管理状态、调用工具、恢复错误或验证结果，一个生产级 Agent 的绝大部分工程工作，都发生在 Runtime 中。

理解 Runtime，才能理解为什么 Prompt Engineering 和 Context Engineering 已经不足以支撑今天的 Agent；也才能理解，为什么越来越多的团队开始讨论 Harness Engineering。

## 第二章 ｜Prompt、Context、Harness 与 Runtime：AI 工程的控制边界演进

近年 AI 工程领域涌现出大量新概念：Prompt Engineering、Context Engineering、Harness Engineering、Runtime Engineering、Agent Engineering。这些概念常被混用、误解，普遍存在「新范式替代旧范式」的认知偏差。

事实上，五者并非迭代替代关系，而是工程控制边界持续向外扩张、能力逐层包含的层级体系。生产级 Agent 的落地，需要同时依赖所有范式，缺一不可。技术演进的核心本质，是开发者可管控的系统边界（Engineering Scope）持续扩大，工程精细化程度持续提升。

### 2.1 Prompt Engineering：控制单次模型推理

大模型落地初期，Prompt 是开发者唯一可干预、可调控的核心变量。Prompt Engineering 的核心命题是：如何精准引导模型的单次推理行为，输出符合预期的结果。

其核心能力覆盖：指令设计、角色设定（Role Prompt）、少样本学习（Few-shot Learning）、思维链引导（Chain of Thought）、输出格式约束、示例标准化。核心目标极致聚焦于优化单次模型推理（Single Inference）的输出质量。

此阶段的工程控制范围极小，整个控制范围可以表示为：

```
User
│
Prompt
│
LLM
│
Output
```

仅覆盖「用户输入-Prompt-模型输出」的单次链路，模型之外的运行、执行、异常问题完全无法管控，仅能满足简单聊天机器人场景，无法支撑复杂 Agent 任务。

### 2.2 Context Engineering：控制模型能看到什么

随着 Agent 开始调用工具、接入知识库、承载多轮长期对话，行业发现：真正决定模型推理质量的，并非 Prompt 话术本身，而是模型可感知、可调用的全部信息。由此，Context Engineering 正式成为核心工程范式。

相较于 Prompt Engineering，Context Engineering 大幅拓宽了管控边界，不再局限于 Prompt 话术，而是统筹管理模型的全部输入信息，涵盖：检索增强知识库（RAG）、长期记忆、对话历史、文件上下文、工具描述、环境状态、上下文压缩、Token 预算管控。

Prompt 只是 Context 的一部分，整个系统变成了：

```
Knowledge
Memory
History
Files
Tools
Environment
     │
     ▼
 Context Builder
     │
     ▼
     LLM
```

此时，工程师开始控制模型输入，而不仅仅是 Prompt。

Context Engineering 的核心命题从「模型应该怎么回答」升级为模型应该知道什么、感知什么，彻底解决模型信息缺失、视野局限导致的推理偏差问题，是 AI 工程的一次巨大进步。

### 2.3 Harness Engineering：控制模型的落地行动

然而，当 Agent 真正开始工作以后，仅仅管理输入已经不够了。模型不仅需要思考。它还需要调用工具、修改文件、执行 Shell、浏览网页、控制浏览器、操作数据库、协调多个 Agent 等。

于是，新的问题出现了。例如：

```
如果 Tool 超时怎么办？
如果 JSON 不合法怎么办？
如果 Browser 崩溃怎么办？
如果 API 调用失败怎么办？
如果 Agent 中途退出怎么办？
```

Context 无法回答这些问题。因为这些问题都发生在模型推理之后。于是，Harness Engineering 出现了。

Harness Engineering 的核心命题是：如何管控模型的落地行动，保障推理意图可靠转化为真实执行动作。其核心能力覆盖：工具 Runtime 调度、失败重试、断点恢复、沙箱隔离、任务调度、权限管控、结果校验、全链路观测。

整个系统开始变成：

```
Context
           │
           ▼
          LLM
           │
           ▼
────────────────────────
 Harness Runtime
────────────────────────

Tool Execution

Retry

Recovery

Verification

Sandbox

Permission

Telemetry
```

综上，Prompt 管控单次话术、Context 管控单次推理输入、Harness 管控单次落地行动。

### 2.4 Runtime Engineering：控制 Agent 全生命周期

如果继续扩大控制范围，你会发现 Harness 其实仍然只是 Runtime 的一部分。一个真正的 Agent Runtime，不仅需要 Harness，还需要任务规划、状态全局管理、工作流编排、事件总线、资源调度、效果评估、系统治理、持续优化等核心能力。

完整的 Runtime Engineering，覆盖 Agent 从启动、执行、迭代到终止的全生命周期。Runtime Engineering 回答的是“如何让 Agent 长期、稳定、自适应、可持续地运行”。因此，Runtime 的控制范围已经变成：

```
目标（Goal）

↓

感知（Observe）

↓

推理（Reason）

↓

执行（Act）

↓

验证（Verify）

↓

恢复（Recover）

↓

学习（Learn）

↓

下一轮循环（Next Loop）
```

在这一体系中，大模型只是其中的一个节点，Runtime 负责整个循环。

### 2.5 核心层级包含关系（核心架构图）

很多文章喜欢画：

```Plain
Prompt
   ↓
Context
   ↓
Harness
```

这张图很容易让人误解，Prompt 已经过时，Context 会消失，Harness 会替代它们。但事实完全不是这样：Prompt 在今天依然重要，Context 依然是 Runtime 的核心能力，Harness 依然依赖 Prompt 与 Context。

真正变化的是：工程师正在控制越来越大的系统。我们可以把四者关系画成：

```Plain
┌────────────────────────────┐
│ Runtime Engineering        │
│                            │
│ ┌──────────────────────┐   │
│ │ Harness Engineering  │   │
│ │                      │   │
│ │ ┌────────────────┐   │   │
│ │ │ Context Eng.   │   │   │
│ │ │                │   │   │
│ │ │ Prompt Eng.    │   │   │
│ │ └────────────────┘   │   │
│ └──────────────────────┘   │
└────────────────────────────┘
```

四大核心工程范式为逐层嵌套、全域覆盖的包含关系，无替代、无淘汰，或者画成：

```Plain
Runtime Engineering（全域生命周期管控）
└── Harness Engineering（行动执行管控）
　　└── Context Engineering（推理输入管控）
　　　　└── Prompt Engineering（单次推理管控）
```

这一体系清晰印证：旧范式从未失效，而是被新范式纳入统一工程体系，AI 工程的进化，本质是开发者管控的系统边界持续向外扩张。

### 2.6 一个新的软件工程范式正在形成

回顾软件工程发展史，其核心演进逻辑是抽象层级持续提升：传统软件工程抽象代码逻辑，而 AI 时代的工程体系，完成了多层全新抽象：

（1）Prompt 抽象模型推理能力；

（2）Context 抽象模型信息视野；

（3）Harness 抽象模型落地执行；

（4）Runtime 抽象智能体全生命周期。

行业正在完成一次核心转型：从「编程写逻辑（Programming the Logic）」，走向「编程写 Runtime （Programming the Runtime）」。未来开发者的核心工作，不再是逐行编写业务代码，而是设计 Agent 的运行环境、治理规则、校验机制与生命周期管理策略。

### 小结

AI 四大核心工程范式是层层嵌套的能力体系，而非迭代替代关系。Prompt 优化思考、Context 优化认知、Harness 优化执行、Runtime 统筹全局运行。工程边界的持续扩张，标志着 AI 软件工程正式从「单点模型优化」迈入「全域系统治理」的全新阶段。

## 第三章 ｜ 为什么 AI 软件需要新的 Runtime：软件首次开始管理「概率」

软件工程的百年发展史，本质是一部持续治理系统复杂性的进化史。从面向过程、面向对象，到分布式系统、云计算，每一次架构迭代，都是为了解决上一代工程体系无法承载的复杂问题。AI Agent 的诞生，将软件工程推向了全新的拐点。

过去，我们习惯把 Runtime 理解成一种底层基础设施。例如：Java 有 JVM；JavaScript 有 Node.js；Python 有 CPython，它们负责管理程序运行。传统软件很少会讨论 Runtime Engineering，因为大部分复杂性都来自代码：开发者编写代码，代码决定程序如何运行，Runtime 负责执行代码。因此，软件工程真正关注的对象是代码本身。

然而，当 Agent 成为软件系统的一部分之后，这个前提第一次发生了变化。软件真正需要管理的，不再只是代码，而是概率。

### 3.1 从确定性程序，到概率性程序

过去几十年，软件开发建立在一个基本假设之上：程序是确定性的（Deterministic）。给定同样的输入，程序一定得到同样的输出。例如：

```Plain
price = get_price()

if price > 100:
    buy()
else:
    wait()
```

这里不存在任何歧义，控制流（Control Flow）完全由开发者定义。程序每一步都会按照预先设计好的逻辑执行。因此，程序是否正确，主要取决于代码是否正确。这就是传统软件工程。

AI Agent 不一样。今天的软件里，第一次出现了这样一种控制流：

```Plain
Goal

↓

LLM

↓

Action ?
```

这里的下一步，并非由程序员决定，而是由模型决定。模型可能继续搜索、调用浏览器、打开文件、编写代码、重新规划、结束任务等。这些行为都不是确定性的。同样一个 Goal，今天运行一次和明天运行一次，结果可能完全不同。

这是传统软件从未遇到过的情况：软件第一次拥有了概率控制流（Probabilistic Control Flow）。

### 3.2 为什么 Prompt 无法管理概率

很多人第一次接触 Agent 时，会觉得：Prompt 写得更详细一点，不就可以了吗？

事实上，这正是 Prompt Engineering 的边界。Prompt 可以影响模型，但无法控制模型。它能够提高模型遵循指令的概率，却不能保证模型一定遵循。例如：

Prompt 可以要求“输出 JSON”，模型仍然可能输出 Markdown；

Prompt 可以要求“不要调用危险工具”，模型仍然可能生成危险调用；

Prompt 可以要求“先检查文件是否存在”，模型仍然可能直接覆盖文件。

原因非常简单。Prompt 本质上仍然只是一种软约束（Soft Constraint），仅能提升模型合规输出的概率，无法实现强制约束。即便话术足够精细，模型仍可能出现格式错乱、指令遗漏、越权调用、幻觉输出等问题，无法保障系统级可靠性。

### 3.3 为什么 Context 也无法管理概率

Context 的出现解决了另一个重要问题：相比起 Prompt 如何告诉模型，Context 聚焦于如何让模型知道。通过注入记忆（Memory）、知识库（Knowledge）、历史记录（History）、环境信息（Environment）等，模型拥有了更完整的信息，推理质量因此显著提升。

但是，Context 仍然没有改变一个事实：模型依然是概率性的。

以 Coding Agent 为例。Context 可以告诉模型整个项目结构、所有源代码、测试文件、历史修改记录、Git 状态，模型因此能够写出更好的代码。但是，它仍然可能删除不应该删除的文件、覆盖已有修改、忘记运行测试、无限循环、重复调用工具等。

Context 优化能够完善模型的信息输入，提升推理质量，但无法改变模型会犯错的本质。

简而言之，Prompt 与 Context 只能减少错误，无法杜绝错误，无法支撑生产级稳定运行。

### 3.4 Runtime 的核心价值：为概率系统建立确定性

真正的问题不是模型会犯错，而是软件必须面对犯错。这是 Runtime Engineering 与传统 AI Engineering 最大的区别。

传统 AI 更关注如何提高准确率，Runtime 更关注错误发生以后怎么办。软件工程有一句非常经典的话：系统不是为了成功而设计，而是为了失败而设计（Design for Failure）。

互联网系统如此，数据库如此，分布式系统如此，Agent 也一样。

我们真正需要回答的问题不是“模型会不会出错”，答案一定是“会”。

真正的问题应该是“模型出错以后，系统还能不能继续运行”。例如，JSON 不合法怎么办？API 超时怎么办？Browser 崩溃怎么办？工具返回空结果怎么办？Agent 推理错误怎么办？

这些问题 Prompt 无法回答，Context 也无法回答，Runtime 必须回答。

传统 AI 工程的核心目标是「提升模型准确率」，而 Runtime Engineering 的核心逻辑是面向失败设计（Design for Failure）：默认模型必然出错、工具必然异常、系统必然故障，核心目标不是规避错误，而是保障错误发生后，系统不崩盘、可恢复、可继续运行。

所有 Runtime 核心能力，最终都服务于同一目标：管控模型概率性，将非确定性的智能推理，封装为确定性的工程系统行为。重试机制应对随机失败、校验机制拦截幻觉输出、断点机制保障任务续跑、治理机制规避越权风险，全方位抵消模型的不确定性。

### 3.5 Runtime Engineering 本质上是一门新的软件工程

如果继续沿着软件工程的发展历史来看，会发现过去几十年，软件工程一直在管理不同类型的复杂性：单机时代，管理函数；面向对象时代，管理对象；互联网时代，管理网络；云计算时代，管理分布式资源。

今天，AI Agent 带来了新的复杂性：概率。

因此，Runtime Engineering 并不是 AI 独有的一个小技巧。它更像是软件工程面对概率计算系统之后，自然演化出的新分支。它关注的不再只是如何写代码，而是如何让一个概率系统稳定运行。这也是为什么越来越多的 Agent Framework，都开始把重点放在 Runtime，而不是 Prompt。

### 3.6 AI 软件的全新分层架构

传统软件通常可以抽象成三层：

```Plain
Application（应用层）
────────────
Business Logic（业务逻辑层）
────────────
Infrastructure（基础设施层）
```

而 AI 软件正在演化成新的结构：

```Plain
Application（应用层）
────────────
Agent（Agent 层）
────────────
Runtime（Runtime 层）
────────────
Foundation Model（基础模型层）
```

注意这里的差别：过去，业务逻辑直接建立在基础设施之上，今天，Agent 已经不能直接运行在模型之上。它们之间出现了一个新的 Runtime 层。这层 Runtime 负责吸收模型带来的所有不确定性。

因此，未来真正成熟的 AI 软件，很可能不会把 Runtime 看作一个工具库，而会把它看作新的基础设施层。正如操作系统曾经让软件摆脱了直接操作硬件，Runtime 也正在让 Agent 摆脱直接面对模型的不确定性。

### 小结

过去的软件工程，管理的是确定性的代码。

今天的软件工程，开始管理概率性的智能。

这意味着，我们已经不能再仅仅依靠 Prompt 或 Context 来构建生产级 Agent。真正需要解决的问题，不是让模型少犯一点错误，而是让整个系统能够在模型必然犯错的前提下，依然稳定、安全、持续地运行。这正是 Runtime Engineering 存在的意义。

而在 Runtime Engineering 的众多能力中，有一套实践正在逐渐形成行业共识，它负责组织 Runtime、约束模型行为、协调工具、恢复失败、管理状态，并将各种运行能力整合为一个完整的系统。这套实践，就是近年来越来越受到关注的 Harness Engineering​。


