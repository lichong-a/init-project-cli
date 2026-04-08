# Git 工作流 (Git Workflow)

> 适用范围：全局规范 | 稳定规则 | 跨 Feature 复用

## 分支策略

```
main (生产)
  └── develop (开发)
        ├── feature/PRD-001-user-auth   # Feature 分支
        ├── feature/PRD-002-dashboard   # Feature 分支
        ├── fix/BUG-001-login-error     # Bug 修复分支
        └── refactor/RFC-001-api-layer  # 重构分支
```

### 分支命名规范

| 类型 | 格式 | 示例 |
|------|------|------|
| 功能 | `feature/PRD-XXX-简述` | `feature/PRD-001-user-auth` |
| 修复 | `fix/BUG-XXX-简述` | `fix/BUG-001-login-error` |
| 重构 | `refactor/RFC-XXX-简述` | `refactor/RFC-001-api-layer` |
| 热修 | `hotfix/XXX-简述` | `hotfix/001-payment-crash` |

## Commit 规范

### 格式

```
<type>: <description>

[可选 body - 详细说明]
```

### Type 列表

| Type | 用途 | 示例 |
|------|------|------|
| `feat` | 新功能 | `feat: 新增用户登录功能` |
| `fix` | Bug 修复 | `fix: 修复登录 Token 过期未刷新` |
| `refactor` | 重构 | `refactor: 重构 HTTP 客户端错误处理` |
| `docs` | 文档 | `docs: 更新 API 接口文档` |
| `test` | 测试 | `test: 补充用户模块单元测试` |
| `chore` | 构建/工具 | `chore: 升级 Vite 到 v6` |
| `perf` | 性能 | `perf: 优化列表渲染性能` |
| `ci` | CI/CD | `ci: 添加 GitHub Actions 构建流程` |

### Commit 最佳实践

```bash
# 好的 commit：说明做了什么以及为什么
feat: 新增用户注册邮箱验证功能

注册后发送验证邮件，24小时内未验证则冻结账户。
关联 PRD: PRD-001

# 不好的 commit：过于笼统
fix: 修了一些 bug

# 不好的 commit：包含多个不相关改动
feat: 新增登录 + 修复样式 + 重构数据库
```

## PR 工作流

### 创建 PR 前检查

- [ ] 所有测试通过
- [ ] 代码已自审
- [ ] 无硬编码值
- [ ] 无 console.log
- [ ] 新增代码有对应测试

### PR 模板

```markdown
## 概要
[1-3 句话说明改了什么]

## 改动范围
- [ ] 后端 API
- [ ] 前端页面
- [ ] 数据库变更
- [ ] 测试用例

## 关联文档
- PRD: [链接或文档路径]
- BED: [链接或文档路径]
- FED: [链接或文档路径]

## 测试计划
- [ ] 单元测试通过
- [ ] 集成测试通过
- [ ] E2E 测试通过
- [ ] 手动验证通过

## 截图/录屏
[如有 UI 变动，附截图]
```

### PR Review 规范

1. **自审**：提交 PR 前先审一遍自己的 diff
2. **Agent 审**：使用 `code-reviewer` agent 自动审查
3. **人工审**：至少 1 人 Review
4. **安全审**：涉及敏感操作时使用 `security-reviewer`

## Feature 开发流程

```
1. 从 develop 创建 feature 分支
   git checkout develop
   git pull
   git checkout -b feature/PRD-001-user-auth

2. 开发（遵循 TDD）
   - 写测试 → 写实现 → 重构
   - 每完成一个小功能就 commit

3. 提交 PR 前合并最新 develop
   git fetch origin
   git rebase origin/develop

4. 创建 PR
   git push -u origin feature/PRD-001-user-auth
   # 在 GitHub/GitLab 上创建 PR

5. Code Review 通过后合并
   - Squash Merge 到 develop
   - 删除 feature 分支
```

## 危险操作警告

| 操作 | 风险 | 建议 |
|------|------|------|
| `git push --force` | 覆盖远程历史 | 禁止在 main/develop 上使用 |
| `git reset --hard` | 丢失未提交改动 | 先 `git stash` 备份 |
| `git rebase` 公共分支 | 改写公共历史 | 仅在 feature 分支使用 |
| `--no-verify` | 跳过钩子检查 | 禁止使用 |
