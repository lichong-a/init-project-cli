# 状态管理规范 (State Management)

> 适用范围：全局规范 | 稳定规则 | 跨 Feature 复用

## 核心原则

- **单一数据源**：同一数据在全局只有一份权威来源
- **不可变更新**：永远创建新状态，不要直接修改
- **最小订阅**：组件/模块只订阅需要的数据
- **明确边界**：局部状态用本地变量，共享状态用集中管理

## 状态分层

```
┌─────────────────────────────────────────┐
│  本地状态                                 │  ← 函数变量、表单值、UI 开关
├─────────────────────────────────────────┤
│  跨模块共享状态                           │  ← 用户信息、全局配置、通知
├─────────────────────────────────────────┤
│  服务端状态（查询缓存）                    │  ← API 数据、列表、详情
└─────────────────────────────────────────┘
```

## 何时用什么

| 场景 | 方案 | 示例 |
|------|------|------|
| 仅函数/组件内部使用 | 本地变量 | 临时标志、表单值 |
| 父子模块通信 | Props + Callbacks | 列表传递选中项 |
| 跨模块共享 | 集中式 Store | 用户登录态、主题设置 |
| 服务端数据 | 数据获取层 + 缓存 | 列表数据、详情数据 |
| 全局一次性配置 | 配置文件/环境变量 | 特性开关、API 地址 |

## 集中式 Store 规范

无论使用什么状态管理库，Store 应遵循：

```
结构：
├── State（数据）        ← 只存数据，不含逻辑
├── Getters/Selectors   ← 派生计算，纯函数
└── Actions/Reducers    ← 唯一修改 State 的入口
```

### Store 必须包含

1. **明确的初始状态**：所有字段有默认值
2. **Selectors/Getters**：只读的派生数据
3. **Actions/Reducers**：所有状态变更通过 action 触发
4. **Loading/Error 状态**：异步操作必须跟踪
5. **Reset 方法**：可重置到初始状态

### 异步操作模式

```
function asyncAction(params):
  1. state.loading = true
  2. state.error = null
  3. try:
       result = await service.call(params)
       state.data = result
  4. catch (error):
       state.error = normalize(error)
       throw error  // 让调用方决定是否继续传播
  5. finally:
       state.loading = false
```

## 不可变更新模式

```
// 通用原则（伪代码）

// 新增
state.items = [...state.items, newItem]

// 删除
state.items = state.items.filter(item => item.id !== targetId)

// 更新
state.items = state.items.map(item =>
  item.id === targetId ? { ...item, ...updates } : item
)

// 嵌套更新
state.config = {
  ...state.config,
  theme: { ...state.config.theme, primaryColor: '#1890ff' }
}
```

## 语言/框架映射

| 语言/框架 | 推荐方案 | 备注 |
|-----------|----------|------|
| Vue | Pinia | Composition API 风格 |
| React | Zustand / Redux Toolkit | 选其一 |
| Svelte | Svelte Stores | 内置 |
| Swift | Combine / Observable | 内置 |
| Kotlin | StateFlow / ViewModel | Jetpack |
| Python | 无全局状态管理 | 通过依赖注入 |
| Go | 无全局状态管理 | 通过接口和依赖注入 |
| Rust | 通过类型系统 | 所有权天然可追踪 |

## 状态管理检查清单

- [ ] 本地状态与共享状态边界清晰
- [ ] Store 结构扁平，无深层嵌套
- [ ] 状态更新使用不可变模式
- [ ] 模块只订阅必要的状态
- [ ] 异步操作有 loading/error 状态
- [ ] Store 有 reset 方法
- [ ] 无直接修改 Store 状态（通过 action/reducer）
