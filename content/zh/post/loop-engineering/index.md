---
title: Loop Engineering：别再提示 Agent 了，设计让它自己转起来
date: '2026-06-23T23:51:02+08:00'
lastmod: '2026-06-23T23:51:02+08:00'
draft: false
slug: loop-engineering
description: "Loop Engineering 的核心不是继续提示 Agent，而是设计一个能持续运行的工程循环。"
source:
  - "[[Loops explained Claude, GPT, Mira and what actually works]]"
  - "[[Loop Engineering How One Loop Ships 259 PRs a Month]]"
  - "[[wtf is Loop Engineer & how to setup for real]]"
  - "[[Loops What Every AI Engineer Needs to Know in 2026]]"
  - "[[How To Use Loop Engineering To Build A Self-Improving Quant Trading System]]"
  - "[[Loop Engineering A Technical Roadmap for an Autonomous Loop]]"
  - "[[冷饭硬炒？一文讲明白 Loop Engineering]]"
  - "[[WTF Is a Loop? Peter Steinberger vs. Boris Cherny]]"
  - "[[Loop Engineering从 0 到 1 小白完整教程]]"
  - "[[How Quants Use Loop Engineering to Build Alpha (Full Framework)]]"
  - "[[Loop Engineering The AI skill every builder needs in 2026]]"
  - "[[WTF Is a Loop? Part 2 The 15 Loops People Are Actually Running (and the Commands to Steal Them)]]"
  - "[[Loop Engineering Clearly Explained]]"
created: 2026-06-23
tags:
- AI
- Agent
- Loop
categories:
- AI
comment: false
toc: true
---
![Loop Engineering：别再提示 Agent 了，设计让它自己转起来_cover.webp](<Loop Engineering：别再提示 Agent 了，设计让它自己转起来_cover.webp>)

一个工程师，一个月合了 259 个 PR，而且他说这些代码基本不是自己手写的。他做的事情，是写一套 Loop，让 Agent 自己去发现问题、改代码、跑检查、再进入下一轮。另一边，也有人把 Loop 放着跑了 11 天，最后烧掉 4.7 万美元。

同样一件事，一个人把它变成产出，一个人把它变成账单。差别不在模型，在于他们怎么设计那个循环。

Loop Engineering 听起来像新一轮 AI 黑话，但它背后有个真实的变化：

![Loop Engineering：别再提示 Agent 了，设计让它自己转起来_rewritten-1782232168614.webp](<Loop Engineering：别再提示 Agent 了，设计让它自己转起来_rewritten-1782232168614.webp>)

> 人不再站在循环里，一轮一轮提示 Agent。人开始站到循环外，设计那个提示 Agent 的系统。

这句话才是重点。Loop 本身不神秘，最朴素的形态就是一个 `while` 循环：

```text
观察当前状态
决定下一步
执行动作
检查结果
如果没完成，就继续
```


这东西早就存在，ReAct、AutoGPT、LangGraph、human-in-the-loop 本质上都离不开它。2026 年大家重新讨论它，不是因为 `while` 变高级了，而是因为 Agent 能做的事情多了。以前你让它补一段代码，现在你可以让它看 issue、开 worktree、修测试、提 PR、等 CI、处理 review comment，甚至第二天接着跑。

于是问题就变了。不再是"怎么写一句好 prompt"，而是：**怎么设计一套能持续转动、能自我检查、能及时停下来的工作系统。** 这就是 Loop Engineering。

## 为什么大家突然开始讲 Loop？
![01-framework-from-prompt-to-loop.webp](<01-framework-from-prompt-to-loop.webp>)
过去两年，我们用 Agent 的方式大多是这样的：

```text
你写 prompt
Agent 干一轮
你看结果
你指出问题
Agent 再干一轮
你继续看
```

表面上是 Agent 在工作，其实你才是那个 Loop。下一步该干什么、结果对不对、要不要继续、上下文乱了要不要重整，全靠你判断。所以你会累——不是因为模型不够强，而是整个反馈循环还挂在你身上。

