---
date: '2026-03-09T14:30:00+08:00'
lastmod: '2026-03-09T14:30:00+08:00'
draft: false
title: Agent Tool Call：从“对话”到“执行”的工程实践
slug: agent-tool-call-engineering
description: 在使用大模型的过程中，你肯定发现目前的大语言模型（LLM）在逻辑推理、代码编写和文本生成方面表现优异。但是，它们都有一个共同的局限：无法直接干预外部世界。
tags:
- AI
- Agent
- 工程实践
categories:
- AI
comment: false
toc: false
---

在使用大模型的过程中，你肯定发现目前的大语言模型（LLM）在逻辑推理、代码编写和文本生成方面表现优异。但是，它们都有一个共同的局限：无法直接干预外部世界。
如果你要求模型“查询订单状态”或“发送一封邮件”，它通常会回复：“对不起，我无法访问您的数据库。”这是因为模型本质上只是一个预测下一个 Token 的概率模型，并没有直接访问系统资源的权限。

**Tool Call**（工具调用，也称 Function Calling）的出现，正是为了给模型安装上“手脚”，让它能够通过结构化的方式与外部系统交互。

***

## ## 一、 核心概念：决策与执行分离
![截屏2026-03-09 22.39.50](截屏2026-03-09 22.39.50.png)
理解 Tool Call 的关键在于：**模型本身并不执行代码，它只负责决策。**

我们可以将其理解为“指挥官”与“执行官”的关系：
*   **模型（决策者）**：负责判断“当前需要调用哪个工具”、“需要传入什么参数”。
*   **程序（执行者）**：负责运行真实的后端逻辑，如数据库查询、发送邮件、鉴权、限流等。

这种“决策与执行分离”的架构，确保了系统的安全性。模型产生的只是一个 **JSON 格式的调用指令**，真正的执行权限始终掌握在开发者手中，而不是交给模型。


***

## 二、 通用工作流

无论是查询数据（读操作）还是执行动作（写操作），在工程上通常遵循以下五个步骤：

1.  **定义工具**：开发者向模型描述可选工具的功能（建议使用动词命名，如 `get_order_status`）及其参数规格（使用 JSON Schema）。
2.  **发送上下文**：将用户问题与工具清单一并发送给模型。
3.  **模型决策**：模型判断当前问题是否需要工具。如果需要，它会返回一个结构化的响应，包含工具名、参数及唯一标识符 `call_id`。
4.  **本地执行**：程序解析参数，调用真实的 API 或数据库，并获取结果。
5.  **生成回答**：程序将执行结果回传给模型，模型结合结果生成最终的自然语言回复。

***

## 三、 Tool Call Schema 详解

Schema 是模型与程序之间的“契约”。定义得越严谨，模型调用的准确率就越高。


### 1. 结构示例

```json
{
  "type": "function",
  "function": {
    "name": "get_order_status",
    "description": "根据订单号查询订单状态与物流信息",
    "parameters": {
      "type": "object",
      "properties": {
        "order_id": {
          "type": "string",
          "description": "订单号，例如 1001"
        }
      },
      "required": ["order_id"],
      "additionalProperties": false
    },
    "strict": true
  }
}
```

### 2. 字段含义与最佳实践

| 字段 | 说明 | 最佳实践 |
|------|------|----------|
| `name` | 工具的唯一标识符 | 使用动词式命名，如 `get_order_status` |
| `description` | 工具的用途说明 | **越详细越好**：说明适用场景、限制条件及返回格式 |
| `parameters` | 参数定义 | 每个参数都应包含 `type` 和 `description` |
| `required` | 必填参数名数组 | 明确模型必须提供的字段 |
| `additionalProperties` | 额外参数 | 设为 `false`，防止模型生成多余参数 |
| `strict` | 严格模式 | 设为 `true`，强制模型输出符合 Schema 的 JSON |

### 3. 一个“好”的描述长什么样

**差的描述：**
```json
{ "name": "send_email", "description": "发送邮件" }
```

**好的描述：**
```json
{
  "name": "send_email",
  "description": "通过企业 SMTP 服务发送邮件。调用前须获得用户明确授权。收件人必须符合公司域名白名单。返回状态：'sent' 表示成功，'failed' 表示失败。"
}
```

**一句话总结**：好的描述包含了边界条件和前置要求，是模型进行“决策”时不可或缺的信息。

### 常见平台差异速查

| 平台 | 特殊注意 |
|------|----------|
| **OpenAI** | 支持 `strict: true`，推荐开启 |
| **Claude** | 使用 `tool_choice` 参数控制工具选择行为，描述质量对效果影响极大 |
| **LangChain** | 封装层多，底层还是 OpenAI/Claude 格式，注意 `bind_tools()` 的参数传递 |
| **自研网关** | 统一 schema 校验，建议在设计阶段就定义严格的 JSON Schema |

