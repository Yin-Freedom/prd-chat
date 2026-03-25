# 需求信息收集模型

本文档定义对话式需求收集过程中的数据结构。

---

## 整体数据结构

```typescript
interface RequirementData {
  product: ProductInfo;           // 产品背景
  roles: Role[];                  // 用户角色
  modules: Module[];              // 功能模块
  flows: BusinessFlow[];          // 业务流程
  nonFunctional: NonFunctional;   // 非功能需求
}
```

---

## 产品背景

```typescript
interface ProductInfo {
  name: string;           // 产品名称
  description: string;    // 产品简介
  targetUsers: string[];  // 目标用户群体
  coreValue: string;      // 核心价值
}
```

**示例：**
```json
{
  "name": "工单管理系统",
  "description": "企业内部工单提交和处理平台",
  "targetUsers": ["普通员工", "部门经理", "IT管理员"],
  "coreValue": "提高工单处理效率，实现流程可追溯"
}
```

---

## 用户角色

```typescript
interface Role {
  name: string;           // 角色名称
  type: RoleType;         // 角色类型
  responsibilities: string;  // 职责描述
  scenarios: string;      // 使用场景
  permissions: string;    // 权限特征
}

type RoleType = '系统角色' | '业务角色' | '外部角色';
```

**示例：**
```json
{
  "name": "部门经理",
  "type": "业务角色",
  "responsibilities": "审批工单、分配工单",
  "scenarios": "工单审批、工单分配",
  "permissions": "工单审批权限、工单分配权限"
}
```

---

## 功能模块

```typescript
interface Module {
  id: string;                 // 模块ID，如 M01
  name: string;               // 模块名称
  overview: ModuleOverview;   // 模块概述
  functions: Function[];      // 功能清单
  businessLogic: BusinessLogic[];  // 业务逻辑
  constraints: Constraint[];  // 约束条件
}

interface ModuleOverview {
  description: string;        // 模块描述
  purpose: string;            // 主要解决的问题
  targetRoles: string[];      // 使用角色
}

interface Function {
  id: string;                 // 功能ID，如 F01-01
  name: string;               // 功能名称
  type: FunctionType;         // 功能类型
  description: string;        // 功能描述
  operator: string;           // 操作角色
  priority: Priority;         // 优先级
  precondition: string;       // 前置条件
  inputs: InputField[];       // 输入字段
  outputs: string;            // 输出结果
  processFlow: string;        // 操作流程
}

interface InputField {
  name: string;               // 字段名称
  type: string;               // 字段类型
  required: boolean;          // 是否必填
  validation?: string;        // 校验规则
  defaultValue?: string;      // 默认值
  description?: string;       // 字段说明
}

interface BusinessLogic {
  id: string;                 // 规则ID
  name: string;               // 规则名称
  type: LogicType;            // 规则类型
  trigger: string;            // 触发条件
  logic: string;              // 处理逻辑
  affectedModules?: string[]; // 关联模块
}

type LogicType = '计算规则' | '状态转换' | '触发条件' | '数据联动' | '校验规则' | '其他';

interface Constraint {
  id: string;                 // 约束ID
  type: ConstraintType;       // 约束类型
  target: string;             // 约束对象
  description: string;        // 约束描述
  errorMessage: string;       // 错误提示
}

type ConstraintType = '唯一性' | '格式' | '范围' | '必填' | '关联' | '操作限制' | '其他';

type FunctionType = '新增' | '查询' | '修改' | '删除' | '导入' | '导出' | '审批' | '统计' | '其他';
type Priority = '高' | '中' | '低';
```

**示例：**
```json
{
  "id": "M01",
  "name": "用户管理",
  "overview": {
    "description": "管理系统用户账号，包括创建、维护和删除用户信息",
    "purpose": "实现用户账号的统一管理，确保系统用户信息准确、安全",
    "targetRoles": ["系统管理员"]
  },
  "functions": [
    {
      "id": "F01-01",
      "name": "新增用户",
      "type": "新增",
      "description": "创建新的用户账号",
      "operator": "系统管理员",
      "priority": "高",
      "precondition": "操作者具有管理员权限",
      "inputs": [
        { "name": "用户名", "type": "text", "required": true, "validation": "6-20位字母数字下划线" },
        { "name": "姓名", "type": "text", "required": true },
        { "name": "部门", "type": "select", "required": true },
        { "name": "角色", "type": "select", "required": true },
        { "name": "联系方式", "type": "text", "required": false, "validation": "11位手机号" }
      ],
      "outputs": "创建成功提示、用户ID",
      "processFlow": "填写用户信息 → 验证信息合法性 → 创建账号 → 返回结果"
    }
  ],
  "businessLogic": [
    {
      "id": "BL01-01",
      "name": "新用户默认状态",
      "type": "状态转换",
      "trigger": "用户创建时",
      "logic": "新用户默认状态为'启用'"
    },
    {
      "id": "BL01-02",
      "name": "权限实时生效",
      "type": "数据联动",
      "trigger": "用户角色变更时",
      "logic": "用户权限立即更新"
    }
  ],
  "constraints": [
    {
      "id": "C01-01",
      "type": "唯一性",
      "target": "用户名",
      "description": "系统内用户名不可重复",
      "errorMessage": "用户名已存在"
    },
    {
      "id": "C01-02",
      "type": "格式",
      "target": "用户名",
      "description": "6-20位字母数字下划线",
      "errorMessage": "用户名格式不正确"
    },
    {
      "id": "C01-03",
      "type": "关联",
      "target": "用户删除",
      "description": "删除用户前需先删除其工单记录",
      "errorMessage": "存在关联数据，无法删除"
    }
  ]
}
```

