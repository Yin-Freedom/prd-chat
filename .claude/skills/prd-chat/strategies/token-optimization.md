# 需求分析Skill优化策略

## 一、Token优化背景

### 核心问题

根据《Lost in the Middle: How Language Models Use Long Contexts》研究，LLM存在以下特性：

1. **U型性能曲线**：关键信息放开头或结尾时模型表现最好，放中间时性能下降15-20%
2. **Context Rot（上下文腐烂）**：随着输入token增加，模型准确率、推理能力持续下降
3. **70%法则**：建议使用模型上下文上限的70%以内作为安全区

### 优化后Skill Token分析

| 文件 | 字符数 | 估算Token | 加载时机 |
|------|--------|-----------|----------|
| SKILL.md | 9,938 | ~4,000 | 始终加载 |
| prompts/phases/phase1-product.md | 1,308 | ~520 | 阶段1 |
| prompts/phases/phase2-roles.md | 1,512 | ~600 | 阶段2 |
| prompts/phases/phase3-modules.md | 3,084 | ~1,200 | 阶段3 |
| prompts/phases/phase4-flows.md | 1,973 | ~780 | 阶段4 |
| prompts/phases/phase5-nonfunctional.md | 2,236 | ~890 | 阶段5 |
| prompts/output/document-generation.md | 12,655 | ~5,000 | 生成文档 |
| prompts/output/prototype-generation.md | 14,721 | ~5,800 | 生成原型 |
| references/quick-reference.md | 1,201 | ~480 | 启动时 |
| references/information-model.md | 10,181 | ~4,000 | 需要时 |
| references/document-template.md | 6,636 | ~2,600 | 生成文档 |
| examples/example-conversation.md | 24,099 | ~9,600 | 仅参考 |

**优化效果**：
- 单阶段最大Token: ~6,000（阶段3） vs 优化前 ~36,000
- 减少约 **80%** 的单次加载量

---

## 二、分块策略

### 2.1 按需加载原则

**核心思路**：不一次性加载所有文件，根据当前阶段按需读取。

```
启动 → 加载: SKILL.md + quick-reference.md (~4,500 tokens)
阶段1(产品背景) → 加载: phase1-product.md (~520 tokens)
阶段2(用户角色) → 加载: phase2-roles.md (~600 tokens)
阶段3(功能模块) → 加载: phase3-modules.md (~1,200 tokens)
阶段4(业务流程) → 加载: phase4-flows.md (~780 tokens)
阶段5(非功能需求) → 加载: phase5-nonfunctional.md (~890 tokens)
生成文档 → 加载: document-generation.md (~5,000 tokens)
生成原型 → 加载: prototype-generation.md (~5,800 tokens)
```

### 2.2 Prompt文件分块（已实现）

文件已按阶段拆分为独立模块：

```
prompts/
├── phases/
│   ├── phase1-product.md        # ~520 tokens
│   ├── phase2-roles.md          # ~600 tokens
│   ├── phase3-modules.md        # ~1,200 tokens
│   ├── phase4-flows.md          # ~780 tokens
│   └── phase5-nonfunctional.md  # ~890 tokens
└── output/
    ├── document-generation.md   # ~5,000 tokens
    └── prototype-generation.md  # ~5,800 tokens
```

### 2.3 参考文档精简

将冗长的参考文档精简为核心要点：

```
references/
├── information-model.md    # 数据结构定义 (~4,000 tokens)
├── document-template.md    # 文档模板 (~2,600 tokens)
└── quick-reference.md      # 快速参考卡（新增，~500 tokens）
```

---

## 三、摘要策略

### 3.1 对话历史压缩

在长对话中，定期压缩历史信息：

```
原始对话（多轮问答）:
用户: 这个系统包含哪些模块？
Claude: 识别到3个模块：用户管理、订单管理、报表统计
用户: 用户管理有哪些功能？
Claude: 新增用户、查询用户、修改用户、删除用户
...

压缩后摘要:
[已收集信息摘要]
- 模块：用户管理、订单管理、报表统计
- 用户管理功能：新增、查询、修改、删除
- 当前阶段：功能模块收集
- 待确认：订单管理模块详情
```

### 3.2 阶段性摘要

每个阶段完成后生成结构化摘要：

```markdown
## 阶段X完成摘要

### 已收集数据
{JSON格式的结构化数据}

### 当前状态
- 阶段: X
- 进度: X/5
- 下一步: {下一阶段名称}

### 关键决策
- 决策1: {描述}
- 决策2: {描述}
```

### 3.3 信息位置优化

根据"U型曲线"原则，在上下文中：

