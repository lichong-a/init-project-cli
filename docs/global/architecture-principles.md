# 架构原则 (Architecture Principles)

> 适用范围：全局规范 | 稳定规则 | 跨 Feature 复用

## 核心理念

- **关注点分离**：每个模块只做一件事
- **依赖倒置**：依赖抽象，不依赖具体实现
- **单向数据流**：数据流向可预测、可追踪
- **高内聚低耦合**：模块内部紧密相关，模块之间松散连接

## 项目分层架构

```
src/
├── app/                    # 应用入口与全局配置
│   ├── main.ts
│   ├── router.ts
│   └── App.vue
├── modules/                # 按业务域划分的模块
│   ├── auth/
│   │   ├── components/     # 该模块的组件
│   │   ├── composables/    # 该模块的组合式函数
│   │   ├── services/       # 该模块的 API 调用
│   │   ├── stores/         # 该模块的状态管理
│   │   ├── types/          # 该模块的类型定义
│   │   └── utils/          # 该模块的工具函数
│   └── user/
├── shared/                 # 跨模块共享
│   ├── components/         # 通用 UI 组件
│   ├── composables/        # 通用组合式函数
│   ├── services/           # 通用服务
│   ├── stores/             # 全局状态
│   ├── types/              # 通用类型
│   └── utils/              # 通用工具
└── infrastructure/         # 基础设施
    ├── http/               # HTTP 客户端配置
    ├── storage/            # 存储抽象
    └── logger/             # 日志抽象
```

## 模块边界规则

```typescript
// 允许的依赖方向（从上到下）：
// app → modules → shared → infrastructure
// modules 之间：禁止直接引用，通过事件/Store 通信
// shared → infrastructure：允许
// infrastructure：不依赖任何上层模块
```

## 模块通信模式

```typescript
// 模式一：通过 Store（推荐用于状态共享）
// modules/auth/stores/auth.ts
export const useAuthStore = defineStore('auth', () => {
  const user = ref<User | null>(null)
  return { user }
})

// 模式二：通过事件总线（推荐用于一次性通知）
// shared/utils/eventBus.ts
export const eventBus = {
  loginSuccess: createEvent<User>('auth:login-success'),
  logout: createEvent<void>('auth:logout'),
}

// 模式三：通过 Composable（推荐用于逻辑复用）
// shared/composables/useAuth.ts
export function useAuth() {
  const store = useAuthStore()
  return {
    user: computed(() => store.user),
    login: store.login,
    logout: store.logout,
  }
}
```

## 错误边界

```typescript
// 每个模块应有自己的错误类型
// modules/auth/types/errors.ts
export class AuthError extends Error {
  constructor(
    message: string,
    public code: AuthErrorCode,
    public statusCode?: number,
  ) {
    super(message)
    this.name = 'AuthError'
  }
}

// 全局错误处理
// infrastructure/error/handler.ts
export function handleError(error: unknown): never {
  if (error instanceof AuthError) {
    // 跳转登录页
  }
  if (error instanceof NetworkError) {
    // 显示网络错误提示
  }
  // 兜底处理
  throw error
}
```

## 设计决策记录 (ADR)

当做出重要架构决策时，需记录：

```markdown
## ADR-XXX: [决策标题]

### 上下文
[什么情况下做出的决策]

### 决策
[做出了什么决策]

### 理由
[为什么选择这个方案]

### 后果
[这个决策带来的正面和负面影响]
```

## 架构评审清单

- [ ] 模块边界清晰，无循环依赖
- [ ] 数据流向单一可追踪
- [ ] 错误处理有明确边界
- [ ] 新增模块符合分层架构
- [ ] 公共逻辑抽取到 shared
- [ ] 无跨模块直接引用
- [ ] 类型定义在模块内部或 shared 中
