---
name: spec-runner
description: OpenSpec 执行引擎。自动执行 Change Proposals：创建提案→设计拆解→提案细化→评审→Apply→归档→提交。支持单 CP/批量执行，自动决策，完整审计。
---

# Spec-Runner

OpenSpec 的执行引擎，自动执行完整开发流程。

## 快速开始

```bash
# 用户输入
"用 spec-runner 帮我实现用户认证功能"

# 自动执行（7步，连续流转无停顿）
1.创建提案 → 2.设计拆解 → 3.提案细化 → 4.评审提案
→ 5.Apply施工 → 6.归档 → 7.Git提交

# 输出
- proposal.md, tasks.md, AUDIT.md, DECISIONS.md
- Git commit
- 完成报告（含测试报告）
```

## 核心原则

| 原则 | 说明 |
|------|------|
| Delta Spec 驱动 | 使用 ADDED/MODIFIED/REMOVED Requirements 格式 |
| 场景驱动测试 | 测试从 Spec Scenarios 出发，验证符合规范 |
| 自动决策 | TBD 问题基于 project.md 自主决策并记录 |
| 审计即执行 | 每完成一个动作立即更新 AUDIT.md |
| 显式 Git | 必须执行步骤 8 的 git commit |
| 连续执行 | 所有阶段自动流转，无需等待确认 |

## 执行流程

### 单 CP 模式

```
1.创建提案 → 2.设计拆解（含测试场景） → 3.提案细化
→ 4.评审提案 → 5.Apply施工（每功能点测试）
→ 6.归档（全量测试后） → 7.Git提交
```

### 多 CP 模式

循环执行单 CP 流程，失败记录到 AUDIT.md 后继续下一个。

## 完成报告格式

```markdown
## 执行完成 ✓

**Change**: [change-id] **耗时**: Xh Ym

### 测试报告
| 任务 | Spec Scenarios | 测试场景 | 测试用例 | 状态 |
|------|----------------|----------|----------|------|
| 任务1 | N | M | X | ✓ |

- Spec Scenarios 覆盖: 100%
- 全量测试通过: T/T
- 代码覆盖率: XX%

### Git 提交
**Commit**: [hash]

### 审计文件
- AUDIT.md, DECISIONS.md
```

## 文件索引

### 步骤文件
执行每一步前，必须阅读对应的 prompt 文件。

| 步骤 | 文件 | 内容 |
|------|------|------|
| 1 | 01_proposal_single_prompt.md | 创建提案 |
| 2 | 02_design_tasks_prompt.md | 设计拆解+测试场景 |
| 3 | 03_proposal_refine_prompt.md | 提案细化 |
| 4 | 04_proposal_review_prompt.md | 评审提案 |
| 5 | 05_apply_prompt.md | Apply施工与测试 |
| 6 | 06_archive_prompt.md | 全量测试与归档 |
| 7 | 07_git_commit_prompt.md | Git 提交 |

### 辅助文件
在执行过程中按需查阅。

| 文件 | 阅读时机 |
|------|----------|
| 00_AUDIT_template.md | 步骤 1 初始化 AUDIT.md |
| 00_AUDIT_snapshot_template.md | 各步骤有文件变更时 |
| 00_DECISIONS_template.md | 有 TBD 决策时 |
| 00_testing_helper.md | 步骤 2 设计测试、步骤 5 执行测试、归档前验证 |
| 00_batch_execute_prompt.md | 用户要求批量执行时 |

## 时间命令

```bash
date "+%Y-%m-%d %H:%M:%S"  # 审计用，禁止猜测
```

## 决策树

```
新请求?
├─ Bug/配置/文档? → 直接修复
├─ 简单功能? → OpenSpec 标准流程
├─ 复杂功能? → Spec-Runner 单 CP
└─ 多个功能? → Spec-Runner 批量执行
```

## 中文交互

所有交互和输出使用中文，除非用户明确要求英文。
