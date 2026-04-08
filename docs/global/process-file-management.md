# 过程文件管理规范 (Process File Management)

> 适用范围：全局规范 | 强制执行 | Agent 行为约束

## 核心规则

**所有过程文件必须放在项目 `tmp/` 目录下，不得散落在项目其他位置。**

> 过程文件（Process Files）是指开发过程中产生的临时性、中间性产物，
> 不属于最终交付物的一部分，不应被提交到版本控制中。

## 什么是过程文件

| 分类 | 示例 |
|------|------|
| AI 生成产物 | Agent 分析报告、计划文件、中间输出 |
| 调试产物 | 日志文件、性能分析报告、内存快照 |
| 构建中间物 | 编译缓存、打包中间文件、代码生成产物 |
| 测试产物 | 测试报告、覆盖率原始数据、截图/录屏 |
| 临时脚本 | 一次性迁移脚本、数据清洗脚本、对比脚本 |
| 临时配置 | 本地环境配置、调试配置、覆盖配置 |
| 文档草稿 | 文档的临时版本、评审意见收集 |

## 目录结构

```
tmp/
├── ai/                    # AI Agent 过程文件
│   ├── plans/             # 实现计划
│   ├── analysis/          # 代码分析报告
│   ├── reviews/           # 审查报告
│   └── scratch/           # 临时草稿
├── debug/                 # 调试产物
│   ├── logs/              # 调试日志
│   ├── profiles/          # 性能分析
│   └── snapshots/         # 内存快照
├── build/                 # 构建中间物
│   ├── generated/         # 代码生成产物
│   └── cache/             # 编译缓存
├── test/                  # 测试产物
│   ├── reports/           # 测试报告
│   ├── coverage/          # 覆盖率数据
│   ├── screenshots/       # 测试截图
│   └── traces/            # E2E trace
├── scripts/               # 临时脚本
└── drafts/                # 文档草稿
```

## Agent 强约束规则

### 规则一：过程文件写入 tmp/

```
Agent 在执行任务时产生的所有非最终交付物，必须写入 tmp/ 目录。
```

**正确做法**：
- 分析报告 → `tmp/ai/analysis/report-xxx.md`
- 实现计划 → `tmp/ai/plans/plan-xxx.md`
- 测试截图 → `tmp/test/screenshots/xxx.png`
- 临时脚本 → `tmp/scripts/xxx.sh`

**错误做法**：
- 分析报告 → `analysis/report.md`（散落在项目根目录）
- 临时文件 → `temp-xxx.txt`（随意命名）
- 调试日志 → `debug.log`（放在源码旁）

### 规则二：Git 提交前强制检查

```
Agent 在执行 git commit 或 git push 前，必须执行提交前清理检查。
```

**清理检查流程**：

1. **扫描 tmp/ 目录**：确认所有过程文件都在 tmp/ 内
2. **扫描项目根目录和源码目录**：发现散落的过程文件，移动到 tmp/ 或删除
3. **检查 .gitignore**：确认 tmp/ 已被忽略
4. **检查暂存区**：确认没有过程文件被 `git add`
5. **确认 .env 等敏感文件**：不在暂存区中

### 规则三：提交前清理脚本

Agent 在每次提交前应执行以下检查清单：

```markdown
## Git 提交前检查清单（Agent 强制执行）

- [ ] 扫描 `git status`，确认无过程文件被暂存
- [ ] 确认 `tmp/` 目录下的文件未被 `git add`
- [ ] 确认无 `.env`、密钥文件被暂存
- [ ] 确认无 `*.log` 文件被暂存
- [ ] 确认无 `node_modules/` 被暂存
- [ ] 确认无 AI 产物（`.claude/`、`.aider*`）被暂存
- [ ] 如发现非必要文件在暂存区，执行 `git reset HEAD <file>` 移除
- [ ] 对于已散落在项目中的过程文件，移动到 `tmp/` 或删除
```

### 规则四：Agent 生成文件的归属判定

| 文件性质 | 归属 | 是否提交 |
|----------|------|----------|
| 源码文件（.ts/.vue/.py 等） | `src/` 或对应模块 | 是 |
| 类型定义（.d.ts） | 对应模块 | 是 |
| 配置文件（vite.config.ts 等） | 项目根目录 | 是 |
| 测试文件（.test.ts/.spec.ts） | 对应模块测试目录 | 是 |
| 实现计划文档 | `tmp/ai/plans/` | 否 |
| 代码分析报告 | `tmp/ai/analysis/` | 否 |
| 安全审查报告 | `tmp/ai/reviews/` | 否 |
| 调试日志 | `tmp/debug/logs/` | 否 |
| 测试截图 | `tmp/test/screenshots/` | 否 |
| 临时迁移脚本 | `tmp/scripts/` | 否 |

### 规则五：文件命名规范

```
tmp/ 下的文件命名格式：
  {类型}-{Feature编号/简述}-{日期}-{序号}.{扩展名}

示例：
  tmp/ai/plans/plan-PRD001-auth-20260408-01.md
  tmp/ai/reviews/review-security-20260408-01.md
  tmp/test/screenshots/login-flow-20260408-01.png
  tmp/scripts/migrate-users-20260408-01.sh
```

## .gitignore 配合

确保 `.gitignore` 包含以下条目：

```gitignore
# Process files
tmp/
temp/
.tmp/
.temp/
*.tmp
*.temp
*.bak
```

## 违规处理

当 Agent 发现过程文件散落在项目非 tmp/ 位置时：

1. **非暂存文件**：移动到 `tmp/` 对应子目录
2. **已暂存文件**：`git reset HEAD <file>` 后移动到 `tmp/`
3. **不确定是否为过程文件**：记录到 `tmp/ai/scratch/uncertain-files.md`，由人工判断

## 检查清单

- [ ] 所有过程文件在 `tmp/` 目录下
- [ ] `tmp/` 已被 `.gitignore` 忽略
- [ ] Git 提交前执行了清理检查
- [ ] 暂存区无过程文件
- [ ] 无敏感文件被暂存
- [ ] 文件命名遵循规范格式
