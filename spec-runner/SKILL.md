---
name: spec-runner
description: OpenSpec 执行引擎。自动执行 Change Proposals：创建提案→设计拆解→3道关卡→评审→Apply→测试→归档。支持单 CP/批量执行，自动决策，完整审计。
---

# Spec-Runner 开发框架

Spec-Runner 是 **OpenSpec 的执行引擎**，将 Change Proposals（变更提案）按照规范自动执行完整开发流程。

## 快速开始

**最小示例**：

```bash
# 用户输入（任一形式）
"用 spec-runner 帮我实现用户认证功能"
"执行 cps.md 中的开发计划"
"创建一个 add-user-auth 的提案并完整实现"

# spec-runner 自动执行
步骤 1：创建提案 → 步骤 2：设计拆解 → 步骤 3：3 道关卡
→ 步骤 4：评审提案 → 步骤 5：Apply → 步骤 6：测试
→ 步骤 7：归档 → 步骤 8：Git提交

# 输出
- changes/[change-id]/proposal.md
- changes/[change-id]/tasks.md
- changes/[change-id]/AUDIT.md
- changes/[change-id]/DECISIONS.md
- Git commit（记录在 AUDIT.md）
- 完成报告
```

## 核心定位

```
┌─────────────────────────────────────────────────────────────┐
│                     OpenSpec 生态                            │
├─────────────────────────────────────────────────────────────┤
│  OpenSpec (规范层)   →  定义 CP 结构、delta spec 格式        │
│  Spec-Runner (执行层) →  自动执行、自动决策、审计追踪         │
└─────────────────────────────────────────────────────────────┘
```

## 触发场景

**用户可能使用的触发词**：
- "用 spec-runner 执行..."
- "批量执行 cps.md"
- "创建并实现一个 proposal"
- "帮我从提案到交付"
- "spec-runner：xxx 功能"

**自动识别场景**：
- 输入包含多个功能描述 → 自动进入多 CP 批量执行模式
- 输入是单个功能/需求 → 自动进入单 CP 执行模式
- 输入是 cps.md 文件 → 自动解析并批量执行

## 前置条件

1. **OpenSpec 已初始化**：项目目录下存在 `openspec/` 目录
2. **OpenSpec CLI 可用**：`openspec` 命令可用
3. **项目规范存在**：`openspec/project.md` 文件存在

## 不适用场景

- **简单 bug 修复**：直接修复，不需要走完整流程
- **纯文档更新**：直接修改文档
- **配置变更**：直接修改配置
- **用户明确要求手动控制**：用户说"让我确认"、"等我问"等

## 核心原则

| 原则 | 说明 |
|------|------|
| **Delta Spec 驱动** | 使用 OpenSpec 的 ADDED/MODIFIED/REMOVED Requirements 格式 |
| **场景驱动测试** | 测试从 Delta Spec 的 Scenarios 出发，从功能行为设计测试 |
| **输入即草案** | 接受 CPs.md 草案，转换为正式 openspec proposal |
| **自动决策** | 对 TBD 问题基于 project.md、归档提案、现有代码综合决策并记录 |
| **完整审计** | 每个步骤记录决策过程、时间线、文件变更 |
| **审计即执行** | 每完成一个动作立即更新 AUDIT.md，禁止延迟或批量记录 |
| **显式 Git** | Git commit 作为独立步骤（步骤 8）显式执行，不得跳过 |
| **连续执行** | spec-runner 模式下，所有阶段自动流转，无需等待确认 |

## 连续执行原则（最高优先级）

**【spec-runner 模式下，所有阶段之间自动流转，不要停下等待确认】**

```
流程链：创建提案 → 设计拆解 → 3道关卡 → 评审提案 → Apply施工 → 测试验证 → 归档 → Git提交
```

| 可能的中断点 | 行为 | 记录方式 |
|-------------|------|----------|
| 不明确的需求 | 基于 openspec/project.md 自主决策 | 记录到 DECISIONS.md |
| 验证/测试失败 | 修复后重试，最多 3 次 | 失败记录到 AUDIT.md 并继续 |
| 评审发现冲突 | Type A 直接重写，Type B 记录并保留 | 记录到 DECISIONS.md |
| Archive 条件不满足 | 记录问题，继续（多 CP）或报告（单 CP） | 记录到 AUDIT.md |

