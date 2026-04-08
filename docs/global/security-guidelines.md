# 安全指南 (Security Guidelines)

> 适用范围：全局规范 | 稳定规则 | 跨 Feature 复用

## 核心原则

- **最小权限原则**：只授予完成任务所需的最小权限
- **纵深防御**：多层安全措施，不依赖单一防线
- **零信任**：不信任任何外部输入，包括用户输入和 API 响应
- **安全左移**：在开发阶段就考虑安全，而非事后补救

## 强制安全检查（提交前必检）

- [ ] 无硬编码密钥（API Key、密码、Token）
- [ ] 所有用户输入已验证
- [ ] SQL 注入防护（参数化查询）
- [ ] XSS 防护（输出转义）
- [ ] CSRF 保护已启用
- [ ] 认证/授权已验证
- [ ] 接口有速率限制
- [ ] 错误消息不泄露敏感信息

## 密钥管理

```typescript
// NEVER: 硬编码密钥
const apiKey = 'sk-proj-xxxxx'
const dbPassword = 'admin123'

// ALWAYS: 环境变量
const apiKey = process.env.API_KEY
if (!apiKey) {
  throw new Error('API_KEY not configured')
}

// .env.local（不提交到 Git）
// API_KEY=sk-proj-xxxxx
// DB_PASSWORD=secure_password

// .env.example（提交到 Git 作为模板）
// API_KEY=your_api_key_here
// DB_PASSWORD=your_db_password_here
```

## 输入验证

```typescript
import { z } from 'zod'

// 使用 Zod 进行运行时验证
const createUserSchema = z.object({
  email: z.string().email('邮箱格式不正确'),
  password: z
    .string()
    .min(8, '密码至少 8 位')
    .regex(/[A-Z]/, '需要包含大写字母')
    .regex(/[0-9]/, '需要包含数字'),
  age: z.number().int().min(0).max(150),
  role: z.enum(['user', 'admin']).default('user'),
})

type CreateUserInput = z.infer<typeof createUserSchema>

// 在边界层验证
function createUser(input: unknown): Promise<User> {
  const validated = createUserSchema.parse(input) // 不合法直接抛错
  return userService.create(validated)
}
```

## XSS 防护

```typescript
// NEVER: 直接插入 HTML
element.innerHTML = userInput

// ALWAYS: 使用 textContent 或框架的模板语法
element.textContent = userInput

// Vue 中自动转义（安全）
// <div>{{ userInput }}</div>  ← 安全，自动转义
// <div v-html="userInput"></div>  ← 危险！仅在信任内容时使用

// 如必须使用 v-html，先清理
import DOMPurify from 'dompurify'
const safeHtml = DOMPurify.sanitize(userInput)
```

## 认证与授权

```typescript
// Token 存储：使用 httpOnly Cookie（优先）或内存 + Refresh Token
// NEVER: localStorage 存储敏感 Token

// 路由守卫
router.beforeEach((to) => {
  const authStore = useAuthStore()

  if (to.meta.requiresAuth && !authStore.isAuthenticated) {
    return { name: 'login', query: { redirect: to.fullPath } }
  }

  if (to.meta.requiresRole) {
    const requiredRole = to.meta.requiresRole as string
    if (!authStore.hasRole(requiredRole)) {
      return { name: 'forbidden' }
    }
  }
})

// API 权限检查
function canPerformAction(action: string, resource: string): boolean {
  const { permissions } = useAuthStore()
  return permissions.some(
    p => p.action === action && p.resource === resource,
  )
}
```

## CSRF 防护

```typescript
// 方案一：SameSite Cookie（推荐）
// Set-Cookie: token=xxx; SameSite=Strict; Secure; HttpOnly

// 方案二：CSRF Token
// 每个表单提交携带服务端生成的 CSRF Token
const csrfToken = document.querySelector('meta[name="csrf-token"]')?.content

fetch('/api/data', {
  method: 'POST',
  headers: {
    'X-CSRF-Token': csrfToken || '',
  },
})
```

## 安全响应协议

发现安全问题时：

1. **立即停止**当前开发工作
2. **标记问题**：使用 `security-reviewer` agent 分析
3. **修复 CRITICAL** 级别问题后再继续
4. **轮换密钥**：任何暴露的密钥立即轮换
5. **全量扫描**：排查同类问题

## 安全评审清单

- [ ] 无硬编码密钥/密码/Token
- [ ] 用户输入全部经过验证和清理
- [ ] 数据库查询使用参数化
- [ ] 输出经过转义，无 XSS 风险
- [ ] 认证 Token 安全存储
- [ ] 敏感操作有权限校验
- [ ] 错误消息不含内部信息
- [ ] 依赖包无已知漏洞
