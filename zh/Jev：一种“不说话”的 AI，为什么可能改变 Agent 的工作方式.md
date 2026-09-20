# Jev：一种“不说话”的 AI，为什么可能改变 Agent 的工作方式

我们已经习惯了这样使用 AI：“帮我写一封邮件”、“总结一下这篇文章”、“分析一下这个问题”、“帮我制定一个计划”。所以，当一个新的 AI 模型出现时，我们通常会问：它比 GPT、Claude 更聪明吗？

但 TypeSafe 最近发布的 Jev，走了一条完全不同的路线：它甚至不会生成文字。你给它一段信息，再给它几个明确的问题，它返回的不是一段自然语言，而是一个结构化的判断加上概率和置信度。

例如，输入“客户说如果今天还不能解决问题，就要取消订单”，问题是“这是紧急工单吗？”，Jev 返回 { urgent: true, probability: 0.97, confidence: 0.94 }。然后你的程序就可以直接写 if result.urgent and result.confidence > 0.9: escalate_to_human()。

这就是 Jev 最核心的思想：LLM 负责生成内容，而 Jev 负责做判断。TypeSafe 将这种模型称为 System One Model，Jev 是目前公开的第一个模型。TypeSafe 的创始人 Diogo Almeida 此前参与过 OpenAI 早期的相关研究，旨在让语言模型更好地遵循指令以及与人对话。

## 一个简单的问题：软件真的需要 AI“说话”吗

想象你正在做一个客服 Agent。用户发来：“我的银行卡已经扣了两次钱，你们到底什么时候退款？如果今天不给我处理，我就投诉。”

传统做法可能是：用户消息传给 LLM，LLM 返回“这是一条比较严重的投诉，建议转交人工客服……”，然后你的程序再解析这句话，决定是不是升级工单。

问题来了。其实整个过程中，LLM 根本不需要写那段话。软件真正想知道的只有几个问题：是不是紧急？是不是退款问题？是不是投诉？应该交给哪个部门？是否需要人工介入？

也就是说，软件需要的不是“请告诉我你怎么看”，而是“请做一个判断”。这正是 Jev 所针对的场景。

TypeSafe 给出的核心接口可以简单理解成：State → Question → Typed Decision，也就是状态经过问题得到结构化判断，而不是传统 LLM 的 Prompt → Token → 文字。

## Jev 到底是什么

如果用一句话解释：Jev 是一个专门给软件做“语义判断”的 AI 模型。你可以把它理解成一个拥有语言理解能力的 if / else。

普通代码非常擅长确定性的事情，比如 if temperature > 38: fever = True，因为“38”是一个明确的数字。但现实世界有大量事情不是这么简单：这条消息是不是在抱怨？这个客户是不是快流失了？这个新闻是不是和我们的公司有关？这个回答是不是回答了用户的问题？这个商品是不是适合这个用户？这个网页是不是包含我要的信息？这个 Agent 当前是不是应该继续执行？

这些问题很难写成传统的规则。你可以写几百条 if "投诉" in text、if "退款" in text、if "马上" in text，但很快你就会发现：自然语言世界太复杂了，规则很难覆盖。于是过去我们会直接把这些问题丢给一个大语言模型。

而 Jev 想做的是：把“理解自然语言之后做判断”本身，变成一个独立的软件原语。

## Jev 和普通 LLM 最大的区别

最容易理解的方式，是比较两者的输出。

普通 LLM 你问“这个客户应该交给哪个团队？”，它可能返回：“从客户的问题来看，他主要是在反馈支付失败问题，同时存在一定的投诉倾向。因此建议将该工单优先转交给支付团队处理。”这对人来说很好读，但对程序来说就比较麻烦。程序真正想要的是 { "department": "payment" }。

于是工程师就开始做 Prompt Engineering + JSON Mode + Structured Output + Schema Validation + Retry + Error Handling，最后发现：我只是想让 AI 告诉我一个值。

Jev 的思路则从一开始就把问题定义成：state 是用户消息，question 是“应该由哪个团队处理？”，choices 是 payment、sales、technical、account，返回 { "department": "payment" }，同时还可以得到不同答案的概率和模型对判断的 confidence。