```
[开头]
→ 当前阶段任务描述
→ 核心问题/目标
→ 已确认的关键决策

[中间]
→ 详细的历史信息（可压缩）
→ 参考文档内容

[结尾]
→ 当前需要用户确认的内容
→ 下一步行动项
```

---

## 四、Token控制策略

### 4.1 各阶段Token预算

| 阶段 | Prompt Token | 对话Token | 总预算 |
|------|--------------|-----------|--------|
| 产品背景 | ~1,500 | ~2,000 | ~4,000 |
| 用户角色 | ~1,500 | ~3,000 | ~5,000 |
| 功能模块 | ~3,000 | ~8,000 | ~12,000 |
| 业务流程 | ~1,500 | ~3,000 | ~5,000 |
| 非功能需求 | ~1,000 | ~2,000 | ~4,000 |
| 文档生成 | ~5,000 | ~3,000 | ~8,000 |
| 原型生成 | ~6,000 | ~5,000 | ~12,000 |

### 4.2 触发压缩的条件

当满足以下任一条件时，执行对话压缩：

1. **Token超限**：当前上下文超过预算的80%
2. **阶段完成**：一个阶段确认完成后
3. **用户请求**：用户明确要求回顾或总结
4. **修改历史**：用户进行了多次修改请求

### 4.3 压缩策略

```python
def compress_context(conversation, collected_data):
    """
    压缩上下文策略
    """
    # 1. 保留关键结构
    essential = [
        collected_data,  # 已收集的结构化数据
        current_phase,   # 当前阶段
        pending_items    # 待处理项
    ]

    # 2. 压缩历史对话
    history_summary = summarize_conversation(conversation)

    # 3. 丢弃详细过程
    discard = [
        detailed_prompts,      # 已使用的详细prompt
        redundant_confirmations # 重复的确认对话
    ]

    return essential + history_summary
```

---

## 五、实现建议

### 5.1 SKILL.md优化

**当前问题**：SKILL.md包含所有阶段的详细流程，占用较多token。

**优化方案**：
1. SKILL.md只保留框架和启动流程
2. 各阶段详细prompt移到独立文件
3. 通过"按需读取"机制加载

### 5.2 引导问题精简

**当前问题**：每个阶段有多个引导问题，部分可精简。

**优化方案**：
- 合并相似问题
- 提供示例减少解释空间
- 关键问题放开头

### 5.3 示例文件处理

**当前问题**：example-conversation.md占用~9,600 tokens。

**优化方案**：
1. 精简为最小可用示例
2. 或移到单独目录，不作为默认加载项
3. 仅在用户需要参考时读取

---

## 六、优化后的文件结构

```
product-requirement-v2/
├── SKILL.md                      # 主文件（精简版，~3,000 tokens）
│
├── prompts/
│   ├── phases/                   # 分阶段prompt
│   │   ├── phase1-product.md     # ~1,500 tokens
│   │   ├── phase2-roles.md       # ~1,500 tokens
│   │   ├── phase3-modules.md     # ~3,000 tokens
│   │   ├── phase4-flows.md       # ~1,500 tokens
│   │   └── phase5-nonfunctional.md # ~1,000 tokens
│   └── output/
│       ├── document-generation.md # ~5,000 tokens
│       └── prototype-generation.md # ~5,800 tokens
│
├── references/
│   ├── information-model.md      # ~4,000 tokens
│   ├── document-template.md      # ~2,600 tokens
│   └── quick-reference.md        # ~500 tokens（新增）
│
├── examples/
│   └── example-conversation.md   # 仅按需加载
│
└── strategies/
    └── token-optimization.md     # 本文档
```

---

## 七、监控与迭代

### 7.1 Token监控指标

| 指标 | 目标值 | 说明 |
|------|--------|------|
| 单阶段最大Token | < 15,000 | 避免Context Rot |
| Prompt占比 | < 40% | 留足对话空间 |
| 压缩频率 | 每阶段1次 | 阶段完成后压缩 |
| 总Token使用 | < 100,000 | 完整流程的预算 |

### 7.2 迭代优化方向

1. **Prompt压缩**：使用更简洁的语言，删除冗余解释
2. **智能加载**：基于用户输入判断需要加载哪些参考文档
3. **增量更新**：修改时只更新受影响的部分，不重新加载全部
4. **缓存机制**：缓存已生成的摘要，避免重复处理

---

## 参考资料

1. [Lost in the Middle: How Language Models Use Long Contexts](https://direct.mit.edu/tacl/article/doi/10.1162/tacl_a_00638/119630/Lost-in-the-Middle-How-Language-Models-Use-Long) - Liu et al., TACL 2023
2. [Context Rot: How Increasing Input Tokens Impacts LLM Performance](https://research.trychroma.com/context-rot) - Chroma Research, 2025
3. [Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) - Anthropic, 2025