Loop Engineering 要做的，就是把这部分外包给系统：

```text
你定义目标
Loop 读取状态
Agent 执行任务
Verifier 检查结果
状态写回文件或系统
Loop 判断继续、停止、回滚或升级给人
```

注意，这不是让 Agent 自由发挥，正相反，Loop 的关键是把自由度收窄。它必须知道目标是什么、当前状态是什么、验证信号在哪里、失败后怎么修、什么时候必须停。

说得再直白一点，这几个工程概念是一层套一层的：Prompt Engineering 解决"怎么问"，Context Engineering 解决"给它看什么"，Harness Engineering 解决"它在哪里干活"，而 Loop Engineering 解决"这套系统怎么自己转起来"。它们不是互相替代，而是叠在一起。

## Loop 到底新在哪里？

很多人第一次看到 Loop Engineering，会觉得这不就是炒冷饭吗？这个反应很正常。如果只看里面那段循环，它确实不新——一个模型调用工具、拿到 observation、再决定下一步，这个模式几年前就有了。

真正的变化在外层。以前我说的 Loop，是 Agent 工作流内部的一段控制逻辑，解决的是"这个任务怎么跑完"。现在**大家讲**的 Loop，更像一个外部操作系统，它要回答的是另一类问题：今天哪些任务值得启动？哪个 Agent 负责实现，哪个负责检查？每个 Agent 去哪个 worktree 干活？失败结果写到哪里？明天从哪里继续？跑偏了谁踩刹车？

这就不是单个 Agent 的 `while True` 了，它变成了"组织 AI 劳动力"的系统。你可以把它理解成三层：

| 层级 | 解决的问题 | 典型部件 |
|---|---|---|
| Agent Loop | 一个任务怎么完成 | Think / Act / Observe、工具调用、错误重试 |
| Harness | Agent 在哪里安全干活 | 工具、权限、上下文、日志、沙箱、状态 |
| Outer Loop | 系统下一步该做什么 | automation、schedule、goal、worktree、verifier、memory |

我也见过一种反驳：Loop 不就是 cron 换皮吗？这个说法对了一半。如果你的 Loop 只是定时跑一个固定脚本，那它确实就是 cron，1975 年就有了，不用重新发明。

但真实的 Loop 中间多了一个东西——**一个会根据当前状态做决策的模型**。cron 只能按固定路径执行，Loop 会读当前状态：看测试为什么失败、看 PR 评论说了什么、看日志里哪个错误最多，然后决定下一步修哪里。所以更准确的说法是：

> Loop 是 cron 加上一个会决策的 Agent，再加上一套能防止它跑飞的工程系统。

新东西不在"定时"，而在"状态驱动的下一步决策"。

## 一个能跑的 Loop 需要什么？

![02-framework-loop-components.webp](<02-framework-loop-components.webp>)

我试着搭过几个 Loop，也拆过别人的实现，最后发现大家都会收敛到差不多的组件。名字不完全一样，但骨架很稳定。下面这六个，我认为是一个能跑的 Loop 的最小集合。

### 1. Automation：心跳

没有自动触发，就只是你手动跑了一次。Automation 可以是 cron、webhook、`/loop`，也可以是某个云端 schedule，它负责让系统动起来。比如：

```text
每 30 分钟检查一次 auth 模块有没有新失败
每天早上扫描过去 24 小时的 production error
每次 PR 更新后自动跑 review loop
```

这里最容易犯的错，是只写触发条件，不写停止条件。"每小时检查一次"只是心跳，"测试通过、lint 干净、PR 描述更新后停止"才是 Loop。没有停止条件的 Loop，就是一台按 token 烧钱的机器。

### 2. State：循环之间唯一可靠的记忆