TypeSafe 将这种接口描述为：unstructured state in → typed probabilistic decisions out，也就是输入非结构化状态，输出带类型的概率判断。

## Jev 的三大核心能力

Jev 目前最核心的几个 primitive，可以简单理解成三种问题。

第一种是 Noul：这件事是真的吗？例如“这个用户是否表现出了流失风险？”，返回类似 true: 0.91, false: 0.09，你可以把它理解成 AI 版的概率判断。

第二种是 Choice：应该选哪个？例如“这个工单应该交给谁？A. 销售 B. 技术支持 C. 财务 D. 人工客服”，Jev 可以返回技术支持 0.82、财务 0.11、人工客服 0.05、销售 0.02。程序可以直接选择最高概率，也可以自己设置阈值。

第三种是 Score：这个东西有多严重？例如“请判断这个投诉的严重程度：1 = 普通，2 = 较低，3 = 中等，4 = 严重，5 = 极其严重”，返回 score: 4，于是你可以在代码中写 if score >= 4: escalate()。

这三种能力组合起来，就已经可以覆盖大量实际的软件判断任务。TypeSafe 的公开资料将 Jev 的核心 primitive 概括为 Choice、Score 和 Noul。

## Jev 的核心不是“结构化输出”，而是“决策原生”

这里有一个非常重要的区别。很多人第一眼看到 Jev，可能会想：“这不就是 LLM 的 Structured Output 吗？”其实并不完全一样。

比如你让 GPT“请判断这条消息是不是垃圾信息，只返回 JSON：{ "spam": true }”，它依然是一个生成文字的模型，只是你要求它把最终生成的文字限制成 JSON。而 Jev 的设计目标从一开始就是：不要生成文字。

TypeSafe 官方把它称为 Decisions, not strings，即“不是字符串，而是决策”。这看起来只是 API 形式上的区别，但实际上可能代表完全不同的模型设计方向。

## 为什么“不生成文字”反而可能让 AI 更快、更便宜

这是 Jev 最值得关注的地方之一。

传统 LLM 做一个判断，实际上走了一条相当复杂的路径：输入 → 理解问题 → 推理 → 组织答案 → 生成 Token → 生成 JSON / 自然语言 → 你的程序解析。但如果你的问题只是“A 还是 B？”，那生成几百个 token 其实没有意义。

Jev 的目标是把这个过程压缩成：输入 → 判断 → A / B + probability。因此，它不需要承担自然语言生成的成本。

TypeSafe 目前宣称 Jev 针对 System One 任务，相比现有 LLM 可以实现数量级上的速度和成本优势；其官网当前展示的一个工作流对比是 193.6× faster、444.6× cheaper。这些属于 TypeSafe 自己公布的测试结果，具体优势会取决于任务和比较方法，因此不应该简单理解成“所有场景都快 193 倍”。

这也是 Jev 背后的一个重要假设：如果一个 AI 调用只是为了做一个判断，那么让它生成文字可能是一种浪费。

## Jev 的独特性：置信度

这可能比“快”和“便宜”更加重要。

假设你让 AI 判断“这个客户是不是高价值客户？”，传统 LLM 可能告诉你：“根据客户的购买历史、活跃程度以及最近的行为来看，该客户很可能属于高价值客户。”问题是，“很可能”到底是多少？60%？80%？99%？软件不知道。

而 Jev 的设计是让判断天然带有概率和 confidence。于是程序可以写成：if confidence > 0.95: auto_execute() elif confidence > 0.75: soft_execute() else: ask_human()。

这实际上给 AI Agent 增加了一个非常重要的能力：知道什么时候应该自己做，什么时候应该停下来。TypeSafe 强调，Jev 的判断包含 confidence，使软件可以自己设置自动执行与人工审核之间的阈值。

## Jev 可能改变 Agent 的架构

如果把这个思路放到 Agent 里，就会变得更有意思。

今天一个 Agent 往往是：LLM 负责 Thinking / Planning，然后几乎所有事情都交给一个大模型，再分发到 Tool A、Tool B、Tool C。但实际上，一个 Agent 里面存在很多非常小的判断：这个任务属于什么类型？应该调用哪个 Tool？Tool 的结果是否可信？要不要继续？是否需要重新尝试？是否应该交给人工？哪个结果最好？

