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

## 执行流程

### 单 CP 模式

```
1.创建提案 → 2.设计拆解（含测试场景） → 3.提案细化
→ 4.评审提案 → 5.Apply施工（每功能点测试）
→ 6.归档（全量测试后） → 7.Git提交
```

### 多 CP 模式

循环执行单 CP 流程，失败记录到 AUDIT.md 后继续下一个。

## 核心原则

| 原则 | 说明 |
|------|------|
| Delta Spec 驱动 | 使用 ADDED/MODIFIED/REMOVED Requirements 格式 |
| 场景驱动测试 | 测试从 Spec Scenarios 出发，验证符合规范 |
| 自动决策 | TBD 问题基于 project.md 自主决策并记录 |
| 审计即执行 | 每完成一个动作立即更新 AUDIT.md |
| 显式 Git | 必须执行步骤 7 的 git commit |
| 连续执行 | 所有阶段自动流转，无需等待确认 |
| 全绿为准 | 测试不通过不能标记任务完成，禁止取巧 |

## 文件索引

### 步骤文件

执行每一步前，必须阅读对应的 prompt 文件。

| 步骤 | 文件 | 内容 |
|------|------|------|
| 1 | references/steps/01_proposal.md | 创建提案 |
| 2 | references/steps/02_design_tasks.md | 设计拆解+测试场景 |
| 3 | references/steps/03_refine.md | 提案细化 |
| 4 | references/steps/04_review.md | 评审提案 |
| 5 | references/steps/05_apply.md | Apply施工与测试 |
| 6 | references/steps/06_archive.md | 全量测试与归档 |
| 7 | references/steps/07_commit.md | Git 提交 |

### 模板文件

用于初始化特定文件。

| 文件 | 用途 |
|------|------|
| references/templates/AUDIT.md | 初始化 AUDIT.md |
| references/templates/AUDIT_snapshot.md | 创建审计快照 |
| references/templates/DECISIONS.md | 记录 TBD 决策 |
| references/templates/STATE_RELOAD.md | 压缩恢复状态文件 |

### 指南文件

特定主题的详细指导。

| 文件 | 阅读时机 |
|------|----------|
| references/guides/testing.md | 步骤 2 设计测试、步骤 5 执行测试、归档前验证 |
| references/guides/batch_execute.md | 批量执行时 |
| references/guides/compression_recovery.md | 上下文压缩后恢复 |

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

## 实用信息

### 时间命令

```bash
date "+%Y-%m-%d %H:%M:%S"  # 审计用，禁止猜测
```

### 中文交互

所有交互和输出使用中文，除非用户明确要求英文。

## 上下文压缩与恢复机制

详细说明：`references/guides/compression_recovery.md`

### 快速检测

看到以下提示说明上下文已被压缩：
> This session is being continued from a previous conversation that ran out of context. The conversation is summarized below:

### 快速恢复

```bash
# 读取状态恢复文件
cat ./openspec/changes/[change-id]/STATE_RELOAD.md

# 刷新核心原则
cat ./spec-runner/SKILL.md
```