---

## 业务流程

```typescript
interface BusinessFlow {
  id: string;             // 流程ID
  name: string;           // 流程名称
  description: string;    // 流程描述
  participants: string[]; // 参与角色
  steps: FlowStep[];      // 流程步骤
  branches: Branch[];     // 分支情况
}

interface FlowStep {
  step: number;           // 步骤序号
  action: string;         // 操作名称
  role: string;           // 执行角色
  input: string;          // 输入
  output: string;         // 输出
  note?: string;          // 备注
}

interface Branch {
  condition: string;      // 分支条件
  handling: string;       // 处理方式
  nextStep: string;       // 后续步骤
}
```

**示例：**
```json
{
  "id": "F01",
  "name": "工单处理流程",
  "description": "工单从提交到关闭的完整流程",
  "participants": ["普通员工", "部门经理", "IT管理员"],
  "steps": [
    { "step": 1, "action": "提交工单", "role": "普通员工", "input": "工单信息", "output": "待审批工单" },
    { "step": 2, "action": "审批工单", "role": "部门经理", "input": "工单详情", "output": "已审批工单" },
    { "step": 3, "action": "处理工单", "role": "IT管理员", "input": "工单内容", "output": "已处理工单" },
    { "step": 4, "action": "关闭工单", "role": "IT管理员", "input": "处理结果", "output": "已关闭工单" }
  ],
  "branches": [
    { "condition": "审批拒绝", "handling": "退回修改", "nextStep": "步骤1" },
    { "condition": "处理中需要补充信息", "handling": "退回补充", "nextStep": "步骤3" }
  ]
}
```

---

## 非功能需求

```typescript
interface NonFunctional {
  performance: PerformanceRequirement[];   // 性能需求
  security: SecurityRequirement[];         // 安全需求
  compatibility: CompatibilityRequirement[]; // 兼容性需求
  availability?: AvailabilityRequirement;  // 可用性需求
}

interface PerformanceRequirement {
  id: string;
  item: string;           // 需求项
  metric: string;         // 指标要求
  note?: string;          // 说明
}

interface SecurityRequirement {
  id: string;
  item: string;           // 需求项
  description: string;    // 描述
}

interface CompatibilityRequirement {
  id: string;
  item: string;           // 需求项
  requirement: string;    // 要求
}

interface AvailabilityRequirement {
  uptime: string;         // 可用性指标
  backup: string;         // 备份策略
}
```

**示例：**
```json
{
  "performance": [
    { "id": "NFR01", "item": "页面响应时间", "metric": "< 3秒", "note": "普通页面" },
    { "id": "NFR02", "item": "并发用户数", "metric": "100", "note": "正常业务场景" }
  ],
  "security": [
    { "id": "NFR03", "item": "身份认证", "description": "用户名密码登录" },
    { "id": "NFR04", "item": "权限控制", "description": "基于角色的权限控制(RBAC)" },
    { "id": "NFR05", "item": "操作日志", "description": "记录所有关键操作日志" }
  ],
  "compatibility": [
    { "id": "NFR06", "item": "浏览器支持", "requirement": "Chrome、Firefox、Edge最新版本" },
    { "id": "NFR07", "item": "设备支持", "requirement": "PC端、平板" }
  ],
  "availability": {
    "uptime": "99.5%",
    "backup": "每日增量备份"
  }
}
```

---

## ID编号规则

| 类型 | 前缀 | 格式 | 示例 |
|------|------|------|------|
| 模块 | M | M{两位数字} | M01, M02 |
| 功能 | F | F{模块编号}-{功能编号} | F01-01, F01-02 |
| 业务规则 | R | R{模块编号}-{规则编号} | R01-01 |
| 业务流程 | FL | FL{两位数字} | FL01 |
| 非功能需求 | NFR | NFR{三位数字} | NFR001 |

---

## 状态管理

在对话过程中，维护一个状态对象：

```typescript
interface ConversationState {
  currentPhase: Phase;        // 当前阶段
  currentQuestion: number;    // 当前问题序号
  collectedData: RequirementData;  // 已收集数据
  history: HistoryItem[];     // 修改历史
}

type Phase = 'product' | 'roles' | 'modules' | 'flows' | 'nonFunctional' | 'output' | 'modify';

interface HistoryItem {
  timestamp: string;
  action: string;
  before?: any;
  after?: any;
}
```