模型会忘，对话会满，session 会断，所以状态不能只放在 chat history 里。一个靠谱的 Loop 至少要有一个外部状态层，它可以很简单——`STATUS.md`、`progress.md`、`STATE.json`，也可以是 Linear board、数据库、git log。关键不是形式，而是职责：

```text
已经做了什么
正在卡在哪里
下一步是什么
哪些地方永远不要碰
上次失败原因是什么
预算已经花了多少
```

这也是 Ralph Loop 这类做法有意思的地方：每轮都用新的上下文启动，但开头先读进度文件和 git log。换句话说，**记忆不靠模型脑子，靠文件系统和仓库。**

这一点很重要，因为长对话会烂掉。工具输出、错误尝试、历史推理都会堆进上下文，最后模型开始被自己的过去干扰——这个现象有个名字叫 context rot，本质就是上下文越来越脏、质量越来越差、成本越来越高。

一个反直觉的做法是：每轮都"失忆"，但状态写在盘上。Agent 每次只读当前需要的状态、当前失败、相关文件，然后重新开始。这样上下文短、成本稳、行为也更可控。

### 3. Skills：把项目知识写在循环外

如果每一轮 Agent 都要重新理解项目，那 Loop 不会复利，只会重复烧钱。Skills 的作用，是把稳定知识沉淀在循环外：

```text
怎么跑测试
哪些目录不能碰
代码风格是什么
遇到某类错误怎么诊断
PR 描述必须包含哪些内容
```

以前这些东西在工程师脑子里，现在要写成 Agent 能读、能触发、能复用的文件。这跟 Harness 说的同一件事：仓库正在变成 Agent 的大脑。只是到了 Loop 阶段，这个大脑还得支持跨天、跨任务、跨 Agent 的协作。

没有 Skills 的 Loop，就像一个 `while true` 包着一个陌生人，每次都要重新解释背景。有了 Skills，系统才会越跑越省。

### 4. Worktree：让并行 Agent 不互相踩脚

一个 Agent 修登录模块，另一个 Agent 也在修登录模块，如果共用同一个目录，迟早互相覆盖。git worktree 的价值就在这里：每个 Agent 一个独立 checkout、一个独立 branch，共享同一份历史，但工作目录隔离。

但它不是银弹。它只能解决文件冲突，不能解决你 review 不过来的问题。你可以同时启动 10 个 Agent，但最后要不要合并，还是得有人或某个可靠 Gate 来决定。所以 worktree 解决的是"并行执行的物理隔离"，不解决"结果是否可信"。

### 5. Connectors：让 Loop 进入真实世界

一个只能读写本地文件的 Loop，是很小的 Loop。真实工作发生在 GitHub、Linear、Slack、数据库、监控系统、浏览器、CI、broker API 里，Connectors 的作用就是给 Loop 接上这些外部系统。

差别在于：普通 Agent 会告诉你"你可以这样修"，而完整的 Loop 会真的去做——读 issue、开 worktree、改代码、跑测试、开 PR、等 CI、修 review comment、更新 Linear、通知 Slack。这时候 Loop 才从"会建议"变成"会推进"。

但 Connectors 也是风险来源。一旦接上真实工具，Loop 就不只是生成文本了，它可能发消息、下单、改数据库、部署代码。所以越接近真实世界，越需要权限、审批、审计和回滚。

### 6. Verifier：让"完成"变成证据

这是整件事最硬的一层。Loop 最怕的不是模型不会做，而是模型以为自己做完了。所以一个靠谱的 Loop 必须有独立的验证信号：

```text
测试通过
lint 通过
类型检查通过
页面截图符合要求
回测通过 out-of-sample gate
错误率下降到阈值以下
```

"看起来更好了"不算，"Agent 说完成了"更不算。最好把 Maker 和 Checker 分开：写代码的是一个 Agent，检查的是另一个 Agent，甚至用不同模型。这和人类 code review 一样——写作业的人给自己打分，天然容易放过自己。

