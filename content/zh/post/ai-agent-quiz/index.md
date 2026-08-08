---
date: '2026-06-12T15:41:22+08:00'
lastmod: '2026-06-12T15:41:22+08:00'
draft: true
title: AI Agent 测试题
slug: ai-agent-quiz
description: 根据您的提问，我总结您的需求如下：
tags:
- prompt
- 可测试性
categories:
- AI
comment: false
toc: false
---

根据您的提问，我总结您的需求如下：

## 核心需求
**从 我提供的页面提取所有模型的完整信息，并输出为结构化的 JSON 格式**

## 具体要求

1. **数据结构**：按照指定的格式输出，包含以下字段：
   - `provider`: 供应商（ppio）
   - `model`: 模型名称
   - `input_price_per_million`: 输入价格（每百万token）
   - `output_price_per_million`: 输出价格（每百万token）
   - `currency`: 货币单位（CNY）
   - `source_url`: 来源网址
   - `retrieved_at`: 获取时间
   - `raw_unit`: 原始单位
   - `raw_text`: 原始文本信息

2. **完整性要求**：
   - 必须找到**所有**大语言模型
   - 不能只抓取当前可见的部分，需要完整加载所有数据
   - 需要点击"所有模型"按钮，切换到完整列表视图

3. **数据准确性**：
   - 提取真实的价格信息（输入价格、输出价格、缓存价格等）
   - 包含上下文长度信息
   - 保留特殊标签（如"满血版"、"全参数"、"推理模型"等）
把价格统一规范到“每百万 token”，并区分输入/输出价格
结果需要后处理/分析，不能直接用原始抓取行：必须避免把价格行当成 model，并在模型识别不可靠时进行过滤或修正。
以下是我提供的 url：
https://www.volcengine.com/docs/82379/1544106?lang=zh
https://www.qiniu.com/ai/models
https://www.siliconflow.cn/pricing
https://ppio.com/ai-computing/llm-api