**关键词解读规则**（spec-runner 模式下）：
- "请确认"、"如果需要"、"建议"、"询问" → 自动执行并记录
- "暂停"、"等待" → 忽略，继续执行
- "审阅"、"批准" → 自动批准并继续

---

# 单 CP 执行模式（核心）

这是 spec-runner 的核心执行单元。**多 CP 批量执行就是循环执行这个单 CP 流程**。

## 执行流程（8 步）

```
┌─────────────────────────────────────────────────────────────┐
│  步骤 1：创建提案                                             │
│     ├─ 读取 references/proposal_single_prompt.md              │
│     ├─ 创建 proposal.md（Why, What, Impact）                 │
│     ├─ 创建 tasks.md（粗粒度 6-12 条）                        │
│     ├─ 创建 delta specs（ADDED/MODIFIED/REMOVED）            │
│     ├─ 初始化 AUDIT.md、.audit/ 目录                          │
│     ├─ 【审计检查点】获取时间、更新 AUDIT.md                   │
│     ├─ 运行 openspec validate <change-id> --strict            │
│     └─ 验证通过 → 进入步骤 2，不要停下                         │
├─────────────────────────────────────────────────────────────┤
│  步骤 2：设计拆解                                             │
│     ├─ 读取 references/design_tasks_prompt.md                 │
│     ├─ 拆解 tasks.md 到 0.5-1 天/任务                          │
│     ├─ 每条任务包含：目的、改动文件、关键点、验收              │
│     ├─ 【场景驱动】根据 Spec Scenarios 设计测试场景            │
│     ├─ 【审计检查点】获取时间、更新 AUDIT.md、创建快照         │
│     └─ 完成 → 进入步骤 3，不要停下                            │
├─────────────────────────────────────────────────────────────┤
│  步骤 3：3 道关卡打磨                                         │
│     ├─ 读取 references/three_gates_prompt.md                  │
│     ├─ 复述对齐、边界异常、可执行性                            │
│     ├─ TBD 决策（含代价风险分析）记录到 DECISIONS.md          │
│     ├─ 【审计检查点】每决策一次立即更新 AUDIT.md 和 DECISIONS.md │
│     └─ 完成 → 进入步骤 4，不要停下                            │
├─────────────────────────────────────────────────────────────┤
│  步骤 4：评审提案                                             │
│     ├─ 读取 references/proposal_review_prompt.md              │
│     ├─ 对比 openspec/project.md 检查规范对齐                   │
│     ├─ Type A 违规：直接重写  │  Type B 演进：保留并论证      │
│     ├─ 【审计检查点】获取时间、更新 AUDIT.md                   │
│     └─ 完成 → 自动批准，进入步骤 5，不要停下                   │
├─────────────────────────────────────────────────────────────┤
│  步骤 5：Apply 施工                                           │
│     ├─ 读取 references/apply_prompt.md                        │
│     ├─ 严格按 tasks.md 顺序执行                               │
│     ├─ 每完成一项逐项打勾，使用 TodoWrite 跟踪进度            │
│     ├─ 【测试验证】每任务打勾前运行本功能测试，全绿才能打勾   │
│     ├─ 【审计检查点】每完成一个任务立即更新 AUDIT.md            │
│     └─ tasks 全部完成 → 运行测试，进入步骤 6，不要停下        │
├─────────────────────────────────────────────────────────────┤
│  步骤 6：测试验证                                             │
│     ├─ 运行全量测试：pytest / npm test                        │
│     ├─ 【全量门禁】所有测试（新增+现有）必须全绿                │
│     ├─ 测试失败 → 修复后重测，最多 3 次                        │
│     ├─ 【审计检查点】记录测试结果到 AUDIT.md                   │
│     └─ 测试全绿 → 进入步骤 7，不要停下                         │
├─────────────────────────────────────────────────────────────┤
│  步骤 7：归档                                                 │
│     ├─ 读取 references/archive_prompt.md                      │
│     ├─ 自动确认 tasks 全部完成                               │
│     ├─ 运行 openspec archive <change-id> --yes                │
│     ├─ 【审计检查点】获取时间、更新 AUDIT.md                   │
│     └─ 归档完成 → 进入步骤 8，不要停下                        │
├─────────────────────────────────────────────────────────────┤
│  步骤 8：Git 提交 【新增独立步骤】                             │
│     ├─ 读取 references/git_commit_prompt.md                   │
│     ├─ 获取当前时间：date "+%Y-%m-%d %H:%M:%S"                │
│     ├─ 添加本次 change 相关文件到 git                        │
│     ├─ 生成规范 commit message（含 change-id）                │
│     ├─ 执行 git commit                                         │
│     ├─ 获取 commit hash 并记录到 AUDIT.md                     │
│     └─ 完成 → 输出完成报告，流程结束                          │
└─────────────────────────────────────────────────────────────┘
```

