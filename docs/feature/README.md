# Feature 开发指南

> 本目录包含 Feature 开发所需的模板和工作流说明

## 核心理念

**规范驱动开发 (SDD)**：每个 Feature 必须先完成规范文档，再进入实现。文档是 AI 协作的基础上下文。

## 四层文档体系

```
Feature: PRD-XXX-功能名称
  │
  ├── 第一层：PRD（产品需求文档）
  │   └─ 承接业务目标、场景、范围、规则和需求边界
  │
  ├── 第二层：BED（后端实现设计）
  │   └─ 承接后端实现设计、接口、数据结构、约束、服务逻辑和单元测试
  │
  ├── 第三层：FED（前端实现设计）
  │   └─ 承接前端页面、交互、状态流转、联动逻辑和实现关注点
  │
  └── 第四层：TEST（测试验证文档）
      └─ 承接功能验收测试和回归测试
```

## 闭环流程

```
PRD → BED → FED → TEST → PRD（反馈闭环）
 │                │
 └── 实现 ────────┘
         │
    Code Review
         │
    合并到主分支
         │
    评估回归测试
         │
    更新回归用例库
         │
    再次验证 ← ── ┘
```

## 如何使用

### 1. 创建新 Feature

```bash
# 在 docs/feature/ 下创建新目录
mkdir -p docs/feature/PRD-001-user-auth
```

### 2. 按顺序填写文档

1. 复制 `templates/PRD-template.md` → `PRD-001-user-auth/PRD.md`
2. 复制 `templates/BED-template.md` → `PRD-001-user-auth/BED.md`
3. 复制 `templates/FED-template.md` → `PRD-001-user-auth/FED.md`
4. 复制 `templates/TEST-template.md` → `PRD-001-user-auth/TEST.md`

### 3. 遵循工作流

详见 [workflow.md](./workflow.md)

## 模板文件

| 文件 | 说明 |
|------|------|
| [PRD-template.md](./templates/PRD-template.md) | 产品需求文档模板 |
| [BED-template.md](./templates/BED-template.md) | 后端实现设计模板 |
| [FED-template.md](./templates/FED-template.md) | 前端实现设计模板 |
| [TEST-template.md](./templates/TEST-template.md) | 测试验证文档模板 |

## Feature 命名规范

```
PRD-{编号}-{简短英文描述}
例：
  PRD-001-user-auth
  PRD-002-order-management
  PRD-003-data-dashboard
```
