# AGENTS.md

> 本文件是 AI Agent 的导航索引。Agent 应在合适的时机读取对应文档，以获取上下文和规范约束。
> 本文件仅作为目录存在，不存放具体规范内容。

---

## OpenSpec 规则 (Mandatory)

> **强制遵循**：Agent 在执行任何涉及系统变更的任务时，必须遵循 OpenSpec 工作规范。
> OpenSpec 是规范驱动开发（SDD）的执行引擎，通过结构化的变更流程确保每个改动可追溯、可审查、可回滚。

### 目录结构

```
openspec/
├── specs/                                  # 系统规格：声明当前系统如何工作
│   └── <domain>-spec.md                    # 按领域拆分的系统行为描述
├── change/                                 # 变更目录
│   ├── <change-name>/                      # 当前正在执行的 change
│   │   ├── specs/                          # 本次变更涉及的 spec 变动
│   │   ├── proposal.md                     # 需求实现提案（拆解需求）
│   │   ├── plan.md                         # 执行方案（怎么实现）
│   │   └── task.md                         # 执行步骤节点（具体任务拆解）
│   └── archive/                            # 已归档的已完成 change
│       └── <change-name>/                  # 保持原始结构
```

### 核心规则

| 规则 | 说明 |
|------|------|
| **Spec 先行** | 修改系统行为前，先更新 `openspec/specs/` 或在 change 的 `specs/` 中声明变动 |
| **Change 驱动** | 任何非 trivial 的改动必须创建一个 change，走 proposal → plan → task 流程 |
| **proposal.md** | 声明本次变更的需求拆解：改什么、为什么、影响范围 |
| **plan.md** | 声明执行方案：技术方案、实现路径、风险点 |
| **task.md** | 声明执行步骤：有序的任务节点列表，每步有明确的完成标准 |
| **specs/** | 声明本次 change 涉及的系统行为变动，完成后合入 `openspec/specs/` |
| **归档闭环** | 所有 task 完成并验证后，将 change 移入 `archive/`，并同步更新 `openspec/specs/` |

### Change 生命周期

```
创建 change 目录
  │
  ▼
编写 proposal.md（需求拆解，明确变更范围）
  │
  ▼
编写 plan.md（技术方案，确认实现路径）
  │
  ▼
编写 task.md（任务拆解，列出执行步骤）
  │
  ▼
编写 specs/（声明涉及的 spec 变动）
  │
  ▼
按 task.md 逐步执行实现
  │
  ▼
全部 task 完成 → 验证通过
  │
  ▼
将 specs/ 合入 openspec/specs/
  │
  ▼
将 change 移入 archive/
```

### Agent 读取时机

| 时机 | 必读路径 |
|------|----------|
| 接到系统变更任务 | `openspec/specs/` → 了解系统当前状态 |
| 创建新 change | `openspec/change/<name>/` 全部文件 |
| 执行实现 | `openspec/change/<name>/task.md` → 按步骤执行 |
| 提交前检查 | `openspec/change/<name>/specs/` → 确认 spec 同步 |
| 日常理解系统 | `openspec/specs/` → 系统完整行为描述 |

### 文件模板引用

| 文件 | 模板位置 |
|------|----------|
| proposal.md | [docs/feature/templates/PRD-template.md](docs/feature/templates/PRD-template.md)（可精简） |
| plan.md | 按 change 实际需要编写 |
| task.md | 按 change 实际需要拆解 |
| specs/*.md | [openspec/specs/](openspec/specs/) 风格一致 |

### 与 Feature SDD 流程的关系

OpenSpec 是 SDD 流程的**执行引擎**：

- `docs/feature/` 的 PRD/BED/FED/TEST 文档定义了**文档规范和模板**
- `openspec/` 是**实际的变更工作区**，每次 Feature 开发对应一个 change
- 两者配合：用 docs 的模板写 openspec 的文件，形成完整闭环

---

## 全局规范 (Global Specifications)

> 读取时机：开发任何代码前，Agent 应先阅读相关全局规范。
> 这些规范是跨 Feature 复用的稳定规则，适用于整个项目。

| 文档 | 描述 | 引用路径 | 读取时机 |
|------|------|----------|----------|
| 编码标准 | 命名规范、TypeScript 规范、不可变性、禁止清单、代码质量检查 | [docs/global/coding-standards.md](docs/global/coding-standards.md) | 编写任何代码时 |
| 架构原则 | 项目分层、模块边界、通信模式、错误边界、ADR 模板 | [docs/global/architecture-principles.md](docs/global/architecture-principles.md) | 设计模块结构、创建新模块时 |
| API 接口规范 | RESTful 设计、统一响应格式、HTTP 客户端封装、Service 层、错误码 | [docs/global/api-conventions.md](docs/global/api-conventions.md) | 设计或实现后端接口时 |
| 测试标准 | TDD 流程、测试分层、单元/集成/E2E 测试规范、Mock 策略 | [docs/global/testing-standards.md](docs/global/testing-standards.md) | 编写任何测试时 |
| 安全指南 | 密钥管理、输入验证、XSS/CSRF 防护、认证授权、安全响应协议 | [docs/global/security-guidelines.md](docs/global/security-guidelines.md) | 涉及认证/授权/用户输入时 |
| Git 工作流 | 分支策略、Commit 规范、PR 流程、Feature 开发流程 | [docs/global/git-workflow.md](docs/global/git-workflow.md) | 创建分支、提交代码、创建 PR 时 |
| 错误处理 | 错误分类、四层处理模式、异步错误处理、全局兜底 | [docs/global/error-handling.md](docs/global/error-handling.md) | 设计异常场景、编写 try/catch 时 |
| 状态管理 | 状态分层、Pinia Store 规范、Composable 数据获取、不可变更新 | [docs/global/state-management.md](docs/global/state-management.md) | 设计前端状态、创建 Store 时 |
| 组件模式 | 组件结构、分类、受控组件、插槽组合、加载/错误/空状态 | [docs/global/component-patterns.md](docs/global/component-patterns.md) | 设计或实现 Vue 组件时 |
| 数据库规范 | 表/字段命名、表结构模板、数据类型、查询规范、迁移、索引 | [docs/global/database-conventions.md](docs/global/database-conventions.md) | 设计数据模型、编写 SQL 时 |
| 评审清单 | 代码评审维度、PR 评审流程、Conventional Comments、完成标准 | [docs/global/review-checklist.md](docs/global/review-checklist.md) | 提交代码评审时 |
| **过程文件管理** | **tmp/ 目录规范、Agent 强约束、提交前清理检查、文件归属判定** | [docs/global/process-file-management.md](docs/global/process-file-management.md) | **每次 Agent 执行任务时、Git 提交前** |

---

## Feature 开发 (Feature Development)

> 读取时机：开始一个新 Feature 开发时，按 PRD → BED → FED → TEST 顺序阅读和编写。

| 文档 | 描述 | 引用路径 | 读取时机 |
|------|------|----------|----------|
| Feature 开发指南 | 四层文档体系说明、Feature 命名规范、使用方法 | [docs/feature/README.md](docs/feature/README.md) | 开始新 Feature 时首先阅读 |
| 闭环工作流 | PRD→BED→FED→TEST 完整流程、各阶段输入/输出/评审、Agent 协作指引 | [docs/feature/workflow.md](docs/feature/workflow.md) | 规划 Feature 开发流程时 |
| PRD 模板 | 产品需求文档模板：业务目标、用户场景、功能范围、业务规则、验收标准 | [docs/feature/templates/PRD-template.md](docs/feature/templates/PRD-template.md) | 编写 Feature 需求文档时 |
| BED 模板 | 后端实现设计模板：API 接口、数据模型、服务逻辑、约束、单元测试计划 | [docs/feature/templates/BED-template.md](docs/feature/templates/BED-template.md) | 设计 Feature 后端实现时 |
| FED 模板 | 前端实现设计模板：页面结构、交互设计、状态流转、联动逻辑、性能 | [docs/feature/templates/FED-template.md](docs/feature/templates/FED-template.md) | 设计 Feature 前端实现时 |
| TEST 模板 | 测试验证文档模板：验收测试、单元测试审查、集成测试、回归评估 | [docs/feature/templates/TEST-template.md](docs/feature/templates/TEST-template.md) | Feature 实现完成后验证时 |

---

## Agent 行为指引

### 强制约束（每次执行时必须遵循）

1. **过程文件归 tmp/**：所有非最终交付物（分析报告、计划、临时脚本、调试产物）必须写入 `tmp/` 目录，不得散落在项目其他位置。详见 [process-file-management.md](docs/global/process-file-management.md)。

2. **Git 提交前强制检查**：每次 `git commit` 或 `git push` 前，必须执行以下检查：
   - 扫描 `git status`，确认暂存区无过程文件
   - 确认 `tmp/`、`.env`、密钥文件、日志文件未被暂存
   - 如发现非必要文件已暂存，执行 `git reset HEAD <file>` 移除
   - 散落在项目中的过程文件移动到 `tmp/` 或删除
   - 确认 `.gitignore` 覆盖了 `tmp/` 目录

3. **提交前清理**：确认上述检查全部通过后，方可执行提交。

### 开发新 Feature 时的阅读顺序

```
1. 本文件（AGENTS.md） → 了解有哪些文档可用
2. docs/feature/workflow.md → 了解完整闭环流程
3. 按需读取全局规范 → 根据具体任务选择
4. 按 PRD → BED → FED → TEST 顺序推进
```

### 常见场景的文档选择

| 场景 | 必读文档 |
|------|----------|
| 开始新 Feature | workflow.md → PRD-template.md |
| 设计后端 API | api-conventions.md → database-conventions.md → BED-template.md |
| 设计前端页面 | component-patterns.md → state-management.md → FED-template.md |
| 编写测试 | testing-standards.md → TEST-template.md |
| 提交代码 | git-workflow.md → review-checklist.md → process-file-management.md |
| 安全相关 | security-guidelines.md |
| 错误处理 | error-handling.md |

### 文档更新规则

- 全局规范变更需团队评审后修改
- Feature 文档随开发进度同步更新
- 模板变更需同步更新 AGENTS.md 索引
- 新增文档必须在 AGENTS.md 中注册引用
