# 一文读懂 AI Agent 的三种 Graph

(English Version)[https://github.com/JCat007/Jacky-s-Thinking/blob/main/en/Understanding%20Three%20Types%20of%20Graphs%20in%20AI%20Agents.md]

如果你最近关注 AI Agent，一定会发现一个现象：越来越多的人开始讨论 Graph（图结构）。

LangGraph、Google ADK、CrewAI Flow 等框架，都在强调 Graph；OpenAI、Anthropic 等公司也开始越来越多地讨论 Agent 的 Graph 架构和复杂协作（Complex Orchestration）。Peter Steinberger 在 X 上的一句反问（“Are we still talking loops or did we shift to graphs yet（我们还在讨论 Loop，还是已经转到 Graph 的话题了）?”），再次掀起 Graph 的讨论热潮。Graph 再次成为 Agent 领域的热议关键词。

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

## 一、Graph，到底是什么

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
## AI Agent Graph 三大类型对比
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

## 二、Task Graph：组织任务如何执行

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

### 2.1 Node：Graph 中真正执行的单元

节点（Node）是 Task Graph 的基础组成单元，是所有具体工作的落地载体，其核心定义是「可独立完成某项具体工作的执行单元」，形态并不固定。

常见的 Node 类型包括：单次大模型推理、工具调用指令、自定义 Python 函数、MCP 服务调用、独立子 Agent 执行任务等。比如在研究报告生成场景中，搜索资料、阅读网页、内容总结、文案撰写，每一项独立操作都是一个专属 Node，各司其职，共同支撑完整任务落地。

### 2.2 Edge：定义任务流转的核心逻辑

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

### 2.3 State：支撑 Graph 连续工作的共享上下文

仅有 Node 和 Edge，还不足以完成一个 Agent。因为每个节点都需要知道“前面的节点完成了什么”、“已经收集了哪些信息”、“下一步应该继续处理什么”。这些共享的信息，就是 State（状态）。

例如，一个 Research Agent 可以维护这样一个状态：

```Plain
## 研究状态

### 研究主题

### 搜索查询

### 参考资料

### 研究总结

### 报告初稿

### 质量评估
```

每一个节点都会读取 State，也会更新 State。例如：

```Plain
Search Node：新增参考资料。
Read Node：新增研究总结。
Write Node：新增初稿。
```

整个 Graph 就像很多人共同编辑同一份文档，每个人负责不同部分，但所有人的工作都基于同一份最新的数据。因此，State 可以理解为是，Graph 在执行过程中不断演化的共享上下文。

### 2.4 State Handoff：节点之间的高效协作机制

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

### 2.5 Task Graph 的核心价值

很多人误以为 Task Graph 只是简单的「提示词拆分步骤」。实际上，其真正的核心价值，是让 AI Agent 的任务执行从一次性模型调用，升级为标准化、可管控、可运维的工程化流程。

具体体现在四大核心能力：

第一，容错可恢复。单个任务节点执行失败时，无需整体重启，仅需重试对应节点即可，大幅降低故障成本；

第二，高效可并行。支持多节点同步执行，打破线性执行的效率瓶颈，大幅提升复杂任务处理速度；

第三，模块可复用。标准化的任务节点可灵活组合，适配不同业务场景，无需重复开发；

第四，全程可调试。所有节点的输入输出、状态迭代全程可记录、可回放，极大降低问题排查与优化难度。

当然，Task Graph 存在明确的边界局限：其所有节点均定义为「被动执行任务的单元」，无法自主规划、自主迭代。当执行单元升级为具备独立思考、自主决策能力的完整 Agent 时，Task Graph 便无法适配，此时就需要第二种 Graph 架构：Agent Graph。

## 三、Agent Graph：组织多个 Agent 如何协作

随着 Agent 能力不断增强，一个新的问题开始出现：一个 Agent，真的应该负责所有事情吗？

以一个 Research Agent 为例。如果让一个 Agent 独立完成整份行业研究报告，它通常需要同时承担很多不同的职责：理解用户需求、制定研究计划、搜索资料、阅读网页、判断资料可信度、总结信息、撰写报告、检查事实是否正确、修改表达方式、输出最终结果。

理论上，一个足够强大的大模型确实可以完成这一切。但随着任务越来越复杂，这种"万能 Agent"开始暴露出越来越多的问题。例如：

```Plain
Prompt 越来越长，角色越来越复杂； 
上下文不断增长，Token 消耗越来越高； 
一个模块出现错误，很容易影响整个任务； 
每个能力都绑定在同一个 Prompt 中，难以独立优化和复用。
```

这与传统软件的发展过程非常相似。早期的软件通常是一个庞大的单体应用（Monolith），所有功能都集中在同一个系统中；随着系统规模不断扩大，软件工程逐渐演化出模块化架构、微服务架构等设计方式，将不同职责拆分成独立模块。AI Agent 也正在经历类似的演进。越来越多的系统开始选择多个 Agent 协同工作，而不是一个 Agent 包揽所有职责。

### 3.1 一个复杂任务可以由多个 Agent 共同完成

以前面的 Research Agent 为例，一个典型的 Multi-Agent 系统可能会拆分为下面几个角色：

```Plain
Planner Agent
                       │
      ┌────────────────┼────────────────┐
      ▼                ▼                ▼
 Search Agent     Search Agent     Search Agent
      └────────────────┼────────────────┘
                       ▼
               Summarizer Agent
                       ▼
                 Writer Agent
                       ▼
                 Reviewer Agent
```

这里每一个节点都不再是一个简单的 Function，而是一个完整的 Agent。例如：

（1）Planner Agent 负责理解用户目标，并制定研究计划；

（2）Search Agent 根据计划搜索不同的信息源；

（3）Summarizer Agent 汇总多个搜索结果，提取关键信息；

（4）Writer Agent 根据整理后的内容撰写初稿；

（5）Reviewer Agent 对报告进行事实校验、逻辑检查和语言润色。

相比起只有一个万能 Agent，这种拆分方式有几个明显优势：

（1）每个 Agent 的职责更加单一。Planner 不需要关心写作，Writer 不需要关心搜索，每个 Agent 都可以围绕自己的职责进行优化。

（2）不同 Agent 可以使用不同的模型。例如：Planner 使用推理能力更强的大模型；Search 使用速度更快的小模型；Reviewer 使用专门的事实校验模型。 整个系统可以根据任务特点灵活组合，而不是所有工作都依赖同一种模型。

（3）多个 Agent 可以并行工作。例如，同时启动多个 Search Agent 分别检索论文、新闻、博客和技术文档，再统一汇总结果，比单个 Agent 依次搜索效率更高。

因此，Agent Graph 的出现，并不是为了让系统变得更复杂，而是为了让复杂任务能够被合理拆解和协同完成。

### 3.2 Agent Graph 的核心是协作关系，而非 Agent 数量

很多人第一次接触 Multi-Agent 时，关注点往往放在“我应该拆多少个 Agent”。实际上，更重要的问题是：这些 Agent 应该如何协作？

这正是 Graph 存在的意义。

Agent Graph 真正组织的，不是 Agent 本身，而是 Agent 之间的关系。例如，一个 Planner Agent 可以把任务交给多个 Search Agent；多个 Search Agent 可以共同把结果交给 Summarizer Agent；Reviewer Agent 发现问题之后，还可以把任务重新交回 Writer Agent 修改。

这意味着，Graph 中的边（Edge）描述的不再是简单的执行顺序，而是协作关系。这些关系通常包括：

（1）Delegation（任务委派）​：一个 Agent 将子任务交给另一个 Agent；

​（2）Routing（任务路由）：根据不同情况，把任务发送给不同 Agent；

​（3）Aggregation（结果汇聚）：多个 Agent 的结果汇总后继续处理；

​（4）Handoff（任务交接）​：一个 Agent 完成自己的职责后，将任务交给下一位 Agent。

Graph 描述的，正是这些关系如何共同组成一个协作网络。

### 3.3 State Handoff 与 Agent Handoff 的核心区别

在上一章中，我们介绍了 State Handoff（状态传递）。当 Graph 的节点是 Task 时，节点之间传递的是共享状态（State）。

而在 Agent Graph 中，除了共享状态之外，还经常会出现另一个概念：Agent Handoff。两者容易混淆，但含义并不相同。

State Handoff 强调的是数据的传递。例如，Search Agent 将搜索结果写入共享 State，Writer Agent 再读取这些内容继续写作。整个过程中，变化的是数据。而 Agent Handoff 强调的是控制权的转移。例如，Planner Agent 判断下一步应该进入搜索阶段，于是把当前任务交给 Search Agent；搜索完成之后，再把控制权交给 Writer Agent。整个过程中，变化的是当前负责执行任务的 Agent。

简单来说，State Handoff 回答的是“数据交给谁”， Agent Handoff 回答的是“接下来谁负责”。

在实际系统中，两者通常同时发生：控制权转移到新的 Agent，同时共享状态也被传递和更新。

### 3.4 Agent Graph 并非越复杂越好

Multi-Agent 并不是万能方案。事实上，很多 Agent 项目在早期都会犯一个共同的错误：把所有能力都拆成不同的 Agent。

结果往往适得其反。Agent 数量越多，往往导致通信成本越高、Token 消耗越大、状态同步越复杂、调试难度也会显著增加。 因此，一个经验丰富的 Agent 系统设计者，首先思考的并不是“能不能拆更多 Agent”，而是“哪些职责真的需要独立 Agent”。

通常来说，只有当不同职责具有明显不同的目标、能力或工具时，才值得拆分为独立 Agent。否则，一个设计良好的单 Agent 往往更加简单、高效，也更容易维护。

Agent Graph 的目标不是追求更多 Agent，而是让协作关系与问题复杂度相匹配。

### 3.5 Agent Graph 的核心边界

Task Graph 解决的是“任务如何执行”，Agent Graph 解决的是“多个 Agent 如何协作”。然而，无论采用单 Agent 还是 Multi-Agent，它们都默认了一个前提：只要不断执行任务，系统就会越来越好。

但真实世界并非如此：一个 Agent 可能会不断重复错误的策略；评价指标可能已经偏离真实目标；多个 Agent 之间甚至可能互相放大彼此的偏差。

换句话说，Task Graph 和 Agent Graph 负责让系统"跑起来"，却没有回答另一个更重要的问题：系统如何持续变得更好？

此时，真正需要组织的，已经不再是 Task，也不再是 Agent，而是一个又一个负责优化系统的 Feedback Loop（反馈循环）。这也是近年来越来越多人开始讨论 “From Loops to Graphs” 的原因。

## 四、Feedback Graph：组织系统如何持续优化

如果说 Task Graph 回答的是“任务应该如何执行”，Agent Graph 回答的是“多个 Agent 应该如何协作”，那么，Feedback Graph 回答的则是另一个更加底层的问题：系统如何持续变得更好？

Feedback Graph 关注的，不是如何完成一次任务，而是如何让整个 Agent 系统在长期运行中不断学习、不断修正、不断演进。

### 4.1 Loop：Agent 持续优化的最小单位

任何一个能够不断改进自己的系统，都离不开一个最基本的结构：

```Plain
目标（Goal）
      │
      ▼
观察（Observe）
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
恢复（Recover）
      │
      ▼
学习（Learn）
      │
      └──────────────┐
                     ▼
                下一次循环
```

这就是一个 Feedback Loop。它描述了 Agent 从获取目标，到执行任务，再根据结果不断调整自己的完整过程。例如，一个 Research Agent 可以形成这样一个循环：

（1）接收研究任务；

（2）搜索相关资料；

（3）撰写研究报告；

（4）使用 Reviewer 检查事实；

（5）根据 Reviewer 的反馈修改内容；

（6）更新下一次搜索和写作策略。

整个过程不断重复。每一次循环，都让 Agent 比上一次更接近目标。

因此，Loop 可以理解为：Agent 持续优化自己的最小反馈单元。事实上，很多 Agent 今天都已经具备了 Loop。像 Reflection（反思）、Self-Refine（自迭代优化）、ReAct（推理行动框架）、Critic-Reviewer（评审器架构）、Plan → Execute → Reflect（规划 - 执行 - 反思）等机制，本质上都是 Feedback Loop 的不同实现。

### 4.2 为什么一个 Loop 不够

如果只有一个评价指标，一个 Loop 往往可以取得不错的效果。但随着系统越来越复杂，一个问题开始出现：Loop 优化的，真的是我们想要的吗？

假设我们开发了一个 Research Agent，希望不断提升它生成行业分析报告的速度。于是，我们建立了一个 Feedback Loop，只优化一个指标：生成速度（Report Generation Time）。Agent 每完成一次报告，就根据耗时调整下一次策略，希望报告越来越快完成。

几个月后，这个指标持续改善。Agent 从原来的 20 分钟生成一份报告，缩短到了 3 分钟。看起来，一切都在变好。但真正查看报告时却发现：Agent 开始越来越少地阅读原始资料，只引用已有记忆中的内容；搜索步骤越来越少，甚至直接跳过部分信息验证；生成的报告虽然更快，却充满了过时信息和事实错误。

这里 Loop 并没有出错，它只是忠实地优化了那个唯一的目标。这正是著名的 Goodhart's Law（古德哈特定律）：当一个指标成为优化目标时，它就不再是一个好的指标。

Loop 的问题，不是不会优化。恰恰相反。它太擅长优化了。

### 4.3 复杂系统，必然存在多组并行 Loop

现实中的 AI Agent，远比一个 Loop 更复杂。

例如，一个生产级 Research Agent，至少可能存在下面这些 Feedback Loop：

```Plain
规划 Loop
        │
执行 Loop
        │
评估 Loop
        │
恢复 Loop
        │
学习 Loop
```

每一个 Loop，都负责优化不同的能力。例如：

**​（1）规划 Loop：​**不断优化任务拆解策略；

**​（2）执行 Loop：​**不断优化工具调用成功率；

**​（3）评估 Loop：​**不断优化输出质量；

**​（4）恢复 Loop：​**不断降低失败恢复时间；

**​（5）学习 Loop：​**不断更新 Memory 和长期策略。

这些 Loop 并非彼此独立，它们会互相影响。例如，如果规划策略发生变化，执行方式也需要调整；执行结果发生变化，评估标准也可能需要改变；评估发现问题，学习策略又需要更新。

于是，Loop 开始彼此连接，Graph 就出现了。

### 4.4 Feedback Graph：多 Feedback Loop 的组网体系

包括 Peter Steinberger 在内的许多开发者提出的 Graph，指的是 Network of Feedback Loops（反馈循环网络），​可以表示为：

```Plain
规划 loop
              │
              ▼
          执行loop
              │
              ▼
          评估loop
         │           │
         ▼           ▼
     治理 loop      恢复 loop
         │           │
         └─────┬─────┘
               ▼
          学习 loop
               │
               └─────────────┐
                             ▼
                          下一轮规划
```

Graph 在这里的意义，不再是任务如何流转，而是多个 Feedback Loop 如何共同推动整个系统持续演进。因此，Feedback Graph 关注的是：

（1）哪些 Loop 应该彼此监督？

（2）哪些 Loop 可以调整其它 Loop？

（3）哪些指标不能被直接优化？

（4）哪些策略需要长期学习？

Graph 描述的，不再是执行流程，而是整个优化系统的结构。

### 4.5 Feedback Graph 解决了什么

相较于单一 Feedack Loop，Feedback Graph 从根源上解决了复杂 Agent 系统的迭代短板，核心价值体现在四个维度：

第一，规避单一指标异化。通过多维度 Loop 相互制衡，同时优化效率、质量、体验、稳定性等多重目标，避免单一指标过度优化导致的系统偏差；

第二，实现目标动态迭代。单一 Loop 只能优化既定目标，而 Feedback Graph 可以通过 Governance Loop、Review Loop，动态调整系统评价标准、核心目标与优化方向，适配业务变化；例如：季度 Review Loop 可以修改 Planner 的策略，年度 Governance Loop 又可以重新定义整个系统的评价标准。因此，Graph 不只是优化行为，也在不断优化"什么才叫做好"。

第三，让多个反馈循环彼此制衡。规划不能无限增加复杂度；恢复不能无限增加重试次数；学习不能无限更新 Memory。不同 Loop 之间，需要共同维持系统平衡。

第四，让系统真正具备长期演进能力。一个生产级 Agent，并不是执行一次任务。它可能连续运行几个月、几年。Graph 描述的是整个系统如何随着时间不断成长，而不是一次任务如何完成。

### 4.6 3 种 Graph 小结

Task Graph、Agent Graph 和 Feedback Graph，看起来都叫 Graph，但它们组织的对象完全不同：

Task Graph 的节点是 Task，旨在组织任务如何执行；Agent Graph 的节点是 Agent，旨在组织多个 Agent 如何协作；而 Feedback Graph 的节点，则是一个又一个 Feedback Loop，关注整个 Agent 系统如何持续优化，而不仅仅是如何完成一次任务。

这也是近年来 AI Agent 架构正在发生的重要变化：过去，我们关注的是如何让 Agent 完成任务（Task Completion）；今天，我们开始关注如何让 Agent 持续改进（Continuous Improvement）。

Graph 的真正意义，也因此发生了变化。

## 五、Graph、Loop、Runtime 与 Harness，到底是什么关系

很多读者发现，Loop，Graph 这些概念常常与另一些词一起出现，比如 Harness、Runtime。有些文章甚至会提出 Loop Engineering、Graph Engineering 等说法。

那么，它们之间到底是什么关系？

我认为，理解这些概念，最重要的是不要把它们放在同一个层次。它们关注的问题，其实完全不同。

### 5.1 Loop：Agent 思考的基本循环

Loop 是最容易理解的概念。

对于一个 Agent 来说，无论它完成的是写代码、搜索资料还是控制机器人，本质上都会不断重复下面这个过程：

```Plain
Goal（目标）

↓

Observe（感知）

↓

Reason（推理）

↓

Act（行动）

↓

Verify（验证）

↓

Recover（恢复）

↓

Learn（学习）

↓

Next Loop（下一个循环）
```

也就是说，Loop 描述的是 Agent 如何完成一次完整的认知循环。它关注的是 Agent 如何思考、如何行动、 如何根据结果调整下一次行动。

Loop 本质上描述的是一种行为模式（Behavior Pattern）。

### 5.2 Graph：组织多个 Loop、Task 或 Agent

如果说 Loop 是一个最小循环，那么 Graph，就是组织多个循环之间关系的方法。只不过，根据组织对象不同，Graph 又可以分成三种：

第一种，是 Task Graph​。Graph 的节点是 Task，描述哪些任务先执行、哪些任务并行执行、哪些任务执行完成以后再继续下一步；

第二种，是 ​Agent Graph​。Graph 的节点变成了 Agent，描述多个 Agent 如何分工、如何交接、如何协作完成复杂任务；

第三种，是 ​Feedback Graph​。Graph 的节点变成了一个个 Feedback Loop，描述整个系统如何持续优化自己。

因此，Loop 与 Graph 的关系其实非常简单。Loop 是一个循环。Graph 是多个 Loop（或者多个 Task、多个 Agent）之间的组织关系。Graph 并不是比 Loop 更高级，它只是组织对象发生了变化。

### 5.3 Runtime：让整个 Graph 持续运行

那么，Graph 有了以后，是不是 Agent 就可以运行了？答案是否定的。

假设我们已经设计好了一个 Research Agent。它拥有 Planner Agent、Search Agent、Writer Agent、 Reviewer Agent； 多个 Agent 又组成了完整的 Task Graph。与此同时，

系统还有一套 Feedback Graph，不断优化整个 Agent。

Graph 全部设计完成。但是，一个新的问题出现了。如果：

```Plain
某个 Agent 崩溃了怎么办？
工具调用超时怎么办？
状态丢失怎么办？
任务执行两天以后断电怎么办？
Memory 如何更新？
Checkpoint 如何恢复？
多个 Agent 如何调度？
权限如何控制？
```

这些问题，都不是 Graph 能解决的。Graph 负责回答“应该如何组织”，Runtime 回答“应该如何运行”。

因此，我们可以把 Runtime 理解为整个 Agent 的运行平台（Execution Platform），Graph 运行在 Runtime 上，Loop 运行在 Runtime 上，Memory、Scheduler、State Store、Tool Runtime、Sandbox、Recovery 等能力，同样运行在 Runtime 上。

如果把 Agent 比作一个程序，那么 Runtime 就像操作系统。程序决定做什么，操作系统决定程序如何真正运行。

### 5.4 Harness：Runtime 的工程化实现

理解了 Runtime，Harness 就很好理解了。

Runtime 更像是一种架构思想，它回答的是：一个 Agent Runtime 应该具备哪些能力？例如状态存储（State Store）、安全治理（Governance）、校验器（Verifier）等 。

而 Harness 是对应这套标准的工程落地载体，将 Runtime 提出的各项能力具象实现，例如：

（1）针对 Runtime 的状态恢复需求，Harness 提供检查点、快照与恢复（Checkpoint、Snapshot、Resume）能力；

（2）针对 Runtime 的安全治理需求，Harness 落地权限管控、策略规则、沙箱环境与人机审批（Policy、Sandbox、Human Approval）机制；

（3）针对 Runtime 可观测性需求，Harness 完整实现链路追踪、日志、指标与流程回放（Trace、Log、Metrics、Replay）工具链。

Harness 把 Runtime 从一套设计思想，真正变成一个能够部署到生产环境中的工程系统。

### 结语

综上所述，Loop、Graph、Runtime 和 Harness，并不是彼此替代，而是在不同层次共同构成现代 AI Agent 的工程体系。它们之间的关系可以用这个图来表示：

```
                Goal
                  │
                  ▼
          Agent Intelligence
     （Reason / Planning）
                  │
                  ▼
────────────────────────────────
          Runtime
────────────────────────────────

     Graph（执行拓扑）

     Loop（执行循环）

     Scheduler（调度）

     State（状态）

     Memory（记忆）

     Recovery（恢复）

     Governance（治理）

     Tool Runtime（工具）

────────────────────────────────
                  │
                  ▼
               Harness
        （工程实现）
```

最后，针对社区中热议的“从 Loop 走向 Graph”。我认为，这里面反映的真正变化，是 Agent 设计哲学正在从设计单个 Agent，走向设计整个 Agent 系统。
