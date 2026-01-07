# 步骤 2：设计拆解

## 使用时机

单 CP 执行模式的第 2 步。在步骤 1（创建提案）完成后自动进入。

## Prompt 模板

```
在不改变 proposal 范围的前提下，把 tasks.md 拆到 0.5–1 天/任务。

## 每条任务必须包含

1. **目的**：这条任务要达成什么
2. **来源 Spec**：对应 Delta Spec 中的 Requirement
3. **改动文件/模块**：具体涉及哪些文件或模块
4. **关键实现点**：实现的核心逻辑
5. **测试场景**：从 Spec Scenarios 映射的测试场景
6. **对应的验收/测试**：如何验证任务完成

## 额外要求

- 标注任务之间的依赖关系
- 标注错误处理方式
- 确保每条任务可独立验收
- 不能完成就重拆
```

## 测试场景设计（场景驱动测试）

**参考**：详见 `references/guides/testing.md` 的测试设计原则。

### 从 Spec Scenarios 映射测试场景

每个任务必须根据 Delta Spec 的 Scenarios 设计测试场景：

```markdown
- [ ] ### 任务 X：[任务名称]

**目的**: [实现目的]

**来源 Spec**: Delta Spec > ADDED/MODIFIED/REMOVED > [Requirement 名称]

**改动文件**: [代码文件], [测试文件]

**测试场景**（从 Scenarios 映射）:
#### Spec Scenarios（必须覆盖）
- [Spec 场景1]: [测试描述] → [预期结果]
- [Spec 场景2]: [测试描述] → [预期结果]

#### 补充场景（边界/异常）
- [边界场景1]: [测试描述] → [预期结果]
- [异常场景1]: [测试描述] → [预期结果]

**关键实现点**: [关键点]

**验收标准**: 所有测试场景通过
```

### 测试场景设计原则

| 原则 | 说明 |
|------|------|
| **必须覆盖** | 所有 Spec Scenarios 必须有对应测试场景 |
| **补充边界** | 根据需要补充边界、异常场景 |
| **从功能出发** | 测试场景描述功能行为，不涉及实现细节 |
| **可测试性** | 每个场景可独立验证 |

### 示例

```markdown
- [ ] ### 任务 1：实现用户登录

**目的**: 实现"用户登录"Requirement

**来源 Spec**: Delta Spec > ADDED > 用户登录

**改动文件**: src/auth.py, tests/test_auth_login.py

**测试场景**（从 Scenarios 映射）:
#### Spec Scenarios（必须覆盖）
- 正常登录：valid email + valid password → 返回 JWT token
- 密码错误：valid email + wrong password → 返回 401
- 用户不存在：non-existent email → 返回 404

#### 补充场景（边界/异常）
- 邮箱为空：empty email → 返回 400
- 密码为空：empty password → 返回 400
- 邮箱格式无效：invalid email format → 返回 400
- SQL 注入：sql injection attempt → 返回 400

**关键实现点**: JWT token 生成、密码验证

**验收标准**: 所有测试场景通过
```

## 输出要求

更新 `changes/[change-id]/tasks.md` 文件。

- 每条任务必须包含测试场景设计
- 测试场景从 Spec Scenarios 映射而来
- 完成后 → 进入**步骤 3：3 道关卡打磨**，不要停下等待确认

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

在 AUDIT.md 时间线中添加

```markdown
| 步骤2：设计拆解 | [使用 date 命令获取] | | | 进行中 |
```

### tasks.md 更新后（立即执行）

```bash
# 1. 获取时间
TIME=$(date "+%Y-%m-%d %H:%M:%S")

# 2. 立即创建 .audit/ 快照 002_tasks_breakdown.md
# 3. 更新文件变更索引
```

### 步骤结束时

```bash
# 1. 获取结束时间
END_TIME=$(date "+%Y-%m-%d %H:%M:%S")

# 2. 计算耗时
# 3. 更新 AUDIT.md 时间线（记录结束时间和耗时）
```

**禁止延迟记录或批量记录审计信息。**
