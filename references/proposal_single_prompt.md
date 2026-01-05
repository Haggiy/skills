# 创建单个 Change Proposal Prompt

## 使用时机

当用户需要为单个 change 创建提案时使用此 prompt。

## 提案命名规则

`yyyy-mm-dd-n-proposal_name`

- 格式：日期 + 序号 + 描述
- 描述使用 kebab-case，动词开头：`add-`, `update-`, `remove-`, `refactor-`

例如：`2026-01-05-1-add-user-auth`, `2026-01-05-2-update-api-error-handling`

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

修复所有验证错误后，方可提交用户审阅。

## 审计日志要求

**步骤开始时**：在 AUDIT.md 中记录

```markdown
# 审计日志

## 时间线

| 步骤 | 开始时间 | 结束时间 | 耗时 | 状态 |
|------|----------|----------|------|------|
| 创建提案 | [开始时间] | | | 进行中 |
```

**文件创建后**：在 `.audit/` 目录创建快照

```markdown
# 快照 001: initial_proposal

**时间**: [当前时间]
**步骤**: 创建提案
**操作**: admin (Claude)
**文件**: proposal.md

## 变更内容

\`\`\`diff
+ [完整文件内容]
\`\`\`
```

**步骤结束时**：更新 AUDIT.md 时间线，记录结束时间和耗时
