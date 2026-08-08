---
date: '2026-06-12T15:41:22+08:00'
lastmod: '2026-06-12T15:41:22+08:00'
draft: false
title: Harness 的实践：使用skill search 提高 skill 调用准确度
slug: harness-skill-search
description: 最近给自己的 Agent 做了一次关键的改造。表面上看，只是新增了一个 `Skill Search` 工具，但真正改变的，是 Agent
  的工作方式。
tags:
- 公众号
- Agent
- 架构设计
categories:
- AI
comment: false
toc: false
---

最近给自己的 Agent 做了一次关键的改造。表面上看，只是新增了一个 `Skill Search` 工具，但真正改变的，是 Agent 的工作方式。

以前 Skill 的调用非常不稳定。明明已经很明确告诉它"搜索天气""搜索新闻"，也提供了相应的工具，但它就是不能自动触发。尝试优化提示词、改进 Skill 描述，做了很多尝试，效果都不理想。

后来看到 Claude Code 的源码，里面有一个 `Tool Search` 机制：每次使用工具之前，先查询一下有没有可用工具。受这个启发，我做了一个 `Skill Search` 工具，希望在 Agent 触发能力边界时——比如需要搜索网络、写脚本、调用外部命令时——先检查有没有现成 Skill 可以用，而不是从头造轮子。

## 核心问题

原来的 Skill 调用机制是这样的：把所有 Skill 的名字和描述都塞进系统提示词里，然后告诉模型"如果当前任务适合某个 Skill，就优先使用"。

这个思路一开始是成立的，模型确实"看得到"这些 Skill，也能在某些场景下主动调用。但真正用久了之后，问题就暴露出来了。

**只有显式调用才稳定**。当直接输入 `/web-search` 或明确指定某个 Skill 时，没问题——因为这时候不是模型在判断，而是用户替它做了决策。但真实使用中，用户更常见的表达是：

- "帮我搜一下这个话题"
- "去网上查一下最近有什么信息"
- "看看有没有现成工具能做这件事"
- "帮我处理一下这个文件"

这些输入对人类来说已经足够明确，大家都知道这时候应该优先检查 Skill。但模型不是每次都这么做。很多时候，明明已经有现成 Skill，它还是直接跳过去，自己动手。

**根本原因不是 Skill 描述不够好，而是机制本身有问题。** 原来的调用流程太长了：模型先看用户输入，再判断是否需要执行任务，再回忆提示词里有没有对应 Skill，再匹配描述，最后才决定要不要调用。只要其中任何一步松掉，它就会进入更省事的路径：自己来做。

随着 Skill 越来越多，问题还会继续恶化。Skill 越多，提示词越长，模型越容易漂移。这些 Skill 只是混在上下文里的动态信息，对模型来说不是强制执行的流程，只是可能参考的背景材料。

## 改造方案

核心上做了三件事。

### 1. 从隐式到显式

把 Skill 从"提示词中的隐式能力"变成"运行时中的显式能力"。新增真正的 `skill_search`，让 Agent 在面对非纯文本任务时，不是默认自己动手，而是先查有没有现成 Skill。

### 2. 重新结构化提示词

参考 Claude Code 的结构，把提示词拆成明确的块，用 XML 标签把不同职责分开：

- 工具使用规则块
- Skill 规则块
- 环境信息块

这样做是为了减少模型注意力漂移，让"什么时候要先查 Skill"变成一条更清晰的运行规则。

### 3. 两层搜索机制

Skill Search 拆成两层：
- 第一层：本地搜索（快速、便宜）
- 第二层：AI 意图识别（语义理解）

只靠关键词不够，只靠 AI 又太贵太慢。两层结合，才是适合当前阶段的方案。

## 提示词改造

原来的提示词是这样的：

```text
你可以使用这些 Skills：
- search: 用于搜索网页信息
- browse: 用于网页操作
- imagegen: 用于生成图片
...

如果用户请求适合某个 Skill，请优先使用 Skill。
```

问题是没有"硬边界"。它只是展示信息，希望模型自己判断。对"稳定性"没有保障。

改造后变成：

```xml
<skill-policy>
If the task is not a direct text-only answer, first check whether an installed skill can handle it.
Use skill_search before falling back to generic tools or custom code.
Explicit slash skill invocation still has highest priority.
</skill-policy>

<available-skills>
search
browse
imagegen
...
</available-skills>

<tool-policy>
If no suitable skill is found, then use normal tools.
Prefer existing reusable capability over writing fresh code.
</tool-policy>
```

最关键的变化不是标签本身，而是规则表达方式：
- 以前是"你可以用这些 Skill"
- 现在是"如果这不是纯文本回答，那你先查 Skill"

一个是提醒，一个是流程。

## 决策链设计

整个流程拆成了明确的判断链：

```typescript
function handleTask(userRequest) {
  // 显式调用优先级最高
  if (hasExplicitSkillInvocation(userRequest)) {
    return runExplicitSkill(userRequest)
  }

  // 纯文本回答不需要搜 Skill
  if (isDirectTextAnswer(userRequest)) {
    return answerDirectly(userRequest)
  }

  // 涉及执行、搜索、联网等，先搜 Skill
  const skillMatches = skillSearch(userRequest)

  if (skillMatches.hasClearWinner) {
    return runSkill(skillMatches.topSkill)
  }

  return continueWithNormalTools(userRequest)
}
```

