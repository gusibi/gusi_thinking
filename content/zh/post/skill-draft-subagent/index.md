---
date: '2026-05-06T21:48:54+08:00'
lastmod: '2026-05-06T21:48:54+08:00'
draft: false
title: "一次 Skill Draft 命名问题，为什么最后变成了一个专用 Subagent"
slug: skill-draft-subagent
description: "从 Skill Draft 命名问题出发，梳理可复用工作流与专用 Subagent 的边界。"
tags:
- 其它/闪念笔记
categories:
- AI
comment: false
toc: true
type: template
next:
---

有些问题看起来很小。

比如这次，表面上只是 Skill Draft 的 `name` 不好用。

但顺着查下去，它其实暴露了一个更底层的问题：我们到底是在“记录一次对话”，还是在“沉淀一个以后能复用的工作流”？

这两件事长得很像，但对系统来说完全不是一回事。

## 改动前：草稿名来自用户原话

之前 Molibot 会在一次复杂运行成功后，自动保存一个 Skill Draft。

它的目标是把这次跑通的流程沉淀下来，之后遇到类似任务时，不用每次都重新摸索。

问题出在草稿 metadata，尤其是 `name`。

当用户发出这样的消息：

```text
为什么没有昨日数据回顾，只有今天的数据，我这个是要有昨日数据回顾的，要在第一条就列出来昨日数据
```

系统生成的草稿名会接近：

```yaml
name: 为什么没有昨日数据回顾-只有今天的数据-我这个是要有昨日数据回顾的-要在第一条就列出来昨日数据
```

另一个更典型的例子是：

```text
重试一下
```

它也可能变成：

```yaml
name: 重试一下
```

这显然不是一个 Skill 名。

Skill 的 `name` 应该是稳定的功能标识，比如：

```yaml
name: yesterday-data-review
```

用户原话可以保留，但它应该出现在触发描述、示例、上下文里，而不是变成 Skill 的主标识。

## 为什么必须改

Skill Draft 不是聊天记录。

聊天记录关心的是“用户当时怎么说”。

Skill Draft 关心的是“这次跑通了什么可复用能力”。

如果 `name` 直接来自用户原话，会带来几个实际问题。

第一，名字不可读。

长句、抱怨、纠错、口语化表达都会进入 `name`。设置页里扫一眼，根本看不出这是哪个能力。

第二，后续触发不稳定。

Skill 的 metadata 是触发系统理解它的重要入口。`重试一下` 这种名字既不能描述能力，也不能帮助模型判断什么时候该用它。

第三，草稿无法自然升级成正式 Skill。

草稿阶段可以粗糙一点，但不能粗糙到主标识不可用。否则每次 Review 都要人工重命名，自动沉淀的价值就打折了。

第四，和 skill-creator 规范不一致。

项目里已经声明创建 Skill 要遵守 skill-creator 规范：`name` 是 skill identifier，`description` 才负责说明功能和触发场景。

但原来的自动草稿链路只是 runner 的后处理逻辑，不会真正读取并执行这套规范。

这就是问题的根源：规范写在提示词里，但自动保存草稿的代码路径没有应用规范。

## 当时有哪些方案

这次讨论里主要有三个方案。

### 方案一：继续本地启发式命名

最直接的做法，是把原来的“截断用户消息”改成一套更聪明的本地规则。

比如识别“昨日、数据、回顾”，生成：

```yaml
name: yesterday-data-review
```

遇到“重试一下”这种泛化消息时，再从最终答案里反推功能名。

这个方案优点是简单、快、稳定，不需要额外模型调用。

但缺点也明显：规则会越来越多，而且它只是修命名，并没有真正解决“遵守 skill-creator 规范”的问题。

### 方案二：单独做 metadata normalizer

第二种方案，是把 Skill Draft 的 metadata 生成单独拆出来。

runner 不再直接决定 `name`。

它只提供这些输入：

- 用户原始消息
- 最终答案
- 本轮使用过的工具
- 配置的 workflow `SKILL.md`
- 可选的人工指定名称和触发词

然后由一个独立模块输出：

```json
{
  "name": "yesterday-data-review",
  "description": "Use when the user needs ...",
  "aliases": ["yesterday-data-review"]
}
```

这个方案比方案一干净。

它把“是否保存草稿”和“草稿 metadata 怎么生成”拆开了，后续可以换实现，不影响 runner 主流程。

### 方案三：用专用 Subagent 生成 metadata

第三种方案，是再进一步：让主 agent 只判断是否需要生成草稿，一旦确定要生成，就交给一个专用 subagent。

这个 subagent 只负责一件事：

根据本轮运行结果，生成符合 skill-creator 规范的 Skill Draft metadata。

它不参与最终回复，不修改业务文件，不接管 runner。

它的输出也被限制成 JSON。

如果输出不可解析，或者 subagent 失败，系统就回退到本地 normalizer。

## 为什么最后选了“Subagent + 本地回退”

最终选择的是方案三，但不是纯 subagent。

准确说，是：

```text
优先 skill-drafter subagent
失败后回退本地 normalizer
```

这样选有几个原因。

### 1. 它最符合职责边界

主 runner 的职责应该是推进会话、执行工具、判断是否值得保存草稿。

它不应该塞一堆命名规则。