这些任务很多时候根本不需要一个“会写文章”的大模型。于是可以变成：Reasoning LLM 负责 Plan，Jev 判断负责 Route、Score、Verify，Tools 负责执行任务，最后决定 Continue。

换句话说：LLM 负责“想”，Jev 负责“判”，代码负责“做”。这其实是一个非常值得关注的 Agent 架构方向。

举一个具体的例子。

比如你做一个自动化销售 Agent。用户填写：“我们公司大概有 500 人，目前正在考虑采购 AI 客服，希望下个月上线。”

Agent 可以先让 Jev 做几个判断：这是一个潜在销售线索吗？是不是企业客户？规模是否较大？是否存在明确采购需求？是否具有时间紧迫性？

Jev 返回：lead true 0.98、enterprise true 0.96、large true 0.91、intent true 0.97、urgent false 0.72。然后代码自己决定：if lead and enterprise and intent: create_sales_lead()，if urgent and confidence > 0.9: notify_sales_manager()。

注意，Jev 并没有负责整个 Agent，它只是成为 Agent 里面一个非常便宜、非常快速的判断器。

Jev 还有一个非常有意思的设计：同一个 state 可以并行决策多个问题。

例如，State 是用户刚刚发送的客服消息，Questions 包括：是否紧急？是否投诉？是否退款？是否需要人工？属于哪个部门？情绪是否负面？这些问题可以独立且并行决策。

传统 LLM 同样可以在一次调用中同时完成多个判断，例如一次输出“是否紧急、是否投诉、是否退款”等多个字段。Jev 的不同之处在于它把这些判断本身定义成模型原生支持的 typed decision，并让每个 decision 都可以独立获得概率和 confidence。

因此，Jev 更像一个面向软件的“决策层”：同一个 state 可以被映射到多个独立的 decision primitives，而不是把所有判断都封装在一次自然语言生成任务中。对于需要大量、高频、可程序化消费的判断任务，这种抽象可能更自然。

## Jev 最适合什么

如果把 AI 应用粗略分成三类：

第一类是生成，写文章、写代码、写邮件、生成图片、生成视频，传统生成模型更合适。

第二类是推理，复杂数学、复杂规划、长链条分析、复杂 coding，Reasoning Model 更合适。

第三类是判断，是不是？选哪个？有多严重？相关吗？应该路由到哪里？是否继续？Jev 这样的 System One Model 就非常有意思。

因此它并不是"另一个 GPT"，更像是 AI 软件栈里一个新的基础组件。

## Jev 可以替代许多传统 AI 分类器

过去如果我们需要垃圾邮件检测、意图识别、情感分类、内容审核、Lead Scoring、路由、Ranking，可能会训练一个专门的 classifier。但问题是，每换一个任务，就可能需要数据、标注、训练、部署、维护。

Jev 的思路则是：定义问题 → 定义答案空间 → 直接判断。例如 Question 是“这个用户是否准备购买？”，Criteria 是 true = 明确表达购买意愿，false = 没有明确购买意愿。于是很多以前需要单独训练的分类任务，就有可能变成一个 API 调用。

当然，这并不意味着 Jev 会自动替代所有传统 classifier。如果一个任务极其稳定、规模极大、延迟要求极低，那么专门训练的小模型甚至规则系统仍然可能更合适。Jev 更有价值的地方，是大量存在于“规则太死，但训练专用模型又太麻烦”之间的灰色地带。

## Jev 和 Agent 的关系

Agent 正在从“一个大模型干所有事情”变成“很多不同组件协同完成任务”，例如 LLM 下面有 Planner、Memory、Retriever、Tool Router、Verifier、Evaluator、Guardrail、Executor。Jev 非常适合成为其中的 Router、Evaluator、Verifier、Classifier、Ranker、Guardrail。也就是说，它不是 Agent 的大脑，更像 Agent 的神经反射系统。大脑负责复杂思考，但一个生物体并不会每一个动作都经过完整的意识推理。看到危险，判断“危险？”，Yes，躲开，这个过程应该非常快。

