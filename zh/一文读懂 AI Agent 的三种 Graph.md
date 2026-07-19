如果你最近关注 AI Agent，一定会发现一个现象：越来越多的人开始讨论 Graph（图结构）。

LangGraph、Google ADK、CrewAI Flow 等框架，都在强调 Graph；OpenAI、Anthropic 等公司也开始越来越多地讨论 Agent 的 Graph 架构和复杂协作（Complex Orchestration）。Peter Steinberger 在 X 上的一句反问，再次掀起 Graph 的讨论热潮。Graph 再次成为 Agent 领域的热议关键词。

（Peter 的反问截图）

但问题也随之而来：很多人第一次接触 Graph，是因为 LangGraph，因此很容易认为 “Graph = Workflow = DAG”。还有一些文章则把 Graph 描述为多个 Agent 之间的协作网络（Multi-Agent Graph），又或者说，Graph 是由多个 Feedback Loop 构成的网络（Network of Feedback Loops）。于是，一个简单的词，开始代表三种完全不同的含义。这也是目前 AI Agent 领域最容易产生概念混淆的地方。

事实上，这三种 Graph 并不是竞争关系，也不是彼此替代，它们解决的是三个不同层次的问题：

（1）Task Graph：组织任务应该如何执行；

（2）Agent Graph：组织多个 Agent 应该如何协作；

（3）Feedback Graph：组织系统应该如何持续优化。

理解了这三种 Graph，你就能理解为什么 LangGraph、Anthropic、OpenAI、Google 等公司都在讨论 Graph，却又似乎在说不同的东西。

在正式展开全文内容前，先建立一套共识：Loop 是 Agent 的基础认知循环；Graph 是组织 Loop、Task、Agent 的核心架构。虽然它们在很多地方被成为一门工程（如 xx Engineering），但我认为它们更像 Runtime 内部的重要设计模式。Runtime 是支撑 Loop/Graph 持续运行的底层平台，而 Harness 是 Runtime 的工程化落地实现。

本文不聚焦单一框架的实操教程，也不局限于 LangGraph 的基础使用，核心目标是搭建一套统一、完整、体系化的 AI Agent Graph 认知框架。本文希望帮助读者厘清三个核心问题：

1. 为什么 AI Agent 的架构，正在从单一 Loop 迭代升级为 Graph 架构？
2. 三种核心 Graph 分别解决哪些底层问题，彼此之间如何协同赋能？
3. Graph、Loop、Runtime、Harness 四大核心概念的层级关系与核心定位是什么？

接下来，我们从最基础的问题出发，逐层拆解 AI Agent 的 Graph 体系。

# 一、Graph，到底是什么

提到 AI Agent 中的 Graph，很多人的第一联想都是 LangGraph。

过去一年，LangGraph 的快速普及，让很多开发者认为 Graph 就是 AI Agent 的工作流（Workflow），等同于 DAG（有向无环图），但这只是 Graph 在 AI 领域的一个应用场景。Graph 并非 AI Agent 的专属概念，其本源是计算机科学中的「Graph（图）」数据结构。图由节点（Node）和边（Edge）两大基础单元组成，核心作用是精准描述多个对象之间的关联关系。

Graph 最大的价值，不是节点本身，而是如何组织这些节点之间的关系。例如，在社交网络中，Graph 用来描述人与人之间的关系；在交通导航中，Graph 用来描述城市之间的连接关系；在知识图谱中，Graph 用来描述不同知识之间的关联关系。

AI Agent 也是如此。

随着 AI Agent 从简单的对话机器人，迭代为具备自主规划、工具调用、长期运行、多 Agent 协同能力的复杂系统，单一的线性逻辑、固定循环模式已经无法承载复杂的业务需求。而 Graph 结构化、可扩展、可协同的特性，成为适配复杂 Agent 系统的最优解。

这里有一个极易被忽略的核心细节：不同层级的问题，需要 Graph 组织的核心对象完全不同。

有时候，我们需要组织的是任务（Task）。例如，一个 Research Agent 可以按照「搜索 → 阅读 → 总结 → 写作」的流程完成一项研究任务。这时，每一个节点代表的是一个任务步骤，Graph 描述的是整个任务的执行流程。

