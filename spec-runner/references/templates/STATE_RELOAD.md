# 状态恢复文件

**目的**：在上下文被压缩后，快速恢复执行状态和核心原则。

---

## 当前状态

- **Change ID**: [change-id]
- **当前步骤**: 步骤 [N] - [步骤名称]
- **压缩检测点**: 步骤 [N-1] 结束时创建
- **更新时间**: [使用 date 命令获取]

## 核心原则（必须遵守）

| 原则 | 说明 |
|------|------|
| Delta Spec 驱动 | 使用 ADDED/MODIFIED/REMOVED Requirements 格式 |
| 场景驱动测试 | 测试从 Spec Scenarios 出发，验证符合规范 |
| 自动决策 | TBD 问题基于 ./openspec/project.md 自主决策并记录 |
| 审计即执行 | 每完成一个动作立即更新 AUDIT.md |
| 连续执行 | 所有阶段自动流转，无需等待确认 |
| 全绿为准 | 测试不通过不能标记任务完成，禁止取巧 |

## 当前阶段目标

[根据当前步骤填写]

### 步骤 3（提案细化）
- 基于 tasks.md 设计拆解完善提案
- 确保每个 Requirement 都有 Scenario
- 运行 validate --strict 检查

### 步骤 4（评审提案）
- 评审提案与现有体系的兼容性
- 区分 Type A（违规）和 Type B（演进）
- 自动批准后进入步骤 5

### 步骤 5（Apply 施工）
- 严格按任务顺序执行
- 每完成一项测试全绿后打勾
- 所有任务完成后进入步骤 6

## 下一步行动

[按优先级排序的下一步行动清单]

1. [最优先行动]
2. [次要行动]
3. [后续行动]

## 关键文件路径

### 当前 Change 文件
```
./openspec/changes/[change-id]/
├── proposal.md              # 提案文档
├── tasks.md                 # 任务清单
├── STATE_RELOAD.md          # 本文件
├── specs/                   # Delta Spec
├── AUDIT.md                 # 审计日志
└── DECISIONS.md             # 决策日志
```

### 参考 Prompt 文件（技能目录）
```
./spec-runner/references/
├── 00_testing_helper.md          # 测试执行原则
├── 00_STATE_RELOAD_template.md   # 本模板
├── [当前步骤]_prompt.md           # 当前步骤的详细指令
└── [下一步骤]_prompt.md           # 下一步骤的详细指令
```

### 项目规范
```
./openspec/
├── project.md              # 项目规范（用于自动决策）
└── specs/                  # 现有 Capabilities
```

## 压缩恢复指令

**如果你正在读取此文件，说明上下文已被压缩。请立即执行以下步骤：**

1. 读取 `./openspec/changes/[change-id]/AUDIT.md` 获取完整执行历史
2. 读取 `./spec-runner/references/[当前步骤]_prompt.md` 获取详细指令
3. 检查 `./openspec/changes/[change-id]/tasks.md` 确认任务进度
4. 根据"下一步行动"继续执行

## 禁止行为

- 使用 skip/xfail 跳过测试
- 删除或注释失败的测试
- 弱化断言让测试通过
- 延迟记录审计信息
- 等待用户确认（连续执行模式）

## 多 CP 模式特殊说明

如果是批量执行模式：
- 当前处理的是第 [X]/[Y] 个 CP
- 失败后继续下一个 CP
- 最后生成批量执行报告
