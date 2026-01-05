# spec-runner

OpenSpec 的批量执行引擎。

## 安装

```bash
mkdir -p ~/.claude/skills
ln -s $(pwd) ~/.claude/skills/spec-runner
```

## 功能

- 批量执行多个 Change Proposals
- 自动决策（基于 openspec/project.md、已归档提案、现有代码）
- 完整审计追踪（AUDIT.md + DECISIONS.md + .audit/）
- 支持 3 道关卡打磨
