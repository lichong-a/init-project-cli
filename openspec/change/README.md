# OpenSpec Change Directory

> 此目录管理所有进行中和已归档的变更。

## 目录结构

```
change/
├── README.md               # 本文件（目录说明）
├── <change-name>/          # 进行中的 change
│   ├── specs/              # 本次变更涉及的 spec 变动
│   ├── proposal.md         # 需求实现提案
│   ├── plan.md             # 执行方案
│   └── task.md             # 执行步骤节点
└── archive/                # 已完成的 change 归档
    └── <change-name>/      # 保持原始结构
```

## Change 命名规范

```
<类型>-<简述>
例：
  feat-user-auth
  fix-login-timeout
  refactor-api-layer
  perf-list-render
```

| 类型前缀 | 用途 |
|----------|------|
| `feat-` | 新功能 |
| `fix-` | Bug 修复 |
| `refactor-` | 重构 |
| `perf-` | 性能优化 |
| `docs-` | 文档变更 |

## 创建新 Change

```bash
mkdir -p openspec/change/<change-name>/specs
touch openspec/change/<change-name>/proposal.md
touch openspec/change/<change-name>/plan.md
touch openspec/change/<change-name>/task.md
```

## 文件职责

| 文件 | 职责 | 必须包含 |
|------|------|----------|
| `proposal.md` | 需求拆解：改什么、为什么、影响范围 | 变更目标、范围、影响分析 |
| `plan.md` | 技术方案：怎么实现、路径、风险 | 技术方案、实现步骤、风险点 |
| `task.md` | 任务拆解：有序的执行步骤列表 | 编号任务列表、每步完成标准 |
| `specs/` | 本次涉及的 spec 变动 | 变更前后的 spec 对比或新 spec |

## 归档流程

1. 所有 task.md 中的任务完成并验证通过
2. 将 `specs/` 中的变动合入 `openspec/specs/`
3. 将整个 change 目录移入 `archive/`
4. 归档后不可修改（append-only）
