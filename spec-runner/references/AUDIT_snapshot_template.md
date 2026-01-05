# .audit 快照模板

.audit/ 目录中的每个快照文件记录一次文件变更。

---

## 快照命名规则

`NNN_step_description.md`

- NNN: 3位递增序号 (001, 002, 003, ...)
- step_description: 步骤描述

---

## 快照文件模板

```markdown
# 快照 NNN: [步骤描述]

**时间**: YYYY-MM-DD HH:MM:SS
**步骤**: [步骤名称]
**操作**: admin (Claude)
**文件**: [文件名]
**变更类型**: 创建 / 修改 / 删除

## 变更原因

[说明为什么进行这次变更]

## 变更内容

\`\`\`diff
[diff 内容]
+ 新增内容
- 删除内容
~ 修改内容
\`\`\`

## 影响分析

- [ ] 影响其他文件
- [ ] 影响后续任务
- [ ] 无影响

## 关联决策

- [TBD-X]: [相关决策](../DECISIONS.md#tbd-x)
```

---

## 示例

```markdown
# 快照 001: initial_proposal

**时间**: 2026-01-05 10:15:00
**步骤**: 创建提案
**操作**: admin (Claude)
**文件**: proposal.md
**变更类型**: 创建

## 变更原因

初始化 change proposal，定义用户认证功能的范围和验收标准。

## 变更内容

\`\`\`diff
+ # Change Proposal: 用户认证功能
+
+ ## Scope
+ - 用户注册（邮箱验证）
+ - 用户登录（JWT Token）
+ - 密码重置
+
+ ## Out of Scope
+ - 第三方登录（OAuth）
+ - 多因素认证（MFA）
+ - 手机号登录
+
+ ## Acceptance Criteria
+ - 用户可使用邮箱+密码注册
+ - 注册后发送验证邮件
+ - 用户可登录获取 JWT Token
+ - Token 有效期 24 小时
+
+ ## Data / API 接触面
+ - users 表 (id, email, password_hash, verified_at)
+ - POST /api/auth/register
+ - POST /api/auth/login
+ - POST /api/auth/verify
+
+ ## Risks & Mitigations
+ - 风险：密码泄露 → 缓解：bcrypt 哈希
+ - 风险：暴力破解 → 缓解：登录限流
+
+ ## Rollout / Rollback
+ - Rollout: 灰度发布 10% → 50% → 100%
+ - Rollback: 下线认证服务，回滚数据库迁移
\`\`\`

## 影响分析

- [x] 影响其他文件: 需创建 User 模型、Auth 控制器
- [x] 影响后续任务: 需要先完成数据库迁移
- [ ] 无影响

## 关联决策

- [TBD-1]: 选择 JWT 而非 Session（见 DECISIONS.md#tbd-1）
```
