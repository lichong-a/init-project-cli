# 状态管理规范 (State Management)

> 适用范围：全局规范 | 稳定规则 | 跨 Feature 复用

## 核心原则

- **单一数据源**：同一数据在全局只有一份权威来源
- **不可变更新**：永远用展开运算符/map/filter 创建新状态
- **最小订阅**：组件只订阅需要的数据，避免不必要的重渲染
- **明确边界**：局部状态用 ref/reactive，共享状态用 Store

## 状态分层

```
┌─────────────────────────────────────────┐
│  组件本地状态 (ref / reactive)            │  ← 表单值、UI 开关、临时状态
├─────────────────────────────────────────┤
│  跨组件共享状态 (Pinia Store)             │  ← 用户信息、全局配置、通知
├─────────────────────────────────────────┤
│  服务端状态 (查询缓存 / SWR)              │  ← API 数据、列表、详情
└─────────────────────────────────────────┘
```

## 何时用什么

| 场景 | 方案 | 示例 |
|------|------|------|
| 仅组件内部使用 | `ref()` / `reactive()` | 弹窗开关、表单值 |
| 父子组件通信 | `props` + `emits` | 列表传递选中项 |
| 跨组件共享 | Pinia Store | 用户登录态、主题设置 |
| 服务端数据缓存 | 查询层/Composable | 用户列表、订单详情 |
| 全局一次性配置 | App Config | 环境变量、特性开关 |

## Pinia Store 规范

```typescript
// modules/auth/stores/auth.ts
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'
import { authService } from '../services/authService'
import type { User, LoginInput } from '../types'
import { BusinessError } from '@/infrastructure/error/types'

export const useAuthStore = defineStore('auth', () => {
  // === State ===
  const user = ref<User | null>(null)
  const token = ref<string | null>(null)
  const loading = ref(false)
  const error = ref<string | null>(null)

  // === Getters ===
  const isAuthenticated = computed(() => !!token.value)
  const userName = computed(() => user.value?.name ?? '未登录')
  const userRole = computed(() => user.value?.role ?? 'guest')

  // === Actions ===
  async function login(input: LoginInput): Promise<void> {
    loading.value = true
    error.value = null

    try {
      const result = await authService.login(input)
      user.value = result.user
      token.value = result.token
    } catch (err) {
      error.value = err instanceof BusinessError
        ? err.message
        : '登录失败'
      throw err
    } finally {
      loading.value = false
    }
  }

  function logout(): void {
    user.value = null
    token.value = null
  }

  function $reset(): void {
    user.value = null
    token.value = null
    loading.value = false
    error.value = null
  }

  return {
    // State
    user,
    token,
    loading,
    error,
    // Getters
    isAuthenticated,
    userName,
    userRole,
    // Actions
    login,
    logout,
    $reset,
  }
})
```

## Composable 数据获取模式

```typescript
// shared/composables/useQuery.ts
import { ref, type Ref } from 'vue'

interface QueryResult<T> {
  data: Ref<T | null>
  loading: Ref<boolean>
  error: Ref<string | null>
  refresh: () => Promise<void>
}

export function useQuery<T>(
  fetcher: () => Promise<T>,
  options?: { immediate?: boolean },
): QueryResult<T> {
  const data = ref<T | null>(null) as Ref<T | null>
  const loading = ref(false)
  const error = ref<string | null>(null)

  async function refresh(): Promise<void> {
    loading.value = true
    error.value = null

    try {
      data.value = await fetcher()
    } catch (err) {
      error.value = err instanceof Error ? err.message : '请求失败'
    } finally {
      loading.value = false
    }
  }

  if (options?.immediate !== false) {
    refresh()
  }

  return { data, loading, error, refresh }
}

// 使用示例
export function useUserList() {
  return useQuery(() => userService.list({ page: 1, limit: 20 }))
}
```

## 不可变更新模式

```typescript
// Store 中的状态更新（不可变）

// 列表新增
state.items = [...state.items, newItem]

// 列表删除
state.items = state.items.filter(item => item.id !== targetId)

// 列表更新
state.items = state.items.map(item =>
  item.id === targetId ? { ...item, ...updates } : item,
)

// 对象更新
state.user = { ...state.user, name: 'new name' }

// 嵌套更新
state.config = {
  ...state.config,
  theme: {
    ...state.config.theme,
    primaryColor: '#1890ff',
  },
}
```

## 状态管理检查清单

- [ ] 本地状态与共享状态边界清晰
- [ ] Store 结构扁平，无深层嵌套
- [ ] 状态更新使用不可变模式
- [ ] 组件只订阅必要的状态
- [ ] 异步操作有 loading/error 状态
- [ ] Store 有 $reset 方法
- [ ] 无直接修改 Store 状态（通过 action）