我印象最深的是量化交易这个场景：模型可以生成一千个策略，但只有 Loop 能告诉你哪个策略真的活过了样本外测试。如果你只在同一段历史数据上反复优化，Loop 不会更快找到 alpha，只会更快拟合噪声。这也是所有领域通用的规律：**没有可信 Gate 的 Loop，只是在自动化犯错。**

## Open Loop 和 Closed Loop，不要混着用

![03-comparison-open-vs-closed.webp](<03-comparison-open-vs-closed.webp>)

还有一个区分我觉得特别实用：Open Loop 和 Closed Loop。

Open Loop 是开放探索。你给 Agent 一个大目标，让它自己发现、规划、执行、分解，再派发更多 Agent。它很有想象力，也很贵，适合那种你有足够预算、有强隔离、有强观测，而且能接受很多探索失败的场景。

Closed Loop 是封闭循环。你给它明确的边界、输入、检查和停止条件，它能做的事情少一些，但更稳定、也更便宜。比如：

```text
发现 failing test
读取相关文件
修复
再次运行测试
如果通过就停止
如果连续 3 次无进展就交给人
```

大多数团队今天应该从 Closed Loop 开始，原因很简单：它可控。Open Loop 的演示视频更好看，但 Closed Loop 更容易真的帮你省时间。

## 什么时候值得做成 Loop？

不是所有任务都值得 Loop。如果只是一次性写一段文案、问一个问题，一个好 prompt 就够了。Loop 有搭建成本，也有运行成本，它会消耗 token、占用工具、制造 review 压力。判断标准可以简单一点，下面四条最好同时成立。

**第一，任务会重复。** 至少每周发生一次，比如 CI 失败处理、PR review、内容选题收集、生产错误归因、数据拉取、策略回测。一年只做一次的事，不要上来就设计 Loop。

**第二，结果可以自动检查。** 有测试、linter、指标、截图检查或回测 Gate。如果质量完全靠主观判断，Loop 很难闭合，它可以辅助，但不适合全自动。

**第三，Agent 能真的执行。** 它要有工具、有权限、有上下文。如果它只能告诉你"建议这样做"，那不叫 Loop，只是一个会重复提建议的聊天窗口。

**第四，失败代价可控。** 能隔离、能回滚、能限额、能停。如果一个错误动作会直接影响线上用户、真实资金或不可逆数据，那必须加人工审批。写操作不要只靠 prompt 约束，要靠 runtime、policy、approval、audit log。

## Loop 最容易死在哪？

![04-infographic-loop-guardrails.webp](<04-infographic-loop-guardrails.webp>)

比起"怎么搭 Loop"，我觉得更值得记的是它怎么死。下面这几个坑，我自己踩过，也反复看到别人踩。

**一是没有停止条件。** Loop 一直在跑，看起来很努力，但没有靠近目标。解决办法很土，就是给它装上最大迭代次数、最大预算、最大运行时间、无进展检测和明确的 done 条件。这些东西不性感，但必须先装——车还没发动，刹车要先有。

**二是验证器被糊弄。** Agent 为了通过测试可能去改测试，为了让指标变好可能绕开指标，为了让截图不报错可能直接隐藏组件，这就是 reward hacking。所以 Gate 不能太天真：测试文件是否允许修改？关键指标是否来自独立系统？Verifier 是否和 Maker 隔离？这些都要提前想清楚。

**三是上下文越跑越脏。** 一开始 Loop 很聪明，跑到第 20 轮开始胡言乱语。不是模型突然变傻，是上下文里塞满了旧失败、旧输出、旧推理。解决办法是把状态放到外部，每轮只组装窄上下文——当前状态、当前失败、相关文件、上一轮 diff、明确预算——不要把整个仓库、整段对话、所有日志都塞进去。上下文是预算，不是垃圾桶。

