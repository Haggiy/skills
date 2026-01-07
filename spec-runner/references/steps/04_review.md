# 步骤 4：评审提案

## 使用时机

单 CP 执行模式的第 4 步。在步骤 3（3 道关卡打磨）完成后自动进入。

## 步骤开始：状态检查（压缩恢复点）

**进入本步骤时，首先执行以下检查**：

### 检查 1：上下文完整性

```
请自我检查：
- 你能完整回忆起 spec-runner 的 7 条核心原则吗？
- 你清楚当前执行的 change-id 吗？
- 你知道如何处理 Type A vs Type B 冲突吗？
```

**如果任何答案为"否"**，立即执行恢复：

```bash
# 1. 读取状态恢复文件
cat ./openspec/changes/[change-id]/STATE_RELOAD.md

# 2. 读取审计日志
cat ./openspec/changes/[change-id]/AUDIT.md

# 3. 刷新核心原则
cat ./spec-runner/SKILL.md

# 4. 重新读取本文件
cat ./spec-runner/references/steps/04_review.md
```

### 检查 2：确认当前状态

- 确认 `change-id` 是否正确
- 确认步骤 1-3 已完成（查看 AUDIT.md）
- 确认 proposal.md、tasks.md 已创建

**只有完成状态检查后，才继续执行评审工作。**

## Prompt 模板

```
请作为首席架构师评审此提案。

## 评审资料来源

1. **规范依据**
   - 读取 `openspec/project.md` 了解项目规范

2. **历史决策**
   - 查阅 `changes/archive/` 中已归档提案
   - 了解类似功能的历史决策和实现方式

3. **现有实现**
   - 扫描与本次提案相关的已实现代码
   - 确保新提案与现有代码风格、模式一致

## 评审内容

请基于以上资料，重点区分两类冲突：

### Type A（违规）
与现有体系不兼容且无必要的偏离。
- 请直接重写这部分以对齐规范

### Type B（演进）
虽然偏离现有规范，但代表了技术演进方向或更优解。
- 请保留并论证其合理性

## 输出要求

基于以上逻辑输出一份修订报告，包含：
1. 发现的冲突点（按 Type A / Type B 分类）
2. Type A 冲突的重写建议
3. Type B 演进的合理性论证
4. 总体评审结论

【spec-runner 模式】评审完成后：
- Type A 冲突：直接重写并继续
- Type B 演进：保留并记录到 DECISIONS.md
- 评审完成后 → 自动批准，进入**步骤 5：Apply 施工**，不要停下等待确认
```

## 评审后验证

评审完成后，必须执行：

```bash
# 验证提案格式
openspec validate <change-id> --strict
```

- 验证通过后 → 进入**步骤 5：Apply 施工**

## 审计日志要求

**步骤开始时**：在 AUDIT.md 时间线中添加

```markdown
| 步骤4：评审提案 | [开始时间] | | | 进行中 |
```

**步骤结束时**：
1. 更新 AUDIT.md 时间线
2. **创建/更新 STATE_RELOAD.md**（重要！这是压缩恢复的关键点）
3. 在 `.audit/` 创建快照（如有文件修改）
4. 如有 Type B 演进，记录到 DECISIONS.md

### STATE_RELOAD.md 写入指令

在步骤 4 结束时，必须在 change 目录创建/更新 STATE_RELOAD.md：

```bash
# 参考 spec-runner/references/templates/STATE_RELOAD.md
# 更新文件：
./openspec/changes/[change-id]/STATE_RELOAD.md
```

**必填内容**：
- Change ID
- 当前步骤：步骤 4 结束
- 下一步：步骤 5 Apply 施工
- 核心原则（从 SKILL.md 提炼）
- 下一步行动：从 tasks.md 第一个任务开始，每项测试全绿后打勾

### 压缩前准备

**STATE_RELOAD.md 写入完成后，确认以下事项**：

- [ ] STATE_RELOAD.md 已更新（当前步骤、下一步行动）
- [ ] AUDIT.md 已同步最新审计记录
- [ ] .audit/ 快照已创建（如有文件修改）
- [ ] tasks.md 第一个任务已确认

**说明**：系统可能随时自动压缩，上述准备确保压缩后能完整恢复。
