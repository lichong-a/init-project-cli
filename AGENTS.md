# AGENTS.md

> 本文件是 AI Agent 的导航索引。Agent 应在合适的时机读取对应文档，以获取上下文和规范约束。
> 本文件仅作为目录存在，不存放具体规范内容。

---

## OpenSpec 规则 (Mandatory)

> **强制遵循**：Agent 在执行任何涉及系统变更的任务时，必须遵循 OpenSpec 工作规范。
> 参考来源：[Fission-AI/OpenSpec](https://github.com/Fission-AI/OpenSpec)

### 前置条件检查（Agent 每次会话启动时必须执行）

Agent 在开始工作前，必须检查以下前置条件是否满足：

```bash
# 1. 检查 Node.js 版本（需要 >= 20.19.0）
node --version

# 2. 检查 OpenSpec CLI 是否已安装
openspec --version

# 3. 如果未安装，执行全局安装
npm install -g @fission-ai/openspec@latest

# 4. 检查项目是否已初始化 OpenSpec
ls openspec/

# 5. 如果未初始化，执行初始化
openspec init

# 6. 确保 agent instructions 是最新的
openspec update
```

**检查清单**：

- [ ] Node.js >= 20.19.0
- [ ] `openspec` CLI 全局可用
- [ ] 项目 `openspec/` 目录已初始化
- [ ] Agent 指令已通过 `openspec update` 更新到最新

### 目录结构

```
openspec/
├── specs/                                  # 系统规格（Source of Truth）：声明当前系统如何工作
│   ├── README.md                           # Specs 编写说明
│   └── <domain>/                           # 按领域拆分的系统行为描述
│       └── spec.md                         # 含 Requirements + Scenarios
├── change/                                 # 变更目录（Changes）
│   ├── README.md                           # Change 创建/归档说明
│   ├── <change-name>/                      # 当前正在执行的 change
│   │   ├── specs/                          # Delta Specs：声明本次变更涉及的 spec 变动
│   │   │   └── <domain>/                   # 按领域组织的 delta spec
│   │   │       └── spec.md                 # ADDED / MODIFIED / REMOVED Requirements
│   │   ├── proposal.md                     # 需求实现提案（Why + Scope + Approach）
│   │   ├── design.md                       # 技术设计方案（How + Architecture Decisions）
│   │   ├── tasks.md                        # 执行步骤清单（Steps with checkboxes）
│   │   └── .openspec.yaml                  # Change 元数据（可选）
│   └── archive/                            # 已归档的已完成 change
│       └── <date>-<change-name>/           # 带日期前缀的归档目录
```

### 核心概念

| 概念 | 说明 |
|------|------|
| **Specs** | 系统行为的"真实来源"，描述系统**当前**如何工作，不是实现计划 |
| **Changes** | 提议的修改，每个 change 一个独立文件夹，可并行存在 |
| **Delta Specs** | 描述**变更**而非全文重写：ADDED / MODIFIED / REMOVED |
| **Artifacts** | change 内的文档：proposal → specs → design → tasks |
| **Archive** | 完成后归档，delta specs 合入主 specs，保留完整上下文 |

### Spec 编写规范

Spec 是**行为契约**，不是实现计划：

```markdown
# <Domain> Specification

## Purpose
[这个 spec 描述什么]

## Requirements

### Requirement: [需求名称]
[系统必须具备的行为，使用 RFC 2119 关键词：SHALL / MUST / SHOULD / MAY]

#### Scenario: [场景名称]
- GIVEN [前置条件]
- WHEN [触发操作]
- THEN [预期结果]
```

**Delta Spec 格式**（change 内使用）：

```markdown
# Delta for <Domain>

## ADDED Requirements
### Requirement: [新增需求]
[新行为描述]
#### Scenario: [场景]
...

## MODIFIED Requirements
### Requirement: [修改需求]
[新行为，标注与旧版的差异]

## REMOVED Requirements
### Requirement: [废弃需求]
[废弃原因说明]
```

**Spec 中应避免**：
- 内部类/函数名
- 库或框架选择
- 逐步实现细节（这些属于 `design.md` 或 `tasks.md`）

### Artifact 流程

```
proposal ──────► specs ──────► design ──────► tasks ──────► implement
   │               │             │              │
  why            what           how          steps
+ scope        changes       approach      to take
```

| Artifact | 职责 | 何时更新 |
|----------|------|----------|
| `proposal.md` | Intent + Scope + Approach | 范围变化、意图明确、方案根本性改变 |
| `specs/` | Delta Specs（ADDED/MODIFIED/REMOVED） | 新发现的行为变更 |
| `design.md` | 技术方案 + 架构决策 | 实现发现方案不可行、发现更好方案 |
| `tasks.md` | 实现步骤清单（带 checkbox） | 学习驱动的小调整 |

### 核心规则

| 规则 | 说明 |
|------|------|
| **Spec 先行** | 修改系统行为前，先在 change 的 `specs/` 中声明 Delta Spec |
| **Change 驱动** | 任何非 trivial 的改动必须创建一个 change |
| **Delta 优先** | 描述变更而非重写全文，使用 ADDED/MODIFIED/REMOVED |
| **归档闭环** | 完成后归档，delta 自动合入主 specs |
| **保持聚焦** | 一个 change = 一个逻辑单元，避免混合不相关改动 |

### 命令速查

| 命令 | 用途 | 使用时机 |
|------|------|----------|
| `openspec init` | 初始化项目 OpenSpec | 首次使用 |
| `openspec update` | 更新 agent 指令到最新 | 升级后或定期执行 |
| `openspec --version` | 检查版本 | 会话启动时 |
| `/opsx:propose` | 创建 change + 生成全部规划文档 | 快速路径（推荐） |
| `/opsx:explore` | 探索问题空间 | 需求不清晰时 |
| `/opsx:apply` | 按 tasks.md 执行实现 | 规划完成，准备写代码 |
| `/opsx:verify` | 验证实现是否匹配 specs | 归档前检查 |
| `/opsx:archive` | 归档 change，合入主 specs | 全部工作完成 |

### Change 生命周期

```
前置条件检查（Node.js + OpenSpec CLI + 项目初始化）
  │
  ▼
/opsx:propose <change-name>（创建 change + 全部 artifacts）
  │
  ▼
/opsx:apply（按 tasks.md 逐步实现）
  │
  ▼
/opsx:verify（验证实现匹配 specs）[可选但推荐]
  │
  ▼
/opsx:archive（归档 + delta specs 合入主 specs）
```

### Agent 读取时机

| 时机 | 必读路径 |
|------|----------|
| 会话启动 | 执行前置条件检查 |
| 接到系统变更任务 | `openspec/specs/` → 了解系统当前状态 |
| 创建新 change | 先读 `openspec/specs/`，再创建 change |
| 执行实现 | `openspec/change/<name>/tasks.md` → 按步骤执行 |
| 提交前检查 | `openspec/change/<name>/specs/` → 确认 spec 同步 |
| 日常理解系统 | `openspec/specs/` → 系统完整行为描述 |

### 与 Feature SDD 流程的关系

OpenSpec 是 SDD 流程的**执行引擎**：

- `docs/feature/` 的 PRD/BED/FED/TEST 文档定义了**文档规范和模板**
- `openspec/` 是**实际的变更工作区**，每次 Feature 开发对应一个 change
- 两者配合：用 docs 的模板写 openspec 的文件，形成完整闭环

| Feature 文档 | OpenSpec Artifact | 对应关系 |
|-------------|-------------------|----------|
| PRD（需求） | proposal.md + specs/ | 需求拆解 → Delta Specs |
| BED（后端设计） | design.md（后端部分） | 技术方案 |
| FED（前端设计） | design.md（前端部分） | 技术方案 |
| TEST（测试） | /opsx:verify | 验证实现匹配 specs |

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

## UI 风格参考 (UI Style Reference)

> 读取时机：需要确定项目 UI 设计风格、编写前端页面或组件时。
> 此目录为 git submodule，克隆仓库时需使用 `--recurse-submodules`。

| 文档 | 描述 | 引用路径 | 读取时机 |
|------|------|----------|----------|
| UI 风格合集 | Awesome Design MD — 收录各类 UI 设计风格 Markdown 描述，用于选择项目整体视觉风格 | [docs/awesome-design-md/README.md](docs/awesome-design-md/README.md) | 确定项目 UI 风格、设计前端页面时 |

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
0. 前置条件检查 → Node.js + OpenSpec CLI + openspec init
1. 本文件（AGENTS.md） → 了解有哪些文档可用
2. openspec/specs/ → 了解系统当前状态
3. docs/feature/workflow.md → 了解完整闭环流程
4. 按需读取全局规范 → 根据具体任务选择
5. /opsx:propose <change-name> → 创建 change
6. 按 PRD → BED → FED → TEST 顺序推进（在 change 内）
```

### 常见场景的文档选择

| 场景 | 必读文档 |
|------|----------|
| 开始新 Feature | workflow.md → PRD-template.md |
| 设计后端 API | api-conventions.md → database-conventions.md → BED-template.md |
| 设计前端页面 | component-patterns.md → state-management.md → awesome-design-md/README.md → FED-template.md |
| 编写测试 | testing-standards.md → TEST-template.md |
| 提交代码 | git-workflow.md → review-checklist.md → process-file-management.md |
| 安全相关 | security-guidelines.md |
| 错误处理 | error-handling.md |

### 文档更新规则

- 全局规范变更需团队评审后修改
- Feature 文档随开发进度同步更新
- 模板变更需同步更新 AGENTS.md 索引
- 新增文档必须在 AGENTS.md 中注册引用
