# 测试辅助指南 - 场景驱动测试

## 定位

**规范驱动 + 场景驱动测试**

- Delta Spec 定义功能和场景（驱动源）
- 根据 Scenarios 设计测试场景
- 实现代码后，用测试验证符合规范

## 与 TDD 的区别

| 阶段 | TDD 纯粹派 | 规范驱动 + 场景测试 |
|------|-----------|---------------------|
| 驱动源 | 测试用例 | Delta Spec (Requirements + Scenarios) |
| 测试设计 | 先写测试（Red） | 根据 Scenarios 设计测试场景 |
| 实现 | 测试驱动代码 | 规范驱动代码 |
| 测试作用 | 驱动设计 | 验证符合规范 |

## 从 Scenarios 到测试场景的映射

### Delta Spec 示例

```markdown
## ADDED Requirements

### Requirement: 用户登录
The system SHALL authenticate users with email and password.

#### Scenario: 正常登录
- **WHEN** user provides valid email and password
- **THEN** system returns JWT token

#### Scenario: 密码错误
- **WHEN** user provides valid email but wrong password
- **THEN** system returns 401 Unauthorized

#### Scenario: 用户不存在
- **WHEN** user provides non-existent email
- **THEN** system returns 404 Not Found
```

### 映射到测试场景

| Delta Spec Scenario | 测试场景 | 测试用例名 |
|---------------------|----------|-----------|
| 正常登录 | 正常登录场景 | test_login_success |
| 密码错误 | 密码错误场景 | test_login_wrong_password |
| 用户不存在 | 用户不存在场景 | test_login_user_not_found |
| （补充边界） | 邮箱格式错误 | test_login_invalid_email |
| （补充边界） | 空值处理 | test_login_empty_fields |
| （补充边界） | SQL 注入防护 | test_login_sql_injection |

### 映射原则

1. **必须覆盖** - 所有 Spec Scenarios 必须有对应测试场景
2. **补充边界** - 根据需要补充边界、异常场景
3. **从功能出发** - 测试场景描述功能行为，不涉及实现细节
4. **可测试性** - 每个场景可独立验证

## 测试场景设计模板

### 单功能测试场景

```markdown
### 功能：[功能名称]

#### Spec Scenarios（必须覆盖）
| Scenario | 测试场景 | 用例名 | 预期结果 |
|----------|----------|--------|----------|
| [Spec 中的场景描述] | [对应测试场景] | test_xxx | [预期结果] |

#### 补充场景（边界/异常）
| 场景类型 | 测试场景 | 用例名 | 预期结果 |
|----------|----------|--------|----------|
| 边界 | [边界场景] | test_xxx_edge | [预期结果] |
| 异常 | [异常场景] | test_xxx_exception | [预期结果] |
```

### 示例：用户登录

```markdown
### 功能：用户登录

#### Spec Scenarios（必须覆盖）
| Scenario | 测试场景 | 用例名 | 预期结果 |
|----------|----------|--------|----------|
| 正常登录 | 有效邮箱+密码 | test_login_success | 返回 200 + JWT token |
| 密码错误 | 有效邮箱+错误密码 | test_login_wrong_password | 返回 401 |
| 用户不存在 | 不存在的邮箱 | test_login_user_not_found | 返回 404 |

#### 补充场景（边界/异常）
| 场景类型 | 测试场景 | 用例名 | 预期结果 |
|----------|----------|--------|----------|
| 边界 | 邮箱为空 | test_login_empty_email | 返回 400 |
| 边界 | 密码为空 | test_login_empty_password | 返回 400 |
| 边界 | 邮箱格式无效 | test_login_invalid_email | 返回 400 |
| 异常 | SQL 注入尝试 | test_login_sql_injection | 返回 400 |
| 异常 | 超长输入 | test_login_too_long_input | 返回 400 |
```

## 测试命令参考

### Python (pytest)

```bash
# 运行所有测试
pytest

# 运行指定测试文件
pytest tests/test_auth_login.py

# 运行指定测试用例
pytest tests/test_auth_login.py::test_login_success

# 详细输出
pytest -v

# 显示打印输出
pytest -s

# 覆盖率报告
pytest --cov=src --cov-report=term-missing

# 失败时停止
pytest -x

# 重新运行失败的测试
pytest --lf
```

### JavaScript/TypeScript (npm/jest)

```bash
# 运行所有测试
npm test

# 运行指定测试文件
npm test -- auth/login.test.js

# 监听模式
npm test -- --watch

# 覆盖率报告
npm test -- --coverage

# 显示详细输出
npm test -- --verbose
```

## 测试结果记录格式

### 单任务测试记录

```markdown
### 任务 N：[任务名称]

**测试场景**:
- 场景1: [描述] → ✓
- 场景2: [描述] → ✓
- 场景3: [描述] → ✓

**测试文件**: tests/xxx_test.py
**测试命令**: pytest tests/xxx_test.py -v
**测试结果**: ✓ 全绿（X/X 通过）
**覆盖 Scenarios**: N/N（补充边界场景 M 个）
**代码覆盖**: XX%
```

### 全量测试记录

```markdown
### 全量测试结果

**测试命令**: pytest / npm test
**执行时间**: YYYY-MM-DD HH:MM:SS
**测试结果**: ✓ 全绿

#### 统计
- Spec Scenarios 覆盖: 10/10（100%）
- 新增测试场景: 15 个（来自 Scenarios: 10 个，补充边界: 5 个）
- 新增测试用例: 42 条
- 修改测试用例: 5 条
- 单元测试通过: 42/42
- 集成测试通过: 8/8
- 全量测试通过: 186/186
- 代码覆盖率: 87%

#### 测试失败修复（如有）
- 失败1: [描述] → 修复方式: [描述]
- 失败2: [描述] → 修复方式: [描述]
```

## 测试失败处理原则

| 失败类型 | 处理方式 | 禁止行为 |
|----------|----------|----------|
| 代码 bug | 修复代码 | 禁止修改测试让测试通过 |
| 测试 bug | 修复测试 | 禁止删除测试用例 |
| Spec 不一致 | 更新 Spec 或代码协商 | 禁止忽略不一致 |
| 环境问题 | 修复环境后重测 | 禁止跳过测试 |

**禁止行为**：
- 使用 `@pytest.mark.skip` 或 `test.skip()` 跳过失败的测试
- 使用 `@pytest.mark.xfail` 标记预期失败
- 删除或注释失败的测试用例
- 弱化断言（如把 `assertEqual` 改为 `assertIn`）
- 修改测试让测试通过（除非测试本身有 bug）

## 测试覆盖率建议

| 覆盖率类型 | 建议值 | 说明 |
|-----------|--------|------|
| Scenario 覆盖 | 100% | 所有 Spec Scenarios 必须有测试 |
| 代码行覆盖 | ≥80% | 核心代码 ≥90% |
| 分支覆盖 | ≥75% | 核心逻辑 ≥85% |

## 常见问题

### Q: Spec Scenario 很简单，还需要拆分多个测试用例吗？

A: 建议拆分。一个 Scenario 可以对应多个测试用例，覆盖不同输入组合。

### Q: 补充边界场景要加多少？

A: 根据功能复杂度决定。简单的 2-3 个，复杂的 5-10 个。目标是覆盖常见边界和异常。

### Q: 测试失败了但功能正常，怎么办？

A: 检查测试是否正确反映了 Spec。如果测试写错了，修改测试；如果 Spec 需要更新，更新 Spec 并同步测试。

### Q: 老代码没有测试，需要补吗？

A: 本次修改的代码必须配套测试。未修改的老代码不强求，但建议逐步补充。