有时候，我们需要组织的是多个 Agent。例如，一个大型 Research Agent 可以拆分为 Planner Agent、Search Agent、Writing Agent、Reviewer Agent 等多个 Agent，每个 Agent 负责不同职责，再共同完成复杂任务。这时，Graph 描述的是多个 Agent 之间的协作关系。

当 Agent 开始长期运行之后，我们真正需要优化的，已经不再是某一个任务，也不是某一个 Agent，而是整个系统如何持续学习、持续优化、持续纠错。例如：

（1）如何验证 Agent 的结果是否正确？

（2）如何发现评价指标已经失效？

（3）如何不断调整 Agent 的策略？

（4）如何避免系统陷入局部最优？

这时，被组织的对象变成了一个又一个 Feedback Loop（反馈循环）。Graph 描述的，是多个 Feedback Loop 之间如何相互监督、相互约束、共同推动整个系统持续演进。

因此，虽然大家都在讨论 Graph，但实际上讨论的是三个不同层次的问题。为了便于理解，我们可以将 AI Agent 中的 Graph 分为三种类型：

```Plain
AI Agent Graph

        ┌────────────┼────────────┐
        │            │            │
        ▼            ▼            ▼
   Task Graph   Agent Graph   Feedback Graph
```

它们分别回答三个完全不同的问题：

```markdown
# AI Agent Graph 三大类型对比
| Graph 类型 | 核心节点 Node | 核心解决问题 |
| :--- | :--- | :--- |
| Task Graph | 任务、函数、工作流单元 | 规范复杂任务的拆分、执行与流转流程 |
| Agent Graph | 独立 Agent | 统筹多 Agent 的分工、协作与任务交接 |
| Feedback Graph | 反馈循环（Feedback Loop） | 实现整个 Agent 系统的持续纠错、优化与演进 |
```

需要强调的是，这三种 Graph 不是替代关系，而是互补关系。一个真正的生产级 AI Agent，往往会同时存在三种 Graph。

例如，一个 Research Agent 在执行一份行业分析报告时：

（1）Task Graph 决定「搜索 → 阅读 → 总结 → 写作」的执行流程；

（2）Agent Graph 决定 Planner Agent、Search Agent、Writer Agent、Reviewer Agent 如何分工协作；

（3）Feedback Graph 则负责不断评估报告质量、调整搜索策略、优化写作结果，并持续改进整个系统。

也就是说，它们关注的是同一个系统的三个不同层面：执行、协作与演进。这也是为什么 OpenAI、Anthropic、Google、LangGraph 等公司都在讨论 Graph，却又似乎在谈论不同的东西，因为它们关注的是不同层次的问题。

# 二、Task Graph：组织任务如何执行

对于绝大多数 AI Agent 来说，Graph 最早出现的地方，就是任务执行（Task Execution）。

假设我们希望 AI 帮我们完成一份《2026 年 AI Agent 行业研究报告》。如果把整个任务交给一次大模型调用，模型很可能因为上下文不足、信息不完整或者推理链过长，最终生成一份质量很一般的报告。于是，我们很自然会把一个复杂任务拆解成多个更小的任务，例如：

```Plain
确定研究主题
        │
        ▼
搜索相关资料
        │
        ▼
阅读与筛选内容
        │
        ▼
整理关键信息
        │
        ▼
撰写初稿
        │
        ▼
事实校验
        │
        ▼
生成最终报告
```

这就是最简单的 Task Graph。每一个节点（Node）代表一个任务，每一条边（Edge）代表任务之间的执行关系。Task Graph 的核心思想并不复杂：不要一次完成整个任务，而是把复杂任务拆成多个可管理、可组合、可复用的任务节点。

事实上，这种思想并不是 AI Agent 发明的。传统软件中的工作流（Workflow）、CI/CD Pipeline、数据处理流水线（Data Pipeline），本质上都是一种 Task Graph。AI Agent 只是把 Graph 的节点，从传统的软件模块，变成了可以由 LLM 驱动的任务节点。

## 2.1 Node：Graph 中真正执行的单元

节点（Node）是 Task Graph 的基础组成单元，是所有具体工作的落地载体，其核心定义是「可独立完成某项具体工作的执行单元」，形态并不固定。

