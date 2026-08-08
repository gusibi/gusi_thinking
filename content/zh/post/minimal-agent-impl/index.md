---
date: '2026-03-06T10:19:31+08:00'
lastmod: '2026-03-06T10:19:31+08:00'
draft: false
title: 如何实现一个极简的 Agent
slug: minimal-agent-impl
description: 这两年大家一提 Agent，脑子里很容易浮现出一个“会思考、会规划、会调用工具、还会自我修复”的高级智能体。听起来很玄，但如果把那些花哨概念都剥掉，一个能跑起来的极简
  Agent，其实没有那么复杂。
tags:
- AI
- Agent
categories:
- AI
comment: false
toc: false
---

![如何实现一个极简的 Agent-cover](如何实现一个极简的 Agent-cover.webp)

> 如何从 0 实现一个极简 Agent


这两年大家一提 Agent，脑子里很容易浮现出一个“会思考、会规划、会调用工具、还会自我修复”的高级智能体。听起来很玄，但如果把那些花哨概念都剥掉，一个能跑起来的极简 Agent，其实没有那么复杂。

说到底，它就两件事：

- 一个持续运行的 `while` 循环。
- 一套精心设计的上下文工程。

大模型本身既没有状态，也没有手脚。它只负责在当前上下文里做判断：这轮该说话，还是该调用某个工具；如果调用工具，工具名是什么，参数应该怎么填。真正读文件、执行命令、写入内容、保存记忆、控制权限的，都是你写的程序。

所以 Agent 的核心从来不是“让模型像人一样思考”，而是：**如何把合适的信息在合适的时机交给模型，再把模型的决策稳稳地落到执行环境里。**

这篇文章就结合我手头这套玩具代码，从 0 到 1 拆一下：如何把一个看起来很复杂的 Agent，拆成几个可以逐步实现的小能力。

## 先讲结论：Agent 的最小闭环是什么？

![如何实现一个极简的 Agent-loop](如何实现一个极简的 Agent-loop.webp)

一个最小 Agent，至少要有下面这条闭环：

```text
用户提出任务
-> 模型判断是否需要工具
-> 程序执行工具
-> 把执行结果回填给模型
-> 模型继续决策
-> 直到模型不再调用工具，输出最终答案
```

这个流程也可以写成更工程化一点的五步：

1. 定义工具，并用 JSON Schema 描述参数约束。
2. 把用户消息、系统提示词、工具清单一起发给模型。
3. 模型返回普通文本，或者返回 `tool_call`。
4. 程序解析 `tool_call`，在本地执行真实工具。
5. 将工具结果作为 `tool` 消息带回模型，进入下一轮。

只要这条链打通，一个最简 Agent 就已经成立了。

## 为什么说本质是 `while` 循环？

因为 Agent 和普通聊天机器人的根本区别，不在于“更聪明”，而在于“能继续行动”。

普通聊天模型通常是一次请求、一次回复，停在那里。Agent 则是在程序外面再包一层循环，让它可以不断经历：

```text
Think -> Act -> Observe -> Think
```

这在代码里其实非常朴素。`simple-agent.py` 里就是典型的双层循环：

- 外层循环处理用户输入，构成一个 REPL。
- 内层循环负责模型的自主行动，只要模型还在请求工具，就一直跑下去。

```python
while True:  # 外层：处理用户输入
    user_input = input("\n👤 你: ")
    if user_input.lower() in ["exit", "quit"]:
        break

    messages.append({"role": "user", "content": user_input})

    while True:  # 内层：Think -> Act -> Observe
        response = client.chat.completions.create(
            model="doubao-seed-2.0-code",
            messages=messages,
            tools=TOOLS_SCHEMA,
        )

        message = response.choices[0].message
        messages.append(message)

        if message.tool_calls:
            for tool_call in message.tool_calls:
                func_name = tool_call.function.name
                func_args = json.loads(tool_call.function.arguments)
                result = AVAILABLE_FUNCTIONS[func_name](**func_args)

                messages.append({
                    "role": "tool",
                    "tool_call_id": tool_call.id,
                    "name": func_name,
                    "content": result,
                })
        else:
            print(f"\n🤖 回复:\n{message.content}")
            break
```

这段代码不高级，但已经足够说明问题：**Agent 的运行感，本质上就是程序不断把“观察结果”重新塞回上下文，驱动模型继续往前走。**

## 示例 1：先做一个能干活的最简版本

这一版要解决的问题很单纯：**先让模型不只是会回答，而是真的能动手做事。**

最小化实现时，我只给 Agent 4 个基础工具：

