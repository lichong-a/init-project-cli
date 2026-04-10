# 组件模式 (Component Patterns)

> 适用范围：全局规范 | 稳定规则 | 跨 Feature 复用

## 核心原则

- **声明式优先**：描述"是什么"而非"怎么做"
- **Props Down, Events Up**：单向数据流
- **单一职责**：一个组件只做一件事
- **可组合 > 可配置**：通过组合而非 Props 配置实现灵活性

## 组件分类

```
shared/components/
├── ui/                 # 基础 UI 组件（无业务逻辑）
│   ├── Button
│   ├── Input
│   ├── Modal
│   └── Table
├── layout/             # 布局组件
│   ├── Header
│   ├── Sidebar
│   └── Layout
└── business/           # 通用业务组件（跨模块复用）
    ├── UserAvatar
    └── StatusBadge
```

## 通用组件设计模式

### 受控组件模式

所有 UI 框架通用的受控组件模式：

```
Properties:
  - value: T          // 当前值（外部传入）
  - onChange: (T) => void  // 值变化回调

// 使用方控制数据，组件只负责展示和通知
<MyInput value={searchText} onChange={setSearchText} />
```

### 组合模式（插槽/子组件）

```
<Card>
  <Card.Header>
    <Title>标题</Title>
  </Card.Header>
  <Card.Body>
    内容区域
  </Card.Body>
  <Card.Footer>
    <Button>确定</Button>
  </Card.Footer>
</Card>
```

### 加载/错误/空状态模式

```
Component {
  States:
    | Loading   → 展示加载指示器
    | Error     → 展示错误信息 + 重试
    | Empty     → 展示空状态引导
    | Success   → 展示实际内容
}
```

## 组件结构规范

每个组件应包含：

1. **Props/接口定义**：明确的输入类型
2. **事件/回调定义**：明确的输出
3. **状态管理**：本地状态最小化
4. **样式**：Scoped / CSS Modules（如适用）

```
// 通用组件结构
ComponentName/
├── ComponentName.ext        # 组件逻辑
├── ComponentName.style.ext  # 样式（如需要独立文件）
├── ComponentName.test.ext   # 测试
└── index.ext                # 导出
```

## 组件命名规范

| 类型 | 前缀/规则 | 示例 |
|------|-----------|------|
| 基础 UI | 无前缀 | `Button`, `Input`, `Modal` |
| 布局 | `App` / `Layout` | `AppHeader`, `MainLayout` |
| 业务组件 | 业务名 | `UserAvatar`, `OrderList` |
| 高阶/包装 | `with*` / `*Wrapper` | `withAuth`, `ErrorBoundary` |
| 页面 | `*Page` / `*View` | `UserListPage`, `LoginView` |

## 框架映射

| 模式 | React | Vue | Svelte | Angular | Flutter |
|------|-------|-----|--------|---------|---------|
| 受控组件 | useState + props | v-model + defineProps | bind:on | @Input/@Output | StatefulWidget |
| 组合/插槽 | children / render props | slots | slot | ng-content | Widget tree |
| 状态管理 | useState/useReducer | ref/reactive | $: / stores | Signals | setState/ValueNotifier |
| 样式隔离 | CSS Modules / styled | scoped / CSS Modules | scoped | encapsulation | ThemeData |
| 生命周期 | useEffect | onMounted/etc | onMount | ngOnInit | initState |

## 组件评审清单

- [ ] 单一职责，不超过 200 行
- [ ] Props/接口有类型定义和默认值
- [ ] 事件/回调有明确定义
- [ ] 支持 Loading/Error/Empty 状态
- [ ] 使用 scoped/CSS Modules 样式
- [ ] 可访问性（a11y）基本合规
- [ ] 列表渲染有唯一 key