Skill Draft metadata 是一个独立的小任务：总结能力、生成标识符、写清触发描述。

这很适合交给一个专门角色。

### 2. 它能真实验证 subagent 链路

这次也正好想验证 Molibot 的 subagent 是否正常运行。

如果只是写一个本地函数，验证不到 subagent 的真实生命周期。

现在自动草稿生成时，会真的跑一次内置 `skill-drafter` subagent。

日志里能看到：

```text
skill_draft_subagent_start
subagent_start
subagent_task_start
subagent_task_end
subagent_end
skill_draft_subagent_end
```

这比单独写一个 demo 更接近真实产品路径。

### 3. 它不会把系统稳定性交给模型

完全依赖 subagent 有风险。

模型可能返回 Markdown。

可能返回解释。

可能 JSON 不合法。

也可能当前环境没有可用模型。

所以这次没有让 subagent 成为唯一通道。

系统会解析它的 JSON；解析失败就回退本地规则。

也就是说，subagent 是增强路径，不是单点故障。

### 4. 它为以后扩展留了干净接口

现在的接口很明确：

输入是本轮运行摘要。

输出是 Skill Draft metadata。

以后如果要让 subagent 进一步做：

- 合并相似草稿
- 生成更完整的触发描述
- 判断是否应该升级成正式 Skill
- 给 Review 页面写人工建议

都可以沿着这个边界继续扩展。

## 具体改了什么

这次改动主要分成四层。

### 第一层：新增 metadata normalizer

新增了一个独立模块，用来本地生成和兜底规范化 metadata。

它负责处理这些情况：

- 用户原话太长
- 用户消息是抱怨或纠错
- 用户只说“重试一下”
- 需要从最终答案里反推功能
- 需要把中文功能短语转成稳定英文标识

比如：

```text
为什么没有昨日数据回顾...
```

会得到：

```yaml
name: yesterday-data-review
```

而不是原始长句。

### 第二层：新增 skill-drafter subagent

新增内置 subagent：

```text
skill-drafter
```

它的特点很克制：

- 使用 `haiku` 级别模型路由
- 只给 `read` 权限
- 输出只允许 JSON
- 专门生成 `name`、`description`、`aliases`

它不应该改文件，也不应该参与其他业务决策。

### 第三层：runner 自动草稿保存接入 subagent

原来 runner 判断要保存草稿后，会直接调用保存逻辑。

现在多了一步：

```text
shouldSuggestSkillDraft
  -> skill-drafter subagent 生成 metadata
  -> 成功则使用 subagent metadata
  -> 失败则使用本地 normalizer
  -> saveSkillDraft
```

这样，自动草稿生成会真实经过 subagent 路径。

### 第四层：手动 skillManage 也走同一套 metadata

手动创建 draft 时，如果模型或用户传入了 name、description、triggers，也会进入同一套规范化逻辑。

这避免了“自动草稿一套规则，手动草稿另一套规则”的分叉。

## 选择这个方案后的效果

最直接的效果，是草稿名终于像 Skill 名了。

之前：

```yaml
name: 为什么没有昨日数据回顾-只有今天的数据-我这个是要有昨日数据回顾的-要在第一条就列出来昨日数据
```

现在：

```yaml
name: yesterday-data-review
```

之前：

```yaml
name: 重试一下
```

现在会从最终结果里反推：

```yaml
name: yesterday-data-review
```

第二个效果，是 skill-creator 规范不再只是“提示词里说了”。

它开始在自动草稿生成链路里真实落地。

`name` 回到标识符职责。

`description` 承担功能和触发场景。

用户原话留在上下文里，而不是污染主标识。

第三个效果，是 subagent 有了一个很小但真实的产品使用场景。

这不是为了展示 subagent 而硬塞一个复杂流程。

它刚好适合：

- 输入明确
- 输出明确
- 失败可回退
- 不需要写文件
- 能验证模型隔离 session 是否正常

第四个效果，是后续演进空间更清楚。

现在 Skill Draft 的生成流程已经拆成了几段：

```text
是否值得保存
metadata 怎么生成
正文结构怎么生成
相似草稿怎么合并
草稿怎么提升成正式 Skill
```

每一段都可以单独改。

这比把所有逻辑塞在 runner 里健康很多。

## 这次真正修掉的不是一个名字

这次看起来是在修 Skill Draft 的 `name`。

但真正修掉的是一个产品边界问题。

用户说的话，当然重要。

但系统要沉淀的，不是用户这一句话。

系统要沉淀的是：

这句话背后，哪一套流程被跑通了？

以后遇到类似事情，能不能更快、更稳、更少返工？

Skill Draft 的意义就在这里。

所以它的名字不应该像聊天记录。

它应该像一个工具。

短、稳、能复用。

这次把这件事往前推了一小步。

## 本次变更小结

- 修复 Skill Draft `name` 直接来自用户原话的问题。
- 新增本地 metadata normalizer，保证失败时也有稳定兜底。
- 新增 `skill-drafter` subagent，自动草稿保存前优先用它生成 metadata。
- 手动 `skillManage` draft 也复用同一套 metadata 规则。
- 增加测试覆盖“昨日数据回顾”和“重试一下”两个典型问题。

最终目标很简单：

让 Molibot 不只是完成一次任务，而是能把跑通的经验沉淀成真正可复用的能力。
