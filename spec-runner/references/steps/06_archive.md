# 步骤 6：归档

## 使用时机

单 CP 执行模式的第 6 步。在步骤 5（Apply施工）完成后自动进入。

## 步骤 6：全量测试验证（强制执行，不得跳过）

**【重要】本步骤必须执行全量测试，不得跳过或部分执行。**

### 测试检测与执行（必须按顺序执行）

**第 1 步：检测项目测试结构**

```bash
# 1. 检测是否有 pytest 测试
ls -la tests/ 2>/dev/null || echo "无 tests/ 目录"
find . -name "*_test.py" -o -name "test_*.py" | head -5

# 2. 检测是否有 e2e 测试（常见位置）
ls -la tests/e2e/ 2>/dev/null || ls -la e2e/ 2>/dev/null || ls -la e2e-tests/ 2>/dev/null || echo "未找到 e2e 目录"

# 3. 检测 package.json 是否有测试脚本
cat package.json 2>/dev/null | grep -A 5 '"test"'
```

**第 2 步：执行全量测试（必须全部执行）**

```bash
# A. 运行所有 pytest 测试（如果有）
pytest -v                         # Python 项目全量测试
# 或
pytest tests/ -v                  # 指定 tests 目录

# B. 运行 e2e 测试（如果存在）
pytest tests/e2e/ -v              # e2e 在 tests/e2e/
# 或
pytest e2e/ -v                    # e2e 在独立目录
# 或
npm run test:e2e                  # npm 项目的 e2e

# C. 运行覆盖率测试（可选但推荐）
pytest --cov=src --cov-report=term-missing
```

**第 3 步：确认测试全绿**

```bash
# 确认所有测试通过，输出测试统计
pytest -v --tb=short
# 必须看到 "X passed in Y.ZZs" 无任何 failed
```

### 全量测试强制要求（必须遵守）

| 要求 | 说明 |
|------|------|
| **必须执行** | 全量测试是强制步骤，不得跳过 |
| **全部运行** | 必须运行项目中所有类型测试（pytest + e2e） |
| **全绿才能继续** | 只有所有测试全绿才能进入归档步骤 |
| **禁止取巧** | 不得使用 skip、xfail、删除测试等方式让测试通过 |
| **最多重试 3 次** | 测试失败时修复后重测，最多 3 次 |

### 测试失败处理流程

```
测试失败
    ↓
根因分析（代码问题 vs 测试问题 vs 环境问题）
    ↓
修复对应问题
    ↓
重新运行全量测试
    ↓
最多重试 3 次 → 仍失败则记录到 AUDIT.md 并停止
```

**【重要】只有全量测试全绿后，才能继续下面的归档检查点。**

- **测试全绿** → 进入**归档前检查**，不要停下等待
- **测试失败** → 修复后重新测试，最多 3 次重试

## 归档前检查

**参考**：详见 `references/guides/testing.md` 的修改问题原则。

### 检查点 1：测试覆盖

检查本次提案中是否所有 Spec Scenarios 都有对应测试，如有缺失自动补充：
- 检查 tasks.md 中定义的测试场景是否全部实现
- 如有缺失，自动补充测试用例
- 不要等待用户确认

## 检查点 2：全量测试

在本仓库执行全量测试，并将所有失败修复到全绿：

### 全量测试要求（必须遵循）

1. **运行所有测试**：包括新增测试和现有测试
2. **禁止取巧**：不得通过以下方式让测试通过
   - 使用 skip/xfail 跳过失败的测试
   - 删除或注释失败的测试用例
   - 弱化断言（如把 assertEqual 改为 assertIn）
3. **根因分析**：对每个失败先做根因分析，再修复
4. **修复原则**：
   - 本次提案引入的问题：必须修复
   - 本次提案无关的失败：记录到 AUDIT.md，继续或修复

### 测试失败处理

| 失败类型 | 处理方式 | 禁止行为 |
|----------|----------|----------|
| 代码 bug | 修复代码 | 禁止修改测试让测试通过 |
| 测试 bug | 修复测试 | 禁止删除测试用例 |
| Spec 不一致 | 更新 Spec 或代码协商 | 禁止忽略不一致 |
| 环境问题 | 修复环境后重测 | 禁止跳过测试 |

## 检查点 3：确认与归档

确认 tasks 全部完成（如有未完成，记录到 AUDIT.md 并继续）：
2. 自动确认 spec delta 已合并且与实现一致
3. 自动确认所有 Spec Scenarios 都有测试覆盖
4. 如有遗留项，必须显式记录为后续 change
5. 执行 openspec archive <change-id> --yes
6. 归档完成后 → 进入步骤 7（Git 提交），不要停下等待
```

### 归档命令（严格执行）

```bash
# 1. 获取当前时间（审计用）
date "+%Y-%m-%d %H:%M:%S"

# 2. 验证 Change ID
openspec list

# 3. 执行归档
openspec archive <change-id> --yes

# 4. 验证归档结果
openspec validate --strict
openspec show <change-id>
```

### 归档命名规范

归档后的目录将被自动重命名为：`YYYY-MM-DD-[change-name]`
- 这是 OpenSpec 自动处理的
- 对应归档位置：`openspec/changes/archive/YYYY-MM-DD-[change-name]/`

### 归档完成后的行为

- **单 CP 执行模式**：归档完成后 → 进入步骤 7（Git 提交），不要停下等待确认
- **多 CP 批量执行**：归档完成后 → 进入步骤 7（Git 提交）→ 继续处理下一个 CP

## 全量测试结果记录

```markdown
### 全量测试结果

**测试命令**: pytest / npm test
**执行时间**: YYYY-MM-DD HH:MM:SS
**测试结果**: ✓ 全绿

#### 统计
- Spec Scenarios 覆盖: N/N（100%）
- 新增测试场景: M 个（来自 Scenarios: N 个，补充边界: K 个）
- 新增测试用例: X 条
- 修改测试用例: Y 条
- 单元测试通过: X/X
- 集成测试通过: Y/Y
- 全量测试通过: T/T
- 代码覆盖率: XX%

#### 测试失败修复（如有）
- 失败1: [描述] → 修复方式: [描述]
- 失败2: [描述] → 修复方式: [描述]
```

## 审计日志要求

**步骤 6 开始时**：在 AUDIT.md 时间线中添加

```markdown
| 步骤6：归档 | [开始时间] | | | 进行中 |
```

**归档检查点执行时**：
1. 获取当前时间：`date "+%Y-%m-%d %H:%M:%S"`
2. 更新 AUDIT.md 记录检查结果
3. 如有问题，记录到 AUDIT.md 遗留问题区域

**步骤 6 完成时**：更新 AUDIT.md 时间线（记录结束时间和耗时）
