# 步骤 1：创建提案

## 使用时机

单 CP 执行模式的第 1 步。当用户要求处理单个 change 或批量执行时处理每个 CP 的第一步。

## 提案命名规则

`yyyy-mm-dd-n-proposal_name`

- 格式：日期 + 序号 + 描述
- 描述使用 kebab-case，动词开头：`add-`, `update-`, `remove-`, `refactor-`

例如：`2026-01-05-1-add-user-auth`, `2026-01-05-2-update-api-error-handling`

### 序号确定步骤（重要）

**在确定提案序号前，必须执行以下步骤**：

1. 查看已归档提案：`ls openspec/changes/archive/`
2. 查看当前活跃提案：`openspec list`
3. 确定当天最新的序号（如已有 `2026-01-05-1-*`，则下一个使用 `-2-`）
4. 如果同一天有多人协作，序号递增避免冲突

## Prompt 模板

```
为以下 change 写一个 OpenSpec proposal：

## 必需内容

### Scope / Out of scope
- Scope：本次变更明确要做的事情
- Out of scope：明确不做的事情（至少列出 3 条）

### Acceptance criteria
- 必须可测试、可判定
- 避免含糊词汇如"支持""优化""完善"

### Data / API 接触面
- 仅列清单，不展开设计
- 列出涉及的数据表、API 接口

### Risks & mitigations
- 列出潜在风险
- 对应的缓解措施

### Rollout / rollback
- 上线计划
- 回滚方案

### tasks.md 骨架
- 粗粒度 6–12 条任务
- 每条任务描述主要工作内容
```

## 输出要求

- 生成 `changes/[change-id]/proposal.md`
- 生成 `changes/[change-id]/tasks.md`
- 初始化 `changes/[change-id]/AUDIT.md`（审计日志）
- 创建 `changes/[change-id]/.audit/` 目录
- 主干内容用中文写

## 验证步骤

提案创建完成后，必须运行：

```bash
openspec validate <change-id> --strict
```

- 修复所有验证错误后继续
- 验证通过后 → 进入**步骤 2：设计拆解**，不要停下等待确认

## 审计日志要求

**审计即执行：每完成一个动作必须立即更新审计日志**

### 审计检查点

**每完成一个动作后，立即执行以下操作**：

```bash
# 1. 获取当前时间（必须使用命令，禁止猜测）
date "+%Y-%m-%d %H:%M:%S"

# 2. 立即更新 AUDIT.md
# 3. 如有文件变更，立即创建 .audit/ 快照
```

### 步骤开始时

在 AUDIT.md 中记录

```markdown
# 审计日志

## 时间线

| 步骤 | 开始时间 | 结束时间 | 耗时 | 状态 |
|------|----------|----------|------|------|
| 步骤1：创建提案 | [使用 date 命令获取] | | | 进行中 |
```

### 每创建一个文件后

```bash
# 1. 获取时间
TIME=$(date "+%Y-%m-%d %H:%M:%S")

# 2. 立即创建 .audit/ 快照
# 在 .audit/ 目录创建对应的快照文件

# 3. 更新文件变更索引
```

### 步骤结束时

```bash
# 1. 获取结束时间
END_TIME=$(date "+%Y-%m-%d %H:%M:%S")

# 2. 计算耗时
# 3. 更新 AUDIT.md 时间线
```

**禁止延迟记录或批量记录审计信息。**

### 快照格式

每创建一个文件后，立即在 `.audit/` 目录创建快照：

```markdown
# 快照 NNN: [描述]

**时间**: [使用 date 命令获取]
**步骤**: 创建提案
**操作**: admin (Claude)
**文件**: [文件名]
**变更类型**: 创建

## 变更内容

\`\`\`diff
+ [完整文件内容]
\`\`\`
```