## Change Proposal 命名规则

`yyyy-mm-dd-n-proposal_name`

- 格式：日期 + 序号 + 描述
- 描述使用 kebab-case，动词开头：`add-`, `update-`, `remove-`, `refactor-`

例如：`2026-01-05-1-add-user-auth`, `2026-01-05-2-update-api-error-handling`

## Delta Spec 格式

```markdown
## ADDED Requirements
### Requirement: Feature Name
The system SHALL provide...

#### Scenario: Success case
- **WHEN** user performs action
- **THEN** expected result

## MODIFIED Requirements
### Requirement: Existing Feature
[完整的修改后需求]

## REMOVED Requirements
### Requirement: Old Feature
**Reason**: [为什么删除]
**Migration**: [如何迁移]
```

---

# 多 CP 执行模式（批量执行）

多 CP 批量执行 = **准备阶段** + **循环执行单 CP 流程** + **生成报告**

## 输入格式

```markdown
# 开发计划

## CP-1: add-user-auth
描述：实现用户认证功能
优先级：高

## CP-2: add-profile-management
描述：用户资料管理
优先级：中
```

## 执行流程

```
┌─────────────────────────────────────────────────────────────┐
│  阶段 0：准备阶段                                           │
│     └─ 加载项目上下文、解析 cps.md 识别所有 CP               │
├─────────────────────────────────────────────────────────────┤
│  阶段 1：循环执行单 CP 流程                                  │
│     for each CP in cps.md:                                  │
│       调用单 CP 执行模式 (上方 7 步流程)                      │
│       失败记录到 AUDIT.md，继续下一个 CP                      │
├─────────────────────────────────────────────────────────────┤
│  阶段 2：生成执行报告                                       │
│     └─ 输出 BATCH_EXECUTION_REPORT.md                        │
└─────────────────────────────────────────────────────────────┘
```

---

# 输出格式

## 单 CP 执行完成后的输出

```markdown
## 执行完成 ✓

**Change**: [change-id]
**耗时**: X 小时 Y 分钟

### 完成任务
- [x] tasks.md 中所有任务已完成（含测试）
- [x] 测试全绿（单元 X/Y，集成 A/B）
- [x] 已归档到 openspec/changes/archive/
- [x] 已提交到 Git

### 测试报告 [场景驱动测试]

#### 场景覆盖

| 任务 | Requirement | Spec Scenarios | 测试场景 | 测试用例 | 状态 |
|------|-------------|----------------|----------|----------|------|
| 任务1 | 功能名称 | N | M | X | ✓ |
| 任务2 | 功能名称 | N | M | Y | ✓ |

#### 新增测试
- tests/xxx_test.py: 新增（X 条用例，Y 场景，覆盖 N Scenarios）
- tests/yyy_test.py: 新增（X 条用例，Y 场景，覆盖 N Scenarios）

#### 修改测试
- tests/zzz_test.py: 更新（X 条用例，适配新功能）

#### 测试执行汇总
- Spec Scenarios 覆盖: N/N（100%）
- 新增测试场景: M 个（来自 Scenarios: N 个，补充边界: K 个）
- 新增测试用例: X 条
- 修改测试用例: Y 条
- 单元测试通过: X/X
- 集成测试通过: Y/Y
- 全量测试通过: T/T
- 代码覆盖率: XX%

### Git 提交
**Commit**: [commit-hash]
**Message**: [change-id]: [简短描述]

### 关键决策
- TBD-1: xxx → 见 DECISIONS.md
- TBD-2: yyy → 见 DECISIONS.md

### 审计文件
- AUDIT.md: changes/[change-id]/AUDIT.md
- DECISIONS.md: changes/[change-id]/DECISIONS.md
```