## 本地搜索层

第一反应可能想用向量搜索，但在当前阶段，更需要的是先立起一条稳定的搜索链路。所以本地搜索用的是轻量方案：

```typescript
function searchSkillsLocally(intent, skills) {
  return skills
    .map(skill => {
      let score = 0

      if (intent contains skill.name) score += 50
      if (intent overlaps skill.aliases) score += 30
      if (intent overlaps skill.description keywords) score += 20

      return { skill, score }
    })
    .filter(match => match.score > 0)
    .sort(byScoreDesc)
}
```

本地层先做一轮低成本召回，优点是快、便宜、结果可解释，调试方便。

## AI 复判层

本地搜索解决不了所有问题。用户的表达经常是自然的、口语的、抽象的，甚至和 Skill 描述不在同一个语言里。比如 Skill 写的是中文"网页搜索"，用户输入英文 "research this topic"。这种时候只靠关键词，命中率不够稳定。

所以加了 AI 路由判断层。它只做一件事：根据当前意图和候选 Skill，判断有没有合适 Skill，以及哪个最合适。

```text
User intent:
需要联网搜索最近关于某个主题的信息，并整理结果

Available skills:
- name=search; description=用于网页搜索和在线信息查询
- name=browse; description=用于浏览器自动化操作
- name=imagegen; description=用于生成图片

Choose at most one skill.
Return JSON only with keys: matched, skillName, confidence, reason.
If no skill clearly fits, return matched=false.
```

两个设计点很关键：

1. **最多选一个**：这是路由，不是推荐列表，不允许模棱两可
2. **结构化返回**：JSON 比自然语言更容易消费、更稳定解析

AI 这一层不再是模糊的"帮我想一想"，而是真正可控的路由器。

## 工程层处理

#### 设置开关

为了测试效果，加上了本地搜索和 AI 搜索做成了两个开关，可以单独测试，也可以组合观察。调试阶段这很重要，能看清楚每层在干什么。

#### Provider 复用

直接复用系统现有的 AI Provider 配置，不再单独搞一套。好处是配置来源统一，不会出现一套功能维护两套配置的混乱局面。

#### 兼容和回退

- **兼容旧配置**：如果没有选 Provider 但旧的直连配置完整，仍然允许继续使用
- **模型回退**：如果保存的模型名已不属于当前 Provider，自动退回到默认模型

这些细节决定了配置变化后系统会不会突然掉到很难定位的失败状态。

## 可观测性

Skill Search 最怕"感觉变好了"，却不知道为什么。所以加了一套日志，记录完整决策链：

- 这次搜索的意图
- 本地/AI 搜索是否开启
- 本地搜到的候选
- AI 的改判结果
- 最终选中的 Skill
- 没命中的原因

```typescript
log("skill_search_start", { intent, localEnabled, apiEnabled })
log("skill_search_local_result", { matches })
log("skill_search_api_result", { matched, skillName, confidence })
log("skill_search_end", { finalMatches, source })
```

有了这套日志，测试时就能真实看到它到底有没有先搜、搜到了什么、为什么选择这个 Skill。把"Skill 触发准不准"从体感问题变成可观测问题。

## 效果对比

改造前后的逻辑对比：

**以前**：你先做任务，如果你记得有 Skill，再去用。
**现在**：你先查 Skill，如果没有，再去做任务。

顺序一换，整个系统行为立刻不一样。以前 Skill 更像"也许会被参考"的知识块，现在变成了一条真实的决策路径。

测试中效果非常明显。以前很多任务只有明确写 `/search` 才稳，现在即使只是自然语言说"帮我查一下""去网上搜一下""看看有没有现成工具"，它也会先经过 Skill Search 再决定。

用户未必能看出背后做了什么，但使用感受会明显稳定很多。你会感觉到这个 Agent 更像在"优先复用已有能力"，而不是每次都从零开始 improvisation。

## 总结

如果想要能力稳定，就不要只把它写在提示词里。

提示词适合表达原则、约束风格、提供背景。但一旦一个行为真的重要——比如"遇到执行类任务时，要先检查有没有现成 Skill"——那它就不应该只是提示词里一句"优先使用 Skill"，而应该是一条真正存在的运行时流程。

这次做的就是：把 Skill 从"模型可能会想起来的东西"，变成"系统必须先检查的一步"。这一步补上后，很多问题会一起变好：

- Skill 调用更稳定
- 提示词更轻
- Token 压力更低
- 行为更容易调试

如果你也在做 Agent，也遇到过"明明有工具 Skill，但模型就是不稳定触发"的问题，建议试试这个思路。不要急着往系统提示词里堆更多说明，也不要一上来就上很重的检索基础设施。先把"搜索 Skill"这件事，做成一个明确步骤。

很多时候，你缺的不是更聪明的模型，而是一个更明确的入口，这也算是一个harness 的实践。
