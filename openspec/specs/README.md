# OpenSpec System Specifications

> 此目录声明当前系统是如何工作的。
> 所有 spec 文件共同构成系统的完整行为描述。

## 目录用途

- `openspec/specs/` 下的每个文件描述系统的一个侧面
- 这些 spec 是系统的"真实来源"，代码实现必须与 spec 保持一致
- 变更系统行为时，先更新 spec，再实现代码

## 文件命名规范

```
openspec/specs/
├── README.md                # 本文件（目录说明）
├── overview.md              # 系统总览
├── architecture.md          # 系统架构
├── <domain>-spec.md         # 按领域拆分的规格
└── ...
```

## Spec 与 Change 的关系

```
openspec/specs/          ← 系统当前状态（如何工作）
       │
       │  变更触发
       ▼
openspec/change/<name>/
       ├── specs/        ← 本次变更涉及的 spec 变动
       ├── proposal.md   ← 变更提案（需求拆解）
       ├── plan.md       ← 执行方案
       └── task.md       ← 执行步骤
```

## 编写规范

每个 spec 文件应包含：

1. **概述**：这个 spec 描述什么
2. **行为定义**：系统在此方面的完整行为规则
3. **约束**：不可违反的硬性约束
4. **接口**：与其他 spec 的交互点
5. **变更记录**：记录历次变更