- `execute_bash`：执行 shell 命令。
- `read_file`：读取文件内容，并自动加行号。
- `write_file`：创建或覆盖文件。
- `list_files`：列出项目里的文件结构。

这四个工具已经足够覆盖一个非常朴素的“代码助手”场景：先看目录，再读文件，需要时写文件，最后跑命令验证。

例如 `execute_bash` 和 `read_file` 的实现都很直接：

```python
def execute_bash(command: str) -> str:
    """执行 Bash 命令"""
    result = subprocess.run(
        command, shell=True, capture_output=True, text=True, timeout=300
    )
    output = result.stdout + result.stderr
    if len(output) > 2000:
        output = output[:2000] + "\n... (输出过长已截断)"
    return output.strip() or "(无输出)"


def read_file(path: str) -> str:
    """读取文件内容"""
    if not os.path.exists(path):
        return f"文件不存在: {path}"

    with open(path, "r", encoding="utf-8") as f:
        content = f.read()
        lines = content.splitlines()
        numbered_lines = [f"{i + 1:4d} | {line}" for i, line in enumerate(lines)]
        return "\n".join(numbered_lines)
```

这两个细节其实都很关键。

### 1. 为什么命令输出要截断？

因为工具输出最终都会进入模型上下文。如果不截断，`pytest`、`npm install` 或者 `tree` 这种命令一跑，很容易直接把上下文打爆。极简 Agent 的第一原则不是“信息越多越好”，而是“信息足够且可控”。

### 2. 为什么读取文件时要加行号？

因为模型处理代码时，天然更适合引用“位置”而不是纯文本。加了行号之后，模型可以更稳定地表达：

- 问题在第 42 行附近。
- 需要修改第 18 到 25 行。
- 报错和第 10 行的 import 有关。

这属于很小的工程处理，但对可用性帮助很大。

## 工具为什么一定要用 Schema 描述？

因为你不能只在提示词里写一句“你可以读取文件”。那样对模型来说太模糊了。

真实可用的方式，是把每个工具都声明成结构化接口。`simple-agent.py` 里每个工具都配了 JSON Schema，明确告诉模型：

- 工具叫什么。
- 适合干什么。
- 参数有哪些。
- 哪些参数必填。

例如：

```python
{
    "type": "function",
    "function": {
        "name": "write_file",
        "description": "创建或覆盖文件内容。",
        "parameters": {
            "type": "object",
            "properties": {
                "path": {"type": "string", "description": "文件路径"},
                "content": {"type": "string", "description": "要写入的完整内容"},
            },
            "required": ["path", "content"],
        },
    },
}
```

这一步的价值，在于把“自然语言能力”变成“半结构化决策”。模型不再只是“理解你想写文件”，而是必须产出一个符合约束的参数对象。

所以很多人说 Agent 是“LLM + Tools”，我更愿意写成：

```text
Agent = LLM + Tool Schema + Runtime Loop
```

少掉任何一个，都不稳定。

## 示例 2：如果想让它从“一次性执行”变成“可持续交互”

这一版要解决的问题是：**任务做完之后，怎么让 Agent 不停在原地，而是自然进入下一轮。**

最简版本虽然能工作，但它更像“一次性工具”：用户提一个需求，模型做完，流程结束。

如果你希望它像一个真正的助手，而不是一次性脚本，一个很直接的方向就是：让它在完成当前任务后，能够自然地进入下一轮交互。

这里最简单的做法，不是重写整个架构，而是补一个专门负责“续轮”的工具。`simple-agent-v2.py` 里我加了一个 `continue_interaction`：

```python
def continue_interaction() -> str:
    """询问用户是否继续交互"""
    print("\n" + "=" * 50)
    user_choice = input("继续? [按 N 退出，或输入新的指令] ").strip()
    print("=" * 50)

    if user_choice.lower() in ["n", ""]:
        return "__EXIT__"
    return user_choice
```

这个设计看起来有点“绕”：明明程序自己就能 `input()`，为什么还要把“继续吗”也做成一个工具？

因为这样做之后，**是否进入下一轮交互，不再只是主程序的控制逻辑，而是变成 Agent 工作流的一部分。**

也就是说，系统提示词里可以明确要求模型：

> 每次完成任务、给出最终回复后，必须调用 `continue_interaction`，询问用户是否继续。

这样一来，如果你想把一次执行改造成“连续会话”，路径就很清楚了：

```text
完成当前任务
-> 给出答案
-> 主动确认是否继续
-> 若继续，携带旧上下文进入下一轮
```

在运行时，程序只要识别特殊退出信号即可：

```python
if result == "__EXIT__":
    print("\n再见！")
    return
```

