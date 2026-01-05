---
name: spec-runner
description: Spec-Runner 是 OpenSpec 的批量执行引擎。当用户请求：(1) 批量执行多个 CPs（cps.md 开发计划），(2) 单个复杂功能需要全流程管理（3 道关卡 + 审计），(3) 需要自动决策和审计追踪时使用此技能。接受 Change Proposals 草案，执行完整开发流程（创建提案→设计拆解→3道关卡→评审提案→Apply施工→测试验证→归档），输出实现结果和审计报告。
---

# Spec-Runner 开发框架

Spec-Runner 是 **OpenSpec 的批量执行引擎**，将多个 Change Proposals（变更提案）按照规范自动执行完整开发流程。它整合了 OpenSpec 的 delta spec 格式，通过自动决策、审计追踪，实现从提案到交付的自动化。

## 核心定位

```
┌─────────────────────────────────────────────────────────────┐
│                     OpenSpec 生态                            │
├─────────────────────────────────────────────────────────────┤
│  OpenSpec (规范层)   →  定义 CP 结构、delta spec 格式、3 阶段  │
│  Spec-Runner (执行层) →  批量执行、自动决策、审计追踪           │
└─────────────────────────────────────────────────────────────┘
```

## 核心原则

- **Delta Spec 驱动**：使用 OpenSpec 的 ADDED/MODIFIED/REMOVED Requirements 格式
- **输入即草案**：接受 CPs.md 草案，转换为正式 openspec proposal
- **自动决策**：对 TBD 问题基于 openspec/project.md、已归档提案、现有代码综合决策并记录
- **完整审计**：每个步骤记录决策过程、时间线、文件变更
- **顺序执行**：按 cps.md 顺序从前向后逐个执行 CP

---

## 主流程：批量执行模式 ⭐

**触发条件**：用户提供一个包含多个 change proposals 的开发计划文件

### 输入格式

```markdown
# 开发计划

## CP-1: add-user-auth
描述：实现用户认证功能
优先级：高

## CP-2: add-profile-management
描述：用户资料管理
优先级：中
```

### 执行流程

```
┌─────────────────────────────────────────────────────────────┐
│  1. 准备阶段                                                 │
│     - 运行 openspec list 查看现有活跃 changes                │
│     - 读取 openspec/project.md 获取项目规则                  │
│     - 查阅已归档提案（changes/archive/），了解历史决策        │
│     - 扫描与本次提案相关的已实现代码                         │
├─────────────────────────────────────────────────────────────┤
│  2. 对每个 CP 执行完整流程（按 cps.md 顺序从前向后）         │
│     ┌─────────────────────────────────────────────────────┐ │
│     │ 2.1 创建提案（草案 → 正式 openspec proposal）         │ │
│     │     - 创建 proposal.md, tasks.md, delta specs        │ │
│     │     - 运行 openspec validate <change-id> --strict    │ │
│     ├─────────────────────────────────────────────────────┤ │
│     │ 2.2 设计拆解                                         │ │
│     │     - 拆解 tasks.md 到 0.5-1 天/任务                  │ │
│     ├─────────────────────────────────────────────────────┤ │
│     │ 2.3 3 道关卡打磨                                     │ │
│     │     - 复述对齐、边界异常、可执行性                     │ │
│     │     - TBD 自动决策（代价风险分析），记录到 DECISIONS.md │ │
│     ├─────────────────────────────────────────────────────┤ │
│     │ 2.4 评审提案                                         │ │
│     │     读取 references/proposal_review_prompt.md        │ │
│     │     - 对比 openspec/project.md 检查规范对齐           │ │
│     │     - 查阅已归档提案，了解历史决策和实现方式          │ │
│     │     - 扫描与本次提案相关的已实现代码，确保一致性      │ │
│     │     - Type A 违规：直接重写                           │ │
│     │     - Type B 演进：保留并论证                         │ │
│     ├─────────────────────────────────────────────────────┤ │
│     │ 2.5 Apply 施工                                       │ │
│     │     - 按 tasks.md 顺序执行                           │ │
│     │     - 每完成一项打勾                                  │ │
│     ├─────────────────────────────────────────────────────┤ │
│     │ 2.6 测试验证                                         │ │
│     │     - 运行单元测试、集成测试                          │ │
│     │     - 测试全绿后继续                                  │ │
│     ├─────────────────────────────────────────────────────┤ │
│     │ 2.7 归档                                             │ │
│     │     - 确认 tasks 全部完成                             │ │
│     │     - 运行 openspec archive <change-id> --yes        │ │
│     └─────────────────────────────────────────────────────┘ │
├─────────────────────────────────────────────────────────────┤
│  3. 生成执行报告                                             │
│     - 输出 BATCH_EXECUTION_REPORT.md                        │
│     - 包含每个 CP 的状态、耗时、关键决策、审计链接            │
└─────────────────────────────────────────────────────────────┘
```

