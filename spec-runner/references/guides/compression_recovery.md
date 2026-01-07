# 上下文压缩与恢复机制

## 压缩-恢复完整流程

```
步骤 4 结束 → 写入 STATE_RELOAD.md → 完成压缩前准备
                                    ↓
                          系统自动压缩（无法主动触发）
                                    ↓
              检测到压缩 → 恢复步骤 → 继续执行步骤 5
```

## 压缩策略

| 方式 | 时机 | 说明 |
|------|------|------|
| 压缩前准备 | 步骤 3/4 结束时 | 写入 STATE_RELOAD.md，做好恢复准备 |
| 系统压缩 | 任意时刻 | Claude Code 自动触发，被动检测 |

## 压缩检测

判断上下文是否被压缩（满足任一即触发）：

1. **看到系统提示**：
   > This session is being continued from a previous conversation that ran out of context. The conversation is summarized below:

2. **无法回忆核心原则**：记不清"核心原则"表格中的内容

3. **不确定当前状态**：不清楚当前步骤或下一步行动

## 恢复步骤

**第 1 步：读取状态文件**
```bash
cat ./openspec/changes/[change-id]/STATE_RELOAD.md
```

**第 2 步：读取审计日志**
```bash
cat ./openspec/changes/[change-id]/AUDIT.md
```

**第 3 步：刷新核心原则**
```bash
cat ./spec-runner/SKILL.md
```

**第 4 步：读取当前步骤指令**
```bash
cat ./spec-runner/references/steps/[0X]_*.md
```

## 优先读取顺序

压缩恢复时按以下顺序读取文件：
1. `STATE_RELOAD.md`（当前 change 目录）- 核心状态
2. `AUDIT.md`（当前 change 目录）- 执行历史
3. `SKILL.md` - 刷新核心原则
4. `[当前步骤]_prompt.md`（`references/steps/`）- 详细指令
5. `testing.md`（`references/guides/`）- 测试原则（如适用）