从工程角度看，这一步不是必须的，但它是一个非常轻量、非常实用的升级。只加一个工具和一条提示词规则，就能把 Agent 从“一次性执行器”变成“可持续交互的会话体”。

## 示例 3：如果想引入 Skills，最简单怎么做？

![如何实现一个极简的 Agent-skills](如何实现一个极简的 Agent-skills.webp)

这一版要解决的问题是：**当能力越来越多时，怎么扩展 Agent，而不把主提示词写成一团巨大的说明书。**

Skill 这两年很火，原因也不复杂：它提供了一种很自然的方式，让 Agent 在不膨胀主提示词的前提下，按需获得额外能力。

如果你也想给自己的 Agent 加一个“Skill 系统”，最简单的做法其实不是上来就做复杂的插件框架，而是先把 Skill 当成一种**按需加载的外部说明书**。

很多人第一反应是：把所有说明文档、所有规则、所有用法，统统塞进系统提示词。

这在初期看似简单，后期几乎一定崩。

原因很直接：

- Token 成本越来越高。
- 无关信息越来越多。
- 模型更容易被噪音干扰。
- 每一轮都重复注入同一大坨文本，非常浪费。

如果你想用一个很轻的方法实现 Skills，`simple-agent-skills.py` 的思路其实就够用了，本质上就是**渐进式加载（Progressive Disclosure）**。

技能目录大概长这样：

```text
skills/
├── skill1/
│   ├── SKILL.md
│   ├── scripts/
│   ├── references/
│   └── assets/
└── skill2/
    └── SKILL.md
```

其中 `SKILL.md` 既是说明文档，也是技能入口。实现上不需要什么复杂协议，先约定每个 Skill 都有一个 `SKILL.md`，并在文件头放上最基本的 frontmatter，比如 `name` 和 `description`。

然后 `skills_loader.py` 在启动时做两件事：

- 扫描 `skills/` 目录。
- 解析每个 `SKILL.md` 的摘要信息。

```python
def parse_skill_frontmatter(content: str) -> Optional[dict]:
    lines = content.split('\n')
    if not lines[0].strip() == '---':
        return None

    end_idx = -1
    for i, line in enumerate(lines[1:], 1):
        if line.strip() == '---':
            end_idx = i
            break

    if end_idx == -1:
        return None

    yaml_lines = lines[1:end_idx]
    body_content = '\n'.join(lines[end_idx + 1:])

    metadata = {}
    for line in yaml_lines:
        line = line.strip()
        if ':' in line:
            key, value = line.split(':', 1)
            metadata[key.strip()] = value.strip().strip('"').strip("'")

    if 'name' not in metadata or 'description' not in metadata:
        return None

    return {
        'metadata': metadata,
        'body': body_content,
        'full': content
    }
```

这段代码并不复杂，但已经能把一个“极简 Skill 系统”跑起来了。关键点在于：启动时只加载“摘要”，不加载“全文”。

系统提示词里给模型看的，不是每个 Skill 的完整正文，而是一个技能目录：

```python
def get_skills_prompt() -> str:
    lines = ["可用 Skills:", ""]

    for skill_name, skill_data in LOADED_SKILLS.items():
        metadata = skill_data['metadata']
        lines.append(f"  - /{metadata['name']}: {metadata['description']}")
        lines.append(f"    SKILL 位置: {skill_data['skill_md_path']}")

    return "\n".join(lines)
```

也就是说，模型先知道“有什么技能”，如果任务真的需要，再通过 `read_file` 读取某个 `SKILL.md` 的完整内容，或者继续去读它的 `scripts`、`references`、`assets`。

这是一种非常典型的 Agent 上下文工程模式：

- 第一层只暴露索引。
- 第二层按需读取正文。
- 第三层按需展开依赖资源。

如果你只是想快速给 Agent 加一个 Skill 概念，这个方案已经有几个明显优点：

- 实现简单，本质上只是目录扫描加 Markdown 约定。
- 扩展成本低，新增 Skill 基本就是加一个目录。
- 不会把所有 Skill 正文都塞进上下文。

换句话说，Skill 不一定非得是一个复杂框架。最小实现完全可以只是：**“给模型一份技能索引，需要时自己去读说明书。”**

## 示例 4：如果想加一个极简 Memory，可以怎么做？

这一版要解决的问题是：**如果希望 Agent 保留一些长期信息，最小可以怎么落地。**

很多人在做 Agent 时，都会想到 Memory。这个方向当然有价值，但一上来没必要把它想得太重。

如果你的目标只是做一个极简可用的记忆系统，最简单的办法不是上向量库，而是先落一个本地文件，把值得保留的信息存起来，并提供几个最基础的检索方法。