详见 [references/batch_execute_prompt.md](references/batch_execute_prompt.md)

---

## 单 CP 执行模式

**触发条件**：用户直接要求处理单个 change

### 执行流程

```
┌─────────────────────────────────────────────────────────────┐
│  1. 创建提案                                                 │
│     读取 references/proposal_single_prompt.md                │
│     - 创建 proposal.md（Why, What, Impact）                 │
│     - 创建 tasks.md（粗粒度 6-12 条）                        │
│     - 创建 delta specs（ADDED/MODIFIED/REMOVED）            │
│     - 初始化 AUDIT.md、.audit/ 目录                          │
│     - 运行 openspec validate <change-id> --strict            │
├─────────────────────────────────────────────────────────────┤
│  2. 设计拆解                                                 │
│     读取 references/design_tasks_prompt.md                   │
│     - 拆解 tasks.md 到 0.5-1 天/任务                          │
│     - 每条任务包含：目的、改动文件、关键点、验收              │
├─────────────────────────────────────────────────────────────┤
│  3. 3 道关卡打磨                                             │
│     读取 references/three_gates_prompt.md                    │
│     - 复述对齐：防止理解偏差                                 │
│     - 边界与异常：列出 15 个场景，标注纳入/推迟              │
│     - 可执行性：确保每条任务可独立完成                       │
│     - TBD 决策（含代价风险分析）记录到 DECISIONS.md          │
├─────────────────────────────────────────────────────────────┤
│  4. 评审提案                                                 │
│     读取 references/proposal_review_prompt.md                │
│     - 对比 openspec/project.md 检查规范对齐                   │
│     - 查阅已归档提案，了解历史决策和实现方式                  │
│     - 扫描与本次提案相关的已实现代码，确保一致性              │
│     - Type A 违规：直接重写                                  │
│     - Type B 演进：保留并论证                                │
├─────────────────────────────────────────────────────────────┤
│  5. Apply 施工                                               │
│     读取 references/apply_prompt.md                          │
│     - 严格按 tasks.md 顺序执行                               │
│     - 每完成一项逐项打勾                                     │
│     - 遇到不明确处先停下提出澄清                             │
├─────────────────────────────────────────────────────────────┤
│  6. 测试验证                                                 │
│     - 运行单元测试：pytest / npm test                          │
│     - 运行集成测试（如有）                                      │
│     - 执行手动 smoke test（关键路径）                           │
│     - 测试全绿后继续                                            │
├─────────────────────────────────────────────────────────────┤
│  7. 归档                                                     │
│     读取 references/archive_prompt.md                        │
│     - 确认 tasks 全部完成                                    │
│     - 运行 openspec archive <change-id> --yes                │
└─────────────────────────────────────────────────────────────┘
```

### Change Proposal 命名规则

`yyyy-mm-dd-n-proposal_name`

- 格式：日期 + 序号 + 描述
- 描述使用 kebab-case，动词开头：`add-`, `update-`, `remove-`, `refactor-`

例如：`2026-01-05-1-add-user-auth`, `2026-01-05-2-update-api-error-handling`

### Delta Spec 格式（遵循 OpenSpec）

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

## 审计系统

每个 change 目录在 spec-runner 执行期间包含完整的审计追踪。

> **模板文件**：references/ 目录提供审计模板
> - `AUDIT_template.md` - 审计日志模板
> - `DECISIONS_template.md` - 决策日志模板
> - `AUDIT_snapshot_template.md` - 快照模板

### 审计文件