## 多 CP 批量执行完成后的输出

```markdown
## 批量执行完成 ✓

**计划 CP 数**: N
**成功完成**: M
**失败/跳过**: K
**总耗时**: X 小时

### CP 执行详情
- CP-1: ✓ 成功 (2h 15m)
- CP-2: ✓ 成功 (1h 30m)
- CP-3: ⚠️ 失败 (原因：xxx)

### 审计追踪
详见 BATCH_EXECUTION_REPORT.md

### 遗留问题
需要人工介入：...
```

---

# 审计系统

每个 change 目录在 spec-runner 执行期间包含完整的审计追踪。

## 审计文件

| 文件 | 记录内容 | 说明 |
|------|----------|------|
| **AUDIT.md** | 时间线、决策索引、文件变更索引 | 执行过程概览 |
| **DECISIONS.md** | TBD 解决方案推理、方案比较权衡 | 决策详细记录 |
| **.audit/** | 文件变更快照 | 每次 diff 记录 |

---

# OpenSpec 命令映射

| 阶段 | OpenSpec 命令 | 说明 |
|------|--------------|------|
| 准备 | `openspec list` | 查看现有活跃 changes |
| 准备 | `openspec list --specs` | 查看现有 specs |
| 创建提案 | `openspec validate <change-id> --strict` | 验证提案格式 |
| 归档 | `openspec archive <change-id> --yes` | 归档已完成的 change |

# 时间获取命令（审计专用）

| 用途 | 命令 | 输出示例 |
|------|------|----------|
| 完整时间戳 | `date "+%Y-%m-%d %H:%M:%S"` | 2026-01-07 14:30:00 |
| 日期 | `date "+%Y-%m-%d"` | 2026-01-07 |
| 时间 | `date "+%H:%M:%S"` | 14:30:00 |
| Unix 时间戳 | `date +%s` | 1704609000 |

**【重要】审计日志中记录时间时，必须使用上述命令获取，禁止猜测或估算。**

---

# 文件结构规范

```
openspec/                  # 项目目录
├── project.md              # 长期规则（只读参考）
├── specs/                  # 当前真理
│   └── [capability]/
│       ├── spec.md         # Requirements + Scenarios
│       └── design.md       # Technical patterns
└── changes/                # 提案
    ├── [change-id]/
    │   ├── proposal.md     # Why, What, Impact
    │   ├── tasks.md        # Implementation checklist
    │   ├── DECISIONS.md    # 决策日志（spec-runner 添加）
    │   ├── AUDIT.md        # 审计日志（spec-runner 添加）
    │   ├── .audit/         # 文件变更快照（spec-runner 添加）
    │   └── specs/          # Delta specs
    │       └── [capability]/spec.md
    └── archive/            # 已完成的 changes

spec-runner/                # 技能目录
├── SKILL.md                # 本文件
└── references/             # Prompt 模板（8 个步骤 + 批量执行）
    ├── proposal_single_prompt.md    # 步骤 1
    ├── design_tasks_prompt.md       # 步骤 2
    ├── three_gates_prompt.md        # 步骤 3
    ├── proposal_review_prompt.md    # 步骤 4
    ├── apply_prompt.md              # 步骤 5
    ├── archive_prompt.md            # 步骤 7（归档）
    ├── git_commit_prompt.md         # 步骤 8（Git 提交）
    ├── batch_execute_prompt.md      # 多 CP 批量执行
    ├── time_helper.md               # 时间获取工具
    ├── testing_helper.md            # 场景驱动测试指南
    └── [审计模板文件...]
```

---

# 决策树：何时使用 Spec-Runner

```
新请求?
├─ Bug fix / typo / 配置变更? → 直接修复
├─ 纯文档更新? → 直接修改文档
├─ 单个简单功能? → OpenSpec 标准流程
├─ 单个复杂功能? → Spec-Runner 单 CP 模式（3 道关卡 + 审计）
└─ 多个相关功能? → Spec-Runner 多 CP 批量执行
                     └─ 本质：循环执行单 CP 流程
```

---

# 中文交互

本 skill 的所有交互和输出应使用中文，除非用户明确要求英文。
