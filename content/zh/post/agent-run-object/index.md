---
date: '2026-07-27T22:21:51+08:00'
lastmod: '2026-07-27T22:21:51+08:00'
draft: false
title: "03. 没有 Run 对象，你的 Agent 只是个高级 Chatbot 套壳"
slug: agent-run-object
description: "用 Run 对象把一次 Agent 执行过程建模清楚，避免 Agent 只是 Chatbot 套壳。"
tags:
- AI
- Agent
- Engineering
categories:
- AI
comment: false
toc: true
---
# 03. 没有 Run 对象，你的 Agent 只是个高级 Chatbot 套壳

> 系列来源：本文属于 Agent 开发系列，内容来自 [gusibi/molibot](https://github.com/gusibi/molibot) 的真实开发记录与项目文档整理。这是系列第三篇，前两篇分别讲了 Agent 的四层分类和 runtime 的整体架构——没看过不影响理解本篇。

![没有 Run 对象，你的 Agent 只是个高级 Chatbot 套壳](00%20个人/02%20内容生产/cover-image/no-run-no-agent/cover.png)


你有没有过这种经历——

线上 Agent 出问题了，你翻日志翻了半小时，还没搞清楚它当时到底在执行哪个任务。用户问"刚才那个请求还在跑吗"，你只能回答"大概吧"。服务一重启，内存里那些 running 状态全丢了，外面还以为任务卡住了。

如果你有过，那这篇就是写给你的。

上一篇我们区分了 Chatbot、Tool-using Chatbot、Agent Service 和 Agent System。结论很简单：**差别不在模型，而在 runtime。**

但 runtime 这个词还是太大了。真正落到工程里，第一个问题应该更小：

> 一条用户消息进来以后，系统到底怎么把这一轮跑完？

先别急着做多 Agent。也别急着接 Telegram、飞书、微信。更不要一开始就设计一堆工具市场、插件系统、长期记忆。

如果一轮 run 都跑不清楚，后面所有功能都会挂在一团不稳定的东西上。

> **没有 Run 对象的 Agent 系统，本质上还是个 Chatbot 套壳。**

这一篇只讲一件事：一个最小 Agent 服务，如何把一轮 run 跑清楚。

---

## 为什么先讲 run？

很多 Agent demo 没有 run 的概念。

用户发来一句话，程序把历史消息拼一下，调用模型。模型要工具，就执行工具。最后把回答发回去。

看起来也能跑。

但一旦出问题，你很快会发现系统没有抓手。工具失败了，你只看到一条错误日志，不知道属于哪轮对话。用户发"停一下"，你不知道要停哪一次执行。最终回答发出去了，中间工具结果没保存，后面没法复盘。

这些问题的共同根因是：**系统没有把"这一次执行"当成一个对象。**

run 就是这个对象。

- **session**：一段连续对话
- **run**：一次用户输入触发的执行过程

一个 session 里可以有很多 run。比如用户上午问了一次"帮我生成图片"，下午又问"把那张图做成视频"——这是同一个 session，但一定是两个 run。

把这个边界分清楚，后面很多问题都会变简单。

![会话（Session）与 执行（Run）的关系](<01-comparison-session-vs-run.png>)

---

## 一轮 run 从哪里开始？

一轮 run 的起点不是模型调用。

它从用户输入进入系统开始。

假设用户发来一句：

```text
帮我看一下这个目录里有没有过期的配置文件。
如果有，列出来并解释原因。
```

最小 Agent 服务收到这句话后，第一步不是立刻调模型，而是先确定上下文：

```text
inbound message
-> normalize          # 不同入口的消息统一结构
-> find session       # 找到或创建会话
-> start run          # 标记一次执行开始
-> build context      # 准备模型能看到的内容
```

每一步都有意义。

**normalize** 是把 Web 表单、CLI 输入、Telegram 消息这些不同外形的输入，进了 runtime 以后尽量长成一个样子。

**find session** 是为了知道这句话接在哪段对话后面。继续老任务就带历史，开新任务就开新上下文。

**start run** 是给这次执行一个身份证。后面的模型调用、工具调用、失败、停止、最终回答，都要挂到这个 run 上。

**build context** 才是准备模型输入。它决定模型能看到什么：系统规则、用户消息、最近历史、工具结果、相关记忆，以及这轮 run 的临时信息。

所以最小流程不是 `user -> model`，而是：

```text
user -> session -> run -> context -> model
```

![从用户消息到模型输入的生命周期](<02-flowchart-run-start-lifecycle.png>)

多了几步，但每一步都在给后续恢复和排障留位置。

这一路需要哪些对象？一个最小 Agent 服务不需要复杂的对象模型，但至少要有五个。

| 对象 | 回答的问题 | 关键字段 |
|---|---|---|
| **Message** | 模型看到了什么？ | role, content, toolCallId? |
| **Session** | 这句话接在哪段对话后面？ | id, messages[], activeRunId? |
| **Run** | 当前这次任务跑到哪了？ | id, sessionId, status, startedAt, finishedAt? |
| **ToolCall** | 模型想调用什么？ | id, runId, name, arguments |
| **ToolResult** | 工具执行得怎么样？ | toolCallId, status, content, error? |

用一个嵌套结构表示更直观：

```text
Session
  messages[]
  activeRunId?

Run
  id
  sessionId
  status
  startedAt
  finishedAt?
  stopReason?

ToolCall
  id
  runId
  name
  arguments

ToolResult
  toolCallId
  status
  content
  error?
```

这不是数据库设计，只是对象边界。

对象边界清楚，存哪里都好说。可以先用文件，也可以用 SQLite。重要的是**不要把它们混成一坨聊天历史**。

![最小 Agent 服务的 5 个核心实体模型](<03-framework-core-objects.png>)

---

## 模型调用不是终点，而是循环的一步

很多人会把模型调用当成核心。

确实，模型调用很重要。

但在 Agent 服务里，它只是循环的一步。

模型可能返回两类结果——

第一类是普通回答，比如：

```text
这个目录里没有明显过期的配置文件。
```

这时 run 可以进入提交阶段。

第二类是工具调用，比如：

```text
tool_call:
  name: listFiles
  arguments:
    path: "./config"
```

这时 run 不能结束。runtime 要执行工具，把结果回灌给模型，再进入下一轮模型调用。

所以最小 Agent loop 更像这样：

```text
while run is active:
  response = model(messages, tools)

  if response is final answer:
    commit answer
    finish run
    break

  if response has tool calls:
    execute tools
    append tool results to messages
    continue
```

这段逻辑不复杂。

复杂的是边界——工具执行失败怎么办？模型返回空怎么办？用户中途 stop 怎么办？上下文太长怎么办？这些都必须进入 run 的状态。否则循环看起来能跑，实际一碰异常就散。

而这里面最容易被低估的一点，是工具结果必须回灌给模型。

很多系统执行工具后，会把结果直接展示给用户，然后让模型写一句总结。问题是，**模型没有看到工具结果**。

比如模型调用了 `listFiles`，runtime 得到结果：

```text
config/
  app.old.json
  app.json
  sandbox.local.json
```

如果这个结果只显示在 UI，不进入模型上下文，模型下一步就不知道目录里有什么。它可能会编一个总结。

更糟的是工具失败时。假设工具返回：

```text
permission denied: config/
```

如果模型看不到这个失败，它可能仍然回答"我已经检查过了"。这就是很典型的 Agent 幻觉——不是模型凭空坏，而是系统没有把真实观察给它。

所以工具结果要作为 tool result 回到 messages：

```text
assistant:
  tool_call listFiles({ path: "./config" })

tool:
  tool_call_id: "call_123"
  content: "permission denied: config/"
```

模型看到这个结果后，才有机会继续判断：

- 是否换一个路径？
- 是否请求审批？
- 是否告诉用户权限不足？
- 是否停止当前任务？

> Agent 的智能不是模型单独产生的，而是模型和 runtime 通过"观察结果"来回迭代产生的。

![正确的观察迭代循环 vs. 幻觉陷阱](<04-flowchart-agent-loop-feedback.png>)

---

## 三个最容易踩的坑

### 坑 1：状态只存在内存里

最小版本可以从很少的状态开始：

```text
queued / running / waiting_approval / completed / failed / stopped
```

但这些状态**不要只存在内存里**。

只存在内存里的状态，服务重启后就消失。用户以为任务还在跑，系统却已经忘了。或者反过来，系统内存以为没有任务，但外部工具还在执行。

最小持久化可以很朴素：

```text
RunStore.start(run)
RunStore.markRunning(run.id)
RunStore.recordToolCall(run.id, toolCall)
RunStore.recordToolResult(run.id, result)
RunStore.finish(run.id, status)
```

哪怕后面再换数据库或结构，这个事实流也应该保留——它是系统恢复、审计和排障的基础。

### 坑 2：把 streaming 当成最终答案

很多聊天产品都有 streaming。模型一边生成，前端一边显示。用户体验很好。

但在 Agent 服务里，streaming 不能等同于最终回答。

用户看到屏幕上出现了一段字，只能说明系统正在生成草稿。它还没有完成提交。

为什么要区分？因为 run 可能在 streaming 过程中失败——模型生成到一半，发现还需要调用工具。或者用户按了 stop。或者平台消息编辑失败。或者服务重启。

如果系统把 streaming 中的半截内容当成最终答案保存，后面 session 就会被污染。

更好的做法是区分三类输出：

| 类型 | 给谁看 | 进不进模型上下文 |
|---|---|---|
| progress | 用户 | 不一定 |
| draft（streaming 中） | 用户 | 失败时不提交 |
| committed answer | 用户 + 模型 | 正式历史 |

这个边界看起来细，但它会直接影响停止、重试和恢复。

### 坑 3：只写 happy path

最小闭环不是只写 happy path。

Agent 服务的失败路径很多，而且都是正常情况。模型可能返回空响应。工具可能参数错误。工具可能被权限策略拒绝。外部 provider 可能超时。用户可能中途 stop。上下文可能超过模型限制。平台可能发送失败。

如果你只在最外层 catch 一个异常，然后告诉用户"失败了"，系统就失去了继续能力。

更好的方式是把失败分类：

```text
model_empty_response     → 回灌给模型，让它重试
tool_validation_error    → 回灌给模型，修正参数
tool_policy_denied       → 回灌给模型，请求审批
tool_execution_failed    → 回灌给模型，换个方案
user_aborted             → 结束 run，不回灌
context_too_large        → runtime 处理，压缩上下文
delivery_failed          → 用户层面提示，不影响 run 状态
```

给模型看的，是它能用来继续推理的部分。给用户看的，是能帮助他理解当前状态的说明。给开发者看的，是结构化错误码和 runId。

**三者不要混在一起。**

![Agent 执行错误的分流分拣机制](<05-comparison-failure-routing.png>)

---

## 一个完整的最小 run 长什么样？

把前面的内容串起来，可以得到一个更完整的伪代码：

```text
function handleUserMessage(inbound):
  message = normalize(inbound)
  session = SessionStore.findOrCreate(message.conversationId)
  run = RunStore.start(session.id, message)

  try:
    messages = ContextBuilder.build(session, message)

    while RunStore.isActive(run.id):
      response = Model.call(messages, ToolRegistry.available())

      if response.isEmpty:
        RunStore.fail(run.id, "model_empty_response")
        return userMessage("模型没有返回有效内容")

      if response.hasToolCalls:
        for toolCall in response.toolCalls:
          RunStore.recordToolCall(run.id, toolCall)
          result = ToolRuntime.execute(toolCall)
          RunStore.recordToolResult(run.id, result)
          messages.append(asToolMessage(toolCall.id, result))
        continue

      answer = response.text
      SessionStore.commitAnswer(session.id, answer)
      RunStore.finish(run.id, "completed")
      return userMessage(answer)

  catch AbortError:
    RunStore.finish(run.id, "stopped")
    return userMessage("已停止")

  catch error:
    RunStore.fail(run.id, classify(error))
    return userMessage("这次任务失败了，已记录诊断信息")
```

这段伪代码仍然很简化。真实系统还会有 streaming、审批、队列、trace、上下文压缩和多渠道发送。

但它已经包含最重要的骨架：

- 每轮用户输入都有 session
- 每次执行都有 run
- 模型调用在 loop 里
- 工具结果回灌给模型
- 状态和失败被记录
- 最终答案才提交到 session

只要这个骨架稳定，后面扩展就有位置。

---

## 最小 Agent 服务的及格线

"最小 Agent 服务"很容易被误解成"少写一点代码"。

其实不是。最小的意思是：只保留必要对象和必要流程。不是没有状态。不是没有错误处理。不是把所有历史都塞给模型。不是工具失败就直接丢给用户。

一个好的最小版本，应该能回答下面这 7 个问题：

| # | 问题 | 答不上来意味着什么 |
|---|---|---|
| 1 | 这轮 run 的 id 是什么？ | 出了问题找不到根 |
| 2 | 它属于哪个 session？ | 上下文串不起来 |
| 3 | 当前状态是什么？ | 用户问"还在跑吗"你答不上 |
| 4 | 调用了哪些工具？ | 不知道时间花在哪了 |
| 5 | 工具结果回灌了吗？ | 模型可能在幻觉 |
| 6 | 最终答案提交了吗？ | session 历史可能被半截内容污染 |
| 7 | 如果失败，失败类型是什么？ | 没法分类处理，只能笼统说"失败了" |

> 如果这些问题答得上来，就算功能很少，它也是一个 Agent Service 的起点。
> 如果这些问题答不上来，就算工具很多，它也只是一个更复杂的 demo。

---

## 回到开头那个问题

还记得开头那个场景吗——线上出问题了，翻日志翻了半小时，用户问「刚才那个请求还在跑吗」，你只能回答「大概吧」。

如果你现在能回答「run_20260727_001，状态是 waiting_approval，卡在权限检查，需要你确认」，你就不是在写 demo 了。

你在做服务。

---

> 本文中讨论的所有概念——Session、Run、ToolCall、ToolResult——在 [gusibi/molibot](https://github.com/gusibi/molibot) 中都有对应的实现。如果你不想从零搭，可以直接看源码。下一篇我们拆 Agent runtime 的核心对象模型：哪些东西该进模型上下文，哪些只该进 trace。