**四是成本不是线性增长。** 如果让对话一直累积，第 1 轮读 1 轮历史，第 10 轮读前 9 轮，第 50 轮读前 49 轮，这就不是"N 次模型调用"那么简单了。所以 stateless loop 很重要：每轮用短上下文重新启动、读取外部状态，而不是拖着整段聊天往前跑，成本才有可能接近线性。

**五是人的位置错了。** Loop 不是让人消失，而是把人的位置从"每轮按回车"挪到"设计目标、边界、验证和升级路径"。如果你把人完全拿掉，又没有可信 Gate，最后只是把错误规模化。这也是 Loop Engineering 最容易被误解的地方——它不是偷懒，它要求你更像一个系统设计者。

## 我的判断：Loop 是 Harness 的下一层

如果把 Harness Engineering 那篇接着往下写，Loop Engineering 正好是下一层。Harness 解决的是 Agent 的运行环境，Loop 解决的是这个环境如何持续驱动工作。可以用一个公式概括：

```text
Loop = Harness + Cadence + Gate + Feedback + Memory
```

Harness 是壳，Cadence 是节奏，Gate 是闸门，Feedback 是反馈，Memory 是跨轮次的积累。少任何一个，Loop 都会变形：只有 Harness 没有 Cadence，它只是一个能跑 Agent 的环境；只有 Cadence 没有 Gate，它是定时烧钱；只有 Feedback 没有 Memory，每一轮都从零开始；只有 Memory 没有隔离和权限，迟早把脏状态写进系统。

所以 Loop Engineering 的重点不是"让 AI 多跑几次"——多跑几次只是重复。真正的 Loop 要知道差距在哪里、反馈从哪里来、怎么判断变好还是变坏、什么时候继续、什么时候停、什么时候让人接管。

## 七个可以直接抄的杠杆

如果你现在就想做一个 Loop，不要一上来搞多 Agent 大军，先做小。

**1. 从一个重复任务开始。** 比如每天扫一遍 failing CI、每晚整理当天新增 issue、每次 PR 后做一次 adversarial review、每周检查文档和代码是否不一致。任务越小，越容易闭合。

**2. 写清楚完成合同。** 不要写"把这个模块修好"，要写成可验证的条件：

```text
auth 目录下所有测试通过
lint 通过
没有修改测试断言
PR 描述包含风险和验证步骤
最多尝试 5 轮
```

Loop 需要 contract，不需要愿望。

**3. 状态写到文件。** 最小版本就两个文件，`STATUS.md` 给人看、`.loop_state.json` 给机器读。人看的可以自由一点，机器读的要结构化。

**4. 每轮重新组装上下文。** 不要让对话无限变长，每一轮只给 Agent 这些：任务目标、当前状态、当前失败、相关文件、允许修改范围、停止条件。这比"把所有历史都带上"更稳。

**5. Maker 和 Verifier 分开。** 一个 Agent 负责做，另一个 Agent 或确定性工具负责查。能用测试就用测试，能用 linter 就用 linter，能用截图就用截图，LLM Judge 是补充，不是第一道闸门。

**6. 先装刹车。** 最小刹车包括 max iterations、max budget、timeout、no-progress detector、human escalation、rollback plan。这些不是上线后再补的东西——Loop 一旦能自己跑，刹车就必须先存在。

**7. 把成功经验沉淀成 Skill。** Loop 跑完一次，别只看结果，看它哪里卡住、哪里需要你反复解释、哪里总犯同一个错，然后把这些写进 Skill、规则、检查器或状态模板。这才是复利。

## 最后

Loop Engineering 不是让 AI 替你思考所有事，而是把你已经在重复做的判断、检查、推进和收尾，设计成一个可运行的系统。

真正的变化不是"AI 会循环了"，而是**你不再是那个循环——你开始设计循环。** 这件事听起来像自动化，落到工程里其实更像管理：给目标，给上下文，给工具，给边界，给反馈，给升级路径，然后看它跑。

车速越快，护栏越重要。Agent 越强，Loop 越需要被认真设计。