`simple-agent-skills-memory.py` 这一步就做得很克制：不搞向量数据库，不搞 embedding 检索，先用一个 `memory.md` 文件把记忆保存下来。

记忆写入函数是这样的：

```python
def save_memory(content: str, category: Optional[str] = None) -> str:
    init_memory()
    timestamp = datetime.now().strftime("%Y-%m-%d %H:%M:%S")

    with open(MEMORY_FILE, "a", encoding="utf-8") as f:
        f.write(f"---\n")
        f.write(f"**时间:** {timestamp}\n")
        if category:
            f.write(f"**分类:** {category}\n")
        f.write(f"**内容:**\n{content}\n\n")

    return f"已保存记忆到 {MEMORY_FILE}"
```

这个方案做的事情非常朴素：

- 每条记忆都有时间戳。
- 可以选填分类。
- 存成纯 Markdown，方便人工查看和手动编辑。

然后再配上几个最基本的检索工具：

- `read_memory(limit=10)`：读取最近 N 条。
- `search_memory(keyword)`：按关键词搜索。
- `search_memory_by_category(category)`：按分类检索。

例如关键词搜索：

```python
class MemorySearcher:
    @staticmethod
    def search_by_keyword(keyword: str) -> str:
        init_memory()

        results = []
        with open(MEMORY_FILE, "r", encoding="utf-8") as f:
            content = f.read()

        lines = content.split("\n")
        current_entry = []
        in_entry = False

        for line in lines:
            if line.startswith("---"):
                if in_entry and current_entry:
                    entry_text = "\n".join(current_entry)
                    if keyword.lower() in entry_text.lower():
                        results.append(entry_text)
                current_entry = []
                in_entry = True
            elif in_entry:
                current_entry.append(line)

        if results:
            return f"找到 {len(results)} 条匹配的记忆:\n\n" + "\n---\n".join(results)
        return f"未找到包含 '{keyword}' 的记忆"
```

如果你只是想实现一个“能用的极简 Memory”，这已经足够了。它当然不是最强方案，但有两个很现实的优点：

- 足够简单，今天就能跑起来。
- 行为可解释，不是一个黑盒检索系统。

很多时候，极简 Agent 的重点不是“最先进”，而是“最先可用”。

## 这套极简 Agent，为什么适合用来递进式实现？

因为它不是一上来就追求“什么都有”，而是把看起来很复杂的能力，拆成几个可以逐步叠加的小模块。

### 1. 先用最少工具把主链路跑通

读、写、列目录、跑命令，再加上后面的技能读取和记忆管理，已经能支撑一个相当像样的代码 Agent 了。

第一步不是追求工具多，而是先确保最关键的动作都能完成。

### 2. 再按问题逐层补能力，而不是一次堆满功能

不是急着加十几个工具，而是先保证：

- 工具描述清楚。
- 返回结果可控。
- 文件内容可定位。
- 想扩展能力时再引入 Skill。
- 想保留长期信息时再补 Memory。

### 3. 每一步演进都围绕一个具体诉求

这四个版本的升级路径不是“功能炫技”，而是你在继续往前做时，很自然会遇到的几个问题：

- `simple-agent.py`：解决“怎么让模型真正动手”。
- `simple-agent-v2.py`：如果想把单次执行变成连续会话，怎么做。
- `simple-agent-skills.py`：如果想按需扩展能力、又不想把提示词塞爆，怎么做。
- `simple-agent-skills-memory.py`：如果想让 Agent 记住一些长期信息，怎么做。

这种递进方式更容易让读者理解：Agent 的很多“高级能力”，其实都可以从一个很小的实现开始。

## 如果继续往前做，真正需要打磨的是上下文工程

很多人实现 Agent 时，注意力都放在模型选型上：是 GPT、Claude、还是别的模型；是大参数、还是长上下文。模型当然重要，但真落到工程上，决定 Agent 体验的，往往不是参数规模，而是上下文管理。

这也是为什么前面的每一步扩展，看起来都像是在补“小功能”，本质上却都在处理上下文问题。比如这套代码里，已经能看到几个很典型的策略：

### 1. 结果裁剪

过长的 shell 输出直接截断，防止上下文膨胀。

### 2. 结构化观察

`read_file` 自动加行号，把“原始文本”变成“可引用观察”。

### 3. 渐进式加载

Skill 只暴露索引，正文和资源由模型按需读取。

### 4. 长期记忆外置

把易丢失的信息沉淀到 `memory.md`，而不是指望模型“记住”。

如果你在这套极简实现上继续往前推，下一层通常会补这些能力：