这也正是 TypeSafe 为什么选择 System One 这个名字背后的思想之一。它受到 Daniel Kahneman《Thinking, Fast and Slow》中 System 1 / System 2 概念的启发：System 1 更偏快速、直觉式判断，而 System 2 更偏慢速、审慎的推理。TypeSafe 希望构建的是面向软件的快速决策模型。

## Jev 的边界

理解 Jev，最重要的一点其实是：不要把它当成一个更便宜的 GPT。因为它故意放弃了很多能力。

它不能帮你写一篇文章、写一封邮件、写代码、长篇解释一个问题、进行开放式对话。它更适合判断、分类、选择、评分、路由、验证、排序。而且它的判断质量依然取决于你给它的 state 和问题定义得好不好。如果你的输入本身没有足够的信息，比如 State 是“这个客户怎么样？”，Question 是“他是不是好客户？”，Jev 也不可能凭空知道。

所以一个非常重要的工程原则是：把事实放进 state，把规则放进 question / criteria，把最终的业务政策放进代码。例如 State = 客户行为数据，Jev = 判断客户是否存在高价值信号，Code = 决定什么情况下发送优惠券。这三层应该分开。

## TypeSafe 的野心

如果只看产品，你可能会认为 TypeSafe 在做“一个特别便宜的小模型”。但从它自己的产品定位来看，它想做的事情其实更大：让 AI 成为软件可以直接依赖的基础能力。

过去的软件主要由 Code + Database + APIs 构成。AI 加进来之后，我们往往变成 Code + Database + APIs + LLM。但 LLM 和传统软件之间一直存在一个巨大的接口问题：软件世界是确定的类型、结构、函数、返回值，AI 世界是自然语言、概率、不确定性。

Jev 想做的，是把两者之间的接口重新设计成 Software ↔ Typed AI Decision。所以它的名字叫 TypeSafe。它试图让 AI 更像一个软件 primitive：input → typed question → decision → probability → code，而不是 prompt → 一段可能很好、也可能很奇怪的文字 → 程序努力解析。

## AI 正在从“聊天”走向“基础设施”

过去几年，我们理解 AI 的方式非常直观：AI = Chatbot，后来变成 AI = Copilot，再后来 AI = Agent。而 Jev 代表的方向可能是另一层：AI = Software Primitive。

也就是说，AI 不一定需要出现在用户界面里，用户甚至完全不知道它存在。例如用户 → App → Code，里面有 Jev 判断意图、Jev 判断风险、Jev 选择 Tool、Jev 验证结果，还有 LLM 做复杂推理，最后 Code 执行，输出最终结果。用户看到的可能只是“订单已经帮你处理好了”，但背后已经运行了几十次 AI 判断。

这也是为什么：一个 10 0ms、几乎不产生文字的 AI 模型，可能比一个会写 1000 字文章的模型更适合成为软件基础设施。

## Jev 真正值得关注的，不是“它有多聪明”

我认为理解 Jev 时，最值得关注的不是“Jev 能不能打败 GPT？”这个问题本身可能就问错了。

更重要的问题是：我们是不是一直在用一种“生成文本”的模型，解决大量其实只需要“做判断”的问题？如果答案是 yes，那么未来的 AI 软件栈可能不会只有一种模型，而可能变成 Generate、Reason、Decide 三层并行：Generate 用 GPT/Claude，Reason 用 Reasoning LLM，Decide 用 Jev，最后都汇入 Code 执行 Actions。

## 结尾：Jev 可能代表什么

Jev 现在还非常早期，TypeSafe 也明确将它作为 early access 产品推出。因此，现在下结论说它一定会成为新的 AI 基础设施还为时过早。

但它提出的问题非常值得关注：如果 AI 的价值不仅是“生成内容”，而是让软件拥有判断能力，那么我们为什么还要让一个会生成几千个 token 的 LLM 来完成一个只需要 Yes / No 的任务？这可能是 Jev 最有意思的地方。它在问另一个问题：“AI 能不能像一个函数一样，被软件调用？”

如果未来 Agent 真正进入大量生产环境，那么 Agent 可能需要的并不只是一个更强的大脑，它还需要大量快速判断、概率评估、路由、验证、排序、决策。而 Jev 的方向，就是把这些能力从“大模型的一部分能力”，变成一种独立、快速、可组合、机器原生的 AI primitive。
