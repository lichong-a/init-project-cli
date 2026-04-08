# API 接口规范 (API Conventions)

> 适用范围：全局规范 | 稳定规则 | 跨 Feature 复用

## RESTful 设计原则

### URL 规范

```
GET    /api/v1/users           # 获取列表
GET    /api/v1/users/:id       # 获取详情
POST   /api/v1/users           # 创建
PUT    /api/v1/users/:id       # 全量更新
PATCH  /api/v1/users/:id       # 部分更新
DELETE /api/v1/users/:id       # 删除

# 嵌套资源
GET    /api/v1/users/:id/orders
POST   /api/v1/users/:id/orders

# 动作类接口（非 CRUD）
POST   /api/v1/users/:id/activate
POST   /api/v1/auth/login
POST   /api/v1/auth/logout
```

### 统一响应格式

```typescript
// 成功响应
interface ApiResponse<T> {
  success: true
  data: T
  meta?: {
    total: number
    page: number
    limit: number
    hasNext: boolean
  }
}

// 错误响应
interface ApiError {
  success: false
  error: {
    code: string        // 业务错误码 'USER_NOT_FOUND'
    message: string     // 用户可读消息
    details?: Record<string, string[]>  // 字段级错误
  }
}
```

### 分页参数

```typescript
interface PaginationParams {
  page?: number     // 页码，从 1 开始，默认 1
  limit?: number    // 每页数量，默认 20，最大 100
  sort?: string     // 排序字段，如 'createdAt:desc'
}

// 请求示例
GET /api/v1/users?page=1&limit=20&sort=createdAt:desc&status=active
```

## HTTP 客户端封装

```typescript
// infrastructure/http/client.ts
import type { ApiResponse, ApiError } from '@/types/api'

interface RequestOptions {
  params?: Record<string, unknown>
  headers?: Record<string, string>
  signal?: AbortSignal
}

class HttpClient {
  private baseUrl: string

  constructor(baseUrl: string) {
    this.baseUrl = baseUrl
  }

  async get<T>(url: string, options?: RequestOptions): Promise<T> {
    return this.request<T>('GET', url, undefined, options)
  }

  async post<T>(url: string, data?: unknown, options?: RequestOptions): Promise<T> {
    return this.request<T>('POST', url, data, options)
  }

  private async request<T>(
    method: string,
    url: string,
    data?: unknown,
    options?: RequestOptions,
  ): Promise<T> {
    const config: RequestInit = {
      method,
      headers: {
        'Content-Type': 'application/json',
        ...this.getAuthHeaders(),
        ...options?.headers,
      },
      signal: options?.signal,
    }

    if (data && method !== 'GET') {
      config.body = JSON.stringify(data)
    }

    const response = await fetch(`${this.baseUrl}${url}`, config)
    return this.handleResponse<T>(response)
  }

  private getAuthHeaders(): Record<string, string> {
    const token = localStorage.getItem('auth_token')
    return token ? { Authorization: `Bearer ${token}` } : {}
  }

  private async handleResponse<T>(response: Response): Promise<T> {
    const json = await response.json()

    if (!response.ok) {
      const error: ApiError = json
      throw new ApiRequestError(
        error.error.message,
        error.error.code,
        response.status,
      )
    }

    const result: ApiResponse<T> = json
    return result.data
  }
}

export const apiClient = new HttpClient(import.meta.env.VITE_API_BASE_URL)
```

## Service 层规范

```typescript
// modules/user/services/userService.ts
import { apiClient } from '@/infrastructure/http/client'
import type { User, CreateUserInput, UpdateUserInput } from '../types'

interface UserListParams {
  page?: number
  limit?: number
  status?: string
}

export const userService = {
  async list(params: UserListParams): Promise<{ items: User[]; total: number }> {
    return apiClient.get('/users', { params })
  },

  async getById(id: string): Promise<User> {
    return apiClient.get(`/users/${id}`)
  },

  async create(input: CreateUserInput): Promise<User> {
    return apiClient.post('/users', input)
  },

  async update(id: string, input: UpdateUserInput): Promise<User> {
    return apiClient.patch(`/users/${id}`, input)
  },

  async remove(id: string): Promise<void> {
    return apiClient.delete(`/users/${id}`)
  },
}
```

## 错误码规范

```typescript
// 全局错误码：COMMON_*
// 认证错误：AUTH_*
// 用户模块：USER_*
// 订单模块：ORDER_*

enum ErrorCode {
  // 通用 1xxx
  COMMON_VALIDATION_ERROR = 'COMMON_1001',
  COMMON_NOT_FOUND = 'COMMON_1002',
  COMMON_CONFLICT = 'COMMON_1003',
  COMMON_RATE_LIMIT = 'COMMON_1004',

  // 认证 2xxx
  AUTH_INVALID_CREDENTIALS = 'AUTH_2001',
  AUTH_TOKEN_EXPIRED = 'AUTH_2002',
  AUTH_FORBIDDEN = 'AUTH_2003',

  // 用户 3xxx
  USER_NOT_FOUND = 'USER_3001',
  USER_EMAIL_EXISTS = 'USER_3002',
}
```

## API 评审清单

- [ ] URL 符合 RESTful 规范
- [ ] 响应格式统一（ApiResponse / ApiError）
- [ ] 分页参数标准化
- [ ] 错误码遵循编码规范
- [ ] Service 层封装完整
- [ ] 请求/响应类型定义完整
- [ ] 认证 Token 自动注入
- [ ] 请求取消（AbortSignal）支持