***

## 四、 示例一：数据查询（读操作）

### 场景
用户：帮我查一下订单 1001 的最新状态。

### 工具定义
`strict: true` 是个好习惯，它让模型输出更稳定地符合 schema 结构，减少「参数乱写」。

```json
{
  "type": "function",
  "name": "get_order_status",
  "description": "根据订单号查询订单状态与物流信息",
  "parameters": {
    "type": "object",
    "properties": {
      "order_id": { "type": "string", "description": "订单号，例如 1001" }
    },
    "required": ["order_id"],
    "additionalProperties": false
  },
  "strict": true
}
```

### 完整一轮交互

**模型返回（不是最终回答，是 tool call）：**
```json
{
  "type": "function_call",
  "call_id": "call_abc",
  "name": "get_order_status",
  "arguments": "{\"order_id\":\"1001\"}"
}
```

**你的后端执行，返回真实数据：**
```json
{
  "order_id": "1001",
  "status": "Shipped",
  "carrier": "DHL",
  "tracking_no": "DHL123456",
  "last_update": "2026-03-04 10:12"
}
```

**你把工具结果回传（注意要带上 call_id）：**
```json
{
  "type": "function_call_output",
  "call_id": "call_abc",
  "output": "{\"order_id\":\"1001\",\"status\":\"Shipped\",\"carrier\":\"DHL\",\"tracking_no\":\"DHL123456\"}"
}
```

**模型最终回答：** 「您的订单 1001 已发货，承运商 DHL，运单号 DHL123456，昨天上午更新。需要我帮您追踪实时轨迹吗？」

**要点：** 工具结果必须来自你的系统，模型不能「假装已经查到了」——这是你在 code review 里要严格把关的边界。

***

## 五、 示例二：执行动作（写操作）

### 场景
用户：帮我给 zhangsan@example.com 发邮件，说明明天 10 点开会。

### 工具定义

```json
{
  "type": "function",
  "name": "send_email",
  "description": "发送邮件，需对接企业邮箱或第三方邮件服务",
  "parameters": {
    "type": "object",
    "properties": {
      "to": { "type": "string", "description": "收件人邮箱" },
      "subject": { "type": "string", "description": "邮件标题" },
      "body": { "type": "string", "description": "邮件正文" }
    },
    "required": ["to", "subject", "body"],
    "additionalProperties": false
  },
  "strict": true
}
```

对于涉及状态变更的操作（如发邮件、退款、创建工单），工程上必须遵循 **Human-in-the-loop（人工确认）** 原则：

1.  模型生成工具调用指令。
2.  **程序不直接执行**，而是将“准备执行的内容”（如收件人、邮件正文）展示给用户确认。
3.  用户确认无误后，程序再触发真实的 API 调用，并回传结果。

这能有效防止由于模型幻觉导致的误操作，也能抵御针对 AI 的指令注入攻击（Prompt Injection）。

***

## 六、 进阶：用 Tool Call 实现 Skills 渐进式加载

当你的 AI Agent 开始复杂起来，你会遇到一个新问题：技能（Skills）很多，每个技能又有详细说明、话术模板、规则文档……如果全塞进初始上下文，很快就会撑爆 token 限制，而且大量无关内容反而会干扰模型判断。

解法是 **Progressive Disclosure（渐进式披露）**，其思路和 Tool Call 如出一辙：

*   **第一层**：在系统提示词中仅列出技能目录（名称和一句话简介）。
*   **第二层**：暴露一个 `load_skill(name)` 工具。当模型判断需要某个技能时，调用该工具读取完整的 `SKILL.md` 文档。
*   **第三层**：按需读取具体的资源文件（如话术模板或审批规则）。

这种按需读取的机制，是构建大规模技能体系的必要条件。


***

## 七、 什么时候需要 Tool Call？


判断标准非简单，可以参考以下标准：

*   **需要实时或私有数据**：如订单、库存、用户信息，模型的训练数据永远是过期的。
*   **需要触发外部动作**：如修改数据库、发送通知、启动工作流。
*   **需要工程化落地的场景**：需要进行鉴权、审计、限流或 A/B 测试时，结构化的 Tool Call 是唯一的切入点。

***

## 八、最后一页：一个简单的 checklist

在实现第一个 Tool Call 集成时，请核对以下几点：

- [ ] **命名规范**：工具名称是否使用了清晰的动词？（如 `create_ticket`）
- [ ] **参数锁死**：是否开启了 `strict: true` 并禁用了额外属性？
- [ ] **安全确认**：所有“写操作”是否都包含了人工确认环节？
- [ ] **数量控制**：单次交互的工具清单建议控制在 10-15 个以内。
- [ ] **标识匹配**：回传结果时，是否始终带上了一一对应的 `call_id`？

最重要的是： 让 AI 帮你检查一下！😂