常见的 Node 类型包括：单次大模型推理、工具调用指令、自定义 Python 函数、MCP 服务调用、独立子 Agent 执行任务等。比如在研究报告生成场景中，搜索资料、阅读网页、内容总结、文案撰写，每一项独立操作都是一个专属 Node，各司其职，共同支撑完整任务落地。

## 2.2 Edge：定义任务流转的核心逻辑

如果说 Node 决定了「系统要做什么」，那么 Edge（边）就决定了「系统下一步要做什么」，是管控任务流转、分支、并行、跳转的核心。

最简单的情况是顺序执行：

```Plain
Search
    │
    ▼
Read
    │
    ▼
Write
```

但真实的 Agent 往往不会这么简单。例如，一个 Research Agent 在搜索完成之后，可能需要判断搜索结果是否足够。如果资料不足，就继续搜索；如果资料足够，再进入阅读阶段。于是，Graph 开始出现分支：

```Plain
Search
             │
      ┌──────┴──────┐
      ▼             ▼
继续搜索         阅读资料
```

同样，一个任务也可能拆分成多个并行节点：

```Plain
Planner
        ┌─────┼─────┐
        ▼     ▼     ▼
 Search A Search B Search C
        └─────┼─────┘
              ▼
          Merge Result
```

因此，Edge 不只是连接节点，更决定了整个任务的执行逻辑。

## 2.3 State：支撑 Graph 连续工作的共享上下文

仅有 Node 和 Edge，还不足以完成一个 Agent。因为每个节点都需要知道“前面的节点完成了什么”、“已经收集了哪些信息”、“下一步应该继续处理什么”。这些共享的信息，就是 State（状态）。

例如，一个 Research Agent 可以维护这样一个状态：

```Plain
# 研究状态

## 研究主题

## 搜索查询

## 参考资料

## 研究总结

## 报告初稿

## 质量评估
```

每一个节点都会读取 State，也会更新 State。例如：

```Plain
Search Node：新增参考资料。
Read Node：新增研究总结。
Write Node：新增初稿。
```

整个 Graph 就像很多人共同编辑同一份文档，每个人负责不同部分，但所有人的工作都基于同一份最新的数据。因此，State 可以理解为是，Graph 在执行过程中不断演化的共享上下文。

## 2.4 State Handoff：节点之间的高效协作机制

很多文章都会提到一个概念：State Handoff（状态传递）。其实，它并不复杂。它表示的是一个节点完成工作后，把最新的 State 交给下一个节点继续执行。例如：

```Plain
Planner
    │
    ▼
Search
    │
    ▼
Read
    │
    ▼
Write
```

Planner 首先生成搜索计划，并把 Query（搜索关键词）写入 State。Search 根据 Query 搜索网页，再把 Documents（参考资料）写入 State。Read 根据 Documents 阅读网页，继续补充 Summary（研究总结）。最后，Write 根据 Summary 生成最终报告。

整个过程中，真正流动的并不是 Node，而是不断更新的 State。这也是为什么很多 Graph Framework 都强调：Node 是无状态（Stateless）的，而 State 是有状态（Stateful）的。

Node 负责完成工作。State 负责保存工作成果。

## 2.5 Task Graph 的核心价值

很多人误以为 Task Graph 只是简单的「提示词拆分步骤」。实际上，其真正的核心价值，是让 AI Agent 的任务执行从一次性模型调用，升级为标准化、可管控、可运维的工程化流程。

具体体现在四大核心能力：

第一，容错可恢复。单个任务节点执行失败时，无需整体重启，仅需重试对应节点即可，大幅降低故障成本；

第二，高效可并行。支持多节点同步执行，打破线性执行的效率瓶颈，大幅提升复杂任务处理速度；

第三，模块可复用。标准化的任务节点可灵活组合，适配不同业务场景，无需重复开发；

第四，全程可调试。所有节点的输入输出、状态迭代全程可记录、可回放，极大降低问题排查与优化难度。

当然，Task Graph 存在明确的边界局限：其所有节点均定义为「被动执行任务的单元」，无法自主规划、自主迭代。当执行单元升级为具备独立思考、自主决策能力的完整 Agent 时，Task Graph 便无法适配，此时就需要第二种 Graph 架构：Agent Graph。
