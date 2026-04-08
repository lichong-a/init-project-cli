# 测试标准 (Testing Standards)

> 适用范围：全局规范 | 稳定规则 | 跨 Feature 复用

## 核心原则

- **TDD 强制**：先写测试，再写实现
- **覆盖率门槛**：最低 80%，核心业务逻辑 90%+
- **测试金字塔**：单元 > 集成 > E2E（7:2:1）
- **测试独立性**：每个测试可独立运行，无执行顺序依赖

## 测试分层

```
tests/
├── unit/                    # 单元测试（70%）
│   ├── utils/               # 工具函数测试
│   ├── services/            # 服务层测试
│   ├── stores/              # Store 测试
│   └── composables/         # Composable 测试
├── integration/             # 集成测试（20%）
│   ├── api/                 # API 集成测试
│   └── modules/             # 模块集成测试
└── e2e/                     # 端到端测试（10%）
    ├── auth.spec.ts         # 认证流程
    └── core-flows.spec.ts   # 核心业务流程
```

## 单元测试规范

```typescript
// tests/unit/utils/formatDate.test.ts
import { describe, it, expect } from 'vitest'
import { formatDate } from '@/shared/utils/formatDate'

describe('formatDate', () => {
  it('应该将 ISO 日期字符串格式化为 YYYY-MM-DD', () => {
    const input = '2024-01-15T08:30:00Z'
    const result = formatDate(input)
    expect(result).toBe('2024-01-15')
  })

  it('应该处理空值返回默认格式', () => {
    const result = formatDate(null)
    expect(result).toBe('--')
  })

  it('应该在传入无效日期时抛出错误', () => {
    expect(() => formatDate('invalid')).toThrow('Invalid date')
  })
})
```

## 服务层测试规范

```typescript
// tests/unit/services/userService.test.ts
import { describe, it, expect, vi, beforeEach } from 'vitest'
import { userService } from '@/modules/user/services/userService'

// Mock HTTP 客户端
vi.mock('@/infrastructure/http/client', () => ({
  apiClient: {
    get: vi.fn(),
    post: vi.fn(),
    patch: vi.fn(),
    delete: vi.fn(),
  },
}))

describe('userService', () => {
  beforeEach(() => {
    vi.clearAllMocks()
  })

  describe('list', () => {
    it('应该正确传递分页参数', async () => {
      const mockResponse = { items: [], total: 0 }
      vi.mocked(apiClient.get).mockResolvedValue(mockResponse)

      await userService.list({ page: 2, limit: 10 })

      expect(apiClient.get).toHaveBeenCalledWith('/users', {
        params: { page: 2, limit: 10 },
      })
    })
  })
})
```

## Composable 测试规范

```typescript
// tests/unit/composables/useCounter.test.ts
import { describe, it, expect } from 'vitest'
import { useCounter } from '@/shared/composables/useCounter'

describe('useCounter', () => {
  it('应该正确初始化计数', () => {
    const { count } = useCounter(10)
    expect(count.value).toBe(10)
  })

  it('应该正确递增', () => {
    const { count, increment } = useCounter(0)
    increment()
    expect(count.value).toBe(1)
  })
})
```

## E2E 测试规范

```typescript
// tests/e2e/auth.spec.ts
import { test, expect } from '@playwright/test'

test.describe('认证流程', () => {
  test('用户可以登录并跳转到首页', async ({ page }) => {
    await page.goto('/login')

    await page.fill('[data-testid="email-input"]', 'test@example.com')
    await page.fill('[data-testid="password-input"]', 'password123')
    await page.click('[data-testid="login-button"]')

    await expect(page).toHaveURL('/')
    await expect(page.locator('[data-testid="user-menu"]')).toBeVisible()
  })
})
```

## Mock 规范

```typescript
// 优先使用 MSW 拦截网络请求（集成测试）
import { setupServer } from 'msw/node'
import { http, HttpResponse } from 'msw'

export const server = setupServer(
  http.get('/api/v1/users', () => {
    return HttpResponse.json({
      success: true,
      data: { items: mockUsers, total: 2 },
    })
  }),
)

// 单元测试使用 vi.mock
vi.mock('@/modules/user/services/userService')
```

## 测试命名规范

```
describe('模块名/函数名')
  describe('方法名或场景')
    it('应该 [预期行为] 当 [条件]')   ← 中文描述
    it('should [expected behavior] when [condition]')  ← 或英文
```

## 测试质量检查清单

- [ ] 测试覆盖率 ≥ 80%
- [ ] 核心业务逻辑覆盖率 ≥ 90%
- [ ] 无跳过的测试（skip/only）
- [ ] Mock 最小化，不 mock 被测单元本身
- [ ] 测试命名清晰描述预期行为
- [ ] 边界条件有覆盖（空值、异常、极限值）
- [ ] 测试独立运行无依赖
- [ ] 异步操作正确使用 async/await
