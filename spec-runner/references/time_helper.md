# 时间获取工具

## 用途

在 spec-runner 执行过程中，所有需要记录时间的地方，**必须使用本工具提供的命令获取准确时间**，禁止猜测或估算。

## 核心原则

| 原则 | 说明 |
|------|------|
| **命令获取** | 必须使用 `date` 命令获取时间 |
| **禁止猜测** | 不得使用"大约"、"可能"等模糊表述 |
| **立即执行** | 需要时间时立即执行命令，不延迟 |
| **格式统一** | 按照指定格式输出时间 |

## 时间格式命令

### 完整时间戳（推荐）

```bash
date "+%Y-%m-%d %H:%M:%S"
```

**输出示例**: `2026-01-07 14:30:00`

**使用场景**:
- 审计日志中的开始/结束时间
- 步骤详情中的执行时间
- 快照文件中的时间记录

### 仅日期

```bash
date "+%Y-%m-%d"
```

**输出示例**: `2026-01-07`

**使用场景**:
- Change ID 中的日期部分
- 日期范围的记录

### 仅时间

```bash
date "+%H:%M:%S"
```

**输出示例**: `14:30:00`

**使用场景**:
- 时间范围的记录

### Unix 时间戳

```bash
date +%s
```

**输出示例**: `1704609000`

**使用场景**:
- 时间差计算
- 耗时统计

## 耗时计算

### 计算两个时间点的差值

```bash
# 开始时间（记录）
START_TIME=$(date +%s)

# ... 执行操作 ...

# 结束时间
END_TIME=$(date +%s)

# 计算耗时（秒）
ELAPSED=$((END_TIME - START_TIME))

# 转换为分钟（可选）
MINUTES=$((ELAPSED / 60))
```

### 人性化耗时显示

```bash
# 计算耗时并显示为 Xh Ym 格式
function format_duration {
    local seconds=$1
    local hours=$((seconds / 3600))
    local minutes=$(((seconds % 3600) / 60))
    if [ $hours -gt 0 ]; then
        echo "${hours}h ${minutes}m"
    else
        echo "${minutes}m"
    fi
}

# 使用示例
START_TIME=$(date +%s)
# ... 执行操作 ...
END_TIME=$(date +%s)
ELAPSED=$((END_TIME - START_TIME))
echo "耗时: $(format_duration $ELAPSED)"
```

## 在审计日志中的使用

### 步骤开始时

```bash
# 1. 获取当前时间
CURRENT_TIME=$(date "+%Y-%m-%d %H:%M:%S")

# 2. 更新 AUDIT.md 时间线
echo "| 步骤X：XXX | $CURRENT_TIME | | | 进行中 |" >> AUDIT.md

# 3. 记录开始时间戳供后续计算
echo $CURRENT_TIME > .audit/step_x_start.txt
```

### 步骤结束时

```bash
# 1. 获取当前时间
CURRENT_TIME=$(date "+%Y-%m-%d %H:%M:%S")

# 2. 读取开始时间
START_TIME=$(cat .audit/step_x_start.txt)

# 3. 计算耗时
START_SECONDS=$(date -d "$START_TIME" +%s)
END_SECONDS=$(date -d "$CURRENT_TIME" +%s)
ELAPSED_SECONDS=$((END_SECONDS - START_SECONDS))
ELAPSED_MINUTES=$((ELAPSED_SECONDS / 60))

# 4. 更新 AUDIT.md 时间线（需要使用 Edit 工具修改对应行）
# 将：| 步骤X：XXX | $START_TIME | | | 进行中 |
# 改为：| 步骤X：XXX | $START_TIME | $CURRENT_TIME | ${ELAPSED_MINUTES}m | ✓ |
```

## 快捷命令参考

| 需求 | 命令 |
|------|------|
| 当前完整时间 | `date "+%Y-%m-%d %H:%M:%S"` |
| 当前日期 | `date "+%Y-%m-%d"` |
| 当前时间 | `date "+%H:%M:%S"` |
| Unix 时间戳 | `date +%s` |
| 记录开始时间 | `START_TIME=$(date +%s)` |
| 计算耗时（秒） | `echo $(($(date +%s) - START_TIME))` |

## 常见错误

| 错误 | 正确做法 |
|------|----------|
| 写"大约 14:30" | 执行 `date "+%H:%M:%S"` 获取准确时间 |
| 使用记忆中的时间 | 每次需要时都执行 date 命令 |
| 格式不统一 | 使用模板中指定的格式 |
| 忘记记录结束时间 | 步骤开始/结束都记录时间 |

## 示例：完整的审计时间记录流程

```bash
# === 步骤开始 ===
STEP_START_TIME=$(date "+%Y-%m-%d %H:%M:%S")
STEP_START_SECONDS=$(date +%s)

# 更新 AUDIT.md（添加行）
# "| 步骤X：XXX | $STEP_START_TIME | | | 进行中 |"

# ... 执行步骤操作 ...

# === 步骤结束 ===
STEP_END_TIME=$(date "+%Y-%m-%d %H:%M:%S")
STEP_END_SECONDS=$(date +%s)

# 计算耗时
ELAPSED_SECONDS=$((STEP_END_SECONDS - STEP_START_SECONDS))
ELAPSED_MINUTES=$((ELAPSED_SECONDS / 60))

# 更新 AUDIT.md（修改行）
# "| 步骤X：XXX | $STEP_START_TIME | $STEP_END_TIME | ${ELAPSED_MINUTES}m | ✓ |"
```