- 上下文压缩：把旧消息总结成结构化摘要。
- 权限分级：读操作和写操作区分处理。
- 人工确认：高风险动作必须经过确认。
- 错误恢复：工具失败后，把错误回传给模型让它重试。
- 会话持久化：中断之后还能接着跑。

也就是说，前面那些看起来复杂的特性，很多本质上都只是上下文工程的不同形式。

## 极简不等于粗糙，最少也要注意这些问题

虽然这套实现已经能用，但如果你真要拿它继续往前做，有几个坑是绕不过去的。

### 1. `execute_bash` 很强，也很危险

只要给了模型任意 shell 能力，它理论上就能执行任何命令。玩具项目无所谓，真实环境里至少要考虑：

- 白名单命令。
- 沙箱执行。
- 超时和资源限制。
- 高风险命令确认。

否则 Agent 不是在帮你干活，是在帮你制造事故。

### 2. `write_file` 是覆盖写，不是增量编辑

这对最简原型足够，但对复杂项目不够友好。真实开发里更常见的需求其实是：

- 只改某几行。
- 用 patch 方式修改。
- 保留文件原格式。

如果一直全量重写，文件越大，出错概率越高。

### 3. Memory 目前还是“字符串检索”

基于 Markdown 和关键词的记忆系统非常直观，但在记忆量变大后，召回效果会逐步下降。这时才需要考虑 embedding、向量库或者分层记忆。

但重点是顺序别反了：**先有可解释的记忆，再追求高级检索。**

## 一条更容易上手的实现路径

如果你也想自己写一个 Agent，我会建议按下面这个顺序来，不要一上来就追求“全能智能体”。

1. 先做最小闭环。
2. 如果想让它连续工作，再补可持续交互。
3. 如果想扩展能力但不想把提示词堆爆，再加 Skills。
4. 如果想让它保留长期信息，再补一个极简 Memory。
5. 最后再做容错、确认、压缩这些工程化增强。

换句话说：

```text
先让它能跑
-> 再让它能连续跑
-> 再让它跑得久
-> 最后再让它跑得稳
```

这个顺序的好处是，读者能很直观地看到：每一个看起来复杂的功能，背后都可以有一个非常小、非常具体的起点。

比起一开始就堆满 Planner、Router、Reflection、Multi-Agent 协同之类的大词，这种实现路径更容易落地。

## 完整代码结构

这套示例代码目前大致是这样分层的：

```text
agent-learning/
├── simple-agent.py
├── simple-agent-v2.py
├── simple-agent-skills.py
├── simple-agent-skills-memory.py
├── skills_loader.py
├── memory_tools.py
├── memory.md
└── skills/
    └── web-search/
        └── SKILL.md
```

各自职责也比较清楚：

- `simple-agent.py`：最小可运行 Agent，只有基础工具和双层循环。
- `simple-agent-v2.py`：加入 `continue_interaction`，把会话真正串起来。
- `simple-agent-skills.py`：引入 Skill 索引和按需加载。
- `simple-agent-skills-memory.py`：把长期记忆工具并进 Agent。
- `skills_loader.py`：负责扫描技能目录、解析 `SKILL.md`、生成技能摘要。
- `memory_tools.py`：负责记忆初始化、写入、读取和检索。

这个结构的好处是：每一步升级都不是推倒重来，而是在前一个版本上补一层能力。所以整篇文章也更适合按“先做最小版，再逐步叠加”的方式来理解。

## 总结

如果把 Agent 神秘化，你会觉得它像一个“会自主思考的数字生命”；但如果回到代码层面看，它其实就是一个很朴素、而且可以逐步搭起来的工程系统：

- 用循环驱动多轮决策。
- 用工具把语言能力接到真实世界。
- 如果需要连续对话，就补一个续轮机制。
- 如果需要按需扩展能力，就补一个极简 Skill 系统。
- 如果需要保留长期信息，就补一个极简 Memory 系统。
- 再通过上下文工程控制它每一轮看到什么。

所以，实现一个极简 Agent 的关键，不是先追求“像人”，而是先把能力拆小，然后一步一步加上去。

先想清楚这些问题：

- 它能调用哪些工具？
- 每个工具的边界是什么？
- 工具结果如何回填？
- 如果想持续交互，入口放在哪里？
- 如果想引入 Skills，索引和正文怎么拆？
- 如果想保留记忆，先用什么最简单的存储方式？

把这些问题答清楚，一个能真正干活的 Agent，基本也就搭出来了。

说得再直白一点：

**Agent 不神秘，它只是一个被设计得足够好的循环系统。**
