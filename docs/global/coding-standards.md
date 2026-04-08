# 编码标准 (Coding Standards)

> 适用范围：全局规范 | 稳定规则 | 跨 Feature 复用

## 核心原则

- **不可变性优先**：永远创建新对象，不要修改原对象
- **函数式风格**：纯函数、无副作用、显式数据流
- **小文件优先**：单文件 200-400 行为佳，上限 800 行
- **小函数优先**：单函数 <50 行，单一职责

## 命名规范

### 文件命名

| 类型 | 格式 | 示例 |
|------|------|------|
| 组件 | PascalCase | `UserProfile.vue` |
| 工具函数 | camelCase | `formatDate.ts` |
| 类型定义 | camelCase | `userTypes.ts` |
| 常量 | UPPER_SNAKE | `API_ENDPOINTS.ts` |
| 测试 | 被测文件名.test | `formatDate.test.ts` |

### 代码命名

```typescript
// 变量/函数：camelCase
const userName = 'test'
function getUserById(id: string) {}

// 类/接口/类型：PascalCase
class UserService {}
interface UserProfile {}
type UserRole = 'admin' | 'user'

// 常量：UPPER_SNAKE_CASE
const MAX_RETRY_COUNT = 3
const API_BASE_URL = 'https://api.example.com'

// 私有属性：以 _ 开头（仅 class 内部）
class Cache {
  private _store: Map<string, unknown>
}

// 布尔变量：is/has/can/should 前缀
const isLoading = true
const hasPermission = false
```

## TypeScript 规范

```typescript
// 优先使用 interface，type 用于联合类型/工具类型
interface UserProps {
  id: string
  name: string
}

type Status = 'active' | 'inactive'
type Nullable<T> = T | null

// 显式返回类型（公共函数）
export function calculateTotal(items: Item[]): number {
  return items.reduce((sum, item) => sum + item.price, 0)
}

// 使用 enum 替代魔法值
enum HttpStatus {
  OK = 200,
  NotFound = 404,
  InternalError = 500,
}

// 泛型约束
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key]
}
```

## 不可变性实践

```typescript
// WRONG: 修改原对象
function addItem(cart: Cart, item: Item): Cart {
  cart.items.push(item) // MUTATION!
  return cart
}

// CORRECT: 创建新对象
function addItem(cart: Cart, item: Item): Cart {
  return {
    ...cart,
    items: [...cart.items, item],
  }
}

// 数组操作：使用展开运算符或 slice/filter/map
const added = [...arr, newItem]
const removed = arr.filter(item => item.id !== targetId)
const updated = arr.map(item =>
  item.id === targetId ? { ...item, done: true } : item
)
```

## 导入规范

```typescript
// 分组排序：1. 外部库 → 2. 内部模块 → 3. 类型 → 4. 样式
import { ref, computed } from 'vue'
import { useRouter } from 'vue-router'

import { useUserStore } from '@/stores/user'
import { formatDate } from '@/utils/date'

import type { UserProfile } from '@/types/user'

import './style.css'
```

## 禁止清单

| 禁止项 | 原因 |
|--------|------|
| `console.log` | 用正式 logger 替代 |
| `any` 类型 | 用 `unknown` + 类型守卫 |
| 硬编码值 | 抽取为常量/配置 |
| 嵌套 > 4 层 | 提前返回/抽取函数 |
| 魔法数字/字符串 | 定义常量/枚举 |
| 直接修改参数 | 保持不可变性 |
| `eval()` / `Function()` | 安全风险 |

## 代码质量检查清单

- [ ] 代码可读且命名清晰
- [ ] 函数 <50 行，文件 <800 行
- [ ] 无深层嵌套（≤4 层）
- [ ] 错误处理完善
- [ ] 无 console.log
- [ ] 无硬编码值
- [ ] 使用不可变模式
- [ ] TypeScript 类型完整
