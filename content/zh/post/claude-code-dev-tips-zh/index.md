---
date: '2026-03-19T10:27:14+08:00'
lastmod: '2026-03-19T10:27:14+08:00'
draft: false
title: Claude Code 开发者的一些使用建议（中文版）
slug: claude-code-dev-tips-zh
description: 作者：Boris（Claude Code 的创造者）
tags:
- Agent
- Coding
- ClaudeCode
categories:
- AI
comment: false
toc: false
---

# Claude Code 开发者的一些使用建议

作者：Boris（Claude Code 的创造者）

译者：Claude

日期：2026年3月17日

我是 Boris，Claude Code 的创造者。我想快速分享一些来自 Claude Code 团队的使用建议。团队使用 Claude 的方式和我个人使用的方式不太一样。记住：使用 Claude Code 没有唯一正确的方式——每个人的配置都不同。你应该多尝试，找到适合自己的方法！

## 1. 多任务并行处理

同时启动 3-5 个 git worktree，每个运行独立的 Claude 会话。这是最大的生产力提升点，也是团队的首要建议。就我个人而言，我使用多个 git checkout，但大多数 Claude Code 团队更喜欢 worktree——这正是 @amorriscode 在 Claude Desktop 应用中内置支持 worktree 的原因！

有些人还会给 worktree 命名，并设置 shell 别名（za, zb, zc），这样一键就能在它们之间切换。还有人专门设置一个"分析" worktree，只用于读取日志和运行 BigQuery。

参考：https://code.claude.com/docs/en/common-workflows#run-parallel-claude-code-sessions-with-git-worktrees

## 2. 复杂任务先从 plan 模式开始

把精力投入到计划中，让 Claude 一次性完成实现。

有人会让一个 Claude 写计划，然后启动第二个 Claude 以高级工程师的身份审查它。

另一个人说，一旦事情开始偏离轨道，就切换回 plan 模式重新规划。不要硬撑下去。他们还明确告诉 Claude 在验证步骤时进入 plan 模式，而不仅仅是在构建时。

## 3. 投资你的 CLAUDE.md

每次纠正后，以"更新你的 CLAUDE.md，这样就不会再犯同样的错误"结束。Claude 在为自己编写规则方面出奇地擅长。

随着时间的推移，无情地编辑你的 CLAUDE.md。不断迭代，直到 Claude 的错误率明显下降。

一位工程师告诉 Claude 为每个任务/项目维护一个笔记目录，在每次 PR 后更新。然后让 CLAUDE.md 指向它。

## 4. 创建你自己的技能并提交到 git

在每个项目中重复使用。

团队的建议：
- 如果你一天做某件事超过一次，就把它变成技能或命令
- 建立一个 /techdebt 斜杠命令，在每个会话结束时运行，查找并消除重复代码
- 设置一个斜杠命令，将 7 天的 Slack、GDrive、Asana 和 GitHub 同步到一个上下文转储中
- 构建类似分析工程师风格的 agent，编写 dbt 模型、审查代码并在开发环境中测试更改

了解更多：https://code.claude.com/docs/en/skills#extend-claude-with-skills

## 5. Claude 自己修复大多数 bug

我们这样做：

启用 Slack MCP，然后将 Slack bug 线程粘贴到 Claude，只说"修复"。零上下文切换。

或者，直接说"去修复失败的 CI 测试"。不要微观管理如何做。

让 Claude 查看 docker 日志来排查分布式系统——它在这方面出人意料地能干。

## 6. 提升你的提示词水平

a. 挑战 Claude。说"仔细审查我的更改，在我通过你的测试之前不要创建 PR"。让 Claude 成为你的审查员。或者，说"向我证明这有效"，让 Claude 在 main 和你的功能分支之间对比行为差异。

b. 在平庸的修复后，说："根据你现在知道的一切，废弃这个并实现优雅的解决方案"。

c. 在交接前编写详细的规格说明并减少歧义。你越具体，输出就越好。

## 7. 终端和环境设置

团队喜欢 Ghostty！多人喜欢它的同步渲染、24 位颜色和正确的 unicode 支持。

为了更容易地切换 Claude，使用 /statusline 自定义你的状态栏，始终显示上下文使用情况和当前 git 分支。我们中的许多人还会对终端标签页进行颜色编码和命名，有时使用 tmux——每个任务/worktree 一个标签页。

使用语音听写。你说话的速度是打字的三倍，你的提示词因此会详细得多。（在 macOS 上按 fn x2）

更多提示：https://code.claude.com/docs/en/terminal-config

## 8. 使用子 agent

a. 在任何你想让 Claude 投入更多计算资源的请求后追加"use subagents"。

b. 将个别任务卸载给子 agent，以保持主 agent 的上下文窗口干净和专注。

c. 通过钩子将权限请求路由给 Opus 4.5——让它扫描攻击并自动批准安全的请求（参见 code.claude.com/docs/en/hooks#…）

## 9. 使用 Claude 学习

团队使用 Claude Code 学习的一些技巧：

a. 在 /config 中启用"解释性"或"学习性"输出风格，让 Claude 解释其更改背后的*原因*

b. 让 Claude 生成可视化的 HTML 演示文稿来解释不熟悉的代码。它制作的幻灯片出奇地好！

c. 让 Claude 为新协议和代码库绘制 ASCII 图表，帮助你理解它们

d. 构建一个间隔重复学习技能：你解释你的理解，Claude 提出后续问题填补空白，存储结果

---

**译者注**：这篇文章来自 Claude Code 团队的一手经验。老实说，这些建议都是实战总结，不是纸上谈兵。特别是第 1 条"多任务并行"和第 3 条"投资 CLAUDE.md"，可以说是提升效率的关键。说白了，工具再好，也要用对方法。希望这些建议对你有帮助。


Claude Code 开发者的一些使用建议
Learn Claude Code
raw/tech/02 Agent/你不知道的 Claude Code：架构、治理与工程实践
应该知道的 Claude Code：架构、治理与工程实践  --- 总结
Writing a good CLAUDE
