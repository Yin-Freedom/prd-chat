# 对话式需求文档生成Skill实现计划

## Context

用户希望创建一个新的skill（product-requirement-v2），用于通过对话式引导问答生成产品需求文档和HTML原型。该skill的特点：

- **对话模式**：引导式问答，通过结构化问题逐步收集产品信息
- **可视化修改**：生成HTML原型后，用户通过对话反馈修改
- **与v1关系**：独立skill，但复用v1的部分资源（如prototype-style-guide.md）
- **输出物**：需求文档 + HTML原型

---

## 目录结构

```
D:\program\llm\prd-skill\product-requirement-v2\
├── SKILL.md                      # 主文件（frontmatter + 工作流程）
├── prompts/                      # Prompt模板
│   ├── question-flow.md          # 引导式问答流程
│   ├── document-generation.md    # 需求文档生成
│   └── prototype-generation.md   # HTML原型生成
├── references/                   # 参考文档
│   ├── information-model.md      # 信息收集模型
│   └── document-template.md      # 需求文档模板
└── examples/                     # 示例
    └── example-conversation.md   # 完整对话示例
```

---

## 实现步骤

### 步骤1: 创建SKILL.md主文件

**路径**: `D:\program\llm\prd-skill\product-requirement-v2\SKILL.md`

**内容要点**:
- Frontmatter定义name、description和触发词
- 启动流程：欢迎语 + 第一个问题
- 5个信息收集阶段的定义
- 输出生成流程
- 对话式修改机制

### 步骤2: 创建引导式问答Prompt

**路径**: `prompts/question-flow.md`

**内容要点**:
- 角色设定：资深产品经理引导用户
- 5个阶段的引导问题设计：
  1. 产品背景（名称、简介、用户、价值）
  2. 用户角色（角色名、职责、场景、权限）
  3. 功能模块（模块概述、功能清单、功能详情、业务逻辑、约束条件）
  4. 业务流程（流程步骤、参与角色、分支条件）
  5. 非功能需求（性能、安全、兼容性）
- 每阶段确认机制
- 中途修改处理逻辑

### 步骤3: 创建需求文档生成Prompt

**路径**: `prompts/document-generation.md`

**内容要点**:
- 输入：结构化的需求信息
- 输出格式：标准PRD文档结构
- 包含章节：产品概述、用户角色、功能模块、业务流程、非功能需求

### 步骤4: 创建HTML原型生成Prompt

**路径**: `prompts/prototype-generation.md`

**内容要点**:
- 页面规划策略：功能模块 → 页面映射
- 组件映射：需求元素 → HTML组件
- 技术栈：Tailwind CSS + 原生JS
- 设计规范：引用v1的prototype-style-guide.md
- 交互实现：表单、弹窗、导航

### 步骤5: 创建参考文档

**信息收集模型** (`references/information-model.md`):
- 定义需求信息的数据结构
- 各字段的数据类型和约束

**需求文档模板** (`references/document-template.md`):
- Markdown格式的PRD模板
- 各章节的标准格式

### 步骤6: 创建示例文件

**路径**: `examples/example-conversation.md`

展示完整的对话流程示例，包括：
- 引导问答过程
- 信息确认和修改
- 输出生成
- 对话式迭代修改

---

## 关键设计

### 引导式问答流程

```
启动 → 产品背景 → 确认 → 用户角色 → 确认 → 功能模块 → 确认 → 业务流程 → 确认 → 非功能需求 → 确认 → 选择输出 → 生成 → 迭代修改
```

### 中途修改机制

用户可在任意阶段请求修改之前内容：
1. 识别修改目标
2. 更新对应数据
3. 展示修改结果确认
4. 继续当前阶段

### 对话式原型修改

用户可通过对话描述修改需求，skill重新生成原型：
- "用户列表增加筛选功能"
- "原型改成深色主题"
- "表单增加手机号字段"

---

## 复用资源

从v1复用以下资源：
- `product-requirement-v1/references/prototype-style-guide.md` - 原型设计规范
- 参考 `product-requirement-v1/prompts/*.md` 的prompt结构和角色设定模式

---

## 输出文件命名

| 输出物 | 文件名 |
|--------|--------|
| 需求文档 | `requirement-doc.md` |
| HTML原型 | `prototype.html` |

---

## Verification

1. 创建所有文件后，测试skill触发是否正常
2. 模拟完整对话流程，验证：
   - 各阶段问答是否流畅
   - 信息收集是否完整
   - 确认机制是否有效
3. 验证输出生成：
   - 需求文档格式是否规范
   - HTML原型是否可交互
4. 测试修改机制：
   - 中途修改是否正确更新
   - 原型修改是否正确反映需求变化