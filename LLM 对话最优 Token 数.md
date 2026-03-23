# LLM 对话最优 Token 数

## 核心问题

使用 LLM 时，一次对话放多少 token 才最合适？更长的上下文真的意味着更好的效果吗？

答案是：**不是越长越好**。研究表明，上下文过长会导致模型性能显著下降。

---

## 关键研究：Lost in the Middle

### 论文背景

《Lost in the Middle: How Language Models Use Long Contexts》（Liu et al., 2023，发表于 *Transactions of the Association for Computational Linguistics*）是目前关于 LLM 长上下文利用的最重要研究之一。

论文通过多文档问答（Multi-Document QA）和键值检索（Key-Value Retrieval）两类任务，系统评估了 LLM 在长上下文中的信息利用能力。

### 核心发现：U 型性能曲线

**关键信息的位置直接决定模型表现。**

实验结果显示，LLM 的回答准确率与相关信息在上下文中的位置呈 **U 型曲线**关系：

| 信息位置   | 模型表现                         |
| ---------- | -------------------------------- |
| 上下文开头 | ✅ 最优（首位偏好，Primacy Bias） |
| 上下文结尾 | ✅ 较优（近因偏好，Recency Bias） |
| 上下文中间 | ❌ 最差，性能显著下降             |

当需要检索的关键文档被放在 20 个文档列表的中间时，准确率相比放在首位或末位**下降 15–20 个百分点**。

### 另一个重要发现：长度越长，问题越大

即使是专门为长上下文设计的模型，在上下文增长时也会出现性能下降：

- 随着输入文档数从 1 增加到 30，模型准确率持续下滑
- 扩大上下文窗口大小并不能解决"中间遗忘"问题
- 模型在形式上"能处理"长上下文，但实际上无法有效利用中间部分

---

## 相关研究

### Context Rot（上下文腐烂）

Chroma Research（2025）对 18 个主流 LLM 进行系统测试，提出了"Context Rot"（上下文腐烂）概念：

> 随着输入 token 数量增加，LLM 的准确率、推理能力和指令遵循能力会出现可观测的持续下降。

**核心结论：**
- 上下文越长，模型越难区分重要信号与噪声
- 即使是最新的旗舰模型（GPT-4.1、Gemini 2.5 等），在实际任务中也存在这一问题
- 简单的检索基准（如 Needle-in-a-Haystack）往往高估了模型处理长上下文的实际能力

### Needle-in-a-Haystack 测试的局限

"大海捞针"测试是测量 LLM 长上下文能力的常用基准，但它只测试**简单词汇匹配**。

真实任务需要模型在长上下文中进行**多步推理和语义理解**，这比简单检索难得多——这也是为什么模型在基准测试上表现良好，但在实际使用中仍感觉"忘事"。

### 上下文窗口的快速扩张

自 2023 年以来，主流 LLM 的最大上下文窗口迅速增长：

| 时期    | 主流窗口大小   | 代表模型                      |
| ------- | -------------- | ----------------------------- |
| 2022 年 | 4K tokens      | GPT-3                         |
| 2023 年 | 16K–32K tokens | GPT-3.5 Turbo, GPT-4          |
| 2024 年 | 128K tokens    | GPT-4 Turbo, Claude 3, Gemini |
| 2025 年 | 200K–1M tokens | Claude 4, Gemini 2.5          |

窗口越来越大，但**有效利用率**并没有同步提升。

---

## 最优 Token 数的实践建议

### 经验法则

| 场景          | 建议 Token 范围  | 说明                       |
| ------------- | ---------------- | -------------------------- |
| 普通对话/问答 | < 8K tokens      | 信息清晰，模型表现最稳定   |
| 代码审查/分析 | 8K–32K tokens    | 聚焦当前文件，避免无关代码 |
| 文档处理      | 32K–64K tokens   | 确保关键内容在开头或结尾   |
| 长文档分析    | < 模型上限的 70% | 保留缓冲，避免接近极限     |

**实用原则：将模型最大上下文的 70%–75% 作为实际使用上限。**

以 Claude 3 Sonnet/Opus（200K tokens 窗口）为例：
- 理论上限：200K tokens
- 系统提示 + 工具定义等占用：约 30K–40K tokens
- 实际可用：约 147K tokens
- **推荐使用：< 100K tokens**

### 关键信息要放在开头或结尾

根据"Lost in the Middle"论文的发现：

```
❌ 不好的做法：
[大量背景信息] → [关键需求] → [更多背景信息]

✅ 好的做法：
[关键需求] → [必要背景信息]
或：
[必要背景信息] → [关键需求]
```

### 优化上下文的具体方法

**1. 只放相关内容**
- 不要将整个代码库或文档都塞进上下文
- 只包含当前任务真正需要的文件或段落

**2. 定期压缩/总结**
- 对话进行一段时间后，将历史对话总结成摘要
- ClaudeCode 的 `/compact` 命令可以自动压缩上下文

**3. 使用子代理隔离**
- 将不同的子任务拆分给独立的子代理
- 每个子代理有独立的上下文窗口，避免相互干扰

**4. 避免重复内容**
- 不要多次粘贴相同的背景信息
- 使用 `CLAUDE.md` 等机制固化通用规则，而不是每次重复

**5. 任务拆解**
- 将大任务拆分为多个小任务
- 每个任务保持上下文简洁，完成后再进行下一步

---

## 与 ClaudeCode 的关系

结合 ClaudeCode 的使用场景：

1. **任务越小效果越好**（文档已提及）：这与研究结论一致，小任务意味着更短的上下文，性能更稳定。

2. **CLAUDE.md 要保持简洁**（文档已提及）：过长的 CLAUDE.md 会占用有效上下文，降低可用 token 数。

3. **子代理的价值**：子代理在独立的上下文窗口中运行，天然隔离了上下文污染，这正是应对 Context Rot 的架构级解决方案。

4. **Plan 模式**：先制定计划，分步执行，每步聚焦单一任务，避免一次对话承载过多信息。

---

## 总结

| 关键结论             | 说明                                       |
| -------------------- | ------------------------------------------ |
| 信息位置比长度更重要 | 关键内容放开头或结尾，避免放中间           |
| 70% 法则             | 使用模型上下文上限的 70% 以内为安全区      |
| 越长越慢越贵越差     | 更长上下文带来更高延迟、成本，和更低准确率 |
| 主动管理上下文       | 定期压缩、只保留相关内容、善用子代理       |
| 任务拆小             | 比扩大上下文更有效                         |

**结论：对于日常对话和代码开发任务，将单次对话控制在 8K–32K tokens 以内，可以获得最稳定的模型表现。需要处理长文档时，确保关键信息在开头或结尾，并主动管理上下文大小。**

---

## 参考资料

- [Lost in the Middle: How Language Models Use Long Contexts](https://direct.mit.edu/tacl/article/doi/10.1162/tacl_a_00638/119630/Lost-in-the-Middle-How-Language-Models-Use-Long) - Liu et al., TACL 2023
- [Context Rot: How Increasing Input Tokens Impacts LLM Performance](https://research.trychroma.com/context-rot) - Chroma Research, 2025
- [Context rot explained (& how to prevent it)](https://redis.io/blog/context-rot/) - Redis, 2025
- [Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) - Anthropic, 2025
- [LLMs now accept longer inputs, and the best models can use them more effectively](https://epoch.ai/data-insights/context-windows) - Epoch AI, 2024
- [Why larger LLM context windows are all the rage](https://research.ibm.com/blog/larger-context-window) - IBM Research, 2024