| 文件 | 记录内容 | 说明 |
|------|----------|------|
| **AUDIT.md** | 时间线、决策索引、文件变更索引 | 执行过程概览 |
| **DECISIONS.md** | TBD 解决方案推理、方案比较权衡 | 决策详细记录 |
| **.audit/** | 文件变更快照 | 每次 diff 记录 |

> 注意：这些文件是 spec-runner 运行时添加的，不在 openspec 标准结构中。

### AUDIT.md 结构

```markdown
# 审计日志

## 时间线

| 步骤 | 开始时间 | 结束时间 | 耗时 | 状态 |
|------|----------|----------|------|------|
| 创建提案 | 2026-01-05 10:00 | 10:15 | 15m | ✓ |
| 设计拆解 | 2026-01-05 10:15 | 10:45 | 30m | ✓ |
| 3道关卡 | 2026-01-05 10:45 | 11:00 | 15m | ✓ |
| 评审提案 | 2026-01-05 11:00 | 11:10 | 10m | ✓ |
| Apply施工 | 2026-01-05 11:10 | 14:00 | 2h50m | ✓ |
| 测试验证 | 2026-01-05 14:00 | 14:30 | 30m | ✓ |

## 决策记录索引

- [TBD-1] 数据模型选择 → 见 DECISIONS.md
- [TBD-2] API 认证方案 → 见 DECISIONS.md

## 文件变更索引

| 快照 | 文件 | 变更类型 | 关联步骤 |
|------|------|----------|----------|
| 001 | proposal.md | 创建 | 创建提案 |
| 002 | proposal.md | 修改 | 3道关卡 |
| 003 | proposal.md | 修改 | 评审提案 |
| 004 | tasks.md | 创建 | 设计拆解 |
```

### TBD 自动决策策略

| 原则 | 说明 |
|------|------|
| 项目利益优先 | 站在对项目整体最有利的角度决策 |
| 双重视角 | 以首席架构师和首席开发者的视角综合判断 |
| 不偏不倚 | 不怕难，也不搞过度复杂 |
| 代价风险分析 | 给出 3 种方案（简单/中等/稳健），比较复杂度、风险、扩展成本 |
| 记录推理 | 所有 TBD 决策必须记录到 `DECISIONS.md`（包含推理过程） |

---

## OpenSpec 命令映射

| 阶段 | OpenSpec 命令 | 说明 |
|------|--------------|------|
| 准备 | `openspec list` | 查看现有活跃 changes |
| 准备 | `openspec list --specs` | 查看现有 specs |
| 创建提案 | `openspec validate <change-id> --strict` | 验证提案格式 |
| 归档 | `openspec archive <change-id> --yes` | 归档已完成的 change |

---

## 文件结构规范

> **说明**：openspec/ 是项目目录（存放 proposals 和 specs），spec-runner/ 是技能目录。

```
openspec/                  # 项目目录
├── project.md              # 长期规则（只读参考）
├── specs/                  # 当前真理 - OpenSpec 格式
│   └── [capability]/
│       ├── spec.md         # Requirements + Scenarios
│       └── design.md       # Technical patterns
└── changes/                # 提案 - 应该改变什么
    ├── [change-id]/
    │   ├── proposal.md     # Why, What, Impact
    │   ├── tasks.md        # Implementation checklist
    │   ├── design.md       # Technical decisions (optional)
    │   ├── DECISIONS.md    # 决策日志（spec-runner 运行时添加）
    │   ├── AUDIT.md        # 审计日志（spec-runner 运行时添加）
    │   ├── .audit/         # 文件变更快照（spec-runner 运行时添加）
    │   │   ├── 001_xxx.md
    │   │   └── ...
    │   └── specs/          # Delta specs
    │       └── [capability]/
    │           └── spec.md # ADDED/MODIFIED/REMOVED
    └── archive/            # 已完成的 changes
        └── YYYY-MM-DD-[change-id]/

spec-runner/                # 技能目录
├── SKILL.md                # 技能定义
└── references/             # Prompt 模板
    ├── batch_execute_prompt.md
    ├── proposal_single_prompt.md
    ├── design_tasks_prompt.md
    ├── three_gates_prompt.md
    ├── proposal_review_prompt.md
    ├── apply_prompt.md
    ├── archive_prompt.md
    ├── AUDIT_template.md
    ├── DECISIONS_template.md
    └── AUDIT_snapshot_template.md
```

---

## 与 OpenSpec 的关系

| 特性 | OpenSpec | Spec-Runner |
|------|----------|-------------|
| **定位** | 规范层 | 执行引擎 |
| **核心价值** | 定义 CP 结构和 delta spec 格式 | 批量执行、自动决策、审计 |
| **文件格式** | proposal.md, tasks.md, delta specs | 兼容 OpenSpec 格式 |
| **工作流** | 3 阶段（创建/实现/归档） | 3 阶段 + 3 道关卡 + 审计 |
| **CLI** | openspec 命令 | 调用 openspec 命令 |
| **审计** | 无 | AUDIT.md + DECISIONS.md + .audit/ |

---

## 决策树：何时使用 Spec-Runner

```
新请求?
├─ Bug fix / typo / 配置变更? → 直接修复
├─ 单个简单功能? → OpenSpec 标准流程
├─ 单个复杂功能? → Spec-Runner 单 CP 模式（3 道关卡 + 审计）
└─ 多个相关功能? → Spec-Runner 批量执行模式 ⭐
```

---

## 中文交互

本 skill 的所有交互和输出应使用中文，除非用户明确要求英文。
