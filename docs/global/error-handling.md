# 错误处理规范 (Error Handling)

> 适用范围：全局规范 | 稳定规则 | 跨 Feature 复用

## 核心原则

- **错误分类**：区分业务错误、系统错误、网络错误
- **错误传播**：底层抛出具体错误，边界层统一处理
- **错误恢复**：提供降级方案，不让用户看到技术细节
- **错误追踪**：关键错误记录日志，便于排查

## 错误分类体系

```typescript
// infrastructure/error/types.ts

// 基础应用错误
class AppError extends Error {
  constructor(
    message: string,
    public code: string,
    public isOperational = true,
  ) {
    super(message)
    this.name = 'AppError'
  }
}

// 网络错误
class NetworkError extends AppError {
  constructor(
    message: string,
    public statusCode: number,
    code = 'NETWORK_ERROR',
  ) {
    super(message, code)
    this.name = 'NetworkError'
  }
}

// 业务错误
class BusinessError extends AppError {
  constructor(
    message: string,
    code: string,
    public context?: Record<string, unknown>,
  ) {
    super(message, code)
    this.name = 'BusinessError'
  }
}

// 验证错误
class ValidationError extends AppError {
  constructor(
    message: string,
    public fields: Record<string, string[]>,
  ) {
    super(message, 'VALIDATION_ERROR')
    this.name = 'ValidationError'
  }
}

// 权限错误
class AuthorizationError extends AppError {
  constructor(
    message = '无权限执行此操作',
    public requiredPermission?: string,
  ) {
    super(message, 'FORBIDDEN')
    this.name = 'AuthorizationError'
  }
}
```

## 错误处理层次

### 第一层：Service 层（抛出具体错误）

```typescript
// modules/user/services/userService.ts
export const userService = {
  async create(input: unknown): Promise<User> {
    // 输入验证
    const validated = createUserSchema.safeParse(input)
    if (!validated.success) {
      throw new ValidationError(
        '用户数据验证失败',
        validated.error.flatten().fieldErrors,
      )
    }

    try {
      return await apiClient.post<User>('/users', validated.data)
    } catch (error) {
      if (error instanceof NetworkError && error.statusCode === 409) {
        throw new BusinessError('邮箱已被注册', 'USER_EMAIL_EXISTS')
      }
      throw error // 向上传播其他错误
    }
  },
}
```

### 第二层：Composable/Store 层（转换和降级）

```typescript
// modules/user/composables/useUserForm.ts
export function useUserForm() {
  const error = ref<string | null>(null)
  const fieldErrors = ref<Record<string, string[]>>({})

  async function submit(input: unknown) {
    error.value = null
    fieldErrors.value = {}

    try {
      return await userService.create(input)
    } catch (err) {
      if (err instanceof ValidationError) {
        fieldErrors.value = err.fields
        error.value = err.message
      } else if (err instanceof BusinessError) {
        error.value = err.message
      } else {
        error.value = '操作失败，请稍后重试'
        logger.error('Unexpected error in user creation', err)
      }
      throw err // 让调用方决定是否继续传播
    }
  }

  return { error, fieldErrors, submit }
}
```

### 第三层：UI 层（展示和兜底）

```vue
<template>
  <form @submit.prevent="handleSubmit">
    <Alert v-if="error" type="error">{{ error }}</Alert>
    <FormField
      v-model="form.email"
      label="邮箱"
      :errors="fieldErrors.email"
    />
    <Button type="submit" :loading="loading">提交</Button>
  </form>
</template>
```

### 第四层：全局兜底

```typescript
// infrastructure/error/globalHandler.ts
export function setupGlobalErrorHandler() {
  // 未捕获的 Promise 错误
  window.addEventListener('unhandledrejection', (event) => {
    const error = event.reason
    logger.error('Unhandled rejection', error)

    if (error instanceof AuthorizationError) {
      router.push('/login')
    } else if (error instanceof NetworkError) {
      toast.error('网络连接异常，请检查网络设置')
    } else {
      toast.error('系统异常，请稍后重试')
    }

    event.preventDefault()
  })

  // 全局路由错误
  router.onError((error) => {
    logger.error('Router error', error)
  })
}
```

## 异步错误处理模式

```typescript
// 模式一：try/catch（推荐用于需要错误转换的场景）
async function fetchData() {
  try {
    const result = await apiClient.get('/data')
    return result
  } catch (error) {
    throw new BusinessError('数据获取失败', 'DATA_FETCH_FAILED', {
      originalError: error,
    })
  }
}

// 模式二：Result 模式（推荐用于不抛错的场景）
type Result<T> =
  | { success: true; data: T }
  | { success: false; error: AppError }

async function safeFetch<T>(url: string): Promise<Result<T>> {
  try {
    const data = await apiClient.get<T>(url)
    return { success: true, data }
  } catch (error) {
    return { success: false, error: normalizeError(error) }
  }
}
```

## 错误处理检查清单

- [ ] Service 层抛出具体错误类型
- [ ] 边界层（Composable/Store）处理并降级
- [ ] UI 层展示用户友好的错误消息
- [ ] 全局兜底处理未捕获错误
- [ ] 关键错误记录到日志系统
- [ ] 错误消息不泄露内部实现细节
- [ ] 网络错误有重试机制
