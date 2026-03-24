# 快速参考卡

## 信息收集阶段

| 阶段 | 收集内容 | Prompt文件 |
|------|----------|------------|
| 1 | 产品背景 | `prompts/phases/phase1-product.md` |
| 2 | 用户角色 | `prompts/phases/phase2-roles.md` |
| 3 | 功能模块 | `prompts/phases/phase3-modules.md` |
| 4 | 业务流程 | `prompts/phases/phase4-flows.md` |
| 5 | 非功能需求 | `prompts/phases/phase5-nonfunctional.md` |

## 输出生成

| 输出物 | Prompt文件 |
|--------|------------|
| 需求文档 | `prompts/output/document-generation.md` |
| HTML原型 | `prompts/output/prototype-generation.md` |

## ID命名规则

| 类型 | 前缀 | 示例 |
|------|------|------|
| 模块 | M | M01 |
| 功能 | F | F01-01 |
| 业务逻辑 | BL | BL01-01 |
| 约束条件 | C | C01-01 |
| 业务流程 | FL | FL01 |
| 非功能需求 | NFR | NFR01 |

## 功能类型

新增、查询、修改、删除、导入、导出、审批、统计、其他

## 规则类型

计算规则、状态转换、触发条件、数据联动、校验规则、其他

## 约束类型

唯一性、格式、范围、必填、关联、操作限制、其他

## 优先级

高（必须有）、中（建议有）、低（可延后）